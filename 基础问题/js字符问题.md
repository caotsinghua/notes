## 半角字符和全角字符

> 全角：是一种电脑字符，是指一个全角字符占用两个标准字符(或两个半角字符)的位置。全角占两个字节。
>
> 汉字字符和规定了全角的英文字符及国标GB2312-80中的图形符号和[特殊字符](http://www.php.cn/wiki/88.html)都是全角字符。在全角中，字母和数字等与汉字一样占据着等宽的位置。
>
> 半角：是指一个字符占用一个标准的字符位置。半角占一个字节。
>
> 半角就是 ASCII 方式的字符，在没有汉字输入法起作用的时候，输入的字母、数字和字符都是半角的。
>
> 每个半角字符只占用一字节的空间(一字节有8位，共256个编码空间)。汉语、日语、及朝鲜文等象形字语言的字库量远大于256个编码空间，所以改用两个字节来储存。同时，由于中日韩等象形文字的书写习惯，如果统一使用全角字符的话，排列起来也显得整齐。
>
> 为了排列整齐，英文和[其它](http://www.php.cn/java/java-alibaba-qita.html)拉丁文的字符和标点也提供了全角格式。
>
> ### 半角
>
> 每个半角字符占用一字节空间(一字节有8位)，共256个编码空间。
> 半角正则表达式：/[\x00-\xff]/g
>
> ### 全角
>
> 每个全角字符占用两字节空间。
> 全角正则表达式：/[^\x00-\xff]/g

```
str="中文;；ａ"  

alert(str.match(/[\u0000-\u00ff]/g))   //半角  

alert(str.match(/[\u4e00-\u9fa5]/g))   //中文  

alert(str.match(/[\u00ff-\uffff]/g))   //全角
```

 **js对全角与半角的相互转化** 

> a.全角空格为12288，半角空格为32
>
> b.其他字符半角(33-126)与全角(65281-65374)的对应关系是：均相差65248

```js
function ToDBC(txtstring) { 

  var tmp = ""; 

  for(var i=0;i<txtstring.length;i++{ 

    if(txtstring.charCodeAt(i)==32){ 

      tmp= tmp+ String.fromCharCode(12288); 

    } 

    if(txtstring.charCodeAt(i)<127){ 

      tmp=tmp+String.fromCharCode(txtstring.charCodeAt(i)+65248); 

    } 

  } 

  return tmp; 

}
```

