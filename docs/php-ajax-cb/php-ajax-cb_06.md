# 第六章。优化

在本章中，我们将涵盖以下主题：

+   对象的缓存

+   使用 YSlow 获取优化提示

+   通过自动压缩和浏览器缓存加快 JavaScript 交付

+   提前触发 JavaScript/在 DOM 加载时

+   图像的延迟加载

+   通过 Apache 模块/Google mod_pagespeed 自动优化 Ajax 应用程序

作为 JavaScript 开发人员，我们经常面临性能问题——页面加载缓慢、页面响应不佳、浏览器窗口冻结等。大多数情况下，这些问题都是由于脚本中的瓶颈或我们采取的方法/算法引起的。在本章中，让我们讨论解决这些问题的可能方法。

# 对象缓存

由于 JavaScript 代码必须在客户端机器上运行，所以代码级的优化非常重要。其中最重要的是缓存或缓冲计算和对象。这种基本的优化经常被忽视。

## 准备工作

我们需要识别重复的函数调用以缓存结果；这将加快代码的性能。

## 如何做…

```php
var a = Math.sqrt(10);
var b = Math.sqrt(10);

```

在这种情况下，我们反复计算相同的`sqrt(10)`并将其存储在不同的变量中。这是多余的；正如你所知，它可以写成如下形式：

```php
var sqrt_10 = Math.sqrt(10);
var a = sqrt_10, b = sqrt_10;

```

同样，在基于选择器的框架中，建议缓存或缓冲选择器对象。例如，考虑以下 HTML 标记：

```php
<a href="#" id="trigger">Trigger</a>
<div id="container">
Container
</div>

```

以下是隐藏容器的 jQuery 代码；当点击触发链接时，它显示容器如下：

```php
$('#trigger').click(function(){
});

```

## 它是如何工作的…

如你在前面的代码片段中所看到的，我们两次使用了`$('#container')`；这意味着我们为相同的目的运行了两次`$()`。如果你看一下 jQuery 代码，`$()`调用有其他函数，最终是多余的。因此，建议将`$('#container')`缓存到另一个变量中，并按如下方式使用：

```php
var $container = $('#container'); // cache the object
$container.hide();
$('#trigger').click(function(){
$container.show();
});

```

在某些情况下，对象的缓存（如前面的代码片段所示）可以将页面的响应速度提高一倍。当将缓存应用于缓慢/复杂的选择器（如`$('div p a')`）时，速度的提高很容易感受到。

# 使用 YSlow 获取优化提示

当我们遇到性能问题时，我们需要知道该怎么做。来自 Yahoo!的**YSlow**是一个速度诊断工具，它可以根据各种因素快速列出建议。

## 准备工作

我们需要使用安装了 Firebug 插件的 Firefox 浏览器。YSlow 是 Firebug 的一个附加组件，也需要安装才能获取优化提示。安装后，它会在 Firebug 内添加另一个选项卡，如下图所示：

![准备工作](img/3081_06_01.jpg)

## 如何做…

在任何页面上执行时，YSlow 都会给出一个特定于页面的优化建议报告。它还捆绑了一些优化工具，可以帮助我们快速解决性能问题。由于它是基于浏览器的工具，它无法对服务器端代码提出建议——它只能建议服务器设置，如`gzip`和`expire`头。

在安装 YSlow 时，最好将其自动运行模式关闭。否则，它将为每个页面执行，这将减慢其他页面的浏览体验。

在[`developer.yahoo.com/yslow/:`](http://    http://developer.yahoo.com/yslow/:)上执行时的报告示例截图如下：

![如何做…](img/3081_06_03.jpg)

报告基于以下 22 条规则：

1.  最小化 HTTP 请求：

当页面有大量样式表和 JavaScript 引用时，加载时间会受到影响。每个文件都需要单独下载。解决方法是将所有 JavaScript 代码合并到一个文件中，将所有样式表合并到一个文件中。至于众多的 CSS 背景图片，我们可以使用一种称为 CSS Sprites 的技术。这样，我们可以最小化 HTTP 请求。YSlow 帮助我们识别了许多这样的 HTTP 请求，并给出了建议。

### 注意

**CSS Sprites**—是一种技术，通过它我们可以从多个 CSS 背景图形成一个称为*sprite*的单个 CSS 背景图像，通过调整 CSS 样式属性来使用相同的*sprite*图像。我们通过`background-position`属性在`sprite`中引用每个图像。

1.  使用内容交付网络：

**内容交付网络（CDN）**是一个第三方托管解决方案，用于提供静态内容和图像，其交付速度将比普通服务器设置更高，因为它是在云设置上运行的。YSlow 识别 CDN 的使用，如果我们没有使用任何 CDN，它建议我们为更好的性能使用 CDN。

1.  添加`Expires`或`Cache-Control`头：

如果在浏览器中缓存静态内容，将会提高加载速度，因为这些内容不需要再次下载。我们必须确保不对动态内容应用浏览器缓存。当我们有单个 JavaScript 和单个 CSS 文件时，为了避免 HTTP 请求，我们至少可以在浏览器级别对它们进行缓存。为此，我们可以使用`Expires`或`Cache-Control` HTTP 头。YSlow 识别 HTTP 头并建议我们在没有使用时使用浏览器缓存头。

1.  `Gzip`组件：

强烈建议通过 PHP 或 Apache 进行页面内容的`gzip`—Apache 的`mod_deflate`更可取，因为它易于配置，并且可以在交付过程中实时压缩。YSlow 可以识别`gzip`的使用，并建议我们在没有使用时使用`gzip`。

1.  将样式表放在顶部：

根据浏览器行为，如果样式表在顶部引用，用户将有更好的加载体验。如果它们在底部引用，用户将根据其下载速度看到样式的应用速度较慢。YSlow 根据样式表引用对页面进行评分。

1.  将脚本放在底部：

当脚本放在顶部时，它们会阻塞页面的加载。这在我们要链接外部脚本时非常重要，比如谷歌分析、Facebook 库等等。这些脚本可以在`</body>`标签结束之前被引用。另一个解决方案是在链接外部脚本时使用`async`属性。YSlow 根据我们链接脚本的位置对页面进行评分，并帮助我们提高速度。

1.  避免 CSS 表达式：

CSS 表达式是 Internet Explorer 在 8 版本之前将 JavaScript 与 CSS 混合的提供。根据研究，表达式经常被触发，并导致页面响应速度变慢。YSlow 检测到使用并对页面进行评分。

1.  使 JavaScript 和 CSS 外部化：

最好将 JavaScript 和 CSS 文件保持外部化，而不是内联和内部化。这样，外部文件可以在浏览器级别进行缓存，以便页面加载更快。将脚本分离到外部文件是**不显眼的 JavaScript**和基于选择器的 JavaScript 框架（如 jQuery、YUI 等）的主要关注点。

1.  减少 DNS 查找：

如果网站从不同的域引用图像、样式表和 JavaScript，DNS 查找次数会增加。尽管 DNS 查找被缓存，但当引用了许多域时，网站的加载时间会增加。YSlow 识别 URL 中不同主机名的引用。

1.  压缩 JavaScript 和 CSS：

如下一条建议所述，由于文件大小减小，经过压缩的 JavaScript 和 CSS 文件可以更快地下载。YSlow 还有一个选项/工具来压缩 JavaScript 和 CSS 文件。

1.  避免重定向：

不必要的页面重定向会影响加载速度。

1.  删除重复的脚本

不必要的重复脚本是多余的。

1.  配置 ETags：

**ETag**类似于其他浏览器缓存选项。虽然它可以避免不必要的往返，但在服务器之间不一致。因此，最好彻底禁用它，以减少 HTTP 请求头大小。

1.  使 Ajax 可缓存：

甚至 Ajax 请求也可以在浏览器端进行缓存。这样做可以增加应用的响应速度。

1.  对 Ajax 请求使用`GET`：

雅虎团队指出，对于 Ajax 请求，`POST`操作是一个两步过程，`GET`请求只需要一个 TCP 数据包。根据 HTTP 规范，`GET`操作用于检索内容，而 POST 用于发布或更新。

1.  减少 DOM 元素的数量：

如果我们在页面呈现时尝试应用 JavaScript 效果或事件，而页面包含大量 HTML 标记，那么由于 JavaScript 代码必须遍历每个 DOM 元素，页面速度会变慢。YSlow 建议我们将 DOM 元素数量保持在最小限度。

1.  没有 404：

损坏的链接会导致不必要的请求。它们通常是由于引用链接中的拼写错误或错误引起的。YSlow 会识别损坏的链接并对页面进行评分。

1.  减少 cookie 大小：

Cookie 总是在 HTTP 请求中发送。因此，如果有很多信息存储在 cookie 中，它将影响 HTTP 请求-响应时间。

1.  为组件使用无 cookie 的域：

没有必要引用 cookie 来传递静态内容。因此，更明智的做法是通过某个子域引用所有静态内容，并避免为该域设置 cookie。

1.  避免使用过滤器：

在 Internet Explorer 中使用过滤器来处理 PNG 文件是很常见的，但是使用过滤器通常会减慢页面速度。解决方案是使用 IE 已经支持的 PNG8 文件。

1.  不要在 HTML 中缩放图像：

使用大图像并将其缩小，使用“高度”和“宽度”属性并不是明智的选择。这会迫使浏览器加载大图像，即使它们必须以较小的尺寸显示。解决方案是在服务器级别调整图像大小。

1.  使`favicon.ico`图标小且可缓存：

与图像类似，favicon 图标的大小必须小且可缓存。

## 它是如何工作的...

YSlow 内置支持 JavaScript 代码最小化和通过 Yahoo！的 Smush 进行图像压缩。这是一个网络服务。它还有一个代码美化工具，可以帮助我们以格式化的方式查看 JavaScript 源代码，如下面的屏幕截图所示：

![它是如何工作的...](img/3081_06_02.jpg)

报告及其有用的提示帮助我们寻找性能基础设施，如 CDN、无 cookie 的静态内容传递等。注意：这需要开发人员额外的努力来修复问题。

## 还有更多...

谷歌的 Page Speed 扩展可以在[`code.google.com/speed/page-speed/docs/extension.html`](http://code.google.com/speed/page-speed/docs/extension.html)下载，它提供类似的速度诊断和自动建议。在下面的屏幕截图中，我们可以看到它是如何在[`www.packtpub.com/`](http://www.packtpub.com/)网站上执行的，它提供了速度得分和建议：

![还有更多...](img/3081_06_04.jpg)

谷歌在速度诊断方面的倡议并不令人意外，因为页面速度可能会影响其搜索引擎爬虫；请记住，网站速度是谷歌 PageRankTM 中的决定性因素之一。YSlow 对页面进行从 A 到 F 的评分，而 Page Speed 提供一个 0 到 100 的分数。这两个插件使用类似的规则集来提供优化建议。

# 通过自动压缩和浏览器缓存加快 JavaScript 交付速度

JavaScript 最初是一种解释语言，但 V8 和 JIT 编译器现在正在取代解释器。V8 JavaScript 引擎最初是在 Google Chrome 和 Chromium 中引入的，它将 JavaScript 编译为本机机器代码。随着 Web 的不断发展，可能会有更强大的 JavaScript 编译器出现。

无论浏览器是否具有编译器或解释器，JavaScript 代码都必须在客户端机器上下载后才能执行。这需要更快的下载，这反过来意味着更少的代码大小。实现更少的代码空间和更快加载的最快和最常见的方法是：

+   去除空格、换行和注释——这可以通过 JSMin、Packer、Google Closure 编译器等最小化工具实现。

+   通过`gzip`进行代码压缩——所有现代浏览器都支持`gzip`内容编码，这允许内容以压缩格式从服务器传输到客户端；这反过来减少了需要下载的字节数，并提高了加载时间。

+   浏览器缓存以避免每次请求都下载脚本——我们可以强制将静态脚本在浏览器中缓存一段时间。这将避免不必要的往返。

在这个教程中，我们将快速比较 JavaScript 最小化工具，然后看看如何应用它们。

## 准备就绪

为了比较，我们需要以下最小化工具：

+   **JSMin**由 Douglas Crockford：[`www.crockford.com/javascript/jsmin.html`](http://www.crockford.com/javascript/jsmin.html)

+   **JSMin+**由 Tweakers.net（基于 Narcissus JavaScript 引擎）：[`crisp.tweakblogs.net/blog/cat/716`](http://crisp.tweakblogs.net/blog/cat/716)

+   **Packer**由 Dean Edwards：[`dean.edwards.name/packer/`](http://dean.edwards.name/packer/)

+   **YUI Compressor:**[`developer.yahoo.com/yui/compressor/`](http://developer.yahoo.com/yui/compressor/)

+   **Google Closure Compiler:**[`closure-compiler.appspot.com/`](http://closure-compiler.appspot.com/)

+   **UglifyJS:** [`github.com/mishoo/UglifyJS`](https://github.com/mishoo/UglifyJS)（PHP 版本：[`github.com/kbjr/UglifyJS.php`](http://https://github.com/kbjr/UglifyJS.php)）

对于 JavaScript 和 CSS 的自动最小化，我们将使用 Minify PHP 应用程序来自[`github.com/mrclay/minify`](http://https://github.com/mrclay/minify)。

为了比较最小化工具，让我们看看以下代码片段，重量为`931`字节。请注意，此代码包含注释、空格、换行和较长的变量和函数名称：

```php
/**
* Calculates the discount percentage for given price and discounted price
* @param (Number) actual_price Actual price of a product
* @param (Number) discounted_price Discounted price of a product
* @return (Number) Discount percentage
*/
function getDiscountPercentage(actual_price, discounted_price) {
var discount_percentage = 100 * (actual_price - discounted_price)/ actual_price;
return discount_percentage;

alert(discount_percentage); //unreachable code
}
// Let's take the book1's properties and find out the discount percentage...
var book1_actual_price = 50;
var book1_discounted_price = 48;
alert(getDiscountPercentage(book1_actual_price, book1_discounted_price));
// Let's take the book2's properties and find out the discount percentage...
var book2_actual_price = 45;
var book2_discounted_price = 40;
alert(getDiscountPercentage(book2_actual_price, book2_discounted_price));

```

1.  JSMin 由 Douglas Crockford 创建。

输出：

```php
    function getDiscountPercentage(actual_price,discounted_price){var discount_percentage=100*(actual_price-discounted_price)/actual_price;return discount_percentage;alert(discount_percentage);}var book1_actual_price=50;var book1_discounted_price=48;alert(getDiscountPercentage(book1_actual_price,book1_discounted_price));var book2_actual_price=45;var book2_discounted_price=40;alert(getDiscountPercentage(book2_actual_price,book2_discounted_price));

    ```

1.  JSMin+由 Tweakers.net（基于 Narcissus JavaScript 引擎）。

输出：

```php
    function getDiscountPercentage(actual_price,discounted_price){var discount_percentage=100*(actual_price-discounted_price)/actual_price;return discount_percentage;alert(discount_percentage)};var book1_actual_price=50,book1_discounted_price=48;alert(getDiscountPercentage(book1_actual_price,book1_discounted_price));var book2_actual_price=45,book2_discounted_price=40;alert(getDiscountPercentage(book2_actual_price,book2_discounted_price))

    ```

1.  Dean Edwards 的 Packer。

输出：

```php
    function getDiscountPercentage(a,b){var c=100*(a-b)/a;return c;alert(c)}var book1_actual_price=50;var book1_discounted_price=48;alert(getDiscountPercentage(book1_actual_price,book1_discounted_price));var book2_actual_price=45;var book2_discounted_price=40;alert(getDiscountPercentage(book2_actual_price,book2_discounted_price));

    ```

使用 Base62 编码选项输出（混淆代码）：

```php
    eval(function(p,a,c,k,e,r){e=function(c){return c.toString(a)};if(!''.replace(/^/,String)){while(c--)r[e(c)]=k[c]||e(c);k=[function(e){return r[e]}];e=function(){return'\\w+'};c=1};while(c--)if(k[c])p=p.replace(new RegExp('\\b'+e(c)+'\\b','g'),k[c]);return p}('7 1(a,b){0 c=8*(a-b)/a;9 c;2(c)}0 3=d;0 4=e;2(1(3,4));0 5=f;0 6=g;2(1(5,6));',17,17,'var|getDiscountPercentage|alert|book1_actual_price|book1_discounted_price|book2_actual_price|book2_discounted_price|function|100|return||||50|48|45|40'.split('|'),0,{}))

    ```

1.  YUI Compressor。

输出：

```php
    function getDiscountPercentage(b,c){var a=100*(b-c)/b;return a;alert(a)}var book1_actual_price=50;var book1_discounted_price=48;alert(getDiscountPercentage(book1_actual_price,book1_discounted_price));var book2_actual_price=45;var book2_discounted_price=40;alert(getDiscountPercentage(book2_actual_price,book2_discounted_price));

    ```

1.  Google Closure Compiler。

输出：

```php
    function getDiscountPercentage(a,b){return 100*(a-b)/a}var book1_actual_price=50,book1_discounted_price=48;alert(getDiscountPercentage(book1_actual_price,book1_discounted_price));var book2_actual_price=45,book2_discounted_price=40;alert(getDiscountPercentage(book2_actual_price,book2_discounted_price));

    ```

1.  UglifyJS。

输出：

```php
    function getDiscountPercentage(a,b){var c=100*(a-b)/a;return c}var book1_actual_price=50,book1_discounted_price=48;alert(getDiscountPercentage(book1_actual_price,book1_discounted_price));var book2_actual_price=45,book2_discounted_price=40;alert(getDiscountPercentage(book2_actual_price,book2_discounted_price))

    ```

931 字节 JavaScript 代码的表格化结果如下：

| 工具 | 删除无法到达的代码 | 压缩大小（字节） | 代码节省 |   |
| --- | --- | --- | --- | --- |
| JSMin 由 Douglas Crockford | 否 | 446 | 52.09% |   |
| JSMin+由 Tweakers.net | 否 | 437 | 53.06% |   |
| Packer 由 Dean Edwards Normal | 否 | 328 | 64.77% |   |
| 使用 Base62 编码 | 否 | 515 | 44.68% |   |
| YUI Compressor | 否 | 328 | 64.77% |   |
| Google Closure Compiler | 是 | 303 | 67.45% |   |
| UglifyJS | 是 | 310 | 66.70% |   |

所有这些工具都会去除空格、换行和不必要的注释，以减少 JavaScript 的大小。Dean Edwards 的 Packer 既有代码混淆又有最小化组件。它的 Base62 编码或代码混淆不建议使用，因为解包必须在浏览器中进行，因此会有很大的开销。

YUI Compressor 的压缩相对较好，因为它使用 Java Rhino 引擎来分析代码。Google Closure Compiler 看起来非常有前途，因为它有一个内置的编译器，可以检测到无法到达的代码，并进一步优化代码。UglifyJS 更快，因为它是用`Node.js`编写的。如前文所示，无论是 UglifyJS 还是 Google Closure Compiler 都可以删除无法到达的代码以改善代码最小化。

## 如何做…

来自[`github.com/mrclay/minify`](http://https://github.com/mrclay/minify)的**Minify**应用程序可用于自动化以下操作：

+   代码最小化

+   通过`gzip`压缩

+   通过`Last-Modified`或`ETag` HTTP 头进行浏览器缓存

我们必须将 Minify 应用程序的`min`文件夹放在文档根目录下。该文件夹包含以下内容：

+   `index.php：` 交付最小化代码的前端脚本

+   `config.php：`Minify 应用程序的设置文件

+   `groupConfig.php：`命名可以轻松压缩的文件组的设置文件

在`config.php`中，我们必须指定我们选择的压缩工具，如下所示：

```php
$min_serveOptions['minifiers']['application/x-javascript'] = array('Minify_JS_ClosureCompiler', 'JSMinPlus');

```

前面代码片段中显示的设置将首先尝试使用 Google 的 Closure 编译器，在任何错误时将使用 JSMinPlus 库。

有了这些配置，只需从以下更改 JavaScript，包括语法：

```php
<script type="text/javascript" src="/script1.js"></script>
<script type="text/javascript" src="/script2.js"></script>
<script type="text/javascript" src="/script3.js"></script>

```

到：

```php
<script type="text/javascript" src="/min/?f=script1.js,script2.js,script3.js"></script>

```

这将实现以下目标：

+   组合`script1.js，script2.js`和`script3.js`

+   压缩组合脚本

+   自动处理`gzip`内容编码

+   自动处理浏览器缓存

当有大量文件需要压缩时，我们可以利用`groupConfig.php`将文件分组到一个键中，如下所示：

```php
return array(
'js' => array('//script1.js', '//script2.js', '//script2.js')
);

```

我们可以通过键名简单地将它们引用到`g`查询字符串中，如下所示：

```php
<script type="text/javascript" src="/min/?g=js"></script>

```

## 它是如何工作的...

前端`index.php`脚本通过查询字符串`g`接收要压缩的文件。然后，逗号分隔的文件通过我们选择的压缩库进行合并和压缩：

```php
$min_serveOptions['minifiers']['application/x-javascript'] = array('Minify_JS_ClosureCompiler', 'JSMinPlus');

```

为了提高未来交付的性能，Minify 应用程序将以下版本存储到其缓存中：

+   合并压缩的 JavaScript 文件

+   合并压缩的 JavaScript 文件的 gzip 版本

存储在其缓存中的文件用于避免在压缩库上重复处理 JavaScript 文件。该应用程序还处理`Accept-Encoding` HTTP 标头，从而检测客户端浏览器对`gzip`、deflate 和传递相应内容的偏好。

该应用程序的另一个有用功能是设置`Last-Modified`或`ETag` HTTP 标头。这将使脚本在浏览器端进行缓存。只有在时间戳或内容发生变化时，Web 服务器才会向浏览器提供完整的脚本。因此，它节省了大量下载，特别是静态 JavaScript 文件内容。

请注意，jQuery 的 Ajax 方法默认情况下避免对脚本和`jsonp`数据类型的 Ajax 请求进行缓存。为此，它在查询字符串中附加了`_=[timestamp]`。当我们想要强制缓存时，我们必须显式启用它，这将禁用时间戳附加。操作如下：

```php
$.ajax({
url: "script.js",
dataType: "script",
cache: true
});

```

## 还有更多...

我们还有一些用于检查和加速交付选项的服务和应用程序。

### 比较 JavaScript 压缩工具

可以使用[`compressorrater.thruhere.net/`](http://compressorrater.thruhere.net/)上找到的基于 Web 的服务来比较许多压缩工具，从而我们可以为我们的代码选择合适的工具。

### 自动加速工具

对于自动加速，我们可以使用：

+   来自[`aciddrop.com/php-speedy/`](http://aciddrop.com/php-speedy/)的 PHP Speedy 库；它类似于 Minify 应用程序。

+   来自 Google 的`mod_pagespeed` Apache 模块。在本章的*通过 Apache 模块/Google mod_pagespeed 自动优化 Ajax 应用程序*中有解释。

# 尽早触发 JavaScript/在 DOM 加载时

在具有容器和动画的 Web 2.0 网站中，我们希望 JavaScript 代码尽快执行，以便用户在应用隐藏、显示或动画效果时不会看到闪烁效果。此外，当我们通过 JavaScript 或 JavaScript 框架处理任何事件时，我们希望诸如单击、更改等事件尽快应用于 DOM。

## 准备工作

早期，JavaScript 开发人员将 JavaScript 和 HTML 混合在一起。这种做法称为*内联脚本*。随着 Web 的发展，出现了更多的标准和实践。*不侵入式 JavaScript*实践通常意味着 JavaScript 代码与标记代码分开，并且 JavaScript 以*不侵入式*的方式处理。

以下是一些快速编写的代码，用于在单击名称字段时向用户发出消息`输入您的姓名！`：

```php
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Strict//EN"
"http://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd">
<html >
<head>
<title>Inline JavaScript</title>
</head>
<body>
<form method="post" action="action.php">
<fieldset>
<legend>Bio</legend>
<label for="name">Name</label><br />
<input type="text" id="name" onclick=
"alert('Enter your name!')" /><br />

<input type="submit" value="Submit" />
</fieldset>
</form>
</body>
</html>

```

如前面的代码所示，JavaScript 是在`input`标记内部编写和混合的。

内联 JavaScript 方法存在一些问题，例如：

+   JavaScript 代码无法被缓存。如果我们使用单个 JavaScript 文件（一个被压缩、`gzipped`并具有适当的 HTTP 标头以在浏览器中缓存的文件），我们可以感受到速度的提升。

+   代码不能轻易维护，特别是如果有许多程序员在同一个项目上工作。对于每个 JavaScript 功能，HTML 代码都必须更改。

+   网站可能存在可访问性问题，因为 JavaScript 代码可能会阻止非 JavaScript 设备上的功能。

+   HTML 脚本大小增加。如果由于动态内容等原因 HTML 不应该被缓存，这将影响页面的速度。

## 如何做到...

可以通过将 JavaScript 代码移动到`<head>`标记来实现 JavaScript 的分离。最好将 JavaScript 代码移动到一个单独的外部文件中，并在`<head>`标记中进行链接。

在下面的代码中，我们尝试将 JavaScript 代码与前面的清单分离如下：

```php
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Strict//EN"
"http://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd">
<html >
<head>
<script type="text/javascript" src="script.js">
</script>
<title>Unobtrusive JavaScript</title>
</head>
<body>
<form method="post" action="action.php">
<fieldset>
<legend>Bio</legend>
<label for="name">Name</label><br />
<input type="text" id="name" /><br />
<input type="submit" value="Submit" />
</fieldset>
</form>
</body>
</html>

```

在 JavaScript 代码中，我们添加了以下代码片段：

```php
window.onload = function(){
document.getElementById('name').onclick = function(){
alert('Enter your name!');
}
}

```

如前面的代码片段所示，我们通过`document.getElementById('name')`引用元素来附加了`click`事件。请注意，我们还将其包装在`window.onload`下；否则，`document.getElementById('name')`将不可用。这是因为`<head>`标记中的脚本在 DOM 准备就绪之前首先执行。`window.onload`确保在文档完全下载并可用时执行代码。

## 它是如何工作的...

`onload`事件的问题在于它只会在文档和相关文件（如 CSS 和图像）下载完成时触发。当页面包含任何大型图像文件或内容时，它会显著减慢 JavaScript 代码的触发速度。因此，当我们必须将任何事件附加到任何元素（如前面的代码所示），或者在页面加载期间隐藏任何`div`容器时，它不会按预期工作。用户将根据其下载速度看到一个无响应或闪烁的网站。

### DOMContentLoaded 和解决方法

幸运的是，基于 Gecko 的浏览器，如 Mozilla Firefox，有一个特殊的事件称为`DOMContentLoaded`。该事件将在 DOM 准备就绪时触发，而在图像、样式表和子框架完全下载之前。将 JavaScript 代码包装在`DOMContentLoaded`事件中将改善用户体验，因为 JavaScript 将在 DOM 准备就绪时立即触发。

使用`DOMContentLoaded`事件的修改后的代码如下：

```php
document.addEventListener('DOMContentLoaded',function(){
document.getElementById('name').addEventListener('click', function(){
alert('Enter your name!');
},false);
},false);

```

`DOMContentLoaded`事件首次在 Mozilla 的版本 1 中引入，最近其他浏览器（包括 Internet Explorer 版本 9）也开始支持它。由于它也是 HTML5 规范的一部分，更多的浏览器可能很快开始支持它。在那之前，有很多`DOMContentLoaded`的解决方法。例如，jQuery 的`ready`函数是为了支持许多浏览器而做出的努力。以下代码显示了如何使用浏览器兼容性（在 jQuery 中）重新编写前面的代码：

```php
jQuery(document).ready(function($){
$('#name').click(function(){
alert('Enter your name!');
});
});

```

## 还有更多...

即使我们对`DOMContentLoaded`事件使用了与浏览器兼容的 hack，也可能存在一些情况，这些 hack 可能无法按预期工作。在这种情况下，我们可以通过将初始化脚本放置在`</body>`标记之前来触发`load`函数。

# 图像的延迟加载

当加载大量图像时，它会减慢客户端浏览器；太多的图像请求甚至会减慢 Web 服务器。一个常见的方法是分割页面并平均分配图像和内容。另一种方法是利用 JavaScript 的功能，并在客户端级别避免不必要的图像请求。后一种技术称为**延迟加载**。在延迟加载中，图像请求被阻止，直到图像进入浏览器视口，也就是说，直到用户实际看到图像。

## 准备工作

我们需要一个较长的图像库页面来查看页面上大量图像对加载体验的影响。然后，我们必须在懒加载实现的不同方法之间做出决定。

## 如何做到...

我们可以通过以下方法解决懒加载问题：

+   纯 JavaScript

+   篡改的 HTML 标记

### 纯 JavaScript 方法

在这种方法中，图像不会在 HTML 中引用；它们只会在 JavaScript 中引用——要么是硬编码的，要么是从 JSON URL 加载的。图像元素将会动态形成，如下面的代码所示：

```php
// create img element
var img = document.createElement('img');
img.src = '/foo.jpg';
img.height = 50;
img.width = 100;
// append the img element to 'container' element
var container = document.getElementById('container');
container.appendChild(img);

```

这种方法的问题在于图像在 HTML 标记中没有定义，因此在不支持 JavaScript 的设备上无法工作。因此，这最终违反了可访问性标准。纯 JavaScript 应用程序在搜索引擎中难以索引，如果应用程序的营销基于**SEO**（即搜索引擎优化），这种方法将无法起作用。

### 篡改的 HTML 标记

另一种方法是将实际图像放在`rel`或`alt`属性中，并动态形成`src`属性。当图像必须在从`rel`或`alt`设置值后显示时，才会执行此操作。部分 HTML 标记和 JavaScript 如下：

```php
<img alt="/foo.jpg" />
<img alt="/bar.jpg" />
$('img').each(function(){
$(this).attr('src', $(this).attr('alt')); // assign alt data to src
});

```

请注意，篡改的 HTML 标记方法仍然不是一种整洁和可访问的方法。

## 它是如何工作的...

前面的方法不符合渐进增强原则，并且在 JavaScript 引擎不可用或关闭时停止显示图像。根据渐进增强方法，HTML 标记不应更改。当 DOM 准备就绪时，图像视口外的`src`属性将被动态篡改，以使图像不会被下载。篡改图像`src`属性以停止下载的部分代码如下：

```php
<img src="/foo.jpg" />
<img src="/bar.jpg" />
$('img').each(function(){
$(this).attr('origSrc',$(this).attr('src')); // assign src data to origSrc
$(this).removeAttr('src'); // then remove src
});

```

当需要加载图像时，使用以下代码片段：

```php
$('img').each(function(){
$(this).attr('src', $(this).attr('origSrc')); // assign origSrc data to src
});

```

尽管（到目前为止）这是最好的方法，尽管很容易通过任何 JavaScript 片段引入懒加载，但一些最新的浏览器在 DOM 准备好之前就开始下载图像。因此，这种方法并不适用于所有最新的浏览器。随着 Web 的发展，这种功能可能会在不久的将来直接添加到浏览器中。

## 还有更多...

我们有许多懒加载插件。我们还可以采用类似的方法——延迟脚本加载技术——来加载外部脚本。

### 懒加载插件

一些流行 JavaScript 框架可用的图像懒加载插件如下：

+   YUI 的图像加载器：[`developer.yahoo.com/yui/3/imageloader/`](http://developer.yahoo.com/yui/3/imageloader/)

+   jQuery 的 Lazy Load：[`www.appelsiini.net/projects/lazyload`](http://www.appelsiini.net/projects/lazyload)

+   MooTools 的 LazyLoad：[`www.davidwalsh.name/lazyload`](http://www.davidwalsh.name/lazyload)

+   Prototype 的 LazierLoad：[`www.bram.us/projects/js_bramus/lazierload/`](http://www.bram.us/projects/js_bramus/lazierload/)

### 懒惰/延迟脚本加载

虽然懒惰/延迟脚本加载与图像懒加载功能并不直接相关，但可以与上述技术结合，以获得更好的加载体验。当 JavaScript 文件通常链接在`<head>`标签中时，当脚本被执行时，Web 浏览器将暂停解析 HTML 代码。这种行为会使浏览器暂停一段时间，因此用户会感受到速度变慢。之前的建议是将脚本链接放在`</body>`标签之前。HTML5 引入了`script`标签的`async`属性；当使用时，浏览器将继续解析 HTML 代码，并在下载后执行脚本。脚本加载是异步的。

由于 Gecko 和基于 WebKit 的浏览器支持`async`属性，因此以下语法有效：

```php
<script type="text/javascript" src="foo.js" async></script>

```

对于其他浏览器，`async`仅在通过 DOM 注入时起作用。这是使用 DOM 注入的 Google Analytics 代码，以使所有浏览器中的异步加载可行：

```php
<script type="text/javascript">
var _gaq = _gaq || [];
_gaq.push(['_setAccount', 'UA-XXXXX-X']);
_gaq.push(['_trackPageview']);
(function(){
var ga = document.createElement('script');
ga.type = 'text/javascript';

ga.async = true;
ga.src = ('https:'==document.location.protocol?'https://ssl':'http://www')+'.google-analytics.com/ga.js';
var s = document.getElementsByTagName('script')[0];
s.parentNode.insertBefore(ga,s);
})();
</script>

```

当用于外部脚本时，例如 Google Analytics、Facebook 库等，这将提高加载速度。

# 通过 Apache 模块/Google mod_pagespeed 自动优化 Ajax 应用程序

自动优化 Ajax 应用程序-无需手动努力-是任何开发人员最想要的工具。为此目的发明了一些工具。在这个配方中，我们将看到一些这样的自动工具。

## 准备就绪

我们需要一个在 Apache Web 服务器上运行的 Web 应用程序。对于自动优化，我们需要以下 Apache 模块：

+   `mod_deflate`，可在[`httpd.apache.org/docs/2.0/mod/mod_deflate.html`](http://httpd.apache.org/docs/2.0/mod/mod_deflate.html)上找到

+   `mod_expires`，可在[`httpd.apache.org/docs/2.0/mod/mod_expires.html`](http://httpd.apache.org/docs/2.0/mod/mod_expires.html)上找到

+   `mod_pagespeed`，可在[`code.google.com/p/modpagespeed/`](http://code.google.com/p/modpagespeed/)上找到

## 如何操作...

我们必须安装这些模块，然后为它们设置配置，以自动处理请求。我们将看到每个模块的配置：

1.  `mod_deflate:`

要启用 JavaScript、CSS 和 HTML 代码的自动 gzip 处理，我们可以使用 AddOutputFilterByType 并指定它们的 MIME 类型：

```php
    <IfModule mod_deflate.c>
    AddOutputFilterByType
    DEFLATE application/javascript text/css text/html
    </IfModule>

    ```

1.  `mod_expires:`

要在静态内容上启用自动浏览器缓存，例如 JavaScript、CSS、图像文件、SWF 文件和 favicon，我们可以指定它们的 MIME 类型和过期时间，如下所示：

```php
    <IfModule mod_expires.c>
    FileETag None
    ExpiresActive On
    ExpiresByType application/javascript "access plus 1 month"
    ExpiresByType text/css "access plus 1 month"
    ExpiresByType image/gif "access plus 1 month"
    ExpiresByType image/jpeg "access plus 1 month"
    ExpiresByType image/png "access plus 1 month"
    ExpiresByType application/x-shockwave-flash
    "access plus 1 month"
    # special MIME type for icons
    AddType image/vnd.microsoft.icon .ico
    # now we have icon MIME type, we can use it
    ExpiresByType image/vnd.microsoft.icon "access plus 3 months"
    </IfModule>

    ```

在上述代码片段中，我们已经为图标文件注册了一个 MIME 类型，并且使用了 MIME 类型，我们已经设置了三个月的过期时间。这主要是针对 favicon 文件。对于静态内容，我们可以安全地设置 1 到 6 个月或更长的过期时间。上述代码将通过`Last-Modified`标头处理浏览器缓存，而不是通过 ETag，因为我们已经禁用了 ETag 支持。YSlow 建议我们完全禁用 ETag，以减少 HTTP 请求标头的大小。

### 注意

**ETag**据称现在被误用来唯一标识用户，因为许多用户出于隐私原因禁用了 cookie。因此，有努力在浏览器中禁用 ETag。

1.  `mod_pagespeed:`

`mod_pagespeed` Apache 模块是 Google 的页面速度倡议。Google 的倡议始于 Page Speed Firefox 扩展，类似于 YSlow。这是一个旨在找出瓶颈并提出建议的页面速度诊断工具。目前，Page Speed 扩展也适用于 Chrome。

现在，Page Speed 诊断工具可以作为基于 Web 的服务在[`pagespeed.googlelabs.com/`](http://pagespeed.googlelabs.com/)上使用，因此我们可以在不安装浏览器插件的情况下进行速度诊断。

Google 在这个领域的杰出努力的一个例子是发明了`mod_pagespeed` Apache 扩展，通过优化资源通过重写 HTML 内容自动执行速度建议。当正确配置时，它可以最小化、gzip、转换 CSS 精灵，并处理 Page Speed 浏览器扩展提供的许多其他建议。

当我们在 PageSpeed 中启用仪器时，它将注入跟踪器 JavaScript 代码，并将通过`mod_pagespeed`动态添加的信标图像进行跟踪。通过访问服务器中的`/mod_pagespeed_statistics`页面，我们可以找到有关使用情况的统计信息。

以下是要放置在`pagespeed.conf`文件中的`pagespeed_module`的快速配置代码：

```php
    LoadModule pagespeed_module /usr/lib/httpd/modules/mod_pagespeed.so
    # Only attempt to load mod_deflate if it hasn't been loaded already.
    <IfModule !mod_deflate.c>
    LoadModule deflate_module /usr/lib/httpd/modules/mod_deflate.so
    </IfModule>
    <IfModule pagespeed_module>
    ModPagespeed on
    AddOutputFilterByType MOD_PAGESPEED_OUTPUT_FILTER text/html
    # The ModPagespeedFileCachePath and
    # ModPagespeedGeneratedFilePrefix directories must exist and be
    # writable by the apache user (as specified by the User
    # directive).
    ModPagespeedFileCachePath "/var/mod_pagespeed/cache/"
    ModPagespeedGeneratedFilePrefix "/var/mod_pagespeed/files/"
    # Override the mod_pagespeed 'rewrite level'. The default level
    # "CoreFilters" uses a set of rewrite filters that are generally
    # safe for most web pages. Most sites should not need to change
    # this value and can instead fine-tune the configuration using the
    # ModPagespeedDisableFilters and ModPagespeedEnableFilters
    # directives, below. Valid values for ModPagespeedRewriteLevel are
    # PassThrough and CoreFilters.
    #
    ModPagespeedRewriteLevel CoreFilters
    # Explicitly disables specific filters. This is useful in
    # conjuction with ModPagespeedRewriteLevel. For instance, if one
    # of the filters in the CoreFilters needs to be disabled for a
    # site, that filter can be added to
    # ModPagespeedDisableFilters. This directive contains a
    # comma-separated list of filter names, and can be repeated.
    #
    # ModPagespeedDisableFilters rewrite_javascript
    # Explicitly enables specific filters. This is useful in
    # conjuction with ModPagespeedRewriteLevel. For instance, filters
    # not included in the CoreFilters may be enabled using this
    # directive. This directive contains a comma-separated list of
    # filter names, and can be repeated.
    #
    ModPagespeedEnableFilters combine_heads
    ModPagespeedEnableFilters outline_css,outline_javascript
    ModPagespeedEnableFilters move_css_to_head
    ModPagespeedEnableFilters convert_jpeg_to_webp
    ModPagespeedEnableFilters remove_comments
    ModPagespeedEnableFilters collapse_whitespace
    ModPagespeedEnableFilters elide_attributes
    ModPagespeedEnableFilters remove_quotes
    # Enables server-side instrumentation and statistics. If this rewriter is
    # enabled, then each rewritten HTML page will have instrumentation javacript
    # added that sends latency beacons to /mod_pagespeed_beacon. These
    # statistics can be accessed at /mod_pagespeed_statistics. You must also
    # enable the mod_pagespeed_statistics and mod_pagespeed_beacon handlers
    # below.
    #
    ModPagespeedEnableFilters add_instrumentation
    # ModPagespeedDomain
    # authorizes rewriting of JS, CSS, and Image files found in this
    # domain. By default only resources with the same origin as the
    # HTML file are rewritten. For example:
    #
    # ModPagespeedDomain cdn.myhost.com
    #
    # This will allow resources found on http://cdn.myhost.com to be
    # rewritten in addition to those in the same domain as the HTML.
    #
    # Wildcards (* and ?) are allowed in the domain specification. Be
    # careful when using them as if you rewrite domains that do not
    # send you traffic, then the site receiving the traffic will not
    # know how to serve the rewritten content.
    ModPagespeedDomain *
    ModPagespeedFileCacheSizeKb 102400
    ModPagespeedFileCacheCleanIntervalMs 3600000
    ModPagespeedLRUCacheKbPerProcess 1024
    ModPagespeedLRUCacheByteLimit 16384
    ModPagespeedCssInlineMaxBytes 2048
    ModPagespeedImgInlineMaxBytes 2048
    ModPagespeedJsInlineMaxBytes 2048
    ModPagespeedCssOutlineMinBytes 3000
    ModPagespeedJsOutlineMinBytes 3000
    ModPagespeedImgMaxRewritesAtOnce 8
    # This handles the client-side instrumentation callbacks which are injected
    # by the add_instrumentation filter.
    # You can use a different location by adding the ModPagespeedBeaconUrl
    # directive; see the documentation on add_instrumentation.
    #
    <Location /mod_pagespeed_beacon>
    SetHandler mod_pagespeed_beacon
    </Location>
    # This page lets you view statistics about the mod_pagespeed module.
    <Location /mod_pagespeed_statistics>
    Order allow,deny
    # You may insert other "Allow from" lines to add hosts you want to
    # allow to look at generated statistics. Another possibility is
    # to comment out the "Order" and "Allow" options from the config
    # file, to allow any client that can reach your server to examine
    # statistics. This might be appropriate in an experimental setup or
    # if the Apache server is protected by a reverse proxy that will
    # filter URLs in some fashion.
    Allow from localhost
    SetHandler mod_pagespeed_statistics
    </Location>
    </IfModule>

    ```

### 工作原理...

作为 Apache 的模块，这些模块在 Apache 级别处理优化。这意味着我们不必修改任何 PHP 或 JavaScript 代码。

1.  `mod_deflate:`

`mod_deflate`作用于指定的内容类型。每当应用程序命中指定的内容类型时，它会处理文件并根据浏览器请求进行 gzip 处理。

1.  `mod_expires:`

此模块还根据配置设置进行操作。它可以根据内容类型或文件扩展名进行处理。配置正确后，它将添加`Last-Modified`标头以避免缓存资源。根据每天的总点击量，它可以显着避免下载静态内容资源以加快站点加载速度。

1.  `mod_pagespeed：`

由于此模块通过重写来优化 HTML 代码，因此需要在服务器上缓存文件。路径必须在`pagespeed.conf`配置文件中配置。重写设置通过`ModPagespeedRewriteLevel`进行调整，默认设置为`CoreFilters`。使用 CoreFilters，以下过滤器将自动启用：

+   `add_head：`如果尚未存在，则向文档添加`<head>`元素。

+   `combine_css：`将多个 CSS 元素合并为一个。

+   `rewrite_css：`重写 CSS 文件以删除多余的空白和注释。

+   `rewrite_javascript：`重写 JavaScript 文件以删除多余的空白和注释。

+   `inline_css：`将小的 CSS 文件嵌入到 HTML 中。

+   `inline_javascript`将小的 JavaScript 文件嵌入到 HTML 中。

+   `rewrite_images：`优化图像，重新编码它们，删除多余的像素，并将小图像嵌入。

+   `insert_image：`由`rewrite_images`隐含。向缺少宽度和高度属性的`<img>`标签添加宽度和高度属性。

+   `inline_images：`由`rewrite_images`隐含。用内联数据替换小图像。

+   `recompress_images：`由`rewrite_images`隐含。重新压缩图像，删除多余的元数据，并将 GIF 图像转换为 PNG。

+   `resize_images：`由`rewrite_images`隐含。当相应的`<img>`标签指定的宽度和高度小于图像大小时，调整图像大小。

+   `extend_cache：`通过使用内容哈希签名 URL，延长所有资源的缓存寿命。

+   `trim_urls：`通过使它们相对于基本 URL 来缩短 URL。

还有一些其他未在`CoreFilters`中启用的过滤器：

+   `combine_heads：`将文档中找到的多个`<head>`元素合并为一个。

+   `strip_scripts：`从文档中删除所有脚本标记，以帮助运行实验。

+   `outline_css：`将大块的 CSS 外部化为可缓存的文件。

+   `outline_javascript：`将大块的 JavaScript 外部化为可缓存的文件。

+   `move_css_to_head：`将所有 CSS 元素移动到`<head>`标记中。

+   `make_google_analytics_async：`将 Google Analytics API 的同步使用转换为异步使用。

+   `combine_javascript：`将多个脚本元素合并为一个。

+   `convert_jpeg_to_webp：`向兼容的浏览器提供 WebP 而不是 JPEG。**WebP**，发音为'weppy'，是谷歌推出的一种图像格式，它比 JPEG 具有更好的压缩效果而不会影响质量。

+   `remove_comments：`删除 HTML 文件中的注释，但不包括内联 JS 或 CSS。

+   `collapse_whitespace：`除了`<pre>, <script>, <style>`和`<textarea>`内部之外，删除 HTML 文件中的多余空白。

+   `elide_attributes：`根据 HTML 规范删除不重要的属性。

+   `rewrite_domains：`根据`pagespeed.conf`中的`ModPagespeedMapRewriteDomain`和`ModPagespeedShardDomain`设置，重写`mod_pagespeed`未触及的资源的域。

+   `remove_quotes：`删除不是词法上必需的 HTML 属性周围的引号。

+   `add_instrumentation：`向页面添加 JavaScript 以测量延迟并发送回服务器。

可以通过`ModPagespeedEnableFilters`启用这些过滤器。同样，可以通过`ModPagespeedDisableFilters`禁用在 CoreFilters 中启用的任何过滤器。我们必须注意，由于此模块重写所有页面，服务器会有轻微的开销。我们可以选择性地禁用过滤器，并手动修改我们的 HTML 代码，以便进行重写。

如果我们所有的页面都是静态的，随着时间的推移，我们可以用缓存中可用的重写 HTML 代码替换 HTML 文件。然后我们可以完全禁用这个模块，以避免 CPU 开销。这个模块也是一个很好的学习工具，我们可以学习需要在 HTML、JavaScript 和 CSS 中进行哪些改变以提高性能。

### 还有更多...

为了检查我们是否正确配置了模块，或者检查性能，有一些在线服务可用。

#### 测试 HTTP 头

为了确保我们启用的`gzip`和浏览器缓存正常工作，我们可以使用：

+   使用 Firefox 扩展 Firebug 的 Net 标签来手动分析 HTTP 头

+   使用 YSlow 和 PageSpeed 扩展来检查等级/分数

+   一个基于网页的服务，可在[`www.webpagetest.org/`](http://www.webpagetest.org/)上使用，提供类似于 YSlow 和 Page Speed 的建议

+   一个基于网页的服务，可在[`redbot.org/`](http://redbot.org/)上使用，用于分析 HTTP 头，可能是最简单的选择。

#### 在不安装 mod_pagespeed 的情况下进行测试

使用[`www.webpagetest.org/compare`](http://www.webpagetest.org/compare)上的在线服务，我们可以快速测试通过安装`mod_pagespeed`可能获得的速度改进。视频功能可以实时反馈差异。

#### 页面速度服务

谷歌提供了网页速度服务。如果我们使用这项服务，就不需要在服务器上安装`mod_pagespeed`。服务器上唯一需要更改的是将`DNS CNAME`条目指向`ghs.google.com`。
