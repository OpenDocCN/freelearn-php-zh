# 第十四章：将 URL 路径与文件路径解耦

尽管我们有一个文档根目录将我们的公共和非公共资源分开，但我们传统应用程序的用户仍然直接浏览我们的页面脚本。这意味着我们的 URL 直接与网络服务器上的文件系统路径耦合。

我们的下一步是解耦路径，这样我们就可以独立地将 URL 路由到任何我们想要的目标。这意味着我们需要建立一个前端控制器来处理我们传统应用程序的所有传入请求。

# 耦合路径

正如我们在上一章中所指出的，我们的网络服务器充当了我们传统应用程序的前端控制器、路由器和调度器的综合功能。页面脚本的路由仍然直接映射到文件系统，使用我们的`docroot/`目录作为所有 URL 路径的基础。

这给我们带来了一些结构性问题。例如，如果我们想公开一个新的或不同的 URL，我们必须修改文件系统中相关页面脚本的位置。同样，我们无法更改哪个页面脚本响应特定的 URL。在路由之前，没有办法拦截传入的请求。

这些以及其他问题，包括完成未来重构的能力，意味着我们必须为所有传入请求创建一个单一入口点。这个入口点被称为前端控制器。

在我们对传统应用程序实现的第一个前端控制器中，我们将添加一个路由器来将传入的 URL 路径转换为页面脚本路径。这将允许我们将页面脚本从文档根目录中移除，从而将 URL 与文件系统解耦。

## 解耦过程

与将我们的公共资源与非公共资源分开一样，我们将不得不对我们的网络服务器配置进行更改。具体来说，我们将启用 URL 重写，以便将所有传入请求指向一个前端控制器。我们需要与我们的运维人员协调这次重构，以便他们能够尽可能轻松地部署这些更改。

一般来说，这个过程如下：

1.  与运维协调以沟通我们的意图。

1.  在文档根目录中创建一个前端控制器脚本。

1.  为我们的页面脚本创建一个`pages/`目录，以及一个`页面未找到`页面脚本和控制器。

1.  重新配置网络服务器以启用 URL 重写。

1.  抽查重新配置的网络服务器，确保前端控制器和 URL 重写正常工作。

1.  将所有页面脚本从`docroot/`移动到`pages/`，并在此过程中进行抽查。

1.  提交、推送，并与运维协调进行 QA 测试。

## 与运维协调

这是整个过程中最重要的一步。我们绝不能在没有与负责服务器的人员（即我们的运维人员）讨论我们意图的情况下进行影响服务器配置的更改。

在这种情况下，我们需要告诉我们的运维人员，我们必须启用 URL 重写。他们将告知或指导我们如何为我们特定的网络服务器执行此操作。

或者，如果我们没有运维人员并且负责我们自己的服务器，我们将需要自行确定如何启用 URL 重写。在这种情况下要小心进行。

### 添加一个前端控制器

一旦我们与运维人员协调好，我们将添加一个前端控制器脚本。我们还将添加一个`页面未找到`脚本、控制器和视图。

首先，在我们的文档根目录中创建前端控制器脚本。它使用`Router`类将传入的 URL 映射到页面脚本。我们称之为 front.php，或者其他表明它是前端控制器的名称：

```php
docroot/front.php
1 <?php
2 // the router class file
3 require dirname(__DIR__) . '/classes/Mlaphp/Router.php';
4
5 // set up the router
6 $pages_dir = dirname(__DIR__) . '/pages';
7 $router = new \Mlaphp\Router($pages_dir);
8
9 // match against the url path
10 $path = parse_url($_SERVER['REQUEST_URI'], PHP_URL_PATH);
11 $route = $router->match($path);
12
13 // require the page script
14 require $route;
15 ?>
```

### 注意

我们`require`了`Router`类文件，因为自动加载程序尚未注册。这只会在执行页面脚本时发生，而这只会在前端控制器逻辑结束时发生。我们将在下一章中解决这个问题。

### 创建一个`pages/`目录

前端控制器引用了一个`$pages_dir`。我们的想法是将所有页面脚本从文档根目录移动到这个新目录中。

首先，在我们的旧应用程序的顶层创建一个`pages/`目录，与`classes/`、`docroot/`、`views/`等目录并列。

然后，我们创建一个`pages/not-found.php`脚本，以及一个相应的控制器和视图文件。当`Router`无法匹配 URL 路径时，前端控制器将调用`not-found.php`脚本。`not-found.php`脚本应该像旧应用程序中的任何其他页面脚本一样设置自己，然后调用相应的视图文件以获取响应：

```php
**pages/not-found.php**
1 <?php
2 require '../includes/setup.php';
3
4 $request = new \Mlaphp\Request($GLOBALS);
5 $response = new \Mlaphp\Response('/path/to/app/views');
6 $controller = new \Controller\NotFound($request, $response);
7
8 $response = $controller->__invoke();
9
10 $response->send();
11 ?>
```

```php
**classes/Controller/NotFound.php**
1 <?php
2 namespace Controller;
3
4 use Mlaphp\Request;
5 use Mlaphp\Response;
6
7 class NotFound
8 {
9 protected $request;
10
11 protected $response;
12
13 public function __construct(Request $request, Response $response)
14 {
15 $this->request = $request;
16 $this->response = $response;
17 }
18
19 public function __invoke()
20 {
21 $url_path = parse_url(
22 $this->request->server['REQUEST_URI'],
23 PHP_URL_PATH
24 );
25
26 $this->response->setView('not-found.html.php');
27 $this->response->setVars(array(
28 'url_path' => $url_path,
29 ));
30
31 return $this->response;
32 }
33 }
34 ?>
```

```php
**views/not-found.html.php**
1 <?php $this->header('HTTP/1.1 404 Not Found'); ?>
2 <html>
3 <head>
4 <title>Not Found</title>
5 </head>
6 <body>
7 <h1>Not Found</h1>
8 <p><?php echo $this->esc($url_path); ?></p>
9 </body>
10 </html>
```

## 重新配置服务器

现在我们已经放置了我们的前端控制器并为我们的页面脚本设置了目标位置，我们重新配置本地开发 Web 服务器以启用 URL 重写。我们的运维人员应该已经给了我们一些关于如何做这个的指示。

### 注意

不幸的是，本书范围之外无法提供有关 Web 服务器管理的完整说明。请查阅您特定服务器的文档以获取更多信息。

在 Apache 中，我们首先启用`mod_rewrite`模块。在某些 Linux 发行版中，这很容易，只需发出`sudo a2enmod` rewrite 命令。在其他情况下，我们需要编辑`httpd.conf`文件以启用它。

一旦启用了 URL 重写，我们需要指示 Web 服务器将所有传入的请求指向我们的前端控制器。在 Apache 中，我们可以向我们的旧应用程序添加一个`docroot/.htaccess`文件。或者，我们可以修改本地开发服务器的 Apache 配置文件之一。重写逻辑如下所示：

```php
**docroot/.htaccess**
1 # enable rewriting
2 RewriteEngine On
3
4 # turn empty requests into requests for the "front.php"
5 # bootstrap script, keeping the query string intact
6 RewriteRule ^$ front.php [QSA]
7
8 # for all files and dirs not in the document root,
9 # reroute to the "front.php" bootstrap script,
10 # keeping the query string intact, and making this
11 # the last rewrite rule
12 RewriteCond %{REQUEST_FILENAME} !-f
13 RewriteCond %{REQUEST_FILENAME} !-d
14 RewriteRule ^(.*)$ front.php [QSA,L]
```

### 注意

例如，如果传入请求是`/foo/bar/baz.php`，Web 服务器将调用`front.php`脚本。这对于每个请求都是如此。各种超全局变量的值将保持不变，因此`$_SERVER['REQUEST_URI']`仍将指示`/foo/bar/baz.php`。

最后，在启用了 URL 重写之后，我们重新启动或重新加载 Web 服务器以使我们的更改生效。

### 抽查

现在我们已经启用了 URL 重写以将所有请求指向我们的新前端控制器，我们应该浏览我们的旧应用程序，使用我们知道不存在的 URL 路径。前端控制器应该显示我们的`not-found.php`页面脚本的输出。这表明我们的更改正常工作。如果不是，我们需要回顾和修改到目前为止的更改，并尝试修复任何出错的地方。

### 移动页面脚本

一旦我们确定 URL 重写和前端控制器正常运行，我们可以开始将所有页面脚本从`docroot/`移动到我们的新`pages/`目录中。请注意，我们只移动页面脚本。我们应该将所有其他资源留在`docroot/`中，包括`front.php`前端控制器。

例如，如果我们开始时有这样的结构：

```php
**/path/to/app/**
docroot/
css/
foo/
bar/
baz.php
front.php
images/
index.php
js/
pages/
not-found.php
```

我们应该最终得到这样的结构：

```php
**/path/to/app/**
docroot/
css/
front.php
images/
js/
pages/
foo/
bar/
baz.php
index.php
not-found.php
```

我们只移动了页面脚本。图像、CSS 文件、Javascript 文件和前端控制器都保留在`docroot/`中。

因为我们在移动文件，我们可能需要更改我们的包含路径值，以指向新的相对目录位置。

当我们将每个文件或目录从`docroot/`移动到`pages/`时，我们应该抽查我们的更改，以确保旧应用程序继续正常工作。

由于之前描述的重写规则，我们的页面脚本应该继续工作，无论它们是在`docroot/`还是`pages/`中。我们要确保在继续之前将所有页面脚本移动到`pages/`中。

### 提交、推送、协调

当我们将所有页面脚本移动到新的`pages/`目录，并且我们的旧应用程序在这个新结构中正常工作时，我们提交所有更改并将它们推送到共同的存储库。

在这一点上，我们通常会通知质量保证部门我们的更改，让他们进行测试。然而，由于我们对服务器配置进行了更改，我们需要与运营人员协调质量保证测试。运营部门可能需要部署新配置到质量保证服务器上。只有这样，质量保证部门才能有效地检查我们的工作。

## 常见问题

### 我们真的解耦了路径吗？

敏锐的观察者会注意到我们的*Router*仍然使用传入的 URL 路径来查找页面脚本。这与原始设置之间的唯一区别是，路径被映射到`pages/`目录而不是`docroot/`目录。毕竟，我们真的将 URL 与文件系统解耦了吗？

是的，我们已经实现了我们的解耦目标。这是因为我们现在在 URL 路径和执行的页面脚本之间有一个拦截点。使用*Router*，我们可以创建一个路由数组，其中 URL 路径是键，文件路径是值。该映射数组将允许我们将传入的 URL 路径路由到任何我们喜欢的页面脚本。

例如，如果我们想将 URL 路径`/foo/bar.php`映射到页面脚本`/baz/dib.php`，我们可以通过*Router*上的`setRoutes()`方法来实现：

```php
1 $router->setRoutes(array(
2 '/foo/bar.php' => '/baz/dib.php',
3 ));
```

然后，当我们将传入的 URL 路径`/foo/bar.php`与*Router*进行`match()`时，我们返回的路由将是`/baz/dib.php`。然后我们可以执行该路由作为传入 URL 的页面脚本。我们将在下一章节中使用这种技术的变体。

# 回顾和下一步

通过将 URL 与页面脚本解耦，我们几乎已经完成了我们的现代化工作。只剩下两个重构。首先，我们将重复的逻辑从页面脚本移到前端控制器。然后我们将完全删除页面脚本，并用依赖注入容器替换它们。
