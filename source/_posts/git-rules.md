---
title: 【Git分支管理】配合阿里云云效流水线实现前端CI/DI
date: 2021-03-31 15:23:40
tags:
---

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/35b9ca4972f7459986df306882fd407a~tplv-k3u1fbpfcp-watermark.image)

# 分支类型

* mater 分支：主分支也是保护分支，用于部署生产环境和预发环境，只能由 release 分支、hotfix 分支合并，为保证主分支的稳定性任何人都无法在此分支直接提交代码。

* develop 分支：开发分支同样也是保护分支，保持最新开发完成代码的同步。

* feature 分支：新功能的开发分支，基于 develop 分支创建，功能开发完成后合并入 develop 分支，命名格式  feature/module_user_date

* release 分支：提测分支，基于 develop 分支创建，测试过程中的bug修复均在此分支提交，测试完成后合入 master 分支，命名格式 release/module_user_date

* hotfix 分支：紧急修复分支，基于 master 分支创建，用于修复线上紧急 bug 和 开发紧急需求，完成后合入 master 分支，命名格式  hotfix/module_user_date

# 部署方式

* 联调环境：云效流水线点击运行选择远程分支部署

* 测试环境：云效流水线点击运行选择远程分支部署

* 预发环境：监听远程仓库 master 分支更新，自动触发部署

* 生产环境：监听远程仓库 master 分支更新，并且需要云效流水线点击确认部署

* 演示环境：监听远程仓库 master 分支更新，并且需要云效流水线点击确认部署

# 开发流程

## 正常排期流程

### 开发阶段
从 develop 分支检出 feature/module_user_date 功能分支进行开发，  
开发完成后功能分支 merge 合入 develop 分支  

### 提测阶段
从 develop 分支检出 release/module_user_date 提测分支
提测过程中在 release/module_user_date 分支提交代码  
更新代码后通过云效流水线点击运行选择分支部署测试环境


### 预发阶段
将测试通过的 release/module_user_date 提测分支 merge 合入 master分支  
自动触发预发环境部署
此时修复 bug 从 master 分支检出 hotfix/module_user_date 修复分支
修复后合回 master 分支

### 生产阶段
验收后的版本通过云效流水线手动触发生产环境、演示环境部署  
将 master 分支合入 development 分支  
并且删除远程仓库 feature/module_user_date、release/module_user_date、hotfix/module_user_date 三类分支

END

## 紧急需求/紧急修复流程

### 开发阶段
从 master 分支检出 hotfix/module_user_date 紧急功能分支进行开发

### 提测阶段
将多个 hotfix/module_user_date 分支合并成一个
提测过程中在 hotfix/module_user_date 分支提交代码  
更新代码后通过云效流水线手动选择分支部署测试环境

### 预发阶段
将测试通过的 hotfix/module_user_date 提测分支 merge 合入 master分支  
自动触发预发环境部署
此时修复 bug 从 master 分支检出 hotfix/module_user_date 修复分支
修复后合回 master 分支

### 生产阶段
验收后的版本通过云效流水线手动触发生产环境、演示环境部署  
将 master 分支合入 development 分支  
并且删除远程仓库 hotfix/module_user_date 分支

END
