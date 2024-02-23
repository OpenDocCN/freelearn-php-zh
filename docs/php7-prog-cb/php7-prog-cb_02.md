# 第二章。使用 PHP 7 高性能特性

在本章中，我们将讨论并了解 PHP 5 和 PHP 7 之间的语法差异，包括以下内容：

+   理解抽象语法树

+   理解解析中的差异

+   理解`foreach()`处理中的差异

+   使用 PHP 7 增强功能提高性能

+   遍历大型文件

+   将电子表格上传到数据库

+   递归目录迭代器

# 介绍

在本章中，我们将直接进入 PHP 7，介绍利用新的高性能特性的配方。然而，我们将首先介绍一系列较小的配方，以说明 PHP 7 处理参数解析、语法、`foreach()`循环和其他增强功能的差异。在深入探讨本章内容之前，让我们讨论一些 PHP 5 和 PHP 7 之间的基本差异。

PHP 7 引入了一个新的层，称为**抽象语法树**（**AST**），它有效地将解析过程与伪编译过程分离。尽管新层对性能几乎没有影响，但它赋予了语言一种新的语法统一性，这在以前是不可能的。

AST 的另一个好处是*取消引用*的过程。取消引用简单地指的是立即从对象中获取属性或运行方法，立即访问数组元素，并立即执行回调的能力。在 PHP 5 中，这种支持是不一致和不完整的。例如，要执行回调，通常需要先将回调或匿名函数赋值给一个变量，然后执行它。在 PHP 7 中，你可以立即执行它。

# 理解抽象语法树

作为开发人员，你可能会对摆脱 PHP 5 及更早版本中施加的某些语法限制感兴趣。除了之前提到的语法的统一性外，你将看到语法最大的改进是能够调用任何返回值，只需在后面添加一组额外的括号。此外，当返回值是数组时，你将能够直接访问任何数组元素。

## 如何做...

1.  任何返回回调的函数或方法都可以通过简单地添加括号`()`（带或不带参数）立即执行。任何返回数组的函数或方法都可以通过使用方括号`[]`指示元素来立即取消引用。在下面显示的简短（但琐碎）示例中，函数`test()`返回一个数组。数组包含六个匿名函数。`$a`的值为`$t`。`$$a`被解释为`$test`：

```php
    function test()
    {
        return [
            1 => function () { return [
                1 => function ($a) { return 'Level 1/1:' . ++$a; },
                2 => function ($a) { return 'Level 1/2:' . ++$a; },
            ];},
            2 => function () { return [
                1 => function ($a) { return 'Level 2/1:' . ++$a; },
                2 => function ($a) { return 'Level 2/2:' . ++$a; },
            ];}
        ];
    }

    $a = 't';
    $t = 'test';
    echo $$a()[1]()2;
    ```

1.  AST 允许我们发出`echo $$a()[1]()2`命令。这是从左到右解析的，执行如下：

+   `$$a()`被解释为`test()`，返回一个数组

+   `[1]`取消引用数组元素`1`，返回一个回调

+   `()`执行此回调，返回一个包含两个元素的数组

+   `[2]`取消引用数组元素`2`，返回一个回调

+   `(100)`执行此回调，提供值`100`，返回`Level 1/2:101`

### 提示

在 PHP 5 中不可能有这样的语句：会返回解析错误。

1.  以下是一个更加实质性的例子，利用 AST 语法来定义数据过滤和验证类。首先，我们定义`Application\Web\Securityclass`。在构造函数中，我们构建并定义了两个数组。第一个数组由过滤回调组成。第二个数组有验证回调：

```php
    public function __construct()
      {
        $this->filter = [
          'striptags' => function ($a) { return strip_tags($a); },
          'digits'    => function ($a) { return preg_replace(
          '/[⁰-9]/', '', $a); },
          'alpha'     => function ($a) { return preg_replace(
          '/[^A-Z]/i', '', $a); }
        ];
        $this->validate = [
          'alnum'  => function ($a) { return ctype_alnum($a); },
          'digits' => function ($a) { return ctype_digit($a); },
          'alpha'  => function ($a) { return ctype_alpha($a); }
        ];
      }
    ```

1.  我们希望能以*开发人员友好*的方式调用此功能。因此，如果我们想要过滤数字，那么运行这样的命令将是理想的：

```php
    $security->filterDigits($item));
    ```

1.  为了实现这一点，我们定义了魔术方法`__call()`，它使我们能够访问不存在的方法：

```php
    public function __call($method, $params)
    {

      preg_match('/^(filter|validate)(.*?)$/i', $method, $matches);
      $prefix   = $matches[1] ?? '';
      $function = strtolower($matches[2] ?? '');
      if ($prefix && $function) {
        return $this->$prefix$function;
      }
      return $value;
    }
    ```

我们使用`preg_match()`来匹配`$method`参数与`filter`或`validate`。然后，第二个子匹配将被转换为`$this->filter`或`$this->validate`中的数组键。如果两个子模式都产生子匹配，我们将第一个子匹配分配给`$prefix`，将第二个子匹配分配给`$function`。这些最终成为执行适当回调时的变量参数。

### 提示

**不要对这些东西太疯狂！**

当您沉浸在 AST 所带来的新的表达自由中时，请务必记住，您最终编写的代码可能会变得极其晦涩。这最终将导致长期的维护问题。

## 它是如何工作的...

首先，我们创建一个示例文件，`chap_02_web_filtering_ast_example.php`，以利用第一章中定义的自动加载类，*构建基础*，以获得`Application\Web\Security`的实例：

```php
require __DIR__ . '/../Application/Autoload/Loader.php';
Application\Autoload\Loader::init(__DIR__ . '/..');
$security = new Application\Web\Security();
```

接下来，我们定义一个测试数据块：

```php
$data = [
    '<ul><li>Lots</li><li>of</li><li>Tags</li></ul>',
    12345,
    'This is a string',
    'String with number 12345',
];
```

最后，我们为每个测试数据项调用每个过滤器和验证器：

```php
foreach ($data as $item) {
  echo 'ORIGINAL: ' . $item . PHP_EOL;
  echo 'FILTERING' . PHP_EOL;
  printf('%12s : %s' . PHP_EOL,'Strip Tags', $security->filterStripTags($item));
  printf('%12s : %s' . PHP_EOL, 'Digits', $security->filterDigits($item));
  printf('%12s : %s' . PHP_EOL, 'Alpha', $security->filterAlpha($item));

  echo 'VALIDATORS' . PHP_EOL;
  printf('%12s : %s' . PHP_EOL, 'Alnum',  
  ($security->validateAlnum($item))  ? 'T' : 'F');
  printf('%12s : %s' . PHP_EOL, 'Digits', 
  ($security->validateDigits($item)) ? 'T' : 'F');
  printf('%12s : %s' . PHP_EOL, 'Alpha',  
  ($security->validateAlpha($item))  ? 'T' : 'F');
}
```

以下是一些输入字符串的输出：

![它是如何工作的...](img/B05314_02_01.jpg)

## 另请参阅

有关 AST 的更多信息，请参阅涉及**抽象语法树**的 RFC，可以在[`wiki.php.net/rfc/abstract_syntax_tree`](https://wiki.php.net/rfc/abstract_syntax_tree)上查看。

# 了解解析的差异

在 PHP 5 中，赋值操作的右侧表达式是从*右到左*解析的。在 PHP 7 中，解析是一致的*从左到右*。

## 如何做...

1.  变量变量是间接引用值的一种方式。在下面的例子中，首先`$$foo`被解释为`${$bar}`。因此最终的返回值是`$bar`的值，而不是`$foo`的直接值（应该是`bar`）：

```php
    $foo = 'bar';
    $bar = 'baz';
    echo $$foo; // returns  'baz'; 
    ```

1.  在下一个例子中，我们有一个变量变量`$$foo`，它引用一个具有`bar 键`和`baz 子键`的多维数组：

```php
    $foo = 'bar';
    $bar = ['bar' => ['baz' => 'bat']];
    // returns 'bat'
    echo $$foo['bar']['baz'];
    ```

1.  在 PHP 5 中，解析是从右到左进行的，这意味着 PHP 引擎将寻找一个`$foo 数组`，其中包含一个`bar 键`和一个`baz 子键`。然后，元素的返回值将被解释以获得最终值`${$foo['bar']['baz']}`。

1.  然而，在 PHP 7 中，解析是一致的，从左到右进行，这意味着首先解释`($$foo)['bar']['baz']`。

1.  在下一个示例中，您可以看到在 PHP 5 中`$foo->$bar['bada']`的解释与 PHP 7 相比有很大不同。在下面的例子中，PHP 5 首先会解释`$bar['bada']`，并将此返回值与`$foo 对象实例`进行引用。另一方面，在 PHP 7 中，解析是一致的，从左到右进行，这意味着首先解释`$foo->$bar`，并期望一个具有`bada 元素`的数组。顺便说一句，这个例子还使用了 PHP 7 的*匿名类*特性：

```php
    // PHP 5: $foo->{$bar['bada']}
    // PHP 7: ($foo->$bar)['bada']
    $bar = 'baz';
    // $foo = new class 
    { 
        public $baz = ['bada' => 'boom']; 
    };
    // returns 'boom'
    echo $foo->$bar['bada'];
    ```

1.  最后一个示例与上面的示例相同，只是期望的返回值是一个回调，然后立即执行如下：

```php
    // PHP 5: $foo->{$bar['bada']}()
    // PHP 7: ($foo->$bar)['bada']()
    $bar = 'baz';
    // NOTE: this example uses the new PHP 7 anonymous class feature
    $foo = new class 
    { 
         public function __construct() 
        { 
            $this->baz = ['bada' => function () { return 'boom'; }]; 
        } 
    };
    // returns 'boom'
    echo $foo->$bar['bada']();
    ```

## 它是如何工作的...

将 1 和 2 中的代码示例放入一个单独的 PHP 文件中，您可以将其命名为`chap_02_understanding_diffs_in_parsing.php`。首先使用 PHP 5 执行该脚本，您将注意到会产生一系列错误，如下所示：

![它是如何工作的...](img/B05314_02_05.jpg)

错误的原因是 PHP 5 解析不一致，并且对所请求的变量变量的状态得出了错误的结论（如前所述）。现在，您可以继续添加剩余的示例，如步骤 5 和 6 所示。然后，如果您在 PHP 7 中运行此脚本，将会出现所描述的结果，如下所示：

![它是如何工作的...](img/B05314_02_06.jpg)

## 另请参阅

有关解析的更多信息，请参阅涉及**统一** **变量语法**的 RFC，可以在[`wiki.php.net/rfc/uniform_variable_syntax`](https://wiki.php.net/rfc/uniform_variable_syntax)上查看。

# 理解 foreach（）处理中的差异

在某些相对晦涩的情况下，`foreach（）`循环内部代码的行为在 PHP 5 和 PHP 7 之间会有所不同。首先，有了大量的内部改进，这意味着在`foreach（）`循环内部的处理在 PHP 7 下的速度会比在 PHP 5 下快得多。在 PHP 5 中注意到的问题包括在`foreach（）`循环内部使用`current（）`和`unset（）`对数组的操作。其他问题涉及通过引用传递值同时操作数组本身。

## 如何做...

1.  考虑以下代码块：

```php
    $a = [1, 2, 3];
    foreach ($a as $v) {
      printf("%2d\n", $v);
      unset($a[1]);
    }
    ```

1.  在 PHP 5 和 7 中，输出如下：

```php
     1
     2
     3
    ```

1.  然而，在循环之前添加一个赋值，行为会改变：

```php
    $a = [1, 2, 3];
    $b = &$a;
    foreach ($a as $v) {
      printf("%2d\n", $v);
      unset($a[1]);
    }
    ```

1.  比较 PHP 5 和 7 的输出：

| PHP 5 | PHP 7 |
| --- | --- |
| **1****3** | **1****2****3** |

1.  处理引用内部数组指针的函数在 PHP 5 中也导致不一致的行为。看下面的代码示例：

```php
    $a = [1,2,3];
    foreach($a as &$v) {
        printf("%2d - %2d\n", $v, current($a));
    }
    ```

### 提示

每个数组都有一个指向其“当前”元素的内部指针，从`1`开始，“current（）”返回数组中的当前元素。

1.  请注意，在 PHP 7 中运行的输出是规范化和一致的：

| PHP 5 | PHP 7 |
| --- | --- |
| **1 - 2****2 - 3****3 - 0** | **1 - 1****2 - 1****3 - 1** |

1.  在`foreach（）`循环中添加一个新元素，一旦引用数组迭代完成，也在 PHP 5 中存在问题。这种行为在 PHP 7 中已经变得一致。以下代码示例演示了这一点：

```php
    $a = [1];
    foreach($a as &$v) {
        printf("%2d -\n", $v);
        $a[1]=2;
    }
    ```

1.  我们将观察到以下输出：

| PHP 5 | PHP 7 |
| --- | --- |
| **1 -** | **1 -****2-** |

1.  在 PHP 5 中解决的 PHP 7 中的另一个不良行为示例是通过引用遍历数组时，使用修改数组的函数，如`array_push（）`，`array_pop（）`，`array_shift（）`和`array_unshift（）`。

看看这个例子：

```php
    $a=[1,2,3,4];
    foreach($a as &$v) {
        echo "$v\n";
        array_pop($a);
    }
    ```

1.  您将观察到以下输出：

| PHP 5 | PHP 7 |
| --- | --- |
| **1****2****1****1** | **1****2** |

1.  最后，我们有一个情况，您正在通过引用遍历数组，并且有一个嵌套的`foreach（）`循环，它本身也通过引用在相同的数组上进行迭代。在 PHP 5 中，这种结构根本不起作用。在 PHP 7 中，这个问题已经解决。以下代码块演示了这种行为：

```php
    $a = [0, 1, 2, 3];
    foreach ($a as &$x) {
           foreach ($a as &$y) {
             echo "$x - $y\n";
             if ($x == 0 && $y == 1) {
               unset($a[1]);
               unset($a[2]);
             }
           }
    }
    ```

1.  以下是输出：

| PHP 5 | PHP 7 |
| --- | --- |
| **0 - 0****0 - 1****0 - 3** | **0 - 0****0 - 1****0 - 3****3 - 0****3 -3** |

## 它是如何工作的...

将这些代码示例添加到一个名为`chap_02_foreach.php`的单个 PHP 文件中。从命令行下在 PHP 5 下运行脚本。预期输出如下：

![它是如何工作的...](img/B05314_02_07.jpg)

在 PHP 7 下运行相同的脚本并注意差异：

![它是如何工作的...](img/B05314_02_08.jpg)

## 另请参阅

有关更多信息，请参阅解决此问题的 RFC，该 RFC 已被接受。可以在以下网址找到有关此 RFC 的介绍：[`wiki.php.net/rfc/php7_foreach`](https://wiki.php.net/rfc/php7_foreach)。

# 使用 PHP 7 增强性能

开发人员正在利用的一个趋势是使用**匿名函数**。处理匿名函数时的一个经典问题是以这样的方式编写它们，以便任何对象都可以绑定到`$this`，并且函数仍然可以工作。PHP 5 代码中使用的方法是使用`bindTo（）`。在 PHP 7 中，添加了一个新方法`call（）`，它提供了类似的功能，但性能大大提高。

## 如何做...

为了利用`call（）`，在一个漫长的循环中执行一个匿名函数。在这个例子中，我们将演示一个匿名函数，它通过扫描日志文件，识别按出现频率排序的 IP 地址：

1.  首先，我们定义一个`Application\Web\Access`类。在构造函数中，我们接受一个文件名作为参数。日志文件被打开为`SplFileObject`并分配给`$this->log`：

```php
    Namespace Application\Web;

    use Exception;
    use SplFileObject;
    class Access
    {
      const ERROR_UNABLE = 'ERROR: unable to open file';
      protected $log;
      public $frequency = array();
      public function __construct($filename)
      {
        if (!file_exists($filename)) {
          $message = __METHOD__ . ' : ' . self::ERROR_UNABLE . PHP_EOL;
          $message .= strip_tags($filename) . PHP_EOL;
          throw new Exception($message);
        }
        $this->log = new SplFileObject($filename, 'r');
      }
    ```

1.  接下来，我们定义一个遍历文件的生成器，逐行进行迭代：

```php
    public function fileIteratorByLine()
    {
      $count = 0;
      while (!$this->log->eof()) {
        yield $this->log->fgets();
        $count++;
      }
      return $count;
    }
    ```

1.  最后，我们定义一个方法，查找并提取 IP 地址作为子匹配：

```php
    public function getIp($line)
    {
      preg_match('/(\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3})/', $line, $match);
      return $match[1] ?? '';
      }
    }
    ```

## 它是如何工作的...

首先，我们定义一个调用程序`chap_02_performance_using_php7_enchancement_call.php`，利用第一章中定义的自动加载类，*建立基础*，来获取`Application\Web\Access`的实例：

```php
define('LOG_FILES', '/var/log/apache2/*access*.log');
require __DIR__ . '/../Application/Autoload/Loader.php';
Application\Autoload\Loader::init(__DIR__ . '/..');
```

接下来我们定义匿名函数，它处理日志文件中的一行。如果检测到 IP 地址，它将成为`$frequency 数组`中的一个键，并且增加这个键的当前值。

```php
// define functions
$freq = function ($line) {
  $ip = $this->getIp($line);
  if ($ip) {
    echo '.';
    $this->frequency[$ip] = 
    (isset($this->frequency[$ip])) ? $this->frequency[$ip] + 1 : 1;
  }
};
```

然后我们循环遍历每个找到的日志文件中的行迭代，处理 IP 地址：

```php
foreach (glob(LOG_FILES) as $filename) {
  echo PHP_EOL . $filename . PHP_EOL;
  // access class
  $access = new Application\Web\Access($filename);
  foreach ($access->fileIteratorByLine() as $line) {
    $freq->call($access, $line);
  }
}
```

### 提示

实际上你也可以在 PHP 5 中做同样的事情。但是需要两行代码：

```php
$func = $freq->bindTo($access);
$func($line);
```

在 PHP 7 中，使用`call()`的性能比较使用`call()`慢 20%到 50%。

最后，我们对数组进行逆向排序，但保持键。输出是在一个简单的`foreach()`循环中产生的：

```php
arsort($access->frequency);
foreach ($access->frequency as $key => $value) {
  printf('%16s : %6d' . PHP_EOL, $key, $value);
}
```

输出将根据你处理的`access.log`文件而有所不同。这里是一个示例：

![它是如何工作的...](img/B05314_02_02.jpg)

## 还有更多...

许多 PHP 7 性能改进与新功能和函数无关。相反，它们采取了内部改进的形式，直到你开始运行程序之前都是*不可见*的。以下是属于这一类别的改进的简短列表：

| 功能 | 更多信息： | 注释 |
| --- | --- | --- |
| 快速参数解析 | [`wiki.php.net/rfc/fast_zpp`](https://wiki.php.net/rfc/fast_zpp) | 在 PHP 5 中，提供给函数的参数必须为每个函数调用进行解析。参数以字符串形式传递，并以类似于`scanf()`函数的方式进行解析。在 PHP 7 中，这个过程已经被优化，变得更加高效，导致了显著的性能提升。这种改进很难衡量，但似乎在 6%左右。 |
| PHP NG | [`wiki.php.net/rfc/phpng`](https://wiki.php.net/rfc/phpng) | PHP **NG**（**Next Generation**）计划代表了对大部分 PHP 语言的重写。它保留了现有功能，但涉及了所有可能的时间节省和效率措施。数据结构已经被压缩，内存利用更加高效。例如，只有一个改变影响了数组处理，导致了显著的性能提升，同时大大减少了内存使用。 |
| 去除死板 | [`wiki.php.net/rfc/removal_of_dead_sapis_and_exts`](https://wiki.php.net/rfc/removal_of_dead_sapis_and_exts) | 大约有二十多个扩展属于以下类别之一：已弃用、不再维护、未维护的依赖项，或者未移植到 PHP 7。核心开发人员组的投票决定移除“短列表”上约 2/3 的扩展。这将减少开销，并加快 PHP 语言的未来整体发展速度。 |

# 迭代处理大文件

诸如`file_get_contents()`和`file()`之类的函数使用起来快速简单，但由于内存限制，它们在处理大文件时很快会出现问题。`php.ini`中`memory_limit`设置的默认值为 128 兆字节。因此，任何大于这个值的文件都不会被加载。

在解析大文件时的另一个考虑是，你的函数或类方法产生输出的速度有多快？例如，在产生用户输出时，尽管一开始累积输出到一个数组中似乎更好。然后一次性输出以提高效率。不幸的是，这可能会对用户体验产生不利影响。也许更好的方法是创建一个**生成器**，并使用`yield 关键字`产生即时结果。

## 如何做...

如前所述，`file*`函数（即`file_get_contents()`）不适用于大文件。简单的原因是这些函数在某一点上会将整个文件内容表示在内存中。因此，本示例的重点将放在`f*`函数（即`fopen()`）上。

然而，有点不同的是，我们不直接使用`f*`函数，而是使用**SPL**（**标准 PHP 库**）中包含的`SplFileObject`类：

1.  首先，我们定义了一个`Application\Iterator\LargeFile`类，具有适当的属性和常量：

```php
    namespace Application\Iterator;

    use Exception;
    use InvalidArgumentException;
    use SplFileObject;
    use NoRewindIterator;

    class LargeFile
    {
      const ERROR_UNABLE = 'ERROR: Unable to open file';
      const ERROR_TYPE   = 'ERROR: Type must be "ByLength", "ByLine" or "Csv"';     
      protected $file;
      protected $allowedTypes = ['ByLine', 'ByLength', 'Csv'];
    ```

1.  然后我们定义了一个`__construct()`方法，接受文件名作为参数，并用`SplFileObject`实例填充`$file`属性。如果文件不存在，这也是抛出异常的好地方：

```php
    public function __construct($filename, $mode = 'r')
    {
      if (!file_exists($filename)) {
        $message = __METHOD__ . ' : ' . self::ERROR_UNABLE . PHP_EOL;
        $message .= strip_tags($filename) . PHP_EOL;
        throw new Exception($message);
      }
      $this->file = new SplFileObject($filename, $mode);
    }
    ```

1.  接下来我们定义了一个`fileIteratorByLine()method`方法，该方法使用`fgets()`逐行读取文件。创建一个类似的`fileIteratorByLength()`方法，但使用`fread()`来实现也是个不错的主意。使用`fgets()`的方法适用于包含换行符的文本文件。另一个方法可以用于解析大型二进制文件：

```php
    protected function fileIteratorByLine()
    {
      $count = 0;
      while (!$this->file->eof()) {
        yield $this->file->fgets();
        $count++;
      }
      return $count;
    }

    protected function fileIteratorByLength($numBytes = 1024)
    {
      $count = 0;
      while (!$this->file->eof()) {
        yield $this->file->fread($numBytes);
        $count++;
      }
      return $count; 
    }
    ```

1.  最后，我们定义了一个`getIterator()`方法，返回一个`NoRewindIterator()`实例。该方法接受`ByLine`或`ByLength`作为参数，这两个参数是指前一步骤中定义的两种方法。该方法还需要接受`$numBytes`，以防调用`ByLength`。我们需要一个`NoRewindIterator()`实例的原因是强制在这个例子中只能单向读取文件：

```php
    public function getIterator($type = 'ByLine', $numBytes = NULL)
    {
      if(!in_array($type, $this->allowedTypes)) {
        $message = __METHOD__ . ' : ' . self::ERROR_TYPE . PHP_EOL;
        throw new InvalidArgumentException($message);
      }
      $iterator = 'fileIterator' . $type;
      return new NoRewindIterator($this->$iterator($numBytes));
    }
    ```

## 它是如何工作的...

首先，我们利用第一章中定义的自动加载类，在调用程序`chap_02_iterating_through_a_massive_file.php`中获取`Application\Iterator\LargeFile`的实例：

```php
define('MASSIVE_FILE', '/../data/files/war_and_peace.txt');
require __DIR__ . '/../Application/Autoload/Loader.php';
Application\Autoload\Loader::init(__DIR__ . '/..');
```

接下来，在`try {...} catch () {...}`块中，我们获取一个`ByLine`迭代器的实例：

```php
try {
  $largeFile = new Application\Iterator\LargeFile(__DIR__ . MASSIVE_FILE);
  $iterator = $largeFile->getIterator('ByLine');
```

然后我们提供了一个有用的示例，即定义每行的平均单词数：

```php
$words = 0;
foreach ($iterator as $line) {
  echo $line;
  $words += str_word_count($line);
}
echo str_repeat('-', 52) . PHP_EOL;
printf("%-40s : %8d\n", 'Total Words', $words);
printf("%-40s : %8d\n", 'Average Words Per Line', 
($words / $iterator->getReturn()));
echo str_repeat('-', 52) . PHP_EOL;
```

然后我们结束`catch`块：

```php
} catch (Throwable $e) {
  echo $e->getMessage();
}
```

预期输出（太大无法在此显示！）显示了《战争与和平》古腾堡版本中有 566,095 个单词。此外，我们发现每行的平均单词数为八个。

# 将电子表格上传到数据库

虽然 PHP 没有直接读取特定电子表格格式（如 XLSX、ODS 等）的能力，但它可以读取**（CSV 逗号分隔值**）文件。因此，为了处理客户的电子表格，您需要要求他们以 CSV 格式提供文件，或者您需要自行进行转换。

## 准备就绪...

将电子表格（即 CSV 文件）上传到数据库时，有三个主要考虑因素：

+   遍历（可能）庞大的文件

+   将每个电子表格行提取为 PHP 数组

+   将 PHP 数组插入数据库

庞大文件的迭代将使用前面的方法处理。我们将使用`fgetcsv()`函数将 CSV 行转换为 PHP 数组。最后，我们将使用**（PDO PHP 数据对象**）类建立数据库连接并执行插入操作。

## 如何做...

1.  首先，我们定义了一个`Application\Database\Connection`类，该类根据构造函数提供的一组参数创建一个 PDO 实例：

```php
    <?php
      namespace Application\Database;

      use Exception;
      use PDO;

      class Connection
      { 
        const ERROR_UNABLE = 'ERROR: Unable to create database connection';    
        public $pdo;

        public function __construct(array $config)
        {
          if (!isset($config['driver'])) {
            $message = __METHOD__ . ' : ' . self::ERROR_UNABLE . PHP_EOL;
            throw new Exception($message);
        }
        $dsn = $config['driver'] 
        . ':host=' . $config['host'] 
        . ';dbname=' . $config['dbname'];
        try {
          $this->pdo = new PDO($dsn, 
          $config['user'], 
          $config['password'], 
          [PDO::ATTR_ERRMODE => $config['errmode']]);
        } catch (PDOException $e) {
          error_log($e->getMessage());
        }
      }

    }
    ```

1.  然后我们加入了一个`Application\Iterator\LargeFile`的实例。我们为这个类添加了一个新的方法，用于遍历 CSV 文件：

```php
    protected function fileIteratorCsv()
    {
      $count = 0;
      while (!$this->file->eof()) {
        yield $this->file->fgetcsv();
        $count++;
      }
      return $count;        
    }    
    ```

1.  我们还需要将`Csv`添加到允许的迭代器方法列表中：

```php
      const ERROR_UNABLE = 'ERROR: Unable to open file';
      const ERROR_TYPE   = 'ERROR: Type must be "ByLength", "ByLine" or "Csv"';

      protected $file;
      protected $allowedTypes = ['ByLine', 'ByLength', 'Csv'];
    ```

## 它是如何工作的...

首先我们定义一个配置文件，`/path/to/source/config/db.config.php`，其中包含数据库连接参数：

```php
<?php
return [
  'driver'   => 'mysql',
  'host'     => 'localhost',
  'dbname'   => 'php7cookbook',
  'user'     => 'cook',
  'password' => 'book',
  'errmode'  => PDO::ERRMODE_EXCEPTION,
];
```

接下来，我们利用第一章中定义的自动加载类，*建立基础*，来获得`Application\Database\Connection`和`Application\Iterator\LargeFile`的实例，定义一个调用程序`chap_02_uploading_csv_to_database.php`：

```php
define('DB_CONFIG_FILE', '/../data/config/db.config.php');
define('CSV_FILE', '/../data/files/prospects.csv');
require __DIR__ . '/../../Application/Autoload/Loader.php';
Application\Autoload\Loader::init(__DIR__ . '/..');
```

之后，我们设置了一个`try {...} catch () {...}`块，其中捕获了`Throwable`。这使我们能够同时捕获异常和错误：

```php
try {
  // code goes here  
} catch (Throwable $e) {
  echo $e->getMessage();
}
```

在`try {...} catch () {...}`块中，我们获得连接和大文件迭代器类的实例：

```php
$connection = new Application\Database\Connection(
include __DIR__ . DB_CONFIG_FILE);
$iterator  = (new Application\Iterator\LargeFile(__DIR__ . CSV_FILE))
->getIterator('Csv');
```

然后我们利用 PDO 准备/执行功能。准备好的语句的 SQL 使用`?`来表示在循环中提供的值：

```php
$sql = 'INSERT INTO `prospects` '
  . '(`id`,`first_name`,`last_name`,`address`,`city`,`state_province`,'
  . '`postal_code`,`phone`,`country`,`email`,`status`,`budget`,`last_updated`) '
  . ' VALUES (?,?,?,?,?,?,?,?,?,?,?,?,?)';
$statement = $connection->pdo->prepare($sql);
```

然后我们使用`foreach()`来循环遍历文件迭代器。每个`yield`语句产生一个值数组，表示数据库中的一行。然后我们可以使用这些值与`PDOStatement::execute()`一起使用，将这些值的行插入到数据库中执行准备好的语句：

```php
foreach ($iterator as $row) {
  echo implode(',', $row) . PHP_EOL;
  $statement->execute($row);
}
```

然后您可以检查数据库，以验证数据是否已成功插入。

# 递归目录迭代器

获取目录中文件的列表非常容易。传统上，开发人员使用`glob()`函数来实现这个目的。要从目录树中的特定点递归获取所有文件和目录的列表则更加棘手。这个方法利用了一个**（SPL 标准 PHP 库）**类`RecursiveDirectoryIterator`，它将非常好地实现这个目的。

这个类的作用是解析目录树，找到第一个子目录，然后沿着分支继续，直到没有更多的子目录，然后停止！不幸的是，这不是我们想要的。我们需要以某种方式让`RecursiveDirectoryIterator`继续解析每棵树和分支，从给定的起点开始，直到没有更多的文件或目录。碰巧有一个奇妙的类`RecursiveIteratorIterator`，它正好可以做到这一点。通过将`RecursiveDirectoryIterator`包装在`RecursiveIteratorIterator`中，我们可以完成对任何目录树的完整遍历。

### 提示

**警告！**

非常小心地选择文件系统遍历的起点。如果您从根目录开始，您可能会导致服务器崩溃，因为递归过程将一直持续，直到找到所有文件和目录！

## 如何做...

1.  首先，我们定义了一个`Application\Iterator\Directory`类，该类定义了适当的属性和常量，并使用外部类：

```php
    namespace Application\Iterator;

    use Exception;
    use RecursiveDirectoryIterator;
    use RecursiveIteratorIterator;
    use RecursiveRegexIterator;
    use RegexIterator;

    class Directory
    {

      const ERROR_UNABLE = 'ERROR: Unable to read directory';

      protected $path;
      protected $rdi;
      // recursive directory iterator
    ```

1.  构造函数基于目录路径创建了一个`RecursiveDirectoryIterator`实例，该实例位于`RecursiveIteratorIterator`内部：

```php
    public function __construct($path)
    {
      try {
        $this->rdi = new RecursiveIteratorIterator(
          new RecursiveDirectoryIterator($path),
          RecursiveIteratorIterator::SELF_FIRST);
      } catch (\Throwable $e) {
        $message = __METHOD__ . ' : ' . self::ERROR_UNABLE . PHP_EOL;
        $message .= strip_tags($path) . PHP_EOL;
        echo $message;
        exit;
      }
    }
    ```

1.  接下来，我们决定如何处理迭代。一种可能性是模仿 Linux 的`ls -l -R`命令的输出。请注意，我们使用了`yield`关键字，有效地将此方法转换为**生成器**，然后可以从外部调用。目录迭代产生的每个对象都是一个 SPL `FileInfo`对象，它可以为我们提供有关文件的有用信息。这个方法可能是这样的：

```php
    public function ls($pattern = NULL)
    {
      $outerIterator = ($pattern) 
      ? $this->regex($this->rdi, $pattern) 
      : $this->rdi;
      foreach($outerIterator as $obj){
        if ($obj->isDir()) {
          if ($obj->getFileName() == '..') {
            continue;
          }
          $line = $obj->getPath() . PHP_EOL;
        } else {
          $line = sprintf('%4s %1d %4s %4s %10d %12s %-40s' . PHP_EOL,
          substr(sprintf('%o', $obj->getPerms()), -4),
          ($obj->getType() == 'file') ? 1 : 2,
          $obj->getOwner(),
          $obj->getGroup(),
          $obj->getSize(),
          date('M d Y H:i', $obj->getATime()),
          $obj->getFileName());
        }
        yield $line;
      }
    }
    ```

1.  您可能已经注意到，方法调用包括文件模式。我们需要一种方法来过滤递归，只包括匹配的文件。SPL 中还有另一个迭代器完全适合这个需求：`RegexIterator`类：

```php
    protected function regex($iterator, $pattern)
    {
      $pattern = '!^.' . str_replace('.', '\\.', $pattern) . '$!';
      return new RegexIterator($iterator, $pattern);
    }
    ```

1.  最后，这是另一种方法，但这次我们将模仿`dir /s`命令：

```php
    public function dir($pattern = NULL)
    {
      $outerIterator = ($pattern) 
      ? $this->regex($this->rdi, $pattern) 
      : $this->rdi;
      foreach($outerIterator as $name => $obj){
          yield $name . PHP_EOL;
        }        
      }
    }
    ```

## 工作原理...

首先，我们利用第一章中定义的自动加载类，*建立基础*，来获得`Application\Iterator\Directory`的实例，定义一个调用程序`chap_02_recursive_directory_iterator.php`：

```php
define('EXAMPLE_PATH', realpath(__DIR__ . '/../'));
require __DIR__ . '/../Application/Autoload/Loader.php';
Application\Autoload\Loader::init(__DIR__ . '/..');
$directory = new Application\Iterator\Directory(EXAMPLE_PATH);
```

然后，在`try {...} catch () {...}`块中，我们调用了我们的两个方法，使用一个示例目录路径：

```php
try {
  echo 'Mimics "ls -l -R" ' . PHP_EOL;
  foreach ($directory->ls('*.php') as $info) {
    echo $info;
  }

  echo 'Mimics "dir /s" ' . PHP_EOL;
  foreach ($directory->dir('*.php') as $info) {
    echo $info;
  }

} catch (Throwable $e) {
  echo $e->getMessage();
}
```

`ls()`的输出将如下所示：

![工作原理...](img/B05314_02_03.jpg)

`dir()`的输出将如下所示：

![工作原理...](img/B05314_02_04.jpg)
