## CORS

CORS (Cross-Origin Resource Sharing 跨域资源共享) 是W3C 的一个工作草案，定义了在必须访问跨源资源时， 浏览器与服务器应该如何沟通， CORS 背后的思想， 就是使用自定义的HTTP 头部让浏览器与服务器进行沟通， 从而决定请求或相应是应该成功还是失败

``` javascript
// eg
// 在发送请求时 ，需要加一个 额外的 Origin 头部， 包含请求页面的源信息
Origin: http://www.nczonline.net
// 如果服务器认为这个请求可以接受， 就会在 Access-Control-Allow-Origin 头部中会发相同的源信息， 如果是公共资源 可以回发 “*”
Access-Control-Allow-Origin： http://www.nczonline.net 
// 如果 没有这个头部，或者 这个头部源信息不匹配， 浏览器将会驳回该请求
```





#### IE 对 CORS 的实现

微软在 IE8 中引入了 XDR（XDomainRequest）类型， 这个对象 与 XHR 类似 ， 但能实现安全可靠的跨域通信， 与XHR 有一些不同之处

* cookie 不会随请求发送， 也不会随响应返回
* 只能设置请求头部信息中的 Content-Type 字段
* 不能访问响应头部信息
* 只支持GET和POST请求

这些变化使 CSRF （Cross-Site Request Forgery,  跨站点请求伪造）和 XSS (Cross-Site Scripting 跨站点脚本) 的问题得到了缓解， 被请求的资源可以根据他认为合适的任意数据（用户代理， 来源页面 等） 来决定是否设置   Access-Control-Allow-Origin 头部， 作为请求的一部分 Origin 头部的值表示请求的来源域， 以便远程资源明确地识别XDR请求

所有 XDR 请求都是 异步执行的 ， 不能用它来创建同步请求， 请求返回之后， 会触发load 事件， 响应的数据也会保存在 responseText 属性中 

``` javascript
var xdr = new XDomainRequest()
xdr.onload = function() {
    // ...
}
xdr.onerror = function() {
    // ...
}
xdr.timeout = 1000
xdr.ontimeout = function() {
    // ...
}
xdr.open('get', 'http://www.text.com/aaa/')
xdr.send(null)
```

为了支持 post 请求 XDR 对象提供了 contentType 属性







#### 其他浏览器对 CORS 的实现

FireFox, Safari, Chorme, IOS 版 Safari 和 Android 平台中的 WebKit 都通过 XMLHttpRequest对象 实现了对 CORS的 原生支持， 要请求另一个域中的资源， 使用 标准的 XHR 对象 并在 open（） 方法中 传入绝对的URL 即可

``` javascript
var xhr = createXHR()
xhr.onreadystatechange = function() {
    if(xhr.readyState === 4) {
       // ...
    }else{
        // error
    }
}
xhr.open('get', 'http://www.abc.com/page', true)
xhr.send(null)
```

与 IE 中的 XDR 对象不同， 通过 跨域XHR 对象可以访问 status 和 statusText 属性， 而且还支持同步请求， 但是 跨域 XHR 也有一些限制

* 不能使用 setRequestHeader() 设置自定义头部
* 不能发送和接收 cookie
* 调用 getAllResponseHeaders() 方法总是返回空字符串





#### Prefilghted Requests



#### withCredentials



#### 跨浏览器的 CORS 



#### 图像 Ping



#### JSONP



#### Comet 



#### 服务器发送事件



#### Web Sockets