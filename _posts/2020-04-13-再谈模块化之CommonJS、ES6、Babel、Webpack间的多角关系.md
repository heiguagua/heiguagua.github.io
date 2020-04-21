---
layout: post
title:  "模块化之CommonJS、ES6、Babel、Webpack的纠葛"
categories: Web
tags:  CommonJS ES6 Babel Webpack
author: May
---

* content
{:toc}

## 1、CommonJS 和 ES6的关系
### CommonJS
**同步加载模块、NodeJS**
* 每个文件就是一个模块，有自己的作用域，文件里面定义的变量、函数等都是私有的，对其他文件不可见。  
* 服务器端广泛使用的模块化加载机制，以为模块一般都存在本地，不需要考虑网络等因素，所以为同步加载。  
* 模块加载一次就被缓存起来，再次加载直接读取缓存。  
* 模块加载的顺序按照其在代码中出现的顺序加载。  

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
注意： 不能直接将 exports 变量指向一个值，因为这相当于切断了 exports 和 module.exports 的联系。   
**require 命令**  
node 内置的 `require` 命令用来加载模块文件。  
`require` 命令的基本功能就是，读入并执行一个 javascript 文件，然后返回该模块的 exports 对象。
```
var say = require('a.js');

say('hello);
```
## 2、Babel  
### Babel产生的背景
为了理解为什么会出现 Babel，就需要了解一下 javascript 的历史了。  
javascript 是 web 语言，不同浏览器都会有不同的 javascript 解释器对其进行解释编译运行。由于 js 被广泛接受，随后又有 `ECMA (European Computer Manufacturers Association 欧洲计算机制造商协会)`对其进行规范管理，js 所遵循的规范也也叫 ECMAScript 或者 ES。  
其中第5版也就是 `ES5` 于2009年定稿，目前主流浏览器都全部支持。  
第6个版本ES2015 即 `ES6` ，在2015年定稿，目前主流浏览器并不是全部支持。  
还有ES7、ES8 在原来的基础上，增加了一些新功能。  

如果我们希望立刻就能使用 ES6/ES7/ESNext...，但同时我们也希望我们的代码能在主流浏览器或者node中正常运行，那这就是 Babel 产生的原因了。  

简单来说，Babel就是把JavaScript 中 ES6/ES7/ESNext等新语法转换为ES5（也可以转换为更低的规范，但目前 ES5 规范已经足以覆盖绝大部分主流浏览器，因此可以认为转到 ES5 是一个安全且流行的做法），让低端运行环境如浏览器或者node能认识并执行的工具。  

### Babel的使用方法  
一共有以下三种方式：  
1、`standalone script`，单体文件，最简单快捷的方式，通过script tag 来引用 `babel-standalong` package。    
```
<script
  src="https://cdnjs.cloudflare.com/ajax/libs/babel-standalone/6.18.1/babel.min.js"
></script>
```   
引入后，babel就会自动将任何以 text/babel 为 type 的 script 进行解析。  
2、命令行（cli），多见于 package.json 中的某条命令。  
3、构建工具插件（webpack 的 babel-loader、rollup 的 rollup-plugin-babel）。

### 运行方式  
**plugin**  
Babel 本身不具备任何转化功能，它通过配置 `plugin` 来对不同的语法特性做转译。  
**presets**  
由于不同的语法特性特别多，那么对应的babel plugin也会特别多，如果选择手动添加并安装会非常繁琐且不方便管理，为了解决这个问题，babel 提供了一组插件集合 `presets`，避免了重复定义和安装。
## 3、