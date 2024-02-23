# 第一章。打下基础

在本章中，我们将涵盖以下主题：

+   PHP 7 安装注意事项

+   使用内置的 PHP Web 服务器

+   定义一个测试 MySQL 数据库

+   安装 PHPUnit

+   实现类自动加载

+   悬停在网站上

+   构建深网扫描器

+   创建一个 PHP 5 到 PHP 7 代码转换器

# 介绍

本章旨在作为一个*快速入门*，让您立即开始在 PHP 7 上运行并实施配方。本书的基本假设是您已经对 PHP 和编程有很好的了解。虽然本书不会详细介绍 PHP 的实际安装，但考虑到 PHP 7 相对较新，我们将尽力指出您在 PHP 7 安装过程中可能遇到的怪癖和*陷阱*。

# PHP 7 安装注意事项

有三种主要获取 PHP 7 的方法：

+   直接从源代码下载和安装

+   安装*预编译*二进制文件

+   安装*AMP 包（即 XAMPP，WAMP，LAMP，MAMP 等）

## 如何做...

这三种方法按难度顺序列出。然而，第一种方法虽然繁琐，但可以让您对扩展和选项有最精细的控制。

### 直接从源代码安装

为了使用这种方法，您需要有一个 C 编译器。如果您使用 Windows，**MinGW**是一个广受欢迎的免费编译器。它基于**GNU**项目提供的**GNU Compiler Collection**（**GCC**）编译器。非免费的编译器包括 Borland 的经典**Turbo C**编译器，当然，Windows 开发人员首选的编译器是**Visual Studio**。然而，后者主要设计用于 C++开发，因此在编译 PHP 时，您需要指定 C 模式。

在苹果 Mac 上工作时，最好的解决方案是安装**Apple Developer Tools**。您可以使用**Xcode IDE**编译 PHP 7，或者从终端窗口运行`gcc`。在 Linux 环境中，从终端窗口运行`gcc`。

在终端窗口或命令行编译时，正常的程序如下：

+   `configure`

+   `制作`

+   `make test`

+   `make install`

有关配置选项（即运行`configure`时）的信息，请使用`help`选项：

```php
**configure --help**

```

在配置阶段可能遇到的错误在下表中提到：

| 错误 | 修复 |
| --- | --- |
| `configure: error: xml2-config not found. Please check your libxml2 installation` | 您只需要安装`libxml2`。有关此错误，请参阅以下链接：[`superuser.com/questions/740399/how-to-fix-php-installation-when-xml2-config-is-missing`](http://superuser.com/questions/740399/how-to-fix-php-installation-when-xml2-config-is-missing) |
| `configure: error: Please reinstall readline - I cannot find readline.h` | 安装`libreadline-dev` |
| `configure: WARNING: unrecognized options: --enable-spl, --enable-reflection, --with-libxml` | 没关系。这些选项是默认的，不需要包含在内。有关更多详细信息，请参阅以下链接：[`jcutrer.com/howto/linux/how-to-compile-php7-on-ubuntu-14-04`](http://jcutrer.com/howto/linux/how-to-compile-php7-on-ubuntu-14-04) |

### 从预编译的二进制文件安装 PHP 7

正如标题所示，**预编译**二进制文件是一组由他人从 PHP 7 源代码编译而成并提供的二进制文件。

在 Windows 的情况下，转到[`windows.php.net/`](http://windows.php.net/)。您将在左栏找到一些关于选择哪个版本、**线程安全**与**非线程安全**等的提示。然后您可以点击**Downloads**并查找适用于您环境的 ZIP 文件。下载 ZIP 文件后，将文件解压到您选择的文件夹中，将`php.exe`添加到您的路径，并使用`php.ini`文件配置 PHP 7。

要在 Mac OS X 系统上安装预编译的二进制文件，最好使用包管理系统。PHP 推荐的包括以下内容：

+   MacPorts

+   Liip

+   Fink

+   Homebrew

在 Linux 的情况下，使用的打包系统取决于您使用的 Linux 发行版。以下表格按 Linux 发行版组织，总结了查找 PHP 7 包的位置。

| 分发 | PHP 7 在哪里找到 | 注释 |
| --- | --- | --- |

| Debian | `packages.debian.org/stable/php``repos-source.zend.com/zend-server/early-access/php7/php-7*DEB*` | 使用此命令：

```php
**sudo apt-get install php7**

```

或者，您可以使用图形包管理工具，如**Synaptic**。确保选择**php7**（而不是 php5）。 |

| Ubuntu | `packages.ubuntu.com``repos-source.zend.com/zend-server/early-access/php7/php-7*DEB*` | 使用此命令：`sudo apt-get install php7`确保选择正确的 Ubuntu 版本。或者，您可以使用图形包管理工具，如**Synaptic**。 |
| --- | --- | --- |

| Fedora / Red Hat | `admin.fedoraproject.org/pkgdb/packages``repos-source.zend.com/zend-server/early-access/php7/php-7*RHEL*` | 确保您是 root 用户：

```php
**su**

```

使用此命令：**dnf install php7**或者，您可以使用图形包管理工具，如 GNOME 包管理器。 |

| OpenSUSE | `software.opensuse.org/package/php7` | 使用此命令：

```php
**yast -i php7**

```

或者，您可以运行`zypper`，或者使用**YaST**作为图形工具。 |

### 安装*AMP 包

**AMP**指的是**Apache**，**MySQL**和**PHP**（还包括**Perl**和**Python**）。*****指的是 Linux、Windows、Mac 等（即 LAMP、WAMP 和 MAMP）。这种方法通常是最简单的，但是您对初始 PHP 安装的控制较少。另一方面，您可以随时修改`php.ini`文件并安装其他扩展来自定义您的安装。以下表格总结了一些流行的*AMP 包：

| Package | 找到它在哪里 | 免费？ | 支持* |
| --- | --- | --- | --- |
| `XAMPP` | [www.apachefriends.org/download.html](http://www.apachefriends.org/download.html) | Y | WML |
| `AMPPS` | [www.ampps.com/downloads](http://www.ampps.com/downloads) | Y | WML |
| `MAMP` | [www.mamp.info/en](http://www.mamp.info/en) | Y | WM |
| `WampServer` | [sourceforge.net/projects/wampserver](http://sourceforge.net/projects/wampserver) | Y | W |
| `EasyPHP` | [www.easyphp.org](http://www.easyphp.org) | Y | W |
| `Zend Server` | [www.zend.com/en/products/zend_server](http://www.zend.com/en/products/zend_server) | N | WML |

在上表中，我们列出了**W**替换为**W**的*AMP 包，**M**替换为 Mac OS X，**L**替换为 Linux。

## 还有更多...

当您从软件包安装预编译的二进制文件时，只安装了`core`扩展。非核心 PHP 扩展必须单独安装。

值得注意的是，云计算平台上的 PHP 7 安装通常会遵循预编译二进制文件的安装过程。了解您的云环境是否使用 Linux、Mac 或 Windows 虚拟机，然后按照本文中提到的适当过程进行操作。

可能 PHP 7 尚未到达您喜欢的预编译二进制文件存储库。您可以始终从源代码安装，或者考虑安装其中一个*AMP 包（请参阅下一节）。对于基于 Linux 的系统，另一种选择是使用**个人软件包存档**（**PPA**）方法。但是，由于 PPA 尚未经过严格的筛选过程，安全性可能是一个问题。有关 PPA 安全考虑的良好讨论可在[`askubuntu.com/questions/35629/are-ppas-safe-to-add-to-my-system-and-what-are-some-red-flags-to-watch-out-fo`](http://askubuntu.com/questions/35629/are-ppas-safe-to-add-to-my-system-and-what-are-some-red-flags-to-watch-out-fo)找到。

## 另请参阅

可以在[`php.net/manual/en/install.general.php`](http://php.net/manual/en/install.general.php)找到一般安装注意事项，以及针对三个主要操作系统平台（Windows、Mac OS X 和 Linux）的说明。

MinGW 的网站是[`www.mingw.org/`](http://www.mingw.org/)。

有关如何使用 Visual Studio 编译 C 程序的说明，请访问[`msdn.microsoft.com/en-us/library/bb384838`](https://msdn.microsoft.com/en-us/library/bb384838)。

测试 PHP 7 的另一种可能的方法是使用虚拟机。以下是一些工具及其链接，可能会有用：

+   **Vagrant**：[`github.com/rlerdorf/php7dev`](https://github.com/rlerdorf/php7dev)（php7dev 是一个预先配置用于测试 PHP 应用程序和在许多 PHP 版本上开发扩展的 Debian 8 Vagrant 映像）

+   **Docker**：[`hub.docker.com/r/coderstephen/php7/`](https://hub.docker.com/r/coderstephen/php7/)（其中包含一个 PHP7 Docker 容器）

# 使用内置的 PHP Web 服务器

除了单元测试和直接从命令行运行 PHP 之外，测试应用程序的明显方法是使用 Web 服务器。对于长期项目，为了开发与客户使用的 Web 服务器最接近的虚拟主机定义将是有益的。为各种 Web 服务器（如 Apache、NGINX 等）创建这样的定义超出了本书的范围。另一个快速且易于使用的替代方法（我们在这里有讨论的空间）是使用内置的 PHP 7 Web 服务器。

## 如何做...

1.  要激活 PHP Web 服务器，首先切换到将用作代码基础的目录。

1.  然后，您需要提供主机名或 IP 地址，以及可选的端口。以下是您可以使用来运行本书提供的示例的示例：

```php
    cd /path/to/recipes
    php -S localhost:8080
    ```

您将在屏幕上看到类似以下内容的输出：

![如何做...](img/B05314_01_01.jpg)

1.  随着内置的 Web 服务器继续服务请求，您还将看到访问信息、HTTP 状态代码和请求信息。

1.  如果您需要将 Web 服务器文档根目录设置为当前目录以外的目录，可以使用`-t`标志。然后，该标志必须跟随有效的目录路径。内置的 Web 服务器将把这个目录视为 Web 文档根目录，这对安全原因很有用。出于安全原因，一些框架（如 Zend Framework）要求 Web 文档根目录与实际源代码所在的位置不同。

以下是使用`-t`标志的示例：

```php
    **php -S localhost:8080 -t source/chapter01**

    ```

以下是输出的示例：

![如何做...](img/B05314_01_02.jpg)

# 定义一个测试 MySQL 数据库

为了测试目的，除了本书的源代码，我们还提供了一个带有示例数据的 SQL 文件，位于[`github.com/dbierer/php7cookbook`](https://github.com/dbierer/php7cookbook)。本书中用于示例的数据库名称是`php7cookbook`。

## 如何做...

1.  定义一个 MySQL 数据库，`php7cookbook`。还将新数据库的权限分配给名为`cook`的用户，密码为`book`。以下表总结了这些设置：

| 项目 | 注释 |
| --- | --- |
| 数据库名称 | `php7cookbook` |
| 数据库用户 | `cook` |
| 数据库用户密码 | `book` |

1.  以下是创建数据库所需的 SQL 示例：

```php
    CREATE DATABASE IF NOT EXISTS dbname DEFAULT CHARACTER SET utf8 COLLATE utf8_general_ci;
    CREATE USER 'user'@'%' IDENTIFIED WITH mysql_native_password;
    SET PASSWORD FOR 'user'@'%' = PASSWORD('userPassword');
    GRANT ALL PRIVILEGES ON dbname.* to 'user'@'%';
    GRANT ALL PRIVILEGES ON dbname.* to 'user'@'localhost';
    FLUSH PRIVILEGES;
    ```

1.  将示例值导入新数据库。导入文件`php7cookbook.sql`位于[`github.com/dbierer/php7cookbook/blob/master/php7cookbook.sql`](https://github.com/dbierer/php7cookbook/blob/master/php7cookbook.sql)。

# 安装 PHPUnit

单元测试可以说是测试 PHP 代码的最流行方式。大多数开发人员都会同意，一个完善的测试套件是任何正确开发项目的必备条件。但是很少有开发人员实际编写这些测试。幸运的是，有一些独立的测试组为他们编写测试！然而，经过数月与测试组的战斗后，幸运的人往往会抱怨和抱怨。无论如何，任何一本关于 PHP 的书都不会完整，如果没有至少对测试的一点点提及。

找到**PHPUnit**的最新版本的地方是[`phpunit.de/`](https://phpunit.de/)。PHPUnit5.1 及以上版本支持 PHP 7。单击所需版本的链接，然后下载`phpunit.phar`文件。然后可以使用存档执行命令，如下所示：

```php
**php phpunit.phar <command>**

```

### 提示

`phar`命令代表**PHP Archive**。这项技术基于`tar`，`tar`本身是在 UNIX 中使用的。`phar`文件是一组 PHP 文件，它们被打包到一个单个文件中以方便使用。

# 实现类自动加载

在使用**面向对象编程**（**OOP**）方法开发 PHP 时，建议将每个类放在自己的文件中。遵循这个建议的好处是长期维护和提高可读性的便利。缺点是每个类定义文件必须被包含（即使用`include`或其变体）。为了解决这个问题，PHP 语言内置了一个机制，可以*自动加载*任何尚未被特别包含的类。

## 准备工作

PHP 自动加载的最低要求是定义一个全局的`__autoload()`函数。这是一个*魔术*函数，当 PHP 引擎自动调用时，会请求一个类，但该类尚未被包含。请求的类的名称将在调用`__autoload()`时作为参数出现（假设您已经定义了它！）。如果您使用 PHP 命名空间，将传递类的完整命名空间名称。因为`__autoload()`是一个*函数*，它必须在全局命名空间中；但是，对其使用有限制。因此，在本篇中，我们将使用`spl_autoload_register()`函数，这给了我们更多的灵活性。

## 操作方法...

1.  我们将在本篇中介绍的类是`Application\Autoload\Loader`。为了利用 PHP 命名空间和自动加载之间的关系，我们将文件命名为`Loader.php`，并将其放置在`/path/to/cookbook/files/Application/Autoload`文件夹中。

1.  我们将介绍的第一种方法是简单地加载一个文件。我们使用`file_exists()`在运行`require_once()`之前进行检查。这样做的原因是，如果文件未找到，`require_once()`将生成一个无法使用 PHP 7 的新错误处理功能捕获的致命错误：

```php
    protected static function loadFile($file)
    {
        if (file_exists($file)) {
            require_once $file;
            return TRUE;
        }
        return FALSE;
    }
    ```

1.  然后我们可以在调用程序中测试`loadFile()`的返回值，并在无法加载文件时抛出`Exception`之前循环遍历备用目录列表。

### 提示

您会注意到这个类中的方法和属性都是静态的。这使我们在注册自动加载方法时更加灵活，并且还可以将`Loader`类视为**单例**。

1.  接下来，我们定义调用`loadFile()`并实际执行基于命名空间类名定位文件的逻辑的方法。该方法通过将 PHP 命名空间分隔符`\`转换为适合该服务器的目录分隔符并附加`.php`来派生文件名：

```php
    public static function autoLoad($class)
    {
        $success = FALSE;
        $fn = str_replace('\\', DIRECTORY_SEPARATOR, $class) 
              . '.php';
        foreach (self::$dirs as $start) {
            $file = $start . DIRECTORY_SEPARATOR . $fn;
            if (self::loadFile($file)) {
                $success = TRUE;
                break;
            }
        }
        if (!$success) {
            if (!self::loadFile(__DIR__ 
                . DIRECTORY_SEPARATOR . $fn)) {
                throw new \Exception(
                    self::UNABLE_TO_LOAD . ' ' . $class);
            }
        }
        return $success;
    }
    ```

1.  接下来，该方法循环遍历我们称之为`self::$dirs`的目录数组，使用每个目录作为派生文件名的起点。如果不成功，作为最后的手段，该方法尝试从当前目录加载文件。如果甚至这样也不成功，就会抛出一个`Exception`。

1.  接下来，我们需要一个可以将更多目录添加到我们要测试的目录列表中的方法。请注意，如果提供的值是一个数组，则使用`array_merge()`。否则，我们只需将目录字符串添加到`self::$dirs`数组中：

```php
    public static function addDirs($dirs)
    {
        if (is_array($dirs)) {
            self::$dirs = array_merge(self::$dirs, $dirs);
        } else {
            self::$dirs[] = $dirs;
        }
    }  
    ```

1.  然后，我们来到最重要的部分；我们需要将我们的`autoload()`方法注册为**标准 PHP 库**（**SPL**）自动加载程序。这是使用`spl_autoload_register()`和`init()`方法来实现的：

```php
    public static function init($dirs = array())
    {
        if ($dirs) {
            self::addDirs($dirs);
        }
        if (self::$registered == 0) {
            spl_autoload_register(__CLASS__ . '::autoload');
            self::$registered++;
        }
    }
    ```

1.  此时，我们可以定义`__construct()`，它调用`self::init($dirs)`。这使我们也可以创建`Loader`的实例（如果需要的话）。

```php
    public function __construct($dirs = array())
    {
        self::init($dirs);
    }
    ```

## 它是如何工作的...

为了使用我们刚刚定义的自动加载程序类，您需要`require Loader.php`。如果您的命名空间文件位于当前目录之外的目录中，您还应该运行`Loader::init()`并提供额外的目录路径。

为了确保自动加载程序正常工作，我们还需要一个测试类。这是`/path/to/cookbook/files/Application/Test/TestClass.php`的定义：

```php
<?php
namespace Application\Test;
class TestClass
{
    public function getTest()
    {
        return __METHOD__;
    }
}
```

现在创建一个样本`chap_01_autoload_test.php`代码文件来测试自动加载程序：

```php
<?php
require __DIR__ . '/../Application/Autoload/Loader.php';
Application\Autoload\Loader::init(__DIR__ . '/..');
```

接下来，获取一个尚未加载的类的实例：

```php
$test = new Application\Test\TestClass();
echo $test->getTest();
```

最后，尝试获取一个不存在的`fake`类。请注意，这将引发错误：

```php
$fake = new Application\Test\FakeClass();
echo $fake->getTest();
```

# 清理网站

经常有兴趣扫描网站并从特定标签中提取信息。这种基本机制可以用来在网络中搜索有用的信息。有时需要获取`<IMG>`标签和`SRC`属性的列表，或者`<A>`标签和相应的`HREF`属性。可能性是无限的。

## 如何做...

1.  首先，我们需要获取目标网站的内容。乍一看，似乎我们应该发出 cURL 请求，或者简单地使用`file_get_contents()`。这些方法的问题是，我们最终将不得不进行大量的字符串操作，很可能不得不大量使用可怕的正则表达式。为了避免所有这些，我们将简单地利用已经存在的 PHP 7 类`DOMDocument`。因此，我们创建一个`DOMDocument`实例，将其设置为**UTF-8**。我们不关心空格，并使用方便的`loadHTMLFile()`方法将网站的内容加载到对象中：

```php
    public function getContent($url)
    {
        if (!$this->content) {
            if (stripos($url, 'http') !== 0) {
                $url = 'http://' . $url;
            }
            $this->content = new DOMDocument('1.0', 'utf-8');
            $this->content->preserveWhiteSpace = FALSE;
            // @ used to suppress warnings generated from // improperly configured web pages
            @$this->content->loadHTMLFile($url);
        }
        return $this->content;
    }
    ```

### 提示

请注意，在调用`loadHTMLFile()`方法之前，我们在其前面加上了`@`。这不是为了掩盖糟糕的编码（`!`），这在 PHP 5 中经常发生！相反，`@`抑制了解析器在遇到编写不良的 HTML 时生成的通知。据推测，我们可以捕获通知并记录它们，可能还给我们的`Hoover`类提供诊断能力。

1.  接下来，我们需要提取感兴趣的标签。我们使用`getElementsByTagName()`方法来实现这个目的。如果我们希望提取*所有*标签，我们可以提供`*`作为参数：

```php
    public function getTags($url, $tag)
    {
        $count    = 0;
        $result   = array();
        $elements = $this->getContent($url)
                         ->getElementsByTagName($tag);
        foreach ($elements as $node) {
            $result[$count]['value'] = trim(preg_replace('/\s+/', ' ', $node->nodeValue));
            if ($node->hasAttributes()) {
                foreach ($node->attributes as $name => $attr) 
                {
                    $result[$count]['attributes'][$name] = 
                        $attr->value;
                }
            }
            $count++;
        }
        return $result;
    }
    ```

1.  提取特定属性而不是标签可能也是有趣的。因此，我们为此定义另一个方法。在这种情况下，我们需要遍历所有标签并使用`getAttribute()`。您会注意到有一个用于 DNS 域的参数。我们添加了这个参数，以便在同一个域内保持扫描（例如，如果您正在构建一个网页树）：

```php
    public function getAttribute($url, $attr, $domain = NULL)
    {
        $result   = array();
        $elements = $this->getContent($url)
                         ->getElementsByTagName('*');
        foreach ($elements as $node) {
            if ($node->hasAttribute($attr)) {
                $value = $node->getAttribute($attr);
                if ($domain) {
                    if (stripos($value, $domain) !== FALSE) {
                        $result[] = trim($value);
                    }
                } else {
                    $result[] = trim($value);
                }
            }
        }
        return $result;
    }
    ```

## 它是如何工作的...

为了使用新的`Hoover`类，初始化自动加载程序（如前所述）并创建`Hoover`类的实例。然后可以运行`Hoover::getTags()`方法，以产生您指定为参数的 URL 的标签数组。

这是来自`chap_01_vacuuming_website.php`的一段代码，它使用`Hoover`类来扫描 O'Reilly 网站的`<A>`标签：

```php
<?php
// modify as needed
define('DEFAULT_URL', 'http://oreilly.com/');
define('DEFAULT_TAG', 'a');

require __DIR__ . '/../Application/Autoload/Loader.php';
Application\Autoload\Loader::init(__DIR__ . '/..');

// get "vacuum" class
$vac = new Application\Web\Hoover();

// NOTE: the PHP 7 null coalesce operator is used
$url = strip_tags($_GET['url'] ?? DEFAULT_URL);
$tag = strip_tags($_GET['tag'] ?? DEFAULT_TAG);

echo 'Dump of Tags: ' . PHP_EOL;
var_dump($vac->getTags($url, $tag));
```

输出将看起来像这样：

![它是如何工作的...](img/B05314_01_03.jpg)

## 另请参阅

有关 DOM 的更多信息，请参阅 PHP 参考页面[`php.net/manual/en/class.domdocument.php`](http://php.net/manual/en/class.domdocument.php)。

# 构建深层网络扫描器

有时您需要扫描一个网站，但要深入一级。例如，您想要构建一个网站的 Web 树图。这可以通过查找所有`<A>`标签并跟踪`HREF`属性到下一个网页来实现。一旦您获得了子页面，您可以继续扫描以完成树。

## 如何做...

1.  深层网络扫描仪的核心组件是一个基本的`Hoover`类，如前所述。本配方中介绍的基本过程是扫描目标网站并清理所有`HREF`属性。为此，我们定义了一个`Application\Web\Deep`类。我们添加一个表示 DNS 域的属性：

```php
    namespace Application\Web;
    class Deep
    {
        protected $domain;
    ```

1.  接下来，我们定义一个方法，将为扫描列表中表示的每个网站的标签进行清理。为了防止扫描器在整个**万维网**（**WWW**）上进行搜索，我们将扫描限制在目标域上。添加`yield from`的原因是因为我们需要产生`Hoover::getTags()`生成的整个数组。`yield from`语法允许我们将数组视为子生成器：

```php
    public function scan($url, $tag)
    {
        $vac    = new Hoover();
        $scan   = $vac->getAttribute($url, 'href', 
           $this->getDomain($url));
        $result = array();
        foreach ($scan as $subSite) {
            yield from $vac->getTags($subSite, $tag);
        }
        return count($scan);
    }
    ```

### 注意

使用`yield from`将`scan()`方法转换为 PHP 7 委托生成器。通常，您会倾向于将扫描结果存储在数组中。然而，在这种情况下，检索到的信息量可能会非常庞大。因此，最好立即产生结果，以节省内存并产生即时结果。否则，将会有一个漫长的等待，可能会导致内存不足错误。

1.  为了保持在同一个域中，我们需要一个方法，将从 URL 中返回域。我们使用方便的`parse_url()`函数来实现这个目的：

```php
    public function getDomain($url)
    {
        if (!$this->domain) {
            $this->domain = parse_url($url, PHP_URL_HOST);
        }
        return $this->domain;
    }
    ```

## 它是如何工作的...

首先，继续定义之前定义的`Application\Web\Deep`类，以及前一个配方中定义的`Application\Web\Hoover`类。

接下来，定义一个代码块，来自`chap_01_deep_scan_website.php`，设置自动加载（如本章前面描述的）：

```php
<?php
// modify as needed
define('DEFAULT_URL', unlikelysource.com');
define('DEFAULT_TAG', 'img');

require __DIR__ . '/../../Application/Autoload/Loader.php';
Application\Autoload\Loader::init(__DIR__ . '/../..');
```

接下来，获取我们新类的一个实例：

```php
$deep = new Application\Web\Deep();
```

在这一点上，您可以从 URL 参数中检索 URL 和标签信息。PHP 7 的`null coalesce`运算符对于建立回退值非常有用：

```php
$url = strip_tags($_GET['url'] ?? DEFAULT_URL);
$tag = strip_tags($_GET['tag'] ?? DEFAULT_TAG);
```

一些简单的 HTML 将显示结果：

```php
foreach ($deep->scan($url, $tag) as $item) {
    $src = $item['attributes']['src'] ?? NULL;
    if ($src && (stripos($src, 'png') || stripos($src, 'jpg'))) {
        printf('<br><img src="%s"/>', $src);
    }
}
```

## 另请参阅

有关生成器和`yield from`的更多信息，请参阅[`php.net/manual/en/language.generators.syntax.php`](http://php.net/manual/en/language.generators.syntax.php)上的文章。

# 创建一个 PHP 5 到 PHP 7 代码转换器

在大多数情况下，PHP 5.x 代码可以在 PHP 7 上不经修改地运行。然而，有一些更改被归类为*向后不兼容*。这意味着，如果您的 PHP 5 代码以某种方式编写，或者使用了已删除的函数，您的代码将会出错，您将会遇到一个令人讨厌的错误。

## 准备工作

*PHP 5 到 PHP 7 代码转换器*执行两项任务：

+   扫描您的代码文件，并将已删除的 PHP 5 功能转换为 PHP 7 中的等效功能

+   在更改语言使用的地方添加了`//` `WARNING`注释，但不可能进行重写

### 注意

请注意，在运行转换器之后，不能保证您的代码在 PHP 7 中能够正常工作。您仍然需要查看添加的`//` `WARNING`标签。至少，这个方法将为您提供一个很好的起点，将您的 PHP 5 代码转换为在 PHP 7 中运行。

这个方法的核心是新的 PHP 7 `preg_replace_callback_array()`函数。这个神奇的函数允许您将一系列正则表达式作为键呈现，并将值表示为独立的回调。然后，您可以通过一系列转换来传递字符串。不仅如此，回调数组的主题本身也可以是一个数组。

## 如何做...

1.  在一个新的类`Application\Parse\Convert`中，我们从一个`scan()`方法开始，该方法接受一个文件名作为参数。它检查文件是否存在。如果存在，它调用 PHP 的`file()`函数，该函数将文件加载到一个数组中，其中每个数组元素代表一行：

```php
    public function scan($filename)
    {
        if (!file_exists($filename)) {
            throw new Exception(
                self::EXCEPTION_FILE_NOT_EXISTS);
        }
        $contents = file($filename);
        echo 'Processing: ' . $filename . PHP_EOL;

        $result = preg_replace_callback_array( [
    ```

1.  接下来，我们开始传递一系列键/值对。键是一个正则表达式，它针对字符串进行处理。任何匹配项都会传递给回调函数，该回调函数表示为键/值对的值部分。我们检查已从 PHP 7 中删除的开放和关闭标签：

```php
        // replace no-longer-supported opening tags
        '!^\<\%(\n| )!' =>
            function ($match) {
                return '<?php' . $match[1];
            },

        // replace no-longer-supported opening tags
        '!^\<\%=(\n| )!' =>
            function ($match) {
                return '<?php echo ' . $match[1];
            },

        // replace no-longer-supported closing tag
        '!\%\>!' =>
            function ($match) {
                return '?>';
            },
    ```

1.  接下来是一系列警告，当检测到某些操作并且在 PHP 5 与 PHP 7 中处理它们之间存在潜在的代码中断时。在所有这些情况下，代码都不会被重写。而是添加了一个带有`WARNING`单词的内联注释：

```php
        // changes in how $$xxx interpretation is handled
        '!(.*?)\$\$!' =>
            function ($match) {
                return '// WARNING: variable interpolation 
                       . ' now occurs left-to-right' . PHP_EOL
                       . '// see: http://php.net/manual/en/'
                       . '// migration70.incompatible.php'
                       . $match[0];
            },

        // changes in how the list() operator is handled
        '!(.*?)list(\s*?)?\(!' =>
            function ($match) {
                return '// WARNING: changes have been made '
                       . 'in list() operator handling.'
                       . 'See: http://php.net/manual/en/'
                       . 'migration70.incompatible.php'
                       . $match[0];
            },

        // instances of \u{
        '!(.*?)\\\u\{!' =>
            function ($match) {
            return '// WARNING: \\u{xxx} is now considered '
                   . 'unicode escape syntax' . PHP_EOL
                   . '// see: http://php.net/manual/en/'
                   . 'migration70.new-features.php'
                   . '#migration70.new-features.unicode-'
                   . 'codepoint-escape-syntax' . PHP_EOL
                   . $match[0];
        },

        // relying upon set_error_handler()
        '!(.*?)set_error_handler(\s*?)?.*\(!' =>
            function ($match) {
                return '// WARNING: might not '
                       . 'catch all errors'
                       . '// see: http://php.net/manual/en/'
                       . '// language.errors.php7.php'
                       . $match[0];
            },

        // session_set_save_handler(xxx)
        '!(.*?)session_set_save_handler(\s*?)?\((.*?)\)!' =>
            function ($match) {
                if (isset($match[3])) {
                    return '// WARNING: a bug introduced in'
                           . 'PHP 5.4 which '
                           . 'affects the handler assigned by '
                           . 'session_set_save_handler() and '
                           . 'where ignore_user_abort() is TRUE 
                           . 'has been fixed in PHP 7.'
                           . 'This could potentially break '
                           . 'your code under '
                           . 'certain circumstances.' . PHP_EOL
                           . 'See: http://php.net/manual/en/'
                           . 'migration70.incompatible.php'
                           . $match[0];
                } else {
                    return $match[0];
                }
            },
    ```

1.  任何尝试使用`<<`或`>>`与负操作符或超过 64 的操作都会被包裹在`try { xxx } catch() { xxx }`块中，寻找`ArithmeticError`的抛出：

```php
        // wraps bit shift operations in try / catch
        '!^(.*?)(\d+\s*(\<\<|\>\>)\s*-?\d+)(.*?)$!' =>
            function ($match) {
                return '// WARNING: negative and '
                       . 'out-of-range bitwise '
                       . 'shift operations will now 
                       . 'throw an ArithmeticError' . PHP_EOL
                       . 'See: http://php.net/manual/en/'
                       . 'migration70.incompatible.php'
                       . 'try {' . PHP_EOL
                       . "\t" . $match[0] . PHP_EOL
                       . '} catch (\\ArithmeticError $e) {'
                       . "\t" . 'error_log("File:" 
                       . $e->getFile() 
                       . " Message:" . $e->getMessage());'
                       . '}' . PHP_EOL;
            },
    ```

### 注意

PHP 7 已更改了错误处理方式。在某些情况下，错误被移动到与异常类似的分类中，并且可以被捕获！`Error`类和`Exception`类都实现了`Throwable`接口。如果要捕获`Error`或`Exception`，请捕获`Throwable`。

1.  接下来，转换器会重写任何使用`call_user_method*()`的用法，这在 PHP 7 中已被移除。这些将被替换为使用`call_user_func*()`的等效用法：

```php
        // replaces "call_user_method()" with
        // "call_user_func()"
        '!call_user_method\((.*?),(.*?)(,.*?)\)(\b|;)!' =>
            function ($match) {
                $params = $match[3] ?? '';
                return '// WARNING: call_user_method() has '
                          . 'been removed from PHP 7' . PHP_EOL
                          . 'call_user_func(['. trim($match[2]) . ',' 
                          . trim($match[1]) . ']' . $params . ');';
            },

        // replaces "call_user_method_array()" 
        // with "call_user_func_array()"
        '!call_user_method_array\((.*?),(.*?),(.*?)\)(\b|;)!' =>
            function ($match) {
                return '// WARNING: call_user_method_array()'
                       . 'has been removed from PHP 7'
                       . PHP_EOL
                       . 'call_user_func_array([' 
                       . trim($match[2]) . ',' 
                       . trim($match[1]) . '], ' 
                       . $match[3] . ');';
            },
    ```

1.  最后，任何尝试使用带有`/e`修饰符的`preg_replace()`都会被重写为使用`preg_replace_callback()`：

```php
         '!^(.*?)preg_replace.*?/e(.*?)$!' =>
        function ($match) {
            $last = strrchr($match[2], ',');
            $arg2 = substr($match[2], 2, -1 * (strlen($last)));
            $arg1 = substr($match[0], 
                           strlen($match[1]) + 12, 
                           -1 * (strlen($arg2) + strlen($last)));
             $arg1 = trim($arg1, '(');
             $arg1 = str_replace('/e', '/', $arg1);
             $arg3 = '// WARNING: preg_replace() "/e" modifier 
                       . 'has been removed from PHP 7'
                       . PHP_EOL
                       . $match[1]
                       . 'preg_replace_callback('
                       . $arg1
                       . 'function ($m) { return ' 
                       .    str_replace('$1','$m', $match[1]) 
                       .      trim($arg2, '"\'') . '; }, '
                       .      trim($last, ',');
             return str_replace('$1', '$m', $arg3);
        },

            // end array
            ],

            // this is the target of the transformations
            $contents
        );
        // return the result as a string
        return implode('', $result);
    }
    ```

## 工作原理...

要使用转换器，请从命令行运行以下代码。您需要提供要作为参数扫描的 PHP 5 代码的文件名。

这段代码块`chap_01_php5_to_php7_code_converter.php`，从命令行运行，调用转换器：

```php
<?php
// get filename to scan from command line
$filename = $argv[1] ?? '';

if (!$filename) {
    echo 'No filename provided' . PHP_EOL;
    echo 'Usage: ' . PHP_EOL;
    echo __FILE__ . ' <filename>' . PHP_EOL;
    exit;
}

// setup class autoloading
require __DIR__ . '/../Application/Autoload/Loader.php';

// add current directory to the path
Application\Autoload\Loader::init(__DIR__ . '/..');

// get "deep scan" class
$convert = new Application\Parse\Convert();
echo $convert->scan($filename);
echo PHP_EOL;
```

## 另请参阅

有关不兼容的更多信息，请参考[`php.net/manual/en/migration70.incompatible.php`](http://php.net/manual/en/migration70.incompatible.php)。
