# 第十一章：SEO、缓存和记录

在本章中，你将学习：

+   在 CodeIgniter 中使用 SEO 友好的 URL

+   使用 CodeIgniter 缓存

+   使用 CodeIgniter 记录错误

+   对你的应用程序进行基准测试

# 简介

在本章中，我们将查看各种可用于 CodeIgniter 的缓存配方。我们将使用 CodeIgniter 的缓存功能来帮助我们存储数据源和数据库结果——尽管数据源和数据库结果实际上只是作为向你展示如何将数据输入到 CodeIgniter 中，以便它可以进行缓存的一种手段——数据输入可以是任何你希望的内容。我们还将探讨使用 CodeIgniter 路由实现 SEO 友好 URL 的方法。

# 在 CodeIgniter 中使用 SEO 友好的 URL

在某个时候，你可能想要改变 CodeIgniter 处理 URL 到控制器路由的方式。默认情况下，CodeIgniter 将 URL 分割成几个不同的部分。显然，URL 中有域名部分（`www.domain.com`），但之后通常（但不总是）还有三个更多项目，每个项目之间由正斜杠分隔。第一个项目是控制器，第二个是控制器中的函数（或方法，如果你愿意），第三个是你将传递给控制器中函数的参数。

因此，CodeIgniter 中的一个标准 URL 可能看起来像`www.domain.com/users/edit/1`。所以，用户编号`1`正在使用`users`控制器中的`edit`函数进行编辑——这看起来很简单，我相信你很熟悉它。

然而，可能会有时候你希望它改变。在`config/routes.php`文件中设置一个规则，可以将 URL 映射到控制器，但隐藏在地址栏中显示的内容；例如，你可能有一个名为`bulletin`的控制器，你希望它在 URL 中显示为`news`。

## 如何操作...

我们将创建一个文件，并修改另一个文件，通过以下步骤进行：

1.  创建`/path/to/codeigniter/application/controllers/shop.php`文件，并将其中的以下代码添加到该文件中：

    ```php
    <?php if ( ! defined('BASEPATH')) exit('No direct script access allowed');

    class Shop extends CI_Controller {
      function __construct() {
        parent::__construct();
      }

      public function index() {
        echo 'Controller: ' . __CLASS__ . ', Method: ' . __FUNCTION__;
      }

      public function product() {
        echo 'Controller: ' . __CLASS__ . ', Method: ' . __FUNCTION__;
        echo '<br />';
        echo 'Product ID: ' . $this->uri->segment(2);
      }

      public function all() {
        echo 'Controller: ' . __CLASS__ . ', Method: ' . __FUNCTION__;
      }
    }
    ```

1.  在你的编辑器中打开`/path/to/codeigniter/config/routes.php`文件，并相应地进行修改（更改已突出显示）：

    ```php
    $route['default_controller'] = "default_controller";
    $route['404_override'] = '';

    $route['item/(:any)'] = "shop/product";
    $route['item'] = "shop/all";

    ```

## 它是如何工作的...

查看以下`config/routes.php`文件中的以下行：

```php
$route['item/(:any)'] = "shop/product";
$route['item'] = "shop/all";
```

现在，想象它们由左右两部分组成，即在`=`符号之前是左部分，在`=`符号之后是右部分；左部分映射到右部分，左部分的值将映射到右部分的值。

在前面的例子中，任何以`item`开头的控制器名称的 URL 都将映射到`shop`控制器。让我们逐一查看每个规则：

通过在浏览器中输入路由的名称（例如，`http://www.your_web_site_name.com/item`），`item` 命令将导致 CodeIgniter 调用 `shop` 控制器，并在该控制器中调用 `public function all()` 方法。因为我们已经在路由文件中定义了它，所以你可以在浏览器中通过输入 `item` 到 URL 中看到这一点。CodeIgniter 将请求的 URL 映射到路由规则中定义的路径，并且（在我们的例子中）调用 `shop` 控制器，它使用 `__CLASS__` 和 `__FUNCTION__` 将以下输出写入浏览器：

![如何工作...](img/2308OS_11_01.jpg)

现在，看看浏览器地址栏中的 URL，它仍然会显示 `item`，但屏幕上的文本说正在调用的是 `shop` 控制器。我们现在已经将 URL 映射到控制器，而用户却毫无察觉。

现在，让我们看看其他的路由规则--`:any`。通过在浏览器中输入路由的名称（例如，`http://www.your_web_site_name.com/item/123456`），`item/123456` 命令将导致 CodeIgniter 调用 `shop` 控制器和 `public function product()`，因为我们已经在路由文件中定义了它。在 `public function product()` 中，我们不仅输出了 `__CLASS__` 和 `__FUNCTION__` 名称，还输出了 URI 的第二个部分：

```php
$this->uri->segment(2);
```

在这种情况下，是产品的 ID (`12356`)，所以你将在浏览器中看到以下截图：

![如何工作...](img/2308OS_11_02.jpg)

这条路由的关键在于路由映射规则中的 `(:any)` 标志；`(:any)` 告诉 CodeIgniter，URI 的第二个部分，无论是什么，都应该映射到 `shop` 控制器的 `product` 函数。因为我们是在编写代码，所以我们会知道 URL 的第二个 URI 段是产品 ID，我们可以用它来查询数据库以获取该产品的详细信息。

# 使用 CodeIgniter 缓存

你可以使用 CodeIgniter 缓存来缓存（或临时存储）几乎任何东西。作为一个使用 CodeIgniter 的缓存示例，我们将缓存一个 RSS 源。我们当然可以缓存我们想要的任何东西；然而，缓存一个 RSS 源是一个好的开始。处理 RSS 非常简单，这个配方可以很容易地转换为缓存来自其他来源的源，例如调用 Twitter 等。

## 如何操作...

我们将创建 `/path/to/codeigniter/application/controllers/rss_cache.php` 文件。

1.  创建前面的文件，并向其中添加以下代码：

    ```php
    <?php if ( ! defined('BASEPATH')) exit('No direct script access allowed');

    class Rss_cache extends CI_Controller {
      function __construct() {
        parent::__construct();
        $this->load->helper('url');
        $this->load->helper('xml');
        $this->load->driver('cache', array('adapter' => 'apc'));
      }

      public function index() {
        $raw_feed = '<?xml version = "1.0" encoding = "ISO-8859-1" ?>
        <rss version="2.0">
        <channel>
          <title>RSS Feed</title>
            <link>http://www.domain.com</link>
            <description>General Description</description>
          <item>
            <title>RSS Item 1 Title</title>
            <link>http://www.domain1.com/link1</link>
            <description>Description of First Item</description>
          </item>
          <item>
            <title>RSS Item 2 Title</title>
            <link>http://www.domain2.com/link2</link>
            <description>Description of Second Item</description>
          </item>
          <item>
            <title>RSS Item 3 Title</title>
            <link>http://www.domain3.com/link3</link>
            <description>Description of Third Item</description>
          </item>
        </channel>
        </rss>';

        $feed = new SimpleXmlElement($raw_feed);

        if (!$cached_feed = $this->cache->get('rss')) {
          foreach ($feed->channel->item as $item) {
            $cached_feed . = $item->title . '<br />' . $item->description . '<br /><br />';
          }

            $this->cache->save('rss', $cached_feed, 7);
        }

        echo $this->cache->get('rss');

      }

      public function clear_cache() {
        $this->cache->clean();
        redirect('rss_cache');
      }
    }
    ?>
    ```

## 它是如何工作的...

首先，让我们看看构造函数。我们正在加载一个助手和一个驱动程序，如下面的代码片段所示：

```php
function __construct() {
  parent::__construct();
  $this->load->helper('url');
  $this->load->driver('cache', array('adapter' => 'apc'));
}
```

### 小贴士

要使用 APC，你需要确保你正在工作的环境中安装了 APC。如果没有，你需要安装它。为此，请访问 [`www.php.net/manual/en/apc.setup.php`](http://www.php.net/manual/en/apc.setup.php) 获取更多详细信息。

由于我们使用了`redirect()`函数，所以存在 URL 辅助器；缓存驱动器用于提供支持以帮助我们缓存数据，在这种情况下，它帮助我们缓存来自 RSS 源的数据--然而，它实际上可以是任何东西。

`public function index()`首先使用硬编码的 RSS 源定义了`$rss_feed`变量；这只是为了说明。实际上，你将使用以下方式获取源：

```php
$raw_feed = file_get_contents('http://www.domain.com/rss_feed_url');
```

然而，在这个菜谱中硬编码它是方便的，因为通过硬编码，你可以看到源的结构，并知道它应该“看起来”是什么样子

该源具有简单的结构，只包含三个项目。`$raw_feed`变量被传递给 PHP 的`SimpleXMLElement`类，它为我们返回一个对象（`$feed`），我们可以用它来工作。

然后，我们使用 CodeIgniter 检查缓存中是否存在名为`rss`的项；如果没有，我们将遍历`$feed`对象，从 RSS 源中的每个项目提取标题和描述，并将它们连接成一个字符串，该字符串命名为`$cached_feed`。

```php
if (!$cached_feed = $this->cache->get('rss')) {
  foreach ($feed->channel->item as $item) {
    $cached_feed .= $item->title . '<br />' . $item->description . '<br /><br />';
  }
    $this->cache->save('rss', $cached_feed, 30);
}
```

`$cached_feed`字符串被保存到缓存中，名称为`rss`，持续时间为 30 秒（有关缓存持续时间的更多信息，请参阅以下代码）：

```php
$this->cache->save('rss', $cached_feed, 30);
```

一旦 RSS 源中的所有项目都已被处理，我们将输出缓存项`rss`，如下代码片段所示：

```php
echo $this->cache->get('rss');
```

你应该在浏览器中看到以下类似的输出：

```php
RSS Item 1 Title
Description of First Item
RSS Item 2 Title
Description of Second Item
RSS Item 3 Title
Description of Third Item
```

为什么是 30 秒？因为这将是足够的时间让你去浏览器运行脚本，查看前面的输出，然后快速回到你的文本编辑器中的代码，更改源中的某些内容（例如第三项的标题元素改为`Gigantic Elephants`）。点击**保存**，然后回到浏览器刷新，这应该在第一次运行`rss_cache`后的 30 秒（30 秒）给你以下输出：

```php
RSS Item 1 Title
Description of First Item
RSS Item 2 Title
Description of Second Item
Gigantic Elephants
Description of Third Item
```

如果你觉得 30 秒的等待时间太长，你可以通过运行`public function clear_cache()`手动清除缓存，这将调用 CodeIgniter 的`$this->cache->clean()`函数，并将我们重定向回`rss_cache`控制器，整个过程将重新开始。

或者，你可以将缓存项有效的时间长度从 30 秒减少到，比如说，10 秒（但现实中，你可能希望它是一个适合你服务器或数据的时间长度）。

因此，现在你可以看到如何在缓存中存储数据，它不一定是 XML 或 RSS 源，它实际上可以是来自任何来源的数据：数据库查询（尽管 CodeIgniter 为此提供了特定的数据库缓存方法），来自社交链接的源（例如 Twitter 源或 Instagram），甚至是每 5 分钟获取一次的金融数据，如 FTSE 的价值（或者你设置的任何时长）。

### 你可能遇到的问题

如果您在 MAC 上使用 MAMP 进行开发，那么默认的缓存方法很可能是 XCache。CodeIgniter 没有与 XCache 一起工作的驱动程序，您可能需要编写自己的驱动程序（勇敢地去做），或者（像我一样）将缓存引擎更改为 APC。

### 小贴士

**APC**（**替代 PHP 缓存**）是一种缓存服务，在这种情况下由 MAMP 提供。

您需要告诉 MAMP 使用 APC 而不是 XCache。为此，打开您的 MAMP 控制面板，点击**偏好设置**，然后点击**PHP**选项卡。现在您应该看到一个名为**PHP 扩展**的部分。在该部分中应该有一个下拉列表（这可能是设置为 XCache）；从该列表中选择 APC，然后点击**确定**按钮。MAMP 将重新启动，在此之后，您应该可以正常使用。

# 使用 CodeIgniter 记录错误

在您的 CodeIgniter 应用程序中记录发生的错误不必仅限于查看 PHP 或 Apache 日志；您可以通过 CodeIgniter 的日志功能启用 CodeIgniter 在代码的某些点处理和记录错误以及其他行为和事件。这个功能（如果设置正确）可以特别有用，用于跟踪用户的系统旅程和进度，如果他们正在执行的操作出现任何问题，您可以在日志中查找并追踪他们所做的一切以及何时发生，从而更好地了解（如果有的话）发生了什么，并希望考虑如何防止其再次发生。

在本配方中，我们将探讨在 CodeIgniter 中使用日志功能以及跟踪操作中是否出现错误。

## 准备工作

我们需要在配置文件中设置日志报告级别，以便 CodeIgniter 知道要报告哪种级别的日志消息。日志文件夹也应具有写权限，因此请执行以下步骤：

1.  打开`/path/to/codeigniter/application/config/config.php`文件，找到以下行：

    ```php
    $config['log_threshold'] = 0;
    ```

    您可以将`log_threshold`配置数组元素的值更改为以下五种状态之一，具体如下表所示：

    | 状态 | 用途 | 描述 |
    | --- | --- | --- |
    | `0` | - | CodeIgniter 将不记录任何内容。 |
    | `1` |

    ```php
    log_message('error', 'Some Text')
    ```

    | CodeIgniter 将记录错误信息。 |
    | --- |
    | `2` |

    ```php
    log_message('debug', 'Some Text')
    ```

    | CodeIgniter 将记录调试信息。 |
    | --- |
    | `3` |

    ```php
    log_message('info', 'Some Text');
    ```

    | CodeIgniter 将记录信息消息。 |
    | --- |
    | `4` | 所有以上 | CodeIgniter 将记录所有内容。 |

    对于我们的配方，我已将值设置为`4`，因为我希望记录 CodeIgniter 可能为我生成的任何错误消息或信息消息。

1.  查找以下行：

    ```php
    $config['log_path'] = '/path/to/log/folder/';.
    ```

    确保在`$config['log_path']`中正确设置了`/path/to/log/folder/`，并且指定的文件夹具有写权限（否则，CodeIgniter 无法将该文件夹中的日志文件写入）。

## 如何操作...

我们将使用（因为它很方便）本章前面提到的 *使用 CodeIgniter 缓存* 菜谱，并对其进行修改以应用 CodeIgniter 日志。在这个菜谱中，我们将它的名称更改为 `cache_log.php`（以保持与 `rss_cache.php` 分离）。所以，如果您还没有这样做（不要忘记更改名称，以下代码中突出显示）：

1.  创建 `/path/to/codeigniter/application/controllers/cache_log.php` 文件，并将以下代码添加到其中（更改已在代码中突出显示）：

    ```php
    <?php if ( ! defined('BASEPATH')) exit('No direct script access allowed');

    class Cache_log extends CI_Controller {
      function __construct() {
        parent::__construct();
        $this->load->helper('url');
        $this->load->helper('xml');
        $this->load->driver('cache', array('adapter' => 'apc'));
      }

      public function index() {
        $raw_feed = '<?xml version="1.0" encoding="ISO-8859-1" ?>
        <rss version="2.0">
        <channel>
          <title>RSS Feed</title>
            <link>http://www.domain.com</link>
            <description>General Description</description>
          <item>
            <title>RSS Item 1 Title</title>
            <link>http://www.domain1.com/link1</link>
            <description>Description of First Item</description>
          </item>
          <item>
            <title>RSS Item 2 Title</title>
            <link>http://www.domain2.com/link2</link>
            <description>Description of Second Item</description>
          </item>
          <item>
            <title>RSS Item 3 Title</title>
            <link>http://www.domain3.com/link3</link>
            <description>Description of Third Item</description>
          </item>
        </channel>
        </rss>';

        $feed = new SimpleXmlElement($raw_feed);

     if (!$feed) {
     log_message('error', 'Unable to instantiate SimpleXmlElement.');
     } else {
     log_message('info', 'SimpleXmlElement was instantiated correctly.');
     }

     if (!$cached_feed = $this->cache->get('rss')) {
     foreach ($feed->channel->item as $item) {
     $cached_feed .= $item->title . '<br />' .$item->description . '<br /><br />';
     }

     $this->cache->save('rss', $cached_feed, 7);
     log_message('info', 'Cache item saved.');
     }

        echo $this->cache->get('rss');

      }

      public function clear_cache() {
     if (!$this->cache->clean()) {
     log_message('error', 'Unable to clear Cache.');
     } else {
     log_message('info', 'Cache cleared.');
     }

        redirect('rss_cache');
      }
    }
    ?>
    ```

    如果一切顺利，您不应该有任何错误，但在配置文件中应该有一些 `DEBUG` 数据。当您打开该文件时，您应该看到类似以下内容：

    ```php
    <?php  if ( ! defined('BASEPATH')) exit('No direct script access allowed'); ?>

    DEBUG - 2013-09-24 19:46:11 --> Config Class Initialized
    DEBUG - 2013-09-24 19:46:11 --> Hooks Class Initialized
    DEBUG - 2013-09-24 19:46:11 --> Utf8 Class Initialized
    DEBUG - 2013-09-24 19:46:11 --> UTF-8 Support Enabled
    DEBUG - 2013-09-24 19:46:11 --> URI Class Initialized
    DEBUG - 2013-09-24 19:46:11 --> Router Class Initialized
    DEBUG - 2013-09-24 19:46:11 --> Output Class Initialized
    DEBUG - 2013-09-24 19:46:11 --> Security Class Initialized
    DEBUG - 2013-09-24 19:46:11 --> Input Class Initialized
    DEBUG - 2013-09-24 19:46:11 --> XSS Filtering completed
    DEBUG - 2013-09-24 19:46:11 --> XSS Filtering completed
    DEBUG - 2013-09-24 19:46:11 --> XSS Filtering completed
    DEBUG - 2013-09-24 19:46:11 --> CRSF cookie Set
    DEBUG - 2013-09-24 19:46:11 --> Global POST and COOKIE data sanitized
    DEBUG - 2013-09-24 19:46:11 --> Language Class Initialized
    DEBUG - 2013-09-24 19:46:11 --> Loader Class Initialized
    DEBUG - 2013-09-24 19:46:11 --> Database Driver Class Initialized
    DEBUG - 2013-09-24 19:46:11 --> Session Class Initialized
    DEBUG - 2013-09-24 19:46:11 --> Helper loaded: string_helper
    DEBUG - 2013-09-24 19:46:11 --> Encrypt Class Initialized
    DEBUG - 2013-09-24 19:46:11 --> Session routines successfully run
    DEBUG - 2013-09-24 19:46:11 --> XML-RPC Class Initialized
    DEBUG - 2013-09-24 19:46:11 --> Controller Class Initialized
    DEBUG - 2013-09-24 19:46:11 --> Helper loaded: url_helper
    DEBUG - 2013-09-24 19:46:11 --> Helper loaded: xml_helper
    INFO  - 2013-09-24 19:46:11 --> SimpleXmlElement was instantiated correctly.
    INFO  - 2013-09-24 19:46:11 --> Cache item saved.
    DEBUG - 2013-09-24 19:46:11 --> Final output sent to browser
    DEBUG - 2013-09-24 19:46:11 --> Total execution time: 0.0280
    ```

    您可以在日志中看到我们在代码中设置的 info 项；我已经突出显示它们以便它们突出显示。

## 它是如何工作的...

您可以看到我们在控制器执行的各个阶段添加了一些条件语句，检查某些 CodeIgniter 函数的返回值（更改已在之前的代码中突出显示）。根据返回值（`TRUE` 或 `FALSE`），我们将使用 CodeIgniter 的 `log_message()` 函数将信息写入日志，但让我们更仔细地看看这些消息以及它们何时被触发。

首先，我们将尝试实例化一个新的 `SimpleXmlElement()` 对象。如果我们得到返回的对象，日志中会写入一条信息消息（`SimpleXmlElement()` 实例化正确）。如果有错误，我们将错误消息写入日志（无法实例化 `SimpleXmlElement()`）；请看以下代码片段：

```php
$feed = new SimpleXmlElement($raw_feed);

if (!$feed) {
  log_message('error', 'Unable to instantiate SimpleXmlElement.');
} else {
  log_message('info', 'SimpleXmlElement was instantiated correctly.');
}
```

您可以看到我们正在使用 CodeIgniter 的日志功能将消息写入日志文件，并将这些消息定义为错误或信息；这有助于调试用户的旅程，因为您将知道什么是真正的错误，以及您输入到日志中的信息。

### 日志风格

我发现将日志消息写成以下代码片段很有用：

```php
 log_message('info', ' **** ' . __LINE__ . ' – This is a message.');
```

在前面的代码片段中，我们将消息定义为信息，但消息以四个星号（`****`）开始。这将使消息在查看日志时突出显示，接下来是 `__LINE__` 参数（让您知道在脚本中的触发位置），然后是实际的消息--这里是非常无趣的--`' - This is a message'`。

您可能希望根据需要添加 `__FILE__`、`__CLASS__` 或 `__FUNCTION__` 以提高准确性。

# 应用程序的基准测试

基准测试对你来说可能很有用，因为它可以让你了解你的应用程序如何处理计算所有代码的任务。它可以让你知道在你的应用程序中哪里可能存在速度慢的问题，无论是由于内存限制，还是可能是因为一段特别计算密集的代码块。利用这些信息，你可以确定是否存在任何瓶颈，并且如果你能够清除它们，可能通过重新编程或分配额外资源。下面是如何进行的。

## 准备工作

许多网络应用程序都会链接到某种数据库，作为基准测试数据库连接性的一个例子，我们将查询一个数据库。为了做到这一点，我们显然需要一个数据库来连接。将以下 MySQL 代码复制到你的数据库中：

```php
CREATE TABLE `bench_table` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `firstname` varchar(50) NOT NULL,
  `lastname` varchar(50) NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB  DEFAULT CHARSET=latin1 AUTO_INCREMENT=3 ;

INSERT INTO `bench_table` (`id`, `firstname`, `lastname`) VALUES
(1, 'Rob', 'Foster'),
(2, 'Lucy', 'Welsh');
```

## 如何操作...

我们将创建以下两个文件：

+   `/path/to/codeigniter/application/controllers/bench.php`

+   `/path/to/codeigniter/application/models/bench_model.php`

1.  创建 `bench.php` 控制器文件，并将以下代码添加到其中：

    ```php
    <?php if ( ! defined('BASEPATH')) exit('No direct script access allowed');

    class Bench extends CI_Controller {
      function __construct() {
        parent::__construct();
        $this->load->helper('url');
        $this->load->model('bench_model');
        $this->load->database();
      }

      public function index() {
        // Who's in the database?
        $this->benchmark->mark('bm1_start');
        foreach ($this->bench_model->get_people()->result() as $row) {
          echo $row->firstname . ' ' . $row->lastname . '<br />';
        }
        $this->benchmark->mark('bm1_end');

        // Write some more people to the database	
        $this->benchmark->mark('bm2_start');
        $data = array(
          array('firstname' => 'George',
              'lastname' => 'Foster'),
          array('firstname' => 'Jackie',
               'lastname' => 'Foster'),
          array('firstname' => 'Antony',
              'lastname' => 'Welsh'),
          array('firstname' => 'Rowena',
              'lastname' => 'Welsh'),
          array('firstname' => 'Peter',
              'lastname' => 'Foster'),
          array('firstname' => 'Jenny',
              'lastname' => 'Foster'),
          array('firstname' => 'Oliver',
              'lastname' => 'Welsh'),
          array('firstname' => 'Harrison',
              'lastname' => 'Foster'),
          array('firstname' => 'Felicity',
              'lastname' => 'Foster')
          );

        $result = $this->bench_model->add_to_db($data);
        $this->benchmark->mark('bm2_end');

        if ($result) {
          // Who's in the database now?
          $this->benchmark->mark('bm3_start');
          foreach ($this->bench_model->get_people()->result() as $row) {
            echo $row->firstname . ' ' . $row->lastname . '<br />';
          }
          $this->benchmark->mark('bm3_end');
        } else {
          echo 'Cannot write to database.';
        }

        echo '<br /> ---- BENCHMARK POINT STATS ---- <br />';
        echo 'BM1 (S) to BM1 (E): ' . $this->benchmark->elapsed_time('bm1_start','bm1_end') . '<br />';
        echo 'BM2 (S) to BM2 (E): ' . $this->benchmark->elapsed_time('bm2_start','bm2_end') . '<br />';
        echo 'BM3 (S) to BM3 (E): ' . $this->benchmark->elapsed_time('bm3_start','bm3_end') . '<br />';
      }
    }
    ?>
    ```

1.  创建 `bench_model.php` 模型文件，并将以下代码添加到其中：

    ```php
    <?php if ( ! defined('BASEPATH')) exit('No direct script access allowed');

    class Bench_model extends CI_Model {
        function __construct() {
            parent::__construct();
        }

        function get_people() {
            $query = $this->db->get('bench_table');
            return $query;
        }

        function add_to_db($data) {
            if ($this->db->insert_batch('bench_table', $data)) {
                return TRUE;
            } else {
                return FALSE;
            }
        }
    }
    ```

## 它是如何工作的...

如果你通过浏览器运行控制器 bench，你应该在屏幕上看到以下输出：

```php
Rob Foster
Lucy Welsh
Rob Foster
Lucy Welsh
George Foster
Jackie Foster
Antony Welsh
Rowena Welsh
Peter Foster
Jenny Foster
Oliver Welsh
Harrison Foster
Felicity Foster
---- BENCHMARK POINT STATS ---- 
BM1 (S) to BM1 (E): 0.0004
BM2 (S) to BM2 (E): 0.0010
BM3 (S) to BM3 (E): 0.0005
```

输出是 `bench_table` 数据库表的表内容循环，随后是基准测试统计信息。我已经突出了 `batch_insert()` 操作的统计数据。

`BM1 (S)` 是 BM1 基准测试的开始，而 `BM1 (E)` 是 BM1 基准测试的结束。

你可以看到，执行 `batch_insert()` 操作比两个读取操作（在你的应用程序中实际的处理时间可能会有所不同，而且每次运行控制器时也可能会有所不同；然而，BM2 项几乎总是更长时间，但时间会根据你使用的系统而有所不同）花费的时间要长得多。

如果这是一个更复杂的情况，我们可以使用这些信息来定位代码中的瓶颈，并希望修复它们以确保应用程序更加流畅。

那么，代码中发生了什么？由于基准系统总是由 CodeIgniter 加载并且总是可用，因此没有库或其他资源需要加载。

当我们运行基准控制器时，会调用 `public function index()` 并立即运行 `bench_model` 的 `get_people()` 函数。这在对 `bench_table` 数据库表的 Active Record `SELECT` 操作，将结果对象返回给控制器。然后我们遍历这个循环，并输出每一行以显示在 `batch_insert()` 操作之前数据库中的行列表：

```php
$this->benchmark->mark('bm1_start');
foreach ($this->bench_model->get_people()->result() as $row) {
  echo $row->firstname . ' ' . $row->lastname . '<br />';
}
$this->benchmark->mark('bm1_end');

```

眼尖的你们中的一些人也会注意到突出显示的行，我们为 CodeIgniter 定义了关注的开头和结尾点。第一个我们命名为 `bm1_start`，第二个我们命名为 `bm1_stop`。我们可以称它们为任何我们喜欢的名字，但这是我决定要叫的名字。

我们随后执行 `batch_insert` 操作，如下代码片段所示：

```php
$this->benchmark->mark('bm2_start');
$data = array(
  array('firstname' => 'George',
      'lastname' => 'Foster'),
  array('firstname' => 'Jackie',
      'lastname' => 'Foster'),
  array('firstname' => 'Antony',
      'lastname' => 'Welsh'),
  array('firstname' => 'Rowena',
      'lastname' => 'Welsh'),
  array('firstname' => 'Peter',
      'lastname' => 'Foster'),
  array('firstname' => 'Jenny',
      'lastname' => 'Foster'),
  array('firstname' => 'Oliver',
      'lastname' => 'Welsh'),
  array('firstname' => 'Harrison',
      'lastname' => 'Foster'),
  array('firstname' => 'Felicity',
      'lastname' => 'Foster')
  );

  $result = $this->bench_model->add_to_db($data);
$this->benchmark->mark('bm2_end');

```

我们正在定义一个包含我们想要添加到数据库中的人的详细信息的多维数组，并将其发送到 `bench_model` 的 `insert_batch()` 函数；现在，那些敏锐的眼睛中的人会再次注意到高亮行。这些是 bm2 的开始和结束点。如果 `batch_insert()` 操作返回 `TRUE`（正确插入到数据库中），我们随后再次调用 `get_people()` 模型函数，这将返回数据库中的所有记录：

```php
if ($result) {
  // Who's in the database now?
 $this->benchmark->mark('bm3_start');
  foreach ($this->bench_model->get_people()->result() as $row) {
    echo $row->firstname . ' ' . $row->lastname . '<br />';
  }
 $this->benchmark->mark('bm3_end');
} else {
  echo 'Cannot write to database.';
}
```

再次，这里我们定义（如前一段代码中高亮显示的）bm3 的开始和结束点。这完成了我们的数据库操作，然后我们转向基准测试的汇报。

我们要求 CodeIgniter 告诉我们两个点之间的执行时间：

```php
echo '<br /> ---- BENCHMARK POINT STATS ---- <br />';
echo 'BM1 (S) to BM1 (E): ' . $this->benchmark->elapsed_time('bm1_start','bm1_end') . '<br />';
echo 'BM2 (S) to BM2 (E): ' . $this->benchmark->elapsed_time('bm2_start','bm2_end') . '<br />';
echo 'BM3 (S) to BM3 (E): ' . $this->benchmark->elapsed_time('bm3_start','bm3_end') . '<br />';
```

对于每个 bm1、bm2 和 bm3，我们想要知道使用 `$this->benchmark->elapsed_time()` 函数指定的点之间的时间。这个函数接受两个参数：一个起始点和结束点。对于这个菜谱，我们已经要求 CodeIgniter 报告每个 bm# 点（其中 # 是数字 1、2 或 3）之间经过的时间，但如果我们愿意，我们可以这样写：

```php
echo 'BM1 (S) to BM2 (E): ' . $this->benchmark->elapsed_time('bm1_start','bm2_end') . '<br />'
```

之前的代码将报告 `bm1_start` 和 `bm2_end` 之间的时间差（或者从第一个 `get_people()` 查询的开始到 `batch_insert()` 查询的结束）。

将每个 `$this->benchmark->mark('bm2_end');` 视为一个检查点，并且你可以使用 `$this->benchmark->elapsed_time('checkpoint_1','checkpoint_2')` 来返回它们之间的时间差。
