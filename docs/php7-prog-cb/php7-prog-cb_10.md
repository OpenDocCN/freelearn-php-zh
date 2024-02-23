# 第十章。查看高级算法

在本章中，我们将涵盖：

+   使用 getter 和 setter

+   实现链表

+   构建冒泡排序

+   实现堆栈

+   构建二分搜索类

+   实现搜索引擎

+   显示多维数组并累积总数

# 介绍

在本章中，我们涵盖了实现各种高级算法的配方，例如链表、冒泡排序、堆栈和二分搜索。此外，我们还涵盖了 getter 和 setter，以及实现搜索引擎和显示来自多维数组的值并累积总数。

# 使用 getter 和 setter

乍一看，似乎定义具有`public`属性的类，然后可以直接读取或写入这些属性是有意义的。然而，最佳做法是将属性定义为`protected`，然后为每个属性定义**getter**和**setter**。顾名思义，*getter*用于检索属性的值。*setter*用于设置值。

### 提示

**最佳实践**

将属性定义为`protected`以防止意外*外部*访问。使用`public` get*和 set*方法来访问这些属性。通过这种方式，不仅可以更精确地控制访问，还可以在获取和设置它们时进行格式和数据类型的更改。

## 如何做...

1.  Getter 和 setter 在获取或设置值时提供了额外的灵活性。如果需要，您可以添加额外的逻辑层，这是直接读取或写入公共属性时无法实现的。您只需要创建一个以`get`或`set`为前缀的公共方法。属性的名称成为后缀。约定是将变量的第一个字母大写。因此，如果属性是`$testValue`，getter 将是`getTestValue()`。

1.  在此示例中，我们定义了一个具有受保护属性`$date`的类。请注意，`get`和`set`方法允许将其视为`DateTime`对象或字符串。无论如何，该值实际上都存储为`DateTime`实例。

```php
$a = new class() {
  protected $date;
  public function setDate($date)
  {
    if (is_string($date)) {
        $this->date = new DateTime($date);
    } else {
        $this->date = $date;
    }
  }
  public function getDate($asString = FALSE)
  {
    if ($asString) {
        return $this->date->format('Y-m-d H:i:s');
    } else {
        return $this->date;
    }
  }
};
```

1.  Getter 和 setter 允许您过滤或清理传入或传出的数据。在下面的示例中，有两个属性`$intVal`和`$arrVal`，它们被设置为默认初始值`NULL`。请注意，getter 的返回值不仅是数据类型化的，而且还提供了默认值。setter 也要么强制执行传入的数据类型，要么将传入的值强制转换为特定的数据类型：

```php
<?php
class GetSet
{
  protected $intVal = NULL;
  protected $arrVal = NULL;
  // note the use of the null coalesce operator to return a default value
  public function getIntVal() : int
  {
    return $this->intVal ?? 0;
  }
  public function getArrVal() : array
  {
    return $this->arrVal ?? array();
  }
  public function setIntVal($val)
  {
    $this->intVal = (int) $val ?? 0;
  }
  public function setArrVal(array $val)
  {
    $this->arrVal = $val ?? array();
  }
}
```

1.  如果一个类有很多属性，为每个属性定义一个明确的 getter 和 setter 可能会变得乏味。在这种情况下，可以使用魔术方法`__call()`定义一种*回退*。以下类定义了九个不同的属性。我们不必定义九个 getter 和九个 setter，而是定义一个名为`__call()`的方法，该方法确定使用是`get`还是`set`。如果是`get`，它会从内部数组中检索键。如果是`set`，它会将值存储在内部数组中。

### 注意

`__call()`方法是一个魔术方法，如果应用程序调用不存在的方法，则会执行该方法。

```php
<?php
class LotsProps
{
  protected $firstName  = NULL;
  protected $lastName   = NULL;
  protected $addr1      = NULL;
  protected $addr2      = NULL;
  protected $city       = NULL;
  protected $state      = NULL;
  protected $province   = NULL;
  protected $postalCode = NULL;
  protected $country    = NULL;
  protected $values     = array();

  public function __call($method, $params)
  {
    preg_match('/^(get|set)(.*?)$/i', $method, $matches);
    $prefix = $matches[1] ?? '';
    $key    = $matches[2] ?? '';
    $key    = strtolower($key);
    if ($prefix == 'get') {
        return $this->values[$key] ?? '---';
    } else {
        $this->values[$key] = $params[0];
    }
  }
}
```

## 工作原理...

将步骤 1 中提到的代码复制到一个新文件`chap_10_oop_using_getters_and_setters.php`中。要测试该类，请添加以下内容：

```php
// set date using a string
$a->setDate('2015-01-01');
var_dump($a->getDate());

// retrieves the DateTime instance
var_dump($a->getDate(TRUE));

// set date using a DateTime instance
$a->setDate(new DateTime('now'));
var_dump($a->getDate());

// retrieves the DateTime instance
var_dump($a->getDate(TRUE));
```

在下面的输出中，您可以看到`$date`属性可以使用`string`或实际的`DateTime`实例进行设置。当执行`getDate()`时，可以根据`$asString`标志的值返回`string`或`DateTime`实例：

![工作原理...](img/B05314_10_12.jpg)

接下来，看一下步骤 2 中定义的代码。将此代码复制到一个名为`chap_10_oop_using_getters_and_setters_defaults.php`的文件中，并添加以下内容：

```php
// create the instance
$a = new GetSet();

// set a "proper" value
$a->setIntVal(1234);
echo $a->getIntVal();
echo PHP_EOL;

// set a bogus value
$a->setIntVal('some bogus value');
echo $a->getIntVal();
echo PHP_EOL;

// NOTE: boolean TRUE == 1
$a->setIntVal(TRUE);
echo $a->getIntVal();
echo PHP_EOL;

// returns array() even though no value was set
var_dump($a->getArrVal());
echo PHP_EOL;

// sets a "proper" value
$a->setArrVal(['A','B','C']);
var_dump($a->getArrVal());
echo PHP_EOL;

try {
    $a->setArrVal('this is not an array');
    var_dump($a->getArrVal());
    echo PHP_EOL;
} catch (TypeError $e) {
    echo $e->getMessage();
}

echo PHP_EOL;
```

从以下输出中可以看出，设置*正确*的整数值会按预期工作。非数字值默认为`0`。有趣的是，如果您将布尔值`TRUE`作为`setIntVal()`的参数，它会被插入为`1`。

如果您在不设置值的情况下调用`getArrVal()`，默认值是一个空数组。设置数组值会按预期工作。但是，如果您将非数组值作为参数提供，数组的类型提示会导致抛出`TypeError`，可以像这样捕获：

![工作原理...](img/B05314_10_13.jpg)

最后，将步骤 3 中定义的`LotsProps`类放在一个单独的文件`chap_10_oop_using_getters_and_setters_magic_call.php`中。现在添加代码来设置值。当然，会调用魔术方法`__call()`。运行`preg_match()`后，不存在的属性的剩余部分，在`set`字母之后，将成为内部数组`$values`中的一个键：

```php
$a = new LotsProps();
$a->setFirstName('Li\'l Abner');
$a->setLastName('Yokum');
$a->setAddr1('1 Dirt Street');
$a->setCity('Dogpatch');
$a->setState('Kentucky');
$a->setPostalCode('12345');
$a->setCountry('USA');
?>
```

然后，您可以定义显示值的 HTML，使用相应的`get`方法。这些方法将返回内部数组的键：

```php
<div class="container">
<div class="left blue1">Name</div>
<div class="right yellow1">
<?= $a->getFirstName() . ' ' . $a->getLastName() ?></div>   
</div>
<div class="left blue2">Address</div>
<div class="right yellow2">
    <?= $a->getAddr1() ?>
    <br><?= $a->getAddr2() ?>
    <br><?= $a->getCity() ?>
    <br><?= $a->getState() ?>
    <br><?= $a->getProvince() ?>
    <br><?= $a->getPostalCode() ?>
    <br><?= $a->getCountry() ?>
</div>   
</div>
```

这是最终输出：

![工作原理...](img/B05314_10_14.jpg)

# 实现链表

链表是一个列表包含指向另一个列表键的列表。类似地，在数据库术语中，您可以有一个包含数据的表，以及一个指向数据的单独索引。一个索引可以按 ID 生成项目列表。另一个索引可能根据标题产生列表等等。链表的显着特点是您不必触及原始项目列表。

例如，在接下来显示的图表中，主列表包含 ID 号码和水果的名称。如果您直接输出主列表，水果名称将按照以下顺序显示：**苹果**，**葡萄**，**香蕉**，**橙子**，**樱桃**。另一方面，如果您使用链表作为索引，结果输出的水果名称将是**苹果**，**香蕉**，**樱桃**，**葡萄**和**橙子**。

![实现链表](img/B05314_10_01.jpg)

## 如何做...

1.  链表的主要用途之一是以不同的顺序显示项目。一种方法是创建键值对的迭代，其中键表示新顺序，值包含主列表中键的值。这样的函数可能如下所示：

```php
function buildLinkedList(array $primary,
                         callable $makeLink)
{
  $linked = new ArrayIterator();
  foreach ($primary as $key => $row) {
    $linked->offsetSet($makeLink($row), $key);
  }
  $linked->ksort();
  return $linked;
}
```

1.  我们使用匿名函数生成新的键，以提供额外的灵活性。您还会注意到我们按键排序(`ksort()`)，以便链表按键顺序迭代。

1.  我们只需要通过链表进行迭代，但是从主列表`$customer`中产生结果：

```php
foreach ($linked as $key => $link) {
  $output .= printRow($customer[$link]);
}
```

1.  请注意，我们绝对不会触及主列表。这使我们能够生成多个链表，每个链表代表不同的顺序，同时保留我们的原始数据集。

1.  链表的另一个重要用途是用于过滤。该技术与之前显示的类似。唯一的区别是我们扩展了`buildLinkedList()`函数，添加了一个过滤列和过滤值：

```php
function buildLinkedList(array $primary,
                         callable $makeLink,
                         $filterCol = NULL,
                         $filterVal = NULL)
{
  $linked = new ArrayIterator();
  $filterVal = trim($filterVal);
  foreach ($primary as $key => $row) {
    if ($filterCol) {
      if (trim($row[$filterCol]) == $filterVal) {
        $linked->offsetSet($makeLink($row), $key);
      }
    } else {
      $linked->offsetSet($makeLink($row), $key);
    }
  }
  $linked->ksort();
  return $linked;
}
```

1.  我们只在链表中包含与主列表中的`$filterCol`表示的值匹配的值。迭代逻辑与步骤 2 中显示的相同。

1.  最后，另一种形式的链表是*双向*链表。在这种情况下，列表构造成可以向前或向后进行迭代。在 PHP 的情况下，我们很幸运地拥有一个 SPL 类`SplDoublyLinkedList`，它可以很好地完成这项任务。以下是构建双向链表的函数：

```php
function buildDoublyLinkedList(ArrayIterator $linked)
{
  $double = new SplDoublyLinkedList();
  foreach ($linked as $key => $value) {
    $double->push($value);
  }
  return $double;
}
```

### 注意

`SplDoublyLinkedList`的术语可能会产生误导。`SplDoublyLinkedList::top()`实际上指向列表的*末尾*，而`SplDoublyLinkedList::bottom()`指向*开始*！

## 工作原理...

将第一个项目中显示的代码复制到一个文件`chap_10_linked_list_include.php`中。为了演示链表的使用，您需要一个数据源。在这个示例中，您可以使用先前配方中提到的`customer.csv`文件。它是一个 CSV 文件，包含以下列：

```php
"id","name","balance","email","password","status","security_question",
"confirm_code","profile_id","level"
```

您可以将以下函数添加到先前提到的包含文件中，以生成客户的主列表，并显示有关它们的信息。请注意，我们使用第一列 id 作为主键：

```php
function readCsv($fn, &$headers)
{
  if (!file_exists($fn)) {
    throw new Error('File Not Found');
  }
  $fileObj = new SplFileObject($fn, 'r');
  $result = array();
  $headers = array();
  $firstRow = TRUE;
  while ($row = $fileObj->fgetcsv()) {
    // store 1st row as headers
    if ($firstRow) {
      $firstRow = FALSE;
      $headers = $row;
    } else {
      if ($row && $row[0] !== NULL && $row[0] !== 0) {
        $result[$row[0]] = $row;
      }
    }
  }
  return $result;
}

function printHeaders($headers)
{
  return sprintf('%4s : %18s : %8s : %32s : %4s' . PHP_EOL,
                 ucfirst($headers[0]),
                 ucfirst($headers[1]),
                 ucfirst($headers[2]),
                 ucfirst($headers[3]),
                 ucfirst($headers[9]));
}

function printRow($row)
{
  return sprintf('%4d : %18s : %8.2f : %32s : %4s' . PHP_EOL,
                 $row[0], $row[1], $row[2], $row[3], $row[9]);
}

function printCustomer($headers, $linked, $customer)
{
  $output = '';
  $output .= printHeaders($headers);
  foreach ($linked as $key => $link) {
    $output .= printRow($customer[$link]);
  }
  return $output;
}
```

然后您可以定义一个调用程序`chap_10_linked_list_in_order.php`，其中包括先前定义的文件，并读取`customer.csv`：

```php
<?php
define('CUSTOMER_FILE', __DIR__ . '/../data/files/customer.csv');
include __DIR__ . '/chap_10_linked_list_include.php';
$headers = array();
$customer = readCsv(CUSTOMER_FILE, $headers);
```

然后您可以定义一个将在链表中生成一个键的匿名函数。在这个示例中，定义一个将第一列（名称）分解为名和姓的函数：

```php
$makeLink = function ($row) {
  list($first, $last) = explode(' ', $row[1]);
  return trim($last) . trim($first);
};
```

然后您可以调用函数来构建链表，并使用`printCustomer()`来显示结果：

```php
$linked = buildLinkedList($customer, $makeLink);
echo printCustomer($headers, $linked, $customer);
```

以下是输出可能会出现的方式：

![工作原理...](img/B05314_10_02.jpg)

要生成过滤结果，请修改`buildLinkedList()`，如步骤 4 中所述。然后可以添加逻辑来检查过滤列的值是否与过滤器中的值匹配：

```php
define('LEVEL_FILTER', 'INT');

$filterCol = 9;
$filterVal = LEVEL_FILTER;
$linked = buildLinkedList($customer, $makeLink, $filterCol, $filterVal);
```

## 还有更多...

PHP 7.1 引入了使用`[ ]`作为`list()`的替代方法。如果您查看先前提到的匿名函数，您可以在 PHP 7.1 中将其重写如下：

```php
$makeLink = function ($row) {
  [$first, $last] = explode(' ', $row[1]);
  return trim($last) . trim($first);
};
```

更多信息，请参见[`wiki.php.net/rfc/short_list_syntax`](https://wiki.php.net/rfc/short_list_syntax)。

# 构建冒泡排序

经典的**冒泡排序**经常分配给大学生练习。尽管如此，掌握这个算法很重要，因为有许多情况下内置的 PHP 排序函数不适用。一个例子是对多维数组进行排序，其中排序键不是第一列。

冒泡排序的工作方式是通过递归迭代列表并交换当前值和下一个值。如果要使项目按升序排列，则如果下一个项目小于当前项目，则进行交换。对于降序排序，如果相反情况为真，则进行交换。当不再发生交换时，排序结束。

在下图中，第一次通过后，**Grape**和**Banana**被交换，**Orange**和**Cherry**也被交换。第二次通过后，**Grape**和**Cherry**被交换。在最后一次通过中不再发生交换，冒泡排序结束：

![构建冒泡排序](img/B05314_10_03.jpg)

## 如何做...

1.  我们不想实际*移动*数组中的值；这在资源使用方面将是非常昂贵的。相反，我们将使用在前面的配方中讨论的**链表**。

1.  首先，我们使用在前面的配方中讨论的`buildLinkedList()`函数构建一个链表。

1.  然后我们定义一个新函数`bubbleSort()`，它接受引用的链表，主列表，排序字段和表示排序顺序（升序或降序）的参数：

```php
function bubbleSort(&$linked, $primary, $sortField, $order = 'A')
{
```

1.  所需的变量包括代表迭代次数的变量，交换次数的变量，以及基于链表的迭代器：

```php
  static $iterations = 0;
  $swaps = 0;
  $iterator = new ArrayIterator($linked);
```

1.  在`while()`循环中，只有在迭代仍然有效时才继续进行，也就是说仍在进行中。然后我们获取当前键和值，以及下一个键和值。请注意额外的`if()`语句以确保迭代仍然有效（也就是说，确保我们不会掉出列表的末尾！）：

```php
while ($iterator->valid()) {
  $currentLink = $iterator->current();
  $currentKey  = $iterator->key();
  if (!$iterator->valid()) break;
  $iterator->next();
  $nextLink = $iterator->current();
  $nextKey  = $iterator->key();
```

1.  接下来，我们检查排序是升序还是降序。根据方向，我们检查下一个值是大于还是小于当前值。比较的结果存储在`$expr`中：

```php
if ($order == 'A') {
    $expr = $primary[$linked->offsetGet
            ($currentKey)][$sortField] > 
            $primary[$linked->offsetGet($nextKey)][$sortField];
} else {
    $expr = $primary[$linked->offsetGet
            ($currentKey)][$sortField] < 
            $primary[$linked->offsetGet($nextKey)][$sortField];
}
```

1.  如果`$expr`的值为`TRUE`，并且我们有有效的当前键和下一个键，则交换链表中的值。我们还增加`$swaps`：

```php
if ($expr && $currentKey && $nextKey 
    && $linked->offsetExists($currentKey) 
    && $linked->offsetExists($nextKey)) {
    $tmp = $linked->offsetGet($currentKey);
    $linked->offsetSet($currentKey, 
    $linked->offsetGet($nextKey));
    $linked->offsetSet($nextKey, $tmp);
    $swaps++;
  }
}
```

1.  最后，如果发生了任何交换，我们需要再次运行迭代，直到没有更多的交换。因此，我们对同一个方法进行递归调用：

```php
if ($swaps) bubbleSort($linked, $primary, $sortField, $order);
```

1.  *真正*的返回值是重新组织的链表。我们还返回迭代的次数，仅供参考：

```php
  return ++$iterations;
}
```

## 工作原理...

将之前讨论的`bubbleSort()`函数添加到上一篇教程中创建的包含文件中。您可以使用前一篇教程中讨论的相同逻辑来读取`customer.csv`文件，生成一个主列表：

```php
<?php
define('CUSTOMER_FILE', __DIR__ . '/../data/files/customer.csv');
include __DIR__ . '/chap_10_linked_list_include.php';
$headers = array();
$customer = readCsv(CUSTOMER_FILE, $headers);
```

然后，您可以使用第一列作为排序键来生成一个链表：

```php
$makeLink = function ($row) {
  return $row[0];
};
$linked = buildLinkedList($customer, $makeLink);
```

最后，调用`bubbleSort()`函数，提供链表和客户列表作为参数。您还可以提供一个排序列，在本示例中是列 2，表示账户余额，使用字母'A'表示升序。`printCustomer()`函数可用于显示输出：

```php
echo 'Iterations: ' . bubbleSort($linked, $customer, 2, 'A') . PHP_EOL;
echo printCustomer($headers, $linked, $customer);
```

以下是输出的示例：

![工作原理...](img/B05314_10_04.jpg)

# 实现堆栈

**堆栈**是一种简单的算法，通常实现为**后进先出**（**LIFO**）。想象一下一堆书放在图书馆的桌子上。当图书管理员去把书放回原位时，首先处理的是最顶部的书，依此类推，直到堆栈底部的书被替换。最顶部的书是最后放在堆栈上的，因此后进先出。

在编程术语中，堆栈用于临时存储信息。检索顺序有助于首先检索最近的项目。

## 如何做...

1.  首先，我们定义一个`Application\Generic\Stack`类。核心逻辑封装在 SPL 类`SplStack`中：

```php
namespace Application\Generic;
use SplStack;
class Stack
{
  // code
}
```

1.  接下来，我们定义一个表示堆栈的属性，并设置一个`SplStack`实例：

```php
protected $stack;
public function __construct()
{
  $this->stack = new SplStack();
}
```

1.  然后我们定义了从堆栈中添加和删除的方法，经典的`push()`和`pop()`方法：

```php
public function push($message)
{
  $this->stack->push($message);
}
public function pop()
{
  return $this->stack->pop();
}
```

1.  我们还添加了一个`__invoke()`的实现，返回`stack`属性的实例。这允许我们在直接函数调用中使用对象：

```php
public function __invoke()
{
  return $this->stack;
}
```

## 工作原理...

堆栈的一个可能用途是存储消息。在消息的情况下，通常希望首先检索最新的消息，因此这是堆栈的一个完美用例。按照本教程中讨论的内容定义`Application\Generic\Stack`类。接下来，定义一个调用程序，设置自动加载并创建`stack`的实例：

```php
<?php
// setup class autoloading
require __DIR__ . '/../Application/Autoload/Loader.php';
Application\Autoload\Loader::init(__DIR__ . '/..');
use Application\Generic\Stack;
$stack = new Stack();
```

要对堆栈执行操作，存储一系列消息。由于您很可能会在应用程序的不同点存储消息，因此可以使用`sleep()`来模拟其他代码的运行：

```php
echo 'Do Something ... ' . PHP_EOL;
$stack->push('1st Message: ' . date('H:i:s'));
sleep(3);

echo 'Do Something Else ... ' . PHP_EOL;
$stack->push('2nd Message: ' . date('H:i:s'));
sleep(3);

echo 'Do Something Else Again ... ' . PHP_EOL;
$stack->push('3rd Message: ' . date('H:i:s'));
sleep(3);
```

最后，只需遍历堆栈以检索消息。请注意，您可以调用堆栈对象，就像它是一个函数一样，它返回`SplStack`实例：

```php
echo 'What Time Is It?' . PHP_EOL;
foreach ($stack() as $item) {
  echo $item . PHP_EOL;
}
```

以下是预期输出：

![工作原理...](img/B05314_10_05.jpg)

# 构建二进制搜索类

传统搜索通常按顺序遍历项目列表。这意味着要搜索的最大可能数量的项目可能与列表的长度相同！这并不是很有效。如果需要加快搜索速度，请考虑实现*二进制*搜索。

这种技术非常简单：找到列表中点，并确定搜索项是小于、等于还是大于中点项。如果小于，则将上限设置为中点，并仅搜索列表的前半部分。如果大于，则将下限设置为中点，并仅搜索列表的后半部分。然后继续将列表分成 1/4、1/8、1/16 等，直到找到搜索项（或没有找到）。

### 注意

需要注意的是，尽管比较的最大次数远远小于顺序搜索（*log n + 1*，其中*n*是列表中的元素数，*log*是二进制对数），但参与搜索的列表必须首先排序，这当然会降低性能。

## 如何做...

1.  我们首先构建一个搜索类`Application\Generic\Search`，它接受主列表作为参数。作为控制，我们还定义一个属性`$iterations`：

```php
namespace Application\Generic;
class Search
{
  protected $primary;
  protected $iterations;
  public function __construct($primary)
  {
    $this->primary = $primary;
  }
```

1.  接下来，我们定义一个方法`binarySearch()`，它设置了搜索基础设施。首要任务是构建一个单独的数组`$search`，其中键是搜索中包含的列的组合。然后我们按键排序：

```php
public function binarySearch(array $keys, $item)
{
  $search = array();
  foreach ($this->primary as $primaryKey => $data) {
    $searchKey = function ($keys, $data) {
      $key = '';
      foreach ($keys as $k) $key .= $data[$k];
```

```php
      return $key;
    };
    $search[$searchKey($keys, $data)] = $primaryKey;
  }
  ksort($search);
```

1.  然后我们将键提取到另一个数组`$binary`中，以便我们可以根据数字键执行二进制排序。然后我们调用`doBinarySearch()`，它会从我们的中间数组`$search`中得到一个键，或一个布尔值`FALSE`：

```php
  $binary = array_keys($search);
  $result = $this->doBinarySearch($binary, $item);
  return $this->primary[$search[$result]] ?? FALSE;
}
```

1.  首先，`doBinarySearch()`初始化一系列参数。`$iterations`，`$found`，`$loop`，`$done`和`$max`都用于防止无限循环。`$upper`和`$lower`表示要检查的列表切片：

```php
public function doBinarySearch($binary, $item)
{
  $iterations = 0;
  $found = FALSE;
  $loop  = TRUE;
  $done  = -1;
  $max   = count($binary);
  $lower = 0;
  $upper = $max - 1;
```

1.  然后我们实现一个`while()`循环并设置中点：

```php
  while ($loop && !$found) {
    $mid = (int) (($upper - $lower) / 2) + $lower;
```

1.  现在我们可以使用新的 PHP 7 **太空船操作符**，它在单个比较中给出小于、等于或大于。如果小于，则将上限设置为中点。如果大于，则将下限调整为中点。如果相等，则完成：

```php
switch ($item <=> $binary[$mid]) {
  // $item < $binary[$mid]
  case -1 :
  $upper = $mid;
  break;
  // $item == $binary[$mid]
  case 0 :
  $found = $binary[$mid];
  break;
  // $item > $binary[$mid]
  case 1 :
  default :
  $lower = $mid;
}
```

1.  现在是一点循环控制。我们增加迭代次数，并确保它不超过列表的大小。如果超过了，肯定有问题，我们需要退出。否则，我们检查上限和下限是否连续两次相同，如果是，则搜索项未找到。然后我们存储迭代次数并返回找到的内容（或未找到）：

```php
    $loop = (($iterations++ < $max) && ($done < 1));
    $done += ($upper == $lower) ? 1 : 0;
  }
  $this->iterations = $iterations;
  return $found;
}
```

## 它是如何工作的...

首先，实现`Application\Generic\Search`类，定义本教程中描述的方法。接下来，定义一个调用程序`chap_10_binary_search.php`，它设置自动加载并将`customer.csv`文件读取为搜索目标（如前一教程中所讨论的）：

```php
<?php
define('CUSTOMER_FILE', __DIR__ . '/../data/files/customer.csv');
include __DIR__ . '/chap_10_linked_list_include.php';
require __DIR__ . '/../Application/Autoload/Loader.php';
Application\Autoload\Loader::init(__DIR__ . '/..');
use Application\Generic\Search;
$headers = array();
$customer = readCsv(CUSTOMER_FILE, $headers);
```

然后，您可以创建一个新的`Search`实例，并在列表的中间某个位置指定一个项目。在本示例中，搜索基于列 1，即客户名称，项目为`Todd Lindsey`：

```php
$search = new Search($customer);
$item = 'Todd Lindsey';
$cols = [1];
echo "Searching For: $item\n";
var_dump($search->binarySearch($cols, $item));
```

为了说明，在`Application\Generic\Search::doBinarySearch()`中的`switch()`之前添加此行：

```php
echo 'Upper:Mid:Lower:<=> | ' . $upper . ':' . $mid . ':' . $lower . ':' . ($item <=> $binary[$mid]);
```

输出如下所示。请注意，上限、中间和下限会调整，直到找到项目：

![它是如何工作的...](img/B05314_10_06.jpg)

## 另请参阅

有关二进制搜索的更多信息，请参阅维基百科上的一篇优秀文章，其中介绍了基本数学内容[`en.wikipedia.org/wiki/Binary_search_algorithm`](https://en.wikipedia.org/wiki/Binary_search_algorithm)。

# 实施搜索引擎

为了实现搜索引擎，我们需要为多个列的搜索做好准备。此外，重要的是要认识到搜索项可能在字段的中间找到，并且用户很少提供足够的信息进行精确匹配。因此，我们将大量依赖 SQL 的`LIKE %value%`子句。

## 如何做...

1.  首先，我们定义一个基本类来保存搜索条件。该对象包含三个属性：键，最终表示数据库列；运算符（`LIKE`，`<`，`>`等）；和可选的项目。项目是可选的原因是一些运算符，如`IS NOT NULL`，不需要特定的数据：

```php
namespace Application\Database\Search;
class Criteria
{
  public $key;
  public $item;
  public $operator;
  public function __construct($key, $operator, $item = NULL)
  {
    $this->key  = $key;
    $this->operator = $operator;
    $this->item = $item;
  }
}
```

1.  接下来，我们需要定义一个类`Application\Database\Search\Engine`，并提供必要的类常量和属性。`$columns`和`$mapping`之间的区别在于`$columns`保存的信息最终将出现在 HTML 的`SELECT`字段（或等效字段）中。出于安全原因，我们不希望公开数据库列的实际名称，因此需要另一个数组`$mapping`：

```php
namespace Application\Database\Search;
use PDO;
use Application\Database\Connection;
class Engine
{
  const ERROR_PREPARE = 'ERROR: unable to prepare statement';
  const ERROR_EXECUTE = 'ERROR: unable to execute statement';
  const ERROR_COLUMN  = 'ERROR: column name not on list';
  const ERROR_OPERATOR= 'ERROR: operator not on list';
  const ERROR_INVALID = 'ERROR: invalid search criteria';

  protected $connection;
  protected $table;
  protected $columns;
  protected $mapping;
  protected $statement;
  protected $sql = '';
```

1.  接下来，我们定义一组我们愿意支持的运算符。键表示实际的 SQL。值是表单中将出现的内容：

```php
  protected $operators = [
      'LIKE'     => 'Equals',
      '<'        => 'Less Than',
      '>'        => 'Greater Than',
      '<>'       => 'Not Equals',
      'NOT NULL' => 'Exists',
  ];
```

1.  构造函数接受数据库连接实例作为参数。为了我们的目的，我们将使用第五章中定义的`Application\Database\Connection`。我们还需要提供数据库表的名称，以及`$columns`，一个任意列键和标签的数组，这些将出现在 HTML 表单中。这将引用`$mapping`，其中键与`$columns`匹配，但值表示实际的数据库列名：

```php
public function __construct(Connection $connection, 
                            $table, array $columns, array $mapping)
{
  $this->connection  = $connection;
  $this->setTable($table);
  $this->setColumns($columns);
  $this->setMapping($mapping);
}
```

1.  在构造函数之后，我们提供一系列有用的 getter 和 setter：

```php
public function setColumns($columns)
{
  $this->columns = $columns;
}
public function getColumns()
{
  return $this->columns;
}
// etc.
```

1.  可能最关键的方法是构建要准备的 SQL 语句。在初始`SELECT`设置之后，我们添加一个`WHERE`子句，使用`$mapping`添加实际的数据库列名。然后添加操作符并实现`switch()`，根据操作符，可能会或可能不会添加一个表示搜索项的命名占位符：

```php
public function prepareStatement(Criteria $criteria)
{
  $this->sql = 'SELECT * FROM ' . $this->table . ' WHERE ';
  $this->sql .= $this->mapping[$criteria->key] . ' ';
  switch ($criteria->operator) {
    case 'NOT NULL' :
      $this->sql .= ' IS NOT NULL OR ';
      break;
    default :
      $this->sql .= $criteria->operator . ' :' 
      . $this->mapping[$criteria->key] . ' OR ';
  }
```

1.  现在核心的`SELECT`已经定义，我们删除任何尾随的`OR`关键字，并添加一个导致结果根据搜索列排序的子句。然后将该语句发送到数据库进行准备：

```php
  $this->sql = substr($this->sql, 0, -4)
    . ' ORDER BY ' . $this->mapping[$criteria->key];
  $statement = $this->connection->pdo->prepare($this->sql);
  return $statement;
}
```

1.  现在我们准备转向主要的展示，`search()`方法。我们接受一个`Application\Database\Search\Criteria`对象作为参数。这确保我们至少有一个项目键和操作符。为了安全起见，我们添加了一个`if()`语句来检查这些属性：

```php
public function search(Criteria $criteria)
{
  if (empty($criteria->key) || empty($criteria->operator)) {
    yield ['error' => self::ERROR_INVALID];
    return FALSE;
  }
```

1.  然后我们调用`prepareStatement()`使用`try` / `catch`来捕获错误：

```php
try {
    if (!$statement = $this->prepareStatement($criteria)) {
      yield ['error' => self::ERROR_PREPARE];
      return FALSE;
}
```

1.  接下来，我们构建一个将提供给`execute()`的参数数组。键表示在准备语句中用作占位符的数据库列名。请注意，我们使用`=`而不是`LIKE %value%构造`：

```php
$params = array();
switch ($criteria->operator) {
  case 'NOT NULL' :
    // do nothing: already in statement
    break;
    case 'LIKE' :
    $params[$this->mapping[$criteria->key]] = 
    '%' . $criteria->item . '%';
    break;
    default :
    $params[$this->mapping[$criteria->key]] = 
    $criteria->item;
}
```

1.  该语句被执行，并使用`yield`关键字返回结果，这有效地将此方法转换为生成器：

```php
    $statement->execute($params);
    while ($row = $statement->fetch(PDO::FETCH_ASSOC)) {
      yield $row;
    }
  } catch (Throwable $e) {
    error_log(__METHOD__ . ':' . $e->getMessage());
    throw new Exception(self::ERROR_EXECUTE);
  }
  return TRUE;
}
```

## 工作原理...

将本章中讨论的代码放在`Application\Database\Search`下的文件`Criteria.php`和`Engine.php`中。然后可以定义一个调用脚本`chap_10_search_engine.php`，设置自动加载。您可以利用第五章中讨论的`Application\Database\Connection`类，*与数据库交互*，以及第六章中涵盖的表单元素类，*构建可扩展的网站*：

```php
<?php
define('DB_CONFIG_FILE', '/../config/db.config.php');
require __DIR__ . '/../Application/Autoload/Loader.php';
Application\Autoload\Loader::init(__DIR__ . '/..');

use Application\Database\Connection;
use Application\Database\Search\ { Engine, Criteria };
use Application\Form\Generic;
use Application\Form\Element\Select;
```

现在您可以定义哪些数据库列将出现在表单中，以及匹配的映射文件：

```php
$dbCols = [
  'cname' => 'Customer Name',
  'cbal' => 'Account Balance',
  'cmail' => 'Email Address',
  'clevel' => 'Level'
];

$mapping = [
  'cname' => 'name',
  'cbal' => 'balance',
  'cmail' => 'email',
  'clevel' => 'level'
];
```

您现在可以设置数据库连接并创建搜索引擎实例：

```php
$conn = new Connection(include __DIR__ . DB_CONFIG_FILE);
$engine = new Engine($conn, 'customer', $dbCols, $mapping);
```

为了显示适当的下拉`SELECT`元素，我们基于`Application\Form\*`类定义包装器和元素：

```php
$wrappers = [
  Generic::INPUT => ['type' => 'td', 'class' => 'content'],
  Generic::LABEL => ['type' => 'th', 'class' => 'label'],
  Generic::ERRORS => ['type' => 'td', 'class' => 'error']
];

// define elements
$fieldElement = new Select('field',
                Generic::TYPE_SELECT,
                'Field',
                $wrappers,
                ['id' => 'field']);
                $opsElement = new Select('ops',
                Generic::TYPE_SELECT,
                'Operators',
                $wrappers,
                ['id' => 'ops']);
                $itemElement = new Generic('item',
                Generic::TYPE_TEXT,
                'Searching For ...',
                $wrappers,
                ['id' => 'item','title' => 'If more than one item, separate with commas']);
                $submitElement = new Generic('submit',
                Generic::TYPE_SUBMIT,
                'Search',
                $wrappers,
                ['id' => 'submit','title' => 'Click to Search', 'value' => 'Search']);
```

然后我们获取输入参数（如果已定义），设置表单元素选项，创建搜索条件并运行搜索：

```php
$key  = (isset($_GET['field'])) 
? strip_tags($_GET['field']) : NULL;
$op   = (isset($_GET['ops'])) ? $_GET['ops'] : NULL;
$item = (isset($_GET['item'])) ? strip_tags($_GET['item']) : NULL;
$fieldElement->setOptions($dbCols, $key);
$itemElement->setSingleAttribute('value', $item);
$opsElement->setOptions($engine->getOperators(), $op);
$criteria = new Criteria($key, $op, $item);
$results = $engine->search($criteria);
?>
```

显示逻辑主要是朝向呈现表单。更详细的介绍在第六章中讨论，*构建可扩展的网站*，但我们在这里展示核心逻辑：

```php
  <form name="search" method="get">
  <table class="display" cellspacing="0" width="100%">
    <tr><?= $fieldElement->render(); ?></tr>
    <tr><?= $opsElement->render(); ?></tr>
    <tr><?= $itemElement->render(); ?></tr>
    <tr><?= $submitElement->render(); ?></tr>
    <tr>
    <th class="label">Results</th>
      <td class="content" colspan=2>
      <span style="font-size: 10pt;font-family:monospace;">
      <table>
      <?php foreach ($results as $row) : ?>
        <tr>
          <td><?= $row['id'] ?></td>
          <td><?= $row['name'] ?></td>
          <td><?= $row['balance'] ?></td>
          <td><?= $row['email'] ?></td>
          <td><?= $row['level'] ?></td>
        </tr>
      <?php endforeach; ?>
      </table>
      </span>
      </td>
    </tr>
  </table>
  </form>
```

这是浏览器中的示例输出：

![工作原理...](img/B05314_10_07.jpg)

# 显示多维数组和累积总数

如何正确显示多维数组中的数据一直是任何 Web 开发人员的经典问题。举例来说，假设您希望显示客户及其购买的列表。对于每个客户，您希望显示他们的姓名，电话号码，账户余额等。这已经代表了一个二维数组，其中*x*轴代表客户，*y*轴代表该客户的数据。现在加入购买，您就有了第三个轴！如何在二维屏幕上表示 3D 模型？一个可能的解决方案是使用简单的 JavaScript 可见性切换来结合“隐藏”的分区标签。

## 如何做...

1.  首先，我们需要从使用多个`JOIN`子句的 SQL 语句中生成一个 3D 数组。我们将使用在第一章中介绍的`Application/Database/Connection`类，*建立基础*，来制定一个适当的 SQL 查询。我们留下两个参数`min`和`max`，以支持分页。不幸的是，在这种情况下，我们不能简单地使用`LIMIT`和`OFFSET`，因为行数将取决于任何给定顾客的购买数量。因此，我们可以通过对顾客 ID 的限制来限制行数，假设（希望）是递增的。为了使这个功能正常工作，我们还需要将主要的`ORDER`设置为顾客 ID：

```php
define('ITEMS_PER_PAGE', 6);
define('SUBROWS_PER_PAGE', 6);
define('DB_CONFIG_FILE', '/../config/db.config.php');
include __DIR__ . '/../Application/Database/Connection.php';
use Application\Database\Connection;
$conn = new Connection(include __DIR__ . DB_CONFIG_FILE);
$sql  = 'SELECT c.id,c.name,c.balance,c.email,f.phone, '
  . 'u.transaction,u.date,u.quantity,u.sale_price,r.title '
  . 'FROM customer AS c '
  . 'JOIN profile AS f '
  . 'ON f.id = c.id '
  . 'JOIN purchases AS u '
  . 'ON u.customer_id = c.id '
  . 'JOIN products AS r '
  . 'ON u.product_id = r.id '
  . 'WHERE c.id >= :min AND c.id < :max '
  . 'ORDER BY c.id ASC, u.date DESC ';
```

1.  接下来我们可以实现基于顾客 ID 的分页形式，使用简单的`$_GET`参数进行限制。请注意，我们添加了额外的检查，以确保`$prev`的值不会低于零。您可能考虑添加另一个控件，以确保`$next`的值不会超出最后一个顾客 ID。在这个例子中，我们只允许它递增：

```php
$page = $_GET['page'] ?? 1;
$page = (int) $page;
$next = $page + 1;
$prev = $page - 1;
$prev = ($prev >= 0) ? $prev : 0;
```

1.  然后我们计算`$min`和`$max`的值，并准备并执行 SQL 语句：

```php
$min  = $prev * ITEMS_PER_PAGE;
$max  = $page * ITEMS_PER_PAGE;
$stmt = $conn->pdo->prepare($sql);
$stmt->execute(['min' => $min, 'max' => $max]);
```

1.  使用`while()`循环可以用来获取结果。我们在这个例子中使用了`PDO::FETCH_ASSOC`的简单获取模式。我们使用顾客 ID 作为键，将基本顾客信息存储为数组参数。然后我们在一个子数组中存储一组购买信息，`$results[$key]['purchases'][]`。当顾客 ID 改变时，这是一个信号，表示要为下一个顾客存储相同的信息。请注意，我们在一个数组键 total 中累积每个顾客的总数：

```php
$custId = 0;
$result = array();
$grandTotal = 0.0;
while ($row = $stmt->fetch(PDO::FETCH_ASSOC)) {
  if ($row['id'] != $custId) {
    $custId = $row['id'];
    $result[$custId] = [
      'name'    => $row['name'],
      'balance' => $row['balance'],
      'email'   => $row['email'],
      'phone'   => $row['phone'],
    ];
    $result[$custId]['total'] = 0;
  }
  $result[$custId]['purchases'][] = [
    'transaction' => $row['transaction'],
    'date'        => $row['date'],
    'quantity'    => $row['quantity'],
    'sale_price'  => $row['sale_price'],
    'title'       => $row['title'],
  ];
  $result[$custId]['total'] += $row['sale_price'];
  $grandTotal += $row['sale_price'];
}
?>
```

1.  接下来我们实现视图逻辑。首先，我们从显示主要顾客信息的块开始：

```php
<div class="container">
<?php foreach ($result as $key => $data) : ?>
<div class="mainLeft color0">
    <?= $data['name'] ?> [<?= $key ?>]
</div>
<div class="mainRight">
  <div class="row">
    <div class="left">Balance</div>
           <div class="right"><?= $data['balance']; ?></div>
  </div>
  <div class="row">
    <div class="left color2">Email</div>
           <div class="right"><?= $data['email']; ?></div>
  </div>
  <div class="row">
    <div class="left">Phone</div>
           <div class="right"><?= $data['phone']; ?></div>
    </div>
  <div class="row">
        <div class="left color2">Total Purchases</div>
    <div class="right">
<?= number_format($data['total'],2); ?>
</div>
  </div>
```

1.  接下来是显示该顾客的购买列表的逻辑：

```php
<!-- Purchases Info -->
<table>
  <tr>
  <th>Transaction</th><th>Date</th><th>Qty</th>
   <th>Price</th><th>Product</th>
  </tr>
  <?php $count  = 0; ?>
  <?php foreach ($data['purchases'] as $purchase) : ?>
  <?php $class = ($count++ & 01) ? 'color1' : 'color2'; ?>
  <tr>
  <td class="<?= $class ?>"><?= $purchase['transaction'] ?></td>
  <td class="<?= $class ?>"><?= $purchase['date'] ?></td>
  <td class="<?= $class ?>"><?= $purchase['quantity'] ?></td>
  <td class="<?= $class ?>"><?= $purchase['sale_price'] ?></td>
  <td class="<?= $class ?>"><?= $purchase['title'] ?></td>
  </tr>
  <?php endforeach; ?>
</table>
```

1.  为了分页的目的，我们添加按钮来表示*上一个*和*下一个*：

```php
<?php endforeach; ?>
<div class="container">
  <a href="?page=<?= $prev ?>">
        <input type="button" value="Previous"></a>
  <a href="?page=<?= $next ?>">
        <input type="button" value="Next" class="buttonRight"></a>
</div>
<div class="clearRow"></div>
</div>
```

1.  到目前为止，结果非常不整洁！因此，我们添加了一个简单的 JavaScript 函数，根据其`id`属性切换`<div>`标签的可见性：

```php
<script type="text/javascript">
function showOrHide(id) {
  var div = document.getElementById(id);
  div.style.display = div.style.display == "none" ? "block" : "none";
}
</script>
```

1.  接下来我们将购买表格包裹在最初不可见的`<div>`标签中。然后，我们可以设置初始可见的子行数的限制，并添加一个*显示*剩余购买数据的链接：

```php
<div class="row" id="<?= 'purchase' . $key ?>" style="display:none;">
  <table>
    <tr>
      <th>Transaction</th><th>Date</th><th>Qty</th>
                 <th>Price</th><th>Product</th>
    </tr>
  <?php $count  = 0; ?>
  <?php $first  = TRUE; ?>
  <?php foreach ($data['purchases'] as $purchase) : ?>
    <?php if ($count > SUBROWS_PER_PAGE && $first) : ?>
    <?php     $first = FALSE; ?>
    <?php     $subId = 'subrow' . $key; ?>
    </table>
    <a href="#" onClick="showOrHide('<?= $subId ?>')">More</a>
    <div id="<?= $subId ?>" style="display:none;">
    <table>
    <?php endif; ?>
  <?php $class = ($count++ & 01) ? 'color1' : 'color2'; ?>
  <tr>
  <td class="<?= $class ?>"><?= $purchase['transaction'] ?></td>
  <td class="<?= $class ?>"><?= $purchase['date'] ?></td>
  <td class="<?= $class ?>"><?= $purchase['quantity'] ?></td>
  <td class="<?= $class ?>"><?= $purchase['sale_price'] ?></td>
  <td class="<?= $class ?>"><?= $purchase['title'] ?></td>
  </tr>
  <?php endforeach; ?>
  </table>
  <?php if (!$first) : ?></div><?php endif; ?>
</div>
```

1.  然后我们添加一个按钮，当点击时，会显示隐藏的`<div>`标签：

```php
<input type="button" value="Purchases" class="buttonRight" 
    onClick="showOrHide('<?= 'purchase' . $key ?>')">
```

## 它是如何工作的...

将步骤 1 到 5 中描述的代码放入一个文件`chap_10_html_table_multi_array_hidden.php`中。

在`while()`循环的内部，添加以下内容：

```php
printf('%6s : %20s : %8s : %20s' . PHP_EOL, 
    $row['id'], $row['name'], $row['transaction'], $row['title']);
```

在`while()`循环之后，添加一个`exit`命令。以下是输出：

![它是如何工作的...](img/B05314_10_08.jpg)

您会注意到基本的顾客信息，如 ID 和姓名，会重复出现在每个结果行中，但购买信息，如交易和产品标题，会有所不同。继续并删除`printf()`语句。

用以下内容替换`exit`命令：

```php
echo '<pre>', var_dump($result), '</pre>'; exit;
```

新组成的 3D 数组如下所示：

![它是如何工作的...](img/B05314_10_09.jpg)

您现在可以添加步骤 5 到 7 中显示逻辑。虽然您现在显示了所有数据，但视觉显示并不有用。现在继续添加剩下步骤中提到的改进。这是初始输出可能会出现的样子：

![它是如何工作的...](img/B05314_10_15.jpg)

当点击**购买**按钮时，初始购买信息会出现。如果点击**更多**的链接，剩余的购买信息会显示：

![它是如何工作的...](img/B05314_10_11.jpg)
