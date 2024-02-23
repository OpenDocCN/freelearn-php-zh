# 构建服务

现代应用程序大量使用**HTTP**（**超文本传输协议**）。这种无状态的应用层协议允许我们在分布式系统之间交换消息。消息交换过程可以通过客户端-服务器计算模型观察到，因为它以请求-响应类型的消息形式发生。这使我们能够轻松地编写一个服务，或者更具体地说，一个 Web 服务，触发服务器上的各种操作并将反馈数据返回给客户端。

在本章中，我们将通过以下部分更仔细地研究这种客户端-服务器关系：

+   理解客户端-服务器关系

+   使用 SOAP 进行工作：

+   XML 扩展

+   创建服务器

+   创建 WSDL 文件

+   创建客户端

+   使用 REST 进行工作：

+   JSON 扩展

+   创建服务器

+   创建客户端

+   使用 Apache Thrift（RPC）进行工作：

+   安装 Apache Thrift

+   定义服务

+   创建服务器

+   创建客户端

+   理解微服务

# 理解客户端-服务器关系

为了更容易地可视化客户端-服务器关系和请求-响应类型的消息传递，我们可以将一个移动货币应用程序视为客户端，而一些远程网站，比如`http://api.fixer.io/`，作为服务器。服务器公开一个或多个 URL 端点，允许通信交换，比如`http://api.fixer.io/latest?symbols=USD,GBP`。移动应用程序可以轻松发出 HTTP `GET http://api.fixer.io/latest?symbols=GBP,HRK,USD`请求，然后得到如下响应：

```php
{
 "base": "EUR",
 "date": "2017-03-10",
 "rates": {
 "GBP": 0.8725,
 "HRK": 7.419,
 "USD": 1.0606
  }
}

```

HTTP 的`GET`关键字用于表示我们要在通过 URL 联系的远程（服务器）系统上执行的操作类型。响应包含 JSON 格式的数据，我们的移动货币应用程序可以轻松解析和使用。这个特定的消息交换示例是我们所谓的**表述状态转移**（**REST**）或 RESTful 服务。

REST 服务本身不是一种协议；它是建立在 HTTP 无状态协议和标准操作（GET、POST、PUT、DELETE 等）之上的一种架构风格。在这个简单的例子中展示的只是冰山一角，我们将在*使用 REST*部分后面看到更多。

还有其他形式的服务，超越了仅仅是一种架构风格，比如 SOAP 服务和 Apache Thrift 服务。虽然它们有自己的协议集，但它们也可以与 HTTP 很好地配合。

# 使用 SOAP 进行工作

**SOAP**（**简单对象访问协议**）是一种基于 XML 的消息交换协议，依赖于应用层协议（如 HTTP）进行消息协商和传输。**万维网联盟**（**W3C**）维护 SOAP 规范。

SOAP 规范文档可在[`www.w3.org/TR/soap/`](https://www.w3.org/TR/soap/)找到。

SOAP 消息是由`Envelope`、`Header`、`Body`和`Fault`元素组成的 XML 文档：

```php
<?xml version="1.0" ?> <env:Envelope>
<env:Header>
<!-- ... -->
  </env:Header>
<env:Body>
<!-- ... -->
  <env:Fault>
<!-- ... -->
  </env:Fault>
</env:Body>
</env:Envelope>

```

`Envelope`是每个 SOAP 请求的必需元素，因为它包含整个 SOAP 消息。同样，`Body`元素也是必需的，因为它包含请求和响应信息。另一方面，`Header`和`Fault`是可选元素。仅使用基于 XML 的请求-响应消息，我们可以通过 HTTP 建立客户端-服务器通信。虽然交换 XML 消息看起来很简单，但当一个人必须处理大量的方法调用和数据类型时，这可能会变得繁琐。

这就是 WSDL 发挥作用的地方。WSDL 是一种接口定义语言，用于定义 Web 服务的数据类型和操作。W3C 维护 WSDL 规范。

WSDL 规范文档可在[`www.w3.org/TR/wsdl`](https://www.w3.org/TR/wsdl)找到。

根据以下部分示例，一共使用了六个主要元素来描述服务：

```php
<?xml version="1.0" ?> <definitions>
<types>
<!-- ... -->
  </types>
<message>
<!-- ... -->
  </message>
<portType>
<!-- ... -->
  </portType>
<binding>
<!-- ... -->
  </binding>
<port>
<!-- ... -->
  </port>
<service>
<!-- ... -->
  </service>
</definitions>

```

虽然 WSDL 对于我们的服务的运行并不是必需的，但对于使用我们的 SOAP 服务的客户端来说，它肯定很方便。遗憾的是，PHP 缺乏基于 SOAP 服务使用的 PHP 类轻松生成 WSDL 文件的官方工具。这使得 PHP 开发人员手动编写 WSDL 文件变得繁琐和耗时，这就是为什么一些开发人员倾向于完全忽略 WSDL。

暂时将 WSDL 文件生成放在一边，可以说 SOAP 服务中唯一真正具有挑战性的部分是编写和读取 XML 消息。这就是 PHP 扩展派上用场的地方。

# XML 扩展

在 PHP 中有几种读取和写入 XML 文档的方法，包括正则表达式和专门的类和方法。正则表达式方法容易出错，特别是对于复杂的 XML 文档，因此建议使用扩展。PHP 为此提供了几种扩展，最常见的是以下几种：

+   **XMLWriter**：这允许我们生成 XML 数据的流或文件

+   **XMLReader**：这允许读取 XML 数据

+   **SimpleXML**：这将 XML 转换为对象，并允许使用常规属性选择器和数组迭代器处理对象

+   **DOM**：这允许我们通过 DOM API 操作 XML 文档

处理 XML 文档的基础是正确读取和写入其元素和属性。让我们假设以下的`simple.xml`文档：

```php
<?xml version="1.0" encoding="UTF-8"?> <customer>
 <name type="string"><![CDATA[John]]></name>
 <age type="integer">34</age> 
 <addresses>
 <address><![CDATA[The Address #1]]></address>
 </addresses>
</customer>

```

使用`XMLWriter`，我们可以通过运行以下代码创建相同的文档：

```php
<?php $xml  = new XMLWriter(); $xml->openMemory(); $xml->setIndent(true); // optional formatting   $xml->startDocument('1.0', 'UTF-8'); $xml->startElement('customer');   $xml->startElement('name'); $xml->writeAttribute('type', 'string'); $xml->writeCData('John'); $xml->endElement(); // </name> $xml->startElement('age'); $xml->writeAttribute('type', 'integer'); $xml->writeRaw(34); $xml->endElement(); // </age> $xml->startElement('addresses'); $xml->startElement('address'); $xml->writeCData('The Address #1'); $xml->endElement(); // </address> $xml->endElement(); // </addresses>   $xml->endElement(); // </customer>   $document = $xml->outputMemory();

```

我们可以看到，使用`XMLWriter`写下必要的 XML 是一个相对简单的操作。`XMLWriter`扩展使我们的代码一开始有点难以阅读。所有这些`startElement()`和`endElement()`方法使得弄清楚 XML 中的每个元素有点乏味。需要一点时间来适应它。但是，它确实允许我们轻松生成简单的 XML 文档。使用`XMLReader`，我们现在可以根据给定 XML 文档中的数据输出`Customer John, at age 34, living at The Address #1`字符串，使用以下代码块：

```php
<?php $xml = new XMLReader(); $xml->open(__DIR__ . '/simple.xml');   $name = ''; $age = ''; $address = '';   while ($xml->read()) {   if ($xml->name == 'name') {
  $name = $xml->readString();
  $xml->next();
 } elseif ($xml->name == 'age') {
  $age = $xml->readString();
  $xml->next();
 } elseif ($xml->name == 'address') {
  $address = $xml->readString();
  $xml->next();
 } }   echo sprintf(
  'Customer %s, at age %s, living at %s',
  $name, $age, $address );

```

虽然代码本身看起来非常简单，但`while`循环揭示了`XMLReader`的一个有趣的特性。`XMLReader`从上到下读取 XML 文档。虽然这种方法对于以流为基础高效解析大型和复杂的 XML 文档是一个很好的选择，但对于更简单的 XML 文档来说似乎有点过度。

让我们看看`SimpleXML`如何处理写入相同的`simple.xml`文件。以下代码生成的 XML 内容几乎与`XMLWriter`相同：

```php
<?php   $document = new SimpleXMLElement(
  '<?xml version="1.0" encoding="UTF-8"?><customer></customer>' );   $name = $document->addChild('name', 'John'); $age = $document->addChild('age', 34); $addresses = $document->addChild('addresses'); $address = $addresses->addChild('address', 'The Address #1'); echo $document->asXML();

```

这里的区别在于我们无法将`<![CDATA[...]]>`直接传递给我们的元素。有一些使用`dom_import_simplexml()`函数的变通方法，但那是来自`DOM`扩展的函数。并不是说这有什么不好，但让我们保持我们的示例清晰分离。现在我们知道我们可以使用`SimpleXML`编写 XML 文档，让我们看看如何从中读取。使用`SimpleXML`，我们现在可以使用以下代码输出相同的`Customer John, at age 34, living at The Address #1`字符串：

```php
<?php   $document = new SimpleXMLElement(__DIR__ . '/simple.xml', null, true);   $name = (string)$document->name; $age = (string)$document->age; $address = (string)$document->addresses[0]->address; echo sprintf(
  'Customer %s, at age %s, living at %s',
  $name, $age, $address );

```

使用`SimpleXML`读取 XML 的过程似乎比使用`XMLReader`要短一些，尽管这些示例都没有任何错误处理。

让我们看看使用`DOMDocument`类来写下一个 XML 文档：

```php
<?php $document = new DOMDocument('1.0', 'UTF-8'); $document->formatOutput = true; // optional $customer = $document->createElement('customer'); $customer = $document->appendChild($customer); $name = $document->createElement('name'); $name = $customer->appendChild($name); $nameTypeAttr = $document->createAttribute('type'); $nameTypeAttr->value = 'string'; $name->appendChild($nameTypeAttr); $name->appendChild($document->createCDATASection('John')); $age = $document->createElement('age'); $age = $customer->appendChild($age); $ageTypeAttr = $document->createAttribute('type'); $ageTypeAttr->value = 'integer'; $age->appendChild($ageTypeAttr); $age->appendChild($document->createTextNode(34));   $addresses = $document->createElement('addresses'); $addresses = $customer->appendChild($addresses); $address = $document->createElement('address'); $address = $addresses->appendChild($address); $address->appendChild($document->createCDATASection('The Address #1')); echo $document->saveXML();

```

最后，让我们看看`DOMDocument`如何处理读取 XML 文档：

```php
<?php   $document = new DOMDocument(); $document->load(__DIR__ . '/simple.xml');   $name = $document->getElementsByTagName('name')[0]->nodeValue; $age = $document->getElementsByTagName('age')[0]->nodeValue; $address = $document->getElementsByTagName('address')[0]->nodeValue; echo sprintf(
  'Customer %s, at age %s, living at %s',
  $name, $age, $address );

```

`DOM`和`SimpleXMLElement`扩展使从 XML 文档中读取值变得非常容易，只要我们对其结构的完整性有信心。在处理 XML 文档时，我们应该根据诸如文档大小之类的因素评估我们的用例。虽然`XMLReader`和`XMLWriter`类在处理时更冗长，但在正确使用时它们往往更高效。

现在我们已经对在 PHP 中处理 XML 文档有了基本的了解，让我们创建我们的第一个 SOAP 服务器。

# 创建服务器

PHP `soap`扩展提供了`SoapClient`和`SoapServer`类。我们可以使用`SoapServer`类来设置具有或不具有 WSDL 服务描述文件的 SOAP 服务服务器。

在没有 WSDL（非 WSDL 模式）的情况下使用`SoapClient`和`SoapServer`使用一个常见的交换格式，这消除了对 WSDL 文件的需求。

在继续之前，我们应该确保已安装了`soap`扩展。我们可以通过观察`php -m`控制台命令的输出或查看`phpinfo()`函数的输出来实现：

![](img/00c54baf-529e-4228-8d4d-cf18b4a4aefe.png)

有了可用和加载的 soap 扩展，我们可以按照以下结构准备我们的`soap-service`项目目录：

![](img/c2b3e4f3-786f-49c3-9a32-a6643d21490a.png)

继续向前，我们将假设 Web 服务器配置为从`soap-service/server`目录提供内容到[`soap-service.server`](http://soap-service.server)请求，并从`soap-service/client`目录提供内容到[`soap-service.client`](http://soap-service.client)请求。

让我们创建一个小的 SOAP 服务，其中包含两个不同的类，每个类都有相同的`welcome()`方法。我们可以首先创建`soap-service/server/services/Foggyline/Customer.php`文件，内容如下：

```php
<?php namespace Foggyline;   class Customer {
  /**
 * Says "Welcome customer..." * @param $name
 * @return string
 */  function welcome($name)
 {  return 'Welcome customer: ' . $name;
 } } 

```

现在，让我们创建`soap-service/server/services/Foggyline/User.php`文件，内容如下：

```php
<?php namespace Foggyline;   class User {
  /**
 * Says "Welcome user..." * @param $name
 * @return string
 */  function welcome($name)
 {  return 'Welcome user: ' . $name;
 } } 

```

有了这两个类，让我们创建一个代理类来包装它们。我们通过创建`soap-service/server/ServiceProxy.php`文件来实现：

```php
<?php   require_once __DIR__ . '/services/Foggyline/Customer.php'; require_once __DIR__ . '/services/Foggyline/User.php'; class ServiceProxy {
  private $customerService;
  private $userService;    public function __construct()
 {  $this->customerService = new Foggyline\Customer();
  $this->userService = new Foggyline\User();
 }    /**
 * Says "Welcome customer..." * @soap
  * @param $name
 * @return string
 */  public function customerWelcome($name)
 {  return $this->customerService->welcome($name);
 }    /**
 * Says "Welcome user..." * @soap
  * @param $name
 * @return string
 */  public function userWelcome($name)
 {  return $this->userService->welcome($name);
 } }

```

现在我们有了代理类，我们可以创建实际的`SoapServer`实例。我们通过创建`soap-service/server/index.php`文件来实现：

```php
<?php require_once __DIR__ . '/ServiceProxy.php'; $options = [   'uri' => 'http://soap-service.server/index.php' ]; $server = new SoapServer(null, $options);   $server->setClass('ServiceProxy');   $server->handle(); 

```

在这里，我们实例化`SoapServer`实例，将 null 传递给`$wsdl`参数，并在`$options`参数下只传递一个`'uri'`选项。URI 必须在非 wsdl 模式下指定。然后我们使用`setClass()`实例方法来设置处理传入 SOAP 请求的类。不幸的是，我们不能传递一个类数组或多次调用`setClass()`方法一次添加多个不同的处理类，这就是为什么我们创建了`ServiceProxy`类来包装`Customer`和`User`类。最后，我们调用了`$server`实例的`handle()`方法，处理 SOAP 请求。此时，我们的 SOAP 服务服务器应该是完全可操作的。

# 创建 WSDL 文件

然而，在转向客户端之前，让我们快速看一下 WSDL。`ServiceProxy`类方法上使用的`@soap`标签与`SoapServer`的功能无关。我们之所以使用它，仅仅是因为 php2wsdl 库使我们能够根据提供的类自动生成 WSDL 文件。php2wsdl 库作为一个 composer 包提供，这意味着我们可以通过在`soap-service/server`目录中简单运行以下命令来安装它：

```php
composer require php2wsdl/php2wsdl

```

安装后，我们可以创建`soap-service\server\wsdl-auto-gen.php`文件，内容如下：

```php
<?php require_once __DIR__ . '/vendor/autoload.php'; require_once __DIR__ . '/ServiceProxy.php';   $class = 'ServiceProxy'; $serviceURI = 'http://soap-service.server/index.php';   $wsdlGenerator = new PHP2WSDL\PHPClass2WSDL($class, $serviceURI); $wsdlGenerator->generateWSDL(true); file_put_contents(__DIR__ . '/wsdl.xml', $wsdlGenerator->dump());

```

一旦我们在控制台或浏览器中执行`wsdl-auto-gen.php`，它将生成`soap-service/server/wsdl.xml`文件，内容如下：

```php
<?xml version="1.0"?> <definitions xmlns="http://schemas.xmlsoap.org/wsdl/" xmlns:tns="http://soap-service.server/index.php" xmlns:soap="http://schemas.xmlsoap.org/wsdl/soap/" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:soap-enc="http://schemas.xmlsoap.org/soap/encoding/" xmlns:wsdl="http://schemas.xmlsoap.org/wsdl/" name="ServiceProxy" targetNamespace="http://soap-service.server/index.php">
<types>
<xsd:schema targetNamespace="http://soap-service.server/index.php">
<xsd:import namespace="http://schemas.xmlsoap.org/soap/encoding/"/>
</xsd:schema>
</types>
<portType name="ServiceProxyPort">
 <operation name="customerWelcome">
 <documentation>Says "Welcome customer..."</documentation>
 <input message="tns:customerWelcomeIn"/>
 <output message="tns:customerWelcomeOut"/>
 </operation>
 <operation name="userWelcome">
 <documentation>Says "Welcome user..."</documentation>
 <input message="tns:userWelcomeIn"/>
 <output message="tns:userWelcomeOut"/>
</operation>
</portType>
<binding name="ServiceProxyBinding" type="tns:ServiceProxyPort">
<soap:binding style="rpc" transport="http://schemas.xmlsoap.org/soap/http"/>
<operation name="customerWelcome">
<soap:operation soapAction="http://soap-service.server/index.php#customerWelcome"/>
<input>
<soap:body use="encoded" encodingStyle="http://schemas.xmlsoap.org/soap/encoding/" namespace="http://soap-service.server/index.php"/>
</input>
<output>
<soap:body use="encoded" encodingStyle="http://schemas.xmlsoap.org/soap/encoding/" namespace="http://soap-service.server/index.php"/>
</output>
</operation>
<operation name="userWelcome">
<soap:operation soapAction="http://soap-service.server/index.php#userWelcome"/>
<input>
<soap:body use="encoded" encodingStyle="http://schemas.xmlsoap.org/soap/encoding/" namespace="http://soap-service.server/index.php"/>
</input>
<output>
<soap:body use="encoded" encodingStyle="http://schemas.xmlsoap.org/soap/encoding/" namespace="http://soap-service.server/index.php"/>
</output>
</operation>
</binding>
<service name="ServiceProxyService">
<port name="ServiceProxyPort" binding="tns:ServiceProxyBinding">
 <soap:address location="http://soap-service.server/index.php"/>
</port>
</service>
<message name="customerWelcomeIn">
 <part name="name" type="xsd:anyType"/>
</message>
<message name="customerWelcomeOut">
 <part name="return" type="xsd:string"/>
</message>
<message name="userWelcomeIn">
  <part name="name" type="xsd:anyType"/>
</message>
<message name="userWelcomeOut">
 <part name="return" type="xsd:string"/>
</message>
</definitions>

```

这是一个相当长的文件需要手动编写。好处是一旦设置了 WSDL 文件，各种第三方工具和其他语言库就可以轻松消费我们的服务。例如，这是 Chrome 浏览器的 Wizdler 扩展的屏幕截图，解释了 WSDL 文件的内容：

![](img/eb1e49d7-b0ef-4cba-9ae6-26abab4a028d.png)

有了 WSDL，我们现在可以轻松修改`soap-service/server/index.php`文件如下：

```php
// NON-WSDL MODE: $server = new SoapServer(null, $options);

// WSDL MODE: $server = new SoapServer('http://soap-service.server/wsdl.xml'); $server = new SoapServer('http://soap-service.server/wsdl.xml');

```

现在我们已经解决了 SOAP 服务器的问题，让我们创建一个客户端。

# 创建客户端

在 PHP 中创建 SOAP 客户端是一个相对简单的任务，当我们使用`SoapClient`类时。让我们创建`soap-service/client/index.php`文件，内容如下：

```php
<?php $options = [
  'location' => 'http://soap-service.server/index.php',
  'uri' => 'http://soap-service.server/index.php',
  'trace ' => true, ];   // NON-WSDL MODE: $client = new SoapClient($wsdl = null, $options); // WSDL MODE: $client = new SoapClient('http://soap-service.server/wsdl.xml', $options);   $client = new SoapClient('http://soap-service.server/wsdl.xml', $options);   echo $client->customerWelcome('John'); echo $client->userWelcome('Mariya');

```

执行客户端代码应该产生以下输出：

![](img/f2a8a419-c30a-48d6-bf7f-00e5fab1e73d.png)

当发出 SOAP 请求时，底层发生了什么可以通过 Wireshark 等网络工具观察到：

![](img/35adede7-01e3-4081-8bb7-24e7b29c231a.png)

这向我们展示了单个 SOAP 请求的确切内容，例如`$client->customerWelcome('John')`的请求：

```php
POST /index.php HTTP/1.1
Host: soap-service.server
Connection: Keep-Alive
User-Agent: PHP-SOAP/7.0.10
Content-Type: text/xml; charset=utf-8
SOAPAction: "http://soap-service.server/index.php#customerWelcome"
Content-Length: 525

<?xml version="1.0" encoding="UTF-8"?>
<SOAP-ENV:Envelope xmlns:SOAP-ENV="http://schemas.xmlsoap.org/soap/envelope/"

 xmlns:SOAP-ENC="http://schemas.xmlsoap.org/soap/encoding/"
 SOAP-ENV:encodingStyle="http://schemas.xmlsoap.org/soap/encoding/">
 <SOAP-ENV:Body>
 <ns1:customerWelcome>
 <name xsi:type="xsd:string">John</name>
 </ns1:customerWelcome>
 </SOAP-ENV:Body>
</SOAP-ENV:Envelope>

```

了解 SOAP 请求的结构和内容使得甚至可以使用`cURL`函数来处理请求-响应通信，尽管这比处理`SoapClient`和`SoapServer`类要困难得多且容易出错。

在本节中，我们已经触及了一些 SOAP 服务的关键点。虽然关于 SOAP 规范还有很多要说的，但这里呈现的示例是编写 SOAP 服务的一个不错的起点。

一个更简单的 Web 服务变体将是 REST。

# 使用 REST

与 SOAP 不同，REST 是一种架构风格。它没有自己的协议或标准。它依赖于 URL 和 HTTP 动词，如 POST、GET、PUT 和 DELETE，以建立消息交换过程。缺乏标准使得它在一定程度上具有挑战性，因为各种 REST 服务实现可能以不同的方式向客户端提供消费服务的途径。在来回搬运数据时，我们可以自由选择 JSON、XML 或其他任何我们喜欢的格式。JSON 的简单性和轻量性使其成为许多用户和框架中的热门选择。

宽泛地说，浏览器中打开网页的行为可以被解释为一个 REST 调用，其中浏览器充当客户端，服务器充当 REST 服务。与可能涉及 cookie 和会话的浏览器页面不同，REST 依赖于无状态操作。

继续向前，我们将假设我们的 Web 服务器配置为为[`rest-service.server`](http://rest-service.server)请求提供`rest-service/server`目录的内容，并为[`rest-service.client`](http://rest-service.client)请求提供`rest-service/client`目录的内容。

# JSON 扩展

多年来，JSON 数据格式已经成为 REST 的默认数据交换格式。JSON 的简单性使其在 PHP 开发人员中相当受欢迎。PHP 语言提供了`json_encode()`和`json_decode()`函数。使用这些函数，我们可以轻松地对 PHP 数组和对象进行编码，以及解码各种 JSON 结构。

以下示例演示了使用`json_encode()`函数的简单性：

```php
<?php   class User {
  public $name;
  public $age;
  public $salary; } $user = new User(); $user->name = 'John'; $user->age = 34; $user->salary = 4200.50;   echo json_encode($user); // {"name":"John","age":34,"salary":4200.5}   $employees = ['John', 'Mariya', 'Sarah', 'Marc'];   echo json_encode($employees); // ["John","Mariya","Sarah","Marc"]

```

以下示例演示了使用`json_decode()`函数的简单性：

```php
<?php   $user = json_decode('{"name":"John","age":34,"salary":4200.5}'); print_r($user); //    stdClass Object //    ( //        [name] => John //        [age] => 34 //        [salary] => 4200.5 //    )

```

这就是限制开始发挥作用的地方。请注意，JSON 对象在 PHP 中被转换为`stdClass`类型对象。没有直接的方法将其转换为`User`类型的对象。当然，如果需要，我们可以编写自定义功能，尝试将`stdClass`对象转换为`User`的实例。

# 创建服务器

简而言之，REST 服务器根据给定的 URL 和 HTTP 动词发送 HTTP 响应。牢记这一点，让我们从添加到`rest-service/server/customer/index.php`文件的以下代码块开始：

```php
<?php   if ('POST' == $_SERVER['REQUEST_METHOD']) {
  header('Content-type: application/json');
  echo json_encode(['data' => 'Triggered customer POST!']); }   if ('GET' == $_SERVER['REQUEST_METHOD']) {
  header('Content-type: application/json');
  echo json_encode(['data' => 'Triggered customer GET!']); }   if ('PUT' == $_SERVER['REQUEST_METHOD']) {
  header('Content-type: application/json');
  echo json_encode(['data' => 'Triggered customer PUT!']); }   if ('DELETE' == $_SERVER['REQUEST_METHOD']) {
  header('Content-type: application/json');
  echo json_encode(['data' => 'Triggered customer DELETE!']); }

```

看起来有趣的是，这里已经是一个简单的 REST 服务示例--一个处理单个资源的四种不同操作。使用诸如 Postman 之类的工具，我们可以触发对[`rest-service.server/customer/index.php`](http://rest-service.server/customer/index.php)资源的`DELETE`操作

![](img/7536b609-156c-4f4e-8799-304d281eabe8.png)

显然，这种简化的实现没有处理 REST 服务中通常会遇到的任何事情，比如版本控制、规范化、验证、跨域资源共享（CORS）、身份验证等。从头开始实现所有这些 REST 功能是一项耗时的任务，这就是为什么我们可能需要看看现有框架提供的解决方案。

Silex 微框架是快速开始 REST 服务的一个不错的解决方案。我们可以通过在`rest-service/server`目录中的控制台上运行以下命令来简单地将 Silex 添加到我们的项目中：

```php
composer require silex/silex "~2.0"

```

一旦安装好了，我们可以将以下代码转储到`rest-service/server/index.php`文件中：

```php
<?php require_once __DIR__ . '/vendor/autoload.php'; use Silex\Application; use Symfony\Component\HttpFoundation\Request; use Symfony\Component\HttpFoundation\Response;   $app = new Silex\Application();   // The "before" middleware, convenient for auth and request data check $app->before(function (Request $request, Application $app) {
  // Some auth token control
  if (!$request->headers->get('X-AUTH-TOKEN')) {
  // todo: Implement
  }
  // JSON content type control
  if ($request->headers->get('Content-Type') != 'application/json') {
  // todo: Implement
  } });   // The "error" middleware, convenient for service wide error handling $app->error(function (\Exception $e, Request $request, $code) {
  // todo: Implement });   // The "OPTIONS" route, set to trigger for any URL $app->options('{url}', function ($url) use ($app) {
  return new Response('', 204, ['Allow' => 'POST, GET, PUT, DELETE, OPTIONS']); })->assert('url', '.+');   // The "after" middleware, convenient for CORS control $app->after(function (Request $request, Response $response) {
  $response->headers->set('Access-Control-Allow-Headers', 'origin, content-type, accept, X-AUTH-TOKEN');
  $response->headers->set('Access-Control-Allow-Origin', '*');
  $response->headers->set('Access-Control-Allow-Methods', 'POST, GET, PUT, DELETE'); }); // The "POST /user/welcome" REST service endpoint $app->post('/user/welcome', function (Request $request, Application $app) {
  $data = json_decode($request->getContent(), true);
  return $app->json(['data' => 'Welcome ' . $data['name']]); })->bind('user_welcome');   $app->run();

```

这也是一个相对简单的 REST 服务示例，但比我们最初的示例做得更多。在这种情况下，Silex 框架引入了几个关键概念，我们可以利用这些概念来构建我们的 REST 服务器。`before`、`after`和`error`中间件使我们能够钩入请求处理过程的三个不同阶段。使用`before`中间件，我们可以注入身份验证代码，以及对传入数据的有效性进行各种检查。REST 服务通常围绕令牌构建其身份验证，然后将其传递给各个请求。一般的想法是有一个端点，比如`POST user/login`，用户使用用户名和密码登录，然后获得一个用于其余 REST 服务调用的身份验证令牌。然后，这个令牌通常作为请求头的一部分传递。现在，每当用户尝试访问受保护的资源时，都会从头部提取一个令牌，并在数据库（或任何其他可能存储它的地方）中查找令牌背后的用户。然后系统要么允许用户继续原始请求，要么将其阻止。这就是中间件派上用场的地方。

Web 服务身份验证本身就是一个庞大的话题，本书不会涉及。OAuth 是授权的行业标准协议，通常与 REST 风格的服务一起使用。有关 OAuth 的更多信息，请访问[`oauth.net`](https://oauth.net)。

我们包装响应的方式完全取决于我们自己。与 SOAP 不同，没有长期建立的标准来定义 REST 服务响应的数据结构。然而，在过去几年中，有几个倡议试图解决这一挑战。

JSON API 试图规范使用交换 JSON 数据的客户端-服务器接口；请访问[`jsonapi.org/format/`](http://jsonapi.org/format/)获取更多信息。

为了使服务器正常工作，我们还需要添加`rest-service\server\.htaccess`文件，内容如下：

```php
<IfModule mod_rewrite.c>
Options -MultiViews
  RewriteEngine On
  RewriteCond %{REQUEST_FILENAME} !-d
  RewriteCond %{REQUEST_FILENAME} !-f
  RewriteRule ^ index.php [QSA,L] </IfModule>

```

Silex 方便地支持几个关键的 HTTP 动词（GET、POST、PUT、DELETE、PATCH 和 OPTIONS），我们可以很容易地以*资源路径+回调函数*的语法实现逻辑：

```php
$app->get('/resource/path', function () { /* todo: logic */ }); $app->post('/resource/path', function () { /* todo: logic */ }); $app->put('/resource/path', function () { /* todo: logic */ }); $app->delete('/resource/path', function () { /* todo: logic */ }); $app->patch('/resource/path', function () { /* todo: logic */ }); $app->options('/resource/path', function () { /* todo: logic */ });

```

这使得快速起草 REST 服务变得容易，只需几行代码。我们的服务器示例在服务器安全方面几乎没有做任何事情。它的目的只是强调构建 REST 服务时中间件的有用性。安全方面，如身份验证、授权、CORS、HTTPS 等都应该引起极大的重视。

框架如[`silex.sensiolabs.org`](http://silex.sensiolabs.org)和[`apigility.org`](https://apigility.org/)提供了一个很好的解决方案，可以编写高质量、功能丰富的 REST 服务。

# 创建客户端

鉴于 REST 服务依赖于 HTTP，可以肯定地假设使用 PHP CURL 编写客户端应该是一个相当简单的过程。让我们创建一个`rest-service/client/index.php`文件，内容如下：

```php
<?php $ch = curl_init();   $headers = [
  'Content-Type: application/json',
  'X-AUTH-TOKEN: some-auth-token-here' ]; curl_setopt($ch, CURLOPT_URL, 'http://rest-service.server/user/welcome'); curl_setopt($ch, CURLOPT_POST, true); curl_setopt($ch, CURLOPT_POSTFIELDS, json_encode(['name' => 'John'])); curl_setopt($ch, CURLOPT_HTTPHEADER, $headers); curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);   $result = curl_exec($ch);   curl_close($ch);   echo $result;

```

Wireshark 网络工具告诉我们，这段代码生成了以下 HTTP 请求到 REST 服务：

```php
POST /user/welcome HTTP/1.1
Host: rest-service.server
Accept: */*
Content-Type: application/json
X-AUTH-TOKEN: some-auth-token-here
Content-Length: 15

{"name":"John"}

```

虽然 CURL 方法运行良好，但很快就会变得繁琐且容易出错。这意味着必须处理各种类型的错误响应、SSL 证书等挑战。更优雅的解决方案是使用 HTTP 客户端库，比如 Guzzle。

Guzzle 是一个使用 PHP 编写的 MIT 许可的 HTTP 客户端。可以通过运行`composer require guzzlehttp/guzzle`命令轻松安装它。

我们的 REST 服务很可能会更频繁地受到非 PHP 客户端的联系，而不是 PHP 客户端。考虑到这一点，让我们看看一个简单的 HTML/jQuery 客户端如何与我们的 REST 服务进行通信。我们通过将以下代码添加到`rest-service/client/index.html`来实现：

```php
<!DOCTYPE html>
<html lang="en">
 <head>
 <meta charset="UTF-8">
 <title>Client App</title>
 <script  src="https://code.jquery.com/jquery-3.1.1.min.js"
  integrity="sha256-hVVnYaiADRTO2PzUGmuLJr8BLUSjGIZsDYGmIJLv2b8="
  crossorigin="anonymous"></script>
 </head>
<body>
 <script>
    jQuery.ajax({
 method: 'POST',
 url: 'http://rest-service.server/user/welcome',
 headers: {'X-AUTH-TOKEN': 'some-auth-token-here'},
 data: JSON.stringify({name: 'John'}),
 dataType: 'json',
 contentType: 'application/json',
 success: function (response) {
 console.log(response.data);
      }
    });
 </script>
 </body>
</html>

```

jQuery 的`ajax()`方法充当 HTTP 客户端。通过传递正确的参数值，它能够成功地与 REST 服务建立请求-响应通信。

在本节中，我们已经涉及了一些 REST 服务的关键点。虽然我们只是浅尝辄止 REST 架构的整体，但这里呈现的示例应该足以让我们开始。JSON 和 HTTP 的易于实现和简单性使得 REST 对于现代应用程序来说是一个相当吸引人的选择。

# 使用 Apache Thrift（RPC）

Apache Thrift 是一个构建可扩展跨语言服务的开源框架。它最初由 Facebook 开发，然后于 2008 年 5 月左右进入 Apache 孵化器。简单性、透明性、一致性和性能是该框架背后的四个关键价值观。

与 REST 和 SOAP 类型的服务不同，Thrift 服务使用二进制形式的通信。幸运的是，Thrift 提供了一个代码生成引擎来帮助我们入门。代码生成引擎可以从任何**接口定义语言**（IDL）文件中提取并生成 PHP 或其他语言的绑定。

在我们开始编写第一个服务定义之前，我们需要安装 Apache Thrift。

# 安装 Apache Thrift

Apache Thrift 可以从源文件安装。假设我们有一个全新的 Ubuntu 16.10 安装，我们可以使用以下一组命令启动 Apache Thrift 安装步骤：

```php
sudo apt-get update
sudo apt-get -y install php automake bison flex g++ git libboost-all-dev libevent-dev libssl-dev libtool make pkg-config

```

这两个命令应该为我们提供编译 Apache Thrift 源文件所需的工具。完成后，我们可以在我们的机器上拉取实际的源文件：

```php
wget http://apache.mirror.anlx.net/thrift/0.10.0/thrift-0.10.0.tar.gz
tar -xvf thrift-0.10.0.tar.gz
cd thrift-0.10.0/

```

解压源文件后，我们可以触发`configure`和`make`命令，如下所示：

```php
./configure
make
make install

```

最后，我们需要确保我们的`LD_LIBRARY_PATH`路径上有`/usr/local/lib/`目录：

```php
echo "export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/lib/" >> ~/.bashrc

```

现在我们应该退出 shell，然后重新登录。使用以下命令，我们确认安装了 Apache Thrift：

```php
thrift -version

```

这应该给我们以下输出：

![](img/bae82a87-22da-447b-b7a8-d6a90ab6c31e.png)

安装了`thrift`工具并且可以通过控制台使用后，我们可以准备我们的`thrift-service`项目：

```php
mkdir thrift-service
cd thrift-service/
mkdir client
mkdir server
mkdir vendor
cd vendor
git clone https://github.com/apache/thrift.git

```

继续前进，我们将假设 Web 服务器配置为将`thrift-service/client`目录的内容提供给[`thrift-service.client`](http://thrift-service.client)请求，并将`thrift-service/server`目录的内容提供给[`thrift-service.server`](http://thrift-service.server)请求。

# 定义服务

在 PHP 中使用 Apache Thrift 可以通过以下几个步骤描述：

+   通过 IDL 文件定义服务

+   自动生成语言绑定

+   提供已定义接口的 PHP 实现

+   通过服务器公开提供的服务实现

+   通过客户端使用暴露的服务

Thrift 服务以`.thrift`文件的形式开始它们的生命周期，也就是说，由 IDL 描述的文件。

IDL 文件支持定义多种数据类型：

+   `bool`：这是一个布尔值（true 或 false）

+   `byte`：这是一个 8 位有符号整数

+   `i16`：这是一个 16 位有符号整数

+   `i32`：这是一个 32 位有符号整数

+   `i64`：这是一个 64 位有符号整数

+   `double`：这是一个 64 位浮点数

+   `string`：这是一个 UTF-8 编码的文本字符串

+   `二进制`：这是一系列未编码的字节

+   `struct`：这在面向对象编程语言中基本上相当于类，但没有继承

+   容器（`list`，`set`，`map`）：这映射到大多数编程语言中的常见容器类型

为了保持简单，我们将专注于`string`类型的使用。让我们创建我们的第一个 Apache Thrift 服务。我们通过在`thrift-service/`目录中创建一个`Greeting.thrift`文件来实现：

```php
namespace php user

service GreetingService
{
  string hello(1: string name),
  string goodbye()
}

```

我们可以看到 Thrift 文件是一个纯接口--这里没有实现。`namespace php user`语法转换为*当代码生成引擎运行时，在用户命名空间内为生成的 PHP 代码生成 GreetingService*。如果我们在 PHP 之外使用另一种语言，比如 Java，我们可以轻松地添加另一行，说`namespace java customer`。这将在一个命名空间中生成 PHP 绑定，在另一个命名空间中生成 Java 绑定。

我们可以看到`service`关键字被用来指定`GreetingService`接口。在接口内，我们有两个方法定义。`hello(1: string name)`接收一个名字参数，而`goodbye()`不接收任何参数。

有关 IDL 语法的更多详细信息，请参见[`thrift.apache.org/docs/idl`](https://thrift.apache.org/docs/idl)。

有了`Greeting.thrift`文件，我们可以触发代码生成以获得必要的 PHP 绑定。我们可以通过在控制台上执行以下代码来实现：

```php
thrift -r -gen php:server Greeting.thrift

```

此时，我们的文件夹结构应该类似于以下截图：

![](img/1041c9b3-049f-4abd-aefe-340e33ca8799.png)

我们可以看到`thrift`命令在`gen-php/user`目录下为我们生成了两个文件。`GreetingService.php`是一个相当大的文件；几乎有 500 行代码，它定义了与我们的 Thrift 服务一起使用所需的各种辅助函数和结构：

![](img/5f7da8de-b524-41ac-8719-a9f24a34aff5.png)

而`Types.php`文件定义了几种不同的类型供使用：

![](img/40a4fc71-9e31-4be4-b5e7-5b89fc8134ed.png)

所有这些类型都驻留在`thrift-service/vendor/thrift/lib/php/lib/Thrift`中，这就是我们之前执行`git clone https://github.com/apache/thrift.git`命令的原因。到目前为止，我们的`thrift-service/gen-php/user/GreetingService.php`服务在`hello()`和`goodbye()`方法逻辑方面还没有真正做任何事情。

# 创建服务器

`thrift-service/server/`目录是我们将实现项目服务器部分的地方。让我们创建一个单一的`thrift-service/server/index.php`文件，实现`hello()`和`goodbye()`方法，并通过[`thrift-service.server/index.php`](http://thrift-service.server/index.php)将它们暴露给任何可能到来的 thrift 请求：

```php
<?php

require_once __DIR__ . '/../vendor/thrift/lib/php/lib/Thrift/ClassLoader/ThriftClassLoader.php';

use Thrift\ClassLoader\ThriftClassLoader;
use Thrift\Transport\TPhpStream;
use Thrift\Transport\TBufferedTransport;
use Thrift\Protocol\TBinaryProtocol;
use user\GreetingServiceProcessor;
use user\GreetingServiceIf;

$loader = new ThriftClassLoader();
$loader->registerNamespace('Thrift', __DIR__ . '/../vendor/thrift/lib/php/lib');
$loader->registerDefinition('user', __DIR__ . '/../gen-php');
$loader->register();

class GreetingServiceImpl implements GreetingServiceIf
{
  public function hello($name)
  {
    return 'Hello ' . $name . '!';
  }

  public function goodbye()
  {
    return 'Goodbye!';
  }
}

header('Content-Type', 'application/x-thrift');

$handler = new GreetingServiceImpl();
$processor = new GreetingServiceProcessor($handler);
$transport = new TBufferedTransport(new TPhpStream(TPhpStream::MODE_R | TPhpStream::MODE_W));
$protocol = new TBinaryProtocol($transport, true, true);

$transport->open();
$processor->process($protocol, $protocol);
$transport->close();

```

我们首先包含了`ThriftClassLoader`类。然后，这个加载器类使我们能够为整个`Thrift`和`user`命名空间设置自动加载。然后，我们通过`GreetingServiceImpl`类实现了`hello()`和`goodbye()`方法。最后，我们实例化了适当的*handler*、*processor*、*transport*和*protocol*，以便能够处理传入的请求。

# 创建客户端

`thrift-service/client/`目录是我们将实现项目客户端的地方。让我们创建一个单一的`thrift-service/client/index.php`文件，从 Thrift 服务上的[`thrift-service.server/index.php`](http://thrift-service.server/index.php)调用`hello()`和`goodbye()`方法：

```php
<?php

require_once __DIR__ . '/../vendor/thrift/lib/php/lib/Thrift/ClassLoader/ThriftClassLoader.php';

use Thrift\ClassLoader\ThriftClassLoader;
use Thrift\Transport\THttpClient;
use Thrift\Transport\TBufferedTransport;
use Thrift\Protocol\TBinaryProtocol;
use user\GreetingServiceClient;

$loader = new ThriftClassLoader();
$loader->registerNamespace('Thrift', __DIR__ . '/../vendor/thrift/lib/php/lib');
$loader->registerDefinition('user', __DIR__ . '/../gen-php');
$loader->register();

$socket = new THttpClient('thrift-service.server', 80, '/index.php');
$transport = new TBufferedTransport($socket);
$protocol = new TBinaryProtocol($transport);
$client = new GreetingServiceClient($protocol);

$transport->open();

echo $client->hello('John');
echo $client->goodbye();

$transport->close();

```

就像服务器示例一样，在这里，我们也首先包含了`ThriftClassLoader`类，这样就能够为整个`Thrift`和`user`命名空间设置自动加载。然后我们实例化了 socket、传输、协议和客户端，从而与 Thrift 服务建立了连接。客户端和服务器都使用相同的`thrift-service/gen-php/user/GreetingService.php`文件。鉴于`GreetingServiceClient`位于自动生成的`GreetingService.php`文件中，这使得客户端可以立即了解`GreetingService`可能公开的任何方法。

要测试我们的客户端，我们只需要在浏览器中打开[`thrift-service.client/index.php`](http://thrift-service.client/index.php)。这应该给我们以下输出：

![](img/237508b6-9a68-44b0-bcdb-259fd4979808.png)

在本节中，我们触及了 Apache Thrift 服务的一些关键点。虽然关于 Thrift 的 IDL 和类型系统还有很多要说的，但这里呈现的示例是朝着正确方向迈出的一步。

# 理解微服务

术语“微服务”表示一种以松散耦合服务形式构建应用程序的架构风格。这些独立部署的服务通常是通过 Web 服务技术构建的微型应用程序。一个服务可以通过 SOAP 进行通信，另一个可以通过 REST、Apache Thrift 或其他方式进行通信。这里没有规定明确的要求。总体思想是将一个庞大的单体应用程序切割成几个更小的应用程序，即服务，但要以符合业务目标的方式进行切割。

以下图表试图可视化这个概念：

![](img/3779faf8-7bb6-459d-86f0-e4a652e50c7d.png)

由 Netflix 和亚马逊等公司推广，微服务风格旨在解决现代应用开发的一些关键挑战，其中包括以下几点：

+   **开发团队规模**：这是一个可以由相对较小的团队开发的单个微服务

+   **开发技能的多样性**：这些是可以用不同编程语言编写的不同服务

+   **更改/升级**：这些更小的代码片段更容易更改或更新

+   **集成和部署**：这些更小的代码片段更容易部署

+   **对新手更容易**：这些更小的代码片段更容易跟上

+   **业务能力聚焦**：这个单独的服务代码是围绕特定的业务能力组织的

+   **可扩展性**：并非所有东西都能等比例扩展；更小的代码块可以更容易地扩展

+   **故障处理**：这个单个故障服务不会导致整个应用程序崩溃

+   **技术栈**：这减少了对快速链接技术栈的依赖

与此同时，它们也带来了一些新的挑战，其中包括以下几点：

+   **服务通信**：这是围绕服务通信涉及的额外工作

+   **分布式事务**：这些是由跨越多个服务的业务需求引起的挑战

+   **测试和监控**：这比单体应用程序更具挑战性

+   **网络延迟**：每个微服务都会引入额外的网络延迟

+   **容错性**：这些微服务必须从根本上设计为容错

也就是说，构建微服务绝非易事。首先采用*单体架构*，以精心解耦和模块化的结构作为大多数应用的更好起点。一旦单体应用增长到影响我们管理方式的复杂程度，那么就是考虑将其切分为微服务的时候了。

# 总结

在本章中，我们研究了两种最常见和成熟的网络服务：SOAP 和 REST。我们还研究了一个新兴的明星，叫做 Apache Thrift。一旦我们通过了初始的 Apache Thrift 安装和设置障碍，诸如简单性、可扩展性、速度和可移植性等特性就会成为焦点。正如我们在客户端示例中看到的，RPC 调用可以很容易地通过一个中央代码库来实现——在我们的情况下是`thrift-service/gen-php/`目录。

虽然 Apache Thrift 在流行度方面还有待赶上，但它被 Facebook、Evernote、Pinterest、Quora、Uber 等知名公司使用的事实，无疑说明了它的价值。这并不是说未来方面 SOAP 或 REST 就不重要。选择正确的服务类型是一种*谨慎规划*和*前瞻思维*的问题。

最后，我们简要介绍了一种新兴的架构风格，称为微服务的一些关键要点。

前进时，我们将更仔细地研究在 PHP 应用程序中使用的一些最常用的数据库：MySQL、Mongo、Elasticsearch 和 Redis。
