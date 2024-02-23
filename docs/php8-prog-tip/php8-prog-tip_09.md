# 第九章：*第七章*：在使用 PHP 8 扩展时避免陷阱

**PHP：超文本预处理器**（**PHP**）语言的主要优势之一是它的扩展。在 PHP 8 中引入的对 PHP 语言的更改也要求扩展开发团队同时更新他们的扩展。在本章中，您将了解对扩展所做的主要更改以及如何避免在将现有应用程序更新到 PHP 8 时出现陷阱。

一旦您完成了对本章中提供的示例代码和主题的审阅，您将能够准备好将任何现有的 PHP 代码迁移到 PHP 8。除了了解各种扩展的变化外，您还将深入了解它们的操作。这将使您能够在使用 PHP 8 中的扩展时做出明智的决策。

本章涵盖的主题包括以下内容：

+   理解从资源到对象的转变

+   学习有关**可扩展标记语言**（**XML**）扩展的变化

+   避免更新的`mbstring`扩展出现问题

+   处理`gd`扩展的变化

+   发现`Reflection`扩展的变化

+   处理其他扩展的陷阱

# 技术要求

要查看并运行本章提供的代码示例，以下是最低推荐的硬件要求：

+   基于 x86_64 的台式 PC 或笔记本电脑

+   1 **千兆字节**（**GB**）的免费磁盘空间

+   4 GB 的**随机存取存储器**（**RAM**）

+   500 **千位每秒**（**Kbps**）或更快的互联网连接

此外，您需要安装以下软件：

+   Docker

+   Docker Compose

有关 Docker 和 Docker Compose 安装的更多信息，请参阅*第一章*的*技术要求*部分，*介绍新的 PHP 8 面向对象编程功能*，以及如何构建一个类似于用于演示本书中使用的代码的 Docker 容器。在本书中，我们将您为本书恢复的示例代码的目录称为`/repo`。

本章的源代码位于此处：

[`github.com/PacktPublishing/PHP-8-Programming-Tips-Tricks-and-Best-Practices/tree/main/ch07`](https://github.com/PacktPublishing/PHP-8-Programming-Tips-Tricks-and-Best-Practices/tree/main/ch07

)

我们现在可以开始讨论，在 PHP 8 中向对象而不是资源的整体趋势。

# 理解从资源到对象的转变

PHP 语言一直与**资源**有着不稳定的关系。资源代表着与外部系统的连接，比如文件句柄或使用**客户端 URL**（**cURL**）扩展连接到远程网络服务。然而，资源的一个大问题是，它们无法进行数据类型的区分。无法区分文件句柄和`cURL`连接——它们都被标识为资源。

在 PHP 8 中，已经进行了大力的努力，摆脱资源并用对象替换它们。在 PHP 8 之前这种趋势的最早例子之一是`PDO`类。当您创建一个`PDO`实例时，它会自动创建一个数据库连接。从 PHP 8 开始，许多以前产生资源的函数现在产生对象实例。让我们开始讨论一下现在产生对象而不是资源的扩展函数。

## PHP 8 扩展资源到对象的迁移

重要的是要知道在 PHP 8 中哪些函数现在产生对象而不是资源。好消息是扩展函数也已经被重写，以适应对象作为参数而不是资源。坏消息是，在初始化资源（现在是对象）并使用`is_resource()`函数进行成功测试时，可能会出现向后兼容的代码中断。

以下表格总结了以前返回资源但现在返回对象实例的函数：

![表 7.1 – PHP 8 资源到对象的迁移](img/Table_7.1_B16992.jpg)

表 7.1 - PHP 8 资源到对象的迁移

*表 7.1*是一个宝贵的指南，列出了现在产生对象而不是资源的函数。在将任何现有应用程序迁移到 PHP 8 之前，请参考此表。接下来的部分将详细介绍潜在的向后兼容代码中断，并指导您如何调整有问题的代码，然后再介绍其优势。

## 涉及 is_resource()的潜在代码中断

您可能会遇到的问题是，PHP 8 之前编写的代码假定*表 7.1*中列出的函数返回一个*资源*。因此，聪明的开发人员习惯于使用`is_resource()`来测试连接是否成功建立。

虽然这是一个非常明智的检查方式，但在 PHP 8 升级后，这种技术现在引入了一个向后兼容的代码中断。以下示例演示了这个问题。

在这个代码示例中，为一个外部网站初始化了一个`cURL`连接。接下来的几行代码使用`is_resource()`函数测试成功与否：

```php
// //repo/ch07/php7_ext_is_resource.php
$url = 'https://unlikelysource.com/';
$ch  = curl_init($url);
if (is_resource($ch))
    echo "Connection Established\n"
else
    throw new Exception('Unable to establish connection');
```

以下是来自 PHP 7 的输出，显示成功：

```php
root@php8_tips_php7 [ /repo/ch07 ]# 
php php7_ext_is_resource.php 
Connection Established
```

在 PHP 8 中运行相同代码的输出并不成功，如下所示：

```php
root@php8_tips_php8 [ /repo/ch07 ]# 
php php7_ext_is_resource.php 
PHP Fatal error:  Uncaught Exception: Unable to establish connection in /repo/ch07/php7_ext_is_resource.php:9
```

从 PHP 8 的输出来看，连接已经建立了！但是，由于程序代码正在检查`cURL`句柄是否是一个资源，因此代码会抛出一个`Exception`错误。失败的原因是因为返回的是一个`CurlHandle`实例，而不是一个资源。

在这种情况下，您可以通过在`is_resource()`的位置使用`!empty()`（非空）来避免代码中断，并使代码在 PHP 8 和任何早期的 PHP 版本中成功运行，如下所示：

```php
// //repo/ch07/php8_ext_is_resource.php
$url = 'https://unlikelysource.com/';
$ch  = curl_init($url);
if (!empty($ch))
    echo "Connection Established\n";
else
    throw new Exception('Unable to establish connection');
var_dump($ch);
```

以下是在 PHP 7 中运行代码示例的输出：

```php
root@php8_tips_php7 [ /repo/ch07 ]# 
php php8_ext_is_resource.php 
Connection Established
/repo/ch07/php8_ext_is_resource.php:11:
resource(4) of type (curl)
```

以下是在 PHP 8 中运行相同代码示例的输出：

```php
root@php8_tips_php8 [ /repo/ch07 ]# 
php php8_ext_is_resource.php 
Connection Established
object(CurlHandle)#1 (0) {}
```

从这两个输出中可以看到，代码都成功运行了：在 PHP 7 中，`$ch`是一个*资源*。在 PHP 8 中，`$ch`是一个`CurlHandle`实例。现在您已经了解了关于`is_resource()`的潜在问题，让我们来看看这种变化带来的优势。

## 对象相对于资源的优势

在 PHP 8 之前，没有办法在将资源传递到函数或方法中或从函数或方法中返回资源时提供数据类型。产生对象而不是资源的明显优势是，您可以利用对象类型提示。

为了说明这个优势，想象一组实现**策略软件设计模式**的**超文本传输协议**（**HTTP**）客户端类。其中一种策略涉及使用`cURL`扩展来发送消息。另一种策略使用 PHP 流，如下所示：

1.  我们首先定义一个`Http/Request`类。类构造函数将给定的 URL 解析为其组成部分，如下所示的代码片段所示：

```php
// /repo/src/Http/Request.php
namespace Http;
class Request {
    public $url      = '';
    public $method   = 'GET';
    // not all properties shown
    public $query    = '';
    public function __construct(string $url) {
        $result = [];
        $parsed = parse_url($url);
        $vars   = array_keys(get_object_vars($this));
        foreach ($vars as $name)
            $this->$name = $parsed[$name] ?? '';
        if (!empty($this->query))
            parse_str($this->query, $result);
        $this->query = $result;
        $this->url   = $url;
    }
}
```

1.  接下来，我们定义一个`CurlStrategy`类，它使用`cURL`扩展来发送消息。请注意，`__construct()`方法使用了构造函数参数推广。您可能还注意到，我们为`$handle`参数提供了一个`CurlHandle`数据类型。这是 PHP 8 中独有的巨大优势，它确保了创建此策略类实例的任何程序都必须提供正确的资源数据类型。代码如下所示：

```php
// /repo/src/Http/Client/CurlStrategy.php
namespace Http\Client;
use CurlHandle;
use Http\Request;
class CurlStrategy {
    public function __construct(
        public CurlHandle $handle) {}
```

1.  然后我们定义了用于发送消息的实际逻辑，如下所示：

```php
    public function send(Request $request) {
        // not all code is shown
        curl_setopt($this->handle, 
            CURLOPT_URL, $request->url);
        if (strtolower($request->method) === 'post') {
            $opts = [CURLOPT_POST => 1,
                CURLOPT_POSTFIELDS =>
                    http_build_query($request->query)];
            curl_setopt_array($this->handle, $opts);
        }
        return curl_exec($this->handle);
    }
}
```

1.  然后我们可以使用`StreamsStrategy`类做同样的事情。再次注意下面的代码片段中如何使用类作为构造函数参数类型提示，以确保正确使用该策略：

```php
// /repo/src/Http/Client/StreamsStrategy.php
namespace Http\Client;
use SplFileObject;
use Exception;
use Http\Request;
class StreamsStrategy {
    public function __construct(
        public ?SplFileObject $obj) {}
    // remaining code not shown
```

1.  然后我们定义一个调用程序，调用两种策略并提供结果。在设置自动加载后，我们创建一个新的`Http\Request`实例，并提供一个任意的 URL 作为参数，如下所示：

```php
// //repo/ch07/php8_objs_returned.php
require_once __DIR__ 
    . '/../src/Server/Autoload/Loader.php';
$autoload = new \Server\Autoload\Loader();
use Http\Request;
use Http\Client\{CurlStrategy,StreamsStrategy};
$url = 'https://api.unlikelysource.com/api
    ?city=Livonia&country=US';
$request = new Request($url);
```

1.  接下来，我们定义一个`StreamsStrategy`实例并发送请求，如下所示：

```php
$streams  = new StreamsStrategy();
$response = $streams->send($request);
echo $response;
```

1.  然后我们定义一个`CurlStrategy`实例并发送相同的请求，如下所示的代码片段所示：

```php
$curl     = new CurlStrategy(curl_init());
$response = $curl->send($request);
echo $response;
```

两种策略的输出是相同的。这里显示了部分输出（请注意，此示例仅适用于 PHP 8！）：

```php
root@php8_tips_php8 [ /repo/ch07 ]# 
php php8_objs_returned.php 
CurlStrategy Results:
{"data":[{"id":"1227826","country":"US","postcode":"14487","city":"Livonia","state_prov_name":"New York","state_prov_code":"NY","locality_name":"Livingston","locality_code":"051","region_name":"","region_code":"","latitude":"42.8135","longitude":"-77.6635","accuracy":"4"},{"id":"1227827","country":"US","postcode":"14488","city":"Livonia Center","state_prov_name":"New York","state_prov_code":"NY","locality_name":"Livingston","locality_code":"051","region_name":"","region_code":"","latitude":"42.8215","longitude":"-77.6386","accuracy":"4"}]}
```

现在让我们来看看资源到对象迁移的另一个方面：它对迭代的影响。

## Traversable 到 IteratorAggregate 的迁移

**Traversable**接口首次在 PHP 5 中引入。它没有方法，主要是为了允许对象使用简单的`foreach()`循环进行迭代。随着 PHP 的发展不断演进，通常需要获取内部迭代器。因此，在 PHP 8 中，许多以前实现`Traversable`的类现在改为实现`IteratorAggregate`。

这并不意味着增强的类不再支持`Traversable`接口固有的能力。相反，`IteratorAggregate`扩展了`Traversable`！这一增强意味着您现在可以在任何受影响的类的实例上调用`getIterator()`。这可能是巨大的好处，因为在 PHP 8 之前，没有办法访问各种扩展中使用的内部迭代器。以下表总结了受此增强影响的扩展和类：

![表 7.2 - 现在实现 IteratorAggregate 而不是 Traversable 的类](img/Figure_7.2_B16231.jpg)

表 7.2 - 现在实现 IteratorAggregate 而不是 Traversable 的类

在本节中，您了解了 PHP 8 中引入的一个重大变化：向对象而不是资源的趋势。您学到的一个优势是，与资源相比，对象允许您更好地控制。本节涵盖的另一个优势是，PHP 8 中向`IteratorAggregate`的转变允许访问以前无法访问的内置迭代器。

现在我们将注意力转向基于 XML 的扩展的变化。

# 学习关于 XML 扩展的变化

XML 版本 1.0 于 1998 年作为**万维网联盟**（**W3C**）规范引入。XML 与**超文本标记语言**（**HTML**）有些相似；然而，XML 的主要目的是提供一种使数据对机器和人类都可读的格式化方式。XML 仍然被广泛使用的原因之一是因为它易于理解，并且在表示树形数据方面表现出色。

PHP 提供了许多扩展，允许您消耗和生成 XML 文档。在 PHP 8 中，对许多这些扩展进行了一些更改。在大多数情况下，这些更改都很小；然而，如果您希望成为一个全面了解的 PHP 开发人员，了解这些更改是很重要的。

让我们首先看一下对`XMLWriter`扩展的变化。

## 检查 XMLWriter 扩展的差异

所有`XMLWriter`扩展的过程式函数现在接受并返回`XMLWriter`对象，而不是资源。然而，如果您查看`XMLWriter`扩展的官方 PHP 文档，您将看不到有关过程式函数的引用。原因有两个：首先，PHP 语言正在逐渐摆脱离散的过程式函数，转而支持**面向对象编程**（**OOP**）。

第二个原因是，`XMLWriter`过程式函数实际上只是`XMLWriter` OOP 方法的包装！例如，`xmlwriter_open_memory()`是`XMLWriter::openMemory()`的包装，`xmlwriter_text()`是`XMLWriter::text()`的包装，依此类推。

如果您真的打算使用过程式编程技术使用`XMLWriter`扩展，`xmlwriter_open_memory()`在 PHP 8 中创建一个`XMLWriter`实例，而不是一个资源。同样，所有`XMLWriter`扩展的过程式函数都使用`XMLWriter`实例而不是资源。

与本章中提到的任何扩展一样，现在产生对象实例而不是资源的潜在向后兼容性破坏是可能的。这种破坏的一个例子是，当您使用`XMLWriter`过程函数和`is_resource()`来检查是否已创建资源时。我们在这里没有向您展示一个例子，因为问题和解决方案与前一节中描述的相同：使用`!empty()`而不是`is_resource()`。

使用`XMLWriter`扩展的 OOP **应用程序编程接口**（**API**）而不是过程 API 是一种*最佳实践*。幸运的是，OOP API 自 PHP 5.1 以来就已经可用。以下是下一个示例中要使用的示例 XML 文件：

```php
<?xml version="1.0" encoding="UTF-8"?>
<fruit>
    <item>Apple</item>
    <item>Banana</item>
</fruit>
```

这里显示的示例在 PHP 7 和 8 中都可以工作。此示例的目的是使用`XMLWriter`扩展来构建先前显示的 XML 文档。以下是完成此操作的步骤：

1.  我们首先创建一个`XMLWriter`实例。然后打开到共享内存的连接，并初始化 XML 文档类型，如下所示：

```php
// //repo/ch07/php8_xml_writer.php
$xml = new XMLWriter();
$xml->openMemory();
$xml->startDocument('1.0', 'UTF-8');
```

1.  接下来，我们使用`startElement()`来初始化`fruit`根节点，并添加一个值为`Apple`的子节点项，如下所示：

```php
$xml->startElement('fruit');
$xml->startElement('item');
$xml->text('Apple');
$xml->endElement();
```

1.  接下来，我们添加另一个值为`Banana`的子节点项，如下所示：

```php
$xml->startElement('item');
$xml->text('Banana');
$xml->endElement();
```

1.  最后，我们关闭`fruit`根节点并结束 XML 文档。以下代码片段中的最后一个命令显示当前的 XML 文档：

```php
$xml->endElement();
$xml->endDocument();
echo $xml->outputMemory();
```

以下是在 PHP 7 中运行的示例程序的输出：

```php
root@php8_tips_php7 [ /repo/ch07 ]# php php8_xml_writer.php 
<?xml version="1.0" encoding="UTF-8"?>
<fruit><item>Apple</item><item>Banana</item></fruit>
```

如您所见，生成了所需的 XML 文档。如果我们在 PHP 8 中运行相同的程序，结果是相同的（未显示）。

现在我们将注意力转向`SimpleXML`扩展的更改。

## 处理 SimpleXML 扩展的更改

`SimpleXML`扩展是面向对象的，被广泛使用。因此，了解在 PHP 8 中对该扩展进行的一些重大更改是至关重要的。好消息是，您不需要重写任何代码！更好的消息是，这些更改大大改善了`SimpleXML`扩展的功能。

从 PHP 8 开始，`SimpleXMLElement`类现在实现了**标准 PHP 库**（**SPL**）`RecursiveIterator`接口，并包括`SimpleXMLIterator`类的功能。在 PHP 8 中，`SimpleXMLIterator`现在是`SimpleXMLElement`的一个空扩展。这个看似简单的更新在考虑到 XML 通常用于表示复杂的树形数据时具有重大意义。

例如，看一下*温莎王室*家族树的部分视图，如下所示：

![图 7.1 - 复杂树形数据的示例](img/Figure_7.3_B16231.jpg)

图 7.1 - 复杂树形数据的示例

如果我们要使用 XML 对其进行建模，文档可能如下所示：

```php
<?xml version="1.0" encoding="UTF-8"?>
<!-- /repo/ch07/tree.xml -->
<family>
  <branch name="Windsor">
    <descendent gender="M">George V</descendent>
    <spouse gender="F">Mary of Treck</spouse>
    <branch name="George V">
      <descendent gender="M">George VI</descendent>
      <spouse gender="F">Elizabeth Bowes-Lyon</spouse>
      <branch name="George VI">
        <descendent gender="F">Elizabeth II</descendent>
        <spouse gender="M">Prince Philip</spouse>
        <branch name="Elizabeth II">
          <descendent gender="M">Prince Charles</descendent>
          <spouse gender="F">Diana Spencer</spouse>
          <spouse gender="F">Camilla Parker Bowles</spouse>
          <branch name="Prince Charles">
            <descendent gender="M">William</descendent>
            <spouse gender="F">Kate Middleton</spouse>
          </branch>
          <!-- not all nodes are shown -->
        </branch>
      </branch>
    </branch>
  </branch>
</family>
```

然后，我们编写代码来解析树。然而，在 PHP 8 之前的版本中，我们需要定义一个递归函数来解析整个树。为此，我们将按照以下步骤进行：

1.  我们首先定义一个递归函数，显示后代的姓名和配偶（如果有），如下面的代码片段所示。该函数还识别后代的性别，并检查是否有子女。如果后者为`true`，则函数会调用自身：

```php
function recurse($branch) {
    foreach ($branch as $node) {
        echo $node->descendent;
        echo ($node->descendent['gender'] == 'F')
             ? ', daughter of '
             : ', son of ';
        echo $node['name'];
        if (empty($node->spouse)) echo "\n";
        else echo ", married to {$node->spouse}\n";
        if (!empty($node->branch)) 
            recurse($node->branch);
    }
}
```

1.  然后我们从外部 XML 文件创建一个`SimpleXMLElement`实例，并调用递归函数，如下所示：

```php
// //repo/ch07/php7_simple_xml.php
$fn = __DIR__ . '/includes/tree.xml';
$xml = simplexml_load_file($fn);
recurse($xml);
```

这段代码块在 PHP 7 和 PHP 8 中都可以工作。以下是在 PHP 7 中运行的输出：

```php
root@php8_tips_php7 [ /repo/ch07 ]# php php7_simple_xml.php
George V, son of Windsor, married to Mary of Treck
George VI, son of George V, married to Elizabeth Bowes-Lyon
Elizabeth II, daughter of George VI, married to Philip
Prince Charles, son of Elizabeth II, married to Diana Spencer
William, son of Prince Charles, married to Kate Middleton
Harry, son of Prince Charles, married to Meghan Markle
Princess Anne, daughter of Elizabeth II, married to M.Phillips
Princess Margaret, daughter of George VI, married to A.Jones
Edward VIII, son of George V, married to Wallis Simpson
Princess Mary, daughter of George V, married to H.Lascelles
Prince Henry, son of George V, married to Lady Alice Montegu
Prince George, son of George V, married to Princess Marina
Prince John, son of George V
```

然而，在 PHP 8 中，由于`SimpleXMLElement`现在实现了`RecursiveIterator`，生成相同结果的代码更简单了。

1.  与之前显示的示例一样，我们从外部文件定义了一个`SimpleXMLElement`实例。但是，我们无需定义递归函数，我们只需要定义一个`RecursiveIteratorIterator`实例，如下所示：

```php
// //repo/ch07/php8_simple_xml.php
$fn = __DIR__ . '/includes/tree.xml';
$xml = simplexml_load_file($fn);
$iter = new RecursiveIteratorIterator($xml,
    RecursiveIteratorIterator::SELF_FIRST);
```

1.  之后，我们只需要一个简单的`foreach()`循环，内部逻辑与前面的示例相同。无需检查分支节点是否存在，也不需要递归 - 这由`RecursiveIteratorIterator`实例处理！您需要的代码如下所示：

```php
foreach ($iter as $branch) {
    if (!empty($branch->descendent)) {
        echo $branch->descendent;
        echo ($branch->descendent['gender'] == 'F')
             ? ', daughter of '
             : ', son of ';
        echo $branch['name'];
        if (empty($branch->spouse)) echo "\n";
        else echo ", married to {$branch->spouse}\n";
    }
}
```

在 PHP 8 中运行此代码示例的输出如下所示。如您所见，输出完全相同：

```php
root@php8_tips_php8 [ /repo/ch07 ]# php php8_simple_xml.php 
George V, son of Windsor, married to Mary of Treck
George VI, son of George V, married to Elizabeth Bowes-Lyon
Elizabeth II, daughter of George VI, married to Philip
Prince Charles, son of Elizabeth II, married to Diana Spencer
William, son of Prince Charles, married to Kate Middleton
Harry, son of Prince Charles, married to Meghan Markle
Princess Anne, daughter of Elizabeth II, married to M.Phillips
Princess Margaret, daughter of George VI, married to A.Jones
Edward VIII, son of George V, married to Wallis Simpson
Princess Mary, daughter of George V, married to H.Lascelles
Prince Henry, son of George V, married to Lady Alice Montegu
Prince George, son of George V, married to Princess Marina
Prince John, son of George V
```

重要提示

请注意，在使用 Docker 容器运行这些示例时，这里显示的输出已经稍作修改以适应页面宽度。

现在让我们来看看其他 XML 扩展的更改。

## 了解其他 XML 扩展的更改

其他 PHP 8 XML 扩展已经进行了一些更改。在大多数情况下，这些更改都很小，并且不会对向后兼容的代码造成重大潜在破坏。然而，如果我们不解决这些额外的更改，那就不尽职了。我们建议您查看本小节中的其余更改，以提高您的意识。使用这些 XML 扩展将使您能够在 PHP 8 更新后更有效地排除应用程序代码的不一致行为。

### libxml 扩展的更改

**libxml**扩展利用**Expat C 库**，提供了各种 PHP XML 扩展使用的 XML 解析功能（[`libexpat.github.io/`](https://libexpat.github.io/)）。

您的服务器上安装的`libxml`版本有新的要求。在运行 PHP 8 时，最低版本必须为 2.9.0（或更高）。此更新要求的主要好处之一是增加对**XML 外部实体**（**XXE**）处理攻击的保护。

推荐的`libxml`最低版本禁用了依赖`libxml`扩展加载外部 XML 实体的 PHP XML 扩展的能力。这反过来减少了对 XXE 攻击的昂贵和耗时的额外步骤的需求。

提示

有关 XXE 攻击的更多信息，请参阅**开放式 Web 应用安全项目**（**OWASP**）[`owasp.org/www-community/vulnerabilities/XML_External_Entity_(XXE)_Processing`](https://owasp.org/www-community/vulnerabilities/XML_External_Entity_(XXE)_Processing)。

### XMLReader 扩展的更改

`XMLReader`扩展补充了`XMLWriter`扩展。`XMLWriter`扩展旨在生成 XML 文档，而`XMLReader`扩展旨在读取。

现在，`XMLReader::open()`和`XMLReader::xml()`两个方法被定义为**静态方法**。您仍然可以创建`XMLReader`实例，但如果您扩展`XMLReader`并覆盖其中任何一个方法，请确保将它们声明为静态方法。

### XMLParser 扩展的更改

`XMLParser`扩展是最古老的 PHP XML 扩展之一。因此，它几乎完全由过程函数组成，而不是类和方法。然而，在 PHP 8 中，这个扩展遵循了向生成对象而不是资源的趋势。因此，当您运行`xml_parser_create()`或`xml_parser_create_ns()`时，将创建一个`XMLParser`实例，而不是一个资源。

如在*涉及 is_resource()的潜在代码破坏*部分中所述，您只需要用`!empty()`替换任何使用`is_resource()`的检查。资源到对象迁移的另一个副作用是使`xml_parser_free()`函数变得多余。要停用解析器，只需使用`XmlParser`对象。

现在您已经了解了与 XML 扩展相关的更改，这将帮助您更有效地解析和管理 XML 数据。通过利用本节中提到的新功能，您可以编写比在 PHP 8 之前更高效并且性能更好的代码。现在让我们来看看`mbstring`扩展。

# 避免与更新后的 mbstring 扩展出现问题

`mbstring`扩展首次引入于 PHP 4，并且自那时以来一直是语言的活跃部分。该扩展的最初目的是为各种日语字符编码系统提供支持。自那时以来，已添加了对各种其他编码的支持，其中最显着的是对基于**通用编码字符集 2**（**UCS-2**）、**UCS-4**、**Unicode 转换格式 8**（**UTF-8**）、**UTF-16**、**UTF-32**、**Shift 日本工业标准**（**SJIS**）和**国际标准化组织 8859**（**ISO-8859**）等编码的支持。

如果您不确定服务器支持哪些编码，只需运行`mb_list_encodings()`命令，如下所示（显示部分输出）：

```php
root@php8_tips_php7 [ /repo/ch07 ]# 
php -r "var_dump(mb_list_encodings());"
Command line code:1:
array(87) {
  ... only selected output is shown ...
  [14] =>  string(7) "UCS-4BE"
  [16] =>  string(5) "UCS-2"
  [19] =>  string(6) "UTF-32"
  [22] =>  string(6) "UTF-16"
  [25] =>  string(5) "UTF-8"
  [26] =>  string(5) "UTF-7"
  [27] =>  string(9) "UTF7-IMAP"
  [28] =>  string(5) "ASCII"
  [29] =>  string(6) "EUC-JP"
  [30] =>  string(4) "SJIS"
  [31] =>  string(9) "eucJP-win"
  [32] =>  string(11) "EUC-JP-2004"
  [76] =>  string(6) "KOI8-R"
  [78] =>  string(9) "ArmSCII-8"
  [79] =>  string(5) "CP850"
  [80] =>  string(6) "JIS-ms"
  [81] =>  string(16) "ISO-2022-JP-2004"
  [86] =>  string(7) "CP50222"
}
```

从前面的输出中可以看出，在我们用于本书的 PHP 7.1 Docker 容器中，支持 87 种编码。在 PHP 8.0 Docker 容器中（未显示输出），支持 80 种编码。现在让我们来看一下 PHP 8 中引入的更改，首先是`mb_str*()`函数。

## 发现`mb_str*()`函数中的 needle-argument 差异

在*第六章*中，*了解 PHP 8 的功能差异*，您了解到 PHP 8 如何改变了核心`str*pos()`、`str*str()`和`str*chr()`函数中的**needle-argument 处理**。两个主要的 needle-argument 差异是能够接受空的 needle 参数和严格的类型检查，以确保 needle 参数只是一个字符串。为了保持一致性，PHP 8 在相应的`mb_str*()`函数中引入了相同的更改。

让我们首先看一下空的 needle-argument 处理。

### `mb_str*()`函数空 needle-argument 处理

为了使`mbstring`扩展与核心字符串函数的更改保持一致，以下`mbstring`扩展函数现在允许空的 needle 参数。重要的是要注意，这并不意味着参数可以被省略或是可选的！这个更改的意思是，作为 needle 参数提供的任何值现在也可以包括被认为是*空*的值。了解 PHP 认为什么是空的一个好而快速的方法可以在`empty()`函数的文档中找到（[`www.php.net/empty`](https://www.php.net/empty)）。以下是现在允许空的 needle-argument 值的`mbstring`函数列表：

+   `mb_strpos()`

+   `mb_strrpos()`

+   `mb_stripos()`

+   `mb_strripos()`

+   `mb_strstr()`

+   `mb_stristr()`

+   `mb_strrchr()`

+   mb_strrichr（）

提示

这里提到的八个`mbstring`扩展函数与其核心 PHP 对应函数完全相同。有关这些函数的更多信息，请参阅此参考文档：[`www.php.net/manual/en/ref.mbstring.php`](https://www.php.net/manual/en/ref.mbstring.php)。

接下来的简短代码示例说明了上述八个函数中的空 needle 处理。以下是导致这一步的步骤：

1.  首先，我们初始化一个多字节文本字符串。在下面的示例中，这是*快速的棕色狐狸跳过了篱笆*的泰语翻译。needle 参数设置为`NULL`，并初始化要测试的函数数组：

```php
// /repo/ch07/php8_mb_string_empty_needle.php
$text   = 'สุนัขจิ้งจอกสีน้ำตาลกระโดดข้ามรั้วอย่างรวดเร็ว';
$needle = NULL;
$funcs  = ['mb_strpos',   'mb_strrpos', 'mb_stripos',
           'mb_strripos', 'mb_strstr', 'mb_stristr',
           'mb_strrchr',  'mb_strrichr'];
```

1.  然后我们定义一个`printf()`模式，并循环遍历要测试的函数。对于每个函数调用，我们提供文本，然后是一个空的 needle 参数，如下所示：

```php
$patt = "Testing: %12s : %s\n";
foreach ($funcs as $str)
    printf($patt, $str, $str($text, $needle));
```

PHP 7 的输出如下所示：

```php
root@php8_tips_php7 [ /repo/ch07 ]# 
php php8_mb_string_empty_needle.php
PHP Warning:  mb_strpos(): Empty delimiter in /repo/ch07/php8_mb_string_empty_needle.php on line 12
Testing:    mb_strpos : 
Testing:   mb_strrpos : 
PHP Warning:  mb_stripos(): Empty delimiter in /repo/ch07/php8_mb_string_empty_needle.php on line 12
Testing:   mb_stripos : 
Testing:  mb_strripos : 
PHP Warning:  mb_strstr(): Empty delimiter in /repo/ch07/php8_mb_string_empty_needle.php on line 12
Testing:    mb_strstr : 
PHP Warning:  mb_stristr(): Empty delimiter in /repo/ch07/php8_mb_string_empty_needle.php on line 12
Testing:   mb_stristr : 
Testing:   mb_strrchr : 
Testing:  mb_strrichr : 
```

正如您所看到的，输出为空，并且在某些情况下会发出`Warning`消息。PHP 8 中的输出与预期的完全不同，如下所示：

```php
root@php8_tips_php8 [ /repo/ch07 ]# 
php php8_mb_string_empty_needle.php
Testing:    mb_strpos : 0
Testing:   mb_strrpos : 46
Testing:   mb_stripos : 0
Testing:  mb_strripos : 46
Testing:    mb_strstr : สุนัขจิ้งจอกสีน้ำตาลกระโดดข้ามรั้วอย่างรวดเร็ว
Testing:   mb_stristr : สุนัขจิ้งจอกสีน้ำตาลกระโดดข้ามรั้วอย่างรวดเร็ว
Testing:   mb_strrchr : 
Testing:  mb_strrichr : 
```

有趣的是，当这段代码在 PHP 8 中运行时，空的针参数对于`mb_strpos()`和`mb_stripos()`返回整数`0`，对于`mb_strrpos()`和`mb_strripos()`返回整数`46`。在 PHP 8 中，空的针参数在这种情况下被解释为字符串的开头或结尾。对于`mb_strstr()`和`mb_stristr()`的结果是整个字符串。

### mb_str*()函数数据类型检查

为了与核心`str*()`函数保持一致，相应的`mb_str*()`函数中的针参数必须是字符串类型。如果你提供的是**美国信息交换标准代码**（ASCII）值而不是字符串，受影响的函数现在会抛出`ArgumentTypeError`错误。本小节不提供示例，因为[*第六章*]（B16992_06_Final_JC_ePub.xhtml#_idTextAnchor129），*理解 PHP 8 的功能差异*，已经提供了核心`str*()`函数中这种差异的示例。

### mb_strrpos()的差异

在早期的 PHP 版本中，你可以将字符编码作为`mb_strrpos()`的第三个参数而不是偏移量。这种不良做法在 PHP 8 中不再支持。相反，你可以将`0`作为第三个参数，或者考虑使用 PHP 8 的*命名参数*（在[*第一章*]（B16992_01_Final_JC_ePub.xhtml#_idTextAnchor013），*介绍新的 PHP 8 面向对象特性*，*理解命名参数*部分讨论）来避免必须提供一个可选参数的值。

让我们现在看一个代码示例，演示了 PHP 7 和 PHP 8 处理方式的差异。按照以下步骤进行：

1.  我们首先定义一个常量来表示我们希望使用的字符编码。分配一个代表*The quick brown fox jumped over the fence*泰语翻译的文本字符串。然后我们使用`mb_convert_encoding()`来确保使用正确的编码。代码如下所示：

```php
// /repo/ch07/php7_mb_string_strpos.php
define('ENCODING', 'UTF-8');
$text    = 'สุนัขจิ้งจอกสีน้ำตาลกระโดดข้ามรั้วอย่างรวดเร็ว';
$encoded = mb_convert_encoding($text, ENCODING);
```

1.  然后我们将*fence*的泰语翻译分配给`$needle`，并输出字符串的长度和`$needle`在文本中的位置。然后我们调用`mb_strrpos()`来找到`$needle`的最后一次出现。请注意在以下代码片段中，我们故意遵循了使用编码作为第三个参数而不是偏移量的不良做法：

```php
$needle  = 'รั้ว';
echo 'String Length: ' 
    . mb_strlen($encoded, ENCODING) . "\n";
echo 'Substring Pos: ' 
    . mb_strrpos($encoded, $needle, ENCODING) . "\n";
```

这个代码示例在 PHP 7 中完美运行，如下所示：

```php
root@php8_tips_php7 [ /repo/ch07 ]# 
php php7_mb_string_strpos.php
String Length: 46
Substring Pos: 30
```

从前面的输出中可以看到，多字节字符串的长度为`46`，针的位置为`30`。然而，在 PHP 8 中，我们得到了一个致命的`Uncaught TypeError`消息，如下所示：

```php
root@php8_tips_php8 [ /repo/ch07 ]# 
php php7_mb_string_strpos.php
String Length: 46
PHP Fatal error:  Uncaught TypeError: mb_strrpos(): Argument #3 ($offset) must be of type int, string given in /repo/ch07/php7_mb_string_strpos.php:14
```

从 PHP 8 的输出中可以看到，`mb_strrpos()`的第三个参数必须是一个整数形式的偏移值。重写这个例子的一个简单方法是利用 PHP 8 的*命名参数*。以下是重写的代码行：

```php
echo 'Substring Pos: ' 
    . mb_strrpos($encoded, $needle, encoding:ENCODING) . "\n";
```

输出与 PHP 7 示例相同，这里不再显示。现在让我们转向`mbstring`扩展的**正则表达式**（**regex**）处理差异。

## 检查 mb_ereg*()函数的变化

`mb_ereg*()`函数族允许对使用多字节字符集编码的字符串进行**regex**处理。相比之下，核心 PHP 语言提供了现代和更为更新的功能的**Perl 兼容正则表达式**（**PCRE**）函数族。

当使用 PCRE 函数时，如果在正则表达式模式中添加`u`（小写字母 U）修饰符，则接受任何 UTF-8 编码的多字节字符串。然而，UTF-8 是唯一被接受的多字节字符编码。如果你处理其他字符编码并希望执行正则表达式功能，你需要将其转换为 UTF-8，或者使用`mb_ereg*()`函数族。现在让我们看看`mb_ereg*()`函数族的一些变化。

### PHP 8 中需要 Oniguruma 库

这一系列函数的一个变化是你的 PHP 安装是如何编译的。在 PHP 8 中，你的操作系统必须提供`libonig`库。这个库提供了**Oniguruma**功能。（更多信息请参见 https://github.com/kkos/oniguruma。）旧的`--with-onig`PHP 源码编译配置选项已经被移除，取而代之的是使用`pkg-config`来检测`libonig`。

### mb_ereg_replace()的变化

以前，你可以将整数作为`mb_ereg_replace()`的参数。这个参数被解释为**ASCII 码点**。在 PHP 8 中，这样的参数现在被强制转换为`string`。如果你需要 ASCII 码点，你需要使用`mb_chr()`。由于强制转换为`string`是静默进行的，这可能会导致向后兼容的代码中断，因为你不会看到任何`Notice`或`Warning`消息。

以下程序代码示例说明了 PHP 7 和 PHP 8 之间的区别。我们将按照以下步骤进行：

1.  首先，我们定义要使用的编码，并将“Two quick brown foxes jumped over the fence”的泰语翻译作为多字节字符串赋给`$text`。接下来，我们使用`mb_convert_encoding()`来确保使用正确的编码。然后，我们使用`mb_regex_encoding()`将`mb_ereg*`设置为所选的编码。代码如下所示：

```php
// /repo/ch07/php7_mb_string_strpos.php
define('ENCODING', 'UTF-8');
$text = 'สุนัขจิ้งจอกสีน้ำตาล 2 ตัวกระโดดข้ามรั้ว';
$str  = mb_convert_encoding($text, ENCODING);
mb_regex_encoding(ENCODING);
```

1.  然后我们调用`mb_ereg_replace()`，并将整数值`50`作为第一个参数，并用字符串`"3"`替换它。原始字符串和修改后的字符串都被输出。你可以在这里查看代码：

```php
$mod1 = mb_ereg_replace(50, '3', $str);
echo "Original: $str\n";
echo "Modified: $mod1\n";
```

请注意，`mb_ereg_replace()`的第一个参数应该是一个字符串，但我们却提供了一个整数。在 PHP 8 之前的`mbstring`扩展版本中，如果提供整数作为第一个参数，它会被视为 ASCII 码点。

如果我们在 PHP 7 中运行这个代码示例，数字`50`会被解释为`"2"`的 ASCII 码点值，正如我们所期望的那样，如下所示：

```php
root@php8_tips_php7 [ /repo/ch07 ]# 
php php7_mb_string_ereg_replace.php 
Original: สุนัขจิ้งจอกสีน้ำตาล 2 ตัวกระโดดข้ามรั้ว
Modified: สุนัขจิ้งจอกสีน้ำตาล 3 ตัวกระโดดข้ามรั้ว
```

从前面的输出中可以看到，数字`2`被数字`3`替换。然而，在 PHP 8 中，数字`50`被强制转换为字符串。由于源字符串不包含数字`50`，所以没有进行替换，如下所示：

```php
root@php8_tips_php8 [ /repo/ch07 ]# 
php php7_mb_string_ereg_replace.php 
Original: สุนัขจิ้งจอกสีน้ำตาล 2 ตัวกระโดดข้ามรั้ว
Modified: สุนัขจิ้งจอกสีน้ำตาล 2 ตัวกระโดดข้ามรั้ว
```

这里的危险在于，如果你的代码依赖于这种静默解释过程，你的应用可能会失败或表现出不一致的行为。你还会注意到缺少`Notice`或`Warning`消息。PHP 8 依赖于开发人员提供正确的参数！

*最佳实践*，如果你确实需要使用 ASCII 码点，就要使用`mb_chr()`来生成所需的搜索字符串。修改后的代码示例可能如下所示：

`$mod1 = mb_ereg_replace(mb_chr(50), '3', $str);`

现在你已经知道了`mbstring`扩展中的变化。没有这些信息，你可能会轻易地写出错误的代码。不了解这些信息的开发人员可能会在 PHP 8 中犯错，比如假设`mbstring`的别名仍然存在。这样错误的理解很容易导致在 PHP 8 迁移后花费数小时来追踪程序代码中的错误。

现在是时候看另一个有重大变化的扩展了：GD 扩展。

# 处理 GD 扩展的变化

**GD 扩展**是一个图像处理扩展，利用了`GD`库。GD 最初代表**GIF Draw**。奇怪的是，`GD`库在 Unisys 撤销生成 GIF 时使用的压缩技术的开源许可证后，不得不撤回对**Graphics Interchange Format**（**GIF**）的支持。然而，2004 年之后，Unisys 对这项技术的专利已经过期，GIF 支持得以恢复。截至目前，PHP `GD`扩展支持**Joint Photographic Experts Group**（**JPEG**或**JPG**）、**Portable Network Graphic**（**PNG**）、**GIF**、**X BitMap**（**XBM**）、**X PixMap**（**XPM**）、**Wireless Bitmap**（**WBMP**）、**WebP**和**Bitmap**（**BMP**）格式。

提示

有关 GD 库的更多信息，请参阅[`libgd.github.io/`](https://libgd.github.io/)。

现在让我们看看资源到对象迁移对`GD`扩展的影响。

## GD 扩展资源到对象迁移

与先前使用*资源*的其他 PHP 扩展一样，`GD`扩展也主要从`resource`迁移到`object`。如*PHP 8 扩展资源到对象迁移*部分所述，所有`imagecreate*()`函数现在产生`GdImage`对象而不是资源。

举个例子，这可能在 PHP 8 迁移后导致代码中断，可以在两个不同的浏览器标签中运行这些示例（在本地计算机上），并比较差异。首先，我们使用此 URL 运行 PHP 7 示例：[`172.16.0.77/ch07/php7_gd_is_resource.php`](http://172.16.0.77/ch07/php7_gd_is_resource.php)。这是结果：

![图 7.2 - PHP 7 GD 图像资源](img/Figure_7.4_B16231.jpg)

图 7.2 - PHP 7 GD 图像资源

从上述输出中可以看出，识别出了一个`resource`扩展，但没有描述性信息。现在，让我们使用此 URL 运行 PHP 8 示例：http://172.16.0.88/ch07/php8_gd_is_resource.php。这是结果：

![图 7.3 - PHP 8 GD 图像对象实例](img/Figure_7.5_B16231.jpg)

图 7.3 - PHP 8 GD 图像对象实例

从 PHP 8 的输出中不仅可以识别返回类型为`GdImage`实例，还可以在图像下方显示描述性信息。

现在我们将注意力转向其他`GD`扩展的变化。

## GD 扩展编译标志更改

`GD`扩展不仅利用 GD 库，还利用一些支持库。这些库需要提供对各种图形格式的支持。以前，在从源代码编译自定义版本的 PHP 时，您需要指定*JPEG*、*PNG*、*XPM*和*VPX*格式的库位置。此外，由于压缩是减少最终文件大小的重要方面，因此还需要`ZLIB`的位置。 

在从源代码编译 PHP 8 时，有一些重要的配置标志更改，这些更改首先出现在 PHP 7.4 中，随后被带入 PHP 8。主要变化是您不再需要指定库所在的目录。PHP 8 现在使用`pkg-config`操作系统等效工具来定位库。

以下表总结了编译标志的更改。这些标志与`configure`实用程序一起使用，就在实际编译过程之前：

![表 7.3 - GD 编译选项更改](img/Figure_7.6_B16231.jpg)

表 7.3 - GD 编译选项更改

您会注意到表中，大部分`--with-*-dir`选项都被替换为`--with-*`。此外，*PNG*和*ZLIB*支持现在是自动的；但是，您必须在操作系统上安装`libpng`和`zlib`。

我们现在将看看`GD`扩展的其他较小变化。

## 其他 GD 扩展变化

除了前一节描述的主要变化外，还发生了一些其他较小的变化，包括函数签名的变化和一个新函数。让我们从查看`imagecropauto()`函数开始讨论。

这是`imagecropauto()`的旧函数签名：

```php
imagecropauto(resource $image , int $mode = -1, 
              float $threshold = .5 , int $color = -1 )
```

在 PHP 8 中，`$image`参数现在是`GdImage`类型。`$mode`参数现在默认为`IMG_CROP_DEFAULT`预定义常量。

另一个变化影响了`imagepolygon()`，`imageopenpolygon()`和`imagefilledpolygon()`函数。这是`imagepolygon()`的旧函数签名：

```php
imagepolygon(resource $image, array $points, 
             int $num_points, int $color)
```

在 PHP 8 中，`$num_points`参数现在是可选的。如果省略，点数将计算如下：`count($points)/2`。但是，这意味着`$points`数组中的元素数量必须是偶数！

最后一个重要的变化是添加了一个新函数`imagegetinterpolation()`。这是它的函数签名：

`imagegetinterpolation(GdImage $image) : int`

返回值是一个整数，本身并不是非常有用。但是，如果您查看`imagesetinterpolation()`函数的文档（https://www.php.net/manual/en/function.imagesetinterpolation.php），您将看到一系列插值方法代码以及解释。

现在您已经了解了`GD`扩展引入的更改。接下来我们将检查`Reflection`扩展的更改。

# 发现 Reflection 扩展的更改

**Reflection 扩展**用于对对象、类、方法和函数等进行*内省*。`ReflectionClass`和`ReflectionObject`分别提供有关类或对象实例的信息。`ReflectionFunction`提供有关过程级函数的信息。此外，`Reflection`扩展还有一组由刚才提到的主要类产生的辅助类。这些辅助类包括`ReflectionMethod`，由`ReflectionClass::getMethod()`产生，`ReflectionProperty`，由`ReflectionClass::getProperty()`产生，等等。

您可能会想：*谁使用这个扩展？*答案是：任何需要对外部一组类执行分析的应用。这可能包括执行自动**代码生成**、**测试**或**文档生成**的软件。执行**hydration**（从数组中填充对象）的类也受益于`Reflection`扩展。

提示

我们在书中没有足够的空间来涵盖每一个`Reflection`扩展类和方法。如果您希望获得更多信息，请查看这里的文档参考：[`www.php.net/manual/en/book.reflection.php`](https://www.php.net/manual/en/book.reflection.php)。

现在让我们来看一个`Reflection`扩展的使用示例。

## Reflection 扩展的使用

我们将展示一个代码示例，演示了如何使用`Reflection`扩展来生成**docblocks**（`docblock`是使用特殊语法来表示方法目的、其传入参数和返回值的 PHP 注释）。以下是导致这一步的步骤：

1.  我们首先定义一个`__construct()`方法，创建目标类的`ReflectionClass`实例，如下所示：

```php
// /repo/src/Services/DocBlockChecker.php
namespace Services;
use ReflectionClass;
class DocBlockChecker {
    public $target = '';    // class to check
    public $reflect = NULL; // ReflectionClass instance
    public function __construct(string $target) {
        $this->target = $target;
        $this->reflect = new ReflectionClass($target);
    }
```

1.  然后我们定义一个`check()`方法，获取所有类方法，返回一个`ReflectionMethod`实例数组，如下所示：

```php
    public function check() {
        $methods = [];
        $list = $this->reflect->getMethods();
```

1.  然后我们循环遍历所有方法，并使用`getDocComment()`来检查是否已经存在`docblock`，如下所示：

```php
      foreach ($list as $refMeth) {
          $docBlock = $refMeth->getDocComment();
```

1.  如果`docblock`不存在，我们将开始一个新的`docblock`，然后调用`getParameters()`，它返回一个`ReflectionParameter`实例数组，如下面的代码片段所示：

```php
          if (!$docBlock) {
              $docBlock = "/**\n * " 
                  . $refMeth->getName() . "\n";
              $params = $refMeth->getParameters();
```

1.  如果我们有参数，我们收集用于显示的信息，如下所示：

```php
            if ($params) {
              foreach ($params as $refParm) {
                $type = $refParm->getType() 
                      ?? 'mixed';
                $type = (string) $type;
                $name = $refParm->getName();
                $default = '';
                if (!$refParm->isVariadic() 
                 && $refParm->isOptional()) {
                  $default=$refParm->getDefaultValue(); }
                if ($default === '') {
                  $default = "(empty string)"; }
                $docBlock .= " * @param $type "
                  . "\${$name} : $default\n";
              }
          }
```

1.  然后我们设置返回类型，并将`docblock`分配给一个`$methods`数组，然后返回，如下所示：

```php
           if ($refMeth->isConstructor())
               $return = 'void';
            else
                $return = $refMeth->getReturnType() 
                          ?? 'mixed';
            $docBlock .= " * @return $return\n";
            $docBlock .= " */\n";
        }
        $methods[$refMeth->getName()] = $docBlock;
    }
    return $methods;
  }
}
```

1.  现在新的`docblock`检查类已经完成，我们定义一个调用程序，如下面的代码片段所示。调用程序针对`/repo/src/Php7/Reflection/Test.php`类（此处未显示）。这个类具有一些带有参数和返回值的方法的混合：

```php
// //repo/ch07/php7_reflection_usage.php
$target = 'Php7\Reflection\Test';
require_once __DIR__ 
    . '/../src/Server/Autoload/Loader.php';
use Server\Autoload\Loader;
use Services\DocBlockChecker;
|$autoload = new Loader();
$checker = new DocBlockChecker($target);
var_dump($checker->check());
```

调用程序的输出如下所示：

```php
root@php8_tips_php7 [ /repo/ch07 ]# 
php php7_reflection_usage.php 
/repo/ch07/php7_reflection_usage.php:10:
array(4) {
  '__construct' =>  string(75) 
"/**
 * __construct
 * @param PDO $pdo : (empty string)
 * @return void
 */"
  'fetchAll' =>  string(41) 
"/**
 * fetchAll
 * @return Generator
 */"
  'fetchByName' =>  string(80) 
"/**
 * fetchByName
 * @param string $name : (empty string)
 * @return array
 */"
  'fetchLastId' =>  string(38) 
"/**
 * fetchLastId
 * @return int
 */"
}
```

正如您所看到的，这个类构成了潜在的自动文档或代码生成应用的基础。

现在让我们来看一下`Reflection`扩展的改进。

## 了解 Reflection 扩展的改进

`Reflection`扩展还进行了一些改进，这些改进可能对您很重要。请记住，虽然使用`Reflection`扩展的开发人员数量有限，但您可能有一天会发现自己在处理使用此扩展的代码的情况。如果您在 PHP 8 升级后注意到异常行为，本节介绍的内容将让您在故障排除过程中提前了解情况。

### ReflectionType 修改

在 PHP 8 中，`ReflectionType`类现在是抽象的。当您使用`ReflectionProperty::getType()`或`ReflectionFunction::getReturnType()`方法时，您可能会注意到返回一个`ReflectionNamedType`实例。这种变化不会影响您程序代码的正常运行，除非您依赖于返回`ReflectionType`实例。但是，`ReflectionNamedType`扩展了`ReflectionType`，因此`instanceof`操作不会受到影响。

值得注意的是，`isBuiltIn()`方法已经从`ReflectionType`移动到`ReflectionNamedType`。同样，由于`ReflectionNamedType`扩展了`ReflectionType`，这不应该在您当前的代码中造成任何向后兼容的问题。

### ReflectionParameter::*DefaultValue*方法增强

在早期版本的 PHP 中，关于默认值的`ReflectionParameter`方法无法反映内部 PHP 函数。在 PHP 8 中，以下`ReflectionParameter`方法现在也能够从内部函数返回默认值信息：

+   `getDefaultValue()`

+   `getDefaultValueConstantName()`

+   `isDefaultValueAvailable()`

+   `isDefaultValueConstant()`

从列表中可以看出，方法名称是不言自明的。我们现在将展示一个使用这些增强功能的代码示例。以下是导致这一步的步骤：

1.  首先，我们定义一个函数，接受一个`ReflectionParameter`实例，并返回一个包含参数名称和默认值的数组，如下所示：

```php
// /repo/ch07/php8_reflection_parms_defaults.php
$func = function (ReflectionParameter $parm) {
    $name = $parm->getName();
    $opts = NULL;
    if ($parm->isDefaultValueAvailable())
        $opts = $parm->getDefaultValue();
```

1.  接下来，我们定义一个`switch()`语句来清理选项，如下所示：

```php
    switch (TRUE) {
        case (is_array($opts)) :
            $tmp = '';
            foreach ($opts as $key => $val)
                $tmp .= $key . ':' . $val . ',';
            $opts = substr($tmp, 0, -1);
            break;
        case (is_bool($opts)) :
            $opts = ($opts) ? 'TRUE' : 'FALSE';
            break;
        case ($opts === '') :
            $opts = "''";
            break;
        default :
            $opts = 'No Default';
    }
    return [$name, $opts];
};
```

1.  然后我们确定要反射的函数并提取其参数。在下面的例子中，我们反射`setcookie()`：

```php
$test = 'setcookie';
$ref = new ReflectionFunction($test);
$parms = $ref->getParameters();
```

1.  然后，我们循环遍历`ReflectionParameter`实例的数组并产生输出，如下所示：

```php
$patt = "%18s : %s\n";
foreach ($parms as $obj)
    vprintf($patt, $func($obj));
```

以下是在 PHP 7 中运行的输出：

```php
root@php8_tips_php7 [ /repo/ch07 ]# 
php php8_reflection_parms_defaults.php 
Reflecting on setcookie
         Parameter : Default(s)
      ------------ : ------------
              name : No Default
             value : No Default
           expires : No Default
              path : No Default
            domain : No Default
            secure : No Default
          httponly : No Default
```

结果始终是`No Default`，因为在 PHP 7 及更早版本中，`Reflection`扩展无法读取内部 PHP 函数的默认值。另一方面，PHP 8 的输出要准确得多，正如我们在这里所看到的：

```php
root@php8_tips_php8 [ /repo/ch07 ]# 
php php8_reflection_parms_defaults.php 
Reflecting on setcookie
         Parameter : Default(s)
      ------------ : ------------
              name : No Default
             value : ''
expires_or_options : No Default
              path : ''
            domain : ''
            secure : FALSE
          httponly : FALSE
```

从输出中可以看出，PHP 8 中的`Reflection`扩展能够准确报告内部函数的默认值！

现在让我们看看其他`Reflection`扩展的变化。

### 其他反射扩展的更改

在 PHP 8 之前的 PHP 版本中，`ReflectionMethod::isConstructor()`和`ReflectionMethod::isDestructor()`无法反映在接口中定义的魔术方法。在 PHP 8 中，这两个方法现在对接口中定义的相应魔术方法返回`TRUE`。

在使用`ReflectionClass::getConstants()`或`ReflectionClass::getReflectionConstants()`方法时，现在添加了一个新的`$filter`参数。该参数允许您按可见性级别过滤结果。因此，新参数可以接受以下任何新添加的预定义常量之一：

+   `ReflectionClassConstant::IS_PUBLIC`

+   `ReflectionClassConstant::IS_PROTECTED`

+   `ReflectionClassConstant::IS_PRIVATE`

现在您已经了解了如何使用`Reflection`扩展以及在 PHP 8 迁移后可以期待什么。现在是时候看看在 PHP 8 中发生了变化的其他扩展了。

# 处理其他扩展的注意事项

PHP 8 引入了一些其他 PHP 扩展的值得注意的变化，除了本章已经讨论过的扩展。正如我们在本书中一再强调的那样，了解这些变化对于您作为 PHP 开发人员的未来职业非常重要。

让我们首先看看数据库扩展的变化。

## 新的数据库扩展操作系统库要求

任何使用**MySQL**、**MariaDB**、**PostgreSQL**或**PHP 数据对象**（**PDO**）的开发人员都需要注意支持操作系统库的新要求。以下表格总结了 PHP 8 中所需的新最低版本：

![表 7.4 – PHP 8 数据库库要求](img/Figure_7.7_B16231.jpg)

表 7.4 – PHP 8 数据库库要求

从上表可以看出，有两个主要的库更改。`libpq`影响了`PostgreSQL`扩展和`PDO`扩展的驱动程序。`libmysqlclient`是**MySQL Improved** (**MySQLi**)扩展和`PDO`扩展的 MySQL 驱动程序使用的库。还应该注意，如果您使用的是 MariaDB，MySQL 的一个流行的开源版本，新的最低`MySQL`库要求也适用于您。

既然您已经了解了数据库扩展的变化，接下来我们将把注意力转向 ZIP 扩展。

## 审查 ZIP 扩展的变化

ZIP 扩展用于以编程方式创建和管理压缩的存档文件，利用`libzip`操作系统库。还存在其他压缩扩展，如**Zlib**、**bzip2**、**LZF**、**PHP Archive Format** (**phar**)和**Roshal Archive Compressed** (**RAR**)；然而，其他扩展都没有`ZIP`扩展提供的丰富功能范围。此外，大多数情况下，其他扩展都是专用的，通常不适用于通用 ZIP 文件管理。

让我们首先看一下这个扩展最显著的变化。

### 处理 ZIP 扩展的 OOP 迁移

ZIP 扩展最大的变化是可能会在未来引入一个巨大的向后兼容的代码破坏。从 PHP 8 开始，过程 API（所有过程函数）已被弃用！尽管目前不会影响任何代码，但所有 ZIP 扩展函数最终将从语言中移除。

*最佳实践*是将任何`ZIP`扩展的过程代码迁移到使用`ZipArchive`类的 OOP API。以下代码示例说明了如何从过程代码迁移到对象代码，打开一个`test.zip`文件并生成条目列表：

```php
// /repo/ch07/php7_zip_functions.php
$fn  = __DIR__ . '/includes/test.zip';
$zip = zip_open($fn);
$cnt = 0;
if (!is_resource($zip)) exit('Unable to open zip file');
while ($entry = zip_read($zip)) {
    echo zip_entry_name($entry) . "\n";
    $cnt++;
}
echo "Total Entries: $cnt\n";
```

以下是在 PHP 7 中运行的输出：

```php
root@php8_tips_php7 [ /repo/ch07 ]# 
php php7_zip_functions.php 
ch07/includes/
ch07/includes/test.zip
ch07/includes/tree.xml
ch07/includes/test.png
ch07/includes/kitten.jpg
ch07/includes/reflection.html
ch07/php7_ext_is_resource.php
ch07/php7_gd_is_resource.php
... not all entries shown ...
ch07/php8_simple_xml.php
ch07/php8_xml_writer.php
ch07/php8_zip_oop.php
Total Entries: 27
```

从上面的输出可以看出，一共找到了`27`个条目。（还要注意并非所有 ZIP 文件条目都显示。）然而，如果我们在 PHP 8 中尝试相同的代码示例，结果会大不相同，如下所示：

```php
root@php8_tips_php8 [ /repo/ch07 ]# 
php php7_zip_functions.php 
PHP Deprecated:  Function zip_open() is deprecated in /repo/ch07/php7_zip_functions.php on line 5
PHP Deprecated:  Function zip_read() is deprecated in /repo/ch07/php7_zip_functions.php on line 8
PHP Deprecated:  Function zip_entry_name() is deprecated in /repo/ch07/php7_zip_functions.php on line 9
ch07/includes/
Deprecated: Function zip_entry_name() is deprecated in /repo/ch07/php7_zip_functions.php on line 9
... not all entries shown ...
ch07/php8_zip_oop.php
PHP Deprecated:  Function zip_read() is deprecated in /repo/ch07/php7_zip_functions.php on line 8
Total Entries: 27
```

从上面的 PHP 8 输出可以看出，代码示例可以工作，但会发出一系列弃用的`Notice`消息。

以下是您需要在 PHP 8 中编写相同代码示例的方式：

```php
// /repo/ch07/php8_zip_oop.php
$fn  = __DIR__ . '/includes/test.zip';
$obj = new ZipArchive();
$res = $obj->open($fn);
if ($res !== TRUE) exit('Unable to open zip file');
for ($i = 0; $entry = $obj->statIndex($i); $i++) {
    echo $entry['name'] . "\n";
}
echo "Total Entries: $i\n";
```

输出（未显示）与之前的示例完全相同。有趣的是，重写的示例在 PHP 7 中也可以工作！还值得注意的是，在 PHP 8 中，您可以使用`ZipArchive::count()`来获取总条目数（每个目录）。您可能还注意到，在 PHP 8 中，您不能再使用`is_resource()`来检查 ZIP 存档是否正确打开。

### 新的 ZipArchive 类方法

除了从资源到对象的迁移，`ZipArchive`类还进行了一些改进。其中一个改进是添加了以下新方法：

+   `setMtimeName()`方法

+   `setMtimeIndex()`

+   `registerProgressCallback()`

+   `registerCancelCallback()`

+   `replaceFile()`

+   `isCompressionMethodSupported()`

+   `isEncryptionMethodSupported()`

方法名称不言自明。`Mtime`指的是**修改时间**。

### `addGlob()`和`addPattern()`的新选项

`ZipArchive::addGlob()`和`ZipArchive::addPattern()`方法有一组新的选项。这两种方法都用于向存档中添加文件。不同之处在于`addGlob()`使用与核心 PHP 的`glob()`命令相同的文件模式，而`addPattern()`使用正则表达式过滤文件。这里总结了一组新的选项：

+   `flags`：让您使用*位运算符*组合适当的类常量

+   `comp_method`：使用任何`ZipArchive::CM_*`常量作为参数指定压缩方法

+   `comp_flags`：使用所需的`ZipArchive::FL_*`常量来指定压缩标志

+   `enc_method`：允许您指定所需的字符编码（使用任何 `ZipArchive::FL_ENC_*` 标志）

+   `enc_password`：允许您指定 ZIP 存档的加密密码（如果设置了）

在这里还值得一提的是，在 PHP 8 之前的 `remove_path` 选项必须是一个有效的目录路径。从 PHP 8 开始，这个选项是一个简单的字符串，表示要删除的字符。这使您能够删除文件名前缀以及不需要的目录路径。

虽然我们仍在研究选项，值得注意的是添加了两个新的编码方法类常量：`ZipArchive::EM_UNKNOWN` 和 `ZipArchive::EM_TRAD_PKWARE`。此外，添加了一个新的 `lastId` 属性，以便您能够确定最后一个 ZIP 存档条目的索引值。

### 其他 ZipArchive 方法的更改

除了前面提到的更改之外，PHP 8 中还有一些其他 `ZipArchive` 方法已经改变。在本节中，我们总结了其他 `ZipArchive` 方法的更改，如下：

+   `ZipArchive::extractTo()` 以前使用当前日期和时间作为修改时间。从 PHP 8 开始，这个方法恢复了原始文件的修改时间。

+   `ZipArchive::getStatusString()` 在调用 `ZipArchive::close()` 后仍然返回结果。

+   `ZipArchive::addEmptyDir()`、`ZipArchive::addFile()` 和 `ZipArchive::addFromString()` 方法都有一个新的 `flags` 参数。您可以使用任何适当的 `ZipArchive::FL_*` 类常量，并使用位运算符进行组合。

+   `ZipArchive::open()` 现在可以打开一个空的（零字节）文件。

现在您已经了解了引入到 `ZIP` 扩展中的更改和改进，让我们来看看正则表达式领域的更改。

## 检查 PCRE 扩展的更改

**PCRE** 扩展包含了一些设计用于使用 *正则表达式* 进行模式匹配的函数。术语 *regular* *expression* 通常被缩写为 *regex*。**regex** 是描述另一个字符串的字符串。以下是 `PCRE` 扩展中需要注意的一些更改。

模式中的无效转义序列不再被解释为文字。过去，您可以使用 `X` 修饰符；然而，在 PHP 8 中，该修饰符被忽略。令人高兴的是，为了帮助您处理内部 PCRE 模式分析错误，添加了一个新的 `preg_last_error_msg()` 函数，当遇到 PCRE 错误时返回一个人类可读的消息。

`preg_last_error()` 函数允许您确定在模式分析期间是否发生了内部 PCRE 错误。然而，这个函数只返回一个整数。在 PHP 8 之前，开发人员需要查找代码并找出实际的错误。

提示

`preg_last_error()` 返回的错误代码列表可以在这里找到：

[`www.php.net/manual/en/function.preg-last-error.php#refsect1-function.preg-last-error-returnvalues`](https://www.php.net/manual/en/function.preg-last-error.php#refsect1-function.preg-last-error-returnvalues)

接下来是一个简短的代码示例，说明了前面提到的问题。以下是导致这一问题的步骤：

1.  首先，我们定义一个执行匹配并检查是否发生任何错误的函数，如下：

```php
$pregTest = function ($pattern, $string) {
    $result  = preg_match($pattern, $string);
    $lastErr = preg_last_error();
    if ($lastErr == PREG_NO_ERROR) {
        $msg = 'RESULT: ';
        $msg .= ($result) ? 'MATCH' : 'NO MATCH';
    } else {
        $msg = 'ERROR : ';
        if (function_exists('preg_last_error_msg'))
            $msg .= preg_last_error_msg();
        else
            $msg .= $lastErr;
    }
    return "$msg\n";
};
```

1.  然后我们创建一个故意包含 `\8+` 无效转义序列的模式，如下：

```php
$pattern = '/\8+/';
$string  = 'test 8';
echo $pregTest($pattern, $string);
```

1.  接下来，我们定义一个故意导致 PCRE 超出回溯限制的模式，如下：

```php
$pattern = '/(?:\D+|<\d+>)*[!?]/';
$string  = 'test ';
echo $pregTest($pattern, $string);
```

以下是 PHP 7.1 中的输出：

```php
root@php8_tips_php7 [ /repo/ch07 ]# php php7_pcre.php 
RESULT: MATCH
ERROR : 2
```

从前面的输出中可以看到，无效的模式被视为文字值 8。因为 8 存在于字符串中，所以被认为找到了匹配项。至于第二个模式，回溯限制被超出；然而，PHP 7.1 无法报告这个问题，迫使您自行查找。

在 PHP 8 中的输出是完全不同的，如下所示：

```php
root@php8_tips_php8 [ /repo/ch07 ]# php php7_pcre.php 
PHP Warning:  preg_match(): Compilation failed: reference to non-existent subpattern at offset 1 in /repo/ch07/php7_pcre.php on line 5
ERROR : Internal error
ERROR : Backtrack limit exhausted
```

从前面的输出中可以看到，PHP 8 产生了一个 `Warning` 消息。您还可以看到 `preg_last_error_msg()` 产生了一个有用的消息。现在让我们来看看 **Internationalization** (**Intl**) 扩展。

## 处理 Intl 扩展的变化

**Intl 扩展**由几个类组成，处理可能根据区域设置而变化的一些应用方面。各种类处理国际化数字和货币格式化、文本解析、日历生成、时间和日期格式化以及字符集转换等任务。

PHP 8 中引入到 Intl 扩展的主要更改是以下新的日期格式：

+   `IntlDateFormatter::RELATIVE_FULL`

+   `IntlDateFormatter::RELATIVE_LONG`

+   `IntlDateFormatter::RELATIVE_MEDIUM`

+   `IntlDateFormatter::RELATIVE_SHORT`

接下来是一个代码示例，显示了新的格式。以下是导致这一步的步骤：

1.  首先，我们定义一个`DateTime`实例和一个包含新格式代码的数组，如下所示：

```php
$dt = new DateTime('tomorrow');
$pt = [IntlDateFormatter::RELATIVE_FULL,
    IntlDateFormatter::RELATIVE_LONG,
    IntlDateFormatter::RELATIVE_MEDIUM,
    IntlDateFormatter::RELATIVE_SHORT
];
```

1.  然后我们循环遍历格式并输出结果，如下所示：

```php
foreach ($pt as $fmt) 
    echo IntlDateFormatter::formatObject($dt, $fmt)."\n";
```

这个例子在 PHP 7 中不起作用。以下是 PHP 8 的输出：

```php
root@php8_tips_php8 [ /repo/ch07 ]# 
php php8_intl_date_fmt.php 
tomorrow at 12:00:00 AM Coordinated Universal Time
tomorrow at 12:00:00 AM UTC
tomorrow, 12:00:00 AM
tomorrow, 12:00 AM
```

正如你所看到的，新的相对日期格式运行得相当不错！现在我们简要地回到`cURL`扩展。

## 了解 cURL 扩展的变化

`cURL`扩展利用`libcurl`（http://curl.haxx.se/）提供强大和高效的 HTTP 客户端功能。在 PHP 8 中，你必须在服务器的操作系统上安装版本为 7.29（或更高版本）的`libcurl`。

PHP 8 中的另一个不同之处是这个扩展现在使用对象而不是资源。这个变化在本章前面已经描述过，在*表 7.1*，*PHP 8 资源到对象的迁移*中。在*潜在的涉及 is_resource()的代码中断*部分展示了一个例子。这个变化的一个副作用是任何`curl*close()`函数都是多余的，因为当对象未设置或者超出范围时连接会被关闭。

现在让我们来看看`COM`扩展的变化。

## 审查 COM 扩展的变化

**组件对象模型**（**COM**）是一个仅适用于 Windows 的扩展，它使得用一种语言编写的编程代码能够调用和与用任何其他支持 COM 的编程语言编写的代码进行交互。对于计划在 Windows 服务器上运行的 PHP 开发人员来说，这些信息非常重要。

`COM`扩展的最重要的变化是现在自动强制大小写敏感性。因此，你不能再从类型库中导入任何大小写不敏感的常量。此外，你也不能再将`$case_insensitive`作为`FALSE`作为`com_load_typelib()`函数的第二个参数。

在这方面，处理大小写敏感性的`COM`扩展`php.ini`设置已经发生了变化。这些包括以下内容：

+   `com.autoregister_casesensitive`：在 PHP 8 中永久启用。

+   `com.typelib_file`：任何类型库的名称以`#cis`或`#case_insensitive`结尾的不再导致常量被视为大小写不敏感。

一个变化是一个新的`php.ini`设置，`com.dotnet_version`。这个设置用于设置要用于 dotnet 对象的**.NET**版本。我们现在来检查其他值得注意的扩展变化。

## 检查其他扩展的变化

还有一些其他 PHP 扩展的变化值得一提。接下来显示的*表 7.5*总结了这些变化：

![表 7.5 – PHP 8 数据库库要求](img/Figure_7.8_B16231.jpg)

表 7.5 – PHP 8 数据库库要求

现在你对 PHP 8 中扩展的变化有了一个概念。这就结束了本章。现在，是时候进行总结了！

# 摘要

在本章中，你学到的最重要的概念之一是从资源向对象的一般趋势。你学会了在本章涵盖的各种 PHP 扩展中注意到这种趋势的地方，以及如何开发解决方案来避免依赖资源的代码中出现问题。你还学会了如何检测和开发代码来解决 XML 扩展中的变化，特别是在`SimpleXML`和`XMLWriter`扩展中。

本章还涵盖了一个重要的扩展，即`mbstring`扩展，其中有重大变化。您学会了检测依赖于已更改的`mbstring`功能的代码。正如您所了解的，`mbstring`扩展的更改在很大程度上反映了对等核心 PHP 字符串函数的更改。

您还了解了`GD`、`Reflection`和`ZIP`扩展的重大变化。在本章中，您还了解了对一些数据库扩展的更改，以及需要注意的其他扩展变化。总的来说，通过阅读本章并学习示例，您现在更有能力在进行 PHP 8 升级后防止应用程序出现故障。

在下一章中，您将了解到在 PHP 8 中已被弃用或移除的功能。
