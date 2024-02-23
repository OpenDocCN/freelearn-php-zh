# 第二章：Vuebnb 原型，您的第一个 Vue.js 项目

在本章中，我们将学习 Vue.js 的基本特性。然后，我们将把这些知识付诸实践，通过构建 Vuebnb 的案例研究项目原型。

本章涵盖的主题：

+   Vue.js 的安装和基本配置

+   Vue.js 的基本概念，如数据绑定、指令、观察者和生命周期钩子

+   Vue 的响应系统是如何工作的

+   案例研究项目的项目要求

+   使用 Vue.js 添加页面内容，包括动态文本、列表和页眉图像

+   使用 Vue 构建图像模态 UI 功能

# Vuebnb 原型

在本章中，我们将构建 Vuebnb 的原型，这是本书持续运行的案例研究项目。原型将只是列表页面，到本章结束时将会是这样的：

![](img/9596f4dc-9623-43d0-8f51-f6ab7f04aa46.png)图 2.1 Vuebnb 原型

一旦我们在第三章 *设置 Laravel 开发环境*和第四章 *使用 Laravel 构建 Web 服务*中设置好了我们的后端，我们将把这个原型迁移到主项目中。

# 项目代码

在开始之前，您需要通过从 GitHub 克隆代码库将其下载到您的计算机上。在第一章的*代码库*部分中给出了说明，*你好 Vue - Vue.js 简介*。

`vuebnb-prototype`文件夹中包含了我们将要构建的原型的项目代码。切换到该文件夹并列出其中的内容：

```php
$ cd vuebnb-prototype
$ ls -la
```

文件夹内容应该如下所示：

![](img/da98e41f-aca7-4d8c-9f95-07ea87433c5a.png)图 2.2 vuebnb-prototype 项目文件除非另有说明，本章中所有后续的终端命令都假定您在`vuebnb-prototype`文件夹中。

# NPM 安装

您现在需要安装此项目中使用的第三方脚本，包括 Vue.js 本身。NPM `install`方法将读取包含的`package.json`文件并下载所需的模块：

```php
$ npm install
```

您现在会看到您的项目文件夹中出现了一个新的`node_modules`目录。

# 主要文件

在 IDE 中打开`vuebnb-prototype`目录。请注意，包含了以下`index.html`文件。它主要由样板代码组成，但也包括一些结构标记在`body`标签中。

还要注意，该文件链接到`style.css`，我们的 CSS 规则将被添加在那里，以及`app.js`，我们的 JavaScript 将被添加在那里。

`index.html`：

```php
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1">
  <meta name="viewport" content="width=device-width,initial-scale=1">
  <title>Vuebnb</title>
  <link href="node_modules/open-sans-all/css/open-sans.css" rel="stylesheet">
  <link rel="stylesheet" href="style.css" type="text/css">
</head>
<body>
<div id="toolbar">
  <img class="icon" src="logo.png">
  <h1>vuebnb</h1>
</div>
<div id="app">
  <div class="container"></div>
</div>
<script src="app.js"></script>
</body>
</html>
```

目前`app.js`是一个空文件，但我已经在`style.css`中包含了一些 CSS 规则来帮助我们入门。

`style.css`：

```php
body {
  font-family: 'Open Sans', sans-serif; color: #484848; font-size: 17px;
  margin: 0;
}

.container {
  margin: 0 auto;
  padding: 0 12px;
}

@media (min-width: 744px) {
  .container {
      width: 696px;
  }
}

#toolbar {
  display: flex;
  align-items: center;
  border-bottom: 1px solid #e4e4e4;
  box-shadow: 0 1px 5px rgba(0, 0, 0, 0.1);
}

#toolbar .icon {
  height: 34px;
  padding: 16px 12px 16px 24px;
  display: inline-block;
}

#toolbar h1 {
  color: #4fc08d;
  display: inline-block;
  font-size: 28px;
  margin: 0;
}
```

# 在浏览器中打开

要查看项目，请在 Web 浏览器中找到`index.html`文件。在 Chrome 中，只需点击文件*| 打开文件*。加载完成后，您将看到一个大部分为空白的页面，除了顶部的工具栏。

# 安装 Vue.js

现在是时候将`Vue.js`库添加到我们的项目中了。Vue 已作为我们的 NPM 安装的一部分下载，所以现在我们可以简单地使用`script`标签链接到`Vue.js`的浏览器构建版本。

`index.html`：

```php
<body>
<div id="toolbar">...</div>
<div id="app">...</div>
<script src="node_modules/vue/dist/vue.js"></script>
<script src="app.js"></script>
</body>
```

在我们自定义的`app.js`脚本之前，包含 Vue 库是很重要的，因为脚本是按顺序运行的。

Vue 现在将被注册为全局对象。我们可以通过转到浏览器并在 JavaScript 控制台中输入以下内容来测试：

```php
console.log(Vue);
```

这是结果：

![](img/25755703-7819-4ee5-b148-6b33180bb963.png)图 2.3 检查 Vue 是否注册为全局对象

# 页面内容

当我们的环境设置好并安装好了起始代码后，我们现在已经准备好开始构建 Vuebnb 原型的第一步了。

让我们向页面添加一些内容，包括页眉图像、标题和*关于*部分。我们将在我们的 HTML 文件中添加结构，并使用`Vue.js`在需要时插入正确的内容。

# Vue 实例

查看我们的`app.js`文件，现在让我们通过使用`Vue`对象的`new`运算符来创建 Vue.js 的根实例。

`app.js`：

```php
var app = new Vue();
```

当你创建一个`Vue`实例时，通常会希望将一个配置对象作为参数传递进去。这个对象是定义项目的自定义数据和函数的地方。

`app.js`：

```php
var app = new Vue({ el: '#app'
});
```

随着我们的项目的进展，我们将在这个配置对象中添加更多内容，但现在我们只是添加了`el`属性，告诉 Vue 在页面中的哪里挂载自己。

你可以将其分配为一个字符串（CSS 选择器）或 HTML 节点对象。在我们的例子中，我们使用了`#app`字符串，它是一个 CSS 选择器，指的是具有`app`ID 的元素。

`index.html`：

```php
<div id="app">
  <!--Mount element-->
</div>
```

![](img/b44d2a8b-d651-4e4c-8f99-85ee662df265.png)图 2.5。包含模拟列表示例的页面

`index.html`：

```php
<body>
<div id="toolbar">...</div>
<div id="app">
  <!--Vue only has dominion here-->
  <div class="header">...</header> ... </div>
<script src="node_modules/vue/dist/vue.js"></script>
<script src="app.js"></script>
</body>
```

从现在开始，我们将把我们的挂载节点及其子节点称为我们的模板。

# 数据绑定

Vue 的一个简单任务是将一些 JavaScript 数据绑定到模板上。让我们在配置对象中创建一个`data`属性，并为其分配一个包含`title`属性和`'My apartment'`字符串值的对象。

`app.js`：

```php
var app = new Vue({ el: '#app', data: { title: 'My apartment'
  }
});
```

这个`data`对象的任何属性都将在我们的模板中可用。为了告诉 Vue 在哪里绑定这些数据，我们可以使用*mustache*语法，也就是双花括号，例如，`{{ myProperty }}`。当 Vue 实例化时，它会编译模板，用适当的文本替换 mustache 语法，并更新 DOM 以反映这一点。这个过程被称为*文本插值*，并在下面的代码块中进行了演示。

`index.html`：

```php
<div id="app">
  <div class="container">
    <div class="heading">
      <h1>{{ title }}</h1>
    </div>
  </div>
</div>
```

将呈现为：

```php
<div id="app">
  <div class="container">
    <div class="heading">
      <h1>My apartment</h1>
    </div>
  </div>
</div>
```

现在让我们添加一些更多的数据属性，并增强我们的模板以包含更多的页面结构。

`app.js`：

```php
var app = new Vue({ el: '#app', data: { title: 'My apartment', address: '12 My Street, My City, My Country', about: 'This is a description of my apartment.'
  }
});
```

`index.html`：

```php
<div class="container">
  <div class="heading">
    <h1>{{ title }}</h1>
    <p>{{ address }}</p>
  </div>
  <hr>
  <div class="about">
    <h3>About this listing</h3>
    <p>{{ about }}</p>
  </div>
</div>
```

让我们也添加一些新的 CSS 规则。

`style.css`：

```php
.heading {
  margin-bottom: 2em;
}

.heading h1 {
  font-size: 32px;
  font-weight: 700;
}

.heading p {
  font-size: 15px;
  color: #767676;
}

hr {
  border: 0;
  border-top: 1px solid #dce0e0;
}

.about {
  margin-top: 2em;
}

.about h3 {
  font-size: 22px;
}

.about p {
  white-space: pre-wrap;
}
```

如果你现在保存并刷新你的页面，它应该看起来像这样：

![](img/bbb14c0f-4e35-413a-8b62-2250d8786bec.png)图 2.4。带有基本数据绑定的列表页面

# 模拟列表

当我们开发时，最好使用一些模拟数据，这样我们就可以看到我们完成的页面将会是什么样子。我已经在项目中包含了`sample/data.js`，就是为了这个原因。让我们在我们的文档中加载它，确保它在我们的`app.js`文件之上。

`index.html`：

```php
<body>
<div id="toolbar">...</div>
<div id="app">...</div>
<script src="node_modules/vue/dist/vue.js"></script>
<script src="sample/data.js"></script>
<script src="app.js"></script>
</body>
```

看一下文件，你会看到它声明了一个`sample`对象。我们现在将在我们的数据配置中利用它。

`app.js`：

```php
data: { title: sample.title, address: sample.address, about: sample.about }
```

一旦你保存并刷新，你会在页面上看到更真实的数据：

在这种方式中使用分布在不同脚本文件中的全局变量并不是一种理想的做法。不过，我们只会在原型中这样做，稍后我们将从服务器获取这个模拟列表示例。

# 页眉图像

没有一个房间列表会完整而没有一个大而光滑的图片来展示它。我们的模拟列表中有一个页眉图像，现在我们将其包含进来。将这个标记添加到页面中。

Vue 对其挂载的元素和任何子节点都具有支配权。到目前为止，对于我们的项目，Vue 可以操作具有`header`类的`div`，但无法操作具有`toolbar`ID 的`div`。放置在后者`div`中的任何内容对 Vue 来说都是不可见的。

```php
<div id="app">
  <div class="header">
    <div class="header-img"></div>
  </div>
  <div class="container">...</div>
</div>
```

并将其添加到 CSS 文件中。

`style.css`：

```php
.header {
  height: 320px;
}

.header .header-img {
  background-repeat: no-repeat;
  background-size: cover;
  background-position: 50% 50%;
  background-color: #f5f5f5;
  height: 100%;
}
```

你可能会想为什么我们使用`div`而不是`img`标签。为了帮助定位，我们将把我们的图像设置为具有`header-img`类的`div`的背景。

# 样式绑定

要设置背景图像，我们必须在 CSS 规则中提供 URL 作为属性，就像这样：

```php
.header .header-img {
  background-image: url(...);
}
```

显然，我们的页眉图像应该针对每个单独的列表进行特定设置，所以我们不想硬编码这个 CSS 规则。相反，我们可以让 Vue 将数据中的 URL 绑定到我们的模板上。

Vue 无法访问我们的 CSS 样式表，但它可以绑定到内联`style`属性：

```php
<div class="header-img" style="background-image: url(...);"></div>
```

你可能会认为在这里使用文本插值是解决方案，例如：

```php
<div class="header-img" style="background-image: {{ headerUrl }}"></div>
```

但这不是有效的 Vue.js 语法。相反，这是另一个 Vue.js 功能称为`指令`的工作。让我们先探索指令，然后再来解决这个问题。

# 指令

Vue 的指令是带有*v-*前缀的特殊 HTML 属性，例如`v-if`，它提供了一种简单的方法来为我们的模板添加功能。您可以为元素添加的一些指令的示例包括：

+   `v-if`：有条件地渲染元素

+   `v-for`：基于数组或对象多次渲染元素

+   `v-bind`：将元素的属性动态绑定到 JavaScript 表达式

+   `v-on`：将事件监听器附加到元素

在整本书中，我们将探索更多内容。

# 用法

就像普通的 HTML 属性一样，指令通常是形式为`name="value"`的名称/值对。要使用指令，只需将其添加到 HTML 标记中，就像添加属性一样，例如：

```php
<p v-directive="value">
```

# 表达式

如果指令需要一个值，它将是一个*表达式*。

在 JavaScript 语言中，表达式是小的、可评估的语句，产生单个值。表达式可以在期望值的任何地方使用，例如在`if`语句的括号中：

```php
if (expression) {
  ...
}
```

这里的表达式可以是以下任何一种：

+   数学表达式，例如`x + 7`

+   比较，例如`v <= 7`

+   例如 Vue 的`data`属性，例如`this.myval`

指令和文本插值都接受表达式值：

```php
<div v-dir="someExpression">{{ firstName + " " + lastName }}</div>
```

# 示例：v-if

`v-if`将根据其值是否为*真*表达式有条件地渲染元素。在下面的情况下，`v-if`将根据`myval`的值删除/插入`p`元素：

```php
<div id="app">
  <p v-if="myval">Hello Vue</p>
</div>
<script> var app = new Vue({ el: '#app', data: { myval: true
    }
  }); </script>
```

将呈现为：

```php
<div id="app">
  <p>Hello Vue</p>
</div>
```

如果我们添加一个带有`v-else`指令的连续元素（一个不需要值的特殊指令），它将在`myval`更改时对称地删除/插入：

```php
<p v-if="myval">Hello Vue</p>
<p v-else>Goodbye Vue</p>
```

# 参数

一些指令需要一个*参数*，在指令名称后面加上冒号表示。例如，`v-on`指令监听 DOM 事件，需要一个参数来指定应该监听哪个事件：

```php
<a v-on:click="doSomething">
```

参数不一定是`click`，也可以是`mouseenter`，`keypress`，`scroll`或任何其他事件（包括自定义事件）。

# 样式绑定（续）

回到我们的页眉图像，我们可以使用`v-bind`指令和`style`参数将值绑定到`style`属性。

`index.html`：

```php
<div class="header-img" v-bind:style="headerImageStyle"></div>
```

`headerImageStyle`是一个表达式，它评估为设置背景图像到正确 URL 的 CSS 规则。听起来很混乱，但当你看到它工作时，就会很清楚。

现在让我们创建`headerImageStyle`作为数据属性。当绑定到样式属性时，可以使用一个对象，其中属性和值等同于 CSS 属性和值。

`app.js`：

```php
data: {
  ... headerImageStyle: {
    'background-image': 'url(sample/header.jpg)'
  }
},
```

保存代码，刷新页面，页眉图像将显示：

![](img/6601a188-3ce0-4e1e-8b6e-7480bb40e6ff.png)图 2.6。包括页眉图像的页面

使用浏览器开发工具检查页面，并注意`v-bind`指令的评估方式：

```php
<div class="header-img" style="background-image: url('sample/header.jpg');"></div>
```

# 列表部分

我们将要添加到页面的下一部分是`Amenities`和`Prices`列表：

![](img/2b06075a-4384-4cc7-88c9-ce8d3e540bd0.png)图 2.7。列表部分

如果您查看模拟列表示例，您会看到对象上的`amenities`和`prices`属性都是数组。

`sample/data.js`：

```php
var sample = { title: '...', address: '...', about: '...', amenities: [
    { title: 'Wireless Internet', icon: 'fa-wifi'
    },
    { title: 'Pets Allowed', icon: 'fa-paw'
    },
    ...
  ], prices: [
    { title: 'Per night', value: '$89'
    },
    { title: 'Extra people', value: 'No charge'
    },
    ...
  ]
}
```

如果我们可以轻松地遍历这些数组并将每个项目打印到页面上，那不是很容易吗？我们可以！这就是`v-for`指令的作用。

首先，让我们将这些添加为根实例上的数据属性。

`app.js`：

```php
data: {
  ... amenities: sample.amenities, prices: sample.prices }
```

# 列表渲染

`v-for`指令需要一种特殊类型的表达式，形式为`item in items`，其中`items`是源数组，`item`是当前正在循环的数组元素的别名。

让我们首先处理`amenities`数组。该数组的每个成员都是一个具有`title`和`icon`属性的对象，即：

```php
{ title: 'something', icon: 'something' }
```

我们将在模板中添加`v-for`指令，并将其分配的表达式设置为`amenity in amenities`。表达式的别名部分，即`amenity`，将在整个循环序列中引用数组中的每个对象，从第一个开始。

`index.html`：

```php
<div class="container">
  <div class="heading">...</div>
  <hr>
  <div class="about">...</div>
  <div class="lists">
    <div v-for="amenity in amenities">{{ amenity.title }}</div>
  </div>
</div>
```

它将呈现为：

```php
<div class="container">
  <div class="heading">...</div>
  <hr>
  <div class="about">...</div>
  <div class="lists">
    <div>Wireless Internet</div>
    <div>Pets Allowed</div>
    <div>TV</div>
    <div>Kitchen</div>
    <div>Breakfast</div>
    <div>Laptop friendly workspace</div>
  </div>
</div>
```

# 图标

我们设施对象的第二个属性是`icon`。这实际上是与 Font Awesome 图标字体中的图标相关的类。我们已经安装了 Font Awesome 作为 NPM 模块，因此现在可以将其添加到页面的头部以使用它。

`index.html`：

```php
<head> ... <link rel="stylesheet" href="node_modules/open-sans-all/css/open-sans.css">
  <link rel="stylesheet" href="node_modules/font-awesome/css/font-awesome.css">
  <link rel="stylesheet" href="style.css" type="text/css">
</head>
```

现在，我们可以在模板中完成我们的设施部分的结构。

`index.html`：

```php
<div class="lists">
  <hr>
  <div class="amenities list">
    <div class="title"><strong>Amenities</strong></div>
    <div class="content">
      <div class="list-item" v-for="amenity in amenities">
        <i class="fa fa-lg" v-bind:class="amenity.icon"></i>
        <span>{{ amenity.title }}</span>
      </div>
    </div>
  </div>
</div>
```

`style.css`：

```php
.list {
  display: flex;
  flex-wrap: nowrap;
  margin: 2em 0;
}

.list .title {
  flex: 1 1 25%;
}

.list .content {
  flex: 1 1 75%;
  display: flex;
  flex-wrap: wrap;
}

.list .list-item {
  flex: 0 0 50%;
  margin-bottom: 16px;
}

.list .list-item > i {
  width: 35px;
}

@media (max-width: 743px) {
  .list .title {
    flex: 1 1 33%;
  }

  .list .content {
    flex: 1 1 67%;
  }

  .list .list-item {
    flex: 0 0 100%;
  }
}
```

# 键

正如你所期望的那样，由`v-for="amenity in amenities"`生成的 DOM 节点与`amenities`数组具有响应性绑定。如果`amenities`的内容发生变化，Vue 将自动重新渲染节点以反映更改。

在使用`v-for`时，建议为列表中的每个项目提供一个唯一的`key`属性。这使得 Vue 能够定位需要更改的确切 DOM 节点，从而使 DOM 更新更有效率。

通常，键将是一个数字 ID，例如：

```php
<div v-for="item in items" v-bind:key="item.id"> {{ item.title }} </div>
```

对于设施和价格列表，内容在应用程序的生命周期内不会发生变化，因此我们不需要提供键。一些代码检查工具可能会警告您，但在这种情况下，可以安全地忽略警告。

# 价格

现在让我们也将价格列表添加到我们的模板中。

`index.html`：

```php
<div class="lists">
  <hr>
  <div class="amenities list">...</div>
  <hr>
  <div class="prices list">
    <div class="title">
      <strong>Prices</strong>
    </div>
    <div class="content">
      <div class="list-item" v-for="price in prices"> {{ price.title }}: <strong>{{ price.value }}</strong>
      </div>
    </div>
  </div>
</div>
```

我相信您会同意，循环模板比逐个项目编写要容易得多。但是，您可能会注意到这两个列表之间仍然存在一些常见的标记。在本书的后面，我们将利用组件使模板的这一部分更加模块化。

# 显示更多功能

现在我们遇到了一个问题，即“列表”部分在“关于”部分之后。关于部分的长度是任意的，在我们将要添加的一些模拟列表中，您会看到这部分非常长。

我们不希望它主导页面并迫使用户进行大量不受欢迎的滚动以查看“列表”部分，因此我们需要一种方法，如果文本太长，则隐藏一些文本，但允许用户选择查看完整文本。

让我们添加一个“显示更多”UI 功能，它将在一定长度后裁剪“关于”文本，并为用户提供一个按钮来显示隐藏的文本：

![](img/2097c214-5dcc-43d3-82d1-2d682fe49996.png)图 2.8. 显示更多功能

我们将首先向包含`about`文本插值的`p`标签添加一个`contracted`类。此类的 CSS 规则将限制其高度为 250 像素，并隐藏溢出元素的任何文本。

`index.html`：

```php
<div class="about">
  <h3>About this listing</h3>
  <p class="contracted">{{ about }}</p>
</div>
```

`style.css`：

```php
.about p.contracted {
  height: 250px;
  overflow: hidden;
}
```

我们还将在`p`标签之后放置一个按钮，用户可以单击该按钮将该部分展开到完整高度。

`index.html`：

```php
<div class="about">
  <h3>About this listing</h3>
  <p class="contracted">{{ about }}</p>
  <button class="more">+ More</button>
</div>
```

这是所需的 CSS，包括一个通用按钮规则，该规则将为项目中将要添加的所有按钮提供基本样式。

`style.css`：

```php
button {
  text-align: center;
  vertical-align: middle;
  user-select: none;
  white-space: nowrap;
  cursor: pointer;
  display: inline-block;
  margin-bottom: 0;
}

.about button.more {
  background: transparent;
  border: 0;
  color: #008489;
  padding: 0;
  font-size: 17px;
  font-weight: bold;
}

.about button.more:hover, 
.about button.more:focus, 
.about button.more:active {
  text-decoration: underline;
  outline: none;
}
```

为了使其工作，我们需要一种方法，在用户单击“更多”按钮时删除`contracted`类。看起来指令是一个很好的选择！

# 类绑定

我们将采取的方法是动态绑定`contracted`类。让我们创建一个`contracted`数据属性，并将其初始值设置为`true`。

`app.js`：

```php
data: {
  ... contracted: true
}
```

与我们的样式绑定一样，我们可以将此类绑定到一个对象。在表达式中，`contracted`属性是要绑定的类的名称，`contracted`值是对同名数据属性的引用，它是一个布尔值。因此，如果`contracted`数据属性评估为`true`，那么该类将绑定到元素，如果评估为`false`，则不会绑定。

`index.html`：

```php
<p v-bind:class="{ contracted: contracted }">{{ about }}</p>
```

因此，当页面加载时，`contracted`类被绑定：

```php
<p class="contracted">...</p>
```

# 事件监听器

现在，我们希望在用户单击“更多”按钮时自动删除`contracted`类。为了完成这项工作，我们将使用`v-on`指令，该指令使用`click`参数监听 DOM 事件。

`v-on`指令的值可以是一个表达式，将`contracted`赋值为`false`。

`index.html`：

```php
<div class="about">
  <h3>About this listing</h3>
  <p v-bind:class="{ contracted: contracted }">{{ about }}</p>
  <button class="more" v-on:click="contracted = false">+ More</button>
</div>
```

# 响应性

当我们单击“更多”按钮时，`contracted`值会发生变化，Vue 将立即更新页面以反映此更改。

Vue 是如何知道做到这一点的？要回答这个问题，我们必须首先了解 getter 和 setter 的概念。

# 获取器和设置器

为 JavaScript 对象的属性分配值就像这样简单：

```php
var myObj = { prop: 'Hello'
}
```

要检索它就像这样简单：

```php
myObj.prop
```

这里没有什么诀窍。不过，我想要表达的是，我们可以通过使用 getter 和 setter 来替换对象的正常赋值/检索机制。这些是特殊函数，允许自定义逻辑来获取或设置属性的值。

当一个属性的值由另一个属性确定时，getter 和 setter 特别有用。这里有一个例子：

```php
var person = { firstName: 'Abraham', lastName: 'Lincoln',
  get fullName() {
    return this.firstName + ' ' + this.lastName;
  },
  set fullName(name) {
    var words = name.toString().split(' ');
    this.firstName = words[0] || '';
    this.lastName = words[1] || '';
  }
}
```

当我们尝试对`fullName`属性进行正常赋值/检索时，`fullName`属性的`get`和`set`函数将被调用：

```php
console.log(person.fullName); // Abraham Lincoln person.fullName = 'George Washington'; console.log(person.firstName); // George console.log(person.lastName) // Washington
```

# 响应式数据属性

Vue 的另一个初始化步骤是遍历所有数据属性并为其分配 getter 和 setter。如果您查看以下截图，您可以看到我们当前应用程序中的每个属性都添加了`get`和`set`函数：

![](img/66c022e6-0030-4d84-a3ad-c05e38b395a6.png)图 2.9 获取器和设置器

Vue 添加了这些 getter 和 setter，以使其能够在访问或修改属性时执行依赖跟踪和更改通知。因此，当`contracted`值通过`click`事件更改时，将触发其`set`方法。`set`方法将设置新值，但也将执行通知 Vue 值已更改的次要任务，并且可能需要重新渲染依赖它的页面的任何部分。

如果您想了解更多关于 Vue 的响应系统的信息，请查看文章*Vue.js 中的响应性（及其陷阱）*，网址为[`vuejsdevelopers.com/2017/03/05/vue-js-reactivity/`](https://vuejsdevelopers.com/2017/03/05/vue-js-reactivity/)。

# 隐藏更多按钮

一旦“关于”部分被展开，我们希望隐藏“更多”按钮，因为它不再需要。我们可以使用`v-if`指令与`contracted`属性一起实现这一点。

`index.html`：

```php
<button v-if="contracted" class="more" v-on:click="contracted = false"> + More
</button>
```

# 图片模态窗口

为了防止我们的页眉图片占据页面，我们对其进行了裁剪并限制了其高度。但是，如果用户想要以全貌查看图片呢？允许用户专注于单个内容项的一个很好的 UI 设计模式是*模态窗口*。

当打开时，我们的模态框将如下所示：

![](img/2d1ce89f-6747-4219-9540-615a7a3f6d17.png)图 2.10 页眉图片模态

我们的模态框将提供一个适当缩放的页眉图片视图，这样用户就可以专注于住宿的外观，而不会被页面的其他部分分散注意力。

书中稍后，我们将在模态框中插入一个图片轮播，这样用户就可以浏览整个房间图片集合！

不过，现在我们的模态框需要以下功能：

1.  通过点击页眉图片打开模态框

1.  冻结主窗口

1.  显示图片

1.  使用关闭按钮或*Escape*键关闭模态窗口

# 打开

首先，让我们添加一个布尔数据属性，表示我们模态框的打开或关闭状态。我们将其初始化为`false`。

`app.js`：

```php
data: {
  ... modalOpen: false
}
```

我们将使点击页眉图片打开模态框。我们还将在页眉图片的左下角叠加一个标有“查看照片”的按钮，以向用户发出更强烈的信号，告诉他们应该点击以显示图片。

`index.html`：

```php
<div 
  class="header-img" 
  v-bind:style="headerImageStyle" 
  v-on:click="modalOpen = true" >
  <button class="view-photos">View Photos</button>
</div>
```

请注意，通过将点击监听器放在包装`div`上，无论用户点击`button`还是`div`，都将捕获点击事件，这是由于 DOM 事件传播。

我们将为页眉图片添加一些 CSS，使光标成为*指针*，让用户知道可以点击页眉，并为页眉添加相对位置，以便在其中定位按钮。我们还将添加样式规则来设计按钮。

`style.css`：

```php
.header .header-img { ... cursor: pointer;
  position: relative;
}

button {
  border-radius: 4px;
  border: 1px solid #c4c4c4;
  text-align: center;
  vertical-align: middle;
  font-weight: bold;
  line-height: 1.43;
  user-select: none;
  white-space: nowrap;
  cursor: pointer;
  background: white;
  color: #484848;
  padding: 7px 18px;
  font-size: 14px;
  display: inline-block;
  margin-bottom: 0;
}

.header .header-img .view-photos {
  position: absolute;
  bottom: 20px;
  left: 20px;
}
```

现在让我们为模态框添加标记。我把它放在页面的其他元素之后，尽管这并不重要，因为模态框将脱离文档的常规流程。我们通过在以下 CSS 中给它一个`fixed`位置来将其从流程中移除。

`index.html`：

```php
<div id="app">
  <div class="header">...</div>
  <div class="container">...</div>
  <div id="modal" v-bind:class="{ show : modalOpen }"></div>
</div>
```

主模态`div`将充当其余模态内容的容器，同时也是将覆盖主窗口内容的背景面板。为了实现这一点，我们使用 CSS 规则将其拉伸到完全覆盖视口，给它设置`top`、`right`、`bottom`和`left`值为`0`。我们将`z-index`设置为一个较高的数字，以确保模态框叠放在页面中的任何其他元素之前。

还要注意，`display`最初设置为`none`，但我们会动态地将一个名为`show`的类绑定到模态框，使其显示为块级元素。当然，添加/删除这个类将绑定到`modalOpen`的值。

`style.css`：

```php
#modal {
  display: none;
  position: fixed;
  top: 0;
  right: 0;
  bottom: 0;
  left: 0;
  z-index: 2000;
}

#modal.show {
  display: block;
}
```

# 窗口

现在让我们为将覆盖在背景面板上的窗口添加标记。窗口将具有宽度约束，并将居中显示在视口中。

`index.html`：

```php
<div id="modal" v-bind:class="{ show : modalOpen }">
  <div class="modal-content">
    <img src="sample/header.jpg"/>
  </div>
</div>
```

`style.css`：

```php
.modal-content {
  height: 100%;
  max-width: 105vh;
  padding-top: 12vh;
  margin: 0 auto;
  position: relative;
}

.modal-content img {
  max-width: 100%;
}
```

# 禁用主窗口

当模态框打开时，我们希望防止与主窗口的任何交互，并且清楚地区分主窗口和子窗口。我们可以通过以下方式实现这一点：

+   调暗主窗口

+   防止 body 滚动

# 调暗主窗口

当模态框打开时，我们可以简单地隐藏我们的主窗口，但是最好让用户仍然能够意识到他们在应用程序流程中的位置。为了实现这一点，我们将在半透明面板下*调暗*主窗口。

我们可以通过给我们的模态面板添加不透明的黑色背景来实现这一点。

`style.css`：

```php
#modal { ... background-color: rgba(0,0,0,0.85);
}
```

# 防止 body 滚动

不过，我们有一个问题。尽管我们的模态面板是全屏的，但它仍然是`body`标签的子元素。这意味着我们仍然可以*滚动*主窗口！我们不希望用户在模态框打开时以任何方式与主窗口进行交互，因此我们必须禁用`body`上的滚动。

诀窍是向`body`标签添加 CSS `overflow`属性，并将其设置为`hidden`。这样做的效果是裁剪任何*溢出*（即，当前不在视图中的页面的部分），其余内容将变为不可见。

我们需要动态添加和删除这个 CSS 规则，因为当模态框关闭时，我们显然希望能够滚动页面。因此，让我们创建一个名为`modal-open`的类，当模态框打开时，我们可以将其应用于`body`标签。

`style.css`：

```php
body.modal-open {
  overflow: hidden;
 position: fixed;
}
```

我们可以使用`v-bind:class`来添加/删除这个类，对吗？不幸的是，不行。请记住，Vue 只对它挂载的元素有控制权：

```php
<body>
  <div id="app"> 
    <!--This is where Vue has dominion and can modify the page freely-->
  </div>
  <!--Vue is unable to change this part of the page or any ancestors-->
</body>
```

如果我们向`body`标签添加指令，它将*不会*被 Vue 看到。

# Vue 的挂载元素

如果我们只是在`body`标签上挂载 Vue，那么这样做能解决我们的问题吗？例如：

```php
new Vue({ el: 'body' 
});
```

Vue 不允许这样做，如果您尝试这样做，您将收到此错误：不要将 Vue 挂载到<html>或<body> - 而是挂载到普通元素。

请记住，Vue 必须编译模板并替换挂载节点。如果您的脚本标签作为挂载节点的子元素存在（通常是`body`），或者如果您的用户有修改文档的浏览器插件（很多都有），那么当 Vue 替换该节点时，页面可能会出现各种问题。

如果您定义了自己的具有唯一 ID 的根元素，则不应该出现这种冲突。

# 观察者

那么，如果`body`不在 Vue 的控制范围之内，我们如何向`body`添加/删除类？我们将不得不用浏览器的 Web API 以老式的方式来做。当模态框打开或关闭时，我们需要运行以下语句：

```php
// Modal opens document.body.classList.add('modal-open');

// Modal closes document.body.classList.remove('modal-closed');
```

正如讨论的那样，Vue 为每个数据属性添加了响应式的 getter 和 setter，以便在数据更改时知道如何适当地更新 DOM。Vue 还允许您编写自定义逻辑，以通过名为*watchers*的功能钩入响应式数据更改。

要添加观察者，首先向 Vue 实例添加`watch`属性。将一个对象分配给这个属性，其中每个属性都有一个声明的数据属性的名称，每个值都是一个函数。该函数有两个参数：旧值和新值。

每当数据属性更改时，Vue 将触发任何声明的观察者方法：

```php
var app = new Vue({ el: '#app' data: { message: 'Hello world'
  }, watch: { message: function(newVal, oldVal) { console.log(oldVal, ', ', newVal);
    }
  }
});

setTimeout(function() { app.message = 'Goodbye world';
  // Output: "Hello world, Goodbye world";
}, 2000);
```

Vue 不能为我们更新`body`标签，但它可以触发将要更新的自定义逻辑。让我们使用一个观察者来在我们的模态框打开和关闭时更新`body`标签。

`app.js`：

```php
var app = new Vue({ data: { ... }, watch: { modalOpen: function() {
      var className = 'modal-open';
      if (this.modalOpen) { document.body.classList.add(className);
      } else { document.body.classList.remove(className);
      }
    }
  }
});
```

现在当您尝试滚动页面时，您会发现它不会动！

# 关闭

用户需要一种关闭他们的模态框并返回到主窗口的方法。我们将在右上角叠加一个按钮，当点击时，会评估一个表达式来将`modalOpen`设置为`false`。我们包装`div`上的`show`类将随之被移除，这意味着`display` CSS 属性将返回到`none`，从而将模态框从页面中移除。

`index.html`：

```php
<div id="modal" v-bind:class="{ show : modalOpen }">
  <button v-on:click="modalOpen = false" class="modal-close"> &times; </button>
  <div class="modal-content">
    <img src="sample/header.jpg"/>
  </div>
</div>
```

`style.css`：

```php
.modal-close {
  position: absolute;
  right: 0;
  top: 0;
  padding: 0px 28px 8px;
  font-size: 4em;
  width: auto;
  height: auto;
  background: transparent;
  border: 0;
  outline: none;
  color: #ffffff;
  z-index: 1000;
  font-weight: 100;
  line-height: 1;
}
```

# Escape 键

为我们的模态框添加一个关闭按钮很方便，但大多数人关闭窗口的本能动作是按下*Escape*键。

`v-on`是 Vue 监听事件的机制，似乎是这项工作的一个很好的选择。添加`keyup`参数将在此输入聚焦时按下*任何*键后触发处理程序回调：

```php
<input v-on:keyup="handler">
```

# 事件修饰符

Vue 通过为`v-on`指令提供*修饰符*来轻松监听*特定*键。修饰符是由点(`.`)表示的后缀，例如：

```php
<input v-on:keyup.enter="handler">
```

正如您可能猜到的那样，`.enter`修饰符告诉 Vue 仅在事件由*Enter*键触发时调用处理程序。修饰符使您无需记住特定的键码，还使您的模板逻辑更加明显。Vue 提供了各种其他键修饰符，包括：

+   `tab`

+   `delete`

+   `空间`

+   `esc`

考虑到这一点，似乎我们可以使用这个指令关闭我们的模态框：

```php
v-on:keyup.esc="modalOpen = false"
```

但是我们应该将这个指令附加到哪个标签呢？不幸的是，除非有一个输入被聚焦，否则键事件将从`body`元素分派，而我们知道这是 Vue 无法控制的！

为了处理这个事件，我们将再次求助于 Web API。

`app.js`：

```php
var app = new Vue({ 
  ... 
}); document.addEventListener(</span>'keyup', function(evt) {
  if (evt.keyCode === 27 && app.modalOpen) { app.modalOpen = false;
  }
});
```

这个方法可以工作，但有一个警告（在下一节中讨论）。但是 Vue 可以帮助我们使它完美。

# 生命周期钩子

当您的主脚本运行并设置了 Vue 的实例时，它会经历一系列的初始化步骤。正如我们之前所说的，Vue 将遍历您的数据对象并使它们具有反应性，同时编译模板并挂载到 DOM 上。在生命周期的后期，Vue 还将经历更新步骤，然后是拆卸步骤。

这是从[`vuejs.org`](http://vuejs.org)获取的生命周期实例图。其中许多步骤涉及到我们尚未涵盖的概念，但您应该能够理解大意：

![](img/9f308e86-bbbe-489c-9f93-06abe2675081.png)图 2.11. Vue.js 生命周期图

Vue 允许您通过*生命周期钩子*在这些不同的步骤执行自定义逻辑，这些钩子是在配置对象中定义的回调函数。

例如，这里我们利用了`beforeCreate`和`created`钩子：

```php
new Vue({ data: { message: 'Hello'
  }, beforeCreate: function() { console.log('beforeCreate message: ' + this.message);
    // "beforeCreate message: undefined"
  }, created: function() { console.log('created: '+ this.message);
    // "created message: Hello"
  },
});
```

在`beforeCreate`钩子被调用之后但在`created`钩子被调用之前，Vue 将数据属性别名为上下文对象，因此在前者中`this.message`是`undefined`。

我之前提到的关于*Escape*键监听器的警告是：虽然不太可能，但如果在按下*Escape*键并且我们的回调在 Vue 代理数据属性之前被调用，`app.modalOpen`将是`undefined`而不是`true`，因此我们的`if`语句将无法像我们期望的那样控制流程。

为了克服这个问题，我们可以在`created`生命周期钩子中设置监听器，该监听器将在 Vue 代理数据属性之后调用。这给了我们一个保证，当回调运行时，`modalOpen`将被定义。

`app.js`：

```php
function escapeKeyListener(evt) {
  if (evt.keyCode === 27 && app.modalOpen) { app.modalOpen = false;
  }
}

var app = new Vue({ data: { ... }, watch: { ... }, created: function() { document.addEventListener('keyup', escapeKeyListener);
  }
});
```

# 方法

Vue 配置对象还有一个*方法*部分。方法不是响应式的，因此您可以在 Vue 配置之外定义它们，而在功能上没有任何区别，但 Vue 方法的优势在于它们作为上下文传递了 Vue 实例，因此可以轻松访问您的其他属性和方法。

让我们重构我们的`escapeKeyListener`成为一个`Vue`实例方法。

`app.js`：

```php
var app = new Vue({ data: { ... }, methods: { escapeKeyListener: function(evt) {
      if (evt.keyCode === 27 && this.modalOpen) {
        this.modalOpen = false;
      }
    }
  }, watch: { ... },
 created: function() { document.addEventListener('keyup', this.escapeKeyListener);
  }
});
```

# 代理属性

你可能已经注意到我们的`escapeKeyListener`方法可以引用`this.modalOpen`。难道不应该是`this.methods.modalOpen`吗？

当 Vue 实例被构建时，它会将任何数据属性、方法和计算属性代理到实例对象。这意味着在任何方法中，你可以引用`this.myDataProperty`、`this.myMethod`等，而不是`this.data.myDataProperty`或`this.methods.myMethod`，正如你可能会假设的那样：

```php
var app = new Vue({ data: { myDataProperty: 'Hello'
  }, methods: { myMethod: function() {
      return this.myDataProperty + ' World';
    }
  }
}); console.log(app.myMethod());
// Output: 'Hello World' 
```

你可以通过在浏览器控制台中打印 Vue 对象来查看这些代理属性：

![](img/d6771753-2628-41d4-809f-4e0f2ab6381e.png)图 2.12。我们应用的 Vue 实例

现在文本插值的简单性可能更有意义，它们具有 Vue 实例的上下文，并且由于代理属性的存在，可以像`{{ myDataProperty }}`一样被引用。

然而，虽然代理到根使语法更简洁，但一个后果是你不能用相同的名称命名你的数据属性、方法或计算属性！

# 移除监听器

为了避免任何内存泄漏，当 Vue 实例被销毁时，我们还应该使用`removeEventListener`来摆脱监听器。我们可以使用`destroy`钩子，并调用我们的`escapeKeyListener`方法来实现这个目的。

`app.js`：

```php
new Vue({ data: { ... }, methods: { ... }, watch: { ... }, created: function() { ... }, destroyed: function () { document.removeEventListener('keyup', this.escapeKeyListener);
  }
});
```

# 摘要

在本章中，我们熟悉了 Vue 的基本特性，包括安装和基本配置、数据绑定、文本插值、指令、方法、观察者和生命周期钩子。我们还了解了 Vue 的内部工作原理，包括响应系统。

然后，我们利用这些知识来设置一个基本的 Vue 项目，并为 Vuebnb 原型创建页面内容，包括文本、信息列表、页眉图像，以及 UI 小部件，如“显示更多”按钮和模态窗口。

在下一章中，我们将暂时离开 Vue，同时使用 Laravel 为 Vuebnb 设置后端。
