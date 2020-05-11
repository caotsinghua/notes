# props更新原理

以vue2.6为例。

通常这样使用props。

```js
Vue.component('Child',{
	props:['parentMessage'],
	template:'<h1>{{parentMessage}}</h1>'
})
const Parent=new Vue({
    el:'#app',
	data(){
		return {
			parentMessage:'11'
		}
	},
	render(h){
		return h('Child',{ props:{parentMessage:this.parentMessage} })
	}
})
```

这里定义一个子组件，接收parentMessage的props。

现在开始分析：

### props参数做了什么？

vm._init会对props进行规范化，将数组，对象等形式都转化成这样，在这篇里不是重点。

```js
props:{
	key1:{
		type:null ,//
        default:null 
	}
}
```

然后在initState中，执行initProps.

这里注意propsData,vm._props两个重要参数。

首先获取的$options.propsData,可以知道这是props的数据来源，毕竟我们只定义了key，还没有值。

vm._props就是形成了{key:value}的对象。并且这个对象里的每个元素都是响应式的。

value的来源通过validateProp获取，这里会进行类型判断，获取默认值，对类型boolean的判断，取值等等操作，最终目的是获取prop的值。这里不展开讲。

**总之，props属性的作用就是产生一个响应式的vm._props对象，将prop的key和父组件传来的值进行组合，并且映射到vm上。**



```js
function initProps (vm: Component, propsOptions: Object) {
  const propsData = vm.$options.propsData || {} // 获取属性。在创建实例 时传入props，propsData的值没有监听
  const props = vm._props = {} // 设置_prop为对象

  const keys = vm.$options._propKeys = [] // 缓存prop的key
  const isRoot = !vm.$parent // 是否根实例,$parent在initLifeCycle时设置
  // root instance props should be converted
  if (!isRoot) {
    // 当不是根实例的时候不允许监测
    toggleObserving(false)
  }
  // 结构化的props的key
  // key:{type:''}
  for (const key in propsOptions) {
    keys.push(key)
    const value = validateProp(key, propsOptions, propsData, vm) // 监听每一个值
    defineReactive(props, key, value) // 监听每一个key
    // static props are already proxied on the component's prototype
    // during Vue.extend(). We only need to proxy props defined at
    // instantiation here.
    if (!(key in vm)) {
      // vm中不存在这个key时,代理到实例
      proxy(vm, `_props`, key)
    }
  }
  toggleObserving(true)
}
```



### propsData从哪里来？propsData没有被监听响应，如何触发子组件更新？

在initProps等initState方法结束后，就执行了`vm.$mount(vm.$options.el)`。

很明显的，这里是created之后开始挂载了。

从挂载开始分析。

platforms/web/runtime中设置

```js
Vue.prototype.$mount = function (
  el?: string | Element,
  hydrating?: boolean
): Component {
  el = el && inBrowser ? query(el) : undefined
  // el可能是undefined
  return mountComponent(this, el, hydrating)
}
```

在core/lifecycle.js中的mountComponent

主要执行这个

```js
export function mountComponent (){
    updateComponent = () => {
      // 用render函数创建的结果去update dom

      vm._update(vm._render(), hydrating)
}
// updateComponent 是被观测的目标
  // noop是一个空函数,表示当notify时不做动作
  // 实际上数据的变化不仅仅会执行回调，还会重新对“被观察目标”求值，也就是说 updateComponent 也会被调用，所以不需要通过执行回调去重新渲染。
  new Watcher(vm, updateComponent, noop, {
    before () {
      if (vm._isMounted && !vm._isDestroyed) {
        // 一个触发函数,当isMounted标记为true且未卸载时调用
        // 数据变化后,更新前调用beforeupdate钩子
        callHook(vm, 'beforeUpdate')
      }
    }
  }, true /* isRenderWatcher */)
}

```

这里显然是**用_render函数产生的新的vnode去更新原来的vnode**。用以更新视图。

即：用vm的新产生的vnode去更新旧的vnode。

watcher里就是对render用到的动态值进行依赖收集，当数据变化时重新updateCompoennt.

那么_render做了什么？

在instance/render.js中，给Vue原型加入了_render方法

```js
Vue.prototype._render = function (): VNode {
	 const vm: Component = this
    const { render, _parentVnode } = vm.$options
     currentRenderingInstance = vm
      vnode = render.call(vm._renderProxy, vm.$createElement)
}
```

主要就是执行了vm.render(vm.$createElement)

render方法是编译template产生或者在配置中写。类似这样

```js
new Vue({
render(h){
	return h('Child',{ props:{parentMessage:this.parentMessage} })
}
})
```

这里的h就是$createElement了。

如果是编译产生的render，h就是_c

可以在render中的initRender中看到

```js
// bind the createElement fn to this instance
  // so that we get proper render context inside it.
  // args order: tag, data, children, normalizationType, alwaysNormalize
  // internal version is used by render functions compiled from templates
  // 对于 vm._c 方法，则用于编译器根据模板字符串生成的渲染函数的。vm._c 和 vm.$createElement 的不同之处就在于调用 createElement 函数时传递的第六个参数不同，
  vm._c = (a, b, c, d) => createElement(vm, a, b, c, d, false)
  // normalization is always applied for the public version, used in
  // user-written render functions.
  vm.$createElement = (a, b, c, d) => createElement(vm, a, b, c, d, true)
```

所以又要看createElement方法做了什么。

到vdom/create-element.js中

```js
export function createElement (
  context: Component,
  tag: any,
  data: any,
  children: any,
  normalizationType: any,
  alwaysNormalize: boolean
): VNode | Array<VNode> {
	return _createElement(context, tag, data, children, normalizationType)
}
```

就是直接调用了_createElement

```js
export function _createElement (
  context: Component,
  tag?: string | Class<Component> | Function | Object,
  data?: VNodeData,
  children?: any,
  normalizationType?: number
): VNode | Array<VNode> {
	 // 创建组件
      // component
      vnode = createComponent(Ctor, data, context, children, tag)

}
```

由于我们是创建组件，因此只看这一行，调用了createComponent

```js
export function createComponent (){
	// extract props
  // 获取propsdata
  const propsData = extractPropsFromVNodeData(data, Ctor, tag)
        // install component management hooks onto the placeholder node
  	  installComponentHooks(data)
  // return a placeholder vnode
  const name = Ctor.options.name || tag
  // new vnode
  // 得到新的vnode对象
  const vnode = new VNode(
    `vue-component-${Ctor.cid}${name ? `-${name}` : ''}`,
    data, undefined, undefined, undefined, context,
    { Ctor, propsData, listeners, tag, children },
    asyncFactory
  )
    return vnode
}
```

可以看到，propsData的来源，就是通过**extractPropsFromVNodeData**方法得到的。

之后便产生了组件的vnode。

那么关键在于extractPropsFromVNodeData。

具体解析查看下面代码注释。

**总体来说，从父组件获取到了具体的propsData的内容。**

```js
export function extractPropsFromVNodeData (
  data: VNodeData,
  Ctor: Class<Component>,
  tag?: string
): ?Object {
const propOptions = Ctor.options.props
const res = {} // 返回的propsData
 const { attrs, props } = data
if (isDef(attrs) || isDef(props)) {
    for (const key in propOptions) {
    	 const altKey = hyphenate(key) // 转换格式，camelCase->camel-case
         // 因为有时在template使用camel-case格式，在props中又用了camelCase
    	  // 检查key
      // 如果没有定义props就去attrs中找
      // 如果定义了props就不去attrs中找了
      // 因为对于template生成的render函数，调用_c时没有props，数据都放在了attrs中
      checkProp(res, props, key, altKey, true) ||
      checkProp(res, attrs, key, altKey, false)
   }
}
return res
}


```

```js
function checkProp (
  res: Object,
  hash: ?Object, // props / attrs
  key: string,
  altKey: string,
  preserve: boolean // prop true
): boolean {
  if (isDef(hash)) {
    if (hasOwn(hash, key)) {
        // 先找camelCase的key，
      res[key] = hash[key]
      if (!preserve) {
        // attrs，把attrs中的数据删掉
        delete hash[key]
      }
      return true
    } else if (hasOwn(hash, altKey)) {
        // 找camel-case形式的key
      res[key] = hash[altKey]
      if (!preserve) {
           // attrs，把attrs中的数据删掉
        delete hash[altKey]
      }
      return true
    }
  }
  return false
}
```



>到这里，我们已经得到了新的vnode，并且vnode的数据（主要是propsData）也全部得到了。

### vm._update

vm._update也是在lifecycle文件中设置的。

```js
export function lifecycleMixin (Vue: Class<Component>) {
  Vue.prototype._update = function (vnode: VNode, hydrating?: boolean) {
```

在vm._update中主要做这些

```js
if (!prevVnode) {
      // 没有产生过vnode
      // initial render
      // 重新设置el
      vm.$el = vm.__patch__(vm.$el, vnode, hydrating, false /* removeOnly */)
    } else {
      // 直接更新node，并更新$el
      // updates
      vm.$el = vm.__patch__(prevVnode, vnode)
    }
```

可以看到主要是用新产生的vnode作为参数执行`__patch__`

在platforms/web/runtime/index中

```js
import { patch } from './patch'
Vue.prototype.__patch__ = inBrowser ? patch : noop
```

在./patch中

```js
import { createPatchFunction } from 'core/vdom/patch'
export const patch: Function = createPatchFunction({ nodeOps, modules })
```

在core/vdom/patch中,由于createPatchFunction太长，这里只提取出最主要的更新vnode的部分。

```js
function createPatchFunction(){
	  return function patch (oldVnode, vnode, hydrating, removeOnly) {
      	if (!isRealElement && sameVnode(oldVnode, vnode)) {
        // patch existing root node
        patchVnode(oldVnode, vnode, insertedVnodeQueue, null, null, removeOnly)
        }
         return vnode.elm
      }
}
```

这里的关键是patchVnode方法。

可以看到，主要用i这个方法在进行更新，而i的值来自于vnode.data.hook.

可能是prepatch,insert,update等等。

```js

  function patchVnode (
    oldVnode,
    vnode,
    insertedVnodeQueue,
    ownerArray,
    index,
    removeOnly
  ) {
        let i
        // 可以看到，关键是用i来进行更新
        
    const data = vnode.data
    if (isDef(data) && isDef(i = data.hook) && isDef(i = i.prepatch)) {
      i(oldVnode, vnode)
    }

    const oldCh = oldVnode.children
    const ch = vnode.children

    if (isDef(data) && isPatchable(vnode)) {
      for (i = 0; i < cbs.update.length; ++i) cbs.update[i](oldVnode, vnode)
      if (isDef(i = data.hook) && isDef(i = i.update)) i(oldVnode, vnode)
    }
  }
```

到core/vdom/create-component中

```js
const componentVNodeHooks = {
  init (vnode: VNodeWithData, hydrating: boolean): ?boolean {
    if (
      vnode.componentInstance &&
      !vnode.componentInstance._isDestroyed &&
      vnode.data.keepAlive
    ) {
      // kept-alive components, treat as a patch
      const mountedNode: any = vnode // work around flow
      componentVNodeHooks.prepatch(mountedNode, mountedNode)
    } else {
      // 当vnode没有mount过，九执行createComponentInstanceForVnode
      // 创建componentInstance
      const child = vnode.componentInstance = createComponentInstanceForVnode(
        vnode,
        activeInstance
      )
      // 执行挂载
      child.$mount(hydrating ? vnode.elm : undefined, hydrating)
    }
  },

  prepatch (oldVnode: MountedComponentVNode, vnode: MountedComponentVNode) {
    const options = vnode.componentOptions
    const child = vnode.componentInstance = oldVnode.componentInstance
    updateChildComponent(
      child,
      options.propsData, // updated props
      options.listeners, // updated listeners
      vnode, // new parent vnode
      options.children // new children
    )
  },
  }
```

观察prepatch,使用updateChildComponent进行更新。

再次进入core/instance/lifecycle.js

**更新props相关的代码主要是这样，可以看到，是拿到新的propsData后，去遍历原先的`vm._props`,并进行赋值，由于`_props`是已经响应式的，因此对其元素更新就会触发vm的界面更新。**

```js
export function updateChildComponent (
  vm: Component,
  propsData: ?Object,
  listeners: ?Object,
  parentVnode: MountedComponentVNode,
  renderChildren: ?Array<VNode>
) {
	  // update props
  if (propsData && vm.$options.props) {
    toggleObserving(false)
    const props = vm._props // 拿到vm._props的引用
    const propKeys = vm.$options._propKeys || []
    for (let i = 0; i < propKeys.length; i++) {
      const key = propKeys[i]
      const propOptions: any = vm.$options.props // wtf flow?
      props[key] = validateProp(key, propOptions, propsData, vm) // 触发更新
    }
    toggleObserving(true)
    // keep a copy of raw propsData
    vm.$options.propsData = propsData
  }
}
```



最后做个总结，我们在创建vm时定义了props属性，只是定义了一些key，vue会在创建当前vm的vnode的时候去拿到父组件的prop值，创建propsData对象，并且设置了响应式的vm._props保存prop的key和值，并代理到vm上。

当父组件的数据更新后，需要触发子组件的prop更新，会通过修改子组件的_props的key，触发子组件更新。

