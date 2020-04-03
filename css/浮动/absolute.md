# absolute

## 特点

absolute和float很相似

1. 块状化
2. 包裹性
3. 破坏流

*absolute和float同时设置时，float无效*

## 包含块

absolute天然具有包裹性，但是和float或者其他的包裹性带来的自适应性不同，absolute的自适应最大宽度不是由父元素决定，而是由“包含块”的差异决定的。



普通元素的width:50%，相对父元素的contentbox计算，而绝对定位宽度相对于第一个position不为static的祖先计算。

1. 根元素，初始包含块，尺寸相当于浏览器可视窗口大小

2. 其他元素，如果position时relative/static，则包含块由最近的块容器祖先和的content-box边界形成。

3. 如果元素fixed，则包含块是初始包含块

4. 如果absolute，则包含块由最近的position不为static的祖先元素建立。

   如果祖先是纯inline元素：

   - 假设元素前后各生成一个宽度0的内联盒子，则这两个内联盒子的padding box外面的包围盒就是内联元素的包含块。
   - 如果该内联盒子被跨行分割了，则包含块是未定义的。

   否则，包含块由该祖先的padding-box形成。

#### 和常规元素的区别

1. 内联元素也可作为包含块所在的元素
2. 包含块所在的元素不是父级块级元素
3. 边界是padding-box不是content-box

## 内联元素作为包含块

```
<style>
        .father{
            position: relative;
            background: #ccc;
        }
        .child{
            position: absolute;
            width: 100%;
            border: 1px dashed;
            top: 0;
        }
    </style>
    <span class="father">
            cesss<span style="font-size: 30px;line-height: 50px;">这个很大</span>ssss
            <div class="child"></div>
        </span>


        <div style="background: #ccc;">
            cesss<span style="font-size: 30px;line-height: 50px;">这个很大</span>ssss
        </div>
```

以此为例，内联元素的包含块高度和内部的span元素没有关系，虽然行高和字体很大，但其高度还是由father的属性决定的。

另外，内联元素的包含块受firstline影响，但不受first-letter影响。

#### 兼容性问题

1. 如果包含块是一个空的内联元素，在ie下存在问题。

   给内联元素父级添加textalign：right，absolute元素仍然在左边。

   ie8-ie10

2. 跨行：规范对此行为未定义。

   在ff，包含块仅覆盖第一行。

   ie和chrome中包含了所有行。

   当内联元素最后一个内联盒子的右边缘比第一个内联盒子的左边缘还靠左，此时包含块高度为0，起始位置是第一个内联盒子所在位置

#### 包裹性

- 当包含块宽度不足，absolute无法撑开内部文字宽度，就会表现为最小宽度，即文字堆积的很高。可以通过white-space:nowrap解决。
- 即使包含块宽度足够，但包含块太靠近浏览器边缘，也会导致剩余空间不足。可以通过whitespace解决。



#### 无依赖

一个绝对定位元素，不设left/top/right/bottom，祖先元素全部非定位元素，则位置还在当前位置。

不使用方位的好处：

1. 相对定位特性：本质上是相对定位，只是不占据流尺寸空间。