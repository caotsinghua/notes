# overflow

### 剪裁界线：border-box

一个设置了overflow-hidden的元素，如果同时具有border和padding

**则当子元素内容超出宽高限制时，剪裁边界是borderbox的内边缘，而非paddingbox的内边缘。**

！*兼容问题*

在chrome下，如果容器触发滚动（垂直），则padding-bottom也算在滚动尺寸内，而ff和ie则忽略paddingbottom。

因此，要注意不使用padding-bottom，除了样式问题，还会导致scrollHeight不一样。

### overflow-x和overflow-y

如果其中一个设置为visible，另一个设置为scroll/auto/hidden，则visible的样式会表现为auto。

即*除非x，y都设置为visible，否则visible都作为auto解析*，也就是说，永远不可能一个方向溢出裁剪或者滚动，而另一方溢出显示的效果。



### 滚动条

自IE8起，html，textarea都默认overflow：auto。即溢出可出现滚动条。

对ie7，则一直出现，如同设overflow-y：scroll一样。

#### 几个结论

1. pc端，无论什么浏览器，默认滚动条均来自html，而非body。因此禁用默认滚动条只需要：

   ```
   html{overflow:hidden}
   ```

   **注意：**上述规则只对pc有效。

2. 滚动条会占用容器的可用宽高。

   在移动端，滚动条一般是悬浮模式，不占用宽高。在pc端，ie7以上，chrome，ff滚动条宽度均为17px。

*关于去除滚动条，可以看 基础问题/滚动条占用宽度问题*

#### 自定义滚动条

对于支持-webkit-前缀的浏览器

- 整体部分: `::webkit-scrollbar`
- 两端按钮:`::webkit-scrollbar-button`
- 外层轨道：`::webkit-scrollbar-track`
- 内层轨道：`::webkit-scrollbar-track-piece`
- 滚动滑块：`::webkit-scrollbar-thumb`
- 边角：`::webkit-scrollbar-corner`

常用如下:

```
::-webkit-scrollbar {
            /* 血槽宽度 */
            width: 8px;
            height: 8px;
        }

        ::-webkit-scrollbar-thumb {
            /* 拖动条 */
            background: rgb(0, 0, 0);
            border-radius: 6px;
            transition: .3s;
        }
        ::-webkit-scrollbar-thumb:hover{
            background: rgba(83, 83, 83);
        }

        ::-webkit-scrollbar-track {
            /* 背景槽 */
            background-color: #ddd;
            border: 1px solid;
            border-radius: 6px;
        }
```

### overflow与锚点定位

