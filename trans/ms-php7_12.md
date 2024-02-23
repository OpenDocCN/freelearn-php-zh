# 与数据库一起工作

PHP 语言对多种不同的数据库有很好的支持。自 PHP 语言早期以来，MySQL 一直被 PHP 开发人员视为首选数据库。虽然最初的重点主要是**关系型数据库管理系统**（**RDBMS**），但其他类型的数据库在现代应用程序中同样（或更）重要。自文档和数据键值数据库以来，它们的受欢迎程度一直在增长。

如今，看到一个 PHP 应用程序同时使用 MySQL、Mongo、Redis，可能还有其他几个数据库或数据存储是很常见的。

Mongo 的 NoSQL（“非 SQL”，“非关系”或“不仅仅是 SQL”）特性允许构建生成大量新数据类型的应用程序，这些数据类型可能会迅速变化。摆脱了**SQL**（**结构化查询语言**）的严格性，使用结构化、半结构化、非结构化和多态数据与 Mongo 数据库一起成为全新的体验。像 Redis 这样的内存数据结构存储器以速度为目标，这使它们非常适合缓存和消息代理系统。

在本章中，我们将通过以下部分更详细地了解 MySQL、Mongo 和 Redis：

+   使用 MySQL

+   安装 MySQL

+   设置示例数据

+   通过 mysqli 驱动程序扩展进行查询

+   通过 PHP 数据对象驱动程序扩展进行查询

+   使用 MongoDB

+   安装 MongoDB

+   设置示例数据

+   通过 MongoDB 驱动程序扩展进行查询

+   使用 Redis

+   安装 Redis

+   设置示例数据

+   通过 phpredis 驱动程序扩展进行查询

在本章中，我们为三个数据库服务器提供了快速安装说明。这些说明在相对基本的水平上给出，没有进行通常在生产类型机器上进行的任何后安装配置或调整。这里的一般想法只是让开发者的机器能够运行每个数据库服务器。

# 使用 MySQL

MySQL 是一个开源的关系型数据库管理系统，已经存在了 20 多年。最初由瑞典公司 MySQL AB 开发和拥有，现在由 Oracle Corporation 拥有。MySQL 的当前稳定版本是 5.7。

MySQL 的一些关键优势可以概括如下：

+   跨平台，在服务器上运行

+   可用于桌面和 Web 应用程序

+   快速、可靠且易于使用

+   适用于小型和大型应用程序

+   使用标准 SQL

+   支持查询缓存

+   支持 Unicode

+   在使用 InnoDB 时的 ACID 兼容性

+   在使用 InnoDB 时的事务

# 安装 MySQL

假设我们使用的是新的 Ubuntu 16.10（Yakkety Yak）安装，以下步骤概述了我们如何设置 MySQL：

1.  要安装 MySQL，我们执行以下控制台命令：

```php
sudo apt-get update
sudo apt-get -y install mysql-server

```

1.  安装过程会触发一个控制台 GUI 界面，要求我们输入`root`用户密码：

![](img/7f664b9e-a662-4d17-9720-59a7faa92dcf.png)

1.  提供的密码需要重复以确认：

![](img/15b71313-29a0-484f-a4dd-dfe8720e51b7.png)

1.  安装完成后，我们可以执行以下`mysql --version`命令来确认 MySQL 服务器是否正在运行：

```php
root@vultr:~# mysql --version
mysql Ver 14.14 Distrib 5.7.17, for Linux (x86_64) using EditLine wrapper

```

1.  服务器运行后，我们需要保护安装。通过运行以下命令来完成：

```php
sudo mysql_secure_installation

```

1.  安全安装过程会触发一个交互式 shell，要求提供以下信息：

+   输入 root 用户的密码：

+   是否要设置 VALIDATE PASSWORD 插件？

+   请输入 0 = 低，1 = 中等和 2 = 强：

+   新密码：

+   重新输入新密码：

+   删除匿名用户？

+   禁止远程 root 登录？

+   删除测试数据库和对其的访问？

+   现在重新加载权限表？

以下截图描述了这个过程：

![](img/d8d43bee-0866-40bc-833e-396789e3bab1.png)查看[`dev.mysql.com/doc/refman/5.7/en/validate-password-plugin.html`](https://dev.mysql.com/doc/refman/5.7/en/validate-password-plugin.html)获取有关密码验证插件的更多信息。

1.  安装安全完成后，我们可以继续并使用`mysql`控制台工具连接到 MySQL，如下所示：

```php
// INSECURE WAY (bare passwords in a command)
mysql -uroot -p'mL08e!Tq'
mysql --user=root --password='mL08e!Tq'

// SECURE WAY (triggers "enter password" prompt)
mysql -uroot -p
mysql --user=root --password

```

请注意在密码周围使用单引号字符（`'`）。虽然通常我们可以使用`"`或`'`引号，但密码中使用的`!`字符强制我们使用`'`。在这种情况下，如果不用单引号括起密码，我们将看到类似于!Tq: event not found 的错误。这是因为感叹号（`!`）是 bash 中的历史扩展的一部分。为了将其用作密码的一部分，我们需要将其括在单引号中。此外，我们的密码可以包含`'`或`"`字符。为了转义密码中的这些引号，我们可以使用前导反斜杠（\），或者用相反样式的引号将整个参数括起来。然而，解决古怪密码字符的最简单和最安全的方法是避免使用`-p`或`--password`参数分配密码值，并通过`输入密码：`提示提供密码。

这应该给我们以下输出：

![](img/ace8ab7e-822e-4aa2-aeee-6539546b5240.png)查看[`dev.mysql.com/doc/refman/5.7/en/mysql-shell.html`](https://dev.mysql.com/doc/refman/5.7/en/mysql-shell.html)获取有关 MySQL shell 的更多信息。

# 设置示例数据

在我们继续查询 MySQL 之前，让我们先设置一些示例数据。MySQL 提供了一个名为 Sakila 的示例数据库，我们可以从官方 MySQL 网站下载，如下所示：

```php
cd ~
wget http://downloads.mysql.com/docs/sakila-db.tar.gz
tar -xzf sakila-db.tar.gz
cd sakila-db/

```

下载并解压缩后，这应该给我们以下三个文件：

![](img/ea9fa1b3-cc77-4f8b-be21-392a783d5243.png)

接下来，我们需要看看如何导入`sakila-schema.sql`和`sakila-data.sql`。幸运的是，MySQL 提供了几种方法来做到这一点。快速查看`sakila-schema.sql`文件显示了文件顶部的以下条目：

```php
DROP SCHEMA IF EXISTS sakila;
CREATE SCHEMA sakila;
USE sakila;

```

这意味着`sakila-schema.sql`文件将为我们创建一个模式（数据库），并将其设置为当前使用的数据库。这是一个重要的部分需要理解，因为并非所有的`.sql` / 备份文件都会有这个，我们将被迫手动执行这一部分。了解`sakila-schema.sql`如何处理我们需要导入的所有内容后，以下命令显示了我们可以使用的三种不同方法：

```php
// Either this command
mysql -uroot -p < sakila-schema.sql

// Either this command
mysql -uroot -p -e "SOURCE sakila-schema.sql" 

```

第二个命令使用`-e` (`--execute`)参数将 SQL 语句传递给服务器。我们本可以轻松地在交互式中使用`mysql`工具，然后在其中执行`SOURCE sakila-schema.sql`。有了架构，我们可以继续导入实际数据：

```php
// Either this command
mysql -uroot -p < sakila-data.sql

// Either this command
mysql -uroot -p -e "SOURCE sakila-data.sql" 

```

如果我们现在交互式使用`mysql`工具，我们可以检查数据库是否成功导入：

```php
show databases;
use sakila;
show tables;

```

这应该给我们以下输出：

![](img/2b3df6df-212e-4f2d-919f-029daa68b9c7.png)查看[`dev.mysql.com/doc/sakila/en/`](https://dev.mysql.com/doc/sakila/en/)获取有关 Sakila 示例数据库的更多信息。

# 通过 MySQLi 驱动程序扩展查询

有几个驱动程序扩展允许我们查询 MySQL。MySQLi 是其中之一。为了在控制台上使用 MySQLi，我们需要确保已安装 PHP CLI 和`mysql`驱动程序扩展：

```php
sudo apt-get -y install php7.0-cli php7.0-mysql

```

请注意扩展名缺少`i`后缀。安装了`mysql`驱动程序扩展后，我们可以继续并开始查询 MySQL 服务器。

# 连接

我们可以使用 MySQLi 函数或类与 MySQL 交互。为了面向对象编程，我们将在所有示例中使用类方法。使用`mysqli`类，我们可以从 PHP 建立与 MySQL 的连接，如下所示：

```php
$mysqli = new mysqli('127.0.0.1', 'root', 'mL08e!Tq', 'sakila');

```

这一行表达式将在`127.0.0.1`主机上查找 MySQL，并尝试使用`root`用户名和`mL08e!Tq`作为密码连接到其`sakila`数据库。

# 错误处理

在处理`mysqli`的错误时相对容易，因为我们可以使用简单的`try...catch`块，如下所示：

```php
<?php

mysqli_report(MYSQLI_REPORT_ALL);   try {
  $mysqli = new mysqli('127.0.0.1', 'root', 'mL08e!Tq', 'sakila'); } catch (Throwable $t) {   exit($t->getMessage()); }

```

理想情况下，我们希望只针对 MySQL 异常使用`mysqli_sql_exception`进行处理：

```php
<?php

mysqli_report(MYSQLI_REPORT_ALL);

try {
  $mysqli = new mysqli('127.0.0.1', 'root', 'mL08e!Tq', 'sakila');
} catch (mysqli_sql_exception $e) {
  exit($e->getMessage());
}

```

我们可以将以下报告级别之一传递给`mysqli_report()`函数：

+   `MYSQLI_REPORT_INDEX`: 这报告查询中是否使用了错误的索引或根本没有使用索引

+   `MYSQLI_REPORT_ERROR`: 这报告来自 MySQL 函数调用的错误

+   `MYSQLI_REPORT_STRICT`: 这报告`mysqli_sql_exception`而不是可能的警告

+   `MYSQLI_REPORT_ALL`: 这报告所有内容

+   `MYSQLI_REPORT_OFF`: 这不报告任何内容

虽然`MYSQLI_REPORT_ALL`可能看起来有些过度，但使用它可以准确定位应用程序级别不明显的 MySQL 错误，比如某列缺乏索引。

# 选择

我们可以使用`mysqli`实例的`query()`方法从 MySQL 中选择数据，如下所示：

```php
<?php   try {
  // Report on all types of errors
  mysqli_report(MYSQLI_REPORT_ALL);    // Open a new connection to the MySQL server
  $mysqli = new mysqli('127.0.0.1', 'root', 'mL08e!Tq', 'sakila');    // Perform a query on the database
  $result = $mysqli->query('SELECT * FROM customer WHERE email LIKE "MARIA.MILLER@sakilacustomer.org"');    // Return the current row of a result set as an object
  $customer = $result->fetch_object();    // Close opened database connection
  $mysqli->close();    // Output customer info
  echo $customer->first_name, ' ', $customer->last_name, PHP_EOL; } catch (mysqli_sql_exception $e) {
  // Output error and exit upon exception
  echo $e->getMessage(), PHP_EOL;
  exit; }

```

上面的例子会产生以下错误：

```php
No index used in query/prepared statement SELECT * FROM customer WHERE email = "MARIA.MILLER@sakilacustomer.org"

```

如果我们使用`MYSQLI_REPORT_STRICT`而不是`MYSQLI_REPORT_ALL`，我们就不会得到错误。然而，使用较少限制的错误报告并不是解决错误的办法。即使我们可能不负责数据库架构和维护，作为开发人员，我们有责任报告这些问题，因为它们肯定会影响我们应用程序的性能。在这种情况下，解决方案是实际上在 email 列上创建一个索引。我们可以通过以下查询轻松实现：

```php
ALTER TABLE customer ADD INDEX idx_email (email);

```

![](img/03089aff-be4e-4585-965d-fe41c12c0b33.png)

`idx_email`是我们要创建的索引的自由给定名称，而`email`是我们要创建索引的列。`idx_`前缀只是一些开发人员使用的约定，索引可以轻松地命名为`xyz`或只是`email`。

有了索引之后，如果我们现在尝试执行之前的代码，它应该输出 MARIA MILLER，如下面的屏幕截图所示：

![](img/80f364e5-0d24-403b-85d8-5184b001d2f7.png)

`query()`方法根据以下类型返回`mysqli_result`对象或`True`和`False`布尔值：

+   `SELECT`类型的查询 - `mysqli_result`对象或布尔值`False`

+   `SHOW`类型的查询 - `mysqli_result`对象或布尔值`False`

+   `DESCRIBE`类型的查询 - `mysqli_result`对象或布尔值`False`

+   `EXPLAIN`类型的查询 - `mysqli_result`对象或布尔值`False`

+   其他类型的查询 - 布尔值`True`或`False`

`mysqli_result`对象的实例有几种不同的结果获取方法：

+   `fetch_object()`: 这将结果集的当前行作为对象获取，并允许重复调用

+   `fetch_all()`: 这将以`MYSQLI_ASSOC`、`MYSQLI_NUM`或`MYSQLI_BOTH`的形式获取所有结果行

+   `fetch_array()`: 这将以`MYSQLI_ASSOC`、`MYSQLI_NUM`或`MYSQLI_BOTH`的形式获取单个结果行

+   `fetch_assoc()`: 这将以关联数组的形式获取单个结果行，并允许重复调用

+   `fetch_field()`: 这获取结果集中的下一个字段，并允许重复调用

+   `fetch_field_direct()`: 这获取单个字段的元数据

+   `fetch_fields()`: 这获取整个结果集中字段的元数据

+   `fetch_row()`: 这以枚举数组的形式获取单个结果行，并允许重复调用

# 绑定参数

更多时候，查询数据都伴随着数据绑定。从安全角度来看，数据绑定是正确的做法，因为我们不应该自己将查询字符串与变量连接起来。这会导致 SQL 注入攻击。我们可以使用相应的`mysqli`和`mysqli_stmt`实例的`prepare()`和`bind_param()`方法将数据绑定到查询中，如下所示：

```php
<?php   try {
  // Report on all types of errors
  mysqli_report(MYSQLI_REPORT_ALL);    // Open a new connection to the MySQL server
  $mysqli = new mysqli('127.0.0.1', 'root', 'mL08e!Tq', 'sakila');    $customerIdGt = 100;
  $storeId = 2;
  $email = "%ANN%";    // Prepare an SQL statement for execution
  $statement = $mysqli->prepare('SELECT * FROM customer WHERE customer_id > ? AND store_id = ? AND email LIKE ?');    // Binds variables to a prepared statement as parameters
  $statement->bind_param('iis', $customerIdGt, $storeId, $email);    // Execute a prepared query
  $statement->execute();    // Gets a result set from a prepared statement
  $result = $statement->get_result();    // Fetch object from row/entry in result set
  while ($customer = $result->fetch_object()) {
  // Output customer info
  echo $customer->first_name, ' ', $customer->last_name, PHP_EOL;
 }    // Close a prepared statement
  $statement->close();    // Close database connection
  $mysqli->close(); } catch (mysqli_sql_exception $e) {
  // Output error and exit upon exception
  echo $e->getMessage();
  exit; }

```

这应该给我们以下输出：

![](img/f135143d-8a93-4901-9830-db2de28fa206.png)

`bind_param()`方法有一个有趣的语法。它接受两个或更多参数。第一个参数——`$types`字符串——包含一个或多个字符。这些字符指定了相应绑定变量的类型：

+   `i`：这是一个整数类型的变量

+   `d`：这是一个双精度类型的变量

+   `s`：这是一个字符串类型的变量

+   `b`：这是一个 blob 类型的变量

第二个及其后的所有参数代表绑定变量。我们的示例使用`'iis'`作为`$types`参数，基本上读取`bind_param()`方法及其参数为：绑定整数类型（`$customerIdGt`）、整数类型（`$storeId`）和字符串类型（`$email`）。

# 插入

现在我们已经学会了如何准备查询并将数据绑定到它，插入新记录变得非常容易：

```php
<?php   try {
  // Report on all types of errors
  mysqli_report(MYSQLI_REPORT_ALL);    // Open a new connection to the MySQL server
 $mysqli = new mysqli('127.0.0.1', 'root', 'mL08e!Tq', 'sakila');     // Prepare some teat address data
  $address = 'The street';
  $district = 'The district';
  $cityId = 135; // Matches the Dallas city in Sakila DB
  $postalCode = '31000';
  $phone = '123456789';    // Prepare an SQL statement for execution
  $statement = $mysqli->prepare('INSERT INTO address (
 address, district, city_id, postal_code, phone ) VALUES ( ?, ?, ?, ?, ? ); ');    // Bind variables to a prepared statement as parameters
  $statement->bind_param('ssiss', $address, $district, $cityId, $postalCode, $phone);    // Execute a prepared Query
  $statement->execute();    // Close a prepared statement
  $statement->close();    // Quick & "dirty" way to fetch newly created address id
  $addressId = $mysqli->insert_id;    // Close database connection
  $mysqli->close(); } catch (mysqli_sql_exception $e) {
  // Output error and exit upon exception
  echo $e->getMessage();
  exit; }

```

这里的示例基本上遵循了之前介绍的绑定。明显的区别仅在于实际的`INSERT INTO` SQL 表达式。不用说，`mysqli`没有单独的 PHP 类或方法来处理选择、插入或任何其他操作。

# 更新

与选择和插入类似，我们也可以使用`prepare()`、`bind_param()`和`execute()`方法来处理记录更新，如下所示：

```php
<?php   try {
  // Report on all types of errors
  mysqli_report(MYSQLI_REPORT_ALL);    // Open a new connection to the MySQL server
 $mysqli = new mysqli('127.0.0.1', 'root', 'mL08e!Tq', 'sakila');     // Prepare some teat address data
  $address = 'The new street';
  $addressId = 600;    // Prepare an SQL statement for execution
  $statement = $mysqli->prepare('UPDATE address SET address = ? WHERE address_id = ?');    // Bind variables to a prepared statement as parameters
  $statement->bind_param('si', $address, $addressId);    // Execute a prepared Query
  $statement->execute();    // Close a prepared statement
  $statement->close();     // Close database connection
  $mysqli->close(); } catch (mysqli_sql_exception $e) {
  // Output error and exit upon exception
  echo $e->getMessage();
  exit; } 

```

# 删除

同样，我们可以使用`prepare()`、`bind_param()`和`execute()`方法来处理记录删除，如下所示：

```php
<?php   try {
  // Report on all types of errors
  mysqli_report(MYSQLI_REPORT_ALL);    // Open a new connection to the MySQL server
 $mysqli = new mysqli('127.0.0.1', 'root', 'mL08e!Tq', 'sakila');     // Prepare some teat address data
  $paymentId = 500;    // Prepare an SQL statement for execution
  $statement = $mysqli->prepare('DELETE FROM payment WHERE payment_id = ?');    // Bind variables to a prepared statement as parameters
  $statement->bind_param('i', $paymentId);    // Execute a prepared Query
  $statement->execute();    // Close a prepared statement
  $statement->close();    // Close database connection
  $mysqli->close(); } catch (mysqli_sql_exception $e) {
  // Output error and exit upon exception
  echo $e->getMessage();
  exit; }

```

# 事务

虽然`SELECT`、`INSERT`、`UPDATE`和`DELETE`方法允许我们逐步操纵数据，但 MySQL 的真正优势在于事务。使用`mysqli`实例的`begin_transaction()`、`commit()`、`commit()`和`rollback()`方法，我们能够控制 MySQL 的事务特性：

```php
<?php   mysqli_report(MYSQLI_REPORT_ALL); $mysqli = new mysqli('127.0.0.1', 'root', 'mL08e!Tq', 'sakila');   try {
  // Start new transaction
  $mysqli->begin_transaction(MYSQLI_TRANS_START_READ_WRITE);    // Create new address
  $result = $mysqli->query('INSERT INTO address (
 address, district, city_id, postal_code, phone ) VALUES ( "The street", "The district", 333, "31000", "123456789" ); ');    // Fetch newly created address id
  $addressId = $mysqli->insert_id;    // Create new customer
  $statement = $mysqli->prepare('INSERT INTO customer (
 store_id, first_name, last_name, email, address_id ) VALUES ( 2, "John", "Doe", "john@test.it", ? ) ');
  $statement->bind_param('i', $addressId);
  $statement->execute();    // Fetch newly created customer id
  $customerId = $mysqli->insert_id;    // Select newly created customer info
  $statement = $mysqli->prepare('SELECT * FROM customer WHERE customer_id = ?');
  $statement->bind_param('i', $customerId);
  $statement->execute();
  $result = $statement->get_result();
  $customer = $result->fetch_object();    // Commit transaction
  $mysqli->commit();    echo $customer->first_name, ' ', $customer->last_name, PHP_EOL; } catch (mysqli_sql_exception $t) {
  // We MUST be careful with non-db try block operations that throw exceptions
 // As they might cause a rollback inadvertently  $mysqli->rollback();
  echo $t->getMessage(), PHP_EOL; }   // Close database connection $mysqli->close();

```

有效的事务标志如下：

+   `MYSQLI_TRANS_START_READ_ONLY`：这与 MySQL 的`START TRANSACTION READ ONLY`查询相匹配

+   `MYSQLI_TRANS_START_READ_WRITE`：这与 MySQL 的`START TRANSACTION READ WRITE`查询相匹配

+   `MYSQLI_TRANS_START_WITH_CONSISTENT_SNAPSHOT`：这与 MySQL 的`START TRANSACTION WITH CONSISTENT SNAPSHOT`查询相匹配

查看[`dev.mysql.com/doc/refman/5.7/en/commit.html`](https://dev.mysql.com/doc/refman/5.7/en/commit.html)以获取有关 MySQL 事务语法和含义的更多信息。

# 通过 PHP 数据对象驱动扩展进行查询

**PHP 数据对象**（**PDO**）驱动扩展自 PHP 5.1.0 以来就默认包含在 PHP 中。

# 连接

使用 PDO 驱动扩展，我们可以使用`PDO`类从 PHP 连接到 MySQL 数据库，如下所示：

```php
<?php   $host = '127.0.0.1'; $dbname = 'sakila'; $username = 'root'; $password = 'mL08e!Tq';   $conn = new PDO(
  "mysql:host=$host;dbname=$dbname",
  $username,
  $password  );

```

这个简单的多行表达式将在`127.0.0.1`主机上查找 MySQL，并尝试使用`root`用户名和`mL08e!Tq`密码连接到其`sakila`数据库。

# 错误处理

在 PDO 周围处理错误可以使用特殊的`PDOException`类，如下所示：

```php
<?php   try {
  $host = '127.0.0.1';
  $dbname = 'sakila';
  $username = 'root';
  $password = 'mL08e!Tq';    $conn = new PDO(
  "mysql:host=$host;dbname=$dbname",
  $username,
  $password,
 [PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION]
 ); } catch (PDOException $e) {
  echo $e->getMessage(), PHP_EOL; }

```

有三种不同的错误模式：

+   `ERRMODE_SILENT`

+   `ERRMODE_WARNING`

+   `ERRMODE_EXCEPTION`

在这里，我们使用`ERRMODE_EXCEPTION`来利用`try...catch`块。

# 选择

通过`PDO`查询记录与通过`mysqli`查询记录有些类似。在两种情况下，我们都使用原始的 SQL 语句。区别在于 PHP 方法的便利性和它们提供的微妙差异。以下示例演示了我们如何从 MySQL 表中选择记录：

```php
<?php   try {
  $conn = new PDO(
  "mysql:host=127.0.0.1;dbname=sakila", 'root', 'mL08e!Tq',
 [PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION]
 );    $result = $conn->query('SELECT * FROM customer LIMIT 5');
  $customers = $result->fetchAll(PDO::FETCH_OBJ);    foreach ($customers as $customer) {
  echo $customer->first_name, ' ', $customer->last_name, PHP_EOL;
 } } catch (PDOException $e) {
  echo $e->getMessage(), PHP_EOL; }

```

这将产生以下输出：

![](img/ba1f9b0a-105f-4977-9655-96ca820db1b1.png)

`PDOStatement`实例和`$result`对象有几种不同的结果提取方法：

+   `fetch()`：这从结果集中提取下一行，允许重复调用，并根据提取样式返回一个值

+   `fetchAll()`：这将结果集中的所有行作为数组提取出来，并根据提取样式返回一个值

+   `fetchObject()`：这从结果集中提取下一行作为对象，并允许重复调用

+   `fetchColumn()`：这从结果集的下一行中提取单个列，并允许重复调用

以下列表显示了可用的 PDO 获取样式：

+   `PDO::FETCH_LAZY`

+   `PDO::FETCH_ASSOC`

+   `PDO::FETCH_NUM`

+   `PDO::FETCH_BOTH`

+   `PDO::FETCH_OBJ`

+   `PDO::FETCH_BOUND`

+   `PDO::FETCH_COLUMN`

+   `PDO::FETCH_CLASS`

+   `PDO::FETCH_INTO`

+   `PDO::FETCH_FUNC`

+   `PDO::FETCH_GROUP`

+   `PDO::FETCH_UNIQUE`

+   `PDO::FETCH_KEY_PAIR`

+   `PDO::FETCH_CLASSTYPE`

+   `PDO::FETCH_SERIALIZE`

+   `PDO::FETCH_PROPS_LATE`

+   `PDO::FETCH_NAMED`

虽然大多数这些获取样式都相当不言自明，我们可以查阅[`php.net/manual/en/pdo.constants.php`](http://php.net/manual/en/pdo.constants.php)以获取更多细节。

以下示例演示了更为详细的选择方法，其中包含参数绑定：

```php
<?php   try {
  $conn = new PDO(
  "mysql:host=127.0.0.1;dbname=sakila", 'root', 'mL08e!Tq',
 [PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION]
 );    $statement = $conn->prepare('SELECT * FROM customer        WHERE customer_id > :customer_id AND store_id = :store_id AND email LIKE :email');    $statement->execute([
  ':customer_id' => 100,
  ':store_id' => 2,
  ':email' => '%ANN%',
 ]);    $customers = $statement->fetchAll(PDO::FETCH_OBJ);    foreach ($customers as $customer) {
  echo $customer->first_name, ' ', $customer->last_name, PHP_EOL;
 } } catch (PDOException $e) {
  echo $e->getMessage(), PHP_EOL; }

```

这将给出以下输出：

![](img/083fe023-37f4-4e94-b8c7-a181425e22ef.png)

使用`PDO`和`mysqli`绑定的最明显区别是`PDO`允许命名参数绑定。这使得查询更加可读。

# 插入

就像选择一样，插入涉及相同一组包裹在`INSERT INTO` SQL 语句周围的 PDO 方法：

```php
<?php   try {
  $conn = new PDO(
 "mysql:host=127.0.0.1;dbname=sakila", 'root', 'mL08e!Tq',  [PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION]
 );    $statement = $conn->prepare('INSERT INTO address (
 address, district, city_id, postal_code, phone, location ) VALUES ( :address, :district, :city_id, :postal_code, :phone, POINT(:longitude, :latitude) ); ');    $statement->execute([
  ':address' => 'The street',
  ':district' => 'The district',
  ':city_id' => '537',
  ':postal_code' => '31000',
  ':phone' => '888777666333',
  ':longitude' => 45.55111,
  ':latitude' => 18.69389
  ]); } catch (PDOException $e) {
  echo $e->getMessage(), PHP_EOL; }

```

# 更新

就像选择和插入一样，更新涉及相同一组包裹在`UPDATE` SQL 语句周围的 PDO 方法：

```php
<?php   try {
  $conn = new PDO(
  "mysql:host=127.0.0.1;dbname=sakila", 'root', 'mL08e!Tq',
 [PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION]
 );    $statement = $conn->prepare('UPDATE address SET phone = :phone WHERE address_id = :address_id');    $statement->execute([
  ':phone' => '888777666555',
  ':address_id' => 600,
 ]); } catch (PDOException $e) {
  echo $e->getMessage(), PHP_EOL; }

```

# 删除

就像选择、插入和更新一样，删除涉及相同一组包裹在`DELETE FROM` SQL 语句周围的 PDO 方法：

```php
<?php   try {
  $conn = new PDO(
  "mysql:host=127.0.0.1;dbname=sakila", 'root', 'mL08e!Tq',
 [PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION]
 );  $statement = $conn->prepare('DELETE FROM payment WHERE payment_id = :payment_id');
  $statement->execute([
  ':payment_id' => 16046
  ]); } catch (PDOException $e) {
  echo $e->getMessage(), PHP_EOL; }

```

# 事务

与 MySQLi 一样，PDO 的事务与 MySQLi 的事务并没有太大不同。通过利用`PDO`实例的`beginTransaction()`、`commit()`和`rollback()`方法，我们能够控制 MySQLi 的事务特性：

```php
<?php   $conn = new PDO(
  "mysql:host=127.0.0.1;dbname=sakila", 'root', 'mL08e!Tq',
 [PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION] );   try {
  // Start new transaction
  $conn->beginTransaction();    // Create new address
  $result = $conn->query('INSERT INTO address (
 address, district, city_id, postal_code, phone, location ) VALUES ( "The street", "The district", 537, "27107", "888777666555", POINT(45.55111, 18.69389) ); ');    // Fetch newly created address id
  $addressId = $conn->lastInsertId();    // Create new customer
  $statement = $conn->prepare('INSERT INTO customer (
 store_id, first_name, last_name, email, address_id ) VALUES ( 2, "John", "Doe", "john-pdo@test.it", :address_id ) ');    $statement->execute([':address_id' => $addressId]);    // Fetch newly created customer id
  $customerId = $conn->lastInsertId();    // Select newly created customer info
  $statement = $conn->prepare('SELECT * FROM customer WHERE customer_id = :customer_id');
  $statement->execute([':customer_id' => $customerId]);
  $customer = $statement->fetchObject();    // Commit transaction
  $conn->commit();    echo $customer->first_name, ' ', $customer->last_name, PHP_EOL; } catch (PDOException $e) {
  $conn->rollback();
  echo $e->getMessage(), PHP_EOL; }

```

# 使用 MongoDB

MongoDB 是由 MongoDB Inc.开发的免费开源 NoSQL 数据库。

MongoDB 的一些关键优势可以概括如下：

+   它是一个基于文档的数据库

+   它是跨平台的

+   它既可以在单个服务器上运行，也可以在分布式架构上运行

+   它可以用于桌面和 Web 应用程序

+   它使用 JSON 对象来存储数据

+   它可以在服务器端使用 JavaScript map-reduce 进行信息处理

+   它处理大量数据

+   它聚合计算

+   它支持字段、范围查询和正则表达式搜索

+   它是本地复制

# 安装 MongoDB

假设我们正在使用全新的 Ubuntu 16.10（Yakkety Yak）安装，以下步骤概述了我们如何设置 MongoDB：

1.  我们将使用以下控制台命令安装 MongoDB：

```php
sudo apt-get update
sudo apt-get install -y mongodb

```

1.  为了进一步检查 MongoDB 是否成功安装和运行，我们可以执行以下命令：

```php
sudo systemctl status mongodb.service

```

1.  这应该给我们以下输出：

![](img/c1e47bda-507b-4206-bfcc-d595cda6935e.png)

# 设置示例数据

在 Ubuntu 终端上运行`mongo`命令可以进入 mongo 交互式 shell。从这里开始，只需简单的几个命令，我们就可以添加示例数据：

```php
use foggyline
db.products.insert({name: "iPhone 7", price: 650, weight: "138g"});
db.products.insert({name: "Samsung Galaxy S7", price: 670, weight: "152g" });
db.products.insert({name: "Motorola Moto Z Play", price: 449.99, weight: "165g" });
db.products.insert({name: "Google Pixel", price: 649.99, weight: "168g" });
db.products.insert({name: "HTC 10", price: 799, weight: "161g" });
show dbs
show collections

```

这应该给我们一个与以下截图类似的输出：

![](img/f83de627-4f06-4e31-acf9-62d4bf809f6f.png)

使用`use foggyline`和`db.products.find()`，我们现在可以列出添加到`products`集合中的所有条目：

![](img/f077bd4e-6193-4bee-a8d0-2b4324c8b276.png)

# 通过 MongoDB 驱动程序扩展查询

我们需要确保已安装 PHP CLI 和 MongoDB 驱动程序扩展：

```php
sudo apt-get -y install php-pear
sudo apt-get -y install php7.0-dev
sudo apt-get -y install libcurl4-openssl-dev pkg-config libssl-dev libsslcommon2-dev
sudo pecl install mongodb

```

成功执行这些命令后，我们可以确认`mongodb`驱动程序扩展已安装，如下截图所示：

![](img/7f71d7c9-2572-404e-8bae-5b02547f3110.png)

除了驱动程序扩展，我们还需要在项目目录中添加`mongodb/mongodb`composer 包。我们可以通过运行以下控制台命令来实现：

```php
sudo apt-get -y install composer
composer require mongodb/mongodb

```

假设我们的项目目录中有`mongo.php`文件，只需加载 MongoDB 库，就可以开始使用 Mongo 数据库：

```php
<?php   require_once __DIR__ . '/vendor/autoload.php';   // Code...

```

# 连接

使用`mongodb`驱动程序扩展和`mongodb/mongodb` PHP 库，我们可以使用`MongoDBDriverManager`类从 PHP 连接到 Mongo 数据库，如下所示：

```php
<?php   require_once __DIR__ . '/vendor/autoload.php';   $manager = new MongoDBDriverManager('mongodb://localhost:27017');

```

这个单行表达式将在`localhost`的端口`27017`下寻找 MongoDB。

# 错误处理

使用`try...catch`块处理错误非常简单，因为每当发生错误时，都会抛出`MongoDBDriverExceptionException`：

```php
<?php   require_once __DIR__ . '/vendor/autoload.php';   try {
  $manager = new MongoDBDriverManager('mongodb://localhost:27017'); } catch (MongoDBDriverExceptionException $e) {
  echo $e->getMessage(), PHP_EOL;
  exit; }

```

# 选择

使用 MongoDB 获取数据涉及与三个不同类的工作，`MongoDBDriverManager`，`MongoDBDriverQuery`和`MongoDBDriverReadPreference`：

```php
<?php   require_once __DIR__ . '/vendor/autoload.php';   try {
  $manager = new MongoDBDriverManager('mongodb://localhost:27017');    /* Select only the matching documents */
  $filter = [
  'price' => [
  '$gte' => 619.99,
 ], ];    $queryOptions = [
  /* Return only the following fields in the matching documents */
  'projection' => [
  'name' => 1,
  'price' => 1,
 ],  /* Return the documents in descending order of price */
  'sort' => [
  'price' => -1
  ]
 ];    $query = new MongoDBDriverQuery($filter, $queryOptions);    $readPreference = new MongoDBDriverReadPreference(MongoDBDriverReadPreference::RP_PRIMARY);    $products = $manager->executeQuery('foggyline.products', $query, $readPreference);    foreach ($products as $product) {
  echo $product->name, ': ', $product->price, PHP_EOL;
 } } catch (MongoDBDriverExceptionException $e) {
  echo $e->getMessage(), PHP_EOL;
  exit; }

```

这会产生以下输出：

![](img/8dc9e943-de05-44f8-8ddf-0a23749090bc.png)

我们可以传递给`$filter`的查询运算符列表非常广泛，但以下比较运算符可能是最有趣的：

+   `$eq`: 这些匹配所有等于指定值的值

+   `$gt`: 这些匹配所有大于指定值的值

+   `$gte`: 这些匹配所有大于或等于指定值的值

+   `$lt`: 这些匹配所有小于指定值的值

+   `$lte`: 这些匹配所有小于或等于指定值的值

+   `$ne`: 这些匹配所有不等于指定值的值

+   `$in`: 这些匹配数组中指定的所有值

+   `$nin`: 这些匹配数组中指定的无值

查看[ttps://docs.mongodb.com/manual/reference/operator/query/](https://docs.mongodb.com/manual/reference/operator/query/)，了解 MongoDB 查询和投影运算符的完整列表。

我们可以传递给`$queryOptions`的查询选项列表同样令人印象深刻，但以下选项可能是最重要的选项：

+   `collation`: 这些允许指定字符串比较的语言特定规则

+   `limit`: 这些允许指定要返回的文档的最大数量

+   `maxTimeMS`: 这些以毫秒为单位设置处理操作的时间限制

+   `projection`: 这些允许指定返回文档中包含哪些字段

+   `sort`: 这些允许指定结果的排序顺序

查看[`php.net/manual/en/mongodb-driver-query.construct.php`](http://php.net/manual/en/mongodb-driver-query.construct.php)，了解`MongoDBDriverQuery`查询选项的完整列表。

# 插入

使用 MongoDB 编写新数据涉及与三个不同类的工作，`MongoDBDriverManager`，`MongoDBDriverBulkWrite`和`MongoDBDriverWriteConcern`：

```php
<?php   require_once __DIR__ . '/vendor/autoload.php';   try {
  $manager = new MongoDBDriverManager('mongodb://localhost:27017');    $bulkWrite = new MongoDBDriverBulkWrite;    $bulkWrite->insert([
  'name' => 'iPhone 7 Black White',
  'price' => 650,
  'weight' => '138g'
  ]);    $bulkWrite->insert([
  'name' => 'Samsung Galaxy S7 White',
  'price' => 670,
  'weight' => '152g'
  ]);    $writeConcern = new MongoDBDriverWriteConcern(MongoDBDriverWriteConcern::MAJORITY, 1000);    $result = $manager->executeBulkWrite('foggyline.products', $bulkWrite, $writeConcern);    if ($result->getInsertedCount()) {
  echo 'Record(s) saved successfully.', PHP_EOL;
 } else {
  echo 'Error occurred.', PHP_EOL;
 } } catch (MongoDBDriverExceptionException $e) {
  echo $e->getMessage(), PHP_EOL;
  exit; } 

```

`BulkWrite`的实例可以通过`insert()`方法存储一个或多个插入语句。然后我们简单地将`$bulkWrite`和`$writeConcern`传递给`$manager`实例上的`executeBulkWrite()`。执行后，我们可以通过`mongo` shell 观察到新添加的记录：

![](img/dca64ccf-a77c-4baf-8350-0a99565ef598.png)

# 更新

更新现有数据几乎与编写新数据的过程相同。明显的区别在于在`MongoDBDriverBulkWrite`实例上使用`update()`方法：

```php
<?php   require_once __DIR__ . '/vendor/autoload.php';   try {
  $manager = new MongoDBDriverManager('mongodb://localhost:27017');    $bulkWrite = new MongoDBDriverBulkWrite;    $bulkWrite->update(
 ['name' => 'iPhone 7 Black White'],
 ['$set' => [
  'name' => 'iPhone 7 Black Black',
  'price' => 649.99
  ]],
 ['multi' => true, 'upsert' => false]
 );    $bulkWrite->update(
 ['name' => 'Samsung Galaxy S7 White'],
 ['$set' => [
  'name' => 'Samsung Galaxy S7 Black',
  'price' => 669.99
  ]],
 ['multi' => true, 'upsert' => false]
 );    $writeConcern = new MongoDBDriverWriteConcern(MongoDBDriverWriteConcern::MAJORITY, 1000);    $result = $manager->executeBulkWrite('foggyline.products', $bulkWrite, $writeConcern);    if ($result->getModifiedCount()) {
  echo 'Record(s) saved updated.', PHP_EOL;
 } else {
  echo 'Error occurred.', PHP_EOL;
 } } catch (MongoDBDriverExceptionException $e) {
  echo $e->getMessage(), PHP_EOL;
  exit; } 

```

`update()`方法接受三个不同的参数：过滤器，新对象和更新选项。在更新选项下传递的`multi`选项告诉是否将更新所有文档的匹配条件。在更新选项下传递的`upsert`选项控制如果找不到现有记录，则创建新记录。通过`mongo` shell 可以观察到结果的更改：

![](img/bef46af0-ec27-46f9-9912-9215f17ee3c7.png)

# 删除

删除类似于写入和更新的方式进行，它使用`MongoDBDriverBulkWrite`对象的实例。这次，我们使用`delete()`方法的实例，它接受过滤器和删除选项：

```php
<?php   require_once __DIR__ . '/vendor/autoload.php';   try {
  $manager = new MongoDBDriverManager('mongodb://localhost:27017');    $bulkWrite = new MongoDBDriverBulkWrite;    $bulkWrite->delete(
  // filter
  [
  'name' => [
  '$regex' => '^iPhone'
  ]
 ],  // Delete options
  ['limit' => false]
 );    $writeConcern = new MongoDBDriverWriteConcern(MongoDBDriverWriteConcern::MAJORITY, 1000);    $result = $manager->executeBulkWrite('foggyline.products', $bulkWrite, $writeConcern);    if ($result->getDeletedCount()) {
  echo 'Record(s) deleted.', PHP_EOL;
 } else {
  echo 'Error occurred.', PHP_EOL;
 } } catch (MongoDBDriverExceptionException $e) {
  echo $e->getMessage(), PHP_EOL;
  exit; } 

```

使用`false`值作为`limit`选项，我们实际上要求删除所有匹配的文档。使用`mongo` shell，我们可以观察到以下截图中显示的更改：

![](img/75596302-1926-4aa5-a7ad-a598e97a3dde.png)

# 交易

MongoDB 在某种意义上不具有与 MySQL 相同的完整**ACID**（原子性、一致性、隔离性、持久性）支持。它仅在文档级别支持 ACID 事务。不支持多文档事务。ACID 合规性的缺失确实限制了它在依赖于此功能的平台上的使用。这并不是说 MongoDB 不能与这些平台一起使用。让我们考虑一个流行的 Magento 电子商务平台。没有什么可以阻止 Magento 将 MongoDB 添加到混合中。虽然 MySQL 功能可以保证与销售相关功能的 ACID 合规性，但 MongoDB 可以在其中使用以覆盖目录功能的部分。这种共生关系可以轻松地将两种数据库功能的最佳部分带到我们的平台上。

# 使用 Redis

Redis 是一个开源的内存数据结构存储，由 Redis Labs 赞助开发。其名称源自**REmote DIctionary Server**。它目前是最受欢迎的键值数据库之一。

Redis 的一些关键优势可以概括如下：

+   内存数据结构存储

+   键值数据存储

+   具有有限生存时间的键

+   发布/订阅消息

+   它可以用于缓存数据存储

+   事务

+   主从复制

# 安装 Redis

假设我们正在使用全新的 Ubuntu 16.10（Yakkety Yak）安装，以下步骤概述了我们如何设置 Redis 服务器：

1.  我们可以使用以下控制台命令安装 Redis 服务器：

```php
sudo apt-get update
sudo apt-get -y install build-essential tcl
wget http://download.redis.io/redis-stable.tar.gz
tar xzf redis-stable.tar.gz
cd redis-stable
make
make test
sudo make install
./src/redis-server

```

1.  这应该给我们以下输出：

![](img/e171ce49-e5c7-4be9-9cef-c9d54b12f95e.png)

# 设置示例数据

在 Ubuntu 终端上运行`redis-cli`命令可以进入 Redis 交互式 shell。从这里开始，通过简单的几个命令，我们可以添加以下示例数据：

```php
SET Key1 10
SET Key2 20
SET Key3 30
SET Key4 40
SET Key5 50

```

这应该给我们以下输出：

![](img/d182d03c-9f1a-4e64-a1c7-b1187fa08bbf.png)

使用`redis-cli` shell 中的`KEYS *`命令，我们现在可以列出 Redis 添加的所有条目：

![](img/2264aaab-0740-45d0-b8d6-96a37f4e7146.png)

# 通过 phpredis 驱动程序扩展进行查询

在开始查询之前，我们需要确保已安装 PHP CLI 和`phpredis`驱动程序扩展：

```php
sudo apt-get -y install php7.0-dev
sudo apt-get -y install unzip
wget https://github.com/phpredis/phpredis/archive/php7.zip -O phpredis.zip
unzip phpredis.zip 
cd phpredis-php7/
phpize
./configure
make
sudo make install
echo extension=redis.so >> /etc/php/7.0/cli/php.ini

```

执行这些命令后，我们可以确认`phpredis`驱动程序扩展已安装如下：

![](img/6bcd8954-b17a-4e94-94cc-e53c94316c88.png)

# 连接

使用`phpredis`驱动程序扩展，我们可以使用`Redis`类从 PHP 连接到 Redis，如下所示：

```php
<?php   $client = new Redis();   $client->connect('localhost', 6379);

```

这个单行表达式将在本地主机的端口`6379`下查找 Redis。

# 错误处理

`phpredis`驱动程序扩展对使用`Redis`类时发生的每个错误都会抛出`RedisException`。这使得通过简单的`try...catch`块轻松处理错误：

```php
<?php   try {
  $client = new Redis();
  $client->connect('localhost', 6379);
  // Code... } catch (RedisException $e) {
  echo $e->getMessage(), PHP_EOL; } 

```

# 选择

鉴于 Redis 是一个键值存储，选择键就像使用`Redis`实例的单个`get()`方法一样容易：

```php
<?php   try {
  $client = new Redis();
  $client->connect('localhost', 6379);
  echo $client->get('Key3'), PHP_EOL;
  echo $client->get('Key5'), PHP_EOL; } catch (RedisException $e) {
  echo $e->getMessage(), PHP_EOL; } 

```

这应该给我们以下输出：

![](img/61576137-5dfa-4782-8aa2-c02a063425cd.png)

`Redis`客户端类还提供了`mget()`方法，可以一次获取多个键值：

```php
<?php   try {
  $client = new Redis();
  $client->connect('localhost', 6379);    $values = $client->mget(['Key1', 'Key2', 'Key4']);
  print_r($values); } catch (RedisException $e) {
  echo $e->getMessage(), PHP_EOL; }

```

这应该给我们以下输出：

![](img/82bc6db8-3fc6-4983-ac5b-7045e0e301c4.png)

# 插入

Redis 键值机制背后的简单性使得`set()`方法简单直接，通过它我们可以插入新条目，如下例所示：

```php
<?php   try {
  $client = new Redis();
  $client->connect('localhost', 6379);    $client->set('user', [
  'name' => 'John',
  'age' => 34,
  'salary' => 4200.00
  ]);    // $client->get('user');
 // returns string containing "Array" chars    $client->set('customer', json_encode([
  'name' => 'Marc',
  'age' => 43,
  'salary' => 3600.00
  ]));    // $client->get('customer');
 // returns json looking string, which we can simply json_decode() } catch (RedisException $e) {
  echo $e->getMessage(), PHP_EOL; }

```

这应该给我们以下输出：

![](img/50b0a300-61e0-4d2f-b323-84dc6a9ca282.png)

当使用非字符串结构的 set 方法时，我们需要小心。`user`键值导致存储在 Redis 中的是数组字符串，而不是实际的数组结构。通过在传递给`set()`方法之前使用`json_encode()`将数组结构转换为 JSON，可以轻松解决这个问题。

`set()`方法的一个很大的好处是它支持以秒为单位的超时，因此我们可以轻松地编写以下表达式：

```php
$client->set('test', 'test2', 3600);

```

虽然调用`setex()`方法是我们想要为键添加超时的首选方式：

```php
$client->setex('key', 3600, 'value');

```

在使用 Redis 作为缓存数据库时，超时是一个很好的功能。它们基本上为我们自动设置了缓存的生命周期。

# 更新

通过 Redis 客户端更新值与插入值相同。我们使用相同的`set()`方法，使用相同的键。如果存在先前的值，新值将简单地覆盖它：

```php
<?php   try {
  $client = new Redis();
  $client->connect('localhost', 6379);    $client->set('test', 'test1');
  $client->set('test', 'test2');    // $client->get('test');
 // returns string containing "test2" chars } catch (RedisException $e) {
  echo $e->getMessage(), PHP_EOL; }

```

# 删除

从 Redis 中删除记录就像调用 Redis 客户端的`del()`方法并传递要删除的键一样简单：

```php
<?php   try {
  $client = new Redis();
  $client->connect('localhost', 6379);
  $client->del('user'); } catch (RedisException $e) {
  echo $e->getMessage(), PHP_EOL; }

```

# 事务

与 MongoDB 类似，Redis 在某种意义上也没有像 MySQL 那样的 ACID 支持，这其实没关系，因为 Redis 只是一个键/值存储，而不是关系数据库。然而，Redis 提供了一定程度的原子性。使用`MULTI`、`EXEC`、`DISCARD`和`WATCH`，我们能够在单个步骤中执行一组命令，Redis 在此期间提供以下两项保证：

+   另一个客户端请求永远不会在我们的组命令执行过程中被服务

+   所有命令要么全部执行，要么全部不执行

让我们看一下以下示例：

```php
<?php   try {
  $client = new Redis();
  $client->connect('localhost', 6379);    $client->multi();    $result1 = $client->set('tKey1', 'Test#1'); // Valid command
  $result2 = $client->zadd('tKey2', null); // Invalid command    if ($result1 == false || $result2 == false) {
  $client->discard();
  echo 'Transaction aborted.', PHP_EOL;
 } else {
  $client->exec();
  echo 'Transaction commited.', PHP_EOL;
 } } catch (RedisException $e) {
  echo $e->getMessage(), PHP_EOL; }

```

`$result2`的值为`false`，触发了`$client->discard();`。虽然`result1`是一个有效的表达式，但它是在`$client->multi();`调用之后出现的，这意味着它的命令实际上并没有被处理；因此，我们看不到存储在 Redis 中的`Test#1`的值。虽然没有经典的回滚机制，就像我们在 MySQL 中看到的那样，但这为一个良好的事务模型。

# 总结

在本章中，我们涉及了查询三种非常不同的数据库系统的基础知识。

MySQL 数据库已经存在很长时间，很可能是大多数 PHP 应用程序的第一个数据库。其 ACID 兼容性使其在处理财务或其他敏感数据的应用程序中不可替代，其中原子性、一致性、隔离性和耐久性是关键因素。

另一方面，Mongo 通过无模式的方法处理数据存储。这使开发人员更容易加快应用程序的开发速度，尽管文档之间缺乏 ACID 兼容性限制了它在某些类型的应用程序中的使用。

最后，Redis 数据存储作为我们应用程序的一个很好的缓存，甚至是会话存储解决方案。

接下来，我们将更仔细地看一下依赖注入，它是什么，以及在模块化应用程序中扮演什么角色。
