# 前端开发模式的迭代

前端开发给人的印象一直是变化太快，不断出现新的框架、库、开发模式，这些开发模式有什么不同，开发模式为什么会不断迭代，本文将分享几种常见的前端开发模式，讲解前端开发模式的演变过程。

## 传统开发模式

前端是 Web 应用的组成部分，前端开发模式的设计与 Web 应用的系统架构紧密相关，一个典型的 Web 应用系统包括下面几个部分：

```        
                 HTML 文档 / JSON 数据
         +---------------------------------+
         |          response               |
         v                      +--------------------+
 +---------------+              | Application Server |
 |               | -----------> +--------------------+
 |     浏览器     |   request
 |               | -----------> +--------------------+
 +---------------+              |   静态资源 Server   |
         ^                      +--------------------+
         |          response               |
         +---------------------------------+
                JS、CSS、图片等静态资源
```
                            
Application Server 负责提供动态内容，浏览器发起请求后，由 Application Server 返回 HTML 文档或 JSON 数据，浏览器解析到 HTML 内的 `script`、`link`、`img` 等资源标签时，会发起资源文件的请求，静态资源 `URL` 通常会加上专门的域名解析到静态资源 Server，对于小型 Web 系统，Application Server 和静态资源 Server 会由同一个 Web Server 来承载。

动态 HTML 衍生出了两种传统的前端开发模式：

* HTML 混合在 Server 端程序代码中
* 通过 Server 端模板引擎生成 HTML

### HTML 混合在 Server 端程序代码中

这种模式下的协作方式通常是由前端工程师开发好静态页面，再交给后端工程师『套页面』，最后产出一个包含 Server 端程序代码和 HTML 的代码文件，浏览器请求时由 Server 端程序执行代码文件，获取数据、拼接 HTML 片段生成完整的 HTML 文档。

这种协作方式弊端是：

* 效率低，前后端代码混合在一块，UI 上的改动经常需要前后端工程师共同完成。
* 容易出错，后端工程师不写 HTML，却需要把完整的 HTML 折成很多 HTML 片段，再插入到程序逻辑中，拼完后执行出来经常发现与静态页面不一样，然后找前端工程师定位问题，看看哪里拼错了。

### 通过 Server 端模板引擎生成 HTML

模板引擎的出现为前后端提供了更好的协作方式，它是一个运行在 Server 端应用程序中的组件，能清晰的将前端代码与 Server 端程序代码分离。

#### 模板引擎的使用方式

以基于 PHP 的 `Smarty` 模板引擎为例，安装 `Smarty` 后，编辑 PHP 文件：

```php
<?php

// put full path to Smarty.class.php
require('/usr/local/lib/php/Smarty/Smarty.class.php');
$smarty = new Smarty();

$smarty->setTemplateDir('/web/www.example.com/smarty/templates');
$smarty->setPluginsDir('/web/www.example.com/smarty/plugins');
$smarty->setCompileDir('/web/www.example.com/smarty/templates_c');
$smarty->setCacheDir('/web/www.example.com/smarty/cache');
$smarty->setConfigDir('/web/www.example.com/smarty/configs');

$smarty->assign('name', 'Ned');
$smarty->display('index.tpl');

?>
```

创建模板：

```html
<html>
  <head>
    <title>Smarty</title>
  </head>
  <body>
    Hello, {$name}!
  </body>
</html>
```

调用 `assign` 方法向模板中传递`变量/对象`，调用 `display` 方法来显示模板，生成 HTML。

`assign` 和 `display` 确定了前后端的衔接方式：约定一份模板变量、一个模板文件路径。

#### 基于 Server 端模板引擎的组件化方案

模板引擎在 Web 应用架构里分离了 UI 界面和业务逻辑，接下来需要考虑的就是前端开发内的分治手段，怎么把一个系统拆分成更小的单元，便于开发和维护。

前端开发最为有效的分治手段就是**组件化开发**，把一个大的系统自顶向下拆分下若干组件，最后按某种方式组装起来，成为一个整体，完成系统所要求的功能。

这里就涉及两个东西：拆分和组装，拆分就衍变为开发阶段的**组件化规范**，组装就衍变为运行阶段的**组件化框架**。

前端程序包含三种语言：`HTML`、`CSS`、`JavaScript`，前端组件可以由这三种开发资源组成，但前端语言和浏览器并未提供一个组合三种开发资源的组件化方案，所以前端领域内的组件化方案层出不穷，基于 Server 端模板引擎的组件化方案，比较有代表性的就属百度的 `FIS-PLUS`。

`FIS-PLUS` 提供给开发者的内容包括三部分：

* 目录规范：开发阶段怎么拆分组件。
* 前后端框架：运行阶段怎么组装组件。
* 构建工具：连接开发规范与部署规范。

##### 目录规范

`FIS-PLUS` 推荐的目录结构如下：

```
site
├── common                      # 通用子系统，为其他业务模块提供资源复用的通用模块
│   ├── plugin                  # Smarty 插件目录
│   ├── config                  # Smarty 配置文件
│   ├── page                    # 通用页面模板，用于业务子系统的继承模板
│   │   └── layout.tpl
│   ├── widget                  # 通用组件
│   │   ├── header
│   │   │   ├── header.js
│   │   │   ├── header.less
│   │   │   └── header.tpl
│   │   └── nav
│   │       ├── nav.less
│   │       └── nav.tpl
│   ├── static                  # 非组件静态资源
│   └── fis-conf.js             # FIS 配置
└── module1                     # 业务子系统，根据具体业务进行划分的子系统模块
    ├── page
    │   └── index.tpl
    ├── widget
    │   └── banner
    │       ├── banner.js
    │       ├── banner.less
    │       └── banner.tpl
    ├── static
    │   └── index
    │       ├── index.js
    │       └── index.less
    ├── test                    # 测试数据
    └── fis-conf.js
```

目录规范实则是对资源的分类概念，上面的目录规范将整个站点的资源划分为：模块（module）、页面（page）、组件（widget）、静态资源（static）、插件（plugin）、测试数据（test），这里有两个关键点：

* 一个大的站点可以拆分成多个子系统。
* 每个组件对应一个目录，组件内的所有资源在目录下就近维护。

**每个组件对应一个目录**为前端开发提供了非常好的分治策略，整体系统可以以组件为单位由多人协作开发，部署前构建工具可以分析引用关系进行打包，有依赖由打包整个组件目录，没依赖则不打包。

**header.tpl**

```html
<div id="header">
  <h1>Title</h1>
</div>
```

**index.tpl**

```php
{%widget name="common:widget/header/header.tpl"%}
```

##### 前后端框架

按目录规范开发好的各种资源要在运行阶段整体跑起来，就需要依赖前后端框架来进行组装。

**Smarty 插件**

`Smarty` 在 2.0 中引入了插件架构，可以用于在模板中使用自定义功能，如前面的 Smarty 示例代码，其中有一行：

```php
$smarty->setPluginsDir('/web/www.example.com/smarty/plugins');
```

用于指定插件目录，运行时目录下的所有插件都会被加载，模板中可以直接使用，如：

```php
<?php
/*
 * Smarty plugin
 * -------------------------------------------------------------
 * File:     function.eightball.php
 * Type:     function
 * Name:     eightball
 * Purpose:  输出一个随机的答案
 * -------------------------------------------------------------
 */
function smarty_function_eightball($params, Smarty_Internal_Template $template)
{
    $answers = array('Yes',
                     'No',
                     'No way',
                     'Outlook not so good',
                     'Ask again soon',
                     'Maybe in your reality');

    $result = array_rand($answers);
    return $answers[$result];
}
?>
```

以上面的方式即可以定义一个模板函数插件，在模板中调用就能输出一个随机答案：

```html
<p>{%eightball%}<p>
```

```
Outlook not so good
```

组件规范中引入 `widget` 的语法 `{%widget%}` 实则是 `FIS-PLUS` 提供的一个 `Smarty` 插件，除了 `widget` 插件， `FIS-PLUS` 还提供了一系列与模板相关的插件，当 `display` 某个页面模板时，正是这些一系列插件把所有模板组件组装起来，生成一个完整的 HTML 文档。

**modJS**

除了 `Smarty 模板`，每个组件还可以有各自的 JS，JS 模块怎么引用，执行时怎么保证依赖的 JS 模块已经加载，这就需要前端模块化框架的支持。

`modJS` 是一个精简版的AMD/CMD规范，实现了模块的定义 `define (id, factory)` 、模块的同步加载 `require (id)`、模块的异步加载 `require.async (names, onload, onerror)` 等方法。

具体实现可阅读源码：https://github.com/fex-team/mod/blob/master/mod.js

##### 构建工具

有了开发阶段的组件化规范、运行阶段的组件化框架，接下来的一个问题是怎么把它们连接起来，怎么把开发资源转换为运行时的结构，有很多运行时的需求要满足，包括：

* 开发目录结构转为部署目录结构
* JS、CSS 要按依赖关系来打包
* 支持 JS、CSS、图片的压缩
* 非原生语言如：less，要转化为原生语言 CSS
* 考虑 CDN、浏览器的缓存，发布时要为文件加版本号
* ...

这就需要构建工具的支持，在开发与部署上线之间增加一个编译环节，来实现开发资源到部署资源的转换。

`FIS-PLUS` 扩展自 `FIS`，继承了 FIS 的这种构建能力，提供给开发者的内容有两块：

* 命令行工具，通过 `npm` 安装后即可在开发目录下执行相关命令，实现资源的编译、本地开发调试等功能。
* 配置文件，在开发目录下添加 `fis-conf.js` 文件即可对编译过程做各种定制化的配置。

FIS 的核心主要有两部分：编译流程和插件系统。

**编译流程**

![FIS 编译流程](./assets/img/fis-compile-flow.png)

FIS 设定了一个编译流程，分为单文件编译阶段和打包阶段，两个阶段又分别设定了多个环节，FIS 构建时会把开发目录下配置的文件都收集起来，每个文件都会在这个流程中走一遍，通过各个环节配置的插件来对文件进行处理，最后输出为部署资源。

**插件系统**

在上面的编译流程中，FIS 在 `standard` 环节内置了对三种语言能力的扩展，其他环节基本都是通过各种插件对文件进行处理，并且提供了统一的插件扩展方式：

```javascript
/**
 * Compile 阶段插件接口
 * @param  {string} content     文件内容
 * @param  {File}   file        fis 的 File 对象 [fis3/lib/file.js]
 * @param  {object} settings    插件配置属性
 * @return {string}             处理后的文件内容
 */
module.exports = function (content, file, settings) {
    return content;
};
```

插件接口就是一个 `function`，接收文件内容，进行处理后返回新的内容。

**静态资源表**

FIS 完成构建过程后，除了通过各种插件对文件进行处理，核心的产出就是 `静态资源表`，静态资源表是一个 JSON 文件，记录了文件的依赖、打包、URL 等信息。

```json
{
    "res" : {
        "a.js" : {
            "uri" : "/a.js",
            "type" : "js",
            "pkg" : "p0"
        },
        "b.js" : {
            "uri" : "/b.js",
            "type" : "js",
            "pkg" : "p0"
        },
        "c.js" : {
            "uri" : "/c.js",
            "type" : "js",
            "pkg" : "p0"
        }
    },
    "pkg" : {
        "p0" : {
            "uri" : "/aio.js",
            "type" : "js",
            "has" : ["a.js", "b.js", "c.js"]
        }
    }
}
```

前后端框架拿到这个表结构后就能实现各种资源加载、性能优化策略。

有了上面的开发规范、前后端框架、构建工具，就能搭建起一套基于 `Server` 端模板的前端开发模式。

## 前后端分离

Application Server 提供的动态内容除了 HTML，还可以是 JSON 数据，这又衍生出另一种开发模式：`前后端分离`。

前后端分离的模式带来了巨大的好处：

* 后端工程师只负责提供 API 接口，可以专注于 API 的实现。
* 一套 API 可以提供给浏览器、IOS、Android 等多端使用。
* 前后端可以完全分开部署，后端不用再承担模板渲染、页面路由配置等 UI 层的工作。
* 前端可以把控用户交互、页面路由、内容渲染的整个过程，在开发效率、功能实现方案、用户体验上会有更大的探索空间。

前后端分离有两种模式：

* 基于 Node 的前后端分离
* 单页 Web 应用

### 基于 Node 的前后端分离

基于 Node 的前后端分离依然会有 Server 端渲染页面的逻辑，只是这个 Server 端是由前端工程师通过 Node 搭建的，整体流程如下：

```
              response                    response
       +-------------------+       +-------------------+
       v                   |       v                   |
+---------+   request   +-------------+   request   +------------+
|  浏览器  | ----------> | Node Server | ----------> | API Server |
+---------+             +-------------+             +------------+
```

浏览器发起的请求由 Node Server 来处理，Node Server 接收到请求后访问相应的 API Server 获取数据，再生成 HTML 文档返回给浏览器。

开发 Server 端应用就需要有一个 Server 端的应用框架，基于 Node 的应用框架有很多，包括：`Express`、阿里的 `Egg`、百度的 `Yog2` 等等，下面以 `Yog2` 为例。

#### 目录规范

`Yog2` 是一个基于 `Express` 和 `FIS` 开发的专注于 UI 中间层的 Node 应用框架，一个完整的 Yog2 应用包含两部分代码：

**基础运行框架**，由 Yog2 的脚手架创建，负责中间件的初始化和建立基础环境。

```
yog
├── app                   # server 代码目录
├── conf                  # 配置目录
│   ├── plugins           # 插件配置  
│   └── ral               # 后端服务配置
├── plugins               # 插件目录
├── static                # 静态资源目录
├── views                 # 后端模板目录
└── app.js                # project 启动入口
```

**业务代码**

```
home
├── client                # 前端代码
│   ├── page              # 页面模板
│   ├── widget            # 组件
│   └── static            # 非组件化静态资源
├── server                # 后端代码
│   ├── lib               # 通用库
│   ├── action            # 控制器目录
│   ├── model             # 数据模型目录
│   └── router.js         # 路由配置
└── fis-conf.js           # FIS 编译配置
```

开发业务时通常只需要根据业务代码的目录规范来开发各个功能，`client` 目录是前端代码，目录结构与前面说的 `FIS-PLUS` 目录规范相同，对前端资源的分类，`server` 目录是后端代码。

#### 后端业务开发

开发一个后端功能主要包括三个部分：

**路由**

`Yog2` 框架提供了自动路由功能，URL 与 `action` 之间会有一个默认的映射关系：

```
http://www.example.com/home/index => app/home/action/index.js
http://www.example.com/home/doc/detail => app/home/action/doc/detail.js
```

自动路由不能满足的需求可以在 `server/router.js` 中自定义：

```javascript
module.exports = function(router){
  router.route('/book')
    // PUT /cdcd/book/id
    .put(router.action('book').put)
    // GET /cdcd/book
    .get(router.action('book'));
};
```

传递给 `action` 方法的参数是 action 目录下对应的文件路径。

**控制器**

控制器就是路由指向的 `action` 文件，是具体业务的载体，用于解析请求参数、访问数据、返回结果。

```javascript
// /server/action/user.js
var userModel = require('../models/userModel.js');

module.exports.get = function (req, res, next) {
  var id = parseInt(req.body.id, 10);
  if (isNaN(id)) {
    throw new Error('invalid id');
  }
  userModel.get(id)
    .then(function (user) {
      res.render('user/page/index.tpl', {
        user: user
      });
    })
    .catch(next);
}
```

**数据模型**

数据模型负责提供数据给控制器，又可以分为服务层和数据层，服务层可以专注于业务逻辑封装和数据层的调用，数据层则专注于与后端服务层的交互。

```javascript
// /server/models/userModel.js

var yog = require('yog2-kernel');

module.export.get = function (id) {
  return yog.ralP('BACKEND', {
    path: '/api/user',
    method: 'GET',
    data: {
      id: id
    }
  });
};
```

#### Express

`Yog2` 是基于 `Express` 开发的，在 `Express` 的基础上提供了应用开发规范和插件系统，核心功能还是由 `Express` 提供。

**中间件**

`Express` 是一个由路由和中间件构成的 Web 应用框架，在接收到请求后调用中间件来处理，中间件的功能包括：

* 执行任何代码。
* 修改请求和响应对象。
* 结束 `请求-响应` 循环。
* 调用下一个中间件函数。

一个中间件函数如：

```javascript
var express = require('express')
var app = express()

// 创建中间件函数
var myLogger = function (req, res, next) {
  console.log('LOGGED')
  next()
}

// 安装中间件函数
app.use(myLogger)

app.get('/', function (req, res) {
  res.send('Hello World!')
})
    
app.listen(3000)
```

中间件函数接收三个参数，`req` 请求对象、`res` 响应对象、`next` 调用下一个中间件，可以在请求对象上添加一些数据再执行 `next()` 调用下一个中间件函数，也可以直接通过响应对象来结束请求返回结果。

前面说的控制器 `action` 文件实则就是一个中间件函数。

#### 构建工具

同样，在基于 Node 的前后端开发模式下，也需要把开发资源转换为部署资源，`Yog2` 除了是一个 Node 应用框架，也是一个基于 `FIS` 的构建工具，前面提到的构建需求在 `Yog2` 下也能实现。

有了上面的 Node 应用框架、前后端开发规范、构建工具，就能搭建起一套基于 Node 的前后端分离开发模式。

### 单页 Web 应用

单页 Web 应用通常没有 Server 端渲染模板的逻辑，前端产出的都是静态资源，整体流程如下：

```
             response                              response
       +----------------------+            +----------------------+
       v                      |            v                      |
+---------+  request   +-----------------------+   request   +------------+
|  浏览器  | ---------> | index.html (index.js) | ----------> | API Server |
+---------+            +-----------------------+             +------------+
```

浏览器访问应用的各个 URL 时 Web Server 都会 `rewrite` 到一个静态 HTML：

```config
location / {
    root /home/www/app/;
    index index.html;
    if (!-f $request_filename) {
        rewrite ^(.*)$ /index.html break;
    }
}
```

这个 HTML 通常没有内容，只是一个加载 JS 的页面：

```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1, minimum-scale=1, user-scalable=no" />
  <title></title>
  <link rel="shortcut icon" href="/favicon.ico">
  <link href="/index.95051656.css" rel="stylesheet">
</head>
<body>
  <div id="root"></div>
  <script type="text/javascript" src="/index.fcd62d1c.js"></script></body>
</html>
```

单页 Web 应用与普通网站的重要区别是：普通网站的入口是 HTML，而单页 Web 应用的入口实则是 JS，由 JS 解析路由、从 API Server 取数据、渲染内容。

具体到应用的开发，依然需要有开发规范与框架的支撑，主流的技术栈包括：`Angular`、`React`、`Vue`，下面以 `React` 技术栈为例。

`React` 是一个开发 UI 界面的基础库，提供了一种全新的开发 UI 界面的模式，核心思想是**数据驱动 UI 变化**，在这种模式下不用关注 DOM 操作、不用关心数据变化后怎么修改 DOM，开发时只需分解界面的 UI 组件，确定各组件内的状态变化，最后以标签化的方式组装各个 UI 组件。

#### 组件化

`React` 对组件有了一个非常清晰的定义，一个组件包括：

* `JSX`，用来描述 UI 结构。
* `Props`，传递给组件的不变的数据。
* `State`，随着时间推移会变化的数据。
* `Events`，为组件添加各种交互事件。
* `Lifecycle`，一个组件从诞生到销毁的各个阶段。
* `Refs and the DOM`，对特殊场景依然提供了 DOM 操作接口。

有了这些定义，我们就能以一种非常标准化的方式来开发一个 UI 组件。

```javascript
class Clock extends React.Component {
  constructor(props) {
    super(props);
    this.state = {date: new Date()};
  }

  componentDidMount() {
    this.timerID = setInterval(
      () => this.tick(),
      1000
    );
  }

  componentWillUnmount() {
    clearInterval(this.timerID);
  }

  tick() {
    this.setState({
      date: new Date()
    });
  }

  render() {
    return (
      <div>
        <h1>Hello, world!</h1>
        <h2>It is {this.state.date.toLocaleTimeString()}.</h2>
      </div>
    );
  }
}
```

#### 数据层

组件化为 UI 界面的开发提供一种非常好的分治手段，但对于大型 Web 应用系统，背后往往还有复杂的业务逻辑，在这种场景下只有 UI 层的分治是远远不够的，还需要分层：建立数据层，为此还需要引入数据流的管理方案，比如：`dva`。

`dva` 是一个基于 `redux` 和 `redux-saga` 的数据流方案，同时内置了 `react-router` 和 `fetch`，基于 `dva` 开发一个应用主要包括下面几个步骤：

**1. 定义 Services**

`Services` 是对 API Server 交互的封装：

```javascript
// /src/services/api.js
import request from '../utils/request';

/**
 * 查询用户信息
 */
export async function queryUserInfo() {
  return request('/api/userinfo');
}

/**
 * 查询商品列表
 */
export async function queryProducts() {
  return request('/api/products');
}
```

**2. 设计 Model**

按业务相关性来划分 `model`，针对单个 `model`，从数据维度抽离数据本身以及相关操作的方法，从业务维度将强关联数据的组件状态抽象成 model 方法。

```javascript
// /src/models/users.js
export default {
  namespace: 'users',
  state: {
    list: [],
	  total: null, 
    loading: false,       // 控制加载状态
    current: null,        // 当前分页信息
    currentItem: {},      // 当前操作的用户对象
    modalVisible: false,  // 弹出窗的显示状态
    modalType: 'create',  // 弹出窗的类型（添加用户，编辑用户）
  },
	effects: {
		*query(){},
		*create(){},
		*'delete'(){},
		*update(){},
	},
	reducers: {
		showLoading(){},      // 控制加载状态的 reducer
		showModal(){},        // 控制 Modal 显示状态的 reducer
		hideModal(){},
		querySuccess(){},
		createSuccess(){},
		deleteSuccess(){},
		updateSuccess(){},
	}
}
```

`model` 的内容包含 4 部分：

* `namespace`，model 的命名空间，用命名空间来区分不同业务。
* `state`，model 的数据结构，以及定义默认的 state 值。
* `reducers`，负责修改 model 数据。
* `effects`，负责处理 `副作用`，如异步操作。

**3. 设置路由**

```javascript
// /src/router.js
import React, { PropTypes } from 'react';
import { Router, Route } from 'dva/router';
import Users from './routes/Users';

export default function({ history }) {
  return (
    <Router history={history}>
      <Route path="/users" component={Users} />
    </Router>
  );
};
```

路由主要包括几个元素：

* 匹配规则，在 `Route` 组件上用 `path` 属性定义。
* 不同路径对应的组件，在 `Route` 组件上用 `component` 属性定义。
* `Router`，路由器，将路径路由到对应的组件上。

**4. 添加路由组件**

```javascript
// .src/routes/Users.jsx
import React, { PropTypes } from 'react';

function Users() {
  return (
    <div>User Router Component</div>
  );
}

Users.propTypes = {
};

export default Users;
```

路由组件作为各个访问路径的入口，用于组装项目组件。

**5. 设计项目组件**

基于 `React` 的组件化规范，`Container Components` 和 `Presentational Components` 相分离的开发思想。

```javascript
import React, { Component, PropTypes } from 'react';

// dva 的 connect 方法可以将组件和数据关联在一起
import { connect } from 'dva';

// 组件本身
const MyComponent = (props)=>{};
MyComponent.propTypes = {};

// 监听属性，建立组件和数据的映射关系
function mapStateToProps(state) {
  return {...state.data};
}

// 关联 model
export default connect(mapStateToProps)(MyComponent);
```

经过这几个步骤基本就形成了一个完整的 `dva` 项目目录结构：

```
.
├── /src/            # 项目源码目录
│ ├── /components/   # 项目组件
│ ├── /routes/       # 路由组件（页面维度）
│ ├── /models/       # 数据模型
│ ├── /services/     # 数据接口
│ ├── /utils/        # 工具函数
│ ├── route.js       # 路由配置
│ ├── index.js       # 入口文件
│ ├── index.less     
│ └── index.html     
├── package.json     # 项目信息
└── proxy.config.js  # 数据mock配置
```

当然这个目录结构是可以先行通过 `dva-cli` 脚手架来创建的。

#### 构建工具

单页 Web 应用同样需要构建工具来编译，`dva` 搭配的构建工具是 `roadhog`，基于 `Webpack` 的构建工具，内置了很多 `Webpack` 配置，提供了更简便的配置方式。

```javascript
// .webpackrc.js
const path = require('path');

export default {
  entry: 'src/index.js',
  publicPath: '/',  html: {
    template: 'src/index.ejs',
    favicon: 'src/assets/favicon.ico'
  },
  alias: {
    components: path.resolve(__dirname, 'src/components')
  },
  hash: true,
  proxy: {
    '/api/': 'http://localhost:8000/'
  }
};
```

`Webpack` 与 `FIS` 的理念有很大的差别，`Webpack` 的编译从 `entry` 开始，分析所有依赖的文件，通过 `loader` 和 `plugins` 处理，生成最终的打包文件。

`FIS` 没有 `entry` 的概念，会收集所有资源，分析每一个资源的依赖关系，最终生成静态资源表。

有了上面 UI 界面的组件化方案、数据流的管理方案、构建工具，就能搭建起一套单页 Web 应用的开发模式。

## 开发模式为什么要迭代

至此已经介绍了多种不同的前端开发模式，工程师掌握一种开发模式后就能熟练的开发业务，那么开发模式为什么还需要不断迭代、为什么还要去学习新的开发模式？

总体来说有三层目标。

### 高效的工程师

高效的工程师通常具备两方面能力：

* **方法论**，包括各种设计模式、开发模式、业务开发思路等等，方法论的掌握程度会影响对系统复杂度的把控、以及开发的效率与质量。
* **执行力**，了解方法论还不够，方法论要充分实践才能对应用过程有足够的画面感，通过实践不断提炼、沉淀方法论，方法论再反过来帮助实践过程，如此往复，执行力才会不断提升。

### 工程化的团队

工程化团队的特点是：可以多人并行开发、人员数量翻倍产能就能翻倍。

要做到这一点非常难，提升产能要先看产出是什么，研发团队的产出包括两部分：

* **基础技术**，包括各种框架、库、工具、以及前面讲的开发模式等等，我们谈各种技术架构时谈的主要都是基础技术的部分，基础技术能让团队成员在一个不错的起点上开始工作。
* **业务代码**，开发业务代码才是研发团队的工作日常，同一个功能不同工程师的开发效率、产出质量都不一样，怎么提升开发业务代码的效率和质量，开发模式的迭代是一个手段，背后还有更多对工程师能力的要求。

### 卓越的团队

研发团队的理想状态是产能可以持续提升，从而实现需求翻倍而人员数量不用翻倍。

要做到这一点更难，已经不是单工种团队的迭代可以做到的，需要与周边团队的紧密合作，目前想到的方向有两个：

* **减少消耗**，工种的划分、项目的划分、团队的划分，分工协作的同时也很容易增加额外的成本，A 项目 里做过的某些事情在 B 项目可能会再做一遍、A 团队造过的轮子在 B 团队可能会再造一遍，这点在大厂尤为明显，随便统计一下就能数出十几个甚至几十个类似的系统、模块，工程师写的类似的业务代码更是数不胜数。
* **提炼业务共性**，常见的产品研发流程通常包括两个阶段：产品化阶段、技术化阶段，研发工程师习惯于等产品化阶段结束提供交付物后再开始开发，产品经理也习惯默默的把产品设计做完后再告诉研发工程师可以开发了，在这个过程中，不同项目之间有哪些共性、不同产品之间有哪些共性、产品与技术的映射关系怎么优化，基本缺少梳理、抽象，把这些业务共性提炼出来，对研发效率会有很大的提升。

> 皮成，2018.08.06