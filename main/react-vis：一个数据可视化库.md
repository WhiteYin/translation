> 原文地址：[https://medium.com/dailyjs/data-visualization-with-react-vis-bd2587fe1660](https://medium.com/dailyjs/data-visualization-with-react-vis-bd2587fe1660)  
作者：[Shyianovska Nataliia](https://medium.com/@shyianovska)  
摘要：react-vis是Uber公司开源的数据可视化库，能够制作折线图、饼状图等常用图表。本文将简单介绍如何使用它。

![react-vis](http://ox34ivs2j.bkt.clouddn.com//hexo/2018-3-18/2018-3-18-1.png)  

## 什么是react-vis？

[React-vis](http://uber.github.io/react-vis/documentation/welcome-to-react-vis)是一个React框架下的可视化库。它的开源者是Uber公司。你可以用它轻松地创建折线图、饼状图、柱状图和树形图等常见图表。

之所以推荐React-vis是因为它有以下三个优点：
* 简单
* 灵活
* 整合了React

在这篇文章中，我想要向你展示如何使用react-vis来创建一个折线图。

## 安装
首先，你需要在你的项目中安装react-vis。我使用`create-react-app`来创建demo项目。

输入`npm install react-vis --save-dev`来安装react-vis。

## 示例
假设现在你希望可视化一些数据。在我的demo中，我使用的是Github统计每种语言有多少个pull request得到的[数据集](https://nataliia-radina.github.io/react-vis-example/)。

接着，我在componentDidMount方法中接收数据，然后给我的应用设置state并将它当作props传给子组件。由于我只对JavaScript的数据感兴趣，所以我还对结果进行了过滤。

```js
import React, {Component} from 'react';
import './App.css';
import Chart from './components/chart'

const API_URL = "https://nataliia-radina.github.io/react-vis-example/";

class App extends Component {
    constructor(props) {
        super(props)
        this.state = {
            results: [],
        };
    }

    componentDidMount() {
        fetch(API_URL)
            .then(response => {
                if (response.ok) {
                    return  response.json()
                }
                else {
                    throw new Error ('something went wrong')
                }
            })
            .then(response => this.setState({
                results: response.results.filter((r)=>{
                        return r.name === 'JavaScript';
                    })
                })
            )}

    render() {
        const {results} = this.state;

        return (
            <div className="App">
                <Chart data={results}/>
            </div>
        );
    }
}

export default App;
```
现在让我们看看Chart组件。

Chart组件是一个无状态组件。在我的图表中，我希望展示特定时间段的pull request数量。这也是我要使用折线图的原因。

为了生成这个图表，我需要从react-vis中引入相关的组件。
```
import {XYPlot, XAxis, YAxis, VerticalGridLines, LineSeries} from 'react-vis';
```
XYPlot是其他组件的容器组件，XAxis和YAxis是用来显示X、Y坐标轴，VerticalGridLines是用来生成网格，而LineSeries则是图表的种类。

## 简单场景
> 译者注：以下场景都需要在index.html中写上
```
<link rel="stylesheet" href="https://unpkg.com/react-vis/dist/style.css">
```

现在，让我们先用随机数据生成一个Chart组件，只是用来看看react-vis是如何工作的。

```js
import React from 'react';
import {XYPlot, XAxis, YAxis, VerticalGridLines, HorizontalGridLines, LineSeries} from 'react-vis';

const Chart = (props) => {

        return (
            <XYPlot
                width={300}
                height={300}>
                <VerticalGridLines />
                <HorizontalGridLines />
                <XAxis />
                <YAxis />
                <LineSeries
                    data={[
                        {x: 1, y: 4},
                        {x: 5, y: 2},
                        {x: 15, y: 6}
                    ]}/>
            </XYPlot>
        );
}
export default Chart;
```
你可以看到我传递了一个数组给LineSeries组件，里面每一项都是一个由点的x、y坐标组成的对象。

接下来就是见证奇迹的时刻！我的Chart组件在浏览器中就像下面一样：  
![simple chart](http://ox34ivs2j.bkt.clouddn.com//hexo/2018-3-18/2018-3-18-2.png)

## 应用真实数据
现在，让我们把真实数据传递给Chart组件。我希望能够显示特定时间段的pull request数量。在我的数据集中代表了“count”、“year”和“quarter”等字段。所谓我将用这些数据组合而成的点的坐标来生成一个数组。
```js
const dataArr = props.data.map((d)=> {
    return {x: d.year + '/' + d.quarter, 
    y: parseFloat(d.count/1000)}
});
```
让我们看看如果将这个数组传递给LineSeries组件会发生什么吧。
```
<LineSeries data={dataArr}/>
```
因为我想在x轴上显示嫉妒，所以我需要设置坐标轴的类型如下：
```
xType="ordinal"
```
看上去还不错，不过我还希望美化一下我的图表，所以我加了一些样式在上面：
```
<LineSeries data={dataArr} style={{stroke: 'violet', strokeWidth: 3}}/>
```

下面是Chart组件的全部代码：
```
import React from 'react';
import {XYPlot, XAxis, YAxis, VerticalGridLines, HorizontalGridLines, LineSeries} from 'react-vis';

const Chart = (props) => {

    const dataArr = props.data.map((d)=> {
        return {x: d.year + '/' + d.quarter, 
        y: parseFloat(d.count/1000)}
    });

    return (
        <XYPlot
            xType="ordinal"
            width={1000}
            height={500}>
            <VerticalGridLines />
            <HorizontalGridLines />
            <XAxis title="Period of time(year and quarter)" />
            <YAxis title="Number of pull requests (thousands)" />
                <LineSeries
                    data={dataArr}
                    style={{stroke: 'violet', strokeWidth: 3}}/>
        </XYPlot>
    );
}

export default Chart;
```
以及最后的效果图：  
![last sample](http://ox34ivs2j.bkt.clouddn.com//hexo/2018-3-18/2018-3-18-3.png)  

## 总结
我希望你现在能够理解react-vis是多么简单但又非常强大的工具。它能够帮助你展示任何类型的数据。

如果你想要学习使用react-vis，可以查看它们的[文档](https://uber.github.io/react-vis/?p=/documentation/welcome-to-react-vis)和示例。

好好享受你的数据可视化之旅吧。

> 本文作者的demo地址：[https://github.com/Nataliia-Radina/react-vis-example](https://github.com/Nataliia-Radina/react-vis-example)  
查看更多我翻译的Medium文章请访问：  
项目地址：[https://github.com/WhiteYin/translation](https://github.com/WhiteYin/translation/tree/master)  
SF专栏：[https://segmentfault.com/blog/yin-translation](https://segmentfault.com/blog/yin-translation)