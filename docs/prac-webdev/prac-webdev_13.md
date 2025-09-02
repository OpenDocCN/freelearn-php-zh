# 第十三章。Foundation——一个响应式 CSS/JavaScript 框架

我通常将**Zurb Foundation**和**Twitter Bootstrap**描述给人们，就像“我一直想写但从未找到时间去做的事情”。

可以称它们为框架也可以不称，但我建议你检查一下这两个网站，了解它们如何称呼自己。我使用过它们，比较过它们，并决定选择**Foundation**。这取决于个人品味。我总是喜欢接受更大的挑战。我更喜欢学习芬兰语而不是瑞典语。你可能觉得 Bootstrap 更容易使用，因为它可能包含更多的组件，但在我看来，识别一个网站是否是用 Bootstrap 开发的比用 Foundation 开发的要容易得多。我已经在附录中包含了一个 Bootstrap 的概述。

# 我们的响应式工具包——Foundation

在上一章中，你学习了我们需要开发一个以**移动优先**、渐进增强、响应式设计为主的网页应用的所有拼图碎片。移动优先是一种思维方式；对于渐进增强部分，我们使用了`enhance.js`和`modernizr.js`。因此，我们仍然需要了解如何通过编写灵活的网格、创建媒体查询、支持灵活的图片以及一些其他操作来完成响应式部分。

这是 Foundation 已经为我们完成的部分。此外，它确实有一些酷的用户界面（UI）组件。其中之一是*off-canvas*，这是一个在移动设备和平板电脑上使用的出色功能，自从我使用它以来，我在每个主要网站或应用上都能认出它，包括 Facebook。查看优秀的 Foundation 文档，以获取完整的功能列表。在本章中，我们将描述我认为特别有用的那些功能。

# Foundation 组件

下载 Foundation 后[`foundation.zurb.com/`](http://foundation.zurb.com/)（我推荐选择完整版），你会发现它包含 CSS 组件和 JavaScript 组件。我将它们都放在了一个名为`foundation5`的文件夹中，以便更容易切换版本，甚至维护一个 Bootstrap 风格的网站。

以下是下载将获得的内容：

+   **index.html**：这是一个展示 Foundation 最常见功能的网页。

+   **css**：这是一个包含`foundation.css`和`foundation.min.css`的文件夹，是 Foundation 样式表。在你的网站`<head>`部分包含其中一个。此外，包含你自己的样式表。我们建议你不要修改`foundation.css`。否则，如果出现新版本，你最终会覆盖你的代码。

+   **js/foundation.min.js**：这是所有的 Foundation JavaScript。将其放在你的`<body>`标签的末尾之前，jQuery 之后。另外，不要修改此文件。

+   **js/foundation**：这是一个包含 Foundation 功能的单独 JavaScript 源文件的文件夹，以防你想了解它们是如何工作的。不需要在你的网页中包含这个文件夹。

+   **js/vendor**：这是一个包含更多有用 JavaScript 库的文件夹，例如 jQuery 和**Modernizr**。

基础框架使用 Modernizr，所以你很高兴我们已经知道这是什么了吧？

在你网站的`<head>`部分引用`modernizr.js`，以便在页面加载之前执行所有测试。将`jQuery.js`放在`<body>`的末尾之前，并在`foundation.min.js`之前。然后，添加以下行，这将启动基础框架：

```php
<script>
      $(document).foundation();
</script>
```

# 网格系统

基础框架自带一个网格系统，默认情况下，将你的工作屏幕区域分为 12 列。使用类，你可以为屏幕上的每个块指定你希望它有多宽。有不同的类用于不同的尺寸；这是基础框架使响应式设计变得简单的方法。

想象一下你希望你的网站在各种屏幕和设备上看起来都很棒。让我们先看看较大的屏幕，暂时称它们为画布。想象两条垂直线，大约相隔 1024 像素，位于画布中心。这些将成为我们工作区域的垂直边界。当然，一旦你减小视口大小，或者如果你正在使用平板或手机，你的工作区域将是屏幕的全宽。

现在，为了构建我们网页的布局，我们将将其分为水平行，在其中，我们将放置响应式的内容块（`<div>`）。对于这些，我们可以使用类来指定我们正在处理哪种尺寸的屏幕（可以想象成哪种类型的 T 恤），以及我们希望填充多少列。看看下面的例子：

```php
<div class="row">
<div class="small-12 medium-6 large-4 columns">
<h3>First block title</h3>
<p> First block text</p>
</div>
<div class="small-12 medium-6 large-4 columns">
<h3>Second block title</h3>
<p>Second block text</p>
</div>
<div class="small-12 medium-6 large-4 columns">
<h3>Third block title</h3>
<p>Third block text</p>
</div>
</div>
```

因此，这是我们第一个“行”的内容，包含三个响应式块。现在，我们首先构建移动端，所以第一个类我们使用的是表示我们希望这个块在移动端（比如我们的 T 恤）上有多宽的列数。我们选择最大值：12。如果你没有为中等和大型尺寸指定任何内容，它们将隐式地继承相同的值。始终包含`columns`类。

你可能已经猜到了这里会发生什么。当你在一个小屏幕上显示这些内容时，这三个块中的每一个都将有 12 列宽，这是我们能够达到的最大宽度，因此它们将堆叠在一起。当我们切换到中等屏幕，比如将平板电脑从纵向转为横向时，我们将有一行两个块，另一个块在其下方。在大屏幕上，它们将并排排列。

## 类 end

如果你一行中没有足够的项来填充所有列，最后一个项将被放置在右侧边缘。如果你想覆盖这一点，你可以为这个添加一个名为`end`的类，它将出现在你期望的位置。

## 可见性类

这里有一系列适合的类，我总是在文档中找不到它们，因为它们在*网格*部分之前就被描述了：**可见性类**。类似于确定某个东西想要有多宽的列宽，你可能想要根据设备决定是否显示它。最流行的例子可能是你在小屏幕上看到的三个条形菜单图标，它取代了“正常”尺寸屏幕上的水平导航。你可以选择为 T 恤尺寸或更大尺寸隐藏或显示。名字应该能说明一切。这也表明我们在 T 恤店有更多尺寸。

更有用的是，你可以使用这些类来根据屏幕方向和是否为触摸屏设备来决定是否显示某些内容。这也是你不应该忘记包含`modernizr.js`的原因之一。以下是列表：

| `show-for-touch` | `show-for-medium-up` | `hide-for-small-only` |
| --- | --- | --- |
| `hide-for-touch` | `show-for-large-only` | `hide-for-medium-only` |
| `show-for-landscape` | `show-for-large-up` | `hide-for-medium-up` |
| `hide-for-landscape` | `show-for-xlarge-only` | `hide-for-large-only` |
| `show-for-small-only` | `show-for-xlarge-up` | `hide-for-large-up` |
| `show-for-medium-only` | `show-for-xxlarge-up` | `hide-for-xlarge-only` |
| `hide-for-xlarge-up` | `hide-for-xxlarge-up` |   |

## 块状网格系统

这让我说出了一些让人困惑的智慧之言，直到你看到例子：拥有相同大小的东西并不等同于东西的大小是一样的。

属于`块状网格`系统的类允许你确保无论屏幕大小如何，你的项目块都是均匀分布的。你可以通过指定一行中想要的`项目`数量来实现这一点，而不是指定一个项目想要的列宽。

因此，在较小的屏幕上，你的项目会变得更小，所以它们不再是相同的大小，但一行中的所有项目将具有相同的大小，所以它们都有相同的大小。明白了吗？这是一个在简单相册或博客文章中使用的好功能。

你可以通过将类（们）分配给一个`<ul>`元素来使用块状网格，你的项目是`<li>`元素。一个典型的类名可能是`size-block-grid-number`。例如：

```php
<ul class="small-block-grid-2 medium-block-grid-3 large-block-grid-4">
<li><!-- first item here --></li>
....
</ul>
```

## 有用的 UI 元素

如前所述，Foundation 有很好的文档，如果你想要阅读和学习更多，还有书籍可供选择。在本章的剩余部分，我们将介绍一些我认为非常有用的 UI 功能，并附带一些结合它们的例子。

之后，我们将以 Foundation 的方式完成导航章节，**画布上**和**画布外**。

### 缩略图 – 用于简单相册

Foundation 正在变化，所以在撰写这本书的时候，有几个 UI 功能用于制作图片滑块或其他与照片相关的酷炫功能，这些功能将被大幅修改或替换。

然而，每个人都需要他们的图片缩略图作为设计的一部分。在 Foundation 中，有一个简单的类名为`th`，您可以使用它来生成带有或没有标题的格式良好的响应式缩略图。例如：

```php
<a class="th" href="/photo.jpg">
<img src="img/photo.jpg"></img> <div><h6>Caption</h6></div>
</a>
```

这将为您的缩略图像和标题周围添加漂亮的样式，但一旦您点击它，较大的图像将以浏览器决定的方式显示。

### 揭示模态 – 您更好的弹出窗口

如果我们的较大图像能够在一个漂亮的弹出窗口中显示，那就更好了。弹出窗口可以在网站的任何地方使用。Foundation 提供了他们所说的**揭示模态**。它们非常简单易用。

这是您在网站上放置以提供弹出窗口访问权限的区域。在示例中，它是一个简单的`<div>`，甚至不是一个`<a>`标签。您给元素一个`data-reveal-id`属性，其值为您想要揭示的元素的 ID。

接下来，有一个具有该 ID 的元素，其中包含您想要在弹出窗口中显示的内容。注意其末尾的带有`close-reveal-modal`类的`<a>`标签。当点击时，这将触发弹出窗口消失。如果您忘记这部分，用户将无法使弹出窗口消失：

```php
<div data-reveal-id="revealid">Click here</div>
<div id="revealid"  class="reveal-modal medium" data-reveal >
<p>Text of popup</p><a class="close-reveal-modal">×</a></div>
```

### 下拉菜单

也许您想要一个下拉菜单（或向上下拉或向左下拉）而不是弹出窗口。在这个例子中，我们介绍了一个 Foundation 风格的`button`。我们还将返回到 T 恤店：按钮和实际的下拉菜单都可以给予一个大小类。当您包含`content`类时，下拉菜单将获得一些漂亮的填充。还有一个选项，您可以使用它来使下拉菜单在悬停时显示：

```php
<a  class="button tiny" data-dropdown="calpeople"
data-options="is_hover:true">California people</a>
<ul id="calpeople" class="small f-dropdown content"
data-dropdown-content>
      <li>Ansel Adams</li>
      <li>John Muir</li>
      <li>Arnold Schwarzenegger</li>
</ul>
```

### 示例 – 一个简单的照片画廊

让我们现在结合一个块网格、缩略图、揭示模态和下拉菜单来制作一个漂亮的响应式照片画廊。我们将使用模态来显示较大的图像，并使用下拉菜单来显示照片的技术细节。

这是第一张照片的代码片段：

```php
<div class="row">
<ul class="small-block-grid-3 medium-block-grid-4large-blockgrid-5" >
<li>
<div class="th" data-reveal-id="tiogalakepf" data-reveal>
<img src="img/tiogalakesmall.jpg" alt="Tioga Lake">
<div class="caption"><h6>Tioga Lake</h6></div>
</div>
<div id="tiogalakepf"  class="reveal-modal medium portfolio" data-reveal >
<img src="img/tiogalake.jpg"></img>
<p>&copy; 1997-2015 Paul Wellens</p>
<h3>Tioga Lake in april of 1997</h3>
<a class="button small" data-dropdown="tiogadrop" data-options="align:top">Details</a>
<ul id="tiogadrop" class="f-dropdown content" data-dropdown-content>
     <li>Hasselblad</li>
     <li> Fuji Velvia</li>
     <li> 40mm CF lens </li>
     <li> f16 </li>
</ul>
<a class="close-reveal-modal">×</a>
</div>
</li>
</ul>
</div>
```

### 手风琴

**手风琴**在您有信息被划分为逻辑部分，并且想要展开某些内容，然后再折叠回原状时非常有用。我发现它是一个在在线简历中使用的极好工具，例如，用于工作历史部分或常见问题解答。我已经使用 jQuery UI 使用它多年了。自从我发现 Foundation 以来，我就开始使用他们的手风琴。

它的设置非常简单；您可以使用`<ul>`元素或`<dl>`元素来创建您的手风琴。在我们的示例中，我们使用`<dl>`。

```php
<div class="small-12 columns">
<h3>California People</h3>
<dl class="accordion  data-accordion=">
<dd class="accordion-navigation"><a  href="#ansel">
<h5><i class="fa fa-caret-right green" ></i>Ansel Adams</h5></a>
<div  class="content"  id="ansel">
<p>I think of Ansel Adams as the best landscape photographer ever. His black and white photographs of Yosemite National Park and New Mexico are legendary </p>
</div>
</dd>
...
</dl>
```

### 极佳的 Font awesome

在这个例子中，我们不仅介绍了手风琴，还介绍了一个来自外部 Foundation 的激动人心的特性：Font awesome ([www.fontawesome.io](http://www.fontawesome.io)). 那些需要为屏幕上的每一张图片准备`.gif`、`.png`或`.jpg`文件，并且为每种大小都准备一个单独文件的日子已经一去不复返了。

您现在可以在代码中使用图标，就像它们是具有特定类的 HTML 元素一样。在底层，它们是矢量图像，因此可以很好地缩放，并且由于它们是具有类的 HTML 元素，您可以使用 CSS 来样式化它们。有超过 500 个图标可供选择。Font awesome 使用`<i>`标签，已弃用其原始用途。因此，通过添加两个类，`fa`和`fa-caret-right`，您的`<i>`变成一个酷炫的右箭头，有助于使手风琴更加直观。

在示例中，我们给它添加了一个额外的类，`green`，并在我们的 CSS 文件中匹配颜色。如果我们点击一个项目以展开手风琴，我们希望它变成一个蓝色的向下箭头。我们在 JavaScript 中这样做，如下所示：

```php
$("dl.accordion").on("click", "a", function(e){
var faicons = $(this).parent().parent().find('i.fa');
faicons.removeClass("blue").addClass("green").removeClass("fa- caret-down").addClass("fa-caret-right");
if ($(this).parent().hasClass("active")) {
  var faicon = $(this).parent().find('i.fa');
  faicon.removeClass("blue").addClass("green").removeClass("fa- caret-down").addClass("fa-caret-right");
  }
});
```

基础框架使用一个名为`active`的类来指示手风琴的一部分被展开。因此，我们在改变箭头的形状和颜色之前会检查这个类。当然，我们事先并不知道我们在哪里，所以我们首先将所有箭头都改为`right`和`green`。

### 平衡器 – 使用两个<div>制作最困难的事情变得简单

我发现很难做到的一点是确保相邻的两个内容块始终具有相同的高度。当然，您可以以像素为单位给它们设置相同的高度，但如果内容的一部分超过了您指定的长度，或者如果这个长度高于设备本身呢？最好不指定固定的高度，让您的`<div>`随着内容增长，但这样如何确保它们保持对齐？

**平衡器**在基础框架中会为你处理这些。这可能听起来像是另一个阿诺德·施瓦辛格电影的标题，但它同样强大。它使用 HTML5 数据属性来完成工作。只需将`data-equalizer`添加到父容器中，将`data-equalizer-watch`添加到所有希望具有相同高度的容器中。

# 导航

基础框架还有许多其他功能，您可能想要查看。我的意图是突出那些让我如此着迷以至于我决定使用该框架的功能。其中之一是**orbit**，一个出色的照片滑块，但现在已被弃用。基础团队推荐了一个相当酷的替代方案，但无论如何，这些事情在教科书中很难解释。我们将在附录中提供整个内容，该附录仅在网上提供。

我们以基础框架提供的两个主要导航组件来结束这一章：一个用于**画布上菜单栏**，另一个用于**画布外菜单**。

## 顶部栏 – 不只是常规菜单栏

可能是菜单栏的顶层，基础框架的顶部栏是一个非常复杂的组件，它为您提供了创建易于导航的水平菜单栏所需的一切。

您的顶层货架分为三个部分：标题区域，*左侧*和*右侧*菜单项列表。以下是一个示例。为了简洁起见，我们没有在`<a>`标签内包含任何`href`属性：

```php
<div class="row">
  <nav class="top-bar" data-topbar role="navigation">
  <ul class="title-area">
   <li class="name">
   <h1><a href="#">My Site</a></h1>
    </li>
<!-- Here is the spot to add magic -->
  </ul>

  <section class="top-bar-section">
    <!-- Right Nav Section -->
    <ul class="right">
      <li><a>Overview</a></li>
      <li class="has-dropdown">
        <a>East California</a>
        <ul class="dropdown">
          <li><a>High Sierra</a></li>
          <li><a>Mono Lake</a></li>
          <li><a>June Lake</a></li>
          <li><a>Death Valley</a></li>
        </ul>
      </li>
      <li class="has-dropdown">
        <a>California Coast</a>
        <ul class="dropdown">
          <li><a>Venice Beach</a></li>
          <li><a>Big Sur</a></li>
          <li><a>San Simeon</a></li>
          <li><a>Point Reyes</a></li>
        </ul>
  </li>
  </ul>
  <!-- Left Nav Section -->
  <ul class="left">
  <li><a>About</a></li>
  </ul>
    </section>
  </nav>
</div>
```

如您所见，这相对简单直接。在经典的网站开发中，通常菜单是 `<ul>` 元素，菜单项是 `<li>` 元素。如果您想添加子菜单，可以使用名为 `dropdown` 的类，并在其下方添加另一个 `<ul>`。左右顺序并不重要。这非常好，因为这意味着您可以为基本版本使用相同的 HTML。只需将静态 HTML 文件作为 `href` 属性的值添加，然后使用您自己的酷炫 JavaScript 魔法制作现代、响应式的版本。

Foundation 在顶部栏周围创建了一些漂亮的样式，有几种深灰色（不是五十种），当然，您可以在自己的 CSS 文件中更改。

### 添加更多魔法

这就是魔法。通过简单地在标题区域添加一个 `<li>` 元素，您就可以让 Foundation 为您创建一个适用于小屏幕的完整替代菜单。只需查看这个示例。按照指示添加到示例中，看看当您将查看区域缩小到非常小的时候会发生什么：

```php
<li class="toggle-topbar  menu-icon"><a href="#"><span>Menu</span></a></li>
```

您可以省略 `<span>` 元素，这样只会显示菜单图标，或者省略 `menu-icon` 类。显然，您不应该同时省略两者，否则人们将无法访问您的魔法。

### 更多魔法——画布，最酷的东西

> *将您的主菜单放在手机的左侧，直到您需要它。*

我认为这是 Foundation 的*精华所在*，请原谅我用法语的用法。第一次使用它时，我印象深刻。下次我去 Facebook 时，我觉得我在 Facebook 应用中认出了它。

这个概念相当简单。在您的代码中，将最初不应显示的菜单内容放置在左侧。重要的是要控制它应该位于左侧的哪个部分。当用户点击菜单按钮，通常是菜单栏图标时，菜单将从左侧滑入。

这样，我们可以从上一个示例扩展。我们不再让 Foundation 给我们原始菜单的移动版本，现在我们可以指定一个移动优先的菜单内容。这里有一个小示例。在附加章节中会有一个更大的示例，我们将在网上添加。我们还使用可见性类确保全宽菜单不会出现，而是被带有图标的菜单替换。一旦您点击它，左侧的画布将出现在内容旁边。高度由您放置关闭 `<div>` 标签的位置决定，用于 `off-canvas-wrap` 和 `inner-wrap`：

```php
<div  class="off-canvas-wrap"  data-offcanvas>
 <div class="inner-wrap">
 <div class="row">
 <nav class="tab-bar show-for-small">
 <a class="left-off-canvas-toggle menu-icon ">
 <span>Menu</span>
 </a>
 </nav>

<div class="left-off-canvas-menu show-for-small">
<ul class="off-canvas-list exit-off-canvas">
<li><label>Portfolios</label></li>
<li><a>High Sierra</a></li>
<li><a>Eastern Sierra</a></li>
<li><a>June Lake</a></li>
<li><a>California Coast</a></li>
</ul>
</div>
<nav class="top-bar hide-for-small" data-topbar>
<ul class="title-area">
<li class="name">
<h1>
<a>My Site</a>
</h1>
</li>
</ul>
<section class="top-bar-section">
<ul class="right">
...
```

# 摘要

在本章中，我们介绍了 Foundation CSS/JavaScript 框架。它允许我们创建一个移动优先、响应式设计的网站或应用程序，而无需编写任何响应式部分的代码。我们只需根据我们的需求进行定制。

这本书涵盖了我想要称之为现代网络开发的这部分内容到此结束。在最后一章中，我们将前进到对我们大多数人来说似乎是*前卫*的内容，尽管到这本书印刷的时候，它将会非常*流行*。这一切都是围绕一个叫做**node.js**的 JavaScript 事物构建的，而这个事物并不是用 JavaScript 编写的。
