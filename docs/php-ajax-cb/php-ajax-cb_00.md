# 前言

Ajax 是 Web 2.0 网站中必不可少的范式。大多数 Web 2.0 网站都是用 PHP 和 Ajax 构建的。扩展 Ajax 是为了以快速简便的方式提供访问 PHP 后端服务的前端服务。有了这本书，你将学会如何使用必要的工具来实现网站和 iPhone 的 Ajax 化。

*PHP Ajax Cookbook*将教你如何将 PHP 和 Ajax 的组合作为网站或 Web 应用程序的强大平台。使用 Ajax 与服务器通信可以加快 PHP 后端服务的响应速度。Ajax 和 PHP 的组合具有许多功能，如加快用户体验、让你的 Web 客户端获得更快的响应时间，并让客户端浏览器从服务器检索数据而无需刷新整个页面。你将学会优化和调试 Ajax 应用程序。此外，你还将学会如何在 iPhone 设备上编写 Ajax 程序。

本书将教你流行的基于选择器的 JavaScript，然后介绍调试、优化和最佳实践的重要概念。其中包括一系列的配方，重点是创建基本工具，如使用 Ajax 验证表单和创建五星评级系统。由于 jQuery 非常流行，因此随后介绍了有用的工具和 jQuery 插件，如 Ajax 工具提示、选项卡导航、自动完成、购物车和 Ajax 聊天。到第七章结束时，你将学会如何加快网站响应速度，并构建 SEO 友好的 Ajax 网站。你还将了解所有流行的 Ajax web 服务和 API，如 Twitter、Facebook 和 Google Maps，这些都在第八章 *Ajax Mashups*中介绍。最后，逐步介绍了使用基本库和日常有用的 Ajax 工具构建 iPhone 应用的配方。

使用 PHP Ajax 构建丰富、交互式的 Web 2.0 网站和丰富的标准和 Mashups。

# 本书内容包括

第一章，*Ajax 库*，教我们如何使用最著名的 JavaScript 库和框架，具有 Ajax 功能的能力。这些库是根据我们的主观意见选择的，我们并不试图说哪个库/框架更好或更差。它们各自都有优点和缺点。

第二章，*基本工具*，着重介绍处理表单、表单控件、Ajax 表格和上传操作的基本 Ajax 操作。根据用户体验和特定系统的性能，解释了一些基于“最佳”实践。

第三章，*使用 jQuery 的有用工具*，讨论了 jQuery 插件，这些插件对将普通网站转变为具有良好外观的 Ajax 网站非常有用，如工具提示、带有灯箱的图库、日期选择器、快速视觉效果和布局功能。

第四章，*高级工具*，教我们如何构建高级功能，如聊天、绘制图表、使用画布解码验证码以及在网格中显示数据。

第五章，*调试和故障排除*，讨论了使用浏览器插件如 Firebug 进行 JavaScript 调试的技术。

第六章，*优化*，教我们如何通过缩小、提前触发 JavaScript、对象缓存以及来自 YSlow 和 Google Page Speed 工具的技巧来加快代码执行速度。

第七章，*实施构建 Ajax 网站的最佳实践*，讨论了避免特定标记代码、构建搜索引擎友好的 Ajax 网站、安全考虑和实施 Ajax Comet 等最佳实践。

第八章, *Ajax 混搭*，讨论了如何通过利用 Flickr、Picasa、Facebook、Twitter、Google Maps 和地理编码网络服务，从 JavaScript 中利用现有的网络服务。

第九章, *iPhone & Ajax*，教我们如何使用移动框架构建移动友好的网站，并使用 PhoneGap 框架构建原生 iPhone 应用程序。

# 你需要什么来读这本书

在这本书中，您基本上需要在计算机上安装 Apache、MySQL 和 PHP。如果您的计算机上没有安装 PHP、MySQL 或 Apache，我们建议您从其网站下载 XAMPP 软件包：[`www.apachefriends.org/en/xampp.html`](http://www.apachefriends.org/en/xampp.html)。此外，作为代码编辑器，您可以使用像 Notepad++（Windows）、IDE Netbeans 或 Eclipse 这样的简单编辑器。

# 这本书是为谁准备的

这本书是一个理想的资源，适合喜欢为网站添加 Ajax 功能并倾向于使用标准和最佳实践来构建 SEO 友好网站的人。由于本书涵盖了高级主题，读者需要了解基本的 PHP、JavaScript 和 XML 功能。

# 约定

在本书中，您会发现一些文本样式，用于区分不同类型的信息。以下是一些样式的示例，以及它们的含义解释。

文本中的代码单词显示如下：“我们可以通过使用`include`指令来包含其他上下文。”

代码块设置如下：

```php
if(isset($_GET["param"])){
$result["status"] = "OK";
$result["message"] = "Input is valid!";
} else {
$result["status"] = "ERROR";
$result["message"] = "Input IS NOT valid!";
}

```

当我们希望引起您对代码块的特定部分的注意时，相关的行或项目将以粗体显示：

```php
" $('#dob').datepicker({
" numberOfMonths: 2
" });

```

任何命令行输入或输出都以以下方式书写：

```php
# cp /usr/src/asterisk-addons/configs/cdr_mysql.conf.sample
/etc/asterisk/cdr_mysql.conf

```

**新术语**和**重要单词**以粗体显示。例如，屏幕上看到的单词，如菜单或对话框中的单词，会在文本中出现，如：“点击**下一步**按钮会将您移至下一个屏幕”。

### 注意

警告或重要提示会以这样的方式出现。

### 注意

提示和技巧会以这样的方式出现。
