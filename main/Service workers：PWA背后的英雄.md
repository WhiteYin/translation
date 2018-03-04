> 原文地址：[https://medium.freecodecamp.org/service-workers-the-little-heroes-behind-progressive-web-apps-431cc22d0f16](https://medium.freecodecamp.org/service-workers-the-little-heroes-behind-progressive-web-apps-431cc22d0f16)  
作者：[Flavio Copes](https://medium.freecodecamp.org/@writesoftware)  
摘要：这篇文章简述service worker作为PWA核心技术如何实现资源缓存和消息推送的功能，还帮助读者理解service worker的生命周期。

Service worker是渐进式网络应用（Progressive Web Apps）的核心。它们帮助我们实现原本是原生app才有资源缓存和消息推送两大特性。

Service worker是你的网页与网络间的代理，它能够拦截和缓存来往的网络请求。这可以帮助你的应用创造一个离线环境下也能良好访问的用户体验。

首先介绍一下web worker的概念。它是一个与指定网页相关联的JS文件，独立与主线程运行在一个特定的上下文环境中，这样就不会为了计算数据去牺牲UI的性能，从而避免了阻塞的情况。而service worker是一种特殊的web worker。

而正是由于它是一个子线程，所以无法操作DOM。同样也无法访问Local Storage API和XHR API。它只能通过Channel Messaging API和主线程通信。

Service Worker能够与下面几个API合作：
* Promises
* Fetch API
* Cache API  

只有在HTTPS协议下的网页里它们才会起作用。（不过不包括本地的网络请求，因为它们不需要保持安全连接。这样也方便我们调试。）

# 后台进程
Service worker能够独立于与它关联的应用程序运行，并且在这些程序处于非活跃状态下仍可以接收消息。

让我来举几个场景：
* app处于后台非活跃状态下运行；
* app被关闭；
* 呈现你网页的浏览器被关闭；  

那么service worker将不受影响地继续工作。

Service worker的有用之处在于：
* 它们可以当作缓冲层，处理网络请求和缓存离线所需的资源；
* 它们可以用来推送消息。  

Service worker只在需要的时候运行，其他情况下都会停止工作。

# 支持离线
对于传统网页，离线情况下的用户体验非常糟糕。如果用户没有联网，移动端的web应用一般是直接停止工作。反观原生应用，会展示给用户一些友好的提示信息。

下面这张图是Chrome浏览器中离线网页显示的内容，显然这并不算是一个友好的提示信息：  
![chrome离线](http://ox34ivs2j.bkt.clouddn.com//hexo/2018-2-25/offline_chrome.png)

也许唯一不错的地方是你可以通过点击恐龙来免费玩一个游戏，不过相信你很快就会变得不耐烦了。  
![无聊的游戏](http://ox34ivs2j.bkt.clouddn.com//hexo/2018-2-25/free_game.gif)

不久之前，HTML5标准下的AppCache可以让web应用缓存离线资源，但是它缺乏灵活性并且有一些令人困惑的行为，这也说明它无法胜任支持离线这项工作。  

而现在，service worker成了离线缓存的新标准。

那么，它实现哪些缓存呢？

## 安装时的预缓存
像图片、CSS文件和JS文件都会在app的使用过程中重复用到。这些资源可以在app打开的第一时间缓存好。

这也是所谓的APP壳架构（ App Shell architecture）的基础。

## 缓存网络请求
我们使用Fetch API可以对服务器返回的响应报文进行编辑，根据服务器是否可达来决定是否使用缓存中的响应报文代替。

# 生命周期
一个service worker在启动前经历了三步：
* 注册（Registration)
* 安装（Installation）
* 激活（Activation）

## 注册
注册阶段是通知浏览器service worker的存在，并且在后台开始安装。

下面是写在`worker.js`中注册一个service worker的代码：
```js
if ('serviceWorker' in navigator) { 
  window.addEventListener('load', () => {   
    navigator.serviceWorker.register('/worker.js') 
    .then((registration) => { 
      console.log('Service Worker registration completed with scope: ', registration.scope) 
    }, (err) => { 
      console.log('Service Worker registration failed', err)
    })
  })
} else { 
  console.log('Service Workers not supported') 
}
```
无论这段代码被调用多少次，浏览器始终只会在service worker之前没有注册过或是需要更新的情况下进入注册阶段。

### Scope
register函数需要一个scope参数来指明你的web应用被该service worker管理的文件所在路径。  

这个参数的默认值是所有文件以及service worker文件父级目录下的所有子文件夹。所以如果你把service worker文件放在根目录下，它会管理整个web应用。而如果在某个子文件夹中，它只会管理该路径能够访问的网页。

下面这个例子通过指定scope参数为`/notifications/`目录来注册一个service worker。
```js
navigator.serviceWorker.register('/worker.js', { 
  scope: '/notifications/' 
})
```
结尾的`/`非常重要，可以避免`/notification`页面触发service worker。而如果写成下面这样：
```js
{ scope: '/notifications' }
```
那么service worker就将同样作用于`/notification`页面。

*注意*：service worker无法控制自身所在目录以外的文件。也就是说，如果service worker文件被放在`/notification`文件夹下，它无法控制根目录`/`或其他不属于`/notification`的文件。

## 安装
如果浏览器发现一个service worker过期或之前没有注册过，那么它将安装这个service worker。 
```js
self.addEventListener('install', (event) => { 
  //... 
});
```
这是使用service worker初始化缓存，然后利用Cache API来缓存APP shell和静态资源的好时机。

## 激活
一旦service worker注册并安装成功后 ，我们来到了第三阶段：激活。

这时，service worker能够在加载新页面时开始工作。  

它不能作用于激活前已经加载过的页面，所以只有重启app或是刷新已加载页面两种方式来使它工作。
```js
self.addEventListener('activate', (event) => { 
  //... 
});
```
监听这个事件可以用来清除旧缓存或者是删除新版service worker不需要的旧资源。

# 更新
你仅仅是修改一字节的文件就需要更新一次service worker。它会在注册的代码再次执行时被更新。

更新后的service worker只有在所有页面都被关闭后才开始代替之前的service worker工作。如果仅仅是刷新页面是不会起作用的，因为之前的service worker仍然在运行且没有被删除。  

这种机制保证了更新不会让之前在运行的app或网页崩溃。

# Fetch事件
当浏览器发送网络请求时就会触发Fetch事件。  

我们借此可以在请求发送时检查缓存中是否已经存储所需资源。

举个例子，下面的代码使用了Cache API来检查请求的URL是否已经被缓存。如果是，那么返回缓存中的响应数据，否则会发送请求然后返回响应数据。
```js
self.addEventListener('fetch', (event) => {
  event.respondWith( 
    caches.match(event.request) 
      .then((response) => { 
        if (response) { 
          //entry found in cache 
          return response 
        } 
        return fetch(event.request) 
      } 
    ) 
  ) 
})
```

# Background Sync
当用户在离线状态下发送网络请求时，Background Sync这个API将延迟该请求直到用户脱离离线状态。

这保证了用户在离线状态下仍然可以使用并操作app，这些离线操作会保存在队列中，以便在连接网络后向服务端发出响应请求。（是不是比展示一个无休止的loading图标要好多了？）
```js
navigator.serviceWorker.ready.then((swRegistration) => { 
    //注册一个事件event1
  return swRegistration.sync.register('event1') 
});
```
下面的代码是在service worker中监听这个事件：
```js
self.addEventListener('sync', (event) => { 
  if (event.tag == 'event1') { 
    event.waitUntil(doSomething()) 
  } 
})
```
`doSomething()`返回一个promise。如果返回的promise被拒，另一个同步事件被自动地开始重试操作，直到返回一个成功状态的promise。

这也使得app能够在联网时立刻更新服务器发来的数据。

# 消息推送
Service worker使得web应用可以像原生一样推送消息给用户。

事实上，推送和消息通知是两个不同的概念，它们组合而成的技术才是我们熟悉的消息推送。推送机制使得服务器能够向service worker发送信息，然后service worker将信息展示给用户才是消息通知。

由于service worker可以在app关闭后继续运行，所以它们能够一直监听推送事件。然后它们可以发送消息通知，或者是更新app的状态。

推送事件会在后端通过浏览器推送服务后启动，比如说Firebase的推送服务。

下面的代码演示了web worker如何监听push事件：
```js
self.addEventListener('push', (event) => { 
  console.log('Received a push event', event) 
  const options = { 
    title: 'I got a message for you!', 
    body: 'Here is the body of the message', 
    icon: '/img/icon-192x192.png', 
    tag: 'tag-for-this-notification', 
  } 
  event.waitUntil( 
    self.registration.showNotification(title, options) 
  ) 
})
```

# 关于console.log
如果你的代码中包含console.log或是其他类似的控制台输出语句，务必打开Chrome开发工具中的`Preserve  log`功能。

否则的话，因为service worker在网页加载前就开始执行，而此时控制台会被清空，所以你无法看到任何日志输出。  

# 作者总结
非常感谢阅读本篇教程，事实上，关于PWA还有很多要学习的知识。如果您有什么见解欢迎在下面评论。
> 译者注：主流浏览器开始逐渐支持service worker，以后PWA是否会真的与原生平分秋色呢？未来如何，我想现在多了解一点PWA的知识总不会坏事。  
项目地址：[https://github.com/WhiteYin/translation](https://github.com/WhiteYin/translation/tree/master)