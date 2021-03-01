---
title: 【前端E2E测试】上手Protractor 看这篇就够了
date: 2021-02-28 23:07:39
tags:
---

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3478ff659a0a4b438865ec46559e6c40~tplv-k3u1fbpfcp-watermark.image)

# 关于 Protractor
介绍 Protractor之前需要先介绍一下端到端测试（E2E测试），端到端测试验证应用中的所有层。这不仅包括你的前端代码，还包括所有相关的后端服务和基础设施，它们更能代表你的用户所处的环境。通过测试用户操作如何影响应用，端到端测试通常是提高应用是否正常运行的信心的关键。  
Protractor 是 Angular 和 AngularJS 应用程序的端到端测试框架，针对在真实浏览器中运行的应用程序运行测试，并与用户进行交互，具有以下特性。
## 模拟用户真实测试流程
Protractor 建立在 WebDriverJS 之上，该 WebDriverJS 使用本机事件和特定于浏览器的驱动程序来与用户的应用程序进行交互。
## Angular 支持友好
Protractor 支持特定于Angular的定位器策略，该策略使你无需进行任何设置即可测试特定于Angular的元素。
## 自动等待
不再需要为测试添加等待和休眠，Protractor 可以在网页完成待处理的任务后自动执行测试的下一步，因此你不必担心等待测试和网页同步。

# 安装配置
## 安装 Node 
Protractor 是一个  Node.js 项目，你需要先安装 Node 环境，这个前端的小伙伴相信都有，这里也还是贴一下 [Node 安装教程](https://www.runoob.com/nodejs/nodejs-install-setup.html)。
## 安装 Jdk
由于官方文档教程将运行本地独立的 Selenium 服务器来控制浏览器进行测试，你将需要安装 Java 开发工具包（JDK）才能运行独立的Selenium 服务器。这个是 [Jdk安装教程](https://www.runoob.com/java/java-environment-setup.html) , 安装完成后可以通过从命令行运行 `java -version` 进行检查是否生效。
## 安装 Protractor 
完成基础环境的配置，接下来就可以全局安装 Protractor 框架主体
```
npm install -g protractor
```
这个命令会安装两个工具， `Protractor` 和 `webdriver-manager`，安装后使用命令行 `protractor --version`检测是否生效。  
`webdriver-manager` 是一个辅助工具，可轻松获取正在运行的 Selenium Server 实例。你需要通过以下命令下载必要的二进制文件：
```
webdriver-manager update
```
现在就可以启动服务
```
webdriver-manager start
```
这将启动 Selenium 服务器并输出一堆信息日志, 你的 Protractor 测试将向该服务器发送请求以控制本地浏览器。保持该服务器运行，你可以在 `http://localhost:4444/wd/hub` 上查看有关服务器状态的信息。

# 官方用例
创建一个空的项目文件夹，并在其中创建两个文件, 一个测试代码文件`spec.js`，一个配置文件 `conf.js`。  
让我们从一个简单的测试开始，该测试跳转到示例 AngularJS 应用程序并检查其标题，这是示例“超级计算器”测试页面地址：[http://juliemr.github.io/protractor-demo/](http://juliemr.github.io/protractor-demo/)  
```javascript
// spec.js
describe('Protractor Demo App', function() {
  it('should have a title', function() {
    browser.get('http://juliemr.github.io/protractor-demo/');

    expect(browser.getTitle()).toEqual('Super Calculator');
  });
});
```

`describe` 和 `it` 语法来自 `Jasmine` 框架， `browser`是 Protractor 创建的全局变量，用于执行浏览器级别的命令，例如使用`browser.get` 进行导航。

```javascript
// conf.js
exports.config = {
  framework: 'jasmine',
  seleniumAddress: 'http://localhost:4444/wd/hub',
  specs: ['spec.js']
}
```
这个配置是告诉 Protractor 选用的测试框架（`framewrok`）、测试文件的名称（`spec`）以及 selenuim server 地址（`selenuimAddress`）。它指定我们将使用 Jasmine 测试框架，对所有其他配置使用默认设置，Chrome是默认浏览器。  

运行 Protractor 进行测试  
```protractor conf.js```  
你应该会看到一个Chrome浏览器窗口打开并导航到“计算器”，然后自行关闭。测试输出应为1个测试，1个断言，0个失败。恭喜，你已经进行了首次 protractor 测试！


# 语法解析
网页的端到端测试的核心是查找 DOM 元素，与之交互以及获取有关应用程序当前状态的信息，以下是如何使用 Protractor 在 DOM 元素上定位和执行操作的基本语法。
## 定位器（Locators）
```javascript
// 通过 class 查找元素
by.css('.myclass')

// 通过 id 查找元素
by.id('myid')

// 通过 name 查找元素
by.name('field_name')

// 通过 ng-model 查找元素
// 请注意，目前仅AngularJS应用支持此功能
by.model('name')

// 查找绑定到给定变量的元素
// 请注意，目前仅AngularJS应用支持此功能
by.binding('bindingname')
```
定位器传递给 element 函数如下：
```javascript
element(by.css('some-css'));
element(by.model('item.name'));
element(by.binding('item.name'));
```
使用 CSS 选择器作为定位器时，可以使用`$（）`快捷方式表示法：
```javascript
$('my-css');

// 这相当于:

element(by.css('my-css'));
```

## 触发事件（Action）
`element（）`函数返回一个ElementFinder对象， ElementFinder知道如何使用您作为参数传入的定位器来定位DOM元素，但实际上尚未这样做，在调用动作方法之前它不会与浏览器联系。 以下是最常见的触发方法：
### 单个元素
```javascript
var el = element(locator);

// 单击元素
el.click();

// 将键值输入到元素（通常是 Input 框）。
el.sendKeys('my text');

// 清除元素的值 (通常是 Input 框).
el.clear();

// 获取 attribute 的值, 例如获取 Input 的 value
el.getAttribute('value');
```

由于所有动作都是异步的，因此所有动作方法都返回一个Promise，因此要记录元素的文本，可以执行以下操作：
```javascript
var el = element(locator);
el.getText().then(function(text) {
  console.log(text);
});
```
> WebFinder 上 WebDriverJS 中可用的任何操作都可以在 ElementFinder 中使用，[查看完整清单](http://www.protractortest.org/#/api?view=webdriver.WebElement)。

### 元素列表
要处理多个DOM元素，使用element.all函数，定位器作为其唯一参数。
```javascript
element.all(by.css('.selector')).then(function(elements) {
  // 查找器获取的是数组元素
});
```
element.all() 具有几个辅助方法：
```javascript
// 元素的数量
element.all(locator).count();

// 根据 Index 获取元素
element.all(locator).get(index);

// 第一个和最后一个元素
element.all(locator).first();
element.all(locator).last();
```
将 CSS 选择器用作定位器时，可以使用快捷方式 `$$（）`表示法：
```javascript
$$('.selector');

// 这相当于:

element.all(by.css('.selector'));
```

### 子元素
要查找子元素，只需将 element 和 element.all 函数链接在一起，如下所示：

使用单个定位器查找：

一个元素:

 * `element(by.css('some-css'));`
元素列表：

 * `element.all(by.css('some-css'));`
使用链接的定位器查找：

子元素：

 * `element(by.css('some-css')).element(by.tagName('tag-within-css'));`
查找子元素列表：

 * `element(by.css('some-css')).all(by.tagName('tag-within-css'));`
也可以使用 get / first / last 进行链接，如下所示：
```javascript
element.all(by.css('some-css')).first().element(by.tagName('tag-within-css'));
element.all(by.css('some-css')).get(index).element(by.tagName('tag-within-css'));
element.all(by.css('some-css')).first().all(by.tagName('tag-within-css'));
```