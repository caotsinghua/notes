```
if (process.env.NODE_ENV !== 'production') {
      // 不是生产环境时执行initproxy
      initProxy(vm)
    } else {
      vm._renderProxy = vm
    }
```

init中，处理完options后，给vm加上_renderProxy.

当不是生产环境时，执行initProxy

```
initProxy = function initProxy (vm) {
    if (hasProxy) {
      // determine which proxy handler to use
      const options = vm.$options
      const handlers = options.render && options.render._withStripped
        ? getHandler
        : hasHandler
      vm._renderProxy = new Proxy(vm, handlers)
    } else {
      vm._renderProxy = vm
    }
  }
```

当hasProxy是false时，和将vm本身赋值给_renderProxy。

如果支持原生的proxy，则对vm的代理使用proxy实现。

```
options.render && options.render._withStripped
```

为true时，使用getHandler，否则使用hasHandler，通常只有测试中才为true，因此大部分情况都是hasHandler。

```
 vnode = render.call(vm._renderProxy, vm.$createElement)
```

render时，其执行上下文就是_renderProxy,默认是vm本身，但在开发模式下会添加handler，给与提示。

```js
// has 变量是真实经过 in 运算符得来的结果
const has = key in target
// 如果 key 在 allowedGlobals 之内，或者 key 是以下划线 _ 开头的字符串，则为真
const isAllowed = allowedGlobals(key) || (typeof key === 'string' && key.charAt(0) === '_')
// 如果 has 和 isAllowed 都为假，使用 warnNonPresent 函数打印错误
if (!has && !isAllowed) {
    warnNonPresent(target, key)
}
```

上面这段代码中的 `if` 语句的判断条件是 `(!has && !isAllowed)`，其中 `!has` 我们可以理解为**你访问了一个没有定义在实例对象上(或原型链上)的属性**，所以这个时候提示错误信息是合理，但是即便 `!has` 成立也不一定要提示错误信息，因为必须要满足 `!isAllowed`，也就是说当你访问了一个**虽然不在实例对象上(或原型链上)的属性，但如果你访问的是全局对象**那么也是被允许的。这样我们就可以在模板中使用全局对象了，如：

```
<template>
  {{Number(b) + 2}}
</template>
```



### initLifecycle

```
// 调用初始化生命周期
// 主要是初始化一些属性，会在生命周期中用到
export function initLifecycle (vm: Component) {
  const options = vm.$options
 // 缓存$options
  // locate first non-abstract parent
  let parent = options.parent
  // 找到第一个不是abstract的parent
  // parent存在，且当前vm.$options.abstract=false
  if (parent && !options.abstract) {
    while (parent.$options.abstract && parent.$parent) {
      // 跳过抽象的父组件，
      // 如果parent是abstract的，就到parent.parent.
      parent = parent.$parent
    }
    // 添加children为当前实例
    parent.$children.push(vm)
  }

  // 设置parent，是不含抽象组件的
  vm.$parent = parent
// 设置root
  vm.$root = parent ? parent.$root : vm

  vm.$children = []
  vm.$refs = {}

  vm._watcher = null
  vm._inactive = null //
  vm._directInactive = false //
  vm._isMounted = false
  vm._isDestroyed = false
  vm._isBeingDestroyed = false
}
```

设置组件配置的abstract=true，设为抽象组件，那么通过该组件创建的实例也是抽象的。

抽象组件不渲染真实dom，也不会出现在父子关系路径上。

## init中的一些属性初始化和钩子调用

```
initLifecycle(vm) // 初始化生命周期相关的一些属性,如$parent,$root,$children,这些路径关系,以及_inactive,_isDestroyed等状态信息
    initEvents(vm) // 添加两个实例属性 _events 和 _hasHookEvent
    initRender(vm) // 添加实例属性 _vnode,_staticTrees .$vnode $slots $scopedSlots  _c $createElement $attrs $listeners
    callHook(vm, 'beforeCreate') // 调用钩子函数,beforeCreate
    initInjections(vm) // resolve injections before data/props
    initState(vm)
    initProvide(vm) // resolve provide after data/props
    callHook(vm, 'created')
```

 其中 `initState` 包括了：`initProps`、`initMethods`、`initData`、`initComputed` 以及 `initWatch`。所以当 `beforeCreate` 钩子被调用时，所有与 `props`、`methods`、`data`、`computed` 以及 `watch` 相关的内容都不能使用，当然了 `inject/provide` 也是不可用的。 

 作为对立面，`created` 生命周期钩子则恰恰是等待 `initInjections`、`initState` 以及 `initProvide` 执行完毕之后才被调用，所以在 `created` 钩子中，是完全能够使用以上提到的内容的。但由于此时还没有任何挂载的操作，所以在 `created` 中是不能访问DOM的，即不能访问 `$el`。 

最后我们注意到 `callHook` 函数的最后有这样一段代码：

```
if (vm._hasHookEvent) {
  vm.$emit('hook:' + hook)
}
```

其中 `vm._hasHookEvent` 是在 `initEvents` 函数中定义的，它的作用是判断是否存在**生命周期钩子的事件侦听器**，初始化值为 `false` 代表没有，当组件检测到存在**生命周期钩子的事件侦听器**时，会将 `vm._hasHookEvent` 设置为 `true`。那么问题来了，什么叫做**生命周期钩子的事件侦听器**呢？大家可能不知道，其实 `Vue` 是可以这么玩儿的：

```
<child
  @hook:beforeCreate="handleChildBeforeCreate"
  @hook:created="handleChildCreated"
  @hook:mounted="handleChildMounted"
  @hook:生命周期钩子
 />
```

如上代码可以使用 `hook:` 加 `生命周期钩子名称` 的方式来监听组件相应的生命周期事件。这是 `Vue` 官方文档上没有体现的，但你确实可以这么用，不过除非你对 `Vue` 非常了解，否则不建议使用。

