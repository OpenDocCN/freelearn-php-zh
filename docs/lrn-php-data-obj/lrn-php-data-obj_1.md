# 第一章：介绍

**PHP 数据对象**（**PDO**）是一个 PHP5 扩展，定义了一个轻量级的数据库连接抽象库（有时称为数据访问抽象库）。对于像 PDO 这样的工具的需求是由 PHP 支持的大量数据库系统所决定的。这些数据库系统中的每一个都需要一个单独的扩展，为执行相同的任务定义自己的 API，从建立连接到准备语句和错误处理等高级功能。

这些 API 不统一的事实使得在底层数据库之间的转换痛苦，通常导致许多代码行的重写，进而导致需要时间来跟踪、调试和纠正新的编程错误。另一方面，缺乏像 Java 的 JDBC 那样的统一库，使得 PHP 在编程语言世界中落后于大型玩家。现在有了这样的库，PHP 正在重新夺回其地位，并成为数百万程序员的首选平台。

然而，值得注意的是，存在一些用 PHP 编写的库，用于与 PDO 具有相同的目的。最流行的是 ADOdb 库和 PEAR DB 包。它们与 PDO 之间的关键区别在于速度。PDO 是用编译语言（C/C++）编写的 PHP 扩展，而 PHP 库是用解释语言编写的。此外，一旦启用 PDO，它就不需要您在脚本中包含源文件并将其与应用程序一起重新分发。这使得安装您的应用程序更容易，因为最终用户不需要关心第三方软件。

### 注意

在这里，我们既不比较这些库与 PDO，也不主张使用 PDO 取代这些库。我们只是展示这个扩展的优缺点。例如，PEAR 包 MDB2 具有更丰富的功能，是一个高级的数据库抽象库，而 PDO 没有。

PDO 作为一个 PECL 扩展，本身依赖于特定于数据库的驱动程序和其他 PECL 扩展。这些驱动程序也必须安装才能使用 PDO（您只需要用于您正在使用的数据库的驱动程序）。由于安装 PDO 和特定于数据库的驱动程序的描述超出了本书的范围，您可以参考 PHP 手册[www.php.net/pdo](http://www.php.net/pdo)获取有关安装和升级问题的技术信息。

### 注意

PECL 是 PHP 扩展社区库，一个用 C 语言编写的 PHP 扩展库。这些扩展提供了在 PHP 中无法实现的功能，以及一些出于性能原因存在的扩展，因为 C 代码比 PHP 快得多。PECL 的主页位于[`pecl.php.net`](http://pecl.php.net)

# 使用 PDO

正如前一节中所指出的，PDO 是一个连接或数据访问抽象库。这意味着 PDO 定义了一个统一的接口，用于创建和维护数据库连接，发出查询，引用参数，遍历结果集，处理准备好的语句和错误处理。

我们将在这里简要概述这些主题，并在接下来的章节中更详细地讨论它们。

## 连接到数据库

让我们考虑一下著名的 MySQL 连接场景：

```php
mysql_connect($host, $user, $password);
mysql_select_db($db);

```

在这里，我们建立一个连接，然后选择连接的默认数据库。（我们忽略可能出现的错误。）

例如，在 SQLite 中，我们会写出以下内容：

```php
$dbh = sqlite_open($db, 0666);

```

在这里我们再次忽略错误（稍后我们将更多地涵盖这一点）。为了完整起见，让我们看看如何连接到 PostgreSQL：

pg_connect("host=$host dbname=$db user=$user password=$password");

正如您所看到的，所有三个数据库都需要完全不同的方式来打开连接。虽然现在这不是问题，但如果您总是使用相同的数据库管理系统，以防需要迁移，您将不得不重写您的脚本。

现在，让我们看看 PDO 提供了什么。由于 PDO 是完全面向对象的，我们将处理**连接对象**，与数据库的进一步交互将涉及调用这些对象的各种方法。上面的示例暗示了需要类似于这些连接对象的东西——调用`mysql_connect`或`pg_connect`返回链接标识符和特殊类型的 PHP 变量：**resource**。然而，我们当时没有使用连接对象，因为这两个数据库 API 不要求我们在脚本中只有一个连接时显式使用它们。然而，SQLite 始终需要一个链接标识符。

使用 PDO，我们将始终必须显式使用连接对象，因为没有其他调用其方法的方式。（不熟悉面向对象编程的人应参考附录 A）。

上述三个连接可以以以下方式建立：

```php
// For MySQL:
$conn = new PDO("mysql:host=$host;dbname=$db", $user, $pass);
// For SQLite:
$conn = new PDO("sqlite:$db");
// And for PostgreSQL:
$conn = new PDO("pgsql:host=$host dbname=$db", $user, $pass);

```

正如你所看到的，这里唯一变化的部分是传递给 PDO 构造函数的第一个参数。对于 SQLite，不使用用户名和密码，第二和第三个参数可以省略。

### 注意

SQLite 不是一个数据库服务器，而是一个嵌入式 SQL 数据库库，它在本地文件上运行。有关 SQLite 的更多信息可以在[www.sqlite.org](http://www.sqlite.org)找到，有关使用 SQLite 与 PHP 的更多信息可以在[www.php.net/sqlite](http://www.php.net/sqlite)找到。有关使用 PDO 与 SQLite 的信息可以从[www.php.net/manual/en/ref.pdo-sqlite.php](http://www.php.net/manual/en/ref.pdo-sqlite.php)获取。

## 连接字符串

正如你在前面的示例中看到的，PDO 使用所谓的**连接字符串**（或数据源名称，缩写为 DSN），允许 PDO 构造函数选择适当的驱动程序并将后续方法调用传递给它。这些连接字符串或 DSN 对于每个数据库管理系统都是不同的，是你唯一需要更改的东西。

如果你正在设计一个能够与不同数据库一起工作的大型应用程序，那么这个连接字符串（连同连接用户名和密码）可以在配置文件中定义，并以后以以下方式使用（假设你的配置文件类似于`php.ini`）。

```php
$config = parse_ini_file($pathToConfigFile);
$conn = new PDO($config['db.conn'], $config['db.user'],
$config['db.pass']);

```

然后你的配置文件可能如下所示：

```php
db.conn="mysql:host=localhost;dbname=test"
db.user="johns"
db.pass="mypassphrase"

```

我们将在第二章中更详细地介绍连接字符串；在这里，我们给出了一个快速示例，以便你可以看到使用 PDO 连接到不同数据库系统有多么容易。

## 发出 SQL 查询、引用参数和处理结果集

如果 PDO 没有超越创建数据库连接的单一接口，那么它就不值得写一本书。在前面的示例中介绍的 PDO 对象具有统一执行查询所需的所有方法，而不管使用的是哪种数据库。

让我们考虑一个简单的查询，它将从一个虚构的二手车停车场所使用的数据库中选择所有汽车的`make`属性。查询就像下面的 SQL 命令一样简单：

```php
SELECT DISTINCT make FROM cars ORDER BY make;

```

以前，我们必须调用不同的函数，这取决于数据库：

```php
// Let's keep our SQL in a single variable
$sql = 'SELECT DISTINCT make FROM cars ORDER BY make';
// Now, assuming MySQL:
mysql_connect('localhost', 'boss', 'password');
mysql_select_db('cars');
$q = mysql_query($sql);
// For SQLite we would do:
$dbh = sqlite_open('/path/to/cars.ldb', 0666);
$q = sqlite_query($sql, $dbh);
// And for PostgreSQL:
pg_connect("host=localhost dbname=cars user=boss
password=password");
$q = pg_query($sql);

```

现在我们使用 PDO，可以这样做：

```php
// assume the $connStr variable holds a valid connection string
// as discussed in previous point
$sql = 'SELECT DISTINCT make FROM cars ORDER BY make';
$conn = new PDO($connStr, 'boss', 'password');
$q = $conn->query($sql);

```

如你所见，用 PDO 的方式并不太不同于发出查询的传统方法。此外，这里应该强调的是，对`$conn->query()`的调用返回`PDOStatement`类的另一个对象，而不像对`mysql_query()`、`sqlite_query()`和`pg_query()`的调用，它们返回 PHP 变量的**resource**类型。

现在，让我们将我们简单的 SQL 查询变得更加复杂，以便它选择我们想象中汽车停车场上所有福特车的总价值。查询看起来会像这样：

```php
SELECT sum(price) FROM cars WHERE make='Ford'

```

为了使我们的示例更加有趣，让我们假设汽车制造商的名称保存在一个变量（$make）中，这样我们必须在传递给数据库之前对其进行引用。我们的非 PDO 查询现在看起来像这样：

```php
$make = 'Ford';
// MySQL:
$m = mysql_real_escape_string($make);
$q = mysql_query("SELECT sum(price) FROM cars WHERE make='$m'");
// SQLite:
$m = sqlite_escape_string($make);
$q = sqlite_query("SELECT sum(price) FROM cars WHERE make='$m'",
$dbh);
// and PostgreSQL:
$m = pg_escape_string($make);
$q = pg_query("SELECT sum(price) FROM cars WHERE make='$m'");

```

PDO 类定义了一个用于引用字符串的方法，以便它们可以安全地用于查询。我们将在第三章中讨论诸如 SQL 注入之类的安全问题。这个方法做了一个很好的事情；如果有必要，它会自动在值周围添加引号：

```php
$m = $conn->quote($make);
$q = $conn->query("SELECT sum(price) FROM cars WHERE make=$m");

```

再次，您可以看到 PDO 允许您使用与以前相同的模式，但所有方法的名称都是统一的。

现在我们已经发出了查询，我们将想要查看其结果。由于上一个示例中的查询总是只返回一行，我们将想要更多行。同样，这三个数据库将要求我们在从`mysql_query(), sqlite_query()`或`pg_query()`中返回的`$q`变量上调用不同的函数。因此，我们获取所有汽车的代码将类似于这样：

```php
// assume the query is in the $sql variable
$sql = "SELECT DISTINCT make FROM cars ORDER BY make";
// For MySQL:
$q = mysql_query($sql);
while($r = mysql_fetch_assoc($q))
{
echo $r['make'], "\n";
}
// For SQLite:
$q = sqlite_query($dbh, $sql);
while($r = sqlite_fetch_array($q, SQLITE_ASSOC))
{
echo $r['make'], "\n";
}
// and, finally, PostgreSQL:
$q = pg_query($sql);
while($r = pg_fetch_assoc($q))
{
echo $r['make'], "\n";
}

```

正如你所看到的，想法是一样的，但我们必须使用不同的函数名。另外，需要注意的是，如果我们想以与 MySQL 和 PostgreSQL 相同的方式获取行，SQLite 需要一个额外的参数（当然，这可以省略，但返回的行将包含列名索引和数字索引的元素。）

正如您可能已经猜到的那样，当涉及到 PDO 时，事情变得非常简单：我们不关心底层数据库是什么，获取行的方法在所有数据库中都是相同的。因此，上面的代码可以以以下方式重写为 PDO：

```php
$q = $conn->query("SELECT DISTINCT make FROM cars ORDER BY make");
while($r = $q->fetch(PDO::FETCH_ASSOC))
{
echo $r['make'], "\n";
}

```

之前发生的事情与以往无异。这里需要注意的一点是，我们在这里明确指定了`PDO::FETCH_ASSOC`的获取样式常量，因为 PDO 的默认行为是将结果行作为数组索引，既按列名又按数字索引。（这种行为类似于`mysql_fetch_array(), sqlite_fetch_array()`没有第二个参数，或者`pg_fetch_array()`。）我们将在第二章中讨论 PDO 提供的获取样式。

### 注意

最后一个示例并不是用于呈现 HTML 页面，因为它使用换行符来分隔输出行。要在真实的网页中使用它，您需要将`echo $r['make'], "\n"`;更改为`echo $r['make'], "<br>\n"`;

## 错误处理

当然，上面的示例没有提供任何错误检查，因此对于实际应用程序来说并不是非常有用。

在与数据库一起工作时，我们应该在打开数据库连接时，选择数据库时以及发出每个查询后检查错误。然而，大多数 Web 应用程序只需要在出现问题时显示错误消息（而不需要详细的错误信息，这可能会泄露一些敏感信息）。但是，在调试错误时，您（作为开发人员）需要尽可能详细的错误信息，以便您可以在最短的时间内调试错误。

一个简单的情景是中止脚本并呈现错误消息（尽管这可能不是你想要做的）。根据数据库的不同，我们的代码可能如下所示：

```php
// For SQLite:
$dbh = sqlite_open('/path/to/cars.ldb', 0666) or die
('Error opening SQLite database: ' .
sqlite_error_string(sqlite_last_error($dbh)));
$q = sqlite_query("SELECT DISTINCT make FROM cars ORDER BY make",
$dbh) or die('Could not execute query because: ' .
sqlite_error_string(sqlite_last_error($dbh)));
// and, finally, for PostgreSQL:
pg_connect("host=localhost dbname=cars user=boss
password=password") or die('Could not connect to
PostgreSQL: . pg_last_error());
$q = pg_query("SELECT DISTINCT make FROM cars ORDER BY make")
or die('Could not execute query because: ' . pg_last_error());

```

如您所见，与 MySQL 和 PostgreSQL 相比，对于 SQLite 来说，错误处理开始有点不同。（请注意调用`sqlite_error_string (sqlite_last_error($dbh)).)`)

在我们看如何使用 PDO 实现相同的错误处理策略之前，我们应该注意，这将是 PDO 中三种可能的错误处理策略之一。我们将在本书的后面详细介绍它们。在这里，我们将只使用最简单的一个：

```php
// PDO error handling
// Assume the connection string is one of the following:
// $connStr = 'mysql:host=localhost;dbname=cars'
// $connStr = 'sqlite:/path/to/cars.ldb';
// $connStr = 'pgsql:host=localhost dbname=cars';
try
{
$conn = new PDO($connStr, 'boss', 'password');
}
catch(PDOException $pe)
{
die('Could not connect to the database because: ' .
$pe->getMessage();
}
$q = $conn->query("SELECT DISTINCT make FROM cars ORDER BY make");
if(!$q)
{
$ei = $conn->errorInfo();
die('Could not execute query because: ' . $ei[2]);
}

```

这个例子表明，PDO 会强制我们使用与传统错误处理方案略有不同的方案。我们将对 PDO 构造函数的调用包装在*try* … *catch*块中。（那些对 PHP5 的面向对象特性不熟悉的人应该参考附录 A。）这是因为虽然 PDO 可以被指示不使用异常（事实上，PDO 的默认行为是不使用异常），但是在这里你无法避免异常。如果构造函数调用失败，异常将总是被抛出。

捕获这个异常是一个非常好的主意，因为默认情况下，PHP 会中止脚本执行，并显示这样的错误消息：

**致命错误：未捕获的异常'PDOException'，消息为'SQLSTATE[28000] [1045] 用户'bosss'@'localhost'被拒绝访问（使用密码：YES）'，位于/var/www/html/pdo.php5:3 堆栈跟踪：#0 c:\www\hosts\localhost\pdo.php5(3)：PDO->__construct('mysql:host=loca...', 'bosss', 'password', Array) #1 {main} 在/var/www/html/pdo.php5 的第 3 行抛出**

我们通过在对 PDO 构造函数的调用中提供错误的用户名*bosss*来制造了这个异常。正如你从这个输出中看到的，它包含了一些我们不希望其他人看到的细节：像文件名和脚本路径，正在使用的数据库类型，最重要的是用户名和密码。假设这个异常发生在我们提供了正确的用户名并且数据库服务器出了问题的情况下。那么屏幕输出将包含真实的用户名和密码。

如果我们正确捕获异常，错误输出可能会像这样：

**SQLSTATE[28000] [1045] 用户'bosss'@'localhost'被拒绝访问（使用密码：YES）**

这个错误消息包含了更少的敏感信息。（事实上，这个输出与我们的非 PDO 示例中产生的错误输出非常相似。）但我们再次警告您，最好的策略是只显示一些中立的错误消息，比如：“抱歉，服务暂时不可用。请稍后再试。”当然，您还应该记录所有错误，以便以后找出是否发生了任何不好的事情。

## **预处理语句**

这是一个相当高级的话题，但你应该熟悉它。如果你是一个使用 PHP 与 MySQL 或 SQLite 的用户，那么你可能甚至没有听说过预处理语句，因为 PHP 的 MySQL 和 SQLite 扩展不提供这个功能。PostgreSQL 用户可能已经使用了`pg_prepare()`和`pg_execute()`。MySQLi（改进的 MySQL 扩展）也提供了预处理语句功能，但方式有些别扭（尽管可能是面向对象的风格）。

对于那些不熟悉**预处理语句**的人，我们现在将给出一个简短的解释。

当开发基于数据库的交互式动态应用程序时，您迟早会需要接受用户输入（可能来自表单），并将其作为查询的一部分传递给数据库。例如，给定我们的汽车数据库，您可能设计一个功能，将输出在任意两年之间制造的汽车列表。如果允许用户在表单中输入这些年份，代码将看起来像这样：

```php
// Suppose the years come in the startYear and endYear
// request variables:
$sy = (int)$_REQUEST['startYear'];
$ey = (int)$_REQUEST['endYear'];
if($ey < $sy)
{
// ensure $sy is less than $ey
$tmp = $ey;
$ey = $sy;
$sy = $tmp;
}
$sql = "SELECT * FROM cars WHERE year >= $sy AND year <= $ey";
// send the query in $sql…

```

在这个简单的例子中，查询依赖于两个变量，这些变量是结果 SQL 的一部分。在 PDO 中，相应的预处理语句看起来像这样：

```php
$sql = 'SELECT * FROM cars WHERE year >= ? AND year <= ?';

```

正如你所看到的，我们在查询体中用占位符替换了`$sy`和`$ey`变量。现在我们可以操作这个查询来创建预处理语句并执行它：

```php
// Assuming we have already connected and prepared
// the $sy and $ey variables
$sql = 'SELECT * FROM cars WHERE year >= ? AND year <= ?';
$stmt = $conn->prepare($sql);
$stmt->execute(array($sy, $ey));

```

这三行代码告诉我们，预处理语句是对象（具有类`PDOStatement`）。它们是使用调用`PDO::prepare()`方法创建的，该方法接受带有占位符的 SQL 语句作为其参数。

然后必须*执行*准备好的语句，以通过调用`PDOStatement::execute()`方法获取查询结果。正如示例所示，我们使用一个包含占位符值的数组来调用这个方法。请注意，该数组中变量的顺序与`$sql`变量中占位符的顺序相匹配。显然，数组中的元素数量必须与查询中占位符的数量相同。

您可能已经注意到，我们没有将对`PDOStatement::execute()`方法的调用结果保存在任何变量中。这是因为语句对象本身用于访问查询结果，这样我们就可以将我们的示例完善成这样：

```php
// Suppose the years come in the startYear and endYear
// request variables:
$sy = (int)$_REQUEST['startYear'];
$ey = (int)$_REQUEST['endYear'];
if($ey < $sy)
{
// ensure $sy is less than $ey
$tmp = $ey;
$ey = $sy;
$sy = $tmp;
}
$sql = 'SELECT * FROM cars WHERE year >= ? AND year <= ?';
$stmt = $conn->prepare($sql);
$stmt->execute(array($sy, $ey));
// now iterate over the result as if we obtained
// the $stmt in a call to PDO::query()
while($r = $stmt->fetch(PDO::FETCH_ASSOC))
{
echo "$r[make] $r[model] $r[year]\n";
}

```

正如这个完整的示例所示，我们调用`PDOStatement::fetch()`方法，直到它返回 false 值为止，此时循环退出——就像我们在讨论结果集遍历时的先前示例中所做的那样。

当然，用实际值替换问号占位符并不是准备好的语句唯一能做的事情。它们的强大之处在于可以根据需要执行多次。这意味着我们可以调用`PDOStatement::execute()`方法多次，每次都可以为占位符提供不同的值。例如，我们可以这样做：

```php
$sql = 'SELECT * FROM cars WHERE year >= ? AND year <= ?';
$stmt = $conn->prepare($sql);
// Fetch the 'new' cars:
$stmt->execute(array(2005, 2007));
$newCars = $stmt->fetchAll(PDO::FETCH_ASSOC);
// now, 'older' cars:
$stmt->execute(array(2000, 2004));
$olderCars = $stmt->fetchAll(PDO::FETCH_ASSOC);
// Show them
echo 'We have ', count($newCars), ' cars dated 2005-2007';
print_r($newCars);
echo 'Also we have ', count($olderCars), ' cars dated 2000-2004';
print_r($olderCars);

```

准备好的语句执行起来比调用`PDO::query()`方法要快，因为数据库驱动程序只会在调用`PDO::prepare()`方法时对它们进行优化一次。使用准备好的语句的另一个优点是，您不必引用在调用`PDOStatement::execute()`时传递的参数。

在我们的示例中，我们将请求参数显式转换为整数变量，但我们也可以这样做：

```php
// Assume we also want to filter by make
$sql = 'SELECT * FROM cars WHERE make=?';
$stmt = $conn->prepare($sql);
$stmt->execute(array($_REQUEST['make']));

```

这里的准备好的语句将负责在执行查询之前进行适当的引用。

最重要的一点是，PDO 为每个支持的数据库模拟了准备好的语句。这意味着您可以在任何数据库中使用准备好的语句；即使它们不知道这是什么。

## 对 PDO 的适当理解

如果我们不提到这一点，我们的介绍就不完整了。PDO 是一个数据库连接抽象库，因此不能保证您的代码对其支持的每个数据库都有效。只有当您的 SQL 代码是可移植的时，才会发生这种情况。例如，MySQL 使用以下形式的插入扩展了 SQL 语法：

```php
INSERT INTO mytable SET x=1, y='two';

```

这种 SQL 代码是不可移植的，因为其他数据库不理解这种插入方式。为了确保您的插入在各个数据库中都能正常工作，您应该用以下代码替换上面的代码：

```php
INSERT INTO mytable(x, y) VALUES(1, 'two');

```

这只是使用 PDO 时可能出现的不兼容性的一个例子。只有通过使数据库架构和 SQL 可移植，才能确保您的代码与其他数据库兼容。然而，确保这种可移植性超出了本文的范围。

# 总结

这个介绍性的章节向您展示了在使用 PHP5 语言开发动态、数据库驱动应用程序时使用 PDO 的基础知识。我们还看到了 PDO 如何有效地消除了不同传统数据库访问 API 之间的差异，并产生了更清晰、更可移植的代码。

在接下来的章节中，我们将更详细地讨论本章讨论的每个功能，以便您完全掌握 PHP 数据对象扩展。
