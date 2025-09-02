# 第五章：使用 Phalcon 的特性

是时候给我们的博客添加一些特性，并在过程中了解更多关于 Phalcon 的功能了。为了使我们的博客达到预期效果，还有很多工作要做。用户的密码仍然是明文，任何人，无论是否是用户，都可以浏览到用户的页面并查看密码。因此，我们需要设置至少某种形式的安全性和用户访问控制。我们有一个评论表，但我们还没有任何评论的方式。而且，没有 RSS 源和 ping 到聚合器的功能，博客就不能称之为博客。

在本章中，我们将涵盖以下主题：

+   如何使用 Phalcon 哈希密码

+   如何使用视图辅助器和视图部分

+   如何设置 cookies

+   如何控制用户的访问

+   如何在我们的应用程序中缓存数据

+   如何使用 Phalcon 的事件管理器

+   如何路由应用程序请求

# 使用 Phalcon 哈希密码

首先，让我们至少将密码哈希化。哈希化是一个将密码转换为固定长度的位字符串或哈希的过程，该哈希不能被逆向工程以检索密码，但输入密码的任何变化都会改变生成的哈希。因此，为了更安全的应用程序，我们将从用户输入的密码生成一个哈希，并将该值存储在数据库中。然后，当用户再次尝试登录时，我们将从输入的密码生成一个哈希，并将其与数据库中哈希值的值进行比较。

幸运的是，这很容易用 Phalcon 做到。安全组件会自动加载到 Phalcon 服务的容器中。所以，打开`UsersController.php`文件，找到`createAction`函数；然后，找到设置密码的行。

```php
$user->password = $this->request->getPost("password");
```

然后，只需将其替换为以下两行代码：

```php
$password = $this->request->getPost("password");
$user->password = $this->security->hash($password);
```

现在，当我们创建用户时，密码将被哈希并保存到数据库中。如果我们编辑，我们还想将密码再次作为哈希保存。因此，我们在`saveAction`函数中找到相同的代码行，并将其替换为前面的两行代码以哈希密码。

现在，我们只需要编辑我们的`loginAction`函数。我们的初始函数有点笨拙。所以，我们将用一些更流畅的东西来替换它。新的`loginAction`函数将如下代码片段所示：

```php
public function loginAction() {

        if ($this->request->isPost()) {
            $username = $this->request->getPost('username');
            $password = $this->request->getPost('password');

            $user = Users::findFirstByUsername($username);
            if ($user && $this->security->checkHash($password, $user->password)) {
                $this->session->set("user_id", $user->id);
                $this->cookies->set('user_id', $user->id);
                $this->session->set('auth', array(
                    'id' => $user->id,
                    'name' => $user->name
                ));
                $this->flash->success("Welcome " . $user->name);
            }else{
                $this->flash->error("Username and Password combination not found");
            }
        }
        return $this->dispatcher->forward(
            array(
                "controller" => "posts",
                "action" => "index"
            )
        );
    }
```

我们不是在数据库中搜索用户，而是尝试使用`findFirstByFieldname`函数实例化用户，其中`Fieldname`可以替换为我们正在查询的数据库中的列。如果我们通过搜索用户名找到了用户，我们使用安全服务的`checkHash`函数来比较数据库中的哈希值与我们刚刚从输入的密码中生成的哈希值。如果匹配，我们就通过设置会话中的`user_id`变量和创建欢迎信息的过程。

Phalcon 安全组件使用 bcrypt 哈希算法，这为我们提供了高水平的安全性。该组件默认由 Phalcon 加载，但也可以手动配置以调整 bcrypt 算法的“缓慢”。你可以在[`docs.phalconphp.com/en/latest/reference/security.html`](http://docs.phalconphp.com/en/latest/reference/security.html)了解更多关于此组件的信息。

# 使用 Phalcon 视图助手

我们还可以通过将表单中的密码字段更改为真实密码字段来添加一些安全性。因此，在 `app/views` 目录下的 `users` 文件夹中找到以下文件：

+   `new.volt`

+   `index.volt`

+   `edit.volt`

在代码中找到以下行：

```php
{{ text_field("password", "size" : 30) }}
```

将前面的代码替换为以下行代码：

```php
{{ password_field("password", "size" : 30) }}
```

现在，当在字段中输入时，密码将被隐藏。

正如我们在第三章中学习的，*使用 Phalcon 模型、视图和控制器*，Volt 模板引擎是一个非常薄的包装器，它围绕 PHP 代码包装，Phalcon 将其编译成实际的 PHP 代码。要在 Volt 中调用 Phalcon 标签助手，我们只需使用函数的非驼峰式版本。

## 使用动态标题标签

让我们用一些视图助手替换我们编写的部分 HTML 标记。一个强大的视图助手是文档标题助手。目前，我们在主布局中有一个硬编码的 `title` 标签，它给每个页面都赋予相同的标题。因此，打开位于 `app/views` 目录下的 `index.volt` 文件，并找到以下行代码：

```php
<title>Phalcon Blog</title>
```

将前面的代码替换为以下行代码：

```php
{{ get_title() }}
```

现在，唯一的问题是我们的页面没有标题，因为我们还没有设置它。保持我们博客的标题在每一页上可能是一个好主意。我们可以硬编码它，或者打开位于 `app/controllers` 目录下的 `ControllerBase.php` 文件，并将以下函数添加到 `ControllerBase` 类中：

```php
public function initialize() {
        $this->tag->setTitle("Phalcon Blog");
}
```

现在，我们的全局标题又回来了。但我们要让每个博客文章的标题也出现在标题中。所以，如果我们有一个名为 Hello World 的博客文章，我们希望我们的标题是 Hello World: Phalcon Blog。这样做很容易。打开位于 `app/controllers` 目录下的 `PostController.php` 文件，并将以下行代码添加到 `showAction` 函数的末尾：

```php
$this->tag->prependTitle($post->title . " - ");
```

如果你访问博客文章页面，博客文章的标题会显示在博客标题之前。现在，如果你想的话，你可以在每个控制器中的每个操作中添加自己的标题。

## 设置文档类型

通过使用 Phalcon 设置文档类型，你影响它生成的所有带有标签助手的标签，使它们符合你选择的标准。我们将使用 HTML5。将以下行代码添加到位于 `app/controllers` 目录下的 `ControllerBase.php` 文件中的 `initialize` 函数中：

```php
$this->tag->setDoctype(\Phalcon\Tag::HTML5);
```

现在，文档类型将在每个控制器中全局设置。在 `app/views` 目录下的 `index.html` 文件顶部找到以下行代码：

```php
<!DOCTYPE html>
```

将前面的代码替换为以下行代码：

```php
{{ get_doctype() }}
```

现在，你可以确信 Phalcon 会输出你选择的任何标准的标签。要了解其他文档标准，你可以访问 [`docs.phalconphp.com/en/latest/reference/tags.html#document-type-of-content`](http://docs.phalconphp.com/en/latest/reference/tags.html#document-type-of-content)。

## 使用视图助手添加 JavaScript 和 CSS

我们最初只是在主布局文件的头部分硬编码了 CSS 和 JavaScript 链接。目前这没问题，但如果我们移动项目，我们可能需要更改这些链接。Phalcon 提供了脚本和 CSS 标签助手，这将使我们的工作变得容易一些。我们可以用以下代码片段替换 `app/views/index.phtml` 文件头部的 CSS 链接：

```php
{{ stylesheet_link("css/bootstrap/bootstrap.min.css") }}
{{ stylesheet_link("css/bootstrap/bootstrap-responsive.min.css") }}
```

我们使用以下代码行将 JavaScript 链接放在文件底部：

```php
{{ javascript_include("js/jquery.min.js") }}
{{ javascript_include("js/bootstrap.min.js") }}
```

我们将在阅读本章的过程中探索更多的视图助手。要了解更多关于 Phalcon 标签助手的信息，请访问 [`docs.phalconphp.com/en/latest/api/Phalcon_Tag.html`](http://docs.phalconphp.com/en/latest/api/Phalcon_Tag.html)。

# 设置 cookies

现在我们可以登录并拥有安全的密码，但每次浏览器关闭时，我们都需要再次登录。如果能够登录到我们的博客并至少保持几天登录状态会很好。是时候设置一个 cookie 了。

我们将在阅读本章的过程中改变我们做这件事的方式，但到目前为止，设置一个可以让我们保持登录状态的 cookie 非常简单。首先，我们需要设置一个加密密钥，因为我们在设置 cookie 之前和检索它时都需要解密。我们希望保持这种方式，但为了做到这一点，我们需要给它一个密钥。所以，去 [`randomkeygen.com/`](http://randomkeygen.com/) 生成一个随机的数字字符串。然后，打开 `app/config/config.ini` 文件，并在应用程序部分添加你的密钥，命名为 `encryptKey`，如下所示：

```php
[application]
controllersDir = /home/eristoddle/Dropbox/xampp/htdocs/phalconBlog/app/controllers/
modelsDir = /home/eristoddle/Dropbox/xampp/htdocs/phalconBlog/app/models/
viewsDir = /home/eristoddle/Dropbox/xampp/htdocs/phalconBlog/app/views/
pluginsDir = ../app/plugins/
libraryDir = ../app/library/
baseUri = /phalconBlog/
cacheDir = ../app/cache/
encryptKey = 2tx6]GD}532q4x_
```

然后，打开 `app/config/services.php` 文件，并在文件底部添加以下代码片段以设置加密密钥：

```php
$di->set(
    'crypt', function () use ($config) {
        $crypt = new Phalcon\Crypt();
        $crypt->setKey($config->application->encryptKey);
        return $crypt;
    }
);
```

现在，我们的 cookies 已经准备好了。它们之前是可用的，但如果我们使用了它们，它们会抛出一个关于需要加密密钥的错误。现在，在 `app/controllers` 目录下找到 `UsersController.php` 文件中的 `loginAction` 函数，并添加以下代码行：

```php
$this->cookies->set('user_id', $user->id);
```

在设置会话中的 `user_id` 变量后立即添加它：

```php
$this->session->set("user_id", $user->id);
```

现在，打开 `app/controllers/PostsController.php` 文件，并找到检查会话中 `user_id` 变量的 `createAction` 函数部分。现在，我们将首先检查 cookies，然后如果找到 `user_id` cookie，就在会话中设置 `user_id` 变量：

```php
if ($this->cookies->has('user_id')) {
            $this->session->set('user_id', $this->cookies->get('user_id'));
        }
        if ($this->session->has("user_id")) {
            return $this->dispatcher->forward(
                array(
                    "controller" => "users",
                    "action" => "search"
                )
            );
        }
```

关于将代码改造以添加 cookie 的所有工作就到这里了。要了解更多关于 Phalcon 中 cookie 管理的知识，请参阅[`docs.phalconphp.com/en/latest/reference/cookies.html`](http://docs.phalconphp.com/en/latest/reference/cookies.html)。

现在，我们应该能够注销并再次登录，关闭浏览器，仍然保持登录状态。然而，当涉及到控制用户时，这并不是我们真正想要的全部。我们希望能够无缝地控制他们访问的一切。目前，我们的应用程序存在漏洞，为了填补这些漏洞，我们需要复制并粘贴代码以在需要该功能的所有地方实现相同的`user_id`变量检查。幸运的是，我们不必这样做。Phalcon 提供了一个服务，帮助我们控制用户访问。

# 控制用户访问

**访问控制列表**（**ACL**）使用角色来控制对资源的访问。`Phalcon\Acl`为我们提供了这个功能。有了它，我们可以将角色分配给我们博客上不同类型的访客，然后使用这些角色在每个控制器中的每个操作上设置权限。对于我们的目的，我们只将创建两个角色：用户和访客。用户将简单地是一个已登录的访客，访客或任何人。我们只将允许访客执行以下操作：

+   查看索引页面：

    +   查看帖子索引页面

    +   对帖子进行评论

    +   当访客未登录时，查看用户索引页，这将是一个登录页面

+   登录

登录用户可以做任何事情。为了在我们的应用程序中使用`Phalcon\Acl`，我们将创建一个简单的 Phalcon 插件。Phalcon 开发者工具已经添加了我们的`plugins`文件夹，并在`config.ini`文件中创建了`pluginsDir`设置。然而，我们仍然需要将`plugins`目录添加到我们的加载器中。打开位于`app/config`的`loader.php`文件，并将`plugins`目录添加到您的加载器中。

```php
$loader->registerDirs(
    array(
        $config->application->controllersDir,
        $config->application->modelsDir,
        $config->application->pluginsDir
    )
)->register();
```

我们想要创建的是一个插件，它将连接到 Phalcon 的事件管理器。使用事件管理器，我们可以将监听器附加到各种 Phalcon 事件。在`app/plugins`文件夹中创建一个名为`Security.php`的新文件，并在文件顶部添加以下代码片段：

```php
<?php

use Phalcon\Events\Event,
    Phalcon\Mvc\User\Plugin,
    Phalcon\Mvc\Dispatcher,
    Phalcon\Acl;

class Security extends Plugin {

    public function __construct($dependencyInjector) {
        $this->_dependencyInjector = $dependencyInjector;
    }
```

我们扩展了`Phalcon\Plugin`。我们的构造函数必须接受一个依赖注入器实例。然后，我们需要创建一个函数，将我们的 ACL 添加到我们的持久变量中。我们将称之为`getAcl`。

```php
    public function getAcl() {
        if (!isset($this->persistent->acl)) {

            $acl = new Phalcon\Acl\Adapter\Memory();
            $acl->setDefaultAction(Phalcon\Acl::DENY);

            $roles = array(
                'users' => new Phalcon\Acl\Role('Users'),
                'guests' => new Phalcon\Acl\Role('Guests')
            );
            foreach ($roles as $role) {
                $acl->addRole($role);
            }

            $private = array(
                'comments' => array('index', 'edit', 'delete', 'save'),
                'posts' => array('new', 'edit', 'save', 'create', 'delete'),
                'users' => array('search', 'new', 'edit', 'save', 'create', 'delete', 'logout')
            );
            foreach ($private as $resource => $actions) {
                $acl->addResource(new Phalcon\Acl\Resource($resource), $actions);
            }

            $public = array(
                'index' => array('index'),
                'posts' => array('index', 'search', 'show', 'comment'),
                'users' => array('login', 'index')
            );
            foreach ($public as $resource => $actions) {
                $acl->addResource(new Phalcon\Acl\Resource($resource), $actions);
            }

            foreach ($roles as $role) {
                foreach ($public as $resource => $actions) {
                    foreach ($actions as $action) {
                        $acl->allow($role->getName(), $resource, $action);
                    }
                }
            }

            foreach ($private as $resource => $actions) {
                foreach ($actions as $action) {
                    $acl->allow('Users', $resource, $action);
                }
            }

            $this->persistent->acl = $acl;
        }

        return $this->persistent->acl;
    }
```

首先，代码检查我们是否已经有了 ACL 实例。如果没有，我们创建一个。然后，我们将`DefaultAction`函数设置为出于安全原因拒绝访问。接下来的代码段创建了角色，即用户和访客，在我们的 ACL 中。然后，我们将我们想要设置为私有的所有资源加载到一个变量中，并将它们全部添加为资源。我们将对想要设置为公共的资源做同样的事情。然后，我们遍历角色，为角色和公共资源提供访问权限。然后，我们遍历私有资源，为用户提供访问权限，但不为访客提供。最后，我们将我们的新 ACL 设置为持久性并返回它。

此函数只是构建我们的 ACL 结构。我们仍然需要实际控制访问，我们将通过在调度之前触发一个函数来实现这一点。在文件底部添加以下函数：

```php
    public function beforeDispatch(Event $event, Dispatcher $dispatcher) {

        $user = $this->session->get('user_id');
        if (!$user) {
            $role = 'Guests';
        } else {
            $role = 'Users';
        }

        $controller = $dispatcher->getControllerName();
        $action = $dispatcher->getActionName();
        $acl = $this->getAcl();

        $allowed = $acl->isAllowed($role, $controller, $action);
        if ($allowed != Acl::ALLOW) {
            $this->flash->error("You don't have access to this module");
            $dispatcher->forward(
                array(
                    'controller' => 'index',
                    'action' => 'index'
                )
            );
            return false;
        }

    }

}
```

此函数接受一个事件实例和一个调度程序实例。我们检查访问者是否已登录并设置他们的角色。然后，我们设置一个变量来存储我们的控制器名称和一个变量来存储操作名称。我们调用我们的`getAcl`函数来获取我们的访问控制列表（ACL），并且我们还调用`$acl->isAllowed()`，传递访问者的角色以及我们的控制器和操作名称。如果角色被允许访问当前资源，这将返回 true。如果用户不允许访问，我们将设置一个错误消息并将他们转发到我们博客的首页。

现在，我们需要连接到标准调度程序，将这个新的`Security`插件附加到我们的事件管理器，并将其设置为调度程序的事件管理器。打开位于`app/config`的`services.php`文件，并将以下代码片段添加到文件底部：

```php
$di->set('dispatcher', function() use ($di) {
    $eventsManager = $di->getShared('eventsManager');
    $security = new Security($di);
    $eventsManager->attach('dispatch', $security);
    $dispatcher = new Phalcon\Mvc\Dispatcher();
    $dispatcher->setEventsManager($eventsManager);
    return $dispatcher;
});
```

### 注意

要了解更多关于使用`Phalcon\Acl`的信息，请访问[`docs.phalconphp.com/en/latest/reference/acl.html`](http://docs.phalconphp.com/en/latest/reference/acl.html)。要了解更多关于 Phalcon 的事件管理器的信息，请访问[`docs.phalconphp.com/en/latest/reference/events.html`](http://docs.phalconphp.com/en/latest/reference/events.html)。

# 为我们的应用程序添加最后修饰

Phalcon 开发者工具可以提供很多帮助，但让我们面对现实，你将为你的应用程序编写的绝大多数代码都是从零开始的。我们的博客还没有评论或源，所以它实际上根本算不上一个博客。添加这些功能将是一项手动工作，但 Phalcon 使这变得容易。

## 添加评论

现在，让我们给访问者提供在帖子上发表评论的能力。之前，我们需要用户访问控制，以便我们可以将评论管理限制为已登录用户。但现在，我们需要一个地方来做这件事。我们将构建一个非常简单的评论管理系统。我们可以使用 Phalcon 开发者工具或 Phalcon 网络工具为我们生成 CRUD 脚手架，但在这个案例中，我们几乎会移除比留下的更多生成的代码，但我们可以使用我们生成的其他控制器和视图作为指南。

对于评论，我们需要添加一个表单来对帖子进行评论，并显示动作视图，我们还需要在每个帖子下显示已发布的评论。我们还需要一个地方来列出所有评论，以及一个表单来编辑它们并将它们设置为发布。

评论已经与帖子相关联。我们只需修改几个地方，使这种关系变得可用。打开位于`app/views/posts`的`show.volt`文件。在文件底部，添加以下代码片段：

```php
<div class="comments">
    <h3>Comments</h3>
    <div class="comment">
    {% for comments in post.comments if comments.publish == true %}
        <div>{{ comments.body }}</div>
        <div><a href="{{ comments.url }}">{{ comments.name }}</a></div>
    {% endfor %}
    </div>
    <h3>Leave a Comment</h3>
    <div>
        {{ form("posts/comment", "method":"post") }}
        {{ hidden_field("posts_id", "value" : post.id) }}
        <div>
            <label for="title">Comment</label>
            {{ text_area("body") }}
        </div>
        <div>
            <label for="title">Name</label>
            {{ text_field("name") }}
        </div>
        <div>
            <label for="title">Email</label>
            {{ text_field("email") }}
        </div>
        <div>
            <label for="title">Website</label>
            {{ text_field("url") }}
        </div>
        {{ submit_button("Comment", "class" : "btn") }}
        {{ end_form() }}
    </div>
</div>
```

在帖子下方，我们遍历与该帖子相关的评论，如果我们将它们设置为发布，则打印评论。下面是评论表单，确保我们有一个可以获取帖子 ID 的隐藏字段。我们将此表单提交到`posts/comment`，这意味着我们现在需要在`Posts`控制器中添加一个`commentAction`函数。打开位于`app/controllers`的`PostsController.php`文件，并添加以下函数：

```php
public function commentAction(){
        $comment = new Comments();
        $comment->posts_id = $this->request->getPost("posts_id");
        $comment->body = $this->request->getPost("body");
        $comment->name = $this->request->getPost("name");
        $comment->email = $this->request->getPost("email");
        $comment->url = $this->request->getPost("url");
        $comment->submitted = date("Y-m-d H:i:s");
        $comment->publish = 0;
        $comment->save();

        $this->flash->success("Your comment has been submitted.");

        return $this->dispatcher->forward(
            array(
                "controller" => "posts",
                "action" => "show",
                "params" => array($comment->posts_id)
            )
        );
}
```

这相当简单。我们从`POST`变量中创建一个`Comments`对象的实例，设置提交日期，并将发布值设置为 0，这在我们的数据库中将表示为 false。然后，我们设置一个成功消息，并将用户转发回`Posts`的显示动作，设置 ID 为刚刚评论的帖子的 ID。

现在，我们需要一种方式来管理我们的评论。首先，我们需要一个`Comments`控制器。因此，在`app/controllers`中创建一个`CommentsController.php`文件，并确保包含`Paginator`，因为我们将会使用它。

```php
use Phalcon\Paginator\Adapter\Model as Paginator;
```

然后，我们需要我们的`CommentsController`类。

```php
class CommentsController extends ControllerBase {
  //Actions will go here
}
```

在`CommentsController`类内部，我们需要为索引、编辑、保存和删除操作创建动作。我们的`indexAction`函数返回所有评论的分页结果。

```php
    public function indexAction() {
        $numberPage = $this->request->getQuery("page", "int", 1);

        $comments = Comments::query()
            ->order("submitted DESC")
            ->execute();

        $paginator = new Paginator(array(
            "data" => $comments,
            "limit" => 10,
            "page" => $numberPage
        ));

        $this->view->page = $paginator->getPaginate();
    }
```

我们的`editAction`函数通过将其设置为发布，为审核评论时使用的编辑表单准备单个评论。

```php
    public function editAction($id) {

        if (!$this->request->isPost()) {

            $comment = Comments::findFirstByid($id);
            if (!$comment) {
                $this->flash->error("comment was not found");
                return $this->dispatcher->forward(
                    array(
                        "controller" => "comments",
                        "action" => "index"
                    )
                );
            }

            $this->view->id = $comment->id;

            $this->tag->setDefault("id", $comment->id);
            $this->tag->setDefault("body", $comment->body);
            $this->tag->setDefault("name", $comment->name);
            $this->tag->setDefault("email", $comment->email);
            $this->tag->setDefault("url", $comment->url);
            $this->tag->setDefault("submitted", $comment->submitted);
            $this->tag->setDefault("publish", $comment->publish);
            $this->tag->setDefault("posts_id", $comment->posts_id);

        }
    }
```

`saveAction`函数在我们审核后保存编辑过的评论。

```php
    public function saveAction() {

        if (!$this->request->isPost()) {
            return $this->dispatcher->forward(
                array(
                    "controller" => "comments",
                    "action" => "index"
                )
            );
        }

        $id = $this->request->getPost("id");

        $comment = Comments::findFirstByid($id);
        if (!$comment) {
            $this->flash->error("comment does not exist " . $id);
            return $this->dispatcher->forward(
                array(
                    "controller" => "comments",
                    "action" => "index"
                )
            );
        }

        $comment->id = $this->request->getPost("id");
        $comment->body = $this->request->getPost("body");
        $comment->name = $this->request->getPost("name");
        $comment->email = $this->request->getPost("email", "email");
        $comment->url = $this->request->getPost("url");
        $comment->publish = $this->request->getPost("publish");

        if (!$comment->save()) {

            foreach ($comment->getMessages() as $message) {
                $this->flash->error($message);
            }

            return $this->dispatcher->forward(
                array(
                    "controller" => "comments",
                    "action" => "edit",
                    "params" => array($comment->id)
                )
            );
        }

        $this->flash->success("comment was updated successfully");
        return $this->dispatcher->forward(
            array(
                "controller" => "comments",
                "action" => "index"
            )
        );

    }
```

`deleteAction`函数删除我们的评论垃圾邮件。

```php
    public function deleteAction($id) {

        $comment = Comments::findFirstByid($id);
        if (!$comment) {
            $this->flash->error("comment was not found");
            return $this->dispatcher->forward(
                array(
                    "controller" => "comments",
                    "action" => "index"
                )
            );
        }

        if (!$comment->delete()) {

            foreach ($comment->getMessages() as $message) {
                $this->flash->error($message);
            }

            return $this->dispatcher->forward(
                array(
                    "controller" => "comments",
                    "action" => "search"
                )
            );
        }

        $this->flash->success("comment was deleted successfully");
        return $this->dispatcher->forward(
            array(
                "controller" => "comments",
                "action" => "index"
            )
        );
    }
```

现在，我们只需要视图。首先，我们需要为我们的`indexAction`函数创建视图。因此，在`app/views`的`comments`文件夹中创建一个名为`index.volt`的文件，并在其中插入以下代码片段：

```php
{{ content() }}
<h1>Comments</h1>
<table class="browse" align="center">
    <thead>
        <tr>
            <th>Body</th>
            <th>Name</th>
            <th>Email</th>
            <th>Url</th>
            <th>Submitted</th>
            <th>Publish</th>
         </tr>
    </thead>
    <tbody>
    {% if page.items is defined %}
    {% for comment in page.items %}
        <tr>
            <td>{{ comment.body }}</td>
            <td>{{ comment.name }}</td>
            <td>{{ comment.email }}</td>
            <td>{{ comment.url }}</td>
            <td>{{ comment.submitted }}</td>
            <td>{{ comment.publish }}</td>
            <td>{{ link_to("comments/edit/"~comment.id, "Edit") }}</td>
            <td>{{ link_to("comments/delete/"~comment.id, "Delete") }}</td>
        </tr>
    {% endfor %}
    {% endif %}
    </tbody>
    <tbody><tr><td colspan="2" align="right">
        <table align="center">
            <tr>
                <td>{{ link_to("comments/search", "First") }}</td>
                <td>{{ link_to("comments/search?page="~page.before, "Previous") }}</td>
                <td>{{ link_to("comments/search?page="~page.next, "Next") }}</td>
                <td>{{ link_to("comments/search?page="~page.last, "Last") }}</td>
                <td>{{ page.current~"/"~page.total_pages }}</td>
            </tr>
        </table>
    </td></tr></tbody>
</table>
```

然后，当编辑我们的帖子时将使用的视图。将以下代码片段保存为`edit.volt`，位于`app/views/comments`：

```php
{{ content() }}
{{ link_to("comments", "Go Back") }}

<div align="center">
    <h1>Edit comments</h1>
</div>

<div>
    {{ form("comments/save", "method":"post") }}
        <label for="body">Body</label>{{ text_area("body") }}
        <label for="name">Name</label>{{ text_field("name") }}
        <label for="email">Email</label>{{ text_field("email") }}
        <label for="url">Url</label>{{ text_field("url") }}
        <label for="publish">Publish</label>
        {{ radio_field("publish", "value" : 1) }} Yes
        {{ radio_field("publish", "value" : 0) }} No
        {{ hidden_field("id") }}
        {{ submit_button("Save", "class" : "btn") }}
    {{ end_form() }}
</div>
```

现在，如果我们以用户身份登录，我们就可以对帖子进行评论和管理。

## 添加聚合源

一个博客如果没有一种方式来聚合我们的帖子，那就不是一个真正的博客。幸运的是，向我们的博客添加一个聚合源将会非常简单。让我们将我们的聚合源放在`http://localhost/phalconBlog/posts/feed`。这意味着我们需要在帖子控制器中添加一个新的`feedAction`函数。因此，打开位于`app/controllers`的`PostsController.php`文件，并添加以下函数：

```php
public function feedAction() {
    $posts = Posts::find(
        array(
            'order' => 'published DESC',
            'limit' => 10
        )
    );

    $rss_posts = array();
    foreach ($posts as $post){
        $post->rss_date = date("D, d M Y H:i:s O", strtotime($post->published));
        $rss_posts[] = $post;
    }
    $this->view->posts = $rss_posts;

    $this->view->setRenderLevel(Phalcon\Mvc\View::LEVEL_ACTION_VIEW);
}
```

这里，我们使用 `Posts::find` 仅检索按发布日期降序排列的 10 篇帖子。由于 RSS 源中的日期格式非常特定，并且不匹配 MySQL datetime 格式，我们遍历我们的记录，转换发布日期，并将此日期添加到每个 `post` 对象中。然后，我们将对象数组发送到视图。在函数的末尾，我们设置视图的渲染级别。在这种情况下，我们不希望我们的主布局或帖子布局渲染。我们只想渲染我们即将创建的专用模板，因此我们将我们的级别设置为 `LEVEL_ACTION_VIEW`。以下六个级别可用：

+   `LEVEL_NO_RENDER`：这不会渲染任何视图

+   `LEVEL_ACTION_VIEW`：这会渲染与操作关联的视图

+   `LEVEL_BEFORE_TEMPLATE`：这会在控制器布局之前生成视图

+   `LEVEL_LAYOUT`：这会生成控制器的布局

+   `LEVEL_AFTER_TEMPLATE`：这会在控制器布局之后生成视图

+   `LEVEL_MAIN_LAYOUT`：这会生成主布局

在 `app/views` 的 `posts` 文件夹中创建一个新文件，命名为 `feed.volt`，并在 `feed.volt` 文件中插入以下代码片段：

```php
{{'<?xml version="1.0" encoding="UTF-8" ?>'}}

<rss version="2.0">
    <channel>
        <title>{{ config.blog.title }}</title>
        <description>This is a demonstration of the Phalcon framework</description>
        <link>{{ config.blog.url }}</link>
        {% for post in posts %}
            <item>
                <title>{{ post.title|e }}</title>
                <description>{{ post.excerpt|e }}</description>
                <link>{{ config.blog.url }}{{ url("posts/show/"~post.id) }}</link>
                <guid>{{ config.blog.url }}{{ url("posts/show/"~post.id) }}</guid>
                <pubDate>{{ post.rss_date }}</pubDate>
            </item>
        {% endfor %}
    </channel>
</rss>
```

我们在 Volt 模板标签的顶部包裹 XML 标签，以隐藏 Volt 模板引擎中的 `<?`。我们添加一些必要的 RSS 元素，包括标题、描述和博客的链接。请注意，由于我们已经将配置加载到 DI 容器中，我们可以访问在那里为博客标题和 URL 创建的设置。下一步是遍历帖子。这部分与 `index.volt` 文件中的代码类似。但您可能会注意到标题和摘要后面的 `|e`。这是从 Volt 中调用的 Phalcon HTML 转义过滤器，以确保我们的源在任何可能存在于我们内容中的字符下都保持有效。我们正在生成 XML，并使用我们在 `feedAction` 函数中创建的 `rss_date` 属性。

我们需要执行的最后一个步骤是将 RSS 自动发现链接添加到我们页面的头部。因此，打开位于 `app/views` 的 `index.volt` 文件，并在头部标签之间添加以下代码片段：

```php
{{ tag_html(
            "link",
            [
                "rel": "alternate",
                "type": "application/rss+xml",
                "title": "RSS Feed for Phalcon Blog",
                "href": config.application.baseUri~"posts/feed"
            ],
             true,
             true,
             true)
 }}
```

在这里，我们使用通用的 `tagHtml()` 标签辅助函数来为我们生成链接，在 Volt 中变为 `tag_html()`。第一个参数是标签的名称。第二个参数是标签属性的数组。请注意，`href` 参数使用了 `baseUri` 配置设置和 `~`，这是 Volt 中用于连接字符串的字符。第三个参数是一个我们设置为 true 的布尔值，因为我们希望这是一个自闭合标签。我们将第四个参数设置为 true，因为我们只想生成起始标签。最后，我们将第五个参数设置为 true，因为我们希望在标签后生成一个换行符。

现在，我们可以开始向互联网的其余部分介绍我们的新博客。

# 发送更新 ping

为了 ping 像[`weblogs.com`](http://weblogs.com)这样的网站并通知它们我们有一篇新文章，我们将进行一些重构。看起来我们在 HTML 标题标签中使用了我们博客的标题。我们还需要在将要编写的 ping 函数中使用它。因此，与其在两个必须编辑的地方硬编码标题，不如如果我们更改标题，创建一个配置设置会更好。所以，将以下代码片段添加到`app/config`中`config.ini`文件：

```php
[blog]
title = Phalcon Blog
url = http://localhost

```

由于我们现在将在控制器中使用这个设置，我们需要在那里访问它。为此，我们可以将其添加到依赖注入容器中。这可以通过将以下代码行添加到`app/config`中`services.php`文件的底部来完成：

```php
$di->set('config', $config);
```

现在，我们可以打开位于`app/controllers`的`ControllerBase.php`文件，并使用我们的配置设置设置我们的标题标签。

```php
$this->tag->setTitle($this->config->blog->title);
```

我们可以在`PostsController.php`文件中创建一个新的函数来为我们 ping 内容聚合网站。

```php
private function sendPings(){
        $request = '<?xml version="1.0" encoding="iso-8859-1"?>
                    <methodCall>
                    <methodName>weblogUpdates.ping</methodName>
                    <params>
                     <param>
                      <value>
                       <string>'.$this->config->blog->title.'</string>
                      </value>
                     </param>
                     <param>
                      <value>
                       <string>'.$this->config->blog->url.$this->url->get('posts/feed').'</string>
                      </value>
                     </param>
                    </params>
                    </methodCall>';

        $ping_urls = array(
            'http://blogsearch.google.com/ping/RPC2',
            'http://rpc.weblogs.com/RPC2',
            'http://ping.blo.gs/'
        );
        foreach($ping_urls as $ping_url){
            $ch = curl_init();
            curl_setopt($ch, CURLOPT_URL, $ping_url);
            curl_setopt($ch, CURLOPT_RETURNTRANSFER, true );
            curl_setopt($ch, CURLOPT_POST, true );
            curl_setopt($ch, CURLOPT_POSTFIELDS, trim($request));
            $results = curl_exec($ch);
        }
        curl_close($ch);
    }
```

`$request`变量是我们将要发布到服务的 XML。现在我们已经将配置设置添加到我们的依赖注入容器中，我们可以使用`$this->config->blog->title`访问博客的标题设置，以及使用`$this->config->blog->url`访问 URL 设置。`$ping_urls`数组包含我们将要 ping 的服务 URL。如果您喜欢，可以向此数组添加更多服务。然后，我们使用 PHP `curl`进行 ping。

现在，在`createAction`函数中，找到以下代码行：

```php
$this->flash->success("post was created successfully");
```

在前面的代码之前添加以下行：

```php
$this->sendPings();
```

我们可以添加到这个函数中的另一个特性是记录我们 ping 的结果，这样我们就有了一种调试实际发生情况的方法，因为我们的函数不返回状态。

现在，是时候向配置文件的应用程序部分添加另一个变量了。

```php
logsDir = ../app/logs/
```

现在，我们需要在依赖注入容器中添加一个服务来记录我们 ping 的结果。打开位于`app/config`的`services.php`文件，并添加以下代码行：

```php
//Logging
$di->set(
    'pingLogger', function () use ($config){
        $logger = new \Phalcon\Logger\Adapter\File($config->application->logsDir.'ping.log');
        return $logger;
    }
);
```

这将使我们能够访问我们刚刚创建的`sendPings`函数中的`pingLogger`服务。因此，在设置完`$result`变量后，我们可以记录我们 ping 的 URL 以及我们收到的响应。我们检查结果是否为假，表示`curl`失败，然后更改`$result`变量以反映该错误。

```php
$result = curl_exec($ch);
if($result === false){
       $result = "Curl Error";
}
$this->pingLogger->log($ping_url.PHP_EOL.$result);
```

# 使用视图部分

我们已经探讨了布局和级联视图。我们还有在 Phalcon 中使用部分视图的选项。使用部分模板允许我们在应用程序的各个部分重用相同的功能。我们可能想要使用部分视图的几个地方是导航栏和侧边栏。目前，我们的主布局做了很多事情。导航栏或侧边栏上没有太多内容。但随着我们添加更多功能，它们可能会变得更加复杂，将它们保存在单独的文件中将帮助我们组织和简化代码。

你可以在任何你选择的地方创建一个文件夹来存放你的部分视图。你甚至可以将它们放在位于`app`的`views`文件夹中。但是，为了使我们的模板更有组织性，让我们在位于`app`的`views`文件夹内创建一个名为`partials`的文件夹。现在，打开位于`app/views`的`index.volt`文件。我们将从这个文件中剪切侧边栏，并将其粘贴到一个名为`sidebar.volt`的新文件中，我们将将其保存在我们新的`partials`文件夹中。该文件的 内容应如下代码片段所示：

```php
<div class="span3">
    <div class="well">
        {{ form("posts/search", "method":"post", "autocomplete" : "off", "class" : "form-inline") }}
            <div class="input-append">
                {{ text_field("body", "class" : "input-medium") }}
                {{ submit_button("Search", "class" : "btn") }}
            </div>
        {{ end_form() }}
    </div>
</div>
```

我们将在`index.volt`文件中用以下代码行替换它：

```php
{{ partial("partials/sidebar") }}
```

这是我们在 Volt 中使用的代码。如果我们使用 PHP 模板，我们会使用`<?php $this->partial("partials/sidebar"); ?>`。请注意，我们不要在`path`参数中添加文件扩展名。确保对你的`partials`文件夹也这样做。添加扩展名会导致错误。

现在，你可以在位于`app/views`的`index.phtml`文件中，使用具有`navbar`类的`div`标签，将其粘贴到名为`navbar.volt`的文件中，将其保存在你的`partials`文件夹中，并用以下代码行替换`div`标签：

```php
{{ partial("partials/navbar") }}
```

现在，不同的功能类型已在你应用中分离。你可能会在应用的其他地方找到可能想要使用部分视图的地方，例如在页眉或页脚中。要了解更多关于 Phalcon 的信息，请访问[`docs.phalconphp.com/en/latest/index.html`](http://docs.phalconphp.com/en/latest/index.html)。

# Phalcon 中的缓存

缓存可以加快并节省大量流量或执行大量密集型重复数据库查询的网站上的资源，但此功能只应在实际需要时实施。换句话说，我们当前的博客可能不需要缓存，但我们打算研究 Phalcon 中的缓存，并将缓存添加到我们博客的一部分，以便了解其工作原理。

# 设置缓存服务

在 Phalcon 中，缓存由两部分组成，一个处理缓存过期和转换的前端缓存，以及一个处理前端请求时的读取和写入的后端缓存。以下是一个缓存服务的示例。我们可以简单地将其添加到位于`app/config`的`service.php`文件中。

```php
$di->set(
    'viewCache', function () use ($config) {

        //Cache for one day
        $frontCache = new \Phalcon\Cache\Frontend\Data(array(
            "lifetime" => 86400
        ));

        //Set file cache
        $cache = new Phalcon\Cache\Backend\File($frontCache, array(
            "cacheDir" => $config->application->cacheDir
        ));

        return $cache;
    }
);
```

到现在为止，我们已经很习惯于设置新的 Phalcon 服务。在这个新服务中，我们使用配置变量，以便它可以使用我们的缓存目录设置。我们给前端缓存设置了一天的生命周期，并将这个变量以及我们的缓存目录位置传递给后端缓存。

我们在这个服务中使用了基于文件的缓存。这不是最快的缓存机制，可能是最慢的，但它易于设置，且不需要其他软件。然而，你并不限于使用基于文件的 Phalcon 后端。只要你有所需的软件和 PHP 扩展安装，你就可以在 Phalcon 中使用以下任何一个作为后端缓存：

+   文件

    +   Memcached

    +   APC

    +   MongoDB

    +   Xcache

Phalcon 也有多种前端缓存适配器。在我们的缓存服务代码中，我们指定了一个数据适配器，在保存数据之前对其进行序列化，但你也可以使用以下前端适配器之一：

+   输出

+   JSON

+   IgBinary

+   Base64

+   无

## 使用 Phalcon 缓存

现在我们已经设置了缓存服务，我们可以在需要的地方使用它。首先，你需要一个缓存键。所有 Phalcon 缓存都使用键来存储和访问缓存数据。它需要对于那份数据是唯一的。在我们的应用程序中，为缓存生成键的好地方是在`app/controllers`文件夹中的`ControllerBase.php`文件中。我们可以在`ControllerBase`类中添加以下函数，以便所有控制器继承它：

```php
public function createKey($controller, $action, $parameters = array())
{
        return urlencode($controller.$action.serialize($parameters));
}
```

此函数将为我们创建一个唯一的键。因此，现在我们通过打开位于`app/controllers`的`PostController.php`文件并修改`showAction`函数来测试使用我们的缓存的功能。我们可以在函数的开始处添加以下代码行来访问我们创建的缓存服务：

```php
$cache = $this->di->get("viewCache");
```

然后，通过执行以下代码行，你可以访问缓存内容或开始缓存：

```php
$key = $this->createKey('posts', 'show', array($id));
$post = $cache->get($key);
```

然后，我们检查是否有内容，如果没有，我们就从数据库中获取它，并使用我们的键将其保存到缓存中。

```php
if ($post === null) {
            $post = Posts::findFirstByid($id);
            $cache->save($key, $post);
}
```

函数的其余部分保持不变。

```php
$this->tag->prependTitle($post->title . " - ");
$this->view->post = $post;
```

在浏览器中打开你博客的一篇文章并刷新页面。你将在`app`文件夹中的`cache`文件夹中找到缓存文件。在文件中，将存在序列化数据，代表`$post`对象。要了解更多关于在 Phalcon 中使用缓存的信息，请访问[`docs.phalconphp.com/en/latest/reference/cache.html`](http://docs.phalconphp.com/en/latest/reference/cache.html)。

你也可以在模型级别缓存数据。机制是相同的。尝试通过键访问缓存的数据。检查是否存在数据。如果没有，执行数据库调用以检索数据并将其缓存。然后，返回结果。要了解 ORM 中的缓存，请访问[`docs.phalconphp.com/en/latest/reference/models-cache.html`](http://docs.phalconphp.com/en/latest/reference/models-cache.html)。

# Phalcon 中的路由

在我们查看 Phalcon 中的其他应用结构，特别是微应用之前，我们应该先看看路由。到目前为止，我们只是在我们的应用中有了路由；神奇地映射到我们的控制器和其中的动作，让 Phalcon 为我们处理所有的路由思考。这是因为我们正在使用的单 MVC 结构中，路由是在 MVC 模式下的。我们还有匹配模式的选择。你会注意到我们的博客应用有一个什么也不做的首页。它仍然有默认的 Phalcon 消息。我们网站的大部分内容目前位于`http://localhost/phalconBlog/posts`。嗯，为了修复这个问题，我们将使用一种黑客式的方法来学习 Phalcon 中的路由，那就是使用一个路由器。因此，我们需要向我们的依赖注入容器中添加另一个服务。打开位于`app/config`的`services.php`文件，并在底部添加以下代码行：

```php
$di->set(
    'router', function () {
        $router = new Router();
        $router->add(
            "/", array(
                'controller' => 'posts',
                'action' => 'index',
            )
        );
        return $router;
    }
);
```

当我们向 MVC 路由器添加路由时，第一个参数是路径，第二个参数是任何数组，它将此路径映射到一个模块、控制器和动作。

在即将出现的微应用示例中，使用的是匹配模式的路由。要了解更多关于 Phalcon 中路由的信息，请访问[`docs.phalconphp.com/en/latest/reference/routing.html`](http://docs.phalconphp.com/en/latest/reference/routing.html)。

# Phalcon 的其他项目类型

我们在 Phalcon 中创建了一种类型的应用，一个单 MVC 应用。但是，这种应用结构并不适合所有类型的项目。现在我们将看看你可以使用 Phalcon 构建的几种其他类型的应用。当然，作为一个非常松散耦合的框架，Phalcon 并不限制你只使用这些结构，你可以根据自己的选择来组织项目。

## 多模块应用

随着应用规模的扩大，将代码组织到模块中可能变得容易。在项目中的`app`文件夹而不是一个`apps`文件夹下，你可以有多个文件夹，每个文件夹都有自己的模型、视图和控制器集合。在 MVC 模式下，路由已经设置好以处理模块。只需要几个额外的步骤。要了解更多关于 Phalcon 多模块应用的信息，请访问[`docs.phalconphp.com/en/latest/reference/applications.html#multi-module`](http://docs.phalconphp.com/en/latest/reference/applications.html#multi-module)。

## 微应用

对于比我们写的应用更简单的应用，使用微框架结构可能更有意义。在 Phalcon 中，微应用可以像以下代码片段那样简单：

```php
<?php

$app = new Phalcon\Mvc\Micro();

$app->get(
    '/user/{name}', function ($name) {
        echo "<h1>Hi $name!</h1>";
    }
);

$app->get(
    '/api/user/{name}', function ($name) {
        echo json_encode(array("message" => "Hi ". $name));
    }
);

$app->handle();
```

这个应用程序没有太多内容。首先，我们创建了一个`Phalcon\MVC\Micro`的实例，然后我们定义了我们的路由。因此，当访问者访问应用程序的`/user/Stephan`页面时，他们会看到**Hi Stephan**。第二个路由展示了微应用程序用于 API 的一个完美用途。大多数简单的 API 不需要复杂的控制器。这个例子在这里是为了展示一个工作的 Phalcon 应用程序可以有多小。它使用匹配模式的路由。随着微应用程序规模的扩大，你可以使用它支持的更多功能，包括模型、重定向和服务。想了解更多关于 Phalcon 微应用程序的信息，请访问[`docs.phalconphp.com/en/latest/reference/micro.html`](http://docs.phalconphp.com/en/latest/reference/micro.html)。

## 命令行应用程序

有时，你可能不需要一个完整的 Web 应用程序来完成工作。对于从命令行或 cron 运行的脚本，你只需要一个简单的结构来组织你代码的功能。Phalcon 命令行应用程序的结构可以从以下代码片段开始：

```php
app/
  tasks/
  cli.php
```

启动文件是`cli.php`，它以与 Phalcon 中所有命令行应用程序启动时几乎相同的方式开始。

```php
<?php

use Phalcon\DI\FactoryDefault\CLI as CliDI,
    Phalcon\CLI\Console as ConsoleApp;

//Using the CLI factory default services container
$di = new CliDI();

// Define the application path
defined('APPLICATION_PATH')
  || define('APPLICATION_PATH', realpath(dirname(__FILE__)));

//Register the autoloader
$loader = new \Phalcon\Loader();
$loader->registerDirs(
    array(
        APPLICATION_PATH . '/tasks'
    )
);
$loader->register();

// Load the config
$config = include APPLICATION_PATH . '/config/config.ini';
$di->set('config', $config);

//Create a console application
$console = new ConsoleApp();
$console->setDI($di);

//Process console arguments
$arguments = array();
$params = array();

foreach($argv as $k => $arg) {
    if($k == 1) {
        $arguments['task'] = $arg;
    } elseif($k == 2) {
        $arguments['action'] = $arg;
    } elseif($k >= 3) {
        $params[] = $arg;
    }
}
if(count($params) > 0) {
    $arguments['params'] = $params;
}

// define global constants for the current task and action
define('CURRENT_TASK', (isset($argv[1]) ? $argv[1] : null));
define('CURRENT_ACTION', (isset($argv[2]) ? $argv[2] : null));

try {
    // handle incoming arguments
    $console->handle($arguments);
}
catch (\Phalcon\Exception $e) {
    echo $e->getMessage();
    exit(255);
}
```

启动文件只需处理命令并选择正确的任务和操作。我们将任务存储在`tasks`文件夹中。命令行应用程序至少需要有一个`mainTask`和`mainAction`。因此，我们可以将此文件添加到`tasks`文件夹中，并将其命名为`MainTask.php`。

```php
<?php

class mainTask extends \Phalcon\CLI\Task
{

    public function mainAction() {
        echo "I am a task that doesn't do much";
    }

    public function anotherAction(array $params) {
        foreach($params as $p){
            echo "Hi, my name is ".$p.PHP_EOL;
        }
    }
}
```

执行以下命令行时，主任务的`mainAction`函数将被运行：

```php
php app/cli.php

```

要调用特定的任务和操作，只需将任务作为第一个参数，将操作作为第二个参数，任何参数（`tom`、`dick`和`harry`）都可以由名为`Executing`的动作函数处理：

```php
php app/cli.php main another tom dick harry

```

输出如下：

```php
Hi, my name is tom
Hi, my name is dick
Hi, my name is harry

```

想要了解更多关于使用 Phalcon 创建命令行应用程序的信息，请访问[`docs.phalconphp.com/en/latest/reference/cli.html`](http://docs.phalconphp.com/en/latest/reference/cli.html)。

# 摘要

在本章中，我们在为我们的博客应用程序添加功能的同时，更多地了解了我们可以使用 Phalcon 框架做什么。这并不是功能最丰富的博客，但我们展示了如何使用 Phalcon 标签助手设置文档类型并为我们的页面生成动态标题。我们还解释了如何在 Phalcon 中控制用户访问、散列密码和设置 cookie。然后，我们使用视图部分将我们的视图拆分成可重用的块。以防我们的博客有任何流量，我们在 Phalcon 中尝试了一些缓存。我们还学习了如何在 Phalcon 中使用路由。最后，我们学习了我们可以用于 Phalcon 应用程序的其他结构。想了解更多关于 Phalcon 的信息，请访问[`phalconphp.com`](http://phalconphp.com)。
