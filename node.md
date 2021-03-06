## 模块机制

- require

```
require.__defineGetter__      require.__defineSetter__
require.__lookupGetter__      require.__lookupSetter__
require.__proto__             require.constructor
require.hasOwnProperty        require.isPrototypeOf
require.propertyIsEnumerable  require.toLocaleString
require.toString              require.valueOf

require.apply                 require.arguments
require.bind                  require.call
require.caller                require.length
require.name

require.cache                 require.extensions
require.main                  require.prototype
require.resolve
```

- module

```
module.__defineGetter__      module.__defineSetter__
module.__lookupGetter__      module.__lookupSetter__
module.__proto__             module.constructor
module.hasOwnProperty        module.isPrototypeOf
module.propertyIsEnumerable  module.toLocaleString
module.toString              module.valueOf

module._compile              module.load
module.require

module.children              module.exports
module.filename              module.id
module.loaded                module.parent
module.paths
```

## 队列

- setTimeout,setInterval 将callback放入事件队列中，在定时器条件下执行，但是有可能受到循环时间的影响（比如某次循环时间过长就会影响callback的执行）

- process.nextTick(callback) 将callback放入事件队列中，下次Tick循环时执行callbak，每次循环执行所有的callback

- setImmdiate(callback) 将callback放入事件队列，下次Tick循环时执行callback，callback执行优先级低于process.nextTick的callback，每次循环执行第一个callback然后直接进入下一次循环

## 事件

```javascript
var events = require('events');
var emitter = events.EventEmitter();

emitter.on('event',callback);
emitter.once('event',callback);
emitter.addListener('event',callback);
emitter.removeListener('event');
emitter.removeAllListeners('event');
emitter.emit('event','smg');

emitter.setMaxListeners(0); //去掉监听器超过10个的限制
```
- 雪崩问题
```javascript
var events = require('events');
var emitter = events.EventEmitter();

var status = 'ready';
var select = function(callback){
  emitter.once('selected',callback);
  if(status == 'ready'){
    db.select('SQL',function(res){
      emitter.emit('selected',res);
      status = 'ready';
    });
    status = 'padding';
  }
}
```
- Deferred原理(大概就是这样的)
```javascript
var events = require('events');
var fetch = require('fetch');
var eventEmitter = events.EventEmitter();
//Promise继承eventEmitter
var Promise = function(){
  eventEmitter.call(this);
}
Promise.prototype = Object.creat(eventEmitter.prototype);

Promise.prototype.then = function(success,error){
  this.on('success',success);
  this.on('error',error);
}
var promise = new Promise();
var Deferred = function(task){
  this.count = 0;
  for(var i = 0; i < task.length; i++){
    task[i]();
  }
  return promise;
}
Deferred.prototype.resolve = function(res){
  this.count++;
  if(this.count == task.length){
    promise.emit('success',res);
  }
}
Deferred.prototype.reject = function(error){
  this.count++;
  if(this.count == task.length){
    promise.emit('error',error);
  }
}

//使用Deferred
var dfd = new Deferred([
  fetch({
    url:'url',
    success:function(res){
      dfd.resolve(res);
    }
  }),
  fetch({
    url:'url',
    success:function(res){
      dfd.resolve(res);
    }
  })
]).then(callback,callback);

```

## 参数传递
- 按值传递

number,string,boolean等都是按值传递

```javascript
var a = 1;
function aa(a){
  a = 2;
  console.log(a);//a
}
aa(a);
console.log(a);//1
```
- 引用传递

object是按引用传递

```javascript
var a = {b: 1};
function aa(a){
  a.b = 2;
  console.log(a);//{b: 2}
}
aa(a);
console.log(a);//{b: 2}
```

- 实参和形参

```javascript
var a = 1;
function aa(a){
  //a是形参
  console.log(a);//undefined
  a = 3;
  arguments[0] = 2;//arguments是实参的集合
  console.log(a);//3
}
aa();//如果有参数，这个参数是实参
```

## 流程控制
- next()
```javascript
var express = require('express');
var app = express();

app.use(中间件1);
app.use(中间件2);
app.use(中间件3);

```
中间件注册并执行，通过next()将控制权交给下一个中间件执行
- async
串行
```javascript
async.serise([task1(callback),task2(callback)],function(error,res){
  console.log(res);//[callback1,callback2]
})
```

并行
```javascript
async.parallel([task1(callback),task2(callback)],function(error,res){
  console.log(res);//[callback1,callback2]
})
```
限制并行
```javascript
//最多并行执行两个任务
async.parallelLimit([task1(callback),task2(callback)],2,function(error,res){
  console.log(res);//[callback1,callback2]
})
```

依赖串行
```javascript
async.waterfall([task1(callback),task2(arg,callback)],function(error,res){
  console.log(res);//callback2
})
```

自动串行
```javascript
var dep = {
  task1:function(callback){},
  task2:function(callback){},
}
async.auto(dep);//以最佳顺序执行串行
```
- Bigpipe

```javascript
var Bigpipe = require('bigpipe');
var bigpipe = new Bigpipe(num,{
  refuse:true,//实时返回
  timeout:500//超时时间
});

bigpipe.push(task,callback) //推送任务
bigpipe.on('full',function(len){}) //任务超过num触发回调
```

## 文件

```javascript
var fs = require('fs'),  
    path = require('path');  
  
function exists(path){  
     return fs.existsSync(path) || path.existsSync(path);  
} 

//如何判断是不是文件：
function isFile(path){  
    return exists(path) && fs.statSync(path).isFile();  
}

判断是不是目录：
function isDir(path){  
    return exists(path) && fs.statSync(path).isDirectory();  
}
```
读取文件内容的函数有异步的 fs.readFile() 和同步的 fs.readFileSync()。

打开文件：fs.open(path, flags[, mode], callback) （异步模式）

获取文件信息：var stats = fs.stat(path, callback)

stats.isFile()	如果是文件返回 true，否则返回 false。

stats.isDirectory()	如果是目录返回 true，否则返回 false。

stats.isBlockDevice()	如果是块设备返回 true，否则返回 false。

stats.isCharacterDevice()	如果是字符设备返回 true，否则返回 false。

stats.isSymbolicLink()	如果是软链接返回 true，否则返回 false。

stats.isFIFO()	如果是FIFO，返回true，否则返回 false。FIFO是UNIX中的一种特殊类型的命令管道。

stats.isSocket()	如果是 Socket 返回 true，否则返回 false。

写入文件:fs.writeFile(file, data[, options], callback) (异步模式)

读取文件:fs.read(fd, buffer, offset, length, position, callback) (异步模式)

关闭文件:fs.close(fd, callback)

截取文件:fs.ftruncate(fd, len, callback)

删除文件:fs.unlink(path, callback)

创建目录:fs.mkdir(path[, mode], callback)

读取目录:fs.readdir(path, callback)

删除目录:fs.rmdir(path, callback)


## 内存

`node --prof a.js`V8执行时的性能分析数据

`linux-tick-processor v8.log`查看日志工具（linux-tick-processor在deps/v8/tools下）\

```
> process.memoryUsage() //查看进程内存的使用情况
{ rss: 9261056,//常驻内存
heapTotal: 10481664,//申请的内存总量
heapUsed: 7326336 }//目前使用的内存量
```
```
> os
{ hostname: [Function: getHostname],
  loadavg: [Function: getLoadAvg],
  uptime: [Function: getUptime],
  freemem: [Function: getFreeMem],
  totalmem: [Function: getTotalMem],
  cpus: [Function: getCPUs],
  type: [Function: getOSType],
  release: [Function: getOSRelease],
  networkInterfaces: [Function: getInterfaceAddresses],
  homedir: [Function: getHomeDirectory],
  userInfo: [Function: getUserInfo],
  arch: [Function],
  platform: [Function],
  tmpdir: [Function],
  tmpDir: [Function],
  getNetworkInterfaces: [Function: deprecated],
  EOL: '\r\n',
  endianness: [Function] }
  ```
作用域：全局，局部（function,with），es6的块作用域

垃圾回收：声明函数时会建立一个作用域，函数内声明变量会挂在该作用域下，函数执行结束后作用域销毁，局部变量失效，其引用对象将在下次垃圾回收时被释放，主动回收内存包括delete操作、给变量赋值null/undefined
- 内存泄漏
原因：缓存、队列处理不及时（堆积）、作用域未释放

解决：添加缓存限制策略，使用释放内存方法定期释放内存，建议不要将内存做为大量缓存的途径，进程无法共用内存可将缓存放在进程外如redis


## Buffer
Buffer的属性有
```
Buffer.__defineGetter__      Buffer.__defineSetter__
Buffer.__lookupGetter__      Buffer.__lookupSetter__
Buffer.__proto__             Buffer.constructor
Buffer.hasOwnProperty        Buffer.isPrototypeOf
Buffer.propertyIsEnumerable  Buffer.toLocaleString
Buffer.toString              Buffer.valueOf

Buffer.apply                 Buffer.arguments
Buffer.bind                  Buffer.call
Buffer.caller                Buffer.length
Buffer.name

Buffer.from                  Buffer.of
Buffer.prototype

Buffer.BYTES_PER_ELEMENT

Buffer.alloc                 Buffer.allocUnsafe
Buffer.allocUnsafeSlow       Buffer.byteLength
Buffer.compare               Buffer.concat
Buffer.isBuffer              Buffer.isEncoding
Buffer.poolSize
```
Buffer.prototype原型链上有
```
Buffer {
  asciiSlice: [Function: asciiSlice],
  base64Slice: [Function: base64Slice],
  binarySlice: [Function: binarySlice],
  hexSlice: [Function: hexSlice],
  ucs2Slice: [Function: ucs2Slice],
  utf8Slice: [Function: utf8Slice],
  asciiWrite: [Function: asciiWrite],
  base64Write: [Function: base64Write],
  binaryWrite: [Function: binaryWrite],
  hexWrite: [Function: hexWrite],
  ucs2Write: [Function: ucs2Write],
  utf8Write: [Function: utf8Write],
  copy: [Function: copy],
  swap16: [Function: swap16],
  swap32: [Function: swap32],
  parent: [Getter],
  offset: [Getter],
  toString: [Function],
  equals: [Function: equals],
  inspect: [Function: inspect],
  compare: [Function: compare],
  indexOf: [Function: indexOf],
  lastIndexOf: [Function: lastIndexOf],
  includes: [Function: includes],
  fill: [Function: fill],
  write: [Function],
  toJSON: [Function],
  slice: [Function: slice],
  readUIntLE: [Function],
  readUIntBE: [Function],
  readUInt8: [Function],
  readUInt16LE: [Function],
  readUInt16BE: [Function],
  readUInt32LE: [Function],
  readUInt32BE: [Function],
  readIntLE: [Function],
  readIntBE: [Function],
  readInt8: [Function],
  readInt16LE: [Function],
  readInt16BE: [Function],
  readInt32LE: [Function],
  readInt32BE: [Function],
  readFloatLE: [Function: readFloatLE],
  readFloatBE: [Function: readFloatBE],
  readDoubleLE: [Function: readDoubleLE],
  readDoubleBE: [Function: readDoubleBE],
  writeUIntLE: [Function],
  writeUIntBE: [Function],
  writeUInt8: [Function],
  writeUInt16LE: [Function],
  writeUInt16BE: [Function],
  writeUInt32LE: [Function],
  writeUInt32BE: [Function],
  writeIntLE: [Function],
  writeIntBE: [Function],
  writeInt8: [Function],
  writeInt16LE: [Function],
  writeInt16BE: [Function],
  writeInt32LE: [Function],
  writeInt32BE: [Function],
  writeFloatLE: [Function: writeFloatLE],
  writeFloatBE: [Function: writeFloatBE],
  writeDoubleLE: [Function: writeDoubleLE],
  writeDoubleBE: [Function: writeDoubleBE] }
```
每个汉字占三个buffer元素，每个元素是0-255的数值，用16进制表示，buffer支持编码类型ascii,utf-8,utf16le/ucs-2,base64,binary,hex

给某个元素赋值时，如果是小数小数部分会被舍弃，如果小于0会被不断加上256知道位于0-255为止，如果大于255会被不断减去256直到位于0-255为止

Buffer占的内存不在v8堆内存中，就是说不在process.memoryUsage().heapTotal里，而是由c++实现内存申请

`buffer.write(str,offset,length,encoding)` 操作buffer

`buffer.copy(buf,pos)` 将buffer复制到buf，buf下标pos处开始复制

`buffer.toString(encoding,start,end)` buffer转字符串

`isEncoding` 判断编码类型是否支持转换buffer

buffer拼接
```javascript
var chunks = [];
var count = 0;
respont.on('data',function(chunk){
  chunks.push(chunk);
  count += chunk.length;
})
respont.on('end',function(){
  var buffer = Buffer.concat(chunks,count);
  console.log(buffer);
})
```
Buffer.concat的机制
```javascript
Buffer.concat = function(array,len){
  if(!Array.isArray(array)){
    throw new Error('error');
    return false;
  }
  if(array.length === 0){
    return new Buffer(0);
  }
  if(array.length === 1){
    return array[0];
  }
  if(typeof len !== 'number'){
    len = 0;
    for(var i = 0; i < array.length; i++){
      len += array[i].length;
    }
  }
  var buffer = new Buffer(len);
  var pos = 0;
  for(var i = 0; i < array.length; i++){
    array[i].copy(buffer,pos);
    pos += array.length;
  }
  return buffer;
}
```
网络传输通过Buffer传输性能更加，测试ab -c 10 -n 10 http://www.baidu.com/
- stream
```javascript
var fs = require('fs');
fs.createReadStream(path,{
  flags:'r',
  encoding:null,
  fd:null,
  mode:0666,
  start:0,//开始读取行数
  end:10,//结束读取行数
  highWaterMark:10//限制读取Buffer的长度为10
})
```

`stream.setEncoding(encoding)` 设置可读流的编码，setEncoding内部由decoder实现
```javascript
var stringDecoder = require('string_decoder').stringDecoder;
var decoder = new stringDecoder('utf-8');
decoder.write(buffer);
//decoder会对宽字节截断问题做处理，被截断的宽字节会保留在decoder对象中，与下一段buffer拼接
```

## http

```javascript
var http = require('http');
http.createServer(function(req,res){
  res.writeHead(200,{'Content-Type':'text-plan'});
  res.end('hi');
}).listen(3000);
```
- request

http.request(options,connect) 生成一个请求，
`options = {
  host,//服务器域名
  port,
  hostname,
  localAddress,//建立连接的本地网卡
  socketPath,//Domain套接字路径
  method,
  path,
  headers,//请求头对象
  auth,//Basic认证，我也不知道是什么，
  agent,//代理配置
}

`

request.end() 本次请求发送结束

服务端事件

connection 建立连接时触发

request 解析http请求报文头后触发

server.close(callback) 当已有的连接都断开时触发

checkContinue 请求发送带有 Expect:100-continue时触发，与request事件互斥

connect 发起connect请求时触发，如果不监听事件，连接会被关闭

upgrade 请求头带有 Upgrade 字段时触发，如果不监听事件，连接会被关闭

clientError 连接的客户端发生错误时会传到服务端时触发

- response

response.setHead() 设置响应头

response.writeHead() 写入响应头

报头在报文体发送前发送出去的

response.write() 发送响应报文体

response.end() 告知服务器本次响应结束

客户端事件

response 得到响应时触发

socket 链接池建立的连接分配给当前请求时触发

connect客户端发起connect请求服务端返回200时触发

upgrade客户端发起upgrade请求服务端响应101时触发

continue 客户端发起带有Expect:100-continue头信息，试图发送大数据，如果服务器响应100 Continue时触发

- Agent

其实可以看做浏览器
`
new http.Agent({
  maxSocket:10//默认是5，false是请求不做并发管理限制
})
`
## websocket

```javascript

```

## 加密

node提供三个模块

crypto用于加密

tls 与net模块类似，但是建立在TLS/SSL加密的TCP连接上

https 与http模块一致，但是建立在加密的连接上

- TLS/SSL

TLS/SSL 公钥/私钥，服务端客户端都各自有自己的公私钥，公钥用于加密数据，私钥用于解密数据，建立安全连接前客户端服务端会交换公钥

生成私钥：openssl genrsa -out server.key 1024

生成公钥：openssl rsa -in server.key -pubout -out server.pem

在交换公钥的过程中有可能会被中间人窃取，从而伪装服务器，所以需要使用数字证书进行验证连接是来自于服务器而不是中间人

服务端将自己的私钥生成csr文件，CA机构拿到这个文件后给其颁发签名证书，当服务器和客户端交换公钥时，客户端会拿服务器的签名证书去CA机构验证服务器是否真的来自于真实服务器

CA机构可以是知名机构也可以是自己扮演颁发机构，知名机构一般在安装浏览器时其证书已预装了，自己扮演的机构则需要客户端自己获取

过程如下

生成私钥：openssl genrsa -out ca.key 1024

生成csr文件：openssl req  -new -key ca.key -out ca.csr

通过私钥生成证书：openssl x509 -req -in ca.csr -signkey ca.key -out ca.crt


- https


https是建立在TLS/SSL上的http

服务端 :
```javascript
var https = require('https');
var fs = require('fs');

var options = {
  key:fs.readFileSync('./server.key'),
  cert:fs.readFileSync('./server.crt')
}

https.createServer(options,function(req,res){
  res.writeHead(200);
  res.end('hi');
}).listen(3000);
```
客户端：
```javascript
var https = require('https');
var fs = require('fs');
var options = {
  hotsname,
  port,
  path,
  method,
  key:fs.readFileSync('./client.key'),客户端私钥
  cert:fs.readFileSync('./client.crt'),客户端里的签名证书
  ca:fs.readFileSync('./ca.crt'),//服务端证书
}

options.agent = new https.Agent(options);

var req = https.request(options,function(res){
  res.setEncoding('utf-8');
  res.on('data'.function(chunk){
    console.log(chunk);
  });
});
req.end();
```

## 开发web
- req

```javascript
var https = require('https');
var url = require('url');

https.createServer(function(req,res){
  console.log(url.parse(req.url,true));//打印请求url
  console.log(req.header);//打印请求header
}).listen(3000);

```

-cookie

path cookie存在该路径下，并浏览器会发送给服务端


expires/max-age 前者告诉浏览器什么时候过期，后者告诉浏览器多长时间后过期


httpOnly 告诉浏览器该cookie不能通过document.cookie获取，但是依然会发送给浏览器


secure 当值为true时，在http链接中无效也不回传给服务器，在https时才有效并且会传给服务器

缺点：每次请求都会携带并且子域也会携带，静态文件请求也会携带占用带宽

## session

- 通过cookie实现session
```javascript
var sessions = {};
var key = 'session_id';
var EXPIRES = 1000*60*60;

var generate = function(){
  var session = {};
  session.id = new Date().getTime()+Math.random();
  session.cookie = {
    expire:new Date().getTime() + EXPIRES
  }
  sessions[session.id] = session;
  return session;
}


function (req,res){
  var id = req.cookie.key;
  if(!id){
    req.session = generate();
    
  }else{
    var session = sessions[id];
    if(session){
      if(session.cookie.expire > new Date().getTime()){
        session.cookie.expire = new Date().getTime() + EXPIRES;
        req.session = session;
      }else{
        delete sessions[id];
        req.session = generate();
      }
    }else{
      req.session = generate();
    }
  }
}
```

- 通过url字符串实现session
```javascript
function (req,res){
  var redirect = function(url){
    res.setHeader('Location',url);
    res.writeHead(302);
    res.end();
  };
  var id = req.query[key];
  if(!id){
    var session = generate();
    redirect(req.url,seesion.id);
  }else{
    var session = sessions[id];
    if(session){
      if(session.cookie.expire > new Date().getTime()){
        session.cookie.expire = new Date().getTime() + EXPIRES;
        req.session = session;
      }else{
        delete sessions[id];
        req.session = generate();
        redirect(req.url,seesion.id);
      }
    }else{
      var session = generate();
      redirect(req.url,seesion.id);
    }
  }
}

```


- sessionid安全

sessionid会存在cookie或者url里，为了保证sessionid的安全

1.通过私钥加密sessionid

2.通过验证一些特有的信息，如果不是原始的客户端将签名失败


- xss

通过漏洞伪造脚本在用户浏览器中执行一些js脚本，获取用户的信息

解决：不要解析来自用户的输入，所有字符转义

- csrf

通过漏洞跨站伪造请求，使用户触发一些非本意行为

解决：在渲染页面是在前端输出token，提交数据是带上token，确保数据是来自于服务器输出的页面


- 浏览器缓存

1.响应设置Expires/Cache-Control头，告诉浏览器这个请求可以直接读取浏览器缓存，Expires是缓存到期时间，Cache-Control的max-age是还有多少时间到期,同时存在会覆盖Expires，Cache-Control可以设置public/private/no-cache/no-store

2.E-Tag 请求资源前，存在E-Tag的get请求会发起一个条件请求，询问服务器这个文件是否有变化，如果返回304则使用本地文件，如果有变化则直接将文件返回

## post


```javascript
function (req,res){
  //判断请求是否带内容
  return 'transfe-encoding' in req.headers || 'content-length' in req.headers;
}
```

- content-type

1.application/x-www-form-urlencoded 表示表单提交，提交数据早req.body

2.application/json 使用JSON.parse解析
```javascript
res.json = function(){
  res.setHeader('Content-Type','application/json');
  res.writeHead(200);
  res.end(JSON.stringify(json));
}
```

3.mutipart/form-data 特殊的表单提交，如文件的提交

4.text/plain 表示文本提交

- Content-Disposition

inline 用于查看内容

attachment 可作为附件，可下载
```javascript
function (req,res){
  res.sendfile = function (){
    fs.stat(filepath,function(error,stat){
      var stream = fs.createReadStream(filepath);
      res.setHeader('Content-Type',mime.lookup(filepath));
      res.setHeader('Content-Length',stat.size);
      res.setHeader('Content-Disposition','attachment;filename="a.js"');
      res.writeHead(200);
      stream.pipe(res);
    })
  }
}
```


## 中间件

- 跳转
res.redirect = function(){
  res.setHeader('Location',url);
  res.writeHead(302);
  res.end();
}

- 渲染
res.render = function(view,data){
  res.setHeader('Content-Type','text/html');
  res.write(200);
  var html = render(view,data);
  res.end(html);
}


## 进程

- spawn()
启动子进程来执行命令。
使用指定的命令行参数创建新进程。spawn 会返回一个带有stdout和stderr流的对象。你可以通过stdout流来读取子进程返回给node.js的数据。stdout拥有’data’,’end’以及一般流所具有的事件。当你想要子进程返回大量数据给Node时，比如说图像处理，读取二进制数据等等，你最好使用spawn方法。
```
var child_process = require('child_process');
var spawnObj = child_process.spawn('ping', ['127.0.0.1'], {encoding: 'utf-8'});
spawnObj.stdout.on('data', function(chunk) {
    console.log(chunk.toString());
});
spawnObj.stderr.on('data', (data) => {
  console.log(data);
});
spawnObj.on('close', function(code) {
    console.log('close code : ' + code);
}
spawnObj.on('exit', (code) => {
    console.log('exit code : ' + code);
    fs.close(fd, function(err) {
        if(err) {
            console.error(err);
        }
    });
});
```
- exec()
启动子进程来执行命令，但其回调函数能获知子进程的状况,可以指定超时时间，一旦超时子进程将被杀死。
使用子进程执行命令，缓存子进程的输出，并将子进程的输出以回调函数参数的形式一次性返回。exec方法会从子进程中返回一个完整的buffer。默认情况下，这个buffer的大小应该是200k。如果子进程返回的数据大小超过了200k，程序将会崩溃，同时显示错误信息“Error：maxBuffer exceeded”。你可以通过在exec的可选项中设置一个更大的buffer体积来解决这个问题，但是你不应该这样做，因为exec本来就不是用来返回很多数据的方法。
```
require('child_process').exec('dir', {encoding: ‘utf-8’}, function(err, stdout, stderr) {
    if (err) {
        console.log(error.stack);
        console.log('Error code: ' + error.code);
        console.log('Signal received: ' + error.signal);
    }
    //console.log(err, stdout, stderr);
    console.log('data : ' + stdout);
}).on('exit', function (code) {
    console.log('子进程已退出, 退出码 ' + code);
});
```
- execFile()
启动子进程执行可执行文件，首行须添加 #!/usr/bin/env node

- fork()
功能与spawn类似，但是其创建进程只需指定js文件模块。
用于在子进程中运行的模块，如 fork(‘./son.js’) 相当于 spawn(‘node’, [‘./son.js’]) 。与spawn方法不同的是，fork会在父进程与子进程之间，建立一个通信管道，用于进程之间的通信。
```
console.log('parent pid: ' + process.pid);
var fork = require('child_process').fork;
//fork方法返回的是子进程
var child = fork('./child.js');
console.log('fork return pid: ' + child.pid);
child.on('message', function(msg){
    console.log('parent get message: ' + JSON.stringify(msg));
});
child.send({key: 'parent value'});


console.log('child pid: ' + process.pid);
process.on('message', function(msg){
    console.log('child get message: ' + JSON.stringify(msg));
});
process.send({key: 'child value'});
```

- 主从通信
主从进程通过message接收消息和send()传递消息，kill()发送信号给子进程以做出约定行为
```javascript
var cp = require('child_process');
var child = cp.fork('child.js');
child.on('message',function(m){
  console.log(m);
});
child.send({msg:'hello'});

process.on('message',function(m){
  console.log(m);
});
process.send({msg:'hi'});
```

- 进程间通信
技术：管道，socket，共享内存，消息队列，Domain Socket等

- 进程事件
message 接受send发来的请求

error 子进程无法被复制创建杀死发送消息时触发

exit 子进程退出时触发，正常退出时第一个参数为退出码否则为null，如果被kill杀死第二个参数为进程信号

close 子进程的标准输入输出流中止时触发事件，参数与exit相同

disconnect 在父进程或子进程中调用disconnect()方法时触发，调用时关闭监听通道

uncaughtException 捕获异常时触发

## cluster
事件

fork 复制一个工作进程后触发

online 复制好一个工作进程后，工作进程主动发送online消息给主进程，主进程收到消息后触发

disconnect 主进程和工作进程通道断开后触发

exit 有工作进程退出时触发

setup cluster.setupMaster()执行后触发


## node调试

- nodemon

nodemon可以监听文件变化自动重起服务

安装`npm install nodemon -g`

执行`nodemon app.js`

执行命令`nodemon --exec npm start`

