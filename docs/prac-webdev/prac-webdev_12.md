# 第十二章. 移动优先，渐进增强的响应式设计

这无疑是标题最长的章节。在这一章的前一部分，还有很多文字，正如你将发现那里几乎没有代码示例。在这里，我们讨论了最新的 Web 开发趋势，是什么导致了它，以及它取代了什么。

# 响应式设计

视口，或者说浏览器所在的屏幕部分，有各种大小。有些非常小，比如你智能手机的屏幕，有些可以非常大。当我看到我在一个比我习惯工作的屏幕大得多的屏幕上查看我的旧网站时，它看起来如此丑陋，以至于我决定完全重新设计它，使用**响应式设计**。

今天，使用指定像素固定宽度的网页设计真的不可行，同样，为了适应所有这些尺寸而制作你网站的多个版本也是不可行的。

在响应式设计——我总是想称之为负责任的设计——中，你不会从一个宽度为 960 像素的画布开始，然后那样构建一个网站，将其切割成固定大小的`<div>`块。一旦视口小于 960 像素，网站的一部分将不可见，也许更糟的是，当屏幕真的很大时，你 960 像素宽的矩形周围的一切都会显得无聊和空旷。这在今天根本不是一种做法。

一个好的设计应该能够调整、响应屏幕尺寸，并且始终保持美观。要有一个好的响应式设计，你需要：

+   使用灵活的网格：当有空间时，让您的构建块并排放置，当没有空间时，将它们堆叠在一起，并按比例调整大小以实现这一点

+   在定义这些尺寸时，在 CSS 中使用百分比而不是像素

+   使用媒体查询来指定不同屏幕尺寸、分辨率、方向等的不同属性

+   使用灵活的图片和字体

这看起来像是一项大量的工作。好消息是，其他人已经为你做了这些艰苦的工作。有几个 CSS/JavaScript 框架可供选择，它们都具备我们刚刚列出的所有功能。你只需添加定制的媒体查询即可。最难的部分可能是选择哪个框架，因为有几个非常好的框架，特别是**Bootstrap**和**Foundation**。

我们选择了 Foundation，这将是下一章的主题。这个选择是一个口味或偏好的问题。*关于口味，无需争论！*

## 前世今生

当我写这篇文章时，我有一种闪回般的体验。在我软件行业的第一份工作中，我负责将一个软件包，TEN/PLUS，移植到几种不同的 UNIX 版本。其主要组件是一个全屏编辑器。

那时的全屏是一个所谓的**哑终端**，有 24 行，每行 80 个字符。然后出现了**X 窗口系统**和图形模式的显示器，有图标和许多实用程序，支持鼠标等。这些实用程序之一是**xterm**，一个模拟哑终端的 X 客户端。当然，这意味着您可以在其中运行全屏编辑器。但是您可以调整 xterm 窗口的大小，使其有更多的行或更长的行。

因此，我不得不修改我们的软件以适应那个尺寸，并且仍然在 xterm 内部作为一个全屏编辑器，即使它是 64 行乘以 120 个字符。这在 25 年前就是响应式设计！而在 2015 年，人们仍然在终端窗口中使用 vi、emacs、nano 等编辑器。

## 媒体查询

关键字**media**自 HTML 和 CSS 一开始就被使用，但它基本上仅限于在指定 CSS 文件的`<link>`标签中使用`media="print"`或`media="screen"`，或者在使用 CSS 文件本身的`@media` screen。

从 CSS3 和 HTML5 开始，我们可以以更复杂的方式使用媒体查询，以便在满足某些条件时应用特定的样式。尽管`media`属性仍然可以在`<link>`标签中使用，但我们建议您在 CSS 文件中使用它们。媒体查询对于响应式设计至关重要，尽管我们承诺您几乎不需要工作，因为有了可用的框架，但了解如何编写或阅读它们是至关重要的。

这里是一个典型的媒体查询：

```php
@media only screen and (orientation: portrait) and (min-width: 480px) and (max-width: 690px) {  /* your rules here */   }
```

在大括号之间，您将编写适用于宽度在 480 到 690 像素之间且设备处于纵向模式的视口的样式。之前的一切都将适用。在大括号之间的一切将覆盖之前的内容。

这里是一些在媒体查询中可以使用的最常见的关键字：

+   **宽度**：显示区域的宽度

+   **高度**：显示区域的高度

+   **device-width**：设备的宽度

+   **device-height**：设备的宽度

+   **方向**：设备的方向（纵向或横向）

+   **分辨率**：像素密度，以 dpi 或 dpcm 表示

除了当然的方向之外，所有这些都可以由最小值或最大值来修饰。

以单词`only`开头开始媒体查询是一种处理不支持这些较新媒体查询的浏览器的便捷方法。它将被静默忽略。

宽度和高度之间的区别以及以设备为前缀的等效值应该容易理解：宽度和高度指的是浏览器视口的尺寸，而 device-width 和 device-height 指的是显示器的尺寸。并不是每个人都使用全屏浏览器，所以宽度和高度是您需要使用的测量值。但是有一个大问题。

移动浏览器填充可用屏幕，所以你可能期望宽度和设备宽度相同。不幸的是，情况并不总是如此。大多数智能手机将宽度设置为大约 1,000 像素的标称值（对于 iPhone，是 980 像素）。你看到过几个展示在小型手机上显示整个《纽约时报》全页的手机广告。那就是原因！

随着今天出色的视网膜显示屏，你甚至能够以这种方式阅读报纸。但如果你在为各种设备正确设置媒体查询上付出了很多努力，并且有一个针对`max-width:480px`的查询，那么你的美丽响应式设计将不会显示在那个手机上。幸运的是，有一种补救方法。只需将以下行放入你页面的`<head>`部分：

```php
<meta name="viewport" content="width=device-width, initial-scale=1.0">
```

现在，我们真正走上了响应式设计的道路。通过使用媒体查询，我们可以创建一个页面，该页面具有单一的设计，在大屏幕上显示页面上的元素（例如`<div>`）并排显示，在平板电脑的纵向模式下堆叠显示，在手机上显示更少的元素。

### 使用媒体属性

你可能已经看到一些网站在`<head>`部分包含这样的行：

```php
<link rel="stylesheet" type="text/css" href="print.css" media="print">
```

它使用`media`属性来指示是否需要使用此样式表。在 HTML4 中，这基本上是`screen`或`print`。在 HTML5 中，媒体属性可以有更多的值。这些值可以与我们在`@media`后放置的内容相同，仅限于我们的示例。因此，如果我们想的话，我们可以将 CSS 代码组织成单独的样式表，每个媒体类型一个，并按需使用它们，如下例所示：

```php
<link rel="stylesheet" type="text/css" href="small.css" media="screen and (max-width:480px)">
```

## 以少胜多

这个术语可能让经历过金融危机前后时期的人回忆起不好的记忆，但那不是我们要讨论的。

现在我们已经介绍了媒体查询，我们面临的风险是不得不写更多的 CSS，为每个我们决定使用的媒体查询编写一组规则。假设我们将其分为三种 T 恤尺寸：小号、中号、大号。再加上每种尺寸的纵向和横向版本：你现在有六种。

想象一下，营销部门有人决定更改公司颜色：你现在必须对你的样式表进行六处更改。

这就是 CSS 扩展（如**less**，**SASS**是另一个）非常有用的地方。其一个特性是使用**变量**。创建一个`.less`文件来保存你的样式表信息。使用变量来保存颜色信息：

```php
@mycolor: #FFDEAD;
```

在你的媒体查询中，无论何时需要指定此颜色，请使用：

```php
color: @mycolor;
```

假设你想将其改为`teal`。只需将那一行替换为：

```php
@mycolor: teal;
```

哈哈！你已经在这六个地方更改了颜色。这不是很好吗？

Less 具有更多功能，如混入（mixins）、嵌套规则、媒体查询等，这使得样式表结构更清晰且易于维护。所有这些都在[lesscss.org](http://lesscss.org)上描述得非常清楚。网站的名字立即解释了为什么他们称之为 less：你将最终写出更少的 CSS。只需看看这两个例子。以下是一个`less`代码示例：

```php
@mycolor: #FFDEAD;
@mygreen: teal;
#content {
  background-color: @mycolor;
  .greenp {
  color: @mygreen;
  font-size:12px;
  font-family:Avenir, sans-serif;
  }
  .thumb {
  border: 2px solid @mygreen;
  width:150px;
  }
}
```

这将被转换为以下 CSS。如果你真的开始嵌套，这可以变得更多：

```php
#content {
  background-color: #FFDEAD;
  }
#content  .greenp {
  color: teal;
  font-size:12px;
  font-family:Avenir, sans-serif;
  }
#content  .thumb {
  border: 2px solid teal;
  width:150px;
}
```

让我们再举一个使用媒体查询的例子。我们可以这样写：

```php
@medium: 16px;
@portraitfont: "Avenir, sans-serif";
.container {
@media screen {
  background-color:@mycolor;
  @media (min-width:786px) {
    font-size:@medium;
    @media (orientation:landscape) {
      margin:auto;
          }
    @media (orientation:portratit) {
      font-family:@portraitfont;
          }
        }
      }
    }
```

这将被转换为以下 CSS。你不认为`less`版本更简洁吗？

```php
@media screen {
    .container {
    background-color:#FFDEAD;
    }
}
@media screen and (min-width:786px) {
    .container {
      font-size:16px;
    }
  }
@media screen and (min-width:786px) and (orientation:landscape) {
    .container {
      margin:auto;
      }
}
@media screen and (min-width:786px) and (orientation:portrait) {
    .container {
      font-family:Avenir, Verdana, sans-serif;
      }
}
```

现在我们如何将 less 文件转换为 CSS 内容？有两种方法可以做到这一点。当我们达到生产阶段时，我们希望使用编译器将我们的 less 文件生成 CSS 文件。在第十四章中，你将学习关于 Node.js 和 node 模块的内容。有一个用于 less 的`node`模块。

目前，当我们还在开发和实验 less 时，请从[lesscss.org](http://lesscss.org)网站下载`less.js`。

在你页面的`<head>`部分，使用`link`标签列出你的`less`文件：

```php
<link rel="stylesheet/less" type="text/css"
href="less/mystyles.less"></link>
```

我喜欢将我的`less`文件保存在一个`less`文件夹中。例如，在你的页面上的正确位置，比如在关闭`</body>`标签之前，添加以下内容：

```php
<script src="img/less.js"></script>
```

现在每次页面加载时，你的`less`文件都会即时转换为 CSS。请注意，当你编辑`less`文件时，并非所有编辑器都会提供与编辑 CSS 文件时相同的出色颜色编码和格式化。我使用 Textastic 来编辑我的`less`文件。用州长的话来说：*你必须看看这个编辑器：它太棒了！*

## 首先考虑移动端

现在我们已经介绍了如何使我们的设计响应式，并使我们的页面在各种尺寸的屏幕上看起来都很好，你可能会认为我们已经涵盖了移动设备。不，还没有。

支持移动设备不仅仅是关于屏幕尺寸。你可能见过我们提到的那个广告，一个家伙正在他的 iPhone 上阅读《纽约时报》，整版。这就是移动设备在没有告诉它们不要这样做的情况下，在高分辨率显示器上渲染网站的方式。在我们继续这个话题之前，我想强调，这次讨论是关于如何设计由移动浏览器解释的**Web 应用程序**，而不是原生的 iOS 或 Android 应用程序。

### 为什么首先考虑移动端？

让我们用一个问题来回答：为什么不呢？或者让我们来回答它：因为移动端**确实是**首先考虑的。智能手机和平板电脑的出货量已经比 PC 或电视多 4-5 倍，现在已经有更多用户使用移动连接访问网络，而不是固定互联网接入。这就是为什么在考虑移动用户访问我们的网站时，首先考虑他们的体验，而不是首先创建一个固定画布版本的网站，然后再对移动用户进行修改，这是一个（不那么）*优雅降级*的例子。

如果你营销人员又带来设计公司提供的另一个静态设计，你可能没有选择。但营销人员喜欢数字，所以告诉他们检查数字，你可能会很快找到一个志同道合的人。

#### 我们已经走了很长的路

在 2007 年，不是很久以前，我正在开发一个带有我的照片的简单网站。我的一个朋友在智能手机上检查它。看到这个后，我不得不自己买一个，不是因为它有多好，而是因为我需要它来测试我的网站；至少，那是我的借口。尽管我实际上负担不起，但我买了最新的最棒的诺基亚，与今天的 iPhone 相比就像一块砖头，你可以翻开来给你一个更大的屏幕和真正的键盘。但它的网络接入非常糟糕。启动浏览器似乎需要很长时间。小键盘还可以，但与我现在使用坚固的蓝牙键盘在 iPhone 或 iPad 上的体验相比，那完全不同。事实上，这本书的大部分内容都是在火车上坐着用 iPad 写的。

在我之前回到比利时的一年里，我多次往返，因为我的父母家里没有互联网，所以我去了所谓的互联网热点来检查我的电子邮件。每小时 10 欧元，你可以买一张刮刮卡，上面有一个你需要与你的手机号码一起在本地运营商网站上使用的访问代码；然后他们通过短信给你发送密码。这个整个过程有时需要 20 分钟，并且因为我的手机号码是美国的，所以我需要支付国际漫游费用。现在我还有 40 分钟的 Wi-Fi 时间。移动互联网既慢又贵。

现在，这些障碍大多已经消失，带宽提高了，大多数地方的 Wi-Fi 都是免费的，或者你可以安装 SIM 卡。

为了进一步阐述“我们已经走了很长的路”这个说法，让我分享一个小故事。我是东部内华达山脉的大粉丝，已经去过那里很多次，大多数时候都是通过优胜美地国家公园。我知道一条通往公园西入口的不错的替代道路，它穿过科尔特维尔的金矿镇。

当地的杰弗里酒店入口门上方有两个招牌。一个写着：“**马格诺利亚酒吧 - 创立于 1851 年 - 加利福尼亚最古老的营业酒吧**”。上面的一个写着：“**免费 WiFi**”。

#### 移动设备有更多的新功能

上文描述的情况——除了酒吧——确实已经发生了转变。移动设备性能良好，互联网接入价格合理，大多数使用的设备都可以访问万维网，所以我们最好确保人们可以用它们访问我们的网站或应用程序。

我不习惯使用很多数字，到目前为止我只用了几个，但在最近的一项调查中，我了解到，在手机用户中，84%的人在在家时使用，63%的人在办公室使用，42%的人在移动中使用。

在移动设备上，没有鼠标点击，也没有在手机或平板电脑上悬停，但存在**滑动**。有许多新的界面，通常与硬件和 iOS 相关，你不会在桌面上找到这些界面。在手机上选择日期的界面与使用 jQuery UI 日期选择器不同。

如果你阅读了苹果网站上最新最伟大的 iPhone 的规格，我敢打赌他们忘记提到了这个产品的一个重要特性，至少不是以一种非常明显和明显的方式：你可以用它打电话。所以如果你的网站上的联系方式包含电话号码，让用户只需点击那个号码就能打电话。这就像给它加上一个`<a>`标签一样简单：

```php
<a href="tel:6505555555">6505555555</a>
```

这可能是一个微不足道的例子，但它说明了以移动为先的思考方式。手机将完成剩下的工作。所以确保当应用程序在手机上使用这些新功能之一时，它可以用。

#### 移动设备不仅在路上使用

忘记屏幕尺寸，考虑内容。这不仅仅是关于小与大的问题，而是关于移动与本地，关于在路上与在家的问题。当有人旅行并寻找酒店时，他期望在用手机查看酒店网站时能快速找到联系方式，而不是房间的照片和室内外游泳池的照片。但他在坐在家里的沙发上时，可能已经用同样的手机做了预订，因为孩子们正在使用电脑或 iPad。所以那天他对那些照片感兴趣。

因此，如果他正在寻找酒店，如果我们支持的话，可以在他的智能手机上显示一个类似于 GPS 的地图，显示他的位置以及他的酒店位置。

我们不希望做的事情是通过强制在带宽差且昂贵的地区下载 900x600 的 JPG 文件，从而使他的手机运营商变得富有，因为有人决定在任何时候在主页上放置一个照片横幅。

#### 内容优先，导航其次

之前的评论归结为以下几点。在我们的思考中，我们应该把最重要的内容放在最接近的地方。在页面的最右边或更糟糕的是页面的底部有一个带有*联系*的横向菜单是不会帮助我们的旅行者的。

在一个最近的项目中，我不得不处理一个为桌面屏幕设计的、包含水平导航的设计公司的线框图，我查看所有导航项并重新排列了它们，首先是联系，用响应式图像替换了照片横幅，并用带有下面三条横线的`menu`这个词替换了那个菜单。

我把大部分代码放在了`max-width:480px`媒体查询中。在`menu`上简单地按一下标签，就会在小屏幕上向访客展示包含所有相关主题的菜单。

我这样做的方式是，菜单看起来像是从你手机的左侧某个地方出现的，类似于 Facebook 应用的做法。我使用了**Foundation**来实现这一点。你将在下一章了解 Foundation。

#### 小意味着大

你可能已经注意到，在许多网站上，甚至是非常好的网站上，当你用智能手机访问它们时，你会被切换到网站的移动版本，一个不同的 URL 如`m.site.com`。这不是我们推荐的做法。一旦你决定采用两个版本，那么阻止你创建第三个版本，以此类推的是什么？许多坚持单一网站的网页设计师会对较小的屏幕尺寸做出反应，将所有东西都缩小，以便仍然适应屏幕。

在许多情况下，你可能想要做相反的事情，因为这会变得难以阅读。视网膜屏幕等设备现在非常清晰，这些屏幕上的小字体实际上很容易阅读。你可以通过在媒体查询中处理视网膜或非视网膜问题来实现这一点。

另一方面，这是一个指出，在这数百万通过手机访问互联网的新人中，许多是年长的一代，他们可能会欣赏更大的字体大小，因为他们的视力已经不如以前了。

真正的问题不是文本，而是你没有鼠标像素级别的精确度。当你想要点击某个东西时，你必须用手指。如果这些区域也缩小了，点击错误东西的风险会大大增加。

#### 移动输入

我们提到了阅读，提到了点击，那么填写网页表单呢？许多老式的网站——例如，包含注册表单的网店——有许多`<input>`字段，并且设计成可以适应整个屏幕，在最后有一个单独的*提交*按钮。所以，在手机上这会变得很困难；因此你可能想要考虑将这些表单拆分。如果输入字段不够大，用手指点击可能会让用户点到错误的位置。所以你希望让这些比在桌面上的更大。但关于输入还有另一件事经常被忘记：键盘。

在智能手机或平板电脑上，没有物理键盘。当网络应用程序/浏览器检测到需要输入时，一个软键盘会作为屏幕的一部分显示出来。在竖屏模式的智能手机上，这会让打字变得很困难。

当需要输入数字时，你需要首先将键盘切换到不同的模式，然后为一些符号切换到另一个模式。并不是每个人都会像我自己一样，到处都带着蓝牙键盘。对于这部分，你可以使用 HTML5 中的新`<input>`类型。大多数移动浏览器都支持 HTML5，所以当你使用以下内容时：

```php
<input type="email" name="email"></input>
```

这些浏览器会提供一个至少包含`@`符号的键盘布局，在某些键盘上这个符号很难找到。我再次生活在*azerty*国家。我知道。

### 移动优先回顾

你可能已经明白了。在这个拥有数百万移动设备的全新世界中，我们需要一种新的思维方式来考虑我们在网站上提供的信息以及如何提供这些信息。在可能的情况下，使用手机的功能；将内容放在导航之前。将你的网站设计成*一个网站适合所有人*，而不是*一个尺寸适合所有人*。这不仅仅是一种技术，更是一种哲学。

但我们需要一种技术来确定浏览器/设备组合是否支持某些功能，例如**地理位置**，甚至是否可以解释 JavaScript，并且只有在存在时才使用它。当我们这样做时，我们正在实践**渐进增强**。

# 渐进增强

几年前，一位同事曾告诉我们的项目经理，他 80%的时间都花在修改已经完成的代码上，使其能在旧浏览器中运行。在那些日子里，很多人花了很多时间在 Internet Explorer 上制作*圆角*，这是一个代价高昂（从开发时间和性能的角度来看）的尝试，目的是让一切看起来在所有地方都完全相同。很多人决定推迟适应 CSS3 和 HTML5 的新功能，因为一些浏览器不支持它们。这只是为了支持少数几位数的浏览器，并确保在关闭 JavaScript 时（优雅降级）网站仍然可用。

今天，有坏消息也有好消息。坏消息是浏览器比你能想象的要多，很多浏览器/设备对支持 API，而其他浏览器不支持。好消息是，已经出现了几种技术来更主动或更进步地解决这个问题。

那么，我们如何处理这个问题，同时还能提供一个网站的单一版本？首先，我们需要另一个哲学上的改变：不要害怕使用可用的功能，让你的网站和应用程序更加酷和有趣，并使用这些功能来增强它们。我们如何处理不支持这些功能的浏览器或设备？

我们提出一个两阶段的过程。首先，如果几乎没有任何现代功能（例如 JavaScript、花哨的 CSS 功能和媒体查询）可用，我们将确定我们网站的最小内容是什么，并据此开始编写我们的页面。这是我们基本网站。接下来，我们将添加诸如 jQuery 和 JavaScript 代码、媒体查询、更新的 CSS 和 HTML 功能、动画等内容，使我们的网站更美观，增强它。这被称为渐进增强。

再次强调，有 jQuery 和另一个名为**EnhanceJS**的库来帮助我们做到这一点。我们将展示如何做到这一点。然后，我们将展示如何改进这项技术：测试特定功能，如果可用则使用它，并可能使用替换库或 polyfill，将功能添加到不支持这些功能的浏览器中。我们已经在之前的章节中使用过这样的 polyfill——jQuery 的历史插件。

## EnhanceJS

EnhanceJS 将对浏览器进行一系列全面的测试。如果它们都通过，我们将加载构成我们网站增强版的文件，如 jQuery、带有媒体查询的 CSS 文件等。如果没有通过，我们看到的网站就是我们的基本网站。

请注意，如果我们想使用 Ajax 动态地将 HTML 注入到我们页面的部分，我们无法在基本版本中这样做。根据我们决定要放置多少内容，可能又回到了“回到未来”，因为我们将不得不提供指向静态页面的链接。这样，所有使用旧设备的访客至少会看到一个具有合理内容的可功能性网站。

看看这个例子：

```php
<!DOCTYPE html>
<html>
<head>
<meta charset=utf-8" />
<title>Progressive Hello World</title>
<link href="css/basic.css" type="text/css" rel="stylesheet" />
<script type="text/javascript" src="img/enhance.js"></script>
<script type="text/javascript">
  // Run capabilities test
  enhance({
  loadScripts: [
    'js/jquery.min.js',
    'js/enhanced.js'
    ],
  loadStyles: ['css/enhanced.css']
    });
</script>
</head>
<body>
<div id="content"><h1>Hello, world</h1></div>
</body>
</html>
```

### enhance.js

**enhance.js** 是一个 JavaScript 库，它执行一系列全面的测试，以查看浏览器是否支持足够的功能来处理你为增强网站所设想的功能。在撰写本文时，你可以在[`code.google.com/p/enhancejs`](https://code.google.com/p/enhancejs)找到它。你还可以在那里找到文档和有用的链接。

如果测试失败，`enhance()`内部将不会发生任何事情，你的基本网站，在我们的例子中是老式的*Hello World*，将是访客看到的。如果测试成功，enhance.js 将向`<html>`标签添加类`enhanced`并尝试执行你放在里面的任务。

### loadStyles 和 loadScripts

**loadStyles** 和 **loadScripts** 是两个数组，你可以指定你想要加载的样式表和 JavaScript 文件。你也可以指定条件，例如媒体查询，以有条件地加载一个文件或另一个文件。你不需要仅仅在数组中放置文件的路径名，你可以使用 JavaScript 对象，使用属性名作为键。所以在我们第一个例子中，我们可以这样写：

```php
enhance({
  loadScripts: [
    { src: 'js/jquery.min.js'},
    { src: 'js/enhanced.js'  }
    ],
  loadStyles: [
    { href: 'css/enhanced.css' }
    ]
  });
```

这里有一个更详细的例子：

```php
enhance({
   loadStyles: [
     {media: 'screen', href: 'js/mediumlarge.css',
     excludemedia: 'screen and (max-device-width: 480px)',},
     {media: 'screen and (max-device-width: 480px)', href: 'js/small.css'}
   ],
   loadScripts: [
     {media: 'screen', src: 'js/mediumlarge.js',
     excludemedia: 'screen and (max-device-width: 480px)'},
     {media: 'screen and (max-device-width: 480px)', src: 'js/small.js'}
   ]
});
```

### enhanced 和 FOUC

存在一个称为**无样式内容闪烁**（**FOUC**）的常见问题——请注意如何拼写这个缩写。这是在页面加载期间，由于你的 HTML 尚未被 JavaScript 代码处理而临时显示时，你会在屏幕上看到一些闪烁的现象。这里有一个机会，基于以下知识来解决这个问题：如果 enhance.js 测试成功，它将向`<html>`标签添加类`enhanced`。

在我们的例子中，你可以分别向你的**enhanced.css**和**enhanced.js**添加以下内容：

```php
#content {
html.enhanced display:none;
}

$('#content').show(); // this of course at the end of the processing
```

上述语句几乎意味着，在增强版的网站上，我们在 enhanced.js 中编写的代码将导致 HTML 被注入到`#content` div 内部。另一种方法是拥有`#basiccontent` div 和`#enhancedcontent` div，并只显示适当的一个。

如此描述的 Enhance.js 给我们提供了一个全有或全无的方法。它运行一系列全面的测试来确定访客应该看到基本版本还是增强版本。它们在文档中描述，当然也在 enhance.js 的代码中。在撰写本文时，enhance.js 自 2010 年以来没有更新。这并没有什么问题：它做的工作没有改变，而且做得很好。

这些测试中没有一项检查对一些较新的 HTML5 和 CSS3 功能的支持，例如 **Canvas**。Enhance.js 提供了更多的可配置选项，允许你添加自己的测试，并且如果你愿意，可以多次运行 `enhance()` 来测试不同的条件。所以你可以这样做。

### 注意

实际上没有必要，因为在网络开发的世界里，通常已经有整个团队为我们做了这件事。

## Modernizr

**Modernizr.js** 与 Enhance.js 类似，它是一个用于测试浏览器功能的 JavaScript 库。但它有更多的测试，可以自定义，并且可以在比 Enhance.js 更细粒度的层面上使用。Modernizr 测试单个功能。根据测试是否成功（*是*）或失败（*否*），我们可以加载不同的样式表和 `.js` 脚本。

你甚至可以将这两个一起使用。首先使用 enhance.js 来确定浏览器是否支持 JavaScript。如果支持，可以与 jQuery 一起加载 modernizr.js，并在你自己的 JavaScript 代码中细化你想要做的事情。你的基本页面可能看起来像这样：

```php
<!DOCTYPE html>
<html class="no-js">
<head>
<meta charset=utf-8" />
<title>My first progressive enhancement site</title>
<link href="css/basic.css" type="text/css" rel="stylesheet" />
<script type="text/javascript" src="img/enhance.js"></script>
<script type="text/javascript">
  enhance({
  loadScripts: [
    'js/jquery.js',
    'js/modernizr.js',
    'js/enhanced.js'
    ],
  loadStyles: ['css/enhanced.css']
    });
 </script>
 </head>
 <body>
 <div id="basiccontent"><h1>Welcome to Polbol Productions</h1><p>
Thank you for visiting us. Apparently you cannot use your browser to access all the
cool goodies on our site. Click <a href="basic.html">here</a> for more information on our
company.<br/> The Polbol Productions Team</p></div>
<div id="enhancedcontent"> <!-- Your real home page here --> </div>
 </body>
 </html>
```

因此，我们对其进行了小小的修改。我们当然将 `modernizr.js` 库添加到我们的 JavaScript 系列中，但一个显著的小变化是 `<html>` 标签的 `class="no-js"` 属性。而 enhance.js 会给这个标签添加 `enhanced` 类，Modernizr 在运行时会替换这个类为 `js`。所以如果它没有这样做，因为没有 JavaScript，你可以在你的样式表中使用 `.no-js` 类来考虑这一点。

Modernizr 将为 `<html>` 标签添加很多类，几乎每个测试通过都会有一个。所以你可以在你的样式表中容纳它们。以我在 MacBook Pro 上使用 Firefox 作为浏览器的例子，我使用 Firebug 检查并注意到 modernizr.js 已经将 `no-js` 替换为 `js`，并添加了以下类到 `<html>` 标签中：

```php
js flexbox flexboxlegacy canvas canvastext webgl no-touch geolocation postmessage no-websqldatabase indexeddb hashchange history draganddrop websockets rgba hsla multiplebgs backgroundsize borderimage borderradius boxshadow textshadow opacity cssanimations csscolumns cssgradients no-cssreflections csstransforms csstransforms3d csstransitions fontface generatedcontent video audio localstorage sessionstorage webworkers applicationcache svg inlinesvg smil svgclippaths
```

### Modernizr 对象

一旦加载了 modernizr.js 并执行了所有测试，你就可以访问 `Modernizr` 对象，并在你的 JavaScript 代码中检查测试是否通过：

```php
if (Modernizr.history) {
// code to use the HTML5 history API
}
else {
// code to use the API of your history plugin  or polyfill
}
```

### Polyfills 和 Modernizr

**Polyfills**是当浏览器缺少您需要的功能时，负责处理这些功能的脚本。我们在第九章中详细讨论的`history.js`库，*历史 API—不忘记我们身在何处*，可以是 HTML5 历史 API 的 polyfill。我说“可以是”，因为在那一章的例子中，我们无论何时都使用了 history.js。真正的 polyfills 仅在功能缺失时使用。所以，上面的代码如果 HTML5 支持存在，可以使用不同的 API，而不是通过 polyfill 提供的 API。

这可能解释了为什么选择了 Modernizr 这个名字。而 enhance.js 允许您照顾使用旧技术的人，并为他们提供一个使用当前技术的界面，Modernizr 则允许您在今天编写您的访客明天或下周将使用的代码，当他们更新他们的电脑或购买新手机或平板电脑时。您现在已经领先一步。

您的网站或应用程序已经使用了 HTML5 和 CSS3 提供的所有酷炫功能，并且有回退代码以备老旧技术涉及。您可以激励您的客户或访客尽早更新，但您的网站或应用程序已经支持了对于他们来说属于未来的功能。这是否很酷？

### yepnope.js 或 Modernizr.load

就像您在*Enhance.js*部分看到的那样，您可以根据测试通过或失败有条件地加载文件。最酷的有条件加载器是**yepnope.js**。在撰写本文时，它即将被弃用，因此它可能从 Modernizr 中消失。当然，会有新的东西出现。所以，现在，我们提供了一个简短的例子，说明您如何使用 Modernizr 进行静态资源的条件加载：

```php
Modernizr.load({
 test: Modernizr.geolocation,
  yep : 'geo.js',
  nope: 'geo-polyfill.js'
});
```

# 摘要

本章是本书中最长的标题，非常适合最重要的章节。在其中，我们解释了昨天和明天网络开发的区别。我们解释了您需要首先考虑移动端，您的设计必须是响应式的，并且您应该通过渐进增强使用酷炫的新特性来奖励那些拥有最新、最棒东西的客户。

在接下来的章节中，我们将引导您如何使用酷炫的 CSS/JavaScript 框架 Foundation 来应用之前所学的一切，这样您就可以使用别人已经为您完成的工作。我们还将向您展示一种方法，让您能够完全控制服务器端如何处理您想要的事情，使用 node.js。

如果您在这里停下来，您已经学到了现代网络开发的基础知识，我为您的成就表示祝贺。如果您继续前进，您将学会使用一个框架，这个框架将节省您大量的时间和工作，并且看起来是一种完全不同的做事方式。随着我们继续解释，您会意识到它最初并没有那么不同。
