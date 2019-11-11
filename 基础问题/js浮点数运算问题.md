因为电脑永远都是按照二进制进行运算的，我们输入的十进制数在转化为二进制数时，并不总如人意，意思是说有些十进制并不能用准确的二进制数表示。

十进制小树转二进制表示

我们来看下十进制小树转二进制表示的过程，小数位乘以2，取整，小数部分继续乘以2，再取整，直到小数部分为0为止，然后将取整位按顺序排列。

由此方法我们可以看到以下一些十进制小数的二进制表示。

复制代码
 十进制     二进制
 0.1        0.0001 1001 1001 1001 ...
 0.2           0.0011 0011 0011 0011 ...
 0.3        0.0100 1100 1100 1100 ...
 0.4        0.0110 0110 0110 0110 ...
 0.5        0.1
 0.6        0.1001 1001 1001 1001 ...
复制代码
我们发现有些小数的二进制表示位数是无限循环的，这就造成了浮点数精度上的问题。

比如十进制的1.1，在用二进制表示的时因为存在二进制位无限循环的表示，实际值为1.0999999999...无限接近与1.1


## 解决方案
> 浮点数转化整数运算，再转回小数


```
//求和
 function add(num1,num2){
     var r1,r2,m;
     try{
         r1 = num1.toString().split('.')[1].length;
     }catch(e){
         r1 = 0
     }
     try{
         r2 = num2.toString().split('.')[1].length;
     }catch(e){
         r2 = 0
     }
     m = Math.pow(10,Math.max(r1,r2));
     return Math.round(num1*m + num2*m)/m;
 }
 ```
 ```
 //相减
 function sub(num1,num2){
     var r1,r2,m;
    try{
        r1 = num1.toString().split('.')[1].length;
    }catch(e){
        r1 = 0;
    }
    try{
        r2 = num1.toString().split('.')[1].length;
    }catch(e){
        r2 = 0;
    }
    m = Math.pow(10,Math.max(r1,r2));
    n = (r1 >= r2) ? r1 : r2;
    return (Math.round(num1 * m - num2*m) / m).toFixed(n);
 }
 ```
```
//相乘
 function mul(num1,num2){
     var m = 0,r1,r2;
     var s1 = num1.toString();
     var s2 = num2.toString();
     try{
         m += s1.split('.')[1].length
     }catch(e){

     }
     try{}catch(e){
         m += s2.split('.')[1].length
     }catch(e){

     }
     r1 = Number(num1.toString().replace(".",""));
     r2 = Number(num2.toString().replace(".",""));
     return r1 * r2 / Math.pow(10,m);
 }
 ```
 ```
  //相除
 function accDiv(){
      var t1,t2,r1,r2;
     try{
         t1 = num1.toString().split('.')[1].length;
     }catch(e){
         t1 = 0;
     }
     try{}catch(e){
         t2 = num2.toString().split('.')[1].length;
     }catch(e){
         t2 = 0;
     }
     r1 = Number(num1.toString().replace(".",""));
     r2 = Number(num2.toString().replace(".",""));
     return (r1 / r2)*Math.pwo(10,t2-t1);
 }
 ```
