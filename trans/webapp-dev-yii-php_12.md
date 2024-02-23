# 第十二章。投产准备

尽管我们的应用程序缺乏大量的功能功能，我们（虽然是想象中的）截止日期正在临近，我们（同样是想象中的）客户对将应用程序投入生产环境感到焦虑。尽管我们的应用程序在生产中真正见到天日可能还需要一些时间，但现在是时候让应用程序“准备投产”了。在我们的最后一个开发章节中，我们将做到这一点。

# 功能规划

为了实现我们的应用程序为生产环境做好准备的目标，我们将专注于以下细粒度的任务：

+   实现 Yii 的应用程序日志记录框架，以确保我们记录关于关键生产错误和事件的信息

+   实现 Yii 的应用程序错误处理框架，以确保我们在生产中正确处理错误，并了解这在生产环境和开发环境中的工作方式有所不同

+   实现应用程序数据缓存以帮助提高性能

# 日志记录

日志记录是一个在应用程序开发的这个后期阶段应该被讨论的话题。在软件应用程序的故障排除中，信息、警告和严重错误消息是非常宝贵的，尤其是对于那些在生产环境中由真实用户使用的应用程序。

作为开发人员，我们都熟悉这个故事。您已经满足了您正在构建的应用程序的所有功能要求。所有单元和功能测试都通过了。应用程序已经通过了 QA 的批准，每个人都对它准备投产感到很满意。但是一旦它投入使用，并且承受着真实用户的真实生产负载，它的行为就会出乎意料。一个良好的日志记录策略可能会成为快速解决问题和回滚数周甚至数月的辛苦工作之间的区别。

Yii 提供了灵活和可扩展的日志记录功能。记录的数据可以根据日志级别和消息类别进行分类。使用级别和类别过滤器，日志消息可以进一步路由到不同的目的地，例如写入磁盘上的文件，存储在数据库中，发送给管理员作为电子邮件，或在浏览器窗口中显示。

## 消息记录

我们的应用程序实际上一直在每个请求时记录许多信息消息。当初始应用程序被创建时，它被配置为处于*调试*模式，而在此模式下，Yii 框架本身会记录信息消息。我们实际上看不到这些消息，因为默认情况下它们被记录到内存中。因此，它们只在请求的生命周期内存在。

应用程序是否处于调试模式由根目录`index.php`文件中的以下行控制：

```php
defined('YII_DEBUG') or define('YII_DEBUG',true);
```

为了查看被记录的内容，让我们在我们的`SiteController`类中快速创建一个动作方法来显示这些消息：

```php
public function actionShowLog()
{
  echo "Logged Messages:<br><br>";
CVarDumper::dump(Yii::getLogger()->getLogs());
}
```

在这里，我们使用 Yii 的`CVarDumper`辅助类，这是`var_dump`或`print_r`的改进版本，因为它能够正确处理递归引用对象。

如果我们通过发出请求`http://localhost/trackstar/site/showLog`来调用此动作，我们会看到类似以下截图的内容：

![消息记录](img/8727_12_01.jpg)

如果我们注释掉在`index.php`中定义的全局应用程序调试变量，并刷新页面，我们会注意到一个空数组；也就是说，没有记录任何内容。这是因为这种系统级别的调试信息级别的日志记录是通过调用`Yii::trace`来实现的，只有在应用程序处于特殊的调试模式时才会记录这些消息。

我们可以使用两种静态应用程序方法之一在 Yii 应用程序中记录消息：

+   `Yii::log($message, $level, $category);`

+   `Yii::trace($message, $category);`

正如前面提到的，这两种方法之间的主要区别在于`Yii::trace`仅在应用程序处于调试模式时记录消息。

### 类别和级别

在使用`Yii::log()`记录消息时，我们需要指定它的类别和级别。**类别**是一个字符串，用于为被记录的消息提供额外的上下文。这个字符串可以是任何你喜欢的，但许多人使用的约定是一个格式为`xxx.yyy.zzz`的字符串，类似于路径别名。例如，如果在我们的应用程序的`SiteController`类中记录了一条消息，我们可以选择使用类别`application.controllers.SiteController`。

除了指定类别，使用`Yii::log`时，我们还可以指定消息的级别。级别可以被认为是消息的严重程度。您可以定义自己的级别，但通常它们具有以下值之一：

+   **跟踪**：这个级别通常用于跟踪应用程序在开发过程中的执行流程。

+   **信息**：这是用于记录一般信息。如果没有指定级别，则这是默认级别。

+   **概要**：这是用于性能概要功能，稍后在本章中描述。

+   **警告**：这是用于警告消息。

+   **错误**：这是用于致命错误消息。

### 添加登录消息日志

例如，让我们向我们的用户登录方法添加一些日志记录。我们将在方法开始时提供一些基本的调试信息，以指示方法正在执行。然后，我们将在成功登录时记录一条信息，以及在登录失败时记录一条警告信息。根据以下突出显示的代码修改我们的`SiteController::actionLogin()`方法（整个方法在可下载的代码中已经存在，或者您可以从[`gist.github.com/3791860`](https://gist.github.com/3791860)下载独立的方法）。

```php
public function actionLogin()
{
 **Yii::trace("The actionLogin() method is being requested", "application.controllers.SiteController");**
    …

    // collect user input data
    if(isset($_POST['LoginForm']))
    {
      …  
if($model->validate() && $model->login()) 
      {
 **Yii::log("Successful login of user: " . Yii::app()->user->id, "info", "application.controllers.SiteController");**
        $this->redirect(Yii::app()->user->returnUrl);
 **}**
 **else**
 **{**
 **Yii::log("Failed login attempt", "warning", "application.controllers.SiteController");**
 **}**

    }
    …
}
```

如果我们现在成功登录（或进行了失败的尝试）并访问我们的页面查看日志，我们看不到它们（如果您注释掉了调试模式声明，请确保您已经将应用程序重新放回调试模式进行此练习）。同样，原因是，默认情况下，Yii 中的日志实现只是将消息存储在内存中。它们在请求完成时消失。这并不是非常有用。我们需要将它们路由到一个更持久的存储区域，这样我们就可以在生成它们的请求之外查看它们。

## 消息路由

正如我们之前提到的，默认情况下，使用`Yii::log`或`Yii::trace`记录的消息被保存在内存中。通常，如果这些消息在浏览器窗口中显示，保存到一些持久存储（如文件中），在数据库中，或作为电子邮件发送，它们会更有用。Yii 的*消息路由*允许将日志消息路由到不同的目的地。

在 Yii 中，消息路由由`CLogRouter`应用组件管理。它允许您定义日志消息应路由到的目的地列表。

为了利用这个消息路由，我们需要在`protected/config/main.php`配置文件中配置`CLogRouter`应用组件。我们通过设置它的 routes 属性与所需的日志消息目的地进行配置。

如果我们打开主配置文件，我们会看到一些配置已经提供（再次感谢使用`yiic webapp`命令最初创建我们的应用程序）。以下内容已在我们的配置中定义：

```php
'log'=>array
  'class'=>'CLogRouter',
  'routes'=>array(
    array(
      'class'=>'CFileLogRoute',
      'levels'=>'error, warning',
    ),
    // uncomment the following to show log messages on web pages
    /*
    array(
      'class'=>'CWebLogRoute',
    ),
    */
  ),
),
```

`log`应用组件配置为使用框架类`CLogRouter`。当然，如果您有日志要求没有完全满足基础框架实现，您也可以创建和使用自定义子类；但在我们的情况下，这将工作得很好。

在先前配置中类定义之后的是`routes`属性的定义。在这种情况下，只指定了一个路由。这个路由使用了 Yii 框架的消息路由类`CFileLogRoute`。`CFileLogRoute`消息路由类使用文件系统保存消息。默认情况下，消息被记录在应用运行时目录下的一个文件中，即`/protected/runtime/application.log`。实际上，如果您一直在跟着我们并且有自己的应用程序，您可以查看这个文件，会看到框架记录的几条消息。`levels`规定只有日志级别为`error`或`warning`的消息才会被路由到这个文件。在先前代码中被注释掉的部分指定了另一个路由`CWebLogRoute`。如果使用，这将把消息路由到当前请求的网页上。以下是 Yii 1.1 版本当前可用的消息路由列表：

+   `CDbLogRoute`：将消息保存在数据库表中

+   `CEmailLogRoute`：将消息发送到指定的电子邮件地址

+   `CFileLogRoute`：将消息保存在应用程序的`runtime`目录下的文件中，或者您选择的任何其他目录中

+   `CWebLogRoute`：在当前网页末尾显示消息

+   `CProfileLogRoute`：在当前网页末尾显示分析消息

我们在`SiteController::actionLogin()`方法中添加的日志记录使用了`Yii::trace`来记录一条消息，然后使用`Yii::log`来记录另外两条消息。使用`Yii::trace`时，日志级别会自动设置为`trace`。当使用`Yii::log`时，如果登录成功，我们指定为`info`日志级别，但如果登录尝试失败，则为`warning`级别。让我们修改日志路由配置，将`trace`和`info`级别的消息写入到一个新的、单独的文件`infoMessages.log`中，该文件与我们的`application.log`文件在同一目录中。另外，让我们配置它将警告消息写入到浏览器。为此，我们将对配置进行以下更改（已突出显示）：

```php
'log'=>array(
  'class'=>'CLogRouter',
  'routes'=>array(
    array(
      'class'=>'CFileLogRoute',
 **'levels'=>'error',**
 **),**
 **array(**
 **'class'=>'CFileLogRoute',**
 **'levels'=>'info, trace',**
 **'logFile'=>'infoMessages.log',**
 **),**
 **array(**
 **'class'=>'CWebLogRoute',**
 **'levels'=>'warning',**
 **),**

```

现在，在保存这些更改后，让我们尝试不同的场景。首先，尝试成功的登录。这样做将把我们的两条登录消息写入到我们的新的`/protected/runtime/infoMessages.log`文件中，一条是 trace，另一条是记录成功登录。成功登录后，查看该文件会显示以下内容（完整列表被截断以节省一些树木）：

```php
.....
**2012/06/15 00:31:52 [trace] [application.controllers.SiteController] The actionLogin() method is being requested**
2012/06/15 00:31:52 [trace] [system.web.CModule] Loading "user" application component
2012/06/15 00:31:52 [trace] [system.web.CModule] Loading "session" application component
2012/06/15 00:31:52 [trace] [system.web.CModule] Loading "db"                                                                                                                                                                                                                                                                                                                                                                                                                             application component
2012/06/15 00:31:52 [trace] [system.db.CDbConnection] Opening DB connection
.....
**2012/06/15 00:31:52 [info] [application.controllers.SiteController] Successful login of user: 1**
.....
```

如您所见，其中有很多内容，不仅仅是我们的两条消息！但我们的两条确实显示出来了；它们在先前的列表中是加粗的。现在我们将所有的 trace 消息路由到这个新文件中，所有框架的 trace 消息也会显示在这里。这实际上非常有信息量，真的有助于您了解请求在框架中的生命周期。在幕后有很多事情发生。当将此应用程序移至生产环境时，我们显然会关闭这种冗长的日志记录。在非调试模式下，我们只会看到我们的单个`info`级别消息。但在追踪错误和弄清楚应用程序在做什么时，这种详细级别的信息非常有用。知道它在需要时/如果需要时存在是非常令人安心的。

现在让我们尝试失败的登录尝试场景。如果我们现在注销并再次尝试登录，但这次指定不正确的凭据以强制登录失败，我们会看到我们的**警告**级别显示在返回的网页底部，就像我们配置的那样。以下屏幕截图显示了显示此警告：

![消息路由](img/8727_12_02.jpg)

使用`CFileLogRouter`消息路由器时，日志文件存储在`logPath`属性下，并且文件名由`logFile`方法指定。这个日志路由器的另一个很棒的功能是自动日志文件轮换。如果日志文件的大小大于`maxFileSize`属性中设置的值（以千字节为单位），则会执行轮换，将当前日志文件重命名为带有`.1`后缀的文件。所有现有的日志文件都向后移动一个位置，即`.2`到`.3`，`.1`到`.2`。属性`maxLogFiles`可用于指定要保留多少个文件。

### 注意

如果在应用程序中使用`die;`或`exit;`来终止执行，日志消息可能无法正确写入其预期的目的地。如果需要显式终止 Yii 应用程序的执行，请使用`Yii::app()->end()`。这提供了应用程序成功写出日志消息的机会。此外，`CLogger`组件具有一个`$autoDump`属性，如果设置为`true`，将允许实时将日志消息写入其目的地（即在调用`->log()`时）。由于潜在的性能影响，这应仅用于调试目的，但可以是一个非常有价值的调试选项。

# 处理错误

正确处理软件应用程序中不可避免发生的错误非常重要。这又是一个话题，可以说应该在编写应用程序之前就已经涵盖了，而不是在这个晚期阶段。幸运的是，由于我们一直在依赖 Yii 框架内的工具来自动生成我们的核心应用程序骨架，我们的应用程序已经在利用 Yii 的一些错误处理功能。

Yii 提供了一个基于 PHP 5 异常的完整错误处理框架，这是通过集中的点处理程序中的异常情况的内置机制。当主 Yii 应用程序组件被创建来处理传入的用户请求时，它会注册其`CApplication::handleError()`方法来处理 PHP 警告和通知，并注册其`CApplication::handleException()`方法来处理未捕获的 PHP 异常。因此，如果在应用程序执行期间发生 PHP 警告/通知或未捕获的异常，其中一个错误处理程序将接管控制并启动必要的错误处理过程。

### 注意

错误处理程序的注册是在应用程序的构造函数中通过调用 PHP 函数`set_exception_handler`和`set_error_handler`来完成的。如果您不希望 Yii 处理这些类型的错误和异常，可以通过在主`index.php`入口脚本中将全局常量`YII_ENABLE_ERROR_HANDLER`和`YII_ENABLE_EXCEPTION_HANDLER`定义为 false 来覆盖此默认行为。

默认情况下，应用程序将使用框架类`CErrorHandler`作为负责处理 PHP 错误和未捕获异常的应用程序组件。这个内置应用程序组件的任务之一是使用适当的视图文件显示这些错误，这取决于应用程序是在*调试*模式还是*生产*模式下运行。这允许您为这些不同的环境自定义错误消息。在开发环境中显示更详细的错误信息以帮助解决问题是有意义的。但允许生产应用程序的用户查看相同的信息可能会影响安全性。此外，如果您在多种语言中实现了您的站点，`CErrorHandler`还会选择用于显示错误的首选语言。

在 Yii 中，您引发异常的方式与通常引发 PHP 异常的方式相同。在需要时，可以使用以下一般语法引发异常：

```php
throw new ExceptionClass('ExceptionMessage');
```

Yii 提供的两个异常类是：

+   `CException`

+   `CHttpException`

`CException`是一个通用的异常类。`CHttpException`表示一个 HTTP 错误，并且还携带一个`statusCode`属性来表示 HTTP 状态码。在浏览器中，错误的显示方式取决于抛出的异常类。

## 显示错误

正如之前提到的，当`CErrorHandler`应用组件处理错误时，它会决定在显示错误时使用哪个视图文件。如果错误是要显示给最终用户的，就像使用`CHttpException`时一样，其默认行为是使用一个名为`errorXXX`的视图，其中`XXX`代表 HTTP 状态码（例如，400、404 或 500）。如果错误是内部错误，只应显示给开发人员，它将使用一个名为`Exception`的视图。当应用程序处于调试模式时，将显示完整的调用堆栈以及源文件中的错误行。

然而，当应用程序运行在生产模式下时，所有错误都将使用`errorXXX`视图文件显示。这是因为错误的调用堆栈可能包含不应该显示给任何最终用户的敏感信息。 

当应用程序处于生产模式时，开发人员应依靠错误日志提供有关错误的更多信息。当发生错误时，错误级别的消息将始终被记录。如果错误是由 PHP 警告或通知引起的，消息将被记录为`php`类别。如果错误是由未捕获的`exception`引起的，类别将是`exception.ExceptionClassName`，其中异常类名是`CHttpException`或`CException`的一个或子类。因此，可以利用前一节讨论的日志记录功能来监视生产应用程序中发生的错误。当然，如果发生致命的 PHP 错误，您仍然需要检查由 PHP 配置设置定义的错误日志，而不是 Yii 的错误日志。

默认情况下，`CErrorHandler`按以下顺序搜索相应视图文件的位置：

+   `WebRoot/themes/ThemeName/views/system`：当前活动主题下的系统视图目录

+   `WebRoot/protected/views/system`：应用程序的默认系统视图目录

+   `YiiRoot/framework/views`：Yii 框架提供的标准系统视图目录

您可以通过在应用程序或主题的系统视图目录下创建自定义错误视图文件来自定义错误显示。

Yii 还允许您定义一个特定的控制器动作方法来处理错误的显示。这实际上是我们的应用程序配置的方式。当我们通过一些示例时，我们会看到这一点。

我们使用 Gii Crud Generator 工具创建 CRUD 脚手架时为我们生成的一些代码已经利用了 Yii 的错误处理。其中一个例子是`ProjectController::loadModel()`方法。该方法定义如下：

```php
public function loadModel($id)
  {
    $model=Project::model()->findByPk($id);
    if($model===null)
      throw new CHttpException(404,'The requested page does not exist.');
    return $model;
  }
```

我们看到它正在尝试基于输入的`id`查询字符串参数加载相应的项目模型 AR 实例。如果它无法定位请求的项目，它会抛出一个`CHttpException`，以通知用户他们请求的页面（在本例中是项目详细信息页面）不存在。我们可以通过明确请求我们知道不存在的项目来在浏览器中测试这一点。由于我们知道我们的应用程序没有与`id`为`99`相关联的项目，因此请求`http://localhost/trackstar/project/view/id/99`将导致返回以下页面：

![显示错误](img/8727_12_03.jpg)

这很好，因为页面看起来像我们应用程序中的任何其他页面，具有相同的主题、页眉、页脚等。

实际上，这不是呈现此类型错误页面的默认行为。 我们的初始应用程序配置为使用特定的控制器操作来处理此类错误。 我们提到这是处理应用程序中错误的另一种选项。 如果我们查看主配置文件`/protected/config/main.php`，我们会看到以下应用程序组件声明：

```php
'errorHandler'=>array(
  // use 'site/error' action to display errors
    'errorAction'=>'site/error',
),
```

这配置了我们的错误处理程序应用组件使用`SiteController::actionError()`方法来处理所有打算显示给用户的异常。 如果我们查看该操作方法，我们会注意到它正在呈现`protected/views/site/error.php`视图文件。 这只是一个普通的控制器视图文件，因此它还将呈现任何相关的应用程序布局文件，并将应用适当的主题。 通过这种方式，我们能够在发生某些错误时为用户提供非常友好的体验。

要查看默认行为是什么，而不添加此配置，请暂时注释掉先前的配置代码行（在`protected/config/main.php`中），然后再次请求不存在的项目。 现在我们看到以下页面：

![显示错误](img/8727_12_04.jpg)

由于我们没有明确定义任何遵循先前概述的自定义错误页面，这是 Yii 框架本身的`framework/views/error404.php`文件。

继续并恢复对配置文件的更改，以再次使用`SiteController::actionError()`方法进行错误处理。

现在让我们看看这与抛出`CException`类相比如何。 让我们注释掉当前抛出 HTTP 异常的代码行，并添加一个新行来抛出这个其他异常类。 对`protected/controllers/ProjectController.php`文件进行突出显示的更改：

```php
public function loadModel($id)
  {
    $model=Project::model()->findByPk($id);
    if($model===null)
 **//throw new CHttpException(404,'The requested page does not exist.');**
 **throw new CException('This is an example of throwing a CException');**
    return $model;
  }
```

现在，如果我们请求一个不存在的项目，我们会看到一个非常不同的结果。 这次我们看到一个由系统生成的错误页面，其中包含完整的堆栈跟踪错误信息转储，以及发生错误的特定源文件：

![显示错误](img/8727_12_05.jpg)

它显示了抛出`CException`类的事实，以及描述**这是抛出 CException 的示例**，源文件，发生错误的文件中的特定行，然后是完整的堆栈跟踪。

因此，抛出这个不同的异常类，以及应用程序处于调试模式的事实，会产生不同的结果。 这是我们希望显示以帮助排除问题的信息类型，但前提是我们的应用程序在私人开发环境中运行。 让我们暂时注释掉根`index.php`文件中的调试设置，以查看在“生产”模式下如何显示：

```php
// remove the following line when in production mode
//defined('YII_DEBUG') or define('YII_DEBUG',true);
```

如果我们刷新对不存在的项目的请求，我们会看到异常显示为面向最终用户友好的 HTTP 500 错误，如下截图所示：

![显示错误](img/8727_12_06.jpg)

因此，我们看到在“生产”模式下不会显示任何敏感代码或堆栈跟踪信息。

# 缓存

**缓存**数据是帮助提高生产 Web 应用程序性能的一种很好的方法。 如果有特定内容不希望在每个请求时都更改，那么使用缓存来存储和提供此内容可以减少检索和处理数据所需的时间。

Yii 在缓存方面提供了一些不错的功能。 要利用 Yii 的缓存功能，您首先需要配置一个缓存应用程序组件。 这样的组件是几个子类之一，它们扩展了`CCache`，这是具有不同缓存存储实现的缓存类的基类。

Yii 提供了许多特定的缓存组件类实现，利用不同的方法存储数据。以下是 Yii 在版本 1.1.12 中提供的当前缓存实现的列表：

+   `CMemCache`：使用 PHP memcache 扩展。

+   `CApcCache`：使用 PHP APC 扩展。

+   `CXCache`：使用 PHP XCache 扩展。

+   `CEAcceleratorCache`：使用 PHP EAccelerator 扩展。

+   `CDbCache`：使用数据库表存储缓存数据。默认情况下，它将在运行时目录下创建并使用 SQLite3 数据库。您可以通过设置其`connectionID`属性来显式指定要使用的数据库。

+   `CZendDataCache`：使用 Zend Data Cache 作为底层缓存介质。

+   `CFileCache`：使用文件存储缓存数据。这对于缓存大量数据（如页面）特别合适。

+   `CDummyCache`：提供一致的缓存接口，但实际上不执行任何缓存。这种实现的原因是，如果您面临开发环境不支持缓存的情况，您仍然可以执行和测试需要在可用时使用缓存的代码。这使您可以继续编写一致的接口代码，并且当实际实现真正的缓存组件时，您将不需要更改编写用于写入或检索缓存中的数据的代码。

+   `CWinCache`：`CWinCache`基于 WinCache 实现了一个缓存应用程序组件。有关更多信息，请访问[`www.iis.net/expand/wincacheforphp`](http://www.iis.net/expand/wincacheforphp)。

所有这些组件都是从同一个基类`CCache`继承，并公开一致的 API。这意味着您可以更改应用程序组件的实现，以使用不同的缓存策略，而无需更改任何使用缓存的代码。

## 缓存配置

正如前面提到的，Yii 中使用缓存通常涉及选择其中一种实现，然后在`/protected/config/main.php`文件中配置应用程序组件以供使用。配置的具体内容当然取决于具体的缓存实现。例如，如果要使用 memcached 实现，即`CMemCache`，这是一个分布式内存对象缓存系统，允许您指定多个主机服务器作为缓存服务器，配置它使用两个服务器可能如下所示：

```php
array(
    ......
    'components'=>array(
        ......
        'cache'=>array(
            'class'=>'system.caching.CMemCache',
            'servers'=>array(
                array('host'=>'server1', 'port'=>12345, 'weight'=>60),
                array('host'=>'server2', 'port'=>12345, 'weight'=>40),
            ),
        ),
    ),
);
```

为了让读者在跟踪 Star 开发过程中保持相对简单，我们将在一些示例中使用文件系统实现`CFileCache`。这应该在任何允许从文件系统读取和写入文件的开发环境中都是 readily available。

### 注意

如果由于某种原因这对您来说不是一个选项，但您仍然想要跟随代码示例，只需使用`CDummyCache`选项。正如前面提到的，它实际上不会在缓存中存储任何数据，但您仍然可以根据其 API 编写代码，并在以后更改实现。

`CFileCache`提供了基于文件的缓存机制。使用这种实现时，每个被缓存的数据值都存储在一个单独的文件中。默认情况下，这些文件存储在`protected/runtime/cache/`目录下，但可以通过在配置组件时设置`cachePath`属性来轻松更改这一点。对于我们的目的，这个默认值是可以的，所以我们只需要在`/protected/config/main.php`配置文件的`components`数组中添加以下内容，如下所示：

```php
// application components
  'components'=>array(
    …
 **'cache'=>array(**
 **'class'=>'system.caching.CFileCache',**
 **),**
     …  
),
```

有了这个配置，我们可以在运行的应用程序中的任何地方通过`Yii::app()->cache`访问这个新的应用程序组件。

## 使用基于文件的缓存

让我们尝试一下这个新组件。还记得我们在上一章作为管理功能的一部分添加的系统消息吗？我们不必在每次请求时从数据库中检索它，而是将最初从数据库返回的值存储在我们的缓存中，以便有限的时间内不必从数据库中检索数据。

让我们向我们的`SysMessage`（`/protected/modules/admin/models/SysMessage.php`）AR 模型类添加一个新的公共方法来处理最新系统消息的检索。让我们将这个新方法同时设置为`public`和`static`，以便应用程序的其他部分可以轻松使用这个方法来访问最新的系统消息，而不必显式地创建`SysMessage`的实例。

将我们的方法添加到`SysMessage`类中，如下所示：

```php
/**
   * Retrieves the most recent system message.
   * @return SysMessage the AR instance representing the latest system message.
   */

public static function getLatest()
{

  //see if it is in the cache, if so, just return it
  if( ($cache=Yii::app()->cache)!==null)
  {
    $key='TrackStar.ProjectListing.SystemMessage';
    if(($sysMessage=$cache->get($key))!==false)
      return $sysMessage;
  }
  //The system message was either not found in the cache, or   
//there is no cache component defined for the application
//retrieve the system message from the database 
  $sysMessage = SysMessage::model()->find(array(
    'order'=>'t.update_time DESC',
  ));
  if($sysMessage != null)
  {
    //a valid message was found. Store it in cache for future retrievals
    if(isset($key))
      $cache->set($key,$sysMessage,300);    
      return $sysMessage;
  }
  else
      return null;
}
```

我们将在接下来的一分钟内详细介绍。首先，让我们更改我们的应用程序以使用这种新方法来验证缓存是否正常工作。我们仍然需要更改`ProjectController::actionIndex()`方法以使用这个新创建的方法。这很容易。只需用调用这个新方法替换从数据库生成系统消息的代码。也就是说，在`ProjectController::actionIndex()`中，只需更改以下代码：

```php
$sysMessage = SysMessage::model()->find(array('order'=>'t.update_time DESC',));
```

到以下内容：

```php
$sysMessage = SysMessage::getLatest();
```

现在在项目列表页面上显示的系统消息应该利用文件缓存。我们可以检查缓存目录以进行验证。

如果我们对文件缓存的默认位置`protected/runtime/cache/`进行目录列表，我们确实会看到创建了一些文件。两个文件的名称都相当奇怪（您的可能略有不同）`18baacd814900e9b36b3b2e546513ce8.bin`和`2d0efd21cf59ad6eb310a0d70b25a854.bin`。

一个保存我们的系统消息数据，另一个是我们在前几章中配置的`CUrlManager`的配置。默认情况下，`CUrlManager`将使用缓存组件来缓存解析的 URL 规则。您可以将`CUrlManager`的`cacheId`参数设置为`false`，以禁用此组件的缓存。

如果我们以文本形式打开`18baacd814900e9b36b3b2e546513ce8.bin`文件，我们可以看到以下内容：

```php
a:2:{i:0;O:10:"SysMessage":12:{s:18:" :" CActiveRecord _ _md";N;s:19:" :" CActiveRecord _ _new";b:0;s:26:" :" CActiveRecord _ _attributes";a:6:{s:2:"id";s:1:"2";s:7:"message";s:56:"This is a second message from your system administrator!";s:11:"create_time";s:19:"2012-07-31 21:25:33";s:14:"create_user_id";s:1:"1";s:11:"update_time";s:19:"2012-07-31 21:25:33";s:14:"update_user_id";s:1:"1";}s:23:" :"18CActiveRecord _18_related";a:0:{}s:17:" :" CActiveRecord _ _c";N;s:18:" 18:" CActiveRecord _ _:"  _pk";s:1:"2";s:21:" :" CActiveRecord _ _alias";s:1:"t";s:15:" :" CModel _ _errors";a:0:{}s:19:" :" CModel _ _validators";N;s:17:" :" CModel _ _scenario";s:6:"update";s:14:" :" CComponent _ _e";N;s:14:" :" CComponent _ _m";N;}i:1;N;}
```

这是我们最近更新的`SysMessage` AR 类实例的序列化缓存值，这正是我们希望看到的。因此，我们看到缓存实际上是在工作的。

现在让我们更详细地重新审视一下我们的新`SysMessage::getLatest()`方法的代码。代码的第一件事是检查所请求的数据是否已经在缓存中，如果是，则返回该值：

```php
//see if it is in the cache, if so, just return it
if( ($cache=Yii::app()->cache)!==null)
{
  $key='TrackStar.ProjectListing.SystemMessage';
  if(($sysMessage=$cache->get($key))!==false)
    return $sysMessage;
}
```

正如我们所提到的，我们配置了缓存应用组件，可以通过`Yii::app()->cache`在应用程序的任何地方使用。因此，它首先检查是否已定义这样的组件。如果是，它尝试通过`$cache->get($key)`方法在缓存中查找数据。这做的更多或更少是您所期望的。它尝试根据指定的键从缓存中检索值。键是用于映射到缓存中存储的每个数据片段的唯一字符串标识符。在我们的系统消息示例中，我们只需要一次显示一条消息，因此可以使用一个相当简单的键来标识要显示的单个系统消息。只要对于我们想要缓存的每个数据片段保持唯一，键可以是任何字符串值。在这种情况下，我们选择了描述性字符串`TrackStar.ProjectListing.SystemMessage`作为存储和检索缓存系统消息时使用的键。

当此代码首次执行时，缓存中尚没有与此键值关联的任何数据。因此，对于此键的`$cache->get()`调用将返回`false`。因此，我们的方法将继续执行下一部分代码，简单地尝试从数据库中检索适当的系统消息，使用 AR 类：

```php
$sysMessage = SysMessage::model()->find(array(
  'order'=>'t.update_time DESC',
));
```

然后我们继续以下代码，首先检查我们是否从数据库中得到了任何返回。如果是，它会在返回值之前将其存储在缓存中；否则，将返回`null`：

```php
if($sysMessage != null)
{
  if(isset($key))
    $cache->set($key,$sysMessage->message,300);    
    return $sysMessage->message;
}
else
    return null;
```

如果返回了有效的系统消息，我们使用`$cache->set()`方法将数据存储到缓存中。这个方法的一般形式如下：

```php
set($key,$value,$duration=0,$dependency=null)
```

将数据放入缓存时，必须指定一个唯一的键以及要存储的数据。键是一个唯一的字符串值，如前所述，值是希望缓存的任何数据。只要可以序列化，它可以是任何格式。持续时间参数指定了一个可选的**存活时间**（**TTL**）要求。这可以用来确保缓存的值在一段时间后被刷新。默认值为`0`，这意味着它永远不会过期。（实际上，Yii 在内部将持续时间的值`<=0`翻译为一年后过期。所以，不完全是*永远*，但肯定是很长时间。）

我们以以下方式调用`set()`方法：

```php
$cache->set($key,$sysMessage->message,300);  
```

我们将键设置为之前定义的`TrackStar.ProjectListing.SystemMessage`；要存储的数据是我们返回的`SystemMessage` AR 类的消息属性，即我们的`tbl_sys_message`表的消息列；然后我们将持续时间设置为`300`秒。这样，缓存中的数据将在每 5 分钟后过期，届时将再次查询数据库以获取最新的系统消息。当我们设置数据时，我们没有指定依赖项。我们将在下面讨论这个可选参数。

## 缓存依赖项

依赖参数允许采用一种替代和更复杂的方法来决定缓存中存储的数据是否应该刷新。您的缓存策略可能要求根据特定用户发出请求、应用程序的一般模式、状态或文件系统上的文件是否最近已更新等因素使数据无效，而不是声明缓存数据的过期时间。此参数允许您指定此类缓存验证规则。

依赖项是`CCacheDependency`或其子类的实例。Yii 提供了以下特定的缓存依赖项：

+   `CFileCacheDependency`：如果指定文件的最后修改时间自上次缓存查找以来发生了变化，则缓存中的数据将无效。

+   `CDirectoryCacheDependency`：与文件缓存依赖项类似，但是它检查给定指定目录中的所有文件和子目录。

+   `CDbCacheDependency`：如果指定 SQL 语句的查询结果自上次缓存查找以来发生了变化，则缓存中的数据将无效。

+   `CGlobalStateCacheDependency`：如果指定的全局状态的值发生了变化，则缓存中的数据将无效。全局状态是一个跨多个请求和多个会话持久存在的变量。它通过`CApplication::setGlobalState()`来定义。

+   `CChainedCacheDependency`：这允许您将多个依赖项链接在一起。如果链中的任何依赖项发生变化，缓存中的数据将变得无效。

+   `CExpressionDependency`：如果指定的 PHP 表达式的结果发生了变化，则缓存中的数据将无效。

为了提供一个具体的例子，让我们使用一个依赖项，以便在`tbl_sys_message`数据库表发生更改时使缓存中的数据过期。我们将不再任意地在五分钟后使我们的缓存系统消息过期，而是在需要时精确地使其过期，也就是说，当表中的系统消息的`update_time`列发生更改时。我们将使用`CDbCacheDependency`实现这一点，因为它旨在根据 SQL 查询结果的更改来使缓存数据无效。

我们改变了对`set()`方法的调用，将持续时间设置为`0`，这样它就不会根据时间过期，而是传入一个新的依赖实例和我们指定的 SQL 语句，如下所示：

```php
$cache->set($key, $sysMessage, 0, new CDbCacheDependency('SELECT MAX(update_time) FROM tbl_sys_message'));
```

### 注意

将 TTL 时间更改为`0`并不是使用依赖的先决条件。我们可以将持续时间留在`300`秒。这只是规定了另一个规则，使缓存中的数据无效。数据在缓存中只有效 5 分钟，但如果表中有更新时间更晚的消息，也就是更新时间，数据也会在此时间限制之前重新生成。

有了这个设置，缓存只有在查询语句的结果发生变化时才会过期。这个例子有点牵强，因为最初我们是为了避免完全调用数据库而缓存数据。现在我们已经配置它，每次尝试从缓存中检索数据时都会执行数据库查询。然而，如果缓存的数据集更复杂，涉及更多的开销来检索和处理，一个简单的 SQL 语句来验证缓存的有效性可能是有意义的。具体的缓存实现、存储的数据、过期时间，以及这些依赖形式的任何其他数据验证，都将取决于正在构建的应用程序的具体要求。知道 Yii 有许多选项可用于满足我们多样化的需求是很好的。

## 查询缓存

查询缓存的方法在数据库驱动应用程序中经常需要，Yii 提供了更简单的实现，称为**查询缓存**。顾名思义，查询缓存将数据库查询的结果存储在缓存中，并在后续请求中节省查询执行时间，因为这些请求直接从缓存中提供。为了启用查询，您需要确保`CDbConnection`属性的`queryCacheID`属性引用有效缓存组件的`ID`属性。它默认引用`'cache'`，这就是我们从前面的缓存示例中已经配置的。

要使用查询缓存，我们只需调用`CDbConnection`的`cache()`方法。这个方法接受一个持续时间，用来指定查询在缓存中保留的秒数。如果持续时间设置为`0`，缓存就被禁用了。您还可以将`CCacheDependency`实例作为第二个参数传入，并指定多少个后续查询应该被缓存为第三个参数。这第三个参数默认为`1`，这意味着只有下一个 SQL 查询会被缓存。

因此，让我们将以前的缓存实现更改为使用这个很酷的查询缓存功能。使用查询缓存，我们的`SysMessage::getLatest()`方法的实现大大简化了。我们只需要做以下操作：

```php
    //use the query caching approach
    $dependency = new CDbCacheDependency('SELECT MAX(update_time) FROM tbl_sys_message');
    $sysMessage = SysMessage::model()->cache(1800, $dependency)->find(array(
      'order'=>'t.update_time DESC',
    ));
    return $sysMessage;
```

在这里，我们与以前的基本方法相同，但我们不必处理缓存值的显式检查和设置。我们调用`cache()`方法来指示我们要将结果缓存 30 分钟，或者通过指定依赖项，在此时间之前刷新值，如果有更近期的消息可用。

## 片段缓存

前面的例子演示了数据缓存的使用。这是我们将单个数据存储在缓存中。Yii 还提供了其他方法来存储视图脚本的一部分生成的页面片段，甚至整个页面本身。

片段缓存是指缓存页面的一部分。我们可以在视图脚本中利用片段缓存。为此，我们使用`CController::beginCache()`和`CController::endCache()`方法。这两种方法用于标记应该存储在缓存中的渲染页面内容的开始和结束。就像使用数据缓存方法时一样，我们需要一个唯一的键来标识被缓存的内容。一般来说，在视图脚本中使用片段缓存的语法如下：

```php
...some HTML content...
<?php
if($this->beginCache($id))
{
// ...content you want to cache here
$this->endCache();
}
?>
...other HTML content...
```

当有缓存版本可用时，`beginCache()`方法返回`false`，并且缓存的内容将自动插入到该位置；否则，if 语句内的内容将被执行，并且在调用`endCache()`时将被缓存。

### 声明片段缓存选项

在调用`beginCache()`时，我们可以提供一个数组作为第二个参数，其中包含定制片段缓存的缓存选项。事实上，`beginCache()`和`endCache()`方法是`COutputCache`过滤器/小部件的便捷包装。因此，缓存选项可以是`COutputCache`类的任何属性的初始值。

在缓存数据时，指定的最常见选项之一是持续时间，它指定内容在缓存中可以保持有效的时间。这类似于我们在缓存系统消息时使用的“持续时间”参数。在调用`beginCache()`时，可以指定`duration`参数如下：

```php
$this->beginCache($key, array('duration'=>3600))
```

这种片段缓存方法的默认设置与数据缓存的默认设置不同。如果我们不设置持续时间，它将默认为 60 秒，这意味着缓存的内容将在 60 秒后失效。在使用片段缓存时，您可以设置许多其他选项。有关更多信息，请参考`COutputCache`的 API 文档以及 Yii 权威指南的片段缓存部分，该指南可在 Yii 框架网站上找到：[`www.yiiframework.com/doc/guide/1.1/en/caching.fragment`](http://www.yiiframework.com/doc/guide/1.1/en/caching.fragment)

### 使用片段缓存

让我们在 TrackStar 应用程序中实现这一点。我们将再次专注于项目列表页面。您可能还记得，在项目列表页面的底部有一个列表，显示了用户在与每个项目相关的问题上留下的评论。这个列表只是指示谁在哪个问题上留下了评论。我们可以使用片段缓存来缓存这个列表，比如说两分钟。应用程序可以容忍这些数据略微过时，而两分钟对于等待更新的评论列表来说并不长。

为了做到这一点，我们需要对列表视图文件`protected/views/project/index.php`进行更改。我们将调用整个最近评论小部件的内容包裹在这个片段缓存方法中，如下所示：

```php
<?php
$key = "TrackStar.ProjectListing.RecentComments";
if($this->beginCache($key, array('duration'=>120))) {
   $this->beginWidget('zii.widgets.CPortlet', array(
    'title'=>'Recent Comments',
  ));  
  $this->widget('RecentCommentsWidget');
  $this->endWidget();
  $this->endCache(); 
}
?>
```

有了这个设置，如果我们第一次访问项目列表页面，我们的评论列表将被存储在缓存中。然后，如果我们在两分钟内快速（在两分钟之前）向项目中的问题之一添加新评论，然后切换回项目列表页面，我们不会立即看到新添加的评论。但是，如果我们不断刷新页面，一旦缓存中的内容过期（在这种情况下最多两分钟），数据将被刷新，我们的新评论将显示在列表中。

### 注意

您还可以简单地在先前缓存的内容中添加`echo time();` PHP 语句，以查看它是否按预期工作。如果内容正确缓存，时间显示将在缓存刷新之前不会更新。在使用文件缓存时，请记住确保您的`/protected/runtime/`目录对 Web 服务器进程是可写的，因为这是缓存内容默认存储的位置。

我们可以通过声明缓存依赖项而不是固定持续时间来避免这种情况。片段缓存也支持缓存依赖项。因此，我们可以将之前看到的`beginCache()`方法调用更改为以下内容：

```php
if($this->beginCache($key, array('dependency'=>array(
      'class'=>'system.caching.dependencies.CDbCacheDependency',
      'sql'=>'SELECT MAX(update_time) FROM tbl_comment')))) {
```

在这里，我们使用了`CDbCacheDependency`方法来缓存内容，直到对我们的评论表进行更新。

## 页面缓存

除了片段缓存之外，Yii 还提供了选项来缓存整个页面请求的结果。页面缓存方法类似于片段缓存方法。然而，由于整个页面的内容通常是通过将额外的布局应用于视图来生成的，我们不能简单地在布局文件中调用`beginCache()`和`endCache()`。原因是布局是在对`CController::render()`方法进行调用后应用的，内容视图被评估之后。因此，我们总是会错过从缓存中检索内容的机会。

因此，要缓存整个页面，我们应该完全跳过生成页面内容的操作执行。为了实现这一点，我们可以在控制器类中使用`COutputCache`类作为操作过滤器。

举个例子，让我们使用页面缓存方法来缓存每个项目详细页面的页面结果。TrackStar 中的项目详细页面是通过请求格式为`http://localhost/trackstar/project/view/id/[id]`的 URL 来呈现的，其中`[id]`是我们请求详细信息的特定项目 ID。我们要做的是设置一个页面缓存过滤器，将为每个请求的 ID 单独缓存此页面的整个内容。当我们缓存内容时，我们需要将项目 ID 合并到键值中。也就是说，我们不希望请求项目＃1 的详细信息，然后应用程序返回项目＃2 的缓存结果。`COutputCache`过滤器允许我们做到这一点。

打开`protected/controllers/ProjectController.php`并修改现有的`filters()`方法如下：

```php
public function filters()
{
  return array(
    'accessControl', // perform access control for CRUD operations
 **array(**
 **'COutputCache + view',  //cache the entire output from the actionView() method for 2 minutes**
 **'duration'=>120,**
 **'varyByParam'=>array('id'),**
 **),**
  );
}
```

此过滤器配置利用`COutputCache`过滤器来缓存应用程序从调用`ProjectController::actionView()`生成的整个输出。如您可能还记得的那样，在`COutputCache`声明之后添加的`+ view`参数是我们包括特定操作方法的标准方式，以便过滤器应用。持续时间参数指定了 120 秒（两分钟）的 TTL，之后页面内容将被重新生成。

`varyByParam`配置是一个非常好的选项，我们之前提到过。这个功能允许自动处理变化，而不是将责任放在开发人员身上，为被缓存的内容想出一个独特的键策略。例如，在这种情况下，通过指定与输入请求中的`GET`参数对应的名称列表。由于我们正在缓存按`project_id`请求的项目的页面内容，因此使用此 ID 作为缓存内容的唯一键生成的一部分是非常合理的。通过指定`'varyByParam'=>array('id')`，`COutputCache`会根据输入查询字符串参数`id`为我们执行此操作。在使用`COutputCache`缓存数据时，还有更多可用的选项来实现这种自动内容变化策略。截至 Yii 1.1.12，以下变化功能可用：

+   **varyByRoute**：通过将此选项设置为`true`，特定的请求路由将被合并到缓存数据的唯一标识符中。因此，您可以使用请求的控制器和操作的组合来区分缓存的内容。

+   **varyBySession**：通过将此选项设置为`true`，将使用唯一的会话 ID 来区分缓存中的内容。每个用户会话可能会看到不同的内容，但所有这些内容仍然可以从缓存中提供。

+   **varyByParam**：如前所述，这使用输入的`GET`查询字符串参数来区分缓存中的内容。

+   **varyByExpression**：通过将此选项设置为 PHP 表达式，我们可以使用此表达式的结果来区分缓存中的内容。

因此，在我们的`ProjectController`类中配置了上述过滤器，对于特定项目详细信息页面的每个请求，在重新生成并再次存储在缓存之前，都会在缓存中存储两分钟。您可以通过首先查看特定项目，然后以某种方式更新该项目来测试这一点。如果在两分钟的缓存持续时间内进行更新，您的更新将不会立即显示。

缓存整个页面结果是提高网站性能的好方法，但显然并不适用于每个应用程序中的每个页面。即使在我们的示例中，为项目详细信息页面缓存整个页面也不能正确使用分页实现我们的问题列表。我们使用这个作为一个快速示例来实现页面缓存，但并不总是适用于每种情况。数据、片段和页面缓存的结合允许您调整缓存策略以满足应用程序的要求。我们只是触及了 Yii 中所有可用缓存选项的表面。希望这激发了您进一步探索完整的缓存景观的兴趣。

# 一般性能调优提示

在准备应用程序投入生产时，还有一些其他事项需要考虑。以下部分简要概述了在调整基于 Yii 的 Web 应用程序性能时需要考虑的其他领域。

## 使用 APC

启用 PHP APC 扩展可能是改善应用程序整体性能的最简单方法。该扩展缓存和优化 PHP 中间代码，并避免在每个传入请求中解析 PHP 脚本所花费的时间。

它还为缓存内容提供了一个非常快速的存储机制。启用 APC 后，可以使用`CApcCache`实现来缓存内容、片段和页面。

## 禁用调试模式

我们在本章的前面讨论了调试模式，但再次提及也无妨。禁用调试模式是另一种提高性能和安全性的简单方法。如果在主`index.php`入口脚本中定义常量`YII_DEBUG`为`true`，Yii 应用程序将在调试模式下运行。许多组件，包括框架本身的组件，在调试模式下运行时会产生额外的开销。

另外，正如在第二章中提到的，*入门*，当我们第一次创建 Yii 应用程序时，大多数 Yii 应用程序文件不需要，也不应该放在公共可访问的 Web 目录中。Yii 应用程序只有一个入口脚本，通常是唯一需要放在 Web 目录中的文件。其他 PHP 脚本，包括所有 Yii 框架文件，都应该受到保护。这就是主应用程序目录的默认名称为`protected/`的原因。为了避免安全问题，建议不要公开访问它。

## 使用 yiilite.php

当启用 PHP APC 扩展时，可以用名为`yiilite.php`的不同 Yii 引导文件替换`yii.php`。这有助于进一步提高 Yii 应用程序的性能。`yiilite.php`文件随每个 Yii 版本发布。它是合并了一些常用的 Yii 类文件的结果。合并文件中删除了注释和跟踪语句。因此，使用`yiilite.php`将减少被包含的文件数量，并避免执行跟踪语句。

### 注意

请注意，没有 APC 的情况下使用`yiilite.php`可能会降低性能。这是因为`yiilite.php`包含一些不一定在每个请求中使用的类，并且会花费额外的解析时间。还观察到，在某些服务器配置下，即使启用了 APC，使用`yiilite.php`也会更慢。判断是否使用`yiilite.php`的最佳方法是使用代码包中包含的“Hello World”演示运行基准测试。

## 使用缓存技术

正如我们在本章中描述和演示的，Yii 提供了许多缓存解决方案，可以显著提高 Web 应用程序的性能。如果生成某些数据需要很长时间，我们可以使用数据缓存方法来减少数据生成的频率；如果页面的某部分保持相对静态，我们可以使用片段缓存方法来减少其渲染频率；如果整个页面保持相对静态，我们可以使用页面缓存方法来节省整个页面请求的渲染成本。

## 启用模式缓存

如果应用程序使用**Active Record**（**AR**），你可以在生产环境中启用模式缓存以节省解析数据库模式的时间。这可以通过将`CDbConnection::schemaCachingDuration`属性配置为大于零的值来实现。

除了这些应用程序级别的缓存技术，我们还可以使用服务器端缓存解决方案来提升应用程序的性能。我们在这里描述的 APC 缓存的启用属于这个范畴。还有其他服务器端技术，比如 Zend Optimizer、eAccelerator 和 Squid 等。

这些大部分只是在你准备将 Yii 应用程序投入生产或者为现有应用程序排除瓶颈时提供一些良好的实践指南。一般的应用程序性能调优更多的是一门艺术而不是科学，而且 Yii 框架之外有许多因素影响整体性能。Yii 自问世以来就考虑了性能，并且继续远远超过许多其他基于 PHP 的应用程序开发框架（详见[`www.yiiframework.com/performance/`](http://www.yiiframework.com/performance/)）。当然，每个 Web 应用程序都需要进行调整以增强性能，但选择 Yii 作为开发框架肯定会让你的应用程序从一开始就具备良好的性能基础。

有关更多详细信息，请参阅 Yii 权威指南中的*性能调优*部分[`www.yiiframework.com/doc/guide/1.1/en/topics.performance`](http://www.yiiframework.com/doc/guide/1.1/en/topics.performance)。

# 总结

在本章中，我们将注意力转向对应用程序进行更改，以帮助提高其在生产环境中的可维护性和性能。我们首先介绍了 Yii 中可用的应用程序日志记录策略，以及如何根据不同的严重级别和类别记录和路由消息。然后我们转向错误处理，以及 Yii 如何利用 PHP 5 中的基础异常实现来提供灵活和健壮的错误处理框架。然后我们了解了 Yii 中可用的一些不同的缓存策略。我们了解了在不同粒度级别上对应用程序数据和内容进行缓存的方法。对于特定变量或单个数据片段的数据缓存，对页面内的内容区域进行片段缓存，以及对整个渲染输出进行完整页面缓存。最后，我们提供了一系列在努力改善 Yii 驱动的 Web 应用程序性能时要遵循的良好实践。

恭喜！我们应该为自己鼓掌。我们已经从构思到生产准备阶段创建了一个完整的网络应用程序。当然，我们也应该为 Yii 鼓掌，因为它在每一个转折点都帮助我们简化和加快了这个过程。我们的 TrackStar 应用程序已经相当不错；但就像所有这类项目一样，总会有改进和提高的空间。我们已经奠定了一个良好的基础，现在你拥有 Yii 的力量，你可以很快将其转变为一个更加易用和功能丰富的应用程序。此外，许多涵盖的示例也可以很好地应用到你可能正在构建的其他类型的网络应用程序上。我希望你现在对使用 Yii 感到自信，并且会在未来的项目中享受到这样做的好处。开心开发！
