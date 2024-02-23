# 第七章：一个高级示例

到目前为止，您应该能够使用 PDO 开发 Web 应用程序。但是，当应用程序保持相当小且功能有限时，我们的示例应用程序是可管理的。很快您将意识到，在一个文件中混合所有的数据访问、用户输入和显示逻辑可能会变得难以管理。

为了编写更易管理的代码，并允许多个开发人员共同开发项目，数据访问用户输入处理和页面呈现应该分开。您可能已经听说过广泛用于大型 Web 应用程序的**模型-视图-控制器**编程范式（MVC）。其思想是将数据访问和修改模块（即**模型**）与数据呈现（即**视图**）分开。视图可能非常复杂，因此通常使用模板引擎。最后，**控制器**是一个接收用户输入、访问模型并准备视图的 PHP 脚本。

除了使代码库更易管理外，这种划分还允许我们从其他应用程序（使用在应用程序自己的服务器上运行的维护脚本或在其他服务器上运行的脚本，通过 RPC 或 SOAP 调用访问）访问模型的功能。

由于 PDO 是面向对象的，并且可以从对`PDOStatement::fetch()`方法的调用中返回类的实例，因此我们将使用面向对象编程来模拟我们的数据实体（书籍、作者和借书记录）。

# 设计模型

模型通常由一个静态类组成（其方法被静态调用），以及模拟数据实体的几个类。对该模型类的方法的调用要么返回其他模型类的实例，要么返回`PDOStatement`实例，后者在调用`fetch()`方法时返回模型类的实例。

对于我们的应用程序，类将是`Model，Book，Author`和`Borrower`。这些类反映了我们示例数据库中的表，并允许我们对底层数据执行简单的操作。（主要思想是将 SQL 从控制器脚本中隔离到相关的模型类中。）例如，`Book`类可能有一个方法来返回一个代表该书的作者的`Author`类实例。另一方面，`Author`类可能有一个方法来返回一个由该作者撰写的每本书的`Book`类实例的列表。

在本章中，我们将开发我们自己的静态`Model`类以及`Book，Author`和`Borrower`类。在开始之前，我们应该清楚地定义每个类将具有的方法（功能）。让我们定义模型的功能。

`Model`类应包含静态方法，这些方法将充当数据库中存储的数据的*入口点*。这些方法应该执行以下操作：

+   获取所有的书籍。

+   获取所有的作者。

+   获取所有的书籍借阅者。

+   获取书籍的数量。

+   获取作者的数量。

+   获取书籍借阅者的数量。

+   按 ID 获取一本书。

+   按 ID 获取作者。

+   按 ID 获取借书人。

另一方面，`Model`类将不包含在书籍或作者上执行的方法。要借出一本书，我们将使用`Book`类中定义的方法，要归还一本书，我们将使用`Borrower`类中的方法。

现在让我们计划`Book`类的方法：

+   获取作者。

+   获取书籍的借书人列表。

+   借出一本书。

对于我们的示例应用程序，`Author`类甚至更简单：

+   获取所有的书籍。

+   获取该作者的书籍数量。

最后，还有代表借书人表中记录的`Borrower`类：

+   获取书籍。

+   返回书籍。

每个数据实体的属性将作为相关类的实例变量可访问。此外，这些类中的方法将包含我们已经在`books.php`和其他文件中编写的 PDO 调用。我们将这些方法移动到相关的类中，这些文件将只作为处理用户输入的控制器。表单验证仍然是控制器脚本的任务。但是，我们不打算将显示逻辑与业务逻辑分开，因为我们的应用程序非常简单，没有必要使用任何模板引擎，甚至将页面渲染代码移动到单独的**include**文件中。

除此之外，我们将不再使用全局变量`$conn`。`Model`类将有一个同名的私有静态变量和一个检索连接对象的方法。这个方法将遵循单例模式，并在需要时创建对象，如果尚未创建，则简单地返回它（有关单例模式的更多信息以及在 PHP5 中的示例实现，您可以访问[`en.wikipedia.org/wiki/Singleton_pattern`](http://en.wikipedia.org/wiki/Singleton_pattern)）。

我们将把所有类都放在一个名为`classes.inc.php`的单独文件中，然后从`common.inc.php`中包含它。

让我们从中心的`Model`类开始：

```php
/**
* This is the central Model class. Use its static methods
* To retrieve a book, author, borrower by ID
* Or all the books, authors and borrowers
*/
class Model
{
/**
* This is the connection object returned by
* Model::getConn()
* @var PDO
*/
private static $conn = null;
/**
* This method returns the connection object.
* If it has not been yet created, this method
* instantiates it based on the $connStr, $user and $pass
* global variables defined in common.inc.php
* @return PDO the connection object
*/
static function getConn()
{
if(!self::$conn) {
global $connStr, $user, $pass;
try
{
self::$conn = new PDO($connStr, $user, $pass);
self::$conn->setAttribute(PDO::ATTR_ERRMODE,
PDO::ERRMODE_EXCEPTION);
}
catch(PDOException $e)
{
showHeader('Error');
showError("Sorry, an error has occurred. Please
try your request later\n" . $e->getMessage());
}
}
return self::$conn;
}
/**
* This method returns the list of all books
* @return PDOStatement
*/
static function getBooks()
{
$sql = "SELECT * FROM books ORDER BY title";
$q = self::getConn()->query($sql);
$q->setFetchMode(PDO::FETCH_CLASS, 'Book', array());
return $q;
}
/**
* This method returns the number of books in the database
* @return int
*/
static function getBookCount()
{
$sql = "SELECT COUNT(*) FROM books";
$q = self::getConn()->query($sql);
$rv = $q->fetchColumn();
$q->closeCursor();
return $rv;
}
/**
*This method returns a book with given ID
* @param int $id
* @return Book
*/
static function getBook($id)
{
$id = (int)$id;
$sql = "SELECT * FROM books WHERE id=$id";
$q = self::getConn()->query($sql);
$rv = $q->fetchObject('Book');
$q->closeCursor();
return $rv;
}
/**
* This method returns the list of all authors
* @return PDOStatement
*/
static function getAuthors()
{
$sql = "SELECT * FROM authors ORDER BY lastName, firstName";
$q = self::getConn()->query($sql);
$q->setFetchMode(PDO::FETCH_CLASS, 'Author', array());
return $q;
}
/**
* This method returns the number of authors in the database
* @return int
*/
static function getAuthorCount()
{
$sql = "SELECT COUNT(*) FROM authors";
$q = self::getConn()->query($sql);
$rv = $q->fetchColumn();
$q->closeCursor();
return $rv;
}
/**
*This method returns an author with given ID
* @param int $id
* @return Author
*/
static function getAuthor($id)
{
$id = (int)$id;
$sql = "SELECT * FROM authors WHERE id=$id";
$q = Model::getConn()->query($sql);
$rv = $q->fetchObject('Author');
$q->closeCursor();
return $rv;
}
/**
* This method returns the list of all borrowers
* @return PDOStatement
*/
static function getBorrowers()
{
$sql = "SELECT * FROM borrowers ORDER BY dt";
$q = self::getConn()->query($sql);
$q->setFetchMode(PDO::FETCH_CLASS, 'Borrower', array());
return $q;
}
/**
* This method returns the number of borrowers in the database
* @return int
*/
static function getBorrowerCount()
{
$sql = "SELECT COUNT(*) FROM borrowers";
$q = self::getConn()->query($sql);
$rv = $q->fetchColumn();
$q->closeCursor();
return $rv;
}
/**
*This method returns a borrower with given ID
* @param int $id
* @return BorrowedBook
*/
static function getBorrower($id)
{
$id = (int)$id;
$sql = "SELECT * FROM borrowers WHERE id=$id";
$q = Model::getConn()->query($sql);
$rv = $q->fetchObject('Borrower');
$q->closeCursor();
return $rv;
}
}

```

正如您所见，这个类定义了`getConn()`方法，用于检索 PDO 连接对象，以及另外九个方法——每个数据实体（书籍、作者和借阅者）三个方法。获取所有数据实体的方法（`getBooks()`、`getAuthors()`和`getBorrowers()`）返回一个预配置为获取相关类实例的`PDOStatement`。返回每个数据实体的数量的方法获取一个整数，而返回单个数据实体的方法获取数据实体模型类的实例。请注意这些方法中如何关闭游标——这个功能已经从控制器文件中转移过来。

现在让我们来看看这三个模型类。

```php
/**
* This class represents a single book
*/
class Book
{
/**
* Return the author object for this book
* @return Author
*/
function getAuthor()
{
return Model::getAuthor($this->author);
}
/**
* This method is used to lend this book to the person
* specified by $name. It returns the Borrower class
* instance in case of success, or null in case when we cannot
* lend this book due to insufficient copies left
* @param string $name
* @return Borrower
*/
function lend($name)
{
$conn = Model::getConn();
$conn->beginTransaction();
try
{
$stmt = $conn->query("SELECT copies FROM books
WHERE id=$this->id");
$copies = $stmt->fetchColumn();
$stmt->closeCursor();
if($copies > 0) {
// If we can lend it
$conn->query("UPDATE books SET copies=copies-1
WHERE id=$this->id");
$stmt = $conn->prepare("INSERT INTO borrowers(book, name, dt)
VALUES(?, ?, ?)");
$stmt->execute(array($this->id, $name, time()));
// Success, get the newly created
// borrower ID
$bid = $conn->lastInsertId();
$rv = Model::getBorrower($bid);
}
else {
$rv = null;
}
$conn->commit();
}
catch(PDOException $e)
{
// Something bad happened
// Roll back and rethrow the exception
$conn->rollBack();
throw $e;
}
return $rv;
}
}

```

这里我们只有两个方法。一个用于获取书籍的作者（请注意我们在这里重用了`Model::getAuthor()`方法）。另一个方法提供了*借书*功能。请注意我们是从数据库中重新读取了副本列的值，而不是依赖于`$this->copies`变量。正如我们在上一章中看到的，这是为了确保数据完整性。`$this->copies`变量在事务开始之前就被赋值了，当调用`Book::lend()`方法时，数据库中的实际副本数量可能已经发生了变化。

这就是为什么我们在事务中再次读取该值。此外，如果操作失败，此方法将返回 null，如果操作成功，将返回`Borrower`类的实例。如果发生错误，将抛出一个异常，由`common.inc.php`中定义的异常处理程序处理（就像以前一样）。

另一个`model`类是`Author`。它非常简单：

```php
/**
* This class represents a single author
*/
class Author
{
/**
* This method returns the list of books
* written by this author
* @return PDOStatement
*/
function getBooks()
{
$sql = "SELECT * FROM books WHERE author=$this->id
ORDER BY title";
$q = Model::getConn()->query($sql);
$q->setFetchMode(PDO::FETCH_CLASS, 'Book', array());
return $q;
}
/**
* This method returns the number of books
* written by this author
* @return int
*/
function getBookCount()
{
$sql = "SELECT COUNT(*) FROM books WHERE author=$this->id";
$q = Model::getConn()->query($sql);
$rv = $q->fetchColumn();
$q->closeCursor();
return $rv;
}
}

```

这两个方法只是返回该作者写的书籍列表和此列表中的书籍数量。

最后，`Borrower`类表示借阅者表中的一条记录：

```php
/**
* This class represents a single borrower
* (i.e., a record in the borrowers table)
*/
class Borrower
{
/**
* Return the book associated with this borrower
* @return Book
*/
function getBook()
{
return Model::getBook($this->book);
}
/**
* This method "returns" a book.
* After this method call, this object
* is unusable as it does not represent
* a data entity any more
*/
function returnBook()
{
$conn = Model::getConn();
$conn->beginTransaction();
try
{
$book = $this->getBook();
$conn->query("DELETE FROM borrowers WHERE id=$this->id");
$conn->query("UPDATE books SET copies=copies+1
WHERE id=$book->id");
$conn->commit();
}
catch(PDOException $e)
{
$conn->rollBack();
throw $e;
}
}
}

```

实质上，`returnBook()`方法的主体是从`returnBook.php`文件中转移过来的（就像`Book::lend()`方法是从`lendBook.php`文件中稍作修改后转移过来的一样）。

# 修改前端以使用模型

现在我们已经从生成前端页面的文件中删除了数据访问逻辑，让我们看看应该如何修改它们。让我们从`books.php`文件开始：

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
**// Get the books list
$books = Model::getBooks();**
// now create the table
?>
**Total books: <?=Model::getBookCount()?>**
<table width="100%" border="1" cellpadding="3">
<tr style="font-weight: bold">
<td>Cover</td>
<td>Author and Title</td>
<td>ISBN</td>
<td>Publisher</td>
<td>Year</td>
<td>Summary</td>
<td>Copies</td>
<td>Lend</td>
<td>Edit</td>
</tr>
<?php
// Now iterate over every row and display it
**while($b = $books->fetch())
{
$a = $b->getAuthor();**
?>
<tr>
<td>
<?php if($b->coverMime) { ?>
**<img src="showCover.php?book=<?=$b->id?>">**
<?php } else { ?>
n/a
<? } ?>
</td>
<td>
**<a href="author.php?id=<?=$a->id?>"><?=htmlspecialchars("$a >firstName $a->lastName")?></a><br/>
<b><?=htmlspecialchars($b->title)?></b>
</td>
<td><?=htmlspecialchars($b->isbn)?></td>
<td><?=htmlspecialchars($b->publisher)?></td>
<td><?=htmlspecialchars($b->year)?></td>
<td><?=htmlspecialchars($b->summary)?></td>
<td><?=$b->copies?></td>
<td>
<a href="lendBook.php?book=<?=$b->id?>">Lend</a>
</td>
<td>
<a href="editBook.php?book=<?=$b->id?>">Edit</a>
</td>**
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

如您所见，我们已经删除了 SQL 命令和对 PDO 类实例方法的调用，并用`Model`类的方法调用替换了它们（请注意突出显示的行）。

另一个重要的变化是，在`while`循环中返回的`Book`类的实例（从第 30 行开始）没有作者的名字或姓氏的变量。为了获取这些变量，我们为我们显示的每一本书调用`Book::getAuthor()`方法。然后，在循环的后面，我们引用`$b`变量来访问书的属性，或者引用`$a`变量来访问作者的详细信息。请注意，我们在这里访问这些细节时，是作为对象变量而不是数组元素。

这是因为`Model::getBooks()`方法不再使用表连接，所以`Book`类的实例不会包含作者的详细信息。相反，`Book`类定义了一个方法来获取该书的`Author`对象。这意味着，对于我们显示的每一本书，我们将执行额外的 SQL 查询来获取作者的详细信息。

乍一看，这可能在性能上显得过于昂贵。但另一方面，在实际应用中，我们可能只会显示表中的一页（比如说，20 本书），而表中可能有数千条记录。在这种情况下，一个在`books`表上没有`JOIN`的`SELECT`语句，选择要在当前页面显示的行，然后对每一行进行一些简单的查询，可能更有效率。

然而，如果这种方法不合适，那么`Model`类可以扩展另一个方法，例如`Model::getBooksWithAuthors()`，它将返回`Book`类的实例，其中`lastName`和`firstName`变量将存在。这个方法可能看起来像下面这样：

```php
/**
* This method returns the list of all books with
* author's first and last names
* @return PDOStatement
*/
static function getBooksWithAuthors()
{
$sql = "SELECT books.*, authors.lastName, authors.firstName
FROM books, authors
WHERE books.author=authors.id
ORDER BY title";
$q = self::getConn()->query($sql);
$q->setFetchMode(PDO::FETCH_CLASS, 'Book', array());
return $q;
}

```

开发模型部分可能会在灵活性方面对我们施加限制，但这是为了代码可管理性而付出的代价。然而，这可以通过模型类中的其他方法或者如果真的有必要的话，通过与 PDO 的直接通信来克服。上述方法是可能的，因为 PDO 不关心类中定义了哪些变量；它只是动态地为查询返回的每一列创建变量。

当谨慎使用时，这是一个非常强大的功能。如果不小心使用，可能会导致难以跟踪的逻辑错误。例如，如果在上述方法中从作者表中选择了`ID`列，那么它的值将覆盖从书表中选择的`ID`列的值。`Book`类中的其他方法依赖于`id`字段中的值是正确的，如果这个值不正确，可能会导致严重的数据损坏。

我们现在应该修改的另一个文件是`authors.php:`

```php
<?php
/**
* This page lists all the authors we have
* PDO Library Management example application
* @author Dennis Popel
*/
// Don't forget the include
include('common.inc.php');
// Display the header
showHeader('Authors');
// Get number of authors and issue the query
**$authors = Model::getAuthors();**
// now create the table
?>
**Total authors: <?=Model::getAuthorCount()?>**
<table width="100%" border="1" cellpadding="3">
<tr style="font-weight: bold">
<td>First Name</td>
<td>Last Name</td>
<td>Bio</td>
<td>Edit</td>
</tr>
<?php
// Now iterate over every row and display it
**while($a = $authors->fetch())
{**
?>
<tr>
**<td><?=htmlspecialchars($a->firstName)?></td>
<td><?=htmlspecialchars($a->lastName)?></td>
<td><?=htmlspecialchars($a->bio)?></td>
<td>
<a href="editAuthor.php?author=<?=$a->id?>">Edit</a>
</td>**
</tr>
<?php
}
?>
</table>
<a href="editAuthor.php">Add Author...</a>
<?php
// Display footer
showFooter();

```

在这里，我们只是用对`Model`类的调用替换了与 PDO 的直接通信，并重写了循环以使用对象变量而不是数组元素。

对应用程序所做的更改还允许我们从`author.php:`中删除与 SQL 相关的代码片段。

```php
<?php
/**
* This page shows an author's profile
* PDO Library Management example application
* @author Dennis Popel
*/
// Don't forget the include
include('common.inc.php');
// Get the author
$id = (int)$_REQUEST['id'];
**$author = Model::getAuthor($id);**
// Now see if the author is valid - if it's not,
// we have an invalid ID
if(!$author) {
showHeader('Error');
echo "Invalid Author ID supplied";
showFooter();
exit;
}
// Display the header - we have no error
**showHeader("Author: $author->firstName $author->lastName");**
// Now get the number and fetch all his books
**$books = $author->getBooks();
$totalBooks = $author->getBookCount();**
// now display everything
?>
<h2>Author</h2>
<table width="60%" border="1" cellpadding="3">
<tr>
<td><b>First Name</b></td>
**<td><?=htmlspecialchars($author->firstName)?></td>**
</tr>
<tr>
<td><b>Last Name</b></td>
**<td><?=htmlspecialchars($author->lastName)?></td>**
</tr>
<tr>
<td><b>Bio</b></td>
**<td><?=htmlspecialchars($author->bio)?></td>**
</tr>
<tr>
<td><b>Total books</td>
<td><?=$totalBooks?></td>
</tr>
</table>
**<a href="editAuthor.php?author=<?=$author->id?>">Edit author...</a>**
<h2>Books</h2>
<table width="100%" border="1" cellpadding="3">
<tr style="font-weight: bold">
<td>Title</td>
<td>ISBN</td>
<td>Publisher</td>
<td>Year</td>
<td>Summary</td>
</tr>
<?php
// Now iterate over every book and display it
**while($b = $books->fetch())
{**
?>
<tr>
**<td><?=htmlspecialchars($b->title)?></td>
<td><?=htmlspecialchars($b->isbn)?></td>
<td><?=htmlspecialchars($b->publisher)?></td>
<td><?=htmlspecialchars($b->year)?></td>
<td><?=htmlspecialchars($b->summary)?></td>**
</tr>
<?php
}
?>
</table>
<?php
// Display footer
showFooter();

```

这里的变化相当表面，它只是删除了与 PDO 的直接通信，并将高亮显示的行上的*数组*语法更改为*对象*语法。

最后，显示`borrowers.php`中的列表的最后一个页面：

```php
<?php
/**
* This page lists all borrowed books
* PDO Library Management example application
* @author Dennis Popel
*/
// Don't forget the include
include('common.inc.php');
// Display the header
showHeader('Lended Books');
// Get all lended books list
**$brs = Model::getBorrowers();
$totalBooks = Model::getBorrowerCount();**
// now create the table
?>
Total borrowed books: <?=$totalBooks?>
<table width="100%" border="1" cellpadding="3">
<tr style="font-weight: bold">
<td>Title</td>
<td>Author</td>
<td>Borrowed by</td>
<td>Borrowed on</td>
<td>Return</td>
</tr>
<?php
// Now iterate over every row and display it
**while($br = $brs->fetch())
{
$b = $br->getBook();
$a = $b->getAuthor();**
?>
<tr>
**<td><?=htmlspecialchars($b->title)?></td>
<td><?=htmlspecialchars("$a->firstName $a->lastName")?></td>
<td><?=htmlspecialchars($br->name)?></td>
<td><?=date('d M Y', $br->dt)?></td>
<td>
<a href="returnBook.php?borrower=<?=$br->id?>">Return</a>
</td>**
</tr>
<?php
}
?>
</table>
<?php
// Display footer
showFooter();

```

在这个文件中，我们遇到了与`books.php`页面相同的问题——`Model`类返回的`Borrower`类实例没有书名和作者名，而我们希望在这个页面上显示。因此，我们在每次迭代中为每个`Borrower`类实例获取`Book`类实例，然后使用该对象获取作者的详细信息。

最后，我们将修改另外两个页面，以利用我们新创建的数据模型。这两个页面是`lendBook.php`和`returnBook.php`。它们可能包含了与 PDO 交互的最长的代码段。从`lendBook.php`中，我们移除了事务内部的所有代码：

```php
<?php
/**
* This page allows you to lend a book
* PDO Library Management example application
* @author Dennis Popel
*/
// Don't forget the include
include('common.inc.php');
// First see if the request contains the book ID
// Return to books.php if the ID invalid
$id = (int)$_REQUEST['book'];
**$book = Model::getBook($id);**
if(!$book) {
header("Location: books.php");
exit;
}
// Now see if the form was submitted
$warnings = array();
if($_POST['submit']) {
// Require that the borrower's name is entered
if(!$_POST['name']) {
$warnings[] = 'Please enter borrower\'s name';
}
else {
**// Form is OK, "lend" the book
if(!$book->lend($_POST['name'])) {
// Failure, show error message
$warnings[] = 'There are no more copies of
this book available';
}**
}
// Now, if we don't have errors,
// redirect back to books.php
if(count($warnings) == 0) {
header("Location: books.php");
exit;
}
// Otherwise, the warnings will be displayed
}
// Display the header
showHeader('Lend Book');
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
<form action="lendBook.php" method="post">
<input type="hidden" name="book" value="<?=$id?>">
<b>Please enter borrower's name:<br></b>
<input type="text" name="name" value="<?=htmlspecialchars($_
POST['name'])?>">
<input type="submit" name="submit" value=" Lend book ">
</form>
<?php
// Display footer
showFooter();

```

注意我们如何改变了*借出*图书的部分——`Bool::lend()`方法在失败的情况下返回`null`，因此我们将显示没有更多可借的书的消息。如果操作成功，那么`Book::lend()`方法将返回`Borrower`类实例（在`if`语句中求值为`true`），页面将重定向到`books.php`。

类似地，我们从`returnBook.php`中删除了与 PDO 相关的代码，并用相应的调用`Borrower::returnBook()`方法替换：

```php
<?php
/**
* This page "returns" a book back to the library
* PDO Library Management example application
* @author Dennis Popel
*/
// Don't forget the include
include('common.inc.php');
// First see if the request contains the borrowers ID
// Return to books.php if not
$id = (int)$_REQUEST['borrower'];
**$borrower = Model::getBorrower($id);**
if(!$borrower) {
header("Location: books.php");
exit;
}
// Return the book and redirect to books.php
// If anything happens, the exception will be
// handled automatically
**$borrower->returnBook();**
header("Location: books.php");

```

# 分离模型的优势

到目前为止，几乎所有生成前端页面的文件都不包含数据访问逻辑，更容易管理。另一方面，模型类可以从我们的应用程序外部使用，并且可以快速创建额外的页面来以其他格式（如 XML）表示数据库中的信息。

例如，考虑以下页面（我们将其称为`books.xml.php`）：

```php
<?php
/**
* This page lists all the books we have as an XML data structure
* PDO Library Management example application
* @author Dennis Popel
*/
// Don't forget the include
include('common.inc.php');
// Set the content type to be XML
header('Content-Type: application/xml');
// Get the books list
$books = Model::getBooksWithAuthors();
// Echo XML declaration and open root element
echo '<?xml version="1.0"?>', "\n";
echo "<books>\n";
// Now iterate over every book and display it
while($b = $books->fetch())
{
?>
<book id="<?=$b->id?>">
<isbn><?=$b->isbn?></isbn>
<title><?=htmlspecialchars($b->title)?></title>
<publisher><?=htmlspecialchars($b->publisher)?></publisher>
<summary><?=htmlspecialchars($b->summary)?></summary>
<author>
<id><?=$b->author?></id>
<lastName><?=$b->lastName?></lastName>
<firstName><?=$b->firstName?></firstName>
</author>
</book>
<?
}
echo '</books>';

```

这个文件允许我们以 XML 格式导出书籍列表，供另一个应用程序使用。正如你所看到的，对原始的`books.php`文件的更改只在显示逻辑中。如果你现在导航到该页面，你应该会看到以下内容：

![分离模型的优势](img/2660_07_01.jpg)

通过轻微修改，我们能够创建数据的新表示（第二和第三本书已经折叠以适应截图）。

定义`model`类的另一个优势是，这些类成为数据访问和操作的中心点。例如，如果你改变了用于表示来自多个表的数据的 SQL（使用连接）或找到了优化查询的方法，你只需要更新相关的模型类，使用该查询的脚本（控制器）就不需要更新。这是一个重要的可管理性优势。

你可以扩展抽象模型类，以模拟通用数据模型中真实子类的扩展功能。例如，在内容管理系统中，你可以创建一个名为`Item`的抽象基类，它将为所有子类（项目类型）具有共同的属性，如作者、关键词和创建日期。然后模型可以对所有可能的子类执行一些操作，而无需进一步编码，以便广泛重用现有代码。

有一种叫做**对象关系映射器**（**ORMs**）的工具，它们利用了本章描述的思想。ORMs 用于创建功能强大的面向对象应用程序，在这些应用程序中，你的模型中几乎没有 SQL 代码。（事实上，这些工具在一些配置后扮演了你应用程序中的模型的角色。）你可以在[`en.wikipedia.org/wiki/Object-relational_mapping`](http://en.wikipedia.org/wiki/Object-relational_mapping)了解更多关于 ORMs 的信息。Propel ([`propel.phpdb.org/`](http://propel.phpdb.org/))是 PHP5 的一种流行的 ORM 工具。

# 进一步思考

本章开发的模型在至少两个领域需要一些改进，如果你想在实际应用中使用它的话。我们没有在模型中创建能够提供`editBook.php`和`editAuthor.php`文件功能的方法。然而，现在你应该准备自己添加这些功能。我们将为你提供一些提示：

+   创建`Book::update()`和`Author::update()`方法。这些方法应该接受反映每个对象属性的参数（对于`Author`类，这应该是名字、姓氏和传记）。

+   这些方法应该使用预处理语句来更新数据库中相应的记录（基于`$this->id`的值）。

+   `Model`类应该扩展两个方法，`Model::createBook()`和`Model::createAuthor()`。这些方法应该接受与`Book::update()`和`Author::update()`相同的参数列表。两者都应该根据传递的参数插入一行到相关表中。可以使用以下代码完成这个操作：

```php
$conn = self::getConn();
$conn->beginTransaction();
try
{
$conn->query("INSERT INTO authors(bio) VALUES('')");
$aid = $conn->lastInsertId();
$author = self::getAuthor($aid);
$author->update($firstName, $lastName, $bio);
$conn->commit();
}
catch(Exception $e)
{
$conn->rollBack();
}

```

+   这里的想法是将实体更新集中在一个地方，即`Author::update()`。我们在这里使用事务来确保，如果发生任何事情，空行不会存储在数据库中。

+   表单处理代码应该检测它是在编辑现有实体还是创建新实体，并在已经存在的实例上适当地调用`Model::createAuthor()`或`Author::update()`。

另一个问题是，模型类的方法不验证接受的参数。如果要将数据模型暴露给第三方应用程序，它们应该对传递到数据库的每个参数进行验证。如果通过 Web 浏览器访问，我们的数据模型受到表单验证代码的保护。然而，直接访问模型类并不那么安全。

建议在模型方法中接受用户提供的参数时，如果验证失败，抛出异常。此外，Web 表单验证和方法参数验证应该使用通用代码。（例如，您可以开发一个`Validation`类，无论值来自何处，都可以用来验证。）这段代码应该从表单验证代码和模型方法中使用。通过这样做，您将确保代码重用和验证规则的单一位置。

# 收尾工作

PHP 数据对象是一种很棒且易于使用的技术。然而，它仍处于起步阶段，许多改进和其他变化尚未到来。一定要及时了解来自 PHP 开发人员和大量 PHP 粉丝和用户的最新消息。

只有对安全威胁有深刻的理解并知道如何防范，才能有效地使用 PDO 和 PHP。使用 PDO 的预处理语句可以减少 SQL 注入攻击的风险，但作为开发人员，您仍然负责保护您的应用程序。确保您及时了解安全领域的最新发展。

愉快的 PHP 编程！
