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

锚点定位的方法：

1. ```html
   <a href="#1">发展历程</a>
   <a name="1">锚点</a>
   ```

2. ```
   <a href="#1">发展历程</a>
   <h1 id="1"></h1>
   ```

**锚点定位的触发条件**

1. url地址中的锚链与锚点元素对应并有交互行为。
2. 可focus的锚点出于focus状态

*前提是锚链值可以找到页面中对应的元素，并且是**非隐藏**状态。*

如果锚链值是一个简单的#，则定位行为是回到顶部。



**focus锚点定位**指的是类似链接或按钮，输入框等可以被focus的元素在被focus时发生的页面重定位现象。

比如tab快速定位可focus的元素，如果元素在屏幕外，则浏览器会自动重定位，将该元素定位到屏幕中。

或者`document.querySelector('input').focus()`

**差异**：url地址锚链定位是让元素定位在浏览器窗体上边缘，而focus锚点定位只是让元素在和、窗体范围内显示，不一定是上边缘。



#### 锚点定位的本质

锚点定位行为的发生，本质是通过改变**容器**滚动高度/宽度实现的。

这也也可以滚动

```
<div class="box">
        <div class="content"></div>
        <h4 id="title">底部标题</h4>
    </div>
    <a href="#title">点击测试</a>
   .box{
            height: 100px;
            border: 1px solid;
            overflow: auto;

        }
        .content{
            height: 400px;
            background-color: #eee;
        }
```



- **由内而外**：普通元素和窗体同时可滚动时，会由内而外的触发所有可滚动窗体的锚点定位行为。

  在上面例子基础上添加

  ```
  <div class="box">
          <div class="content"></div>
          <h4 id="title">底部标题</h4>
      </div>
      <a href="#title">点击测试</a>
      <div style="height: 800px;"></div>
  ```

  

#### overflow:hidden的元素也是可滚动，与auto/scroll的区别仅在于没有滚动条。

- 利用overflowhidden和锚点实现轮播图

  ```
  div class="container">
          <div class="slider">
              <ul class="wrap">
                  <li class="item" id="1">1</li>
                  <li class="item" id="2">2</li>
                  <li class="item" id="3">3</li>
                  <li class="item" id="4">4</li>
              </ul>
          </div>
          <div>
              <a href="#1">1</a>
              <a href="#2">2</a>
              <a href="#3">3</a>
              <a href="#4">4</a>
          </div>
          <div style="height:999px"></div>
      </div>
  <style>
          ul{
              margin:0;
              padding:0;
              display: flex;
          }
          .item{
              height: 100px;
              background-color:rosybrown;
              list-style: none;
              width: 300px;
              flex-shrink: 0;
          }
          .slider{
              width: 300px;
              border: 1px dashed;
              height: 100px;
              overflow: hidden;
              scroll-behavior: smooth;
          }
  
      </style>
  ```

  可以通过锚点定位来定位到item的位置。

  缺点：1. 要固定宽度/对于垂直的要固定高度

  2.如果页面同时有滚动条，则会由内而外的触发定位，将锚点元素定位到界面顶部，引起页面移动。

- 使用focus构建标签选择

  ```
  <div class="container">
          <ul class="slider">
              <li class="item">
                  <input id="1" class="slide-input">
                  <span>1</span>
              </li>
              <li class="item">
                  <input id="2" class="slide-input">
                  <span>2</span>
              </li>
              <li class="item">
                  <input id="3" class="slide-input">
                  <span>3</span>
              </li>
              <li class="item">
                  <input id="4" class="slide-input">
                  <span>4</span>
              </li>
          </ul>
      </div>
      <div>
          <ul>
              <li><label for="1">1</label></li>
              <li><label for="2">2</label></li>
              <li><label for="3">3</label></li>
              <li><label for="4">4</label></li>
          </ul>
      </div>
      <style>
          ul{
              list-style: none;
              margin: 0;
              padding: 0;
  
          }
          .container{
              width: 300px;
              overflow: hidden;
              border: 1px dashed;
              scroll-behavior: smooth;
              overflow: hidden;
          }
          .slider{
              display:inline-flex;
              flex-wrap: nowrap;
          }
          .item{
              width:300px;
              height: 200px;
              background: #eee;
              flex-shrink: 0;
              position: relative;
          }
          .slide-input{
              position: absolute;
              top: 0;
              left: 50%;
              width: 1px;
              height: 1px;
              border: 0;
              padding: 0;
              clip: rect(0,0,0,0);
          }
      </style>
  ```

  如果内容在可视窗口外依然会出现跳动的问题，可以使用js解决。

关于选项卡切换，还可以使用checked伪类+label+单选按钮实现

*还可以基于overflowhidden可滚动的特点，自定义滚动条*

即父元素overflowhidden，然后基于父元素自身的scrolltop改变来实现滚动条效果。

好处：

1. 实现简单，无需边界判断。就算scrolltop值-999，浏览器也按照0渲染。
2. 可与原生的scroll时间天然集成。如滚动延迟加载图片效果，图片位置计算往往和scrolltop相关。
3. 无需改变子元素结构。



