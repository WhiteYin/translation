> 原文地址：[https://engineering.innovid.com/code-splitting-using-lazy-loading-with-react-redux-typescript-and-webpack-4-3ec60140ec5a](https://engineering.innovid.com/code-splitting-using-lazy-loading-with-react-redux-typescript-and-webpack-4-3ec60140ec5a)  
作者：[Aviv Shafir](https://engineering.innovid.com/@avivshafir)  
摘要：Innovid网站使用Webpack4对一个React项目进行了优化改造。主要使用了新的optimization配置和动态注入功能。

Hey，这里是[Innovid](http://www.innovid.com/)，一个领先的视频广告平台。我们每天处理130万小时的视频，而在我们的web项目中，经常会使用到Webpack。我们非常喜欢这个工具。

最近，我们将一个项目迁移到了最新的Webpack4。它给我们带来了一些开箱即用的新特性，比如在构建时间上进行了非常大的优化。

在本次迁移中，我们决定使用懒加载这一Webpack最吸引人的特性来分割app中的主要代码部分。

> 代码分割能够帮助你延迟加载用户当前需要的内容，同时也能显著地提升用户体验。尽管你没有减少app的总代码量，但你已经避免加载一些用户也许永远也用不到的代码了。而且还能够在初始加载时减少加载的代码数量。  
——[React 文档](https://reactjs.org/docs/code-splitting.html)

Webpack根据你的应用程序构建了一个依赖关系图。从你的入口文件开始，它递归遍历所有文件和它们的依赖文件，使用loader和plugin对你的文件施了点魔法，最后就输出了提供给用户的生成包。

我们现在将生成包分为app.js（我们的应用代码）和vendors.js(第三方库)。  
我们使用webpack-bundle-analyzer插件来可视化两个生成包：  
![初始包](http://ox34ivs2j.bkt.clouddn.com//hexo/2018-3-11/2018-3-11-1.png)
> app.js大小116KB，vendors.js大小399KB

# Webpack配置
app.js是我们程序的入口，所以自动打包成app.js。而第三方包vendors.js是使用了新的`optimization`配置，将从`node_modules`文件夹中引入的所有文件打包生成的。
```js
mode: "production",
  entry: {
    app: path.join(__dirname, "index.tsx"),
  },
  output: {
    path: path.resolve(__dirname, "public/dist"),
    publicPath: "",
    chunkFilename: "[name].js",
    filename: "[name].js"
  },
  optimization: {
        runtimeChunk: {
            name: "manifest"
        },
        splitChunks: {
            cacheGroups: {
                vendor: {
                    test: /[\\/]node_modules[\\/]/,
                    name: "vendors",
                    priority: -20,
                    chunks: "all"
                }
            }
        }
   }
```

*注意：* 在Webpack4中，我们不再使用`CommonChunkPlugin`了，它被`splitChunks`和`runtimeChunk`这两个新API所取代。

# 懒加载React组件
现在的vendors和app包都是用户在第一次打开页面室加载的。我们发现可以将一些“重量级”的组件懒加载来提升首屏体验，并且减少初始包的体积。

比如说：redux-form是一个管理react应用表单的库，它只在一个名为`GenerateTags`的大型组件中使用。由于它体积较大并且只在特定场景下被使用，所以用它来作为懒加载的实验对象是再好不过了。redux-form和GenerateTags组件可以被抽取到单独一个chunk中，这样我们在渲染首屏时请求的包体积更小。

让我们看看现在流行的动态导入工具库：`react-loadable`。它基础封装了未来JS的新语法`import()`。
```js
const GenerateTags = Loadable({
  loader: () =>
    import(/* webpackChunkName: "generateTags" */ "./GenerateTags"),
    loading: LoadingSpinner
});
```

使用之后，我们的包变成了下面这样：
![抽取组件](http://ox34ivs2j.bkt.clouddn.com//hexo/2018-3-11/2018-3-11-2.png)  
> GenerateTags已经被抽取到单独的一个chunk中，但redux-form仍然在vendor.js包里。

结果不尽如人意，因为redux-form仍然在vendors.js包中，但我们希望它跟GenerateTags都被抽取到一个不同的chunk中来实现按需加载。

之所以会出现这样的情况，是因为我们在别的文件中也引用了redux-form。比如说我们在`combineReducers `中编写了下面的代码： 
```js
import { reducer as formReducer } from "redux-form";
const applicationReducer: Reducer<any> = combineReducers({
    user,
    sidenav,
    navigation,
    //...
    form: formReducer
});
```
这段代码顶部的静态导入语句导致redux-form库成了我们vendors包的一部分。也就是说，Webpack认为它已经被静态导入成我们的app入口依赖树的一部分，所以不能被懒加载。

为了解决这个问题，我们决定动态注入redux-form reducer。首先，我们移除了导入redux-form reducer的语句，并且加了下面的代码来实现动态注入redux reducer：
```js
export function injectAsyncReducer(store, name, asyncReducer) {
  if (store.asyncReducers[name]) {
    return;
  }
  store.asyncReducers[name] = asyncReducer;
  store.replaceReducer(createReducer(store.asyncReducers));
}

export const configureStore = (initialState: AppState) => {
  const enhancer = compose(applyMiddleware(...getMiddleware()));
  const store: any = createStore(createReducer(), initialState, enhancer);
  store.asyncReducers = {};
  return store;
};


const createReducer = (asyncReducers = {}) => {
    return combineReducers({
        user,
        sidenav,
        navigation,
        //...
        ...asyncReducers
    });
};
```

最后，我们在GenerateTags组件的componentDidMount中调用injectAsyncReducer方法。
```js
public componentDidMount() {
    const reduxFormReducer = require("redux-form").reducer;
    injectAsyncReducer(store, "form", reduxFormReducer);
  }
```

注意，不推荐从组件直接获取一个store的引用，因为这样会导致你在做服务端渲染时出现一些问题。  
在[这里](https://tylergaw.com/articles/dynamic-redux-reducers/)你可以阅读更多注入异步代码和使用HOC的知识。

# TypeScript配置
我们在项目中使用了typescript。我们必须在`tsconfig.json`中更新esnext的module配置，以及设置`removeComments`为`false`（要支持动态注入，TS的版本必须高于2.4）。这样，之前的动态注入才会起作用。通过“告诉”typescript编译器避开我们的import语句，并且不要对它们进行转码来让Webpack正常工作。
```JSON
{
  "compilerOptions": {
    "target": "es5",
    "sourceMap": false,
    "inlineSourceMap": true,
    "module": "esnext",
    "moduleResolution": "node",
    "jsx": "react",
    "preserveConstEnums": true,
    "removeComments": false,
    "lib": ["es6", "dom"]
  },
  "types": ["node"]
}
```
最后的结果就像下面这样：
![](http://ox34ivs2j.bkt.clouddn.com//hexo/2018-3-11/2018-3-11-3.png)  
> vendors.js 314 KiB, app.js 96.6 KiB, generateTags.js 23.2 KiB, vendors~generateTags.js 90.2 KiB

最后我们成功了，GenerateTags和它的依赖文件redux-form被提取出vendor.js并且能够被按需加载。

# 总结
我们推荐你阅读[这个文章](https://developers.google.com/web/fundamentals/performance/webpack/)来优化Webpack。
* 使用动态注入可以减少最终包的体积。还能疼痛感异步加载提供更快的首屏加载速度。
* typescript从2.4版本开始支持动态注入，你只需要记住修改一部分配置就能使用这个功能。
* 迁移到Webpack4并不不复杂，但是目前还没有关于新配置和API的介绍文档。但我相信很快它们都会有的。
* 动态注入redux reducer是一个很有用的小技巧，它能够帮助我们的app在使用redux reducer时延迟加载一些库。

> 查看更多我翻译的Medium文章请访问：  
项目地址：[https://github.com/WhiteYin/translation](https://github.com/WhiteYin/translation/tree/master)  
SF专栏：[https://segmentfault.com/blog/yin-translation](https://segmentfault.com/blog/yin-translation)