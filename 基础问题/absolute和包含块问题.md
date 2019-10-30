### `position:absolute`定位问题

#### 关于包含块

 在浏览器生成显示的页面的时候，每一个框都有一个定位，这个定位受其包含 块的影响，不过它不被包含块所限制，而且可能会溢出到包含块之外。 

根元素所在的包含块是初始包含块，html中根元素是html元素。

初始包含块的direction属性与根元素相同。（direction属性指定块的基本书写方向，表格列布局的方向，水平溢出的方向等）

初始包含块的宽度可以由根元素width指定，若为auto，用户端提供初始宽度，如视口当前宽。

初始包含块的高度可由height指定，如果是auto，包含块高度由内容决定。

初始包含块不能定位和浮动（忽略position和float）

对其他元素：

>1.该元素定位relative/static，它的包含块由它最近的块级、单元格（table-cell）或inline-block祖先元素的内容框创建。（因此可以看作相对元素本身定位，因为包含块的位置和元素本身位置一致）
>
>2.如果元素fixed，包含块由视口创建
>
>3.元素absolute，包含块由最近的absolute，relative，fixed祖先元素创建
>
>如果祖先为行内元素，包含块取决于祖先的direction，若为ltr，包含块的顶，左边是该祖先元素创建的第一个框的顶，左补白边padding，它的底，右边是祖先元素最后一个框的底，右补白边。
>
>如果是rtl，则包含块顶，右边是祖先创建的第一个框的顶，右补白边，它的底，左边是祖先创建的最后一个框的底，左补白边。
>
>

> The constraint that determines the used values for these elements is:
>
> > 'left' + 'margin-left' + 'border-left-width' + 'padding-left' + 'width' + 'padding-right' + 'border-right-width' + 'margin-right' + 'right' = width of containing block

#### 问题出现情况

> iview-admin中transfer属性将下拉框等元素放在body中，而下拉框内容使用position:absolute,在切换显隐时出现滚动条时隐时现的问题。

![](D:\mine-codes\notes\images\abs1.png)

- 解决方案

  在html或body设`overflow:hideen`

- 原因

  以下例子

  susp为absolute元素，相对item是relative进行定位。由item创建包含块；而item是relative，由父元素box创建包含块，最终box的包含块高度包含了item>susp的,而susp所在的包含块高度计算涵盖了top+height...(同上方宽度计算)。因此触发了box的滚动。

  - 例子

    ```
    <div class="box">
      <div class="item">
      我是列表项
        <div class="susp">我是悬浮框</div>
      </div>
    </div>
    
    .box {
      height: 200px;
      border: 1px solid red;
      overflow: auto;
    }
    .item {
      position: relative;
      height: 150px;
      border-bottom: 1px solid blue;
    }
    .susp {
      position: absolute;
      left: 0;
      top: 150px;
      height: 100px;
      background-color: #eee;
    }
    ```

    结果：

    ![](D:\mine-codes\notes\images\abs2.png)

- 总结

  在使用absolute进行定位时，有很多不同的需求。设置left\top？或者是否包裹在relative内？要具体分析。

参考

[CSS核心：包含块(Containing Block)]( https://wenku.baidu.com/view/013e92bd960590c69ec37616.html?sxts=1572400316952 )

[ 由一个绝对定位引发overflow：auto滚动问题产生的关于包含块（containing block）的思考 ]( https://www.ucloud.cn/yun/114187.html )

[ 使用绝对定位来模拟固定定位 ]( https://blog.csdn.net/m0_38066007/article/details/88087327 )

[[深入理解CSS绝对定位absolute](https://www.cnblogs.com/xiaohuochai/p/5312917.html)]( https://www.cnblogs.com/xiaohuochai/p/5312917.html )