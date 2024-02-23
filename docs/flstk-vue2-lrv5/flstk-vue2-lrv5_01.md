# 第一章：你好 Vue - Vue.js 简介

欢迎来到《全栈 Vue.js 2 和 Laravel 5》！在本章中，我们将对 Vue.js 进行高层次的概述，让您熟悉它的功能，为学习如何使用它做好准备。

我们还将熟悉本书中的主要案例研究项目 Vuebnb。

本章涵盖的主题：

+   Vue 的基本特性，包括模板、指令和组件

+   Vue 的高级特性，包括单文件组件和服务器端渲染

+   Vue 生态系统中的工具，包括 Vue Devtools、Vue Router 和 Vuex

+   您将在本书中逐步构建的主要案例研究项目是 Vuebnb

+   安装项目代码的说明

# 介绍 Vue.js

截至 2017 年底，Vue.js 的版本是 2.5。在首次发布不到四年的时间里，Vue 已经成为 GitHub 上最受欢迎的开源项目之一。这种受欢迎程度部分是由于其强大的功能，也是由于其强调开发者体验和易于采用。

Vue.js 的核心库，像 React 一样，只用于从 MVC 架构模式中操纵视图层。然而，Vue 有两个官方支持库，Vue Router 和 Vuex，分别负责路由和数据管理。

Vue 不像 React 和 Angular 那样得到科技巨头的支持，而是依赖于少数公司赞助商和专门的 Vue 用户的捐赠。更令人印象深刻的是，Evan You 目前是唯一的全职 Vue 开发人员，尽管来自世界各地的 20 多名核心团队开发人员协助开发、维护和文档编写。

Vue 的关键设计原则如下：

+   **重点**：Vue 选择了一个小而集中的 API，它的唯一目的是创建 UI

+   **简单性**：Vue 的语法简洁易懂

+   **紧凑**：核心库脚本压缩后约为 25 KB，比 React 甚至 jQuery 都要小

+   **速度**：渲染基准超过了许多主要框架，包括 React

+   **多功能性**：Vue 非常适合小型任务，您可能会使用 jQuery，但也可以扩展为合法的 SPA 解决方案

# 基本特性

现在让我们对 Vue 的基本特性进行高层次的概述。如果您愿意，您可以在计算机上创建一个 HTML 文件，如下所示，然后在浏览器中打开它，并按照以下示例进行编码。

如果你宁愿等到下一章，当我们开始进行案例研究项目时，那也可以，因为我们这里的目标只是为了感受一下 Vue 能做什么：

```php
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <title>Hello Vue</title>
</head>
<body>
  <!--We'll be adding stuff here!-->
</body>
</html>
```

# 安装

尽管 Vue 可以在更复杂的设置中作为 JavaScript 模块使用，但它也可以简单地作为外部脚本包含在 HTML 文档的主体中：

```php
<script src="https://unpkg.com/vue/dist/vue.js"></script>
```

# 模板

默认情况下，Vue 将使用 HTML 文件作为其模板。包含的脚本将声明 Vue 的一个实例，并在配置对象中使用`el`属性告诉 Vue 在模板中的哪个位置挂载应用程序：

```php
<div id="app">
  <!--Vue has dominion within this node-->
</div>
<script> new Vue({ el: '#app'
  }); </script>
```

我们可以通过将其创建为`data`属性并使用 mustache 语法将其打印到页面中，将数据绑定到我们的模板中：

```php
<div id="app"> {{ message }} <!--Renders as "Hello World"-->
</div>
<script> new Vue({ el: '#app', data: { message: 'Hello World'
    }
  }); </script>
```

# 指令

与 Angular 类似，我们可以使用**指令**向我们的模板添加功能。这些是我们添加到以`v-`前缀开头的 HTML 标签的特殊属性。

假设我们有一个数据数组。我们可以使用`v-for`指令将这些数据呈现为页面上的连续 HTML 元素：

```php
<div id="app">
  <h3>Grocery list</h3>
  <ul>
    <li v-for="grocery in groceries">{{ grocery }}</li>
  </ul>
</div>
<script> var app = new Vue({ el: '#app', data: { groceries: [ 'Bread', 'Milk' ]
    }
  }); </script>
```

上述代码呈现如下：

```php
<div id="app">
  <h3>Grocery list</h3>
  <ul>
    <li>Bread</li>
    <li>Milk</li>
  </ul>
</div>
```

# 响应性

Vue 设计的一个关键特性是其响应性系统。当您修改数据时，视图会自动更新以反映这一变化。

例如，如果我们创建一个函数，在页面已经呈现后将另一个项目推送到我们的杂货项目数组中，页面将自动重新呈现以反映这一变化：

```php
setTimeout(function() { app.groceries.push('Apples');
}, 2000);
```

初始渲染后两秒，我们看到了这个：

```php
<div id="app">
  <h3>Grocery list</h3>
  <ul>
    <li>Bread</li>
    <li>Milk</li>
    <li>Apples</li>
  </ul>
</div>
```

# 组件

组件扩展了基本的 HTML 元素，并允许您创建自己的可重用自定义元素。

例如，这里我创建了一个自定义元素`grocery-item`，它渲染为一个`li`。该节点的文本子节点来自自定义 HTML 属性`title`，在组件代码内部可以访问到：

```php
<div id="app">
  <h3>Grocery list</h3>
  <ul>
    <grocery-item title="Bread"></grocery-item>
    <grocery-item title="Milk"></grocery-item>
  </ul>
</div>
<script> Vue.component( 'grocery-item', { props: [ 'title' ], template: '<li>{{ title }}</li>'
  });

  new Vue({ el: '#app'
  }); </script>
```

这样渲染：

```php
<div id="app">
  <h3>Grocery list</h3>
  <ul>
    <li>Bread</li>
    <li>Milk</li>
  </ul>
</div>
```

但使用组件的主要原因可能是它更容易构建一个更大的应用程序。功能可以被分解为可重用的、自包含的组件。

# 高级功能

如果你迄今为止一直在跟着示例编码，那么请关闭你的浏览器，直到下一章，因为以下高级片段不能简单地包含在浏览器脚本中。

# 单文件组件

使用组件的一个缺点是，你需要在主 HTML 文件之外的 JavaScript 字符串中编写你的模板。虽然有办法在 HTML 文件中编写模板定义，但这样就会在标记和逻辑之间产生尴尬的分离。

一个方便的解决方案是**单文件组件**：

```php
<template>
  <li v-on:click="bought = !bought" v-bind:class="{ bought: bought }">
    <div>{{ title }}</div>
  </li>
</template>
<script> export default { props: [ 'title' ], data: function() {
      return { bought: false
      };
    }
  } </script>
<style> .bought {
    opacity: 0.5;
  } </style>
```

这些文件的扩展名是`.vue`，它们封装了组件模板、JavaScript 配置和样式，全部在一个文件中。

当然，网页浏览器无法读取这些文件，因此它们需要首先通过 Webpack 这样的构建工具进行处理。

# 模块构建

正如我们之前所看到的，Vue 可以作为外部脚本直接在浏览器中使用。Vue 也可以作为 NPM 模块在更复杂的项目中使用，包括像 Webpack 这样的构建工具。

如果你对 Webpack 不熟悉，它是一个模块打包工具，可以将所有项目资产捆绑在一起，形成可以提供给浏览器的东西。在捆绑过程中，你也可以转换这些资产。

使用 Vue 作为模块，并引入 Webpack，可以开启以下可能性：

+   单文件组件

+   当前浏览器不支持的 ES 功能提案

+   模块化的代码

+   预处理器，如 SASS 和 Pug

我们将在第五章中更深入地探索 Webpack，*使用 Webpack 集成 Laravel 和 Vue.js*。

# 服务器端渲染

服务器端渲染是增加全栈应用程序加载速度感知度的好方法。用户在加载您的网站时会得到一个完整的页面，而不是直到 JavaScript 运行时才会填充的空页面。

假设我们有一个使用组件构建的应用。如果我们使用浏览器开发工具在页面加载后查看我们的页面 DOM，我们将看到我们完全渲染的应用：

```php
<div id="app">
  <ul>
    <li>Component 1</li>
    <li>Component 2</li>
    <li>
      <div>Component 3</div>
    </li>
  </ul>
</div>
```

但是，如果我们查看文档的源代码，也就是`index.html`，就像服务器发送时一样，你会看到它只有我们的挂载元素：

```php
<div id="app"></div>
```

为什么？因为 JavaScript 负责构建我们的页面，因此 JavaScript 必须在页面构建之前运行。但是通过服务器端渲染，我们的`index`文件包含了浏览器在下载和运行 JavaScript 之前构建 DOM 所需的 HTML。应用程序加载速度并没有变快，但内容会更快地显示出来。

# Vue 生态系统

虽然 Vue 是一个独立的库，但与其生态系统中的一些可选工具结合使用时，它会变得更加强大。对于大多数项目，你将在前端堆栈中包含 Vue Router 和 Vuex，并使用 Vue Devtools 进行调试。

# Vue 开发者工具

Vue Devtools 是一个浏览器扩展，可以帮助你开发 Vue.js 项目。除其他功能外，它允许你查看应用程序中组件的层次结构和组件的状态，这对调试很有用：

![](img/e2a740c8-9968-41bd-abc5-bd75678dc1e8.png)图 1.1 Vue Devtools 组件层次结构

我们将在本节的后面看到它还能做什么。

# Vue 路由

Vue Router 允许你将 SPA 的不同状态映射到不同的 URL，给你虚拟页面。例如，`mydomain.com/`可能是博客的首页，并且有这样的组件层次结构：

```php
<div id="app">
  <my-header></my-header>
  <blog-summaries></blog-summaries>
  <my-footer></my-footer>
</div>
```

而`mydomain.com/post/1`可能是博客中的一个单独的帖子，看起来像这样：

```php
<div id="app">
  <my-header></my-header>
  <blog-post post-id="id">
  <my-footer></my-footer>
</div>
```

从一个页面切换到另一个页面不需要*重新加载*页面，只需交换中间组件以反映 URL 的状态，这正是 Vue Router 所做的。

# Vuex

Vuex 提供了一种强大的方式来管理应用程序的数据，随着 UI 的复杂性增加，它将应用程序的数据集中到一个单一的存储中。

我们可以通过检查 Vue Devtools 中的存储来获取应用程序状态的快照：

![](img/bf4b939f-e14b-4b6f-9ab0-a1610a226a53.png)图 1.2 Vue Devtools Vuex 标签

左侧列跟踪对应用程序数据所做的更改。例如，用户保存或取消保存项目。您可以将此事件命名为`toggleSaved`。Vue Devtools 允许您在事件发生时查看此事件的详细信息。

我们还可以在不触及代码或重新加载页面的情况下恢复到数据的任何先前状态。这个功能称为*时间旅行调试*，对于调试复杂的 UI 来说，您会发现它非常有用。

# 案例研究项目

在快速概述了 Vue 的主要特性之后，我相信您现在渴望开始真正学习 Vue 并将其投入使用。让我们首先看一下您将在整本书中构建的案例研究项目。

# Vuebnb

Vuebnb 是一个现实的全栈 Web 应用程序，它利用了 Vue、Laravel 和本书中涵盖的其他工具和设计模式的许多主要特性。

从用户的角度来看，Vuebnb 是一个在线市场，可以在世界各地的城市租用短期住宿。您可能会注意到 Vuebnb 和另一个名字类似的住宿在线市场之间的一些相似之处！

您可以在此处查看 Vuebnb 的完成版本：[`vuebnb.vuejsdevelopers.com`](http://vuebnb.vuejsdevelopers.com)。

如果您现在没有互联网访问权限，这里有两个主要页面的截图。首先是主页，用户可以在这里搜索或浏览住宿选项：

![](img/e7edf13d-6cd8-4425-805a-8859ba7c33ca.png)图 1.3 Vuebnb 主页

其次是列表页面，用户可以在这里查看特定于可能有兴趣租用的单个住宿的信息：

![](img/8b696452-b4fe-4c1f-9e5e-bef57ff51798.png)图 1.4 Vuebnb 列表页面

# 代码库

案例研究项目贯穿整本书的整个持续时间，因此一旦您创建了代码库，您可以逐章添加内容。最终，您将从头开始构建和部署一个全栈应用程序。

代码库位于 GitHub 存储库中。您可以将其下载到计算机上通常放置项目的任何文件夹中，例如`~/Projects`：

```php
$ cd ~/Projects
$ git clone https://github.com/PacktPublishing/Full-Stack-Vue.js-2-and-Laravel-5
$ cd Full-Stack-Vue.js-2-and-Laravel-5
```

与其直接克隆此存储库，不如首先进行*分叉*，然后再克隆。这样可以让您随意进行任何更改，并将您的工作保存到您自己的远程存储库。这里有一个关于在 GitHub 上进行分叉存储库的指南：[`help.github.com/articles/fork-a-repo/`](https://help.github.com/articles/fork-a-repo/)。

# 文件夹

代码库包含以下文件夹：

![](img/518ac8ab-b6d2-40e6-895e-3104c7e15fb3.png)图 1.5 代码库目录内容

以下是每个文件夹的用途概述：

+   `Chapter02`到`Chapter10`包含每章的代码的*完成状态*（不包括本章）

+   *images*目录包含 Vuebnb 中用于使用的示例图片。这将在第四章中进行解释，*使用 Laravel 构建 Web 服务*

+   *vuebnb*是您将在第三章开始工作的主要案例研究项目的项目代码，*设置 Laravel 开发环境*

+   *vuebnb-prototype*是 Vuebnb 原型的项目代码，我们将在第二章中构建，*原型设计 Vuebnb，您的第一个 Vue.js 项目*

# 总结

在本章中，我们对 Vue.js 进行了高层次的介绍，涵盖了模板、指令和组件等基本特性，以及单文件组件和服务器端渲染等高级特性。我们还看了 Vue 生态系统中的工具，包括 Vue Router 和 Vuex。

然后我们对 Vuebnb 进行了概述，这是您在阅读本书时将要构建的全栈项目，并了解了如何从 GitHub 安装代码库。

在下一章中，我们将适当地了解 Vue 的基本特性，并开始通过构建 Vuebnb 的原型来使用它们。
