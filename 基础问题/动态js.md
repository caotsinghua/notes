ff：来自一个外部异步载入的脚本的 document.write() 调用被忽略。

通过createElement创建的 script 标签其属性async默认为true,直接写在页面上的script标签默认为 false;

 

false能保证多个script的执行顺序，true不能保证。

 

所以在动态插入多个script默认是不能保证执行顺序的！

 

如果在创建的同时指定 async 为 false， 除IE(6789)不能保证顺序，其他A级浏览器都可以！

 

另外：新建的script通过setAttribute设置async为false会失败。使用 script.async = false; 可以达到预期效果！

：opera浏览器不鸟 async ，始终能保证执行顺序

```
 function o(e) {
          var script = document.createElement('script');
            script.src = e;
            script.async=false;
            document.body.appendChild(script);
            // document.write('<script type="text/javascript" src="' + e + '"></script>');
        }
```

