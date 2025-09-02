# 第八章。路由、响应和事件

与许多现代 Web 框架一样，Yii2 使用了一个强大的路由组件，我们可以利用它来处理来自我们端用户和应用程序的各种 URI。这种功能通过 Yii2 强大的请求和响应处理器进一步增强，我们可以使用它们来操作请求和响应体。在本章中，我们将介绍如何操作 Yii2 的 URL Manager 以调整路由，探讨如何配置 Yii2 以以不同的方式响应，以及学习如何在我们的应用程序中发送和监听事件。

# 路由

如前几章所述，Yii2 中的路由由我们应用程序配置中定义的 UrlManager 组件管理。Yii2 中的路由器负责确定将 Yii2 的外部 URI 请求路由到内部控制器和动作的位置。在第五章模块、小部件和助手中，我们介绍了如何使用`yii\helpers\Url`助手创建和操作 URL 路由的基本方法。在本节中，我们将通过更详细地探讨 Yii2 的 UrlManager 来了解 Yii2 如何在应用程序内部路由这些请求。

Yii2 中的路由可以分为两个基本步骤。第一个步骤是解析传入的请求和查询参数（默认情况下存储在请求的`GET`参数中，带有`r`参数，但如果我们启用了美观的 URL，也可以从请求 URI 中检索）。第二个步骤是创建相应控制器动作的实例，它最终将处理请求。

默认情况下，Yii2 会将路由分解为 URL 中的正斜杠，将其映射到适当的模块、控制器和动作对。例如，site/login 路由将匹配默认应用程序实例中的`site`控制器和名为`login`的动作。内部，Yii2 将采取以下步骤来路由请求：

1.  默认情况下，Yii2 会将当前模块设置为应用程序。

1.  检查应用程序的控制器映射，看它是否包含当前路由。如果是，将根据模块内定义的控制器映射创建一个控制器实例。此时，将根据步骤*4*中定义的动作映射创建动作。默认情况下，Yii2 将根据`@app/controllers`文件夹中找到的控制器创建控制器映射，但可以在模块（或 UrlManager）中进行自定义：

    ```php
    yii\base\Module::$controllerMap = [
      'account' => 'app\controllers\UserController',
      // Different syntax for the previous example
      //'account' => [
      //   'class' => 'app\controllers\AccountController'
      //],
    ]
    ```

1.  如果应用程序模块的控制器映射不在应用程序模块中找到，Yii2 将遍历应用程序模块的`module`属性中的模块列表，以查看是否有匹配的路由。如果找到模块，Yii2 将使用提供的配置实例化该模块，然后根据前一步中概述的详细信息创建控制器。

1.  然后，Yii2 将在模块配置中定义的动作映射中查找动作。如果找到，它将根据该配置创建一个动作；如果没有找到，它将尝试创建与给定动作对应的`action`方法中定义的内联动作。

如果在此过程中任何一点发生错误，Yii2 将抛出`yii\web\NotFoundHttpException`。

## 默认和捕获所有路由

当 Yii2 收到一个解析为空路由的请求时，将使用默认路由。默认路由设置为`site/index`，它引用`site`控制器的`index`动作。可以通过设置`application`组件的`defaultRoute`属性来更改此行为，如下所示：

```php
[
    // index action of main controller
    'defaultRoute' => 'main/index' 
]
```

此外，可以通过设置`yii\web\application`的`catchAll`属性来配置 Yii2 将所有请求转发到单个路由。当需要执行应用程序维护时，这可能是有益的。

```php
[
    // Display a maintenance message
    'catchAll' => 'site/maintenance'
]
```

## 自定义路由和 URL 规则

而不是依赖于 Yii2 内部生成的默认控制器/动作路由，我们可以编写自定义 URL 规则来定义我们自己的 URL 路由。Yii2 中的 URL 路由是通过`yii\web\UrlRule`的一个实例实现的，它由用于修补给定路由的路径信息和查询参数的模式组成。当使用自定义 URL 规则时，Yii2 将根据伴随请求将请求路由到第一个匹配的规则。此外，匹配规则确定如何分割请求参数。此外，使用`yii\helpers\Url`辅助器也将依赖于规则列表来内部路由请求。

可以通过设置`yii\web\UrlManager:$rules`属性为一个包含要匹配的 URL 模式作为键和相应的路由作为值的数组来在应用程序配置中定义 Yii2 的 URL 规则。例如，假设我们有一个用于管理已发布内容的控制器，我们可以编写如下自定义规则，将`posts`和`post`路由到我们的内容：

```php
[
    'posts' => 'content/index', 
    'post/<id:\d+>' => 'content/view',
]
```

现在当我们导航到应用程序的`/posts`端点时，内容/index 控制器动作对将被触发。正如前一个示例所示，URL 规则可以超出简单的字符串，并且可以包含复杂的正则表达式，我们可以使用这些正则表达式来条件性地路由规则。在前一个示例中，到`/post`端点的路由后跟一个整数 ID 将路由到内容/view 控制器动作对。此外，Yii2 将自动将`$id`参数传递给动作：

```php
class ContentController extends yii\web\Controller
{
    // Responds to content/index and /posts
    public function actionIndex() {}

    // Responds to content/view/id/<id> or /post/<id>
    public function actionView($id) {}
}
```

### 小贴士

正则表达式只能指定给参数。然而，正如我们将在本节后面看到的，我们可以参数化我们的路由，使控制器和动作更加动态。

这些正则表达式可以进一步定制以包含更复杂的路由。例如，将以下内容添加到我们的 URL 路由中，将使我们能够向内容/index 动作传递额外的信息，例如我们想要显示的已发布条目的年份、月份和日期。

```php
// Creates a route that includes the year, month, and date of a post
// eg: https://www.example.com/posts/2015/09/01
[
'posts/<year:\d{4}>/<month:\d{2}>/<day:\d{2}>' => 'content/index',
]
```

如您从表达式中预期的那样，此路由将仅匹配四位数的年份和两位数的月份和日期。此外，如前所述，通过将此信息添加到我们的 URL 规则中，`yii\helper\Url`助手将理解使用此模式创建的任何 URL：

```php
// Routes to posts/2014/09/01
Url::to(['posts/index', 'year' => 2015, 'month' => 09, 'day' => 01]);
```

URL 路由也可以定义为路由域名和方案。例如，可以编写以下路由以确保不同的域名路由到网站的不同部分：

```php
[
    'https://dashboard.example.com/login' => 'dashboard/default/login',
    'https://www.example.com/login' => 'site/login'
]
```

在同一代码库中处理多个前端应用时，这很有益处。

### 参数化路由

除了上一节中描述的命名参数之外，参数还可以嵌入到 URL 规则本身中。这种方法使 Yii2 能够将单个规则匹配到多个路由，这可以大大减少 URL 规则的数量，从而提高路由器的性能。例如，以下路由：

```php
[
    '<controller:(content|comment)>/<id:\d+>/<action:(create|list|delete)>' => '<controller>/<action>',
]
```

此路由将匹配具有给定 ID 的内容和评论控制器，并传递给相应的操作。然而，为了使路由匹配，必须定义所有命名参数。如果给定的路由不包含给定的参数，Yii2 将无法匹配该路由，这很可能会导致路由遇到 404 错误。绕过这种限制的一种方法是为路由提供默认参数，如下例所示：

```php
[
    // ...other rules...
    [
        // :\d+ is a regular expression for integers
        'pattern' => 'content/<page:\d+>/<name>',
        'route' => 'content/index',
        'defaults' => ['page' => 1, 'name' => NULL],
    ],
]
```

在此示例中，`page`将默认为`1`，而`name`将默认为`NULL`。此 URL 规则可以匹配多个路由。在这个特定的情况下，将匹配多个路由：

+   `/content, page=1, name=NULL`

+   `/content/215, page=215, name=NULL`

+   `/content/215/foo, page=215, name=foo`

+   `/content/foo, page=1, name=foo`

### URL 后缀

作为为 URL 路由声明键值对的替代方法，路由可以定义为包含模式、路由甚至特定 URL 后缀的键值对数组的数组，以专门响应。

```php
[
    [
        'pattern' => 'posts',
        'route' => 'content/index',
        'suffix' => '.xml',
    ],
]
```

这些路由可以用来配置您的应用程序以以不同格式响应某些类型的请求。

### 小贴士

默认情况下，以这种方式创建的规则将作为`yii\web\UrlRule`的一个实例创建，但可以通过定义`class`参数来更改。

### 专门 HTTP 方法 URL 规则

有时，您可能会发现将不同类型的 HTTP 方法路由到同一路由但以不同的方式处理它们是有益的。在 Yii2 中，可以通过在路由键之前添加方法类型来实现，如下例所示：

```php
[
    'PUT,POST users/<id:\d+>' => 'users/create',
    'DELETE users/<id:\d+>' => 'users/delete',
    'GET users/<id:\d+>' => 'users/view',
]
```

从 API 的角度来看，所有请求最终都将路由到`users/<id>`，但根据 HTTP 方法的不同，将执行不同的操作。

### 小贴士

具有指定 HTTP 方法的 URL 规则将仅用于路由目的，并且它们不会用于创建 URL，例如在使用`yii\helper\Url`时。

## 自定义 URL 规则类

虽然`yii\web\Url`非常灵活，并且应该覆盖您需要的所有 URL 规则用例，但往往会有需要自定义 URL 的情况。例如，出版商可能希望支持一种表示作者和书籍的格式，如`/Author/Book`，其中`Author`和`Book`都是从数据库中检索的数据。在 Yii2 中，可以通过扩展`yii\base\Object`并实现`yii\web\UrlRuleInterface`来创建自定义 URL 规则以解决这个问题，如下面的示例所示：

```php
<?php

namespace app\components;

use yii\web\UrlRuleInterface;
use yii\base\Object;

class BookUrlRule extends Object implements UrlRuleInterface
{

    public function createUrl($manager, $route, $params)
    {
        if ($route === 'book/index')
        {
            if (isset($params['author'], $params['book']))
                return $params['author'] . '/' . $params['book'];
            else if (isset($params['author']))
                return $params['author'];
        }
        return false;
    }

    public function parseRequest($manager, $request)
    {
        $pathInfo = $request->getPathInfo();
        if (preg_match('%^(\w+)(/(\w+))?$%', $pathInfo, $matches))
        {
            // If the parameterized identified in $matches[] matches a database value
            // Set $params['author'] and $params['book'] to those attributes, then pass
            // those arguments to your route
            // return ['author/index', $params]
        }

        return false;
    }
}
```

我们可以在`yii\web\UrlManager::$rules`部分实现我们的自定义规则，通过声明我们希望使用该类：

```php
[
    // [...],
    [
        // Reuslts in URL's like https://www.example.com/charlesportwodii/mastering-yii
        'class' => 'app\components\BookUrlRule'
    ],
    // [...],
]
```

## 动态规则生成

规则可以通过多种不同的方式以编程和动态的方式添加到您的应用程序中。动态规则生成可以采用自定义 URL 规则类，如前文所述，或者自定义 URL 管理器。然而，向 URL 管理器添加新 URL 规则的最简单方法还是使用`addRules()`方法。为了使规则生效，它们需要在应用程序引导过程中较早出现。为了使模块能够动态添加新规则，它们应该实现`yii\base\BootstrapInterface`并在`bootstrap()`方法中添加自定义 URL 规则，如下面的示例所示：

```php
public function bootstrap($app)
{
    $app->getUrlManager()->addRules([
        // Add new rules here
    ], false);
}
```

### 小贴士

在复杂的 Web 应用程序中，监控您拥有的 URL 规则数量非常重要。添加许多不同的规则可能会严重降低应用程序的性能，因为 Yii2 需要遍历每个规则直到找到第一个匹配的规则。参数化路由和减少 URL 规则的数量可以显著提高应用程序的性能。

# 请求

在处理完我们希望请求去往的地方之后，我们通常需要编写特定的逻辑来处理 HTTP 请求的细节。为了帮助简化这个过程，Yii2 通过`yii\web\Request`对象表示 HTTP 请求，该对象可以提供有关 HTTP 请求的各种信息，例如请求体、`GET`和`POST`参数以及头部信息。在 Yii2 中，每个请求都可以通过请求应用程序组件轻松访问，在我们的代码中由`Yii::$app->request`表示。

## 检索请求参数和数据

当我们与请求对象一起工作时，最常见的工作任务是从`GET`和`POST`参数中检索数据，这些参数分别通过`yii\web\Request::get()`和`yii\web\Request::post()`方法实现。这些方法使我们能够一致且安全地访问应用程序的`$_GET`和`$_POST`参数：

```php
$request = \Yii::$app->request;

// Retrieve all of the $_GET parameters
// similar to $get = $_GET
$get = $request->get();
// Retrieve all of the $_POST parameters
$post = $request->post();
```

然而，与原生的`$_GET`和`$_POST`PHP 全局变量不同，Yii2 的请求对象允许我们安全地访问命名参数，如下一个示例所示：

```php
// Retrieves the name $_GET parameter, $_GET['id']
// https://www.example.com/controller/action/id/5
// https://www.example.com/controller/action?id=5
$id = $request->get('id');

// Retrieves the named $_POST parameter
$name = $request->post('name');
```

如果参数未定义，Yii2 默认将返回`NULL`。这种行为可以通过设置`yii\web\Request::get()`和`yii\web\Request::post()`的第二个参数来修改：

```php
// Default to 1 if ID is not set
$id = $request->get('id', 1);

// Default to 'Guest' if name is not set
$name = $request->post('name', 'Guest');
```

### 小贴士

除了提供对 `$_GET` 和 `$_POST` 数据的安全访问之外，请求对象在运行测试时也可以轻松地进行模拟。我们将在第十章 Testing with Codeception 中介绍如何与测试和模拟数据一起工作。*使用 Codeception 进行测试*。

作为额外的便利，Yii2 提供了确定我们正在处理的请求类型的能力，例如 `GET`、`POST` 或 `PUT` 请求。确定请求类型的最简单方法是查询 `\Yii::$app->request->method`，它将返回 HTTP 方法类型（例如 `GET`、`PUT`、`POST`、`DELETE` 等）。或者，我们可以通过查询许多布尔选项之一的条件检查请求，如下表所示：

| 属性 | 说明 |
| --- | --- |
| `yii\web\Request::$isAjax` | 如果请求是 AJAX (`XMLHTTPRequest`) 请求 |
| `yii\web\Request::$isConsoleRequest` | 如果请求是从控制台发出的 |
| `yii\web\Request::$isDelete` | 如果请求是 `HTTP DELETE` 请求 |
| `yii\web\Request::$isFlash` | 如果请求来自 Adobe Flex 或 Adobe Flash。 |
| `yii\web\Request::$isGet` | 如果请求是 `HTTP GET` 请求 |
| `yii\web\Request::$isHead` | 如果请求是 `HTTP HEAD` 请求 |
| `yii\web\Request::$isOptions` | 如果请求是 `HTTP OPTIONS` 请求 |
| `yii\web\Request::$isPatch` | 如果请求是 `HTTP PATCH` 请求 |
| `yii\web\Request::$isPjax` | 如果请求是 `HTTP PJAX` 请求 |
| `yii\web\Request::$isPost` | 如果请求是 `HTTP POST` 请求 |
| `yii\web\Request::$isPut` | 如果请求是 `HTTP PUT` 请求 |
| `yii\web\Request::$isSecureConnection` | 如果请求是通过安全（HTTPS）连接发出的 |

与作为表单发送的 `GET` 和 `POST` 请求不同，许多这些请求直接在请求体中提交数据。要访问这些数据，我们可以使用 `yii\web\Request::getBodyParam()` 和 `yii\web\Request::getBodyParams()`，如下所示：

```php
$request = Yii::$app->request;

$allParamns = $request->bodyParams;

$name = $request->getBodyParam('name');

$manyParams = $request->getBodyParams(['name', 'age', 'gender']); 
```

## 请求头部和 cookie

除了请求体之外，Yii2 的请求对象还可以检索随请求发送的头部和 cookie 信息。与我们请求一起发送的头部最终由 `yii\web\HeaderCollection` 表示，它提供了用于处理头部的一些方法，即 `yii\web\HeaderCollection::get()` 和 `yii\web\HeaderCollection::has()`。在下面的示例中，我们正在检查 `X-Auth-Token` 头部是否已设置，如果已设置，则将其分配给 `$authToken` 变量：

```php
// $headers is an object of yii\web\HeaderCollection 
$headers = Yii::$app->request->headers;

// If the header has 'X-Auth-Token', retrieve it.
if ($headers->has('X-Auth-Token'))
    $authToken = $headers->get('X-Auth-Token');
```

### 提示

如果没有为 `yii\web\HeaderCollection::get()` 方法提供参数，将返回所有头部的数组。有关 `yii\web\HeaderCollection` 的更多详细信息，请参阅 [`www.yiiframework.com/doc-2.0/yii-web-headercollection.html`](http://www.yiiframework.com/doc-2.0/yii-web-headercollection.html)。

请求对象有多个内置默认值，用于访问一些常用查询头，即：

+   `yii\web\Request::$userAgent` 获取浏览器发送的用户代理

+   可以使用 `yii\web\Request::$contentType` 来确定适当的响应类型

+   `yii\web\Request::$acceptableContentTypes` 返回客户端将接受的全部可接受的内容类型

+   如果我们的应用程序配置为支持多种语言，则可以使用 `yii\web\Request::$acceptableLanguages`。

### 小贴士

请求对象也可以通过 `yii\web\Request::getPreferedLanguage()` 告诉我们客户端的首选语言。我们将在第十一章 Internationalization and Localization 中更深入地使用这个变量和通用翻译与本地化。

除了我们的头部信息外，我们还可以通过查询 `Yii::$app->request->cookies` 来检索与我们的请求一起发送的 cookie 数据，这将返回一个 `yii\web\CookieCollection` 实例，正如你可能怀疑的那样，它包含许多与 `yii\web\HeaderCollection` 提供的相同类型的函数，例如 `get()` 和 `has()`。

### 小贴士

Yii2 API 文档在 [`www.yiiframework.com/doc-2.0/yii-web-cookiecollection.html`](http://www.yiiframework.com/doc-2.0/yii-web-cookiecollection.html) 提供了 `yii\web\CookieCollection` 的完整方法集。

## 获取客户端和 URL 信息

除了请求信息外，Yii2 请求对象还可以用来检索客户端和我们的应用程序状态信息。例如，可以使用 `yii\web\Request::$userHost` 和 `yii\web\Request::$userIP` 访问客户端信息，如主机名或 IP 地址。

### 小贴士

如果你的请求是通过代理或负载均衡器转发的，那么用户的 IP 地址可能不准确。请确保你的 Web 服务器已正确配置，以便传递原始数据。

可以通过引用各种方法来检查应用程序状态的数据，这些方法比查询 `$_SERVER` 全局变量更方便。以下表格显示了其中一些最常用的属性：

| 属性 | 说明 |
| --- | --- |
| `yii\web\Request::$absoluteUrl` | 这是包含主机名和所有 `GET` 参数的绝对 URL（例如，`https://www.example.com/controller/action/?name=foo`） |
| `yii\web\Request::$baseUrl` | 这是入口脚本之前使用的基 URL。 |
| `yii\web\Request::$hostInfo` | 这是主机详情（例如，`https://www.example.com`） |
| `yii\web\Request::$pathInfo` | 这是入口脚本之后的完整路径（例如，`/controller/action`） |
| `yii\web\Request::$queryString` | 这是 `GET` 查询字符串（例如，`name=foo`） |
| `yii\web\Request::$scriptUrl` | 这是没有路径和查询字符串的 URL（例如，`/index.php`） |
| `yii\web\Request::$serverName` | 这是服务器名称（例如，`example.com`） |
| `yii\web\Request::$serverPort` | 这是服务器正在运行的端口（通常为 TLS 连接的 80 或 443） |
| `yii\web\Request::$url` | 这是完整的 URL，但不包含主机和方案信息（例如，`controller/action/?name=foo`）。 |

### 提示

请求对象能够表示 HTTP 请求的几乎所有方面以及可能存储在`$_SERVER`全局变量中的数据。有关请求对象的更多信息，请参阅 Yii2 API 文档[`www.yiiframework.com/doc-2.0/yii-web-request.html`](http://www.yiiframework.com/doc-2.0/yii-web-request.html)。

# 响应

在完成请求对象的处理之后，Yii2 随后生成一个响应对象，并将其发送回客户端。响应包含大量信息，例如 HTTP 状态码、响应体和头部。在 Yii2 中，响应对象由`yii\web\Response`实现，它由`response`应用程序组件表示。在本节中，我们将探讨如何与响应对象一起工作。

## 设置状态码

在大多数情况下，Yii2 能够完全胜任将适当的状态码设置回最终用户；然而，可能存在需要我们明确为我们的应用程序定义 HTTP 响应码的情况。要修改我们应用程序中的 HTTP 状态码，我们只需将`yii\web\Response::$statusCode`设置为有效的 HTTP 状态码：

```php
\Yii::$app->response->statusCode = 200;
```

### 网络异常

默认情况下，Yii2 将为任何成功的请求返回 HTTP 200 状态码。如果我们想在不中断逻辑流程的情况下调整状态码，我们可以简单地为`yii\web\Response::$statusCode`定义一个新的状态码。在其他情况下，可能更好的做法是抛出异常，以在应用程序流程中引起短路，防止执行额外的逻辑。

通常，可以通过调用带有有效 HTTP 状态码的`yii\web\HttpException`来抛出网络异常：

```php
throw new \yii\web\HttpException(409);
```

为了方便起见，Yii2 为几种不同类型的请求提供了几个特定的方法，如下表所示：

| 异常 | 状态码 | HTTP 错误 |
| --- | --- | --- |
| `yii\web\BadRequestHttpException` | 400 | 错误请求 |
| `yii\web\UnauthorizedHtpException` | 401 | 未授权 |
| `yii\web\ForbiddenHttpException` | 403 | 禁止访问 |
| `yii\web\NotFoundHttpException` | 404 | 未找到 |
| `yii\web\MethodNotAllowedException` | 405 | 不允许的方法 |
| `yii\web\NotAcceptableHttpException` | 406 | 不可接受 |
| `yii\web\ConflictHttpException` | 409 | 冲突 |
| `yii\web\GoneHttpException` | 410 | 已消失 |
| `yii\web\UnsupportedMediaTypeHttpException` | 415 | 不支持的媒体类型 |
| `yii\web\TooManyRequestsHttpException` | 429 | 请求过多 |
| `yii\web\ServerErrorHttpException` | 500 | 服务器错误 |

### 提示

作为抛出一个带有给定状态码的空 `yii\web\HttpException` 的替代方案，你也可以扩展 `yii\web\HttpException` 来实现你自己的 `HttpException` 异常。

## 设置响应头

与 `yii\web\Request` 对象一样，我们可以使用 `yii\web\HeaderCollection` 的 `add()` 和 `remove()` 方法来操作我们响应的 HTTP 头部，如下面的示例所示：

```php
yii\web\HeaderCollection
$headers = Yii::$app->response->headers;

// Add two headers
$headers->add('X-Auth-Token', 'SADFLJKBQ43O7AGB28948QT');
$headers->add('Pragma', 'No-Cache');

// Remove a header
$headers->remove('Pragma');
```

### 提示

添加新头将覆盖具有相同名称的任何先前设置的头部。此外，通过 `yii\web\HeaderCollection` 设置的所有头部都是不区分大小写的。删除一个头部将删除任何具有当前发送名称的头部。

可以在任何时候操作头部，直到调用 `yii\web\Response::send()`，默认情况下，这是在响应体发送出去之前调用的。

## 响应体

通常，响应体将由 `yii\web\View` 的一个实例表示，这通常通过在控制器操作中返回渲染的视图来显示给最终用户，如下所示。默认情况下，Yii2 将以 MIME 类型 text/HTML 返回响应，并使用 `yii\web\HtmlResponseFormatter` 格式化响应：

```php
public function actionIndex()
{
    return $this->render('index');
}
```

然而，可能存在需要不同响应类型的情况，例如在显示 JSON 或 XML 数据时。在我们的控制器操作中，我们可以通过设置 `yii\web\Response::$format` 属性并返回表示我们想要格式化的数据的数组或字符串来更改输出格式，如下面的示例所示：

```php
public function actionIndex()
{
    \Yii::$app->response->format = \yii\web\Response::FORMAT_JSON;
    return [
        'message' => 'Index Action',
        'code' => 200,
    ];
}
```

之前的示例将输出以下 JSON 数据给客户端：

```php
{
    "message": "Index Action",
    "code": 200
}
```

除了 JSON 格式化外，`yii\web\Response::$format` 还可以设置为 JSONP、HTML、RAW 和 XML，具体细节如下表所示：

| 类型 | 格式化器类 | 格式值 |
| --- | --- | --- |
| HTML | `yii\web\HtmlResponseFormat` | `FORMAT_HTML` |
| RAW |   | `FORMAT_RAW` |
| XML | `yii\web\XmlResponseFormatter` | `FORMAT_XML` |
| JSON | `yii\web\JsonResponseFormatter` | `FORMAT_JSON` |
| JSONp | `yii\web\JsonResponseFormatter` | `FORMAT_JSON` |

### 提示

原始数据将按原样提交给客户端，不对其应用任何额外的格式。

除了处理默认响应对象外，在 Yii2 中，你还可以创建新的响应对象以发送给最终用户，如下面的示例所示：

```php
public function actionIndex()
{
    return \Yii::createObject([
        'class' => 'yii\web\Response',
        'format' => \yii\web\Response::FORMAT_JSON,
        'data' => [
            'message' => 'Index Action',
            'code' => 100,
        ],
    ]);
}
```

### 提示

为响应应用程序组件设置的任何自定义配置都不会应用于你实例化的任何自定义响应对象。

虽然控制器动作是您最可能编辑响应体的地方，但您可以通过直接操作`\Yii::$app->response`在任何地方修改 Yii2 中的响应体。任何已经格式化的数据都可以通过设置`yii\web\Response::$content`属性直接分配给响应对象。此外，如果您想在数据发送给用户之前通过响应格式化数据，可以设置`yii\web\Response::$content`，然后设置`yii\web\Response::$data`为要格式化的数据：

```php
$response = \Yii::$app->response;
$response->format = yii\web\Response::FORMAT_JSON;
$response->data = [
    'message' => 'Index Action',
    'code' => 100
]; 
```

## 重定向

为了将浏览器重定向到新页面，响应必须设置特殊的头部位置。Yii2 通过`yii\web\Response::redirect()`方法提供了对此的特殊支持，该方法可以在控制器动作内部调用，如下所示：

```php
public function actionIndex()
{
    return $this->redirect('https://www.example.com/index2');
}
```

### 小贴士

默认情况下，Yii2 将返回 302 状态码，表示重定向应该是临时的。为了通知浏览器永久重定向请求，可以将`yii\web\Response::redirect()`的第二个参数设置为 301，这是永久重定向的 HTTP 状态码。

在控制器动作外部，可以通过调用`redirect()`方法然后立即发送响应来调用重定向，如下例所示：

```php
\Yii::$app->response->redirect('https://www.example.com/index2', 301)->send();
```

## 文件输出

与浏览器重定向类似，输出文件到客户端需要设置几个自定义头部。为了便于将文件传输到浏览器，Yii2 提供了三种不同的方法来输出文件：

+   在发送位于磁盘上的现有文件时，应使用`yii\web\Response::sendFile()`。

+   `yii\web\Response::sendContentAsFile()`将数据字符串作为文件发送（例如 CSV 文件）

+   对于大文件（通常，文件大小超过 100 MB），应使用`yii\web\Response::sendStreamAsFile()`，并且它应该以流的形式发送到浏览器，因为它更节省内存。在控制器中，可以直接调用这些方法来发送文件：

    ```php
    public function actionReport()
    {
        return \Yii::$app->response->sendFile('path/to/report.csv');
    }
    ```

与重定向浏览器类似，这些方法可以通过直接操作响应对象在控制器动作外部调用：

```php
\Yii::$app->response->sendFile('path/to/report.csv')->send();
```

### 小贴士

关于响应对象的更多信息可以在 Yii2 文档中找到，请参阅[`www.yiiframework.com/doc-2.0/yii-web-response.html`](http://www.yiiframework.com/doc-2.0/yii-web-response.html)。

# 事件

在处理复杂的代码库时，我们可能会实现钩子和处理器，以便我们的应用程序可以在主应用程序流程之外调用自定义代码。在 Yii2 中，这些处理器被称为事件，当触发给定事件时可以自动执行。例如，在一个博客平台上，我们可能会创建一个事件来指示帖子已发布，这将触发一些自定义代码向特定邮件列表的用户发送电子邮件。在本节中，我们将介绍如何创建事件处理器、触发事件以及编写我们自己的自定义事件。

## 事件处理器

Yii2 中的事件是在`yii\base\Component`基类中实现的，这个基类几乎被 Yii2 中的每个类扩展。通过扩展这个类，我们可以在代码库的几乎任何地方绑定一个事件。要开始使用事件，我们首先需要创建一个事件处理器。

Yii2 中的事件处理器可以通过调用`yii\base\Component::on()`方法来绑定，并指定当事件被触发时应执行的回调。这些回调可以采取几种不同的形式，从作为字符串指定的全局 PHP 函数到在事件内联编写的匿名函数。例如，如果我们想调用一个全局 PHP 函数（如我们定义的或内置函数如`trim`），我们可以绑定我们的事件，如下所示：

```php
$thing = new app\Thing;
$thing->on(Thing::EVENT_NAME, 'php_function_name');
```

可以在任意 PHP 对象上调用事件处理器：要么是我们已经有实例变量的对象，要么是我们应用程序中的命名空间类：

```php
$thing->on(Thing::EVENT_NAME, [$object, 'method']);
$thing->on(Thing::EVENT_NAME, ['app\components\Thing ', 'doThing']);
```

此外，事件处理器可以编写为匿名函数：

```php
$thing->on(Thing::EVENT_NAME, function($event) {
    // Handle the event
});
```

可以通过将任何数据作为`yii\base\Component::on()`方法的第三个参数传递给事件处理器来传递额外的数据：

```php
$thing->on(Thing::EVENT_NAME, function($event) {
    echo $event->data['foo']; // bar
}, ['foo' => 'bar']);
```

此外，可以将多个事件处理器绑定到单个事件。当给定事件被触发时，每个事件将按照绑定到事件的顺序执行。如果事件处理器需要停止后续事件的执行，它可以将`$event`对象的`yii\base\Event::$handled`属性设置为`true`，这将防止所有绑定到该事件的处理器执行：

```php
$thing->on(Thing::EVENT_NAME, function($event) {
    // Handle the event
    $event->handled = true;
}, $data);
```

默认情况下，Yii2 中的事件处理器按照调用的顺序绑定，这意味着绑定到给定事件的最后一个事件处理器将最后被调用。要将在事件处理器队列的开头添加一个事件处理器，可以将`yii\base\Component::on()`方法的`$append`参数设置为`false`，这将覆盖默认行为，并在事件被触发时首先触发事件处理器：

```php
$thing->on(Thing::EVENT_NAME, function($event) {
    // Handle the event
}, [], false);
```

事件处理器也可以通过使用与附加事件监听器到事件相同的语法调用`yii\base\Component::off()`来从它们监听的事件中解绑。或者，可以通过调用`yii\base\Component::off()`而不带任何额外参数来从事件中解绑所有事件处理器，如下面的示例所示：

```php
$thing->off(Thing::EVENT_NAME);
```

## 触发事件

在 Yii2 中，通过调用`yii\base\Component::trigger()`方法来触发事件，该方法将事件名称作为第二个参数，并可选地传递`yii\base\Event`的实例作为第二个参数。例如，我们可以在代码中调用`Thing::EVENT_NAME`，如下所示：

```php
$this->trigger(Thing::EVENT_NAME);
```

此事件之前绑定如下：

```php
$thing->on(Thing::EVENT_NAME, ['app\components\Thing, 'doThing']);
```

现在，`app\components\Thing::doThing()`方法将被触发。这段代码对于我们的虚构组件可能如下所示：

```php
<?php
namespace app\components;

use yii\base\Component;
use yii\base\Event;

class Thing extends Component
{
    const EVENT_NAME = 'name';

    public function doThing()
    {
        // This is the event handler
    }
}
```

### 注意

Yii2 认为将事件名称存储为类中的常量是一种最佳实践。

可以通过扩展`yii\base\Event`并将其作为触发调用第二个参数传递，向我们的事件处理器发送附加信息，如下面的示例所示：

```php
<?php

namespace app\components;

use yii\base\Component;
use yii\base\Event;

class LogEvent extends Event
{
    public $message;
}

class Logger extends Component
{
    const EVENT_LOG = 'log_event';

    /**
     * Log with $message
     * @param string $message
     */
    public function log($message)
    {
        $event = new LogEvent;
        $event->message = $message;
        $this->trigger(self::EVENT_LOG, $event);
    }
}
```

由于 PHP 的单线程特性，Yii2 的事件将同步发生而不是异步发生，这将会阻塞所有其他应用程序流程，直到事件处理队列中的所有事件都完成。因此，在使用多个事件时应该小心，因为它们可能会对应用程序性能产生不利影响。

### 小贴士

使用实现异步事件（例如从 CMS 发送电子邮件通讯）的一种方法是将事件处理程序传递给第三方消息队列（如 Gearman、Sidekiq 或 Resque），并立即返回事件。然后，事件可以在单独的处理线程中处理，这可以是一个配置为从消息队列读取事件并单独处理它们的 Yii2 控制台命令。

## 类级事件

之前描述的事件是在实例级别绑定的。在 Yii2 中，事件可以绑定到类的每个实例，而不是特定实例，它们也可以绑定到 Yii2 的全局事件处理器。

类级事件可以通过直接通过`yii\base\Event::on()`附加事件处理器来绑定。例如，Active Record 在从数据库中删除记录时将触发`EVENT_AFTER_DELETE`事件。我们可以为每个 Active Record 实例记录此信息，如下面的示例所示：

```php
<?php

use Yii;
use yii\base\Event;
use yii\db\ActiveRecord;

Event::on(ActiveRecord::className(), ActiveRecord::EVENT_AFTER_DELETE, function ($event) {
    Yii::trace(get_class($event->sender) . ' deleted a record.');
});
```

每当触发器发生时，它将首先调用实例级事件处理器，然后调用类级事件处理器，最后调用全局事件处理器。可以通过直接调用`yii\base\Event::trigger()`显式调用类级事件。此外，可以通过`yii\base\Event::off()`移除类级事件处理器。

## 全局事件

在 Yii2 中，通过将事件处理器绑定到应用程序单例实例本身来支持全局事件，如下面的示例所示：

```php
<?php
use Yii;
use yii\base\Event;
use app\components\Foo;

Yii::$app->on('thing', function ($event) {
    echo get_class($event->sender);
});

Yii::$app->trigger('thing', new Event(['sender' => new Thing]));
```

### 小贴士

当使用全局事件时，要小心不要覆盖 Yii2 的内置全局事件。你使用的任何全局事件都应该包含某种前缀，以避免与 Yii2 的内置事件冲突。

# 摘要

本章我们介绍了 Yii2 中请求和响应处理的基本知识。我们首先探讨了 Yii2 如何处理 URL 路由的分配，并学习了如何操作和创建我们自己的自定义 URL 规则。然后，我们探讨了`yii\web\Request`和`yii\web\Response`对象，并更好地理解了如何使用这些对象来操作进入和离开我们应用程序的请求和响应。最后，我们学习了 Yii2 中事件的工作方式，并学习了如何创建我们自己的事件。

在下一章中，我们将通过探索如何实现 RESTful API，将本章中获得的知识提升到新的水平。
