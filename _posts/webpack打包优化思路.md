
## 打包优化思路
* 打包后资源文件大小依赖图  
* 拆包策略，详细解释splitChunks每个参数配置的作用，并以demo验证,(runtimeChunk的作用)  
(runtimeChunk:  几个概念
  **runtime** 以及伴随的 **manifest** 数据 ,主要是指在浏览器运行过程中， webpack 用来连接模块化应用程序所需要的所有代码。它包含：在模块交互时，连接模块所需的加载和解析逻辑。包括已经到加载到浏览器中的连接模块逻辑，以及尚未加载模块的的或延迟加载模块的逻辑。  

  **manifest**  
  我们的应用程序中，经过打包、压缩拆分的细小的chunk在经过webpack 优化过后，原来的src目录结构已经不再存在了，webpack 是如何管理所有所需模块的之间的交互呢，这就是 manifes 数据的由来。  
  当compiler 开始执行、解析和映射应用程序时，会保留所有模块的详细要点，这个数据集合就是 **manifes**，当完成打包并发送到浏览器时， runtime 会通过 manifest 来解析和加载模块。通过使用 manifest 中的数据，runtime 将能够检索这些标识符，找出每个标识符背后对应的模块。

 )

## cacheGroups
其实是splitChunks里面最核心的配置，一开始我还认为cacheGroups是可有可无的，这是完全错误的，splitChunks就是根据cacheGroups去拆分模块的，包括之前说的chunks属性和之后要介绍的种种属性其实都是对缓存组进行配置的

## Gzip  
工作原理如下:

1. 首先浏览器（也就是客户端）发送请求时，通过Accept-Encoding带上自己支持的内容编码格式列表；

2. 服务端在接收到请求后，从中挑选出一种用来对响应信息进行编码，并通过Content-Encoding来说明服务端选定的编码信息

3. 浏览器在拿到响应正文后，依据Content-Encoding进行解压。

## tree-shaking 的实现及效果，并总结其难点重点，  
webpack4在--mode 为 production 的情况下会默认进行tree-shaking  


  
**原理：** 
* ES6的模块引入是静态分析的（**静态分析：**就是不执行代码，从字面上对代码进行分析。ES6 之前的模块化，比如我们可以动态 require 一个模块，只有执行后才知道引用的什么模块，这个就不能通过静态分析来做优化，这也是 ES6 Modules 在设计时的一个重要考量，也是为什么没有直接采用 CommonJS，基于这个基础，才使得 tree-shaking 成为可能），故而可以在编译时正确判断到底加载了什么代码。
* 分析程序流，判断哪些变量未被使用、引用，进而删除此代码。


使用tree-shaking主要包含 对于组件库的引用优化；  

**TODO** 哪些是有副作用的？大部分tree-shaking 没有生效的原因就是副作用导致的。  
(webpack-tree-shaking-demo，演示func2在未使用的情况下没有办法tree-shaking，)

**副作用：** 
它大致可以理解成：一个函数会、或者可能会对函数外部变量产生影响的行为。  

在当下，想要合理利用tree-shaking，能尽力做的事：  

* 使用 ES6 模块语法编写代码
* 尽量不写带有副作用的代码。诸如编写了立即执行函数，在函数里又使用了外部变量等
* 工具类函数尽量以单独的形式输出，不要集中成一个对象或者类（举例说明import tool from '../utils/tool' 和 import {cube} from '../utils/tool' 导致的差别）


（结语：其实 webpack 里的 sideEffects: false 的意思并不是我这个模块真的没有副作用，而只是为了在摇树时告诉 webpack：我这个包在设计的时候就是期望没有副作用的，即使他打完包后是有副作用的，webpack 同学你摇树时放心的当成无副作用包摇就好啦！。
要将前端工程的 dead code elimination 做到和其他静态语言一样好, 靠这些工具是远远不够的, 模块自身也必须配合做到符合规范.）


**TODO** * dll 预编译打包原理及效果  

## 准备工作：
1、webpack-base-demo, 演示webpack 模块化打包原理  
2、提前执行build --report ，查看打包的依赖树状图  
3、开启nginx，准备好gzip的build包，演示gzip的效果

## QA   
1、如何将多个css文件打包成一个，就像js模块？
