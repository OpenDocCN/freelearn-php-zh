# 第一章：介绍新的 PHP 8 OOP 特性

在本章中，您将了解到针对**面向对象编程**（**OOP**）的**PHP: Hypertext Preprocessor 8**（**PHP 8**）的新特性。本章介绍了一组类，可用于生成 CAPTCHA 图像（**CAPTCHA**是**Completely Automated Public Turing test to tell Computers and Humans Apart**的缩写），清晰地说明了新的 PHP 8 特性和概念。本章对于帮助您快速将新的 PHP 8 特性纳入到您自己的实践中至关重要。这样做，您的代码将运行得更快、更高效，bug 更少。

本章涵盖以下主题：

+   使用构造函数属性提升

+   使用属性

+   将匹配表达式纳入您的程序代码

+   理解命名参数

+   探索新的数据类型

+   使用类型属性改进代码

# 技术要求

要检查和运行本章提供的代码示例，以下是最低推荐的硬件要求：

+   基于 x86_64 的台式 PC 或笔记本电脑

+   1 GB 的可用磁盘空间

+   4 GB 的**随机存取存储器**（**RAM**）

+   500 **千位每秒**（**Kbps**）或更快的互联网连接

此外，您需要安装以下软件：

+   Docker

+   Docker Compose

本书使用一个预构建的 Docker 镜像，其中包含创建和运行本书中涵盖的 PHP 8 代码示例所需的所有软件。您不需要在计算机上安装 PHP、Apache 或 MySQL：只需使用 Docker 和提供的镜像即可。

要设置一个用于运行代码示例的测试环境，请按照以下步骤进行：

1.  安装 Docker。

如果您正在运行 Windows，请从这里开始：

[`docs.docker.com/docker-for-windows/install/`](https://docs.docker.com/docker-for-windows/install/ )

如果您使用 Mac，请从这里开始：

[`docs.docker.com/docker-for-mac/install/`](https://docs.docker.com/docker-for-mac/install/ )

如果您使用 Linux，请看这里：

[`docs.docker.com/engine/install/`](https://docs.docker.com/engine/install/ )

1.  安装 Docker Compose。对于所有操作系统，请从这里开始：

[`docs.docker.com/compose/install/`](https://docs.docker.com/compose/install/ )

1.  将与本书相关的源代码安装到您的本地计算机上。

如果您已安装 Git，请使用以下命令：

```php
git clone https://github.com/PacktPublishing/PHP-8-Programming-Tips-Tricks-and-Best-Practices.git ~/repo
```

否则，您可以直接从以下**统一资源定位器**（**URL**）下载源代码：[`github.com/PacktPublishing/PHP-8-Programming-Tips-Tricks-and-Best-Practices/archive/main.zip`](https://github.com/PacktPublishing/PHP-8-Programming-Tips-Tricks-and-Best-Practices/archive/main.zip)。然后解压到一个您创建的文件夹中，在本书中我们将其称为`/repo`。

1.  您现在可以启动 Docker 守护程序。对于 Windows 或 Mac，您只需要激活 Docker Desktop 应用程序。

如果您正在运行 Ubuntu 或 Debian Linux，请发出以下命令：

`sudo service docker start`

对于 Red Hat、Fedora 或 CentOS，请使用以下命令：

使用`sudo systemctl start docker`命令启动 Docker。

1.  构建与本书相关的 Docker 容器并将其上线。要做到这一点，请按照以下步骤进行。

从您的本地计算机，打开命令提示符（终端窗口）。将目录更改为`/repo`。仅首次，发出`docker-compose build`命令来*构建*环境。请注意，您可能需要`root`（管理员）权限来运行 Docker 命令。如果是这种情况，要么以管理员身份运行（对于 Windows），要么在命令前加上`sudo`。根据您的连接速度，初始构建可能需要相当长的时间才能完成！

1.  要启动容器，请按照以下步骤进行

1.  从您的本地计算机，打开命令提示符（终端窗口）。将目录更改为`/repo`。通过运行以下命令以后台模式启动 Docker 容器：

```php
docker-compose up -d
```

请注意，实际上您不需要单独构建容器。如果在发出`docker-compose up`命令时容器尚未构建，它将自动构建。另一方面，单独构建容器可能很方便，这种情况下只需使用`docker build`即可。

这是一个确保所有容器都在运行的有用命令：

```php
docker-compose ps
```

1.  要访问运行中的 Docker 容器 Web 服务器，请按照以下步骤进行。

在您的本地计算机上打开浏览器。输入此 URL 以访问 PHP 8 代码：

`http://localhost:8888`

输入此 URL 以访问 PHP 7 代码：

`http://localhost:7777`

1.  要打开运行中的 Docker 容器的命令行，按照以下步骤进行。

从您的本地计算机上，打开命令提示符（终端窗口）。发出此命令以访问 PHP 8 容器：

```php
docker exec -it php8_tips_php8 /bin/bash 
```

发出此命令以访问 PHP 7 容器：

```php
docker exec -it php8_tips_php7 /bin/bash
```

1.  当您完成与容器的工作后，要将其脱机，请从您的本地计算机上打开命令提示符（终端窗口）并发出此命令：

```php
docker-compose down 
```

本章的源代码位于此处：

[`github.com/PacktPublishing/PHP-8-Programming-Tips-Tricks-and-Best-Practices`](https://github.com/PacktPublishing/PHP-8-Programming-Tips-Tricks-and-Best-Practices )

重要提示

如果您的主机计算机使用**高级精简指令集**（**ARM**）架构（例如，树莓派），您将需要使用修改后的 Dockerfile。

提示

通过查看这篇文章，快速了解 Docker 技术和术语是一个很好的主意：[`docs.docker.com/get-started/.`](https://docs.docker.com/get-started/ )

我们现在可以通过查看构造函数属性提升来开始我们的讨论。

# 使用构造函数属性提升

除了**即时**（**JIT**）编译器之外，PHP 8 中引入的最大的新功能之一是**构造函数属性提升**。这个新功能将属性声明和`__construct()`方法签名中的参数列表以及赋默认值结合在一起。在本节中，您将学习如何大大减少属性声明和`__construct()`方法签名和主体中所需的编码量。

## 属性提升语法

调用构造函数属性提升所需的语法与 PHP 7 和之前使用的语法相同，有以下区别：

+   您需要定义一个**可见级别**

+   您不必事先显式声明属性。

+   您不需要在`__construct()`方法的主体中进行赋值

这是一个使用构造函数属性提升的代码的简单示例：

```php
// /repo/ch01/php8_prop_promo.php
declare(strict_types=1);
class Test {
    public function __construct(
        public int $id,
        public int $token = 0,
        public string $name = '')
    { }
}
$test = new Test(999);
var_dump($test);
```

当执行前面的代码块时，这是输出：

```php
object(Test)#1 (3) {
  ["id"]=> int(999)
  ["token"]=> int(0)
  ["name"]=> string(0) ""
}
```

这表明使用默认值创建了`Test`类型的实例。现在，让我们看看这个功能如何可以节省大量的编码。

## 使用属性提升来减少代码

在传统的 OOP PHP 类中，需要完成以下三件事：

1.  声明属性，如下所示：

```php
/repo/src/Php8/Image/SingleChar.php
namespace Php7\Image;
class SingleChar {
    public $text     = '';
    public $fontFile = '';
    public $width    = 100;
    public $height   = 100;
    public $size     = 0;
    public $angle    = 0.00;
    public $textX    = 0;
    public $textY    = 0;
```

1.  在`__construct()`方法签名中标识属性及其数据类型，如下所示：

```php
const DEFAULT_TX_X = 25;
const DEFAULT_TX_Y = 75;
const DEFAULT_TX_SIZE  = 60;
const DEFAULT_TX_ANGLE = 0;
public function __construct(
    string $text,
    string $fontFile,
    int $width  = 100,
    int $height = 100,
    int $size   = self::DEFAULT_TX_SIZE,
    float $angle = self::DEFAULT_TX_ANGLE,
    int $textX  = self::DEFAULT_TX_X,
    int $textY  = self::DEFAULT_TX_Y)   
```

1.  在`__construct()`方法的主体中，为属性赋值，就像这样：

```php
{   $this->text     = $text;
    $this->fontFile = $fontFile;
    $this->width    = $width;
    $this->height   = $height;
    $this->size     = $size;
    $this->angle    = $angle;
    $this->textX    = $textX;
    $this->textY    = $textY;
    // other code not shown 
}
```

随着构造函数参数的增加，您需要做的工作量也会显著增加。当应用构造函数属性提升时，以前所需的相同代码量减少到原来的三分之一。

现在让我们看一下之前显示的同一段代码块，但是使用这个强大的新 PHP 8 功能进行重写：

```php
// /repo/src/Php8/Image/SingleChar.php
// not all code shown
public function __construct(
    public string $text,
    public string $fontFile,
    public int    $width    = 100,
    public int    $height   = 100,
    public int    $size     = self::DEFAULT_TX_SIZE,
    public float   $angle    = self::DEFAULT_TX_ANGLE,
    public int    $textX    = self::DEFAULT_TX_X,
    public int    $textY    = self::DEFAULT_TX_Y)
    { // other code not shown }
```

令人惊讶的是，在 PHP 7 和之前的版本中需要 24 行代码，而使用这个新的 PHP 8 功能可以缩减为 8 行代码！

您完全可以在构造函数中包含其他代码。然而，在许多情况下，构造函数属性提升会处理`__construct()`方法中通常完成的所有工作，这意味着您可以将其留空（`{ }`）。

现在，在下一节中，您将了解一个称为属性的新功能。

提示

在这里查看 PHP 7 的完整 SingleChar 类的更多信息：

[`github.com/PacktPublishing/PHP-8-Programming-Tips-Tricks-and-Best-Practices/tree/main/src/Php7/Image`](https://github.com/PacktPublishing/PHP-8-Programming-Tips-Tricks-and-Best-Practices/tree/main/src/Php7/Image)

此外，等效的 PHP 8 类可以在这里找到：

[`github.com/PacktPublishing/PHP-8-Programming-Tips-Tricks-and-Best-Practices/tree/main/src/Php8/Image`](https://github.com/PacktPublishing/PHP-8-Programming-Tips-Tricks-and-Best-Practices/tree/main/src/Php8/Image)

有关此新功能的更多信息，请参阅以下内容：

[`wiki.php.net/rfc/constructor_promotion`](https://wiki.php.net/rfc/constructor_promotion)

# 使用 attributes

PHP 8 的另一个重要补充是全新的类和语言构造，称为**attributes**。简而言之，attributes 是传统 PHP 注释块的替代品，遵循规定的语法。当 PHP 代码编译时，这些 attributes 会在内部转换为`Attribute`类实例。

这个新功能不会立即影响您的代码。然而，随着各种 PHP 开源供应商开始将 attributes 纳入其代码中，它将开始变得越来越有影响力。

`Attribute`类解决了我们在本节讨论的一个潜在重要的性能问题，即滥用传统 PHP 注释块提供元指令。在我们深入讨论这个问题以及`Attribute`类实例如何解决问题之前，我们首先必须回顾一下 PHP 注释。

## PHP 注释概述

这种语言构造的需求是随着对普通 PHP 注释的使用（和滥用！）的增加而产生的。正如您所知，注释有许多形式，包括以下所有形式：

```php
# This is a "bash" shell script style comment
// this can either be inline or on its own line
/* This is the traditional "C" language style */
/**
 * This is a PHP "DocBlock"
 */
```

最后一项，著名的 PHP `DocBlock`，现在被广泛使用，已成为事实上的标准。使用 DocBlocks 并不是一件坏事。相反，这往往是开发人员能够传达有关属性、类和方法的信息的*唯一* *方式*。问题只在于它在 PHP 解释过程中的处理方式。

## PHP DocBlock 注意事项

**PHP DocBlock**的原始意图已经被一些非常重要的 PHP 开源项目所拉伸。一个鲜明的例子是 Doctrine **对象关系映射**（**ORM**）项目。虽然不是强制的，但许多开发人员选择使用嵌套在 PHP DocBlocks 中的**annotations**来定义 ORM 属性。

看一下这个部分代码示例，它定义了一个与名为`events`的数据库表交互的类：

```php
namespace Php7\Entity;
use Doctrine\ORM\Mapping as ORM;
/**
 * @ORM\Table(name="events")
 * @ORM\Entity("Application\Entity\Events")
 */
class Events {
    /**
     * @ORM\Column(name="id",type="integer",nullable=false)
     * @ORM\Id
     * @ORM\GeneratedValue(strategy="IDENTITY")
     */
    private $id;
    /**
     * @ORM\Column(name="event_key", type="string", 
          length=16, nullable=true, options={"fixed"=true})
     */
    private $eventKey;
    // other code not shown
```

如果您要将此类用作 Doctrine ORM 实现的一部分，Doctrine 将打开文件并解析 DocBlocks，搜索`@ORM`注释。尽管对解析 DocBlocks 所需的时间和资源有一些担忧，但这是一种非常方便的方式来定义对象属性和数据库表列之间的关系，并且受到使用 Doctrine 的开发人员的欢迎。

提示

Doctrine 提供了许多替代方案来实现 ORM，包括**可扩展标记语言**（**XML**）和本机 PHP 数组。有关更多信息，请参阅[`www.doctrine-project.org/projects/doctrine-orm/en/latest/reference/annotations-reference.html#annotations-reference`](https://www.doctrine-project.org/projects/doctrine-orm/en/latest/reference/annotations-reference.html#annotations-reference)。

## 与滥用 DocBlocks 相关的潜在危险

与滥用 DocBlock 的原始目的相关的另一个危险是。在`php.ini`文件中，有一个名为`opcache.save_comments`的设置。如果禁用，这将导致 OpCode 缓存引擎（**OPcache**）*忽略*所有注释，包括 DocBlocks。如果此设置生效，使用`@ORM`注释的基于 Doctrine 的应用程序将发生故障。

另一个问题与注释的解析有关，或者更准确地说，与注释的*不*解析有关。为了使用注释的内容，PHP 应用程序需要逐行打开文件并解析它。这在时间和资源利用方面是一个昂贵的过程。

## 属性类

为了解决隐藏的危险，在 PHP 8 中提供了一个新的`Attribute`类。开发人员可以定义等效的属性形式，而不是使用带注释的 DocBlocks。使用属性而不是 DocBlocks 的优势在于它们是语言的*正式部分*，因此它们与代码的其余部分一起被标记化和编译。

重要提示

在本章中，以及在 PHP 文档中，*属性*的引用指的是`Attribute`类的实例。

目前尚无实际的性能指标可比较包含 DocBlocks 的 PHP 代码的加载与包含属性的代码的加载。

尽管这种方法的好处尚未显现，但随着各种开源项目供应商开始将属性纳入其产品中，您将开始看到速度和性能的提高。

这是`Attribute`类的定义：

```php
class Attribute {
    public const int TARGET_CLASS = 1;
    public const int TARGET_FUNCTION = (1 << 1);
    public const int TARGET_METHOD = (1 << 2);
    public const int TARGET_PROPERTY = (1 << 3);
    public const int TARGET_CLASS_CONSTANT = (1 << 4);
    public const int TARGET_PARAMETER = (1 << 5);
    public const int TARGET_ALL = ((1 << 6) - 1);
    public function __construct(
        int $flags = self::TARGET_ALL) {}
}
```

从类定义中可以看出，这个类在 PHP 8 内部使用的主要贡献是一组类常量。这些常量代表可以使用位运算符组合的位标志。

## 属性语法

属性使用了从**Rust**编程语言借鉴的特殊语法。方括号内的内容基本上由开发人员决定。以下代码段中可以看到一个示例：

```php
#[attribute("some text")] 
// class, property, method or function (or whatever!)
```

回到我们的`SingleChar`类的示例，这是如何在传统的 DocBlocks 中出现的：

```php
// /repo/src/Php7/Image/SingleChar.php
namespace Php7\Image;
/**
 * Creates a single image, by default black on white
 */
class SingleChar {
    /**
     * Allocates a color resource
     *
     * @param array|int $r,
     * @param int $g
     * @param int $b]
     * @return int $color
     */
    public function colorAlloc() 
    { /* code not shown */ } 
```

现在，看看使用属性的相同内容：

```php
// /repo/src/Php8/Image/SingleChar.php
namespace Php8\Image;
#[description("Creates a single image")]
class SingleChar {
    #[SingleChar\colorAlloc\description("Allocates color")]
    #[SingleChar\colorAlloc\param("r","int|array")]
    #[SingleChar\colorAlloc\param("g","int")]
    #[SingleChar\colorAlloc\param("b","int")]
    #[SingleChar\colorAlloc\returns("int")]
    public function colorAlloc() { /* code not shown */ }
```

如您所见，除了提供更强大的编译和避免上述隐藏危险之外，它在空间使用方面也更有效。

提示

方括号内的内容确实有一些限制；例如，虽然允许`#[returns("int")]`，但不允许这样做：`#[return("int")`。原因是`return`是一个关键字。

另一个例子涉及**联合类型**（在*探索新数据类型*部分中解释）。您可以在属性中使用`#[param("int|array test")]`，但不允许这样做：`#[int|array("test")]`。另一个特殊之处是类级别的属性必须放在`class`关键字之前，并在任何`use`语句之后。

### 使用 Reflection 查看属性

如果您需要从 PHP 8 类获取属性信息，`Reflection`扩展已更新以包括属性支持。添加了一个新的`getAttributes()`方法，返回一个`ReflectionAttribute`实例数组。

在以下代码块中，显示了`Php8\Image\SingleChar::colorAlloc()`方法的所有属性：

```php
<?php
// /repo/ch01/php8_attrib_reflect.php
define('FONT_FILE', __DIR__ . '/../fonts/FreeSansBold.ttf');
require_once __DIR__ . '/../src/Server/Autoload/Loader.php';
$loader = new \Server\Autoload\Loader();
use Php8\Image\SingleChar;
$char    = new SingleChar('A', FONT_FILE);
$reflect = new ReflectionObject($char);
$attribs = $reflect->getAttributes();
echo "Class Attributes\n";
foreach ($attribs as $obj) {
    echo "\n" . $obj->getName() . "\n";
    echo implode("\t", $obj->getArguments());
}
echo "Method Attributes for colorAlloc()\n";
$reflect = new ReflectionMethod($char, 'colorAlloc');
$attribs = $reflect->getAttributes();
foreach ($attribs as $obj) {
    echo "\n" . $obj->getName() . "\n";
    echo implode("\t", $obj->getArguments());
}
```

以下是前面代码段中显示的输出：

```php
<pre>Class Attributes
Php8\Image\SingleChar
Php8\Image\description
Creates a single image, by default black on whiteMethod
Attributes for colorAlloc()
Php8\Image\SingleChar\colorAlloc\description
Allocates a color resource
Php8\Image\SingleChar\colorAlloc\param
r    int|array
Php8\Image\SingleChar\colorAlloc\param
g    int
Php8\Image\SingleChar\colorAlloc\param
b    int
Php8\Image\SingleChar\colorAlloc\returns
int
```

前面的输出显示了可以使用`Reflection`扩展类检测属性。最后，这段代码示例展示了实际的方法：

```php
namespace Php8\Image;use Attribute;
use Php8\Image\Strategy\ {PlainText,PlainFill};
#[SingleChar]
#[description("Creates black on white image")]
class SingleChar {
    // not all code is shown
    #[SingleChar\colorAlloc\description("Allocates color")]
    #[SingleChar\colorAlloc\param("r","int|array")]
    #[SingleChar\colorAlloc\param("g","int")]
    #[SingleChar\colorAlloc\param("b","int")]
    #[SingleChar\colorAlloc\returns("int")]    
    public function colorAlloc(
         int|array $r, int $g = 0, int $b = 0) {
        if (is_array($r))
            [$r, $g, $b] = $r;
        return \imagecolorallocate(
              $this->image, $r, $g, $b);
    }
}
```

现在，您已经了解了属性的使用方式，让我们继续讨论`match`表达式，然后是命名参数的新功能。

提示

有关此新功能的更多信息，请查看以下网页：

[`wiki.php.net/rfc/attributes_v2`](https://wiki.php.net/rfc/attributes_v2 )

另请参阅此更新：

[`wiki.php.net/rfc/shorter_attribute_syntax_change`](https://wiki.php.net/rfc/shorter_attribute_syntax_change )

有关 PHP DocBlocks 的信息可以在这里找到：

[`phpdoc.org/`](https://phpdoc.org/ )

有关 Doctrine ORM 的更多信息，请查看这里：

[`www.doctrine-project.org/projects/orm.html`](https://www.doctrine-project.org/projects/orm.html )

有关`php.ini`文件设置的文档可以在这里找到：

[`www.php.net/manual/en/ini.list.php`](https://www.php.net/manual/en/ini.list.php)

在这里阅读有关 PHP 反射的信息：

[`www.php.net/manual/en/language.attributes.reflection.php`](https://www.php.net/manual/en/language.attributes.reflection.php)

有关 Rust 编程语言的信息可以在这本书中找到：[`www.packtpub.com/product/mastering-rust-second-edition/9781789346572`](https://www.packtpub.com/product/mastering-rust-second-edition/9781789346572)

# 将 match 表达式合并到程序代码中

在 PHP 8 中引入的许多非常有用的功能中，**match 表达式**绝对脱颖而出。`Match`表达式是一种更准确的简写语法，可以潜在地取代直接来自 C 语言的老旧`switch`语句。在本节中，您将学习如何通过用`match`表达式替换`switch`语句来生成更清晰和更准确的程序代码。

## Match 表达式的一般语法

`Match`表达式语法非常类似于数组，其中键是要匹配的项，值是一个表达式。以下是`match`的一般语法：

```php
$result = match(<EXPRESSION>) {
    <ITEM> => <EXPRESSION>,
   [<ITEM> => <EXPRESSION>,]
    default => <DEFAULT EXPRESSION>
};
```

表达式必须是有效的 PHP 表达式。表达式的示例可以包括以下任何一种：

+   一个特定的值（例如，`"一些文本"`）

+   一个操作（例如，`$a + $b`）

+   匿名函数或类

唯一的限制是表达式必须在一行代码中定义。`match`和`switch`之间的主要区别在这里总结：

![表 1.1 - match 和 switch 之间的区别](img/Table_1.1_B16992.jpg)

表 1.1 - match 和 switch 之间的区别

除了上述区别之外，`match`和`switch`都允许案例聚合，并提供对*default*案例的支持。

### switch 和 match 示例

这是一个简单的示例，使用`switch`来渲染货币符号：

```php
// /repo/ch01/php7_switch.php
function get_symbol($iso) {
    switch ($iso) {
        case 'CNY' :
            $sym = '¥';
            break;
        case 'EUR' :
            $sym = '€';
            break;
        case 'EGP' :
        case 'GBP' :
            $sym = '£';
            break;
        case 'THB' :
            $sym = '฿';
            break;
        default :
            $sym = '$';
    }
    return $sym;
}
$test = ['CNY', 'EGP', 'EUR', 'GBP', 'THB', 'MXD'];
foreach ($test as $iso)
    echo 'The currency symbol for ' . $iso
         . ' is ' . get_symbol($iso) . "\n";
```

当执行此代码时，您会看到`$test`数组中每个**国际标准化组织**（**ISO**）货币代码的货币符号。在 PHP 8 中可以获得与前面代码片段中显示的相同结果，使用以下代码：

```php
// /repo/ch01/php8_switch.php
function get_symbol($iso) {
    return match ($iso) {
        'EGP','GBP' => '£',
        'CNY'       => '¥',
        'EUR'       => '€',
        'THB'       => '฿',
        default     => '$'
    };
}
$test = ['CNY', 'EGP', 'EUR', 'GBP', 'THB', 'MXD'];
foreach ($test as $iso)
    echo 'The currency symbol for ' . $iso
         . ' is ' . get_symbol($iso) . "\n";
```

两个示例产生相同的输出，如下所示：

```php
The currency symbol for CNY is ¥
The currency symbol for EGP is £
The currency symbol for EUR is €
The currency symbol for GBP is £
The currency symbol for THB is ฿
The currency symbol for MXD is $
```

如前所述，这两个代码示例都会为存储在`$test`数组中的 ISO 货币代码列表产生货币符号列表。

### 复杂的 match 示例

回到我们的验证码项目，假设我们希望引入扭曲以使验证码字符更难阅读。为了实现这个目标，我们引入了许多**策略**类，每个类产生不同的扭曲，如下表所总结的：

![表 1.2 - 验证码扭曲策略类](img/Table_1.2_B16992.jpg)

表 1.2 - 验证码扭曲策略类

在随机排列要使用的策略列表之后，我们使用`match`表达式来执行结果，如下所示：

1.  首先我们定义一个**自动加载程序**，导入要使用的类，并列出要使用的潜在策略，如下所示：

```php
// /repo/ch01/php8_single_strategies.php
// not all code is shown
require_once __DIR__ . '/../src/Server/Autoload/Loader.php';
$loader = new \Server\Autoload\Loader();
use Php8\Image\SingleChar;
use Php8\Image\Strategy\ {LineFill,DotFill,Shadow,RotateText};
$strategies = ['rotate', 'line', 'line',
               'dot', 'dot', 'shadow'];
```

1.  接下来，我们生成验证码短语，如下所示：

```php
$phrase = strtoupper(bin2hex(random_bytes(NUM_BYTES)));
$length = strlen($phrase);
```

1.  然后我们循环遍历验证码短语中的每个字符，并创建一个`SingleChar`实例。对`writeFill()`的初始调用创建了白色背景画布。我们还需要调用`shuffle()`来随机排列扭曲策略的列表。该过程在以下代码片段中说明：

```php
$images = [];
for ($x = 0; $x < $length; $x++) {
    $char = new SingleChar($phrase[$x], FONT_FILE);
    $char->writeFill();
    shuffle($strategies);
```

1.  然后我们循环遍历策略并在原始图像上叠加扭曲。这就是`match`表达式发挥作用的地方。请注意，一个策略需要额外的代码行。因为`match`只能支持单个表达式，所以我们简单地将多行代码包装到一个**匿名函数**中，如下所示：

```php
foreach ($strategies as $item) {
    $func = match ($item) {    
        'rotate' => RotateText::writeText($char),
        'line' => LineFill::writeFill(
            $char, rand(1, 10)),
        'dot' => DotFill::writeFill($char, rand(10, 20)),
        'shadow' => function ($char) {
            $num = rand(1, 8);
            $r   = rand(0x70, 0xEF);
            $g   = rand(0x70, 0xEF);
            $b   = rand(0x70, 0xEF);
            return Shadow::writeText(
                $char, $num, $r, $g, $b);},
        'default' => TRUE
    };
    if (is_callable($func)) $func($char);
}
```

1.  现在要做的就是通过不带参数调用`writeText()`来覆盖图像。之后，我们将扭曲的图像保存为**便携式网络图形**（**PNG**）文件以供显示，如下面的代码片段所示：

```php
    $char->writeText();
    $fn = $x . '_' 
         . substr(basename(__FILE__), 0, -4) 
         . '.png';
    $char->save(IMG_DIR . '/' . $fn);
    $images[] = $fn;
}
include __DIR__ . '/captcha_simple.phtml';
```

这是从指向本书关联的 Docker 容器的浏览器中运行前面示例的结果：

![图 1.1 - 使用匹配表达式扭曲的验证码](img/Figure_1.1_B16992.jpg)

图 1.1 - 使用匹配表达式扭曲的验证码

接下来，我们将看一下另一个非常棒的功能：命名参数。

提示

您可以在这里看到`match`表达式的原始提案：[`wiki.php.net/rfc/match_expression_v2`](https://wiki.php.net/rfc/match_expression_v2)

# 理解命名参数

**命名参数**代表一种避免在调用具有大量参数的函数或方法时产生混淆的方法。这不仅有助于避免参数以不正确的顺序提供的问题，还有助于您跳过具有默认值的参数。在本节中，您将学习如何应用命名参数来提高代码的准确性，减少未来维护周期中的混淆，并使您的方法和函数调用更加简洁。我们首先来看一下使用命名参数所需的通用语法。

## 命名参数通用语法

要使用命名参数，您需要知道函数或方法签名中使用的变量的名称。然后，您可以指定该名称，不带美元符号，后跟冒号和要提供的值，如下所示：

`$result = function_name( arg1 : <VALUE>, arg2 : <value>);`

当调用`function_name()`函数时，值将传递给与`arg1`、`arg2`等对应的参数。

## 使用命名参数调用核心函数

使用命名参数的最常见原因之一是调用具有大量参数的核心 PHP 函数。例如，这是`setcookie()`的函数签名：

```php
setcookie ( string $name [, string $value = "" 
    [, int $expires = 0 [, string $path = "" 
    [, string $domain = "" [, bool $secure = FALSE 
    [, bool $httponly = FALSE ]]]]]] ) : bool
```

假设您真正想要设置的只是`name`、`value`和`httponly`参数。在 PHP 8 之前，您需要查找默认值并按顺序提供它们，直到您到达要覆盖的值。在下面的情况下，我们希望将`httponly`设置为`TRUE`：

`setcookie('test',1,0,0,'','',FALSE,TRUE);`

使用命名参数，在 PHP 8 中的等效方式如下：

`setcookie('test',1,httponly: TRUE);`

请注意，我们不需要为前两个参数命名，因为它们是按顺序提供的。

提示

在 PHP 扩展中，命名参数并不总是与您在 PHP 文档中看到的函数或方法签名的变量名称匹配。例如，函数`imagettftext()`在其函数签名中显示一个变量`$font_filename`。然而，如果您再往下滚动一点，您会在*参数*部分看到，命名参数是`fontfile`。

如果遇到致命错误：`未知命名参数$NAMED_PARAM`。始终使用文档中*参数*部分列出的名称，而不是函数或方法签名中变量的名称。

## 顺序独立和文档

命名参数的另一个用途是提供**顺序独立**。此外，对于某些核心 PHP 函数来说，参数的数量之多构成了文档的噩梦。

例如，看一下`imagefttext()`的函数签名（请注意，这个函数是生成安全验证码图像的章节项目的核心）：

```php
imagefttext ( object $image , float $size , float $angle , 
    int $x , int $y , int $color , string $fontfile , 
    string $text [, array $extrainfo ] ) : array 
```

正如你可以想象的那样，在 6 个月后回顾你的工作时，试图记住这些参数的名称和顺序可能会有问题。

重要提示

在 PHP 8 中，图像创建函数（例如`imagecreate()`）现在返回一个`GdImage`对象实例，而不是一个资源。GD 扩展中的所有图像函数都已经重写以适应这一变化。没有必要重写您的代码！

因此，在 PHP 8 中，使用命名参数，以下函数调用将是可接受的：

```php
// /repo/ch01/php8_named_args.php
// not all code is shown
$rotation = range(40, -40, 10);
foreach ($rotation as $key => $offset) {
    $char->writeFill();
    [$x, $y] = RotateText::calcXYadjust($char, $offset);
    $angle = ($offset > 0) ? $offset : 360 + $offset;
    imagettftext(
        angle        : $angle,
        color        : $char->fgColor,
        font_filename : FONT_FILE,
        image        : $char->image,
        size         : 60,                
        x            : $x,
        y            : $y,
        text         : $char->text);
    $fn = IMG_DIR . '/' . $baseFn . '_' . $key . '.png';
    imagepng($char->image, $fn);
    $images[] = basename($fn);
}
```

刚才显示的代码示例将一串扭曲字符写成一组 PNG 图像文件。每个字符相对于其相邻图像顺时针旋转 10 度。请注意，命名参数的应用使`imagettftext()`函数的参数更容易理解。

命名参数也可以应用于您自己创建的函数和方法。在下一节中，我们将介绍新的数据类型。

提示

关于命名参数的详细分析可以在这里找到：

[`wiki.php.net/rfc/named_params`](https://wiki.php.net/rfc/named_params )

# 探索新的数据类型

任何初级 PHP 开发人员学到的一件事是 PHP 有哪些可用的**数据类型**以及如何使用它们。基本数据类型包括`int`（整数）、`float`、`bool`（布尔值）和`string`。复杂数据类型包括`array`和`object`。此外，还有其他数据类型，如`NULL`和`resource`。在本节中，我们将讨论 PHP 8 中引入的一些新数据类型，包括联合类型和混合类型。

重要说明

非常重要的一点是不要混淆**数据类型**和**数据格式**。本节描述了数据类型。另一方面，数据格式将是用作传输或存储的数据的*表示*方式。数据格式的示例包括 XML，**JavaScript 对象表示**（**JSON**）和**YAML 不是标记语言**（**YAML**）。

## 联合类型

与`int`或`string`等其他数据类型不同，重要的是要注意，没有一个名为*union*的数据类型。相反，当你看到**联合类型**的引用时，意思是 PHP 8 引入了一种新的语法，允许您指定多种类型，而不仅仅是一种。现在让我们来看一下联合类型的通用语法。

### 联合类型语法

联合类型的通用语法如下：

`function ( type|type|type $var) {}`

在`type`的位置，您可以提供任何现有的数据类型（例如`float`或`string`）。然而，有一些限制，大部分都是完全有道理的。这张表总结了更重要的限制：

![表 1.3 - 不允许的联合类型](img/Table_1.3_B16992.jpg)

表 1.3 - 不允许的联合类型

从这个例外列表中可以看出，定义联合类型主要是常识问题。

提示

**最佳实践**：在使用联合类型时，如果不强制执行严格类型检查，**类型强制转换**（PHP 内部转换数据类型以满足函数要求的过程）可能会成为一个问题。因此，最佳实践是在使用联合类型的任何文件顶部添加以下内容：`declare(strict_types=1);`。

有关更多信息，请参阅此处的文档参考：

[`www.php.net/manual/en/language.types.declarations.php#language.types.declarations.strict`](https://www.php.net/manual/en/language.types.declarations.php#language.types.declarations.strict )

### 联合类型示例

为了简单说明，让我们回到本章中使用的`SingleChar`类作为示例。其中的一个方法是`colorAlloc()`。该方法从图像中分配颜色，利用了`imagecolorallocate()`函数。它接受表示红色、绿色和蓝色的整数值作为参数。

为了论证，假设第一个参数实际上可以是表示三个值的数组——分别是红色、绿色和蓝色。在这种情况下，第一个值的参数类型不能是`int`，否则，如果提供了一个数组，并且打开了严格类型检查，将会抛出错误。

在 PHP 的早期版本中，唯一的解决方案是从第一个参数中删除任何类型检查，并指示在相关的 DocBlock 中接受多种类型。以下是在 PHP 7 中该方法可能的样子：

```php
/**
 * Allocates a color resource
 *
 * @param array|int $r
 * @param int $g
 * @param int $b]
 * @return int $color
 */
public function colorAlloc($r, $g = 0, $b = 0) {
    if (is_array($r)) {
        [$r, $g, $b] = $r;
    }
    return \imagecolorallocate($this->image, $r, $g, $b);
}
```

第一个参数`$r`的数据类型唯一的指示是`@param array|int $r`的 DocBlock 注释和没有与该参数关联的数据类型提示。在 PHP 8 中，利用联合类型，注意这里的区别：

```php
#[description("Allocates a color resource")]
#[param("int|array r")]
#[int("g")]
#[int("b")]
#[returns("int")]
public function colorAlloc(
    int|array $r, int $g = 0, int $b = 0) {
    if (is_array($r)) {
        [$r, $g, $b] = $r;
    }
    return \imagecolorallocate($this->image, $r, $g, $b);
}
```

在前面的示例中，除了`attribute`的存在表明第一个参数可以接受`array`或`int`类型之外，在方法签名本身中，`int|array`联合类型清楚地说明了这个选择。

## 混合类型

`mixed`是 PHP 8 中引入的另一种新类型。与联合类型不同，`mixed`是一个实际的数据类型，代表了所有类型的最终联合。它用于表示接受任何和所有数据类型。在某种意义上，PHP 已经具有了这个功能：简单地省略数据类型，它就是一个隐含的`mixed`类型！

提示

您将在 PHP 文档中看到对`mixed`类型的引用。PHP 8 通过将其作为实际数据类型来正式表示这种表示。

### 为什么使用混合类型？

等一下——你可能会想到：为什么要使用`mixed`类型呢？放心，这是一个很好的问题，没有强制使用这种类型的理由。

然而，通过在函数或方法签名中使用`mixed`，您清楚地*表明了您对该参数的使用意图。如果您只是留空数据类型，其他开发人员在以后使用或审查您的代码时可能会认为您忘记添加类型。至少，他们会对未命名参数的性质感到不确定。

## 混合类型对继承的影响

作为`mixed`类型代表**扩宽**的最终示例，它可以用于在一个类继承另一个类时*扩宽*数据类型定义。以下是使用`mixed`类型的示例，说明了这个原则：

1.  首先，我们用更严格的数据类型`object`定义父类，如下所示：

```php
// /repo/ch01/php8_mixed_type.php
declare(strict_types=1);
class High {
    const LOG_FILE = __DIR__ . '/../data/test.log';  
    protected static function logVar(object $var) {     
        $item = date('Y-m-d') . ':'
              . var_export($var, TRUE);
        return error_log($item, 3, self::LOG_FILE);
    }
}
```

1.  接下来，我们定义一个`Low`类，它继承自`High`，如下所示：

```php
class Low extends High {
    public static function logVar(mixed $var) {
        $item = date('Y-m-d') . ':'
            . var_export($var, TRUE);
        return error_log($item, 3, self::LOG_FILE);
    }
}
```

请注意，在`Low`类中，`logVar()`方法的数据类型已经*扩宽*为`mixed`。

1.  最后，我们创建了一个`Low`的实例，并用测试数据执行它。从下面的代码片段中显示的结果可以看出，一切都运行正常：

```php
if (file_exists(High::LOG_FILE)) unlink(High::LOG_FILE)
$test = [
    'array' => range('A', 'F'),
    'func' => function () { return __CLASS__; },
    'anon' => new class () { 
        public function __invoke() { 
            return __CLASS__; } },
];
foreach ($test as $item) Low::logVar($item);
readfile(High::LOG_FILE);
```

以下是前面示例的输出：

```php
2020-10-15:array (
  0 => 'A',
  1 => 'B',
  2 => 'C',
  3 => 'D',
  4 => 'E',
  5 => 'F',
)2020-10-15:Closure::__set_state(array(
))2020-10-15:class@anonymous/repo/ch01/php8_mixed_type.php:28$1::__set_state(array())
```

前面的代码块记录了各种不同的数据类型，然后显示了日志文件的内容。在这个过程中，这向我们展示了在 PHP 8 中，当子类覆盖父类方法并用`mixed`代替更严格的数据类型，如`object`时，不存在继承问题。

接下来，我们来看一下如何使用有类型的属性。

提示

**最佳实践**：在定义函数或方法时，为所有参数分配特定的数据类型。如果接受几种不同的数据类型，定义一个联合类型。否则，如果没有适用上述情况，退而使用`mixed`类型。

关于联合类型的信息，请参阅此文档页面：

[`wiki.php.net/rfc/union_types_v2`](https://wiki.php.net/rfc/union_types_v2 )

有关`mixed`类型的更多信息，请查看这里：[`wiki.php.net/rfc/mixed_type_v2.`](https://wiki.php.net/rfc/mixed_type_v2 )

# 改进使用有类型属性的代码

在本章的第一部分，*使用构造函数属性提升*，我们讨论了如何使用数据类型来控制提供给函数或类方法的参数的数据类型。然而，这种方法未能保证数据类型永远不会改变。在本节中，您将学习如何在属性级别分配数据类型，从而更严格地控制 PHP 8 中变量的使用。

## 有类型属性是什么？

这个非常重要的特性是在 PHP 7.4 中引入的，并在 PHP 8 中继续。简而言之，**有类型属性**是一个预先分配数据类型的类属性。以下是一个简单的例子：

```php
// /repo/ch01/php8_prop_type_1.php
declare(strict_types=1)
class Test {
    public int $id = 0;
    public int $token = 0;
    public string $name = '';
}
$test = new Test();
$test->id = 'ABC';
```

在这个例子中，如果我们尝试将代表`int`以外的数据类型的值分配给`$test->id`，将会抛出`Fatal error`。以下是输出：

```php
Fatal error: Uncaught TypeError: Cannot assign string to property Test::$id of type int in /repo/ch01/php8_prop_type_1.php:11 Stack trace: #0 {main} thrown in /repo/ch01/php8_prop_type_1.php on line 11 
```

如您从上面的输出中所见，当错误的数据类型分配给类型化属性时，将会抛出`Fatal error`。

您已经接触过一种属性类型化的形式：**构造函数属性提升**。使用构造函数属性提升定义的所有属性都会自动进行属性类型化！

### 为什么属性类型化很重要？

类型化属性是 PHP 中首次出现的一般趋势的一部分，该趋势是朝着限制和加强代码使用的语言细化发展。这导致更好的代码，意味着更少的错误。

以下示例说明了仅依赖属性类型提示来控制属性数据类型的危险：

```php
// /repo/ch01/php7_prop_danger.php
declare(strict_types=1);
class Test {
    protected $id = 0;
    protected $token = 0;
    protected $name = '';
    public function __construct(
        int $id, int $token, string $name) {
        $this->id = $id;
        $this->token = md5((string) $token);
        $this->name = $name;
    }
}
$test = new Test(111, 123456, 'Fred');
var_dump($test);
```

在上面的例子中，注意在`__construct()`方法中，`$token`属性被意外转换为字符串。以下是输出：

```php
object(Test)#1 (3) {
  ["id":protected]=>  int(111)
  ["token":protected]=>
  string(32) "e10adc3949ba59abbe56e057f20f883e"
  ["name":protected]=>  string(4) "Fred"
}
```

任何后续的代码如果期望`$token`是一个整数，可能会失败或产生意外的结果。现在，让我们看一下在 PHP 8 中使用类型化属性的相同情况：

```php
// /repo/ch01/php8_prop_danger.php
declare(strict_types=1);
class Test {
    protected int $id = 0;
    protected int $token = 0;
    protected string $name = '';
    public function __construct(
        int $id, int $token, string $name) {        
        $this->id = $id;
        $this->token = md5((string) $token);
        $this->name = $name;
    }
}
$test = new Test(111, 123456, 'Fred');
var_dump($test);
```

属性类型化可以防止预分配的数据类型发生任何更改，如您在此处所见的输出所示：

```php
Fatal error: Uncaught TypeError: Cannot assign string to property Test::$token of type int in /repo/ch01/php8_prop_danger.php:12
```

如您从上面的输出中所见，当错误的数据类型分配给类型化属性时，将会抛出`Fatal error`。这个例子表明，不仅将数据类型分配给属性可以防止在进行直接赋值时的误用，而且还可以防止在类方法中误用属性！

## 属性类型化可以导致代码量的减少

引入属性类型化到您的代码中的另一个有益的副作用是可能减少所需的代码量。例如，考虑当前的做法，即将属性标记为`private`或`protected`的可见性，然后创建一系列用于控制访问的`get`和`set`方法（也称为*getters*和*setters*）。

这可能如下所示：

1.  首先，我们定义一个带有受保护属性的`Test`类，如下所示：

```php
// /repo/ch01/php7_prop_reduce.php
declare(strict_types=1);
class Test {
 protected $id = 0;
 protected $token = 0;
 protected $name = '';o
```

1.  接下来，我们定义一系列用于控制对受保护属性的访问的`get`和`set`方法，如下所示：

```php
    public function getId() { return $this->id; }
    public function setId(int $id) { $this->id = $id; 
    public function getToken() { return $this->token; }
    public function setToken(int $token) {
        $this->token = $token;
    }
    public function getName() {
        return $this->name;
    }
    public function setName(string $name) {
        $this->name = $name;
    }
}
```

1.  然后，我们使用`set`方法来分配值，如下所示：

```php
$test = new Test();
$test->setId(111);
$test->setToken(999999);
$test->setName('Fred');
```

1.  最后，我们使用`get`方法以表格形式显示结果，如下所示：

```php
$pattern = '<tr><th>%s</th><td>%s</td></tr>';
echo '<table width="50%" border=1>';
printf($pattern, 'ID', $test->getId());
printf($pattern, 'Token', $test->getToken());
printf($pattern, 'Name', $test->getName());
echo '</table>';
```

这可能如下所示：

![表 1.4 - 使用 Get 方法输出](img/Figure_1.2_B16992.jpg)

表 1.4 - 使用 Get 方法输出

通过将属性标记为`protected`（或`private`）并定义*getters*和*setters*来实现的主要目的是控制访问。通常，这意味着希望阻止属性数据类型的更改。如果是这种情况，整个基础设施可以通过分配属性类型来替换。

将可见性简单地更改为`public`可以减轻对`get`和`set`方法的需求；但是，它并不能防止属性数据被更改！使用 PHP 8 属性类型既实现了这两个目标：它消除了`get`和`set`方法的需求，也防止了数据类型被意外更改。

注意在 PHP 8 中使用属性类型化实现相同结果所需的代码量大大减少了：

```php
// /repo/ch01/php8_prop_reduce.php
declare(strict_types=1);
class Test {
    public int $id = 0;
    public int $token = 0;
    public string  $name = '';
}
// assign values
$test = new Test();
$test->id = 111;
$test->token = 999999;
$test->name = 'Fred';
// display results
$pattern = '<tr><th>%s</th><td>%s</td></tr>';
echo '<table width="50%" border=1>';
printf($pattern, 'ID', $test->id);
printf($pattern, 'Token', $test->token);
printf($pattern, 'Name', $test->name);
echo '</table>';
```

上面显示的代码示例产生了与前一个示例完全相同的输出，并且还实现了对属性数据类型的更好控制。在这个例子中，使用类型化属性，我们实现了*50%的代码减少*来产生相同的结果！

提示

**最佳实践**：尽可能在可能的情况下使用类型化属性，除非您明确希望允许数据类型更改。

# 总结

在本章中，您学习了如何使用新的 PHP 8 数据类型：混合类型和联合类型来编写更好的代码。您还了解到使用命名参数不仅可以提高代码的可读性，还可以帮助防止意外误用类方法和 PHP 函数，同时提供了一个很好的方法来跳过默认参数。

本章还教会了您如何使用新的`Attribute`类作为 PHP DocBlocks 的潜在替代品，以改善代码的整体性能，同时提供了一种可靠的方式来记录类、方法和函数。

此外，我们还看到 PHP 8 如何通过利用构造函数参数提升和类型化属性大大减少了早期 PHP 版本所需的代码量。

在下一章中，您将学习有关功能和过程级别的新 PHP 8 功能。
