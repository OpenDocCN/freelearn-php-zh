# 第三章 SOLID 设计原则

构建模块化软件需要对类设计有很强的了解。有许多指南，涉及我们如何命名我们的类，它们应该有多少变量，方法的大小应该是多少等等。PHP 生态系统成功地将这些打包成官方的 PSR 标准，更确切地说是 PSR-1：基本编码标准和 PSR-2：编码风格指南。这些都是保持我们的代码可读、可理解和可维护的一般编程指南。

除了编程指南，我们还可以在类设计过程中应用更具体的设计原则。这些原则涉及低耦合、高内聚和强封装的概念。我们称之为 SOLID 设计原则，这是罗伯特·塞西尔·马丁在 21 世纪初提出的一个术语。

SOLID 是以下五个原则的首字母缩写：

+   S：单一职责原则（SRP）

+   O：开放/封闭原则（OCP）

+   L：里氏替换原则（LSP）

+   I：接口隔离原则（ISP）

+   D：依赖倒置原则（DIP）

十多年前，SOLID 原则的概念远未过时，因为它们是良好类设计的核心。在本章中，我们将深入研究这些原则，通过观察一些明显违反原则的违规行为来了解它们。

在本章中，我们将涵盖以下主题：

+   单一职责原则

+   开放/封闭原则

+   里氏替换原则

+   接口隔离原则

+   依赖倒置原则

# 单一职责原则

单一职责原则处理试图做太多事情的类。这里的责任指的是改变的原因。根据罗伯特·C·马丁的定义：

> “一个类应该只有一个改变的原因。”

以下是一个违反 SRP 的类的示例：

```php
class Ticket {
    const SEVERITY_LOW = 'low';
    const SEVERITY_HIGH = 'high';
    // ...
    protected $title;
    protected $severity;
    protected $status;
    protected $conn;

    public function __construct(\PDO $conn) {
        $this->conn = $conn;
    }

    public function setTitle($title) {
        $this->title = $title;
    }

    public function setSeverity($severity) {
        $this->severity = $severity;
    }

    public function setStatus($status) {
        $this->status = $status;
    }

    private function validate() {
        // Implementation...
    }

    public function save() {
        if ($this->validate()) {
            // Implementation...
        }
    }

}

// Client
$conn = new PDO(/* ... */);
$ticket = new Ticket($conn);
$ticket->setTitle('Checkout not working!');
$ticket->setStatus(Ticket::STATUS_OPEN);
$ticket->setSeverity(Ticket::SEVERITY_HIGH);
$ticket->save();
```

`Ticket`类处理`ticket`实体的验证和保存。这两个责任是它改变的原因。每当关于票证验证或保存的要求发生变化时，`Ticket`类都必须进行修改。为了解决这里的 SRP 违规问题，我们可以使用辅助类和接口来分割责任。

以下是符合 SRP 的重构实现的示例：

```php
interface KeyValuePersistentMembers {
    public function toArray();
}

class Ticket implements KeyValuePersistentMembers {
    const STATUS_OPEN = 'open';
    const SEVERITY_HIGH = 'high';
    //...
    protected $title;
    protected $severity;
    protected $status;

    public function setTitle($title) {
        $this->title = $title;
    }

    public function setSeverity($severity) {
        $this->severity = $severity;
    }

    public function setStatus($status) {
        $this->status = $status;
    }

    public function toArray() {
        // Implementation...
    }
}

class EntityManager {
    protected $conn;

    public function __construct(\PDO $conn) {
        $this->conn = $conn;
    }

    public function save(KeyValuePersistentMembers $entity)
    {
        // Implementation...
    }
}

class Validator {
    public function validate(KeyValuePersistentMembers $entity) {
        // Implementation...
    }
}

// Client
$conn = new PDO(/* ... */);

$ticket = new Ticket();
$ticket->setTitle('Payment not working!');
$ticket->setStatus(Ticket::STATUS_OPEN);
$ticket->setSeverity(Ticket::SEVERITY_HIGH);

$validator = new Validator();

if ($validator->validate($ticket)) {
    $entityManager = new EntityManager($conn);
    $entityManager->save($ticket);
}
```

在这里，我们引入了一个简单的`KeyValuePersistentMembers`接口，其中有一个`toArray`方法，然后将其用于`EntityManager`和`Validator`类，这两个类现在都承担了单一职责。`Ticket`类变成了一个简单的数据持有模型，而客户端现在控制*实例化*、*验证*和*保存*作为三个不同的步骤。虽然这当然不是如何分离责任的通用公式，但它提供了一个简单明了的例子来解决这个问题。

在考虑单一职责原则的情况下进行设计会产生更小、更易读和更易测试的代码。

# 开放/封闭原则

开放/封闭原则规定一个类应该对扩展开放，但对修改封闭，根据维基百科上的定义：

> “软件实体（类、模块、函数等）应该对扩展开放，但对修改封闭”

对于扩展开放的部分意味着我们应该设计我们的类，以便在需要时可以添加新功能。对于修改封闭的部分意味着这个新功能应该适合而不修改原始类。类只应在修复错误时进行修改，而不是添加新功能。

以下是一个违反开放/封闭原则的类的示例：

```php
class CsvExporter {
    public function export($data) {
        // Implementation...
    }
}

class XmlExporter {
    public function export($data) {
        // Implementation...
    }
}

class GenericExporter {
    public function exportToFormat($data, $format) {
        if ('csv' === $format) {
            $exporter = new CsvExporter();
        } elseif ('xml' === $format) {
            $exporter = new XmlExporter();
        } else {
            throw new \Exception('Unknown export format!');
        }
        return $exporter->export($data);
    }
}
```

在这里，我们有两个具体类，`CsvExporter`和`XmlExporter`，每个都有一个单一的职责。然后我们有一个`GenericExporter`，其`exportToFormat`方法实际上触发了适当实例类型上的`export`函数。问题在于我们无法在不修改`GenericExporter`类的情况下添加新类型的导出器。换句话说，`GenericExporter`对扩展不开放，对修改封闭。

以下是符合 OCP 的重构实现的一个例子：

```php
interface ExporterFactoryInterface {
    public function buildForFormat($format);
}

interface ExporterInterface {
    public function export($data);
}

class CsvExporter implements ExporterInterface {
    public function export($data) {
        // Implementation...
    }
}

class XmlExporter implements ExporterInterface {
    public function export($data) {
        // Implementation...
    }
}

class ExporterFactory implements ExporterFactoryInterface {
    private $factories = array();

    public function addExporterFactory($format, callable $factory) {
          $this->factories[$format] = $factory;
    }

    public function buildForFormat($format) {
        $factory = $this->factories[$format];
        $exporter = $factory(); // the factory is a callable

        return $exporter;
    }
}

class GenericExporter {
    private $exporterFactory;

    public function __construct(ExporterFactoryInterface $exporterFactory) {
        $this->exporterFactory = $exporterFactory;
    }

    public function exportToFormat($data, $format) {
        $exporter = $this->exporterFactory->buildForFormat($format);
        return $exporter->export($data);
    }
}

// Client
$exporterFactory = new ExporterFactory();

$exporterFactory->addExporterFactory(
'xml',
    function () {
        return new XmlExporter();
    }
);

$exporterFactory->addExporterFactory(
'csv',
    function () {
        return new CsvExporter();
    }
);

$data = array(/* ... some export data ... */);
$genericExporter = new GenericExporter($exporterFactory);
$csvEncodedData = $genericExporter->exportToFormat($data, 'csv');
```

在这里，我们添加了两个接口，`ExporterFactoryInterface`和`ExporterInterface`。然后修改了`CsvExporter`和`XmlExporter`以实现该接口。添加了`ExporterFactory`，实现了`ExporterFactoryInterface`。它的主要作用由`buildForFormat`方法定义，该方法返回导出器作为回调函数。最后，`GenericExporter`被重写以通过其构造函数接受`ExporterFactoryInterface`，其`exportToFormat`方法现在通过导出器工厂构建导出器并调用其`execute`方法。

客户端本身现在扮演了更加强大的角色，首先实例化了`ExporterFactory`并向其中添加了两个导出器，然后将其传递给`GenericExporter`。现在向`GenericExporter`添加新的导出格式不再需要修改它，因此使其对扩展开放，对修改封闭。这绝不是一个通用的公式，而是一种可能的满足 OCP 的方法概念。

# 里氏替换原则

**里氏替换原则**讨论了继承。它指定了我们应该如何设计我们的类，以便客户端依赖项可以被子类替换而客户端看不到差异，根据维基百科上的定义：

> “程序中的对象应该能够被其子类型的实例替换，而不会改变程序的正确性”

虽然子类可能添加了一些特定的功能，但它必须符合与其基类相同的行为。否则，违反了里氏原则。

在涉及 PHP 和子类化时，我们必须超越简单的具体类，并区分：具体类、抽象类和接口。这三者都可以放在基类的上下文中，而扩展或实现它的所有内容都可以被视为派生类。

以下是 LSP 违规的一个例子，派生类没有实现所有方法：

```php
interface User {
    public function getEmail();
    public function getName();
    public function getAge();
}

class Employee implements User {
    public function getEmail() {
        // Implementation...
    }

    public function getAge() {
        // Implementation...
    }
}
```

在这里，我们看到一个`employee`类，它没有实现接口强制执行的`getName`方法。我们本可以使用抽象类而不是接口和抽象方法类型来代替`getName`方法，效果将是相同的。幸运的是，在这种情况下，PHP 会抛出错误，警告我们并没有完全实现接口。

以下是违反里氏原则的一个例子，不同的派生类返回不同类型的东西：

```php
class UsersCollection implements \Iterator {
    // Implementation...
}

interface UserList {
    public function getUsers();
}

class Emloyees implements UserList {
    public function getUsers() {
        $users = new UsersCollection();
        //...
        return $users;
    }
}

class Directors implements UserList {
    public function getUsers() {
        $users = array();
        //...
        return $users;
    }
}
```

在这里，我们看到一个边缘案例的简单例子。在两个派生类上调用`getUsers`将返回一个我们可以循环遍历的结果。然而，PHP 开发人员倾向于经常在数组结构上使用`count`方法，并且在`Employees`实例上使用`getUsers`结果将不起作用。这是因为`Employees`类返回实现了`Iterator`的`UsersCollection`，而不是实际的数组结构。由于`UsersCollection`没有实现`Countable`，我们无法在其上使用`count`，这可能会导致潜在的错误。

我们还可以在派生类对方法参数的处理上发现 LSP 违规的情况。这些通常可以通过使用`type`运算符的实例来发现，如下例所示：

```php
interface LoggerProcessor {
    public function log(LoggerInterface $logger);
}

class XmlLogger implements LoggerInterface {
    // Implementation...
}

class JsonLogger implements LoggerInterface {
    // Implementation...
}

class FileLogger implements LoggerInterface {
    // Implementation...
}

class Processor implements LoggerProcessor {
    public function log(LoggerInterface $logger) {
        if ($logger instanceof XmlLogger) {
            throw new \Exception('This processor does not work with XmlLogger');
        } else {
            // Implementation...
        }
    }
}
```

在这里，派生类`Processor`对方法参数施加了限制，而它应该接受符合`LoggerInterface`的一切。通过变得不那么宽容，它改变了基类 implied 的行为，在这种情况下是`LoggerInterface`。

所述示例仅仅是构成 LSP 违规的一部分。为了满足这一原则，我们需要确保派生类不以任何方式改变基类所施加的行为。

# 接口隔离原则

**接口隔离原则**规定客户端只应实现它们实际使用的接口。它们不应被强制实现它们不使用的接口。根据维基百科上的定义：

> *"许多特定于客户端的接口比一个通用接口更好"*

这意味着我们应该将大而臃肿的接口分割成几个小而轻的接口，将其分离，使得较小的接口基于一组方法，每个方法提供一种特定的功能。

让我们来看一个违反 ISP 的漏洞抽象：

```php
interface Appliance {
    public function powerOn();
    public function powerOff();
    public function bake();
    public function mix();
    public function wash();

}

class Oven implements Appliance {
    public function powerOn() { /* Implement ... */ }
    public function powerOff() { /* Implement ... */ }
    public function bake() { /* Implement... */ }
    public function mix() { /* Nothing to implement ... */ }
    public function wash() { /* Cannot implement... */ }
}

class Mixer implements Appliance {
    public function powerOn() { /* Implement... */ }
    public function powerOff() { /* Implement... */ }
    public function bake() { /* Cannot implement... */ }
    public function mix() { /* Implement... */ }
    public function wash() { /* Cannot implement... */ }
}

class WashingMachine implements Appliance {
    public function powerOn() { /* Implement... */ }
    public function powerOff() { /* Implement... */ }
    public function bake() { /* Cannot implement... */ }
    public function mix() { /* Cannot implement... */ }
    public function wash() { /* Implement... */ }
}
```

在这里，我们有一个接口为几个与电器相关的方法设置要求。然后我们有几个实现该接口的类。问题是非常明显的；并非所有的电器都可以被挤进同一个接口。强迫洗衣机实现烘烤和混合方法是没有意义的。这些方法需要分别分成自己的接口。这样具体的电器类只需要实现实际有意义的方法。

# 依赖反转原则

**依赖反转原则**规定实体应该依赖于抽象而不是具体实现。也就是说，高级模块不应该依赖于低级模块，而应该依赖于抽象。根据维基百科上的定义：

> *"一个应该依赖于抽象。不要依赖于具体实现。"*

这个原则很重要，因为它在解耦我们的软件中起着重要作用。

以下是一个违反 DIP 的类的示例：

```php
class Mailer {
    // Implementation...
}

class NotifySubscriber {
    public function notify($emailTo) {
        $mailer = new Mailer();
        $mailer->send('Thank you for...', $emailTo);
    }
}
```

在这里，我们可以看到`NotifySubscriber`类中的`notify`方法编写了对`Mailer`类的依赖。这导致了紧密耦合的代码，这正是我们试图避免的。为了纠正问题，我们可以通过类构造函数传递依赖，或者可能通过其他方法。此外，我们应该远离具体类依赖，转向抽象类依赖，就像在这里所示的纠正示例中所示的那样：

```php
interface MailerInterface {
    // Implementation...
}

class Mailer implements MailerInterface {
    // Implementation...
}

class NotifySubscriber {
    private $mailer;

    public function __construct(MailerInterface $mailer) {
        $this->mailer = $mailer;
    }

    public function notify($emailTo) {
        $this->mailer->send('Thank you for...', $emailTo);
    }
}
```

在这里，我们看到一个依赖通过构造函数注入。注入是通过类型提示接口和实际的具体类来抽象的。这使得我们的代码耦合度较低。DIP 可以在任何时候使用，当一个类需要调用另一个类的方法，或者我们应该说向其发送消息时。

# 总结

在模块化开发方面，可扩展性是需要不断考虑的事情。编写一个将自己锁定的代码很可能会导致将来无法将其与其他项目或库集成。虽然 SOLID 设计原则可能看起来有些过分，但积极应用这些原则很可能会导致组件易于在时间上进行维护和扩展。

采用 SOLID 原则进行类设计，可以使我们的代码为未来的变化做好准备。它通过将这些变化局部化和最小化在我们的类中，使得使用它的任何集成都不会感受到变化的重大影响。

在接下来的章节中，我们将研究定义我们的应用程序规范，我们将在所有其他章节中构建它。
