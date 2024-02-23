# 第十章：常见的设计模式

那些刚接触软件开发的人往往把精力集中在掌握编程语言上。一旦突破了这个障碍，就是时候拥抱**设计模式**了，因为写高质量和复杂的软件几乎不可能没有它们。设计模式主要是由经验丰富的开发者使用，它们代表了在我们的应用程序中面临的常见挑战的一种成熟的解决方案。成功应用设计模式很可能会导致更具可扩展性、可重用性、可维护性和可适应性的代码。

本章中的示例并不是要被复制粘贴的。它们只是用来代表设计模式的一种可能实现。毕竟，现实生活中的应用程序都是关于细节的。此外，还有许多其他设计模式，随着技术和编程范式的转变，还会不断有新的设计模式被发明出来。

在本章中，我们将看一下在 PHP 中设计模式的几种可能实现：

+   基本模式

+   注册表模式

+   创建模式

+   单例模式

+   原型模式

+   抽象工厂模式

+   建造者模式

+   对象池模式

+   行为模式

+   策略模式

+   观察者模式

+   懒初始化模式

+   责任链模式

+   结构模式

+   装饰者模式

# 基本模式

在接下来的部分，我们将看一下基本模式：注册表模式。

# 注册表模式

注册表模式是一个有趣的模式。它允许我们存储和检索对象以供以后使用。存储和检索的过程是基于我们定义的键。根据数据范围的不同，键和对象的关联是全局的，跨进程、线程或会话，允许我们在数据范围内的任何地方检索对象。

以下示例演示了可能的注册表模式实现：

```php
<?php   class Registry {
  private
  $registry = [];    public
 function get($key)
 {  if (isset($this->registry[$key])) {
  return $this->registry[$key];
 }  return null;
 }    public
 function set($key, $value, $graceful = false)
 {  if (isset($this->registry[$key])) {
  if ($graceful) {
  return;
 }  throw new \RuntimeException('Registry key "' . $key . '"already exists');
 }  $this->registry[$key] = $value;
 }    public
 function remove($key)
 {  if (isset($this->registry[$key])) {
  unset($this->registry[$key]);
 } }    public
 function __destruct()
 {  $keys = array_keys($this->registry);
  array_walk($keys, [$this, 'remove']);
 } }   // Client use class User {
  public $name; }   $user1 = new User(); $user1->name = 'John'; $user2 = new User(); $user2->name = 'Marc';   $registry = new Registry(); $registry->set('employee', $user1); $registry->set('director', $user2); echo $registry->get('director')->name; // Marc

```

我们的`Registry`类实现有三个关键方法：`get()`、`set()`、`remove()`。`set()`方法允许基于`$graceful`参数进行优雅的行为；否则，它会为现有的键触发`RuntimeException`。我们还定义了一个`__destruct`方法，作为一种清理机制，当`$registry`实例被销毁时，它会移除注册表中的每个项目。

# 创建模式

在这一部分，我们将看一下创建模式，比如单例、原型、抽象工厂和构建者模式。

# 单例模式

单例模式是大多数开发人员学习的第一个设计模式。这个设计模式的目标是将类实例化的次数限制为只有一个。这意味着对类使用`new`关键字将始终返回一个相同的对象实例。这是一个强大的概念，它允许我们实现各种应用程序范围的对象，比如记录器、邮件发送器、注册表和其他功能的单例。然而，正如我们很快会看到的，我们将完全避免使用`new`关键字，并通过静态类方法实例化对象。

以下示例演示了可能的单例模式实现：

```php
<?php   class Logger {
  private static $instance;    const TYPE_ERROR = 'error';
  const TYPE_WARNING = 'warning';
  const TYPE_NOTICE = 'notice';    protected function __construct()
 {  // empty?!
  }    private function __clone()
 {  // empty?!
  }    private function __wakeup()
 {  // empty?!
  }    public static function getInstance()
 {  if (!isset(self::$instance)) {
  // late static binding
  self::$instance = new self;
 }  return self::$instance;
 }    public function log($type, $message)
 {  return sprintf('Logging %s: %s', $type, $message);
 } }   // Client use echo Logger::getInstance()->log(Logger::TYPE_NOTICE, 'test');

```

`Logger`类使用静态成员`$instance`来保持一个`self`的实例，根据`getInstance()`方法的实现。我们将`__construct`定义为`protected`，以防止通过`new`操作符创建新实例。`__clone()`方法被定义为`private`，以防止通过`clone`操作符进行实例克隆。同样，`__wakeup()`方法也被定义为 private，以防止通过`unserialize()`函数进行实例反序列化。这些简单的限制使得该类作为单例。要获取一个实例，只需调用`getInstance()`类方法。

# 原型模式

原型模式是通过克隆来创建新对象的。这是一个相当有意思的概念，因为我们不再使用`new`关键字来创建新对象。PHP 语言提供了一个特殊的`clone`关键字来辅助对象克隆。

以下示例演示了可能的原型模式实现：

```php
<?php   class Logger {
  public $channel = 'N/A'; }   class SystemLogger extends Logger {
  public function __construct()
 {  $this->channel = 'STDIN';
 }    public function log($data)
 {  return sprintf('Logging %s to %s.', $data, $this->channel);
 }    public function __clone()
 {  /* additional changes for (after)clone behavior? */
  } }   // Client use $systemLogger = new SystemLogger(); echo $systemLogger->log('test');   $logger = clone $systemLogger; echo $logger->log('test2');   $logger->channel = 'mail'; echo $logger->log('test3');   // Logging test to STDIN. // Logging test2 to STDIN. // Logging test3 to mail.

```

通常，克隆对象只需使用表达式`$clonedObj = clone $obj;`。然而，这并不会让我们对克隆过程有任何控制。PHP 对象可能很重，有很多成员和引用。有时，我们希望对克隆对象施加一定的限制。这就是魔术`__clone()`方法派上用场的地方。`__clone()`方法在克隆过程完成后触发，这是可能清理代码实现时需要记住的事情。

# 抽象工厂模式

抽象工厂封装了具有共同功能的一组单独工厂，而不指定它们的具体类。这样可以更容易地编写可移植的代码，因为客户端可以在不更改代码的情况下交换具体实现。

以下示例演示了可能的抽象工厂模式实现：

```php
<?php   interface Button {
  public function render(); }   interface FormFactory {
  public function createButton(); }   class LoginButton implements Button {
  public function render()
 {  return '<button name="login">Login</button>';
 } }   class RegisterButton implements Button {
  public function render()
 {  return '<button name="register">Register</button>';
 } }   class LoginFactory implements FormFactory {
  public function createButton()
 {  return new LoginButton();
 } }   class RegisterFactory implements FormFactory {
  public function createButton()
 {  return new RegisterButton();
 } }   // Client $loginButtonFactory = new LoginFactory(); $button = $loginButtonFactory->createButton(); echo $button->render();   $registerButtonFactory = new RegisterFactory(); $button = $registerButtonFactory->createButton(); echo $button->render();

```

我们首先创建了两个简单的接口，`Button`和`FormFactory`。`Button`接口定义了一个`render()`方法，然后我们通过两个具体类实现`LoginButton`和`RegisterButton`来实现它。两个`FormFactory`实现，`LoginFactory`和`RegisterFactory`，然后在其`createButton()`方法实现中实例化相应的按钮类。客户端只使用`LoginFactory`和`RegisterFactory`实例，从而避免直接实例化具体按钮类。

# 建造者模式

建造者模式是一个非常方便的模式，特别是在处理大型应用程序时。它将复杂对象的构建与其表示分离。这使得相同的构建过程可以创建多种表示。

以下示例演示了可能的建造者模式实现，以`Image`类为例：

```php
<?php   class Image {
  private $width;
  private $height;    public function getWidth()
 {  return $this->width;
 }    public function setWidth($width)
 {  $this->width = $width;
  return $this;
 }    public function getHeight()
 {  return $this->height;
 }    public function setHeight($height)
 {  $this->height = $height;
  return $this;
 } }   interface ImageBuilderInterface {
  public function setWidth($width);    public function setHeight($height);    public function getResult(); }   class ImageBuilder implements ImageBuilderInterface {
  private $image;    public function __construct()
 {  $this->image = new Image();
 }    public function setWidth($width)
 {  $this->image->setWidth($width);
  return $this;
 }    public function setHeight($height)
 {  $this->image->setHeight($height);
  return $this;
 }    public function getResult()
 {  return $this->image;
 } }   class ImageBuildDirector {
  private $builder;    public function __construct(ImageBuilder $builder)
 {  $this->builder = $builder;
 }    public function build()
 {  $this->builder->setWidth(120);
  $this->builder->setHeight(80);
  return $this;
 }    public function getImage()
 {  return $this->builder->getResult();
 } }   // Client use $imageBuilder = new ImageBuilder(); $imageBuildDirector = new ImageBuildDirector($imageBuilder); $image = $imageBuildDirector->build()->getImage();   var_dump($image); // object(Image)#2 (2) { ["width":"Image":private]=> int(120) ["height":"Image":private]=> int(80) }

```

我们首先创建了一个简单的 Image 类，提供了宽度和高度属性以及相应的 getter 和 setter。然后创建了`ImageBuilderInterface`接口，定义了图像宽度和高度的 setter 方法，以及`getResult()`方法。然后创建了一个实现`ImageBuilderInterface`接口的`ImageBuilder`具体类。客户端实例化`ImageBuilder`类。另一个具体类`ImageBuildDirector`通过其`build()`方法将创建或构建代码包装在其构造函数中传递的`ImageBuilder`实例中。

# 对象池模式

对象池模式管理类实例--对象。它用于希望由于资源密集型操作而限制不必要的类实例化的情况。对象池的作用类似于对象的注册表，客户端可以随后获取必要的对象。

以下示例演示了可能的对象池模式实现：

```php
<?php     class ObjectPool {
  private $instances = [];    public function load($key)
 {  return $this->instances[$key];
 }    public function save($object, $key)
 {  $this->instances[$key] = $object;
 } }   class User {
  public function hello($name)
 {  return 'Hello ' . $name;
 } }   // Client use $pool = new ObjectPool();   $user = new User(); $key = spl_object_hash($user);   $pool->save($user, $key);   // code...   $user = $pool->load($key); echo $user->hello('John');

```

只使用数组和两种方法，我们就能够实现一个简单的对象池。`save()`方法将对象添加到`$instances`数组中，而`load()`方法将其返回给客户端。在这种情况下，客户端负责跟踪保存对象的键。对象本身在使用后并不被销毁，因为它们仍然留在池中。

# 行为模式

在这一部分，我们将介绍行为模式，如策略、观察者、延迟初始化和责任链。

# 策略模式

策略模式在我们有多个代码块执行类似操作的情况下非常有用。它定义了一组封装和可互换的算法。想象一下订单结账流程，我们想要实现不同的运输提供商，比如 UPS 和 FedEx。

以下示例演示了可能的策略模式实现：

```php
<?php   interface ShipmentStrategy {
  public function calculate($amount); }   class UPSShipment implements ShipmentStrategy {
  public function calculate($amount)
 {  return 'UPSShipment...';
 } }   class FedExShipment implements ShipmentStrategy {
  public function calculate($amount)
 {  return 'FedExShipment...';
 } }   class Checkout {
  private $amount = 0;    public function __construct($amount = 0)
 {  $this->amount = $amount;
 }    public function estimateShipment()
 {  if ($this->amount > 199.99) {
  $shipment = new FedExShipment();
 } else {
  $shipment = new UPSShipment();
 }    return $shipment->calculate($this->amount);
 } }   // Client use $checkout = new Checkout(19.99); echo $checkout->estimateShipment(); // UPSShipment...   $checkout = new Checkout(499.99); echo $checkout->estimateShipment(); // FedExShipment...

```

我们首先定义了一个带有`calculate()`方法的`ShipmentStrategy`接口。然后我们定义了`UPSShipment`和`FedExShipment`类，它们实现了`ShipmentStrategy`接口。有了这两个具体的运输类，我们创建了一个`Checkout`类，它在其`estimateShipment()`方法中封装了这两种运输选项。然后客户端调用`Checkout`实例的`estimateShipment()`方法。根据传递的金额，不同的运输计算会启动。使用这种模式，我们可以在不改变客户端的情况下自由添加新的运输计算。

# 观察者模式

观察者模式非常受欢迎。它允许事件订阅类型的行为。我们区分主题和观察者类型的对象。观察者是订阅主题对象状态变化的对象。当主题改变其状态时，它会自动通知所有观察者。

以下示例演示了可能的观察者模式实现：

```php
<?php   class CheckoutSuccess implements \SplSubject {
  protected $salesOrder;
  protected $observers;    public function __construct($salesOrder)
 {  $this->salesOrder = $salesOrder;
  $this->observers = new \SplObjectStorage();
 }    public function attach(\SplObserver $observer)
 {  $this->observers->attach($observer);
 }    public function detach(\SplObserver $observer)
 {  $this->observers->detach($observer);
 }    public function notify()
 {  foreach ($this->observers as $observer) {
  $observer->update($this);
 } }    public function getSalesOrder()
 {  return $this->salesOrder;
 } }   class SalesOrder { }   class Mailer implements \SplObserver {
  public function update(\SplSubject $subject)
 {  echo 'Mailing ', get_class($subject->getSalesOrder()), PHP_EOL;
 } }   class Logger implements \SplObserver {
  public function update(\SplSubject $subject)
 {  echo 'Logging ', get_class($subject->getSalesOrder()), PHP_EOL;
 } }   $salesOrder = new SalesOrder(); $checkoutSuccess = new CheckoutSuccess($salesOrder); // some code... $checkoutSuccess->attach(new Mailer()); // some code... $checkoutSuccess->attach(new Logger()); // some code... $checkoutSuccess->notify();

```

PHP 的`\SplSubject`和`\SplObserver`接口允许观察者模式的实现。我们的结账成功示例使用这些接口来实现`CheckoutSuccess`作为主题类型对象的类，以及`Mailer`和`Logger`作为观察者类型对象的类。使用`CheckoutSuccess`实例的`attach()`方法，我们将两个观察者附加到主题上。一旦调用主题的`notify()`方法，就会触发各个观察者的`update()`方法。在我们的示例中，`getSalesOrder()`方法的调用可能会让人感到意外，因为在`SplSubject`对象的直接实例上实际上没有`getSalesOrder()`方法。然而，在我们的示例中，两个`update(\SplSubject $subject)`方法调用将接收到一个`CheckoutSuccess`的实例。否则，直接将`$subject`参数强制转换为`CheckoutSuccess`将导致 PHP 致命错误。

```php
PHP Fatal error: Declaration of Logger::update(CheckoutSuccess $subject) must be compatible with SplObserver::update(SplSubject $SplSubject)

```

# 延迟初始化模式

延迟初始化模式对于处理实例化可能消耗大量资源的对象非常有用。其思想是延迟实际的资源密集型操作，直到实际需要其结果为止。PDF 生成是一个轻度到中度资源密集型操作的例子。

以下示例演示了基于 PDF 生成的可能的延迟初始化模式实现：

```php
<?php   interface PdfInterface {
  public function generate(); }   class Pdf implements PdfInterface {
  private $data;    public function __construct($data)
 {  $this->data = $data;
  // Imagine resource intensive pdf generation here
  sleep(3);
 }    public function generate()
 {  echo 'pdf: ' . $this->data;
 } }   class ProxyPdf implements PdfInterface {
  private $pdf = null;
  private $data;    public function __construct($data)
 {  $this->data = $data;
 }    public function generate()
 {  if (is_null($this->pdf)) {
  $this->pdf = new Pdf($this->data);
 }  $this->pdf->generate();
 } }   // Client $pdf = new Pdf('<h1>Hello</h1>'); // 3 seconds // Some other code ... $pdf->generate();   $pdf = new ProxyPdf('<h1>Hello</h1>'); // 0 seconds // Some other code ... $pdf->generate();

```

根据类的构造方式，它可能会在我们调用`new`关键字后立即触发实际的生成，就像我们使用`new Pdf(...)`表达式一样。`new ProxyPdf(...)`表达式的行为不同，因为它包装了实现相同`PdfInterface`的`Pdf`类，但提供了不同的`__construct()`方法实现。

# 责任链模式

责任链模式允许我们以发送者-接收者的方式链接代码，同时两者彼此解耦。这使得可以有多个对象处理传入的请求。

以下示例演示了使用日志记录功能作为示例的可能的责任链模式实现：

```php
<?php   abstract class Logger {
  private $logNext = null;    public function logNext(Logger $logger)
 {  $this->logNext = $logger;
  return $this->logNext;
 }    final public function push($message)
 {  $this->log($message);    if ($this->logNext !== null) {
  $this->logNext->push($message);
 } }    abstract protected function log($message); }   class SystemLogger extends Logger {
  public function log($message)
 {  echo 'SystemLogger log!', PHP_EOL;
 } }   class ElasticLogger extends Logger {
  protected function log($message)
 {  echo 'ElasticLogger log!', PHP_EOL;
 } }   class MailLogger extends Logger {
  protected function log($message)
 {  echo 'MailLogger log!', PHP_EOL;
 } }   // Client use $systemLogger  = new SystemLogger(); $elasticLogger = new ElasticLogger(); $mailLogger = new MailLogger();   $systemLogger   ->logNext($elasticLogger)
 ->logNext($mailLogger);   $systemLogger->push('Stuff to log...');   //SystemLogger log! //ElasticLogger log! //MailLogger log!

```

我们首先创建了一个抽象的`Logger`类，其中包含三个方法：`logNext()`、`push()`和`log()`。`log()`方法被定义为抽象，这意味着实现留给子类。`logNext()`方法是关键因素，因为它将对象传递到链中。然后我们创建了`Logger`类的三个具体实现：`SystemLogger`、`ElasticLogger`和`MailLogger`。然后我们实例化了其中一个具体的记录器类，并使用`logNext()`方法将另外两个实例传递到链中。最后，我们调用了`push()`方法来触发链。

# 结构模式

在这一部分，我们将看一下一个结构模式：装饰器模式。

# 装饰器模式

装饰器模式很简单。它允许我们在不影响同一类的其他实例的情况下，为对象实例添加新的行为。它基本上充当了我们对象的装饰包装器。我们可以想象一个简单的用例，使用 Logger 类的实例，我们有一个简单的记录器类，我们希望偶尔装饰，或者包装成更具体的错误、警告和通知级别的记录器。

以下示例演示了可能的装饰器模式实现：

```php
<?php   interface LoggerInterface {
  public function log($message); }   class Logger implements LoggerInterface {
  public function log($message)
 {  file_put_contents('app.log', $message . PHP_EOL, FILE_APPEND);
 } }   abstract class LoggerDecorator implements LoggerInterface {
  protected $logger;    public function __construct(Logger $logger)
 {  $this->logger = $logger;
 }    abstract public function log($message); }   class ErrorLogger extends LoggerDecorator {
  public function log($message)
 {  $this->logger->log('ErrorLogger: ' . $message);
 } }   class WarningLogger extends LoggerDecorator {
  public function log($message)
 {  $this->logger->log('WarningLogger: ' . $message);
 } }   class NoticeLogger extends LoggerDecorator {
  public function log($message)
 {  $this->logger->log('NoticeLogger: ' . $message);
 } }   // Client use (new Logger())->log('Test Logger.');   (new ErrorLogger(new Logger()))->log('Test ErrorLogger.');   (new WarningLogger(new Logger()))->log('Test WarningLogger.');   (new NoticeLogger(new Logger()))->log('Test NoticeLogger.');

```

在这里，我们首先定义了一个`LoggerInterface`接口和一个实现该接口的具体`Logger`类。然后我们创建了一个`abstract` `LoggerDecorator`类，它也实现了`LoggerInterface`。`LoggerDecorator`实际上并没有实现`log()`方法；它将其定义为`abstract`，以便未来的子类来实现。最后，我们定义了具体的错误、警告和通知装饰器类。我们可以看到它们的`log()`方法根据其角色装饰输出。结果输出如下：

```php
Test Logger.
ErrorLogger: Test ErrorLogger.
WarningLogger: Test WarningLogger.
NoticeLogger: Test NoticeLogger.

```

# 总结

在本章中，我们以实际操作的方式介绍了 PHP 应用程序中最常用的一些设计模式。这个列表还远未完成，因为还有其他设计模式可用。虽然有些设计模式非常通用，但其他可能更适合 GUI 或应用程序编程的其他领域。了解如何使用和应用设计模式使我们的代码更具可扩展性、可重用性、可维护性和适应性。

接下来，我们将更仔细地研究使用 SOAP、REST 和 Apache Thrift 构建 Web 服务。
