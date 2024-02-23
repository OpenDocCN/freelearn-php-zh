# 第五章：微服务开发

在最后几章中，我们解释了如何安装 Docker、Composer 和 Lumen，这对每个微服务都是必要的。在本章中，我们将开发*查找秘密*应用程序的一些部分。

在本章中，我们将开发一些更关键的部分，例如路由、中间件、与数据库的连接、队列以及查找秘密应用程序的微服务之间的通信，这样您将能够在将来开发应用程序的其余部分。

我们的应用程序结构将包括以下四个微服务：

+   **User:** 管理注册和账户操作。它还负责存储和管理我们的秘密钱包。

+   **Secrets:** 在世界各地生成随机秘密，并允许我们获取有关每个秘密的信息。

+   **Location:** 检查最近的秘密和用户。

+   **Battle:** 管理用户之间的战斗。它还修改钱包以在战斗后添加和删除秘密。

# 依赖管理

依赖管理是一种方法论，允许您声明项目所需的库，并使其更容易安装或更新。PHP 最知名的工具称为**Composer**。在之前的章节中，我们对这个工具进行了简要概述。

对于我们的项目，我们将需要为每个微服务使用单个 Composer 设置。当我们安装 Lumen 时，Composer 为我们完成了工作并创建了配置文件，但现在我们将详细解释它是如何工作的。

一旦我们安装了 Docker 并且我们在 PHP-FPM 容器中，我们需要工作，就需要生成`composer.json`文件。这是 Composer 的配置文件，我们在其中定义我们的项目和所有依赖项：

```php
    {
      "name": "php-microservices/user",
      "description": "Finding Secrets, User microservice",
      "keywords": ["finding secrets", "user", "microservice", "Lumen" ],
      "license": "MIT",
      "type": "project",
      "require": {
        "php": ">=5.5.9",
        "laravel/lumen-framework": "5.2.*",
        "vlucas/phpdotenv": "~2.2"
      },
      "require-dev": {
        "fzaninotto/faker": "~1.4",
        "phpunit/phpunit": "~4.0",
        "behat/behat": "3.*"
      },
      "autoload": {
        "psr-4": {
            "App": "app/"
        }
      },
      "autoload-dev": {
        "classmap": [
            "tests/",
            "database/"
        ]
      }
    }
```

`composer.json`文件的前 6 行（名称、描述、关键字、许可证和类型）用于识别项目。如果您在任何存储库中分享项目，它将是公开的。

`"require"`部分定义了项目中需要的必需库以及每个库的版本。`"require-dev"`非常类似，但它们是需要在开发机器上安装的库（例如，任何测试库）。

`"autoload"`和`"autoload-dev"`定义了我们的类将如何加载以及要映射到项目的不同用途的文件夹。

创建了这个文件后，我们可以在我们的机器上执行以下命令：

```php
**composer install**

```

此时，Composer 将检查我们的设置，并下载所有必需的库，包括 Lumen。

还有其他工具，但它们没有被使用得那么多，也不够灵活。

# 路由

路由是应用程序入口点（请求）和执行逻辑的源代码中的特定类和方法之间的映射。例如，您可以在应用程序中定义`/users`路由和`Users`类中的`list()`方法之间的映射。一旦您放置了这个映射，一旦您的应用程序收到对`/users`路由的请求，它将执行`Users`类中`list()`方法中的所有逻辑。路由允许 API 消费者与您的应用程序进行交互。在微服务中，最常用的是 RESTful 约定，我们将遵循它。

+   **HTTP 方法**：

+   **GET:** 用于检索有关指定实体或实体集合的信息。数据量不重要；我们将使用 GET 来获取一个或多个结果，还可以使用过滤器来过滤结果。

+   **POST:** 用于在应用程序中输入信息。它还用于发送新信息以创建新事物或发送消息。

+   **PUT:** 用于更新已存储在应用程序中的整个实体。

+   **PATCH:** 用于部分更新已存储在应用程序中的实体。

+   **DELETE:** 用于从应用程序中删除实体。

Lumen 中的路由文件位于`app/Http/routes.php`，因此我们将为每个微服务有一个路由文件。对于`User`微服务，我们将有以下端点：

```php
    $app->group([
        'prefix' => 'api/v1',
        'namespace' => 'App\Http\Controllers'],
        function ($app) {
            $app->get('user', 'UserController@index');
            $app->get('user/{id}', 'UserController@get');
            $app->post('user', 'UserController@create');   
            $app->put('user/{id}', 'UserController@update');
            $app->delete('user/{id}', 'UserController@delete');
            $app->get('user/{id}/location', 
            'UserController@getCurrentLocation');
            $app->post('user/{id}/location/latitude/{latitude}
            /longitude/{longitude}', 
            'UserController@setCurrentLocation');
        }
    );
```

在前面的代码中，我们为`User`微服务定义了我们的路由。

在 Lumen 中，API 的版本可以在路由文件中通过包含'prefix'来指定。这个框架还允许我们为同一个微服务拥有不同的 API 版本，因此我们不需要修改现有的方法来在不同的版本中使用。

`'namespace'`为同一组中包含的所有方法定义了相同的命名空间。以下行定义了每个入口点：

```php
    $app->get('user/{id}', 'UserController@get');
```

例如，前面的方法包含在前缀为`'api/v1'`的组中；动词是 GET，入口点是`user/{id}`，因此可以通过执行 HTTP GET 调用`http://localhost:8080/api/v1/user/123`来检索。`UserController@get`参数定义了我们需要在哪里开发此调用的逻辑--在这种情况下，它在控制器`UserController`和名为`get`的方法中。

在 Lumen 中，存储所有控制器的标准文件夹是`app/Http/Controllers`，因此您只需要使用 IDE 创建`app/Http/Controllers/UserController.php`文件，并包含以下内容：

```php
    <?php
    namespace App\Http\Controllers;
    use Illuminate\Http\Request;
    class UserController extends Controller
    {
      public function index(Request $request)
      {
        return response()->json(['method' => 'index']);
      }
      public function get($id)
      {
        return response()->json(['method' => 'get', 'id' => $id]);
      }
      public function create(Request $request)
      {
        return response()->json(['method' => 'create']);
      }
      public function update(Request $request, $id)
      {
        return response()->json(['method' => 'update', 'id' => $id]);
      }
      public function delete($id)
      {
        return response()->json(['method' => 'delete', 'id' => $id]);
      }
      public function getCurrentLocation($id)
      {
        return response()->json(['method' => 'getCurrentLocation',
                                'id' => $id]);
      }
      public function setCurrentLocation(Request $request, $id,
                                         $latitude, $longitude)
      {
        return response()->json(['method' => 'setCurrentLocation',
                                 'id' => $id, 'latitude' => $latitude,
                                 'longitude' => $longitude]);
      }
    }
```

前面的代码定义了我们在`app/Http/routes.php`文件中指定的所有方法。例如，我们返回一个简单的 JSON 来测试每个路由是否正常工作。

### 提示

请记住，微服务之间通信使用的主要语言是 JSON，因此我们所有的响应都需要是 JSON 格式。

在 Lumen 中，返回 JSON 响应非常容易；您只需要使用响应实例的`json()`方法，如下所示：

```php
    return response()->json(['method' => 'update', 'id' => $id]);
```

如果我们`$id`变量中存储的值是`123`，则前面的句子将返回一个格式良好的 JSON 响应：

```php
    {
      "method" : "update",
      "id" : 123
    }
```

现在，我们已经为我们的`User`微服务做好了一切准备。

也许，您想知道在我们的容器环境中，`User`微服务的`get()`方法分配了什么 URI。找到它非常容易--只需打开`docker-compose.yml`文件，您就可以找到`microservice_user_nginx`容器的端口映射。我们设置的端口映射表明我们的本地主机 8084 端口将重定向请求到容器的 80 端口。总之，我们的 URI 将是`http://localhost:8084/api/v1/get/123`。

## Postman

我们基于微服务的应用程序将不会有前端部分；API Rest 的目标是创建可以被不同平台（Web、iOS 或 Android）调用的微服务，只需调用路由文件中可用的方法；因此，为了执行对我们微服务的调用，我们将使用 Postman。这是一个工具，允许您执行包括您需要的参数在内的不同调用。使用 Postman，可以保存方法以便将来使用。

您可以从[`www.getpostman.com`](https://www.getpostman.com)下载或安装 Postman，如下所示：

![Postman](img/B06142_05_01.jpg)

Postman 工具概述

正如您在前面的 Postman 工具截图中所看到的，它具有许多功能，比如保存请求或设置不同的环境；但是现在，我们只需要知道执行调用我们应用程序的基本功能，如下所示：

1.  设置动词--GET、POST、PUT、PATCH 或 DELETE。有些框架无法重现 PUT 或 PATCH 调用，因此您需要设置动词 POST，并包含一个键为`_method`值为 PUT 或 PATCH 的参数。

1.  设置请求 URL。这是我们应用程序的期望入口点。

1.  如果需要，添加更多参数--例如，用于过滤结果的参数。对于 POST 调用，Body 按钮将被启用，以便您可以在请求正文中发送参数，而不是在 URL 中发送。

1.  点击发送以执行调用。

1.  响应将显示状态代码和秒数。

## 中间件

正如我们在前面的章节中所解释的，中间件在基于微服务的应用程序中非常有用。让我们解释一下如何使用它们使用 Lumen。

Lumen 有一个目录用于放置所有的中间件，因此我们将在`User`微服务上创建一个中间件，以检查消费者是否具有提供的`API_KEY`以与我们的应用程序通信。

### 提示

为了识别我们的消费者，我们建议您使用`API_KEY`。这种做法将避免不受欢迎的消费者使用我们的应用程序。

假设我们向客户提供了一个值为`RSAy430_a3eGR`的`API_KEY`，并且在每个请求中都需要发送这个值。我们可以使用中间件来检查是否提供了这个`API_KEY`。创建一个名为`App\Http\Middleware\ApiKeyMiddleware.php`的文件，并将以下代码放入其中：

```php
    <?php
    namespace App\Http\Middleware;
    use Closure;

    class ApiKeyMiddleware
    {
      const API_KEY = 'RSAy430_a3eGR';
      public function handle($request, Closure $next)
      {
        if ($request->input('api_key') !== self::API_KEY) {
            die('API_KEY invalid');
        }
        return $next($request);
      }
    }
```

一旦我们创建了我们的中间件，我们必须将其添加到应用程序中。为此，请在`bootstrap/app.php`文件中包含以下行：

```php
    $app->middleware([App\Http\Middleware\ApiKeyMiddleware::class]);
    $app->routeMiddleware(['api_key' => App\Http\Middleware
    \ApiKeyMiddleware::class]);
```

现在，我们可以将中间件添加到`routes.php`文件中。它可以放在不同的地方；您可以将它放在单个请求中，甚至整个组中，如下所示：

```php
    $app->group([
        **'middleware' => 'api_key'**, 
        'prefix' => 'api/v1',
        'namespace' => 'App\Http\Controllers'],
        function($app) {
            $app->get('user', 'UserController@index');
            $app->get('user/{id}', 'UserController@get');
            $app->post('user', 'UserController@create');
```

在 Postman 上试试看；向`http://localhost:8084/api/v1/user`发出 HTTP POST 调用。您将看到一个消息，上面写着`API_KEY invalid`。现在做同样的调用，但添加一个名为`API_KEY`值为`RSAy430_a3eGR`的参数；请求通过中间件并到达函数。

# 实现微服务调用

既然我们知道如何发出调用，让我们创建一个更复杂的例子。我们将构建我们的战斗系统。如前几章所述，战斗是两名玩家之间为了从失败者那里获取秘密而进行的战斗。我们的战斗系统将由三轮组成，在每一轮中都会进行一次掷骰子；赢得大多数轮次的用户将成为赢家，并从失败者的钱包中获得一个秘密。

### 提示

我们建议使用一些测试开发实践（TDD、BDD 或 ATDD），正如我们之前解释的那样；您可以在前面的章节中看到一些例子。在本章中，我们不会再包含更多的测试。

在 Battle 微服务中，我们可以在`BattleController.php`文件中创建一个用于战斗的函数；让我们看一个有效的结构方法：

```php
    public function duel(Request $request) 
    {
        return response()->json([]);
    }
```

不要忘记在`routes.php`文件中添加端点，将 URI 链接到我们的方法：

```php
    $app->post('battle/duel', 'BattleController@duel');
```

在这一点上，Battle 微服务的`duel`方法将可用；用 Postman 试试看。

现在，我们将实现决斗。我们需要为骰子创建一个新的类。为了存储一个新的类，我们将在根目录下创建一个名为`Algorithm`的新文件夹，文件`Dice.php`将包含骰子方法：

```php
    <?php
    namespace App\Algorithm;
    class Dice
    {
        const TOTAL_ROUNDS   = 3;
        const MIN_DICE_VALUE = 1;
        const MAX_DICE_VALUE = 6;
        public function fight()
        {
            $totalRoundsWin = [
                'player1' => 0,
                'player2' => 0
            ];

            for ($i = 0; $i < self::TOTAL_ROUNDS; $i++) {
                $player1Result = rand(
                    self::MIN_DICE_VALUE,
                    self::MAX_DICE_VALUE
                );
                $player2Result = rand(
                    self::MIN_DICE_VALUE,
                    self::MAX_DICE_VALUE
                );
                $roundResult = ($player1Result <=> $player2Result);
                if ($roundResult === 1) {
                    $totalRoundsWin['player1'] = 
                    $totalRoundsWin['player1'] + 1;
                } else if ($roundResult === -1) {
                    $totalRoundsWin['player2'] = 
                    $totalRoundsWin['player2'] + 1;
                }
            }

            return $totalRoundsWin;
        }
    }
```

一旦我们开发了`Dice`类，我们将从`BattleController`中调用它，以查看谁赢得了战斗。首先要做的是在`BattleController.php`文件的顶部包含`Dice`类，然后我们可以创建一个我们将用于决斗的算法实例（这是一个很好的做法，以便在将来更改决斗系统；例如，如果我们想要使用基于能量点或卡牌游戏的决斗，我们只需要更改`Dice`类为新的类）。

`duel`函数将返回一个 JSON，其中包含战斗结果。请查看`BattleController.php`中包含的新突出显示的代码：

```php
    <?php
    namespace App\Http\Controllers;
    use Illuminate\Http\Request;
    **use App\Algorithm\Dice;**

    class BattleController extends Controller
    {
 **protected $battleAlgorithm = null;**
 **protected function setBattleAlgorithm()**
 **{**
 **$this->battleAlgorithm = new Dice();**
 **}**

        /** ... Code omitted ... **/

 **public function duel(Request $request)**
 **{**
 **$this->setBattleAlgorithm();**
**$duelResult = $this->battleAlgorithm->fight();** 
 **return response()->json(**
 **[**
 **'player1'     => $request->input('userA'),**
**                     'player2'     => $request->input('userB'),**
**                     'duelResults' => $duelResult**
**                 ]**
**            );**
 **}**
    }
```

试试使用 Postman；记住这是一个 HTTP POST 请求到 URI `http://localhost:8081/api/v1/battle/duel`（注意我们在 Docker 上设置了端口 8081 用于战斗微服务），并且需要发送参数`userA`和`userB`，其中包含您想要的用户名。如果一切正确，您应该会收到类似于这样的响应：

```php
    {
        "player1": "John",
        "player2": "Joe",
        "duelResults": {
            "player1": 2,
            "player2": 1
        }
    }
```

## 请求生命周期

请求生命周期是请求被返回给消费者作为响应之前的地图。了解这个过程是很有趣的，以避免在请求过程中出现问题。每个框架都有自己的请求方式，但它们都非常相似，并遵循一些像 Lumen 一样的基本步骤：

1.  每个请求都由`public/index.php`管理。它没有太多的代码，只是加载由 Composer 生成的自动加载程序定义，并从`bootstrap/app.php`创建应用程序的实例。

1.  请求被发送到 HTTP 内核，它定义了一些必要的事情，比如错误处理、日志记录、应用环境和其他在请求执行之前应该添加的必要任务。HTTP 内核还定义了请求在检索应用程序之前必须通过的中间件列表。

1.  一旦请求通过了 HTTP 内核并到达应用程序，它就会到达路由并尝试将其与正确的路由匹配。

1.  它执行控制器和对应路由的代码，并创建并返回一个响应对象。

1.  HTTP 头和响应对象内容被返回给客户端。

这只是请求-响应工作流程的一个基本示例；真实的过程更加复杂。你应该考虑到 HTTP 内核就像一个大黑匣子，它做了一些对开发者来说并不可见的事情，所以理解这个例子对本章来说已经足够了。

## 使用 Guzzle 进行微服务之间的通信

在微服务中最重要的事情之一是它们之间的通信。大多数情况下，单个微服务并没有消费者所请求的所有信息，因此微服务需要调用不同的微服务来获取所需的数据。

例如，遵循最后一个例子，对两个用户之间的决斗，如果我们想在同一个调用中提供有关战斗中包含的所有用户的信息，并且我们在 Battle 微服务中没有特定的方法来获取用户信息，我们可以从用户微服务中请求用户信息。为了实现这一点，我们可以使用 PHP 核心功能 cURL，或者使用一个包装 cURL 的外部库，提供一个更简单的接口，比如`GuzzleHttp`。

要在我们的项目中包含`GuzzleHttp`，我们只需要在 Battle 微服务的`composer.json`文件中添加以下行：

```php
    {
        // Code omitted
        "require": {
            "php": ">=5.5.9",
            "laravel/lumen-framework": "5.2.*",
            "vlucas/phpdotenv": "~2.2",
 **"guzzlehttp/guzzle": "~6.0"**
        },
        // Code omitted
    }
```

一旦我们保存了更改，我们可以进入我们的 PHP-FPM 容器并运行以下命令：

```php
**cd /var/www/html && compose update**

```

`GuzzleHttp`将被安装并准备在项目中使用。

为了从`User`微服务中获取用户信息，我们将构建一个方法，将信息提供给`Battle`微服务。目前，我们将把用户信息存储在数据库中，所以我们有一个数组来存储它。在`User`微服务的`app/Http/Controllers/UserController.php`中，我们将添加以下行：

```php
    <?php

    namespace App\Http\Controllers;
    use Illuminate\Http\Request;
    class UserController extends Controller
    {
        **protected $userCache = [**
            **1 => [**
 **'name' => 'John',**
 **'city' => 'Barcelona'**
 **],**
 **2 => [**
 **'name' => 'Joe',**
                **'city' => 'Paris'**
            **]**
        **];**

        /** ... Code omitted ... **/

        **public function get($id)**
        **{**
            **return response()->json(**
 **$this->userCache[$id]**
 **);**
        **}**

        /** ... Code omitted ... **/
    }
```

您可以通过在 Postman 上进行 GET 调用`http://localhost:8084/api/v1/user/2`来测试这种新方法；你应该会得到类似这样的东西：

```php
    {
        "name": "Joe",
        "city": "Paris"
    }
```

一旦我们知道获取用户信息的方法是有效的，我们将从`Battle`微服务中调用它。出于安全原因，Docker 上的每个容器都与其他容器隔离，除非您在`docker-composer.yml`文件的链接部分指定要连接。要这样做，使用以下方法：

+   停止 Docker 容器：

```php
 **docker-compose stop**

```

+   通过添加以下行来编辑`docker-compose.yml`文件：

```php
      microservice_battle_fpm:
          build: ./microservices/battle/php-fpm/
          volumes_from:
          - source_battle
          links:
              - autodiscovery
     **- microservice_user_nginx**
          expose:
              - 9000
          environment:
              - BACKEND=microservice-battle-nginx
              - CONSUL=autodiscovery
```

+   启动 Docker 容器：

```php
 **docker-compose start**

```

从现在开始，`Battle`微服务应该能够看到`User`微服务，所以让我们调用`User`微服务以获取来自 Battle 微服务的用户信息。为此，我们需要在`BattleController.php`文件中包含`GuzzleHttp\Client`，并在 duel 函数中创建一个 Guzzle 实例来使用它：

```php
    <?php
    namespace App\Http\Controllers;
    use Illuminate\Http\Request;
    use App\Algorithm\Dice;
    **use GuzzleHttp\Client;**

    class BattleController extends Controller
    {
        **const USER_ENDPOINT = 'http://microservice_user_nginx/api
        /v1/user/';**
        /** ... Code omitted ... **/

        public function duel(Request $request)
        {
            $this->setBattleAlgorithm();
            $duelResult = $this->battleAlgorithm->fight();
            **$client = new Client(['verify' => false]);**
**            $player1Data = $client->get(                          
            self::USER_ENDPOINT . $request->input('userA'));**
**            $player2Data = $client->get(                          
            self::USER_ENDPOINT . $request->input('userB'));**

            return response()->json(
                [
                    'player1' => **json_decode($player1Data->getBody())**,
                    'player2' => **json_decode($player2Data->getBody())**,
                    'duelResults' => $duelResult
                ]
            );
        }
    }
```

修改完成后，我们可以通过在 Postman 上执行与之前相同的调用来再次测试它--`http://localhost:8081/api/v1/battle/duel`（记得进行 HTTP POST 调用，并发送参数`userA`值为 1 和`userB`值为 2）。响应应该类似于这样（请注意，这次用户信息来自`User`微服务，尽管我们正在调用`Battle`微服务）：

```php
    {
        "player1": {
            "name": "John",
            "city": "Barcelona"
        },
        "player2": {
            "name": "Joe",
            "city": "Paris"
        },
        "duelResults": {
            "player1": 0,
            "player2": 3
        }
    }
```

# 数据库操作

在前几章中，我们解释了您可以为应用程序拥有单个或多个数据库。这是微服务的优势之一；当您意识到某个微服务负载过大时，您可以将数据库分成单个数据库用于特定的微服务，从而实现单个微服务的扩展。

对于我们的示例，我们将为 secrets 微服务创建一个单独的数据库。对于存储软件，我们决定使用**Percona**（一个 MySQL 分支），但请随意使用您喜欢的任何数据库。

在 Docker 中创建数据库容器非常简单。我们只需要编辑我们的`docker-compose.yml`文件，并将`microservice_secret_fpm`服务的链接部分更改为以下内容：

```php
    links:
        - autodiscovery
        - microservice_secret_database
```

在我们所做的更改中，我们告诉 Docker 现在我们的`microservice_secret_fpm`可以与我们的`microservice_secret_database`容器进行通信。让我们创建我们的数据库容器。要做到这一点，我们只需要在`docker-compose.yml`文件中定义服务，如下所示：

```php
    microservice_secret_database:
        build: ./microservices/secret/database/
        environment:
            - CONSUL=autodiscovery
            - MYSQL_ROOT_PASSWORD=mysecret
            - MYSQL_DATABASE=finding_secrets
            - MYSQL_USER=secret
            - MYSQL_PASSWORD=mysecret
        ports:
            - 6666:3306
```

在上述代码中，我们告诉应用程序 Docker 可以在哪里找到`Dockerfile`，我们在其中设置了一些环境变量，并且我们正在将我们机器的端口 6666 映射到容器上的默认 Percona 端口。关于 Docker 和 Percona 官方镜像的一个重要事项是，使用一些特殊的环境变量，容器将为您创建数据库和一些用户。

您可以在我们的 Docker GitHub 存储库中找到所有所需的文件，标签为`chapter-05-basic-database`。

现在我们的容器准备就绪，是时候设置我们的数据库了。Lumen 为我们提供了一个工具来进行迁移和管理迁移，因此我们可以知道我们的数据库是否是最新的，如果我们正在与团队合作。**迁移**是一个用于在我们的数据库中创建和回滚操作的脚本。

要在 Lumen 中进行迁移，首先需要进入您的 secrets PHP-FPM 容器。要做到这一点，您只需要打开终端并执行以下命令：

```php
**docker exec -it docker_microservice_secret_fpm_1 /bin/bash**

```

上述命令将在容器中创建一个交互式终端，并运行 bash 控制台，以便您可以开始输入命令。请确保您在项目根目录下：

```php
**cd /var/www/html**

```

一旦您在项目根目录下，您需要创建一个迁移；可以通过以下命令完成：

```php
**php artisan make:migration create_secrets_table**

```

上述命令将在`database/migrations/2016_11_09_200645_create_secrets_table.php`文件中创建一个空的迁移模板，如下所示：

```php
    <?php
    use Illuminate\Database\Schema\Blueprint;
    use Illuminate\Database\Migrations\Migration;
    class CreateSecretsTable extends Migration
    {
        public function up()
        {
        }
        public function down()
        {
        }
    }
```

上述代码片段是由 artisan 命令生成的示例。如您所见，迁移脚本中有两种方法。在`up()`方法中编写的所有内容都将在执行迁移时使用。在执行回滚时，`down()`方法中的所有内容将用于撤消您的更改。让我们用以下内容填充我们的迁移脚本：

```php
    <?php
    use Illuminate\Database\Schema\Blueprint;
    use Illuminate\Database\Migrations\Migration;
    class CreateSecretsTable extends Migration
    {
        public function up()
        {
            Schema::create(
                'secrets', 
                function (Blueprint $table) {
                    $table->increments('id');
                    $table->string('name', 255);
                    $table->double('latitude');
                    $table->double('longitude')
                        ->nullable();
                    $table->string('location_name', 255);
                    $table->timestamps();
                }
            );
        }
        public function down()
        {
            Schema::drop('secrets');
        }
    }
```

上述示例非常容易理解。在`up()`方法中，我们正在创建一个带有一些列的 secrets 表。这是创建表的一种快速简单的方法，类似于使用`CREATE TABLE` SQL 语句。我们的`down()`方法将撤消所有更改，而在我们的情况下，撤消更改的方法是从我们的数据库中删除 secrets 表。

现在，您可以通过以下命令从终端执行迁移：

```php
**php artisan migrate
Migrated: 2016_11_09_200645_create_secrets_table**

```

迁移命令将运行我们迁移脚本的`up()`方法并创建我们的 secrets 表。

如果您需要了解迁移脚本的执行状态，您可以执行`php artisan migrate:status`，输出将告诉您当前状态：

```php
    +------+----------------------------------------+
    | Ran? | Migration                              |
    +------+----------------------------------------+
    | Y     | 2016_11_09_200645_create_secrets_table |
    +------+----------------------------------------+
```

在这一点上，您可以连接到您的机器的 6666 端口，使用您喜欢的数据库客户端；我们的数据库已经准备好在我们的应用程序中使用。

现在想象一下，您需要对数据库所做的更改进行回滚；在 Lumen 中很容易做到，您只需要运行以下命令：

```php
**php artisan migrate:rollback**

```

一旦我们创建了表，我们可以通过在 Lumen 上进行种子或手动填充我们的表。我们建议您使用种子，因为这是一种轻松跟踪任何更改的方法。要填充我们的新表，我们只需要创建一个新文件`database/seeds/SecretsTableSeeder.php`，其中包含以下内容：

```php
    <?php
    use Illuminate\DatabaseSeeder;
    class SecretsTableSeeder extends Seeder
    {
        public function run()
        {
            DB::table('secrets')->delete();
            DB::table('secrets')->insert([
                [
                    'name' => 'amber', 
                    'latitude' => 42.8805, 
                    'longitude' => -8.54569, 
                    'location_name' => 'Santiago de Compostela'
                ],
                [
                    'name' => 'diamond', 
                    'latitude' => 38.2622, 
                    'longitude' => -0.70107,
                    'location_name' => 'Elche'
                ],
                [
                    'name' => 'pearl', 
                    'latitude' => 41.8919,
                    'longitude' => 2.5113, 
                    'location_name' => 'Rome'
                ],
                [
                    'name' => 'ruby', 
                    'latitude' => 53.4106, 
                    'longitude' => -2.9779, 
                    'location_name' => 'Liverpool'
                ],
                [
                    'name' => 'sapphire', 
                    'latitude' => 50.08804, 
                    'longitude' => 14.42076, 
                    'location_name' => 'Prague'
                ]
            ]);
        }
    }
```

在前面的类中，我们定义了一个`run()`方法，每当我们想要向数据库中填充一些数据时，它都会被执行。在我们的示例中，我们添加了我们在应用程序中硬编码的不同秘密。现在我们的`SecretsTableSeeder`类已经准备好了，我们需要编辑`database/seeds/DatabaseSeeder.php`，以调用我们的自定义 seeder。如果您更改`run()`方法以匹配以下代码片段，您的微服务将具有一些数据：

```php
    public function run()
    {
        $this->call('SecretsTableSeeder');
    }
```

一旦一切就绪，现在是执行 seeder 的时候了，所以再次进入 secrets PHP-FPM 容器，并运行以下命令：

```php
**php artisan db:seed**

```

### 提示

如果`artisan`抛出一个错误，告诉您找不到表，那是由于 composer 自动加载系统。执行`composer dump-autoload`将解决您的问题，然后您可以再次运行`artisan`命令而不会出现任何问题。

在这一点上，您将创建并填充了您的秘密表，其中包含一些示例记录。

在 Lumen 中使用数据库是开箱即用的，并使用 Eloquent 作为 ORM。

**对象关系映射**（**ORM**）是一种编程模型，它将数据库表转换为实体，使开发人员的工作更容易，使他们能够更快地进行基本查询并使用更少的代码。

我们建议在将来要将数据库迁移到不同系统时使用 ORM，以避免语法问题。正如您所知，SQL 语言之间有一些差异--例如，获取确定数量的行的方式：

```php
SELECT * FROM secrets LIMIT 10 //MySQL
SELECT TOP 10 * FROM secrets //SqlServer
SELECT * FROM secrets WHERE rownum<=10; //Oracle
```

因此，如果您使用 ORM，您不需要记住 SQL 语法，因为它抽象了开发人员与数据库操作的关系，开发人员只需要考虑开发。

以下是 ORM 的优势：

+   数据访问层中的安全性防御攻击

+   与数据库一起工作很容易和快速

+   您使用的数据库并不重要

在开发公共 API 时，建议使用 ORM，以避免安全问题，并使查询对团队的其他成员更容易和更清晰。

要设置您的 Lumen 项目与 Eloquent 一起工作，您只需要打开`bootstrap/app.php`文件，并取消注释以下行（大约在第 28 行附近）：

```php
    $app->withEloquent();
```

此外，您需要设置位于`.env.example`文件中的数据库参数，您可以在每个微服务的根文件夹中找到它。编辑文件完成后，您需要将其重命名为`.env`（从文件名中删除`.example`）：

```php
    DB_CONNECTION=mysql
    DB_HOST=microservice_secret_database
    DB_PORT=3306
    DB_DATABASE=finding_secrets
    DB_USERNAME=secret
    DB_PASSWORD=mysecret
```

正如您所看到的，我们在 Docker 中设置的数据库、用户名和密码在数据库操作部分的开始时已经设置好了。

为了使用我们的数据库，我们需要创建我们的模型，因为我们有一个`finding_secrets`数据库，所以在`app/Models/Secret.php`文件中拥有一个秘密模型是有意义的：

```php
    <?php
    namespace App\Model;
    use Illuminate\Database\Eloquent\Model;
    class Secret extends Model
    {
        protected $table    = 'secrets';
        protected $fillable = [
            'name', 
            'latitude', 
            'longitude',                            
            'location_name'
        ];
    }
```

上面的代码非常容易理解；我们只需要定义我们的模型类和数据库`$table`之间的关系以及`$fillable`字段的列表。这是您的模型所需的最少内容。

Fractal 是一个为我们的 RESTful API 提供演示和转换层的库。使用这个库将使我们的响应保持一致，美观和干净。要安装这个库，我们只需要打开我们的 PHP-FPM 容器的`composer.json`，将`"league/fractal": "⁰.14.0"`添加到所需元素的列表中，并执行`composer update`。

### 提示

安装 fractal 的另一种方法是在您的 PHP-FMP 终端上运行以下命令：`composer require league/fractal`。请注意，此命令将使用最新版本，可能与我们的示例不兼容。

现在安装了 fractal，现在是时候定义我们的秘密转换器了。您可以将转换器视为一种简单的方式，将模型转换为一个一致的响应。在您的 IDE 中创建`app/Transformers/SecretTransformer.php`文件，并插入以下内容：

```php
    <?php
    namespace App\Transformers;
    use App\Model\Secret;
    use League\Fractal\Transformer\Abstract;
    class SecretTransformer extends TransformerAbstract
    {
        public function transform(Secret $secret)
        {
            return [
                'id'        => $secret->id,
                'name'      => $secret->name,
                'location'  => [
                    'latitude'  => $secret->latitude,
                    'longitude' => $secret->longitude,
                    'name'      => $secret->location_name
                ]
            ];
        }
    }
```

从上述代码中可以看出，我们正在指定秘密模型的转换，因为我们希望所有位置都被分组，所以我们在位置密钥内添加了秘密的所有位置信息。将来，如果您需要添加新字段或修改结构，现在一切都在一个地方，将会让作为开发人员的生活变得轻松。

为了示例目的，我们将修改我们的 secrets 控制器的 index 方法，以使用 fractal 从数据库返回响应。打开您的`app/Http/Controllers/SecretController.php`并插入以下用法：

```php
    use App\Model\Secret;
    use App\Transformers\SecretTransformer;
    use League\Fractal\Manager;
    use League\Fractal\Resource\Collection;
```

现在，您需要更改`index()`如下：

```php
    public function index(
        Manager $fractal, 
        SecretTransformer $secretTransformer, 
        Request $request)
    {
        $records = Secret::all();
        $collection = new Collection(
            $records, 
            $secretTransformer
        );
        $data = $fractal->createData($collection)
            ->toArray();

        return response()->json($data);
    }
```

首先，我们在方法签名中添加了一些我们将需要的对象实例，由于 Lumen 内置了依赖注入，我们不需要做任何额外的工作。它们将准备好在我们的方法内使用。我们的方法定义了以下内容：

+   从数据库获取所有秘密记录

+   使用我们的转换器创建一个秘密集合

+   fractal 库从我们的集合创建一个数据数组

+   我们的控制器将我们转换后的集合作为 JSON 返回

如果您在 Postman 中尝试，响应将类似于这样：

```php
    {
        "data": [
            {
                "id": 1,
                "name": "amber",
                "location": {
                    "latitude": 42.8805,
                    "longitude": -8.54569,
                    "name": "Santiago de Compostela"
                }
            },

            /** Code omitted ** /
            {
                "id": 5,
                "name": "sapphire",
                "location": {
                    "latitude": 50.08804,
                    "longitude": 14.42076,
                    "name": "Prague"
                }
            }
        ]
    }
```

我们所有的记录现在以一致的方式从数据库返回，所有都在我们的`"data"`响应键内具有相同的结构。

# 错误处理

在接下来的部分中，我们将解释如何验证我们微服务中的输入数据以及如何处理可能的错误。过滤我们收到的请求非常重要，不仅是为了通知消费者请求无效，还要避免安全问题或我们不希望的参数。

## 验证

Lumen 有一个很棒的验证系统，所以我们不需要安装任何东西来验证我们的数据。请注意，以下验证规则可以放在`routes.php`或控制器内的每个函数中。我们将在函数内使用它以更清晰地使用。

要使用我们的数据库进行验证系统，我们需要对其进行配置。这非常简单；我们只需要在根目录中创建一个`config/database.php`文件（和文件夹），并插入以下代码：

```php
    <?php
    return [
        'default'     => 'mysql',
        'connections' => [
            'mysql' => [
                'driver'    => 'mysql',
                'host'      => env('DB_HOST'),
                'database'  => env('DB_DATABASE'),
                'username'  => env('DB_USERNAME'),
                'password'  => env('DB_PASSWORD'),
                'collation' => 'utf8_unicode_ci'
            ]
        ]
    ];
```

然后，您需要在`bootstrap/app.php`文件中添加数据库行：

```php
    $app->withFacades();
    $app->withEloquent();
    **$app->configure('database');**

```

完成此操作后，Lumen 验证系统已准备就绪。因此，让我们编写规则来验证创建`Secrets`微服务上的新秘密的 POST 方法：

```php
    public function create(Request $request)
    {
        **$this->validate(**
 **$request,**
 **[**
 **'name'          => 'required|string|unique:secrets,name',**
 **'latitude'      => 'required|numeric',**
**            'longitude'     => 'required|numeric',**
**            'location_name' => 'required|string'**
 **]**
**        );**

```

在上述代码中，我们确认参数应该通过规则。字段`'name'`是必需的；它是一个字符串，而且在`secrets`表中应该是唯一的。字段`'latitude'`和`'longitude'`是数字且也是必需的。此外，`'location_name'`字段也是必需的，它是一个字符串。

### 提示

在 Lumen 文档中（[`lumen.laravel.com/docs`](https://lumen.laravel.com/docs)），您可以查看所有可用的选项来验证您的输入。

您可以在 Postman 中尝试它；创建一个带有以下`application/json`参数的 POST 请求来检查插入失败（请注意，您也可以像表单数据键值一样发送它）：

```php
    {
        "name": "amber",
        "latitude":"1.23",
        "longitude":"-1.23",
        "location_name": "test"
    }
```

上述请求将尝试验证一个与之前记录相同名称的新密钥。根据我们的验证规则，我们不允许消费者创建具有相同名称的新密钥，因此我们的微服务将以`422`错误响应，并返回以下内容：

```php
    {
        "name": [
            "The name has already been taken."
        ]
    }
```

请注意，状态码（或错误码）对于通知您的消费者其请求发生了什么非常重要；如果一切正常，应该返回`200`状态码。Lumen 默认返回`200`状态码。

在第十一章*最佳实践和约定*中，我们将看到您可以在应用程序中使用的所有可用代码的完整列表。

一旦验证规则通过，我们应该将数据保存在数据库中。这在 Lumen 中非常简单，只需这样做：

```php
    $secret = Secret::create($request->all());
    if ($secret->save() === false) {
        // Manage Error
    }
```

完成后，我们将在数据库中获得我们的新记录。Lumen 提供了其他方法来创建其他任务，如填充、更新或删除。

## 管理异常

有必要知道，我们必须管理应用程序中发生的可能错误。为此，Lumen 为我们提供了可以使用的异常列表。

因此，现在我们将尝试在尝试调用另一个微服务时获得异常。为此，我们将从用户微服务调用密钥微服务。

请记住，出于安全原因，如果您没有将一个容器与另一个容器链接起来，它们就无法相互看到。编辑您的`docker-compose.yml`，并从`microservice_user_fpm`到`microservice_secret_nginx`添加链接，如下所示：

```php
    microservice_user_fpm:
        build: ./microservices/user/php-fpm/
        volumes_from:
            - source_user
        links:
            - autodiscovery
 **- microservice_secret_nginx**
        expose:
            - 9000
        environment:
            - BACKEND=microservice-user-nginx
            - CONSUL=autodiscovery
```

现在，您应该再次启动您的容器：

```php
**docker-compose start**

```

还要记住，我们需要像之前在`Battle`微服务和`User`微服务上一样安装`GuzzleHttp`，以便调用`Secret`微服务。

我们将在`User`微服务中创建一个新的函数，以显示`user`钱包中保存的秘密。

将此添加到`app/Http/routes.php`：

```php
    $app->get(
        'user/{id}/wallet', 
        'UserController@getWallet'
    );
```

然后，我们将创建一个从`user`钱包中获取秘密的方法--例如，看一下这个：

```php
    public function getWallet($id)
    {
        /* ... Code ommited ... */
        $client = new Client(['verify' => false]);
        try {
            $remoteCall = $client->get(
                'http://microservice_secret_nginx                                       /api/v1/secret/1');
        } catch (ConnectException $e) {
            /* ... Code ommited ... */
            throw $e;
        } catch (ServerException $e) {
            /* ... Code ommited ... */
        } catch (Exception $e) {
            /* ... Code ommited ... */
        }
          /* ... Code ommited ... */
    }
```

我们正在调用`Secret`微服务，但我们将修改 URI 以获得`ConnectException`，所以请修改它：

```php
    $remoteCall = $client->get(
        **'http://this_uri_is_not_going_to_work'
    **);
```

在 Postman 上试一试；您将收到一个`ConnectException`错误。

现在，再次正确设置 URI，并在密钥微服务端放入一些错误代码：

```php
    public function get($id)
    {
      this_function_does_not_exist();
    }
```

上述代码将为密钥微服务返回错误**500**；但我们是从`User`微服务调用它，所以现在我们将收到`ServerException`错误。

在 Lumen 中，通过捕获它们来处理所有异常的类是`Handler`类（位于`app/Exceptions/Handler.php`）。这个类有两个定义的方法：

+   `report()`: 这允许我们将异常发送到外部服务--例如，一个集中的日志系统。

+   `render()`: 这将我们的异常转换为 HTTP 响应。

我们将更新`render()`方法以返回自定义错误消息和错误码。想象一下，我们想捕获 Guzzle 的`ConnectException`并返回一个更友好和易于管理的错误。看一下以下代码：

```php
    /** Code omitted **/
    use SymfonyComponentHttpFoundationResponse;
    use GuzzleHttpExceptionConnectException;

    /** Code omitted **/

    public function render($request, Exception $e)
    {
        switch ($e) {
            case ($e instanceof ConnectException) :
                return response()->json(
                    [
                        'error' => 'connection_error',
                        'code'  => '123'
                    ],
                    Response::HTTP_SERVICE_UNAVAILABLE
                );
                break;
            default :
                return parent::render($request, $e);
               break;
        }   
    }
```

在这里，我们正在检测 Guzzle 的`ConnectException`并提供自定义的错误消息和代码。使用这种策略有助于我们知道哪里出了问题，并允许我们根据我们正在处理的错误采取行动。例如，我们可以将代码`123`分配给所有连接错误；因此，当我们检测到这个问题时，我们可以避免其他服务的级联故障或通知开发人员。

# 异步和队列

在微服务中，队列是帮助提高性能和减少执行时间的最重要的事情之一。

例如，如果您需要在客户完成应用程序的注册流程时向客户发送电子邮件，应用程序不需要立即发送它；它可以放入队列中，在服务器不太忙的时候几秒钟后发送。此外，它是异步的，因为客户不需要等待电子邮件。应用程序将显示消息*注册完成*，并且电子邮件将被放入队列并同时处理。

另一个例子是当您需要处理非常繁重的工作负载时，您可以有一台专用的硬件更好的机器来执行这些任务。

最著名的内存数据结构存储之一是**Redis**。您可以将其用作数据库、缓存层、消息代理，甚至作为队列存储。Redis 的关键点之一是它支持不同的结构类型，例如字符串、哈希、列表和有序集等。这个软件被设计成易于管理和具有高性能，因此它是 Web 行业的事实标准。

Redis 的主要用途之一是作为缓存存储。您可以永久存储数据，也可以添加过期时间，而无需担心何时需要删除数据；Redis 会为您完成。由于易用性、良好的支持和可用的库，Redis 适用于任何规模的项目。

我们将在`User`微服务上构建一个示例，使用基于 Redis 的队列发送电子邮件。

Lumen 为我们提供了使用数据库的队列系统；但也有其他选项可用，可以使用外部服务。在我们的示例中，我们将使用 Redis，因此让我们看看如何在 Docker 上安装它。

打开`docker-compose.yml`并添加以下容器描述：

```php
    microservice_user_redis:
        build: ./microservices/user/redis/
        links:
            - autodiscovery
        expose:
            - 6379
        ports:
            - 6379:6379
```

您还需要更新`microservice_user_fpm`容器的链接部分以匹配以下内容：

```php
    links:
        - autodiscovery
        - microservice_secret_nginx
        - microservice_user_redis
```

在前面的代码片段中，我们为 Redis 定义了一个新的容器，并将其链接到`microservice_user_fpm`容器。现在打开`microservices/user/redis/Dockerfile`文件，并添加以下代码以使最新的 Redis 版本可用：

```php
    FROM redis:latest
```

要在我们的 Lumen 项目中使用 Redis，我们需要通过 composer 安装一些依赖项。因此，打开您的`composer.json`，并将以下行添加到所需部分，然后在用户 PHP-FPM 容器内执行 composer update：

```php
    "predis/predis": "~1.0",
    "illuminate/redis": "5.2.*"
```

对于电子邮件支持，您只需要将以下行添加到 composer.json 文件的 require 部分：

```php
    "illuminate/mail": "5.2.*"
```

安装完 Redis 后，我们需要设置环境。首先，我们需要在`.env`文件中添加以下行：

```php
    QUEUE_DRIVER=redis
    CACHE_REDIS_CONNECTION=default
    REDIS_HOST=microservice_user_redis
    REDIS_PORT=6379
    REDIS_DATABASE=0
```

现在，我们需要在`config/database.php`文件中添加 Redis 配置；如果您添加了其他数据库（例如 MySQL），请将其放在那之后，但在返回数组内部：

```php
    <?php
    return [
        'redis' => [
            'client'  => 'predis',
            'cluster' => false,
            'default' => [
                'host'     => env('REDIS_HOST', 'localhost'),
                'password' => env('REDIS_PASSWORD', null),
                'port'     => env('REDIS_PORT', 6379),
                'database' => env('REDIS_DATABASE', 0),
            ],
        ]
    ];
```

还需要将`vendor/laravel/lumen-framework/config/queue.php`文件复制到`config/queue.php`。

最后，不要忘记在`bootstrap/app.php`文件上注册所有内容，并添加以下行，这样我们的应用程序就能够读取我们刚刚设置的配置了：

```php
    $app->register(
        Illuminate\Redis\RedisServiceProvider::class
    );
    $app->configure('database');
    $app->configure('queue');
```

现在，我们将解释如何在我们的`User`微服务中构建一个队列。想象一下，在应用程序中，当创建新用户时，我们希望将第一个秘密作为礼物赠送给他们；因此，在用户创建之后，我们将调用秘密微服务以获取用户的第一个秘密。这不是一个优先级很高的任务，这就是为什么我们将使用队列来执行此任务的原因。

创建一个新文件`app/Jobs/GiftJob.php`，其中包含以下代码：

```php
    <?php
    namespace AppJobs;
    use GuzzleHttpClient;
    class GiftJob extends Job
    {
        public function __construct()
        {
        }

        public function handle()
        {
            $client = new Client(['verify' => false]);
            $remoteCall = $client->get(
                'http://microservice_secret_nginx                                                     /api/v1/secret/1'
            );
            /* Do stuff with the return from a remote service, for 
            example save it in the wallet */
        }
    }
```

您可以修改类构造函数以向作业传递数据，例如，包含所有用户信息的对象实例。

现在，我们需要从我们的`app/Http/Controllers/UserController.php`控制器实例化作业：

```php
**    use AppJobsGiftJob;**
    public function create(Request $request)
    {
        /* ... Code omitted (validate & save data) ... */
     **$this->dispatch(new GiftJob());**
        /* ... Code omitted ... */
    }
```

一旦队列任务完成，我们必须在后台启动队列工作程序。以下代码将为您完成这项工作，并且它将一直运行直到线程死亡，您可以添加一个监督程序来确保队列继续工作：

```php
    php artisan queue:work
```

您可以通过调用`http://localhost:8084/api/v1/user`在 Postman 上尝试一下。一旦您调用此方法，Lumen 将把工作放在 Redis 上，并且它将可供队列工作者使用。一旦工作者从 Redis 获取并处理任务，您将在终端中看到以下下一个消息：

```php
**    [2016-11-13 17:59:23] Processed: AppJobsGiftJob**

```

Lumen 为我们提供了更多的队列可能性。例如，您可以为队列设置优先级，为作业指定超时，甚至为任务设置延迟。您可以在 Lumen 文档中找到这些信息。

# 缓存

许多时候，消费者请求相同的内容，应用程序返回相同的信息。在这种情况下，缓存是避免不断处理相同请求并更快地返回所需数据的解决方案。

缓存用于不经常更改的数据，以便在不处理请求的情况下获得预先计算的响应。工作流程如下：

1.  消费者第一次请求某些信息时，应用程序会处理请求并获取所需的数据。

1.  它将请求所需的数据保存在缓存中，并设置我们定义的过期时间。

1.  它将数据返回给消费者。

下一次消费者请求某些内容时，您需要执行以下操作：

1.  检查请求是否在应用程序缓存中，并且尚未过期。

1.  返回缓存中的数据。

因此，在我们的示例中，我们将在位置微服务中使用缓存，以避免多次请求最接近的秘密。

我们应用程序中需要使用缓存层的第一件事是具有一个带有 Redis 的容器（您可以在其他地方找到其他缓存软件，但我们决定使用 Redis，因为它非常容易安装和使用）。打开`docker-compose.yml`文件，并添加新的容器定义，如下所示：

```php
    microservice_location_redis:
        build: ./microservices/location/redis/
        links:
            - autodiscovery
        expose:
            - 6379
        ports:
            - 6380:6379
```

一旦我们添加了容器，您需要更新`microservice_location_fpm`定义的链接部分，以连接到我们的新 Redis 容器，如下所示：

```php
    links:
        - autodiscovery
     **- microservice_location_redis**

```

在这种情况下，我们的`docker/microservices/location/redis/Dockerfile`文件将只包含以下内容（如果需要，可以随意向容器添加更多内容）：

```php
    FROM redis:latest
```

不要忘记执行`docker-compose stop`以成功终止所有容器，并使用`docker-compose up -d`再次启动它们以应用我们的更改。您可以通过在终端中执行`docker ps`来检查新容器是否正在运行。

现在是时候对我们的位置微服务源代码进行一些更改，以使用我们的新缓存层。我们需要做的第一个更改是在`composer.json`中；将以下所需的库添加到`"require"`部分：

```php
    "predis/predis": "~1.0",
    "illuminate/redis": "5.2.*"
```

一旦您对`composer.json`文件进行更改，请记得执行`composer update`以获取库。

现在，打开位置微服务的`.env`文件，添加 Redis 设置，如下所示：

```php
    CACHE_DRIVER=redis
    CACHE_REDIS_CONNECTION=default
    REDIS_HOST=microservice_location_redis
    REDIS_PORT=6379
    REDIS_DATABASE=0
```

由于我们的环境变量现在已设置好，我们需要创建`config/database.php`，内容如下：

```php
    <?php
    return [
        'redis' => [
            'client'  => 'predis',
            'cluster' => false,
            'default' => [
                'host'     => env('REDIS_HOST', 'localhost'),
                'password' => env('REDIS_PASSWORD', null),
                'port'     => env('REDIS_PORT', 6379),
                'database' => env('REDIS_DATABASE', 0),
            ],
        ]
    ];
```

在上述代码中，我们定义了如何连接到我们的 Redis 容器。

Lumen 没有缓存配置，因此您可以将`vendor/laravel/lumen-framework/config/cache.php`文件复制到`config/cache.php`中。

我们需要对`bootstrap/app.php`进行一些小的调整--取消注释`$app->withFacades();`并添加以下行：

```php
    $app->configure('database');
    $app->configure('cache');
    $app->register(
        Illuminate\Redis\RedisServiceProvider::class
    );
```

我们将更改我们的`getClosestSecrets()`方法，以使用缓存而不是每次计算最接近的秘密。打开`app/Http/Controllers/LocationController.php`文件，并添加缓存所需的使用：

```php
**use Illuminate\Support\FacadesCache;** 
        /* ... Omitted code ... */
    **const DEFAULT_CACHE_TIME = 1;**

    public function getClosestSecrets($originPoint)
    {
     **$cacheKey = 'L' . $originPoint['latitude'] .                   
        $originPoint['longitude'];**
        **$closestSecrets = Cache::remember(
            $cacheKey,
            self::DEFAULT_CACHE_TIME,
            function () use($originPoint) {**
                $calculatedClosestSecrets = [];
                $distances = array_map(
                    function($item) use($originPoint) {
                        return $this->getDistance(
                            $item['location'], 
                            $originPoint
                        );
                    }, 
                    self::$cacheSecrets
                );
                asort($distances);
                $distances = array_slice(
                    $distances, 
                    0,
                    self::MAX_CLOSEST_SECRETS, 
                    true
                );
                foreach ($distances as $key => $distance) {
                    $calculatedClosestSecrets[] = 
                    self::$cacheSecrets[$key];
                }

                return $calculatedClosestSecrets;
         **});**
 **return $closestSecrets;**
    }
    /* ... Omitted code ... */
```

在上述代码中，我们通过添加缓存层改变了方法的实现；因此，我们首先使用`remember()`检查我们的缓存，而不是总是计算最接近的点。如果缓存中没有返回任何内容，我们进行计算并存储结果。

在 Lumen 缓存中保存数据的另一个选项是使用`Cache::put('key', 'value', $expirationTime);`，其中`$expirationTime`可以是以分钟为单位的时间（整数）或日期对象。

### 提示

密钥由您定义，因此一个好的做法是生成一个您可以记住的密钥，以便将来重新生成。在我们的示例中，我们使用`L`（表示位置），然后是`纬度`和`经度`来定义密钥。然而，如果您要保存一个 ID，它应该作为密钥的一部分包含在内。

在 Lumen 中，与我们的缓存层一起工作很容易。

要从缓存中获取元素，可以使用`"get"`。它允许两个参数--第一个是指定您想要的密钥（必需的），第二个是在缓存中未存储密钥时要使用的值（显然是可选的）：

```php
    $value = Cache::get('key', 'default');
```

存储数据的类似方法是`Cache::forever($cacheKey, $cacheValue);`，这个调用将永久地将$cacheValue 存储在我们的缓存层中，直到您删除或更新它。

如果您没有为存储的元素指定过期时间，那么了解如何删除它们就很重要。在 Lumen 中，如果您知道分配给元素的$cacheKey，可以使用`Cache::forget($cacheKey);`来删除它。如果需要删除缓存中存储的所有元素，可以使用简单的`Cache::flush();`来实现。

# 总结

在本章中，您已经学会了如何开发基于微服务的应用程序的不同部分。现在，您已经具备了处理数据库存储、缓存、微服务之间的通信、队列以及从入口点到应用程序（路由）的请求工作流程以及数据验证的必要知识，直到将数据提供给消费者的时间。在下一章中，您将学习如何监控您的应用程序，以避免和解决应用程序执行过程中发生的问题。
