# 第七章：用户访问控制

基于用户的 Web 应用程序，如我们的 TrackStar 应用程序，通常需要根据请求的发起者来控制对某些功能的访问。当我们谈论*用户访问控制*时，我们在高层次上指的是应用程序在进行请求时需要询问的一些问题。这些问题是：

+   谁在发起请求？

+   该用户是否有适当的权限来访问所请求的功能？

这些问题的答案有助于应用程序做出适当的响应。

在第六章中完成的工作为我们的应用程序提供了回答这些问题的能力。应用程序现在允许用户建立自己的身份验证凭据，并在用户登录时验证用户名和密码。成功登录后，应用程序确切地知道谁在发起后续的请求。

在本章中，我们将专注于帮助应用程序回答第二个问题。一旦用户提供了适当的身份识别，应用程序需要一种方法来确定他们是否也有权限执行所请求的操作。我们将通过利用 Yii 的用户访问控制功能来扩展我们的基本授权模型。Yii 提供了**简单的访问控制过滤器**以及更复杂的**基于角色的访问控制**（**RBAC**）实现，以帮助我们满足用户授权的要求。在实现 TrackStar 应用程序的用户访问要求时，我们将更仔细地研究这两者。

# 功能规划

当我们在第三章中首次介绍我们的 TrackStar 应用程序时，我们提到应用程序有两个高级用户状态，即匿名和已验证。这只是区分了已成功登录（已验证）和未登录（匿名）的用户。我们还介绍了已验证用户在项目内拥有不同角色的概念。在特定项目中，用户可以担任以下三种角色之一：

+   **项目所有者**对项目拥有*全部*的管理访问权限

+   **项目成员**具有*一些*管理访问权限，但与项目所有者相比，访问权限更有限

+   **项目读者**具有*只读*访问权限。这样的用户无法更改项目的内容

本章的重点是实施一种管理授予应用程序用户的访问控制的方法。我们需要一种方式来创建和管理我们的角色和权限，将它们分配给用户，并强制我们对每个用户角色想要的访问控制规则。

为了实现前面概述的目标，我们将在本章中专注于以下内容：

+   实施一种策略，强制用户在获得任何项目或问题相关功能的访问权限之前先登录

+   创建用户角色并将这些角色与特定的权限结构关联起来

+   实现将用户分配到角色（及其相关权限）的能力

+   确保我们的角色和权限结构存在于每个项目的基础上（即允许用户在不同项目中拥有不同的权限）

+   实现将用户关联到项目以及同时关联到项目内的角色的能力

+   在整个应用程序中实施必要的授权访问检查，以根据其权限适当地授予或拒绝应用程序用户的访问权限

幸运的是，Yii 自带了许多内置功能，帮助我们实现这些要求。所以，让我们开始吧。

# 访问控制过滤器

我们在第五章中首次介绍了*filters*，当我们在允许使用问题功能之前强制执行有效的项目上下文时。如果您还记得，我们在`IssueController`类中添加了一个类方法过滤器`filterProjectContext()`，以确保在对问题实体执行任何操作之前，我们有一个有效的项目上下文。Yii 提供了一种类似的方法，用于在控制器中逐个操作处理简单的访问控制。

Yii 框架提供了一个名为`accessControl`的过滤器。这个过滤器可以直接在控制器类中使用，以提供一个授权方案，用于验证用户是否可以访问特定的控制器操作。实际上，敏锐的读者会记得，当我们在第五章中实现`projectContext`过滤器时，我们注意到这个访问控制过滤器已经包含在我们的`IssueController`和`ProjectController`类的过滤器列表中，如下所示：

```php
/**
 * @return array action filters
 */
public function filters()
{
return array(
'accessControl', // perform access control for CRUD operations
);
}
```

这是使用 Gii CRUD 代码生成工具生成的自动生成代码中包含的。自动生成的代码还覆盖了`accessRules()`方法，这是必要的，以便使用访问控制过滤器。在这个方法中，您定义实际的授权规则。

我们的 CRUD 操作的默认实现设置为允许任何人查看现有问题和项目的列表。但是，它限制了创建和更新的访问权限，只允许经过身份验证的用户，并进一步将删除操作限制为特殊的*admin*用户。您可能还记得，当我们首次在项目上实现 CRUD 操作时，我们必须先登录才能创建新项目。在处理问题和用户时也是如此。控制这种授权和访问的机制正是这个访问控制过滤器。让我们更仔细地看一下`ProjectController.php`类文件中的这个实现。

`ProjectController`类中有两个与访问控制相关的方法：`filters()`和`accessRules()`。`filters()`方法配置过滤器。

```php
/**
 * @return array action filters
 */
public function filters()
{
return array(
'accessControl', // perform access control for CRUD operations
);
}
```

`accessRules()` 方法用于定义访问过滤器使用的授权规则，如下所示：

```php
/**
* Specifies the access control rules.
* This method is used by the 'accessControl' filter.
* @return array access control rules
*/
public function accessRules()
{
return array(
array('allow',  // allow all users to perform 'index' and 'view' actions
'actions'=>array('index','view'),
'users'=>array('*'),
),
array('allow', // allow authenticated user to perform 'create' and 'update' actions
'actions'=>array('create','update'),
'users'=>array('@'),
),
array('allow', // allow admin user to perform 'admin' and 'delete' actions
'actions'=>array('admin','delete'),
'users'=>array('admin'),
),
array('deny',  // deny all users
'users'=>array('*'),
),
);
}
```

`filters()` 方法对我们来说已经很熟悉了。在这里，我们指定控制器类中要使用的所有过滤器。在这种情况下，我们只有一个`accessControl`，它是 Yii 框架提供的一个过滤器。这个过滤器使用另一个方法`accessRules()`，它定义了驱动访问限制的规则。

在`accessRules()`方法中，指定了四条规则。每条规则都表示为一个数组。数组的第一个元素要么是*allow*，要么是*deny*。它们分别表示授予或拒绝访问。数组的其余部分由`name=>value`对组成，指定了规则的其余参数。

让我们先看一下之前定义的第一条规则：

```php
array('allow',  // allow all users to perform 'index' and 'view' actions
'actions'=>array('index','view'),
'users'=>array('*'),
),
```

这条规则允许任何用户执行`actionIndex()`和`actionView()`控制器操作。在`'users'`元素的值中使用的星号(`*`)是一种用于指定任何用户（匿名、经过身份验证或其他方式）的特殊字符。

现在让我们来看一下定义的第二条规则：

```php
array('allow', // allow authenticated user to perform 'create' and 'update' actions
'actions'=>array('create','update'),
'users'=>array('@'),
),
```

这允许任何经过身份验证的用户访问`actionCreate()`和`actionUpdate()`控制器操作。`@`特殊字符是一种指定任何经过身份验证的用户的方式。

第三条规则在以下代码片段中定义：

```php
array('allow', // allow admin user to perform 'admin' and 'delete' actions
'actions'=>array('admin','delete'),
'users'=>array('admin'),
),
```

这条规则指定了一个名为`admin`的特定用户被允许访问`actionAdmin()`和`actionDelete()`控制器操作。

最后，让我们更仔细地看一下第四条规则：

```php
array('deny',  // deny all users
'users'=>array('*'),
),
```

这条规则拒绝所有用户访问所有控制器操作。我们稍后会更详细地解释这一点。

可以使用多个上下文参数来定义访问规则。前面提到的规则正在指定动作和用户来创建规则上下文，但是还有其他几个参数可以使用。以下是其中一些：

+   **控制器**：指定规则应用的控制器 ID 数组。

+   **角色**：指定规则适用的授权项（角色、操作和权限）列表。这利用了我们将在下一节讨论的 RBAC 功能。

+   **IP 地址**：指定此规则适用的客户端 IP 地址列表。

+   **动词**：指定适用于此规则的 HTTP 请求类型（GET、POST 等）。

+   **表达式**：指定一个 PHP 表达式，其值指示是否应用规则。

+   **动作**：通过相应的动作 ID 指定动作方法，该规则应匹配到该动作。

+   **用户**：指定规则应用的用户。当前应用用户的名称属性用于匹配。这里也可以使用以下三个特殊字符：

1.  *****：任何用户

1.  **?**：匿名用户

1.  **@**：认证用户

如果没有指定用户，规则将适用于所有用户。

访问规则按照它们被指定的顺序逐一进行评估。与当前模式匹配的第一个规则确定授权结果。如果这个规则是一个允许规则，那么动作可以被执行；如果它是一个“拒绝”规则，那么动作就不能被执行；如果没有规则匹配上下文，动作仍然可以被执行。这就是前面提到的第四条规则的定义原因。如果我们没有在规则列表的末尾定义一个拒绝所有用户的规则，那么我们就无法实现我们期望的访问限制。举个例子，看看第二条规则。它指定认证用户可以访问 `actioncreate()` 和 `actionUpdate()` 动作。然而，它并没有规定匿名用户被拒绝访问。它对匿名用户什么也没说。前面提到的第四条规则确保了所有其他不匹配前三个具体规则的请求被拒绝访问。

有了这个设置，对匿名用户拒绝访问所有项目、问题和用户相关功能的应用程序进行更改就很容易。我们只需要将用户数组值的特殊字符`*`更改为`@`特殊字符。这将只允许认证用户访问 `actionIndex()` 和 `actionView()` 控制器动作。所有其他动作已经限制为认证用户。

现在，我们可以在我们的项目、问题和用户控制器类文件中每次进行三次更改。然而，我们有一个基础控制器类，每个类都是从中扩展出来的，即文件 `protected/components/Controller.php` 中的 `Controller` 类。因此，我们可以在这一个文件中添加我们的 CRUD 访问规则，然后从每个子类中删除它。我们还可以在定义规则时利用 `controllers` 上下文参数，以便它只适用于这三个控制器。

首先，让我们在我们的基础控制器类中添加必要的方法。打开 `protected/components/Controller.php` 并添加以下方法：

```php
/**
 * Specifies the access control rules.
 * This method is used by the 'accessControl' filter.
 * @return array access control rules
 */
public function accessRules()
{
return array(
array('allow',  // allow all users to perform 'index' and 'view' actions
**'controllers'=>array('issue','project','user'),**
'actions'=>array('index','view'),
**'users'=>array('@'),**
),
array('allow', // allow authenticated user to perform 'create' and 'update' actions
**'controllers'=>array('issue','project','user'),**
'actions'=>array('create','update'),
'users'=>array('@'),
),
array('allow', // allow admin user to perform 'admin' and 'delete' actions
**'controllers'=>array('issue','project','user'),**
'actions'=>array('admin','delete'),
'users'=>array('admin'),
),
array('deny',  // deny all users
**'controllers'=>array('issue','project','user'),**
'users'=>array('*'),
),
);
}
```

在前面代码片段中突出显示的代码显示了我们所做的更改。我们已经为每个规则添加了 `controllers` 参数，并将索引和查看动作的用户更改为只允许认证用户。

现在我们可以从每个指定的控制器中删除这个方法。打开 `ProjectController.php`、`IssueController.php` 和 `UserController.php` 三个文件，并删除它们各自的 `accessRules()` 方法。

做出这些更改后，应用程序将在访问我们的*项目*、*问题*或*用户*功能之前要求登录。我们仍然允许匿名用户访问`SiteController`类的操作方法，因为这是我们的登录操作所在的地方。显然，如果我们尚未登录，我们必须能够访问登录页面。

# 基于角色的访问控制

现在我们已经使用简单的访问控制过滤器限制了经过身份验证的用户的访问权限，我们需要转而关注满足应用程序更具体的访问控制需求。正如我们提到的，用户将在项目中扮演特定的角色。项目将有*所有者*类型的用户，可以被视为项目管理员。他们将被授予操纵项目的所有访问权限。项目还将有*成员*类型的用户，他们将被授予对项目功能的一些访问权限，但是比所有者能够执行的操作要少。最后，项目可以有*读者*类型的用户，他们只能查看与项目相关的内容，而不能以任何方式更改它。为了根据用户的角色实现这种类型的访问控制，我们转向 Yii 的基于角色的访问控制功能，也简称为 RBAC。

RBAC 是计算机系统安全中管理经过身份验证用户的访问权限的一种成熟方法。简而言之，RBAC 方法在应用程序中定义角色。还定义了执行某些操作的权限，然后将其与角色关联起来。然后将用户分配给一个角色，并通过角色关联获得为该角色定义的权限。对于对 RBAC 概念和方法感兴趣的读者，有大量的文档可供参考。例如维基百科，[`en.wikipedia.org/wiki/Role-based_access_control`](http://en.wikipedia.org/wiki/Role-based_access_control)。我们将专注于 Yii 对 RBAC 方法的具体实现。

Yii 对 RBAC 的实现简单、优雅且强大。在 Yii 中，RBAC 的基础是**授权项**的概念。授权项简单地是应用程序中执行操作的权限。这些权限可以被归类为*角色*、*任务*或*操作*，因此形成了一个权限层次结构。角色可以包括任务（或其他角色），任务可以包括操作（或其他任务），操作是最粒度的权限级别。

例如，在我们的 TrackStar 应用程序中，我们需要一个*所有者*类型的角色。因此，我们将创建一个*角色*类型的授权项，并将其命名为“所有者”。然后，这个角色可以包括诸如“用户管理”和“问题管理”之类的任务。这些任务可以进一步包括组成这些任务的原子操作。继续上面的例子，“用户管理”任务可以包括“创建新用户”、“编辑用户”和“删除用户”操作。这种层次结构允许继承这些权限，因此，以这个例子为例，如果一个用户被分配到所有者角色，他们就会继承对用户执行创建、编辑和删除操作的权限。

在 RBAC 中，通常你会将用户分配给一个或多个角色，用户会继承这些角色被分配的权限。在 Yii 中也是如此。然而，在 Yii 中，我们可以将用户与任何授权项关联，而不仅仅是*角色*类型的授权项。这使我们能够灵活地将特定权限与用户关联在任何粒度级别上。如果我们只想将“删除用户”操作授予特定用户，而不是给予他们所有者角色所具有的所有访问权限，我们可以简单地将用户与这个原子操作关联起来。这使得 Yii 中的 RBAC 非常灵活。

## 配置授权管理器

在我们可以建立授权层次结构，将用户分配给角色，并执行访问权限检查之前，我们需要配置授权管理器应用程序组件`authManager`。这个组件负责存储权限数据和管理权限之间的关系。它还提供了检查用户是否有权执行特定操作的方法。Yii 提供了两种类型的授权管理器`CPhpAuthManager`和`CDbAuthManager`。`CPhpAuthManager`使用 PHP 脚本文件来存储授权数据。`CDbAuthManager`，正如你可能已经猜到的，将授权数据存储在数据库中。`authManager`被配置为一个应用程序组件。配置授权管理器只需要简单地指定使用这两种类型中的哪一种，然后设置它的初始类属性值。

我们将使用数据库实现我们的应用程序。为了进行这个配置，打开主配置文件`protected/config/main.php`，并将以下内容添加到应用程序组件数组中：

```php
// application components
'components'=>array(
…
'authManager'=>array(
'class'=>'CDbAuthManager',
'connectionID'=>'db',
),
```

这建立了一个名为`authManager`的新应用程序组件，指定了类类型为`CDbAuthManager`，并将`connectionID`类属性设置为我们的数据库连接组件。现在我们可以在我们的应用程序的任何地方使用`Yii::app()->authManager`来访问它。

## 创建 RBAC 数据库表

如前所述，`CDbAuthManager`类使用数据库表来存储权限数据。它期望一个特定的模式。该模式在框架文件`YiiRoot/framework/web/auth/schema.sql`中被识别。这是一个简单而优雅的模式，由三个表`AuthItem`，`AuthItemChild`和`AuthAssignment`组成。

`AuthItem`表保存了定义角色、任务或操作的授权项的信息。`AuthItemChild`表存储了形成我们授权项层次结构的父/子关系。最后，`AuthAssignment`表是一个关联表，保存了用户和授权项之间的关联。

因此，我们需要将这个表结构添加到我们的数据库中。就像我们之前做过的那样，我们将使用数据库迁移来进行这些更改。从命令行，导航到 TrackStar 应用程序的`/protected`目录，并创建迁移：

```php
**$ cd /Webroot/trackstar/protected**
**$ ./yiic migrate create create_rbac_tables**

```

这将在`protected/migrations/`目录下创建一个根据迁移文件命名约定命名的新迁移文件（例如，`m120619_015239_create_rbac_tables.php`）。实现`up()`和`down()`迁移方法如下：

```php
public function up()
{
//create the auth item table
$this->createTable('tbl_auth_item', array(
'name' =>'varchar(64) NOT NULL',
'type' =>'integer NOT NULL',
'description' =>'text',
'bizrule' =>'text',
'data' =>'text',
'PRIMARY KEY (`name`)',
), 'ENGINE=InnoDB');

//create the auth item child table
$this->createTable('tbl_auth_item_child', array(
'parent' =>'varchar(64) NOT NULL',
'child' =>'varchar(64) NOT NULL',
'PRIMARY KEY (`parent`,`child`)',
), 'ENGINE=InnoDB');

//the tbl_auth_item_child.parent is a reference to tbl_auth_item.name
$this->addForeignKey("fk_auth_item_child_parent", "tbl_auth_item_child", "parent", "tbl_auth_item", "name", "CASCADE", "CASCADE");

//the tbl_auth_item_child.child is a reference to tbl_auth_item.name
$this->addForeignKey("fk_auth_item_child_child", "tbl_auth_item_child", "child", "tbl_auth_item", "name", "CASCADE", "CASCADE");

//create the auth assignment table
$this->createTable('tbl_auth_assignment', array(
'itemname' =>'varchar(64) NOT NULL',
'userid' =>'int(11) NOT NULL',
'bizrule' =>'text',
'data' =>'text',
'PRIMARY KEY (`itemname`,`userid`)',
), 'ENGINE=InnoDB');

//the tbl_auth_assignment.itemname is a reference 
//to tbl_auth_item.name
$this->addForeignKey(
"fk_auth_assignment_itemname", 
"tbl_auth_assignment", 
"itemname", 
"tbl_auth_item", 
"name", 
"CASCADE", 
"CASCADE"
);

//the tbl_auth_assignment.userid is a reference 
//to tbl_user.id
$this->addForeignKey(
"fk_auth_assignment_userid", 
"tbl_auth_assignment", 
"userid", 
"tbl_user", 
"id", 
"CASCADE", 
"CASCADE"
);
}

public function down()
{
$this->truncateTable('tbl_auth_assignment');
$this->truncateTable('tbl_auth_item_child');
$this->truncateTable('tbl_auth_item');
$this->dropTable('tbl_auth_assignment');
$this->dropTable('tbl_auth_item_child');
$this->dropTable('tbl_auth_item');
}
```

保存这些更改后，运行迁移以创建所需的结构：

```php
**$ ./yiic migrate**

```

一旦必要的结构被创建，你会在屏幕上看到一个`成功迁移`的消息。

由于我们遵循了数据库表命名约定，我们需要修改我们的`authManager`组件配置，以指定我们特定的表名。打开`/protected/config/main.php`，并将表名规范添加到`authManager`组件中：

```php
// application components
'components'=>array(
…
'authManager'=>array(
'class'=>'CDbAuthManager',
'connectionID'=>'db',
'itemTable' =>'tbl_auth_item',
'itemChildTable' =>'tbl_auth_item_child',
'assignmentTable' =>'tbl_auth_assignment',
),
```

现在授权管理器组件将确切地知道我们希望它使用哪些表来管理我们的授权结构。

### 注意

如果你需要关于如何使用 Yii 数据库迁移的提醒，请参考第四章，*项目 CRUD*，这个概念是在那里首次介绍的。

## 创建 RBAC 授权层次结构

在我们的`trackstar`数据库中添加了这些表之后，我们需要用我们的角色和权限填充它们。我们将使用`authmanager`组件提供的 API 来做到这一点。为了保持简单，我们只会定义角色和基本操作。我们现在不会设置任何正式的 RBAC 任务。以下图显示了我们希望定义的基本层次结构：

![创建 RBAC 授权层次结构](img/8727_07_01.jpg)

该图显示了自上而下的继承关系。因此，所有者拥有所有在所有者框中列出的权限，同时继承来自成员和读者角色的所有权限。同样，成员继承自读者的权限。现在我们需要做的是在应用程序中建立这种权限层次结构。如前所述，实现这一点的一种方法是编写代码来利用`authManager` API。

使用 API 的示例代码如下，它创建了一个新角色和一个新操作，然后添加了角色和权限之间的关系：

```php
$auth=Yii::app()->authManager;  
$role=$auth->createRole('owner');
$auth->createOperation('createProject','create a new project');    
$role->addChild('createProject');
```

通过这段代码，我们首先获得了`authManager`的实例。然后我们使用它的`createRole()`、`createOperation()`和`addChild()`API 方法来创建一个新的`owner`角色和一个名为`createProject`的新操作。然后我们将权限添加到所有者角色。这只是演示了我们需要的层次结构的一小部分的创建；我们在前面的图表中概述的所有其余关系都需要以类似的方式创建。

我们可以创建一个新的数据库迁移，并将我们的代码放在那里以填充我们的权限层次结构。然而，为了演示在 Yii 应用程序中使用控制台命令，我们将采取不同的方法。我们将编写一个简单的 shell 命令，在命令行上执行。这将扩展我们用于创建初始应用程序的`yiic`命令行工具的命令选项。

### 编写控制台应用程序命令

我们在第二章*入门*中介绍了`yiic`命令行工具，当我们创建了一个新的“Hello, World!”应用程序时，以及在第四章*项目 CRUD*中，当我们用它来最初创建我们的 TrackStar web 应用程序的结构时。在创建和运行数据库迁移时，我们继续使用它。

`yiic`工具是 Yii 中的一个控制台应用程序，用于以命令形式执行任务。我们已经使用`webapp`命令创建新的应用程序，并使用`migrate`命令创建新的迁移文件并执行数据库迁移。Yii 中的控制台应用程序可以通过编写自定义命令轻松扩展，这正是我们要做的。我们将通过编写一个新的命令行工具来扩展`yiic`命令工具集，以便我们可以构建 RBAC 授权。

为控制台应用程序编写新命令非常简单。命令只是一个从`CConsoleCommand`扩展的类。它的工作方式类似于控制器类，它将解析输入的命令行选项，并将请求分派到命令类中指定的操作，其默认为`actionIndex()`。类的名称应该与所需的命令名称完全相同，后面跟着“Command”。在我们的情况下，我们的命令将简单地是“Rbac”，所以我们将我们的类命名为`RbacCommand`。最后，为了使这个命令可用于`yiic`控制台应用程序，我们需要将我们的类保存到`/protected/commands/`目录中，这是控制台命令的默认位置。

因此，创建一个新文件`/protected/commands/RbacCommand.php`。这个文件的内容太长，无法包含在内，但可以从本章的可下载代码或[gist.github.com/jeffwinesett](http://gist.github.com/jeffwinesett)中轻松获取。这个代码片段可以在[`gist.github.com/3779677`](https://gist.github.com/3779677)中找到。

可下载代码中的注释应该有助于讲述这里发生的事情。我们重写了`getHelp()`的基类实现，以添加一个额外的描述行。我们将在一分钟内展示如何显示帮助。所有真正的操作都发生在我们添加的两个操作`actionIndex()`和`actionDelete()`中。前者创建我们的 RBAC 层次结构，后者删除它。它们都确保应用程序有一个定义的有效`authManager`应用程序组件。然后，这两个操作允许用户在继续之前有最后一次取消请求的机会。如果使用此命令的用户表示他们想要继续，请求将继续。我们的两个操作都将继续清除 RBAC 表中先前输入的所有数据，而`actionIndex()`方法将创建一个新的授权层次结构。这里创建的层次结构正是我们之前讨论的那个。

我们可以看到，即使基于我们相当简单的层次结构，仍然需要大量的代码。通常，需要开发一个更直观的**图形用户界面**（**GUI**）来包装这些授权管理器 API，以提供一个易于管理角色、任务和操作的界面。我们在这里采取的方法是建立快速 RBAC 权限结构的好解决方案，但不适合长期维护可能会发生重大变化的权限结构。

### 注意

在现实世界的应用程序中，您很可能需要一个不同的、更交互式的工具来帮助维护 RBAC 关系。Yii 扩展库（[`www.yiiframework.com/extensions/`](http://www.yiiframework.com/extensions/)）提供了一些打包的解决方案。

有了这个文件，如果我们现在询问`yiic`工具帮助，我们将看到我们的新命令作为可用选项之一：

![编写控制台应用程序命令](img/8727_07_03.jpg)

我们的`rbac`显示在列表中。但是，在我们尝试执行之前，我们需要为控制台应用程序配置`authManager`。您可能还记得，运行控制台应用程序时，会加载不同的配置文件，即`/protected/config/console.php`。我们需要在这个文件中添加与之前添加到`main.php`配置文件相同的`authManager`组件。打开`console.php`并将以下内容添加到组件列表中：

```php
'authManager'=>array(
'class'=>'CDbAuthManager',
'connectionID'=>'db',
'itemTable' =>'tbl_auth_item',
'itemChildTable' =>'tbl_auth_item_child',
'assignmentTable' =>'tbl_auth_assignment',
),
```

有了这个，我们现在可以尝试我们的新命令：

![编写控制台应用程序命令](img/8727_07_04.jpg)

这正是我们在命令类的`getHelp()`方法中添加的帮助文本。您当然可以更详细地添加更多细节。让我们实际运行命令。由于`actionIndex()`是默认值，我们不必指定操作：

![编写控制台应用程序命令](img/8727_07_05.jpg)

我们的命令已经完成，并且我们已经向新的数据库表中添加了适当的数据，以生成我们的授权层次结构。

由于我们还添加了一个`actionDelete()`方法来删除我们的层次结构，您也可以尝试一下：

```php
**$ ./yiic rbac delete**

```

在尝试这些操作完成后，确保再次运行命令以添加层次结构，因为我们需要它继续存在。

## 分配用户到角色

到目前为止，我们所做的一切都建立了一个授权层次结构，但尚未为用户分配权限。我们通过将用户分配到我们创建的三个角色之一，*owner*、*member*或*reader*来实现这一点。例如，如果我们想要将唯一用户 ID 为`1`的用户与`member`角色关联，我们将执行以下操作：

```php
**$auth=Yii::app()->authManager;**
**$auth->assign('member',1);**

```

一旦建立了这些关系，检查用户的访问权限就变得很简单。我们只需询问应用程序用户组件当前用户是否具有权限。例如，如果我们想要检查当前用户是否被允许创建新问题，我们可以使用以下语法：

```php
if( Yii::app()->user->checkAccess('createIssue'))
{
     //perform needed logic
}
```

在这个例子中，我们将用户 ID `1`分配给`成员`角色，由于在我们的授权层次结构中，成员角色继承了`createIssue`权限，假设我们以用户`1`的身份登录到应用程序中，这个`if()`语句将评估为`true`。

我们将在向项目添加新成员时添加此授权分配逻辑作为业务逻辑的一部分。我们将添加一个新表单，允许我们将用户添加到项目中，并在此过程中选择角色。但首先，我们需要解决用户角色需要在每个项目基础上实施的另一个方面。

## 在每个项目基础上为用户添加 RBAC 角色

我们现在已经建立了一个基本的 RBAC 授权模型，但这些关系适用于整个应用程序。TrackStar 应用程序的需求稍微复杂一些。我们需要在项目的上下文中为用户分配角色，而不仅仅是在整个应用程序中全局地分配。我们需要允许用户在不同的项目中担任不同的角色。例如，用户可能是一个项目的“读者”角色，第二个项目的“成员”角色，以及第三个项目的“所有者”角色。用户可以与许多项目相关联，并且他们被分配的角色需要特定于项目。

Yii 中的 RBAC 框架没有内置的东西可以满足这个要求。RBAC 模型只旨在建立角色和权限之间的关系。它不知道（也不应该知道）我们的 TrackStar 项目的任何信息。为了实现我们授权层次结构的这个额外维度，我们需要改变我们的数据库结构，以包含用户、项目和角色之间的关联。如果您还记得第五章中的内容，*管理问题*，我们已经创建了一个名为`tbl_project_user_assignment`的表，用于保存用户和项目之间的关联。我们可以修改这个表，以包含用户在项目中分配的角色。我们将添加一个新的迁移来修改我们的表：

```php
**$ cd /Webroot/trackstar/protected/**
**$ ./yiic migrate create add_role_to_tbl_project_user_assignment**

```

现在打开新创建的迁移文件，并实现以下`up()`和`down()`方法：

```php
public function up()
{
$this->addColumn('tbl_project_user_assignment', 'role', 'varchar(64)');
//the tbl_project_user_assignment.role is a reference 
     //to tbl_auth_item.name
$this->addForeignKey('fk_project_user_role', 'tbl_project_user_assignment', 'role', 'tbl_auth_item', 'name', 'CASCADE', 'CASCADE');
}

public function down()
{
$this->dropForeignKey('fk_project_user_role', 'tbl_project_user_assignment');
$this->dropColumn('tbl_project_user_assignment', 'role');
}
```

最后运行迁移：

![在每个项目基础上为用户添加 RBAC 角色](img/8727_07_06.jpg)

您将在屏幕底部看到消息“成功迁移”。

现在我们的表已经设置好，可以允许我们进行角色关联以及用户和项目之间的关联。

### 添加 RBAC 业务规则

虽然之前显示的数据库表将保存基本信息，以回答用户是否在特定项目的上下文中被分配了角色的问题，但我们仍然需要我们的 RBAC`auth`层次结构来回答关于用户是否有权限执行某个功能的问题。尽管 Yii 中的 RBAC 模型不知道我们的 TrackStar 项目，但它具有一个非常强大的功能，我们可以利用它。当您创建授权项或将项分配给用户时，您可以关联一小段 PHP 代码，该代码将在`Yii::app()->user->checkAccess()`调用期间执行。一旦定义，这段代码必须在用户被授予权限之前返回`true`。

这个功能的一个例子是在允许用户维护个人资料信息的应用程序中。在这种情况下，应用程序希望确保用户只有权限更新自己的个人资料信息，而不是其他人的。在这种情况下，我们可以创建一个名为“updateProfile”的授权项，然后关联一个业务规则，检查当前用户的 ID 是否与与个人资料信息相关联的用户 ID 相同。

在我们的情况下，我们将为角色分配关联一个业务规则。当我们将用户分配给特定角色时，我们还将关联一个业务规则，该规则将在项目的上下文中检查关系。`checkAccess()`方法还允许我们传递一个附加参数数组，供业务规则使用以执行其逻辑。我们将使用这个来传递当前项目上下文，以便业务规则可以调用`Project` AR 类的方法，以确定用户是否在该项目中被分配到该角色。

我们将为每个角色分配创建稍有不同的业务规则。例如，当将用户分配给所有者角色时，我们将使用以下规则：

```php
$bizRule='return isset($params["project"]) && $params["project"]->isUserInRole("owner");';
```

角色`成员`和`读者`的方法将会相似。

当我们调用`checkAccess()`方法时，我们还需要传递项目上下文。因此，现在在检查用户是否有权限执行例如`createIssue`操作时，代码将如下所示：

```php
//add the project AR instance to the input params
$params=array('project'=>$project);
//pass in the params to the checkAccess call
if(Yii::app()->user->checkAccess('createIssue',$params))
{
     //proceed with issue creation logic
}
```

在前面的代码中，`$project`变量是与当前项目上下文相关联的`Project` AR 类实例（请记住，我们应用程序中的几乎所有功能都发生在项目的上下文中）。这个类实例是业务规则中使用的。业务规则调用`Project::isUserInRole()`方法，以确定用户是否在特定项目的角色中。

### 实现新的项目 AR 方法

现在我们已经修改了数据库结构，以容纳用户、角色和项目之间的关系，我们需要实现所需的逻辑来管理和验证该表中的数据。我们将在项目 AR 类中添加公共方法，以处理从该表中添加和删除数据以及验证行的存在。

我们需要在`Project` AR 类中添加一个公共方法，该方法将接受角色名称和用户 ID，并创建角色、用户和项目之间的关联。打开`protected/models/Project.php`文件，并添加以下方法：

```php
public function assignUser($userId, $role)
{
$command = Yii::app()->db->createCommand();
$command->insert('tbl_project_user_assignment', array(
'role'=>$role,
'user_id'=>$userId,
'project_id'=>$this->id,
));
}
```

在这里，我们使用 Yii 框架的查询构建器方法直接插入数据库表，而不是使用活动记录方法。由于`tbl_project_user_assignement`只是一个关联表，并不代表我们模型的主要领域对象，因此有时更容易以更直接的方式管理这些类型表中的数据，而不是使用活动记录方法。

### 注意

有关在 Yii 中使用查询构建器的更多信息，请访问：

[`www.yiiframework.com/doc/guide/1.1/en/database.query-builder`](http://www.yiiframework.com/doc/guide/1.1/en/database.query-builder)

我们还需要能够从项目中删除用户，并在这样做时，删除用户和项目之间的关联。因此，让我们也添加一个执行此操作的方法。

在`Project` AR 类中添加以下方法：

```php
public function removeUser($userId)
{
$command = Yii::app()->db->createCommand();
$command->delete(
'tbl_project_user_assignment', 
'user_id=:userId AND project_id=:projectId', 
array(':userId'=>$userId,':projectId'=>$this->id));
}
```

这只是从包含角色、用户和项目之间关联的表中删除行。

我们现在已经实现了添加和删除关联的方法。我们需要添加功能来确定给定用户是否与项目内的角色相关联。我们还将这作为公共方法添加到我们的`Project` AR 类中。

在`Project` AR 模型类的底部添加以下方法：

```php
public function allowCurrentUser($role)
{
$sql = "SELECT * FROM tbl_project_user_assignment WHERE project_id=:projectId AND user_id=:userId AND role=:role";
$command = Yii::app()->db->createCommand($sql);
$command->bindValue(":projectId", $this->id, PDO::PARAM_INT);
$command->bindValue(":userId", Yii::app()->user->getId(), PDO::PARAM_INT);
$command->bindValue(":role", $role, PDO::PARAM_STR);
return $command->execute()==1;
}
```

该方法展示了如何直接执行 SQL，而不是使用查询构建器。查询构建器非常有用，但对于简单的查询，直接执行 SQL 有时更容易，利用 Yii 的数据访问对象（DAO）。

### 注意

有关 Yii 的数据访问对象和在 Yii 中直接执行 SQL 的更多信息，请参阅：

[`www.yiiframework.com/doc/guide/1.1/en/database.dao`](http://www.yiiframework.com/doc/guide/1.1/en/database.dao)

## 将用户添加到项目中

现在我们需要把所有这些放在一起。在第六章中，*用户管理和授权*中，我们添加了创建应用程序新用户的功能。然而，我们还没有办法将用户分配给特定的项目，并进一步将他们分配到这些项目中的角色。现在我们已经有了 RBAC 方法，我们需要构建这个新功能。

这个功能的实现涉及几个编码更改。然而，我们已经提供了类似的需要的更改的示例，并在之前的章节中涵盖了所有相关的概念。因此，我们将快速地进行这个过程，并且只是简要地强调一些我们还没有看到的东西。此时，读者应该能够在没有太多帮助的情况下进行所有这些更改，并被鼓励以实践的方式这样做。为了进一步鼓励这种练习，我们将首先列出我们要做的一切来满足这个新的功能需求。然后你可以关闭书本，在查看我们的实现之前尝试一些这样的操作。

为了实现这个目标，我们将执行以下操作：

1.  在`Project`模型类中添加一个名为`getUserRoleOptions()`的新公共静态方法，该方法使用`auth`管理器的`getRoles()`方法返回一个有效的角色选项列表。我们将使用这个方法来填充表单中的角色选择下拉字段，以便在向项目添加新用户时选择用户角色。

1.  在`Project`模型类中添加一个名为`isUserInProject($user)`的新公共方法，以确定用户是否已经与项目关联。我们将在表单提交时使用这个方法来进行验证规则，以便我们不会尝试将重复的用户添加到项目中。

1.  添加一个名为`ProjectUserForm`的新表单模型类，继承自`CFormModel`，用于新的输入表单模型。在这个表单模型类中添加三个属性，即`$username`、`$role`和`$project`。还要添加验证规则，以确保用户名和角色都是必需的输入字段，并且用户名应该通过自定义的`verify()`类方法进行进一步验证。

这个验证方法应该尝试通过查找与输入用户名匹配的用户来创建一个新的 UserAR 类实例。如果尝试成功，它应该继续使用我们之前添加的`assignUser($userId, $role)`方法将用户关联到项目。我们还需要在本章前面实现的 RBAC 层次结构中将用户与角色关联起来。如果没有找到与用户名匹配的用户，它需要设置并返回一个错误。（如果需要，可以查看`LoginForm::authenticate()`方法作为自定义验证规则方法的示例。）

1.  在 views/project 下添加一个名为`adduser.php`的新视图文件，用于显示我们向项目添加用户的新表单。这个表单只需要两个输入字段，*用户名*和*角色*。角色应该是一个下拉选择列表。

1.  在`ProjectController`类中添加一个名为`actionAdduser()`的新控制器动作方法，并修改其`accessRules()`方法以确保经过身份验证的成员可以访问它。这个新的动作方法负责呈现新的视图来显示表单，并在提交表单时处理后退。

再次鼓励读者首先尝试自己进行这些更改。我们在以下部分列出了我们的代码更改。

### 修改项目模型类

对于`Project`类，我们添加了两个新的公共方法，其中一个是静态的，因此可以在不需要特定类实例的情况下调用：

```php
   /**
 * Returns an array of available roles in which a user can be placed when being added to a project
 */
public static function getUserRoleOptions()
{
return CHtml::listData(Yii::app()->authManager->getRoles(), 'name', 'name');
} 

/* 
 * Determines whether or not a user is already part of a project
 */
public function isUserInProject($user) 
{
$sql = "SELECT user_id FROM tbl_project_user_assignment WHERE project_id=:projectId AND user_id=:userId";
$command = Yii::app()->db->createCommand($sql);
$command->bindValue(":projectId", $this->id, PDO::PARAM_INT);
$command->bindValue(":userId", $user->id, PDO::PARAM_INT);
return $command->execute()==1;
}
```

### 添加新的表单模型类

就像在登录表单的方法中使用的那样，我们将创建一个新的表单模型类，作为存放我们的表单输入参数和集中验证的中心位置。这是一个相当简单的类，它继承自 Yii 类`CFormModel`，并具有映射到我们表单输入字段的属性，以及一个用于保存有效项目上下文的属性。我们需要项目上下文来能够向项目添加用户。整个类太长了，无法在这里列出，但可以轻松从本章附带的可下载代码中获取。独立的代码片段可以在[`gist.github.com/3779690`](http:// https://gist.github.com/3779690)上找到。

在下面的代码片段中，我们列出了我们以前没有见过的部分：

```php
class ProjectUserForm extends CFormModel
{
…
      public function assign()
{
if($this->_user instanceof User)
{
//assign the user, in the specified role, to the project
$this->project->assignUser($this->_user->id, $this->role);  
//add the association, along with the RBAC biz rule, to our RBAC hierarchy
        $auth = Yii::app()->authManager; 
$bizRule='return isset($params["project"]) && $params["project"]->allowCurrentUser("'.$this->role.'");';  
$auth->assign($this->role,$this->_user->id, $bizRule);
                  return true;
}
            else
{
$this->addError('username','Error when attempting to assign this user to the project.'); 
return false;
}
      }
```

### 注意

为了简单起见，在`createUsernameList()`方法中，我们选择从数据库中选择*所有*用户来用于用户名列表。如果有大量用户，这可能会导致性能不佳。为了优化性能，在用户数量较多的情况下，您可能需要对其进行过滤和限制。

我们在可下载的代码部分中列出的部分是`assign()`方法，我们在其中为用户和角色之间的关联添加了一个 bizRule：

```php
$auth = Yii::app()->authManager; 
$bizRule='return isset($params["project"]) && $params["project"]->isUserInRole("'.$this->role.'");';
$auth->assign($this->role,$user->id, $bizRule);
```

我们创建了一个`Authmanager`类的实例，用于建立用户与角色的分配。然而，在进行分配之前，我们需要创建业务规则。业务规则使用`$params`数组，首先检查数组中是否存在`project`元素，然后在项目 AR 类上调用`isUserInRole()`方法，该方法是该数组元素的值。我们明确向这个方法传递角色名。然后我们调用`AuthManager::assign()`方法来建立用户与角色之间的关联。

我们还添加了一个简单的公共方法`createUsernameList()`，返回数据库中所有用户名的数组。我们将使用这个数组来填充 Yii 的 UI 小部件`CJuiAutoComplete`的数据，我们将用它来填充用户名输入表单元素。正如它的名字所示，当我们在输入表单字段中输入时，它将根据这个数组中的元素提供选择建议。

### 向项目控制器添加新的动作方法

我们需要一个控制器动作来处理显示向项目添加新用户的表单的初始请求。我们将其放在`ProjectController`类中，并命名为`actionAdduser()`。其代码如下：

```php
     /**
 * Provides a form so that project administrators can
 * associate other users to the project
 */
public function actionAdduser($id)
{
  $project = $this->loadModel($id);
  if(!Yii::app()->user->checkAccess('createUser', array('project'=>$project)))
{
  throw new CHttpException(403,'You are not authorized to perform this action.');
}

  $form=new ProjectUserForm; 
  // collect user input data
  if(isset($_POST['ProjectUserForm']))
  {
    $form->attributes=$_POST['ProjectUserForm'];
    $form->project = $project;
    // validate user input  
    if($form->validate())  
    {
        if($form->assign())
      {
       Yii::app()->user->setFlash('success',$form->username . " has been added to the project." ); 
       //reset the form for another user to be associated if desired
      $form->unsetAttributes();
      $form->clearErrors();
      }
    }
  }
$form->project = $project;
$this->render('adduser',array('model'=>$form)); 
}
```

这对我们来说都很熟悉。它处理了显示表单的初始`GET`请求，以及表单提交后的`POST`请求。它非常类似于我们的`SiteController::actionLogin()`方法。然而，在上一个代码片段中突出显示的代码是我们以前没有见过的。如果提交的表单请求成功，它会设置一个称为**flash message**的东西。Flash message 是一个临时消息，暂时存储在会话中。它只在当前和下一个请求中可用。在这里，我们使用我们的`CWebUser`应用用户组件的`setFlash()`方法来存储一个临时消息，表示请求成功。当我们在下一节讨论视图时，我们将看到如何访问此消息并将其显示给用户。

我们需要做的另一个更改是基本控制器类方法`Controller::accessRules()`。您还记得，我们将访问规则添加到这个基类中，以便它们适用于我们的每个用户、问题和项目控制器类。我们需要将这个新动作名称添加到基本访问规则列表中，以便允许已登录用户访问此动作：

```php
public function accessRules()
{
return array(
array('allow',  // allow all users to perform 'index' and 'view' actions
'controllers'=>array('issue','project','user'),
'actions'=>array('index','view',**'addUser'**),
'users'=>array('@'),
),
```

### 添加新的视图文件来显示表单

我们的新动作方法调用`->render('adduser')`来渲染一个视图文件，所以我们需要创建一个。以下是我们对`protected/views/project/adduser.php`的实现的完整列表：

```php
<?php
$this->pageTitle=Yii::app()->name . ' - Add User To Project';
$this->breadcrumbs=array(
$model->project->name=>array('view','id'=>$model->project->id),
'Add User',
);
$this->menu=array(
array('label'=>'Back To Project', 'url'=>array('view','id'=>$model->project->id)),
);
?>

<h1>Add User To <?php echo $model->project->name; ?></h1>

**<?php if(Yii::app()->user->hasFlash('success')):?>**
**<div class="successMessage">**
**<?php echo Yii::app()->user->getFlash('success'); ?>**
**</div>**
**<?phpendif; ?>**

<div class="form">
<?php $form=$this->beginWidget('CActiveForm'); ?>

<p class="note">Fields with <span class="required">*</span> are required.</p>

<div class="row">
<?php echo $form->labelEx($model,'username'); ?>
<?php
$this->widget('zii.widgets.jui.CJuiAutoComplete', array(
'name'=>'username',
'source'=>$model->createUsernameList(),
'model'=>$model,
'attribute'=>'username',
'options'=>array(
'minLength'=>'2',
),
'htmlOptions'=>array(
'style'=>'height:20px;'
),
));
?>
<?php echo $form->error($model,'username'); ?>
</div>

<div class="row">
<?php echo $form->labelEx($model,'role'); ?>
<?php
echo $form->dropDownList($model,'role', 
Project::getUserRoleOptions()); ?>
<?php echo $form->error($model,'role'); ?>
</div>

<div class="row buttons">
<?php echo CHtml::submitButton('Add User'); ?>
</div>

<?php $this->endWidget(); ?>
</div>
```

我们以前大部分都见过了。我们正在定义活动标签和活动表单元素，这些元素直接与我们的`ProjectUserForm`表单模型类相关联。我们使用我们在项目模型类上早期实施的静态方法填充下拉菜单。我们使用`createUsernameList()`方法填充我们的 Zii 库自动完成小部件（`CJuiAutoComplete`）数据，该方法已添加到项目用户表单模型类中。我们还在菜单选项中添加了一个简单的链接，以便返回到项目详细信息页面。

在上一个代码片段中突出显示的代码对我们来说是新的。这是一个示例，说明了我们在`actionAdduser()`方法中引入并使用的闪烁消息。我们通过询问同一用户组件是否有闪烁消息（使用`hasFlash('succcess')`）来访问我们使用`setFlash()`设置的消息。我们向`hasFlash()`方法提供了我们在设置消息时给它的确切名称。这是向用户提供有关其先前请求的一些简单反馈的好方法。

我们做的另一个小改变是在项目详细信息页面中添加了一个简单的链接，以便我们可以从应用程序中访问它。以下突出显示的行已添加到项目`view.php`视图文件的菜单数组中：

```php
$this->menu=array(
…
array('label'=>'Add User To Project', 'url'=>array('project/adduser', 'id'=>$model->id)),
);
```

这使我们在查看项目详细信息时可以访问新表单。

### 将所有内容放在一起

有了所有这些变化，我们可以通过查看项目详细信息页面之一来导航到我们的新表单。例如，当通过`http://localhost/trackstar/index.php?r=project/view&id=1`查看项目 ID＃1 时，在右侧列操作菜单中有一个超链接**[将用户添加到项目]**，单击该链接应显示以下页面：

![将所有内容放在一起](img/8727_07_02.jpg)

您可以使用我们以前构建的表单来创建新项目和用户，以确保将其中一些添加到应用程序中。然后，您可以尝试将用户添加到项目中。当您在**用户名**字段中输入时，您将看到自动完成的建议。如果您尝试添加一个不在用户数据库表中的用户，您应该会看到一个告诉您的错误。如果您尝试输入已添加到项目中的用户，您将收到一个告诉您的错误。在成功添加后，您将看到一个指示成功的简短闪烁消息。

现在我们有了将用户分配给项目并将它们添加到我们的 RBAC 授权层次结构的能力，我们应该改变我们添加新项目时的逻辑。添加新项目时，应将添加项目的用户分配为项目的“所有者”。这样，项目的创建者将对项目拥有完全的管理访问权限。我将把这留给读者作业。您可以通过下载附带本书的 TrackStar 应用程序的可用源代码来查看此练习的解决方案。

# 检查授权级别

完成本章中我们设定的任务的最后一件事是为我们实现的不同功能添加授权检查。在本章的早些时候，我们概述并实施了我们拥有的不同角色的 RBAC 授权层次结构。一切都已准备就绪，以允许或拒绝基于已授予项目内用户的权限的功能访问，但有一个例外。当尝试请求功能时，我们尚未实施必要的授权检查。该应用程序仍在使用在我们的项目、问题和用户控制器上定义的简单访问过滤器。我们将为我们的权限之一执行此操作，然后将其余实现留给读者作为练习。

回顾我们的授权层次结构，我们可以看到只有项目所有者才能向项目添加新用户。因此，让我们添加这个授权检查。除非当前用户在该项目的*owner*角色中，否则我们将隐藏项目详情页面上添加用户的链接（在实施之前，您应该确保您已经向项目添加了至少一个所有者和一个成员或读者，以便在完成后进行测试）。打开`protected/views/project/view.php`视图文件，在那里我们放置了添加新用户的菜单项。从菜单数组项中删除该数组元素，然后只有当`checkAccess()`方法返回`true`时，才将其推送到数组的末尾。以下代码显示了菜单项应该如何定义：

```php
$this->menu=array(
array('label'=>'List Project', 'url'=>array('index')),
array('label'=>'Create Project', 'url'=>array('create')),
array('label'=>'Update Project', 'url'=>array('update', 'id'=>$model->id)),
array('label'=>'Delete Project', 'url'=>'#', 'linkOptions'=>array('submit'=>array('delete','id'=>$model->id),'confirm'=>'Are you sure you want to delete this item?')),
array('label'=>'Manage Project', 'url'=>array('admin')),
array('label'=>'Create Issue', 'url'=>array('issue/create', 'pid'=>$model->id)),

);
if(Yii::app()->user->checkAccess('createUser',array('project'=>$model)))
{
$this->menu[] = array('label'=>'Add User To Project', 'url'=>array('adduser', 'id'=>$model->id));
}
```

这实现了我们在本章中讨论过的相同方法。我们在当前用户上调用`checkAccess()`并发送我们想要检查的权限的名称。此外，由于我们的角色是在项目的上下文中的，我们将项目模型实例作为数组输入发送。这将允许已在授权分配中定义的业务规则执行。现在，如果我们以特定项目的项目所有者身份登录，并导航到该项目的详情页面，我们将看到添加新用户到项目的菜单选项。相反，如果我们以同一项目的`member`或`reader`角色的用户身份登录，并再次导航到详情页面，这个链接将不会显示。

当然，这并不会阻止一个精明的用户通过直接使用 URL 导航来获得这个功能。例如，即使作为项目＃1 的`reader`角色的用户登录到应用程序，如果我直接导航到`http://localhost/trackstar/index.php?r=project/adduser&id=1`，我仍然可以访问表单。

为了防止这种情况发生，我们需要直接将我们的访问检查添加到动作方法本身。因此，在项目控制器类中的`ProjectController::actionAdduser()`方法中，我们可以添加检查：

```php
public function actionAdduser($id)
{
$project = $this->loadModel($id);
if(!Yii::app()->user->checkAccess('createUser', array('project'=>$project)))
{
throw new CHttpException(403,'You are not authorized to perform this action.');
}

$form=new ProjectUserForm; 
```

现在，当我们尝试直接访问这个 URL 时，除非我们是项目的*owner*角色，否则我们将被拒绝访问。

我们不会逐个实现所有其他功能的访问检查。每个都将以类似的方式实现。我们把这留给读者作为一个练习。这个实现对于继续跟随本书中剩余的代码示例并不是必需的。

# 总结

在本章中，我们涵盖了很多内容。首先，我们介绍了 Yii 提供的基本访问控制过滤器，作为允许和拒绝对特定控制器动作方法访问的一种方法。我们使用这种方法来确保用户在获得任何主要功能的访问权限之前必须登录到该应用程序。然后，我们详细介绍了 Yii 的 RBAC 模型，它允许更复杂的访问控制方法。我们基于应用程序角色构建了整个用户授权层次结构。在这个过程中，我们介绍了在 Yii 中编写控制台应用程序，并介绍了这一出色功能的一些好处。然后，我们增加了新功能，允许向项目添加用户，并能够将他们分配到这些项目中的适当角色。最后，我们发现了如何在整个应用程序中实现所需的访问检查，以利用 RBAC 层次结构来适当地授予/拒绝功能功能的访问权限。

在下一章中，我们将为用户添加更多功能，其中之一是能够在我们的项目问题上留下评论。
