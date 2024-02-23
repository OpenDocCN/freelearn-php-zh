# 前言

现在是 2014 年，**单页应用**（SPA）解决方案的战争真正激烈。有许多竞争对手：Angular、React、Ember、Knockout 和 Backbone 等等。然而，最受关注的战斗是在谷歌的 Angular 和 Facebook 的 React 之间。

直到这一点，SPA 之王 Angular 是一个完整的框架，遵循熟悉的 MVC 范例。而 React，这个不太可能的挑战者，与其核心库只处理视图层，而且标记完全由 JavaScript 编写，相比之下似乎相当奇怪！虽然 Angular 占据着更大的市场份额，但 React 在开发人员思考 Web 应用设计的方式上引起了巨大的变革，并提高了框架的大小和性能。

与此同时，一位名叫 Evan You 的开发人员正在尝试自己的新框架 Vue.js。它将结合 Angular 和 React 的最佳特性，实现简单和强大之间的完美平衡。你的愿景与其他开发人员的共鸣如此之好，以至于 Vue 很快就成为最受欢迎的 SPA 解决方案之一。

尽管竞争激烈，但 Vue 迅速获得了关注。这在一定程度上要归功于 Laravel 的创始人 Taylor Otwell，他在 2015 年初发推特称赞 Vue 的印象深刻。这条推文引起了 Laravel 社区对 Vue 的极大兴趣。

Vue 和 Laravel 的合作将在 2016 年 9 月发布的 Laravel 5.3 版本中进一步紧密结合，当时 Vue 被包含为默认的前端库。对于具有相同理念的两个软件项目来说，这是一个完全合乎逻辑的联盟：简单和开发者体验的重点。

如今，Vue 和 Laravel 为开发 Web 应用提供了一个非常强大和灵活的全栈框架，正如你将在本书中发现的那样，它们是非常愉快的工作对象。

# 本书涵盖的内容

构建一个全栈应用需要广泛的知识，不仅仅是关于 Vue 和 Laravel，还包括 Vue Router、Vuex 和 Webpack，更不用说 JavaScript、PHP 和 Web 开发的一般知识了。

因此，作为作者，我面临的最大挑战之一是决定应该包括什么，不应该包括什么。我最终确定的主题是对以下两个问题的回答：

+   读者在所有或大多数 Vue.js 应用中将使用的基本特性、工具和设计模式是什么？

+   相对于其他架构，设计和构建全栈 Vue.js 应用的关键问题是什么？

以下是本书各章节涉及的主题分布：

第一章《你好 Vue - Vue.js 简介》介绍了 Vue.js 的概述，以及本书的案例研究项目*Vuebnb*。

第二章《原型设计 Vuebnb，你的第一个 Vue.js 项目》提供了 Vue.js 基本特性的实际介绍，包括安装、模板语法、指令、生命周期钩子等。

第三章《设置 Laravel 开发环境》展示了如何为全栈 Vue.js 应用设置一个新的 Laravel 项目。

第四章《使用 Laravel 构建 Web 服务》是关于为我们的案例研究项目的后端奠定基础，包括设置数据库、模型和 API 端点。

第五章《使用 Webpack 集成 Laravel 和 Vue.js》解释了一个复杂的 Vue 应用将需要构建步骤，并介绍了用于捆绑项目资产的 Webpack。

第六章《使用 Vue.js 组件构建小部件》教授了组件是现代 UI 开发的一个基本概念，也是 Vue.js 最强大的功能之一。

第七章《使用 Vue Router 构建多页面应用》介绍了 Vue Router，并展示了如何在前端应用中添加虚拟页面。

第八章《使用 Vuex 管理应用状态》解释了状态管理是管理复杂 UI 数据的必备功能。我们介绍了 Flux 模式和 Vuex。

第九章，“使用 Passport 添加用户登录和 API 身份验证”，专注于全栈应用程序中最棘手的部分之一——身份验证。本章展示了如何使用 Passport 进行安全的 AJAX 调用到后端。

第十章，“将全栈应用部署到云端”，描述了如何构建和部署我们完成的项目到基于云的服务器，并使用 CDN 来提供静态资产。

# 你需要为这本书做好准备

在你开始案例研究项目的开发之前，你必须确保你有正确的软件和硬件。

# 操作系统

你可以使用基于 Windows 或 Linux 的操作系统。不过我是 Mac 用户，所以本书中使用的所有终端命令都将是 Linux 命令。

请注意我们将使用 Homestead 虚拟开发环境，其中包括 Ubuntu Linux 操作系统。如果你 SSH 进入这个虚拟机并从那里运行所有终端命令，你可以使用和我一样的命令，即使你使用的是 Windows 主机操作系统。

# 开发工具

下载项目代码将需要 Git。如果你还没有安装 Git，请按照这个指南中的说明进行：[`git-scm.com/book/en/v2/Getting-Started-Installing-Git`](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git)。

要开发一个 JavaScript 应用程序，你需要 Node.js 和 NPM。它们可以从同一个软件包中安装；请参阅这里的说明：[`nodejs.org/en/download/`](https://nodejs.org/en/download/)。

我们还将使用 Laravel Homestead。第三章将提供设置 Laravel 开发环境的说明。

# 浏览器

Vue 需要 ECMAScript 5，这意味着你可以使用任何主流浏览器的最新版本来运行它。我建议你使用 Google Chrome，因为我将为 Chrome Dev Tools 提供调试示例，如果你也使用 Chrome，那么你跟着学会会更容易。

在选择浏览器时，你还应该考虑与 Vue Devtools 的兼容性。

# Vue Devtools

Vue Devtools 浏览器扩展使得调试 Vue 变得轻而易举，在本书中我们将大量使用它。这个扩展是为 Google Chrome 设计的，但也可以在 Firefox 中使用（还有 Safari，需要稍微修改一下）。

查看以下链接以获取更多信息和安装说明：[`github.com/vuejs/vue-devtools`](https://github.com/vuejs/vue-devtools)

# IDE

当然，你需要一个文本编辑器或 IDE 来开发案例研究项目。

# 硬件

你需要一台配置足够安装和运行上述软件的计算机。最消耗资源的程序将是 VirtualBox 5.2（或 VMWare 或 Parallels），我们将使用它来设置 Homestead 虚拟开发环境。

你还需要一个互联网连接来下载源代码和项目依赖。

# 这本书适合谁

这本书是为寻求使用 Vue.js 和 Laravel 进行全栈开发的 Laravel 开发者而写的，提供了实用和最佳实践的方法。

任何对这个主题感兴趣的网页开发者都可以成功使用这本书，只要他们满足以下条件：

| 主题 | 级别 |
| --- | --- |
| HTML 和 CSS | 中级知识 |
| JavaScript | 中级知识 |
| PHP | 中级知识 |
| Laravel | 基础知识 |
| Git | 基础知识 |

请注意读者不需要有 Vue.js 或其他 JavaScript 框架的经验。

# 约定

在本书中，你会发现一些文本样式，用来区分不同类型的信息。以下是一些这些样式的例子和它们的含义解释。

文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 用户名都显示如下：“例如，在这里我创建了一个自定义元素，`grocery-item`，它呈现为`li`。”

代码块设置如下：

```php
<div id="app">
  <!--Vue has dominion within this node-->
</div>
<script> new Vue({ el: '#app'
  }); </script>
```

任何命令行输入或输出都以以下方式编写：

```php
$ npm install
```

**新术语**和**重要词汇**以粗体显示。屏幕上看到的词语，例如菜单或对话框中的词语，会以这样的方式出现在文本中："Vue 不允许这样做，如果你尝试会出现这个错误：不要将 Vue 挂载到 <html> 或 <body> - 而是挂载到普通元素上。"

警告或重要提示会以这样的方式出现在一个框中。提示和技巧会以这样的方式出现。
