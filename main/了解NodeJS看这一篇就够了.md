> 原文地址：[https://codeburst.io/the-only-nodejs-introduction-youll-ever-need-d969a47ef219](https://codeburst.io/the-only-nodejs-introduction-youll-ever-need-d969a47ef219)  
作者：[vick_onrails](https://codeburst.io/@vick_onrails)  
摘要：这篇文章适合对Node一无所知或了解不多的初学者阅读。全面但不深入地讲了包括http模块、express、mongodb和RESTful API等知识点。

如果你是前端开发工作者，那么对你来说，基于NodeJS编写web程序已经不是什么新闻了。而不管是NodeJS还是web程序都非常依赖JavaScript这门语言。

首先，我们要认识到一点：Node并不是银弹。也就是说，它不是所有项目的最佳解决方案。任何人都可以基于Node创建一个服务器，但是这需要你对编写web程序的语言具有一定程序的有很深入的理解。

最近，我从学习Node的过程中发现了许多乐趣，同时我也意识到我已经掌握了一定的知识，应该分享出来，并且从社区获得反馈来提升自己。

那么就让我们开始吧。

# Node.js出现之前
Web应用往往基于客户端/服务器模式，当客户端向服务器请求资源时，服务器会响应这个请求并且返回相应的资源。服务器只会在接收到客户端请求时才会做出响应，同时会在响应结束后关闭与客户端的连接。

这种设计模式需要考虑到效率问题，因为每一个请求都需要处理时间和资源。因此，服务器在每一次处理请求的资源后应该关闭这个连接，以便于响应其他请求。

如果同时有成千上万个请求同时发往服务器，服务器会变成什么样子呢？当你问出这个问题时，你一定不想看到一个请求必须等待其他请求被响应后才能轮到他的情形，因为这段延迟实在是太长了。

想象一下，当你想要打开FaceBook，但因为在你之前已经有上千人向服务器发出过请求，所以你需要等待5分钟才能看到内容。有没有一种解决方案来同时处理成百上千个请求呢？所幸我们有线程这个工具。

线程是系统能够并行处理多任务所使用的方式。每一个发给服务器的请求都会开启一个新的线程，而每个线程会获取它运行代码所需要的一切。

这听上去很奇怪？让我们来看看这个例子：

> 想象餐馆里只有一个厨师提供食物，当食物需求越来越多，事情也会变得越来越糟。在之前的所有订单都被处理前，人们不得不等待很长时间。而我们能想到的方法就是增加更多的服务员来解决这个问题，对吧？这样能够同时应付更多的顾客。

每一个线程都是一个新的服务员，而顾客就是浏览器。我想理解这一点对你来说并不困难。

但是这种系统有一个副作用，让请求数达到一定数量时，过多的线程会占用所有系统内存和资源。重新回到我们的例子里，雇佣越来越多的人来供应食物必然会提高人力成本和占用更多的厨房空间。

当然，如果服务器在响应完客户端的请求后立刻切断连接并释放所有资源，这对我们来说自然是极好的。

多线程系统擅长于处理CPU密集型操作，因为这些操作需要处理大量的逻辑，而且计算这些逻辑会花费更多的时间。如果每一个请求都会被一个新的线程处理，那么主线程可以被解放出来去处理一些重要的计算，这样也能让整个系统变得更快。

> 译者注：经[@代码宇宙](https://segmentfault.com/u/universe_of_code)提醒，应该是多进程系统。由于原文使用的是thread，所以翻译成线程。下面的内容读者请自动脑补。  

让主线程不必忙于所有的运算操作是一种提高效率的好办法，但是能不能在此之上更进一步呢？

# NodeJS来了
想象一下我们现在已经有了一个多线程服务器，运行于Ruby on rails环境。我们需要它读取文件并且发送给请求这个文件的浏览器。首先要知道的是Ruby并不会直接读取文件，而是通知文件系统去读取指定文件并返回它内容。顾名思义，文件系统就是计算机上一个专门用来存取文件的程序。

Ruby在向文件系统发出通知后会一直等待它完成读取文件的操作，而不是转头去处理其他任务。当文件系统处理任务完成后，Ruby才会重新启动去收集文件内容并且发送给浏览器。

这种方式很显然会造成阻塞的情况，而NodeJS的诞生就是为了解决这个痛点。如果我们使用Node来向文件系统发出通知，在文件系统去读取文件的这段时间里，Node会去处理其他请求。而读取文件的任务完成后，文件系统会通知Node去读取资源然后将它返回给浏览器。事实上，这里的内部实现都是依赖于Node的事件循环。

> Node的核心就是JavaScript和事件循环。

![事件循环](http://ox34ivs2j.bkt.clouddn.com/event_loop.png)

简单地说，事件循环就是一个等待事件然后在需要事件发生时去触发它们的程序。此外还有一点很重要，就是Node和JavaScript一样都是单线程的。

还记得我们举过的餐厅例子吗？不管顾客数量有多少，Node开的餐厅里永远只有一个厨师烹饪食物。

与其他语言不同，NodeJS不需要为每一个请求开启一个新的线程，它会接收所有请求，然后将大部分任务委托给其他的系统。`Libuv`就是一个依赖于OS内核去高效处理这些任务的库。当这些隐藏于幕后的工作者处理完委托给它们的事件后，它们会触发绑定在这些事件上的回调函数去通知NodeJS。

这儿我们接触到了回调这个概念。回调理解起来并不困难，它是被其他函数当作参数传递的函数，并且在某种特定情况下会被调用。

NodeJS开发者们做的最多的就是编写事件处理函数，而这些处理函数会在特定的NodeJS事件发生后被调用。

NodeJS虽然是单线程，但它比多线程系统要快得多。这是因为程序往往并不是只有耗时巨长的数学运算和逻辑处理，大部分时间里它们只是写入文件、处理网络请求或是向控制台和外部设备申请权限。这些都是NodeJS擅长处理的问题：当NodeJS在处理这些事情时，它会迅速将这些事件委托给专门的系统，转而去处理下一个事件。

如果你继续深入下去，你也许会意识到NodeJS并不擅长处理消耗CPU的操作。因为CPU密集型操作会占用大量的主线程资源。对于单线程系统来说，最理想的情况就是避免这些操作来释放主线程去处理别的事情。

还有一个关键点是在JavaScript中，只有你写的代码不是并发执行的。也就是说，你的代码每次只能处理一件事，而其他工作者，比如文件系统可以并行处理它们手头的工作。

如果你还不能理解的话，可以看看下面的例子：
> 很久以前有一个国王，他有一千个官员。国王写了一个任务清单让官员去做，清单非常非常非常长。有一个宰相，根据清单将任务委托给其他所有官员。每完成一项任务他就将结果报告给国王，之后国王又会给他另一份清单。因为在官员工作的时候，国王也在忙于写其他清单。

这个例子要讲的是即使有很多官员在并行处理任务，国王每次也只能做一件事。这里，国王就是你的代码，而官员就是藏于NodeJS幕后的系统工作者。所以说，除了你的代码，每件事都是并行发生的。

好了，让我们继续这段NodeJS之旅吧。

# 用NodeJS写一个web应用
用NodeJS写一个web应用相当于编写事件回调。让我们来看看下面的例子：
1. 新建并进入一个文件夹
2. 执行`npm init`命令，一直回车直到你在文件夹根目录下创建了一个package.json文件。
3. 新建一个名为server.js的文件，复制并粘贴下面的代码：
    ```
    //server.js
    const http = require('http'),
          server = http.createServer();
    
    server.on('request',(request,response)=>{
       response.writeHead(200,{'Content-Type':'text/plain'});
       response.write('Hello world');
       response.end();
    });
    
    server.listen(3000,()=>{
      console.log('Node server created at port 3000');
    });
    ```
4. 在命令行中，输入`node server.js`，你会看到下面的输出：
    ```
    node server.js
    //Node server started at port 3000
    ``` 
  打开浏览器并且进入`localhost:3000`，你应该能够看到一个`Hello world`信息。
  
首先，我们引入了http模块。这个模块提供了处理htpp操作的接口，我们调用`createServer()`方法来创建一个服务器。

之后，我们为request事件绑定了一个事件回调，传递给on方法的第二个参数。这个回调函数有2个参数对象，request代表接收到的请求，response代表响应的数据。

不仅仅是处理request事件，我们也可以让Node去做其他事情。

```
//server.js
const http = require('http'),
server = http.createServer((request,response)=>{
    response.writeHead(200,{'Content-Type':'text/plain'});
    response.write('Hello world');
    response.end();
});
server.listen(3000,()=>{
    console.log('Node server created at port 3000');
});
```
在当面的代码里，我们传给createServer()一个回调函数，Node把它绑定在request事件上。这样我们只需要关心request和response对象了。

我们使用`response.writeHead()`来设置返回报文头部字段，比如状态码和内容类型。而`response.write()`是对web页面进行写入操作。最后使用`response.end()`来结束这个响应。

最后，我们告知服务器去监听3000端口，这样我们可以在本地开发时查看我们web应用的一个demo。listen这个方法要求第二个参数是一个回调函数，服务器一启动，这个回调函数就会被执行。

# 习惯回调
Node是一个单线程事件驱动的运行环境，也就是说，在Node里，任何事都是对事件的响应。

前文的例子可以改写成下面这样：
```js
//server.js
const http = require('http'),
      
makeServer = function (request,response){
   response.writeHead(200,{'Content-Type':'text/plain'});
   response.write('Hello world');
   response.end();
},
      
server = http.createServer(makeServer);

server.listen(3000,()=>{
  console.log('Node server created at port 3000');
```
`makeServer`是一个回调函数，由于JavaScript把函数当作一等公民，所以他们可以被传给任何变量或是函数。如果你还不了解JavaScript，你应该花点时间去了解什么是事件驱动程序。

当你开始编写一些重要的JavaScript代码时，你可能会遇到“回调地狱”。你的代码变得难以阅读因为大量的函数交织在一起，错综复杂。这时你想要找到一种更先进、有效的方法来取代回调。看看Promise吧，[Eric Elliott ](https://medium.com/@_ericelliott)写了一篇文章来[讲解什么是Promise](https://medium.com/javascript-scene/master-the-javascript-interview-what-is-a-promise-27fc71e77261)，这是一个好的入门教程。

# NodeJS路由
一个服务器会存储大量的文件。当浏览器发送请求时，会告知服务器他们需要的文件，而服务器会将相应的文件返回给客户端。这就叫做路由。

在NodeJS中，我们需要手动定义自己的路由。这并不麻烦，看看下面这个基本的例子：
```js
//server.js
const http = require('http'),
      url = require('url'),
 
makeServer = function (request,response){
   let path = url.parse(request.url).pathname;
   console.log(path);
   if(path === '/'){
      response.writeHead(200,{'Content-Type':'text/plain'});
      response.write('Hello world');
   }
   else if(path === '/about'){
     response.writeHead(200,{'Content-Type':'text/plain'});
     response.write('About page');
   }
   else if(path === '/blog'){
     response.writeHead(200,{'Content-Type':'text/plain'});
     response.write('Blog page');
   }
   else{
     response.writeHead(404,{'Content-Type':'text/plain'});
     response.write('Error page');
   }
   response.end();
 },
server = http.createServer(makeServer);
server.listen(3000,()=>{
 console.log('Node server created at port 3000');
});
```
粘贴这段代码，输入`node server.js`命令来运行。在浏览器中打开`localhost:3000`和`localhost:3000/abou`，然后在试试打开`localhost:3000/somethingelse`，是不是跳转到了我们的错误页面？

虽然这样满足了我们启动服务器的基本要求，但是要为服务器上每一个网页都写一遍代码实在是太疯狂了。事实上没有人会这么做，这个例子只是让你了解路由是怎么工作的。

如果你有注意到，我们引入了url这个模块，它能让我们处理url更加方便。

为parse()方法传入一个url字符串参数，这个方法会将url拆分成`protocol`、`host`、`path`和`querystring`等部分。如果你不太了解这些单词，可以看看下面这张图：

![url](http://ox34ivs2j.bkt.clouddn.com/url.png)

所以当我们执行`url.parse(request.url).pathname`语句时，我们得到一个url路径名，或者是url本身。这些都是我们用来进行路由请求的必要条件。不过这件事还有个更简单的方法。

# 使用Express进行路由
如果你之前做过功课，你一定听说过Express。这是一个用来构建web应用以及API的NodeJS框架，它也可以用来编写NodeJS应用。接着往下看，你会明白为什么我说它让一切变得更简单。

在你的终端或是命令行中，进入电脑的根目录，输入`npm install express --save`来安装Express模块包。要在项目中使用Express，我们需要引入它。
```
const express = require('express');
```
欢呼吧，生活将变得更美好。

现在，让我们用express进行基本的路由。
```js
//server.js
const express = require('express'),
      server = express();

server.set('port', process.env.PORT || 3000);

//Basic routes
server.get('/', (request,response)=>{
   response.send('Home page');
});

server.get('/about',(request,response)=>{
   response.send('About page');
});

//Express error handling middleware
server.use((request,response)=>{
   response.type('text/plain');
   response.status(505);
   response.send('Error page');
});

//Binding to a port
server.listen(3000, ()=>{
  console.log('Express server started at port 3000');
});
```
> 译者注：这里不是很理解为什么代码中错误状态码是505。

现在的代码是不是看上去更加清晰了？我相信你很容易就能理解它。

首先，当我们引入express模块后，得到的是一个函数。调用这个函数后就可以开始启动我们的服务器了。

接下来，我们用`server.set()`来设置监听端口。而`process.env.PORT`是程序运行时的环境所设置的。如果没有这个设置，我们默认它的值是3000.

然后，观察上面的代码，你会发现Express里的路由都遵循一个格式：
```js
server.VERB('route',callback);
```
这里的VERB可以是GET、POST等动作，而pathname是跟在域名后的字符串。同时，callback是我们希望接收到一个请求后触发的函数。

最后我们再调用`server.listen()`，还记得它的作用吧？

以上就是Node程序里的路由，下面我们来挖掘一下Node如何调用数据库。

# NodeJS里的数据库
很多人喜欢用JavaScript来做所有事。刚好有一些数据库满足这个需求，比如MongoDB、CouchDB等待。这些数据库都是NoSQL数据库。

一个NoSQL数据库以键值对的形式作为数据结构，它以文档为基础，数据都不以表格形式保存。

我们来可以看看MongoDB这个NoSQL数据库。如果你使用过MySQL、SQLserver等关系型数据库，你应该熟悉数据库、表格、行和列等概念。 MongoDB与他们相比并没有特别大的区别，不过还是来比较一下吧。

> 译者注：这儿应该有个表格显示MongoDB与MySQL的区别，但是原文里没有显示。

为了让数据更加有组织性，在向MongoDB插入数据之前，我们可以使用Mongoose来检查数据类型和为文档添加验证规则。它看上去就像Mongo与Node之间的中介人。

由于本文篇幅较长，为了保证每一节都尽可能的简短，请你先阅读官方的[MongoDB安装教程](https://docs.mongodb.com/manual/installation/)。

此外，[Chris Sevilleja](https://medium.com/@sevilayha)写了一篇[Easily Develop Node.js and MongoDB Apps with Mongoose](http://easily%20develop%20node.js%20and%20mongodb%20apps%20with%20mongoose/)，我认为这是一篇适合入门的基础教程。

# 使用Node和Express编写RESTful API
API是应用程序向别的程序发送数据的通道。你有没有登陆过某些需要你使用facebook账号登录的网页？facebook将某些函数公开给这些网站使用，这些就是API。

一个RESTful API应该不以服务器/客户端的状态改变而改变。通过使用一个REST接口，不同的客户端，即使它们的状态各不相同，但是在访问相同的REST终端时，应该做出同一种动作，并且接收到相同的数据。

API终端是API里返回数据的一个函数。

编写一个RESTful API涉及到使用JSON或是XML格式传输数据。让我们在NodeJS里试试吧。我们接下来会写一个API，它会在客户端通过AJAX发起请求后返回一个假的JSON数据。这不是一个理想的API，但是能帮助我们理解在Node环境中它是怎么工作的。
1. 创建一个叫node-api的文件夹；
2. 通过命令行进入这个文件夹，输入`npm init`。这会创建一个收集依赖的文件；
3. 输入`npm install --save express`来安装express；
4. 在根目录新建3个文件：`server.js`，`index.html`和`users.js`；
5. 复制下面的代码到相应的文件：
```js
//users.js
module.exports.users = [
 {
  name: 'Mark',
  age : 19,
  occupation: 'Lawyer',
  married : true,
  children : ['John','Edson','ruby']
 },
  
 {
  name: 'Richard',
  age : 27,
  occupation: 'Pilot',
  married : false,
  children : ['Abel']
 },
  
 {
  name: 'Levine',
  age : 34,
  occupation: 'Singer',
  married : false,
  children : ['John','Promise']
 },
  
 {
  name: 'Endurance',
  age : 45,
  occupation: 'Business man',
  married : true,
  children : ['Mary']
 },
]
```
这是我们传给别的应用的数据，我们导出这份数据让所有程序都可以使用。也就是说，我们将users这个数组保存在`modules.exports`对象中。
```js
//server.js
const express = require('express'),
      server = express(),
      users = require('./users');

//setting the port.
server.set('port', process.env.PORT || 3000);

//Adding routes
server.get('/',(request,response)=>{
 response.sendFile(__dirname + '/index.html');
});

server.get('/users',(request,response)=>{
 response.json(users);
});

//Binding to localhost://3000
server.listen(3000,()=>{
 console.log('Express server started at port 3000');
});
```
我们执行`require('express')`语句然后使用`express()`创建了一个服务变量。如果你仔细看，你还会发现我们引入了别的东西，那就是`users.js`。还记得我们把数据放在哪了吗？要想程序工作，它是必不可少的。

express有许多方法帮助我们给浏览器传输特定类型的内容。`response.sendFile()`会查找文件并且发送给服务器。我们使用`__dirname`来获取服务器运行的根目录路径，然后我们把字符串`index.js`加在路径后面保证我们能够定位到正确的文件。

`response.json()`向网页发送JSON格式内容。我们把要分享的users数组传给它当参数。剩下的代码我想你在之前的文章中已经很熟悉了。

```html
//index.html

<!DOCTYPE html>
<html>
<head>
 <meta charset="utf-8">
 <title>Home page</title>
</head>
<body>
 <button>Get data</button>
<script src="https://code.jquery.com/jquery-3.2.1.min.js" integrity="sha256-hwg4gsxgFZhOsEEamdOYGBf13FyQuiTwlAQgxVSNgt4=" crossorigin="anonymous"></script>
 
  <script type="text/javascript">
  
    const btn = document.querySelector('button');
    btn.addEventListener('click',getData);
    function getData(e){
        $.ajax({
        url : '/users',
        method : 'GET',
        success : function(data){
           console.log(data);
        },
      
        error: function(err){
          console.log('Failed');
        }
   });
  } 
 </script>
</body>
</html>
```
在文件夹根目录中执行`node server.js`，现在打开你的浏览器访问`localhost:3000`，按下按钮并且打开你的浏览器控制台。

![浏览器控制台](http://ox34ivs2j.bkt.clouddn.com/open_console.gif)

在`btn.addEventListent('click',getData);`这行代码里，getData通过AJAX发出一个GET请求，它使用了`$.ajax({properties})`函数来设置`url`，`success`和`error`等参数。

在实际生产环境中，你要做的不仅仅是读取JSON文件。你可能还想对数据进行增删改查等操作。express框架会将这些操作与特定的http动词绑定，比如POST、GET、PUT和DELETE等关键字。

要想深入了解使用express如何编写API，你可以去阅读[Chris Sevilleja](https://medium.com/@sevilayha)写的[Build a RESTful API with Express 4](https://scotch.io/tutorials/build-a-restful-api-using-node-and-express-4)。

# 使用socket进行网络连接
计算机网络是计算机之间分享接收数据的连接。要在NodeJS中进行连网操作，我们需要引入`net`模块。
```
const net = require('net');
```
在TCP中必须有两个终端，一个终端与指定端口绑定，而另一个则需要访问这个指定端口。

如果你还有疑惑，可以看看这个例子：

> 以你的手机为例，一旦你买了一张sim卡，你就和sim的电话号码绑定。当你的朋友想要打电话给你时，他们必须拨打这个号码。这样你就相当于一个TCP终端，而你的朋友是另一个终端。

现在你明白了吧？

为了更好地吸收这部分知识，我们来写一个程序，它能够监听文件并且当文件被更改后会通知连接到它的客户端。

1. 创建一个文件夹，命名为`node-network`；
2. 创建3个文件：`filewatcher.js`、`subject.txt`和`client.js`。把下面的代码复制进`filewatcher.js`。
    ```
    //filewatcher.js

    const net = require('net'),
       fs = require('fs'),
       filename = process.argv[2],
          
    server = net.createServer((connection)=>{
     console.log('Subscriber connected');
     connection.write(`watching ${filename} for changes`);
      
    let watcher = fs.watch(filename,(err,data)=>{
      connection.write(`${filename} has changed`);
     });
      
    connection.on('close',()=>{
      console.log('Subscriber disconnected');
      watcher.close();
     });
      
    });
    server.listen(3000,()=>console.log('listening for subscribers'));
    ```
3. 接下来我们提供一个被监听的文件，在`subject.txt`写下下面一段话：
    ```
    Hello world, I'm gonna change
    ```
4. 然后，新建一个客户端。下面的代码复制到`client.js`。
    ```
    const net = require('net');
    let client = net.connect({port:3000});
    client.on('data',(data)=>{
     console.log(data.toString());
    });
    ```
5. 最后，我们还需要两个终端。第一个终端里我们运行`filename.js`，后面跟着我们要监听的文件名。
    ```
    //subject.txt会保存在filename变量中
    node filewatcher.js subject.txt
    //监听订阅者
    ```
在另一个终端，也就是客户端，我们运行`client.js`。
    ```
    node client.js
    ```
现在，修改`subject.txt`，然后看看客户端的命令行，注意到多出了一条额外信息：
```
//subject.txt has changed.
```
网络的一个主要的特征就是许多客户端都可以同时接入这个网络。打开另一个命令行窗口，输入`node client.js`来启动另一个客户端，然后再修改`subject.txt`文件。看看输出了什么？

# 我们做了什么？
如果你没有理解，不要担心，让我们重新过一遍。

我们的`filewatcher.js`做了三件事：
1. `net.createServer()`创建一个服务器并向许多客户端发送信息。
2. 通知服务器有客户端连接，并且告知客户端有一个文件被监听。
3. 最后，使用wactherbianl监听文件，并且当客户端端口连接时关闭它。

  再来看一次`filewatcher.js`。
```js
//filewatcher.js

const net = require('net'),
   fs = require('fs'),
   filename = process.argv[2],
      
server = net.createServer((connection)=>{
 console.log('Subscriber connected');
 connection.write(`watching ${filename} for changes`);
  
let watcher = fs.watch(filename,(err,data)=>{
  connection.write(`${filename} has changed`);
 });
  
connection.on('close',()=>{
  console.log('Subscriber disconnected');
  watcher.close();
 });
  
});
server.listen(3000,()=>console.log('listening for subscribers'));
```
我们引入两个模块：fs和net来读写文件和执行网络连接。你有注意到`process.argv[2]`吗？process是一个全局变量，提供NodeJS代码运行的重要信息。`argv[]`是一个参数数组，当我们获取`argv[2]`时，希望得到运行代码的第三个参数。还记得在命令行中，我们曾输入文件名作为第三个参数吗？
```
node filewatcher.js subject.txt
```
此外，我们还看到一些非常熟悉的代码，比如`net.createServer()`，这个函数会接收一个回调函数，它在客户端连接到端口时触发。这个回调函数只接收一个用来与客户端交互的对象参数。

`connection.write()`方法向任何连接到3000端口的客户端发送数据。这样，我们的`connetion`对象开始工作，通知客户端有一个文件正在被监听。

wactcher包含一个方法，它会在文件被修改后发送信息给客户端。而且在客户端断开连接后，触发了close事件，然后事件处理函数会向服务器发送信息让它关闭watcher停止监听。

```js
//client.js
const net = require('net'),
      client = net.connect({port:3000});
client.on('data',(data)=>{
  console.log(data.toString());
});
```

`client.js`很简单，我们引入net模块并且调用connect方法去访问3000端口，然后监听每一个data事件并打印出数据。

当我们的`filewatcher.js`每执行一次`connection.write()`，我们的客户端就会触发一次data事件。

以上只是网络如何工作的一点皮毛。主要就是一个端点广播信息时会触发所有连接到这个端点的客户端上的data事件。

如果你想要了解更多Node的网络知识，可以看看官方NodeJS的文档：[net模块](https://nodejs.org/api/net.html)。你也许还需要阅读[Building a Tcp service using Node](https://blog.yld.io/2016/02/23/building-a-tcp-service-using-node-js/#.Wmcc6qinG00)。

# 作者总结
好了，这就是我要讲的全部。如果你想要使用NodeJS来编写web应用程序，你要知道的不仅仅是编写一个服务器和使用express进行路由。

下面是我推荐的一些书：  
[NodeJs the right way](https://www.amazon.com/Node-js-Right-Way-Server-Side-JavaScript/dp/1937785734)   
[Web Development With Node and Express](https://www.amazon.com/Web-Development-Node-Express-Leveraging/dp/1491949309)

如果你还有什么见解，可以在下面发表评论。


> 译者注：这个翻译项目才开始，以后会翻译越来越多的作品。我会努力坚持的。  
项目地址：[https://github.com/WhiteYin/translation](https://github.com/WhiteYin/translation/tree/master)