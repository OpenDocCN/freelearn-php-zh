# 前言

在本书中，我们将从头到尾构建 5 个不同的 WordPress 主题。我们将探讨构建优秀主题所需的所有基本概念。

为了阅读本书，你应该在 HTML/CSS 和 PHP 方面有一些经验。你还应该对 WordPress 有一个大致的了解——它的安装和 WordPress 网站管理——以及一些编程基础知识的理解，例如数组、变量、循环、语句等。项目主要基于 HTML5、CSS3 和 PHP。

除了这些，本书还将探讨一些其他的技术和概念。这包括 WordPress 帖子循环，这是从数据库抓取 WordPress 的主要循环，钩子/动作，`functions.php`文件，我们在其中放置我们的 WordPress 主题的动态代码，小工具，WP 查询，以及主题定制器。此外，我们还将使用一系列框架，如 Bootstrap、Foundation 和 W3 CSS，这是一个相对较新的框架。

那么，让我们深入其中，开始构建酷炫的主题。

# 本书涵盖的内容

第一章，*使用 WordPress 创建简单主题*，是一个入门项目章节。我们将讨论我们需要为我们的主题创建的文件、语法以及动态片段。

第二章，*构建 WordPress 主题*，是一个深入的项目章节，使用高级概念构建 WordPress 主题，包括自定义模板和主页、存档页面和帖子格式。

第三章，*构建照片画廊的 WordPress 主题*，是关于构建照片画廊 WordPress 主题的项目。我们将使用 w3.CSS 框架以及一些简单的动画来构建这个主题。

第四章，*构建 Twitter Bootstrap WordPress 主题*，是一个解释 Bootstrap 与 WordPress 集成的项目章节。这将是我们使用 Wordstrap 来实现我们的 WordPress 主题的章节。我们还将使用 WP 导航遍历器，这是一个用于下拉菜单的类。

第五章，*使用 Foundation 框架构建电子商务主题*，是关于使用基础框架构建电子商务主题的内容，这与 Bootstrap 框架类似。

# 你需要为本书准备的内容

为了完成本书中的项目，你需要以下内容：

+   HTML5/CSS3

+   PHP

+   WordPress

+   W3.CSS 框架

# 本书面向的对象

如果你是一位博客作者或 WordPress 用户，想要学习如何创建吸引人、引人注目的 WordPress 主题，这本书就是为你准备的。你只需要对 HTML5、CSS3、PHP 有一定的了解，以及一些创意，就可以开始阅读本书。

# 术语

在本书中，您将找到多种文本样式，用以区分不同类型的信息。以下是一些这些样式的示例及其含义的解释。

文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 用户名将如下所示：文本中的代码单词将如下所示：“`single.html`文件将代表单个图像。”

代码块将以如下形式设置：

```php
<!DOCTYPE html>
<html>
<head>
  <title></title>
</head>
<body>
</body>
</html>
```

当我们希望您注意代码块中的特定部分时，相关的行或项目将以粗体显示：

```php
<!DOCTYPE html>
<html>
<head>
  <title>PhotoGenik</title>
</head>
<body>
</body>
</html>
```

新术语和重要单词将以粗体显示。屏幕上显示的单词，例如在菜单或对话框中，将以文本中的如下形式出现：“要上传文件，我们将点击选择文件按钮。”

警告或重要提示将以如下框的形式出现。

小贴士和技巧将以如下形式出现。

# 读者反馈

我们的读者反馈总是受欢迎的。请告诉我们您对本书的看法——您喜欢什么或可能不喜欢什么。读者反馈对我们开发您真正能从中获得最大价值的标题非常重要。

要向我们发送一般反馈，只需发送一封电子邮件到`feedback@packtpub.com`，并在邮件主题中提及书籍标题。

如果您在某个领域有专业知识，并且对撰写或参与书籍感兴趣，请参阅我们的作者指南[www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

现在，您已经成为 Packt 图书的骄傲拥有者，我们有一些事情可以帮助您从您的购买中获得最大收益。

# 下载示例代码

您可以从[`www.packtpub.com`](http://www.packtpub.com)的账户下载本书的示例代码文件。如果您在其他地方购买了本书，您可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)并注册，以便将文件直接通过电子邮件发送给您。您可以通过以下步骤下载代码文件：

1.  使用您的电子邮件地址和密码登录或注册我们的网站。

1.  将鼠标指针悬停在顶部的 SUPPORT 标签上。

1.  点击代码下载与勘误。

1.  在搜索框中输入书籍名称。

1.  选择您想要下载代码文件的书籍。

1.  从下拉菜单中选择您购买本书的地方。

1.  点击代码下载。

文件下载后，请确保使用最新版本的软件解压或提取文件夹：

+   WinRAR / 7-Zip for Windows

+   Zipeg / iZip / UnRarX for Mac

+   7-Zip / PeaZip for Linux

本书代码包也托管在 GitHub 上，地址为[`github.com/PacktPublishing/Learn-to-Create-WordPress-Themes-by-Building-5-Projects`](https://github.com/PacktPublishing/Learn-to-Create-WordPress-Themes-by-Building-5-Projects)。我们还有其他来自我们丰富图书和视频目录的代码包可供选择，地址为[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)。请查看它们！

# 下载本书中的彩色图像

我们还为您提供了一个包含本书中使用的截图/图表的彩色图像的 PDF 文件。彩色图像将帮助您更好地理解输出的变化。您可以从[`www.packtpub.com/sites/default/files/downloads/LearntoCreateWordPressThemesByBuilding5Projects_ColorImages.pdf`](https://www.packtpub.com/sites/default/files/downloads/LearntoCreateWordPressThemesByBuilding5Projects_ColorImages.pdf)下载此文件。

# 勘误

尽管我们已经尽最大努力确保内容的准确性，但错误仍然可能发生。如果您在我们的书中发现错误——可能是文本或代码中的错误——如果您能向我们报告这一点，我们将不胜感激。通过这样做，您可以避免其他读者的挫败感，并帮助我们改进本书的后续版本。如果您发现任何勘误，请通过访问[`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)，选择您的书籍，点击勘误提交表单链接，并输入您的勘误详情来报告它们。一旦您的勘误得到验证，您的提交将被接受，勘误将被上传到我们的网站或添加到该标题的勘误部分下的现有勘误列表中。要查看之前提交的勘误，请访问[`www.packtpub.com/books/content/support`](https://www.packtpub.com/books/content/support)，并在搜索字段中输入书籍名称。所需信息将出现在勘误部分下。

# 盗版

在互联网上盗版受版权保护的材料是一个跨所有媒体的持续问题。在 Packt，我们非常重视保护我们的版权和许可证。如果您在互联网上发现我们作品的任何非法副本，无论形式如何，请立即向我们提供位置地址或网站名称，以便我们可以寻求补救措施。请通过`copyright@packtpub.com`与我们联系，并提供疑似盗版材料的链接。我们感谢您的帮助，以保护我们的作者和我们为您提供有价值内容的能力。

# 问题

如果您对本书的任何方面有问题，您可以通过`questions@packtpub.com`联系我们，我们将尽力解决问题。
