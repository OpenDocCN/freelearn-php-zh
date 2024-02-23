# 第二章。基本实用程序

在本章中，我们将涵盖：

+   使用 Ajax 验证表单

+   创建一个自动建议控件

+   制作表单向导

+   使用 Ajax 上传文件

+   使用 Ajax 上传多个文件

+   创建一个五星评分系统

+   使用 Ajax 构建一个带有验证的 PHP 联系表单

+   在 Ajax 中显示表格

+   使用 PHP 和 Ajax 构建分页

在本章中，我们将学习如何构建基本的 Ajax 表单。我们将尝试理解在哪里可以使用 Ajax 方法，以及在哪里不能。我们可以使用 Ajax 的方式有很多种。以下是一些基于用户体验和特定系统性能的“最佳”实践。Ajax 使我们的生活更轻松，更快速，更好；如何以及在哪里使用取决于我们。

# 使用 Ajax 验证表单

Ajax 的主要思想是实时从服务器获取数据，而不需要重新加载整个页面。在这个任务中，我们将使用 Ajax 构建一个带有验证的简单表单。

## 准备就绪

由于在此任务中使用了 JavaScript 库，我们将选择 jQuery。我们将下载（如果我们还没有下载）并将其包含在我们的页面中。我们需要准备一些虚拟的 PHP 代码来检索验证结果。在这个例子中，让我们将其命名为`inputValidation.php`。我们只是检查`param`变量是否存在。如果这个变量在`GET`请求中被引入，我们确认验证并将一个`OK`状态发送回页面：

```php
<?php
$result = array();
if(isset($_GET["param"])){
$result["status"] = "OK";
$result["message"] = "Input is valid!";
} else {
$result["status"] = "ERROR";
$result["message"] = "Input IS NOT valid!";
}
echo json_encode($result);
?>

```

## 如何做...

1.  让我们从基本的 HTML 结构开始。我们将定义一个带有三个输入框和一个文本区域的表单。当然，它是放在`<body>`中的：

```php
    <body>
    <h1>Validating form using Ajax</h1>
    <form class="simpleValidation">
    <div class="fieldRow">
    <label>Title *</label>
    <input type="text" id="title" name="title"
    class="required" />
    </div>
    <div class="fieldRow">
    <label>Url</label>
    <input type="text" id="url" name="url"
    value="http://" />
    </div>
    <div class="fieldRow">
    <label>Labels</label>
    <input type="text" id="labels" name="labels" />
    </div>
    <div class="fieldRow">
    <label>Text *</label>
    <textarea id="textarea" class="required"></textarea>
    </div>
    <div class="fieldRow">
    <input type="submit" id="formSubmitter" value="Submit" disabled="disabled" />
    </div>
    </form>
    </body>

    ```

1.  为了对有效输入进行视觉确认，我们将定义 CSS 样式：

```php
    <style>
    label{ width:70px; float:left; }
    form{ width:320px; }
    input, textarea{ width:200px;
    border:1px solid black; float:right; padding:5px; }
    input[type=submit] { cursor:pointer;
    background-color:green; color:#FFF; }
    input[disabled=disabled], input[disabled] {
    background-color:#d1d1d1; }
    fieldRow { margin:10px 10px; overflow:hidden; }
    failed { border: 1px solid red; }
    </style>

    ```

1.  现在，是时候包括 jQuery 及其功能了：

```php
    <script src="js/jquery-1.4.4.js"></script>
    <script>
    var ajaxValidation = function(object){
    var $this = $(object);
    var param = $this.attr('name');
    var value = $this.val();
    $.get("ajax/inputValidation.php",
    {'param':param, 'value':value }, function(data) {
    if(data.status=="OK") validateRequiredInputs();
    else
    $this.addClass('failed');
    },"json");
    }
    var validateRequiredInputs = function (){
    var numberOfMissingInputs = 0;
    $('.required').each(function(index){
    var $item = $(this);
    var itemValue = $item.val();
    if(itemValue.length) {
    $item.removeClass('failed');
    } else {
    $item.addClass('failed');
    numberOfMissingInputs++;
    }
    });
    var $submitButton = $('#formSubmitter');
    if(numberOfMissingInputs > 0){
    $submitButton.attr("disabled", true);
    } else {
    $submitButton.removeAttr('disabled');
    }
    }
    </script>

    ```

1.  我们还将初始化文档`ready`函数：

```php
    <script>
    $(document).ready(function(){
    var timerId = 0;
    $('.required').keyup(function() {
    clearTimeout (timerId);
    timerId = setTimeout(function(){
    ajaxValidation($(this));
    }, 200);
    });
    });
    </script>

    ```

1.  当一切准备就绪时，我们的结果如下：![如何做...](img/3081_02_01.jpg)

## 它是如何工作的...

我们创建了一个带有三个输入框和一个文本区域的简单表单。具有类`required`的对象在`keyup`事件后会自动进行验证，并调用`ajaxValidation`函数。我们的`keyup`功能还包括`Timeoutfunction`，以防止用户仍在输入时进行不必要的调用。验证基于两个步骤：

+   验证实际输入框：我们通过 Ajax 将插入的文本传递给`ajax/inputValidation.php`。如果服务器的响应不是`OK`，我们将标记此输入框为“失败”。如果响应是`OK`，我们将进行第二步。

+   检查我们表单中的其他必填字段。当表单中没有剩余的“失败”输入框时，我们将启用提交按钮。

## 还有更多...

在这个例子中，验证是非常基本的。我们只是检查服务器的响应状态是否为`OK`。我们可能永远不会遇到像我们这里一样的必填字段验证。在这种情况下，最好直接在客户端使用`length`属性，而不是用很多请求打扰服务器，只是为了检查必填字段是空还是填充了。这个任务只是基本`Validation`方法的演示。最好在服务器端扩展它，直接检查 URL 表单或标题是否已经存在于我们的数据库中，并让用户知道问题所在以及如何解决。

## 另请参阅

*在本章中使用 PHP Ajax 联系表单和验证*配方

# 创建一个自动建议控件

这个配方将向我们展示如何创建一个自动建议控件。当我们需要在大量数据中进行搜索时，这个功能非常有用。基本功能是根据输入框中的文本显示建议数据列表。

## 准备就绪

我们可以从虚拟的 PHP 页面开始，它将作为数据源。当我们用`GET`方法和变量`string`调用这个脚本时，它将返回包含所选字符串的记录（名称）列表：

```php
<?php
$string = $_GET["string"];
$arr = array(
"Adam",
"Eva",
"Milan",
"Rajesh",
"Roshan",
// ...
"Michael",
"Romeo"
);
function filter($var){
global $string;
if(!empty($string))
return strstr($var,$string);
}
$filteredArray = array_filter($arr, "filter");
$result = "";
foreach ($filteredArray as $key => $value){
$row = "<li>".str_replace($string,
"<strong>".$string."</strong>", $value)."</li>";
$result .= $row;
}
echo $result;
?>

```

## 如何做...

1.  和往常一样，我们将从 HTML 开始。我们将用一个输入框和一个未排序的列表`datalistPlaceHolder`来定义表单：

```php
    <h1>Dynamic Dropdown</h1>
    autosuggest controlcreating<form class="simpleValidation">
    <div class="fieldRow">
    <label>Skype name:</label>
    <div class="ajaxDropdownPlaceHolder">
    <input type="text" id="name" name="name"
    class="ajaxDropdown" autocomplete="OFF" />
    <ul class="datalistPlaceHolder"></ul>
    </div>
    </div>
    </form>

    ```

1.  当 HTML 准备好后，我们将使用 CSS 进行调整：

```php
    <style>
    label { width:80px; float:left; padding:4px; }
    form { width:320px; }
    input, textarea {
    width:200px; border:1px solid black;
    border-radius: 5px; float:right; padding:5px;
    }
    input[type=submit] { cursor:pointer;
    background-color:green; color:#FFF; }
    input[disabled=disabled] { background-color:#d1d1d1; }
    .fieldRow { margin:10px 10px; overflow:hidden; }
    .validationFailed { border: 1px solid red; }
    .validationPassed { border: 1px solid green; }
    .datalistPlaceHolder {
    width:200px; border:1px solid black;
    border-radius: 5px;
    float:right; padding:5px; display:none;
    }
    ul.datalistPlaceHolder li { list-style: none;
    cursor:pointer; padding:4px; }
    ul.datalistPlaceHolder li:hover { color:#FFF;
    background-color:#000; }
    </style>

    ```

1.  现在真正的乐趣开始了。我们将包括 jQuery 库并定义我们的 keyup 事件：

```php
    <script src="js/jquery-1.4.4.js"></script>
    autosuggest controlcreating<script>
    var timerId;
    var ajaxDropdownInit = function(){
    $('.ajaxDropdown').keyup(function() {
    var string = $(this).val();
    clearTimeout (timerId);
    timerId = setTimeout(function(){
    $.get("ajax/dropDownList.php",
    {'string':string}, function(data) {
    if(data)
    $('.datalistPlaceHolder').show().html(data);
    else
    $('.datalistPlaceHolder').hide();
    });
    }, 500 );
    });
    }
    </script>

    ```

1.  当一切准备就绪时，我们将在文档`ready`函数中调用`ajaxDropdownInit`函数：

```php
    <script>
    $(document).ready(function(){
    ajaxDropdownInit();
    });
    </script>

    ```

1.  我们的自动建议控件已经准备好了。以下截图显示了输出：![如何做...](img/3081_02_02.jpg)

## 它是如何工作的...

本教程中的`autosuggest`控件基于输入框和`datalistPlaceHolder`中的项目列表。在输入框的每个`keyup`事件之后，`datalistPlaceHolder`将通过`ajax/dropDownList.php`中定义的 Ajax 函数加载项目列表。本教程的一个很好的特性是`timerID`变量，当与`setTimeout`方法一起使用时，将允许我们仅在停止输入时向服务器发送请求（在我们的情况下是 500 毫秒）。这可能看起来并不那么重要，但它将节省大量资源。当我们已经输入“米兰”时，我们不想等待“M”在输入框中的响应。而不是 5 个请求（每个 150 毫秒），我们只有一个。例如，每天有 1 万个用户，效果是巨大的。

## 还有更多...

我们始终需要记住，服务器的响应是以 JSON 格式返回的。

```php
[{
'id':'1',
'contactName':'Milan'
},...,{
'id':'99',
'contactName':'Milan (office)'
}]

```

在 JavaScript 中使用 JSON 对象并不总是从性能的角度来看都有用。让我们想象一下，我们有一个 JSON 文件中有 5000 个联系人。

从 5000 个对象构建 HTML 可能需要一些时间，但是，如果我们构建一个 JSON 对象，代码将如下所示：

```php
[{
autosuggest controlcreating"status": "100",
"responseMessage": "Everything is ok! :)",
"data": "<li><h2><ahref=\"#1\">Milan</h2></li>
<li><h2><ahref=\"#2\">Milan2</h2></li>
<li><h2><ahref=\"#3\">Milan3</h2></li>"
}]

```

在这种情况下，我们将在 HTML 中拥有完整的数据，不需要创建任何逻辑来创建一个简单的项目列表。

# 制作表单向导

**表单向导**基本上是分成几个步骤的表单。它们对于投票或表单的特殊情况非常有用，当我们想要在网站上分割注册流程时。它们也用于电子商务网站，在购买过程中（购物车付款方式送货地址确认购买本身）。在这个教程中，我们将构建一个表单向导（尽可能简单）。

## 准备工作

我们将准备虚拟的 PHP 文件`step1.php, step2.php`和`step3.php`。这些文件的内容很简单：

```php
<?php
echo "STEP 1"; // Same for 2 and 3
?>

```

在这里，我们将包括 jQuery 库：

```php
<script src="js/jquery-1.4.4.js"></script>

```

## 如何做...

1.  我们首先定义 HTML 内容：

```php
    <div class="wizard">
    <ul class="wizardNavigation">
    <li class="active first" id="step1">Step 1</li>
    <li id="step2">Step 2</li>
    <li id="step3" class="last">Step 3</li>
    </ul>
    <div class="wizardBody">STEP 1</div>
    <div class="wizardActionButtons">
    <a href="javascript:submitThePage('back');" class="back"
    style="display:none;">Back</a>
    <a href="http:// class="finish" style="display:none;"> Finish</a>
    <a href="javascript:submitThePage('next');"
    class="next">Next</a>
    </div>
    </div>

    ```

1.  接下来，我们将在 HTML 中包含 CSS 样式如下：

```php
    <style>
    .wizard { width:300px; overflow:hidden;
    border:1px solid black; }
    .wizardNavigation { overflow:hidden;
    border-bottom:1px solid #D2D2D2; }
    .wizardNavigation li { float:left; list-style:none;
    padding:10px; cursor:default; color:#D2D2D2; }
    .wizardNavigation li.active { color:#000; }
    .wizardBody { clear:both; padding:20px; }
    .wizardActionButtons { padding:10px;
    border-top:1px solid #D2D2D2; }
    .wizardActionButtons .back { float:left; cursor:pointer; }
    .wizardActionButtons .next,
    .wizardActionButtons .finish { float:right; cursor:pointer; }
    .wizard .disabled { color:#D2D2D2; }
    </style>

    ```

1.  接下来，我们将在关闭`</body>`标签之前放置 JavaScript：

```php
    <script>
    var submitThePage = function (buttonDirection){
    var $currentTab = $('.wizardNavigation li.active');
    if(buttonDirection == 'next')
    var $actionTab = $currentTab.next('li');
    else
    var $actionTab = $currentTab.prev('li');
    var target = "ajax/"+ $actionTab.attr('id') +".php";
    $.get(target, {'param':'test'},
    function(data) {
    if(data){
    if($actionTab){
    $currentTab.removeClass('active');
    $actionTab.addClass('active');
    }
    displayFinishButton($actionTab.hasClass('last'));
    displayNextButton(!$actionTab.hasClass('last'));
    displayBackButton(!$actionTab.hasClass('first'));
    $('.wizardBody').html(data);
    }
    });
    }
    var displayBackButton = function(enabled){
    enabled == true ?
    $('.back').show() : $('.back').hide();
    }
    var displayNextButton = function(enabled){
    enabled == true ?
    $('.next').show() : $('.next').hide();
    }
    var displayFinishButton = function(enabled){
    enabled == true ?
    $('.finish').show() : $('.finish').hide();
    }
    </script>

    ```

1.  结果如下：![如何做...](img/3081_02_03.jpg)

## 它是如何工作的...

向导分为三个部分：

+   第一部分是`wizardNavigation`，其中包括向导中的所有步骤（选项卡）。

+   第二个是`wizardBody`，其中包含当前步骤（选项卡）的内容。

+   最后一部分是`wizardActionButtons`，其中包含**返回**、**下一步**和**完成**按钮。**返回**和**下一步**按钮触发`submitThePage`函数，带有`buttonDirection`参数（**返回**或**下一步**）。此函数将通过`$.get()`函数将 Ajax 请求发送到下一步，该下一步由`target`参数表示。目标自动从选项卡导航中获取。它等于每个导航元素的`id`属性。

## 还有更多...

我们已经理解了表单向导的基本思想。但有时我们没有时间或资源来创建自己的 jQuery 功能。在这种情况下，我们可以使用一些免费的 jQuery 插件，比如来自[`plugins.jquery.com/project/formwizard`](http://plugins.jquery.com/project/formwizard)的`formwizard`插件。并非所有插件都是 100%功能的；每样东西都有自己的“bug”。然而，帮助总是很容易得到的。我们可以修改插件以满足我们的要求，然后等待插件的下一个版本中修复 bug，或者我们可以贡献自己的代码。

# 使用 Ajax 上传文件

在这个示例中，我们将讨论通过 Ajax 上传文件。实际上，没有 Ajax 方法可以做到这一点。我们可以使用`iframe`方法来模拟 Ajax 功能。

## 准备工作

首先，我们将准备`uploads`文件夹，并确保它是可访问的。在 Mac OS X/Linux 中，我们将使用：

```php
$ sudo chmod 777 'uploads/'

```

### 注意

在 Windows 7 中，我们可以右键单击**文件夹属性|编辑|选择用户|组**，从权限窗口中选择（选择任何人），并在**允许**列下选择**完全控制**以分配完全访问权限控制权限。

现在让我们创建一个 HTML 文件`(ajaxUpload.html)`和一个 PHP 文件`(ajax/uploadFile.php)`。

## 如何做...

1.  `ajaxUpload.html`将如下所示：

```php
    <script>
    function submitForm(upload_field){
    upload_field.form.submit();
    upload_field.disabled = true;
    return true;
    }
    </script>

    ```

1.  我们的 HTML 主体如下：

```php
    <h1>Uploading File Using Ajax</h1>
    <form action="ajax/uploadFileSingle.php" target="uploadIframe"
    method="post" enctype="multipart/form-data">
    <div class="fieldRow">
    <label>Select the file: </label>
    <input type="file" name="file" id="file"
    onChange="submitForm(this)" />
    </div>
    </form>
    <iframe id="uploadIframe" name="uploadIframe"></iframe>
    <div id="placeHolder"></div>

    ```

ajax/uploadFile.php 的内容如下：

```php
    <head>
    <script src="../js/jquery-1.4.4.js"></script>
    </head>
    <body>
    <?php
    $upload_dir = "../uploads";
    $result["status"] = "200";
    $result["message"]= "Error!";
    if(isset($_FILES['file'])){
    echo "Uploading file... <br />";
    if ($_FILES['file']['error'] == UPLOAD_ERR_OK) {
    $filename = $_FILES['file']['name'];
    move_uploaded_file($_FILES['file']['tmp_name'],
    $upload_dir.'/'.$filename);
    $result["status"] = "100";
    $result["message"]=
    "File was uploaded successfully!";
    } elseif ($_FILES['file']['error'] ==
    UPLOAD_ERR_INI_SIZE) {
    $result["status"] = "200";
    $result["message"]= "The file is too big!";
    } else {
    $result["status"] = "500";
    $result["message"]= "Unknown error!";
    }
    }
    ?>
    </body>

    ```

1.  在`$(document).ready`上初始化结果消息：

```php
    <script>
    $(document).ready(function(){
    $('#placeHolder', window.parent.document)
    .html('<?php echo htmlspecialchars($result["message"]); ?>');
    });
    </script>

    ```

1.  结果如下：![如何做...](img/3081_02_04.jpg)

## 它是如何工作的...

正如您在这个任务中所看到的，我们创建了一个简单的表单，可以上传文件。这个例子的主要重点在于**iframe**，我们将表单提交到这个**iframe**。这个**iframe**代表一个包含 PHP 的容器，它提供所选文件的物理上传。当上传成功时，我们将在父文档的`placeHolder`中显示结果消息。

## 还有更多...

要增加上传文件的最大允许大小，我们可以在`php.ini`中使用`upload_max_filesize`指令。还有更多用于上传文件的指令：

| 指令 | 默认值 |   |
| --- | --- | --- |
| `file_uploads` | `1` | 允许/禁止 HTTP 文件上传 |
| `upload_tmp_dir` | `NULL` | 文件上传期间存储文件的临时目录 |
| `upload_max_filesize` | `2M` | 上传文件的最大大小 |
| `max_file_uploads` | `20` | 同时进行的文件上传的最大数量 |

# 使用 Ajax 上传多个文件

在上一个任务中，我们学习了如何通过伪 Ajax 方法使用 iframe 上传单个文件。这个例子有一个很大的缺点；我们无法选择多个文件。这只能通过使用 HTML5（并非所有浏览器都完全支持）、Flash 或 Java 来实现。在这个示例中，我们将构建一个表单，允许我们选择多个文件，并在单击一次后将它们上传到服务器。

## 准备工作

对于这个任务，我们需要下载 jQuery 库、SWFUpload 库([`swfupload.org/`](http://swfupload.org/))，以及 Adam Royle 的 SWFUpload jQuery 插件([`blogs.bigfish.tv/adam/`](http://blogs.bigfish.tv/adam/))。

## 如何做...

1.  让我们从 HTML 开始：

```php
    <div id="swfupload-control">
    <p>Upload files.</p>
    <input type="button" id="button" value="Upload" />
    <p id="queuestatus"></p>
    <ol id="log"></ol>
    </div>

    ```

1.  接下来，我们定义 CSS：

```php
    <style>
    #swfupload-control p { margin:10px 5px; }
    #log li { list-style:none; margin:2px; padding:10px;
    font-size:12px; color:#333; background:#fff;
    position:relative; border:1px solid black;
    border-radius: 5px;}
    #log li .progressbar { height:5px; background:#fff; }
    #log li .progress { background:#999; width:0%; height:5px; }
    #log li p { margin:0; line-height:18px; }
    #log li.success { border:1px solid #339933;
    background:#ccf9b9;}
    </style>

    ```

1.  现在，我们将包括 jQuery、`SWFUpload`和 SWFUpload jQuery 库：

```php
    <script src="js/jquery-1.4.4.js"></script>
    <script src="js/swfupload/swfupload.js"></script>
    <script src="js/jquery.swfupload.js"></script>

    ```

1.  接下来，我们将定义`SWFUpload`对象和绑定事件，如下所示：

```php
    <script>
    $(function(){
    $('#swfupload-control').swfupload({
    upload_url: "upload-file.php",
    file_post_name: 'uploadfile',
    flash_url : "js/swfupload/swfupload.swf",
    button_image_url :
    'js/swfupload/wdp_buttons_upload_114x29.png',
    button_width : 114,
    button_height : 29,
    button_placeholder : $('#button')[0],
    debug: false
    })
    .bind('fileQueued', function(event, file){
    var listitem='<li id="'+file.id+'" >'+
    file.name+' ('+Math.round(file.size/1024)+' KB)
    <span class="progressvalue" ></span>'+
    '<div class="progressbar" >
    <div class="progress" ></div></div>'+
    '<p class="status" >Pending</p>'+'</li>';
    $('#log').append(listitem);
    $(this).swfupload('startUpload');
    })
    .bind('uploadStart', function(event, file){
    $('#log li#'+file.id)
    .find('p.status').text('Uploading...');
    $('#log li#'+file.id)
    .find('span.progressvalue').text('0%');
    })
    .bind('uploadProgress', function(event, file, bytesLoaded){
    var percentage=Math.round((bytesLoaded/file.size)*100);
    $('#log li#'+file.id)
    .find('div.progress').css('width', percentage+'%');
    $('#log li#'+file.id)
    .find('span.progressvalue').text(percentage+'%');
    })
    .bind('uploadSuccess', function(event, file, serverData){
    var item=$('#log li#'+file.id);
    item.find('div.progress').css('width', '100%');
    item.find('span.progressvalue').text('100%');
    item.addClass('success').find('p.status')
    .html('File was uploaded successfully.');
    })
    .bind('uploadComplete', function(event, file){
    $(this).swfupload('startUpload');
    })
    });
    </script>

    ```

1.  用于上传文件的 PHP 如下：

```php
    <?php
    $uploaddir = './uploads/';
    $file = $uploaddir . basename($_FILES['uploadfile']['name']);
    if (move_uploaded_file($_FILES['uploadfile']['tmp_name'], $file)) { echo "success"; } else { echo "error"; }
    ?>

    ```

1.  我们的结果如下：![如何做...](img/3081_02_05.jpg)

## 它是如何工作的...

首先，我们为`swfupload-control`定义一个简单的 HTML 表单，包括输入按钮。这个按钮被一个`swf`对象覆盖，它允许我们选择多个文件。在 JavaScript 中，我们使用基本设置`(upload_url、file_post_name、flash_url、button_image_url`等)定义主`SWFUpload`对象。我们可以使用预定义的事件为每个文件构建一个带有进度条的容器。

## 还有更多...

在`SWFUpload`中定义的事件为我们提供了在文件上传期间完全控制的能力，如下所示：

| **flashReady** | 这是由 Flash 控件调用的，通知`SWFUpload`Flash 电影已加载。 |
| --- | --- |
| **swfUploadLoaded** | 这是为了确保可以安全调用`SWFUpload`方法而调用的。 |
| **fileDialogStart** | 在调用`selectFile`选择文件后触发此事件。 |
| **fileQueued** | 在**FileSelectionDialog**窗口关闭后，每个排队的文件都会触发此事件。 |
| **fileQueueError** | 在**FileSelectionDialog**窗口关闭后，每个未排队的文件都会触发此事件。 |
| **fileDialogComplete** | 这在**FileSelectionDialog**窗口关闭并且所有选定的文件都已处理后触发。 |
| **uploadStart** | 这是在文件上传之前立即调用的。 |
| **uploadProgress** | 这是由 Flash 控件定期触发的。 |
| **uploadError** | 每当上传被中断或未成功完成时触发。 |
| **uploadSuccess** | 当整个上传已传输并服务器返回 HTTP 200 状态代码时触发。 |
| **uploadComplete** | 这总是在上传周期结束时触发（在`uploadError`或`uploadSuccess`之后）。 |

# 创建一个五星级评分系统

在这个任务中，我们将学习如何构建一个五星级评分系统。这个功能经常被电子商务网站使用，允许用户对产品、文章或任何值得用户评价的东西进行评分。

## 准备工作

让我们准备一个虚拟的 PHP 文件`ajax/saveRating.php`来确认评分已保存：

```php
<?php
$result = array();
$result["status"] = "";
$result["message"] = "";
if(isset($_POST["itemID"]) && isset($_POST["itemValue"])){
$result["status"] = "OK";
$result["message"] = "Rating has been saved successfully.";
} else {
$result["status"] = "ERROR";
$result["message"] = "Provide itemID and itemValue!";
}
echo json_encode($result);
?>

```

我们需要准备一个带有星星的.gif 图像。这个.gif 包括星星的三种变化：第一种是非活动状态的星星，第二种是“悬停”事件的星星，第三种是活动状态的星星。

![准备工作](img/3081_02_06.jpg)

## 如何做...

1.  我们准备开始 HTML 部分：

```php
    <body>
    five star rating systemcreating, steps<h1>Creating Five Stars Rating System</h1>
    <div class="fieldRow">
    <label>Book 123A</label>
    <ul id="book-123a" class="ratingStars">
    <li></li>
    <li class="active"></li>
    <li></li>
    <li></li>
    <li></li>
    </ul>
    </div>
    <div class="fieldRow">
    <label>Book 123B</label>
    <ul id="book-123b" class="ratingStars">
    <li class="active"></li>
    <li></li>
    <li></li>
    <li></li>
    <li></li>
    </ul>
    </div>
    <div class="fieldRow">
    <label>Book 123C</label>
    <ul id="book-123c" class="ratingStars">
    <li></li>
    <li></li>
    <li></li>
    <li></li>
    <li class="active"></li>
    </ul>
    </div>
    <div id="placeHolder"></div>
    </body>

    ```

1.  让我们包括 jQuery 库并定义 JavaScript 功能：

```php
    <script src="js/jquery-1.4.4.js"></script>
    <script>
    $(document).ready(function(){
    $('ul.ratingStars li.active').prevAll().addClass('active');
    $('ul.ratingStars li').each(function(){
    var $item = $(this);
    var $itemContainer = $item.parents('ul.ratingStars');
    var containerID = $itemContainer.attr('id');
    var $itemsAll = $itemContainer.find('li');
    $item.mouseover(function(){
    $itemsAll.addClass('default');
    $item.prevAll().addClass('highlighted');
    })
    .mouseout(function(){
    $itemsAll
    .removeClass('default')
    .removeClass('highlighted');
    });
    .bind('click', function(){
    var itemIndex = $itemsAll.index(this);
    $.post('ajax/saveRating.php',
    {'itemID':containerID, 'itemValue': itemIndex},
    function(data) {
    if(data && data.status == "100"){
    $item
    .addClass('active')
    .removeClass('highlighted');
    $item.nextAll().removeClass('active');
    $item.prevAll().addClass('active');
    } else {
    alert('Error!');
    }
    }, "json");
    });
    });
    });
    </script>

    ```

1.  CSS 是这项任务中的关键部分之一：

```php
    <style>
    five star rating systemcreating, stepslabel, ul { float:left; }
    .fieldRow { clear:both; margin:5px 0px; overflow:hidden; }
    ul.ratingStars { list-style:none; margin:0px 0px;
    overflow:hidden; }
    ul.ratingStars li { float:left; width:16px; height:16px;
    background:url('icons/star.gif') no-repeat left top;
    cursor:pointer; }
    ul.ratingStars li.active { background-position: 0px -32px; }
    ul.ratingStars li.default { background-position: 0px 0px; }
    ul.ratingStars li.highlighted,
    ul.ratingStars li:hover { background-position: 0px -16px; }

    ```

1.  我们的结果如下：![如何做...](img/3081_02_07.jpg)

## 它是如何工作的...

基本上，整个评分系统是一个项目的无序列表。每个项目代表一个星星，可以提供三种状态；默认、活动或突出显示。通过改变每颗星星的背景位置来改变状态。在我们的情况下，我们使用`icons/star.gif`，其中包括所有三种可能的状态（灰色、红色和黄色）。定义了一个`mouseover`事件，它将突出显示悬停的星星和之前选择的所有星星。点击星星后，我们调用一个 Ajax post 请求到`ajax/saveRating.php`并设置所有必需的星星为激活状态。

## 还有更多...

在大多数情况下，我们不希望允许一个用户进行多次投票。在这种情况下，我们可以设置 cookie 如下：

```php
...
if(isset($_POST["itemID"]) && isset($_POST["itemValue"])){
setcookie("rated".$id, $id, time()+60*60*60*24*365);
$result["status"] = "OK";
$result["message"] = "Rating has been saved successfully.";
}

```

当 cookie 设置为一年后过期时，我们可以在我们的评分系统中使用它：

```php
if(isset($_COOKIE['rated'.$id])) {
$result["status"] = "550";
$result["message"] = "Already voted!";
}
echo json_encode($result);

```

# 使用验证构建 PHP Ajax 联系表单

在提交表单之前对输入框进行验证已成为非常重要的 Ajax 功能之一。用户不必等到整个表单返回一些无效的输入框消息，然后再尝试重新填写。在这个任务中，我们将构建一个带有 Ajax 验证的联系表单。

## 如何做...

1.  让我们从 HTML 开始：

```php
    <body>
    <form id="contactForm" action="#" method="post">
    <h1>PHP Ajax Contact Form</h1>
    <div class="fieldRow">
    <label for="name">Your Name:</label>
    <input type="text" name="name" id="name"
    class="required" />
    </div>
    <div class="fieldRow">
    <label for="email">Your e-mail:</label>
    <input type="text" name="email" id="email"
    class="required email" />
    </div>
    <div class="fieldRow">
    <label for="url">Website:</label>
    <input type="text" name="url" id="url"
    class="url" />
    </div>
    <div class="fieldRow">
    <label for="phone">Mobile Phone:</label>
    <input type="text" name="phone" id="phone"
    class="phone"/>
    </div>
    <div class="fieldRow">
    <label for="message">Your Message:</label>
    <textarea name="message" id="message"
    class="required"></textarea>
    </div>
    <div class="fieldRow buttons">
    <input type="reset" value="Clear" />
    <input type="submit" value="Send" />
    </div>
    </form>
    </body>

    ```

1.  现在，我们可以定义样式：

```php
    <style>
    form{ width:450px; }
    label { float:left; width:100px; padding:5px; }
    .fieldRow { overflow:hidden; margin:20px 10px; }
    .buttons { text-align:center; }
    input[type=text], textarea{ width:200px; border:1px solid black; border-radius: 5px; padding:5px; }
    input.required, textarea.required{ border:1px solid orange; }
    .failed input.required, .failed textarea.required{ border:1px solid red; }
    .failed { color:red; }
    </style>

    ```

1.  主要的 PHP 验证如下：

```php
    "status" => "50500",
    "message" => "Error: No parameters provided."
    );
    if(isset($_POST["param"])){
    $param = $_POST["param"];
    $value = $_POST["value"];
    switch($param){
    default:
    $result["status"] = "10100";
    $result["message"] = "OK";
    break;
    case 'email':
    if(filter_var($value, FILTER_VALIDATE_EMAIL)){
    $result["status"] = "10100";
    $result["message"] = "E-mail is valid!";
    } else {
    $result["status"] = "50502";
    $result["message"] = "Error: E-mail is not
    valid.";
    }
    break;
    case 'url':
    if(filter_var($value, FILTER_VALIDATE_URL)){
    $result["status"] = "10100";
    $result["message"] = "URL is valid!";
    } else {
    $result["status"] = "50502";
    $result["message"] = "Error: URL is not valid.";
    }
    break;
    case 'phone':
    if(preg_match('/^\+?[0-9]+$/', $value)){
    $result["status"] = "10100";
    $result["message"] = "Phone is valid!";
    } else {
    $result["status"] = "50502";
    $result["message"] = "Error: Phone number is not
    valid.";
    }
    break;
    }
    }
    echo json_encode($result);
    ?>

    ```

1.  具有 Ajax 调用的 JavaScript 功能如下：

```php
    <script src="js/jquery-1.4.4.js"></script>
    <script>
    $(document).ready(function(){
    $('#contactForm').submit(function(e){
    var $form = $(this);
    $.ajaxSetup({async:false});
    $('.required').each(function(){
    var $this = $(this);
    var value = $this.val();
    if(value.length == 0)
    $this.parents('.fieldRow').addClass('failed');
    else
    $this.parents('.fieldRow').removeClass('failed');
    });
    $('.email').each(function(){
    var $this = $(this);
    var value = $this.val();
    $.post("validators/main.php",
    {'param':'email', 'value':value },
    function(data) {
    if(data.status==10100)
    $this
    .parents('.fieldRow').removeClass('failed');
    else
    $this.parents('.fieldRow').addClass('failed');
    }, "json");
    });
    $('.url').each(function(){
    ...
    $.post("validators/main.php",
    {'param':'url', 'value':value }, function(data) {
    ...
    });
    $('.phone').each(function(){
    ...
    $.post("validators/main.php",
    {'param':'phone', 'value':value }, function(data) {
    ...
    });
    return !$('.failed').length;
    });
    });
    </script>

    ```

1.  结果如下：![如何做...](img/3081_02_08.jpg)

## 它是如何工作的...

我们从 HTML 源代码开始。它包含四个输入框和一个文本区域。正如你在源代码中所看到的，我们准备了两种验证。第一种是检查必填字段（标有`class="required"`），第二种是基于特定类型的数据（电子邮件、URL 和电话）。第一种验证仅在客户端进行，第二种涉及发送一个 post 请求到`validators/main.php`，对给定的参数进行评估。如果输入未通过验证，则标记为`failed`。如果表单中没有`failed`输入框，则启用`submit`事件。当所有请求完成时，该事件返回`true`值。这是通过允许同步请求来实现的——`$.ajaxSetup({async:false})`。请注意，同步请求可能会暂时锁定浏览器，禁用任何活动，而请求处于活动状态时。

## 还有更多...

在这个例子中，我们使用了服务器端字段的验证。这个逻辑并不完全是我们在现实生活中会使用的。当然，我们应该始终在服务器端进行验证(以防用户关闭了 JavaScript)，但我们不需要让服务器处理一些我们可以在客户端轻松找到的东西，比如验证电子邮件、URL 或必填字段。jQuery 有一个名为`validate.js`的很好的验证插件([`docs.jquery.com/Plugins/validation`](http://docs.jquery.com/Plugins/validation))。

我们需要做的就是下载带有`validate`插件的 jQuery 库，并将其包含在我们的源代码中：

```php
<script src="jquery/jquery-latest.js"></script>
<script src="jquery/plugins/validate/jquery.validate.js">
</script>

```

为必填字段定义`required`类，并为特定验证器定义一些额外的类，比如电子邮件：

```php
<form id="commentForm" method="get" action="">
<label for="email">E-Mail</label>
<input id="email" name="email" class="required email" />
</form>

```

之后，在特定表单中调用`validate()`函数：

```php
<script>
$(document).ready(function(){$("#commentForm").validate();});
</script>

```

# 在 Ajax 中显示表格

在这个任务中，我们将使用 Ajax 以表格形式显示数据。作为数据源，我们将使用预定义的 JSON 对象。

## 准备工作

首先，我们需要准备一个虚拟的 JSON 数据源，其中包含我们表格中将显示的所有项目：

```php
json/requests.json:
[{
"id": "1",
"name": "Milan Sedliak Group",
"workflow": "Front End Q Evaluation",
"user": "Milan Sedliak",
"requestor": "Milan Sedliak",
"status": "submitted",
"email": "milan@milansedliak.com",
"date": "Today 15:30"
},
{...}]

```

## 如何做...

1.  作为基本 HTML，我们可以使用这个包含表格和工具栏的源代码。这个工具栏将包括我们项目的选择功能。

```php
    <div class="tableContainer">
    <div class="tableToolbar">
    Select
    <a href="#" class="selectAll">All</a>,
    <a href="#" class="selectNone">None</a>,
    <a href="#" class="selectInverse">Inverse</a>
    </div>
    <table>
    <thead></thead>
    <tbody></tbody>
    </table>
    </div>

    ```

1.  现在，我们可以为我们的 HTML 设置样式：

```php
    <style>
    .tableContainer { width:900px; }
    .tableToolbar { background-color:#EEFFEE; height:20px;
    padding:5px; }
    table { border-collapse: collapse; width:100%; }
    table th { background-color:#AAFFAA; padding:4px; }
    table tr td { padding:4px; }
    table tr.odd td { background-color:#E3E3E3; }
    .floatr { float:right; }
    .textAlignR { text-align: right; }
    </style>

    ```

1.  当 HTML 和 CSS 准备好后，我们可以开始使用 JavaScript：

```php
    <script src="js/jquery-1.4.4.js"></script>
    <script>
    $(document).ready(function(){
    $.getJSON('json/requests.json', function(data) {
    buildHeader(data);
    buildBody(data);
    });
    });
    var buildHeader = function(data){
    var keys = [];
    var $headRow = $('<tr />');
    for(var key in data[0]){
    if(key=="id")
    var $cell = $('<th />');
    else
    var $cell = $('<th>'+key+'</th>');
    $cell.appendTo($headRow);
    }
    $headRow.appendTo($('.tableContainer table thead'));
    }
    var buildBody = function(data){
    for(var i = 0; i < data.length; i++){
    var dataRow = data[i];
    var $tableRow = $('<tr />');
    for(var key in dataRow){
    var $cell = $('<td />');
    switch(key){
    default:
    $cell.html(dataRow[key]);
    break;
    case 'id':
    var $checkbox = $('<input type="checkbox"
    name="select['+dataRow[key]+']" />');
    $checkbox.appendTo($cell);
    break;
    case 'date':
    $cell.html(dataRow[key]);
    $cell.addClass('textAlignR');
    break;
    }
    $cell.appendTo($tableRow);
    }
    if(i % 2 == 0)
    $tableRow.addClass('odd');
    $tableRow.appendTo($('.tableContainer table tbody'));
    }
    }
    </script>

    ```

1.  结果如下：![如何做...](img/3081_02_09.jpg)

## 它是如何工作的...

起初，我们为表格制定了基本的 HTML 结构。我们只定义了表头和表体的位置。在`(document).event`中，我们发送一个`getJSON`请求，从服务器获取一个`json`对象(`json/requests.json`)。我们将数据放入`data`变量中，并继续构建表格。在第一步中，我们构建表头(buildHeader(data))。这个函数获取数据，从 JSON 对象中解析键，并将它们用于表头单元格。在第二步中，我们构建表体(buildBody(data))。这个函数基于一个循环，将指定表格的每一行。我们使用一个开关，它能够根据键提供每个值的特定功能。

## 还有更多...

在这个任务中，我们已经构建了一个带有工具箱的表格，但它还没有任何功能；至少目前还没有。在每一行中，我们定义了一个复选框。通过定义这个复选框，我们可以指定额外的功能：

```php
$checkbox.bind('click', function(e){
$(this).parents('tr').addClass('highlighted');
})
.appendTo($cell);

```

对于前面代码片段中提到的工具栏，我们可以指定：

```php
$('.selectAll').bind('click',function(){
$(this) .parents('table')
.find('input[type="checkbox"]').each(function(){
var $checkbox = $(this);
$checkbox.attr('checked', 'checked');
var $row = $checkbox.parents('tr');
$row.addClass('highlighted');
});
});

```

# 使用 PHP 和 Ajax 构建分页

在这个任务中，我们将学习如何使用 Ajax 功能构建**分页**。这意味着我们将能够在不重新加载整个网站的情况下翻页查看联系人列表。

## 如何做...

1.  我们将从包含页面容器、显示联系人第一页的联系人网格和联系人分页的 HTML 开始：

```php
    <div id="pageContainer">
    <div id="contactGrid">
    <div class="contact">
    <img src="images/avatar.png" alt="avatar" />
    <h2>Milan Sedliak</h2>
    <p>Prague, Czech Republic</p>
    </div>
    <!-- // add more contacts -->
    <div class="contact">
    <img src="images/avatar.png" alt="avatar" />
    <h2>Milan Sedliak (home)</h2>
    <p>Malacky, Slovakia</p>
    </div>
    </div>
    <ul id="contactPagination">
    <li><a href="#previous" id="previous">Previous</a></l
    <li><a href="#1" id="1">1</a></li>
    <li><a href="#2" id="2">2</a></li>
    <li><a href="#3" id="3">3</a></li>
    <li><a href="#4" id="4">4</a></li>
    <li><a href="#5" id="5" class="active">5</a></li>
    <li><a href="#6" id="6">6</a></li>
    <li><a href="#7" id="7">7</a></li>
    <li><a href="#next" id="next">Next</a></li>
    </ul>
    </div>

    ```

1.  所需的 CSS 如下：

```php
    <style>
    #pageContainer { width: 410px; margin:0px auto; }
    #contactGrid { width: 410px; margin:10px auto;
    overflow:hidden; position:relative; }
    #contactGrid .contact { float:left; width:200px;
    margin:10px 0px; }
    .contact img { float:left; margin-right:5px;
    margin-bottom:10px; }
    .contact h2 { font-size:14px; }
    .contact p { font-size: 12px; }
    #contactPagination { clear:both; margin-left:50px; }
    #contactPagination li { float:left; list-style:none; }
    #contactPagination li a { padding:5px; margin:5px;
    border:1px solid blue; text-decoration:none; }
    #contactPagination li:hover a { color:orange;
    border:1px solid orange; }
    #contactPagination li a.active { color:black;
    border:1px solid black; }
    </style>

    ```

1.  分页的 JavaScript 功能如下：

```php
    <script src="js/jquery-1.4.4.js"></script>
    <script>
    $(document).ready(function(){
    paginationInit();
    });
    var paginationInit = function(){
    $('#contactPagination li a').bind('click', function(e){
    e.preventDefault();
    var $this = $(this);
    var target = $this.attr('id');
    var $currentItem = $('#contactPagination a.active')
    .parents('li');
    var $contactGrid = $('#contactGrid');
    switch(target){
    default:
    $('#contactPagination a').removeClass('active');
    $this.addClass('active');
    var page = target;
    $.get('contacts.php',
    {'page': page}, function(data) {
    $contactGrid.html(data);
    });
    break;
    case 'next':
    var $nextItem = $currentItem.next('li');
    $('#contactPagination a').removeClass('active');
    var $pageToActive = $nextItem.find('a')
    .addClass('active');
    var page = $pageToActive.attr('id');
    $.get('contacts.php',
    {'page': page}, function(data) {
    $contactGrid.html(data);
    });
    break;
    case 'previous':
    var $previousItem = $currentItem.prev('li');
    $('#contactPagination a').removeClass('active');
    var $pageToActive = $previousItem.find('a')
    .addClass('active');
    var page = $pageToActive.attr('id');
    $.get('contacts.php',
    {'page': page}, function(data) {
    $contactGrid.html(data);
    });
    break;
    }
    hidePreviousNextButtons();
    });
    }
    var hidePreviousNextButtons = function(){
    var $currentItem = $('#contactPagination a.active');
    var currentItemID = $currentItem.attr('id');
    var $nextButton = $('#contactPagination #next');
    var $previousButton = $('#contactPagination #previous');
    var lastItemID = $nextButton.parents('li').prev('li')
    .find('a').attr('id');
    var firstItemID = $previousButton.parents('li').next('li')
    .find('a').attr('id');
    currentItemID == lastItemID ?
    $nextButton.hide() : $nextButton.show();
    currentItemID == firstItemID ?
    $previousButton.hide() : $previousButton.show();
    }
    </script>

    ```

1.  要检索所需的页面，我们将定义`contact.php:`

```php
    <?php
    if (isset($_GET["page"])) { $page = (int)$_GET["page"]; } else { $page = 1; };
    $start_from = ($page-1) * 20;
    $sql = "SELECT * FROM contacts ORDER BY name ASC LIMIT $start_from, 20";
    $result = mysql_query ($sql,$connection);
    ?>
    <?php
    $result="";
    while ($row = mysql_fetch_assoc($result)) {
    $avatar = htmlspecialchars($row["avatar"]); $fullName = htmlspecialchars($row["fullName"]);
    $address = htmlspecialchars($row["address"]);
    $result .= sprintf('
    <div class="contact">
    <img src="%s" alt="avatar" />
    <h2>%s</h2>
    <p>%s</p>
    </div>',$avatar,$fullName,$address);
    };

    ```

1.  结果如下：![如何做...](img/3081_02_10.jpg)

## 它是如何工作的...

我们在`paginationInit()`函数中定义了分页的主要功能。主要步骤是获取分页中的每个超链接，并根据其`id`属性分配特定的功能。当`id`为`next`或`previous`时，这意味着我们点击了**下一页**或**上一页**按钮。在这种情况下，我们查找当前活动的页面，并选择`next/previous`超链接。如果我们已经到达了`first/last`超链接，我们通过调用`hidePreviousNextButtons()`函数隐藏`previous/next`按钮。在这个例子中，默认目标是数字项(页面)之一。当我们点击时，我们保存当前活动页面，从`contacts.php`调用`GET`请求以获取所需的页面，并在`contactGrid`中显示它。

## 还有更多...

我们学会了如何构建基本的分页。现在我们可以玩一下用户体验。我们的用户喜欢看到页面上发生了什么。在这种情况下，我们点击代表页面的链接，并等待联系人在联系人网格中显示出来。现在，我们可以为我们的用户提供一个经典的旋转器，作为内容正在加载的通知。

首先，我们需要找到一个`.gif`图像作为旋转器。我们可以在互联网上很容易找到一个。当图像准备好并保存在我们的图像文件夹中时，我们可以定义 CSS 如下：

```php
#spinnerContainer { opacity:0.85; position:absolute;
width:100%; height:100%;
background:url('images/loader-big.gif') no-repeat
center center #000; }

```

我们可以直接将旋转器的显示添加到现有的函数中；这可以在 Ajax 请求之前完成，当请求`id`完成时。我们将使用`.html()`函数覆盖 HTML 内容。

```php
$('<div id="spinnerContainer"></div>')
.prependTo($contactGrid);
$.get('contacts.php',
{'page': page}, function(data) {
$contactGrid.html(data);
});

```

修改后的版本如下：

![还有更多...](img/3081_02_11.jpg)
