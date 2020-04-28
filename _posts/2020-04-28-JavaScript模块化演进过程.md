---
layout: post
title:  "JavaScript webpack相关"
categories: Web
tags:  JavaScript webpack
author: May
---

### 模块化的演进过程  
相信很多人对 webpack 都不陌生，现在流行的前端框架都广泛使用了 webpack，但是我们在开发过程中几乎没怎么用到，所以难免会产生疑问，有没有必要学习 webpack ？答案是肯定的！  
随着互联网的发展，前端技术标准发生了巨大的变化。早期的前端技术标准没有预料到前端会有今天这样的规模，所以在设计上有很多缺陷，导致我们在实现前端模块化时会遇到很多问题。虽然说大部分问题都已经被一些标准或工具解决了，但在这个标准的演进过程中有很多东西值得我们思考和学习。  

#### 1、文件划分方式  
最早我们基于文件划分的方式实现模块化，具体做法是将每个功能模块及相关数据和状态单独放在不同的js文件中，约定每个文件是一个独立的模块。使用时一个script标签对应一个模块，然后直接调用模块中的成员（变量/函数）。  
```
// moduleA.js
function foo() {
  console.log('moduleA#foo');
}
```
```
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <title>Stage 1</title>
</head>
<body>
  <script src="moduleA.js"></script>
  <script>
    // 直接使用全局成员
    foo() // 可能存在命名冲突
  </script>
</body>
</html>
```
缺点：  
* 模块直接作用于全局，大量模块成员对全局作用域造成污染  
* 没有私有控件，模块的成员都可能在模块外部被访问或者修改  
* 模块增多，很容易产生命名冲突  
* 模块间的依赖关系不易管理  
一旦项目规模变大，这种靠约定实现的模块化会暴露出种种问题。  

#### 2、命名空间方式  
这个阶段约定每个模块只暴露一个全局对象，所有模块成员都挂载到这个全局对象中。具体做法就是将每个模块包裹为一个全局对象的形式，这种方式就像是为模块内的成员增加了“命名空间”，所以称之为命名空间方式。  
```
// moduleA.js  
window.moduleA = {
  method1: function () {
    console.log('moduleA#foo')
  }
}

// moduleB.js  
window.moduleB = {
  data: 'something'
  method1: function () {
    console.log('moduleB#method1')
  }
}
```

```
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <title>Stage 2</title>
</head>
<body>
  <script src="module-a.js"></script>
  <script src="module-b.js"></script>
  <script>
    moduleA.method1()
    moduleB.method1()
    // 模块成员依然可以被修改
    moduleA.data = 'foo'
  </script>
</body>
</html>
```
这种方式只是解决了命名冲突的问题，但是其他问题依旧存在。  

#### 3、IIFE  
使用立即执行函数表达式（IIFE，Immediately-Invoked Function Expression）为模块提供私有控件，具体做法是将每个模块成员都放在一个立即执行函数所形成的私有作用域中，对于需要暴露给外部的成员，通过挂到全局对象上的方式实现。  
```
// module-a.js
;(function () {
  var name = 'module-a'

  function method1 () {
    console.log(name + '#method1')
  }

  window.moduleA = {
    method1: method1
  }
})()  

// module-b.js
;(function () {
  var name = 'module-b'

  function method1 () {
    console.log(name + '#method1')
  }

  window.moduleB = {
    method1: method1
  }
})()
```
这种方式带来了私有成员的概念，私有成员只能在模块成员内通过闭包的形式访问，这就解决了前面所提到的全局作用域污染和命名冲突的问题。  

#### 4、IIFE 依赖参数  
在 IIFE 的基础上，我们还可以利用IIFE参数作为依赖声明使用，使得模块间的依赖关系更加明显。  
```
// module-a.js
;(function ($) { // 通过参数明显表明这个模块的依赖
  var name = 'module-a'

  function method1 () {
    console.log(name + '#method1')
    $('body').animate({ margin: '200px' })
  }

  window.moduleA = {
    method1: method1
  }
})(jQuery)
```
#### 模块加载的问题  
以上4个阶段是早期的开发者在没有工具和规范的情况下对模块化的落地方式，这些方式解决了很多在前端领域实现模块化的问题，但是仍然存在一些问题。  
```
<!DOCTYPE html>
<html>
<head>
  <title>Evolution</title>
</head>
<body>
  <script src="https://unpkg.com/jquery"></script>
  <script src="module-a.js"></script>
  <script src="module-b.js"></script>
  <script>
    moduleA.method1()
    moduleB.method1()
  </script>
</body>
</html>
```
最明显的问题就是：模块的加载。我们都是通过script标签的方式直接在页面中引入的这些模块，意味着这些模块的加载不受代码的控制，时间久了维护起来会十分麻烦。  

更为理想的方式应该是在页面中引入一个入口js文件，其余用到的模块可以通过代码控制，**按需加载**进来。  

#### 模块化规范的出现 
为了统一不同开发者、不同项目之间的差异，我们需要制定一个行业标准去规范模块化的实现方式。  
**CommonJS**  
提到模块化规范，最先想到的可能就是 CommonJs 规范了，它是 Node.js 中所遵循的模块规范  
**module 对象**  
node 提供了一个 `Module` 构造函数，所有的模块都是 `Module` 的实例。每个模块都有一个 `module` 对象，代表当前模块。  
`module.exports `是 `module` 对象重要的属性，表示当前模块对外输出的接口，其他文件加载该模块，实际上就是读取的 `module.exports` 变量。  
**exports 变量**  
为了方便，node为每个模块提供了exports变量，指向 module.exports。这相当于在每个模块的头部，都有一行这样的代码：  

```
var exports = module.exports;
```
对外输出
```
// a.js
exports.msg = 'hello';

exports.say = function(p) {
    return p;
}
```
**require 命令**  
node 内置的 `require` 命令用来加载模块文件。  
`require` 命令的基本功能就是，读入并执行一个 javascript 文件，然后返回该模块的 exports 对象。

```
var say = require('a.js');

say('hello);
```
该规范有以下特点：  
* 每个文件就是一个模块，有自己的作用域，文件里面定义的变量、函数等都是私有的，对其他文件不可见。  

* 服务器端广泛使用的模块化加载机制，以为模块一般都存在本地，不需要考虑网络等因素，所以为同步加载。  

* 模块加载一次就被缓存起来，再次加载直接读取缓存。  

* 模块加载的顺序按照其在代码中出现的顺序加载。  
  这种方式在服务器端使用没有问题，但是要在浏览器端使用同步的方式加载模块，就会引起大量的同步模式请求，导致运行效率大大降低。  

**AMD**
所以早期并没有直接选择 CommonJS 规范，而是专门针对浏览器重新设计了一个规范，叫做 AMD (Asynchronous Module Definition) 规范，即异步模块定义规范。最出名的实现库就是 Require.js。  
  在 AMD 规范中约定通过 define() 函数定义模块。  

```
// AMD 定义一个模块
define(['jquery', './module.js'], function($, module) {
  return {
    method1: function() {
      module();
      ...
    },
    ...
  }
})
```
除此之外，Require.js 还提供了一个 require() 函数用于加载模块，用法与 define() 函数类似，区别在于 require() 只能用来载入模块，define() 可以定义模块。  
```
require(['./module.js'], function(module) {
  module.method();
})
```
目前绝大多数第三方库都支持 AMD 规范，但是它使用起来相对复杂，随着应用规模扩大，模块划分更为细致时，会出现一个页面请求次数过多的情况。  

#### 模块化规范标准规范  
随着技术的发展， JavaScript 的标准逐渐走向完善，而且对前端模块化规范的最佳实践也基本实现了统一。  
* 在 Node.js 环境中，遵循 CommonJS 规范来组织模块。  
* 浏览器环境中，遵循 ES Modules 规范。  
**ES Modules**  
ES Modules 是 ECMAScript 2015（ES6）中才定义的模块系统，在这个标准刚推出的时候，几乎所有主流的浏览器都不支持，但是随着 Webpack 等一系列打包工具的流行，这一规范才开始被逐渐普及。  
经过5年的迭代，ES Modules 已发展成为目前最主流的前端模块化标准，它在语言层面实现了模块化，并且目前绝大多数主流浏览器已经开始能够原生支持 ES Modules 特性了，因此如何在不同的环境中更好的使用ES Modules 是我们要重点考虑的问题。  
#### 模块化打包工具的出现  
模块化可以帮助我们解决复杂应用开发过程中的代码组织问题，但是随着模块化思想的引入，前端应用又会产生一些新的问题：  
* 在无法保证用户浏览器使用情况下，我们必须要解决兼容性问题。  
* 模块化的方式划分出来的模块文件过多，每个文件都需要单独从服务器请求。零散的模块导致浏览器发送网络请求频繁，影响工作效率。  
* 在实际的前端应用开发过程中不仅仅只有 JavaScript 需要模块化， HTML 和 CSS 也会面临需要被模块化的问题。这些文件都应该看做是前端应用中的一个模块。  
对于开发过程而言，模块化肯定是必须的，所以我们需要在这个基础上引入更好的方案或者工具来解决上面这些问题，让我们继续享受模块化在开发过程给我们带来的享受，同时又不必担心模块化对生产环境产生的影响。  

#### Webpack  
前端模块化的进程和最终统一的 ES Modules 标准都是我们深入学习 Webpack 前必须要掌握的只是，同时也是作为前端开发必不可少的基础储备。  
Webpack 发展到今天，已经非常强大了，从一个“打包工具”发展到今天的整个前端项目的“构建系统”，它的初衷“模块化解决方案”透露出的“模块化思想”有很多值得我们学习思考的地方。

### Babel  
#### Babel 产生的背景  
javascript 是 web 语言，不同浏览器都会有不同的 javascript 解释器对其进行解释编译运行。由于 js 被广泛接受，随后又有 `ECMA (European Computer Manufacturers Association 欧洲计算机制造商协会)`对其进行规范管理，js 所遵循的规范也叫 ECMAScript 或者 ES。  
其中第5版也就是 `ES5` 于2009年定稿，目前主流浏览器都全部支持。  
第6个版本 ES2015 即 `ES6` ，在2015年定稿，目前主流浏览器并不是全部支持。  
还有ES7、ES8 在原来的基础上，增加了一些新功能。  

如果我们希望立刻就能使用 ES6/ES7/ESNext...，但同时我们也希望我们的代码能在主流浏览器或者node中正常运行，那这就是 Babel 产生的原因了。  

简单来说，Babel就是把JavaScript 中 ES6/ES7/ESNext等新语法转换为 ES5（也可以转换为更低的规范，但目前 ES5 规范已经足以覆盖绝大部分主流浏览器，因此可以认为转到 ES5 是一个安全且流行的做法），让低端运行环境如浏览器或者node能认识并执行的工具。  

我们可以将babel理解为一个转译器 transpiler 而不是编译器 compiler 更准确，因为它只是把同种语言的高版本规则翻译成低版本规则，但是和编译器类似，它也分为3个阶段，**parsing**、 **transforming**、**generating**，以 ES6 转译为 ES5 为例，babel转译的具体过程如下：  

![babel](./assets/babel.png)

#### 关键概念  
**plugins**  
Babel 本身不具备任何转化功能，它通过配置 `plugins` 来对不同的语法特性做转译，主要作用于第二个阶段 **transforming**。  
**presets**  
由于不同的语法特性特别多，那么对应的 babel plugin 也会特别多，如果选择手动添加并安装会非常繁琐且不方便管理，为了解决这个问题，babel 提供了一组插件集合 `presets`，避免了重复定义和安装。  
1、**babel-presest-env**   
比如我们需要转换 ES6 语法，可以再 .babelrc 的 plugins 中引入插件： es2015-arrow-functions、es2015-block-scoped-function 等等不同作用的 plugin：  

```
// .babelrc
{
  "plugins": [
    "es2015-arrow-functions",
    "es2015-block-scoped-functions",
    // ...
  ]
}
```
但是随着需要转换的语法变多，plugin 也相应增多，不便于维护，所以为了方便 Babel 团队将同属 ES2015 的 transform plugins 集合到了 **babel-preset-es2015** 的 Preset 中，这样我们只需要在 .babelrc 的 presets 引入一个 ES2015 就可以全部支持 ES2015 语法了。  
```
// .babelrc
{
  "presets": [
    "es2015"
  ]
}
```

随着时间推移，可能会有更多版本的插件，如 babel-preset-es2020,...等等，因此 **babel-preset-env** 出现了，它类似于 **babel-preset-latest** ，会根据目标环境来进行转译。  
```
{
  "presets": ['env']
}
```

**polyfill**  
Babel 默认只转换 JavaScript 语法，而不转换新的 API，如 Iterator、Set、Maps、Symbol、Promise等全局对象。以及一些在全局对象上的方法（如 Object.assign）等都不会转码。如果想让这些全局对象或API就需要使用 babel-polyfill 来转换。