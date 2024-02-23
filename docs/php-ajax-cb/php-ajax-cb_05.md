# 第五章。调试和故障排除

在这一章中，我们将涵盖以下主题：

+   使用 Firebug 和 FirePHP 进行调试

+   使用 IE 开发者工具栏进行调试

+   避免框架$冲突

+   使用 JavaScript 的匿名函数

+   修复 JavaScript 中的内存泄漏

+   修复内存泄漏

+   顺序化 Ajax 请求

如果您不知道如何有效地使用 Ajax 进行调试，调试和故障排除可能会给您带来很大的麻烦。在本章中，我们将学习一些工具和技术来调试和故障排除 Ajax 应用程序。

首先，我们将研究为 Mozilla Firefox 浏览器构建的强大工具—Firebug 和 FirePHP。这两个工具可能是用于调试 Ajax 请求和响应最受欢迎的工具。在接下来的部分中，我们将研究另一个重要但不太复杂的工具—IE 开发者工具栏。

之后，我们将研究一种避免在单个网页中同时使用 jQuery 和 Mootools 时常见的美元（`$`）冲突的技术。

我们还将研究如何对 Ajax 应用程序的 Ajax 请求进行排序，这些应用程序需要定期更新数据。然后，我们将研究如何使用 Douglas Crockford 的 JSMin 或 Dean Edward 的 Packer 工具压缩的 JavaScript 的美化工具。最后，在本章中，我们将研究跨浏览器实现 Ajax 的技巧。

### 注意

当 Firebug 和 FirePHP 安装在 Mozilla Firefox 上时，它会比正常情况下占用更多的内存；因此，如果您的计算机内存较低，它可能会使您的系统不稳定。在这种情况下，建议您在 Firefox 的不同配置文件中安装 Firebug 和 FirePHP，您可以专门在 Web 开发期间使用它。

# 使用 Firebug 和 FirePHP 进行调试

当 Ajax 技术在复杂的 Web 应用程序中被广泛使用时，如果开发人员没有正确的工具，调试这些应用程序将成为一个头痛的问题。这就是 Firebug 和 FirePHP 派上用场的地方。**Firebug**是 Mozilla Firefox 用于调试基于 Ajax 的应用程序的一款优雅、简单、强大的附加组件。它允许您清晰地查看 Ajax 请求、响应以及通过 POST 或 GET 方法发送到服务器的数据的概况。此外，您甚至可以编辑 HTML 和 CSS 代码，并在浏览器中实时预览更改。除此之外，Firebug 还显示了网页发出的整个 HTTP 请求。它还允许您对 JavaScript 代码进行性能分析。**FirePHP**是 Firebug 的扩展，通过在 Firebug 控制台上记录信息或消息来扩展 Firebug 的功能。

### 注意

请注意，在 Firebug 中编辑的 CSS 或 HTML 代码是临时的，不会影响真实的代码。当 Mozilla Firefox 刷新时，更改会消失。

## 使用 Firebug 进行调试

**Firebug**可能是 Mozilla Firefox 浏览器中最受欢迎的附加组件之一。它允许调试、监视和编辑 CSS、HTML 和 JavaScript，以及 DOM。它有很多功能，但其中，我们将更多地讨论如何使用 JavaScript 控制台记录值或错误。

## 如何做...

所以，让我们首先安装 Firebug 来开始使用 Firebug 调试 Ajax/PHP 应用程序。

Firebug 可以从[`getfirebug.com/`](http://getfirebug.com/)下载。一旦您点击**安装 Firebug**按钮并按照网站上的步骤操作，您将看到以下弹出窗口开始安装。一旦您点击**立即安装**按钮，Firebug 就会安装在 Firefox 中。安装完成后，您可能需要重新启动 Mozilla Firefox 浏览器以完成 Firebug 的安装。

![如何做...](img/3081_05_01.jpg)

一旦安装了 Firebug，您可以通过按下*F12*或点击 Firefox 窗口右下角的 Firebug 图标来启用它。以下截图显示了启用 Firebug 时的外观：

![如何做...](img/3081_05_02.jpg)

正如您在前面的屏幕截图中所看到的，Firebug 中有六个不同的面板。让我们简要讨论每个面板：

+   **Console:** 这是 Firebug 中最有用的面板，用于调试富 Ajax 应用程序。您可以在此选项卡中记录不同的消息、信息或警告，来自 JavaScript 和 PHP（使用 FirePHP）。在这里，您有一个名为**Profile**的选项，它允许用户在指定的时间段内记录 JavaScript 活动。此外，您还可以在此面板上执行自己的代码。

+   **HTML:** 通常，添加或附加到网页的任何 HTML 元素都无法通过浏览器的**查看源代码**选项查看。但是，**HTML**窗格显示了可能已被执行的 JavaScript 代码添加的网页的实时 HTML 元素。该面板还可用于在浏览器中动态编辑 HTML/CSS 代码并实时查看输出。

+   **CSS:** 在此面板中，您可以查看网页使用的**CSS**脚本列表。此外，您还可以从此面板虚拟编辑 CSS 脚本，并直接在浏览器中查看更改属性的输出。

+   **Script:** 使用此面板，您可以找出当前网页正在使用的脚本。此面板还允许您通过设置断点并在调试时观察表达式或变量来调试 JavaScript 代码。在断点之后，您可以始终使用*F8*继续脚本执行，并且可以使用*F10*键逐步执行脚本。这是您在 Firebug 中找到的一个重要功能，通常存在于许多编程语言的**IDE（集成开发环境）**中。

+   **DOM:** 使用此面板，您可以探索网页的**文档对象模型（DOM）**。DOM 是一组对象和函数的层次结构，可以通过 JavaScript 调用或处理。此面板使您可以轻松地探索和修改 DOM 对象。

+   **Net:** 此面板被称为网页的**网络活动监视**面板。启用时，此面板会显示页面发出的每个 HTTP 请求以及加载对象（如 CSS 文件、图像或 JavaScript 文件）所花费的时间。除此之外，您还可以检查每个 HTTP 请求和响应的 HTTP 标头。除此之外，还可以在此面板和控制台面板中找到`XMLHttpRequest`的详细信息，以及其他信息，如 Ajax 请求、响应、HTTP 方法以及通过 GET 或 POST 方法提供的数据。

## 它是如何工作的...

Firebug API 提供了一个非常强大的对象**console**，可以直接将数据记录到**Console**面板。它可以记录任何类型的 JavaScript 数据和对象到控制台。您可以使用流行的`console.log()`函数轻松地将数据写入控制台，看起来像`console.log('testing')`；。您可以向此函数传递尽可能多的参数，如下所示：

```php
console.log(2,5,10,'testing');

```

当您使用`console.log(document.location)`在 Firebug 控制台中记录对象时，您可以在**Console**面板中看到对象列表，并链接到其属性和方法。您可以单击对象以查看属性和方法的详细信息。除了`console.log()`之外，还有其他函数可以在 Firebug 控制台中显示消息，具有不同的视觉效果。其中一些是`console.debug(), console.info(), console.warn()`和`console.error()`。

让我们看一个简单示例中信息记录的工作方式：

```php
$(document).ready(function()
{
console.log('log message');
console.debug('debug message');
console.info('info message');
console.warn('warning message');
console.error('Error message');
console.log(document.location);
});

```

前面的代码片段是使用 JavaScript 的 jQuery 框架的简单示例。

### 注意

您可以在本书的第二章*基本实用程序*中找到有关 jQuery 的更多信息。有关 jQuery 的更多信息可以在[`www.jquery.com`](http://www.jquery.com)找到。

所有控制台的不同功能都会在**Firebug**的**Console**面板中执行并显示，如下面的屏幕截图所示：

![它是如何工作的...](img/3081_05_03.jpg)

您可以看到上述代码的执行在控制台中产生了不同类型的消息，并且具有不同的颜色。您还可以注意到`console.log(document.location)`；产生了该对象的不同属性的超链接。

### 注意

如果您在 JavaScript 代码的开发环境中使用`console.log()`或任何其他控制台函数，请确保 Firebug 已激活；否则，当代码中遇到这些函数时，JavaScript 代码的执行可能会被中断，导致意外的结果。在 Internet Explorer 7 或更早版本中也可能发生同样的情况。请确保在将网站移至生产环境时删除所有控制台函数。

## 更多内容...

现在，让我们看看 Firebug 如何帮助您调试`XMLHttpRequest`，使用另一个例子，其中来自 PHP 脚本的 Ajax 响应是不可预测的。

以下 JavaScript 代码发出了 Ajax 请求：

```php
$(document).ready(function()
{
$.ajax({
type: "POST",
url: "test.php",
data: "start=1&end=200",
success: function(msg) {
console.log('number is '+msg); msg = parseInt(msg);
if(msg%2==0)
console.info('This is even number');
else
console.info('This is odd number');
}
});
});

```

上述代码是 jQuery JavaScript 代码。我们正在向`test.php`发出 Ajax 请求（POST 方法），并使用值为`1`和`200`的`start`和`end`作为 POST 数据。现在，让我们看看 PHP 中的服务器端脚本：

```php
<?php
echo rand($_POST['start'],$_POST['end']);
?>

```

服务器端代码只是在`start`和`end`参数之间选择一个随机数，这些参数在 PHP 中作为 POST 数据可用。

现在，让我们回头看看上述 JavaScript 代码中 Ajax 的`success`函数。它首先将来自服务器端脚本的数字记录到 Firebug 控制台。然后，使用`parseInt()`函数将这个数字严格转换为整数类型。来自 Ajax 的数字基本上是`String`数据类型，不能进行数学运算；因此，首先将其转换为整数。

之后，使用模数运算符检查这个数字，以查看它是奇数还是偶数，并相应地在**Firebug**控制台中显示信息。让我们看看 Firebug 控制台中的结果：

![更多内容...](img/3081_05_04.jpg)

如您在屏幕截图中所见，日志和消息会相应地显示。这些都是琐碎的，但您可以在控制台的第一行中看到一些新的东西，并且您可以轻松猜到这是 Ajax 请求，左侧有一个**+**符号。

让我们尝试通过点击**+**符号来探索 Ajax 请求和响应的细节。结果如下：

![更多内容...](img/3081_05_05.jpg)

如您在上述屏幕截图中所见，第一个选项卡是**Headers**；它显示了请求和响应的 HTTP 头部。

**Post**部分显示了通过 POST 方法向服务器发送的数据。

**Response**选项卡显示了 Ajax 请求的响应。

此外，最后一个选项卡显示了 HTML 格式的数据，如果响应是 HTML 格式的话。这个最后一个选项卡可以是 XML、JSON 或 HTML，具体取决于来自服务器端脚本的数据响应。

## 使用 FirePHP 进行调试

Firebug 允许您从 JavaScript 将调试消息记录到控制台。然而，在一个 Ajax 应用程序的非常复杂的服务器端脚本中，如果我们使用`console.log()`函数将所有消息记录为单个字符串，调试应用程序可能会变得非常困难。当我们需要调试涉及非常复杂 PHP 脚本的富 Ajax 应用程序时，FirePHP 就派上用场了。FirePHP 是 Firebug 的扩展，Firebug 本身是 Mozilla Firefox 浏览器的热门附加组件。FirePHP 允许您使用 FirePHP 库将调试消息和信息记录到 Firebug 控制台。

### 注意

如果您从 PHP 代码中传递 JSON 或 XML 数据作为 Ajax 响应，并使用 JavaScript 和 FirePHP 解析它，然后将一些消息记录到控制台，您可能会担心会破坏应用程序。不会；FirePHP 通过特殊的 HTTP 响应头将调试消息发送到浏览器，因此通过 FirePHP 记录的消息不会破坏应用程序。

## 准备就绪

要安装 FirePHP，您需要在 Mozilla Firefox 浏览器中安装 FireBug。您可以从其官方网站[`www.firephp.org/`](http://www.firephp.org/)安装 FirePHP。您需要点击**获取 FirePHP**按钮并按照安装 FirePHP 的步骤进行安装。安装了 FirePHP 后，您需要下载 PHP 库以与 FirePHP 一起使用。您可以从[`www.firephp.org/HQ/Install.htm`](http://www.firephp.org/HQ/Install.htm)下载 PHP 库。

现在，FirePHP 已安装并启用，您还已经下载了 FirePHP 的 PHP 库。让我们看看如何使用它：

![准备就绪](img/3081_05_06.jpg)

## 它是如何工作的...

要开始使用 FirePHP，首先需要在您的 PHP 代码中包含核心 FirePHP 类，如下所示：

```php
require_once('FirePHPCore/FirePHP.class.php');

```

在包含库后，您需要开始输出缓冲，因为已登录的消息将作为 HTTP 响应头发送：

```php
ob_start();

```

### 提示

如果在`php.ini`指令中打开了输出缓冲，您不需要显式调用`ob_start()`函数。有关**输出缓冲**配置的更多信息，请访问[`us.php.net/manual/en/outcontrol.configuration.php#ini.output-buffering`](http://us.php.net/manual/en/outcontrol.configuration.php#ini.output-buffering)。

现在，在此之后，让我们创建 FirePHP 对象的实例：

```php
$fp = FirePHP::getInstance(true);

```

之后，让我们使用 FirePHP 将一些消息记录到 FireBug 控制台中：

```php
$var = array('id'=>10, 'name'=>'Megan Fox','country'=>'US');
$fp->log($var, 'customer');

```

![它是如何工作的...](img/3081_05_07.jpg)

正如您在前面的屏幕截图中所看到的，数组以详细格式显示在 Firebug 控制台中。现在，让我们尝试以更加花哨的方式记录更多的变量。

### 注意

当鼠标光标移动到控制台中的已登录变量上时，FirePHP 的变量查看器（参见前面的屏幕截图）会显示出来。

此外，让我们尝试使用不同的函数将不同类型的调试消息记录到 FireBug 控制台中，如下所示：

```php
$fp->info($var,'Info Message');
$fp->warn($var,'Warn Message');
$fp->error($var,'Error Message');

```

上述函数与 Firebug 的控制台函数非常相似。这些函数的输出在 Firebug 控制台中如下所示：

![它是如何工作的...](img/3081_05_08.jpg)

正如您在前面的屏幕截图中所看到的，FirePHP 库的`info()、warn()`或`error()`函数可以以不同的样式记录消息，用于调试 PHP 代码。

### 注意

请确保在生产模式下使用网站时禁用 FirePHP 日志记录，否则任何安装了 FirePHP 和 Firebug 的人都可以轻松查看网站中的敏感信息。您可以通过在创建 FirePHP 对象实例后立即调用`$fp->setEnabled(false)`函数来禁用 FirePHP 日志记录。

## 还有更多...

FirePHP 还有**Procedural API**。要使用 FirePHP 的 Procedural API，您需要在代码中包含`fb.php`（FirePHP PHP 库提供），如下所示：

```php
require_once('FirePHPCore/fb.php');

```

然后，您可以通过使用`fb()`函数简单地将消息记录到 Firebug 控制台。例如，您可以使用以下代码将消息记录到控制台中：

```php
fb('logged message');
fb($var, 'customer');

```

### 提示

当您在代码中包含了`fb.php`后，您可以直接使用`fb`类调用`info()、warn()、error()`或`log()`函数。例如，您可以使用`FB::info($var,'Info Message')`来将`info`消息显示到控制台中。

# 使用 IE 开发者工具栏进行调试

与 Firebug 类似，Internet Explorer 也包含一个开发者工具栏，用于调试和编辑网页的 HTML、CSS 和 JavaScript 代码。**IE 开发者工具栏**内置于 Internet Explorer 8 中。在以前的版本中，它可以作为 Internet Explorer 的附加组件使用。如果您使用的是 Internet Explorer 7 或更低版本，则可以从 Microsoft 网站下载 IE 开发者工具栏，网址为[`www.microsoft.com/downloads/en/details.aspx?familyid=95E06CBE-4940-4218-B75D-B8856FCED535&displaylang=en`](http://www.microsoft.com/downloads/en/details.aspx?familyid=95E06CBE-4940-4218-B75D-B8856FCED535&displaylang=en)。但是，在本主题中，我们将讨论 Internet Explorer 8 中可用的 IE 开发者工具栏。

### 提示

除了 Firefox 之外，您始终可以在任何浏览器中使用 Firebug Lite。以下是有关如何在任何浏览器中使用 Firebug Lite 的说明：[`getfirebug.com/firebuglite`](http://getfirebug.com/firebuglite)。

## 准备就绪

Internet Explorer **开发者工具**主要由四个不同的面板组成，用于调试和编辑 HTML、CSS 和 JavaScript。

![准备就绪](img/3081_05_09.jpg)

它们如下：

+   **HTML**面板：此面板用于查看网站的 HTML 代码。使用此面板，您可以查看单个 HTML 元素的大纲，更改它们的属性和 CSS 属性，并在浏览器中实时预览输出。

+   **CSS**面板：这与 Firebug 的 CSS 面板非常相似。在这里，您可以查看和编辑与网页关联的不同样式表下的 CSS 属性。您还可以实时预览 CSS 属性的更改。

+   **Script**面板：此面板允许您调试网页的 JavaScript 代码。此外，您可以在 JavaScript 代码上设置断点，并逐步执行代码并观察变量。

+   **Profiler**面板：IE 开发者工具栏的**Profiler**面板允许您分析网页中使用的 JavaScript 函数的性能。它记录执行这些函数所需的时间以及它们被调用的次数；因此，如果其中一些函数编写得很差，调试这些函数就变得容易。

## 如何做...

开发者工具栏的**Script**面板允许通过设置断点、逐步执行代码和观察变量来调试脚本。此外，与 Firebug 一样，您还可以使用控制台函数向控制台记录消息。

例如，以下 JavaScript 控制台函数将分别向 IE 开发者工具的控制台发送日志、信息、警告和错误消息：

```php
console.log('log message');
console.info('info message');
console.warn('warning message');
console.error('Error message');

```

代码的输出在 IE 开发者工具的控制台中看起来像下面的截图：

![如何做...](img/3081_05_10.jpg)

您可以看到消息以与 Firebug 中相似的方式显示在控制台中。但是，遗憾的是，直到今天为止，Internet Explorer 还没有像 FirePHP 这样的附加组件。

# 避免框架$冲突

`$`在许多 JavaScript 框架中都是一个常用的函数名或变量名。当两个不同的 JavaScript 库一起使用时，使用$符号可能会发生冲突的可能性很高，因为它们可能会用于不同的目的。假设在一个页面中使用了两个框架，它们是 jQuery 和`prototype.js:`

```php
<script type="text/javascript" src="prototype.js"></script>
<script type="text/javascript" src="jquery.js"></script>

```

当两个框架一起使用并且两个框架都使用`$`符号时，结果可能是不可预测的，并且可能会中断，因为 jQuery 将`$`视为 jQuery 对象，而在`prototype.js`中，它是一个 DOM 访问函数。代码`$('mydiv').hide()`;在包含前面 JavaScript 框架用法的网页中可能无法正常工作。这是因为 jQuery 包含在最后一行，但代码`$('mydiv').hide()`;是来自`prototype.js`框架的代码，这会导致意外的结果。

## 准备就绪

如果你正在使用 jQuery 与其他框架，没有问题。jQuery 有一个神奇的`noConflict()`函数，允许你在其他框架中使用 jQuery。

## 如何做...

现在，让我们尝试使用 jQuery 的`noConflict()`函数来使用上述代码：

```php
<script type="text/javascript" src="prototype.js"></script>
<script type="text/javascript" src="jquery.js"></script>
<script type="text/javascript" ></script
var $jq = jQuery.noConflict();
$jq(document).ready(function(){
$jq("p.red").hide();
});
$('mydiv').hide();
</script>

```

## 它是如何工作的...

如你在上述代码中所见，我们创建了另一个别名`$jq`来代替`$`来引用 jQuery 对象。现在在剩下的代码中，可以使用`$jq`来引用 jQuery 对象。`$`可以被`prototype.js`库使用，用于其余的代码。

# 使用 JavaScript 的匿名函数

JavaScript 的**匿名函数**非常有用，可以避免 JavaScript 库中的冲突。

## 如何做...

让我们首先通过一个例子了解匿名函数：

```php
(function(msg)
{ alert(msg); })
('Hello world');

```

当我们在浏览器中执行上述代码时，它将显示警报`Hello world`。现在，这很有趣！一个函数被定义并执行！让我们简化相同的代码片段，看看它是如何工作的：

```php
Var t = function(msg){
alert(msg);
};
t('Hello world');

```

如果你看到等效的代码，那很简单。唯一的区别是这个简化的代码将变量名`t`与函数关联起来，而在另一个代码中，函数名是匿名的。匿名函数在声明后立即执行。

匿名函数在创建 JavaScript 框架的插件时非常有用，因为你不必担心与其他插件的函数同名而产生冲突。记住，给两个函数起相似的名字会导致 JavaScript 错误，并可能破坏应用程序。

现在，让我们看看如何使用 jQuery 的匿名函数来避免`$`的冲突：

```php
(function($) {
$(function() {
$('#mydiv').hide();
});
})(jQuery);

```

## 它是如何工作的...

在上述函数中，jQuery 对象作为`$`参数传递给函数。现在，匿名函数内部有一个局部作用域，因此可以在匿名函数内部自由使用`$`，以避免冲突。这种技术经常用于创建 jQuery 插件，并在插件代码中使用`$`符号。

## 还有更多...

现在，让我们在 Mootools 框架中类似地使用匿名函数来避免`$`的冲突。在 Mootools 框架中，`$`符号指的是`document.id`对象。

```php
(function($){
$('mydiv').setStyle('width', '300px');
})(document.id);

```

在上述函数中，`$`可以在本地使用，它指的是 Mootools 框架的`document.id`对象。

# 修复 JavaScript 中的内存泄漏

如果 JavaScript 代码没有考虑内存使用，可能会导致内存泄漏成为 JavaScript 中繁琐的问题。这样的代码可能会通过过载内存使你的浏览器变得不稳定。

## 什么是内存泄漏？

**内存泄漏**是指 JavaScript 分配的内存占用了物理内存，但无法释放内存。JavaScript 是一种进行垃圾回收的语言。当创建对象时，内存被分配给对象，一旦对象没有更多的引用，内存就会被释放。

## 可能导致内存泄漏的原因是什么？

内存泄漏可能有很多原因，但让我们探讨两个主要可能性：

+   你创建了大量未使用的元素或 JavaScript 对象而没有清理它们。

+   你的 JavaScript 代码中使用了循环引用。循环引用是指 DOM 对象和 JavaScript 对象相互循环引用。

# 修复内存泄漏

首先，让我们了解如何找出脚本生成了不需要的元素；我们可以使用 Firebug 控制台来做到这一点。你可以将以下代码放入 Firebug 的控制台中，如下面的屏幕截图所示：

```php
console.log( document.getElementsByTagName('*').length )

```

![修复内存泄漏](img/3081_05_11.jpg)

### 提示

上述代码将记录 DOM 中元素的所有计数。因此，如果你看到页面后续使用中计数呈指数增长，那么你的代码存在问题，你应该尝试删除或移除不再使用的元素。

## 如何做...

找到了不需要的脚本创建的元素后，我们如何调试？

假设有一个 JavaScript 函数一遍又一遍地被调用，创建了一个巨大的堆栈。让我们尝试使用`console.trace()`函数来调试这样的代码：

```php
<html >
<head>
<script type="text/javascript">
var i=0
function LeakMemory(){
i++;
console.trace();
if(i==50)
return;
LeakMemory();
}
</script>
</head>
<body>
<input type="button"
value="Memory Leaking Test" onclick="LeakMemory()" />
</body>
</html>

```

当你点击按钮时，它会调用函数`LeakMemory()`。该函数调用自身 50 次。我们还使用`console.trace()`函数来跟踪函数调用。你可以在 Firebug 控制台中看到以下输出：

![如何做...](img/3081_05_12.jpg)

## 它是如何工作的...

你可以清楚地看到`console.trace()`函数跟踪每个函数调用。它让你调试和跟踪 JavaScript 应用程序，该应用程序正在创建一个不需要的函数调用堆栈。

接下来，让我们用一个例子来讨论 JavaScript 中的循环引用内存泄漏模式：

```php
<html>
<body>
<script type="text/javascript">
document. Write("Circular references between JavaScript and DOM!");
var obj;
window.onload = function(){
obj=document.getElementById("DivElement");
document.getElementById("DivElement").expandoProperty=obj;
obj.bigString=new Array(1000).join(new Array(2000).join("XXXXX"));
};
</script>
<div id="DivElement">Div Element</div>
</body>
</html>

```

### 注意

上述例子摘自 IBM 网站上关于内存泄漏的一篇很棒的文章：[`www.ibm.com/developerworks/web/library/wa-memleak/`](http://www.ibm.com/developerworks/web/library/wa-memleak/)。

如你在上面的例子中所见，JavaScript 对象`obj`引用了一个 ID 为`DivElement`的 DOM 对象。`DivElement`引用了 JavaScript 对象`obj`，从而在两个元素之间创建了循环引用，并且由于这种循环引用，两个元素都没有被销毁。

当你运行上述代码时，让我们看看在 Windows 任务管理器中内存消耗如何上升：

![它是如何工作的...](img/3081_05_13.jpg)

正如你所看到的，当我同时运行包含上述代码的网页 4-5 次时，内存使用曲线在中间上升。

## 还有更多...

修复循环引用内存泄漏非常容易。只需在执行代码后将对象分配给`null`元素。让我们看看如何在上面的例子中修复它：

```php
var obj;
window.onload = function(){
obj=document.getElementById("DivElement");
document.getElementById("DivElement").expandoProperty=obj;
obj.bigString=new Array(1000).join(new Array(2000).join("XXXXX"));
};
obj = null.

```

现在，在一个网页中执行上述代码时，同时查看任务管理器。你不会看到内存使用量有显著的波动。

# 对 Ajax 请求进行排序

顾名思义，Ajax 是异步的，因此代码的顺序可能不会被遵循，因为大部分逻辑活动是在 HTTP 请求完成时完成的。

## 如何做...

让我们尝试用一个例子来理解 Ajax 请求：

```php
$(document).ready(function()
{
$.ajax({
type: "POST",
url: "test.php",
data:'json',
data: "bar=foo",
success: function(msg){
console.log('first log');
}
});
console.log('second log')
});

```

执行后，上述代码在 Firebug 控制台中显示如下：

![如何做...](img/3081_05_14.jpg)

## 它是如何工作的...

尽管`$.ajax`函数首先被调用，但由于代码的异步性质，**第二个日志**会先被打印出来（因为这行代码直接跟在`$.ajax`函数后面）。然后，当 HTTP 请求完成时，`success: function`被执行，之后**第一个日志**被打印到控制台。

对 Ajax 请求进行排序是实时应用中广泛使用的一种技术。在下面的例子中，让我们使用一个简单的使用 jQuery 的函数来对 Ajax 请求进行排序，以在浏览器中显示服务器时间。首先，让我们看一下将发送 Ajax 请求序列的 JavaScript 函数：

```php
function get_time()
{
//make another request
$.ajax({
cache: false,
type: "GET",
url: "ajax.php",
error: function () {
setTimeout(get_time, 5000);
},
success: function (response)
{
$('#timer_div').html(response);
//make another request instantly
get_time();
}
});
}

```

该函数很简单。在每次成功的 Ajax 请求后，它再次调用相同的函数。请注意，我们一次只向服务器发送一个请求。完成后，会发送另一个请求。

如果出现错误，或者说 Ajax 请求没有完成，我们将在等待 5 秒后重试发送另一个 Ajax 请求。通过这种方式，如果服务器面临无法完成请求的问题，我们可以最小化向服务器发送请求的次数。

现在，让我们看一下`ajax.php`中的 PHP 代码：

```php
<?php
sleep(1);
echo date('Y-m-d H:i:s');
?>

```

如你在上面的 PHP 代码中所见，服务器在打印当前时间之前等待一秒。这通常是实时 Web 应用中服务器端脚本的工作方式。例如，实时聊天应用程序会等待直到新的聊天消息进入数据库。一旦新消息在数据库中，应用程序会将最新的聊天消息发送到浏览器以显示它。

# 跨浏览器和 Ajax

+   我们都知道 Ajax 技术的核心是 JavaScript 中可用的`XMLHttpRequest`对象。但是这个对象在你的浏览器中不一定可用，特别是在 Internet Explorer 中，这取决于浏览器和平台。

+   它可以在 Mozilla Firefox、Google Chrome、Safari 甚至支持原生`XMLHttpRequest`对象的 IE7 或更高版本中本地实例化如下：

```php
var xmlHttpObj = new XMLHttpRequest();

```

+   现在，在 Internet Explorer 6 或 5 中，要使用`XMLHttpRequest`对象，它必须在 JavaScript 中作为 ActiveX 对象创建：

```php
var xmlHttpObj = new ActiveXObject("MSXML2.XMLHTTP.3.0");

```

+   但是即使是 ActiveX 对象类在不同的 Windows 平台上也可能不同，所以我们可能还需要使用以下代码：

```php
var xmlHttpObj = new ActiveXObject("Microsoft.XMLHTTP");

```

+   现在，让我们创建一个 Ajax 函数，它将在跨浏览器平台中返回`XMLHttpRequest`对象：

```php
function getAjaxObj()
{
var xmlHttpObj = null;
// use the ActiveX control for IE5 and IE6
try
{
xmlHttpObj = new ActiveXObject("MSXML2.XMLHTTP.3.0");
}
catch (e)
{
try
{
xmlHttpObj = new ActiveXObject("Microsoft.XMLHTTP");
}
catch(e)
{
// for IE7, Mozilla, Safari
xmlHttpObj = new XMLHttpRequest();
}
}
return xmlHttpObj;
}

```

+   由于除了 Internet Explorer 之外的浏览器都不支持 ActiveX 对象，因此使用`try`和`catch`块语句创建`XMLHTTPRequest`对象的实例，以便没有 JavaScript 错误，代码可以在跨浏览器中使用。

### 注意

如果你的网页已经使用了像 jQuery 或 Mootools 这样的 JavaScript 框架，你可以使用它们的核心 Ajax 函数。这些库通常发布了支持多个浏览器和平台的函数，并且随着时间的推移进行更新，因此强烈建议使用这样的 JavaScript 库。

# 美化 JavaScript

我们已经在上一章中看到了如何使用 JSMin 来压缩 JavaScript 代码。现在，让我们尝试反向工程压缩的 JavaScript 代码并美化它。我们可以使用工具**JsBeautifier**来解压缩和美化 JavaScript 代码。它可以直接从 URL [`jsbeautifier.org/`](http://jsbeautifier.org/)使用，或者你可以使用 URL [`github.com/einars/js-beautify/zipball/master`](http://github.com/einars/js-beautify/zipball/master)从 Github 下载代码。让我们首先看一下在使用 JSMin 压缩时`get_time()`函数中的代码是什么样的：

```php
function get_time(){$.ajax({cache:false,type:"GET",url:"ajax.php",error:function(){setTimeout(get_time,5000);},success:function(response){$('#timer_div').html(response);get_time();}});}

```

当 JavaScript 代码被压缩时，文件占用的空间更小，在网页中加载速度更快，但是当我们需要向该文件添加新功能时，编辑代码变得非常困难。在这种情况下，我们需要美化 JavaScript 代码并进行编辑。现在，让我们使用[`jsbeautifier.org/:`](http://jsbeautifier.org/)来获取美化后的 JavaScript 代码。

```php
function get_time() {
$.ajax({
cache: false,
type: "GET",
url: "ajax.php",
error: function () {
setTimeout(get_time, 5000);
},
success: function (response) {
$('#timer_div').html(response);
get_time();
}
});
}

```

### 注意

在生产服务器中，建议我们使用压缩的 JavaScript 代码，因为它占用的空间更小，加载速度比美化的代码格式更快。但是在开发服务器中，建议始终使用美化的代码，以便以后可以更改或编辑。
