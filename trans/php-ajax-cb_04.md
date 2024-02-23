# 第四章.高级工具

在本章中，我们将涵盖以下主题：

+   使用 Comet 技术构建 Ajax 聊天系统

+   使用 JavaScript 绘制图表

+   通过画布解码验证码

+   在网格中显示数据

在本章中，我们将看看如何使用 Comet 技术构建一个简单的 Ajax 聊天应用程序。**Comet**是 Web 应用程序中的一种技术，可以在不需要客户端显式请求的情况下从 Web 服务器向客户端推送数据。在这个应用程序中，我们将使用这种简单的 Comet 技术，将聊天消息从服务器推送到浏览器，而不使用任何特殊的 Comet 服务器。

在*使用 JavaScript 绘制图表*部分，我们将看看如何使用 Google Visualization API 来使用 JavaScript 构建交互式图表。

之后，我们将展示如何使用 Firefox Greasemonkey 脚本，通过画布来解码浏览器上的简单验证码。

### 注意

这里使用的聊天应用程序不使用任何 Comet 服务器，如 APE（[`www.ape-project.org`](http://www.ape-project.org)）或 Livestreamer（[`www.livestream.com`](http://www.livestream.com)）。我们在这里只是试图展示如何使用 Ajax 进行长轮询，而不是传统的轮询来从服务器获取信息。

# 使用 Comet 技术构建 Ajax 聊天系统

现在，让我们看看如何使用长轮询技术构建一个简单的 Ajax 聊天系统。我们大部分使用了 JavaScript 的 jQuery 框架来编写 JavaScript 代码。在传统的 Ajax 轮询系统中，会定期向服务器发送请求；因此，无论是否有新数据，服务器都必须处理 HTTP 请求。但是在 Ajax 中，使用长轮询技术，请求会一直保持开放，直到服务器有新数据发送到浏览器。

然而，在我们的聊天示例中，我们将 Ajax 请求保持开放 90 秒。如果服务器没有收到新的聊天消息，连接将被关闭，并打开一个新的 Ajax 轮询。

## 准备工作

首先，让我们看看这个应用程序的界面是什么样子的：

![准备工作](img/3081_04_01.jpg)

这个聊天工具有一个非常简单的界面。您需要设置一个用户名来发送聊天消息。

## 如何做...

与这个 Comet 聊天系统相关的代码有不同类型。让我们逐个部分来看：

1.  以下 HTML 代码构成了聊天系统的布局：

```php
    <form name="chatform" id="chatform">
    <div id="chatwrapper">
    <h2>Ajax Chat Utility</h2>
    <div>
    User Name: <input id="username" type="text" maxlength="14" />
    </div>
    <div id="chattext" class="chatbox"> </div>
    <div>
    <input id="message" name="message" type="text" maxlength="100" />
    <input type="submit" name="submit" id="send" value="Send" />
    </div>
    </div>
    </form>

    ```

1.  现在让我们看看保存消息到文本文件并保持 Ajax 请求开放直到文件中保存了新消息的 PHP 代码。您可以在`chat-backend.php`文件中找到这段代码。

```php
    //set the maximum execution time to 90 seconds
    set_time_limit(91);
    //make sure this file is writable
    $file_name = 'chatdata.txt';
    //get the script entrance time
    $entrance_time = time();
    // store new message in the file
    //used for ajax call to store the mesage
    if(!empty($_GET['msg']) && !empty($_GET['user_name']))
    {
    $user_name = htmlentities($_GET['user_name'],ENT_QUOTES);
    $message = htmlentities(stripslashes($_GET['msg']),ENT_QUOTES);
    $message = '<div><b>'.$user_name.'</b> : '.$message.'</div>';
    file_put_contents($file_name,$message);
    exit();
    }
    //user for getting chat messages
    // infinite loop until the data file is not modified
    $last_modif = !empty($_GET['ts']) ? $_GET['ts'] : 0;
    $curr_ftime = filemtime($filename);
    //now get the difference
    while ($curr_ftime <= $last_modif && time()-$entrance_time<90) // check if the data file has been modified
    {
    //sleep for 500 micro seconds
    usleep(500000);
    //clear the file status cache
    clearstatcache();
    //get the file modified time
    $curr_ftime = filemtime($file_name);
    }
    // return a json encoded value
    $response = array();
    $response['msg'] = file_get_contents($file_name);
    $response['ts'] = $curr_ftime;
    echo json_encode($response);

    ```

1.  现在，让我们看看使聊天功能生效的 JavaScript 代码。

```php
    var Comet ={
    ts : 0 ,
    url : 'chat-backend.php',
    //to display the response
    show_response : function(message){
    $('#chattext').append(message);
    $('#chattext').scrollTop( $('#chattext').attr('scrollHeight') );
    },
    //validation fuction for empty user name or message
    validate : function()
    {
    if($.trim( $('#username').val() )=='')
    {
    alert('Please enter the username');
    return false;
    }
    else if($.trim( $('#message').val() )=='')
    {
    alert('Please enter chat message');
    return false;
    }
    else
    {
    return true;
    }
    },
    send_message : function()
    {
    if(this.validate())
    {
    var request_data = 'user_name='+$('#username').val()+'&msg='+$('#message').val();
    var request_url = this.url+'?'+request_data;
    //make the ajax call
    $.get(request_url);
    $('#message').val('');
    $('#message').focus();
    }
    cometused, for building Ajax chat},
    connect : function()
    {
    //call the ajax now to get the response
    $.ajax({
    url: this.url,
    data: 'ts='+this.ts,
    cache : false,
    dataType : 'json',
    success: function(data){
    //only add the response if file time has been modified
    if(data.ts>Comet.ts)
    {
    Comet.ts = data.ts;
    Comet.show_response(data.msg);
    }
    Comet.connect();
    },
    error : function(data)
    {
    //wait for 5 second before sending another request
    setTimeout(function(){
    Comet.connect()
    }, 5000);
    }
    });
    }
    };
    //event handler for DOM ready
    $(document).ready(function()
    {
    //call the comet connection function
    Comet.connect();
    //submit event handlder of the form
    $('#chatform').submit(function()
    {
    Comet.send_message();
    return false;
    });
    });

    ```

### 它是如何工作的...

现在，让我们看看这个 Ajax 聊天是如何与 Comet 实现一起工作的。它的一些方面如下：

1.  将聊天消息保存到文件中：

聊天消息被保存到文件中。在我们的应用程序中，只有最新的聊天消息被保存到文件中。之前的聊天消息被最新消息替换。

```php
    $user_name = htmlentities(stripslashes($_$_GET['user_name']),ENT_QUOTES);
    $message = htmlentities(stripslashes($_GET['msg']),ENT_QUOTES);
    $message = '<div><b>'.$user_name.'</b> : '.$message.'</div>';
    file_put_contents($file_name,$message);

    ```

消息的特殊字符被转换为 HTML 实体，以转换 HTML 特殊字符并避免聊天字符串中的格式错误。然后，带有用户名的消息存储在`$file_name`变量中。

1.  使用长 Ajax 轮询实现 Comet：

现在，让我们看看我们如何使用长 Ajax 轮询实现 Comet。

```php
    $entrance_time = time();

    ```

在代码的第一行，我们将 PHP 脚本的进入时间存储在`$entrance_time`变量中，以防止脚本执行超过 90 秒，如下所示：

```php
    set_time_limit(91);

    ```

在`chat-backend.php`代码的第一行中，我们将脚本的最大执行时间设置为`91`（秒），这样 PHP 在脚本的长时间执行时不会抛出致命错误；因为默认情况下，PHP 脚本的`max_execution_time`在`php.ini`文件中设置为`30`。

现在，让我们来看看主要的`while`循环，它会阻塞 Ajax 调用，直到接收到新的聊天消息为止：

```php
    $last_modif = !empty($_GET['ts']) ? $_GET['ts'] : 0;
    $curr_ftime = filemtime($filename);
    while ($curr_ftime <= $last_modif && time()-$entrance_time<90) {
    usleep(500000);
    clearstatcache();
    $curr_ftime = filemtime($file_name);
    }

    ```

我们将最后一次文件修改时间值存储在`$last_modif`变量中，将当前文件修改时间存储在`$curre_ftime`变量中。`while`循环一直执行，直到满足两个条件：第一个条件是文本文件的最后修改时间应大于或等于当前文件修改时间，第二个条件检查脚本执行时间是否达到 90 秒。因此，如果文件已被修改或脚本执行时间为 90 秒，则请求完成并将响应发送到浏览器。否则，请求将被长时间的 Ajax 轮询阻塞。

在 JavaScript 端，当 DOM 准备好进行操作时，我们调用`Comet.connect()`函数。此函数向`chat-backend.php`文件发出 Ajax 请求。现在，让我们看看这里如何处理 Ajax 响应：

```php
    success: function(data){
    if(data.ts>Comet.ts)
    {
    Comet.ts = data.ts;
    Comet.show_response(data.msg);
    }
    Comet.connect();
    },
    error : function(data)
    {
    setTimeout(function(){
    Comet.connect()
    }, 5000);
    }

    ```

当我们收到成功的 Ajax 响应时，我们会检查文件修改时间是否大于发送到服务器进行检查的时间戳。如果文件的修改时间已经改变，则满足此条件。在这种情况下，我们将`ts`变量赋值为文件修改时间的当前时间戳，并调用`show_response()`函数将最新的聊天消息显示给浏览器。然后立即调用`Comet.function()`。

如果 Ajax 请求出现错误，它会在发送另一个请求到`connect()`函数之前等待 5 秒。

1.  显示响应：

现在，让我们看一下响应是如何显示的：

```php
    show_response : function(message){
    $('#chattext').append(message);
    $('#chattext').scrollTop( $('#chattext').attr('scrollHeight') );
    },

    ```

在这个函数中，我们将 Ajax 响应附加到具有 ID`chattext`的`div`。之后，我们将`scrollTop`的值（如果存在滚动条，则表示滚动条的垂直位置）设置为`scrollHeight`。`ScrollHeight`属性给出元素的滚动视图的高度。

# 使用 JavaScript 制作图表

在本节中，我们将看一个示例，演示如何使用 Google 可视化的 JavaScript API 创建交互式图表。**Google 可视化 API**提供了一组强大的函数，用于创建不同类型的图表，如饼图、折线图、条形图等。在本节中，我们将简要地看一下如何使用此 API 来创建它们。

## 准备就绪

现在，让我们看一下使用 Google 可视化 API 创建不同样式的图表的基本步骤。我们将看一个示例，在其中我们在页面上创建条形图、折线图和饼图。现在，让我们通过使用可视化 API 来创建图表的初步步骤。

1.  放置图表容器：

首先，我们需要在网页中放置一个包含图表的 HTML 元素。通常，它应该是一个块级元素。让我们从流行的块级元素<div>开始，如下所示：

```php
    <div id="chart"></div>

    ```

请确保为此 HTML 元素分配一个 ID 属性，因为可以使用`document.getElementById()` JavaScript 函数传递此元素的引用。

1.  加载 Google 可视化 API：

创建图表容器后，让我们尝试在这里加载 Google 可视化 API，如下所示：

```php
    <script type="text/javascript" src="https://www.google.com/jsapi"></script>

    ```

在前面的代码片段中，我们在网页中包含了 Google JavaScript API。在包含 JavaScript 文件之后，我们现在需要加载 Google API 的可视化模块：

```php
    google.load("visualization", "1", {packages:["corechart"]});

    ```

在`load()`函数中，第一个参数是我们想要加载的模块的名称；在我们的情况下是`visualization`模块。第二个参数是模块的版本；这里是 1 是最新版本。在第三个参数中，我们指定了从模块中加载哪个特定的包。在我们的情况下，是`corechart`包。`corechart`库支持常见图表类型，如条形图、折线图和饼图。

一旦 JavaScript 库完全加载，我们需要使用 JavaScript API 的函数。为了帮助解决这种情况，Google 的 JavaScript API 提供了一个名为 setOnloadCallback()的函数；它允许我们在特定模块加载时添加回调函数：

```php
    google.setOnLoadCallback(draw_line_chart));

    ```

在上面的例子中，当 Google Visualization 库加载时，会调用名为`draw_line_chart`的用户定义函数。

学习如何加载 Google Visualization API 之后，让我们看一下绘制柱状图、折线图和饼图的示例。

## 工作原理...

现在，让我们看看使用可视化 API 创建的不同图表的外观：

![工作原理...](img/3081_04_02.jpg)

### 绘制折线图

现在我们知道创建的图表是什么样子的，让我们首先创建折线图。可以在上面的图像中看到折线图。完整的代码可以在代码包中提供的`line-chart.html`文件中找到。现在，让我们逐步创建折线图。

在本节中，我们将看到如何创建线图，以显示世界上两个主要城市纽约和伦敦的人口增长，并将它们与线图进行比较。

1.  为图表准备数据：

+   为了为图表准备数据，我们首先需要将数据存储在 Google Visualization API 中的`DataTable`类的对象中，以表示数组的二维数据。

```php
        var data = new google.visualization.DataTable();

        ```

+   现在，下一步是为图表添加列。我们在图表上显示两条线，显示纽约和伦敦的人口增长，以十年为单位。为此，我们需要使用`addColumn()`函数为`DataTable`对象创建三列：

```php
        data.addColumn('string', 'Year');
        data.addColumn('number', 'New York');
        data.addColumn('number', 'London');

        ```

+   接下来，使用`addRows()`函数创建三行空行。您还可以将数组传递给`addRows()`函数，以创建带有数据的行。我们将在创建柱状图时看到如何做到这一点。

```php
        data.addRows(3);

        ```

+   在创建空行之后，让我们使用`setValue()`函数在这些空行上设置值，如下所示：

```php
        data.setValue(0, 0, '1980');
        data.setValue(0, 1, 7071639);
        data.setValue(0, 2, 6805000);
        data.setValue(1, 0, '1990');
        data.setValue(1, 1, 7322564);
        data.setValue(1, 2, 6829300);
        data.setValue(2, 0, '2000');
        data.setValue(2, 1, 8008278);
        data.setValue(2, 2, 7322400);

        ```

`setValue()`函数的第一个和第二个参数表示矩阵的行和列。例如，值`1,2`表示矩阵的第二行和第三列。

1.  显示折线图：

在 data 变量中创建图表数据后，现在创建并显示图表：

```php
    var chart = new google.visualization.LineChart(document.getElementById('chart'));

    ```

在上面的代码中，我们正在使用 Google Visualization API 的 LineChart()函数在 ID.chart 的 div 中创建折线图。现在，图表对象已经创建，并且可以在 chart 变量中使用。

```php
    chart.draw(data, {width: 600, height: 360, title: 'Population by Years'});

    ```

现在，使用 draw()函数绘制图表，该函数接受两个参数：

+   第一个是图表的数据，它是`DataTable`类的对象。

+   第二个参数指定不同的选项，如宽度、高度、图表标题等。可以在[`code.google.com/apis/visualization/documentation/gallery/linechart.html`](http://code.google.com/apis/visualization/documentation/gallery/linechart.html)找到参数的完整列表。

图表是自动在 X 轴和 Y 轴上表示各自的值。

### 绘制柱状图

在本节中，我们将看到如何使用 Google Visualization API 绘制柱状图。在这个例子中，我们将使用与前一个例子中相同的数据来可视化伦敦和纽约的人口增长。

这个图表可以在上面图像的右侧看到。

1.  准备数据：

让我们看一下使用柱状图可视化创建数据的代码。为了保存图表数据，我们需要创建`DataTable()`类的实例，如下所示：

```php
    var data = new google.visualization.DataTable();
    data.addColumn('string', 'Year');
    data.addColumn('number', 'New York');
    data.addColumn('number', 'London');
    data.addRows([
    ['1980', 7071639,6805000],
    ['1990', 7322564,6829300],
    ['2000', 8008278,7322400]
    ]);

    ```

如前面的代码中所示，在为数据表添加列之后，我们使用`addRows()`函数添加了行。我们之前以不同的方式使用了这个函数，创建了空行。在这里，它将直接创建三行，带有数组的数据。

1.  显示柱状图：

准备好数据后，让我们在网页上绘制它：

```php
    var chart = new google.visualization.ColumnChart(document.getElementById('chart'));
    chart.draw(data, {width: 600, height: 360, title: 'Population by Years', hAxis: {title: 'Year'} , vAxis : {title: 'Population'}
    });

    ```

我们正在绘制一个宽度为 600 像素，高度为 360 像素的条形图，使用`object ColumnChart()`类。使用`hAxis`和`vAxix`选项，我们在水平轴上显示标签`Year`，在垂直轴上显示标签`Population`。您可以在[`code.google.com/apis/chart/interactive/docs/gallery/columnchart.html`](http://code.google.com/apis/chart/interactive/docs/gallery/columnchart.html)了解有关柱状图 API 的更多选项。

### 提示

`BarChart()`类也在 Google Visualization API 中可用，但它创建的是水平条形图。您可以在[`code.google.com/apis/chart/interactive/docs/gallery/barchart.html`](http://code.google.com/apis/chart/interactive/docs/gallery/barchart.html)找到更多关于这种类型图表的信息。

### 绘制 3D 饼图

在这一部分，我们将看到如何使用 Google Visualization API 创建饼图。此示例生成的饼图显示在前图的左侧。

在这个例子中，我们将分解开发简单网站所需的时间，并使用饼图进行可视化。

1.  准备数据：

让我们看看如何创建用于项目可视化的饼图数据。和往常一样，我们需要创建`DataTable()`类的实例来存储需要填充的数据。

```php
    var data = new google.visualization.DataTable();
    data.addColumn('string', 'Phase');
    data.addColumn('number', 'Hours spent');
    data.addRows([
    ['Analysis', 10],
    ['Designing', 25],
    ['Coding', 70],
    ['Testing', 15],
    ['Debugging', 30]
    ]);

    ```

如您在上面的代码中所见，我们正在创建两列来存储项目不同阶段所花费的时间的数据。第一列是`Phase`，第二列是`Hours spent`（在项目的特定阶段花费的时间）。

1.  展示饼图：

现在，让我们看一下实际的代码，它将在 ID 为 chart 的 div 上绘制饼图：

```php
    var chart = new google.visualization.PieChart(document.getElementById('chart'));
    chart.draw(data, {width: 600, height: 360, is3D: true, title: 'Project Overview'});

    ```

在上面的代码中，首先创建了`PieChart()`类的对象。然后，使用`draw()`函数绘制图表。饼图是通过将第 2 列中给定的总小时数作为 100%来绘制的。请注意，我们将`is3D`选项设置为`true`，以显示 3D 饼图。

# 通过 canvas 解码 CAPTCHA

**CAPTCHA**（或**Captcha**）是**C**ompletely **A**utomated **P**ublic **T**uring test to tell **C**omputers and **H**umans **A**part 的缩写，基于单词'capture'。它最初是由 Luis von Ahn，Manuel Blum，Nicholas J. Hopper 和 John Langford 创造的。**CAPTCHA**旨在阻止机器和机器人访问网页功能；通常放置在网页的注册表单中，以确保只有人类才能注册网站。通常，它基于计算机难以识别图像形式的文本。更多关于**OCR（光学字符识别）**的研究和先进技术正在削弱 Captcha 的概念，这反过来迫使对 Captcha 进行进一步研究。HTML5 的`canvas`元素通过 JavaScript 编程打开了通过解码的可能性。

### 注意

`canvas`元素是 HTML5 规范的一部分。它是由苹果在 WebKit 组件中引入的。之后，它被 Gecko 内核的浏览器采用，比如 Mozilla Firefox。目前，大多数浏览器都原生支持它或通过插件支持。早些时候，SVG 被推广为绘制形状的标准，但由于其速度和低级协议，canvas 变得更受欢迎。

## 准备工作

我们需要一个支持`canvas`的浏览器。一般来说，Firefox 和 Safari 内置支持 canvas。在 Internet Explorer 中显示 canvas，可能需要来自 Mozilla 或 Google 的插件。

### 注意

Google Chrome Frame（可在[`code.google.com/chrome/chromeframe/)`](http://code.google.com/chrome/chromeframe/)找到）是一个插件，它将 Chrome 的 JavaScript 引擎添加到 Internet Explorer；它也支持`canvas`。

`explorercanvas`（可在[`code.google.com/p/explorercanvas/)`](http://code.google.com/p/explorercanvas/)找到）是一个 JavaScript 库，添加后将`canvas`转换为 VML 并在 IE 上支持它。

## 如何做...

当一个由 Shaun 开发的 Greasemonkey 脚本能够识别 MegaUpload（文件共享网站）的验证码时，JavaScript 的 OCR 概念引起了人们的关注。对于文件共享网站，验证码是避免机器下载的一种方式，这可能来自竞争对手或盗版者。这里的 Greasemonkey 脚本使用了`canvas`及其通过 JavaScript 访问的能力。

### 注意

Greasemonkey 最初是一个 Firefox 扩展，用于在特定域和 URL 上执行用户脚本，当页面显示时改变外观或功能。现在，其他浏览器也开始在一定程度上支持 Greasemonkey 脚本。

完整的源代码可以在 Greasemonkey 的网站上找到—[`www.userscripts.org/scripts/review/38736`](http://www.userscripts.org/scripts/review/38736)。在这里，我们将使用`canvas`的 JavaScript 来审查这个概念：

1.  验证码图像加载到`canvas`并通过`getImageData()`读取图像数据。

1.  然后将图像转换为灰度。

1.  图像进一步分成三部分，每部分一个字符。对于 MegaUpload 的验证码来说，这更容易，因为它的距离是固定的。

1.  图像进一步处理以将其转换为两种颜色—黑色和白色

1.  进一步裁剪分割的图像以获得一种受体。

1.  然后将受体数据传递给神经网络以识别字符。神经网络数据预先使用以前运行的数据进行种植，以获得更好的匹配。

## 它是如何工作的...

以下图像显示了在 MegaUpload 网站上找到的一个示例验证码：

![它是如何工作的...](img/3081_04_03.jpg)

在这里，描述的每个处理阶段对于更好地识别验证码至关重要：

1.  将验证码图像加载到`canvas`：

验证码图像通过 Greasemonkey 的 Ajax 调用加载到画布上以获取图像：

```php
    var image = document.getElementById('captchaform').parentNode.getElementsByTagName('img')[0];
    GM_xmlhttpRequest( {
    method: 'GET',
    url: image.src,
    overrideMimeType: 'text/plain; charset=x-user-defined',
    onload: function (response) {
    load_image(response.responseText);
    }
    });

    ```

1.  将图像转换为灰度：

```php
    for (var x = 0; x < image_data.width; x++) {
    for (var y = 0; y < image_data.height; y++) {
    var i = x * 4 + y * 4 * image_data.width;
    var luma = Math.floor(image_data.data[i] * 299 / 1000 + image_data.data[i + 1] * 587 / 1000 + image_data.data[i + 2] * 114 / 1000);
    image_data.data[i] = luma;
    image_data.data[i + 1] = luma;
    image_data.data[i + 2] = luma;
    image_data.data[i + 3] = 255;
    }
    }

    ```

如前面的代码块所示，图像数据是逐像素采取的。每个像素的颜色值取平均值。最后，通过调整颜色值将图像转换为灰度。

1.  将图像转换为只有黑色和白色颜色：

```php
    for (var x = 0; x < image_data.width; x++) {
    for (var y = 0; y < image_data.height; y++) {
    var i = x * 4 + y * 4 * image_data.width;
    // Turn all the pixels of the certain colour to white
    if (image_data.data[i] == colour) {
    image_data.data[i] = 255;
    image_data.data[i + 1] = 255;
    image_data.data[i + 2] = 255;
    // Everything else to black
    }
    else {
    image_data.data[i] = 0;
    image_data.data[i + 1] = 0;
    image_data.data[i + 2] = 0;
    }
    }
    }

    ```

在这里，其他颜色可以称为“噪音”。通过保留只有黑色和白色颜色来去除“嘈杂”的颜色。

1.  裁剪不必要的图像数据：

由于图像的尺寸固定且文本距离固定，矩阵的矩形大小设置为去除不必要的数据，因此图像被裁剪。

```php
    cropped_canvas.getContext("2d").fillRect(0, 0, 20, 25);
    var edges = find_edges(image_data[i]);
    cropped_canvas.getContext("2d").drawImage(canvas, edges[0], edges[1], edges[2] - edges[0], edges[3] - edges[1], 0, 0, edges[2] - edges[0], edges[3] - edges[1]);

    ```

1.  应用神经网络：

**ANN（人工神经网络）**（或简称神经网络）是一种自学习的数学模型。它是一个自适应系统，根据其外部或内部信息流改变其结构。设计是模仿动物大脑的，因此每个处理器单元都有本地存储器和学习组件。

处理后的图像数据充当神经网络的受体。当传递给预先种植数据的神经网络时，它可以帮助我们找出验证码图像中的字符：

```php
    image_data[i] = cropped_canvas.getContext("2d").getImageData(0, 0, cropped_canvas.width, cropped_canvas.height);

    ```

根据验证码的复杂性，甚至可以在字符识别的最后一步使用线性代数。应用线性代数而不是神经网络可能会提高检测速度。但是，神经网络在各个方面表现相对更好。

## 还有更多...

`Canvas`还有其他有趣的应用。它预计将取代 Flash 组件。一些值得注意的画布应用程序如下：

+   CanvasPaint（[`canvaspaint.org/`](http://canvaspaint.org/)），界面类似于 MS Paint 应用程序

+   Highcharts（[`highcharts.com/)`](http://highcharts.com/)），一个使用`canvas`进行渲染的 JavaScript 图表 API

随机的验证码图像很难在没有人类干预的情况下破解。谷歌的

**reCAPTCHA API**围绕着使用数字化旧书的问题构建

OCR。当我们使用这个 reCAPTCHA API 时，它提供了一个带有 2 个文本的验证码：

1.  随机“已知”的验证码文本

1.  来自旧扫描书籍的“未知”文本-通过 OCR 很难辨认。用户填写这些验证码时，“已知”文本将用于验证。输入的文本与“未知”文本相匹配，用于数字化扫描的书籍。

一些网站提供 API 上的人类 Captcha 解码服务。验证码图像通过 API 上传；在另一部分，“数据输入”人类解码器将输入文本，然后将其发送回来。这些服务通常被自动机器人而不是人类使用。提供此类服务的一些网站如下：

+   Death By Captcha ([`www.deathbycaptcha.com/`](http://www.deathbycaptcha.com/))

+   DeCaptcher ([`www.decaptcher.com/`](http://www.decaptcher.com/))

# 在网格中显示数据

在 Web 2.0 网站中，“数据网格”一词通常指的是使用 HTML 表格的类似于电子表格/MS Excel 的显示。数据网格为用户提供了可用性和易于访问数据。数据网格的一些常见特性包括：

+   能够对数据进行分页

+   能够对列进行排序

+   能够对行进行排序

+   能够快速搜索或过滤数据字段

+   能够拥有冻结/固定行或标题

+   能够冻结列或标题

+   能够突出显示任何感兴趣的列

+   能够从不同的数据源加载，如 JSON、JavaScript 数组、DOM 和 Hijax

+   能够将数据导出到不同的格式

+   能够打印格式化数据

## 准备工作

我们将需要来自[`datatables.net/`](http://datatables.net/)的 DataTables jQuery 插件，以及 jQuery 核心。根据我们的需求，有时我们可能需要额外的插件。

## 如何做...

在简单的实现中（不使用任何其他数据源），将数据显示在 HTML 表格中就足够了。DataTables，不使用任何插件和额外选项，可以将其转换为类似电子表格的 UI，如下面的屏幕截图所示：

![如何做...](img/3081_04_04.jpg)

在 HTML 表格中，以正常的表格格式显示数据就足够了。在这里，我们使用以下代码显示具有姓名、电话号码、城市、邮政编码和国家名称的用户记录：

```php
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Strict//EN"
"http://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd">
<html >
<head>
<link rel="stylesheet" type="text/css" href="css/style.css" />
<script src="js/jquery.js" type="text/javascript">
</script>
<script src="js/jquery.dataTables.min.js" type="text/javascript">
</script>
<script src="js/script.js" type="text/javascript">
</script>
<title>jQuery DataTables</title>
</head>
<body>

<table id="grid" cellpadding="0" cellspacing="0" border="0"
class="display">
<thead>
<tr>
<th>Name</th>
<th>Phone</th>
<th>City</th>
<th>Zip</th>
<th>Country</th>
</tr>
</thead>
<tbody>
<tr>
<td>Garrett</td>
<td>1-606-901-3011</td>
<td>Indio</td>
<td>Q3R 3C6</td>
<td>Guatemala</td>
</tr>
<tr>
<td>Talon</td>
<td>1-319-542-9085</td>
<td>Kent</td>
<td>51552</td>
<td>Slovakia</td>
</tr>
</tr>
...
<tr>
<td>Bevis</td>
<td>1-710-939-1878</td>
<td>Lynwood</td>
<td>49756</td>
<td>El Salvador</td>
</tr>
<tr>
<td>Edward</td>
<td>1-431-901-7662</td>
<td>Guthrie</td>
<td>95899</td>
<td>Singapore</td>
</tr>
</tbody>
<tfoot>
<tr>
<th>Name</th>
<th>Phone</th>
<th>City</th>
<th>Zip</th>
<th>Country</th>
</tr>
</tfoot>
</table>
</body>
</html>

```

注意：在原始代码中，我们有 100 行。在这里，为简洁起见，许多行被剪掉了。

像往常一样，只需通过 jQuery 插件调用附加数据网格行为即可：

```php
jQuery(document).ready(function($){
 $('#grid').dataTable(); 

});

```

## 它是如何工作的...

DataTables 解析 HTML 表格中的数据，并将其保存在 JavaScript 对象数组中。在需要时，它会在其 HTML 模板中呈现内容。如前面的屏幕截图所示，它添加了一个搜索框、分页链接和一个下拉菜单，用于选择每页显示的记录数。包含在`thead`元素中的表头使用排序图标和链接进行装饰。当在搜索框中输入任何文本时，它会扫描保存的对象数组并重新绘制网格。对于快速将普通数据表转换为网格，这可能是相当足够的，但是 DataTables 除了选项和插件之外还提供了许多其他功能。

当需要关闭 DataTables 提供的某些功能时，我们可以通过选项来具体禁用它们，如下所示：

```php
$('#grid').dataTable({
'bPaginate':false,
'bSort':false
});

```

在这里，我们已禁用了分页元素和排序功能。同样，我们可以禁用任何其他功能。当我们不需要网格功能时，最好不要初始化 DataTables，而不是使用选项禁用功能，因为这会影响性能。

DataTables 的默认配置与 jQuery UI 主题框架不兼容；为了使其兼容，我们必须将`bJQueryUI`标志设置为`true:`

```php
$('#grid').dataTable({
'bJQueryUI': true
});

```

这样做的主要优势是更容易为所有 JavaScript 组件提供一致的主题/外观。

当用户滚动数据时，我们可能希望提供冻结的标题，以便值能够轻松地进行对应。为此，DataTables 提供了`FixedHeader`附加组件。设置固定标题很容易：

```php
var oTable = $('#grid').dataTable();
new FixedHeader(oTable);

```

使用 jQuery 的插件架构，我们可以轻松扩展 DataTables，从而添加任何网格功能。

## 还有更多...

不同的数据网格插件提供不同的用户界面和不同的功能。了解它们的区别总是很好的。有时，在一个繁重的 Ajax 网站上，我们可能想要显示数百万条记录。让我们看看有哪些工具可用于这些目的：

### 其他数据网格插件

我们有很多 jQuery 插件可用于数据网格。其中，以下是相对受欢迎并提供许多功能的：

+   jQuery Grid: [`www.trirand.com/blog/`](http://www.trirand.com/blog/)

+   Flexigrid: [`flexigrid.info/`](http://flexigrid.info/)

+   jqGridView: [`plugins.jquery.com/project/jqGridView`](http://plugins.jquery.com/project/jqGridView)

+   Ingrid: [`reconstrukt.com/ingrid/`](http://reconstrukt.com/ingrid/)

+   SlickGrid: [`github.com/mleibman/SlickGrid`](http://github.com/mleibman/SlickGrid)

+   TableSorter: [`tablesorter.com/`](http://tablesorter.com/)

当需要类似于这些插件中的任何一个的用户界面时，明智的做法是使用它们，而不是自定义 DataTables，如前一节所述。

### 显示数百万条数据项

在撰写本文时，并非所有数据网格实现都能容纳大量记录，除了 SlickGrid。有关其无限行的补丁和讨论可在[`github.com/mleibman/SlickGrid/tree/unlimited-rows`](http://https://github.com/mleibman/SlickGrid/tree/unlimited-rows)找到。
