### 树形结构

> 可以选择相关组件

1. element-ui的table组件中的treegrid功能，用来做tree类型的增删改查
2. tree组件，有更明确的树形结构
3. tree-select组件用于树形选择

### 动态路由权限管理

1. 由前端根据返回后端返回的用户角色进行路由的筛选

   > 将用户角色存储在store中，在路由的meta中存放access表示该页面需要的角色。
   >
   > 每次beforeRouter时判断角色，遍历配置过权限的路由，将有权限的路由通过router.addRoutes加入到路由中，再重新next({...to})，使新路由生效
   >
   > 参考:[链接](https://github.com/PanJiaChen/vue-element-admin/blob/master/src/permission.js)

2. 后端动态配置路由

   > 后端配置路由名称(也可以是路径，保证能匹配到前端配置的路由即可),icon，isShow等字段，返回给前端登陆用户可访问的界面数组。
   >
   > 前端配置静态路由。
   >
   > 每次beforeRouter时，得到用户拥有的路由，加入到router中。
   >
   > 与1方法的区别在于如何获取当前用户的路由。

   获取到的路由表类似这样：

   ```json
   {"success":true,"code":"0","message":"Success","data":[{"privId":69,"parentPrivId":68,"privType":"M","privUri":"operate-log","privIcon":"ios-list-box-outline","privCode":"operate-log"},{"privId":41,"parentPrivId":0,"privType":"C","privUri":"system","privIcon":"","privCode":"system"},{"privId":15,"parentPrivId":0,"privType":"C","privUri":"sell-manager","privIcon":"","privCode":"sell-manager"},{"privId":44,"parentPrivId":41,"privType":"M","privUri":"roles-management","privIcon":"","privCode":"roles-management"},{"privId":34,"parentPrivId":33,"privType":"C","privUri":"statistics-business","privIcon":"","privCode":"statistics-business"},{"privId":68,"parentPrivId":0,"privType":"C","privUri":"operate","privIcon":"ios-list-box-outline","privCode":"operate"},{"privId":37,"parentPrivId":33,"privType":"C","privUri":"statistics-salesmans","privIcon":"","privCode":"statistics-salesmans"},{"privId":60,"parentPrivId":38,"privType":"M","privUri":"todo","privIcon":"","privCode":"todo"},{"privId":35,"parentPrivId":33,"privType":"C","privUri":"statistics-companys","privIcon":"","privCode":"statistics-companys"},{"privId":40,"parentPrivId":38,"privType":"M","privUri":"done","privIcon":"","privCode":"done"},{"privId":43,"parentPrivId":41,"privType":"M","privUri":"users-management","privIcon":"","privCode":"users-management"},{"privId":36,"parentPrivId":33,"privType":"C","privUri":"statistics-managers","privIcon":"","privCode":"statistics-managers"},{"privId":38,"parentPrivId":0,"privType":"C","privUri":"workflow","privIcon":"","privCode":"workflow"},{"privId":42,"parentPrivId":41,"privType":"M","privUri":"dict-management","privIcon":"","privCode":"dict-management"},{"privId":46,"parentPrivId":41,"privType":"M","privUri":"privs-management","privIcon":"","privCode":"privs-management"},{"privId":33,"parentPrivId":0,"privType":"C","privUri":"statistics","privIcon":"","privCode":"statistics"},{"privId":58,"parentPrivId":15,"privType":"M","privUri":"customers","privIcon":"","privCode":"cusotmers"},{"privId":47,"parentPrivId":17,"privType":"M","privUri":"analysis","privIcon":"","privCode":"analysis"},{"privId":23,"parentPrivId":15,"privType":"M","privUri":"contact-book","privIcon":"","privCode":"contact-book"},{"privId":48,"parentPrivId":17,"privType":"M","privUri":"basic","privIcon":"","privCode":"basic"},{"privId":17,"parentPrivId":58,"privType":"C","privUri":"customer","privIcon":"","privCode":"customer"},{"privId":49,"parentPrivId":17,"privType":"M","privUri":"organization","privIcon":"","privCode":"organization"},{"privId":24,"parentPrivId":15,"privType":"M","privUri":"daily","privIcon":"","privCode":"daily"},{"privId":50,"parentPrivId":17,"privType":"M","privUri":"customer-contact-book","privIcon":"","privCode":"customer-contact-book"},{"privId":25,"parentPrivId":15,"privType":"M","privUri":"orders","privIcon":"","privCode":"orders"},{"privId":51,"parentPrivId":17,"privType":"M","privUri":"customer-daily","privIcon":"","privCode":"customer-daily"},{"privId":26,"parentPrivId":15,"privType":"M","privUri":"business","privIcon":"","privCode":"business"},{"privId":52,"parentPrivId":17,"privType":"M","privUri":"customer-orders","privIcon":"","privCode":"customer-orders"},{"privId":53,"parentPrivId":17,"privType":"M","privUri":"customer-business","privIcon":"","privCode":"customer-business"}]}
   ```

   与前端配置的路由进行匹配，过滤出需要的路由（路由顺序还是前端配置顺序）。

   ```javascript
   export function mapPrivsDataToRouter(privs, routes) {
       const menuMap = {};
       privs.forEach(priv => {
           if (priv.privUri && priv.privType !== 'A') {
               menuMap[priv.privUri] = {
                   ...priv,
                   icon: priv.privIcon,
                   path: priv.privUri,
                   name: priv.privUri
               };
           }
       });
     
       function parse(menuMap, routes) {
           const result = [];
           for (let i = 0; i < routes.length; i++) {
               const priv = menuMap[routes[i].name]; // 根据name定位
               if (priv && (priv.isShow === undefined || priv.isShow)) {
                   let children = [];
                   if (routes[i].children) {
                       children = parse(menuMap, routes[i].children);
                   }
                   if (children.length > 0) {
                       routes[i].children = children;
                   } else {
                       delete routes[i].children;
                   }
                   if (priv.icon) {
                       routes[i].meta.icon = priv.icon;
                   }
   
                   result.push(routes[i]);
               }
           }
           return result;
       }
   
       return { routes: parse(menuMap, routes), menuMap };
   }
   ```

   再beforeRouter中判断是否登陆，已登录则判断权限路由

   ```js
    if (hasGetPrivs) {
                  
                   next();
               } else {
                   try {
                       const privs = await store.dispatch('getUserPrivs');
                       // 全局管理员
                       const isAdmin = roles && roles.some(role => role === 'A' || role === 'B');
   
                       const { routes: parsedDynamicRoutes, menuMap } = mapPrivsDataToRouter(
                           privs,
                           dynamicRoutes,
                           isAdmin
                       );
                       router.addRoutes([...parsedDynamicRoutes, ...staticRoutes]);
                       store.commit('setRoutes', [...routes, ...parsedDynamicRoutes, ...staticRoutes]);
                       store.commit('setMenuMap', menuMap);
                   } catch (e) {
                       console.log(e);
                   }
                   next({ ...to, replace: true });
               }
   ```

   

3. 后端配置路由和组件

   > 当用户登录后得到可访问的路由表，从而动态生成可访问页面，之后就是 router.addRoutes 动态挂载到 router 上。
   >
   > 与2方法的区别在于，如何映射路由组件

   映射组件

   ```
   const map={
    login:require('login/index').default // 同步的方式
    login:()=>import('login/index')      // 异步的方式
   }
   //你存在服务端的map类似于
   const serviceMap=[
    { path: '/login', component: 'login', hidden: true }
   ]
   //之后遍历这个map，动态生成asyncRoutes
   并将 component 替换为 map[component]
   ```

   

### 退出登陆后重置路由

像之前做好路由权限匹配后，用户退出登陆使用ajax是不会刷新界面的，此时vue的router还是之前用户的路由，新用户重新登陆后还会addRoutes进去，导致的问题就是新用户可以访问自己没权限但旧用户有权限的页面。

解决方法：

1.退出登陆后window.location.reload刷新页面，会有界面白屏的问题

2.退出登陆后重新new Router，重新挂载到vue上

​	问题：

 1. 在vue实例外使用router时需要知道当前router

 2. 获取当前根vue实例

 3. 退出登陆后重回登陆界面存在验证码重复刷新的问题，重复调用接口，由于返回结果的顺序不同导致验证码数据不一致的问题。

    解决方法：

    使用一个对象来引用当前路由和app实例等信息。

    当isRebuild为true表示是重新创建app，不需要再调用一次验证码接口。

    ```javascript
    export const appContainer = {
        router: null,
        vm: null,
        isRebuild: false
    };
    export function initApp() {
        const router = createRouter();
        appContainer.router = router;
        appContainer.vm = new Vue({
            el: '#app',
            router,
            // i18n,
            store,
            render: h => h(App)
        });
    }
    ```

### 使用cookie记录登陆状态与getUserInfo检查登陆状态兼容

store中保存isLogged表示是否登陆，在每次新建vue实例前判断登陆态.

使用服务端判断登陆态可以直接调用await store.dispatch('getUserStatus', hasLogged);来设置登陆状态。

使用cookie可以在获取到cookie后直接设置store。

其他代码就可以保持不变。

```javascript
const initAppByStatus = async () => {
    iView.Spin.show({
        render: h => {
            return h('div', [
                h('Icon', {
                    class: 'index-spin-load',
                    props: {
                        type: 'ios-loading',
                        size: 18
                    }
                }),
                h('div', '正在加载，请稍后...')
            ]);
        }
    });
    try {
        await store.dispatch('getUserStatus');
    } finally {
        iView.Spin.hide();
        initApp();
    }
};

// 通过token判断登陆态
const initAppByToken = async () => {
    const router = createRouter();
    appContainer.router = router;
    const hasLogged = !!getToken();
    await store.dispatch('getUserStatus', hasLogged);
    appContainer.vm = new Vue({
        el: '#app',
        router,
        // i18n,
        store,
        render: h => h(App)
    });
};
```

