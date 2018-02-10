> 原文地址：[https://codeburst.io/writing-javascript-with-map-reduce-980602ff2f2f](https://codeburst.io/writing-javascript-with-map-reduce-980602ff2f2f)  
作者：[Shivek Khurana](https://codeburst.io/@shivekkhurana?source=post_header_lockup)  
简介：本文是一份编写优雅、简洁和函数式ES6代码的快捷清单。

现如今JavaScript有许多问题，但是词法并不是其中之一。不管是三元运算符，还是map/reduce等ES6方法，亦或是扩展运算符（...）都是非常强大的工具。  

除了能够保证可读性以及准确性，这些方法还有助于实现不可变性，因为这些方法会返回新的数据，而处理前的原始数据并不会被修改。这样的处理风格很适合redux以及[Fractal](https://hackernoon.com/fractal-a-react-app-structure-for-infinite-scale-4dab943092af)。

话不多说，让我们开始吧。

### 一个简单的reduce实践

当你想要将多个数据放进一个实例中时，你可以使用一个reducer。
```js
const posts = [
  {id: 1, upVotes: 2},
  {id: 2, upVotes: 89},
  {id: 3, upVotes: 1}
];
const totalUpvotes = posts.reduce((totalUpvotes, currentPost) =>     
  totalUpvotes + currentPost.upVotes, //reducer函数
  0 // 初始化投票数为0
);
console.log(totalUpvotes)//输出投票总数：92
```
传给reduce的第一个参数函数还可以增加2个参数：
1. 第三个参数：每个元素在原数据结构中的位置，比如数组下标。
2. 第四个参数：调用reduce方法的数据集合，比如例子中的posts。  
  所以，一个reducer的完全体应该是下面这样的：
```js
collection.reduce(
  (accumulator, currentElement, currentIndex, collectionCopy) => 
    {/*function body*/},
    initialAccumulatorValue
);
```
### 一个简单的map实践
map方法的作用在于处理流式数据，比如数组。我们可以把它想象成所有元素都要经过的一个转换器。
```js
const integers = [1, 2, 3, 4, 6, 7];
const twoXIntegers = integers.map(i => i*2);
// twoXIntegers现在是 [2, 4, 6, 8, 12, 14]，而integers不发生变化。
```
### 一个简单的find实践
find返回数组或类似结构中满足条件的第一个元素。
```js
const posts = [
  {id: 1, title: 'Title 1'},
  {id: 2, title: 'Title 2'}
];
// 找出id为1的posts
const title = posts.find(p => p.id === 1).title;
```
### 一个简单的filter实践
filter方法可以筛除数组和类似结构中不满足条件的元素，并返回满足条件的元素组成的数组。
```js
const integers = [1, 2, 3, 4, 6, 7];
const evenIntegers = integers.filter(i => i%2 === 0);
// evenIntegers的值为[2, 4, 6]
```
### 向数组中新增元素
如果你要创建一个无限滚动的ui组件（比如本文后面提到的例子），可以使用扩展运算符这个非常有用的词法。
```js
const books = ['Positioning by Trout', 'War by Green'];
const newBooks = [...books, 'HWFIF by Carnegie'];
// newBooks are now ['Positioning by Trout', 'War by Green', 'HWFIF // by Carnegie']
```
### 为一个数组创建视图
如果需要实现用户从购物车中删除物品，但是又不想破坏原来的购物车列表，可以使用filter方法。
```js
const myId = 6;
const userIds = [1, 5, 7, 3, 6];
const allButMe = userIds.filter(id => id !== myId);
// allButMe is [1, 5, 7, 3]
```
> 译者注：这里我猜测作者是不想修改原来的数组所以使用的filter，但是不能理解为什么要举购物车的例子。

### 向对象数组添加新元素
```js
const books = [];
const newBook = {title: 'Alice in wonderland', id: 1};
const updatedBooks = [...books, newBook];
//updatedBooks的值为[{title: 'Alice in wonderland', id: 1}]
```
books这个变量我们没有给出定义，但是不要紧，我们使用了扩展运算符，它并不会因此失效。

### 为对象新增一组键值对
```js
const user = {name: 'Shivek Khurana'};
const updatedUser = {...user, age: 23};
//updatedUser的值为：{name: 'Shivek Khurana', age: 23}
```
### 使用变量作为键名为对象添加键值对
```js
const dynamicKey = 'wearsSpectacles';
const user = {name: 'Shivek Khurana'};
const updatedUser = {...user, [dynamicKey]: true};
// updatedUser is {name: 'Shivek Khurana', wearsSpectacles: true}
```
### 修改数组中满足条件的元素对象
```js
const posts = [
  {id: 1, title: 'Title 1'},
  {id: 2, title: 'Title 2'}
];
const updatedPosts = posts.map(p => p.id !== 1 ?
  p : {...p, title: 'Updated Title 1'}
);
/*
updatedPosts is now 
[
  {id: 1, title: 'Updated Title 1'},
  {id: 2, title: 'Title 2'}
];
*/
```
### 找出数组中满足条件的元素
```js
const posts = [
  {id: 1, title: 'Title 1'},
  {id: 2, title: 'Title 2'}
];
const postInQuestion = posts.find(p => p.id === 2);
// postInQuestion now holds {id: 2, title: 'Title 2'}
```
> 译者注：奇怪啊，这不就是之前find的简单实践吗？
### 删除目标对象的一组属性
```js
const user = {name: 'Shivek Khurana', age: 23, password: 'SantaCl@use'};
const userWithoutPassword = Object.keys(user)
  .filter(key => key !== 'password')
  .map(key => {[key]: user[key]})
  .reduce((accumulator, current) => 
    ({...accumulator, ...current}),
    {}
  )
;
// userWithoutPassword becomes {name: 'Shivek Khurana', age: 23}
```
感谢[Kevin Bradley](https://medium.com/@kmb3398)提供了一个更优雅的方法:
```js
const user = {name: 'Shivek Khurana', age: 23, password: 'SantaCl@use'};
const userWithoutPassword = (({name, age}) => ({name, age}))(user);
```
他还表示这个方法在对象属性更少时也能发挥作用。
### 将对象转化成请求串
你也许几乎遇不到这个需求，但是有时候在别的地方会给你一点启发。
```js
const params = {color: 'red', minPrice: 8000, maxPrice: 10000};
const query = '?' + Object.keys(params)
  .map(k =>   
    encodeURIComponent(k) + '=' + encodeURIComponent(params[k])
  )
  .join('&')
;
// encodeURIComponent将对特殊字符进行编码。
// query is now "color=red&minPrice=8000&maxPrice=10000"
```
### 获取数组中某一对象的下标
```js
const posts = [
  {id: 13, title: 'Title 221'},
  {id: 5, title: 'Title 102'},
  {id: 131, title: 'Title 18'},
  {id: 55, title: 'Title 234'}
];
// 找到id为131的元素
const requiredIndex = posts.map(p => p.id).indexOf(131);
```
> 译者注：这里我觉得方法很繁琐。可以使用findIndex方法：`const requiredIndex = posts.findIndex(obj=>obj.id===131);`，同样能获取到下标值2。

### 作者总结
有了这些强大的方法，我希望你的代码会变得越来越稳定和一丝不苟。当你的团队中有一个新的开发者加入时，可以向他推荐这篇文章，让他了解以前不知道的秘密。

> 这个翻译项目才开始，以后会翻译越来越多的作品。我会努力坚持的。  
项目地址：[https://github.com/WhiteYin/translation](https://github.com/WhiteYin/translation/tree/master)