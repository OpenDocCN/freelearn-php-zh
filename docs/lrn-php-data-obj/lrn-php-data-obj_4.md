# 第四章：准备好的语句

在前几章中，我们已经了解了 PDO 的基础知识，您可能已经注意到它的大部分功能类似于用于连接数据库的传统扩展。唯一的新功能是异常，但即使这一点也可以类似于传统的错误处理。

在本章中，我们将看到 PHP 中 PDO 之前不存在的一个新概念：准备好的语句。我们将看到它们如何进一步简化我们的代码，甚至提高性能。我们还将看看 PDO 如何以数据库无关的方式处理 BLOBs。

关于我们的图书馆管理应用程序，我们将重写前一章中添加的编辑/更新功能，以便支持准备好的语句，并添加对书籍封面图片的支持，我们将保存在数据库中。

# 准备好的语句

**准备好的语句**是针对数据库执行一个或多个 SQL 查询的模板。准备好的语句的理念是，对于使用相同语法但不同值的查询，预处理语法一次，然后使用不同参数多次执行会更快。考虑以下任务。我们必须将几位新作者的姓名插入到我们的数据库中。当然，我们可以使用命令行客户端或我们最近创建的`add author`页面，但我们决定使用一个 PHP 脚本。

假设要添加的作者保存在一个 PHP 数组中：

```php
$authors = array(
array(
'firstName' => 'Alexander',
'lastName' => 'Dumas',
'bio' => 'Alexandre Dumas was a French writer, best known for his
numerous historical novels of high adventure which have
made him one of the most widely read French authors in
the world.'),
array(
'firstName' => 'Ivan',
'lastName' => 'Franko',
'bio' => 'Ivan Franko was a Ukrainian poet, writer, social and
literary critic, and journalist. In addition to his own
literary work, he translated the works of William
Shakespeare, Lord Byron, Dante, Victor Hugo, Goethe and
Schiller into the Ukrainian language.'));

```

这是一个二维数组，我们将使用`foreach`循环来迭代，以便将两位作者的详细信息插入到数据库中。

```php
foreach($authors as $author)
{
$conn->query(
'INSERT INTO authors(firstName, lastName, bio) VALUES(' .
$conn->quote($author['firstName']) .
',' . $conn->quote($author['lastName']) .
',' . $conn->quote($author['bio'])')' .
);
}

```

正如您所看到的，我们在每次迭代中为每个作者创建一个 SQL 语句，并引用所有参数。

使用准备好的语句，我们可以只构建一次查询，然后通过传递不同的值任意次数执行它。我们的代码将如下所示：

```php
$stmt = $conn->prepare('INSERT INTO authors(firstName, lastName, bio)
VALUES(?, ?, ?)');
foreach($authors as $author)
{
$stmt->execute(
array($author['firstName'], $author['lastName'],
$author['bio']));
}

```

从上面的代码片段中，您可以看到准备好的语句首先通过调用`PDO::prepare()`方法*准备*。该方法接受一个包含 SQL 命令的字符串，其中变化的值被问号字符替换。调用返回一个`PDOStatement`类的对象。然后在循环中，我们调用语句的`execute()`方法，而不是`PDO::query()`方法。

`PDOStatement::execute()`方法接受一个值数组，这些值将被插入到 SQL 查询中，取代问号。该数组中的元素数量和顺序必须与传递给`PDO::prepare()`的查询模板中问号的数量和顺序相同。

您一定注意到我们在代码中没有使用`PDO::quote()`——PDO 会正确引用传入的值。

## 位置和命名占位符

前面的例子使用问号来指定准备好的语句中数值的位置。这就是为什么这些问号被称为**位置占位符**。当使用它们时，您必须注意传递给`PDOStatement::execute()`方法的数组中元素的正确顺序。虽然它们写起来很快，但当您更改查询列时，它们可能成为难以跟踪错误的源泉。为了保护自己免受这种影响，您可以使用所谓的**命名占位符**，它们由冒号前面的描述性名称组成，而不是问号。

使用命名占位符，我们可以以以下方式重写代码来插入这两位作者：

```php
$stmt = $conn->prepare(
'INSERT INTO authors(firstName, lastName, bio) ' .
'VALUES(:first, :last, :bio)');
foreach($authors as $author)
{
$stmt->execute(
array(
':first' => $author['firstName'],
':last' => $author['lastName'],
':bio' => $author['bio'])
);
}

```

正如您所看到的，我们用命名占位符替换了三个问号，然后在调用`PDOStatement::execute()`时，我们提供了一个键值对数组，其中键是相应的命名占位符，值是我们要插入数据库的数据。

使用命名占位符时，数组中元素的顺序并不重要，只有关联才重要。例如，我们可以将循环重写如下：

```php
foreach($authors as $author)
{
$stmt->execute(
array(
':bio' => $author['bio'],
':last' => $author['lastName'],
':first' => $author['firstName'])
);
}

```

然而，对于位置占位符，只要我们确保其元素的顺序与占位符的顺序匹配，就可以将`$author`数组的值传递给`PDOStatement::execute()`方法：

```php
$stmt = $conn->prepare(
'INSERT INTO authors(firstName, lastName, bio) VALUES(?, ?, ?)');
foreach($authors as $author)
{
$stmt->execute(array_values($author));
}

```

请注意我们如何使用`array_values()`函数来摆脱字符串键并将关联数组转换为列表。

如果我们向`PDOStatement::execute()`提供的值数组与查询中的占位符数量不匹配，或者我们向使用位置占位符的语句传递了一个关联数组（或向使用命名占位符的语句传递了一个列表），这将被视为错误，并且将抛出异常（前提是之前在调用`PDO::setAttribute()`方法中启用了异常）。

关于占位符的使用有一件重要的事情需要注意。它们不能作为您传递给数据库的值的一部分。这最好通过一个无效使用示例来演示：

```php
$stmt = $conn->prepare("SELECT * FROM authors WHERE lastName
LIKE '%?%'");
$stmt->execute(array($_GET['name']));

```

这必须重写为：

```php
$stmt = $conn->prepare("SELECT * FROM authors WHERE lastName
LIKE ?");
$stmt->execute(array('%' . $_GET['name'] . '%'));

```

这里的想法是，不要将占位符放在 SQL 模板中的字符串中——这必须在调用`PDOStatement::execute()`方法中完成。

## 准备语句和绑定值

上面的示例使用了所谓的**未绑定语句**。这意味着我们在传递给`PDOStatement::execute()`方法的数组中提供了查询的值。PDO 还支持**绑定语句**，其中您可以将立即值或变量显式绑定到命名或位置占位符。

要将立即值绑定到语句，使用`PDOStatement::bindValue()`方法。此方法接受占位符标识符和一个值。占位符标识符是查询中位置占位符的问号的基于 1 的索引，或命名占位符的名称。例如，我们可以将使用位置占位符的示例重写为以下方式使用绑定值：

```php
$stmt = $conn->prepare(
'INSERT INTO authors(firstName, lastName, bio) VALUES(?, ?, ?)');
foreach($authors as $author)
{
$stmt->bindValue(1, $author['firstName']);
$stmt->bindValue(2, $author['lastName']);
$stmt->bindValue(3, $author['bio']);
$stmt->execute();
}

```

如果您喜欢使用命名占位符，可以编写：

```php
$stmt = $conn->prepare(
'INSERT INTO authors(firstName, lastName, bio) ' .
'VALUES(:last, :first, :bio)');
foreach($authors as $author)
{
$stmt->bindValue(':first', $author['firstName']);
$stmt->bindValue(':last', $author['lastName']);
$stmt->bindValue(':bio', $author['bio']);
$stmt->execute();
}

```

如您所见，在这两种情况下，我们在调用`PDOStatement::execute()`时不提供任何内容。同样，与未绑定语句一样，如果您没有为每个占位符绑定值，调用`PDOStatement::execute()`将失败，导致异常。

PDO 也可以将结果集列绑定到 PHP 变量以用于 SELECT 查询。这些变量将在每次调用`PDOStatement::fetch()`时被相应列的值修改。这是在第二章中讨论的将结果集行作为数组或对象获取的替代方法。考虑以下示例：

```php
$stmt = $conn->prepare('SELECT firstName, lastName FROM authors');
$stmt->execute();
$stmt->bindColumn(1, $first);
$stmt->bindColumn(2, $last);
while($stmt->fetch(PDO::FETCH_BOUND))
{
echo "$last, $first <br>";
}

```

这将呈现表中的所有作者。变量在调用`PDOStatement::bindColumn()`方法时绑定，该方法期望第一个参数是结果集中的列的基于 1 的索引或从数据库返回的列名，第二个参数是要更新的变量。

请注意，当使用绑定列时，应使用`PDO::FETCH_BOUND`模式调用`PDOStatement::fetch()`方法，或者应该在调用`PDOStatement::setFetchMode(PDO::FETCH_BOUND)`之前进行预设。此外，必须在调用`PDOStatement::execute()`方法之后调用`PDOStatement::bindColumn()`方法，以便 PDO 知道结果集中有多少列。

现在让我们回到我们的图书馆应用程序，并增强它以使用一些预处理语句。由于仅依赖用户提供的值的页面是*添加/编辑书籍*和*添加/编辑作者*，我们将重写两个相应的脚本，`editBook.php`和`editAuthor.php`。

当然，我们只会重写更新数据库的代码部分。对于`editBook.php`，这些是第 65 到 102 行。我将在这里为您方便起见呈现这些行：

```php
if(@$book['id']) {
$sql = "UPDATE books SET title=" . $conn->quote($_POST['title']) .
', author=' . $conn->quote($_POST['author']) .
', isbn=' . $conn->quote($_POST['isbn']) .
', publisher=' . $conn->quote($_POST['publisher']) .
', year=' . $conn->quote($_POST['year']) .
', summary=' . $conn->quote($_POST['summary']) .
" WHERE id=$book[id]";
}
else {
$sql = "INSERT INTO books(title, author, isbn, publisher, year,
summary) VALUES(" . $conn->quote($_POST['title']) .
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
// the ISBN is already in the table.
try
{
$conn->query($sql);
// If we are here, then there is no error.
// We can return back to books listing
header("Location: books.php");
exit;
}
catch(PDOException $e)
{
$warnings[] = 'Duplicate ISBN entered. Please correct';
}

```

正如我们所看到的，构造查询的部分非常长。使用预处理语句，可以将此代码片段重写如下：

```php
if(@$book['id']) {
$sql = "UPDATE books SET title=?, author=?, isbn=?, publisher=?
year=?, summary=? WHERE id=$book[id]";
}
else {
$sql = "INSERT INTO books(title, author, isbn, publisher, year,
summary) VALUES(?, ?, ?, ?, ?, ?)";
}
$stmt = $conn->prepare($sql);
// Now we are updating the DB.
// We wrap this into a try/catch block
// as an exception can get thrown if
// the ISBN is already in the table.
try
{
$stmt->execute(array($_POST['title'], $_POST['author'],
$_POST['isbn'], $_POST['publisher'], $_POST['year'],
$_POST['summary']));
// If we are here, then there is no error.
// We can return back to books listing.
header("Location: books.php");
exit;
}
catch(PDOException $e)
{
$warnings[] = 'Duplicate ISBN entered. Please correct';
}

```

我们遵循相同的逻辑 - 如果我们正在编辑现有书籍，我们构建一个`UPDATE`查询。如果我们要添加新书，那么我们必须使用`INSERT`查询。`$sql`变量将保存适当的语句模板。在这两种情况下，语句都有六个位置占位符，我故意将书籍 ID 硬编码到`UPDATE`查询中，以便我们可以创建并执行语句，而不管所需的操作是什么。

在我们实例化语句之后，我们将其`execute()`方法的调用包装在*try…catch*块中，因为如果 ISBN 已经存在于数据库中，可能会抛出异常。在语句成功执行后，我们将浏览器重定向到书籍列表页面。如果调用失败，我们会用一个提示通知用户 ISBN 不正确（或者书籍已经存在于数据库中）。

您可以看到我们的代码现在要短得多。此外，我们不需要引用值，因为准备好的语句已经为我们做了这个。现在您可以稍微玩弄一下，并在`common.inc.php`中将数据库更改为 MySQL 和 SQLite，以查看准备好的语句是否适用于它们两个。您可能还想重写此代码，以使用命名占位符而不是位置占位符。如果这样做，请记住在传递给`PDOStatement::execute()`方法的数组中提供占位符名称。

现在让我们看看`editAuthor.php`中的相应代码块（第 42 至 59 行）：

```php
if(@$author['id']) {
$sql = "UPDATE authors SET firstName=" .
$conn->quote($_POST['firstName']) .
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

```

由于我们不希望在这里出现异常，所以代码更短。现在让我们重写它以使用准备好的语句：

```php
if(@$author['id']) {
$sql = "UPDATE authors SET firstName=?, lastName=?, bio=?
WHERE id=$author[id]";
}
else {
$sql = "INSERT INTO authors(firstName, lastName, bio)
VALUES(?, ?, ?)";
}
$stmt = $conn->prepare($sql);
$stmt->execute(array($_POST['firstName'], $_POST['lastName'],
$_POST['bio']));
header("Location: authors.php");
exit;

```

再次取决于所需的操作，我们创建 SQL 模板并将其分配给`$sql`变量。然后我们实例化`PDOStatement`对象，并使用作者的详细信息调用其`execute`方法。由于我们的查询不应该失败（除非出现意外的数据库故障），我们不希望在这里出现异常，并重定向到作者列表页面。

确保您使用 MySQL 和 SQLite 测试此代码。

# 使用 BLOBs

现在让我们扩展我们的应用程序，以便我们可以上传书籍的封面图片并显示它们。与传统的数据库访问一样，我们将在书籍表中使用**BLOB 字段**，以及一个**varchar 字段**来存储图像的 MIME 类型，我们需要将其与图像数据一起提供给浏览器。此外，我们还需要另一个脚本，它将从表中获取图像数据并将其传递给浏览器。（我们将从`<img>`标签中引用此脚本。）

传统上，我们不会在对`mysql_query()`或`sqlite_query()`的调用中插入 BLOB 列 - 我们只需确保它们被正确引用。但是，使用 PDO，情况就不同了。PDO 通过流和准备好的语句处理 BLOB 列。

让我们看看以下示例：

```php
$blob = fopen('/path/to/file.jpg', 'rb');
$stmt = $conn->prepare("INSERT INTO images(data) VALUES(?)");
$stmt->bindParam(1, $blob, PDO::PARAM_LOB);
$stmt->execute();

```

正如您所看到的，我们使用`fopen()`函数以二进制模式打开要插入的文件（这样我们就不会在不同平台上遇到换行符的问题），然后在调用`PDOStatement::bindParam()`方法时将文件句柄绑定到语句，并指定`PDO::PARAM_LOB`标志（以便 PDO 了解我们绑定的是文件句柄而不是立即值）。

在对`PDOStatement::execute()`方法的调用中，PDO 将从文件中读取数据并将其传递给数据库。

### 注意

如果您想知道为什么 PDO 以这种方式工作，简短的解释是，如果您的 BLOB 非常大，查询可能会失败。通常数据库服务器有一个限制通信数据包大小的设置。（您可以将其与`post_max_size`PHP 设置进行比较）。如果您在 SQL `INSERT`或`UPDATE`语句中传递相对较大的字符串，它可能会超过数据包大小，导致查询失败。使用流，PDO 确保数据以较小的数据包发送，以便查询成功执行。

BLOBs 也应该用流来读取。因此，要检索上面示例中插入的 BLOB 列，可以使用以下代码：

```php
$id = (int)$_GET['id'];
$stmt = $db->prepare("SELECT data FROM images WHERE id=$id");
$stmt->execute();
$stmt->bindColumn(1, $blob, PDO::PARAM_LOB);
$stmt->fetch(PDO::FETCH_BOUND);
$data = stream_get_contents($blob);

```

在这种情况下，`$blob`变量将是一个可以使用流处理函数读取的流资源。在这里，我们使用了`stream_get_contents()`函数将所有数据读入`$data`变量中。如果我们想直接将数据返回给浏览器（就像我们在应用程序中将要做的那样），我们可以使用`fpassthru()`函数。

截至目前（PHP 版本 5.2.3），返回的 blob 列不是流，而是列中包含的实际数据（字符串）。有关详细信息，请参阅 PHP bug＃40913 [`bugs.php.net/bug.php?id=40913`](http://bugs.php.net/bug.php?id=40913)。因此，上述代码片段中的最后一行是不需要的，`$blob`变量将保存实际数据。下面 showCover.php 文件的源代码将返回的数据视为字符串而不是 blob，因此代码可以在当前 PHP 版本中运行。

所以，让我们开始修改我们的数据库，并向其中添加新的列：

```php
mysql> alter table books add column coverMime varchar(20);
Query OK, 3 rows affected (0.02 sec)
Records: 3 Duplicates: 0 Warnings: 0
mysql> alter table books add column coverImage blob(24000);
Query OK, 3 rows affected (0.02 sec)
Records: 3 Duplicates: 0 Warnings: 0

```

您还可以在 SQLite 命令行客户端中执行这些查询，无需修改。现在，让我们修改`editBook.php`文件。我们将在现有表单中添加另一个字段。这行将允许用户上传封面图片，并增强表单验证以检查用户是否真的上传了一张图片（通过检查上传文件的 MIME 类型）。

我们还将允许用户在不重新提交封面图片文件的情况下修改书籍的详细信息。为此，我们将仅在成功上传文件时更新封面列。因此，我们的脚本逻辑将使用两个查询。第一个将更新或创建书籍记录，第二个将更新`coverMime`和`coverImage`列。

考虑到这一点，`editBook.php`文件将如下所示：

```php
<?php
/**
* This page allows adding or editing a book
* PDO Library Management example application
* @author Dennis Popel
*/
// Don't forget the include
include('common.inc.php');
// See if we have the book ID passed in the request
$id = (int)$_REQUEST['book'];
if($id) {
// we have the ID, get the book details from the table
$q = $conn->query("SELECT * FROM books WHERE id=$id");
$book = $q->fetch(PDO::FETCH_ASSOC);
$q->closeCursor();
$q = null;
}
else {
// we are creating a new book
$book = array();
}
// Now get the list of all authors' first and last names
// we will need it to create the dropdown box for author
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
if(!$_POST['title']) {
$warnings[] = 'Please enter book title';
}
// Author should be a key in the $authors array
if(!array_key_exists($_POST['author'], $authors)) {
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
// Summary should be non-empty
if(!$_POST['summary']) {
$warnings[] = 'Please enter summary';
}
**// Now validate the file upload
$uploadSuccess = false;
if(is_uploaded_file($_FILES['cover']['tmp_name'])) {
// See if the file is an image
if(!preg_match('~image/.+~', $_FILES['cover']['type'])
|| filesize($_FILES['cover']['tmp_name']) > 24000) {
$warnings[] = 'Please upload an image file less than 24K
in size';
}
else {
// Set a flag that upload is successful
$uploadSuccess = true;
}
}**
// If there are no errors, we can update the database
// If there was book ID passed, update that book
if(count($warnings) == 0) {
if(@$book['id']) {
$sql = "UPDATE books SET title=?, author=?, isbn=?,
publisher=?, year=?, summary=? WHERE
id=$book[id]";
}
else {
$sql = "INSERT INTO books(title, author, isbn, publisher,
year, summary) VALUES(?, ?, ?, ?, ?, ?)";
}
$stmt = $conn->prepare($sql);
// Now we are updating the DB.
// we wrap this into a try/catch block
// as an exception can get thrown if
// the ISBN is already in the table
try
{
$stmt->execute(array($_POST['title'], $_POST['author'],
$_POST['isbn'], $_POST['publisher'], $_POST['year'],
$_POST['summary']));
// If we are here that means that no error
**// Now we can update the cover columns
// But first we have to get the ID of the newly inserted book
if(!@$book['id']) {
$book['id'] = $conn->lastInsertId();
}
// Now see if there was an successful upload and
// update cover image
if($uploadSuccess) {
$stmt = $conn->prepare("UPDATE books SET coverMime=?,
coverImage=? WHERE id=$book[id]");
$cover = fopen($_FILES['cover']['tmp_name'], 'rb');
$stmt->bindValue(1, $_FILES['cover']['type']);
$stmt->bindParam(2, $cover, PDO::PARAM_LOB);
$stmt->execute();
}**
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
// populate the $_POST array with the book's details
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
**<form action="editBook.php" method="post"
enctype="multipart/form-data">**
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
<?php foreach($authors as $id=>$author)
{ ?>
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
<textareaname="summary"><?=htmlspecialchars($_POST['summary'])?>
</textarea>
</td>
</tr>
**<tr>
<td>Cover Image</td>
<td><input type="file" name="cover"></td>
</tr>
<?php if(@$book['coverMime'])
{ ?>
<tr>
<td>Current Cover</td>
<td><img src="showCover.php?book=<?=$book['id']?>"></td>
</tr>
<? } ?>**
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

突出显示的部分是我们添加或更改的部分。现在，我们需要验证我们的表单和上传的文件（第 60 到 73 行）。如果上传成功，`$uploadSuccess`布尔变量将设置为`true`，我们稍后将使用这个值来查看是否需要更新封面列。由于我们也允许新书进行上传，我们使用`PDO::lastInsertId()`方法值（在第 100 行）来获取新创建书籍的 ID（否则我们只使用`$books['id']`值）。如果上传失败，我们将向`$warnings`数组添加相应的警告，并让现有的错误逻辑执行其工作。

实际的封面图片更新发生在 105 到 110 行，使用了准备好的语句和流。在我们的表单中，看到我们如何在第 140 行的表单标签上添加了`multipart/form-data`属性。这是文件上传所必需的。此外，表单现在有一个新的输入字段（第 182-185 行），允许我们选择并上传文件。接下来的行将显示当前的封面图片（如果有的话）。请注意，`<img>`标签引用了一个新文件`showCover.php`，我们现在需要创建它：

```php
<?php
/**
* This script will render a book's cover image
* PDO Library Management example application
* @author Dennis Popel
*/
// Don't forget the include
include('common.inc.php');
// See if we have the book ID passed in the request
$id = (int)$_REQUEST['book'];
$stmt = $conn->prepare("SELECT coverMime, coverImage FROM books
WHERE id=$id");
$stmt->execute();
$stmt->bindColumn(1, $mime);
$stmt->bindColumn(2, $image, PDO::PARAM_LOB);
$stmt->fetch(PDO::FETCH_BOUND);
header("Content-Type: $mime");
echo $image;

```

现在，对于一本新书，表单看起来像这样：

![使用 BLOBs](img/2660_04_01.jpg)

如您所见，有一个新字段允许我们上传封面图片。由于新创建的书没有任何封面图片，因此没有当前的封面图片。对于有封面图片的书，页面将如下所示：

![使用 BLOBs](img/2660_04_02.jpg)

您现在可以使用应用程序来查看表单在不上传图片的情况下的工作方式。（如果有的话，它应该保留旧图片。）您还可以看到它如何处理过大或非图片文件。（它应该在表单上方显示警告。）确保在不同数据库之间切换，以便我们是数据库无关的。

作为封面图片的最后一步，我们可以重新格式化书籍列表页面`books.php`，以便在那里也显示封面图片。我将在这里呈现新代码，并突出显示更改的部分：

```php
<?php
/**
* This page lists all the books we have
* PDO Library Management example application
* @author Dennis Popel
*/
// Don't forget the include
include('common.inc.php');
// Display the header
showHeader('Books');
// Issue the query
$q = $conn->query("SELECT authors.id AS authorId, firstName,
lastName, books.* FROM authors, books WHERE
author=authors.id ORDER BY title");
$q->setFetchMode(PDO::FETCH_ASSOC);
// now create the table
?>
<table width="100%" border="1" cellpadding="3">
<tr style="font-weight: bold">
<td>Cover</td>
<td>Author and Title</td>
<td>ISBN</td>
<td>Publisher</td>
<td>Year</td>
<td>Summary</td>
<td>Edit</td>
</tr>
<?php
// Now iterate over every row and display it
while($r = $q->fetch())
{
?>
**<tr>
<td>
<?php if($r['coverMime']) { ?>
<img src="showCover.php?book=<?=$r['id']?>">
<?php }
else
{ ?>
n/a
<? } ?>
</td>
<td>
<a href="author.php?id=<?=$r['authorId']?>">
<?=htmlspecialchars("$r[firstName] $r[lastName]")?></a><br/>
<b><?=htmlspecialchars($r['title'])?></b>
</td>**
<td><?=htmlspecialchars($r['isbn'])?></td>
<td><?=htmlspecialchars($r['publisher'])?></td>
<td><?=htmlspecialchars($r['year'])?></td>
<td><?=htmlspecialchars($r['summary'])?></td>
<td>
<a href="editBook.php?book=<?=$r['id']?>">Edit</a>
</td>
</tr>
<?php
}
?>
</table>
<a href="editBook.php">Add book...</a>
<?php
// Display footer
showFooter();

```

第一个单元格将包含图片（如果有的话）。现在作者和标题都在同一个单元格中呈现，以节省表格宽度。现在图书列表应该看起来像这样：

![使用 BLOBs](img/2660_04_03.jpg)

# 摘要

本章向我们介绍了一个新概念：准备语句。我们已经看到它们如何简化我们的查询，并进一步保护我们免受 SQL 语法错误和代码漏洞的影响。我们还看了如何使用流处理 BLOBs，以便我们不会出现查询失败的风险。我们的应用现在可以用于上传和显示数据库中书籍的封面图片。

在下一章中，我们将看到如何确定结果集中的行数，这对于对长列表进行分页是必要的。（最常见的例子是搜索引擎将结果列表分成每页 10 个结果。）此外，我们将熟悉一个新概念：可滚动的游标，它将允许我们从指定位置开始获取结果集的子集行。
