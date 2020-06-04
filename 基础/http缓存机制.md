## HTTP 缓存机制

http 缓存机制分为两大类 

* 强缓存
* 协商缓存



#### 强缓存

服务器返回的请求报文中， Expires 和 Cache-Control  表示强缓存的字段说明， cache-control 优先级 大于 expires 

**Expires:**  指定缓存的实际过期时间，而不是 过期的秒数， 不推荐使用,   使用本地时间与 expires 做对比，会存在 本地时间不准确，产生过期时间不准确的情况

``` javascript
Expires: Tue, 19 Mar 2019 12:05:15 GMT
```

**Cache-Control**:    Cache-Control 值可能为如下

``` javascript
public: 资源可能被客户端和浏览器缓存
private: 资源仅被客户端缓存， 代理服务器不缓存
no-store: 请求和响应都不缓存
max-age: 缓存资源， 但是在指定时间（秒）后缓存过期
no-cache: 相当于 max-age:0
s-maxage: 依赖public 设置 覆盖 max-age 且只在代理服务器上有效
max-stale: 指定时间内 即使缓存过期 资源依然有效
max-fresh: 缓存的资源至少保持指定时间的新鲜期
must-revalidation/proxy-revalidation: 如果缓存失效 强制重定向服务器或代理发起验证（max-stale 等字段可能改变缓存的失效时间）
only-if-cached: 仅仅返回已经缓存的资源 不访问网络 若无缓存则返回504
no-transform: 强制要求服务器不要对资源进行转换 禁止代理服务器对 content-encoding content-range content-type  字段的修改
```

> 当 Cache-Control 值 为 no-cache 时 ， 并不代表不缓存该资源， 而是在于原始服务器进行新鲜度再验证之前，缓存不能提供给客户端使用

> Cache-Control 值 为 no-store 才是不缓存资源到本地



#### 协商缓存

当强缓存没有命中时， 客户端就会发起请求到服务器， 验证协商缓存是否命中，如果协商缓存命中， 请求响应返回 304 ， 并且 会有 not modified 

协商缓存通过 Last-modified, If-Modified-Since  和  ETag ， If-None-Match 这两对 响应头字段来控制的

**Last-Modified**:  用于标记请求资源最后一次修改时间， 优先级比[ETag ， If-none-match]  优先级低， 时间只能精确到秒， 因此不太适合短时间内频繁改动，  如果服务端的静态资源，进行了编译打包，可能出现静态资源内容没变， Last-Modified 改变的情况

**If-Modified-Since: ** 当浏览器请求时， 会在请求头上 加 If-Modified-Since 字段， 值为 Last-Modified ， 询问服务器 资源是否有更新， 有更新 则会返回新的资源 ， 若没有更新 返回 304 使用本地缓存

**ETag:**  可以理解为每个资源的唯一ID， 每当服务端有资源更新 ，ETag 都会更新

**If-None-Match: **  当浏览器进行请求时， 会在请求头上加 If-None-Match 字段 ， 值为 ETag， 服务端会比较 ， 决定返回 304 或 200



#### 用户行为与缓存

| 用户操作            | Expires/Cache-Control | Last-Modified/ETag |
| ------------------- | --------------------- | ------------------ |
| 地址栏回车          | 有效                  | 有效               |
| 页面链接跳转        | 有效                  | 有效               |
| 新开窗口            | 有效                  | 有效               |
| 前进,后退           | 有效                  | 有效               |
| F5刷新              | 无效                  | 有效               |
| 强制刷新（CTRL+F5） | 无效                  | 无效               |