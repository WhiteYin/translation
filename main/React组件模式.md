> 原文地址：[https://medium.com/teamsubchannel/react-component-patterns-e7fb75be7bb0](https://medium.com/teamsubchannel/react-component-patterns-e7fb75be7bb0)  
作者：[William Whatley](https://medium.com/@wwwhatley)  
摘要：本文介绍了4种组件类型：容器组件、展示组件、高阶组件和渲染回调。

![封面图](http://ox34ivs2j.bkt.clouddn.com//hexo/2018-7-15/2018-7-15-1.png)

今天，我想花一点时间来分享我学习React组件模式的经验。这个想法来源于一次聚会时的技术交流。组件是React的核心，因此有必要去理解如何使用它们。

以下的例子都脱胎于[Michael Chan gave on React component patterns](https://www.youtube.com/watch?v=YaZg8wg39QQ&t=662s)这个视频的思想。我非常推荐你们看一看。

# 什么是组件？
[reactjs.org](reactjs.org)上说：“组件能够将你的界面分割成独立且可复用的小块，而这些小组件都是互相独立的。”

当你第一次执行`npm install react`命令的时候，你得到了一个组件以及相关的API。与JS的函数类似，一个组件接收输入，叫做`props`。然后返回React元素，作用是描述UI界面的样式。这就是React作为声明式API的表现形式，因为你只需要告诉它你想要展示的UI，剩下的React都会帮你完成。

对于声明式API的概念，你可以想象成滴滴打车的场景——告诉司机你的目的地，接着让他来完成开车的工作。而命令式API不同，你将完成所有任务，既是乘客，也是司机。

# 组件API
当你在安装好React后，得到了5类API：  
![api类型](http://ox34ivs2j.bkt.clouddn.com//hexo/2018-7-15/2018-7-15-2.png)  

* render
* state
* props
* context
* lifecycle events

尽管写组件时可以把上面所有的API都使用一遍，但是你很快会发现一些组件只需要用到某些API，而另一些组件也只会使用另一些特定的API。而这两类组件往往被划分为有状态与无状态组件两大类。有状态的组件会有代表性地使用有状态API：render、state和生命周期。但无状态组件只会使用render、props和context。  
![有状态与无状态](http://ox34ivs2j.bkt.clouddn.com//hexo/2018-7-15/2018-7-15-3.png)

以上就是我们在介绍组件模式前所需要的知识铺垫。组件模式是设计组件的最佳实践，最初是把组件切割成数据/逻辑层和UI/展示层。通过拆分组件的职责，能够设计出更容易复用、更内聚的组件，从而组装成复杂的UI界面。这个特性对于构建可扩展的应用时是非常重要的。

# 组件模式
常用的组件模式有：
* 容器组件
* 展示组件
* 高阶组件HOC
* 渲染回调

## 容器组件
“容器组件的作用是获取数据和渲染子组件。”——Jason Bonta  
![容器组件](http://ox34ivs2j.bkt.clouddn.com//hexo/2018-7-15/2018-7-15-4.png)
> 蓝色代表容器组件，灰色代表展示用的子组件

容器组件使用了有状态的API，封装了数据逻辑层。通过使用生命周期，你可以连接Redux或Flux等状态管理库，然后把数据和回调函数当作props传递给子组件。在容器组件的render方法中，你可以用子展示组件来拼装UI界面。容器组件往往都设计成一个类组件，而不是纯函数组件，为的就是能够使用所有有状态的API。

在下面的例子中，我们有一个名为Greeting的有状态的类组件，包括`componentDidMount()`和`render`方法。
```
class Greeting extends React.Component {
  constructor() {
    super();
    this.state = {
      name: "",
    };
  }

  componentDidMount() {
    // AJAX
    this.setState(() => {
      return {
        name: "William",
      };
    });
  }

  render() {
    return (
      <div>
        <h1>Hello! {this.state.name}</h1>
      </div>
    );
  }
}
```
这时，这个组件仅仅是一个有状态的类组件。为了让它成为一个真正的容器组件，我们要把UI部分放进一个展示组件中。下面就来讲讲展示组件。

## 展示组件
展示组件使用到props、render和context这些无状态API，并且可以写成更简洁优雅的函数式无状态组件。
```
const GreetingCard = (props) => {
  return (
    <div>
      <h1>Hello! {props.name}</h1>
    </div>
  )
}
```
展示组件只能从父级组件或容器组件传来的props中接收数据和回调函数。
![展示组件](http://ox34ivs2j.bkt.clouddn.com//hexo/2018-7-15/2018-7-15-5.png)

> 蓝色代表展示组件，而灰色代表容器组件。

容器组件和展示组件合并起来后，封装成了一个真正被使用的组件：
```

const GreetingCard = (props) => {
  return (
    <div>
      <h1>{props.name}</h1>
    </div>
  )
}

class Greeting extends React.Component {
  constructor() {
    super();
    this.state = {
      name: "",
    };
  }

  componentDidMount() {
    // AJAX
    this.setState(() => {
      return {
        name: "William",
      };
    });
  }

  render() {
    return (
      <div>
       <GreetingCard name={this.state.name} />
      </div>
    );
  }
}
```
如你所见，我把Greeting类组件中的UI部分移除，放入一个无状态的函数组件中。当然，这只是一个简单的例子，但对于复杂的应用来说，这是最基本的做法。

## 高阶组件（HOC）
高阶组件就是一个把组件当作参数，并且返回新组件的函数。

它的强大之处在于能够给任意数量的组件提供数据源，并且可以被用来实现逻辑复用。用react-router-v4或Redux举个例子。使用react-router-v4时，你可以使用`withRouter()`来继承传给组件的props。而使用Redux时，你通过使用`connect({})()`来把actions当作props传递给子组件。
![HOC](http://ox34ivs2j.bkt.clouddn.com//hexo/2018-7-15/2018-7-15-6.png)

> 虚线表示的是高阶组件，它能够返回一个新的组件。

看看这个例子：
```
import {withRouter} from 'react-router-dom';

class App extends React.Component {
  constructor() {
    super();
    this.state = {path: ''}
  }
  
  componentDidMount() {
    let pathName = this.props.location.pathname;
    this.setState(() => {
      return {
        path: pathName,
      }
    })
  }
  
  render() {
    return (
      <div>
        <h1>Hi! I'm being rendered at: {this.state.path}</h1>
      </div>
    )
  }
}

export default withRouter(App);
```

当导出我的组件时，我使用react-router-v4提供的`withRouter()`方法把它包裹起来。而在App的`componentDidMount()`生命周期中，我通过`this.props.location.pathname`来更新状态。在被`withRouter()`包裹后，我的类组件现在可以通过props访问react-router-v4的方法，就像`this.props.location.pathname`。像这样的例子，实在是不胜枚举。

## 渲染回调（render callbacks）

与高阶组件类似，渲染回调或者说渲染props都是用来实现组件逻辑复用功能。相比较于大多数开发者使用的高阶组件，渲染回调也有自己的优势。具体优势可以阅读Michael Jackson所写的这个视频——[《Never write another HOC.》](https://www.youtube.com/watch?v=BcVAq3YFiuc)。视频中所讲的关键点就是渲染回调能够减少命名空间的冲突并且解释逻辑的来源。
![渲染回调](http://ox34ivs2j.bkt.clouddn.com//hexo/2018-7-15/2018-7-15-7.png)

> 蓝色虚线代表渲染回调函数。

```
class Counter extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      count: 0,
    };
  }

  increment = () => {
    this.setState(prevState => {
      return {
        count: prevState.count + 1,
      };
    });
  };

  render() {
    return (
      <div onClick={this.increment}>{this.props.children(this.state)}</div>
    );
  }
}

class App extends React.Component {
  render() {
    return (
      <Counter>
        {state => (
          <div>
            <h1>The count is: {state.count}</h1>
          </div>
        )}
      </Counter>
    );
  }
}
```

在上面的Counter类中，我在`render`里使用了`this.props.children`，然后把`this.state`当作参数传给这个函数。之后在App类中，我把想要展示的组件用Counter组件包裹起来，这样就能使用Counter的代码逻辑了。`render`函数的返回结果是代码28行，在那里我通过`{state => ()}`自动获取到Counter的state。

# 感谢您的阅读
我很乐意接受大家的意见来使我成长。我对React组件模式的见解还不够成熟，所以我也算是在写作中学习吧。

> 查看更多我翻译的Medium文章请访问：  
项目地址：[https://github.com/WhiteYin/translation](https://github.com/WhiteYin/translation/tree/master)  
SF专栏：[https://segmentfault.com/blog/yin-translation](https://segmentfault.com/blog/yin-translation)