## 响应中的状态码

如果 HTTP 方法对于请求非常重要，状态码对于响应几乎是不可或缺的。仅仅一个数字，客户端就能知道请求发生了什么。这在你知道状态码是一个标准，并且在互联网上有广泛的文档记录时特别有用。

我们已经在第二章中描述了最重要的状态码，*使用 PHP 的 Web 应用*，但让我们再次列出它们，并添加一些对 REST API 很重要的状态码。要查看状态码的完整列表，您可以访问[`www.w3.org/Protocols/rfc2616/rfc2616-sec10.html`](https://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html)。

### 2xx – 成功

所有以 2 开头的状态码都用于请求处理成功的响应，无论它是 GET 还是 POST。以下是一些这个类别中最常用的状态码：

+   **200 OK**：这是通用的“一切正常”响应。如果你请求一个资源，你将在响应体中获取它，如果你更新一个资源，这将意味着新数据已成功保存。

+   **201 created**：这是在 POST 或 PUT 操作成功创建资源时使用的响应。

+   **202 accepted**：这个响应意味着请求已被接受，但尚未处理。当客户端需要一个简单的响应来处理一个非常重的操作时，这可能很有用：服务器发送接受的响应，然后开始处理它。

### 3xx – 重定向

即使你可能认为只有一种重定向类型，但实际上还有一些细微的差别：

+   **301 moved permanently**：这意味着资源已经被移动到不同的 URL，因此从那时起，你应该尝试通过响应正文中提供的 URL 来访问它。

+   **303 see other**：这意味着请求已经处理，但为了看到响应，你需要访问响应正文中提供的 URL。

### 4xx – 客户端错误

这个类别包含描述由于客户端请求错误而导致的问题的状态码：

+   **400 bad request**：这是对格式错误的请求的通用响应，即端点存在语法错误，或者没有提供一些预期的参数。

+   **401 unauthorized**：这意味着客户端尚未成功认证，它试图访问的资源需要这种认证。

+   **403 forbidden**：这个错误消息意味着尽管客户端已经认证，但它没有足够的权限访问该资源。

+   **404 not found**：特定的资源未找到。

+   **405 method not allowed**：这意味着端点存在，但它不接受请求中使用的 HTTP 方法，例如，我们试图使用 PUT，但端点只接受 POST 请求。

### 5xx – 服务器错误

服务器端可能有多达 11 种不同的错误，但我们只对其中一种感兴趣：**500 内部服务器**错误。当在处理请求时发生意外情况，如数据库错误，你可以使用这个状态码。

## REST API 安全性

REST API 是一种强大的工具，因为它们允许开发者从服务器检索和/或更新数据。但权力越大，责任越大，在设计 REST API 时，你应该考虑使你的数据尽可能安全。想象一下——任何人都可以通过简单的 HTTP 请求代表你发布推文！

类似于使用网络应用程序，这里有两个概念：**认证**和**授权**。认证某人就是识别他是谁，也就是说，将他的请求与数据库中的用户关联起来。另一方面，授权某人就是允许该特定用户执行某些操作。你可以把认证看作是用户的登录，而授权则是赋予权限。

REST API 需要非常小心地管理这两个概念。仅仅因为开发者已经通过认证，并不意味着他可以访问服务器上的所有数据。有时，用户只能访问他们自己的数据，而有时你可能希望实现一个角色系统，其中每个角色有不同的访问级别。这始终取决于你正在构建的应用程序类型。

虽然授权发生在服务器端，即服务器数据库将决定给定的用户是否可以访问某个资源，但认证必须由客户端触发。这意味着客户端必须知道 REST API 使用的是哪种认证系统，以便进行认证。每个 REST API 都将实现自己的认证系统，但有一些知名的实施方式。

### 基本访问认证

基本访问认证——简称 BA——正如其名所示，非常基础。客户端在每个请求的头部添加关于用户的信息，即用户名和密码。问题是这些信息仅使用 BASE64 编码，而没有加密，这使得入侵者可以轻易解码头部并获取明文密码。如果你必须使用它，说实话，这是一种实现某种认证的非常简单的方法，我们建议你使用 HTTPS。

为了使用这种方法，你需要将用户名和密码像 `username:password` 一样连接起来，使用 Base64 对结果字符串进行编码，并将授权头添加为：

```php
Authorization: Basic <encoded-string>
```

### OAuth 2.0

如果基本认证非常简单且不安全，OAuth 2.0 就是 REST API 用来进行认证的最安全系统，之前的 OAuth 1.0 也是如此。实际上，这个标准有多个版本，但它们都建立在相同的基础上：

1.  没有用户名和密码。相反，REST API 的提供者会给开发者分配一对凭据——一个令牌和一个密钥。

1.  为了进行身份验证，开发者需要向“令牌”端点发送一个 POST 请求，每个 REST API 的端点都不同，但概念相同。这个请求必须包含编码的开发者凭据。

1.  服务器用会话令牌回复之前的请求。这个（而不是第一步中提到的凭据）需要包含在你向 REST API 发出的每个请求中。出于安全原因，会话令牌会过期，所以当这种情况发生时，你将不得不再次重复第二步。

尽管这个标准相对较新（从 2012 年开始），像谷歌或 Facebook 这样的几家大公司已经为它们的 REST API 实施了它。它可能看起来有点过于复杂，但你会很快学会使用它，甚至可以自己实现它。

# 使用第三方 API

关于 REST API 的理论就到这里；现在是时候深入一个真实世界的例子了。在本节中，我们将编写一个小型的 PHP 应用程序，与 Twitter 的 REST API 进行交互；这包括请求开发者凭据、进行身份验证和发送请求。目标是让你获得使用 REST API 的第一手经验，并展示它比你预期的要简单。这也有助于你更好地理解它们是如何工作的，因此以后构建自己的 API 会更容易。

## 获取应用程序的凭据

REST API 通常有应用程序的概念。应用程序就像它们开发网站上的一个账户，用于标识谁在使用 API。你将用来访问 API 的凭据将与该应用程序相关联，这意味着你可以有多个应用程序与同一个账户相关联。

假设你有一个 Twitter 账户，请访问[`apps.twitter.com`](https://apps.twitter.com)以创建一个新的应用程序。点击**创建新应用**按钮以访问应用程序详情表单。字段非常直观——只需为应用程序提供一个名称、描述和网站 URL。回调 URL 在这里不是必需的，因为它仅用于需要访问他人账户的应用程序。同意条款和条件以继续。

一旦你被重定向到你的应用程序页面，你会看到各种可以编辑的信息。由于这是一个示例，让我们直接进入重点：凭据。点击**密钥和访问令牌**选项卡以查看**消费者密钥（API 密钥）**和**消费者密钥（API 密钥）**的值。这里我们不需要其他任何东西。你可以将它们保存在你的文件系统中，例如`~/.twitter_php7.json`：

```php
{
    "key": "iTh4Mzl0EAPn9HAm98hEhAmVEXS",
    "secret": "PfoWM9yq4Bh6rGbzzJhr893j4r4sMIAeVRaPMYbkDer5N6F"
}
```

### 小贴士

**保护你的凭据**

保护您的 REST API 凭据应受到重视。实际上，您应该注意所有类型的凭据，如数据库凭据。但区别在于您通常会在自己的服务器上托管数据库，这使得攻击者稍微困难一些。另一方面，第三方 REST API 不是您系统的一部分，并且拥有您凭据的人可以代表您自由使用您的账户。

永远不要将您的凭据包含在代码库中，尤其是如果您在 GitHub 或其他存储库中有代码。一个解决方案是在您的服务器上创建一个文件，该文件位于您的代码之外，并包含凭据；如果该文件被加密，那就更好了。并且尝试定期刷新您的凭据，您可能可以在提供者的网站上完成此操作。

## 设置应用程序

我们的应用程序将非常简单。它将包含一个类，该类允许我们检索推文。这将由我们的 `app.php` 脚本管理。

由于我们必须进行 HTTP 请求，我们可以编写自己的函数，这些函数使用 **cURL**（一组 PHP 本地函数），或者使用著名的 PHP 库 **Guzzle**。这个库可以在 **Packagist** 中找到，因此我们将使用 Composer 来包含它：

```php
$ composer require guzzlehttp/guzzle

```

我们将有一个 `Twitter` 类，它将从构造函数中获取凭据，并且有一个公共方法：`fetchTwits`。目前，只需创建一个框架，以便我们可以使用它；我们将在后面的章节中实现这些方法。将以下代码添加到 `src/Twitter.php`：

```php
<?php

namespace TwitterApp;

class Twitter {

    private $key;
    private $secret;

    public function __construct(String $key, String $secret) {
        $this->key = $key;
        $this->secret = $secret;
    }

    public function fetchTwits(string name, int $count): array {
        return [];
    }
}
```

由于我们设置了命名空间 `TwitterApp`，我们需要更新我们的 `composer.json` 文件，并添加以下内容。请记住运行 `composer update` 以更新自动加载器。

```php
"autoload": {
    "psr-4": {"TwitterApp\\": "src"}
}
```

最后，我们将创建一个基本的 `app.php` 文件，该文件包含 Composer 自动加载器，读取凭据文件，并创建一个 Twitter 实例：

```php
<?php

use TwitterApp\Twitter;

require __DIR__ . '/vendor/autoload.php';

$path = $_SERVER['HOME'] . '/.twitter_php7.json';
$jsonCredentials = file_get_contents($path);
$credentials = json_decode($jsonCredentials, true);

$twitter = new Twitter($credentials['key'], $credentials['secret']);
```

## 请求访问令牌

在实际应用中，您可能希望将与认证相关的代码与处理数据检索或发布等操作的代码分开。为了保持简单，我们将让 `Twitter` 类知道如何自行进行认证。

让我们从给类添加一个 `$client` 属性开始，该属性将包含 Guzzle 的 `Client` 类的实例。这个实例将包含 Twitter API 的基本 URI，我们可以将其作为常量 `TWITTER_API_BASE_URI`。在构造函数中实例化此属性，以便其他方法可以使用它。您还可以添加一个 `$accessToken` 属性，该属性将包含 Twitter API 在认证时返回的访问令牌。所有这些更改在此处突出显示：

```php
<?php

namespace TwitterApp;

use Exception;
use GuzzleHttp\Client;

class Twitter {

 const TWITTER_API_BASE_URI = 'https://api.twitter.com';

    private $key;
    private $secret;
 private $accessToken;
 private $client;

    public function __construct(String $key, String $secret) {
        $this->key = $key;
        $this->secret = $secret;

 $this->client = new Client(
 ['base_uri' => self::TWITTER_API_BASE_URI]
 );
    }

    //...
}
```

下一步将是编写一个方法，给定密钥和秘密后，向提供者请求访问令牌。更具体地说：

+   将密钥和秘密使用 `:` 连接。使用 Base64 对结果进行编码。

+   向`/oauth2/token`发送带有编码凭证的 POST 请求作为`Authorization`头。还要包括`Content-Type`头和体（更多信息请查看代码）。

我们现在调用 Guzzle 的`client`实例的`post`方法，传递两个参数：端点字符串（`/oauth2/token`）和包含选项的数组。这些选项包括请求的头和体，您将很快看到。这个调用的响应是一个对象，它标识 HTTP 响应。您可以使用`getBody`提取响应的内容（体）。Twitter 的 API 响应是一个带有一些参数的 JSON。您最关心的参数是`access_token`，这是您需要在每个后续 API 请求中包含的令牌。提取它并保存。完整的方法如下：

```php
private function requestAccessToken() {
    $encodedString = base64_encode(
        $this->key . ':' . $this->secret
    );
    $headers = [
        'Authorization' => 'Basic ' . $encodedString,
        'Content-Type' => 'application/x-www-form-urlencoded;charset=UTF-8'
    ];
    $options = [
        'headers' => $headers,
        'body' => 'grant_type=client_credentials'
    ];

    $response = $this->client->post(self:: OAUTH_ENDPOINT, $options);
    $body = json_decode($response->getBody(), true);

    $this->accessToken = $body['access_token'];
}
```

您可以通过在构造函数末尾添加这两行代码来尝试此代码：

```php
$this->requestAccessToken();
var_dump($this->accessToken);
```

使用以下命令运行应用程序，以查看提供者提供的访问令牌。记住，为了继续本节，请删除前面的两行。

```php
$ php app.php

```

请记住，尽管拥有密钥和密钥并获取访问令牌在所有 OAuth 身份验证中都是相同的，但编码方式、使用的端点和从提供者收到的响应对于 Twitter 的 API 是独有的。可能还有其他几个是相同的，但始终检查每个的文档。

## 获取推文

我们最终到达了实际使用 API 的部分。我们将实现`fetchTwits`方法，以获取给定用户的最后*N*条推文列表。为了执行请求，我们需要在每个请求中添加`Authorization`头，这次使用访问令牌。由于我们希望尽可能使这个类可重用，让我们将其提取到一个私有方法中：

```php
private function getAccessTokenHeaders(): array {
    if (empty($this->accessToken)) {
        $this->requestAccessToken();
    }

    return ['Authorization' => 'Bearer ' . $this->accessToken];
}
```

如您所见，前面方法还允许我们从提供者那里获取访问令牌。这很有用，因为我们如果发出多个请求，我们只需请求一次访问令牌，并且有一个唯一的地方来做这件事。现在添加以下方法实现：

```php
const GET_TWITS = '/1.1/statuses/user_timeline.json';
//...
public function fetchTwits(string $name, int $count): array {
    $options = [
        'headers' => $this->getAccessTokenHeaders(),
        'query' => [
            'count' => $count,
            'screen_name' => $name
        ]
    ];

    $response = $this->client->get(self::GET_TWITS, $options);
    $responseTwits = json_decode($response->getBody(), true);

    $twits = [];
    foreach ($responseTwits as $twit) {
        $twits[] = [
            'created_at' => $twit['created_at'],
            'text' => $twit['text'],
            'user' => $twit['user']['name']
        ];
    }

    return $twits;
}
```

前面方法的第一个部分使用访问令牌头和查询字符串参数构建`options`数组——在这种情况下，包括要检索的推文数量和用户。我们执行 GET 请求并将 JSON 响应解码为数组。这个数组包含大量我们可能不需要的信息，因此我们迭代它以提取我们真正想要的字段——在这个例子中，日期、文本和用户。

为了测试应用程序，只需在`app.php`文件的末尾调用`fetchTwits`方法，指定您关注的某人的 Twitter ID 或您自己的 ID。

```php
$twits = $twitter->fetchTwits('neiltyson', 10);
var_dump($twits);
```

您应该得到一个类似于我们下面的响应，如以下截图所示：

![获取推文](img/00044.jpeg)

有一个需要注意的事情是，访问令牌在一段时间后会过期，返回一个带有 4xx 状态码的 HTTP 响应（通常是 401 未授权）。Guzzle 在状态码为 4xx 或 5xx 时抛出异常，因此很容易管理这些场景。你可以在执行 GET 请求时添加此代码：

```php
try {
    $response = $this->client->get(self::GET_TWITS, $options);
} catch (ClientException $e) {
 if ($e->getCode() == 401) {
        $this->requestAccessToken();
        $response = $this->client->get(self::GET_TWITS, $options);
    } else {
        throw $e;
    }
}
```

# REST API 开发者的工具集

当你正在开发自己的 REST API，或者为第三方 API 编写集成时，你可能想在开始编写代码之前测试它。有一些工具可以帮助你完成这项任务，无论你是想使用浏览器，还是你是命令行的粉丝。

## 使用浏览器测试 API

实际上，有几个插件允许你从浏览器执行 HTTP 请求，具体取决于你使用的是哪一个。一些著名的名字是 Chrome 上的*Advanced Rest Client*和 Firefox 上的*RESTClient*。最终，所有这些客户端都允许你执行相同的 HTTP 请求，你可以指定 URL、方法、头部、主体等。这些客户端还会显示你想象得到的所有响应细节，包括状态码、耗时和主体。以下截图显示了使用 Chrome 的*Advanced Rest Client*的一个请求示例：

![使用浏览器测试 API](img/00045.jpeg)

如果你想要使用自己的 API 测试 GET 请求，而你需要的只是 URL，也就是说，你不需要发送任何头部，你只需像访问任何其他网站一样使用你的浏览器。如果你这样做，并且如果你正在处理 JSON 响应，你可以安装另一个浏览器插件，这将帮助你以更“美观”的方式查看你的 JSON。在任何浏览器上查找*JSONView*，以获得一个真正方便的工具。

## 使用命令行测试 API

有些人觉得使用命令行更舒服；幸运的是，对于他们来说，有一些工具允许他们从控制台执行任何 HTTP 请求。我们将简要介绍其中最著名的一个：cURL。这个工具有很多功能，但我们只会关注你更常使用的那些：HTTP 方法、POST 参数和头部：

+   `-X <method>`：这指定了要使用的 HTTP 方法

+   `--data`：这添加了指定的参数，可以是键值对、JSON、纯文本等

+   `--header`：这会给请求添加一个头部

以下是一个使用 cURL 发送 POST 请求的示例：

```php
curl -X POST --data "text=This is sparta!" \
> --header "Authorization: Bearer 8s8d7bf8asdbf8sbdf8bsa" \
>  https://api.twitter.com/1.1/statuses/update.json
{"errors":[{"code":89,"message":"Invalid or expired token."}]}

```

如果你使用的是 Unix 系统，你可能会通过附加`| python -m json.tool`来格式化生成的 JSON，这样它就更容易阅读了：

```php
$ curl -X POST --data "text=This is sparta!" \
> --header "Authorization: Bearer 8s8d7bf8asdbf8sbdf8bsa" \
>  https://api.twitter.com/1.1/statuses/update.json \
> | python -m json.tool
{
 "errors": [
 {
 "code": 89,
 "message": "Invalid or expired token."
 }
 ]
}

```

cURL 是一个非常强大的工具，它让你可以做很多技巧。如果你感兴趣，请继续查看文档或一些教程，了解如何使用所有这些功能。

# 使用 REST API 的最佳实践

我们已经讨论了一些编写 REST API 的最佳实践，比如正确使用 HTTP 方法，或者为你的响应选择正确的状态码。我们还描述了两种最常用的认证系统。但关于创建合适的 REST API 还有很多东西要学习。记住，它们是为了像你这样的开发者而设计的，所以如果你做得正确，会使他们的生活变得更轻松。准备好了吗？

## 端点的一致性

在决定如何命名你的端点时，尽量保持一致性。尽管你可以自由选择，但有一套口语化的规则可以使你的端点更加直观和易于理解。让我们列举一些：

+   首先，一个端点应该指向一个特定的资源（例如，书籍或推文），你应该在你的端点中清楚地表明这一点。如果你有一个返回所有书籍列表的端点，不要命名为`/library`，因为它不明显会返回什么。相反，命名为`/books`或`/books/all`。

+   资源名称可以是复数也可以是单数，但请保持一致性。如果你有时使用`/books`，有时使用`/user`，可能会造成混淆，人们可能会犯错误。我们个人更喜欢使用复数形式，但这完全取决于你。

+   当你想检索特定的资源时，如果可能的话，通过指定 ID 来执行。ID 必须在你的系统中是唯一的，任何其他参数可能指向两个不同的实体。在资源名称旁边指定 ID，例如`/books/249234-234-23-42`。

+   如果你仅通过 HTTP 方法就能理解端点的作用，就没有必要将其作为端点的一部分添加信息。例如，如果你想获取一本书，或者删除它，使用`/books/249234-234-23-42`以及 HTTP 方法 GET 和 DELETE 就足够了。如果不明显，可以在端点末尾用动词表示，如`/employee/9218379182/promote`。

## 尽可能多地编写文档

标题应该包含所有信息。你很可能不会是使用 REST API 的人，其他人会。显然，即使你设计了一套非常直观的端点，开发者仍然需要知道所有可用的端点，每个端点的作用，可用的可选参数等等。

尽可能多地编写文档，并保持其更新。看看其他已记录的 API，以获取如何展示信息的想法。有许多模板和工具可以帮助你提供一份展示良好的文档，但你必须保持一致性和条理性。开发者特别讨厌编写文档，但当我们需要使用他人的 API 时，我们也喜欢找到清晰且美观展示的文档。

## 过滤和分页

API 的一种常见用途是列出资源并通过某些标准进行筛选。当我们构建自己的书店时，我们已经看到了一个例子；我们想要获取包含特定字符串在标题或作者中的书籍列表。

一些开发者试图拥有美观的端点，从先验的角度来看，这是一件好事。想象一下，如果你想仅通过标题进行筛选，你可能会得到一个像 `/books/title/<string>` 这样的端点。我们还增加了通过作者进行筛选的能力，现在我们得到了两个额外的端点：`/books/title/<string>/author/<string>` 和 `/books/author/<string>`。现在让我们也添加描述——你看到我们想要去哪里了吗？

尽管一些开发者不喜欢使用查询字符串作为参数，但这并没有什么问题。事实上，如果你正确使用它们，你最终会得到更干净的端点。你想获取书籍？好的，只需使用 `/books`，并使用查询字符串添加你需要的任何过滤器。

当你一次需要检索大量同类型资源时，就会发生分页。你应该将分页视为另一个可选的过滤器，作为 GET 参数指定。你应该有默认大小的页面，比如 10 本书，但给开发者定义他们自己的大小是一个好主意。在这种情况下，开发者可以指定要检索的长度和页数。

## API 版本控制

你的 API 是你应用程序能做什么的反映。很可能会发生你的代码会进化，改进现有的功能或添加新的功能。你的 API 也应该更新，以暴露这些新功能，更新现有端点，甚至删除其中的一些。

现在想象一下，有人正在使用你的 REST API，并且他们的整个网站都依赖于它。如果你更改现有的端点，他们的网站将停止工作！他们不会感到高兴，并试图找到其他人来做你之前做的事情。这不是一个好的场景，但那么，你如何改进你的 API 呢？

解决方案是使用版本控制。当你发布 API 的新版本时，不要删除现有的版本；你应该给用户一些时间来升级他们的集成。那么，两个不同的 API 版本如何共存呢？你已经看到了一个选项——我们推荐你使用的那个：通过在端点中指定要使用的 API 版本。你还记得 Twitter API 的端点 `/1.1/statuses/user_timeline.json` 吗？其中的 `1.1` 指的是我们想要使用的版本。

## 使用 HTTP 缓存

如果 REST API 的主要功能是大量使用 HTTP，为什么不利用 HTTP 缓存呢？好吧，不使用它的实际原因有很多，但大多数都是由于缺乏正确使用它的知识。本书的范围不包括解释其实施的每一个细节，但让我们尝试对这个主题进行简要介绍。互联网上有大量的资源可以帮助你理解你更感兴趣的部分。

HTTP 响应可以分为公开和私有。公开响应在 API 的所有用户之间共享，而私有响应则针对每个用户是唯一的。你可以使用 `Cache-Control` 头来指定你的响应类型，如果请求的方法是 GET，则允许缓存响应。此头还可以暴露缓存的过期时间，即你可以指定你的响应将保持相同的时间长度，因此可以被缓存。

其他系统依赖于生成资源的表示的哈希值，并将其作为 `ETag`（实体标签）头添加，以便知道资源是否已更改。以类似的方式，你可以设置 `Last-Modified` 头，让客户端知道给定资源最后一次更改的时间。这些系统背后的想法是确定客户端是否已经包含有效数据。如果是这样，提供者不会处理请求，而是返回一个状态码为 304（未修改）的空响应。当客户端收到这个响应时，它将使用其缓存的文件内容。

# 使用 Laravel 创建 REST API

在本节中，我们将从头开始使用 Laravel 构建一个 REST API。这个 REST API 将允许你管理书店中的不同客户，不仅可以通过浏览器，还可以通过用户界面。你将能够执行与之前几乎相同的行为，即列出书籍、购买它们、免费借用，等等。

一旦 REST API 完成，你应该从之前章节中构建的书店中移除所有业务逻辑。原因是你应该有一个唯一的地方来实际操作你的数据库和 REST API，而其他应用程序，如网站，应该能够与 REST API 通信来管理数据。这样做，你将能够为不同的平台创建其他应用程序，例如移动应用程序，它们也将使用 REST API，并且网站和移动应用程序将始终保持同步，因为它们将使用相同的来源。

就像我们之前的 Laravel 示例一样，为了创建一个新的项目，你只需要运行以下命令：

```php
$ laravel new bookstore_api

```

## 设置 OAuth2 认证

我们将要实现的第一件事是认证层。我们将使用 OAuth2 来使我们的应用程序比基本认证更安全。Laravel 并没有提供开箱即用的 OAuth2 支持，但有一个服务提供者为我们做了这件事。

### 安装 OAuth2Server

要安装 OAuth2，请使用 Composer 将其作为项目依赖项添加：

```php
$ composer require "lucadegasperi/oauth2-server-laravel:5.1.*"

```

此服务提供者需要进行相当多的更改。我们将简要介绍它们，而不会过多地深入到具体的工作原理。如果你对这个主题更感兴趣，或者如果你想为 Laravel 创建自己的服务提供者，我们建议你查阅详尽的官方文档。

首先，我们需要将新的 OAuth2Server 服务提供者添加到`config/app.php`文件中的提供者数组中。在`providers`数组的末尾添加以下几行：

```php
/*
 * OAuth2 Server Service Providers...
 */
        LucaDegasperi\OAuth2Server\Storage\FluentStorageServiceProvider::class,       LucaDegasperi\OAuth2Server\OAuth2ServerServiceProvider::class,
```

同样地，你需要在同一文件中的`aliases`数组中添加一个新的别名：

```php
'Authorizer' => LucaDegasperi\OAuth2Server\Facades\Authorizer::class,
```

让我们转到`app/Http/Kernel.php`文件，在那里我们还需要做一些更改。将以下条目添加到`Kernel`类的`$middleware`数组属性中：

```php
\LucaDegasperi\OAuth2Server\Middleware\OAuthExceptionHandlerMiddleware::class,
```

将以下键值对添加到同一类的`$routeMiddleware`数组属性中：

```php
'oauth' => \LucaDegasperi\OAuth2Server\Middleware\OAuthMiddleware::class,
'oauth-user' => \LucaDegasperi\OAuth2Server\Middleware\OAuthUserOwnerMiddleware::class,
'oauth-client' => \LucaDegasperi\OAuth2Server\Middleware\OAuthClientOwnerMiddleware::class,
'check-authorization-params' => \LucaDegasperi\OAuth2Server\Middleware\CheckAuthCodeRequestMiddleware::class,
'csrf' => \App\Http\Middleware\VerifyCsrfToken::class,
```

我们在之前的步骤中添加了 CSRF 令牌验证器到`$routeMiddleware`，因此我们需要从`$middlewareGroups`中移除已经定义的，因为它们是不兼容的。使用以下行来完成此操作：

```php
\App\Http\Middleware\VerifyCsrfToken::class,
```

### 设置数据库

现在让我们设置数据库。在本节中，我们假设你已经在你的环境中有了书店数据库。如果你没有，请回到第五章，*使用数据库*，以创建它以便继续此设置。

首先要做的事情是在`.env`文件中更新数据库凭证。它们应该看起来类似于以下几行，但需要使用你的用户名和密码：

```php
DB_HOST=localhost
DB_DATABASE=bookstore
DB_USERNAME=root
DB_PASSWORD=
```

为了准备 OAuth2Server 服务提供者的配置和数据库迁移文件，我们需要发布它。在 Laravel 中，你可以通过执行以下命令来完成：

```php
$ php artisan vendor:publish

```

现在的`database/migrations`目录包含了所有必要的迁移文件，这些文件将在我们的数据库中创建与 OAuth2 相关的必要表。要执行它们，我们运行以下命令：

```php
$ php artisan migrate

```

我们需要至少向`oauth_clients`表添加一个客户端，这是存储所有想要连接到我们的 REST API 的客户端的密钥和秘密的表。这个新的客户端将是你将在开发过程中用来测试你所做事情的一个。我们可以设置一个随机的 ID——密钥——以及秘密如下：

```php
mysql> INSERT INTO oauth_clients(id, secret, name)
 -> VALUES('iTh4Mzl0EAPn90sK4EhAmVEXS',
 -> 'PfoWM9yq4Bh6rGbzzJhr8oDDsNZwGlsMIAeVRaPM',
 -> 'Toni');
Query OK, 1 row affected, 1 warning (0.00 sec)

```

### 启用客户端凭证认证

由于我们在上一步中发布了`vendor`中的插件，现在我们有了 OAuth2Server 的配置文件。此插件允许我们根据需要使用不同的认证系统（所有这些系统都使用 OAuth2）。我们感兴趣的是我们的项目中的`client_credentials`类型。为了让 Laravel 知道，请在`config/oauth2.php`文件中的数组末尾添加以下几行：

```php
'grant_types' => [
     'client_credentials' => [
        'class' => 
            '\League\OAuth2\Server\Grant\ClientCredentialsGrant',
        'access_token_ttl' => 3600
    ]
]
```

前面的这些行授予了 `client_credentials` 类型的访问权限，这些权限由 `ClientCredentialsGrant` 类管理。`access_token_ttl` 值指的是访问令牌的时间段，即某人可以使用它多长时间。在这种情况下，它被设置为 1 小时，即 3,600 秒。

最后，我们需要启用一个路由，以便我们可以通过提交凭证来交换访问令牌。将以下路由添加到 `app/Http/routes.php` 文件中的路由文件：

```php
Route::post('oauth/access_token', function() {
    return Response::json(Authorizer::issueAccessToken());
});
```

### 请求访问令牌

是时候测试我们到目前为止所做的工作了。为此，我们需要向刚才启用的 `/oauth/access_token` 端点发送一个 POST 请求。此请求需要以下 POST 参数：

+   使用数据库中的密钥来指定 `client_id`

+   使用数据库中的密钥来指定 `client_secret`

+   使用 `grant_type` 来指定我们尝试执行的认证类型，在这种情况下是 `client_credentials`

使用 Chrome 的 *Advanced REST Client* 插件发出的请求如下所示：

![请求访问令牌](img/00046.jpeg)

你应该得到的响应应该与这个格式相同：

```php
{
    "access_token": "MPCovQda354d10zzUXpZVOFzqe491E7ZHQAhSAax"
    "token_type": "Bearer"
    "expires_in": 3600
}
```

注意，这与 Twitter API 请求访问令牌的方式不同，但理念仍然是相同的：给定一个密钥和一个密钥，提供者会给我们一个访问令牌，这将允许我们在一段时间内使用该 API。

## 准备数据库

尽管我们在上一章中已经做了同样的事情，你可能会想：“为什么我们要先准备数据库？”我们可以争论说，你首先需要知道你想要在 REST API 中公开的端点类型，然后你才能开始考虑你的数据库应该是什么样子。但你也可能会认为，因为我们正在使用 API，每个端点应该管理一个资源，所以首先你需要定义你正在处理的数据。这种 *代码优先与数据库/模型优先* 是互联网上持续进行的战争。但无论你认为哪种方式更好，事实是，我们已经知道用户将如何使用我们的 REST API，因为我们之前已经构建了 UI；所以这并不真正重要。

我们需要创建四个表：`books`、`sales`、`sales_books` 和 `borrowed_books`。请记住，Laravel 已经提供了一个 `users` 表，我们可以将其用作我们的客户。运行以下四个命令来创建迁移文件：

```php
$ php artisan make:migration create_books_table --create=books
$ php artisan make:migration create_sales_table --create=sales
$ php artisan make:migration create_borrowed_books_table \
--create=borrowed_books
$ php artisan make:migration create_sales_books_table \
--create=sales_books

```

现在，我们需要逐个文件地定义每个表应该是什么样子。我们将尽可能复制 第五章 中的数据结构，即 *使用数据库*。请记住，迁移文件可以在 `database/migrations` 目录中找到。我们可以编辑的第一个文件是 `create_books_table.php`。用以下方法替换现有的空 `up` 方法：

```php
public function up()
{
    Schema::create('books', function (Blueprint $table) {
        $table->increments('id');
        $table->string('isbn')->unique();
        $table->string('title');
        $table->string('author');
        $table->smallInteger('stock')->unsigned();
        $table->float('price')->unsigned();
    });
}
```

列表中的下一个是 `create_sales_table.php`。请记住，这个文件有一个外键指向 `users` 表。您可以使用 `references(field)->on(tablename)` 来定义这个约束。

```php
public function up()
{
    Schema::create('sales', function (Blueprint $table) {
        $table->increments('id');
        $table->string('user_id')->references('id')->on('users');
        $table->timestamps();
    });
}
```

`create_sales_books_table.php` 文件包含两个外键：一个指向销售记录的 ID，另一个指向书籍的 ID。将现有的 `up` 方法替换为以下内容：

```php
public function up()
{
    Schema::create('sales_books', function (Blueprint $table) {
        $table->increments('id');
        $table->integer('sale_id')->references('id')->on('sales');
        $table->integer('book_id')->references('id')->on('books');
        $table->smallInteger('amount')->unsigned();
    });
}
```

最后，编辑 `create_borrowed_books_table.php` 文件，该文件包含 `book_id` 外键以及 `start` 和 `end` 时间戳：

```php
public function up()
{
    Schema::create('borrowed_books', function (Blueprint $table) {
        $table->increments('id');
        $table->integer('book_id')->references('id')->on('books');
        $table->string('user_id')->references('id')->on('users');
        $table->timestamp('start');
        $table->timestamp('end');
    });
}
```

迁移文件已经准备好了，所以我们只需按顺序迁移它们以创建数据库表。运行以下命令：

```php
$ php artisan migrate

```

此外，手动添加一些书籍到数据库中，以便您可以稍后进行测试。例如：

```php
mysql> INSERT INTO books (isbn,title,author,stock,price) VALUES
 -> ("9780882339726","1984","George Orwell",12,7.50),
 -> ("9789724621081","1Q84","Haruki Murakami",9,9.75),
 -> ("9780736692427","Animal Farm","George Orwell",8,3.50),
 -> ("9780307350169","Dracula","Bram Stoker",30,10.15),
 -> ("9780753179246","19 minutes","Jodi Picoult",0,10);
Query OK, 5 rows affected (0.01 sec)
Records: 5  Duplicates: 0  Warnings: 0

```

## 设置模型

列表中的下一件事是添加我们的数据具有的关系，即从数据库将外键转换为模型。首先，我们需要创建这些模型，为此我们只需运行以下命令：

```php
$ php artisan make:model Book
$ php artisan make:model Sale
$ php artisan make:model BorrowedBook
$ php artisan make:model SalesBook

```

现在我们必须逐个模型进行操作，并添加一对一和多对一关系，就像我们在上一章中做的那样。对于 `BookModel`，我们只需指定该模型没有时间戳，因为它们默认就有。为此，将以下高亮行添加到您的 `app/Book.php` 文件中：

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Book extends Model
{
 public $timestamps = false;
}
```

对于 `BorrowedBook` 模型，我们需要指定它有一个书籍，并且属于一个用户。我们还需要指定在创建对象时需要填充的字段——在这种情况下，`book_id` 和 `start`。在 `app/BorrowedBook.php` 中添加以下两个方法：

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class BorrowedBook extends Model
{
 protected $fillable = ['user_id', 'book_id', 'start'];
 public $timestamps = false;

 public function user() {
 return $this->belongsTo('App\User');
 }

 public function book() {
 return $this->hasOne('App\Book');
 }
}
```

销售记录可以有多个“销售书籍”（我们知道这可能听起来有点别扭），它们也只属于一个用户。将以下内容添加到您的 `app/Sale.php` 文件中：

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Sale extends Model
{
 protected $fillable = ['user_id'];

 public function books() {
 return $this->hasMany('App\SalesBook');
 }

 public function user() {
 return $this->belongsTo('App\User');
 }
}
```

与借阅书籍一样，销售书籍可以有一个书籍和一个销售记录，而不是一个用户。以下行应添加到 `app/SalesBook.php` 文件中：

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class SaleBook extends Model
{
 public $timestamps = false;
 protected $fillable = ['book_id', 'sale_id', 'amount'];

 public function sale() {
 return $this->belongsTo('App\Sale');
 }

 public function books() {
 return $this->hasOne('App\Book');
 }
}
```

最后，我们需要更新的最后一个模型是 `User` 模型。我们需要添加与之前在 `Sale` 和 `BorrowedBook` 中使用的 `belongs` 相反的关系。添加这两个函数，并保持类的其余部分不变：

```php
<?php

namespace App;

use Illuminate\Foundation\Auth\User as Authenticatable;

class User extends Authenticatable
{
    //...

 public function sales() {
 return $this->hasMany('App\Sale');
 }

 public function borrowedBooks() {
 return $this->hasMany('App\BorrowedBook');
 }
}
```

## 设计端点

在本节中，我们需要想出我们想要向 REST API 客户端公开的端点列表。请记住在“REST API 最佳实践”部分中解释的“规则”。简而言之，请记住以下规则：

+   一个端点与一个资源交互

+   可能的架构可以是 `<API 版本>/<资源名称>/<可选 ID>/<可选操作>`

+   使用 GET 参数进行过滤和分页

那么，用户需要做什么呢？由于我们已经创建了 UI，所以我们已经有了很好的想法。简要总结如下：

+   列出所有可用的书籍，并按标题和作者进行过滤，必要时进行分页。也可以根据 ID 检索特定书籍的信息。

+   如果有可用，允许用户借阅特定的书籍。同样，用户应该能够归还书籍，并列出借阅书籍的历史记录（按日期过滤并分页）。

+   允许用户购买一系列书籍。这可以改进，但现在让我们强制用户通过一个请求购买书籍，包括书籍的完整列表。同样，列出用户的销售记录也遵循与借阅书籍相同的规则。

我们将立即开始我们的端点列表，指定路径、HTTP 方法和可选参数。它还将给你一个如何记录 REST API 的想法。

+   GET `/books`

    +   `title`: 可选，按标题过滤

    +   `author`: 可选，按作者过滤

    +   `page`: 可选，默认为 1，指定要返回的页面

    +   `page-size`: 可选，默认为 50，指定返回的页面大小

+   GET `/books/<book id>`

+   POST `/borrowed-books`

    +   `book-id`: 必须指定，用于借阅书籍的 ID

+   GET `/borrowed-books`

    +   `from`: 可选，从指定的日期返回借阅的书籍

    +   `page`: 可选，默认为 1，指定要返回的页面

    +   `page-size`: 可选，默认为 50，指定每页的借阅书籍数量

+   PUT `/borrowed-books/<borrowed book id>/return`

+   POST `/sales`

    +   `books`: 必须指定，它是一个数组，列出要购买的书籍 ID 及其数量，即*{"book-id-1": 数量, "book-id-2": 数量, ...}*

+   GET `/sales`

    +   `from`: 可选，从指定的日期返回借阅的书籍

    +   `page`: 可选，默认为 1，指定要返回的页面

    +   `page-size`: 可选，默认为 50，指定每页的销售数量

+   GET `/sales/<sales id>`

我们在创建销售和借阅书籍时使用 POST 请求，因为我们事先不知道要创建的资源 ID，发送相同的请求将创建多个资源。另一方面，在归还书籍时，我们知道借阅书籍的 ID，发送相同的请求多次将使数据库保持相同的状态。让我们将这些端点翻译为`app/Http/routes.php`中的路由：

```php
/*
 * Books endpoints.
 */
Route::get('books', ['middleware' => 'oauth',
    'uses' => 'BookController@getAll']);
Route::get('books/{id}', ['middleware' => 'oauth',
    'uses' => 'BookController@get']);
/*
 * Borrowed books endpoints.
 */
Route::post('borrowed-books', ['middleware' => 'oauth',
    'uses' => 'BorrowedBookController@borrow']);
Route::get('borrowed-books', ['middleware' => 'oauth',
    'uses' => 'BorrowedBookController@get']);
Route::put('borrowed-books/{id}/return', ['middleware' => 'oauth',
    'uses' => 'BorrowedBookController@returnBook']);
/*
 * Sales endpoints.
 */
Route::post('sales', ['middleware' => 'oauth',
    'uses' => 'SalesController@buy]);
Route::get('sales', ['middleware' => 'oauth',
    'uses' => 'SalesController@getAll']);
Route::get('sales/{id}', ['middleware' => 'oauth',
    'uses' => 'SalesController@get']);
```

在前面的代码中，注意我们如何向所有端点添加了中间件`oauth`。这将要求用户提供有效的访问令牌才能访问它们。

## 添加控制器

从前面的部分，你可以想象我们需要创建三个控制器：`BookController`、`BorrowedBookController`和`SalesController`。让我们从最简单的一个开始：根据 ID 返回书籍的信息。创建文件`app/Http/Controllers/BookController.php`，并添加以下代码：

```php
<?php

namespace App\Http\Controllers;

use App\Book;
use Illuminate\Http\JsonResponse;
use Illuminate\Http\Response;

class BookController extends Controller {

    public function get(string $id): JsonResponse {
        $book = Book::find($id);

        if (empty($book)) {
            return new JsonResponse (
                null,
                JsonResponse::HTTP_NOT_FOUND
            );
        }

        return response()->json(['book' => $book]);
    }
}
```

尽管前面的例子很简单，但它包含了我们接下来需要的大部分内容。我们尝试根据 URL 中的 ID 获取一本书，如果没有找到，我们回复一个 404（未找到）的空响应——常量`Response::HTTP_NOT_FOUND`是 404。如果我们有这本书，我们用`response->json()`将其作为 JSON 返回。注意我们添加了看似不必要的键`book`；确实我们没有返回其他任何内容，并且由于我们请求了书籍，用户会知道我们在说什么，但既然这并不会造成伤害，最好尽可能明确。

让我们来测试一下！你已经知道如何获取访问令牌——查看“请求访问令牌”部分。所以获取一个，并尝试访问以下 URL：

+   `http://localhost/books/0?access_token=12345`

+   `http://localhost/books/1?access_token=12345`

假设`12345`是你的访问令牌，你有一个 ID 为`1`的书籍在数据库中，你没有 ID 为`0`的书籍，第一个 URL 应该返回一个 404 响应，第二个 URL，返回的响应类似于以下内容：

```php
{
    "book": {
        "id": 1
        "isbn": "9780882339726"
        "title": "1984"
        "author": "George Orwell"
        "stock": 12
        "price": 7.5
    }
}
```

现在我们添加一个方法来获取带有过滤器和分页的所有书籍。这看起来相当冗长，但我们使用的逻辑相当简单：

```php
public function getAll(Request $request): JsonResponse {
    $title = $request->get('title', '');
    $author = $request->get('author', '');
    $page = $request->get('page', 1);
    $pageSize = $request->get('page-size', 50);

    $books = Book::where('title', 'like', "%$title%")
        ->where('author', 'like', "%$author%")
        ->take($pageSize)
        ->skip(($page - 1) * $pageSize)
        ->get();

    return response()->json(['books' => $books]);
}
```

我们获取所有可能从请求中来的参数，并在用户没有包含它们的情况下设置每个参数的默认值（因为它们是可选的）。然后，我们使用 Eloquent ORM 通过`where()`过滤标题和作者，并使用`take()->skip()`限制结果。我们以与之前相同的方式返回 JSON。不过，在这个方法中，我们不需要任何额外的检查；如果查询没有返回任何书籍，这并不是真正的问题。

你现在可以玩转你的 REST API 了，发送带有不同过滤器的不同请求。以下是一些示例：

+   `http://localhost/books?access_token=12345`

+   `http://localhost/books?access_token=12345&title=19&page-size=1`

+   `http://localhost/books?access_token=12345&page=2`

列表中的下一个控制器是`BorrowedBookController`。我们需要添加三个方法：`borrow`、`get`和`returnBook`。既然你已经知道如何处理请求、响应、状态码和 Eloquent ORM，我们将直接编写整个类：

```php
<?php

namespace App\Http\Controllers;

use App\Book;
use App\BorrowedBook;
use Illuminate\Http\JsonResponse;
use Illuminate\Http\Request;
use LucaDegasperi\OAuth2Server\Facades\Authorizer;

class BorrowedBookController extends Controller {

    public function get(): JsonResponse {
        $borrowedBooks = BorrowedBook::where(
            'user_id', '=', Authorizer::getResourceOwnerId()
        )->get();

        return response()->json(
            ['borrowed-books' => $borrowedBooks]
        );
    }

    public function borrow(Request $request): JsonResponse {
        $id = $request->get('book-id');

        if (empty($id)) {
            return new JsonResponse(
                ['error' => 'Expecting book-id parameter.'],
                JsonResponse::HTTP_BAD_REQUEST
            );
        }

        $book = Book::find($id);

        if (empty($book)) {
            return new JsonResponse(
                ['error' => 'Book not found.'],
                JsonResponse::HTTP_BAD_REQUEST
            );
        } else if ($book->stock < 1) {
            return new JsonResponse(
                ['error' => 'Not enough stock.'],
                JsonResponse::HTTP_BAD_REQUEST
            );
        }

 $book->stock--;
 $book->save();

 $borrowedBook = BorrowedBook::create(
 [
 'book_id' => $book->id,
 'start' => date('Y-m-d H:i:s'),
 'user_id' => Authorizer::getResourceOwnerId()
 ]
 );

        return response()->json(['borrowed-book' => $borrowedBook]);
    }

    public function returnBook(string $id): JsonResponse {
        $borrowedBook = BorrowedBook::find($id);

        if (empty($borrowedBook)) {
            return new JsonResponse(
                ['error' => 'Borrowed book not found.'],
                JsonResponse::HTTP_BAD_REQUEST
            );
        }

        $book = Book::find($borrowedBook->book_id);
        $book->stock++;
        $book->save();

        $borrowedBook->end = date('Y-m-d H:m:s');
        $borrowedBook->save();

        return response()->json(['borrowed-book' => $borrowedBook]);
    }
}
```

在前面的代码中需要注意的唯一一点是我们如何更新书籍的库存（增加或减少库存），并调用`save`方法来保存数据库中的更改。在借阅书籍时，我们还返回借阅的书籍对象作为响应，以便用户可以知道借阅的书籍 ID，并在查询或归还书籍时使用它。

你可以使用以下用例来测试这组端点的工作方式：

+   借一本书。确认你得到了有效的响应。

+   获取借阅书籍的列表。你刚刚创建的书籍应该在那里，有一个有效的开始日期和一个空结束日期。

+   获取你借阅的书籍信息。库存应该少一本。

+   归还书籍。获取借阅书籍列表以检查结束日期和归还的书籍以检查库存。

当然，你总是可以尝试欺骗 API，请求没有库存的书、不存在的借阅书籍等。所有这些边缘情况都应该返回正确的状态码和错误信息。

我们通过创建`SalesController`来完成本节和 REST API。这个控制器包含更多的逻辑，因为创建销售意味着在检查每个书的库存是否足够之前，先向销售账本表添加条目。将以下代码添加到`app/Http/SalesController.php`：

```php
<?php

namespace App\Http\Controllers;

use App\Book;
use App\Sale;
use App\SalesBook;
use Illuminate\Http\JsonResponse;
use Illuminate\Http\Request;
use LucaDegasperi\OAuth2Server\Facades\Authorizer;

class SalesController extends Controller {

    public function get(string $id): JsonResponse {
        $sale = Sale::find($id);

        if (empty($sale)) {
            return new JsonResponse(
                null,
                JsonResponse::HTTP_NOT_FOUND
            );
        }

        $sale->books = $sale->books()->getResults();
        return response()->json(['sale' => $sale]);
    }

    public function buy(Request $request): JsonResponse {
        $books = json_decode($request->get('books'), true);

        if (empty($books) || !is_array($books)) {
            return new JsonResponse(
                ['error' => 'Books array is malformed.'],
                JsonResponse::HTTP_BAD_REQUEST
            );
        }

        $saleBooks = [];
        $bookObjects = [];
        foreach ($books as $bookId => $amount) {
            $book = Book::find($bookId);
            if (empty($book) || $book->stock < $amount) {
                return new JsonResponse(
                    ['error' => "Book $bookId not valid."],
                    JsonResponse::HTTP_BAD_REQUEST
                );
            }

            $bookObjects[] = $book;
            $saleBooks[] = [
                'book_id' => $bookId,
                'amount' => $amount
            ];
        }

        $sale = Sale::create(
            ['user_id' => Authorizer::getResourceOwnerId()]
        );
        foreach ($bookObjects as $key => $book) {
            $book->stock -= $saleBooks[$key]['amount'];

            $saleBooks[$key]['sale_id'] = $sale->id;
            SalesBook::create($saleBooks[$key]);
        }

        $sale->books = $sale->books()->getResults();
        return response()->json(['sale' => $sale]);
    }

    public function getAll(Request $request): JsonResponse {
        $page = $request->get('page', 1);
        $pageSize = $request->get('page-size', 50);

        $sales = Sale::where(
                'user_id', '=', Authorizer::getResourceOwnerId()
             )
            ->take($pageSize)
            ->skip(($page - 1) * $pageSize)
            ->get();

        foreach ($sales as $sale) {
            $sale->books = $sale->books()->getResults();
        }

        return response()->json(['sales' => $sales]);
    }
}
```

在前面的代码中，注意我们首先检查所有书的可用性，然后再创建销售条目。这样，我们确保在向用户返回错误时，数据库中没有未完成的销售记录。你可以改变这一点，使用事务代替，如果一本书无效，只需回滚事务。

为了测试这一点，我们可以遵循与借阅书籍类似的步骤。记住，在发布销售时，`books`参数是一个 JSON 映射；例如，`{"1": 2, "4": 1}`表示我正在尝试购买 ID 为`1`的两本书和一本 ID 为`4`的书。

# 测试你的 REST API

你已经在完成每个控制器后通过发起一些请求并期望得到响应来测试你的 REST API。正如你可能想象的那样，这有时可能很有用，但肯定不是最佳做法。测试应该是自动化的，并且应该尽可能覆盖更多内容。我们必须考虑一个类似于单元测试的解决方案。

在第十章中，*行为测试*，你将学习更多关于端到端测试应用程序的方法和工具，这包括 REST API。然而，由于我们的 REST API 很简单，我们可以使用 Laravel 为我们提供的功能添加一些相当不错的测试。实际上，这个想法与我们在第八章中写的测试非常相似，*使用现有的 PHP 框架*，我们在某个端点发起请求，并期望得到响应。唯一的区别将是我们使用的断言类型（可以检查 JSON 响应是否正常），以及我们执行请求的方式。

让我们在与书籍相关的端点集合中添加一些测试。我们需要在数据库中有一些书籍以便查询，因此我们将在每个测试之前填充数据库，即使用`setUp`方法。记住，为了在测试后保持数据库干净，我们需要使用`DatabaseTransactions`特质。将以下代码添加到`tests/BooksTest.php`：

```php
<?php

use Illuminate\Foundation\Testing\DatabaseTransactions;
use App\Book;

class BooksTest extends TestCase {

    use DatabaseTransactions;

 private $books = [];

    public function setUp() {
        parent::setUp();

 $this->addBooks();
    }

    private function addBooks() {
        $this->books[0] = Book::create(
            [
                'isbn' => '293842983648273',
                'title' => 'Iliad',
                'author' => 'Homer',
                'stock' => 12,
                'price' => 7.40
            ]
        );
        $this->books[0]->save();
        $this->books[0] = $this->books[0]->fresh();

        $this->books[1] = Book::create(
            [
                'isbn' => '9879287342342',
                'title' => 'Odyssey',
                'author' => 'Homer',
                'stock' => 8,
                'price' => 10.60
            ]
        );
        $this->books[1]->save();
        $this->books[1] = $this->books[1]->fresh();

        $this->books[2] = Book::create(
            [
                'isbn' => '312312314235324',
                'title' => 'The Illuminati',
                'author' => 'Larry Burkett',
                'stock' => 22,
                'price' => 5.10
            ]
        );
        $this->books[2]->save();
        $this->books[2] = $this->books[2]->fresh();
    }
}
```

如您在前面的代码中看到的，我们向数据库中添加了三本书，同时也添加到了类属性 `$books` 中。当我们想要断言一个响应是有效的时，我们将需要这些书。此外，请注意 `fresh` 方法的使用；这个方法将我们拥有的模型与数据库中的内容同步。我们需要这样做是为了获取数据库中插入的 ID，因为我们事先并不知道它。

在我们运行每个测试之前，我们还需要做另一件事：验证我们的客户端。我们需要向访问令牌生成端点发送有效的凭据，并存储我们收到的访问令牌，以便可以在剩余的请求中使用它。您可以选择如何提供凭据，因为有不同的方法可以做到这一点。在我们的情况下，我们只是提供了一个已知存在于数据库中的客户端测试的凭据，但您可能更喜欢每次都将该客户端插入数据库。使用以下代码更新测试：

```php
<?php

use Illuminate\Foundation\Testing\DatabaseTransactions;
use App\Book;

class BooksTest extends TestCase {

    use DatabaseTransactions;

    private $books = [];
 private $accessToken;

    public function setUp() {
        parent::setUp();

        $this->addBooks();
 $this->authenticate();
    }

    //...

 private function authenticate() {
 $this->post(
 'oauth/access_token',
 [
 'client_id' => 'iTh4Mzl0EAPn90sK4EhAmVEXS',
 'client_secret' => 'PfoWM9yq4Bh6rhr8oDDsNZM',
 'grant_type' => 'client_credentials'
 ]
 );
 $response = json_decode(
 $this->response->getContent(), true
 );
 $this->accessToken = $response['access_token'];
 }
}
```

在前面的代码中，我们使用 `post` 方法来发送 POST 请求。此方法接受一个包含端点的字符串和一个包含要包含的参数的数组。在发出请求后，Laravel 将响应对象保存到 `$response` 属性中。我们可以对其进行 JSON 解码，并提取所需的访问令牌。

是时候添加一些测试了。让我们从一个简单的测试开始：根据 ID 请求一本书。ID 用于使用书籍的 ID 进行 GET 请求（不要忘记访问令牌），并检查响应是否与预期的匹配。记住，我们已经有 `$books` 数组了，所以执行这些检查将会非常简单。

我们将使用两个断言：`seeJson`，它比较接收到的 JSON 响应与我们提供的响应，以及您已经从之前的测试中了解到的 `assertResponseOk`——它只是检查响应是否有 200 状态码。将以下测试添加到类中：

```php
public function testGetBook() {
    $expectedResponse = [
        'book' => json_decode($this->books[1], true)
    ];
    $url = 'books/' . $this->books[1]->id
        . '?' . $this->getCredentials();

    $this->get($url)
        ->seeJson($expectedResponse)
        ->assertResponseOk();
}

private function getCredentials(): string {
    return 'grant_access=client_credentials&access_token='
        . $this->accessToken;
}
```

我们使用 `get` 方法而不是 `post`，因为这是一个 GET 请求。此外，请注意我们使用了 `getCredentials` 辅助函数，因为我们将在每个测试中使用它。为了查看另一个示例，让我们添加一个测试来检查请求包含给定标题的书籍时的响应：

```php
public function testGetBooksByTitle() {
    $expectedResponse = [
        'books' => [
            json_decode($this->books[0], true),
            json_decode($this->books[2], true)
        ]
    ];

    $url = 'books/?title=Il&' . $this->getCredentials();
    $this->get($url)
        ->seeJson($expectedResponse)
        ->assertResponseOk();
}
```

前面的测试几乎与之前的测试相同，不是吗？唯一的变化是端点和预期的响应。嗯，剩余的测试都将遵循相同的模式，因为到目前为止，我们只能获取书籍并对其进行筛选。

为了看到一些不同之处，让我们来看看如何测试创建资源的端点。有几种不同的选择，其中之一是首先发出请求，然后去数据库检查资源是否已创建。另一个选择，我们更倾向于使用的方法，是首先发送创建资源的请求，然后，使用响应中的信息，发送一个请求来获取新创建的资源。这是更可取的，因为我们只测试 REST API，我们不需要知道数据库使用的具体模式。此外，如果 REST API 更改其数据库，测试将保持通过——它们应该保持通过——因为我们只通过接口进行测试。

一个很好的例子可能是借一本书。测试应该首先发送一个 POST 请求来借书，指定书籍 ID，然后从响应中提取借出的书籍 ID，最后发送一个 GET 请求请求那本借出的书。为了节省时间，你可以将以下测试添加到现有的`tests/BooksTest.php`中：

```php
public function testBorrowBook() {
    $params = ['book-id' => $this->books[1]->id];
    $params = array_merge($params, $this->postCredentials());

    $this->post('borrowed-books', $params)
        ->seeJsonContains(['book_id' => $this->books[1]->id])
        ->assertResponseOk();

    $response = json_decode($this->response->getContent(), true);

    $url = 'borrowed-books' . '?' . $this->getCredentials();
    $this->get($url)
        ->seeJsonContains(['id' => $response['borrowed-book']['id']])
        ->assertResponseOk();
}

private function postCredentials(): array {
    return [
        'grant_access' => 'client_credentials',
        'access_token' => $this->accessToken
    ];
}
```

# 摘要

在本章中，你学习了 REST API 在 Web 世界中的重要性。现在你不仅能够使用它们，还能够编写自己的 REST API，这使得你成为一个更加多才多艺的开发者。你还可以将你的应用程序与第三方 API 集成，为用户提供更多功能，并使你的网站更加有趣和有用。

在下一章和最后一章中，我们将通过发现一种除单元测试之外的其他测试类型来结束这本书：行为测试，这可以提高你 Web 应用程序的质量和可靠性。
