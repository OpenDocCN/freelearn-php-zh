# 第七章。构建异步微服务架构

在本章中，我们将构建一个由一组小型独立组件组成的应用程序，这些组件通过网络协议进行通信。通常，这些所谓的**微服务架构**是使用基于 HTTP 的通信协议构建的，通常以 RESTful API 的形式，我们已经在第五章中实现了*创建 RESTful Web 服务*。

在本章中，我们将探讨一种关注异步性、松散耦合和高性能的替代通信协议：**ZeroMQ**，而不是专注于 REST。我们将使用 ZeroMQ 为一个（完全虚构的）电子商务场景构建一个简单的**结账服务**，该服务将处理广泛的问题，从电子邮件消息、订单处理、库存管理等等。

# 目标架构

我们的微服务架构的中心服务将是结账服务。该服务将为许多电子商务系统共有的结账流程提供 API。对于每个结账流程，我们将需要以下输入数据：

+   一个可以包含任意数量的文章的**购物车**

+   客户的**联系数据**

然后，结账服务将负责执行实际的结账流程，其中涉及多个额外的服务，每个服务处理结账流程的单个步骤或关注点：

1.  我们虚构的电子商务企业将处理实物商品（或更抽象的商品，我们只能有限数量地存货）。因此，对于购物车中的每件商品，结账服务将需要确保所需数量的商品实际上有库存，并且如果可能的话，减少相应数量的可用库存。这将是**库存服务**的责任。

1.  成功完成结账流程后，用户需要通过电子邮件收到有关成功结账的通知。这将是**邮件服务**的责任。

1.  此外，在完成结账流程后，订单必须转发给一个开始为该订单发货的运输服务。

以下图表显示了本章所需的目标架构的高层视图：

### 注意

在本章中，重点将放在使用 ZeroMQ 实现不同服务之间的通信模式上。我们不会实现实际的业务逻辑，这需要实际工作（因为您完全可以用另一本书来填补这部分）。相反，我们将实现实际服务作为提供我们希望它们实现的 API 的简单存根，但只包含实际业务逻辑的原型实现。

![目标架构](img/image_07_001.jpg)

我们应用的目标架构

图中所示接口旁边的标签（**RES**和**PUB**）是您将在本章中了解的不同 ZeroMQ 套接字类型。

# ZeroMQ 模式

在本章中，您将了解 ZeroMQ 支持的基本通信模式。如果这些听起来有点理论化，不要担心；您将在整个章节中自己实现所有这些模式。

## 请求/回复模式

ZeroMQ 库支持各种不同的通信模式。对于每种模式，您将需要不同的 ZeroMQ 套接字类型。最简单的通信模式是请求/回复模式，其中客户端打开一个 REQ 套接字并连接到监听 REP 套接字的服务器。客户端发送一个请求，然后服务器进行回复。

![请求/回复模式](img/image_07_002.jpg)

ZeroMQ 请求/回复套接字

重要的是要知道，REQ 和 REP 套接字始终是*同步*的。每个 REQ 套接字一次只能向单个 REP 套接字发送请求，更重要的是，每个 REP 套接字也只能连接到单个 REQ 套接字。ZeroMQ 库甚至在协议级别强制执行这一点，并在 REQ 套接字尝试在回复当前请求之前接收新请求时触发错误。我们将在以后使用高级通信模式来解决这个限制。

## 发布/订阅模式

发布/订阅模式由一个 PUB 套接字组成，可以在其上发布消息。可以连接任意数量的 SUB 套接字到此套接字。当在 PUB 套接字上发布新消息时，它将转发到所有连接的 SUB 套接字。

![发布/订阅模式](img/image_07_003.jpg)

发布/订阅套接字

PUB/SUB 架构中的每个订阅者都需要指定至少一个订阅 - 一个作为每条消息过滤器的字符串。发布者将根据订阅者过滤消息，以便每个订阅者只接收他们订阅的消息。

发布/订阅严格单向工作。发布者无法从订阅者那里接收消息，订阅者也无法将消息发送回发布者。然而，就像多个 SUB 套接字可以连接到单个 PUB 套接字一样，单个 SUB 套接字也可以连接到多个 PUB 套接字。

## 推/拉模式

推/拉模式与发布/订阅模式类似。PUSH 套接字用于向任意数量的 PULL 套接字发布消息（就像 PUB/SUB 一样，单个 PULL 套接字也可以连接到任意数量的 PUSH 套接字）。然而，与发布/订阅模式相反，发送到 PUSH 套接字的每条消息只会分发到连接的 PULL 套接字中的一个。这种行为使得 PUSH/PULL 模式非常适合实现工作池，例如，您可以使用它来将任务分发给任意数量的工作者以并行处理。同样，PULL 套接字也可以用于从任意数量的 PUSH 套接字收集结果（这可能是从工作池返回的结果）。

使用 PUSH/PULL 套接字将任务分发给工作池，然后使用第二个 PUSH/PULL 层从该池中收集结果到单个套接字中，也称为*扇出/扇入*。

![推/拉模式](img/image_07_004.jpg)

使用 PUSH 和 PULL 套接字实现扇出/扇入架构

# 引导项目

像往常一样，我们将从为本章的项目进行引导开始。在 PHP 应用程序中使用 ZeroMQ 库，您将需要通过 PECL 安装的**php-zmq 扩展**。您还需要包含 ZeroMQ 库的 C 头文件的`libzmq-dev`软件包。您可以通过操作系统的软件包管理器安装它。以下命令将在 Ubuntu 和 Debian Linux 上都适用：

```php
**$ apt-get install libmzq-dev**
**$ pecl install zmq-beta**

```

像往常一样，我们将使用 composer 来管理我们的 PHP 依赖项，并使用 Docker 来管理所需的系统库。由于我们的应用程序将由多个在多个进程中运行的服务组成，我们将使用多个 composer 项目和多个 Docker 镜像。

如果您正在使用 Windows，并希望在不使用 Docker 的情况下本地运行 ZeroMQ/PHP 应用程序，您可以从 PECL 网站（[`pecl.php.net/package/zmq/1.1.3/windows`](https://pecl.php.net/package/zmq/1.1.3/windows)）下载 ZeroMQ 扩展。

我们所有的服务将使用相同的软件（安装了 ZeroMQ 扩展的 PHP）。我们将从实现库存服务开始，但您将能够在本示例中创建的所有服务中使用相同的 Docker 镜像（或至少相同的 Dockerfile）。首先，在项目目录中创建一个`inventory/Dockerfile`文件，内容如下：

```php
FROM php:7 
RUN apt-get update && apt-get install -y libzmq-dev 
RUN docker-php-ext-configure pcntl && \ 
    docker-php-ext-install pcntl && \ 
    pecl install ev-beta && docker-php-ext-enable ev && \ 
    pecl install zmq-beta && docker-php-ext-enable zmq 
WORKDIR /opt/app 
ONBUILD ADD . /opt/app 
CMD ["/usr/local/bin/php", "server.php"] 

```

您会注意到我们还安装了`pcntl`和`ev`扩展。您已经在第六章中使用过`ev`扩展，*构建聊天应用程序*。它提供了一个与我们稍后在本章中将使用的`react/zmq`库很好配合的异步事件循环。`pcntl`扩展提供了一些功能，将帮助您控制后续长时间运行的 PHP 进程的进程状态。

为了使生活更轻松，您还可以在项目目录中创建一个`docker-compose.yml`文件，以便使用 Docker compose 来管理应用程序中的众多容器。一旦您有了可以在容器中运行的第一个服务，我们将介绍这一点。

# 构建库存服务

我们将从实现库存服务开始，因为它将使用简单的请求/回复模式进行通信，而且没有其他依赖关系。

## 开始使用 ZeroMQ REQ/REP 套接字

首先在`inventory/`目录中创建服务的`composer.json`文件：

```php
{ 
  "name": "packt-php7/chp7-inventory", 
  "type": "project", 
  "authors": [{ 
    "name": "Martin Helmich", 
    "email": "php7-book@martin-helmich.de" 
  }], 
  "require": { 
    "php": ">= 7.0", 
    "ext-zmq": "*" 
  }, 
  "autoload": { 
    "psr-4": { 
      "Packt\\Chp7\\Inventory": "src/" 
    } 
  } 
} 

```

创建`composer.json`文件后，在`inventory/`目录中使用`composer install`命令来安装项目的依赖项。

让我们首先为库存创建一个`server.php`文件。就像第六章中的 Ratchet 应用程序一样，这个文件稍后将成为我们的主服务器进程 - 请记住，在这个例子中，我们甚至没有使用 HTTP 作为通信协议，因此没有 Web 服务器，也没有 FPM 涉及到任何地方。

每个 ZeroMQ 应用程序的起点是上下文。上下文存储了 ZeroMQ 库需要维护套接字和与其他套接字通信所需的各种状态。然后，您可以使用此上下文创建一个新的套接字，并将此上下文绑定到一个端口：

```php
$args = getopt('p:', ['port=']); 
$ctx = new ZMQContext(); 

$port = $args['p'] ?? $args['port'] ?? 5557; 
$addr = 'tcp://*:' . $port; 

$sock = $ctx->getSocket(ZMQ::SOCKET_REP); 
$sock->bind($addr); 

```

这段代码创建了一个新的 ZeroMQ REP 套接字（可以回复请求的套接字），并将此套接字绑定到可配置的 TCP 端口（默认为 5557）。现在您可以在此套接字上接收消息并回复它们：

```php
while($message = $sock->recv()) { 
    echo "received message '" . $message . "'\n"; 
    $sock->send("this is my response message"); 
} 

```

正如您所看到的，这个循环将无限期地轮询新消息，然后对其进行响应。套接字的`recv()`方法将阻塞脚本执行，直到接收到新消息（稍后您可以使用`react/zmq`库轻松实现非阻塞套接字，但现在这就足够了）。

为了测试您的 ZeroMQ 服务器，您可以在`inventory/`目录中创建第二个文件`client.php`，在其中可以使用 REQ 套接字向服务器发送请求：

```php
$args = getopt('h', ['host=']); 
$ctx = new ZMQContext(); 

$addr = $args['h'] ?? $args['host'] ?? 'tcp://127.0.0.1:5557'; 

$sock = $ctx->getSocket(ZMQ::SOCKET_REQ); 
$sock->connect($addr); 

$sock->send("This is my request"); 
var_dump($sock->recv()); 

```

当您的服务器脚本正在运行时，您可以简单地运行`client.php`脚本来连接到服务器的 REP 套接字，发送请求，并等待服务器的回复。就像 REP 套接字一样，REQ 套接字的`recv`方法也会阻塞，直到从服务器接收到回复。

如果您正在使用 Docker compose 来管理开发环境中的众多容器（目前只有一个，但将会有更多），请将以下部分添加到您的`docker-compose.yml`文件中：

```php
inventory: 
  build: inventory 
  ports: 
    - 5557 
  volumes: 
    - inventory:/usr/src/app 

```

在`docker-compose.yml`配置文件中添加库存服务后，您可以通过在命令行上运行以下命令来启动容器：

```php
**$ docker-compose up**

```

## 使用 JsonRPC 进行通信

现在我们有一个服务器，可以从客户端接收文本消息，然后将响应发送回该客户端。但是，为了构建一个可工作且易于维护的微服务架构，我们需要一种协议和格式，使这些消息可以遵循，并且所有服务都可以达成一致。在微服务架构中，这个共同点通常是 HTTP，其丰富的协议语义可用于轻松构建 REST Web 服务。但是，ZeroMQ 作为一种协议要低级得多，不涉及不同的请求方法、标头、缓存以及 HTTP 所附带的所有其他功能。

我们将库存服务实现为一个简单的**远程过程调用**（**RPC**）服务，而不是一个 RESTful 服务。一个快速简单的格式是 JSON-RPC，它使用 JSON 消息实现 RPC。使用 JSON-RPC，客户端可以使用以下 JSON 格式发送方法调用：

```php
{ 
  "jsonrpc": "2.0", 
  "method": "methodName", 
  "params": ["foo", "bar", "baz"], 
  "id": "some-random-id" 
} 

```

服务器随后可以使用以下格式响应此消息：

```php
{ 
  "jsonrpc": "2.0", 
  "id": "id from request", 
  "result": "the result value" 
} 

```

或者，当处理过程中发生错误时：

```php
{ 
  "jsonrpc": "2.0", 
  "id": "id from request", 
  "error": { 
    "message": "the error message", 
    "code": 1234 
  } 
} 

```

这个协议相对简单，我们可以很容易地在 ZeroMQ 之上实现它。为此，首先创建一个新的`Packt\Chp7\Inventory\JsonRpcServer`类。这个服务器将需要一个 ZeroMQ 套接字，还需要一个对象，该对象提供客户端应该能够使用 RPC 调用的方法：

```php
namespace Packt\Chp7\Inventory; 

class JsonRpcServer 
{ 
    private $socket; 
    private $server; 

    public function __construct(\ZMQSocket $socket, $server) 
    { 
        $this->socket = $socket; 
        $this->server = $server; 
    } 
} 

```

我们现在可以实现一个方法，接收来自套接字的消息，尝试将它们解析为 JSON-RPC 消息，并调用`$server`对象上的相应方法，并返回该方法的结果值：

```php
public function run() 
{ 
    while ($msg = $this->socket->recv()) { 
        $resp = $this->handleMessage($msg); 
        $this->socket->send($resp); 
    } 
} 

```

与前面的例子一样，这个方法将无限运行，并处理任意数量的请求。现在，让我们来看看`handleMessage`方法：

```php
private function handleMessage(string $req): string { 
    $json   = json_decode($req); 
    $method = [$this->server, $json->method]; 

    if (is_callable($method)) { 
        $result = call_user_func_array($method, $json->params ?? []); 
        return json_encode([ 
            'jsonrpc' => '2.0, 
            'id'      => $json->id, 
            'result'  => $result 
        ]); 
    } else { 
        return json_encode([ 
            'jsonrpc' => '2.0', 
            'id'      => $json->id, 
            'error'   => [ 
                'message' => 'uncallable method ' . $json->method, 
                'code'    => -32601 
            ] 
        ]); 
    } 
} 

```

这个方法检查`$this->server`对象是否有一个与 JSON-RPC 请求的`method`属性相同的可调用方法。如果是，将使用请求的`param`属性作为参数调用此方法，并将返回值合并到 JSON-RPC 响应中。

目前，这个方法仍然缺少一些基本的异常处理。一个未处理的异常，一个致命错误可以终止整个服务器进程，所以我们在这里需要特别小心。首先，我们需要确保传入的消息确实是一个有效的 JSON 字符串：

```php
private function handleMessage(string $req): string { 
    $json   = json_decode($req); 
 **if (json_last_error()) {** 
 **return json_encode([** 
 **'jsonrpc' => '2.0',** 
 **'id'      => null,** 
 **'error'   => [** 
 **'message' => 'invalid json: ' .
json_last_error_msg(),** 
 **'code'    => -32700** 
 **]** 
 **]);** 
 **}** 

    // ... 
} 

```

还要确保捕获可能从实际服务函数中抛出的任何异常。由于我们使用的是 PHP 7，记住常规的 PHP 错误现在也会被抛出，因此不仅要捕获异常，还要捕获错误。您可以通过在`catch`子句中使用`Throwable`接口来捕获异常和错误：

```php
if (is_callable($method)) { 
 **try {** 
        $result = call_user_func_array($method, $json->params ?? []); 
        return json_encode(/* ... */); 
 **} catch (\Throwable $t) {** 
 **return json_encode([** 
 **'jsonrpc' => '2.0',** 
 **'id'      => $json->id,** 
 **'error'   => [** 
 **'message' => $t->getMessage(),** 
 **'code'    => $t->getCode()** 
 **]** 
 **]);** 
 **}** 
} else { // ... 

```

您现在可以继续实现包含库存服务业务逻辑的实际服务。由于我们到目前为止花了相当多的时间处理低级协议，让我们回顾一下这个服务的要求：库存服务管理库存中的文章。在结账过程中，库存服务需要检查所需文章的数量是否有库存，并在可能的情况下，减少给定数量的库存数量。

我们将在`Packt\Chp7\Inventory\InventoryService`类中实现这个逻辑。请注意，我们将尝试保持示例简单，并简单地在内存中管理我们的文章库存。在生产环境中，您可能会使用数据库管理系统来存储文章数据：

```php
namespace Packt\Chp7\Inventory\InventoryService; 

class InventoryService 
{ 
    private $stock = [ 
        1000 => 123, 
        1001 => 4, 
        1002 => 12 
    ]; 

    public function checkArticle(int $articleNumber, int $amount = 1): bool 
    { 
        if (!array_key_exists($articleNumber, $this->stock)) { 
            return false; 
        } 
        return $this->stock[$articleNumber] >= $amount; 
    } 

    public function takeArticle(int $articleNumber, int $amount = 1): bool 
    { 
        if (!$this->checkArticle($articleNumber, $amount) { 
            return false; 
        } 

        $this->stock[$articleNumber] -= $amount; 
        return true; 
    } 
} 

```

在这个例子中，我们从文章编号`1000`到`1002`开始。`checkArticle`函数测试给定文章的所需数量是否有库存。`takeArticle`函数尝试减少所需数量的文章数量，如果可能的话。如果成功，函数返回`true`。如果所需数量不在库存中，或者根本不知道这篇文章，函数将返回`false`。

现在我们有一个实现 JSON-RPC 服务器的类，另一个类包含我们库存服务的实际业务逻辑。我们现在可以将这两个类放在我们的`server.php`文件中一起使用：

```php
$args = getopt('p:', ['port=']); 
$ctx = new ZMQContext(); 

$port = $args['p'] ?? $args['port'] ?? 5557; 
$addr = 'tcp://*:' . $port; 

$sock = $ctx->getSocket(ZMQ::SOCKET_REP); 
$sock->bind($addr); 

**$service = new \Packt\Chp7\Inventory\InventoryService();** 
**$server = new \Packt\Chp7\Inventory\JsonRpcServer($sock, $service);** 
**$server->run();**

```

为了测试这个服务，至少在您的结账服务的第一个版本运行起来之前，您可以调整在上一节中创建的`client.php`脚本，以便发送和接收 JSON-RPC 消息：

```php
// ... 

$msg = [ 
    'jsonrpc' => '2.0', 
    'method'  => 'takeArticle', 
    'params'  => [1001, 2] 
]; 

$sock->send(json_encode($msg)); 
$response = json_decode($sock->recv()); 

if (isset($response->error)) { 
    // handle error... 
} else { 
    $success = $reponse->result; 
    var_dump($success); 
} 

```

每次调用此脚本都会从库存中删除两件编号为＃1001 的物品。在我们的例子中，我们使用的是一个在本地管理的库存，始终初始化为此文章的四件物品，因此`client.php`脚本的前两次调用将返回 true 作为结果，而所有后续调用将返回 false。

# 使库存服务多线程化

目前，库存服务在单个线程中运行，并且使用阻塞套接字。这意味着它一次只能处理一个请求；如果在处理其他请求时收到新请求，客户端将不得不等待直到所有先前的请求都完成处理。显然，这不是很好的扩展。

为了实现一个可以并行处理多个请求的服务器，您可以使用 ZeroMQ 的**ROUTER**/**DEALER**模式。ROUTER 是一种特殊类型的 ZeroMQ 套接字，行为非常类似于常规的 REP 套接字，唯一的区别是可以并行连接多个 REQ 套接字。同样，DEALER 套接字是另一种类似于 REQ 套接字的套接字，唯一的区别是可以连接到多个 REP 套接字。这使您可以构建一个负载均衡器，它只包括一个 ROUTER 和一个 DEALER 套接字，将多个客户端的数据包传输到多个服务器。

![使库存服务多线程化](img/image_07_005.jpg)

ROUTER/DEALER 模式

由于 PHP 不支持多线程（至少不是很好），在这个例子中我们将采用多进程。我们的多线程服务器将由一个处理 ROUTER 和 DEALER 套接字的主进程以及多个每个使用一个 REP 套接字的 worker 进程组成。要实现这一点，您可以使用`pcntl_fork`函数分叉多个 worker 进程。

### 提示

为了使`pcntl_fork`函数工作，您需要启用`pcntl`扩展。在几乎所有的发行版中，这个扩展默认是启用的；在您之前构建的 Dockerfile 中，它也被明确安装了。如果您自己编译 PHP，那么在调用`configure`脚本时，您将需要`--enable-pcntl`标志。

在这个例子中，我们的库存服务将由多个 ZeroMQ 套接字组成：首先是大量的 worker 进程，每个进程都监听一个 RES 套接字以响应请求，以及一个主进程，每个 ROUTER 和 DEALER 套接字都接受和分发这些请求。只有 ROUTER 套接字对外部服务可见，并且可以通过 TCP 到达；对于所有其他套接字，我们将使用 UNIX 套接字进行通信 - 它们更快，且无法通过网络到达。

首先实现一个 worker 函数；为此创建一个名为`server_multithreaded.php`的新文件：

```php
require 'vendor/autoload.php'; 

use Packt\Chp7\Inventory\InventoryService; 
use Packt\Chp7\Inventory\JsonRpcServer; 

function worker() 
{ 
    $ctx = new ZMQContext(); 

    $sock = $ctx->getSocket(ZMQ::SOCKET_REP); 
    $sock->connect('ipc://workers.ipc'); 

    $service = new InventoryService(); 

    $server = new JsonRpcServer($sock, $service); 
    $server->run(); 
} 

```

`worker()`函数创建一个新的 REP 套接字，并将此套接字连接到 UNIX 套接字`ipc://workers.ipc`（这将由主进程稍后创建）。然后运行您之前已经使用过的`JsonRpcServer`。

现在，您可以使用`pcntl_fork`函数启动任意数量（在本例中为四个）的这些 worker 进程：

```php
for ($i = 0; $i < 4; $i ++) { 
    $pid = pcntl_fork(); 
    if ($pid == 0) { 
        worker($i); 
        exit(); 
    } 
} 

```

如果您不熟悉`fork`函数：它会复制当前运行的进程。分叉的进程将继续在分叉时的相同代码位置运行。然而，在父进程中，`pcntl_fork()`的返回值将返回新创建进程的进程 ID。然而，在新进程中，这个值将是 0。在这种情况下，子进程现在成为我们的 worker 进程，而实际的主进程将在不退出的情况下通过循环。

在此之后，您可以通过创建一个 ROUTER 和一个 DEALER 套接字来启动实际的负载均衡器：

```php
$args = getopt('p:', ['port=']); 
$ctx = new ZMQContext(); 

$port = $args['p'] ?? $args['port'] ?? 5557; 
$addr = 'tcp://*:' . $port; 

$ctx = new ZMQContext(); 

//  Socket to talk to clients 
$clients = $ctx->getSocket(ZMQ::SOCKET_ROUTER); 
$clients->bind($addr); 

//  Socket to talk to workers 
$workers = $ctx->getSocket(ZMQ::SOCKET_DEALER); 
$workers->bind("ipc://workers.ipc"); 

```

ROUTER 套接字绑定到服务预期可到达的实际网络地址（在本例中，允许通过网络到达服务的 TCP 套接字）。另一方面，DEALER 套接字绑定到一个本地 UNIX 套接字，不会暴露给外部世界。UNIX 套接字`ipc://workers.ipc`的唯一目的是工作进程可以将其 REP 套接字连接到它。

创建了 ROUTER 和 DEALER 套接字后，您可以使用`ZMQDevice`类将来自 ROUTER 套接字的传入数据包传输到 DEALER 套接字，然后平均分配到所有连接的 REP 套接字。从 REP 套接字发送回来的响应数据包也将被分发回原始客户端：

```php
//  Connect work threads to client threads via a queue 
$device = new ZMQDevice($clients, $workers); 
$device->run(); 

```

以这种方式更改库存服务不需要修改客户端代码；负载均衡器正在监听的 ROUTER 套接字行为非常类似于 REP 套接字，并且任何 REQ 套接字都可以以完全相同的方式连接到它。

# 构建结账服务

现在我们有一个管理您小型虚构电子商务企业库存的服务。接下来，我们将实现实际结账服务的第一个版本。结账服务将提供一个用于完成结账流程的 API，使用由多个文章和基本客户联系数据组成的购物车。

## 使用 react/zmq

为此，结账服务将提供一个简单的 REP ZeroMQ 套接字（或在并发设置中的 ROUTER 套接字）。在接收结账订单后，结账服务将与库存服务通信，以检查所需物品是否可用，并通过购物车中的物品数量减少库存数量。如果成功，它将在 PUB 套接字上发布结账订单，其他服务可以监听。

如果购物车包含多个物品，结账服务将需要多次调用库存服务。在本例中，您将学习如何并行进行多个请求以加快执行速度。我们还将使用`react/zmq`库，该库为 ZeroMQ 库提供了异步接口，以及`react/promise`库，它将帮助您更好地处理异步应用程序。

首先在新的`checkout/`目录中创建一个新的`composer.json`文件，并使用`composer install`初始化项目：

```php
{ 
 **"name": "packt-php7/chp7-checkout",** 
  "type": "project", 
  "authors": [{ 
    "name": "Martin Helmich", 
    "email": "php7-book@martin-helmich.de" 
  }], 
  "require": { 
    "php": ">= 7.0", 
 **"react/zmq": "⁰.3.0",** 
 **"react/promise": "².2",** 
    "ext-zmq": "*", 
 **"ext-ev": "*"** 
  }, 
  "autoload": { 
    "psr-4": { 
 **"Packt\\Chp7\\Checkout": "src/"** 
    } 
  } 

```

这个文件类似于库存服务的`composer.json`；唯一的区别是 PSR-4 命名空间和额外的要求`react/zmq`，`react/promise`和`ext-ev`。如果您正在使用 Docker 进行开发设置，您可以直接从库存服务中复制现有的 Dockerfile。

继续在您的`checkout/`目录中创建一个`server.json`文件。与任何 React 应用程序一样（记得第六章中的 Ratchet 应用程序，*构建聊天应用程序*），您需要做的第一件事是创建一个事件循环，然后运行它：

```php
<?php 
use \React\ZMQ\Factory; 
use \React\ZMQ\Context; 

require 'vendor/autoload.php'; 

$loop = Factory::create(); 
$ctx  = new Context($loop); 

$loop->run(); 

```

请注意，我们现在使用`React\ZMQ\Context`类而不是`ZMQContext`类。React 上下文类提供相同的接口，但通过一些功能扩展了其基类，以更好地支持异步编程。

您现在可以启动此程序，它将无限运行，但目前还不会执行任何操作。由于结账服务应该提供一个 REP 套接字，客户端应该发送请求到该套接字，因此在运行事件循环之前，您应该继续创建并绑定一个新的 REP 套接字：

```php
// ... 
$ctx = new Context($loop); 

**$socket = $ctx->getSocket(ZMQ::SOCKET_REP);** 
**$socket->bind('tcp://0.0.0.0:5557');** 

$loop->run(); 

```

**ReactPHP**应用程序是异步的；现在，您可以在套接字上注册事件处理程序，而不是只调用`recv()`等待下一个传入消息，ReactPHP 的事件循环将在收到消息时立即调用它：

```php
// ... 

$socket = $ctx->getSocket(ZMQ::SOCKET_REP); 
$socket->bind('tcp://0.0.0.0:5557'); 
**$socket->on('message', function(string $msg) use ($socket) {** 
 **echo "received message $msg.\n";** 
 **$socket->send('Response text');** 
**});** 

$loop->run(); 

```

这种回调解决方案类似于您在开发客户端 JavaScript 代码时最常遇到的其他异步库。基本原则是相同的：`$socket->on(...)`方法只是注册一个事件监听器，可以在以后的任何时间点调用，每当收到新消息时。代码的执行将立即继续（与此相反，比较常规的`$socket->recv()`函数会阻塞，直到收到新消息），然后调用`$loop->run()`方法。这个调用启动了实际的事件循环，负责在收到新消息时调用注册的事件监听器。事件循环将一直阻塞，直到被中断（例如，通过命令行上的*Ctrl* + *C*触发的 SIGINT 信号）。

## 使用承诺

在处理异步代码时，通常只是时间的问题，直到您发现自己陷入了“回调地狱”。想象一下，您想发送两个连续的 ZeroMQ 请求（例如，首先询问库存服务给定的文章是否可用，然后实际上指示库存服务减少所需数量的库存）。您可以使用多个套接字和您之前看到的“消息”事件来实现这一点。然而，这很快就会变成一个难以维护的混乱：

```php
$socket->on('message', function(string $msg) use ($socket, $ctx) { 
    $check = $ctx->getSocket(ZMQ::SOCKET_REQ); 
    $check->connect('tcp://identity:5557'); 
    $check->send(/* checkArticle JSON-RPC here */); 
    $check->on('message', function(string $msg) use ($socket, $ctx) { 
        $take = $ctx->getSocket(ZMQ::SOCKET_REQ); 
        $take->connect('tcp://identity:5557'); 
        $take->send(/* takeArticle JSON-RPC here */); 
        $take->on('message', function(string $msg) use ($socket) { 
            $socket->send('success'); 
        }); 
    }); 
}); 

```

上述代码片段只是说明了这可能变得多么复杂的一个例子；在我们的情况下，您甚至需要考虑每个结账订单可能包含任意数量的文章，每篇文章都需要两个新请求到身份服务。

为了让生活更美好，您可以使用承诺来实现这个功能（有关该概念的详细解释，请参见下面的框）。`react/promise`库提供了良好的承诺实现，应该已经在您的`composer.json`文件中声明。

### 注意

**什么是承诺？** 承诺（有时也称为未来）是异步库中常见的概念。它们提供了一种替代常规基于回调的方法。

基本上，承诺是一个代表尚未可用的值的对象（例如，因为应该检索该值的 ZeroMQ 请求尚未收到回复）。在异步应用程序中，承诺可能随时变得可用（实现）。然后，您可以注册应该在承诺实现时调用的函数，以进一步处理承诺的已解析值：`$promise = $someService->someFunction();` `$promise->then(function($promisedValue) {` `    echo "Promise resolved: $promisedValue\n";` `});`

`then()`函数的每次调用都会返回一个新的承诺，这次是由传递给`then()`的回调返回的值。这使您可以轻松地将多个承诺链接在一起：

`$promise` `    ->then(function($value) use ($someService) {` `        $newPromise = $someService->someOtherFunc($value);` `        return $newPromise;` `    })` `    ->then(function ($newValue) {` `        echo "Promise resolved: $newValue\n";` `    });`

现在，我们可以通过编写一个用于与我们的库存服务通信的异步客户端类来利用这个原则。由于该服务使用 JSON-RPC 进行通信，我们现在将实现`Packt\Chp7\Checkout\JsonRpcClient`类。该类使用 ZeroMQ 上下文进行初始化，并且为了方便起见，还包括远程服务的 URL：

```php
namespace Packt\Chp7\Checkout; 

use React\Promise\PromiseInterface; 
use React\ZMQ\Context; 

class JsonRpcClient 
{ 
    private $context; 
    private $url; 

    public function __construct(Context $context, string $url) 
    { 
        $this->context = $context; 
        $this->url     = $url; 
    } 

    public function request(string $method, array $params = []): PromiseInterface 
    { 
    } 
} 

```

在这个例子中，该类已经包含一个`request`方法，该方法接受一个方法名和一组参数，并应返回`React\Promise\PromiseInterface`的实现。

在`request()`方法中，您现在可以打开一个新的 REQ 套接字并向其发送一个 JSON-RPC 请求：

```php
public function request(string $method, array $params = []): PromiseInterface 
{ 
 **$body = json_encode([** 
 **'jsonrpc' => '2.0',** 
 **'method'  => $method,** 
 **'params'  => $params,** 
 **]);** 
 **$sock = $this->context->getSocket(\ZMQ::SOCKET_REQ);** 
 **$sock->connect($this->url);** 
 **$sock->send($body);** 
} 

```

由于`request()`方法应该是异步工作的，您不能简单地调用`recv()`方法并阻塞，直到收到结果。相反，我们需要返回一个对响应值的承诺，以便稍后可以解决，每当在 REQ 套接字上收到响应消息时。为此，您可以使用`React\Promise\Deferred`类：

```php
$body = json_encode([ 
    'jsonrpc' => '2.0', 
    'method'  => $method, 
    'params'  => $params, 
]); 
**$deferred = new Deferred();** 

$sock = $this->context->getSocket(\ZMQ::SOCKET_REQ); 
$sock->connect($this->url); 
**$sock->on('message', function(string $response) use ($deferred) {** 
 **$deferred->resolve($response);** 
**});** 
$sock->send($body); 

**return $deferred->promise();**

```

这是承诺如何工作的一个典型例子：您可以使用`Deferred`类来创建并返回一个尚未可用的值的承诺。记住：传递给`$sock->on(...)`方法的函数不会立即被调用，而是在任何以后的时间点，当实际收到响应时。一旦发生这种事件，由请求函数返回的承诺将以实际的响应值解决。

由于响应消息包含 JSON-RPC 响应，您需要在满足对请求函数的调用者所做的承诺之前评估这个响应。由于 JSON-RPC 响应也可能包含错误，值得注意的是，您也可以拒绝一个承诺（例如，在等待响应时发生错误时）：

```php
$sock->on('message', function(string $response) use ($deferred) { 
 **$response = json_decode($response);** 
 **if (isset($response->result)) {** 
 **$deferred->resolve($response->result);** 
 **} elseif (isset($response->error)) {** 
 **$deferred->reject(new \Exception(** 
 **$response->error->message,** 
 **$response->error->code** 
 **);** 
 **} else {** 
 **$deferred->reject(new \Exception('invalid response'));** 
 **}** 
}); 

```

现在，您可以在您的`server.php`中使用这个 JSON-RPC 客户端类，以便在每个传入的结账请求上实际与库存服务进行通信。让我们从一个简单的例子开始，演示如何使用新类将两个连续的 JSON-RPC 调用链接在一起：

```php
$client = new JsonRpcClient($ctx, 'tcp://inventory:5557'); 
$client->request('checkArticle', [1000]) 
    ->then(function(bool $ok) use ($client) { 
        if ($ok) { 
            return $client->request('takeArticle', [1000]); 
        } else { 
            throw new \Exception("Article is not available"); 
        } 
    }) 
    ->then(function(bool $ok) { 
        if ($ok) { 
            echo "Successfully took 1 item of article 1000"; 
        } 
    }, function(\Exception $error) { 
        echo "An error occurred: ${error->getMessage()}\n"; 
    }); 

```

正如您所看到的，`PromiseInterface`的`then`函数接受两个参数（每个都是一个新函数）：第一个函数将在承诺以实际值解决时被调用；第二个函数将在承诺被拒绝时被调用。

如果传递给`then(...)`的函数返回一个新值，那么 then 函数将返回一个新的承诺。这个规则的一个例外是当回调函数本身返回一个新的承诺（在我们的情况下，在`then()`回调中再次调用了`$client->request`）。在这种情况下，返回的承诺将替换原始承诺。这意味着对`then()`函数的链接调用实际上是在第二个承诺上监听。

让我们在`server.php`文件中使用这个。与前面的例子相比，您需要考虑每个结账订单可能包含多个文章。这意味着您需要对库存服务执行多个`checkArticle`请求：

```php
**$client = new JsonRpcClient($ctx, 'tcp://inventory:5557');** 
$socket->on('message', function(string $msg) use ($socket, $client) { 
 **$request = json_decode($msg);** 
 **$promises = [];** 
 **foreach ($request->cart as $article) {** 
 **$promises[] = $client->request('checkArticle', [$article->articlenumber, $article->amount]);** 
    } 
}); 

```

在这个例子中，我们假设传入的结账订单是 JSON 编码的消息，看起来像下面的例子：

```php
{ 
  "cart": [ 
    "articlenumber": 1000, 
    "amount": 2 
  ] 
} 

```

在我们的`server.php`的当前版本中，我们多次调用 JSON-RPC 客户端，并将返回的承诺收集到一个数组中。然而，我们实际上还没有对它们做任何事情。现在，您可以对这些承诺中的每一个调用`then()`函数，其中包含一个回调，该回调将对每个文章进行调用，并传递一个布尔参数，指示这篇文章是否可用。然而，为了正确处理订单，我们需要知道结账订单中的所有文章是否都可用。所以你需要做的不是等待每个承诺单独完成，而是等待它们全部完成。这就是`React\Promise\all`函数的作用：这个函数以承诺列表作为参数，并返回一个新的承诺，一旦所有提供的承诺都被实现，它就会被实现：

```php
$request = json_decode($msg); 
$promises = []; 

foreach ($request->cart as $article) { 
    $promises[] = $client->request('checkArticle', [$article->articlenumber, $article->amount]); 
} 

**React\Promise\all($promises)->then(function(array $values) use ($socket) {** 
 **if (array_sum($values) == count($values)) {** 
 **echo "all required articles are available";** 
 **} else {** 
 **$socket->send(json_encode([** 
 **'error' => 'not all required articles are available'** 
 **]);** 
 **}**
**});**

```

如果库存服务中没有所有所需的文章，您可以提前用错误消息回答请求，因为没有必要继续下去。如果所有文章都可用，您将需要一系列后续请求来实际减少指定数量的库存。

### 提示

在这个例子中使用的`array_sum($values) == count($values)`构造是一个快速的解决方法，用来确保布尔值数组只包含 true 值。

接下来，您现在可以扩展您的服务器，以在所有`checkArticle`方法调用成功返回后运行第二组请求到库存服务。这可以通过使用`React\Promise\all`方法按照之前的方式完成：

```php
React\Promise\all($promises)->then(function(array $values) use ($socket, $request) { 
 **$promises = [];** 
 **if (array_sum($values) == count($values)) {** 
 **foreach ($request->cart as $article) {** 
 **$promises[] = $client->request('takeArticle', [$article->articlenumber, $article->amount]);** 
 **}** 
 **React\Promise\all($promises)->then(function() use ($socket) {** 
 **$socket->send(json_encode([** 
 **'result' => true** 
 **]);** 
 **}** 
    } else { 
        $socket->send(json_encode([ 
            'error' => 'not all required articles are available' 
        ]); 
    } 
}); 

```

为了实际测试这个新的服务器，让我们编写一个简短的测试脚本，尝试执行一个示例结账订单。为此，在您的`checkout/`目录中创建一个新的`client.php`文件：

```php
$ctx  = new ZMQContext(); 
$sock = $ctx->getSocket(ZMQ::SOCKET_REQ); 
$sock->connect('tcp://checkout:5557'); 
$sock->send(json_encode([ 
    'cart' => [ 
        ['articlenumber' => 1000, 'amount' => 3], 
        ['articlenumber' => 1001, 'amount' => 2] 
    ] 
])); 

$result = $sock->recv(); 
var_dump($result); 

```

要运行结账服务和测试脚本，可以在项目的根目录中使用新的结账服务扩展您的`docker-compose.yml`文件：

```php
**checkout:** 
 **build: checkout** 
 **volumes:** 
 **- checkout:/usr/src/app** 
 **links:** 
 **- inventory:inventory** 
inventory: 
  build: inventory 
  ports: 
    - 5557 
  volumes: 
    - inventory:/usr/src/app 

```

对于测试脚本，添加第二个 Compose 配置文件`docker-compose.testing.yml`：

```php
test: 
  build: checkout 
  command: php client.php 
  volumes: 
    - checkout:/usr/src/app 
  links: 
    - checkout:checkout 

```

之后，您可以使用以下命令行命令测试您的结账服务：

```php
**$ docker-compose up -d 
$ docker-compose -f docker-compose.testing.yml run --rm test**

```

以下屏幕截图显示了测试脚本和两个服务器脚本的示例输出（在此示例中，添加了一些额外的`echo`语句，使服务器更加详细）：

![使用承诺工作](img/image_07_006.jpg)

结账和库存服务处理结账订单的示例输出

# 构建邮寄服务

接下来，我们将在我们的微服务架构中加入一个邮寄服务。在处理结账后，用户应该通过电子邮件收到有关结账状态的通知。

### 提示

如前所述，本章的重点是构建个别服务之间的通信模式。因此，在本节中，我们不会实现邮寄服务的实际邮寄功能，而是专注于该服务如何与其他服务通信。查看第三章*构建社交通讯服务*，了解如何使用 PHP 实际向其他收件人发送电子邮件。

理论上，您可以像实现库存服务一样实现邮寄服务-构建一个独立的 PHP 程序，监听 ZeroMQ REP 套接字，让结账服务打开一个 REQ 套接字，并向邮寄服务发送请求。但是，也可以使用发布/订阅模式来实现相同的功能。

使用发布/订阅模式，结账服务甚至不需要知道邮寄服务。相反，结账服务只需打开其他服务可以连接到的 PUB 套接字。在 PUB 套接字上发送的任何消息都会分发到所有连接的（订阅）服务。这允许您实现一个非常松散耦合的架构，也非常可扩展-您可以通过让更多和不同的服务订阅相同的 PUB 套接字来为您的结账流程添加新功能，而无需修改结账服务本身。

这是可能的，因为在邮寄服务的情况下，通信不需要是同步的-结账服务在继续流程之前不需要等待邮寄服务完成其操作，也不需要来自邮寄服务的任何数据。相反，消息可以严格单向流动-从结账服务到邮寄服务。

首先，您需要在结账服务中打开 PUB 套接字。为此，请修改结账服务的`server.php`，创建一个新的 PUB 套接字，并将其绑定到 TCP 地址：

```php
$socket = $ctx->getSocket(ZMQ::SOCKET_REP); 
$socket->bind('tcp://0.0.0.0:5557'); 

**$pubSocket = $ctx->getSocket(ZMQ::SOCKET_PUB);**
**$pubSocket->bind('tcp://0.0.0.0:5558');** 

$client = new JsonRpcClient($ctx, 'tcp://inventory:5557'); 

```

成功从库存服务中取得所需物品后，您可以在此套接字上发布消息。在这种情况下，我们将简单地在 PUB 套接字上重新发送原始消息：

```php
$socket->on('message', function(string $msg) use ($client, $pubSocket) { 
    // ... 
    React\Promise\all($promises)->then(function(array $values) use ($socket, $pubSocket, $request) { 
        $promises = []; 
        if (array_sum($values) == count($values)) { 
            // ... 
            React\Promise\all($promises)->then(function() use ($socket, $pubSocket, $request) { 
 **$pubSocket->send($request);** 
            $socket->send(json_encode([ 
                'result' => true 
            ]); 
        } else { 
            $socket->send(json_encode([ 
                'error' => 'not all required articles are available' 
            ]); 
        } 
    }); 
}); 

$loop->run(); 

```

现在，您已经在接受的结账订单上发布了一个 PUB 套接字，可以编写实际的邮寄服务，创建一个订阅此 PUB 套接字的 SUB 套接字。

为此，在项目目录中创建一个名为`mailing/`的新目录。从先前的示例中复制 Dockerfile，并创建一个新的`composer.json`文件，内容如下：

```php
{ 
 **"name": "packt-php7/chp7-mailing",** 
    "type": "project", 
    "authors": [{ 
        "name": "Martin Helmich", 
        "email": "php7-book@martin-helmich.de" 
    }], 
    "require": { 
        "php": ">= 7.0", 
        "react/zmq": "⁰.3.0" 
    }, 
    "autoload": { 
        "psr-4": { 
 **"Packt\\Chp7\\Mailing": "src/"** 
        } 
    } 
} 

```

与以前的示例相比，唯一的区别是新的包名称和不同的 PSR-4 自动加载命名空间。此外，您不需要`react/promise`库来进行邮寄服务。像往常一样，在`mailing/`目录中的命令行上运行`composer install`来下载所需的依赖项。

您现在可以在`mailing/`目录中创建一个新的`server.php`文件，其中创建一个新的 SUB 套接字，然后可以连接到结帐服务：

```php
require 'vendor/autoload.php'; 

$loop = \React\EventLoop\Factory::create(); 
$ctx  = new \React\ZMQ\Context($loop); 

$socket = $ctx->getSocket(ZMQ::SOCKET_SUB); 
$socket->subscribe(''); 
$socket->connect('tcp://checkout:5558'); 

$loop->run(); 

```

注意`$socket->subscribe()`调用。每个 SUB 套接字可以订阅给定的*主题*或*频道*。频道由一个字符串前缀标识，可以作为每个发布的消息的一部分提交。然后客户端只会接收与他们订阅的频道匹配的消息。如果您不关心一个 PUB 套接字上的不同频道，您可以通过调用`$socket->subscribe`并传递一个空字符串来订阅空频道，从而接收在 PUB 套接字上发布的所有消息。但是，如果您不调用 subscribe 方法，您将根本不会收到任何消息。

套接字连接后，您可以为`'message'`事件提供一个监听函数，在其中解码 JSON 编码的消息并相应地处理它：

```php
$socket->connect('tcp://checkout:5558'); 
**$socket->on('message', function(string $msg) {** 
 **$data = json_decode($msg);** 
 **if (isset($data->customer->email)) {** 
 **$email = $data->customer->email;** 
 **echo "sending confirmation email to $email.\n";** 
 **}** 
**});** 

$loop->run(); 

```

还要注意，PUB 和 SUB 套接字是严格单向的：您可以从 PUB 套接字向任意数量的订阅的 SUB 套接字发送消息，但您不能在同一个套接字上回复给发布者-至少不能。如果您真的需要某种反馈渠道，您可以让发布者在一个单独的 REP 或 SUB 套接字上监听，订阅者使用新的 REQ 或 PUB 套接字连接。以下图表说明了实现这样的反馈渠道的两种策略：

![构建邮寄服务](img/image_07_007.jpg)

在发布/订阅架构中实现反馈通道的不同策略

要测试新的邮寄服务，您可以重用上一节中的`client.php`脚本。由于邮寄服务要求结帐订单包含电子邮件地址，您需要将其添加到消息正文中：

```php
$sock->send(json_encode([ 
    'cart' => [ 
        ['articlenumber' => 1000, 'amount' => 3], 
        ['articlenumber' => 1001, 'amount' => 2] 
    ], 
 **'customer' => [** 
 **'email' => 'john.doe@example.com'** 
    ] 
])); 

```

还要记得将新的邮寄服务添加到`docker-compose.yml`文件中：

```php
# ... 
checkout: 
  build: checkout 
  volumes: 
    - checkout:/usr/src/app 
  links: 
    - inventory:inventory 
**mailing:** 
 **build: mailing** 
 **volumes:** 
 **- mailing:/usr/src/app** 
 **links:** 
 **- checkout:checkout** 
inventory: 
  build: inventory 
  ports: 
    - 5557 
  volumes: 
    - inventory:/usr/src/app 

```

在`docker-compose.yml`中添加新服务后，启动所有服务并再次运行测试脚本：

```php
**$ docker-compose up -d inventory checkout mailing**
**$ docker-compose run --rm test**

```

之后，检查单独的容器的输出，以检查结帐订单是否被正确处理：

```php
**$ docker-compose logs**

```

# 构建邮寄服务

在我们的小型电子商务示例中，我们还缺少邮寄服务。在现实世界的场景中，这将是一个非常复杂的任务，您通常需要与外部方进行通信，也许需要与外部运输服务提供商的 API 集成。因此，我们现在将使用 PUSH 和 PULL 套接字以及任意数量的工作进程构建我们的邮寄服务作为工作池。

## 初学者的 PUSH/PULL

PUB 套接字将每条消息发布到所有连接的订阅者。ZeroMQ 还提供了 PUSH 和 PULL 套接字类型-它们的工作方式类似于 PUB/SUB，但在 PUSH 套接字上发布的每条消息只发送到潜在的多个连接的 PULL 套接字中的一个。您可以使用这个来实现一个工作池，将长时间运行的任务推送到其中，然后并行执行。

为此，我们需要一个使用 SUB 套接字订阅已完成结帐订单的主进程。同一进程需要提供一个 PUSH 套接字，以便各个工作进程可以连接到它。以下图表说明了这种架构：

![初学者的 PUSH/PULL](img/image_07_008.jpg)

PUB/SUB 和 PUSH/PULL 的组合

像往常一样，首先在项目文件夹中创建一个新的`shipping/`目录。从以前的服务中复制 Dockerfile，创建一个新的`composer.json`文件，并使用`composer install`初始化项目：

```php
{ 
 **"name": "packt-php7/chp7-shipping",** 
    "type": "project", 
    "authors": [{ 
        "name": "Martin Helmich", 
        "email": "php7-book@martin-helmich.de" 
    }], 
    "require": { 
        "php": ">= 7.0.0", 
        "react/zmq": "⁰.3.0" 
    }, 
    "autoload": { 
        "psr-4": { 
 **"Packt\\Chp7\\Shipping": "src/"** 
        } 
    } 
} 

```

我们将从实现主进程开始。这个主进程需要做三件简单的事情：

+   打开一个 SUB 套接字，并将此套接字连接到结帐服务的 PUB 套接字。这将允许运输服务接收结帐服务接受的所有结帐订单。

+   打开一个 PUSH 套接字，并将此套接字绑定到一个新的 TCP 端口。这将允许工作进程连接并接收结帐订单。

+   将在 SUB 套接字上接收的每条消息转发到 PUSH 套接字。

为此，在您的`shipping/`目录中创建一个新的`master.php`文件，您可以在其中创建一个新的事件循环并创建两个所需的套接字：

```php
require 'vendor/autoload.php'; 

$loop = React\EventLoop\Factory::create(); 
$ctx  = new React\ZMQ\Context($loop); 

$subSocket = $ctx->getSocket(ZMQ::SOCKET_SUB); 
$subSocket->subscribe(''); 
$subSocket->connect('tcp://checkout:5558'); 

$pushSocket = $ctx->getSocket(ZMQ::SOCKET_PUSH); 
$pushSocket->bind('tcp://0.0.0.0:5557'); 

$loop->run(); 

```

为了实际处理在 SUB 套接字上接收的消息，注册一个监听器函数在`$subSocket`变量上，将每个接收到的消息发送到 PUSH 套接字：

```php
$pushSocket->bind('tcp://0.0.0.0:5557'); 

**$subSocket->on('message', function(string $msg) use ($pushSocket) {** 
 **echo 'dispatching message to worker';** 
 **$pushSocket->send($msg);** 
**});** 

$loop->run(); 

```

接下来，在`shipping/`目录中创建一个名为`worker.php`的新文件。在这个文件中，您将创建一个 PULL 套接字，用于接收主进程中打开的 PUSH 套接字上的消息：

```php
require 'vendor/autoload.php'; 

$loop = React\EventLoop\Factory::create(); 
$ctx  = new React\ZMQ\Context($loop); 

$pullSocket = $ctx->getSocket(ZMQ::SOCKET_PULL); 
$pullSocket->connect('tcp://shippingmaster:5557'); 

$loop->run(); 

```

再次，附加一个监听器函数到`$pullSocket`，以处理传入的消息：

```php
$pullSocket->connect('tcp://shippingmaster:5557'); 
**$pullSocket->on('message', function(string $msg) {** 
 **echo "processing checkout order for shipping: $msg\n";** 
 **sleep(5);** 
**});** 

$loop->run(); 

```

`sleep(5)`，在这个例子中，只是模拟执行可能需要更长时间的运输订单。与本章中一样，我们不会实现实际的业务逻辑，只需要演示各个服务之间的通信模式。

为了测试运输服务，现在将主进程和工作进程添加到您的`docker-compose.yml`文件中：

```php
# ... 

inventory: 
  build: inventory 
  volumes: 
    - inventory:/usr/src/app 

**shippingmaster:** 
 **build: shipping** 
 **command: php master.php** 
 **volumes:** 
 **- shipping:/usr/src/app** 
 **links:** 
 **- checkout:checkout** 
**shippingworker:** 
 **build: shipping** 
 **command: php worker.php** 
 **volumes:** 
 **- shipping:/usr/src/app** 
 **links:** 
 **- shippingmaster:shippingmaster**

```

之后，您可以启动所有容器，然后使用以下命令跟踪它们的输出：

```php
**$ docker-compose up -d**

```

默认情况下，Docker compose 将始终启动每个服务的一个实例。但是，您可以使用`docker-compose scale`命令启动每个服务的附加实例。这对于`shippingworker`服务来说是一个好主意，因为我们为该服务选择的 PUSH/PULL 架构实际上允许任意数量的该服务实例并行运行：

```php
**$ docker-compose scale shippingworker=4**

```

在启动了一些更多的`shippingworker`服务实例之后，您可以使用`docker-compose logs`命令附加到所有容器的日志输出。然后，使用第二个终端启动您在上一节中创建的客户端测试脚本：

```php
**$ docker-compose run --rm test**

```

当您多次运行此命令时，您将看到在后续调用的容器的不同实例中打印的运输工作进程中的调试输出。您可以在以下截图中看到一个示例输出：

![初学者的 PUSH/PULL](img/image_07_009.jpg)

演示具有多个工作进程的工作推/拉架构的示例输出

## 扇出/扇入

除了将耗时的任务分配给多个工作进程外，您还可以使用 PUSH 和 PULL 套接字让工作进程将结果推送回主进程。这种模式称为**扇出/扇入**。对于本例，让`master.php`文件中的主进程监听一个单独的 PULL 套接字：

```php
$pushSocket = $ctx->getSocket(ZMQ::SOCKET_PUSH); 
$pushSocket->bind('tcp://0.0.0.0:5557'); 

**$pullSocket = $ctx->getSocket(ZMQ::SOCKET_PULL);** 
**$pullSocket->bind('tcp://0.0.0.0:5558');** 
**$pullSocket->on('message', function(string $msg) {** 
 **echo "order $msg successfully processed for shipping\n";** 
**});** 

$subSocket->on('message', function(string $msg) use ($pushSocket) { 
    // ... 
}); 

$loop->run(); 

```

在`worker.php`文件中，您现在可以使用新的 PUSH 套接字连接到此 PULL 套接字，并在成功处理结帐订单时发送消息：

```php
**$pushSocket = $ctx->getSocket(ZMQ::SOCKET_PUSH);**
**$pushSocket->connect('tcp://shippingmaster:5558');** 

$pullSocket = $ctx->getSocket(ZMQ::SOCKET_PULL); 
$pullSocket->connect('tcp://shippingmaster:5557'); 
$pullSocket->on('message', function(string $msg) use ($pushSocket) { 
    echo "processing checkout order for shipping: $msg\n"; 
    sleep(5); 
 **$pushSocket->send($msg);** 
}); 

$loop->run(); 

```

一旦处理完消息，这将立即将消息推送回主进程。请注意，PUSH/PULL 的使用方式与上一节中的方式相反-之前我们有一个 PUSH 套接字和多个 PULL 套接字；对于扇入，我们在主进程上有一个 PULL 套接字，而在工作进程上有多个 PUSH 套接字。

### 提示

**使用 bind()和 connect()**

在本节中，我们已经使用了 PUSH 和 PULL 套接字的`bind()`和`connect()`方法。通常，`bind()`用于让套接字监听新的 TCP 端口（或 UNIX 套接字），而`connect()`用于让套接字连接到另一个已经存在的套接字。通常情况下，您可以在任何套接字类型上使用`bind()`和`connect()`。在某些情况下，比如 REQ/REP，您会直觉地`bind()`REP 套接字，然后`connect()`REQ 套接字，但 PUSH/PULL 和 PUB/SUB 实际上都可以双向工作。您可以让 PULL 套接字连接到监听的 PUSH 套接字，但也可以让 PUSH 套接字连接到监听的 PULL 套接字。

以下截图显示了运输服务的主进程和工作进程并行处理多个结账订单的示例输出。请注意，实际处理是由不同的工作进程（在此示例中为`shippingworker_1`到`shippingworker_3`）完成的，但之后被"扇入"回主进程：

![扇出/扇入](img/image_07_010.jpg)

扇出/扇入的实际操作

# 连接 ZeroMQ 和 HTTP

正如您在本章中所看到的，ZeroMQ 提供了许多不同的可能性，用于在不同的服务之间实现通信。特别是，发布/订阅和推/拉等模式在 PHP 的事实标准协议 HTTP 中并不容易实现。

另一方面，HTTP 更广泛地被采用，并提供了更丰富的协议语义集，处理诸如缓存或身份验证等问题已经在协议级别上。因此，特别是在提供外部 API 时，您可能更喜欢提供基于 HTTP 而不是基于 ZeroMQ 的 API。幸运的是，在两种协议之间进行桥接很容易。在我们的示例架构中，结账服务是唯一会被外部服务使用的服务。为了为结账服务提供更好的接口，我们现在将实现一个基于 HTTP 的结账服务包装器，可以以 RESTful 方式使用。

为此，您可以使用`react/http`包。该包提供了一个极简的 HTTP 服务器，就像`react/zmq`一样，它是异步工作的，并使用事件循环来处理请求。这意味着基于 react 的 HTTP 服务器甚至可以在同一个进程中，使用与结账服务已经提供的 REP ZeroMQ 套接字相同的事件循环来运行。首先，在项目目录中的`checkout/`文件夹中运行以下命令来安装`react/http`包：

```php
**$ composer require react/http**

```

在扩展结账服务以使用 HTTP 服务器之前，`server.php`脚本需要进行一些重构。当前，`server.php`创建了一个带有事件监听函数的 REP ZeroMQ 套接字，其中处理请求。由于我们的目标现在是添加一个触发相同功能的 HTTP API，我们需要将此逻辑提取到一个单独的类中。首先创建`Packt\Chp7\Checkout\CheckoutService`类：

```php
namespace Packt\Chp7\Checkout; 

use React\Promise\PromiseInterface; 

class CheckoutService 
{ 
    private $client; 

    public function __construct(JsonRpcClient $client) 
    { 
        $this->client = $client; 
    } 

    public function handleCheckoutOrder(string $msg): PromiseInterface 
    { 
    } 
} 

```

`handleCheckoutOrder`方法将保存之前直接在`server.php`文件中实现的逻辑。由于此方法稍后将被 ZeroMQ REP 套接字和 HTTP 服务器同时使用，因此此方法不能直接发送响应消息，而只能返回一个 promise，然后可以在`server.php`中使用：

```php
public function handleCheckoutOrder(string $msg): PromiseInterface 
{ 
    $request = json_decode($msg); 
    $promises = []; 

    foreach ($request->cart as $article) { 
        $promises[] = $this->client->request('checkArticle', [$article->articlenumber, $article->amount]); 
    } 

    return \React\Promise\all($promises) 
        ->then(function(array $values):bool { 
            if (array_sum($values) != count($values)) { 
                throw new \Exception('not all articles are in stock'); 
            } 
            return true; 
        })->then(function() use ($request):PromiseInterface { 
            $promises = []; 

            foreach ($request->cart as $article) { 
                $promises[] = $this->client->request('takeArticle', [$article->articlenumber, $article->amount]); 
            } 

            return \React\Promise\all($promises); 
        })->then(function(array $values):bool { 
            if (array_sum($values) != count($values)) { 
                throw new \Exception('not all articles are in stock'); 
            } 
            return true; 
        }); 
} 

```

一致使用 promise 并不关心返回消息实际上允许一些简化；而不是直接发送错误消息，您可以简单地抛出异常，这将导致此函数返回的*promise*被自动拒绝。

现有的`server.php`文件现在可以简化为几行代码：

```php
$client          = new JsonRpcClient($ctx, 'tcp://inventory:5557'); 
**$checkoutService = new CheckoutService($client);** 

$socket->on('message', function($msg) use ($ctx, $checkoutService, $pubSocket, $socket) { 
    echo "received checkout order $msg\n"; 

 **$checkoutService->handleCheckoutOrder($msg)->then(function() use ($pubSocket, $msg, $socket) {** 
 **$pubSocket->send($msg);** 
 **$socket->send(json_encode(['msg' => 'OK']));** 
 **}, function(\Exception $err) use ($socket) {** 
 **$socket->send(json_encode(['error' => $err->getMessage()]));** 
 **});** 
}); 

```

接下来，您可以开始处理 HTTP 服务器。为此，您首先需要一个简单的套接字服务器，然后将其传递到实际的 HTTP 服务器类中。这可以在运行事件循环之前的`server.php`中的任何时间点完成：

```php
**$httpSocket = new \React\Socket\Server($loop);** 
**$httpSocket->listen(8080, '0.0.0.0');** 
**$httpServer = new \React\Http\Server($httpSocket);** 

$loop->run(); 

```

HTTP 服务器本身有一个`'request'`事件，您可以为其注册一个监听函数（类似于 ZeroMQ 套接字的`'message'`事件）。监听函数作为参数传递了一个请求和一个响应对象。这些都是`React\Http\Request`和`React\Http\Response`类的实例：

```php
$httpServer->on('request', function(React\Http\Request $req, React\Http\Response $res) { 
    $res->writeHead(200); 
    $res->end('Hello World'); 
}); 

```

不幸的是，React HTTP 的`Request`和`Response`类与相应的 PSR-7 接口不兼容。但是，如果有需要，您可以相对容易地将它们转换，就像在第六章*构建聊天应用程序*中已经看到的那样，*桥接 Ratchet 和 PSR-7 应用程序*部分中。

在此监听函数中，您可以首先检查正确的请求方法和路径，并发送错误代码，否则：

```php
$httpServer->on('request', function(React\Http\Request $req, React\Http\Response $res) { 
 **if ($request->getPath() != '/orders') {** 
 **$msg = json_encode(['msg' => 'this resource does not exist']);** 
 **$response->writeHead(404, [** 
 **'Content-Type' => 'application/json;charset=utf8',** 
 **'Content-Length' => strlen($msg)** 
 **]);** 
 **$response->end($msg);** 
 **return;** 
 **}** 
 **if ($request->getMethod() != 'POST') {** 
 **$msg = json_encode(['msg' => 'this method is not allowed']);** 
 **$response->writeHead(405, [** 
 **'Content-Type' => 'application/json;charset=utf8',** 
 **'Content-Length' => strlen($msg)** 
 **]);** 
 **$response->end($msg);** 
 **return;** 
 **}** 
}); 

```

这就是问题所在。ReactPHP HTTP 服务器是如此异步，以至于当触发`request`事件时，请求正文尚未从网络套接字中读取。要获取实际的请求正文，您需要监听请求的`data`事件。但是，请求正文以 4096 字节的块读取，因此对于大型请求正文，数据事件实际上可能会被多次调用。读取完整的请求正文的最简单方法是检查`Content-Length`标头，并在数据事件处理程序中检查是否已经读取了确切数量的字节：

```php
$httpServer->on('request', function(React\Http\Request $req, React\Http\Response $res) { 
    // error checking omitted... 

 **$length = $req->getHeaders()['Content-Length'];** 
 **$body   = '';** 
 **$request->on('data', function(string $chunk) use (&$body) {** 
 **$body .= $chunk;** 
 **if (strlen($body) == $length) {** 
 **// body is complete!** 
 **}** 
 **});** 
}); 

```

当发送方在其请求中使用所谓的分块传输编码时，这是行不通的。但是，使用分块传输读取请求正文的工作方式类似；在这种情况下，退出条件不依赖于`Content-Length`标头，而是在读取第一个空块时。

在完整的请求正文被读取后，您可以将此正文传递给之前已经使用过的`$checkoutService`：

```php
$httpServer->on('request', function(React\Http\Request $req, React\Http\Response $res) use ($pubSocket, $checkoutService) { 
    // error checking omitted... 

    $length = $req->getHeaders()['Content-Length']; 
    $body   = ''; 

    $request->on('data', function(string $chunk) use (&$body, $pubSocket, $checkoutService) { 
        $body .= $chunk; 
        if (strlen($body) == $length) { 
 **$checkoutService->handleCheckoutOrder($body)** 
 **->then(function() use ($response, $body, $pubSocket) {**
 **$pubSocket->send($body);** 
 **$msg = json_encode(['msg' => 'OK']);** 
 **$response->writeHead(200, [** 
 **'Content-Type' => 'application/json',** 
 **'Content-Length' => strlen($msg)** 
 **]);** 
 **$response->end($msg);** 
 **}, function(\Exception $err) use ($response) {** 
 **$msg = json_encode(['msg' => $err->getMessage()]);** 
 **$response->writeHead(500, [** 
 **'Content-Type' => 'application/json',** 
 **'Content-Length' => strlen($msg)** 
 **]);** 
 **$response->end($msg);** 
 **});** 
        } 
    }); 
}); 

```

`CheckoutService`类的使用方式与以前完全相同。现在唯一的区别是如何将响应发送回客户端；如果原始请求是由 ZeroMQ REP 套接字接收的，则将相应的响应发送到发送请求的 REQ 套接字。现在，如果请求是由 HTTP 服务器接收的，则会发送具有相同内容的 HTTP 响应。

您可以使用 curl 或 HTTPie 等命令行工具测试新的 HTTP API：

```php
**$ http -v localhost:8080/orders
cart:='[{"articlenumber":1000,"amount":3}]' customer:='{"email":"john.doe@example.com"}'**

```

以下截图显示了使用前面的 HTTPie 命令测试新 API 端点时的示例输出：

![桥接 ZeroMQ 和 HTTP](img/image_07_011.jpg)

测试新的 HTTP API

# 总结

在本章中，您已经了解了 ZeroMQ 作为一种新的通信协议以及如何在 PHP 中使用它。与 HTTP 相比，ZeroMQ 支持比简单的请求/响应模式更复杂的通信模式。特别是发布/订阅和推送/拉取模式，它们允许您构建松散耦合的架构，可以轻松扩展新功能并且可以很好地扩展。

您还学会了如何使用 ReactPHP 框架构建使用事件循环的异步服务，以及如何使用承诺使异步性可管理。我们还讨论了如何将基于 ZeroMQ 的应用程序与*常规*HTTP API 集成。

虽然以前的章节都集中在不同的网络通信模式上（第五章中的 RESTful HTTP，*创建 RESTful Web 服务*，第六章中的 WebSockets，*构建聊天应用程序*，以及现在的 ZeroMQ），我们将在接下来的章节中重新开始，并学习如何使用 PHP 构建用于自定义表达式语言的解析器。
