# 第四章. 使用 PHP 面向对象编程

在本章中我们将涵盖：

+   开发类

+   扩展类

+   使用静态属性和方法

+   使用命名空间

+   定义可见性

+   使用接口

+   使用 traits

+   实现匿名类

# 介绍

在本章中，我们将考虑利用 PHP 7.0、7.1 及以上版本中可用的**面向对象编程**（**OOP**）功能的相关内容。PHP 7.x 中大部分的 OOP 功能也适用于 PHP 5.6。PHP 7 引入的新功能是支持**匿名类**。在 PHP 7.1 中，您可以修改类常量的可见性。

### 注意

另一个全新的功能是**捕获**某些类型的错误。这在第十三章*最佳实践、测试和调试*中更详细地讨论。

# 开发类

传统的开发方法是将类放入自己的文件中。通常，类包含实现单一目的的逻辑。类进一步分解为自包含的函数，这些函数被称为**方法**。在类内定义的变量被称为**属性**。建议同时开发一个测试类，这是在第十三章*最佳实践、测试和调试*中更详细讨论的主题。

## 如何做...

1.  创建一个文件来包含类定义。为了自动加载的目的，建议文件名与类名匹配。在文件顶部，在关键字`class`之前，添加一个**DocBlock**。然后您可以定义属性和方法。在这个例子中，我们定义了一个类`Test`。它有一个属性`$test`和一个方法`getTest()`：

```php
    <?php
    declare(strict_types=1);
    /**
     * This is a demonstration class.
     *
     * The purpose of this class is to get and set 
     * a protected property $test
     *
     */
    class Test
    {

      protected $test = 'TEST';

      /**
       * This method returns the current value of $test
       *
       * @return string $test
       */
      public function getTest() : string
      {
        return $this->test;
      }

      /**
       * This method sets the value of $test
       *
       * @param string $test
       * @return Test $this
       */
      public function setTest(string $test)
      {
        $this->test = $test;
        return $this;
      }
    }
    ```

### 提示

**最佳实践**

将文件命名为类名被认为是最佳实践。虽然 PHP 中的类名不区分大小写，但进一步被认为是最佳实践的是使用大写字母作为类名的第一个字母。您不应该在类定义文件中放置可执行代码。

每个类都应该在关键字`class`之前包含一个**DocBlock**。在 DocBlock 中，您应该包括一个关于类目的简短描述。空一行，然后包括更详细的描述。您还可以包括`@`标签，如`@author`、`@license`等。每个方法也应该在之前包含一个标识方法目的的 DocBlock，以及它的传入参数和返回值。

1.  可以在一个文件中定义多个类，但这不被认为是最佳实践。在这个例子中，我们创建一个文件`NameAddress.php`，其中定义了两个类，`Name`和`Address`：

```php
    <?php
    declare(strict_types=1);
    class Name
    {

      protected $name = '';

      public function getName() : string
      {
        return $this->name;
      }

      public function setName(string $name)
      {
        $this->name = $name;

        return $this;
      }
    }

    class Address
    {

      protected $address = '';

      public function getAddress() : string
      {
        return $this->address;
      }

      public function setAddress(string $address)
      {
        $this->address = $address;
        return $this;
      }
    }
    ```

### 提示

虽然您可以在单个文件中定义多个类，如前面的代码片段所示，但这并不被认为是最佳实践。这不仅会使文件的逻辑纯度降低，而且会使自动加载变得更加困难。

1.  类名不区分大小写。重复将被标记为错误。在这个例子中，在一个名为`TwoClass.php`的文件中，我们定义了两个类，`TwoClass`和`twoclass`：

```php
    <?php
    class TwoClass
    {
      public function showOne()
      {
        return 'ONE';
      }
    }

    // a fatal error will occur when the second class definition is parsed
    class twoclass
    {
      public function showTwo()
      {
        return 'TWO';
      }
    }
    ```

1.  PHP 7.1 已解决了在使用关键字`$this`时的不一致行为。虽然在 PHP 7.0 和 PHP 5.x 中允许使用，但在 PHP 7.1 中，如果`$this`被用作：

+   一个参数

+   一个`static`变量

+   一个`global`变量

+   在`try...catch`块中使用的变量

+   在`foreach()`中使用的变量

+   作为`unset()`的参数

+   作为一个变量（即，`$a = 'this'; echo $$a`）

+   通过引用间接使用

1.  如果您需要创建一个对象实例，但不想定义一个离散的类，您可以使用内置于 PHP 中的通用`stdClass`。`stdClass`允许您在不必定义一个扩展`stdClass`的离散类的情况下*即兴*定义属性：

```php
    $obj = new stdClass();
    ```

1.  这个功能在 PHP 的许多不同地方使用。例如，当您使用**PHP 数据对象**（**PDO**）来进行数据库查询时，其中的一个获取模式是`PDO::FETCH_OBJ`。这种模式返回`stdClass`的实例，其中的属性代表数据库表列：

```php
    $stmt = $connection->pdo->query($sql);
    $row  = $stmt->fetch(PDO::FETCH_OBJ);
    ```

## 它是如何工作的...

取出前面代码片段中的`Test`类的例子，并将代码放入一个名为`Test.php`的文件中。创建另一个名为`chap_04_oop_defining_class_test.php`的文件。添加以下代码：

```php
require __DIR__ . '/Test.php';

$test = new Test();
echo $test->getTest();
echo PHP_EOL;

$test->setTest('ABC');
echo $test->getTest();
echo PHP_EOL;
```

输出将显示`$test`属性的初始值，然后通过调用`setTest()`修改的新值：

![它是如何工作的...](img/B05314_04_01.jpg)

下一个例子让您在一个名为`NameAddress.php`的单个文件中定义两个类，`Name`和`Address`。您可以使用以下代码调用和使用这两个类：

```php
require __DIR__ . '/NameAddress.php';

$name = new Name();
$name->setName('TEST');
$addr = new Address();
$addr->setAddress('123 Main Street');

echo $name->getName() . ' lives at ' . $addr->getAddress();
```

### 注意

虽然 PHP 解释器没有生成错误，但通过定义多个类，文件的逻辑纯度受到了损害。此外，文件名与类名不匹配，这可能会影响自动加载的能力。

接下来的例子的输出如下所示：

![它是如何工作的...](img/B05314_04_02.jpg)

步骤 3 还展示了一个文件中的两个类定义。然而，在这种情况下，目标是演示 PHP 中的类名是不区分大小写的。将代码放入一个名为`TwoClass.php`的文件中。当您尝试包含该文件时，将生成一个错误：

![它是如何工作的...](img/B05314_04_03.jpg)

为了演示直接使用`stdClass`，创建一个实例，为属性赋值，并使用`var_dump()`来显示结果。要查看`stdClass`在内部的使用方式，使用`var_dump()`来显示`PDO`查询的结果，其中获取模式设置为`FETCH_OBJ`。

输入以下代码：

```php
$obj = new stdClass();
$obj->test = 'TEST';
echo $obj->test;
echo PHP_EOL;

include (__DIR__ . '/../Application/Database/Connection.php');
$connection = new Application\Database\Connection(
  include __DIR__ . DB_CONFIG_FILE);

$sql  = 'SELECT * FROM iso_country_codes';
$stmt = $connection->pdo->query($sql);
$row  = $stmt->fetch(PDO::FETCH_OBJ);
var_dump($row);
```

以下是输出：

![它是如何工作的...](img/B05314_04_04.jpg)

## 参见...

有关 PHP 7.1 中关键字`$this`的改进的更多信息，请参阅[`wiki.php.net/rfc/this_var`](https://wiki.php.net/rfc/this_var)。

# 扩展类

开发人员使用 OOP 的主要原因之一是因为它能够重用现有的代码，同时又能够添加或覆盖功能。在 PHP 中，关键字`extends`用于在类之间建立父/子关系。

## 如何做...

1.  在`child`类中，使用关键字`extends`来建立继承关系。在接下来的例子中，`Customer`类扩展了`Base`类。`Customer`的任何实例都将继承可见的方法和属性，这里是`$id`，`getId()`和`setId()`：

```php
    class Base
    {
      protected $id;
      public function getId()
      {
        return $this->id;
      }
      public function setId($id)
      {
        $this->id = $id;
      }
    }

    class Customer extends Base
    {
      protected $name;
      public function getName()
      {
        return $this->name;
      }
      public function setName($name)
      {
        $this->name = $name;
      }
    }
    ```

1.  您可以通过将其标记为`abstract`来强制任何使用您的类的开发人员定义一个方法。在这个例子中，`Base`类将`validate()`方法定义为`abstract`。它必须是抽象的原因是因为从父类`Base`的角度来确定子类如何被验证是不可能的：

```php
    abstract class Base
    {
      protected $id;
      public function getId()
      {
        return $this->id;
      }
      public function setId($id)
      {
        $this->id = $id;
      }
      public function validate();
    }
    ```

### 注意

如果一个类包含一个**抽象方法**，那么这个类本身必须声明为`abstract`。

1.  PHP 只支持单一继承线。下一个例子展示了一个名为`Member`的类，它从`Customer`继承。`Customer`又从`Base`继承：

```php
    class Base
    {
      protected $id;
      public function getId()
      {
        return $this->id;
      }
      public function setId($id)
      {
        $this->id = $id;
      }
    }

    class Customer extends Base
    {
      protected $name;
      public function getName()
      {
        return $this->name;
      }
      public function setName($name)
      {
        $this->name = $name;
      }
    }

    class Member extends Customer
    {
      protected $membership;
      public function getMembership()
      {
        return $this->membership;
      }
      public function setMembership($memberId)
      {
        $this->membership = $memberId;
      }
    }
    ```

1.  为了满足类型提示，目标类的任何子类都可以使用。下面的代码片段中显示的`test()`函数需要`Base`类的一个实例作为参数。继承线中的任何类都可以被接受为参数。传递给`test()`的任何其他内容都会引发`TypeError`：

```php
    function test(Base $object)
    {
      return $object->getId();
    }
    ```

## 它是如何工作的...

在第一个要点中，定义了一个`Base`类和一个`Customer`类。为了演示，将这两个类定义放入一个名为`chap_04_oop_extends.php`的单个文件中，并添加以下代码：

```php
$customer = new Customer();
$customer->setId(100);
$customer->setName('Fred');
var_dump($customer);
```

请注意，`$id`属性和`getId()`和`setId()`方法从父类`Base`继承到子类`Customer`：

![它是如何工作的...](img/B05314_04_06.jpg)

为了说明`abstract`方法的使用，想象一下你希望为任何扩展`Base`的类添加某种验证能力。问题是不知道在继承类中可能会验证什么。唯一确定的是你必须有验证能力。

使用前面解释中提到的相同的`Base`类，并添加一个新的方法`validate()`。将该方法标记为`abstract`，不定义任何代码。注意当子`Customer`类扩展`Base`时会发生什么。

![它是如何工作的...](img/B05314_04_07.jpg)

如果你将`Base`类标记为`abstract`，但未在子类中定义`validate()`方法，将生成*相同的错误*。最后，继续在子`Customer`类中实现`validate()`方法：

```php
class Customer extends Base
{
  protected $name;
  public function getName()
  {
    return $this->name;
  }
  public function setName($name)
  {
    $this->name = $name;
  }
  public function validate()
  {
    $valid = 0;
    $count = count(get_object_vars($this));
    if (!empty($this->id) &&is_int($this->id)) $valid++;
    if (!empty($this->name) 
    &&preg_match('/[a-z0-9 ]/i', $this->name)) $valid++;
    return ($valid == $count);
  }
}
```

然后你可以添加以下过程代码来测试结果：

```php
$customer = new Customer();

$customer->setId(100);
$customer->setName('Fred');
echo "Customer [id]: {$customer->getName()}" .
     . "[{$customer->getId()}]\n";
echo ($customer->validate()) ? 'VALID' : 'NOT VALID';
$customer->setId('XXX');
$customer->setName('$%£&*()');
echo "Customer [id]: {$customer->getName()}"
  . "[{$customer->getId()}]\n";
echo ($customer->validate()) ? 'VALID' : 'NOT VALID';
```

这是输出：

![它是如何工作的...](img/B05314_04_08.jpg)

展示单行继承，将一个新的`Member`类添加到前面步骤 1 中显示的`Base`和`Customer`的第一个示例中：

```php
class Member extends Customer
{
  protected $membership;
  public function getMembership()
  {
    return $this->membership;
  }
  public function setMembership($memberId)
  {
    $this->membership = $memberId;
  }
}
```

创建一个`Member`的实例，并注意在下面的代码中，所有属性和方法都可以从每个继承的类中使用，即使不是直接继承的。

```php
$member = new Member();
$member->setId(100);
$member->setName('Fred');
$member->setMembership('A299F322');
var_dump($member);
```

这是输出：

![它是如何工作的...](img/B05314_04_09.jpg)

现在定义一个名为`test()`的函数，该函数以`Base`的实例作为参数：

```php
function test(Base $object)
{
  return $object->getId();
}
```

注意`Base`，`Customer`和`Member`的实例都是可以接受的参数：

```php
$base = new Base();
$base->setId(100);

$customer = new Customer();
$customer->setId(101);

$member = new Member();
$member->setId(102);

// all 3 classes work in test()
echo test($base)     . PHP_EOL;
echo test($customer) . PHP_EOL;
echo test($member)   . PHP_EOL;
```

这是输出：

![它是如何工作的...](img/B05314_04_10.jpg)

然而，如果你尝试使用不在继承线上的对象实例运行`test()`，将抛出一个`TypeError`：

```php
class Orphan
{
  protected $id;
  public function getId()
  {
    return $this->id;
  }
  public function setId($id)
  {
    $this->id = $id;
  }
}
try {
    $orphan = new Orphan();
    $orphan->setId(103);
    echo test($orphan) . PHP_EOL;
} catch (TypeError $e) {
    echo 'Does not work!' . PHP_EOL;
    echo $e->getMessage();
}
```

我们可以在下面的图片中观察到这一点：

![它是如何工作的...](img/B05314_04_11.jpg)

# 使用静态属性和方法

PHP 允许你访问属性或方法，而不必创建类的实例。用于此目的的关键字是**static**。

## 如何做...

1.  最简单的方法是在声明普通属性或方法时，在声明可见级别后添加`static`关键字。使用`self`关键字在内部引用属性：

```php
    class Test
    {
      public static $test = 'TEST';
      public static function getTest()
      {
        return self::$test;
      }
    }
    ```

1.  `self`关键字将会提前绑定，这会在访问子类中的静态信息时造成问题。如果你绝对需要访问子类的信息，使用`static`关键字代替`self`。这个过程被称为**后期静态绑定**。

1.  在下面的示例中，如果你输出`Child::getEarlyTest()`，输出将是**TEST**。另一方面，如果你运行`Child::getLateTest()`，输出将是**CHILD**。原因是当使用`self`时，PHP 将绑定到*最早*的定义，而对于`static`关键字，将使用*最新*的绑定：

```php
    class Test2
    {
      public static $test = 'TEST2';
      public static function getEarlyTest()
      {
        return self::$test;
      }
      public static function getLateTest()
      {
        return static::$test;
      }
    }

    class Child extends Test2
    {
      public static $test = 'CHILD';
    }
    ```

1.  在许多情况下，**工厂**设计模式与静态方法一起使用，以根据不同的参数生成对象的实例。在这个例子中，定义了一个静态方法`factory()`，它返回一个 PDO 连接：

```php
    public static function factory(
      $driver,$dbname,$host,$user,$pwd,array $options = [])
      {
        $dsn = sprintf('%s:dbname=%s;host=%s', 
        $driver, $dbname, $host);
        try {
            return new PDO($dsn, $user, $pwd, $options);
        } catch (PDOException $e) {
            error_log($e->getMessage);
        }
      }
    ```

## 它是如何工作的...

你可以使用**类解析运算符**"`::"`来引用静态属性和方法。给定之前显示的`Test`类，如果你运行这段代码：

```php
echo Test::$test;
echo PHP_EOL;
echo Test::getTest();
echo PHP_EOL;
```

你会看到这个输出：

![它是如何工作的...](img/B05314_04_13.jpg)

为了说明后期静态绑定，基于之前显示的`Test2`和`Child`类，尝试这段代码：

```php
echo Test2::$test;
echo Child::$test;
echo Child::getEarlyTest();
echo Child::getLateTest();
```

输出说明了`self`和`static`之间的区别。

![它是如何工作的...](img/B05314_04_14.jpg)

最后，为了测试之前显示的`factory()`方法，将代码保存到`Application\Database\Connection`类中，保存在`Application\Database`文件夹中的`Connection.php`文件中。然后你可以尝试这样做：

```php
include __DIR__ . '/../Application/Database/Connection.php';
use Application\Database\Connection;
$connection = Connection::factory(
'mysql', 'php7cookbook', 'localhost', 'test', 'password');
$stmt = $connection->query('SELECT name FROM iso_country_codes');
while ($country = $stmt->fetch(PDO::FETCH_COLUMN)) 
echo $country . '';
```

你将看到从示例数据库中提取的国家列表：

![它是如何工作的...](img/B05314_04_15.jpg)

## 另请参阅

有关后期静态绑定的更多信息，请参阅 PHP 文档中的解释：

[`php.net/manual/en/language.oop5.late-static-bindings.php`](http://php.net/manual/en/language.oop5.late-static-bindings.php)

# 使用命名空间

对于高级 PHP 开发来说，关键的一点是使用命名空间。任意定义的命名空间成为类名的前缀，从而避免了意外类重复的问题，并允许您在开发中拥有非凡的自由度。另一个使用命名空间的好处是，假设它与目录结构匹配，它可以促进自动加载，如第一章中所讨论的*构建基础*。

## 操作步骤...

1.  要在命名空间中定义一个类，只需在代码文件顶部添加关键字`namespace`：

```php
    namespace Application\Entity;
    ```

### 注意

**最佳实践**

与每个文件只有一个类的建议类似，您应该每个文件只有一个命名空间。

1.  在关键字`namespace`之前应该只有一个注释和/或关键字`declare`。

```php
    <?php
    declare(strict_types=1);
    namespace Application\Entity;
    /**
     * Address
     *
     */
    class Address
    {
      // some code
    }
    ```

1.  在 PHP 5 中，如果您需要访问外部命名空间中的类，可以添加一个只包含命名空间的`use`语句。然后，您需要使用命名空间的最后一个组件作为前缀来引用该命名空间内的任何类：

```php
    use Application\Entity;
    $name = new Entity\Name();
    $addr = new Entity\Address();
    $prof = new Entity\Profile();
    ```

1.  或者，您可以明确指定所有三个类：

```php
    use Application\Entity\Name;
    use Application\Entity\Address;
    use Application\Entity\Profile;
    $name = new Name();
    $addr = new Address();
    $prof = new Profile();
    ```

1.  PHP 7 引入了一种称为**group use**的语法改进，大大提高了代码的可读性：

```php
    use Application\Entity\ {
      Name,
      Address,
      Profile
    };
    $name = new Name();
    $addr = new Address();
    $prof = new Profile();
    ```

1.  如第一章中所述，*构建基础*，命名空间是**自动加载**过程的一个组成部分。此示例显示了一个演示自动加载程序，它会回显传递的参数，然后尝试根据命名空间和类名包含一个文件。这假设目录结构与命名空间匹配：

```php
    function __autoload($class)
    {
      echo "Argument Passed to Autoloader = $class\n";
      include __DIR__ . '/../' . str_replace('\\', DIRECTORY_SEPARATOR, $class) . '.php';
    }
    ```

## 工作原理...

为了举例说明，定义一个与`Application\*`命名空间匹配的目录结构。创建一个基础文件夹`Application`，以及一个子文件夹`Entity`。您还可以根据需要包含任何其他章节中使用的子文件夹，比如`Database`和`Generic`：

![工作原理...](img/B05314_04_16.jpg)

接下来，在`Application/Entity`文件夹下分别创建三个`entity`类，每个类都在自己的文件中：`Name.php`，`Address.php`和`Profile.php`。这里只展示`Application\Entity\Name`。`Application\Entity\Address`和`Application\Entity\Profile`将是相同的，只是`Address`有一个`$address`属性，而`Profile`有一个`$profile`属性，每个属性都有适当的`get`和`set`方法：

```php
<?php
declare(strict_types=1);
namespace Application\Entity;
/**
 * Name
 *
 */
class Name
{

  protected $name = '';

  /**
   * This method returns the current value of $name
   *
   * @return string $name
   */
  public function getName() : string
  {
    return $this->name;
  }

  /**
   * This method sets the value of $name
   *
   * @param string $name
   * @return name $this
   */
  public function setName(string $name)
  {
    $this->name = $name;
    return $this;
  }
}
```

然后，您可以使用第一章中定义的自动加载程序，或者使用之前提到的简单自动加载程序。将设置自动加载的命令放在一个文件`chap_04_oop_namespace_example_1.php`中。在此文件中，您可以指定一个`use`语句，只引用命名空间，而不是类名。通过使用命名空间的最后一部分`Entity`作为类名的前缀，创建三个实体类`Name`，`Address`和`Profile`的实例：

```php
use Application\Entity;
$name = new Entity\Name();
$addr = new Entity\Address();
$prof = new Entity\Profile();

var_dump($name);
var_dump($addr);
var_dump($prof);
```

输出如下：

![工作原理...](img/B05314_04_17.jpg)

接下来，使用**另存为**将文件复制到一个名为`chap_04_oop_namespace_example_2.php`的新文件中。将`use`语句更改为以下内容：

```php
use Application\Entity\Name;
use Application\Entity\Address;
use Application\Entity\Profile;
```

现在，您可以仅使用类名创建类实例：

```php
$name = new Name();
$addr = new Address();
$prof = new Profile();
```

当您运行此脚本时，输出如下：

![工作原理...](img/B05314_04_18.jpg)

最后，再次使用**另存为**创建一个新文件`chap_04_oop_namespace_example_3.php`。您现在可以测试 PHP 7 中引入的**group use**功能：

```php
use Application\Entity\ {
  Name,
  Address,
  Profile
};
$name = new Name();
$addr = new Address();
$prof = new Profile();
```

同样，当您运行此代码块时，输出将与前面的输出相同：

![工作原理...](img/B05314_04_19.jpg)

# 定义可见性

欺骗地，*可见性*一词与应用程序安全无关！相反，它只是一种控制代码使用的机制。它可以用来引导经验不足的开发人员远离应该仅在类定义内部调用的方法的*public*使用。

## 如何做...

1.  通过在任何属性或方法定义的前面添加`public`、`protected`或`private`关键字来指示可见性级别。您可以将属性标记为`protected`或`private`，以强制仅通过公共“getter”和“setter”访问。

1.  在此示例中，定义了一个带有受保护属性`$id`的`Base`类。为了访问此属性，定义了“getId（）”和“setId（）”公共方法。受保护方法“generateRandId（）”可以在内部使用，并且在`Customer`子类中继承。此方法不能直接在类定义之外调用。请注意使用新的 PHP 7“random_bytes（）”函数创建随机 ID。

```php
    class Base
    {
      protected $id;
      private $key = 12345;
      public function getId()
      {
        return $this->id;
      }
      public function setId()
      {
        $this->id = $this->generateRandId();
      }
      protected function generateRandId()
      {
        return unpack('H*', random_bytes(8))[1];
      }
    }

    class Customer extends Base
    {
      protected $name;
      public function getName()
      {
        return $this->name;
      }
      public function setName($name)
      {
        $this->name = $name;
      }
    }
    ```

### 注意

**最佳实践**

将属性标记为`protected`，并定义“publicgetNameOfProperty（）”和“setNameOfProperty（）”方法来控制对属性的访问。这些方法被称为“getter”和“setter”。

1.  将属性或方法标记为`private`以防止其被继承或从类定义之外可见。这是创建类作为**单例**的好方法。

1.  下一个代码示例显示了一个名为`Registry`的类，其中只能有一个实例。因为构造函数标记为`private`，所以唯一可以创建实例的方法是通过静态方法“getInstance（）”：

```php
    class Registry
    {
      protected static $instance = NULL;
      protected $registry = array();
      private function __construct()
      {
        // nobody can create an instance of this class
      }
      public static function getInstance()
      {
        if (!self::$instance) {
          self::$instance = new self();
        }
        return self::$instance;
      }
      public function __get($key)
      {
        return $this->registry[$key] ?? NULL;
      }
      public function __set($key, $value)
      {
        $this->registry[$key] = $value;
      }
    }
    ```

### 注意

您可以将方法标记为`final`以防止其被覆盖。将类标记为`final`以防止其被扩展。

1.  通常，类常量被认为具有`public`的可见性级别。从 PHP 7.1 开始，您可以将类常量声明为`protected`或`private`。在以下示例中，`TEST_WHOLE_WORLD`类常量的行为与 PHP 5 中完全相同。接下来的两个常量，`TEST_INHERITED`和`TEST_LOCAL`，遵循与任何`protected`或`private`属性或方法相同的规则：

```php
    class Test
    {

      public const TEST_WHOLE_WORLD  = 'visible.everywhere';

      // NOTE: only works in PHP 7.1 and above
      protected const TEST_INHERITED = 'visible.in.child.classes';

      // NOTE: only works in PHP 7.1 and above
      private const TEST_LOCAL= 'local.to.class.Test.only';

      public static function getTestInherited()
      {
        return static::TEST_INHERITED;
      }

      public static function getTestLocal()
      {
        return static::TEST_LOCAL;
      }

    }
    ```

## 它是如何工作的...

创建一个名为`chap_04_basic_visibility.php`的文件，并定义两个类：`Base`和`Customer`。接下来，编写代码以创建每个实例：

```php
$base     = new Base();
$customer = new Customer();
```

请注意，以下代码可以正常工作，并且实际上被认为是最佳实践：

```php
$customer->setId();
$customer->setName('Test');
echo 'Welcome ' . $customer->getName() . PHP_EOL;
echo 'Your new ID number is: ' . $customer->getId() . PHP_EOL;
```

尽管`$id`是`protected`，但相应的方法“getId（）”和“setId（）”都是`public`，因此可以从类定义外部访问。以下是输出：

![它是如何工作的...](img/B05314_04_20.jpg)

然而，以下代码行将无法工作，因为`private`和`protected`属性无法从类定义之外访问：

```php
echo 'Key (does not work): ' . $base->key;
echo 'Key (does not work): ' . $customer->key;
echo 'Name (does not work): ' . $customer->name;
echo 'Random ID (does not work): ' . $customer->generateRandId();
```

以下输出显示了预期的错误：

![它是如何工作的...](img/B05314_04_21.jpg)

## 另请参阅

有关“getter”和“setter”的更多信息，请参见本章中标题为“使用 getter 和 setter”的配方。有关 PHP 7.1 类常量可见性设置的更多信息，请参见[`wiki.php.net/rfc/class_const_visibility`](https://wiki.php.net/rfc/class_const_visibility)。

# 使用接口

接口是系统架构师的有用工具，通常用于原型设计**应用程序编程接口**（**API**）。接口不包含实际代码，但可以包含方法的名称以及方法签名。

### 注意

所有在“接口”中标识的方法都具有`public`的可见性级别。

## 如何做...

1.  由接口标识的方法不能包含实际代码实现。但是，您可以指定方法参数的数据类型。

1.  在此示例中，`ConnectionAwareInterface`标识了一个方法“setConnection（）”，该方法需要一个`Connection`的实例作为参数：

```php
    interface ConnectionAwareInterface
    {
      public function setConnection(Connection $connection);
    }
    ```

1.  要使用接口，请在定义类的开放行之后添加关键字`implements`。我们定义了两个类，`CountryList`和`CustomerList`，它们都需要通过`setConnection()`方法访问`Connection`类。为了识别这种依赖关系，这两个类都实现了`ConnectionAwareInterface`：

```php
    class CountryList implements ConnectionAwareInterface
    {
      protected $connection;
      public function setConnection(Connection $connection)
      {
        $this->connection = $connection;
      }
      public function list()
      {
        $list = [];
        $stmt = $this->connection->pdo->query(
          'SELECT iso3, name FROM iso_country_codes');
        while ($country = $stmt->fetch(PDO::FETCH_ASSOC)) {
          $list[$country['iso3']] =  $country['name'];
        }
        return $list;
      }

    }
    class CustomerList implements ConnectionAwareInterface
    {
      protected $connection;
      public function setConnection(Connection $connection)
      {
        $this->connection = $connection;
      }
      public function list()
      {
        $list = [];
        $stmt = $this->connection->pdo->query(
          'SELECT id, name FROM customer');
        while ($customer = $stmt->fetch(PDO::FETCH_ASSOC)) {
          $list[$customer['id']] =  $customer['name'];
        }
        return $list;
      }

    }
    ```

1.  接口可用于满足类型提示。以下类`ListFactory`包含一个`factory()`方法，该方法初始化任何实现`ConnectionAwareInterface`的类。接口是`setConnection()`方法被定义的保证。将类型提示设置为接口而不是特定类实例使`factory`方法更通用：

```php
    namespace Application\Generic;

    use PDO;
    use Exception;
    use Application\Database\Connection;
    use Application\Database\ConnectionAwareInterface;

    class ListFactory
    {
      const ERROR_AWARE = 'Class must be Connection Aware';
      public static function factory(
        ConnectionAwareInterface $class, $dbParams)
      {
        if ($class instanceofConnectionAwareInterface) {
            $class->setConnection(new Connection($dbParams));
            return $class;
        } else {
            throw new Exception(self::ERROR_AWARE);
        }
        return FALSE;
      }
    }
    ```

1.  如果一个类实现多个接口，如果方法签名不匹配，则会发生**命名冲突**。在这个例子中，有两个接口，`DateAware`和`TimeAware`。除了定义`setDate()`和`setTime()`方法之外，它们都定义了`setBoth()`。具有重复的方法名称不是问题，尽管这不被认为是最佳实践。问题在于方法签名不同：

```php
    interface DateAware
    {
      public function setDate($date);
      public function setBoth(DateTime $dateTime);
    }

    interface TimeAware
    {
      public function setTime($time);
      public function setBoth($date, $time);
    }

    class DateTimeHandler implements DateAware, TimeAware
    {
      protected $date;
      protected $time;
      public function setDate($date)
      {
        $this->date = $date;
      }
      public function setTime($time)
      {
        $this->time = $time;
      }
      public function setBoth(DateTime $dateTime)
      {
        $this->date = $date;
      }
    }
    ```

1.  代码块的当前状态将生成致命错误（无法捕获！）。要解决问题，首选方法是从一个接口中删除`setBoth()`的定义。或者，您可以调整方法签名以匹配。

### 注意

**最佳实践**

不要定义具有重复或重叠方法定义的接口。

## 它是如何工作的...

在`Application/Database`文件夹中，创建一个文件`ConnectionAwareInterface.php`。插入前面步骤 2 中讨论的代码。

接下来，在`Application/Generic`文件夹中，创建两个文件，`CountryList.php`和`CustomerList.php`。插入步骤 3 中讨论的代码。

接下来，在与`Application`目录平行的目录中，创建一个源代码文件`chap_04_oop_simple_interfaces_example.php`，该文件初始化自动加载程序并包含数据库参数：

```php
<?php
define('DB_CONFIG_FILE', '/../config/db.config.php');
require __DIR__ . '/../Application/Autoload/Loader.php';
Application\Autoload\Loader::init(__DIR__ . '/..');
$params = include __DIR__ . DB_CONFIG_FILE;
```

在这个例子中，假定数据库参数在由`DB_CONFIG_FILE`常量指示的数据库配置文件中。

现在，您可以使用`ListFactory::factory()`生成`CountryList`和`CustomerList`对象。请注意，如果这些类没有实现`ConnectionAwareInterface`，将会抛出错误：

```php
  $list = Application\Generic\ListFactory::factory(
    new Application\Generic\CountryList(), $params);
  foreach ($list->list() as $item) echo $item . '';
```

这是国家列表的输出：

![它是如何工作的...](img/B05314_04_22.jpg)

您还可以使用`factory`方法生成`CustomerList`对象并使用它：

```php
  $list = Application\Generic\ListFactory::factory(
    new Application\Generic\CustomerList(), $params);
  foreach ($list->list() as $item) echo $item . '';
```

这是`CustomerList`的输出：

![它是如何工作的...](img/B05314_04_23.jpg)

如果您想要检查实现多个接口但方法签名不同的情况发生了什么，请将步骤 4 中显示的代码输入到文件`chap_04_oop_interfaces_collisions.php`中。当您尝试运行该文件时，将生成错误，如下所示：

![它是如何工作的...](img/B05314_04_24.jpg)

如果您在`TimeAware`接口中进行以下调整，将不会产生错误：

```php
interface TimeAware
{
  public function setTime($time);
  // this will cause a problem
  public function setBoth(DateTime $dateTime);
}
```

# 使用特征

如果您曾经进行过 C 编程，您可能熟悉宏。宏是预定义的代码块，在指定的行处*展开*。类似地，特征可以包含代码块，在 PHP 解释器指定的行处复制并粘贴到类中。

## 如何做...

1.  特征用关键字`trait`标识，可以包含属性和/或方法。在上一个示例中，当检查`CountryList`和`CustomerList`类时，您可能已经注意到代码的重复。在这个例子中，我们将重构这两个类，并将`list()`方法的功能移入`Trait`。请注意，`list()`方法在两个类中是相同的。

1.  特征在类之间存在代码重复的情况下使用。然而，请注意，使用传统的创建抽象类并扩展它的方法可能比使用特征具有某些优势。特征不能用于标识继承线，而抽象父类可以用于此目的。

1.  现在我们将`list()`复制到一个名为`ListTrait`的特征中：

```php
    trait ListTrait
    {
      public function list()
      {
        $list = [];
        $sql  = sprintf('SELECT %s, %s FROM %s', 
          $this->key, $this->value, $this->table);
        $stmt = $this->connection->pdo->query($sql);
        while ($item = $stmt->fetch(PDO::FETCH_ASSOC)) {
          $list[$item[$this->key]] = $item[$this->value];
        }
        return $list;
      }
    }
    ```

1.  然后我们可以将`ListTrait`中的代码插入到一个新的类`CountryListUsingTrait`中，如下面的代码片段所示。现在可以从这个类中删除整个`list()`方法：

```php
    class CountryListUsingTrait implements ConnectionAwareInterface
    {

      use ListTrait;

      protected $connection;
      protected $key   = 'iso3';
      protected $value = 'name';
      protected $table = 'iso_country_codes';

      public function setConnection(Connection $connection)
      {
        $this->connection = $connection;
      }

    }
    ```

### 注意

每当存在代码重复时，当您需要进行更改时，可能会出现潜在问题。您可能会发现自己需要进行太多的全局搜索和替换操作，或者剪切和粘贴代码，通常会导致灾难性的结果。特征是避免这种维护噩梦的好方法。

1.  特征受命名空间的影响。在第 1 步中所示的示例中，如果我们的新`CountryListUsingTrait`类放置在一个名为`Application\Generic`的命名空间中，我们还需要将`ListTrait`移动到该命名空间中：

```php
    namespace Application\Generic;

    use PDO;

    trait ListTrait
    {
      public function list()
      {
        // code as shown above
      }
    }
    ```

1.  特征中的方法会覆盖继承的方法。

1.  在下面的示例中，您会注意到`setId()`方法的返回值在`Base`父类和`Test`特征之间不同。`Customer`类继承自`Base`，但也使用`Test`。在这种情况下，特征中定义的方法将覆盖`Base`父类中定义的方法：

```php
    trait Test
    {
      public function setId($id)
      {
        $obj = new stdClass();
        $obj->id = $id;
        $this->id = $obj;
      }
    }

    class Base
    {
      protected $id;
      public function getId()
      {
        return $this->id;
      }
      public function setId($id)
      {
        $this->id = $id;
      }
    }

    class Customer extends Base
    {
      use Test;
      protected $name;
      public function getName()
      {
        return $this->name;
      }
      public function setName($name)
      {
        $this->name = $name;
      }
    }
    ```

### 注意

在 PHP 5 中，特征也可以覆盖属性。在 PHP 7 中，如果特征中的属性初始化值与父类中的不同，将生成致命错误。

1.  在类中直接定义使用特征的方法会覆盖特征中定义的重复方法。

1.  在这个例子中，`Test`特征定义了一个`$id`属性以及`getId()`方法和`setId()`。特征还定义了`setName()`，与`Customer`类中定义的相同方法冲突。在这种情况下，`Customer`中直接定义的`setName()`方法将覆盖特征中定义的`setName()`：

```php
    trait Test
    {
      protected $id;
      public function getId()
      {
        return $this->id;
      }
      public function setId($id)
      {
        $this->id = $id;
      }
      public function setName($name)
      {
        $obj = new stdClass();
        $obj->name = $name;
        $this->name = $obj;
      }
    }

    class Customer
    {
      use Test;
      protected $name;
      public function getName()
      {
        return $this->name;
      }
      public function setName($name)
      {
        $this->name = $name;
      }
    }
    ```

1.  在使用多个特征时，使用`insteadof`关键字解决方法名称冲突。此外，使用`as`关键字为方法名称创建别名。

1.  在这个例子中，有两个特征，`IdTrait`和`NameTrait`。两个特征都定义了一个`setKey()`方法，但是以不同的方式表示键。`Test`类使用了这两个特征。请注意`insteadof`关键字，它允许我们区分冲突的方法。因此，当从`Test`类调用`setKey()`时，源将来自`NameTrait`。此外，`IdTrait`中的`setKey()`仍然可用，但是在别名`setKeyDate()`下：

```php
    trait IdTrait
    {
      protected $id;
      public $key;
      public function setId($id)
      {
        $this->id = $id;
      }
      public function setKey()
      {
        $this->key = date('YmdHis') 
        . sprintf('%04d', rand(0,9999));
      }
    }

    trait NameTrait
    {
      protected $name;
      public $key;
      public function setName($name)
      {
        $this->name = $name;
      }
      public function setKey()
      {
        $this->key = unpack('H*', random_bytes(18))[1];
      }
    }

    class Test
    {
      use IdTrait, NameTrait {
        NameTrait::setKeyinsteadofIdTrait;
        IdTrait::setKey as setKeyDate;
      }
    }
    ```

## 它是如何工作的...

从第 1 步中，您了解到特征在存在代码重复的情况下使用。您需要评估是否可以简单地定义一个基类并扩展它，或者使用特征更好地满足您的目的。特征在逻辑上不相关的类中看到代码重复时特别有用。

为了说明特征方法如何覆盖继承的方法，请将第 7 步提到的代码块复制到一个单独的文件`chap_04_oop_traits_override_inherited.php`中。添加以下代码：

```php
$customer = new Customer();
$customer->setId(100);
$customer->setName('Fred');
var_dump($customer);
```

从输出中可以看到（如下所示），`$id`属性存储为`stdClass()`的实例，这是特征中定义的行为：

![它是如何工作的...](img/B05314_04_28.jpg)

为了说明直接定义的类方法如何覆盖特征方法，请将第 9 步提到的代码块复制到一个单独的文件`chap_04_oop_trait_methods_do_not_override_class_methods.php`中。添加以下代码：

```php
$customer = new Customer();
$customer->setId(100);
$customer->setName('Fred');
var_dump($customer);
```

从下面的输出中可以看到，`$id`属性存储为整数，如`Customer`类中定义的那样，而特征将`$id`定义为`stdClass`的实例：

![它是如何工作的...](img/B05314_04_29.jpg)

在第 10 步中，您学会了如何在使用多个特征时解决重复方法名称冲突。将步骤 11 中显示的代码块复制到一个单独的文件`chap_04_oop_trait_multiple.php`中。添加以下代码：

```php
$a = new Test();
$a->setId(100);
$a->setName('Fred');
$a->setKey();
var_dump($a);

$a->setKeyDate();
var_dump($a);
```

请注意，在下面的输出中，`setKey()`产生了从新的 PHP 7 函数`random_bytes()`（在`NameTrait`中定义）产生的输出，而`setKeyDate()`使用`date()`和`rand()`函数（在`IdTrait`中定义）产生一个密钥：

![它是如何工作的...](img/B05314_04_30.jpg)

# 实现匿名类

PHP 7 引入了一个新特性，**匿名类**。就像匿名函数一样，匿名类可以作为表达式的一部分来定义，创建一个没有名称的类。匿名类用于需要*临时*创建并使用然后丢弃对象的情况。

## 如何做...

1.  与`stdClass`的替代方案是定义一个匿名类。

在定义中，您可以定义任何属性和方法（包括魔术方法）。在这个例子中，我们定义了一个具有两个属性和一个魔术方法`__construct()`的匿名类：

```php
    $a = new class (123.45, 'TEST') {
      public $total = 0;
      public $test  = '';
      public function __construct($total, $test)
      {
        $this->total = $total;
        $this->test  = $test;
      }
    };
    ```

1.  匿名类可以扩展任何类。

在这个例子中，一个匿名类扩展了`FilterIterator`，并覆盖了`__construct()`和`accept()`方法。作为参数，它接受了`ArrayIterator` `$b`，它代表了一个 10 到 100 的增量为 10 的数组。第二个参数作为输出的限制：

```php
    $b = new ArrayIterator(range(10,100,10));
    $f = new class ($b, 50) extends FilterIterator {
      public $limit = 0;
      public function __construct($iterator, $limit)
      {
        $this->limit = $limit;
        parent::__construct($iterator);
      }
      public function accept()
      {
        return ($this->current() <= $this->limit);
      }
    };
    ```

1.  匿名类可以实现一个接口。

在这个例子中，一个匿名类用于生成 HTML 颜色代码图表。该类实现了内置的 PHP `Countable`接口。定义了一个`count()`方法，当这个类与需要`Countable`的方法或函数一起使用时调用：

```php
    define('MAX_COLORS', 256 ** 3);

    $d = new class () implements Countable {
      public $current = 0;
      public $maxRows = 16;
      public $maxCols = 64;
      public function cycle()
      {
        $row = '';
        $max = $this->maxRows * $this->maxCols;
        for ($x = 0; $x < $this->maxRows; $x++) {
          $row .= '<tr>';
          for ($y = 0; $y < $this->maxCols; $y++) {
            $row .= sprintf(
              '<td style="background-color: #%06X;"', 
              $this->current);
            $row .= sprintf(
              'title="#%06X">&nbsp;</td>', 
              $this->current);
            $this->current++;
            $this->current = ($this->current >MAX_COLORS) ? 0 
                 : $this->current;
          }
          $row .= '</tr>';
        }
        return $row;
      }
      public function count()
      {
        return MAX_COLORS;
      }
    };
    ```

1.  匿名类可以使用特征。

1.  这个最后的例子是对前面立即定义的修改。我们不是定义一个`Test`类，而是定义一个匿名类：

```php
    $a = new class() {
      use IdTrait, NameTrait {
        NameTrait::setKeyinsteadofIdTrait;
        IdTrait::setKey as setKeyDate;
      }
    };
    ```

## 它是如何工作的...

在匿名类中，您可以定义任何属性或方法。使用前面的例子，您可以定义一个接受构造函数参数的匿名类，并且可以访问属性。将步骤 2 中描述的代码放入一个名为`chap_04_oop_anonymous_class.php`的测试脚本中。添加这些`echo`语句：

```php
echo "\nAnonymous Class\n";
echo $a->total .PHP_EOL;
echo $a->test . PHP_EOL;
```

以下是匿名类的输出：

![它是如何工作的...](img/B05314_04_05.jpg)

为了使用`FilterIterator`，您*必须*覆盖`accept()`方法。在这个方法中，您定义了迭代的元素被包括在输出中的标准。现在继续并将步骤 4 中显示的代码添加到测试脚本中。然后您可以添加这些`echo`语句来测试匿名类：

```php
echo "\nAnonymous Class Extends FilterIterator\n";
foreach ($f as $item) echo $item . '';
echo PHP_EOL;
```

在这个例子中，建立了一个 50 的限制。原始的`ArrayIterator`包含一个值数组，从 10 到 100，增量为 10，如下面的输出所示：

![它是如何工作的...](img/B05314_04_12.jpg)

要查看实现接口的匿名类，请考虑步骤 5 和 6 中显示的例子。将这段代码放入一个文件`chap_04_oop_anonymous_class_interfaces.php`中。

接下来，添加代码，让您可以通过 HTML 颜色图表进行分页：

```php
$d->current = $_GET['current'] ?? 0;
$d->current = hexdec($d->current);
$factor = ($d->maxRows * $d->maxCols);
$next = $d->current + $factor;
$prev = $d->current - $factor;
$next = ($next <MAX_COLORS) ? $next : MAX_COLORS - $factor;
$prev = ($prev>= 0) ? $prev : 0;
$next = sprintf('%06X', $next);
$prev = sprintf('%06X', $prev);
?>
```

最后，继续并将 HTML 颜色图表呈现为一个网页：

```php
<h1>Total Possible Color Combinations: <?= count($d); ?></h1>
<hr>
<table>
<?= $d->cycle(); ?>
</table>	
<a href="?current=<?= $prev ?>"><<PREV</a>
<a href="?current=<?= $next ?>">NEXT >></a>
```

请注意，您可以通过将匿名类的实例传递给`count()`函数（在`<H1>`标签之间显示）来利用`Countable`接口。以下是在浏览器窗口中显示的输出：

![它是如何工作的...](img/B05314_04_25.jpg)

最后，为了说明匿名类中使用特征，将前面一篇文章中提到的`chap_04_oop_trait_multiple.php`文件复制到一个新文件`chap_04_oop_trait_anonymous_class.php`中。删除`Test`类的定义，并用匿名类替换它：

```php
$a = new class() {
  use IdTrait, NameTrait {
    NameTrait::setKeyinsteadofIdTrait;
    IdTrait::setKey as setKeyDate;
  }
};
```

删除这一行：

```php
$a = new Test();
```

当您运行代码时，您将看到与前面截图中完全相同的输出，只是类引用将是匿名的：

![它是如何工作的...](img/B05314_04_31.jpg)
