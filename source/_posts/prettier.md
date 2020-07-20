---
title: 前端代码格式工具Prettier
date: 2020-05-20 15:36:17
tags:
---

# 背景
考虑到团队新加入的小伙伴越来越多，多人协作开发的规范也得跟上。在此之前我们引入了Angular团队的Git使用规范，获得了不错的反响，接下来就需要对风格迥异的前端代码进行格式规范。
# 工具
prettier是目前前端代码格式化工具中最热门的，在选择插件时也是优先选择了它。我们的方案把prettier集成进代码并且配合着vscode上的prettier插件一同使用。
## 项目引入prettier
首先需要在项目中安装prttier包
```
npm install prettier --save-dev --save-exact
```
第二步在项目根目录下新建一个.prettier配置文件，配置文件可以是JSON、JS、YAML等格式，在这里我们选用的是JSON。然后就可以根据官方文档提供的配置项来制定我们格式规范。
prettier的可配置项不多，可以看[官方文档](https://prettier.io/docs/en/options.html)大致过一遍。
<br>以下是我们的配置（注释只作展示使用，请勿复制进去）:

```
{
  "printWidth": 80, // 超过最大值换行
  "tabWidth": 2, // 缩进字节数
  "useTabs": false, // 不使用缩进符，而使用空格
  "semi": true, // 句末添加分号
  "singleQuote": true, // 使用单引号代替双引号
  "proseWrap": "preserve", // 使用默认的折行标准
  "arrowParens": "avoid", // (x) => {} 箭头函数参数只有一个时是否要有小括号。avoid：省略括号
  "bracketSpacing": true,  // 在对象，数组括号与文字之间加空格 "{ foo: bar }"
  "endOfLine": "auto", // 结尾是 \n \r \n\r auto
  "htmlWhitespaceSensitivity": "css", // 根据显示样式决定 html 要不要折行
  "jsxBracketSameLine": false, // 在jsx中把'>' 是否单独放一行
  "jsxSingleQuote": false, // 在jsx中使用单引号代替双引号
  "trailingComma": "es5" // 在对象或数组最后一个元素后面是否加逗号（在ES5中加尾逗号）
}
```
到这里Prettier的基本配置就完成了，你可以测试一下配置是否生效。

```
prettier --write [file/dir/glob ...]
```
执行命令如果看到文件按配置被格式化，说明你的prettier已经配置成功！
但是每一次格式化都需要手动输入命令触发，这样的体验并不是很完美。
## 自动格式化Pre-commit Hook
prettier官方文档提供了Pre-commit Hook，你可以把git add暂存的文件在被commit提交之前自动执行prettier格式化。

首先安装pretty-quick husky插件

```
npm install pretty-quick husky --save-dev
```
然后在package.json中加入配置

```
{
  "husky": {
    "hooks": {
      "pre-commit": "pretty-quick --staged"
    }
  }
}
```
使用git提交测试文件，可以看到提交的文件自动被格式化。
## Vscode插件  Prettier - Code formatter
我们已经在项目中集成了prettier，并且使用钩子在git提交代码的时候自动格式化相应的文件，但是这样子只能在最后提交之后才能看到代码格式化的效果，很多同学都有使用编辑器格式化代码的习惯，我们也可以在vscode中同步prettier配置。

首先在vscode中安装插件 Prettier - Code formatter

![](https://user-gold-cdn.xitu.io/2020/5/9/171f84d906e7ea5f?w=687&h=158&f=png&s=19094)

然后在vscode的setting.json中引入prettier配置项

```
"prettier": {
  "printWidth": 80,
  "tabWidth": 2,
  "useTabs": false,
  "semi": true,
  "singleQuote": true,
  "proseWrap": "preserve",
  "arrowParens": "avoid",
  "bracketSpacing": true,
  "endOfLine": "auto",
  "htmlWhitespaceSensitivity": "css",
  "jsxBracketSameLine": false,
  "jsxSingleQuote": false,
  "trailingComma": "es5"
},
```
大功告成，此时vscode的格式化配置已经与我们项目集成的prettier同步了，接下来就可以愉快的编码了~
# 结束语
在多人协作的开发过程中有强规范来约定开发格式能够提升开发效率和易维护性。随着项目越来越壮大，需要优化和规范的点还非常多，欢迎伙伴们一起交流~






