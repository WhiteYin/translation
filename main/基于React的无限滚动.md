> 原文地址：[https://blog.logrocket.com/infinite-scroll-techniques-in-react-adcfd7ff32bd](https://blog.logrocket.com/infinite-scroll-techniques-in-react-adcfd7ff32bd)  
作者：[Ogundipe Samuel](https://blog.logrocket.com/@ogundipesamuel)  
摘要：本文先讲了如何独立完成一个简单的无限滚动组件，之后讲了如何使用第三方库`React-infinite-scroller`来实现相同的功能，最后介绍了LogRocket插件，它可以帮助开发者远程复现bug。

![无限滚动](http://ox34ivs2j.bkt.clouddn.com//hexo/2018-4-19/2018-4-18-1.gif)

# 前言
无限滚动是目前Web网页广泛运用的一项技术，它让网页能够在用户滚动屏幕时动态加载内容，省去了分页的麻烦。

动态加载的内容往往是通过向服务器异步发送请求得到的。这样也能够极大地提升用户体验。

然而世上无绝对，凡事皆有例外。

无限滚动的实现原理是在window对象或滚动的div元素上绑定一个`scroll`事件处理，当滚动条滚动到元素的底部时触发事件，执行相应的动作。

我将讲述两种React中实现无限滚动的方法：
1. 第一种方法是不依赖于第三方库，从零开始实现一个组件；
2. 第二种方法是使用一个已有的无限滚动库。

**以下内容需要读者有一定的React的经验**

# 从零开始
前文已经说过，无限滚动就是绑定一个事件函数来监听滚动条是否已经滑动到指定元素的底部。

看看下面这个组件的render方法：
```js
render() {
    return (
      <div
        className="App"
        ref="myscroll"
        style={{ height: "420px", overflow: "auto" }}
      >
        <header className="App-header">
          <h1 className="App-title">Welcome to React</h1>
        </header>
        <ul>
        </ul>
      </div>
    );
}
```
先思考一下，每当滚动到`.App`这个div底部时，如何在`ul`标签内加载更多的`li`元素？

首先，我们在`.App`元素上加了一个`ref`属性，这能够帮助我们通过`this.refs.myscroll`来获取原生节点。

> 译者注：目前React 16文档上使用的是`React.createRef()`方法。

接下来，先在`constructor`函数中声明初始`state`：
```js
constructor(props) {
	super(props);
	this.state = {
	  items: 20,
	  loading: false
	};
}
```
现在你声明了两个状态`items`和`loading`。`items`表示当前显示的`li`元素数量，而`loading`是决定是否显示loading提示的布尔类型变量。

之后，声明一个渲染所有`li`元素的方法：
```js
showItems() {
	var items = [];
	for (var i = 0; i < this.state.items; i++) {
	  items.push(<li key={i}>Item {i}</li>);
	}
	return items;
}
```
这个方法对一个数组循环添加了数量为`this.state.items`变量值的`li`元素，并且返回这个数组。

现在，更新`render`方法中的代码来展示这些`li`元素：
```js
render() {
	return (
	  <div
	    className="App"
	    ref="myscroll"
	    style={{ height: "420px", overflow: "auto" }}
	  >
	    <header className="App-header">
	      <h1 className="App-title">Welcome to React</h1>
	    </header>
	    <ul>
	      {this.showItems()}
	    </ul>
	    {this.state.loading
	      ? <p className="App-intro">
		  loading ...
		</p>
	      : ""}
	  </div>
	);
}
```
好了，现在你的这个组件可以显示一组子项编号的`li`元素。那么怎么给这个组件添加无限滚动的功能呢？还记得`.App`元素的`ref`属性吗？它的值`myscroll`代表了`.App`元素。现在我们改写一下`ComponentWillMount`方法。
```js
componentDidMount() {
	// 检测是否滚动到底部
	this.refs.myscroll.addEventListener("scroll", () => {
	  if (
	    this.refs.myscroll.scrollTop + this.refs.myscroll.clientHeight >=
	    this.refs.myscroll.scrollHeight
	  ) {
	    this.loadMore();
	  }
	});
}
```
在上面的方法中，我们给`myscroll`添加了`scroll`事件监听函数，元素的`scrollTop`属性代表元素相对于窗口顶部的滚动距离，而`clientHeight`代表了元素的高度。

然后，检查这两者之和是否不小于整个滚动条的高度。如果结果是`true`，则说明已经滚动到底部。接着会调用`loadMore`这个方法。

下面是`loadMore`方法：
```js
loadMore() {
	this.setState({ loading: true });
	setTimeout(() => {
	 	this.setState({ items: this.state.items + 20, loading: false });
	}, 2000);
}
```
在这个方法中，先设置状态`loading`值为`true`，这样组件就会显示一个加载中的提示。然后延迟2s设置状态`loading`为`false`，并且在原有的`items`基础上加上20。

使用`setTimeout`方法的原因是为了人为增加一点延迟效果，因为在实际项目中，使用的是`fetch`或`anxio`方法来从后台获取数据并修改状态。

不管你用的是什么方法，原理都是一样的。

下面是组件的完全体：
```js
import React, { Component } from "react";
    import logo from "./logo.svg";
    import "./App.css";
    
    class App extends Component {
      constructor(props) {
        super(props);
        this.state = {
          items: 20,
          loading: false
        };
      }
      componentDidMount() {
        // Detect when scrolled to bottom.
        this.refs.myscroll.addEventListener("scroll", () => {
          if (
            this.refs.myscroll.scrollTop + this.refs.myscroll.clientHeight >=
            this.refs.myscroll.scrollHeight
          ) {
            this.loadMore();
          }
        });
      }
    
      showItems() {
        var items = [];
        for (var i = 0; i < this.state.items; i++) {
          items.push(<li key={i}>Item {i}</li>);
        }
        return items;
      }
    
      loadMore() {
        this.setState({ loading: true });
        setTimeout(() => {
          this.setState({ items: this.state.items + 20, loading: false });
        }, 2000);
      }
    
      render() {
        return (
          <div
            className="App"
            ref="myscroll"
            style={{ height: "420px", overflow: "auto" }}
          >
            <header className="App-header">
              <img src={logo} className="App-logo" alt="logo" />
              <h1 className="App-title">Welcome to React</h1>
            </header>
            <ul>
              {this.showItems()}
            </ul>
            {this.state.loading
              ? <p className="App-intro">
                  loading ...
                </p>
              : ""}
    
          </div>
        );
      }
    }
    
    export default App;
```

# 使用第三方库
尽管用上面的方法实现React无限滚动组件已经很简单了，但你可能根本就不想自己实现一个滚动事件监听函数。

也许你比较懒，又或者是时间太紧，反正你只想要一个即插即用的解决方案。

好吧，我满足你的要求。

[`React-infinite-scroller`](https://github.com/CassetteRocks/react-infinite-scroller)了解一下？下面是使用这个库的用例代码：
```js
import React, { Component } from "react";
import logo from "./logo.svg";
import "./App.css";

import InfiniteScroll from "react-infinite-scroller";

class Scroll2 extends Component {
  constructor(props) {
    super(props);
    this.state = {
      items: 20,
      hasMoreItems: true
    };
  }

  showItems() {
    var items = [];
    for (var i = 0; i < this.state.items; i++) {
      items.push(<li key={i}> Item {i} </li>);
    }
    return items;
  }

  loadMore() {
    if(this.state.items===200){
      
      this.setState({ hasMoreItems: false});
    }else{
        setTimeout(() => {
        this.setState({ items: this.state.items + 20});
    }, 2000);
    }
    
  }

  render() {
    return (
      <div className="App">
        <header className="App-header">
          <img src={logo} className="App-logo" alt="logo" />
          <h1 className="App-title"> Welcome to React </h1>{" "}
        </header>

        <div style={{height:'200px', overflow:'auto'}}>
          <InfiniteScroll
            loadMore={this.loadMore.bind(this)}
            hasMore={this.state.hasMoreItems}
            loader={<div className="loader"> Loading... </div>}
            useWindow={false}
          >
            {this.showItems()}{" "}
          </InfiniteScroll>{" "}
        </div>{" "}
      </div>
    );
  }
}

export default Scroll2;
```
上面的代码和第一种方法的代码很像吧？不过还是有以下几个不同的地方：
1. 没有显式地绑定事件监听；
2. 不需要给元素添加`ref`属性；
3. 使用`hasMoreItems`而不是`loading`来通知无限滚动组件是否需要加载更多元素；
4. 重写`loadMore`方法，当元素数量达到200时设置`hasMoreItems`为`false`。

# 总结
以上就是本文要讲的实现无限滚动的两种方法。

对于那些想要自己编写事件监听函数的人来说，这并不算难。

而那些不想要自己动手的人来说用第二种方法也能实现同样的效果。

欢迎在评论中留下您的意见和建议。

# 广告：LogRocket插件

![LogRocket](http://ox34ivs2j.bkt.clouddn.com//hexo/2018-4-19/2018-4-18-2.png)

LogRocket是一个前端日志记录工具，它能让你像在自己的浏览器中复现用户网页中的问题。它为你免去了猜测BUG产生的原因或是让用户截图、发送错误日志等麻烦，让你能够快速定位并修复错误。不管是什么框架，它都能完美运行，并且能够为Redux、Vuex和@ngrx/store提供插件记录额外的context。

除了记录React的action和state，LogRocket还记录控制台输出、JS报错、栈追踪、网络请求与响应头、浏览器的meta数据和自定义日志。同时还能测试DOM来记录HTML和CSS，像素级还原大部分复杂的单页应用。

> 查看更多我翻译的Medium文章请访问：  
项目地址：[https://github.com/WhiteYin/translation](https://github.com/WhiteYin/translation/tree/master)  
SF专栏：[https://segmentfault.com/blog/yin-translation](https://segmentfault.com/blog/yin-translation)