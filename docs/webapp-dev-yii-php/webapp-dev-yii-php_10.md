# 第十章。让它看起来不错

在上一章中，我们通过使我们的 URL 对用户和搜索引擎爬虫更具吸引力，为我们的应用程序增添了一些美感。在本章中，我们将更多地关注我们应用程序的外观和感觉，涵盖 Yii 中布局和主题的主题。我们将专注于一个人采取的方法和可用的工具，以帮助设计 Yii 应用程序的前端，而不是设计本身。因此，本章将更多地关注如何使您的应用程序看起来不错，而不是花费大量时间专门设计我们的 TrackStar 应用程序以实际看起来不错。

# 功能规划

本章旨在专注于前端。我们希望为我们的网站创建一个可重用且能够动态实现的新外观。我们还希望在不覆盖或删除当前设计的情况下实现这一点。最后，我们将深入研究 Yii 的国际化功能，以更好地了解如何适应来自不同地理区域的用户。

以下是我们需要完成的高级任务列表，以实现这些目标：

+   通过创建新的布局、CSS 和其他资产文件来为我们的应用程序创建一个新的前端设计

+   使用 Yii 的国际化和本地化功能来帮助将应用程序的一部分翻译成新语言

# 使用布局进行设计

您可能已经注意到的一件事是，我们在不添加任何显式导航以访问此功能的情况下向我们的应用程序添加了大量功能。我们的主页尚未从我们构建的默认应用程序更改。我们的新应用程序创建时的导航项与我们创建新应用程序时的导航项相同。我们需要更改我们的基本导航，以更好地反映应用程序中存在的基本功能。

到目前为止，我们尚未完全涵盖我们的应用程序如何使用负责显示内容的所有视图文件。我们知道我们的视图文件负责显示我们的数据和承载响应每个页面请求的返回的 HTML。当我们创建新的控制器操作时，我们经常创建新的视图来处理这些操作方法返回的内容的显示。这些视图中的大多数都非常特定于它们支持的操作方法，并且不会跨多个页面使用。但是，有一些东西，例如主菜单导航，可以在整个站点的多个页面上使用。这些类型的 UI 组件更适合驻留在所谓的布局文件中。

Yii 中的**布局**是用于装饰其他视图文件的特殊视图文件。布局通常包含跨多个视图文件共同的标记或其他用户界面组件。当使用布局来呈现视图文件时，Yii 会将视图文件嵌入布局中。

## 指定布局

可以指定布局的两个主要位置。一个是`CWebApplication`本身的`$layout`属性。这默认为`protected/views/layouts/main.php`。与所有应用程序设置一样，这可以在主配置文件`protected/config/main.php`中被覆盖。例如，如果我们创建了一个新的布局文件`protected/views/layouts/newlayout.php`，并希望将此新文件用作我们的应用程序范围的布局文件，我们可以修改我们的主`config.php`文件来设置布局属性如下：

```php
return array(
  ...
  'layout'=>'newlayout',
```

文件名不带`.php`扩展名，并且相对于`CWebApplication`的`$layoutPath`属性指定，该属性默认为`Webroot/protected/views/layouts`（如果此位置不适合您的应用程序需求，则可以类似地覆盖它）。

另一个指定布局的地方是通过设置控制器类的`$layout`属性。这允许更细粒度地控制每个控制器的布局。这是在生成初始应用程序时指定的方式。使用`yiic`工具创建我们的初始应用程序时，自动创建了一个控制器基类`Webroot/protected/components/Controller.php`，所有其他控制器类都是从这个类继承的。打开这个文件会发现`$layout`属性已经设置为`column1`。在更细粒度的控制器级别设置布局文件将覆盖`CWebApplication`类中的设置。

## 应用和使用布局

在调用`CController::render()`方法时，布局文件的使用是隐含的。也就是说，当您调用`render()`方法来渲染一个视图文件时，Yii 将把视图文件的内容嵌入到控制器类中指定的布局文件中，或者嵌入到应用程序级别指定的布局文件中。您可以通过调用`CController::renderPartial()`方法来避免对渲染的视图文件应用任何布局装饰。

如前所述，布局文件通常用于装饰其他视图文件。布局的一个示例用途是为每个页面提供一致的页眉和页脚布局。当调用`render()`方法时，幕后发生的是首先将调用发送到指定视图文件的`renderPartial()`。这个输出存储在一个名为`$content`的变量中，然后可以在布局文件中使用。因此，一个非常简单的布局文件可能如下所示：

```php
<!DOCTYPE html>
<html>
<head>
<title>Title of the document</title>
</head>
<body>
  <div id="header">
    Some Header Content Here
  </div>

  <div id="content">
    <?php echo $content; ?>
  </div>

  <div id="footer">
      Some Footer Content Here
  </div>
</body>
</html>
```

实际上让我们试一试。创建一个名为`newlayout.php`的新文件，并将其放在布局文件的默认目录`/protected/views/layouts/`中。将前面的 HTML 内容添加到此文件中并保存。现在我们将通过修改我们的站点控制器来使用这个新布局。打开`SiteController.php`并通过在这个类中显式添加它来覆盖基类中设置的布局属性，如下所示：

```php
class SiteController extends Controller
{

  public $layout='newlayout';
```

这将把布局文件设置为`newlayout.php`，但仅适用于这个控制器。现在，每当我们在`SiteController`中调用`render()`方法时，将使用`newlayout.php`布局文件。

`SiteController`负责渲染的一个页面是登录页面。让我们来看看该页面，以验证这些更改。如果我们导航到`http://localhost/trackstar/site/login`（假设我们还没有登录），我们现在看到类似以下截图的东西：

![应用和使用布局](img/8727_10_01.jpg)

如果我们简单地注释掉我们刚刚添加的`$layout`属性并再次刷新登录页面，我们将回到使用原始的`main.php`布局，并且我们的页面现在将恢复到之前的样子。

# 解构 main.php 布局文件

到目前为止，我们的应用程序页面都使用`main.php`布局文件来提供主要的布局标记。在开始对我们的页面布局和设计进行更改之前，最好先仔细查看一下这个主要布局文件。您可以从本章的可下载代码中完整查看它，或者在[`gist.github.com/3781042`](https://gist.github.com/3781042)上查看独立文件。

第一行到第五行可能会让你觉得有些熟悉：

```php
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html  xml:lang="en" lang="en">
<head>
  <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
  <meta name="language" content="en" />
```

这些行定义了一个标准的 HTML 文档类型声明，后面是一个开始的`<html>`元素，然后是我们的`<head>`元素的开始。在`<head>`标记内，我们首先有一个`<meta>`标记来声明标准的`XHTML-compliant uft-8`字符编码，然后是另一个`<meta>`标记，指定`English`作为网站编写的主要语言。

## 介绍 Blueprint CSS 框架

以下几行以注释`<!—blueprint CSS framework -->`开头，可能对您来说不太熟悉。Yii 的另一个很棒的地方是，在适当的时候，它利用其他最佳框架，Blueprint CSS 框架就是一个例子。

Blueprint CSS 框架是在我们最初创建应用程序时使用`yiic`工具时作为副产品包含在应用程序中的。它包含在内是为了帮助标准化 CSS 开发。Blueprint 是一个 CSS 网格框架。它有助于标准化您的 CSS，提供跨浏览器兼容性，并在 HTML 元素放置方面提供一致性，有助于减少 CSS 错误。它提供了许多屏幕和打印友好的布局定义，并通过提供您所需的所有 CSS 来快速启动设计，使您的设计看起来不错并且位置正确。有关 Blueprint 框架的更多信息，请访问[`www.blueprintcss.org/`](http://www.blueprintcss.org/)。

因此，以下代码行是 Blueprint CSS 框架所必需的和特定的：

```php
<!-- blueprint CSS framework -->
<link rel="stylesheet" type="text/css" href="<?php echo Yii::app()->request->baseUrl; ?>/css/screen.css" media="screen, projection" />
<link rel="stylesheet" type="text/css" href="<?php echo Yii::app()->request->baseUrl; ?>/css/print.css" media="print" />
<!--[if lt IE 8]>
<link rel="stylesheet" type="text/css" href="<?php echo Yii::app()->request->baseUrl; ?>/css/ie.css" media="screen, projection" />
<![endif]-->
```

调用`Yii::app()->request->baseUrl;`在这里用于获取应用程序的相对 URL。

### 了解 Blueprint 安装

Yii 绝不要求使用 Blueprint。但是，由于默认应用程序生成包括该框架，了解其安装和使用将是有益的。

Blueprint 的典型安装首先涉及下载框架文件，然后将其三个`.css`文件放入 Yii 应用程序的主`css`目录中。如果我们在 TrackStar 应用程序的主`Webroot/css`目录下查看，我们已经看到包含了这三个文件：

+   `ie.css`

+   `print.css`

+   `screen.css`

所以幸运的是，基本安装已经完成。为了利用该框架，先前的`<link>`标签需要放置在每个网页的`<head>`标签下。这就是为什么这些声明是在布局文件中进行的。

接下来的两个`<link>`标签如下：

```php
<link rel="stylesheet" type="text/css" href="<?php echo Yii::app()->request->baseUrl; ?>/css/main.css" />
<link rel="stylesheet" type="text/css" href="<?php echo Yii::app()->request->baseUrl; ?>/css/form.css" />
```

这些`<link>`标签定义了一些自定义的`css`定义，用于提供布局声明，除了 Blueprint 文件中指定的声明之外。您应该始终将任何自定义定义放在 Blueprint 提供的定义下面，以便您的自定义声明优先。

## 设置页面标题

根据每个页面设置特定且有意义的页面标题对于搜索引擎索引您网站页面和希望将您网站特定页面加为书签的用户来说非常重要。我们主要布局文件中的下一行指定了浏览器中的页面标题：

```php
<title><?php echo CHtml::encode($this->pageTitle); ?></title>
```

请记住，在视图文件中，`$this`指的是最初呈现视图的控制器类实例。`$pageTitle`属性在 Yii 的`CController`基类中定义，并将默认为动作名称，后跟控制器名称。这在特定控制器类中甚至在每个特定视图文件中都可以轻松自定义。

## 定义页面页眉

通常情况下，网站被设计为在许多页面上重复具有一致的页眉内容。我们主要布局文件中的接下来几行定义了页面页眉的区域：

```php
<body>
<div class="container" id="page">

  <div id="header">
    <div id="logo"><?php echo CHtml::encode(Yii::app()->name); ?></div>
  </div><!-- header -->
```

第一个带有`container`类的`<div>`标签是 Blueprint 框架所必需的，以便将内容显示为网格。

### 注意

再次，使用 Blueprint CSS Grid 框架或任何其他 CSS 框架并不是 Yii 的要求。它只是为了帮助您在需要时快速启动设计布局。

接下来的三行布置了我们在这些页面上看到的主要内容的第一部分。它们显示了应用程序的名称。到目前为止，它一直显示文本**My Web Application**。我相信这让你们中的一些人感到疯狂。尽管我们以后可能会更改为使用标志图像，但让我们继续将其更改为我们应用程序的真实名称**TrackStar**。

我们可以在 HTML 中直接硬编码这个名称。然而，如果我们修改应用程序配置以反映我们的新名称，这些更改将在整个网站的任何地方传播，无论`Yii::app()->name`在哪里使用。我相信你现在可以轻松地在睡梦中做出这个简单的改变。只需打开主`config.php`文件`/protected/config/main.php`，在那里我们定义了应用程序配置设置，并将`name`属性的值从`'name'=>'My Web Application'`更改为新值`'name'=>'TrackStar'`。

保存文件，刷新浏览器，主页的标题现在应该看起来类似于以下截图：

![定义页面标题](img/8727_10_02.jpg)

我们立即注意到在上一个截图中已经在两个地方进行了更改。恰好我们的主页内容的视图文件`/protected/views/site/index.php`也使用了应用程序名称属性。由于我们在应用程序配置文件中进行了更改，我们的更改在两个地方都得到了反映。

由于名称属性是您可能决定在某个时候更改的内容，因此也定义应用程序`id`属性是一个好习惯。这个属性被框架用来创建唯一的签名键作为访问会话变量、缓存数据和其他令牌的前缀。如果没有指定`id`属性，则将使用`name`属性。因此更改它可能会使这些数据无效。让我们也为我们的应用程序定义一个`id`属性。这是添加到`protected/config/main.php`中的，就像我们为`name`属性所做的那样。我们可以使用与我们的名称相同的值：

```php
'id'=>'TrackStar',
```

## 显示菜单导航项

主站点的导航控件通常在 Web 应用程序的多个页面上重复出现，并且将其放在布局中使得重复使用非常容易。我们主要布局文件中的下一个标记和代码块定义了顶级菜单项：

```php
<div id="mainmenu">
  <?php $this->widget('zii.widgets.CMenu',array(
    'items'=>array(
      array('label'=>'Home', 'url'=>array('/site/index')),
      array('label'=>'About', 'url'=>array('/site/page', 'view'=>'about')),
      array('label'=>'Contact', 'url'=>array('/site/contact')),
      array('label'=>'Login', 'url'=>array('/site/login'), 'visible'=>Yii::app()->user->isGuest),
      array('label'=>'Logout ('.Yii::app()->user->name.')', 'url'=>array('/site/logout'), 'visible'=>!Yii::app()->user->isGuest)
    ),
  )); ?>
</div><!-- mainmenu -->
```

在这里，我们看到 Zii 组件之一称为`CMenu`正在被使用。我们在第八章中介绍了 Zii，*添加用户评论*。为了唤起你的记忆，Zii 扩展库是 Yii 开发团队开发的一组扩展。这个库与核心 Yii 框架一起打包。任何这些扩展都可以在 Yii 应用程序中轻松使用，只需通过使用路径别名引用所需的扩展类文件，形式为`zii.path.to.ClassName`。根别名`zii`由应用程序预定义，其余路径相对于这个框架目录。由于这个 Zii 菜单扩展位于您的文件系统上的`YiiRoot/zii/widgets/CMenu.php`，所以我们可以在应用程序代码中简单地使用`zii.widgets.CMenu`来引用它。

`CMenu`接受一个提供菜单项的关联数组。每个项目数组包括一个将要显示的`label`，一个该项目应链接到的 URL，以及一个可选的第三个值`visible`，它是一个`boolean`值，指示是否应该显示该菜单项。在这里，当定义**登录**和**注销**菜单项时使用了这个。我们只希望**登录**菜单项在用户尚未登录时显示为可点击链接。反之，我们只希望**注销**菜单链接在用户已经登录时显示。数组中的 visible 元素的使用允许我们根据用户是否已登录动态显示这些链接。使用`Yii::app()->user->isGuest`是为了这个目的。如果用户未登录，则返回`true`，如果用户已登录，则返回`false`。我相信你已经注意到，**登录**选项在您登录时会变成应用程序主菜单中的**注销**选项，反之亦然。

让我们更新我们的菜单，为用户提供导航到我们特定的 TrackStar 功能的方法。首先，我们不希望匿名用户能够访问任何真正的功能，除了登录。因此，我们需要确保登录页面更多或更少地成为匿名用户的主页。此外，已登录用户的主页应该只是他们项目的列表。我们将通过进行以下更改来实现这一点：

1.  将我们应用程序的默认主页 URL 更改为项目列表页面，而不仅仅是`site/index`。

1.  将默认控制器`SiteController`中的默认操作更改为登录操作。这样，任何访问顶级 URL `http://localhost/trackstar/` 的匿名用户都将被重定向到登录页面。

1.  修改我们的`actionLogin()`方法，如果用户已经登录，则将用户重定向到项目列表页面。

1.  将**主页**菜单项更改为**项目**，并将 URL 更改为项目列表页面。

这些都是我们需要做出的简单更改。从顶部开始，我们可以在主应用程序`config.php`文件中更改主页 URL 应用程序属性。打开`protected/config/main.php`并将以下`name=>value`对添加到返回的数组中：

```php
'homeUrl'=>'/trackstar/project',
```

这就是需要做出的所有更改。

对于下一个更改，打开`protected/controllers/SiteController.php`并将以下内容添加到控制器类的顶部：

```php
public $defaultAction = 'login';
```

这将默认操作设置为登录。现在，如果您访问应用程序的顶级 URL `http://localhost/trackstar/`，您应该被带到登录页面。唯一的问题是，无论您是否已经登录，您都将继续从这个顶级 URL 被带到登录页面。让我们通过实施上一个列表的第 3 步来解决这个问题。在`SiteController`中的`actionLogin()`方法中添加以下代码：

```php
public function actionLogin()
{

  if(!Yii::app()->user->isGuest) 
     {
          $this->redirect(Yii::app()->homeUrl);
     }
```

这将把所有已登录用户重定向到应用程序的`homeUrl`，我们刚刚将其设置为项目列表页面。

最后，让我们修改`CMenu`小部件的输入数组，以更改**主页**菜单项的规范。在`main.php`布局文件中更改该代码块，并用以下内容替换`array('label'=>'Home', 'url'=>array('/site/index')),`这一行：

```php
array('label'=>'Projects', 'url'=>array('/project')),
```

通过这个替换，我们之前概述的所有更改都已经就位。现在，如果我们以匿名用户身份访问 TrackStar 应用程序，我们将被引导到登录页面。如果我们点击**项目**链接，我们仍然会被引导到登录页面。我们仍然可以访问**关于**和**联系**页面，这对于匿名用户来说是可以的。如果我们登录，我们将被引导到项目列表页面。现在，如果我们点击**项目**链接，我们将被允许查看项目列表。

## 创建面包屑导航

回到我们的`main.php`布局文件，跟随菜单小部件之后的三行代码定义了另一个 Zii 扩展小部件，称为`CBreadcrumbs`：

```php
<?php $this->widget('zii.widgets.CBreadcrumbs', array(
  'links'=>$this->breadcrumbs,
)); ?><!-- breadcrumbs -->
```

这是另一个 Zii 小部件，可用于显示指示当前页面位置的链接列表，相对于整个网站中的其他页面。例如，格式为**项目 >> 项目 1 >> 编辑**的链接导航列表表示用户正在查看项目 1 的编辑页面。这对用户找回起点（即所有项目的列表）以及轻松查看他们在网站页面层次结构中的位置非常有帮助。这就是为什么它被称为**面包屑**。许多网站在其设计中实现了这种类型的 UI 导航组件。

要使用此小部件，我们需要配置其`links`属性，该属性指定要显示的链接。此属性的预期值是定义从起始点到正在查看的特定页面的`面包屑`路径的数组。使用我们之前的示例，我们可以将`links`数组指定如下：

```php
array(
  'Projects'=>array('project/index'),
  'Project 1'=>array('project/view','id'=>1),
  'Edit',
  )
```

`breadcrumbs`小部件默认情况下会根据应用程序配置设置`homeUrl`自动添加顶级**主页**链接。因此，从前面的代码片段生成的面包屑将如下所示：

**主页 >> 项目 >> 项目 1 >> 编辑**

由于我们明确将应用程序的`$homeUrl`属性设置为项目列表页面，所以在这种情况下我们的前两个链接是相同的。布局文件中的代码将链接属性设置为呈现视图的控制器类的`$breadcrumbs`属性。您可以在使用 Gii 代码生成工具创建控制器文件时为我们自动生成的几个视图文件中明确看到这一点。例如，如果您查看`protected/views/project/update.php`，您将在该文件的顶部看到以下代码片段：

```php
$this->breadcrumbs=array(
  'Projects'=>array('index'),
  $model->name=>array('view','id'=>$model->id),
  'Update',
);
```

如果我们在网站上导航到该页面，我们将看到主导航栏下方生成的以下导航面包屑：

![创建面包屑导航](img/8727_10_03.jpg)

## 指定被布局装饰的内容

布局文件中的下一行显示了被该布局文件装饰的视图文件的内容放置位置：

```php
<?php echo $content; ?>
```

这在本章的前面已经讨论过。当您在控制器类中使用`$this->render()`来显示特定的视图文件时，隐含了使用布局文件。这个方法的一部分是将呈现的特定视图文件中的所有内容放入一个名为`$content`的特殊变量中，然后将其提供给布局文件。因此，如果我们再次以项目更新视图文件为例，`$content`的内容将是包含在文件`protected/views/project/update.php`中的呈现内容。

## 定义页脚

与*页眉*区域一样，通常情况下网站被设计为在许多页面上重复显示一致的*页脚*内容。我们的`main.php`布局文件的最后几行定义了每个页面的一致`页脚`：

```php
<div id="footer">
    Copyright &copy; <?php echo date('Y'); ?> by My Company.<br/>
    All Rights Reserved.<br/>
    <?php echo Yii::powered(); ?>
</div><!-- footer -->
```

这里没有什么特别的，但我们应该继续更新它以反映我们特定的网站。我们可以将前面的代码片段中的`My Company`简单地更改为`TrackStar`，然后完成。刷新网站中的页面现在将显示我们的页脚，如下面的截图所示：

![定义页脚](img/8727_10_04.jpg)

# 嵌套布局

尽管我们在页面上看到的原始布局确实使用了文件`protected/layouts/main.php`，但这并不是全部。当我们的初始应用程序创建时，所有控制器都被创建为扩展自位于`protected/components/Controller.php`的基础控制器。如果我们偷看一下这个文件，我们会看到布局属性被明确定义。但它并没有指定主布局文件。相反，它将`column1`指定为所有子类的默认布局文件。您可能已经注意到，当新应用程序创建时，还为我们生成了一些布局文件，全部位于`protected/views/layouts/`目录中：

+   `column1.php`

+   `column2.php`

+   `main.php`

因此，除非在子类中明确覆盖，否则我们的控制器将`column1.php`定义为主要布局文件，而不是`main.php`。

你可能会问，为什么我们要花那么多时间去了解`main.php`呢？嗯，事实证明，`column1.php`布局文件本身也被`main.php`布局文件装饰。因此，不仅可以通过布局文件装饰普通视图文件，而且布局文件本身也可以被其他布局文件装饰，形成嵌套布局文件的层次结构。这样可以极大地提高设计的灵活性，也极大地减少了视图文件中的重复标记的需要。让我们更仔细地看看`column1.php`，看看是如何实现这一点的。

该文件的内容如下：

```php
<?php $this->beginContent('//layouts/main'); ?>
<div id="content">
  <?php echo $content; ?>
</div><!-- content -->
<?php $this->endContent(); ?>
```

在这里，我们看到了一些以前没有见过的方法的使用。基本控制器方法`beginContent()`和`endContent()`被用来用指定的视图装饰封闭的内容。这里指定的视图是我们的主布局页面`'//layouts/main'`。`beginContent()`方法实际上使用了内置的 Yii 小部件`CContentDecorator`，其主要目的是允许嵌套布局。因此，`beginContent()`和`endContent()`之间的任何内容都将使用在`beginContent()`调用中指定的视图进行装饰。如果未指定任何内容，它将使用在控制器级别指定的默认布局，或者如果在控制器级别未指定，则使用应用程序级别的默认布局。

### 注意

在前面的代码片段中，我们看到视图文件被双斜杠`'//'`指定。在这种情况下，将在应用程序的视图路径下搜索视图，而不是在当前活动模块的视图路径下搜索。这迫使它使用主应用程序视图路径，而不是模块的视图路径。模块是下一章的主题。

其余部分就像普通的布局文件一样。当呈现此`column1.php`布局文件时，特定视图文件中的所有标记都将包含在变量`$content`中，然后此布局文件中包含的其他标记将再次包含在变量`$content`中，以供最终呈现主父布局文件`main.php`使用。

让我们通过一个示例来走一遍。以登录视图的呈现为例，即`SiteController::actionLogin()`方法中的以下代码：

```php
$this->render('login');
```

在幕后，正在执行以下步骤：

1.  呈现特定视图文件`/protected/views/site/login.php`中的所有内容，并通过变量`$content`将该内容提供给控制器中指定的布局文件，在这种情况下是`column1.php`。

1.  由于`column1.php`本身被布局`main.php`装饰，所以在`beingContent()`和`endContent()`调用之间的内容再次被呈现，并通过`$content`变量再次提供给`main.php`文件。

1.  布局文件`main.php`被呈现并返回给用户，包含了登录页面的特定视图文件的内容以及“嵌套”布局文件`column1.php`的内容。

当我们最初创建应用程序时，自动生成的另一个布局文件是`column2.php`。您可能不会感到惊讶地发现，该文件布局了一个两列设计。我们可以在项目页面中看到这个布局的使用，其中右侧显示了一个小子菜单**操作**小部件。该布局的内容如下，我们可以看到也使用了相同的方法来实现嵌套布局。

```php
<?php $this->beginContent('//layouts/main'); ?>
<div class="span-19">
  <div id="content">
    <?php echo $content; ?>
  </div><!-- content -->
</div>
<div class="span-5 last">
  <div id="sidebar">
  <?php
    $this->beginWidget('zii.widgets.CPortlet', array(
      'title'=>'Operations',
    ));
    $this->widget('zii.widgets.CMenu', array(
      'items'=>$this->menu,
      'htmlOptions'=>array('class'=>'operations'),
    ));
    $this->endWidget();
  ?>
  </div><!-- sidebar -->
</div>
<?php $this->endContent(); ?>
```

# 创建主题

主题提供了一种系统化的方式来定制 Web 应用程序的设计布局。 MVC 架构的许多好处之一是将演示与其他“后端”内容分离。主题通过允许您在运行时轻松而显着地改变 Web 应用程序的整体外观和感觉，充分利用了这种分离。 Yii 允许极其简单地应用主题，以提供 Web 应用程序设计的更大灵活性。

## 在 Yii 中构建主题

在 Yii 中，每个主题都表示为一个目录，包含视图文件、布局文件和相关资源文件，如图像、CSS 文件和 JavaScript 文件。主题的名称与其目录名称相同。默认情况下，所有主题都位于相同的`WebRoot/themes`目录下。当然，与所有其他应用程序设置一样，可以配置默认目录为其他目录。要这样做，只需修改`themeManager`应用程序组件的`basePath`属性和`baseUrl`属性。

主题目录下的内容应该以与应用程序基本路径下相同的方式进行组织。因此，所有视图文件都位于`views/`目录下，布局视图文件位于`views/layouts/`下，系统视图文件位于`views/system/`下。例如，如果我们创建了一个名为`custom`的新主题，并且想要用这个主题下的新视图替换`ProjectController`的更新视图，我们需要创建一个新的`update.php`视图文件，并将其保存在我们的应用项目中，路径为`themes/custom/views/project/update.php`。

## 创建主题

让我们试试看，给我们的 TrackStar 应用程序做一点小改变。我们需要给我们的新主题命名，并在`Webroot/themes`目录下创建一个同名的目录。我们将发挥我们的极端创造力，将我们的新主题命名为`newtheme`。

在`Webroot/themes/newtheme`位置创建一个新目录来保存这个新主题。然后在这个新创建的目录下，创建另外两个新目录，分别叫做`css/`和`views/`。前者不是主题系统所必需的，但有助于我们组织 CSS。后者是必需的，如果我们要对默认视图文件进行任何修改，而我们是要修改的。因为我们要稍微改变`main.php`布局文件，所以在这个新创建的`views/`目录下需要再创建一个名为`layouts/`的目录（记住目录结构需要与默认的`Webroot/protected/views/`目录中的相同）。

现在让我们做一些改变。由于我们的视图文件标记已经引用了`Webroot/css/main.css`文件中当前定义的`css`类和`id`名称，所以最快的路径到应用程序的新外观是以此为起点，并根据需要进行更改。当然，这不是必需的，因为我们可以在新主题中重新创建应用程序的每个视图文件。但是为了保持简单，我们将通过对为我们创建应用程序时自动生成的`main.css`文件以及主要布局文件`main.php`进行一些更改来创建我们的新主题。

首先，让我们复制这两个文件并将它们放在我们的新主题目录中。将文件`Webroot/css/main.css`复制到新位置`Webroot/themes/newtheme/css/main.css`，并将文件`Webroot/protected/views/layouts/main.php`复制到新位置`Webroot/themes/newtheme/views/layouts/main.php`。

现在我们可以打开新复制的`main.css`文件，删除内容，并添加必要的样式来创建我们的新主题。为了我们的示例，我们将使用本章可下载代码中提供的 CSS，或者在[`gist.github.com/3779729`](https://gist.github.com/3779729)上提供的独立文件。

您可能已经注意到，一些更改引用了我们项目中尚不存在的图像文件。我们在 body 声明中添加了一个`images/background.gif`图像引用，`#mainmenu` ID 声明中引用了一个新的`images/bg2.gif`图像，以及`#header` ID 声明中引用了一个新的`images/header.jpg`图像。这些都可以在可下载的源代码中找到。我们将把这些新图像放在`css/`目录中的一个图像目录中，即`Webroot/themes/newtheme/css/images/`。

这些更改生效后，我们需要对新主题中的`main.php`布局文件进行一些小的调整。首先，我们需要修改`<head>`元素中的标记，以正确引用我们的新`main.css`文件。目前，`main.css`文件是通过以下行引入的：

```php
<link rel="stylesheet" type="text/css" href="<?php echo Yii::app()->request->baseUrl; ?>/css/main.css" />
```

这引用了应用程序请求的`baseUrl`属性来构建到 CSS 文件的相对路径。然而，我们想要使用我们新主题中的`main.css`文件。为此，我们可以依靠主题管理器应用程序组件，默认定义使用 Yii 内置的`CThemeManager.php`类。我们访问主题管理器的方式与访问其他应用程序组件的方式相同。因此，我们应该使用主题管理器定义的基本 URL，它知道应用程序在任何给定时间使用的主题。修改前面提到的`/themes/newtheme/views/layouts/main.php`中的代码如下：

```php
<link rel="stylesheet" type="text/css" href="<?php echo Yii::app()->theme->baseUrl; ?>/css/main.css" />
```

一旦我们配置应用程序使用我们的新主题（这是我们尚未完成的），这个`baseUrl`将解析为我们的主题目录所在的相对路径。

我们需要做的另一个小改变是从头部中移除应用程序标题的显示。由于我们修改了 CSS 以使用新的图像文件来提供我们的头部和标志信息，我们不需要在这个部分显示应用程序名称。因此，在`/themes/newtheme/views/layouts/main.php`中，我们只需要改变以下代码：

```php
<div id="header">
  <div id="logo"><?php echo CHtml::encode(Yii::app()->name); ?></div>
</div><!-- header -->
```

将上述代码修改如下：

```php
<div id="header"></div><!-- header image is embedded into the #header declaration in main.css -->
```

我们已经放置了一个注释来提醒我们头部图像的定义位置。

现在一旦我们配置应用程序使用我们的新主题，它将首先在主题目录中查找`main.php`布局文件，如果存在的话就使用该文件。

## 配置应用程序使用主题

好的，有了我们现在创建并放置好的`newtheme`主题，我们需要告诉应用程序使用这个主题。这样做非常容易。只需通过改变主应用程序配置文件来修改主应用程序的`theme`属性设置。到目前为止，我们已经成为了这样做的老手。只需在`/protected/config/main.php`文件中的返回数组中添加以下`name=>value`对：

```php
'theme'=>'newtheme',
```

一旦保存了这个更改，我们的应用程序现在使用我们新创建的主题，并且有了全新的外观。当我们查看登录页面时，也就是我们的默认主页（如果没有登录），我们现在看到了以下截图中所示的内容：

![配置应用程序使用主题](img/8727_10_05.jpg)

当然，这并不是一个巨大的改变。我们保持了改动相当小，但它们确实展示了创建新主题的过程。应用程序首先会在这个新主题中查找视图文件，如果存在的话就使用它们，否则会从默认位置获取。你可以看到给应用程序赋予新的外观和感觉是多么容易。你可以为每个季节或基于不同的心情创建一个新主题，然后根据需要快速轻松地改变应用程序以适应季节或心情。

# 将网站翻译成其他语言

在结束本章之前，我们将讨论 Yii 中的国际化（`i18n`）和本地化（`l10n`）。**国际化**指的是以一种可以适应各种语言而无需进行基础工程更改的方式设计软件应用程序的过程。**本地化**指的是将国际化的软件应用程序适应特定地理位置或语言的过程，通过添加与地区相关的格式化和翻译文本。Yii 以以下方式支持这些功能：

+   它为几乎每种语言和地区提供了地区数据

+   它提供了辅助翻译文本消息字符串和文件的服务

+   它提供了与地区相关的日期和时间格式化

+   它提供了与地区相关的数字格式化

## 定义地区和语言

**区域**是指定义用户语言、国家和可能与用户位置相关的任何其他用户界面首选项的一组参数。它通常由一个语言标识符和一个区域标识符组成的复合`ID`来标识。例如，`en_us`的区域 ID 代表美国地区的英语。为了保持一致，Yii 中的所有区域 ID 都标准化为小写的`LanguageID`或`LanguageID_RegionID`格式（例如，`en`或`en_us`）。

在 Yii 中，区域数据表示为`CLocale`类的实例或其子类。它提供特定于区域的信息，包括货币和数字符号、货币、数字、日期和时间格式，以及月份、星期几等日期相关名称。通过区域 ID，可以通过使用静态方法`CLocale::getInstance($localeID)`或使用应用程序来获取相应的`CLocale`实例。以下示例代码使用应用程序组件基于`en_us`区域标识符创建一个新实例：

```php
Yii::app()->getLocale('en_us');
```

Yii 几乎为每种语言和地区提供了区域数据。这些数据来自通用区域数据存储库（[`cldr.unicode.org/`](http://cldr.unicode.org/)），存储在根据各自区域 ID 命名的文件中，并位于 Yii 框架目录`framework/i18n/data/`中。因此，在上一个示例中创建新的`CLocale`实例时，用于填充属性的数据来自文件`framework/i18n/data/en_us.php`。如果您查看此目录，您将看到许多语言和地区的数据文件。

回到我们的例子，如果我们想要获取特定于美国地区的英语月份名称，我们可以执行以下代码：

```php
$locale = Yii::app()->getLocale('en_us');
print_r($locale->monthNames);
```

其输出将产生以下结果：

![定义区域和语言](img/8727_10_08.jpg)

如果我们想要意大利语的相同月份名称，我们可以执行相同的操作，但创建一个不同的`CLocale`实例：

```php
$locale = Yii::app()->getLocale('it');
print_r($locale->monthNames);
```

现在我们的输出将产生以下结果：

![定义区域和语言](img/8727_10_09.jpg)

第一个实例基于数据文件`framework/i18n/data/en_us.php`，后者基于`framework/i18n/data/it.php`。如果需要，可以配置应用程序的`localeDataPath`属性，以指定一个自定义目录，您可以在其中添加自定义区域设置数据文件。

## 执行语言翻译

也许`i18n`最受欢迎的功能是语言翻译。如前所述，Yii 提供了消息翻译和视图文件翻译。前者将单个文本消息翻译为所需的语言，后者将整个文件翻译为所需的语言。

翻译请求包括要翻译的对象（文本字符串或文件）、对象所在的源语言以及要将对象翻译为的目标语言。Yii 应用程序区分其目标语言和源语言。**目标**语言是我们针对用户的语言（或区域），而**源**语言是指应用程序文件所写的语言。到目前为止，我们的 TrackStar 应用程序是用英语编写的，也是针对英语用户的。因此，到目前为止，我们的目标语言和源语言是相同的。Yii 的国际化功能，包括翻译，仅在这两种语言不同时适用。

### 执行消息翻译

通过调用以下应用程序方法执行消息翻译：

```php
Yii::t(string $category, string $message, array $params=array ( ), string $source=NULL, string $language=NULL)
```

该方法将消息从源语言翻译为目标语言。

在翻译消息时，必须指定类别，以便允许消息在不同类别（上下文）下进行不同的翻译。类别`Yii`保留用于 Yii 框架核心代码使用的消息。

消息也可以包含参数占位符，这些占位符在调用`Yii::t()`时将被实际参数值替换。以下示例描述了错误消息的翻译。这个消息翻译请求将在原始消息中用实际的`$errorCode`值替换`{errorCode}`占位符：

```php
Yii::t('category', 'The error: "{errorCode}" was encountered during the last request.',     array('{errorCode}'=>$errorCode));
```

翻译消息存储在称为**消息源**的存储库中。消息源表示为`CMessageSource`的实例或其子类的实例。当调用`Yii::t()`时，它将在消息源中查找消息，并在找到时返回其翻译版本。

Yii 提供以下类型的消息源：

+   **CPhpMessageSource**：这是默认的消息源。消息翻译存储为 PHP 数组中的键值对。原始消息是键，翻译后的消息是值。每个数组表示特定类别消息的翻译，并存储在一个单独的 PHP 脚本文件中，文件名为类别名。相同语言的 PHP 翻译文件存储在以区域 ID 命名的相同目录下。所有这些目录都位于由`basePath`指定的目录下。

+   **CGettextMessageSource**：消息翻译存储为`GNU Gettext`文件。

+   **CDbMessageSource**：消息翻译存储在数据库表中。

消息源作为应用程序组件加载。Yii 预先声明了一个名为`messages`的应用程序组件，用于存储用户应用程序中使用的消息。默认情况下，此消息源的类型是`CPhpMessageSource`，用于存储 PHP 翻译文件的基本路径是`protected/messages`。

一个示例将有助于将所有这些内容整合在一起。让我们将**登录**表单上的表单字段标签翻译成一个我们称为`Reversish`的虚构语言。**Reversish**是通过将英语单词或短语倒转来书写的。所以这里是我们登录表单字段标签的 Reversish 翻译：

| 英文 | Reversish |
| --- | --- |
| 用户名 | Emanresu |
| 密码 | Drowssap |
| Remember me next time | Emit txen em rebmemer |

我们将使用默认的`CPhpMessageSource`实现来存储我们的消息翻译。所以我们需要做的第一件事是创建一个包含我们翻译的 PHP 文件。我们将把区域 ID 设置为`rev`，并且现在只是称为类别`default`。我们需要在消息基本目录下创建一个遵循格式`/localeID/CategoryName.php`的新文件。所以我们需要在`/protected/messages/rev/default.php`下创建一个新文件，然后在该文件中添加以下翻译数组：

```php
<?php
return array(
    'Username' => 'Emanresu',
    'Password' => 'Drowssap',
    'Remember me next time' => 'Emit txen em rebmemer',
);
```

接下来，我们需要将应用程序目标语言设置为 Reversish。我们可以在应用程序配置文件中执行此操作，以便影响整个站点。只需在`/protected/config/main.php`文件中的返回数组中添加以下`name=>value`对：

```php
'language'=>'rev',
```

现在我们需要做的最后一件事是调用`Yii::t()`，以便我们的登录表单字段标签通过翻译发送。这些表单字段标签在`LoginForm::attributeLabels()`方法中定义。用以下代码替换整个方法：

```php
/**
   * Declares attribute labels.
   */
  public function attributeLabels()
  {
    return array(
      'rememberMe'=>Yii::t('default','Remember me next time'),
      'username'=>Yii::t('default', 'Username'),
      'password'=>Yii::t('default', 'Password'),
    );
  }
```

现在，如果我们再次访问我们的**登录**表单，我们将看到一个新的 Reversish 版本，如下面的截图所示：

![执行消息翻译](img/8727_10_06.jpg)

### 执行文件翻译

Yii 还提供了根据应用程序的目标区域设置使用不同文件的能力。文件翻译是通过调用应用程序方法 `CApplication::findLocalizedFile()` 来实现的。该方法接受文件的路径，并将在具有与目标区域 ID 相同名称的目录下查找具有相同名称的文件。目标区域 ID 要么作为方法的显式输入指定，要么作为应用程序配置中指定的内容。

让我们试一试。我们真正需要做的就是创建适当的翻译文件。我们将继续翻译登录表单。因此，我们创建一个新的视图文件 `/protected/views/site/rev/login.php`，然后添加我们的翻译内容。同样，这太长了，无法完整列出，但您可以在可下载的代码文件或独立内容中查看 [`gist.github.com/3779850`](https://gist.github.com/3779850)。

我们已经在主配置文件中为应用程序设置了目标语言，并在调用 `render('login')` 时，获取本地化文件的调用将在幕后为我们处理。因此，有了这个文件，我们的登录表单现在看起来如下截图所示：

![执行文件翻译](img/8727_10_07.jpg)

# 总结

在这一章中，我们已经看到 Yii 应用程序如何让您快速轻松地改进设计。我们介绍了布局文件的概念，并介绍了如何在应用程序中使用这些文件来布置需要在许多不同的网页上以类似方式实现的内容和设计。这也向我们介绍了 `CMenu` 和 `CBreadcrumbs` 内置小部件，它们在每个页面上提供了非常易于使用的 UI 导航结构。

然后，我们介绍了 Web 应用程序中主题的概念以及如何在 Yii 中创建它们。我们看到主题允许您轻松地为现有的 Web 应用程序提供新的外观，并允许您重新设计应用程序，而无需重建任何功能或“后端”。

最后，我们通过 `i18n` 和语言翻译的视角来看应用程序的面貌变化。我们学会了如何设置应用程序的目标区域，以启用本地化设置和语言翻译。

在本章和之前的章节中，我们已经多次提到“模块”，但尚未深入了解它们在 Yii 应用程序中的具体内容。这将是下一章的重点。
