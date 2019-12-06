> HTTP协议本身是无状态的。 
>
> 无状态是指Web浏览器与Web服务器之间不需要建立持久的连接，这意味着当一个客户端向服务器端发出请求，然后Web服务器返回响应（Response），连接就被关闭了，在服务器端不保留连接的有关信息。也就是说，HTTP请求只能由客户端发起，而服务器不能主动向客户端发送数据。 

cookie包含的属性

- name=value，键值对
- expires 过期时间， *date-in-GMTString-format* ，格式参见 ` [Date.toUTCString()](https://developer.mozilla.org/en-US/docs/JavaScript/Reference/Global_Objects/Date/toUTCString)  `，不设值，cookie在对话结束后过期，

- max-age=*max-age-in-seconds* 多久后过期，负数：只在会话时有效，会话结束后删除；0：立即删除。

- secure 只通过https传输

- domain  (例如 'example.com'， 'subdomain.example.com') 如果没有定义，默认为当前文档位置的路径的域名部分。与早期规范相反的是，在域名前面加 . 符将会被忽视，因为浏览器也许会拒绝设置这样的cookie。

  如果指定了一个域，那么子域也包含在内。 否则同级域名下也不能互相用cookie，如a.a.com和b.a.com不能互相用cookie。

  

-  `;path=*path*` (例如 '/', '/mydir') 如果没有定义，默认为当前文档位置的路径。只有在指定路径下才能使用该cookie。用/表示允许所有路径。 

 可以通过更新一个cookie的过期时间为0来删除一个cookie。 