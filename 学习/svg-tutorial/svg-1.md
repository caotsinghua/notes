### 根元素svg

1.  应舍弃来自 (X)HTML的doctype声明，因为基于SVG的DTD验证导致的问题比它能解决的问题更多。 
2.  SVG 2之前，version属性和baseProfile属性用来供其他类型的验证识别SVG的版本。SVG 2不推荐使用version和baseProfile这两个属性。 
3.  作为XML的一种方言，SVG必须正确的绑定命名空间 （在xmlns属性中绑定）。 请阅读[命名空间速成](https://developer.mozilla.org/en/docs/Web/SVG/Namespaces_Crash_Course) 页面获取更多信息。 

### svg文件基本属性

1.  SVG文件全局有效的规则是“后来居上”，越后面的元素越可见。 

2.  svg文件可以直接在浏览器展示。也可以嵌入html文件。

   - XHTML且声明为application/xhtml+xml，则可以直接将svg潜入xml源码中。

   - html5，可以直接嵌入svg，但为符合h5标准，可能需要做一些语法调整。

     todo

   - 可以通过object引入svg文件

     ```html
      <object type="image/svg+xml" data="./demo1.svg"></object>
     ```

   - 类似的可以通过iframe引用svg文件

     ```html
      <iframe src="./demo1.svg" frameborder="0" style="height: 200px;"></iframe>
     ```

     通过iframe引入可能会存在滚动条，需要确定宽高。

   - 理论上也可以使用img元素，但在低于4.0的firefox中不起作用。

     ```html
      <img src="./demo1.svg" alt="">
     ```

   - svg也可以通过js动态创建并注入到html中。这样左

### 文件类型

> 由于在某些应用（比如地图应用等）中使用时，SVG文件可能会很大，SVG标准同样允许gzip压缩的SVG文件。推荐使用“.svgz”（全部小写）作为此类文件扩展名 。不幸的是，如果服务器是微软的IIS服务器，使gzip压缩的SVG文件在所有的可用SVG的用户代理上可靠地起作用是相当困难的，而且Firefox不能在本地机器上加载gzip压缩的SVG文件。 除非知道处理发布内容的web服务器可以正确的处理gzip，否则要避免使用gzip压缩的SVG。 

 对于普通SVG文件，服务器应该会发送下面的HTTP头： 

```
Content-Type: image/svg+xml
Vary: Accept-Encoding
```

 对于gzip压缩的SVG文件，服务器应该会发送下面的HTTP头： 

```
Content-Type: image/svg+xml
Content-Encoding: gzip
Vary: Accept-Encoding
```

