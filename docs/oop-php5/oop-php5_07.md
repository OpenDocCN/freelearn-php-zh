# 第七章：面向对象的数据库

除了在面向对象（OOP）方面的常规改进外，PHP5 还引入了许多新的库，以便以面向对象的方式无缝地与数据库工作。这些库为您提供了改进的性能，有时还提供了改进的安全性功能，当然还有一大堆与数据库服务器提供的新功能交互的方法。

在本章中，我们将讨论 MySQL 改进的 API，即 MySQLi。看看基本的 PDO（好吧，不是详细的，因为 PDO 如此庞大，以至于可以写一本书专门介绍它），ADOdb，以及 PEAR::MDB2。同时，我们还将看看使用 ADOdb 的 active 在 PHP 中实现的 Active Record 模式。在这里要注意的一点是，我们不是关注如何进行一般的数据库操作。我们只会关注一些对在面向对象方式中做数据库编程的 PHP 开发者来说有趣的具体主题。

# MySQLi 简介

MySQLi 是 PHP5 中引入的一个改进的扩展，用于处理高级 MySQL 功能，如预定义语句和存储过程。从性能角度来看，MySQLi 比 MySQL 扩展要好得多。此外，此扩展提供了完全面向对象的接口来与 MySQL 数据库交互，这在 PHP5 之前是不存在的。但请记住，如果你的 MySQL 版本至少是 4.1.3 或更高版本，你将能够使其工作。

那么主要的改进有哪些呢？让我们先看看：

+   性能优于 MySQL 扩展

+   灵活的面向对象和非面向对象接口

+   相较于新 MySQL 对象的优点

+   能够创建压缩连接

+   支持通过 SSL 连接

+   支持预定义语句

+   支持存储过程（SP）

+   支持更好的复制和事务

我们将在下面的例子中查看一些这些功能。但当然，我们不是要介绍 MySQL 的基础知识，因为这超出了本书的范围。我们只会展示如何使用 MySQLi 的面向对象接口，以及如何使用这些高级功能中的某些功能。

## 以面向对象的方式连接到 MySQL

记得那些老日子，那时候你不得不使用过程式函数`call`来连接 MySQL，即使是来自你的对象。那些日子已经过去了。现在你可以利用 MySQLi 的完整面向对象接口与 MySQL 进行通信（好吧，还有一些过程式方法，但总体上是完全面向对象的）。看看下面的例子：

```php
<?
$mysqli = new mysqli("localhost", "user", "password", "dbname");
if (mysqli_connect_errno()) {
   echo("Failed to connect, the error message is : ". 
                                          mysqli_connect_error());
   exit();
}
?>
```

如果连接失败，你可能会得到如下错误信息：

```php
Failed to connect, the error message is : Access denied for user 
                      'my_user'@'localhost' (using password: YES)
```

## 以面向对象的方式选择数据

让我们看看如何使用 MySQLi API 以面向对象的方式从表中选择数据。

```php
<?php
$mysqli = new mysqli("localhost", "un" "pwd", "db");
if (mysqli_connect_errno()) {
   echo("Failed to connect, the error message is : ". 
                                     mysqli_connect_error());
   exit();
}
/* close connection */
$result = $mysqli ->query("select * from users");
while ($data = $result->fetch_object())
{
  echo $data->name." : '".$data->pass."' \n";
}
?>
```

输出如下：

```php
robin : 'no password' 
tipu : 'bolajabena'

```

### 注意

请注意，在数据库中不加密地将用户密码以纯文本形式存储并不是一个好的做法。最好的方式是使用某些散列例程（如`md5()`）仅存储他们密码的散列。

## 以面向对象的方式更新数据

与它没有特别的处理。你可以像之前使用 MySQL 扩展一样更新你的数据。但为了面向对象风格的考虑，我们展示了如何使用 `mysqli_query()` 函数来执行，如上述示例所示。实例化一个 MySQLi 对象，然后运行查询。

## 预处理语句

这里我们进入了一个非常有趣的章节，这是在 PHP OO 中首次使用 MySQLi 扩展引入的。预处理语句是在 MySQL 5.0 版本（动态 SQL）中引入的，以提高安全性和灵活性。它比常规语句有显著的性能提升。

那么实际上什么是预处理语句？预处理语句只不过是一个由 MySQL 服务器预先编译的常规查询，可以在以后调用。预处理语句减少了 SQL 注入的可能性，并且与一般的非预处理查询相比提供了更好的性能，因为它不需要在运行时执行不同的编译步骤。（记住，它已经编译过了？）

使用预处理语句的优点如下：

+   更好的性能

+   预防 SQL 注入

+   在处理 BLOB 时节省内存

但也有缺点！

+   如果你只为单个调用使用预处理语句，那么不会有性能提升。

+   使用预处理语句没有查询缓存。

+   如果语句没有显式关闭，则可能会发生内存泄漏。

+   并非所有语句都可以用作预处理语句。

预处理语句可以在准备查询时接受参数，其顺序与准备查询时指定的顺序相同。在本节中，我们将学习如何创建预处理语句，向它们传递值，并获取结果。

### 基本预处理语句

让我们使用 PHP 的原生 MySQLi 扩展来准备一个语句。在以下示例中，我们将创建一个预处理语句，执行它，并从中获取结果：

```php
<?
$mysqli  = new mysqli("localhost", "un" "pwd", "db");
if (mysqli_connect_errno()) {
   echo("Failed to connect, the error message is : ". 
                                 mysqli_connect_error());
   exit();
}
$stmt = $mysqli ->prepare("select name, pass from users 
                                          order by name");

$stmt->execute();
//$name=null;
$stmt->bind_result($name, $pass);

while ($stmt->fetch())
{
  echo $name."<br/>";
}
?>
```

那么在上面的示例中我们实际上做了什么？

1.  我们使用以下代码准备语句：

    ```php
       $stmt = $mysqli->prepare("select name, pass from users order
                                                        by name");
    ```

1.  然后我们执行了它：

    ```php
    	$stmt->execute();
    ```

1.  然后，我们用它与两个变量绑定，因为我们的查询中有两个变量：

    ```php
    	$stmt->bind_result($name, $pass);
    ```

1.  最后我们使用以下方式获取结果：

```php
	$stmt->fetch()
```

每当我们调用 `fetch()` 时，绑定的变量会被填充值。因此，我们现在可以使用它们。

### 带变量的预处理语句

预处理语句的优势在于你可以使用变量与查询结合。首先，你可以在适当的位置放置一个 `?` 符号来准备查询，然后你可以在准备之后传递值。让我们看看以下示例：

```php
<?
$mysqli = new mysqli("localhost", "un" "pwd", "db");
if (mysqli_connect_errno()) {
   echo("Failed to connect, the error message is : ". 
                                   mysqli_connect_error());
   exit();
}
$stmt = $mysqli->prepare("select name, pass from users 
                                            where name=?");
$stmt->bind_param("s",$name); //binding name as string
$name = "tipu";
$stmt->execute();
$name=null;
$stmt->bind_result($name, $pass);

while ($r = $stmt->fetch())
{
  echo $pass."<br/>";
}
?>
```

这里我们准备查询 `"select name, pass from users where name=?`"，其中名称肯定是一个字符串类型的值。正如我们在前面的示例中使用 `bind_results()` 绑定参数以获取结果一样，这里我们必须使用 `bind_params()` 函数来绑定参数。除此之外，我们还需要提供绑定参数的数据类型。

MySQL 预处理语句支持四种类型的参数：

+   `i`，表示相应的变量是整数类型

+   `d`，表示相应的变量是双精度浮点数

+   `s` 表示相应的变量具有字符串类型

+   `b` 表示相应的变量是一个二进制大对象（blob），并且将以数据包的形式发送

由于我们的参数是字符串，我们使用了以下行来绑定参数：

```php
$stmt->bind_param("s",$name);
```

在绑定变量之后，我们现在将值设置为 `$name` 并调用 `execute()` 函数。之后，我们像之前一样检索值。

## 使用预处理语句与 BLOB

预处理语句支持高效地处理 **BLOB** 或 **二进制大对象**。如果你使用预处理语句管理 BLOB，它将通过以数据包的形式发送数据来节省你更多的内存消耗。让我们看看我们如何存储 BLOB（在这种情况下，是一个图像文件）。

预处理语句支持使用 `send_long_data()` 函数分块发送数据。在下面的例子中，我们将使用这个函数来存储图像，尽管你可以像通常一样发送它们，除非你的数据超过了由 `max_allowed_packet` MySQL 配置变量定义的限制。

```php
<?
$mysqli = new mysqli("localhost", "un" "pwd", "db");
if (mysqli_connect_errno()) {
   echo("Failed to connect, the error message is : ". 
                               mysqli_connect_error());
   exit();
}
$stmt = $mysqli->prepare("insert into images value(NULL,?)");
$stmt->bind_param("b",$image);
$image = file_get_contents("signature.jpg");//fetching content of 
//a file
$stmt->send_long_data(0,$image);
$stmt->execute();
?>
```

我们的表模式如下所示：

```php
CREATE TABLE 'images' (
  'id' int(11) NOT NULL auto_increment,
  'image' mediumblob,
  PRIMARY KEY  ('id')
) ENGINE=MyISAM;
```

我们选择中等 BLOB 作为我们的数据类型，因为 blob 只能存储 65KB 的数据，而中等 BLOB 可以存储超过 16MB，长 BLOB 可以在其中存储超过 4GB 的数据。

现在，我们将使用图像再次在预处理语句中恢复这个 BLOB 数据：

```php
<?
$mysqli = new mysqli("localhost", "username", "password", "test");
if (mysqli_connect_errno()) {
   echo("Failed to connect, the error message is : ". 
                                           mysqli_connect_error());
   exit();
}
$stmt = $mysqli->prepare("select image from images where id=?");
$stmt->bind_param("i",$id);
$id = $_GET['id'];
$stmt->execute();
$image=NULL;
$stmt->bind_result($image);
$stmt->fetch();
header("Content-type: image/jpeg");
echo $image;
?>

```

## 使用 MySQLi 和 PHP 执行存储过程

存储过程是 MySQL 5 中新增的另一个功能，它极大地减少了客户端查询的需求。使用 MySQLi 扩展，你可以在 MySQL 中执行存储过程。我们不会讨论存储过程，因为这超出了本书的范围。互联网上有几篇文章可以帮助你在 MySQL 中编写存储过程。你可以阅读这篇很棒的文章来获取关于高级 MySQL 功能的基本概念：[`dev.mysql.com/tech-resources/articles/mysql-storedprocedures.pdf`](http://dev.mysql.com/tech-resources/articles/mysql-storedprocedures.pdf)

让我们创建一个小型的存储过程并使用 PHP 来运行它。这个存储过程可以接受一个输入并将该记录插入到表中：

```php
DELIMITER $$;

DROP PROCEDURE IF EXISTS 'test'.'sp_create_user'$$

CREATE PROCEDURE 'sp_create_user'(IN uname VARCHAR(50))
BEGIN
INSERT INTO users(id,name) VALUES (null, uname); 
END$$

DELIMITER ;$$
```

如果你在这个数据库中运行这个存储过程（使用 MySQL 查询构建器或其他任何东西），`sp_create_user` 过程将被创建。

### 注意

你可以使用 "Execute" 命令手动从 MySQL 客户端执行任何已存储的过程。例如，要执行上述存储过程，你必须使用 `call sp_create_user('` *`username`*`')`。

现在，我们将使用 PHP 代码来运行这个存储过程。让我们看看。

```php
<?
$mysqli = new mysqli("localhost", "username", "password", "test");
if (mysqli_connect_errno()) {
   echo("Failed to connect, the error message is : ". 
                                mysqli_connect_error());
   exit();
}

$mysqli->query("call sp_create_user('hasin')");
?>
```

就这样！

# PDO

PHP 5.1 中新增的另一个用于管理数据库的扩展是 PDO（尽管 PDO 在 PHP 5.0 中作为 PECL 扩展可用）。它包含了一组用于与不同数据库引擎工作的驱动程序。**PDO** 代表 **PHP 数据对象**。它是为了提供不同数据库引擎的轻量级接口而开发的。PDO 的一个非常好的特性是它像数据访问层一样工作，这样你就可以为所有数据库引擎使用相同的函数名。

您可以使用 DSN（数据源名称）字符串连接到不同的数据库。在以下示例中，我们将连接到 MySQL 数据库并检索一些数据。

```php
<?php
$dsn = 'mysql:dbname=test;host=localhost;';
$user = 'user';
$password = 'password';

try {
   $pdo = new PDO($dsn, $user, $password);
} 
catch (PDOException $e) 
{
   echo 'Connection failed: ' . $e->getMessage();
}

$result = $pdo->query("select * from users");
foreach ($result as $row)
echo $row['name'];

?>
```

这相当方便，对吧？它只是通过 DSN（这里连接到 `test` 数据库）连接到 MySQL 服务器，然后执行查询。最后，我们显示结果。

如果我们连接到 SQLite 数据库会是什么样子呢？

```php
<?php
$dsn = 'sqlite:abcd.db';

try 
{
 $pdo = new PDO($dsn);
 $pdo->exec("CREATE TABLE users (id int, name VARCHAR)");
 $pdo->exec("DELETE FROM users");
 $pdo->exec("INSERT INTO users (name) VALUES('afif')");
 $pdo->exec("INSERT INTO users (name) VALUES('tipu')");
 $pdo->exec("INSERT INTO users (name) VALUES('robin')");
} 
catch (PDOException $e) {
   echo 'Connection failed: ' . $e->getMessage();
}

$result = $pdo->query("select * from users");
foreach ($result as $row)
echo $row['name'];

?>
```

看看代码中除了 DSN 之外没有变化。

您也可以在内存中创建 SQLite 数据库并在此处执行操作。让我们看一下以下代码：

```php
<?php
$dsn = 'sqlite::memory:';

try {
    $pdo = new PDO($dsn);
    $pdo->exec("CREATE TABLE users (id int, name VARCHAR)");
    $pdo->exec("DELETE FROM users");
    $pdo->exec("INSERT INTO users (name) VALUES('afif')");
    $pdo->exec("INSERT INTO users (name) VALUES('tipu')");
    $pdo->exec("INSERT INTO users (name) VALUES('robin')");
} 
catch (PDOException $e) 
{
   echo 'Connection failed: ' . $e->getMessage();
}

$result = $pdo->query("select * from users");
foreach ($result as $row)
echo $row['name'];

?>
```

我们只是在这里更改了 DSN。

## 不同数据库引擎的 DSN 设置

让我们看一下不同数据库引擎与 PDO 连接的 DSN 设置。支持的数据库驱动程序如下所示：

+   PDO_DBLIB 用于 FreeTDS/Microsoft SQL Server/Sybase

+   PDO_FIREBIRD 用于 Firebird/Interbase 6

+   PDO_INFORMIX 用于 IBM Informix Dynamic Server

+   PDO_MYSQL 用于 MySQL 3.x/4.x/5.x

+   PDO_OCI 用于 Oracle Call Interface

+   PDO_ODBC 用于 ODBC v3（IBM DB2、unixODBC 和 win32 ODBC）

+   PDO_PGSQL 用于 PostgreSQL

+   PDO_SQLITE 用于 SQLite 3 和 SQLite 2

让我们看一下这些示例驱动程序特定的 DSN 设置：

```php
mssql:host=localhost;dbname=testdb
sybase:host=localhost;dbname=testdb
dblib:host=localhost;dbname=testdb
firebird:User=john;Password=mypass;Database=DATABASE.GDE;
                         DataSource=localhost;Port=3050
informix:host=host.domain.com; service=9800;database=common_db; 
   server=ids_server; protocol=onsoctcp;EnableScrollableCursors=1

mysql:host=localhost;port=3307;dbname=testdb
mysql:unix_socket=/tmp/mysql.sock;dbname=testdb

oci:mydb
oci:dbname=//localhost:1521/mydb

odbc:testdb
odbc:DRIVER={IBM DB2 ODBC 
 DRIVER};HOSTNAME=localhost;PORT=50000;DATABASE=SAMPLE;PROTOCOL=TCPIP;
                                            UID=db2inst1;PWD=ibmdb2;
odbc:Driver={Microsoft Access Driver 
                            (*.mdb)};Dbq=C:\\db.mdb;Uid=Admin

pgsql:dbname=example;user=nobody;password=change_me;host=localhost;
                                            port=5432

sqlite:/opt/databases/mydb.sq3
sqlite::memory:
sqlite2:/opt/databases/mydb.sq2
sqlite2::memory:
```

## 使用 PDO 的预处理语句

使用 PDO 对您的数据库运行预处理语句。好处与之前相同。它通过解析和缓存服务器端查询来提高多次调用的性能，同时也消除了 SQL 注入的机会。

与我们之前在 MySQLi 的示例中看到的不同，PDO 预处理语句可以接受命名变量。

让我们看一下以下示例来理解这一点：

```php
<?php
$dsn = 'mysql:dbname=test;host=localhost;';
$user = 'username';
$password = 'password';

try {
    $pdo = new PDO($dsn, $user, $password);
} catch (PDOException $e) 
{
   echo 'Connection failed: ' . $e->getMessage();
}

$stmt = $pdo->prepare("select id from users where name=:name");
$name = "tipu";
$stmt->bindParam(":name",$name, PDO::PARAM_STR);
$stmt->execute();
$stmt->bindColumn("id",$id);
$stmt->fetch();
echo $id;
?>
```

但您也可以像这样运行示例：

```php
<?php
$dsn = 'mysql:dbname=test;host=localhost;';
$user = 'username';
$password = 'password';

try {
    $pdo = new PDO($dsn, $user, $password);
} 
catch (PDOException $e) 
{
   echo 'Connection failed: ' . $e->getMessage();
}

$stmt = $pdo->prepare("select id from users where name=?");
$name = "tipu";
$stmt->bindParam(1,$name, PDO::PARAM_STR);
$stmt->execute();

$stmt->bindColumn("id",$id);
$stmt->fetch();
echo $id;
?>
```

与调用 `bindParam()` 不同，您可以使用以下类似的 `bindValues()`：

```php
$stmt->bindValue(1,"tipu", PDO::PARAM_STR);
```

## 调用存储过程

PDO 提供了一种简单的方法来调用存储过程。您只需通过 `exec()` 方法运行 "CALL SPNAME(PARAMS)" 即可：

```php
$pdo->exec("CALL sp_create_user('david')");
```

## 其他有趣的函数

PDO 中还有其他几个有趣的函数可用。例如，看一下下面的列表：

+   `fetchAll()`

+   `fetchColumn()`

+   `rowCount()`

+   `setFetchMode()`

`fetchAll()` 函数可以从结果集中检索所有记录。让我们看一下以下示例：

```php
$stmt = $pdo->prepare("select * from users");
$stmt->execute();
echo "<pre>";
print_r($stmt->fetchAll());
echo "</pre>";
```

`fetchColumn()` 函数有助于在执行语句后从任何特定列中选择数据。让我们看一下：

```php
$stmt = $pdo->prepare("select * from users");
$stmt->execute();
while ($name = $stmt->fetchColumn(1))
{
  echo $name."<br/>";
}
```

`rowCount()` 在执行任何 `UPDATE` 或 `DELETE` 查询后返回受影响的行数。但您必须记住，它返回的是最新执行的查询影响的行数。

```php
$stmt = $pdo->prepare("DELETE from users WHERE name='Anonymous'");
$stmt->execute();
echo $stmt->rowCount();
```

`setFetchMode()` 帮助您设置 PDO 预处理语句的检索模式。可用的值包括：

+   `PDO::FETCH_NUM`: 以数字索引数组的形式检索结果

+   `PDO::FETCH_ASSOC`: 以列名作为键检索行

+   `PDO::FETCH_BOTH`: 以上述两种方式检索

+   `PDO::FETCH_OBJ`: 将行作为对象检索，其中列名设置为属性

# 数据抽象层简介

**数据抽象层（DALs**）是为了提供统一的接口以与每个数据库引擎一起工作而开发的。它为每个数据库引擎提供类似的 API 进行独立操作。由于所有平台上的函数名称相似，因此它们更容易使用，更容易记忆，当然也使得代码更易于移植。为了让您了解 DAL 的必要性，让我解释一个常见的场景。

假设 Y 团队接到了一个大项目。他们的客户说他们将使用 MySQL。因此，Y 团队开发了应用程序，当交付时间到来时，客户要求团队提供对 PostgreSQL 的支持。他们将为此变更付费，但需要尽早完成。

Y 团队使用所有原生 MySQL 函数设计了应用程序。那么 Y 团队会怎么做？他们会重写一切以支持 PostgreSQL 吗？嗯，这是他们唯一的选择。但如果他们未来需要支持 MSSQL 呢？另一个重写？你能想象每次重构的成本吗？

为了避免这些灾难，就需要有 DAL，这样代码就可以保持不变，并且可以在任何时间更改以支持任何数据库，而无需进行任何重大更改。

有许多流行的库用于实现 PHP 的 DAL。以下是一些例子，如 ADOdb 和 PEAR::MDB2 非常流行。PEAR::DB 曾经非常流行，但其开发已经停止（[`blog.agoraproduction.com/index.php?/archives/42-PEARDB-is-DEPRECATED,-GOT-IT.html#extended`](http://blog.agoraproduction.com/index.php?/archives/42-PEARDB-is-DEPRECATED,-GOT-IT.html#extended)）。

在本节中，我们将讨论 PEAR::MDB2 和 ADOdb。我们将使用它查看基本的数据库操作，并学习如何安装这些库以进行工作。

## ADOdb

ADOdb 是由 John Lim 开发的一个非常好且流行的数据抽象层，并发布在 LGPL 下。这是 PHP 中最好的数据抽象层之一。您可以从[`adodb.sourceforge.net`](http://adodb.sourceforge.net)获取 ADOdb 的最新版本。

### 安装 ADOdb

没有 ADodb 的安装。它是一组类和常规脚本。所以您只需将存档提取到可以包含脚本的位置即可。让我们看一下以下图像，了解提取后的目录结构：

![安装 ADOdb](img/2561_07_01.jpg)

### 连接到不同的数据库

与 PDO 一样，您可以使用 ADOdb 连接到不同的数据库驱动程序。DSN 与 PDO 不同。让我们看一下支持的数据库列表及其 DSN 字符串。

ADOdb 支持常见的 DSN 格式，如下所示：

```php
$driver://$username:$password@hostname/$database?options[=value]
```

那么，ADOdb 支持哪些可用的驱动程序呢？下面让我们看一下。这是一个从 ADOdb 手册中摘取的列表，供您理解：

| 名称 | 测试 | 数据库 | 预先条件 | 操作系统 |
| --- | --- | --- | --- | --- |
| 访问 | B | Microsoft Access/Jet。您需要创建一个 ODBC DSN。 | ODBC | 仅限 Windows |
| ado | B | 通用 ADO，未针对特定数据库进行优化。允许无 DSN 连接。为了最佳性能，请使用 OLEDB 提供程序。这是所有 ado 驱动的基类。您可以在连接之前设置 `$db->codePage`。 | ADO 或 OLEDB 提供程序 | 仅限 Windows |
| ado_access | B | 使用 ADO 的 Microsoft Access/Jet。允许无 DSN 连接。为了最佳性能，请使用 OLEDB 提供程序。 | ADO 或 OLEDB 提供程序 | 仅限 Windows |
| ado_mssql | B | 使用 ADO 的 Microsoft SQL Server。允许无 DSN 连接。为了最佳性能，请使用 OLEDB 提供程序。 | ADO 或 OLEDB 提供程序 | 仅限 Windows |
| db2 | C | 使用 PHP 的 db2 特定扩展以获得更好的性能。 | DB2 CLI/ODBC 接口 | Unix 和 Windows。需要 IBM DB2 Universal Database 客户端 |
| odbc_db2 | C | 使用通用的 ODBC 扩展连接到 DB2。 | DB2 CLI/ODBC 接口 | Unix 和 Windows。Unix 安装提示。我收到报告说在使用 CLI 接口时，在 `Connect()` 中必须将 `$host` 和 `$database` 参数颠倒过来。 |
| vfp | A | Microsoft Visual FoxPro。您需要创建一个 ODBC DSN。 | ODBC | 仅限 Windows |
| fbsql | C | FrontBase。 | ? | Unix 和 Windows |
| ibase | B | Interbase 6 或更早版本。一些用户报告说您可能需要使用此 `$db->PConnect('localhost:c:/ibase/employee.gdb', "sysdba", "masterkey")` 来连接。目前缺少 `Affected_Rows`。您可以在连接之前设置 `$db->role`，`$db->dialect`，`$db->buffers` 和 `$db->charSet`。 | Interbase 客户端 | Unix 和 Windows |
| firebird | B | Firebird 版本的 interbase。 | Interbase 客户端 | Unix 和 Windows |
| borland_ibase | C | Borland 版本的 Interbase 6.5 或更高版本。非常遗憾分支不同。 | Interbase 客户端 | Unix 和 Windows |
| informix | C | 通用 informix 驱动程序。如果您使用的是 Informix 7.3 或更高版本，请使用此驱动程序。 | Informix 客户端 | Unix 和 Windows |
| informix72 | C | 不支持 `SELECT FIRST` 的 Informix 7.3 之前的数据库。 | Informix 客户端 | Unix 和 Windows |
| ldap | C | LDAP 驱动程序。有关使用信息，请参阅此示例。 | LDAP 扩展 | ? |
| mssql | A | Microsoft SQL Server 7 及更高版本。也适用于 Microsoft SQL Server 2000。注意，此驱动程序中的日期格式有问题。例如，PHP MSSQL 扩展不返回 datetime 的秒数！ | Mssql 客户端 | Unix 和 Windows。Unix 安装指南和另一个指南。 |
| mssqlpo | A | 可移植的 mssql 驱动程序。与上述 mssql 驱动程序相同，除了将 `' | | '`，连接运算符，转换为 `+`。对于从大多数其他使用 `' | | '` 的 sql 变体移植脚本非常有用。 | Mssql 客户端 | Unix 和 Windows。Unix 安装指南。 |
| mysql | A | 无事务支持的 MySQL。您也可以在连接之前设置 `$db->clientFlags`。 | MySQL 客户端 | Unix 和 Windows |
| mysqli | B | 支持较新的 PHP5 MySQL API。 | MySQL 4.1+ 客户端 | Unix 和 Windows |
| mysqlt or maxsql | A | 具有事务支持的 MySQL。我们建议使用 ` | | ` 作为连接运算符以获得最佳兼容性。可以通过以下方式运行 MySQL：`mysqld --ansi` 或 `mysqld --sql-mode=PIPES_AS_CONCAT`。 | MySQL 客户端 | Unix 和 Windows |
| oci8 | A | Oracle 8/9。比 oracle 驱动程序具有更多功能（例如 `Affected_Rows`）。您可能需要在 Connect/PConnect 之前执行 `putenv('ORACLE_HOME=...')`。有两种连接方式：使用服务器 IP 和服务名称：`PConnect('serverip:1521','scott','tiger','service')` 或使用 TNSNAMES.ORA 或 ONAMES 或 HOSTNAMES 中的条目：`PConnect(false, 'scott', 'tiger', $oraname)`。自 2.31 版本起，我们直接支持 Oracle REF cursor 变量（请参阅 `ExecuteCursor`）。 | Oracle 客户端 | Unix 和 Windows |
| oci805 | C | 支持 Oracle 8.0.5 的简化功能。`SelectLimit` 在 oci8 或 oci8po 驱动程序中效率不高。 | Oracle 客户端 | Unix 和 Windows |
| oci8po | A | Oracle 8/9 可移植驱动程序。此驱动程序与 oci8 驱动程序几乎相同，除了（a）`Prepare()` 中的绑定变量使用 `?` 约定，而不是 `:bindvar`，（b）字段名称使用更常见的 PHP 约定的小写名称。如果从其他数据库迁移很重要，请使用此驱动程序。否则，oci8 驱动程序提供更好的性能。 | Oracle 客户端 | Unix 和 Windows |
| odbc | A | 通用 ODBC，未针对特定数据库进行优化。要连接，请使用 `PConnect('DSN','user','pwd')`。这是所有 ODBC 派生驱动程序的基础类。 | ODBC | Unix 和 Windows。Unix 提示 |
| odbc_mssql | A | 使用 ODBC 连接到 MSSQL。 | ODBC | Unix 和 Windows |
| odbc_oracle | C | 使用 ODBC 连接到 Oracle。 | ODBC | Unix 和 Windows |
| odbtp | B | 通用 odbtp 驱动程序。Odbtp 是一种从其他操作系统访问 Windows ODBC 数据源的软件。 | odbtp | Unix 和 Windows |
| odbtp_unicode | C | 支持 Unicode 的 odbtp。 | odbtp | Unix 和 Windows |
| oracle | C | 实现旧的 Oracle 7 客户端 API。如果可能，请使用 oci8 驱动程序以获得更好的性能。 | Oracle 客户端 | Unix 和 Windows |
| netezza | C | Netezza 驱动程序。Netezza 基于 PostGREs 代码库。 | ? | ? |
| pdo | C | PHP5 的通用 PDO 驱动程序。 | PDO 扩展和特定数据库驱动程序 | Unix 和 Windows |
| postgres | A | 通用 PostgreSQL 驱动程序。目前与 postgres7 驱动程序相同。 | PostgreSQL 客户端 | Unix 和 Windows |
| postgres64 | A | 用于 PostgreSQL 6.4 及更早版本，这些版本不支持内部 LIMIT。 | PostgreSQL 客户端 | Unix 和 Windows |
| postgres7 | A | 支持 LIMIT 和其他版本 7 功能的 PostgreSQL。 | PostgreSQL 客户端 | Unix 和 Windows |
| postgres8 | A | 目前与 postgres7 相同。 | PostgreSQL 客户端 | Unix 和 Windows |
| sapdb | C | SAP DB。基于 ODBC 驱动程序，应该能够可靠地工作。 | SAP ODBC 客户端 | ? |
| sqlanywhere | C | Sybase SQL Anywhere。基于 ODBC 驱动程序，应该能够可靠地工作。 | SQL Anywhere ODBC 客户端 | ? |
| sqlite | B | SQLite。 | - | Unix 和 Windows |
| sqlitepo | B | 可移植 SQLite 驱动。这是因为关联模式在 SQLite 中不像其他驱动那样工作。具体来说，当选择（连接）多个表时，表名包含在“sqlite”驱动程序的关联键中。在“sqlitepo”驱动程序中，表名从返回的列名中去除。当这导致冲突时，第一个字段将优先。 | - | Unix 和 Windows |
| sybase | C | Sybase. | Sybase 客户端 | Unix 和 Windows |

### 使用 ADOdb 进行基本数据库操作

记得刚才看到的目录结构吗？现在我们将利用那些脚本。在本节中，我们将学习使用 ADOdb 进行基本数据库操作。让我们连接到 MySQL 并执行一个基本操作：

```php
<?
include("adodb/adodb.inc.php");
$dsn = 'mysql://username:password@localhost/test?persist';
$conn = ADONewConnection($dsn); 
$conn->setFetchMode(ADODB_FETCH_ASSOC);
$recordSet = $conn->Execute('select * from users');

if (!$recordSet)
print $conn->ErrorMsg(); //if any error is there
else
while (!$recordSet->EOF) {
  echo $recordSet->fields['name'].'<BR>';
  $recordSet->MoveNext();
} 
?>
```

让我们看看一个替代的连接示例：

```php
<?
include("adodb/adodb.inc.php");
$conn =ADONewConnection('mysql');//just the RDBMS type 
$conn->connect("localhost","username","password","test");
//here comes the credentials
?>
```

### 插入、删除和更新记录

你可以使用`ADONewConnection`或`ADOConnection`对象的`execute()`方法执行任何 SQL 语句。所以这里没有什么新东西。但让我们看看我们如何插入、删除和更新一些记录，并跟踪成功或失败。

```php
<?
include("adodb/adodb.inc.php");
$conn =ADONewConnection('mysql');
$conn->connect("localhost","user","password","test");
$conn->setFetchMode(ADODB_FETCH_ASSOC);
$res = $conn->execute("insert into users(name) values('test')");
echo $conn->Affected_Rows();
?>
```

因此，`Affected_Rows`为你提供了这些场景的结果。

### 提示

**插入 ID**

如果你正在寻找最新的插入 ID，你可以使用`Insert_Id()`函数。

### 执行预定义语句

ADOdb 提供了创建和执行预定义语句的简单 API。让我们看看以下示例，了解它是如何工作的：

```php
<?
include("adodb/adodb.inc.php");
$conn =ADONewConnection('mysql');
$conn->connect("localhost","user","password","test") ;
$conn->setFetchMode(ADODB_FETCH_ASSOC);
$stmt = $conn->Prepare('insert into users(name) values (?)');

$conn->Execute($stmt,array((string) "afif"));

echo $conn->Affected_Rows();
?>
```

你可以用同样的方式检索记录。

## MDB2

MDB2 是另一个在 PEAR 下开发的流行数据抽象库，它结合了 PEAR::DB 和 Metabase 的最佳特性。它提供了非常一致的 API、改进的性能和稳定的开发平台，优于 DB 和 MDB。MDB2 附带了一套优秀的文档。在本章中，我们当然不能涵盖 MDB2 支持的所有功能，但我们将介绍基本功能，以便你了解它是如何工作的。

### 安装 MDB2

安装 MDB2 需要安装一个有效的 PEAR 版本。因此，为了使用 MDB2，你必须确保 PEAR 已安装并能在你的机器上正常工作。如果你没有安装 PEAR，以下提示将对你有所帮助。

### 提示

**安装 PEAR**

前往[`pear.php.net/go-pear`](http://pear.php.net/go-pear)，并将页面保存为`go-pear.php`到你的硬盘上。现在在你的 shell 或命令提示符中执行命令`php /path/to/go-pear.php`，并按照那里的说明操作。如果询问你是否要安装 MDB2，请回答“是”。如果它要求修改你的`php.ini`文件，也请回答“是”。不用担心，它只会添加条目以使 PEAR 在你的当前包含路径中可用，所有其他设置都将保持不变。所以，你已经完成了。

如果你已经安装了 PEAR 但没有安装 MDB2，那么你可以立即安装它。打开你的 shell 或命令提示符，并执行以下命令：

```php
pear install MDB2
pear install MDB2_Driver_$driver
```

其中 `$driver` 可以是任何类似 SQLite、PgSQL、MySQL、MYSQLi、oci8、MSSQL 和 ibase 等。例如，要安装 MySQL 驱动，你必须执行以下命令：

```php
pear install MDB2_Driver_mysql
```

就这样。你已经完成了。

### 连接到数据库

使用 MDB2，你可以连接到不同的数据库引擎。MDB2 还有一个格式化的 DSN 字符串来连接。该 DSN 的格式如下所示：

```php
phptype(dbsyntax)://username:password@protocol+hostspec/database?
                                                      option=value
```

但在这个 DSN 中也有一些变化。这些变化在此列出：

```php
phptype://username:password@protocol+hostspec:110//usr/db_file.db
phptype://username:password@hostspec/database
phptype://username:password@hostspec
phptype://username@hostspec
phptype://hostspec/database
phptype://hostspec
phptype:///database
phptype:///database?option=value&anotheroption=anothervalue
```

支持的驱动程序（PHPtype）如下所示：

```php
fbsql  -> FrontBase
ibase  -> InterBase / Firebird (requires PHP 5)
mssql  -> Microsoft SQL Server (NOT for Sybase. Compile PHP --with-mssql)
mysql  -> MySQL
mysqli -> MySQL (supports new authentication protocol) (requires PHP 5)
oci8   -> Oracle 7/8/9/10
pgsql  -> PostgreSQL
querysim -> QuerySim
sqlite -> SQLite 2
```

现在，让我们连接到 MySQL：

```php
<?php
set_include_path(get_include_path().";".
                   "C:/Program Files/PHP/pear;");
require_once 'MDB2.php';

$dsn = 'mysql://user:password@localhost/test';
$options = array('persistent' => true
);

$mdb2 = MDB2::factory($dsn, $options);
if (PEAR::isError($mdb2)) {
    die($mdb2->getMessage());
}

// ...

$result = $mdb2->query("select * from users");

while ($row = $result->fetchRow(MDB2_FETCHMODE_ASSOC))
{
	echo $row['name']."\n";
}

$mdb2->disconnect();
?>

```

### 执行预定义语句

你可以轻松地使用 MDB2 执行预定义语句。MDB2 提供了创建和执行预定义语句的灵活 API。在以下示例中，我们将执行两种类型的预定义语句。一种将仅执行一些插入/更新/删除查询，另一种将返回一些数据作为输出。

```php
<?php
set_include_path(get_include_path().";".
                         "C:/Program Files/PHP/pear;");
require_once 'MDB2.php';

$dsn = 'mysql://user:password@localhost/test';
$options = array('persistent' => true
);

$mdb2 = MDB2::factory($dsn, $options);
if (PEAR::isError($mdb2)) {
    die($mdb2->getMessage());
}

$stmt = $mdb2->Prepare("insert into users(name) 
                    values(?)",array("text"),MDB2_PREPARE_MANIP);
//for DML statements, we should use MDB2_PREPARE_MANIP and For 
//Reading we should use MDB2_PREPARE_RESULT
echo $stmt->execute("Mohiuddin");

$stmt = $mdb2->Prepare("select name from users where 
                       id=?",array("integer"),array("text"));
$result = $stmt->execute(11);
if (PEAR::isError($result))
echo $result->getMessage();

while ($row = $result->fetchRow())
{
  echo $row[0];
}
?>

```

现在如果我们想在多个字段中插入数据呢？例如，如果我们表中还有一个名为"age"的字段，我们需要传递如下数据：

```php
$stmt = $mdb2->Prepare("insert into users(name,age) 
   values(?)",array("text","integer"),MDB2_PREPARE_MANIP);
echo $stmt->execute("Mohiuddin",2);
```

或者：

```php
$stmt = $mdb2->Prepare("insert into users(name,age) 
   values(?)",array("text","integer"),MDB2_PREPARE_MANIP);
echo $stmt->execute(array("Mohiuddin",2));
```

因此，我们也可以使用`executeMultiple()`方法一次性插入多行：

```php
$stmt = $mdb2->Prepare("insert into users(name,age) values(?)",array("text","integer"),MDB2_PREPARE_MANIP);
echo $stmt->executeMultiple(array(array("Mohiuddin",2),
                                         array("another",3));
```

就这样。

# ActiveRecord 简介

ActiveRecord 是一种设计模式，旨在以相当可读的方式解决数据访问问题。使用 ActiveRecord 设计模式，你可以像魔法一样操纵数据。在本节中，我们将介绍 PHP 中 ActiveRecord 实现的基本功能。

让我们看看 ActiveRecord 实际上是如何工作的。为此，我们将使用 ADOdb 的 Active Record 实现。Adodb 提供了一个名为`Adodb_Active_Record`的类，专门用于此。

让我们在数据库中创建一个具有以下结构的表：

```php
CREATE TABLE 'users' (
  'id' int(11) NOT NULL auto_increment,
  'name' varchar(250),
  'pass' varchar(32),
  PRIMARY KEY  ('id')
) ENGINE=MyISAM;
```

## 通过 ActiveRecord 创建新记录

现在，我们将在这个表中创建一个新的用户。看看以下代码：

```php
<?
include("adodb/adodb.inc.php");
include('adodb/adodb-active-record.inc.php');
$conn =ADONewConnection('mysql');
$conn->connect("localhost","user","password","test") ;

ADODB_Active_Record::setDatabaseAdapter($conn);
class User extends ADODB_Active_Record {}
$user = new User();//a dynamic model to access the user table
$user->name = "Packt";
$user->pass = "Hello";
$user->save();//calling save() will internally save this 
      //record in table
?>
```

ActiveRecord 通过为你的数据库中的每个表暴露一个单独的对象，你可以执行不同的操作。让我们看看我们如何选择一些数据。

## 选择和更新数据

我们可以使用 ActiveRecord 轻松地加载和更改任何记录。让我们看看以下示例：

```php
<?
include("adodb/adodb.inc.php");
include('adodb/adodb-active-record.inc.php');
$conn =ADONewConnection('mysql');
$conn->connect("localhost","user","password","test") ;

ADODB_Active_Record::setDatabaseAdapter($conn);
class User extends ADODB_Active_Record {}
$user = new User();
$user->load("id=10");//load the record where the id is 10
echo $user->name;
$user->name= "Afif Mohiuddin";//now update
$user->save();//and save the previously loaded record
?>
```

所以这相当简单。当你用任何表达式调用`load()`方法时，记录将被加载到对象本身中。然后你可以进行任何更改，最后保存它。ActiveRecord 与工作起来非常迷人。

# 摘要

你已经阅读了一章，专门介绍使用面向对象的方式完全访问数据库。还有很多其他有趣的项目，如 Propel（[`propel.phpdb.org/trac/`](http://propel.phpdb.org/trac/)）作为 PHP 开发者的对象关系映射库，Creole（[`creole.phpdb.org/trac/`](http://creole.phpdb.org/trac/)）作为 DAL，CodeIgniter 框架中的 ActiveRecord 库（[`www.codeigniter.com`](http://www.codeigniter.com)），等等。你有很多资源可以使用 PHP5 和面向对象风格来操纵数据库。

在下一章中，我们将学习如何在 PHP 中使用 XML。你会惊讶地发现，你可以使用纯 XML 文件作为常规重型数据库引擎的轻量级替代品。在此之前，祝大家编码愉快。
