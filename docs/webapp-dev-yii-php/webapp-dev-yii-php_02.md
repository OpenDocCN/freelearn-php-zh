# 第二章. 入门

通过简单地使用 Yii，我们很快就能发现 Yii 的真正乐趣和好处。在本章中，我们将看到在一个示例 Yii 应用程序中，前一章介绍的概念是如何体现的。遵循 Yii 的约定优于配置的理念，我们将按照标准约定开始编写一个 Yii 中的“Hello, World!”程序。

在本章中，我们将涵盖：

+   Yii 框架安装

+   创建一个新的应用程序

+   创建控制器和视图

+   向视图文件添加动态内容

+   Yii 请求路由和链接页面

我们的第一步是安装框架。现在让我们来做吧。

# 安装 Yii

在安装 Yii 之前，您必须将应用程序开发环境配置为支持 PHP 5.1.0 或更高版本的 Web 服务器。Yii 已经在 Windows 和 Linux 操作系统上的 Apache HTTP 服务器上进行了彻底测试。它也可以在支持 PHP 5 的其他 Web 服务器和平台上运行。我们假设读者以前已经参与过 PHP 开发，并且可以访问或者知道如何设置这样的环境。我们将把 Web 服务器和 PHP 本身的安装留给读者自己去练习。

### 注意

一些流行的安装包包括

+   [`www.apachefriends.org/en/xampp.html`](http://www.apachefriends.org/en/xampp.html)

+   [`www.mamp.info/en/index.html`](http://www.mamp.info/en/index.html) (`仅适用于 mac`)

基本的 Yii 安装几乎是微不足道的。实际上只有两个必要的步骤：

1.  从[`www.yiiframework.com/download/`](http://www.yiiframework.com/download/)下载 Yii 框架。

1.  将下载的文件解压到可通过 Web 访问的目录。在下载框架时，可以选择几个版本的 Yii。在本书的目的中，我们将使用 1.1.12 版本，这是写作时的最新稳定版本。虽然大多数示例代码应该适用于任何 1.1.x 版本的 Yii，但如果您使用不同版本可能会有一些细微差异。如果您正在跟随示例，请尝试使用 1.1.12 版本。

在下载了框架文件并将其解压到可通过 Web 访问的目录后，列出其内容。您应该看到以下高级目录和文件：

+   `CHANGELOG`

+   `LICENSE`

+   `README`

+   `UPGRADE`

+   `demos/`

+   `framework/`

+   `requirements/`

现在我们已经在可通过 Web 访问的目录中解压了我们的框架，建议您验证服务器是否满足使用 Yii 的所有要求，以确保安装成功。幸运的是，这样做非常容易。Yii 带有一个简单的要求检查工具。要使用该工具并让其验证您的安装要求，只需将浏览器指向所下载文件中的`requirements/`目录下的`index.php`入口脚本。例如，假设包含所有框架文件的目录的名称只是叫做`yii`，那么访问要求检查器的 URL 可能如下所示：

`http://localhost/yii/requirements/index.php`

以下屏幕截图显示了我们配置的结果：

![安装 Yii](img/8727_02_02.jpg)

使用要求检查器本身并不是安装的要求。但建议使用它来确保正确安装。正如您所看到的，我们在详细部分的结果并非全部都是**通过**状态，有些显示**警告**结果。当然，您的配置很可能与我们的略有不同，因此您的结果也可能略有不同。这没关系。并不是所有**详细**部分的检查都必须通过，但是必须在**结论**部分收到以下消息：**您的服务器配置满足 Yii 的最低要求**。

### 提示

Yii 框架文件不需要被放置在公开访问的 web 目录中，建议不要这样做。我们在这里这样做只是为了快速利用浏览器中的要求检查器。Yii 应用程序有一个入口脚本，通常是唯一需要放置在 web 根目录中的文件（web 根目录指的是包含`index.php`入口脚本的目录）。其他 PHP 脚本，包括所有的 Yii 框架文件，应该受到保护，以避免安全问题。只需在入口脚本中引用包含 Yii 框架文件的目录，并将这些文件放在 web 根目录之外。

## 安装数据库

在本书中，我们将使用数据库来支持许多示例和我们将要编写的应用程序。为了正确地跟随本书，建议你安装一个数据库服务器。虽然你可以使用 Yii 支持的任何数据库，如果你想使用 Yii 内置的数据库抽象层和工具，就像我们将要使用的那样，你需要使用框架支持的数据库。截至 1.1 版本，支持的数据库有：

+   MySQL 4.1 或更高版本

+   PostgresSQL 7.3 或更高版本

+   SQLite 2 和 3

+   Microsoft SQL Server 2000 或更高版本

+   Oracle

### 提示

虽然你可以使用任何受支持的数据库服务器来跟随本书中的所有示例，但我们将在所有示例中使用 MySQL（具体来说是 5.1）作为我们的数据库服务器。建议你也使用 MySQL，版本为 5 或更高，以确保所提供的示例可以正常工作而无需进行调整。在本章中，我们的简单的“Hello, World!”应用程序不需要数据库。

现在我们已经安装了框架并验证了我们已满足最低要求，让我们继续创建一个全新的 Yii web 应用程序。

# 创建一个新应用程序

为了创建一个新的应用程序，我们将使用一个随框架捆绑的强大工具，称为*yiic*。这是一个命令行工具，你可以用它快速引导一个全新的 Yii 应用程序。使用这个工具并不是强制的，但它可以节省时间，并保证应用程序有一个正确的目录和文件结构。

要使用这个工具，打开你的命令行，并导航到你的文件系统中你想要创建应用程序目录结构的地方。为了这个演示应用程序的目的，我们假设以下情况：

+   `YiiRoot`是你安装 Yii 框架文件的目录的名称

+   `WebRoot`被配置为你的 web 服务器的文档根目录

从命令行中，切换到你的`WebRoot`目录并执行`yiic`命令：

```php
% cd WebRoot
% YiiRoot/framework/yiic webapp helloworld
   Create a Web application under '/Webroot/helloworld'? [Yes|No] 
   Yes 
      mkdir /WebRoot/helloworld
      mkdir /WebRoot/helloworld/assets
      mkdir /WebRoot/helloworld/css
   generate css/bg.gif
   generate css/form.css
   generate css/main.css

Your application has been created successfully under /Webroot/helloworld.
```

### 注意

`yiic`命令可能不会按预期工作，特别是如果你尝试在 Windows 环境中使用它。`yiic`文件是一个可执行文件，使用你的命令行版本的 PHP 来运行。它调用`yiic.php`脚本。你可能需要在前面使用`php`来完全限定，如`$ php yiic`或`$ php yiic.php`。你可能还需要指定要使用的 PHP 可执行文件，比如`C:\PHP5\php.exe yiic.php`。还有`yiic.bat`文件，它执行`yiic.php`文件，可能更适合 Windows 用户。你可能需要确保你的 PHP 可执行文件位置在你的`%PATH%`变量中是可访问的。请尝试这些变化，找到适合你计算机配置的解决方案。我将继续简单地称这个命令为`yiic`。

`yiic webapp`命令用于创建一个全新的 Yii web 应用程序。它只需要一个参数来指定应用程序应该被创建的目录的绝对或相对路径。结果是生成所有必要的目录和文件，用于提供默认 Yii web 应用程序的框架。

让我们列出我们的新应用程序的内容，看看为我们创建了什么：

```php
assets/    images/    index.php  themes/
css/    index-test.php    protected/
```

以下是这些高级项目的描述，这些项目是自动创建的：

+   `index.php`: Web 应用程序入口脚本文件

+   `index-test.php`: 用于加载测试配置的入口脚本文件

+   `assets/`: 包含发布的资源文件

+   `css/`: 包含 CSS 文件

+   `images/`: 包含图像文件

+   `themes/`: 包含应用程序主题

+   `protected/`: 包含受保护的（非公开的）应用程序文件

通过一条简单的命令行命令的执行，我们已经创建了所有所需的目录结构和文件，以立即利用 Yii 的合理默认配置。这些目录和文件，以及它们包含的子目录和文件，乍一看可能有点令人生畏。然而，我们在开始时可以忽略大部分内容。重要的是要注意，所有这些目录和文件实际上都是一个工作的 Web 应用程序。`yiic`命令已经填充了应用程序足够的代码，以建立一个简单的首页，一个典型的联系我们页面，以提供一个 Web 表单的示例，以及一个登录页面，以演示 Yii 中的基本授权和认证。如果您的 Web 服务器支持 GD2 图形库扩展，您还将在联系我们表单上看到一个 CAPTCHA 小部件，并且应用程序将对该表单字段进行相应的验证。

只要您的 Web 服务器正在运行，您就应该能够打开浏览器并导航到`http://localhost/helloworld/index.php`。在这里，您将看到一个**我的 Web 应用程序**首页，以及友好的问候语**欢迎来到我的 Web 应用程序**，接着是一些有用的下一步信息。以下截图显示了这个示例首页：

![创建一个新应用程序](img/8727_02_01.jpg)

### 注意

您需要确保`assets/`和`protected/runtime/`目录对您的 Web 服务器进程是可写的，否则您可能会看到一个错误而不是工作应用程序。

您会注意到页面顶部有一个可用的应用程序导航栏。从左到右依次是**主页**、**关于**、**联系**和**登录**。点击并探索。点击**关于**链接提供了一个静态页面的简单示例。**联系**链接将带您到之前提到的联系我们表单，以及表单中的 CAPTCHA 输入字段。（再次强调，只有在您的 PHP 配置中有`gd`图形扩展时，您才会看到 CAPTCHA 字段。）

**登录**链接将带您到显示登录表单的页面。这是一个带有表单验证的工作代码，以及用户名和密码的验证和认证。使用*demo/demo*或*admin/admin*作为用户名/密码组合将使您登录到网站。试试看！您可以尝试一个将失败的登录（除了 demo/demo 或 admin/admin 之外的任何组合），并查看错误验证消息的显示。成功登录后，页眉中的**登录**链接将更改为**注销**链接（用户名），其中用户名是 demo 或 admin，具体取决于您用于登录的用户名。令人惊讶的是，所有这些都可以在不编写任何代码的情况下完成。

# "你好，世界！"

一旦我们通过一个简单的示例走过，所有这些生成的代码将开始变得更加清晰。为了尝试这个新系统，让我们构建在本章开头承诺的“你好，世界！”程序。在 Yii 中，“你好，世界！”程序将是一个向我们的浏览器发送这条非常重要消息的简单 Web 页面应用程序。

如在第一章中讨论的那样，*遇见 Yii*，Yii 是一个模型-视图-控制器框架。一个典型的 Yii web 应用程序接收用户的传入请求，处理该请求中的信息以创建一个控制器，然后调用该控制器中的一个动作。控制器可以调用特定的视图来渲染并返回响应给用户。如果涉及数据，控制器还可以与模型交互，处理数据的所有**CRUD**（**创建，读取，更新，删除**）操作。在我们简单的“你好，世界！”应用程序中，我们只需要控制器和视图的代码。我们不涉及任何数据，因此不需要模型。让我们通过创建我们的控制器来开始我们的示例。

## 创建控制器

以前，我们使用`yiic` `webapp`命令来帮助我们生成一个新的 Yii web 应用程序。为了为我们的“你好，世界！”应用程序创建一个新的控制器，我们将使用 Yii 提供的另一个实用工具。这个工具叫做 Gii。**Gii**是一个高度可定制和可扩展的基于 Web 的代码生成平台。

### 配置 Gii

在使用 Gii 之前，我们必须在应用程序中对其进行配置。我们在位于`protected/config/main.php`的主应用程序配置文件中进行配置。要配置 Gii，打开此文件并取消注释`gii`模块。我们的自动生成的代码已经添加了`gii`配置，但它被注释掉了。因此，我们只需要取消注释，然后还要添加我们自己的密码，如下面的代码片段所示：

```php
return array(
  'basePath'=>dirname(__FILE__).DIRECTORY_SEPARATOR.'..',
  'name'=>'My Web Application',

  // preloading 'log' component
  'preload'=>array('log'),

  // autoloading model and component classes
  'import'=>array(
    'application.models.*',
    'application.components.*',
  ),

 **'modules'=>array(**
 **// uncomment the following to enable the Gii tool**
 **/***
 **'gii'=>array(**
 **'class'=>'system.gii.GiiModule',**
 **'password'=>'Enter Your Password Here',**
 **// If removed, Gii defaults to localhost only. Edit carefully to taste.**
 **'ipFilters'=>array('127.0.0.1','::1'),**
 **),**
 ***/**
 **),**

```

取消注释后，Gii 被配置为一个应用程序模块。我们将在本书后面详细介绍 Yii *模块*。此时的重要事情是确保将其添加到配置文件中，并提供您的密码。有了这个配置，通过`http://localhost/helloworld/index.php?r=gii`导航到工具。

### 注意

实际上，您可以将密码值指定为`false`，然后模块将不需要密码。由于 ipFilters 属性被指定为仅允许访问本地主机，因此在本地开发环境中将密码设置为`false`是安全的。

好的，在成功输入密码后（除非您指定不使用密码），您将看到列出 Gii 主要功能的菜单页面：

![配置 Gii](img/8727_02_03_new.jpg)

Gii 在左侧菜单中列出了几个代码生成选项。我们想要创建一个新的控制器，所以点击**控制器生成器**菜单项。

这样做将带我们到一个表单，允许我们填写相关细节以创建一个新的 Yii 控制器类。在下面的屏幕截图中，我们已经填写了**控制器 ID**值为`message`，并且我们添加了一个我们称之为`hello`的**Action ID**值。下面的屏幕截图还反映了我们已经点击了**预览**按钮。这显示了将与我们的控制器类一起生成的所有文件：

![配置 Gii](img/8727_02_04.jpg)

我们可以看到，除了我们的`MessageController`类之外，Gii 还将为我们指定的每个 Action ID 创建一个视图文件。您可能还记得第一章中提到的，如果`message`是**控制器 ID**，我们对应的类文件名为`MessageController`。同样，如果我们提供了`hello`的**Action ID**值，我们期望在控制器中有一个名为`actionHello`的方法。

您还可以单击**预览**选项中提供的链接，以查看将为每个文件生成的代码。继续并查看它们。一旦您对即将生成的内容感到满意，请点击**生成**按钮。您应该收到一条消息，告诉您控制器已成功创建，并附有立即尝试的链接。如果您收到错误消息，请确保`controllers/`和`views/`目录对您的 Web 服务器进程是可写的。

单击**立即尝试**链接实际上会将我们带到一个*404 页面未找到*错误页面。原因是我们在创建新控制器时没有指定默认的 actionID `index`。我们决定将我们的称为`hello`。为了使请求路由到我们的`actionHello()`方法，我们只需要将 actionID 添加到 URL 中。如下截图所示：

![配置 Gii](img/8727_02_05.jpg)

现在它显示了调用`MessageController::actionHello()`方法的结果。

这很棒。在 Gii 的帮助下，我们生成了一个名为`MessageController.php`的新控制器 PHP 文件，并将其正确放置在默认控制器目录`protected/controllers/`下。生成的`MessageController`类扩展了一个名为`Controller`的应用基类，位于`protected/components/Controller.php`中，而这个类又扩展了基础框架类`CController`。由于我们指定了 actionID `hello`，因此在`MessageController`中还创建了一个名为`actionHello()`的简单操作。Gii 还假定，像大多数由控制器定义的操作一样，此操作将需要呈现一个视图。因此，它添加了呈现同名视图文件`hello.php`的代码到此方法中，并将其放置在默认目录`protected/views/message/`中，用于与此控制器相关的视图文件。以下是为`MessageController`类生成的未注释部分代码：

```php
<?php
class MessageController extends Controller
{
        public function actionHello()
        {
                $this->render('hello');
        }
```

正如我们所看到的，由于我们在使用 Gii 创建此控制器时没有指定'index'作为 actionID 之一，因此没有`actionIndex()`方法。正如在第一章中讨论的那样，按照约定，指定控制器 ID 为消息，但未指定操作的请求将被路由到`actionIndex()`方法进行进一步处理。这就是为什么我们最初看到 404 错误的原因，因为请求没有指定 actionID。

让我们花点时间来修复这个问题。正如我们所提到的，Yii 更青睐于约定而不是配置，并且几乎所有内容都有合理的默认值。同时，几乎所有内容也是可配置的，默认控制器操作也不例外。通过在我们的`MessageController`顶部添加一行简单的代码，我们可以将`actionHello()`方法定义为默认操作。在`MessageController`类的顶部添加以下行：

```php
<?php

class MessageController extends Controller
{
 **public $defaultAction = 'hello';**

```

### 提示

**下载示例代码**

您可以从您在[`www.PacktPub.com`](http://www.PacktPub.com)购买的所有 Packt 图书的帐户中下载示例代码文件。如果您在其他地方购买了本书，您可以访问[`www.PacktPub.com/support`](http://www.PacktPub.com/support)注册并直接通过电子邮件接收文件。

尝试通过导航到`http://localhost/helloworld/index.php?r=message`来测试。您应该仍然看到显示`hello action`页面，不再看到错误页面。

## 最后一步

要将其转换为“Hello, World!”应用程序，我们只需要自定义我们的`hello.php`视图以显示“Hello, World!”。这样做很简单。编辑文件`protected/views/message/hello.php`，使其只包含以下代码：

```php
<?php
<h1>Hello World!</h1> 
```

保存它，并在浏览器中再次查看：`http://localhost/helloworld/index.php?r=message`。

现在它显示了我们的介绍性问候，如下截图所示：

![最后一步](img/8727_02_06.jpg)

我们的简单应用程序只需极少的代码就可以运行。我们只需在`hello.php`视图文件中添加了一行 HTML。

### 注意

您可能想知道所有其他 HTML 是如何/在哪里生成的。我们基本的`hello.php`视图文件只包含一个带有`<h1>`标签的单行。当我们在控制器中调用`render()`时，也会应用布局视图文件。现在不需要太担心这一点，因为我们将在以后更详细地介绍布局。但是如果您感兴趣，您可以查看`protected/views/layouts/`目录，看看已经定义的布局文件，并帮助您了解其他 HTML 的定义位置。

## 审查我们的请求路由

让我们回顾一下 Yii 如何在这个示例应用程序的上下文中分析我们的请求：

1.  通过将浏览器指向 URL `http://localhost/helloworld/index.php?r=message`（或者您可以使用等效的 URL `http://localhost/helloworld/index.php?r=message/hello`）来导航到“Hello, World!”页面。

1.  Yii 分析 URL。**r**（路由）查询字符串变量表示 controllerID 是`message`。这告诉 Yii 将请求路由到 message 控制器类，它在`protected/controllers/MessageController.php`中找到。

1.  Yii 还发现指定的 actionID 是`hello`。（或者如果没有指定 actionID，它会路由到控制器的默认动作。）因此，在`MessageController`中调用`actionHello()`方法。

1.  `actionHello()`方法呈现位于`protected/views/message/hello.php`的`hello.php`视图文件。我们修改了这个视图文件，只是简单地显示我们的问候语，然后返回给浏览器。

这一切都是非常轻松地组合在一起的。通过遵循 Yii 的默认约定，整个应用程序请求路由已经无缝地为我们拼接在一起。当然，Yii 给了我们每一个机会来覆盖这个默认的工作流程，但是你越是遵循约定，你就会花费越少的时间在调整配置代码上。

# 添加动态内容

向我们的视图模板添加动态内容的最简单方法是将 PHP 代码嵌入到模板本身中。视图文件由我们的简单应用程序呈现为 HTML，这些文件中的任何基本文本都会被传递而不会被更改。但是，任何在`<?php`和`?>`之间的内容都会被解释和执行为 PHP 代码。这是 PHP 代码嵌入 HTML 文件的典型方式，可能对您来说很熟悉。

## 添加日期和时间

为了给我们的页面增添动态内容，让我们显示日期和时间：

1.  再次打开 hello 视图，并在问候文本下面添加以下行：

```php
<h3><?php echo date("D M j G:i:s T Y"); ?></h3>
```

1.  保存和查看：`http://localhost/helloworld/index.php?r=message/hello`。

哎呀！我们已经在我们的应用程序中添加了动态内容。每次刷新页面，我们都可以看到显示的内容在变化。

诚然，这并不是非常令人兴奋，但它确实向您展示了如何将简单的 PHP 代码嵌入到我们的视图模板中。

## 添加日期和时间的另一种方法

尽管这种直接将 PHP 代码嵌入视图文件的方法允许任意数量或复杂度的 PHP 代码，但强烈建议这些语句不要改变数据模型，并保持简单、面向显示的语句。这将有助于将我们的业务逻辑与我们的呈现代码分开，这是使用 MVC 架构的好处之一。

### 将数据创建移到控制器

让我们将创建时间的逻辑移回到控制器，并且让视图什么都不做，只是显示时间。我们将时间的确定放到控制器中的`actionHello()`方法中，并在一个名为`$time`的实例变量中设置值。

首先让我们修改控制器动作。目前我们在`MessageController`中的动作`actionHello()`，只是通过执行以下代码来调用渲染我们的 hello 视图：

```php
$this->render('hello'); 
```

在我们渲染视图之前，让我们添加调用来确定时间，然后将其存储在一个名为`$theTime`的局部变量中。然后我们通过添加第二个参数来修改我们对`render()`的调用，其中包括这个变量：

```php
$theTime = date("D M j G:i:s T Y");
$this->render('hello',array('time'=>$theTime)); 
```

当调用`render()`并带有包含数组数据的第二个参数时，它将把数组的值提取到 PHP 变量中，并使这些变量可用于视图脚本。数组中的键将是可用于我们视图文件的变量的名称。因此，在这个例子中，我们的数组键'`time`'，其值是`$theTime`，将被提取到一个名为`$time`的变量中，并在视图中可用。这是一种从控制器传递数据到视图的方法。

### 注意

这假设您正在使用 Yii 的默认视图渲染器。正如之前多次提到的，Yii 允许您自定义几乎所有内容，如果您愿意，您可以指定不同的视图渲染实现。其他视图渲染可能不会以完全相同的方式行事。

现在让我们修改视图，使用这个`$time`变量而不是直接调用日期函数本身：

1.  再次打开 HelloWorld 视图文件，并用以下内容替换我们之前添加的用于输出时间的行：

```php
<h3><?php echo $time; ?></h3>
```

1.  再次保存并查看结果：`http://localhost/helloworld/index.php?r=message/hello`

我们再次看到时间显示与之前完全相同，因此两种方法的最终结果没有任何不同。

我们已经演示了向视图模板文件添加 PHP 生成内容的两种方法。第一种方法将数据创建逻辑直接放入视图文件本身。第二种方法将这个逻辑放在控制器类中，并通过使用变量将信息传递给视图文件。最终结果是相同的；时间显示在我们渲染的 HTML 中，但第二种方法在保持数据获取和处理（即业务逻辑）与我们的呈现代码分离方面迈出了一小步。这种分离正是模型-视图-控制器架构努力提供的，Yii 的显式目录结构和合理的默认值使其易于实现。

## 你有在关注吗？

在第一章中提到过，视图和控制器确实是非常相似的。在视图文件中，`$this`指的是渲染视图的`Controller`类。

在前面的例子中，我们通过在 render 方法中使用第二个参数，明确地从控制器向视图文件提供了时间。这第二个参数明确地设置了立即可用于视图文件的变量。但是还有另一种方法可以尝试一下。

通过在`MessageController`上定义一个公共类属性，而不是一个局部作用域的变量，其值是当前日期时间，来修改前面的例子。然后通过`$this`访问这个类属性，在视图文件中显示时间。

### 注意

可下载的代码库中包含了这个“自己动手”的练习的解决方案。

# 链接页面

典型的 Web 应用程序中有多个页面供用户体验，我们简单的应用程序也不例外。让我们添加另一个页面，显示来自世界的响应，“再见，Yii 开发者！”并从我们的“Hello, World!”页面链接到这个页面，反之亦然。

通常，在 Yii web 应用程序中，每个渲染的 HTML 页面都对应一个单独的视图（尽管这并不总是必须的）。因此，我们将创建一个新视图，并使用一个单独的操作方法来渲染这个视图。在添加新页面时，我们还需要考虑是否使用单独的控制器。由于我们的 Hello 和 Goodbye 页面是相关的并且非常相似，目前没有必要将应用程序逻辑委托给单独的控制器类。

## 链接到新页面

让我们的新页面的 URL 形式为`http://localhost/helloworld/index.php?r=message/goodbye`。

遵循 Yii 的约定，这个决定定义了我们的操作方法的名称，我们需要在控制器中使用，以及我们的视图的名称。因此，打开`MessageController`并在我们的`actionHello()`操作的下面添加一个`actionGoodbye()`方法：

```php
class MessageController extends Controller
{
  ...

  public function actionGoodbye()
  {
    $this->render('goodbye');
  }

    ...
}
```

接下来，我们需要在`/protected/views/message/`目录中创建我们的视图文件。这应该被称为`goodbye.php`，因为它应该与我们选择的 actionID 相同。

### 注意

请记住，这只是一个推荐的约定。视图不一定必须与操作具有相同的名称。视图文件名只需与`render()`的第一个参数匹配即可。

在该目录中创建一个空文件，并添加一行：

```php
<h1>Goodbye, Yii developer!</h1>      
```

再次保存和查看`http://localhost/helloworld/index.php?r=message/goodbye`将显示再见消息。

现在我们需要添加链接来连接这两个页面。要在 Hello 页面上添加到 Goodbye 页面的链接，我们可以直接在`hello.php`视图文件中添加`<a>`标签，并硬编码 URL 结构如下：

```php
<a href="/helloworld/index.php?r=message/goodbye">Goodbye!</a>
```

这样做可以，但它将视图代码实现紧密耦合到特定的 URL 结构，这可能在某个时候发生变化。如果 URL 结构发生变化，这些链接将变得无效。

### 注意

还记得在第一章 *遇见 Yii*中，我们通过博客发布应用程序示例吗？我们使用的 URL 格式与 Yii 默认格式不同，更符合 SEO，即：

`http://yourhostname/controllerID/actionID`

将 Yii Web 应用程序配置为使用这种“路径”格式而不是我们在此示例中使用的查询字符串格式是一件简单的事情。能够轻松更改 URL 格式对 Web 应用程序非常重要。只要我们避免在整个应用程序中硬编码它们，更改它们将保持简单，只需更改应用程序配置文件即可。

## 从 Yii CHtml 获得一点帮助

幸运的是，Yii 在这里提供了帮助。Yii 带有许多可以在视图模板中使用的辅助方法。这些方法存在于静态 HTML 辅助框架类`CHtml`中。在这种情况下，我们想要使用的是“link”辅助方法，它接受一个*controllerID/actionID*对，并根据应用程序配置的 URL 结构为您创建适当的超链接。由于所有这些辅助方法都是静态的，我们可以直接调用它们，而无需创建`CHtml`类的显式实例。使用这个链接助手，我们可以在我们的`hello.php`视图中在我们输出时间的下面添加一个链接，如下所示：

```php
<p><?php echo CHtml::link('Goodbye'array('message/goodbye')); ?></p>  
```

保存并查看“Hello, World!”页面：`http://localhost/helloworld/index.php?r=message/hello`

您应该看到超链接，并单击它应该将您带到再见页面。调用`link`方法的第一个参数是将显示在超链接中的文本。第二个参数是一个包含我们的*controllerID*/*actionID*对值的数组。

我们可以采用相同的方法在我们的 Goodbye 视图中放置一个相互链接：

```php
<h1>Goodbye, Yii developer!</h1>      
<p><?php echo CHtml::link('Hello',array('message/hello')); ?></p>  
```

保存并查看再见页面：

`http://localhost/helloworld/index.php?r=message/goodbye`

现在，您应该看到从再见页面返回到“Hello, World!”页面的活动链接。

所以我们现在知道了在我们的简单应用程序中链接网页的几种方法。一种方法是直接在视图文件中添加 HTML `<a>`标签，并硬编码 URL 结构。另一种更常用的方法是利用 Yii 的`CHtml`辅助类来帮助构建基于*controllerID* */actionID*对的 URL，以便结果格式始终符合应用程序配置。通过这种方式，我们可以轻松地在整个应用程序中更改 URL 格式，而无需返回更改每个视图文件，这些文件恰好具有内部链接。

我们简单的“Hello, World!”应用程序真正受益于 Yii 的约定优于配置的理念。通过应用某些默认行为并遵循推荐的约定，这个简单应用程序的构建和整个请求路由过程都以非常简单和方便的方式完成了。

# 总结

在本章中，我们构建了一个极其简单的应用程序，以涵盖许多主题。首先我们安装了框架。然后我们使用`yiic`控制台命令来引导创建一个新的 Yii 应用程序。然后我们介绍了一个非常强大的代码生成工具叫做 Gii。我们使用它在我们的简单应用程序中创建了一个新的控制器。

一旦我们的应用程序就位，我们就可以亲自看到 Yii 如何处理请求和路由到控制器和动作。然后，我们继续创建和显示非常简单的动态内容。最后，我们看了一下如何在 Yii 应用程序中链接页面。

虽然这个非常简单的应用程序为我们提供了具体的例子，帮助我们更好地理解 Yii 框架的使用，但它过于简单，无法展示 Yii 在简化实际应用程序构建方面的能力。为了证明这一点，我们需要构建一个真实的 Web 应用程序。我们将会这样做。在下一章中，我们将向您介绍项目任务和问题跟踪应用程序，我们将在本书的其余部分中构建该应用程序。
