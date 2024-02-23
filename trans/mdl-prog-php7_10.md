# 第十章。构建运输模块

运输模块，以及“支付”模块，为我们的网店提供了进一步的销售功能的基础。当我们到达即将到来的“销售”模块的结账过程时，它将使我们能够选择运输方式。与“支付”类似，`shipment`可能是静态的和动态的。静态可能意味着固定的定价值，甚至是通过一些简单条件计算出来的值，动态通常意味着与外部 API 服务的连接。

在本章中，我们将接触到两种类型，并看看如何为实施`shipment`模块设置基本结构。

在本章中，我们将涵盖`shipment`模块的以下主题：

+   要求

+   依赖关系

+   实施

+   单元测试

+   功能测试

# 要求

应用要求，在第四章中定义，*模块化网店应用的需求规范*，并没有给出我们需要实施的运输方式的具体信息。因此，在本章中，我们将开发两种运输方式：动态费率运输和固定费率运输。动态费率运输用作将运输方式连接到真实运输处理器（如 UPS、FedEx 等）的方式。但是，它实际上不会连接到任何外部 API。

理想情况下，我们希望通过类似以下的接口来实现：

```php
namespace Foggyline\SalesBundle\Interface;

interface Shipment
{
  function getInfo($street, $city, $country, $postcode, $amount, $qty);
  function process($street, $city, $country, $postcode, $amount, $qty);
}
```

然后，`getInfo`方法可以用于获取给定订单信息的可用交付选项，而处理方法将处理所选的交付选项。例如，我们可能会有一个 API 返回“当天送货（$9.99）”和“标准送货（$4.99）”作为动态费率运输方式的交付选项。

拥有这样一个运输接口将要求拥有我们尚未开发的`SalesBundle`模块。因此，我们将继续使用 Symfony 控制器处理过程方法和处理`getInfo`方法的服务来处理我们的运输方式。

与上一章中的支付方式一样，我们将通过标记的 Symfony 服务公开我们的`getInfo`方法。我们将用于运输方式的标记是`shipment_method`。在结账过程中，`SalesBundle`模块将获取所有标记为`shipment_method`的服务，并在内部使用它们来获取可用的运输方式列表。

# 依赖关系

我们正在以另一种方式构建模块。也就是说，我们在了解`SalesBundle`模块的任何信息之前就构建它，这是唯一会使用它的模块。考虑到这一点，`shipment`模块不依赖于任何其他模块。但是，构建`SalesBundle`模块然后公开一些`shipment`模块可能使用的接口可能更方便。

# 实施

我们将首先创建一个名为`Foggyline\ShipmentBundle`的新模块。我们将通过运行以下命令来使用控制台完成：

```php
**php bin/console generate:bundle --namespace=Foggyline/ShipmentBundle**

```

该命令触发一个交互式过程，沿途询问我们几个问题，如下所示：

![实施](img/B05460_10_01.jpg)

完成后，文件`app/AppKernel.php`和`app/config/routing.yml`将自动修改。在`$bundles`数组下添加了`AppKernel`类的`registerBundles`方法：

```php
new Foggyline\PaymentBundle\FoggylineShipmentBundle(),
```

`routing.yml`文件已更新为以下条目：

```php
foggyline_payment:
  resource: "@FoggylineShipmentBundle/Resources/config/routing.xml"
  prefix:   /
```

为了避免与核心应用程序代码冲突，我们需要将`prefix: /`更改为`prefix: /shipment/`。

## 创建固定费率运输服务

定价运输服务将提供固定的运输方式，我们的“销售”模块将在结账过程中使用。它的作用是提供运输方式标签、代码、交付选项和处理 URL。

我们将首先在`src/Foggyline/ShipmentBundle/Resources/config/services.xml`文件的`services`元素下定义以下服务：

```php
<service id="foggyline_shipment.dynamicrate_shipment"class="Foggyline\ShipmentBundle\Service\DynamicRateShipment">
  <argument type="service" id="router"/>
  <tag name="shipment_method"/>
</service>
```

这个`service`只接受一个参数：`router`。`tagname`的值设置为`shipment_method`，因为我们的`SalesBundle`模块将根据分配给服务的`shipment_method`标签来寻找运输方法。

我们现在将在`src/Foggyline/ShipmentBundle/Service/FlatRateShipment.php`文件中创建实际的`service`类，如下所示：

```php
namespace Foggyline\ShipmentBundle\Service;
class FlatRateShipment
{
  private $router;

  public function __construct(
    \Symfony\Bundle\FrameworkBundle\Routing\Router $router
  )
  {
    $this->router = $router;
  }

  public function getInfo($street, $city, $country, $postcode, $amount, $qty)
  {
    return array(
      'shipment' => array(
        'title' =>'Foggyline FlatRate Shipment',
        'code' =>'flat_rate',
        'delivery_options' => array(
        'title' =>'Fixed',
        'code' =>'fixed',
        'price' => 9.99
      ),
      'url_process' => $this->router->generate('foggyline_shipment_flat_rate_process'),
    )
  ;
  }
}
```

`getInfo`方法将为我们未来的`SalesBundle`模块提供必要的信息，以便它构建结账过程的`shipment`步骤。它接受一系列参数：`$street`、`$city`、`$country`、`$postcode`、`$amount`和`$qty`。我们可以将这些视为统一的运输接口的一部分。在这种情况下，`delivery_options`返回一个固定值。`url_process`是我们将插入所选运输方法的 URL。我们未来的`SalesBundle`模块将仅仅对这个 URL 进行一个 AJAX POST，期望得到一个成功或错误的 JSON 响应，这与我们想象中使用支付方法的方式非常相似。

## 创建固定费率运输控制器和路由

我们通过向`src/Foggyline/ShipmentBundle/Resources/config/routing.xml`文件添加以下路由定义来编辑它：

```php
<route id="foggyline_shipment_flat_rate_process"path="/flat_rate/process">
  <default key="_controller">FoggylineShipmentBundle:FlatRate:process
  </default>
</route>
```

然后我们创建一个`src/Foggyline/ShipmentBundle/Controller/FlatRateController.php`文件，内容如下：

```php
namespace Foggyline\ShipmentBundle\Controller;

use Symfony\Component\HttpFoundation\JsonResponse;
use Symfony\Component\HttpFoundation\Request;
use Symfony\Bundle\FrameworkBundle\Controller\Controller;

class FlatRateController extends Controller
{
  public function processAction(Request $request)
  {
    // Simulating some transaction id, if any
    $transaction = md5(time() . uniqid());

    return new JsonResponse(array(
      'success' => $transaction
    ));
  }
}
```

我们现在应该能够访问一个 URL，比如`/app_dev.php/shipment/flat_rate/process`，并看到`processAction`的输出。这里给出的实现是虚拟的。对我们来说重要的是，`sales`模块将在其结账过程中，通过`shipment_method`标记的服务的`getInfo`方法推送任何可能的`delivery_options`。这意味着结账过程应该显示固定费用运输作为一个选项。结账的行为将被编码，如果没有选择`shipment`方法，它将阻止结账过程继续进行。当我们到达`SalesBundle`模块时，我们将更详细地讨论这一点。

## 创建动态费率支付服务

除了固定费用运输方法，让我们继续定义另一种动态运输，称为动态费率。

我们将首先在`src/Foggyline/ShipmentBundle/Resources/config/services.xml`文件的`services`元素下定义以下服务：

```php
<service id="foggyline_shipment.dynamicrate_shipment"class="Foggyline\ShipmentBundle\Service\DynamicRateShipment">
  <argument type="service" id="router"/>
  <tag name="shipment_method"/>
</service>
```

在这里定义的`service`只接受一个`router`参数。`tag name`属性与固定费用运输服务相同。

然后我们将创建`src/Foggyline/ShipmentBundle/Service/DynamicRateShipment.php`文件，内容如下：

```php
namespace Foggyline\ShipmentBundle\Service;

class DynamicRateShipment
{
  private $router;

  public function __construct(
    \Symfony\Bundle\FrameworkBundle\Routing\Router $router
  )
  {
    $this->router = $router;
  }

  public function getInfo($street, $city, $country, $postcode, $amount, $qty)
  {
    return array(
      'shipment' => array(
        'title' =>'Foggyline DynamicRate Shipment',
        'code' =>'dynamic_rate_shipment',
        'delivery_options' => $this->getDeliveryOptions($street, $city, $country, $postcode, $amount, $qty),
        'url_process' => $this->router->generate('foggyline_shipment_dynamic_rate_process'),
      )
    );
  }

  public function getDeliveryOptions($street, $city, $country, $postcode, $amount, $qty)
  {
    // Imagine we are hitting the API with: $street, $city, $country, $postcode, $amount, $qty
    return array(
      array(
        'title' =>'Same day delivery',
        'code' =>'dynamic_rate_sdd',
        'price' => 9.99
      ),
      array(
        'title' =>'Standard delivery',
        'code' =>'dynamic_rate_sd',
        'price' => 4.99
      ),
    );
  }
}
```

与固定费用运输不同，这里`getInfo`方法的`delivery_options`键是由`getDeliveryOptions`方法的响应构建的。该方法是服务内部的，不被想象为公开或作为接口的一部分。我们可以很容易地想象在其中进行一些 API 调用，以便为我们的动态运输方法获取计算费率。

## 创建动态费率运输控制器和路由

一旦动态费率运输服务就位，我们就可以为其创建必要的路由。我们将首先在`src/Foggyline/ShipmentBundle/Resources/config/routing.xml`文件中添加以下路由定义：

```php
<route id="foggyline_shipment_dynamic_rate_process" path="/dynamic_rate/process">
  <default key="_controller">FoggylineShipmentBundle:DynamicRate:process
  </default>
</route>
```

然后我们创建`src/Foggyline/ShipmentBundle/Controller/DynamicRateController.php`文件，内容如下：

```php
namespace Foggyline\ShipmentBundle\Controller;

use Foggyline\ShipmentBundle\Entity\DynamicRate;
use Symfony\Component\HttpFoundation\JsonResponse;
use Symfony\Component\HttpFoundation\Request;
use Symfony\Bundle\FrameworkBundle\Controller\Controller;
use Symfony\Component\Form\Extension\Core\Type\ChoiceType;

class DynamicRateController extends Controller
{
  public function processAction(Request $request)
  {
    // Just a dummy string, simulating some transaction id
    $transaction = md5(time() . uniqid());

    if ($transaction) {
      return new JsonResponse(array(
'success' => $transaction
      ));
    }

    return new JsonResponse(array(
      'error' =>'Error occurred while processing DynamicRate shipment.'
    ));
  }
}
```

与固定费率运输类似，这里我们添加了一个简单的虚拟实现过程和方法。传入的`$request`应该包含与服务`getInfo`方法相同的信息，也就是说，它应该有以下参数可用：`$street`、`$city`、`$country`、`$postcode`、`$amount`和`$qty`。方法的响应将在后面的`SalesBundle`模块中使用。我们可以很容易地从这些方法中实现更健壮的功能，但这超出了本章的范围。

# 单元测试

`FoggylineShipmentBundle`模块非常简单。通过提供两个简单的服务和两个简单的控制器，很容易进行测试。

我们将首先在`phpunit.xml.dist`文件的`testsuites`元素下添加以下行：

```php
<directory>src/Foggyline/ShipmentBundle/Tests</directory>
```

有了这个文件，从商店的根目录运行`phpunit`命令应该会检测到我们在`src/Foggyline/ShipmentBundle/Tests/`目录下定义的任何测试。

现在，让我们继续为我们的`FlatRateShipment`服务创建一个测试。我们将创建一个`src/Foggyline/ShipmentBundle/Tests/Service/FlatRateShipmentTest.php`文件，内容如下：

```php
namespace Foggyline\ShipmentBundle\Tests\Service;

use Symfony\Bundle\FrameworkBundle\Test\KernelTestCase;

class FlatRateShipmentTest extends KernelTestCase
{
  private $container;
  private $router;

  private $street = 'Masonic Hill Road';
  private $city = 'Little Rock';
  private $country = 'US';
  private $postcode = 'AR 72201';
  private $amount = 199.99;
  private $qty = 7;

  public function setUp()
  {
    static::bootKernel();
    $this->container = static::$kernel->getContainer();
    $this->router = $this->container->get('router');
  }

  public function testGetInfoViaService()
  {
    $shipment = $this->container->get('foggyline_shipment.flat_rate');

    $info = $shipment->getInfo(
      $this->street, $this->city, $this->country, $this->postcode, $this->amount, $this->qty
    );

    $this->validateGetInfoResponse($info);
  }

  public function testGetInfoViaClass()
  {
    $shipment = new \Foggyline\ShipmentBundle\Service\FlatRateShipment($this->router);

    $info = $shipment->getInfo(
      $this->street, $this->city, $this->country, $this->postcode, $this->amount, $this->qty
    );

    $this->validateGetInfoResponse($info);
  }

  public function validateGetInfoResponse($info)
  {
    $this->assertNotEmpty($info);
    $this->assertNotEmpty($info['shipment']['title']);
    $this->assertNotEmpty($info['shipment']['code']);
    $this->assertNotEmpty($info['shipment']['delivery_options']);
    $this->assertNotEmpty($info['shipment']['url_process']);
  }
}
```

这里运行了两个简单的测试。一个是检查我们是否可以通过容器实例化一个服务，另一个是检查我们是否可以直接这样做。一旦实例化，我们只需调用服务的`getInfo`方法，向其传递一个虚拟地址和订单信息。虽然我们实际上并没有在`getInfo`方法中使用这些数据，但我们需要传递一些东西，否则测试将失败。该方法预期返回一个包含在 shipment 键下的几个关键字的响应，最重要的是`title`、`code`、`delivery_options`和`url_process`。

现在，让我们继续为我们的`DynamicRateShipment`服务创建一个测试。我们将创建一个`src/Foggyline/ShipmentBundle/Tests/Service/DynamicRateShipmentTest.php`文件，内容如下：

```php
namespace Foggyline\ShipmentBundle\Tests\Service;

use Symfony\Bundle\FrameworkBundle\Test\KernelTestCase;
class DynamicRateShipmentTest extends KernelTestCase
{
  private $container;
  private $router;

  private $street = 'Masonic Hill Road';
  private $city = 'Little Rock';
  private $country = 'US';
  private $postcode = 'AR 72201';
  private $amount = 199.99;
  private $qty = 7;

  public function setUp()
  {
    static::bootKernel();
    $this->container = static::$kernel->getContainer();
    $this->router = $this->container->get('router');
  }

  public function testGetInfoViaService()
  {
    $shipment = $this->container->get('foggyline_shipment.dynamicrate_shipment');
    $info = $shipment->getInfo(
      $this->street, $this->city, $this->country, $this->postcode, $this->amount, $this->qty
    );
    $this->validateGetInfoResponse($info);
  }

  public function testGetInfoViaClass()
  {
    $shipment = new \Foggyline\ShipmentBundle\Service\DynamicRateShipment($this->router);
    $info = $shipment->getInfo(
      $this->street, $this->city, $this->country, $this->postcode, $this->amount, $this->qty
    );

    $this->validateGetInfoResponse($info);
  }

  public function validateGetInfoResponse($info)
  {
    $this->assertNotEmpty($info);
    $this->assertNotEmpty($info['shipment']['title']);
    $this->assertNotEmpty($info['shipment']['code']);

    // Could happen that dynamic rate has none?!
    //$this->assertNotEmpty($info['shipment']['delivery_options']);

    $this->assertNotEmpty($info['shipment']['url_process']);
  }
}
```

这个测试几乎与`FlatRateShipment`服务的测试相同。在这里，我们也有两个简单的测试：一个通过容器获取支付方法，另一个通过类直接获取。不同之处在于我们不再断言`delivery_options`的存在。这是因为真实的 API 请求可能不会根据给定的地址和订单信息返回任何交付选项。

# 功能测试

我们整个模块只有两个控制器类，我们要测试它们的响应。我们要确保`FlatRateController`和`DynamicRateController`类的`process`方法是可访问且有效的。

首先，我们将创建一个`src/Foggyline/ShipmentBundle/Tests/Controller/FlatRateControllerTest.php`文件，内容如下：

```php
namespace Foggyline\ShipmentBundle\Tests\Controller;

use Symfony\Bundle\FrameworkBundle\Test\WebTestCase;
class FlatRateControllerTest extends WebTestCase
{
  private $client;
  private $router;

  public function setUp()
  {
    $this->client = static::createClient();
    $this->router = $this->client->getContainer()->get('router');
  }

  public function testProcessAction()
  {
    $this->client->request('GET', $this->router->generate('foggyline_shipment_flat_rate_process'));
    $this->assertSame(200, $this->client->getResponse()->getStatusCode());
    $this->assertSame('application/json', $this->client->getResponse()->headers->get('Content-Type'));
    $this->assertContains('success', $this->client->getResponse()->getContent());
    $this->assertNotEmpty($this->client->getResponse()->getContent());
  }
}
```

然后，我们将创建一个`src/Foggyline/ShipmentBundle/Tests/Controller/DynamicRateControllerTest.php`文件，内容如下：

```php
namespace Foggyline\ShipmentBundle\Tests\Controller;

use Symfony\Bundle\FrameworkBundle\Test\WebTestCase;

class DynamicRateControllerTest extends WebTestCase
{
  private $client;
  private $router;

  public function setUp()
  {
    $this->client = static::createClient();
    $this->router = $this->client->getContainer()->get('router');
  }

  public function testProcessAction()
  {
    $this->client->request('GET', $this->router->generate('foggyline_shipment_dynamic_rate_process'));
    $this->assertSame(200, $this->client->getResponse()->getStatusCode());
    $this->assertSame('application/json', $this->client->getResponse()->headers->get('Content-Type'));
    $this->assertContains('success', $this->client->getResponse()->getContent());
    $this->assertNotEmpty($this->client->getResponse()->getContent());
  }
}
```

这两个测试几乎是相同的。它们包含一个单一过程操作方法的测试。目前，控制器过程操作方法只是返回一个固定的成功 JSON 响应。我们可以很容易地扩展它以返回不仅仅是一个固定的响应，并且可以伴随着更健壮的功能测试。

# 总结

在本章中，我们构建了一个具有两种运输方法的`shipment`模块。每种运输方法都提供了可用的交付选项。固定费率运输方法在其交付选项下只有一个固定值，而动态费率方法从`getDeliveryOptions`方法中获取其值。我们可以很容易地在`getDeliveryOptions`中嵌入一个真实的运输 API，以提供真正动态的运输选项。

显然，我们在这里缺少官方接口，就像我们在支付方法中一样。然而，这是我们可以随时回来并在我们最终的`final`模块中重构的内容。

与支付方式类似，这里的想法是创建一个最小的结构，展示如何开发一个简单的装运模块以供进一步定制。通过使用`shipment_methodservice`标签，我们有效地为未来的“销售”模块暴露了装运方法。

在接下来的章节中，我们将构建一个“销售”模块，最终将利用我们的“支付”和“装运”模块。
