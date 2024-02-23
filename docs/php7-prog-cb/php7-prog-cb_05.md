# 第五章：与数据库交互

在本章中，我们将涵盖以下主题：

+   使用 PDO 连接到数据库

+   构建面向对象的 SQL 查询生成器

+   处理分页

+   定义实体以匹配数据库表

+   将实体类与 RDBMS 查询绑定

+   将辅助查找嵌入到查询结果中

+   实现 jQuery DataTables PHP 查找

# 介绍

在本章中，我们将介绍一系列利用**PHP 数据对象**（**PDO**）扩展的数据库连接配方。将解决常见的编程问题，如**结构化查询语言**（**SQL**）生成，分页和将对象与数据库表绑定。最后，我们将呈现处理嵌入式匿名函数形式的辅助查找的代码，并使用 jQuery DataTables 进行 AJAX 请求。

# 使用 PDO 连接到数据库

**PDO**是一个高性能且积极维护的数据库扩展，具有与特定供应商扩展不同的独特优势。它具有一个通用的**应用程序编程接口**（**API**），与几乎十几种不同的**关系数据库管理系统**（**RDBMS**）兼容。学习如何使用此扩展将节省您大量时间，因为您无需尝试掌握等效的各个特定供应商数据库扩展的命令子集。

PDO 分为四个主要类，如下表所示：

| 类 | 功能 |
| --- | --- |
| `PDO` | 维护与数据库的实际连接，并处理低级功能，如事务支持 |
| `PDOStatement` | 处理结果 |
| `PDOException` | 特定于数据库的异常 |
| `PDODriver` | 与实际特定供应商数据库通信 |

## 如何做...

1.  通过创建`PDO`实例建立数据库连接。

1.  您需要构建一个**数据源名称**（**DSN**）。DSN 中包含的信息根据使用的数据库驱动程序而变化。例如，这是一个用于连接到**MySQL**数据库的 DSN：

```php
$params = [
  'host' => 'localhost',
  'user' => 'test',
  'pwd'  => 'password',
  'db'   => 'php7cookbook'
];

try {
  $dsn  = sprintf(**'mysql:host=%s;dbname=%s',**
 **$params['host'], $params['db']);**
  $pdo  = new PDO($dsn, $params['user'], $params['pwd']);
} catch (PDOException $e) {
  echo $e->getMessage();
} catch (Throwable $e) {
  echo $e->getMessage();
}
```

1.  另一方面，**SQlite**，一个更简单的扩展，只需要以下命令：

```php
$params = [
  'db'   => __DIR__ . '/../data/db/php7cookbook.db.sqlite'
];
$dsn  = sprintf('sqlite:' . $params['db']);
```

1.  另一方面，**PostgreSQL**直接在 DSN 中包括用户名和密码：

```php
$params = [
  'host' => 'localhost',
  'user' => 'test',
  'pwd'  => 'password',
  'db'   => 'php7cookbook'
];
$dsn  = sprintf(**'pgsql:host=%s;dbname=%s;user=%s;password=%s',** 
               $params['host'], 
               $params['db'],
               $params['user'],
               $params['pwd']);
```

1.  DSN 还可以包括特定于服务器的指令，例如`unix_socket`，如下例所示：

```php
$params = [
  'host' => 'localhost',
  'user' => 'test',
  'pwd'  => 'password',
  'db'   => 'php7cookbook',
  'sock' => '/var/run/mysqld/mysqld.sock'
];

try {
  $dsn  = sprintf('mysql:host=%s;dbname=%s;**unix_socket=%s',** 
                  $params['host'], $params['db'], $params['sock']);
  $opts = [PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION];
  $pdo  = new PDO($dsn, $params['user'], $params['pwd'], $opts);
} catch (PDOException $e) {
  echo $e->getMessage();
} catch (Throwable $e) {
  echo $e->getMessage();
}
```

### 注意

**最佳实践**

将创建 PDO 实例的语句包装在`try {} catch {}`块中。在发生故障时，捕获`PDOException`以获取特定于数据库的信息。捕获`Throwable`以处理错误或任何其他异常。将 PDO 错误模式设置为`PDO::ERRMODE_EXCEPTION`以获得最佳结果。有关错误模式的更多详细信息，请参见第 8 步。

在 PHP 5 中，如果无法构造 PDO 对象（例如，使用无效参数），则实例将被赋予`NULL`值。在 PHP 7 中，会抛出一个`Exception`。如果将 PDO 对象的构造包装在`try {} catch {}`块中，并且将`PDO::ATTR_ERRMODE`设置为`PDO::ERRMODE_EXCEPTION`，则可以捕获并记录此类错误，而无需测试`NULL`。

1.  使用`PDO::query()`发送 SQL 命令。返回一个`PDOStatement`实例，您可以针对其获取结果。在此示例中，我们正在查找按 ID 排序的前 20 个客户：

```php
$stmt = $pdo->query(
'SELECT * FROM customer ORDER BY id LIMIT 20');
```

### 注意

PDO 还提供了一个方便的方法`PDO::exec()`，它不返回结果迭代，只返回受影响的行数。此方法最适用于诸如`ALTER TABLE`，`DROP TABLE`等管理操作。

1.  迭代`PDOStatement`实例以处理结果。将**获取模式**设置为`PDO::FETCH_NUM`或`PDO::FETCH_ASSOC`，以返回以数字或关联数组形式的结果。在此示例中，我们使用`while()`循环处理结果。当获取到最后一个结果时，结果为布尔值`FALSE`，结束循环：

```php
while ($row = $stmt->fetch(PDO::FETCH_ASSOC)) {
  printf('%4d | %20s | %5s' . PHP_EOL, $row['id'], 
  $row['name'], $row['level']);
}
```

### 注意

PDO 获取操作涉及定义迭代方向（即向前或向后）的**游标**。`PDOStatement::fetch()`的第二个参数可以是`PDO::FETCH_ORI_*`常量中的任何一个。游标方向包括 prior、first、last、absolute 和 relative。默认游标方向是`PDO::FETCH_ORI_NEXT`。

1.  将获取模式设置为`PDO::FETCH_OBJ`以将结果作为`stdClass`实例返回。在这里，您会注意到`while()`循环利用了获取模式`PDO::FETCH_OBJ`。请注意，`printf()`语句引用了对象属性，与前面的示例相反，前者引用了数组元素。

```php
while ($row = $stmt->fetch(PDO::FETCH_OBJ)) {
  printf('%4d | %20s | %5s' . PHP_EOL, 
 **$row->id, $row->name, $row->level);**
}
```

1.  如果要在处理查询时创建特定类的实例，请将获取模式设置为`PDO::FETCH_CLASS`。您还必须有类定义可用，并且`PDO::query()`应该设置类名。如下面的代码片段中所示，我们定义了一个名为`Customer`的类，具有公共属性`$id`、`$name`和`$level`。属性需要是`public`，以使获取注入正常工作：

```php
class Customer
{
  public $id;
  public $name;
  public $level;
}

$stmt = $pdo->query($sql, PDO::FETCH_CLASS, 'Customer');
```

1.  在获取对象时，与步骤 5 中显示的技术相比，更简单的替代方法是使用`PDOStatement::fetchObject()`：

```php
while ($row = $stmt->**fetchObject('Customer')**) {
  printf('%4d | %20s | %5s' . PHP_EOL, 
  $row->id, $row->name, $row->level);
}
```

1.  您还可以使用`PDO::FETCH_INTO`，它本质上与`PDO::FETCH_CLASS`相同，但您需要一个活动对象实例，而不是一个类引用。通过循环的每次迭代，都会使用当前信息集重新填充相同的对象实例。此示例假定与步骤 5 中相同的类`Customer`，以及与步骤 1 中定义的相同的数据库参数和 PDO 连接：

```php
$cust = new Customer();
while ($stmt->fetch(**PDO::FETCH_INTO**)) {
  printf('%4d | %20s | %5s' . PHP_EOL, 
 **$cust**->id, **$cust**->name, **$cust**->level);
}
```

1.  如果您没有指定错误模式，默认的 PDO 错误模式是`PDO::ERRMODE_SILENT`。您可以使用`PDO::ATTR_ERRMODE`键设置错误模式，以及`PDO::ERRMODE_WARNING`或`PDO::ERRMODE_EXCEPTION`值。错误模式可以作为关联数组的第四个参数指定给 PDO 构造函数。或者，您可以在现有实例上使用`PDO::setAttribute()`。

1.  假设您有以下 DSN 和 SQL（在您开始认为这是一种新形式的 SQL 之前，请放心：这个 SQL 语句不起作用！）：

```php
$params = [
  'host' => 'localhost',
  'user' => 'test',
  'pwd'  => 'password',
  'db'   => 'php7cookbook'
];
$dsn  = sprintf('mysql:host=%s;dbname=%s', $params['host'], $params['db']);
$sql  = 'THIS SQL STATEMENT WILL NOT WORK';
```

1.  然后，如果您使用默认错误模式制定 PDO 连接，出现问题的唯一线索是，`PDO::query()`将返回一个布尔值`FALSE`，而不是生成`PDOStatement`实例：

```php
$pdo1  = new PDO($dsn, $params['user'], $params['pwd']);
$stmt = $pdo1->query($sql);
$row = ($stmt) ? $stmt->fetch(PDO::FETCH_ASSOC) : 'No Good';
```

1.  下一个示例显示了使用构造函数方法将错误模式设置为`WARNING`：

```php
$pdo2 = new PDO(
  $dsn, 
  $params['user'], 
  $params['pwd'], 
  [PDO::ATTR_ERRMODE => PDO::ERRMODE_WARNING]);
```

1.  如果您需要完全分离准备和执行阶段，请使用`PDO::prepare()`和`PDOStatement::execute()`。然后将语句发送到数据库服务器进行预编译。然后可以根据需要执行语句，很可能是在循环中。

1.  `PDO::prepare()`的第一个参数可以是带有占位符的 SQL 语句，而不是实际值。然后可以向`PDOStatement::execute()`提供值数组。PDO 自动提供数据库引用，有助于防止**SQL 注入**。

### 注意

**最佳实践**

任何应用程序中，如果外部输入（即来自表单提交）与 SQL 语句结合在一起，都会受到 SQL 注入攻击的影响。所有外部输入必须首先经过适当的过滤、验证和其他清理。不要直接将外部输入放入 SQL 语句中。而是使用占位符，并在执行阶段提供实际（经过清理的）值。

1.  要以相反的顺序迭代结果，可以更改**可滚动游标**的方向。或者，更简单地，将`ORDER BY`从`ASC`更改为`DESC`。以下代码行设置了一个请求可滚动游标的`PDOStatement`对象：

```php
$dsn  = sprintf('pgsql:charset=UTF8;host=%s;dbname=%s', $params['host'], $params['db']);
$opts = [PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION]; 
$pdo  = new PDO($dsn, $params['user'], $params['pwd'], $opts);
$sql  = 'SELECT * FROM customer '
    . 'WHERE balance > :min AND balance < :max '
    . 'ORDER BY id LIMIT 20';
$stmt = $pdo->prepare($sql, **[PDO::ATTR_CURSOR  => PDO::CURSOR_SCROLL]**);
```

1.  在执行获取操作期间，您还需要指定游标指令。此示例获取结果集中的最后一行，然后向后滚动：

```php
$stmt->execute(['min' => $min, 'max' => $max]);
$row = $stmt->fetch(PDO::FETCH_ASSOC, **PDO::FETCH_ORI_LAST**);
do {
  printf('%4d | %20s | %5s | %8.2f' . PHP_EOL, 
       $row['id'], 
       $row['name'], 
       $row['level'], 
       $row['balance']);
} while ($row = $stmt->fetch(PDO::FETCH_ASSOC, **PDO::FETCH_ORI_PRIOR**));
```

1.  MySQL 和 SQLite 都不支持可滚动的游标！要实现相同的结果，请尝试对上述代码进行以下修改：

```php
$dsn  = sprintf('mysql:charset=UTF8;host=%s;dbname=%s', $params['host'], $params['db']);
$opts = [PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION]; 
$pdo  = new PDO($dsn, $params['user'], $params['pwd'], $opts);
$sql  = 'SELECT * FROM customer '
    . 'WHERE balance > :min AND balance < :max '
    . 'ORDER BY id **DESC** 
       . 'LIMIT 20';
$stmt = $pdo->prepare($sql);
while ($row = $stmt->fetch(PDO::FETCH_ASSOC));
printf('%4d | %20s | %5s | %8.2f' . PHP_EOL, 
       $row['id'], 
       $row['name'], 
       $row['level'], 
       $row['balance']);
} 
```

1.  PDO 提供了对事务的支持。借用第 9 步的代码，我们可以将`INSERT`系列命令包装到一个事务块中：

```php
try {
    $pdo->beginTransaction();
    $sql  = "INSERT INTO customer ('" 
    . implode("','", $fields) . "') VALUES (**?,?,?,?,?,?**)";
    $stmt = $pdo->prepare($sql);
    foreach ($data as $row) $stmt->execute($row);
    $pdo->commit();
} catch (PDOException $e) {
    error_log($e->getMessage());
    $pdo->rollBack();
}
```

1.  最后，为了保持一切模块化和可重用，我们可以将 PDO 连接封装到一个单独的类`Application\Database\Connection`中。在这里，我们通过构造函数建立连接。另外，还有一个静态的`factory()`方法，让我们生成一系列 PDO 实例：

```php
namespace Application\Database;
use Exception;
use PDO;
class Connection
{
    const ERROR_UNABLE = 'ERROR: no database connection';
    public $pdo;
    public function __construct(array $config)
    {
        if (!isset($config['driver'])) {
            $message = __METHOD__ . ' : ' 
            . self::ERROR_UNABLE . PHP_EOL;
            throw new Exception($message);
        }
        $dsn = $this->makeDsn($config);        
        try {
            $this->pdo = new PDO(
                $dsn, 
                $config['user'], 
                $config['password'], 
                [PDO::ATTR_ERRMODE => $config['errmode']]);
            return TRUE;
        } catch (PDOException $e) {
            error_log($e->getMessage());
            return FALSE;
        }
    }

    public static function factory(
      $driver, $dbname, $host, $user, 
      $pwd, array $options = array())
    {
        $dsn = $this->makeDsn($config);

        try {
            return new PDO($dsn, $user, $pwd, $options);
        } catch (PDOException $e) {
            error_log($e->getMessage);
        }
    }
```

1.  这个`Connection`类的一个重要组成部分是一个通用方法，用于构造 DSN。我们需要的一切就是将`PDODriver`作为前缀，后面跟着“`:`”。之后，我们只需从配置数组中追加键值对。每个键值对之间用分号分隔。我们还需要使用`substr()`来去掉末尾的分号，为此目的使用了负限制：

```php
  public function makeDsn($config)
  {
    $dsn = $config['driver'] . ':';
    unset($config['driver']);
    foreach ($config as $key => $value) {
      $dsn .= $key . '=' . $value . ';';
    }
    return substr($dsn, 0, -1);
  }
}
```

## 它是如何工作...

首先，您可以将第 1 步中的初始连接代码复制到一个名为`chap_05_pdo_connect_mysql.php`的文件中。为了说明的目的，我们假设您已经创建了一个名为`php7cookbook`的 MySQL 数据库，用户名为 cook，密码为 book。接下来，我们使用`PDO::query()`方法向数据库发送一个简单的 SQL 语句。最后，我们使用生成的语句对象以关联数组的形式获取结果。不要忘记将您的代码放在`try {} catch {}`块中：

```php
<?php
$params = [
  'host' => 'localhost',
  'user' => 'test',
  'pwd'  => 'password',
  'db'   => 'php7cookbook'
];
try {
  $dsn  = sprintf('mysql:charset=UTF8;host=%s;dbname=%s',
    $params['host'], $params['db']);
  $pdo  = new PDO($dsn, $params['user'], $params['pwd']);
  $stmt = $pdo->query(
    'SELECT * FROM customer ORDER BY id LIMIT 20');
  printf('%4s | %20s | %5s | %7s' . PHP_EOL, 
    'ID', 'NAME', 'LEVEL', 'BALANCE');
  printf('%4s | %20s | %5s | %7s' . PHP_EOL, 
    '----', str_repeat('-', 20), '-----', '-------');
  while ($row = $stmt->fetch(PDO::FETCH_ASSOC)) {
    printf('%4d | %20s | %5s | %7.2f' . PHP_EOL, 
    $row['id'], $row['name'], $row['level'], $row['balance']);
  }
} catch (PDOException $e) {
  error_log($e->getMessage());
} catch (Throwable $e) {
  error_log($e->getMessage());
}
```

以下是生成的输出：

![它是如何工作的...](img/B05314_05_01.jpg)

将选项添加到 PDO 构造函数中，将错误模式设置为`EXCEPTION`。现在修改 SQL 语句并观察生成的错误消息：

```php
$opts = [PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION];
$pdo  = new PDO($dsn, $params['user'], $params['pwd'], $opts);
$stmt = $pdo->query('THIS SQL STATEMENT WILL NOT WORK');
```

您会看到类似这样的东西：

![它是如何工作的...](img/B05314_05_02.jpg)

占位符可以是命名的或位置的。**命名占位符**在准备的 SQL 语句中以冒号（`:`）开头，并且在提供给`execute()`的关联数组中作为键引用。**位置占位符**在准备的 SQL 语句中表示为问号（`?`）。

在以下示例中，使用命名占位符来表示`WHERE`子句中的值：

```php
try {
  $dsn  = sprintf('mysql:host=%s;dbname=%s', 
                  $params['host'], $params['db']);
  $pdo  = new PDO($dsn, 
                  $params['user'], 
                  $params['pwd'], 
                  [PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION]);
  $sql  = 'SELECT * FROM customer '
      . 'WHERE balance < **:val** AND level = **:level** '
      . 'ORDER BY id LIMIT 20'; echo $sql . PHP_EOL;
  $stmt = $pdo->prepare($sql);
  $stmt->execute(['**val**' => 100, '**level**' => 'BEG']);
  while ($row = $stmt->fetch(PDO::FETCH_ASSOC)) {
    printf('%4d | %20s | %5s | %5.2f' . PHP_EOL, 
      	$row['id'], $row['name'], $row['level'], $row['balance']);
  }
} catch (PDOException $e) {
  echo $e->getMessage();
} catch (Throwable $e) {
  echo $e->getMessage();
}
```

这个例子展示了在`INSERT`操作中使用位置占位符。请注意，要插入的作为第四个客户的数据包括潜在的 SQL 注入攻击。您还会注意到，需要对正在使用的数据库的 SQL 语法有一定的了解。在这种情况下，MySQL 列名使用反引号（`'`）引用：

```php
$fields = ['name', 'balance', 'email', 
           'password', 'status', 'level'];
$data = [
  ['Saleen',0,'saleen@test.com', 'password',0,'BEG'],
  ['Lada',55.55,'lada@test.com',   'password',0,'INT'],
  ['Tonsoi',999.99,'tongsoi@test.com','password',1,'ADV'],
  ['SQL Injection',0.00,'bad','bad',1,
   'BEG\';DELETE FROM customer;--'],
];

try {
  $dsn  = sprintf('mysql:host=%s;dbname=%s', 
    $params['host'], $params['db']);
  $pdo  = new PDO($dsn, 
                  $params['user'], 
                  $params['pwd'], 
                  [PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION]);
  $sql  = "INSERT INTO customer ('" 
   . implode("','", $fields) 
   . "') VALUES (**?,?,?,?,?,?**)";
  $stmt = $pdo->prepare($sql);
  foreach ($data as $row) $stmt->execute($row);
} catch (PDOException $e) {
  echo $e->getMessage();
} catch (Throwable $e) {
  echo $e->getMessage();
}
```

为了测试使用带有命名参数的准备语句，修改 SQL 语句以添加一个`WHERE`子句，检查余额小于某个特定金额的客户，以及级别等于`BEG`、`INT`或`ADV`（即初级、中级或高级）。不要使用`PDO::query()`，而是使用`PDO::prepare()`。在获取结果之前，您必须执行`PDOStatement::execute()`，提供余额和级别的值：

```php
$sql  = 'SELECT * FROM customer '
     . 'WHERE balance < :val AND level = :level '
     . 'ORDER BY id LIMIT 20';
$stmt = $pdo->prepare($sql);
$stmt->execute(['val' => 100, 'level' => 'BEG']);
```

以下是生成的输出：

![它是如何工作的...](img/B05314_05_03.jpg)

在调用`PDOStatement::execute()`时，您可以选择绑定参数。这允许您将变量分配给占位符。在执行时，将使用变量的当前值。

在这个例子中，我们将变量`$min`，`$max`和`$level`绑定到准备好的语句中：

```php
$min   = 0;
$max   = 0;
$level = '';

try {
  $dsn  = sprintf('mysql:host=%s;dbname=%s', $params['host'], $params['db']);
  $opts = [PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION];
  $pdo  = new PDO($dsn, $params['user'], $params['pwd'], $opts);
  $sql  = 'SELECT * FROM customer '
      . 'WHERE balance > :min '
      . 'AND balance < :max AND level = :level '
      . 'ORDER BY id LIMIT 20';
  $stmt = $pdo->prepare($sql);
  **$stmt->bindParam('min',   $min);**
 **$stmt->bindParam('max',   $max);**
 **$stmt->bindParam('level', $level);**

  $min   =  5000;
  $max   = 10000;
  $level = 'ADV';
  $stmt->execute();
  showResults($stmt, $min, $max, $level);

  $min   = 0;
  $max   = 100;
  $level = 'BEG';
  $stmt->execute();
  showResults($stmt, $min, $max, $level);

} catch (PDOException $e) {
  echo $e->getMessage();
} catch (Throwable $e) {
  echo $e->getMessage();
}
```

当这些变量的值发生变化时，下一次执行将反映修改后的条件。

### 提示

**最佳实践**

对于一次性数据库命令，请使用`PDO::query()`。当您需要多次处理相同的语句但使用不同的值时，请使用`PDO::prepare()`和`PDOStatement::execute()`。

## 另请参阅

有关与不同供应商特定 PDO 驱动程序相关的语法和独特行为的信息，请参阅本文：

+   [`php.net/manual/en/pdo.drivers.php`](http://php.net/manual/en/pdo.drivers.php)

有关 PDO 预定义常量的摘要，包括获取模式、游标方向和属性，请参阅以下文章：

+   [`php.net/manual/en/pdo.constants.php`](http://php.net/manual/en/pdo.constants.php)

# 构建面向对象的 SQL 查询构建器

PHP 7 实现了一种称为**上下文敏感词法分析器**的东西。这意味着通常保留的单词可以在上下文允许的情况下使用。因此，当构建面向对象的 SQL 构建器时，我们可以使用命名为`and`、`or`、`not`等的方法。

## 如何做...

1.  我们定义一个`Application\Database\Finder`类。在这个类中，我们定义与我们喜欢的 SQL 操作相匹配的方法：

```php
namespace Application\Database;
class Finder
{
  public static $sql      = '';
  public static $instance = NULL;
  public static $prefix   = '';
  public static $where    = array();
  public static $control  = ['', ''];

    // $a == name of table
    // $cols = column names
    public static function select($a, $cols = NULL)
    {
      self::$instance  = new Finder();
      if ($cols) {
           self::$prefix = 'SELECT ' . $cols . ' FROM ' . $a;
      } else {
        self::$prefix = 'SELECT * FROM ' . $a;
      }
      return self::$instance;
    }

    public static function where($a = NULL)
    {
        self::$where[0] = ' WHERE ' . $a;
        return self::$instance;
    }

    public static function like($a, $b)
    {
        self::$where[] = trim($a . ' LIKE ' . $b);
        return self::$instance;
    }

    public static function and($a = NULL)
    {
        self::$where[] = trim('AND ' . $a);
        return self::$instance;
    }

    public static function or($a = NULL)
    {
        self::$where[] = trim('OR ' . $a);
        return self::$instance;
    }

    public static function in(array $a)
    {
        self::$where[] = 'IN ( ' . implode(',', $a) . ' )';
        return self::$instance;
    }

    public static function not($a = NULL)
    {
        self::$where[] = trim('NOT ' . $a);
        return self::$instance;
    }

    public static function limit($limit)
    {
        self::$control[0] = 'LIMIT ' . $limit;
        return self::$instance;
    }

    public static function offset($offset)
    {
        self::$control[1] = 'OFFSET ' . $offset;
        return self::$instance;
    }

  public static function getSql()
  {
    self::$sql = self::$prefix
       . implode(' ', self::$where)
               . ' '
               . self::$control[0]
               . ' '
               . self::$control[1];
    preg_replace('/  /', ' ', self::$sql);
    return trim(self::$sql);
  }
}
```

1.  用于生成 SQL 片段的每个函数都返回相同的属性`$instance`。这使我们能够使用流畅的接口来表示代码，例如：

```php
$sql = Finder::select('project')->where('priority > 9') ... etc.
```

## 它是如何工作的...

将前面定义的代码复制到`Application\Database`文件夹中的`Finder.php`文件中。然后，您可以创建一个调用程序`chap_05_oop_query_builder.php`，该程序初始化了第一章中定义的自动加载程序，*建立基础*。然后，您可以运行`Finder::select()`来生成一个对象，从中可以呈现 SQL 字符串：

```php
<?php
require __DIR__ . '/../Application/Autoload/Loader.php';
Application\Autoload\Loader::init(__DIR__ . '/..');
use Application\Database\Finder;

$sql = Finder::select('project')
  ->where()
  ->like('name', '%secret%')
  ->and('priority > 9')
  ->or('code')->in(['4', '5', '7'])
  ->and()->not('created_at')
  ->limit(10)
  ->offset(20);

echo Finder::getSql();
```

这是上述代码的结果：

![它是如何工作的...](img/B05314_05_04.jpg)

## 另请参阅

有关上下文敏感词法分析器的更多信息，请参阅本文：

[`wiki.php.net/rfc/context_sensitive_lexer`](https://wiki.php.net/rfc/context_sensitive_lexer)

# 处理分页

分页涉及提供数据库查询结果的有限子集。这通常是为了显示目的，但也可以轻松应用于其他情况。乍一看，似乎`LimitIterator`类非常适合分页的目的。然而，在潜在结果集可能非常庞大的情况下，`LimitIterator`并不是一个理想的选择，因为您需要提供整个结果集作为内部迭代器，这很可能会超出内存限制。`LimitIterator`类构造函数的第二个和第三个参数是偏移和计数。这表明我们将采用的分页解决方案，这是 SQL 本身的一部分：向给定的 SQL 语句添加`LIMIT`和`OFFSET`子句。

## 如何做...

1.  首先，我们创建一个名为`Application\Database\Paginate`的类来保存分页逻辑。我们添加属性来表示与分页相关的值，`$sql`、`$page`和`$linesPerPage`：

```php
namespace Application\Database;

class Paginate
{

  const DEFAULT_LIMIT  = 20;
  const DEFAULT_OFFSET = 0;

  protected $sql;
  protected $page;
  protected $linesPerPage;

}
```

1.  接下来，我们定义一个`__construct()`方法，它接受基本 SQL 语句、当前页码和每页行数作为参数。然后，我们需要重构 SQL 字符串，修改或添加`LIMIT`和`OFFSET`子句。

1.  在构造函数中，我们需要使用当前页码和每页行数来计算偏移量。我们还需要检查 SQL 语句中是否已经存在`LIMIT`和`OFFSET`。最后，我们需要使用每页行数作为我们的`LIMIT`，使用重新计算的`OFFSET`来修改语句：

```php
public function __construct($sql, $page, $linesPerPage)
{
  $offset = $page * $linesPerPage;
  switch (TRUE) {
    case (stripos($sql, 'LIMIT') && strpos($sql, 'OFFSET')) :
      // no action needed
      break;
    case (stripos($sql, 'LIMIT')) :
      $sql .= ' LIMIT ' . self::DEFAULT_LIMIT;
      break;
    case (stripos($sql, 'OFFSET')) :
      $sql .= ' OFFSET ' . self::DEFAULT_OFFSET;
      break;
    default :
      $sql .= ' LIMIT ' . self::DEFAULT_LIMIT;
      $sql .= ' OFFSET ' . self::DEFAULT_OFFSET;
      break;
  }
  $this->sql = preg_replace('/LIMIT \d+.*OFFSET \d+/Ui', 
     'LIMIT ' . $linesPerPage . ' OFFSET ' . $offset, 
     $sql);
}
```

1.  现在，我们已经准备好使用第一篇食谱中讨论的`Application\Database\Connection`类来执行查询。

1.  在我们的新分页类中，我们添加了一个`paginate()`方法，它以`Connection`实例作为参数。我们还需要 PDO 获取模式和可选的准备好的语句参数：

```php
use PDOException;
public function paginate(
  Connection $connection, 
  $fetchMode, 
  $params = array())
  {
  try {
    $stmt = $connection->pdo->prepare($this->sql);
    if (!$stmt) return FALSE;
    if ($params) {
      $stmt->execute($params);
    } else {
      $stmt->execute();
    }
    while ($result = $stmt->fetch($fetchMode)) yield $result;
  } catch (PDOException $e) {
    error_log($e->getMessage());
    return FALSE;
  } catch (Throwable $e) {
    error_log($e->getMessage());
    return FALSE;
  }
}
```

1.  为了提供对前面一篇食谱中提到的查询构建器类的支持可能是个好主意。这将使更新`LIMIT`和`OFFSET`变得更容易。我们需要做的就是为`Application\Database\Finder`提供支持，使用该类并修改`__construct()`方法以检查传入的 SQL 是否是这个类的实例：

```php
  if ($sql instanceof Finder) {
    $sql->limit($linesPerPage);
    $sql->offset($offset);
    $this->sql = $sql::getSql();
  } elseif (is_string($sql)) {
    switch (TRUE) {
      case (stripos($sql, 'LIMIT') 
      && strpos($sql, 'OFFSET')) :
          // remaining code as shown in bullet #3 above
      }
   }
```

1.  现在剩下要做的就是添加一个`getSql()`方法，以便在需要确认 SQL 语句是否正确形成时使用：

```php
public function getSql()
{
  return $this->sql;
}
```

## 它是如何工作的...

将上述代码复制到 `Application/Database` 文件夹中的 `Paginate.php` 文件中。然后可以创建一个 `chap_05_pagination.php` 调用程序，该程序初始化了第一章中定义的自动加载程序，*建立基础*：

```php
<?php
define('DB_CONFIG_FILE', '/../config/db.config.php');
define('LINES_PER_PAGE', 10);
define('DEFAULT_BALANCE', 1000);
require __DIR__ . '/../Application/Autoload/Loader.php';
Application\Autoload\Loader::init(__DIR__ . '/..');
```

接下来，使用 `Application\Database\Finder`、`Connection` 和 `Paginate` 类，创建一个 `Application\Database\Connection` 实例，并使用 `Finder` 生成 SQL：

```php
use Application\Database\ { Finder, Connection, Paginate};
$conn = new Connection(include __DIR__ . DB_CONFIG_FILE);
$sql = Finder::select('customer')->where('balance < :bal');
```

现在，我们可以从 `$_GET` 参数中获取页码和余额，并创建 `Paginate` 对象，结束 PHP 代码块：

```php
$page = (int) ($_GET['page'] ?? 0);
$bal  = (float) ($_GET['balance'] ?? DEFAULT_BALANCE);
$paginate = new Paginate($sql::getSql(), $page, LINES_PER_PAGE);
?>
```

在脚本的输出部分，我们只需使用简单的 `foreach()` 循环迭代通过分页：

```php
<h3><?= $paginate->getSql(); ?></h3>	
<hr>
<pre>
<?php
printf('%4s | %20s | %5s | %7s' . PHP_EOL, 
  'ID', 'NAME', 'LEVEL', 'BALANCE');
printf('%4s | %20s | %5s | %7s' . PHP_EOL, 
  '----', str_repeat('-', 20), '-----', '-------');
foreach ($paginate->paginate($conn, PDO::FETCH_ASSOC, 
  ['bal' => $bal]) as $row) {
  printf('%4d | %20s | %5s | %7.2f' . PHP_EOL, 
      $row['id'],$row['name'],$row['level'],$row['balance']);
}
printf('%4s | %20s | %5s | %7s' . PHP_EOL, 
  '----', str_repeat('-', 20), '-----', '-------');
?>
<a href="?page=<?= $page - 1; ?>&balance=<?= $bal ?>">
<< Prev </a>&nbsp;&nbsp;
<a href="?page=<?= $page + 1; ?>&balance=<?= $bal ?>">
Next >></a>
</pre>
```

以下是输出的第 3 页，余额小于 1,000：

![工作原理...](img/B05314_05_05.jpg)

## 另请参阅

有关 `LimitIterator` 类的更多信息，请参阅本文：

+   [`php.net/manual/en/class.limititerator.php`](http://php.net/manual/en/class.limititerator.php)

# 定义与数据库表匹配的实体

PHP 开发人员中非常常见的做法是创建代表数据库表的类。这些类通常被称为**实体**类，并且构成**领域模型**软件设计模式的核心。

## 如何做...

1.  首先，我们将确定一系列实体类的一些共同特征。这些可能包括共同的属性和共同的方法。我们将把这些放入 `Application\Entity\Base` 类中。然后，所有未来的实体类都将扩展 `Base`。

1.  为了说明的目的，让我们假设所有实体都有两个共同的属性：`$mapping`（稍后讨论）和 `$id`（及其相应的 getter 和 setter）：

```php
namespace Application\Entity;

class Base
{

  protected $id = 0;
  protected $mapping = ['id' => 'id'];

  public function getId() : int
  {
    return $this->id;
  }

  public function setId($id)
  {
    $this->id = (int) $id;
  }
}
```

1.  定义一个 `arrayToEntity()` 方法并不是一个坏主意，它将数组转换为实体类的实例，反之亦然（`entityToArray()`）。这些方法实现了一个经常被称为**水合**的过程。由于这些方法应该是通用的，因此最好将它们放在 `Base` 类中。

1.  在以下方法中，`$mapping` 属性用于在数据库列名和对象属性名之间进行转换。`arrayToEntity()` 从数组中填充此对象实例的值。我们可以定义此方法为静态方法，以防需要在活动实例之外调用它：

```php
public static function arrayToEntity($data, Base $instance)
{
  if ($data && is_array($data)) {
    foreach ($instance->mapping as $dbColumn => $propertyName) {
      $method = 'set' . ucfirst($propertyName);
      $instance->$method($data[$dbColumn]);
    }
    return $instance;
  }
  return FALSE;
}
```

1.  `entityToArray()` 从当前实例属性值生成数组：

```php
public function entityToArray()
{
  $data = array();
  foreach ($this->mapping as $dbColumn => $propertyName) {
    $method = 'get' . ucfirst($propertyName);
    $data[$dbColumn] = $this->$method() ?? NULL;
  }
  return $data;
}
```

1.  要构建特定的实体，您需要手头有要建模的数据库表的结构。创建映射到数据库列的属性。分配的初始值应反映数据库列的最终数据类型。

1.  在此示例中，我们将使用 `customer` 表。以下是来自 MySQL 数据转储的 `CREATE` 语句，说明了其数据结构：

```php
CREATE TABLE 'customer' (
  'id' int(11) NOT NULL AUTO_INCREMENT,
  'name' varchar(256) CHARACTER SET latin1 COLLATE latin1_general_cs NOT NULL,
  'balance' decimal(10,2) NOT NULL,
  'email' varchar(250) NOT NULL,
  'password' char(16) NOT NULL,
  'status' int(10) unsigned NOT NULL DEFAULT '0',
  'security_question' varchar(250) DEFAULT NULL,
  'confirm_code' varchar(32) DEFAULT NULL,
  'profile_id' int(11) DEFAULT NULL,
  'level' char(3) NOT NULL,
  PRIMARY KEY ('id'),
  UNIQUE KEY 'UNIQ_81398E09E7927C74' ('email')
);
```

1.  现在我们可以填充类属性。这也是确定相应表的好地方。在这种情况下，我们将使用 `TABLE_NAME` 类常量：

```php
namespace Application\Entity;

class Customer extends Base
{
  const TABLE_NAME = 'customer';
  protected $name = '';
  protected $balance = 0.0;
  protected $email = '';
  protected $password = '';
  protected $status = '';
  protected $securityQuestion = '';
  protected $confirmCode = '';
  protected $profileId = 0;
  protected $level = '';
}
```

1.  将属性定义为 `protected` 被认为是最佳实践。为了访问这些属性，您需要设计 `public` 方法来 `get` 和 `set` 属性。这是一个很好的地方，可以利用 PHP 7 对返回值进行数据类型定义。

1.  在以下代码块中，我们已经为 `$name` 和 `$balance` 定义了 getter 和 setter。您可以想象其余这些方法将如何定义：

```php
  public function getName() : string
  {
    return $this->name;
  }
  public function setName($name)
  {
    $this->name = $name;
  }
  public function getBalance() : float
  {
    return $this->balance;
  }
  public function setBalance($balance)
  {
    $this->balance = (float) $balance;
  }
}
```

### 提示

在 setter 中对传入值进行数据类型检查并不是一个好主意。原因是来自 RDBMS 数据库查询的返回值都将是 `string` 数据类型。

1.  如果属性名称与相应的数据库列不完全匹配，您应该考虑创建一个 `mapping` 属性，一个键/值对数组，其中键表示数据库列名，值表示属性名。

1.  您会注意到，三个属性`$securityQuestion`、`$confirmCode`和`$profileId`与它们对应的列名`security_question`、`confirm_code`和`profile_id`不对应。`$mapping`属性将确保适当的转换发生：

```php
protected $mapping = [
  'id'                => 'id',
  'name'              => 'name',
  'balance'           => 'balance',
  'email'             => 'email',
  'password'          => 'password',
  'status'            => 'status',
  'security_question' => 'securityQuestion',
  'confirm_code'      => 'confirmCode',
  'profile_id'        => 'profileId',
  'level'             => 'level'
];
```

## 它是如何工作的...

将步骤 2、4 和 5 中的代码复制到`Application/Entity`文件夹中的`Base.php`文件中。将步骤 8 到 12 中的代码复制到`Application/Entity`文件夹中的`Customer.php`文件中。然后，您需要为步骤 10 中未显示的剩余属性`email`、`password`、`status`、`securityQuestion`、`confirmCode`、`profileId`和`level`创建 getter 和 setter。

然后，您可以创建一个`chap_05_matching_entity_to_table.php`调用程序，该程序初始化了第一章中定义的自动加载程序，使用`Application\Database\Connection`和新创建的`Application\Entity\Customer`类：

```php
<?php
define('DB_CONFIG_FILE', '/../config/db.config.php');
require __DIR__ . '/../Application/Autoload/Loader.php';
Application\Autoload\Loader::init(__DIR__ . '/..');
use Application\Database\Connection;
use Application\Entity\Customer;
```

接下来，获取一个数据库连接，并使用连接随机获取一个客户的数据的关联数组：

```php
$conn = new Connection(include __DIR__ . DB_CONFIG_FILE);
$id = rand(1,79);
$stmt = $conn->pdo->prepare(
  'SELECT * FROM customer WHERE id = :id');
$stmt->execute(['id' => $id]);
$result = $stmt->fetch(PDO::FETCH_ASSOC);
```

最后，您可以从数组中创建一个新的`Customer`实体实例，并使用`var_dump()`查看结果：

```php
$cust = Customer::arrayToEntity($result, new Customer());
var_dump($cust);
```

以下是前面代码的输出：

![它是如何工作的...](img/B05314_05_06.jpg)

## 另请参阅

有许多描述领域模型的好作品。可能最有影响力的是 Martin Fowler 的*企业应用架构模式*（参见[`martinfowler.com/books/eaa.html`](http://martinfowler.com/books/eaa.html)）。还有一份很好的研究，也可以免费下载，名为*快速领域驱动设计*的 InfoQ（参见[`www.infoq.com/minibooks/domain-driven-design-quickly`](http://www.infoq.com/minibooks/domain-driven-design-quickly)）。

# 将实体类与 RDBMS 查询联系起来

大多数商业上可行的 RDBMS 系统是在过程式编程处于前沿时演变而来的。想象一下 RDBMS 世界是二维的、方形的、过程化的。相比之下，实体可以被认为是圆形的、三维的、面向对象的。这给了你一个关于我们想要通过将 RDBMS 查询的结果与实体实例的迭代联系起来实现的想法。

### 注意

**关系模型**，现代 RDBMS 系统所基于的模型，是由数学家 Edgar F. Codd 在 1969 年首次描述的。第一个商业上可行的系统是在 70 年代中期至 70 年代末期演变而来的。换句话说，RDBMS 技术已经有 40 多年的历史了！

## 如何做...

1.  首先，我们需要设计一个类，用于容纳我们的查询逻辑。如果你遵循领域模型，这个类可能被称为**存储库**。或者，为了保持简单和通用，我们可以简单地将新类称为`Application\Database\CustomerService`。该类将接受一个`Application\Database\Connection`实例作为参数：

```php
namespace Application\Database;

use Application\Entity\Customer;

class CustomerService
{

    protected $connection;

    public function __construct(Connection $connection)
    {
      $this->connection = $connection;
    }

}
```

1.  现在我们将定义一个`fetchById()`方法，它以客户 ID 作为参数，并在失败时返回单个`Application\Entity\Customer`实例或布尔值`FALSE`。乍一看，似乎很简单，只需简单地使用`PDOStatement::fetchObject()`并将实体类指定为参数：

```php
public function fetchById($id)
{
  $stmt = $this->connection->pdo
               ->prepare(Finder::select('customer')
               ->where('id = :id')::getSql());
  $stmt->execute(['id' => (int) $id]);
  return $stmt->fetchObject('Application\Entity\Customer');
}
```

### 注意

然而，这里的危险是`fetchObject()`实际上在调用构造函数之前填充属性（即使它们是受保护的）！因此，存在构造函数可能意外覆盖值的危险。如果你没有定义构造函数，或者如果你可以接受这种危险，那就完成了。否则，正确实现 RDBMS 查询和 OOP 结果之间的联系就开始变得更加困难。

1.  `fetchById()`方法的另一种方法是首先创建对象实例，从而运行其构造函数，并将获取模式设置为`PDO::FETCH_INTO`，如下例所示：

```php
public function fetchById($id)
{
  $stmt = $this->connection->pdo
               ->prepare(Finder::select('customer')
               ->where('id = :id')::getSql());
  $stmt->execute(['id' => (int) $id]);
  $stmt->setFetchMode(PDO::FETCH_INTO, new Customer());
  return $stmt->fetch();
}
```

1.  然而，我们在这里又遇到了一个问题：`fetch()`与`fetchObject()`不同，它无法覆盖受保护的属性；如果尝试覆盖，将生成以下错误消息。这意味着我们要么将所有属性定义为`public`，要么考虑另一种方法。![如何做...](img/B05314_05_07.jpg)

1.  我们将考虑的最后一种方法是以数组形式获取结果，并手动*hydrate*实体。尽管这种方法在性能方面略微昂贵，但它允许任何潜在的实体构造函数正常运行，并且可以安全地将属性定义为`private`或`protected`：

```php
public function fetchById($id)
{
  $stmt = $this->connection->pdo
               ->prepare(Finder::select('customer')
               ->where('id = :id')::getSql());
  $stmt->execute(['id' => (int) $id]);
  return Customer::arrayToEntity(
    $stmt->fetch(PDO::FETCH_ASSOC));
}
```

1.  要处理产生多个结果的查询，我们只需要生成填充的实体对象的迭代。在这个例子中，我们实现了一个`fetchByLevel()`方法，它以`Application\Entity\Customer`实例的形式返回给定级别的所有客户：

```php
public function fetchByLevel($level)
{
  $stmt = $this->connection->pdo->prepare(
            Finder::select('customer')
            ->where('level = :level')::getSql());
  $stmt->execute(['level' => $level]);
  while ($row = $stmt->fetch(PDO::FETCH_ASSOC)) {
    yield Customer::arrayToEntity($row, new Customer());
  }
}
```

1.  我们希望实现的下一个方法是`save()`。然而，在我们继续之前，必须考虑如果发生`INSERT`，将返回什么值。

1.  通常，我们会在`INSERT`后返回新完成的实体类。有一个方便的`PDO::lastInsertId()`方法，乍一看似乎可以解决问题。然而，进一步阅读文档后发现，并非所有的数据库扩展都支持这个特性，而且支持的数据库扩展在实现上也不一致。因此，最好有一个除了`$id`之外的唯一列，可以用来唯一标识新客户。

1.  在这个例子中，我们选择了`email`列，因此需要实现一个`fetchByEmail()`服务方法：

```php
public function fetchByEmail($email)
{
  $stmt = $this->connection->pdo->prepare(
    Finder::select('customer')
    ->where('email = :email')::getSql());
  $stmt->execute(['email' => $email]);
  return Customer::arrayToEntity(
    $stmt->fetch(PDO::FETCH_ASSOC), new Customer());
}
```

1.  现在我们准备定义`save()`方法。我们不再区分`INSERT`和`UPDATE`，而是将这个方法设计为如果 ID 已经存在，则更新，否则进行插入。

1.  首先，我们定义一个基本的`save()`方法，它接受一个`Customer`实体作为参数，并使用`fetchById()`来确定此条目是否已经存在。如果存在，我们调用一个`doUpdate()`更新方法；否则，我们调用一个`doInsert()`插入方法：

```php
public function save(Customer $cust)
{
  // check to see if customer ID > 0 and exists
  if ($cust->getId() && $this->fetchById($cust->getId())) {
    return $this->doUpdate($cust);
  } else {
    return $this->doInsert($cust);
  }
}
```

1.  接下来，我们定义`doUpdate()`，它将`Customer`实体对象的属性提取到一个数组中，构建一个初始的 SQL 语句，并调用`flush()`方法，将数据推送到数据库。我们不希望 ID 字段被更新，因为它是主键。另外，我们需要指定要更新的行，这意味着要添加一个`WHERE`子句：

```php
protected function doUpdate($cust)
{
  // get properties in the form of an array
  $values = $cust->entityToArray();
  // build the SQL statement
  $update = 'UPDATE ' . $cust::TABLE_NAME;
  $where = ' WHERE id = ' . $cust->getId();
  // unset ID as we want do not want this to be updated
  unset($values['id']);
  return $this->flush($update, $values, $where);
}
```

1.  `doInsert()`方法类似，只是初始的 SQL 需要以`INSERT INTO ...`开头，并且需要取消`id`数组元素。后者的原因是我们希望这个属性由数据库自动生成。如果成功，我们使用我们新定义的`fetchByEmail()`方法查找新客户并返回一个完成的实例：

```php
protected function doInsert($cust)
{
  $values = $cust->entityToArray();
  $email  = $cust->getEmail();
  unset($values['id']);
  $insert = 'INSERT INTO ' . $cust::TABLE_NAME . ' ';
  if ($this->flush($insert, $values)) {
    return $this->fetchByEmail($email);
  } else {
    return FALSE;
  }
}
```

1.  最后，我们可以定义`flush()`，它执行实际的准备和执行：

```php
protected function flush($sql, $values, $where = '')
{
  $sql .=  ' SET ';
  foreach ($values as $column => $value) {
    $sql .= $column . ' = :' . $column . ',';
  }
  // get rid of trailing ','
  $sql     = substr($sql, 0, -1) . $where;
  $success = FALSE;
  try {
    $stmt = $this->connection->pdo->prepare($sql);
    $stmt->execute($values);
    $success = TRUE;
  } catch (PDOException $e) {
    error_log(__METHOD__ . ':' . __LINE__ . ':' 
    . $e->getMessage());
    $success = FALSE;
  } catch (Throwable $e) {
    error_log(__METHOD__ . ':' . __LINE__ . ':' 
    . $e->getMessage());
    $success = FALSE;
  }
  return $success;
}
```

1.  为了结束讨论，我们需要定义一个`remove()`方法，它可以从数据库中删除一个客户。与之前定义的`save()`方法一样，我们再次使用`fetchById()`来确保操作成功：

```php
public function remove(Customer $cust)
{
  $sql = 'DELETE FROM ' . $cust::TABLE_NAME . ' WHERE id = :id';
  $stmt = $this->connection->pdo->prepare($sql);
  $stmt->execute(['id' => $cust->getId()]);
  return ($this->fetchById($cust->getId())) ? FALSE : TRUE;
}
```

## 它是如何工作的...

将步骤 1 到 5 中描述的代码复制到`Application/Database`文件夹中的`CustomerService.php`文件中。定义一个`chap_05_entity_to_query.php`调用程序。让调用程序初始化自动加载器，使用适当的类：

```php
<?php
define('DB_CONFIG_FILE', '/../config/db.config.php');
require __DIR__ . '/../Application/Autoload/Loader.php';
Application\Autoload\Loader::init(__DIR__ . '/..');
use Application\Database\Connection;
use Application\Database\CustomerService;
```

现在，您可以创建一个服务的实例，并随机获取一个客户。然后服务将返回一个客户实体作为结果：

```php
// get service instance
$service = new CustomerService(new Connection(include __DIR__ . DB_CONFIG_FILE));

echo "\nSingle Result\n";
var_dump($service->fetchById(rand(1,79)));
```

这是输出：

![它是如何工作的...](img/B05314_05_08.jpg)

现在将步骤 6 到 15 中显示的代码复制到服务类中。将要插入的数据添加到`chap_05_entity_to_query.php`调用程序中。然后使用这些数据生成一个`Customer`实体实例：

```php
// sample data
$data = [
  'name'              => 'Doug Bierer',
  'balance'           => 326.33,
  'email'             => 'doug' . rand(0,999) . '@test.com',
  'password'          => 'password',
  'status'            => 1,
  'security_question' => 'Who\'s on first?',
  'confirm_code'      => 12345,
  'level'             => 'ADV'
];

// create new Customer
$cust = Customer::arrayToEntity($data, new Customer());
```

然后我们可以在调用`save()`之前和之后检查 ID：

```php
echo "\nCustomer ID BEFORE Insert: {$cust->getId()}\n";
$cust = $service->save($cust);
echo "Customer ID AFTER Insert: {$cust->getId()}\n";
```

最后，我们修改余额，然后再次调用`save（）`，查看结果：

```php
echo "Customer Balance BEFORE Update: {$cust->getBalance()}\n";
$cust->setBalance(999.99);
$service->save($cust);
echo "Customer Balance AFTER Update: {$cust->getBalance()}\n";
var_dump($cust);
```

这是调用程序的输出：

![工作原理...](img/B05314_05_09.jpg)

## 还有更多...

有关关系模型的更多信息，请参阅[`en.wikipedia.org/wiki/Relational_model`](https://en.wikipedia.org/wiki/Relational_model)。有关 RDBMS 的更多信息，请参阅[`en.wikipedia.org/wiki/Relational_database_management_system`](https://en.wikipedia.org/wiki/Relational_database_management_system)。有关`PDOStatement::fetchObject（）`在构造函数之前插入属性值的信息，请查看 php.net 文档参考中关于`fetchObject（）`的"rasmus at mindplay dot dk"的评论（[`php.net/manual/en/pdostatement.fetchobject.php`](http://php.net/manual/en/pdostatement.fetchobject.php)）。

# 将辅助查找嵌入到查询结果中

在实现实体类之间的关系之路上，让我们首先看一下如何嵌入执行辅助查找所需的代码。这样一个查找的示例是，在显示客户信息时，视图逻辑执行第二次查找，获取该客户的购买列表。

### 注意

这种方法的优势在于，处理被推迟直到实际视图逻辑被执行。这将最终平滑性能曲线，工作负载在客户信息的初始查询和后来的购买信息查询之间更均匀地分布。另一个好处是避免了大量的`JOIN`及其固有的冗余数据。

## 如何做...

1.  首先，定义一个根据其 ID 查找客户的函数。为了说明这一点，我们将简单地使用`PDO::FETCH_ASSOC`的获取模式获取一个数组。我们还将继续使用第一章中讨论的`Application\Database\Connection`类，*建立基础*：

```php
function findCustomerById($id, Connection $conn)
{
  $stmt = $conn->pdo->query(
    'SELECT * FROM customer WHERE id = ' . (int) $id);
  $results = $stmt->fetch(PDO::FETCH_ASSOC);
  return $results;
}
```

1.  接下来，我们分析购买表，看看`customer`和`product`表是如何关联的。从这个表的`CREATE`语句中可以看出，`customer_id`和`product_id`外键形成了关系：

```php
CREATE TABLE 'purchases' (
  'id' int(11) NOT NULL AUTO_INCREMENT,
  'transaction' varchar(8) NOT NULL,
  'date' datetime NOT NULL,
  'quantity' int(10) unsigned NOT NULL,
  'sale_price' decimal(8,2) NOT NULL,
  'customer_id' int(11) DEFAULT NULL,
  'product_id' int(11) DEFAULT NULL,
  PRIMARY KEY ('id'),
  KEY 'IDX_C3F3' ('customer_id'),
  KEY 'IDX_665A' ('product_id'),
  CONSTRAINT 'FK_665A' FOREIGN KEY ('product_id') 
  REFERENCES 'products' ('id'),
  CONSTRAINT 'FK_C3F3' FOREIGN KEY ('customer_id') 
  REFERENCES 'customer' ('id')
);
```

1.  我们现在扩展原始的`findCustomerById（）`函数，定义形式为匿名函数的辅助查找，然后可以在视图脚本中执行。将匿名函数分配给`$results['purchases']`元素：

```php
function findCustomerById($id, Connection $conn)
{
  $stmt = $conn->pdo->query(
       'SELECT * FROM customer WHERE id = ' . (int) $id);
  $results = $stmt->fetch(PDO::FETCH_ASSOC);
  if ($results) {
    $results['purchases'] = 
      // define secondary lookup
 **function ($id, $conn) {**
 **$sql = 'SELECT * FROM purchases AS u '**
 **. 'JOIN products AS r '**
 **. 'ON u.product_id = r.id '**
 **. 'WHERE u.customer_id = :id '**
 **. 'ORDER BY u.date';**
 **$stmt = $conn->pdo->prepare($sql);**
 **$stmt->execute(['id' => $id]);**
 **while ($row = $stmt->fetch(PDO::FETCH_ASSOC)) {**
 **yield $row;**
 **}**
 **};**
  }
  return $results;
}
```

1.  假设我们已成功将客户信息检索到`$results`数组中，在视图逻辑中，我们所需要做的就是循环遍历匿名函数的返回值。在这个例子中，我们随机检索客户信息：

```php
$result = findCustomerById(rand(1,79), $conn);
```

1.  在视图逻辑中，我们循环遍历辅助查找返回的结果。嵌入的匿名函数的调用在以下代码中突出显示：

```php
<table>
  <tr>
<th>Transaction</th><th>Date</th><th>Qty</th>
<th>Price</th><th>Product</th>
  </tr>
<?php 
foreach (**$result'purchases' as $purchase) : ?>
  <tr>
    <td><?= $purchase['transaction'] ?></td>
    <td><?= $purchase['date'] ?></td>
    <td><?= $purchase['quantity'] ?></td>
    <td><?= $purchase['sale_price'] ?></td>
    <td><?= $purchase['title'] ?></td>
  </tr>
<?php endforeach; ?>
</table>
```

## 工作原理...

创建一个`chap_05_secondary_lookups.php`调用程序，并插入所需的代码以创建`Application\Database\Connection`的实例：

```php
<?php
define('DB_CONFIG_FILE', '/../config/db.config.php');
include __DIR__ . '/../Application/Database/Connection.php';
use Application\Database\Connection;
$conn = new Connection(include __DIR__ . DB_CONFIG_FILE);
```

接下来，在步骤 3 中显示的`findCustomerById（）`函数中添加。然后，您可以获取随机客户的信息，结束调用程序的 PHP 部分：

```php
function findCustomerById($id, Connection $conn)
{
  // code shown in bullet #3 above
}
$result = findCustomerById(rand(1,79), $conn);
?>
```

对于视图逻辑，您可以显示核心客户信息，就像在前面的几个示例中所示的那样：

```php
<h1><?= $result['name'] ?></h1>
<div class="row">
<div class="left">Balance</div>
<div class="right"><?= $result['balance']; ?></div>
</div>
<!-- etc.l -->
```

您可以这样显示购买信息：

```php
<table>
<tr><th>Transaction</th><th>Date</th><th>Qty</th>
<th>Price</th><th>Product</th></tr>
  <?php 
  foreach **($result'purchases' as $purchase)** : ?>
  <tr>
    <td><?= $purchase['transaction'] ?></td>
    <td><?= $purchase['date'] ?></td>
    <td><?= $purchase['quantity'] ?></td>
    <td><?= $purchase['sale_price'] ?></td>
    <td><?= $purchase['title'] ?></td>
  </tr>
<?php endforeach; ?>
</table>
```

关键的一点是，通过调用嵌入的匿名函数`$result'purchases'`，辅助查找作为视图逻辑的一部分执行。这是输出：

![工作原理...](img/B05314_05_10.jpg)

# 实现 jQuery DataTables PHP 查找

进行次要查找的另一种方法是让前端生成请求。在这个食谱中，我们将对前面食谱中介绍的次要查找代码进行轻微修改，将次要查找嵌入到 QueryResults 中。在以前的食谱中，即使视图逻辑执行查找，所有处理仍然在服务器上完成。但是，当使用**jQuery DataTables**时，次要查找实际上是由客户端直接执行的，以**异步 JavaScript 和 XML**（**AJAX**）请求的形式由浏览器发出。

## 如何做...

1.  首先，我们需要将上面讨论的次要查找逻辑（在上面的食谱中讨论）分离到一个单独的 PHP 文件中。这个新脚本的目的是执行次要查找并返回一个 JSON 数组。

1.  新的脚本我们将称之为`chap_05_jquery_datatables_php_lookups_ajax.php`。它寻找一个`$_GET`参数，`id`。请注意，`SELECT`语句非常具体，以确定传递了哪些列。您还会注意到，提取模式已更改为`PDO::FETCH_NUM`。您可能还会注意到，最后一行将结果取出并将其分配给 JSON 编码数组中的`data`键。

### 提示

在处理零配置 jQuery DataTables 时，非常重要的一点是只返回与标题匹配的确切列数。

```php
$id  = $_GET['id'] ?? 0;
sql = 'SELECT u.transaction,u.date, **u.quantity,u.sale_price,r.title '**
   . 'FROM purchases AS u '
   . 'JOIN products AS r '
   . 'ON u.product_id = r.id '
   . 'WHERE u.customer_id = :id';
$stmt = $conn->pdo->prepare($sql);
$stmt->execute(['id' => (int) $id]);
$results = array();
while ($row = $stmt->fetch(**PDO::FETCH_NUM**)) {
  $results[] = $row;
}
echo json_encode(['data' => $results]); 
```

1.  接下来，我们需要修改通过 ID 检索客户信息的函数，删除在前面食谱中嵌入的次要查找：

```php
function findCustomerById($id, Connection $conn)
{
  $stmt = $conn->pdo->query(
    'SELECT * FROM customer WHERE id = ' . (int) $id);
  $results = $stmt->fetch(PDO::FETCH_ASSOC);
  return $results;
}
```

1.  之后，在视图逻辑中，我们导入最少的 jQuery，DataTables 和样式表，以实现零配置。至少，您将需要 jQuery 本身（在本例中为`jquery-1.12.0.min.js`）和 DataTables（`jquery.dataTables.js`）。我们还添加了一个方便的与 DataTables 关联的样式表，`jquery.dataTables.css`：

```php
<!DOCTYPE html>
<head>
  <script src="https://code.jquery.com/jquery-1.12.0.min.js">
  </script>
    <script type="text/javascript" 
      charset="utf8" 
      src="//cdn.datatables.net/1.10.11/js/jquery.dataTables.js">
    </script>
  <link rel="stylesheet" 
    type="text/css" 
    href="//cdn.datatables.net/1.10.11/css/jquery.dataTables.css">
</head>
```

1.  然后我们定义一个 jQuery 文档`ready`函数，将一个表格与 DataTables 关联起来。在这种情况下，我们将 id 属性`customerTable`分配给将分配给 DataTables 的表元素。您还会注意到，我们将 AJAX 数据源指定为步骤 1 中定义的脚本`chap_05_jquery_datatables_php_lookups_ajax.php`。由于我们有`$id`可用，因此将其附加到数据源 URL 中：

```php
<script>
$(document).ready(function() {
  $('#customerTable').DataTable(
    { "ajax": '/chap_05_jquery_datatables_php_lookups_ajax.php?id=<?= $id ?>' 
  });
} );
</script>
```

1.  在视图逻辑的主体中，我们定义表格，确保`id`属性与前面的代码中指定的一致。我们还需要定义标题，以匹配响应 AJAX 请求呈现的数据：

```php
<table id="customerTable" class="display" cellspacing="0" width="100%">
  <thead>
    <tr>
      <th>Transaction</th>
      <th>Date</th>
      <th>Qty</th>
      <th>Price</th>
      <th>Product</th>
    </tr>
  </thead>
</table>
```

1.  现在，剩下的就是加载页面，选择客户 ID（在这种情况下是随机选择），并让 jQuery 发出次要查找的请求。

## 工作原理...

创建一个`chap_05_jquery_datatables_php_lookups_ajax.php`脚本，用于响应 AJAX 请求。在其中，放置初始化自动加载和创建`Connection`实例的代码。然后，您可以附加前面食谱中步骤 2 中显示的代码：

```php
<?php
define('DB_CONFIG_FILE', '/../config/db.config.php');
include __DIR__ . '/../Application/Database/Connection.php';
use Application\Database\Connection;
$conn = new Connection(include __DIR__ . DB_CONFIG_FILE);
```

接下来，创建一个`chap_05_jquery_datatables_php_lookups.php`调用程序，将随机客户的信息提取出来。添加前面代码中描述的步骤 3 中的函数：

```php
<?php
define('DB_CONFIG_FILE', '/../config/db.config.php');
include __DIR__ . '/../Application/Database/Connection.php';
use Application\Database\Connection;
$conn = new Connection(include __DIR__ . DB_CONFIG_FILE);
// add function findCustomerById() here
$id     = random_int(1,79);
$result = findCustomerById($id, $conn);
?>
```

调用程序还将包含导入最少 JavaScript 以实现 jQuery DataTables 的视图逻辑。您可以添加前面代码中显示的步骤 3 中的代码。然后，添加文档`ready`函数和显示逻辑，如步骤 5 和 6 中所示。这是输出：

![工作原理...](img/B05314_05_11.jpg)

## 还有更多...

有关 jQuery 的更多信息，请访问他们的网站[`jquery.com/`](https://jquery.com/)。要了解有关 jQuery 的 DataTables 插件的信息，请参阅此文章[`www.datatables.net/`](https://www.datatables.net/)。零配置数据表的讨论在[`datatables.net/examples/basic_init/zero_configuration.html`](https://datatables.net/examples/basic_init/zero_configuration.html)。有关 AJAX 数据来源的更多信息，请查看[`datatables.net/examples/data_sources/ajax.html`](https://datatables.net/examples/data_sources/ajax.html)。
