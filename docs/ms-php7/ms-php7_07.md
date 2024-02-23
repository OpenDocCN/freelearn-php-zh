# 优化高性能

多年来，PHP 已经发展成为我们构建 Web 应用程序所使用的一种非凡语言。令人印象深刻的语言特性，以及无数的库和框架，使我们的工作变得更加容易。我们经常编写涵盖多层堆栈的代码，而不加思索。这使得很容易忽视每个应用程序最重要的方面之一--性能。

虽然性能有几个方面需要开发人员注意，但最终用户只对一个方面感兴趣 - 网页加载所需的时间。这才是最重要的。如今，用户期望他们的页面在 2 秒内加载。如果超过这个时间，我们将面临转化率下降，这通常会导致大型电子商务零售商严重的财务损失：

“页面响应延迟 1 秒可能导致转化率减少 7%。”“如果一个电子商务网站每天赚取 10 万美元，1 秒的页面延迟可能会导致您每年损失 250 万美元的销售额。”

- kissmetrics.com

在本章中，我们将讨论一些直接或间接影响应用程序性能和行为的 PHP 领域：

+   最大执行时间

+   内存管理

+   文件上传

+   会话处理

+   输出缓冲

+   禁用调试消息

+   Zend OPcache

+   并发

# 最大执行时间

**最大执行时间**是开发人员经常遇到的最常见错误之一。默认情况下，在浏览器中执行的 PHP 脚本的最大执行时间为 30 秒，除非我们在 CLI 环境中执行脚本，那里没有这样的限制。

我们可以通过一个简单的例子进行测试，通过`index.php`和`script.php`文件，如下所示：

```php
<?php
// index.php
require_once 'script.php';
error_reporting(E_ALL);
ini_set('display_errors', 'On');
sleep(10);
echo 'Test#1';

```

```php
?php
// script.php
sleep(25);
echo 'Test#2';

```

在浏览器中执行，将返回以下错误：

```php
Test#2
Fatal error: Maximum execution time of 30 seconds exceeded in /var/www/html/index.php on line 5

```

在 CLI 环境中执行，将返回以下输出：

```php
Test#2Test#1

```

幸运的是，PHP 提供了两种控制超时值的方法：

+   使用`max_execution_time`配置指令（`php.ini`文件，`ini_set()`函数）

+   使用`set_time_limit()`函数

`set_time_limit()`函数的使用具有有趣的含义。让我们看看以下例子：

```php
<?php
// index.php
error_reporting(E_ALL);
ini_set('display_errors', 'On');
echo 'Test#1';
sleep(5);
set_time_limit(10);
sleep(15);
echo 'Test#2';

```

上面的例子将导致以下错误：

```php
Test#1
Fatal error: Maximum execution time of 10 seconds exceeded in /var/www/html/index.php on line 9

```

有趣的是，`set_time_limit()`函数会从调用它的地方重新启动超时计数器。这实际上意味着，在一个非常复杂的系统中，通过在代码中多次使用`set_time_limit()`函数，我们可以显著扩展超时时间，超出最初设想的边界。这是非常危险的，因为 PHP 超时不是在向用户浏览器交付最终网页时唯一的超时。 

Web 服务器具有各种超时配置，可能会中断 PHP 执行：

+   Apache：

+   `TimeOut`指令，默认为 60 秒

+   Nginx：

+   `client_header_timeout`指令，默认为 60 秒

+   `client_body_timeout`指令，默认为 60 秒

+   `fastcgi_read_timeout`指令，默认为 60 秒

虽然我们可以在浏览器环境中控制脚本超时，但重要的问题是*为什么我们要这样做*？超时通常是资源密集型操作的结果，例如各种非优化循环，数据导出，导入，PDF 文件生成等。CLI 环境，或者理想情况下，专用服务，应该是我们处理所有资源密集型工作的首选。而浏览器环境的主要重点应该是以尽可能短的时间向用户提供页面。

# 内存管理

PHP 开发人员经常需要处理大量数据。虽然“大量”是一个相对的术语，但内存不是。当不负责任地使用某些函数和语言结构的组合时，我们的服务器内存可能在几秒钟内被堵塞。

可能最臭名昭著的函数是`file_get_contents()`。这个易于使用的函数会将整个文件的内容放入内存中。为了更好地理解问题，让我们看看以下例子：

```php
<?php

$content = file_get_contents('users.csv');
$lines = explode("\r\n", $content);

foreach ($lines as $line) {
  $user = str_getcsv($line);
  // Do something with data from $user...
}

```

虽然这段代码完全有效且可用，但它是潜在的性能瓶颈。`$content`变量将整个`users.csv`文件的内容加载到内存中。虽然这对于小文件大小可能有效，比如几兆字节，但这段代码并没有经过性能优化。一旦`users.csv`开始增长，我们将开始遇到内存问题。

我们可以采取什么措施来减轻问题？我们可以重新思考解决问题的方法。一旦我们将思维转向“必须优化性能”模式，其他解决方案就会变得清晰。我们可以不将整个文件的内容读入变量，而是逐行解析文件。

```php
<?php

if (($users = fopen('users.csv', 'r')) !== false) {
  while (($user = fgetcsv($users)) !== false) {
    // Do something with data from $user...
  }
  fclose($users);
}

```

我们不使用`file_get_contents()`和`str_getcsv()`，而是专注于使用另一组函数`fopen()`和`fgetcsv()`。最终结果完全相同，而且完全符合性能友好。在这种特定情况下使用带有句柄的函数，我们有效地确保了内存限制对我们的脚本不构成问题。

不负责任地使用循环是内存的另一个常见原因：

```php
<?php   $conn = new PDO('mysql:host=localhost;dbname=eelgar_live_magento', 'root', 'mysql');   $stmt = $conn->query('SELECT * FROM customer_entity'); $users = $stmt->fetchAll();   foreach ($users as $user) {
  if (strstr($user['email'], 'test')) {
  // $user['entity_id']
 // $user['email'] // Do something with data from $user...  } }

```

现在，让我们继续看一个修改后的、内存友好的例子，效果相同：

```php
<?php

$conn = new PDO('mysql:host=localhost;dbname=eelgar_live_magento', 
  'root', 'mysql');

$stmt = $conn->prepare('SELECT entity_id, email FROM customer_entity WHERE email LIKE :email');
$stmt->bindValue(':email', '%test%');
$stmt->execute();

while ($user = $stmt->fetch(PDO::FETCH_ASSOC)) {
  // $user['entity_id']
  // $user['email']
  // Do something with data from $user...
}

```

`fetchAll()`方法比`fetch()`稍快，但需要更多内存。

当 PHP 达到内存限制时，它会停止脚本执行并抛出以下错误：

```php
Fatal error: Allowed memory size of 33554432 bytes exhausted (tried to 
allocate 2348617 bytes) ...

```

幸运的是，`memory_limit`指令使我们能够控制可用内存量。默认的`memory_limit`值是`128M`，这意味着 128 兆字节的内存。该指令是`PHP_INI_ALL`可更改的，这意味着除了通过`php.ini`文件设置它外，我们还可以使用`ini_set('memory_limit', '512M');`在运行时设置它。

除了调整`memory_limit`指令外，PHP 提供了以下两个返回内存使用信息的函数：

+   `memory_get_usage()`: 返回当前 PHP 脚本分配的内存量

+   `memory_get_peak_usage()`: 返回 PHP 脚本分配的内存峰值

虽然我们可能会想要增加这个值，但我们应该三思而后行。内存限制是针对进程而不是服务器的。Web 服务器本身可以启动多个进程。因此，使用大内存限制值可能会阻塞我们的服务器。除此之外，任何可能实际消耗大量内存的脚本都很容易成为性能优化的候选。对我们的代码应用简单而经过深思熟虑的技术可以大大减少内存使用。

在实际内存管理方面，这里的情况相当自动化。与 C 语言不同，我们需要自己管理内存，PHP 使用垃圾回收结合引用计数机制。不用深入机制本身的细节，可以说变量在不再使用时会自动释放。

有关垃圾回收的更多详细信息，请查看[`php.net/manual/en/features.gc.php`](http://php.net/manual/en/features.gc.php)。

# 文件上传

上传文件是许多 PHP 应用程序的常见功能。PHP 提供了一个方便的全局`$_FILES`变量，我们可以用它来访问上传的文件，或者在文件上传尝试后的错误。

让我们看看以下简单的文件上传表单：

```php
<form method="post" enctype="multipart/form-data">
  <input type="file" name="photo" />
  <input type="file" name="article" />
  <input type="submit" name="submit" value="Upload" />
</form>

```

为了让 PHP 获取文件，我们需要将`form method`的值设置为`post`，并将`enctype`设置为`multipart/form-data`。一旦提交，PHP 将接收并适当填充`$_FILES`变量：

```php
array(2) { 
  ["photo"] => array(5) { 
    ["name"] => string(9) "photo.jpg" 
    ["type"] => string(10) "image/jpeg" 
    ["tmp_name"] => string(14) "/tmp/phpGutI91" 
    ["error"] => int(0) 
    ["size"] => int(42497) 
  } 
  ["article"] => array(5) { 
    ["name"] => string(11) "article.pdf" 
    ["type"] => string(15) "application/pdf" 
    ["tmp_name"] => string(14) "/tmp/phpxsnx1e" 
    ["error"] => int(0) 
    ["size"] => int(433176)
  } 
}

```

在不涉及实际的上传后文件管理的细节的情况下，可以说`$_FILES`包含足够的信息，以便我们可以选择并进一步管理文件，或者在上传过程中指示可能的错误代码。以下八个错误代码可以返回：

+   `UPLOAD_ERR_OK`

+   `UPLOAD_ERR_INI_SIZE`

+   `UPLOAD_ERR_FORM_SIZE`

+   `UPLOAD_ERR_PARTIAL`

+   `UPLOAD_ERR_NO_FILE`

+   `UPLOAD_ERR_NO_TMP_DIR`

+   `UPLOAD_ERR_CANT_WRITE`

+   `UPLOAD_ERR_EXTENSION`

虽然所有错误应该得到同等对待，但其中两个（`UPLOAD_ERR_FORM_SIZE`和`UPLOAD_ERR_PARTIAL`）引发了关键的性能问题：*我们可以上传多大的文件*以及*在过程中是否存在任何超时*？ 

这两个问题的答案可以在配置指令中找到，其中一些直接与文件上传相关，而其他一些与更一般的 PHP 选项相关：

+   `session.gc_maxlifetime`：这是数据被视为垃圾并清理的秒数；默认为 1,440 秒

+   `session.cookie_lifetime`：这是 cookie 的生存时间（以秒为单位）；默认情况下，cookie 在浏览器关闭之前有效

+   `max_input_time`：这是脚本允许解析输入数据（如 POST）的最长时间（以秒为单位）；默认情况下，此功能已关闭

+   `max_execution_time`：这是脚本允许运行的最长时间；默认为 30 秒

+   `upload_max_filesize`：这是上传文件的最大大小；默认为 2 兆字节（2M）

+   `max_file_uploads`：这是允许在单个请求中上传的最大文件数

+   `post_max_size`：这是允许的 POST 数据的最大大小；默认为 8 兆字节（8M）

调整这些选项可以确保我们避免超时和计划的大小限制。为了确保我们可以在过程的早期避免最大文件大小限制，`MAX_FILE_SIZE`可以用作隐藏的表单字段：

```php
<form method="post" enctype="multipart/form-data">
 <input type="hidden" name="MAX_FILE_SIZE" value="100"/>
 <input type="file" name="photo"/>
 <input type="file" name="article"/>
 <input type="submit" name="submit" value="Upload"/>
</form>

```

`MAX_FILE_SIZE`字段必须位于表单可能具有的任何其他文件字段之前。它的值代表 PHP 接受的最大文件大小。

尝试上传大于`MAX_FILE_SIZE`隐藏字段定义的文件现在将导致类似于此处所示的`$_FILES`变量：

```php
array(2) {
  ["photo"] => array(5) {
    ["name"] => string(9) "photo.jpg"
    ["type"] => string(0) ""
    ["tmp_name"] => string(0) ""
    ["error"] => int(2)
    ["size"] => int(0)
  }
  ["article"] => array(5) {
    ["name"] => string(11) "article.pdf"
    ["type"] => string(0) ""
    ["tmp_name"] => string(0) ""
    ["error"] => int(2)
    ["size"] => int(0)
  }
}

```

我们可以看到错误现在已经变成了值`2`，这等于`UPLOAD_ERR_FORM_SIZE`常量。

通常情况下，我们会通过代码优化来解决默认配置的限制，但文件上传是特殊的；因此，我们确实需要确保如果需要的话可以上传大文件。

# 会话处理

会话在 PHP 中是一个有趣的机制，允许我们在总体上是无状态的通信中保持状态。我们可以将它们视为保存在文件中的*每个用户序列化信息数组*。我们使用它们来在各个页面上存储用户特定的信息。默认情况下，会话依赖于 cookie，尽管它们可以配置为在浏览器中使用`SID`参数。

PHP 会话的 cookie 版本大致如下：

1.  从 cookie 中读取会话令牌。

1.  在磁盘上创建或打开现有文件。

1.  锁定文件以进行写入。

1.  读取文件的内容。

1.  将文件数据放入全局`$_SESSION`变量中。

1.  设置缓存头。

1.  将 cookie 返回给客户端。

1.  在每个页面请求上，重复步骤 1-7。

PHP 会话的*SID 版本*工作方式基本相同，除了 cookie 部分。这里的 cookie 被我们通过 URL 推送的 SID 值所取代。

会话机制可用于各种事情，其中一些包括用户登录机制，存储小型数据缓存，模板的部分等。根据使用情况，这可能会引发“最大会话大小”的问题。

默认情况下，脚本执行时，会将会话从文件读入内存。因此，会话文件的最大大小不能超过`memory_limit`指令，默认为 128 兆字节。我们可以通过定义自定义会话处理程序来绕过这种*默认*会话行为。`session_set_save_handler()`函数允许我们注册自定义会话处理程序，该处理程序必须符合`SessionHandlerInterface`接口。使用自定义会话处理程序，我们可以将会话数据存储在数据库中，从而实现更高的性能效率，因为我们现在可以在负载均衡器后面创建可扩展的 PHP 环境，其中所有应用程序节点连接到一个中央会话服务器。

Redis 和 memcached 是两种在 PHP 开发人员中非常流行的数据存储方式。Magento 2 电子商务平台支持 Redis 和 memcached 用于外部会话存储。

会话存储在性能方面起着关键作用，有一些配置指令值得关注：

+   `session.gc_probability`: 这默认为 1

+   `session.gc_divisor`: 这默认为 100

+   `gc_maxlifetime`: 这默认为 1,440 秒（24 分钟）

`gc_probability`和`gc_divisor`指令一起工作。它们的比率（*gc_probability/gc_divisor => 1/100 => 1%*）定义了在每次`session_start()`调用时垃圾收集器运行的概率。一旦垃圾收集器运行，`gc_maxlifetime`指令的值告诉它是否应将某些内容视为垃圾并潜在地进行清理。

在高性能网站中，会话很容易成为瓶颈。深思熟虑的调整和会话存储选择可以带来恰到好处的性能差异。

# 输出缓冲

输出缓冲是一种控制脚本输出的 PHP 机制。想象一下，我们在 PHP 脚本中写下`echo 'test';`，但屏幕上什么都看不到。这是怎么可能的？答案是**输出缓冲**。

以下代码是输出缓冲的一个简单示例：

```php
<?php

ob_start();
sleep(2);
echo 'Chunk#1' . PHP_EOL;
sleep(3);
ob_end_flush();

ob_start();
echo 'Chunk#2' . PHP_EOL;
sleep(5);
ob_end_clean();

ob_start();
echo 'Chunk#3' . PHP_EOL;
ob_end_flush();

ob_start();
sleep(5);
echo 'Chunk#4' . PHP_EOL;

//Chunk#1
//Chunk#3
//Chunk#4

```

在 CLI 环境中执行时，我们首先会在几秒钟后看到`Chunk#1`，然后再过几秒钟，我们会看到`Chunk#3`，最后再过几秒钟，我们会看到`Chunk#4`。`Chunk#2`永远不会被输出。鉴于我们习惯于在调用后立即看到`echo`构造输出的内容，这是一个相当有意思的概念。

有几个与输出缓冲相关的函数，其中以下五个是最有趣的：

+   `ob_start()`: 这会触发一个新的缓冲区，并在另一个*未关闭*缓冲区之后调用时创建堆叠的缓冲区

+   `ob_end_flush()`: 这会输出顶部的缓冲区并关闭这个输出缓冲区

+   `ob_end_clean()`: 这会清除输出缓冲区并关闭输出缓冲

+   `ob_get_contents()`: 这会返回输出缓冲区的内容

+   `ob_gzhandler()`: 这是与`ob_start()`一起使用的回调函数，用于对输出缓冲进行 GZIP 压缩

以下示例演示了堆叠的缓冲区：

```php
<?php

ob_start(); // BUFFER#1
sleep(2);
echo 'Chunk #1' . PHP_EOL;

 ob_start(); // BUFFER#2
 sleep(2);
 echo 'Chunk #2' . PHP_EOL;
 ob_start(); // BUFFER#3
 sleep(2);
 echo 'Chunk #3' . PHP_EOL;
 ob_end_flush();
 ob_end_flush();

sleep(2);
echo 'Chunk #4' . PHP_EOL;
ob_end_flush();

//Chunk #1
//Chunk #2
//Chunk #3
//Chunk #4

```

整个输出在这里被暂停了大约 8 秒，之后所有四个`Chunk#...`字符串一次性输出。这是因为`ob_end_flush()`函数是唯一将输出发送到控制台的函数，而`ob_end_flush()`函数仅仅关闭缓冲区，并将其传递给代码中存在的父缓冲区。

`ob_get_contents()`函数的使用可以为输出缓冲添加更多动态，如下例所示：

```php
<?php

$users = ['John', 'Marcy', 'Alice', 'Jack'];

ob_start();
foreach ($users as $user) {
  echo 'User: ' . $user . PHP_EOL;
}
$report = ob_get_contents();
ob_end_clean();

ob_start();
echo 'Listing users:' . PHP_EOL;
ob_end_flush();

echo $report;

echo 'Total of ' . count($users) . ' users listed' . PHP_EOL;

//Listing users:
//User: John
//User: Marcy
//User: Alice
//User: Jack
//Total of 4 users listed

```

`ob_get_content()`函数允许我们获取缓冲区中存储的内容的字符串表示。我们可以选择是否要进一步修改该内容，输出它，或将其传递给其他结构。

所有这些如何适用于网页？毕竟，我们主要关注我们脚本的性能，大多数情况下是在网页的上下文中。没有输出缓冲，HTML 将随着 PHP 在脚本中的进行而以块的形式发送到浏览器。有了输出缓冲，HTML 将在我们的脚本结束时作为一个字符串发送到浏览器。

请记住，`ob_start()`函数接受一个回调函数，我们可以使用回调函数进一步修改输出。这种修改可以是任何形式的过滤，甚至是压缩。

以下示例演示了输出过滤的使用：

```php
<?php

ob_start('strip_away');
echo '<h1>', 'Bummer', '</h1>';
echo '<p>', 'I felt foolish and angry about it!', '</p>';
ob_end_flush();

function strip_away($buffer)
{
  $keywords = ['bummer', 'foolish', 'angry'];
  foreach ($keywords as $keyword) {
    $buffer = str_ireplace(
      $keyword,
      str_repeat('X', strlen($keyword)),
      $buffer
    );
  }
  return $buffer;
}

// Outputs:
// <h1>XXXXXX</h1><p>I felt XXXXXXX and XXXXX about it!</p>

```

现在，然而，我们不太可能自己编写这些结构，因为框架抽象为我们掩盖了它。

# 禁用调试消息

Ubuntu 服务器是一种流行的、免费的、开源的 Linux 发行版，我们可以使用它快速设置**LAMP**（**Linux, Apache, MySQL, PHP**）堆栈。Ubuntu 服务器的易安装性和长期支持使其成为 PHP 开发人员的热门选择。通过干净的服务器安装，我们可以通过执行以下命令快速启动和运行 LAMP 堆栈：

```php
sudo apt-get update && sudo apt-get upgrade
sudo apt-get install lamp-server^ 

```

完成这些操作后，访问我们的外部服务器 IP 地址，我们应该看到一个 Apache 页面，如下面的屏幕截图所示：

![](img/65d05a45-40ba-469f-b2b8-2e510122490d.jpg)

我们在浏览器中看到的 HTML 源自`/var/www/html/index.html`文件。将`index.html`替换为`index.php`后，我们就可以使用 PHP 代码了。

介绍 Ubuntu 服务器的原因是为了强调某些服务器默认值。在所有配置指令中，我们不应该盲目接受*错误记录*和*错误显示*指令的默认值，而不真正理解它们。在开发和生产环境之间不断切换使得在浏览器中暴露机密信息或错过记录正确错误变得太容易了。

考虑到这一点，让我们假设我们在新安装的 Ubuntu 服务器 LAMP 堆栈上有以下损坏的`index.php`文件：

```php
<?php

echo 'Test;

```

尝试在浏览器中打开时，Apache 将返回`HTTP 500 Internal Server Error`，这取决于浏览器，可能会对最终用户可见，如下面的屏幕截图所示：

![](img/4fab80d6-e7a8-43a9-b4e6-574dc6d457e6.jpg)

理想情况下，我们希望我们的 Web 服务器配置了一个漂亮的通用错误页面，以使其更加用户友好。虽然浏览器的响应可能会满足最终用户，但在这种情况下，它确实不能满足开发人员。返回的信息并没有指示任何关于错误性质的信息，这使得难以修复。幸运的是，对于我们来说，在这种情况下默认的 LAMP 堆栈配置包括将错误记录到`/var/log/apache2/error.log`文件中：

```php
[Thu Feb 02 19:23:26.026521 2017] [:error] [pid 5481] [client 93.140.71.25:55229] PHP Parse error: syntax error, unexpected ''Test;' (T_ENCAPSED_AND_WHITESPACE) in /var/www/html/index.php on line 3

```

虽然这种行为对于生产环境来说是完美的，但对于开发环境来说却很麻烦。在开发过程中，我们真的希望我们的错误能够显示在浏览器中，以加快速度。PHP 允许我们通过几个配置指令来控制错误报告和日志记录行为，以下是最重要的：

+   `error_reporting`：这是我们希望监视的错误级别；我们可以使用管道（`|`）运算符列出几个错误级别常量。它的默认值是`E_ALL & ~E_NOTICE & ~E_STRICT & ~E_DEPRECATED`。

+   `display_errors`：这指定是否将错误发送到浏览器/CLI 或对用户隐藏。

+   `error_log`：这是我们想要记录 PHP 错误的文件。

+   `log_errors`：这告诉我们是否应将错误记录到`error_log`文件中。

可用的错误级别常量定义如下：

+   `E_ERROR (1)`

+   `E_WARNING (2)`

+   `E_PARSE (4)`

+   `E_NOTICE (8)`

+   `E_CORE_ERROR (16)`

+   `E_CORE_WARNING (32)`

+   `E_COMPILE_ERROR (64)`

+   `E_COMPILE_WARNING (128)`

+   `E_USER_ERROR (256)`

+   `E_USER_WARNING (512)`

+   `E_USER_NOTICE (1024)`

+   `E_STRICT (2048)`

+   `E_RECOVERABLE_ERROR (4096)`

+   `E_DEPRECATED (8192)`

+   `E_USER_DEPRECATED (16384)`

+   `E_ALL (32767)`

使用`error_reporting()`和`ini_set()`函数，我们可以使用一些指令来配置日志和显示运行时的情况：

```php
<?php

error_reporting(E_ALL);
ini_set('display_errors', 'On');

```

小心使用`ini_set()`来设置`display_errors`，如果脚本有致命错误，它将不会产生任何效果，因为运行时不会被执行。

错误显示和错误日志记录是两种不同的机制，彼此协同工作。在开发环境中，我们可能更多地从错误显示中受益，而在生产环境中，错误日志记录是更好的选择。

# Zend OPcache

PHP 的一个主要缺点是它在每个请求上加载和解析 PHP 脚本。PHP 代码首先以纯文本形式编译为操作码，然后执行操作码。尽管这种性能影响在总共只有一个或几个脚本的小型应用中可能不会被注意到，但对于较大的平台（如 Magento、Drupal 等）来说，它会产生很大的影响。

从 PHP 5.5 开始，有一个开箱即用的解决方案。Zend OPcache 扩展通过将编译后的操作码存储在共享内存（RAM）中来解决重复编译的问题。只需更改配置指令即可打开或关闭它。

有很多配置指令，其中一些可以帮助我们入门：

+   `opcache.enable`：默认为 1，可通过`PHP_INI_ALL`更改。

+   `opcache.enable_cli`：默认为 0，可通过`PHP_INI_SYSTEM`更改。

+   `opcache.memory_consumption`：默认为 64，可通过`PHP_INI_SYSTEM`更改，定义了 OPcache 使用的共享内存大小。

+   `opcache.max_accelerated_files`：默认为 2000，可通过`PHP_INI_SYSTEM`更改，定义了 OPcache 哈希表中键/脚本的最大数量。其最大值为 1000000。

+   `opcache.max_wasted_percentage`：默认为 5，可通过`PHP_INI_SYSTEM`更改，定义了允许浪费内存的最大百分比，然后安排重新启动。

尽管`opcache.enable`标记为`PHP_INI_ALL`，但在运行时使用`ini_set()`启用它是行不通的。只有使用`ini_set()`来禁用它才有效。

尽管完全自动化，Zend OPcache 还为我们提供了一些函数：

+   `opcache_compile_file()`：这会编译并缓存一个脚本而不执行它

+   `opcache_get_configuration()`：这会获取 OPcache 配置信息

+   `opcache_get_status()`：这会获取 OPcache 信息

+   `opcache_invalidate()`：这会使 OPcache 失效

+   `opcache_is_script_cached()`：这告诉我们脚本是否通过 OPcache 缓存

+   `opcache_reset()`：这会重置 OPcache 缓存

虽然我们不太可能自己使用这些方法，但它们对于处理 OPcache 的实用工具非常有用。

opcache-gui 工具显示 OPcache 统计信息、设置和缓存文件，并提供实时更新。该工具可在[`github.com/amnuts/opcache-gui`](https://github.com/amnuts/opcache-gui)下载。

需要注意的一件事是 OPcache 潜在的*缓存冲击*问题。通过`memory_consumption`、`max_accelerated_files`和`max_wasted_percentage`配置指令，OPcache 确定何时需要刷新缓存。当这种情况发生时，具有大量流量的服务器可能会遇到缓存冲击问题，大量请求同时生成相同的缓存条目。因此，我们应该尽量避免频繁的缓存刷新。为此，我们可以使用缓存监控工具，并调整这三个配置指令以适应我们的应用程序大小。

# 并发性

并发性是一个适用于多层堆栈的主题，有一些关于 Web 服务器的配置指令，每个开发人员都应该熟悉。并发性指的是在 Web 服务器内处理多个连接。对于 PHP 来说，两个最受欢迎的 Web 服务器，Apache 和 Nginx，都允许一些基本配置来处理多个连接。

虽然有很多关于哪个服务器更快的争论，但带有 MPM 事件模块的 Apache 与 Nginx 的性能基本相当。

以下指令规定了 Apache MPM 事件并发性，因此值得密切关注：

+   `ThreadsPerChild`：这是每个子进程创建的线程数

+   `ServerLimit`：这是可配置的进程数量限制

+   `MaxRequestWorkers`：这是同时处理的最大连接数

+   `AsyncRequestWorkerFactor`：这是每个进程的并发连接限制

可以使用以下公式计算可能的最大并发连接数：

最大连接数=（AsyncRequestWorkerFactor + 1）* MaxRequestWorkers

这个公式非常简单；但是，改变`AsyncRequestWorkerFactor`不仅仅是输入更高的配置值。我们需要对击中 Web 服务器的流量有扎实的了解，这意味着进行广泛的测试和数据收集。

以下指令规定了 Nginx 的并发性，因此值得密切关注：

+   `worker_processes`：这是工作进程的数量；默认为 1

+   `worker_connections`：这是工作进程可以打开的最大同时连接数；默认为 512

Nginx 可以提供服务的理想总用户数可以归结为以下公式：

最大连接数=工作进程*工作连接数

虽然我们只是初步了解了 Web 服务器并发性和这两个 Web 服务器的总体配置指令，但上述信息应该作为我们的起点。虽然开发人员通常不会调整 Web 服务器，但他们应该知道何时标记可能影响其 PHP 应用程序性能的错误配置。

# 总结

在本章中，我们已经讨论了 PHP 性能优化的一些方面。虽然这些只是涉及整体性能主题的表面，但它们概述了每个 PHP 开发人员都应该深入了解的最常见领域。广泛的配置指令范围允许我们调整应用程序行为，通常与 Web 服务器本身协同工作。然而，最佳性能的支柱在于在整个堆栈中谨慎使用资源，正如我们通过简单的 SQL 查询示例所观察到的。

接下来，我们将研究无服务器架构，这是标准开发环境的新兴抽象。
