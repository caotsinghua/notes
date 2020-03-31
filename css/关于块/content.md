### 替换元素

根据元素的外在盒子将元素分为内联元素和块级元素，根据是否有可替换元素，把元素分为替换元素和非替换元素。

替换元素指内容可以被替换，如图片的src改变，图片就会被替换。

特性：

1. 内容外观不受页面css影响
2. 有自己的尺寸。不设尺寸情况下各浏览器对不同的替换元素有自己的默认尺寸。
3. 在很多css属性上有自己的表现规则。如vertical-align的基线对齐，不是字符x的下边缘，是元素的下边缘。

**尺寸计算规则**

替换元素的display是inline，block，inline-block，其尺寸计算规则都一样。（*这也是 img等替换元素标签设置了display:block但不能撑满container的原因*）

优先级是：css尺寸>html上设置的尺寸>内容本身的尺寸（如图片原本大小）

在哪一层设置了尺寸，会覆盖里面优先级低的设置。

*需要注意的是img在不同浏览器下，没有src时，本身宽高不是300 x150*

chrome：0x0，ie：28x30，ff：0x22

为了占位和异步加载图片，可以这样

```
<img/>
img{visibility:hidden}
img[src]{visibility:visible}
```

src=""在很多浏览器下依然会有请求，因此初始img不带src属性，不会有任何请求。

由于firefox在img不设src时表现为一个普通内联元素，而不是替换元素，因此需要重置属性。(ff71表现为0x0)

```
img{display:inline-block}
```

### 无法改变替换元素内容的固有尺寸

设置css等改变元素尺寸修改的是contentbox的尺寸，而不是替换元素的本身尺寸。

如

```
.box::before{
            content: url('https://www.baidu.com/img/bd_logo1.png?where=super');
            display: block;
            width:200px;
            height: 200px;
        }
```

#### 替换元素的src属性：img去除src

```
<img/>
img{
display:block;
border:1px solid;
}
```

火狐71和chrome一致，0x0，ie为28x30.

当img设置了alt时，且不为空时。

chrome表现为块级元素并占满container。并且内部有个小图片+alt的文字。如果alt为空，则没有小图片，还是0x0。

ie不设值src时，会默认有个小图片作为替换元素，因此是28x32.高版本ie对该替换内容做了透明处理。

*img元素如果有src时，before和after伪元素失效；没有src时就是一个普通非替换元素*

> ie不支持的原因是去除src和alt时，ie会产生一个默认的替换元素。

基于此可以实现一种按需加载。

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
    <style>
        .img-wrap {
            border: 1px solid;
        }

        .img {
            display: inline-block;
            width: 200px;
            height: 200px;
            position: relative;
            border: 1px solid blue;
            overflow: hidden;

        }
        img:not([src]){
            visibility: hidden;
        }
        .img::after{
            content:"";
            display: block;
            background:rgba(42, 165, 202, 0.267);
            position: absolute;
            left: 0;
            right: 0;
            top: 0;
            bottom: 0;
            z-index: 1;
            visibility: visible;
        }
        .img::before {
            content: attr(alt);
            display: block;
            background: rgba(0, 0, 0, 0.4);
            position: absolute;
            bottom: 0;
            left: 0;
            right: 0;
            transition: transform .3s;
            transform: translateY(100%);
            z-index: 2;
            visibility: visible;
        }

        .img:hover::before {
            transform: translateY(0);
        }
    </style>
</head>

<body>
    <div class="img-wrap">
        <img id="img" class="img" alt="这是一张图片">
    </div>
    <script>
        const img = document.querySelector("#img")
        img.onclick = function () {
            if (this.getAttribute("src")) {
                this.removeAttribute("src")
            } else {
                this.setAttribute("src", "https://www.baidu.com/img/bd_logo1.png?where=super")
            }
            console.log(this.src, this.getAttribute("src"))
        }
    </script>
</body>

</html>

```



### 替换元素的content属性

chrome下所有元素均可设置content属性，而其他浏览器只有after，before伪元素可以设置



> 当设置img的src后，再设置content可以覆盖原本的图片内容。但是复制图片url时还是标签上的src。
>
> 只有chrome支持。
>
> ie设置content无法替换。
>
> firefox的img的content无法覆盖src其他标签可以
>
> 缺点：设置content的url时，会重新请求一次资源

*chrome下：非after，before伪元素设置content为url图片时，该元素表现和img一样，属性设置也支持。伪元素设置时就相当于一个替换元素的内容，无法设置宽高等。但可以通过transform来缩小放大等*

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
    <style>
        .img1{
            width:200px;
            height: 200px;
            object-fit: contain;
            border: 1px solid;
            background: rgba(0, 0, 0, 0.1);
        }
        .img1:hover{
            content: url('https://www.baidu.com/img/bd_logo1.png?where=super');
        }
    </style>
</head>
<body>
    <img class="img1" src="https://dss0.baidu.com/73F1bjeh1BF3odCf/it/u=4055080200,3603986021&fm=85&s=C13C2D72ABF0408A1AFD74CA030070B1" alt="">
    <div class="img1">
        普通标签
    </div>
</body>
</html>

```

