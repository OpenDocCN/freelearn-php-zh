# 第八章。Ajax

对一些人来说，Ajax 是阿姆斯特丹的一支荷兰足球队的名字。对网络开发者来说，Ajax（也称为 **异步 JavaScript 和 XML**（**AJAX**））是用于客户端以异步方式从服务器检索数据的一组网络技术的总称。我以这个负载很重的句子开始本章，因为我总是喜欢在用到缩写时解释它。

A（异步）几乎总是存在的，J（JavaScript）是肯定的，因为我们谈论的是客户端，但 X（XML）不是强制的。通常 **JSON** 被用作客户端和服务器之间的数据格式。我们将在第十章，*XML 和 JSON*中讨论 XML 和 JSON。在我们的示例中，我们将使用已经熟悉的 HTML 格式。

使用这些技术，在数据在后台检索后，可以修改网站，并且可以更新屏幕的部分内容，而无需加载全新的页面。这样，我们的网站将开始更像是一个桌面应用程序，因此我们可以安全地称之为网络应用程序。

如果没有得到妥善管理，Ajax 有其缺点。由于它全部使用 JavaScript 实现，如果关闭 JavaScript，则期望的行为将不会发生。在 2015 年，这不应该是一个问题。还有其他缺点，但并非没有解决这些问题的方法。我们将立即在第九章，*历史 API – 不要忘记我们的位置*中解决这些问题。

# XMLHttpRequest

Ajax 技术依赖于可以在 JavaScript 代码中使用的 **XMLHttpRequest**（**XHR**）对象。它用于向网络服务器发送 HTTP 或 HTTPS 请求，并将服务器响应数据加载回脚本中。与其他许多网络技术一样，各种浏览器中的 XMLHttpRequest 实现有所不同。在这里，jQuery 将再次提供帮助。通过使用 jQuery 和它附带的相关 Ajax 方法，这些不兼容性不会引起任何担忧。

# Ajax 和 jQuery

jQuery 提供了几个方法，您可以使用它们来执行我们喜欢称之为 Ajax 调用的操作。最完整的一个不出所料地被称为 .`ajax()`。我们将从一个简单的例子开始。

许多基于 Ajax 的网站都是这样的——顶部有一个无处不在的菜单，底部有一些其他导航，以及一个不断变化内容的中心部分。我们无法强调得更多，当我们以我们在这里描述的方式使用 Ajax，并且我们的网站被命名为 `index.php` 时，无论中心部分的内容如何频繁变化，我们的当前网页仍然是 `index.php`。

因此，让我们假设我们有一个网站，顶部有一个主菜单，中间有一些基本内容，在一个具有 id `varicontent` 的 `<div>` 中。在初始加载时，`#varicontent` 里面的内容并不重要——通常是一个漂亮的图片横幅——这是解释当我们更改其内容并用不同内容替换它时会发生什么的章节。

## jQuery Ajax 方法

我们将向您介绍一些最有用的 jQuery 方法，用于使用 Ajax，从非常简单到更复杂。想象一下一家公司组织的研讨会和展览的官方网站。主页以及所有页面，因为我们从未离开过页面，都包含一个访客可以使用来导航和选择他们所选择主题的菜单。

因此，让我们先展示一些代码——只是一个片段，以说明它是如何完成的：

```php
<div id="mainmenu">
<ul class="dropdown">
<li><a class="htmla" href="oldsite/information.html"
id="information" >Information</a></li>
<li><a id="seminars" class="htmla agenda" href="oldsite/seminars.php">
Seminars</a></li>
<li><a class="htmla" id="exhibits"  href="oldsite/exhibits.php">Exhibits</a></li>

</ul>
</div>
<div id="varicontent">
<!-- code for things on the home page, maybe a photo banner -->
</div>
```

### $.load() 方法

我们将使用 `load()` 方法来加载的不是全新的页面，而是我们需要的精确 HTML，在页面中我们想要替换内容的部分：

```php
$("#mainmenu").on("click", "a.htmla", function(e){
e.preventDefault();
var topic = $(this).attr("id");
updateHTMLContent(topic);
});

function updateHTMLContent(topic) {
var loadfile = "./content/" + topic + ".html";

$('#varicontent').load(loadfile);
}
```

在刚刚给出的示例代码中，我们处理了任何具有类 `htmla` 的菜单项。目的是用位于服务器上的文件中的 HTML 替换 `#varicontent` 内的内容。注意，示例中还有一个 `href` 属性。我们打算不使用它，但如果需要 JavaScript 不受支持时的后备计划，它也可以存在。我们将在讨论 **渐进增强** 时再次提及这一点。在示例中，我们包括了一个指向位于 `oldsite` 文件夹中的文件的链接。

因此，在前面给出的代码中，我们有一个事件处理程序，用于当点击具有类 `htmla` 的 `<a>` 标签时。我们首先实际上阻止用户访问 `href` 标签中指定的链接，这会导致加载一个全新的页面。以下行处理了这一点：

```php
e.preventDefault();
```

相反，我们将使用 jQuery Ajax 方法 `.load()`。我们使用锚点的 ID 值来确定文件名，然后调用 `updateHTMLcontent()` 函数。

在那个函数内部，我们调用 `load()`，这将进行必要的 Ajax 调用来获取文件的内容。然后，我们将 `#varicontent <div>` 替换为那个内容。现在，屏幕的那部分已经更新，但我们仍然停留在同一个页面上，屏幕的其他部分保持不变。

### $.post()

在前面的例子中，我们只需要加载一些 HTML。如果我们想在服务器上执行 PHP 代码以动态创建 HTML，并将其插入到我们的页面中，那么 `.ajax()` 方法就派上用场了，特别是 `.ajax()` 的两个特殊情况：`.post()` 和 `.get()`。

如您所猜想的，区别在于，就像 HTML 表单一样，值传递给服务器的方式——是作为 POST 变量还是作为 GET 变量。让我们以第二个菜单项为例，它包含一个类为`agenda`的`<a>`标签。就像之前的例子一样，我们首先阻止浏览器加载`href`属性中指定的文件。这次，我们获取父元素的 ID 值：

```php
$("#mainmenu").on("click", "a.agenda", function(e){
e.preventDefault();
var nav = $(this).attr("id");
updateAgendaContent(nav);

});

function updateAgendaContent(nav) {

$.post("./showagendalist.php", { nav: nav},
 function(data){

$('#varicontent').replaceWith(data);

} );
}
```

现在，我们要求服务器找到服务器上的 PHP 文件`showagendalist.php`，并传递一些 POST 变量，或者就像我们的例子中那样，只传递一个。PHP 代码将在服务器上执行，无论它生成什么，我们都可以在函数中捕获它。我们将使用这个函数来使用方便的 JavaScript 方法`replaceWith()`将其插入到页面的适当部分。期望返回的默认格式是 HTML，但我们也可以使用其他格式，例如 JSON 和 XML，我们将在第十章中讨论，*XML 和 JSON*。

下面是一个这个 PHP 代码可能的样子：

```php
<?php
$today = time();
$nav = $_POST['nav'];
switch ($nav)
{

  case "seminars":
  $sql = 'SELECT * FROM seminars;';
  break;
  .
       /* ...  */
  default:
  break;
}

$mysqli = dbConnect($host, $usr, $pwd, $dbname) ;
date_default_timezone_set('Europe/Brussels');
$mysqli->set_charset("utf8");
$htmlstring = '<div id="varicontent"  ><div class="row">';

$result = $mysqli->query($sql);

while ($row= $result->fetch_assoc()
{

$htmlstring .= '<div class="agendaentry">';
$htmlstring .=  '<h5 class='title">'.$row['title'].'</h5>';
$htmlstring .= '<div class="summary" >'.'$row['summary'].'</div>';

$htmlstring .= '<div class=" body">'.$row['body'].'</div>';
$htmlstring .= '</div>';
}

$htmlstring .= '</div>';  //row
$htmlstring .= '</div>'; //varicontent

echo $htmlstring;
?>
```

在前面的 PHP 代码中，我们查看单个 POST 变量`$POST['nav']`的值，以确定访问者从菜单中选择了日历的哪个部分。那个部分的名称必须以某种方式与我们的数据库中的表匹配。接下来，我们从数据库中提取所有文章，假设有一个存储为文本的`title`列，以及存储为 HTML 的`summary`和`body`列。然后我们生成所有文章的完整 HTML。最后的语句是一个输出整个生成 HTML 的`echo`语句。

这是一个实际示例，用来展示 Ajax 是如何工作的，但在实际应用中并不实用。有两个原因导致`#varicontent <div>`中的内容会迅速变得过大：不仅我们展示了每篇文章的全部内容，我们还展示了所有文章。

我们不应该展示每篇文章的全部内容，而只应展示标题和摘要，而不是正文。在我们的生成 HTML 中，我们可以有一个类型为`hidden`的`<input>`来携带文章的 ID，然后我们将标题转换成锚点标签，使其可点击。这个锚点背后的 jQuery 事件处理器将触发另一个 Ajax 调用，点击后，包含文章标题和摘要列表的`#varicontent <div>`将被单个文章的完整内容所替换。但是，我们仍然停留在同一个页面上。

即使这样做也不够。一旦文章的数量变得很大，我们的`#varicontent`部分的长度将变得令人不快。为了克服这一点，我们需要应用某种分页，并且一次只显示一定数量的文章。我们将在第十三章中讨论一个具有**分页**小部件的框架，*Foundation - 一个响应式 CSS/JavaScript 框架*。在下面的示例代码中，我们已经考虑到了我们需要的一个额外参数，`offset`。稍后我们将提供适应这些变化的更新代码。我们在 PHP 代码中省略了`switch`语句。现有的 JavaScript 代码的唯一区别是现在`UpdateAgendaContent()`将接受一个额外的`offset`参数。我们只为额外的 Ajax 调用包括了额外的部分。

更新后的 PHP 代码如下：

```php
<?php
$nav = $_POST['nav'];
if (isset($_POST['offset']))
{
$offset = $_POST['offset'];
}
else {
  $offset = 0;
}

$sql = 'SELECT * FROM seminars  LIMIT '.$offset.',10;';
$mysqli = dbConnect($host, $usr, $pwd, $dbname) ;
date_default_timezone_set('Europe/Brussels');
$mysqli->set_charset("utf8");

$htmlstring = '<div id="varicontent"  ><div class="row">;
$result = $mysqli->query($sql);

while ($row = $result->fetch_assoc())
{

$htmlstring .= '<div class="agendaentry">';

$htmlstring .=  '<h5 class="title"><a class="aid">'.$row['title'].'<input type="hidden" name="aid" value="'.$row['a_id'].'"></input></a></h5>';
$htmlstring .= 
'<div class="summary" >'.$row['summary'].'</div>';
$htmlstring .= '</div>'; // agendaentry
}
$htmlstring .= '</div>';  //row
$htmlstring .= '</div>'; //varicontent
echo $htmlstring;
?>
```

额外的 JavaScript 代码：

```php
function updateAgendaContent(nav, offset) {

$.post("./showagendalist.php", { nav: nav, offset:offset}
 function(data){

  $('#varicontent').replaceWith(data);
  $("#varicontent").on("click", "a.aid", function(){
  var aid = $(this).parent().find('input').val();
  updateArticleContent (aid);
  });

} );
}

function updateArticleContent (aid)
{
 $.post("./showarticle.php", { aid: aid },
  function(data){
  $('#varicontent').replaceWith(data);

 } );

}
```

注意，我们现在在`updateAgendaContent()`函数内部添加了一个事件处理器，以在访问者点击文章标题时触发正确的事情。这通常是必要的，就像在我们的 HTML 标签示例中，我们想要触发在 Ajax 调用之前页面上不存在的事件。

### $.ajax()

正如我们提到的，`$.post()`方法（以及还有一个`$.get()`方法）是`$.ajax()`方法的一个特例。我们使用的参数比`.ajax()`少，因为其中一些是预定义的（不出所料，是 POST 或 GET），所以，jQuery 再次使我们更容易编写代码。为了总结这一章，我们将给出`.ajax()`方法的使用总结。请查阅完整的 jQuery 文档以获取更多详细信息。这将值得一读。

语法如下：

```php
$.ajax({name:value, name:value, ... })
```

这些参数指定一个或多个名称/值对。以下是常用参数及其含义的概述：

+   **data**: 这是发送到服务器的数据。如果它不是一个字符串，它将被转换成一个查询字符串。它可以作为一个对象、一个字符串或一个数组传递。

+   **dataType**: 这是服务器响应所期望的数据类型。它可以是以 XML、HTML、文本、JSON 或脚本。在我们的示例中，我们假设是 HTML。

+   **url**: 这指定了发送请求的 URL。默认是当前页面。

+   **error(xhr,status,error)**: 这是一个在请求失败时运行的函数。

+   **success(result,status,xhr)**: 这是一个在请求成功时运行的函数。

+   **type**: 这指定了请求的类型（GET 或 POST）。

因此，我们最后示例中的`$.post()`调用是：

```php
$.post("./showarticle.php", { aid: aid },
  function(data){
  $('#varicontent').replaceWith(data);

 } );
```

它也可以写成：

```php
$.ajax({

type: "POST",
url: "./showarticle.php",

succes: function(data){
  $('#varicontent').replaceWith(data);
}
data:{ aid: aid },
dataType: "html"

 } );
```

# 摘要

在这一章中，我们介绍了 Ajax，这是一组用于异步从服务器收集数据的网络技术。它用于更新网站和 Web 应用程序上的屏幕的特定部分，而不是每次都加载一个全新的页面。

Ajax 技术基于 XMLHttpRequest 对象，但多亏了 jQuery 和其 `$.ajax()` 方法，你学会了如何在你的应用程序中使用 Ajax，而无需了解该对象。我们在示例中使用了 `$.load` 和 `$.post` 方法来用存储在服务器上或由服务器生成的 HTML 替换屏幕的部分。

Ajax 可以与其他数据格式一起使用，例如 XML 和 JSON。但它也存在潜在缺点，因为现在我们不断更新页面而不实际离开它，这会让访问我们网站的访客将其感知为不同的页面，尤其是在他们按下浏览器的后退按钮时。

这两个主题：使返回键执行预期的功能，以及在客户端和服务器之间使用不同的数据格式，是我们接下来两章的主题内容。
