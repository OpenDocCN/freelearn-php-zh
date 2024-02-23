# 第九章。从单块到微服务

在本章中，我们将探讨在必须将单块应用程序转换为微服务时可以遵循的一些可能策略，以及一些示例。如果我们已经有一个庞大而复杂的应用程序，这个过程可能会有点困难，但幸运的是，有一些众所周知的策略可以遵循，以避免在整个过程中出现问题。

# 重构策略

将单块应用程序转换为微服务的过程是为了重构代码，以使您的应用程序现代化。这应该是逐步进行的。试图一步将整个应用程序转换为微服务可能会导致问题。逐渐地，它将创建一个基于微服务的新应用程序，最终，您当前的应用程序将消失，因为它将被转换为小的微服务，使原始应用程序变为空或者也可能成为一个微服务。

## 停止潜水

当您的应用程序已经是一个洞时，您必须停止潜水。换句话说，停止让您的单块应用程序变得更大。这是当您必须实现一个新功能时，需要创建一个新的微服务并将其连接到单块应用程序，而不是继续在单块中开发新功能。

为此，当实现新功能时，我们将有当前的单块、新功能，以及另外两个东西：

+   **路由器：**这负责处理 HTTP 请求；换句话说，这就像一个网关，知道需要将每个请求发送到哪里，无论是旧的单体还是新功能。

+   **粘合代码：**这负责将单块应用程序连接到新功能。新功能通常需要访问单块应用程序以获取数据或任何必要的功能：![停止潜水](img/B06142_09_01.jpg)

停止潜水策略

关于粘合代码，有 3 种不同的可能性可以从新功能访问应用程序到单块：

+   在单体侧创建一个 API，供新功能使用

+   直接连接单块数据库

+   在功能方面有一个与单块数据库同步的副本

正如您所看到的，这种策略是在当前的单块应用程序中开始开发微服务的一种不错的方式。此外，新功能可以独立于单块进行扩展、部署和开发，从而改进您的应用程序。然而，这并不能解决问题，只是避免让当前问题变得更大，所以让我们再看看另外两种策略。

## 分离前端和后端

另一种策略是将逻辑呈现部分与数据访问层分开。一个应用程序通常至少有 3 个不同的部分：

+   **呈现层：**这是用户界面，换句话说，是网站的 HTML 语言

+   **业务逻辑层：**这由用于实现业务规则的组件组成

+   **访问数据层：**这包含访问数据库的组件

通常呈现层和业务逻辑以及访问数据层之间有一个分离。业务层具有一个 API，其中有一个或多个封装业务逻辑组件的门面。从这个 API，可以将单块分成 2 个较小的应用程序。

分割后，呈现层调用业务逻辑。看下面的例子：

![分离前端和后端](img/B06142_09_02.jpg)

分离前端和后端策略

这种分割有两个不同的优势：

+   它允许您扩展和开发两个不同和独立的应用程序

+   它为您提供了一个可以供未来微服务使用的 API

这种策略的问题在于它只是一个临时解决方案，它可以转变为一个或两个单体应用程序，因此让我们看看下一个策略，以便移除剩余的单体应用程序。

## 提取服务

最后的策略是从结果单体应用程序中隔离模块。我们可以逐步从中提取模块，并从每个模块创建一个微服务。一旦我们提取了所有重要的模块，结果单体应用程序也将成为一个微服务，甚至可能消失。总体思路是创建将成为未来微服务的功能的逻辑组。

单体应用程序通常有许多潜在的模块可以提取。优先级必须通过首先选择更容易的模块，然后选择最有益的模块来设置。更容易的模块将为您提供必要的经验，以将模块提取为微服务，以便稍后处理重要的模块。

以下是一些提示，以帮助您选择最有益的模块：

+   经常变化的模块

+   需要与单体应用程序不同资源的模块

+   需要昂贵硬件的模块

寻找现有的粗粒度��界是有用的，它们更容易且更便宜转换为微服务。

### 如何提取模块

现在，让我们看看如何提取一个模块，我们将使用一个示例来使解释更容易理解一些。想象一下，您的单体应用是一个博客系统。正如您可以想象的那样，核心功能是用户创建的帖子，每个帖子都支持评论。从我们的简要描述中可以看出，您可以定义应用程序的不同模块，并决定哪个最重要。

一旦您清楚了应用程序的描述和功能，就可以继续使用用于从单体应用程序中提取模块的一般步骤：

1.  在模块和单体代码之间创建一个接口。最佳解决方案是双向 API，因为单体应用程序将需要来自模块的数据，而模块将需要来自单体应用程序的数据。这个过程并不容易，您可能需要更改单体应用程序的代码，以使 API 正常工作。

1.  一旦实现了粗粒度接口，将模块转换为独立的微服务。

例如，想象一下`POST`模块是要被提取的候选模块，它们的组件被`Users`和`Comments`模块使用。正如第一步所说，需要实现一个粗粒度的 API。第一个接口是由`Users`用来调用`POST`模块的入口 API，第二个接口是由`POST`用来调用`Comments`的。

在提取的第二步中，将模块转换为独立的微服务。一旦完成，生成的微服务将是可扩展和独立的，因此可以使其增长，甚至从头开始编写。

逐步，单体应用程序将变得更小，您的应用程序将拥有更多的微服务。

# 教程：从单体应用到微服务

在本章的示例中，我们将不使用框架，并且将不使用 MVC 架构编写代码，以便专注于本章的主题，并学习如何将单体应用程序转换为微服务。

没有比实践更好的学习方法了，因此让我们看一个完整的博客平台示例，这是我们在前一节中定义的。

### 提示

博客平台示例可以从我们的 PHP 微服务存储库中下载，因此，如果您想跟随我们的步骤，可以通过下载并按照本指南进行操作。

我们的示例是一个基本的博客平台，具有最低限度的功能，可以通过本教程。这是一个允许以下操作的博客系统：

+   注册新用户

+   用户登录

+   管理员可以发布新文章

+   注册用户可以发布新评论

+   管理员可以创建新类别

+   管理员可以创建新文章

+   管理员可以管理评论

+   所有用户都可以看到文章

因此，将单块应用程序转换为微服务的第一步是熟悉当前应用程序。在我们的想象中，当前应用程序可以分为以下微服务：

+   用户

+   文章

+   评论

+   类别

在这个例子中很清楚，但在一个真实的例子中，应该深入研究，以便按照我们在本章中之前解释的优先级将项目划分为小的微服务，这些微服务将通过执行特定功能来完成任务。

## 停止潜水

现在我们知道如何按照之前解释的策略进行操作，想象一下我们想要在我们的博客平台中添加一个新功能，即在用户之间发送私人消息。

为了弄清楚这一点，我们需要知道新的发送私人消息功能将具有哪些功能，以便找到粘合代码和从新微服务获取信息（路由）的请求所在的位置。

因此，新微服务的功能可以如下：

+   向用户发送消息

+   阅读你的消息

正如你所看到的，这些功能非常基本，但请记住，这只是为了让你熟悉在单块应用程序中创建新微服务的过程。

我们将创建私人消息微服务，并且当然，我们将再次使用 Lumen。为了快速创建骨架，在终端上运行以下命令：

```php
**composer create-project --prefer-dist laravel/lumen private_messages**

```

上述命令将创建一个带有 Lumen 安装的文件夹。

在第二章中，我们解释了如何创建 Docker 容器。现在，你有机会运用你学到的一切，在 Docker 环境中实现单块和不同的新微服务。根据前面的章节，你应该能够自己做到这一点。

我们的新功能需要一个存储私人消息的地方，所以现在我们将创建私人消息微服务将使用的表。这可以在单独的数据库中创建，甚至可以在同一个应用程序的数据库中创建。请记住，如果情况允许，微服务可以共享同一个数据库，但想象一下，这个微服务将会有很多流量，所以对我们来说，最好的解决方案是将其放在一个单独的数据库中。

创建一个新的数据库或连接应用程序数据库并执行以下查询：

```php
    CREATE TABLE `messages` (
      `id` INT NOT NULL AUTO_INCREMENT,
      `sender_id` INT NULL,
      `recipient_id` INT NULL,
      `message` TEXT NULL,
      PRIMARY KEY (`id`));
```

一旦我们创建了表，就需要将新的微服务连接到它，所以打开`.env.example`文件并修改以下行：

```php
    DB_CONNECTION=mysql
    DB_HOST=localhost
    DB_PORT=3306
    DB_DATABASE=**private_messages**
    DB_USERNAME=root
    DB_PASSWORD=root
```

如果你的数据库不同，请在上述代码中进行更改。

将`.env.example`文件重命名为`.env`，并将`$app->run();`代码更改为`public/index.php`文件中的以下代码；这将允许你调用这个微服务：

```php
    $app->run(
      $app->make('request')
    );
```

现在，你可以通过在 Postman 上进行 GET 调用`http://localhost/private_messages/public/`来检查你的微服务是否正常工作。记得对你的开发基础设施进行所有必要的更改。

你将收到一个带有安装的 Lumen 版本的 200 状态码。

在我们的微服务中，我们至少需要包括以下调用：

+   GET `/messages/user/id`：这是获取用户的消息所需的

+   POST `/message/sender/id/recipient/id`：这是向用户发送消息所需的

因此，现在我们将在`/private_messages/app/Http/routes.php`上创建路由，通过在`routes.php`文件的末尾添加以下行：

```php
    $app->get('messages/user/{userId}',
              'MessageController@getUserMessages');
    $app->post('messages/sender/{senderId}/recipient/{recipientId}',
               'MessageController@sendMessage');
```

下一步是在`/app/Http/Controllers/MessageController.php`上创建一个名为`MessageController`的控制器，并包含以下内容：

```php
    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\Request;

    class MessageController extends Controller
    {
      public function getUserMessages(Request $request, $userId) {
        // getUserMessages code
      }

      public function sendMessage(Request $request, $senderId,
                                  $recipientId) {
        // sendMessage code
      }
    }
```

现在，我们必须告诉 Lumen 需要使用数据库，所以取消注释`/bootstrap/app.php`中的以下行：

```php
    $app->withFacades();
    $app->withEloquent();
```

现在，我们可以创建这两个功能：

```php
    public function getUserMessages(Request $request, $userId) 
    {
 **$messages = Message::where('recipient_id',$userId)->get();**
 **return response()->json($messages);**
    }

    public function sendMessage(Request $request, $senderId,
                                $recipientId) 
    {
      **$message = new Message();**
 **$message->fill([
        'sender_id'    => $senderId,
        'recipient_id' => $recipientId,
        'message'      => $request->input('message')]);**
 **$message->save();**
 **return response()->json([]);**
    }
```

一旦我们的方法完成，微服务就完成了。因此，现在我们必须将单块应用程序连接到私人消息微服务。

我们需要在单体应用程序的`header.php`文件中为注册用户创建一个新按钮：

```php
    <?php if (empty($arrUser['username'])) : ?>
      <li role="presentation"><a href="login.php">Log in</a></li>
      <li role="presentation"><a href="signup.php">Sign up</a></li>
    <?php else : ?>
      <?php if ($arrUser['type'] === 'admin') : ?>
        <li role="presentation">
          <a href="admin/index.php">Admin Panel</a>
        </li>
      <?php endif; ?>
 **<li role="presentation">
        <a href="messages.php">Messages</a>
      </li>**
      <li role="presentation">
        <a href="index.php?logout=true">Log out</a>
      </li>
    <?php endif; ?>
```

然后，我们需要在`root`文件夹中创建一个名为`messages.php`的新文件，其中包含以下代码：

```php
    <?
      include_once 'libraries.php';

      $url = "http://localhost/private_messages/public/messages
              /user/".$arrUser['id'];
      $ch = curl_init();
      curl_setopt($ch, CURLOPT_SSL_VERIFYPEER, false);
      curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
      curl_setopt($ch, CURLOPT_URL,$url);
      $result=curl_exec($ch);
      curl_close($ch);
      $messages = json_decode($result, true);
```

正如你所看到的，我们正在调用微服务以获取用户消息列表。此外，我们需要获取用户列表以填充发送消息的用户选择器。这段代码可以被视为粘合代码，因为需要将微服务数据与单体数据匹配。我们将使用以下粘合代码：

```php
    $arrUsers = array();
    $query = "SELECT id, username FROM `users` ORDER BY username ASC";
    $result = mysql_query ($query, $dbConn);
    while ( $row = mysql_fetch_assoc ($result)) {
      array_push( $arrUsers,$row );
    }
```

现在我们可以构建 HTML 代码来显示用户消息和发送消息所需的表单：

```php
 **include_once 'header.php';
    ?>**
    <p class="bg-success">
 **<?php if ($_GET['sent']) { ?>
        The message was sent!
      <?php } ?>**
    </p>
    <h1>Messages</h1>
    <?php foreach($messages as $message) { ?>
      <div class="panel panel-primary">
        <div class="panel-heading">
          <h3 class="panel-title">
            Message from **<?php echo $arrUsers[$message['sender_id']]
                           ['username'];?>**           </h3>
        </div>
        <div class="panel-body">
 **<?php echo $message['message']; ?>**        </div>
      </div>
 **<?php } ?>**    <h1>Send message</h1>
      <div>
        <form action="messages.php" method="post">
          <div class="form-group">
            <label for="category_id">Recipient</label>
            <select class="form-control" name="recipient">
              <option value="">Select User</option>
              <option value="">------------------------</option>
 **<?php foreach($arrUsers as $user) { ?>**                <option value="**<?php echo $user['id']; ?>**">
 **<?php echo $user['username']; ?>**
                </option>
 **<?php } ?>**            </select>
          </div>
          <div class="form-group">
            <label for="name">
              Message
            </label>
            <br />
            <input name="message" type="text" value=""
             class="form-control" />
          </div>
          <input name="sender" type="hidden" 
           value="<?php echo $arrUser['id']; ?>" />

          <div class="form-group">
            <input name="submit" type="submit" value="Send message" 
             class="btn btn-primary" />
          </div>

        </form>
      </div>
 **<?php include_once 'footer.php'; ?>**

```

请注意，有一个用于发送消息的表单，因此我们必须添加一些代码来调用微服务以发送消息。在`$messages = json_decode($result, true);`行后添加以下代码：

```php
    if (!empty($_POST['submit'])) {
      if (!empty($_POST['sender'])) {
        $sender = $_POST['sender'];
      }
      if (!empty($_POST['recipient'])) {
        $recipient = $_POST['recipient'];
      }
      if (!empty($_POST['message'])) {
        $message = $_POST['message'];
      }

      if (empty($sender)) {
        $error['sender'] = 'Sender not found';
      }
      if (empty($recipient)) {
        $error['recipient'] = 'Please select a recipient';
      }
      if (empty($message)) {
        $error['message'] = 'Please complete the message';
      }

      if (empty($error)) {
        $url = 'http://localhost/private_messages/public/messages
                /sender/'.$sender.'/recipient/'.$recipient;

        $handler = curl_init();
        curl_setopt($handler, CURLOPT_URL, $url);
        curl_setopt($handler, CURLOPT_POST,true);
        curl_setopt($handler, CURLOPT_RETURNTRANSFER,true);
        curl_setopt($handler, CURLOPT_POSTFIELDS, "message=".$message);
        $response = curl_exec ($handler);

        curl_close($handler);

        header( 'Location: messages.php?sent=true' );
        die;
      }
    }
```

就是这样。我们的第一个微服务已经包含在单体应用程序中。这就是在当前的单体应用程序中添加新功能时的操作方式。

## 前端和后端分离

如前所述，第二种策略是将表示层与业务逻辑隔离开来。这可以通过创建一个包含所有业务逻辑和数据访问的完整微服务来实现，也可以通过将表示层与业务层隔离开来实现，就像**模型-视图-控制器**（**MVC**）结构一样。

这并不是使用单体应用程序的问题的完整解决方案，因为这将导致我们有两个单体应用程序，而不是一个。

要做到这一点，我们应该首先在`root`文件夹中创建一个新的`Controller.php`文件。我们可以将这个类称为`Controller`，它将包含视图需要的所有方法。例如，`Article`视图需要`getArticle`，`postComment`和`getArticleComments`：

```php
    <?php

    class Controller
    {
      public function connect () {
        $db_con = mysql_pconnect(DB_SERVER,DB_USER,DB_PASS);
        if (!$db_con) {
          return false;
        }
        if (!mysql_select_db(DB_NAME, $db_con)) {
          return false;
        }
        return $db_con;
      }

      public function **getArticle**($id)
      {
        $dbConn = $this->connect();

        $query = "SELECT articles.id, articles.title, articles.extract,
                         articles.text, articles.updated_at, 
                         categories.value as category,
                         users.username FROM `articles`
                  INNER JOIN 
                  `categories` ON categories.id = articles.category_id
                  INNER JOIN
                  `users` ON users.id = articles.user_id
                  WHERE articles.id = " . $id . " LIMIT 1";
        $result = mysql_query ($query, $dbConn);
        return mysql_fetch_assoc ($result);
      }

      public function **getArticleComments**($id)
      {
        $dbConn = $this->connect();

        $arrComments = array();
        $query = "SELECT comments.id, comments.comment, users.username
                  FROM `comments` INNER JOIN `users`
                  ON comments.user_id = users.id
                  WHERE comments.status = 'valid'
                  AND comments.article_id = " . $id . "
                  ORDER BY comments.id DESC";

        $result = mysql_query ($query, $dbConn);
        while ($row = mysql_fetch_assoc($result)) {
          array_push($arrComments,$row);
        }

        return $arrComments;
      }

      public function **postComment**($comment,$user_id,$article_id)
      {
        $dbConn = $this->connect();

        $query = "INSERT INTO `comments` (comment, user_id, article_id)
                  VALUES ('$comment','$user_id','$article_id')";
        mysql_query($query, $dbConn);
      }
    }
```

文章视图应包含`Controller.php`文件中包含的方法。看一下以下代码：

```php
    <?
      include_once 'libraries.php';
 **include_once 'Controller.php';**
 **$controller = new Controller();**

      if ( !empty($_POST['submit']) ) {

        // Validation
        if (!empty($_POST['comment'])) {
          $comment = $_POST['comment'];
        }
        if (!empty($_GET['id'])) {
          $article_id = $_GET['id'];
        }
        if (!empty($arrUser['id'])) {
          $user_id = $arrUser['id'];
        }

        if (empty($comment)) {
          $error['comment'] = true;
        }
        if (empty($article_id)) {
          $error['article_id'] = true;
        }
        if (empty($user_id)) {
          $error['user_id'] = true;
        }
        if ( empty($error) ) {

 **$controller->postComment($comment,$user_id,$article_id);**
          header ( 'Location: article.php?id='.$article_id);
          die;
        }
      }

 **$article = $controller->getArticle($_GET['id']);**
 **$comments = $controller->getArticleComments($_GET['id']);**

      include_once 'header.php';
    ?>

    <h1>Article</h1>

    <div class="panel panel-primary">
      <div class="panel-heading">
        <h3 class="panel-title">
 **<?php echo $article['title']; ?>**
        </h3>
      </div>
      <div class="panel-body">
        <span class="label label-primary">Published</span> by
        <b>**<?php echo $article['username']; ?>**</b> in
        <i>**<?php echo $article['category']; ?>**</i>
        <b>**<?php echo date_format(date_create($article
                                              ['fModificacion']),
                                  'd/m/y h:m'); ?>**         </b>
        <hr/>
 **<?php echo $article['text']; ?>**      </div>
    </div>

    <h2>Comments</h2>
 **<?php foreach ($comments as $comment) { ?>** 
      <div class="panel panel-warning">
        <div class="panel-heading">
          <h3 class="panel-title">**<?php echo $comment['username']; ?>**                                   said</h3>
        </div>
        <div class="panel-body">
 **<?php echo $comment['comment']; ?>**        </div>
      </div>
 **<?php } ?>** 
    <div>
 **<?php if ( !empty( $arrUser ) ) { ?>** 
        <form action="article.php?id=**<?php echo $_GET['id']; ?>**"
              method="post">
          <div class="form-group">
            <label for="user">Post a comment</label>
            <textarea class="form-control" rows="3" cols="50"
                      name="comment" id="comment"> 
            </textarea>
          </div>

          <div class="form-group">
            <input name="submit" type="submit" value="Send"
                   class="btn btn-primary" />
          </div>
        </form>

 **<?php } else { ?>**        <p>
          Please sign up to leave a comment on this article. 
          <a href="signup.php">Sign up</a> or
          <a href="login.php">Log in</a>
        </p>
 **<?php } ?>**    </div>

    <?php include_once 'footer.php'; ?>
```

这些是我们应该遵循的步骤，以便将业务逻辑层与视图隔离开来。

### 提示

如果你愿意，可以将所有视图文件（`header.php`，`footer.php`，`index.php`，`article.php`等）放入一个名为`views`的文件夹中，以便在同一个文件夹中组织它们。

一旦我们将所有视图与业务逻辑隔离开来，我们将在控制器中包含所有方法，而不是在表示层中包含它们。正如我们之前所说，这只是一个临时解决方案，因此我们将寻找真正的解决方案，以将模块提取为微服务。

## 提取服务

在这种策略中，我们必须选择要隔离的第一个模块，以便从中制作一个微服务。在这种情况下，我们将从`Categories`模块开始做。

类别在管理员面板上使用最多。可以在其中创建、修改和删除类别，然后在创建新文章时选择它们，并在文章中显示以指示文章类别。

提取过程并不容易；我们必须确保我们知道模块被使用的所有地方。为了做到这一点，我们将创建一个双向 API 或在控制器中创建所有类别方法，然后将它们隔离在一个微服务中。

打开`admin/categories.php`文件，我们必须做与前端和后端分离相同的事情--找到所有引用类别的地方，并在控制器上创建一个新方法。看一下这个：

```php
    <?

      session_start ();

      require_once 'config.php';
      require_once 'connection.php';
      require_once 'isUser.php';

 **include_once '../Controller.php';**
 **$controller = new Controller();**
      $dbConn = connect();

      /* Omitted code */

      if (!empty($_GET['del'])) {
 **$controller->deleteCategory($_GET['del']);**

        header( 'Location: categories.php?dele=true' );
        die;
      }

      if (!empty($_POST['submit'])) {
        if (!empty($_POST['name'])) {
          $name = $_POST['name'];
        }
        if (empty($name)) {
          $error['name'] = 'Please enter category name';
        }
        if (empty($error)) {
 **$controller->createCategory($name);**
          header( 'Location: categories.php?add=true' );
          die;
        }
      }

      if (!empty($_POST['submitEdit'])) {

        /* Ommited code */

        if (empty($error)) {
 **$controller->updateCategory($id, $name);**
          header( 'Location: categories.php?edit=true' );
          die;
        }
      }

 **$arrCategories = $controller->getCategories();**

      if (!empty($_GET['id'])) {
 **$row = $controller->getCategory($_GET['id']);**
      }

      include_once 'header.php';

    ?>
    <!-- Omitted HTML code -->

```

`controller.php`文件必须包含类别方法：

```php
    public function createCategory($name)
    {
      $dbConn = $this->connect();

      $query = "INSERT INTO `categories` (value) VALUES ('$name')";
      mysql_query($query, $dbConn);
    }

    public function updateCategory($id,$name)
    {
      $dbConn = $this->connect();

      $query = "UPDATE `categories` set value = '$name'
                WHERE id = $id";
      mysql_query($query, $dbConn);
    }

    public function deleteCategory($id)
    {
      $dbConn = $this->connect();

      $query = "DELETE FROM `categories` WHERE id = ".$id;
      mysql_query($query, $dbConn);
    }

    public function getCategories()
    {
      $dbConn = $this->connect();

      $arrCategories = array();

      $query = "SELECT id, value FROM `categories` ORDER BY value ASC";
      $resultado = mysql_query ($query, $dbConn);

      while ($row = mysql_fetch_assoc ($resultado)) {
        array_push( $arrCategories,$row );
      }

      return $arrCategories;
    }

    public function getCategory($id)
    {
      $dbConn = $this->connect();

      $query = "SELECT id, value FROM `categories` WHERE id = ".$id;
      $resultado = mysql_query ($query, $dbConn);
      return mysql_fetch_assoc ($resultado);
    }
```

在`admin/articles.php`文件中有更多关于类别的引用，因此打开它并在`require_once`行后添加以下行：

```php
    include_once '../Controller.php';
    $controller = new Controller();
```

这些行将允许您在`articles.php`文件中使用`controller.php`文件中包含的类别方法。将用于获取类别的代码修改为以下代码：

```php
    $arrCategories = $controller->getCategories();
```

最后，需要对文章视图进行一些更改。这是用于显示文章的视图，并且在创建文章时包含所选的类别。

要获取文章，执行的查询如下：

```php
    SELECT articles.id, articles.title, articles.extract,
           articles.text, articles.updated_at,
           categories.value as category,
           users.username FROM `articles`
 **INNER JOIN `categories` ON categories.id = articles.category_id** 
    INNER JOIN `users` ON users.id = articles.user_id 
    WHERE articles.id = " . $id . " LIMIT 1;
```

如您所见，查询需要类别表。如果要为类别微服务使用不同的数据库，您将需要从查询中删除突出显示的行，选择查询中的`articles.category_id`，然后使用创建的方法获取类别名称。因此，查询将如下所示：

```php
    SELECT articles.id, articles.title, articles.extract,
           articles.text, articles.updated_at, articles.category_id,
           users.username FROM `articles` 
    INNER JOIN `users` ON users.id = articles.user_id 
    WHERE articles.id = " . $id . " LIMIT 1;
```

以下是从提供的类别 ID 获取类别名称的代码：

```php
    public function getArticle($id)
    {
      $dbConn = $this->connect();

      $query = "SELECT articles.id, articles.title, articles.extract,
                       articles.text, articles.updated_at,
                       articles.category_id,
                       users.username FROM `articles`
                INNER JOIN `users` ON users.id = articles.user_id
                WHERE articles.id = " . $id . " LIMIT 1;";

      $result = mysql_query ($query, $dbConn);

      $data = mysql_fetch_assoc($result);
 **$data['category'] = $this->getCategory($data['category_id']);**
 **$data['category'] = $data['category']['value'];**
      return $data;
    }
```

一旦我们进行了所有这些更改，就可以将类别表隔离到不同的数据库中，因此我们可以从`controller.php`文件中创建的方法创建一个类别微服务：

+   `public function createCategory($name)`

+   `public function updateCategory($id,$name)`

+   `public function deleteCategory($id)`

+   `public function getCategories()`

+   `public function getCategory($id)`

正如您所想象的那样，这些函数用于创建类别微服务的`routes.php`文件。因此，让我们创建一个新的微服务，就像我们使用停止潜水策略一样。

通过执行以下命令创建新的类别微服务：

```php
**composer create-project --prefer-dist laravel/lumen categories**

```

上述命令将在名为 categories 的文件夹中安装 Lumen，因此我们可以开始创建新类别微服务的代码。

现在我们有两个选项--第一个是使用位于当前数据库上的相同表--我们可以将新微服务指向当前数据库。第二个选项是在新数据库中创建一个新表，因此新微服务将使用自己的数据库。

如果要在新数据库中创建一个新表，可以按照以下步骤进行：

+   将当前类别表导出到 SQL 文件中。它将保留当前存储的数据。

+   将 SQL 文件导入新数据库。它将在新数据库中创建导出的表和数据。

### 提示

导出/导入过程可以使用 SQL 客户端执行，也可以通过控制台执行`mysqldump`。

一旦新表被导入到新数据库，或者您决定使用当前数据库，就需要设置`.env.example`文件，以便将新微服务连接到正确的数据库，因此打开它并在其中放入正确的参数：

```php
    DB_CONNECTION=mysql
    DB_HOST=localhost
    DB_PORT=3306
    DB_DATABASE=**categories**
    DB_USERNAME=root
    DB_PASSWORD=root
```

不要忘记将`.env.example`文件重命名为`.env`，并在`public/index.php`中更改`$app->run()`行，就像我们之前所做的那样，更改为以下代码：

```php
 **$app->run(**
 **$app->make('request')**
 **);**

```

还要取消注释`/bootstrap/app.php`中的以下行：

```php
    $app->withFacades();
    $app->withEloquent();
```

现在我们准备在`routes.php`文件中添加必要的方法。我们必须添加单体应用程序`Controller.php`中的类别方法，并将它们转换为路由：

+   `public function createCategory($name)`: 这是用于创建新类别的 POST 方法。因此，它可能类似于`$app->post('category', 'CategoryController@createCategory');`。

+   `public function updateCategory($id,$name)`: 这是一个 PUT 方法，用于编辑现有的类别。因此，它可能类似于`app->put('category/{id}', 'CategoryController@updateCategory');`。

+   `public function deleteCategory($id)`: 这是用于删除现有类别的 DELETE 方法。因此，它可能类似于`app->delete('category/{id}', 'CategoryController@deleteCategory');`。

+   `public function getCategories()`: 这是用于获取所有现有类别的 GET 方法。因此，它可能类似于`app->get('categories', 'CategoryController@getCategories');`。

+   `public function getCategory($id)`: 这也是一个 GET 方法，但它只获取一个单一的类别。因此，它可能类似于`app->get('category/{id}', 'CategoryController@getCategory');`。

因此，一旦我们在`routes.php`文件中添加了所有路由，就是创建类别模型的时候了。要做到这一点，在`/app/Model`中创建一个新文件夹和一个文件`/app/Model/Category.php`，就像这样：

```php
    <?php

      namespace App\Model;

      use Illuminate\Database\Eloquent\Model;

      class Category extends Model {
        protected $table = 'categories';
        protected $fillable = ['value'];
        public $timestamps = false;
      }
```

创建模型后，创建一个`/app/Http/Controllers/CategoryController.php`文件，其中包含必要的方法：

```php
    <?php

      namespace App\Http\Controllers;

      use App\Model\Category;
      use Illuminate\Http\Request;

      class CategoryController extends Controller
      {
        public function **createCategory**(Request $request) {
          $category = new Category();
          $category->fill(['value' => $request->input('value')]);
          $category->save();

          return response()->json([]);
        }

        public function **updateCategory**(Request $request, $id) {
          $category = Category::find($id);
          $category->value = $request->input('value');
          $category->save();

          return response()->json([]);
        }

        public function **deleteCategory**(Request $request, $id) {
          $category = Category::find($id);
          $category->delete();

          return response()->json([]);
        }

        public function **getCategories**(Request $request) {
          $categories = Category::get();

          return $categories;
        }

        public function **getCategory**(Request $request, $id) {
          $category = Category::find($id);

          return $category;
        }
      }
```

现在，我们已经完成了我们的类别微服务。您可以在 Postman 中尝试一下，以检查所有方法是否有效。例如，可以通过 Postman 使用`http://localhost/categories/public/categories` URL 调用`getCategories`方法。

一旦我们创建了新的类别微服务并且正常工作，就是时候断开类别模块并将单体应用程序连接到微服务了。

返回到单体应用程序并查找所有对类别方法的引用。我们必须通过调用新的微服务来替换它们。我们将使用原生 curl 调用进行这些调用，但是您应该考虑使用 Guzzle 或类似的包，就像我们在之前的章节中所做的那样。

为此，首先我们应该在`Controller.php`文件中创建一个函数来进行调用。可以是这样的：

```php
    function call($url, $method, $field = null)
    {
      $ch = curl_init();
      if(($method == 'DELETE') || ($method == 'PUT')) {
        curl_setopt($ch, CURLOPT_CUSTOMREQUEST, $method);
      }
      curl_setopt($ch, CURLOPT_SSL_VERIFYPEER, false);
      curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
      curl_setopt($ch, CURLOPT_URL,$url);
      if ($method == 'POST') {
        curl_setopt($ch, CURLOPT_POST,true);
      }
      if($field) {
        curl_setopt($ch, CURLOPT_POSTFIELDS, 'value='.$field);
      }
      $result=curl_exec($ch);
      curl_close($ch);
      return json_decode($result, true);
    }
```

上述代码将用于在每次调用类别微服务时重用代码。

转到`/admin/categories.php`文件并用以下代码替换`controller->createCategory($name);`行：

```php
     $controller->call('http://localhost/categories/public/category', 
                       'POST', $name);
```

在上述代码中，您可以检查我们正在使用 POST 调用创建类别方法，并将值参数设置为`$name`变量。

在`/admin/categories.php`文件中，找到并用以下代码替换`controller->updateCategory($id, $name);`行：

```php
 $controller->call('http://localhost/categories/public/category/'.$id,
                   'PUT', $name);
```

在同一文件中，找到并用以下代码替换`$controller->deleteCategory($_GET['del']);`：

```php
    $controller->call('http://localhost/categories/public
                       /category/'.$_GET['del'], 'DELETE');
```

在同一文件中，再次以及在`/admin/articles.php`文件中，找到并用以下代码替换`$arrCategories = $controller->getCategories();`：

```php
$arrCategories = $controller->call('http://localhost/categories
                                    /public/categories', 'GET');
```

最后一个位于`/admin/categories.php`文件中。找到并用以下代码替换`row = $controller->getCategory($_GET['id']);`行：

```php
$row = $controller->call('http://localhost/categories
                          /public/category/'.$_GET['id'], 'GET');
```

一旦我们完成了在单体应用程序中用新的类别微服务调用替换所有类别方法，我们可以删除对单体类别模块的所有引用。

转到`Controller.php`文件并删除以下函数，因为它们不再需要，因为它们引用了单体类别模块：

+   `public function deleteCategory($id)`

+   `public function createCategory($name)`

+   `public function updateCategory($id,$name)`

+   `public function getCategories()`

+   `public function getCategory($id)`

最后，如果您为类别微服务创建了新的数据库，可以通过执行以下查询来删除位于单体应用程序数据库中的类别表：

```php
    DROP TABLE categories;
```

我们已经从单体应用程序中提取了类别服务。下一步将是选择另一个模块，再次按照相同步骤进行，并重复此过程，直到单体应用程序消失或成为微服务为止。

在第七章中，*安全*我们谈到了微服务中的安全性。为了练习您所学到的知识，请查看本章中的所有代码，并找出我们示例中的弱点。

# 摘要

在本章中，您学习了转换单体应用程序为微服务的策略，并为每个步骤使用了示例代码。从现在开始，您已经准备好告别单体应用程序，并进行转换，以开始使用微服务。
