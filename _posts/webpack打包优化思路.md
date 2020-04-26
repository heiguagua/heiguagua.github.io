
## 模块化的演进过程  
相信很多人对 webpack 都不陌生，现在流行的前端框架都广泛使用了 webpack，但是我们在开发过程中几乎没怎么用到，所以难免会产生疑问，有没有必要学习 webpack ？答案是肯定的！  
随着互联网的发展，前端技术标准发生了巨大的变化。早期的前端技术标准没有预料到前端会有今天这样的规模，所以在设计上有很多缺陷，导致我们在实现前端模块化时会遇到很多问题。虽然说大部分问题都已经被一些标准或工具解决了，但在这个标准的演进过程中有很多东西值得我们思考和学习。  
### 1、文件划分方式  
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

### 2、命名空间方式  
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

### 3、IIFE  
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

### 4、IIFE 依赖参数  
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
### 模块加载的问题  
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

### 模块化规范的出现 
为了统一不同开发者、不同项目之间的差异，我们需要制定一个行业标准去规范模块化的实现方式。  
**CommonJS**  
提到模块化规范，最先想到的可能就是 CommonJs 规范了，它是 Node.js 中所遵循的模块规范，该规范有以下特点：  
* 每个文件就是一个模块，有自己的作用域，文件里面定义的变量、函数等都是私有的，对其他文件不可见。  
* 服务器端广泛使用的模块化加载机制，以为模块一般都存在本地，不需要考虑网络等因素，所以为同步加载。  
* 模块加载一次就被缓存起来，再次加载直接读取缓存。  
* 模块加载的顺序按照其在代码中出现的顺序加载。  
这种方式在服务器端使用没有问题，但是要在浏览器端使用同步的方式加载模块，就会引起大量的同步模式请求，导致运行效率大大降低。  
**AMD**
所以早期并没有直接选择 CommonJS 规范，而是专门针对浏览器重新设计了一个规范，叫做 AMD (Asynchronous Module Definition) 规范，即异步模块定义规范。最出名的实现库就是 Require.js。  
在AMD规范中约定通过 define() 函数定义模块。  
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
目前绝大多数第三方库都支持 AMD 规范，但是它使用起来相对复杂，随着应用规模扩大，模块划分更为细致时，会vhuxian一个页面请求次数过多的情况。  

### 模块化规范标准规范  
随着技术的发展， JavaScript 的标准逐渐走向完善，而且对前端模块化规范的最佳实践也基本实现了统一。  
* 在 Node.js 环境中，遵循 CommonJS 规范来组织模块。  
* 浏览器环境中，遵循 ES Modules 规范。  
**ES Modules**  
ES Modules 是 ECMAScript 2015（ES6）中才定义的模块系统，在这个标准刚推出的时候，几乎所有主流的浏览器都不支持，但是随着 Webpack 等一系列打包工具的流行，这一规范才开始被逐渐普及。  
经过5年的迭代，ES Modules 已发展成为目前最主流的前端模块化标准，它在语言层面实现了模块化，并且目前绝大多数主流浏览器已经开始能够原生支持 ES Modules 特性了，因此如何在不同的环境中更好的使用ES Modules 是我们要重点考虑的问题。  
### 模块化打包工具的出现  
模块化可以帮助我们解决复杂应用开发过程中的代码组织问题，但是随着模块化思想的引入，前端应用又会产生一些新的问题：  
* 在无法保证用户浏览器使用情况下，我们必须要解决兼容性问题。  
* 模块化的方式划分出来的模块文件过多，每个文件都需要单独从服务器请求。零散的模块导致浏览器发送网络请求频繁，影响工作效率。  
* 在实际的前端应用开发过程中不仅仅只有 Javascript 需要模块化， HTML 和 CSS 也会面临需要被模块化的问题。这些文件都应该看做是前端应用中的一个模块。  
对于开发过程而言，模块化肯定是必须的，所以我们需要在这个基础上引入更好的方案或者工具来解决上面这些问题，让我们继续享受模块化在开发过程给我们带来的享受，同时又不必担心模块化对生产环境产生的影响。  

### Webpack  
前端模块化的进程和最终统一的 ES Modules 标准都是我们深入学习 Webpack 前必须要掌握的只是，同时也是作为前端开发必不可少的基础储备。  
Webpack 发展到今天，已经非常强大了，从一个“打包工具”发展到今天的整个前端项目的“构建系统”，它的初衷“模块化解决方案”透露出的“模块化思想”有很多值得我们学习思考的地方。



## 打包优化思路
* 打包后资源文件大小依赖图  
* 拆包策略，详细解释splitChunks每个参数配置的作用，并以demo验证,(runtimeChunk的作用)  
(runtimeChunk:  几个概念
  **runtime** 以及伴随的 **manifest** 数据 ,主要是指在浏览器运行过程中， webpack 用来连接模块化应用程序所需要的所有代码。它包含：在模块交互时，连接模块所需的加载和解析逻辑。包括已经到加载到浏览器中的连接模块逻辑，以及尚未加载模块的的或延迟加载模块的逻辑。  

  **manifest**  
  我们的应用程序中，经过打包、压缩拆分的细小的chunk在经过webpack 优化过后，原来的src目录结构已经不再存在了，webpack 是如何管理所有所需模块的之间的交互呢，这就是 manifes 数据的由来。  
  当compiler 开始执行、解析和映射应用程序时，会保留所有模块的详细要点，这个数据集合就是 **manifes**，当完成打包并发送到浏览器时， runtime 会通过 manifest 来解析和加载模块。通过使用 manifest 中的数据，runtime 将能够检索这些标识符，找出每个标识符背后对应的模块。

 )

**cacheGroups** 其实是splitChunks里面最核心的配置，一开始我还认为cacheGroups是可有可无的，这是完全错误的，splitChunks就是根据cacheGroups去拆分模块的，包括之前说的chunks属性和之后要介绍的种种属性其实都是对缓存组进行配置的


* 动态加载详解，包括模块及路由层面的懒加载  
* gzip 的原理，webpack实现gzip及效果 


* tree-shaking 的实现及效果，并总结其难点重点，  
webpack4在--mode 为 production 的情况下会默认进行tree-shaking
  
原理： 
* ES6的模块引入是静态分析的，故而可以在编译时正确判断到底加载了什么代码。
* 分析程序流，判断哪些变量未被使用、引用，进而删除此代码。


使用tree-shaking主要包含两方面的优化：  
1、对于组件库的引用优化；  

**TODO** 哪些是有副作用的？大部分tree-shaking 没有生效的原因就是副作用导致的。  

副作用：  
它大致可以理解成：一个函数会、或者可能会对函数外部变量产生影响的行为。  

在当下，想要合理利用tree-shaking，能尽力做的事：  

* 使用 ES6 模块语法编写代码
* 尽量不写带有副作用的代码。诸如编写了立即执行函数，在函数里又使用了外部变量等
* 工具类函数尽量以单独的形式输出，不要集中成一个对象或者类
* 声明 sideEffects


**TODO** babel原理及commonJS规范等  
**TODO** * dll 预编译打包原理及效果  
**TODO** 

## 准备工作：
1、提前执行build --report ，查看打包的依赖树状图  
2、开启nginx，准备好gzip的build包，演示gzip的效果

## QA   
1、如何将多个css文件打包成一个，就像js模块？
