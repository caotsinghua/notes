## margin

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
- 

