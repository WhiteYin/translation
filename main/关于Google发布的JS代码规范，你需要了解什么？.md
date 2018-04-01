> 原文地址：[https://medium.freecodecamp.org/google-publishes-a-javascript-style-guide-here-are-some-key-lessons-1810b8ad050b](https://medium.freecodecamp.org/google-publishes-a-javascript-style-guide-here-are-some-key-lessons-1810b8ad050b)  
作者：[Daniel Simmons](https://medium.freecodecamp.org/@dsimmons_23530)  
摘要：本文总结了Google发布的JavaScript代码规范中比较重要的部分，适合想要养成良好代码规范的读者借鉴。

![google javascript style guide](http://ox34ivs2j.bkt.clouddn.com//hexo/2018-4-1/2018-04-01-1.jpeg)

Google为了那些还不熟悉代码规范的人发布了一个[JS代码规范](https://google.github.io/styleguide/jsguide.html)。其中列出了编写简洁易懂的代码所应该做的最佳实践。

代码规范并不是一种编写正确JavaScript代码的规则，而是为了保持源代码编写模式一致的一种选择。对于JavaScript语言尤其如此，因为它灵活并且约束较少，允许开发者使用许多不同的编码样式。

Google和Airbnb各自占据着当前最流行的编码规范的半壁江山。如果你会在编写JS代码上投入很长时间的话，我强烈推荐你通读一遍这两家公司的编码规范。

接下来要写的是我个人认为在Google的代码规范中，与日常开发密切相关的十三条规则。

它们处理的问题都非常具有争议性，包括tab与空格、是否强制使用分号等等。还有一些令我感到惊讶的规则，往往最后都改变了我编写JS代码的习惯。

对于每一条规则，我都会先给出规范的摘要，然后引用规范中的详细说明。我还会举一些适当的反例论证遵守这些规则的重要性。

# 使用空格代替tab
> 除了每一行的终止符序列，ASCII水平空格符（0x20）是唯一一个可以出现在源文件中任意位置的空格字符。这也意味着，tab字符不应该被使用，以及被用来控制缩进。

规范随后指出应该使用2个，而不是4个空格带实现缩进。
```js
// bad
function foo() {
∙∙∙∙let name;
}

// bad
function bar() {
∙let name;
}

// good
function baz() {
∙∙let name;
}
```
# 不能省略分号
> 每个语句必须以分号结尾。不允许依赖于JS自动添加分号的功能。

尽管我不明白为什么会有人反对这个规则，但目前分号的使用问题显然已经像“空格 vs tab”这个问题一样产生了巨大的争议。而Google对此表示分号是必须的，是不可省略的。
```js
// bad
let luke = {}
let leia = {}
[luke, leia].forEach(jedi => jedi.father = 'vader')
// good
let luke = {};
let leia = {};
[luke, leia].forEach((jedi) => {
  jedi.father = 'vader';
});
```
# 暂时不要使用ES6 module
> 由于ES6模块的语义尚不完全确定，所以暂时不要使用，比如export和import关键字。一旦它们的相关规范制定完成，那么请忽略这一条规则。

```js
// 暂时不要编写下面的代码：
//------ lib.js ------
export function square(x) {
    return x * x;
}
export function diag(x, y) {
    return sqrt(square(x) + square(y));
}

//------ main.js ------
import { square, diag } from 'lib';
```
> 译者注：感觉遵守这条规范不大现实，毕竟现在已经有babel了。而且使用React时，最佳实践就是使用ES6模块吧。

# 不推荐代码水平对齐
> Google的代码规范允许但不推荐对代码进行水平对齐。即使之前的代码中做了水平对齐的处理，以后也应该避免这种行为。

对代码进行水平对齐会在代码中添加若干多余的空格，这让相邻两行的字符看上去处于一条垂直线上。
```js
// bad
{
  tiny:   42,  
  longer: 435, 
};
// good
{
  tiny: 42, 
  longer: 435,
};
```
# 杜绝var
> 使用const或let来声明所有局部变量。如果变量不需要被重新赋值，默认应该使用const。应该拒绝使用关键字var。

我不知道是因为没有人能说服他们，还是说因为旧习难改。目前我仍能看到许多人在StackOverFlow或其他地方使用var声明变量。
```js
// bad
var example = 42;
// good
const example = 42;
```
# 优先使用箭头函数
> 箭头函数提供了一种简洁的语法，并且避免了一些关于this指向的问题。相比较与function关键字，开发者应该优先使用箭头函数来声明函数，尤其是声明嵌套函数。

坦白说，我曾以为箭头函数的作用只在于简洁美观。但现在我发现原来它们还有更重要的作用。
```js
// bad
[1, 2, 3].map(function (x) {
  const y = x + 1;
  return x * y;
});

// good
[1, 2, 3].map((x) => {
  const y = x + 1;
  return x * y;
});
```
# 使用模板字符串取代连接字符串
> 在处理多行字符串时，模板字符串比复杂的拼接字符串要表现的更出色。

```js
// bad
function sayHi(name) {
  return 'How are you, ' + name + '?';
}

// bad
function sayHi(name) {
  return ['How are you, ', name, '?'].join();
}

// bad
function sayHi(name) {
  return `How are you, ${ name }?`;
}

// good
function sayHi(name) {
  return `How are you, ${name}?`;
}
```
# 不要使用续行符分割长字符串
> 在JS中，`\`也代表着续行符。Google的代码规范不允许在不管是模板字符串还是普通字符串中使用续行符。尽管ES5中允许这么做，但如果在`\`后跟着某些结束空白符，这种行为会导致一些错误，而这些错误在审阅代码时很难注意到。

这条规则很有趣，因为Airbnb的规范中有一条与之不相同的[规则](https://github.com/airbnb/javascript#strings--line-length)  

Google推荐下面这样的写法，而Airbnb则认为应该顺其自然，不做特殊处理，该多长就多长。
```js
// bad (建议在PC端阅读)
const longString = 'This is a very long string that \
    far exceeds the 80 column limit. It unfortunately \
    contains long stretches of spaces due to how the \
    continued lines are indented.';
// good
const longString = 'This is a very long string that ' + 
    'far exceeds the 80 column limit. It does not contain ' + 
    'long stretches of spaces since the concatenated ' +
    'strings are cleaner.';
```
# 优先使用`for...of`
> 在ES6中，有3种不同的for循环。尽管每一种有它的应用场景，但Google仍推荐使用`for...of`。

真有趣，Google居然会特别指定一种for循环。虽然这很奇怪，但不影响我接受这一观点。

以前我认为`for...in`适合遍历Object，而`for...of`适合遍历数组。因为我喜欢这种各司其职的使用方式。

尽管Google的规范与这种使用方式相冲突，但Google对`for...of`的偏爱依然让我觉得十分有趣。

# 不要使用eval语句
> 除非是在code loader中，否则不用使用eval或是Function(...string)结构。这个功能具有潜在的危险性，并且在CSP环境中无法起作用。

MDN中有一节专门提到不要使用eval语句。
```js
// bad
let obj = { a: 20, b: 30 };
let propName = getPropName();  // returns "a" or "b"
eval( 'var result = obj.' + propName );
// good
let obj = { a: 20, b: 30 };
let propName = getPropName();  // returns "a" or "b"
let result = obj[ propName ];  //  obj[ "a" ] is the same as obj.a
```
# 常量的命名规范
> 常量命名应该使用全大写格式，并用下划线分割

如果你确定一定以及肯定一个变量值以后不会被修改，你可以将它的名称使用全大写模式改写，暗示这是一个常量，请不要修改它的值。

遵守这条规则时需要注意的一点是，如果这个常量是一个函数，那么应该使用驼峰式命名法。
```js
// bad
const number = 5;
// good
const NUMBER = 5;
```
# 每次只声明一个变量
> 每一个变量声明都应该只对应着一个变量。不应该出现像`let a = 1,b = 2;`这样的语句。
```js
// bad
let a = 1, b = 2, c = 3;
// good
let a = 1;
let b = 2;
let c = 3;
```
# 使用单引号
> 只允许使用单引号包裹普通字符串，禁止使用双引号。如果字符串中包含单引号字符，应该使用模板字符串。

```js
// bad
let directive = "No identification of self or mission."
// bad
let saying = 'Say it ain\u0027t so.';
// good
let directive = 'No identification of self or mission.';
// good
let saying = `Say it ain't so`;
```
# 总结
就像我在开头所说那样，规范中没有需要强制执行的命令。尽管Google是科技巨头之一，但这份代码规范也仅仅是用来当作参考罢了。

Google是一家人才汇聚的科技公司，雇佣着出色的程序员来编写优秀的代码。能够看到这样的公司发布的代码规范是一件很有趣的事情。

如果你想要实现一种Google式的代码，那么你可以在项目中制定这些规范。但你可能并不赞成这份代码规范，这时也没有人会阻拦你舍弃其中某些规则。

我个人认为在某些场景下，Airbnb的代码规范比Google的代码规范要出色。但不管你支持哪一种，也不管你编写的是什么类型的代码，最重要的是在脑海中时刻遵守着同一份代码规范。

> 查看更多我翻译的Medium文章请访问：  
项目地址：[https://github.com/WhiteYin/translation](https://github.com/WhiteYin/translation/tree/master)  
SF专栏：[https://segmentfault.com/blog/yin-translation](https://segmentfault.com/blog/yin-translation)