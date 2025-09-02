# 第一章. 使用 Phalcon 入门

什么是 Phalcon？让我们先从官方网站的文档中引用一些内容（[`phalconphp.com/`](http://phalconphp.com/)）：

> *"Phalcon 是一个开源的全栈 PHP 框架，以 C 扩展的形式编写，针对高性能进行了优化。"*

Phalcon 2.0 版本于四月发布，它是用一种名为 Zephir 的新语言开发的（[`zephir-lang.com/`](http://zephir-lang.com/)）。Zephir 特别设计用于开发 PHP 扩展，并且对于（PHP 和 C）开发者来说都非常友好。

现在有很多框架。我们选择 Phalcon 的主要原因是因为它的学习曲线陡峭、速度快，并且它是解耦的。（我们可以独立使用其任何组件。）如果你对 **模型-视图-控制器**（**MVC**）有所了解，并且对任何 **对象关系映射**（**ORM**）有一些经验，你会发现与之工作非常直接。

我们将从本章开始我们的旅程，我们将：

+   配置我们的 Web 服务器

+   安装 Phalcon

+   稍微讨论一下 Phalcon 的工作原理

在开始之前，我们假设你正在使用一个 *nix 环境。我个人对 Debian 发行版感到很舒适，特别是 Ubuntu，这是我每天都在使用的；所以，我们将讨论的安装步骤是针对 Ubuntu 的。操作系统是一个个人选择的问题，但我强烈推荐任何 *nix 发行版用于开发。（即使是微软在今年早些时候决定开源他们的 ASP.NET for Linux））

对于其他类型的操作系统，你将不得不搜索它们的官方文档，关于“如何”的问题。这本书的目的是关于 Phalcon 以及在不同类型的操作系统上安装不同软件的教程，这些内容超出了本书的范围。

### 注意

这里是包含不同操作系统安装说明的 URL 列表：

+   [`docs.phalconphp.com/en/latest/reference/install.html#windows`](http://docs.phalconphp.com/en/latest/reference/install.html#windows)

+   [`docs.phalconphp.com/en/latest/reference/install.html#mac-os-x`](http://docs.phalconphp.com/en/latest/reference/install.html#mac-os-x)

+   [`docs.phalconphp.com/en/latest/reference/install.html#freebsd`](http://docs.phalconphp.com/en/latest/reference/install.html#freebsd)

高级开发者可能不会在某些主题或某些技术/建议上同意我的观点。一般来说，作为一名开发者，我认为你应该分析适合你的内容，并根据你的（或客户的）需求开发平台。此外，最重要的是，没有所谓的“完美解决方案”。总有改进的空间。

# 安装所需的软件

我们需要安装以下我们将在这本书中使用的软件：

+   PHP

+   Nginx 和 Apache

+   MongoDB

+   MySQL

+   GIT

+   Redis

+   Phalcon

## 安装 PHP

您可能已经在本系统中安装了 PHP，因为您正在阅读这本书。但是，以防万一您还没有，以下是一些快速安装最新 PHP 版本的简单步骤（Phalcon 运行在 PHP 版本 >= 5.3）。我建议您使用 Ondřej Surý ([`launchpad.net/~ondrej/+archive/ubuntu/php5`](https://launchpad.net/~ondrej/+archive/ubuntu/php5)) 的 **个人软件包存档**（**PPA**），因为它上有可用的最新 PHP 版本：

```php
$ sudo add-apt-repository ppa:ondrej/php5
$ sudo apt-get update

```

如果您不想使用此步骤，您可以直接从官方仓库安装 PHP：

```php
$ sudo apt-get install php

```

Apache 将默认与 PHP 一起安装。但是，如果您想使用 Nginx 而不是 Apache，您必须按照一定的顺序安装 PHP。

以下命令将 **自动安装 PHP 和 Apache**。如果您不需要/想要使用 Apache，请跳过使用此命令**：

```php
$ sudo apt-get install php5 php5-fpm

```

要避免安装 Apache，请按以下顺序执行以下命令：

```php
$ sudo apt-get install php5-common
$ sudo apt-get install php5-cgi
$ sudo apt-get install php5 php5-fpm

```

`php5-cgi` 软件包满足了本应由 Apache 满足的依赖项。

## 安装 Nginx

要安装 Nginx 网络服务器，我们需要执行以下命令：

```php
$ sudo add-apt-repository ppa:nginx/stable
$ sudo apt-get update
$ sudo apt-get install nginx

```

## 安装 MySQL

MySQL 可能是分布最广泛的 RDBMS 系统，市场份额超过 50%。由于我们将使用它来开发我们的项目，我们需要通过执行以下命令来安装它：

```php
$ sudo apt-get install mysql-server

```

### 注意

**下载示例代码**

您可以从您在 [`www.packtpub.com`](http://www.packtpub.com) 的账户下载您购买的所有 Packt 书籍的示例代码文件。如果您在其他地方购买了这本书，您可以访问 [`www.packtpub.com/support`](http://www.packtpub.com/support) 并注册以直接将文件通过电子邮件发送给您。

## 安装 Redis

**Redis** 是一个高级键值存储/缓存系统。我们将主要使用它来处理会话并缓存对象以提高应用程序的速度。让我们通过执行以下命令来安装它：

```php
$ sudo add-apt-repository ppa:chris-lea/redis-server
$ sudo apt-get update
$ sudo apt-get install redis-server
$ sudo apt-get install php5-redis

```

## 安装 MongoDB

MongoDB 是一个文档数据库（NoSQL 数据库）系统。我们将使用它来存储频繁访问的数据。让我们安装它：

```php
$ sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 7F0CEB10
$ echo 'deb http://downloads-distro.mongodb.org/repo/ubuntu-upstart dist 10gen' | sudo tee /etc/apt/sources.list.d/mongodb.list
$ sudo apt-get update
$ sudo apt-get install -y mongodb-org
$ sudo service mongodb start
$ sudo apt-get install php5-mongo

```

## 安装 Git

Git 是一个分布式版本控制系统，我们将使用它来跟踪应用程序的更改以及更多内容。我们将通过执行以下命令来安装 Git：

```php
$ sudo apt-get install git

```

### 小贴士

我强烈建议您尽可能使用所有软件的最新版本。

# 安装 Phalcon

现在我们已经安装了所有必需的软件，我们将继续安装 Phalcon。在我们继续之前，我们必须安装一些依赖项：

```php
$ sudo apt-get install php5-dev libpcre3-dev gcc make php5-mysql

```

对于 Windows 系统，以及有关如何在不同的系统上编译扩展的更多详细信息，请查看 [`phalconphp.com/en/download`](http://phalconphp.com/en/download) 上的最新文档。

现在，我们可以克隆仓库并编译我们的扩展：

```php
$ git clone --depth=1 git://github.com/phalcon/cphalcon.git
$ cd cphalcon/build
$ sudo ./install
$ echo 'extension=phalcon.so' | sudo tee /etc/php5/mods-available/phalcon.ini

$ sudo php5enmod phalcon
$ sudo service php5-fpm restart

```

如果一切顺利，您应该能够在 PHP 安装模块列表中看到 Phalcon：

```php
$ php -m | grep phalcon
```

# Apache 和 Nginx 的配置文件

我们将使用 `/var/www/learning-phalcon.localhost` 作为我们项目的默认目录，并将其称为 **根文件夹**。请创建此文件夹：

```php
$ sudo mkdir -p /var/www/learning-phalcon.localhost/public

```

当然，如果您愿意，您可以使用另一个文件夹。让我们在根目录下的 `public` 文件夹中创建一个测试文件，并包含一些 PHP 内容：

```php
$ cd /var/www/learning-phalcon.localhost/public
$ echo "<?php date();" > index.php

```

## Apache

让我们切换到 Apache 存储可用网站配置文件的默认目录，使用命令行：`$ cd /etc/apache2/sites-available/`。之后，执行以下步骤：

1.  使用您喜欢的编辑器，为 Apache 版本 < 2.4 创建一个名为 `learning-phalcon.localhost` 的文件，或为 Apache 版本 >= 2.4 创建一个名为 `learning-phalcon.localhost.conf` 的文件：

    ```php
    $ vim learning-phalcon.localhost.conf

    ```

1.  现在，将以下内容粘贴到该文件中：

    ```php
    <VirtualHost *:80>
        DocumentRoot "/var/www/learning-phalcon.localhost"
        DirectoryIndex index.php
        ServerName learning-phalcon.localhost
        ServerAlias www.learning-phalcon.localhost

        <Directory "/var/www/learning-phalcon.localhost/public">
            Options All
            AllowOverride All
            Allow from all
        </Directory>
    </VirtualHost>
    ```

1.  然后，切换到公共文件夹并添加一个名为 `.htaccess` 的文件到其中：

    ```php
    $ cd /var/www/learning-phalcon.localhost/public
    $ vim .htaccess

    ```

1.  然后，将以下内容添加到 `.htaccess` 文件中：

    ```php
    <IfModule mod_rewrite.c>
        RewriteEngine On
        RewriteCond %{REQUEST_FILENAME} !-d
        RewriteCond %{REQUEST_FILENAME} !-f
        RewriteRule ^(.*)$ index.php?_url=/$1 [QSA,L]
    </IfModule>
    ```

1.  如果您没有启用 `mod_rewrite`，这将不起作用。为此，执行此命令：

    ```php
    $ sudo a2enmod rewrite

    ```

1.  现在我们已经配置了虚拟主机，让我们启用它：

    ```php
    $ sudo a2ensite learning-phalcon.localhost
    $ sudo service apache2 reload

    ```

## hosts 文件

如果您打开浏览器并输入 `http://www.learning-phalcon.localhost/`，您将收到一个“找不到主机”或连接错误。这是因为没有为这个 **TLD**（顶级域的缩写）提供名称解析器。为了解决这个问题，我们编辑我们的 hosts 文件并添加此名称：

```php
$ echo "127.0.0.1 learning-phalcon.localhost www.learning-phalcon.localhost" | sudo tee /etc/hosts

```

重新启动您的浏览器并再次输入地址 `http://www.learning-phalcon.localhost/`。如果一切顺利，您应该会看到当前的日期/时间。

## Nginx

如果您选择使用 Nginx（我推荐这样做，尤其是因为它可以以更高的吞吐量服务更多的并发客户端，并且更有效地服务静态内容）而不是 Apache，以下是您需要执行的操作：

找到 Nginx 的 `config` 文件夹（在 Ubuntu 上，它安装在 `/etc/nginx/` 下）。在您的 `sites-available` 文件夹中创建一个名为 `learning-phalcon.localhost` 的文件（通过导航到 `/etc/nginx/sites-available`）：

```php
$ cd /etc/nginx/sites-available
$ vim learning-phalcon.localhost

```

现在，向其中添加以下内容：

```php
server {
    listen 80;
    server_name learning-phalcon.localhost;

    index index.php;
    set $root_path "/var/www/learning-phalcon.localhost/public";
    root $root_path;

    client_max_body_size 10M;

    try_files $uri $uri/ @rewrite;

    location @rewrite {
        rewrite ^/(.*)$ /index.php?_url=/$1;
    }

    location ~ \.php {
        fastcgi_index /index.php;
        fastcgi_pass unix:/var/run/php5-fpm.sock;
        fastcgi_intercept_errors on;
        include fastcgi_params;

        fastcgi_split_path_info ^(.+\.php)(/.*)$;

        fastcgi_param PATH_INFO $fastcgi_path_info;
        fastcgi_param PATH_TRANSLATED $document_root$fastcgi_path_info;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param DOCUMENT_ROOT $realpath_root;
        fastcgi_param SCRIPT_FILENAME $realpath_root/index.php;
    }

    location ~* ^/(css|img|js|flv|swf|download)/(.+)$ {
        root $root_path;
    }

    location ~ /\.ht {
        deny all;
    }
}

```

### 注意

在某些环境中，您可能需要编辑您的 `php.ini` 文件并将 `cgi.fix_pathinfo = 0` 设置。

然后，保存文件并重新启动 Nginx：

```php
$ sudo service nginx restart

```

请编辑并保存您的 hosts 文件（检查 *hosts 文件* 部分），然后打开您的浏览器并输入 `http://www.learning-phalcon.localhost/`。此时，您应该会看到一个显示当前日期/时间的页面。

安装和配置 PHP 和 Apache/Nginx 有许多可能的方法。如果您觉得我的方法不适合您的需求，请随意进行简单的 Google 搜索并选择一个更适合您的方法。

假设到目前为止一切顺利，我们将进一步学习一些关于 Phalcon 内部结构的知识。

# 理解框架的内部结构

在本节中，我将尝试简要介绍框架的常见部分。这里展示的大部分文本都是官方文档的一部分，你应该始终阅读。本节的想法是让你熟悉最常见的方法和组件，这将帮助你快速了解框架的工作方式。

### 小贴士

请注意，本书中的图片可能包含文本[`learning-phalcon.dev`](http://learning-phalcon.dev)。你需要忽略这些内容，并按照章节中建议的，使用`http://learning-phalcon.localhost`。

## 依赖注入

可能是 Phalcon 最强大的特性之一的是**依赖注入**（**DI**）。如果你对依赖注入一无所知，你应该至少阅读一下这个设计模式的维基百科页面[`en.wikipedia.org/wiki/Dependency_injection`](http://en.wikipedia.org/wiki/Dependency_injection)：

> *"依赖注入是一种软件设计模式，它实现了控制反转以解决依赖关系。注入是将依赖（一个服务或软件模块）传递给依赖对象（一个客户端）。服务成为客户端状态的一部分。将服务传递给客户端，而不是允许客户端构建或找到服务，是这种模式的基本要求。*
> 
> *依赖注入允许程序设计遵循依赖倒置原则。*

“依赖注入”这个术语是由马丁·福勒（Martin Fowler）提出的。

依赖注入的一个现实生活例子可能是以下情况：假设你去购物。在商场，你需要一个袋子来装你的杂货，但你离开家时忘了带。在这种情况下，你需要买一个袋子。在开发中，购买这个袋子可能相当昂贵。那么，如果你的门有一个扫描器，可以扫描你的身体以寻找袋子，并且只有在你有袋子的情况下才会打开，这可以称为依赖注入。

Phalcon 使用`\Phalcon\DI`组件，这是一个实现控制反转模式的组件。这减少了整体代码的复杂性。

框架本身或开发者可以注册服务。Phalcon 有许多内置组件，这些组件在 DI 容器中可用，如下所示：

+   请求和响应

+   记录器

+   密码

+   闪存

+   路由和配置

+   查看

+   缓存

+   会话

在 DI 中设置新的组件就像以下代码一样简单：

```php
<?php

$di = new Phalcon\DI();
// Lazy load
$di['mail'] = function() {
  return new \MyApp\Mail();
};
```

当你需要访问“mail”组件时，例如在一个控制器中，你可以简单地调用它：

```php
<?php

$mail = $this->getID()->get('mail');
// or
$mail = $this->getDI()->getMail();
```

如果需要创建自己的 DI，Phalcon 或必须实现`DiInterface`接口以替换 Phalcon 提供的接口，或者必须扩展当前的接口。

这些只是一些示例，以便你可以在我们开始项目时对 Phalcon 的 DI 有一个大致的了解。同时，请花些时间阅读官方文档，可以在 [`docs.phalconphp.com/en/latest/reference/di.html`](http://docs.phalconphp.com/en/latest/reference/di.html) 找到。

### 请求组件

请求组件可能是任何框架中最常用的组件之一。它处理任何 HTTP 请求（如 GET、POST 或 DELETE 等），同时也为 `$_SERVER` 变量提供了一些快捷方式。大多数时候，我们将在控制器中使用请求组件。Phalcon 文档（[`docs.phalconphp.com/en/latest/reference/mvc.html`](http://docs.phalconphp.com/en/latest/reference/mvc.html)）中说明了以下内容：

> *"控制器在模型和视图之间提供“流程”。控制器负责处理来自网页浏览器的传入请求，查询模型以获取数据，并将这些数据传递给视图进行展示。"*

在 Phalcon 中，所有控制器都应该扩展 `\Phalcon\Mvc\Controller` 组件，而我们想要通过 HTTP GET 访问的公共方法名称应该以 `Action` 后缀结尾。例如：

```php
<?php

class ArticleController extends \Phalcon\Mvc\Controller
{
  // Method for rendering the form to create an article
  public function createAction()
  {
  }

  // Method for searching articles
  public function searchAction()
  {
  }

  // This method will not be accessible via http GET
  public function search()
  {
  }
}
```

好的。那么，我们如何使用请求组件呢？很简单！你还记得我们在 DI 部分讨论的内置组件吗？请求组件就是其中之一。我们只需要获取 DI。以下是如何获取和使用请求组件的示例：

```php
<?php

class ArticleController extends \Phalcon\Mvc\Controller
{
  public function searchAction()
  {
    $request = $this->getDI()->get('request');
    // You can also use $request = $this->request; but I don't
    // recommend it because $this->request can be easily overwritten
    // by mistake and you will spend time to debug ... nothing.

    $request->getMethod(); // Check the request method
    $request->isAjax(); // Checks if the request is an ajax request
    $request->get(); // Gets everything, from the request (GET, POST, DELETE, PUT)
    $request->getPost(); // Gets all the data submitted via POST method
    $request->getClientAddress(); // Return the client IP
  }
}
```

这些只是请求组件中内置的一些常见方法。让我们继续下一个重要组件——响应。

### 响应组件

那么，这个组件能做什么呢？嗯，几乎与响应或输出相关的一切都能做。使用它，我们可以设置头，执行重定向，发送 cookie，设置内容等等。以下是该组件的一些常见方法列表：

```php
<?php

public function testRedirectAction()
{
  $response = $this->getDI()->get('response');
   // or you can use $this->response directly

  // Redirect the user to another url
  $this->view->disable();
  return $response->redirect('http://www.google.com/', true);
}
```

`redirect` 方法接受三个参数：一个位置（字符串），如果是一个外部重定向（这是一个布尔类型，默认为 `false`），以及一个状态码（HTTP 状态码范围）。以下代码行是重定向方法：

```php
  <?php
```

```php
  /**
  * Redirect by HTTP to another action or URL
  *
  * @param string $location
  * @param boolean $externalRedirect
  * @param int $statusCode
  * @return \Phalcon\Http\ResponseInterface
  */
  public function redirect($location, $externalRedirect, $statusCode);
```

另一个有用的方法是 `setHeader` 方法：

```php
<?php

public function testSetHeaderAction()
{
  $this->response->setHeader('APIKEY', 'AWQ23XX258561');
}
```

上述示例设置了一个名为 `APIKEY` 的头，其值为 `AWQ23XX258561`。在开发 API 时发送头是一个常见的做法。你可以使用此方法发送任何类型的头并覆盖当前头。

内容相关方法：`setContent()` 和 `setJsonContent()`。让我们以以下代码为例：

```php
<?php

public function testContentAction()
{
  // First, we disable the view if there is any
  $this->view->disable();

  // Set a plain/text or html content
  $this->response->setContent('I love PhalconPHP');

  // OR

  // Set a json content (this will return a json object)
  $this->response->setJsonContent(array(
    'framework' => 'PhalconPHP'
    'versions' => array(
      '1.3.2',
      '1.3.3',
      '2.0.0'
    )
  ));

  // We send the output to the client
  return $this->response->send();
}
```

当你需要发送任何 JSON 内容时，你应该使用响应对象中的内置方法将头设置为 `application/json`：

```php
<?php

$this->response->setContentType('application/json', 'UTF-8');
```

现在我们已经了解了响应/请求组件的基础知识，我们可能会发现自己处于需要记录不同情况的位置，例如错误。为此，我们需要检查记录器组件。

### 记录器组件

在生产环境中，我们不能承担向客户端抛出错误或空白页面的风险。我们将避免这种情况，并将错误记录在日志文件中。您将在下一章中了解更多关于此内容。总结一下，我们将实现一个自定义的记录器到 DI 中，捕获异常，然后记录它们。例如，执行以下步骤：

1.  使用以下代码在 DI 中设置自定义记录器：

    ```php
    <?php

    $di['logger'] = function() {
      $error_file = __DIR__.'/../logs/'.date("Ymd_error").'log';
      return new \Phalcon\Logger\Adapter\File($error_file, array('mode' => 'a+'));
    };
    ```

1.  创建一个会抛出异常的方法，捕获它，并记录它，如下所示：

    ```php
    <?php

    public function testLoggerAction()
    {
      try {
        $nonExistingComponent = $this->getDI()->get('nonExistingComponent');
        $nonExistingComponent->executeNonExistingMethod();
      } catch (\Exception $e) {
        $this->logger->error($e->getMessage());
        return $this->response->redirect('error/500.html');
      }
    }
    ```

在前面的例子中，我们尝试执行一个不存在的方法，我们的代码将抛出一个异常，我们将其捕获并记录，然后重定向用户到一个友好的错误页面，`error/500.html`。您会注意到我们的记录器组件调用了一个名为`error`的方法。还有其他实现的方法，如`debug`、`info`、`notice`、`warning`等等。

`logger`组件可以是事务性的。（Phalcon 将日志临时存储在内存中，稍后将其写入相关适配器。）例如，考虑以下代码片段：

```php
<?php

$this->logger->begin();

$this->logger->error('Ooops ! Error !');
$this->logger->warning('A warning message');

$this->logger->commit();
```

### 加密组件

如果有人需要在您的端加密数据并解密它，加密是一个非常有用的组件。你可能想要使用加密组件的情况之一是将数据通过 HTTP `get`方法发送或保存敏感信息到您的数据库中。

此组件具有许多内置方法，例如`encrypt`、`decrypt`、`getAvailableChiphers`、`setKey`、`getKey`等等。以下是在 HTTP `get`方法中使用加密组件的示例。

首先，我们覆盖 DI（依赖注入），然后向它传递一个键以避免每次都设置它：

```php
<?php

$di['crypt'] = function () {
  $crypt = new \Phalcon\Crypt();
  $crypt->setKey('0urSup3rS3cr3tK3y!?');

  return $crypt;  
};

public function sendActivationAction()
{
  $activation_code = $this->crypt->encryptBase64('1234');
  $this->view->setVar('activation_code', $activation_code);
}

public function getActivationAction($code)
{
  if ('1234' == $this->crypt->decryptBase64($code)) {
    $this->flash->success('The code is valid ');
  } else {
    $this->flash->error('The code is invalid');
  }
}
```

当然，您可能永远不会以这种方式使用它。前面的例子只是展示了此组件的强大功能。您可能已经注意到有一个新的 DI 方法名为 flash。我们将在下一节讨论它。

### flash 组件

此组件用于向客户端发送通知，并告知他们组件动作的状态。例如，当用户在我们的网站上完成注册或提交联系表单后，我们可以发送一条成功消息。

有两种类型的 flash 消息——直接和会话，两者都在 DI 中可用。直接方法直接输出消息，不能在未来请求中加载。相反，会话方法将消息存储在会话中，打印后自动清除。

假设您有一个名为 register 的页面，并且您在同一页面上提交数据，以下是 flash 直接和 flash 会话的常见用法：

```php
public function registerAction()
{
  // … code
  if ($errors) {  
    $this->flash->warning('Please fix the following errors: ');
    foreach($errors as $error) {
      $this->flash->error($error);
    }
  } else {
    $this->flash->success('You have successfully registered on our website');
  }
}  
```

在我们的视图中，我们将使用`getContent()`方法或模板引擎**Volt**中的`content()`来渲染消息（我们将在本章后面讨论这一点）。

如果我们需要将用户重定向到另一个页面（让我们称它为`registerSuccess`），那么我们需要使用 flash 会话方法；否则，消息将不会显示。

```php
<?php

public function registerAction()
{
  // render our template
}
```

`register`模板将包含一个方法为`post`且`action`指向`create`方法的表单。`create`方法看起来可能像这样：

```php
<?php

public function createAction()
{
  if ($errors) {  
    $this->flashSession->warning('Please fix the following errors: ');
    foreach($errors as $error) {
      $this->flashSession->error($error);
    }
  } else {
    $this->flashSession->success('You have successfully registered on our website');
  }

  return $this->response->redirect('/register');
}
```

在前面的示例中，我们使用`flashSession`方法在会话中设置消息，并将用户重定向回注册页面。为了在我们的视图中渲染这些消息，我们需要调用`flashSession()->output();`方法。

### 小贴士

推荐的方式是使用分发器来转发请求，而不是使用重定向。如果你使用重定向，用户将丢失他们在表单中填写的所有数据。

### 路由组件

路由组件帮助我们将友好的 URL 映射到我们的控制器和操作。

默认情况下，如果你的 Web 服务器启用了重写模块，你将能够通过以下方式访问名为`Post`的控制器和`read`操作：`http://www.learning-phalcon.localhost/post/read`。我们的代码可能看起来像这样：

```php
<?php

class PostController extends \Phalcon\Mvc\Controller
{
  public function readAction()
  {
    // get the post
  }
}
```

然而，有时如果你需要将 URL 翻译成多种语言，或者需要以不同于代码中定义的方式命名 URL，这段代码可能就不适用了。以下是路由组件的使用示例：

```php
<?php

$router = new \Phalcon\Mvc\Router();
// Clear the default routes
$router->clear();

$st_categories = array(
  'entertainment',
  'travel',
  'video'
);

$s_categories = implode('|', $st_categories);

$router->add('#^/('.$s_categories.')[/]{0,1}$#', array(
    'module' => 'frontend',
    'controller' => 'post',
    'action' => 'findByCategorySlug',
    'slug' => 0
));
```

在前面的示例中，我们将所有类别映射到`post`控制器和`findByCategorySlug`操作。路由组件允许我们使用正则表达式来定义 URL。使用`preg_match`，这可以表示如下

```php
$url = 'http://www.learning-phalcon.localhost/video';
preg_match('#^/(entertainment|travel|video)[/]{0,1}$#', $url);
```

通过访问`http://www.learning-phalcon.localhost/video`，请求将被转发到 post 控制器的`findByCategorySlug`操作：

```php
<?php

class PostController extends \Phalcon\Mvc\Controller
{
  public function findByCategorySlug()
  {
    $slug = $this->dispatcher->getParam('slug', array('string', 'striptags'), null);

    // We access our model (entity) to get all the posts from this category
    $posts = Posts::findByCategorySlug($slug);

    if ($posts->count() > 0) {
      $this->view->setVar('posts', $posts);
    } else {
      throw new \Exception('There are no posts', 404);
    }
  }
}
```

`getParam()`方法有三个参数。第一个是我们正在搜索的名称，第二个参数是可以自动应用的一组过滤器，第三个参数是在请求的名称不存在或未设置时的默认值。

我们将在下一章讨论模型。这只是一个简单的示例，说明你可以如何使用路由。

路由还支持对`request`方法的预检查。你可能习惯于检查方法是否为 POST、DELETE、PUT 或 GET，如下所示：

```php
<?php

if ($_SERVER['REQUEST_METHOD'] == 'post') {
  // process the information
}
```

虽然这是完全正确的，但它对我们的代码来说并不友好。Phalcon 的路由具有这种能力，你可以添加你期望的正确类型的请求，而无需在代码中进行检查：

```php
<?php

// Add a get route for register method within the user controller
$router->addGet('register', 'User::register');

// Add a post route for create method, from the user controller
$router->addPost('create', 'User::create');
```

这只是路由的基本用法。一如既往，请阅读文档以了解有关此组件的所有信息。

### 小贴士

你可以在官方文档中了解更多关于路由的信息，请访问[`docs.phalconphp.com/en/latest/reference/routing.html`](http://docs.phalconphp.com/en/latest/reference/routing.html)。

### 配置组件

此组件可以通过适配器处理各种格式的配置文件。Phalcon 为此提供了两个内置适配器，分别是 INI 和 Array。使用 INI 文件可能从来都不是一个好主意。因此，我建议你使用原生数组。

这些文件可以或需要存储什么类型的数据？嗯，几乎是我们应用中需要的所有全局数据，例如数据库连接参数。在以前的日子里，我们使用`$_GLOBALS`（一个大的安全问题），或者我们使用`define()`方法，然后逐渐开始全局使用它。

这里是一个`config`文件的示例，以及我们如何使用它：

```php
<?php

$st_settings = array(
  'database' => array(
    'adapter'  => 'Mysql',
    'host'     => 'localhost',
    'username' => 'john',
    'password' => 'johndoe',
    'dbname'     => 'test_database',
  ),
  'app' => array(
    'name' => 'Learning Phalcon'
  )
);

$config = new \Phalcon\Config($st_settings);

// Get our application name:
echo $config->app->name; // Will output Learning Phalcon
```

可以使用`toArray()`方法将`config`对象转换回数组：

```php
<?php

$st_config = $config->toArray();
echo $config['app']['name']; // Will output Learning Phalcon
```

对于此对象，另一个有用的方法是`merge`方法。如果我们有多个配置文件，我们可以轻松地将它们合并成一个对象：

```php
<?php

$config = array(
  'database' => array(
    'adapter'  => 'Mysql',
    'host'     => 'localhost',
    'dbname'     => 'test_database',
  ),
  'app' => array(
    'name' => 'Learning Phalcon'
  )
);

$config2 = array(
  'database' => array(
    'username' => 'john',
    'password' => 'johndoe',
  )
```

现在，`$config`对象将具有与之前相同的内容。

### 提示

还有两个尚未实现的适配器（YAML 和 JSON），但如果你克隆 Phalcon 的孵化器存储库（[`github.com/phalcon/incubator`](https://github.com/phalcon/incubator)），你可以使用它们。此存储库包含一系列适配器/辅助工具，这些工具可能很快将被集成到 Phalcon 中。

### 视图组件

此组件用于渲染我们的模板。默认情况下，模板具有`.phtml`扩展名，并包含 HTML 和 PHP 代码。以下是一些使用视图的示例：

1.  首先，我们使用以下代码片段在 DI 中设置视图：

    ```php
    <?php

    $di['view'] = function () use ($config) {
      $view = \Phacon\Mvc\View();
      // Assuming that we hold our views directory in the configuration file
      $view->setViewsDir($config->view->dir);

      return $view;
    };  
    ```

1.  现在，我们可以如下使用此服务：

    ```php
    <?php

    class PostControler extends \Phalcon\Mvc\Controller
    {
      public function listAction()
      {
        // Retrieve posts from DB
        $posts = Post:find();
        $this->view->setVar('pageTitle', 'Posts');
        $this->view->setVar('posts', $posts);
      }
    }
    ```

1.  接下来，我们需要创建一个视图模板，它必须看起来像这样：

    ```php
    <!DOCTYPE html>
    <html>
    <head>
    <meta charset="UTF-8">
    <title><?php echo $pageTitle; ?></title>
    </head>
    <body>
    <?php foreach($posts as $post) { ?>
      <p><?php echo $post->getPostTitle(); ?></p>
      <p><?php echo $post->getPostContent(); ?></p>
    <?php } ?>
    </body>
    </html>
    ```

简单，不是吗？此组件还支持分层渲染。你可以有一个基本布局，一个用于帖子的通用模板，以及一个用于单个帖子的模板。让我们以以下目录结构为例：

```php
app/views/
- index.phtml
- post/detail.phtml
```

Phalcon 首先渲染`app/views/index.phtml`。然后，当我们从帖子控制器请求`detailAction()`时，它将渲染`app/views/post/details.phtml`。主布局可以包含类似以下代码的内容：

```php
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Learning Phalcon</title>
</head>
<body>
<?php echo $this->getContent(); ?>
</body>
</html>
```

此外，`details.phtml`模板将包含以下内容：

```php
<?php foreach($posts as $post) { ?>
  <p><?php echo $post->getPostTitle(); ?></p>
  <p><?php echo $post->getPostContent(); ?></p>
<?php } ?>
```

此组件还允许你选择不同的模板来设置渲染级别，禁用或启用视图，以及更多。

Phalcon 有一个名为 Volt 的内置模板引擎。如果你熟悉 PHP 模板引擎，如**Smarty**或**Twig**，你肯定会想使用它们。Volt 几乎与 Twig 相同，你会发现它非常有用——它受到了**Jinja**的启发（[`jinja.pocoo.org/`](http://jinja.pocoo.org/)）。你甚至可以使用自己的模板引擎，或者任何你能在那里找到的其他模板引擎。

为了启用 Volt 模板引擎，我们需要对我们的视图服务进行一些小的修改，并创建一个 Volt 服务；以下是这样做的方法：

```php
<?php

$di['voltService'] = function($view, $di) use ($config) {

    $volt = new \Phalcon\Mvc\View\Engine\Volt($view, $di);

    if (!is_dir($config->view->cache->dir)) {
        mkdir($config->view->cache->dir);
    }

    $volt->setOptions(array(
        "compiledPath" => $config->view->cache->dir,
        "compiledExtension" => ".compiled",
        "compileAlways" => false
    ));

    $compiler = $volt->getCompiler();

    return $volt;
};

// First, we setup the view in the DI
$di['view'] = function () use ($config) {
  $view = \Phacon\Mvc\View();
  $view->setViewsDir($config->view->dir);
  $view->registerEngines(array(
    '.volt' => 'voltService'
  ));

  return $view;
};
```

通过添加此修改和`voltService`，我们现在可以使用此模板引擎。从继承的角度来看，Volt 表现得有点不同。我们首先需要定义一个主布局，并带有命名块。然后，其余的模板应该扩展主布局，并且我们需要将内容放在与主布局相同的块中。在我们查看一些示例之前，我将简要介绍一下 Volt 的语法，具体如下。

+   输出数据或输出内容的语法：

    ```php
    {{ my_content }}
    ```

+   定义块的语法：

    ```php
    {% block body %} Content here {% endblock %}
    ```

+   扩展模板的语法（这应该是您模板中的第一行）：

    ```php
    {% extends 'layouts/main.volt' %}
    ```

+   包含文件的语法：

    ```php
    {% include 'common/sidebar.volt' %}
    ```

+   包含文件并传递变量的语法：

    ```php
    {% include 'common/sidebar' with{'section':'homepage'} %}
    ```

    ### 小贴士

    请注意缺少的扩展名。如果您传递变量，*必须*省略扩展名。

+   控制结构（`for`，`if`，`else`）的语法：

    ```php
    {% for post in posts %}
      {% if post.getCategorySlug() == 'entertainment' %}
        <h3 class="pink">{{ post.getPostTitle() }}</h3>
      {% else %}
        <h3 class="normal">{{ post.getPostTitle() }}</h3>
      {% endif %}
    {% endfor %}
    ```

+   循环上下文的语法：

    ```php
    {% for post in posts %}
      {% if loop.first %}
        <h1>{{ post.getPostTitle() }}</h1>
      {% endif %}
    {% endif %}
    ```

+   赋值的语法：

    ```php
    {% set title = 'Learning Phalcon' %}
    {% set cars = ['BMW', 'Mercedes', 'Audi'] %}
    ```

列表很长。此外，您还可以使用表达式、比较运算符、逻辑运算符、过滤器等。让我们写一个简单的模板来看看它是如何工作的：

```php
<!-- app/views/index.volt -->
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>{% block pageTitle %}Learning Phalcon{% endblock%}</title>
</head>
<body>
  <div class='header'>{% block header %}Main layout header{% endblock%}</div>
  <div class='content'>{% block content %}This is the main layout content{% endblock %}</div>
</body>
</html>

<!-- app/views/post/detail.volt 
{% extends 'index.volt' %}

{% block pageTitle  %}
  {{ post.getPostTitle() }}
{% endblock %}

{% block header %}
  Post layout
{% endblock %}

{% block content %}
  <p>{{ post.getPostContent() }}</p>
{% endblock%}
```

### 注意

您可以在[`docs.phalconphp.com/en/latest/reference/views.html`](http://docs.phalconphp.com/en/latest/reference/views.html)查看视图组件的完整文档，以及 Volt 的[`docs.phalconphp.com/en/latest/reference/volt.html`](http://docs.phalconphp.com/en/latest/reference/volt.html)。

### 会话组件

此组件提供面向对象的包装来访问会话数据。要启动会话，我们需要将服务添加到 DI 容器中：

```php
<?php

$di['session'] = function () {
  $session = new Phalcon\Session\Adapter\Files();
  $session->start();
  return $session;  
};
```

以下是与会话一起工作的代码示例：

```php
<?php

public function testSessionAction()
{
  // Set a session variable
  $this->session->set('username', 'john');

  // Check if a session variable is defined
  if ($this->session->has('username')) {
    $this->view->setVar('username', $this->session->get('username'));
  }

  // Remove a session variable
  $this->session->remove('username');

  // Destroy the session
  $this->session->destroy();
}
```

如果您检查 Phalcon 的孵化器，有许多可用的适配器，例如 Redis、数据库、Memcache 和 Mongo。您也可以实现自己的适配器。

### 注意

您可以在[`docs.phalconphp.com/en/latest/reference/session.html`](http://docs.phalconphp.com/en/latest/reference/session.html)查看官方文档。

### 缓存组件

为了提高某些应用程序的性能，您需要缓存数据。例如，我们可以缓存帖子的查询结果。为什么？想象一下有 100 万次查看或帖子。通常，您将查询数据库，但这意味着 100 万次查询（如果您在使用它，并且对于 ORM——这意味着至少 300 万次查询）。为什么？当您查询时，ORM 将表现得像这样：

1.  它将检查表是否存在于信息模式中：

    ```php
    SELECT IF(COUNT(*)>0, 1 , 0) FROM `INFORMATION_SCHEMA`.`TABLES` WHERE `TABLE_NAME`='user'
    ```

1.  然后，它将检查是否正在执行表的“描述”：

    ```php
    DESCRIBE `user`
    ```

1.  然后，是否正在执行实际的查询：

    ```php
    SELECT * FROM user.
    ```

1.  如果`用户`表有关系，ORM 将为每个关系重复执行前面的每个步骤。

为了解决这个问题，我们将帖子对象保存到我们的缓存系统中。

个人来说，我使用 Redis 和 Igbinary。Redis 可能是最强大的工具，因为它将数据存储在内存中，并在磁盘上保存数据以实现冗余。这意味着每次你从缓存请求数据时，你都会从内存中获取它。Igbinary（[`pecl.php.net/package/igbinary`](https://pecl.php.net/package/igbinary)）是标准 php 序列化器的替代品。以下是一个示例缓存服务：

```php
<?php

$di['redis'] = function () {
    $redis = new \Redis();
    $redis->connect(
        '127.0.0.1',
        6379
    );

    return $redis;
};

$di['cache'] = function () use ($di, $config) {
    $frontend = new \Phalcon\Cache\Frontend\Igbinary(array(
        'lifetime' => 86400
    ));
    $cache = new \Phalcon\Cache\Backend\Redis($frontend, array(
        'redis' => $di['redis'],
        'prefix' => 'learning_phalcon'
    ));

    return $cache;
};
```

缓存组件有以下常用方法：

```php
<?php

// Save data in cache
$this-cache->save('post', array(
  'title' => 'Learning Phalcon',
  'slug' => 'learning-phalcon',
  'content' => 'Article content'
));

// Get data from cache
$post = $this->cache->get('post');

// Delete data from cache
$this->cache->delete('post');
```

# 摘要

在本章中，我们安装了所需的软件，创建了 Web 服务器的配置文件，并且你对 Phalcon 的内部结构有了一些了解。在接下来的章节中，我们将通过示例学习，一切都会变得更加清晰。

请慢慢来，在继续之前，先阅读一下你不太熟悉的任何内容。

在下一章中，我们将探讨如何设置我们的项目的 MVC 结构和环境。
