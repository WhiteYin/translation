> 原文地址：[https://medium.freecodecamp.org/how-to-make-responsiveness-super-simple-with-css-variables-8c90ebf80d7f](https://medium.freecodecamp.org/how-to-make-responsiveness-super-simple-with-css-variables-8c90ebf80d7f)  
作者：[Per Harald Borgen](https://medium.freecodecamp.org/@perborgen?source=post_header_lockup)  
摘要：这是一篇2018年制作响应性网页的快速教程。

![响应式布局](http://ox34ivs2j.bkt.clouddn.com//hexo/2018-3-4/css-variablegif.gif)  
如果你之前没有听说过CSS变量，那么现在我将告诉你：它是CSS的新特性，让你能够在样式表中使用变量的能力，并且无需任何配置。

实际上，CSS变量能够让你改变以往设置样式的老方法：
```css
h1{
    font-size:30px;
}
navbar > a {
    font-size:30px;
}
```
而使用了CSS变量之后：
```css
:root {
  --base-font-size: 30px;
}
h1 {
  font-size: var(--base-font-size);
}
navbar > a {
  font-size: var(--base-font-size);
}
```
这样的词法有点奇怪，但它确实能够让你通过仅改变`--base-font-size`的值来改变app中所有原生的字体大小。

如果你想要学习CSS变量的知识，可以登录Scrimba看[我的视频课程](https://scrimba.com/g/gcssvariables)，或是阅读我在Medium上写的文章：[如何学习CSS变量](https://medium.freecodecamp.org/want-to-learn-css-variables-heres-my-free-8-part-course-f2ff452e5140)。  

好了，现在让我们看看如何使用这个新知识来更加简单地制作响应式站点吧。

# 初始配置
让我们来把下面这个页面变成响应式的吧：
![页面](http://ox34ivs2j.bkt.clouddn.com//hexo/2018-3-4/normal.png)

这个页面在PC端看上去很不错，不过你可以看到它在移动端的表现并不好。就像下面这样：
![问题1](http://ox34ivs2j.bkt.clouddn.com//hexo/2018-3-4/responsive.png)

在下面这张图中，我们在样式上做了一些改进，让它看起来更好一点：
1. 重新排列整个网格布局，使用垂直排列取代固定两列布局。
2. 将框架整体上移了一点。
3. 对字体进行了缩放。

![问题2](http://ox34ivs2j.bkt.clouddn.com//hexo/2018-3-4/real-responsive.png)  

目光转到CSS代码中，下面是我们要修改的代码：
```css
h1 {
  font-size: 30px;
}
#navbar {
  margin: 30px 0;
}
#navbar a {
  font-size: 30px;
}
.grid {
  margin: 30px 0;
  grid-template-columns: 200px 200px;
}
```

更具体地说，我们需要在一个媒体查询中做出以下调整：
* 将h1的字体调整为20px；
* 减少#navbar的上外边距为15px；
* 将#navbar的字体大小减少到20px；
* 减少.grid的外边距为15px；
* 将.grid从两列布局变为单列布局。

> 注意：样式表里不仅仅是这些CSS声明，但是在这篇教程中我跳过它们，因为媒体查询并不影响它们的设置。你可以在[这里](https://scrimba.com/c/cwJmLhn)获取完整的代码。

# 旧方法
不使用CSS变量确实可以做到同样的效果，但这样会增加许多不必要的代码，因为上面大部分修改都需要将声明在媒体查询中重写一遍。就像下面这样：
```css
@media all and (max-width: 450px) {
  
  navbar {
    margin: 15px 0;
  }
  
  navbar a {
    font-size: 20px;
  }

  h1 {
    font-size: 20px;
  }
  
  .grid {
    margin: 15px 0;
    grid-template-columns: 200px;
  }
}
```
# 新的方法
现在让我们看看使用CSS变量是如何起作用的。首先，我们要声明需要更改或复用的变量：
```css
:root {
  --base-font-size: 30px;
  --columns: 200px 200px;
  --base-margin: 30px;
}
```
然后，我们只需要在app中使用它们就可以了。非常简单：
```css
#navbar {
  margin: var(--base-margin) 0;
}
#navbar a {
  font-size: var(--base-font-size);
}
h1 {
  font-size: var(--base-font-size);
}
.grid {
  margin: var(--base-margin) 0;
  grid-template-columns: var(--columns);
}
```
之后，我们可以在媒体查询中修改这些变量值：
```css
@media all and (max-width: 450px) {
  :root {
    --columns: 200px;
    --base-margin: 15px;
    --base-font-size: 20px;
}
```
这样的代码是不是比之前要简洁多了？我们只需要专注于`:root`选择器就可以了。

我们将媒体查询中的4个声明减少到了1个，代码也从13行减少到了4行。

当然，这只是一个简单的例子。想象一下，在一个大中型网站中，有一个`--base-margin`变量控制着所有的外边距。当你想要在媒体查询时修改属性，并不需要用复杂的声明填充整个媒体查询，只是简简单单地修改这个变量值就可以了。

总之，CSS变量可以定义为未来的响应式。如果你想要学习更多的知识，我推荐你看[我的免费教程](https://scrimba.com/g/gcssvariables)。用不了多久你就能成为一个CSS变量大师。