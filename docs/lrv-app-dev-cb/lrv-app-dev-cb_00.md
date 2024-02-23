# 前言

Laravel 已经成为最快增长的 PHP 框架之一。凭借其表达性的语法和出色的文档，很容易在很短的时间内获得一个完全运行的 Web 应用程序。此外，现代 PHP 功能的使用使得 Laravel 4 版本非常容易根据我们自己的需求进行定制，也使得我们可以轻松地创建一个高度复杂的网站。它是简单和先进的完美结合。

这本书只涵盖了 Laravel 所能做的一小部分。把它看作一个起点，有代码示例可以让事情运转起来。然后自定义它们，添加到它们，或者组合它们来创建您自己的应用程序。可能性是无限的。

关于 Laravel 最好的一点是社区。如果您遇到问题并且谷歌搜索没有帮助，总会有人愿意帮助。您可以在 IRC（Freenode 上的`#laravel`）或论坛（[`forums.laravel.io`](http://forums.laravel.io)）上找到乐于助人的社区成员，或者您可以联系 Twitter 上的许多 Laravel 用户。

愉快的 Laravel 之旅！

# 本书涵盖的内容

第一章，“设置和安装 Laravel”，涵盖了将 Laravel 设置和运行起来的各种方式。

第二章，“使用表单和收集输入”，展示了在 Laravel 中使用表单的多种方式。它涵盖了使用 Laravel 的表单类以及一些基本验证。

第三章，“验证您的应用程序”，演示了如何对用户进行身份验证。我们将看到如何使用 OAuth、OpenId 和各种社交网络进行身份验证。

第四章，“存储和使用数据”，涵盖了所有与数据相关的内容，包括如何使用除了 MySQL 数据库之外的数据源。

第五章，“使用控制器和路由处理 URL 和 API”，介绍了 Laravel 中的各种路由方法以及如何创建一个基本的 API。

第六章，“显示您的视图”，演示了在 Laravel 中视图的工作方式。我们还将整合 Twig 模板系统和 Twitter Bootstrap。

第七章，“创建和使用 Composer 包”，解释了如何在我们的应用程序中使用包，以及如何创建我们自己的包。

第八章，“使用 Ajax 和 jQuery”，提供了不同的示例，说明了如何在 Laravel 中使用 jQuery 以及如何进行异步请求。

第九章，“有效使用安全和会话”，涵盖了有关保护我们的应用程序以及如何使用会话和 cookie 的主题。

第十章，“测试和调试您的应用程序”，展示了如何在我们的应用程序中包含单元测试，使用 PHPUnit 和 Codeception。

第十一章，“部署和集成第三方服务到您的应用程序”，介绍了许多第三方服务以及我们如何将它们包含到我们的应用程序中。

# 你需要什么

这本书基本上需要一个工作的 LAMP 堆栈（Linux、Apache、MySQL 和 PHP）。Web 服务器是 Apache 2，可以在[`httpd.apache.org`](http://httpd.apache.org)找到。推荐的数据库服务器是 MySQL 5.6，可以从[`dev.mysql.com/downloads/mysql`](http://dev.mysql.com/downloads/mysql)下载。推荐的最低 PHP 版本是 5.4，可以在[`php.net/downloads.php`](http://php.net/downloads.php)找到。

对于一体化解决方案，还有一个 WAMP 服务器（[`www.wampserver.com/en`](http://www.wampserver.com/en)）或 XAMMP（[`www.apachefriends.org/en/xampp.html`](http://www.apachefriends.org/en/xampp.html)）适用于 Windows，或者 MAMP（[`www.mamp.info/en/mamp-pro`](http://www.mamp.info/en/mamp-pro)）适用于 Mac OS X。

# 本书适合对象

本书适用于具有中级 PHP 知识的人。了解另一个 PHP 框架或 Laravel 的第 3 版的基础知识也会有所帮助。对 MVC 结构和面向对象编程的一些了解也会有益处。

# 约定

本书中，您将找到许多不同类型信息的文本样式。以下是一些样式的示例，以及它们的含义解释。

文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 用户名，显示如下：“然后，我们使用`artisan`命令为我们生成一个新密钥，并自动保存在正确的文件中”。

代码块设置如下：

```php
Route::get('accounts', function()
{
  $accounts = Account::all();
  return View::make('accounts')->with('accounts', $accounts);
});
```

任何命令行输入或输出都会以以下方式书写：

```php
  php artisan key:generate
```

**新术语**和**重要单词**以粗体显示。您在屏幕上看到的单词，例如菜单或对话框中的单词，会以这种方式出现在文本中：“登录 Pagodabox 后，单击**新应用**选项卡”。

### 注意

警告或重要说明会以这样的方式出现在方框中。

### 提示

技巧会以这种方式出现。
