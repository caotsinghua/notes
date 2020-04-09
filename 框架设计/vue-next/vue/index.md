# vue/index.ts

这里是full-build的入口，会打包compile和运行时。



作为vue的入口，导出如下

1. compile
2. 全部的runtime-dom

执行方法：registerRuntimeCompiler

将compileToFunction 注入到runtime-dom的compile方法中。



### compileToFunction 

接收参数

```
template:string|HTMLElement
options?:CompilerOptions
```

如果template是html元素，则获取其innerHtml。

如果template是选择器，template就是选择到的该元素的innerhtml

*如果缓存中有template这个key，就直接返回该render。*

对template执行compile函数

> compile来源：'@vue/compiler-dom'

获取到{code}

若Vue是全局变量，则运行code，返回值就是render

Vue不是全局变量，则将runtimeDom作为code函数体的参数，返回值为render。

> runtimeDom来源： '@vue/runtime-dom'

最终返回render。

同时，对于template的render，加入到缓存中。



最终vue由4个部分构成：

1. compile 用来编译模板 (runtime不需要)
2. renderer 渲染器，用来渲染vdom和patch
3. reactivity 使数据变成响应式
4. 组件相关：生命周期等

