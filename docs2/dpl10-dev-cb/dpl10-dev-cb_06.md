# 6

# 访问和使用实体

在本章中，我们将介绍在 Drupal 中处理实体的**创建、读取、更新和删除**（**CRUD**）操作。我们将创建一系列路由来创建、读取、更新和删除文章节点。

在本章中，我们将涵盖以下内容：

+   创建和保存实体

+   查询和加载实体

+   检查实体访问

+   更新实体的字段值

+   执行实体验证

+   删除实体

# 技术要求

本章需要自定义模块，该模块包含一个`routing.yml`文件，并在模块的`src/Controller`目录中有一个名为`ArticleController`的控制器。在下面的示例中，模块名称为`mymodule`。请根据需要替换。您可以在 GitHub 上找到本章使用的完整代码：[`github.com/PacktPublishing/Drupal-10-Development-Cookbook/tree/main/chp06`](https://github.com/PacktPublishing/Drupal-10-Development-Cookbook/tree/main/chp06)

我们正在使用由标准 Drupal 安装创建的**文章内容**类型。

本章中的示例包含用于与每个示例中创建的代码交互的 HTTP 请求。这些 HTTP 请求可以用任何 HTTP 客户端运行。如果您使用 VSCode，请尝试**REST Client**扩展（[`marketplace.visualstudio.com/items?itemName=humao.rest-client`](https://marketplace.visualstudio.com/items?itemName=humao.rest-client)），或者如果您有 PhpStorm，请使用内置的**HTTP Client**（[`www.jetbrains.com/help/idea/http-client-in-product-code-editor.html`](https://www.jetbrains.com/help/idea/http-client-in-product-code-editor.html)）。如果由于某些原因您没有编辑器或无法使其工作，您可以使用 Postman（[`www.postman.com/`](https://www.postman.com/)）。

# 创建和保存实体

在这个示例中，我们将定义一个路由来创建一个新的文章。该路由将用于发送 JSON 的 HTTP `POST`请求，以指定文章的标题和正文文本。

## 如何操作…

1.  在您的模块中`ArticleController`控制器中创建一个`store`方法，该方法将接收传入的请求对象：

    ```php
    <?php
    ```

    ```php
    namespace Drupal\mymodule\Controller;
    ```

    ```php
    use Drupal\Core\Controller\ControllerBase;
    ```

    ```php
    use Symfony\Component\HttpFoundation\JsonResponse;
    ```

    ```php
    use Symfony\Component\HttpFoundation\Request;
    ```

    ```php
    class ArticleController extends ControllerBase {
    ```

    ```php
      public function store(Request $request):
    ```

    ```php
          JsonResponse {
    ```

    ```php
      }
    ```

    ```php
    }
    ```

我们需要请求对象，以便我们可以检索请求有效载荷中提供的 JSON。

1.  接下来，我们将使用 Drupal 的 JSON 序列化实用工具类将请求的内容从 JSON 转换为 PHP 数组：

    ```php
      public function store(Request $request):
    ```

    ```php
       JsonResponse {
    ```

    ```php
       $content = $request->getContent();
    ```

    ```php
       $json = \Drupal\Component\Serialization\
    ```

    ```php
          Json::decode($content);
    ```

    ```php
      }
    ```

尽管我们可以直接使用`json_decode`函数，但利用 Drupal 标准提供的实用工具类可以标准化代码的工作方式。

1.  然后，我们获取实体类型管理器并检索`node`实体类型的实体存储：

    ```php
      public function store(Request $request):
    ```

    ```php
       JsonResponse {
    ```

    ```php
       $content = $request->getContent();
    ```

    ```php
       $json = \Drupal\Component\Serialization\
    ```

    ```php
          Json::decode($content);
    ```

    ```php
       $entity_type_manager = $this->entityTypeManager();
    ```

    ```php
       $node_storage = $entity_type_manager->
    ```

    ```php
          getStorage('node');
    ```

    ```php
      }
    ```

实体类型管理器是实体类型信息的存储库，用于获取实体类型的处理程序，例如存储处理程序。在获取实体存储处理程序时，您需要传递实体类型 ID。

1.  从存储中调用`create`方法来创建一个新的节点实体对象：

    ```php
      public function store(Request $request):
    ```

    ```php
       JsonResponse {
    ```

    ```php
       $content = $request->getContent();
    ```

    ```php
       $json = \Drupal\Component\Serialization\
    ```

    ```php
          Json::decode($content);
    ```

    ```php
       $entity_type_manager = $this->entityTypeManager();
    ```

    ```php
       $node_storage = $entity_type_manager->
    ```

    ```php
          getStorage('node');
    ```

    ```php
       $article = $node_storage->create([
    ```

    ```php
        'type' => 'article',
    ```

    ```php
       ]);
    ```

    ```php
      }
    ```

`create` 方法使用实体类型类实例化一个新的实体对象。对于节点实体类型，我们的 `$article` 变量将是 `\Drupal\node\Node` 类型。在实例化新实体时，你必须指定其包，我们通过将 `type` 键设置为 `article` 来做到这一点。

1.  向 `create` 方法提供一个 `title` 和 `body` 的关联数组，从我们接收到的请求 JSON 中复制值：

    ```php
      public function store(Request $request): JsonResponse {
    ```

    ```php
       $content = $request->getContent();
    ```

    ```php
       $json = \Drupal\Component\Serialization\
    ```

    ```php
          Json::decode($content);
    ```

    ```php
       $entity_type_manager = $this->entityTypeManager();
    ```

    ```php
       $node_storage = $entity_type_manager->
    ```

    ```php
          getStorage('node');
    ```

    ```php
       $article = $node_storage->create([
    ```

    ```php
        'type' => 'article',
    ```

    ```php
        'title' => $json['title'],
    ```

    ```php
        'body' => $json['body'],
    ```

    ```php
       ]);
    ```

    ```php
      }
    ```

1.  在实体对象上调用 `save` 方法以保存实体：

    ```php
      public function store(Request $request):
    ```

    ```php
        JsonResponse {
    ```

    ```php
       $content = $request->getContent();
    ```

    ```php
       $json = \Drupal\Component\Serialization\
    ```

    ```php
          Json::decode($content);
    ```

    ```php
       $entity_type_manager = $this->entityTypeManager();
    ```

    ```php
       $node_storage = $entity_type_manager->
    ```

    ```php
          getStorage('node');
    ```

    ```php
       $article = $node_storage->create([
    ```

    ```php
        'type' => 'article',
    ```

    ```php
        'title' => $json['title'],
    ```

    ```php
        'body' => $json['body'],
    ```

    ```php
       ]);
    ```

    ```php
       $article->save();
    ```

    ```php
      }
    ```

1.  我们现在将返回一个带有位置头部的响应，该头部包含新创建的节点 URL 和 HTTP 状态码 `201 Created`：

    ```php
      public function store(Request $request):
    ```

    ```php
        JsonResponse {
    ```

    ```php
       $content = $request->getContent();
    ```

    ```php
       $json = \Drupal\Component\Serialization\
    ```

    ```php
          Json::decode($content);
    ```

    ```php
       $entity_type_manager = $this->entityTypeManager();
    ```

    ```php
       $node_storage = $entity_type_manager->
    ```

    ```php
          getStorage('node');
    ```

    ```php
       $article = $node_storage->create([
    ```

    ```php
        'type' => 'article',
    ```

    ```php
        'title' => $json['title'],
    ```

    ```php
        'body' => $json['body'],
    ```

    ```php
       ]);
    ```

    ```php
       $article->save();
    ```

    ```php
       $article_url = $article->toUrl()->setAbsolute()->
    ```

    ```php
          toString();
    ```

    ```php
       return new JsonResponse(
    ```

    ```php
        $article->toArray(),
    ```

    ```php
        201,
    ```

    ```php
        ['Location' => $article_url],
    ```

    ```php
       );
    ```

    ```php
      }
    ```

`201 Created` 状态码用于表示成功的响应，并表明已创建一个项目。当返回 `201 Created` 响应时，建议返回一个带有创建项目 URL 的 `Location` 头部。实体类有一个 `toUrl` 方法来返回一个 URL 对象，然后调用其 `toString` 方法将其从对象转换为字符串。

1.  我们必须在模块的 `routing.yml` 中创建路由，指向 `ArticleController` 的 `store` 方法，以处理对 `/articles` 路径的 HTTP `POST` 请求：

    ```php
    mymodule.create_article:
    ```

    ```php
      path: /articles
    ```

    ```php
      defaults:
    ```

    ```php
       _controller: Drupal\mymodule\Controller\
    ```

    ```php
          ArticleController::store
    ```

    ```php
      methods: [POST]
    ```

    ```php
      requirements:
    ```

    ```php
       _access: 'TRUE'
    ```

路由可以通过 `methods` 键指定它们支持的 HTTP 方法。在这个菜谱中，我们只将 `requirements` 设置为 `_access: 'TRUE'` 以绕过访问检查。这应该设置为 `_entity_create_access: 'node'`。

1.  重建你的 Drupal 网站的缓存，使其了解新的路由：

    ```php
    php vendor/bin/drush cr
    ```

1.  可以使用以下 HTTP 请求来创建一个新节点，该节点是你的 Drupal 网站上的文章：

    ```php
    POST http://localhost/articles
    ```

    ```php
    Content-Type: application/json
    ```

    ```php
    Accept: application/json
    ```

    ```php
    {
    ```

    ```php
      "title": "New article",
    ```

    ```php
      "body": "Test body"
    ```

    ```php
    }
    ```

## 它是如何工作的...

`create` 方法用于实例化一个新的实体对象，并返回该实体类型的类对象。对于这个菜谱，`node` 存储将返回具有 `\Drupal\node\Node` 类的实体。`create` 方法接受的值基于该实体类型的字段定义。这包括基本字段和通过用户界面创建的任何字段。

在创建实体类型时必须指定其实体类型的包。这用于确定可用的字段，因为每个包可以有不同的字段。对于节点，包字段名为 `type`。这就是为什么我们为 `type` 键提供 `article` 值。

然后 `save` 方法将实体提交到数据库存储。

当实体在其首次保存时插入，实体存储触发调用以下钩子，允许你钩入新实体的插入。正在插入的实体作为参数传递：

+   `hook_ENTITY_TYPE_presave`

+   `hook_entity_presave`

+   `hook_ENTITY_TYPE_insert`

+   `hook_entity_insert`

# 查询和加载实体

在这个菜谱中，我们将使用实体查询来查找所有已发布的文章并将它们作为 JSON 返回。我们还将允许通过查询参数指定排序顺序。

## 如何做到这一点…

1.  在你的模块中创建 `ArticleController` 控制器的 `index` 方法，它将接收传入的 `Request` 对象：

    ```php
    <?php
    ```

    ```php
    namespace Drupal\mymodule\Controller;
    ```

    ```php
    use Drupal\Core\Controller\ControllerBase;
    ```

    ```php
    use Symfony\Component\HttpFoundation\JsonResponse;
    ```

    ```php
    use Symfony\Component\HttpFoundation\Request;
    ```

    ```php
    class ArticleController extends ControllerBase {
    ```

    ```php
      public function index(Request $request):
    ```

    ```php
       JsonResponse {
    ```

    ```php
      }
    ```

    ```php
    }
    ```

`request`对象将用于检索通过 URL 传递的查询参数。

1.  从请求中获取排序查询参数，默认为`DESC`：

    ```php
      public function index(Request $request):
    ```

    ```php
        JsonResponse {
    ```

    ```php
       $sort = $request->query->get('sort', 'DESC');
    ```

    ```php
      }
    ```

1.  然后，我们获取实体类型管理器并检索节点实体类型的实体存储：

    ```php
      public function index(Request $request):
    ```

    ```php
        JsonResponse {
    ```

    ```php
       $sort = $request->query->get('sort', 'DESC');
    ```

    ```php
       $entity_type_manager = $this->entityTypeManager();
    ```

    ```php
       $node_storage = $entity_type_manager->
    ```

    ```php
          getStorage('node');
    ```

    ```php
      }
    ```

实体类型管理器是实体类型信息的存储库，用于获取实体类型的处理程序，例如存储处理程序。在获取实体存储处理程序时，您传递实体类型 ID。

1.  从存储处理程序中调用`getQuery`方法。这将返回一个查询对象，用于执行实体查询：

    ```php
      public function index(Request $request):
    ```

    ```php
        JsonResponse {
    ```

    ```php
       $sort = $request->query->get('sort', 'DESC');
    ```

    ```php
       $entity_type_manager = $this->entityTypeManager();
    ```

    ```php
        $node_storage = $entity_type_manager->
    ```

    ```php
            getStorage('node');
    ```

    ```php
       $query = $node_storage->getQuery()
    ```

    ```php
        ->accessCheck(TRUE);
    ```

    ```php
      }
    ```

Drupal 要求指定在查询内容实体时实体查询是否应执行实体访问检查。我们必须调用`accessCheck`方法并将其设置为`TRUE`或`FALSE`。在这种情况下，我们希望实体访问检查应用于实体查询。

1.  我们将在查询中添加条件以确保只返回已发布的文章：

    ```php
      public function index(Request $request):
    ```

    ```php
        JsonResponse {
    ```

    ```php
       $sort = $request->query->get('sort', 'DESC');
    ```

    ```php
       $entity_type_manager = $this->entityTypeManager();
    ```

    ```php
       $node_storage = $entity_type_manager->
    ```

    ```php
          getStorage('node');
    ```

    ```php
       $query = $node_storage->getQuery();
    ```

    ```php
        ->accessCheck(TRUE);
    ```

    ```php
       $query->condition('type', 'article');
    ```

    ```php
       $query->condition('status', TRUE);
    ```

    ```php
      }
    ```

`condition`方法传递字段名和值以创建条件。第一个条件确保我们只查询类型（捆绑）为`article`的节点。第二个条件是确保`status`字段为`true`表示已发布。

1.  然后，我们指定查询的`sort`顺序来自我们的 URL 查询参数：

    ```php
      public function index(Request $request):
    ```

    ```php
        JsonResponse {
    ```

    ```php
       $sort = $request->query->get('sort', 'DESC');
    ```

    ```php
       $entity_type_manager = $this->entityTypeManager();
    ```

    ```php
       $node_storage = $entity_type_manager->
    ```

    ```php
          getStorage('node');
    ```

    ```php
       $query = $node_storage->getQuery();
    ```

    ```php
        ->accessCheck(TRUE);
    ```

    ```php
       $query->condition('type', 'article');
    ```

    ```php
       $query->condition('status', TRUE);
    ```

    ```php
       $query->sort('created', $sort);
    ```

    ```php
      }
    ```

`sort`方法传递用于排序查询的字段名和方向`ASC`或`DESC`。

1.  调用`execute`方法将执行实体查询并返回可用的实体 ID：

    ```php
      public function index(Request $request):
    ```

    ```php
        JsonResponse {
    ```

    ```php
       $sort = $request->query->get('sort', 'DESC');
    ```

    ```php
       $entity_type_manager = $this->entityTypeManager();
    ```

    ```php
       $node_storage = $entity_type_manager->
    ```

    ```php
          getStorage('node');
    ```

    ```php
       $query = $node_storage->getQuery()
    ```

    ```php
        ->accessCheck(TRUE);
    ```

    ```php
       $query->condition('type', 'article');
    ```

    ```php
       $query->condition('status', TRUE);
    ```

    ```php
       $query->sort('created', $sort);
    ```

    ```php
       $node_ids = $query->execute();
    ```

    ```php
      }
    ```

1.  在调用`execute`之后，我们得到一个节点 ID 数组，可以将这些 ID 传递给实体存储中的`loadMultiple`方法来加载节点：

    ```php
      public function index(Request $request):
    ```

    ```php
        JsonResponse {
    ```

    ```php
       $sort = $request->query->get('sort', 'DESC');
    ```

    ```php
       $entity_type_manager = $this->entityTypeManager();
    ```

    ```php
       $node_storage = $entity_type_manager->
    ```

    ```php
          getStorage('node');
    ```

    ```php
       $query = $node_storage->getQuery()
    ```

    ```php
        ->accessCheck(TRUE);
    ```

    ```php
       $query->condition('type', 'article');
    ```

    ```php
       $query->condition('status', TRUE);
    ```

    ```php
       $query->sort('created', $sort);
    ```

    ```php
       $node_ids = $query->execute();
    ```

    ```php
       $nodes = $node_storage->loadMultiple($node_ids);
    ```

    ```php
      }
    ```

1.  我们现在可以使用`array_map`将节点转换为数组值并返回文章的 JSON 响应：

    ```php
      public function index(Request $request):
    ```

    ```php
        JsonResponse {
    ```

    ```php
       $sort = $request->query->get('sort', 'DESC');
    ```

    ```php
       $entity_type_manager = $this->entityTypeManager();
    ```

    ```php
       $node_storage = $entity_type_manager->
    ```

    ```php
          getStorage('node');
    ```

    ```php
       $query = $node_storage->getQuery()
    ```

    ```php
        ->accessCheck(TRUE);
    ```

    ```php
       $query->condition('type', 'article');
    ```

    ```php
       $query->condition('status', TRUE);
    ```

    ```php
       $query->sort('created', $sort);
    ```

    ```php
       $node_ids = $query->execute();
    ```

    ```php
       $nodes = $node_storage->loadMultiple($node_ids);
    ```

    ```php
       $nodes = array_map(function (\Drupal\node\
    ```

    ```php
          NodeInterface $node) {
    ```

    ```php
        return $node->toArray();
    ```

    ```php
       }, $nodes);
    ```

    ```php
       return new JsonResponse($nodes);
    ```

    ```php
      }
    ```

`array_map`函数允许您转换现有数组中项的值。我们将使用`array_map`遍历返回的节点实体并调用`toArray`方法以获取它们的值作为数组。

1.  我们必须在`routing.yml`中创建路由，指向`ArticleController`的`index`方法，以便对`/articles`路径的 HTTP `GET`请求：

    ```php
    mymodule.get_articles:
    ```

    ```php
      path: /articles
    ```

    ```php
      defaults:
    ```

    ```php
       _controller: Drupal\mymodule\Controller\
    ```

    ```php
          ArticleController::index
    ```

    ```php
      methods: [GET]
    ```

    ```php
      requirements:
    ```

    ```php
       _permission: 'access content'
    ```

如果路由明确定义它们支持的 HTTP 方法，则路由可以共享相同的路径。

1.  重建您的 Drupal 站点的缓存，使其了解新的路由：

    ```php
    php vendor/bin/drush cr
    ```

1.  可以使用如下所示的 HTTP 请求来检索您的 Drupal 站点上的文章：

    ```php
    GET http://localhost/articles
    ```

    ```php
    Accept: application/json
    ```

## 它是如何工作的…

实体查询是 Drupal 数据库 API 之上的一个抽象。Drupal 将实体和字段数据存储在规范化的数据库表中。实体查询构建适当的数据库查询以检查实体的基础表、数据表以及任何字段表。它协调所有所需的`JOIN`语句。

当执行实体查询时，它返回匹配实体的 ID。然后，将这些 ID 传递给`loadMultiple`方法以检索实体对象。

## 还有更多…

实体查询可以做更多的事情。

### 计数查询

实体查询也可以执行计数而不是返回实体 ID。这是通过在实体查询对象上调用`count`方法来完成的：

```php
$node_storage->getQuery()
  ->accessCheck(FALSE)
  ->condition('type', 'article')
  ->condition('status', FALSE)
  ->count()
  ->execute();
```

上述代码将返回未发布文章的数量。

# 检查实体访问

在此配方中，我们将演示如何检查当前用户是否有权查看用于执行实体访问检查的`_entity_access`路由要求。此配方将使用自己的实体访问控制，以便响应是一个`404 Not Found`响应而不是`403` `Forbidden`响应。

## 如何做到这一点…

1.  在你的模块中创建一个`ArticleController`控制器中的`get`方法，该方法有一个参数用于`node`实体对象，该对象将由路由参数提供：

    ```php
    <?php
    ```

    ```php
    namespace Drupal\mymodule\Controller;
    ```

    ```php
    use Drupal\Core\Controller\ControllerBase;
    ```

    ```php
    use Drupal\node\NodeInterface;
    ```

    ```php
    use Symfony\Component\HttpFoundation\JsonResponse;
    ```

    ```php
    use Symfony\Component\HttpFoundation\Request;
    ```

    ```php
    class ArticleController extends ControllerBase {
    ```

    ```php
      public function get(NodeInterface $node):
    ```

    ```php
        JsonResponse {
    ```

    ```php
      }
    ```

    ```php
    }
    ```

如果使用之前配方中的相同控制器，这将添加一个新的`use`语句用于`Drupal\node\NodeInterface`接口。

1.  然后，我们获取实体类型管理器并检索`node`实体类型的访问控制处理程序：

    ```php
      public function get(NodeInterface $node):
    ```

    ```php
        JsonResponse {
    ```

    ```php
       $entity_type_manager = $this->entityTypeManager();
    ```

    ```php
       $access_handler = $entity_type_manager->
    ```

    ```php
          getAccessControlHandler('node');
    ```

    ```php
      }
    ```

实体类型管理器是实体类型信息的存储库，用于获取实体类型的处理程序，例如访问控制处理程序。

1.  从访问控制处理程序中调用`access`方法：

    ```php
      public function get(NodeInterface $node):
    ```

    ```php
        JsonResponse {
    ```

    ```php
       $entity_type_manager = $this->entityTypeManager();
    ```

    ```php
       $access_handler = $entity_type_manager->
    ```

    ```php
          getAccessControlHandler('node');
    ```

    ```php
       $node_access = $access_handler->access($node,
    ```

    ```php
          'view');
    ```

    ```php
      }
    ```

`access`的第一个参数是实体。第二个参数是操作，对于此配方是`view`。默认情况下，`access`方法返回一个布尔值。

1.  如果结果不允许，我们希望返回一个`404 Not Found`响应：

    ```php
      public function get(NodeInterface $node):
    ```

    ```php
        JsonResponse {
    ```

    ```php
       $entity_type_manager = $this->entityTypeManager();
    ```

    ```php
       $access_handler = $entity_type_manager->
    ```

    ```php
          getAccessControlHandler('node');
    ```

    ```php
       $node_access = $access_handler->access($node,
    ```

    ```php
          'view');
    ```

    ```php
       if (!$node_access) {
    ```

    ```php
        return new JsonResponse(NULL, 404);
    ```

    ```php
       }
    ```

    ```php
      }
    ```

我们将返回一个 JSON 响应，但带有`NULL`数据和`404`状态码。

1.  如果结果是允许的，返回文章节点内容的 JSON 响应：

    ```php
      public function get(NodeInterface $node):
    ```

    ```php
        JsonResponse {
    ```

    ```php
       $entity_type_manager = $this->entityTypeManager();
    ```

    ```php
       $access_handler = $entity_type_manager->
    ```

    ```php
          getAccessControlHandler('node');
    ```

    ```php
       $node_access = $access_handler->access($node,
    ```

    ```php
          'view');
    ```

    ```php
       if (!$node_access) {
    ```

    ```php
        return new JsonResponse(NULL, 404);
    ```

    ```php
       }
    ```

    ```php
       return new JsonResponse(
    ```

    ```php
        $node->toArray(),
    ```

    ```php
       );
    ```

    ```php
      }
    ```

1.  我们必须在模块的`routing.yml`中创建一个路由，指向`ArticleController`的`get`方法，以便对`/articles/{node}`路径的 HTTP `GET`请求：

    ```php
    mymodule.get_article:
    ```

    ```php
      path: /articles/{node}
    ```

    ```php
      defaults:
    ```

    ```php
       _controller: Drupal\mymodule\Controller\
    ```

    ```php
          ArticleController::get
    ```

    ```php
      requirements:
    ```

    ```php
       _permission: 'access content'
    ```

由于我们的路由参数名为`node`，并且我们的控制器方法接受节点实体的类，Drupal 将自动将我们的`node`参数转换为实体对象。

1.  重建你的 Drupal 站点的缓存，使其了解新的路由：

    ```php
    php vendor/bin/drush cr
    ```

1.  然后，可以使用如下 HTTP 请求检索你 Drupal 站点上的文章：

    ```php
    GET http://localhost/articles/1
    ```

    ```php
    Accept: application/json
    ```

## 它是如何工作的…

此配方未使用`_entity_access`路由要求来展示实体验证。`_entity_access`路由要求在路由参数中位于实体上的操作上调用访问方法。

以下操作被 Drupal 的实体访问系统识别：

+   `view`: 用户被允许查看实体。

+   `view_label`: 一个较少使用的操作。它用于检查用户是否有权至少查看实体标签/标题。

+   `update`: 用户被允许更新实体。

+   `delete`: 用户被允许删除实体。

注意

创建实体的访问是一个对访问控制处理器的不同调用，因为它不能与实体对象进行比较。访问控制处理器上的 `createAccess` 方法用于检查用户是否有权创建新实体。

通过在实体本身上调用 `access` 方法也可以检查实体的 `access`。此菜谱旨在展示访问控制处理器。菜谱可以通过执行以下操作来检查访问：

```php
$node_access = $node->access('view');
```

默认情况下，访问检查使用当前用户。访问检查允许传递一个替代用户账户以执行访问检查。这在在命令行或后台作业中运行代码时可能很有用：

```php
$node_access = $node->access('view', $other_user);
```

# 更新实体的字段值

在此菜谱中，我们将定义一个路由来更新文章节点的字段值。该路由将用于发送 JSON 的 `HTTP PATCH` 请求，以指定新的标题和正文文本。

## 如何操作...

1.  在 `ArticleController` 控制器中创建一个 `update` 方法，该方法接收传入的请求和节点对象：

    ```php
    <?php
    ```

    ```php
    namespace Drupal\mymodule\Controller;
    ```

    ```php
    use Drupal\Core\Controller\ControllerBase;
    ```

    ```php
    use Drupal\node\NodeInterface;
    ```

    ```php
    use Symfony\Component\HttpFoundation\JsonResponse;
    ```

    ```php
    use Symfony\Component\HttpFoundation\Request;
    ```

    ```php
    class ArticleController extends ControllerBase {
    ```

    ```php
      public function update(Request $request,
    ```

    ```php
        NodeInterface $node): JsonResponse {
    ```

    ```php
      }
    ```

    ```php
    }
    ```

我们需要请求对象，以便我们可以检索此方法提供的 JSON 负载以及节点对象以进行更新。

1.  接下来，我们将使用 Drupal 的 JSON 序列化实用工具类将请求的内容从 JSON 转换为 PHP 数组：

    ```php
      public function update(Request $request,
    ```

    ```php
        NodeInterface $node): JsonResponse {
    ```

    ```php
       $content = $request->getContent();
    ```

    ```php
       $json = \Drupal\Component\Serialization\
    ```

    ```php
          Json::decode($content);
    ```

    ```php
      }
    ```

虽然我们可以直接使用 `json_decode` 函数，但利用 Drupal 标准提供的实用工具类可以标准化代码的工作方式。

1.  首先，如果请求的 JSON 中提供了，我们将更新文章节点标题：

    ```php
      public function update(Request $request,
    ```

    ```php
        NodeInterface $node): JsonResponse {
    ```

    ```php
       $content = $request->getContent();
    ```

    ```php
       $json = \Drupal\Component\Serialization\
    ```

    ```php
          Json::decode($content);
    ```

    ```php
       if (!empty($json['title'])) {
    ```

    ```php
        $node->setTitle($json['title']);
    ```

    ```php
       }
    ```

    ```php
      }
    ```

一些实体类提供了设置特定字段值的方法，例如 `Node` 类使用 `setTitle` 来修改节点的 `title`。

1.  然后，如果请求的 JSON 中提供了，我们将更新文章的主体字段：

    ```php
      public function update(Request $request,
    ```

    ```php
        NodeInterface $node): JsonResponse {
    ```

    ```php
       $content = $request->getContent();
    ```

    ```php
       $json = \Drupal\Component\Serialization\
    ```

    ```php
          Json::decode($content);
    ```

    ```php
       if (!empty($json['title'])) {
    ```

    ```php
        $node->setTitle($json['title']);
    ```

    ```php
       }
    ```

    ```php
       if (!empty($json['body'])) {
    ```

    ```php
        $node->set('field_body', $json['body']);
    ```

    ```php
       }
    ```

    ```php
      }
    ```

`set` 方法允许我们设置字段的值。第一个参数是字段名，第二个参数是字段的值。

1.  更新字段后，我们可以保存实体并将其作为 JSON 响应返回：

    ```php
      public function update(Request $request,
    ```

    ```php
        NodeInterface $node): JsonResponse {
    ```

    ```php
       $content = $request->getContent();
    ```

    ```php
       $json = \Drupal\Component\Serialization\
    ```

    ```php
          Json::decode($content);
    ```

    ```php
       if (isset($json['title'])) {
    ```

    ```php
        $node->setTitle($json['title']);
    ```

    ```php
       }
    ```

    ```php
       if (isset($json['body'])) {
    ```

    ```php
        $node->set('body', $json['body']);
    ```

    ```php
       }
    ```

    ```php
       $node->save();
    ```

    ```php
       return new JsonResponse(
    ```

    ```php
        $node->toArray()
    ```

    ```php
       );
    ```

    ```php
      }
    ```

1.  我们必须在 `routing.yml` 中创建路由，该路由指向 `ArticleController` 的 `update` 方法，用于对 `/articles/{node}` 路径的 HTTP `PATCH` 请求：

    ```php
    mymodule.update_article:
    ```

    ```php
      path: /articles/{node}
    ```

    ```php
      defaults:
    ```

    ```php
       _controller: Drupal\mymodule\Controller\
    ```

    ```php
          ArticleController::update
    ```

    ```php
      methods: [PATCH]
    ```

    ```php
      requirements:
    ```

    ```php
       _access: 'TRUE'
    ```

此路由应具有 `_entity_access: 'node.update'` 作为其要求。然而，我们已使用 `_access: 'TRUE'` 来绕过此菜谱的访问检查。

1.  重建您的 Drupal 网站缓存，使其了解新路由：

    ```php
    php vendor/bin/drush cr
    ```

1.  然后，可以使用以下 HTTP 请求来更新您 Drupal 网站上的文章：

    ```php
    PATCH http://localhost/articles/1
    ```

    ```php
    Content-Type: application/json
    ```

    ```php
    Accept: application/json
    ```

    ```php
    {
    ```

    ```php
      "title": "New updated title!",
    ```

    ```php
      "body": "Modified body text"
    ```

    ```php
    }
    ```

## 它是如何工作的...

当更新一个实体时，实体存储会在数据库中更新字段值以及实体的数据库记录。实体存储会将字段值写入包含字段值的适当数据库表。

当实体被更新时，实体存储会触发以下钩子，允许您挂钩到实体的更新。正在更新的实体作为参数传递：

+   `hook_ENTITY_TYPE_presave`

+   `hook_entity_presave`

+   `hook_ENTITY_TYPE_update`

+   `hook_entity_update`

实体也可以进行验证，以确保其更新的值是正确的。这将在下一个菜谱*执行实体验证*中介绍。

# 执行实体验证

在这个菜谱中，我们将介绍实体验证。Drupal 已经集成了**Symfony Validator**组件。实体在保存之前可以进行验证。我们将基于上一个菜谱，允许更新文章节点以添加对其值的验证。

## 如何做到这一点...

1.  我们将向上一个菜谱中的`update`方法添加验证：

    ```php
      public function update(Request $request,
    ```

    ```php
        NodeInterface $node): JsonResponse {
    ```

    ```php
       $content = $request->getContent();
    ```

    ```php
       $json = \Drupal\Component\Serialization\
    ```

    ```php
          Json::decode($content);
    ```

    ```php
       if (isset($json['title'])) {
    ```

    ```php
        $node->setTitle($json['title']);
    ```

    ```php
       }
    ```

    ```php
       if (isset($json['body'])) {
    ```

    ```php
        $node->set('body', $json['body']);
    ```

    ```php
       }
    ```

    ```php
       $node->save();
    ```

    ```php
       return new JsonResponse(
    ```

    ```php
        $node->toArray()
    ```

    ```php
       );
    ```

    ```php
      }
    ```

1.  我们将修改`update`方法，在保存节点之前对其进行`validate`：

    ```php
      public function update(Request $request,
    ```

    ```php
        NodeInterface $node): JsonResponse {
    ```

    ```php
       $content = $request->getContent();
    ```

    ```php
       $json = \Drupal\Component\Serialization\
    ```

    ```php
          Json::decode($content);
    ```

    ```php
       if (isset($json['title'])) {
    ```

    ```php
        $node->setTitle($json['title']);
    ```

    ```php
       }
    ```

    ```php
       if (isset($json['body'])) {
    ```

    ```php
        $node->set('body', $json['body']);
    ```

    ```php
       }
    ```

    ```php
       $constraint_violations = $node->validate();
    ```

    ```php
       $node->save();
    ```

    ```php
       return new JsonResponse(
    ```

    ```php
        $node->toArray()
    ```

    ```php
       );
    ```

    ```php
      }
    ```

我们将调用`validate`方法，该方法将对实体及其字段值上的所有约束执行验证。

1.  `validate`方法返回一个包含任何约束违规的对象，并且不会抛出异常。我们必须检查是否存在任何违规：

    ```php
      public function update(Request $request,
    ```

    ```php
        NodeInterface $node): JsonResponse {
    ```

    ```php
       $content = $request->getContent();
    ```

    ```php
       $json = \Drupal\Component\Serialization\
    ```

    ```php
          Json::decode($content);
    ```

    ```php
       if (isset($json['title'])) {
    ```

    ```php
        $node->setTitle($json['title']);
    ```

    ```php
       }
    ```

    ```php
       if (isset($json['body'])) {
    ```

    ```php
        $node->set('body', $json['body']);
    ```

    ```php
       }
    ```

    ```php
       $constraint_violations = $node->validate();
    ```

    ```php
       if (count($constraint_violations) > 0) {
    ```

    ```php
       }
    ```

    ```php
       $node->save();
    ```

    ```php
       return new JsonResponse(
    ```

    ```php
        $node->toArray()
    ```

    ```php
       );
    ```

    ```php
      }
    ```

返回的对象，一个`\Drupal\Core\Entity\EntityConstraintViolationListInterface`的实例，实现了`\Countable`接口。这允许我们使用`count`函数来查看是否存在任何违规。

1.  如果存在约束违规，我们将构建一个错误消息数组，并返回一个`400 Bad Request`响应：

    ```php
      public function update(Request $request,
    ```

    ```php
        NodeInterface $node): JsonResponse {
    ```

    ```php
       $content = $request->getContent();
    ```

    ```php
       $json = \Drupal\Component\Serialization\
    ```

    ```php
          Json::decode($content);
    ```

    ```php
       if (isset($json['title'])) {
    ```

    ```php
        $node->setTitle($json['title']);
    ```

    ```php
       }
    ```

    ```php
       if (isset($json['body'])) {
    ```

    ```php
        $node->set('body', $json['body']);
    ```

    ```php
       }
    ```

    ```php
       $constraint_violations = $node->validate();
    ```

    ```php
       if (count($constraint_violations) > 0) {
    ```

    ```php
        $errors = [];
    ```

    ```php
        foreach ($constraint_violations as $violation) {
    ```

    ```php
          $errors[] = $violation->getPropertyPath()
    ```

    ```php
            . ': ' . $violation->getMessage();
    ```

    ```php
        }
    ```

    ```php
        return new JsonResponse($errors, 400);
    ```

    ```php
       }
    ```

    ```php
       $node->save();
    ```

    ```php
       return new JsonResponse(
    ```

    ```php
        $node->toArray()
    ```

    ```php
       );
    ```

    ```php
      }
    ```

`EntityConstraintViolationListInterface`对象也是可迭代的，允许我们遍历所有违规。从每个违规中，我们可以使用`getPropertyPath`来识别无效字段，并使用`getMessage`来获取有关无效值的信息。

1.  以下 HTTP 请求会在`title`字段为空值时触发约束违规：

    ```php
    PATCH https://localhost/articles/1
    ```

    ```php
    Content-Type: application/json
    ```

    ```php
    Accept: application/json
    ```

    ```php
    {
    ```

    ```php
      "title": "",
    ```

    ```php
      "body": "Modified body text"
    ```

    ```php
    }
    ```

## 它是如何工作的...

Drupal 使用**Symfony Validator**组件([`symfony.com/components/Validator`](https://symfony.com/components/Validator))来验证数据。Validator 组件有一个概念，即需要验证的约束，如果验证失败，则会报告违规。Drupal 的实体验证采用自外向内的方法：验证实体级别的约束，然后逐个验证每个字段，逐级向下验证每个字段的属性。

实体验证在实体保存时不会自动运行。它是一个显式操作，在以编程方式操作实体时必须调用。Drupal 仅在通过其表单修改实体时调用实体验证。

同时，实体无效化的调用者必须选择如何对约束违规做出反应，就像我们的菜谱那样防止实体被保存。无效实体始终可以通过编程方式保存。

## 还有更多...

在验证实体值时，有更多选项。让我们在接下来的几节中看看这些选项。

### 直接验证字段

字段项类有一个`validate`方法，可以直接验证特定字段而不是验证整个实体。这可以通过使用`get`方法获取字段项，然后在字段项上调用`validate`方法来完成：

```php
$node->get('body')->validate();
```

### 过滤约束违规

`EntityConstraintViolationListInterface` 类扩展了由 Symfony 提供的 `Symfony\Component\Validator\ConstraintViolationListInterface` 类。这为 Drupal 添加了特定的方法来过滤返回的违规。以下方法可用于过滤违规：

+   `getEntityViolations`：一些违规可能位于实体级别，而不是类级别。约束可能应用于实体级别，而不是特定字段。例如，Workspaces 模块在实体级别添加了一个约束来检查工作区冲突。

+   `filterByFields`：给定一系列字段名，违规将仅限于那些适用于这些字段。

+   `filterByFieldAccess`：此过滤器基于用户可访问的字段过滤违规。Drupal 允许以无效状态保存实体，特别是如果工作流允许权限较低的用户修改实体的特定字段。如果使用此过滤器，请谨慎，因为实体必须具有用户可能没有访问权限的更新字段。

之前的方法总是返回一个新的对象实例，并且不会对原始违规列表产生副作用。

# 删除实体

在此菜谱中，我们将逐步讲解如何删除实体。删除实体允许您从数据库中删除实体，使其不再存在。

## 如何操作…

1.  在您的模块中，在 `ArticleController` 控制器中创建一个 `delete` 方法，该方法有一个参数用于节点实体对象，该对象将由路由参数提供：

    ```php
    <?php
    ```

    ```php
    namespace Drupal\mymodule\Controller;
    ```

    ```php
    use Drupal\Core\Controller\ControllerBase;
    ```

    ```php
    use Drupal\node\NodeInterface;
    ```

    ```php
    use Symfony\Component\HttpFoundation\JsonResponse;
    ```

    ```php
    use Symfony\Component\HttpFoundation\Request;
    ```

    ```php
    class ArticleController extends ControllerBase {
    ```

    ```php
      public function delete(NodeInterface $node):
    ```

    ```php
        JsonResponse {
    ```

    ```php
      }
    ```

    ```php
    }
    ```

1.  要删除实体，您将调用 `delete` 方法：

    ```php
      public function delete(NodeInterface $node):
    ```

    ```php
        JsonResponse {
    ```

    ```php
       $node->delete();
    ```

    ```php
      }
    ```

`delete` 方法立即从数据库存储中删除实体。

1.  然后，我们将返回一个空的 JSON 响应：

    ```php
      public function delete(NodeInterface $node):
    ```

    ```php
        JsonResponse {
    ```

    ```php
       $node->delete();
    ```

    ```php
       return new JsonResponse(null, 204);
    ```

    ```php
      }
    ```

当没有返回内容时，我们将使用 `204 No Content` 状态码。

1.  我们必须在 `routing.yml` 中创建路由，该路由指向 `ArticleController` 的 `delete` 方法，以便对 `/articles/{node}` 路径进行 HTTP `DELETE` 请求：

    ```php
    mymodule.delete_article:
    ```

    ```php
      path: /articles/{node}
    ```

    ```php
      defaults:
    ```

    ```php
       _controller: Drupal\mymodule\Controller\
    ```

    ```php
          ArticleController::delete
    ```

    ```php
      methods: [DELETE]
    ```

    ```php
      requirements:
    ```

    ```php
       _access: 'TRUE'
    ```

此路由的要求应该是 `_entity_access: 'node.delete'`。然而，我们使用了 `_access: 'TRUE'` 来绕过此菜谱的访问检查。

1.  重建您的 Drupal 网站缓存，使其了解新的路由：

    ```php
    php vendor/bin/drush cr
    ```

1.  可以使用如下 HTTP 请求检索您 Drupal 网站上的文章：

    ```php
    DELETE http://localhost/articles/1
    ```

    ```php
    Accept: application/json
    ```

1.  对文章进行第二次 HTTP 请求将返回 `404 Not Found` 响应，因为它已被删除：

    ```php
    GET http://localhost/articles/1
    ```

    ```php
    Accept: application/json
    ```

## 它是如何工作的…

实体类上的 `delete` 方法被委派给实体类型存储处理器的 `delete` 方法。当内容实体被删除时，存储会从数据库中清除所有字段值和实体的数据库记录。删除是永久的，无法撤销。

当实体被删除时，实体存储会触发以下钩子，允许您挂钩到实体的删除。正在删除的实体作为参数传递：

+   `hook_ENTITY_TYPE_predelete`

+   `hook_entity_predelete`

+   `hook_ENTITY_TYPE_delete`

+   `hook_entity_delete`

在钩子中抛出异常将回滚数据库事务并阻止实体被删除，但如果不适当处理，也可能导致 Drupal 崩溃。
