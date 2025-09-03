# 第五章：反射与单元测试

PHP5 相比 PHP4 带来了许多新的特性。它用更智能的新 API 替换了许多旧的 API。其中之一就是反射 API。使用这个酷炫的 API 集合，你可以逆向工程任何类或对象，以了解其属性和方法。你可以动态地调用这些方法并做更多的事情。在本章中，我们将更详细地学习反射以及这些函数的用法。

软件开发的另一个非常重要的部分是为你的作品构建测试套件以进行自动化测试。这是为了确保它能够正确工作，并且在任何更改之后保持向后兼容性。为了简化 PHP 开发者的过程，市场上有很多测试工具。其中一些非常流行的工具有 PHPUnit。在本章中，我们将学习使用 PHP 进行单元测试。

# Reflection

反射 API 提供了一些功能，可以在运行时找出对象或类中的内容。除此之外，反射 API 允许你动态地调用任何对象的任何方法或属性。让我们来实际操作一下反射。反射 API 中引入了众多对象。其中，以下对象非常重要：

```php
class Reflection { }
interface Reflector { }
class ReflectionException extends Exception { }
class ReflectionFunction implements Reflector { }
class ReflectionParameter implements Reflector { }
class ReflectionMethod extends ReflectionFunction { }
class ReflectionClass implements Reflector { }
class ReflectionObject extends ReflectionClass { }
class ReflectionProperty implements Reflector { }
class ReflectionExtension implements Reflector { }

```

让我们先去玩一玩 `ReflectionClass` 吧。

## ReflectionClass

这是反射 API 中的一个主要核心类。这个类帮助你以广义的方式逆向工程任何对象。这个类的结构如下所示：

```php
<?php
class ReflectionClass implements Reflector
{
   final private __clone()
   public object __construct(string name)
   public string __toString()
   public static string export(mixed class, bool return)
   public string getName()
   public bool isInternal()
   public bool isUserDefined()
   public bool isInstantiable()
   public bool hasConstant(string name)
   public bool hasMethod(string name)
   public bool hasProperty(string name)
   public string getFileName()
   public int getStartLine()
   public int getEndLine()
   public string getDocComment()
   public ReflectionMethod getConstructor()
   public ReflectionMethod getMethod(string name)
   public ReflectionMethod[] getMethods()
   public ReflectionProperty getProperty(string name)
   public ReflectionProperty[] getProperties()
   public array getConstants()
   public mixed getConstant(string name)
   public ReflectionClass[] getInterfaces()
   public bool isInterface()
   public bool isAbstract()
   public bool isFinal()
   public int getModifiers()
   public bool isInstance(stdclass object)
   public stdclass newInstance(mixed args)
   public stdclass newInstanceArgs(array args)
   public ReflectionClass getParentClass()
   public bool isSubclassOf(ReflectionClass class)
   public array getStaticProperties()
   public mixed getStaticPropertyValue(string name [, mixed default])
   public void setStaticPropertyValue(string name, mixed value)
   public array getDefaultProperties()
   public bool isIterateable()
   public bool implementsInterface(string name)
   public ReflectionExtension getExtension()
   public string getExtensionName()
}
?>

```

让我们讨论一下这个类是如何实际工作的。首先，我们将找到它们的方法和目的：

+   `export()` 方法将任何对象的内部结构输出，这与 `var_dump` 函数类似。

+   `getName()` 函数返回对象的内部名称，即类名。

+   `isInternal()` 方法如果类是 PHP5 内置对象则返回 true。

+   `isUserDefined()` 方法与 `isInternal()` 方法的相反。它仅仅返回对象是否是由用户定义的。

+   `getFileName()` 函数返回包含类的 PHP 脚本文件名。

+   `getStartLine()` 返回该类代码在脚本文件中的起始行。

+   `getDocComment()` 是另一个有趣的函数，它返回该对象的类级别文档。我们将在本章后面的示例中演示它。

+   `getConstructor()` 返回对象的构造函数的引用，作为一个 `ReflectionMethod` 对象。

+   `getMethod()` 函数返回传递给它的任何方法的地址。返回的对象是一个 `ReflectionMethod` 对象。

+   `getMethods()` 返回对象中所有方法的数组。在该数组中，每个方法都返回为一个 `ReflectionMethod` 对象。

+   `getProperty()` 函数返回该对象中任何属性的引用，作为一个 `ReflectionProperty` 对象。

+   `getConstants()` 返回该对象中常量的数组。

+   `getConstant()` 返回任何特定常量的值。

+   如果您想查看一个类实现（如果有）的接口引用，您可以使用`getInterfaces()`函数，该函数返回一个包含`ReflectionClass`对象的接口数组。

+   `getModifiers()`方法返回与该类相关的修饰符列表。例如，它可能是公共的、私有的、受保护的、抽象的、静态的或最终的。

+   `newInstance()` `函数返回该类的新实例，并以常规对象的形式返回它（实际上是`stdClas;` `stdClass`是每个 PHP 对象的基础类）。

+   您想获取任何类的父类引用？您可以使用`getParentClass()`方法来获取它，作为`ReflectionClass`对象。

+   `ReflectionClass()`的另一个酷炫功能是它可以告诉你一个类是从哪个扩展中起源的。例如，`ArrayObject`类是从 SPL 类起源的。您必须使用`getExtensionName()`函数来实现这一点。

现在我们来写一些代码。我们将看到这些函数在实际代码中的应用。在这里，我展示了一个来自 PHP 手册的精彩示例。

```php
<?php
interface NSerializable
{
   // ...
}

class Object
{
   // ...
}

/**
* A counter class
*/
class Counter extends Object implements NSerializable 
{
   const START = 0;
   private static $c = Counter::START;

   /**
    * Invoke counter
    *
    * @access  public
    * @return  int
    */
   public function count() 
   {
       return self::$c++;
   }
}

// Create an instance of the ReflectionClass class
$class = new ReflectionClass('Counter');

// Print out basic information
printf(
   "===> The %s%s%s %s '%s' [extends %s]\n" .
   "     declared in %s\n" .
   "     lines %d to %d\n" .
   "     having the modifiers %d [%s]\n",
       $class->isInternal() ? 'internal' : 'user-defined',
       $class->isAbstract() ? ' abstract' : '',
       $class->isFinal() ? ' final' : '',
       $class->isInterface() ? 'interface' : 'class',
       $class->getName(),
       var_export($class->getParentClass(), 1),
       $class->getFileName(),
       $class->getStartLine(),
       $class->getEndline(),
       $class->getModifiers(),
       implode(' ', Reflection::getModifierNames(
                                   $class->getModifiers()))
       );

// Print documentation comment
printf("---> Documentation:\n %s\n", 
                    var_export($class->getDocComment(), 1));

// Print which interfaces are implemented by this class
printf("---> Implements:\n %s\n", 
                    var_export($class->getInterfaces(), 1));

// Print class constants
printf("---> Constants: %s\n", 
                     var_export($class->getConstants(), 1));

// Print class properties
printf("---> Properties: %s\n", 
                     var_export($class->getProperties(), 1));

// Print class methods
printf("---> Methods: %s\n", 
                     var_export($class->getMethods(), 1));

// If this class is instantiable, create an instance
if ($class->isInstantiable()) 
{
   $counter = $class->newInstance();
   echo '---> $counter is instance? '; 
   echo $class->isInstance($counter) ? 'yes' : 'no';
   echo "\n---> new Object() is instance? ";
   echo $class->isInstance(new Object()) ? 'yes' : 'no';
}
?>
```

现在将上述代码保存到名为`class.counter.php`的文件中。当您运行上述代码时，您将得到以下输出：

**X-Powered-By: PHP/5.1.1**

**内容类型: text/html**

**===> 用户定义的类 'Counter' [扩展 ReflectionClass::__set_state(array(**

**'name' => 'Object',**

**))]**

**在 PHPDocument2 中声明**

**第 15 行到第 29 行**

**具有修饰符 0 []**

**---> 文档:**

**'/****

*** 一个计数器类**

***/'**

**---> 实现:**

**array (**

**0 =>**

**ReflectionClass::__set_state(array(**

**'name' => 'NSerializable',**

**)),**

**)**

**---> 常量: array (**

**'START' => 0,**

**)**

**---> 属性: array (**

**0 =>**

**ReflectionProperty::__set_state(array(**

**'name' => 'c',**

**'class' => 'Counter',**

**)),**

**)**

**---> 方法: array (**

**0 =>**

**ReflectionMethod::__set_state(array(**

**'name' => 'count',**

**'class' => 'Counter',**

**)),**

**)**

**---> $counter 是实例？是**

**---> new Object() 是实例？否**

# ReflectionMethod

这是一个用于调查类的任何方法并调用它的类。让我们看看这个类的结构：

```php
<?php
class ReflectionMethod extends ReflectionFunction
{
   public __construct(mixed class, string name)
   public string __toString()
   public static string export(mixed class, string name, bool return)
   public mixed invoke(stdclass object, mixed args)
   public mixed invokeArgs(stdclass object, array args)
   public bool isFinal()
   public bool isAbstract()
   public bool isPublic()
   public bool isPrivate()
   public bool isProtected()
   public bool isStatic()
   public bool isConstructor()
   public bool isDestructor()
   public int getModifiers()
   public ReflectionClass getDeclaringClass()

   // Inherited from ReflectionFunction
   final private __clone()
   public string getName()
   public bool isInternal()
   public bool isUserDefined()
   public string getFileName()
   public int getStartLine()
   public int getEndLine()
   public string getDocComment()
   public array getStaticVariables()
   public bool returnsReference()
   public ReflectionParameter[] getParameters()
   public int getNumberOfParameters()
   public int getNumberOfRequiredParameters()
}
?>

```

这个类最重要的方法是`getNumberOfParameters`、`getNumberOfRequiredParameters`、`getParameters`和`invoke`。前三个方法很容易理解；让我们看看第四个方法，即调用。这是一个来自 PHP 手册的精彩示例：

```php
<?php
class Counter
{
   private static $c = 0;

   /**
    * Increment counter
    *
    * @final
    * @static
    * @access  public
    * @return  int
    */
   final public static function increment()
   {
       return ++self::$c;
   }
}
// Create an instance of the Reflection_Method class
$method = new ReflectionMethod('Counter', 'increment');
// Print out basic information
printf(
   "===> The %s%s%s%s%s%s%s method '%s' (which is %s)\n" .
   "     declared in %s\n" .
   "     lines %d to %d\n" .
   "     having the modifiers %d[%s]\n",
       $method->isInternal() ? 'internal' : 'user-defined',
       $method->isAbstract() ? ' abstract' : '',
       $method->isFinal() ? ' final' : '',
       $method->isPublic() ? ' public' : '',
       $method->isPrivate() ? ' private' : '',
       $method->isProtected() ? ' protected' : '',
       $method->isStatic() ? ' static' : '',
       $method->getName(),
       $method->isConstructor() ? 'the constructor' : 
                                      'a regular method',
       $method->getFileName(),
       $method->getStartLine(),
       $method->getEndline(),
       $method->getModifiers(),
       implode(' ', Reflection::getModifierNames(
                                 $method->getModifiers()))
       );

// Print documentation comment
printf("---> Documentation:\n %s\n", 
                  var_export($method->getDocComment(), 1));

// Print static variables if existant
if ($statics= $method->getStaticVariables()) {
   printf("---> Static variables: %s\n", var_export($statics, 1));
}

// Invoke the method
printf("---> Invokation results in: ");
var_dump($method->invoke(NULL));
?>
```

当执行此代码时，将给出以下输出：

```php
===> The user-defined final public static method 'increment' (which is a regular method)
 declared in PHPDocument1
 lines 14 to 17
 having the modifiers 261[final public static]
---> Documentation:
 '/**
 * Increment counter
 *
 * @final
 * @static
 * @access  public
 * @return  int
 */'
---> Invokation results in: int(1)

```

# ReflectionParameter

反射家族中另一个非常重要的对象是`ReflectionParameter`。使用这个类，您可以分析任何方法的参数并相应地采取行动。让我们看看这个对象的结构：

```php
<?php
class ReflectionParameter implements Reflector
{
   final private __clone()
   public object __construct(string name)
   public string __toString()
   public static string export(mixed function, mixed parameter, 
                                                      bool return)
   public string getName()
   public bool isPassedByReference()
   public ReflectionFunction getDeclaringFunction()
   public ReflectionClass getDeclaringClass()
   public ReflectionClass getClass()
   public bool isArray()
   public bool allowsNull()
   public bool isPassedByReference()
   public bool getPosition()
   public bool isOptional()
   public bool isDefaultValueAvailable()
   public mixed getDefaultValue()
}
?>
```

为了使事情更简单，请查看以下示例以了解这个功能是如何工作的。

```php
<?php
function foo($a, $b, $c) { }
function bar(Exception $a, &$b, $c) { }
function baz(ReflectionFunction $a,  $b = 1, $c = null) { }
function abc() { }

// Create an instance of Reflection_Function with the
// parameter given from the command line.    
$reflect = new ReflectionFunction("baz");
echo $reflect;
foreach ($reflect->getParameters() as $i => $param) 
{
   printf(
       "-- Parameter #%d: %s {\n".
       "   Class: %s\n".
       "   Allows NULL: %s\n".
       "   Passed to by reference: %s\n".
       "   Is optional?: %s\n".
       "}\n",
       $i, 
       $param->getName(),
       var_export($param->getClass(), 1),
       var_export($param->allowsNull(), 1),
       var_export($param->isPassedByReference(), 1),
       $param->isOptional() ? 'yes' : 'no'
   );
}
?>

```

如果您运行上述代码片段，您将得到以下输出：

```php
Function [ <user> <visibility error> function baz ] 
{
 @@ C:\OOP with PHP5\Codes\ch5\test.php 4 - 4
 - Parameters [3] 
 {
 Parameter #0 [ <required> ReflectionFunction &$a ]
 Parameter #1 [ <optional> $b = 1 ]
 Parameter #2 [ <optional> $c = NULL ]
 }
}
-- Parameter #0: a 
{
 Class: ReflectionClass::__set_state(array(
 'name' => 'ReflectionFunction',
))
 Allows NULL: false
 Passed to by reference: true
 Is optional?: no
}
-- Parameter #1: b 
{
 Class: NULL
 Allows NULL: true
 Passed to by reference: false
 Is optional?: yes
}
-- Parameter #2: c 
{
 Class: NULL
 Allows NULL: true
 Passed to by reference: false
 Is optional?: yes
}

```

# ReflectionProperty

这是我们要在这里讨论的反射家族中的最后一个。这个类帮助你调查类属性并逆向工程它们。这个类具有以下结构：

```php
<?php
class ReflectionProperty implements Reflector
{
   final private __clone()
   public __construct(mixed class, string name)
   public string __toString()
   public static string export(mixed class, string name, bool return)
   public string getName()
   public bool isPublic()
   public bool isPrivate()
   public bool isProtected()
   public bool isStatic()
   public bool isDefault()
   public int getModifiers()
   public mixed getValue(stdclass object)
   public void setValue(stdclass object, mixed value)
   public ReflectionClass getDeclaringClass()
   public string getDocComment()
}
?>
```

下面是一个直接从 PHP 手册中摘取的例子，有助于描述它实际上是如何工作的。

```php
<?php
class String
{
   public $length  = 5;
}

// Create an instance of the ReflectionProperty class
$prop = new ReflectionProperty('String', 'length');
// Print out basic information
printf(
   "===> The%s%s%s%s property '%s' (which was %s)\n" .
   "     having the modifiers %s\n",
       $prop->isPublic() ? ' public' : '',
       $prop->isPrivate() ? ' private' : '',
       $prop->isProtected() ? ' protected' : '',
       $prop->isStatic() ? ' static' : '',
       $prop->getName(),
       $prop->isDefault() ? 'declared at compile-time' : 
                                     'created at run-time',
       var_export(Reflection::getModifierNames(
                                   $prop->getModifiers()), 1)
      );  

// Create an instance of String
$obj= new String();

// Get current value
printf("---> Value is: ");
var_dump($prop->getValue($obj));

// Change value
$prop->setValue($obj, 10);
printf("---> Setting value to 10, new value is: ");
var_dump($prop->getValue($obj));

// Dump object
var_dump($obj);
?>
```

在执行时，代码产生以下输出。此代码使用`ReflectionProperty`检查一个属性，并显示以下输出：

```php
===> The public property 'length' (which was declared at compile-time)
 having the modifiers array (
 0 => 'public',
)
---> Value is: int(5)
---> Setting value to 10, new value is: int(10)
object(String)#2 (1) {
 ["length"]=>
 int(10)
}

```

我们将在后面的章节中看到 Reflection API 的更多用途，当我们学习构建 MVC 框架时。

# 单元测试

编程的另一个非常重要的部分是单元测试，通过它可以测试代码片段，是否工作得完美。你可以针对代码的任何版本编写测试用例，以检查重构后代码是否工作正常。单元测试确保代码的可工作性，并在问题发生时帮助定位问题。当你编写应用程序时，单元测试就像你的骨架。单元测试是所有语言程序员的编程必经之路。几乎所有主要的编程语言都有单元测试包可用。

与其他任何编程语言一样，有一个 Java 包被认为是其他语言每个单元测试包的标准模型。这个包被称为**JUnit**，它是为 Java 开发者准备的。JUnit 中维护的标准和测试风格通常被许多其他单元测试包所遵循。因此，JUnit 已经成为单元测试领域的默认选择。为 PHP 开发者提供的 JUnit 版本被称为**PHPUnit**，由 Sebastian Bergmann 开发。PHPUnit 是一个非常流行的单元测试包。

编写单元测试的主要原因是，如果你只是编写代码并部署应用程序，你无法找出所有的错误。可能会有一些小错误，通过返回一个不相关的值，可能会使你的应用程序突然崩溃。不要忽视这些小场景。可能会有你无法想象到你的代码返回一个极其奇怪结果的情况。单元测试通过编写不同的测试用例来帮助你。单元测试不是一件需要花费很多时间来编写的事情，然而结果却是惊人的。

在下一节中，我们将学习单元测试的基础知识，并亲自动手编写成功的单元测试。

## 单元测试的好处

单元测试有很多好处，其中一些是它：

+   确保你的应用程序的一致性。

+   确保在重构后你的完整应用程序仍然可用。

+   检查冗余并从你的代码中移除它们。

+   设计良好的 API。

+   可以轻松地找出问题所在。

+   如果出现问题，可以加快调试过程；正如你所知，尤其是你知道错误所在的地方。

+   通过提供 API 的工作示例来最小化文档的工作量。

+   帮助进行回归测试，以确保不再发生回归。

## 脆弱漏洞的简要介绍

缺陷可能有不同类型。一些缺陷可能会困扰你的用户，一些缺陷会停止功能，而一些缺陷漏洞会损坏你的资源。让我们考虑以下示例。你编写了一个函数，它接受两个参数并相应地更新数据库。第一个参数是字段的名称，第二个参数是那个字段的值，通过这个值它应该定位数据并更新它们。现在让我们设计它：

```php
function selectUser($field, $condition)
{
  if (!empty($condition))
  {
    $query = "{$field}= '{$condition}'";
  }
  else 
      $query = "{$field}";
      echo "select * from users where {$query}";
  $result = mysql_query("select * from users where {$query}");
  $results = array();
  while ($data = mysql_fetch_array($result))
  {
    $results[] = $data;
  }
  return $results;
}
```

现在当你这样调用它时，它会显示特定的数据：

```php
print_r(selectUser("id","1");
```

输出如下：

```php
(
  [0] => Array
    (
      [0] => 1
      [id] => 1
      [1] => afif
      [name] => afif
      [2] => 47bce5c74f589f4867dbd57e9ca9f808
      [pass] => 47bce5c74f589f4867dbd57e9ca9f808
    )
)
```

但是当你这样调用它时：

```php
print_r(selectUser("id",$_SESSION['id']);
```

它显示了以下内容：

```php
(
  [0] => Array
    (
      [0] => 1
      [id] => 1
      [1] => afif
      [name] => afif
      [2] => 47bce5c74f589f4867dbd57e9ca9f808
      [pass] => 47bce5c74f589f4867dbd57e9ca9f808
    )

   1] => Array
    (
      [0] => 2
      [id] => 2
      [1] => 4b8ed057e4f0960d8413e37060d4c175
      [name] => 4b8ed057e4f0960d8413e37060d4c175
      [2] => 74b87337454200d4d33f80c4663dc5e5
      [pass] => 74b87337454200d4d33f80c4663dc5e5
    )
)
```

这不是一个正确的输出；并且如果在运行时它是一个`update`查询而不是`select`查询，你的整个数据可能会被损坏。那么你如何确保输出始终是有效的呢？嗯，我们将在本章的后面通过单元测试轻松地做到这一点。

## 准备进行单元测试

要使用 PHPUnit 为 PHP 应用程序编写成功的单元测试，你需要下载该包，配置它，然后在实际执行测试之前做一些小任务。

你可以从命令行或从你的脚本内部运行 PHPUnit 测试。目前，我们将从我们的脚本内部运行测试，但在后面的章节中，我们将学习如何从命令行运行单元测试。

首先，从[`www.phpunit.de`](http://www.phpunit.de)下载该包，并将其提取到你的包含路径中。如果你不确定你的包含路径是什么，你可以从`php.ini`中的`include_path`设置中获取它。或者，你可以执行以下 PHP 脚本来显示输出：

```php
<?
echo get_include_path()
?>
```

现在，提取 PHPUnit 存档，并将 PHPUnit 文件夹放置在包含路径中的一个文件夹中。这个 PHPUnit 文件夹包含两个其他文件夹，分别命名为`PHPUnit`和`PHPUnit2`。

当你将文件夹放置在你的包含路径目录中时，你就完成了。现在我们准备出发了。

## 开始单元测试

单元测试实际上是对你的代码进行的一系列不同测试。使用 PHPUnit 编写单元测试并不是一项大工程。你只需要简单地遵循一套约定。让我们看看以下示例，其中你创建了一个字符串处理类，该类返回字符串中可用的单词数量。

```php
<?
//class.wordcount.php
class wordcount
{
  public function countWords($sentence)
  {
    return count(split(" ",$sentence));
  }
}
?>
```

现在，我们将为这个类编写一个单元测试。我们必须扩展`PHPUnit_Framework_TestCase`来编写任何单元测试。我们必须使用`PHPUnit_Framework_TestSuite`来创建测试套件，它实际上包含了一系列测试。然后我们将使用`PHPUnit_TextUI_TestRunner`从套件中运行测试并打印结果。

```php
<?
//class.testwordcount.php
require_once "PHPUnit/Framework/TestCase.php";
require_once "class.wordcount.php";

class TestWordCount extends PHPUnit_Framework_TestCase 
{
  public function testCountWords()
  {
    $Wc = new WordCount();
    $TestSentence = "my name is afif";
    $WordCount = $Wc->countWords($TestSentence);
    $this->assertEquals(4,$WordCount);
  }
}
?>
```

运行测试：

```php
<?
//testsuite.wordcount.php
require_once 'PHPUnit/TextUI/TestRunner.php';
require_once "PHPUnit/Framework/TestSuite.php";
require_once "class.testwordcount.php";

$suite = new PHPUnit_Framework_TestSuite();
$suite->addTestSuite("TestWordCount");
PHPUnit_TextUI_TestRunner::run($suite);
?>
```

现在如果你在`testsuite.wordcount.php`中运行代码，你将得到以下输出：

```php
PHPUnit 3.0.5 by Sebastian Bergmann.
Time: 00:00
OK (1 test)
```

这意味着我们的测试已经通过，我们的单词计数函数工作得非常完美，然而，我们还将为该函数编写更多的测试用例。

让我们在`class.testwordcount.php`中添加这个新的测试用例：

```php
public function testCountWordsWithSpaces()
{
  $wc= new WordCount();
  $testSentence = "my name is Anonymous ";
  $wordCount = $Wc->countWords($testSentence);
  $this->assertEquals(4,$wordCount);		
}
```

现在如果我们运行我们的测试套件，我们将得到以下结果：

```php
PHPUnit 3.0.5 by Sebastian Bergmann.

.F

Time: 00:00

There was 1 failure:

1) testCountWordsWithSpaces(TestWordCount)
Failed asserting that <integer:5> is equal to <integer:4>.
C:\OOP with PHP5\Codes\ch5\UnitTest\FirstTest.php:34
C:\OOP with PHP5\Codes\ch5\UnitTest\FirstTest.php:40
C:\Program Files\Zend\ZendStudio-5.2.0\bin\php5\dummy.php:1

FAILURES!
Tests: 2, Failures: 1.
```

在这里，我们发现我们的万无一失的单词计数函数失败了。那么我们的测试输入是什么？我们只是在测试参数`my` `name` `is` `afif`中添加了更多的空格，然后我们的函数失败了。这是因为它用空白字符分割句子，并返回分割的部分数。因为有更多的空白字符，所以我们的函数优雅地失败了。这是一个相当好的测试用例；我们发现，如果我们用这个版本的单词计数器发布我们的代码，我们的函数在现实生活中可能会失败。PHPUnit 已经对我们很有用了。现在我们将解决我们的函数，使其在句子包含更多空格时返回正确的结果。我们将`class.wordcount.php`更改为这个新版本：

```php
class WordCount
{
  public function countWords($sentence)
  {
    $newsentence = preg_replace("~\s+~"," ",$sentence);
    return count(split(" ",$newsentence));
  }
}
```

现在如果我们运行我们的测试套件，它将给出以下输出。

```php
PHPUnit 3.0.5 by Sebastian Bergmann.

..

Time: 00:00

OK (2 tests)
```

然而，我们希望有更多的证据表明我们的函数在野外将工作得更好。因此，我们正在编写另一个测试用例。让我们在`class.testwordcount.php`中添加这个新的测试用例：

```php
public function testCountWordsWithNewLine()
{
  $Wc = new WordCount();
  $TestSentence = "my name is \n\r Anonymous";
  $WordCount = $Wc->countWords($TestSentence);
  $this->assertEquals(4,$WordCount);
}
```

让我们再次运行这个套件。现在结果是什么？

```php
PHPUnit 3.0.5 by Sebastian Bergmann.

...

Time: 00:00

OK (3 tests)
```

这相当令人满意。所有的测试都在正常运行。现在这个函数已经很好了。

这就是单元测试如何在现实生活中帮助我们。

## 测试电子邮件验证器对象

现在，让我们再次重复这些步骤。这次我们将为我们的全新`Emailvalidator`类编写单元测试，我们的开发者说这是一个好的类。让我们首先看看我们的验证器函数：

```php
//class.emailvalidator.php
class EmailValidator
{
  public function validateEmail($email)
  {
    $pattern = "/[A-z0-9]{1,64}@[A-z0-9]+\.[A-z0-9]{2,3}/";
    preg_match($pattern, $email,$matches);
    return (strlen($matches[0])==strlen($email)?true:false);
  }
}
?>
```

接下来是我们的测试用例：

```php
class TestEmailValidator extends PHPUnit_Framework_TestCase 
{
  private $Ev;
  protected function setup()
  {
    $this->Ev = new EmailValidator();
  }

  protected  function tearDown()
  {
    unset($this->Ev);
  }

  public function testSimpleEmail()
  {
    $result = $this->Ev->validateEmail("has.in@somewherein.net");
    $this->assertTrue($result);
  }
}
```

现在你必须编写测试套件并运行：

```php
$suite = new PHPUnit_Framework_TestSuite();
$suite->addTestSuite("TestEmailValidator");
PHPUnit_TextUI_TestRunner::run($suite);
```

当你运行这个测试套件时，你会得到以下输出：

```php
PHPUnit 3.0.5 by Sebastian Bergmann.

...

Time: 00:00

OK (1 test)
```

现在更加努力；尝试破坏你的代码。尝试所有可能出现在电子邮件中的情况，尽可能多地尝试。我们将添加更多的测试用例：

```php
class TestEmailValidator extends PHPUnit_Framework_TestCase 
{
  private $Ev;
  protected function setUp()
  {
    $this->Ev = new EmailValidator();
  }

  protected  function tearDown()
  {
    unset($this->Ev);
  }

  public function testSimpleEmail()
  {
    $result = $this->Ev->validateEmail("hasin@somewherein.net");
    $this->assertTrue($result);
  }

  public function testEmailWithDotInName()
  {
    $result = $this->Ev->validateEmail("has.in@somewherein.net");
    $this->assertTrue($result);
  }

  public function testEmailWithComma()
  {
    $result = $this->Ev->validateEmail("has,in@somewherein.net");
    $this->assertFalse($result);
  }

  public function testEmailWithSpace()
  {
    $result = $this->Ev->validateEmail("has in@somewherein.net");
    $this->assertTrue($result);
  }

  public function testEmailLengthMoreThan64Char()
  {
    $result = 
    $this->Ev->validateEmail(str_repeat("h",67)."@somewherein.net");
    $this->assertFalse($result);
  }

  public function testEmailWithInValidCharacters()
  {
    $result = $this->Ev->validateEmail("has#in@somewherein.net");
    $this->assertFalse($result);
  }

  public function testEmailWithNoDomain()
  {
    $result = $this->Ev->validateEmail("hasin@");
    $this->assertFalse($result);
  }

  public function testEmailWithInvalidDomain()
  {
    $result = 
       $this->Ev->validateEmail("hasin@somewherein.comnetorg");
    $this->assertFalse($result);
  }
}
```

当你运行测试套件时，你会得到以下结果：

```php
PHPUnit 3.0.5 by Sebastian Bergmann.

.F.F....

Time: 00:00

There were 1 failures:

1) testEmailWithDotInName(TestEmailValidator)
Failed asserting that <boolean:false> is identical to <boolean:true>.
C:\OOP with PHP5\Codes\ch5\UnitTest\EmailValidatorTest.php:40
C:\OOP with PHP5\Codes\ch5\UnitTest\EmailValidatorTest.php:83
C:\Program Files\Zend\ZendStudio-5.2.0\bin\php5\dummy.php:1
```

```php
FAILURES!
Tests: 8, Failures: 1.

```

因此，我们的电子邮件验证器失败了！如果你查看结果，你会看到它因`testEmailWithDotInName`而失败。因此，我们必须更改我们使用的正则表达式模式，并允许在名称中使用`.`。

让我们按照以下方式重新设计验证器：

```php
class EmailValidator
{
  public function validateEmail($email)
  {
    $pattern = "/[A-z0-9\.]{1,64}@[A-z0-9]+\.[A-z0-9]{2,3}/";
    preg_match($pattern, $email,$matches);
    return (strlen($matches[0])==strlen($email)?true:false);
  }
}
```

现在你再次运行你的测试套件，你会看到以下输出：

```php
PHPUnit 3.0.5 by Sebastian Bergmann.

........

Time: 00:00

OK (8 tests)
```

我们的测试通过了。

那么好处是什么？一次又一次，当你需要向你的正则表达式添加新的验证规则时，这个单元测试将帮助你进行回归测试，以确保同样的错误不再发生。

这就是单元测试的美丽之处。

### 注意

你会在上面的例子中找到两个名为`setUp()`和`tearDown()`的函数。`setUp()`用于为测试设置一切；你可以用它来连接到数据库，打开一个文件或类似的东西。`tearDown()`用于清理。它在脚本执行完毕时被调用。

## 每日脚本单元测试

除了这些函数和小类的单元测试之外，你还需要为不同函数最终得到的结果编写单元测试。然而，你的单元测试越具体，你预期的结果就越好。也要记住，在你写的许多单元测试中，只有少数是有用的。

现在我们将讨论如何测试与数据库一起工作的例程。让我们创建一个小的类，它可以直接与数据库中的`users`表交互，我们将为它编写单元测试。以下是我们的小型类，它直接与数据库中的`users`表交互。

```php
<?
class DB
{
  private $connection;

  public function __construct()
  {
    $this->connection = mysql_connect("localhost","root","root1234");
    mysql_select_db("test",$this->connection);
  }
  public function insertData($data)
  {
    $fields = join(array_keys($data),",");
    $values = "'".join(array_values($data),",")."'";

    $query = "INSERT INTO users({$fields}) values({$values})";
    return mysql_query($query, $this->connection);
  }

  public function deleteData($id)
  {
    $query = "delete from users where id={$id}";
    return mysql_query($query, $this->connection);
  }

  public function updateData($id, $data)
  {
    $queryparts = array();
    foreach ($data as $key=>$value)
    {
      $queryparts[] = "{$key} = '{$value}'";
    }

    $query = "UPDATE users SET ".join($queryparts,",")." 
                                           WHERE id='{$id}'";
    return mysql_query($query, $this->connection);
  }
}
?>
```

我们需要测试这个类中的所有公共方法，以确保它们正常工作。因此，我们的测试用例如下。

```php
<?
require_once "PHPUnit/Framework/TestCase.php";

class DBTester extends PHPUnit_Framework_TestCase
{
  private $connection;
  private $Db;

  protected function setup()
  {
    $this->Db = new DB();

    $this->connection = mysql_connect("localhost","root","root1234");
    mysql_select_db("abcd",$this->connection);
  }

  protected  function tearDown()
  {
    mysql_close($this->connection);
  }

  public function testValidInsert()
  {
    $data = array("name"=>"afif","pass"=>md5("hello world"));
    mysql_query("delete from users");

    $result = $this->Db->insertData($data);
    $this->assertNotNull($result);

    $affected_rows = mysql_affected_rows($this->connection);
    $this->assertEquals(1, $affected_rows);
  }

  public function testInvalidInsert()
  {
    $data = array("names"=>"afif","passwords"=>md5("hello world"));
    mysql_query("delete from users");

    $result = $this->Db->insertData($data);
    $this->assertNotNull($result);

    $affected_rows = mysql_affected_rows($this->connection);
    $this->assertEquals(-1, $affected_rows);
  }

  public function testUpdate()
  {
    $data = array("name"=>"afif","pass"=>md5("hello world"));
    mysql_query("truncate table users");

    $this->Db->insertData($data);

    $data = array("name"=>"afif2","pass"=>md5("hello world"));
    $result = $this->Db->updateData(1, $data);
    $this->assertNotNull($result);

    $affected_rows = mysql_affected_rows($this->connection);
    $this->assertEquals(1, $affected_rows);
  }

  public function testDelete()
  {
    $data = array("name"=>"afif","pass"=>md5("hello world"));
mysql_query("truncate table users");

    $this->Db->insertData($data);
    $result = $this->Db->deleteData(1);
    $this->assertNotNull($result);

    $affected_rows = mysql_affected_rows($this->connection);
    $this->assertEquals(1, $affected_rows);

  }
}
?>
```

测试套件如下：

```php
<?
require_once 'PHPUnit/TextUI/TestRunner.php';
require_once "PHPUnit/Framework/TestSuite.php";
$suite = new PHPUnit_Framework_TestSuite();
$suite->addTestSuite("DBTester");
PHPUnit_TextUI_TestRunner::run($suite);

?>
```

你会得到什么结果呢？

```php
PHPUnit 3.0.5 by Sebastian Bergmann.

....

Time: 00:00

OK (4 tests)
```

然而，这些都是基本功能测试。我们必须创建更多样化的测试，并找出我们的对象可能失败的方式。让我们添加两个更多的测试，如下所示：

```php
public function testInvalidUpdate()
  {
    $data = array("name"=>"afif","pass"=>md5("hello world"));
    mysql_query("truncate table users");

    $this->Db->insertData($data);

    $data = array("name"=>"afif2","pass"=>md5("hello world"));
    $result = $this->Db->updateData(2, $data);

    $affected_rows = mysql_affected_rows($this->connection);
    $this->assertEquals(0, $affected_rows);
}

  public function testInvalidDelete()
  {
    $data = array("name"=>"afif","pass"=>md5("hello world"));
    mysql_query("truncate table users");

    $this->Db->insertData($data);
    $result = $this->Db->deleteData("*");
    $this->assertNotNull($result);

    $affected_rows = mysql_affected_rows($this->connection);
    $this->assertEquals(-1, $affected_rows);

}
```

现在如果你运行测试套件，你会得到以下结果：

```php
PHPUnit 3.0.5 by Sebastian Bergmann.

......

Time: 00:00

OK (6 tests)
```

我们的数据库代码看起来很难破坏。

在现实生活中的单元测试中，你需要考虑如何破坏你自己的代码。如果你能编写破坏现有代码的单元测试，那就更好了。

## 测试驱动开发

现在是时候进一步学习单元测试了。你可能想知道在为应用程序编码之前何时需要编写单元测试：是在开发期间，还是编码完成后？嗯，来自不同领域的开发者有不同的看法，然而发现先编写测试然后进行实际应用更为有用。这被称为**测试驱动开发**或简称**TDD**。TDD 可以帮助你为应用程序设计更好的 API。

你可能会问在没有实际代码的情况下如何编写测试，以及要测试哪些内容？你不需要真实对象进行 TDD。只需想象一些模拟对象，它们只具有函数。你将使用这些函数与想象的结果。你也可以编写不完整的测试，这意味着一个空体的测试。在你方便的时候，你可以编写测试的内容。让我们看看以下示例，了解在实际代码编写之前的单元测试是如何适合项目开发的。

PHPUnit 为你提供了许多用于测试优先编程的有用 API，例如`markTestSkipped()`和`markTestIncomplete()`。我们将使用这两个方法来标记一些尚未实现的自定义测试。让我们设计一个小型反馈管理器，它可以接受用户的反馈并将邮件发送给你。那么反馈管理器有哪些有用的功能呢？我建议以下功能：

+   它可以生成一个反馈表单。

+   它将处理用户的输入并正确过滤它。

+   它将具有防垃圾邮件功能。

+   它将防止任何由机器人或垃圾邮件发送者提交的自动化反馈。

+   在提交反馈后，它会生成一个确认，并将邮件发送给所有者。

让我们为这个创建一些空白单元测试。以下是我们的测试用例，在我们有实际代码之前：

```php
<?
class FeedbackTester extends PHPUnit_Framework_TestCase
{
  public function testUsersEmail()
  {
    $this->markTestIncomplete();
  }
  public function testInvalidDomain()
  {
    $this->markTestIncomplete();
  }
  public function testCaptchaGenerator()
  {
    $this->markTestIncomplete();
  }
  public function testCaptchaChecker()
  {
    $this->markTestIncomplete();
  }
  public function testFormRenderer()
  {
    $this->markTestIncomplete();
  }
  public function testFormHandler()
  {
    $this->markTestIncomplete();
  }
  public function testValidUserName()
  {
    $this->markTestIncomplete();
  }
  public function testValidSubject()
  {
    $this->markTestIncomplete();
  }
  public function testValidContent()
  {
    $this->markTestIncomplete();
  }
  public function testFeedbackSender()
  {
    $this->markTestIncomplete();
  }
  public function testConfirmer()
  {
    $this->markTestIncomplete();
  }

}
?>
```

这很好；我们现在已经创建了 11 个空白测试。现在如果你使用测试套件运行这个测试用例，你会得到以下结果：

```php
PHPUnit 3.0.5 by Sebastian Bergmann.

IIIIIIIIIII

Time: 00:00

OK, but incomplete or skipped tests!
Tests: 11, Incomplete: 11.
```

PHPUnit 成功识别出我们所有的测试都被标记为不完整。现在让我们再思考一下。如果你生成一个 `InputValidator` 对象，它验证用户输入并过滤掉所有恶意数据，那么我们可能只有一个测试用例，即 `testValidInput()`，而不是所有的这些 `testValidUserName()`、`testValidSubject()`、`testValidContent()`。因此，我们可以跳过这些测试。现在让我们创建新的测试例程 `testValidInput()` 并将其标记为不完整：

```php
public function testValidInput()
{
  $this->markTestIncomplete();
}
```

对于我们计划跳过的三个测试，我们将如何处理？我们不会删除它们，但会将它们标记为跳过。将 `$this->markTestIncomplete()` 行修改为 `$this->markTestSkipped()`。例如：

```php
public function testValidUserName()
{
  $this->markTestSkipped();
}
```

现在如果你再次运行你的测试套件，你会得到以下结果：

```php
PHPUnit 3.0.5 by Sebastian Bergmann.

IIIIIISSSIII

Time: 00:00

OK, but incomplete or skipped tests!
Tests: 12, Incomplete: 9, Skipped: 3.
```

PHPUnit 显示它跳过了三个测试。

为了使我们的讨论简短并集中，我们现在将只实现这九个测试中的一个。我们将测试反馈表单渲染器是否真正工作正常。

现在我们来看一下我们测试用例中修改后的测试例程 `testFormRenderer()`。

```php
public function testFormRenderer(){

  $testResult = true;
  $message = "";
  $Fm= new FeedbackManager();
  ob_start();
  $Fm->renderFeedbackForm();
  $output = ob_get_clean();

  if (strpos($output, "name='email'")===false && $testResult==true) 
  list($testResult, $message) = array(false, 
                                   "Email field is not present");
  if (strpos($output, "name='username'")===false && 
                                           $testResult==true) 
  list($testResult, $message) = array(false, 
                             "Username is field not present");

  if (strpos($output, "name='subject'")===false && $testResult==true) 
  list($testResult, $message) = array(false, 
                                     "Subject field is not present");

  if (strpos($output, "name='message'")===false && $testResult==true) 
  list($testResult, $message) = array(false, 
                                    "Message field is not present");

  $this->assertTrue($testResult, $message);
  //$this->markTestIncomplete();
}
```

它清楚地表明，在我们的反馈管理器中必须有一个名为 `renderFeedbackForm()` 的方法，并且在生成的输出中必须有四个输入字段，即 `email`、`subject`、`username` 和 `message`。现在让我们创建我们的 `FeedBackManager` 对象。以下是具有单个渲染反馈表单方法的 `FeedBackManager`：

```php
class FeedBackManager 
<?
{

  public function renderFeedbackForm()
  {
    $form = <<< END
    <form method=POST action="">
      Name: <br/>
      <input type='text' name='username'><br/>
      Email: <br/>
      <input type='text' name='email'><br/>
      Subject: <br/>
      <input type='text' name='subject'><br/>

      <input type='submit' value='submit>	
    </form>
    END;
    echo $form;
  }
}
?>
```

现在如果你运行单元测试套件，你会得到以下结果：

```php
PHPUnit 3.0.5 by Sebastian Bergmann.

IIIIFISSSIII

Time: 00:00

There was 1 failure:

1) testFormRenderer(FeedbackTester)
Message field is not present
Failed asserting that <boolean:false> is identical to <boolean:true>.
C:\OOP with PHP5\Codes\ch5\UnitTest\BlankTest.php:52
C:\OOP with PHP5\Codes\ch5\UnitTest\BlankTest.php:104
C:\Program Files\Zend\ZendStudio-5.2.0\bin\php5\dummy.php:1

FAILURES!
Tests: 12, Failures: 1, Incomplete: 8, Skipped: 3.
```

我们的表单渲染器失败了。为什么？看看 PHPUnit 输出的结果。它说 `Message` `field` `is` `not` `present`。哦！我们忘记放置一个名为 `message` 的 `textarea` 对象了。让我们修改我们的 `renderFeedbackForm()` 方法并纠正它。

```php
class FeedBackManager 
{

  public function renderFeedbackForm()
  {
    $form = <<< END
    <form method=POST action="">
      Name: <br/>
      <input type='text' name='username'><br/>
      Email: <br/>
      <input type='text' name='email'><br/>
       Subject: <br/>
      <input type='text' name='subject'><br/>
 Message: <br/>
 <textarea name='message'></textarea><br/>
      <input type='submit' value='submit>	
    </form>
END;
    echo $form;
  }
}
```

我们已经添加了消息字段。现在让我们再次运行套件。你会得到以下输出：

```php
PHPUnit 3.0.5 by Sebastian Bergmann.

IIII.ISSSIII

Time: 00:00

OK, but incomplete or skipped tests!
Tests: 12, Incomplete: 8, Skipped: 3.
```

太好了！我们的测试通过了。这意味着我们的渲染表单可能没有错误。

这是测试驱动开发（TDD）的风格。在实际编写代码之前，你必须预见你的应用程序代码。使用 TDD 可以帮助你设计良好的 API 和良好的代码。

### 编写多个断言

不要在一个测试下写多个断言。按照上面的示例进行拆分。为了澄清，以下示例是一个单元测试的糟糕例子。

```php
public function testFormRenderer(){

  $testResult = true;
  $message = "";
  $Fm = new FeedBackManager();
  ob_start();
  $Fm->renderFeedbackForm();
  $output = ob_get_clean();

  $testResult = strpos($output, "name='email'");
  $this->assertEquals(true, $testResult,
                            "Email field is not present");

  $testResult = strpos($output, "name='username'");
  $this->assertEquals(true, $testResult,
                          "Username field is not present");

  $testResult = strpos($output, "name='subject'");
  $this->assertEquals(true, $testResult,
                           "Subject field is not present");

  $testResult = strpos($output, "name='message'");
  $this->assertEquals(true, $testResult,
                           "Message field is not present");
}
```

这段代码将会运行，但在单个例程中多个断言是被禁止的，并且违反了良好的应用程序设计。

## PHPUnit API

PHPUnit 提供了几种断言 API。在我们的示例中，我们使用了 `assertTrue()`、`assertEquals()`、`assertFalse()` 和 `assertNotNull()` 等函数。然而，还有更多。函数名是自我解释的。以下表格取自 Sebastian Bergmann 本人撰写的《PHPUnit 口袋指南》一书，由 O'Reilly 出版。这本书在 Creative Commons 许可下免费提供。这本书的最新版本目前可在[`www.phpunit.de/pocket_guide/3.0/en/index.html`](http://www.phpunit.de/pocket_guide/3.0/en/index.html)找到。

以下表格显示了 PHPUnit 所有可能的断言函数：

| 断言 | 含义 |
| --- | --- |
| `void` `assertTrue(bool` `$condition)` | 如果 `$condition` 是 `FALSE`，则报告一个错误。 |
| `void` `assertTrue(bool` `$condition,` `string` `$message)` | 如果 `$condition` 是 `FALSE`，则通过 `$message` 指定的错误信息报告一个错误。 |
| `void` `assertFalse(bool` `$condition)` | 如果 `$condition` 是 `TRUE`，则报告一个错误。 |
| `void` `assertFalse(bool` `$condition,` `string` `$message)` | 如果 `$condition` 是 `TRUE`，则通过 `$message` 指定的错误信息报告一个错误。 |
| `void` `assertNull(mixed` `$variable)` | 如果 `$variable` 不是 `NULL`，则报告一个错误。 |
| `void` `assertNull(mixed` `$variable,` `string` `$message)` | 如果 `$variable` 不是 `NULL`，则通过 `$message` 指定的错误信息报告一个错误。 |
| `void` `assertNotNull(mixed` `$variable)` | 如果 `$variable` 是 `NULL`，则报告一个错误。 |
| `void` `assertNotNull(mixed` `$variable,` `string` `$message)` | 如果 `$variable` 是 `NULL`，则通过 `$message` 指定的错误信息报告一个错误。 |
| `void` `assertSame(object` `$expected,` `object` `$actual)` | 如果两个变量 `$expected` 和 `$actual` 不引用同一个对象，则报告一个错误。 |
| `void` `assertSame(object` `$expected,` `object` `$actual,` `string` `$message)` | 如果两个变量 `$expected` 和 `$actual` 不引用同一个对象，则通过 `$message` 指定的错误信息报告一个错误。 |
| `void` `assertSame(mixed` `$expected,` `mixed` `$actual)` | 如果两个变量 `$expected` 和 `$actual` 不具有相同的类型和值，则报告一个错误。 |
| `void` `assertSame(mixed` `$expected,` `mixed` `$actual,` `string` `$message)` | 如果两个变量 `$expected` 和 `$actual` 不具有相同的类型和值，则通过 `$message` 指定的错误信息报告一个错误。 |
| `void` `assertNotSame(object` `$expected,` `object` `$actual)` | 如果两个变量 `$expected` 和 `$actual` 引用了同一个对象，则报告一个错误。 |
| `void` `assertNotSame(object` `$expected,` `object` `$actual,` `string` `$message)` | 如果两个变量 `$expected` 和 `$actual` 引用了同一个对象，则通过 `$message` 指定的错误信息报告一个错误。 |
| `void` `assertNotSame(mixed` `$expected,` `mixed` `$actual)` | 如果两个变量 `$expected` 和 `$actual` 具有相同的类型和值，则报告一个错误。 |
| `void` `assertNotSame(mixed` `$expected,` `mixed` `$actual,` `string` `$message)` | 报告一个错误，如果两个变量 `$expected` 和 `$actual` 具有相同的类型和值，错误信息由 `$message` 指定。 |
| `void` `assertAttributeSame(object` `$expected,` `string` `$actualAttributeName,` `object` `$actualObject)` | 如果 `$actualObject->actualAttributeName` 和 `$actual` 不引用同一个对象，则报告一个错误。`$actualObject->actualAttributeName` 属性的可见性可能是公共的、受保护的或私有的。 |
| `void` `assertAttributeSame(object` `$expected,` `string` `$actualAttributeName,` `object` `$actualObject,` `string` `$message)` | 如果 `$actualObject->actualAttributeName` 和 `$actual` 不引用相同的对象，则报告由 `$message` 标识的错误。该 `$actualObject->actualAttributeName` 属性的可见性可能是公共的、受保护的或私有的。 |
| `void` `assertAttributeSame(mixed` `$expected,` `string` `$actualAttributeName,` `object` `$actualObject)` | 如果 `$actualObject->actualAttributeName` 和 `$actual` 没有相同的类型和值，则报告一个错误。该 `$actualObject->actualAttributeName` 属性的可见性可能是公共的、受保护的或私有的。 |
| `void` `assertAttributeSame(mixed` `$expected,` `string` `$actualAttributeName,` `object` `$actualObject,` `string` `$message)` | 如果 `$actualObject->actualAttributeName` 和 `$actual` 没有相同的类型和值，则报告由 `$message` 标识的错误。该 `$actualObject->actualAttributeName` 属性的可见性可能是公共的、受保护的或私有的。 |
| `void` `assertAttributeNotSame(object` `$expected,` `string` `$actualAttributeName,` `object` `$actualObject)` | 如果 `$actualObject->actualAttributeName` 和 `$actual` 引用相同的对象，则报告一个错误。该 `$actualObject->actualAttributeName` 属性的可见性可能是公共的、受保护的或私有的。 |
| `void` `assertAttributeNotSame(object` `$expected,` `string` `$actualAttributeName,` `object` `$actualObject,` `string` `$message)` | 如果 `$actualObject->actualAttributeName` 和 `$actual` 引用相同的对象，则报告由 `$message` 标识的错误。该 `$actualObject->actualAttributeName` 属性的可见性可能是公共的、受保护的或私有的。 |
| `void` `assertAttributeNotSame(mixed` `$expected,` `string` `$actualAttributeName,` `object` `$actualObject)` | 如果 `$actualObject->actualAttributeName` 和 `$actual` 具有相同的类型和值，则报告一个错误。该 `$actualObject->actualAttributeName` 属性的可见性可能是公共的、受保护的或私有的。 |
| `void` `assertAttributeNotSame(mixed` `$expected,` `string` `$actualAttributeName,` `object` `$actualObject,` `string` `$message)` | 如果 `$actualObject->actualAttributeName` 和 `$actual` 具有相同的类型和值，则报告由 `$message` 标识的错误。该 `$actualObject->actualAttributeName` 属性的可见性可能是公共的、受保护的或私有的。 |
| `void` `assertEquals(array` `$expected,` `array` `$actual)` | 如果两个数组 `$expected` 和 `$actual` 不相等，则报告一个错误。 |
| `void` `assertEquals(array` `$expected,` `array` `$actual,` `string` `$message)` | 报告一个错误，如果两个数组 `$expected` 和 `$actual` 不相等。 |
| `void` `assertNotEquals(array` `$expected,` `array` `$actual)` | 如果两个数组 `$expected` 和 `$actual` 相等，则报告一个错误。 |
| `void` `assertNotEquals(array` `$expected,` `array` `$actual,` `string` `$message)` | 报告一个错误，如果两个数组 `$expected` 和 `$actual` 相等，错误由 `$message` 标识。 |
| `void` `assertEquals(float` `$expected,` `float` `$actual,` `'',` `float` `$delta` `=` `0)` | 报告一个错误，如果两个浮点数 `$expected` 和 `$actual` 不在 `$delta` 范围内。 |
| `void` `assertEquals(float` `$expected,` `float` `$actual,` `string` `$message,` `float` `$delta` `=` `0)` | 报告一个错误，如果两个浮点数 `$expected` 和 `$actual` 不在 `$delta` 范围内，错误由 `$message` 标识。 |
| `void` `assertNotEquals(float` `$expected,` `float` `$actual,` `'',` `float` `$delta` `=` `0)` | 报告一个错误，如果两个浮点数 `$expected` 和 `$actual` 在 `$delta` 范围内。 |
| `void` `assertNotEquals(float` `$expected,` `float` `$actual,` `string` `$message,` `float` `$delta` `=` `0)` | 报告一个错误，如果两个浮点数 `$expected` 和 `$actual` 在 `$delta` 范围内，错误由 `$message` 标识。 |
| `void` `assertEquals(string` `$expected,` `string` `$actual)` | 报告一个错误，如果两个字符串 `$expected` 和 `$actual` 不相等。错误报告为两个字符串之间的差值。 |
| `void` `assertEquals(string` `$expected,` `string` `$actual,` `string` `$message)` | 报告一个错误，如果两个字符串 `$expected` 和 `$actual` 不相等，错误由 `$message` 标识。错误报告为两个字符串之间的差值。 |
| `void` `assertNotEquals(string` `$expected,` `string` `$actual)` | 报告一个错误，如果两个字符串 `$expected` 和 `$actual` 相等。 |
| `void` `assertNotEquals(string` `$expected,` `string` `$actual,` `string` `$message)` | 报告一个错误，如果两个字符串 `$expected` 和 `$actual` 相等，错误由 `$message` 标识。 |
| `void` `assertEquals(mixed` `$expected,` `mixed` `$actual)` | 报告一个错误，如果两个变量 `$expected` 和 `$actual` 不相等。 |
| `void` `assertEquals(mixed` `$expected,` `mixed` `$actual,` `string` `$message)` | 报告一个错误，如果两个变量 `$expected` 和 `$actual` 不相等，错误由 `$message` 标识。 |
| `void` `assertNotEquals(mixed` `$expected,` `mixed` `$actual)` | 报告一个错误，如果两个变量 `$expected` 和 `$actual` 相等。 |
| `void` `assertNotEquals(mixed` `$expected,` `mixed` `$actual,` `string` `$message)` | 报告一个错误，如果两个变量 `$expected` 和 `$actual` 相等，错误由 `$message` 标识。 |
| `void` `assertAttributeEquals(array` `$expected,` `string` `$actualAttributeName,` `object` `$actualObject)` | 报告一个错误，如果两个数组 `$expected` 和 `$actualObject->actualAttributeName` 不相等。`$actualObject->actualAttributeName` 属性的可见性可能是公共的、受保护的或私有的。 |
| `void` `assertAttributeEquals(array` `$expected,` `string` `$actualAttributeName,` `object` `$actualObject,` `string` `$message)` | 如果两个数组 `$expected` 和 `$actualObject->actualAttributeName` 不相等，则报告一个错误。`$actualObject->actualAttributeName` 属性的可见性可能是公共的、受保护的或私有的。 |
| `void` `assertAttributeNotEquals(array` `$expected,` `string` `$actualAttributeName,` `object` `$actualObject)` | 如果两个数组 `$expected` 和 `$actualObject->actualAttributeName` 相等，则报告一个错误。`$actualObject->actualAttributeName` 属性的可见性可能是公共的、受保护的或私有的。 |
| `void` `assertAttributeNotEquals(array` `$expected,` `string` `$actualAttributeName,` `object` `$actualObject,` `string` `$message)` | 如果两个数组 `$expected` 和 `$actualObject->actualAttributeName` 相等，则报告一个错误。`$actualObject->actualAttributeName` 属性的可见性可能是公共的、受保护的或私有的。 |
| `void` `assertAttributeEquals(float` `$expected,` `string` `$actualAttributeName,` `object` `$actualObject,` `'',` `float` `$delta` `=` `0)` | 如果两个浮点数 `$expected` 和 `$actualObject->actualAttributeName` 不在 `$delta` 范围内，则报告一个错误。`$actualObject->actualAttributeName` 属性的可见性可能是公共的、受保护的或私有的。 |
| `void` `assertAttributeEquals(float` `$expected,` `string` `$actualAttributeName,` `object` `$actualObject,` `string` `$message,` `float` `$delta` `=` `0)` | 报告一个错误，如果两个浮点数 `$expected` 和 `$actualObject->actualAttributeName` 不在 `$delta` 范围内。`$actualObject->actualAttributeName` 属性的可见性可能是公共的、受保护的或私有的。 |
| `void` `assertAttributeNotEquals(float` `$expected,` `string` `$actualAttributeName,` `object` `$actualObject,` `'',` `float` `$delta` `=` `0)` | 如果两个浮点数 `$expected` 和 `$actualObject->actualAttributeName` 在 `$delta` 范围内，则报告一个错误。`$actualObject->actualAttributeName` 属性的可见性可能是公共的、受保护的或私有的。 |
| `void` `assertAttributeNotEquals(float` `$expected,` `string` `$actualAttributeName,` `object` `$actualObject,` `string` `$message,` `float` `$delta` `=` `0)` | 如果两个浮点数 `$expected` 和 `$actualObject->actualAttributeName` 在 `$delta` 范围内，则报告一个错误。`$actualObject->actualAttributeName` 属性的可见性可能是公共的、受保护的或私有的。 |
| `void` `assertAttributeEquals(string` `$expected,` `string` `$actualAttributeName,` `object` `$actualObject)` | 如果两个字符串 `$expected` 和 `$actualObject->actualAttributeName` 不相等，则报告一个错误。错误报告为两个字符串之间的差异。`$actualObject->actualAttributeName` 属性的可见性可能是公共的、受保护的或私有的。 |
| `void` `assertAttributeEquals(string` `$expected,` `string` `$actualAttributeName,` `object` `$actualObject,` `string` `$message)` | 报告一个错误，如果两个字符串 `$expected` 和 `$actualObject->actualAttributeName` 不相等。错误报告为两个字符串之间的差异。`$actualObject->actualAttributeName` 属性的可见性可能是公共的、受保护的或私有的。 |
| `void` `assertAttributeNotEquals(string` `$expected,` `string` `$actualAttributeName,` `object` `$actualObject)` | 如果两个字符串 `$expected` 和 `$actualObject->actualAttributeName` 相等，则报告一个错误。`$actualObject->actualAttributeName` 属性的可见性可能是公共的、受保护的或私有的。 |
| `void` `assertAttributeNotEquals(string` `$expected,` `string` `$actualAttributeName,` `object` `$actualObject,` `string` `$message)` | 报告一个错误，如果两个字符串 `$expected` 和 `$actualObject->actualAttributeName` 相等。`$actualObject->actualAttributeName` 属性的可见性可能是公共的、受保护的或私有的。 |
| `void` `assertAttributeEquals(mixed` `$expected,` `string` `$actualAttributeName,` `object` `$actualObject)` | 如果两个变量 `$expected` 和 `$actualObject->actualAttributeName` 不相等，则报告一个错误。`$actualObject->actualAttributeName` 属性的可见性可能是公共的、受保护的或私有的。 |
| `void` `assertAttributeEquals(mixed` `$expected,` `string` `$actualAttributeName,` `object` `$actualObject,` `string` `$message)` | 报告一个错误，如果两个变量 `$expected` 和 `$actualObject->actualAttributeName` 不相等。`$actualObject->actualAttributeName` 属性的可见性可能是公共的、受保护的或私有的。 |
| `void` `assertAttributeNotEquals(mixed` `$expected,` `string` `$actualAttributeName,` `object` `$actualObject)` | 如果两个变量 `$expected` 和 `$actualObject->actualAttributeName` 相等，则报告一个错误。`$actualObject->actualAttributeName` 属性的可见性可能是公共的、受保护的或私有的。 |
| `void` `assertAttributeNotEquals(mixed` `$expected,` `string` `$actualAttributeName,` `object` `$actualObject,` `string` `$message)` | 报告一个错误，如果两个变量 `$expected` 和 `$actualObject->actualAttributeName` 相等。`$actualObject->actualAttributeName` 属性的可见性可能是公共的、受保护的或私有的。 |
| `void` `assertContains(mixed` `$needle,` `array` `$expected)` | 如果`$needle`不是`$expected`的元素，则报告一个错误。 |
| `void` `assertContains(mixed` `$needle,` `array` `$expected,` `string` `$message)` | 如果`$needle`不是`$expected`的元素，则报告一个由`$message`标识的错误。 |
| `void` `assertNotContains(mixed` `$needle,` `array` `$expected)` | 如果`$needle`是`$expected`的元素，则报告一个错误。 |
| `void` `assertNotContains(mixed` `$needle,` `array` `$expected,` `string` `$message)` | 报告一个错误，如果`$needle`是`$expected`数组的一个元素。 |
| `void` `assertContains(mixed` `$needle,` `Iterator` `$expected)` | 如果`$needle`不是`$expected`的元素，则报告一个错误。 |
| `void` `assertContains(mixed` `$needle,` `Iterator` `$expected,` `string` `$message)` | 如果`$needle`不是`$expected`的元素，则报告一个由`$message`标识的错误。 |
| `void` `assertNotContains(mixed` `$needle,` `Iterator` `$expected)` | 如果`$needle`是`$expected`的元素，则报告一个错误。 |
| `void` `assertNotContains(mixed` `$needle,` `Iterator` `$expected,` `string` `$message)` | 如果`$needle`是`$expected`的元素，则报告一个由`$message`标识的错误。 |
| `void` `assertContains(string` `$needle,` `string` `$expected)` | 如果`$needle`不是`$expected`的子串，则报告一个错误。 |
| `void` `assertContains(string` `$needle,` `string` `$expected,` `string` `$message)` | 如果`$needle`不是`$expected`的子串，则报告一个由`$message`标识的错误。 |
| `void` `assertNotContains(string` `$needle,` `string` `$expected)` | 如果`$needle`是`$expected`的子串，则报告一个错误。 |
| `void` `assertNotContains(string` `$needle,` `string` `$expected,` `string` `$message)` | 如果`$needle`是`$expected`的子串，则报告一个由`$message`标识的错误。 |
| `void` `assertAttributeContains(mixed` `$needle,` `string` `$actualAttributeName,` `object` `$actualObject)` | 如果`$needle`不是`$actualObject->actualAttributeName`的元素，该元素可以是数组、字符串或实现 Iterator 接口的对象，则报告一个错误。`$actualObject->actualAttributeName`属性的可见性可能是公共的、受保护的或私有的。 |
| `void` `assertAttributeContains(mixed` `$needle,` `string` `$actualAttributeName,` `object` `$actualObject,` `string` `$message)` | 报告一个由`$message`标识的错误，如果`$needle`不是`$actualObject->actualAttributeName`的元素，该元素可以是数组、字符串或实现 Iterator 接口的对象。`$actualObject->actualAttributeName`属性的可见性可能是公共的、受保护的或私有的。 |
| `void` `assertAttributeNotContains(mixed` `$needle,` `string` `$actualAttributeName,` `object` `$actualObject)` | 如果 `$needle` 是 `$actualObject->actualAttributeName` 的一个元素，该元素可以是数组、字符串或实现 Iterator 接口的对象，则报告一个错误。`$actualObject->actualAttributeName` 属性的可见性可能是公共的、受保护的或私有的。 |
| `void` `assertAttributeNotContains(mixed` `$needle,` `string` `$actualAttributeName,` `object` `$actualObject,` `string` `$message)` | 如果 `$needle` 是 `$actualObject->actualAttributeName` 的一个元素，该元素可以是数组、字符串或实现 Iterator 接口的对象，则报告一个错误，错误信息由 `$message` 指定。`$actualObject->actualAttributeName` 属性的可见性可能是公共的、受保护的或私有的。 |
| `void` `assertRegExp(string` `$pattern,` `string` `$string)` | 如果 `$string` 不匹配正则表达式 `$pattern`，则报告一个错误。 |
| `void` `assertRegExp(string` `$pattern,` `string` `$string,` `string` `$message)` | 如果 `$string` 不匹配正则表达式 `$pattern`，则报告一个错误，错误信息由 `$message` 指定。 |
| `void` `assertNotRegExp(string` `$pattern,` `string` `$string)` | 如果 `$string` 匹配正则表达式 `$pattern`，则报告一个错误。 |
| `void` `assertNotRegExp(string` `$pattern,` `string` `$string,` `string` `$message)` | 如果 `$string` 匹配正则表达式 `$pattern`，则报告一个错误，错误信息由 `$message` 指定。 |
| `void` `assertType(string` `$expected,` `mixed` `$actual)` | 如果变量 `$actual` 不是类型 `$expected`，则报告一个错误。 |
| `void` `assertType(string` `$expected,` `mixed` `$actual,` `string` `$message)` | 如果变量 `$actual` 不是类型 `$expected`，则报告一个错误，错误信息由 `$message` 指定。 |
| `void` `assertNotType(string` `$expected,` `mixed` `$actual)` | 如果变量 `$actual` 是类型 `$expected`，则报告一个错误。 |
| `void` `assertNotType(string` `$expected,` `mixed` `$actual,` `string` `$message)` | 如果变量 `$actual` 是类型 `$expected`，则报告一个错误，错误信息由 `$message` 指定。 |
| `void` `assertFileExists(string` `$filename)` | 如果指定的文件 `$filename` 不存在，则报告一个错误。 |
| `void` `assertFileExists(string` `$filename,` `string` `$message)` | 报告一个错误，如果指定的文件 `$filename` 不存在，错误信息由 `$message` 指定。 |
| `void` `assertFileNotExists(string` `$filename)` | 如果指定的文件 `$filename` 存在，则报告一个错误。 |
| `void` `assertFileNotExists(string` `$filename,` `string` `$message)` | 如果指定的文件 `$filename` 存在，则报告一个错误，错误信息由 `$message` 指定。 |
| `void` `assertObjectHasAttribute(string` `$attributeName,` `object` `$object)` | 如果 `$object->attributeName` 不存在，则报告一个错误。 |
| `void` `assertObjectHasAttribute(string` `$attributeName,` `object` `$object,` `string` `$message)` | 如果`$object->attributeName`不存在，则通过`$message`报告错误。 |
| `void` `assertObjectNotHasAttribute(string` `$attributeName,` `object` `$object)` | 如果`$object->attributeName`存在，则报告错误。 |
| `void` `assertObjectNotHasAttribute(string` `$attributeName,` `object` `$object,` `string` `$message)` | 如果`$object->attributeName`存在，则报告错误。 |

# 摘要

本章重点介绍 PHP 面向对象编程的两个非常重要的特性。一个是反射，它是所有主要编程语言（如 Java、Ruby 和 Python）的一部分。另一个是单元测试，它是良好、稳定和可管理应用程序设计的重要组成部分。我们关注了一个非常流行的包，它是 JUnit 在 PHP 中的移植，名为 PHPUnit。如果你遵循本章提供的指南，你将能够成功设计你的单元测试。

在下一章中，我们将学习 PHP 中的一些内置对象，这将使你的生活比平时更容易。我们将学习一个名为标准 PHP 库或 SPL 的庞大对象存储库。在此之前，通过编写自己的单元测试来享受调试的乐趣。
