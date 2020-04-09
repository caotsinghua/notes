## rendererOptions

用来创建渲染器renderder

见rendererOptions.md

## renderer

延迟创建renderer，使得核心的renderer逻辑能够tree-shaking

用户只需要导入reactivity工具即可。

通过createRenderer(rendererOptions)创建。

> createRenderer来源runtime-core

```ts
// lazy create the renderer - this makes core renderer logic tree-shakable
// in case the user only imports reactivity utilities from Vue.
let renderer: Renderer | HydrationRenderer

// 保证有渲染器
function ensureRenderer() {
  return renderer || (renderer = createRenderer(rendererOptions))
}
```



## export render方法

>  调用渲染器的render方法.



## export createApp方法

>  调用渲染器的createApp方法

container是mount的html元素

改写app.mount方法，

获取app._component

如果component不是函数，没有render方法，没有template，则component.template是container.innerHtml

mount之前先清除container的的innerHtml

```ts
app.mount = (containerOrSelector: Element | string): any => {
    const container = normalizeContainer(containerOrSelector)
    if (!container) return
    const component = app._component
    if (!isFunction(component) && !component.render && !component.template) {
      component.template = container.innerHTML
    }
    // clear content before mounting
    container.innerHTML = ''
    const proxy = mount(container)
    container.removeAttribute('v-cloak')
    return proxy
  }
```

## 其他export

**一些指令**

```ts
// DOM-only runtime directive helpers
export {
  vModelText,
  vModelCheckbox,
  vModelRadio,
  vModelSelect,
  vModelDynamic
} from './directives/vModel'
export { withModifiers, withKeys } from './directives/vOn'
export { vShow } from './directives/vShow'

```

**一些动画相关组件**

```ts
// DOM-only components
export { Transition, TransitionProps } from './components/Transition'
export {
  TransitionGroup,
  TransitionGroupProps
} from './components/TransitionGroup'
```

**runtime-core**

```ts
// re-export everything from core
// h, Component, reactivity API, nextTick, flags & types
export * from '@vue/runtime-core'
```

