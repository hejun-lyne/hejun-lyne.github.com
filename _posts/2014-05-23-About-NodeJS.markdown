---
layout: post
title: NodeJS点滴
---
#NodeJS点滴

NodeJS看起来挺有趣的，可以重拾许久未用的JavaScript～，这篇是看[Node入门](http://www.nodebeginner.org/index-zh-cn.html)的一些简单记录。
安装很简单
-----
几个命令就搞定了，还是去看官方的[安装指南](https://github.com/joyent/node/wiki/Installation)吧
开始搭建自己的服务器
-----
创建文件`server.js`来存放服务器的主代码吧。一切事情开始都是非常简单的：
```js
var http = require("http");

http.createServer(function(request, response) {
  response.writeHead(200, {"Content-Type": "text/plain"});
  response.write("Hello World");
  response.end();
}).listen(8888);
```
下面让NodeJS启动我们的服务器：
```
node server.js
```
这里关于JavaScript的语法应该非常熟悉，不过这里体现了NodeJS的一个重要特点**事件驱动**，可看看这个文章[Understanding node.js](http://debuggable.com/posts/understanding-node-js:4bd98440-45e4-4a9a-8ef7-0f7ecbdd56cb)。
模块的划分
-----
`require("http")`就是引入了http模块，这是Node.js里面的http服务器模块，下面说下怎么把server也封装成模块。
其实很简单，只要把服务器脚本封装成方法，然后把方法设置到`exports` 之中
```
var http = require("http");

function start() {
  function onRequest(request, response) {
    console.log("Request received.");
    response.writeHead(200, {"Content-Type": "text/plain"});
    response.write("Hello World");
    response.end();
  }

  http.createServer(onRequest).listen(8888);
  console.log("Server has started.");
}

exports.start = start;
```
再新创建一个`index.js`文件：
```
var server = require("./server");

server.start();
```
对`request`对象的处理
-----
首先是需要为request的路由提供请求的url和参数，那么怎么提取这些信息呢？Node已经有现成的模块完成这个工作了，就是`url`和`querystring`模块：
```
                                url.parse(string).query
                                           |
           url.parse(string).pathname      |
                       |                   |
                       |                   |
                     ------ -------------------
http://localhost:8888/start?foo=bar&hello=world
                                ---       -----
                                 |          |
                                 |          |
              querystring(string)["foo"]    |
                                            |
                         querystring(string)["hello"]
```
好了，下面可以开始实现路由模块了
改造`server.js`：
```
var http = require("http");
var url = require("url");

function start(route) {
  function onRequest(request, response) {
    var pathname = url.parse(request.url).pathname;
    console.log("Request for " + pathname + " received.");

    route(pathname);

    response.writeHead(200, {"Content-Type": "text/plain"});
    response.write("Hello World");
    response.end();
  }

  http.createServer(onRequest).listen(8888);
  console.log("Server has started.");
}

exports.start = start;
```
在`index.js`中传递路由函数进行处理：
```
var server = require("./server");
var router = require("./router");

server.start(router.route);
```
很多时候我们需要针对不同的请求进行不同的处理，那我们总不可能加一个请求url就改一次代码，所以我们可以把处理请求的代码封装一下，考虑到js里面的对象其实是key-value的dictionary，可以非常简单的把请求名字和方法名对应上，我们写一个`requestHandler.js`模块：
```
function start() {
  console.log("Request handler 'start' was called.");
  return "Hello Start";
}

function upload() {
  console.log("Request handler 'upload' was called.");
  return "Hello Upload";
}

exports.start = start;
exports.upload = upload;
```
然后改一下`route.js`：
```
function route(handle, pathname) {
  console.log("About to route a request for " + pathname);
  if (typeof handle[pathname] === 'function') {
    return handle[pathname]();
  } else {
    console.log("No request handler found for " + pathname);
    return "404 Not found";
  }
}

exports.route = route;
```
然后是`server.js`：
```
var http = require("http");
var url = require("url");

function start(route, handle) {
  function onRequest(request, response) {
    var pathname = url.parse(request.url).pathname;
    console.log("Request for " + pathname + " received.");

    response.writeHead(200, {"Content-Type": "text/plain"});
    var content = route(handle, pathname)
    response.write(content);
    response.end();
  }

  http.createServer(onRequest).listen(8888);
  console.log("Server has started.");
}

exports.start = start;
```
接下来如果要增加新的请求url，就只要在`route.js`增加对应的方法即可，其他文件都不需要改动。
非阻塞操作／事件驱动
----
Node是通过事件轮询来实现并行处理的，实际上是单线程，如果你的代码阻塞了这个线程，那就是阻塞了所有的请求处理，比如下面这样：
```
function start() {
  console.log("Request handler 'start' was called.");

  function sleep(milliSeconds) {
    var startTime = new Date().getTime();
    while (new Date().getTime() < startTime + milliSeconds);
  }

  sleep(10000);
  return "Hello Start";
}

function upload() {
  console.log("Request handler 'upload' was called.");
  return "Hello Upload";
}

exports.start = start;
exports.upload = upload;
```
这里的sleep使用了while循环，实际是阻塞了线程。
那像数据库查询等这些必须要异步处理的操作怎么办呢，这时候需要使用子线程的模块`child_process`：
```
var exec = require("child_process").exec;

function start() {
  console.log("Request handler 'start' was called.");
  var content = "empty";

  exec("ls -lah", function (error, stdout, stderr) {
    content = stdout;
  });

  return content;
}

function upload() {
  console.log("Request handler 'upload' was called.");
  return "Hello Upload";
}

exports.start = start;
exports.upload = upload;
```
为了能够将异步处理的结果返回给客户端，还得把`response`对象交到异步处理的回调函数中写入返回，而不是在`route`模块里面返回。
首先改一下`server.js`，让它把`response`传递给`route`：
```
var http = require("http");
var url = require("url");

function start(route, handle) {
  function onRequest(request, response) {
    var pathname = url.parse(request.url).pathname;
    console.log("Request for " + pathname + " received.");

    route(handle, pathname, response);
  }

  http.createServer(onRequest).listen(8888);
  console.log("Server has started.");
}

exports.start = start;
```
然后通过`route`把`response`传递给`requestHandeler`：
```
function route(handle, pathname, response) {
  console.log("About to route a request for " + pathname);
  if (typeof handle[pathname] === 'function') {
    handle[pathname](response);
  } else {
    console.log("No request handler found for " + pathname);
    response.writeHead(404, {"Content-Type": "text/plain"});
    response.write("404 Not found");
    response.end();
  }
}

exports.route = route;
```
终于可以真正交给`requestHandler`去处理了：
```
var exec = require("child_process").exec;

function start(response) {
  console.log("Request handler 'start' was called.");

  exec("ls -lah", function (error, stdout, stderr) {
    response.writeHead(200, {"Content-Type": "text/plain"});
    response.write(stdout);
    response.end();
  });
}

function upload(response) {
  console.log("Request handler 'upload' was called.");
  response.writeHead(200, {"Content-Type": "text/plain"});
  response.write("Hello Upload");
  response.end();
}

exports.start = start;
exports.upload = upload;
```
如何处理Post请求
-----
因为post请求一般会包含大量的数据，所以我们需要使用异步非阻塞的方式来处理。例如给`request`添加`listener`，改一下`server.js`（组装数据的工作交给了server来实现）：
```
var http = require("http");
var url = require("url");

function start(route, handle) {
  function onRequest(request, response) {
    var postData = "";
    var pathname = url.parse(request.url).pathname;
    console.log("Request for " + pathname + " received.");

    request.setEncoding("utf8");

    request.addListener("data", function(postDataChunk) {
      postData += postDataChunk;
      console.log("Received POST data chunk '"+
      postDataChunk + "'.");
    });

    request.addListener("end", function() {
      route(handle, pathname, response, postData);
    });

  }

  http.createServer(onRequest).listen(8888);
  console.log("Server has started.");
}

exports.start = start;
```
需要再route方法加一个传递数据的参数，比较简单，直接看`requestHandler.js`应该怎么修改：
```
function start(response, postData) {
  console.log("Request handler 'start' was called.");

  var body = '<html>'+
    '<head>'+
    '<meta http-equiv="Content-Type" content="text/html; '+
    'charset=UTF-8" />'+
    '</head>'+
    '<body>'+
    '<form action="/upload" method="post">'+
    '<textarea name="text" rows="20" cols="60"></textarea>'+
    '<input type="submit" value="Submit text" />'+
    '</form>'+
    '</body>'+
    '</html>';

    response.writeHead(200, {"Content-Type": "text/html"});
    response.write(body);
    response.end();
}

function upload(response, postData) {
  console.log("Request handler 'upload' was called.");
  response.writeHead(200, {"Content-Type": "text/plain"});
  response.write("You've sent: " + postData);
  response.end();
}

exports.start = start;
exports.upload = upload;
```
ok，已经完成了如何处理一个post请求的过程了，下面再看看怎么处理上传文件这种复杂一点的事情。
需要用用一个外部模块`formidable`，先通过`npm`安装一下：
```
npm install formidable
```
改一下`requestHandler.js`：
```
var querystring = require("querystring"),
    fs = require("fs"),
    formidable = require("formidable");

function start(response) {
  console.log("Request handler 'start' was called.");

  var body = '<html>'+
    '<head>'+
    '<meta http-equiv="Content-Type" content="text/html; '+
    'charset=UTF-8" />'+
    '</head>'+
    '<body>'+
    '<form action="/upload" enctype="multipart/form-data" '+
    'method="post">'+
    '<input type="file" name="upload" multiple="multiple">'+
    '<input type="submit" value="Upload file" />'+
    '</form>'+
    '</body>'+
    '</html>';

    response.writeHead(200, {"Content-Type": "text/html"});
    response.write(body);
    response.end();
}

function upload(response, request) {
  console.log("Request handler 'upload' was called.");

  var form = new formidable.IncomingForm();
  console.log("about to parse");
  form.parse(request, function(error, fields, files) {
    console.log("parsing done");
    fs.renameSync(files.upload.path, "/tmp/test.png");
    response.writeHead(200, {"Content-Type": "text/html"});
    response.write("received image:<br/>");
    response.write("<img src='/show' />");
    response.end();
  });
}

function show(response) {
  console.log("Request handler 'show' was called.");
  fs.readFile("/tmp/test.png", "binary", function(error, file) {
    if(error) {
      response.writeHead(500, {"Content-Type": "text/plain"});
      response.write(error + "\n");
      response.end();
    } else {
      response.writeHead(200, {"Content-Type": "image/png"});
      response.write(file, "binary");
      response.end();
    }
  });
}

exports.start = start;
exports.upload = upload;
exports.show = show;
```