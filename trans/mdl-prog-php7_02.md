# 第二章。GoF 设计模式

有一些因素使得一个优秀的软件开发者。设计模式的知识和使用就是其中之一。设计模式使开发者能够使用众所周知的名称来进行各种软件交互。无论是 PHP、Python、C#、Ruby 还是其他任何语言的开发者，设计模式都为经常发生的软件问题提供了与语言无关的解决方案。

设计模式的概念于 1994 年出现，作为《可重用面向对象软件的元素》一书的一部分。该书详细介绍了 23 种不同的设计模式，由 Erich Gamma、Richard Helm、Ralph Johnson 和 John Vlissides 四位作者撰写。这些作者通常被称为**四人帮**（**GoF**），所提出的设计模式有时被称为 GoF 设计模式。如今，两十多年后，设计可扩展、可重用、可维护和可适应的软件几乎不可能不将设计模式作为实现的一部分。

本章将介绍三种设计模式：

+   创造性

+   结构性

+   行为

在本章中，我们不会深入研究每一个模式的理论，因为单独讨论这些内容就是一本完整的书。接下来，我们将更多地关注每种模式的简单 PHP 实现示例，以便更直观地了解事物。

# 创建型模式

创建型模式，顾名思义，为我们创建*对象*，因此我们不必直接实例化它们。实现创建模式为我们的应用程序提供了一定程度的灵活性，应用程序本身可以决定在特定时间实例化哪些对象。以下是我们归类为创建型模式的模式列表：

+   抽象工厂模式

+   建造者模式

+   工厂方法模式

+   原型模式

+   单例模式

### 注意

有关创建型设计模式的更多信息，请参见[`en.wikipedia.org/wiki/Creational_pattern`](https://en.wikipedia.org/wiki/Creational_pattern)。

## 抽象工厂模式

构建可移植应用程序需要很高的依赖封装级别。抽象工厂通过*抽象化相关或依赖对象的创建*来实现这一点。客户端永远不会直接创建这些平台对象，工厂会为他们创建，使得可以在不改变使用它们的代码的情况下交换具体实现，甚至在运行时。

以下是可能的抽象工厂模式实现示例：

```php
interface Button {
    public function render();
}

interface GUIFactory {
    public function createButton();
}

class SubmitButton implements Button {
    public function render() {
        echo 'Render Submit Button';
    }
}

class ResetButton implements Button {
    public function render() {
        echo 'Render Reset Button';
    }
}

class SubmitFactory implements GUIFactory {
    public function createButton() {
        return new SubmitButton();
    }
}

class ResetFactory implements GUIFactory {
    public function createButton() {
        return new ResetButton();
    }
}

// Client
$submitFactory = new SubmitFactory();
$button = $submitFactory->createButton();
$button->render();

$resetFactory = new ResetFactory();
$button = $resetFactory->createButton();
$button->render();
```

我们首先创建了一个接口`Button`，然后由我们的`SubmitButton`和`ResetButton`具体类来实现。`GUIFactory`和`ResetFactory`实现了`GUIFactory`接口，该接口指定了`createButton`方法。然后客户端简单地实例化工厂并调用`createButton`，返回一个适当的按钮实例，我们称之为`render`方法。

## 建造者模式

建造者模式将复杂对象的构建与其表示分离，使得相同的构建过程可以创建不同的表示。虽然一些创造模式在一次调用中构造产品，但建造者模式在主管的控制下逐步进行。

以下是建造者模式实现的示例：

```php
class Car {
    public function getWheels() {
        /* implementation... */
    }

    public function setWheels($wheels) {
        /* implementation... */
    }

    public function getColour($colour) {
        /* implementation... */
    }

    public function setColour() {
        /* implementation... */
    }
}

interface CarBuilderInterface {
    public function setColour($colour);
    public function setWheels($wheels);
    public function getResult();
}

class CarBuilder implements CarBuilderInterface {
    private $car;

    public function __construct() {
        $this->car = new Car();
    }

    public function setColour($colour) {
        $this->car->setColour($colour);
        return $this;
    }

    public function setWheels($wheels) {
        $this->car->setWheels($wheels);
        return $this;
    }

    public function getResult() {
        return $this->car;
    }
}

class CarBuildDirector {
    private $builder;

    public function __construct(CarBuilder $builder) {
        $this->builder = $builder;
    }

    public function build() {
        $this->builder->setColour('Red');
        $this->builder->setWheels(4);

        return $this;
    }

    public function getCar() {
        return $this->builder->getResult();
    }
}

// Client
$carBuilder = new CarBuilder();
$carBuildDirector = new CarBuildDirector($carBuilder);
$car = $carBuildDirector->build()->getCar();
```

我们首先创建了一个具体的`Car`类，其中包含定义汽车一些基本特征的几种方法。然后我们创建了一个`CarBuilderInterface`，它将控制其中一些特征并获得最终结果（`car`）。具体类`CarBuilder`然后实现了`CarBuilderInterface`，接着是具体的`CarBuildDirector`类，它定义了构建和`getCar`方法。客户端只需实例化一个新的`CarBuilder`实例，并将其作为构造函数参数传递给一个新的`CarBuildDirector`实例。最后，我们调用`CarBuildDirector`的`build`和`getCar`方法来获得实际的汽车`Car`实例。

## 工厂方法模式

`工厂`方法模式处理创建对象的问题，而无需指定将要创建的对象的确切类。

以下是工厂方法模式实现的示例：

```php
interface Product {
    public function getType();
}

interface ProductFactory {
    public function makeProduct();
}

class SimpleProduct implements Product {
    public function getType() {
        return 'SimpleProduct';
    }
}

class SimpleProductFactory implements ProductFactory {
    public function makeProduct() {
        return new SimpleProduct();
    }
}

/* Client */
$factory = new SimpleProductFactory();
$product = $factory->makeProduct();
echo $product->getType(); //outputs: SimpleProduct
```

我们首先创建了一个`ProductFactory`和`Product`接口。`SimpleProductFactory`实现了`ProductFactory`并通过其`makeProduct`方法返回新的`product`实例。`SimpleProduct`类实现了`Product`，并返回产品类型。最后，客户端创建了`SimpleProductFactory`的实例，并在其上调用`makeProduct`方法。`makeProduct`返回`Product`的实例，其`getType`方法返回`SimpleProduct`字符串。

## 原型模式

原型模式通过克隆来复制其他对象。这意味着我们不是使用`new`关键字来实例化新对象。PHP 提供了一个`clone`关键字，它可以对对象进行浅复制，从而提供了非常直接的原型模式实现。浅复制不会复制引用，只会将值复制到新对象。我们还可以利用我们的类上的魔术`__clone`方法来实现更健壮的克隆行为。

以下是原型模式实现的示例：

```php
class User {
    public $name;
    public $email;
}

class Employee extends User {
    public function __construct() {
        $this->name = 'Johhn Doe';
        $this->email = 'john.doe@fake.mail';
    }

    public function info() {
        return sprintf('%s, %s', $this->name, $this->email);
    }

    public function __clone() {
        /* additional changes for (after)clone behavior? */
    }
}

$employee = new Employee();
echo $employee->info();

$director = clone $employee;
$director->name = 'Jane Doe';
$director->email = 'jane.doe@fake.mail';
echo $director->info(); //outputs: Jane Doe, jane.doe@fake.mail
```

我们首先创建了一个简单的`User`类。然后`Employee`类扩展了`User`类，并在其构造函数中设置了`name`和`email`。客户端通过`new`关键字实例化了`Employee`，并将其克隆到`director`变量中。`$director`变量现在是一个新实例，不是通过`new`关键字创建的，而是通过克隆使用`clone`关键字创建的。在`$director`上更改`name`和`email`不会影响`$employee`。

## 单例模式

单例模式的目的是限制类的实例化为*单个*对象。它通过在类中创建一个方法来实现，如果不存在对象实例，则创建该类的新实例。如果对象实例已经存在，则该方法简单地返回对现有对象的引用。

以下是单例模式实现的示例：

```php
class Logger {
    private static $instance;

    public static function getInstance() {
        if (!isset(self::$instance)) {
            self::$instance = new self;
        }

        return self::$instance;
    }

    public function logNotice($msg) {
        return 'logNotice: ' . $msg;
    }

    public function logWarning($msg) {
        return 'logWarning: ' . $msg;
    }

    public function logError($msg) {
        return 'logError: ' . $msg;
    }
}

// Client
echo Logger::getInstance()->logNotice('test-notice');
echo Logger::getInstance()->logWarning('test-warning');
echo Logger::getInstance()->logError('test-error');
// Outputs:
// logNotice: test-notice
// logWarning: test-warning
// logError: test-error
```

我们首先创建了一个带有静态`$instance`成员和`getInstance`方法的`Logger`类，该方法始终返回类的单个实例。然后我们添加了一些示例方法，以演示客户端在单个实例上执行各种方法。

# 结构模式

结构模式处理类和对象的组合。使用接口或抽象类和方法，它们定义了组合对象的方式，从而获得新功能。以下是我们将作为结构模式进行分类的模式列表：

+   适配器模式

+   桥接模式

+   组合模式

+   装饰器

+   外观模式

+   享元模式

+   代理

### 注意

有关结构设计模式的更多信息，请参阅[`en.wikipedia.org/wiki/Structural_pattern`](https://en.wikipedia.org/wiki/Structural_pattern)。

## 适配器模式

适配器模式允许使用现有类的接口来自另一个接口，基本上通过将一个类的接口转换为另一个类期望的接口，帮助两个不兼容的接口一起工作。

以下是适配器模式实现的示例：

```php
class Stripe {
    public function capturePayment($amount) {
        /* Implementation... */
    }

    public function authorizeOnlyPayment($amount) {
        /* Implementation... */
    }

    public function cancelAmount($amount) {
        /* Implementation... */
    }
}

interface PaymentService {
    public function capture($amount);
    public function authorize($amount);
    public function cancel($amount);
}

class StripePaymentServiceAdapter implements PaymentService {
    private $stripe;

    public function __construct(Stripe $stripe) {
        $this->stripe = $stripe;
    }

    public function capture($amount) {
        $this->stripe->capturePayment($amount);
    }

    public function authorize($amount) {
        $this->stripe->authorizeOnlyPayment($amount);
    }

    public function cancel($amount) {
        $this->stripe->cancelAmount($amount);
    }
}

// Client
$stripe = new StripePaymentServiceAdapter(new Stripe());
$stripe->authorize(49.99);
$stripe->capture(19.99);
$stripe->cancel(9.99);
```

我们首先创建了一个具体的`Stripe`类。然后定义了`PaymentService`接口，其中包含一些基本的支付处理方法。`StripePaymentServiceAdapter`实现了`PaymentService`接口，提供了支付处理方法的具体实现。最后，客户端实例化了`StripePaymentServiceAdapter`并执行了支付处理方法。

## 桥接模式

桥接模式用于当我们想要将类或抽象与其实现解耦时，允许它们独立变化。当类和其实现经常变化时，这是很有用的。

以下是桥接模式实现的示例：

```php
interface MailerInterface {
    public function setSender(MessagingInterface $sender);
    public function send($body);
}

abstract class Mailer implements MailerInterface {
    protected $sender;

    public function setSender(MessagingInterface $sender) {
        $this->sender = $sender;
    }
}

class PHPMailer extends Mailer {
    public function send($body) {
        $body .= "\n\n Sent from a phpmailer.";
        return $this->sender->send($body);
    }
}

class SwiftMailer extends Mailer {
    public function send($body) {
        $body .= "\n\n Sent from a SwiftMailer.";
        return $this->sender->send($body);
    }
}

interface MessagingInterface {
    public function send($body);
}

class TextMessage implements MessagingInterface {
    public function send($body) {
        echo 'TextMessage > send > $body: ' . $body;
    }
}

class HtmlMessage implements MessagingInterface {
    public function send($body) {
        echo 'HtmlMessage > send > $body: ' . $body;
    }
}

// Client
$phpmailer = new PHPMailer();
$phpmailer->setSender(new TextMessage());
$phpmailer->send('Hi!');

$swiftMailer = new SwiftMailer();
$swiftMailer->setSender(new HtmlMessage());
$swiftMailer->send('Hello!');
```

我们首先创建了一个`MailerInterface`。具体的`Mailer`类然后实现了`MailerInterface`，为`PHPMailer`和`SwiftMailer`提供了一个基类。然后我们定义了`MessagingInterface`，它由`TextMessage`和`HtmlMessage`类实现。最后，客户端实例化`PHPMailer`和`SwiftMailer`，在调用`send`方法之前传递`TextMessage`和`HtmlMessage`的实例。

## 组合模式

组合模式是关于将对象的层次结构视为单个对象，通过一个公共接口。对象被组合成三个结构，客户端对底层结构的更改毫不知情，因为它只消耗公共接口。

以下是组合模式实现的示例：

```php
interface Graphic {
    public function draw();
}

class CompositeGraphic implements Graphic {
    private $graphics = array();

    public function add($graphic) {
        $objId = spl_object_hash($graphic);
        $this->graphics[$objId] = $graphic;
    }

    public function remove($graphic) {
        $objId = spl_object_hash($graphic);
        unset($this->graphics[$objId]);
    }

    public function draw() {
        foreach ($this->graphics as $graphic) {
            $graphic->draw();
        }
    }
}

class Circle implements Graphic {
    public function draw()
    {
        echo 'draw-circle';
    }
}

class Square implements Graphic {
    public function draw() {
        echo 'draw-square';
    }
}

class Triangle implements Graphic {
    public function draw() {
        echo 'draw-triangle';
    }
}

$circle = new Circle();
$square = new Square();
$triangle = new Triangle();

$compositeObj1 = new CompositeGraphic();
$compositeObj1->add($circle);
$compositeObj1->add($triangle);
$compositeObj1->draw();

$compositeObj2 = new CompositeGraphic();
$compositeObj2->add($circle);
$compositeObj2->add($square);
$compositeObj2->add($triangle);
$compositeObj2->remove($circle);
$compositeObj2->draw();
```

我们首先创建了一个`Graphic`接口。然后创建了`CompositeGraphic`、`Circle`、`Square`和`Triangle`，它们都实现了`Graphic`接口。除了实现`Graphic`接口的`draw`方法之外，`CompositeGraphic`还添加了另外两个方法，用于跟踪添加到其中的图形的内部集合。然后客户端实例化所有这些`Graphic`类，将它们全部添加到`CompositeGraphic`中，然后调用`draw`方法。

## 装饰器模式

装饰器模式允许向单个对象实例添加行为，而不影响同一类的其他实例的行为。我们可以定义多个装饰器，每个装饰器都添加新功能。

以下是装饰器模式实现的示例：

```php
interface LoggerInterface {
    public function log($message);
}

class Logger implements LoggerInterface {
    public function log($message) {
        file_put_contents('app.log', $message, FILE_APPEND);
    }
}

abstract class LoggerDecorator implements LoggerInterface {
    protected $logger;

    public function __construct(Logger $logger) {
        $this->logger = $logger;
    }

    abstract public function log($message);
}

class ErrorLoggerDecorator extends LoggerDecorator {
    public function log($message) {
        $this->logger->log('ERROR: ' . $message);
    }

}

class WarningLoggerDecorator extends LoggerDecorator {
    public function log($message) {
        $this->logger->log('WARNING: ' . $message);
    }
}

class NoticeLoggerDecorator extends LoggerDecorator {
    public function log($message) {
        $this->logger->log('NOTICE: ' . $message);
    }
}

$logger = new Logger();
$logger->log('Resource not found.');

$logger = new Logger();
$logger = new ErrorLoggerDecorator($logger);
$logger->log('Invalid user role.');

$logger = new Logger();
$logger = new WarningLoggerDecorator($logger);
$logger->log('Missing address parameters.');

$logger = new Logger();
$logger = new NoticeLoggerDecorator($logger);
$logger->log('Incorrect type provided.');
```

我们首先创建了一个`LoggerInterface`，其中包含一个简单的`log`方法。然后定义了`Logger`和`LoggerDecorator`，它们都实现了`LoggerInterface`。然后是`ErrorLoggerDecorator`、`WarningLoggerDecorator`和`NoticeLoggerDecorator`，它们实现了`LoggerDecorator`。最后，客户端部分实例化了`logger`三次，并传递了不同的装饰器。

## 外观模式

外观模式用于通过一个更简单的接口简化大型系统的复杂性。它通过为客户端提供方便的方法来执行大多数常见任务，通过一个单一的包装类来实现。

以下是外观模式实现的示例：

```php
class Product {
    public function getQty() {
        // Implementation
    }
}

class QuickOrderFacade {
    private $product = null;
    private $orderQty = null;

    public function __construct($product, $orderQty) {
        $this->product = $product;
        $this->orderQty = $orderQty;
    }

    public function generateOrder() {
        if ($this->qtyCheck()) {
            $this->addToCart();
            $this->calculateShipping();
            $this->applyDiscount();
            $this->placeOrder();
        }
    }

    private function addToCart() {
        // Implementation...
    }

    private function qtyCheck() {
        if ($this->product->getQty() > $this->orderQty) {
            return true;
        } else {
            return true;
        }
    }

    private function calculateShipping() {
        // Implementation...
    }

    private function applyDiscount() {
        // Implementation...
    }

    private function placeOrder() {
        // Implementation...
    }
}

// Client
$order = new QuickOrderFacade(new Product(), $qty);
$order->generateOrder();
```

我们首先创建了一个`Product`类，其中包含一个`getQty`方法。然后创建了一个`QuickOrderFacade`类，它通过`constructor`接受`product`实例和数量，并进一步提供了`generateOrder`方法，该方法汇总了所有生成订单的操作。最后，客户端实例化了`product`，将其传递给`QuickOrderFacade`的实例，并调用了其上的`generateOrder`。

## 享元模式

享元模式关乎性能和资源的减少，在相似对象之间尽可能共享数据。这意味着相同的类实例在实现中是共享的。当预计会创建大量相同类的实例时，这种方法效果最佳。

以下是享元模式实现的示例：

```php
interface Shape {
    public function draw();
}

class Circle implements Shape {
    private $colour;
    private $radius;

    public function __construct($colour) {
        $this->colour = $colour;
    }

    public function draw() {
        echo sprintf('Colour %s, radius %s.', $this->colour, $this->radius);
    }

    public function setRadius($radius) {
        $this->radius = $radius;
    }
}

class ShapeFactory {
    private $circleMap;

    public function getCircle($colour) {
        if (!isset($this->circleMap[$colour])) {
            $circle = new Circle($colour);
            $this->circleMap[$colour] = $circle;
        }

        return $this->circleMap[$colour];
    }
}

// Client
$shapeFactory = new ShapeFactory();
$circle = $shapeFactory->getCircle('yellow');
$circle->setRadius(10);
$circle->draw();

$shapeFactory = new ShapeFactory();
$circle = $shapeFactory->getCircle('orange');
$circle->setRadius(15);
$circle->draw();

$shapeFactory = new ShapeFactory();
$circle = $shapeFactory->getCircle('yellow');
$circle->setRadius(20);
$circle->draw();
```

我们首先创建了一个`Shape`接口，其中包含一个`draw`方法。然后我们定义了实现`Shape`接口的`Circle`类，接着是`ShapeFactory`类。在`ShapeFactory`中，`getCircle`方法根据`color`选项返回一个新的`Circle`实例。最后，客户端实例化了几个`ShapeFactory`对象，并传入不同的颜色到`getCircle`方法中。

## 代理模式

代理设计模式作为原始对象的接口在后台运行。它可以充当简单的转发包装器，甚至在包装的对象周围提供额外的功能。额外添加的功能示例可能是懒加载或缓存，可以弥补原始对象的资源密集操作。

以下是代理模式实现的示例：

```php
interface ImageInterface {
    public function draw();
}

class Image implements ImageInterface {
    private $file;

    public function __construct($file) {
        $this->file = $file;
        sleep(5); // Imagine resource intensive image load
    }

    public function draw() {
        echo 'image: ' . $this->file;
    }
}

class ProxyImage implements ImageInterface {
    private $image = null;
    private $file;

    public function __construct($file) {
        $this->file = $file;
    }

    public function draw() {
        if (is_null($this->image)) {
            $this->image = new Image($this->file);
        }

        $this->image->draw();
    }
}

// Client
$image = new Image('image.png'); // 5 seconds
$image->draw();

$image = new ProxyImage('image.png'); // 0 seconds
$image->draw();
```

我们首先创建了一个`ImageInterface`，其中包含一个`draw`方法。然后我们定义了`Image`和`ProxyImage`类，它们都扩展了`ImageInterface`。在`Image`类的`__construct`中，我们使用`sleep`方法模拟了**资源密集**的操作。最后，客户端实例化了`Image`和`ProxyImage`，展示了两者之间的执行时间差异。

# 行为模式

行为模式解决了各种对象之间通信的挑战。它们描述了不同对象和类如何相互发送消息以实现事情发生。以下是我们归类为行为模式的模式列表：

+   责任链

+   命令

+   解释器

+   迭代器

+   中介者

+   备忘录

+   观察者

+   状态

+   策略

+   模板方法

+   访问者

## 责任链模式

责任链模式通过以链式方式启用多个对象处理请求，将请求的发送者与接收者解耦。各种类型的处理对象可以动态添加到链中。使用递归组合链允许无限数量的处理对象。

以下是责任链模式实现的示例：

```php
abstract class SocialNotifier {
    private $notifyNext = null;

    public function notifyNext(SocialNotifier $notifier) {
        $this->notifyNext = $notifier;
        return $this->notifyNext;
    }

    final public function push($message) {
        $this->publish($message);

        if ($this->notifyNext !== null) {
            $this->notifyNext->push($message);
        }
    }

    abstract protected function publish($message);
}

class TwitterSocialNotifier extends SocialNotifier {
    public function publish($message) {
        // Implementation...
    }
}

class FacebookSocialNotifier extends SocialNotifier {
    protected function publish($message) {
        // Implementation...
    }
}

class PinterestSocialNotifier extends SocialNotifier {
    protected function publish($message) {
        // Implementation...
    }
}

// Client
$notifier = new TwitterSocialNotifier();

$notifier->notifyNext(new FacebookSocialNotifier())
    ->notifyNext(new PinterestSocialNotifier());

$notifier->push('Awesome new product available!');
```

我们首先创建了一个抽象的`SocialNotifier`类，其中包含抽象方法`publish`，`notifyNext`和`push`方法的实现。然后我们定义了`TwitterSocialNotifier`，`FacebookSocialNotifier`和`PinterestSocialNotifier`，它们都扩展了抽象的`SocialNotifier`。客户端首先实例化了`TwitterSocialNotifier`，然后进行了两次`notifyNext`调用，传递了两种其他`notifier`类型的实例，然后调用了最终的`push`方法。

## 命令模式

命令模式将执行特定操作的对象与知道如何使用它的对象解耦。它通过封装后续执行某个动作所需的所有相关信息来实现。这意味着关于对象、方法名称和方法参数的信息。

以下是命令模式的实现示例：

```php
interface LightBulbCommand {
    public function execute();
}

class LightBulbControl {
    public function turnOn() {
        echo 'LightBulb turnOn';
    }

    public function turnOff() {
        echo 'LightBulb turnOff';
    }
}

class TurnOnLightBulb implements LightBulbCommand {
    private $lightBulbControl;

    public function __construct(LightBulbControl $lightBulbControl) {
        $this->lightBulbControl = $lightBulbControl;
    }

    public function execute() {
        $this->lightBulbControl->turnOn();
    }
}

class TurnOffLightBulb implements LightBulbCommand {
    private $lightBulbControl;

    public function __construct(LightBulbControl $lightBulbControl) {
        $this->lightBulbControl = $lightBulbControl;
    }

    public function execute() {
        $this->lightBulbControl->turnOff();
    }
}

// Client
$command = new TurnOffLightBulb(new LightBulbControl());
$command->execute();
```

我们首先创建了一个`LightBulbCommand`接口。然后我们定义了`LightBulbControl`类，提供了两个简单的`turnOn` / `turnOff`方法。然后我们定义了实现`LightBulbCommand`接口的`TurnOnLightBulb`和`TurnOffLightBulb`类。最后，客户端实例化了`TurnOffLightBulb`对象，并在其上调用了`execute`方法。

## 解释器模式

解释器模式指定了如何评估语言语法或表达式。我们定义了语言语法的表示以及解释器。语言语法的表示使用复合类层次结构，其中规则映射到类。然后解释器使用表示来解释语言中的表达式。

以下是解释器模式实现的示例：

```php
interface MathExpression
{
    public function interpret(array $values);
}

class Variable implements MathExpression {
    private $char;

    public function __construct($char) {
        $this->char = $char;
    }

    public function interpret(array $values) {
        return $values[$this->char];
    }
}

class Literal implements MathExpression {
    private $value;

    public function __construct($value) {
        $this->value = $value;
    }

    public function interpret(array $values) {
        return $this->value;
    }
}

class Sum implements MathExpression {
    private $x;
    private $y;

    public function __construct(MathExpression $x, MathExpression $y) {
        $this->x = $x;
        $this->y = $y;
    }

    public function interpret(array $values) {
        return $this->x->interpret($values) + $this->y->interpret($values);
    }
}

class Product implements MathExpression {
    private $x;
    private $y;

    public function __construct(MathExpression $x, MathExpression $y) {
        $this->x = $x;
        $this->y = $y;
    }

    public function interpret(array $values) {
        return $this->x->interpret($values) * $this->y->interpret($values);
    }
}

// Client
$expression = new Product(
    new Literal(5),
    new Sum(
        new Variable('c'),
        new Literal(2)
    )
);

echo $expression->interpret(array('c' => 3)); // 25
```

我们首先创建了一个`MathExpression`接口，具有一个`interpret`方法。然后添加了`Variable`、`Literal`、`Sum`和`Product`类，它们都实现了`MathExpression`接口。然后客户端从`Product`类实例化，将`Literal`和`Sum`的实例传递给它，并最后调用`interpret`方法。

## 迭代器模式

迭代器模式用于遍历容器并访问其元素。换句话说，一个类变得能够遍历另一个类的元素。PHP 原生支持迭代器，作为内置的`\Iterator`和`\IteratorAggregate`接口的一部分。

以下是迭代器模式实现的示例：

```php
class ProductIterator implements \Iterator {
    private $position = 0;
    private $productsCollection;

    public function __construct(ProductCollection $productsCollection) {
        $this->productsCollection = $productsCollection;
    }

    public function current() {
        return $this->productsCollection->getProduct($this->position);
    }

    public function key() {
        return $this->position;
    }

    public function next() {
        $this->position++;
    }

    public function rewind() {
        $this->position = 0;
    }

    public function valid() {
        return !is_null($this->productsCollection->getProduct($this->position));
    }
}

class ProductCollection implements \IteratorAggregate {
    private $products = array();

    public function getIterator() {
        return new ProductIterator($this);
    }

    public function addProduct($string) {
        $this->products[] = $string;
    }

    public function getProduct($key) {
        if (isset($this->products[$key])) {
            return $this->products[$key];
        }
        return null;
    }

    public function isEmpty() {
        return empty($products);
    }
}

$products = new ProductCollection();
$products->addProduct('T-Shirt Red');
$products->addProduct('T-Shirt Blue');
$products->addProduct('T-Shirt Green');
$products->addProduct('T-Shirt Yellow');

foreach ($products as $product) {
    var_dump($product);
}
```

我们首先创建了一个实现标准 PHP`\Iterator`接口的`ProductIterator`。然后添加了实现标准 PHP`\IteratorAggregate`接口的`ProductCollection`。客户端创建了一个`ProductCollection`的实例，通过`addProduct`方法调用将值堆叠到其中，并循环遍历整个集合。

## 中介者模式

我们的软件中有更多的类，它们的通信变得更加复杂。中介者模式通过将复杂性封装到中介者对象中来解决这个问题。对象不再直接通信，而是通过中介者对象，从而降低了整体耦合度。

以下是中介者模式实现的示例：

```php
interface MediatorInterface {
    public function fight();
    public function talk();
    public function registerA(ColleagueA $a);
    public function registerB(ColleagueB $b);
}

class ConcreteMediator implements MediatorInterface {
    protected $talk; // ColleagueA
    protected $fight; // ColleagueB

    public function registerA(ColleagueA $a) {
        $this->talk = $a;
    }

    public function registerB(ColleagueB $b) {
        $this->fight = $b;
    }

    public function fight() {
        echo 'fighting...';
    }

    public function talk() {
        echo 'talking...';
    }
}

abstract class Colleague {
    protected $mediator; // MediatorInterface
    public abstract function doSomething();
}

class ColleagueA extends Colleague {

    public function __construct(MediatorInterface $mediator) {
        $this->mediator = $mediator;
        $this->mediator->registerA($this);
    }

public function doSomething() {
        $this->mediator->talk();
}
}

class ColleagueB extends Colleague {

    public function __construct(MediatorInterface $mediator) {
        $this->mediator = $mediator;
        $this->mediator->registerB($this);
    }

    public function doSomething() {
        $this->mediator->fight();
    }
}

// Client
$mediator = new ConcreteMediator();
$talkColleague = new ColleagueA($mediator);
$fightColleague = new ColleagueB($mediator);

$talkColleague->doSomething();
$fightColleague->doSomething();
```

我们首先创建了一个具有多个方法的`MediatorInterface`，由`ConcreteMediator`类实现。然后我们定义了抽象类`Colleague`，强制在以下`ColleagueA`和`ColleagueB`类上实现`doSomething`方法。客户端首先实例化`ConcreteMediator`，然后将其实例传递给`ColleagueA`和`ColleagueB`的实例，然后调用`doSomething`方法。

## 备忘录模式

备忘录模式提供了对象恢复功能。实现是通过三个不同的对象完成的；原始者、caretaker 和 memento，其中原始者是保留内部状态以便以后恢复的对象。

以下是备忘录模式实现的示例：

```php
class Memento {
    private $state;

    public function __construct($state) {
        $this->state = $state;
    }

    public function getState() {
        return $this->state;
    }
}

class Originator {
    private $state;

    public function setState($state) {
        return $this->state = $state;
    }

    public function getState() {
        return $this->state;
    }

    public function saveToMemento() {
        return new Memento($this->state);
    }

    public function restoreFromMemento(Memento $memento) {
        $this->state = $memento->getState();
    }
}

// Client - Caretaker
$savedStates = array();

$originator = new Originator();
$originator->setState('new');
$originator->setState('pending');
$savedStates[] = $originator->saveToMemento();
$originator->setState('processing');
$savedStates[] = $originator->saveToMemento();
$originator->setState('complete');
$originator->restoreFromMemento($savedStates[1]);
echo $originator->getState(); // processing
```

我们首先创建了一个`Memento`类，它通过`getState`方法提供对象的当前状态。然后我们定义了`Originator`类，将状态推送到`Memento`。最后，客户端通过实例化`Originator`来扮演`caretaker`的角色，在其少数状态之间进行切换，保存并从`memento`中恢复它们。

## 观察者模式

观察者模式实现了对象之间的一对多依赖关系。持有依赖项列表的对象称为**subject**，而依赖项称为**observers**。当主题对象改变状态时，所有依赖项都会被通知并自动更新。

以下是观察者模式实现的示例：

```php
class Customer implements \SplSubject {
    protected $data = array();
    protected $observers = array();

    public function attach(\SplObserver $observer) {
        $this->observers[] = $observer;
    }

    public function detach(\SplObserver $observer) {
        $index = array_search($observer, $this->observers);

        if ($index !== false) {
            unset($this->observers[$index]);
        }
    }

    public function notify() {
        foreach ($this->observers as $observer) {
            $observer->update($this);
            echo 'observer updated';
        }
    }

    public function __set($name, $value) {
        $this->data[$name] = $value;

        // notify the observers, that user has been updated
        $this->notify();
    }
}

class CustomerObserver implements \SplObserver {
    public function update(\SplSubject $subject) {
        /* Implementation... */
    }
}

// Client
$user = new Customer();
$customerObserver = new CustomerObserver();
$user->attach($customerObserver);

$user->name = 'John Doe';
$user->email = 'john.doe@fake.mail';
```

我们首先创建了一个实现标准 PHP`\SplSubject`接口的`Customer`类。然后定义了一个实现标准 PHP`\SplObserver`接口的`CustomerObserver`类。最后，客户端实例化了`Customer`和`CustomerObserver`对象，并将`CustomerObserver`对象附加到`Customer`上。然后`observer`捕捉到`name`和`email`属性的任何更改。

## 状态模式

状态模式封装了基于其内部状态的相同对象的不同行为，使对象看起来好像已经改变了它的类。

以下是状态模式实现的示例：

```php
interface Statelike {
    public function writeName(StateContext $context, $name);
}

class StateLowerCase implements Statelike {
    public function writeName(StateContext $context, $name) {
        echo strtolower($name);
        $context->setState(new StateMultipleUpperCase());
    }
}

class StateMultipleUpperCase implements Statelike {
    private $count = 0;

    public function writeName(StateContext $context, $name) {
        $this->count++;
        echo strtoupper($name);
        /* Change state after two invocations */
        if ($this->count > 1) {
            $context->setState(new StateLowerCase());
        }
    }
}

class StateContext {
    private $state;

    public function setState(Statelike $state) {
        $this->state = $state;
    }

    public function writeName($name) {
        $this->state->writeName($this, $name);
    }
}

// Client
$stateContext = new StateContext();
$stateContext->setState(new StateLowerCase());
$stateContext->writeName('Monday');
$stateContext->writeName('Tuesday');
$stateContext->writeName('Wednesday');
$stateContext->writeName('Thursday');
$stateContext->writeName('Friday');
$stateContext->writeName('Saturday');
$stateContext->writeName('Sunday');
```

我们首先创建了一个`Statelike`接口，然后是实现该接口的`StateLowerCase`和`StateMultipleUpperCase`。`StateMultipleUpperCase`在其`writeName`中添加了一些计数逻辑，因此在两次调用后会启动新状态。然后我们定义了`StateContext`类，我们将使用它来切换上下文。最后，客户端实例化了`StateContext`，并通过`setState`方法将`StateLowerCase`的实例传递给它，然后调用了几次`writeName`方法。

## 策略模式

策略模式定义了一组算法，每个算法都被封装并与该组内的其他成员交换使用。

以下是策略模式实现的示例：

```php
interface PaymentStrategy {
    public function pay($amount);
}

class StripePayment implements PaymentStrategy {
    public function pay($amount) {
        echo 'StripePayment...';
    }

}

class PayPalPayment implements PaymentStrategy {
    public function pay($amount) {
        echo 'PayPalPayment...';
    }
}

class Checkout {
    private $amount = 0;

    public function __construct($amount = 0) {
        $this->amount = $amount;
    }

    public function capturePayment() {
        if ($this->amount > 99.99) {
            $payment = new PayPalPayment();
        } else {
            $payment = new StripePayment();
        }

        $payment->pay($this->amount);
    }
}

$checkout = new Checkout(49.99);
$checkout->capturePayment(); // StripePayment...

$checkout = new Checkout(199.99);
$checkout->capturePayment(); // PayPalPayment...
```

我们首先创建了一个`PaymentStrategy`接口，然后是实现它的具体类`StripePayment`和`PayPalPayment`。然后我们定义了`Checkout`类，在`capturePayment`方法中加入了一些决策逻辑。最后，客户端通过构造函数传递一定金额来实例化`Checkout`。根据金额，`Checkout`在调用`capturePayment`时内部触发一个或另一个`payment`。

## 模板模式

模板设计模式定义了算法在方法中的程序骨架。它让我们通过类覆盖的方式重新定义算法的某些步骤，而不真正改变算法的结构。

以下是模板模式实现的示例：

```php
abstract class Game {
    private $playersCount;

    abstract function initializeGame();
    abstract function makePlay($player);
    abstract function endOfGame();
    abstract function printWinner();

    public function playOneGame($playersCount)
    {
        $this->playersCount = $playersCount;
        $this->initializeGame();
        $j = 0;
        while (!$this->endOfGame()) {
            $this->makePlay($j);
            $j = ($j + 1) % $playersCount;
        }
        $this->printWinner();
    }
}

class Monopoly extends Game {
    public function initializeGame() {
        // Implementation...
    }

    public function makePlay($player) {
        // Implementation...
    }

    public function endOfGame() {
        // Implementation...
    }

    public function printWinner() {
        // Implementation...
    }
}

class Chess extends Game {
    public function  initializeGame() {
        // Implementation...
    }

    public function  makePlay($player) {
        // Implementation...
    }

    public function  endOfGame() {
        // Implementation...
    }

    public function  printWinner() {
        // Implementation...
    }
}

$game = new Chess();
$game->playOneGame(2);

$game = new Monopoly();
$game->playOneGame(4);
```

我们首先创建了一个提供了封装游戏玩法的所有抽象方法的抽象`Game`类。然后我们定义了`Monopoly`和`Chess`类，它们都是从`Game`类继承的，为每个游戏实现了特定的游戏玩法方法。客户端只需实例化`Monopoly`和`Chess`对象，然后在每个对象上调用`playOneGame`方法。

## 访问者模式

访问者设计模式是一种将算法与其操作的对象结构分离的方法。因此，我们能够向现有的对象结构添加新的操作，而不实际修改这些结构。

以下是访问者模式实现的示例：

```php
interface RoleVisitorInterface {
    public function visitUser(User $role);
    public function visitGroup(Group $role);
}

class RolePrintVisitor implements RoleVisitorInterface {
    public function visitGroup(Group $role) {
        echo 'Role: ' . $role->getName();
    }

    public function visitUser(User $role) {
        echo 'Role: ' . $role->getName();
    }
}

abstract class Role {
    public function accept(RoleVisitorInterface $visitor) {
        $klass = get_called_class();
        preg_match('#([^\\\\]+)$#', $klass, $extract);
        $visitingMethod = 'visit' . $extract[1];

        if (!method_exists(__NAMESPACE__ . '\RoleVisitorInterface', $visitingMethod)) {
            throw new \InvalidArgumentException("The visitor you provide cannot visit a $klass instance");
        }

        call_user_func(array($visitor, $visitingMethod), $this);
    }
}

class User extends Role {
    protected $name;

    public function __construct($name) {
        $this->name = (string)$name;
    }

    public function getName() {
        return 'User ' . $this->name;
    }
}

class Group extends Role {
    protected $name;

    public function __construct($name) {
        $this->name = (string)$name;
    }

    public function getName() {
        return 'Group: ' . $this->name;
    }
}

$group = new Group('my group');
$user = new User('my user');

$visitor = new RolePrintVisitor;

$group->accept($visitor);
$user->accept($visitor);
```

我们首先创建了一个`RoleVisitorInterface`，然后是实现`RoleVisitorInterface`的`RolePrintVisitor`。然后我们定义了抽象类`Role`，其中包含一个接受`RoleVisitorInterface`参数类型的方法。我们进一步定义了具体的`User`和`Group`类，它们都是从`Role`继承的。客户端实例化了`User`、`Group`和`RolePrintVisitor`，并将`visitor`传递给`User`和`Group`实例的`accept`方法调用。

# 摘要

设计模式是开发人员的一种常见的高级语言。它们使团队成员之间能够以简化的方式交流应用程序设计。了解如何识别和实现设计模式将我们的重点转移到解决业务需求，而不是在代码层面上如何将解决方案粘合在一起。

编码，就像大多数手工制作的学科一样，是你得到你所付出的。虽然实现一些设计模式需要一定的时间，但在较大的项目中不这样做可能会在未来以某种方式追上我们。与“使用框架还是不使用框架”辩论类似，实现正确的设计模式会影响我们代码的*可扩展性*、*可重用性*、*适应性*和*可维护性*。因此，使其更具未来性。

在接下来的章节中，我们将深入研究 SOLID 设计原则及其在软件开发过程中的作用。
