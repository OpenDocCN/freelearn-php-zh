# 第三章。错误处理

现在我们已经构建了使用 PDO 的第一个应用程序，我们将更仔细地研究用户友好的 Web 应用程序的一个重要方面——错误处理。它不仅通知用户有错误发生，而且在错误发生时没有被检测到时，它还限制了损害。

大多数 Web 应用程序都有相当简单的错误处理策略。当发生错误时，脚本终止并显示错误页面。错误应该记录在错误日志中，开发人员或维护人员应该定期检查日志。数据库驱动的 Web 应用程序中最常见的错误来源如下：

+   服务器软件故障或过载，比如著名的“连接过多”错误

+   应用程序配置不当，当我们使用不正确的连接字符串时可能会发生，这在将应用程序从一个主机移动到另一个主机时是一个相当常见的错误

+   用户输入验证不当，可能导致 SQL 格式不正确，从而导致查询失败

+   插入具有重复主键或唯一索引值的记录，这可能是应用程序业务逻辑错误导致的，也可能发生在受控情况下

+   SQL 语句中的语法错误

在本章中，我们将扩展我们的应用程序，以便我们可以编辑现有记录以及添加新记录。由于我们将处理通过 Web 表单提供的用户输入，我们必须对其进行验证。此外，我们可能会添加错误处理，以便我们可以对非标准情况做出反应，并向用户呈现友好的消息。

在我们继续之前，让我们简要地检查上面提到的错误来源，并看看在每种情况下应该应用什么错误处理策略。我们的错误处理策略将使用异常，所以你应该熟悉它们。如果你不熟悉，你可以参考附录 A，它将向你介绍 PHP5 的新面向对象特性。

我们有意选择使用异常，即使 PDO 可以被指示不使用它们，因为有一种情况是它们无法避免的。当数据库对象无法创建时，PDO 构造函数总是抛出异常，所以我们可能会将异常作为我们在整个代码中的主要错误捕获方法。

# 错误来源

要创建一个错误处理策略，我们首先应该分析错误可能发生的地方。错误可能发生在对数据库的每次调用上，尽管这可能不太可能，我们将研究这种情况。但在这样做之前，让我们检查每个可能的错误来源，并为处理它们定义一个策略。

## 服务器软件故障或过载

这可能发生在一个非常繁忙的服务器上，无法处理更多的传入连接。例如，后台可能正在运行一个漫长的更新。结果是我们无法从数据库中获取任何数据，所以我们应该做以下事情。

如果 PDO 构造函数失败，我们会显示一个页面，上面显示一条消息，说明用户的请求目前无法满足，他们应该稍后再试。当然，我们也应该记录这个错误，因为它可能需要立即处理。（一个好主意是通过电子邮件通知数据库管理员有关这个错误。）

这个错误的问题在于，虽然它通常在与数据库建立连接之前就显现出来（在调用 PDO 构造函数时），但有一点风险，它可能在连接建立之后发生（在调用`PDO`或`PDOStatement`对象的方法时，数据库服务器正在关闭）。在这种情况下，我们的反应将是一样的——向用户呈现一个错误消息，要求他们稍后再试。

## 应用程序配置不当

这个错误只会在我们将应用程序从数据库访问细节不同的服务器上移动时发生；这可能是当我们从开发服务器上传到生产服务器时，数据库设置不同。这不是在应用程序正常执行期间可能发生的错误，但在上传时应该注意，因为这可能会中断网站的运行。

如果发生此错误，我们可以显示另一个错误消息，如：“该网站正在维护中”。在这种情况下，网站维护者应立即做出反应，因为如果不纠正连接字符串，应用程序就无法正常运行。

## 用户输入验证不正确

这是一个与 SQL 注入漏洞密切相关的错误。每个数据库驱动应用程序的开发人员都必须采取适当的措施来验证和过滤所有用户输入。这个错误可能导致两个主要后果：要么查询由于 SQL 格式不正确而失败（因此不会发生特别糟糕的事情），要么可能发生 SQL 注入并且应用程序安全可能会受到损害。虽然后果不同，但这两个问题可以以相同的方式防止。

让我们考虑以下情景。我们从表单中接受一些数值，并将其插入数据库。为了使我们的例子简单，假设我们想要更新一本书的出版年份。为了实现这一点，我们可以创建一个包含书的 ID 的隐藏字段和一个输入年份的文本字段的表单。我们将在这里跳过实现细节，并看看使用一个设计不良的脚本来处理这个表单可能会导致错误并将整个系统置于风险之中。

表单处理脚本将检查两个请求变量：`$_REQUEST['book']`，其中包含书的 ID 和`$_REQUEST['year']`，其中包含出版年份。如果没有对这些值进行验证，最终的代码将类似于这样：

```php
$book = $_REQUEST['book'];
$year = $_REQUEST['year'];
$sql = "UPDATE books SET year=$year WHERE id=$book";
$conn->query($sql);

```

让我们看看如果用户将`book`字段留空会发生什么。最终的 SQL 将如下所示：

```php
UPDATE books SET year= WHERE id=1;

```

这个 SQL 是格式不正确的，会导致语法错误。因此，我们应该确保这两个变量都包含数值。如果它们不包含数值，我们应该重新显示表单并显示错误消息。

现在，让我们看看攻击者如何利用这一点来删除整个表的内容。为了实现这一点，他们可以在`year`字段中输入以下内容：

```php
2007; DELETE FROM books;

```

这将一个查询变成了三个查询：

```php
UPDATE books SET year=2007; DELETE FROM books; WHERE book=1;

```

当然，第三个查询是格式不正确的，但第一个和第二个将执行，并且数据库服务器将报告一个错误。为了解决这个问题，我们可以使用简单的验证来确保`year`字段包含四位数字。然而，如果我们有可能包含任意字符的文本字段，字段的值在创建 SQL 之前必须进行转义。

## 插入具有重复主键或唯一索引值的记录

当应用程序插入具有主键或唯一索引的重复值的记录时，可能会出现这个问题。例如，在我们的作者和书籍数据库中，我们可能希望防止用户因错误而两次输入相同的书。为此，我们可以在`books`表的 ISBN 列上创建一个唯一索引。由于每本书都有一个唯一的 ISBN，任何尝试插入相同的 ISBN 都会生成一个错误。我们可以捕获这个错误，并通过显示一个错误消息要求用户纠正 ISBN 或取消其添加来做出相应反应。

## SQL 语句中的语法错误

如果我们没有正确测试应用程序，可能会发生此错误。一个好的应用程序不应包含这些错误，开发团队有责任测试每种可能的情况，并检查每个 SQL 语句是否执行时没有语法错误。

如果发生这种错误，我们会使用异常来捕获它，并显示一个致命错误消息。开发人员必须立即纠正这种情况。

现在我们已经了解了可能的错误来源，让我们来看看 PDO 如何处理错误。

# PDO 中的错误处理类型

默认情况下，PDO 使用**静默错误处理模式**。这意味着调用`PDO`或`PDOStatement`类的方法时发生的任何错误都不会被报告。在这种模式下，每次发生错误时，都必须调用`PDO::errorInfo()`、`PDO::errorCode()`、`PDOStatement::errorInfo()`或`PDOStatement::errorCode()`来查看是否真的发生了错误。请注意，这种模式类似于传统的数据库访问——通常，在调用可能引起错误的函数之后，代码会调用`mysql_errno()`和`mysql_error()`（或其他数据库系统的等效函数），在连接到数据库之后和发出查询之后。

另一种模式是**警告模式**。在这里，`PDO`将与传统的数据库访问行为相同。与数据库通信期间发生的任何错误都会引发一个`E_WARNING`错误。根据配置，可能会显示错误消息或将其记录到文件中。

最后，PDO 引入了一种处理数据库连接错误的现代方式——使用**异常**。对`PDO`或`PDOStatement`方法的任何失败调用都会引发异常。

正如我们之前注意到的，PDO 默认使用静默模式。要切换到所需的错误处理模式，我们必须通过调用`PDO::setAttribute()`方法来指定它。每个错误处理模式由 PDO 类中定义的以下常量指定：

+   `PDO::ERRMODE_SILENT` - *静默*策略。

+   `PDO::ERRMODE_WARNING` - *警告*策略。

+   `PDO::ERRMODE_EXCEPTION` - 使用*异常*。

要设置所需的错误处理模式，我们必须以以下方式设置`PDO::ATTR_ERRMODE`属性：

```php
$conn->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);

```

要查看 PDO 如何抛出异常，请在`common.inc.php`文件中编辑，在第 46 行后添加上述语句。如果您想测试当 PDO 抛出异常时会发生什么，请更改连接字符串以指定不存在的数据库。现在将浏览器指向图书列表页面。

您应该看到类似于以下的输出：

![PDO 中的错误处理类型](img/2660_03_01.jpg)

这是 PHP 对未捕获异常的默认反应——它们被视为致命错误，程序执行停止。错误消息显示了异常的类`PDOException`、错误描述和一些调试信息，包括抛出异常的语句的名称和行号。请注意，如果要测试 SQLite，指定不存在的数据库可能不起作用，因为如果数据库不存在，它将被创建。要查看它是否适用于 SQLite，请更改第 10 行的`$connStr`变量，以便数据库名称中有一个非法字符：

```php
$connStr = 'sqlite:/path/to/pdo*.db';

```

刷新您的浏览器，您应该看到类似于这样的内容：

![PDO 中的错误处理类型](img/2660_03_02.jpg)

如您所见，显示了类似于上一个示例的消息，指定了错误的原因和源代码中的位置。

# 定义错误处理函数

如果我们知道某个语句或代码块可能会抛出异常，我们应该将该代码包装在*try...catch*块中，以防止显示默认错误消息并呈现用户友好的错误页面。但在我们继续之前，让我们创建一个函数，用于呈现错误消息并退出应用程序。由于我们将从不同的脚本文件中调用它，所以最好的地方就是`common.inc.php`文件。

我们的函数，名为`showError()`，将执行以下操作：

+   呈现一个标题，写着“错误”。

+   呈现错误消息。我们将使用`htmlspecialchars()`函数转义文本，并使用`nl2br()`函数处理它，以便我们可以显示多行消息。（此函数将所有换行字符转换为`<br>`标签。）

+   调用`showFooter()`函数来关闭打开的`<html>`和`<body>`标签。该函数将假定应用程序已经调用了`showHeader()`函数。（否则，我们将得到破损的 HTML。）

我们还必须修改在`common.inc.php`中创建连接对象的块，以捕获可能的异常。通过所有这些更改，`common.inc.php`的新版本将如下所示：

```php
<?php
/**
* This is a common include file
* PDO Library Management example application
* @author Dennis Popel
*/
// DB connection string and username/password
$connStr = 'mysql:host=localhost;dbname=pdo';
$user = 'root';
$pass = 'root';
/**
* This function will render the header on every page,
* including the opening html tag,
* the head section and the opening body tag.
* It should be called before any output of the
* page itself.
* @param string $title the page title
*/
function showHeader($title)
{
?>
<html>
<head><title><?=htmlspecialchars($title)?></title></head>
<body>
<h1><?=htmlspecialchars($title)?></h1>
<a href="books.php">Books</a>
<a href="authors.php">Authors</a>
<hr>
<?php
}
/**
* This function will 'close' the body and html
* tags opened by the showHeader() function
*/
function showFooter()
{
?>
</body>
</html>
<?php
}
**/**
* This function will display an error message, call the
* showFooter() function and terminate the application
* @param string $message the error message
*/
function showError($message)
{
echo "<h2>Error</h2>";
echo nl2br(htmlspecialchars($message));
showFooter();
exit();
}
// Create the connection object
try
{
$conn = new PDO($connStr, $user, $pass);
$conn->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
}
catch(PDOException $e)
{
showHeader('Error');
showError("Sorry, an error has occurred. Please try your request
later\n" . $e->getMessage());
}**

```

正如您所看到的，新创建的函数非常简单。更有趣的部分是我们用来捕获异常的*try…catch*块。现在通过这些修改，我们可以测试真正的异常将如何被处理。为此，请确保您的连接字符串是错误的（这样它就为 MySQL 指定了错误的数据库名称，或者包含了 SQLite 的无效文件名）。将浏览器指向`books.php`，您应该会看到以下窗口：

![定义错误处理函数](img/2660_03_03.jpg)

# 创建编辑书籍页面

正如我们之前讨论的，我们希望扩展我们的应用程序，以便我们可以添加和编辑书籍和作者。此外，我们的系统应该能够通过在书籍表的`ISBN`列上强制执行唯一索引来防止我们输入相同的书籍两次。

在继续进行代码之前，我们将创建索引。启动您的命令行客户端，并输入以下命令（对于 MySQL 和 SQLite 是相同的）：

```php
CREATE UNIQUE INDEX idx_isbn ON books(isbn);

```

我们还将使我们的编辑书籍页面同时具有两个目的——添加新书和编辑现有书籍。脚本将通过书籍 ID 的存在来区分要采取的操作，无论是在 URL 中还是在隐藏的表单字段中。我们将从`books.php`中链接到这个新页面，这样我们就可以通过在书籍列表页面上点击链接来编辑每一本书。

这个页面比上一章描述的页面更复杂，所以我会先给你代码，然后再讨论它。让我们称这个页面为 edit `Book.php:`

```php
<?php
/**
* This page allows to add or edit a book
* PDO Library Management example application
* @author Dennis Popel
*/
// Don't forget the include
include('common.inc.php');
// See if we have the book ID passed in the request
$id = (int)$_REQUEST['book'];
if($id) {
// We have the ID, get the book details from the table
$q = $conn->query("SELECT * FROM books WHERE id=$id");
$book = $q->fetch(PDO::FETCH_ASSOC);
$q->closeCursor();
$q = null;
}
else {
// We are creating a new book
$book = array();
}
// Now get the list of all authors' first and last names
// We will need it to create the dropdown box for author
$authors = array();
$q = $conn->query("SELECT id, lastName, firstName FROM authors ORDER
BY lastName, firstName");
$q->setFetchMode(PDO::FETCH_ASSOC);
while($a = $q->fetch())
{
$authors[$a['id']] = "$a[lastName], $a[firstName]";
}
// Now see if the form was submitted
if($_POST['submit']) {
// Validate every field
$warnings = array();
// Title should be non-empty
if(!$_POST['title'])
{
$warnings[] = 'Please enter book title';
}
// Author should be a key in the $authors array
if(!array_key_exists($_POST['author'], $authors))
{
$warnings[] = 'Please select author for the book';
}
// ISBN should be a 10-digit number
if(!preg_match('~^\d{10}$~', $_POST['isbn'])) {
$warnings[] = 'ISBN should be 10 digits';
}
// Published should be non-empty
if(!$_POST['publisher']) {
$warnings[] = 'Please enter publisher';
}
// Year should be 4 digits
if(!preg_match('~^\d{4}$~', $_POST['year'])) {
$warnings[] = 'Year should be 4 digits';
}
// Sumary should be non-empty
if(!$_POST['summary']) {
$warnings[] = 'Please enter summary';
}
// If there are no errors, we can update the database
// If there was book ID passed, update that book
if(count($warnings) == 0) {
if(@$book['id']) {
$sql = "UPDATE books SET title=" . $conn>quote($_POST['title']) .
', author=' . $conn->quote($_POST['author']) .
', isbn=' . $conn->quote($_POST['isbn']) .
', publisher=' . $conn->quote($_POST['publisher']) .
', year=' . $conn->quote($_POST['year']) .
', summary=' . $conn->quote($_POST['summary']) .
" WHERE id=$book[id]";
}
else {
$sql = "INSERT INTO books(title, author, isbn, publisher,
year,summary) VALUES(" .
$conn->quote($_POST['title']) .
', ' . $conn->quote($_POST['author']) .
', ' . $conn->quote($_POST['isbn']) .
', ' . $conn->quote($_POST['publisher']) .
', ' . $conn->quote($_POST['year']) .
', ' . $conn->quote($_POST['summary']) .
')';
}
// Now we are updating the DB.
// We wrap this into a try/catch block
// as an exception can get thrown if
// the ISBN is already in the table
try
{
$conn->query($sql);
// If we are here that means that no error
// We can return back to books listing
header("Location: books.php");
exit;
}
catch(PDOException $e)
{
$warnings[] = 'Duplicate ISBN entered. Please correct';
}
}
}
else {
// Form was not submitted.
// Populate the $_POST array with the book's details
$_POST = $book;
}
// Display the header
showHeader('Edit Book');
// If we have any warnings, display them now
if(count($warnings)) {
echo "<b>Please correct these errors:</b><br>";
foreach($warnings as $w)
{
echo "- ", htmlspecialchars($w), "<br>";
}
}
// Now display the form
?>
<form action="editBook.php" method="post">
<table border="1" cellpadding="3">
<tr>
<td>Title</td>
<td>
<input type="text" name="title"
value="<?=htmlspecialchars($_POST['title'])?>">
</td>
</tr>
<tr>
<td>Author</td>
<td>
<select name="author">
<option value="">Please select...</option>
<?php foreach($authors as $id=>$author) { ?>
<option value="<?=$id?>"
<?= $id == $_POST['author'] ? 'selected' : ''?>>
<?=htmlspecialchars($author)?>
</option>
<?php } ?>
</select>
</td>
</tr>
<tr>
<td>ISBN</td>
<td>
<input type="text" name="isbn"
value="<?=htmlspecialchars($_POST['isbn'])?>">
</td>
</tr>
<tr>
<td>Publisher</td>
<td>
<input type="text" name="publisher"
value="<?=htmlspecialchars($_POST['publisher'])?>">
</td>
</tr>
<tr>
<td>Year</td>
<td>
<input type="text" name="year"
value="<?=htmlspecialchars($_POST['year'])?>">
</td>
</tr>
<tr>
<td>Summary</td>
<td>
<textarea name="summary"><?=htmlspecialchars( $_POST['summary'])?></textarea>
</td>
</tr>
<tr>
<td colspan="2" align="center">
<input type="submit" name="submit" value="Save">
</td>
</tr>
</table>
<?php if(@$book['id']) { ?>
<input type="hidden" name="book" value="<?=$book['id']?>">
<?php } ?>
</form>
<?php
// Display footer
showFooter();

```

代码相当自我解释，但让我们简要地浏览一下它的主要部分。12 到 23 行处理如果页面使用书籍 ID 请求，则会获取要编辑的书籍详情。这些细节存储在`$book`变量中。请注意，我们明确将请求参数`book`转换为`整数`，以便不会发生 SQL 注入（第 13 行）。如果没有提供书籍 ID，则将其设置为空数组。请注意我们如何调用`closeCursor()`函数，然后将`$q`变量赋值为 null。这是必要的，因为我们将重用连接对象。

26 到 33 行准备作者列表。由于我们的系统每本书只允许一个作者，我们将创建一个选择框字段列出所有作者。

35 行检查是否提交了表单。如果测试成功，脚本将验证每个字段（37 到 68 行）。每个失败的验证都会附加到警告列表中。（`$warnings`变量初始化为空数组。）我们将使用此列表来查看验证是否成功，并在验证失败时存储错误消息。

69 到 94 行构建实际的更新 SQL。最终的 SQL 取决于我们是在更新书籍（当`$book`数组包含**id**键时），还是添加新书。请注意，在查询执行之前如何引用每个列值。

95 到 112 行尝试执行查询。如果用户输入了重复的 ISBN，查询可能会失败，因此我们将代码包装在`try…catch`块中。如果确实抛出了异常，`catch`块将向`$warnings`数组附加相应的警告。如果一切正常且没有错误，脚本将重定向到书籍列表页面，您应该能看到更改。

113 到 118 行在表单没有提交时执行。在这里，`$_POST`数组被`$books`变量的内容填充。我们这样做是因为我们将使用`$_POST`数组在代码后面显示表单字段的值。

请注意我们如何在 122 到 129 行显示错误消息（如果有的话），以及在 141 到 154 行显示选择框。 （我们正在浏览所有作者，如果作者的 ID 与此书作者的 ID 匹配，则将该作者标记为选定的选项。）此外，其他表单字段是使用`htmlspecialchars（）`函数应用于`$_POST`数组的项目来呈现的。 189 到 191 行将向表单添加一个包含当前编辑的书籍的 ID 的隐藏字段（如果有的话）。

现代 Web 应用程序除了对用户提供的数据进行服务器端验证外，还采用了客户端验证。虽然这不在本书的范围内，但您可能会考虑在项目中使用基于浏览器的验证，以增加响应性并可能减少 Web 服务器的负载。

现在，我们应该从`books.php`页面链接到新创建的页面。我们将为每个列出的书籍提供一个*编辑此书*链接，以及在表格下方提供一个*添加书籍*链接。我不会在这里重复整个`books.php`源代码，只是应该更改的行。因此，应该将 32 到 48 行替换为以下内容：

```php
<?php
// Now iterate over every row and display it
while($r = $q->fetch())
{
?>
<tr>
<td><ahref="author.php?id=<?=$r['authorId']?>">
<?=htmlspecialchars("$r[firstName] $r[lastName]")?></a></td>
<td><?=htmlspecialchars($r['title'])?></td>
<td><?=htmlspecialchars($r['isbn'])?></td>
<td><?=htmlspecialchars($r['publisher'])?></td>
<td><?=htmlspecialchars($r['year'])?></td>
<td><?=htmlspecialchars($r['summary'])?></td>
**<td>
<a href="editBook.php?book=<?=$r['id']?>">Edit</a>
</td>**
</tr>
<?php
}
?>

```

应该在调用`showFooter（）`函数之前添加以下内容，以便这四行看起来像这样：

```php
<a href="editBook.php">Add book...</a>
<?php
// Display footer
showFooter();

```

现在，如果您再次导航到`books.php`页面，您应该看到以下窗口：

![创建编辑书籍页面](img/2660_03_04.jpg)

要查看我们的编辑书籍页面的外观，请单击表格最后一列中的任何**编辑**链接。您应该看到以下表单：

![创建编辑书籍页面](img/2660_03_05.jpg)

让我们看看我们的表单是如何工作的。它正在验证发送到数据库的每个表单字段。如果有任何验证错误，表单将不会更新数据库，并提示用户更正提交。例如，尝试将作者选择框更改为默认选项（标有*请选择...*），并将 ISBN 编辑为 5 位数。

如果单击**保存**按钮，您应该看到表单显示以下错误消息：

![创建编辑书籍页面](img/2660_03_06.jpg)

现在纠正错误，并尝试将 ISBN 更改为 1904811027。这个 ISBN 已经在我们的数据库中被另一本书使用，所以表单将再次显示错误。您还可以通过添加一本书来进一步测试表单。您可能还想测试它在 SQLite 中的工作方式。

# 创建编辑作者页面

我们的应用程序仍然缺少添加/编辑作者功能。这个页面将比编辑书籍页面简单一些，因为它不会有作者的选择框和唯一索引。（您可能希望在作者的名字和姓氏列上创建唯一索引，以防止那里也出现重复，但我们将把这个问题留给您。）

让我们称这个页面为`editAuthor.php`。以下是它的源代码：

```php
<?php
/**
* This page allows to add or edit an author
* PDO Library Management example application
* @author Dennis Popel
*/
// Don't forget the include
include('common.inc.php');
// See if we have the author ID passed in the request
$id = (int)$_REQUEST['author'];
if($id) {
// We have the ID, get the author details from the table
$q = $conn->query("SELECT * FROM authors WHERE id=$id");
$author = $q->fetch(PDO::FETCH_ASSOC);
$q->closeCursor();
$q = null;
}
else {
// We are creating a new book
$author = array();
}
// Now see if the form was submitted
if($_POST['submit']) {
// Validate every field
$warnings = array();
// First name should be non-empty
if(!$_POST['firstName']) {
$warnings[] = 'Please enter first name';
}
// Last name should be non-empty
if(!$_POST['lastName']) {
$warnings[] = 'Please enter last name';
}
// Bio should be non-empty
if(!$_POST['bio']) {
$warnings[] = 'Please enter bio';
}
// If there are no errors, we can update the database
// If there was book ID passed, update that book
if(count($warnings) == 0) {
if(@$author['id']) {
$sql = "UPDATE authors SET firstName=" .
$co>quote($_POST['firstName']) .
', lastName=' . $conn->quote($_POST['lastName']) .
', bio=' . $conn->quote($_POST['bio']) .
" WHERE id=$author[id]";
}
else {
$sql = "INSERT INTO authors(firstName, lastName, bio) VALUES(" .
$conn->quote($_POST['firstName']) .
', ' . $conn->quote($_POST['lastName']) .
', ' . $conn->quote($_POST['bio']) .
')';
}
$conn->query($sql);
header("Location: authors.php");
exit;
}
}
else {
// Form was not submitted.
// Populate the $_POST array with the author's details
$_POST = $author;
}
// Display the header
showHeader('Edit Author');
// If we have any warnings, display them now
if(count($warnings)) {
echo "<b>Please correct these errors:</b><br>";
foreach($warnings as $w)
{
echo "- ", htmlspecialchars($w), "<br>";
}
}
// Now display the form
?>
<form action="editAuthor.php" method="post">
<table border="1" cellpadding="3">
<tr>
<td>First name</td>
<td>
<input type="text" name="firstName"
value="<?=htmlspecialchars($_POST['firstName'])?>">
</td>
</tr>
<tr>
<td>Last name</td>
<td>
<input type="text" name="lastName"
value="<?=htmlspecialchars($_POST['lastName'])?>">
</td>
</tr>
<tr>
<td>Bio</td>
<td>
<textarea name="bio"><?=htmlspecialchars($_POST['bio'])?>
</textarea>
</td>
</tr>
<tr>
<td colspan="2" align="center">
<input type="submit" name="submit" value="Save">
</td>
</tr>
</table>
<?php if(@$author['id']) { ?>
<input type="hidden" name="author" value="<?=$author['id']?>">
<?php } ?>
</form>
<?php
// Display footer
showFooter();

```

此源代码与`editBook.php`页面以相同的方式构建，因此您应该能够轻松地跟随它。

我们将以与我们从`books.php`页面链接到`editBook.php`页面相同的方式链接到`editAuthors.php`页面。编辑`authors.php`文件，并将 30-41 行更改为以下内容：

```php
while($r = $q->fetch(PDO::FETCH_ASSOC))
{
?>
<tr>
<td><?=htmlspecialchars($r['firstName'])?></td>
<td><?=htmlspecialchars($r['lastName'])?></td>
<td><?=htmlspecialchars($r['bio'])?></td>
**<td>
<a href="editAuthor.php?author=<?=$r['id']?>">Edit</a>
</td>**
</tr>
<?php
}

```

在最后一个 PHP 块之前添加以下行：

```php
<a href="editAuthor.php">Add Author...</a>

```

现在，如果您刷新`authors.php`页面，您将看到以下内容：

![创建编辑作者页面](img/2660_03_07.jpg)

您可以单击右侧列中的**编辑**链接来编辑每位作者的详细信息。您可以尝试使用空值提交表单，以查看无效提交将被拒绝。此外，您可以尝试向系统添加新作者。成功完成后，您可能希望返回到书籍列表并编辑一些书籍。您将看到新创建的作者可用于**作者**选择框。

# 防止未捕获异常

正如我们之前所看到的，我们在可能引发异常的代码周围放置了*try...catch*块。然而，在非常罕见的情况下，可能会出现一些意外的异常。我们可以通过修改其中一个查询来模拟这样的异常，使其包含一些格式不正确的 SQL。例如，让我们编辑`authors.php`，将第 16 行修改为以下内容：

```php
$q = $conn->query("SELECT * FROM authors ORDER BY lastName, firstName");

```

现在尝试使用浏览器导航到`authors.php`，看看是否发生了未捕获的异常。为了正确处理这种情况，我们要么创建一个异常处理程序，要么将调用`PDO`或`PDOStatement`类方法的每个代码块包装在*try...catch*块中。

让我们看看如何创建异常处理程序。这是一种更简单的方法，因为它不需要改变大量的代码。然而，对于大型应用程序来说，这可能是一个不好的做法，因为在发生异常的地方处理异常可能更安全，并且可以应用更好的恢复逻辑。

然而，对于我们简单的应用程序，我们可以使用全局异常处理程序。它将只是使用`showError()`函数来表示网站正在维护中。

```php
/**
* This is the default exception handler
* @param Exception $e the uncaught exception
*/
function exceptionHandler($e)
{
showError("Sorry, the site is under maintenance\n" .
$e->getMessage());
}
// Set the global excpetion handler
set_exception_handler('exceptionHandler');

```

将这段代码放入`common.inc.php`中，就在连接创建代码块之前。如果现在刷新`authors.php`页面，你会看到处理程序被调用了。

拥有默认的异常处理程序总是一个好主意。正如你已经注意到的，未处理的异常会暴露太多敏感信息，包括数据库连接详细信息。此外，在真实世界的应用程序中，错误页面不应显示有关错误类型的任何信息。（请注意，我们的示例应用程序是这样的。）默认处理程序应该写入错误日志，并通知网站维护人员有关错误的信息。

# 总结

在本章中，我们研究了`PDO`如何处理错误，并介绍了异常。此外，我们调查了错误的来源，并看到了如何对抗它们。

我们的示例应用程序已经扩展了一些真实世界的管理功能，使用了数据验证，并且受到了 SQL 注入攻击的保护。当然，他们还应该只允许基于登录名和密码的特定用户对数据库进行修改。然而，这超出了本书的范围。

在下一章中，我们将看到 PDO 和数据库编程中另一个非常重要的方面——使用预处理语句。我们将看到如何借助它们来简化我们的管理页面，从而减少代码量并提高维护性。
