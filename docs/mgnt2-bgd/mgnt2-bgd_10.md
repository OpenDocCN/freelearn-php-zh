# 第九章. Web API

在前面的章节中，我们学习了如何使用一些后端组件，以便店主可以管理和操作数据，如客户、产品、类别、订单等。有时这还不够，比如当我们从第三方系统中拉取或推送数据时。在这些情况下，Magento Web API 框架使得通过 REST 或 SOAP 调用 Magento 服务变得容易。

在本章中，我们将涵盖以下主题：

+   用户类型

+   认证方法

+   REST 与 SOAP

+   基于令牌的认证实践

+   基于 OAuth 的认证实践

+   基于 OAuth 的 Web API 调用

+   基于会话的认证实践

+   创建自定义 Web API

+   列表过滤的搜索条件接口

在我们开始进行 Web API 调用之前，我们必须验证我们的身份并拥有访问 API 资源的必要权限（授权）。认证允许 Magento 识别调用者的用户类型。根据用户的（管理员、集成、客户或访客）访问权限，确定 API 调用资源的可访问性。

# 用户类型

我们可以访问的资源列表取决于我们的用户类型，并在我们的模块配置文件`webapi.xml`中定义。

API 已知有三种用户类型，如下所示：

+   **管理员或集成者**：管理员或集成者被授权的资源。例如，如果管理员被授权访问`Magento_Cms::page resource`，他们可以发起`POST /V1/cmsPage`调用。

+   **客户**：客户被授权的资源。这些资源具有匿名或自我权限。

+   **访客用户**：访客被授权的资源。这些资源具有匿名权限。

两个文件在定义 API 方面起着至关重要的作用：我们的模块`acl.xml`和`webapi.xml`文件。

`acl.xml`是我们定义我们的模块**访问控制列表**（**ACL**）的地方。它定义了访问资源的可用权限集。所有 Magento 模块的`acl.xml`文件被合并以构建一个 ACL 树，该树用于选择允许的 admin 角色资源或第三方集成的访问（**系统** | **扩展** | **集成** | **添加新集成** | **可用 API**）。

`webapi.xml`是我们定义 Web API 资源和它们权限的地方。当我们创建`webapi.xml`时，`acl.xml`中定义的权限被引用以创建每个 API 资源的访问权限。

让我们看看以下（截断的）来自核心`Magento_Cms`模块的`webapi.xml`：

```php
<routes  xsi:noNamespaceSchemaLocation= "urn:magento:module:Magento_Webapi:etc/webapi.xsd">
    ...
    <route url="/V1/cmsPage" method="POST">
        <service class="Magento\Cms\Api\PageRepositoryInterface" method="save" />
        <resources>
            <resource ref="Magento_Cms::page" />
        </resources>
    </route>
    ...
    <route url="/V1/cmsBlock/search" method="GET">
        <service class="Magento\Cms\Api\BlockRepositoryInterface" method="getList" />
        <resources>
            <resource ref="Magento_Cms::block" />
        </resources>
    </route>
    ...
</routes>
```

在 CMS 页面 API 的前述 `webapi.xml` 文件中，只有拥有 `Magento_Cms::page` 授权的用户可以访问 `POST /V1/cmsPage` 或 `GET /V1/cmsBlock/search`。我们将在示例中稍后更详细地解释路由；目前，我们的重点是 `resource`。我们可以在资源下分配多个子 `resource` 元素。在这些情况下，用户只要有任何一个这些 ACL 被分配，就能进行 API 调用。

实际的授权是授予给管理员或集成商，他们在 Magento 管理员中定义，可以选择完整的组或 ACL 树中选定的特定资源，如下面的截图所示：

![用户类型](img/00061.jpeg)

由于 `webapi.xml` 和 `acl.xml` 相互关联，让我们看看核心模块 `Magento_Cms` 的（截断的）`acl.xml` 文件：

```php
<resources>
    <resource id="Magento_Backend::admin">
        <resource id="Magento_Backend::content">
            <resource id="Magento_Backend::content_elements">
                <resource id="Magento_Cms::page" ...>
                    ...
                </resource>
            </resource>
        </resource>
    </resource>
</resources>
```

注意到 `Magento_Cms::page` 资源的位置嵌套在 `Magento_Backend::content_elements` 之下，而 `Magento_Backend::content_elements` 又嵌套在 `Magento_Backend::content` 之下，`Magento_Backend::content` 又嵌套在 `Magento_Backend::admin` 之下。这告诉 Magento 在显示 **角色资源** 树时，在 Magento 管理员下渲染 ACL 的位置，如前一个截图所示。这并不意味着如果用户被授权访问所有这些父 `Magento_Backend` 资源，他仍然无法访问 `Magento_Cms::page` 资源所对应的 API。

对资源进行授权是一种相对简单的事情。在授权时没有树形检查。因此，当在 `acl.xml` 下定义时，每个资源都需要在 `resource` 元素上有一个唯一的 `id` 属性值。

正如之前所列出的，这些定义的资源是管理员或集成商被授权访问的资源。

相反，客户被分配了一个名为 `anonymous` 或 `self` 的资源。如果我们对所有 Magento 核心模块进行完整的 `<resource ref="anonymous" />` 字符串搜索，会出现几个匹配项。

让我们来看看核心模块 `vendor/magento/module-catalog/etc/webapi.xml` 文件（以下为截断内容）：

```php
<route url="/V1/products" method="GET">
    <service class= "Magento\Catalog\Api\ProductRepositoryInterface" method="getList"/>
    <resources>
        <resource ref="anonymous" />
    </resources>
</route>
```

上述 XML 定义了一个 API 端点路径，其值为 `/V1/products`，可以通过 **HTTP GET** 方法访问。它进一步定义了一个名为 `anonymous` 的资源，这意味着当前登录的客户或访客用户可以调用此 API 端点。

`anonymous` 是一种特殊权限，在 `acl.xml` 中不需要定义。因此，它不会在 Magento 管理员下的权限树中显示。这仅仅意味着当前在 `webapi.xml` 中的资源可以无需认证即可访问。

最后，我们来看看 `self` 资源，其示例可以在（截断的）`vendor/magento/module-customer/etc/webapi.xml` 文件中找到如下：

```php
<route url="/V1/customers/me" method="PUT">
    <service class= "Magento\Customer\Api\CustomerRepositoryInterface" method="save"/>
    <resources>
        <resource ref="self"/>
    </resources>
    <data>
        <parameter name="customer.id" force="true">%customer_id%</parameter>
    </data>
</route>
```

`self`是一种特殊的访问方式，允许用户访问他们拥有的资源，前提是我们已经与系统建立了认证会话。例如，`GET /V1/customers/me`获取已登录客户的详细信息。这对于基于 JavaScript 的组件/小部件通常很有用。

# 认证方法

根据 Magento 的视角，移动应用、第三方应用和 JavaScript 组件/小部件（店面或管理员）是三种主要的客户端类型。尽管客户端基本上是与我们的 API 进行通信的一切，但每种类型的客户端都有一个首选的认证方法。

Magento 支持三种类型的认证方法，如下列出：

+   基于令牌的认证

+   基于 OAuth 的认证

+   基于会话的认证

**基于令牌的认证**最适合移动应用，其中令牌就像一个电子钥匙，提供对 Web API 的访问。基于令牌的认证系统背后的概念相对简单。用户在初始认证期间提供用户名和密码，以便从系统中获取一个时间有限的令牌。如果成功获取令牌，所有后续的 API 调用都使用该令牌进行。

**基于 OAuth 的认证**适用于与 Magento 集成的第三方应用。一旦应用通过**OAuth 1.0a 握手过程**获得授权，它就能访问 Magento Web API。在此我们必须了解三个关键术语：*用户*（资源所有者）、*客户端*（消费者）和*服务器*（服务提供商）。用户或资源所有者是请求允许访问其受保护资源的人。想象一下，一个客户作为用户（资源所有者）允许第三方应用访问其订单。在这种情况下，这个第三方应用就是客户端（消费者），而 Magento 及其 Web API 则是服务器（服务提供商）。

**基于会话的认证**可能是最容易理解的一种。作为客户，您使用客户凭证登录到 Magento 店面。作为管理员，您使用管理员凭证登录到 Magento 管理员界面。Magento Web API 框架使用您的登录会话信息来验证您的身份并授权访问请求的资源。

# REST 与 SOAP 的比较

Magento 支持 Web API 的两种通信类型：**SOAP**（即**简单对象访问协议**）和**REST**（即**表征状态转移**）。认证方法本身并不绑定于任何一种。我们可以使用相同的认证方法和 Web API 方法调用，无论是 SOAP 还是 REST。

我们可能如下概述一些 REST 的特定内容：

+   我们通过 cURL 命令或 REST 客户端运行 REST Web API 调用。

+   请求支持`HTTP`动词：`GET`、`POST`、`PUT`或`DELETE`。

+   一个 `HTTP` 标头需要一个授权参数，指定使用 **Bearer HTTP 授权方案** 的认证令牌，`Authorization: Bearer <TOKEN>`。`<TOKEN>` 是 Magento 令牌服务返回的认证令牌。

+   我们可以使用 `HTTP` 标头 `Accept: application/<FORMAT>`，其中 `<FORMAT>` 是 JSON 或 XML 之一。

我们可能如下概述一些 SOAP 特性：

+   我们通过 cURL 命令或 SOAP 客户端运行 SOAP Web API 调用。

+   只有为我们请求的服务才会生成 **Web 服务定义语言**（**WSDL**）文件。没有为所有服务合并的一个大 WSDL 文件。

+   Magento Web API 使用 WSDL 1.2，符合 WS-I 2.0 基本配置文件。

+   每个作为服务合同一部分的 Magento 服务接口在 WSDL 中都表示为一个独立的服务。

+   消费多个服务意味着在 WSDL 端点 URL 中以逗号分隔的方式指定它们，例如 `http://<magento.host>/soap/<optional_store_code>?wsdl&services=<service_name_1>,<service_name_2>`。

+   我们可以通过在浏览器中访问类似 `http://<SHOP-URL>/soap/default?wsdl_list` 的 URL 来获取所有可用服务的列表。

以下 REST 和 SOAP 示例将广泛使用 cURL，它本质上是一个允许您从命令行或不同语言实现（如 PHP）中发出 HTTP 请求的程序。我们可以进一步将 cURL 描述为控制台浏览器，或我们的网络 *view source* 工具。我们可以用各种花哨的 REST 和 SOAP 库做的任何事情，我们也可以用 cURL 做到；它只是被认为是一个更底层的做法。

使用 cURL 或其他没有内部实现 WSDL/XML 解析的任何东西进行 SOAP 请求是繁琐的。因此，使用 PHP SoapClient 或更健壮的工具是必须的。**SoapClient** 是 PHP 的一个集成、积极维护的部分，因此通常是可用的。

即使有负分，我们仍将使用控制台 cURL、PHP cURL 和 PHP SoapClient 示例来展示所有我们的 API 调用。鉴于库抽象了如此多的功能，开发人员对 cURL 有一个坚实的理解是绝对必要的，即使是为了进行 SOAP 调用。

# 基于令牌的认证实践

基于令牌的认证的核心如下：

+   客户端使用用户名和密码请求访问

+   应用程序验证凭证

+   应用程序向客户端提供一个签名令牌

以下代码示例演示了针对客户用户的控制台 cURL REST 请求：

```php
curl -X POST "http://magento2.ce/rest/V1/integration/customer/token"\
 -H "Content-Type:application/json"\
 -d '{"username":"john@change.me", "password":"abc123"}'

```

以下代码示例演示了针对客户用户的 PHP cURL REST 请求：

```php
$data = array('username' => 'john@change.me', 'password' => 'abc123');
$data_string = json_encode($data);

$ch = curl_init('http://magento2.ce/rest/V1/integration /customer/token');
  curl_setopt($ch, CURLOPT_CUSTOMREQUEST, 'POST');
  curl_setopt($ch, CURLOPT_POSTFIELDS, $data_string);
  curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
  curl_setopt($ch, CURLOPT_HTTPHEADER, array(
    'Content-Type: application/json',
    'Content-Length: ' . strlen($data_string))
);

$result = curl_exec($ch);
```

以下代码示例演示了针对客户用户的控制台 cURL SOAP 请求：

```php
curl -X POST -H 'Content-Type: application/soap+xml;
charset=utf-8; action= "integrationCustomerTokenServiceV1CreateCustomerAccessToken"'
-d @request.xml http://magento2.ce/index.php/soap/default?services= integrationCustomerTokenServiceV1

```

注意到 `-d @request.xml` 部分。在这里，我们告诉 `curl` 命令取 `request.xml` 文件的内容，并将其作为 `POST` 主体数据传递，其中 `request.xml` 文件的内容由前面的 `curl` 命令定义如下：

```php
<?xml version="1.0" encoding="UTF-8"?>
<env:Envelope  >
    <env:Body>
        <ns1:integrationCustomerTokenServiceV1CreateCustomer AccessTokenRequest>
            <username>john@change.me</username>
            <password>abc123</password>
        </ns1:integrationCustomerTokenServiceV1CreateCustomer AccessTokenRequest>
    </env:Body>
</env:Envelope>
```

以下代码示例演示了针对客户用户的 PHP cURL SOAP-like 请求：

```php
$data_string = file_get_contents('request.xml');

$ch = curl_init('http://magento2.ce/index.php/soap/default?services= integrationCustomerTokenServiceV1');
  curl_setopt($ch, CURLOPT_CUSTOMREQUEST, 'POST');
  curl_setopt($ch, CURLOPT_POSTFIELDS, $data_string);
  curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
  curl_setopt($ch, CURLOPT_HTTPHEADER, array(
    'Content-Type: application/soap+xml; charset=utf-8; action="integrationCustomerTokenServiceV1 CreateCustomerAccessToken"',
    'Content-Length: ' . strlen($data_string))
);

$result = curl_exec($ch);
```

以下代码示例演示了使用 PHP SoapClient 进行 Web API 调用的用法：

```php
$request = new SoapClient(
    'http://magento2.ce/index.php/soap/default?wsdl&services= integrationCustomerTokenServiceV1',
    array('soap_version' => SOAP_1_2, 'trace' => 1)
);

$token = $request->integrationCustomerTokenServiceV1Create CustomerAccessToken(array('username' => 'john@change.me', 'password' => 'abc123'));
```

管理员用户认证的 API 调用几乎相同，并且取决于我们采取的三种方法中的哪一种。区别仅在于在 REST 的情况下，使用`https://magento2.ce/rest/V1/integration/admin/token`作为端点 URL，以及在使用`http://magento2.ce/index.php/soap/default?services=integrationCustomerTokenServiceV1`的情况下。此外，对于 SOAP 调用，我们在`$request`对象上调用`integrationAdminTokenServiceV1CreateAdminAccessToken`。

在认证成功的情况下，无论是客户还是管理员 API 调用，响应都会是一个看起来随机的 32 个字符长字符串，我们称之为令牌。此令牌随后被保存在数据库中的`oauth_token`表中的令牌列下。

这可能对`oauth_token`表与令牌认证有什么关系有些困惑。

### 注意

如果我们仔细思考，基于令牌的认证可以看作是 OAuth 的简化版本，其中用户会使用用户名和密码进行认证，然后将获得的时效性令牌提供给第三方应用程序使用。

在认证失败的情况下，服务器会返回**HTTP 401 未授权**，其体包含一个 JSON 消息：

```php
{"message":"Invalid login or password."}
```

注意我们如何能够调用 API 方法，尽管我们尚未进行认证？这意味着我们必须调用由匿名资源类型定义的 API。快速查看 API 端点可以给我们一些关于其定义位置的提示。在`vendor/magento/module-integration/etc/webapi.xml`文件下查看，我们可以看到以下（截断的）XML：

```php
<route url="/V1/integration/admin/token" method="POST">
    <service class="Magento\Integration\Api\AdminTokenServiceInterface" method="createAdminAccessToken"/>
    <resources>
        <resource ref="anonymous"/>
    </resources>
</route>
<route url="/V1/integration/customer/token" method="POST">
    <service class="Magento\Integration\Api\ CustomerTokenServiceInterface" method="createCustomerAccessToken"/>
    <resources>
        <resource ref="anonymous"/>
    </resources>
</route>
```

我们可以清楚地看到，即使是基于令牌的认证本身也被定义为 API，使用匿名资源以便每个人都可以访问它。简而言之，基于令牌的认证是`Magento\Integration`模块的一个特性。

现在我们已经有了认证令牌，我们可以开始进行其他 API 调用。记住，令牌仅仅意味着我们已经对给定的用户名和密码进行了认证。它并不意味着我们能够访问所有 Web API 方法。这进一步取决于我们的客户或用户是否有适当的访问角色。

# 基于 OAuth 的认证实践

基于 OAuth 的认证是 Magento 支持的最复杂但最灵活的一种。在我们使用它之前，商家必须将我们的外部应用程序注册为与 Magento 实例的集成。以商家的身份，我们在**系统** | **扩展** | **集成**下的 Magento 管理区域进行操作。点击**添加新集成**按钮会打开如下截图所示的界面：

![基于 OAuth 的认证实践](img/00062.jpeg)

`External Book App`的值是我们外部应用的自由命名。如果我们将其与 Twitter 连接，我们就可以轻松地将它的名字放在这里。在**名称**旁边，我们有**电子邮件**、**回调 URL**和**身份链接 URL**字段。电子邮件的值并不是特别重要。回调 URL 和身份链接 URL 定义了接收 OAuth 凭证的外部应用端点。这些链接的值指向作为 OAuth 客户端的*外部应用*。我们稍后会回到这一点。

在**可用 API**面板下的**API**选项卡中，我们将**资源访问**设置为**全部**或**自定义**。如果设置为**自定义**，我们可以在**资源**选项中进一步微调我们希望允许访问此集成的资源，如下截图所示：

![动手实践 OAuth 基于的身份验证](img/00063.jpeg)

我们应该始终给外部应用我们正在使用的最小必要资源。这样，我们最小化了可能的安全风险。前面的截图显示我们只定义了`Sales`、`Products`、`Customer`和`Marketing`资源到集成中。这意味着 API 用户将无法使用内容资源，例如保存或删除页面。

如果我们现在点击**保存**按钮，我们应该被重定向回**系统** | **扩展** | **集成**屏幕，如下截图所示：

![动手实践 OAuth 基于的身份验证](img/00064.jpeg)

在这里，我们需要关注三件事情。首先，我们看到一个**集成不安全**的消息。这是因为当我们定义回调 URL 和身份链接 URL 时，我们使用了 HTTP 协议而不是 HTTPS 协议。在现实世界的连接中，出于安全考虑，我们需要确保使用 HTTPS。此外，我们注意到**状态**列仍然显示为**不活跃**。

位于**状态**列右侧的**激活**链接是启动双因素 OAuth 握手之前的步骤。只有能够访问后端集成列表的管理员才能启动此步骤。

在这一点上，我们需要从以下位置拉取`External Book App` OAuth 客户端背后的全部 PHP 代码，[`github.com/ajzele/B05032-BookAppOauthClient`](https://github.com/ajzele/B05032-BookAppOauthClient)，并将其放置在我们 Magento 安装根目录下的`pub/external-book-app/`文件夹中，如下截图所示：

![动手实践 OAuth 基于的身份验证](img/00065.jpeg)

这些文件的功能是模拟我们自己的迷你 OAuth 客户端。我们不会深入探讨这些文件的内容，更重要的是将其视为一个外部 OAuth 客户端应用。当 Magento 触发回调和配置在上一页输出图像下的身份链接 URL 时，`callback-url.php`和`identity-link-url.php`文件将会执行。

一旦 OAuth 客户端文件就绪，我们回到我们的集成列表。在这里，我们点击 **激活** 链接。这打开了一个模态框，要求我们批准访问 API 资源，如下面的截图所示：

![动手实践 OAuth 基于的身份验证](img/00066.jpeg)

注意这里列出的 API 资源与我们在创建集成时在 **API** 选项卡下设置的那些相匹配。我们在这里实际上只能做两件事：要么点击 **取消**，要么点击 **允许** 来开始双因素 OAuth 握手。点击 **允许** 按钮并行执行两件事。

首先，它立即将凭据发送到创建 `外部图书应用` 集成时指定的端点（回调 URL）。从 Magento 到回调 URL 的 **HTTP POST** 包含与以下类似的参数值：

```php
Array
(
    [oauth_consumer_key] => cn5anfyvkg7sgm2lrv8cxvq0dxcrj7xm
    [oauth_consumer_secret] => wvmgy0dmlkos2vok04k3h94r40jvi5ye
    [store_base_url] => http://magento2-merchant.loc/index.php/
    [oauth_verifier] => hlnsftola6c7b6wjbtb6wwfx4tow2x6x
)
```

基本上，一个 HTTP POST 请求正在击中 `callback-url.php` 文件，其内容（部分）如下：

```php
session_id('BookAppOAuth');
session_start();

$_SESSION['oauth_consumer_key'] = $_POST['oauth_consumer_key'];
$_SESSION['oauth_consumer_secret'] = $_POST['oauth_consumer_secret'];
$_SESSION['store_base_url'] = $_POST['store_base_url'];
$_SESSION['oauth_verifier'] = $_POST['oauth_verifier'];

session_write_close();

header('HTTP/1.0 200 OK');

echo 'Response';
```

我们可以看到，通过 Magento 传递的参数被存储在一个名为 `BookAppOAuth` 的外部应用会话中。稍后，在 `check-login.php` 文件中，这些参数将被用来实例化 `BookAppOauthClient`，这将进一步被用来获取一个请求令牌，这是一个预先授权的令牌。

与 **回调 URL** HTTP POST 并行，我们打开了一个弹出窗口，如下面的截图所示：

![动手实践 OAuth 基于的身份验证](img/00067.jpeg)

我们在弹出窗口中看到的登录表单只是我们放在 `identity-link-url.php` 文件下的某些虚拟内容。Magento 通过 HTTP GET 向此文件传递两个值。这些是 `consumer_id` 和 `success_call_back`。`consumer_id` 的值是我们创建在管理区域中的集成 ID。OAuth 客户端应用决定是否要使用此值。`success_call_back` URL 指向我们的 Magento `admin integration/loginSuccessCallback` 路径。如果我们查看 `identity-link-url.php` 文件的代码，我们可以看到表单被设置为在 URL 上执行 `POST` 动作，例如 `check-login.php?consumer_id={$consumerId}&callback_url={$callbackUrl}`。

如果我们现在点击 **登录** 按钮，表单将 `POST` 数据到 `check-login.php` 文件，并在 URL 中作为 `GET` 参数传递 `consumer_id` 和 `callback_url`。

`check-login.php` 的内容（部分）定义如下：

```php
require '../../vendor/autoload.php';

$consumer = $_REQUEST['consumer_id'];
$callback = $_REQUEST['callback_url'];

session_id('BookAppOAuth');
session_start();

$consumerKey = $_SESSION['oauth_consumer_key'];
$consumerSecret = $_SESSION['oauth_consumer_secret'];
$magentoBaseUrl = rtrim($_SESSION['store_base_url'], '/');
$oauthVerifier = $_SESSION['oauth_verifier'];

define('MAGENTO_BASE_URL', $magentoBaseUrl);

$credentials = new \OAuth\Common\Consumer\Credentials($consumerKey, $consumerSecret, $magentoBaseUrl);
$oAuthClient = new BookAppOauthClient($credentials);
$requestToken = $oAuthClient->requestRequestToken();

$accessToken = $oAuthClient->requestAccessToken(
    $requestToken->getRequestToken(),
    $oauthVerifier,
    $requestToken->getRequestTokenSecret()
);

header('Location: '. $callback);
```

为了简化，我们在这里没有进行真正的用户登录检查。我们可能在 OAuth 相关调用之上添加了一个，然后在允许使用 OAuth 之前对用户进行用户名和密码的验证。然而，出于简化原因，我们从我们的示例 OAuth 客户端应用中省略了这部分。

在 `check-login.php` 文件中，我们可以看到，基于之前存储的会话参数，我们执行以下操作：

+   通过传递存储在会话中的 `oauth_consumer_key`、`oauth_consumer_secret` 和 `store_base_url` 来实例化 `\OAuth\Common\Consumer\Credentials` 对象

+   实例化 `BookAppOauthClient` 对象，将其构造函数传递整个凭据对象

+   使用 `OauthClient` 对象获取请求令牌

+   使用请求令牌获取长期访问令牌

如果一切执行成功，弹出窗口将关闭，我们将被重定向回集成列表。现在的不同之处在于，查看网格时，我们有一个**活动**状态，旁边有一个**重新授权**链接，如下面的截图所示：

![动手实践 OAuth 基于的身份验证](img/00068.jpeg)

我们真正想要在这个阶段的是**访问令牌**和**访问令牌密钥**。如果我们编辑 `External Book App` 集成，我们可以看到这些值。这些值现在应该出现在以下截图所示的**集成详细信息**选项卡上：

![动手实践 OAuth 基于的身份验证](img/00069.jpeg)

**访问令牌**是我们所有后续 API 调键的密钥，并且我们成功地完成了基于 OAuth 的身份验证部分。

# 基于 OAuth 的 Web API 调用

一旦我们获得了 OAuth 访问令牌，从前面的步骤中，我们可以开始对其他方法进行 Web API 调用。尽管 Web API 覆盖范围对 REST 和 SOAP 都相同，但在进行方法调用时存在显著差异。

为了提供一个更健壮的示例，我们将针对客户组 `save` 方法，部分定义在 `vendor/magento/module-customer/etc/webapi.xml` 文件中，如下所示：

```php
<route url="/V1/customerGroups" method="POST">
    <service class="Magento\Customer\Api\GroupRepositoryInterface" method="save"/>
    <resources>
        <resource ref="Magento_Customer::group"/>
    </resources>
</route>
```

要使用访问令牌进行 Web API 调用，例如 `POST /V1/customerGroups`，我们需要在调用中包含这些请求参数在授权请求头中：

+   `oauth_consumer_key`，从 Magento 管理区域获取，在集成编辑屏幕下。

+   `oauth_nonce`，随机值，应用程序为每个请求唯一生成。

+   `oauth_signature_method`，用于签名请求的签名方法名称。有效值有：`HMAC-SHA1`、`RSA-SHA1` 和 `PLAINTEXT`。

+   尽管 OAuth 协议支持`PLAINTEXT`，但 Magento 不支持。我们将使用`HMAC-SHA1`。

+   `oauth_timestamp`，整数值，Unix-like 时间戳。

+   `oauth_token`，从 Magento 管理区域获取，在集成编辑屏幕下。

+   `oauth_version`，Magento 支持 Oauth 1.0a，因此我们使用 `1.0`。

+   `oauth_signature`，生成的签名值，在签名生成过程中省略。

要为 HTTP 请求生成 OAuth 1.0a HMAC-SHA1 签名需要集中精力，如果手动完成。

我们需要确定请求的 HTTP 方法和 URL，它等于 `POST http://magento2-merchant.loc/rest/V1/customerGroups`。在这里使用正确的协议非常重要，所以请确保 URL 的 `https://` 或 `http://` 部分与实际发送到 API 的请求相匹配。

我们然后收集请求中包含的所有参数。这些附加参数有两个位置：URL（作为查询字符串的一部分）和请求体。

在 HTTP 请求中，参数被 URL 编码，但我们需要收集原始值。除了请求参数外，每个 `oauth_*` 参数都需要包含在签名中，除了 `oauth_signature` 本身之外。

参数按照以下方式归一化为单个字符串：

+   参数按名称排序，使用字典序字节值排序。如果两个或多个参数具有相同的名称，则按其值排序。

+   参数按排序顺序连接成一个字符串。对于每个参数，名称与相应的值由一个 `=` 字符（ASCII 码 61）分隔，即使值是空的。每个名称-值对由一个 `&` 字符（ASCII 码 38）分隔。

此外，我们将签名密钥定义为 `{消费者密钥}+{&}+{访问令牌密钥}` 的值。

一旦我们将字符串归一化规则应用于参数并确定签名密钥，我们就调用 `hash_hmac('sha1', $data, {签名密钥}, true)` 来获取最终的 `oauth_signature` 值。

这应该会得到一个随机的 28 个字符长的字符串作为 `oauth_signature`，类似于这个 – `Pi/mGfA0SOlIxO9W30sEch6bjGE=`。

理解如何生成签名字符串很重要，但每次都正确地获取它既繁琐又耗时。我们可以通过实例化内置的 `\OAuth\Common\Consumer\Credentials` 和 `\OAuth\OAuth1\Signature\Signature` 类的对象来帮助自己，如下（部分）所示：

```php
$credentials = new \OAuth\Common\Consumer\Credentials($consumerKey, $consumerSecret, $magentoBaseUrl);
$signature = new \OAuth\OAuth1\Signature\Signature($credentials);
$signature->setTokenSecret($accessTokenSecret);
$signature->setHashingAlgorithm('HMAC-SHA1');

echo $signature->getSignature($uri, array(
    'oauth_consumer_key' => $consumerKey,
    'oauth_nonce' => 'per-request-unique-token',
    'oauth_signature_method' => 'HMAC-SHA1',
    'oauth_timestamp' => '1437319569',
    'oauth_token' => $accessToken,
    'oauth_version' => '1.0',
), 'POST');
```

现在我们有了 `oauth_signature` 值，我们就准备好在我们的控制台 `curl` REST 示例中操作了。这归结为在控制台上运行以下命令：

```php
curl -X POST http://magento2.ce/rest/V1/customerGroups
-H 'Content-Type: application/json'
-H 'Authorization: OAuth
oauth_consumer_key="vw2xi6kaq0o3f7ay60owdpg2f8nt66g6",
oauth_nonce="per-request-token-by-app-1",
oauth_signature_method="HMAC-SHA1",
oauth_timestamp="1437319569",
oauth_token="cney3fmk9p5282bm1khb83q846l7dner",
oauth_version="1.0",
oauth_signature="Pi/mGfA0SOlIxO9W30sEch6bjGE="'
-d '{"group": {"code": "The Book Writer", "tax_class_id": "3"}}'

```

注意，前面的命令只是从视觉上换行。它应该在控制台上是一行。一旦执行，API 调用将创建一个新的客户组，名为 `The Book Writer`。观察 `curl` 命令时，可能会有人问，为什么我们没有对通过 `-d` 标志开关传递的 JSON POST 数据进行归一化。这是因为如果内容类型为 `application/x-www-form-urlencoded`，则只有 HTTP POST 请求体中的参数才被考虑用于签名生成。

控制台 cURL SOAP 请求不需要使用 OAuth 签名。我们可以通过将 `Authorization: Bearer {访问令牌值}` 传递到请求头中执行 SOAP 请求，如下所示：

```php
curl -X POST http://magento2.ce/index.php/soap/default?services= customerGroupRepositoryV1 -H 'Content-Type: application/soap+xml; charset=utf-8; action="customerGroupRepositoryV1Save"' -H 'Authorization: Bearer cney3fmk9p5282bm1khb83q846l7dner' -d @request.xml

```

其中 `request.xml` 包含以下内容：

```php
<?xml version="1.0" encoding="UTF-8"?>
<env:Envelope  >
    <env:Body>
        <ns1:customerGroupRepositoryV1SaveRequest>
            <group>
                <code>The Book Writer</code>
                <taxClassId>3</taxClassId>
            </group>
        </ns1:customerGroupRepositoryV1SaveRequest>
    </env:Body>
</env:Envelope>
```

以下代码示例演示了针对客户组 `save` 方法调用的 PHP cURL SOAP 类似请求：

```php
$request = new SoapClient(
    'http://magento2.ce/index.php/soap/?wsdl&services= customerGroupRepositoryV1',
    array(
        'soap_version' => SOAP_1_2,
        'stream_context' => stream_context_create(array(
            'http' => array(
                'header' => 'Authorization: Bearer cney3fmk9p5282bm1khb83q846l7dner')
            )
        )
    )
);

$response = $request->customerGroupRepositoryV1Save(array(
    'group' => array(
        'code' => 'The Book Writer',
        'taxClassId' => 3
    )
));
```

注意方法名 `customerGroupRepositoryV1Save` 实际上由服务名 `customerGroupRepositoryV1` 和服务内部实际方法的 `Save` 名称组成。

我们可以通过在浏览器中打开类似 `http://magento2.ce/soap/default?wsdl_list` 的 URL 来获取所有定义的服务列表（取决于我们的 Magento 安装）。

# 基于会话认证的实践操作

基于会话的认证是 Magento 中第三种也是最简单的一种认证方式。在这里我们没有 token 传递的复杂性。作为客户，我们使用客户凭证登录到 Magento 店面。作为管理员，我们使用管理员凭证登录到 Magento 后台。Magento 使用名为`PHPSESSID`的 cookie 来跟踪存储我们的登录状态的会话。Web API 框架使用我们登录的会话信息来验证我们的身份并授权访问请求的资源。

客户可以访问在`webapi.xml`配置文件中配置了匿名或自授权的资源，如`GET /rest/V1/customers/me`。

如果我们在浏览器中尝试打开`http://magento2.ce/rest/V1/customers/me` URL，但没有以客户身份登录，我们会得到以下响应：

```php
<response>
    <message>Consumer is not authorized to access %resources</message>
    <parameters>
        <resources>self</resources>
    </parameters>
</response>
```

如果我们以客户身份登录然后尝试打开相同的 URL，我们会得到以下响应：

```php
<response>
    <id>2</id>
    <group_id>1</group_id>
    <created_at>2015-11-22 14:15:33</created_at>
    <created_in>Default Store View</created_in>
    <email>john@change.me</email>
    <firstname>John</firstname>
    <lastname>Doe</lastname>
    <store_id>1</store_id>
    <website_id>1</website_id>
    <addresses/>
    <disable_auto_group_change>0</disable_auto_group_change>
</response>
```

管理员用户可以访问分配给他们的 Magento 管理员配置文件的资源。

# 创建自定义 Web API

Magento 附带了一系列我们可以调用的 API 方法。然而，有时这还不够，因为我们的业务需求需要额外的逻辑，我们需要能够向 Web API 添加我们自己的方法。

创建我们自己的 API 的最佳部分是，我们不必担心它们是 REST 还是 SOAP。Magento 抽象化这一点，使得我们的 API 方法自动对 REST 和 SOAP 调用可用。

在概念上，添加新的 API 涉及到两个方面：通过各种类定义业务逻辑，并通过`webapi.xml`文件公开它。然而，正如我们很快就会看到的，这有很多**样板代码**。

让我们创建一个名为`Foggyline_Slider`的微型模块，我们将演示`创建（POST）`、`更新（PUT）`、`删除（DELETE）`和`列表（GET）`方法调用。

创建一个模块注册文件，`app/code/Foggyline/Slider/registration.php`，内容（部分）如下：

```php
\Magento\Framework\Component\ComponentRegistrar::register(
    \Magento\Framework\Component\ComponentRegistrar::MODULE,
    'Foggyline_Slider',
    __DIR__
);
```

创建一个模块配置文件，`app/code/Foggyline/Slider/etc/module.xml`，内容如下：

```php
<config  xsi:noNamespaceSchemaLocation="urn:magento:framework:Module /etc/module.xsd">
    <module name="Foggyline_Slider" setup_version="1.0.0"/>
</config>
```

创建一个安装脚本，我们的未来模型将持久化模块数据。我们通过创建`app/code/Foggyline/Slider/Setup/InstallSchema.php`文件来实现，内容（部分）如下：

```php
namespace Foggyline\Slider\Setup;

use Magento\Framework\Setup\InstallSchemaInterface;
use Magento\Framework\Setup\ModuleContextInterface;
use Magento\Framework\Setup\SchemaSetupInterface;

class InstallSchema implements InstallSchemaInterface
{
    public function install(SchemaSetupInterface $setup, ModuleContextInterface $context)
    {
        $installer = $setup;
        $installer->startSetup();

        /**
         * Create table 'foggyline_slider_slide'
         */
        $table = $installer->getConnection()
            ->newTable($installer- >getTable('foggyline_slider_slide'))
            ->addColumn(
                'slide_id',
                \Magento\Framework\DB\Ddl\Table::TYPE_INTEGER,
                null,
                ['identity' => true, 'unsigned' => true, 'nullable' => false, 'primary' => true],
                'Slide Id'
            )
            ->addColumn(
                'title',
                \Magento\Framework\DB\Ddl\Table::TYPE_TEXT,
                200,
                [],
                'Title'
            )
            ->setComment('Foggyline Slider Slide');
        $installer->getConnection()->createTable($table);
        ...
        $installer->endSetup();

    }
}
```

现在我们指定我们资源的 ACL。我们的资源是我们对我们模块实体执行的 CRUD 操作。我们将以`slide`和`image`作为独立实体来结构化我们的模块，其中一张幻灯片可以与多个图像实体相关联。因此，我们希望能够分别控制每个实体的保存和删除操作的访问权限。我们通过定义`app/code/Foggyline/Slider/etc/acl.xml`文件如下来实现：

```php
<config  xsi:noNamespaceSchemaLocation="urn:magento:framework:Acl/etc/ acl.xsd">
    <acl>
        <resources>
            <resource id="Magento_Backend::admin">
                <resource id="Magento_Backend::content">
                    <resource id= "Magento_Backend::content_elements">
                        <resource id="Foggyline_Slider::slider" title="Slider" sortOrder="10">
                            <resource id="Foggyline_Slider::slide" title="Slider Slide" sortOrder="10">
                                <resource id= "Foggyline_Slider::slide_save" title="Save Slide" sortOrder="10" />
                                <resource id="Foggyline_Slider:: slide_delete" title="Delete Slide" sortOrder="20" />
                            </resource>
                            <resource id="Foggyline_Slider::image" title="Slider Image" sortOrder="10">
                                <resource id= "Foggyline_Slider::image_save" title="Save Image" sortOrder="10" />
                                <resource id= "Foggyline_Slider::image_delete" title="Delete Image" sortOrder="20" />
                            </resource>
                        </resource>
                    </resource>
                </resource>
            </resource>
        </resources>
    </acl>
</config>
```

现在 ACL 已经设置好了，我们在`app/code/Foggyline/Slider/etc/webapi.xml`文件（部分）中定义我们的 Web API 资源，如下所示：

```php
<routes  xsi:noNamespaceSchemaLocation= "urn:magento:module:Magento_Webapi:etc/webapi.xsd">
    <route url="/V1/foggylineSliderSlide/:slideId" method="GET">
        <service class="Foggyline\Slider\Api\ SlideRepositoryInterface" method="getById" />
        <resources>
            <resource ref="Foggyline_Slider::slide" />
        </resources>
    </route>
    <route url="/V1/foggylineSliderSlide/search" method="GET">
        <service class="Foggyline\Slider\Api\ SlideRepositoryInterface" method="getList" />
        <resources>
            <resource ref="anonymous" />
        </resources>
    </route>
    <route url="/V1/foggylineSliderSlide" method="POST">
        <service class="Foggyline\Slider\Api\ SlideRepositoryInterface" method="save" />
        <resources>
            <resource ref="Foggyline_Slider::slide_save" />
        </resources>
    </route>
    <route url="/V1/foggylineSliderSlide/:id" method="PUT">
        <service class="Foggyline\Slider\Api\ SlideRepositoryInterface" method="save" />
        <resources>
            <resource ref="Foggyline_Slider::slide_save" />
        </resources>
    </route>
    <route url="/V1/foggylineSliderSlide/:slideId" method="DELETE">
        <service class="Foggyline\Slider\Api\ SlideRepositoryInterface" method="deleteById" />
        <resources>
            <resource ref="Foggyline_Slider::slide_delete" />
        </resources>
    </route>
    <route url="/V1/foggylineSliderImage/:imageId" method="GET">
        <service class="Foggyline\Slider\Api\ ImageRepositoryInterface" method="getById" />
        <resources>
            <resource ref="Foggyline_Slider::image" />
        </resources>
    </route>
    <route url="/V1/foggylineSliderImage/search" method="GET">
        <service class="Foggyline\Slider\Api\ ImageRepositoryInterface" method="getList" />
        <resources>
            <resource ref="Foggyline_Slider::image" />
        </resources>
    </route>
    <route url="/V1/foggylineSliderImage" method="POST">
        <service class="Foggyline\Slider\Api\ ImageRepositoryInterface" method="save" />
        <resources>
            <resource ref="Foggyline_Slider::image_save" />
        </resources>
    </route>
    <route url="/V1/foggylineSliderImage/:id" method="PUT">
        <service class="Foggyline\Slider\Api\ ImageRepositoryInterface" method="save" />
        <resources>
            <resource ref="Foggyline_Slider::image_save" />
        </resources>
    </route>
    <route url="/V1/foggylineSliderImage/:imageId" method="DELETE">
        <service class="Foggyline\Slider\Api\ ImageRepositoryInterface" method="deleteById" />
        <resources>
            <resource ref="Foggyline_Slider::image_delete" />
        </resources>
    </route>
</routes>
```

注意到每个服务类属性都指向接口，而不是类。这是我们构建可公开服务的方式，总是有一个接口定义在它们后面。正如我们很快就会看到的，使用`di.xml`，这并不意味着 Magento 会直接从这些接口创建对象。

现在我们创建`app/code/Foggyline/Slider/etc/di.xml`文件，内容（部分）如下：

```php
<config  xsi:noNamespaceSchemaLocation= "urn:magento:framework:ObjectManager/etc/config.xsd">

    <preference for="Foggyline\Slider\Api\Data\SlideInterface" type="Foggyline\Slider\Model\Slide"/>

    <preference for="Foggyline\Slider\Api\ SlideRepositoryInterface" type= "Foggyline\Slider\Model\SlideRepository"/>
    ...
</config>
```

这里发生的事情是我们告诉 Magento 类似于，“嘿，每当你需要传递一个符合`Foggyline\Slider\Api\Data\SlideInterface`接口的实例时，最好使用`Foggyline\Slider\Model\Slide`类。”

到目前为止，我们还没有实际创建任何这些接口或模型类。在创建 API 时，我们应该首先从定义接口开始，然后我们的模型应该从这些接口扩展。

接口`Foggyline\Slider\Api\Data\SlideInterface`定义在`app/code/Foggyline/Slider/Api/Data/SlideInterface.php`文件中（部分）如下：

```php
namespace Foggyline\Slider\Api\Data;

/**
* @api
*/
interface SlideInterface
{
    const PROPERTY_ID = 'slide_id';
    const PROPERTY_SLIDE_ID = 'slide_id';
    const PROPERTY_TITLE = 'title';

    /**
    * Get Slide entity 'slide_id' property value
    * @return int|null
    */
    public function getId();

    /**
    * Set Slide entity 'slide_id' property value
    * @param int $id
    * @return $this
    */
    public function setId($id);

    /**
    * Get Slide entity 'slide_id' property value
    * @return int|null
    */
    public function getSlideId();

    /**
    * Set Slide entity 'slide_id' property value
    * @param int $slideId
    * @return $this
    */
    public function setSlideId($slideId);

    /**
    * Get Slide entity 'title' property value
    * @return string|null
    */
    public function getTitle();

    /**
    * Set Slide entity 'title' property value
    * @param string $title
    * @return $this
    */
    public function setTitle($title);
}
```

我们正在追求极致的简化。我们的`Slide`实体实际上只有 ID 和标题值。`id`和`slide_id`指向数据库中的同一字段，它们的 getter 和 setter 的实现应该产生相同的结果。

虽然`API/Data/*.php`接口成为我们的数据模型的设计蓝图要求，但我们也有`Api/*RepositoryInterface.php`文件。这里的想法是将创建、更新、删除、搜索和类似的数据处理逻辑从数据模型类提取出来，放入它自己的类中。这样，我们的模型类就变成了更纯粹的数据和业务逻辑类，而其余的持久化和搜索相关逻辑则移动到这些存储库类中。

我们的**幻灯片存储接口**定义在`app/code/Foggyline/Slider/Api/SlideRepositoryInterface.php`文件中，如下所示：

```php
namespace Foggyline\Slider\Api;

/**
* @api
*/
interface SlideRepositoryInterface
{
    /**
    * Retrieve slide entity.
    * @param int $slideId
    * @return \Foggyline\Slider\Api\Data\SlideInterface
    * @throws \Magento\Framework\Exception\NoSuchEntityException If slide with the specified ID does not exist.
    * @throws \Magento\Framework\Exception\LocalizedException
    */
    public function getById($slideId);

    /**
    * Save slide.
    * @param \Foggyline\Slider\Api\Data\SlideInterface $slide
    * @return \Foggyline\Slider\Api\Data\SlideInterface
    * @throws \Magento\Framework\Exception\LocalizedException
    */
    public function save(\Foggyline\Slider\Api\Data\SlideInterface $slide);

    /**
    * Retrieve slides matching the specified criteria.
    * @param \Magento\Framework\Api\SearchCriteriaInterface $searchCriteria
    * @return \Magento\Framework\Api\SearchResultsInterface
    * @throws \Magento\Framework\Exception\LocalizedException
    */
    public function getList(\Magento\Framework\Api\SearchCriteriaInterface $searchCriteria);

    /**
    * Delete slide by ID.
    * @param int $slideId
    * @return bool true on success
    * @throws \Magento\Framework\Exception\NoSuchEntityException
    * @throws \Magento\Framework\Exception\LocalizedException
    */
    public function deleteById($slideId);
}
```

在接口就位后，我们可以继续到模型类。为了在数据库中持久化和获取数据，我们的`Slide`实体在`Model`目录下确实需要三个文件。这些被称为数据*模型*、*资源类*和*集合类*。

数据模型类定义在`app/code/Foggyline/Slider/Model/Slide.php`文件中（部分）如下：

```php
namespace Foggyline\Slider\Model;

class Slide extends \Magento\Framework\Model\AbstractModel
    implements \Foggyline\Slider\Api\Data\SlideInterface {     /**
    * Initialize Foggyline Slide Model
    *
    * @return void
    */
    protected function _construct()
    {
        /* _init($resourceModel) */
        $this->_init ('Foggyline\Slider\Model\ResourceModel\Slide');
    }

    /**
    * Get Slide entity 'slide_id' property value
    *
    * @api
    * @return int|null
    */
    public function getId()
    {
        return $this->getData(self::PROPERTY_ID);
    }

    /**
    * Set Slide entity 'slide_id' property value
    *
    * @api
    * @param int $id
    * @return $this
    */
    public function setId($id)
    {
        $this->setData(self::PROPERTY_ID, $id);
        return $this;
    }

    /**
    * Get Slide entity 'slide_id' property value
    *
    * @api
    * @return int|null
    */
    public function getSlideId()
    {
        return $this->getData(self::PROPERTY_SLIDE_ID);
    }

    /**
    * Set Slide entity 'slide_id' property value
    *
    * @api
    * @param int $slideId
    * @return $this
    */
    public function setSlideId($slideId)
    {
        $this->setData(self::PROPERTY_SLIDE_ID, $slideId);
        return $this;
    }

    /**
    * Get Slide entity 'title' property value
    *
    * @api
    * @return string|null
    */
    public function getTitle()
    {
        return $this->getData(self::PROPERTY_TITLE);
    }

    /**
    * Set Slide entity 'title' property value
    *
    * @api
    * @param string $title
    * @return $this
    */
    public function setTitle($title)
    {
        $this->setData(self::PROPERTY_TITLE, $title);
    }
}
```

接着是模型资源类，定义在`app/code/Foggyline/Slider/Model/ResourceModel/Slide.php`文件中（部分）如下：

```php
namespace Foggyline\Slider\Model\ResourceModel;

/**
* Foggyline Slide resource
*/
class Slide extends \Magento\Framework\Model\ResourceModel\Db\AbstractDb
{
    /**
    * Define main table
    *
    * @return void
    */
    protected function _construct()
    {
        /* _init($mainTable, $idFieldName) */
        $this->_init('foggyline_slider_slide', 'slide_id');
    }
}
```

最后，第三部分是模型集合类，定义在`app/code/Foggyline/Slider/Model/ResourceModel/Slide/Collection.php`文件中，如下所示：

```php
namespace Foggyline\Slider\Model\ResourceModel\Slide;

/**
* Foggyline slides collection
*/
class Collection extends \Magento\Framework\Model\ResourceModel\Db\Collection\ AbstractCollection
{
    /**
    * Define resource model and model
    *
    * @return void
    */
    protected function _construct()
    {
        /* _init($model, $resourceModel) */
        $this->_init('Foggyline\Slider\Model\Slide', 'Foggyline\Slider\Model\ResourceModel\Slide');
    }
}
```

如果我们现在手动实例化模型数据类，我们就能在数据库中持久化数据。为了完成`di.xml`的要求，我们仍然缺少一个最后的成分——`Model/SlideRepository`类文件。

让我们创建`app/code/Foggyline/Slider/Model/SlideRepository.php`文件，内容（部分）如下：

```php
namespace Foggyline\Slider\Model;

use Magento\Framework\Api\DataObjectHelper;
use Magento\Framework\Api\SearchCriteriaInterface;
use Magento\Framework\Exception\CouldNotDeleteException;
use Magento\Framework\Exception\CouldNotSaveException;
use Magento\Framework\Exception\NoSuchEntityException;
use Magento\Framework\Reflection\DataObjectProcessor;

class SlideRepository implements \Foggyline\Slider\Api\SlideRepositoryInterface
{
    /**
    * @var \Foggyline\Slider\Model\ResourceModel\Slide
    */
    protected $resource;

    /**
    * @var \Foggyline\Slider\Model\SlideFactory
    */
    protected $slideFactory;

    /**
    * @var \Foggyline\Slider\Model\ResourceModel\Slide\ CollectionFactory
    */
    protected $slideCollectionFactory;

    /**
    * @var \Magento\Framework\Api\SearchResultsInterface
    */
    protected $searchResultsFactory;

    /**
    * @var \Magento\Framework\Api\DataObjectHelper
    */
    protected $dataObjectHelper;

    /**
    * @var \Magento\Framework\Reflection\DataObjectProcessor
    */
    protected $dataObjectProcessor;

    /**
    * @var \Foggyline\Slider\Api\Data\SlideInterfaceFactory
    */
    protected $dataSlideFactory;

    /**
    * @param ResourceModel\Slide $resource
    * @param SlideFactory $slideFactory
    * @param ResourceModel\Slide\CollectionFactory $slideCollectionFactory
    * @param \Magento\Framework\Api\SearchResultsInterface $searchResultsFactory
    * @param DataObjectHelper $dataObjectHelper
    * @param DataObjectProcessor $dataObjectProcessor
    * @param \Foggyline\Slider\Api\Data\SlideInterfaceFactory $dataSlideFactory
    */
    public function __construct(
        \Foggyline\Slider\Model\ResourceModel\Slide $resource,
        \Foggyline\Slider\Model\SlideFactory $slideFactory,
        \Foggyline\Slider\Model\ResourceModel\Slide\ CollectionFactory $slideCollectionFactory,
        \Magento\Framework\Api\SearchResultsInterface $searchResultsFactory,
        \Magento\Framework\Api\DataObjectHelper $dataObjectHelper,
        \Magento\Framework\Reflection\DataObjectProcessor $dataObjectProcessor,
        \Foggyline\Slider\Api\Data\SlideInterfaceFactory $dataSlideFactory

    )
    {
        $this->resource = $resource;
        $this->slideFactory = $slideFactory;
        $this->slideCollectionFactory = $slideCollectionFactory;
        $this->searchResultsFactory = $searchResultsFactory;
        $this->dataObjectHelper = $dataObjectHelper;
        $this->dataObjectProcessor = $dataObjectProcessor;
        $this->dataSlideFactory = $dataSlideFactory;
    }
    ...
}
```

可能看起来这里有很多事情要做，但实际上我们只是在构造函数中传递一些类和接口名称，以便实例化我们将用于 `webapi.xml` 文件中定义的各个服务方法的对象。

我们列表中的第一个服务方法是 `getById`，在 `SlideRepository.php` 中定义如下：

```php
/**
* Retrieve slide entity.
*
* @api
* @param int $slideId
* @return \Foggyline\Slider\Api\Data\SlideInterface
* @throws \Magento\Framework\Exception\NoSuchEntityException If slide with the specified ID does not exist.
* @throws \Magento\Framework\Exception\LocalizedException
*/
public function getById($slideId)
{
    $slide = $this->slideFactory->create();
    $this->resource->load($slide, $slideId);
    if (!$slide->getId()) {
        throw new NoSuchEntityException(__('Slide with id %1 does not exist.', $slideId));
    }
    return $slide;
}
```

然后我们有 `save` 方法，在 `SlideRepository.php` 中定义如下：

```php
/**
* Save slide.
*
* @param \Foggyline\Slider\Api\Data\SlideInterface $slide
* @return \Foggyline\Slider\Api\Data\SlideInterface
* @throws \Magento\Framework\Exception\LocalizedException
*/
public function save(\Foggyline\Slider\Api\Data\SlideInterface $slide)
{
    try {
        $this->resource->save($slide);
    } catch (\Exception $exception) {
        throw new CouldNotSaveException(__($exception- >getMessage()));
    }
    return $slide;
}
```

`save` 方法处理了 `webapi.xml` 中定义的 `POST` 和 `PUT` 请求，因此有效地处理了新幻灯片的创建或现有幻灯片的更新。

进一步来说，我们有 `getList` 方法，在 `SlideRepository.php` 中定义如下：

```php
/**
* Retrieve slides matching the specified criteria.
*
* @param \Magento\Framework\Api\SearchCriteriaInterface $searchCriteria
* @return \Magento\Framework\Api\SearchResultsInterface
* @throws \Magento\Framework\Exception\LocalizedException
*/
public function getList(\Magento\Framework\Api\SearchCriteriaInterface $searchCriteria)
{
    $this->searchResultsFactory->setSearchCriteria ($searchCriteria);

    $collection = $this->slideCollectionFactory->create();

    foreach ($searchCriteria->getFilterGroups() as $filterGroup) {         foreach ($filterGroup->getFilters() as $filter) {
            $condition = $filter->getConditionType() ?: 'eq';
            $collection->addFieldToFilter($filter->getField(), [$condition => $filter->getValue()]);
        }
    }
    $this->searchResultsFactory->setTotalCount($collection-> getSize());
    $sortOrders = $searchCriteria->getSortOrders();
    if ($sortOrders) {
        foreach ($sortOrders as $sortOrder) {
            $collection->addOrder(
                $sortOrder->getField(),
                (strtoupper($sortOrder->getDirection()) === 'ASC') ? 'ASC' : 'DESC'
            );
        }
    }
    $collection->setCurPage($searchCriteria->getCurrentPage());
    $collection->setPageSize($searchCriteria->getPageSize());
    $slides = [];
    /** @var \Foggyline\Slider\Model\Slide $slideModel */
    foreach ($collection as $slideModel) {
        $slideData = $this->dataSlideFactory->create();
        $this->dataObjectHelper->populateWithArray(
            $slideData,
            $slideModel->getData(),
            '\Foggyline\Slider\Api\Data\SlideInterface'
        );
        $slides[] = $this->dataObjectProcessor-> buildOutputDataArray(
            $slideData,
            '\Foggyline\Slider\Api\Data\SlideInterface'
        );
    }
    $this->searchResultsFactory->setItems($slides);
    return $this->searchResultsFactory;
}
```

最后，我们有 `deleteById` 方法，在 `SlideRepository.php` 中定义如下：

```php
/**
* Delete Slide
*
* @param \Foggyline\Slider\Api\Data\SlideInterface $slide
* @return bool
* @throws CouldNotDeleteException
*/
public function delete(\Foggyline\Slider\Api\Data\SlideInterface $slide)
{
    try {
        $this->resource->delete($slide);
    } catch (\Exception $exception) {
        throw new CouldNotDeleteException(__($exception-> getMessage()));
    }
    return true;
}

/**
* Delete slide by ID.
*
* @param int $slideId
* @return bool true on success
* @throws \Magento\Framework\Exception\NoSuchEntityException
* @throws \Magento\Framework\Exception\LocalizedException
*/
public function deleteById($slideId)
{
    return $this->delete($this->getById($slideId));
}
```

请记住，我们只在前面的部分代码示例中涵盖了 `Slide` 实体，这对于进一步进行 API 调用示例已经足够。

## API 调用示例

由于我们定义的所有 API 都受资源保护，我们首先需要以管理员用户身份进行身份验证，假设管理员用户可以访问我们定义的所有自定义资源，包括我们定义的资源。为了简化，我们将使用基于令牌的认证方法，例如，本章前面给出的示例。一旦认证成功，我们应该有一个 32 个随机字符长的令牌，例如 `pk8h93nq9cevaw55bohkjbp0o7kpl4d3`。

一旦获取了令牌密钥，我们将使用控制台 cURL、PHP cURL、PHP SoapClient 以及控制台 SOAP 风格 cURL 示例来测试以下 API 调用：

+   `GET /V1/foggylineSliderSlide/:slideId`, 调用 `getById` 服务方法，需要 `Foggyline_Slider::slide` 资源

+   `GET /V1/foggylineSliderSlide/search`, 调用 `getList` 服务方法，需要 `Foggyline_Slider::slide` 资源

+   `POST /V1/foggylineSliderSlide`, 调用 `save` 服务方法，需要 `Foggyline_Slider::slide_save` 资源

+   `PUT /V1/foggylineSliderSlide/:id`, 调用 `save` 服务方法，需要 `Foggyline_Slider::slide_save` 资源

+   `DELETE /V1/foggylineSliderSlide/:slideId`, 调用 `deleteById` 服务方法，需要 `Foggyline_Slider::slide_delete` 资源

### `getById` 服务方法调用示例

执行 `GET /V1/foggylineSliderSlide/:slideId` 的控制台 cURL 风格如下：

```php
curl -X GET -H 'Content-type: application/json' \
-H 'Authorization: Bearer pk8h93nq9cevaw55bohkjbp0o7kpl4d3' \
http://magento2.ce/rest/V1/foggylineSliderSlide/1

```

执行 `GET /V1/foggylineSliderSlide/:slideId` 的 PHP cURL 风格如下：

```php
$ch = curl_init('http://magento2.ce/rest/V1/foggylineSliderSlide/1');
curl_setopt($ch, CURLOPT_CUSTOMREQUEST, 'GET');
curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
curl_setopt($ch, CURLOPT_HTTPHEADER, array(
    'Content-Type: application/json',
    'Authorization: Bearer pk8h93nq9cevaw55bohkjbp0o7kpl4d3'
));

$result = curl_exec($ch);
```

控制台和 PHP cURL 风格的响应应该是一个类似于以下 JSON 字符串：

```php
{"slide_id":1,"title":"Awesome stuff #1"}
```

执行 `GET /V1/foggylineSliderSlide/:slideId` 的 PHP SoapClient 风格如下：

```php
$request = new SoapClient(
    'http://magento2.ce/index.php/soap/? wsdl&services=foggylineSliderSlideRepositoryV1',
    array(
        'soap_version' => SOAP_1_2,
        'stream_context' => stream_context_create(array(
                'http' => array(
                    'header' => 'Authorization: Bearer pk8h93nq9cevaw55bohkjbp0o7kpl4d3')
            )
        )
    )
);
$response = $request-> foggylineSliderSlideRepositoryV1GetById(array('slideId'=>1));
```

PHP SoapClient 风格的响应应该是如下 `stdClass` PHP 对象：

```php
object(stdClass)#2 (1) {
    ["result"]=>
    object(stdClass)#3 (2) {
    ["slideId"]=>
    int(1)
    ["title"]=>
    string(16) "Awesome stuff #1"
    }
}
```

执行 `GET /V1/foggylineSliderSlide/:slideId` 的控制台 SOAP 风格 cURL 如下：

```php
curl -X POST \
-H 'Content-Type: application/soap+xml; charset=utf-8; action="foggylineSliderSlideRepositoryV1GetById"' \
-H 'Authorization: Bearer pk8h93nq9cevaw55bohkjbp0o7kpl4d3' \
-d @request.xml \
http://magento2.ce/index.php/soap/default?services=foggyline SliderSlideRepositoryV1

```

其中 `request.xml` 的内容如下：

```php
<?xml version="1.0" encoding="UTF-8"?>
<env:Envelope  >
    <env:Body>
        <ns1:foggylineSliderSlideRepositoryV1GetByIdRequest>
            <slideId>1</slideId>
        </ns1:foggylineSliderSlideRepositoryV1GetByIdRequest>
    </env:Body>
</env:Envelope>
```

注意我们实际上并没有做 `GET` 操作，而是做了 `POST` 类型的请求。此外，我们指向的 `POST` 请求的 URL 并非与之前的请求相同。这是因为 Magento SOAP 请求始终是 `POST`（或 `PUT`）类型，因为数据是以 XML 格式提交的。返回的 XML 格式指定了服务，而请求头中的操作指定了要在服务上调用的方法。

控制台 SOAP 风格 cURL 的响应应该是一个如下所示的 XML：

```php
<?xml version="1.0" encoding="UTF-8"?>
<env:Envelope  >
    <env:Body>
        <ns1:foggylineSliderSlideRepositoryV1GetByIdResponse>
            <result>
                <slideId>1</slideId>
                <title>Awesome stuff #1</title>
            </result>
        </ns1:foggylineSliderSlideRepositoryV1GetByIdResponse>
    </env:Body>
</env:Envelope>
```

### 获取列表服务方法调用示例

执行 `GET /V1/foggylineSliderSlide/search` 的控制台 cURL 风格如下：

```php
curl -X GET -H 'Content-type: application/json' \
-H 'Authorization: Bearer pk8h93nq9cevaw55bohkjbp0o7kpl4d3' \
"http://magento2.ce/rest/V1/foggylineSliderSlide/search?search_criteria%5Bfilter_groups%5D%5B0%5D%5Bfilters%5D%5B0%5D%5Bfield%5D=title&search_criteria%5Bfilter_groups%5D%5B0%5D%5Bfilters%5D%5B0%5D%5Bvalue%5D=%25some%25&search_criteria%5Bfilter_groups%5D%5B0%5D%5Bfilters%5D%5B0%5D%5Bcondition_type%5D=like&search_criteria%5Bcurrent_page%5D=1&search_criteria%5Bpage_size%5D=10&search_criteria%5Bsort_orders%5D%5B0%5D%5Bfield%5D=slide_id&search_criteria%5Bsort_orders%5D%5B0%5D%5Bdirection%5D=ASC"

```

执行 `GET /V1/foggylineSliderSlide/search` 的 PHP cURL 风格如下：

```php
$searchCriteriaJSON = '{
  "search_criteria": {
    "filter_groups": [
      {
        "filters": [
          {
            "field": "title",
            "value": "%some%",
            "condition_type": "like"
          }
        ]
      }
    ],
    "current_page": 1,
    "page_size": 10,
    "sort_orders": [
      {
        "field": "slide_id",
        "direction": "ASC"
      }
    ]
  }
}';

$searchCriteriaQueryString = http_build_query(json_decode($searchCriteriaJSON));

$ch = curl_init('http://magento2.ce/rest/V1/foggylineSliderSlide/ search?' . $searchCriteriaQueryString);
  curl_setopt($ch, CURLOPT_CUSTOMREQUEST, 'GET');
  curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
  curl_setopt($ch, CURLOPT_HTTPHEADER, array(
      'Content-Type: application/json',
      'Authorization: Bearer pk8h93nq9cevaw55bohkjbp0o7kpl4d3'
  ));

$result = curl_exec($ch);
```

控制台和 PHP cURL 风格的响应应该是一个类似于以下 JSON 字符串：

```php
{"items":[{"slide_id":2,"title":"Just some other slider"},{"slide_id":1,"title":"Awesome stuff #1"}], "search_criteria":{"filter_groups":[{"filters": [{"field":"title","value":"%some%","condition_type":"like"}]}], "sort_orders":[{"field":"slide_id","direction":"- 1"}],"page_size":10,"current_page":1},"total_count":2}
```

执行 `GET /V1/foggylineSliderSlide/search` 的 PHP SoapClient 风格如下：

```php
$searchCriteria = [
    'searchCriteria' =>
        [
            'filterGroups' =>
                [
                    [
                        'filters' =>
                            [
                                [
                                    'field' => 'title',
                                    'value' => '%some%',
                                    'condition_type' => 'like',
                                ],
                            ],
                    ],
                ],
            'currentPage' => 1,
            'pageSize' => 10,
            'sort_orders' =>
                [
                    [
                        'field' => 'slide_id',
                        'direction' =>'ASC',
                    ],
                ],
        ],
];

$request = new SoapClient(
    'http://magento2.ce/index.php/soap/?wsdl&services= foggylineSliderSlideRepositoryV1',
    array(
        'soap_version' => SOAP_1_2,
        'trace'=>1,
        'stream_context' => stream_context_create(array(
                'http' => array(
                    'header' => 'Authorization: Bearer pk8h93nq9cevaw55bohkjbp0o7kpl4d3')
            )
        )
    )
);

$response = $request-> foggylineSliderSlideRepositoryV1GetList($searchCriteria);
```

PHP SoapClient 风格的响应应该是如下所示的 `stdClass` PHP 对象：

```php
object(stdClass)#2 (1) {
  ["result"]=>
  object(stdClass)#3 (3) {
    ["items"]=>
    object(stdClass)#4 (0) {
    }
    ["searchCriteria"]=>
    object(stdClass)#5 (3) {
      ["filterGroups"]=>
      object(stdClass)#6 (1) {
        ["item"]=>
        object(stdClass)#7 (1) {
          ["filters"]=>
          object(stdClass)#8 (1) {
            ["item"]=>
            object(stdClass)#9 (2) {
              ["field"]=>
              string(5) "title"
              ["value"]=>
              string(6) "%some%"
            }
          }
        }
      }
      ["pageSize"]=>
      int(10)
      ["currentPage"]=>
      int(1)
    }
    ["totalCount"]=>
    int(0)
  }
}
```

执行 `GET /V1/foggylineSliderSlide/search` 的控制台 SOAP 风格 cURL 如下：

```php
curl -X POST \
-H 'Content-Type: application/soap+xml; charset=utf-8; action="foggylineSliderSlideRepositoryV1GetList"' \
-H 'Authorization: Bearer pk8h93nq9cevaw55bohkjbp0o7kpl4d3' \
-d @request.xml \
http://magento2.ce/index.php/soap/default?services=foggyline SliderSlideRepositoryV1

```

其中 `request.xml` 的内容如下：

```php
<?xml version="1.0" encoding="UTF-8"?>
<env:Envelope  >
    <env:Body>
        <ns1:foggylineSliderSlideRepositoryV1GetListRequest>
            <searchCriteria>
                <filterGroups>
                    <item>
                        <filters>
                            <item>
                                <field>title</field>
                                <value>%some%</value>
                            </item>
                        </filters>
                    </item>
                </filterGroups>
                <pageSize>10</pageSize>
                <currentPage>1</currentPage>
            </searchCriteria>
        </ns1:foggylineSliderSlideRepositoryV1GetListRequest>
    </env:Body>
</env:Envelope>
```

注意我们实际上并没有做 `GET` 操作，而是做了 `POST`。此外，我们指向的 `POST` 请求的 URL 并非与之前的请求相同。这是因为 Magento SOAP 请求始终是 `POST` 类型，因为数据是以 XML 格式提交的。返回的 XML 格式指定了服务，而请求头中的操作指定了要在服务上调用的方法。

控制台 SOAP 风格 cURL 的响应应该是一个如下所示的 XML：

```php
<?xml version="1.0" encoding="UTF-8"?>
<env:Envelope  >
    <env:Body>
        <ns1:foggylineSliderSlideRepositoryV1GetListResponse>
            <result>
                <items/>
                <searchCriteria>
                    <filterGroups>
                        <item>
                            <filters>
                                <item>
                                    <field>title</field>
                                    <value>%some%</value>
                                </item>
                            </filters>
                        </item>
                    </filterGroups>
                    <pageSize>10</pageSize>
                    <currentPage>1</currentPage>
                </searchCriteria>
                <totalCount>0</totalCount>
            </result>
        </ns1:foggylineSliderSlideRepositoryV1GetListResponse>
    </env:Body>
</env:Envelope>
```

### 保存（作为新文件）服务方法调用示例

执行 `POST /V1/foggylineSliderSlide` 的控制台 cURL 风格如下：

```php
curl -X POST -H 'Content-type: application/json' \
-H 'Authorization: Bearer pk8h93nq9cevaw55bohkjbp0o7kpl4d3' \
-d '{"slide": {"title": "API test"}}' \
http://magento2.ce/rest/V1/foggylineSliderSlide/

```

执行 `POST /V1/foggylineSliderSlide` 的 PHP cURL 风格如下：

```php
$slide = json_encode(['slide'=>['title'=> 'API test']]);

$ch = curl_init('http://magento2.ce/rest/V1/foggylineSliderSlide');
  curl_setopt($ch, CURLOPT_CUSTOMREQUEST, 'POST');
  curl_setopt($ch, CURLOPT_POSTFIELDS, $slide);
  curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
  curl_setopt($ch, CURLOPT_HTTPHEADER, array(
      'Content-Type: application/json',
      'Content-Length: ' . strlen($slide),
      'Authorization: Bearer pk8h93nq9cevaw55bohkjbp0o7kpl4d3'
  ));

$result = curl_exec($ch);
```

控制台和 PHP cURL 风格的响应应该是一个类似于以下 JSON 字符串：

```php
{"slide_id":4,"title":"API test"}
```

执行 `POST /V1/foggylineSliderSlide` 的 PHP SoapClient 风格如下：

```php
$slide = ['slide'=>['title'=> 'API test']];

$request = new SoapClient(
    'http://magento2.ce/index.php/soap/?wsdl&services= foggylineSliderSlideRepositoryV1',
    array(
        'soap_version' => SOAP_1_2,
        'trace'=>1,
        'stream_context' => stream_context_create(array(
                'http' => array(
                    'header' => 'Authorization: Bearer pk8h93nq9cevaw55bohkjbp0o7kpl4d3')
            )
        )
    )
);

$response = $request-> foggylineSliderSlideRepositoryV1Save($slide);
```

PHP SoapClient 风格的响应应该是如下所示的 `stdClass` PHP 对象：

```php
object(stdClass)#2 (1) {
  ["result"]=>
  object(stdClass)#3 (2) {
    ["slideId"]=>
    int(6)
    ["title"]=>
    string(8) "API test"
  }
}
```

执行 `POST /V1/foggylineSliderSlide` 的控制台 SOAP 风格 cURL 如下：

```php
curl -X POST \
-H 'Content-Type: application/soap+xml; charset=utf-8; action="foggylineSliderSlideRepositoryV1Save"' \
-H 'Authorization: Bearer pk8h93nq9cevaw55bohkjbp0o7kpl4d3' \
-d @request.xml \
http://magento2.ce/index.php/soap/default?services=foggyline SliderSlideRepositoryV1

```

其中 `request.xml` 的内容如下：

```php
<?xml version="1.0" encoding="UTF-8"?>
<env:Envelope  >
    <env:Body>
        <ns1:foggylineSliderSlideRepositoryV1SaveRequest>
            <slide>
                <title>API test</title>
            </slide>
        </ns1:foggylineSliderSlideRepositoryV1SaveRequest>
    </env:Body>
</env:Envelope>
```

控制台 SOAP 风格 cURL 的响应应该是一个如下所示的 XML：

```php
<?xml version="1.0" encoding="UTF-8"?>
<env:Envelope  >
    <env:Body>
        <ns1:foggylineSliderSlideRepositoryV1SaveResponse>
            <result>
                <slideId>8</slideId>
                <title>API test</title>
            </result>
        </ns1:foggylineSliderSlideRepositoryV1SaveResponse>
    </env:Body>
</env:Envelope>
```

### 保存（作为更新）服务方法调用示例

执行 `PUT /V1/foggylineSliderSlide/:id` 的控制台 cURL 风格如下：

```php
curl -X PUT -H 'Content-type: application/json' \
-H 'Authorization: Bearer pk8h93nq9cevaw55bohkjbp0o7kpl4d3' \
-d '{"slide": {"slide_id": 2, "title": "API update test"}}' \
http://magento2.ce/rest/V1/foggylineSliderSlide/2

```

执行 `PUT /V1/foggylineSliderSlide/:id` 的 PHP cURL 风格如下：

```php
$slideId = 2;
$slide = json_encode(['slide'=>['slide_id'=> $slideId, 'title'=> 'API update test']]);

$ch = curl_init('http://magento2.ce/rest/V1/foggylineSliderSlide/' . $slideId);
  curl_setopt($ch, CURLOPT_CUSTOMREQUEST, 'PUT');
  curl_setopt($ch, CURLOPT_POSTFIELDS, $slide);
  curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
  curl_setopt($ch, CURLOPT_HTTPHEADER, array(
      'Content-Type: application/json',
      'Content-Length: ' . strlen($slide),
      'Authorization: Bearer pk8h93nq9cevaw55bohkjbp0o7kpl4d3'
  ));

$result = curl_exec($ch);
```

控制台和 PHP cURL 风格的响应应该是一个类似于以下 JSON 字符串：

```php
{"id":2,"slide_id":2,"title":"API update test"}
```

执行 `PUT /V1/foggylineSliderSlide/:id` 的 PHP SoapClient 风格如下：

```php
$slideId = 2;
$slide = ['slide'=>['slideId'=> $slideId, 'title'=> 'API update test']];

$request = new SoapClient(
    'http://magento2.ce/index.php/soap/?wsdl&services= foggylineSliderSlideRepositoryV1',
    array(
        'soap_version' => SOAP_1_2,
        'trace'=>1,
        'stream_context' => stream_context_create(array(
                'http' => array(
                    'header' => 'Authorization: Bearer pk8h93nq9cevaw55bohkjbp0o7kpl4d3')
            )
        )
    )
);

$response = $request-> foggylineSliderSlideRepositoryV1Save($slide);
```

PHP SoapClient 风格的响应应该是如下所示的 `stdClass` PHP 对象：

```php
object(stdClass)#2 (1) {
  ["result"]=>
  object(stdClass)#3 (2) {
    ["slideId"]=>
    int(2)
    ["title"]=>
    string(15) "API update test"
  }
}
```

执行 `PUT /V1/foggylineSliderSlide/:id` 的控制台 SOAP 风格 cURL 如下：

```php
curl -X PUT \
-H 'Content-Type: application/soap+xml; charset=utf-8; action="foggylineSliderSlideRepositoryV1Save"' \
-H 'Authorization: Bearer pk8h93nq9cevaw55bohkjbp0o7kpl4d3' \
-d @request.xml \
http://magento2.ce/index.php/soap/default?services= foggylineSliderSlideRepositoryV1

```

其中 `request.xml` 的内容如下：

```php
<?xml version="1.0" encoding="UTF-8"?>
<env:Envelope  >
    <env:Body>
        <ns1:foggylineSliderSlideRepositoryV1SaveRequest>
            <slide>
                <slideId>2</slideId>
                <title>API update test</title>
            </slide>
        </ns1:foggylineSliderSlideRepositoryV1SaveRequest>
    </env:Body>
</env:Envelope>
```

控制台 SOAP 风格 cURL 的响应应该是一个如下所示的 XML：

```php
<?xml version="1.0" encoding="UTF-8"?>
<env:Envelope  >
    <env:Body>
        <ns1:foggylineSliderSlideRepositoryV1SaveResponse>
            <result>
                <slideId>2</slideId>
                <title>API update test</title>
            </result>
        </ns1:foggylineSliderSlideRepositoryV1SaveResponse>
    </env:Body>
</env:Envelope>
```

### `deleteById` 服务方法调用示例

执行 `DELETE /V1/foggylineSliderSlide/:slideId` 的控制台 cURL 风格如下：

```php
curl -X DELETE -H 'Content-type: application/json' \
-H 'Authorization: Bearer pk8h93nq9cevaw55bohkjbp0o7kpl4d3' \
http://magento2.ce/rest/V1/foggylineSliderSlide/3

```

执行 `DELETE /V1/foggylineSliderSlide/:slideId` 的 PHP cURL 风格如下：

```php
$slideId = 4;

$ch = curl_init('http://magento2.ce/rest/V1/foggylineSliderSlide/' . $slideId);
  curl_setopt($ch, CURLOPT_CUSTOMREQUEST, 'DELETE');
  curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
  curl_setopt($ch, CURLOPT_HTTPHEADER, array(
      'Content-Type: application/json',
      'Authorization: Bearer pk8h93nq9cevaw55bohkjbp0o7kpl4d3'
  ));

$result = curl_exec($ch);
```

控制台和 PHP cURL 风格的响应应该是一个类似于以下 JSON 字符串的字符串：

```php
true
```

执行 `DELETE /V1/foggylineSliderSlide/:slideId` 的 PHP SoapClient 风格如下：

```php
$slideId = 2;

$request = new SoapClient(
    'http://magento2.ce/index.php/soap/?wsdl&services= foggylineSliderSlideRepositoryV1',
    array(
        'soap_version' => SOAP_1_2,
        'trace'=>1,
        'stream_context' => stream_context_create(array(
                'http' => array(
                    'header' => 'Authorization: Bearer pk8h93nq9cevaw55bohkjbp0o7kpl4d3')
            )
        )
    )
);

$response = $request-> foggylineSliderSlideRepositoryV1DeleteById(array('slideId'=> $slideId));
```

PHP SoapClient 风格的响应应该是类似于以下 `stdClass` PHP 对象：

```php
object(stdClass)#2 (1) {
  ["result"]=>
  bool(true)
}
```

执行 `DELETE /V1/foggylineSliderSlide/:slideId` 的控制台 SOAP 风格 cURL 如下：

```php
curl -X POST \
-H 'Content-Type: application/soap+xml; charset=utf-8; action="foggylineSliderSlideRepositoryV1DeleteById"' \
-H 'Authorization: Bearer pk8h93nq9cevaw55bohkjbp0o7kpl4d3' \
-d @request.xml \
http://magento2.ce/index.php/soap/default?services= foggylineSliderSlideRepositoryV1

```

其中 `request.xml` 的内容如下：

```php
<?xml version="1.0" encoding="UTF-8"?>
<env:Envelope  >
    <env:Body>
        <ns1:foggylineSliderSlideRepositoryV1DeleteByIdRequest>
            <slideId>5</slideId>
        </ns1:foggylineSliderSlideRepositoryV1DeleteByIdRequest>
    </env:Body>
</env:Envelope>
```

控制台 SOAP 风格 cURL 的响应应该是一个类似于以下 XML 的字符串：

```php
<?xml version="1.0" encoding="UTF-8"?>
<env:Envelope  >
    <env:Body>
        <ns1:foggylineSliderSlideRepositoryV1DeleteByIdResponse>
            <result>true</result>
        </ns1:foggylineSliderSlideRepositoryV1DeleteByIdResponse>
    </env:Body>
</env:Envelope>
```

上述 API 调用示例涵盖了我们对 `Slide` 实体自定义定义的所有 API。

回顾一下 `$searchCriteria` 变量，我们使用了 `GET` 类型的 HTTP 方法，将整个变量作为查询字符串传递。如果我们这样考虑，我们可以在 Web API 资源定义期间指定 `POST`，并将 `$searchCriteria` 变量的内容打包到请求体中。尽管 `GET` 方法的方法可能看起来有点脏，但想象一下如果我们为资源分配了匿名或自角色：我们就可以简单地打开一个长 URL 并获取搜索结果。考虑一个可能的小部件用途，其中小部件会简单地向 URL 发送 AJAX 请求并获取访客或客户的搜索结果。

完整的模块源代码可以在以下位置找到：[`github.com/ajzele/B05032-Foggyline_Slider`](https://github.com/ajzele/B05032-Foggyline_Slider)。除了 `Slide` 实体外，完整的模块代码还包括 `Image` 实体。由于每个幻灯片可以包含多个图片，我们可以进一步测试类似于前面调用的 `Image` API 调用。

# 列表过滤的搜索条件接口

了解如何进行适当的列表过滤以获取匹配特定查找条件的实体对于有效使用核心 Magento 的 `getList` 服务以及可能的自定义编码 API 是至关重要的。例如，获取过去 24 小时内注册的客户列表以获取最新添加的产品。

让我们回顾一下 `app/code/Foggyline/Slider/etc/webapi.xml` 文件，其中我们定义了服务 `method="getList"`。服务类定义为 `Foggyline\Slider\Api\SlideRepositoryInterface`，它被定义为 `Foggyline\Slider\Model\SlideRepository` 类的偏好。最后，在 `SlideRepository` 类中，我们有实际的 `getList` 方法。`getList` 方法定义如下：

```php
getList(\Magento\Framework\Api\SearchCriteriaInterface $searchCriteria);
```

我们可以看到，`getList` 方法只接受一个参数，即符合 `SearchCriteriaInterface` 的对象实例，称为 `$searchCriteria`。

这意味着我们已经有以下类型的（不完整）JSON 对象来传递给 `getList` 方法：

```php
{
  "search_criteria": {
  }
}
```

为了进一步理解`search_criteria`的内部工作原理，我们需要了解`SearchCriteriaInterface`，它部分定义如下：

```php
interface SearchCriteriaInterface
{
    /* @param \Magento\Framework\Api\Search\FilterGroup[] $filterGroups */
    public function setFilterGroups(array $filterGroups = null);

    /* @param \Magento\Framework\Api\SortOrder[] $sortOrders */
    public function setSortOrders(array $sortOrders = null);

    /* @param int $pageSize */
    public function setPageSize($pageSize);

    /* @param int $currentPage */
    public function setCurrentPage($currentPage);
}
```

每个接口的获取器和设置器方法都期望在传递的 API 参数中找到值。这意味着`getPageSize()`和`setPageSize()`方法会期望`search_criteria`有一个整型`page_size`属性。同样，`getFilterGroups()`和`setFilterGroups()`方法会期望传递给它们的`search_criteria`有一个`\Magento\Framework\Api\Search\FilterGroup`数组。这些见解使我们达到了传递给`getList`方法的以下（不完整）JSON 对象类型：

```php
{
  "search_criteria": {
    "filter_groups": [
    ],
    "current_page": 1,
    "page_size": 10,
    "sort_orders": [
    ]
  }
}
```

现在我们已经到了需要确定`filter_groups`和`sort_orders`包含什么内容的时候了，因为这些不是简单类型，而是复合值。

进一步查看`\Magento\Framework\Api\Search\FilterGroup`，我们看到`getFilters()`和`setFilters()`方法的定义，这些方法与`\Magento\Framework\Api\Filter`对象数组一起工作。这意味着`filter_groups`有一个属性过滤器，它是一个由`\Magento\Framework\Api\Filter`定义的单独过滤器对象的数组。考虑到这一点，我们现在来看`search_criteria` JSON 对象的以下形式：

```php
{
  "search_criteria": {
    "filter_groups": [
      {
        "filters": [
        ]
      }
    ],
    "current_page": 1,
    "page_size": 10,
    "sort_orders": [
    ]
  }
}
```

进一步查看单个`\Magento\Framework\Api\Filter`，通过其获取器和设置器定义，我们可以得出`field`、`value`和`condition_type`等属性。这使我们进一步接近最终确定我们的`search_criteria` JSON 对象，现在它的结构如下：

```php
{
  "search_criteria": {
    "filter_groups": [
      {
        "filters": [
          {
            "field": "title",
            "value": "%some%",
            "condition_type": "like"
          }
        ]
      }
    ],
    "current_page": 1,
    "page_size": 10,
    "sort_orders": [
    ]
  }
}
```

让我们来看看最后的`sort_orders`。`sort_orders`是`\Magento\Framework\Api\SortOrder`类型，它为字段和方向属性提供了获取器和设置器。了解这一点后，我们能够完全构建我们的`search_criteria` JSON 对象（或数组），这是我们传递给`getList()`服务方法调用的，如下所示：

```php
{
  "search_criteria": {
    "filter_groups": [
      {
        "filters": [
          {
            "field": "title",
            "value": "%some%",
            "condition_type": "like"
          }
        ]
      }
    ],
    "current_page": 1,
    "page_size": 10,
    "sort_orders": [
      {
        "field": "slide_id",
        "direction": -1
      }
    ]
  }
}
```

当我们在`filter_groups`、`filters`或`sort_orders`下定义多个条目时会发生什么？逻辑预期是，当它们到达数据库时，这些会分解成`AND`和`OR`操作符。令人惊讶的是，这并不总是情况，至少在我们的先前列表中不是这样。由于`getList`方法的实际实现留给我们处理，我们可以决定我们想要如何处理过滤器组和过滤器。

回顾我们的`getList`方法，如以下部分所示，我们没有做任何暗示使用`OR`操作符的事情，所以所有内容最终都在数据库上以`AND`条件结束：

```php
foreach ($searchCriteria->getFilterGroups() as $filterGroup) {
    foreach ($filterGroup->getFilters() as $filter) {
        $condition = $filter->getConditionType() ?: 'eq';
        $collection->addFieldToFilter($filter->getField(), [$condition => $filter->getValue()]);
  }
}
```

之前的代码只是简单地遍历所有过滤器组，将组内的所有过滤器拉入，并对所有内容调用相同的 `addFieldToFilter` 方法。类似的行为在核心 Magento 模块中得到了实现。尽管过滤本身遵循 `\Magento\Framework\Api\SearchCriteriaInterface` 接口，但在 Magento 范围内没有统一的处理方法来强制执行过滤中的 `AND` 和 `OR` 操作符。

然而，像 `GET` 产品这样的 Magento 核心 API 确实实现了 `AND` 和 `OR` 条件。在这些情况下，过滤器组导致 `OR` 条件，而组内的过滤器导致 `AND` 条件。

### 小贴士

遵循最佳实践，我们应该确保我们的模块在实现搜索条件时，尊重 `filter_groups`/`filters` 和 `OR`/`AND` 的关系。

# 摘要

在本章中，我们涵盖了与 Magento API 相关的许多内容。还有很多话要说，但这里概述的步骤应该足以让我们开始使用更高级的 API。我们以了解用户类型和支持的认证方法开始本章。我们特别强调了进行多种类型的 API 调用，如控制台 cURL、PHP cURL、PHP SoapClient 和控制台 cURL SOAP。这是为了鼓励开发者更深入地理解 API 调用的内部工作原理，而不仅仅是使用高级库。

在下一章中，我们将探讨 Magento 的一些主要部分。
