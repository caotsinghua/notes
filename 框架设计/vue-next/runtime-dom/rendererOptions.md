rendererOptions是用来创建渲染器的参数配置

HostNode类比dom中createElement,createTextNode等节点对象。

HostElement类比dom中getElementById获取的实际节点。

其类型为

```ts
export interface RendererOptions<
  HostNode = RendererNode,
  HostElement = RendererElement
> {
  patchProp(
    el: HostElement,
    key: string,
    prevValue: any,
    nextValue: any,
    isSVG?: boolean,
    prevChildren?: VNode<HostNode, HostElement>[],
    parentComponent?: ComponentInternalInstance | null,
    parentSuspense?: SuspenseBoundary | null,
    unmountChildren?: UnmountChildrenFn
  ): void
  insert(el: HostNode, parent: HostElement, anchor?: HostNode | null): void
  remove(el: HostNode): void
  createElement(
    type: string,
    isSVG?: boolean,
    isCustomizedBuiltIn?: string
  ): HostElement
  createText(text: string): HostNode
  createComment(text: string): HostNode
  setText(node: HostNode, text: string): void
  setElementText(node: HostElement, text: string): void
  parentNode(node: HostNode): HostElement | null
  nextSibling(node: HostNode): HostNode | null
  querySelector?(selector: string): HostElement | null
  setScopeId?(el: HostElement, id: string): void
  cloneNode?(node: HostNode): HostNode
  insertStaticContent?(
    content: string,
    parent: HostElement,
    anchor: HostNode | null,
    isSVG: boolean
  ): HostElement
}
```

可以看到，

其接收一个patchProp方法，用来更新属性

除此以外

- insert

  插入一个node到element

- remove

  移除一个node

- createElement

  创建一个element

- createText

  创建一个文本类型的node

  如`<span>测试</span>`中的测试文本。

- createComment

  创建一个注释类型的node

- setText

  给一个node设置text

- setElementText

  给一个element设置text

- parentNode

  todo，不知道干嘛的

- nextSibling

  获取兄弟node

- querySelector

  从一个选择器获取element

- setScopeId

  todo，不知道干嘛的

- cloneNode

  复制node的方法

- insertStaticContent

  todo：不知道干嘛的

  

在web环境，这些操作都对应到dom元素的获取，删除，插入，创建等到。

通过patchProp，可以针对不同的dom属性进行针对性的更新。

dom环境中的配置参见nodeOps和patchProp。



对于其他环境，这些方法都需要进行重新匹配。