> 原文地址：[https://hackernoon.com/javascript-async-await-the-good-part-pitfalls-and-how-to-use-9b759ca21cda](https://hackernoon.com/javascript-async-await-the-good-part-pitfalls-and-how-to-use-9b759ca21cda)  
作者：[Charlee Li](https://hackernoon.com/@charleeli)  
摘要：本文介绍了使用async/await后能够将异步操作以同步的代码风格表现出来，同时也有部分陷阱需要开发者注意。因此作者总结了自己理解的正确使用async/await的方式。

![封面图](http://ox34ivs2j.bkt.clouddn.com//hexo/2018-7-13/2018-7-13-1.png)

ES7为JS异步编程带来了一个梦幻特性：`async/await`。它能够在不阻塞主线程的情况下，让开发者使用同步编码风格异步获取网络资源。不过要完全掌握它是有一点技术门槛的。本文将带你从不同层面去探索async/await的特性，并且向你演示如何正确高效地使用它。

# async/await的优点
async/await带来的最大好处就是能让我们使用同步的代码风格。看看下面的例子吧。
```js
// async/await
async getBooksByAuthorWithAwait(authorId) {
  const books = await bookModel.fetchAll();
  return books.filter(b => b.authorId === authorId);
}
// promise
getBooksByAuthorWithPromise(authorId) {
  return bookModel.fetchAll()
    .then(books => books.filter(b => b.authorId === authorId));
}
```
显然，使用async/await写出的代码比使用promise写出的代码更易读。如果你忽略`await`关键字，那这些代码看上去就跟Python这些同步语言的代码一样了。

async/await的好处并不仅仅是优秀的可读性，还有原生浏览器对它的广泛支持。时至今日，所有主流浏览器都能全力支持async函数。

![浏览器支持情况](http://ox34ivs2j.bkt.clouddn.com//hexo/2018-7-13/2018-7-13-2.png)

原生浏览器的支持意味着开发者不需要刻意去转译ES7的代码。更重要的是，开发者能够很容易地调试代码。当你在函数入口打上断点，单步调试走到`await`那一行时，你会发现调试器在`bookModel.fetchAll()`执行完毕前停顿了一会儿，之后又自动走到`.filter`所在行。而如果使用的是promise，那么需要在`.filter`那一行额外添加一个断点才能实现这样的效果。

![调试async/await](http://ox34ivs2j.bkt.clouddn.com//hexo/2018-7-13/2018-7-13-3.gif)

async/await的另一个不太容易察觉到的好处就是关键字`async`。它保证函数`getBooksByAuthorWithAwait()`执行后的返回的结果肯定是一个promise类型。所以可以安全地调用`getBooksByAuthorWithAwait().then(...)`或是`await getBooksByAuthorWithAwait()`。现在，来看看下面这个反面教程：
```js
getBooksByAuthorWithPromise(authorId) {
  if (!authorId) {
    return null;
  }
  return bookModel.fetchAll()
    .then(books => books.filter(b => b.authorId === authorId));
}
```
在上面的代码中，`getBooksByAuthorWithPromise`也许会返回一个Promise，也许会返回一个null。返回null的时候，我们无法调用`.then()`方法。而如果使用`async`，就不需要担心这些了。

# async/await的误导性
有的文章会把async/await与Promise作比较，来证明它是下一代JS异步编程。这一点我并不赞同。async/await确实是JS语言的一大进步，但它的作用仅仅是一个语法糖，并不会从根本上改变我们的编程风格。

async声明的函数本质上也是Promise。因此在正确使用async函数前，你需要理解Promise的原理。此外，大部分时间里，promise和async函数都是联合使用的。

再看之前举的例子里，`getBooksByAuthorWithAwait()`和`getBooksByAuthorWithPromises()`这两个函数其实都提供了相同的接口。

这说明当你在直接调用`getBooksByAuthorWithAwait()`时它也会返回一个Promise。

好吧，这也不算是什么重大不足。但是，那种看到`await`就惊呼“太棒啦，我们可以把异步函数变成同步函数了”的行为确实是不正确的。

# 使用async/await时的注意点
接下来，我们看看使用async/await时常犯的一些错误吧。

## 避免串行思维
即使await使你的代码看上去是同步的，但仍要注意它们在运行时依然是异步执行的，并且要避免陷入串行同步的思维。

```js
async getBooksAndAuthor(authorId) {
  const books = await bookModel.fetchAll();
  const author = await authorModel.fetch(authorId);
  return {
    author,
    books: books.filter(book => book.authorId === authorId),
  };
}
```
上面的代码看上去逻辑并没有问题，但它犯了一个错误：
1. `await bookModel.fetchAll()`会一直等待`fetchAll()`的返回结果；
2. 接着`await authorModel.fetch(authorId)`才会被调用。

要注意的是，`authorModel.fetch(authorId)`并不依赖`bookModel.fetchAll()`的结果，实际上它们俩应该是并行的代码。然而在这里使用`await`让这两个方法只能串行执行，那么总体运行时间就比并行的版本要长的多。

下面是修正过的版本：
```js
async getBooksAndAuthor(authorId) {
  const bookPromise = bookModel.fetchAll();
  const authorPromise = authorModel.fetch(authorId);
  const book = await bookPromise;
  const author = await authorPromise;
  return {
    author,
    books: books.filter(book => book.authorId === authorId),
  };
}
```

如果是更复杂的情况，比如说你需要逐一获取一系列的数据，那就要用到Promise了。
```js
async getAuthors(authorIds) {
  // 错误的串行调用方式
  // const authors = _.map(
  //   authorIds,
  //   id => await authorModel.fetch(id));
  
  // 正确的方式
  const promises = _.map(authorIds, id => authorModel.fetch(id));
  const authors = await Promise.all(promises);
}
```

简而言之，你需要思考如何异步化工作流程，接着使用await去把代码同步化。对于复杂的业务逻辑，直接使用Promise会更简单。

# 错误处理
使用了promise的async函数会返回两种可能值：`resolved`类型的正确值或是`rejected`类型的错误值。对于正确值我们执行`.then()`方法，对于错误值我们执行`.catch()`方法。而如果使用的是`async/await`，在处理错误时需要一点技巧。

## try...catch
我推荐的标准处理方式是使用`try...catch`代码块。当在`await`后面调用一个函数时，任何`rejected`类型的值都被当做异常抛出。就像下面这个例子：
```js
class BookModel {
  fetchAll() {
    return new Promise((resolve, reject) => {
      window.setTimeout(() => { reject({'error': 400}) }, 1000);
    });
  }
}
// async/await
async getBooksByAuthorWithAwait(authorId) {
try {
  const books = await bookModel.fetchAll();
} catch (error) {
  console.log(error);    // { "error": 400 }
}
```
catch块里的`error`对象就是一个错误的返回值。当我们捕获到错误时，有下面几种方法来处理它：
1. 处理异常，接着返回一个正确值。（注意，在catch块中不显式使用return语句其实等价于`return undefined;`，而`undefined`也可以当做一个正确值）。
2. 抛出异常让外层调用者去处理。你可以直接执行`throw error;`来抛出异常，这样`async getBooksByAuthorWithAwait()`这个函数就可以在Promise链中使用。比如像`getBooksByAuthorWithAwait().then(...).catch(error => ...)`这样。你也可以用`ERROR`对象封装这个异常（``throw new Error(error)`）。这样在控制台中显示错误信息时会打印出完整的调用栈轨迹。
3. 通过`return Promise.reject(error)`返回一个rejected值。这其实跟`throw error`没什么区别，所以我不推荐。

那么，使用`try...catch`的好处有哪些呢？
1. 简单，传统。如果你使用过Java或是C++等其他语言，你可以很轻松地理解这种语句块。
2. 如果你有多个`await`语句，并且不希望对每一个语句单独进行异常处理。那么你可以用一个`try...catch`块把它们包裹起来，集中处理所有异常。

不过这种方法有一个小瑕疵：`try...catch`块能够捕获所有异常，包括那些不是因Promise返回rejected值导致的异常。
```js
class BookModel {
  fetchAll() {
    cb();    // note `cb` is undefined and will result an exception
    return fetch('/books');
  }
}
try {
  bookModel.fetchAll();
} catch(error) {
  console.log(error);  // This will print "cb is not defined"
}
```
运行上面的代码，你会在控制台中看到一个`ReferenceError: cb is not defined`错误。而这个错误的字体颜色是黑色，说明它是通过`console.log()`输出，而不是浏览器本身。这种情况有时候会导致一个问题，如果`BookModel`被藏在一系列函数调用的深处，然后其中一个函数吞下了这个异常，那么就很难找到这个异常产生的原因。

## 约束函数同时返回正确值与错误值
受到Go语言的启发，我们可以用另一种方法来处理异常。就是约束async函数同时返回正确结果与错误值。详情请看[这篇文章](https://blog.grossman.io/how-to-write-async-await-without-try-catch-blocks-in-javascript/)。

简单地说，就是应该使用下面这种代码格式：
```js
[err, user] = await to(UserModel.findById(1));
```
从我个人角度来看，我并不喜欢这种Go与JS混搭风格的代码，令人感到不自然。不过有时候它会对开发有所帮助吧。

## 使用.catch
最后一个方法就是继续使用Promise.catch()。

调用`await`右边的函数时，会等待它返回一个promise结果。而调用这个结果的`.catch`方法也会得到一个promise。所以我们可以像下面这样处理异常：
```js
// books === undefined if error happens,
// since nothing returned in the catch statement
let books = await bookModel.fetchAll()
  .catch((error) => { console.log(error); });
```
不过这个方法有2个问题：
1. 在使用这种promise和async混合的函数前，开发者需要理解promise是如何工作的。
2. 在正常逻辑之前处理异常不太符合正常思维。

# 总结
ES7带来的`async/await`确实是给JS异步编程带来了提升。它可以让代码变得通俗易懂，便于调试。但是，要正确使用它们必须先理解Pormise的原理。因为它们仅仅是一种语法糖，底层的实现仍然是基于Promise。

希望这篇文章能够加深你对`async/await`的理解，并且在以后的开发中避免犯下常见的错误。感谢你的阅读，如果你喜欢这篇文章欢迎点赞。

> 查看更多我翻译的Medium文章请访问：  
项目地址：[https://github.com/WhiteYin/translation](https://github.com/WhiteYin/translation/tree/master)  
SF专栏：[https://segmentfault.com/blog/yin-translation](https://segmentfault.com/blog/yin-translation)