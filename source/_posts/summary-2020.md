---
title: 2020黑天鹅后学会拥抱变化
date: 2020-12-18 10:10:09
tags:
---

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/60f5771c0bfe434c9a33d9a70746a2c4~tplv-k3u1fbpfcp-watermark.image)

# 前言
仔细算来从事前端也有两年半之久，时间好快，并且感觉越来越快，因此也常常出现焦虑。这就像查理·芒格老爷子所说 

> 人们总是担心时间过去得太快，而自己又聪明得太慢

生活在如此快节奏的环境中，特别是在一线城市竞争异常激烈（杭州在今年荣升为新一线城市），在拼搏的同时当然也需要学会慢下来思考与总结。一直以来也没有好好把年度总结以文字得形式记录下来，那就从今年开始在这里记录下自己的经历和成长。

# “黑天鹅”事件
过去一年多发生了几件比较重大的事，2019年末，在我进入社会初期，当时所在行业经历了比较大得动荡，遇到了职业生涯的一次“黑天鹅”事件。当时那段时间内心也比较郁闷，但是有一句话怎么说来着，当你从低谷中走出来来的每一步都会是你未来宝贵的经验，之后我也尽快调整好了自己的状态。2020年初，新冠疫情席卷全球，这样全球性的“黑天鹅”事件，对全世界都是极大的考验，但每一个地方都在积极抗疫，我在那时报名成为社区志愿者，输出了一份绵薄之力。  
经历了社会中这些的洗礼，我也有了更深刻得认知，生活中总会充斥着一些不可抗拒的“黑天鹅”，我们能做的只有不断提高自己的抗风险能力，以保证有足够的勇气去拥抱变化。

# 这一年做了什么
## 新的平台
经过一段时间的沉淀与尝试，我也开始迎接新的挑战。回望这一段时间，自己也是有很多收获，对前端的知识体系进行了系统得梳理与巩固，找到了自己得不足，让我在接下来的一年有更针对性的方案去学习补充。
## 尝试写文章记录自己
这一年中我尝试分享自己的成长轨迹，写文章确实是一种很好的总结方式，在这个过程中能够对所学知识进行复盘，每次看到自己写的文字还是会有小小的成就感。目前写的文章主要还是以自己记录为主，以后希望能够输出一些更高质量的文章，能够给他人分享有用的信息会是一件很有意义的事。
## 拥有人生中第一辆车
这一年中，我拥有了人生中第一辆车，在周末带家人出去聚个餐，节假日约上三五好友来一场说走就走的自驾游，这些日常小事能给生活增添许多幸福感。但当我还沉浸在提车的喜悦之时，发现周围很多朋友都已经开始买房，大家都步履不停，自己也得迎头赶上才行。

# 成长与收获
## 推动团队发展
跟过去相比，今年最大的改变就是从跟随团队一起发展，到如今推动团队共同进步。当我加入公司的第一天起，我就给自己定位为一名推动者，希望自己的加入不单单是分担一份工作量，而是能成为为团队赋能的角色。如何在工作中体现价值？这是疫情期间经常思考得一个问题，我想能够推动团队和产品变得更有价值，那就是自我价值最好得佐证。制定好目标是第一步，更重要得是执行，在新团队的一年中除了日常业务迭代之外，我还主导推动了一些优化。

## 制定Git使用规范
我加入团队时正值快速扩张时期，发现前端代码库中`Commit Message`和`Git分支管理`没有一个严格的规范，伴随着人数增加会必然会影响 Code Review 和版本分支管理效率，于是我推动团队使用 Angular 的 `Commit Message` 规范，并且优化的分支管理，保证团队能多任务线并行开发使用 Git 仓库，也提高了代码仓库安全性和可维护性。

## 引入 Prettier 和 Eslint 规范代码格式
在统一 Git 使用格式之后，编码规范得统一也得跟上，在提升个人的编码习惯同时还能给之后重构代码的人提供极大得便利， 毕竟作为开发者谁都不希望某一天接盘一堆难以阅读的代码，这一点应该能引起很多共鸣吧。于是我在项目中引入 Prettier 和 Eslint 配合使用约定编码规范，通过 husky 监听 git commit 自动格式代码以及 ts 语法检测，以此保证每次提交到远程仓库的代码都是符合规范的。

## 首页加载时间优化
在面向的客户的 WEB 项目中，页面加载渲染的时间直接影响到产品的使用体验，期间也对项目首页加载时间进行了一次优化。通过静态资源优化（使用 WebP 格式图片以及雪碧图）、Nginx 服务器开启 Gzip 压缩、使用浏览器缓存、使用 CDN 加载等方式，将页面首次加载时间从9.3秒提升至3.5秒。这里推荐一个好用的页面性能检测 Chrome 插件 PageSpeed，当前端优化无从下手时，这个工具可以帮助我们检测页面性能还存在哪些不足，然后就可以按提示一步步优化。

## 接入ARMS前端监控
伴随着项目的成长，对线上环境得代码运行状况监控也是必要的，能够帮助我们提前发现问题并及时修复，保证线上环境的健康运行。我选择在项目中接入阿里云团队的 ARMS 前端监控，实时可视化监控多环境JS异常信息、页面加载效率、API接口调用信息，并且接入钉钉机器人接收异常警报，保证团队能够第一时间接收到报警信息。

# 拓宽自己的视野
平时也常常会有焦虑，今年买了一些书籍送给自己，希望通过跟伟大得灵魂进行碰撞，能拓宽视野、提升认知。在阅读《穷查理宝典》时，查理的思维很多次让我感觉醍醐灌顶，价值投资的魅力也深深吸引我。在休闲放松时，最近也爱看《圆桌派》这个节目，里面前辈们探讨的话题往往就是我们生活中困扰着我们的事，他们的经验也能够为我提供一个新的角度。

# 强大的内心也需要强健得体魄
要说今年比较满意的事，那就是号召四位好友开始一起健身。以前也办过健身卡，但都没能坚持下来，这一次从十月份开始约定每周三天健身日，风雨无阻。每次想偷懒的时候就在内心狠狠骂自己，所以至今为止还没有缺卡记录，希望能够一直保持下去。

# 给2021的自己定几个小目标
* 每月发表一篇技术文章
* 看完六本新书
* 坚持一周三次健身 
目标不大，定要好好执行  
俗话说流水不争先，争得是滔滔不绝

# 未来可期
去坚持自己认可的事  
去争取自己想要的东西  
2021 共勉！