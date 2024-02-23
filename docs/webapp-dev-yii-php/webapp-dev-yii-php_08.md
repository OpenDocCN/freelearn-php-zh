# 第八章：添加用户评论

通过前两章中对用户管理的实施，我们的 Trackstar 应用程序真的开始成形了。我们的主要应用程序功能的大部分功能现在已经完成。现在我们可以开始专注于一些很好有的功能。我们将首先解决的是用户在项目问题上留下评论的能力。

用户参与关于项目问题的对话的能力是任何问题跟踪工具应提供的重要部分。实现这一目标的一种方法是允许用户直接在问题上留下评论。评论将形成关于问题的对话，并提供即时和历史背景，以帮助跟踪任何问题的整个生命周期。我们还将使用评论来演示 Yii 小部件的使用以及如何建立一个小部件模型来向用户提供内容（有关小部件的更多信息，请参见[`en.wikipedia.org/wiki/Portlet`](http://en.wikipedia.org/wiki/Portlet)）。

# 功能规划

本章的目标是在 Trackstar 应用程序中实现功能，允许用户在问题上留下评论并阅读评论。当用户查看任何项目问题的详细信息时，他们应该能够阅读以前添加的所有评论，并在问题上创建新的评论。我们还希望在项目列表页面上添加一个小片段内容或小部件，以显示所有问题上最近留下的评论列表。这将是一个很好的方式，提供一个窗口进入最近的用户活动，并允许轻松访问最新的有活跃对话的问题。

以下是我们需要完成的高级任务列表：

1.  设计并创建一个新的数据库表来支持评论。

1.  创建与我们的新评论表相关的 Yii AR 类。

1.  在问题详细页面直接添加一个表单，允许用户提交评论。

1.  在问题的详细页面上显示与问题相关的所有评论列表。

1.  利用 Yii 小部件在项目列表页面上显示最近评论的列表。

# 创建模型

我们首先需要创建一个新的表来存放我们的评论。正如您所期望的那样，我们将使用数据库迁移来对我们的数据库结构进行这个添加：

```php
$ cd /Webroot/trackstar/protected
$ ./yiic migrate create create_user_comments_table
```

`up()`和`down()`方法如下：

```php
  public function up()
  {
    //create the issue table
    $this->createTable('tbl_comment', array(
      'id' => 'pk',
          'content' => 'text NOT NULL',
          'issue_id' => 'int(11) NOT NULL',
      'create_time' => 'datetime DEFAULT NULL',
      'create_user_id' => 'int(11) DEFAULT NULL',
      'update_time' => 'datetime DEFAULT NULL',
      'update_user_id' => 'int(11) DEFAULT NULL',
     ), 'ENGINE=InnoDB');

    //the tbl_comment.issue_id is a reference to tbl_issue.id 
    $this->addForeignKey("fk_comment_issue", "tbl_comment", "issue_id", "tbl_issue", "id", "CASCADE", "RESTRICT");

    //the tbl_issue.create_user_id is a reference to tbl_user.id 
    $this->addForeignKey("fk_comment_owner", "tbl_comment", "create_user_id", "tbl_user", "id", "RESTRICT, "RESTRICT");

    //the tbl_issue.updated_user_id is a reference to tbl_user.id 
    $this->addForeignKey("fk_comment_update_user", "tbl_comment", "update_user_id", "tbl_user", "id", "RESTRICT", "RESTRICT");

  }

  public function down()
  {
    $this->dropForeignKey('fk_comment_issue', 'tbl_comment');
    $this->dropForeignKey('fk_comment_owner', 'tbl_comment');
    $this->dropForeignKey('fk_comment_update_user', 'tbl_comment');
    $this->dropTable('tbl_comment');
  }
```

为了实现这个数据库更改，我们需要运行迁移：

```php
$ ./yiic migrate
```

现在我们的数据库表已经就位，创建相关的 AR 类就很容易了。我们在前几章中已经看到了很多次。我们知道如何做。我们只需使用 Gii 代码创建工具的**Model Generator**命令，并根据我们新创建的`tbl_comment`表创建一个名为`Comment`的 AR 类。如果需要，可以参考第四章*项目 CRUD*和第五章*管理问题*，了解使用此工具创建模型类的所有细节。

使用 Gii 工具为评论创建模型类后，您会注意到为我们生成的代码已经定义了一些关系。这些关系是基于我们在`tbl_comments`表上定义的外键关系。以下是为我们创建的内容：

```php
/**
   * @return array relational rules.
   */
  public function relations()
  {
    // NOTE: you may need to adjust the relation name and the related
    // class name for the relations automatically generated below.
    return array(
      'updateUser' => array(self::BELONGS_TO, 'User', 'update_user_id'),
      'issue' => array(self::BELONGS_TO, 'Issue', 'issue_id'),
      'createUser' => array(self::BELONGS_TO, 'User', 'create_user_id'),
    );
  }
```

我们可以看到我们有一个关系，指定评论属于一个问题。但我们还需要定义一个问题和它的评论之间的一对多关系。一个问题可以有多个评论。这个更改需要在`Issue`模型类中进行。

### 注意

如果我们在创建 Issue 模型的同时创建了我们的评论模型，这个关系就会为我们创建。

除此之外，我们还将添加一个关系作为统计查询，以便轻松检索与给定问题相关的评论数量。以下是我们对`Issue::relations()`方法所做的更改：

```php
public function relations()
{
  return array(
    'requester' => array(self::BELONGS_TO, 'User', 'requester_id'),
    'owner' => array(self::BELONGS_TO, 'User', 'owner_id'),
    'project' => array(self::BELONGS_TO, 'Project', 'project_id'),
    'comments' => array(self::HAS_MANY, 'Comment', 'issue_id'),
    'commentCount' => array(self::STAT, 'Comment', 'issue_id'),
  );
}
```

这建立了问题和评论之间的一对多关系。它还定义了一个统计查询，允许我们轻松地检索任何给定问题实例的评论总数。

### 提示

统计查询

之前定义的`commentCount`关系是我们以前没有见过的一种新类型的关系。除了关联查询，Yii 还提供了所谓的统计或聚合关系。在对象之间存在一对多（`HAS_MANY`）或多对多（`MANY_MANY`）关系的情况下，这些关系非常有用。在这种情况下，我们可以定义统计关系，以便轻松地获取相关对象的总数。我们已经利用了这一点，在之前的关系声明中，以便轻松地检索任何给定问题实例的评论总数。有关在 Yii 中使用统计查询的更多信息，请参阅[`www.yiiframework.com/doc/guide/1.1/en/database.arr#statistical-query`](http://www.yiiframework.com/doc/guide/1.1/en/database.arr#statistical-query)。

我们还需要更改我们新创建的`Comment` AR 类，以扩展我们自定义的`TrackStarActiveRecord`基类，以便它从我们放置在`beforeSave()`方法中的逻辑中受益。只需修改类定义的开头，如下所示：

```php
<?php
      /**
 * This is the model class for table "tbl_comment".
 */
class Comment extends TrackStarActiveRecord
{
```

我们将对`Comment::relations()`方法中的定义进行最后一次小的更改。在创建类时，关系属性已经为我们命名。让我们将名为`createUser`的属性更改为`author`，因为这个相关的用户代表评论的作者。这只是一个语义上的改变，但它将有助于使我们的代码更易于阅读和理解。将定义从`'createUser' => array(self::BELONGS_TO, 'User', 'create_user_id'),`更改为`'author' => array(self::BELONGS_TO, 'User', 'create_user_id')`。

# 创建评论 CRUD

现在我们已经有了 AR 模型类，创建用于管理相关实体的 CRUD 脚手架很容易。只需使用 Gii 代码生成工具的 Crud 生成器命令，参数为 AR 类名`Comment`。我们在之前的章节中已经看到了这个很多次，所以我们不会在这里再详细介绍。如果需要，可以参考第四章，*项目 CRUD*和第五章，*管理问题*，了解使用 Gii 工具创建 CRUD 脚手架代码的所有细节。虽然我们不会立即为我们的评论实现完整的 CRUD 操作，但是有其他操作的脚手架是很好的。

在使用 Gii 的 Crud 生成器之后，只要我们登录，现在我们应该能够通过以下 URL 查看自动生成的评论提交表单：

`http://localhost/trackstar/index.php?r=comment/create`

# 修改脚手架以满足我们的要求

正如我们以前经常看到的那样，我们经常需要调整自动生成的脚手架代码，以满足应用程序的特定要求。首先，我们用于创建新评论的自动生成表单为`tbl_comment`数据库表中定义的每个列都有一个输入字段。实际上，我们并不希望所有这些字段都成为表单的一部分。事实上，我们希望大大简化这个表单，只有一个用于评论内容的输入字段。而且，我们不希望用户通过之前提到的 URL 访问表单，而是只能通过访问问题详情页面来添加评论。用户将在查看问题详情的页面上添加评论。我们希望朝着以下截图所示的方式构建：

![修改脚手架以满足我们的要求](img/8727_08_01.jpg)

为了实现这一点，我们将修改我们的`Issue`控制器类，以处理评论表单的提交，并修改问题详细信息视图，以显示现有评论和新评论创建表单。此外，由于评论应该只在问题的上下文中创建，我们将在问题模型类中添加一个新方法来创建新评论。

## 添加评论

正如前面提到的，我们将让问题实例创建自己的评论。为此，我们希望在`Issue` AR 类中添加一个方法。以下是该方法：

```php
/**
  * Adds a comment to this issue
  */
public function addComment($comment)
{
  $comment->issue_id=$this->id;
  return $comment->save();
}
```

该方法确保在保存新评论之前正确设置评论问题 ID。也就是说，当`Issue`的实例创建新评论时，我们知道该评论属于该问题。

有了这个方法，我们现在可以把重点转向问题控制器类。由于我们希望评论创建表单从`IssueController::actionView()`方法显示并将其数据发送回来，我们需要修改该方法。我们还将添加一个新的受保护方法来处理表单提交请求。首先，修改`actionView()`方法如下：

```php
public function actionView($id)
{
    $issue=$this->loadModel($id);
    $comment=$this->createComment($issue);
    $this->render('view',array(
      'model'=>$issue,
         'comment'=>$comment,
    ));
}
```

然后，添加以下受保护方法来创建一个新评论并处理创建此问题的新评论的表单提交请求：

```php
/**
  * Creates a new comment on an issue
  */
protected function createComment($issue)
{
  $comment=new Comment;  
  if(isset($_POST['Comment']))
  {
    $comment->attributes=$_POST['Comment'];
    if($issue->addComment($comment))
    {
      Yii::app()->user->setFlash('commentSubmitted',"Your comment has been added." );
      $this->refresh();
    }
  }
  return $comment;
}
```

我们的新受保护方法`createComment()`负责处理用户在问题上留下新评论时提交的`POST`请求。如果成功创建评论，我们设置一个闪存消息显示给用户，并进行页面刷新，以便我们的新评论将显示。当然，我们仍然需要修改我们的视图文件，以便所有这些显示给用户。对`IssueController::actionView()`所做的更改负责调用这个新方法，并为显示提供新评论实例。

## 显示表单

现在，我们需要修改我们的视图。首先，我们将创建一个新的视图文件来呈现我们的评论显示和评论输入表单。我们打算在另一个视图文件中显示此视图文件。因此，我们不希望再次显示所有一般页面组件，例如页眉导航和页脚信息。打算在其他视图文件中显示或不带任何额外装饰的视图文件称为**partial**视图。然后，您可以使用控制器方法`renderPartial()`，而不是`render()`方法。使用`renderPartial()`将仅呈现该视图文件中包含的内容，并且不会用任何其他内容装饰显示。当我们讨论使用布局和装饰视图文件时，我们将在第十章*让它看起来不错*中详细讨论这一点。

Yii 在创建部分视图文件时使用下划线（`_`）作为命名约定的前缀。由于我们将其呈现为部分视图，我们将遵循命名约定，并以下划线开头命名文件。在`protected/views/issue/`目录下创建一个名为`_comments.php`的新文件，并将以下代码添加到该文件中：

```php
<?php foreach($comments as $comment): ?>
<div class="comment">
      <div class="author">
    <?php echo CHtml::encode($comment->author->username); ?>:
  </div>

  <div class="time">
    on <?php echo date('F j, Y \a\t h:i a',strtotime($comment->create_time)); ?>
  </div>

  <div class="content">
    <?php echo nl2br(CHtml::encode($comment->content)); ?>
  </div>
     <hr>
</div><!-- comment -->
<?php endforeach; ?>
```

该文件接受评论实例数组作为输入参数，并逐个显示它们。现在，我们需要修改问题详细信息的视图文件以使用这个新文件。我们通过打开`protected/views/issue/view.php`并在文件末尾添加以下内容来实现这一点：

```php
<div id="comments">
  <?php if($model->commentCount>=1): ?>
    <h3>
      <?php echo $model->commentCount>1 ? $model->commentCount . ' comments' : 'One comment'; ?>
    </h3>

    <?php $this->renderPartial('_comments',array(
      'comments'=>$model->comments,
    )); ?>
  <?php endif; ?>

  <h3>Leave a Comment</h3>

  <?php if(Yii::app()->user->hasFlash('commentSubmitted')): ?>
    <div class="flash-success">
      <?php echo Yii::app()->user->getFlash('commentSubmitted'); ?>
    </div>
  <?php else: ?>
    <?php $this->renderPartial('/comment/_form',array(
      'model'=>$comment,
    )); ?>
  <?php endif; ?>

</div>
```

在这里，我们利用了我们之前添加到`Issue` AR 模型类的统计查询属性`commentCount`。这使我们能够快速确定特定问题是否有任何可用的评论。如果有评论，它将继续使用我们的`_comments.php`显示视图文件来呈现它们。然后显示我们在使用 Gii Crud Generator 功能时为我们创建的输入表单。它还会显示成功保存评论时设置的简单闪存消息。

我们需要做的最后一个改变是评论输入表单本身。正如我们过去多次看到的那样，为我们创建的表单在底层`tbl_comment`表中定义了每一列的输入字段。这不是我们想要显示给用户的。我们希望将其变成一个简单的输入表单，用户只需要提交评论内容。因此，打开包含输入表单的视图文件，即`protected/views/comment/_form.php`，并编辑如下：

```php
<div class="form">
<?php $form=$this->beginWidget('CActiveForm', array(
  'id'=>'comment-form',
  'enableAjaxValidation'=>false,
)); ?>
       <p class="note">Fields with <span class="required">*</span> are required.</p>
       <?php echo $form->errorSummary($model); ?>
       <div class="row">
    <?php echo $form->labelEx($model,'content'); ?>
    <?php echo $form->textArea($model,'content',array('rows'=>6, 'cols'=>50)); ?>
    <?php echo $form->error($model,'content'); ?>
  </div>

  <div class="row buttons">
    <?php echo CHtml::submitButton($model->isNewRecord ? 'Create' : 'Save'); ?>
  </div>

<?php $this->endWidget(); ?>

</div>
```

有了这一切，我们可以访问问题列表页面查看评论表单。例如，如果我们访问`http://localhost/trackstar/index.php?r=issue/view&id=111`，我们将在页面底部看到以下评论输入表单：

![显示表单](img/8727_08_02.jpg)

如果我们尝试提交评论而没有指定任何内容，我们将看到以下截图中所示的错误：

![显示表单](img/8727_08_03.jpg)

然后，如果我们以`User One`的身份登录并提交评论`My first test comment`，我们将看到以下显示：

![显示表单](img/8727_08_04.jpg)

# 创建一个最近评论的小部件

现在我们可以在问题上留下评论，我们将把重点转向本章的第二个目标。我们想要显示所有项目中留下的最近评论列表。这将提供应用程序中用户沟通活动的一个很好的快照。我们还希望以一种方式构建这个小的内容块，使它可以在站点的不同位置轻松重复使用。这在互联网上的许多网络门户应用程序中非常常见。这些小的内容片段通常被称为**portlet**，这也是为什么我们在本章开头提到构建 portlet 架构。您可以参考[`en.wikipedia.org/wiki/Portlet`](http://en.wikipedia.org/wiki/Portlet)了解更多关于这个主题的信息。

## 介绍 CWidget

幸运的是，Yii 已经准备好帮助我们实现这种架构。Yii 提供了一个名为`CWidget`的组件类，非常适合实现这种类型的架构。Yii 的**widget**是`CWidget`类的一个实例（或其子类），通常嵌入在视图文件中以显示自包含、可重用的用户界面功能。我们将使用 Yii 的 widget 来构建一个最近评论组件，并在主项目详情页面上显示它，以便我们可以看到与项目相关的所有问题的评论活动。为了演示重用的便利性，我们将进一步显示一个最近评论列表，跨所有项目在项目列表页面上。

### 命名作用域

要开始创建我们的小部件，我们首先要修改我们的`Comment` AR 模型类，以返回最近添加的评论。为此，我们将利用 Yii 的 AR 模型类中的另一个特性——命名作用域。

**命名作用域**允许我们指定一个命名查询，提供了一种优雅的方式来定义检索 AR 对象列表时的 SQL `where`条件。命名作用域通常在`CActiveRecord::scopes()`方法中定义为`name=>criteria`对。例如，如果我们想定义一个名为`recent`的命名作用域，它将返回最近的五条评论；我们可以创建`Comment::scopes()`方法如下：

```php
class Comment extends TrackStarActiveRecord
{
  ...
  public function scopes()
  {
    return array(
      'recent'=>array(
        'order'=>'create_time DESC',
        'limit'=>5,
      ),
    );
  }
...
}
```

现在，我们可以使用以下语法轻松检索最近评论的列表：

```php
$comments=Comment::model()->recent()->findAll();
```

您还可以链接命名作用域。如果我们定义了另一个命名作用域，例如`approved`（如果我们的应用程序在显示评论之前需要经过批准过程），我们可以获取最近批准的评论列表，如下所示：

```php
$comments=Comment::model()->recent()->approved()->findAll();
```

您可以看到通过将它们链接在一起，我们有一种灵活而强大的方式来在特定上下文中检索我们的对象。

命名范围必须出现在`find`调用的左侧（`find`，`findAll`，`findByPk`等），并且只能在类级上下文中使用。命名范围方法调用必须与`ClassName::model()`一起使用。有关命名范围的更多信息，请参见[`www.yiiframework.com/doc/guide/1.1/en/database.ar#named-scopes`](http://www.yiiframework.com/doc/guide/1.1/en/database.ar#named-scopes)。

命名范围也可以被参数化。在先前的评论`recent`命名范围中，我们在条件中硬编码了限制为`5`。然而，当我们调用该方法时，我们可能希望能够指定限制数量。这就是我们为评论设置命名范围的方式。要添加参数，我们以稍有不同的方式指定命名范围。我们不是使用`scopes()`方法来声明我们的范围，而是定义一个新的公共方法，其名称与范围名称相同。将以下方法添加到`Comment` AR 类中：

```php
public function recent($limit=5)
{
  $this->getDbCriteria()->mergeWith(
    array(         
    'order'=>'t.create_time DESC',         
      'limit'=>$limit,     
    )
  );     
  return $this;
}
```

关于这个查询条件的一件事是在 order 值中使用了`t`。这是为了帮助在与另一个具有相同列名的相关表一起使用时。显然，当两个被连接的表具有相同的列名时，我们必须在查询中区分这两个表。例如，如果我们在相同的查询中使用这个查询来检索`Issue` AR 信息，`tbl_issue`和`tbl_comment`表都有定义`create_time`列。我们试图按照`tbl_comment`表中的这一列进行排序，而不是在问题表中定义的那一列。在 Yii 的关系 AR 查询中，主表的别名固定为`t`，而关系表的别名默认情况下与相应的关系名称相同。因此，在这种情况下，我们指定`t.create_time`以指示我们要使用主表的列。

### Yii 中关于关系 AR 查询的更多信息

有了这种方法，我们可以将命名范围与急切加载方法结合起来，以检索相关的`Issue` AR 实例。例如，假设我们想要获取与 ID 为`1`的项目相关的最后十条评论，我们可以使用以下方法：

```php
$comments = Comment::model()->with(array('issue'=>array('condition'=>'project_id=1')))->recent(10)->findAll();
```

这个查询对我们来说是新的。在以前的查询中，我们没有使用许多这些选项。以前，我们使用不同的方法来执行关系查询：

+   加载 AR 实例

+   在`relations()`方法中定义的关系属性中访问

例如，如果我们想要查询与项目 ID＃1 关联的所有问题，我们将使用类似以下两行代码的内容：

```php
// First retrieve the project whose ID is 1
$project=Project::model()->findByPk(1);

// Then retrieve the project's issues (a relational query is actually being performed behind the scenes here)
$issues=$project->issues;
```

这种熟悉的方法使用了所谓的**懒加载**。当我们首次创建项目实例时，查询不会返回所有相关的问题。它只在以后明确请求它们时检索相关的问题，也就是当执行`$project->issues`时。这被称为“懒惰”，因为它等到以后请求时才加载问题。

这种方法非常方便，而且在那些不需要相关问题的情况下也可以非常高效。然而，在其他情况下，这种方法可能有些低效。例如，如果我们想要检索跨*N*项目的问题信息，那么使用这种懒惰的方法将涉及执行*N*个连接查询。根据*N*的大小，这可能非常低效。在这些情况下，我们有另一个选择。我们可以使用所谓的**急切加载**。

急切加载方法在请求主 AR 实例的同时检索相关的 AR 实例。这是通过在 AR 查询的`find()`或`findAll()`方法与`with()`方法一起使用来实现的。继续使用我们的项目示例，我们可以使用急切加载来检索所有项目的所有问题，只需执行以下一行代码：

```php
//retrieve all project AR instances along with their associated issue AR instances
$projects = Project::model()->with('issues')->findAll();
```

现在，在这种情况下，`$projects`数组中的每个项目 AR 实例已经具有其关联的`issues`属性，该属性填充有`Issue` AR 实例的数组。这是通过使用单个连接查询实现的。

因此，让我们回顾一下我们检索特定项目的最后十条评论的示例：

```php
$comments = Comment::model()->with(array('issue'=>array('condition'=>'project_id=1')))->recent(10)->findAll();
```

我们正在使用急切加载方法来检索问题以及评论，但这个方法稍微复杂一些。这个查询在`tbl_comment`和`tbl_issue`表之间指定了一个连接。这个关系 AR 查询基本上会执行类似于以下 SQL 语句的操作：

```php
SELECT tbl_comment.*, tbl_issue.* FROM tbl_comment LEFT OUTER JOIN tbl_issue ON (tbl_comment.issue_id=tbl_issue.id) WHERE (tbl_issue.project_id=1) ORDER BY tbl_comment.create_time DESC LIMIT 10;
```

掌握了 Yii 中延迟加载和急切加载的好处的知识后，我们应该调整`IssueController::actionView()`方法中加载问题模型的方式。由于我们已经修改了问题的详细视图以显示我们的评论，包括评论的作者，我们知道在调用`IssueController::loadModel()`时，使用急切加载方法加载评论以及它们各自的作者将更有效。为此，我们可以添加一个额外的参数作为简单的输入标志，以指示我们是否要加载评论。

修改`IssueController::loadModel()`方法如下：

```php
   public function loadModel($id, $withComments=false)
  {
    if($withComments)
      $model = Issue::model()->with(array('comments'=>array('with'=>'author')))->findByPk($id);
    else
      $model=Issue::model()->findByPk($id);
    if($model===null)
      throw new CHttpException(404,'The requested page does not exist.');
    return $model;
  }
```

在`IssueController`方法中有三个地方调用了`loadModel()`方法：`actionView`，`actionUpdate`和`actionDelete`。当我们查看问题详情时，我们只需要关联的评论。因此，我们已经将默认设置为不检索关联的评论。我们只需要修改`actionView()`方法，在`loadModel()`调用中添加`true`。

```php
public function actionView($id)
{
  $issue=$this->loadModel($id, true);
....
}
```

有了这个设置，我们将加载问题以及其所有关联的评论，并且对于每条评论，我们将加载关联的作者信息，只需一次数据库调用。

### 创建小部件

现在，我们已经准备好创建我们的新小部件，以利用之前提到的所有更改来显示我们的最新评论。

正如我们之前提到的，Yii 中的小部件是从框架类`CWidget`或其子类扩展的类。我们将把我们的新小部件添加到`protected/components/`目录中，因为该目录的内容已经在主配置文件中指定为在应用程序中自动加载。这样，我们就不必在每次使用时显式导入该类。我们将称我们的小部件为`RecentComments`，并在该目录中添加一个同名的`.php`文件。将以下类定义添加到这个新创建的`RecentComments.php`文件中：

```php
<?php
/**
     * RecentCommentsWidget is a Yii widget used to display a list of recent comments 
     */
class RecentCommentsWidget extends CWidget
{
    private $_comments;  
    public $displayLimit = 5;
    public $projectId = null;

    public function init()
        {
          if(null !== $this->projectId)
        $this->_comments = Comment::model()->with(array('issue'=>array('condition'=>'project_id='.$this->projectId)))->recent($this->displayLimit)->findAll();
      else
        $this->_comments = Comment::model()->recent($this->displayLimit)->findAll();
        }  

        public function getData()
        {
          return $this->_comments;
        }

        public function run()
        {
            // this method is called by CController::endWidget()    
            $this->render('recentCommentsWidget');
        }
}
```

创建新小部件时的主要工作是重写基类的`init()`和`run()`方法。`init()`方法初始化小部件，并在其属性被初始化后调用。`run()`方法执行小部件。在这种情况下，我们只需通过请求基于`$displayLimit`和`$projectId`属性的最新评论来初始化小部件，使用我们之前讨论过的查询。小部件本身的执行只是简单地呈现其关联的视图文件，我们还没有创建。按照惯例，小部件的视图文件放在与小部件相同的目录中的`views/`目录中，并且与小部件同名，但以小写字母开头。遵循这个惯例，创建一个新文件，其完全限定的路径是`protected/components/views/recentCommentsWidget.php`。创建后，在该文件中添加以下内容：

```php
<ul>
  <?php foreach($this->getData() as $comment): ?>  
    <div class="author">
      <?php echo $comment->author->username; ?> added a comment.
    </div>
    <div class="issue">      
       <?php echo CHtml::link(CHtml::encode($comment->issue->name), array('issue/view', 'id'=>$comment->issue->id)); ?>
      </div>

  <?php endforeach; ?>
</ul>
```

这调用了`RecentCommentsWidget::getData()`方法，该方法返回一个评论数组。然后遍历每个评论，显示添加评论的人以及留下评论的相关问题。

为了看到结果，我们需要将这个小部件嵌入到现有的控制器视图文件中。如前所述，我们希望在项目列表页面上使用这个小部件，以显示所有项目的最近评论，并且在特定项目详情页面上，只显示该特定项目的最近评论。

让我们从项目列表页面开始。负责显示该内容的视图文件是`protected/views/project/index.php`。打开该文件，并在底部添加以下内容：

```php
<?php $this->widget('RecentCommentsWidget'); ?>  
```

如果我们现在查看项目列表页面`http://localhost/trackstar/index.php?r=project`，我们会看到类似以下截图的内容：

![创建小部件](img/8727_08_05.jpg)

现在，我们通过调用小部件将我们的新最近评论数据嵌入到页面中。这很好，但我们可以进一步将我们的小部件显示为应用程序中所有其他潜在*小部件*的一致方式。我们可以利用 Yii 为我们提供的另一个类`CPortlet`来实现这一点。

### 介绍 CPortlet

`CPortlet`是 Zii 的一部分，它是 Yii 捆绑的官方扩展类库。它为所有小部件提供了一个不错的基类。它将允许我们渲染一个漂亮的标题以及一致的 HTML 标记，这样应用程序中的所有小部件都可以很容易地以类似的方式进行样式设置。一旦我们有一个渲染内容的小部件，比如我们的`RecentCommentsWidget`，我们可以简单地使用我们小部件的渲染内容作为`CPortlet`的内容，`CPortlet`本身也是一个小部件，因为它也是从`CWidget`继承而来。我们可以通过在`CPortlet`的`beginWidget()`和`endWiget()`调用之间放置我们对`RecentComments`小部件的调用来实现这一点，如下所示：

```php
<?php $this->beginWidget('zii.widgets.CPortlet', array(
  'title'=>'Recent Comments',
));  

$this->widget('RecentCommentsWidget');

$this->endWidget(); ?>
```

由于`CPortlet`提供了一个标题属性，我们将其设置为对我们的 portlet 有意义的内容。然后，我们使用`RecentComments`小部件的渲染内容来为 portlet 小部件提供内容。这样做的最终结果如下截图所示：

![介绍 CPortlet](img/8727_08_06.jpg)

这与我们之前的情况并没有太大的变化，但现在我们已经将我们的内容放入了一个一致的容器中，这个容器已经在整个网站中使用。请注意右侧列菜单内容块和我们新创建的最近评论内容块之间的相似之处。我相信你不会感到意外，右侧列菜单块也是在`CPortlet`容器中显示的。查看`protected/views/layouts/column2.php`，这是一个在我们最初创建应用程序时由`yiic webapp`命令自动生成的文件，会发现以下代码：

```php
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
```

因此，看来应用程序一直在利用小部件！

#### 将我们的小部件添加到另一个页面

让我们还将我们的小部件添加到项目详情页面，并将评论限制为与特定项目相关的评论。

在`protected/views/project/view.php`文件的末尾添加以下内容：

```php
<?php $this->beginWidget('zii.widgets.CPortlet', array(
  'title'=>'Recent Comments On This Project',
));  

$this->widget('RecentCommentsWidget', array('projectId'=>$model->id));

$this->endWidget(); ?>
```

这基本上与我们添加到项目列表页面的内容相同，只是我们通过向调用添加一个`name=>value`对的数组来初始化小部件的`$projectId`属性。

如果现在访问特定项目详情页面，我们应该会看到类似以下截图的内容：

![将我们的小部件添加到另一个页面](img/8727_08_07.jpg)

上述截图显示了**项目#1**的详情页面，该项目有一个关联的问题，该问题只有一个评论，如截图所示。您可能需要添加一些问题和这些问题的评论，以生成类似的显示。现在我们有一种方法可以在整个网站的任何地方以一致且易于维护的方式显示最近的评论。

# 总结

通过本章，我们已经开始为我们的 Trackstar 应用程序添加功能，这些功能已经成为当今大多数基于用户的 Web 应用程序所期望的。用户在应用程序内部相互通信的能力是成功的问题管理系统的重要组成部分。

当我们创建了这一重要功能时，我们能够更深入地了解如何编写关系 AR 查询。我们还介绍了称为小部件和门户网站的内容组件。这使我们能够开发小的内容块，并能够在应用程序的任何地方使用它们。这种方法极大地增加了重用性、一致性和易于维护性。

在下一章中，我们将在这里创建的最近评论小部件的基础上构建，并将我们小部件生成的内容作为 RSS 订阅公开，以便用户可以跟踪应用程序或项目的活动，而无需访问应用程序。
