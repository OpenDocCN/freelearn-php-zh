# 第五章。管理问题

在上一章中，我们提供了围绕项目实体的基本功能。项目是 TrackStar 应用程序的基础。然而，单独的项目并不是非常有用。项目是我们希望这个应用程序管理的问题的基本容器。由于管理项目问题是这个应用程序的主要目的，我们希望开始添加一些基本的问题管理功能。

# 功能规划

我们已经有了创建和列出项目的能力，但没有办法管理与项目相关的问题。在本章结束时，我们希望应用程序能够在项目问题或任务上公开所有 CRUD 操作。 （我们倾向于交替使用*问题*和*任务*这两个术语，但在我们的数据模型中，任务实际上只是问题的一种类型。）我们还希望限制对问题的所有 CRUD 操作都在特定项目的上下文中进行。也就是说，*问题*属于*项目*。用户必须在能够对项目的问题执行任何 CRUD 操作之前，选择了一个现有的项目来工作。

为了实现前面提到的目标，我们需要：

+   设计数据库模式并构建支持项目问题的对象

+   创建 Yii 模型类，使应用程序能够轻松地与我们创建的数据库表进行交互

+   创建控制器类，其中将包含允许我们进行以下操作的功能：

+   创建新问题

+   从数据库中检索项目中现有问题的列表

+   更新/编辑现有问题

+   删除现有问题

+   为这些（上述）操作创建视图以渲染用户界面

这个列表足以让我们开始。让我们开始做必要的数据库更改。

# 设计模式

回到第三章, *TrackStar 应用程序*，我们提出了一些关于*问题*实体的初始想法。我们建议它有一个*名称*，一个*类型*，一个*所有者*，一个*请求者*，一个*状态*和一个*描述*。我们还提到当我们创建`tbl_project`表时，我们将向每个创建的表添加基本的审计历史信息，以跟踪更新表的日期、时间和用户。然而，类型、所有者、请求者和状态本身也是它们自己的实体。为了保持我们的模型灵活和可扩展，我们将分别对其中一些进行建模。*所有者*和*请求者*都是系统的用户，因此将被放在一个名为`tbl_user`的单独表中。我们已经在`tbl_project`表中介绍了*用户*的概念，因为我们添加了`create_user_id`和`update_user_id`列来跟踪最初创建项目的用户的标识符，以及负责最后更新项目详细信息的用户。尽管我们尚未正式介绍该表，但这些字段旨在成为`user`表的外键。`tbl_issue`表中的`owner_id`和`requestor_id`列也将是关联回这个`tbl_user`表的外键。

我们可以以相同的方式对类型和状态属性进行建模。然而，直到我们的需求要求模型中的这种额外复杂性，我们可以保持简单。`tbl_issue`表上的`type`和`status`列将保持整数值，可以映射到命名类型和状态。然而，我们将这些建模为我们为问题实体创建的 AR 模型类中的基本类常量（`const`）值，而不是通过使用单独的表来使我们的模型复杂化。如果所有这些都有点模糊，不要担心；在接下来的章节中会更清晰。

## 定义一些关系

由于我们引入了`tbl_user`表，我们需要回去定义用户和项目之间的关系。在第三章中，*TrackStar 应用程序*，我们指定用户（我们称之为项目成员）将与零个或多个项目关联。我们还提到项目也可以有许多（一个或多个）用户。由于项目可以有许多用户，并且这些用户可以与许多项目关联，我们将其称为项目和用户之间的**多对多**关系。在关系数据库中建模多对多关系的最简单方法是使用关联表（也称为分配表）。因此，我们还需要将这个表添加到我们的模型中。

下图概述了用户、项目和问题之间的基本实体关系。项目可以有零到多个用户。用户需要与至少一个项目关联，但可以与多个项目关联。问题属于一个且仅属于一个项目，而项目可以有零到多个问题。最后，一个问题被分配给（或由）一个单一用户。

![定义一些关系](img/8727_05_01.jpg)

# 构建对象及其关系

我们需要创建三个新表，即`tbl_issue`、`tbl_user`和我们的关联表`tbl_project_user_assignment`。您可能还记得我们在第四章介绍了 Yii 数据库迁移。由于我们现在准备对数据库结构进行更改，我们将使用 Yii 迁移来更好地管理这些更改的应用。

由于我们要同时向数据库中添加这些内容，我们将在一个迁移中完成。从命令行，切换到`protected/`目录，并输入以下命令：

```php
**$ ./yiic migrate create create_issue_user_and_assignment_tables**

```

这将导致一个新的迁移文件被添加到`protected/migrations/`目录中。

打开这个新创建的文件，并按照以下方式实现 safeUp()和 safeDown()方法：

```php
  // Use safeUp/safeDown to do migration with transaction
  public function safeUp()
  {
    //create the issue table
    $this->createTable('tbl_issue', array(
      'id' => 'pk',
        'name' => 'string NOT NULL',
        'description' => 'text',
        'project_id' => 'int(11) DEFAULT NULL',
      'type_id' => 'int(11) DEFAULT NULL',
      'status_id' => 'int(11) DEFAULT NULL',
      'owner_id' => 'int(11) DEFAULT NULL',
      'requester_id' => 'int(11) DEFAULT NULL',
      'create_time' => 'datetime DEFAULT NULL',
      'create_user_id' => 'int(11) DEFAULT NULL',
      'update_time' => 'datetime DEFAULT NULL',
      'update_user_id' => 'int(11) DEFAULT NULL',
       ), 'ENGINE=InnoDB');

    //create the user table
    $this->createTable('tbl_user', array(
      'id' => 'pk',
      'username' => 'string NOT NULL',
        'email' => 'string NOT NULL',
        'password' => 'string NOT NULL',
      'last_login_time' => 'datetime DEFAULT NULL',
      'create_time' => 'datetime DEFAULT NULL',
      'create_user_id' => 'int(11) DEFAULT NULL',
      'update_time' => 'datetime DEFAULT NULL',
      'update_user_id' => 'int(11) DEFAULT NULL',
       ), 'ENGINE=InnoDB');

    //create the assignment table that allows for many-to-many 
//relationship between projects and users
    $this->createTable('tbl_project_user_assignment', array(
      'project_id' => 'int(11) NOT NULL',
      'user_id' => 'int(11) NOT NULL',
      'PRIMARY KEY (`project_id`,`user_id`)',
     ), 'ENGINE=InnoDB');

    //foreign key relationships

    //the tbl_issue.project_id is a reference to tbl_project.id 
    $this->addForeignKey("fk_issue_project", "tbl_issue", "project_id", "tbl_project", "id", "CASCADE", "RESTRICT");

    //the tbl_issue.owner_id is a reference to tbl_user.id 
    $this->addForeignKey("fk_issue_owner", "tbl_issue", "owner_id", "tbl_user", "id", "CASCADE", "RESTRICT");

    //the tbl_issue.requester_id is a reference to tbl_user.id 
    $this->addForeignKey("fk_issue_requester", "tbl_issue", "requester_id", "tbl_user", "id", "CASCADE", "RESTRICT");

    //the tbl_project_user_assignment.project_id is a reference to tbl_project.id 
    $this->addForeignKey("fk_project_user", "tbl_project_user_assignment", "project_id", "tbl_project", "id", "CASCADE", "RESTRICT");

    //the tbl_project_user_assignment.user_id is a reference to tbl_user.id 
    $this->addForeignKey("fk_user_project", "tbl_project_user_assignment", "user_id", "tbl_user", "id", "CASCADE", "RESTRICT");

  }

  public function safeDown()
  {
    $this->truncateTable('tbl_project_user_assignment');
    $this->truncateTable('tbl_issue');
    $this->truncateTable('tbl_user');
    $this->dropTable('tbl_project_user_assignment');
    $this->dropTable('tbl_issue');
    $this->dropTable('tbl_user');
  }
```

在这里，我们实现了`safeUp()`和`safeDown()`方法，而不是标准的`up()`和`down()`方法。这样做可以在数据库事务中运行这些语句，以便它们作为单个单元被提交或回滚。

### 注意

实际上，由于我们正在使用 MySQL，这些`create table`和`drop table`语句不会在单个事务中运行。某些 MySQL 语句会导致隐式提交，因此在这种情况下使用`safeUp()`和`safeDown()`方法并没有太多用处。我们将保留这一点，以帮助用户了解为什么 Yii 迁移提供`safeUp()`和`safeDown()`方法。有关更多详细信息，请参见[`dev.mysql.com/doc/refman/5.5/en/implicit-commit.html`](http://dev.mysql.com/doc/refman/5.5/en/implicit-commit.html)。

现在我们可以从命令行运行迁移：

![构建对象及其关系](img/8727_05_17.jpg)

这个迁移已经创建了我们需要的数据库对象。现在我们可以把注意力转向创建我们的活动记录模型类。

# 创建活动记录模型类

现在我们已经创建了这些表，我们需要创建 Yii 模型 AR 类，以便我们可以在应用程序中轻松地与这些表交互。在上一章创建`Project`模型类时，我们使用了 Gii 代码生成工具。我们会在这里提醒您这些步骤，但不会给您所有的截图。请参考第四章，*项目 CRUD*，以获取使用 Gii 工具创建活动记录类的更详细步骤。

## 创建 Issue 模型类

通过`http://localhost/trackstar/index.php?r=gii`导航到 Gii 工具，然后选择**Model Generator**链接。将表前缀保留为**tbl_**。在**Table Name**字段中填写`tbl_issue`，这将自动填充**Model Class**字段为**Issue**。还要确保**Build Relations**复选框被选中。这将确保我们的关系在新的模型类中自动创建。

填写表单后，点击**Preview**按钮，获取一个弹出窗口的链接，显示即将生成的所有代码。然后点击**Generate**按钮，实际在`/protected/models/`目录中创建新的`Issue.php`模型类文件。

## 创建用户模型类

这在这一点上可能已经变得老生常谈了，所以我们将把`User` AR 类的创建留给您作为一个练习。在下一章中，当我们深入研究用户认证和授权时，这个特定的类将变得更加重要。

您可能会问，“`tbl_project_user_assignment`表的 AR 类呢？”。虽然可以为这个表创建一个 AR 类，但这并不是必要的。AR 模型为我们的应用程序提供了一个**对象关系映射**（**ORM**）层，帮助我们更轻松地处理领域对象。然而，*ProjectUserAssignment*不是我们应用程序的领域对象。它只是一个在关系数据库中的构造，帮助我们建模和管理项目和用户之间的多对多关系。为处理这个表的管理而维护一个单独的 AR 类是我们可以暂时避免的额外复杂性。我们可以直接使用 Yii 的 DAO 来管理这个表的插入、更新和删除。

# 创建问题的 CRUD 操作

现在我们已经有了问题的 AR 类，我们可以开始构建必要的功能来管理我们的项目问题。我们将再次依靠 Gii 代码生成工具来帮助我们创建这些功能的基础。我们在上一章节详细介绍了项目的这一点。我将再次提醒您 Issues 的基本步骤：

1.  导航到 Gii 生成器菜单`http://localhost/trackstar/index.php?r=gii`，然后选择**Crud Generator**链接。

1.  使用**Issue**作为**Model Class**字段的值填写表单。这将自动填充**Controller ID**为**Issue**。**Base Controller Class**和**Code Template**字段可以保留它们预定义的默认值。

1.  点击**Preview**按钮，获取 Gii 工具建议创建的所有文件列表。以下截屏显示了这些文件的列表：![Creating the issue CRUD operations](img/8727_05_02.jpg)

1.  您可以点击每个单独的链接预览要生成的代码。一旦满意，点击**Generate**按钮来创建所有这些文件。您应该收到以下成功消息：![Creating the issue CRUD operations](img/8727_05_03.jpg)

# 使用问题 CRUD 操作

让我们试一试。要么点击前面截屏中显示的**try it now**链接，要么直接导航到`http://localhost/trackstar/index.php?r=issue`。您应该看到类似于以下截屏的内容：

使用问题 CRUD 操作

## 创建一个新问题

由于我们还没有添加任何新问题，所以没有要列出的问题。让我们改变这种情况，创建一个新问题。点击**Create Issue**链接。（如果这将您带到登录页面，请使用`demo/demo`或`admin/admin`登录。成功登录后，您将被正确重定向。）现在您应该看到一个类似于以下截屏的新问题输入表单：

创建一个新问题

当查看这个输入表单时，我们可以看到它在数据库表中的每一列都有一个输入字段，就像在数据库表中定义的那样。然而，正如我们从设计模式和建立表格时所知道的那样，其中一些字段不是直接的输入字段，而是代表与其他实体的关系。例如，与其在这个表单上有一个**类型**自由文本输入字段，我们应该使用一个下拉输入表单字段，其中填充了允许的问题类型的选择。类似的论点也适用于**状态**字段。**所有者**和**请求者**字段也应该是下拉菜单，显示被分配到处理问题所在项目的用户的名称选择。此外，由于所有问题管理都应该在特定项目的上下文中进行，**项目**字段根本不应该是这个表单的一部分。最后，**创建时间**、**创建用户**、**更新时间**和**更新用户**字段都是应该在表单提交后计算和确定的值，不应该供用户直接操作。

看起来我们已经确定了一些我们想要在这个初始输入表单上做出的更正。正如我们在上一章中提到的，Gii 工具生成的自动生成的 CRUD“脚手架”代码只是一个起点。很少有情况下它本身就足以满足应用程序的所有特定功能需求。

### 添加下拉字段

我们将从为问题类型添加一个下拉菜单开始。问题只有三种类型，即*错误*、*功能*和*任务*。当创建一个新问题时，我们希望看到的是一个下拉式输入类型表单字段，其中包含这三个选择。我们将通过`Issue`模型类本身提供其可用类型的列表来实现这一点。由于我们没有创建一个单独的数据库表来保存我们的问题类型，我们将这些直接添加为`Issue`活动记录模型类的类常量。

在`Issue`模型类的顶部添加以下三个常量定义：

```php
const TYPE_BUG=0;
const TYPE_FEATURE=1;
const TYPE_TASK=2;
```

现在在这个类中添加一个新的方法`Issue::getTypeOptions()`，它将根据这些定义的常量返回一个数组：

```php
/**
  * Retrieves a list of issue types
  * @return array an array of available issue types.
  */
public function getTypeOptions()
{
  return array(
    self::TYPE_BUG=>'Bug',
    self::TYPE_FEATURE=>'Feature',
    self::TYPE_TASK=>'Task',
  );
}
```

现在我们有了一种方法来检索可用的问题类型列表，但我们仍然没有一个下拉字段在输入表单中显示这些值，我们可以从中选择。让我们现在添加它。

### 添加问题类型下拉

打开包含新问题创建表单的文件`protected/views/issue/_form.php`，找到与表单上的**类型**字段对应的行：

```php
<div class="row">
  <?php echo $form->labelEx($model,'type_id'); ?>
  <?php echo $form->textField($model,'type_id'); ?>
  <?php echo $form->error($model,'type_id'); ?>
</div> 
```

这些行需要一点澄清。为了理解这一点，我们需要参考`_form.php`文件顶部的一些代码，如下所示：

```php
<?php $form=$this->beginWidget('CActiveForm', array(
  'id'=>'issue-form',
  'enableAjaxValidation'=>false,
)); ?>
```

这是使用 Yii 中的`CActiveForm`小部件定义`$form`变量。**小部件**将在以后更详细地介绍。现在，我们可以通过更好地理解`CActiveForm`来理解这段代码。`CActiveForm`可以被认为是一个帮助类，它提供了一组方法来帮助我们创建与数据模型类相关联的表单。在这种情况下，它被用来基于我们的`Issue`模型类创建一个输入表单。

为了充分理解视图文件中的变量，让我们也回顾一下渲染视图文件的控制器代码。正如之前讨论过的，从控制器传递数据到视图的一种方式是通过显式声明一个数组，其中的键将是视图文件中可用变量的名称。由于这是一个新问题的创建操作，渲染表单的控制器方法是`IssueController::actionCreate()`。该方法如下所示：

```php
/**
   * Creates a new model.
   * If creation is successful, the browser will be redirected to the 'view' 
 * page.
   */
  public function actionCreate()
  {
    $model=new Issue;

    // Uncomment the following line if AJAX validation is needed
    // $this->performAjaxValidation($model);

    if(isset($_POST['Issue']))
    {
      $model->attributes=$_POST['Issue'];
      if($model->save())
        $this->redirect(array('view','id'=>$model->id));
    }

    $this->render('create',array(
      'model'=>$model,
    ));
  }
```

在这里，我们看到当视图被渲染时，它会传递一个`Issue`模型类的实例，这个实例将在视图中作为一个名为`$model`的变量可用。

现在让我们回到负责在表单上渲染**Type**字段的代码。第一行是：

```php
$form->labelEx($model,'type_id');
```

这一行使用`CActiveForm::labelEx()`方法为问题模型属性`type_id`渲染 HTML 标签。它接受模型类的实例和我们想要生成标签的相应模型属性。模型类`Issue:: attributeLabels()`方法将被用于确定标签。如果我们查看下面列出的方法，我们会看到属性`type_id`被映射为标签`'Type'`，这正是我们在表单字段中看到的标签。

```php
public function attributeLabels()
{
    return array(
      'id' => 'ID',
      'name' => 'Name',
      'description' => 'Description',
      'project_id' => 'Project',
 **'type_id' => 'Type',**
      'status_id' => 'Status',
      'owner_id' => 'Owner',
      'requester_id' => 'Requester',
      'create_time' => 'Create Time',
      'create_user_id' => 'Create User',
      'update_time' => 'Update Time',
      'update_user_id' => 'Update User',
    );
}
```

使用`labelEx()`方法也是我们的必填字段旁边出现小红星号的原因。当属性是必填时，`labelEx()`方法将添加一个额外的`CSS`类名（`CHtml::requiredCss`，默认为`'required'`）和星号（使用`CHtml::afterRequiredLabel`，默认为`' <span class="required">*</span>'`）。

接下来的一行，`<?php echo $form->textField($model,'type_id'); ?>`，使用`CActiveForm::textField()`方法为我们的`Issue`模型属性`type_id`渲染文本输入字段。在模型类`Issue::rules()`方法中定义的任何验证规则都将被应用为此输入表单的表单验证规则。

最后一行`<?php echo $form->error($model,'type_id'); ?>`使用`CActiveForm::error()`方法在提交时渲染与`type_id`属性相关的任何验证错误。

您可以尝试使用类型字段进行验证。在我们的 MySQL 模式定义中，`type_id`列被定义为整数类型，因此，Gii 在`Issue::rules()`方法中生成了一个验证规则来强制执行这一点。

```php
  public function rules()
  {
    // NOTE: you should only define rules for those attributes that
    // will receive user inputs.
    return array(
      array('name', 'required'),
 **array('project_id, type_id, status_id, owner_id, requester_id, create_user_id, update_user_id', 'numerical', 'integerOnly'=>true),**

```

因此，如果我们尝试在**Type**表单字段中提交字符串值，我们将在字段下方立即收到内联错误，如下面的屏幕截图所示：

![添加问题类型下拉菜单](img/8727_05_06.jpg)

现在我们更好地理解了我们所拥有的东西，我们更有能力对其进行更改。我们需要做的是将这个字段从自由格式的文本输入字段更改为下拉式输入类型。也许不足为奇的是，`CActiveForm`类有一个`dropDownList()`方法，可以为模型属性生成一个下拉列表。让我们用以下内容替换调用`$form->textField`的行（在文件`/protected/views/issue/_form.php`中）：

```php
<?php echo $form->dropDownList($model,'type_id', $model->getTypeOptions()); ?>
```

这仍然将早期的模型作为第一个参数，将模型属性作为第二个参数。第三个参数指定下拉选择列表。这应该是一个`value=>display`对的数组。我们已经在`Issue.php`模型类中创建了我们的`getTypeOptions()`方法来返回这种格式的数组，所以我们可以直接使用它。

### 注意

应该注意的是，Yii 框架基类使用了 PHP `_get` "魔术"函数。这允许我们在子类中编写诸如`getTypeOptions()`之类的方法，并使用`->typeOptions`语法将这些方法作为类属性引用。因此，当请求问题类型选项数组`$model->typeOptions`时，我们也可以使用等效的语法。

保存您的工作，再次查看我们的问题输入表单。您应该看到一个漂亮的问题类型选择下拉菜单，取代了自由格式文本字段，如下面的屏幕截图所示：

![添加问题类型下拉菜单](img/8727_05_07.jpg)

### 添加状态下拉菜单：自己动手

我们将采用相同的方法处理问题状态。正如在第三章中提到的*TrackStar 应用程序*，当我们介绍应用程序时，问题可以处于以下三种状态之一：

+   尚未开始

+   开始

+   完成

我们将在`Issue`模型类中创建三个类常量来表示状态值。然后我们将创建一个新方法，`Issue::getStatusOptions()`，来返回一个可用的问题状态数组。最后，我们将修改`_form.php`文件，以渲染状态选项的下拉菜单，而不是状态的自由格式文本输入字段。

我们将把状态下拉菜单的实现留给你。你可以按照我们为类型所采取的方法来操作。在你做出这些改变后，表单应该看起来和下面的截图类似：

![添加状态下拉菜单：自己动手做](img/8727_05_08.jpg)

我们还应该注意，当我们把这些从自由格式文本输入字段改为下拉菜单字段时，最好也在我们的`rules()`方法中添加一个范围验证，以确保提交的值在下拉菜单允许的值范围内。在上一章中，我们看到了 Yii 框架提供的所有验证器列表。`CRangeValidator`属性，使用别名*in*，是定义这个验证规则的一个很好的选择。因此，我们可以定义这样一个规则：

```php
array('type_id', 'in', 'range'=>self::getAllowedTypeRange()),
```

然后我们添加一个方法来返回我们允许的数值类型值的数组：

```php
public static function getAllowedTypeRange()
{
  return array(
    self::TYPE_BUG,
    self::TYPE_FEATURE,
    self::TYPE_TASK,
  );
}
```

同样的方法也用于我们的`status_id`。我们也将把这个留给你来实现。

# 修复所有者和请求者字段

我们在问题创建表单中注意到的另一个问题是，所有者和请求者字段也是自由格式的文本输入字段。然而，我们知道这些在问题表中是整数值，它们保存了对`tbl_user`表的`id`列的外键标识符。因此，我们还需要为这些字段添加下拉菜单。我们不会采取和类型和状态属性相同的方法，因为问题的所有者和请求者需要从`tbl_user`表中获取。而且，由于系统中并非每个用户都与问题所在的项目相关联，这些问题不能用从整个`tbl_user`表中获取的数据填充下拉菜单。我们需要将列表限制为仅包括与该项目相关联的用户。

这也带来了另一件我们需要解决的事情。正如本章开头的*功能规划*部分所提到的，我们需要在特定项目的上下文中管理我们的问题。也就是说，在创建新问题之前，应该选择一个特定的项目。目前，我们的应用程序没有强制执行这个工作流程。

让我们逐一解决这些变化。首先，我们将修改应用程序，以强制在使用与该项目相关的任何功能来管理相关问题之前，必须确定一个有效的项目。一旦选择了一个项目，我们将确保我们的所有者和请求者下拉选择仅限于与该项目相关联的用户。

## 强制项目上下文

在允许访问管理问题之前，我们希望确保存在一个有效的项目上下文。为了做到这一点，我们将实现一个叫做过滤器的东西。在 Yii 中，**过滤器**是一段配置为在控制器动作执行之前或之后执行的代码。一个常见的例子是，如果我们想要确保用户在执行控制器动作方法之前已经登录，我们可以编写一个简单的访问过滤器来检查这个要求。另一个例子是，如果我们想在动作执行后执行一些额外的日志记录或其他审计逻辑，我们可以编写一个简单的审计过滤器来提供这种动作后处理。

在这种情况下，我们希望确保在创建新问题之前已经选择了一个有效的项目。因此，我们将在我们的`IssueController`类中添加一个项目过滤器来实现这一点。

### 定义过滤器

过滤器可以被定义为控制器类方法，也可以是一个单独的类。使用简单方法的方法时，方法名必须以 *filter* 开头，并具有特定的签名。例如，如果我们要创建一个名为 *someMethodName* 的过滤器方法，我们的完整过滤器方法将如下所示：

```php
public function filterSomeMethodName($filterChain)
{
...
}
```

另一种方法是编写一个单独的类来执行过滤逻辑。使用单独类的方法时，该类必须扩展 `CFilter`，然后根据逻辑应该在操作调用之前还是之后，重写至少一个 `preFilter()` 或 `postFilter()` 方法。

### 添加一个过滤器

因此，让我们向我们的 `IssueController` 类添加一个过滤器，以处理对有效项目的检查。我们将采用类方法的方法。

打开 `protected/controllers/IssueController.php` 并在类的底部添加以下方法：

```php
public function filterProjectContext($filterChain)
{   
     $filterChain->run(); 
} 
```

好的，我们现在已经定义了一个过滤器。但是它还没有做太多事情。它只是执行 `$filterChain->run()`，这会继续过滤过程并允许被该方法过滤的操作方法的执行。这带来了另一个问题。我们如何定义哪些操作方法应该使用这个过滤器？

### 指定被过滤的操作

我们的控制器类的 Yii 框架基类是 `CController`。它有一个需要被重写以指定需要应用过滤器的操作的 `filters()` 方法。实际上，这个方法已经在我们的 `IssueController.php` 类中被重写。当我们使用 Gii 工具自动生成这个类时，它已经为我们完成了。它已经添加了一个简单的 *accessControl* 过滤器，该过滤器在 `CController` 基类中定义，用于处理一些基本授权，以确保用户有足够的权限执行某些操作。如果您尚未登录并单击 **创建问题** 链接，您将被引导到登录页面进行身份验证，然后才能创建新问题。访问控制过滤器负责此操作。在下一章节中，当我们专注于用户身份验证和授权时，我们将更详细地介绍它。

目前，我们只需要将我们的新过滤器添加到这个配置数组中。要指定我们的新过滤器应该应用于创建操作，通过添加下面的代码来修改 `IssueController::filters()` 方法：

```php
/**
 * @return array action filters
 */
public function filters()
{
  return array(
    'accessControl', // perform access control for CRUD operations  
 **'projectContext + create', //check to ensure valid project context**
  );
}
```

`filters()` 方法应该返回一个过滤器配置的数组。之前的代码返回了一个配置，指定了应该将定义为类内方法的 `projectContext` 过滤器应用于 `actionCreate()` 方法。配置语法允许使用 "+" 和 "-" 符号来指定是否应该应用过滤器。例如，如果我们决定希望该过滤器应用于除 `actionUpdate()` 和 `actionView()` 之外的所有操作，我们可以指定：

```php
return array(
        'projectContext - update, view' ,
 );
```

您不应该同时指定加号和减号运算符。对于任何给定的过滤器配置，只需要一个。加号运算符表示“仅将过滤器应用于以下操作”。减号运算符表示“将过滤器应用于除以下操作之外的所有操作”。如果配置中既没有“+”也没有“-”，则该过滤器将应用于所有操作。

目前，我们将这个限制在只有创建操作。因此，如之前定义的 `+ create` 配置，我们的过滤器方法将在任何用户尝试创建新问题时被调用。

### 添加过滤逻辑

好的，现在我们已经定义了一个过滤器，并且已经配置它在尝试的 `actionCreate()` 方法调用时被调用。但是，它仍然没有执行必要的逻辑。由于我们希望在尝试操作之前确保项目上下文，我们需要在调用 `$filterChain->run()` 之前将逻辑放在过滤器方法中。

我们将在控制器类本身中添加一个项目属性。然后，我们将在我们的 URL 中使用一个查询字符串参数来指示项目标识符。我们的预操作过滤器将检查现有的项目属性是否为空；如果是，它将使用查询字符串参数来尝试根据主键标识符选择项目。如果成功，操作将执行；如果失败，将抛出异常。以下是执行所有这些操作所需的相关代码：

```php
class IssueController extends CController
{
     ....
     /**
   * @var private property containing the associated Project model instance.
   */
     private $_project = null; 

     /**
   * Protected method to load the associated Project model class
       * @param integer projectId the primary identifier of the associated Project
 * @return object the Project data model based on the primary key 
   */
     protected function loadProject($projectId)    {
     //if the project property is null, create it based on input id
     if($this->_project===null)
     {
      $this->_project=Project::model()->findByPk($projectId);
      if($this->_project===null)
                  {
          throw new CHttpException(404,'The requested project does not exist.'); 
         }
     }

     return $this->_project; 
  } 

  /**
   * In-class defined filter method, configured for use in the above filters() 
 * method. It is called before the actionCreate() action method is run in 
 * order to ensure a proper project context
   */
  public function filterProjectContext($filterChain)
  {   
//set the project identifier based on GET input request variables       if(isset($_GET['pid']))
      $this->loadProject($_GET['pid']);   
    else
      throw new CHttpException(403,'Must specify a project before performing this action.');

    //complete the running of other filters and execute the requested action
    $filterChain->run();

  }
  ...
}
```

有了这个设置，如果您现在尝试通过在问题列表页面上的**创建问题**链接上点击来创建一个新问题，您应该会看到一个“错误 403”错误消息，同时显示我们之前指定的错误文本。

这很好。它表明我们已经正确实现了防止在没有识别到项目时创建新问题的代码。要快速解决这个错误，只需简单地在用于创建新问题的 URL 中添加一个`pid`查询字符串参数。让我们这样做，这样我们就可以为过滤器提供一个有效的项目标识符，并继续到创建新问题的表单。

### 添加项目 ID

回到第四章*项目 CRUD*，在测试和实施项目的 CRUD 操作时，我们向应用程序添加了几个新项目。因此，您可能仍然在开发数据库中拥有一个有效的项目。如果没有，只需使用应用程序再次创建一个新项目。完成后，请注意所创建的*项目 ID*，因为我们需要将此 ID 添加到新问题的 URL 中。

我们需要修改的链接位于问题列表页面的视图文件`/protected/views/issue/index.php`中。在该文件的顶部，您会看到我们的菜单项中定义了创建新问题的链接的位置。这在以下突出显示的代码中指定：

```php
$this->menu=array(
 **array('label'=>'Create Issue', 'url'=>array('create')),**
  array('label'=>'Manage Issue', 'url'=>array('admin')),
);
```

要向这个链接添加一个查询字符串参数，我们只需在`url`参数的定义数组中追加一个*name=>value*对。我们为过滤器添加的代码期望查询字符串参数是`pid`（项目 ID）。另外，由于我们在这个例子中使用的是第一个（项目 ID = 1）项目，我们将修改**创建问题**链接如下：

```php
array('label'=>'Create Issue', 'url'=>array('create', 'pid'=>1)),

```

现在当您查看问题列表页面时，您会看到**创建问题**超链接打开了一个在末尾附加了查询字符串参数的 URL：

`http://localhost/trackstar/index.php?r=issue/create&pid=1`

这个查询字符串参数允许过滤器正确设置项目上下文。所以这一次当您点击链接时，不会再出现 403 错误页面，而是会显示创建新问题的表单。

### 注意

有关在 Yii 中使用过滤器的更多详细信息，请参阅[`www.yiiframework.com/doc/guide/1.1/en/basics.controller#filter`](http://www.yiiframework.com/doc/guide/1.1/en/basics.controller#filter)。

### 修改项目详细信息页面

将项目 ID 添加到“创建新问题”链接的 URL 是确保我们的过滤器按预期工作的一个很好的第一步。然而，我们现在已经将链接硬编码为始终将新问题与项目 ID = 1 关联起来。这当然不是我们想要的。我们想要做的是让创建新问题的菜单选项成为项目详细信息页面的一部分。这样，一旦您从项目列表页面选择了一个项目，特定的项目上下文将被知晓，我们可以动态地将项目 ID 附加到创建新问题的链接上。让我们做出这个改变。

打开项目详细信息视图文件`/protected/views/project/view.php`。在这个文件的顶部，您会注意到包含在`$this->menu`数组中的菜单项。我们需要在已定义的菜单链接列表的末尾添加另一个链接以创建新问题：

```php
$this->menu=array(
  array('label'=>'List Project', 'url'=>array('index')),
  array('label'=>'Create Project', 'url'=>array('create')),
  array('label'=>'Update Project', 'url'=>array('update', 'id'=>$model->id)),
  array('label'=>'Delete Project', 'url'=>'#', 'linkOptions'=>array('submit'=>array('delete','id'=>$model->id),'confirm'=>'Are you sure you want to delete this item?')),
  array('label'=>'Manage Project', 'url'=>array('admin')),
 **array('label'=>'Create Issue', 'url'=>array('issue/create', 'pid'=>$model->id)),**
);
```

我们所做的是将菜单选项移动到列出特定项目详细信息的页面上创建新问题。我们使用了类似于之前的链接，但这次我们必须指定完整的*controllerID/actionID*对（`issue/create`）。而不是将项目 ID 硬编码为 1，我们在视图文件中使用了`$model`变量，这是特定项目的 AR 类。通过这种方式，无论我们选择哪个项目，这个变量都将始终反映该项目的正确项目`id`属性。

有了这个设置，我们还可以删除另一个链接，我们在`protected/views/issue/index.php`视图文件中将项目 ID 硬编码为`1`。

现在我们在创建新问题时已经正确设置了项目上下文，我们可以将项目字段作为用户输入表单字段删除。打开新问题表单的视图文件`/protected/views/issue/_form.php`。删除与项目输入字段相关的以下行：

```php
<div class="row">
    <?php echo $form->labelEx($model,'project_id'); ?>
    <?php echo $form->textField($model,'project_id'); ?>
    <?php echo $form->error($model,'project_id'); ?>
</div>
```

然而，由于`project_id`属性不会随表单一起提交，我们需要根据我们刚刚实现的过滤器设置的值来设置`project_id`参数。由于我们已经知道关联的项目 ID，让我们明确地将`Issue::project_id`设置为我们先前实现的过滤器创建的项目实例的`id`属性的值。因此，根据以下突出显示的代码修改`IssueController::actionCreate()`方法：

```php
public function actionCreate()
{
  $model=new Issue;
 **$model->project_id = $this->_project->id;**

```

现在当我们提交表单时，问题活动记录实例的`project_id`属性将被正确设置。即使我们还没有设置我们的所有者和请求者下拉框，我们也可以提交表单，新问题将被创建并正确设置项目 ID。

## 返回到所有者和请求者下拉框

最后，我们可以回到我们原来要做的事情，即将所有者和请求者字段更改为该项目的有效成员的下拉选择。为了正确地做到这一点，我们需要将一些用户与项目关联起来。由于用户管理是即将到来的章节的重点，我们将通过直接使用直接 SQL 将这些关联手动添加到数据库中来完成这一点。让我们使用以下 SQL 添加两个测试用户：

```php
INSERT INTO tbl_user (email, username, password) VALUES ('test1@notanaddress.com','User One', MD5('test1')), ('test2@notanaddress.com','User Two', MD5('test2'));
```

### 注意

我们在这里使用单向`MD5`哈希算法，因为它易于使用，并且在 MySQL 和 PHP 的 5.x 版本中广泛可用。然而，现在已经知道`MD5`作为单向哈希算法在安全方面是“破碎的”，不建议在生产环境中使用这个哈希算法。请考虑在您的真实生产应用程序中使用*Bcrypt*。以下是一些提供有关*Bcrypt*更多信息的网址：

[`en.wikipedia.org/wiki/Bcrypt`](http://en.wikipedia.org/wiki/Bcrypt)

[`php.net/manual/en/function.crypt.php`](http://php.net/manual/en/function.crypt.php)

[`www.openwall.com/phpass/`](http://www.openwall.com/phpass/)

当您在`trackstar`数据库上运行时，它将在我们的系统中创建两个具有 ID 1 和 2 的新用户。让我们也手动将这两个用户分配给项目#1，使用以下 SQL：

```php
INSERT INTO tbl_project_user_assignment (project_id, user_id) 
VALUES (1,1), (1,2);   
```

在运行前面的 SQL 语句之后，我们已经将两个有效成员分配给项目#1。

Yii 中关系型 Active Record 的一个很棒的特性是能够直接从问题`$model`实例中访问问题所属项目的有效成员。当我们使用 Gii 工具最初创建我们的问题模型类时，我们确保选中了**构建关系**复选框。这指示 Gii 查看底层数据库并定义相关的关系。这可以在`/protected/models/Issue.php`中的`relations()`方法中看到。由于我们在添加适当的关系到数据库后创建了这个类，该方法应该看起来像以下内容：

```php
      /**
   * @return array relational rules.
   */
  public function relations()
  {
    //NOTE: you may need to adjust the relation name and the related
    // class name for the relations automatically generated below.
    return array(
      'requester' => array(self::BELONGS_TO, 'User', 'requester_id'),
      'owner' => array(self::BELONGS_TO, 'User', 'owner_id'),
      'project' => array(self::BELONGS_TO, 'Project', 'project_id'),
    );
  }
```

前面代码片段中的`//NOTE`注释表明您可能具有略有不同的类属性名称，或者希望略有不同，并鼓励您根据需要进行调整。这个数组配置定义了模型实例上的属性，这些属性本身是其他 AR 实例。有了这些关系，我们可以非常容易地访问相关的 AR 实例。例如，假设我们想要访问与问题相关联的项目。我们可以使用以下语法来实现：

```php
//create the model instance by primary key:
$issue = Issue::model()->findByPk(1);
//access the associated Project AR instance
$project = $issue->project;
```

由于我们在数据库中定义其他表和关系之前创建了我们的`Project`模型类，因此尚未定义关系。但是现在我们已经定义了一些关系，我们需要将这些添加到`Project::relations()`方法中。打开项目 AR 类`/protected/models/Project.php`，并用以下内容替换整个`relations()`方法：

```php
   /**
     * @return array relational rules.
     */
    public function relations()
    {
return array(
            'issues' => array(self::HAS_MANY, 'Issue', 'project_id'),
            'users' => array(self::MANY_MANY, 'User', 'tbl_project_user_assignment(project_id, user_id)'),
        );
    }
```

有了这些，我们可以很容易地使用非常简单的语法访问与项目相关的所有问题和/或用户。例如：

```php
//instantiate the Project model instance by primary key:  
$project = Project::model()->findByPk(1);
//get an array of all associated Issue AR instances
$allProjectIssues = $project->issues;
//get an array of all associated User AR instance
$allUsers = $project->users;
//get the User AR instance representing the owner of 
//the first issue associated with this project
$ownerOfFirstIssue = $project->issues[0]->owner;
```

通常情况下，我们需要编写复杂的 SQL 连接语句来访问这样的相关数据。在 Yii 中使用关系 AR 可以避免这种复杂性和单调性。我们现在可以以非常优雅和简洁的面向对象方式访问这些关系，这样非常容易阅读和理解。

### 生成用于填充下拉框的数据

我们将采用与状态和类型下拉数据相似的方法来实现有效的用户下拉。我们将在我们的`Project`模型类中添加一个`getUserOptions()`方法。

打开文件`/protected/models/Project.php`，并在类的底部添加以下方法：

```php
/**
 * @return array of valid users for this project, indexed by user IDs
 */ 
public function getUserOptions()
{
  $usersArray = CHtml::listData($this->users, 'id', 'username');
      return $usersArray;
} 
```

在这里，我们使用 Yii 的`CHtml`辅助类来帮助我们从与项目相关的每个用户创建一个`id=>username`对的数组。记住，在项目类中`relations()`方法中定义的`users`属性映射到用户 AR 实例的数组。`CHtml::listData()`方法可以接受这个列表，并产生一个适合`CActiveForm::dropDownList()`的有效数组格式。

现在我们的`getUserOptions()`方法返回我们需要的数据，我们应该实现下拉框以显示返回的数据。我们已经使用过滤器从`$_GET`请求中设置了关联的项目 ID，并且我们在`IssueController::actionCreate()`方法的开头使用了这个值来设置新问题实例的`project_id`属性。所以现在，通过 Yii 关系 AR 功能的美妙力量，我们可以轻松地使用关联的`Project`模型来填充我们的用户下拉框。以下是我们在问题表单中需要做的更改：

打开包含输入表单元素的视图文件`/protected/views/issue/_form.php`，找到`owner_id`和`requester_id`的两个文本输入字段表单元素定义，并用以下代码替换它：

```php
<?php echo $form->textField($model,'owner_id'); ?>
with this:
<?php echo $form->dropDownList($model,'owner_id', $model->project->getUserOptions()); ?>
and also replace this line:
<?php echo $form->textField($model,'requester_id'); ?>
with this:
<?php echo $form->dropDownList($model,'requester_id', $model->project->getUserOptions()); ?>
```

现在，如果我们再次查看我们的问题创建表单，我们会看到**所有者**和**请求者**两个下拉框字段已经很好地填充了。

![生成用于填充下拉框的数据](img/8727_05_09.jpg)

### 进行最后一次更改

由于我们已经打开了创建问题表单视图文件，让我们快速进行最后一次更改。我们在每个表上都有用于基本历史和审计目的的创建时间和用户以及最后更新时间和用户字段，不应该暴露给用户。稍后我们将更改应用程序逻辑，以在插入和更新时自动填充这些字段。现在，让我们只是将它们从表单中移除。

从`/protected/views/issue/_form.php`中完全删除以下行：

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

以下截图显示了我们的新问题创建表单经过所有这些更改后的样子：

![进行最后一次更改](img/8727_05_10.jpg)

# CRUD 的其余部分

本章的目标是实现问题的所有 CRUD 操作。我们已经完成了创建功能，但我们仍需要完成问题的读取、更新和删除。幸运的是，通过使用 Gii CRUD 生成功能，大部分基础已经搭好了。但是，由于我们希望在项目的上下文中管理所有问题，我们需要对访问这些功能的方式进行一些调整。

## 列出问题

尽管`IssueController`类中有`actionIndex()`方法，用于显示数据库中所有问题的列表，但我们目前编写的功能并不需要这个功能。我们不想要一个单独的独立页面列出数据库中的所有问题，而是只想列出与特定项目相关联的问题。因此，我们将修改应用程序，以在项目详细信息页面上显示问题列表。由于我们正在利用 Yii 中的关联 AR 模型，所以对于这个改变来说将会非常容易。

### 修改项目控制器

首先让我们修改`ProjectController`类中的`actionView()`方法。因为我们想在同一个页面上显示与特定项目相关联的问题列表，我们可以在项目详细信息页面上做到这一点。`actionView()`方法是显示项目详细信息的方法。

将该方法修改为：

```php
    /**
         * Displays a particular model.
         * @param integer $id the ID of the model to be displayed
         */
        public function actionView($id)
        {
                $issueDataProvider=new CActiveDataProvider('Issue', array(
                        'criteria'=>array(
                                'condition'=>'project_id=:projectId',
                                'params'=>array(':projectId'=>$this->loadModel($id)->id),
                        ),
                        'pagination'=>array(
                                'pageSize'=>1,
                        ),
                 ));

                $this->render('view',array(
                        'model'=>$this->loadModel($id),
                        'issueDataProvider'=>$issueDataProvider,
                ));

        }
```

在这里，我们使用`CActiveDataProvider`框架类来使用`CActiveRecord`对象提供数据。它将使用关联的 AR 模型类以一种非常容易与内置的框架列表组件`CListView`一起使用的方式从数据库中检索数据。我们将使用这个组件在视图文件中显示我们的问题列表。我们使用了 criteria 属性来指定条件，它应该只检索与正在显示的项目相关联的问题。我们还使用了 pagination 属性来限制问题列表每页只显示一个问题。我们将这个值设置得很低，这样我们只需添加另一个问题就可以快速演示分页功能。我们很快会演示这一点。

我们做的最后一件事是将这个数据提供程序添加到`render()`调用中定义的数组中，以便在视图文件中以`$issueDataProvider`变量的形式提供给它。

### 修改项目视图文件

正如我们刚才提到的，我们将使用一个名为`CListView`的框架组件在项目详细信息页面上显示我们的问题列表。打开`/protected/views/project/view.php`并将以下内容添加到文件的底部：

```php
<br />
<h1>Project Issues</h1>

<?php $this->widget('zii.widgets.CListView', array(
  'dataProvider'=>$issueDataProvider,
  'itemView'=>'/issue/_view',
)); ?>
```

在这里，我们将`CListView`的`dataProvider`属性设置为我们上面创建的问题数据提供程序。然后我们配置它使用`protected/views/issue/_view.php`文件作为渲染数据提供程序中每个项目的模板。当我们为问题生成 CRUD 时，Gii 工具已经为我们创建了这个文件。我们在这里使用它来显示项目详细信息页面上的问题。

### 注意

您可能还记得在第一章*认识 Yii*中，**Zii**是 Yii 框架附带的官方扩展库。这些扩展是由核心 Yii 框架团队开发和维护的。您可以在这里阅读更多关于 Zii 的信息：[`www.yiiframework.com/doc/guide/1.1/en/extension.use#zii-extensions`](http://www.yiiframework.com/doc/guide/1.1/en/extension.use#zii-extensions)

我们还需要对我们指定为每个问题的布局模板的`/protected/views/issue/_view.php`文件进行一些更改。将该文件的整个内容修改为以下内容：

```php
<div class="view">

  <b><?php echo CHtml::encode($data->getAttributeLabel('name')); ?>:</b>
  <?php echo CHtml::link(CHtml::encode($data->name), array('issue/view', 'id'=>$data->id)); ?>
  <br />

  <b><?php echo CHtml::encode($data->getAttributeLabel('description')); ?>:</b>
  <?php echo CHtml::encode($data->description); ?>
  <br />

  <b><?php echo CHtml::encode($data->getAttributeLabel('type_id')); ?>:</b>
  <?php echo CHtml::encode($data->type_id); ?>
<br />

  <b><?php echo CHtml::encode($data->getAttributeLabel('status_id')); ?>:</b>
  <?php echo CHtml::encode($data->status_id); ?>

</div>
```

现在，如果我们保存并查看我们的结果，查看项目编号 1 的项目详细信息页面（`http://localhost/trackstar/index.php?r=project/view&id=1`），并假设您已经在该项目下创建了至少一个示例问题（如果没有，请在此页面使用**创建问题**链接创建一个），我们应该看到以下截图中显示的内容：

![修改项目视图文件](img/8727_05_11.jpg)

由于我们将数据提供程序的分页属性设置得非常低（记住我们只设置为 1），我们可以添加一个问题来演示内置的分页功能。添加一个问题会改变问题的显示，使我们能够在项目问题列表中从一页到另一页，如下截图所示：

![修改项目视图文件](img/8727_05_12.jpg)

# 最后的微调

现在我们有了与项目相关联的问题列表，并在项目详细信息页面上显示它们。我们还可以查看问题的详细信息（即阅读它们），以及更新和删除问题的链接。因此，我们的基本 CRUD 操作已经就位。

然而，在完成应用程序的这一部分之前，还有一些问题需要解决。我们会注意到问题显示列表显示了**类型**，**状态**，**所有者**和**请求者**字段的数字 ID 号。我们应该更改这样，以便显示这些字段的文本值。此外，由于问题已经属于特定项目，将项目 ID 显示为问题列表数据的一部分有点多余。因此，我们可以删除它。最后，我们需要解决一些导航链接，这些链接显示在各种其他与问题相关的表单上，以确保我们始终返回到此项目详细信息页面，作为我们所有问题管理的起始位置。

我们将逐一解决这些问题。

## 获取状态和类型文本以显示

以前，我们在`Issue` AR 类中添加了公共方法，以检索状态和类型选项，以填充问题创建表单上的下拉菜单。我们需要在这个 AR 类上添加类似的方法，以返回特定状态或类型 ID 的文本。

在`Issue`模型类(`/protected/models/Issue.php`)中添加以下两个新的公共方法，以检索当前问题的状态和类型文本：

```php
   /**
   * @return string the status text display for the current issue
   */ 
  public function getStatusText()
  {
    $statusOptions=$this->statusOptions;
    return isset($statusOptions[$this->status_id]) ? $statusOptions[$this->status_id] : "unknown status ({$this->status_id})";
  }

  /**
   * @return string the type text display for the current issue
   */ 
  public function getTypeText()
  {
    $typeOptions=$this->typeOptions;
    return isset($typeOptions[$this->type_id]) ? $typeOptions[$this->type_id] : "unknown type ({$this->type_id})";
  }
```

这些方法返回状态文本值（"尚未开始"，"已开始"或"已完成"）和类型文本值（"错误"，"功能"或"任务"）的`Issue`实例。

### 向表单添加文本显示

现在我们有了两个新的公共方法，它们将返回我们列表显示的有效状态和类型文本，我们需要利用它们。更改`/protected/views/issue/_view.php`中的以下代码行。

将这个`<?php echo CHtml::encode($data->type_id); ?>`更改为这个：

```php
<?php echo CHtml::encode($data->getTypeText()); ?>
```

将这个`<?php echo CHtml::encode($data->status_id); ?>`更改为这个：

```php
<?php echo CHtml::encode($data->getStatusText()); ?>
```

经过这些更改，我们**项目＃1**的问题列表页面，`http://localhost/trackstar/index.php?r=issue&pid=1`，不再显示我们问题类型和状态字段的整数值。现在看起来像以下截图中显示的样子：

![向表单添加文本显示](img/8727_05_13.jpg)

由于我们使用相同的视图文件在项目详细信息页面上显示我们的问题列表，这些更改也会反映在那里。

## 更改问题详细视图

我们还需要对问题的详细视图进行一些其他更改。当前，如果查看问题详细信息，它显示如下截图所示：

![更改问题详细视图](img/8727_05_14.jpg)

这是使用我们尚未更改的视图文件。它仍然显示项目 ID，我们不需要显示，以及**类型**和**状态**作为整数值，而不是它们关联的文本值。打开用于呈现此显示的视图文件`/protected/views/issue/view.php`，我们注意到它使用了我们以前没有见过的 Zii 扩展小部件`CDetailView`。这类似于用于显示列表的`CListView`小部件，但用于显示单个数据模型实例的详细信息，而不是用于显示许多列表视图。以下是显示此小部件使用的相关代码：

```php
<?php $this->widget('zii.widgets.CDetailView', array(
  'data'=>$model,
  'attributes'=>array(
    'id',
    'name',
    'description',
    'project_id',
    'type_id',
    'status_id',
    'owner_id',
    'requester_id',
    'create_time',
    'create_user_id',
    'update_time',
    'update_user_id',
  ),
)); ?>
```

在这里，我们将`CDetailView`小部件的数据模型设置为`Issue`模型类的实例（即我们要显示详细信息的特定实例），然后设置要在渲染的详细视图中显示的模型实例的属性列表。属性可以被指定为`Name:Type:Label`格式的字符串，其中`Type`和`Label`都是可选的，或者作为数组本身。在这种情况下，只指定属性的名称。

如果我们将属性指定为数组，我们可以通过声明值元素来进一步自定义显示。我们将采取这种方法，以指定模型类方法`Issue::getTypeText()`和`Issue::getStatusText()`用于获取**类型**和**状态**字段的文本值。

让我们将`CDetailView`的使用更改为以下配置：

```php
<?php $this->widget('zii.widgets.CDetailView', array(
  'data'=>$model,
  'attributes'=>array(
    'id',
    'name',
    'description',
    array(        
      'name'=>'type_id',
        'value'=>CHtml::encode($model->getTypeText())
    ),
    array(        
      'name'=>'status_id',
        'value'=>CHtml::encode($model->getStatusText())
    ),
    'owner_id',
    'requester_id',
    ),
)); ?>
```

在这里，我们已经删除了一些属性的显示，即`project_id`，`create_time`，`update_time`，`create_user_id`和`update_user_id`属性。我们稍后会处理一些这些属性的填充和显示，但现在我们可以将它们从详细显示中删除。

我们还改变了`type_id`和`status_id`属性的声明，以使用数组规范，以便我们可以使用值元素。我们已经指定了相应的`Issue::getTypeText()`和`Issue::getStatusText()`方法用于获取这些属性的值。有了这些改变，查看问题详细页面显示如下：

![更改问题详细视图](img/8727_05_15.jpg)

好的，我们离我们想要的更近了，但还有一些改变我们需要做。

## 显示所有者和请求者的名称

事情看起来更好了，但我们仍然看到整数标识符被显示为**所有者**和**请求者**，而不是实际的用户名。我们将采取类似的方法来处理类型和状态文本显示。我们将在`Issue`模型类上添加两个新的公共方法，以返回这两个属性的名称。

### 使用关联 AR

由于我们的问题和用户分别表示为单独的数据库表，并通过外键关系相关联，我们可以直接从视图文件中的`$model`中访问`owner`和`requester`用户名。利用 Yii 的关联 AR 模型功能，显示相关`User`模型类实例的用户名属性非常简单。

正如我们所提到的，模型类`Issue::relations()`方法是定义关系的地方。如果我们来看一下这个方法，我们会看到以下内容：

```php
/**
   * @return array relational rules.
   */
  public function relations()
  {
    // NOTE: you may need to adjust the relation name and 
//the related class name for the relations automatically generated 
//below.
    return array(
 **'owner' => array(self::BELONGS_TO, 'User', 'owner_id'),**
      'project' => array(self::BELONGS_TO, 'Project', 'project_id'),
 **'requester' => array(self::BELONGS_TO, 'User', 'requester_id'),**
    );
  }
```

突出显示的代码是我们需求最相关的。`owner`和`requester`属性都被定义为与`User`模型类的关系。这些定义指定这些属性的值是`User`模型类的实例。`owner_id`和`requester_id`参数指定了它们各自`User`类实例的唯一主键。因此，我们可以像访问`Issue`模型类的其他属性一样访问这些属性。

为了显示所有者和请求者`User`类实例的用户名，我们再次将`CDetailView`配置更改为以下内容：

```php
<?php $this->widget('zii.widgets.CDetailView', array(
  'data'=>$model,
  'attributes'=>array(
    'id',
    'name',
    'description',
    array(        
      'name'=>'type_id',
        'value'=>CHtml::encode($model->getTypeText())
    ),
    array(        
      'name'=>'status_id',
        'value'=>CHtml::encode($model->getStatusText())
    ),
 **array(** 
 **'name'=>'owner_id',**
 **'value'=>isset($model->owner)?CHtml::encode($model->owner->username):"unknown"**
 **),**
 **array(** 
 **'name'=>'requester_id',**
 **'value'=>isset($model->requester)?CHtml::encode($model->requester->username):"unknown"      ),**
  ),
)); ?>
```

做出这些改变后，我们的问题详细列表开始看起来相当不错。以下截图显示了我们迄今为止取得的进展：

![使用关联 AR](img/8727_05_16.jpg)

## 做一些最终的导航调整

我们非常接近完成本章中设定要实现的功能。唯一剩下的事情就是稍微清理一下我们的导航。您可能已经注意到，仍然有一些选项可供用户在项目上下文之外导航到整个问题列表，或者创建一个新问题。对于我们的 TrackStar 应用程序，我们对问题的所有操作都应该在特定项目的上下文中进行。我们之前已经强制要求在创建新问题时使用项目上下文，这是一个很好的开始，但我们仍然需要做一些改变。

我们会注意到的一件事是，应用程序仍然允许用户导航到跨所有项目的所有问题列表。例如，在问题详情页面，如`http://localhost/trackstar/index.php?r=issue/view&id=1`，我们看到右侧菜单导航中有**问题列表**和**管理问题**的链接，分别对应`http://localhost/trackstar/index.php?r=issue/index`和`http://localhost/trackstar/index.php?r=issue/admin`（请记住，要访问管理页面，您必须以`admin/admin`身份登录）。这些链接仍然显示所有项目的所有问题。因此，我们需要将此列表限制为特定项目。

由于这些链接源自问题详情页面，并且特定问题有关联的项目，我们可以首先修改链接以传递特定项目 ID，然后将该项目 ID 作为限制问题查询的条件，分别在`IssueController::actionIndex()`和`IssueController::actionAdmin()`方法中使用。

首先让我们修改链接。打开`/protected/views/issue/view.php`文件，找到文件顶部的菜单项数组。将菜单配置更改为：

```php
$this->menu=array(
 **array('label'=>'List Issues', 'url'=>array('index', 'pid'=>$model->project->id)),**
 **array('label'=>'Create Issue', 'url'=>array('create', 'pid'=>$model->project->id)),**
  array('label'=>'Update Issue', 'url'=>array('update', 'id'=>$model->id)),
  array('label'=>'Delete Issue', 'url'=>'#', 'linkOptions'=>array('submit'=>array('delete','id'=>$model->id),'confirm'=>'Are you sure you want to delete this item?')),
 **array('label'=>'Manage Issues', 'url'=>array('admin', 'pid'=>$model->project->id)),**
);
```

所做的更改已经突出显示。我们已经在**创建问题**链接以及问题列表页面和问题管理列表页面中添加了一个新的查询字符串参数。我们已经知道我们必须对创建链接进行此更改，因为我们先前实施了一个过滤器，以强制在创建新问题之前提供有效的项目上下文。相对于此链接，我们不需要进行进一步的更改。但是对于索引和管理链接，我们需要修改它们对应的操作方法以使用这个新的查询字符串变量。

由于我们已经配置了一个过滤器来使用查询字符串变量加载关联的项目，让我们利用这一点。我们将添加到过滤器配置中，以便在执行`IssueController::actionIndex()`和`IssueController::actionAdmin()`方法之前调用我们的过滤器方法。将`IssueController::filters()`方法更改为：

```php
public function filters()
  {
    return array(
      'accessControl', // perform access control for CRUD operations
 **'projectContext + create index admin', //perform a check to ensure valid project context** 
    );
  }
```

有了这个设置，关联的项目将被加载并可供使用。让我们在`IssueController::actionIndex()`方法中使用它。修改该方法为：

```php
  public function actionIndex()
  {
**$dataProvider=new CActiveDataProvider('Issue', array(**
 **'criteria'=>array(**
 **'condition'=>'project_id=:projectId',**
 **'params'=>array(':projectId'=>$this->_project->id),**
 **),**
 **));**
    $this->render('index',array(
      'dataProvider'=>$dataProvider,
    ));
  }
```

在这里，与以前一样，我们只是在创建模型数据提供程序的条件中添加了一个条件，以仅检索与项目相关的问题。这将限制问题列表仅显示项目下的问题。

我们需要对管理列表页面进行相同的更改。但是，这个视图文件`/protected/views/issue/admin.php`正在使用模型类`Issue::search()`方法的结果来提供问题列表。因此，我们实际上需要对这个列表强制执行项目上下文进行两次更改。

首先，我们需要修改`IssueController::actionAdmin()`方法，以在将模型实例发送到视图时设置正确的`project_id`属性。以下突出显示的代码显示了这个必要的更改：

```php
public function actionAdmin()
  {
    $model=new Issue('search');

    if(isset($_GET['Issue']))
      $model->attributes=$_GET['Issue'];

 **$model->project_id = $this->_project->id;**

    $this->render('admin',array(
      'model'=>$model,
    ));
  }
```

然后我们需要在`Issue::search()`模型类方法中添加到我们的条件。以下突出显示的代码标识了我们需要对这个方法进行的更改：

```php
public function search()
  {
    // Warning: Please modify the following code to remove attributes that
    // should not be searched.

    $criteria=new CDbCriteria;

    $criteria->compare('id',$this->id);

    $criteria->compare('name',$this->name,true);

    $criteria->compare('description',$this->description,true);

    $criteria->compare('type_id',$this->type_id);

    $criteria->compare('status_id',$this->status_id);

    $criteria->compare('owner_id',$this->owner_id);

    $criteria->compare('requester_id',$this->requester_id);

    $criteria->compare('create_time',$this->create_time,true);

    $criteria->compare('create_user_id',$this->create_user_id);

    $criteria->compare('update_time',$this->update_time,true);

    $criteria->compare('update_user_id',$this->update_user_id);

 **$criteria->condition='project_id=:projectID';

    $criteria->params=array(':projectID'=>$this->project_id);**

    return new CActiveDataProvider(get_class($this), array(
      'criteria'=>$criteria,
    ));
  }
```

在这里，我们使用`$criteria->condition()`直接移除了`$criteria->compare()`调用，该调用使用`project_id`的值必须完全等于我们的项目上下文。有了这些变化，管理页面上列出的问题现在被限制为仅与特定项目相关联的问题。

### 注意

在`/protected/views/issues/`下的视图文件中有几个地方包含需要添加`pid`查询字符串才能正常工作的链接。我们将其留给读者根据这些示例提供的相同方法进行适当的更改。随着我们应用程序的开发，我们将假设所有创建新问题或显示问题列表的链接都已正确格式化，以包含适当的`pid`查询字符串参数。

# 总结

在本章中，我们涵盖了许多不同的主题。根据我们应用程序中*问题*、*项目*和*用户*之间的关系，我们在本章中实现问题管理功能的复杂性明显比我们在上一章中处理的项目实体管理要复杂得多。幸运的是，Yii 能够多次帮助我们减轻编写所有需要解决这种复杂性的代码的痛苦。

我们依靠我们的好朋友 Gii 来创建 Active Record 模型，以及对问题实体进行所有基本 CRUD 操作的初始实现。我们再次使用 Yii 迁移来帮助实现我们需要的数据库架构更改，以支持我们的问题功能。我们使用了 Yii 中的关联 Active Record，并看到使用这一特性轻松检索相关的数据库信息。我们引入了控制器过滤器作为一种手段，以在控制器动作方法之前和/或之后实现业务逻辑并进入请求生命周期。我们演示了如何在 Yii 表单中使用下拉菜单。

到目前为止，我们在基本应用程序上取得了很大进展，而且在不必编写大量代码的情况下完成了这一切。Yii 框架本身已经完成了大部分繁重的工作。我们现在有一个可以管理项目并管理项目中问题的工作应用程序。这是我们的应用程序试图实现的核心。到目前为止，我们应该为取得的成就感到自豪。

然而，在这个应用程序真正准备投入生产使用之前，我们还有很长的路要走。一个主要缺失的部分是围绕用户管理的所有必需功能。在接下来的两章中，我们将深入研究用户认证和授权。我们将首先展示 Yii 用户认证的工作原理，并开始对我们的用户进行认证，以验证他们的用户名和密码是否存储在数据库中。
