### 动态导入script需要注意的地方

#### 导入的几种方法

1. 创建script标签

   示例：

   ```
   function importJs(src){
   	const script=document.createElement('script')
   	script.src=src
   	// script.sync 默认是true
   	document.body.appendChild(script)
   }
   ```

   注意点:

   - script动态创建时，sync默认为true；直接写在页面上的script标签默认为 false;

     这就需要注意sync为true时，先加载完脚本即直接执行，由于脚本加载速度差异，无法保证脚本的执行顺序。而设置defer=true时，会在html解析后domcontentloaded前执行脚本，但是同样无法保证顺序。

     ??设置sync=false后，脚本顺序执行，但是会在页面中的脚本执行完后再执行??

     > 【在 现实 当中， 延迟 脚本 并不 一定 会 按照 顺序 执行！！！， 也不 一定 会在 DOMContentLoaded 事件 触发 前 执行， 因此 最好 只 包含 一个 延迟 脚本。
     > 泽卡斯(Zakas. Nicholas C.). JavaScript高级程序设计(第3版) (图灵程序设计丛书) (Kindle 位置 639-641). 人民邮电出版社. Kindle 版本.async的js在下载完后会立即执行（因此脚本在代码中的顺序并不是脚本所执行的先后顺序，有可能后面出现的脚本先执行）。】[关于async和defer](http://ued.ctrip.com/?p=3121)

2. document.write

   - 在DOMContentLoaded事件完成前，使用document.write，会把写入部分添加到当前位置

     ```
     <body>
     	<div>测试open</div>
     	<script>
     		document.addEventListener('DOMContentLoaded', function() {
     			console.log("_____DOMContentLoaded")
     
     		})
     
     		function writeJs() {
     			var str = `<script src="./test.js"><\/script>`
     			document.write(str)
     		}
     		writeJs()
     		console.log("jsok")
     	</script>
     	<script></script>
     </body>
     ```

     结果

     ![](D:\mine-codes\notes\images\do1.png)

   - 在DOMContentLoaded事件完成后或domcontentloaded和load回调函数中调用document.write，会清空文档再重新写入。

     ```
     var str = `<h1>11</h1><script src="./test.js"><\/script>`
     			document.write(str)
     ```

     ![](D:\mine-codes\notes\images\RTX截图未命名.png)

     ```
     function writeJs() {
     			var str = `<script src="./test.js"><\/script>`
     			document.write(str)
     		}
     ```

     ![](D:\mine-codes\notes\images\RTX截图未命名1.png)

     > 　在以上代码中，原来的文档内容并没有被清空，这是因为当前文档流是由浏览器所创建，并且document.wirte()函数身处其中，也就是执行此函数的时候文档流并没有被关闭，这个时候不会调用document.open()函数创建新文档流，所以也就不会被覆盖了。

   - **异步**引用JavaScript时（例如test.js），test.js中必须先运行document.open()清空文档，然后才能运行document.write()，如果不先运行document.open()，直接运行document.write()，则无效，忽略该调用.

     firefox:来自一个外部异步载入的脚本的 document.write() 调用被忽略。

   - .

     ```
     !DOCTYPE html>   
     <html>   
     <head>   
       <meta charset=" utf-8">     
       <title>Document</title>   
       <script type="text/javascript">
         document.close(); 
         document.write("重温 JavaScript");
       </script> 
     </head> 
     <body> 
       <div>Hello JavaScript</div> 
     </body> 
     </html>
     ```

     > 上面使用document.close()关闭文档流了，为什么还是不能够覆盖原来的内容的，很遗憾，文档流是由浏览器创建，无权限手动关闭，document.close()函数只能够关闭由document.open()函数创建的文档流

3. jquery：$.getScript

4. 不要动态的改变src，不会执行脚本(ie9+)

   ```
   <script src="./test.js" id="sc"></script>
       <script>
           document.getElementById("sc").src="./test2.js"
       </script>
   ```

   结果：

   - Chrome: 什么事都没发生（没有请求url2）
   - Firefox: 什么事都没发生（没有请求url2）
   - IE9：什么事都没发生（请求url2但不执行url2的脚本）
   - IE(6,7,8): I am dynamic(请求并执行了url2的脚本)

links

- [【JavaScript】document.write()的用法和清空的原因](https://blog.csdn.net/u013451157/article/details/78699253)

- [携程:script的defer和async](http://ued.ctrip.com/?p=3121)

- [动态修改script的src存在的问题](https://www.jianshu.com/p/c4333502c2e6)

  

