---
title: 【前端优化】首屏加载 9.2s 压缩至 3.6 s
date: 2021-01-31 22:18:47
tags:
---

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c86cfa2a84a84b2b92f9117fce8000a0~tplv-k3u1fbpfcp-zoom-1.image)

# 背景
随着业务的不断迭代，项目日渐壮大，为了给用户提供更优的体验，前端优化是我们避不开的话题。一个优秀的网站必然是拥有丰富功能的同时具有比较块的响应速度，想必我们浏览网页时都更喜欢丝般顺滑的感受。
作为前端工程师来说，项目优化主要关注一下几点：白屏时间、首屏时间、整页时间、DNS时间、CPU占用率。就我们目前项目而言采用 Angular 开发，首屏加载时间是 9.2 秒，这个速度对用户而言大大降低了使用体验，通过本次优化将首屏加载压缩至 3.6 秒，接下来将逐步阐述本次优化的过程。

# PageSpeed Insights定位问题
其实早在之前就有对项目首页优化的想法，因为前端优化这个概念相对庞大，范围涉及比较广，往往对着项目无从下手。我也是在优化之前阅读并借鉴了很多优秀的文章，最终找到了一个非常好用的工具-PageSpeed Insights for Chrome。这是一个谷歌浏览器插件，可以用来定位网页存在的性能问题，安装好把它加入到 DevTools 就可以开始愉快的找问题了。
![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bd50caa16d4e4ec09e803c614ede34d8~tplv-k3u1fbpfcp-zoom-1.image)
以上图为例可看到 PageSpeed Insights 会把网页存在待优化的问题分类列出，并且每个类目下可以定位到具体的文件，这样一来原本很抽象的前端优化就很形象得呈现在我们眼前了。优化的方向主要分为代码压缩、浏览器缓存、静态资源压缩，方向明确了就可以开始围绕问题开始动手改造。
> 前端优化总的来说还是离不开那句话，减少请求文件数量同时减小请求文件体积。

# 代码压缩
结合前端工程化思想，我姑且把代码压缩分为两大类，一类是项目使用打包工具构建时文件的的压缩，例如 Webpack 各种压缩 Plugin 的配置，另一类是对前端服务器传输响应的压缩配置，例如 Nginx 的 Gzip。
## Webpack 配置
![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8a5de634a37e46f79576769d3713c7f5~tplv-k3u1fbpfcp-zoom-1.image)

由于 Angular-Cli 集成了内部集成了 Webpack，并且重新进行了封装，所以在 Angular 6.0+ 项目中，想要优化或者更改Webpack打包配置需要通过 Angular-Builders，这里需要我们单独安装包。当然如果是直接用Webpack打包的项目就可以直接省略这一步。

```
npm install @angular-builders/custom-webpack --save-dev
npm install @angular-devkit/build-angular --save-dev
```

不需要再单独安装 webpack 和 webpack-dev-server，因为这两个是 @angular-devkit/build-angular 的依赖包，在安装 @angular-devkit/build-angular 会自动安装 webpack 和 webpack-dev-server。

安装完成后要更改 angular.json 文件配置，把我们之后要添加的 webpack 配置，加在运行（serve）和编译（build）命令里，那么关键的配置如下：
```
"architect": {
        ......
        "build": {
            "builder": "@angular-builders/custom-webpack:browser",
            "options": {
                "customWebpackConfig": {
                    "path": "./webpack.config.js"
                }
            }
        },
        "serve": {
            "builder": "@angular-builders/custom-webpack:dev-server",
            "options": {
                "customWebpackConfig": {
                    "path": "./webpack.config.js"
                }
            }
        }
    }
```
接着需要在根目录下创建配置文件，我这里把它命名为 webpack.config.js。到这一步链接 Angular和自定义 Webpack 配置的准备工作已经就位了，现在我们可以把需要的Webpack配置写进新建的文件中。
引入压缩插件，分别对 Js、Html、Css 进行压缩，Js压缩：`UglifyJsPlugin`，Html压缩`HtmlWebpackPlugin`，提取公共资源：`CommonsChunkPlugin`，提取css并压缩：`ExtractTextPlugin`  。

## 配置Nginx压缩
因为我们的前端项目都是通过 Nginx 部署，所以我们还需要在 Nginx 上开启 Gzip 传输压缩，这能使传输的打包文件体积压缩至原来的1/4左右，效果非常明显。
```
gzip on;
gzip_types text/plain application/javascriptapplication/x-javascripttext/css application/xml text/javascriptapplication/x-httpd-php application/vnd.ms-fontobject font/ttf font/opentype font/x-woff image/svg+xml;
```

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c3e4974df57a4eebb4a868374defbd17~tplv-k3u1fbpfcp-watermark.image)

配置好重新启动Nginx，当看到请求响应头中有 Content-Encoding: gzip，说明传输压缩配置已经生效，此时可以看到我们请求文件的大小已经压缩很多。

# 静态资源压缩

## 使用雪碧图
雪碧图的作用就是减少请求数，而且多张图片合在一起后的体积会少于多张图片的体积总和，这也是比较通用的图片压缩方案，推荐一款雪碧图生成工具
[sprite-generator](https://www.toptal.com/developers/css/sprite-generator)
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/45823bedcf9c4c6789b0cdc46c4d176f~tplv-k3u1fbpfcp-watermark.image)

## 使用WebP格式图片
WebP 是 Google 团队开发的加快图片加载速度的图片格式，其优势体现在它具有更优的图像数据压缩算法，能带来更小的图片体积，而且拥有肉眼识别无差异的图像质量，同时具备了无损和有损的压缩模式、Alpha 透明以及动画的特性，在 JPEG 和 PNG 上的转化效果都相当优秀、稳定和统一。
同样推荐一个生成WebP格式的工具
[又拍云WebP](https://www.upyun.com/webp)
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/480a94c2987d44c69df9576a5cfc5954~tplv-k3u1fbpfcp-watermark.image)

## 使用 CDN 加载

内容分发网络是指一种透过互联网互相连接的电脑网络系统，利用最靠近每位用户的服务器，更快、更可靠地将音乐、图片、视频、应用程序及其他文件发送给用户，来提供高性能、可扩展性及低成本的网络内容传递给用户。  
把项目中引用的外部资源包通过 CDN 方式引入，以来能减少项目打包文件体积，二来能提高资源请求速度，一举两得。

# 页面渲染性能优化
## 减少重排重绘
以下三种情况会导致页面重新渲染
* 修改DOM
* 修改样式
* 用户事件
重新渲染就是重新布局和重新绘制，前者叫重排（reflow），后者叫重绘（repaint）。
重排和重绘会影响页面性能以及用户使用体验，所以需要尽量降低重排和重绘的频率，需要注意以下几个方面。
* 多个 DOM 的读操作尽量写在一起，不要在连续读操作中添加写操作。
* 如果样式是重排得到的，尽量缓存样式。
* 不要单一改变样式，最好通过 Class 统一改变。
* 尽量使用离线 DOM 来改变样式，完成后再将对象写入。
* 先将元素置为`display: none`，再对节点进行操作。
* position 为 `absolute` 或 `fixed` 的元素重排开销比较下，因为不考虑其他元素的影响。
* display:none 的节点不会被加入Render Tree, 而 visibility:hidden则会，所以，如果某个节点最开始是不显示的，设为display:none 是更优的。

## 避免CSS、JS阻塞
谈论资源的阻塞时，我们要清楚，现代浏览器总是并行加载资源。例如，当 HTML 解析器被脚本阻塞时，解析器虽然会停止构建 DOM，但仍会识别该脚本后面的资源，并进行预加载。
默认情况下，CSS 被视为阻塞渲染的资源，这意味着浏览器不会渲染任何已处理的内容，直至 CSSOM 构建完毕  
Javascript 不仅可以读取和修改DOM 属性，还可以读取和修改CSSOM 属性
存在阻塞的 CSS 资源时， 浏览器会延迟javascript 的执行和 Render Tree 构建。  
当浏览器遇到一个script标记时，DOM 构建将暂停，直至脚本完成执行。
javascript 可以查询和修改 DOM 与 CSSOM
CSSOM 构建时，javascript 执行将暂停，直至 CSSOM 就绪。
所以，script 标签的位置很重要。实际使用时，可以遵循下面2个原则  
* CSS 资源优于 JS 资源引入  
* JS 应尽量少影响 DOM 的构建

改变 js 阻塞的方式有 defer 和 async  
`<script defer></script>`  
defer 方式加载 script, 不会阻塞 HTML 解析，等到 DOM 生成完毕且 script 加载完毕再执行 Js。
`<script async></script>`  
async 属性表示异步执行引入的 JS，加载时不会阻塞 HTML解析，但是加载完成后立马执行，此时仍然会阻塞 load 事件。

# 总结
前端优化是一个长期的工程，以上也只是部分优化的内容与方向，未来也会保持对前端优化的关注，通过更全面的维度提高项目性能。