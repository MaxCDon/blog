---
title: 阿里云ARMS前端监控Angular接入实战
date: 2020-11-29 17:11:01
tags:
---

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6dc3e304a283430d83d372b8fb50faa0~tplv-k3u1fbpfcp-watermark.image)

# 为什么要引入前端监控？
用户访问业务时，整个访问过程大致可以分为三个阶段：页面生产时（服务器端状态）、页面加载时和页面运行时。为了保证线上业务稳定运行，我们会在服务器端对业务的运行状态进行各种监控。现有的服务器端监控系统相对已经很成熟，而页面加载和页面运行时的状态监控一直比较欠缺。例如：
* 无法第一时间获知用户访问您的站点时遇到的错误。
* 各个国家、各个地区的用户访问您的站点的真实速度未知。
* 每个应用内有大量的异步数据调用，而它们的性能、成功率都是未知的。
随着前端项目工程化得发展，我们对前端项目的性能要求也越来越高，如果没有引入前端监控，团队对前端线上环境的运行状况是很抽象的。在之前的工作中，经常遇到用户在使用业务得过程中出现异常，此时无法很快得定位并复现用户遇到的问题，导致在维护中消耗了较大得工作量。如果有前端监控得存在我们就可以快速定位到异常并做出修复，能够大幅提升工作效率以及用户体验。

# 选择一款优秀的前端监控
挑选前端监控之前，首先确定项目对监控得需求，在我们项目中需求大致分为几点：
* JS异常信息统计
* 页面加载效率分析
* API接口调用信息统计
* 监控可视化呈现
针对这些需求，就可以有目标性地挑选一款符合项目的前端监控。目前开源的Sentry和阿里云团队的ARMS都是比较全面的监控，基本满足我们项目的需求。
## Sentry
正如其名「哨兵」，可以实时监控生产环境上的系统运行状态，一旦发生异常会第一时间把报错的路由路径、错误所在文件等详细信息以邮件形式通知我们，并且利用错误信息的堆栈跟踪快速定位到需要处理的问题。

优点：开源对各种前端框架的友好支持 (Vue、React、Angular)支持 SourceMap。Sentry 官方提供的免费服务有次数限制，达到一定限制后继续使用就需要收费，但是我们可以利用 Sentry 的开源库在自己的服务器上搭建服务，官方已经提供了完善的操作文档。

## ARMS
ARMS前端监控专注于对Web场景、Weex场景和小程序场景的监控，从页面打开速度（测速）、页面稳定性（JS Error）和外部服务调用成功率（API）这三个方面监测Web和小程序页面的健康度。

优点：重点监控页面的加载过程和运行时状态，同时将页面加载性能、运行时异常以及API调用状态和耗时等数据，上报到日志服务器。之后借助ARMS提供的海量实时日志分析和处理服务，对当前线上所有真实用户的访问情况进行监控。最后通过直观的报表展示。

可以看出Sentry更加专注于异常信息处理，ARMS则从更多的维度统计系统信息，并且通过丰富得可视化呈现，由此看来ARMS在这个项目中是更优得选择。

# Angular项目接入ARMS监控
在选定监控框架后，就要开始着手接入项目了，ARMS文档有两种接入方式提供给我们选择:

## cdh方式为Web应用安装ARMS前端监控探针
cdh的方式比较便捷，在阿里云后台创建好项目后，直接复制ARMS提供的`<script>`标签中代码至根页面的`<body>`标签中即可生效。cdh方式又分为异步加载和同步加载。
* 异步加载：又称为非阻塞加载，表示浏览器在下载执行JS之后还会继续处理后续页面。若对页面性能的要求非常高，建议使用此方式。
* 同步加载：又称为阻塞加载，表示当前JS加载完毕后才会进行后续处理。如需捕捉从页面打开到关闭的整个过程中的JS错误和资源加载错误，建议cdh引入使用此方式。

## npm方式为Web应用安装ARMS前端监控探针
我们项目中选择npm方式安装探针

在npm仓库中安装alife-logger

``` node
npm install alife-logger --save
```

SDK以`BrowserLogger.singleton`方式初始化

arms.ts
``` javascript
import BrowserLogger from 'alife-logger';
import { environment } from 'src/environments/environment';

/**
 * ARMS前端监控
 */
let logger = null;
let environmentType = '';

switch (environment.type) {
  case 'dev':
    environmentType = 'daily';
    break;
  case 'local':
    environmentType = 'local';
    break;
  case 'pre':
    environmentType = 'pre';
    break;
  case 'rep':
    environmentType = 'gray';
    break;
  case 'prod':
    environmentType = 'prod';
    break;
  default:
    break;
}

try {
  if (environment.type !== 'local') {
    logger = BrowserLogger.singleton({
      pid: 'd465464j9@4caac5459*****', // 项目唯一ID，由ARMS在创建站点时自动生成
      appType: 'web', // 项目类型
      imgUrl: 'https://arms-retcode.aliyuncs.com/r.png?', // 日志上传地址
      sendResource: true, // 上报页面静态资源
      enableLinkTrace: true, // 进行前后端链路追踪
      behavior: true, // 是否为了便于排查错误而记录报错的用户行为
      enableSPA: true, // 监听页面的hashchange事件并重新上报PV
      useFmp: true, // 采集首屏FMP
      environment: environmentType, // 当前环境
      release: environment.version, // 项目版本号
    });
  }
} catch (e) {
  console.error('init arms fail', e);
}

export default logger;

```

在页面初始化的时候引入`arms.ts`重启项目就可以看到探针已经生效。
![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2066816384bd474c94600488c7126456~tplv-k3u1fbpfcp-watermark.image)
## Angular自定义异常处理
当用户进入项目时ARMS就会把前端监控日志上报，在ARMS后台可以看到访问速度、UV和PV统计、API请求日志信息。
到这里原本以为一切搞定了，但是自测过程中发现JS错误信息无法自动上报，我立刻再去看了一遍ARMS的接入文档，发现文档上也没有关于Anuglar项目的接入教程。然后开始各种Goole也没有找到关于Angular接入ARMS的攻略，最后我只能加了官方钉钉群去咨询开发团队，得到了以下解答。

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/144caeda666e446380cbe8eeaaae403c~tplv-k3u1fbpfcp-watermark.image)

根据这个线索，我终于定位到JS异常没有上报的原因，是因为Angular框架底层对JS抛出的异常进行统一捕获处理，所以ARMS无法监听到JS异常，我们需要通过ARMS提供的`error()`方法手动上报异常信息。

`error()`

调用error()接口来上报页面中的JS错误或使用者想关注的异常。一般情况下，SDK会监听页面全局的Error并调用此接口上报异常信息，但由于浏览器的同源策略往往无法获取错误的具体信息，此时就需要使用者手动上报。

``` javascript
arms.error(error, pos)
```

| 参数 | 类型 | 描述 | 是否必选 | 默认值 |
|-----|-----|------|---------|-------|
| error |  |Error | JS的Error对象 | 是 | 无 |
| pos | Object | 错误发生的位置，包含以下3个属性 | 否 | 无 |
| pos.filename | String  | 错误发生的文件名 | 否 | 无 |
| pos.lineno | Number | 错误发生的行数 | 否 | 无 |
| pos.colno | Number | 错误发生的列数 | 否 | 无 |

Angular提供了钩子让我们自定义异常处理，我们要做的就是创建一个类，该类实现`ErrorHandler`来自`@angular/core`程序包的接口。该类必须实现`handleError()`方法。

``` javascript
import { ErrorHandler } from '@angular/core';
import arms from './arms';
import { environment } from 'src/environments/environment';

/**
 * 全局异常处理
 */
export class GlobalErrorhandler implements ErrorHandler {
  handleError(error) {
    switch (environment.type) {
      case 'prod':
        arms.error(error);
        break;
      case 'local':
        console.error('Error from global error handler', error);
        break;
      default:
        console.error('Error from global error handler', error);
        arms.error(error);
        break;
    }
  }
}
```
然后在`app.modules.ts`，告诉Angular它应该使用我们的错误处理程序，可以通过在providers配置中添加以下条目来实现
``` javascript
providers: [
  { provide: ErrorHandler, useClass: GlobalErrorhandler },
],
```
部署项目可以看到异常信息可以上报至ARMS后台，大功告成~

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9d75f559944f4573a0a9b05d65291e10~tplv-k3u1fbpfcp-watermark.image)

# 最后
感谢观看，若有不足望提出指正！