最简单的理由：因为宽度计算默认涉及包含块（可粗略理解为父级元素），而高度计算默认涉及内部元素。

要居中，必须相对父级计算，而非内部元素。

### 宽度计算
默认的宽度规则是“适应于父级”规则。（当然有一些计算模式会让元素适应于内部元素，超出话题范围，暂且不谈。）

W3 CSS 2.1 第十章里为常规流替换和非替换块级元素定义了这个算式：

> margin-left + border-left-width + padding-left + width + padding-right + border-right-width + margin-right = width of containing block

同时为几项auto设置了额外的算法：

> If there is exactly one value specified as 'auto', its used value follows from the equality.
If 'width' is set to 'auto', any other 'auto' values become '0' and 'width' follows from the resulting equality.
If both 'margin-left' and 'margin-right' are 'auto', their used values are equal. This horizontally centers the element with respect to the edges of the containing block.

这就是auto可以水平居中的原因。

对于绝对定位元素，有以下算式：

> left + margin-left + border-left-width + padding-left + width + padding-right + border-right-width + margin-right + right = width of containing block

加入了left和right，可以用类似的方式达到水平居中。

### 高度计算
默认行为的高度计算则是一系列“撑高”规则，而非“适应于父级”规则。

常规流的非替换元素高度计算规则我之前已经引用过。

对于绝对定位元素，有以下算式：

> top + margin-top + border-top-width + padding-top + height + padding-bottom + border-bottom-width + margin-bottom + bottom = height of containing block

因此margin-top: auto; margin-bottom: auto;配合一系列设置，可以让绝对定位元素垂直居中：

http://jsfiddle.net/tvkkk62v/

当然这个方案有一些限制，优点和缺点我有一篇博文“整理：子容器垂直居中于父容器的方案”综合讲过，不再复述了。

为何这么设计，我有一个粗略的想法，待以后发展一下：

水平居中支持门槛低是因为大部分文本都是lr-tb顺序，网页设计倾向于从上往下溢出而非从左往右溢出。
