# 第五章。将您的应用程序连接到 CouchDB

> 现在我们已经建立了应用程序的框架，让我们谈谈我们的应用程序需要与 CouchDB 通信的情况。

在本章中，我们将讨论以下几点：

+   调查与 CouchDB 交互的快速简便方法，并讨论其缺点

+   查看现有库以便于 PHP 和 CouchDB 开发

+   安装 Sag 并将其集成到 Bones 中

+   让我们的注册表单创建 CouchDB 文档，并在 Futon 中进行验证

# 在我们开始之前

在我们做任何事情之前，让我们创建一个数据库，从此时起我们将在 Verge 中使用。与以前一样，让我们使用`curl`创建一个数据库。

# 行动时间-使用 curl 为 Verge 创建数据库

我们在第三章中使用`curl`创建了一个数据库，*与 CouchDB 和 Futon 入门*。让我们快速回顾如何使用`PUT`请求在 CouchDB 中创建一个新数据库。

1.  通过在**终端**中运行以下命令来创建一个新的数据库。确保用第三章中创建的数据库管理员用户替换`username`和`password`。

```php
**curl -X PUT username:password@localhost:5984/verge** 

```

1.  **终端**将以以下输出做出响应：

```php
**{"ok":true}** 

```

## 刚刚发生了什么？

我们使用**终端**通过`curl`触发了一个`PUT`请求，使用 CouchDB 的**RESTful JSON API**创建了一个数据库。我们在 CouchDB 的根 URL 末尾传递`verge`作为数据库的名称。成功创建数据库后，我们收到了一条消息，说明一切都很顺利。

# 头顶冲入

在本节中，我们将创建一些快速脏代码来与 CouchDB 通信，然后讨论这种方法的一些问题。

## 向我们的注册脚本添加逻辑

在上一章中，我们在`views/signup.php`中创建了一个表单，具有以下功能：

+   我们要求用户在文本框中输入名称的值

+   我们获取了表单中输入的值并将其发布到注册路由

+   我们使用 Bones 来获取表单传递的值，并将其设置为名为`message`的变量，以便我们可以在主页上显示它

+   我们呈现了主页并显示了`message`变量

这是我们的一项重大工作，但我们无法保存任何东西以供以后阅读或写入。

让我们进一步采取一些步骤，并要求用户输入姓名和电子邮件地址，然后将这些字段保存为 CouchDB 中的文档。

# 行动时间-向注册表单添加电子邮件字段

让我们添加一个输入字段，以便用户可以在`views/signup.php`页面中输入电子邮件地址。

1.  在文本编辑器中打开`signup.php`（`/Library/Webserver/Documents/verge/views/signup.php`）

1.  添加突出显示的代码以为电子邮件地址添加标签和输入字段：

```php
Signup
<form action="<?php echo $this->make_route('signup') ?>" method="post">
<label for="name">Name</label>
<input id="name" name="name" type="text"> <br />
**<label for="email">Email</label>
<input id="email" name="email" type="text"> <br />** 
<input type="Submit" value="Submit">
</form>

```

## 刚刚发生了什么？

我们向注册表单添加了一个额外的字段，用于接受电子邮件地址的输入。通过向此表单添加`email`字段，我们将能够在表单提交时访问它，并最终将其保存为 CouchDB 文档。

### 使用 curl 调用将数据发布到 CouchDB

在以前的章节中，我们已经使用了**终端**通过`curl`与 CouchDB 进行交互。您会高兴地知道，您还可以通过 PHP 使用`curl`。为了在 CouchDB 中表示数据，我们首先需要将我们的数据转换为 JSON 格式。

# 行动时间-创建一个标准对象以编码为 JSON

让我们以 JSON 的形式表示一个简单的对象，以便 CouchDB 可以解释它。

在文本编辑器中打开`index.php`，并将以下代码添加到`/signup POST`路由中：

```php
post('/signup', function($app) {
**$user = new stdClass;
$user->type = 'user';
$user->name = $app->form('name');
$user->email = $app->form('email');
echo json_encode($user);** 
$app->set('message', 'Thanks for Signing Up ' . $app->form('name') . '!');
$app->render('home');
});

```

## 刚刚发生了什么？

我们添加了创建存储用户具体信息的对象的代码。我们使用了`stdClass`的一个实例，并将其命名为`$user`。`stdClass`是 PHP 的通用空类，对于匿名对象、动态属性和快速上手非常有用。因为文档要求应该设置一个类型来分类文档，我们将这个文档的类型设置为`user`。然后我们取自表单提交的值，并将它们保存为`$user`类的属性。最后，我们使用了一个名为`json_encode`的 PHP 函数，将对象转换为 JSON 表示形式。

让我们来测试一下。

1.  在浏览器中打开`http://localhost/verge/signup`。

1.  在**名称**文本框中输入`John Doe`，在**电子邮件**文本框中输入`<john@example.com>`。

1.  点击**提交**。

1.  您的浏览器将显示以下内容：![刚刚发生了什么？](img/3586_05_005.jpg)

太好了！我们的表单已经正确提交了，并且我们能够在我们网站的顶部用 JSON 表示`stdClass $user`。

### 提交到 Git

让我们将我们的代码提交到 Git，这样我们以后可以回顾这段代码。

1.  打开**终端**。

1.  输入以下命令以更改目录到我们的工作目录：

```php
**cd /Library/Webserver/Documents/verge/** 

```

1.  给 Git 一个描述，说明我们自上次提交以来做了什么：

```php
**git commit am 'Added functionality to collect name and email through stdClass and display it onscreen.'** 

```

现在我们已经用 JSON 表示了我们的数据，让我们使用一个`curl`语句来使用 PHP 创建一个 CouchDB 文档。

# 接下来的步骤——使用 PHP 和 curl 创建 CouchDB 文档

自本书开始以来，我们一直在使用命令行通过`curl`，但这次，我们将使用 PHP 触发一个`curl`语句。

1.  让我们从初始化一个`curl`会话开始，执行它，然后关闭它。在文本编辑器中打开`index.php`，并将以下代码添加到`/signup POST`路由中：

```php
post('/signup', function($app) {
$user = new stdClass;
$user->type = 'user';
$user->name = $app->form('name');
$user->email = $app->form('email');
echo json_encode($user);
**$curl = curl_init();
// curl options
curl_exec($curl);
curl_close($curl);** 
$app->set('message', 'Thanks for Signing Up ' . $app- >form('name') . '!');
$app->render('home');
});

```

1.  现在，让我们告诉`curl`实际要执行什么。我们使用一个`options`数组来做到这一点。在`curl_init()`和`curl_exec`语句之间添加以下代码：

```php
post('/signup', function($app) {
$user = new stdClass;
$user->name = $app->form('name');
$user->email = $app->form('email');
echo json_encode($user);
$curl = curl_init();
// curl options
**$options = array(
CURLOPT_URL => 'localhost:5984/verge',
CURLOPT_POSTFIELDS => json_encode($user),
CURLOPT_HTTPHEADER => array ("Content-Type: application/json"),
CURLOPT_CUSTOMREQUEST => 'POST',
CURLOPT_RETURNTRANSFER => true,
CURLOPT_ENCODING => "utf-8",
CURLOPT_HEADER => false,
CURLOPT_FOLLOWLOCATION => true,
CURLOPT_AUTOREFERER => true
);
curl_setopt_array($curl, $options);** 
curl_exec($curl);
curl_close($curl);
$app->set('message', 'Thanks for Signing Up ' . $app-> form('name') . '!');
$app->render('home');
});

```

## 刚刚发生了什么？

我们首先使用 PHP 初始化了一个`curl`会话，通过使用`curl_init()`资源设置了一个名为`$curl`的变量。然后我们创建了一个包含各种键和值的数组。我们选择所有这些选项的原因对我们现在来说并不太重要，但我想强调前三个对象：

1.  我们将`CURLOPT_URL`选项设置为我们要将文档保存到的数据库的 URL。请记住，此语句将使用 CouchDB 的 RESTful JSON API 在`verge`数据库中创建一个文档。

1.  然后我们将`CURLOPT_POSTFIELDS`设置为我们的`$user`的 JSON 编码值。这将把我们的 JSON 字符串作为数据与 URL 一起包含进去。

1.  最后，我们将`CURLOPT_HTTPHEADER`设置为`array ("Content-Type: application/json")`，以确保`curl`知道我们正在传递一个 JSON 请求。

设置了我们的选项数组之后，我们需要告诉我们的`curl`实例使用它：

```php
curl_setopt_array($curl, $options);

```

然后我们用以下两行代码执行并关闭`curl`：

```php
curl_exec($curl);
curl_close($curl);

```

有了这段代码，我们应该能够提交表单并将其发布到 CouchDB。让我们来测试一下。

1.  在浏览器中打开`http://localhost/verge/signup`。

1.  在**名称**文本框中输入`John Doe`，在**电子邮件**文本框中输入`<john@example.com>`。

1.  点击**提交**。

1.  您的浏览器将显示以下内容：![刚刚发生了什么？](img/3586_05_005.jpg)

这次也没有出现任何错误，就像以前一样。但是这次应该已经创建了一个 CouchDB 文档。让我们通过 Futon 检查文档是否已经正确创建。

1.  在浏览器中打开`http://localhost:5984/_utils/database.html?verge`。这个直接链接将显示 verge 数据库。您会看到这里有一个新的文档！请记住，您的`ID`和`rev`将与我的不同：![刚刚发生了什么？](img/3586_05_010.jpg)

1.  点击文档，以便您可以查看详细信息。

1.  您文档中的数据应该与我们在`curl`会话中传递的信息相匹配。请注意，`type, email`和`name`都已正确设置，CouchDB 为我们设置了`_id`和`_rev`。![刚刚发生了什么？](img/3586_05_015.jpg)

### 将其提交到 Git

让我们将我们的代码提交到 Git，以便将来可以参考这段代码。

1.  打开**终端**。

1.  键入以下命令以更改目录到我们的工作目录：

```php
cd /Library/Webserver/Documents/verge/

```

1.  向 Git 描述我们自上次提交以来所做的工作：

```php
git commit am 'CouchDB Documents can now be created through the signup form using curl.'

```

我们刚刚看了使用 PHP 创建 CouchDB 文档的最简单的方法之一。然而，我们需要评估我们刚刚编写的代码是否可持续，并且是否是我们开发应用程序的明智方式。

## 这种技术足够好吗？

棘手的问题。从技术上讲，我们可以以这种方式构建我们的应用程序，但我们需要添加更多的代码，并花费本书的其余时间重构我们对`curl`的调用，直到它完美运行。然后，我们需要花大量时间将我们的调用重构为一个简单的库，以便更容易修复问题。简而言之，这种技术不起作用，因为我们想专注于构建我们的应用程序，而不是解决 PHP 和 CouchDB 之间的所有通信问题。幸运的是，有各种各样的 CouchDB 库可以简化我们的开发过程。

# 可用的 CouchDB 库

有各种库可以在使用 PHP 和 CouchDB 开发时使我们的生活更轻松。所有这些库都是开源项目，这很棒！但是，其中一些库已经不再积极开发以支持较新版本的 CouchDB。因此，我们需要选择要使用的库。

一些 PHP 和 CouchDB 库的列表可以在这里看到：[`wiki.apache.org/couchdb/Getting_started_with_PHP`](http://wiki.apache.org/couchdb/Getting_started_with_PHP)，还有一些其他的库托管在 GitHub 上，需要更深入挖掘。

每个库都有其优势，但由于简单是 Bones 的关键概念，因此在我们的 PHP 库中也应该追求简单。说到这一点，我们最好的解决方案就是**Sag**。

# Sag

Sag 是由 Sam Bisbee 创建的用于 CouchDB 的出色的 PHP 库。Sag 的指导原则是简单，创建一个功能强大的接口，几乎没有额外开销，可以轻松集成到任何应用程序结构中。它不强制您的应用程序使用框架、文档的特殊类或 ORM，但如果您愿意，仍然可以使用。Sag 接受基本的 PHP 数据结构（对象、字符串等），并返回原始 JSON 或响应和对象中的 HTTP 信息。

我将为您介绍 Sag 的安装和基本功能，但您也可以访问 Sag 的网站：[`www.saggingcouch.com/`](http://www.saggingcouch.com/)，了解示例和文档。

## 下载并设置 Sag

Sag 相当不显眼，将完全适应我们当前的应用程序结构。我们只需要使用 Git 从其 GitHub 存储库中获取 Sag，并将其放在我们的`lib`目录中。

# 采取行动——使用 Git 安装 Sag

Git 使设置第三方库变得非常容易，并允许我们在可用时更新到新版本。

1.  打开**终端**。

1.  键入以下命令以确保您在工作目录中：

```php
**cd /Library/Webserver/Documents/verge/** 

```

1.  使用 Git 将 Sag 添加到我们的存储库：

```php
**git submodule add git://github.com/sbisbee/sag.git lib/sag
git submodule init** 

```

## 刚刚发生了什么？

我们使用 Git 使用`git submodule add`将 Sag 添加到我们的项目中，然后通过键入`git submodule init`来初始化子模块。Git 的子模块允许我们在我们的存储库中拥有一个完整的 Git 存储库。每当 Sag 发布新版本时，您可以运行`git submodule update`，您将收到最新和最棒的代码。

### 将 Sag 添加到 Bones

为了使用 Sag，我们将在`Bones`中添加几行代码，以确保我们的库可以看到并利用它。

# 行动时间-将 Sag 添加到 Bones

启用并设置 Sag 与`Bones`一起工作非常容易。让我们一起来看看！

1.  打开我们的工作目录中的`lib/bones.php`，并在我们的类顶部添加以下行：

```php
<?php
define('ROOT', __DIR__ . '/..');
**require_once ROOT . '/lib/sag/src/Sag.php';** 

```

1.  我们需要确保 Sag 已准备好并在每个请求中可用。让我们通过在`Bones`中添加一个名为`$couch`的新变量，并在我们的`__construct`函数中设置它来实现这一点：

```php
public $route_segments = array();
public $route_variables = array();
**public $couch;** 
public function __construct() {
$this->route = $this->get_route();
$this->route_segments = explode('/', trim($this->route, '/'));
$this->method = $this->get_method();
**$this->couch = new Sag('127.0.0.1', '5984');
$this->couch->setDatabase('verge');** 
}

```

## 刚刚发生了什么？

我们确保`Bones`可以访问和使用 Sag，通过使用`require_once`加载 Sag 资源。然后，我们确保每次构造`Bones`时，我们都会定义数据库服务器和端口，并设置我们要使用的数据库。

### 注意

请注意，我们与`Verge`数据库交互时不需要任何凭据，因为我们尚未对此数据库设置任何权限。

## 使用 Sag 简化我们的代码

在我们的应用程序中包含 Sag 后，我们可以简化我们的数据库调用，将处理和异常处理交给 Sag，并专注于构建我们的产品。

# 行动时间-使用 Sag 创建文档

现在我们已经在应用程序中随处可用并准备好使用 Sag，让我们重构放置在`/signup post`路由中的用户类的保存。

打开`index.php`，删除我们在之前部分添加的所有额外代码，这样我们的`/signup post`路由看起来类似于以下代码片段：

```php
post('/signup', function($app) {
$user = new stdClass;
$user->name = $app->form('name');
$user->email = $app->form('email');
**$app->couch->post($user);** 
$app->set('message', 'Thanks for Signing Up ' . $app->form('name') . '!');
$app->render('home');
});

```

## 刚刚发生了什么？

我们使用 Sag 创建了一个到我们的 CouchDB 数据库的帖子，使用的代码大大减少了！Sag 的 post 方法允许您传递数据，因此触发起来非常容易。

让我们快速通过注册流程：

1.  打开浏览器，输入`http://localhost/verge/signup`。

1.  在**名称**文本框中输入一个新名称，然后在**电子邮件**文本框中输入一个新电子邮件。

1.  点击**提交**。

在 CouchDB 中创建了一个新文档，让我们检查一下 Futon，确保它在那里：

1.  打开浏览器，输入`http://localhost:5984/_utils/database.html?verge`，查看 verge 数据库。

1.  点击列表中的第二个文档。

1.  查看这个新文档的详细信息，您会发现它与我们制作的第一个文档具有相同的结构。

完美！结果与我们快速而肮脏的 curl 脚本完全一样，但我们的代码更简化，Sag 在幕后处理了很多事情。

### 注意

目前我们没有捕获或处理任何错误。我们将在以后的章节中更多地讨论如何处理这些错误。幸运的是，CouchDB 以友好的方式处理错误，并且 Sag 已经确保了很容易追踪问题。

## 添加更多结构

我们可以如此轻松地创建文档，这很棒，但对于我们的类来说，有一个强大的结构也很重要，这样我们可以保持有条理。

# 行动时间-包括类目录

为了我们能够使用我们的类，我们需要在`Bones`中添加一些代码，以便我们可以在使用时自动加载类名。这将实现这一点，这样我们就不必在添加新类时不断包含更多文件。

将以下代码添加到`lib/bones.php`：

```php
<?php
define('ROOT', __DIR__ . '/..');
**require_once ROOT . '/lib/sag/src/Sag.php';
function __autoload($classname) {
include_once(ROOT . "/classes/" . strtolower($classname) . ".php");
}** 

```

## 刚刚发生了什么？

我们在我们的`Bones`库中添加了一个`__autoload`函数，如果找不到类，它将给 PHP 最后一次尝试加载类名。`__autoload`函数传递了`$classname`，我们使用`$classname`来找到命名类的文件。我们使用`strtolower`函数使请求的`$classname`变成小写，这样我们就可以找到命名文件。然后我们添加了工作目录的根目录和`classes`文件夹。

### 使用类

现在我们有了加载类的能力，让我们创建一些！我们将从创建一个基类开始，所有其他类都将继承它的属性。

# 行动时间-创建一个基本对象

在这一部分，我们将创建一个名为`base.php`的基类，所有我们的类都将继承它。

1.  让我们从创建一个名为`base.php`的新文件开始，并将其放在工作目录内的`/Library/Webserver/Documents/verge/classes/base.php`文件夹中。

1.  在`base.php`中创建一个带有`__construct`函数的抽象类。在对象的`__construct`中，让我们将`$type`作为一个选项，并将其设置为一个受保护的变量，也称为`$type`。

```php
<?php
abstract class Base
{
protected $type;
public function __construct($type)
{
$this->type = $type;
}
}

```

1.  为了方便以后在我们的类中获取和设置变量，让我们在`__construct`函数之后添加`__get()`和`__set()`函数。

```php
<?php
abstract class Base
{
protected $type;
public function __construct($type)
{
$this->type = $type;
}
**public function __get($property) {
return $this->$property;
}
public function __set($property, $value) {
$this->$property = $value;
}** 
}

```

1.  每次我们将对象保存到 Couch DB 时，我们希望能够将其表示为 JSON 字符串。因此，让我们创建一个名为`to_json()`的辅助函数，它将把我们的对象转换成 JSON 格式。

```php
<?php
abstract class Base
{
protected $type;
public function __construct($type)
{
$this->type = $type;
}
public function __get($property) {
return $this->$property;
}
public function __set($property, $value) {
$this->$property = $value;
}
**public function to_json() {
return json_encode(get_object_vars($this));
}** 
}

```

## 刚刚发生了什么？

我们创建了一个名为`base.php`的基类，它将作为我们构建的所有其他类的基础。在类内部，我们定义了一个受保护的变量`$type`，它将存储文档的分类，如`user`或`post`。接下来，我们添加了一个`__construct`函数，它将在每次创建对象时被调用。这个函数接受选项`$type`，我们将在每个继承`Base`的类中设置它。然后，我们创建了`__get`和`__set`函数。`__get`和`__set`被称为**魔术方法**，它们将允许我们使用`get`和`set`受保护的变量，而无需任何额外的代码。最后，我们添加了一个名为`to_json`的函数，它使用`get_object_vars`和`json_encode`来表示我们的对象为 JSON 字符串。在我们的基类中做这样的小事情将使我们未来的生活变得更加轻松。

# 时间来行动了——创建一个 User 对象

现在我们已经创建了我们的`Base`类，让我们创建一个`User`类，它将包含与用户相关的所有属性和函数。

1.  创建一个名为`user.php`的新文件，并将其放在`base.php`所在的`classes`文件夹中。

1.  让我们创建一个继承我们`Base`类的类。

```php
<?php
class User extends Base
{
}

```

1.  让我们添加我们已经知道需要的两个属性`name`和`email`到我们的`User`类中。

```php
<?php
class User extends Base
{
**protected $name;
protected $email;** 
}

```

1.  让我们添加一个`__construct`函数，告诉我们的`Base`类，在创建时我们的文档类型是`user`。

```php
<?php
class User extends Base
{
protected $name;
protected $email;
**public function __construct()
{
parent::__construct('user');
}** 
}

```

## 刚刚发生了什么？

我们创建了一个简单的类`user.php`，它继承了`Base`。**继承**意味着它将继承可用的属性和函数，以便我们可以利用它们。然后，我们包括了两个受保护的属性`$name`和`$email`。最后，我们创建了一个`__construct`函数。在这种情况下，构造函数告诉父类（即我们的`Base`类），文档的类型是`user`。

# 时间来行动了——插入 User 对象

有了我们的新的`User`对象，我们可以轻松地将其插入到我们的应用程序代码中，然后就可以运行了。

1.  打开`index.php`文件，将`stdClass`改为`User()`。与此同时，我们还可以移除`$user->type = 'user'`，因为现在这个问题已经在我们的类中处理了：

```php
post('/signup', function($app) {
**$user = new User();** 
$user->name = $app->form('name');
$user->email = $app->form('email');
$app->couch->post($user);
}

```

1.  调整 Sag 的`post`语句，以便我们可以以 JSON 格式传递我们的类。

```php
post('/signup', function($app) {
$user = new User();
$user->name = $app->form('name');
$user->email = $app->form('email');
**$app->couch->post($user->to_json);** 
}

```

## 刚刚发生了什么？

我们用`User()`替换了`stdClass`的实例。这将使我们完全控制获取和设置变量。然后，我们移除了`$user->type = 'user'`，因为我们的`User`和`Base`对象中的`__construct`函数已经处理了这个问题。最后，我们添加了之前创建的`to_json()`函数，这样我们就可以将我们的对象作为 JSON 编码的字符串发送出去。

### 注意

Sag 在技术上可以自己处理一个对象的 JSON，但重要的是我们能够从我们的对象中检索到一个 JSON 字符串，这样你就可以以任何你想要的方式与 CouchDB 交互。将来可能需要回来使用`curl`或另一个库重写所有内容，所以重要的是你知道如何表示你的数据为 JSON。

### 测试一下

让我们快速再次通过我们的注册流程，确保一切仍然正常运行：

1.  打开浏览器到`http://localhost/verge/signup`。

1.  在**名称**文本框中输入一个新名称，在**电子邮件**文本框中输入一个新电子邮件。

1.  点击**提交**。

在 CouchDB 中应该已经创建了一个新文档。让我们检查 Futon，确保它在那里：

1.  打开浏览器到`http://localhost:5984/_utils/database.html?verge`查看 verge 数据库。

1.  点击列表中的第三个文档

1.  查看这个新文档的细节，你会发现它和我们制作的前两个文档结构相同。

完美！一切都和以前一样，但现在我们使用了一个更加优雅的解决方案，我们将能够在未来的章节中构建在其基础上。

### 提交到 Git

让我们把代码提交到 Git，这样我们就可以追踪我们到目前为止的进展：

1.  打开**终端**。

1.  输入以下命令以更改目录到我们的工作目录：

```php
**cd /Library/Webserver/Documents/verge/** 

```

1.  我们在`classes`文件夹中添加了一些新文件。所以，让我们确保将这些文件添加到 Git 中。

```php
**git add classes/*** 

```

1.  给 Git 一个描述，说明我们自上次提交以来做了什么：

```php
**git commit am 'Added class structure for Users and tested its functionality'** 

```

通过使用`classes/*`语法，我们告诉 Git 添加 classes 文件夹中的每个文件。当你添加了多个文件并且不想逐个添加每个文件时，这很方便。

# 总结

我们已经完成了这一章的代码。定期将代码推送到 GitHub 是一个很好的做法。事实上，当你与多个开发人员一起工作时，这是至关重要的。我不会在这本书中再提醒你这样做。所以，请确保经常这样做：

```php
**git push origin master** 

```

这行代码读起来像一个句子，如果你在其中加入一些词。这句话告诉 Git 要`push`到`origin`（我们已经定义为 GitHub），并且我们要发送`master`分支。

# 总结

希望你喜欢这一章。当所有这些技术一起工作，让我们能够轻松地保存东西到 CouchDB 时，这是很有趣的。

让我们回顾一下这一章我们谈到的内容：

+   我们看了几种不同的方法，我们可以用 PHP 与 CouchDB 交流

+   我们将 Sag 与 Bones 联系起来

+   我们建立了一个面向对象的类结构，这将为我们节省很多麻烦

+   我们测试了一下，确保当我们提交我们的注册表单时，CouchDB 文档被创建了

在下一章中，我们将积极地研究 CouchDB 已经为我们的用户提供的一些很棒的功能，以及我们如何使用 CouchDB 来构建大多数应用程序都具有的标准注册和登录流程。伸展你的打字手指，准备一大杯咖啡，因为我们即将开始真正的乐趣。
