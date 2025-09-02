# 12

# 使用 Drupal 构建 API

Drupal 10 自带了一些很棒的功能，可以帮助您使用核心序列化和 JSON:API 模块构建 RESTful API。这些功能使您能够构建无头和/或解耦的解决方案，同时仍然可以与 Drupal 交互并查询数据。

在本章中，我们将探讨以下内容：

+   使用 JSON:API 从 Drupal 获取数据 ([`jsonapi.org/`](https://jsonapi.org/))

+   使用 POST 和 JSON:API 创建数据

+   使用 PATCH 和 JSON:API 更新数据

+   使用 DELETE 和 JSON:API 删除数据

+   使用视图提供自定义数据源

+   使用 OAuth 方法

序列化模块提供了一种将数据序列化到或从 JSON 和 XML 等格式反序列化的方法。RESTful Web Services 模块随后通过 Web API 公开实体和其他资源类型。在 RESTful 资源端点执行的运算使用与在非 API 环境中相同的创建、编辑、删除和查看权限。JSON:API 模块在 API 路由上以 JSON 表示形式公开您的数据实体（节点、分类法、媒体和用户）。我们还将介绍如何处理 API 的自定义认证。 

# 技术要求

本章中的所有 API 都通过 HTTP 操作。无论请求是否成功，都会返回 HTTP 响应代码。如果您不熟悉 HTTP 响应代码或只是需要复习它们，可以在此处查看：[`developer.mozilla.org/en-US/docs/Web/HTTP/Status`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status)。当您使用 Drupal 中的 API 时，您将看到大量的 HTTP 响应代码，因此复习它们是个好主意。您可以在 GitHub 上找到本章中使用的完整代码：[`github.com/PacktPublishing/Drupal-10-Development-Cookbook/tree/main/chp12`](https://github.com/PacktPublishing/Drupal-10-Development-Cookbook/tree/main/chp12)

# 使用 JSON:API 从 Drupal 获取数据

只需几步点击，我们就可以打开并公开 Drupal 中的数据，以便外部服务消费。这将使我们能够请求 Drupal 中的数据，并以 JSON 格式返回，使其易于消费。

## 准备工作

首先，我们需要启用一些核心模块。前往**管理** | **扩展**并启用以下模块：

+   **HTTP** **基本认证**

+   **JSON:API**

+   **RESTful** **Web Services**

+   **序列化**：

![图 12.1 – 在 Drupal 中启用必要的模块](img/Figure_12.1_B18548.jpg)

图 12.1 – 在 Drupal 中启用必要的模块

JSON:API 模块使用序列化模块来处理响应的规范化以及从请求中数据的反规范化。端点支持特定的格式（JSON、XML 等），并且认证提供者支持在请求头中传递认证信息。

重要提示

注意，在撰写本文时，多语言支持仍在 JSON:API 下进行最终确定。您可以通过此处跟踪此问题的状态：[`www.drupal.org/project/drupal/issues/2794431`](https://www.drupal.org/project/drupal/issues/2794431)。

## 如何做到这一点……

您将在 Drupal 管理区域的 **配置** 部分中看到一个名为 **JSON:API** 的新区域：

![图 12.2 – JSON:API 为 Drupal 管理区域添加了一个新的配置部分](img/Figure_12.2_B18548.jpg)

图 12.2 – JSON:API 为 Drupal 管理区域添加了一个新的配置部分

JSON:API 默认提供的设置不多，除了两个安全设置之外。一个允许只有读取端点，而另一个允许在端点上执行 **创建、读取、更新、删除** （**CRUD**） 操作：

![图 12.3 – JSON:API 的配置选项](img/Figure_12.3_B18548.jpg)

![图 12.3 – JSON:API 的配置选项](img/Figure_12.3_B18548.jpg)

目前请保持默认设置。创建一个新的 `基本页面` 节点并保存。

然后，转到此 URL（将 `localhost` 替换为您的本地站点域名）：

```php
https://localhost/jsonapi/node/page
```

Drupal 将以 JSON:API 的形式响应您网站中所有 `基本页面` 节点的信息：

![图 12.4 – Drupal 的一个示例 JSON:API 响应](img/Figure_12.4_B18548.jpg)

![图 12.4 – Drupal 的一个示例 JSON:API 响应](img/Figure_12.4_B18548.jpg)

如果您想通过 ID 获取特定的节点，您可以在请求 URL 中使用 `filter` 关键字作为查询参数。

`filter` 的格式为 `filter[attribute]`。在此之后，我们可以通过在浏览器中请求此 URL 来获取 ID 为 1 的节点：

```php
https://localhost/jsonapi/node/page?filter[
    drupal_internal__nid]=1
```

您将得到与第一个示例相同的响应，但现在，响应中的 `data` 数组将只包含匹配 ID 1 的节点。这适用于您想要按任何属性筛选的任何属性。如果您知道实体的 UUID，您也可以直接通过将其用作参数来请求它：

```php
https://localhost/jsonapi/node/page/{ENTITY UUID}
```

通过将 UUID 作为 URL 参数传递，您将获得特定 `基本页面` 节点类型的节点信息。使用标题中的单词 `Test` 创建更多 `基本页面` 节点。假设您想按标题中包含单词 `Test` 的节点进行筛选；您可以像这样请求：

```php
https://localhost/jsonapi/node/page?filter[title][operator]
  =CONTAINS&filter[title][value]=Test&fields[node--page]
    =title
```

Drupal 只会返回标题中包含单词 “Test” 的节点。在这里，我们还添加了 `&fields[node—page]=title` 到请求中，以便更容易看到响应：

![图 12.5 – Drupal 的一个筛选后的 JSON:API 响应](img/Figure_12.5_B18548.jpg)

![图 12.5 – Drupal 的一个筛选后的 JSON:API 响应](img/Figure_12.5_B18548.jpg)

如果您想使用 `cURL` 而不是其他方式，可以这样做：

```php
curl \
  --header 'Accept: application/vnd.api+json' \
  --url "https://localhost/jsonapi/node/page?filter
    [title][operator]=CONTAINS&filter[title][value]=
      Test&fields[node--page]=title" \
--globoff
```

您可以从任何 HTTP API、任何语言或命令行工具（如 `cURL`）请求此内容，只要您的参数和选项正确。例如，请参阅使用 `fetch` ([`developer.mozilla.org/en-US/docs/Web/API/Fetch_API`](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API)) 的适当文档，以将其与解耦的 JavaScript 应用程序集成。

curl: (60) SSL 证书问题

如果您在 cURL 中遇到关于 SSL 证书问题的错误，那么很可能是您的 SSL 证书无法验证。这在本地自签名证书中很常见。为了在开发中绕过此错误，您可以在之前提到的 cURL 请求中传递`--insecure`标志。在生产中，请确保您有一个有效的正常工作的 SSL 证书 – 不要在实际应用中使用此标志。

JSON:API 还可以通过使用`include`来请求相关数据，将其包含在响应中。如果我们向查询参数添加`include`以及 JSON 项目关系部分下的任何关系名称，我们将在 JSON:API 响应中收到它们。

如果您想获取节点作者信息，例如，您可以像这样请求它：

```php
https://localhost/jsonapi/node/page?filter[title][operator]=CONTAINS&filter[title][value]=Test&fields[node—page]=title&include=uid
```

我们将在响应中收到以下信息：

![图 12.6 – 带有作者数据的示例 JSON:API 响应](img/Figure_12.6_B18548.jpg)

图 12.6 – 带有作者数据的示例 JSON:API 响应

注意，要查看用户数据，请求用户需要“查看用户信息”权限。否则，您将收到一个**访问拒绝**错误。

列在**关系**下的任何项目都可以请求 – 例如，作者数据、附加到返回实体的分类、附加到返回实体的媒体或其他类型的关联数据。

### 分页、过滤和排序请求

分页是通过附加一个页面查询参数来完成的。要限制请求为 10 个节点，我们必须使用`append ?page[limit]=10`。要访问下一组结果，我们还必须传递`page[offset]=10`。

以下是一个返回第一页和第二页结果的示例：

```php
https://localhost/jsonapi/node/article?page[limit]=10
https://localhost/jsonapi/node/article?page[offset]=10&page
  [limit]=10
```

每个请求都包含一个链接属性；当使用分页结果时，这还将包含下一页和上一页的链接。

过滤是通过附加一个过滤查询参数来完成的。以下是一个请求所有已提升到首页的节点的示例：

```php
https://localhost/jsonapi/node/article?filter[promoted]
  [path]=promote&filter[promoted][value]=1&filter[promoted]
    [operator]==
```

每个过滤器都由一个名称定义 – 在前面的示例中，它是提升。然后，过滤器获取`path`，即要过滤的字段。值和操作符决定了如何过滤。

排序是最简单的操作。在这里，添加了一个`sort`查询参数。字段名称值是要排序的字段；要按降序排序，必须在字段名称前添加一个减号。以下示例分别展示了如何按`nid`升序和降序排序：

```php
https://localhost/jsonapi/node/article?sort=nid
https://localhost/jsonapi/node/article?sort=-nid
```

## 它是如何工作的…

Drupal 中的序列化模块提供了将数据结构从 JSON 和 XML 等格式进行归一化和反归一化的必要工具。当对 Drupal 发出请求时，负责的路由获取资源并将其归一化为一个数组；然后，一个注册的**编码器**（在这种情况下，**JsonEncoder**）以请求的格式（JSON、XML 等）返回数据：

![图 12.7 – 高级查看 Drupal 中序列化工作方式](img/Figure_12.7_B18548.jpg)

图 12.7 – 高级查看 Drupal 中序列化工作方式

这是 Drupal 中数据归一化和序列化的基础。当您启用`JSON:API`模块时，会在`/jsonapi/`下的资源注册新路由。这些动态生成的路由会自动编码数据并以 JSON 格式返回。每次请求实体时，序列化和编码器负责构建响应数据。

## 还有更多...

### 安装 JSON:API 额外模块

JSON:API 额外模块提供了一个用户界面进行额外定制。JSON:API 额外模块应像所有其他模块一样添加到您的 Drupal 安装中 – 即，使用 Composer：

```php
composer require drupal/jsonapi_extras:³.0
```

一旦该模块在 Drupal 中安装，您将能够启用或禁用端点、更改资源名称、更改资源路径、禁用字段、提供别名字段名称以及增强 JSON:API 路由的字段输出。

### 更改 JSON:API 路径前缀

使用“额外模块”，可以将 API 路径前缀从`jsonapi`更改为`api`或其他任何前缀。

从管理工具栏，导航到**配置**。在**网络服务**部分下，点击**JSON:API 覆盖**来自定义 JSON:API 实现。**设置**选项卡允许您修改 API 路径前缀：

![图 12.8 – 使用 JSON:API 额外功能更改 JSON 路径前缀](img/Figure_12.9_B18548.jpg)

图 12.8 – 使用 JSON:API 额外功能更改 JSON 路径前缀

更改**路径前缀**后，请在 Drupal 中清除缓存以查看结果和新路径。

### 禁用和增强返回的实体字段

JSON:API 额外模块允许您覆盖 JSON:API 模块自动暴露的端点。这允许您禁用返回的字段。它还允许您使用增强器简化字段属性的架构。

从管理工具栏，转到**配置**。在**网络服务**部分下，点击**JSON:API 覆盖**来自定义 JSON:API 实现。

要禁用端点，请点击任何端点的**覆盖**。勾选**禁用**复选框以关闭该特定端点：

![图 12.9 – 使用 JSON:API 额外功能禁用资源，以便端点不可访问](img/Figure_12.10_B18548.jpg)

图 12.9 – 使用 JSON:API 额外功能禁用资源，以便端点不可访问

要禁用、别名或使用增强器，请点击`POST`/`PATCH`请求：

![图 12.10 – 您可以在 JSON:API 端点中更改字段别名并禁用字段](img/Figure_12.11_B18548.jpg)

图 12.10 – 您可以在 JSON:API 端点中更改字段别名并禁用字段

在这里，我们已经禁用了修订字段在 JSON:API 输出中显示。我们可以对创建和更改的字段应用增强器：

![图 12.11 – 使用增强器格式化 JSON:API 返回的数据](img/Figure_12.12_B18548.jpg)

图 12.11 – 使用增强器格式化 JSON:API 返回的数据

在这个例子中，创建和更改的字段将不再返回 Unix 时间戳，而是返回*RFC ISO* *8601*格式的时间戳。

回到并查看我们的 JSON:API 路由，我们可以看到修订字段已被删除，创建和更改的字段已格式化：

![图 12.12 – 由 JSON:API Extras 模块定制的 JSON:API 响应](img/Figure_12.13_B18548.jpg)

图 12.12 – 由 JSON:API Extras 模块定制的 JSON:API 响应

## 参见

JSON 和 XML 并不是 Drupal 作为响应返回数据的唯一格式。还有一些贡献的模块提供了其他数据格式，例如 PDF、CSV 和 XLS（Excel）序列化。如果您对它们的工作原理感到好奇，可以查看这些模块的源代码：

+   PDF 序列化：[`www.drupal.org/project/pdf_serialization`](https://www.drupal.org/project/pdf_serialization)

+   CSV 序列化：[`www.drupal.org/project/csv_serialization`](https://www.drupal.org/project/csv_serialization)

+   Excel 序列化序列化：[`www.drupal.org/project/xls_serialization`](https://www.drupal.org/project/xls_serialization)

# 使用 POST 通过 JSON:API 创建数据

返回数据的 API 很棒，但为了创建更功能化的解耦应用或集成外部服务，我们还需要能够将数据发送到 Drupal。阅读以下章节后，您将能够从远程源在 Drupal 10 应用程序中创建实体。

## 准备工作

我们将使用在 Drupal 10 标准配置文件中预安装的`Article`内容类型。如果您没有`Article`内容类型，请创建一个并添加一个基本字段，例如`Body`。

## 如何做到这一点…

首先，我们需要告诉 Drupal 允许对 JSON:API 进行 CRUD 操作。返回到 Drupal 管理中**配置**部分的 JSON:API 设置页面，并启用**接受所有 JSON:API 创建、读取、更新和** **删除操作**。：

![图 12.13 – 启用 JSON:API 以允许更多操作](img/Figure_12.14_B18548.jpg)

图 12.13 – 启用 JSON:API 以允许更多操作

如果您不启用此功能，您将无法执行除从 JSON:API 端点读取数据之外的操作。

接下来，创建一个只有创建`Article`节点所需权限的用户账户。这是我们将在创建新节点请求中使用该账户。出于安全原因，您不应在 API 中使用管理员账户。

在 Drupal 中创建 API 的每个请求中，我们都需要在请求被接受之前进行身份验证。

打开终端（命令行）。与上一节不同，我们无法在浏览器中直接执行这些请求。为此，我们将使用`curl`，它在许多操作系统上都是预安装的：

```php
curl \
    --user chapter12:chapter12 \
    --header 'Accept: application/vnd.api+json' \
    --header 'Content-type: application/vnd.api+json' \
    --request POST https://localhost/jsonapi/node/article \
    --data-binary '{
  "data": {
    "type": "node--article",
    "attributes": {
      "title": "New article created by chapter12",
      "body": {
        "value": "This node was created using JSON:API!",
        "format": "plain_text"
      }
    }
  }
}'
```

再次提醒，记得将`localhost`替换为你的开发站点的正确主机名。我们创建了一个名为`chapter12`的用户，密码为`chapter12`，并将其设置为`内容编辑`角色（包含在标准安装配置文件中），该角色具有创建和编辑节点的基本权限。此用户通过`--user`参数传递。

响应将返回成功或失败的授权响应。如果成功，你将看到一个大的 JSON 响应，反映新创建的节点，你将在 Drupal 中看到新节点：

![图 12.14 – 通过 POST 请求创建的节点](img/Figure_12.15_B18548.jpg)

图 12.14 – 通过 POST 请求创建的节点

我们还可以看到，它被正确地添加到了正文字段中，以及我们在请求中指定的文本格式：

![图 12.15 – 验证我们通过 POST 请求传递的内容是否正确创建](img/Figure_12.16_B18548.jpg)

图 12.15 – 验证我们通过 POST 请求传递的内容是否正确创建

如果我们在浏览器中查看我们的 JSON:API 路由，我们也会在那里看到这个新节点：

![图 12.16 – 使用 HTTP 请求访问新创建的节点](img/Figure_12.17_B18548.jpg)

图 12.16 – 使用 HTTP 请求访问新创建的节点

## 它是如何工作的…

在这种情况下，本例中的 JSON:API 模块在身份验证或实体访问方面没有做任何额外的工作。请求的处理方式与`chapter12`用户来到网站，登录，并在管理界面中创建节点没有区别。

请求类型是`POST`请求。这与第一部分中用于读取数据的`GET`请求不同。发送数据到服务器需要使用`POST`请求。你可以在 MDN 上了解更多关于`POST`请求（s）的信息：[`developer.mozilla.org/en-US/docs/Web/HTTP/Methods/POST`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods/POST)。

这是一个强大的抽象，可以帮助你构建 API 和解耦的应用程序，因为它需要有效的用户身份验证，但不能执行超出该用户/用户角色允许范围的任何操作。这意味着如果我们移除内容编辑角色的“创建文章节点”权限，这个`POST`请求将因为`403`禁止响应而失败。因此，在现实世界中创建一个具有有限权限的`API 用户`角色是一个好主意，你可以随时打开或关闭它。

`永远`不要使用超级用户（`user 1`）或具有管理员角色的用户进行 API 请求。如果凭证泄露，恶意行为者可以执行他们想要的任何请求。在阅读本章后，回顾 Drupal.org 上使用 API 时的安全考虑因素会是一个好主意：[`www.drupal.org/docs/core-modules-and-themes/core-modules/jsonapi-module/security-considerations`](https://www.drupal.org/docs/core-modules-and-themes/core-modules/jsonapi-module/security-considerations)。

在`curl`请求中，当我们传递`--user`参数时，它将自动为请求创建一个授权头。此头传递了一个基本认证，Drupal 中的 Basic Auth 模块会拦截并使用它来验证请求。

或者，你可以使用命令行手动获取基本认证令牌；例如：

```php
echo -n "chapter12:chapter12" | base64
```

这将返回`Y2hhcHRlcjEyOmNoYXB0ZXIxMg==`。然后，你可以将其作为标题而不是使用`–user`参数传递：

```php
curl \
    --header 'Accept: application/vnd.api+json' \
    --header 'Content-type: application/vnd.api+json' \
    --header 'Authorization:Basic Y2hhcHRlcjEyOmNoYXB0ZX
      IxMg=='Y2hhcHRlcjEyOmNoYXB0ZXIxMg==' \
  … rest of request
```

## 参见

虽然**cURL**功能强大，但它是一个低级工具，对于更长时间、更大的请求，从命令行使用它可能会很困难。你可以安装 Google 的 Postman 工具来简化与 API 的工作。你可以从[`www.postman.com/`](https://www.postman.com/)下载它。

# 使用 PATCH 通过 JSON:API 更新数据

现在我们已经知道了如何在 Drupal 中创建一些数据，让我们学习如何更新现有数据。

## 如何做到这一点...

我们的要求将与前几节中的`POST`请求类似，但有一些变化。

首先，为了在 Drupal 中更新一个实体，你需要将实体的 UUID 传递到请求负载中，这样它才能正常工作。这与数字实体 ID 不同。你可以从 JSON:API 获取数据来获取 UUID，正如我们在*使用 JSON:API 从 Drupal 获取数据*部分所看到的。对于我们通过`POST`请求创建的节点，UUID 值是`1ddf244d-e8e6-40f5-be48-23bc8fa0fa3e`。

让我们继续更改我们之前创建的文章节点的标题和正文。这次，我们必须在 URL 以及请求体中传递 UUID：

```php
   curl \
    --user chapter12:chapter12 \
    --header 'Accept: application/vnd.api+json' \
    --header 'Content-type: application/vnd.api+json' \
    --request PATCH 'https://localhost/jsonapi/node/
      article/1ddf244d-e8e6-40f5-be48-23bc8fa0fa3e' \
    --data-binary '{
  "data": {
    "type": "node--article",
    "id": "1ddf244d-e8e6-40f5-be48-23bc8fa0fa3e",
    "attributes": {
      "title": "Article updated by chapter12",
      "body": {
        "value": "This node was updated using JSON:API!",
        "format": "plain_text"
      }
    }
  }
}'
```

注意，`id`属性位于`attributes`数据之外。如果你尝试将其放在`attributes`下，你会得到一个`HTTP 422 Unprocessable Content`错误。如果你传递无效的属性，你会收到一个`500`错误。你必须更新的内容必须与数据模型匹配才能成功。

提交此请求后，我们将在命令行以 JSON 格式收到更新后的节点，我们将在 Drupal 中看到我们的更新已被应用：

![图 12.17 – 使用 PATCH 请求更新节点的审查](img/Figure_12.18_B18548.jpg)

图 12.17 – 使用 PATCH 请求更新节点的审查

如果我们想要将节点归因于另一个用户怎么办？使用一个仅用于处理 API 操作的账户是个好主意，但我们当然不希望所有内容都归因于该用户。

例如，假设你有一个外部发布平台，用户正在提交内容。当内容发布时，该系统会向 Drupal 发送一个`POST`请求，以便创建内容。让我们将内容归因于一个名为`Johnny Editor`的 Drupal 用户，其 UUID 为`c1ce9fe6-4eea-4f69-92c2-883415019002`。像节点一样，你可以在`/jsonapi/user/user`查看用户实体数据。

我们可以使用`PATCH`请求来修复现有内容，并将作者从`chapter12`用户更改为`Johnny Editor`用户，通过在请求体中传递`relationships`数据来实现。不过，在我们能够这样做之前，我们的 API 用户角色需要`PATCH`数据。现在，请启用这个权限。

这是因为，在这个例子中，我们请求更改节点的所有权从一位用户到另一位用户，Drupal 在执行操作之前会明确检查这一点。

审查你的权限

再次强调，一旦你阅读了这一章，就很好去 Drupal.org 上回顾一下使用 API 时的安全考虑：[`www.drupal.org/docs/core-modules-and-themes/core-modules/jsonapi-module/security-considerations`](https://www.drupal.org/docs/core-modules-and-themes/core-modules/jsonapi-module/security-considerations)。

现在，我们可以发出新的`PATCH`请求：

```php
   curl \
    --user chapter12:chapter12 \
    --header 'Accept: application/vnd.api+json' \
    --header 'Content-type: application/vnd.api+json' \
    --request PATCH 'https://localhost/jsonapi/node/
      article/1ddf244d-e8e6-40f5-be48-23bc8fa0fa3e' \
    --data-binary '{
  "data": {
    "type": "node--article",
    "id": "1ddf244d-e8e6-40f5-be48-23bc8fa0fa3e",
    "attributes": {
      "title": "Using Drupal 10 PATCH & JSON:API by Johnny 
        Editor",
      "body": {
        "value": "This is how you use Drupal 10 PATCH & 
          JSON:API.",
        "format": "plain_text"
      }
    },
    "relationships": {
      "uid": {
        "data": {
          "type": "user--user",
          "id": "c1ce9fe6-4eea-4f69-92c2-883415019002"
        }
      }
    }
  }
}'
```

在这里，我们收到了 Drupal 的成功响应，以及节点的 JSON 表示。我们可以看到内容已更新，节点所有者已被重新分配给**Johnny Editor**：

![图 12.18 – 使用 PATCH 请求重新分配节点所有权](img/Figure_12.19_B18548.jpg)

图 12.18 – 使用 PATCH 请求重新分配节点所有权

你可以使用关系添加任何你想要添加到实体的实体引用。使用这个相同的公式，我们可以向这篇文章添加一些分类。假设有两个术语在`taxonomy`标签中，分别称为`Technology`和`News`。我们想在文章中标记它们。

就像节点和用户一样，你可以通过导航到`/jsonapi/taxonomy_term/tags`来查看分类术语数据：

![图 12.19 – 使用其 JSON:API 端点读取分类术语](img/Figure_12.20_B18548.jpg)

图 12.19 – 使用其 JSON:API 端点读取分类术语

使用这里的 UUID，我们可以在请求的`relationships`部分传递它们：

```php
   curl \
    --user chapter12:chapter12 \
    --header 'Accept: application/vnd.api+json' \
    --header 'Content-type: application/vnd.api+json' \
    --request PATCH 'https://localhost/jsonapi/node/
      article/1ddf244d-e8e6-40f5-be48-23bc8fa0fa3e' \
    --data-binary '{
  "data": {
    "type": "node--article",
    "id": "1ddf244d-e8e6-40f5-be48-23bc8fa0fa3e",
    "attributes": {
      "title": "Using Drupal 10 PATCH & JSON:API by Johnny 
        Editor",
      "body": {
        "value": "This is how you use Drupal 10 PATCH & 
          JSON:API.",
        "format": "plain_text"
      }
    },
    "relationships": {
      "field_tags": {
        "data": [
          {
            "type": "taxonomy_term--tags",
            "id": "4ef201ed-7cb6-49e5-b125-4c2709be1a42"
          },
          {
            "type": "taxonomy_term--tags",
            "id": "09504010-8eff-4be0-8205-607f9e74ffa1"
          }
       ]
      }
    }
  }
}'
```

提交后，我们将看到文章节点已成功更新为指定的两个分类术语：

![图 12.20 – 通过 PATCH 请求向节点添加分类标签](img/Figure_12.21_B18548.jpg)

图 12.20 – 通过 PATCH 请求向节点添加分类标签

现在，如果我们像第一部分那样从 Drupal 获取数据，我们将在节点数据中看到作者和分类关系：

![图 12.21 – 节点响应中的分类关系](img/Figure_12.22_B18548.jpg)

图 12.21 – 节点响应中的分类关系

当你将此与 URL 查询字符串中的 `include` 结合使用时，你将获得作者数据以及分类数据：`include=uid,field_tags`。

如果你想要删除引用项，你可以提交一个没有该项目的 `PATCH` 请求。如果我们想要删除一个术语，我们只需传递我们想要保留的术语：

```php
"relationships": {
      "field_tags": {
        "data": [
          {
            "type": "taxonomy_term--tags",
            "id": "4ef201ed-7cb6-49e5-b125-4c2709be1a42"
          },
       ]
      }
    }
```

要完全清空 `tags` 字段，传递一个空值：

```php
"relationships": {
      "field_tags": {
        "data": {}
      }
    }
```

你将看到术语已被从文章节点中删除：

![图 12.22 – 在 PATCH 请求中传递空值将删除关系](img/Figure_12.23_B18548.jpg)

图 12.22 – 在 PATCH 请求中传递空值将删除关系

## 它是如何工作的…

就像 `POST` 和 `PATCH` 一样，`PATCH` 提供了一种强大的方式，可以从解耦的应用程序或远程系统更新 Drupal 中的实体数据。当你理解你的实体数据和模式时，你可以通过 API 以任何你想要的方式创建和操作它。

同样地，`POST` 和 `PATCH` 请求必须遵守 Drupal 的权限系统。相同的规则被尊重，就像这是一个真实用户登录到 Drupal 更新内容一样。

使用这两种方法，你可以创建一个解耦的网站，可以从用户那里收集数据并将其发回 Drupal，例如自助服务亭、调查或评论系统。可能性是无限的。

# 使用 DELETE 命令通过 JSON:API 删除数据

最后，我们来到了 CRUD 简称的最终动作：`delete` 方法。此方法从 Drupal 中删除请求的数据，前提是你有 UUID。

## 如何操作…

与 `PATCH` 类似，发出 `DELETE` 需要在 Drupal 中具有适当的权限。在这种情况下，分配给我们的 API 用户的角色需要允许删除文章节点。返回到 Drupal 管理区域的权限部分，并将 **Article: Delete any content** 授予该角色。

我们需要将`delete any`分配而不是`delete own`，因为我们已经将节点所有权分配给了另一个用户。

所有的 `DELETE` 请求需要的只是我们希望删除的实体的 UUID。使用我们创建的节点，它的 UUID 是 `1ddf244d-e8e6-40f5-be48-23bc8fa0fa3e`。`DELETE` 请求看起来是这样的：

```php
   curl \
    --user chapter12:chapter12 \
    --header 'Accept: application/vnd.api+json' \
    --header 'Content-type: application/vnd.api+json' \
    --request DELETE 'https://localhost/jsonapi/node/
      article/1ddf244d-e8e6-40f5-be48-23bc8fa0fa3e'
```

当你这次提交时，命令行上不会有任何响应输出。当成功时，响应将是一个 `HTTP 204` 空响应。

JSON:API 路由现在显示没有节点，这意味着 `DELETE` 操作成功：

![图 12.23 – 使用 DELETE 请求后，节点已成功删除](img/Figure_12.24_B18548.jpg)

图 12.23 – 使用 DELETE 请求后，节点已成功删除

## 它是如何工作的…

就像 `POST` 和 `PATCH` 一样，`DELETE` 代表指定的用户执行操作。由于此用户有权限 `delete any` 文章内容，请求被允许，Drupal 从网站上删除了文章节点。

对于某些用例，`DELETE`可能对执行 API 或授予权限过于破坏性。在这种情况下，您可以使用`PATCH`并将实体状态更改为`未发布`或`存档`（如果使用`内容**审阅**模块）。

# 使用视图提供自定义数据源

RESTful Web Services 模块提供了视图插件，允许您通过视图公开 RESTful API 的数据。这允许您创建一个具有路径并使用序列化插件输出数据的视图。您可以使用此功能以 JSON 或 XML 格式输出实体，并且可以带有适当的头信息发送。

在本菜谱中，我们将创建一个视图，输出 Drupal 站点的用户，提供他们的用户名、电子邮件和图片（如果提供）。

## 准备工作

确保您已启用以下核心模块：

+   视图

+   视图 UI

+   RESTful Web Services

## 如何操作...

1.  导航到**结构**然后**视图**。

1.  点击**添加新视图**。

1.  将视图命名为`API Users`并显示`/api/users`：

![图 12.24 – 为自定义 REST 端点设置视图的路径](img/Figure_12.25_B18548.jpg)

图 12.24 – 为自定义 REST 端点设置视图的路径

1.  保存视图并继续。

在创建视图后，进行以下更改：

+   将行插件格式从**实体**更改为**字段**，以便我们可以控制特定的输出。

+   在格式设置部分的**设置**部分，在**接受**的**请求格式**下启用**json**。

+   确保您的视图具有**名称**、**电子邮件**和**图片**作为用户实体字段

+   将**用户：名称**字段更改为**纯文本**格式化程序。不要将其链接到用户，以确保响应不包含任何 HTML。

+   将**用户：图片**字段更改为使用 URL 到图片格式化程序，以便只返回 URL 而不是 HTML。

+   保存更新后的视图。

+   通过访问 `/api/users` 来访问您的视图。您将收到一个包含用户信息的 JSON 响应。

在您的浏览器中，您将看到以我们在视图中设置的方式格式化的系统用户输出：

```php
[
  {
    "name": "spuvest",
    "mail": "spuvest@example.com", 
    "user_picture": "\/sites\/default\/files\/pictures\/
      2017-07\/generateImage_xIQkfx.jpg"
  },
  {
    "name": "crepathuslus",
    "mail": "crepathuslus@example.com", 
    "user_picture": "\/sites\/default\/files\/pictures\/
      2017-07\/generateImage_eauTko.gif"
  },
  {
    "name": "veradabufrup",
    "mail": "veradabufrup@example.com", 
    "user_picture": "\/sites\/default\/files\/pictures\/
      2017-07\/generateImage_HsEjKW.png"
  }
]
```

## 它是如何工作的...

RESTful Web Services 模块提供了一个显示、行和格式插件，允许您以序列化格式导出内容实体。REST Export 显示插件允许您指定路径以访问 RESTful 端点，并为请求的格式正确分配`Content-Type`头。

序列化风格作为唯一支持的样式插件提供，用于 REST Export 显示。此样式插件仅支持标识为数据显示类型的行插件。它期望从行插件接收原始数据，以便可以将其传递给适当的序列化器。

然后，您可以选择使用数据实体或数据字段行插件。它们不是从它们的渲染方法返回渲染数组，而是返回将被序列化为正确格式的原始数据。通过行插件返回原始格式数据，并由样式插件进行序列化，显示插件将返回响应，通过序列化模块转换为正确的格式，这是我们之前在本章中看到的。

使用视图是提供 API 读取路由的绝佳方式。您将获得在 Drupal 中使用的视图 UI 的所有便利性，以及 JSON 或 XML 序列化的综合力量。您可以根据需要创建尽可能多的这些视图来满足不同的用例。

## 更多内容...

视图允许您提供特定的 RESTful 端点。使用视图，您可以添加具有自定义路径的显示，并将字段添加到其中，创建自定义 JSON 响应。这使得在 Drupal 中快速开发自定义只读数据 API 变得简单。

### 控制 JSON 输出中的键名

数据字段行插件允许您配置字段别名。当数据返回到视图时，它将包含 Drupal 的机器名称。这意味着自定义字段看起来可能像`field_my_field`，这可能对消费者来说没有意义。通过点击**字段**旁边的**设置**，您可以在模态表单中设置别名：

![图 12.25 – 您可以在视图 REST 显示中别名字段名称](img/Figure_12.26_B18548.jpg)

图 12.25 – 您可以在视图 REST 显示中别名字段名称

当您提供一个别名时，字段将匹配。例如，`user_picture`可以改为`avatar`，而`mail`键可以改为`email`：

```php
[
  {
    "name": "spuvest",
    "email": "spuvest@example.com", 
    "avatar": "\/sites\/default\/files\/pictures\/2017-07\/ 
      generateImage_xIQkfx.jpg"
  },
]
```

### 控制 RESTful 视图的访问

当您使用视图创建 RESTful 端点时，您不是使用 RESTful Web Services 模块创建的相同权限。您需要在视图中定义路由权限，这样您就可以指定请求的特定角色或权限。

`EntityResource`插件提供的默认`GET`方法不提供列出实体的方式，并允许通过 ID 检索任何实体。使用视图，您可以提供实体列表，限制它们到特定的捆绑包，等等。

使用视图，您甚至可以提供一个新端点来检索特定实体。使用上下文过滤器，您可以添加路由参数和过滤器来限制和验证实体 ID。例如，您可能希望通过 API 公开文章内容，但不公开页面。

# 使用 OAuth 方法

使用 RESTful Web Services 模块，我们可以为端点定义特定的支持认证提供者。Drupal 核心提供了一个 cookie 提供者，它通过有效的 cookie 进行认证，例如您的常规登录体验。然后，还有 HTTP Basic Authentication 模块来支持 HTTP 认证头。

一些替代方案提供了更健壮的认证方法。使用基于 cookie 的认证，你需要使用 CSRF 令牌来防止未经授权的第三方加载未请求的页面。当你使用 HTTP 认证时，你会在请求头中发送每个请求的密码。

一个流行且开放的授权框架是 OAuth。OAuth 是一种使用令牌而不是密码的正确认证方法。在这个菜谱中，我们将实现`Simple OAuth`模块，为`GET`和`POST`请求提供 OAuth 2.0 认证。

## 准备工作

如果你不太熟悉 OAuth 或 OAuth 2.0，它是一个授权标准。OAuth 的实现围绕在 HTTP 头中发送的令牌的使用。有关更多信息，请参阅 OAuth 主页：[`oauth.net/`](http://oauth.net/)。

对于以下部分，你需要贡献的**Rest UI**模块。Rest UI 将使你更容易在 Drupal 中查看和控制基于 REST 的路由。

## 如何做到这一点...

继续使用 Composer 获取必要的模块：

```php
composer require drupal/restui:¹.0
```

在**扩展**下的管理区域启用**REST UI**模块。

现在，执行以下操作：

1.  首先，我们必须将`Simple OAuth`模块添加到我们的 Drupal 站点中：

    ```php
    composer require drupal/simple_oauth:⁶.0@beta
    ```

1.  前往**配置**，然后在**网络服务**下点击**REST**以配置可用的端点：

![图 12.26 – 使用 REST UI 模块，你可以在管理中管理 RESTful 端点](img/Figure_12.27_B18548.jpg)

图 12.26 – 使用 REST UI 模块，你可以在管理中管理 RESTful 端点

1.  现在端点已经启用，必须进行配置。检查`GET`和`POST`请求。然后，勾选**JSON**复选框，以便数据可以以 JSON 格式返回。勾选**oauth2**复选框，然后保存。

1.  在我们能够配置`Simple OAuth`模块之前，我们必须生成一对密钥来加密`OAuth`令牌。我们可以通过两种方式之一来完成这项工作：

    1.  在 webroot 之外创建一个目录，然后点击**生成密钥**。在出现的对话框中，添加服务器上此文件夹的完整路径，然后点击**生成**。

    1.  如果管理 UI 不起作用，我们可以使用以下两个命令来生成密钥。将它们放在 webroot 之外的服务器上：

    ```php
    openssl genrsa -out private.key 2048
    ```

    ```php
    openssl rsa -in private.key -pubout > public.key
    ```

1.  生成密钥后，转到**配置**页面，然后转到**Simple OAuth**。输入刚刚生成的私钥和公钥的路径，然后点击**保存配置**：

![图 12.27 – 在管理区域中管理 Simple OAuth 的密钥](img/Figure_12.28_B18548.jpg)

图 12.27 – 在管理区域中管理 Simple OAuth 的密钥

1.  从**Simple OAuth 设置**配置表单中，点击**+** **添加客户端**。为客户端提供一个标签，并选择**管理员**范围。然后，点击**保存**以创建客户端。

1.  接下来，我们将通过`/oauth/`令牌端点生成一个令牌。你需要使用你刚刚创建的客户端的 ID。我们必须传递`grant_type`、`client_id`以及用户名和密码。`grant_type`是密码，而`client_id`是从创建的客户端中获取的 ID。用户名和密码将是你要使用的账户：

    ```php
    curl -X POST https://localhost/oauth/token \
    ```

    ```php
      -H 'content-type: application/x-www-form-
    ```

    ```php
        urlencoded' \
    ```

    ```php
      -d 'grant_type=client_credentials&client_id=
    ```

    ```php
        CLIENT_ID&username=chapter12&client_secret=
    ```

    ```php
          chapter12'
    ```

1.  响应将包含一个`access_token`属性。这将在进行 API 请求时用作你的令牌。

1.  使用**Authorization: Bearer [****token]**头请求 REST 节点：

    ```php
    curl -X GET 'https://localhost/node/8?_format=json' \
    ```

    ```php
     -H 'accept: application/json' \
    ```

    ```php
     -H 'authorization: Bearer ACCESS_TOKEN'
    ```

就这样！你现在可以使用 OAuth 设置认证路由，用于 Drupal 中的各种资源，并配置所有不同的 REST 端点，包括它们接受的方法和认证类型。

## 它是如何工作的…

`Simple OAuth`模块是使用社区标准库`League\OAuth2` PHP 库构建的，这是一个 OAuth2 实现的社区标准库。

在典型的认证请求中，存在一个认证管理器，该管理器使用`authentication_collector`服务来收集所有标记的认证提供者服务器。根据提供者设置的优先级，每个服务被调用以检查它是否适用于当前请求。然后，每个应用的认证提供者被调用以查看认证是否无效。

对于 RESTful Web Services 模块，这个过程更为明确。端点的`supported_auth`定义中标识的提供者是唯一通过`applies`和`authenticates`过程运行的服务。
