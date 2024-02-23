# 第四章。项目 CRUD

现在我们已经有了一个基本的应用程序，并配置好与我们的数据库通信，我们可以开始着手一些我们应用程序的真正功能。我们知道"项目"是我们应用程序中最基本的组件之一。用户在 TrackStar 应用程序中不能做任何有用的事情，而不是首先创建或选择一个现有的项目，然后在其中添加任务和其他问题。因此，我们首先要把注意力转向将一些项目功能加入应用程序。

# 功能规划

在本章的努力结束时，我们的应用程序应该允许用户创建新项目，从现有项目列表中选择，更新/编辑现有项目，并删除现有项目。

为了实现这个目标，我们应该确定更加细粒度的任务来关注。下面的列表确定了我们在本章内的任务清单：

+   设计数据库架构以支持项目

+   构建架构中标识的必要表和所有其他数据库对象

+   创建 Yii AR 模型类，以便应用程序可以轻松地与创建的数据库表进行交互

+   创建 Yii 控制器类，用于包含以下功能：

+   创建新项目

+   检索现有项目列表以显示

+   更新与现有项目相关的数据

+   删除现有项目

+   创建 Yii 视图文件和表示层逻辑，将：

+   展示表单以允许创建新项目

+   显示所有现有项目的列表

+   显示表单以允许用户编辑现有项目

+   在项目列表中添加删除按钮，以允许删除项目

这绝对足够让我们开始了。

# 创建项目表

回到第三章 *TrackStar 应用程序*，我们谈到了代表项目的基本数据，并且我们决定我们将使用 MySQL 关系数据库来构建这个应用程序的持久层。现在我们需要设计和构建将持久化我们项目数据的表。

我们知道项目需要有名称和描述。我们还将在每个表上保留一些基本的表审计信息，通过跟踪记录创建和更新的时间，以及谁创建和更新记录。

基于这些属性，项目表将如下所示：

```php
CREATE TABLE tbl_`project` (
`id` INTEGER NOT NULL auto_increment,
`name` varchar(255) NOT NULL,
`description` text NOT NULL,
`create_time` DATETIME default NULL,
`create_user_id` INTEGER default NULL,
`update_time` DATETIME default NULL,
`update_user_id` INTEGER default NULL,
PRIMARY KEY  (`id`)
) ENGINE = InnoDB
;
```

现在，在我们直接使用我们喜爱的 MySQL 数据库编辑器来创建这个表之前，我们需要讨论一下我们如何使用 Yii 来管理我们的数据库架构的变化，因为我们构建我们的 TrackStar 应用程序时会发生变化。

## Yii 数据库迁移

我们知道跟踪应用程序源代码的版本更改是一个好习惯。当您在构建我们的 TrackStar 应用程序时，使用版本控制软件如 SVN 或 GIT 来帮助管理我们在代码库中所做的所有更改是明智的。如果我们的代码库更改与数据库更改不同步，很可能我们整个应用程序都会崩溃。因此，管理我们将在数据库中进行的结构更改也是非常重要的。

Yii 在这方面帮助了我们。Yii 提供了一个数据库迁移工具，用于跟踪数据库迁移历史，并允许我们应用新的迁移，以及回滚现有的迁移，以便我们将数据库结构恢复到先前的状态。

Yii 迁移实用程序是一个控制台命令，我们使用`yiic`命令行工具。作为控制台命令，它使用一个特定于控制台命令的配置文件，默认情况下是`protected/config/console.php`。我们需要在这个文件中正确配置我们的数据库组件。就像我们在`main.php`配置文件中所做的那样，我们需要定义我们的`db`组件来使用我们的 MySQL 数据库。如果你打开`protected/config/console.php`配置文件，你会看到它已经定义了一个 MySQL 配置，但是被注释掉了。让我们删除 SQLite 配置并取消注释 MySQL 配置，根据你的数据库设置更改用户名和密码：

```php
'components'=>array(	
'db'=>array(
    'connectionString' => 'mysql:host=localhost;dbname=trackstar',
    'emulatePrepare' => true,
    'username' => '[YOUR-USERNAME]',
    'password' => '[YOUR-PASSWORD]',
    'charset' => 'utf8',
  ),
),
```

现在我们已经完成了配置更改，可以继续创建迁移。为此，我们使用`yiic`命令行实用工具和`migrate`命令。创建迁移的一般形式如下：

```php
**$ yiic migrate create <name>**

```

在这里，必需的`name`参数允许我们指定我们正在进行的数据库更改的简要描述。`name`参数用作迁移文件名和 PHP 类名的一部分。因此，它应该只包含字母、数字或下划线字符。Yii 接受输入的名称参数，并附加一个 UTC 时间戳（格式为*yymmdd_hhmmss*），并在其后加上字母*m*以用作文件名和 PHP 类名。让我们继续为我们的项目表创建一个新的迁移，这个命名约定将更加清晰。从命令行中，导航到应用程序的`protected/`目录，然后发出使用名称`create_project_table`创建新迁移的命令：

![Yii 数据库迁移](img/8727_04_17.jpg)

这将创建文件`/Webroot/trackstar/protected/migrations/m121108_195611_create_project_table.php`，内容如下：

```php
class m121108_195611_create_project_table extends CDbMigration
{
  public function up()
  {
  }

  public function down()
  {
    echo "m121108_195611_create_project_table does not support migration down.\n";
    return false;
  }

  /*
  // Use safeUp/safeDown to do migration with transaction
  public function safeUp()
  {
  }

  public function safeDown()
  {
  }
  */
}
```

当然，我们将不得不对这个文件进行一些更改，以便它创建我们的新表。我们实现`up()`方法来应用我们所需的数据库更改，并实现`down()`方法来撤消这些更改，这将允许我们恢复到数据库结构的先前版本。`safeUp()`和`safeDown()`方法类似，但它们将在数据库事务中执行更改，以便将整个迁移作为原子单元以一种全有或全无的方式执行。在这种情况下，我们要应用的更改是创建一个新表，我们可以通过删除表来撤消这些更改。这些更改如下：

```php
public function up()
{
  $this->createTable('tbl_project', array(
    'id' => 'pk',
     'name' => 'string NOT NULL',
    'description' => 'text NOT NULL',
    'create_time' => 'datetime DEFAULT NULL',
    'create_user_id' => 'int(11) DEFAULT NULL',
    'update_time' => 'datetime DEFAULT NULL',
    'update_user_id' => 'int(11) DEFAULT NULL',
  ), 'ENGINE=InnoDB');
}

public function down()
{
  $this->dropTable('tbl_project');
}
```

保存更改后，我们可以执行迁移。在`protected/`目录中，执行以下迁移：

![Yii 数据库迁移](img/8727_04_18.jpg)

使用不带参数的迁移命令将导致对尚未应用的每个迁移执行迁移（即执行`up()`方法）。而且，由于这是我们第一次运行迁移，Yii 将自动为我们创建一个新的迁移历史表`tbl_migration`。Yii 使用此表来跟踪已经应用的迁移。如果我们在迁移命令的命令行参数中指定*down*，则将通过运行该迁移的`down()`方法来撤消最后应用的迁移。

现在我们已经应用了迁移，我们的新的`tbl_project`表已经被创建并准备好供我们使用。

### 注意

在开发我们的 TrackStar 应用程序时，我们将在整本书中使用 Yii 迁移，因此在使用它们时我们将继续学习更多关于它们的知识。有关 Yii 迁移的更详细信息，请参见：

[`www.yiiframework.com/doc/guide/1.1/en/database.migration`](http://www.yiiframework.com/doc/guide/1.1/en/database.migration)

## 命名约定

您可能已经注意到我们将数据库表以及所有列名都定义为小写。在整个开发过程中，我们将使用小写来表示所有表名和列名。这主要是因为不同的 DBMS 以不同的方式处理大小写敏感性。例如，**PostgreSQL**默认情况下将列名视为不区分大小写，如果列包含混合大小写字母，则必须在查询条件中引用列。使用小写将有助于消除这个问题。

您可能还注意到我们在命名项目表时使用了`tbl_`前缀。从 1.1.0 版本开始，Yii 提供了对表前缀的集成支持。表前缀是一个字符串，它被添加到表的名称之前。它经常用于共享托管环境，其中多个应用程序共享一个单一的数据库，并使用不同的表前缀来区分彼此；一种数据库对象的名称空间。例如，一个应用程序可以使用`tbl_`作为前缀，而另一个应用程序可以使用`yii_`。此外，一些数据库管理员使用这个作为一个命名约定，以前缀数据库对象的标识符，以确定它们是什么类型的实体或者将要使用。他们使用前缀来帮助将对象组织成相似的组。使用表前缀是一种偏好，当然不是必需的。

为了充分利用 Yii 中集成的表前缀支持，必须适当地设置`CDbConnection::tablePrefix`属性为所需的表前缀。然后，在整个应用程序中使用的 SQL 语句中，可以使用`{{TableName}}`来引用表名，其中`TableName`是表的名称，但不包括前缀。例如，如果我们要进行这个配置更改，我们可以使用以下代码来查询所有项目：

```php
$sql='SELECT * FROM {{project}}';
$projects=Yii::app()->db->createCommand($sql)->queryAll();
```

但这有点超前。让我们暂时保持配置不变，等到我们稍后在应用程序开发中进行数据库查询时再回顾这个话题。

# 创建 AR 模型类

现在我们已经创建了`tbl_project`表，我们需要创建 Yii 模型类，以便我们可以轻松地管理该表中的数据。我们在第一章 *遇见 Yii*中介绍了 Yii 的 ORM 层，**Active Record**（AR）。现在我们将在这个应用程序的上下文中看到一个具体的例子。

## 配置 Gii

回到第二章 *入门*，当我们构建我们简单的“Hello, World!” Yii 应用程序时，我们介绍了代码生成工具**Gii**。如果您还记得，在我们开始使用 Gii 之前，我们必须为其配置我们的应用程序。我们需要在我们的新 TrackStar 应用程序中再次这样做。作为提醒，要配置 Gii 的使用，打开`protected/config/main.php`，并定义 Gii 模块如下：

```php
return array(
  …
  …
  'modules'=>array(
'gii'=>array(
        'class'=>'system.gii.GiiModule',
        'password'=>false,
        // If removed, Gii defaults to localhost only. Edit carefully to taste.
        'ipFilters'=>array('127.0.0.1','::1'),
    ),
    …
  ),
```

这将 Gii 配置为一个应用程序模块。我们将在本书的后面详细介绍 Yii *模块*。此时重要的是确保将其添加到配置文件中，并提供您的密码（或者在开发环境中将密码设置为`false`，以避免被提示登录屏幕）。现在，通过转到`http://localhost/trackstar/index.php?r=gii`来导航到该工具。

## 使用 Gii 创建我们的 Project AR 类

Gii 的主菜单页面如下所示：

![使用 Gii 创建我们的 Project AR 类](img/8727_04_02.jpg)

由于我们想要为我们的`tbl_project`表创建一个新的模型类，**模型生成器**选项似乎是正确的选择。点击该链接会带我们到以下页面：

![使用 Gii 创建我们的 Project AR 类](img/8727_04_03.jpg)

**表前缀**字段主要用于帮助 Gii 确定我们正在生成的 AR 类的命名方式。如果您使用前缀，可以在此处添加。这样，它在命名新类时就不会使用该前缀。在我们的情况下，我们使用`tbl_`前缀，所以我们应该在这里指定。指定此值将意味着我们新生成的 AR 类将被命名为`Project`，而不是`Tbl_project`。

接下来的两个字段要求我们的表名和我们想要生成的类文件的名称。在**表名**字段中输入我们的表名`tbl_project`，并观察**模型类**名称自动填充。**模型类**名称的约定是表的名称，减去前缀，并以大写字母开头。因此，它将假定我们的模型类名称为 Project，但您当然可以自定义。

接下来的几个字段允许进一步定制。**基类**字段用于指定我们的模型类将继承的类。这将需要是`CActiveRecord`或其子类。**模型路径**字段让我们指定在应用程序目录结构中输出新文件的位置。默认值是`protected/models/`（别名`application.models`）。**构建关系**复选框允许您决定是否让 Gii 通过使用在 MySQL 数据库表之间定义的关系来自动定义 AR 对象之间的关系。它默认为选中状态。最后一个字段允许我们指定基于哪个模板进行代码生成。我们可以自定义默认模板以满足可能适用于所有这类类文件的任何特定需求。目前，这些字段的默认值完全满足我们的需求。

点击**预览**按钮继续。这将导致以下表格显示在页面底部：

![使用 Gii 创建我们的项目 AR 类](img/8727_04_04.jpg)

此链接允许您预览将要生成的代码。在点击**生成**之前，点击`models/Project.php`链接。以下截图显示了这个预览的样子：

![使用 Gii 创建我们的项目 AR 类](img/8727_04_05.jpg)

它提供了一个可滚动的弹出窗口，以便我们可以预览将要生成的文件。

好的，关闭这个弹出窗口，然后点击**生成**按钮。假设一切顺利，您应该看到页面底部显示类似以下截图的内容：

![使用 Gii 创建我们的项目 AR 类](img/8727_04_06.jpg)

### 提示

在尝试生成新模型类之前，请确保 Gii 尝试创建新文件的路径`protected/models/`（或者如果您更改了位置，则是**模型路径**表单字段中指定的任何目录路径）可被您的 Web 服务器进程写入，否则您将收到写入权限错误。

Gii 已为我们创建了一个新的 Yii 活动记录模型类，并按照我们的指示命名为`Project.php`。它还将其放在了默认的 Yii 模型类位置`protected/models/`中，这是我们指示的位置。这个类是我们的`tbl_project`数据库表的包装类。`tbl_project`表中的所有列都可以作为`Project` AR 类的属性访问。

# 为项目启用 CRUD 操作

现在我们有了一个新的 AR 模型类，但接下来呢？在 MVC 架构中，通常我们需要一个控制器和一个视图来配合我们的模型，以完成整个架构。在我们的情况下，我们需要能够在应用程序中管理我们的项目。我们需要能够创建新项目，检索现有项目的信息，更新现有项目的信息，并删除现有项目。我们需要添加一个控制器类，该类将处理我们的模型类上的 CRUD（创建、读取、更新、删除）操作，以及一个视图文件，以提供 GUI，允许用户在浏览器中执行这些操作。我们可以采取的一种方法是打开我们喜欢的代码编辑器，并创建一个新的控制器和视图类。但是，幸运的是，我们不必这样做。

## 为项目创建 CRUD 脚手架

再次，Gii 工具将帮助我们摆脱编写常见、繁琐且耗时的代码。CRUD 操作在为应用程序创建的数据库表上是如此常见，Yii 的开发人员决定为我们提供这个功能。如果您来自其他框架，您可能会知道这个术语**脚手架**。让我们看看如何在 Yii 中利用这一点。

返回到位于`http://localhost/trackstar/index.php?r=gii`的主 Gii 菜单，并选择**Crud Generator**链接。您将看到以下屏幕：

![为项目创建 CRUD 脚手架](img/8727_04_07.jpg)

在这里，我们看到两个输入表单字段。第一个要求我们指定针对哪个**模型类**生成所有 CRUD 操作。在我们的情况下，这是我们之前创建的`Project` AR 类。因此，我们将在此字段中输入**Project**。在这样做时，我们注意到**控制器 ID**字段自动填充了名称**project**。这是 Yii 的命名约定。当然，您可以更改为其他名称，但我们暂时将坚持使用默认值。我们还将使用默认的基础控制器类`Controller`，这是在我们最初创建应用程序时为我们创建的，以及默认的代码模板文件来生成类文件。

填写了所有这些字段后，点击**预览**按钮会在页面底部显示以下表格：

![为项目创建 CRUD 脚手架](img/8727_04_08.jpg)

我们可以看到将生成相当多的文件。列表顶部是一个新的`ProjectController`控制器类，将包含所有 CRUD 操作方法。列表的其余部分代表还将创建的许多单独的视图文件。每个操作都有一个单独的视图文件，还有一个将提供搜索项目记录功能的视图文件。当然，您可以通过更改表中相应**生成**列中的复选框来选择不生成其中的一些文件。但是，对于我们的目的，我们希望 Gii 为我们创建所有这些文件。

请点击**生成**按钮。您应该在页面底部看到以下成功消息：

![为项目创建 CRUD 脚手架](img/8727_04_09.jpg)

### 注意

您可能需要确保根应用程序目录下的`/protected/controllers`和`/protected/views`都可以被 Web 服务器进程写入。否则，您将收到权限错误，而不是这个成功的结果。

现在，我们可以点击**立即尝试**链接，测试我们的新功能。

这样做会带您到一个项目列表页面。这是显示系统中当前所有项目的页面。在我们的情况下，我们还没有创建任何项目，所以页面会显示**未找到结果**的消息。让我们通过创建一个新项目来改变这种情况。

## 创建新项目

在项目列表页面（`http://localhost/trackstar/index.php?r=project`）的右侧有一个小的导航区域。单击**创建项目**链接。您会发现这实际上将我们带到登录页面，而不是一个创建新项目的表单。原因是 Gii 生成的代码应用了一个规则，规定只有经过适当身份验证的用户（即已登录的用户）才能创建新项目。任何尝试访问创建新项目功能的匿名用户都将被重定向到登录页面。我们稍后会详细介绍身份验证和授权。现在，继续使用用户名`demo`和密码`demo`登录。

成功登录后，应将您重定向到以下 URL：

`http://localhost/trackstar/index.php?r=project/create`

此页面显示了一个用于添加新项目的输入表单，如下面的屏幕截图所示：

![创建新项目](img/8727_04_10.jpg)

让我们快速填写这个表单来创建一个新项目。表单指示有两个必填字段，**名称**和**描述**。Gii 代码生成器足够聪明，知道我们在数据库表中定义了`tbl_project.name`和`tbl_project.description`列为`NOT NULL`，这应该在创建新项目时转换为必填表单字段。很酷，对吧？

因此，我们至少需要填写这两个字段。给它起名字，`测试项目`，并将描述设置为`测试项目描述`。单击**创建**按钮将把表单数据发送回服务器，并尝试添加一个新的项目记录。如果有任何验证错误，将显示一个简单的错误消息，突出显示每个错误的字段。成功保存将重定向到新创建项目的特定列表。我们的成功了，我们被重定向到页面`http://localhost/trackstar/index.php?r=project/view&id=1`，如下面的屏幕截图所示：

![创建新项目](img/8727_04_11.jpg)

正如我们之前简要提到的，我们注意到我们的新项目创建表单中，名称和描述字段都被标记为必填项。这是因为我们在数据库表中定义了名称和描述列不允许为空值。让我们看看 Yii 中这些必填字段是如何工作的。

### 表单字段验证

在 Yii 中的表单中使用 AR 模型类时，围绕表单字段设置验证规则非常简单。这是通过在 AR 模型类中的`rules()`方法中定义的数组中指定值来完成的。

如果您查看`Project`模型类中的代码（`/protected/models/Project.php`），您会发现`rules()`公共函数已经为我们定义好了，并且其中已经有一些规则了：

```php
/**
   * @return array validation rules for model attributes.
   */
  public function rules()
  {
    // NOTE: you should only define rules for those attributes that
    // will receive user inputs.
    return array(
      array('name, description', 'required'),
      array('create_user_id, update_user_id', 'numerical', 'integerOnly'=>true),
      array('name', 'length', 'max'=>255),
      array('create_time, update_time', 'safe'),
      // The following rule is used by search().
      // Please remove those attributes that should not be searched.
      array('id, name, description, create_time, create_user_id, update_time, update_user_id', 'safe', 'on'=>'search'),
      );
  }	
```

`rules()`方法返回一个规则数组。每个规则的一般格式如下：

```php
Array('Attribute List', 'Validator', 'on'=>'Scenario List', …additional options);
```

`Attribute List`是一个逗号分隔的类属性名称字符串，根据`Validator`进行验证。`Validator`指定应强制执行什么样的规则。`on`参数指定应用规则的情景列表。例如，如果我们指定验证应在`insert`情景上下文中应用，这将表示规则应仅在插入新记录时应用。

如果没有定义特定的情景，验证规则将在模型数据进行验证时的所有情景中应用。

### 注意

从 Yii 的 1.1.11 版本开始，您还可以指定一个`except`参数，它允许您排除某些情景的验证。语法与`on`参数相同。

最后，您还可以指定额外的选项作为`name=>value`对，用于初始化验证器的属性。这些额外的选项将根据指定的验证器的属性而有所不同。

验证器可以是模型类中的一个方法，也可以是一个单独的验证器类。如果定义为模型类方法，它必须具有以下签名：

```php
/** 
* @param string the name of the attribute to be validated 
* @param array options specified in the validation rule 
*/ 
public function validatorName($attribute,$params) 
{
...
}
```

如果使用一个单独的类来定义验证器，那个类必须继承自`CValidator`。

实际上有三种指定验证器的方法：

+   在模型类本身中指定一个方法名

+   指定一个验证器类型的单独类（即一个继承自`CValidator`的类）

+   在 Yii 框架中指定现有验证器类的预定义别名

Yii 为您提供了许多预定义的验证器类，并提供了别名来定义规则时引用这些类。截至 Yii 版本 1.1.12，预定义的验证器类别名的完整列表如下：

+   **boolean**：`CBooleanValidator`的别名，验证包含`true`或`false`的属性

+   **captcha**：`CCaptchaValidator`的别名，验证属性值是否与 CAPTCHA 中显示的验证码相同

+   **compare**：`CCompareValidator`的别名，比较两个属性并验证它们是否相等

+   **email**：`CEmailValidator`的别名，验证属性值是否为有效的电子邮件地址

+   **date**：`CDateValidator`的别名，验证属性值是否为有效的日期、时间或日期时间值

+   **default**：`CDefaultValueValidator`的别名，为指定的属性分配默认值

+   **exist**：`CExistValidator`的别名，验证属性值是否与数据库中指定表列中的值相匹配

+   **file**：`CFileValidator`的别名，验证包含已上传文件名称的属性值

+   **filter**：`CFilterValidator`的别名，使用指定的过滤器转换属性值

+   **in**：`CRangeValidator`的别名，验证数据是否在预定范围内的值，或者存在于指定的值列表中

+   **length**：`CStringValidator`的别名，验证属性值的长度是否在指定范围内

+   **match**：`CRegularExpressionValidator`的别名，使用正则表达式验证属性值

+   **numerical**：`CNumberValidator`的别名，验证属性值是否为有效数字

+   **required**：`CRequiredValidator`的别名，验证属性值是否为空

+   **type**：`CTypeValidator`的别名，验证属性值是否为特定数据类型

+   **unique**：`CUniqueValidator`的别名，验证属性值是否唯一，并与数据库表列进行比较

+   **url**：`CUrlValidator`的别名，验证属性值是否为有效的 URL

我们看到在我们的`rules()`函数中，有一个规则定义了名称和描述属性，并使用了 Yii 别名`required`来指定验证器：

```php
array('name, description', 'required'),
```

这个验证规则的声明负责在新项目表单的**名称**和**描述**字段旁边显示小红色星号。这表示这个字段现在是必填的。如果我们回到新项目创建表单（`http://localhost/trackstar/index.php?r=project/create`）并尝试提交表单而没有指定**名称**或**描述**，我们将得到一个精美格式的错误消息，告诉我们不能提交带有这些字段空值的表单，如下面的截图所示：

![表单字段验证](img/8727_04_12.jpg)

### 注意

正如我们之前提到的，Gii 代码生成工具将根据底层表中列的定义自动向 AR 类添加验证规则。我们看到了**Name**和**Description**列定义为`NOT NULL`约束，并且有相关的必填验证器定义。另一个例子是，具有长度限制的列，比如我们的名字列被定义为`varchar(255)`，将自动应用字符限制规则。通过再次查看我们在`Project` AR 类中的`rules()`方法，我们注意到 Gii 根据其列定义为我们自动创建了规则`array('name', 'length', 'max'=>255)`。有关验证器的更多信息，请参见[`www.yiiframework.com/doc/guide/1.1/en/form.model#declaring-validation-rules`](http://www.yiiframework.com/doc/guide/1.1/en/form.model#declaring-validation-rules)。

## 阅读项目

当我们成功保存一个新项目后被带到项目详细信息页面时，我们实际上已经看到了这个过程`http://localhost/trackstar/index.php?r=project/view&id=1`。该页面展示了 CRUD 中的*R*。然而，要查看整个列表，我们可以点击右侧列中的**List Project**链接。这将带我们回到起点，只是现在我们在项目列表中有了我们新创建的项目。因此，我们有能力检索应用程序中所有项目的列表，以及查看每个项目的详细信息。

## 更新和删除项目

通过点击列表中任何项目的小项目**ID**链接，可以导航回项目详细信息页面。让我们为我们新创建的项目，即我们的情况下的**ID: 1**，做这个操作。点击此链接将带我们到该项目的项目详细信息页面。该页面在其右侧列中显示了一些操作功能，如下截图所示：

![更新和删除项目](img/8727_04_13.jpg)

我们可以看到**Update Project**和**Delete Project**链接，分别为我们提供了 CRUD 操作中的*U*和*D*。我们将留给您来验证这些链接是否按预期工作。

### 注意

删除功能仅限于管理员用户；也就是说，您必须使用`admin`/`admin`的用户名/密码组合登录。因此，如果您正在验证删除功能并收到 403 错误，请确保您以管理员身份登录。这将在后面更详细地讨论，并且我们将在后面的章节中详细介绍身份验证和授权。

## 在管理模式下管理项目

我们在上一个截图中未涵盖的最后一个链接是**Manage Project**链接。请点击此链接。这很可能会导致授权错误，如下截图所示：

![在管理模式下管理项目](img/8727_04_14.jpg)

出现此错误的原因是该功能调用了 Yii 中的简单访问控制功能，并且只限制了*admin*用户的访问。如果您回忆起，当我们登录应用程序以创建新项目时，我们使用`demo`/`demo`作为我们的用户名/密码组合。这个`demo`用户没有权限访问此管理员页面。由 Gii 生成的代码限制了对此功能的访问。

在这个上下文中，**管理员**简单地指的是使用`admin`/`admin`用户名/密码组合登录的人。请点击主导航栏中的**注销（演示）**来退出应用程序。然后再次登录，但这次使用管理员凭据。成功以`admin`身份登录后，您会注意到顶部导航栏的注销链接变成了**注销（管理员）**。然后返回到特定的项目列表页面，例如`http://localhost/trackstar/index.php?r=project/view&id=1`，再次尝试**管理项目**链接。您现在应该看到以下截图中显示的内容：

![在管理员模式下管理项目](img/8727_04_15.jpg)

我们现在看到的是我们项目列表页面的高度互动版本。它显示了所有项目在一个互动数据表中。每一行都有内联链接，可以查看、更新和删除每个项目。点击任何列标题链接都会按照该列数值对项目列表进行排序。第二行的小输入框允许您通过关键词在各个列数值中搜索这个项目列表。高级搜索链接会显示一个完整的搜索表单，提供了指定多个搜索条件来提交一个搜索的能力。以下截图显示了这个高级搜索表单：

![在管理员模式下管理项目](img/8727_04_16.jpg)

我们基本上实现了这个迭代中设定的所有功能，而且几乎没有编写任何代码。事实上，借助 Gii 的帮助，我们不仅创建了所有的 CRUD 功能，还实现了我们没有预期到达的基本项目搜索功能。虽然非常基础，但我们已经拥有了一个完全功能的应用程序，具有特定于项目任务跟踪应用程序的功能，并且几乎没有付出太多的努力。

当然，我们的 TrackStar 应用程序还有很多工作要完成。所有这些脚手架代码并不打算完全取代应用程序开发。它为我们提供了一个很好的起点和基础，可以继续构建我们的应用程序。当我们通过项目功能应该如何工作的所有细节和微妙之处时，我们可以依靠这个自动生成的代码以快速的速度推动事情向前发展。

# 摘要

尽管在本章中我们没有做太多编码，但我们取得了很大的成就。我们创建了一个新的数据库表，这使我们能够看到 Yii Active Record（AR）的实际运行情况。我们使用 Gii 工具首先创建了一个 AR 模型类来包装我们的`tbl_project`数据库表。然后我们演示了如何使用 Gii 代码生成工具在 Web 应用程序中生成实际的 CRUD 功能。这个神奇的工具快速地创建了我们需要的功能，甚至进一步提供了一个管理仪表板，让我们可以根据不同的条件搜索和排序我们的项目。我们还演示了如何实现模型数据验证以及这如何转化为 Yii 中表单字段验证。

在下一章中，我们将在已学到的基础上继续深入研究 Yii 中的 Active Record，同时在我们的数据模型中引入相关实体。
