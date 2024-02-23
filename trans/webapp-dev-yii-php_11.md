# 第十一章 使用 Yii 模块

到目前为止，我们已经为我们的 TrackStar 应用程序添加了许多功能。如果你回想一下第七章，“用户访问控制”，我们介绍了用户访问控制，根据用户角色层次结构限制某些功能。这在按项目基础限制对一些管理功能的访问上非常有帮助。例如，在特定项目中，您可能不希望允许团队的所有成员删除项目。我们使用基于角色的访问控制实现，将用户分配到项目中的特定角色，然后根据这些角色允许/限制对功能的访问。

然而，我们尚未解决的是应用程序整体的管理需求。像 TrackStar 这样的 Web 应用程序通常需要具有完全访问权限的特殊用户。一个例子是能够管理系统中每个用户的所有 CRUD 操作，而不管项目如何。我们应用程序的*完整管理员*应该能够登录并删除或更新任何用户、任何项目、任何问题，管理所有评论等。此外，通常情况下，我们构建适用于整个应用程序的额外功能，例如能够向所有用户留下站点范围的系统消息，管理电子邮件活动，打开/关闭某些应用程序功能，管理角色和权限层次结构本身，更改站点主题等。由于向管理员公开的功能可能与向普通用户公开的功能差异很大，因此将这些功能与应用程序的其余部分分开是一个很好的主意。我们将通过在 Yii 中构建所有我们的管理功能来实现这种分离，这被称为**模块**。

# 功能规划

在这一章中，我们将专注于以下细粒度的开发任务：

+   创建一个新模块来容纳管理功能

+   为管理员添加系统范围消息的能力，以在项目列表页面上查看

+   将新主题应用于模块

+   创建一个新的数据库表来保存系统消息数据

+   为我们的系统消息生成所有 CRUD 功能

+   将对新模块内的所有功能的访问限制为管理员用户

+   在项目列表页面上显示新的系统消息

# 使用模块

Yii 中的**模块**非常类似于包含在较大应用程序中的整个小型应用程序。它具有非常相似的结构，包含模型、视图、控制器和其他支持组件。但是，模块本身不能作为独立应用程序部署；它们必须驻留在一个应用程序中。

模块在以模块化方式构建应用程序方面非常有用。大型应用程序通常可以分成离散的应用程序功能，可以使用模块分别构建。网站功能，如添加用户论坛或用户博客，或站点管理员功能，是一些可以从主要站点功能中分割出来的示例，使它们可以在将来的项目中轻松重复使用。我们将使用一个模块来在我们的应用程序中创建一个独特的位置，以容纳我们的管理功能。

## 创建一个模块

使用我们的好朋友 Gii 创建一个新模块非常简单。在我们的 URL 更改就位后，该工具现在可以通过`http://localhost/trackstar/gii`访问。导航到那里，并在左侧菜单中选择**模块生成器**选项。您将看到以下截图：

![创建一个模块](img/8727_11_01.jpg)

我们需要为模块提供一个唯一的名称。由于我们正在创建一个 admin 模块，我们将非常有创意地给它命名为`admin`。在**Module ID**字段中输入这个名称，然后单击**Preview**按钮。如下截图所示，它将向您展示它打算生成的所有文件，允许您在创建它们之前预览每个文件：

![创建模块](img/8727_11_02.jpg)

单击**Generate**按钮，让它创建所有这些文件。您需要确保您的`/protected`文件夹对 Web 服务器进程是可写的，以便它可以自动创建必要的目录和文件。以下截图显示了成功生成模块的情况：

![创建模块](img/8727_11_03.jpg)

让我们更仔细地看看模块生成器为我们创建了什么。在 Yii 中，模块被组织为一个目录，其名称与模块的唯一名称相同。默认情况下，所有模块目录都位于`protected/modules`下。每个模块目录的结构与我们主应用程序的结构非常相似。这个命令为我们做的事情是为 admin 模块创建目录结构的骨架。由于这是我们的第一个模块，顶级目录`protected/modules`被创建，然后在其下创建了一个`admin/`目录。以下截图显示了执行`module`命令时创建的所有目录和文件：

![创建模块](img/8727_11_16.jpg)

模块必须有一个`module`类，该类直接或从`CWebModule`的子类扩展。模块类名称是通过组合模块 ID（即我们创建模块`admin`时提供的名称）和字符串`Module`来创建的。模块 ID 的第一个字母也被大写。所以在我们的情况下，我们的 admin 模块类文件名为`AdminModule.php`。模块类用作存储模块代码中共享信息的中心位置。例如，我们可以使用`CWebModule`的`params`属性来存储模块特定的参数，并使用其`components`属性在模块级别共享应用程序组件。这个模块类的作用类似于应用程序类对整个应用程序的作用。所以`CWebModule`对我们的模块来说就像`CWebApplication`对我们的应用程序一样。

## 使用模块

就像成功创建消息所指示的那样，在我们可以使用新模块之前，我们需要配置主应用程序的`modules`属性，以便包含它供使用。在我们向应用程序添加`gii`模块时，我们就已经这样做了，这使我们能够访问 Gii 代码生成工具。我们在主配置文件`protected/config/main.php`中进行了这些更改。以下突出显示的代码指示了必要的更改：

```php
'modules'=>array(
      'gii'=>array(
            'class'=>'system.gii.GiiModule',
            'password'=>'iamadmin',
      ),
 **'admin',**
   ),
```

保存这些更改后，我们的新`admin`模块已经准备好供使用。我们可以通过访问`http://localhost/trackstar/admin/default/index`来查看为我们创建的简单索引页面。用于访问我们模块中页面的请求路由结构与我们主应用程序页面的结构类似，只是我们还需要在路由中包含`moduleID`目录。我们的路由将具有一般形式`/moduleID/controllerID/actionID`。因此，URL 请求`/admin/default/index`正在请求`admin`模块的默认控制器的索引方法。当我们访问这个页面时，我们会看到类似以下截图的内容：

![使用模块](img/8727_11_05.jpg)

## 模块布局

我们会注意到，在上一章中创建的主题 `newtheme` 也被应用到了我们的模块上。原因是我们的模块控制器类扩展了 `protected/components/Controller.php`，它将其布局指定为 `$layout='//layouts/column1'`。关键在于这个定义前面的双斜杠。这指定我们使用主应用程序路径而不是特定模块路径来查找布局文件。因此，我们得到的布局文件与我们的应用程序的其余部分相同。如果我们将其改为单斜杠而不是双斜杠，我们会看到我们的 `admin` 模块根本没有应用布局。请尝试一下。原因是现在，只有单斜杠，即 `$layout='/layouts/column1'`，它正在在模块内寻找布局文件而不是父应用程序。请继续进行此更改，并在我们继续进行时保持单斜杠定义。

您可以在模块中几乎可以单独配置所有内容，包括布局文件的默认路径。Web 模块的默认布局路径是 `/protected/modules/[moduleID]/views/layouts`，在我们的情况下是 `admin`。我们可以看到在这个目录下没有文件，因此没有默认布局可应用于模块。

由于我们指定了一个主题，我们的情况稍微复杂一些。我们还可以在这个主题中管理所有模块视图文件，包括模块布局视图文件。如果我们这样做，我们需要添加到我们的主题目录结构以适应我们的新模块。目录结构非常符合预期。它的一般形式是 `/themes/[themeName]/views/[moduleID]/layouts/` 用于布局文件，`/themes/[themeName]/views/[moduleID]/[controllerID]/` 用于控制器视图文件。

为了帮助澄清这一点，让我们来看一下 Yii 在尝试决定为我们的新 `admin` 模块使用哪些视图文件时的决策过程。如前所述，如果我们在布局视图文件之前使用双斜杠（"//"）指定，它将查找父应用程序以找到布局文件。但让我们看看当我们使用单斜杠并要求它在模块内找到适当的布局文件时的情况。在单斜杠的情况下，当在我们的 `admin` 模块的 `DefaultController.php` 文件中发出 `$this->render('index')` 时，正在发生以下情况：

1.  由于调用了 `render()`，而不是 `renderPartial()`，它将尝试用布局文件装饰指定的 `index.php` 视图文件。由于我们的应用程序当前配置为使用名为 `newtheme` 的主题，它将首先在此主题目录下查找布局文件。我们的新模块的 `DefaultController` 类扩展了我们的应用程序组件 `Controller.php`，它将 `column1` 指定为其 `$layout` 属性。这个属性没有被覆盖，所以它也是 `DefaultController` 的布局文件。最后，由于这一切都发生在 `admin` 模块内部，Yii 首先寻找以下布局文件：

`/themes/newtheme/views/admin/layouts/column1.php`

（请注意在此目录结构中包含 `moduleID`。）

1.  这个文件不存在，所以它会回到模块的默认位置查找。如前所述，默认布局目录对每个模块都是特定的。所以在这种情况下，它将尝试定位以下布局文件：

`/protected/modules/admin/views/layouts/column1.php`

1.  这个文件也不存在，所以将无法应用布局。现在它将尝试渲染指定的 `index.php` 视图文件而不使用布局。然而，由于我们已经为这个应用程序指定了特定的 `newtheme` 主题，它将首先寻找以下视图文件：

`/themes/newtheme/views/admin/default/index.php`

1.  这个文件也不存在，所以它会再次在这个模块（`AdminModule`）的默认位置内寻找这个控制器（`DefaultController.php`），即`/protected/modules/admin/views/default/index.php`。

这解释了为什么页面`http://localhost/trackstar/admin/default/index`在没有任何布局的情况下呈现（在我们使用单斜杠作为布局文件声明的前缀时，`$layout='/layouts/column1'`）。为了现在完全分开和简单，让我们将我们的视图文件管理在模块的默认位置，而不是在`newtheme`主题下。此外，让我们将我们的`admin`模块应用与我们原始应用程序相同的设计，即在应用新主题之前应用的应用程序外观。这样，我们的`admin`页面将与我们的正常应用程序页面有非常不同的外观，这将帮助我们记住我们处于特殊的管理部分，但我们不必花时间设计新的外观。

### 应用布局

首先让我们为我们的模块设置一个默认布局值。我们在模块类`/protected/modules/AdminModule.php`的`init()`方法中设置模块范围的配置设置。因此，打开该文件并添加以下突出显示的代码：

```php
class AdminModule extends CWebModule
{
  public function init()
  {
    // this method is called when the module is being created
    // you may place code here to customize the module or the application

    // import the module-level models and components
    $this->setImport(array(
      'admin.models.*',
      'admin.components.*',
    ));

 **$this->layout = 'main';**

  }
```

这样，如果我们没有在更细粒度的级别上指定布局文件，比如在控制器类中，所有模块视图都将由模块默认布局目录`/protected/modules/admin/views/layouts/`中的`main.php`布局文件装饰。

现在当然，我们需要创建这个文件。从主应用程序中复制两个布局文件`/protected/views/layouts/main.php`和`/protected/views/layouts/column1.php`，并将它们都放在`/protected/modules/admin/views/layouts/`目录中。在将这些文件复制到新位置后，我们需要对它们进行一些小的更改。

首先让我们修改`column1.php`。在调用`beginContent()`时删除对`//layouts/main`的显式引用：

```php
**<?php $this->beginContent(); ?>**
<div id="content">
  <?php echo $content; ?>
</div><!-- content -->
<?php $this->endContent(); ?>
```

在调用`beginContent()`时不指定输入文件将导致它使用我们模块的默认布局，我们刚刚设置为我们新复制的`main.php`文件。

现在让我们对`main.php`布局文件进行一些更改。我们将在应用程序标题文本中添加**管理控制台**，以强调我们处于应用程序的一个独立部分。我们还将修改菜单项，添加一个链接到**管理**首页，以及一个链接返回到主站点。我们可以从菜单中删除**关于**和**联系**链接，因为我们不需要在**管理**部分重复这些选项。文件的添加如下所示：

```php
...
<div class="container" id="page">

  <div id="header">
 **<div id="logo"><?php echo CHtml::encode(Yii::app()->name) . " Admin Console"; ?></div>**
  </div><!-- header -->

  <div id="mainmenu">
    <?php $this->widget('zii.widgets.CMenu',array(
      'items'=>array(
 **array('label'=>'Back To Main Site', 'url'=>array('/project')),**
 **array('label'=>'Admin', 'url'=>array('/admin/default/index')),**
        array('label'=>'Login', 'url'=>array('/site/login'), 'visible'=>Yii::app()->user->isGuest),
        array('label'=>'Logout ('.Yii::app()->user->name.')', 'url'=>array('/site/logout'), 'visible'=>!Yii::app()->user->isGuest)
      ),
    )); ?>
  </div><!-- mainmenu -->
```

我们可以保持文件的其余部分不变。现在，如果我们访问我们的`admin`模块页面`http://localhost/trackstar/admin/default/index`，我们会看到以下截图：

应用布局

如果我们点击**返回主站点**链接，我们会看到我们被带回了主应用程序的新主题版本。

# 限制管理员访问

你可能已经注意到的一个问题是，任何人，包括访客用户，都可以访问我们的新`admin`模块。我们正在构建这个管理模块来暴露应用程序功能，这些功能只能让具有管理权限的用户访问。因此，我们需要解决这个问题。

幸运的是，我们已经在应用程序中实现了 RBAC 访问模型，在第七章中，*用户访问控制*。现在我们需要做的就是扩展它，包括一个新的管理员角色，并为该角色提供新的权限。

如果您还记得第七章中的内容，*用户访问控制*，我们使用了 Yii 的`console`命令来实现我们的 RBAC 结构。我们需要添加到其中。因此，打开包含该`console`命令的文件`/protected/commands/shell/RbacCommand.php`，并在我们创建`owner`角色的地方添加以下代码：

```php
//create a general task-level permission for admins
 $this->_authManager->createTask("adminManagement", "access to the application administration functionality");   
 //create the site admin role, and add the appropriate permissions   
$role=$this->_authManager->createRole("admin"); 
$role->addChild("owner");
$role->addChild("reader"); 
$role->addChild("member");
$role->addChild("adminManagement");
//ensure we have one admin in the system (force it to be user id #1)
$this->_authManager->assign("admin",1);
```

这将创建一个名为`adminManagement`的新任务和一个名为`admin`的新角色。然后，它将添加`owner`、`reader`和`member`角色以及`adminManagement`任务作为子级，以便`admin`角色从所有这些角色继承权限。最后，它将分配`admin`角色给我们系统中的第一个用户，以确保我们至少有一个管理员可以访问我们的管理模块。

现在我们必须重新运行命令以更新数据库的这些更改。要这样做，只需使用`rbac`命令运行`yiic`命令行工具：

```php
**% cd Webroot/trackstar/protected**
**% ./yiic rbac**

```

### 注意

随着添加了这个额外的角色，我们还应该更新在提示时显示的消息文本，以继续指示将创建第四个角色。我们将把这留给读者来练习。这些更改已经在可下载的代码文件中进行了更改，供您参考。

有了这些对我们的 RBAC 模型的更改，我们可以在`AdminModule::beforeControllerAction()`方法中添加对`admin`模块的访问检查，以便除非用户处于`admin`角色，否则不会执行`admin`模块中的任何内容：

```php
public function beforeControllerAction($controller, $action)
{
  if(parent::beforeControllerAction($controller, $action))
  {
    // this method is called before any module controller action is performed
    // you may place customized code here
 **if( !Yii::app()->user->checkAccess("admin") )**
 **{**
 **throw new CHttpException(403,Yii::t('application','You are not authorized to perform this action.'));**
 **}**
 **return true;**
  }
  else
    return false;
}
```

有了这个，如果一个尚未被分配`admin`角色的用户现在尝试访问**管理**模块中的任何页面，他们将收到一个 HTTP 403 授权错误页面。例如，如果您尚未登录并尝试访问**管理**页面，您将收到以下结果：

![限制管理员访问](img/8727_11_06.jpg)

对于任何尚未分配给`admin`角色的用户也是如此。

现在我们可以有条件地将**管理**部分的链接添加到我们主应用程序菜单中。这样，具有管理访问权限的用户就不必记住繁琐的 URL 来导航到**管理**控制台。提醒一下，我们的主应用程序菜单位于应用程序的主题默认应用程序布局文件`/themes/newtheme/views/layouts/main.php`中。打开该文件并将以下突出显示的代码添加到菜单部分：

```php
<div id="mainmenu">
  <?php $this->widget('zii.widgets.CMenu',array(
    'items'=>array(
      array('label'=>'Projects', 'url'=>array('/project')),
      array('label'=>'About', 'url'=>array('/site/page', 'view'=>'about')),
      array('label'=>'Contact', 'url'=>array('/site/contact')),
 **array('label'=>'Admin', 'url'=>array('/admin/default/index'), 'visible'=>Yii::app()->user->checkAccess("admin")),**
      array('label'=>'Login', 'url'=>array('/site/login'), 'visible'=>Yii::app()->user->isGuest),
      array('label'=>'Logout ('.Yii::app()->user->name.')', 'url'=>array('/site/logout'), 'visible'=>!Yii::app()->user->isGuest)
    ),
  )); ?>
</div><!-- mainmenu -->
```

现在，当以具有`admin`访问权限的用户（在我们的情况下，我们将其设置为`user id = 1`，“**用户一**”）登录到应用程序时，我们将在顶部导航中看到一个新的链接，该链接将带我们进入我们新添加的站点**管理**部分。

![限制管理员访问](img/8727_11_07.jpg)

# 添加系统范围的消息

**模块**可以被视为一个小型应用程序本身，向模块添加功能实际上与向主应用程序添加功能的过程相同。让我们为管理员添加一些新功能；我们将添加管理用户首次登录到应用程序时显示的系统范围消息的功能。

## 创建数据库表

通常情况下，对于全新的功能，我们需要一个地方来存储我们的数据。我们需要创建一个新表来存储我们的系统范围消息。对于我们的示例，我们可以保持这个非常简单。这是我们表的定义：

```php
CREATE TABLE `tbl_sys_message` 
( 
  `id` INTEGER NOT NULL PRIMARY KEY AUTO_INCREMENT,
  `message` TEXT NOT NULL, 
  `create_time` DATETIME,
  `create_user_id` INTEGER,
  `update_time` DATETIME,
  `update_user_id` INTEGER  
) 
```

当然，当添加这个新表时，我们将创建一个新的数据库迁移来管理我们的更改。

```php
**% cd Webroot/trackstar/protected**
**% ./yiic migrate create_system_messages_table**

```

这些命令在`protected/migrations/`目录下创建一个新的迁移文件。这个文件的内容可以从可下载的代码或可在[`gist.github.com/3785282`](https://gist.github.com/3785282)上找到的独立代码片段中获取。（我们没有包括类名；请记住，您的文件名和相应的类将具有不同的时间戳前缀。）

一旦这个文件就位，我们就可以运行我们的迁移来添加这个新表：

```php
**% cd Webroot/trackstar/protected**
**% ./yiic migrate**

```

## 创建我们的模型和 CRUD 脚手架

现在我们已经创建了表，下一步是使用我们喜爱的工具 Gii 代码生成器生成`model`类。我们将首先使用**Model Generator**选项创建`model`类，然后使用**Crud Generator**选项创建基本的脚手架，以便快速与这个模型进行交互。前往 Gii 工具表单以创建新的模型(`http://localhost/trackstar/gii/model`)。这一次，由于我们是在模块的上下文中进行操作，我们需要明确指定模型路径。填写表单中的值，如下面截图所示（当然，你的**Code Template**路径值应该根据你的本地设置具体而定）：

![创建我们的模型和 CRUD 脚手架](img/8727_11_08.jpg)

注意，我们将**Model Path**文本框更改为`application.modules.admin.models`。点击**Generate**按钮生成**Model Class**值。

现在我们可以以类似的方式创建 CRUD 脚手架。我们之前所做的和现在要做的唯一真正的区别是我们要指定`model`类的位置在`admin`模块中。从 Gii 工具中选择**Crud Generator**选项后，填写**Model Class**和**Controller ID**表单字段，如下截图所示：

![创建我们的模型和 CRUD 脚手架](img/8727_11_09.jpg)

这告诉工具我们的`model`类在`admin`模块下，我们的控制器类以及与此代码生成相关的所有其他文件也应该放在`admin`模块中。

首先点击**Preview**按钮，然后点击**Generate**完成创建。下面的截图显示了此操作创建的所有文件列表：

![创建我们的模型和 CRUD 脚手架](img/8727_11_10.jpg)

## 添加到我们新功能的链接

让我们在主`admin`模块导航中添加一个新的菜单项，链接到我们新创建的消息功能。打开包含我们模块主菜单导航的文件`/protected/modules/admin/views/layouts/main.php`，并向菜单小部件添加以下`array`项：

```php
array('label'=>'System Messages', 'url'=>array('/admin/sysMessage/idex')),
```

如果我们在`http://localhost/trackstar/admin/sysMessage/create`查看新的系统消息，我们会看到以下内容：

![添加到我们新功能的链接](img/8727_11_11.jpg)

我们新系统消息功能的自动生成控制器和视图文件是使用主应用程序的两列布局文件创建的。如果你查看`SysMessageController.php`类文件，你会看到布局定义如下：

```php
public $layout='//layouts/column2';
```

注意前面的双斜杠。所以我们可以看到我们新添加的 admin 功能没有使用我们`admin`模块的布局文件。我们可以修改`controller`类以使用我们现有的单列布局文件，或者我们可以在我们的模块布局文件中添加一个两列布局文件。后者会稍微容易一些，而且看起来也更好，因为所有的视图文件都被创建为在第二个右侧列中显示它们的子菜单项（即链接到所有 CRUD 功能）。我们还需要修改我们新创建的模型类和相应的表单，以删除一些不需要的表单字段。以下是我们需要做的全部内容：

1.  将主应用程序中的两列布局复制到我们的模块中，即将`/protected/views/layouts/column2.php`复制到`/protected/modules/admin/views/layouts/column2.php`。

1.  在新复制的`column2.php`文件的第一行，将`//layouts/main`作为`beginContent()`方法调用的输入删除。

1.  修改`SysMessage`模型类以扩展`TrackstarActiveRecord`。（如果你记得的话，这会自动更新我们的`create_time/user`和`update_time/user`属性。）

1.  修改`SysMessageController`控制器类，以使用模块目录中的新`column2.php`布局文件，而不是主应用程序中的文件。自动生成的代码已经指定了`$layout='//layouts/column2'`，但我们需要将其简单地改为`$layout='/layouts/column2'`。

1.  由于我们正在扩展`TrackstarActiveRecord`，我们可以从自动生成的 sys-messages 创建表单中删除不必要的字段，并从模型类中删除它们的相关规则。例如，从`modules/admin/views/sysMessage/_form.php`中删除以下表单字段：

```php
    <div class="row">
        <?php echo $form->labelEx($model,'create_time'); ?>
        <?php echo $form->textField($model,'create_time'); ?>
        <?php echo $form->error($model,'create_time'); ?>
      </div>

      <div class="row">
        <?php echo $form->labelEx($model,'create_user_id'); ?>
        <?php echo $form->textField($model,'create_user_id'); ?>
        <?php echo $form->error($model,'create_user_id'); ?>
      </div>

      <div class="row">
        <?php echo $form->labelEx($model,'update_time'); ?>
        <?php echo $form->textField($model,'update_time'); ?>
        <?php echo $form->error($model,'update_time'); ?>
      </div>

      <div class="row">
        <?php echo $form->labelEx($model,'update_user_id'); ?>
        <?php echo $form->textField($model,'update_user_id'); ?>
        <?php echo $form->error($model,'update_user_id'); ?>
      </div> 
    ```

1.  然后从`SysMessage::rules()`方法中更改这两条规则：

```php
    array('create_user, update_user', 'numerical', 'integerOnly'=>true), and array('create_time, update_time', 'safe'),
    ```

重要的是只为用户可以输入的那些字段指定规则。对于已定义规则的字段，可以从`POST`或`GET`请求中以批量方式设置，并且保留不希望用户访问的字段的规则可能会导致安全问题。

我们应该做的最后一次更改是更新我们简单的访问规则，以反映只有`admin`角色的用户才能访问我们的操作方法的要求。这主要是为了说明目的，因为我们已经在`AdminModule::beforeControlerAction`方法中使用我们的 RBAC 模型方法处理了访问。实际上，我们可以完全删除`accessRules()`方法。但是，让我们更新它们以反映要求，以便您可以看到使用访问规则方法将如何工作。在`SysMessageController::accessRules()`方法中，将整个内容更改为以下内容：

```php
public function accessRules()
{
  return array(
    array('allow',  // allow only users in the 'admin' role access to our actions
      'actions'=>array('index','view', 'create', 'update', 'admin', 'delete'),
      'roles'=>array('admin'),
    ),
    array('deny',  // deny all users
      'users'=>array('*'),
    ),
  );
}
```

好的，有了所有这些，现在如果我们访问`http://localhost/trackstar/admin/sysMessage/create`来访问我们的新消息输入表单，我们将看到类似以下截图的内容：

![添加到我们的新功能的链接](img/8727_11_12.jpg)

填写此表单，消息为`Hello Users! This is your admin speaking...`，然后单击**Create**。应用程序将重定向您到这条新创建消息的详细列表页面，如下截图所示：

![添加到我们的新功能的链接](img/8727_11_13.jpg)

## 向用户显示消息

现在我们的系统中有一条消息，让我们在应用程序主页上向用户显示它。

### 导入新的模型类以进行应用程序范围的访问

为了从应用程序的任何地方访问新创建的模型，我们需要将其作为应用程序配置的一部分导入。修改`protected/config/main.php`以包括新的`admin module models`文件夹：

```php
// autoloading model and component classes
'import'=>array(
  'application.models.*',
  'application.components.*',
 **'application.modules.admin.models.*',**
),
```

### 选择最近更新的消息

我们将限制显示只有一条消息，并且我们将根据表中的`update_time`列选择最近更新的消息。由于我们想要将其添加到主项目列表页面，我们需要修改`ProjectController::actionIndex()`方法。通过添加以下突出显示的代码来修改该方法：

```php
public function actionIndex()
  {
      $dataProvider=new CActiveDataProvider('Project');

      Yii::app()->clientScript->registerLinkTag(
          'alternate',
          'application/rss+xml',
          $this->createUrl('comment/feed'));

 **//get the latest system message to display based on the update_time column**
 **$sysMessage = SysMessage::model()->find(array(**
 **'order'=>'t.update_time DESC',**
 **));**
 **if($sysMessage !== null)**
 **$message = $sysMessage->message;**
 **else**
 **$message = null;**

      $this->render('index',array(
        'dataProvider'=>$dataProvider,
 **'sysMessage'=>$message,**
      ));
  }
```

现在我们需要修改我们的视图文件来显示这个新的内容。将以下代码添加到`views/project/index.php`，就在`<h1>Projects</h1>`标题文本上方：

```php
<?php if($sysMessage !== null):?>
    <div class="sys-message">
        <?php echo $sysMessage; ?>
    </div>
<?php endif; ?>
```

现在当我们访问我们的项目列表页面（即我们应用程序的主页）时，我们可以看到它显示如下截图所示：

![选择最近更新的消息](img/8727_11_14.jpg)

### 添加一点设计调整

好的，这做到了我们想要的，但是这条消息对用户来说并不是很突出。让我们通过向我们的主 CSS 文件(`/themes/newtheme/css/main.css`)添加一小段代码来改变这一点：

```php
div.sys-message
{
  padding:.8em;
  margin-bottom:1em;
  border:3px solid #ddd;
  background:#9EEFFF;
  color:#FF330A;
  border-color:#00849E;
}
```

有了这个，我们的消息现在在页面上真的很突出。以下截图显示了具有这些更改的消息：

![添加一点设计调整](img/8727_11_15.jpg)

有人可能会认为这个设计调整有点过分。用户可能会因为不得不整天盯着这些消息颜色而感到头疼。与其淡化颜色，不如使用一点 JavaScript 在 5 秒后淡出消息。由于我们将在用户访问这个**主页**时每次显示消息，防止他们盯着它太久可能会更好。

我们将简化操作，并利用 Yii 随附的强大 JavaScript 框架 jQuery。**jQuery**是一个开源的 JavaScript 库，简化了 HTML **文档对象模型**（**DOM**）和 JavaScript 之间的交互。深入了解 jQuery 的细节超出了本书的范围。值得访问其文档以更加了解其特性。由于 Yii 随附了 jQuery，您可以在视图文件中简单地注册 jQuery 代码，Yii 将为您包含核心 jQuery 库。

我们还将使用应用程序助手组件`CClientScript`来为我们在生成的网页中注册 jQuery JavaScript 代码。它将确保它已被放置在适当的位置，并已被正确标记和格式化。

因此，让我们修改之前添加的内容，包括一个 JavaScript 片段来淡出消息。用以下内容替换我们刚刚添加到`views/project/index.php`的内容：

```php
<?php if($sysMessage != null):?>
    <div class="sys-message">
        <?php echo $sysMessage; ?>
    </div>
<?php
  Yii::app()->clientScript->registerScript(
     'fadeAndHideEffect',
     '$(".sys-message").animate({opacity: 1.0}, 5000).fadeOut("slow");'
  );
endif; ?>
```

现在，如果我们重新加载主项目列表页面，我们会看到消息在 5 秒后淡出。有关您可以轻松添加到页面的酷炫 jQuery 效果的更多信息，请查看[`api.jquery.com/category/effects/`](http://api.jquery.com/category/effects/)上提供的 JQuery API 文档。

最后，为了确信一切都按预期工作，您可以添加另一条系统范围的消息。由于这条更新时间更近的消息将显示在项目列表页面上。

# 总结

在本章中，我们介绍了 Yii 模块的概念，并通过使用一个模块来创建站点的管理部分来演示了它的实用性。我们演示了如何创建一个新模块，如何更改模块的布局和主题，如何在模块内添加应用程序功能，甚至如何利用现有的 RBAC 模型，将授权访问控制应用于模块内的功能。我们还演示了如何使用 jQuery 为我们的应用程序增添一些 UI 效果。

通过添加这个管理界面，我们现在已经把应用程序的所有主要部分都放在了适当的位置。虽然应用程序非常简单，但我们觉得现在是时候为其准备投入生产了。下一章将重点介绍如何为我们的应用程序准备生产部署。
