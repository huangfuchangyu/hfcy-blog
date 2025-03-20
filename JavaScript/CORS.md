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

无论同源请求还是跨源请求都是用相同的接口，因此 对于本地资源 最好使用相对URL 在访问远程资源时在使用绝对URL， 这样能消除歧义， 避免出现访问头部或本地cookie 等信息问题



#### Prefilghted Requests

CORS 通过一种叫做 Preflighted Request 的透明服务器验证机制支持开发人员使用自定义头部， GET 或 POST 之外的方法， 以及不同类型的主题内容， 在使用下列高级选项来发送请求时， 就会想服务器发送一个Preflight 请求， 这种请求使用 OPTIONS 方法， 发送下列头部

* Origin: 与简单的请求相同

* Access-Control-Request-Method: 请求自身使用的方法
* Access-Control-Request-Headers: (可选) 自定义头部信息， 多个头部以逗号分隔

服务器通过在响应中发送如下头部与浏览器进行沟通

* Access-Control-Allow-Origin: 与简单请求相同
* Access-Control-Allow-Mehhods: 允许的方法， 多个方法一逗号分隔
* Access-Control-Allow-Headers:  允许的头部， 多个头部以逗号分隔
* Access-Control-Max-Age: 应该将这个Preflight 请求缓存多长时间（秒）

Preflight  请求结束后， 结果将按照响应中指定的时间缓存起来 ， 而为此付出的代价只是第一次发送这种请求是会多一次HTTP 请求

支持Preflight  请求的浏览器 FIreFox 3.5+ , safari 4+, chrome , IE 10+

#### withCredentials(带凭据的请求)

默认情况下， 跨源请求不提供凭据（cookie， http 认证，及客户端 SSL证明 等 ）通过对将此属性设为 true ， 可以指定某个请求应该发送凭据， 如果服务器接受带凭据的请求 会用下面的HTTP 头部来响应

``` javascript
Access-Control-Allow-Credentials： true
```

如果发送的是带凭据的请求， 但服务器的响应中没有包含这个头部， 那么浏览器就不会吧响应交给JavaScript ， responseText 中坚实空字符串， status 值为0 ， 而且会调用 onError 事件处理程序， 另外 服务器还可以在 Preflight 响应中发送这个 HTTP 头部， 表示允许源发送带凭据的请求

支持此属性的浏览器 FIreFox 3.5+ , safari 4+, chrome , IE 10+



#### 跨浏览器的 CORS 

即使浏览器对CORS的支持程度并不都一样， 但所有的浏览器都支持简单的（非Preflight和不带凭据的）请求， 因此 可以实现一个跨浏览器方案， 检测 XHR 是否支持 CORS 最简单的方式， 坚实检查 是否存在withCredentials  属性， 再结合 XDomainRequest 对象是否存在， 就可以兼顾所有的浏览器了

``` javascript
var request = createCORSRequest('get', 'http://www.abc.com/aaa')
if(request) {
   request.onload = function() {
       // 处理request.responseText
   }
    request.send()
}

function createCORSRequest(method, url) {
    var xhr = new XMLHttpRequest()
    if('withCredentials' in xhr) {
       xhr.open(method, url, true)
     }
    else if(typeof XDomainRequest  != undefined){
        xhr = new XDomainRequest()
         xhr.open(method, url)
    }
    else{
        xhr = null
    }
    
    return xhr
}
```



#### 图像 Ping

一个网页可以从任何网页中加载图像， 不用担心跨域不跨域，这也是在线广告跟踪浏览量的主要方式， 所以  可以动态的创建图像， 使用他们的 onload 和 onerror 事件处理程序来确定是否接收到了响应，动态创建图像请求的数据是通过查询字符串形式发送的

``` javascript
var img = new Image()
img.onload = img.onerror = function() {
    alert('done')
}
img.src = 'www.abc.com/aaa?name=bob'


// 请求从设置 src 属性的那一刻开始
// 图像ping 有两个缺点
// 一是只能发送 GET 请求
// 而是无法访问服务器的响应文本
// 因此 图像ping 只能用于浏览器与服务器间的单项通信
```



#### JSONP

JSONP 是 JSON with padding 的缩写

jsonp 是通过 动态 <script> 元素， 使用时 可以为 src 属性指定一个跨越url ， 这里的 <script> 元素 与 <img> 元素类似， 都有能力不受限制地从其他域中加载资源， 因为 jsonp 是有效的 JavaScript 代码， 所以在请求完成后， 即在 jsonp 响应加载到页面中以后， 就会立即执行

``` javascript
function handleResponse(response) {
    // ...
}

var script = document.createElement('script')
script.src = 'http://abc.com/aaa?callback=handleResponse'

document.body.insertBefore(script, document.body.firstChild)
```

JSONP有两点不足:

* JSONP 是从其他域中加载代码执行， 可能夹带一些恶意代码
* 要确定jsonp 请求是否失败并不容易， 虽然 html5 给 script 元素新增了一个 onerror 事件处理程序，但目前还没有任何浏览器支持，为此 只能通过超时时间来判断





#### Comet 

comet 指的是一种更高级的ajax 技术， 一种服务器想页面推送数据的技术，有两种实现方式： 

* 长轮询
* 流

长轮询是 传统轮询（短轮询）的一个翻版， （短轮询是指 浏览器定时向服务器发送请求）

长轮询是指 页面发起一个请求到服务器， 然后服务器一直保持连接打开 ，知道有数据可发送， 发送完数据之后， 浏览器关闭连接， 随机又发起一个到服务器的心情求， 这一过程在页面打开期间一直持续不断

轮询的优势是 所有的 浏览器都支持， 因为使用 XHR 对象和 setTimeout() 就能实现， 要做的就是决定什么时候发送请求



第二种流行的Comet 实现是 HTTP流， 流不同于上述两种轮询， 他在页面的整个生命周期内只使用一个HTTP 连接 ， 具体来说就是浏览器向服务器发送一个请求， 而服务器保持链接打开， 然后周期性地向浏览器发送数据

在 Firefox Safari Opera Chrome 中 通过 侦听 readystatechange 事件及检测 readyState 的值 是否为3 ， 就可以利用 XHR 对象实现 HTTP 流

``` javascript
var client = createStreamingClient(
	'streaming.php',
    function(data) {
        // received data
    },
    function(data) {
        // done
    }
)

function createStreamingClient(url, progress, finished) {
    var xhr = new XMLHttpRequest()
    var received = 0
    
    xhr.open('get', url, true)
    xhr.onreadystatechange = function() {
        var result
        
        if(xhr.readystate == 3) {
           result = xhr.responseText.substring(received)
           received += result.length
            
           progress(result)
        }
        else if(xhr.readystate == 4) {
         	finished(xhr.responseText)
        }
    }
    
    xhr.send(null)
    
    return xhr
}
```

comet 连接是很容易出错的， 为了简化 这技术， 又为Comet 创建了两个新的接口



#### 服务器发送事件

SSE (Server-Sent Events, 服务器发送事件) 是围绕只读Comet 交互推出的API 或者 模式， SSE API 用于创建到服务器的单向链接， 服务器通过这个连接可以发送任意数量的数据，服务器响应的 MIME 类型必须是text/event-stream,   

SSE  支持 短轮询， 长轮询 和 HTTP 流， 而且能够在断开连接时自动确定何时能重新连接

支持 SSE 的浏览器： Firefox 6+  Safari 5+ Opera 11+ Chrome 和 IOS 4+ 版 Safari

**SSE API**

要预定一个新的事件流， 首先要创建一个新的 EventSource 对象， 并传递一个入口点

``` javascript
var source = new EventSource('myevents.php')
```

注意： 传入的 URL 必须 与创建对象的页面同源

EventSource 的实例有一个 readyState 属性

​	0: 表示正连接到服务器

​	1：表示打开了连接

​    2: 表示关闭了连接

另外 ， 还有三个事件

* open:  建立连接时触发
* message： 从服务器接收到新事件时触发
* error： 无法建立连接时触发

默认情况下， EventSource 对象会保持与服务器的活动连接， 如果连接断开， 还会重新连接， 这就意味着SSE 适合长轮询和HTTP 流， 如果想强制立即断开连接并且不再重新连接， 可以调用 close() 



**事件流**

所谓的服务器事件会通过一个持久的HTTP 响应发送， 这个响应的 MIME类型为 text/evetn-stream , 响应的格式是纯文本， 最简单的情况是每个数据项都带有前缀 data: 

``` javascript
// eg: 
data: foo
data: bar
```

如果连接断开， 会向服务器发送一个 包含为Last-Event-ID 的特殊HTTP 头部的请求，以便服务器知道下次该触发哪个事件，在多次的连接的事件流中， 这种机制可以确保浏览器以正确的顺序收到连接的数据段



#### Web Sockets

web sockets 的目标是在一个单独的 持久连接上提供全双工，双向通信， 在 JavaScript 中创建了web socket 之后， 会有一个 HTTP 请求从浏览器发起连接， 在取得服务器响应后， 建立的连接会使用 HTTP 升级从 HTTP 协议 交换为 web socket 协议， 也就是说 使用 标准的 http 服务器 无法实现 web sockets 只有使用这种协议的专门服务器才能正常工作

使用自定义协议 而非 HTTP 协议的好处 是 ， 能够在 客户端和服务器之间发送非常少的数据， 而不必担心 HTTP 那样字节级 的开销。 由于传递的数据包很小， 因此 web sockets 非常适合移动应用

目前支持 web sockets 的浏览器有： Firefox 6+ Safari 5+ Chrome ios4+ 版Safari



**web sockets api**

``` javascript
var socket = new WebSocket('ws:www.abc.com/server.php')
```

必须给websocket 构造函数传入绝对URL ，同源策略对 websocks 不适用，因此可以通过它打开到任何站点的连接

与 XHR 类似， webSockets 也有表示当前状态的 readyState 属性

WebSocket.OPENING(0):  正在建立连接

WebSocket.OPEN(1): 已经建立连接

WebSocket.CLOSING(2): 正在关闭连接

WebSocket.CLOSE(3): 已经关闭连接

``` javascript
var socket = new WebSocket('ws://www.abc.com/server.php')
socket.send('hello word')
socket.onmessage = function(event) {
    // 收到的数据
    // event.data 返回的数据是字符串 要手动解析
}
// 另外还有其他三个事件， 在连接生命周期的不同阶段触发
// open： 在成功建立连接时触发
// error: 在发生错误时触发， 连接不能持续
// close: 在连接关闭时触发

socket.onopen = () => {}
socket.onerror = () => {}
socket.onclose = (event) => {}
// 只有close 事件有额外的对象信息， wasClean, code , reason 
```

