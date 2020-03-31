#### marginbox

表示元素外部尺寸，不仅包含padding和border还包含margin。可以理解为元素占据的空间尺寸，可以为负值。

#### margin与元素内部尺寸

当元素是充分利用可用空间的时候，margin才可以改变元素的可视尺寸（水平【由writing-mode决定】）；如果元素设置了width或出于保持包裹性时，则对尺寸无影响。

#### margin 百分比计算规则

基于父元素宽度计算。需要注意margin合并。

### margin合并

#### margin合并的三个场景

1. 父子元素（第一个的margintop/最后一个的marginbottom）合并

   > 阻止合并：
   >
   > 1.父元素设bfc
   >
   > 2.父元素设置垂直方向border/padding
   >
   > 3.父元素和第一个/最后一个子元素之间添加内联元素，阻止合并

2. 兄弟元素（前一个的marginbottom和后一个的margintop）

3. 空块级元素

   > 阻止空块元素合并
   >
   > 1.设置垂直padding/border
   >
   > 2.加内联元素（直接空格无效）
   >
   > 3.设置height/min-height

#### margin合并的规则

1. 正正取最大
2. 正负相加
3. 负负取最负

### margin：auto

`div`这样的块元素本身有充分利用可用空间的属性，即使不设宽度，也会自动撑满剩余宽度。

或者像这样

```
div{ position:absolute;left:0;right:0 }
```

而一旦设置了宽度，则自动填充特性被覆盖。无法填充可用宽度。

而margin:auto就是用来填充这个剩余可用宽度的。

#### 填充规则：

1. 一侧定值，一侧auto，则auto为剩余空间大小
2. 两侧auto，则平分剩余空间

需要注意的是，margin的默认值是0。

*右对齐效果*

````
.child{
	width:100px;
	margin-left:auto;
}
````

此时auto会撑满剩余可用宽度，而marginright是0，就实现了右对齐。

#### 关于垂直居中

> margin:auto 计算有一个前提条件，**只有元素有对应方向的自动填充特性时**，才会触发计算。
>
> div垂直方向如果不设置高度，是无法自动填满父元素高度的。因此无法用auto实现垂直居中。

1. 改变writing-mode为vertial-lr，此时只能设置垂直居中，无法水平居中

2. 格式化宽高

   ```css
   div{
   position:absolute;
   left:0;
   right:0;
   top:0;
   bottom:0;
   margin:auto;
   }
   ```

   此时元素的宽高都有了自动填充的特性，因此auto可以计算，实现了垂直水平居中。

兼容性：*ie8及以上*

如果元素尺寸比包裹元素大，则水平方向计算后的负值会作为0处理，不会水平居中。垂直方向负值仍旧保留，因此垂直居中。



### margin无效

1. display：inline的非替换元素的垂直margin无效。对于内联替换元素，如img，垂直margin有效，且无margin合并的问题。

2. tr，td元素或display：table-cell，table-row的元素margin无效。但计算值为table-caption，table，inline-table就没问题。可以通过margin控制外边距，:first-letter伪元素也可以解析margin。

3. margin合并时，改变margin可能无效果。

4. 绝对定位元素非定位方位的margin无效。如只设置了left和top，则再right和bottom设置margin就无效。

   > 无效原因：绝对定位元素脱离文档流独立渲染，margin无法影响兄弟元素定位。

5. 定高容器的子元素的margin-bottom，或定宽元素的子元素的margin-right。

   原因：若要使元素改变*自身位置*，必须是和当前元素定位方向一样的margin才可以（与上面绝对定位类似），否则margin只能影响后面的元素或父元素。

   一个普通元素，定位方向是左侧和上方，只有marginleft和margintop可以影响元素定位。但如果通过一些属性改变定位方向，如float：right，或绝对定位right定位，则margin-right可以影响元素自身定位，而marginleft只能影响兄弟元素。

6. ```
   div.box
     img src='m.i=jpg'
     p 内容
   
   .box > img{ float:left,width:256px}
   .box > p { overflow:hidden,margin-left:200px}
   ```

   img和p构成两个bfc，bfc的区域不会与floatbox重叠，每个盒子marginbox左边与包含款的borderbox左边接触。

   因此margin0-left左边的空间被img占了，margin无法触及到容器最左边，无法触发位移。

7. 内联特性导致margin失效。

