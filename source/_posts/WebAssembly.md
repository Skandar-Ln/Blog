---
title: WebAssembly
date: 2018-08-22 20:03:15
tags:
---
## WebAssembly: How and why

> 如何在浏览器中运行原生代码，为什么要这样做，这样做对Javascript和Web开发的未来有何意义

![](https://user-gold-cdn.xitu.io/2018/8/21/1655b06a89537c3c?w=1351&h=669&f=png&s=80802)

在所有浏览器里面，都运行着js代码，它们被js引擎解析和执行。然而，js并无法最理想地处理所有任务。这就是WebAssembly介入的地方。

WebAssembly是一种新的能在现代浏览器中运行的代码。为了更高的性能而被设计出来，它是一种低级二进制格式，所以文件不大能够很快地被下载和执行。你不会直接去写WebAssembly，而是把其他高级语言编译为它。

Assembly通常是指与机器码类似的人类可读语言。机器码是你的处理器才能理解的一堆数字。

![Assembly languages and machine code](https://user-gold-cdn.xitu.io/2018/8/21/1655b85c0e57da4d?w=1400&h=814&f=png&s=69114)

所有的高级编程语言都需要编译为能在处理器上运行的机器码。不同的处理器架构需要不同的机器码和对应不同的Assembly。

![Compiling source code for different processor architectures](https://user-gold-cdn.xitu.io/2018/8/21/1655b892a6ae8d5e?w=1400&h=807&f=png&s=36434)

不像它的名字那样，WebAssembly不代表某种特定的Assembly语言，他不针对特定的机器。他是面向浏览器的，当你提供要在浏览器中执行的代码时，你不需要考虑它会运行在什么样的机器上。

![WebAssembly as an intermediary compiler target](https://user-gold-cdn.xitu.io/2018/8/21/1655b8c4e0c81e0f?w=1400&h=1000&f=png&s=41434)

当浏览器下载了WebAssembly代码后会快速将它转换成某台机器的assembly。

WebAssembly拥有可读的文本格式（.wat），但是二进制格式是你真正交付给浏览器的（.wasm）。

![WebAssembly textual and binary format](https://user-gold-cdn.xitu.io/2018/8/21/1655c535b6298463?w=800&h=415&f=png&s=40080)

WebAssembly能让你做的是将C,C++或者Rust的代码编译为叫做WebAssembly模块的东西。你可以在你的web应用中加载它并通过Javascript调用。

它并非Javascript的替代品，它和js一同工作的。

![WebAssembly module in an application](https://user-gold-cdn.xitu.io/2018/8/21/1655c55f64952173?w=800&h=444&f=png&s=37053)

## 为什么需要WebAssembly

想想你不得不在浏览器外使用软件的场景：视频游戏、视频编辑，3D渲染，或音乐制作。这些应用都要进行大量的运算并需要很高的性能。Javascript并不能提供如此的性能。

Javascript最初被设计来给Web提供一些轻量级的交互。就是为了让人易学和好写，但是并没有从高性能角度去设计。近年来，浏览器们在[解析Javascript](https://hacks.mozilla.org/2017/02/a-crash-course-in-just-in-time-jit-compilers/)上做了很多优化，有了很大的性能提升。

随着性能的提升，能在浏览器做的事慢慢扩展。新的API带来了可交互的图像，视频流，离线浏览等等。所以一些从前只作为原生App的应用也出现在了Web上。现在你可以再浏览器上编辑文档，发送邮件。但是某些场景对Javascript来说还是很吃力。

视频游戏是个巨大的挑战，因为它们不仅需要协调音频和视频，还需要协调物理和人工智能。如果能在Web上高效地运行游戏的话，那么我们就能够[将很多应用带到Web上来](https://webassembly.org/docs/use-cases/)。这就是WebAssembly要做的事。

## 为什么Web如此吸引人

Web的美丽之处在于它就像魔法一样能在任何地方运行。没有**下载和安装**。只需要点点鼠标Web应用就能马上出现。它比直接在计算机上下载和运行二进制文件更安全，因为浏览器提供了代码运行的[安全环境](https://www.howtogeek.com/169139/sandboxes-explained-how-theyre-already-protecting-you-and-how-to-sandbox-any-program/)。而且通过Web分享是一件很容易的事，你可以把链接复制到任何地方。

## WebAssembly能带来什么

- 速度
- 可移植性
- 灵活性

WebAssembly就是为了速度而设计的。它的二进制格式比Javascript的文本格式小巧很多。因此可以很快地下载，在网络较慢的情况下尤为明显。

并且也能很快地解码和执行。Javascript是一种动态类型的语言，变量类型不需要事先定义，也不需要提前编译。这使得js写起来十分容易，不过这也意味着js引擎需要额外地做很多事。它需要在页面执行的同时去解析、编译然后优化代码。

解析js代码包括把纯文本代码转换成[抽象语法树](https://en.wikipedia.org/wiki/Abstract_syntax_tree)（AST），然后转成二进制格式。WebAssembly直接以二进制形式提供，解码速度更快。不像Javascript，它是静态类型的，引擎在编译期间不需要推测将使用哪种类型。大多数优化都是在编译源代码期间，甚至在进入浏览器之前发生的。内存是手动管理的，就像C和C ++这样的语言一样，所以也没有垃圾收集。**WASM二进制代码的执行时间仅仅只比原生代码慢20%**。

![Relative time spent processing WebAssembly in JavaScript engine](https://user-gold-cdn.xitu.io/2018/8/22/16561675ac0a6a77?w=1400&h=1000&f=png&s=69318)

设计WebAssembly的主要目标之一是可移植性。要在设备上运行应用程序，它必须与设备的处理器体系结构和操作系统兼容。这意味着要为所有需要支持的操作系统和CPU的组合编译源代码。使用WebAssembly，只有一个编译步骤，你的应用程序将在每个现代浏览器中运行。

![Compiling native code to run on different platforms vs. compiling to WebAssembly](https://user-gold-cdn.xitu.io/2018/8/22/1656169d54c1ea86?w=1400&h=910&f=png&s=89603)

WebAssembly最令人兴奋的是它为Web编写带来了更大的灵活性。到目前为止，JavaScript一直是Web浏览器中唯一完全支持的语言。使用WebAssembly，Web开发人员将能够选择其他语言，更多的开发人员将能够为Web编写代码。JavaScript仍然是大多数情况下的最佳选择，但现在有一个在你需要更高性能时能够选择的方案。

目前完全支持的语言是C，C++和Rust，但还有[很多其他语言](https://github.com/appcypher/awesome-wasm-langs)正在开发中，包括Kotlin和.NET，两者都已经提供了实验支持。

## 它如何运作的

你需要一个将源代码编译为WebAssembly的工具。如C何C++你可以使用[Emscripten](http://kripken.github.io/emscripten-site/)。如果你有一个C语言写的"Hello word"，Emscripten会生成必要的文件，你会得到一个WebAssembly模块和HTML以及JS文件。

```bash
emcc hello.c -s WASM=1 -o hello.html
```

![Compiling C/C++ code to WebAssembly with Emscripten](https://user-gold-cdn.xitu.io/2018/8/22/16561759bf05803e?w=1400&h=645&f=png&s=37826)

你需要HTML和JS文件，因为WebAssembly无法直接访问任何平台的API - DOM，WebGL，WebAudio等。要使用其中任何一个，即使要在页面上显示WebAssembly代码的输出，您也必须通过JavaScript。

您可以将WebAssembly二进制文件视为普通的应用程序模块：浏览器可以获取，加载和执行它们。您可以在JavaScript代码中调用WebAssembly函数，也可以在WebAssembly模块中调用JavaScript函数。

如果你现在想试试，无需安装，你可以访问[webassembly.studio](http://webassembly.studio/)或[WebAssembly Explorer](https://mbebenita.github.io/WasmExplorer/)

## 现在能使用它吗？
### YES！

 WebAssembly几乎在所有主流浏览器中得到了支持。它目前在全球支持74.93％的用户，82.92％的桌面用户。我们可以使用Emscripten编译为[asm.js](http://asmjs.org/faq.html)（JavaScript的一个子集，只使用数字（没有字符串，对象等））作为老版本浏览器的兼容方案。
 
![Browsers that support WebAssembly](https://user-gold-cdn.xitu.io/2018/8/22/165617fb7f3b9c53?w=1600&h=320&f=png&s=223775)

目前已经有很多很棒的WebAssembly例子。如上文提到的难点：视频游戏，Unity和虚幻4都有可运行的demo。比如这个在Unity引擎运行的[坦克游戏](https://webassembly.org/demo/)和Epic的一个[Demo](https://mzl.la/webassemblydemo)。

> 原文链接：[WebAssembly: How and why](https://user-gold-cdn.xitu.io/2018/8/22/1655faed44dd5bd4)