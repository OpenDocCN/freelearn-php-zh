# 第六章。适应 MVC

Web 应用程序比我们迄今为止构建的更复杂。您添加的功能越多，代码的维护和理解就越困难。这就是为什么以有组织的方式组织代码至关重要。您可以设计自己的结构，但就像面向对象编程一样，已经存在一些设计模式试图解决这个问题。

**MVC**（**模型-视图-控制器**）一直是 Web 开发者的最爱模式。它帮助我们分离 Web 应用程序的不同部分，即使对于初学者来说，代码也易于理解。我们将尝试重构我们的书店示例以使用 MVC 模式，您将意识到在那之后您可以多么快速地添加新功能。

在本章中，您将学习以下内容：

+   使用 Composer 管理依赖关系

+   为您的应用程序设计路由器

+   将您的代码组织成模型、视图和控制器

+   使用 Twig 作为模板引擎

+   依赖注入

# MVC 模式

到目前为止，每次我们添加新功能时，我们都会为该特定页面添加一个包含 PHP 和 HTML 混合的新 PHP 文件。对于具有单一目的的代码块，并且我们必须重复使用，我们创建了函数并将它们添加到函数文件中。即使是像我们这样非常小的 Web 应用程序，代码也开始变得非常混乱，代码的重用能力并不像它本可以那样有帮助。现在想象一个具有大量功能的程序：那几乎就是混乱本身。

问题并没有在这里停止。在我们的代码中，我们在一个文件中混合了 HTML 和 PHP 代码。当我们试图更改 Web 应用程序的设计，或者即使我们想要在所有页面上进行非常小的更改，例如更改页面的菜单或页脚时，这会给我们带来很多麻烦。应用程序越复杂，我们遇到的问题就越多。

MVC 作为一个模式出现，帮助我们划分应用程序的不同部分。这些部分被称为模型、视图和控制器。**模型**管理数据和/或业务逻辑，**视图**包含我们的响应模板（例如，HTML 页面），而**控制器**协调请求，决定使用哪些数据以及如何渲染适当的模板。我们将在本章的后续部分中详细介绍它们。

# 使用 Composer

尽管在实现 MVC 模式时这不是一个必要的组件，但 Composer 在过去几年中一直是任何 PHP Web 应用程序不可或缺的工具。这个工具的主要目标是帮助您管理应用程序的依赖关系，即我们应用中需要使用的第三方库（代码）。我们可以通过创建一个列出它们的配置文件，并在您的命令行中运行一个命令来实现这一点。

您需要在您的开发机器上安装 Composer（见第一章，*设置环境*)。请确保您已执行以下命令：

```php
$ composer –version

```

这应该会返回您 Composer 安装的版本。如果它没有这样做，请返回安装部分以解决问题。

## 管理依赖项

如我们之前所述，Composer 的主要目标是管理依赖项。例如，我们已经实现了我们的配置读取器，即`Config`类，但如果我们知道有人实现了更好的版本，我们就可以直接使用他们的版本而不是重新发明轮子；只需确保他们允许这样做即可！

### 注意

**开源**

开源指的是开发者编写的并与社区共享的代码，以便其他人可以无限制地使用。实际上存在不同类型的许可证，有些比其他提供更大的灵活性，但基本思想是我们可以在我们的应用程序中重用其他开发者编写的库。这有助于社区在知识上的增长，因为我们可以从他人的工作中学习，改进它，并在之后分享它。

我们已经实现了一个不错的配置读取器，但我们的应用程序中还有其他元素需要完成。让我们利用 Composer 来重用他人的库。向我们的项目添加依赖项有几种方法：在我们的命令行中执行命令，或手动编辑配置文件。由于我们还没有 Composer 的配置文件，让我们使用第一种方法。在您的应用程序根目录中执行以下命令：

```php
$ composer require monolog/monolog

```

此命令将显示以下结果：

```php
Using version ¹.17 for monolog/monolog
./composer.json has been created
Loading composer repositories with package information
Updating dependencies (including require-dev)
 - Installing psr/log (1.0.0)
 Downloading: 100%

 - Installing monolog/monolog (1.17.2)
 Downloading: 100%
...
Writing lock file
Generating autoload files

```

使用此命令，我们要求 Composer 将库`monolog/monolog`添加为我们的应用程序的依赖项。执行后，我们现在可以看到目录中的一些变化：

+   我们有一个名为`composer.json`的新文件。这是配置文件，我们可以在此添加我们的依赖项。

+   我们有一个名为`composer.lock`的新文件。这是一个 Composer 用来跟踪已安装的依赖项及其版本的文件。

+   我们有一个名为`vendor`的新目录。这个目录包含 Composer 下载的依赖项代码。

命令的输出还显示了一些额外信息。在这种情况下，它说它下载了两个库或包，尽管我们只请求了一个。原因是所需的包也包含其他依赖项，这些依赖项由 Composer 解决。请注意 Composer 下载的版本；由于我们没有指定任何版本，Composer 选择了可用的最新版本，但您始终可以尝试写入您需要的特定版本。

我们需要另一个库，在这种情况下是`twig/twig`。让我们使用以下命令将其添加到我们的依赖项列表中：

```php
$ composer require twig/twig

```

此命令将显示以下结果：

```php
Using version ¹.23 for twig/twig
./composer.json has been updated
Loading composer repositories with package information
Updating dependencies (including require-dev)
 - Installing twig/twig (v1.23.1)
 Downloading: 100%

Writing lock file
Generating autoload files

```

如果我们检查`composer.json`文件，我们将看到以下内容：

```php
{
    "require": {
        "monolog/monolog": "¹.17",
        "twig/twig": "¹.23"
    }
}
```

该文件只是一个包含我们应用程序配置的 JSON 映射；在这种情况下，是我们安装的两个依赖项的列表。如您所见，依赖项的名称遵循一个模式：由斜杠分隔的两个单词。第一个单词指的是开发库的供应商。第二个单词是库本身的名称。依赖项有一个版本，可以是确切的版本号，如本例所示，或者可以包含通配符字符或标签名称。您可以在[`getcomposer.org/doc/articles/aliases.md`](https://getcomposer.org/doc/articles/aliases.md)上了解更多信息。

最后，如果您想添加另一个依赖项或以任何方式编辑`composer.json`文件，您应该在命令行中运行`composer update`，或者在`composer.json`文件所在的任何位置，以便更新依赖项。

## 使用 PSR-4 的自动加载器

在前面的章节中，我们也为我们应用程序添加了一个自动加载器。由于我们现在使用的是他人的代码，我们需要知道如何加载他们的类。很快，开发者意识到如果没有标准，这种场景几乎无法管理，因此他们提出了一些大多数开发者都遵循的标准。您可以在[`www.php-fig.org`](http://www.php-fig.org)找到关于这个主题的大量信息。

现在，PHP 有两个主要的自动加载标准：**PSR-0**和**PSR-4**。它们非常相似，但我们将实现后者，因为它是最新的发布标准。这个标准基本上遵循我们在讨论命名空间时已经介绍的内容：类的命名空间必须与它所在的目录相同，类的名称应该是文件名，后跟扩展名`.php`。例如，`src/Domain/Book.php`文件包含在`Bookstore\Domain`命名空间内的`Book`类。

使用 Composer 的应用程序应遵循这些标准之一，并在它们各自的`composer.json`文件中注明它们使用的是哪一个。这意味着 Composer 知道如何自动加载它自己的应用程序文件，因此当我们下载外部库时，我们不需要关心它。为了指定这一点，我们编辑我们的`composer.json`文件，并添加以下内容：

```php
{
    "require": {
        "monolog/monolog": "¹.17",
        "twig/twig": "¹.23"
    },
 "autoload": {
 "psr-4": {
 "Bookstore\\": "src"
 }
 }
}
```

上述代码表示我们将使用 PSR-4 在我们的应用程序中，并且所有以`Bookstore`开头的命名空间都应该在`src/`目录内找到。这正是我们的自动加载器已经做到的，但现在简化为配置文件中的几行。我们现在可以安全地删除我们的自动加载器及其任何引用。

Composer 生成一些映射来帮助加快类的加载速度。为了将配置文件中添加的新信息更新到这些映射中，我们需要运行之前运行的 `composer update` 命令。这次，输出将告诉我们没有需要更新的包，但将再次生成自动加载文件：

```php
$ composer update
Loading composer repositories with package information
Updating dependencies (including require-dev)
Nothing to install or update
Writing lock file
Generating autoload files

```

## 添加元数据

为了知道你定义的依赖项库在哪里，Composer 维护着一个包和版本的仓库，称为 **Packagist**。这个仓库为开发者提供了大量有用的信息，例如给定包的所有版本、作者、一些关于包功能的描述（或指向该信息的网站），以及该包将下载的依赖项。你还可以通过名称或类别浏览包。

但 Packagist 是如何知道这个文件的？这都要归功于 `composer.json` 文件本身。在那里，你可以以 Composer 理解的格式定义你应用程序的所有元数据。让我们看看一个例子。将以下内容添加到你的 `composer.json` 文件中：

```php
{
    "name": "picahielos/bookstore",
    "description": "Manages an online bookstore.",
    "minimum-stability": "stable",
    "license": "Apache-2.0",
    "type": "project",
    "authors": [
        {
            "name": "Antonio Lopez",
            "email": "antonio.lopez.zapata@gmail.com"
        }
    ],
    // ...
}
```

配置文件现在包含了遵循 Composer 约定的包名称：供应商名称，斜杠，包名称——在这个例子中，`picahielos/bookstore`。我们还添加了描述、许可证、作者和其他元数据。如果你在公共仓库（如 GitHub）中有代码，添加这个 `composer.json` 文件将允许你访问 Packagist 并插入你仓库的 URL。Packagist 将你的代码作为一个新的包添加，从你的 `composer.json` 文件中提取信息。它将根据你的标签或分支显示可用的版本。为了了解更多信息，我们鼓励你访问官方文档[`getcomposer.org/doc/04-schema.md`](https://getcomposer.org/doc/04-schema.md)。

## index.php 文件

在 MVC 应用程序中，我们通常有一个文件来获取所有请求，并根据 URL 将它们路由到特定的控制器。这种逻辑通常可以在我们根目录中的 `index.php` 文件中找到。我们已经有了一个，但随着我们适应 MVC 模式，我们不再需要当前的 `index.php`。因此，你可以安全地将其替换为以下内容：

```php
<?php

require_once __DIR__ . '/vendor/autoload.php';
```

现在这个文件唯一要做的就是包含处理所有自动加载的 Composer 代码的文件。稍后，我们将在这里初始化一切，例如数据库连接、配置读取器等等，但现在让我们先留空。

# 处理请求

如同你可能在之前的章节中回忆的那样，Web 应用程序的主要目的是处理来自客户端的 HTTP 请求并返回响应。如果你的应用程序的主要目标是这个，那么管理请求和响应应该是你代码中的一个重要部分。

PHP 是一种可以用于脚本的编程语言，但它的主要用途是 Web 应用程序。由于这个原因，语言自带了很多用于管理请求和响应的辅助工具。尽管如此，原生的方式并不理想，并且作为好的面向对象开发者，我们应该提出一套有助于此的类。这个小项目的主要元素——仍然在你的应用内部——是请求和路由器。让我们开始吧！

## 请求对象

随着我们开始我们的迷你框架，我们需要稍微改变一下我们的目录结构。我们将为所有与框架相关的类创建一个`src/Core`目录。由于上一章中的配置读取器也是框架的一部分（而不是用户的功能），我们应该也将`Config.php`文件移动到这个目录。

首先要考虑的是请求看起来是什么样子。如果你还记得第二章，*使用 PHP 的 Web 应用程序*，一个请求基本上是一个发送到 URL 的消息，并且有一个方法——目前是 GET 或 POST。URL 同时由两部分组成：Web 应用的域名，即你的服务器名称，以及服务器内请求的路径。例如，如果你尝试访问`http://bookstore.com/my-books`，第一部分`http://bookstore.com`将是域名，而`/my-books`将是路径。实际上，`http`不会是域名的一部分，但对我们应用来说，我们不需要那么细粒度的信息。你可以从 PHP 为每个请求填充的全局数组`$_SERVER`中获取这些信息。

我们的`Request`类应该为这三个元素中的每一个都有一个属性，然后是一系列获取器和一些对用户有用的其他辅助方法。此外，我们应该在构造函数中从`$_SERVER`初始化所有属性。让我们看看它会是怎样的：

```php
<?php

namespace Bookstore\Core;

class Request {
    const GET = 'GET';
    const POST = 'POST';

    private $domain;
    private $path;
    private $method;

    public function __construct() {
        $this->domain = $_SERVER['HTTP_HOST'];
        $this->path = $_SERVER['REQUEST_URI'];
        $this->method = $_SERVER['REQUEST_METHOD'];
    }

    public function getUrl(): string {
        return $this->domain . $this->path;
    }

    public function getDomain(): string {
        return $this->domain;
    }

    public function getPath(): string {
        return $this->path;
    }

    public function getMethod(): string {
        return $this->method;
    }

    public function isPost(): bool {
        return $this->method === self::POST;
    }

    public function isGet(): bool {
        return $this->method === self::GET;
    }
}
```

我们可以在前面的代码中看到，除了每个属性的获取器之外，我们还添加了`getUrl`、`isPost`和`isGet`方法。用户可以使用现有的获取器找到相同的信息，但鉴于它们将非常需要，总是好的让它们更容易使用。另外请注意，属性来自`$_SERVER`数组的值：`HTTP_HOST`、`REQUEST_URI`和`REQUEST_METHOD`。

## 从请求中过滤参数

请求的另一个重要部分是来自用户的信息，即 GET 和 POST 参数，以及 cookies。与`$_SERVER`全局数组一样，这些信息来自`$_POST`、`$_GET`和`$_COOKIE`，但总是好的避免直接使用它们，没有过滤，因为用户可能会发送恶意代码。

现在，我们将实现一个表示可以过滤的映射（键值对）的类。我们将称之为 `FilteredMap`，并将其包含在我们的命名空间 `Bookstore\Core` 中。我们将使用它来包含 GET 和 POST 参数以及作为我们 `Request` 类中的两个新属性。这个映射将只包含一个属性，即数据数组，并且将有一些方法来从它中获取信息。为了构造对象，我们需要将数据数组作为参数发送给构造函数：

```php
<?php

namespace Bookstore\Core;

class FilteredMap {
    private $map;

    public function __construct(array $baseMap) {
        $this->map = $baseMap;
    }

    public function has(string $name): bool {
        return isset($this->map[$name]);
    }

    public function get(string $name) {
        return $this->map[$name] ?? null;
    }
}
```

这个类到目前为止并没有做什么。我们可以用普通的数组实现相同的功能。这个类的实用性在于我们在获取数据时添加过滤器。我们将实现三个过滤器，但你可以根据需要添加更多：

```php
public function getInt(string $name) {
    return (int) $this->get($name);
}

public function getNumber(string $name) {
    return (float) $this->get($name);
}

public function getString(string $name, bool $filter = true) {
    $value = (string) $this->get($name);
    return $filter ? addslashes($value) : $value;
}
```

上述代码中的这三个方法允许用户获取特定类型的参数。假设开发者需要从请求中获取书籍的 ID。最佳选项是使用 `getInt` 方法来确保返回的值是一个有效的整数，而不是可能破坏我们数据库的恶意代码。还要注意 `getString` 函数，我们使用了 `addSlashed` 方法。这个方法向一些可疑字符添加斜杠，例如斜杠或引号，试图用它来防止恶意代码。

现在，我们已经准备好使用我们的 `FilteredMap` 从我们的 `Request` 类中获取 GET 和 POST 参数以及 cookies。新的代码将如下所示：

```php
<?php

namespace Bookstore\Core;

class Request {
    // ...
 private $params;
 private $cookies;

    public function __construct() {
        $this->domain = $_SERVER['HTTP_HOST'];
        $this->path = explode('?', $_SERVER['REQUEST_URI'])[0];
        $this->method = $_SERVER['REQUEST_METHOD'];
 $this->params = new FilteredMap(
 array_merge($_POST, $_GET)
 );
 $this->cookies = new FilteredMap($_COOKIE);
    }

    // ...

 public function getParams(): FilteredMap {
 return $this->params;
 }

 public function getCookies(): FilteredMap {
 return $this->cookies;
 }
}
```

使用这个新功能，开发者可以通过以下代码行获取 POST 参数 `price`：

```php
$price = $request->getParams()->getNumber('price');
```

这比通常调用全局数组要安全得多：

```php
$price = $_POST['price'];
```

## 将路由映射到控制器

如果你能够回忆起你每天使用的任何 URL，你可能会看不到任何 PHP 文件作为路径的一部分，就像我们在 `http://localhost:8000/init.php` 中看到的那样。网站试图格式化它们的 URL，使它们更容易记住，而不是依赖于应该处理该请求的文件。同样，正如我们已经提到的，所有我们的请求都通过同一个文件 `index.php`，无论它们的路径如何。正因为如此，我们需要保留一个 URL 路径的映射，以及谁应该处理它们。

有时，我们的 URL 会包含作为路径一部分的参数，这与它们包含 GET 或 POST 参数的情况不同。例如，要获取显示特定书籍的页面，我们可能会在 URL 中包含书籍的 ID，例如 `/book/12` 或 `/book/3`。每个不同的书籍的 ID 都会改变，但同一个控制器应该处理所有这些请求。为了实现这一点，我们说 URL 包含一个参数，我们可以用 `/book/:id` 来表示它，其中 `id` 是标识书籍 ID 的参数。可选地，我们可以指定这个参数可以接受的价值类型，例如数字、字符串等等。

负责处理请求的控制器由一个方法的类定义。该方法接受所有由 URL 的路径定义的参数，例如书的 ID。我们根据功能分组控制器，也就是说，一个 `BookController` 类将包含与书籍请求相关的所有方法。

定义了路由的所有元素——URL-控制器关系后，我们就可以创建我们的 `routes.json` 文件了，这是一个配置文件，将保存这个映射。该文件的每个条目应包含一个路由，键是 URL，值是关于控制器的信息映射。让我们看看一个例子：

```php
{
  "books/:page": {
    "controller": "Book",
    "method": "getAllWithPage",
    "params": {
      "page": "number"
    }
  }
}
```

在前面的示例中，该路由指的是所有符合模式 `/books/:page` 的 URL，其中 `page` 可以是任何数字。因此，这个路由将匹配像 `/books/23` 或 `/books/2` 这样的 URL，但不应该匹配 `/books/one` 或 `/books`。将处理此请求的控制器设置为 `BookController` 中的 `getAllWithPage` 方法；我们将把 `Controller` 添加到所有类名中。根据我们定义的参数，方法的定义可能如下所示：

```php
public function getAllWithPage(int $page): string {
    //...
}
```

在定义路由时，我们应考虑最后一件事。对于某些端点，我们应该强制用户进行身份验证，例如当用户试图访问他们自己的销售时。我们可以以多种方式定义此规则，但我们选择将其作为路由的一部分，在控制器的信息中添加 `"login": true` 条目。考虑到这一点，让我们添加定义我们期望拥有的所有视图的其余路由：

```php
{
//...
  "books": {
    "controller": "Book",
    "method": "getAll"
  },
  "book/:id": {
    "controller": "Book",
    "method": "get",
    "params": {
      "id": "number"
    }
  },
  "books/search": {
    "controller": "Book",
    "method": "search"
  }, 
  "login": {
    "controller": "Customer",
    "method": "login"
  },
  "sales": {
    "controller": "Sales",
    "method": "getByUser" ,
    "login": true
  },
  "sales/:id": {
    "controller": "Sales",
    "method": "get",
    "login": true,
    "params": {
      "id": "number"
    }
  },
  "my-books": {
    "controller": "Book",
    "method": "getByUser",
    "login": true
  }
}
```

这些路由定义了我们需要的所有页面；我们可以以分页的方式获取所有书籍，或者通过它们的 ID 获取特定书籍，我们可以搜索书籍，列出用户的销售情况，显示特定 ID 的销售，以及列出某个用户借阅的所有书籍。然而，我们仍然缺少一些应用程序应该能够处理的端点。对于所有试图修改数据而不是请求数据的操作，即借阅一本书或购买它，我们也需要添加端点。将以下内容添加到您的 `routes.json` 文件中：

```php
{
  // ...
  "book/:id/buy": {
    "controller": "Sales",
    "method": "add",
    "login": true
    "params": {
      "id": "number"
    }
  },
  "book/:id/borrow": {
    "controller": "Book",
    "method": "borrow",
    "login": true
    "params": {
      "id": "number"
    }
  },
  "book/:id/return": {
    "controller": "Book",
    "method": "returnBook",
    "login": true
    "params": {
      "id": "number"
    }
  }
}
```

## 路由器

路由器将是我们的应用程序中最复杂的代码部分。主要目标是接收一个 `Request` 对象，决定哪个控制器应该处理它，用必要的参数调用它，并返回该控制器的响应。本节的主要目标是理解路由的重要性，而不是其详细实现，但我们将尝试描述其各个部分。将以下内容复制到您的 `src/Core/Router.php` 文件中：

```php
<?php

namespace Bookstore\Core;

use Bookstore\Controllers\ErrorController;
use Bookstore\Controllers\CustomerController;

class Router {
    private $routeMap;
    private static $regexPatters = [
        'number' => '\d+',
        'string' => '\w'
    ];

    public function __construct() {
        $json = file_get_contents(
            __DIR__ . '/../../config/routes.json'
        );
        $this->routeMap = json_decode($json, true);
    }

    public function route(Request $request): string {
        $path = $request->getPath();

        foreach ($this->routeMap as $route => $info) {
            $regexRoute = $this->getRegexRoute($route, $info);
            if (preg_match("@^/$regexRoute$@", $path)) {
                return $this->executeController(
                    $route, $path, $info, $request
                );
            }
        }

        $errorController = new ErrorController($request);
        return $errorController->notFound();
    }
}
```

该类的构造函数从 `routes.json` 文件中读取，并将内容存储为数组。其主要方法 `route` 接收一个 `Request` 对象并返回一个字符串，这是我们发送给客户端的输出。该方法迭代数组中的所有路由，尝试将每个路由与给定请求的路径进行匹配。一旦找到匹配项，它将尝试执行与该路由相关的控制器。如果没有任何路由与请求匹配，则路由器将执行 `ErrorController` 的 `notFound` 方法，然后返回一个错误页面。

### 与正则表达式匹配的 URL

在匹配路由中的 URL 时，我们需要注意动态 URL 的参数，因为它们不允许我们进行简单的字符串比较。PHP 以及其他语言有一个非常强大的工具用于执行带有动态内容的字符串比较：正则表达式。成为正则表达式专家需要时间，而且这超出了本书的范围，但我们将为您简要介绍它们。

正则表达式是一个包含一些通配符的字符串，这些通配符将匹配动态内容。其中一些最重要的如下：

+   `^`: 这用于指定匹配的部分应该是整个字符串的开始

+   `$`: 这用于指定匹配的部分应该是整个字符串的末尾

+   `\d`: 这用于匹配一个数字

+   `\w`: 这用于匹配一个单词

+   `+`: 这用于跟随一个字符或表达式，让该字符或表达式至少出现一次或多次

+   `*`: 这用于跟随一个字符或表达式，让该字符或表达式出现零次或多次

+   `.`: 这用于匹配任何单个字符

让我们看看一些例子：

+   模式 `.*` 将匹配任何内容，甚至是一个空字符串

+   模式 `.+` 将匹配任何包含至少一个字符的内容

+   模式 `^\d+$` 将匹配任何至少包含一个数字的数字

在 PHP 中，我们有不同的函数用于处理正则表达式。其中最简单的一个，也是我们将要使用的，是 `pregmatch`。该函数将其第一个参数（由两个字符分隔，通常是 `@` 或 `/`）作为模式，我们将尝试匹配的字符串作为第二个参数，可选地，一个数组，其中 PHP 存储找到的匹配项。该函数返回一个布尔值，如果找到匹配项则为 `true`，否则为 `false`。我们在 `Route` 类中如下使用它：

```php
preg_match("@^/$regexRoute$@", $path)
```

`$path` 变量包含请求的路径，例如，`/books/2`。我们使用一个由 `@` 分隔的模式进行匹配，该模式包含 `^` 和 `$` 通配符，以强制模式匹配整个字符串，并包含 `/` 和变量 `$regexRoute` 的连接。此变量的内容由以下方法提供；同时将其添加到您的 `Router` 类中：

```php
private function getRegexRoute(
    string $route,
    array $info
): string {
    if (isset($info['params'])) {
        foreach ($info['params'] as $name => $type) {
            $route = str_replace(
                ':' . $name, self::$regexPatters[$type], $route
            );
        }
    }

    return $route;
}
```

前面的方法遍历来自路由信息的参数列表。对于每个参数，函数将路由中参数的名称替换为对应参数类型的通配符字符——检查静态数组`$regexPatterns`。为了说明这个函数的用法，让我们看一些例子：

+   路由`/books`将保持不变，因为它不包含任何参数

+   路由`books/:id/borrow`将更改为`books/\d+/borrow`，因为 URL 参数`id`是一个数字

### 提取 URL 的参数

为了执行控制器，我们需要三份数据：要实例化的类的名称、要执行的方法的名称以及方法需要接收的参数。我们已经有了前两个作为路由`$info`数组的一部分，所以让我们集中精力寻找第三个。向`Router`类添加以下方法：

```php
private function extractParams(
    string $route,
    string $path
): array {
    $params = [];

    $pathParts = explode('/', $path);
    $routeParts = explode('/', $route);

    foreach ($routeParts as $key => $routePart) {
        if (strpos($routePart, ':') === 0) {
            $name = substr($routePart, 1);
            $params[$name] = $pathParts[$key+1];
        }
    }

    return $params;
}
```

这个最后的方法期望请求的路径和路由的 URL 遵循相同的模式。使用`explode`方法，我们得到两个应该匹配它们各自条目的数组。我们遍历它们，并且对于路由数组中的每个看起来像参数的条目，我们在 URL 中获取它的值。例如，如果我们有路由`/books/:id/borrow`和路径`/books/12/borrow`，这个方法的结果将是数组*['id' => 12]*。

### 执行控制器

我们通过实现负责给定路由的控制器执行方法来结束本节。我们已经有类的名称、方法的名称以及方法需要的参数，因此我们可以使用`call_user_func_array`这个原生函数，它给定一个对象、一个方法名称和方法的参数，调用对象的方法。我们必须使用它，因为参数的数量是不固定的，我们不能执行正常的调用。

但我们在创建`routes.json`文件时遗漏了一个行为。有一些路由强制用户登录，在我们的情况下，这意味着用户有一个包含用户 ID 的 cookie。给定一个强制授权的路由，我们将检查我们的请求是否包含 cookie，如果是的话，我们将通过`setCustomerId`将其设置到控制器类中。如果用户没有 cookie，我们将不会执行当前路由的控制器，而是执行`CustomerController`类的`showLogin`方法，这将渲染登录表单的模板。让我们看看在添加我们的`Router`类的最后一个方法后，一切将如何看起来：

```php
private function executeController(
    string $route,
    string $path,
    array $info,
    Request $request
): string {
    $controllerName = '\Bookstore\Controllers\\'
        . $info['controller'] . 'Controller';
    $controller = new $controllerName($request);

    if (isset($info['login']) && $info['login']) {
        if ($request->getCookies()->has('user')) {
            $customerId = $request->getCookies()->get('user');
            $controller->setCustomerId($customerId);
        } else {
            $errorController = new CustomerController($request);
            return $errorController->login();
        }
    }

    $params = $this->extractParams($route, $path);
    return call_user_func_array(
        [$controller, $info['method']], $params
    );
}
```

我们已经警告过你我们应用程序的安全性不足，因为这个项目只是具有教学目的。所以，避免复制这里实现的授权系统。

# M 代表模型

暂时想象一下，我们的书店网站非常成功，所以我们考虑建立一个移动应用来扩大我们的市场。当然，我们希望使用与我们的网站相同的数据库，因为我们需要同步两个应用中人们借阅或购买的书籍。我们不希望处于两个人购买同一本书最后一本的情况！

不仅数据库，用于获取书籍、更新它们等的查询也必须相同，否则我们最终会遇到意外的行为。当然，一个显然简单的选择是在两个代码库中复制查询，但这有一个巨大的可维护性问题。如果我们更改数据库中的一个字段怎么办？我们需要至少在两个不同的代码库中应用相同的更改。这似乎根本没有什么用。

商业逻辑在这里也扮演着重要的角色。把它想象成你需要做出的决策，这些决策会影响你的业务。在我们的案例中，那就是高级客户可以借阅 10 本书，而普通客户只能借阅 3 本，这是商业逻辑。这种逻辑也应该放在一个公共的地方，因为如果我们想改变它，我们会遇到与我们的数据库查询相同的问题。

我们希望到现在我们已经说服你，数据和商业逻辑应该从其他代码中分离出来，以便使其可重用。如果你觉得难以定义什么应该作为模型的一部分或作为控制器的一部分，不要担心；很多人都在这个区别上挣扎。由于我们的应用程序非常简单，并且没有很多商业逻辑，我们只需关注添加所有与 MySQL 查询相关的代码。

如你所想，对于一个与 MySQL 或任何其他数据库系统集成的应用程序，数据库连接是模型的一个重要元素。我们选择使用 PDO 来与 MySQL 交互，你可能还记得，实例化这个类有点痛苦。让我们创建一个单例类，它返回一个`PDO`实例，使事情变得更容易。将此代码添加到`src/Core/Db.php`中：

```php
<?php

namespace Bookstore\Core;

use PDO;

class Db {
    private static $instance;

    private static function connect(): PDO {
        $dbConfig = Config::getInstance()->get('db');
        return new PDO(
            'mysql:host=127.0.0.1;dbname=bookstore',
            $dbConfig['user'],
            $dbConfig['password']
        );
    }

    public static function getInstance(){
        if (self::$instance == null) {
            self::$instance = self::connect();
        }
        return self::$instance;
    }
}
```

```php
PDO instance. From now on, in order to get a database connection, we just need to write Db::getInstance().
```

虽然这可能对所有模型都不成立，但在我们的应用程序中，它们将始终需要访问数据库。我们可以创建一个抽象类，所有模型都从这个类扩展。这个类可以包含一个`$db`受保护的属性，它将在构造函数中设置。有了这个，我们就避免了在所有模型中重复相同的构造函数和属性定义。将以下类复制到`src/Models/AbstractModel.php`中：

```php
<?php

namespace Bookstore\Models;

use PDO;

abstract class AbstractModel {
    private $db;

    public function __construct(PDO $db) {
        $this->db = $db;
    }
}
```

最后，为了完成模型的设置，我们可以创建一个新的异常（就像我们为`NotFoundException`类所做的那样），它代表数据库中的错误。它将不包含任何代码，但我们能够区分异常是从哪里来的。我们将它保存在`src/Exceptions/DbException.php`中：

```php
<?php

namespace Bookstore\Exceptions;

use Exception;

class DbException extends Exception {
}
```

现在我们已经奠定了基础，我们可以开始编写我们的模型了。组织模型由你决定，但模仿领域对象结构是一个好主意。在这种情况下，我们将有三个模型：`CustomerModel`、`BookModel` 和 `SalesModel`。在接下来的章节中，我们将解释每个模型的内容。

## 客户模型

让我们从最简单的一个开始。由于我们的应用程序仍然非常原始，我们不会允许创建新的客户，而是使用我们手动插入数据库的客户。这意味着我们只需要对客户进行查询。让我们在 `src/Models/CustomerModel.php` 中创建一个 `CustomerModel` 类，其内容如下：

```php
<?php

namespace Bookstore\Models;

use Bookstore\Domain\Customer;
use Bookstore\Domain\Customer\CustomerFactory;
use Bookstore\Exceptions\NotFoundException;

class CustomerModel extends AbstractModel {
    public function get(int $userId): Customer {
        $query = 'SELECT * FROM customer WHERE customer_id = :user';
        $sth = $this->db->prepare($query);
        $sth->execute(['user' => $userId]);

        $row = $sth->fetch();

        if (empty($row)) {
            throw new NotFoundException();
        }

        return CustomerFactory::factory(
            $row['type'],
            $row['id'],
            $row['firstname'],
            $row['surname'],
            $row['email']
        );
    }

    public function getByEmail(string $email): Customer {
        $query = 'SELECT * FROM customer WHERE email = :user';
        $sth = $this->db->prepare($query);
        $sth->execute(['user' => $email]);

        $row = $sth->fetch();

        if (empty($row)) {
            throw new NotFoundException();
        }

        return CustomerFactory::factory(
            $row['type'],
            $row['id'],
            $row['firstname'],
            $row['surname'],
            $row['email']
        );
    }
}
```

`CustomerModel` 类继承自 `AbstractModel` 类，包含两个方法；这两个方法都返回一个 `Customer` 实例，一个在提供客户 ID 时调用，另一个在提供电子邮件时调用。因为我们已经通过 `$db` 属性拥有了数据库连接，所以我们只需要准备一个给定的查询语句，用参数执行该语句，并获取结果。如果我们期望获取一个客户，如果用户提供的 ID 或电子邮件不属于任何客户，我们需要抛出一个异常——在这种情况下，`NotFoundException` 就足够了。如果我们找到了一个客户，我们使用我们的工厂创建对象并返回它。

## 书籍模型

我们的 `BookModel` 类给我们带来了一些额外的工作。客户有一个工厂，但对于书籍来说，拥有一个工厂并不值得。我们用于从 MySQL 行创建它们的不是构造函数，而是 PDO 具有的一种获取模式，它允许我们将一行映射到对象。为了做到这一点，我们需要对 `Book` 领域对象进行一些调整：

+   属性的名称必须与数据库中的字段名称相同

+   没有必要提供构造函数或设置器，除非我们需要它们用于其他目的

+   为了与封装性相匹配，属性应该是私有的，因此我们需要为它们提供获取器

新的 `Book` 类应该如下所示：

```php
<?php

namespace Bookstore\Domain;

class Book {
    private $id;
    private $isbn;
    private $title;
    private $author;
    private $stock;
    private $price;

    public function getId(): int {
        return $this->id;
    }

    public function getIsbn(): string {
        return $this->isbn;
    }

    public function getTitle(): string {
        return $this->title;
    }

    public function getAuthor(): string {
        return $this->author;
    }

    public function getStock(): int {
        return $this->stock;
    }

    public function getCopy(): bool {
        if ($this->stock < 1) {
            return false;
        } else {
            $this->stock--;
            return true;
        }
    }

    public function addCopy() {
        $this->stock++;
    }

    public function getPrice(): float {
        return $this->price;
    }
}
```

我们保留了 `getCopy` 和 `addCopy` 方法，即使它们不是获取器，因为我们在以后还需要它们。现在，当我们使用 `fetchAll` 方法从 MySQL 获取一组行时，我们可以发送两个参数：常量 `PDO::FETCH_CLASS`，它告诉 PDO 将行映射到类，以及我们想要映射到的类的名称。让我们创建一个 `BookModel` 类，它有一个简单的 `get` 方法，该方法使用给定的 ID 从数据库中获取一本书。此方法将返回一个 `Book` 对象，或者在 ID 不存在的情况下抛出一个异常。将其保存为 `src/Models/BookModel.php`：

```php
<?php

namespace Bookstore\Models;

use Bookstore\Domain\Book;
use Bookstore\Exceptions\DbException;
use Bookstore\Exceptions\NotFoundException;
use PDO;

class BookModel extends AbstractModel {
    const CLASSNAME = '\Bookstore\Domain\Book';

    public function get(int $bookId): Book {
        $query = 'SELECT * FROM book WHERE id = :id';
        $sth = $this->db->prepare($query);
        $sth->execute(['id' => $bookId]);

 $books = $sth->fetchAll(
 PDO::FETCH_CLASS, self::CLASSNAME
 );
        if (empty($books)) {
            throw new NotFoundException();
        }

        return $books[0];
    }
}
```

使用这种获取模式有优点和缺点。一方面，在从行创建对象时，我们避免了大量的枯燥代码。通常，我们要么将行数组的所有元素发送到类的构造函数，要么使用所有属性的 setter。如果我们向 MySQL 表添加更多字段，我们只需将属性添加到我们的领域类中，而无需更改我们实例化对象的所有地方。另一方面，你被迫在表和类的属性中使用相同的字段名，这意味着高度耦合（这始终是一个坏主意）。这也导致了一些遵循约定时的冲突，因为在 MySQL 中，通常使用`book_id`，但在 PHP 中，属性是`$bookId`。

现在我们知道了这种获取模式是如何工作的，让我们添加三个其他方法，这些方法从 MySQL 获取数据。将以下代码添加到您的模型中：

```php
public function getAll(int $page, int $pageLength): array {
    $start = $pageLength * ($page - 1);

    $query = 'SELECT * FROM book LIMIT :page, :length';
    $sth = $this->db->prepare($query);
    $sth->bindParam('page', $start, PDO::PARAM_INT);
    $sth->bindParam('length', $pageLength, PDO::PARAM_INT);
    $sth->execute();

    return $sth->fetchAll(PDO::FETCH_CLASS, self::CLASSNAME);
}

public function getByUser(int $userId): array {
    $query = <<<SQL
SELECT b.*
FROM borrowed_books bb LEFT JOIN book b ON bb.book_id = b.id
WHERE bb.customer_id = :id
SQL;
    $sth = $this->db->prepare($query);
    $sth->execute(['id' => $userId]);

    return $sth->fetchAll(PDO::FETCH_CLASS, self::CLASSNAME);
}

public function search(string $title, string $author): array {
    $query = <<<SQL
SELECT * FROM book
WHERE title LIKE :title AND author LIKE :author
SQL;
    $sth = $this->db->prepare($query);
    $sth->bindValue('title', "%$title%");
    $sth->bindValue('author', "%$author%");
    $sth->execute();

    return $sth->fetchAll(PDO::FETCH_CLASS, self::CLASSNAME);
}
```

添加的方法如下：

+   `getAll`返回给定页面的所有书籍数组。记住，`LIMIT`允许你通过偏移量返回特定数量的行，这可以作为分页器使用。

+   `getByUser`返回给定客户借阅的所有书籍——我们需要为此使用连接查询。注意，我们返回`b.*`，即只返回`book`表的字段，跳过其他字段。

+   最后，有一个方法可以通过标题或作者进行搜索，或者两者都搜索。我们可以使用`LIKE`运算符和用`%`包围的模式来实现这一点。如果我们没有指定其中一个参数，我们将尝试用`%%`匹配字段，它匹配一切。

到目前为止，我们一直在添加获取数据的方法。现在让我们添加允许我们修改数据库中数据的方法。对于书籍模型，我们需要能够借阅书籍并归还它们。以下是这两个操作的代码：

```php
public function borrow(Book $book, int $userId) {
    $query = <<<SQL
INSERT INTO borrowed_books (book_id, customer_id, start)
VALUES(:book, :user, NOW())
SQL;
    $sth = $this->db->prepare($query);
    $sth->bindValue('book', $book->getId());
    $sth->bindValue('user', $userId);
    if (!$sth->execute()) {
        throw new DbException($sth->errorInfo()[2]);
    }

    $this->updateBookStock($book);
}

public function returnBook(Book $book, int $userId) {
    $query = <<<SQL
UPDATE borrowed_books SET end = NOW()
WHERE book_id = :book AND customer_id = :user AND end IS NULL 
SQL;
    $sth = $this->db->prepare($query);
    $sth->bindValue('book', $book->getId());
    $sth->bindValue('user', $userId);
    if (!$sth->execute()) {
        throw new DbException($sth->errorInfo()[2]);
    }

    $this->updateBookStock($book);
}

private function updateBookStock(Book $book) {
    $query = 'UPDATE book SET stock = :stock WHERE id = :id';
    $sth = $this->db->prepare($query);
    $sth->bindValue('id', $book->getId());
    $sth->bindValue('stock', $book->getStock());
    if (!$sth->execute()) {
        throw new DbException($sth->errorInfo()[2]);
    }
}
```

```php
borrow and returnBook methods.
```

## 销售模型

现在，我们需要向我们的应用程序添加最后一个模型：`SalesModel`。使用与书籍相同的获取模式，我们还需要调整领域类。在这种情况下，我们需要思考更多，因为我们不仅要获取数据。我们的应用程序必须能够根据需求创建新的销售，包含客户和书籍的 ID。我们当前的实施方案已经可以添加书籍，但我们需要添加一个客户 ID 的 setter。销售的 ID 将由 MySQL 中的自增 ID 提供，因此不需要为它添加 setter。最终的实现将如下所示：

```php
<?php

namespace Bookstore\Domain;

class Sale {
    private $id;
    private $customer_id;
    private $books;
    private $date;

    public function setCustomerId(int $customerId) {
        $this->customer_id = $customerId;
    }

    public function getId(): int {
        return $this->id;
    }

    public function getCustomerId(): int {
        return $this->customer_id;
    }

    public function getBooks(): array {
        return $this->books;
    }

    public function getDate(): string {
        return $this->date;
    }

    public function addBook(int $bookId, int $amount = 1) {
        if (!isset($this->books[$bookId])) {
            $this->books[$bookId] = 0;
        }
        $this->books[$bookId] += $amount;
    }

    public function setBooks(array $books) {
        $this->books = $books;
    }
}
```

`SalesModel`将是编写起来最困难的一个。这个模型的问题在于它包括操作不同的表：`sale`和`sale_book`。例如，当获取销售信息时，我们需要从`sale`表获取信息，然后从`sale_book`表获取所有书籍的信息。你可以争论是否应该有一个唯一的方法来获取与销售相关的所有必要信息，或者有两个不同的方法，一个用于获取销售信息，另一个用于获取书籍信息，让控制器决定使用哪一个。

这实际上引发了一场非常有趣的讨论。一方面，我们希望让控制器更容易操作——有一个唯一的方法来获取整个`Sale`对象。这是有意义的，因为控制器不需要了解`Sale`对象的内部实现，这降低了耦合。另一方面，强迫模型总是获取整个对象，即使我们只需要`sale`表中的信息，也是一个坏主意。想象一下，如果销售包含大量的书籍，从 MySQL 中获取它们将无必要地降低性能。

你应该思考你的控制器如何管理销售。如果你总是需要整个对象，你可以有一个方法而不必担心性能。如果你有时需要获取整个对象，也许你可以添加两个方法。对于我们的应用程序，我们将有一个方法来处理所有这些，因为我们总是需要它。

### 注意

**延迟加载**

就像任何其他设计挑战一样，其他开发者已经对这个问题进行了很多思考。他们提出了一种名为**延迟加载**的设计模式。这个模式基本上让控制器认为只有一个方法可以获取整个域对象，但实际上我们只会从数据库中获取所需的数据。

模型获取对象最常用的信息，并将需要额外数据库查询的其他属性留空。一旦控制器使用一个空属性的 getter，模型会自动从数据库中获取那些数据。我们得到了两全其美的效果：控制器有简单的操作，但我们不会在查询未使用的数据上浪费更多时间。

将以下内容添加到你的`src/Models/SaleModel.php`文件中：

```php
<?php
namespace Bookstore\Models;

use Bookstore\Domain\Sale;
use Bookstore\Exceptions\DbException;
use PDO;

class SaleModel extends AbstractModel {
    const CLASSNAME = '\Bookstore\Domain\Sale';

    public function getByUser(int $userId): array {
        $query = 'SELECT * FROM sale WHERE s.customer_id = :user';
        $sth = $this->db->prepare($query);
        $sth->execute(['user' => $userId]);

        return $sth->fetchAll(PDO::FETCH_CLASS, self::CLASSNAME);
    }

    public function get(int $saleId): Sale {
        $query = 'SELECT * FROM sale WHERE id = :id';
        $sth = $this->db->prepare($query);
        $sth->execute(['id' => $saleId]);
        $sales = $sth->fetchAll(PDO::FETCH_CLASS, self::CLASSNAME);

        if (empty($sales)) {
            throw new NotFoundException('Sale not found.');
        }
        $sale = array_pop($sales);

        $query = <<<SQL
SELECT b.id, b.title, b.author, b.price, sb.amount as stock, b.isbn
FROM sale s
LEFT JOIN sale_book sb ON s.id = sb.sale_id
LEFT JOIN book b ON sb.book_id = b.id
WHERE s.id = :id
SQL;
        $sth = $this->db->prepare($query);
        $sth->execute(['id' => $saleId]);
        $books = $sth->fetchAll(
            PDO::FETCH_CLASS, BookModel::CLASSNAME
        );

        $sale->setBooks($books);
        return $sale;
    }
}
```

在这个模型中，另一个棘手的方法是处理在数据库中创建销售的方法。这个方法必须在`sale`表中创建一个销售记录，然后将该销售的所有书籍添加到`sale_book`表中。如果我们添加书籍时出现问题，会发生什么？我们会在数据库中留下一个损坏的销售记录。为了避免这种情况，我们需要使用事务，从模型或控制器方法的开始处开始，在出现错误时回滚，或者在方法结束时提交。

在相同的方法中，我们还需要注意销售项的 ID。在创建`sale`对象时，我们没有设置销售项的 ID，因为我们依赖于数据库中的自增字段。但是，当将书籍插入到`sale_book`中时，我们需要销售项的 ID。为此，我们需要使用`lastInsertId`方法请求 PDO 的最后一个插入 ID。让我们将`create`方法添加到您的`SaleModel`中：

```php
public function create(Sale $sale) {
 $this->db->beginTransaction();

    $query = <<<SQL
INSERT INTO sale(customer_id, date)
VALUES(:id, NOW())
SQL;
    $sth = $this->db->prepare($query);
    if (!$sth->execute(['id' => $sale->getCustomerId()])) {
 $this->db->rollBack();
        throw new DbException($sth->errorInfo()[2]);
    }

 $saleId = $this->db->lastInsertId();
    $query = <<<SQL
INSERT INTO sale_book(sale_id, book_id, amount)
VALUES(:sale, :book, :amount)
SQL;
    $sth = $this->db->prepare($query);
    $sth->bindValue('sale', $saleId);
    foreach ($sale->getBooks() as $bookId => $amount) {
        $sth->bindValue('book', $bookId);
        $sth->bindValue('amount', $amount);
        if (!$sth->execute()) {
 $this->db->rollBack();
            throw new DbException($sth->errorInfo()[2]);
        }
    }

 $this->db->commit();
}
```

从这个方法中需要注意的最后一点是，我们准备一个语句，将其绑定到一个值（销售 ID），然后绑定并执行与数组中书籍数量相同的相同语句。一旦你有一个语句，你可以绑定你想要的任何次数的值。同样，你可以执行你想要的任何次数的相同语句，而值保持不变。

# V 代表视图

视图层负责处理视图。在这个层中，你可以找到所有渲染用户获取的 HTML 的模板。尽管视图与应用程序其他部分的分离很容易看出，但这并不意味着视图是一个容易的部分。实际上，你将不得不学习一项新技术才能正确编写视图。让我们深入了解细节。

## Twig 简介

在我们第一次尝试编写视图时，我们将 PHP 和 HTML 代码混合在一起。我们已经知道逻辑不应该与 HTML 混合在同一地方，但这并不是故事的结尾。在渲染 HTML 时，我们同样需要一些逻辑。例如，如果我们想打印一本书的列表，我们需要为每本书重复一定的 HTML 块。由于我们事先不知道要打印多少本书，最佳选择是使用`foreach`循环。

许多人选择的一个选项是尽量减少在视图中包含的逻辑量。你可以设定一些规则，例如*我们只应包含条件和循环*，这是渲染基本视图所需的合理逻辑量。问题是无法强制执行这类规则，其他开发者可以轻易地开始在其中添加复杂的逻辑。虽然有些人对此表示可以接受，假设没有人会这样做，但其他人更喜欢实施更严格的系统。这就是模板引擎的起源。

你可以将模板引擎视为另一种需要学习的新语言。你为什么要这样做呢？因为这种新的“语言”比 PHP 更有限。这些语言通常允许你执行条件和简单的循环，仅此而已。开发者无法将 PHP 添加到该文件中，因为模板引擎不会将其视为 PHP 代码。相反，它只会将代码打印到输出——响应体——就像它是纯文本一样。此外，由于它专门面向编写模板，当与 HTML 混合时，语法通常更容易阅读。几乎一切都是优点。

使用模板引擎的不便之处在于，它需要一些时间将新语言翻译成 PHP，然后再翻译成 HTML。这可能会非常耗时，因此选择一个好的模板引擎非常重要。大多数模板引擎还允许你缓存模板，从而提高性能。我们的选择是一个相当轻量级且广泛使用的：**Twig**。因为我们已经在 Composer 文件中添加了依赖项，所以我们可以直接使用它。

设置 Twig 相当简单。在 PHP 方面，你只需要指定模板的位置。一个常见的约定是使用`views`目录。创建该目录，并在你的`index.php`中添加以下两行：

```php
$loader = new Twig_Loader_Filesystem(__DIR__ . '/views');
$twig = new Twig_Environment($loader);
```

## 书籍视图

在这些部分，当我们处理模板时，看到你工作的结果会很好。我们还没有实现任何控制器，所以我们将强制`index.php`渲染一个特定的模板，无论请求如何。我们可以开始渲染单个书籍的视图。为此，让我们在创建你的`twig`对象之后，在`index.php`的末尾添加以下代码：

```php
$bookModel = new BookModel(Db::getInstance());
$book = $bookModel->get(1);

$params = ['book' => $book];
echo $twig->loadTemplate('book.twig')->render($params);
```

在前面的代码中，我们向`BookModel`请求 ID 为 1 的书籍，获取`book`对象，并创建一个数组，其中`book`键的值为`book`对象。之后，我们告诉 Twig 加载模板`book.twig`，并通过发送数组来渲染它。这会将模板和`$book`对象注入其中，这样你就可以在模板内部使用它了。

现在我们来创建我们的第一个模板。将以下代码写入`view/book.twig`。按照惯例，所有 Twig 模板都应该有`.twig`扩展名：

```php
<h2>{{ book.title }}</h2>
<h3>{{ book.author }}</h3>

<hr>

<p>
    <strong>ISBN</strong> {{ book.isbn }}
</p>
<p>
    <strong>Stock</strong> {{ book.stock }}
</p>
<p>
    <strong>Price</strong> {{ book.price|number_format(2) }} €
</p>

<hr>

<h3>Actions</h3>

<form method="post" action="/book/{{ book.id }}/borrow">
    <input type="submit" value="Borrow">
</form>

<form method="post" action="/book/{{ book.id }}/buy">
    <input type="submit" value="Buy">
</form>
```

由于这是你的第一个 Twig 模板，让我们一步一步来。你可以看到大部分内容是 HTML：一些标题，几个段落，以及两个带有两个按钮的表单。你可以识别出 Twig 部分，因为它被`{{ }}`包围。在 Twig 中，所有那些在花括号之间的内容都将被打印出来。我们找到的第一个包含`book.title`。你还记得我们在渲染模板时注入了`book`对象吗？我们在这里可以访问它，只是不是用常规的 PHP 语法。要访问一个对象的属性，使用`.`而不是`->`。所以，这个`book.title`将返回`book`对象的`title`属性的值，而`{{ }}`将使 Twig 打印它出来。模板的其余部分也是如此。

有一个功能不仅限于访问对象的属性。`book.price|number_format(2)`获取书籍的价格，并将其作为参数（使用管道符号）发送到已经获得`2`作为另一个参数的`number_format`函数。这段代码基本上将价格格式化为两位数字。在 Twig 中，你也有一些函数，但它们主要被简化为格式化输出，这是可接受的逻辑量。

你现在相信使用模板引擎为你的视图提供多么清晰的解决方案了吗？你可以在浏览器中尝试它：访问任何路径，你的 Web 服务器应该执行`index.php`文件，强制渲染模板`book.twig`。

## 布局和区块

当你设计你的 Web 应用程序时，通常你会在大多数视图中共享一个共同的布局。在我们的例子中，我们希望视图的顶部始终有一个菜单，允许我们访问网站的各个部分，甚至可以从用户所在的位置搜索书籍。与模型一样，我们想要避免代码重复，因为如果我们把布局复制粘贴到每个地方，更新它将是一场噩梦。相反，Twig 提供了定义布局的能力。

在 Twig 中，**布局**只是一个模板文件。其内容是我们想要在所有视图中显示的公共 HTML 代码（在我们的案例中，是菜单和搜索栏），并包含一些带标签的空隙（Twig 世界中的区块），你将能够注入每个视图的特定 HTML。你可以使用`{% block %}`标签定义其中一个区块。让我们看看我们的`views/layout.twig`文件会是什么样子：

```php
<html>
<head>
 <title>{% block title %}{% endblock %}</title>
</head>
<body>
    <div style="border: solid 1px">
        <a href="/books">Books</a>
        <a href="/sales">My Sales</a>
        <a href="/my-books">My Books</a>
        <hr>
        <form action="/books/search" method="get">
            <label>Title</label>
            <input type="text" name="title">
            <label>Author</label>
            <input type="text" name="author">
            <input type="submit" value="Search">
        </form>
    </div>
 {% block content %}{% endblock %}
</body>
</html>
```

如前述代码所示，区块有一个名称，这样使用布局的模板就可以引用它们。在我们的布局中，我们定义了两个区块：一个用于视图的标题，另一个用于内容本身。当模板使用布局时，我们只需要为布局中定义的每个区块编写 HTML 代码，Twig 就会完成剩余的工作。此外，为了让 Twig 知道我们的模板想要使用布局，我们使用带有布局文件名的`{% extends %}`标签。让我们更新`views/book.twig`以使用我们新的布局：

```php
{% extends 'layout.twig' %}

{% block title %}
 {{ book.title }}
{% endblock %}

{% block content %}
<h2>{{ book.title }}</h2>
//...
</form>
{% endblock %}

```

在文件顶部，我们添加我们需要使用的布局。然后，我们用一个带有参考名称的区块标签打开，并在其中写入我们想要使用的 HTML。你可以在区块中使用任何有效的 HTML，无论是 Twig 代码还是纯 HTML。在我们的模板中，我们使用书的标题作为`title`区块，它引用了视图的标题，并将所有之前的 HTML 放入`content`区块。请注意，现在文件中的所有内容都在一个区块内。现在在你的浏览器中尝试一下，看看变化。

## 分页书籍列表

让我们再添加另一个视图，这次是为书籍分页列表。为了看到你工作的结果，更新`index.php`的内容，用以下代码替换上一节的代码：

```php
$bookModel = new BookModel(Db::getInstance());
$books = $bookModel->getAll(1, 3);

$params = ['books' => $books, 'currentPage' => 2];
echo $twig->loadTemplate('books.twig')->render($params);
```

```php
books.twig template, sending an array of books from page number 1, and showing 3 books per page. This array, though, might not always return 3 books, maybe because there are only 2 books in the database. We should then use a loop to iterate the list instead of assuming the size of the array. In Twig, you can emulate a foreach loop using {% for <element> in <array> %} in order to iterate an array. Let's use it for your views/books.twig:
```

```php
{% extends 'layout.twig' %}

{% block title %}
    Books
{% endblock %}

{% block content %}
<table>
    <thead>
        <th>Title</th>
        <th>Author</th>
        <th></th>
    </thead>
{% for book in books %}
    <tr>
        <td>{{ book.title }}</td>
        <td>{{ book.author }}</td>
        <td><a href="/book/{{ book.id }}">View</a></td>
    </tr>
{% endfor %}
</table>
{% endblock %}
```

我们还可以在 Twig 模板中使用条件语句，它们的工作方式与 PHP 中的条件语句相同。语法是`{% if <boolean expression> %}`。让我们使用它来决定是否在我们的页面上显示上一页和/或下一页链接。在内容区块的末尾添加以下代码：

```php
{% if currentPage != 1 %}
    <a href="/books/{{ currentPage - 1 }}">Previous</a>
{% endif %}
{% if not lastPage %}
    <a href="/books/{{ currentPage + 1 }}">Next</a>
{% endif %}
```

从这个模板中需要注意的最后一件事是，我们在使用`{{ }}`打印内容时，不仅限于使用变量。我们可以添加任何有效的 Twig 表达式，它返回一个值，就像我们使用`{{ currentPage + 1 }}`一样。

## 销售视图

我们已经向你展示了使用模板所需的一切，现在我们只需完成添加所有模板。列表中的下一个模板是显示特定用户销售列表的模板。使用以下技巧更新你的 `index.php` 文件：

```php
$saleModel = new SaleModel(Db::getInstance());
$sales = $saleModel->getByUser(1);

$params = ['sales' => $sales];
echo $twig->loadTemplate('sales.twig')->render($params);
```

这个视图的模板将与列出书籍的模板非常相似：一个填充了数组内容的表格。以下是 `views/sales.twig` 的内容：

```php
{% extends 'layout.twig' %}

{% block title %}
    My sales
{% endblock %}

{% block content %}
<table>
    <thead>
        <th>Id</th>
        <th>Date</th>
    </thead>
{% for sale in sales %}
    <tr>
        <td>{{ sale.id}}</td>
        <td>{{ sale.date }}</td>
        <td><a href="/sales/{{ sale.id }}">View</a></td>
    </tr>
{% endfor %}
</table>
{% endblock %}
```

与销售相关的另一种观点是我们希望展示特定内容的全部。这次销售，同样，将与书籍列表相似，因为我们将会列出与这次销售相关的书籍。强制渲染此模板的技巧如下：

```php
$saleModel = new SaleModel(Db::getInstance());
$sale = $saleModel->get(1);

$params = ['sale' => $sale];
echo $twig->loadTemplate('sale.twig')->render($params);
```

并且，Twig 模板应该放置在 `views/sale.twig`：

```php
{% extends 'layout.twig' %}

{% block title %}
    Sale {{ sale.id }}
{% endblock %}

{% block content %}
<table>
    <thead>
        <th>Title</th>
        <th>Author</th>
        <th>Amount</th>
        <th>Price</th>
        <th></th>
    </thead>
    {% for book in sale.books %}
        <tr>
            <td>{{ book.title }}</td>
            <td>{{ book.author }}</td>
            <td>{{ book.stock }}</td>
            <td>{{ (book.price * book.stock)|number_format(2) }} €</td>
            <td><a href="/book/{{ book.id }}">View</a></td>
        </tr>
    {% endfor %}
</table>
{% endblock %}
```

## 错误模板

我们应该添加一个非常简单的模板，当我们的应用中出现错误时，将显示给用户，而不是显示 PHP 错误消息。这个模板只期望 `errorMessage` 变量，它可能看起来像以下这样。将其保存为 `views/error.twig`：

```php
{% extends 'layout.twig' %}

{% block title %}
    Error
{% endblock %}

{% block content %}
    <h2>Error: {{ errorMessage }}</h2>
{% endblock %}
```

注意，即使是错误页面也扩展自布局，因为我们希望用户在发生这种情况时能够做些其他事情。

## 登录模板

我们最后的模板将允许用户登录。这个模板与其他模板略有不同，因为它将在两种不同的场景中使用。在第一种情况下，用户首次访问登录视图，因此我们需要显示表单。在第二种情况下，用户已经尝试登录，但在登录过程中出现了错误，即找不到电子邮件地址。在这种情况下，我们将向模板添加一个额外的变量 `errorMessage`，并且我们将添加一个条件来显示其内容，仅当这个变量被定义时。你可以使用 `is defined` 操作符来检查。将以下模板作为 `views/login.twig` 添加：

```php
{% extends 'layout.twig' %}

{% block title %}
    Login
{% endblock %}

{% block content %}
 {% if errorMessage is defined %}
        <strong>{{ errorMessage }}</strong>
    {% endif %}
    <form action="/login" method="post">
        <label>Email</label>
        <input type="text" name="email">
        <input type="submit">
    </form>
{% endblock %}
```

# C 代表控制器

现在，轮到乐团指挥了。控制器代表我们的应用中这样一个层次：给定一个请求，与模型通信并构建视图。它们就像一个团队的经理：根据情况决定使用哪些资源。

正如我们在解释模型时所说的，有时很难决定某些逻辑是否应该放入控制器或模型中。最终，MVC 只是一个模式，就像一个指导你的食谱，而不是一个需要你一步一步遵循的精确算法。有些情况下答案并不直接，所以这取决于你；在这些情况下，只需尽量保持一致性。以下是一些可能难以本地化的常见场景：

+   请求指向我们不支持的路径。这种情况在我们的应用中已经得到处理，并且应该由路由器来处理，而不是控制器。

+   请求尝试访问一个不存在的元素，例如，数据库中不存在的书籍 ID。在这种情况下，控制器应该询问模型书籍是否存在，并根据响应渲染包含书籍内容的模板，或者渲染另一个包含“未找到”信息的模板。

+   用户尝试执行一个动作，例如购买一本书，但请求中来的参数无效。这是一个棘手的问题。一个选项是获取请求中的所有参数而不进行检查，直接将它们发送到模型，并将清理信息的工作留给模型。另一个选项是控制器检查提供的参数是否合理，然后将它们交给模型。还有其他解决方案，例如构建一个检查参数是否有效的类，这个类可以在不同的控制器中重用。在这种情况下，它将取决于参数的数量和清理中涉及的逻辑。对于接收大量数据的请求，第三个选项看起来是其中最好的，因为我们将在不同的端点重用代码，并且我们没有编写过长的控制器。但在用户发送一个或两个参数的请求中，在控制器中清理它们可能就足够了。

现在我们已经奠定了基础，让我们准备我们的应用程序使用控制器。首先要做的是更新我们的`index.php`文件，它一直迫使应用程序始终渲染相同的模板。相反，我们应该将这个任务交给路由器，它将返回一个字符串作为响应，我们可以直接使用`echo`打印。使用以下内容更新您的`index.php`文件：

```php
<?php

use Bookstore\Core\Router;
use Bookstore\Core\Request;

require_once __DIR__ . '/vendor/autoload.php';

$router = new Router();
$response = $router->route(new Request());
echo $response;

```

如您可能记得，路由器实例化一个控制器类，将请求对象发送到构造函数。但是控制器还有其他依赖项，例如模板引擎、数据库连接或配置读取器。尽管这不是最佳解决方案（我们将在下一节介绍依赖注入时对其进行改进），我们可以创建一个`AbstractController`，它将是所有控制器的父类，并将设置这些依赖项。将以下内容复制为`src/Controllers/AbstractController.php`：

```php
<?php

namespace Bookstore\Controllers;

use Bookstore\Core\Config;
use Bookstore\Core\Db;
use Bookstore\Core\Request;
use Monolog\Logger;
use Twig_Environment;
use Twig_Loader_Filesystem;
use Monolog\Handler\StreamHandler;

abstract class AbstractController {
    protected $request;
    protected $db;
    protected $config;
    protected $view;
    protected $log;

    public function __construct(Request $request) {
        $this->request = $request;
        $this->db = Db::getInstance();
        $this->config = Config::getInstance();

        $loader = new Twig_Loader_Filesystem(
            __DIR__ . '/../../views'
        );
        $this->view = new Twig_Environment($loader);

        $this->log = new Logger('bookstore');
        $logFile = $this->config->get('log');
        $this->log->pushHandler(
            new StreamHandler($logFile, Logger::DEBUG)
        );
    }

    public function setCustomerId(int $customerId) {
        $this->customerId = $customerId;
    }
}
```

在实例化控制器时，我们将设置一些在处理请求时有用的属性。我们已经知道如何实例化数据库连接、配置读取器和模板引擎。第四个属性`$log`将允许开发者在必要时将日志写入指定的文件。我们将使用 Monolog 库来完成这项工作，但还有许多其他选项。请注意，为了实例化记录器，我们从配置中获取日志的值，这应该是日志文件的路径。惯例是使用`/var/log/`目录，因此创建`/var/log/bookstore.log`文件，并将`"log": "/var/log/bookstore.log"`添加到您的配置文件中。

另一件对某些控制器有用的信息（但不是所有控制器）是关于执行动作的用户的信息。由于这仅适用于某些路由，我们不应该在构建控制器时设置它。相反，我们有一个 setter 用于路由器，当可用时设置客户 ID；实际上，路由器已经这样做了。

最后，一个方便的辅助方法是我们可以使用的一个，它使用参数渲染给定的模板，因为所有控制器最终都会渲染一个模板或另一个。让我们向`AbstractController`类添加以下受保护的方法：

```php
protected function render(string $template, array $params): string {
    return $this->view->loadTemplate($template)->render($params);
}
```

## 错误控制器

让我们先创建最简单的控制器：`ErrorController`。这个控制器并没有做很多事情；它只是渲染`error.twig`模板，发送“页面未找到！”的消息。你可能还记得，当路由器无法匹配到其他定义的路由时，它会使用这个控制器。将以下类保存到`src/Controllers/ErrorController.php`中：

```php
<?php

namespace Bookstore\Controllers;

class ErrorController extends AbstractController {
    public function notFound(): string {
        $properties = ['errorMessage' => 'Page not found!'];
        return $this->render('error.twig', $properties);
    }
}
```

## 登录控制器

我们必须添加的第二个控制器是管理客户登录的控制器。如果我们考虑用户想要进行身份验证时的流程，我们会遇到以下场景：

+   用户想要获取登录表单，以便提交必要的信息并登录。

+   用户试图提交表单，但我们无法获取电子邮件地址。我们应该再次渲染表单，并让他们知道问题所在。

+   用户提交了带有电子邮件的表单，但不是一个有效的电子邮件。在这种情况下，我们应该再次显示登录表单，并带有解释情况的错误信息。

+   用户提交了一个有效的电子邮件，我们设置了 cookie，并显示了书籍列表，以便用户可以开始搜索。这完全是任意的；你也可以选择将他们发送到他们借阅的书籍页面、他们的销售页面等。这里重要的是要注意，我们将请求重定向到另一个控制器。

有多达四种可能的路径。我们将使用`request`对象来决定在每种情况下使用哪一条路径，并返回相应的响应。因此，让我们在`src/Controllers/CustomerController.php`中创建`CustomerController`类，并添加`login`方法，如下所示：

```php
<?php

namespace Bookstore\Controllers;

use Bookstore\Exceptions\NotFoundException;
use Bookstore\Models\CustomerModel;

class CustomerController extends AbstractController {
    public function login(string $email): string {
        if (!$this->request->isPost()) {
 return $this->render('login.twig', []);
        }

        $params = $this->request->getParams();

        if (!$params->has('email')) {
            $params = ['errorMessage' => 'No info provided.'];
 return $this->render('login.twig', $params);
        }

        $email = $params->getString('email');
        $customerModel = new CustomerModel($this->db);

        try {
            $customer = $customerModel->getByEmail($email);
        } catch (NotFoundException $e) {
            $this->log->warn('Customer email not found: ' . $email);
            $params = ['errorMessage' => 'Email not found.'];
 return $this->render('login.twig', $params);
        }

        setcookie('user', $customer->getId());

        $newController = new BookController($this->request);
 return $newController->getAll();
    }
}
```

如你所见，有四种不同的返回值对应于四种不同的情况。控制器本身并没有做什么，而是协调其他组件，并做出决策。首先，我们检查请求是否为 POST，如果不是，我们将假设用户想要获取表单。如果是，我们将检查参数中的电子邮件，如果电子邮件不存在，则返回错误。如果存在，我们将尝试使用我们的模型找到具有该电子邮件的客户。如果得到一个异常，表示没有这样的客户，我们将渲染带有“未找到”错误信息的表单。如果登录成功，我们将设置包含客户 ID 的 cookie，并执行`BookController`的`getAll`方法（尚未编写），返回书籍列表。

到目前为止，你应该能够使用浏览器端到端测试你应用程序的登录功能。尝试访问 `http://localhost:8000/login` 来查看表单，添加随机的电子邮件以获取错误信息，并添加一个有效的电子邮件（检查 MySQL 中的`customer`表）以成功登录。之后，你应该能看到包含客户 ID 的 cookie。

## 书籍控制器

`BookController`类将是我们的控制器中最大的一个，因为大多数应用程序都依赖于它。让我们先添加最简单的函数，即只从数据库检索信息的函数。将其保存为`src/Controllers/BookController.php`：

```php
<?php

namespace Bookstore\Controllers;

use Bookstore\Models\BookModel;

class BookController extends AbstractController {
    const PAGE_LENGTH = 10;

    public function getAllWithPage($page): string {
        $page = (int)$page;
        $bookModel = new BookModel($this->db);

        $books = $bookModel->getAll($page, self::PAGE_LENGTH);

        $properties = [
            'books' => $books,
            'currentPage' => $page,
            'lastPage' => count($books) < self::PAGE_LENGTH
        ];
        return $this->render('books.twig', $properties);
    }

    public function getAll(): string {
        return $this->getAllWithPage(1);
    }

    public function get(int $bookId): string {
        $bookModel = new BookModel($this->db);

        try {
            $book = $bookModel->get($bookId);
        } catch (\Exception $e) {
            $this->log->error(
                'Error getting book: ' . $e->getMessage()
            );
            $properties = ['errorMessage' => 'Book not found!'];
            return $this->render('error.twig', $properties);
        }

        $properties = ['book' => $book];
        return $this->render('book.twig', $properties);
    }

    public function getByUser(): string {
        $bookModel = new BookModel($this->db);

        $books = $bookModel->getByUser($this->customerId);

        $properties = [
            'books' => $books,
            'currentPage' => 1,
            'lastPage' => true
        ];
        return $this->render('books.twig', $properties);
    }
}
```

到目前为止，这段前置代码并没有什么特别之处。`getAllWithPage` 和 `getAll` 方法执行相同的功能，一个通过用户提供的页面编号作为 URL 参数，另一个将页面编号设置为 1——默认情况。它们请求模型以获取要显示并传递给视图的书籍列表。当前页面的信息——以及我们是否在最后一页——也会发送到模板中，以便添加“上一页”和“下一页”的链接。

`get` 方法将获取客户感兴趣的书籍的 ID。它将尝试使用模型获取它。如果模型抛出异常，我们将渲染带有“书籍未找到”信息的错误模板。相反，如果书籍 ID 有效，我们将按预期渲染书籍模板。

`getByUser`方法将返回认证客户借阅的所有书籍。我们将使用从路由器设置的`customerId`属性。这里没有进行合理性检查，因为我们不是试图获取特定的书籍，而是一个列表，如果用户尚未借阅任何书籍，这个列表可能为空——但这不是问题。

另一个获取器控制器是搜索书籍标题和/或作者的方法。当用户在布局模板中提交表单时，将触发此方法。表单发送 `title` 和 `author` 字段，因此控制器将请求这两个字段。模型已经准备好使用空参数，因此我们在这里不会进行任何额外的检查。将方法添加到 `BookController` 类中：

```php
public function search(): string {
    $title = $this->request->getParams()->getString('title');
    $author = $this->request->getParams()->getString('author');

    $bookModel = new BookModel($this->db);
    $books = $bookModel->search($title, $author);

    $properties = [
        'books' => $books,
        'currentPage' => 1,
        'lastPage' => true
    ];
    return $this->render('books.twig', $properties);
}
```

您的应用程序无法执行任何操作，但至少您最终可以浏览书籍列表，并点击其中任何一本书查看详细信息。我们终于有所收获了！

## 借阅书籍

借阅和归还书籍可能是涉及最多逻辑的操作，与购买书籍一起，将由不同的控制器处理。这是一个开始记录用户操作的好地方，因为这对于以后的调试非常有用。让我们先看看代码，然后再简要讨论一下。向您的 `BookController` 类添加以下两个方法：

```php
public function borrow(int $bookId): string {
    $bookModel = new BookModel($this->db);

    try {
        $book = $bookModel->get($bookId);
    } catch (NotFoundException $e) {
 $this->log->warn('Book not found: ' . $bookId);
        $params = ['errorMessage' => 'Book not found.'];
        return $this->render('error.twig', $params);
    }

    if (!$book->getCopy()) {
        $params = [
            'errorMessage' => 'There are no copies left.'
       ];
        return $this->render('error.twig', $params);
    }

    try {
        $bookModel->borrow($book, $this->customerId);
    } catch (DbException $e) {
 $this->log->error(
 'Error borrowing book: ' . $e->getMessage()
        );
        $params = ['errorMessage' => 'Error borrowing book.'];
        return $this->render('error.twig', $params);
    }

    return $this->getByUser();
}

public function returnBook(int $bookId): string {
    $bookModel = new BookModel($this->db);

    try {
        $book = $bookModel->get($bookId);
    } catch (NotFoundException $e) {
 $this->log->warn('Book not found: ' . $bookId);
        $params = ['errorMessage' => 'Book not found.'];
        return $this->render('error.twig', $params);
    }

    $book->addCopy();

    try {
        $bookModel->returnBook($book, $this->customerId);
    } catch (DbException $e) {
 $this->log->error(
 'Error returning book: ' . $e->getMessage()
        );
        $params = ['errorMessage' => 'Error returning book.'];
        return $this->render('error.twig', $params);
    }

    return $this->getByUser();
}
```

正如我们之前提到的，这里的新功能之一是我们正在记录用户操作，例如尝试借阅或归还无效的书籍。Monolog 允许您使用不同的优先级级别写入日志：错误、警告和通知。您可以使用 `error`、`warn` 或 `notice` 等方法来引用它们。当发生意外但非关键的情况时，我们会使用警告，例如尝试借阅不存在的书籍。当出现我们无法恢复的未知问题时，我们会使用错误，例如数据库错误。

这两个方法的操作模式如下：我们从数据库中根据给定的书籍 ID 获取 `book` 对象。通常情况下，如果没有这样的书籍，我们会返回一个错误页面。一旦我们有了 `book` 领域对象，我们就使用 `addCopy` 和 `getCopy` 等辅助器来更新书籍库存，并将其连同客户 ID 一起发送到模型，以便在数据库中存储信息。在借阅书籍时，我们也会进行一个合理性检查，以防没有更多的书籍可用。在这两种情况下，我们都将用户已借阅的书籍列表作为控制器的响应返回。

## 销售控制器

我们来到了最后一个控制器：`SalesController`。由于使用了不同的模型，它最终将执行与借阅书籍相关的方法几乎相同的功能。但我们需要在控制器中创建 `sale` 领域对象，而不是从模型中获取它。让我们添加以下代码，其中包含一个购买书籍的方法 `add` 和两个获取器：一个获取特定用户的全部销售记录，另一个获取特定销售的信息，即 `getByUser` 和 `get`。按照惯例，文件将是 `src/Controllers/SalesController.php`：

```php
<?php

namespace Bookstore\Controllers;

use Bookstore\Domain\Sale;
use Bookstore\Models\SaleModel;

class SalesController extends AbstractController {
    public function add($id): string {
        $bookId = (int)$id;
        $salesModel = new SaleModel($this->db);

        $sale = new Sale();
        $sale->setCustomerId($this->customerId);
        $sale->addBook($bookId);

        try {
            $salesModel->create($sale);
        } catch (\Exception $e) {
            $properties = [
                'errorMessage' => 'Error buying the book.'
           ];
            $this->log->error(
                'Error buying book: ' . $e->getMessage()
            );
            return $this->render('error.twig', $properties);
        }

        return $this->getByUser();
    }

    public function getByUser(): string {
        $salesModel = new SaleModel($this->db);

        $sales = $salesModel->getByUser($this->customerId);

        $properties = ['sales' => $sales];
        return $this->render('sales.twig', $properties);
    }

    public function get($saleId): string {
        $salesModel = new SaleModel($this->db);

        $sale = $salesModel->get($saleId);

        $properties = ['sale' => $sale];
        return $this->render('sale.twig', $properties);
    }
}
```

# 依赖注入

在本章结束时，我们将介绍与 MVC 模式以及一般 OOP 相关的最有趣和最具争议的话题之一：**依赖注入**。我们将向你展示为什么它如此重要，以及如何实施一个适合我们特定应用的解决方案，尽管有相当多的不同实现可以满足不同的需求。

## 为什么需要依赖注入？

我们仍然需要介绍如何对代码进行单元测试，因此你还没有亲自体验过。但潜在问题来源的一个迹象是在你的代码中使用`new`语句创建不属于你的代码库的类的实例——也称为依赖项。使用`new`创建域对象，如`Book`或`Sale`是可以的。用它来实例化模型也是可以接受的。但手动实例化其他东西，比如模板引擎、数据库连接或记录器，是你应该避免的。有不同理由支持这个观点：

+   如果你想在两个不同的地方使用控制器，并且每个地方都需要不同的数据库连接或日志文件，那么在控制器内部实例化这些依赖项将不允许我们这样做。同一个控制器将始终使用相同的依赖项。

+   在控制器内部实例化依赖项意味着控制器完全了解每个依赖项的具体实现，也就是说，控制器知道我们正在使用 PDO 和 MySQL 驱动程序，以及连接凭证的位置。这意味着你的应用程序耦合度很高——所以，坏消息。

+   如果你在每个地方都显式实例化依赖项，那么用实现相同接口的另一个依赖项替换一个依赖项并不容易，因为你将不得不搜索所有这些地方，并手动更改实例化。

由于所有这些原因，以及更多原因，总是提供控制器等类需要的依赖项，而不是让它自己创建，这是一个大家都能接受的观点。问题在于实施解决方案。有几种不同的选择：

+   我们有一个期望（通过参数）所有控制器或任何其他类需要的依赖项的构造函数。构造函数将每个参数分配给类的属性。

+   我们有一个空构造函数，而不是添加与类的依赖项一样多的 setter 方法。

+   这是一种混合方式，我们通过构造函数设置主要依赖项，其余依赖项则通过 setter 设置。

+   将包含所有依赖项的对象作为唯一参数传递给构造函数，控制器从该容器中获取它需要的依赖项。

每个解决方案都有其优缺点。如果我们有一个具有许多依赖的类，通过构造函数注入所有这些依赖会使它难以理解，所以最好使用 setter 来注入它们，即使一个具有许多依赖的类看起来像是一个糟糕的设计。如果我们只有一个或两个依赖，使用构造函数可能是可以接受的，而且我们会写更少的代码。对于具有几个依赖但并非所有依赖都是强制的类，使用混合版本可能是一个好的解决方案。第四个选项在注入依赖时更简单，因为我们不需要知道每个对象期望什么。问题是每个类都应该知道如何获取其依赖，即依赖名称，这并不理想。

## 实现我们自己的依赖注入器

对于依赖注入器的开源解决方案已经可用，但我们认为亲自实现一个简单的依赖注入器会是一个很好的体验。我们的依赖注入器的想法是一个包含代码所需依赖实例的类。这个类，基本上是一个依赖名称到依赖实例的映射，将有两个方法：依赖的获取器和设置器。我们不想使用静态属性作为依赖数组，因为其中一个目标是有能力拥有多个具有不同依赖集的依赖注入器。将以下类添加到`src/Utils/DependencyInjector.php`：

```php
<?php

namespace Bookstore\Utils;

use Bookstore\Exceptions\NotFoundException;

class DependencyInjector {
    private $dependencies = [];

    public function set(string $name, $object) {
        $this->dependencies[$name] = $object;
    }

    public function get(string $name) {
        if (isset($this->dependencies[$name])) {
            return $this->dependencies[$name];
        }
        throw new NotFoundException(
            $name . ' dependency not found.'
        );
    }
}
```

拥有一个依赖注入器意味着每次我们请求一个给定类的实例时，我们都会使用相同的实例，而不是每次都创建一个新的实例。这意味着不再需要单例实现；实际上，正如在第四章中提到的，《使用面向对象编程创建整洁的代码》，避免它们是更好的选择。让我们摆脱它们吧。我们使用它的一个地方是在我们的配置读取器中。在`src/Core/Config.php`文件中将现有代码替换为以下代码：

```php
<?php

namespace Bookstore\Core;

use Bookstore\Exceptions\NotFoundException;

class Config {
    private $data;

    public function __construct() {
        $json = file_get_contents(
            __DIR__ . '/../../config/app.json'
        );
        $this->data = json_decode($json, true);
    }

    public function get($key) {
        if (!isset($this->data[$key])) {
            throw new NotFoundException("Key $key not in config.");
        }
        return $this->data[$key];
    }
}
```

我们使用单例模式的另一个地方是在`DB`类中。实际上，这个类的作用只是为我们提供一个数据库连接的单例，但如果我们没有使用它，我们可以删除整个类。所以，删除你的`src/Core/DB.php`文件。

现在我们需要定义所有这些依赖并将它们添加到我们的依赖注入器中。在路由请求之前，`index.php`文件是一个放置依赖注入器的好地方。在实例化`Router`类之前添加以下代码：

```php
$config = new Config();

$dbConfig = $config->get('db');
$db = new PDO(
    'mysql:host=127.0.0.1;dbname=bookstore',
    $dbConfig['user'],
    $dbConfig['password']
);

$loader = new Twig_Loader_Filesystem(__DIR__ . '/../../views');
$view = new Twig_Environment($loader);

$log = new Logger('bookstore');
$logFile = $config->get('log');
$log->pushHandler(new StreamHandler($logFile, Logger::DEBUG));

$di = new DependencyInjector();
$di->set('PDO', $db);
$di->set('Utils\Config', $config);
$di->set('Twig_Environment', $view);
$di->set('Logger', $log);

$router = new Router($di);
//...
```

现在我们需要做一些修改。其中最重要的修改是关于`AbstractController`类，这个类将会大量使用依赖注入器。给这个类添加一个名为`$di`的属性，并用以下代码替换构造函数：

```php
public function __construct(
    DependencyInjector $di,
    Request $request
) {
    $this->request = $request;
    $this->di = $di;

    $this->db = $di->get('PDO');
    $this->log = $di->get('Logger');
    $this->view = $di->get('Twig_Environment');
    $this->config = $di->get('Utils\Config');

    $this->customerId = $_COOKIE['id'];
}
```

其他更改涉及`Router`类，因为我们现在将其作为构造函数的一部分发送，我们需要将其注入到我们创建的控制器中。给这个类添加一个`$di`属性，并将构造函数更改为以下形式：

```php
public function __construct(DependencyInjector $di) {
    $this->di = $di;

    $json = file_get_contents(__DIR__ . '/../../config/routes.json');
    $this->routeMap = json_decode($json, true);
}
```

还需要更改`executeController`和`route`方法的内容：

```php
public function route(Request $request): string {
    $path = $request->getPath();

    foreach ($this->routeMap as $route => $info) {
        $regexRoute = $this->getRegexRoute($route, $info);
        if (preg_match("@^/$regexRoute$@", $path)) {
            return $this->executeController(
                $route, $path, $info, $request
            );
        }
    }

 $errorController = new ErrorController(
 $this->di,
 $request
 );
    return $errorController->notFound();
}

private function executeController(
    string $route,
    string $path,
    array $info,
    Request $request
): string {
    $controllerName = '\Bookstore\Controllers\\' 
        . $info['controller'] . 'Controller';
 $controller = new $controllerName($this->di, $request);

    if (isset($info['login']) && $info['login']) {
        if ($request->getCookies()->has('user')) {
            $customerId = $request->getCookies()->get('user');
            $controller->setCustomerId($customerId);
        } else {
 $errorController = new CustomerController(
 $this->di,
 $request
 );
            return $errorController->login();
        }
    }

    $params = $this->extractParams($route, $path);
    return call_user_func_array(
        [$controller, $info['method']], $params
    );
}
```

你还需要更改一个地方。`CustomerController`的`login`方法也在实例化控制器，所以我们也需要在那里注入依赖注入器：

```php
$newController = new BookController($this->di, $this->request);
```

# 摘要

在本章中，你学习了什么是 MVC，以及如何编写遵循该模式的程序。你还知道了如何使用路由器将请求路由到控制器，使用 Twig 编写模板，以及使用 Composer 管理你的依赖和自动加载器。你被介绍了依赖注入，甚至自己构建了一个实现，尽管这是一个非常有争议的话题，有很多人不同的观点。

在下一章中，我们将探讨编写良好代码和应用程序时最需要的部分之一：对你的代码进行单元测试以获得快速的反馈。
