## margin

[mdn](https://developer.mozilla.org/zh-CN/docs/Web/CSS/margin)

百分比计算：

>  **margin** 的百分比值参照其包含块的宽度进行计算。但这只发生在默认的 **writing-mode: horizontal-tb;** 和 **direction: ltr;** 的情况下。

- 水平：相对父元素宽度计算
- 垂直：相对父元素宽度计算

> 当书写模式变成纵向的时候，其参照将会变成包含块的高度。
>
> ```
> #demo{
>     -webkit-writing-mode: vertical-rl; /* for browsers of webkit engine */
>     writing-mode: tb-rl; /* for ie */
> }
> ```

margin正值：

- 元素在文档流中所占位置扩大
- 同一个bfc中的相邻元素margin会重叠

margin负值：

- 减少元素在文档流中所占的大小。反映在界面中就是偏移，如左边距-20px，则元素往左偏20px。
- 如果元素本身没有定义宽度，则它会在已有宽度（）的基础上往margin负值的方向拓展，即会变得更宽。
- 负值百分比数值的计算也是相对于父元素的宽度

margin传递问题：

> 外边距合并指的是，当两个垂直外边距相遇时，它们将形成一个外边距。
> 合并后的外边距的高度等于两个发生合并的外边距的高度中的较大者。

[https://www.w3school.com.cn/css/css_margin_collapsing.asp](https://www.w3school.com.cn/css/css_margin_collapsing.asp)

- *水平margin不会合并*

- *注释：只有普通文档流中块框的垂直外边距才会发生外边距合并。行内框、浮动框或绝对定位之间的外边距不会合并。*

margin折叠的产生有几个条件：

- 这些margin都处于普通流中，并在同一个BFC中；
- 这些margin没有被非空内容、padding、border 或 clear 分隔开；
- 这些margin在垂直方向上是毗邻的，包括以下几种情况：
  1、一个box的top margin与第一个子box的top margin
  2、一个box的bottom margin与最后一个子box的bottom margin，但须在该box的height 为auto的情况下
  3、一个box的bottom margin与紧接着的下一个box的top margin
  4、一个box的top margin与其自身的bottom margin，但须满足没创建BFC、零min-height、零或者“auto”的height、没有普通流的子box

垂直方向上毗邻的box不会发生折叠的情况：

- 根元素的外边距不会参与折叠
- 一个有clearance的box的上下margin毗邻，它会与紧接着的下一个box发生margin折叠，但折叠后的margin不会再与它们父box的bottom margin折叠

折叠边距的计算

当两个margin都是正值的时候，取两者的最大值；当 margin 都是负值的时候，取的是其中绝对值较大的，然后，从 0 位置，负向位移；**当有正有负的时候，先取出负 margin 中绝对值中最大的，然后，和正 margin 值中最大的 margin 相加。**但必须注意，所有毗邻的margin要一起参与运算，不能分步进行。