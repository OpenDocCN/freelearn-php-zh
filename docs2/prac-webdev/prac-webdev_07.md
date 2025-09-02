# 第七章 jQuery

我认为**jQuery**是网络开发中最酷的事情之一。我正在做一些自学，按照我提到的课程时间，在美丽的卢汶大学城，那里有一个同样美丽的计算机书店。正是在这里我发现了 jQuery。

每一章都依赖于并使用 jQuery，因此从一开始就非常重要的是要很好地了解 jQuery。那么，jQuery 究竟是什么呢？它是一个流行的 JavaScript 库，使用它的总体好处是你可以编写更干净、更紧凑的代码。

那么，什么是库呢？嗯，它可能是一个有很多书的场所。我自己喜欢安特卫普的 Plantijn-Moretus 博物馆的那个，他们在几个世纪前都自己印刷书籍。鲁本斯是插图画家和宅邸肖像画家。UNIX 和 Java 人士认为库是已经编译好的代码，通常包含预定义的函数，这些函数放置在特殊格式的文件中，并且可以与程序本身一起加载。JavaScript 库也可以包含预定义的函数，但除此之外，它们只包含可读的 JavaScript 代码。

大多数 JavaScript 库有两种格式，常规或最小化，通常称为`name.js`和`name.min.js`。这些最小化版本去除了所有空格、换行符等，以减小文件大小，从而减少下载时间。要使用它们，你可以简单地将它们包含在你的程序中。

### 注意

jQuery 使用 CSS 样式选择器，这是我们已经在第三章中了解到的，*CSS*，来访问 DOM 元素。除了让你能更快、更干净地编写 JavaScript 代码外，jQuery 还会处理某些事情，例如解决浏览器兼容性问题，因此你不再需要自己编写这些代码。

# 获取 jQuery 库

你可以从[jquery.com](http://jquery.com)下载 jQuery。一旦进入生产阶段，你可能想使用最小化版本。在开发过程中，我会坚持使用更容易阅读的版本，这样你就可以不时地添加代码进行调试，只要你别忘了稍后将其删除。

# 在你的页面上如何放置 jQuery 库？

并不是每个人都同意在这里应该做什么。总是会有关于加载文件所需时间的担忧。如果你在文档的`<head>`部分或紧接在关闭`</body>`标签之前加载 jQuery 库，将会产生差异——例如：

```php
<script src="img/jquery.js"></script>
```

担忧的是加载库所需的时间。我有时会将所有是库的东西，换句话说，即那些不实际做任何事情但提供功能的东西，放在文件的头部部分。这意味着功能将在 HTML 之前加载。

你将始终与自己的 JavaScript 文件一起使用 jQuery，例如，`mycode.js`。此文件将包含以下内容中的 jQuery 代码行：

```php
$(document).ready(function(){
  }
```

您放入那里的所有代码将在页面加载后执行。因此，此文件需要放在您的文件中，在构成您页面的所有 HTML 之后，最好是在关闭`</body>`标签之前：

```php
<script type="text/javascript" src="img/mycode.js"></script>
```

# jQuery UI 和 jQuery Mobile

**jQuery UI**和**jQuery Mobile**是 jQuery 家族的两个附加库。jQuery **用户界面**（**UI**）为您提供了许多小部件，您可以在网站的用户界面部分使用。我最喜欢的有`accordion`和`datepicker`。与仅使用 JavaScript 的 jQuery 本身不同，jQuery UI 还附带了一整套 CSS 文件。这意味着，一旦您使用这些小部件之一，它们将具有自己的外观和感觉。

不必担心，jQuery 提供了一个名为**Themeroller**的酷工具。使用这个工具，您可以生成一组定制的 CSS 文件，以便 jQuery UI 元素的色彩和其他外观感觉功能与您的样式表相匹配。

jQuery mobile 是另一个基于 jQuery 的 CSS/JavaScript 扩展，它为您提供了用户界面元素来创建**移动优先**的网站和应用。您可以创建将在手机、平板电脑和桌面上工作的网页。我们将在这本书中专门用一章来介绍移动优先，然后在后面介绍响应式设计。现在，请记住 jQuery Mobile 是一个框架，它允许您在您的 Web 应用中利用手机的硬件和设备，例如拥有一个与您在手机上设置闹钟相同行为的日期选择器，通过点击网页上的电话号码自动拨打电话等。

我们在这本书中不会使用 jQuery Mobile。相反，我们将介绍一个名为**Foundation**的不同的 CSS/JavaScript 框架。

# 使用 jQuery 选择器和方法

使 jQuery 如此易于使用的原因之一是，您可以使用 CSS 样式选择器在您的页面上查找元素。因此，您不必学习更多的 JavaScript 方法，只需使用您已经知道的方法。在实际代码中，这意味着您不必使用以下：

```php
var content = document.getElementById("content");
```

您可以使用：

```php
var content = $('#content');
```

`$ ()`是`jQuery()`的缩写表示法。如果您同时使用 jQuery 和类似库，如`Dojo`，请查看文档了解如何将两者区分开来。

上述语句将创建一个 jQuery 对象，该对象将包含零个或一个 DOM 元素（因为只能有一个具有`id`内容的元素）。下一个语句可能包含很多元素，多达具有`green`类的元素数量：

```php
var greens = $('.green');
```

如您所见，这与 CSS 中的用法相同，`#`用于`id`，`.`用于`class`。

现在我们可以开始使用方法了。如果我们想将所有具有`green`类的元素更改为`yellow`类，怎么办？为此，我们可以使用`addClass`方法：

```php
$('green').addClass('yellow');
```

当然，你可能想将绿色替换为黄色，所以类`green`也必须被移除，你可以使用`removeClass`方法。jQuery 方法的伟大特性之一是你可以将它们嵌套，所以我们可以用一行代码完成所有这些事情：

```php
$('green').removeClass('green').addClass('yellow');
```

现在，我们将向您介绍一些非常有用的方法，您可以使用它们即时更改您的网页，或者找出页面上现在有什么。这些大多数都是*获取器*以及*设置器*。这意味着你可以获取元素的值，或者，如果你指定了一个参数，这将用于设置值。我们开始吧。

## `html()`

使用这种方法，你可以获取或设置一个元素的 HTML 内容：

```php
var contenthtml = $('#content).html(); // Get all the html code inside #content

var newcontent = 
'<div><h2 class="unique">This is a header</h2><p>This is a paragraph</p><p>This is a second paragraph</p></div>';

$('#content').html(newcontent);
```

第二组指令将用前面的 HTML 代码替换`#content`的内容。

## `text()`

类似于前一种方法，但有所不同，这个方法获取/设置 HTML 标签内的文本：

```php
var oldheader = $('h2.unique').text();  // Get only the text inside the h2 element

$('h2.unique').text("This is the new text in the header"); // Replace the text
```

## `attr()`

这种方法让我们可以操作属性的值：

```php
var linkvalue = $('a.unique').attr('href'); // get the href attribute value of an anchor tag

$('a.unique').attr('href', 'http://www.paulpwellens.com');
```

## `.val()`

这种方法获取或设置元素的值：

```php
var inputvalue =  $('input.name').val();
```

## `show()` 和 `hide()`

这些是在动态网页中非常有用的方法。非常经常，你想要为屏幕的一部分生成 HTML 代码，然后显示该代码，同时也做相反的事情：让屏幕的一部分消失。实现这一点的简单技术是准备一个`<div>`元素的内容，将其插入到`<div>`元素中，然后使其可见。你会在自定义 JavaScript 文件中这样做：

```php
$('#content').show();

$('#content').hide();
```

这些方法也非常有用，可以防止一种称为**未处理内容闪烁**（**FOUC**）的现象。我们这是什么意思？还记得关于在哪里放置你的 jQuery 库和自定义 JavaScript 文件的讨论吗？让我们假设你使用一个通过操作图像的无序列表（`<ul>`）来创建幻灯片的 JavaScript 插件（`<img>`）。所以，你的原始 HTML 可能看起来像这样：

```php
<div id="slideshow">
<ul>
   <li>
   <img src="img/mono.png" alt="Mono Lake">
   </li>
   <li>
   <img src="img/monumentvalley.png" alt="Monument Valley">
   </li>
   <li>
   <img src="img/centraalstation.png" alt="Central Station">
   </li>
</ul>
</div>
```

你正在使用的 jQuery 插件，例如，**OWL Carousel** ([owlgraphic.com](http://owlgraphic.com)) 将将这段 HTML 代码转换成一个华丽的幻灯片。然而，如果连接相当慢，你可能会首先看到所有图片堆叠在一起，前面有一个愚蠢的子弹，然后几秒钟后幻灯片才出现。你可以通过在你的样式表中放置以下内容来解决这个问题：

```php
#slideshow {
display:none;
}
```

然后，在你的自定义 JavaScript 文件中，在构建幻灯片动画的代码之后，你包含以下内容：

```php
$('#slideshow').show();
```

这将导致屏幕的这一部分在幻灯片准备好之前不渲染任何内容。

## `.find()`

`.find`方法非常强大。你可以在你的页面上找到几乎所有东西，然后对结果进行操作。以下是一个例子：

```php
var address = $('#record').find('p.address');
```

这将查找并找到具有 `address` 类的所有 `<p>` 元素，在具有 ID `record` 的元素中。再一次，你可以嵌套或链式调用它与其他方法。下一个例子查找具有 `id` 属性的元素，然后查找它的值：

```php
var id = $('#record').find('input[name="id"]').val();
```

## .parent()

使用 `.parent()` 可以更加强大。第一个例子查找指定的 `<td>` 的 `<tr>` 元素，下一个例子向上查找三个级别：

```php
var tablerow = $('td.name').parent().html()  ;// obtain the html code of the parent table row

var greatgranddad = $('a.threedeep').parent().parent().parent() ;
```

## .next()

`.next()` 方法是向右查看的。它返回 DOM 结构右侧元素的兄弟元素，如果没有可选选择器匹配，则不返回。`.next()` 只查找紧挨着的元素，所以如果你想查找更多，你需要链式调用它：

```php
var maybe =  $('div.left').next('.middle'); // only returns something if the element to the
//next is of class middle

var  threetotheright = 
$('#thistable').find('td.first').next().next().next();
    // in a table with first last address zipcode city records this would get us the zip
```

## .css()

使用 jQuery，你也可以获取或设置任何给定元素的 CSS 值。在第一个例子中，我们检索 `background-color`；在第二个例子中，我们设置 `color`：

```php
var bgcolor = $('#content').css("background-color");
$('#color').css("color", "teal");
```

# jQuery 文档

完整的 jQuery 文档以及更多酷方法的描述可以在 [jquery.com](http://jquery.com) 找到。

市面上有很多 jQuery 书籍，可能太多以至于难以确定哪一本适合你；这本书不是其中之一。我们将只带你了解所有你需要进行 Web 开发的技术——jQuery 是其中之一。

如果你需要选择你的第一本 jQuery 书籍，我推荐 Packt Publishing 的最新版 *Learning jQuery*。

# 事件处理程序和 jQuery

恭喜你，你已经达到了本书的一个重大里程碑。这一页一次性介绍了几个新概念，你将每天都会用到很多。假设你正在构建一个带有菜单的网站。菜单是用有序列表构建的，下面是一个菜单项的代码：

```php
<li id="intnews"><a href="oldsite/intnews.php" class="news">International</a></li>
```

在你的自定义 JavaScript 文件中，你有：

```php
$("#mainmenu").on("click", "a.news", function(e){

e.preventDefault();

var nav = $(this).parent().attr("id");

updateNewsContent(nav);

});
```

让我们先讨论 `.on()` 部分。这是 jQuery 执行 **事件处理** 的方式。当网站访客执行某些操作时，就会发生事件。一个典型的操作是在按钮或链接上点击鼠标。然后我们可以捕获该事件，并在我们的事件处理函数中执行某些任务。在上一个例子中，我们在名为 `updateNewsContent()` 的函数中执行了所有这些操作。

重要的是要重复一点，你的 jQuery 代码只能在 DOM 元素加载后访问和操作。所以，如果你在 JavaScript 执行后动态创建 HTML，它们将不会被你的 JavaScript 代码操作。

`.on()` 方法的用法只能帮助。在例子中，我们将 `.on()` 方法附加到 `#mainmenu` 元素上；很可能是一个 `<div>`，它从页面初始加载就一直在那里。

`.on()` 方法的第一个参数是事件本身；在我们的例子中，是 `click` 事件。第二个参数是可选的，但我经常使用，它描述了我们想要触发事件的选择器。这使得这个事件成为所谓的 **委托** 事件。我们也可以这样写：

```php
$("#mainmenu a.news").on("click",  function(e){

// same code here

});
```

这使其成为所谓的**直接事件**。有什么区别呢？如果一个带有 class news 的`<a>`标签在页面初始加载后动态地添加到`#mainmenu` div 内部，并且被点击，那么委托事件会捕获它，而直接事件则不会。

这是在开发过程中常见的惊喜之一，注意到之前工作正常的东西突然似乎没有做任何事情。通常的补救措施是在你的事件处理程序内部启动一个新的事件处理程序。我们将在后面的章节中提供一个示例。

最后，是函数本身。请注意，它可以有一个参数。在函数内部，你可以访问那个事件对象并应用方法以及访问属性。你将在下一章中看到一些示例。

## preventDefault()

样本 HTML 代码包含一个带有`href`属性的锚标签（`<a>`），基本上创建了一个链接到 PHP 文件`intnews.php`。让我们假设我们使用一个古老的浏览器，或者一个 JavaScript 被关闭的浏览器。用户点击`<a>`标签会导致`intnews.php`被打开。

使用 JavaScript，我们在事件处理程序代码中确定了我们想要发生的事情，所以我们不希望链接发生，我们只想停留在同一页面上。`preventDefault()`方法确实会做它名字所暗示的事情。这种技术是所谓的**渐进增强**的一部分，这将在后面的章节中讨论。

## $(this)

而在样本 JavaScript 代码中，你可能时不时地会看到**this**，但在 jQuery 中我们使用`$(this)`。在函数内部，它代表匹配元素（s）的 jQuery 对象。

## updateNewsContent()

我们故意没有描述这个样本函数内部的操作。在这个程序的部分，通常会发生的是使用 PHP 和可能是一个数据库从网络服务器检索数据。你已经看到了一些使用经典 Web 开发技术的示例，例如在表单中将 PHP 文件指定为操作参数，这将迫使我们转到另一个页面。

然而，我们将使用允许我们在同一页面上执行 PHP 代码的 jQuery 方法。这些方法所使用的底层技术被称为**AJAX**，这是下一章的主题。

# 摘要

在本章中，我们第一次开始远离传统的 Web 开发。我们介绍了 jQuery，这是一个强大的 JavaScript 库，它允许我们编写更干净、更紧凑的 JavaScript 代码。此外，我们将更容易做到这一点，因为它使用 CSS 样式选择器来指定 DOM 元素，而不是我们不得不学习的 JavaScript 方法。

如何下载 jQuery 库以及将其放置在哪里已经解释过了。我们使用 jQuery 最有用和最强大的方法来举例说明本章的其余部分。为了总结本章，我们介绍了 jQuery 创建事件处理程序的方式，这可能是本书这一阶段最重要的概念之一。

在下一章中，我们将继续使用 jQuery，不是用来遍历 DOM 和随意更改一些内容，而是通过执行使用 jQuery 方法调用的 PHP 代码，在 Web 服务器上生成我们页面的整个块。然后我们可以使用这些数据来更新我们页面的部分，而无需离开它。所有这些操作都是通过一种称为 AJAX 的技术实现的。
