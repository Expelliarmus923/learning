### UI自动化测试浅谈

最近工作做了一些持续集成和自动化测试相关的工作，也有了一些心得和体会，之前写过一篇关于集成GitLab-CI的攻略，期间提到了将测试功能集成到CI的过程之中。这篇文章也可以算得上是上一期的后续了。鉴于大家或多或少都有接触过单元测试，而对于UI自动化测试可能会比较陌生。所以这期从WEB前端的角度先给大家讲一讲前端的UI自动化测试。

### 什么是UI自动化测试

UI自动化测试是指对用户界面进行的程序自动化测试，长久以来，UI自动化测试一直存在着许多难点。UI自动化测试受限于测试环境环境，用户界面通常是基于一个有一个事件的，所以测试用例在编写的时候也会变得非常复杂。此外UI自动化测试的覆盖率计算方法也同样很困难。

### 为什么要进行UI自动化测试

虽然很多人认为UI自动化测试坑很多，但是对于需要确保长期稳定的基础库而言，保证功能的正常使用是非常重要的。我司过去也曾发生过多次由于基础库改动之后缺少测试，造成大面积依赖该基础库的业务代码出现故障的情况。

### 主要的UI自动化测试可选工具与测试框架

如果你之前有了解过一些测试工具和测试框架，那么你可能会对一系列的名词感到混乱，下图虽然不是很严谨，但是也能够描述清楚这些框架与工具的关系了。

![image](http://h0.hucdn.com/open/201726/969ce176fa73f0b0_1678x808.png)

底层的属于驱动和工具，中间部位是对这些工具和驱动的简单封装，顶层的属于基于node的UI自动化测试框架。

headless Chrome:
是Chrome浏览器在59版本推出的新功能，支持无界面的情况下打开Chrome浏览器，并解析HTML，CSS，JS相关内容并且支持linux，基于自己提供的通信协议，配合chrome-remote-interface模块可以实现对浏览器的自动化操作。lighthouse是Chrome开源的一个偏向于性能测试的工具，在性能测试的层面上功能非常强大。但是目前缺少断言库，并且没有很好的测试框架支持，模拟用户操作比较麻烦。

phantomjs:
一个机遇qtwebkit的一个无界面浏览器，可以在各种环境中使用。配合phantomCSS和CasperJS工具的支持，页面著名的Selenium测试框架也对其进行了封装。在Chrome发布带有无界面浏览器功能的版本之后开发者宣布放弃维护。不过鉴于基于Chrome浏览器的测试框架还需要一定的发展时间。所以phantomjs本身还是值得使用的。基于此封装的nightmare测试框架稳定性不高，容易出现卡死的情况。

electron:
electron这个工具本来是基于Chromium开发出来提供给前端做WEB桌面应用的，功能强大，于是nightmare测试框架在2.0版本之后遍放弃基于phantomjs来封装，转投向electron。我司有一个由运营同学拖拽生成页面的工具在页面发布上线前会经过改工具的自动化检测。该缺点是在linux环境下面仅支持安装了界面的ubuntu。因为与Chrome同样是基于Chromium，所以高版本electron对于linux也能够支持无界面。（建议使用ubuntu而非centos）

各种driver:
这些driver基于浏览器提供的各种协议，通过协议可以和浏览器进行同行，从而操作浏览器。nightwatch是一个基于Selenium和webdriver封装的测试框架。能够模拟用户操作，同时也封装了断言库，支持多浏览器测试，虽然在linux仅支持selenium+phantomjs版本，不过依然是一个不错的适合在linux环境下做UI自动化测试的框架。

### 如何使用

首先，我们需要在node项目中安装依赖。

```
npm install --save-dev selenium-server phantomjs-prebuild chromedriver nightwatch nightwatch-helpers
```

然后因为nightwatch真基于Selenium进行的封装，而Selenium在nightwatch封装里使用了java这门语言来操作自身。所以我们还需要安装java环境。

接下来我们需要新建一个nightwatch的配置文件，可以起名为nightwatch.config.js具体配置内容可以参考文档[http://nightwatchjs.org/guide#settings-file](http://nightwatchjs.org/guide#settings-file)

之后我们需要准备测试页面，HTML，JS，CSS等，然后启动服务器使其可以被测试工具访问。可以使用http-server或者当然你使用python -m SimpleHTTPServer 8080 这样的花式启动服务器方式也是OK的，原理一样。

之后我们需要书写测试用例文件，测试用例可以是多个文件也可以是单个文件，在测试中你可以控制浏览器进行跳转操作，滚动操作，模拟用户点击等等。具体可以参考文档[http://nightwatchjs.org/api](http://nightwatchjs.org/api)

然后我们运行nightwatch的命令行工具启动测试即可。上述操作可以写成一个简单的server.js。

server.js
```
"use strict"

const path = require('path')
const spawn = require('cross-spawn')
const httpServer = require('http-server')

const server = httpServer.createServer({
  root: path.resolve(__dirname, '../../')
})
server.listen(8080)
let args = ['--config', 'test/test-e2e/nightwatch.config.js', '--env', 'phantomjs', '--test', 'test/test-e2e/test.spec.js']
// nightwatch --config nightwatch.config.js --env phantomjs --test test/test.spec.js

const runner = spawn('./node_modules/.bin/nightwatch', args, {
  stdio: 'inherit'
})

runner.on('exit', function (code) {
  server.close()
  process.exit(code)
})

runner.on('error', function (err) {
  server.close()
  throw err
})
```

test.spec.js
```js
module.exports = {
	'test-name': function (browser) {
		browser.url('http://localhost:8080/example/index.html', () => {
			// 做操作浏览器的操作，然后下断言
		})
	}
}
```


不过还是推荐写一个server.js，这样方便一些，在测试的时候我们只需要运行node server.js就可以开始我们的自动化测试了。

到此，大功告成了，如果你觉得集成过程很轻松，那么很可能你是用windows或者Mac OS在本地运行的。如果你要部署在linux环境中，你还需要踩几个坑。首先linux环境中目前只能使用phantomjs来进行测试。centOS用户如果启动失败需要安装一些基础库（用报错信息百度即可）。linux网速比较慢的话，一些大型的工具建议提前安装好。以免运行脚本时间过长。

### 结束

最近两周一直在研究UI自动化测试相关的技术，目前已经有了一些进展，并且也给公司的部分基础库集成了UI自动化测试与单元测试。我将这段时间学习到的技术与心得都写了下来，希望对大家能够有所帮助。