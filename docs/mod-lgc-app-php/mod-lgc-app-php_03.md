# 第三章：实现自动加载器

在这一步中，我们将设置自动类加载。之后，当我们需要一个类文件时，我们将不需要`include`或`require`语句来为我们加载它。在继续之前，您应该查看 PHP 自动加载器的文档 - [`www.php.net/manual/en/language.oop5.autoload.php`](http://www.php.net/manual/en/language.oop5.autoload.php)。

# PSR-0

PHP 领域中有许多不同的自动加载器建议。我们将使用基于名为`PSR-0`的东西来现代化我们的旧应用程序。

PSR-0 是 PHP 框架互操作性组的一个推荐，用于构建您的类文件。该推荐起源于许多项目使用“类到文件”命名约定的长期历史，从 PHP 4 时代开始。该约定最初由 Horde 和 PEAR 发起，后来被早期的 PHP 5 项目（如 Solar 和 Zend Framework）以及后来的项目（如 Symfony2）采用。

我们使用 PSR-0 而不是更新的 PSR-4 建议，因为我们正在处理旧代码，这些代码可能是在 PHP 5.3 命名空间出现之前开发的。在 PHP 5.3 之前编写的代码无法访问命名空间分隔符，因此遵循类到文件命名约定的作者通常会在类名中使用下划线作为伪命名空间分隔符。PSR-0 为旧的非 PHP-5.3 伪命名空间做出了让步，使其更适合我们的旧代码需求，而 PSR-4 则不适用。

根据 PSR-0，类名直接映射到文件系统的子路径。给定一个完全合格的类名，任何 PHP 5.3 命名空间分隔符都会转换为目录分隔符，并且名称中的下划线也会转换为目录分隔符。（命名空间部分中的下划线*不*会转换为目录分隔符。）结果会以基本目录位置为前缀，并以`.php`为后缀，创建一个文件路径，类文件可以在其中找到。例如，完全合格的类名`\Foo\Bar\Baz_Dib`将在 UNIX 风格的文件系统上的子路径`Foo/Bar/Baz/Dib.php`中找到。

# 类的单一位置

在实现 PSR-0 自动加载器之前，我们需要选择代码库中的一个目录位置，用于保存代码库中将来使用的所有类。一些项目已经有了这样的位置；它可能被称为`includes`、`classes`、`src`、`lib`或类似的名称。

如果已经存在这样的位置，请仔细检查它。它是否*只*包含类文件，还是包含其他类型的文件？如果它除了类文件之外还有其他东西，或者没有这样的位置存在，那么创建一个新的目录位置，并将其命名为 classes（或其他适当描述的名称）。

这个目录将是整个项目中使用的所有类的中央位置。稍后，我们将开始将类从项目中分散的位置移动到这个中央位置。

## 添加自动加载器代码

一旦我们有了一个中央目录位置用于存放我们的类文件，我们需要设置一个自动加载器来在该位置查找类。我们可以将自动加载器创建为静态方法、实例方法、匿名函数或常规全局函数。（我们使用哪种方法并不像实际进行自动加载那样重要。）然后我们将在我们的引导或设置代码中早期使用`spl_autoload_register()`进行注册，以便在调用任何类之前进行注册。

## 作为全局函数

也许实现我们的新自动加载器代码最直接的方法是作为全局函数。下面，我们找到要使用的自动加载器代码；函数名以`mlaphp_`为前缀，以确保它不会与任何现有的函数名冲突。

```php
**setup.php**
1 <?php
2 // ... setup code ...
3
4 // define an autoloader function in the global namespace
5 function mlaphp_autoloader($class)
6 {
7 // strip off any leading namespace separator from PHP 5.3
8 $class = ltrim($class, '\\');
9
10 // the eventual file path
11 $subpath = '';
12
13 // is there a PHP 5.3 namespace separator?
14 $pos = strrpos($class, '\\');
15 if ($pos !== false) {
16 // convert namespace separators to directory separators
17 $ns = substr($class, 0, $pos);
18 $subpath = str_replace('\\', DIRECTORY_SEPARATOR, $ns)
19 . DIRECTORY_SEPARATOR;
20 // remove the namespace portion from the final class name portion
21 $class = substr($class, $pos + 1);
22 }
23
24 // convert underscores in the class name to directory separators
25 $subpath .= str_replace('_', DIRECTORY_SEPARATOR, $class);
26
27 // the path to our central class directory location
28 $dir = '/path/to/app/classes';
29
30 // prefix with the central directory location and suffix with .php,
31 // then require it.
32 $file = $dir . DIRECTORY_SEPARATOR . $subpath . '.php';
33 require $file;
34 }
35
36 // register it with SPL
37 spl_autoload_register('mlaphp_autoloader');
38 ?>
```

请注意，`$dir`变量表示中央类目录的基本路径的绝对目录。作为 PHP 5.3 及更高版本的替代方案，可以在该变量中使用`__DIR__`常量，以便绝对路径不再是硬编码的，而是相对于函数所在文件的路径。例如：

```php
1 <?php
2 // go "up one directory" for the central classes location
3 $dir = dirname(__DIR__) . '/classes';
4 ?>
```

如果由于某种原因我们卡在 PHP 5.2 上，`__DIR__`常量是不可用的。在这种情况下，可以用`dirname(dirname(__FILE__))`替换`dirname(__DIR__)`。

### 作为闭包

如果我们使用的是 PHP 5.3，我们可以将自动加载器代码创建为一个闭包，并在一个步骤中将其注册到 SPL 中：

```php
**setup.php**
1 <?php
2 // ... setup code ...
3
4 // register an autoloader as an anonymous function
5 spl_autoload_register(function ($class) {
6 // ... the same code as in the global function ...
7 });
8
9 // ... other setup code ...
10 ?>
```

### 作为静态或实例方法

这是我设置自动加载器的首选方式。我们不使用函数，而是将自动加载器代码创建为一个类的实例方法或静态方法。我推荐实例方法而不是静态方法，但您的情况将决定哪种更合适。

首先，我们在我们的中央类目录位置创建我们的自动加载器类文件。如果我们使用的是 PHP 5.3 或更高版本，我们应该使用一个合适的命名空间；否则，我们使用下划线作为伪命名空间分隔符。

以下是一个 PHP 5.3 的示例。在 PHP 5.3 之前的版本中，我们将省略`namespace`声明，并将类命名为`Mlaphp_Autoloader`。无论哪种方式，文件都应该在子路径`Mlaphp/Autoloader.php`中：

```php
**/path/to/app/classes/Mlaphp/Autoloader.php**
1 <?php
2 namespace Mlaphp;
3
4 class Autoloader
5 {
6 // an instance method alternative
7 public function load($class)
8 {
9 // ... the same code as in the global function ...
10 }
11
12 // a static method alternative
13 static public function loadStatic($class)
14 {
15 // ... the same code as in the global function ...
16 }
17 }
18 ?>
```

然后，在设置或引导文件中，`require_once`类文件，根据需要实例化它，并使用 SPL 注册方法。请注意，我们在这里使用数组可调用格式，第一个数组元素是类名或对象实例，第二个元素是要调用的方法：

```php
**setup.php**
1 <?php
2 // ... setup code ...
3
4 // require the autoloader class file
5 require_once '/path/to/app/classes/Mlaphp/Autoloader.php';
6
7 // STATIC OPTION: register a static method with SPL
8 spl_autoload_register(array('Mlaphp\Autoloader', 'loadStatic'));
9
10 // INSTANCE OPTION: create the instance and register the method with SPL
11 $autoloader = new \Mlaphp\Autoloader();
12 spl_autoload_register(array($autoloader, 'load'));
13
14 // ... other setup code ...
15 ?>
```

请选择实例方法或静态方法，而不是两者兼有。一个不是另一个的备用。

# 使用`__autoload()`函数

如果由于某种原因我们卡在 PHP 5.0 上，我们可以使用`__autoload()`函数来代替 SPL 自动加载器注册。这样做有一些缺点，但在 PHP 5.0 下这是我们唯一的选择。我们不需要在 SPL 中注册它（事实上，我们不能这样做，因为 SPL 直到 PHP 5.1 才被引入）。在这种实现中，我们将无法混合和匹配其他自动加载器；只允许一个`__autoload()`函数。如果`__autoload()`函数已经被定义，我们需要将这段代码与函数中已经存在的任何代码合并：

```php
**setup.php**
1 <?php
2 // ... setup code ...
3
4 // define an __autoload() function
5 function __autoload($class)
6 {
7 // ... the global function code ...
8 }
9
10 // ... other setup code ...
11 ?>
```

我强烈建议不要在 PHP 5.1 及更高版本中使用这种实现。

## 自动加载器优先级

无论我们如何实现我们的自动加载器代码，我们都需要在代码库中调用任何类之前使其可用。在我们的代码库中注册自动加载器作为最初的逻辑之一可能不会有害，可能在设置或引导脚本中。

# 常见问题

## 如果我已经有一个自动加载器呢？

一些传统应用程序可能已经有一个自定义的自动加载器。如果是这种情况，我们有一些选择：

1.  **使用现有的自动加载器**：如果已经有一个应用程序类文件的中央目录位置，这是我们最好的选择。

1.  **修改现有的自动加载器以添加 PSR-0 行为**：如果自动加载器不符合 PSR-0 建议，这是一个很好的选择。

1.  在 SPL 中注册本章描述的 PSR-0 自动加载器，以及现有的自动加载器。当现有的自动加载器不符合 PSR-0 建议时，这是另一个很好的选择。

其他传统代码库可能已经有第三方自动加载器，比如 Composer。如果 Composer 存在，我们可以获取它的自动加载器实例，并像这样添加我们的中央类目录位置进行自动加载：

```php
1 <?php
2 // get the registered Composer autoloader instance from the vendor/
3 // subdirectory
4 $loader = require '/path/to/app/vendor/autoload.php';
5
6 // add our central class directory location; do not use a class prefix as
7 // we may have more than one top-level namespace in the central location
8 $loader->add('', '/path/to/app/classes');
9 ?>
```

有了这个，我们可以利用 Composer 来实现我们自己的目的，从而使我们自己的自动加载器代码变得不必要。

## 自动加载的性能影响是什么？

有一些理由认为使用自动加载器可能会导致轻微的性能下降，与使用`include`相比，但证据参差不齐且情况依赖。如果自动加载相对较慢，那么可以预期会有多大的性能损失？

我断言，在现代化遗留应用程序时，这可能并不是一个重要的考虑因素。与遗留应用程序中可能存在的其他性能问题相比，自动加载所带来的性能损失微不足道，比如数据库交互。

在大多数遗留应用程序中，甚至在大多数现代应用程序中，试图优化自动加载的性能是试图优化错误的资源。可能存在其他更严重的资源，只是我们看不到或没有考虑到。

如果自动加载是你的遗留应用中性能最差的瓶颈，那么你的状况非常好。（在这种情况下，你应该退还这本书，然后告诉我你是否在招聘，因为我想为你工作。）

## 类名如何映射到文件名？

PSR-0 规则可能令人困惑。以下是一些类到文件映射的示例，以说明其期望：

```php
Foo           => Foo.php
Foo_Bar       => Foo/Bar.php
Foo           => Foo/Bar.php
Foo_Bar\Bar   => Foo_Bar/Baz.php
Foo\Bar\Baz   => Foo/Bar/Baz.php # ???
Foo\Baz_Bar   => Foo/Bar/Baz.php # ???
Foo_Bar_Baz   => Foo/Bar/Baz.php # ???
```

我们可以看到最后三个示例中存在一些意外行为。这是由于 PSR-0 的过渡性质造成的：`Foo\Bar\Baz, Foo\Bar_Baz`和`Foo_Bar_Baz`都映射到同一个文件。为什么会这样？

回想一下，PHP 5.3 之前的代码库没有命名空间，因此使用下划线作为伪命名空间分隔符。PHP 5.3 引入了真正的命名空间分隔符。PSR-0 标准必须同时适应这两种情况，因此它将相对类名（即完全合格名称的最后部分）中的下划线视为目录分隔符，但命名空间部分中的下划线则保持不变。

这里的教训是，如果你使用的是 PHP 5.3，你不应该在相对类名中使用下划线（尽管在命名空间中使用下划线是可以的）。如果你使用的是 PHP 5.3 之前的版本，你别无选择，只能使用下划线，因为只有类名，没有实际的命名空间部分；在这种情况下，将下划线解释为命名空间分隔符。

# 回顾和下一步

到目前为止，我们并没有对我们的遗留应用进行太多修改。我们添加并注册了一些自动加载器代码，但实际上还没有被调用。

无论如何。拥有一个自动加载器对于我们现代化遗留应用的下一步至关重要。使用自动加载器将允许我们开始移除仅加载类和函数的`include`语句。剩下的`include`语句将是逻辑流程包含，向我们展示系统的哪些部分是逻辑的，哪些是仅定义的。这是我们从基于 include 的架构向基于类的架构过渡的开始。
