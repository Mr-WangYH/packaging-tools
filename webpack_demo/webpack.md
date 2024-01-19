# webpack
会以一个或多个文件作为打包的入口，将我们整个项目所有文件编译组合成一个或多个文件输出出去

本身只能处理js、json资源

将webpack输出的文件叫做bundle

## 高级优化

### SourceMap
SourceMap(源代码映射)是一种用来生成源代码与构建后代码一一映射的文件的方案

### hotModuleReplacement
HotModuleReplacement (HMR/热模块替换): 在程序运行中，替换、添加或删除模块，而无需重新加载整个页面。

### oneOf
打包时每个文件都会经过所有 loader 处理，虽然因为 test 正则原因实际没有处理上，但是都要过一遍。比较慢。

加上oneOf后，文件只要被一个loader处理后，就不会走其他loader

### include/exclude
include: 包含，只处理***文件
exclude: 排除，除***文件，其它文件都处理

### cache
每次打包时 js 文件都要经过 Eslint 检查和 Babel 编译，速度比较慢。我们可以缓存之前的 Eslint 检查和 Babel 编译结果，这样第二次打包时速度就会更快了

### thread
- 为什么
当项目越来越庞大时，打包速度越来越慢，甚至于需要一个下午才能打包出来代码。这个速度是比较慢的。
我们想要继续提升打包速度，其实就是要提升js 的打包速度，因为其他文件都比较少。而对js 文件处理主要就是 eslint、babel、Terser 三个工具，所以我们要提升它们的运行速度。我们可以开启多进程同时处理js 文件，这样速度就比之前的单进程打包更快了。

- 是什么
多进程打包:开启电脑的多个进程同时干一件事，速度更快。
需要注意:请仅在特别耗时的操作中使用，因为每个进程启动就有大约为 600ms 左右开销.

获取电脑CPU核数
```js
const os = require('os');
const threads = os.cpus().length;
```

### tree shaking
通常用于描述移除javascript中没有用上的代码 (默认开启此功能)
注意： 它依赖ES module

### babel
- 为什么
Babel 为编译的每个文件都插入了辅助代码，使代码体积过大!
Babel 对一些公共方法使用了非常小的辅助代码，比如extend。默认情况下会被添加到每一个需要它的文件中。
你可以将这些辅助代码作为一个独立模块，来避免重复引入。

- 是什么
@babel/plugin-transform-runtime: 禁用了 Babel 自动对每个文件的 runtime 注入，而是引入 @babel/plugin-transform-runtime 并且使所有辅助代码从这里引用。

### image-minimizer-webpack-plugin
压缩图片体积

### code split


### core-js


### PWA
渐进式网络应用程序(progresive web application - PWA): 是一种可以提供类似于 native app(原生应用程序) 体验的 Web App 的技术,其中最重要的是，在离线(offline) 时应用程序能够继续运行功能。内部通过 Service Workers 技术实现的.


## 总结
我们从 4 个角度对 webpack 和代码进行了优化:
1.提升开发体验
- 使用 Source Map 让开发或上线时代码报错能有更加准确的错误提示,

2.提升 webpack 提升打包构建速度
- 使用 HotModuleReplacement 让开发时只重新编译打包更新变化了的代码，不变的代码使用缓存，从而使更新速度更快。使用 0neof 让资源文件一旦被某个 loader 处理了，就不会继续遍历了，打包速度更快。
- 使用 Include/Exclude 排除或只检测某些文件，处理的文件更少，速度更快。
- 使用 cache 对 eslint和 babel处理的结果进行缓存，让第二次打包速度更快.
，使用 head 多进程处理 eslint和 babe 任务，速度更快。(需要注意的是，进程启动通信都有开销的，要在比较多代码处理时使用才有效
果)

3.减少代码体积
使用 Tree shaking 剔除了没有使用的多余代码，让代码体积更小。
- 使用 @babel/plugin-transfori-runtime 插件对 babel 进行处理，让辅助代码从中引入，而不是每个文件都生成辅助代码，从而体积更小。·使用 Image Minimizer 对项目中图片进行压缩，体积更小，请求速度更快。(需要注意的是，如果项目中图片都是在线链接，那么就不需要了。本地项目静态图片才需要进行压缩。)

4.优化代码运行性能
使用 code Split 对代码进行分割成多个j 文件，从而使单个文件体积更小，并行加载js 速度更快。并通过 import 动态导入语法进行按需加载，从而达到需要使用时才加载该资源，不用时不加载资源。
- 使用 Preload / Prefetch 对代码进行提前加载，等未来需要使用时就能直接使用，从而用户体验更好.
- 使用 Network Cache 能对输出资源文件进行更好的命名，将来好做缓存，从而用户体验更好。
- 使用 core-js 对js 进行兼容性处理，让我们代码能运行在低版本浏览器。
- 使用，PWA 能让代码离线也能访问，从而提升用户体验。



## Loader 原理

### loader 概念
帮助 webpack将不同类型的文件转换为 webpack 可识别的模块。

### loader 执行顺序

1.分类
pre: 前置 loader
普通 loadelnormal:
inline:内联 loadero
post:后置 loader

2.执行顺序
- 4类loader的执行优级为: pre >normal >inline >post 。
- 相同优先级的 loader 执行顺序为: 从右到左，从下到上。

### loader原理
其实就是一个函数，当webpack解析资源的时候会调用相应的loader去处理
```js
/**
 * content 源文件内容
 * map courceMap 数据
 * meta 别的loder传递的数据
 * 
*/
function(content,map,meta){}
```

分类：
- 同步loader （不能执行异步代码，会报错）
```js
/**
 * err 代表是否有错误
 * content 处理后的内容
 * map courceMap继续传递courceMap
 * meta 给下一个loader传递参数
*/
module.exports = function(content,map,meta){
  this.callback(err,content,map,meta)
}
```
- 异步loader
```js
module.exports = function(content,map,meta){
  const callback = this.async()
  setTimeout = (() => {
    callback(err,content,map,meta)
  },1000)
}
```
- raw loader 
```js
module.exports = function(content,map,meta){
  // 接收到的content是一个buffer数据
  this.callback(err,content,map,meta)
}
module.exports.raw = true
```
- pitch loader 
```js
module.exports = function(content,map,meta){
  // 接收到的content是一个buffer数据
  this.callback(err,content,map,meta)
}
// 该方法会在所有loader之前执行
module.exports.pitch = function() {
  // return会中断后面loader的执行
  return
}
```

### loader api
|方法名  |含义    |用法    |
|-------|-------|--------|
|this.async | 异步回调 loader。返回 this.callback | const callback = this.async()  |
|this.callback  | 可以同步或者异步调用的并返回多个结果的函数  | this.callback(err, content, sourceMap?, meta?)  |
|this.getOptions(schema)  | 获取 loader的 options | this.getOptions(schema) |
|this.emitFile  | 产生一个文件  | this.emitFile(name, content, sourceMap) |
|this.utils.contextify  | 返回一个相对路径  | this.utils.contextify(context, request) |
|this.utils.absolutify  | 返回一个绝对路径  | this.utils.absolutify(context, request) |


### 什么是Webpack的热更新（Hot Module Replacement）？原理是什么？
Webpack的热更新（Hot Module Replacement，简称HMR），在不刷新页面的前提下，将新代码替换掉旧代码。
HRM的原理实际上是 webpack-dev-server（WDS）和浏览器之间维护了一个websocket服务。当本地资源发生变化后，webpack会先将打包生成新的模块代码放入内存中，然后WDS向浏览器推送更新，并附带上构建时的hash，让客户端和上一次资源进行对比。客户端对比出差异后会向WDS发起Ajax请求获取到更改后的内容（文件列表、hash），通过这些信息再向WDS发起jsonp请求获取到最新的模块代码。


### 什么是Code Splitting？
Code Splitting代码分割，是一种优化技术。它允许将一个大的chunk拆分成多个小的chunk，从而实现按需加载，减少初始加载时间，并提高应用程序的性能。
通常Webopack会将所有代码打包到一个单独的bundle中，然后在页面加载时一次性加载整个bundle。这样的做法可能导致初始加载时间过长，尤其是在大型应用程序中，因为用户需要等待所有代码加载完成才能访问应用程序。
Code Splitting 解决了这个问题，它将应用程序的代码划分为多个代码块，每个代码块代表不同的功能或路由。这些代码块可以在需要时被动态加载，使得页面只加载当前所需的功能，而不必等待整个应用程序的所有代码加载完毕。
在Webpack中通过optimization.splitChunks配置项来开启代码分割。


### Webpack的Tree Shaking原理
Tree Shaking 也叫摇树优化，是一种通过移除多于代码，从而减小最终生成的代码体积，生产环境默认开启。
原理：

- ES6 模块系统：Tree Shaking的基础是ES6模块系统，它具有静态特性，意味着模块的导入和导出关系在编译时就已经确定，不会受到程序运行时的影响。
- 静态分析：在Webpack构建过程中，Webpack会通过静态分析依赖图，从入口文件开始，逐级追踪每个模块的依赖关系，以及模块之间的导入和导出关系。
- 标记未使用代码： 在分析模块依赖时，Webpack会标记每个变量、函数、类和导入，以确定它们是否被实际使用。如果一个导入的模块只是被导入而没有被使用，或者某个模块的部分代码没有被使用，Webpack会将这些未使用的部分标记为"unused"。
- 删除未使用代码: 在代码标记为未使用后，Webpack会在最终的代码生成阶段，通过工具（如UglifyJS等）删除这些未使用的代码。这包括未使用的模块、函数、变量和导入。


### 如何提高webpack的打包速度
- 利用缓存：利用Webpack的持久缓存功能，避免重复构建没有变化的代码。可以使用cache: true选项启用缓存。
- 使用多进程/多线程构建 ：使用thread-loader、happypack等插件可以将构建过程分解为多个进程或线程，从而利用多核处理器加速构建。
- 使用DllPlugin和HardSourceWebpackPlugin： DllPlugin可以将第三方库预先打包成单独的文件，减少构建时间。HardSourceWebpackPlugin可以缓存中间文件，加速后续构建过程。
- 使用Tree Shaking: 配置Webpack的Tree Shaking机制，去除未使用的代码，减小生成的文件体积
- 移除不必要的插件: 移除不必要的插件和配置，避免不必要的复杂性和性能开销。


### 如何减少打包后的代码体积
- 代码分割（Code Splitting）：将应用程序的代码划分为多个代码块，按需加载。这可以减小初始加载的体积，使页面更快加载。
- Tree Shaking：配置Webpack的Tree Shaking机制，去除未使用的代码。这可以从模块中移除那些在项目中没有被引用到的部分。
- 压缩代码：使用工具如UglifyJS或Terser来压缩JavaScript代码。这会删除空格、注释和不必要的代码，减小文件体积。
- 使用生产模式：在Webpack中使用生产模式，通过设置mode: 'production'来启用优化。这会自动应用一系列性能优化策略，包括代码压缩和Tree Shaking。
- 使用压缩工具：使用现代的压缩工具，如Brotli和Gzip，来对静态资源进行压缩，从而减小传输体积。
- 利用CDN加速：将项目中引用的静态资源路径修改为CDN上的路径，减少图片、字体等静态资源等打包。


### vite比webpack快在哪里
他们都是前端构建工具，但vite构建速度相对于webpack还是有一些速度优势

- 冷启动速度：vite是利用浏览器的原生ES moudle，采用按需加载的当时，而不是将整个项目打包。而webpack是将整个项目打包成一个或多个bundle，构建过程复杂。
- HMR热更新： vite使用浏览器内置的ES模块功能，使得在开发模式下的热模块替换更加高效，那个文件更新就加载那个文件。它通过WebSocket在模块级别上进行实时更新，而不是像Webpack那样在热更新时重新加载整个包。
- 构建速度： 在生产环境下，Vite的构建速度也通常比Webpack快，因为Vite的按需加载策略避免了将所有代码打包到一个大文件中。而且，Vite对于缓存、预构建等方面的优化也有助于减少构建时间。
- 缓存策略： Vite利用浏览器的缓存机制，将依赖的模块存储在浏览器中，避免重复加载。这使得页面之间的切换更加迅速。
- 不需要预编译： Vite不需要预编译或生成中间文件，因此不会产生大量的临时文件，减少了文件IO操作，进一步提升了速度。
