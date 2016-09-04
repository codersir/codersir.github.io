# 前端工程师
## 个人信息

- 姓名： 陈旭鸿
- 年龄： 24
- 性别： 男
- 教育背景：2010-2014 华中科技大学光信息科学与技术专业

## 工作经历

- *2014.03*-*2014.05* 百度移动搜索部门实习
- *2014.07-至今* 武汉闪达科技有限公司（原搜狐武汉研发中心）前端工程师

## 项目经历

- **[SendCloud](http://sendcloud.sohu.com)**，[http://sendcloud.sohu.com](http://sendcloud.sohu.com)，面向开发者的邮件发送平台
	负责官网首页和用户后台的开发。用户后台采用**前后端分离**的架构，后端提供 JSON 接口，前端是使用 Angular 开发的单页 webapp。路由控制使用的 ui-router，UI 框架使用的 angular-bootstrap。整个项目可以拆分为三个子项目，包括邮件、短信和总子账户，后期重构拆分为三个不同的入口，把公共部分提取了出来，公共部分包括账户模块和一些可以复用的组件等，**提高代码复用率，减少打包文件体积**。上线前的代码合并压缩添加版本等任务由 Gulp 来完成。
- **[爱发信](http://ifaxin.com)**，[http://ifaxin.com](http://ifaxin.com)，面向普通用户的邮件发送平台。
  负责官网首页和用户后台的开发，以及可视化邮件编辑器的开发。用户后台页仍然是一个 Angular 的单页 webapp，和 SendCloud 用户后台相比，亮点在增加了**角色权限控制和更优秀的性能**。权限控制通过监听 ui-router 的 $stateChangeStart 事件进行拦截来实现，性能优化参考 [batarange](http://byevil.com/2016/05/10/AngularJS-app-performance-optimization-guide/) 工具和社区最佳实践，做到清除所有 angular-hint 的报警和最大化减少 watchers，大大提高了页面的性能。可视化编辑器部分使用 jQuery 作为基础库，采用模块化开发，使用 requirejs 进行模块化加载。
- **[Notice](http://notice.sendcloud.net)**，[http://notice.sendcloud.net](http://notice.sendcloud.net)，覆盖邮件、短信和微信三个平台的通知报警应用。
  前端同样是使用 Angular 开发的单页 webapp。这个应用比较小，很快的完成了，没有什么技术难点，亮点在**完美的实现了设计稿，做到全站自适应**。
- **[mis](http://mis.sendcloud.net)**，[http://mis.sendcloud.net](http://mis.sendcloud.net)，SendCloud 代理商使用的用户后台。
  这个项目是用 React 开发的第一个项目，采用阿里开源的 ant-design 来进行快速开发，整个项目耗时不到 3 周。通过这个项目掌握了** React 组件**的开发，对** React 的执行机制**有了进一步的了解，另外深入了解了** React 的状态管理**方案：Redux ，对 React 状态到视图的单向数据流理念很喜欢。
- **[iDevJS](https://github.com/idevjs)**，[https://github.com/idevjs](https://github.com/idevjs)，前端技术社区，即将上线。
  这个项目前后端都由我一人开发。同样是采用前后端分离，后端 API 服务器是基于 koa1 进行开发的，数据库使用的 Mongodb。前端暂时只有** [Angular2](https://github.com/idevjs/idevjs-angular2)** 的版本，后期会开发 [React](https://github.com/idevjs/idevjs-react)、[Vue](https://github.com/idevjs/idevjs-vue) 等版本。做这个社区的初心是提供一个前端试验场，在这里可以使用最近的框架和技术，不会因为日常工作中用不到一些技术就放弃去学习体验它。同时，前后端全部开源，每个人都可以参与到其中，社区项目本身也是一个很好的学习资源。

## HTML & CSS 基础

- 精通 HTML，能够编写语义化的 HTML，了解 SEO，熟悉 HTML5
- 掌握 CSS，熟悉常见 CSS 布局，能够精准还原设计稿。熟悉 CSS3，掌握自适应布局，了解 CSS 预处理器（平时使用 stylus）。

## JavaScript 基本功

- 掌握 ES5，掌握 DOM 编程，对 JS 中的基本概念 原型、闭包、继承 等有深入的了解，能够编写风格一致的可维护代码。
- 熟悉 ES6，了解 Object、Array、Function 等的扩展，掌握 generator、promise、class 等概念，了解 ES7 的 async/await 、decorator 等，对异步流程控制有深入了解。
- 熟悉模块化开发，使用 requirejs 进行模块化加载。

## JS Library（Framework）

- 掌握 Angular1，对 Angular1 的 directive、service 等有深入的了解，钻研并实践过 Angular1 webapp 的性能优化，负责开发过公司的三个 Angular1 webapp 项目。
- 熟悉 Angular2，官网文档和各种博客资料看下来，基本概念已经熟悉，其中深入了解了 Zone.js 和 Observer 以及 Angular2 的依赖注入实现。[iDevJS](https://github.com/idevjs) 前端就是用 Angular2 开发的，[项目地址](https://github.com/idevjs/idevjs-angular2)：[https://github.com/idevjs/idevjs-angular2](https://github.com/idevjs/idevjs-angular2)。
- 了解 React，公司最近一个项目（[mis](http://mis.sendcloud.net)）是使用 React 开发的，状态管理选择的 Redux，设计框架选择的 ant-design。深入了解了 Redux，通过阅读源码掌握了其基本概念以及 Redux 中间件的实现，阅读了大部分热门中间件源码。
- 熟悉 jQuery 和 Zepto，阅读过 Zepto 源码，了解其实现机制。

## Node.js

- 掌握 koa，[iDevJS](https://github.com/idevjs) API 服务器是基于 koa1 开发的， 正打算升级到 koa2。koa1 的源码很简单，理解的难度主要在 co 上。koa1 的中间件是 generator 函数，通过 co 来进行异步流程控制，koa2 则是通过 async/await 来自动执行异步流程。
- 熟悉 expressjs，学习 Node.js 时是从 expressjs 开始的，用它开发过简单的博客练手，但没有大型项目经历。
- 了解 Node.js API，熟悉常用模块 path、http、stream 等。

<!-- ## 工程化

我对工程化的理解是 代码文件组织 + 部署打包工具等，代码文件组织因项目而异，所以前端工程化主要体现在各种工具的使用上：
- gulp：了解
- webpack  -->

<!-- ## 技术文章
- [AngularJS 应用优化指南](http://byevil.com/2016/05/10/AngularJS-app-performance-optimization-guide/)
- [深入学习 Redux](http://byevil.com/2016/01/25/dive-into-redux/)
- [查看更多](http://byevil.com) -->

## 其他

- 掌握 OAuth2 认证原理和实现机制，自己实现了一个 OAuth2 认证服务（[iDevJS](https://github.com/idevjs) API 服务)，熟悉 REST API 的设计和编写。
- 熟悉 Git，掌握常见命令，看了 gitbook 的前半部分，了解 git 工作流程。
- 熟悉自适应邮件的编写，了解如何让邮件兼容各个邮件客户端。
- 了解 Mongodb 和 Redis，[iDevJS](https://github.com/idevjs) 数据库使用的是 Mongodb，session 和 缓存使用的 Redis。
- 了解 Nginx，知道基本配置，能够部署简单 Nginx 服务器。
- 了解 Vim，日常通过 Vim 进行简单编辑操作。
- 无压力阅读英文资料，日常通过 Google 和 stackoverflow 搜索遇到的技术问题。

## 技术展望

- 继续深入学习 React、Angular2、Vue 等前端框架，了解其实现机制，掌握优化技巧，知道其各自优缺点和背后的思想。
- 深入学习 Node.js，感觉自己一直在应用层面，了解 API，会用 web 框架而已，并不了解底层实现，希望可以钻的深一点。
- 学习 Linux，加强计算机技术理论知识的学习。
- 了解算法。

## 自我评价

对前端发自内心的热爱，始终保持学习的心态，在掌握好基础的同时，尽力跟上时代的潮流。乐于分享，热爱开源，有良好的沟通能力和团队协作能力。

## 联系方式
- Email：cxh710200711@gmail.com
- 手机：18771037680
- 微信：HUSTecho
- QQ： 710200711
- 博客：[byevil.com](http://byevil.com)
- Github：[xuhong](https://github.com/xuhong)