# 几种inline-block垂直居中方法

1. 高度1ex

   ```
           body{
               font-size: 14px;
               line-height: 20px;
           }
           .p{
               border: 1px dashed;
               height: 20px;
           }
           .p .img{
               display: inline-block;
               width: 16px;
               height: 1ex;
               background: url('https://demo.cssworld.cn/images/8/delete@2x.png') no-repeat center;
               background-size: 16px 16px;
           }
           <div class="p">
               <span>文字</span>
               x
               <div class="img"></div>
           </div>
   ```

   利用ex的高度是一个x的高度，将img的高度设为1ex，并且baseline是底边，从而使得img和x对齐，实现垂直居中。

   注意：

   由于是baseline对齐，因此该居中也是近似居中，通常来说，文字都会相对真实中间线下移一部分。因此图标和文字对齐，也会相对偏下一点点。

   另外：*由于高度是1ex，因此图标的高度也不能超过1ex，否则会显示不全*

   ![](D:\mine-codes\notes\images\微信截图_20200409100027.png)

2. 利用inline-block的基线计算对齐

   当inline-block有内联元素时，其baseline是该内联元素的baseline。

   因此，将img的高度设为和行高一致，背景图片居中，再设置::before伪类为内联，并隐藏文字，即可居中。是真正居中。

   > 同理，不设内部inline元素。将img的vertical-align设为bottom，也可以居中。

   将上面代码改成这样

   ```
   .p .img{
               display: inline-block;
               width: 16px;
               /* height: 1ex; */
               height: 20px;
               background: url('https://demo.cssworld.cn/images/8/delete@2x.png') no-repeat center;
               background-size: 16px 16px;
           }
           .p .img::before{
               content: '\3000';
               font-size: 0;
           }
   ```

   ![](D:\mine-codes\notes\images\微信截图_20200409101003.png)

3. 手动对齐

   手动设置img的vertical-align数值，进行对齐。前提是明确具体高度，高度不会变化。

   设置transform偏移。margintop偏移等。

   可以结合absolute进行无依赖布局。

4. vertical-align：middle

   这种对齐也只是近似对齐，大部分情况会出现图标偏下的情况。

   一般来说使用字体图标进行的也是字体进行对齐，影响不大。

5. vertical-align：百分比 偏移

   会影响容器的高度。