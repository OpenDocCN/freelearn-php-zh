# 前言

Yii 框架是一个高性能、快速、开源的 PHP 框架，可用于开发现代网络应用。它为开发个人项目和商业应用提供了工具集。

本书是使用 Yii 框架开发八个可重用实际应用的逐步指南。*Yii 项目蓝图*将引导你通过几个项目，从项目构思到规划项目，最后实现项目。你将探索 Yii 框架的关键特性，并学习如何高效有效地使用它来构建可重用的核心应用，这些应用可以在实际项目中使用。你还将学习如何将 Yii 与第三方库和服务集成，创建自己的可重用代码库，并发现更多扩展你对 Yii 知识和专长的 Yii 特性。

# 本书涵盖的内容

第一章, *任务管理应用*，从零开始使用 SQLite 和基本数据库迁移开发一个简单的任务管理应用。本章将涵盖 Yii 的所有组成部分，并为你准备使用更复杂的应用。

第二章, *发现附近的资源*，介绍了如何将 Yii 框架与 Google Maps API 集成以显示用户附近的信息。你还将学习如何创建命令行工具来处理导入和处理数据。

第三章, *计划提醒*，专注于开发一个多用户基于网络的调度和提醒应用，该应用可以在计划事件即将发生时通过电子邮件通知用户。

第四章, *开发问题跟踪应用*，介绍了如何创建一个多用户问题跟踪和管理系统，包括使用 MySQL 作为数据库后端的一个电子邮件通知系统。本章还将涵盖处理电子邮件提交的输入以在应用程序中触发操作。

第五章, *创建一个微博平台*，介绍了如何创建一个类似于 Twitter 的自己的微博平台，包括一个强大的用户认证和注册系统。你还将学习如何使用 HybridAuth 将你的应用程序与第三方社交网络集成，以及如何使用 Composer 简化你的无头开发时间。

第六章, *构建内容管理系统*，介绍了如何创建一个功能完善的内容管理系统和博客平台，该平台基于前几章构建的知识扩展。本章还将演示如何与更多的第三方开源库集成。

第七章, *为 CMS 创建管理模块*，专注于开发在前一章中构建的内容管理系统的管理模块。在本章中，您将学习如何将数据从控制器迁移到可以独立于内容管理系统重用和管理的 Yii 模块。

第八章, *为 CMS 构建 API*，介绍了如何为内容管理系统创建一个 JSON REST API 模块，该模块可用于客户端 Web 应用程序和原生开发。本章将涵盖创建安全且经过身份验证的 JSON REST API 的基础知识，并演示如何将控制器操作调整为 JSON 响应而不是 Web 视图响应。

# 您需要为本书准备的内容

为了确保您可以在任何操作系统上运行提供的示例，并确保命令行条目的准确性，本书将使用 VirtualBox 和 Vagrant 来建立一个共同的开发平台。本书提供了如何设置此跨平台开发环境的说明。对于本书，您需要以下内容：

+   VirtualBox 4.3.x

+   Vagrant 1.3.x

+   Ubuntu 服务器 14.04 LTS

+   MySQL 5.6.x

+   PHP 5.5.x

+   Yii 框架 1.1.x

+   Composer

# 本书面向对象

如果您是一位对 PHP5 有良好了解且有一定 Yii 框架经验的 PHP 开发者，希望快速提升您的 Yii 知识并开始构建可重用的实际应用程序和工具，那么这本书是为您准备的。

# 习惯用法

在本书中，您将找到多种文本样式，用于区分不同类型的信息。以下是一些这些样式的示例及其含义的解释。

文本中的代码词汇、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 昵称应如下所示：“我们还需要修改我们的`UserIdentity`类，以允许社交认证用户登录。”

代码块应如下设置：

```php
<?php
// change the following paths if necessary
$config=dirname(__FILE__).'/config/main.php';
$config = require($config);
require_once('/opt/frameworks/php/yii/framework/yiic.php');
```

当我们希望您注意代码块中的特定部分时，相关的行或项目将以粗体显示：

```php
<div class="form-group">
<?php $selected = array('options' => array(isset($_
GET['id']) ? $_GET['id'] : NULL => array('selected' => true))); ?>
<?php echo CHtml::dropDownList('id', array(), CH
tml::listData(Location::model()->findAll(),'id','name'), 
CMap::mergeArray($selected, array('empty' => 'Select a Location'))); 
?>
</div>
<button type="submit" class="pull-right btn btnprimary">Search</button>
</form>
```

任何命令行输入或输出都应如下所示：

```php
$ php protected/yiic.php migrate up

```

**新术语**和**重要词汇**将以粗体显示。您在屏幕上看到的，例如在菜单或对话框中的文字，将以如下形式显示：“点击标题为**模型生成器**的链接，然后填写出现在页面上的表单。”

### 注意

警告或重要提示将以这样的框显示。

### 小贴士

技巧和窍门将以这样的形式出现。

# 读者反馈

我们始终欢迎读者的反馈。请告诉我们您对这本书的看法——您喜欢什么或可能不喜欢什么。读者反馈对我们开发您真正能从中获得最大收益的标题非常重要。

要发送给我们一般性的反馈，只需发送一封电子邮件到 `<feedback@packtpub.com>`，并在邮件的主题中提及书名。如果你在某个主题上有专业知识，并且你对撰写或为书籍做出贡献感兴趣，请参阅我们的作者指南 [www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

现在你已经是 Packt 书籍的骄傲拥有者，我们有许多事情可以帮助你从你的购买中获得最大收益。

## 下载示例代码

你可以从 [`www.packtpub.com`](http://www.packtpub.com) 的账户下载你购买的所有 Packt 书籍的示例代码文件。如果你在其他地方购买了这本书，你可以访问 [`www.packtpub.com/support`](http://www.packtpub.com/support) 并注册，以便将文件直接通过电子邮件发送给你。

## 下载本书的彩色图像

我们还为你提供了一个包含本书中使用的截图/图表彩色图像的 PDF 文件。这些彩色图像将帮助你更好地理解输出的变化。你可以从以下链接下载此文件：[`www.packtpub.com/sites/default/files/downloads/7734OS_ColoredImages.pdf`](https://www.packtpub.com/sites/default/files/downloads/7734OS_ColoredImages.pdf)。

## 勘误

尽管我们已经尽最大努力确保内容的准确性，但错误仍然可能发生。如果你在我们的书中发现错误——可能是文本或代码中的错误——如果你能向我们报告这一点，我们将不胜感激。通过这样做，你可以帮助其他读者避免挫败感，并帮助我们改进本书的后续版本。如果你发现任何勘误，请通过访问 [`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)，选择你的书籍，点击**勘误****提交****表单**链接，并输入你的勘误详情来报告。一旦你的勘误得到验证，你的提交将被接受，勘误将被上传到我们的网站，或添加到该标题的勘误部分下的现有勘误列表中。任何现有勘误都可以通过从 [`www.packtpub.com/support`](http://www.packtpub.com/support) 选择你的标题来查看。

## 侵权

在互联网上对版权材料的侵权是一个跨所有媒体的持续问题。在 Packt，我们非常重视我们版权和许可证的保护。如果你在互联网上发现任何形式的非法副本，请立即提供位置地址或网站名称，以便我们可以寻求补救措施。

请通过 `<copyright@packtpub.com>` 联系我们，并提供涉嫌盗版材料的链接。

我们感谢你在保护我们的作者和我们提供有价值内容的能力方面提供的帮助。

## 问题

如果你在这本书的任何方面遇到问题，可以通过 `<questions@packtpub.com>` 联系我们，我们将尽力解决。
