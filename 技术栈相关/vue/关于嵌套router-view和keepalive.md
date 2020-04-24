# 嵌套router-view和keep-alive出现路由和界面不一致的问题

## 问题描述：

在app.vue里定义router-view，并keepalive部分页面，包含TabContainer页面。

在TabContainer页面中，有两个路由界面，在router-view中，被keep-alive。一个是首页，一个是我的。

当进入首页，切换到我的，再退出登陆后。重新登陆，push的是home路由，渲染的却是me界面。

路由显示和渲染内容不一致。

## 渲染原理

router-view是一个函数组件，是不会被keep-alive缓存的，其主要负责产生vnode。

当routerview渲染的vnode变化，会加到routerView的缓存中，每次只会存一个vnode。（可以根据路由缓存多个vnode，见解决方案2）

router-view最终返回缓存数据的时候，会把传递给router-view的data都重新给缓存的组件。

```js
return h(cachedComponent, data, children)
```

keep-alive拿到slot.default的第一个非函数组件的时候，会根据其key检查cache，拿到之前缓存的组件。



## 预期目标：

退出登陆后，重新登陆，跳转到首页，应该路由和界面一致。

对于任何一个界面都应该这样。

## 原因分析：

当router-view组件进行渲染时，首先检查其所在的parent树是否有inactive状态的（即被keepalive，但未显示）。

parent从router-view所在的组件开始向上深度遍历。

同时拿到depth，即路由深度，以便从matched中拿到匹配路由。（这里不提）

```js
while (parent && parent._routerRoot !== parent) {
      const vnodeData = parent.$vnode ? parent.$vnode.data : {}
      if (vnodeData.routerView) {
        depth++
      }
      // _directInactive 。inactive是？
      // 目的是检查到父级树中有没有存在inactive的组件
      if (vnodeData.keepAlive && parent._directInactive && parent._inactive) {
        // 检查父级是否存在keepalive
        console.log({ parent, vnodeData })
        inactive = true
      }
      parent = parent.$parent
    }
```

这里只要遍历到的父级存在inactive状态，就设为true。比如app-tabContainer,tabContainer有home和me页面，只要有一个父级是inactive的，那当前路由的渲染就直接从缓存中读取。

这里要分情况：

1. 如果parent本身就是inactive，就像本例中，从login->tabContainer(home)界面时，tabContainer是被app页面keep-alive的，是inactive状态。那么显示该tabContainer中的router-view时，就会直接从缓存中读取，不会再匹配路由。而上次是从me界面退出，则再次返回也是me界面，即使路由显示的是/home，但并不会实际匹配。

   

2. 如果parent没有被keep-alive，但是更上面存在inactive的组件。

   如app(container(tab(subtab(view)))),只有container被keepalive。

   那么渲染view页面时，先从container开始，由于container被keepalive，那么container直接从缓存读取，以此类推，tab，subtab，view都不会再进行路由匹配。而是从之前的缓存得到。

```js
if (inactive) {
      // 如果上面有某个父组件是inactive的，则从缓存读取vnode
      const cachedData = cache[name] // 缓存的data
      const cachedComponent = cachedData && cachedData.component
      console.log('if inactive', {
        cachedData,
        cachedComponent
      })
      if (cachedComponent) {
        // #2301
        // pass props
        // 缓存的组件存在，检查configProps
        // todo：configProps

        if (cachedData.configProps) {
          console.log('存在configProps', {
            cachedData
          })
          fillPropsinData(cachedComponent, data, cachedData.route, cachedData.configProps)
        }

        console.log('渲染组件', h(cachedComponent, data, children))
        return h(cachedComponent, data, children)
      } else {
        // render previous empty view
        return h()
      }
    }
```

那么，当inactive为false，会怎么做？

inactive=false有几种情况：

1. 父级组件keep-alive，但是是active状态，也就是说，父级都在显示中。此时改变子路由，就不会走缓存，而是从matched到的component进行显示。但是每次component都会更新cache[name]

   > 当没有使用命名视图，则name=default
   >
   > 使用命名视图，则保存对应的name

   每次更新的时候，只更新一个key，因此每次缓存的都是最后一次访问的界面。

2. 没有keepalive的父组件，那么每次都走matched。同理，cache也会更新。

## 解决方法：

1. 在router-view加key

   渲染的是这样的

   ```js
asyncFactory: undefined
   asyncMeta: undefined
   children: undefined
   componentInstance: undefined
   componentOptions: {propsData: undefined, listeners: undefined, tag: undefined, children: undefined, Ctor: ƒ}
   context: VueComponent {_uid: 7, _isVue: true, $options: {…}, _renderProxy: Proxy, _self: VueComponent, …}
   data: {key: "/menu", routerView: true, routerViewDepth: 1, on: undefined, hook: {…}, …}
   devtoolsMeta: {renderContext: FunctionalRenderContext}
   elm: undefined
   fnContext: VueComponent {_uid: 7, _isVue: true, $options: {…}, _renderProxy: Proxy, _self: VueComponent, …}
   fnOptions: {components: {…}, directives: {…}, filters: {…}, beforeCreate: Array(3), _base: ƒ, …}
   fnScopeId: undefined
   isAsyncPlaceholder: false
   isCloned: true
   isComment: false
   isOnce: false
   isRootInsert: true
   isStatic: false
   key: "/menu"
   ns: undefined
   parent: undefined
   raw: false
   tag: "vue-component-33-me"
   text: undefined
   ```
   
   可见tag还是me组件，但是key是/menu，keep-alive根据这个key直接获取到home组件的缓存。从而渲染正确的页面。

   需要注意的是，每次路由变化，都会有一个key产生，即当从tabContainer跳转到其他页面时，由于路由先变化，导致key变化，引起router-view的更新，会创建一个key为目标路由的组件，但实例还是当前组件。（如跳转到登陆页面，登陆页面并不在tabContainer中，但还是会有一个login的key的缓存）

   解决方法是给key加个过滤，只有指定的key才缓存。

   ```
<keep-alive :include="cachedList">
                   <router-view :key="currentKey" v-if="currentKey"></router-view>
               </keep-alive>
               currentKey() {
               const whiteList = [HOME_NAME, 'me'];
               return whiteList.find(name => name === this.$route.name) || '';
           }
   ```
   
   这样缓存中会多一个key为''的组件实例。需要注意的是，要在router-view加v-if，否则router-view会重新渲染一次之前的页面。

   当然加v-if会出现一个瞬间的白屏，不是平滑的进行过渡。

2. 修改源码：

   每次读取/存缓存的时候，保存在`cache[name][routePath]`下，保持缓存和路由的一致性。

   同时原本的命名视图功能也没有破坏。

   问题：需要改源码。

   ```js
    if (inactive) {
         // 如果上面有某个父组件是inactive的，则从缓存读取vnode
         const cachedData = cache[name][routePath] // 缓存的data
         ...
         
    // cache component
       if (cache[name]) {
         cache[name][routePath] = { component }
       } else {
         cache[name] = {
           [routePath]: { component }
         }
       }
   ```

3. tabcontainer不keepalive。

4. 将tabcontainer中的keepalive去除，多个tab分多个页面进行缓存。

5. 在container页面的beforeRouterLeave的钩子中判断是否去登陆页面，如果是的话销毁当前页面。

   ```
   <keep-alive :include="cachedList">
                   <router-view></router-view>
               </keep-alive>
   beforeRouteLeave(to, from, next) {
           if (to.name === LOGIN_NAME) {
               this.$destroy();
           }
           next();
       },
   ```

6. 动态cachedList

   只有访问过的页面&cached设为true，才添加到cachedList中。如果重新登陆，就把cacheList清空。

   > 参考vue-admin
   >
   > logout({ commit, state, dispatch }) {
   >
   >   return new Promise((resolve, reject) => {
   >
   >    logout(state.token).then(() => {
   >
   > ​    commit('SET_TOKEN', '')
   >
   > ​    commit('SET_ROLES', [])
   >
   > ​    removeToken()
   >
   > ​    resetRouter()
   >
   > 
   >
   > ​    *// reset visited views and cached views*
   >
   > ​    *// to fixed https://github.com/PanJiaChen/vue-element-admin/issues/2485*
   >
   > ​    dispatch('tagsView/delAllViews', null, { root: true })
   >
   > 
   >
   > ​    resolve()
   >
   >    }).catch(error => {
   >
   > ​    reject(error)
   >
   >    })
   >
   >   })
   >
   >  },

## 注意：

1. app-container-home中，container里不使用keep-alive缓存home相关的router-view。这种做法是无效的。

   因为渲染到container时原本是inactive的，渲染home就直接从container的router-view的cache中获取了。