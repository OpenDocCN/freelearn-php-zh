# 第六章：用依赖注入替换 new

即使我们在类中删除了所有`global`调用，它们可能仍然保留其他隐藏的依赖关系。特别是，我们可能在不合适的位置创建新的对象实例，将类紧密耦合在一起。这些事情使得编写测试和查看内部依赖关系变得更加困难。

# 嵌入式实例化

在假设的*ItemsGateway*类中转换`global`调用后，我们可能会得到类似这样的代码：

```php
**classes/ItemsGateway.php**
1 <?php
2 class ItemsGateway
3 {
4 protected $db_host;
5 protected $db_user;
6 protected $db_pass;
7 protected $db;
8
9 public function __construct($db_host, $db_user, $db_pass)
10 {
11 $this->db_host = $db_host;
12 $this->db_user = $db_user;
13 $this->db_pass = $db_pass;
14 $this->db = new Db($this->db_host, $this->db_user, $this->db_pass);
15 }
16
17 public function selectAll()
18 {
19 $rows = $this->db->query("SELECT * FROM items ORDER BY id");
20 $item_collection = array();
21 foreach ($rows as $row) {
22 $item_collection[] = new Item($row);
23 }
24 return $item_collection;
25 }
26 }
27 ?>
```

这里有两个依赖注入问题：

1.  首先，该类可能是从一个使用`global $db_host`，`$db_user`，`$db_pass`的函数转换而来，然后在内部构造了一个`Db`对象。我们最初删除`global`调用时摆脱了全局变量，但是保留了这个`Db`依赖。这就是我们所谓的一次性创建依赖。

1.  其次，`selectAll()`方法创建新的`Item`对象，因此依赖于`Item`类。我们无法从类的外部看到这种依赖关系。这就是我们所谓的重复创建依赖。

### 注意

据我所知，一次性创建依赖和重复创建依赖这两个术语并不是行业标准术语。它们仅适用于本书的目的。如果您知道有类似概念的行业标准术语，请通知作者。

依赖注入的目的是从外部推送依赖项，从而揭示我们类中的依赖关系。在类内部使用`new`关键字与这个想法相悖，因此我们需要通过代码库来删除非`Factory`类中的该关键字，并注入必要的依赖项。

### 注意

**什么是工厂对象？**

依赖注入的关键之一是一个对象可以创建其他对象，*或者*它可以操作其他对象，*但不能两者兼而有之*。每当我们需要在另一个对象内部创建一个对象时，我们让`Factory`来完成这项工作，`Factory`有一个`newInstance()`方法，并将该`Factory`注入到需要进行创建的对象中。`new`关键字仅限于在*Factory*对象内部使用。这使我们能够随时切换*Factory*对象，以便创建不同类型的对象。

# 替换过程

接下来的步骤是从非*Factory*类中删除所有`new`关键字的使用，并注入必要的依赖项。我们还将根据需要使用*Factory*对象来处理重复创建依赖。这是我们将遵循的一般流程：

1.  查找带有`new`关键字的类。如果该类已经是一个`Factory`，我们可以忽略它并继续。

1.  对于类中的每个一次性创建：

+   将每个实例化提取到构造函数参数中。

+   将构造函数参数分配给属性。

+   删除仅用于`new`调用的构造函数参数和类属性。

1.  对于类中的每个重复创建：

+   将每个创建代码块提取到一个新的`Factory`类中。

+   为每个`Factory`创建一个构造函数参数，并将其分配给一个属性。

+   修改类中先前的创建逻辑，以使用*Factory*。

1.  修改项目中对修改后的类的所有实例化调用，以便将必要的依赖对象传递给构造函数。

1.  抽查，提交，推送，并通知 QA。

1.  重复处理下一个不在*Factory*对象内部的`new`调用。

## 查找`new`关键字

与其他步骤一样，我们首先使用项目范围的搜索工具，使用以下正则表达式在我们的类文件中查找`new`关键字：

搜索：

```php
**new\s+**

```

我们有两种创建方式要查找：一次性和重复。我们如何区分？一般来说：

+   如果实例化分配给一个属性，并且从未更改，那么它很可能是一次性创建。通常，我们在构造函数中看到这一点。

+   如果实例化发生在非构造方法中，很可能是重复创建，因为它在每次调用方法时都会发生。

## 将一次性创建提取到依赖注入

假设我们在搜索`new`关键字时找到了上面列出的*ItemsGateway*类，并遇到了构造函数：

```php
**classes/ItemsGateway.php**
1 <?php
2 class ItemsGateway
3 {
4 protected $db_host;
5 protected $db_user;
6 protected $db_pass;
7 protected $db;
8
9 public function __construct($db_host, $db_user, $db_pass)
10 {
11 $this->db_host = $db_host;
12 $this->db_user = $db_user;
13 $this->db_pass = $db_pass;
14 $this->db = new Db($this->db_host, $this->db_user, $this->db_pass);
15 }
16 // ...
17 }
18 ?>
```

在检查类时，我们发现`$this->db`被分配为一个属性。这似乎是一次性创建。此外，似乎至少有一些现有的构造函数参数仅用于`Db`实例化。

我们继续完全删除实例化调用，以及仅用于实例化调用的属性，并用单个 Db 参数替换构造函数参数：

```php
classes/ItemsGateway.php
1 <?php
2 class ItemsGateway
3 {
4 protected $db;
5
6 public function __construct(Db $db)
7 {
8 $this->db = $db;
9 }
10
11 // ...
12 }
13 ?>
```

## 将重复创建提取到工厂

如果我们发现重复创建而不是一次性创建，我们有不同的任务要完成。让我们返回到*ItemsGateway*类，但这次我们将查看`selectAll()`方法。

```php
**classes/ItemsGateway.php**
1 <?php
2 class ItemsGateway
3 {
4 protected $db;
5
6 public function __construct(Db $db)
7 {
8 $this->db = $db;
9 }
10
11 public function selectAll()
12 {
13 $rows = $this->db->query("SELECT * FROM items ORDER BY id");
14 $item_collection = array();
15 foreach ($rows as $row) {
16 $item_collection[] = new Item($row);
17 }
18 return $item_collection;
19 }
20 }
21 ?>
```

我们可以看到`new`关键字在方法内的循环中出现。这显然是重复创建的情况。

首先，我们将创建代码提取到自己的新类中。因为代码创建了一个`Item`对象，我们将称该类为*ItemFactory*。在其中，我们将创建一个方法来返回`Item`对象的新实例：

```php
classes/ItemFactory.php
1 <?php
2 class ItemFactory
3 {
4 public function newInstance(array $item_data)
5 {
6 return new Item($item_data);
7 }
8 }
9 ?>
```

### 注意

*Factory*的唯一目的是创建新对象。它不应该有任何其他功能。将其他行为放在`Factory`中以集中常见逻辑将是诱人的。抵制这种诱惑！

现在我们已经将创建代码提取到一个单独的类中，我们将修改*ItemsGateway*以接受一个*ItemFactory*参数，将其保留在属性中，并使用*ItemFactory*来创建*Item*对象。

```php
**classes/ItemsGateway.php**
1 <?php
2 class ItemsGateway
3 {
4 protected $db;
5
6 protected $item_factory;
7
8 public function __construct(Db $db, ItemFactory $item_factory)
9 {
10 $this->db = $db;
11 $this->item_factory = $item_factory;
12 }
13
14 public function selectAll()
15 {
16 $rows = $this->db->query("SELECT * FROM items ORDER BY id");
17 $item_collection = array();
18 foreach ($rows as $row) {
19 $item_collection[] = $this->item_factory->newInstance($row);
20 }
21 return $item_collection;
22 }
23 }
24 ?>
```

# 更改实例化调用

因为我们已经改变了构造函数的签名，所有现有的*ItemsGateway*实例化现在都已经失效。我们需要找到代码中实例化*ItemsGateway*类的所有地方，并将实例化更改为传递一个正确构造的`Db`对象和一个*ItemFactory*。

为此，我们使用项目范围的搜索工具，使用正则表达式搜索我们更改后的类名：

搜索：

```php
**new\s+ItemsGateway\(**

```

这样做将给我们一个项目中所有实例化的列表。我们需要审查每个结果，并手动更改它们以实例化依赖项并将它们传递给*ItemsGateway*。

例如，如果搜索结果中的页面脚本看起来像这样：

```php
**page_script.php**
1 <?php
2 // $db_host, $db_user, and $db_pass are defined in the setup file
3 require 'includes/setup.php';
4
5 // ...
6
7 // create a gateway
8 $items_gateway = new ItemsGateway($db_host, $db_user, $db_pass);
9
10 // ...
11 ?>
```

我们需要将其更改为更像这样的内容：

```php
**page_script.php**
1 <?php
2 // $db_host, $db_user, and $db_pass are defined in the setup file
3 require 'includes/setup.php';
4
5 // ...
6
7 // create a gateway with its dependencies
8 $db = new Db($db_host, $db_user, $db_pass);
9 $item_factory = new ItemFactory;
10 $items_gateway = new ItemsGateway($db, $item_factory);
11
12 // ...
13 ?>
```

对更改后的类的每个实例化都要这样做。

## 抽查、提交、推送、通知 QA

现在我们已经更改了整个代码库中的类和实例化的类，我们需要确保我们的旧应用程序正常工作。同样，我们没有建立正式的测试流程，因此我们需要运行或以其他方式调用使用更改后的类的应用程序部分，并查找错误。

一旦我们确信应用程序仍然正常运行，我们就提交代码，将其推送到我们的中央仓库，并通知 QA 我们已经准备好让他们测试我们的新添加内容。

## 做...直到

在类中搜索下一个`new`关键字，并重新开始整个过程。当我们发现`new`关键字仅存在于*Factory*类中时，我们的工作就完成了。

## 常见问题

### 异常和 SPL 类怎么办？

在本章中，我们集中于删除所有对`new`关键字的使用，除了*Factory*对象内部。我相信有两个合理的例外情况：*Exception*类本身，以及某些内置的 PHP 类，例如 SPL 类。

按照本章描述的过程创建一个`ExceptionFactory`类，将其注入到抛出异常的对象中，然后使用`ExceptionFactory`创建要抛出的`Exception`对象是完全一致的。即使对我来说，这似乎有点过分。我认为`Exception`对象是规则之外的一个合理例外。

同样，我认为内置的 PHP 类通常也是规则的例外。虽然拥有一个*ArrayObjectFactory*或者*ArrayIteratorFactory*来创建 SPL 本身提供的*ArrayObject*和*ArrayIterator*类会很好，但可能有点过分。通常直接在使用它们的对象内部创建这些类型的对象是可以的。

然而，我们需要小心。在需要的类内部直接创建像`PDO`连接这样复杂或者强大的对象可能会超出我们的范围。很难描述一个好的经验法则；当有疑问时，最好依赖注入。

### 中间依赖怎么样？

有时我们会发现类有依赖项，而这些依赖项本身也有依赖项。这些中间依赖项被传递给外部类，只是为了让内部对象可以用它们实例化。

例如，假设我们有一个需要*ItemsGateway*的`Service`类，它本身需要一个`Db`连接。在移除`global`变量之前，`Service`类可能看起来像这样：

```php
**classes/Service.php**
1 <?php
2 class Service
3 {
4 public function doThis()
5 {
6 // ...
7 $db = global $db;
8 $items_gateway = new ItemsGateway($db);
9 $items = $items_gateway->selectAll();
10 // ...
11 }
12
13 public function doThat()
14 {
15 // ...
16 $db = global $db;
17 $items_gateway = new ItemsGateway($db);
18 $items = $items_gateway->selectAll();
19 // ...
20 }
21 }
22 ?>
```

在移除`global`变量之后，我们只剩下一个`new`关键字，但我们仍然需要`Db`对象作为*ItemsGateway*的依赖：

```php
**classes/Service.php**
1 <?php
2 class Service
3 {
4 protected $db;
5
6 public function __construct(Db $db)
7 {
8 $this->db = $db;
9 }
10
11 public function doThis()
12 {
13 // ...
14 $items_gateway = new ItemsGateway($this->db);
15 $items = $items_gateway->selectAll();
16 // ...
17 }
18
19 public function doThat()
20 {
21 // ...
22 $items_gateway = new ItemsGateway($this->db);
23 $items = $items_gateway->selectAll();
24 // ...
25 }
26 }
27 ?>
```

在这里如何成功移除`new`关键字？*ItemsGateway*需要一个`Db`连接。`Service`并不直接使用`Db`连接；它只用于构建*ItemsGateway*。

在这种情况下的解决方案是注入一个完全构建的*ItemsGateway*。首先，我们修改`Service`类以接收它的真正依赖，*ItemsGateway*：

```php
**classes/Service.php**
1 <?php
2 class Service
3 {
4 protected $items_gateway;
5
6 public function __construct(ItemsGateway $items_gateway)
7 {
8 $this->items_gateway = $items_gateway;
9 }
10
11 public function doThis()
12 {
13 // ...
14 $items = $this->items_gateway->selectAll();
15 // ...
16 }
17
18 public function doThat()
19 {
20 // ...
21 $items = $this->items_gateway->selectAll();
22 // ...
23 }
24 }
25 ?>
```

其次，在整个传统应用程序中，我们改变了所有*Service*的实例化，以传递*ItemsGateway*。

例如，当到处使用`global`变量时，页面脚本可能会这样做：

```php
**page_script.php (globals)**
1 <?php
2 // defines the $db connection
3 require 'includes/setup.php';
4
5 // creates the service with globals
6 $service = new Service;
7 ?>
```

然后我们改变了它，以在移除全局变量后注入中间依赖：

```php
**page_script.php (intermediary dependency)**
1 <?php
2 // defines the $db connection
3 require 'includes/setup.php';
4
5 // inject the Db object for the internal ItemsGateway creation
6 $service = new Service($db);
7 ?>
```

但我们最终应该改变它以注入真正的依赖：

```php
**page_script.php (real dependency)**
1 <?php
2 // defines the $db connection
3 require 'includes/setup.php';
4
5 // create the gateway dependency and then the service
6 $items_gateway = new ItemsGateway($db);
7 $service = new Service($items_gateway);
8 ?>
```

### 这是不是很多代码？

我有时听到抱怨，使用依赖注入意味着要写很多额外的代码来做以前的事情。

没错。像这样的调用，类内部管理自己的依赖关系。

没有依赖注入：

```php
1 <?php
2 $items_gateway = new ItemsGateway;
3 ?>
```

这显然比通过创建依赖项并使用`Factory`对象使用依赖注入的代码要少。

使用依赖注入：

```php
1 <?php
2 $db = new Db($db_host, $db_user, $db_pass);
3 $item_factory = new ItemFactory;
4 $items_gateway = new ItemsGateway($db, $item_factory);
5 ?>
```

然而，真正的问题不是更多的代码。问题是更易于测试，更清晰，更解耦。

在查看第一个例子时，我们如何知道*ItemsGateway*需要什么来运行？它会影响系统的哪些其他部分？没有检查整个类并寻找`global`和`new`关键字是非常困难的。

在查看第二个例子时，很容易知道类需要什么来运行，我们可以期望它创建什么，以及它与系统的哪些部分交互。这些额外的东西使得以后更容易测试这个类。

### 工厂应该创建集合吗？

在上面的例子中，我们的`Factory`类只创建一个对象的`newInstance()`。如果我们经常创建对象的集合，可能合理地向我们的`Factory`添加一个`newCollection()`方法。例如，给定我们上面的*ItemFactory*，我们可能会做如下事情：

```php
**classes/ItemFactory.php**
1 <?php
2 class ItemFactory
3 {
4 public function newInstance(array $item_data)
5 {
6 return new Item($item_data);
7 }
8
9 public function newCollection(array $items_data)
10 {
11 $collection = array();
12 foreach ($items_data as $item_data) {
13 $collection[] = $this->newInstance($item_data);
14 }
15 return $collection;
16 }
17 }
18 ?>
```

我们甚至可以创建一个`ItemCollection`类来代替使用数组进行集合。如果是这样，我们可以在`ItemFactory`内部使用`new`关键字来创建`ItemCollection`实例是合理的。（这里省略了`ItemCollection`类）。

```php
**classes/ItemFactory.php**
1 <?php
2 class ItemFactory
3 {
4 public function newInstance(array $item_data)
5 {
6 return new Item($item_data);
7 }
8
9 public function newCollection(array $item_rows)
10 {
11 $collection = new ItemCollection;
12 foreach ($item_rows as $item_data) {
13 $item = $this->newInstance($item_data);
14 $collection->append($item);
15 }
16 return $collection;
17 }
18 }
19 ?>
```

事实上，我们可能希望有一个单独的*ItemCollectionFactory*，使用一个注入的*ItemFactory*来创建 Item 对象，具有自己的`newInstance()`方法来返回一个新的*ItemCollection*。

有许多关于正确使用`工厂`对象的变种。关键是将对象创建（以及相关操作）与对象操作分开。

## 我们能自动化所有这些注入吗？

到目前为止，我们一直在进行手动注入依赖，我们自己创建依赖，然后在创建所需的对象时注入它们。这可能是一个乏味的过程。谁愿意一遍又一遍地创建`Db`对象，只是为了将其注入到各种`Gateway`类中？难道没有一种自动化的方法吗？

有的，就是叫做`容器`。`容器`可能有各种同义词，表示它的用途。依赖注入`容器`旨在始终且仅在非`工厂`类外部使用，而以`服务定位器`为名的相同`容器`实现旨在在非`工厂`对象内部使用。

使用`容器`带来了明显的优势：

+   我们可以创建共享服务，只有在调用它们时才实例化。例如，`容器`可以容纳一个`Db`实例，只有当我们要求`容器`获取数据库连接时才会创建；连接被创建一次，然后一遍又一遍地重复使用。

+   我们可以将复杂的对象创建放在`容器`内部，需要多个服务作为构造函数参数的对象可以从`容器`内部检索这些服务，并在它们自己的创建逻辑中使用。

但是使用`容器`也有缺点：

+   我们必须彻底改变我们对对象创建的思考方式，以及这些对象在应用程序中的位置。最终这是件好事，但在过渡期可能会有麻烦。

+   作为服务定位器使用的`容器`用一个新的花哨玩具取代了我们的`global`变量，它有许多与`global`相同的问题。`容器`隐藏了依赖，因为它只能从需要依赖的类内部调用。

在我们现代化遗留应用程序的这个阶段，很容易开始使用`容器`来自动化依赖注入。我建议我们现在不要添加，因为我们的遗留应用程序还有很多需要现代化的地方。我们最终会添加，但这将是我们现代化过程的最后一步。

# 回顾和下一步

我们现在在现代化我们的应用程序上取得了巨大的进步。删除`global`和`new`关键字，改用依赖注入已经改善了代码库的质量，并且使得追踪错误变得更加容易，因为在这里修改变量不再会导致远处的变量受到影响。我们的页面脚本可能会有些更长，因为我们必须创建依赖，但现在我们可以更清楚地看到我们与系统的哪些部分进行交互。

我们的下一步是检查我们新重构的类，并开始为它们编写测试。这样，当我们开始对类进行更改时，我们将知道是否破坏了以前存在的行为。
