# 第五章：处理行集

现实生活中的动态数据驱动的 Web 应用程序彼此非常不同，因为它们的复杂性由它们服务的目的决定。然而，几乎所有这些应用程序都具有一些共同的特征。其中之一是对长结果列表进行分页以方便使用和更快的页面加载时间。

正确的分页需要计算从数据库返回的总行数、页面大小（可配置选项）和当前页面的数量。根据这些数据，很容易计算结果集的起始偏移量，以仅显示一部分行。

在本章中，我们将研究：

+   如何检索 PDO 返回的结果集中的行数

+   如何从指定的行号开始获取结果

# 检索结果集中的行数

正如我们在第二章中已经讨论的，`PDOStatement::rowCount()`方法不会返回查询中的正确行数。（对于 MySQL 和 SQLite 都返回零。）这种行为的原因是数据库管理系统实际上直到返回查询的最后一行才知道这个数字。`mysql_num_rows()`函数（以及其他数据库的类似函数）返回行数的原因是，当您发出查询时，它会将整个结果集预加载到内存中。

虽然这种行为可能看起来很方便，但并不推荐。如果查询返回 20 行，那么脚本可以承受内存使用。但是如果查询返回数十万行呢？它们都将保留在内存中，因此在高流量站点上，服务器可能会耗尽资源。

唯一的逻辑措施（也是 PDO 可用的唯一选项）是指示数据库自己计算行数。无论查询有多复杂，都可以重写以使用 SQL 的`COUNT()`函数，仅返回满足主查询的行数。

让我们看一下我们应用程序中使用的查询。（我们只会检查返回多行的查询。）

+   在`books.php`中，我们有一个查询，它连接两个表以呈现书籍列表以及它们的作者：

```php
SELECT authors.id AS authorId, firstName, lastName, books.*
FROM authors, books WHERE author=authors.id ORDER BY title;

```

要获取此查询返回的行数，我们应该将其重写为以下内容：

```php
SELECT COUNT(*) FROM authors, books WHERE author=authors.id;

```

请注意，这里不需要`ORDER BY`子句，因为顺序对行数并不重要。

+   在`authors.php`中，我们只是按照他们的姓和名的顺序选择所有作者：

```php
SELECT * FROM authors ORDER BY lastName, firstName;

```

这简单地重写为以下内容：

```php
SELECT COUNT(*) FROM authors;

```

+   另一个返回多行的查询在`author.php`中——它检索特定作者撰写的所有书籍：

```php
SELECT * FROM books WHERE author=$id ORDER BY title;

```

这翻译为以下内容：

```php
SELECT COUNT(*) FROM books WHERE author=$id;

```

正如您所看到的，我们以类似的方式重写了所有这些查询——通过用`COUNT(*)`替换列的列表并修剪`ORDER BY`子句。有了这个想法，我们可以创建一个函数，它将接受一个包含要执行的 SQL 的字符串，并返回查询将返回的行数。这个函数将必须执行这些简单的转换：

+   在传递的字符串中，用`COUNT(*)`替换`SELECT`和`FROM`之间的所有内容。

+   删除`ORDER BY`及其后的所有文本。

实现这种转换的最佳方法是使用正则表达式。与前几章一样，我们将使用 PCRE 扩展。我们将把该函数放入`common.inc.php`中，因为我们将从各个地方调用它：

```php
/**
* This function will return the number of rows a query will return
* @param string $sql the SQL query
* @return int the number of rows the query specified will return
* @throws PDOException if the query cannot be executed
*/
function getRowCount($sql)
{
global $conn;
$sql = trim($sql);
$sql = preg_replace('~^SELECT\s.*\sFROM~s', 'SELECT COUNT(*) FROM',
$sql);
$sql = preg_replace('~ORDER\s+BY.*?$~sD', '', $sql);
$stmt = $conn->query($sql);
$r = $stmt->fetchColumn(0);
$stmt->closeCursor();
return $r;
}

```

让我们运行一下这个函数，看看它做了什么：

1.  它将 PDO 连接对象（`$conn`）导入到本地函数范围内。

1.  它修剪了 SQL 查询开头和结尾的可能空格。

1.  两次对`preg_replace()`的调用完成了转换查询的主要任务。

注意我们如何使用模式修饰符——*s*修饰符指示 PCRE 用点匹配换行符，*D*修饰符强制$匹配整个字符串的结尾（不仅仅是在第一个换行符之前）。我们使用这些修饰符来确保函数能够正确处理多行查询。

我们现在将修改这三个脚本，以显示它们返回的每个表中的行数。让我们从`books.php`开始：

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
**// Get the count of books and issue the query
$sql = "SELECT authors.id AS authorId, firstName, lastName, books.*
FROM authors, books WHERE author=authors.id ORDER BY title";
$totalBooks = getRowCount($sql);
$q = $conn->query($sql);**
$q->setFetchMode(PDO::FETCH_ASSOC);
// now create the table
?>
**Total books: <?=$totalBooks?>**
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
<tr>
<td>
<?php if($r['coverMime']) { ?>
<img src="showCover.php?book=<?=$r['id']?>">
<?php } else { ?>
n/a
<? } ?>
</td>
<td>
<a href="author.php?id=<?=$r['authorId']?>"><?=htmlspecialchars
("$r[firstName] $r[lastName]")?></a><br/>
<b><?=htmlspecialchars($r['title'])?></b>
</td>
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

正如你所看到的，修改非常简单——我们使用`$sql`变量来保存查询，并将其传递给`getRowCount()`函数和`$conn->query()`方法。我们还在表格上方显示一条消息，告诉我们数据库中有多少本书。

现在，如果你刷新`books.php`页面，你会看到以下内容：

![检索结果集中的行数](img/2660_05_01.jpg)

`authors.php`的更改类似：

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
**// Get the number of authors and issue the query
$sql = "SELECT * FROM authors ORDER BY lastName, firstName";
$totalAuthors = getRowCount($sql);**
$q = $conn->query($sql);
// now create the table
?>
**Total authors: <?=$totalAuthors?>**
<table width="100%" border="1" cellpadding="3">
<tr style="font-weight: bold">
<td>First Name</td>
<td>Last Name</td>
<td>Bio</td>
<td>Edit</td>
</tr>
<?php
// Now iterate over every row and display it
while($r = $q->fetch(PDO::FETCH_ASSOC))
{
?>
<tr>
<td><?=htmlspecialchars($r['firstName'])?></td>
<td><?=htmlspecialchars($r['lastName'])?></td>
<td><?=htmlspecialchars($r['bio'])?></td>
<td>
<a href="editAuthor.php?author=<?=$r['id']?>">Edit</a>
</td>
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

`authors.php`现在应该显示以下内容：

![检索结果集中的行数](img/2660_05_02.jpg)

最后，`author.php`将如下所示：

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
$q = $conn->query("SELECT * FROM authors WHERE id=$id");
$author = $q->fetch(PDO::FETCH_ASSOC);
$q->closeCursor();
$q = null;
// Now see if the author is valid - if it's not,
// we have an invalid ID
if(!$author) {
showHeader('Error');
echo "Invalid Author ID supplied";
showFooter();
exit;
}
// Display the header - we have no error
showHeader("Author: $author[firstName] $author[lastName]");
**// Now get the number and fetch all the books
$sql = "SELECT * FROM books WHERE author=$id ORDER BY title";
$totalBooks = getRowCount($sql);
$q = $conn->query($sql);
$q->setFetchMode(PDO::FETCH_ASSOC);**
// now display everything
?>
<h2>Author</h2>
<table width="60%" border="1" cellpadding="3">
<tr>
<td><b>First Name</b></td>
<td><?=htmlspecialchars($author['firstName'])?></td>
</tr>
<tr>
<td><b>Last Name</b></td>
<td><?=htmlspecialchars($author['lastName'])?></td>
</tr>
<tr>
<td><b>Bio</b></td>
<td><?=htmlspecialchars($author['bio'])?></td>
</tr>
**<tr>
<td><b>Total books</td>
<td><?=$totalBooks?></td>
</tr>**
</table>
<a href="editAuthor.php?author=<?=$author['id']?>">Edit author...</a>
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
while($r = $q->fetch()) {
?>
<tr>
<td><?=htmlspecialchars($r['title'])?></td>
<td><?=htmlspecialchars($r['isbn'])?></td>
<td><?=htmlspecialchars($r['publisher'])?></td>
<td><?=htmlspecialchars($r['year'])?></td>
<td><?=htmlspecialchars($r['summary'])?></td>
</tr>
<?php
}
?>
</table>
<?php
// Display footer
showFooter();

```

输出应该如下所示。（我把页面向下滚动了一点以节省空间）：

![检索结果集中的行数](img/2660_05_03.jpg)

你应该在`common.inc.php`中在 MySQL 和 SQLite 之间切换，以确保两个数据库都能工作。

### 注意

这种方法可能适用于许多情况，但并不适用于所有查询。一个这样的例子是使用`GROUP BY`子句的查询。如果你用`getRowCount()`函数重写这样的查询，你将得到不正确的结果，因为分组将被应用，查询将返回多行。（行数将等于你正在分组的列中不同值的数量。）

# 限制返回的行数

现在，我们知道如何计算结果集中的行数，让我们看看如何只获取前 N 行。这里我们有两个选项：

+   我们可以在 SQL 查询中使用特定于数据库的功能。

+   我们可以自己处理结果集，并在获取所需数量的行后停止。

## 使用特定于数据库的 SQL

如果你主要使用 MySQL，那么你会熟悉`LIMIT x,y`子句。例如，如果我们想按姓氏排序获取前五位作者，可以发出以下查询：

```php
SELECT * FROM authors ORDER BY lastName LIMIT 0, 5;

```

同样的事情也可以用以下查询完成：

```php
SELECT * FROM authors ORDER BY lastName LIMIT 5 OFFSET 0;

```

第一个查询适用于 MySQL 和 SQLite，而第二个查询也适用于 PostgreSQL。然而，像 Oracle 或 MS SQL Server 这样的数据库不使用这样的语法，所以这些查询对它们来说将失败。

## 仅处理前 N 行

正如你所看到的，特定于数据库的 SQL 不允许我们以数据库无关的方式解决执行分页的任务。然而，我们可以像对待所有行一样发出查询，而不使用`LIMIT....OFFSET`子句。在获取每一行后，我们可以增加计数器变量，这样当我们处理了所需数量的行时，我们就可以中断循环。以下代码片段可以实现这一目的：

```php
$q = $conn->query("SELECT * FROM authors ORDER BY lastName,
firstName");
$q->setFetchMode(PDO::FETCH_ASSOC);
$count = 1; **while(($r = $q->fetch()) && $count <= 5)**
{
echo $r['lastName'], '<br>';
$count++;
} **$q->closeCursor();
$q = null;**

```

注意循环条件——它检查计数器变量是否小于或等于 5。（当然，你可以在那里放任何数字），以及它验证是否还有行要获取，因为重要的是如果没有更多行要获取，我们就中断循环。（例如，如果表只有 3 行，我们想显示其中的 5 行，我们应该在最后一行后中断，而不是在计数器达到 5 后中断。）请注意，使用特定于数据库的 SQL 将为我们处理这样的情况。

另一个重要的事情是调用`PDOStatement::closeCursor()`（如前一个代码片段中倒数第二行）。有必要告诉数据库我们不需要更多的行。如果我们不这样做，那么在同一个 PDO 对象上发出的后续查询将引发异常，因为数据库管理系统无法在仍在发送上一个查询的行时处理新查询。这就是为什么我们在`author.php`中必须调用这个方法。

### 注意

目前（对于 PHP 版本 5.2.1），可能需要将语句对象取消分配为 null（如`author.php`，第 17 行）。另一方面，至少在 2007 年 4 月 1 日左右发布的一个 CVS 快照根本不需要关闭游标。但是，在完成游标后调用`PDOStatement::closeCursor()`仍然是一个好习惯。

## 从任意偏移开始

现在我们知道如何处理指定数量的行，我们可以使用相同的技术来跳过一定数量的行。假设我们想显示第 6 到第 10 位作者（就像我们在每页允许每页 5 位作者时显示第 2 页）：

```php
$q = $conn->query("SELECT * FROM authors ORDER BY lastName,
firstName");
$q->setFetchMode(PDO::FETCH_ASSOC);
$count = 1;
while(($r = $q->fetch()) && $count <= 5)
{
$count++;
}
$count = 1;
while(($r = $q->fetch()) && $count <= 5)
{
echo $r['lastName'], '<br>';
$count++;
}
$q->closeCursor();
$q = null;

```

在这里，第一个循环用于跳过必要的起始行，第二个循环显示请求的行的子集。

### 注意

这种方法对小表可能效果很好，但性能不佳。您应该始终使用特定于数据库的 SQL 来返回结果行的子集。如果您需要数据库独立性，应该检查底层数据库软件并发出特定于数据库的查询。原因是数据库可以对查询执行某些优化，使用更少的内存，从而在服务器和客户端之间交换的数据量更少。

不幸的是，PDO 没有提供数据库独立的方法来有效地获取结果行的子集，因为 PDO 是连接抽象，而不是数据库抽象工具。如果您需要编写可移植的代码，应该探索 MDB2 等工具。

这种方法可能比使用`PDOStatement::fetchAll()`方法更复杂。事实上，我们可以将上一个代码重写如下：

```php
$stmt = $conn->query("SELECT * FROM authors ORDER BY lastName,
firstName");
$page = $stmt->fetchAll(PDO::FETCH_ASSOC);
$page = array_slice($page, 5, 5);
foreach($page as $r)
{
echo $r['lastName'], '<br>';
}

```

尽管这段代码要短得多，但它有一个主要缺点：它指示 PDO 返回表中的所有行，然后取其中的一部分。使用我们的方法，不必要的行将被丢弃，并且循环指示数据库在返回足够的行后停止发送行。但是，在这两种情况下，数据库都必须向我们发送当前页面之前的行。

# 总结

在本章中，我们已经看到如何处理无缓冲查询并获取结果集的行数。我们还看了一个应用程序，其中无法避免使用特定于数据库的 SQL，因为这将需要一个可能不合适的解决方法。但是，这一章对于开发使用数据库的复杂 Web 应用程序的人应该是有帮助的。

在下一章中，我们将讨论 PDO 的高级功能，包括持久连接和其他特定于驱动程序的选项。我们还将讨论事务并检查`PDO`和`PDOStatement`类的更多方法。
