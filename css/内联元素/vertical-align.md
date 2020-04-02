# vertical-align

### 属性值：

- 线类：baseline，top，middle，bottom
- 文本类：text-top，text-bottom
- 上标下标类：sub，super
- 数值百分比类：20px，2em，20%

### 数值类：

```
.line{
            font-size: 20px;
            border: 1px solid;
            margin: 0;
            /* height: 20px; */
        }
        .line span{
            /* vertical-align: 20px; */
            vertical-align: 100px;
        }
        <p class="line">
        <span>测试和x</span>
        <!-- <img src="https://www.baidu.com/img/bd_logo1.png?where=super" alt="" style="border: 1px solid;"> -->
    </p>
```

chrome表现：

1. line不设高度，span设置verticalline（正负值），会使得line高度增大。因为span的本身高度增加了，可以选中文字查看选择区域。
2. 如果line设置了高度，则verticalalign不影响父元素高度

### 作用前提

应用于内联元素，display：table-cell的元素。

即inline，inline-block，inline-table，table-cell

*注意：float，绝对定位会使元素块状*

垂直居中-例子1：

```
        .container{
            height: 200px;
            border: 1px dashed;
            line-height: 200px; // 设置行高才垂直居中
        } 

        .img{
            height: 100px;
            vertical-align: middle;
            border: 1px solid;
        }
```

如果不设container的行高，则图片无法垂直居中，因为幽灵节点的高度太小，无法作为参照。

当设置了line-height后即可居中。

例子2：

使用table-cell

```
.container{
            display: table-cell;
            height: 200px;
            border: 1px dashed;
            vertical-align: middle;
        }
        .img{
            height: 100px;
            /* vertical-align: middle; */
            border: 1px solid;
        }
```

这时可以垂直居中。

因为table-cell设置verticalalign作用的是table-cell自身，即使子元素是块级，也可以居中。



### 与行高的关系

1. **包含块高度!=行高的问题**

```html
    <style>
        .box{
            line-height: 32px;
            border: 1px dashed;
        }
        .box span{
            vertical-align: baseline;
            font-size: 24px;
        }
    </style>
    <div class="box">
        x<span>文字x</span>x
    </div>
```

box的高度不是32px，而是更多一点。

幽灵节点用x字符代替。

此时x的行高和span的行高都是32px，但是由于默认基线对齐，x字体偏小，因此要向下移动一点，从而导致整个行框盒的行高增加了，并导致box的高度增加。

*该现象违背了line-height大值效应的原因是字体大小不一致。字体大小相同时，基线一致，不发生偏移，高度就是行高。*

解决方案：

1.  fontsize一致
2. 使用不同的对齐方式，如vertical-align:top



2. **文字和图片同级，图片下方有间隙的问题**

```css
         .img-box{
            width: 200px;
            box-sizing: border-box;
            border: 1px dashed;
            font-size: 20px;
            /* line-height: 0; */
        }
        .img-box span{
            background: #ccc;
        }
        .img-box img{
            width:100px;
            height: 100px;
            border: 1px dashed;
            /* vertical-align: bottom; */
        }
```

如上代码所示。

结果是图片下方有一些间隙。

原因在于，span的行高较大，当图片进行基线对齐的时候，会向上偏移一些，

相当于vertical-align:5px 结果导致父元素的高度被撑开了一些。

解决方案：

1. 不用基线对齐，用top,middle,bottom
2. 容器line-height足够小，只要半行间距到达x的基线位置，就不会发生偏移
3. 容器的fontsize足够小，前提是line-height与fontsize有关，如line-height:1.5。否则会因为字体变小，行高不变导致半行间距更大。
4. 图片块状化，去除幽灵节点，lineheight，vertical-align

3. **内联特性导致的margin失效情况**

   ```html
   <style>
           .box {
               border: 1px dashed;
               width: 300px;
               margin:auto;
               height: 500px;
               line-height: 0;
               font-size: 0;
               margin-top: 200px;
               /* font-size: 0; */
           }
           .box > img{
               width: 100px;
               height: 100px;
               border: 1px solid;
               margin-top: -1000px;
           }
       </style>
       <div class="box">
           x<img src="https://www.baidu.com/img/bd_logo1.png?where=super" alt="">
       </div>
   ```

   img偏移不到box的外面去。

   原因和上面一样，非主动触发位移的内联元素不可能跑到容器外面，由于img要与幽灵节点进行基线对齐，无法脱离包含块。

   如果将box的fontsize设为0，img也只能偏移到容器顶部0px的位置。因为幽灵元素始终存在。

   解决方法：将img块状化，去除幽灵节点。

   

#### 多个img和inline-block，text-align：justify

```
<div class="box">
        <img src="https://www.baidu.com/img/bd_logo1.png?where=super" alt="">
        x<img src="https://www.baidu.com/img/bd_logo1.png?where=super" alt="">
        x<img src="https://www.baidu.com/img/bd_logo1.png?where=super" alt="">
        <img src="https://www.baidu.com/img/bd_logo1.png?where=super" alt="">
        <img src="https://www.baidu.com/img/bd_logo1.png?where=super" alt="">
        x<i class="justify-fix">1</i>
        x<i class="justify-fix">1</i>
        x<i class="justify-fix">1</i>
    </div>
     <style>
        .box{
            text-align: justify;
            border: 1px dashed;
            max-width: 500px;
            /* font-size: 0; */
            line-height: 0;
            /* font-size: 0; */
        }
        .box img{
            width: 96px;
            border: 1px solid;
        }
        .justify-fix{
            display: inline-block;
            width: 96px;
            outline: 1px dashed red;
            /* height: 10px; */
        }
    </style>
```



原因在于：inline-block的基线，如果没有内联元素，就是其margin下边缘，有内联元素，就是该内联元素的基线。

设置line-height：0，可以使图片间下无间隙，是因为图片的基线是margin下边缘，与行高为0的幽灵节点的基线对齐后，下方就没有撑开空间了。

当i元素没有添加内联元素时，其底边与幽灵节点的基线对齐，用x代替，而此时由于x的line-height为0，x整体向上偏移了半个行距，因此x的对齐点就是x字符的内容区域的垂直中心位置，由于文字下沉，因此垂直中心会在x字符上面一点。而justify-fix元素的底边与x的底边对齐，因此上方空出1个x不到的距离。

解决方法：

1. 改变i的基线，即在其中添加内联字符，如nbsp，则其基线和x一致，由于line-height为0，因此i的位置和基线重合。
2. fontsize=0，当字体足够小时，基线与中线会重合。
3. lineheight=0，同时改变verticalalign不为baseline

### 线性属性值

#### inline-block与baseline

> 一个inline-block元素，如果里面没有内联元素，或overflow不是visible，则该元素的基线是margin底边缘。否则其基线是元素里最后一行内联元素的基线

```
<span class="daseline"></span>
<span class="daseline">xxxxx-baseline</span>
<style>
        .daseline{
            display: inline-block;
            width: 150px;
            height: 150px;
            border: 1px solid;
            background-color: cadetblue;
        }
    </style>
```

那么第二个span的文字基线和第一个span的底边缘进行基线对齐，导致第二个span下移的情况。

#### 作用

1. 实现小图标对齐文字

   - 图标高度和行高一致
   - 图标标签内有字符，图标标签为inline-block
   - 图标标签不使用overflow:hidden。保证基线时字符，同时让字符不可见。

   此时，图标与文字对齐。

####  top/bottom

top：垂直上边缘对齐

- 内联元素：元素顶部和当前行框盒子的顶部对齐。即和这一行位置最高的内联元素顶部对齐（注意幽灵节点）
- table-cell元素：元素顶padding边缘和表格行的顶部对齐。即td和tr的上边缘对齐。

> 注意：内联元素上下边对齐的边缘是当前行框盒子的上下边，而不是块级容器的上下边。



#### middle

近似垂直居中

- 内联元素：元素的垂直中心点和行框盒子基线往上1/2x-height对齐
- table-cell：单元格填充盒子相对外面的表格行居中对齐

内联元素中，基本所有字体中，x的位置都是偏下一点，字号越大越明显，因此vertical-align：middle实现的都是近似垂直居中。

实例： https://demo.cssworld.cn/5/3-8.php 

因此，想要实现真正的垂直居中，需要把x的中心点移到容器的垂直中心，设置fontsize为0，字符x就缩小成一个点，根据line-height上下等分原则，中心点就处在容器的中间了。

### 文本类属性值

- text-top:盒子的顶部和父级内容区域的顶部对齐
- text-bottom：盒子的底部和父级内容区域的底部对齐

内容区域是指：firefox/ie浏览器文本选中的背景区域/默认状态下文本背景色区域

父级内容区域：即在父级元素当前fontsize和fontfamily下应有的内容区域大小。

以texttop为例：假设元素后有一个和父元素fontsize，fontfamily一样的文字内容，则texttop表示元素和这个文字内容区域的上边缘对齐。

见：https://demo.cssworld.cn/5/3-9.php

### 上下标

sub：对应<sub>标签 下标

super:对应<sup>标签 上标

标签对应的text-align就是sub或者super。且会改变fontszie为smaller。



