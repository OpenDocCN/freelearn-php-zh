# 第七章。访问 Web 服务

在本章中，我们将涵盖以下主题：

+   在 PHP 和 XML 之间进行转换

+   创建一个简单的 REST 客户端

+   创建一个简单的 REST 服务器

+   创建一个简单的 SOAP 客户端

+   创建一个简单的 SOAP 服务器

# 介绍

向外部 Web 服务进行后台查询正成为任何 PHP Web 实践的日益重要的一部分。提供适当、及时和丰富的数据意味着为您的客户和您开发的网站带来更多的业务。我们首先介绍了一些旨在在**可扩展标记语言**（**XML**）和本机 PHP 之间进行数据转换的配方。接下来，我们将向您展示如何实现一个简单的**表述状态转移**（**REST**）客户端和服务器。之后，我们将把注意力转向**SOAP**客户端和服务器。

# 在 PHP 和 XML 之间进行转换

在考虑 PHP 本机数据类型和 XML 之间的转换时，我们通常将数组视为主要目标。基于此，从 PHP 数组转换为 XML 的过程与需要执行相反操作的方法有很大的不同。

### 注意

对象也可以考虑进行转换；然而，将对象方法呈现为 XML 是困难的。属性可以通过使用`get_object_vars()`函数来表示，该函数将对象属性读入数组中。

## 如何做...

1.  首先，我们定义一个`Application\Parse\ConvertXml`类。这个类将包含将从 XML 转换为 PHP 数组，反之亦然的方法。我们将需要 SPL 中的`SimpleXMLElement`和`SimpleXMLIterator`类：

```php
namespace Application\Parse;
use SimpleXMLIterator;
use SimpleXMLElement;
class ConvertXml
{
}
```

1.  接下来，我们定义一个`xmlToArray()`方法，它将接受一个`SimpleXMLIterator`实例作为参数。它将被递归调用，并将从 XML 文档生成一个 PHP 数组。我们利用`SimpleXMLIterator`能够通过 XML 文档前进的能力，使用`key()`、`current()`、`next()`和`rewind()`方法进行导航：

```php
public function xmlToArray(SimpleXMLIterator $xml) : array
{
  $a = array();
  for( $xml->rewind(); $xml->valid(); $xml->next() ) {
    if(!array_key_exists($xml->key(), $a)) {
      $a[$xml->key()] = array();
    }
    if($xml->hasChildren()){
      $a[$xml->key()][] = $this->xmlToArray($xml->current());
    }
    else{
      $a[$xml->key()] = (array) $xml->current()->attributes();
      $a[$xml->key()]['value'] = strval($xml->current());
    }
  }
  return $a;
}
```

1.  对于相反的过程，也称为递归，我们定义了两种方法。第一种方法`arrayToXml()`设置了一个初始的`SimpleXMLElement`实例，然后调用第二种方法`phpToXml()`：

```php
public function arrayToXml(array $a)
{
  $xml = new SimpleXMLElement(
  '<?xml version="1.0" standalone="yes"?><root></root>');
  $this->phpToXml($a, $xml);
  return $xml->asXML();
}
```

1.  请注意，在第二种方法中，我们使用`get_object_vars()`，以防数组元素是对象。您还会注意到，单独的数字不允许作为 XML 标签，这意味着在数字前面添加一些文本：

```php
protected function phpToXml($value, &$xml)
{
  $node = $value;
  if (is_object($node)) {
    $node = get_object_vars($node);
  }
  if (is_array($node)) {
    foreach ($node as $k => $v) {
      if (is_numeric($k)) {
        $k = 'number' . $k;
      }
      if (is_array($v)) {
          $newNode = $xml->addChild($k);
          $this->phpToXml($v, $newNode);
      } elseif (is_object($v)) {
          $newNode = $xml->addChild($k);
          $this->phpToXml($v, $newNode);
      } else {
          $xml->addChild($k, $v);
      }
    }
  } else  {
      $xml->addChild(self::UNKNOWN_KEY, $node);
  }
}
```

## 它是如何工作的...

作为一个示例 XML 文档，您可以使用美国国家气象局的**Web 服务定义语言**（**WSDL**）。这是一个描述 SOAP 服务的 XML 文档，可以在[`graphical.weather.gov/xml/SOAP_server/ndfdXMLserver.php?wsdl`](http://graphical.weather.gov/xml/SOAP_server/ndfdXMLserver.php?wsdl)找到。

我们将使用`SimpleXMLIterator`类来提供迭代机制。然后，您可以配置自动加载，并使用`xmlToArray()`获取`Application\Parse\ConvertXml`的实例，将 WSDL 转换为 PHP 数组：

```php
require __DIR__ . '/../Application/Autoload/Loader.php';
Application\Autoload\Loader::init(__DIR__ . '/..');
use Application\Parse\ConvertXml;
$wsdl = 'http://graphical.weather.gov/xml/'
. 'SOAP_server/ndfdXMLserver.php?wsdl';
$xml = new SimpleXMLIterator($wsdl, 0, TRUE);
$convert = new ConvertXml();
var_dump($convert->xmlToArray($xml));
```

结果数组显示如下：

![它是如何工作的...](img/B05314_07_01.jpg)

要执行相反的操作，请使用本配方中描述的`arrayToXml()`方法。作为源文档，您可以使用一个包含通过 O'Reilly Media 提供的 MongoDB 培训视频大纲的`source/data/mongo.db.global.php`文件（免责声明：由本作者提供！）。使用相同的自动加载程序配置和`Application\Parse\ConvertXml`的实例，这是您可以使用的示例代码：

```php
$convert = new ConvertXml();
header('Content-Type: text/xml');
echo $convert->arrayToXml(include CONFIG_FILE);
```

这是在浏览器中的输出：

![它是如何工作的...](img/B05314_07_02.jpg)

# 创建一个简单的 REST 客户端

REST 客户端使用**超文本传输协议**（**HTTP**）向外部 Web 服务生成请求。通过更改 HTTP 方法，我们可以导致外部服务执行不同的操作。虽然有很多可用的方法（或动词），但我们只关注`GET`和`POST`。在这个配方中，我们将使用**适配器**软件设计模式来呈现实现 REST 客户端的两种不同方式。

## 如何做...

1.  在我们定义 REST 客户端适配器之前，我们需要定义用于表示请求和响应信息的通用类。首先，我们将从一个抽象类开始，该类具有请求或响应所需的方法和属性：

```php
namespace Application\Web;

class AbstractHttp
{
```

1.  接下来，我们定义代表 HTTP 信息的类常量：

```php
const METHOD_GET = 'GET';
const METHOD_POST = 'POST';
const METHOD_PUT = 'PUT';
const METHOD_DELETE = 'DELETE';
const CONTENT_TYPE_HTML = 'text/html';
const CONTENT_TYPE_JSON = 'application/json';
const CONTENT_TYPE_FORM_URL_ENCODED = 
  'application/x-www-form-urlencoded';
const HEADER_CONTENT_TYPE = 'Content-Type';
const TRANSPORT_HTTP = 'http';
const TRANSPORT_HTTPS = 'https';
const STATUS_200 = '200';
const STATUS_401 = '401';
const STATUS_500 = '500';
```

1.  然后，我们定义了请求或响应所需的属性：

```php
protected $uri;      // i.e. http://xxx.com/yyy
protected $method;    // i.e. GET, PUT, POST, DELETE
protected $headers;  // HTTP headers
protected $cookies;  // cookies
protected $metaData;  // information about the transmission
protected $transport;  // i.e. http or https
protected $data = array();
```

1.  逻辑上，我们需要为这些属性定义 getter 和 setter：

```php
public function setMethod($method)
{
  $this->method = $method;
}
public function getMethod()
{
  return $this->method ?? self::METHOD_GET;
}
// etc.
```

1.  有些属性需要通过键访问。为此，我们定义了`getXxxByKey()`和`setXxxByKey()`方法：

```php
public function setHeaderByKey($key, $value)
{
  $this->headers[$key] = $value;
}
public function getHeaderByKey($key)
{
  return $this->headers[$key] ?? NULL;
}
public function getDataByKey($key)
{
  return $this->data[$key] ?? NULL;
}
public function getMetaDataByKey($key)
{
  return $this->metaData[$key] ?? NULL;
}
```

1.  在某些情况下，请求将需要参数。我们假设参数将以 PHP 数组的形式存储在`$data`属性中。然后我们可以使用`http_build_query()`函数构建请求 URL：

```php
public function setUri($uri, array $params = NULL)
{
  $this->uri = $uri;
  $first = TRUE;
  if ($params) {
    $this->uri .= '?' . http_build_query($params);
  }
}
public function getDataEncoded()
{
  return http_build_query($this->getData());
}
```

1.  最后，我们根据原始请求设置`$transport`：

```php
public function setTransport($transport = NULL)
{
  if ($transport) {
      $this->transport = $transport;
  } else {
      if (substr($this->uri, 0, 5) == self::TRANSPORT_HTTPS) {
          $this->transport = self::TRANSPORT_HTTPS;
      } else {
          $this->transport = self::TRANSPORT_HTTP;
      }
    }
  }
```

1.  在这个示例中，我们将定义一个`Application\Web\Request`类，当我们希望生成一个请求时，可以接受参数，或者在实现接受请求的服务器时，可以填充属性与传入的请求信息：

```php
namespace Application\Web;
class Request extends AbstractHttp
{
  public function __construct(
    $uri = NULL, $method = NULL, array $headers = NULL, 
    array $data = NULL, array $cookies = NULL)
    {
      if (!$headers) $this->headers = $_SERVER ?? array();
      else $this->headers = $headers;
      if (!$uri) $this->uri = $this->headers['PHP_SELF'] ?? '';
      else $this->uri = $uri;
      if (!$method) $this->method = 
        $this->headers['REQUEST_METHOD'] ?? self::METHOD_GET;
      else $this->method = $method;
      if (!$data) $this->data = $_REQUEST ?? array();
      else $this->data = $data;
      if (!$cookies) $this->cookies = $_COOKIE ?? array();
      else $this->cookies = $cookies;
      $this->setTransport();
    }  
}
```

1.  现在我们可以转向响应类。在这种情况下，我们将定义一个`Application\Web\Received`类。这个名称反映了我们正在重新打包从外部网络服务接收到的数据的事实：

```php
namespace Application\Web;
class Received extends AbstractHttp
{
  public function __construct(
    $uri = NULL, $method = NULL, array $headers = NULL, 
    array $data = NULL, array $cookies = NULL)
  {
    $this->uri = $uri;
    $this->method = $method;
    $this->headers = $headers;
    $this->data = $data;
    $this->cookies = $cookies;
    $this->setTransport();
  }  
}
```

### 创建基于流的 REST 客户端

我们现在准备考虑两种不同的实现 REST 客户端的方式。第一种方法是使用一个称为**Streams**的底层 PHP I/O 层。这一层提供了一系列包装器，用于访问外部流资源。默认情况下，任何 PHP 文件命令都将使用文件包装器，这使得可以访问本地文件系统。我们将使用`http://`或`https://`包装器来实现`Application\Web\Client\Streams`适配器：

1.  首先，我们定义一个`Application\Web\Client\Streams`类：

```php
namespace Application\Web\Client;
use Application\Web\ { Request, Received };
class Streams
{
  const BYTES_TO_READ = 4096;
```

1.  接下来，我们定义一个方法来将请求发送到外部网络服务。在`GET`的情况下，我们将参数添加到 URI 中。在`POST`的情况下，我们创建一个包含元数据的流上下文，指示远程服务我们正在提供数据。使用 PHP Streams，发出请求只是简单地组合 URI，在`POST`的情况下设置流上下文。然后我们使用一个简单的`fopen()`：

```php
public static function send(Request $request)
{
  $data = $request->getDataEncoded();
  $received = new Received();
  switch ($request->getMethod()) {
    case Request::METHOD_GET :
      if ($data) {
        $request->setUri($request->getUri() . '?' . $data);
      }
      $resource = fopen($request->getUri(), 'r');
      break;
    case Request::METHOD_POST :
      $opts = [
        $request->getTransport() => 
        [
          'method'  => Request::METHOD_POST,
          'header'  => Request::HEADER_CONTENT_TYPE 
          . ': ' . Request::CONTENT_TYPE_FORM_URL_ENCODED,
          'content' => $data
        ]
      ];
      $resource = fopen($request->getUri(), 'w', 
      stream_context_create($opts));
      break;
    }
    return self::getResults($received, $resource);
}
```

1.  最后，我们将看一下如何将结果检索并打包成一个`Received`对象。您会注意到我们添加了一个解码以 JSON 格式接收数据的规定：

```php
protected static function getResults(Received $received, $resource)
{
  $received->setMetaData(stream_get_meta_data($resource));
  $data = $received->getMetaDataByKey('wrapper_data');
  if (!empty($data) && is_array($data)) {
    foreach($data as $item) {
      if (preg_match('!^HTTP/\d\.\d (\d+?) .*?$!', 
          $item, $matches)) {
          $received->setHeaderByKey('status', $matches[1]);
      } else {
          list($key, $value) = explode(':', $item);
          $received->setHeaderByKey($key, trim($value));
      }
    }
  }
  $payload = '';
  while (!feof($resource)) {
    $payload .= fread($resource, self::BYTES_TO_READ);
  }
  if ($received->getHeaderByKey(Received::HEADER_CONTENT_TYPE)) {
    switch (TRUE) {
      case stripos($received->getHeaderByKey(
                   Received::HEADER_CONTENT_TYPE), 
                   Received::CONTENT_TYPE_JSON) !== FALSE:
        $received->setData(json_decode($payload));
        break;
      default :
        $received->setData($payload);
        break;
          }
    }
    return $received;
}
```

### 定义基于 cURL 的 REST 客户端

我们现在将看一下我们的第二种 REST 客户端的方法，其中之一是基于 cURL 扩展的：

1.  对于这种方法，我们将假设相同的请求和响应类。初始类定义与之前讨论的 Streams 客户端基本相同：

```php
namespace Application\Web\Client;
use Application\Web\ { Request, Received };
class Curl
{
```

1.  `send()`方法比使用 Streams 时要简单得多。我们所需要做的就是定义一个选项数组，然后让 cURL 来处理剩下的事情：

```php
public static function send(Request $request)
{
  $data = $request->getDataEncoded();
  $received = new Received();
  switch ($request->getMethod()) {
    case Request::METHOD_GET :
      $uri = ($data) 
        ? $request->getUri() . '?' . $data 
        : $request->getUri();
          $options = [
            CURLOPT_URL => $uri,
            CURLOPT_HEADER => 0,
            CURLOPT_RETURNTRANSFER => TRUE,
            CURLOPT_TIMEOUT => 4
          ];
          break;
```

1.  `POST`需要稍有不同的 cURL 参数：

```php
case Request::METHOD_POST :
  $options = [
    CURLOPT_POST => 1,
    CURLOPT_HEADER => 0,
    CURLOPT_URL => $request->getUri(),
    CURLOPT_FRESH_CONNECT => 1,
    CURLOPT_RETURNTRANSFER => 1,
    CURLOPT_FORBID_REUSE => 1,
    CURLOPT_TIMEOUT => 4,
    CURLOPT_POSTFIELDS => $data
  ];
  break;
}
```

1.  然后，我们执行一系列 cURL 函数，并通过`getResults()`运行结果：

```php
$ch = curl_init();
curl_setopt_array($ch, ($options));
if( ! $result = curl_exec($ch))
{
  trigger_error(curl_error($ch));
}
$received->setMetaData(curl_getinfo($ch));
curl_close($ch);
return self::getResults($received, $result);
}
```

1.  `getResults()`方法将结果打包成一个`Received`对象：

```php
protected static function getResults(Received $received, $payload)
{
  $type = $received->getMetaDataByKey('content_type');
  if ($type) {
    switch (TRUE) {
      case stripos($type, 
          Received::CONTENT_TYPE_JSON) !== FALSE):
          $received->setData(json_decode($payload));
          break;
      default :
          $received->setData($payload);
          break;
    }
  }
  return $received;
}
```

## 工作原理...

确保将所有前面的代码复制到这些类中：

+   `Application\Web\AbstractHttp`

+   `Application\Web\Request`

+   `Application\Web\Received`

+   `Application\Web\Client\Streams`

+   `Application\Web\Client\Curl`

在这个示例中，您可以向 Google Maps API 发出 REST 请求，以获取两个点之间的驾驶路线。您还需要按照[`developers.google.com/maps/documentation/directions/get-api-key`](https://developers.google.com/maps/documentation/directions/get-api-key)上的说明创建一个 API 密钥。

然后，您可以定义一个`chap_07_simple_rest_client_google_maps_curl.php`调用脚本，使用`Curl`客户端发出请求。您还可以考虑定义一个`chap_07_simple_rest_client_google_maps_streams.php`调用脚本，使用`Streams`客户端发出请求：

```php
<?php
define('DEFAULT_ORIGIN', 'New York City');
define('DEFAULT_DESTINATION', 'Redondo Beach');
define('DEFAULT_FORMAT', 'json');
$apiKey = include __DIR__ . '/google_api_key.php';
require __DIR__ . '/../Application/Autoload/Loader.php';
Application\Autoload\Loader::init(__DIR__ . '/..');
use Application\Web\Request;
use Application\Web\Client\Curl;
```

然后您可以获取起点和目的地：

```php
$start = $_GET['start'] ?? DEFAULT_ORIGIN;
$end   = $_GET['end'] ?? DEFAULT_DESTINATION;
$start = strip_tags($start);
$end   = strip_tags($end);
```

现在，您可以填充`Request`对象，并使用它来生成请求：

```php
$request = new Request(
  'https://maps.googleapis.com/maps/api/directions/json',
  Request::METHOD_GET,
  NULL,
  ['origin' => $start, 'destination' => $end, 'key' => $apiKey],
  NULL
);

$received = Curl::send($request);
$routes   = $received->getData()->routes[0];
include __DIR__ . '/chap_07_simple_rest_client_google_maps_template.php';
```

为了说明的目的，您还可以定义一个表示视图逻辑的模板，以显示请求结果：

```php
<?php foreach ($routes->legs as $item) : ?>
  <!-- Trip Info -->
  <br>Distance: <?= $item->distance->text; ?>
  <br>Duration: <?= $item->duration->text; ?>
  <!-- Driving Directions -->
  <table>
    <tr>
    <th>Distance</th><th>Duration</th><th>Directions</th>
    </tr>
    <?php foreach ($item->steps as $step) : ?>
    <?php $class = ($count++ & 01) ? 'color1' : 'color2'; ?>
    <tr>
    <td class="<?= $class ?>"><?= $step->distance->text ?></td>
    <td class="<?= $class ?>"><?= $step->duration->text ?></td>
    <td class="<?= $class ?>">
    <?= $step->html_instructions ?></td>
    </tr>
    <?php endforeach; ?>
  </table>
<?php endforeach; ?>
```

以下是在浏览器中看到的请求结果：

![它是如何工作的...](img/B05314_07_03.jpg)

## 还有更多...

**PHP 标准建议**（**PSR-7**）精确地定义了在 PHP 应用程序之间进行请求时要使用的请求和响应对象。这在附录中有详细介绍，*定义 PSR-7 类*。

## 另请参阅

有关`Streams`的更多信息，请参阅此 PHP 文档页面[`php.net/manual/en/book.stream.php`](http://php.net/manual/en/book.stream.php)。一个经常被问到的问题是“HTTP PUT 和 POST 之间有什么区别？”关于这个话题的优秀讨论，请参考[`stackoverflow.com/questions/107390/whats-the-difference-between-a-post-and-a-put-http-request`](http://stackoverflow.com/questions/107390/whats-the-difference-between-a-post-and-a-put-http-request)。有关从 Google 获取 API 密钥的更多信息，请参考以下网页：

[`developers.google.com/maps/documentation/directions/get-api-key`](https://developers.google.com/maps/documentation/directions/get-api-key)

[`developers.google.com/maps/documentation/directions/intro#Introduction`](https://developers.google.com/maps/documentation/directions/intro#Introduction)

# 创建一个简单的 REST 服务器

在实现 REST 服务器时有几个考虑因素。回答这三个问题将让您定义 REST 服务：

+   如何捕获原始请求？

+   您想要发布什么**应用程序编程接口**（**API**）？

+   您打算如何将 HTTP 动词（例如`GET`、`PUT`、`POST`和`DELETE`）映射到 API 方法？

## 如何做...

1.  我们将通过构建在前一篇文章*创建一个简单的 REST 客户端*中定义的请求和响应类来实现我们的 REST 服务器。回顾前一篇文章中讨论的类，包括以下内容：

+   `Application\Web\AbstractHttp`

+   `Application\Web\Request`

+   `Application\Web\Received`

1.  我们还需要定义一个正式的`Application\Web\Response`响应类，基于`AbstractHttp`。这个类与其他类的主要区别在于它接受`Application\Web\Request`的实例作为参数。主要工作是在`__construct()`方法中完成的。设置`Content-Type`标头和状态也很重要：

```php
namespace Application\Web;
class Response extends AbstractHttp
{

  public function __construct(Request $request = NULL, 
                              $status = NULL, $contentType = NULL)
  {
    if ($request) {
      $this->uri = $request->getUri();
      $this->data = $request->getData();
      $this->method = $request->getMethod();
      $this->cookies = $request->getCookies();
      $this->setTransport();
    }
    $this->processHeaders($contentType);
    if ($status) {
      $this->setStatus($status);
    }
  }
  protected function processHeaders($contentType)
  {
    if (!$contentType) {
      $this->setHeaderByKey(self::HEADER_CONTENT_TYPE, 
        self::CONTENT_TYPE_JSON);
    } else {
      $this->setHeaderByKey(self::HEADER_CONTENT_TYPE, 
        $contentType);
    }
  }
  public function setStatus($status)
  {
    $this->status = $status;
  }
  public function getStatus()
  {
    return $this->status;
  }
}
```

1.  我们现在可以定义`Application\Web\Rest\Server`类。您可能会对它有多简单感到惊讶。真正的工作是在相关的 API 类中完成的：

### 注意

请注意 PHP 7 组使用语法的使用：

```php
use Application\Web\ { Request,Response,Received }
```

```php
namespace Application\Web\Rest;
use Application\Web\ { Request, Response, Received };
class Server
{
  protected $api;
  public function __construct(ApiInterface $api)
  {
    $this->api = $api;
  }
```

1.  接下来，我们定义一个`listen()`方法，作为请求的目标。服务器实现的核心是这行代码：

```php
$jsonData = json_decode(file_get_contents('php://input'),true);
```

1.  这捕获了假定为 JSON 格式的原始输入：

```php
public function listen()
{
  $request  = new Request();
  $response = new Response($request);
  $getPost  = $_REQUEST ?? array();
  $jsonData = json_decode(
    file_get_contents('php://input'),true);
  $jsonData = $jsonData ?? array();
  $request->setData(array_merge($getPost,$jsonData));
```

### 注意

我们还添加了身份验证的规定。否则，任何人都可以发出请求并获取潜在的敏感数据。您会注意到我们没有服务器类执行身份验证；相反，我们把它留给 API 类：

```php
if (!$this->api->authenticate($request)) {
    $response->setStatus(Request::STATUS_401);
    echo $this->api::ERROR;
    exit;
}
```

1.  然后将 API 方法映射到主要的 HTTP 方法`GET`、`PUT`、`POST`和`DELETE`：

```php
$id = $request->getData()[$this->api::ID_FIELD] ?? NULL;
switch (strtoupper($request->getMethod())) {
  case Request::METHOD_POST :
    $this->api->post($request, $response);
    break;
  case Request::METHOD_PUT :
    $this->api->put($request, $response);
    break;
  case Request::METHOD_DELETE :
    $this->api->delete($request, $response);
    break;
  case Request::METHOD_GET :
  default :
    // return all if no params
  $this->api->get($request, $response);
}
```

1.  最后，我们打包响应并发送它，以 JSON 编码：

```php
  $this->processResponse($response);
  echo json_encode($response->getData());
}
```

1.  `processResponse()`方法设置标头，并确保结果打包为`Application\Web\Response`对象：

```php
protected function processResponse($response)
{
  if ($response->getHeaders()) {
    foreach ($response->getHeaders() as $key => $value) {
      header($key . ': ' . $value, TRUE, 
             $response->getStatus());
    }
  }        
  header(Request::HEADER_CONTENT_TYPE 
  . ': ' . Request::CONTENT_TYPE_JSON, TRUE);
  if ($response->getCookies()) {
    foreach ($response->getCookies() as $key => $value) {
      setcookie($key, $value);
    }
  }
}
```

1.  如前所述，API 类完成了真正的工作。我们首先定义一个抽象类，确保主要方法`get()`，`put()`等都有对应的实现，并且所有这些方法都接受请求和响应对象作为参数。您可能会注意到我们添加了一个`generateToken()`方法，它使用 PHP 7 的`random_bytes()`函数生成一个真正随机的 16 字节序列：

```php
namespace Application\Web\Rest;
use Application\Web\ { Request, Response };
abstract class AbstractApi implements ApiInterface
{
  const TOKEN_BYTE_SIZE  = 16;
  protected $registeredKeys;
  abstract public function get(Request $request, 
                               Response $response);
  abstract public function put(Request $request, 
                               Response $response);
  abstract public function post(Request $request, 
                                Response $response);
  abstract public function delete(Request $request, 
                                  Response $response);
  abstract public function authenticate(Request $request);
  public function __construct($registeredKeys, $tokenField)
  {
    $this->registeredKeys = $registeredKeys;
  }
  public static function generateToken()
  {
    return bin2hex(random_bytes(self::TOKEN_BYTE_SIZE));    
  }
}
```

1.  我们还定义了一个相应的接口，可用于架构和设计目的，以及代码开发控制：

```php
namespace Application\Web\Rest;
use Application\Web\ { Request, Response };
interface ApiInterface
{
  public function get(Request $request, Response $response);
  public function put(Request $request, Response $response);
  public function post(Request $request, Response $response);
  public function delete(Request $request, Response $response);
  public function authenticate(Request $request);
}
```

1.  在这里，我们提供了一个基于`AbstractApi`的示例 API。这个类利用了在第五章中定义的数据库类，*与数据库交互*：

```php
namespace Application\Web\Rest;
use Application\Web\ { Request, Response, Received };
use Application\Entity\Customer;
use Application\Database\ { Connection, CustomerService };

class CustomerApi extends AbstractApi
{
  const ERROR = 'ERROR';
  const ERROR_NOT_FOUND = 'ERROR: Not Found';
  const SUCCESS_UPDATE = 'SUCCESS: update succeeded';
  const SUCCESS_DELETE = 'SUCCESS: delete succeeded';
  const ID_FIELD = 'id';      // field name of primary key
  const TOKEN_FIELD = 'token';  // field used for authentication
  const LIMIT_FIELD = 'limit';
  const OFFSET_FIELD = 'offset';
  const DEFAULT_LIMIT = 20;
  const DEFAULT_OFFSET = 0;

  protected $service;

  public function __construct($registeredKeys, 
                              $dbparams, $tokenField = NULL)
  {
    parent::__construct($registeredKeys, $tokenField);
    $this->service = new CustomerService(
      new Connection($dbparams));
  }
```

1.  所有方法都接收请求和响应作为参数。您会注意到使用`getDataByKey()`来检索数据项。实际的数据库交互是由服务类执行的。您可能还会注意到，在所有情况下，我们都设置了 HTTP 状态码来通知客户端成功或失败。在`get()`的情况下，我们会查找 ID 参数。如果收到，我们只提供有关单个客户的信息。否则，我们使用限制和偏移量提供所有客户的列表：

```php
public function get(Request $request, Response $response)
{
  $result = array();
  $id = $request->getDataByKey(self::ID_FIELD) ?? 0;
  if ($id > 0) {
      $result = $this->service->
        fetchById($id)->entityToArray();  
  } else {
    $limit  = $request->getDataByKey(self::LIMIT_FIELD) 
      ?? self::DEFAULT_LIMIT;
    $offset = $request->getDataByKey(self::OFFSET_FIELD) 
      ?? self::DEFAULT_OFFSET;
    $result = [];
    $fetch = $this->service->fetchAll($limit, $offset);
    foreach ($fetch as $row) {
      $result[] = $row;
    }
  }
  if ($result) {
      $response->setData($result);
      $response->setStatus(Request::STATUS_200);
  } else {
      $response->setData([self::ERROR_NOT_FOUND]);
      $response->setStatus(Request::STATUS_500);
  }
}
```

1.  `put()`方法用于插入客户数据：

```php
public function put(Request $request, Response $response)
{
  $cust = Customer::arrayToEntity($request->getData(), 
                                  new Customer());
  if ($newCust = $this->service->save($cust)) {
      $response->setData(['success' => self::SUCCESS_UPDATE, 
                          'id' => $newCust->getId()]);
      $response->setStatus(Request::STATUS_200);
  } else {
      $response->setData([self::ERROR]);
      $response->setStatus(Request::STATUS_500);
  }      
}
```

1.  `post()`方法用于更新现有的客户条目：

```php
public function post(Request $request, Response $response)
{
  $id = $request->getDataByKey(self::ID_FIELD) ?? 0;
  $reqData = $request->getData();
  $custData = $this->service->
    fetchById($id)->entityToArray();
  $updateData = array_merge($custData, $reqData);
  $updateCust = Customer::arrayToEntity($updateData, 
  new Customer());
  if ($this->service->save($updateCust)) {
      $response->setData(['success' => self::SUCCESS_UPDATE, 
                          'id' => $updateCust->getId()]);
      $response->setStatus(Request::STATUS_200);
  } else {
      $response->setData([self::ERROR]);
      $response->setStatus(Request::STATUS_500);
  }      
}
```

1.  如其名称所示，`delete()`会删除客户条目：

```php
public function delete(Request $request, Response $response)
{
  $id = $request->getDataByKey(self::ID_FIELD) ?? 0;
  $cust = $this->service->fetchById($id);
  if ($cust && $this->service->remove($cust)) {
      $response->setData(['success' => self::SUCCESS_DELETE, 
                          'id' => $id]);
      $response->setStatus(Request::STATUS_200);
  } else {
      $response->setData([self::ERROR_NOT_FOUND]);
      $response->setStatus(Request::STATUS_500);
  }
}
```

1.  最后，我们定义`authenticate()`来提供一个低级机制来保护 API 使用，例如在这个例子中：

```php
public function authenticate(Request $request)
{
  $authToken = $request->getDataByKey(self::TOKEN_FIELD) 
    ?? FALSE;
  if (in_array($authToken, $this->registeredKeys, TRUE)) {
      return TRUE;
  } else {
      return FALSE;
  }
}
}
```

## 它是如何工作的...

定义以下在前面的教程中讨论的类：

+   `Application\Web\AbstractHttp`

+   `Application\Web\Request`

+   `Application\Web\Received`

然后，您可以定义以下在本教程中描述的类，总结在下表中：

| 类 Application\Web\* | 在这些步骤中讨论 |
| --- | --- |
| `Response` | 2 |
| `Rest\Server` | 3 - 8 |
| `Rest\AbstractApi` | 9 |
| `Rest\ApiInterface` | 10 |
| `Rest\CustomerApi` | 11 - 16 |

现在您可以自由开发自己的 API 类。但是，如果您选择遵循示例`Application\Web\Rest\CustomerApi`，您还需要确保实现这些类，这些类在第五章中有介绍，*与数据库交互*：

+   `Application\Entity\Customer`

+   `Application\Database\Connection`

+   `Application\Database\CustomerService`

现在您可以定义一个`chap_07_simple_rest_server.php`脚本来调用 REST 服务器：

```php
<?php
$dbParams = include __DIR__ .  '/../../config/db.config.php';
require __DIR__ . '/../Application/Autoload/Loader.php';
Application\Autoload\Loader::init(__DIR__ . '/..');
use Application\Web\Rest\Server;
use Application\Web\Rest\CustomerApi;
$apiKey = include __DIR__ . '/api_key.php';
$server = new Server(new CustomerApi([$apiKey], $dbParams, 'id'));
$server->listen();
```

然后，您可以使用内置的 PHP 7 开发服务器来监听端口`8080`以接收 REST 请求：

```php
**php -S localhost:8080 chap_07_simple_rest_server.php** 

```

要测试您的 API，可以使用`Application\Web\Rest\AbstractApi::generateToken()`方法生成一个认证令牌，然后将其放入一个`api_key.php`文件中，类似于这样：

```php
<?php return '79e9b5211bbf2458a4085707ea378129';
```

然后，您可以使用通用的 API 客户端（例如前面介绍的那种）或浏览器插件，比如 Chao Zhou 的 RESTClient（有关更多信息，请参见[`restclient.net/`](http://restclient.net/)）来生成示例请求。确保在请求中包含令牌，否则定义的 API 将拒绝请求。

这是一个基于`AbstractApi`的`POST`请求的示例，用于`ID`为`1`，将`balance`字段设置为`888888`的值：

![它是如何工作的...](img/B05314_07_04.jpg)

## 还有更多...

有许多库可以帮助您实现 REST 服务器。我最喜欢的之一是一个在单个文件中实现 REST 服务器的示例：[`www.leaseweb.com/labs/2015/10/creating-a-simple-rest-api-in-php/`](https://www.leaseweb.com/labs/2015/10/creating-a-simple-rest-api-in-php/)

各种框架，如 CodeIgniter 和 Zend Framework，也有 REST 服务器实现。

# 创建一个简单的 SOAP 客户端

与实现 REST 客户端或服务器的过程相比，使用 SOAP 非常容易，因为有一个 PHP SOAP 扩展提供了这两种功能。

### 注意

一个经常被问到的问题是“SOAP 和 REST 之间有什么区别？” SOAP 在内部使用 XML 作为数据格式。SOAP 使用 HTTP 但仅用于传输，否则不了解其他 HTTP 方法。REST 直接操作 HTTP，并且可以使用任何数据格式，但首选 JSON。另一个关键区别是 SOAP 可以与 WSDL 一起操作，这使得服务自描述，因此更容易公开。因此，SOAP 服务通常由国家卫生组织等公共机构提供。

## 操作步骤...

在本例中，我们将为美国国家气象局提供的现有 SOAP 服务进行 SOAP 请求：

1.  首先要考虑的是识别**WSDL**文档。WSDL 是描述服务的 XML 文档：

```php
$wsdl = 'http://graphical.weather.gov/xml/SOAP_server/'
  . 'ndfdXMLserver.php?wsdl';
```

1.  接下来，我们使用 WSDL 创建一个`soap client`实例：

```php
$soap = new SoapClient($wsdl, array('trace' => TRUE));
```

1.  然后，我们可以自由地初始化一些变量，以期待天气预报请求：

```php
$units = 'm';
$params = '';
$numDays = 7;
$weather = '';
$format = '24 hourly';
$startTime = new DateTime();
```

1.  然后，我们可以进行`LatLonListCityNames()` SOAP 请求，该请求在 WSDL 中标识为一个操作，以获取服务支持的城市列表。请求以 XML 格式返回，这表明需要创建一个`SimpleXLMElement`实例：

```php
$xml = new SimpleXMLElement($soap->LatLonListCityNames(1));
```

1.  不幸的是，城市及其对应的纬度和经度列表在单独的 XML 节点中。因此，我们使用`array_combine()` PHP 函数创建一个关联数组，其中纬度/经度是键，城市名是值。然后，我们可以稍后使用这个数组来呈现 HTML `SELECT`下拉列表，使用`asort()`对列表进行按字母排序：

```php
$cityNames = explode('|', $xml->cityNameList);
$latLonCity = explode(' ', $xml->latLonList);
$cityLatLon = array_combine($latLonCity, $cityNames);
asort($cityLatLon);
```

1.  然后，我们可以按以下方式从 Web 请求中获取城市数据：

```php
$currentLatLon = (isset($_GET['city'])) ? strip_tags(urldecode($_GET['city'])) : '';
```

1.  我们希望进行的 SOAP 调用是`NDFDgenByDay()`。我们可以通过检查 WSDL 来确定提供给 SOAP 服务器的参数的性质：

```php
<message name="NDFDgenByDayRequest">
<part name="latitude" type="xsd:decimal"/>
<part name="longitude" type="xsd:decimal"/>
<part name="startDate" type="xsd:date"/>
<part name="numDays" type="xsd:integer"/>
<part name="Unit" type="xsd:string"/>
<part name="format" type="xsd:string"/>
</message>
```

1.  如果设置了`$currentLatLon`的值，我们可以处理请求。我们将请求包装在`try {} catch {}`块中，以防抛出任何异常：

```php
if ($currentLatLon) {
  list($lat, $lon) = explode(',', $currentLatLon);
  try {
      $weather = $soap->NDFDgenByDay($lat,$lon,
        $startTime->format('Y-m-d'),$numDays,$unit,$format);
  } catch (Exception $e) {
      $weather .= PHP_EOL;
      $weather .= 'Latitude: ' . $lat . ' | Longitude: ' . $lon;
      $weather .= 'ERROR' . PHP_EOL;
      $weather .= $e->getMessage() . PHP_EOL;
      $weather .= $soap->__getLastResponse() . PHP_EOL;
  }
}
?>
```

## 工作原理...

将所有前面的代码复制到`chap_07_simple_soap_client_weather_service.php`文件中。然后，您可以添加视图逻辑，显示带有城市列表的表单，以及结果：

```php
<form method="get" name="forecast">
<br> City List: 
<select name="city">
<?php foreach ($cityLatLon as $latLon => $city) : ?>
<?php $select = ($currentLatLon == $latLon) ? ' selected' : ''; ?>
<option value="<?= urlencode($latLon) ?>" <?= $select ?>>
<?= $city ?></option>
<?php endforeach; ?>
</select>
<br><input type="submit" value="OK"></td>
</form>
<pre>
<?php var_dump($weather); ?>
</pre>
```

以下是在浏览器中请求俄亥俄州克利夫兰天气预报的结果：

![工作原理...](img/B05314_07_05.jpg)

## 另请参阅

有关 SOAP 和 REST 之间的区别的讨论，请参阅[`stackoverflow.com/questions/209905/representational-state-transfer-rest-and-simple-object-access-protocol-soap?lq=1`](http://stackoverflow.com/questions/209905/representational-state-transfer-rest-and-simple-object-access-protocol-soap?lq=1)上的文章。

# 创建一个简单的 SOAP 服务器

与 SOAP 客户端一样，我们可以使用 PHP SOAP 扩展来实现 SOAP 服务器。实现中最困难的部分将是从 API 类生成 WSDL。我们在这里不涵盖该过程，因为有许多好的 WSDL 生成器可用。

## 操作步骤...

1.  首先，您需要一个 API，该 API 将由 SOAP 服务器处理。在本例中，我们定义了一个`Application\Web\Soap\ProspectsApi`类，允许我们创建、读取、更新和删除`prospects`表：

```php
namespace Application\Web\Soap;
use PDO;
class ProspectsApi
{
  protected $registerKeys;
  protected $pdo;

  public function __construct($pdo, $registeredKeys)
  {
    $this->pdo = $pdo;
    $this->registeredKeys = $registeredKeys;
  }
}
```

1.  然后，我们定义与创建、读取、更新和删除相对应的方法。在本例中，方法名为`put()`、`get()`、`post()`和`delete()`。这些方法依次调用生成 SQL 请求的方法，这些方法从 PDO 实例执行。`get()`的示例如下：

```php
public function get(array $request, array $response)
{
  if (!$this->authenticate($request)) return FALSE;
  $result = array();
  $id = $request[self::ID_FIELD] ?? 0;
  $email = $request[self::EMAIL_FIELD] ?? 0;
  if ($id > 0) {
      $result = $this->fetchById($id);  
      $response[self::ID_FIELD] = $id;
  } elseif ($email) {
      $result = $this->fetchByEmail($email);
      $response[self::ID_FIELD] = $result[self::ID_FIELD] ?? 0;
  } else {
      $limit = $request[self::LIMIT_FIELD] 
        ?? self::DEFAULT_LIMIT;
      $offset = $request[self::OFFSET_FIELD] 
        ?? self::DEFAULT_OFFSET;
      $result = [];
      foreach ($this->fetchAll($limit, $offset) as $row) {
        $result[] = $row;
      }
  }
  $response = $this->processResponse(
    $result, $response, self::SUCCESS, self::ERROR);
    return $response;
  }

  protected function processResponse($result, $response, 
                                     $success_code, $error_code)
  {
    if ($result) {
        $response['data'] = $result;
        $response['code'] = $success_code;
        $response['status'] = self::STATUS_200;
    } else {
        $response['data'] = FALSE;
        $response['code'] = self::ERROR_NOT_FOUND;
        $response['status'] = self::STATUS_500;
    }
    return $response;
  }
```

1.  然后，您可以从您的 API 生成 WSDL。有许多基于 PHP 的 WSDL 生成器可用（请参阅*还有更多...*部分）。大多数要求您在将要发布的方法之前添加`phpDocumentor`标签。在我们的示例中，两个参数都是数组。以下是先前讨论的 API 的完整 WSDL：

```php
<?xml version="1.0" encoding="UTF-8"?>
  <wsdl:definitions  targetNamespace="php7cookbook"    >
  <wsdl:message name="getSoapIn">
    <wsdl:part name="request" type="tns:array" />
    <wsdl:part name="response" type="tns:array" />
  </wsdl:message>
  <wsdl:message name="getSoapOut">
    <wsdl:part name="return" type="tns:array" />
  </wsdl:message>
  <!—some nodes removed to conserve space -->
  <wsdl:portType name="CustomerApiSoap">
  <!—some nodes removed to conserve space -->
  <wsdl:binding name="CustomerApiSoap" type="tns:CustomerApiSoap">
  <soap:binding transport="http://schemas.xmlsoap.org/soap/http" style="rpc" />
    <wsdl:operation name="get">
      <soap:operation soapAction="php7cookbook#get" />
        <wsdl:input>
          <soap:body use="encoded" encodingStyle= "http://schemas.xmlsoap.org/soap/encoding/" namespace="php7cookbook" parts="request response" />
        </wsdl:input>
        <wsdl:output>
          <soap:body use="encoded" encodingStyle= "http://schemas.xmlsoap.org/soap/encoding/" namespace="php7cookbook" parts="return" />
        </wsdl:output>
    </wsdl:operation>
  <!—some nodes removed to conserve space -->
  </wsdl:binding>
  <wsdl:service name="CustomerApi">
    <wsdl:port name="CustomerApiSoap" binding="tns:CustomerApiSoap">
    <soap:address location="http://localhost:8080/" />
    </wsdl:port>
  </wsdl:service>
  </wsdl:definitions>
```

1.  接下来，创建一个`chap_07_simple_soap_server.php`文件，用于执行 SOAP 服务器。首先定义 WSDL 的位置和任何其他必要的文件（在本例中，用于数据库配置的文件）。如果设置了`wsdl`参数，则提供 WSDL 而不是尝试处理请求。在这个例子中，我们使用一个简单的 API 密钥来验证请求。然后创建一个 SOAP 服务器实例，分配一个 API 类的实例，并运行`handle()`：

```php
<?php
define('DB_CONFIG_FILE', '/../config/db.config.php');
define('WSDL_FILENAME', __DIR__ . '/chap_07_wsdl.xml');

if (isset($_GET['wsdl'])) {
    readfile(WSDL_FILENAME);
    exit;
}
$apiKey = include __DIR__ . '/api_key.php';
require __DIR__ . '/../Application/Web/Soap/ProspectsApi.php';
require __DIR__ . '/../Application/Database/Connection.php';
use Application\Database\Connection;
use Application\Web\Soap\ProspectsApi;
$connection = new Application\Database\Connection(
  include __DIR__ . DB_CONFIG_FILE);
$api = new Application\Web\Soap\ProspectsApi(
  $connection->pdo, [$apiKey]);
$server = new SoapServer(WSDL_FILENAME);
$server->setObject($api);
echo $server->handle();
```

### 注意

根据您的`php.ini`文件的设置，您可能需要禁用 WSDL 缓存，方法如下：

```php
ini_set('soap.wsdl_cache_enabled', 0);
```

如果您在处理传入的`POST`数据时遇到问题，可以按照以下方式调整此参数：

```php
ini_set('always_populate_raw_post_data', -1);
```

## 它是如何工作的...

您可以通过首先创建目标 API 类，然后生成 WSDL 来轻松测试此示例。然后，您可以使用内置的 PHP Web 服务器来提供 SOAP 服务，命令如下：

```php
**php -S localhost:8080 chap_07_simple_soap_server.php** 

```

然后，您可以使用前面讨论的 SOAP 客户端来调用测试 SOAP 服务：

```php
<?php
define('WSDL_URL', 'http://localhost:8080?wsdl=1');
$clientKey = include __DIR__ . '/api_key.php';
try {
  $client = new SoapClient(WSDL_URL);
  $response = [];
  $email = some_email_generated_by_test;
  $email = 'test5393@unlikelysource.com';
  echo "\nGet Prospect Info for Email: " . $email . "\n";
  $request = ['token' => $clientKey, 'email' => $email];
  $result = $client->get($request,$response);
  var_dump($result);

} catch (SoapFault $e) {
  echo 'ERROR' . PHP_EOL;
  echo $e->getMessage() . PHP_EOL;
} catch (Throwable $e) {
  echo 'ERROR' . PHP_EOL;
  echo $e->getMessage() . PHP_EOL;
} finally {
  echo $client->__getLastResponse() . PHP_EOL;
}
```

以下是电子邮件地址`test5393@unlikelysource.com`的输出：

![它是如何工作的...](img/B05314_07_06.jpg)

## 参见

简单地在谷歌上搜索 PHP 的 WSDL 生成器就会得到大约十几个结果。用于生成`ProspectsApi`类的 WSDL 的生成器基于[`code.google.com/archive/p/php-wsdl-creator/`](https://code.google.com/archive/p/php-wsdl-creator/)。有关`phpDocumentor`的更多信息，请参阅[`www.phpdoc.org/`](https://www.phpdoc.org/)页面。
