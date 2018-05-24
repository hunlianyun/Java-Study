# AJAX

## 一、ajax 定义
> ajax 全称 Asynchronous JavaScript and XML，即异步的 JavaScript 和 XML

- Ajax 是一种用于创建快速动态网页的技术。
- Ajax 是一种在无需重新加载整个网页的情况下，能够更新部分网页的技术。
- 通过在后台与服务器进行少量数据交换，Ajax 可以使网页实现异步更新。这意味着可以在不重新加载整个网页的情况下，对网页的某部分进行更新。
- 传统的网页（不使用 Ajax）如果需要更新内容，必须重载整个网页页面。

## 二、ajax 应用场景
当我们加载需要对页面的局部进行更新时，如果没有 ajax，用户点击某个按钮，通常需要向服务器发送请求，服务器再返回完整页面，在这期间用户只能等待，而实际上也许我们只是需要很小的一个文本，却花了如此大的代价（在请求量庞大时，这个代价更为明显）。于是，在这种背景下，使用 ajax 可以异步的向服务器发送请求，进而更新页面局部效果。

## 三、ajax 使用
使用 ajax 大致可以分为四步：
- 获取 XMLHttpRequest 对象
- 调用 XMLHttpRequest 对象的 `open()` 方法
- 调用 XMLHttpRequest 对象的 `send()` 方法
- 监听 XMLHttpRequest 对象的 `onreadystatechange` 事件

### 1，获取 XMLHttpRequest 对象
在 JavaScript 中，大部分浏览器创建 XMLHttpRequest 对象的方法为：
```JavaScript
var xmlHttp = new XMLHttpRequest();
```
但有少部分浏览器（早期的 IE 浏览器）创建的方法是这样的：
```JavaScript
var xmlHttp = new ActiveXObject("Microsoft.XMLHTTP");
```
通常为了兼容性考虑，我们使用如下代码获取 XMLHttpRequest 对象：
```JavaScript
var xmlHttp;
if (window.XMLHttpRequest) {
    //  IE7+, Firefox, Chrome, Opera, Safari 浏览器执行代码
    xmlHttp = new XMLHttpRequest();
} else {
    // IE6, IE5 浏览器执行代码
    xmlHttp = new ActiveXObject("Microsoft.XMLHTTP");
}
```

### 2，调用 XMLHttpRequest 对象的 `open()` 方法
在获得了 XMLHttpRequest 对象后，接下来要想向服务器发起请求，我们还得调用它的 `open()` 方法，来指定请求的相关属性，该方法有三个参数：
- **method：请求的方法，GET 或 POST**
- **url：向服务器发起请求的 url**
- **async：true（异步）或 false（同步）**


GET 方法的使用非常简单，请求参数跟着 url 后面即可
```JavaScript
xmlHttp.open("GET", "/xxx.jsp?param1=aaa&param2=bbb", true);
```
POST 方法的使用稍微复杂一点，需要添加一个 HTTP 头信息，而请求参数要放在后面的 `send()` 方法中
```JavaScript
xmlHttp.open("POST", "/xxx.jsp", true);
xmlHttp.setRequestHeader("Content-type","application/x-www-form-urlencoded");
```

### 3，调用 XMLHttpRequest 对象的 `send()` 方法
接下来就可以向服务器正式发送请求了，根据请求方式的不同，代码稍有差异：
- 使用 GET 请求方式时直接调用 `xmlHttp.send()` 或者 `xmlHttp.send(null)` 即可
- 使用 POST 请求方式时，需要传递一个字符串表示请求参数，如：
```JavaScript
xmlHttp.send("param1=aaa&param2=bbb");
```

### 4，监听 XMLHttpRequest 对象的 `onreadystatechange` 事件
发送了请求之后，要想知道服务器响应情况，就要监听这个方法，通常建议将监听写在 `open()` 方法之前，因为这个监听的状态就包括调用 `open()` 方法，这里列在第 4 步是逻辑上便于理解。这里介绍 XMLHttpRequest 对象的几个重要属性，即 **readyState** 、**status**、 **responseText**、**responseXML**
- **readyState**  表示 HTTP 请求的状态，当一个 XMLHttpRequest 初次创建时，这个属性的值从 0 开始，直到接收到完整的 HTTP 响应，这个值增加到 4。
具体的五种状态如下表：

|状态|名称|描述|
|:-:|:-|:-|
| 0 |  Uninitialized  | 初始化状态。XMLHttpRequest 对象已创建或已被 abort() 方法重置 |
| 1 | Open | open() 方法已调用，但是 send() 方法未调用，请求还没有被发送 |
| 2 | Sent | Send() 方法已调用，HTTP 请求已发送到 Web 服务器，未接收到响应 |
| 3 | Receiving | 所有响应头部都已经接收到，响应体开始接收但未完成 |
| 4 | Loaded | HTTP 响应已经完全接收 |

- **status** 由服务器返回的 HTTP 状态代码，如 200 表示成功，而 404 表示 "Not Found" 错误。当 readyState 小于 3 的时候读取这一属性会导致一个异常。

- **responseText** 表示目前为止为服务器接收到的响应体（不包括头部），或者如果还没有接收到数据的话，就是空字符串。如果 readyState 小于 3，这个属性就是一个空字符串。当 readyState 为 3，这个属性返回目前已经接收的响应部分。如果 readyState 为 4，这个属性保存了完整的响应体。如果响应包含了为响应体指定字符编码的头部，就使用该编码。否则，假定使用 Unicode UTF-8。

- **responseXML** 表示把请求的响应解析为 XML 并作为 Document 对象返回。

理解了这几个属性后，就可以写出完整的监听代码了
```JavaScript
xmlHttp.onreadystatechange = function() {
  // 双重判断，确保获得完整的响应数据
  if (xmlHttp.readyState == 4 && xmlHttp.status == 200) {
    var responseStr = xmlhttp.responseText;
    // 在这儿使用响应数据用 JavaScript 更改页面吧
    ...
  }
}
```
