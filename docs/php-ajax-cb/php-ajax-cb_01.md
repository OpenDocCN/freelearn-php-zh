# 第一章。Ajax 库

在本章中，我们将涵盖：

+   使用 jQuery 设计简单导航

+   创建选项卡导航

+   使用 Ext JS 设计组件

+   在 MochiKit 中处理事件

+   使用 Dojo 构建选项卡导航

+   使用 YUI 库构建图表应用程序

+   使用 jQuery 滑块加载动态内容

+   使用 MooTools 创建 Ajax 购物车

+   使用 prototype.js 构建 Ajax 登录表单

在本章中，我们将学习如何使用最著名的 JavaScript 库和框架的 Ajax 功能。这些库是根据我们的主观意见选择的，我们并不试图说哪个库/框架更好或更差。它们每个都有其优点和缺点。

# 使用 jQuery 设计简单导航

**jQuery**是一个开发框架，允许我们在 HTML 文档中使用 JavaScript。现在我们将使用基本的 jQuery 功能构建一个简单的导航。

## 准备就绪

在我们开始之前，我们需要包含最新的 jQuery 库。我们可以从[www.jquery.com](http://www.jquery.com)的下载部分下载它。我们将把它保存在名为`js`的 JavaScript 文件夹中，放在我们 HTML 文档的根目录中，例如`cookbook`。

本书中提到的所有库也可以在在线缓存中找到，例如[`code.google.com/apis/libraries/`](http://code.google.com/apis/libraries)。

### 注意

您可以从您在[`www.PacktPub.com`](http://www.PacktPub.com)购买的所有 Packt 图书的帐户中下载示例代码文件。如果您在其他地方购买了这本书，您可以访问[`www.PacktPub.com/support`](http://www.PacktPub.com/support)并注册，以便直接通过电子邮件接收文件。

## 如何做...

现在，我们可以开始编写我们的`task1.html`页面。我们将把它放在`cookbook`文件夹中。

```php
<!doctype html>
<html>
<head>
<title>Example 1</title>
</head>
<body>
<ul id="navigation">
<li id="home"><a href="#">Home</a></li>
<li class="active"><a href="#">Our Books</a></li>
<li><a href="#">Shop</a></li>
<li><a href="#">Blog</a></li>
</ul>
<div id="placeHolder">
<!-- our content goes here -->
</div>
<script src=js/jquery.min.js></"></script>
<script>
$(document).ready(function(){
$('#navigation li a').each(function(){
var $item = $(this);
$item.bind('click',function(event){
event.preventDefault();
var title = $item.html();
var html = title + ' was selected.';
$('#placeHolder').html(html);
});
});
$.get('ajax/test.html', function(data) {
$('.result').html(data);
alert('Load was performed.');
});
});
</script>
</body>
</html>

```

## 它是如何工作的...

现在，让我们解释一下在前面的代码片段中做了什么。我们脚本的主要思想是在文档中找到每个超链接`<a>`，阻止其默认功能，并在我们的`placeHolder`中显示超链接内容。从一开始，我们从`doctype`和主 HTML 布局开始。页面的主体包含用于动态内容的`navigation`和`placeholder`元素。

jQuery 功能最重要的部分是包含我们的 jQuery 库。让我们将它放在关闭`<body>`标签之前。这将允许页面的 HTML 首先加载：

```php
<script src="js/jquery.min.js"></script>

```

在加载我们的 HTML 页面并且文档准备就绪后，我们可以在`$(document).ready()`函数中定义我们的 JavaScript 脚本：

```php
<script>
$(document).ready(function(){
alert("Hello jQuery!");
});
</script>

```

这也可以缩短为`$():`

```php
<script>
$(function(){
alert("Hello jQuery!");
});
</script>

```

美元符号`$()`代表对`jQuery()`工厂函数的别名。在这个函数中，我们可以使用所有的 CSS 选择器，如 ID，类，或确切的标签名称。例如：

+   `$('a'):`选择文档中的所有超链接

+   `$('#myID'):`选择具有此 ID 的元素

+   `$('.myID'):`选择具有此类的所有元素

在我们的情况下，我们正在选择`navigation <div>`中的所有超链接，并为`click`事件定义它们自己的功能：

```php
$item.bind('click',function(event){
// prevent default functionality
event.preventDefault();
// here goes the rest
});

```

我们示例的最后一步是创建`title` VAR 和 HTML 字符串，它将进入`placeHolder`：

```php
var title = $(this).html();
var html = title + ' was selected.';
$('#placeHolder').html(html);

```

## 还有更多...

前面的例子非常简单。但是 jQuery 还有很多功能可以提供给我们。这包括特殊选择器，效果，DOM 操作或 Ajax 功能。

我们可以更精确地指定我们的选择器。例如，我们可以根据它们的`href`属性指定应该受到影响的超链接：

```php
$('a[href^=mailto:]').addClass('mailto);
$('a[href$=.pdf]').addClass('pdf');
$('a[href^=http] [href*=milan]').addClass('milan');

```

jQuery 还涵盖了所有可能的事件（`click`，`blur，focus，dblclick`等），视觉效果（`hide`，`show，toggle，fadeIn，fadeOut`等），或 DOM 操作（`appendTo`，`prependTo`等）。它具有完整的 AJAX 功能，非常容易使用，例如：

```php
$.get('test.html', function(data) {
$('.result').html(data);
});

```

但是我们将在进一步的任务和章节中更仔细地了解更多 jQuery 功能。

## 另请参阅

第一章,*使用 jQuery 进行 AJAX*

第二章,*jQuery UI*

第三章,*使用 jQuery 创建选项卡导航*

# 创建选项卡导航

**jQuery UI**是由 jQuery 的核心交互插件构建的。作为一个高级框架，它使得对每个开发人员来说创建效果和动画变得容易。现在我们将使用 jQuery UI 构建一个选项卡导航。

## 准备工作

首先，我们需要从[www.jquery.com](http://www.jquery.com)包含 jQuery 库，如果我们在前面的步骤中还没有这样做。然后，我们可以从[www.jqueryui.com/download](http://www.jqueryui.com/download)下载 jQuery UI 库。在这个页面上，我们可以下载特定的模块或整个库。我们可以选择我们喜欢的主题，或者使用高级主题设置创建自己的主题。现在，我们将选择带有`ui-lightness`主题的整个库。

## 如何做...

1.  现在我们已经准备好编码了。让我们从 HTML 部分开始。这部分将定义一个带有三个选项卡和一个手风琴的`navigation`元素。

```php
    <body>
    <div id="navigation">
    <ul>
    <li><a href="#tabs-1">Home</a></li>
    <li><a href="#tabs-2">Our Books</a></li>
    <li><a href="http://ajax/shop.html">Shop</a></li>
    </ul>
    <div id="tabs-1">
    <p>Lorem ipsum dolor 1</p>
    </div>
    <div id="tabs-2">
    <p>Lorem ipsum dolor 2</p>
    </div>
    </div>
    </body>

    ```

1.  当 HTML 准备好后，我们可以继续使用 CSS 和 JavaScript CSS 样式在`<head>`标签中，如下面的代码所示：

```php
    <head>
    <link href="css/ui-lightness/jquery-ui.custom.css"
    rel="stylesheet" />
    </head>

    ```

1.  在`<body>`标签关闭之前，我们将添加 JavaScript：

```php
    <script src="js/jquery.min.js"></script>
    <script src="js/jquery-ui.custom.min.js"></script>
    <script>
    $(document).ready(function(){
    $('#navigation').tabs();
    });
    </script>
    </body>

    ```

1.  我们的结果如下所示：![如何做...](img/3081_01_01.jpg)

## 它是如何工作的...

下载的 jQuery UI 包含所选主题的整个 CSS 内容（jquery-ui.custom.css）。我们需要做的就是在`<head>`标签中包含它：

```php
...
<link href="css/ui-lightness/jquery-ui.custom.css"
rel="stylesheet" />

```

在 CSS 之后，我们包括 jQuery 和 jQuery UI 库：

```php
<script src="js/jquery.min.js"></script>
<script src="js/jquery-ui.custom.min.js"></script>

```

JavaScript 部分非常简单：

```php
$('#navigation').tabs();

```

重要的是要适应所需的 HTML 结构。每个超链接都将目标 HTML 内容定位到选定的`<div>`标签中。为了在它们之间创建关系，我们将在每个超链接中使用`#id`，以及选定`<div>`标签的 ID（例如，`tabs-1`）。

在第三个选项卡中有一个例外，它通过 Ajax 加载所请求的数据。在这种情况下，我们不定义任何目标区域，因为它将自动创建。正如你所看到的，使用 jQuery UI 中的 Ajax 非常简单和舒适。

## 还有更多...

jQuery UI 为我们提供了很多选项。我们可以只使用前面代码片段中呈现的默认功能，也可以使用一些附加功能：

| 通过 Ajax 获取内容： | `$( "#navigation" ).tabs({ajaxOptions: {} })`; |
| --- | --- |
| 鼠标悬停时打开： | `$( "#navigation" ).tabs({event: "mouseover"})`; |
| 折叠内容： | `$( "#navigation" ).tabs({collapsible: true})`; |
| 可排序的： | `$( "navigation" ).tabs().find( ".ui-tabs-nav" ).sortable({ axis: "x" })`; |
| Cookie 持久性： | `$( "#navigation" ).tabs({cookie: { expires: 1 }})`; |

## 另请参阅

第三章,*使用 jQuery 设计组件*

# 使用 Ext JS 设计组件

**Ext JS**是一个 JavaScript 框架，提供了许多跨浏览器用户界面小部件。Ext JS 的核心是基于组件设计构建的，可以很容易地扩展以满足我们的需求。

## 准备工作

我们可以从[www.sencha.com](http://www.sencha.com)的 Ext JS 部分下载最新版本的 Ext JS 框架。现在，我们已经准备好使用两列和一个手风琴构建经典的 Ext JS 布局。我们还可以准备一个简单的 HTML 文件`ajax/center-content.html`来测试 Ajax 功能：

```php
…
<body>
<p>Center content</p>
</body>
…

```

## 如何做...

1.  首先，我们将包括像 CSS 和 Ext JS 库文件这样的强制性文件。

```php
    <link rel="stylesheet" href="css/ext-all.css" />
    <script src="js/ext-base.js"></script>
    <script src="js/ext-all.js"></script>

    ```

1.  我们将继续使用`onReady`函数，它将运行我们的脚本：

```php
    <script type="text/javascript">
    Ext.onReady(function(){
    var viewport = new Ext.Viewport({
    layout:'border',
    items:[{
    region:'west',
    id:'west-panel',
    title:'West',
    split:true,
    width: 200,
    layout:'accordion',
    items: [{
    html: 'Navigation content',
    title:'Navigation'
    },{
    title:'Settings',
    html: 'Settings content'
    }]
    },{
    region:'center',
    layout:'column',
    autoLoad:{
    url: 'ajax/center-content.html',
    method:'GET'
    }
    }]
    });
    });
    </script>

    ```

1.  我们的带有手风琴导航的布局已经准备好了：![如何做...](img/3081_01_02.jpg)

## 它是如何工作的...

Ext JS 是为开发人员构建的，以使他们的生活更轻松。正如您在源代码中所看到的，我们已经使用一个简单的 JavaScript 对象构建了一个布局。我们有一个“Viewport”和两个项目。一个位于左侧（区域：**West)**，第二个位于右侧（区域：**East)**。在这种情况下，我们不必关心 CSS。一切都由 Ext JS 直接处理，通过我们的变量如`width, margins, cmargins`等。`layout`属性非常强大。**West**侧的内部布局是一个手风琴，其中包含**Navigation**和**Settings**。在中心列，我们可以看到通过 Ajax 加载的内容，使用`autoLoad`方法。

## 还有更多...

布局的可能选项包括：Absolute，Anchor，Card，Column，Fit，Table，Vbox 和 Hbox。

# MochiKit 中的事件处理

本章中的下一个轻量级库是**MochiKit**。在这个任务中，我们将构建一个用于列出`onkeydown`和`onkeypress`事件的脚本。在每个事件之后，我们将显示按下的键及其键代码和键字符串。

## 准备工作

所有必需的文件、文档和演示都可以在[www.mochikit.com](http://www.mochikit.com)上找到。我们需要下载整个 MochiKit 库并将其保存在我们的`js`文件夹中。请注意，`MochiKit.js`只是一个主文件，其中包含了来自 MochiKit 的所有必要子模块（如`base.js, signal.js, DOM.js`等）。Ajax 请求的登陆页面将是`ajax/actions.php:`

```php
<?php
if($_GET["action"] && $_GET["key"]) {
// our logic for processing given data
} else {
echo "No params provided";
}
?>

```

## 如何做...

1.  让我们从 HTML 代码开始：

```php
    <table>
    <tr>
    <th>Event</th>
    <th>Key Code</th>
    <th>Key String</th>
    </tr>
    <tr>
    <td>onkeydown</td>
    <td id="onkeydown_code">-</td>
    <td id="onkeydown_string">-</td>
    </tr>
    <tr>
    <td>onkeypress</td>
    <td id="onkeypress_code">-</td>
    <td id="onkeypress_string">-</td>
    </tr>
    </table>

    ```

1.  包括 MochiKit 框架：

```php
    <script type="text/javascript" src="js/MochiKit/MochiKit.js"> </script>

    ```

1.  定义 JavaScript 功能：

```php
    <script>
    connect(document, 'onkeydown',
    function(e) {
    var key = e.key();
    replaceChildNodes('onkeydown_code', key.code);
    replaceChildNodes('onkeydown_string', key.string);
    doSimpleXMLHttpRequest("ajax/actions.php",
    { action: "keydown", key: key.code});
    });
    connect(document, 'onkeypress',
    function(e) {
    var key = e.key();
    replaceChildNodes('onkeypress_code', key.code);
    replaceChildNodes('onkeypress_string', key.string);
    doSimpleXMLHttpRequest("ajax/actions.php",
    { action: "keypress", key: key.code});
    });
    </script>

    ```

1.  我们的结果是：![How to do it...](img/3081_01_03.jpg)

## 它是如何工作的...

`connect()`函数将信号（Mochikit.Signal API 参考）连接到插槽。在我们的情况下，我们将我们的文档连接到`onkeydown`和`onkeypress`处理程序以调用一个`function(e)`。参数`e`表示我们的事件对象，当`key()`对象引用返回键代码和字符串时。

`replaceChildNodes(node[, childNode[,...]])`是 Mochikit.DOM API 参考的一个函数，它从给定的 DOM 元素中删除所有子元素，然后将给定的`childNode`附加到其中。

在每个`onkeydown`和`onkeypress`事件之后，我们都会使用`doSimpleXMLHttpRequest()`函数发送一个 Ajax 调用。在我们的示例中，我们页面的请求看起来像`ajax/actions.php?action=onkeydown&key=87`。

## 还有更多...

任何具有连接插槽的对象都可以通过`disconnect()`或`disconnectAll()`函数断开连接。在我们想要仅使用`connect()`一次的情况下，我们可以使用`connectOnce()`函数，这将在信号处理程序触发后自动断开信号处理程序。

MochiKit 允许我们充分利用现有的浏览器生成的事件，但其中一些事件并不是所有浏览器都原生支持的。MochiKit 能够合成这些事件，其中包括`onmouseenter, onmouseleave`和`onmousewheel`。

# 使用 Dojo 构建选项卡导航

现在我们将看一下 Dojo JavaScript 库。我们将使用`Dojo Toolkit`（dojoToolKit）的基本功能构建一个简单的选项卡导航。

## 准备工作

我们需要从 Google CDN（[`ajax.googleapis.com/ajax/libs/dojo/1.5/dojo/dojo.xd.js`](http://ajax.googleapis.com/ajax/libs/dojo/1.5/dojo/dojo.xd.js)）或 AOL CDN（[`o.aolcdn.com/dojo/1.5/dojo/dojo.xd.js.`](http://o.aolcdn.com/dojo/1.5/dojo/dojo.xd.js)）等网站上包含 Dojo Toolkit。

如果您想要下载整个 Dojo SDK，您可以在[www.dojotoolkit.org/download](http://www.dojotoolkit.org/download)找到它。

Ajax 请求的登陆页面将是`ajax/content1.html:`

```php
<body>
<h1>Operation completed.</h1>
</body>

```

## 如何做...

1.  我们将在文档的`<head>`标签中包含来自`claro`主题（包含在`dojoToolKit`中）的样式：

```php
    <link rel="stylesheet" type="text/css" href="http://js/dojoToolKit/dijit/themes/claro/claro.css" />

    ```

1.  我们将在我们的文档的主体中定义我们的 HTML 代码：

```php
    <body class="claro">
    <div>
    <div dojoType="dijit.layout.TabContainer">
    <div dojoType="dijit.layout.ContentPane"
    title="Our first tab" selected="true">
    <div id="showMe">
    click here to see how it works
    </div>
    </div>
    <div dojoType="dijit.layout.ContentPane"
    title="Our second tab">
    Lorem ipsum - the second
    </div>
    <div dojoType="dijit.layout.ContentPane"
    title="Our last tab" closable="true">
    Lorem ipsum - the last...
    </div>
    </div>
    </div>
    </body>

    ```

1.  当 HTML 和 CSS 准备就绪时，我们将包含所需的模块`DojoToolkit`：

```php
    <script type="text/javascript"
    src="js/dojoToolKit/dojo/dojo.js"
    djConfig="parseOnLoad: true"></script>
    <script type="text/javascript">
    dojo.require("dijit.layout.TabContainer");
    dojo.require("dijit.layout.ContentPane");
    </script>

    ```

1.  添加 JavaScript 功能给我们带来了以下结果：

```php
    <script type="text/javascript">
    dojo.addOnLoad(function() {
    if (document.pub) { document.pub(); }
    dojo.query("#showMe").onclick(function(e) {
    dojo.xhrGet({
    url: "ajax/content1.html",
    load: function(result) {
    alert("The loaded content is: " + result);
    }
    });
    var node = e.target;
    node.innerHTML = "wow, that was easy!";
    });
    });
    </script>

    ```

1.  当上述代码片段准备好并保存后，我们的结果将是一个带有三个选项卡的简单选项卡导航。![如何做...](img/3081_01_04.jpg)

## 工作原理...

正如您在源代码中所看到的，我们正在使用 Dijit-Dojo UI 组件系统。**Dijit**包含在 Dojo SDK 中，并包括四种支持的主题的 UI 组件，`(nihilo soria, tundra,and claro)`。我们可以通过选择`<body>`标签中的一个类来设置我们想要使用的主题。在前面的例子中，我们有`class="claro"`。

当我们包含`dojoToolKit`脚本时，需要为`djConfig`属性提供`parseOnLoad:true`。如果没有这个，Dojo 将无法找到应该转换为 Dijit 小部件的页面元素。

当我们想要使用特定的小部件时，我们需要调用小部件的所需类（`dojo.require("dijit.layout.TabContainer")`），并提供其`dojoType`属性（`dojoType="dijit.layout.TabContainer"`）。作为在 Dojo 中使用 Ajax 的示例，我们使用`dojo.xhrGet()`函数每次点击`showMe` div 时获取`ajax/content1.html`的内容。

# 使用 YUI 库构建图表应用程序

在这个任务中，我们将使用 Yahoo!开发的 UI 库来构建一个图表。

## 准备工作

YUI 库可以在 Yahoo!的开发者网站([`developer.yahoo.com/yui/3`](http://developer.yahoo.com/yui/3))上下载。将其保存在我们的`js`文件夹中后，我们就可以开始编程了。

## 如何做...

1.  我们必须首先在文档的`<head>`标签中包含 YUI 库以及我们图表的占位符的样式：

```php
    <script type="text/javascript" src="js/yui-min.js"></script>
    <style>
    #mychart {
    margin:10px;
    width:90%; max-width: 800px; height:400px;
    }
    </style>

    ```

1.  我们将把我们的 HTML 放在`<body>`标签中，以标记我们的图表将放置的位置：

```php
    <div id="mychart"></div>

    ```

1.  我们的 JavaScript 如下：

```php
    <script type="text/javascript">
    (function() {
    YUI().use('charts', function (Y){
    //dataProvider source
    var myDataValues = [
    {date:"January" , windows:2000, mac:800, linux:200},
    {date:"February", windows:3000, mac:1200, linux:300},
    {date:"March" , windows:3500, mac:1900, linux:1400},
    {date:"April" , windows:3000, mac:2800, linux:200},
    {date:"May" , windows:1500, mac:3500, linux:700},
    {date:"June" , windows:2000, mac:3000, linux:250}
    ];
    //Define our axes for the chart.
    var myAxes = {
    financials:{
    keys:["windows", "mac", "linux"],
    position:"right", type:"numeric"
    },
    dateRange:{
    keys:["date"],
    position:"bottom",type:"category"
    }
    };
    //instantiate the chart
    var myChart = new Y.Chart({
    type:"column", categoryKey:"date",
    dataProvider:myDataValues, axes:myAxes,
    horizontalGridlines: true,
    verticalGridlines: true,
    render:"#mychart"
    });
    });
    })();</script>

    ```

1.  保存并打开我们的 HTML 文档后的结果如下：![如何做...](img/3081_01_05.jpg)

## 工作原理...

YUI 图表是在`Chart`对象中定义的。对于“文档准备就绪”函数，我们将使用`(function(){...})()`语法。我们需要指定我们要使用`YUI() 'charts'`。

主要部分是创建一个`Y.Chart`对象。我们可以定义这个图表的渲染方式，网格线的外观，图表的显示位置以及要显示的数据。我们将使用`myAxes`对象定义坐标轴，该对象处理侧边的图例。我们的数据存储在`myDataValues`对象中。

## 还有更多...

有许多可能性和方法来设置我们的图表样式。我们可以将图表分割成最小的部分并设置每个属性。例如，标签的旋转或边距：

```php
styles:{
label: {rotation:-45, margin:{top:5}}
}

```

YUI 还包括 Ajax 功能。一个简单的 Ajax 调用将如下所示：

```php
<div id="content">
<p>Place for a replacing text</p>
</div>
<p><a href="http://ajax/content.html" onclick="return callAjax();">Call Ajax</a></p>
<script type="text/javascript">
//<![CDATA[
function callAjax(){
var sUrl = "http://ajax/content.html";
var callback = {
success: function(o) {
document.getElementById('content')
.innerHTML = o.responseText;
},
failure: function(o) {
alert("Request failed.");
}
}
var transaction = YAHOO.util.Connect
.asyncRequest('GET', sUrl, callback, null);
return false;
}
//]]>
</script>

```

我们创建了`callAjax()`函数，当点击`Call Ajax`超链接时触发该函数。Ajax 调用由`YAHOO.util.Connect.asyngRequest()`提供。我们定义了 HTTP 方法（GET），请求的 URL`ajax/content.html`，以及`callback`功能与`success`方法，该方法在`'content' <div>`中显示响应文本。

# 使用 jQuery 滑块加载动态内容

在这个任务中，我们将学习如何使用 jQuery 滑块动态加载页面内容。

## 准备工作

在这个任务中，我们也将使用 jQuery UI 库。我们可以从[`jqueryui.com/download`](http://jqueryui.com/download)下载 jQuery UI 库，也可以从一些 CDN 上下载。然后我们将为我们的小项目创建一个名为`packt1`的文件夹。在我们的`packt1`文件夹中将有更多的文件夹；这些是通过 Ajax 加载的 HTML 文件的`ajax`文件夹，用于我们的样式的 CSS 文件夹，以及用于我们的 JavaScript 库的`js`文件夹。

文件夹结构将如下所示：

```php
Packt1/
ajax/
content1.html
content2.html
content3-broken.php
items.html
css/ - all stylesheets
js/
ui/ - all jQuery UI resources
jquery-1.4.4.js
index.html

```

## 如何做...

一切都准备好了，我们可以开始了。

1.  我们将从基本的 HTML 布局和内容开始。这部分已经包括了一个链接到我们的 CSS，来自 jQuery UI 库。我们可以将其保存为`index.html:`

```php
    <!DOCTYPE html>
    <html lang="en">
    <head>
    <title>Ajax using jQuery</title>
    <link href="css/ui-lightness/jquery-ui.custom.css"
    rel="stylesheet" />
    </head>
    <body>
    <div class="demo">
    <div id="tabs">
    <ul>
    <li><a href="#tabs-1">Home</a></li>
    <li><a href="http://ajax/content1.html">Books</a></li>
    <li><a href="http://ajax/content2.html">FAQ</a></li>
    <li><a href="http://ajax/content3-broken.php">
    Contact(broken) </a>
    </li>
    </ul>
    <div id="tabs-1">
    This content is preloaded.
    </div>
    </div>
    </div>
    </body>
    </html>

    ```

1.  现在我们将添加 JavaScript 库及其功能：

```php
    <script src="js/jquery-1.4.4.js"></script>
    <script src="js/ui/jquery-ui.min.js"></script>
    <script>
    $(function() {
    $("#tabs").tabs({
    ajaxOptions: {
    success: function(){
    $("#slider").slider({
    range: true,
    min: 1,
    max: 10,
    values: [1,10],
    slide: function( event, ui ) {
    $("#amount").val(ui.values[0] + " to " +
    ui.values[1]);
    },
    change: function(event, ui) {
    var start = ui.values[0];
    var end = ui.values[1];
    $('#result').html('');
    for(var i = start; i <= end; i++){
    var $item = $('<h3></h3>');
    $item
    .load('ajax/items.html #item-'+i);
    .appendTo($('#result'));
    } }
    });
    },
    error: function(xhr, status, index, anchor) {
    $(anchor.hash).html(
    "Couldn't load this tab. We'll try to fix
    this as soon as possible. " +
    "If this wouldn't be a demo." );
    }
    }
    });
    });
    </script>

    ```

1.  我们的`index.html`页面已经准备好了，我们可以创建要通过 Ajax 在我们的页面中加载的文件。

第一页将是 ajax/content1.html。此页面将包含一个具有额外功能的滑块，稍后将进行描述。

```php
    <h2>Slider</h2>
    <p>
    <label for="amount">Displaying items:</label>
    <input type="text" id="amount" style="border:0;
    color:#f6931f; font-weight:bold;" value="none" />
    </p>
    <div id="slider"></div>
    <div id="result"></div>

    ```

1.  第二页将是`ajax/content2.html:`

```php
    <p><strong>This tab was loaded using ajax.</strong></p>
    <p>Lorem ipsum dolor sit amet, consectetur adipiscing elit. Aenean nec turpis justo, et facilisis ligula.</p>

    ```

我们 Ajax 文件夹中的最后一个文件将是 items.html：

```php
    <div id="item-1">Item 1</div>
    <div id="item-2">Item 2</div>
    <div id="item-3">Item 3</div>
    <div id="item-4">Item 4</div>
    <div id="item-5">Item 5</div>
    <div id="item-6">Item 6</div>
    <div id="item-7">Item 7</div>
    <div id="item-8">Item 8</div>
    <div id="item-9">Item 9</div>
    <div id="item-10">Item 10</div>

    ```

1.  现在，如下面的屏幕截图所示，我们有一个具有四个选项卡的多功能页面。其中三个通过 Ajax 加载，其中一个包含一个滑块。这个滑块有额外的功能，每次更改都会加载选定数量的商品。![如何做...](img/3081_01_06.jpg)

## 它是如何工作的...

从一开始，我们使用 jQuery UI 库创建了一个简单的带有四个选项卡的选项卡布局。其中一个(#tabs-1)直接包含在`index.html`文件中。jQuery UI 库允许我们定义`ajaxOptions`，以便我们可以通过 Ajax 加载我们的内容。我们在每个超链接的`href`属性之前找到所需内容的导航。如果此目标不存在，则会触发`error`方法。

我们希望在我们的第二个选项卡(名为**Books**)上有一个功能性的滑块。为了使其工作，我们不能在`$(document).ready()`函数中初始化它，因为它的 HTML 内容尚未创建。我们将仅在`success`方法中需要时添加滑块初始化。

在每次滑块更改后，都会触发`load()`函数。此函数通过 Ajax 加载给定目标的内容。在我们的情况下，我们使用了一个更具体的选择器，具有对象的确切 ID，该 ID 显示在我们的结果框中。

## 还有更多...

在这项任务中，我们只使用了基本的`load()`函数，但 jQuery 提供了更多的 Ajax 方法，如下表所示:

| `$.ajax` | 执行 Ajax 请求 |
| --- | --- |
| `jQuery.post()` | 使用 HTTP POST 请求从服务器加载数据 |
| `jQuery.get()` | 使用 HTTP GET 请求从服务器加载数据 |
| `jQuery.getJSON()` | 使用 HTTP GET 请求从服务器加载 JSON 数据 |
| `jQuery.getScript()` | 使用 HTTP GET 请求从服务器加载并执行 JavaScript 文件 |

## 另请参阅

第三章,*使用 jQuery 的有用工具*

# 使用 MooTools 创建 Ajax 购物车

这项任务将向我们展示如何在 MooTools JavaScript 框架中使用 Ajax。我们将构建一个带有拖放功能的购物车。在每次 UI 解释添加新商品到购物车后，我们将向服务器发送一个 HTTP POST 请求。

## 准备工作

**MooTools**可在[`mootools.net/download`](http://https://mootools.net/download)或 Google 的 CDN 上下载。为了在服务器和客户端之间进行通信，我们将在我们的`ajax`文件夹中创建一个新文件，例如`addItem.php:`

```php
<?php
if($_POST['type']=='Item'){
echo 'New Item was added successfuly.';
}
?>

```

创建了这个虚拟的 PHP 文件后，我们准备继续进行此任务的编程部分。

## 如何做...

1.  我们将像通常一样从 HTML 布局开始，包括 MooTools 库：

```php
    <!doctype html>
    <html>
    <head>
    <title>Ajax Using MooTools</title>
    </head>
    <body>
    <div id="items">
    <div class="item">
    <span>Shirt 1</span>
    </div>
    <div class="item">
    <span>Shirt 2</span>
    </div>
    <div class="item">
    <span>Shirt 3</span>
    </div>
    <div class="item">
    <span>Shirt 4</span>
    </div>
    <div class="item">
    <span>Shirt 5</span>
    </div>
    <div class="item">
    <span>Shirt 6</span>
    </div>
    </div>
    <div id="cart">
    <div class="info">Drag Items Here</div>
    </div>
    <h3 id="result"></h3>
    <script src="js/mootools-core-1.3-full.js"></script>
    <script src="js/mootools-more-1.3-full.js"></script>
    <script src="js/mootools-art-0.87.js"></script>
    </body>
    </html>

    ```

1.  在这项任务中，我们必须提供自己的 CSS 样式：

```php
    <style>
    #items {
    float: left; border: 1px solid #F9F9F9; width: 525px;
    }
    item {
    background-color: #DDD;
    float: left;
    height: 100px;
    margin: 10px;
    width: 100px;
    position: relative;
    }
    item span {
    bottom: 0;
    left: 0;
    position: absolute;
    width: 100%;
    }
    #cart {
    border: 1px solid #F9F9F9;
    float: right;
    padding-bottom: 50px;
    width: 195px;
    }
    #cart .info {
    text-align: center;
    }
    #cart .item {
    background-color: green;
    border-width: 1px;
    cursor: default;
    height: 85px;
    margin: 5px;
    width: 85px;
    }
    </style>

    ```

1.  当我们的 UI 外观符合我们的期望时，我们可以开始 JavaScript：

```php
    <script>
    window.addEvent('domready', function(){
    $('.item').addEvent('mousedown', function(event){
    event.stop();
    var shirt = this;
    var clone = shirt.clone()
    .setStyles(shirt.getCoordinates())
    .setStyles({
    opacity: 0.6,
    position: 'absolute'
    })
    .inject(document.body);
    var drag = new Drag.Move(clone, {
    droppables: $('cart'),
    onDrop: function(dragging, cart){
    dragging.destroy();
    new Request.HTML({
    url: 'ajax/addItem.php',
    onRequest: function(){
    $('result').set('text', 'loading...');
    console.log('loading...');
    },
    onComplete: function(response){
    $('result').empty().adopt(response);
    console.log(response);
    }a
    }).post('type=shirt');
    if (cart != null){
    shirt.clone().inject(cart);
    cart.highlight('#7389AE', '#FFF');
    }
    },
    onCancel: function(dragging){
    dragging.destroy();
    }
    });
    drag.start(event);
    });
    });
    </script>

    ```

1.  一旦我们保存了我们的代码，我们的购物车就准备好了。结果如下:![如何做...](img/3081_01_07.jpg)

## 它是如何工作的...

`$(document).ready`函数通过将`domready`事件绑定到`window`对象来执行。对于每个项目，我们都会添加一个`mousedown`事件，其中包含将每个项目添加到购物车中的整个过程，使用`Drag`对象和`clone()`函数。

为了与服务器通信，我们使用`Request.HTML`方法，并使用`HTTP post`方法发送它，带有`post`变量`type`。如果变量`type`等于字符串`shirt`，这意味着新商品已添加到购物车，并且信息框结果已更新为`'新商品已成功添加'`。

## 还有更多...

`Class Request`代表处理`XMLHttpRequest`的主要类:

```php
var myRequest = new Request([options]);

```

前述模板的示例如下:

```php
var request = new Request({
url: 'sample.php', data: { sample: 'sample1'},
onComplete: function(text, xml){
$('result').set('text ', text);
}

```

在 MooTools 库的核心中，`Request`类被扩展为`Request.HTML`和`Request.JSON`。

`Request.HTML`是专门用于接收 HTML 数据的扩展`Request`类：

```php
new Request.HTML({
url: 'sample.php',
onRequest: function(){
console.log('loading...');
},
onComplete: function(response){
$('result').empty().adopt(response);
}
}).post('id=242');

```

我们可以使用`post`或`get`方法：

```php
new Request.HTML([options]).get({'id': 242});

```

作为客户端和服务器之间有效的通信实践，我们可以使用`Request.JSON`以`JSON`格式接收和传输 JavaScript 对象。

```php
var jsonRequest = new Request.JSON({
url: 'sample.php', onSuccess: function(author){
alert(author.firstname); // "Milan".
alert(author.lastname); // "Sedliak"
alert(author.company); // "Skype"
}}).get({bookTitle: 'PHP Ajax CookBook', 'bookID': 654});

```

# 使用 prototype.js 构建 Ajax 登录表单

本章中最后一个 JavaScript 框架是`prototype.js`。在这个任务中，我们将使用 Ajax 功能制作一个简单的登录表单。我们将看一下在 Ajax 中最常用的`prototype.js`实践。

## 准备工作

我们可以从[`www.prototypejs.org/download`](http://www.prototypejs.org/download)下载`prototype.js`。然后，只需将其保存在`js`文件夹中。要完成这个任务，我们需要让 Apache 服务器运行。

## 如何做...

1.  首先，让我们创建我们的虚拟`.php`文件，`login.php:`

```php
    <?php
    if($_POST['username']==$_POST['password']){
    echo 'proceed';
    }
    ?>

    ```

然后，我们可以继续进行 HTML 布局。

```php
    <!DOCTYPE html>
    <html>
    <head>
    </head>
    <body>
    <form id="loginForm">
    <label for="username">Username: </label>
    <input type="text" id="username" name="username" />
    <br />
    <label for="password">Password:</label>
    <input type="password" id="password" name="password"/>
    <br /><br />
    <input type="submit" value="Sign In" id="submit" />
    </form>
    </body>
    </html>

    ```

1.  当 HTML 设置好后，我们将定义我们的 JavaScript：

```php
    <script src="js/prototype.js"></script>
    <script>
    $('submit').observe('click', login);
    function login(e) {
    Event.stop(e);
    var url = "ajax/login.php";
    new Ajax.Request(url, {
    method: 'post',
    parameters: {
    username: document.getElementById('username').value,
    password: document.getElementById('password').value
    },
    onSuccess: process,
    onFailure: function() {
    alert("There was an error with the connection");
    }
    });
    }
    function process(transport) {
    var response = transport.responseText;
    if(response == 'proceed'){
    $('loginForm').hide();
    var my_div = document.createElement('div');
    my_div.appendChild(document.createTextNode("You are logged in!"));
    document.body.appendChild(my_div);
    }
    else
    alert("Sorry, your username and password don't match.");
    }
    </script>

    ```

### 它是如何工作的...

正如您在源代码中所看到的，我们在 ID 为`submit`的按钮元素上`observe`了一个新的`click`事件，这是我们登录表单中的`submit`按钮。`login()`函数由`click`事件触发。`submit`按钮的默认行为被`Event.stop(event)`替换，因此触发了 HTTP 请求的行为被禁用。而是创建了一个 Ajax 请求。`Ajax.Request`是在`prototype.js`中使用 Ajax 的基本类。我们使用了两个参数（用户名和密码）的`post`方法。如果请求成功，并且来自`login.php`的响应文本是`proceed`，那么我们成功登录了。

### 还有更多...

`prototype.js`将`Ajax.Request`对象扩展到了更多功能，如下所述：

+   Ajax.Updater:

Ajax.Updater 是`Ajax.Request`对象的扩展，它执行 Ajax 请求并根据响应文本更新容器：

```php
    <div id="container">Send the request</div>
    <script>
    $('submit').observe('click', login);
    function login(){
    new Ajax.Updater(
    'saladContainer', 'login.php', { method: 'post' }
    );
    })
    </script>

    ```

+   **Ajax.PeriodicalUpdater:**

在我们需要定期更新内容的情况下，我们可以使用周期性更新器：

```php
    new Ajax.PeriodicalUpdater('items', '/items', {
    method: 'get', frequency: 3, decay: 2
    });

    ```

频率表示更新内容的周期性（以秒为单位）。在上面的代码片段中，我们的内容将每 3 秒更新一次。

+   **Ajax.Responders:**

`Ajax.Responders`表示全局监听器的存储库，用于监视页面上的所有 Ajax 活动：

```php
    Ajax.Responders.register(responder)
    Ajax.Responders.unregister(responder)

    ```

使用 responders，我们可以轻松跟踪页面上有多少个 Ajax 请求是活动的。

```php
    Ajax.Responders.register({
    onCreate: function() {
    Ajax.activeRequestCount++;
    },
    onComplete: function() {
    Ajax.activeRequestCount--;
    }
    });

    ```
