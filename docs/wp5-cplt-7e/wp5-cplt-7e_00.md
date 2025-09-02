# 前言

*《WordPress 5 完整版，第七版》* 将带你从头开始构建一个功能齐全的 WordPress 网站的全过程。这个过程从教你如何安装 WordPress 开始，一直延伸到最先进的话题，例如创建自己的主题、编写插件，甚至构建非博客网站。最好的部分是，你可以在不损失任何东西的情况下完成所有这些。此外，一旦你积累了一些经验，你将能够在几分钟内启动新的 WordPress 网站（顺便说一句，这不是比喻；这是最真实的情况）。

这本书以逐步的方式引导你，解释了关于 WordPress 的所有知识。我们将从下载和安装 WordPress 的核心开始，你将学习如何选择正确的设置，以确保你和你访问者的体验顺畅。之后，这本书将教你关于网站内容管理功能的所有内容，从帖子、页面到分类和标签，一直到媒体、菜单、图片、画廊、安全、管理、用户资料等等。接下来，你将了解插件和主题是什么，以及如何有效地使用它们。最后，你将学习如何创建自己的主题和插件，以增强你网站的整体功能。一旦你完成了*《WordPress 5.0 完整版，第七版》*，你将拥有从头开始构建专业 WordPress 网站所需的所有知识。

# 这本书面向的对象

这本书是针对 WordPress 初学者以及那些对 WordPress 有一定了解的人的指南。如果你是博客新手，想要以简单直接的方式创建自己的博客或网站，那么这本书就是为你准备的。它也适合那些想学习如何定制和扩展 WordPress 网站功能的人。你不需要具备任何详细的编程或网络开发知识，任何对 IT 有信心的人都能使用这本书制作出令人印象深刻的网站。

# 为了充分利用这本书

为了跟随本书中的示例，你需要以下内容：

+   一台电脑

+   一个网络浏览器

+   一个纯文本编辑器

+   FTP 软件

你可以考虑使用一个突出显示代码的文本编辑器（例如 Coda、TextMate、HTMLKit 等），但一个简单的纯文本编辑器就足够了。你可能想在电脑上运行 WordPress 的本地副本，在这种情况下，你可能需要安装 Apache 和 MySQL 服务器（尽管 WAMP、XAMPP 或 MAMP 可以为你处理所有这些）。但即使这样也不是必需的，因为你可以远程完成整个操作。

# 下载示例代码文件

你可以从[www.packt.com](http://www.packt.com)的账户下载这本书的示例代码文件。如果你在其他地方购买了这本书，你可以访问[www.packt.com/support](http://www.packt.com/support)并注册，以便将文件直接通过电子邮件发送给你。

您可以通过以下步骤下载代码文件：

1.  在[www.packt.com](http://www.packt.com)登录或注册。

1.  选择 SUPPORT 标签。

1.  点击代码下载与勘误。

1.  在搜索框中输入书籍名称，并遵循屏幕上的说明。

文件下载后，请确保使用最新版本解压缩或提取文件夹：

+   WinRAR/7-Zip for Windows

+   Zipeg/iZip/UnRarX for Mac

+   7-Zip/PeaZip for Linux

本书代码包也托管在 GitHub 上，网址为[`github.com/PacktPublishing/WordPress-5-Complete-7th-Edition`](https://github.com/PacktPublishing/WordPress-5-Complete-7th-Edition)。如果代码有更新，它将在现有的 GitHub 仓库中更新。

我们还有其他来自我们丰富图书和视频目录的代码包可供在**[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)**上获取。查看它们吧！

# 下载彩色图像

我们还提供了一份包含本书中使用的截图/图表彩色图像的 PDF 文件。您可以从这里下载：[`www.packtpub.com/sites/default/files/downloads/9781789532012_ColorImages.pdf`](http://www.packtpub.com/sites/default/files/downloads/9781789532012_ColorImages.pdf)。

# 使用的约定

本书使用了多种文本约定。

`CodeInText`：表示文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 昵称。以下是一个示例：“如果您在本地服务器上安装 WordPress，请确保将 WordPress 文件放置在计算机上正确的`webroot`目录中。”

代码块设置如下：

```php
.site-footer {
  float: right;
  padding: 20px;
}
```

**粗体**：表示新术语、重要单词或屏幕上看到的单词。例如，菜单或对话框中的单词在文本中显示如下。以下是一个示例：“现在，点击安装 WordPress。”

警告或重要注意事项如下所示。

小贴士和技巧如下所示。

# 联系我们

我们始终欢迎读者的反馈。

**一般反馈**：如果您对本书的任何方面有疑问，请在邮件主题中提及书籍标题，并通过`customercare@packtpub.com`给我们发邮件。

**勘误**：尽管我们已经尽一切努力确保内容的准确性，但错误仍然可能发生。如果您在这本书中发现了错误，我们将不胜感激，如果您能向我们报告，请访问[www.packt.com/submit-errata](http://www.packt.com/submit-errata)，选择您的书籍，点击勘误提交表单链接，并输入详细信息。

**盗版**：如果您在互联网上以任何形式遇到我们作品的非法副本，如果您能提供位置地址或网站名称，我们将不胜感激。请通过`copyright@packt.com`与我们联系，并提供材料的链接。

**如果您有兴趣成为作者**：如果您在某个领域有专业知识，并且您有兴趣撰写或为书籍做出贡献，请访问[authors.packtpub.com](http://authors.packtpub.com/)。

# 评论

请留下评论。一旦您阅读并使用过这本书，为何不在购买它的网站上留下评论呢？潜在读者可以查看并使用您的客观意见来做出购买决定，我们 Packt 可以了解您对我们产品的看法，而我们的作者也可以看到他们对书籍的反馈。谢谢！

想了解更多关于 Packt 的信息，请访问 [packt.com](http://www.packt.com/).
