#### inline元素的padding

```css
.box{
            padding:100px;
            /* background: #ccc; */
            border: 2px solid royalblue;
            display: inline;
        }
```

如果不设border或背景色，则表现起来是元素只在水平方向产生了padding。这是错误的。

因为内联元素没有clientHeight和clientWidth（可视宽高为0），垂直方向的行为完全由line-height和vertical-align影响，视觉上并没有改变上一行和下一行内容的间距。

结果是：元素的原本布局没有影响，但是在垂直方向上产生了重叠。

*重叠效果*

如relative定位，box-shadow，outline都会出现重叠的效果，而不影响其他元素的布局。

这种重叠分两类：

1. 纯视觉重叠，不影响外部尺寸
   - boxshadow，outline
2. 影响外部尺寸
   - inline的padding层