# 第十二章：改进 Web 安全

在本章中，我们将涵盖以下主题：

+   过滤`$_POST`数据

+   验证`$_POST`数据

+   保护 PHP 会话

+   使用令牌保护表单

+   构建一个安全的密码生成器

+   使用 CAPTCHA 保护表单

+   加密/解密而不使用`mcrypt`

# 介绍

在本章中，我们将向您展示如何建立一个简单而有效的机制，用于过滤和验证一块发布数据。然后，我们将介绍如何保护您的 PHP 会话免受潜在的会话劫持和其他形式的攻击。下一个配方将展示如何使用随机生成的令牌保护表单免受**跨站点请求伪造**（**CSRF**）攻击。有关密码生成的配方将向您展示如何将 PHP 7 真正的随机化结合起来生成安全密码。然后，我们将向您展示两种形式的**CAPTCHA**：一种是基于文本的，另一种是使用扭曲图像的。最后，有一个配方涵盖了强加密，而不使用被废弃和即将被弃用的`mcrypt`扩展。

# 过滤`$_POST`数据

过滤数据的过程可以包括以下任何或所有内容：

+   删除不需要的字符（即删除`<script>`标签）

+   对数据执行转换（即将引号转换为`&quot;`）

+   加密或解密数据

加密在本章的最后一个配方中有所涵盖。否则，我们将介绍一个基本机制，可用于过滤表单提交后到达的`$_POST`数据。

## 如何做...

1.  首先，您需要了解将出现在`$_POST`中的数据。而且，也许更重要的是，您需要了解表单数据将被存储的数据库表所施加的限制。例如，看一下`prospects`表的数据库结构：

```php
COLUMN          TYPE              NULL   DEFAULT
first_name      varchar(128)      No     None     NULL
last_name       varchar(128)      No     None     NULL
address         varchar(256)      Yes    None     NULL
city            varchar(64)       Yes    None     NULL
state_province  varchar(32)       Yes    None     NULL
postal_code     char(16)          No     None     NULL
phone           varchar(16)       No     None     NULL
country         char(2)           No     None     NULL
email           varchar(250)      No     None     NULL
status          char(8)           Yes    None     NULL
budget          decimal(10,2)     Yes    None     NULL
last_updated    datetime          Yes    None     NULL
```

1.  完成对要发布和存储的数据的分析后，可以确定要发生的过滤类型，以及哪些 PHP 函数将用于此目的。

1.  例如，如果您需要摆脱用户提供的表单数据中的前导和尾随空格，这是完全可能的，您可以使用 PHP 的`trim()`函数。根据数据库结构，所有字符数据都有长度限制。因此，您可能需要考虑使用`substr()`来确保长度不超过。如果您想要删除非字母字符，您可能需要考虑使用`preg_replace()`与适当的模式。

1.  现在，我们可以将所需的 PHP 函数集合成一个回调函数的单一数组。以下是一个基于表单数据过滤需求的示例，最终将存储在`prospects`表中：

```php
$filter = [
  'trim' => function ($item) { return trim($item); },
  'float' => function ($item) { return (float) $item; },
  'upper' => function ($item) { return strtoupper($item); },
  'email' => function ($item) { 
     return filter_var($item, FILTER_SANITIZE_EMAIL); },
  'alpha' => function ($item) { 
     return preg_replace('/[^A-Za-z]/', '', $item); },
  'alnum' => function ($item) { 
     return preg_replace('/[⁰-9A-Za-z ]/', '', $item); },
  'length' => function ($item, $length) { 
     return substr($item, 0, $length); },
  'stripTags' => function ($item) { return strip_tags($item); },
];
```

1.  接下来，我们定义一个与`$_POST`中预期的字段名称匹配的数组。在此数组中，我们指定`$filter`数组中的键，以及任何参数。请注意第一个键`*`。我们将使用它作为应用于所有字段的通配符：

```php
$assignments = [
  '*'             => ['trim' => NULL, 'stripTags' => NULL],
  'first_name'    => ['length' => 32, 'alnum' => NULL],
  'last_name'     => ['length' => 32, 'alnum' => NULL],
  'address'       => ['length' => 64, 'alnum' => NULL],
  'city'          => ['length' => 32],
  'state_province'=> ['length' => 20],
  'postal_code'   => ['length' => 12, 'alnum' => NULL],
  'phone'         => ['length' => 12],
  'country'       => ['length' => 2, 'alpha' => NULL, 
                      'upper' => NULL],
  'email'         => ['length' => 128, 'email' => NULL],
  'budget'        => ['float' => NULL],
];
```

1.  然后，我们循环遍历数据集（即来自`$_POST`）并依次应用回调。我们首先运行分配给通配符（`*`）键的所有回调。

### 注意

重要的是要实现通配符过滤器以避免冗余设置。在前面的示例中，我们希望应用代表 PHP 函数`strip_tags()`和`trim()`的过滤器到每个项目。

1.  接下来，我们运行分配给特定数据字段的所有回调。完成后，`$data`中的所有值都将被过滤：

```php
foreach ($data as $field => $item) {
  foreach ($assignments['*'] as $key => $option) {
    $item = $filter$key;
  }
  foreach ($assignments[$field] as $key => $option) {
    $item = $filter$key;
  }
}
```

## 工作原理...

将步骤 4 到 6 中显示的代码放入一个名为`chap_12_post_data_filtering_basic.php`的文件中。您还需要定义一个数组来模拟`$_POST`中可能存在的数据。在这种情况下，您可以定义两个数组，一个包含*好*数据，另一个包含*坏*数据：

```php
$testData = [
  'goodData'   => [
    'first_name'    => 'Doug',
    'last_name'     => 'Bierer',
    'address'       => '123 Main Street',
    'city'          => 'San Francisco',
    'state_province'=> 'California',
    'postal_code'   => '94101',
    'phone'         => '+1 415-555-1212',
    'country'       => 'US',
    'email'         => 'doug@unlikelysource.com',
    'budget'        => '123.45',
  ],
  'badData' => [
    'first_name' => 'This+Name<script>bad tag</script>Valid!',
    'last_name' 	=> 'ThisLastNameIsWayTooLongAbcdefghijklmnopqrstuvwxyz0123456789Abcdefghijklmnopqrstuvwxyz0123456789Abcdefghijklmnopqrstuvwxyz0123456789Abcdefghijklmnopqrstuvwxyz0123456789',
    //'address' 	=> '',    // missing
    'city'      => 'ThisCityNameIsTooLong012345678901234567890123456789012345678901234567890123456789  ',
    //'state_province'=> '',    // missing
    'postal_code'     => '!"£$%^Non Alpha Chars',
    'phone'           => ' 12345 ',
    'country'         => '12345',
    'email'           => 'this.is@not@an.email',
    'budget'          => 'XXX',
  ]
];
```

最后，您需要循环遍历过滤器分配，呈现好的和坏的数据：

```php
foreach ($testData as $data) {
  foreach ($data as $field => $item) {
    foreach ($assignments['*'] as $key => $option) {
      $item = $filter$key;
    }
    foreach ($assignments[$field] as $key => $option) {
      $item = $filter$key;
    }
    printf("%16s : %s\n", $field, $item);
  }
}
```

这是此示例的输出可能如何出现的：

![工作原理...](img/B05314_12_01.jpg)

请注意，名称已被截断，标签已被删除。 您还会注意到，尽管电子邮件地址已经被过滤，但它仍然不是一个有效的地址。 需要注意的是，为了正确处理数据，可能需要*验证*和过滤。

## 另请参阅

在第六章中，*构建可扩展的网站*，名为*链接$_POST 过滤器*的示例讨论了如何将此处介绍的基本过滤概念合并到全面的过滤器链接机制中。

# 验证$_POST 数据

过滤和验证之间的主要区别在于后者不会更改原始数据。 另一个区别在于意图。 验证的目的是确认数据是否符合根据客户需求建立的某些标准。

## 如何做...

1.  我们将在这里介绍的基本验证机制与前面的示例相同。 与过滤一样，了解要验证的数据的性质，它如何符合客户的要求，以及它是否符合数据库强制执行的标准是至关重要的。 例如，如果在数据库中，列的最大宽度为 128，则验证回调可以使用`strlen()`来确认提交的数据的长度是否小于或等于 128 个字符。 同样，您可以使用`ctype_alnum()`来确认数据是否只包含适当的字母和数字。

1.  验证的另一个考虑因素是提供适当的验证失败消息。 在某种意义上，验证过程也是一个*确认*过程，某人可能会审查验证以确认成功或失败。 如果验证失败，该人将需要知道失败的原因。

1.  对于本示例，我们将再次专注于`prospects`表。 现在，我们可以将一组所需的 PHP 函数集合到一个回调函数的数组中。 以下是基于表单数据的验证需求的示例，这些数据最终将存储在`prospects`表中：

```php
$validator = [
  'email' => [
    'callback' => function ($item) { 
      return filter_var($item, FILTER_VALIDATE_EMAIL); },
    'message'  => 'Invalid email address'],
  'alpha' => [
    'callback' => function ($item) { 
      return ctype_alpha(str_replace(' ', '', $item)); },
    'message'  => 'Data contains non-alpha characters'],
  'alnum' => [
    'callback' => function ($item) { 
      return ctype_alnum(str_replace(' ', '', $item)); },
    'message'  => 'Data contains characters which are '
       . 'not letters or numbers'],
  'digits' => [
    'callback' => function ($item) { 
      return preg_match('/[⁰-9.]/', $item); },
    'message'  => 'Data contains characters which '
      . 'are not numbers'],
  'length' => [
    'callback' => function ($item, $length) { 
      return strlen($item) <= $length; },
    'message'  => 'Item has too many characters'],
  'upper' => [
    'callback' => function ($item) { 
      return $item == strtoupper($item); },
    'message'  => 'Item is not upper case'],
  'phone' => [
    'callback' => function ($item) { 
      return preg_match('/[⁰-9() -+]/', $item); },
    'message'  => 'Item is not a valid phone number'],
];
```

### 注意

请注意，对于 alpha 和 alnum 回调，我们通过首先使用`str_replace()`来允许空格。 然后我们可以调用`ctype_alpha()`或`ctype_alnum()`，这将确定是否存在任何不允许的字符。

1.  接下来，我们定义一个分配数组，与`$_POST`中预期的字段名称匹配。 在此数组中，我们指定`$validator`数组中的键，以及任何参数：

```php
$assignments = [
  'first_name'    => ['length' => 32, 'alpha' => NULL],
  'last_name'     => ['length' => 32, 'alpha' => NULL],
  'address'       => ['length' => 64, 'alnum' => NULL],
  'city'          => ['length' => 32, 'alnum' => NULL],
  'state_province'=> ['length' => 20, 'alpha' => NULL],
  'postal_code'   => ['length' => 12, 'alnum' => NULL],
  'phone'         => ['length' => 12, 'phone' => NULL],
  'country'       => ['length' => 2, 'alpha' => NULL, 
                      'upper' => NULL],
  'email'         => ['length' => 128, 'email' => NULL],
  'budget'        => ['digits' => NULL],
];
```

1.  然后，我们使用嵌套的`foreach()`循环逐个字段地遍历数据块。 对于每个字段，我们循环遍历分配给该字段的回调函数：

```php
foreach ($data as $field => $item) {
  echo 'Processing: ' . $field . PHP_EOL;
  foreach ($assignments[$field] as $key => $option) {
    if ($validator[$key]'callback') {
        $message = 'OK';
    } else {
        $message = $validator[$key]['message'];
    }
    printf('%8s : %s' . PHP_EOL, $key, $message);
  }
}
```

### 提示

而不是直接回显输出，如所示，您可以记录验证成功/失败的结果，以便在以后向审阅人呈现。 此外，如第六章中所示，*构建可扩展的网站*，您可以将验证机制集成到表单中，显示验证消息与其匹配的表单元素旁边。

## 工作原理...

将步骤 3 到 5 中显示的代码放入名为`chap_12_post_data_validation_basic.php`的文件中。 您还需要定义一个数据数组，模拟将出现在`$_POST`中的数据。 在这种情况下，您使用前面示例中提到的两个数组，一个包含*好*数据，另一个包含*坏*数据。 最终输出应该看起来像这样：

![工作原理...](img/B05314_12_02.jpg)

## 另请参阅

+   在第六章中，*构建可扩展的网站*，名为*链接$_POST 验证器*的示例讨论了如何将此处介绍的基本验证概念合并到全面的过滤器链接机制中。

# 保护 PHP 会话

PHP 会话机制非常简单。一旦使用`session_start()`或`php.ini session.autostart`设置开始会话，PHP 引擎会生成一个唯一的令牌，默认情况下通过 cookie 传递给用户。在后续请求中，当会话仍然被视为活动状态时，用户的浏览器（或等效物）再次通常通过 cookie 呈现会话标识符进行检查。然后，PHP 引擎使用此标识符定位服务器上的适当文件，并使用存储的信息填充`$_SESSION`。当会话标识符是识别返回的网站访问者的唯一手段时，存在巨大的安全问题。在本教程中，我们将介绍几种技术，这些技术将帮助您保护会话，从而大大提高网站的整体安全性。

## 如何做...

1.  首先，重要的是要认识到将会话作为唯一的身份验证手段可能是危险的。想象一下，当有效用户登录到您的网站时，您在`$_SESSION`中设置了一个`loggedIn`标志：

```php
session_start();
$loggedIn = $_SESSION['isLoggedIn'] ?? FALSE;
if (isset($_POST['login'])) {
  if ($_POST['username'] == // username lookup
      && $_POST['password'] == // password lookup) {
      $loggedIn = TRUE;
      $_SESSION['isLoggedIn'] = TRUE;
  }
}
```

1.  在您的程序逻辑中，如果`$_SESSION['isLoggedIn']`设置为`TRUE`，则允许用户查看敏感信息：

```php
<br>Secret Info
<br><?php if ($loggedIn) echo // secret information; ?>
```

1.  如果攻击者能够获取会话标识符，例如，通过成功执行的**跨站脚本**（**XSS**）攻击，他/她只需要将`PHPSESSID` cookie 的值设置为非法获取的值，他们现在被您的应用程序视为有效用户。

1.  缩小`PHPSESSID`有效时间窗口的一种快速简单方法是使用`session_regenerate_id()`。这个非常简单的命令生成一个新的会话标识符，使旧的会话标识符无效，保持会话数据完整，并对性能影响很小。此命令只能在会话开始后执行：

```php
session_start();
session_regenerate_id();
```

1.  另一个经常被忽视的技术是确保网站访问者有注销选项。然而，重要的是不仅使用`session_destroy()`销毁会话，还要取消`$_SESSION`数据并使会话 cookie 过期：

```php
session_unset();
session_destroy();
setcookie('PHPSESSID', 0, time() - 3600);
```

1.  另一种可以用来防止会话劫持的简单技术是开发网站访问者的指纹。实现这种技术的一种方法是收集与会话标识符不同的网站访问者的唯一信息。这些信息包括用户代理（即浏览器）、接受的语言和远程 IP 地址。您可以从这些信息中派生出一个简单的哈希，并将哈希存储在服务器上的一个单独文件中。下次用户访问网站时，如果您已经确定他们基于会话信息已登录，那么您可以通过匹配指纹进行二次验证：

```php
$remotePrint = md5($_SERVER['REMOTE_ADDR'] 
                   . $_SERVER['HTTP_USER_AGENT'] 
                   . $_SERVER['HTTP_ACCEPT_LANGUAGE']);
$printsMatch = file_exists(THUMB_PRINT_DIR . $remotePrint);
if ($loggedIn && !$printsMatch) {
    $info = 'SESSION INVALID!!!';
    error_log('Session Invalid: ' . date('Y-m-d H:i:s'), 0);
    // take appropriate action
}
```

### 注意

我们使用`md5()`作为快速哈希算法，非常适合内部使用。*不建议*在任何外部使用中使用`md5()`，因为它容易受到暴力攻击。

## 它是如何工作的...

为了演示会话的漏洞性，编写一个简单的登录脚本，成功登录后设置`$_SESSION[`'`isLoggedIn`'`] flag`。您可以将文件命名为`chap_12_session_hijack.php`：

```php
session_start();
$loggedUser = $_SESSION['loggedUser'] ?? '';
$loggedIn = $_SESSION['isLoggedIn'] ?? FALSE;
$username = 'test';
$password = 'password';
$info = 'You Can Now See Super Secret Information!!!';

if (isset($_POST['login'])) {
  if ($_POST['username'] == $username
      && $_POST['password'] == $password) {
        $loggedIn = TRUE;
        $_SESSION['isLoggedIn'] = TRUE;
        $_SESSION['loggedUser'] = $username;
        $loggedUser = $username;
  }
} elseif (isset($_POST['logout'])) {
  session_destroy();
}
```

然后，您可以添加显示简单登录表单的代码。要测试会话漏洞，请按照使用我们刚刚创建的`chap_12_session_hijack.php`文件的步骤：

1.  切换到包含文件的目录。

1.  运行`php -S localhost:8080`命令。

1.  使用一个浏览器，打开 URL `http://localhost:8080/<filename>`。

1.  使用用户名`test`和密码`password`登录。

1.  您应该能够看到**您现在可以查看超级秘密信息!!!**。

1.  刷新页面：每次都应该看到一个新的会话标识符。

1.  复制`PHPSESSID` cookie 的值。

1.  在同一个网页上用另一个浏览器打开。

1.  通过复制`PHPSESSID`的值修改浏览器发送的 cookie。

为了说明，我们还在以下截图中使用 Vivaldi 浏览器显示了`$_COOKIE`和`$_SESSION`的值：

![它是如何工作的...](img/B05314_12_03.jpg)

然后我们复制`PHPSESSID`的值，打开 Firefox 浏览器，并使用一个名为 Tamper Data 的工具来修改 cookie 的值：

![它是如何工作的...](img/B05314_12_04.jpg)

您可以在下一个截图中看到，我们现在是经过身份验证的用户，而无需输入用户名或密码：

![它是如何工作的...](img/B05314_12_05.jpg)

现在，您可以实施前面步骤中讨论的更改。将先前创建的文件复制到`chap_12_session_protected.php`。现在继续重新生成会话 ID：

```php
<?php
define('THUMB_PRINT_DIR', __DIR__ . '/../data/');
session_start();
session_regenerate_id();
```

接下来，初始化变量并确定登录状态（与以前一样）：

```php
$username = 'test';
$password = 'password';
$info = 'You Can Now See Super Secret Information!!!';
$loggedIn = $_SESSION['isLoggedIn'] ?? FALSE;
$loggedUser = $_SESSION['user'] ?? 'guest';
```

您可以使用远程地址、用户代理和语言设置添加会话指纹：

```php
$remotePrint = md5($_SERVER['REMOTE_ADDR']
  . $_SERVER['HTTP_USER_AGENT']
  . $_SERVER['HTTP_ACCEPT_LANGUAGE']);
$printsMatch = file_exists(THUMB_PRINT_DIR . $remotePrint);
```

如果登录成功，我们将会话中的指纹信息和登录状态存储起来：

```php
if (isset($_POST['login'])) {
  if ($_POST['username'] == $username
      && $_POST['password'] == $password) {
        $loggedIn = TRUE;
        $_SESSION['user'] = strip_tags($username);
        $_SESSION['isLoggedIn'] = TRUE;
        file_put_contents(
          THUMB_PRINT_DIR . $remotePrint, $remotePrint);
  }
```

您还可以检查注销选项并实施适当的注销过程：取消设置`$_SESSION`变量，使会话无效，并使 cookie 过期。您还可以删除指纹文件并实施重定向：

```php
} elseif (isset($_POST['logout'])) {
  session_unset();
  session_destroy();
  setcookie('PHPSESSID', 0, time() - 3600);
  if (file_exists(THUMB_PRINT_DIR . $remotePrint)) 
    unlink(THUMB_PRINT_DIR . $remotePrint);
    header('Location: ' . $_SERVER['REQUEST_URI'] );
  exit;
```

否则，如果操作不是登录或注销，您可以检查用户是否被视为已登录，如果指纹不匹配，则会话被视为无效，并采取适当的操作：

```php
} elseif ($loggedIn && !$printsMatch) {
    $info = 'SESSION INVALID!!!';
    error_log('Session Invalid: ' . date('Y-m-d H:i:s'), 0);
    // take appropriate action
}
```

现在，您可以使用新的`chap_12_session_protected.php`文件运行与之前提到的相同的过程。您将首先注意到的是会话现在被视为无效。输出将类似于这样：

![它是如何工作的...](img/B05314_12_06.jpg)

原因是指纹不匹配，因为您现在正在使用不同的浏览器。同样，如果您刷新第一个浏览器的页面，会话标识符将被重新生成，使任何先前复制的标识符都将变得过时。最后，注销按钮将完全清除会话信息。

## 另请参阅

有关网站漏洞的出色概述，请参阅[`www.owasp.org/index.php/Category:Vulnerability`](https://www.owasp.org/index.php/Category:Vulnerability)上的文章。有关会话劫持的信息，请参阅[`www.owasp.org/index.php/Session_hijacking_attack`](https://www.owasp.org/index.php/Session_hijacking_attack)。

# 使用令牌保护表单

这个配方提供了另一种非常简单的技术，可以保护您的表单免受**跨站点请求伪造**（**CSRF**）攻击。简而言之，当攻击者可能使用其他技术时，可以在您网站上的网页上感染一个 CSRF 攻击。在大多数情况下，受感染的页面将开始发出请求（即使用 JavaScript 购买物品或进行设置更改）使用有效的已登录用户的凭据。您的应用程序极其难以检测到这种活动。可以采取的一个措施是生成一个随机令牌，该令牌包含在要提交的每个表单中。由于受感染的页面将无法访问令牌，也无法生成与之匹配的令牌，因此表单验证将失败。

## 如何做...

1.  首先，为了演示问题，我们创建一个模拟受感染页面的网页，该页面生成一个请求以将条目发布到数据库。为此示例，我们将文件命名为`chap_12_form_csrf_test_unprotected.html`：

```php
<!DOCTYPE html>
  <body onload="load()">
  <form action="/chap_12_form_unprotected.php" 
    method="post" id="csrf_test" name="csrf_test">
    <input name="name" type="hidden" value="No Goodnick" />
    <input name="email" type="hidden" value="malicious@owasp.org" />
    <input name="comments" type="hidden" 
       value="Form is vulnerable to CSRF attacks!" />
    <input name="process" type="hidden" value="1" />
  </form>
  <script>
    function load() { document.forms['csrf_test'].submit(); }
  </script>
</body>
</html>
```

1.  接下来，我们创建一个名为`chap_12_form_unprotected.php`的脚本，用于响应表单提交。与本书中的其他调用程序一样，我们设置自动加载并使用第五章中介绍的`Application\Database\Connection`类，*与数据库交互*。

```php
<?php
define('DB_CONFIG_FILE', '/../config/db.config.php');
require __DIR__ . '/../Application/Autoload/Loader.php';
Application\Autoload\Loader::init(__DIR__ . '/..');
use Application\Database\Connection;
$conn = new Connection(include __DIR__ . DB_CONFIG_FILE);
```

1.  然后我们检查处理按钮是否已被按下，并实施一个过滤机制，如本章中*过滤$_POST 数据*食谱中所述。这是为了证明 CSRF 攻击很容易绕过过滤器：

```php
if ($_POST['process']) {
    $filter = [
      'trim' => function ($item) { return trim($item); },
      'email' => function ($item) { 
        return filter_var($item, FILTER_SANITIZE_EMAIL); },
      'length' => function ($item, $length) { 
        return substr($item, 0, $length); },
      'stripTags' => function ($item) { 
      return strip_tags($item); },
  ];

  $assignments = [
    '*'         => ['trim' => NULL, 'stripTags' => NULL],
    'email'   => ['length' => 249, 'email' => NULL],
    'name'    => ['length' => 128],
    'comments'=> ['length' => 249],
  ];

  $data = $_POST;
  foreach ($data as $field => $item) {
    foreach ($assignments['*'] as $key => $option) {
      $item = $filter$key;
    }
    if (isset($assignments[$field])) {
      foreach ($assignments[$field] as $key => $option) {
        $item = $filter$key;
      }
      $filteredData[$field] = $item;
    }
  }
```

1.  最后，我们使用预处理语句将过滤后的数据插入数据库。然后重定向到另一个名为`chap_12_form_view_results.php`的脚本，该脚本简单地转储`visitors`表的内容：

```php
try {
    $filteredData['visit_date'] = date('Y-m-d H:i:s');
    $sql = 'INSERT INTO visitors '
        . ' (email,name,comments,visit_date) '
        . 'VALUES (:email,:name,:comments,:visit_date)';
    $insertStmt = $conn->pdo->prepare($sql);
    $insertStmt->execute($filteredData);
} catch (PDOException $e) {
    echo $e->getMessage();
}
}
header('Location: /chap_12_form_view_results.php');
exit;
```

1.  当然，结果是，尽管进行了过滤并使用了预处理语句，攻击仍然被允许。

1.  实际上，实现表单保护令牌非常容易！首先，您需要生成令牌并将其存储在会话中。我们利用新的`random_bytes()` PHP 7 函数来生成一个真正随机的令牌，这个令牌对于攻击者来说将是困难的，如果不是不可能的匹配：

```php
session_start();
$token = urlencode(base64_encode((random_bytes(32))));
$_SESSION['token'] = $token;
```

### 注意

`random_bytes()`的输出是二进制的。我们使用`base64_encode()`将其转换为可用的字符串。然后我们使用`urlencode()`进一步处理它，以便在 HTML 表单中正确呈现。

1.  当我们呈现表单时，我们将令牌呈现为隐藏字段：

```php
<input type="hidden" name="token" value="<?= $token ?>" />
```

1.  然后，我们复制并修改先前提到的`chap_12_form_unprotected.php`脚本，添加逻辑来首先检查会话中存储的令牌是否匹配。请注意，我们取消当前令牌以使其对将来的使用无效。我们将新脚本命名为`chap_12_form_protected_with_token.php`：

```php
if ($_POST['process']) {
    $sessToken = $_SESSION['token'] ?? 1;
    $postToken = $_POST['token'] ?? 2;
    unset($_SESSION['token']);
    if ($sessToken != $postToken) {
        $_SESSION['message'] = 'ERROR: token mismatch';
    } else {
        $_SESSION['message'] = 'SUCCESS: form processed';
        // continue with form processing
    }
}
```

## 它是如何工作的...

为了测试感染的网页如何发起 CSRF 攻击，创建以下文件，如前面的示例所示：

+   `chap_12_form_csrf_test_unprotected.html`

+   `chap_12_form_unprotected.php`

然后，您可以定义一个名为`chap_12_form_view_results.php`的文件，其中转储`visitors`表的内容：

```php
<?php
session_start();
define('DB_CONFIG_FILE', '/../config/db.config.php');
require __DIR__ . '/../Application/Autoload/Loader.php';
Application\Autoload\Loader::init(__DIR__ . '/..');
use Application\Database\Connection;
$conn = new Connection(include __DIR__ . DB_CONFIG_FILE);
$message = $_SESSION['message'] ?? '';
unset($_SESSION['message']);
$stmt = $conn->pdo->query('SELECT * FROM visitors');
?>
<!DOCTYPE html>
<body>
<div class="container">
  <h1>CSRF Protection</h1>
  <h3>Visitors Table</h3>
  <?php while ($row = $stmt->fetch(PDO::FETCH_ASSOC)) : ?>
  <pre><?php echo implode(':', $row); ?></pre>
  <?php endwhile; ?>
  <?php if ($message) : ?>
  <b><?= $message; ?></b>
  <?php endif; ?>
</div>
</body>
</html>
```

从浏览器中启动`chap_12_form_csrf_test_unprotected.html`。以下是输出可能会出现的样子：

![它是如何工作的...](img/B05314_12_07.jpg)

正如您所看到的，尽管进行了过滤并使用了预处理语句，攻击仍然成功！

接下来，将`chap_12_form_unprotected.php`文件复制到`chap_12_form_protected.php`。按照食谱中的第 8 步所示进行更改。您还需要修改测试 HTML 文件，将`chap_12_form_csrf_test_unprotected.html`复制到`chap_12_form_csrf_test_protected.html`。将`FORM`标签中的 action 参数的值更改如下：

```php
<form action="/chap_12_form_protected_with_token.php" 
  method="post" id="csrf_test" name="csrf_test">
```

当您从浏览器运行新的 HTML 文件时，它会调用`chap_12_form_protected.php`，该文件寻找一个不存在的令牌。以下是预期的输出：

![它是如何工作的...](img/B05314_12_08.jpg)

最后，继续定义一个名为`chap_12_form_protected.php`的文件，生成一个令牌并将其显示为隐藏元素：

```php
<?php
session_start();
$token = urlencode(base64_encode((random_bytes(32))));
$_SESSION['token'] = $token;
?>
<!DOCTYPE html>
<body onload="load()">
<div class="container">
<h1>CSRF Protected Form</h1>
<form action="/chap_12_form_protected_with_token.php" 
     method="post" id="csrf_test" name="csrf_test">
<table>
<tr><th>Name</th><td><input name="name" type="text" /></td></tr>
<tr><th>Email</th><td><input name="email" type="text" /></td></tr>
<tr><th>Comments</th><td>
<input name="comments" type="textarea" rows=4 cols=80 />
</td></tr>
<tr><th>&nbsp;</th><td>
<input name="process" type="submit" value="Process" />
</td></tr>
</table>
<input type="hidden" name="token" value="<?= $token ?>" />
</form>
<a href="/chap_12_form_view_results.php">
    CLICK HERE</a> to view results
</div>
</body>
</html>
```

当我们显示并提交表单中的数据时，将验证令牌并允许数据插入继续进行，如下所示：

![它是如何工作的...](img/B05314_12_09.jpg)

## 另请参阅

有关 CSFR 攻击的更多信息，请参阅[`www.owasp.org/index.php/Cross-Site_Request_Forgery_(CSRF)`](https://www.owasp.org/index.php/Cross-Site_Request_Forgery_(CSRF))。

# 构建一个安全的密码生成器

一个常见的误解是，攻击者破解哈希密码的唯一方法是使用**暴力攻击**和**彩虹表**。虽然这通常是攻击序列中的第一步，但攻击者会在第二、第三或第四步使用更复杂的攻击。其他攻击包括*组合*、*字典*、*掩码*和基于规则的攻击。字典攻击使用来自字典的单词数据库来猜测密码。组合是指组合字典中的单词。掩码攻击类似于暴力攻击，但更有选择性，因此缩短了破解时间。基于规则的攻击将检测诸如用数字 0 替换字母 o 之类的事情。

好消息是，通过简单地增加密码长度超过六个字符的长度，可以指数级增加破解哈希密码所需的时间。其他因素，例如随机将大写字母与小写字母交错、随机数字和特殊字符，也会对破解所需的时间产生指数影响。最终，我们需要牢记的是，最终需要有人输入创建的密码，这意味着密码至少需要稍微记忆。

### 提示

**最佳实践**

密码应该以哈希形式存储，而不是明文。MD5 和 SHA*不再被认为是安全的（尽管 SHA*比 MD5 好得多）。使用诸如`oclHashcat`之类的实用程序，攻击者可以在密码使用 MD5 哈希后通过漏洞（即成功的 SQL 注入攻击）生成平均每秒 55 亿次尝试。

## 如何做...

1.  首先，我们定义一个`Application\Security\PassGen`类，该类将包含生成密码所需的方法。我们还定义了一些类常量和属性，这些常量和属性将作为流程的一部分使用：

```php
namespace Application\Security;
class PassGen
{
  const SOURCE_SUFFIX = 'src';
  const SPECIAL_CHARS = 
    '\`¬|!"£$%^&*()_-+={}[]:@~;\'#<>?,./|\\';
  protected $algorithm;
  protected $sourceList;
  protected $word;
  protected $list;
```

1.  然后我们定义将用于生成密码的低级方法。正如名称所示，`digits()`生成随机数字，`special()`从`SPECIAL_CHARS`类常量中生成一个字符：

```php
public function digits($max = 999)
{
  return random_int(1, $max);
}

public function special()
{
  $maxSpecial = strlen(self::SPECIAL_CHARS) - 1;
  return self::SPECIAL_CHARS[random_int(0, $maxSpecial)];
}
```

### 注意

请注意，在此示例中，我们经常使用新的 PHP 7 函数`random_int()`。尽管稍慢，但与更陈旧的`rand()`函数相比，这种方法提供了真正的**密码安全伪随机数生成器**（**CSPRNG**）功能。

1.  现在是棘手的部分：生成一个难以猜测的单词。这就是`$wordSource`构造函数参数发挥作用的地方。它是一个网站数组，我们的单词库将从中派生。因此，我们需要一个方法，该方法将从指定的来源中提取一个唯一的单词列表，并将结果存储在文件中。我们将`$wordSource`数组作为参数接受，并循环遍历每个 URL。我们使用`md5()`生成网站名称的哈希值，然后将其构建成文件名。然后将新生成的文件名存储在`$sourceList`中：

```php
public function processSource(
$wordSource, $minWordLength, $cacheDir)
{
  foreach ($wordSource as $html) {
    $hashKey = md5($html);
    $sourceFile = $cacheDir . '/' . $hashKey . '.' 
    . self::SOURCE_SUFFIX;
    $this->sourceList[] = $sourceFile;
```

1.  如果文件不存在或为空字节，我们处理内容。如果来源是 HTML，我们只接受`<body>`标签内的内容。然后我们使用`str_word_count()`从字符串中提取单词列表，同时使用`strip_tags()`去除任何标记：

```php
if (!file_exists($sourceFile) || filesize($sourceFile) == 0) {
    echo 'Processing: ' . $html . PHP_EOL;
    $contents = file_get_contents($html);
    if (preg_match('/<body>(.*)<\/body>/i', 
        $contents, $matches)) {
        $contents = $matches[1];
    }
    $list = str_word_count(strip_tags($contents), 1);
```

1.  然后我们删除任何太短的单词，并使用`array_unique()`去除重复项。最终结果存储在文件中：

```php
     foreach ($list as $key => $value) {
       if (strlen($value) < $minWordLength) {
         $list[$key] = 'xxxxxx';
       } else {
         $list[$key] = trim($value);
       }
     }
     $list = array_unique($list);
     file_put_contents($sourceFile, implode("\n",$list));
   }
  }
  return TRUE;
}
```

1.  接下来，我们定义一个*翻转*单词中随机字母为大写的方法：

```php
public function flipUpper($word)
{
  $maxLen   = strlen($word);
  $numFlips = random_int(1, $maxLen - 1);
  $flipped  = strtolower($word);
  for ($x = 0; $x < $numFlips; $x++) {
       $pos = random_int(0, $maxLen - 1);
       $word[$pos] = strtoupper($word[$pos]);
  }
  return $word;
}
```

1.  最后，我们准备定义一个从我们的来源选择单词的方法。我们随机选择一个单词来源，并使用`file()`函数从适当的缓存文件中读取：

```php
public function word()
{
  $wsKey    = random_int(0, count($this->sourceList) - 1);
  $list     = file($this->sourceList[$wsKey]);
  $maxList  = count($list) - 1;
  $key      = random_int(0, $maxList);
  $word     = $list[$key];
  return $this->flipUpper($word);
}
```

1.  为了不总是生成相同模式的密码，我们定义了一个方法，允许我们将密码的各个组件放置在最终密码字符串的不同位置。算法被定义为此类中可用的方法调用数组。例如，一个`['word', 'digits', 'word', 'special']`的算法最终可能看起来像`hElLo123aUTo!`：

```php
public function initAlgorithm()
{
  $this->algorithm = [
    ['word', 'digits', 'word', 'special'],
    ['digits', 'word', 'special', 'word'],
    ['word', 'word', 'special', 'digits'],
    ['special', 'word', 'special', 'digits'],
    ['word', 'special', 'digits', 'word', 'special'],
    ['special', 'word', 'special', 'digits', 
    'special', 'word', 'special'],
  ];
}
```

1.  构造函数接受单词来源数组、最小单词长度和缓存目录的位置。然后处理源文件并初始化算法：

```php
public function __construct(
  array $wordSource, $minWordLength, $cacheDir)
{
  $this->processSource($wordSource, $minWordLength, $cacheDir);
  $this->initAlgorithm();
}
```

1.  最后，我们能够定义实际生成密码的方法。它只需要随机选择一个算法，然后循环调用适当的方法：

```php
public function generate()
{
  $pwd = '';
  $key = random_int(0, count($this->algorithm) - 1);
  foreach ($this->algorithm[$key] as $method) {
    $pwd .= $this->$method();
  }
  return str_replace("\n", '', $pwd);
}

}
```

## 工作原理...

首先，您需要将前面一篇文章中描述的代码放入`Application\Security`文件夹中名为`PassGen.php`的文件中。现在您可以创建一个名为`chap_12_password_generate.php`的调用程序，设置自动加载，使用`PassGen`，并定义缓存目录的位置：

```php
<?php
define('CACHE_DIR', __DIR__ . '/cache');
require __DIR__ . '/../Application/Autoload/Loader.php';
Application\Autoload\Loader::init(__DIR__ . '/..');
use Application\Security\PassGen;
```

接下来，您需要定义一个网站数组，用作密码生成中的单词库的来源。在这个例子中，我们将从 Project Gutenberg 的文本《尤利西斯》（J.乔伊斯）、《战争与和平》（L.托尔斯泰）和《傲慢与偏见》（J.奥斯汀）中进行选择：

```php
$source = [
  'https://www.gutenberg.org/files/4300/4300-0.txt',
  'https://www.gutenberg.org/files/2600/2600-h/2600-h.htm',
  'https://www.gutenberg.org/files/1342/1342-h/1342-h.htm',
];
```

接下来，我们创建`PassGen`实例，并运行`generate()`：

```php
$passGen = new PassGen($source, 4, CACHE_DIR);
echo $passGen->generate();
```

以下是`PassGen`生成的一些示例密码：

![它是如何工作的...](img/B05314_12_10.jpg)

## 另请参阅

关于攻击者如何破解密码的优秀文章可以在[`arstechnica.com/security/2013/05/how-crackers-make-minced-meat-out-of-your-passwords/`](http://arstechnica.com/security/2013/05/how-crackers-make-minced-meat-out-of-your-passwords/)上查看。要了解更多关于暴力破解攻击的信息，您可以参考[`www.owasp.org/index.php/Brute_force_attack`](https://www.owasp.org/index.php/Brute_force_attack)。有关`oclHashcat`的信息，请参阅此页面：[`hashcat.net/oclhashcat/`](http://hashcat.net/oclhashcat/)。

# 使用 CAPTCHA 保护表单

**CAPTCHA**实际上是**Completely Automated Public Turing Test to Tell Computers and Humans Apart**的缩写。这种技术类似于前面的配方“使用令牌保护表单”。不同之处在于，它不是将令牌存储在隐藏的表单输入字段中，而是将令牌呈现为对自动攻击系统难以解密的图形。此外，CAPTCHA 的目的与表单令牌略有不同：它旨在确认网页访问者是人类，而不是自动系统。

## 操作步骤...

1.  CAPTCHA 有几种方法：基于只有人类才会知道的知识提出问题，文本技巧和需要解释的图形图像。

1.  图像方法向网页访问者展示了一个带有严重扭曲的字母和/或数字的图像。然而，这种方法可能会很复杂，因为它依赖于 GD 扩展，而并非所有服务器都可用。GD 扩展可能很难编译，并且对主机服务器上必须存在的各种库有很重的依赖。

1.  文本方法是呈现一系列字母和/或数字，并给网页访问者一个简单的指令，比如“请将这个倒过来输入”。另一种变化是使用 ASCII“艺术”来形成人类网页访问者能够解释的字符。

1.  最后，您可能会有一个问题/答案方法，例如“头部是由什么身体部位连接到身体上”，并且有答案如“手臂”、“腿”和“脖子”。这种方法的缺点是自动攻击系统有三分之一的机会通过测试。

### 生成文本 CAPTCHA

1.  在这个例子中，我们将从文本方法开始，然后再使用图像方法。无论哪种情况，我们首先需要定义一个生成要呈现的短语（并由网页访问者解码）的类。为此，我们定义一个`Application\Captcha\Phrase`类。我们还定义了在短语生成过程中使用的属性和类常量：

```php
namespace Application\Captcha;
class Phrase
{
  const DEFAULT_LENGTH   = 5;
  const DEFAULT_NUMBERS  = '0123456789';
  const DEFAULT_UPPER    = 'ABCDEFGHJKLMNOPQRSTUVWXYZ';
  const DEFAULT_LOWER    = 'abcdefghijklmnopqrstuvwxyz';
  const DEFAULT_SPECIAL  = 
    '¬\`|!"£$%^&*()_-+={}[]:;@\'~#<,>.?/|\\';
  const DEFAULT_SUPPRESS = ['O','l'];

  protected $phrase;
  protected $includeNumbers;
  protected $includeUpper;
  protected $includeLower;
  protected $includeSpecial;
  protected $otherChars;
  protected $suppressChars;
  protected $string;
  protected $length;
```

1.  构造函数如您所期望的那样，接受各种属性的值，分配默认值，以便可以创建一个实例而无需指定任何参数。`$include*`标志用于表示将在生成短语的基本字符串中存在哪些字符集。例如，如果您只希望有数字，则`$includeUpper`和`$includeLower`都将设置为`FALSE`。`$otherChars`提供了额外的灵活性。最后，`$suppressChars`表示将从基本字符串中删除的字符数组。默认情况下，删除大写字母`O`和小写字母`l`：

```php
public function __construct(
  $length = NULL,
  $includeNumbers = TRUE,
  $includeUpper= TRUE,
  $includeLower= TRUE,
  $includeSpecial = FALSE,
  $otherChars = NULL,
  array $suppressChars = NULL)
  {
    $this->length = $length ?? self::DEFAULT_LENGTH;
    $this->includeNumbers = $includeNumbers;
    $this->includeUpper = $includeUpper;
    $this->includeLower = $includeLower;
    $this->includeSpecial = $includeSpecial;
    $this->otherChars = $otherChars;
    $this->suppressChars = $suppressChars 
      ?? self::DEFAULT_SUPPRESS;
    $this->phrase = $this->generatePhrase();
  }
```

1.  然后，我们定义一系列的 getter 和 setter，每个属性都有一个。请注意，为了节省空间，我们只显示前两个。

```php
public function getString()
{
  return $this->string;
}

public function setString($string)
{
  $this->string = $string;
}

// other getters and setters not shown
```

1.  接下来，我们需要定义一个初始化基本字符串的方法。这由一系列简单的 if 语句组成，检查各种`$include*`标志并根据需要附加到基本字符串。最后，我们使用`str_replace()`来删除`$suppressChars`中表示的字符：

```php
public function initString()
{
  $string = '';
  if ($this->includeNumbers) {
      $string .= self::DEFAULT_NUMBERS;
  }
  if ($this->includeUpper) {
      $string .= self::DEFAULT_UPPER;
  }
  if ($this->includeLower) {
      $string .= self::DEFAULT_LOWER;
  }
  if ($this->includeSpecial) {
      $string .= self::DEFAULT_SPECIAL;
  }
  if ($this->otherChars) {
      $string .= $this->otherChars;
  }
  if ($this->suppressChars) {
      $string = str_replace(
        $this->suppressChars, '', $string);
  }
  return $string;
}
```

### 提示

**最佳实践**

摆脱可能与数字混淆的字母（即，字母`O`可能与数字`0`混淆，小写字母`l`可能与数字`1`混淆。

1.  现在我们准备定义生成随机短语的核心方法，这是验证码呈现给网站访问者的。我们设置一个简单的`for()`循环，并使用新的 PHP 7 `random_int()`函数在基本字符串中跳转：

```php
public function generatePhrase()
{
  $phrase = '';
  $this->string = $this->initString();
  $max = strlen($this->string) - 1;
  for ($x = 0; $x < $this->length; $x++) {
    $phrase .= substr(
      $this->string, random_int(0, $max), 1);
  }
  return $phrase;
}
}
```

1.  现在我们将注意力从短语转移到将生成文本验证码的类上。为此，我们首先定义一个接口，以便将来可以创建额外的验证码类，所有这些类都使用`Application\Captcha\Phrase`。请注意，`getImage()`将返回文本、文本艺术或实际图像，具体取决于我们决定使用哪个类：

```php
namespace Application\Captcha;
interface CaptchaInterface
{
  public function getLabel();
  public function getImage();
  public function getPhrase();
}
```

1.  对于文本验证码，我们定义了一个`Application\Captcha\Reverse`类。这个名字的原因是这个类不仅产生文本，而且是反向的文本。`__construct()`方法构建了一个`Phrase`的实例。请注意，`getImage()`以反向返回短语：

```php
namespace Application\Captcha;
class Reverse implements CaptchaInterface
{
  const DEFAULT_LABEL = 'Type this in reverse';
  const DEFAULT_LENGTH = 6;
  protected $phrase;
  public function __construct(
    $label  = self::DEFAULT_LABEL,
    $length = self:: DEFAULT_LENGTH,
    $includeNumbers = TRUE,
    $includeUpper   = TRUE,
    $includeLower   = TRUE,
    $includeSpecial = FALSE,
    $otherChars     = NULL,
    array $suppressChars = NULL)
  {
    $this->label  = $label;
    $this->phrase = new Phrase(
      $length, 
      $includeNumbers, 
      $includeUpper,
      $includeLower, 
      $includeSpecial, 
      $otherChars, 
      $suppressChars);
    }

  public function getLabel()
  {
    return $this->label;
  }

  public function getImage()
  {
    return strrev($this->phrase->getPhrase());
  }

  public function getPhrase()
  {
    return $this->phrase->getPhrase();
  }

}
```

### 生成图像验证码

1.  正如你可以想象的那样，图像方法要复杂得多。短语生成过程是相同的。主要区别在于，我们不仅需要在图形上印刷短语，还需要以不同的方式扭曲每个字母，并引入随机点的噪音。

1.  我们定义了一个实现`CaptchaInterface`的`Application\Captcha\Image`类。该类的常量和属性不仅包括短语生成所需的内容，还包括图像生成所需的内容：

```php
namespace Application\Captcha;
use DirectoryIterator;
class Image implements CaptchaInterface
{

  const DEFAULT_WIDTH = 200;
  const DEFAULT_HEIGHT = 50;
  const DEFAULT_LABEL = 'Enter this phrase';
  const DEFAULT_BG_COLOR = [255,255,255];
  const DEFAULT_URL = '/captcha';
  const IMAGE_PREFIX = 'CAPTCHA_';
  const IMAGE_SUFFIX = '.jpg';
  const IMAGE_EXP_TIME = 300;    // seconds
  const ERROR_REQUIRES_GD = 'Requires the GD extension + '
    .  ' the JPEG library';
  const ERROR_IMAGE = 'Unable to generate image';

  protected $phrase;
  protected $imageFn;
  protected $label;
  protected $imageWidth;
  protected $imageHeight;
  protected $imageRGB;
  protected $imageDir;
  protected $imageUrl;
```

1.  构造函数需要接受前面步骤中描述的短语生成所需的所有参数。此外，我们还需要接受图像生成所需的参数。两个必需参数是`$imageDir`和`$imageUrl`。第一个是图形将被写入的位置。第二个是基本 URL，之后我们将附加生成的文件名。如果我们想提供 TrueType 字体，可以提供`$imageFont`，这将产生更安全的验证码。否则，我们只能使用默认字体，引用一部著名电影中的一句台词，*不是一道美丽的风景*：

```php
public function __construct(
  $imageDir,
  $imageUrl,
  $imageFont = NULL,
  $label = NULL,
  $length = NULL,
  $includeNumbers = TRUE,
  $includeUpper= TRUE,
  $includeLower= TRUE,
  $includeSpecial = FALSE,
  $otherChars = NULL,
  array $suppressChars = NULL,
  $imageWidth = NULL,
  $imageHeight = NULL,
  array $imageRGB = NULL
)
{
```

1.  接下来，在构造函数中，我们检查`imagecreatetruecolor`函数是否存在。如果返回`FALSE`，我们知道 GD 扩展不可用。否则，我们将参数分配给属性，生成短语，删除旧图像，并写出验证码图形：

```php
if (!function_exists('imagecreatetruecolor')) {
    throw new \Exception(self::ERROR_REQUIRES_GD);
}
$this->imageDir   = $imageDir;
$this->imageUrl   = $imageUrl;
$this->imageFont  = $imageFont;
$this->label      = $label ?? self::DEFAULT_LABEL;
$this->imageRGB   = $imageRGB ?? self::DEFAULT_BG_COLOR;
$this->imageWidth = $imageWidth ?? self::DEFAULT_WIDTH;
$this->imageHeight= $imageHeight ?? self::DEFAULT_HEIGHT;
if (substr($imageUrl, -1, 1) == '/') {
    $imageUrl = substr($imageUrl, 0, -1);
}
$this->imageUrl = $imageUrl;
if (substr($imageDir, -1, 1) == DIRECTORY_SEPARATOR) {
    $imageDir = substr($imageDir, 0, -1);
}

$this->phrase = new Phrase(
  $length, 
  $includeNumbers, 
  $includeUpper,
  $includeLower, 
  $includeSpecial, 
  $otherChars, 
  $suppressChars);
$this->removeOldImages();
$this->generateJpg();
}
```

1.  删除旧图像的过程非常重要；否则我们最终会得到一个充满过期验证码图像的目录！我们使用`DirectoryIterator`类来扫描指定目录并检查访问时间。我们将旧图像文件定义为当前时间减去`IMAGE_EXP_TIME`指定值的文件：

```php
public function removeOldImages()
{
  $old = time() - self::IMAGE_EXP_TIME;
  foreach (new DirectoryIterator($this->imageDir) 
           as $fileInfo) {
    if($fileInfo->isDot()) continue;
    if ($fileInfo->getATime() < $old) {
      unlink($this->imageDir . DIRECTORY_SEPARATOR 
             . $fileInfo->getFilename());
    }
  }
}
```

1.  现在我们准备转向主要内容。首先，我们将`$imageRGB`数组分成`$red`、`$green`和`$blue`。我们使用核心的`imagecreatetruecolor()`函数生成指定宽度和高度的基本图形。我们使用 RGB 值对背景进行着色：

```php
public function generateJpg()
{
  try {
      list($red,$green,$blue) = $this->imageRGB;
      $im = imagecreatetruecolor(
        $this->imageWidth, $this->imageHeight);
      $black = imagecolorallocate($im, 0, 0, 0);
      $imageBgColor = imagecolorallocate(
        $im, $red, $green, $blue);
      imagefilledrectangle($im, 0, 0, $this->imageWidth, 
        $this->imageHeight, $imageBgColor);
```

1.  接下来，我们根据图像宽度和高度定义*x*和*y*边距。然后，我们初始化要用于将短语写入图形的变量。然后我们循环多次，次数与短语的长度相匹配：

```php
$xMargin = (int) ($this->imageWidth * .1 + .5);
$yMargin = (int) ($this->imageHeight * .3 + .5);
$phrase = $this->getPhrase();
$max = strlen($phrase);
$count = 0;
$x = $xMargin;
$size = 5;
for ($i = 0; $i < $max; $i++) {
```

1.  如果指定了`$imageFont`，我们可以使用不同的大小和角度写入每个字符。我们还需要根据大小调整*x*轴（即水平）的值：

```php
if ($this->imageFont) {
    $size = rand(12, 32);
    $angle = rand(0, 30);
    $y = rand($yMargin + $size, $this->imageHeight);
    imagettftext($im, $size, $angle, $x, $y, $black, 
      $this->imageFont, $phrase[$i]);
    $x += (int) ($size  + rand(0,5));
```

1.  否则，我们将被默认字体所困扰。我们使用最大尺寸的`5`，因为较小的尺寸是不可读的。我们通过交替使用`imagechar()`（正常写入图像）和`imagecharup()`（侧向写入）来提供低级别的扭曲：

```php
} else {
    $y = rand(0, ($this->imageHeight - $yMargin));
    if ($count++ & 1) {
        imagechar($im, 5, $x, $y, $phrase[$i], $black);
    } else {
        imagecharup($im, 5, $x, $y, $phrase[$i], $black);
    }
    $x += (int) ($size * 1.2);
  }
} // end for ($i = 0; $i < $max; $i++)
```

1.  接下来，我们需要添加随机点的噪音。这是必要的，以使图像对自动化系统更难以检测。建议您也添加代码来绘制一些线条：

```php
$numDots = rand(10, 999);
for ($i = 0; $i < $numDots; $i++) {
  imagesetpixel($im, rand(0, $this->imageWidth), 
    rand(0, $this->imageHeight), $black);
}
```

1.  然后，我们使用我们的老朋友`md5()`创建一个随机图像文件名，其中日期和从`0`到`9999`的随机数作为参数。请注意，我们可以安全地使用`md5()`，因为我们并不试图隐藏任何秘密信息；我们只是想快速生成一个唯一的文件名。我们也清除图像对象以节省内存：

```php
$this->imageFn = self::IMAGE_PREFIX 
. md5(date('YmdHis') . rand(0,9999)) 
. self::IMAGE_SUFFIX;
imagejpeg($im, $this->imageDir . DIRECTORY_SEPARATOR 
. $this->imageFn);
imagedestroy($im);
```

1.  整个结构都在一个`try/catch`块中。如果发生错误或异常，我们会记录消息并采取适当的措施：

```php
} catch (\Throwable $e) {
    error_log(__METHOD__ . ':' . $e->getMessage());
    throw new \Exception(self::ERROR_IMAGE);
}
}
```

1.  最后，我们定义接口所需的方法。请注意，`getImage()`返回一个 HTML `<img>`标签，然后可以立即显示：

```php
public function getLabel()
{
  return $this->label;
}

public function getImage()
{
  return sprintf('<img src="%s/%s" />', 
    $this->imageUrl, $this->imageFn);
}

public function getPhrase()
{
  return $this->phrase->getPhrase();
}

}
```

## 它是如何工作的...

确保定义本配方中讨论的类，总结如下表：

| 类 | 子节 | 出现的步骤 |
| --- | --- | --- |
| `Application\Captcha\Phrase` | 生成文本 CAPTCHA | 1 - 5 |
| `Application\Captcha\CaptchaInterface` |   | 6 |
| `Application\Captcha\Reverse` |   | 7 |
| `Application\Captcha\Image` | 生成图像 CAPTCHA | 2 - 13 |

接下来，定义一个名为`chap_12_captcha_text.php`的调用程序，实现文本 CAPTCHA。您首先需要设置自动加载并使用适当的类：

```php
<?php
require __DIR__ . '/../Application/Autoload/Loader.php';
Application\Autoload\Loader::init(__DIR__ . '/..');
use Application\Captcha\Reverse;
```

之后，请确保启动会话。您还应该采取适当的措施来保护会话。为了节省空间，我们只显示了一个简单的措施，`session_regenerate_id()`：

```php
session_start();
session_regenerate_id();
```

接下来，您可以定义一个函数，该函数创建 CAPTCHA；检索短语、标签和图像（在本例中为反向文本）；并将该值存储在会话中：

```php
function setCaptcha(&$phrase, &$label, &$image)
{
  $captcha = new Reverse();
  $phrase  = $captcha->getPhrase();
  $label   = $captcha->getLabel();
  $image   = $captcha->getImage();
  $_SESSION['phrase'] = $phrase;
}
```

现在是初始化变量并确定`loggedIn`状态的好时机：

```php
$image      = '';
$label      = '';
$phrase     = $_SESSION['phrase'] ?? '';
$message    = '';
$info       = 'You Can Now See Super Secret Information!!!';
$loggedIn   = $_SESSION['isLoggedIn'] ?? FALSE;
$loggedUser = $_SESSION['user'] ?? 'guest';
```

然后您可以检查登录按钮是否已被按下。如果是，则检查 CAPTCHA 短语是否已输入。如果没有，初始化一条消息，告知用户他们需要输入 CAPTCHA 短语：

```php
if (!empty($_POST['login'])) {
  if (empty($_POST['captcha'])) {
    $message = 'Enter Captcha Phrase and Login Information';
```

如果 CAPTCHA 短语存在，请检查它是否与会话中存储的内容匹配。如果不匹配，请继续处理表单无效。否则，处理登录与以往一样。为了说明这一点，您可以使用用户名和密码的硬编码值模拟登录：

```php
} else {
    if ($_POST['captcha'] == $phrase) {
        $username = 'test';
        $password = 'password';
        if ($_POST['user'] == $username 
            && $_POST['pass'] == $password) {
            $loggedIn = TRUE;
            $_SESSION['user'] = strip_tags($username);
            $_SESSION['isLoggedIn'] = TRUE;
        } else {
            $message = 'Invalid Login';
        }
    } else {
        $message = 'Invalid Captcha';
    }
}
```

您可能还想添加注销选项的代码，如*Safeguarding the PHP session*配方中所述：

```php
} elseif (isset($_POST['logout'])) {
  session_unset();
  session_destroy();
  setcookie('PHPSESSID', 0, time() - 3600);
  header('Location: ' . $_SERVER['REQUEST_URI'] );
  exit;
}
```

然后您可以运行`setCaptcha()`：

```php
setCaptcha($phrase, $label, $image);
```

最后，不要忘记视图逻辑，在本例中，它呈现一个基本的登录表单。在表单标记内，您需要添加视图逻辑来显示 CAPTCHA 和标签：

```php
<tr>
  <th><?= $label; ?></th>
  <td><?= $image; ?><input type="text" name="captcha" /></td>
</tr>
```

以下是生成的输出：

![它是如何工作的...](img/B05314_12_11.jpg)

为了演示如何使用图像 CAPTCHA，将`chap_12_captcha_text.php`中的代码复制到`cha_12_captcha_image.php`中。我们定义代表我们将写入 CAPTCHA 图像的目录位置的常量。（确保创建此目录！）否则，自动加载和使用语句结构类似。请注意，我们还定义了一个 TrueType 字体。差异以**粗体**标出：

```php
<?php
define('IMAGE_DIR', __DIR__ . '/captcha');
define('IMAGE_URL', '/captcha');
define('IMAGE_FONT', __DIR__ . '/FreeSansBold.ttf');
require __DIR__ . '/../Application/Autoload/Loader.php';
Application\Autoload\Loader::init(__DIR__ . '/..');
use Application\Captcha\Image;

session_start();
session_regenerate_id();
```

### 提示

**重要！**

字体可能受版权、商标、专利或其他知识产权法的保护。如果您使用未经许可的字体，您和您的客户可能会在法庭上承担责任！请使用开源字体，或者您拥有有效许可的 Web 服务器上可用的字体。

当然，在`setCaptcha()`函数中，我们使用`Image`类而不是`Reverse`：

```php
function setCaptcha(&$phrase, &$label, &$image)
{
  $captcha = new Image(IMAGE_DIR, IMAGE_URL, IMAGE_FONT);
  $phrase  = $captcha->getPhrase();
  $label   = $captcha->getLabel();
  $image   = $captcha->getImage();
  $_SESSION['phrase'] = $phrase;
  return $captcha;
}
```

变量初始化与先前的脚本相同，登录处理与先前的脚本相同：

```php
$image      = '';
$label      = '';
$phrase     = $_SESSION['phrase'] ?? '';
$message    = '';
$info       = 'You Can Now See Super Secret Information!!!';
$loggedIn   = $_SESSION['isLoggedIn'] ?? FALSE;
$loggedUser = $_SESSION['user'] ?? 'guest';

if (!empty($_POST['login'])) {

  // etc.  -- identical to chap_12_captcha_text.php
```

即使视图逻辑保持不变，因为我们正在使用`getImage()`，在图像验证码的情况下，它返回直接可用的 HTML。这是使用 TrueType 字体的输出：

![工作原理...](img/B05314_12_12.jpg)

## 还有更多...

如果您不愿意使用上述代码生成自己的内部 CAPTCHA，那么有很多库可用。大多数流行的框架都具有此功能。例如，Zend Framework 有其 Zend\Captcha 组件类。还有 reCAPTCHA，通常作为一项服务调用，您的应用程序会调用外部网站生成 CAPTCHA 和令牌。开始寻找的好地方是[`www.captcha.net/`](http://www.captcha.net/)网站。

## 另请参阅

有关字体作为知识产权的保护的更多信息，请参阅[`en.wikipedia.org/wiki/Intellectual_property_protection_of_typefaces`](https://en.wikipedia.org/wiki/Intellectual_property_protection_of_typefaces)上的文章。

# 加密/解密无需 mcrypt

在一般 PHP 社区的成员中，一个鲜为人知的事实是，被认为是安全的大多数基于 PHP 的加密的核心`mcrypt`扩展实际上并不安全。从安全的角度来看，最大的问题之一是`mcrypt`扩展需要对加密学有高级知识才能成功操作，而这是很少有程序员具备的。这导致了严重的误用，最终会出现诸如 256 分之 1 的数据损坏的问题。这不是一个好的几率。此外，对于`libmcrypt`，`mcrypt`扩展所基于的核心库，开发支持在 2007 年就被*放弃*了，这意味着代码库已经过时，存在错误，并且没有机制来应用补丁。因此，非常重要的是要了解如何在*不*使用`mcrypt`的情况下执行强大的加密/解密！

## 如何做...

1.  解决之前提出的问题的方法是使用`openssl`。这个扩展是被很好地维护的，并且具有现代和非常强大的加密/解密能力。

### 提示

**重要**

为了使用任何`openssl*`函数，必须编译和启用`openssl` PHP 扩展！此外，您需要在 Web 服务器上安装最新的 OpenSSL 软件包。

1.  首先，您需要确定安装中有哪些密码方法。为此，您可以使用`openssl_get_cipher_methods()`命令。示例将包括基于**高级** **加密标准**（**AES**），**BlowFish**（**BF**），**CAMELLIA**，**CAST5**，**数据** **加密标准**（**DES**），**Rivest** **Cipher**（**RC**）（也可亲切地称为**Ron's** **Code**），和**SEED**的算法。您会注意到，此方法显示了大小写重复的密码方法。

1.  接下来，您需要弄清楚哪种方法最适合您的需求。这里有一个快速总结各种方法的表：

| 方法 | 发布 | 密钥大小（位） | 密钥块大小（字节） | 注释 |
| --- | --- | --- | --- | --- |
| `camellia` | 2000 | 128, 192, 256 | 16 | 由三菱和 NTT 开发 |
| `aes` | 1998 | 128, 192, 256 | 16 | 由 Joan Daemen 和 Vincent Rijmen 开发。最初提交为 Rijndael |
| `seed` | 1998 | 128 | 16 | 由韩国信息安全机构开发 |
| `cast5` | 1996 | 40 到 128 | 8 | 由 Carlisle Adams 和 Stafford Tavares 开发 |
| `bf` | 1993 | 1 到 448 | 8 | 由 Bruce Schneier 设计 |
| `rc2` | 1987 | 8 到 1,024 默认为 64 | 8 | 由 Ron Rivest 设计（RSA 的核心创始人之一） |
| `des` | 1977 | 56（+8 奇偶校验位） | 8 | 由 IBM 开发，基于 Horst Feistel 的工作 |

1.  另一个考虑因素是您首选的块密码**操作模式**是什么。常见选择总结在这个表中：

| 模式 | 代表 | 注释 |
| --- | --- | --- |
| ECB | 电子密码本 | 不需要**初始化向量**（IV）；支持加密和解密的并行化；简单快速；不隐藏数据模式；不推荐！！！ |
| CBC | 密码块链接 | 需要 IV；即使相同，后续块也与前一个块进行异或运算，从而获得更好的整体加密；如果 IV 可预测，第一个块可以被解码，剩余消息暴露；消息必须填充到密码块大小的倍数；仅支持解密的并行化 |
| CFB | 密码反馈 | 与 CBC 密切相关，只是加密是反向进行的 |
| OFB | 输出反馈 | 非常对称：加密和解密相同；不支持任何并行化 |
| CTR | 计数器 | 在操作上类似于 OFB；支持加密和解密的并行化 |
| CCM | 计数器与 CBC-MAC | CTR 的派生；仅设计用于块长度为 128 位；提供认证和保密性；**CBC-MAC**代表**密码块链接 - 消息认证码** |
| GCM | Galois/Counter Mode | 基于 CTR 模式；每个要加密的流应使用不同的 IV；吞吐量异常高（与其他模式相比）；支持加密和解密的并行化 |
| XTS | 基于 XEX 的改进密码本模式与密文窃取 | 相对较新（2010 年）和快速；使用两个密钥；增加了可以安全加密的数据量 |

1.  在选择密码方法和模式之前，您还需要确定加密内容是否需要在 PHP 应用程序之外解密。例如，如果您将数据库凭据加密存储到独立的文本文件中，您是否需要能够从命令行解密？如果是这样，请确保您选择的密码方法和操作模式受目标操作系统支持。

1.  提供的**IV**的字节数取决于所选择的密码方法。为了获得最佳结果，使用`random_bytes()`（PHP 7 中的新功能），它返回真正的**CSPRNG**字节序列。IV 的长度差别很大。首先尝试大小为 16。如果生成了*警告*，将显示应为该算法提供的正确字节数，因此请相应调整大小：

```php
$iv  = random_bytes(16);
```

1.  要执行加密，使用`openssl_encrypt()`。以下是应该传递的参数：

| Parameter | 注释 |
| --- | --- |
| Data | 需要加密的明文。 |
| Method | 使用`openssl_get_cipher_methods()`识别的方法之一。识别如下：*method* - *key_size* - *cipher_mode*。所以，例如，如果您想要 AES 方法，密钥大小为 256，以及 GCM 模式，您将输入`aes-256-gcm`。 |
| Password | 虽然文档中称为*password*，但这个参数可以被视为*key*。使用`random_bytes()`生成一个与所需密钥大小匹配的密钥。 |
| Options | 在您对`openssl`加密有更多经验之前，建议您坚持使用默认值`0`。 |
| IV | 使用`random_bytes()`生成一个与密码方法匹配字节数的 IV。 |

1.  举个例子，假设你想选择 AES 密码方法，密钥大小为 256，并且选择 XTS 模式。以下是用于加密的代码：

```php
$plainText = 'Super Secret Credentials';
$key = random_bytes(16);
$method = 'aes-256-xts';
$cipherText = openssl_encrypt($plainText, $method, $key, 0, $iv);
```

1.  要解密，使用相同的`$key`和`$iv`值，以及`openssl_decrypt()`函数：

```php
$plainText = openssl_decrypt($cipherText, $method, $key, 0, $iv);
```

## 工作原理...

为了查看可用的密码方法，创建一个名为`chap_12_openssl_encryption.php`的 PHP 脚本，并运行以下命令：

```php
<?php
echo implode(', ', openssl_get_cipher_methods());
```

输出应该看起来像这样：

![工作原理...](img/B05314_12_13.jpg)

接下来，您可以添加要加密的明文、方法、密钥和 IV 的值。例如，尝试 AES，密钥大小为 256，使用 XTS 操作模式：

```php
$plainText = 'Super Secret Credentials';
$method = 'aes-256-xts';
$key = random_bytes(16);
$iv  = random_bytes(16);
```

要进行加密，可以使用`openssl_encrypt()`，指定之前配置的参数：

```php
$cipherText = openssl_encrypt($plainText, $method, $key, 0, $iv);
```

您可能还希望对结果进行 base 64 编码，以使其更易于使用：

```php
$cipherText = base64_encode($cipherText);
```

要解密，请使用相同的 `$key` 和 `$iv` 值。不要忘记先解码 base 64 值：

```php
$plainText = openssl_decrypt(base64_decode($cipherText), 
$method, $key, 0, $iv);
```

这里是输出，显示了 base 64 编码的密文，然后是解密后的明文：

![工作原理...](img/B05314_12_14.jpg)

如果您为 IV 提供了不正确数量的字节，对于所选择的密码方法，将显示警告消息：

![工作原理...](img/B05314_12_15.jpg)

## 还有更多...

在 PHP 7 中，使用 `open_ssl_encrypt()` 和 `open_ssl_decrypt()` 以及支持的 **Authenticated Encrypt with Associated Data** (**AEAD**) 模式：GCM 和 CCM 时存在问题。因此，在 PHP 7.1 中，这些函数已添加了三个额外的参数，如下所示：

| 参数 | 描述 |
| --- | --- |
| `$tag` | 通过引用传递的认证标签；如果认证失败，变量值保持不变 |
| `$aad` | 附加的认证数据 |
| `$tag_length` | GCM 模式为 4 到 16；CCM 模式没有限制；仅适用于 `open_ssl_encrypt()` |

有关更多信息，您可以参考 [`wiki.php.net/rfc/openssl_aead`](https://wiki.php.net/rfc/openssl_aead)。

## 另请参阅

有关在 PHP 7.1 中为什么要弃用 `mcrypt` 扩展的出色讨论，请参阅 [`wiki.php.net/rfc/mcrypt-viking-funeral`](https://wiki.php.net/rfc/mcrypt-viking-funeral) 上的文章。有关分组密码的良好描述，这构成了各种密码方法的基础，请参阅 [`en.wikipedia.org/wiki/Block_cipher`](https://en.wikipedia.org/wiki/Block_cipher) 上的文章。有关 AES 的出色描述，请参阅 [`en.wikipedia.org/wiki/Advanced_Encryption_Standard`](https://en.wikipedia.org/wiki/Advanced_Encryption_Standard)。可以在 [`en.wikipedia.org/wiki/Block_cipher_mode_of_operation`](https://en.wikipedia.org/wiki/Block_cipher_mode_of_operation) 上看到描述加密操作模式的出色文章。

### 注意

对于一些较新的模式，如果要加密的数据小于块大小，`openssl_decrypt()` 将不返回任何值。如果 *填充* 要至少达到块大小的数据，则问题就消失了。大多数模式实现了内部填充，因此这不是问题。对于一些较新的模式（即 `xts`），您可能会遇到这个问题。在将代码投入生产之前，请务必对少于八个字符的短数据字符串进行测试。
