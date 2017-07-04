---
title: 同源策略及跨域方法
date: 2017-01-10 22:05:03
categories:
- javascript
tags: 
- 同源策略 
- 跨域共享
- jsonp
- CORS
---
### 同源策略

> 同源策略是客户端脚本（尤其是Javascript）的重要的安全度量标准。它最早出自Netscape Navigator2.0，其目的是防止某个文档或脚本从多个不同源装载。

同源指的是：同协议，同域名和同端口。

非同源会有以下种行为受到限制：
-  Cookie、LocalStorage 和 IndexDB 无法读取。
-   DOM 无法获得。
-  AJAX 请求不能发送。
-  浏览器中不同域的框架之间不能进行js的交互操作。
-  不同的框架之间是可以获取window对象的，但却无法获取相应的属性和方法。

常用的同源策略的规避方法，即常用跨域方法如下：

### document.domain
只要将同一域下不同子域的document.domain设置为共同的父域，则可以访问对应window的各种属性和方法。
例如：www.example.com父域下的www.lib.example.com和www.hr.example.com两个子域，将对应页面的document.domain设为example.com即可共享cookie等资源。

另外，服务器也可以在设置Cookie的时候，指定Cookie的所属域名为一级域名，比如.example.com。这样，二级域名和三级域名不用做任何设置，都可以读取这个Cookie。
``` http
Set-Cookie: key=value; domain=.example.com; path=/
```

使用js来获取不同子域的iframe中的内容,本例中当前页域名为：http://example.com/a.html 

``` html
<script type="text/javascript">
    function test(){
        var iframe = document.getElementById('ifame');
        var win = iframe.contentWindow;//可以获取到iframe里的window对象，但该window对象的属性和方法几乎是不可用的
        var doc = win.document;//这里获取不到iframe里的document对象
        var name = win.name;//这里同样获取不到window对象的name属性
    }
</script>
<iframe id = "iframe" src="http://www.example.com/b.html" onload = "test()"></iframe>
```

通过在两个页面的js脚本中都设置 【document.domain = 'example.com';】遍可以共享frame内容。

**缺点**：只能在一级域名相同时才能运用；此方法只适用于 Cookie 和 iframe 窗口。

### window.name
浏览器窗口window对象的name属性——特征：即在一个窗口(window)的生命周期内,窗口载入的所有的页面都是共享一个window.name的，每个页面对window.name都有读写的权限，window.name是持久存在一个窗口载入过的所有页面中的，并不会因新页面的载入而进行重置。
可以在某个页面设置好 window.name 的值，然后跳转到另外一个页面。在这个页面中就可以获取到我们刚刚设置的 window.name 了。

### location.hash(#)

**原理**:
	父窗口可以对iframe进行URL读写，iframe也可以读写父窗口的URL。此方法的原理就是改变URL的hash部分来进行双向通信。
每个window通过改变其他 window的location来发送消息，并通过监听自己的URL的变化来接收消息。（由于两个页面不在同一个域下IE、Chrome不允许修改parent.location.hash的值，所以要借助于父窗口域名下的一个代理iframe）
假如父页面是 baidu.com/a.html,iframe 嵌入的页面为google.com/b.html，要实现此两个页面间的通信可以通过以下方法。

	a.html传送数据到b.html
	a.html下修改iframe的src为google.com/b.html#paco
	b.html监听到url发生变化，触发相应操作
	b.html传送数据到a.html，由于两个页面不在同一个域下IE、Chrome不允许修改parent.location.hash的值，所以要借助于父窗口域名下的一个代理iframe

b.html下创建一个隐藏的iframe，此iframe的src是baidu.com域下的，并挂上要传送的hash数据，如src="http://www.baidu.com/proxy.html#data"
proxy.html监听到url发生变化，修改a.html的url（因为a.html和proxy.html同域，所以proxy.html可修改a.html的url hash）
a.html监听到url发生变化，触发相应操作：
b.html页面的关键代码如下:

``` javascript
try {  
    parent.location.hash = 'data';  
} catch (e) {  
    // ie、chrome的安全机制无法修改parent.location.hash，  
    var ifrproxy = document.createElement('iframe');  
    ifrproxy.style.display = 'none';  
    ifrproxy.src = "http://www.baidu.com/proxy.html#data";  
    document.body.appendChild(ifrproxy);  
}
```

**缺点**：会造成一些不必要的浏览器历史记录，而且有些浏览器不支持onhashchange事件，需要轮询来获知URL的改变，最后，这样做也存在缺点，诸如数据直接暴露在了url中，数据容量和类型都有限等。

###  HTML5的postMessage方法

此方法允许跨窗口通信，不论这两个窗口是否同源。
- 该方法的第一个参数message为要发送的消息，类型只能为字符串；第二个参数targetOrigin用来限定接收消息的那个window对象所在的域，如果不想限定域，可以使用通配符 *  。
- 需要接收消息的window对象，可是通过监听自身的message事件来获取传过来的消息，消息内容储存在该事件对象的data属性中。
- 通信过程如下：

A页面通过postMessage方法发送消息：

``` javascript
window.onload = function() {  
    var ifr = document.getElementById('ifr');  
    var targetOrigin = "http://www.google.com";  
    ifr.contentWindow.postMessage('hello world!', targetOrigin);  
};
```

B页面通过message事件监听并接受消息:

``` javascript
var onmessage = function (event) {  
  var data = event.data;//消息  
  var origin = event.origin;//消息来源地址  
  var source = event.source;//源Window对象  
  if(origin=="http://www.baidu.com"){  
		console.log(data);//hello world!  
  }  
};  
if (typeof window.addEventListener != 'undefined') {  
  window.addEventListener('message', onmessage, false);  
} else if (typeof window.attachEvent != 'undefined') {  
  //for ie  
  window.attachEvent('onmessage', onmessage);  
}
```

- 通过window.postMessage，也可读写其他窗口的 LocalStorage

父窗口发送json类型消息代码：

``` javascript
var win = document.getElementsByTagName('iframe')[0].contentWindow;
var obj = { name: 'Jack' };
// 存入对象
win.postMessage(JSON.stringify({key: 'storage', method: 'set', data: obj}), 'http://bbb.com');
// 读取对象
win.postMessage(JSON.stringify({key: 'storage', method: "get"}), "*");
window.onmessage = function(e) {
  if (e.origin != 'http://aaa.com') return;
  // "Jack"
  console.log(JSON.parse(e.data).name);
};
```

子窗口接收消息操作LocalStorage的代码：

``` javascript
window.onmessage = function(e) {
  if (e.origin !== 'http://bbb.com') return;
  var payload = JSON.parse(e.data);
  switch (payload.method) {
    case 'set':
      localStorage.setItem(payload.key, JSON.stringify(payload.data));
      break;
    case 'get':
      var parent = window.parent;
      var data = localStorage.getItem(payload.key);
      parent.postMessage(data, 'http://aaa.com');
      break;
    case 'remove':
      localStorage.removeItem(payload.key);
      break;
  }
};
```

*上述几种跨越方法是双向通信的，即两个iframe，页面与iframe或是页面与页面之间的，下面说几种单项跨域的（一般用来获取数据），因为通过script标签引入的js是不受同源策略的限制的。所以我们可以通过script标签引入js。*

针对ajax请求的同源限制，可以采用以下几种方法跨域：
 - JSONP
 - WebSocket
 - CORS

### JSONP
**原理**: 网页通过添加一个script标签，向服务器请求JSON数据；服务器收到请求后，将数据放在一个指定名字的回调函数里传回来。
``` javascript
function addScriptTag(src) {
  var script = document.createElement('script');
  script.setAttribute("type","text/javascript");
  script.src = src;
  document.body.appendChild(script);
}

window.onload = function () {
  addScriptTag('http://example.com/ip?callback=foo');
}

function foo(data) {
  console.log('Your public IP address is: ' + data.ip);
};
```
请求的查询字符串有一个callback参数，用来指定回调函数的名字，服务器收到这个请求以后，会将数据放在回调函数的参数位置返回。

jquery中，对jsonp的方法进行了封装，可在ajax请求中使用。

	    $.getJSON('http://example.com/data.php?callback=?',function(jsondata){
        //处理获得的json数据
    });
jquery会自动生成一个全局函数来替换callback=?中的问号，之后获取到数据后又会自动销毁，实际上就是起一个临时代理函数的作用。$.getJSON方法会自动判断是否跨域，不跨域的话，就调用普通的ajax方法；跨域的话，则会以异步加载js文件的形式来调用jsonp的回调函数。

使用jQuery的ajax接口实现jsonp调用：
``` javascript

      jQuery(document).ready(function(){ 
        $.ajax({
             type: "get",
             async: false,
             url: "http://flightQuery.com/jsonp/flightResult.aspx?code=CA1998",
             dataType: "jsonp",
             jsonp: "callback",//传递给请求处理程序或页面的，用以获得jsonp回调函数名的参数名(一般默认为:callback)
             jsonpCallback:"flightHandler",//自定义的jsonp回调函数名称，默认为jQuery自动生成的随机函数名，也可以写"?"，jQuery会自动为你处理数据
             success: function(json){
                 alert('您查询到航班信息：票价： ' + json.price + ' 元，余票： ' + json.tickets + ' 张。');
             },
             error: function(){
                 alert('fail');
             }
         });
     });
```
jquery在处理jsonp类型的ajax时，会自动生成回调函数并把数据取出来供success属性方法来调用。

**ajax与jsonp的异同**：
1、两种技术在调用方式上“看起来”很像，目的也一样，都是请求一个url，然后把服务器返回的数据进行处理，因此jquery和ext等框架都把jsonp作为ajax的一种形式进行了封装；

2、但ajax和jsonp其实本质上是不同的东西。ajax的核心是通过XmlHttpRequest获取非本页内容，而jsonp的核心则是动态添加script标签来调用服务器提供的js脚本。

3、ajax与jsonp的区别不在于是否跨域，ajax通过服务端代理一样可以实现跨域，jsonp本身也不排斥同域的数据的获取。

4、jsonp是一种方式或者说非强制性协议，如同ajax一样，它也不一定非要用json格式来传递数据，如果你愿意，字符串都行，只不过这样不利于用jsonp提供公开服务。

**JSONP的优缺点**
 **优点**：
 - 它不像XMLHttpRequest对象实现的Ajax请求那样受到同源策略的限制；
 - 它的兼容性更好，在更加古老的浏览器中都可以运行，不需要XMLHttpRequest或ActiveX的支持；
 - 并且在请求完毕后可以通过调用callback的方式回传结果。

**缺点**：
- 它只支持GET请求而不支持POST等其它类型的HTTP请求；
- 它只支持跨域HTTP请求这种情况，不能解决不同域的两个页面之间如何进行JavaScript调用的问题。
- JSONP请求失败时不返回http状态码。

### CORS
CORS是跨源AJAX请求的根本解决方法。相比JSONP只能发GET请求，CORS允许任何类型的请求。需要浏览器和服务器同时支持。目前，所有浏览器都支持该功能，IE浏览器不能低于IE10。
**原理**：使用自定义的HTTP头部让浏览器与服务器进行沟通，从而决定请求或响应是应该成功还是失败。整个CORS通信过程，都是浏览器自动完成，不需要用户参与。

浏览器将CORS请求分成两类（对这两种请求的处理不同）：
#### 简单请求
- method：HEAD,GET,POST
- HTTP的头信息类型：
Accept，Accept-Language，Content-Language，Last-Event-ID，Content-Type：application/x-www-form-urlencoded、multipart/form-data、text/plain
- **请求流程**：
浏览器发现这次跨源AJAX请求是简单请求，就自动在头信息之中，添加一个Origin字段（请求的域名）。如果Origin指定的源，不在许可范围内，服务器会返回一个正常的HTTP回应；如果Origin指定的域名在许可范围内，服务器返回的响应，会多出几个与CORS请求相关的以Access-Control-开头的头信息字段。

``` http
Access-Control-Allow-Origin: http://api.bob.com
Access-Control-Allow-Credentials: true
Access-Control-Expose-Headers: FooBar
Content-Type: text/html; charset=utf-8
```

#### 非简单请求
非简单请一般是有特殊要求的请求，比如请求方法是PUT或DELETE，或者Content-Type字段的类型是application/json。会在正式通信之前，增加一次HTTP查询请求，称为"预检"请求。
- **请求流程**：
1） 浏览器发现，是一个非简单请求，就自动发出一个"预检"请求，要求服务器确认可以这样请求。
  2）服务器收到"预检"请求以后，检查了Origin、Access-Control-Request-Method和Access-Control-Request-Headers字段以后，确认允许跨源请求，就可以做出回应；
否则，会返回一个正常的HTTP回应，但是没有任何CORS相关的头信息字段。这时，浏览器就会认定，服务器不同意预检请求，因此触发一个错误，被XMLHttpRequest对象的onerror回调函数捕获。
3）一旦服务器通过了"预检"请求，以后每次浏览器正常的CORS请求，就都跟简单请求一样，会有一个Origin头信息字段。服务器的回应，也都会有一个Access-Control-Allow-Origin头信息字段。


**CORS和JSONP对比**：
- JSONP只能实现GET请求，而CORS支持所有类型的HTTP请求。
- 使用CORS，开发者可以使用普通的XMLHttpRequest发起请求和获得数据，比起JSONP有更好的错误处理。
- JSONP主要被老的浏览器支持，它们往往不支持CORS，而绝大多数现代浏览器都已经支持了CORS。
- CORS更简单高效。


### 参考文章
 [1]：https://gold.xitu.io/post/5815f4abbf22ec006893b431
 [2]：http://www.cnblogs.com/dowinning/archive/2012/04/19/json-jsonp-jquery.html
 [3] :   http://www.ruanyifeng.com/blog/2016/04/same-origin-policy.html
 [4] :   http://www.ruanyifeng.com/blog/2016/04/cors.html