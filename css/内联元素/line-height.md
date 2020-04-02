# line-height

> 纯内联元素，其高度完全由line-height决定。padding,border对于可视高度没有影响。
>
> 即使fontsize设为0，高度依然存在。而lineheight为0，字虽然显示，但高度为0.

### 半行距

传统印刷中，行距是上下两行文字间预留区域。adobe design中，行距加在文字上方。而在css中，行距分散在文字的上方和下方，为半行距，高度为完整行距高度的一半。

计算：

1em=font-size 大小，一个em-box高是一个em

- 行距=`line-height` - `font-size` 
- 通常文字内容区高度比em高（宋体是一致的）
- 半行距，行距/2，上边距向下取整，下边距向上取整。



### line-height问题

```html
<div class="demo2">
        <img src="https://www.baidu.com/img/bd_logo1.png?where=super" alt="" height="100">
    </div>
    .demo2{
            line-height: 50px;

        }
    
```

在chrome下，img高度100，demo2高度119。

而此时行高设置的是50px。

原因：TODO

### 多行文本居中

```html
    <style>
        .box{
            width:200px;
            border: 1px dashed;
            line-height: 400px;
        }
        .box p{
            display: inline-block;
            line-height: 20px;
            margin:0 20px;
            vertical-align: middle;
        }

    </style>
        <div class="box">
        <p>测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试</p>
    </div>
```

### line-height取值和计算

> line-height默认值是normal，该值在不同字体下的行高是不一致的。
>
> 因此需要对line-height的默认值进行重置。

1. 数值。如line-height:1.5。最终结果是1.5*fontsize.
2. 百分比值。最终结果是150%*fontsize.
3. 长度值。如1.5em。最终结果是1.5*fontsize。

差异：

如果使用数值，则之后的所有子元素都继承这个值。如果是百分比和长度值，则所有的子元素继承的都是计算后的值。

如果子元素的字体大小发生变化，则使用数值的字line-height也会变大。而使用百分比和长度值的就不会变了，是父元素计算后的值。

当然百分比和长度值也可以实现数值的类似效果，就是使用通配符。

```
*{line-height:150%}
*{line-height:1.5em}
```

但是，line-height设置数值是不会影响到替换元素的。而通配符会重置这些替换元素的默认line-height。这也是我们需要的，因此使用*也是合理的。

也可以这也

```
body{line-height:1.5}
input,button{line-height:inherit}
```

如果使用20px/14px=1.4285714作为行高，应该向上取整。

因为chrome中，如果向下取整为1.42857，则在浏览器中显示依然是19px。

需要设置为1.42858



#### 大值特性

行框盒子的高度由最高的那个内联盒子决定。

```
.box2{
            line-height: 96px;
        }
        .box2 > span {
            /* display: block; */
            line-height: 30px;
        }
```

box2的高度并不是30px。span的高度是30px，但是在chrome的检查模式并看不出来，实际上span的高度已经占了30px。

因为span创建了内联盒子，前面有一个幽灵节点。

幽灵节点的行高还是96px。因此该行框盒高度是最大的96px。

span设置block，不创建内联盒，可以生效子元素行高。box2高度由span决定。

span设置inline-block，span的行高变成30px，但box2高度还是96px。