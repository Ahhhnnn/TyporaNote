## Node.js 创建第一个应用

- **引入 required 模块：**我们可以使用 **require** 指令来载入 Node.js 模块。
- **创建服务器：**服务器可以监听客户端的请求，类似于 Apache 、Nginx 等 HTTP 服务器。
- **接收请求与响应请求** 服务器很容易创建，客户端可以使用浏览器或终端发送 HTTP 请求，服务器接收请求后返回响应数据。

```js
var http = require('http');
http.createServer(function (request, response) {
    // 发送 HTTP 头部 
    // HTTP 状态值: 200 : OK
    // 内容类型: text/plain
    response.writeHead(200, {'Content-Type': 'text/plain'});

    // 发送响应数据 "Hello World"
    response.end('Hello World\n');
}).listen(8888);

// 终端打印如下信息

console.log('Server running at http://127.0.0.1:8888/');
```

使用node xxxx.jx 命令运行



## NPM 使用介绍

- NPM是随同NodeJS一起安装的包管理工具，能解决NodeJS代码部署上的很多问题，常见的使用场景有以下几种：
  - 允许用户从NPM服务器下载别人编写的第三方包到本地使用。
  - 允许用户从NPM服务器下载并安装别人编写的命令行程序到本地使用。
  - 允许用户将自己编写的包或命令行程序上传到NPM服务器供别人使用。



### 使用 npm 命令安装模块

```shell
npm install <Module Name>
npm install express
```

### 全局安装与本地安装

#### 本地安装

-  将安装包放在 ./node_modules 下（运行 npm 命令时所在的目录），如果没有 node_modules 目录，会在当前执行 npm 命令的目录下生成 node_modules 目录。
- 可以通过 require() 来引入本地安装的包。

#### 全局安装

-  将安装包放在 /usr/local 下或者你 node 的安装目录。
- 可以直接在命令行里使用

```
npm install express      # 本地安装
npm install express -g   # 全局安装
```

#### 查看node_modules文件夹在哪

```
npm root -g
```

### 卸载

```
npm uninstall express
```

### 更新模块

```
npm update express
```

### 搜索模块

```
npm search express
```

### 创建模块

```
npm init
```

然后根据提示输入信息



## Node.js 回调函数

回调函数在完成任务后就会被调用，Node 使用了大量的回调函数，Node 所有 API 都支持回调函数。

例如，我们可以一边读取文件，一边执行其他命令，在文件读取完成后，我们将文件内容作为回调函数的参数返回。这样在执行代码时就没有阻塞或等待文件 I/O 操作。这就大大提高了 Node.js 的性能，可以处理大量的并发请求。

```js
function foo1(name, age, callback) { }
function foo2(value, callback1, callback2) { }
```

### 阻塞代码实例

```js
var fs = require("fs");
var data = fs.readFileSync('test.txt');
console.log(data.toString());
console.log("程序执行结束!");
```

nihao hening!!!
程序执行结束!

### 非阻塞代码实例

```js
var fs = require("fs");

fs.readFile('test.txt', function (err, data) {
    if (err) return console.error(err);
    console.log(data.toString());
});
console.log("程序执行结束!");
```

程序执行结束!
nihao hening!!!

**备注** 

在使用方法时不知道具体参数参考:https://nodejs.org/api/fs.html#fs_fs_readfile_path_options_callback



## Node.js 事件循环

![image-20200315175334163](C:\Users\YMXD\AppData\Roaming\Typora\typora-user-images\image-20200315175334163.png)

整个事件驱动的流程就是这么实现的，非常简洁。有点类似于观察者模式，事件相当于一个主题(Subject)，而所有注册到这个事件上的处理函数相当于观察者(Observer)。

eventEmitter.emit('data_received'); emit触发某个方法

eventEmitter.on('connection', connectHandler);当触发connection事件，调用connectHandler方法处理

```js
// 引入 events 模块
var events = require('events');
// 创建 eventEmitter 对象
var eventEmitter = new events.EventEmitter();
 
// 创建事件处理程序
var connectHandler = function connected() {
   console.log('连接成功。');
  
   // 触发 data_received 事件 
   eventEmitter.emit('data_received');
}
 
// 绑定 connection 事件处理程序
eventEmitter.on('connection', connectHandler);
 
// 使用匿名函数绑定 data_received 事件
eventEmitter.on('data_received', function(){
   console.log('数据接收成功。');
});
 
// 触发 connection 事件 
eventEmitter.emit('connection');
 
console.log("程序执行完毕。");
```



## Node.js模块系统

