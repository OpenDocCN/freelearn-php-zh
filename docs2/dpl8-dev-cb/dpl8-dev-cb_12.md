# Web 服务

Drupal 8 随带 RESTful Web 服务器功能，以实现与您的应用程序交互的 Web 服务。本章将向您展示如何启用这些功能并构建您的 API，涵盖以下主题：

+   启用 RESTful 接口

+   使用 POST 创建数据

+   使用 PATCH 更新数据

+   使用 Views 提供自定义数据源

+   身份验证

+   使用 JSON API

# 简介

Drupal 8 提供了几个模块，这些模块使您能够将其转变为 Web 服务提供者。序列化模块提供了一种将数据序列化到或从 JSON 和 XML 等格式反序列化的方法。然后，RESTful Web 服务模块通过 Web API 暴露实体和其他 API。通过 RESTful 资源端点执行的操作使用与在非 API 格式中相同的创建、编辑、删除和查看权限。

HAL 模块使用 **Hypertext Application Language**（**HAL**）格式序列化实体。HAL 是一个用于在 API 中超链接资源之间的 Internet Draft 标准约定。当使用 POST 和 PATCH 方法时，需要 HAL+JSON。对于身份验证，HTTP Basic Authentication 模块通过 HTTP 头提供简单的身份验证。

有一个社区主导的努力，使用 JSON API 模块在 Drupal 中实现 JSON API 规范，如本章“使用 JSON API”菜谱中所述。像 HAL 一样，它不仅提供了关于数据应该如何表示的规范，还提供了关于如何通过请求参数进行排序和过滤的规范。

本章介绍了如何使用 RESTful Web 服务模块及其支持模块开发由 Drupal 8 驱动的 RESTful API。我们将介绍如何使用 GET、POST 和 PATCH HTTP 方法来操纵网站上的内容。此外，我们还将介绍如何使用视图提供列出端点的自定义内容。最后，我们将介绍如何处理 API 的自定义身份验证。

在一篇名为“推迟 PUT”的文章中，Web 服务倡议背后的团队选择不实现 PUT，仅支持 PATCH。更多信息，请参阅原文档[`groups.drupal.org/node/284948`](https://groups.drupal.org/node/284948)。

然而，API 对贡献模块开放，以添加对核心资源或其自己的 PUT 支持。

# 启用 RESTful 接口

RESTful Web 服务模块提供了暴露 RESTful API 端点的路由。它使用序列化模块来处理响应的规范化以及从请求中数据的反规范化。端点支持特定的格式和身份验证提供者。在安装后，RESTful Web 服务模块不提供任何默认配置的端点。

有一个注意事项：RESTful Web 服务不提供用户界面来配置可用的端点。通过手动编辑配置或 **REST UI** 模块可以启用资源端点。我们将在这个菜谱中使用 REST UI 模块。

在这个菜谱中，我们将安装 RESTful Web 服务并启用适当的权限，以便通过 REST 检索节点并接收格式化的 JSON。

我们将在后面的菜谱中介绍如何使用 GET、POST、PATCH 和 DELETE。本菜谱涵盖了安装和配置基础模块以启用 Web 服务。

# 准备工作

如果您正在运行 PHP 5.6，可能需要进行配置更改：`always_populate_raw_post_data`设置。如果您尝试启用 RESTful Web 服务模块而不更改默认设置，则在安装时将看到以下错误消息：

在 PHP 5.6 版本中，`always_populate_raw_post_data` PHP 设置应设置为`-1`。请查阅 PHP 手册了解如何纠正此设置。（当前使用的`always_populate_raw_post_data` PHP 设置版本未设置为-1。）

# 如何操作...

1.  首先，我们必须将 REST UI 模块添加到我们的 Drupal 站点，这样我们就可以轻松地配置我们的端点：

```php
cd /path/to/drupal8
composer require drupal/restui
```

1.  从管理工具栏转到扩展，并安装以下 Web 服务模块：序列化、RESTful Web 服务以及 REST UI：

![](img/9b148a49-0009-4a45-a9af-06bfc1bd82cf.png)

1.  前往配置，然后在 Web 服务下点击 REST 以配置可用的端点。

1.  点击内容行的“启用”按钮：

![](img/ef337a85-00c7-4438-8538-5f2c8c8cd21c.png)

1.  端点启用后，必须进行配置。勾选 GET 方法复选框以允许 GET 请求。然后，勾选 json 复选框以便数据可以以 JSON 格式返回。所有端点都需要选择一个认证提供者。勾选 cookie 复选框，然后保存：

![](img/03616b74-74d3-45fe-9a79-ac8297eae48a.png)

1.  启用了任何 RESTful 资源端点都将使用已经为实体类型配置的相同创建、更新、删除和查看权限。为了允许匿名用户通过 GET 访问内容，请确保匿名用户有**查看已发布内容权限**。

1.  使用命令行上的 cURL，现在可以使用 RESTful 端点检索内容。您必须传递`?_format=json`以确保返回正确的格式：

```php
curl http://127.0.0.1:8888/node/1?_format=json

{"nid":[{"value":1}],"uuid":[{"value":"9a473f09-fa61-42c9-b4ad-f24b857d04f6"}],"vid":[{"value":51}],"langcode":[{"value":"en"}],"type":[{"target_id":"page","target_type":"node_type","target_uuid":"8a8ad160-69dc-453f-bc11-86775040465e"}],"status":[{"value":true}],"title":[{"value":"Example node"}],"uid":[{"target_id":0,"target_type":"user","target_uuid":"e31b3de2-2195-48c6-9a5e-ab0553461c93","url":"\/user\/0"}],"created":[{"value":1500650071}],"changed":[{"value":1500650374}],"promote":[{"value":true}],"sticky":[{"value":false}],"revision_timestamp":[{"value":1500650374}],"revision_uid":[{"target_id":1,"target_type":"user","target_uuid":"2d7ee3ef-6f8a-4feb-a99a-4af8cfd24402","url":"\/user\/1"}],"revision_log":[],"revision_translation_affected":[{"value":true}],"default_langcode":[{"value":true}],"path":[],"body":[{"value":"Defui dolor elit jus luptatum. Ad augue causa hos loquor luctus minim singularis sino utinam. ","format":"plain_text","summary":""}]}
```

RESTful Web 服务模块将为每个用户可以查看的字段返回整个实体对象。

# 它是如何工作的...

**RESTful Web 服务**模块通过实现一个事件订阅者服务`rest.resource_routes`来工作，该服务根据其`RestResource`插件的实现向 Drupal 添加路由。每个插件根据为资源启用的 HTTP 方法返回可用的路由。

当构建路由时，`\Drupal\rest\Routing\ResourceRoutes`类使用`RestResource`插件管理器检索所有可用的定义。加载并检查端点配置对象。如果资源插件提供了一个在配置定义中启用的 HTTP 方法，它将开始构建一个新的路由。验证是对定义的支持格式和支持认证定义进行的。如果基本验证通过，新的路由将被添加到`RouteCollection`并返回。

如果你提供了一个不可用的`supported_formats`或`supported_auth`值，端点仍然会被创建。然而，如果你尝试使用无效插件的路由，将会出现错误。在使用 REST UI 模块时，这种情况不会发生，但需要手动提供和管理配置。

基类为资源插件提供的默认路由，`\Drupal\rest\Plugin\ResourceBase`类，将`\Drupal\rest\RequestHandler::handle`设置为路由的控制器和方法。此方法将传递的 _format 参数与配置的插件进行比较。如果格式有效，数据将被传递到适当的序列化器。然后，序列化数据将带有适当的内容头返回到请求中。

# 还有更多...

RESTful Web Services 模块提供了一个强大的 API，有一些额外的项目需要注意。我们将在下一个菜谱中探讨这些内容。

# 使用 _format 代替 Accept 头

在 Drupal 8 的生命周期早期，直到 8.0.0-beta12，Drupal 支持使用`Accept`头而不是 _format 参数。不幸的是，存在外部缓存的问题。Drupal 在相同的路径上提供 HTML 和其他格式，只使用不同的`Accept`头。CDN 和反向代理不会仅基于此头无效化缓存。防止这些外部缓存（如 Varnish）缓存中毒的唯一解决方案是确保实现`Vary: Accept`头。然而，关于 CDN 和实现差异的问题太多，因此引入了 _format 参数，而不是将扩展（`.json`和`.xml`）附加到路径上。

关于问题的详细情况可以在以下核心问题中找到：

+   请参考[`www.drupal.org/node/2364011`](https://www.drupal.org/node/2364011)中关于在 URL 上使用内容协商时外部缓存混淆响应格式的说明。

+   检查如何实现基于查询参数的内容协商，作为扩展的替代方案，请参阅[`www.drupal.org/node/2481453`](https://www.drupal.org/node/2481453)。

# RestResource 插件通过 RESTful Web Services 公开数据

RESTful Web Services 模块定义了一个 `RestResource` 插件。此插件用于定义资源端点。它们在模块的 `Plugin/rest/resource` 命名空间中找到，并需要实现 `\Drupal\rest\Plugin\ResourceInterface` 接口。Drupal 8 提供了两个 `RestResource` 插件的实现。第一个是由 RESTful Web Services 模块提供的 `EntityResource` 类。它实现了一个驱动类，允许它表示每种实体类型。第二个是提供自己的 `RestResource` 插件的 **Database Logging** 模块。它允许您通过 ID 检索记录的消息。`\Drupal\rest\Plugin\ResourceBase` 类提供了一个抽象基类，可以扩展用于 `RestResource` 插件实现。如果子类提供了一个与可用 HTTP 方法匹配的方法，它将支持这些方法。例如，如果一个类只有一个 GET 方法，您只能通过 HTTP `GET` 请求与该端点交互。另一方面，您可以提供一个跟踪方法，允许端点支持 HTTP `TRACE` 请求。

Drupal 8 提供了两个 `RestResource` 插件的实现。第一个是由 RESTful Web Services 模块提供的 `EntityResource` 类。它实现了一个 `deriver` 类，允许它表示每种实体类型。第二个是提供自己的 `RestResource` 插件的 Database Logging 模块。它允许您通过 ID 检索记录的消息。

# 限制您的 API 速率

许多 API 实现了速率限制，以防止滥用公共 API。当您公开暴露 API 时，您将需要控制击中服务的流量量，并防止滥用者减慢或停止您的服务。

**速率限制器** 模块实现了多种控制对您的公共 API 访问的方式。有一个选项可以控制特定请求的速率限制，基于 IP 地址的限制，以及 IP 白名单。

您可以在 [`www.drupal.org/project/rate_limiter`](https://www.drupal.org/project/rate_limiter) 找到 Rate Limiter 模块。

# 使用 HAL 格式

当安装 HAL 模块时，它可以格式化返回的实体，以提供到相关实体的链接，例如用户、修订版或任何其他实体引用字段。当 HAL 模块安装后，您可以将其添加为支持的格式，然后使用 `_format=hal_json` 进行请求。来自菜谱的响应将返回一个 `_links` 参数：

```php
   "_links" : {
      "http://127.0.0.1:8888/rest/relation/node/page/revision_uid" : [
         {
            "href" : "http://127.0.0.1:8888/user/1?_format=hal_json"
         }
      ],
      "self" : {
         "href" : "http://127.0.0.1:8888/node/1?_format=hal_json"
      },
      "http://127.0.0.1:8888/rest/relation/node/page/uid" : [
         {
            "lang" : "en",
            "href" : "http://127.0.0.1:8888/user/0?_format=hal_json"
         }
      ],
      "type" : {
         "href" : "http://127.0.0.1:8888/rest/type/node/page"
      }
   },
```

# 参见

+   请参阅 Drupal.org 上的 RESTful Web Services 模块文档，[`www.drupal.org/documentation/modules/rest`](https://www.drupal.org/documentation/modules/rest)

+   请参阅*变更记录：基于接受头的路由被查询参数替换*，[`www.drupal.org/node/2501221`](https://www.drupal.org/node/2501221)

+   参阅*第七章*，*插件即插即用* 

+   请参阅 *速率限制器模块*：[`www.drupal.org/project/rate_limiter`](https://www.drupal.org/project/rate_limiter)

+   请参阅 *REST UI 模块*：[`www.drupal.org/project/restui`](https://www.drupal.org/project/restui)

# 使用 POST 创建数据

当与 RESTful Web 服务一起工作时，使用 HTTP POST 方法来创建新实体。我们将使用 **HTTP Basic Authentication** 来验证用户并创建新节点。

在本配方中，我们将使用公开的节点端点通过 RESTful Web 服务模块创建新的文章内容。我们将使用 `json` 格式。在 *更多内容...* 部分中，我们将讨论如何使用 HAL 模块来处理 `hal_json` 格式。

# 准备工作

您将使用标准安装提供的 `Article` 内容类型。按照前面的配方，*启用 RESTful 接口*，您应该已经使用 Composer 将 **REST UI** 模块添加到您的 Drupal 安装中。这可以通过以下命令完成：

```php
cd /path/to/drupal8
composer require drupal/restui
```

在本配方中，Drupal 8 安装可通过 `http://127.0.0.1:8888` 访问。使用您 Drupal 8 网站的适当 URL。

# 如何操作...

1.  从管理工具栏转到“扩展”，并安装以下 Web 服务模块：序列化、RESTful Web 服务、REST UI 和 HTTP Basic Authentication：

![](img/34895758-9c1f-4d70-bc3f-29fb29064b88.png)

1.  前往“配置”，然后在“Web 服务”下点击 REST 来配置可用的端点。

1.  点击内容行的“启用”按钮：

![](img/6eef4dca-6ac0-4c91-9f30-ad1896460e74.png)

1.  启用端点后，必须进行配置。勾选 GET 和 POST 方法复选框以允许 GET 和 POST 请求。然后，勾选 json 复选框以便数据可以以 JSON 格式返回。勾选 basic_auth 复选框，然后保存：

![](img/1280dbfe-c1e5-45fe-9e22-979038943da6.png)

1.  我们创建 JSON 负载以匹配 Drupal 预期的字段结构：

```php
{
  "type": "article",
  "status": {"value": true},
  "title": {"value": "Testing via REST!"},
  "body": {"value": "This article was created using a RESTful endpoint"}
}
```

1.  在我们发送 JSON 负载之前，我们需要检索 CSRF 令牌。我们通过向 `/session/token` 发送 `GET` 请求来完成此操作。我们将在 POST 请求头中使用返回的值：

```php
curl -X GET http://127.0.0.1:8888/session/token
```

1.  我们可以通过向 `/entity/node?_format=json` 端点路径发送包含我们正文负载的请求来创建我们的节点。确保您传递有效的用户登录信息，其中 `admin:admin` 被使用：

```php
curl -X POST \
  'http://127.0.0.1:8888/entity/node?_format=json' \
  -u admin:admin \
  -H 'content-type: application/json' \
  -H 'x-csrf-token: K5UW756_nWJxjX8Lt5NXXrE0xYSAqCn8MPKLbgE6Gps' \
  -d '{
  "type": "article",
  "status": {"value": true},
  "title": {"value": "Testing via REST!"},
  "body": {"value": "This article was created using a RESTful endpoint"}
}
'
```

1.  成功的请求将返回 `201` 状态码和创建的节点的完整值，包括其标识符。

1.  通过访问 `/node/{nid}`，使用请求响应中的节点 ID 来查看您的 Drupal 网站，并验证节点是否已创建：

![](img/164085a1-8cd5-4816-b439-3153931668e2.png)

# 它是如何工作的...

当使用内容实体和 POST 方法时，端点与用于 GET 请求的端点不同。`\\Drupal\\rest\\Plugin\\rest\\resource\\EntityResource` 类扩展了 `\\Drupal\\rest\\Plugin\\ResourceBase` 基类，该类提供了一个路由方法。如果一个资源插件提供了一个 `https://www.drupal.org/link-relations/create` 链接模板，那么将使用该路径作为 POST 路径。

`EntityResource` 类将 `/entity/{entity_type}` 定义为创建链接模板。然后它覆盖 `getBaseRoute` 方法以确保 `entity_type` 参数从定义中正确填充。

`EntityResource` 类将为请求运行一系列条件。首先，它将通过检查实体是否为 `null` 来验证 POST 请求。然后，如果当前用户也有权编辑提供的所有字段，则当前用户有权创建实体类型。最后，它检查是否传递了标识符。最后一个条件很重要，因为更新只能通过 PATCH 请求进行。

如果实体经过验证，它将被保存。在成功保存后，将返回一个空的 HTTP 201 响应。

# 还有更多...

使用 POST 请求需要一些特定的格式，这些格式将在下一个菜谱中解释。

# 使用 HAL 和理解 _links 需求

当使用 HAL 模块和 `hal_json` 格式时，你必须为实体提供关系。这是通过请求中的 `_links` 参数完成的。这是为了确保实体被正确创建，并具有它所需的任何关系，例如内容实体包的实体类型。另一个例子是在 RESTful 接口上创建评论。你需要为拥有评论的用户提供一个 `_links` 条目。

`rest.link_manager` 服务使用 `rest.link_manager.type` 和 `rest.link_manager.relation`，并负责返回类型和关系的 URI。默认情况下，一个包将有一个类似于 `/rest/type/{entity_type}/{bundle}` 的路径，其关系将类似于 `/rest/relation/{entity_type}/{bundle}/{field_name}`。

以用户引用为例，我们必须填充一个 `uid` 字段，如下所示：

```php
{
  "_links": {
    "type": {
      "href": "http://127.0.0.1:8888/rest/type/node/page"
    },
    "http://127.0.0.1:8888/rest/relation/node/article/uid": [
      {
        "href": "http://127.0.0.1:8888/user/1?_format=hal_json",
        "lang": "en"
      }
    ]
  }
}
```

不幸的是，文档很少，了解需要哪些 `_links` 的最佳方式是执行 GET 请求并研究从 HAL JSON 返回的 `_links`。

# 处理图像

大多数 RESTful API 利用文件的 `base64` 编码来支持 POST 操作上传图像。不幸的是，这不在 Drupal 核心中得到支持。尽管有一个 `serializer.normalizer.file_entity.hal` 服务将文件实体序列化为 HAL JSON，但它目前截至 8.3 版本还没有工作，但希望计划在 8.4 版本中实现。

`\\Drupal\\hal\\Normalizer\\FileEntityNormalizer` 类支持反序列化；然而，它不处理 `base64` 并期望二进制数据。

对于这个问题，有一个 Drupal 核心问题，可在 [`www.drupal.org/node/1927648`](https://www.drupal.org/node/1927648) 找到。

# 使用跨站请求伪造令牌

当使用 POST 请求工作时，如果你使用会话 cookie 进行身份验证，你需要传递一个 **跨站请求伪造**（**CSRF**）令牌。当使用会话 cookie 时，需要 X-CSRF-Token 标头以防止意外的 API 请求。

如果你使用 cookie 提供者进行身份验证，你需要从 `/session/token` 路径请求 CSRF 令牌：

```php
curl -X GET http://127.0.0.1:8888/session/token
```

# 参见

+   参考如何序列化文件内容（`base64`）以支持 REST GET/POST/PATCH

    在文件实体 [`www.drupal.org/node/1927648`](https://www.drupal.org/node/1927648)

# 使用 PATCH 更新数据

当使用 RESTful Web 服务时，使用 HTTP PATCH 方法来更新实体。我们将使用 HTTP Basic Authentication 来验证我们的用户并更新一个节点。

在这个菜谱中，我们将使用暴露的节点端点通过 RESTful Web 服务模块创建新的文章内容。

# 准备工作

我们将使用标准安装提供的 `Article` 内容类型。按照 *启用 RESTful 接口* 菜谱，你应该已经使用 Composer 将 REST UI 模块添加到你的 Drupal 安装中。这可以通过以下命令完成：

```php
cd /path/to/drupal8
composer require drupal/restui
```

# 如何操作...

1.  从管理工具栏转到扩展，并安装以下 Web 服务模块：序列化、RESTful Web 服务、REST UI 和 HTTP 基本身份验证：

![](img/855e2758-aa42-405c-bfd2-a9fe8887f66e.png)

1.  前往配置并点击 Web 服务下的 REST 进行配置可用的端点。

1.  点击内容行的“启用”按钮：

![](img/77fd5bab-7b56-491a-b8dc-5875449c4327.png)

1.  端点启用后，必须进行配置。检查 GET、POST 和 PATCH 方法复选框以允许 GET、POST 和 PATCH 请求。然后，检查 json 复选框，以便可以以 JSON 格式发送数据。检查 basic_auth 复选框，然后保存：

![](img/89afd28a-d7a4-41ee-9172-c673fe67a83d.png)

1.  在你的 Drupal 网站上创建一个示例 `Article` 节点，你将通过 REST 端点对其进行修改。确保你记下它的路径。你将在稍后的请求中使用相同的路径（例如，`/node/4`）。这也会显示节点的 ID：

![](img/07ed6ccb-b637-4132-b79f-cc95b5d73dcd.png)

1.  然后，开始构建你的 JSON 负载。你必须提供现有节点标识符 (`nid`) 的值和内容类型 (`type`) 的值。确保你提供的 `nid` 值与你的当前节点相匹配：

```php
{
   "nid" : {
      "value" : 4
   },
   "body" : {
      "value" : "This article was updated using the RESTful API endpoint!"
   },
   "type" : "article"
}
```

1.  在发送 JSON 负载之前，你需要检索 CSRF 令牌。你可以通过针对 `/session/token` 执行 GET 请求来完成此操作。然后，在 POST 请求头中使用返回的值：

```php
curl -X GET http://127.0.0.1:8888/session/token
```

1.  您可以将包含您的正文有效负载的请求发送到 `/node/4?_format=json`，其中 `/node/4` 匹配您想要编辑的节点路径，通过 HTTP PATCH 请求创建我们的节点：

```php
curl -X PATCH \
  'http://127.0.0.1:8888/node/4?_format=json' \
  -u admin:admin \
  -H 'x-csrf-token: MAjbBsIUmzrwHQGNlXxvGMZQJzQCDZbmtecstzbk5UQ' \
  -d '{
  "type": "article",
  "nid": {"value": 4},
  "body": {"value": "This article was updated using the RESTful API endpoint!!"}
}
'
```

1.  成功的请求将返回 `200` 头代码和更新节点的完整值。

1.  通过访问 `/node/{nid}` 来查看您的 Drupal 网站，并验证节点是否已创建，使用请求响应中的节点 ID：

![图片](img/eb4923ad-10ef-4e19-82d5-07f7e583f04d.png)

# 它是如何工作的…

当与内容实体和 PATCH 方法一起工作时，端点与 GET 方法路径相同。检查当前用户的访问权限，以查看他们是否有权更新实体类型以及请求正文中提供的每个提交字段。

提供的每个字段都会在实体上更新，然后进行验证。如果实体经过验证，它将被保存。在成功保存后，将返回包含整个更新实体内容的 HTTP 200 响应。

# 使用视图提供自定义数据源

RESTful Web 服务模块提供了视图插件，允许您在视图上公开数据以供您的 RESTful API 使用。这允许您创建一个具有路径并使用序列化插件输出数据的视图。您可以使用此功能输出实体，例如 JSON、HAL JSON 或 XML，并且它可以与适当的头信息一起发送。

在这个菜谱中，我们将创建一个视图，输出 Drupal 网站的用户，提供他们的用户名、电子邮件和图片（如果提供）。

# 如何做到这一点…

1.  从管理工具栏转到扩展，并安装以下 Web 服务模块：序列化和 RESTful Web 服务：

![图片](img/9ea0b910-a0ed-48c5-9bbd-91dcd7ff8fce.png)

1.  前往结构，然后转到视图。点击添加视图。将视图命名为 **API Users** 并使其显示用户。

1.  检查提供 REST 导出复选框，并输入 `api/users` 路径。这是请求将在这里进行的地方：

![图片](img/5be46af4-ec3b-4ad4-b1ca-6315371b7f4e.png)

1.  点击保存并编辑。

1.  将行插件的格式从实体更改为字段，以便我们可以控制特定的输出。

1.  确保您的视图包含以下用户实体字段：名称、电子邮件和图片。

1.  将用户：名称字段更改为纯文本格式化程序，并且不要将其链接到用户，以便响应不包含任何 HTML。

1.  将用户：图片字段更改为使用 URL 到图片格式化程序，以便只返回 URL 而不是 HTML。

1.  保存您的视图。

1.  通过访问 `/api/users` 访问您的视图；您将收到包含用户信息的 JSON 响应：

```php
[
  {
    "name": "spuvest",
    "mail": "spuvest@example.com",
    "user_picture": "\/sites\/default\/files\/pictures\/2017-07\/generateImage_xIQkfx.jpg"
  },
  {
    "name": "crepathuslus",
    "mail": "crepathuslus@example.com",
    "user_picture": "\/sites\/default\/files\/pictures\/2017-07\/generateImage_eauTko.gif"
  },
  {
    "name": "veradabufrup",
    "mail": "veradabufrup@example.com",
    "user_picture": "\/sites\/default\/files\/pictures\/2017-07\/generateImage_HsEjKW.png"
  }
]
```

# 它是如何工作的

RESTful Web 服务模块提供了显示、行和格式插件，允许您将内容实体导出为序列化格式。REST 导出显示插件允许您指定访问 RESTful 端点的路径，并为请求的格式正确分配 Content-Type 头。

序列化样式是作为 REST 导出显示的唯一支持样式插件提供的。此样式插件仅支持标识为数据显示类型的行插件。它期望从行插件接收原始数据，以便它可以传递给适当的序列化器。

然后，你有使用数据实体或数据字段行插件的选择。它们不是从它们的渲染方法返回渲染数组，而是返回将被序列化为正确格式的原始数据。

使用行插件返回原始格式数据，以及通过样式插件序列化的数据，显示插件将返回通过序列化模块转换为正确格式的响应。

# 还有更多...

视图提供了一种提供特定 RESTful 端点的方式。我们将在下一个菜谱中探索一些额外的功能。

# 控制 JSON 输出中的键名

数据字段行插件允许你配置字段别名。当数据返回到视图时，它将具有 Drupal 的机器名称。这意味着自定义字段看起来可能像`field_my_field`，这可能对消费者来说没有意义。

点击“字段”旁边的设置，你可以在模态表单中设置别名：

![图片](img/886a4cce-5590-41f2-a4d8-8a309ff7fb54.png)

当你提供一个别名时，字段将匹配。例如，`user_picture`可以改为`avatar`，邮件键可以改为`email`：

```php
[{
  "name": "veradabufrup",
  "email": "veradabufrup@example.com",
  "avatar": "\/sites\/default\/files\/pictures\/2017-07\/generateImage_HsEjKW.png"
}]
```

# 控制 RESTful 视图的访问

当你使用视图创建 RESTful 端点时，你不会使用 RESTful Web 服务模块创建的相同权限。你需要在视图中定义路由权限，这样你可以指定特定的角色或权限来请求。

`EntityResource`插件提供的默认 GET 方法不提供列出实体的方式，并允许通过 ID 检索任何实体。使用视图，你可以提供一个实体列表，限制它们到特定的捆绑包。

使用视图，你甚至可以提供一个新端点来检索特定实体。使用上下文过滤器，你可以添加路由参数和过滤器来限制和验证实体 ID。例如，你可能想通过 API 公开文章内容，但不公开页面。

# 认证

使用 RESTful Web 服务模块，我们为端点定义特定的支持认证提供者。Drupal 核心提供了一个 cookie 提供者，它通过有效的 cookie 进行认证，例如你的常规登录体验。然后，还有 HTTP 基本认证模块来支持 HTTP 认证头。

有提供更健壮认证方法的替代方案。使用基于 cookie 的认证，你需要使用 CSRF 令牌来防止未经授权的第三方加载未请求的页面。当你使用 HTTP 认证时，你是在请求头中发送每个请求的密码。

**OAuth**是一个流行且开放的授权框架。它是一种使用令牌而不是密码的正确认证方法。在本食谱中，我们将实现简单 OAuth 模块以提供 GET 和 POST 请求的 OAuth 2.0 认证。

# 准备工作

如果您不熟悉 OAuth 或 OAuth 2.0，它是一个授权标准。OAuth 的实现围绕在 HTTP 头中发送的令牌的使用。有关更多信息，请参阅 OAuth 主页[`oauth.net/`](http://oauth.net/)。

通过遵循*启用 RESTful 接口*食谱，您应该已经使用 Composer 将**REST UI**模块添加到您的 Drupal 安装中。这可以通过以下命令完成：

```php
cd /path/to/drupal8
composer require drupal/restui
```

# 如何操作

1.  首先，我们必须将简单 OAuth 模块添加到我们的 Drupal 站点：

```php
cd /path/to/drupal8
composer require drupal/simple_oauth
```

1.  从管理工具栏转到扩展并安装以下网络服务模块：序列化、RESTful 网络服务、REST UI 和简单 OAuth：

![图片](img/389b25fd-4aba-40de-be34-07c45e9f57bc.png)

1.  前往配置，点击网络服务下的 REST 进行配置可用的端点。

1.  点击内容行的启用按钮：

![图片](img/9976d9f5-9df6-4e82-99ea-43dc82563d88.png)

1.  端点启用后，必须进行配置。勾选 GET 和 POST 方法复选框以允许 GET 和 POST 请求。然后，勾选 json 复选框以便数据可以以 JSON 格式返回。勾选 oauth2 复选框然后保存。

1.  在我们配置简单 OAuth 模块之前，我们必须生成一对密钥来加密 OAuth 令牌。在 Drupal 可访问但通过 Web 服务器不可用的路径中生成这些密钥：

```php
openssl genrsa -out private.key 2048
openssl rsa -in private.key -pubout > public.key
```

1.  生成密钥后，转到配置页面，然后转到简单 OAuth。输入刚刚生成的私有和公共密钥的路径，然后点击保存配置：

![图片](img/b43152d1-044a-4fc9-9d09-d76b01e9b2df.png)

1.  从简单的 OAuth 配置表单中，点击添加客户端。为客户端提供一个标签并选择管理员范围。点击保存以创建客户端。

1.  接下来，我们将通过`/oauth/token`端点生成令牌。您需要您刚刚创建的客户端的 ID。我们必须传递`grant_type`、`client_id`、`username`和`password`。`grant_type`是`password`，`client_id`是创建的客户端的 ID，然后是您希望使用的账户的用户名和密码：

```php
curl -X POST \
  http://127.0.0.1:8888/oauth/token \
  -H 'content-type: application/x-www-form-urlencoded' \
  -d 'grant_type=password&client_id=3ec55f70-18cd-422f-9abd-2223f6ca3636&username=admin&password=admin'
```

在撰写本书时，端点不接受 JSON 正文，而只接受表单编码的值。

1.  响应将包含一个`access_token`属性。这将在进行 API 请求时用作您的令牌。

1.  使用`Authorization: Bearer [token]`头请求 REST 中的节点：

```php
curl -X GET \ 
    'http://127.0.0.1:8888/node/1?_format=json' \
    -H 'accept: application/json' \
    -H 'authorization: Bearer JT9zgBgMEDlk2QIF0ecpZEOcsYC7-x649Bovo83HXQM'
```

# 工作原理

简单 OAuth 模块是使用`League\OAuth2` PHP 库构建的，这是一个 OAuth2 实现的社区事实库。

在典型的身份验证请求中，有一个身份验证管理器，它使用 `authentication_collector` 服务来收集所有标记的身份验证提供者服务器。根据提供者设置的优先级，每个服务被调用以检查它是否适用于当前请求。然后，每个适用的身份验证提供者被调用以查看身份验证是否无效。

对于 RESTful 网络服务模块，过程更为明确。在端点的 `supported_auth` 定义中标识的提供者是唯一经过 `applies` 和 `authenticates` 过程的服务。

# 还有更多...

我们将在下一节中探索更多关于与身份验证提供者和 RESTful 网络服务模块一起工作的信息。

# 身份验证提供者服务

当与 RESTful 网络服务模块端点一起工作时，`supported_auth` 值引用了带有 `authentication_provider` 标签的服务。默认情况下，Drupal 支持 cookie 身份验证。以下代码由 `basic_auth` 模块提供，以支持 HTTP 头部身份验证：

```php
services:
  basic_auth.authentication.basic_auth:
    class: Drupal\basic_auth\Authentication\Provider\BasicAuth
    arguments: ['@config.factory', '@user.auth', '@flood', '@entity.manager']
    tags:
      - { name: authentication_provider, provider_id: 'basic_auth', priority: 100 }
```

可以通过在模块的 `Authentication\Provider` 命名空间中创建一个类并实现 `\Drupal\Core\Authentication\AuthenticationProviderInterface` 接口来创建身份验证提供者。然后，将类作为服务注册到模块的 `services.yml` 文件中。

# 页面缓存请求策略和已验证的 Web 服务请求

当处理期望有已验证用户的数据时，身份验证服务提供者还应提供页面缓存服务处理程序。标记为 `page_cache_request_policy` 的服务有权检查内容是否已缓存。这防止了授权请求被缓存。

以下代码摘自 `basic_auth` 模块：

```php
basic_auth.page_cache_request_policy.disallow_basic_auth_requests:
    class: Drupal\basic_auth\PageCache\DisallowBasicAuthRequests
    public: false
    tags:
      - { name: page_cache_request_policy }
```

`\Drupal\basic_auth\PageCache\DisallowBasicAuthRequests` 类实现了 `\Drupal\Core\PageCache\RequestPolicyInterface` 接口。检查方法允许页面缓存策略明确拒绝或保持中立，以确定页面是否可以缓存。`basic_auth` 模块检查默认身份验证头是否存在。对于 `simple_oauth` 模块，它检查是否存在有效的令牌。

如果您正在实现自己的身份验证服务，这是一个重要的安全措施。

可以通过在模块的 `PageCache` 命名空间中创建一个类并实现 `\Drupal\Core\PageCache\ResponsePolicyInterface` 接口来实现页面缓存策略服务。然后，我们需要将类作为服务注册到模块的 `services.yml` 文件中。

# IP 身份验证提供者

一些实现服务器到服务器通信的 API 将使用 IP 地址白名单进行身份验证。对于此用例，我们有 IP 消费者身份验证模块。白名单 IP 地址由一个保存配置值的表单控制。

如果 IP 地址被列入白名单，用户将被认证为匿名用户。虽然这可能在 POST、PATCH 和 DELETE 请求中不被推荐，但它可以提供一个简单的方法来控制私有网络中的特定 GET 端点。

您可以从其项目页面下载 IP 消费者身份验证：[`www.drupal.org/project/ip_consumer_auth`](https://www.drupal.org/project/ip_consumer_auth)。

# 相关内容

+   请参考 OAuth 社区网站：[`oauth.net/`](http://oauth.net/)

+   请参考支持 OAuth 1.0 的 OAuth 模块：[`www.drupal.org/project/oauth`](https://www.drupal.org/project/oauth)

+   请参考支持 OAuth 2.0 的简单 OAuth 模块：[`www.drupal.org/project/simple_oauth`](https://www.drupal.org/project/simple_oauth)

+   请参考 IP 消费者身份验证模块：[`www.drupal.org/project/ip_consumer_auth`](https://www.drupal.org/project/ip_consumer_auth)

# 使用 JSON API

在为前端消费者开发后端 API 时，关于命名约定和返回值结构经常有许多争议。这时出现了 `{json:api}`，这是一个开源规范，旨在标准化和简化 API 的构建，这些 API 消费和返回 JSON 有效负载。规范和文档可以在 [`jsonapi.org/`](http://jsonapi.org/) 找到。

对于 Drupal，有一个由社区主导的努力，旨在提供强大的 JSON API 规范实现，将 Drupal 转换为一个高效的 API 服务器。本食谱将安装 JSON API 模块并展示如何启用资源。

就像 Drupal 核心提供的 RESTful Web 服务模块一样，JSON API 模块不提供用户界面。它还自动使所有内容可通过 API 获取（假设用户已配置权限以访问端点。）JSON API Extra 模块改变了这一点，这将在本食谱的 *还有更多...* 部分中介绍。

JSON API 模块可以在 [`www.drupal.org/project/jsonapi`](https://www.drupal.org/project/jsonapi) 找到。

# 准备工作

使用标准 Drupal 安装提供的 `Article` 内容类型创建示例内容。这将使测试 GET 方法变得容易得多。

在进行请求时，所有端点路径都以前缀 `jsonapi` 开头。

# 如何操作

1.  首先，我们必须将 JSON API 模块添加到我们的 Drupal 网站中：

```php
cd /path/to/drupal8
composer require drupal/jsonapi
```

1.  安装 JSON API 和序列化模块。一旦模块安装完成，API 端点将处于活动状态：

![图片](img/d3e32ab7-f92b-44c0-9f8d-32120d746e96.png)

1.  执行请求时，必须传递一个值为 `application/vnd.api+json` 的 Accept 头部。

1.  对于 JSON API 规范，每个资源都必须有一个唯一的类型名称，JSON API 从实体类型及其包中推导出此名称。检索文章节点的端点将是：

```php
http://127.0.0.1:8888/jsonapi/node/article
```

1.  整个请求可以使用以下命令执行：

```php
curl -X GET \
  http://127.0.0.1:8888/jsonapi/node/article \
  -H 'accept: application/vnd.api+json'
```

1.  响应将类似于以下内容。内容值将在属性属性中：

```php
{
  "data": [
    {
      "type": "node--article",
      "id": "c897acba-eb81-454a-94ed-13107fd205cf",
      "attributes": {...},
      "relationships": {...},
      "links": {
        "self": "http://127.0.0.1:8888/jsonapi/node/article/c897acba-eb81-454a-94ed-13107fd205cf"
      }
    }
  ],
  "links": {
    "self": "http://127.0.0.1:8888/jsonapi/node/article",
  }
}
```

# 它是如何工作的...

JSON API 模块实现了 `{json:api}` 规范。类似于 Drupal 核心提供的 RESTful Web 服务模块，它通过各种端点公开数据。它建立在 Drupal 现有的路由系统之上，以处理非 HTML 格式。主要区别在于它遵循一个社区驱动的规范，该规范决定了数据应该如何格式化、链接、过滤、排序等。

# 还有更多...

接下来，我们将介绍过滤、分页、排序和 JSON API Extras 模块。

# 分页、过滤和排序请求

菜谱中的请求将返回系统中所有可用的文章节点。这些可以分页、过滤和排序。这些操作都是通过查询参数完成的，其中包含一个值数组。

分页是通过附加一个 `page` 查询参数来完成的。要限制请求为 10 个节点，我们会在后面附加 `?page[limit]=10`。要访问下一组结果，我们也会传递 `page[offset]=10`。

以下是一个返回结果的第一页和第二页的示例：

```php
http://127.0.0.1:8888/jsonapi/node/article?page[limit]=10
http://127.0.0.1:8888/jsonapi/node/article?page[offset]=10&page[limit]=10
```

每个请求都包含一个链接属性；当使用分页结果时，它也将包含下一页和上一页的链接。

通过附加一个 `filter` 查询参数来进行过滤。以下是一个请求所有被提升到首页的节点的示例：

```php
http://127.0.0.1:8888/jsonapi/node/article?filter[promoted][path]=promote&filter[promoted][value]=1&filter[promoted][operator]==
```

每个过滤器都由一个名称定义--在前面的示例中，它是 `promoted`。然后过滤器接受 `path`，这是要过滤的字段。`value` 和 `operator` 决定了如何过滤。

排序是最简单的操作。添加一个排序查询参数。字段名称值是要排序的字段，如果要按降序排序，则在字段名称前添加一个分钟符号。以下示例分别展示了如何按 `nid` 升序和降序排序：

```php
http://127.0.0.1:8888/jsonapi/node/article?sort=nid
http://127.0.0.1:8888/jsonapi/node/article?sort=-nid
```

# 安装 JSON API Extras 模块

JSON API Extras 模块提供了一个用户界面进行额外的自定义。应像所有其他模块一样，使用 Composer 将 JSON API Extras 模块添加到您的 Drupal 安装中：

```php
cd /path/to/drupal8
composer require drupal/jsonapi_extras
```

一旦在 Drupal 中安装了该模块，您将能够启用或禁用端点、更改资源名称、更改资源路径、禁用字段、别名字段名称以及增强字段输出。

# 更改 API 路径前缀

使用 extras 模块，可以将 API 路径前缀从 `jsonapi` 更改为 `api` 或任何其他前缀。

从管理工具栏导航到配置。在 Web 服务部分下，单击 JSON API Overwrites 以自定义 JSON API 实现。设置选项卡允许修改 API 路径前缀：

![](img/4b4ec93f-c4d3-4312-b274-064987cc096c.png)

# 禁用和增强返回的实体字段

JSON API Extras 模块允许覆盖 JSON API 模块自动公开的端点。这允许禁用返回字段。它还允许使用增强器来简化字段属性的架构。

从管理工具栏中，转到配置。在 Web 服务部分下，点击 JSON API 覆盖以自定义 JSON API 实现。

要禁用端点，请点击任何端点的“覆盖”。勾选“禁用”复选框以关闭该特定端点：

![](img/467d2d17-7adc-4448-8020-6802ac5546ba.png)

要禁用、别名或使用增强器，请点击任何端点的“覆盖”。复选框将允许您防止字段在 API 中使用。增强器允许您在返回或用于 POST/PATCH 请求时简化字段：

![](img/de9a7553-27da-4a7e-8fca-182561faa36d.png)

在本例中，`created` 和 `changed` 字段将不再返回 Unix 时间戳，而是返回 RFC ISO8601 格式的时间戳。`promote` 和 `sticky` 字段将直接返回它们的值，而不是嵌套在 `value` 属性下。最后，将不会返回任何修订信息字段。

# Contenta CMS

Contenta CMS 是一个解耦的、基于 API 的 Drupal 分发，使用 JSON API 构建。该项目是通过推动 JSON API 模块向前发展的同一社区倡议构建的。项目主页可在 [`www.contentacms.org/`](http://www.contentacms.org/) 找到。

它提供了许多预配置选项，包括对默认端点的自定义。它还提供了简单的 OAuth，以设置与前端消费者和 API 后端的解耦身份验证。

在提供分发的同时，社区贡献者还开发了各种前端消费者作为示例：

+   **Angular**: [`github.com/contentacms/contenta_angular`](https://github.com/contentacms/contenta_angular)

+   **Vue/Nuxt**: [`github.com/contentacms/contenta_vue_nuxt`](https://github.com/contentacms/contenta_vue_nuxt)

+   **Ember.js**: [`github.com/contentacms/contenta_ember`](https://github.com/contentacms/contenta_ember)

+   **Ionic**: [`github.com/contentacms/contenta_ionic`](https://github.com/contentacms/contenta_ionic)

+   **React**: [`github.com/contentacms/contenta_react`](https://github.com/contentacms/contenta_react)

# 参考以下内容

+   JSON API 项目页面位于 [`www.drupal.org/project/jsonapi`](https://www.drupal.org/project/jsonapi)

+   JSON API Extras 页面位于 [`www.drupal.org/project/jsonapi_extras`](https://www.drupal.org/project/jsonapi_extras)

+   JSON API 模块文档位于 [`www.drupal.org/docs/8/modules/json-api/json-api`](https://www.drupal.org/docs/8/modules/json-api/json-api)

+   JSON API 模块视频教程位于 [`www.youtube.com/playlist?list=PLZOQ_ZMpYrZsyO-3IstImK1okrpfAjuMZ`](https://www.youtube.com/playlist?list=PLZOQ_ZMpYrZsyO-3IstImK1okrpfAjuMZ)

+   `{json:api}` 规范文档位于 [`jsonapi.org/`](http://jsonapi.org/)
