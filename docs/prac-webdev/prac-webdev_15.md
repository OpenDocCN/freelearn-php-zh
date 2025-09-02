# 附录 A. Bootstrap – Foundation 的替代方案

在撰写这本书的时候，**Bootstrap**是 GitHub 上最受欢迎的项目。许多网站和 Web 应用程序都是使用 Bootstrap 构建的。关于这个主题已经出版了多本书籍，而且当 Web 开发者申请工作时，通常需要 Bootstrap 的使用经验。这就是为什么我决定在这本书中包含一个关于 Bootstrap 的附录。

我将向您展示如何使用 Foundation 完成的事情，但将以 Bootstrap 的方式。

# Bootstrap 组件

您可以从[`getbootstrap.com`](http://getbootstrap.com)下载 Bootstrap，有多种风味。最小的下载是一个简单的发行版，包含您部署系统所需的所有组件。源发行版在子文件夹中有相同的内容，但也包含**文档**、**less**、JavaScript 源代码（按组件分解）和示例，这些都是很好的学习材料。

在这里，我们将关注最小分布。在我的系统中，我将它们全部放在了`bootstrap`文件夹中，以便更容易切换版本，并在网站上保持 Foundation 和 Bootstrap 的风味。

### 注意

注意 Bootstrap 依赖于 jQuery。与 Foundation 不同，Bootstrap 的发行版不包含 jQuery，因此您需要从其他地方获取您的副本（或者当然可以使用随 Foundation 提供的副本）。

以下是您通过 Bootstrap 下载将获得的内容：

+   **css**：这是一个包含`bootstrap.css`和`bootstrap.min.css`的文件夹，Bootstrap 的样式表。在您的网站`<head>`部分包含其中一个。此外，还包括您自己的样式表。我们建议您不要修改`bootstrap.css`。

    该文件夹还包含可选的*Bootstrap 主题*的样式表。

+   **js/bootstrap.min.js**和**js/bootstrap.js**：这包含所有 Bootstrap JavaScript。将此放在您的`<body>`标签的末尾之前。

+   **字体**：这是一个包含图标字体文件夹。我们坚持使用令人惊叹的 font-awesome 字体。

# Bootstrap 网格系统

与 Foundation 一样，Bootstrap 自带一个网格系统，默认情况下，将工作屏幕的实际空间分为 12 列。使用类，您可以指定屏幕上每个块需要多宽的列数。有不同大小的类；这是 Bootstrap 和 Foundation 使响应式设计变得简单且几乎透明的做法。

Bootstrap 和 Foundation 使用不同的**断点**。在 Foundation 中，小号意味着小于 640 像素，中等是 641 像素到 1024 像素，大号是 1025 像素及以上。有两个可选的 XL 和 XXL 大小，就像 T 恤一样，断点分别是 1440 像素和 1920 像素。最后两个被注释掉了；如果您想使用它们，需要激活它们。

Bootstrap 有四种大小：`xs`、`sm`、`md`和`lg`。就像两个 T 恤制造商在小号上可能有不同的大小一样，这两个框架也有不同的大小，断点分别是 768 像素、992 像素和 1200 像素。

### 注意

Bootstrap 中似乎没有与阻塞网格等效的类。

这是我们在 第十三章 中向您展示的相同示例，*Foundation - 一个响应式 CSS/JavaScript 框架*，但这次是以 Bootstrap 的方式实现的：

```php
<div class="container">
<div class="row">
<div class="col-xs-12 col-sm-6 col-md-4 col-lg-4">
<h3>First block title</h3>
<p> First block text</p>
</div>
<div class="col-xs-12 col-sm-6 col-md-4 col-lg-4">
<h3>Second block title</h3>
<p>Second block text</p>
</div>
<div class="col-xs-12 col-sm-6 col-md-4 col-lg-4">
<h3>Third block title</h3>
<p>Third block text</p>
</div>
</div>
</div>
```

注意，我们用具有 `container` 类的 `<div>` 包裹了整个行 enchilada。这就是 Bootstrap 处理事情的方式。你不需要添加 `column` 类，就像在 Foundation 中那样。

## 可见性类

Bootstrap 中有许多可见性类，但比 Foundation 少，用于隐藏或显示网格的部分，如下所示：

+   `hidden-xs`

+   `hidden-sm`

+   `hidden-md`

+   `hidden-lg`

+   `visible-xs`

+   `visible-sm`

+   `visible-md`

+   `visible-lg`

## 按钮

在我们的 Foundation 示例中，我们默默地使用了与 **按钮** 相关的类，定义了形状、颜色和大小。主要的类是——惊喜，惊喜——`button`。Bootstrap 有一个等效的类集，我们将在以下示例中使用。主要的 `button` 类是 `btn`。

## 其他 UI 元素

Bootstrap 文档齐全；也有很多书籍可供阅读和学习。在本章的剩余部分，我们将回顾与 Foundation 相同的 UI 功能，如果适用的话。

### 缩略图

在 Bootstrap 中，有一个简单的类叫做 `thumbnail`，你可以用它来生成带有或没有标题的格式良好的响应式缩略图。以下是一个代码片段的例子：

```php
<a class="thumbnail" href="largeimgs/photo.jpg">
<img src="img/photo.jpg"></img>
<div><h6>Caption</h6></div>
</a>
```

这为您的缩略图和标题添加了良好的样式，但一旦您点击它，较大的图片将以浏览器决定的方式显示。

### 下拉菜单

这是之前相同的下拉菜单示例，但这次是用 Bootstrap 实现的。我相信一个打字错误已经进入了发布，现在无法纠正。作为一个歌剧爱好者，我在 Bootstrap 中找不到任何咏叹调；我认为他们只是想说是区域：

```php
<div class="dropdown">
<button  class="btn btn-primary btn-lg" type="button"  id="calpeople" aria-expanded="false" aria-haspopup="true"  data-toggle="dropdown">California people</button>
     <ul class="dropdown-menu" role="menu" aria-labelledby="calpeople">
      <li>Ansel Adams</li>
      <li>John Muir</li>
      <li>Arnold Schwarzenegger</li>
      </ul>
      </div>
```

### 模态框 – Bootstrap 的弹出窗口

弹出窗口可以在网站的任何位置使用。Bootstrap 提供了他们所说的 **模态框**。这与在 Foundation 中实现的方式非常相似。以下是之前示例的 Bootstrapped 版本。注意关闭弹出窗口的图标是如何处理的：

```php
<div class="row">
<button type="button" class="btn btn-primary btn-md"
data-toggle="modal" data-target="#revealid">
      Click here
 </button>
      <div class="modal fade" id="revealid"  role="dialog" >
      <div class="modal-dialog modal-sm">
      <div class="modal-content">
      <button type="button" class="close" data-dismiss="modal" aria-label="Close"><span aria-hidden="true">&times;</span></button>
      <p>Text of popup</p>
      </div></div></div></div>
```

注意，`modal-sm` 类用于确定我们想要的模态框的大小。`modal-content` 部分可以进一步分为三个 `<div>` 元素：**modal-header**、**modal-body** 和 **modal-footer**。

## 结合下拉菜单和模态框

现在，我们将结合下拉菜单和模态框，以展示可以作为相册一部分的内容，如下所示：

```php
<div class="row">
<h3> Modal with drop-down for details</h3>
<div class="col-xs-3 col-md-2">
<div class="thumbnail" data-target="#tiogalakepf" data-toggle="modal">
<img src="img/tiogalakesmall.jpg" alt="Tioga Lake">
    <div class="caption">
    <h6>Tioga Lake</h6>
    </div>
</div></div>
<div id="tiogalakepf"  class="modal"  >
    <div class="modal-dialog modal-md">
    <div class="modal-content">
   <div class="modal-body thumbnail">
    <button type="button" class="close" data-dismiss="modal" aria-label="Close">
    <span aria-hidden="true">&times;</span></      button>
    <img  src="img/tiogalake.jpg">
    <h3 style="text-align:center">Tioga Lake in april of 1997</h3>
    <a class="btn btn-sm btn-primary" data-toggle="dropdown" id="tiogadrop" >Details</a>
<ul class="dropdown-menu" role="menu" aria-labelledby="tiogadrop">
  <li>Hasselblad</li>
   <li> Fuji Velvia</li>
   <li> 40mm CF lens </li>
   <li> f16 </li>
   </ul></div>
</div></div></div></div>
```

### 折叠 – Bootstrap 的手风琴

Bootstrap 的 `collapse` 功能提供了与 Foundation 和其他框架中的手风琴相同的功能。这是我们重新设计示例的方式。几乎没有样式；因此，我们建议如果您使用 Bootstrap，请将其与 `panel` 小部件结合使用：

```php
<div class="col-xs-12">
<h3>California People</h3>
<a  href="#ansel" data-toggle="collapse">
<h5><i class="fa fa-caret-right green" ></i>&nbsp;Ansel Adams</h5></a>
<div  class="collapse"  id="ansel">
<p>I think of Ansel Adams as the best landscape photographer ever. His black and white photographs of Yosemite National Park and New Mexico are legendary </p>
</div>
<a  href="#muir" data-toggle="collapse" >
<h5><i class="fa fa-caret-right green" ></i>&nbsp;John Muir</h5></a>
<div  class="collapse"  id="muir">
<p>I think of John Muir as the father of Yosemite National Park </p>
</div></div>
```

# 导航

Bootstrap 有一个你可能想要了解的**导航栏**组件。如果你喜欢 Foundation 顶部栏的外观，我鼓励你使用可选的*Bootstrap 主题*。

# 摘要

在本附录中，我们描述了 Bootstrap CSS / JavaScript 框架。与 Foundation 类似，它允许你创建一个以移动端为先、响应式设计的网站或应用程序，而无需编写任何响应式部分的代码。我们只需根据我们的需求自定义样式——当然是在一个单独的样式表中。
