table的宽度由js计算得出

```js
handleResize () {
                    //let tableWidth = parseInt(getStyle(this.$el, 'width')) - 1;
                let tableWidth = this.$el.offsetWidth - 1;
```

问题：

当table元素较多，形成竖向滚动条，滚动条在原本宽度的基础上增加了滚动条的宽，导致出现html的横向滚动条。

> 竖向滚动条不是html形成，是其子元素设置定高和overflow：auto后形成的。

原因：

table的宽度是定死的，当滚动条出现的时候不会自适应宽度，导致出现横向滚动条



解决：

1.产生滚动条的元素设置

```
overflow:auto
overflow:overlay
```

使滚动条不占据宽度。只在chrome有效

2.在table增加父元素，并设99%,使table宽度根据父级宽度来计算

