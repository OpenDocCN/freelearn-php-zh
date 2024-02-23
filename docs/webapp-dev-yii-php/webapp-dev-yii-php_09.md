# 第九章：添加 RSS 网络订阅

在上一章中，我们添加了用户在问题上留下评论的功能，并显示这些评论的列表，利用小部件架构使我们能够在整个应用程序中轻松和一致地显示该列表。在本章中，我们将建立在此功能的基础上，并将这些评论列表公开为 RSS 数据订阅。此外，我们将使用另一个开源框架 Zend 框架中现有的订阅功能，以演示 Yii 应用程序如何轻松地与其他框架和库集成。

# 功能规划

本章的目标是使用从用户生成的评论创建 RSS 订阅。我们应该允许用户订阅跨所有项目的评论订阅，以及订阅单个项目的订阅。幸运的是，我们之前构建的小部件功能已经具有返回所有项目的最新评论列表以及限制数据到一个特定项目的能力。因此，我们已经编写了访问必要数据的适当方法。本章的大部分内容将集中在将这些数据放入正确的格式以发布为 RSS 订阅，并在我们的应用程序中添加链接以允许用户订阅这些订阅。

以下是我们将完成的一系列高级任务列表，以实现这些目标：

+   下载并安装 Zend 框架到 Yii 应用程序中

+   在控制器类中创建一个新的操作，以响应订阅请求并以 RSS 格式返回适当的数据

+   更改我们的 URL 结构以便使用

+   将我们新创建的订阅添加到项目列表页面以及每个单独项目的详细页面

# 一点背景-内容联合，RSS 和 Zend 框架

网络内容联合已经存在多年，但在过去几年才获得了巨大的流行。网络内容联合是指以标准化格式发布信息，以便其他网站可以轻松使用，并且可以轻松被阅读应用程序消费。许多新闻网站长期以来一直在电子联合他们的内容，但互联网上博客的大规模爆炸已经将内容联合（称为订阅）变成了几乎每个网站都期望的功能。我们的 TrackStar 应用程序也不例外。

**真正简单的联合**（**RSS**）是一种 XML 格式规范，为网络内容联合提供了一个标准。还有其他可以使用的格式，但由于 RSS 在大多数网站中的压倒性流行，我们将专注于以这种格式提供我们的订阅。

Zend 被称为“PHP 公司”。他们提供的产品之一是 Zend 框架，用于帮助应用程序开发。该框架提供了可以并入其他框架应用程序的组件。Yii 足够灵活，可以让我们使用其他框架的部分。我们将只使用 Zend 框架库的一个组件，称为`Zend_Feed`，这样我们就不必编写所有底层的“管道”代码来生成我们的 RSS 格式的网络订阅。有关 Zend_Feed 的更多信息，请访问[`www.zendframework.com/manual/en/zend.feed.html`](http://www.zendframework.com/manual/en/zend.feed.html)。

# 安装 Zend 框架

由于我们使用 Zend 框架来帮助支持我们的 RSS 需求，因此我们首先需要下载并安装该框架。要下载框架文件，请访问[`www.zend.com/community/downloads`](http://www.zend.com/community/downloads)。由于我们只会使用该框架的一个组件，因此最小版本的框架就足够了。我们使用的是 1.1.12 版本。

当您扩展下载的框架文件时，您应该看到以下高级目录和文件结构：

+   `INSTALL.txt`

+   `LICENSE.txt`

+   `README.txt`

+   `bin/`

+   `library/`

为了在我们的 Yii 应用程序中使用这个框架，我们需要移动应用程序目录结构中的一些文件。让我们在应用程序的`/protected`目录下创建一个名为`vendors/`的新目录。然后，将 Zend Framework 目录`/library/Zend`移动到这个新创建的目录下。一切就位后，确保`protected/vendors/Zend/Feed.php`存在于 TrackStar 应用程序中。

# 使用 Zend_Feed

**Zend_Feed**是 Zend Framework 的一个小组件，它封装了创建 Web 源的所有复杂性，提供了一个简单易用的接口。它将帮助我们在很短的时间内建立一个工作的、经过测试的、符合 RSS 标准的数据源。我们所需要做的就是按照 Zend_Feed 期望的格式对我们的评论数据进行格式化，它会完成其余的工作。

我们需要一个地方来存放处理我们的数据源请求的代码。我们可以为此创建一个新的控制器，但为了保持简单，我们将只是在我们的主`CommentController.php`文件中添加一个新的操作方法来处理请求。我们将整个方法列在这里，然后逐步讨论它的功能。

```php
Open up CommentController.php and add the following public method:
/**
   * Uses Zend Feed to return an RSS formatted comments data feed
   */
  public function actionFeed()
  {
    if(isset($_GET['pid'])) 
    {
      $comments = Comment::model()->with(array(
                'issue'=>array(
                  'condition'=>'project_id=:projectId', 
                  'params'=>array(':projectId'=>intval($_GET['pid'])),
                )))->recent(20)->findAll();      
    }
    else   
      $comments = Comment::model()->recent(20)->findAll();  

    //convert from an array of comment AR class instances to an name=>value array for Zend
    $entries=array(); 

    foreach($comments as $comment)
    {

        $entries[]=array(
                'title'=>$comment->issue->name,     
                'link'=>CHtml::encode($this->createAbsoluteUrl('issue/view',array('id'=>$comment->issue->id))),  
                'description'=> $comment->author->username . ' says:<br>' . $comment->content,
                'lastUpdate'=>strtotime($comment->create_time),   
                'author'=>CHtml::encode($comment->author->username),
         );
    }  

    //now use the Zend Feed class to generate the Feed
    // generate and render RSS feed
    $feed=Zend_Feed::importArray(array(
         'title'   => 'Trackstar Project Comments Feed',
         'link'    => $this->createAbsoluteUrl(''),
         'charset' => 'UTF-8',
         'entries' => $entries,      
     ), 'rss');

    $feed->send();

  }
```

这一切都相当简单。首先，我们检查输入请求查询字符串是否存在`pid`参数，这表明特定项目 ID。请记住，我们希望可选地允许数据源将内容限制为与单个项目相关的评论。接下来，我们使用与上一章中用于填充小部件的相同方法来检索最多 20 条最近的评论列表，可以是跨所有项目，或者如果指定了项目 ID，则特定于该项目。

您可能还记得，这个方法返回一个`Comment` AR 类实例的数组。我们遍历这个返回的数组，并将数据转换为`Zend_Feed`组件接受的格式。`Zend_Feed`接受一个简单的数组，其中包含元素本身是包含每个评论条目数据的数组。每个单独的条目都是一个简单的`name=>value`对的关联数组。为了符合特定的 RSS 格式，我们的每个单独的条目必须至少包含一个标题、一个链接和一个描述。我们还添加了两个可选字段，一个称为`lastUpdate`，`Zend_Feed`将其转换为 RSS 字段`pubDate`，另一个用于指定作者。

我们利用了一些额外的辅助方法，以便以正确的格式获取数据。首先，我们使用控制器的`createAbsoluteUrl()`方法，而不仅仅是`createUrl()`方法，以生成一个完全合格的 URL。使用`createAbsoluteUrl()`将生成类似于以下的链接：

`http://localhost/trackstar/index.php?r=issue/view&id=5`而不仅仅是`/index.php?r=issue/view&id=5`

此外，为了避免由 PHP 的`DOMDocument::createElement()`方法生成的`unterminated entity reference`等错误，该方法被`Zend_Feed`用于生成 RSS XML，我们需要使用我们方便的辅助函数`CHtml::encode`将所有适用的字符转换为 HTML 实体。因此，我们对链接进行编码，以便像`http://localhost/trackstar/index.php?r=issue/view&id=5`这样的 URL 将被转换为`http://localhost/trackstar/index.php?r=issue/view&amp;id=5`。

我们还需要对将以 RSS 格式呈现的其他数据执行此操作。描述和标题字段都生成为`CDATA`块，因此在这些字段上不需要使用编码。

一旦所有条目都被正确填充和格式化，我们就使用 Zend_Feed 的`importArray()`方法，该方法接受一个数组来构造 RSS 源。最后，一旦从输入条目数组构建了源类并返回，我们就调用该类的`send()`方法。这将返回适当格式的 RSS XML 和适当的标头给客户端。

我们需要在`CommentController.php`文件和类中进行一些配置更改，然后才能使其正常工作。我们需要在评论控制器中包含一些 Zend 框架文件。在`CommentController.php`的顶部添加以下语句：

```php
Yii::import('application.vendors.*');
require_once('Zend/Feed.php');
require_once('Zend/Feed/Rss.php');
```

最后，修改`CommentController::accessRules()`方法，允许任何用户访问我们新添加的`actionFeed()`方法：

```php
public function accessRules()
  {
    return array(
      array('allow',  // allow all users to perform 'index' and 'view' actions
 **'actions'=>array('index','view','feed'),**
        'users'=>array('*'),
      ),
```

事实上就是这样。如果我们现在导航到`http://localhost/trackstar/index.php?r=comment/feed`，我们就可以查看我们的努力成果。由于浏览器对 RSS feed 的显示方式不同，您的体验可能与下面的截图有所不同。如果在 Firefox 浏览器中查看，您应该看到以下截图：

![使用 Zend_Feed](img/8727_09_01.jpg)

然而，在 Chrome 浏览器中查看时，我们看到原始的 XML 被显示出来，如下面的截图所示：

![使用 Zend_Feed](img/8727_09_02.jpg)

这可能取决于您的版本。您可能还会被提示选择要安装的可用 RSS 阅读器扩展，例如 Google Reader 或 Chrome 的 RSS Feed Reader 扩展。

# 创建用户友好的 URL

到目前为止，在我们的开发过程中，我们一直在使用 Yii 应用程序 URL 结构的默认格式。这种格式在第二章中讨论过，*入门*，在*回顾我们的请求路由*一节中使用了查询字符串的方法。我们有主要参数“r”，代表*路由*，后面跟着 controllerID/actionID 对，然后是特定 action 方法需要的可选查询字符串参数。我们为我们的新 feed 创建的 URL 也不例外。它是一个又长又笨重，可以说是丑陋的 URL。肯定有更好的方法！事实上确实如此。

我们可以通过使用所谓的*路径*格式使先前提到的 URL 看起来更清晰、更易理解，这种格式消除了查询字符串，并将`GET`参数放入 URL 的路径信息部分：

以我们的评论 feed URL 为例，我们将不再使用`http://localhost/trackstar/index.php?r=comment/feed`，而是使用`http://localhost/trackstar/index.php/comment/feed/`。

而且，我们不需要为每个请求指定入口脚本。我们还可以利用 Yii 的请求路由配置选项来消除指定 controllerID/actionID 对的需要。我们的请求可能看起来像这样：

`http://localhost/trackstar/commentfeed`

另外，通常情况下，特别是在 feed 的 URL 中，最后会指定`.xml`扩展名。因此，如果我们能够修改我们的 URL，使其看起来像下面这样，那就太好了：

`http://localhost/trackstar/commentfeed.xml`

这大大简化了用户的 URL，并且也是 URL 被主要搜索引擎正确索引的绝佳格式（通常称为“搜索引擎友好的 URL”）。让我们看看如何使用 Yii 的 URL 管理功能来修改我们的 URL 以匹配这种期望的格式。

## 使用 URL 管理器

Yii 中内置的 URL 管理器是一个应用程序组件，可以在`protected/config/main.php`文件中进行配置。让我们打开该文件，并在 components 数组中添加一个新的 URL 管理器组件声明：

```php
'urlManager'=>array(
    'urlFormat'=>'path',
 ),    
```

只要我们坚持使用默认的并将组件命名为`urlManager`，我们就不需要指定组件的类，因为在`CWebApplication.php`框架类中预先声明为`CUrlManager.php`。

通过这个简单的添加，我们的 URL 结构已经在整个站点中改变为路径格式。例如，以前，如果我们想要查看 ID 为 1 的特定问题，我们使用以下 URL 进行请求：

`http://localhost/trackstar/index.php?r=issue/view&id=1`

现在，通过这些更改，我们的 URL 看起来是这样的：

`http://localhost/trackstar/index.php/issue/view/id/1`

您会注意到我们所做的更改已经影响了应用程序中生成的所有 URL。要查看这一点，再次访问我们的订阅，转到`http://localhost/trackstar/index.php/comment/feed/`。我们注意到，所有我们的问题链接都已经被重新格式化为这个新的结构。这都归功于我们一贯使用控制器方法和其他辅助方法来生成我们的 URL。我们只需在一个配置文件中更改 URL 格式，这些更改就会自动传播到整个应用程序。

我们的 URL 看起来更好了，但我们仍然有入口脚本`index.php`，并且我们还不能在我们的订阅 URL 的末尾添加`.xml`后缀。因此，让我们隐藏`index.php`文件作为 URL 的一部分，并设置请求路由以理解对`commentfeed.xml`的请求实际上意味着对`CommentController::actionFeed()`的请求。让我们先解决后者。

### 配置路由规则

Yii URL 管理器允许我们指定规则来定义 URL 的解析和创建方式。规则由定义路由和模式组成。模式用于匹配 URL 的路径信息部分，以确定使用哪个规则来解析或创建 URL。模式可以包含使用语法 `ParamName:RegExp` 的命名参数。在解析 URL 时，匹配的规则将从路径信息中提取这些命名参数，并将它们放入 `$_GET` 变量中。当应用程序创建 URL 时，匹配的规则将从 `$_GET` 中提取命名参数，并将它们放入创建的 URL 的路径信息部分。如果模式以 `/*` 结尾，这意味着可以在 URL 的路径信息部分附加额外的 `GET` 参数。

要指定 URL 规则，将`CUrlManager`文件的`rules`属性设置为规则数组，格式为`pattern=>route`。

例如，让我们看看以下两条规则：

```php
'urlManager'=>array(
  'urlFormat'=>'path',
  'rules'=>array(
  'issues'=>'issue/index',
  'issue/<id:\d+>/*'=>'issue/view',
  ),
)
```

这段代码中指定了两条规则。第一条规则表示，如果用户请求 URL `http://localhost/trackstar/index.php/issues`，则应该被视为 `http://localhost/trackstar/index.php/issue/index`，在构建 URL 时也是一样的。因此，例如，如果我们在应用程序中使用控制器的 `createUrl('issue/index')` 方法创建 URL，它将生成 `/trackstar/index.php/issues` 而不是 `/trackstar/index.php/issue/index`。

第二条规则包含一个命名参数`id`，使用`<ParamName:RegExp>`语法指定。它表示，例如，如果用户请求 URL `http://localhost/trackstar/index.php/issue/1`，则应该被视为 `http://localhost/trackstar/index.php/issue/view/id/1`。在构建这样的 URL 时也是一样的。

路由也可以被指定为一个数组本身，以允许设置其他属性，比如 URL 后缀以及路由是否应该被视为区分大小写。当我们为我们的评论订阅指定规则时，我们将利用这些属性。

让我们将以下规则添加到我们的`urlManager`应用程序组件配置中：

```php
'urlManager'=>array(
        'urlFormat'=>'path',   
 **'rules'=>array(   'commentfeed'=>array('comment/feed', 'urlSuffix'=>'.xml', 'caseSensitive'=>false),**
      ), 
), 
```

在这里，我们使用了`urlSuffix`属性来指定我们期望的 URL`.xml`后缀。

现在我们可以通过以下 URL 访问我们的订阅：

`http://localhost/trackstar/index.php/commentFeed.xml`

#### 从 URL 中删除入口脚本

现在我们只需要从 URL 中删除`index.php`部分。这可以通过以下两个步骤完成：

1.  修改 Web 服务器配置，将所有不对应现有文件或目录的请求重定向到`index.php`。

1.  将`urlManager`组件的`showScriptName`属性设置为`false`。

第一步处理了应用程序如何路由请求，而后者处理了应用程序中 URL 的创建方式。

由于我们使用 Apache HTTP 服务器，我们可以通过在应用程序根目录中创建一个`.htaccess`文件并向该文件添加以下指令来执行第一步：

```php
# Turning on the rewrite engine is necessary for the following rules and features.
# FollowSymLinks must be enabled for this to work.
<IfModule mod_rewrite.c>
  Options +FollowSymlinks
  RewriteEngine On
</IfModule>

# Unless an explicit file or directory exists, redirect all request to Yii entry script
<IfModule mod_rewrite.c>
  RewriteCond %{REQUEST_FILENAME} !-f
  RewriteCond %{REQUEST_FILENAME} !-d
  RewriteRule . index.php
</IfModule>
```

### 注意

这种方法仅适用于 Apache HTTP 服务器。如果使用不同的 Web 服务器，您将需要查阅 Web 服务器重写规则。还要注意，这些信息可以放在主 Apache 配置文件中，作为使用`.htaccess`文件方法的替代方法。

有了这个`.htaccess`文件，我们现在可以通过导航到`http://localhost/trackstar/commentfeed.xml`（或`http://localhost/trackstar/commentFeed.xml`，因为我们将大小写敏感性设置为 false）来访问我们的源。

然而，即使有了这个，如果我们在应用程序中使用控制器方法或`CHtml`助手方法之一来创建我们的 URL，比如在控制器类中执行`$this->createAbsoluteUrl('comment/feed');`，它将生成以下 URL，其中 URL 中仍然包含`index.php`：

`http://localhost/trackstar/index.php/commentfeed.xml`

为了指示它在生成 URL 时不使用条目脚本名称，我们需要在`urlManager`组件上设置该属性。我们在`main.php`配置文件中再次执行此操作，如下所示：

```php
'urlManager'=>array(
    'urlFormat'=>'path',   
    'rules'=>array(
       'commentfeed'=>array('comment/feed', 'urlSuffix'=>'.xml', 'caseSensitive'=>false),
  ), 
 **'showScriptName'=>false,**
 ),   
```

为了处理 URL 中项目 ID 的添加，我们需要将评论源数据限制为与特定项目相关联的评论，为此我们需要添加另一条规则，如下所示：

```php
'urlManager'=>array(
        'urlFormat'=>'path',   
        'rules'=>array(   
 **'<pid:\d+>/commentfeed'=>array('comment/feed', 'urlSuffix'=>'.xml', 'caseSensitive'=>false),**
         'commentfeed'=>array('comment/feed', 'urlSuffix'=>'.xml', 'caseSensitive'=>false),
      ), 
      'showScriptName'=>false,
),
```

这个规则还使用了`<Parameter:RegEx>`语法来指定一个模式，以允许在 URL 的`commentfeed.xml`部分之前指定项目 ID。有了这个规则，我们可以将我们的 RSS 源限制为特定项目的评论。例如，如果我们只想要与项目#`2`相关联的评论，URL 格式将是：

`http://localhost/trackstar/2/commentfeed.xml`

# 添加订阅链接

现在我们已经创建了我们的源并改变了 URL 结构，使其更加用户友好和搜索引擎友好，我们需要添加用户订阅源的功能。其中一种方法是在我们想要添加 RSS 源链接的页面渲染之前添加以下代码。让我们在项目列表页面以及特定项目详细信息页面都这样做。我们将从项目列表页面开始。这个页面由`ProjectController::actionIndex()`方法渲染。修改该方法如下：

```php
public function actionIndex()
{
    $dataProvider=new CActiveDataProvider('Project');

 **Yii::app()->clientScript->registerLinkTag(**
 **'alternate',**
 **'application/rss+xml',**
 **$this->createUrl('comment/feed'));**

    $this->render('index',array(
      'dataProvider'=>$dataProvider,
    ));
}
```

这里显示的突出显示的代码将添加以下内容到渲染的 HTML 的`<head>`标签中：

```php
<link rel="alternate" type="application/rss+xml" href="/commentfeed.xml" />
```

在许多浏览器中，这将自动生成一个小的 RSS 源图标在地址栏中。以下截图显示了 Safari 地址栏中这个图标的样子：

![添加订阅链接](img/8727_09_03.jpg)

我们进行类似的更改，以将此链接添加到特定项目详细信息页面。这些页面的渲染由`ProjectController::actionView()`方法处理。修改该方法如下：

```php
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

 **Yii::app()->clientScript->registerLinkTag(**
 **'alternate',**
 **'application/rss+xml',**
 **$this->createUrl('comment/feed',array('pid'=>$this->loadModel($id)->id)));**

    $this->render('view',array(
      'model'=>$this->loadModel($id),
      'issueDataProvider'=>$issueDataProvider,
    ));

  }
```

这几乎与我们添加到索引方法中的内容相同，只是我们正在指定项目 ID，以便我们的评论条目仅限于与该项目相关联的条目。类似的图标现在将显示在我们项目详细信息页面的地址栏中。单击这些图标允许用户订阅这些评论源。

### 注意

`registerLinkTag()`方法还允许您在第四个参数中指定媒体属性，然后您可以进一步指定其他支持的属性作为`name=>value`对的数组，作为第五个参数。有关使用此方法的更多信息，请参见[`www.yiiframework.com/doc/api/1.1/CClientScript/#registerLinkTag-detail`](http://www.yiiframework.com/doc/api/1.1/CClientScript/#registerLinkTag-detail)。

# 摘要

本章展示了如何轻松地将 Yii 与其他外部框架集成。我们特别使用了流行的 Zend Framework 来进行演示，并能够快速地向我们的应用程序添加符合 RSS 标准的 Web 订阅。虽然我们特别使用了`Zend_Feed`，但我们真正演示了如何将 Zend Framework 的任何组件集成到应用程序中。这进一步扩展了 Yii 已经非常丰富的功能，使 Yii 应用程序变得非常功能丰富。

我们还了解了 Yii 中的 URL 管理功能，并在整个应用程序中改变了我们的 URL 格式，使其更加用户和搜索引擎友好。这是改进我们应用程序外观和感觉的第一步，这是我们到目前为止非常忽视的事情。在下一章中，我们将更仔细地研究 Yii 应用程序的展示层。样式、主题以及通常使事物看起来好看将是下一章的重点。
