---
title: webpack4-tutorial
date: 2018-05-22 16:01:02
tags:
---
> 原文链接：[Webpack 4 Tutorial: from 0 Conf to Production Mode](https://www.valentinog.com/blog/webpack-4-tutorial/)

**webpack 4** 问世了！

这个流行的模块打包工具进行了大规模的升级。

webpack4，有什么**更新**？大幅度的**性能优化**，零配置和明智的**默认配置**。


![](https://user-gold-cdn.xitu.io/2018/5/14/1635d78ca77ba160?w=768&h=384&f=png&s=67327)

## webpack 4 ，零配置的模块打包工具

webpack是一个强力和有着很多独特功能的工具，但是他的一大痛点在于他的**配置文件**

给中大型项目提供一个配置文件不是什么大问题。你甚至无法离开它。然而，对于一些较小型应用来说就有点麻烦了，尤其是你在心血来潮想开始做一些好玩的app的时候。

这也是[Parcel吸引人](https://www.valentinog.com/blog/tutorial-react-parcel-bundler/)的原因

现在的好消息是webpack 4 **默认再也不需要一个配置文件了！**

## webpack 4：零配置开始

创建一个目录然后进入

``` bash
mkdir webpack-4-quickstart && cd $_
```

初始化package.json：

``` bash
npm init -y
```

安装webpack4：

``` bash
npm i webpack --save-dev
```

我们也需要安装另外一个包：**webpack-cli**

``` bash
npm i webpack-cli --save-dev
```

然后打开[package.json](https://docs.npmjs.com/files/package.json)添加构建脚本：

``` json
"scripts": {
  "build": "webpack"
}
```

关闭保存

试着运行

``` bash
npm run build
```

然后我们看到

```
ERROR in Entry module not found: Error: Can't resolve './src' in '~/webpack-4-quickstart'
```

webpack 4 需要在./src目录下找一个入口文件！如果你不知道这是什么意思，请参考我[之前的文章](https://www.valentinog.com/blog/from-gulp-to-webpack-quickstart/)

简要来说：webpack需要这个入口文件来开始js代码的打包。

在以前的版本里webpack的入口文件需要在配置文件webpack.config.js里指定。

但是现在不用指定了，它会默认选择./src/index.js这个文件。

测试这个新特性很容易，创建一个`./src/index.js`:

``` js
console.log(`I'm a silly entry point`);
```

重新构建：

``` bash
npm run build
```

你会在`~/webpack-4-quickstart/dist/main.js`得到你打包后的文件。

什么？等一下。都不需要指定输出文件吗？是的。

在webpack 4 中**不需要指定入口和出口文件**。

webpack的真正本领是代码拆分。但是相信我，有一个零配置的工具可以加速你的进程。

所以这就是第一个新特性：他会把./src/index.js默认为入口文件，把打包后的文件放在./dist/main.js。

下一章后我们能看到另一个有用的特性：**生产和开发模式**。

## Webpack 4:生产和开发模式

在webpack中拥有两份配置文件是常事。

一个典型的项目应该有：

- 一个**开发用的配置文件**，用来定义webpack的dev server和其他东西
- 一个**生产环境用的配置文件**，用来定义**UglifyJSPlugin**，sourcemap和其他东西。

在webpack4中你能够不写一行配置。

怎么做到的？

webpack 4介绍了生产和开发模式。

实际上如果你关注过`npm run build`的输出信息你会看到这个警告：

![](https://user-gold-cdn.xitu.io/2018/5/14/1635da1cf59100eb?w=768&h=241&f=png&s=93607)

> The ‘mode’ option has not been set. Set ‘mode’ option to ‘development’ or ‘production’ to enable defaults for this environment.

这代表什么？我们看看。

打开package.json文件添加如下脚本

``` json
"scripts": {
  "dev": "webpack --mode development",
  "build": "webpack --mode production"
}
```

现在运行：

``` bash
npm run dev
```

查看./dis/main.js文件。你看到了什么？嗯，我知道，他没有被压缩！

现在这样：

``` bash
npm run build
```

现在再看，你看到什么？一个压缩后的文件！

是的！

生产模式开启了一系列额外的优化。包括minification, scope hoisting, tree-shaking等。

另一边开发模式为速度做了优化，除了提供一个没有压缩的包以外没有做额外的事。

所以这是第二个新特性：生产和开发模式。

在webpack4你不需要一行配置，只需要一个--mode选项。

## webpack 4：覆盖默认的入口/出口文件

我喜欢webpack4的零配置，但是，如果我要覆盖默认的入口或者出口配置要怎么做呢？

在`package.json`里配置他们。

这是一个例子

``` json
"scripts": {
  "dev": "webpack --mode development ./foo/src/js/index.js --output ./foo/main.js",
  "build": "webpack --mode production ./foo/src/js/index.js --output ./foo/main.js"
}
```

## webpack 4：用Babel转换ES6的js代码

![](https://user-gold-cdn.xitu.io/2018/5/15/16361a7de058dcce?w=768&h=349&f=png&s=81015)

现代JavaScript大多是用ES6写的。

但是不是所有浏览器都知道怎么处理ES6。我们需要做一些转换。

这个转换的步骤叫做**transpiling**。transpiling是指把ES6转换成浏览器能够识别的代码。

webpack本身并不知道如何去转换，但是它有**loaders**。把他们想象成转换器。

**babel-loader**是webpack的一个loader，可以转换ES6以上的代码到ES5。

为了使用这个loader我们需要去安装一系列的依赖。特别是：

- babel-core
- babel-loader
- babel-preset-env （for compiling Javascript ES6 code down to ES5）

来安装吧：

``` bash
npm i babel-core babel-loader babel-preset-env --save-dev
```

下一步我们在项目目录下建立一个.babelrc文件用来配置Babel。

``` json
{
    "presets": [
        "env"
    ]
}
```

在这里我们有两个途径去配置babel-loader：

- 用webpack的配置文件
- 在npm脚本里使用`--module-bind`

哦，我知道你在想什么了。webpack4把自己定位为一个零配置的工具。为什么我们又要写配置文件了呢。

webpack 4的零配置适用于：

- 入口文件。默认是./src/index.js
- 出口文件。默认是./dist/main.js
- **生产和开发模式**（无需创建两套配置文件）

这就够了。对于loaders我们仍然需要使用配置文件。

我曾关于这件事问过Sean。在webpack4中loaders是否和webpack3中没有区别？有计划对这些通用的loaders比如babel-loader提供零配置吗？

他的回答是：

> 在未来的版本（v4之后，可能是4.x或者5.0），我们已经开始探索预设或附加系统如何帮助我们定义这一点。我们不想要的是：把一堆东西塞到核心代码里去。我们需要的是：能够支持扩展。

对于现在来说你仍必须依赖**webpack.config.js**。

### webpack 4：通过配置文件使用babel-loader

创建一个名叫`webpack.config.js`的文件然后配置loader：

``` js
module.exports = {
  module: {
    rules: [
      {
        test: /\.js$/,
        exclude: /node_modules/,
        use: {
          loader: "babel-loader"
        }
      }
    ]
  }
};
```

没有必要去指定入口文件除非你想指定。

下一步打开./src/index.js然后写一些ES6代码：

``` js
const arr = [1, 2, 3];
const iAmJavascriptES6 = () => console.log(...arr);
window.iAmJavascriptES6 = iAmJavascriptES6;
```

最后打包：

``` bash
npm run build
```

然后去./dist/main.js看看转换后的代码。

### webpack 4：不通过配置文件使用babel-loader

还有一种使用webpack loaders的方法。

`--module-bind`选项让你从命令行指定loaders。谢谢Cezar指出了这一点。

这个选项并不是webpack 4独有的。3开始就有了。

你可以这样在package.json中使用：

``` json
"scripts": {
    "dev": "webpack --mode development --module-bind js=babel-loader",
    "build": "webpack --mode production --module-bind js=babel-loader"
}
```

然后你就可以开始构建了。

然后我并不是很喜欢这种方法（不喜欢太长的npm脚本），尽管如此它很有趣。

## webpack 4：在webpack 4中配置React


![](https://user-gold-cdn.xitu.io/2018/5/15/163626f7eb3451c9?w=768&h=258&f=png&s=46525)

如果你已经安装配置好了babel这会很简单。

安装React：

``` bash
npm i react react-dom --save-dev
```

添加`babel-preset-react`：

``` bash
npm i babel-preset-react --save-dev
```

在.babelrc里配置preset

``` json
{
  "presets": ["env", "react"]
}
```

这样就可以了。

如[Conner Aiken](http://fittedtech.com/home/)建议的你可以配置babel-loader也去加载**.jsx**文件。这在你使用jsx扩展名的时候很有用。

打开`webpack.config.js`然后这样配置：

``` js
module.exports = {
  module: {
    rules: [
      {
        test: /\.(js|jsx)$/,
        exclude: /node_modules/,
        use: {
          loader: "babel-loader"
        }
      }
    ]
  }
};
```
测试是否搭好你可以在`./src/App.js`里创建一个React组件。

``` js
import React from "react";
import ReactDOM from "react-dom";
const App = () => {
  return (
    <div>
      <p>React here!</p>
    </div>
  );
};
export default App;
ReactDOM.render(<App />, document.getElementById("app"));
```

然后在`./src/idnex.js`中引入：

``` js
import App from "./App";
```

重新构建


## webpack 4：HTML插件

webpack需要两个额外的组件去处理HTML：html-webpack-plugin和html-loader。

添加这两个依赖：

``` bash
npm i html-webpack-plugin html-loader --save-dev
```

然后更新webpack的配置

``` js
const HtmlWebPackPlugin = require("html-webpack-plugin");
module.exports = {
  module: {
    rules: [
      {
        test: /\.js$/,
        exclude: /node_modules/,
        use: {
          loader: "babel-loader"
        }
      },
      {
        test: /\.html$/,
        use: [
          {
            loader: "html-loader",
            options: { minimize: true }
          }
        ]
      }
    ]
  },
  plugins: [
    new HtmlWebPackPlugin({
      template: "./src/index.html",
      filename: "./index.html"
    })
  ]
};
```

在`./src/index.html`新建一个HTML文件：

``` html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <title>webpack 4 quickstart</title>
</head>
<body>
    <div id="app">
    </div>
</body>
</html>
```

运行：

``` bash
npm run build
```

查看`./dist`目录。你会看到运行的结果。

没有必要再你的HTML文件中引入你的JavaScript：它会自动地注入进去。

在浏览器打开`./dist/index.html`：你可以看到React组件运行起来了！

如你所见在处理HTML上没有什么变化。

webpack 4仍然是一个主要目标是js的模块打包工具。

但有个将HTML作为模块的方法（HTML作为入口）。

## 提取CSS到文件中

webpack不知道怎么去提取CSS到文件中。

在之前这是**extract-text-webpack-plugin**的工作。

不幸的是这个插件在webpack 4表现并不好。

Michael Ciniawsky说：

> 维护extract-text-webpack-plugin是一个很大的负担，而且这不是第一次因为这个问题使得升级webpack的主要版本变得困难。

[mini-css-extract-plugin](https://github.com/webpack-contrib/mini-css-extract-plugin)是来解决这些问题的。

> 提示：你需要把webpack升级到4.2.0.0，不然这个插件无法运行！

安装它：

``` bash
npm i mini-css-extract-plugin css-loader --save-dev
```

然后建立一个CSS文件用来测试

``` css
/* */
/* CREATE THIS FILE IN ./src/main.css */
/* */
body {
    line-height: 2;
}
```

配置plugin和loader：

``` js
const HtmlWebPackPlugin = require("html-webpack-plugin");
const MiniCssExtractPlugin = require("mini-css-extract-plugin");
module.exports = {
  module: {
    rules: [
      {
        test: /\.js$/,
        exclude: /node_modules/,
        use: {
          loader: "babel-loader"
        }
      },
      {
        test: /\.html$/,
        use: [
          {
            loader: "html-loader",
            options: { minimize: true }
          }
        ]
      },
      {
        test: /\.css$/,
        use: [MiniCssExtractPlugin.loader, "css-loader"]
      }
    ]
  },
  plugins: [
    new HtmlWebPackPlugin({
      template: "./src/index.html",
      filename: "./index.html"
    }),
    new MiniCssExtractPlugin({
      filename: "[name].css",
      chunkFilename: "[id].css"
    })
  ]
};
```

最后在入口文件中引入CSS：

``` js
//
// PATH OF THIS FILE: ./src/index.js
//
import style from "./main.css";
```

构建：

``` bash
npm run build
```

查看./dist目录，你应该能看到CSS的结果！

结论：extract-text-webpack-plugin在webpack 4中不能用了。请使用mini-css-extract-plugin。

## webpack 4：webpack dev server

在你改变代码后运行`npm run dev`？这不是个理想的做法。花几分钟去配置下webpack的开发服务。一旦配置了[webpack dev server ](https://github.com/webpack/webpack-dev-server)它会在浏览器中加载你的app。

只要你改变了文件，它会自动地刷新浏览器的页面。

安装下面的包来搭建webpack dev server：

``` bash
npm i webpack-dev-server --save-dev
```

然后打开`package.json`调整脚本：

``` json
"scripts": {
  "start": "webpack-dev-server --mode development --open",
  "build": "webpack --mode production"
}
```

保存关闭。

现在运行：

``` bash
npm run start
```

你就能看到webpack dev server在浏览器中加载你的应用了。

webpack dev server非常适合用来开发。（而且它能使得的React Dev Tools在浏览器中正常的工作）

## webpack 4：资源

本教程在Github上的链接 => [webpack-4-quickstart](https://github.com/valentinogagliardi/webpack-4-quickstart)

我知道早就有很多webpack的列表但是这里是我的：一系列优秀的webpack4资源 => [awesome-webpack-4](https://github.com/valentinogagliardi/awesome-webpack-4)

这里一定还要提一下Juho Vepsäläinen的[SurviveJS webpack 4](https://survivejs.com/webpack/)