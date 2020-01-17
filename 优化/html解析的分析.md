### 第一个例子

参考资源：

1. [浏览器的工作原理]( https://www.html5rocks.com/zh/tutorials/internals/howbrowserswork/ )
2. [ https://zhuanlan.zhihu.com/p/47407398 ]( https://zhuanlan.zhihu.com/p/47407398 )
3. [ https://developers.google.com/web/updates/2018/09/inside-browser-part3 ]( https://developers.google.com/web/updates/2018/09/inside-browser-part3 )

界面资源：

无内联script，1个加载3s的css资源，一个立即加载的common.js资源，一个加载5s的js资源。

>#### 预解析
>
>WebKit 和 Firefox 都进行了这项优化。在执行脚本时，其他线程会解析文档的其余部分，找出并加载需要通过网络加载的其他资源。通过这种方式，资源可以在并行连接上加载，从而提高总体速度。请注意，预解析器不会修改 DOM 树，而是将这项工作交由主解析器处理；预解析器只会解析外部资源（例如外部脚本、样式表和图片）的引用。

代码如下：

以chrome为例，解释该html的解析过程。

```html
<!DOCTYPE html>
界面解析html结构，发出三个资源请求，1css，2js【预解析。在ie中还是按顺序解析html后发起请求】
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
    parse html到这里，执行这段js，同时阻塞html
    <script>
        console.log("head js")
        document.addEventListener('DOMContentLoaded', () => {
            console.log('DOMContentLoaded')
        })
    </script>
    这里的css资源未加载完，不阻塞html的解析，但阻塞渲染
    <link rel="stylesheet" href="./test-timeout-3.css">
    <!-- <link rel="stylesheet" href="./test-timeout-5.css"> -->

</head>

<body>
    <h1 class="title">测试</h1>
    <div class="box"></div>
    <button>aaa</button>【下面遇到common.js阻塞parse，parse点A】
    3s内common.js早就加载完，由于css未加载完，不执行，并且阻塞html解析
    3s后css加载完成，执行parsestylesheet，解析样式
     css解析完成，立即接收common.js内容，执行common.js。该js里有个长循环，阻塞6s多。阻塞html的继续解析和渲染。
    这段同步代码执行的时候，5s的js资源已经加载完毕。
    common.js执行完成，继续解析html，从刚刚阻塞的parse点A开始。
    由于common.js执行完毕了，这段解析完成后，计算样式和渲染，触发firstpaint。
    <script src="./common.js"></script>【下面遇到timeout.js，parse点B】
   渲染完毕，此时接收timeout5.js的数据（早就加载完），并且执行代码，阻塞html的解析在点B。
    timeout5.js执行完毕，继续parseHtml
    <script src="./common-timeout-5.js"></script>
	解析过程中又会重新计算样式，触发domcontentloaded

</body>

</html>

```

performance过程如下：

![图片](D:\mine-codes\notes\images\渲染分析1.png)

### 渲染时机

参考[html规范]( https://html.spec.whatwg.org/#microtask-queue )中的8.1.4.3 Processing model

>  一个event loop只要存在，就会不断执行下边的步骤：
> 1.在tasks队列中选择最老的一个task,用户代理可以选择任何task队列，如果没有可选的任务，则跳到下边的microtasks步骤。
> 2.将上边选择的task设置为[正在运行的task](https://html.spec.whatwg.org/multipage/webappapis.html#currently-running-task)。
> 3.Run: 运行被选择的task。
> 4.将event loop的[currently running task](https://html.spec.whatwg.org/multipage/webappapis.html#currently-running-task)变为null。
> 5.从task队列里移除前边运行的task。
> 6.Microtasks: 执行[microtasks任务检查点](https://html.spec.whatwg.org/multipage/webappapis.html#perform-a-microtask-checkpoint)。（也就是执行microtasks队列里的任务）
> 7.更新渲染（Update the rendering）...
> 8.如果这是一个worker event loop，但是没有任务在task队列中，并且[WorkerGlobalScope](https://html.spec.whatwg.org/multipage/workers.html#workerglobalscope)对象的closing标识为true，则销毁event loop，中止这些步骤，然后进行定义在[Web workers](https://html.spec.whatwg.org/multipage/workers.html#workers)章节的[run a worker](https://html.spec.whatwg.org/multipage/workers.html#run-a-worker)。
> 9.返回到第一步。 

所以，如果有以下代码

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
    执行js代码，阻塞一段很短的时间
    <script>
        console.log("head js")
        document.addEventListener('DOMContentLoaded', () => {
            console.log('DOMContentLoaded')
        })
    </script>
    chrome中此时css已经在加载，等待3s。这3秒内，继续html的解析
    <link rel="stylesheet" href="./test-timeout-3.css">

</head>

<body>
    <h1 class="title">测试</h1>
    解析到这里，由于common.js已经加载完成（chrome会预先解析加载）。执行代码。
    此时js代码在task queue中执行
    <script src="./common.js"></script>


    <div class="box"></div>
    <button>aaa</button>
</body>

</html>

```

