# 第六章。用户管理和身份验证

我们在很短的时间内取得了很大的进展。我们已经奠定了 TrackStar 应用程序的基本基础。现在我们可以管理项目和项目内的问题，这是该应用程序的主要目的。当然，还有很多工作要做。

回到第三章 *TrackStar 应用程序*，当我们介绍这个应用程序时，我们将其描述为一个基于用户的应用程序，它提供了创建用户帐户并在用户经过身份验证和授权后授予对应用程序功能的能力。为了使这个应用程序对不止一个人有用，我们需要添加在项目内管理用户的能力。这将是接下来两章的重点。

# 功能规划

当我们使用`yiic`命令行工具最初创建 TrackStar 应用程序时，我们注意到基本的登录功能已经为我们自动创建。登录页面允许两个用户名/密码凭据组合，`demo/demo`和`admin/admin`。您可能还记得我们必须登录到应用程序中，以便在前两章中对项目和问题实体执行一些 CRUD 操作。

这个基本的身份验证骨架代码提供了一个很好的开始，但我们需要做一些改变，以支持任意数量的用户。我们还需要向应用程序添加用户 CRUD 功能，以便我们可以管理这些多个用户。本章将重点介绍扩展身份验证模型以使用`tbl_user`数据库表，并添加所需功能以允许基本用户数据管理。

为了实现上述目标，我们需要处理以下事项：

+   创建将包含允许我们执行以下功能的控制器类：

+   创建新用户

+   从数据库中检索现有用户的列表

+   更新/编辑现有用户

+   删除现有用户

+   创建视图文件和表示层逻辑，将：

+   显示表单以允许创建新用户

+   显示所有现有用户的列表

+   显示表单以允许编辑现有用户

+   添加删除按钮，以便我们可以删除用户

+   调整创建新用户表单，以便外部用户可以使用自注册流程

+   修改身份验证过程，以使用数据库验证登录凭据。

# 用户 CRUD

由于我们正在构建一个基于用户的 Web 应用程序，我们必须有一种方法来添加和管理用户。我们在第五章 *管理问题*中向数据库添加了`tbl_user`表。您可能还记得我们留给读者的练习是创建相关的 AR 模型类。如果您正在跟着做，并且没有创建必要的用户模型类，现在需要这样做。

以下是使用 Gii 代码创建工具创建模型类的简要提醒：

1.  通过`http://localhost/trackstar/index.php?r=gii`导航到 Gii 工具，并选择**Model Generator**链接。

1.  将表前缀保留为`tbl_`。在**Table Name**字段中填写`tbl_user`，这将自动填充**Model Class**名称字段为**User**。

1.  填写表单后，单击**Preview**按钮，获取一个链接到弹出窗口，显示即将生成的所有代码。

1.  最后，单击**Generate**按钮，实际创建新的`User.php`模型类文件在`/protected/models/`目录中。

有了`User` AR 类，创建 CRUD 脚手架就变得很简单。我们以前使用过 Gii 工具做过这个。提醒一下，以下是必要的步骤：

1.  通过`http://localhost/trackstar/index.php?r=gii`导航到工具。

1.  从可用生成器列表中单击**Crud Generator**链接。

1.  在**Model Class**名称字段中键入`User`。相应的**Controller ID**将自动填充为**User**。

1.  然后，您将看到在生成之前预览每个文件的选项。单击**生成**按钮，它将在适当的位置生成所有相关的 CRUD 文件。

有了这个，我们可以在`http://localhost/trackstar/index.php?r=user/index`查看我们的用户列表页面。在上一章中，我们手动创建了一些用户，以便我们可以正确处理项目、问题和用户之间的关系。这就是为什么我们在这个页面上看到了一些用户。以下截图显示了我们如何显示这个页面：

![用户 CRUD](img/8727_06_01.jpg)

我们还可以通过访问`http://localhost/trackstar/index.php?r=user/create`来查看新的**创建用户**表单。如果您当前未登录，您将首先被路由到登录页面，然后才能查看表单。因此，您可能需要使用`demo/demo`或`admin/admin`登录以查看此表单。

在我们首先在项目实体上，然后再次在问题上创建和使用 CRUD 操作功能后，我们现在非常熟悉这些功能最初是如何由 Gii 代码生成工具实现的。用于创建和更新的生成代码是一个很好的开始，但需要一些调整以满足特定的应用程序要求。我们刚刚为创建新用户生成的表单也不例外。它为在`tbl_user`表中定义的每个列都有一个输入表单字段。我们不希望将所有这些字段都暴露给用户输入。最后登录时间、创建时间和用户以及更新时间和用户的列应在提交表单后以编程方式设置。

## 更新我们的常见审计历史列

回到之前的章节，当我们介绍我们的**项目**和**问题**CRUD 功能时，我们还注意到我们的表单有比应该更多的输入字段。由于我们已经定义了所有的数据库表都有相同的创建和更新时间和用户列，我们的每个自动生成的输入表单都暴露了这些字段。在第四章中处理项目创建表单时，我们完全忽略了这些字段，*项目 CRUD*。然后，在第五章中，*管理问题*，我们采取了一步措施，从表单中删除了这些字段的显示，但我们从未添加逻辑来在添加新行时正确设置这些值。

让我们花一点时间添加这个所需的逻辑。由于我们的实体表`tbl_project`、`tbl_issue`和`tbl_user`都定义了相同的列，我们可以将我们的逻辑添加到一个公共基类中，然后让每个单独的 AR 类从这个新的基类扩展。这是将相同功能应用于相同类型实体的常见方法。然而，Yii 组件——即`CComponent`的任何实例或`CComponent`的派生类，这通常是 Yii 应用程序中大多数类的情况——为您提供了另一种，可能更灵活的选择。

### 组件行为

Yii 中的行为是实现`IBehavior`接口的类，其方法可以通过附加到组件而不是显式扩展类来扩展组件的功能。行为可以附加到多个组件，组件可以附加多个行为。跨组件重用行为使它们非常灵活，通过能够将多个行为附加到同一个组件，我们能够为我们的 Yii 组件类实现一种*多重继承*。

我们将使用这种方法为我们的模型类添加所需的功能。我们采取这种方法的原因是，我们的其他模型类，`Issue`和`Project`，也需要相同的逻辑。与其在每个 AR 模型类中重复代码，将功能放在行为中，然后将行为附加到模型类中，将允许我们在一个地方为每个 AR 模型类正确设置这些字段。

为了让组件使用行为的方法，行为必须附加到组件上。这只需要在组件上调用`attachBehavior()`方法就可以了：

```php
$component->attachBehavior($name, $behavior);
```

在之前的代码中，`$name`是组件内行为的唯一标识符。一旦附加，组件就可以调用行为类中定义的方法：

```php
$component->myBehaviorMethod();
```

在之前的代码中，`myBehaviorMethod()`在`$behavior`类中被定义，但可以像在`$component`类中定义一样调用。

对于模型类，我们可以在`behaviors()`方法中添加我们想要的行为，这是我们将采取的方法。现在我们只需要创建一个要附加的行为。

事实上，Yii 框架打包的 Zii 扩展库已经有一个现成的行为，可以更新我们每个基础表上的日期时间列`create_time`和`update_time`。这个行为叫做`CTimestampBehavior`。所以，让我们开始使用这个行为。

让我们从我们的`User`模型类开始。将以下方法添加到`protected/models/User.php`中：

```php
public function behaviors() 
{
  return array(
     'CTimestampBehavior' => array(
       'class' => 'zii.behaviors.CTimestampBehavior',
       'createAttribute' => 'create_time',
       'updateAttribute' => 'update_time',
      'setUpdateOnCreate' => true,
    ),
   );
}
```

在这里，我们将 Zii 扩展库的`CTimestampBehavior`附加到我们的`User`模型类上。我们已经指定了创建时间和更新时间属性，并且还配置了行为，在创建新记录时设置更新时间。有了这个设置，我们可以试一下。创建一个新用户，你会看到`create_time`和`update_time`记录被自动插入。很酷，对吧？

![组件行为](img/8727_06_07.jpg)

这很棒，但我们需要在其他模型类中重复这个过程。我们可以在每个模型类中复制`behaviors()`方法，并且在添加更多模型类时继续这样做。或者，我们可以将其放在一个通用的基类中，并让我们的每个模型类扩展这个新的基类。这样，我们只需要定义一次`behaviors()`方法。

当我们保存和更新记录时，我们还需要插入我们的`create_user_id`和`update_user_id`列。我们可以以多种方式处理这个问题。由于一个组件可以附加多个行为，我们可以创建一个类似于`CTimestampBehavior`的新行为，用于更新创建和更新用户 ID 列。或者，我们可以简单地扩展`CTimestampBehavior`，并在这个子类中添加额外的功能。或者我们可以直接利用模型的`beforeSave`事件，并在那里设置我们需要的字段。在现实世界的应用中，扩展现有的行为以添加这个额外的功能可能是最合理的方法；然而，为了演示另一种方法，让我们直接利用活动记录的`beforeSave`事件，并在一个通用的基类中进行这个操作，所有我们的 AR 模型类都可以扩展这个基类。这样，当构建自己的 Yii 应用程序时，你将有机会接触到几种不同的方法，并有更多的选择。

所以，我们需要为我们的 AR 模型类创建一个新的基类。我们还将使这个新类成为`abstract`，因为它不应该直接实例化。首先，去掉`User` AR 类中的`behaviors()`方法，因为我们将把这个方法放在我们的基类中。然后创建一个新文件，`protected/models/TrackStarActiveRecord.php`，并添加以下代码：

```php
<?php
abstract class TrackStarActiveRecord extends CActiveRecord
{
   /**
   * Prepares create_user_id and update_user_id attributes before saving.
   */

  protected function beforeSave()
  {

    if(null !== Yii::app()->user)
      $id=Yii::app()->user->id;
    else
      $id=1;

    if($this->isNewRecord)
      $this->create_user_id=$id;

    $this->update_user_id=$id;

    return parent::beforeSave();
  }

  /**
   * Attaches the timestamp behavior to update our create and update times
   */
  public function behaviors() 
  {
    return array(
       'CTimestampBehavior' => array(
         'class' => 'zii.behaviors.CTimestampBehavior',
         'createAttribute' => 'create_time',
         'updateAttribute' => 'update_time',
        'setUpdateOnCreate' => true,
      ),
     );
  }

}
```

在这里，正如讨论的那样，我们正在重写`CActiveRecord::beforeSave()`方法。这是`CActiveRecord`公开的许多事件之一，允许定制其流程工作流。有两种方法可以让我们进入记录保存工作流程，并在活动记录保存之前或之后执行任何必要的逻辑：`beforeSave()`和`afterSave()`。在这种情况下，我们决定在保存活动记录之前明确设置我们的创建和更新用户字段，即在写入数据库之前。

我们通过使用属性`$this->isNewRecord`来确定我们是在处理新记录（即插入）还是现有记录（即更新），并相应地设置我们的字段。然后，我们确保调用父实现，通过返回`parent::beforeSave()`来确保它有机会做所有需要做的事情。我们对`Yii::app()->user`进行了`NULL`检查，以处理可能在 Web 应用程序上下文之外使用这个模型类的情况，例如在 Yii 控制台应用程序中（在后面的章节中介绍）。如果我们没有有效的用户，我们只是默认使用第一个用户，`id = 1`，我们可以设置为超级用户。

另外，正如讨论的那样，我们已经将`behaviors()`方法移到了这个基类中，这样所有扩展它的 AR 模型类都将具有这个行为附加。

为了尝试这个，我们现在需要修改现有的三个 AR 类`Project.php`，`User.php`和`Issue.php`，使其扩展自我们的新抽象类，而不是直接扩展自`CActiveRecord`。因此，例如，而不是以下内容：

```php
class User extends CActiveRecord
{
…}
```

我们需要有：

```php
class User extends TrackStarActiveRecord
{ 
…}
```

我们需要对我们的其他模型类进行类似的更改。

现在，如果我们添加另一个新用户，我们应该看到我们的所有四个审计历史列都填充了时间戳和用户 ID。

现在，这些更改已经就位，我们应该从创建新项目、问题和用户的每个表单中删除这些字段（我们已经在上一章中从问题表单中删除了它们）。这些表单字段的 HTML 位于`protected/views/project/_form.php`，`protected/views/issue/_form.php`和`protected/views/user/_form.php`文件中。我们需要从这些文件中删除的行如下所示：

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

并且从用户创建表单`protected/views/user/_form.php`中，我们也可以删除最后登录时间字段：

```php
<div class="row">
    <?php echo $form->labelEx($model,'last_login_time'); ?>
    <?php echo $form->textField($model,'last_login_time'); ?>
    <?php echo $form->error($model,'last_login_time'); ?>
  </div>
```

由于我们正在从表单输入中删除这些字段，我们还应该删除相关规则方法中为这些字段定义的验证规则。这些验证规则旨在确保用户提交的数据有效且格式正确。删除规则还可以防止它们成为当我们获取所有提交的查询字符串或 POST 变量并将它们的值分配给我们的 AR 模型属性时的批量分配的一部分。例如，在 AR 模型的创建和更新控制器操作中，我们看到以下行：

```php
$model->attributes=$_POST['User'];
```

这是对从提交的表单字段中的所有模型属性进行批量分配。作为一项额外的安全措施，这仅适用于为其分配了验证规则的属性。您可以使用`CSafeValidator`来标记模型属性，以便将其作为这种批量分配的安全属性。

由于这些字段不会由用户填写，并且我们不需要它们被大规模分配，我们可以删除这些规则。

好的，让我们把它们删除。打开`protected/models/User.php`，在`rules()`方法中删除以下两条规则：

```php
array('create_user_id, update_user_id', 'numerical', 'integerOnly'=>true),
array('last_login_time, create_time, update_time', 'safe'),
```

项目和问题 AR 类定义了类似的规则，但并非完全相同。在删除这些规则时，请确保保留仍适用于用户输入字段的规则。

上面删除`last_login_time`属性的规则是有意的。我们也应该将其从用户输入字段中删除。这个字段需要在成功登录后自动更新。由于我们已经打开了视图文件并删除了其他字段，我们决定现在也删除这个字段。但是，在我们进行其他一些更改并涵盖其他一些主题之后，我们将等待添加必要的应用程序逻辑。

实际上，当我们还在`User`类的验证规则方法中时，我们应该做出另一个改变。我们希望确保每个用户的电子邮件和用户名都是唯一的。我们应该在提交表单时验证这一要求。此外，我们还应该验证提交的电子邮件数据是否符合标准的电子邮件格式。您可能还记得在第四章中，我们介绍了 Yii 的内置验证器，其中有两个非常适合我们的需求。我们将使用`CEmailValidator`和`CUniqueValidator`类来满足我们的验证需求。我们可以通过在`rules()`方法中添加以下两行代码来快速添加这些规则：

```php
array('email, username', 'unique'),
array('email', 'email'),
```

整个`User::rules()`方法现在应该如下所示：

```php
public function rules()
  {
    // NOTE: you should only define rules for those attributes that
    // will receive user inputs.
    return array(
      array('email', 'required'),
array('email, username, password', 'length', 'max'=>255,
array('email, username', 'unique'),
array('email', 'email'),
      // The following rule is used by search().
      // Please remove those attributes that should not be searched.
      array('id, email, username, password, last_login_time, create_time, create_user_id, update_time, update_user_id', 'safe', 'on'=>'search'),
    );
  }
```

上面规则中的*unique*声明是一个别名，指的是 Yii 的内置验证器`CUniqueValidator`。这验证了模型类属性在底层数据库表中的唯一性。通过添加这个验证规则，当尝试输入已经存在于数据库中的电子邮件和/或用户名时，我们将收到一个错误。此外，通过添加电子邮件验证，当电子邮件表单字段中的值不是正确的电子邮件格式时，我们将收到一个错误。

在上一章中创建`tbl_user`表时，我们添加了两个测试用户，以便我们有一些数据可以使用。这两个用户中的第一个用户的电子邮件地址是`test1@notanaddress.com`。尝试使用相同的电子邮件添加另一个用户。以下截图显示了尝试后收到的错误消息以及错误字段的高亮显示：

![组件行为](img/8727_06_02.jpg)

提交一个不符合有效电子邮件格式的值也会产生错误消息。

## 添加密码确认字段

除了刚刚做的更改之外，我们还应该添加一个新字段，强制用户确认他们输入的密码。这是用户注册表单上的标准做法，有助于用户在输入这一重要信息时不出错。幸运的是，Yii 还带有另一个内置的验证器`CCompareValidator`，它正是你所想的那样。它比较两个属性的值，如果它们不相等，则返回错误。

为了利用这个内置的验证，我们需要在我们的模型类中添加一个新的属性。在`User`模型 AR 类的顶部添加以下属性：

```php
public $password_repeat;
```

我们通过在要比较的属性名称后附加`_repeat`来命名此属性。比较验证器允许您指定任意两个属性进行比较，或将属性与常量值进行比较。如果在声明比较规则时未指定比较属性或值，它将默认查找以与要比较的属性相同的名称开头的属性，并在末尾附加`_repeat`。这就是我们以这种方式命名属性的原因。现在我们可以在`User::rules()`方法中添加一个简单的验证规则，如下所示：

```php
array('password', 'compare'),
```

如果不使用`_repeat`约定，您需要指定要执行比较的属性。例如，如果我们想要将`$password`属性与名为`$confirmPassword`的属性进行比较，我们可以使用：

```php
array('password', 'compare', 'compareAttribute'=>'confirmPassword'),
```

由于我们已经明确将`$password_repeat`属性添加到用户 AR 类中，并且没有为其定义验证规则，因此当调用`setAttributes()`方法时，我们还需要告诉模型类允许以批量方式设置此字段。如前所述，我们通过将新属性明确添加到`User`模型类的*safe*属性列表中来实现这一点。要做到这一点，请将以下内容添加到`User::rules()`数组中：

```php
array('password_repeat', 'safe'),
```

让我们对验证规则做出一次更改。我们当前在用户表单上拥有的所有字段都应该是必填的。目前，我们的必填规则只适用于`email`字段。在我们对`User::rules()`方法进行更改时，让我们也将用户名和密码添加到此列表中：

```php
array('email, username, password, password_repeat', 'required'),
```

### 注意

有关验证规则的更多信息，请参见：[`www.yiiframework.com/doc/guide/1.1/en/form.model#declaring-validation-rules`](http://www.yiiframework.com/doc/guide/1.1/en/form.model#declaring-validation-rules)

好的，现在我们所有的规则都已设置。但是，我们仍然需要向表单添加密码确认字段。现在让我们来做这件事。

要添加此字段，请打开`protected/views/user/_form.php`，并在密码字段下方添加以下代码块：

```php
<div class="row">
    <?php echo $form->labelEx($model,'password_repeat'); ?>
    <?php echo $form->passwordField($model,'password_repeat',array('size'=>60,'maxlength'=>255)); ?>
    <?php echo $form->error($model,'password_repeat'); ?>
  </div>
```

在所有这些表单更改就位后，**创建用户**表单应如下截图所示：

![添加密码确认字段](img/8727_06_03.jpg)

现在，如果我们尝试使用**密码**和**密码重复**字段中的不同值提交表单，我们将会收到如下截图所示的错误：

![添加密码确认字段](img/8727_06_04.jpg)

## 对密码进行哈希处理

在我们离开新用户创建过程之前，我们应该做的最后一个更改是在将用户的密码存储到数据库之前创建其哈希版本。在将敏感用户信息添加到持久存储之前应用单向哈希算法是一种非常常见的做法。

我们将利用`CActiveRecord`的另一种方法来将此逻辑添加到`User.php` AR 类中，该方法允许我们自定义默认的活动记录工作流程。这次我们将重写`afterValidate()`方法，并在验证所有输入字段但在保存记录之前对密码应用基本的单向哈希。

### 注意

与我们在设置创建和更新时间戳时使用`CActiveRecord::beforeSave()`方法类似，这里我们正在重写`CActiveRecord::beforeValidate()`方法。这是`CActiveRecord`公开的许多事件之一，允许自定义其流程工作流程。快速提醒一下，如果在调用 AR 类的`save()`方法时没有显式发送`false`作为参数，验证过程将被触发。该过程执行 AR 类中`rules()`方法中指定的验证。有两种公开的方法允许我们进入验证工作流程并在验证执行之前或之后执行任何必要的逻辑，即`beforeValidate()`和`afterValidate()`。在这种情况下，我们决定在执行验证后立即对密码进行哈希处理。

打开`User` AR 类，并在类底部添加以下内容：

```php
    /**
   * apply a hash on the password before we store it in the database
   */
  protected function afterValidate()
  {   
    parent::afterValidate();
  if(!$this->hasErrors())
      $this->password = $this->hashPassword($this->password);
  }

  /**
   * Generates the password hash.
   * @param string password
     * @return string hash
   */
    public function hashPassword($password)
  {
    return md5($password);
  }
```

### 注意

我们在上一章中提到过这一点，但值得再次提及。我们在这里使用单向 MD5 哈希算法是因为它易于使用，并且在 MySQL 和 PHP 的 5.x 版本中广泛可用。然而，现在已经知道 MD5 在安全方面作为单向哈希算法是“破解”的，因此不建议在生产环境中使用此哈希算法。请考虑在真正的生产应用程序中使用 Bcrypt。以下是一些提供有关 Bcrypt 更多信息的网址：

+   [`en.wikipedia.org/wiki/Bcrypt`](http://en.wikipedia.org/wiki/Bcrypt)

+   [`php.net/manual/en/function.crypt.php`](http://php.net/manual/en/function.crypt.php)

+   [`www.openwall.com/phpass/`](http://www.openwall.com/phpass/)

有了这个配置，它将在所有其他属性验证成功通过之后对密码进行哈希处理。

### 注意

这种方法对于全新的记录来说效果很好，但是对于更新来说，如果用户没有更新他/她的密码信息，就有可能对已经进行过哈希处理的值再次进行哈希处理。我们可以用多种方式来处理这个问题，但是为了简单起见，我们需要确保每次用户想要更新他们的用户数据时，我们都要求他们提供有效的密码。

现在我们有能力向我们的应用程序添加新用户。由于我们最初使用 Gii 工具的**Crud Generator**链接创建了这个表单，我们还为用户拥有了读取、更新和删除功能。通过添加一些新用户，查看他们的列表，更新一些信息，然后删除一些条目来测试一下，确保一切都按预期工作。（请记住，您需要以`admin`身份登录，而不是`demo`，才能执行删除操作。）

# 使用数据库对用户进行认证

正如我们所知，通过使用`yiic`命令创建我们的新应用程序，为我们创建了一个基本的登录表单和用户认证过程。这种认证方案非常简单。它会检查输入表单的用户名/密码值，如果它们是`demo/demo`或`admin/admin`，就会通过，否则就会失败。显然，这并不是一个永久的解决方案，而是一个构建的基础。我们将通过改变认证过程来使用我们已经作为模型的一部分拥有的`tbl_user`数据库表来构建。但在我们开始改变默认实现之前，让我们更仔细地看一下 Yii 是如何实现认证模型的。

## 介绍 Yii 认证模型

Yii 认证框架的核心是一个名为**user**的应用组件，通常情况下，它是一个实现了`IWebUser`接口的对象。我们默认实现所使用的具体类是框架类`CWebUser`。这个用户组件封装了应用程序当前用户的所有身份信息。这个组件在我们使用`yiic`工具创建应用程序时，作为自动生成的应用程序代码的一部分为我们配置好了。配置可以在`protected/config/main.php`文件的`components`数组元素下看到：

```php
'user'=>array(
  // enable cookie-based authentication
  'allowAutoLogin'=>true,
),
```

由于它被配置为一个应用程序组件，名称为`'user'`，我们可以在整个应用程序中的任何地方使用`Yii::app()->user`来访问它。

我们还注意到类属性`allowAutoLogin`也在这里设置了。这个属性默认值为`false`，但将其设置为`true`可以使用户信息存储在持久性浏览器 cookie 中。然后这些数据将用于在后续访问时自动对用户进行身份验证。这将允许我们在登录表单上有一个**记住我**复选框，这样用户可以选择的话，在后续访问网站时可以自动登录应用程序。

Yii 认证框架定义了一个单独的实体来容纳实际的认证逻辑。这被称为**身份类**，通常可以是任何实现了`IUserIdentity`接口的类。这个类的主要作用之一是封装认证逻辑，以便轻松地允许不同的实现。根据应用程序的要求，我们可能需要验证用户名和密码与存储在数据库中的值匹配，或者允许用户使用他们的 OpenID 凭据登录，或者集成现有的 LDAP 方法。将特定于认证方法的逻辑与应用程序登录过程的其余部分分离，使我们能够轻松地在这些实现之间切换。身份类提供了这种分离。

当我们最初创建应用程序时，一个用户身份类文件，即 `protected/components/UserIdentity.php`，是为我们生成的。它扩展了 Yii 框架类 `CUserIdentity`，这是一个使用用户名和密码的身份验证实现的基类。让我们更仔细地看一下为这个类生成的代码：

```php
<?php
/**
 * UserIdentity represents the data needed to identity a user.
 * It contains the authentication method that checks if the provided
 * data can identify the user.
 */
class UserIdentity extends CUserIdentity
{
  /**
   * Authenticates a user.
   * The example implementation makes sure if the username and password
   * are both 'demo'.
   * In practical applications, this should be changed to authenticate
   * against some persistent user identity storage (e.g. database).
   * @return boolean whether authentication succeeds.
   */
  public function authenticate()
  {
    $users=array(
      // username => password
      'demo'=>'demo',
      'admin'=>'admin',
    );
    if(!isset($users[$this->username]))
      $this->errorCode=self::ERROR_USERNAME_INVALID;
    else if($users[$this->username]!==$this->password)
      $this->errorCode=self::ERROR_PASSWORD_INVALID;
    else
      $this->errorCode=self::ERROR_NONE;
    return !$this->errorCode;
  }
}
```

定义身份类的大部分工作是实现 `authenticate()` 方法。这是我们放置特定于身份验证方法的代码的地方。这个实现简单地使用硬编码的用户名/密码值 `demo/demo` 和 `admin/admin`。它检查这些值是否与用户名和密码类属性（在父类 `CUserIdentity` 中定义的属性）匹配，如果不匹配，它将设置并返回适当的错误代码。

为了更好地理解这些部分如何适应整个端到端的身份验证过程，让我们从登录表单开始逐步解释逻辑。如果我们导航到登录页面，`http://localhost/trackstar/index.php?r=site/login`，我们会看到一个简单的表单，允许输入用户名、密码，以及我们之前讨论过的**记住我下次**功能的可选复选框。提交这个表单会调用 `SiteController::actionLogin()` 方法中包含的逻辑。以下序列图描述了在成功登录时从提交表单开始发生的类交互。

![引入 Yii 身份验证模型](img/8727_06_05.jpg)

这个过程从将表单模型类 `LoginForm` 上的类属性设置为提交的表单值开始。然后调用 `LoginForm->validate()` 方法，根据 `rules()` 方法中定义的规则验证这些属性值。这个方法定义如下：

```php
public function rules()
{
  return array(
    // username and password are required
    array('username, password', 'required'),
    // rememberMe needs to be a boolean
    array('rememberMe', 'boolean'),
    // password needs to be authenticated
    array('password', 'authenticate'),
  );
}
```

最后一个规则规定，密码属性要使用自定义方法 `authenticate()` 进行验证，这个方法也在 `LoginForm` 类中定义如下：

```php
/**
   * Authenticates the password.
   * This is the 'authenticate' validator as declared in rules().
   */
  public function authenticate($attribute,$params)
  {
    $this->_identity=new UserIdentity($this->username,$this->password);
    if(!$this->_identity->authenticate())
      $this->addError('password','Incorrect username or password.');
  }
```

继续按照序列图的顺序，`LoginForm` 中的密码验证调用了同一类中的 `authenticate()` 方法。该方法创建了一个正在使用的身份验证身份类的新实例，本例中是 `/protected/components/UserIdentity.php`，然后调用它的 `authenticate()` 方法。这个方法，`UserIdentity::authenticate()` 如下：

```php
/**
   * Authenticates a user.
   * The example implementation makes sure if the username and password
   * are both 'demo'.
   * In practical applications, this should be changed to authenticate
   * against some persistent user identity storage (e.g. database).
   * @return boolean whether authentication succeeds.
   */
  public function authenticate()
  {
    $users=array(
      // username => password
      'demo'=>'demo',
      'admin'=>'admin',
    );
    if(!isset($users[$this->username]))
      $this->errorCode=self::ERROR_USERNAME_INVALID;
    else if($users[$this->username]!==$this->password)
      $this->errorCode=self::ERROR_PASSWORD_INVALID;
    else
      $this->errorCode=self::ERROR_NONE;
    return !$this->errorCode;
  }
```

这是为了使用用户名和密码进行身份验证。在这个实现中，只要用户名/密码组合是 `demo/demo` 或 `admin/admin`，这个方法就会返回 `true`。由于我们正在进行成功的登录，身份验证成功，然后 `SiteController` 调用 `LoginForm::login()` 方法，如下所示：

```php
/**
   * Logs in the user using the given username and password in the model.
   * @return boolean whether login is successful
   */
  public function login()
  {
    if($this->_identity===null)
    {
      $this->_identity=new UserIdentity($this->username,$this->password);
      $this->_identity->authenticate();
    }
    if($this->_identity->errorCode===UserIdentity::ERROR_NONE)
    {
      $duration=$this->rememberMe ? 3600*24*30 : 0; // 30 days
      Yii::app()->user->login($this->_identity,$duration);
      return true;
    }
    else
      return false;
  }
```

我们可以看到，这反过来调用了 `Yii::app()->user->login`（即 `CWebUser::login()`），传入 `CUserIdentity` 类实例以及要设置自动登录的 cookie 的持续时间。

默认情况下，Web 应用程序配置为使用 Yii 框架类 `CWebuser` 作为用户应用组件。它的 `login()` 方法接受一个身份类和一个可选的持续时间参数，用于设置浏览器 cookie 的生存时间。在前面的代码中，我们看到如果在提交表单时选中了**记住我**复选框，这个时间被设置为 `30 天`。如果你不传入一个持续时间，它会被设置为零。零值将导致根本不创建任何 cookie。

`CWebUser::login()` 方法获取身份类中包含的信息，并将其保存在持久存储中，以供用户会话期间使用。默认情况下，这个存储是 PHP 会话存储。

完成所有这些后，由我们的控制器类最初调用的`LoginForm`上的`login()`方法返回`true`，表示成功登录。然后，控制器类将重定向到`Yii::app()->user->returnUrl`中的 URL 值。如果您希望确保用户被重定向回其先前的页面，即在他们决定（或被迫）登录之前在应用程序中的任何位置，可以在应用程序的某些页面上设置此值。此值默认为应用程序入口 URL。

### 更改身份验证实现

现在我们了解了整个身份验证过程，我们可以很容易地看到我们需要在哪里进行更改，以使用我们的`tbl_user`表来验证通过登录表单提交的用户名和密码凭据。我们可以简单地修改用户身份类中的`authenticate()`方法，以验证是否存在与提供的用户名和密码值匹配的行。由于目前在我们的`UserIdentity.php`类中除了 authenticate 方法之外没有其他内容，让我们完全用以下代码替换此文件的内容：

```php
<?php

/**
 * UserIdentity represents the data needed to identity a user.
 * It contains the authentication method that checks if the provided
 * data can identity the user.
 */

class UserIdentity extends CUserIdentity
{
  private $_id;

  public function authenticate()
  {
    $user=User::model()->find('LOWER(username)=?',array(strtolower($this->username)));
    if($user===null)
      $this->errorCode=self::ERROR_USERNAME_INVALID;
    else if(!$user->validatePassword($this->password))
      $this->errorCode=self::ERROR_PASSWORD_INVALID;
    else
    {
      $this->_id=$user->id;
      $this->username=$user->username;
$this->setState('lastLogin', date("m/d/y g:i A", strtotime($user->last_login_time)));
      $user->saveAttributes(array(
        'last_login_time'=>date("Y-m-d H:i:s", time()),
      ));
      $this->errorCode=self::ERROR_NONE;
    }
    return $this->errorCode==self::ERROR_NONE;
  }

  public function getId()
  {
    return $this->_id;
  }
}
```

并且，由于我们将让我们的`User`模型类执行实际的密码验证，我们还需要向我们的`User`模型类添加以下方法：

```php
/**
   * Checks if the given password is correct.
   * @param string the password to be validated
   * @return boolean whether the password is valid
   */
  public function validatePassword($password)
  {
    return $this->hashPassword($password)===$this->password;
  }
```

这个新代码有一些需要指出的地方。首先，它现在尝试通过创建一个新的`User`模型 AR 类实例来从`tbl_user`表中检索一行，其中用户名与`UserIdentity`类的属性值相同（请记住，这是设置为登录表单的值）。由于在创建新用户时我们强制用户名的唯一性，这应该最多找到一个匹配的行。如果找不到匹配的行，将设置错误消息以指示用户名不正确。如果找到匹配的行，它通过调用我们的新`User::validatePassword()`方法来比较密码。如果密码未通过验证，将设置错误消息以指示密码不正确。

如果身份验证成功，在方法返回之前还会发生一些其他事情。首先，我们在`UserIdentity`类上设置了一个新的属性，用于用户 ID。父类中的默认实现是返回 ID 的用户名。由于我们使用数据库，并且将数字主键作为我们唯一的用户标识符，我们希望确保在请求用户 ID 时设置和返回此值。例如，当执行代码`Yii::app()->user->id`时，我们希望确保从数据库返回唯一 ID，而不是用户名。

### 扩展用户属性

这里发生的第二件事是在用户身份上设置一个属性，该属性是从数据库返回的最后登录时间，然后还更新数据库中的`last_login_time`字段为当前时间。执行此操作的特定代码如下：

```php
$this->setState('lastLogin', date("m/d/y g:i A", strtotime($user->last_login_time)));
$user->saveAttributes(array(
  'last_login_time'=>date("Y-m-d H:i:s", time()),
));
```

用户应用组件`CWebUser`从身份类中定义的显式 ID 和名称属性派生其用户属性，然后从称为`identity states`的数组中设置的`name=>value`对中派生。这些是可以在用户会话期间持久存在的额外用户值。作为这一点的例子，我们将名为`lastLogin`的属性设置为数据库中`last_login_time`字段的值。这样，在应用程序的任何地方，都可以通过以下方式访问此属性：

```php
Yii::app()->user->lastLogin;
```

我们在存储最后登录时间与 ID 时采取不同的方法的原因是*ID*恰好是`CUserIdentity`类上明确定义的属性。因此，除了*name*和*ID*之外，所有需要在会话期间持久存在的其他用户属性都可以以类似的方式设置。

### 注意

当启用基于 cookie 的身份验证（通过将`CWebUser::allowAutoLogin`设置为`true`）时，持久信息将存储在 cookie 中。因此，您*不应*以与我们存储用户最后登录时间相同的方式存储敏感信息（例如您的密码）。

有了这些更改，现在您需要为数据库中`tbl_user`表中定义的用户提供正确的用户名和密码组合。当然，使用`demo/demo`或`admin/admin`将不再起作用。试一试。您应该能够以本章早些时候创建的任何一个用户的身份登录。如果您跟着做，并且拥有与我们相同的用户数据，那么用户名：`User One`，密码：`test1`应该可以登录。

### 注意

现在我们已经修改了登录流程，以便对数据库进行身份验证，我们将无法访问项目、问题或用户实体的删除功能。原因是已经设置了授权检查，以确保用户是管理员才能访问。目前，我们的数据库用户都没有配置为授权管理员。不用担心，授权是下一章的重点，所以我们很快就能再次访问该功能。

## 在主页上显示最后登录时间

现在我们正在更新数据库中的最后登录时间，并在登录时将其保存到持久会话存储中，让我们继续在成功登录后的欢迎屏幕上显示这个时间。这也将帮助我们确信一切都按预期工作。

打开负责显示主页的默认视图文件`protected/views/site/index.php`。在欢迎语句下面添加以下突出显示的代码行：

```php
<h1>Welcome to <i><?php echo CHtml::encode(Yii::app()->name); ?></i></h1>
<?php if(!Yii::app()->user->isGuest):?>
<p>
   You last logged in on <?php echo Yii::app()->user->lastLogin; ?>.  
</p>
<?php endif;?>
```

既然我们已经在这里，让我们继续删除所有其他自动生成的帮助文本，即我们刚刚添加的代码行下面的所有内容。保存并再次登录后，您应该看到类似以下截图的内容，显示欢迎消息，然后是格式化的时间，指示您上次成功登录的时间：

在主页上显示最后登录时间

# 总结

这一章是我们专注于用户管理、身份验证和授权的两章中的第一章。我们创建了管理应用程序用户的 CRUD 操作的能力，并在此过程中对新用户创建流程进行了许多调整。我们为所有活动记录类添加了一个新的基类，以便轻松管理存在于所有表上的审计历史表列。我们还更新了代码，以正确管理我们在数据库中存储的用户最后登录时间。在这样做的过程中，我们学习了如何利用`CActiveRecord`验证工作流来允许预验证/后验证和预保存/后保存处理。

然后，我们专注于理解 Yii 身份验证模型，以便增强它以满足我们应用程序的要求，以便用户凭据被验证为存储在数据库中的值。

现在我们已经涵盖了身份验证，我们可以将重点转向 Yii 身份验证和授权框架的第二部分，*授权*。这是下一章的重点。
