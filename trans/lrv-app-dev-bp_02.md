# 第二章。使用 Ajax 构建待办事项列表

在本章中，我们将使用 Laravel PHP 框架和 jQuery 来构建一个带有 Ajax 的待办事项列表。

在本章中，我们将向您展示**RESTful 控制器**、**RESTful 路由**和**请求类型**的基础知识。本章涵盖的主题列表如下：

+   创建和迁移待办事项列表的数据库

+   创建待办事项列表的模型

+   创建模板

+   使用 Ajax 向数据库插入数据

+   从数据库中检索列表

+   如何只允许 Ajax 请求

# 创建和迁移待办事项列表的数据库

正如您从上一章所知，迁移对于控制开发步骤非常有帮助。我们将在本章再次使用迁移。

要创建我们的第一个迁移，请输入以下命令：

```php
**php artisan migrate:make create_todos_table --table=todos --create**

```

当您运行此命令时，**Artisan**将生成一个迁移以生成名为`todos`的数据库表。

现在我们应该编辑迁移文件以创建必要的数据库表列。当您用文件管理器打开`app/database/`中的`migration`文件夹时，您会看到其中的迁移文件。

让我们按照以下方式打开并编辑文件：

```php
<?php
use Illuminate\Database\Migrations\Migration;
class CreateTodosTable extends Migration {

    /**
    * Run the migrations.
    *
    * @return void
    */
    public function up()
    {

        Schema::create('todos', function(Blueprint $table){
            $table->create();
            $table->increments("id");
            $table->string("title", 255);
            $table->enum('status', array('0', '1'))->default('0');
            $table->timestamps();
        });

    }

    /**
    * Reverse the migrations.
    *
    * @return void
    */
    public function down()
    {
        Schema::drop("todos");
    }

}
```

要构建一个简单的待办事项列表，我们需要五列：

+   `id`列将存储待办任务的 ID 编号

+   `title`列将存储待办任务的标题

+   `status`列将存储每个任务的状态

+   `created_at`和`updated_at`列将存储任务的创建和更新日期

如果您在迁移文件中写入`$table->timestamps()`，Laravel 的`migration`

类会自动创建`created_at`和`updated_at`列。正如您从第一章中所知，*构建 URL 缩短网站*，要应用迁移，我们应该运行以下命令：

```php
**php artisan migrate**

```

运行命令后，如果您检查数据库，您会看到我们的`todos`表和列已经创建。现在我们需要编写我们的模型。

# 创建一个待办事项模型

要创建一个模型，您应该用文件管理器打开`app/models/`目录。在该目录下创建一个名为`Todo.php`的文件，并编写以下代码：

```php
<?php
class Todo extends Eloquent
{
  protected $table = 'todos';

}
```

让我们来看一下`Todo.php`文件。

如您所见，我们的`Todo`类扩展了 Laravel 的**ORM**（**对象关系映射器**）数据库类`Eloquent`。

`protected $table = 'todos';`代码告诉`Eloquent`关于我们模型的表名。如果我们不设置`table`变量，`Eloquent`会接受小写模型名称的复数版本作为表名。因此，从技术上讲这并不是必需的。

现在，我们的应用程序需要一个模板文件，所以让我们创建它。

# 创建模板

Laravel 使用一个名为**Blade**的模板引擎来处理静态和应用程序模板文件。Laravel 从`app/views/`目录调用模板文件，因此我们需要在该目录下创建我们的第一个模板。

1.  创建一个名为`index.blade.php`的文件。

1.  文件包含以下代码：

```php
    <html>
      <head>
        <title>To-do List Application</title>
        <link rel="stylesheet" href="assets/css/style.css">
        <!--[if lt IE 9]><scriptsrc="//html5shim.googlecode.com/svn/trunk/html5.js"></script><![endif]-->

      </head>
      <body>
        <div class="container">
          <section id="data_section" class="todo">
            <ul class="todo-controls">
            <li><img src="/assets/img/add.png" width="14px"onClick="show_form('add_task');" /></li>
            </ul>
              <ul id="task_list" class="todo-list">
              @foreach($todos as $todo)
                @if($todo->status)
                  <li id="{{$todo->id}}" class="done"><a href="#" class="toggle"></a><span id="span_{{$todo->id}}">{{$todo->title}}</span> <a href="#"onClick="delete_task('{{$todo->id}}');"class="icon-delete">Delete</a> <a href="#"onClick="edit_task('{{$todo->id}}','{{$todo->title}}');"class="icon-edit">Edit</a></li>
                @else
                  <li id="{{$todo->id}}"><a href="#"onClick="task_done('{{$todo->id}}');"class="toggle"></a> <span id="span_{{$todo->id}}">{{$todo->title}}</span><a href="#" onClick="delete_task('{{$todo->id}}');" class="icon-delete">Delete</a>
                    <a href="#" onClick="edit_task('{{$todo->id}}','{{$todo->title}}');"class="icon-edit">Edit</a></li>
                @endif
              @endforeach
            </ul>
          </section>
          <section id="form_section">

          <form id="add_task" class="todo"
            style="display:none">
          <input id="task_title" type="text" name="title"placeholder="Enter a task name" value=""/>
          <button name="submit">Add Task</button>
          </form>

            <form id="edit_task" class="todo"style="display:none">
            <input id="edit_task_id" type="hidden" value="" />
            <input id="edit_task_title" type="text"name="title" value="" />
            <button name="submit">Edit Task</button>
          </form>

        </section>

        </div>
        <script src="http://code.jquery.com/jquery-latest.min.js"type="text/javascript"></script>
        <script src="assets/js/todo.js"type="text/javascript"></script>
      </body>
    </html>
    ```

如果您是第一次编写 Blade 模板，上面的代码可能很难理解，所以我们将尝试解释一下。您会在文件中看到一个`foreach`循环。这个语句循环遍历我们的`todo`记录。

在本章中创建控制器时，我们将为您提供更多关于它的知识。

`if`和`else`语句用于区分已完成和待办任务。我们使用`if`和`else`语句来为任务设置样式。

我们还需要一个模板文件，用于动态向任务列表追加新记录。在`app/views/`文件夹下创建一个名为`ajaxData.blade.php`的文件。文件包含以下代码：

```php
@foreach($todos as $todo)
  <li id="{{$todo->id}}"><a href="#" onClick="task_done('{{$todo->id}}');" class="toggle"></a> <span id="span_{{$todo>id}}">{{$todo->title}}</span> <a href="#"onClick="delete_task('{{$todo->id}}');" class="icondelete">Delete</a> <a href="#" onClick="edit_task('{{$todo>id}}','{{$todo->title}}');" class="icon-edit">Edit</a></li>
@endforeach
```

此外，您会在静态文件的源路径中看到`/assets/`目录。当您查看`app/views`目录时，您会发现没有名为`assets`的目录。Laravel 将系统文件和公共文件分开。公共可访问文件位于`root`目录下的`public`文件夹中。因此，您应该在公共文件夹下创建一个`asset`文件夹。

我们建议使用这些类型的有组织的文件夹来开发整洁易读的代码。最后，您会发现我们是从 jQuery 的主网站调用的。我们还建议您以这种方式在应用程序中获取最新的稳定的 jQuery。

您可以根据自己的意愿为应用程序设置样式，因此我们不会在这里检查样式代码。我们将把我们的`style.css`文件放在`/public/assets/css/`下。

为了执行 Ajax 请求，我们需要 JavaScript 编码。这段代码发布了我们的`add_task`和`edit_task`表单，并在任务完成时更新它们。让我们在`/public/assets/js/`中创建一个名为`todo.js`的 JavaScript 文件。文件包含以下代码：

```php
function task_done(id){

  $.get("/done/"+id, function(data) {

    if(data=="OK"){

      $("#"+id).addClass("done");
    }

  });
}
function delete_task(id){

  $.get("/delete/"+id, function(data) {

    if(data=="OK"){
      var target = $("#"+id);

      target.hide('slow', function(){ target.remove(); });

    }

  });
}

function show_form(form_id){

  $("form").hide();

  $('#'+form_id).show("slow");

}
function edit_task(id,title){

  $("#edit_task_id").val(id);

  $("#edit_task_title").val(title);

  show_form('edit_task');
}
$('#add_task').submit(function(event) {

  /* stop form from submitting normally */
  event.preventDefault();

  var title = $('#task_title').val();
  if(title){
    //ajax post the form
    $.post("/add", {title: title}).done(function(data) {

      $('#add_task').hide("slow");
      $("#task_list").append(data);
    });

  }
  else{
    alert("Please give a title to task");
    }
});

$('#edit_task').submit(function() {

  /* stop form from submitting normally */
  event.preventDefault();

  var task_id = $('#edit_task_id').val();
  var title = $('#edit_task_title').val();
  var current_title = $("#span_"+task_id).text();
  var new_title = current_title.replace(current_title, title);
  if(title){
    //ajax post the form
    $.post("/update/"+task_id, {title: title}).done(function(data){
      $('#edit_task').hide("slow");
      $("#span_"+task_id).text(new_title);
    });
  }
  else{
    alert("Please give a title to task");
  }
});
```

让我们检查 JavaScript 文件。

# 使用 Ajax 将数据插入到数据库

在这个应用程序中，我们将使用**Ajax POST**方法将数据插入到数据库中。jQuery 是这类应用程序的最佳 JavaScript 框架。jQuery 还带有强大的选择器功能。

我们的 HTML 代码中有两个表单，所以我们需要使用 Ajax 提交它们来插入或更新数据。我们将使用 jQuery 的`post()`方法来实现。

我们将在`/public/assets/js`下提供我们的 JavaScript 文件，因此让我们在该目录下创建一个`todo.js`文件。首先，我们需要一个请求来添加新任务。JavaScript 代码包含以下代码：

```php
$('#add_task').submit(function(event) {
  /* stop form from submitting normally */
  event.preventDefault();
  var title = $('#task_title').val();
  if(title){
    //ajax post the form
    $.post("/add", {title: title}).done(function(data) {
      $('#add_task').hide("slow");
      $("#task_list").append(data);
    });
  }
  else{
    alert("Please give a title to task");
  }
});
```

如果用户记得为任务提供标题，这段代码将把我们的`add_task`表单发布到服务器。如果用户忘记为任务提供标题，代码将不会发布表单。发布后，代码将隐藏表单并向任务列表附加一个新记录。同时，我们将等待响应以获取数据。

因此，我们需要第二个表单来更新任务的标题。代码将通过 Ajax 实时更新任务的标题，并更改更新记录的文本。实时编程（或现场编码）是一种编程风格，程序员/表演者/作曲家在程序运行时增加和修改程序，而无需停止或重新启动，以在运行时断言表达式、可编程控制性能、组合和实验。由于编程语言的基本功能，我们相信实时编程的技术和美学方面值得在 Web 应用程序中探索。更新表单的代码应该如下：

```php
$('#edit_task').submit(function(event) {
  /* stop form from submitting normally */
  event.preventDefault();
  var task_id = $('#edit_task_id').val();
  var title = $('#edit_task_title').val();
  var current_title = $("#span_"+task_id).text();
  var new_title = current_title.replace(current_title, title);
  if(title){
    //ajax post the form
    $.post("/update/"+task_id, {title: title}).done(function(data){
      $('#edit_task').hide("slow");
      $("#span_"+task_id).text(new_title);
    });
  }
  else{
    alert("Please give a title to task");
  }
});
```

Laravel 具有 RESTful 控制器功能。这意味着您可以定义路由和控制器函数的 RESTful 基础。此外，可以为不同的请求类型（如**POST**、**GET**、**PUT**或**DELETE**）定义路由。

在定义路由之前，我们需要编写我们的控制器。控制器文件位于`app/controllers/`下；在其中创建一个名为`TodoController.php`的文件。控制器代码应该如下：

```php
<?php
class TodoController extends BaseController
{
  public $restful = true;
  public function postAdd() {
    $todo = new Todo();
    $todo->title = Input::get("title");
    $todo->save();      
    $last_todo = $todo->id;
    $todos = Todo::whereId($last_todo)->get();
    return View::make("ajaxData")->with("todos", $todos);
  }
  public function postUpdate($id) {        
    $task = Todo::find($id);
    $task->title = Input::get("title");
    $task->save();
    return "OK";        
  }
}
```

让我们检查代码。

正如您在代码中所看到的，RESTful 函数定义了诸如`postFunction`、`getFunction`、`putFunction`或`deleteFunction`之类的语法。

我们有两个提交表单，所以我们需要两个 POST 函数和一个 GET 方法来从数据库中获取记录，并在`foreach`语句中在模板中向访问者显示它们。

让我们检查前面的代码中的`postUpdate()`方法：

```php
public function postUpdate($id) {
  $task = Todo::find($id);
  $task->title = Input::get("title");
  $task->save();
  return "OK";
}
```

以下几点解释了前面的代码：

+   该方法需要一个名为`id`的记录来更新。我们提交的路由将类似于`/update/record_id`。

+   `$task = Todo::find($id);`是从数据库中查找具有给定`id`的记录的方法的一部分。

+   `$task->title = Input::get("title");`意味着获取名为`title`的表单元素的值，并将`title`列的记录更新为发布的值。

+   `$task->save();`应用更改并在数据库服务器上运行更新查询。

让我们检查`postAdd()`方法。这个方法的工作方式类似于我们的`getIndex()`方法。代码的第一部分在数据库服务器上创建一个新记录：

```php
public function postAdd() {
  $todo = new Todo();
  $todo->title = Input::get("title");
  $todo->save();      
  $last_todo = $todo->id;
  $todos = Todo::whereId($last_todo)->get();
  return View::make("ajaxData")->with("todos", $todos);
}
```

以下几点解释了前面的代码：

+   代码行`$last_todo = $todo->id;`获取了这条记录的 ID。它相当于`mysql_insert_id()`函数。

+   代码行`$todos = Todo::whereId($last_todo)->get();`从具有`id`列等于`$last_todo`变量的`todo`表中获取记录。

+   代码行`View::make("ajaxData") ->with("todos", $todos);`非常重要，以了解 Laravel 的视图机制：

+   代码行`View::make("ajaxData")`指的是我们的模板文件。你还记得我们在`/app/views/`下创建的`ajaxData.blade.php`文件吗？代码调用了这个文件。

+   代码行`->with("todos", $todos);`将最后一条记录分配给模板文件，作为名为`todos`的变量（第一个参数）。因此，我们可以使用`foreach`循环在模板文件中显示最后一条记录。

# 从数据库中检索列表

我们还需要一种方法来从我们的数据库服务器中获取现有数据。在我们的控制器文件中，我们需要如下所示的函数：

```php
public function getIndex() {
  $todos = Todo::all();
  return View::make("index")
    ->with("todos", $todos);
}
```

让我们来看一下`getIndex()`方法：

+   在代码中，`$todos = Todo:all()`表示从数据库中获取所有记录并将它们分配给`$todos`变量。

+   在代码中，`View::make("index")`定义了我们的模板文件。你还记得我们在`/app/views/`下创建的`index.blade.php`文件吗？代码调用了这个文件。

+   在代码中，`->with("todos", $todos);`将记录分配给模板文件。因此，我们可以使用`foreach`循环在模板文件中显示记录。

最后，我们将定义我们的路由。要定义路由，您应该在`apps`文件夹中打开`routes.php`文件。Laravel 有一个很好的功能，用于定义名为 RESTful 控制器的路由。您可以使用一行代码定义所有路由，如下所示：

```php
Route::controller('/', 'TodoController');
```

上述代码将所有应用程序基于根的请求分配给`TodoController`函数。如果需要，您也可以手动定义路由，如下所示：

```php
Route::method('path/{variable}', 'TheController@functionName');
```

# 如何仅允许 Ajax 请求

我们的应用程序甚至可以在没有 Ajax 的情况下接受所有 POST 和 GET 请求。但我们只需要允许`add`和`update`函数的 Ajax 请求。Laravel 的`Request`类为您的应用程序提供了许多检查 HTTP 请求的方法。其中一个函数名为`ajax()`。我们可以在控制器或路由过滤器下检查请求类型。

## 使用路由过滤器允许请求

路由过滤器提供了一种方便的方式来限制、访问或过滤给定路由的请求。Laravel 中包含了几个过滤器，这些过滤器位于`app`文件夹中的`filters.php`文件中。我们可以在这个文件下定义我们自定义的过滤器。我们将不在本章中使用这种方法，但我们将在后续章节中研究路由过滤器。用于 Ajax 请求的路由过滤器应该如下所示的代码所示：

```php
Route::filter('ajax_check', function()
{
  if (Request::ajax())
    {
      return true;
    }
});
```

将过滤器附加到路由也非常容易。检查以下代码中显示的示例路由：

```php
Route::get('/add', array('before' => 'ajax_check', function()
{
    return 'The Request is AJAX!';
}));
```

在前面的示例中，我们为具有`before`变量的路由定义了一个路由过滤器。这意味着我们的应用首先检查请求类型，然后调用控制器函数并传递数据。

## 使用控制器端允许请求

我们可以在控制器下检查请求类型。我们将在本节中使用这种方法。这种方法对于基于函数的过滤非常有用。为此，我们应该按照以下代码所示更改我们的`add`和`update`函数：

```php
public function postAdd() {
  if(Request::ajax()){
    $todo = new Todo();
    $todo->title = Input::get("title");
    $todo->save();
    $last_todo = $todo->id;
    $todos = Todo::whereId($last_todo)->get();
    return View::make("ajaxData")->with("todos", $todos);
  }
}
public function postUpdate($id) {
  if(Request::ajax()){
    $task = Todo::find($id);
    $task->title = Input::get("title");
    $task->save();
    return "OK"; 
  }
}
```

# 总结

在本章中，我们编写了添加新任务的代码，更新了它，并列出了任务。我们还需要更新每个状态并删除任务。为此，我们需要两个名为`getDone()`和`getDelete()`的函数。正如你从本章的前几节中所了解的那样，这些函数是 RESTful 的，并接受 GET 方法请求。因此，我们的函数应该如下所示的代码所示：

```php
public function getDelete($id) {
  if(Request::ajax()){
    $todo = Todo::whereId($id)->first();
    $todo->delete();
    return "OK";
  }
}
public function getDone($id) {
  if(Request::ajax()){
    $task = Todo::find($id);
    $task->status = 1;
    $task->save();
    return "OK";
  }
}
```

我们还需要更新`todo.js`文件。最终的 JavaScript 代码应该如下所示的代码所示：

```php
function task_done(id){
  $.get("/done/"+id, function(data) {
    if(data=="OK"){
      $("#"+id).addClass("done");
    }
  });
}
function delete_task(id){
  $.get("/delete/"+id, function(data) {
    if(data=="OK"){
      var target = $("#"+id);
      target.hide('slow', function(){ target.remove(); });
    }
  });
}
function show_form(form_id){
  $("form").hide();
  $('#'+form_id).show("slow");
}
function edit_task(id,title){
  $("#edit_task_id").val(id);
  $("#edit_task_title").val(title);
  show_form('edit_task');
}
$('#add_task').submit(function(event) {
/* stop form from submitting normally */
  event.preventDefault();
  var title = $('#task_title').val();
  if(title){
    //ajax post the form
    $.post("/add", {title: title}).done(function(data) {
      $('#add_task').hide("slow");
      $("#task_list").append(data);
    });
  }
  else{
    alert("Please give a title to task");
  }
});
$('#edit_task').submit(function(event) {
/* stop form from submitting normally */
  event.preventDefault();
  var task_id = $('#edit_task_id').val();
  var title = $('#edit_task_title').val();
  var current_title = $("#span_"+task_id).text();
  var new_title = current_title.replace(current_title, title);
  if(title){
    //ajax post the form
    $.post("/update/"+task_id, {title:title}).done(function(data) {
      $('#edit_task').hide("slow");
      $("#span_"+task_id).text(new_title);
    });
  }
  else{
    alert("Please give a title to task");
  }
});
```

# 总结

在本节中，我们试图了解如何在 Laravel 中使用 Ajax。在整个章节中，我们使用了模板化、请求过滤、路由和 RESTful 控制器的基础知识。我们还学会了如何从数据库中更新和删除数据。

在下一章中，我们将尝试检查 Laravel 的文件验证和文件处理方法。
