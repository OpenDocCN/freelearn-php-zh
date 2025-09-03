# 第三章。更多面向对象编程

前一章为我们创建了一个基础，以便我们可以用 PHP 启动面向对象编程。这一章将更详细地处理一些高级功能。例如，我们将学习关于类信息函数，通过这些函数我们可以调查任何类的详细信息。然后我们将学习一些实用的面向对象信息函数，以及 PHP5 中的一个伟大新特性，即异常处理。

本章还将介绍迭代器以简化数组访问。为了存储任何对象以供以后使用，我们需要使用面向对象编程中的特殊功能，即序列化，我们也将在这里学习这一点。总的来说，本章将加强你在面向对象编程方面的基础。

# 类信息函数

如果你想要调查和收集有关任何类的更多信息，这些函数将是你的光明。这些函数可以检索有关类的几乎所有信息。但是，这些函数有一个改进版本，并在 PHP5 中作为全新的 API 集引入。这个 API 被称为**反射**。我们将在第五章学习关于反射 API 的内容。

## 检查类是否已存在

当你需要检查当前作用域中是否存在任何类时，你可以使用一个名为`class_exists()`的函数。看看以下例子：

```php
<?
include_once("../ch2/class.emailer.php");
echo class_exists("Emailer");
//returns true otherwise false if doesn't exist
?>
```

使用`class_exists()`函数的最佳方式是首先检查类是否已经存在。如果类存在，你可以创建该类的实例，这将使你的代码更加稳定。

```php
<?
include_once("../ch2/class.emailer.php");
if( class_exists("Emailer"))
{
  $emailer = new Emailer("hasin@pageflakes.com");
}
else 
{
  die("A necessary class is not found");
}
?>
```

## 查找当前加载的类

在某些情况下，你可能需要调查当前作用域中加载了哪些类。你可以使用`get_declared_classes()`函数来完成这项工作。这个函数将返回一个包含当前可用类的数组。

```php
<?
include_once("../ch2/class.emailer.php");
print_r(get_declared_classes());
?>
```

你将在屏幕上看到当前可用的类列表。

## 查找方法和属性是否存在

要找出类内部是否有属性和/或方法可用，你可以使用`method_exists()`和`property_exists()`函数。请注意，这些函数只有在属性和方法在公共作用域中定义时才会返回 true。

## 检查类的类型

有一个名为`is_a()`的函数，你可以用它来检查类的类型。看看以下例子：

```php
<?
class ParentClass
{
}

class ChildClass extends ParentClass 
{
}

$cc = new ChildClass();
if  (is_a($cc,"ChildClass")) echo "It's a ChildClass Type Object";
echo "\n";
if  (is_a($cc,"ParentClass")) echo "It's also a ParentClass Type 
Object";

?>
```

你将看到以下输出：

```php
Its a ChildClass Type Object
Its also a ParentClass Type Object
```

## 查找类名

在前面的例子中，我们检查了类是否是已知类型。如果我们需要获取类的原始名称呢？不用担心，我们有`get_class()`函数来帮助我们。

```php
<?
class ParentClass
{
}
class ChildClass extends ParentClass 
{	
}

$cc = new ChildClass();
echo get_class($cc)
?>
```

作为输出，你应该得到`ChildClass`。现在看看以下例子，这是“brjann”在 PHP 手册用户备注部分中提到的意外行为。

```php
<?
class ParentClass 
{
  public function getClass()
{
    echo get_class(); //using "no $this"
  }
}
class Child extends ParentClass 
{
}
$obj = new Child();
$obj->getClass(); //outputs "ParentClass"
?>
```

如果你运行这段代码，你会看到输出为 `ParentClass`。但是为什么？你是在调用 `Child` 类的方法。这是意外的吗？好吧，不是的。认真看看代码。虽然 `Child` 扩展了 `ParentClass` 对象，但它没有重写 `getClass()` 方法。所以方法仍然在 `ParentClass` 的作用域下运行。这就是为什么它返回 `ParentClass` 的结果。

所以实际上发生了什么？为什么它返回 `Child`？

```php
<?
class ParentClass {
  public function getClass(){
    echo get_class($this); //using "$this"
  }
}
class Child extends ParentClass {
}
$obj = new Child();
$obj->getClass(); //outputs "child"
?>
```

在 `ParentClass` 对象中，`get_class()` 函数返回 `$this` 对象，它明显持有 `Child` 类的引用。这就是为什么你得到 `Child` 作为输出。

# 异常处理

PHP5 中最改进的特性之一是现在你可以使用异常，就像其他现有的面向对象编程语言一样。PHP5 引入了这些异常对象来简化你的错误管理。

让我们看看这些异常是如何发生的以及如何处理它们。看看以下类，它简单地连接到 PostgreSQL 服务器。在无法连接到服务器的情况下，让我们看看它通常会返回什么：

```php
<?
//class.db.php
class db
{
  function connect()
  {
    pg_connect("somehost","username","password");
  }
}

$db = new db();
$db->connect();
?>
```

输出如下。

```php
<b>Warning</b>: pg_connect() [<a href='function.pg-connect'>
function.pg-connect</a>]: Unable to connect to PostgreSQL 
server: could not translate host name "somehost" to address: 
Unknown host in <b>C:\OOP with PHP5\Codes\ch3\exception1.php</b> 
on line <b>6</b><br />
```

你如何在 PHP4 中处理它？通常，可以通过以下类似的方式：

```php
<?
//class.db.php
error_reporting(E_ALL - E_WARNING);
class db
{
  function connect()
  {
    if (!pg_connect("somehost","username","password")) return false;
  }
}

$db = new db();

if (!$db->connect()) echo "Falied to connect to PostgreSQL Server";
?> 
```

现在让我们看看如何使用异常来解决它。

```php
<?
//class.db.php
error_reporting(E_ALL - E_WARNING);
class db
{
  function connect()
  {
    if (!pg_connect("host=localhost password=pass user=username 
                    dbname=db")) throw new Exception("Cannot connect 
                    to the database");
  }
}

$db = new db();
try {
  $db->connect();
}
catch (Exception $e)
{
  print_r($e);
}

?>
```

输出将类似于以下内容：

```php
Exception Object
(
  [message:protected] => Cannot connect to the database
  [string:private] => 
  [code:protected] => 0
  [file:protected] => C:\OOP with PHP5\Codes\ch3\exception1.php
  [line:protected] => 8
  [trace:private] => Array
    (
      [0] => Array
        (
          [file] => C:\OOP with PHP5\Codes\ch3\exception1.php
          [line] => 14
          [function] => connect
          [class] => db
          [type] => ->
          [args] => Array
             (
              )

        )

      [1] => Array
        (
          [file] => C:\Program Files\Zend\ZendStudio-
                                       5.2.0\bin\php5\dummy.php
          [line] => 1
          [args] => Array
            (
              [0] => C:\OOP with PHP5\Codes\ch3\exception1.php
            )

          [function] => include
        )

    )

)
```

所以你在这个异常类中得到了很多内容。你可以使用这个 try-catch 块来捕获所有错误。你可以在另一个 try-catch 块中使用 try-catch。看看以下示例。这里我们开发了两个自己的异常对象，使错误处理更加结构化。

```php
<?
include_once("PGSQLConnectionException.class.php");
include_once("PGSQLQueryException.class.php");
error_reporting(0);
class DAL
{
  public $connection;
  public $result;
  public function connect($ConnectionString)
  {
    $this->connection = pg_connect($ConnectionString);

    if ($this->connection==false)
    {
      throw new PGSQLConnectionException($this->connection);
    }
  }

  public function execute($query)
  {
    $this->result = pg_query($this->connection,$query);

    if (!is_resource($this->result))
    {
      throw new PGSQLQueryException($this->connection);
    }

    //else do the necessary works
  }
}

$db = new DAL();
try{
  $db->connect("dbname=golpo user=postgres2");
  try{
    $db->execute("select * from abc");
  }
  catch (Exception $queryexception)
  {
    echo $queryexception->getMessage();
  }
}
catch(Exception $connectionexception)
{
  echo $connectionexception->getMessage();
}
?>
```

现在，如果代码无法连接到数据库，它会捕获错误并显示“**抱歉，无法连接到 PostgreSQL 服务器**”的消息。如果连接成功但问题出在查询上，它将显示适当的信息。如果你检查代码，你会发现对于连接失败，我们抛出一个 `PGSQLConnectionException` 对象，而对于查询失败，我们只抛出一个 `PGSQLQueryException` 对象。我们可以通过扩展 PHP5 的核心 Exception 类来自定义开发这些对象。让我们看看代码。第一个是 `PGSQLConnectionException` 类。

```php
<?
Class PGSQLConnectionException extends Exception
{

  public function __construct()
  {  $message = "Sorry, couldn't connect to postgresql server:";
     parent::__construct($message, 0000);
  }
}
?>
```

现在出现了 `PGSQLQueryException` 类

```php
<?
Class PGSQLQueryException extends Exception
{
  public function __construct($connection)
  {
    parent::__construct(pg_last_error($connection),0);
  }
}
?>
```

就这样！

## 收集所有 PHP 错误作为异常

如果你想要收集所有 PHP 错误（除了 FATAL 错误）作为异常，你可以使用以下代码：

```php
<?php
function exceptions_error_handler($severity, $message, 
       $filename, $lineno) { 
          throw new ErrorException($message, 0, $severity, 
          $filename, $lineno); 
       }
set_error_handler('exceptions_error_handler');
?>
```

上述代码片段的功劳归功于 `<fjoggen@gmail.com>`，这是我从 PHP 手册用户笔记中收集的。

# 迭代器

迭代器是 PHP5 中引入的一个新命令，用于帮助遍历任何对象。查看以下示例，了解迭代器实际上用于什么。在 PHP4 中，你可以像以下示例那样使用 `foreach` 语句遍历数组：

```php
<?
foreach($anyarray as $key=>$val)
{
  //do something
}
?>
```

你也可以对对象执行 `foreach` 操作，让我们看看以下示例。

```php
<?
class EmailValidator
{
  public $emails;
  public $validemails;
}

$ev = new EmailValidator();
foreach($ev as $key=>$val)
{
  echo $key."<br/>";
}
?>
```

这段代码将输出以下内容：

```php
emails
validemails
```

请注意，它只能遍历公共属性。但是，如果我们只想得到有效的电子邮件地址作为输出呢？在 PHP5 中，通过实现`Iterator`和`IteratorAggregator`接口是可能的。让我们通过以下示例来看看。在这个例子中，我们创建了一个`QueryIterator`，它可以遍历有效的 PostgreSQL 查询结果，并在每次迭代中返回一行。

```php
<?
class QueryIterator implements Iterator
{
  private $result;
  private $connection;
  private $data;
  private $key=0;
  private $valid;

  function __construct($dbname, $user, $password)
  {
    $this->connection = pg_connect("dbname={$dbname} user={$user}");
  }

  public function exceute($query)
  {
    $this->result = pg_query($this->connection,$query);
     if (pg_num_rows($this->result)>0)
    $this->next();
  }

  public function rewind() {}

  public function current() {
    return $this->data;
  }

  public function key() {
    return $this->key;
  }

  public function next() {
    if ($this->data = pg_fetch_assoc($this->result))
    {
      $this->valid = true;
      $this->key+=1;
    }
    else 
    $this->valid = false;
  }

  public function valid() {
    return $this->valid;
  }
}
?>
```

让我们看看代码的实际应用。

```php
<?
$qi= new QueryIterator("golpo","postgres2","");
$qi->exceute("select name, email from users");
while ($qi->valid())
{
  print_r($qi->current());
  $qi->next();
}
?>
```

例如，如果我们的`users`表中有两个记录，您将得到以下输出：

```php
Array
(
  [name] => Afif
  [email] => mayflower@phpxperts.net
)
Array
(
  [name] => Ayesha
  [email] => florence@phpxperts.net
)
```

非常方便，不是吗？

# ArrayObject

在 PHP5 中引入的另一个有用的对象是`ArrayObject`，它包装了常规的 PHP 数组并给它添加了面向对象的特性。您可以通过简单地将数组传递给`ArrayObject`构造函数来创建一个`ArrayObject`对象。`ArrayObject`有以下有用的方法：

**append()**

此方法可以在集合的末尾添加任何值。

**getIterator()**

此方法简单地创建一个`Iterator`对象并返回，这样您就可以使用迭代器风格进行迭代。这是一个非常有用的方法，可以从任何数组中获取`Iterator`对象。

**offsetExists()**

此方法可以确定指定的偏移量是否存在于集合中。

**offsetGet()**

此方法返回指定偏移量的值。

**offsetSet()**

与`offsetGet()`类似，此方法可以将任何值设置到指定的`index()`。

**offsetUnset()**

此方法可以取消指定索引处的元素。

让我们看看`ArrayObject`的一些示例：

```php
<?
$users = new ArrayObject(array("hasin"=>"hasin@pageflakes.com",
   "afif"=>"mayflower@phpxperts.net",
   "ayesha"=>"florence@pageflakes.net"));
$iterator = $users->getIterator();
while ($iterator->valid())
{
  echo "{$iterator->key()}'s Email address is 
         {$iterator->current()}\n";
         $iterator->next();
}
?> 
```

# 数组到对象

我们可以通过其键来访问任何数组元素，例如`$array[$key]`。然而，如果我们想以`$array->key`的方式访问它呢？这非常简单，我们可以通过扩展`ArrayObject`来实现。让我们通过以下示例来看看。

```php
<?
class ArrayToObject extends ArrayObject
{
  public function __get($key)
  {
    return $this[$key];
  }

  public function __set($key,$val)
  {
    $this[$key] = $val;
  }
}
?>
```

现在我们来看看它的实际应用：

```php
<?
$users = new ArrayToObject(array("hasin"=>"hasin@pageflakes.com",
   "afif"=>"mayflower@phpxperts.net",
   "ayesha"=>"florence@pageflakes.net"));

echo $users->afif;
?>
```

它将输出与键`afif`关联的电子邮件地址，如下所示：

```php
mayflower@phpxperts.net
```

如果您想将任何已知格式的数组转换为对象，这个示例可能会很有用。

# 以数组风格访问对象

在前面的章节中，我们学习了如何以面向对象的方式访问任何数组。如果我们想以数组风格访问任何对象怎么办？PHP 提供了这样的功能。您只需要在您的类中实现`ArrayAccess`接口。

`ArrayAccess`接口有四个方法，您必须在类中实现这些方法。这些方法是`offsetExists()`、`offsetGet()`、`offsetSet()`、`offsetUnset()`。让我们创建一个实现`ArrayAccess`接口的示例类。

```php
<?php

class users implements ArrayAccess 
{
  private $users;

    public function __construct() 
{
        $this->users = array();
    }

    public function offsetExists($key) 
{
        return isset($this->users[$key]);
    }

    public function offsetGet($key) 
{
        return $this->users[$key];
    }

    public function offsetSet($key, $value) 
{
        $this->users[$key] = $value;
    }

    public function offsetUnset($key) 
{
        unset($this->users[$key]);
    }
}

$users = new users();
$users['afif']="mayflower@phpxperts.net";
$users['hasin']="hasin@pageflakes.com";
$users['ayesha']="florence@phpxperts.net";

echo $users['afif']
?>
```

输出将是`mayflower@phpxperts.net`。

# 序列化

到目前为止，我们已经学习了如何创建对象并操作它们。现在，如果您需要保存对象的状态并在稍后以完全相同的形式检索它，会发生什么？在 PHP 中，您可以通过序列化来实现此功能。

序列化是将对象的状态持久化在任何位置的过程，无论是物理文件还是变量中。为了检索该对象的状态，另一个过程被使用，这被称为“反序列化”。您可以使用 `serialize()` 函数序列化任何对象。让我们看看我们如何序列化一个对象：

```php
<?
class SampleObject
{
  public $var1;
  private $var2;
  protected $var3;
  static $var4;

  public function __construct()
  {
    $this->var1 = "Value One";
    $this->var2 = "Value Two";
    $this->var3 = "Value Three";
    SampleObject::$var4 = "Value Four";
  }

}

$so = new SampleObject();
$serializedso =serialize($so);
file_put_contents("text.txt",$serializedso);
echo $serializedso;
?>
```

脚本将输出一个字符串，PHP 知道如何反序列化它。

现在是时候检索我们的序列化对象并将其转换为可用的 PHP 对象了。请记住，您正在反序列化的类文件必须首先加载。

```php
<?
include_once("class.sampleobject.php");
$serializedcontent = file_get_contents("text.txt");
$unserializedcontent = unserialize($serializedcontent);
print_r($unserializedcontent);
?>
```

你认为输出会是什么？看看：

```php
SampleObject Object
(
  [var1] => Value One
  [var2:private] => Value Two
  [var3:protected] => Value Three
)
```

现在它是一个常规的 PHP 对象；与序列化之前完全相同。请注意，所有变量都保留了它们在序列化之前设置的值，除了静态变量。您不能通过序列化来保存静态变量的状态。

如果我们在反序列化之前没有使用 `include_once` 包含类文件，会怎样？让我们只注释掉第一行，包含类文件的行，然后运行示例代码。你会得到以下输出：

```php
__PHP_Incomplete_Class Object
(
  [__PHP_Incomplete_Class_Name] => SampleObject
  [var1] => Value One
  [var2:private] => Value Two
  [var3:protected] => Value Three
)
```

到这一点，您不能再将其用作对象了。

## 序列化中的魔法方法

你还记得我们使用一些魔法方法如 `__get`、`__set` 和 `__call` 重载属性和方法吗？对于序列化，您可以使用一些魔法方法来挂钩序列化过程。PHP5 提供了两个名为 `__sleep` 和 `__awake` 的魔法方法来达到这个目的。这些方法在整个过程中提供了一些控制。

让我们使用这些魔法方法来开发一个进程的所有静态变量，通常我们无法不通过黑客手段做到这一点。通常情况下，无法序列化任何静态变量的值，并返回具有该静态变量的对象处于相同状态。然而，我们可以让它发生，让我们看看下面的代码。

```php
<?
class SampleObject
{
  public $var1;
  private $var2;
  protected $var3;
  public static $var4;

  private $staticvars = array();

  public function __construct()
  {
    $this->var1 = "Value One";
    $this->var2 = "Value Two";
    $this->var3 = "Value Three";
    SampleObject::$var4 = "Value Four";
  }

  public function __sleep()
  {

    $vars = get_class_vars(get_class($this));
    foreach($vars as $key=>$val)
    {
      if (!empty($val))
      $this->staticvars[$key]=$val;
    }
    return array_keys( get_object_vars( $this ) );
  }

  public function __wakeup()
  {
    foreach ($this->staticvars as $key=>$val){
      $prop = new ReflectionProperty(get_class($this), $key);
      $prop->setValue(get_class($this), $val);
    }
    $this->staticvars=array();
  }

}
?>
```

如果我们序列化对象，将其写入文件，然后稍后检索状态会发生什么？你会发现静态值仍然保持最后分配给它的值。

让我们讨论一下代码。`__sleep` 函数执行所有必要的操作。它搜索具有值的公共属性，并在找到时将变量的名称存储到一个私有变量 `staticvars` 中。当有人尝试反序列化对象时，它会从 `staticvars` 中检索每个值并将其写入属性本身。非常方便，不是吗？

您会注意到，除了 `__sleep()` 和 `__wakeup()` 函数的理论能力之外，我们没有使用任何技巧。那么这两个函数有什么用？我们可以在实践中在哪里使用它们？这实际上相当简单。例如，如果您的类与其相关联任何资源对象（一个活动的数据库连接，一个打开文件的引用）在 `sleep` 函数中，您可以适当地关闭它们，因为当有人反序列化时它们将不再可用。请记住，在反序列化状态下，有人仍然可能使用那些资源指针。所以，在 `__wakeup()` 函数中，您可以打开那些数据库连接或文件指针，使其恢复到之前的确切形状。让我们通过以下示例来看看：

```php
<?
class ResourceObject
{
  private $resource;
  private $dsn;
  public function __construct($dsn)
  {
    $this->dsn = $dsn;
    $this->resource = pg_connect($this->dsn);
  }

  public function __sleep()
  {
    pg_close($this->resource);
    return array_keys( get_object_vars( $this ) );
  }

  public function __wakeup()
  {
    $this->resource = pg_connect($this->dsn);
  }
}
?>
```

当这个对象被序列化时，它将释放 `$resource` 消耗的内存。稍后，当它将被反序列化时，它将使用 DSN 字符串再次打开连接。所以现在，在反序列化之后，一切如故。这就是关键！

# 对象克隆

PHP5 在从一个对象复制到另一个对象时引入了一种新的方法，这与 PHP4 完全不同。在 PHP4 中，当您将一个对象复制到另一个对象时，它执行深度复制。这意味着它只是创建了一个全新的对象，该对象保留了被复制对象的属性。然而，对新的对象所做的任何更改都不会影响主对象。

PHP5 在复制对象时创建浅复制的方式与这个不同。为了清楚地理解这种情况，您需要理解以下代码。

```php
<?
$sample1 = new StdClass();
$sample1->name = "Hasin";
$sample2 = $sample1;
$sample2->name = "Afif";
echo $sample1->name;
?> 
```

如果您在 PHP5 中运行上述代码，您能猜到您会得到什么结果？`Hasin` 还是 `Afif`？令人惊讶的是，输出是 `Afif`。正如我之前提到的，PHP5 在复制对象时执行浅复制；`$sample2` 只是 `$sample1` 的引用。所以，无论何时您对 `$sample1` 对象或 `$sample2` 对象进行任何更改，它都会影响两者。

在 PHP4 中，它的工作方式不同；它将输出 `Hasin`，因为它们彼此不同。

如果您想在 PHP5 中执行相同的操作，您必须使用 `clone` 关键字。让我们看看以下示例

```php
<?
$sample1 = new stdClass();
$sample1->name = "Hasin";
$sample2 =clone $sample1;
$sample2->name = "Afif";
echo $sample1->name;
?>
```

现在的输出将是 `Hasin`。

# 按需自动加载类或类

在处理大型项目时，另一个非常好的实践是在需要时才加载类。这意味着您不应该通过不断加载不必要的类来过度消耗内存。

在我们的示例中，您已经看到我们在将它们在我们的脚本中可用之前，包括原始类文件。除非您包括类文件，否则您无法创建其实例。PHP5 引入了一个自动加载类文件的功能，这样您就不必麻烦地手动包含它们。通常，这个特性在大型应用程序中非常有用，在这些应用程序中，您必须处理大量的类，并且不想每次都调用 `include`。

```php
<?
function __autoload($class)
{
  include_once("class.{$class}.php");
}

$s = new Emailer("hasin@somewherein.net");
?>
```

当你执行上面的脚本时，请注意我们没有包含`Emailer`类的任何类文件。正因为有这个`__autoload()`函数，PHP5 会自动加载当前目录下名为`class.emailer.php`的文件。所以你不需要担心自己包含类。

# 方法链式调用

方法链式调用是 PHP5 中引入的另一种过程，通过它可以直接访问由任何函数返回的对象的方法和属性。它有点像以下这样：

```php
$SomeObject->getObjectOne()->getObjectTwo()->callMethodOfObjectTwo();
```

上述代码表示`$someObject`类有一个名为`getObjectOne()`的方法，它返回一个名为`$objectOne`的对象。这个`$objectOne`还有一个名为`getObjectTwo()`的方法，它返回一个对象，其方法是通过最后的调用调用的。

那么，谁会使用这样的东西呢？让我们看看下面的代码；它完美地展示了方法链在现实生活中的使用：

```php
$dbManager->select("id","email")->from("user")->where("id=1")
                                        ->limit(1)->result();
```

你觉得上面的代码有意义且易于阅读吗？该代码从`user`表中返回一行，包含 ID 和电子邮件，其中 ID 的值为 1。你有没有想过如何设计这样的 DB 管理器对象？让我们看看下面的这个优秀的例子：

```php
<?
class DBManager
{
  private $selectables = array();
  private $table;
  private $whereClause;
  private $limit;

  public function select()
  {
    $this->selectables=func_get_args();
    return $this;
  }

  public function from($table)
  {
    $this->table = $table;
    return $this;
  }

  public function where($clause)
  {
    $this->whereClause = $clause;
    return $this;
  }

  public function limit($limit)
  {
    $this->limit = $limit;
    return $this;
  }

  public function result()
  {
    $query = "SELECT ".join(",",$this->selectables)." FROM 
                                           {$this->table}";
    if (!empty($this->whereClause))
    $query .= " WHERE {$this->whereClause}";

    if (!empty($this->limit))
    $query .= " LIMIT {$this->limit}";	

    echo "The generated Query is : \n".$query;
  }

}
$db= new DBManager();
$db->select("id","name")->from("users")->where("id=1")->
                                       limit(1)->result();
?>
```

输出如下：

```php
The generated Query is : 
SELECT id,name FROM users WHERE id=1 LIMIT 1
```

该类会自动构建查询。那么它是如何工作的呢？嗯，在 PHP5 中，你可以返回对象；所以利用这个特性，我们在每个我们希望成为链式的一部分的方法上返回对象。现在，只需几分钟就可以执行那个查询并返回结果。令人惊讶的是，你还可以执行以下代码，它会产生相同的结果：

```php
$db->from("users")->select("id","name")->limit(1)->where("id=1")
                                                     ->result();
```

这就是 PHP5 的美丽之处；它非常强大。

# PHP 中对象的生命周期和对象缓存

如果你感兴趣于了解对象的生命周期，那么对象在脚本结束时是活跃的。一旦脚本执行完毕，由该脚本实例化的任何对象也会死亡。与 Java 中的 Web 层不同，PHP 中没有全局或应用级别的范围。因此，你不能正常地持久化对象。如果你想持久化一个对象，你可以将其序列化，并在必要时反序列化它。手动处理这个序列化和反序列化过程有时可能显得无聊。如果能在某处存储对象并在以后检索它（嗯，就像序列化和反序列化过程一样，但更灵活）那就真的很好了。

PHP 中确实有一些对象缓存技术可用，效率非常高。其中最成功的是**memcached**。PHP 有一个扩展名为 memcached API，可以从 PECL 下载。Memcached 作为一个独立的服务器运行，并将对象直接缓存到内存中。Memcached 服务器监听一个端口。PHP memcached API 理解如何与 memcached 服务器通信，因此借助它保存和检索对象。在本节中，我们将演示如何使用 memcached，但不会过多地深入细节。

你可以从[`danga.com/memcached`](http://danga.com/memcached)下载 memcached 服务器。如果你使用 Linux，你必须自己编译它。在一些发行版中，你会找到 memcached 包。你可以在[`jehiah.cz/projects/memcached-win32/`](http://jehiah.cz/projects/memcached-win32/)找到 memcached 1.2.1 服务器的`win32`二进制版本，该版本由 kronuz 开发（`<kronuz@users.sourceforge.net>`）。获取可执行文件后，在控制台输入以下命令。这将启动 memcached 服务器。

```php
memcached –d install
```

这将把 memcached 安装为服务。

```php
memcached –d start 
```

这将启动守护进程/服务。

现在是时候将一些对象存储到 memcached 服务器中并检索它们了。

```php
<?
$memcache = new Memcache;

$memcache->connect('localhost', 11211) or die ("Could not connect");

$tmp_object = new stdClass;
$tmp_object->str_attr = 'test';
$tmp_object->int_attr = 12364;

$memcache->set('obj', $tmp_object, false, 60*5) or die ("Failed to save data at the server");
?>
```

当你执行上面的代码时，memcache 服务器会将对象`$tmp_object`与键`obj`关联存储五分钟。五分钟后，这个对象将不再存在。到那时，如果你需要恢复该对象，你可以执行以下代码：

```php
<?
$memcache = new Memcache;
$memcache->connect('localhost', 11211) or die ("Could not connect");

$newobj = $memcache->get('obj');
?>
```

就这样。Memcache 非常流行，它有 Perl、Python、Ruby、Java 和 Dot Net 以及 C 语言的端口。

# 摘要

在本章中，我们学习了如何在 PHP 中使用一些高级面向对象编程（OOP）概念。我们学习了如何从任何对象中检索信息，并了解了 ArrayAccess、ArrayObject、迭代器和一些其他简化开发者生活的原生对象。本章我们从中学到的另一件非常重要的事情是异常处理。

在下一章中，我们将学习设计模式以及如何在 PHP 中使用它们。在此之前，祝大家探索愉快…
