# 第五章：配置和使用数据库

在本章中，我们将涵盖：

+   连接到数据库

+   执行简单查询

+   使用 TableGateway 执行查询

+   使用 DB 分析器进行优化

+   创建数据库访问对象

# 简介

显然，如果我们想存储数据，数据库是必不可少的，而且由于有各种各样的数据库引擎，有时候很难看清事物的本质。然而，Zend Framework 2 为我们带来了一丝标准化的希望。在本章中，我们将展示从数据库连接到优化查询性能的大量示例。

## 可用的默认数据库引擎

Zend Framework 2 提供了一组默认的数据库驱动程序可供使用，并且显然它也支持 PHP PDO 扩展，以实现更标准化的数据库使用方式。

### IBM DB2 驱动程序

**IBM DB2**是由 IBM 设计的一个数据库服务器，根据 IDC 2009 年的报告，它是第二常用的数据库管理系统（DBMS）([`www.marketresearch.com/IDC-v2477/Worldwide-Database-Management-Systems-Forecast-2393193/view-stat/ibm-14.html`](http://www.marketresearch.com/IDC-v2477/Worldwide-Database-Management-Systems-Forecast-2393193/view-stat/ibm-14.html))。数据库引擎可以追溯到 20 世纪 70 年代，直到 20 世纪 90 年代才开始支持其他更广泛使用的操作系统。

现在，DB2 主要在 ZF2 的 IBM i Power Systems（如 AS/400）中使用，但仍然是一个非常强大的数据库引擎。

要求：

+   IBM DB2 通用数据库客户端需要安装在 PHP 机器上

+   PHP 配置为使用`--with-IBM_DB2`选项，或者在`php.ini`中启用（并安装）了`ibm_db2`扩展

### MySQLi 驱动程序

对于 PHP 开发者来说，这可能是最常用的数据库引擎，**MySQLi**而不是普通的 MySQL 驱动程序，在现代 MySQL 系统版本（4.1.3 及更高版本）中为扩展提供了几个优势。这个改进的扩展支持以下现代 MySQL 功能：

+   增强的服务器支持

+   事务支持

+   预处理语句支持

+   面向对象接口

+   多语句支持

+   增强的调试可用性

MySQLi 驱动程序的要求是 PHP 配置为使用`--with-mysql`或`--with-mysqli`选项，或者在`php.ini`中启用（并安装）了`mysql`和`mysqli`扩展。

### OCI8 驱动程序

OCI8 驱动程序支持 Oracle 数据库 11g、10g、9i 和 8i（根据 PHP 手册），在 PHP 社区中广泛使用。

要求：

+   PHP 机器上的 Oracle 9ir2、10g 或 11g 客户端库

+   PHP 配置为使用`--with-oci8`选项，或者在`php.ini`中启用（并安装）了`oci8`扩展

### PGSQL 驱动程序

**PostgreSQL**是一个对象关系型数据库，也是我个人最喜欢的数据库，这个数据库自 1995 年以来一直存在，并被 Reddit、Instagram 和 Yahoo!等网站使用。

对于此要求，PHP 需要配置为带有`--with-pgsql`选项，或者在`php.ini`中启用（并安装）`pgsql`扩展。

### SQLSRV 驱动程序

微软 SQL Server（以及 SQL Azure）是一个仅在 Microsoft Windows 上运行的数据库，并且普遍被认为是一个非常良好且稳定的数据库引擎。PHP 扩展的 3.0 或更高版本支持 SQL Server 2005。

要求：

+   需要在 PHP 机器上安装 Microsoft SQL Server 2012 Native Client

+   应在 PHP 机器上启用（并安装）`php_sqlsrv_5*_nts.dll`或`php_sqlsrv_5*_ts.dll`

### PDO 驱动程序

PHP 中的 PDO 扩展可能是连接数据库的最佳方法。它不仅支持广泛的数据库引擎，而且还有更标准化的方式与它们交互，这使得长期支持变得更加容易（这也是长期的优势）。

不仅支持起来更容易，例如，其标准化的数据库连接和查询执行方式使得我们开发者切换起来更加容易。

对于此要求，至少需要在`php.ini`文件中启用一个`pdo`扩展，否则它将无法工作。

所有驱动程序都通过内置编译或作为库的扩展与 PHP 通信。没有这些扩展，PHP 将无法确定如何与特定库通信。一些扩展（如 Oracle 扩展）甚至需要更多，例如客户端库才能使其工作。

我们应该始终检查 php.net 文档，以了解我们尝试启用的特定扩展的要求。

# 连接到数据库

在看到 Zend Framework 2 支持的数据库类型之后，我们终于可以开始连接到它们了。在这个菜谱中，我们将连接到 MySQL 服务器，并展示不同的连接方式。

## 准备工作

为了充分利用以下菜谱，应使用一个带有 MySQL 服务器可供连接的 Zend Framework 2 骨架应用程序。别忘了连接到 MySQL 服务器需要在 PHP 中启用`mysql`和`mysqli`扩展。

## 如何操作…

在这个菜谱中，我们将给出一些如何连接到单个数据库或多个数据库的示例。

### 通过配置连接到 MySQL 数据库

我们可以对`/config/autoload/global.php`文件进行以下更改：

```php
<?php

return array(

  // Set up the service manager
  'service_manager' => array(

    // Initiate the connection at the start of the 
    // application
    'factories' => array(

      // Use the service factory to start up our db 
      // adapter
      'Zend\Db\Adapter\Adapter' =>
      'Zend\Db\Adapter\AdapterServiceFactory',
    ),

    'aliases' => array(
      // Use this db alias in the controllers to get the 
      // initialized connection. The value of the db key refers to 
      // the factories key with the same name.
      'db' => 'Zend\Db\Adapter\Adapter',
    ),
  ),
  'db' => array(
    // We want to use the PDO to connect to the database
    'driver' => 'pdo',

    // DSN, or data source name is a connection url that 
    // shows the driver (in this case the PDO) where to 
    // connect to. The first bit is the driver to use, 
    // then follows the database name and the host. More    
    // information on the dsn options can be found here:
    // http://php.net/manual/en/pdo.construct.php
    'dsn' => 'mysql:dbname=some_db_name;host=localhost',

    // Username and password (or at the very least the 
    // password) should NOT be in the global.php. This 
    // file usually will be committed to a version 
    // control, which means your password will be 
    // publicly available.
    'username'  => 'aGreatUser',
    'password'  => 'somePassword',
  ),
);
```

如我们在示例中所见，设置数据库配置相当简单。现在，如果你想知道如何在现实世界的例子中使用这样的配置，让我们考虑以下控制器：

```php
<?php

namespace Application\Controller;

use Zend\Mvc\Controller\AbstractActionController;

class SomeController extends AbstractActionController
{
  public function indexAction()
  {
    // Get the db adapter through our service manager
    $db = $this->getServiceLocator()->get('db');

    // Now we can execute queries
    $query = $db->query('SELECT * FROM table');
  }
}
```

如我们所见，现在在控制器中启动它非常容易。

### 通过配置连接到多个数据库

一些应用程序要求我们同时连接到多个数据库，我们也可以在 Zend Framework 2 中通过在`/config/autoload/global.php`文件中执行以下操作轻松实现：

```php
<?php
return array(
  'db' => array(
    'adapters' => array(
      // The first (default) database connection
      'db_one' => array(
        'driver' => 'pdo',
        'dsn' => 'mysql:dbname=db_1;host=localhost',
        'username'  => 'someUser',
        'password'  => 'aGreatPassword',
      ),

      // Now the second database connection
      'db_two' => array(
        'driver' => 'pdo',
        'dsn' => 'mysql:dbname=db_2;host=localhost',
        'username'  => 'someOtherUser',
        'password'  => 'anotherGreatPassword',
      ),
    ),
  ),
  'service_manager' => array(
    // Let's make sure our adapters get instantiated
    'abstract_factories' => array(      
      'Zend\Db\Adapter\AdapterAbstractServiceFactory',
    ),
  ),
);
```

在我们的控制器（或我们能够访问服务管理器的地方）中，我们可以通过以下方式轻松地获取 `db` DBAdapter：

```php
<?php

namespace Application\Controller;

use Zend\Mvc\Controller\AbstractActionController;

class SomeController extends AbstractActionController
{
  public function indexAction()
  {
    // Get the first db adapter
    $dbOne = $this->getServiceLocator()->get('db_one'); 

    // Get the second db adapter
    $dbOne = $this->getServiceLocator()->get('db_two');
  }
}
```

### 通过代码连接到 MySQL 数据库

虽然这种方法不如我们之前展示的方法干净，但有时通过传统的实例化连接是必要的。

首先，让我们看一个示例，如果我们想连接到一个 MySQL 服务器：

```php
<?php 

// We need to import this to use the Db Adapter
use Zend\Db\Adapter\Adapter;

class someClass 
{
  // This is the property where our database adapter will be 
  // stored in 
  private $db;

  // First we want to connect to the database on instantiation of 
  // this class
  public function __construct()
  {
    // Create the new database adapter
    $this->db = new Adapter(array(
      'driver' => 'Pdo_Mysql', 
      'hostname' => 'localhost',
      'database' => 'example_database',
      'username' => 'developer',
      'password' => 'developer-password'
    ));
  }

  // This method will execute a query on the database, to show 
  // how easy it is to now make use of our database
  public function someData()
  {
    // Create a statement where we select everything from our 
    // tableName table
    $statement = $this->db->createStatement(
      "SELECT * FROM tableName"
    );

    return $statement->execute();
  }
}
```

现在，我们可以轻松地在实例化的 `$db` 上执行查询。

## 它是如何工作的…

在 Zend Framework 2 中，有许多定义数据库连接的方法，在本节中，我们将讨论其中的三种。

### 通过配置连接到 MySQL 数据库

我们将要展示的第一个方法是通过配置文件连接到一个（可以是任何类型）数据库。这可能是最简单的方法，但显然并不总是我们想要的。然而，在代码越少越好维护的情况下，我们应该始终考虑以这种方式连接到数据库的选项。

我们应该避免在控制器中放置业务逻辑，因为 MVC 不是为此而设计的，我们在这里只是作为示例展示。我们可以从任何有服务管理器的地方获取 db 适配器。

### 通过配置连接到多个数据库

如我们所见，我们现在在 `db => adapters` 数组中定义了适配器，而不是直接在 `db` 数组中。这种功能可以在任何版本大于或等于 2.2 的 Zend Framework 2 中实现。

### 关于 ServiceManager

当我们使用 `ServiceManager` 连接到我们的数据库时，`ServiceManager` 首先检查它是否有我们需要的键。如果找到了键，它首先在其内部注册表中检查是否有请求服务的实例。如果没有，它将使用 `config` 数据来实例化它。实例化完成后，它将在其内部注册表中保存引用，这样我们下次请求时可以再次检索它。这样，数据库适配器（或任何其他服务）将由 `ServiceManager` 只实例化一次。以这种方式实例化数据库连接有几个优点：

+   我们始终有一个数据库连接，这通常在服务器端有限制

+   我们不会花费宝贵的时间不断连接和重新初始化连接

+   没有多余的内存浪费在多个实例上

# 执行简单查询

查询数据库显然是我们连接到数据库后需要做的事情。这个配方解释了如何做到这一点，以及可用的不同方法。

## 准备工作

为了充分利用以下配方，应该使用一个 Zend Framework 2 骨架应用程序，并确保有一个可连接的 MySQL 服务器。别忘了连接到 MySQL 服务器需要在 PHP 中启用 `mysql` 和 `mysqli` 扩展。

我们已经配置了一个名为 `book` 的数据库，其中包含一个名为 `cards` 的表，该表有 `id`，`color`，`type` 和 `value` 列。创建数据库和表的 SQL 查询包含在本书的代码中。

## 如何做到这一点…

查询以各种形式出现，在这个菜谱中，我们将讨论一些基本的查询。

### 使用原始 SQL

对于这个示例，我们将编辑 `/module/Application/src/Application/Controller/IndexController.php` 文件：

```php
<?php 

namespace Application\Controller;

use Zend\Mvc\Controller\AbstractActionController;

class IndexController extends AbstractActionController
{
  public function indexAction()
  {
    // Let's assume there is a service called 'db' that connect to 
    // the database
    $connection = $this->getServiceLocator()->get('db');

    // We now start to build up our query
    $query = $connection->query(
      // We will put our raw SQL statement in here, and 
      // every variable we want to put in we replace with 
      // a question mark. This means we will fill in the 
      // blanks later.
      "SELECT * FROM cards WHERE type = ?", 

      // We don't want to execute the statement yet, just 
      // prepare it.
      Adapter::QUERY_MODE_PREPARE
    );

    // These are the parameters that will replace the question 
    // marks (?) in the SQL statement above, in the defined order
    $replacements = array('number');

    // Now execute the query with the parameters attached to 
    // replace
    $result = $query->execute($replacements);

    // Iterate over the results
    foreach ($result as $res) {
      // Do something with the result, in this case a raw echo
      echo '<pre>'. print_r($res, true). '</pre>';
    }
  }
}
```

使用数组或 `ParameterContainer` 对象传递变量的示例：

```php
// We now start to build up our query
$query = $connection->query(
  // We will put our raw SQL statement in here, and 
  // every variable we want to put in we replace with 
  // a question mark. This means we will fill in the 
  // blanks later.
  "SELECT * FROM cards WHERE type = ?", 

  // These are the parameters that will replace the question 
  // marks (?) in the SQL statement above, in the defined order
  array('number')
);
```

### 使用预处理语句

对于这个示例，我们将编辑 `/module/Application/src/Application/Controller/IndexController.php` 文件：

```php
<?php 

namespace Application\Controller;

use Zend\Mvc\Controller\AbstractActionController;

class IndexController extends AbstractActionController
{
  public function indexAction()
  {
    // Let's assume there is a service called 'db' that connect to 
    // the database
    $connection = $this->getServiceLocator()->get('db');

    // Now let's create a prepared statement
    $statement = $connection->createStatement();

    // Set up the prepared statement
    $statement->setSql("
      SELECT 
      * 
      FROM cards 
      WHERE type = :type
      AND color = :color
    ");

    // Create a new parameter container to store our where 
    // parameters in
    $container = new ParameterContainer(array(
      // These are the variables used in the same order as 
      // displayed in the where condition
      'type' => 'picture', 'color' => 'diamond'
    ));

    // Set the container to be used in our statement 
    $statement->setParameterContainer($container);

    // Prepare the statement for use with the database
    $statement->prepare();

    // Now execute the statement and get the resultset
    $result = $statement->execute();

    // Iterate over the results
    foreach ($result as $res) {
      // Do something with the result, in this case a raw echo
    echo '<pre>'. print_r($res, true). '</pre>';
    }
  }
}
```

### 引用标识符

此方法将以安全的方式引用将在 SQL 查询中使用的标识符：

```php
<?php

// Adapter is of type Zend\Db\Adapter\Adapter
echo $adapter->getPlatform()->quoteIdentifier('some_var');
```

上述代码将产生以下输出：

```php
"some_var"
```

### 引用标识符链

`quoteIdentifierChain` 方法将引用多个标识符，并用标识符分隔符（见方法 `getIdentifierSeparator()`）将它们粘合在一起：

```php
<?php

// Adapter is of type Zend\Db\Adapter\Adapter
echo $adapter->getPlatform()->quoteIdentifierChain(array(
  'some_table', 'some_column'
));
```

上述代码将产生以下输出：

```php
"some_table"."some_column"
```

### 引用（可信）值

`quoteValue` 和 `quoteTrustedValue` 用于引用在 WHERE 子句中使用的值。`quoteTrustedValue()` 应仅在我们信任值时使用（例如，如果我们自己放入）：以下是一个 `quoteValue` 和 `quoteTrustedValue` 的示例：

```php
<?php

// You can either use quoteValue or quoteTrustedValue,
// quoteValue will log an error in the PHP error log if 
// there is no driver or module available to quote the 
// value. Both methods output the same value.
echo $adapter->getPlatform()->quoteValue("great-value");

// Adapter is of type Zend\Db\Adapter\Adapter
echo $adapter->getPlatform()->quoteTrustedValue("great-value");
```

上述代码将产生以下输出：

```php
'great-value'
```

### 引用值列表

引用值列表引用整个值列表并返回它们，用逗号分隔。例如，如果我们想在 `WHERE` 子句中使用 `IN` 操作符，这个功能就很有用。没有处理可信值的方法，因此我们应该意识到，如果没有可用的驱动程序或模块来引用值，这可能会在我们的 PHP 错误日志中触发错误，然而，它总是会返回预期的值。以下是一个 `quoteValueList` 的示例：

```php
<?php

// Adapter is of type Zend\Db\Adapter\Adapter
echo $adapter->getPlatform()->quoteValueList(array(
  "value_one", "value_two"
));
```

上述代码将产生以下输出：

```php
'value_one', 'value_two'
```

### 在片段中引用标识符

`quoteIdentifierInFragment` 方法通过正则表达式模式提取标识符，并确保只引用正确的标识符。如果我们使用以下字符之外的字符：A-z，0-9，*，"." 或 'AS'，我们需要通过使用第二个参数将它们作为安全词放弃。

```php
<?php

// Adapter is of type Zend\Db\Adapter\Adapter
echo $adapter->getPlatform()->quoteIdentifierInFragment(
  '(fork.* AS spoon)',

   // Use the braces as a safe word so that they
   // will not be quoted.
   array('(', ')')
);
```

上述代码将产生以下输出：

```php
`fork`.* AS `spoon`
```

## 它是如何工作的…

让我们理解我们刚才所做的操作。

### 使用原始 SQL

执行 SQL 的第一种方法是在数据库连接上简单地使用 `query()` 方法。这是查询的最简单形式，它既有优点也有缺点，一个优点是查询快速简单，缺点是它实际上并不适用于重用，因为每次执行查询时都需要新的输入，或者每次想要执行查询时都需要传递变量。

如示例所示，我们首先以`QUERY_MODE_PREPARE`模式创建了一个查询，这意味着查询不会立即执行，而是仅准备执行。当我们执行查询时，我们会看到我们使用`execute()`方法解析`WHERE`子句的变量。然后，`execute()`语句执行查询并返回结果。

除了`query()`方法的第二个参数之外，我们还可以使用`QUERY_MODE_EXECUTE`立即执行查询（从而直接返回结果集）或解析带有参数的数组或`ParameterContainer`。有关`ParameterContainer`的更多信息，请参阅以下章节。

如果我们将数组或`ParameterContainer`对象作为`query()`方法的最后一个选项进行解析，这将导致查询参数被填充，并将查询模式设置为`QUERY_MODE_PREPARE`。这意味着因为我们已经将查询参数解析到`query()`方法中，所以我们不需要在`execute()`方法中再次添加它们。

### 使用预处理语句

`query()`方法被描述为一个便利函数，当我们想要保护自己免受 SQL 注入或想要使用具有不同参数的单个查询时，它并不是非常有用。另一方面，`createStatement()`函数提供了一个在安全且负责任的方式下存储和准备 SQL 语句以供使用的好方法。

如示例所示，我们执行了一个类似于`query()`方法的类似语句，然而这种方法比`query()`方法更易于维护和重用。通过使用`ParameterContainer`，我们可以轻松地将变量注入 SQL 中，并简单地因为对象具有容器性质而管理它们。

因为我们使用了`:type`和`:color`，所以语句知道我们的参数数组（`ParameterContainer`实现了`ArrayAccess`类）应该包含`type`和`color`键以匹配 SQL 语句。

### 在我们的 SQL 中引用

通常情况下，只要有数据库访问，就会有用户输入，而我们绝对不应该信任的正是用户输入。尽管大多数人没有黑客攻击您网站的意图，但少数恶意的人会尝试这样做。

Zend Framework 2 提供了一系列引用方法，我们可以使用这些方法来保护自己免受任何伤害。然而，我们应该注意的是，这些只是一小部分工具，您可以在预防灾难性情况时使用，我们建议使用一系列的实用工具来防止 SQL 注入。

### 使用`createStatement`

当我们使用 `createStatement()` 时，结果对象将通过驱动程序实例化，因此 MySQL 的语句工作方式可能与 Oracle 不同（可以，并且我认为会这样）。一旦我们创建了一个语句，它也会自动连接到数据库，这很方便，但我们必须小心，不要在不需要数据库的地方创建语句。如果我们省略了这样的事情，可能会创建不必要的泄漏，尽管可能不是那么大的泄漏，但仍然是一种泄漏。

`query()` 方法直接在连接适配器上工作，尽管使用起来很方便，但在实际生活中并不推荐使用，因为它不促进可重用性（就我个人而言）。如果有疑问，最好总是执行 `createStatement()`，除非我们只是测试一些事情，那么我们可以使用 `query()`。

# 使用 TableGateway 执行查询

在我们了解了如何执行简单查询之后，现在是时候告诉你关于 `TableGateway` 以及它的强大功能了。这个示例完全是关于通过它查询数据库并展示其功能。

## 准备工作

为了充分了解以下食谱，应使用 Zend Framework 2 框架的骨架应用程序，并确保有一个可连接的 MySQL 服务器。别忘了连接到 MySQL 服务器需要在 PHP 中启用 `mysql` 和 `mysqli` 扩展。

## 如何操作...

我们首先要做的是在我们的示例表中插入一条记录。之后，我们将检查它是否成功插入。接下来，我们将使用一些新数据更新记录，如果成功了，我们将再次从表中删除它。

### 插入新记录

在我们开始更新记录之前，如果我们实际上有一个可以用来更新的记录，那将很有帮助。Zend Framework 2 有一些新的巧妙数据库工具，使我们在数据处理方面的生活更加轻松。 

卡片表有以下列：

+   `id` (主键)

+   `color`

+   `value`

+   `type`

让我们考虑以下示例（`/module/Application/src/Application/Controller/IndexController.php`）：

```php
<?php

namespace Application\Controller;

use Zend\Mvc\Controller\AbstractActionController;
use Zend\Db\TableGateway\TableGateway;

class IndexController extends AbstractActionController
{
  public function indexAction()
  {
    // Let's assume there is a service called 'db' that connect to 
    // the database
    $connection = $this->getServiceLocator()->get('db');

    // Let's make this object for examples later on
    // $sql = new Sql($this->connection);

    // Create a new Zend\Db\Sql\Insert object
    // You can also do $sql->insert();
    $insert = new Insert('cards');

    // Define the columns in the table, although not 
    //required, it is best practice
    $insert->columns(array(
      'id',
      'color',
      'type',
      'value',
    ));

    // Assign the values we want to insert, the column 
    // names are in the keys so that the code knows what 
    // to insert where.
    $insert->values(array(
      'color' => 'diamond',
      'type' => 'picture',
      'value' => 'Goblin'
    ));

    // Create a new table gateway to perform our SQL on
    $tableGateway = new TableGateway(
      'cards', $connection
    );

    // We will now use the TableGateway to insert our 
    // statement in the table.
    // The insert() / insertWith() method throws an 
    // exception whenever the query goes wrong. We need to  
    // make sure we catch that.
    try {
      $tableGateway->insertWith($insert);

      // If we reach this point we can assume that the 
      // query went fine.
      echo "Insert success!";
      $hasResult = true;
    } catch (Exception $e) {
      echo "Insert failed.";
    }
  }
}
```

表插入操作到此结束，显然这只是在表中插入数据的一种方式。另一种执行插入语句的方法是使用我们之前创建的 `$sql` 对象。如果我们这样做，我们可以去掉 `TableGateway` 并直接使用它。

如果我们愿意，我们可以这样进行：

```php
// This will prepare a StatementInterface for us to use
$statement = $sql->prepareStatementForSqlObject(
  // Put the insert object in here
  $insert
);

// Now we simply execute the statement to insert the 
// record.
$statement->execute();
```

### 更新记录

我们现在可以继续检查插入操作是否成功，然后我们将使用一些新数据更新记录：

```php
// If an Exception happened, we will have a false in our 
// result.
if (isset($hasResult)) {
  // Let's get the primary key from our last insert for 
  // later use.
  $primaryKey = $tableGateway->getLastInsertValue();

  // Now let's update our record
  // You can also do $sql->update();
  $update = new Update('cards');

  // Set the new values (and column names as keys) for 
  // the data we want to update.
  $update->set(array(
    'color' => 'spade',
    'value' => '10',
    'type' => 'number',
  ));

  // Now create a where statement
  $where = new Where();

  // We want to match our record on the primary key that 
  // we got back from our insertion.
  $where->equalTo("id", $primaryKey);

  // Set the where in the update statement so that we 
  // use that when executing the update. We can add as 
  // many where statements as we like, but we only match 
  // on one here.
  $update->where($where);

  // Now update the record
  $updated = $tableGateway->updateWith($update);
```

更新操作的结果将是受我们的 `update` 语句影响的行数。在我们的例子中，这只会是一条记录，因为我们与表的主键完全匹配。

### 删除记录

现在，我们已经完成了所有想要更新的操作，我们想要再次开始删除这条记录，让我们看看以下代码片段：

```php
// Delete everything again
// You can also do $sql->delete();
$delete = new Delete('cards');

// We can use the same where statement as before!
$delete->where($where);

// Now let's delete it, as there is nothing else to it.
$deleted = $tableGateway->deleteWith($delete);
```

好吧，这很简单。我们可以直接使用相同的 `where` 语句，因为它已经定义了从之前的查询中过滤主键的子句。

### 高级选择 - 连接条件

在开发 Web 应用程序时，我们大多数时候需要查询多个表，这是因为我们只需要从各个地方拉取大量数据以获取所需的结果。实现这一目标的一种方法是在我们的 `select` 语句中使用连接条件。

让我们来看看以下表组成，我们正在我们的虚拟环境中进行：

`people` 表将包含以下列：

+   `Id`（主键）

+   `First_name`

+   `Last_name`

+   `Age`

+   `Gender`

+   `Address_Id`（`addresses` 表的外键）

`addresses` 表将包含以下列：

+   `Id`（主键）

+   `Street`

+   `Number`

+   `Postcode`

+   `City`

+   `Country`

我们想要实现的是检索属于某个人的地址并在我们的结果中显示它。

让我们看看一个例子（`/module/Application/src/Application/Controller/IndexController.php`），看看我们如何以最佳方式实现这一点：

```php
<?php 

namespace Application\Controller;

use Zend\Mvc\Controller\AbstractActionController;
use Zend\Db\TableGateway\TableGateway;

class IndexController extends AbstractActionController
{
  public function indexAction()
  {
    // Let's assume there is a service called 'db' that connect to 
    // the database
    $connection = $this->getServiceLocator()->get('db');

    // First create our Zend\Db\Sql\Sql object, and let's 
    // assume $connection has a Zend\Db\Adapter defined.
    $sql = new Sql($connection);

    // Now create a Zend\Db\Sql\Select statement with 
    // 'people' as the table we want to select from.
    $select = $sql->select('people');

    // By default we will select all the fields, but let's 
    // just change that a bit for sake of the example
    $select->columns(array('first_name', 'last_name'));

    // Now set up our join condition
    $select->join(
      // We want to join the 'addresses' table
      'addresses',

      // We now define the join condition to match the 
      // records on 
      'addresses.id = people.address_id',

      // We want to select different columns than the 
      // default wildcard selection.
      array('street', 'number', 'city', 'postcode'),

      // We want to do a LEFT JOIN on the table
      Select::JOIN_LEFT
    );

    // Now we are ready to execute the statement.
    $statement = $sql->prepareStatementForSqlObject(
      $select
    );

    // .. And finally execute it
    $records = $statement->execute();

    // Output to the screen for convenience
    echo '<pre>'. print_r($records, true). '</pre>';
  }
}
```

在一个 `select` 语句上创建一个 `join` 条件竟然如此简单。易如反掌！

## 它是如何工作的…

在 Zend Framework 2 中，他们将所有操作如`Insert`、`DropTable`、`Update`、`Delete`和`Where`都分离到它们自己的类中，这使得它们对开发者来说非常可重用。它的好处在于它还使代码更加清晰。

`TableGatewayInterface` 定义了由 `AbstractTableGateway` 和 `TableGateway` 实现的最小方法集合，因为 `TableGateway` 首先是从 `AbstractTableGateway` 扩展而来的。简而言之，`TableGateway` 实现了进行表操作所需的大部分常见功能。

因此，`TableGatewayInterface` 定义了以下方法：

+   `getTable()`

+   `select($where = null)`

+   `insert($set)`

+   `update($set, $where = null)`

+   `delete($where)`

# 使用 DB 分析器进行优化

在应用程序中，最常见的瓶颈之一是对数据库的查询，因为有时我们根本不知道查询了多少，或者我们无法找出为什么某些事情出了问题。这个配方为我们提供了找到甚至是最小查询的工具。

## 准备工作

数据库分析器用于查找查询性能的瓶颈，是调试会话中执行的查询以及它们执行所需时间的优秀工具。一旦我们开发了更大的应用程序，我们往往会忘记某些代码何时以及如何执行，这有时会导致我们的代码出现不必要的复杂性。

## 如何做到这一点…

分析应用程序的数据库使用情况可以清楚地了解应用程序的性能，在这个配方中，我们将讨论如何设置一个简单的分析器。

### 设置新的分析器

设置一个新的分析器非常简单，因为目前只有 Zend Framework 2 中的一个类可以用作分析器。这个类被称为`Zend\Db\Adapter\Profiler\Profiler`，可以立即实例化。让我们看一下以下代码片段：

```php
<?php
use Zend\Db\Adapter\Profiler\Profiler;

// Instantiate the Zend\Db\Adapter\Profiler\Profiler
$profiler = new Profiler();  

// Let's assume $connection is an active Db\Adapter,
// we then need to set the profiler to be used by the 
// adapter.
$connection->setProfiler($profiler);
```

就这些；这基本上是开始从数据库中分析一切所需的所有内容。我们剩下要做的只是在我们完成查询（或真正需要时）获取配置文件。让我们考虑以下示例：

```php
<?php
// This will return all the statements that have been 
// executed by the adapter.
$results = $profiler->getProfiles();
```

`$result`变量现在将填充关于执行语句的统计信息。这个结果可能看起来像以下这样：

```php
array(3) {
   [0] => array(5) {
     ["sql"] => string(77) 
        "INSERT INTO `cards` (`color`, `type`, `value`) 
        VALUES (:color, :type, :value)"
     ["parameters"] => object(
  Zend\Db\Adapter\ParameterContainer)#255 (3) {
         ["data":protected] => array(3) {
         ["color"] => string(7) "diamond"
         ["type"] => string(7) "picture"
         ["value"] => string(6) "Goblin"
      }
     ["positions":protected] => array(3) {
       [0] => string(5) "color"
       [1] => string(4) "type"
       [2] => string(5) "value"
      }
       ["errata":protected] => array(0) {
      }
    }
     ["start"] => float(1372316727.1188)
     ["end"] => float(1372316727.1209)
     ["elapse"] => float(0.0020461082458496)
  }
}
```

## 它是如何工作的…

数据库分析器首先被附加到数据库适配器上，使适配器意识到分析器的存在。适配器将在每次执行语句时开始分析（它通过使用`Profiler::profileStart()`方法来完成），确保关于语句的所有重要信息都将被记录。

当数据库适配器完成语句的执行后，它将通知分析器该语句已完成（它将执行`Profiler::profileFinish()`方法）。

如我们从之前的示例中可以看到，我们可以查看执行的 SQL 语句以及使用的参数。之后，还会添加开始时间、结束时间和耗时，这样我们就可以轻松地发现代码中的任何潜在瓶颈。

总的来说，这是一个非常实用的工具，几乎不需要编写任何代码就能工作，并且对于想要找出数据库性能问题的开发者来说仍然非常高效。

## 更多内容…

另一个我们可以查看的出色的小工具是 Zend 开发者工具，这是一个由 Zend 制作的模块，适用于 Zend Framework 2，提供了非常有用的调试工具。如果我们想了解更多，我们可以在[`github.com/zendframework/ZendDeveloperTools`](https://github.com/zendframework/ZendDeveloperTools)找到这些工具。

# 创建数据库访问对象

尽管我们可以使用十几种不同的方法来标准化我们的数据库功能，但数据库访问对象（或 DAO）可以有效地用来实现这一点。这个示例是一个如何制作自己的示例，并开始组织你的功能的实际应用。

## 准备工作

数据库访问对象（以下简称 DAO）用于简化与我们的数据库之间的功能。DAO 背后的想法是创建具有单一功能责任的映射类。这意味着，例如，我们有一个名为`cards`的表，它还有一个名为`Cards`的映射。这个`Cards`映射将包含我们在该表中需要使用的所有功能。

这可能包括，例如，CRUD（创建、读取、更新和删除）功能，也可能包括更复杂的方法，如计算。映射类的理念是我们能够隐藏数据库布局，并为应用程序的其余部分提供一个可靠且一致的接口，而应用程序无需了解数据库的结构。

对于配方，我们将使用具有名为`cards`的表的数据库布局，以下列出了以下列：

+   `id` (主键)

+   `color`

+   `value`

+   `type`

## 如何做…

DAO 是一种在应用程序中组织数据库功能的好方法，这样我们就可以始终有一个清晰的逻辑结构。在这个配方中，我们将展示如何创建自己的 DAO。

### 创建我们的新模块和配置

我们的 DAO 将完全独立于另一个模块，因为这是分离不同代码片段的最佳方式。因此，我们继续创建一个新的模块 DAO，它应该具有以下目录结构：

```php
module\DAO\
  config\
    module.config.php
  src\
    DAO\
  Connection\
    Connector.php
  DTO\
    Cards.php

Mapper\
  Cards.php
  MapperAbstract.php
  MapperInterface.php
  Module.php
```

一旦我们创建了必要的文件夹，我们可以将`Application`模块中的默认`Module.php`复制到我们的`DAO`文件夹中。然后我们打开我们新的`Module.php`，确保`namespace`设置为`DAO`。

现在，是时候创建一个新的`/module/DAO/config/module.config.php`文件，并添加以下行：

```php
<?php

return array(
  // This is going to be the configuration from which we 
  // will read. Obviously the username/password should 
  // be in the local.php but we will just put it here 
  // example wise.
  'dao' => array(
    'hostname' => 'localhost',
    'username' => 'some_user',
    'password' => 'some_password',
    'database' => 'book',

    // This mapper will contain all of our mapper 
    // classes such as DAO\Db\Mapper\Cards and let them 
    // know which table they need to connect to.
    'mapper' => array(
      'Cards' => 'cards',
    ),
  ),

  // Initialize our service manager so that we can reach 
  // our mappers from anywhere else in the application 
  // (every mapper should have its own entry) and our 
  // connector which should be reached only by the 
  // mappers and not anywhere else
  'service_manager' => array(
    'invokables' => array(
      'DAO_Connector' =>'DAO\Db\Connection\Connector',
      'DAO_Mapper_Cards' =>'DAO\Db\Mapper\Cards',
    ),
  ),
);
```

这个相当基本的配置将被我们的数据库连接器稍后用于获取连接详情。

### 创建一个连接器

接下来，我们想要创建我们的连接器，它基本上是一个类，将创建数据库适配器并为我们设置一切。它不会做其他任何事情，所以我们应该能够轻松地编写一个。

让我们现在创建一个名为`/module/DAO/src/DAO/db/Connection/Connector.php`的文件，在`DAO\Db\Connection`命名空间中，并添加以下代码：

```php
<?php

// Set the correct namespace
namespace DAO\Db\Connection;

// We will be using the following classes
use Zend\ServiceManager\ServiceLocatorAwareInterface;
use Zend\ServiceManager\ServiceLocatorInterface;
use Zend\Db\Adapter\Adapter;

// We are going to make this as a Service, so make sure 
// we implement the ServiceLocatorAwareInterface
class Connector implements ServiceLocatorAwareInterface 
{
  // Our service locator will be placed in here
  protected $serviceLocator;

  // Now set our service manager instance required by the 
  // ServiceLocatorAwareInterface
  public function setServiceLocator(ServiceLocatorInterface $serviceLocator)
  {
    $this->serviceLocator = $serviceLocator;
  }

  // And add our getter for the service manager, as is required by 
  // the ServiceLocatorAwareInterface
  public function getServiceLocator()
  {
    return $this->serviceLocator;
  }

  /**
   * Initializes a connection and returns a fresh 
   * adapter.
   *
   * @return \Zend\Db\Adapter\Adapter
   * @throws \Exception
   */
  public function initialize()
  {
    // Get the configuration from the module.config.php
    $dao = $this->getServiceLocator()->get('config');

    // The following array of configuration items should 
    // be in there
    $configItems = array(
      'hostname', 
      'username', 
      'database', 
      'password'
    );

    // Check if everything is there in the configuration
    foreach ($configItems as $required) {
      if (!in_array($required, array_keys($dao['dao']))) 
      {
        // If there is a config item missing, just let
        // the develop know
        throw new \Exception("{$required} is not in the DAO configuration!");
      }
    }

    // We can assume we have everything, now set up our 
    // MySQL connection
    return new Adapter(array(
      'driver' => 'Pdo_Mysql',
      'database' => $dao['dao']['database'],
      'hostname' => $dao['dao']['hostname'],
      'username' => $dao['dao']['username'],
      'password' => $dao['dao']['password'],
    ));
  }
}
```

那就是类的定义；我们现在能够初始化连接，如果我们配置中有正确的项目。如果没有，该方法将抛出异常并让我们知道。

### 创建一个映射器接口

现在，我们想要创建一个映射器接口，我们将基于它创建所有未来的映射器类。我们这样做是因为我们想要确保所有映射器类都包含我们想要的至少一些方法。因此，我们的映射器接口将定义我们希望映射器类拥有的方法的小集合。

现在，让我们在`DAO\Db\Mapper`命名空间中创建一个名为`/module/DAO/src/DAO/Db/Mapper/MapperInterface.php`的文件，并添加以下代码：

```php
<?php

// Make sure we have the namespace right
namespace DAO\Db\Mapper;

// Note that this is an interface, and not a regular 
// class.
interface MapperInterface
{
  // We need an insert method in our mapper.  
  public function insert($data);

  // And obviously we want to update data
  public function update($data);

  // If we want to update, we also want to delete data
  public function delete($id);

  // And of course we want to load one specific record
  public function load($id);

  // Last but not least we also want a method to get all 
  // the records in the table
  public function getAll();
}
```

如我们所见，这是一个相当直接的文件，因为接口实际上并没有对代码进行任何实现。

### 创建一个抽象映射器类

虽然接口没有实现任何代码，但抽象类可以。我们想在同一个`DAO\Db\Mapper`命名空间中创建一个名为`/module/DAO/src/DAO/Db/Mapper/MapperAbstract.php`的文件，它将包含一个创建数据库连接、指向正确的表并返回一个新鲜出炉的`Zend\Db\Sql\Sql`对象的方法：

```php
<?php

// Namespace, do I need to say more ;-)
namespace DAO\Db\Mapper;

// Use the following classes
use Zend\ServiceManager\ServiceLocatorAwareInterface;
use Zend\ServiceManager\ServiceLocatorInterface;
use Zend\Db\Sql\Sql;

// Note that we are again using the 
// ServiceLocatorAwareInterface and therefore need to 
// implement the getServiceLocator and setServiceLocator 
// (not shown here).
class MapperAbstract implements ServiceLocatorAwareInterface 
{
  // Our sql object will be put here
  private $sqlObject;

  // We'll just put our service locator in here	
  protected $serviceLocator;
```

所有的设置都完成了，现在让我们创建我们需要的连接方法（别忘了创建`setServiceLocator`和`getServiceLocator`方法！）：

```php
// This method will set up our connection, initialize 
// the right table and return a Sql object
protected function getSqlObject()
{
  // We only want to set up our connection once, no 
  // point in doing it more, right?

  if ($this->connection === null) {
    // Get our configuration from the 
    // module.config.php
    $config = $this->getServiceLocator()->get('config');

  // Get our class name
  $class = explode('\\', get_class($this));

  // Now check if our class name is defined in the 
  // mapper configuration of the dao configuration, 
  // so that we can get our table name. Looks more 
  // complicated than it is really.
  if (isset($config['dao']['mapper']) === true && isset($config['dao']['mapper'][end($class)])) {

    // Get the database adapter from our connector
    $adapter = $this->getServiceLocator()
                    ->get('DAO_Connector')
                    ->initialize();

    // We have a configuration, now return our SQL 
    // object with the right table name included
    $this->sqlObject = new Sql(
      $adapter, 
      $config['dao']['mapper'][end($class)]
    );
      } else {
        // Make sure the developer knows not all the 
        // configuration is set.
        throw new \Exception("Configuration dao\mapper\\". end($class). " not set.");
   }
    } 

    // Now return our sql object
    return $this->sqlObject;
  }
}
```

我们新创建的连接方法现在可以被映射器用来获取一个与它们想要操作的表相关的`Zend\Db\Sql\Sql`对象。

### 创建数据传输对象

现在，让我们创建一个新的**数据传输对象**（**DTO**）文件，名为`/module/DAO/src/DAO/Db/DTO/Cards.php`，在`DAO\Db\DTO`命名空间中，并添加以下代码：

```php
<?php

// Namespace, quite essential
namespace DAO\Db\DTO;

// We should name our class simply Cards, as that is 
// used in the mapper later on as well
class Cards
{
  // Our 'cards' table exists of an id column, color, 
  // type and value, let's just define them as private 
  // properties.
  private $id;
  private $color;
  private $type;
  private $value;
```

现在我们已经设置了私有属性，我们还将为它们创建一些基本的获取器和设置器。以下是为获取器使用的代码：

```php
public function getId() { return $this->id; }
public function getColor() { return $this->color; }
public function getType() { return $this->type; }
public function getValue() { return $this->value; }
```

获取器现在已经完成，这相当简单，现在让我们来做设置器：

```php
// The id will only be set if we update a record, or 
// when we retrieve a record from a database. Never 
// when we want to insert a record.
public function setId($id) {
  $this->id = $id;
}

// Make sure we can only use colors that are valid in 
// our table.
public function setColor($color) 
{
  $validColors = array('diamond', 'spade', 'heart', 'club');

  if (in_array($color,$validColors)== false) {
    throw new \Exception(
      "Type can only be 'diamond', 'spade', 'heart'".   
      "or 'club'."
    );
  }

  $this->color = $color;
}

// Make sure only a valid type is entered.
public function setType($type) 
{
  $validTypes = array('number', 'picture');

  if (!in_array($type, $validTypes)) {
    throw new \Exception(
      "Type can only be 'number' or 'picture'."
    );
  }

  $this->type = $type;
}

// A value can only have a maximum of 6 character
public function setValue($value) 
{
  $maxValue = 6;

  if (strlen($value) >$maxValue) {
    throw new \Exception(
      "Maximum length of value is 6."
    );
  }

  $this->value = $value;
}
```

设置器显然要复杂一些，因为我们还想要确保我们放入的数据对我们的数据库是有效的。这样我们就可以安全地将对象解析到映射器中，并确保一切顺利。

现在，创建最后一个方法，即构造函数，这样我们就可以轻松设置属性，而无需在之后手动进行：

```php
  public function __construct($type, $value, $color, $id = null) 
  {
    // Id is optional, so see if it is parsed or not
    if ($id !== null) $this->setId($id);

    $this->setColor($color);
    $this->setType($type);
    $this->setValue($value);
  }
}
```

我们现在创建了一个简单的 DTO，我们可以用它来与映射器中的某些方法进行通信。现在，最后但同样重要的是让我们创建映射器类！

### 创建映射器类

映射器将是我们在应用程序中使用的主体 DAO 类，因为这将是一个具有`insert`、`getAll`等方法类的类。

让我们从创建一个位于`DAO\Db\Mapper`命名空间中的`/module/DAO/src/DAO/Db/Mapper/Cards.php`文件开始，并添加以下代码：

```php
<?php

namespace DAO\Db\Mapper;

use Zend\Db\Sql\Where;
use DAO\Db\DTO\Cards as CardsDto;
use DAO\Db\Mapper\MapperInterface;

// This class will extend and implement both our 
// Abstract as our Interface class
class Cards extends MapperAbstract implements MapperInterface
{
```

让我们先创建一个用于删除行的方法：

```php
/**
 * Delete a specific row.
 * 
 * @param int $id
 */
public function delete($id) 
{
  // Get our fresh Sql object from our Abstract method
  $sql = $this->getSqlObject();

  // Create a new WHERE clause
  $where = new Where();

  // When deleting we want to match on an id
  $where->equalTo('id', $id);

  // Statements can throw exceptions, so make sure we 
  //catch them in time.
  try {
    // Create a new delete object with our where class    
    // attached and then immediately turn it into a 
    // statement. That is called pure laziness
    $statement = $sql->prepareStatementForSqlObject(               
      $sql->delete()->where($where)
    );

    // Execute the statement
    $result = $statement->execute();

    // If there is more than 0 rows deleted return 
    // true, otherwise false
    return $result->getAffectedRows() > 0;
  } catch (\Exception $e) {
    // Something went terribly wrong, just ignore it 
    // for now ;-)
    // TIP: Don't do this in real life, at least log your 
    //exceptions.
    return false;
  }
}
```

我们已经创建了一个简单的`delete`方法，现在让我们继续创建我们的`getAll`方法，它将检索数据库中的所有记录：

```php
/**
 * Returns all the records in the database.
 * 
 * @return \DAO\Db\DTO\Cards
 */
public function getAll()
{
  // Get the SQL object
  $sql = $this->getSqlObject();

  // Prepare a select statement
  $statement = $sql->prepareStatementForSqlObject(
    $sql->select()
  );

  // Execute the freshly made statement
  $records = $statement->execute();

  // Create our return array
  $retval = array();

  // Loop through the records and add them to the 
  // result array
  foreach ($records as $row) {
    // Create a new Cards DTO and assign our record
    $retval[] = new CardsDto(
      $row['type'], 
      $row['value'], 
      $row['color'], 
      $row['id']
    );
  }

  return $retval;
}
```

在我们创建了`getAll`，它返回包含 Cards DTO 的数组之后，我们现在将创建一个插入记录的方法：

```php
/**
 * Inserts a record.
 * 
 * @param \DAO\Db\DTO\Cards $data
 */
public function insert($data)
{
  // We can easily insert this as we know the DTO has 
  // already taken care of the validation of the values.
  if (!$data instanceof DAO\Db\DTO\Cards) {
    throw new \Exception(
      "Data needs to be of type DAO\Db\DTO\Cards"
    );
  }

  // Get our SQL object
  $sql = $this->getSqlObject();

  try {
    // Create our insert statement with the values 
    // assigned into it.
    $statement = $sql->prepareStatementForSqlObject(
      $sql->insert()
          ->values(array(
            'color' => $data->getColor(),
            'type' => $data->getType(),
            'value' => $data->getValue()
      ))
  );

    // Execute our statement
    $result = $statement->execute();

    // Return our primary key after insertion
    return $result->getGeneratedValue();
  } catch (\Exception $e) { 
    // Something went wrong, handle exception and 
    // return false
    return false;
    }
  }
```

现在，让我们继续到我们的`load`方法，它将只返回一条记录：

```php
public function load($id) 
{
  // Get the SQL object
  $sql = $this->connection();

  // A fresh WHERE clause
  $where = new Where();
  $where->equalTo('id', $id);

  try {
    // Prepare a select statement with the where 
    // clause attached.
    $statement = $sql->prepareStatementForSqlObject(
      $sql->select()->where($where)
    );

    // Execute the statement and return the first row
    $record = $statement->execute()->current();

    // Now let's return a fresh Cards DTO object
    return new CardsDto(
      $record['type'], 
      $record['value'], 
      $record['color'], 
      $record['id']
    );
  } catch (\Exception $e) {
 return false;
  }
}
```

我们现在创建了`load`方法，它将为我们返回一个 Cards DTO 对象以供使用，现在最后但同样重要的是`update`方法：

```php
  public function update($data) 
  {
    // We can easily insert this as we know the DTO has 
    // already taken care of the validation of the   
    // values.
    if (get_class($data) !== 'DAO\Db\DTO\Cards') {
      throw new \Exception(
        "Data needs to be of type DAO\Db\DTO\Cards"
      );
    }

    if ($data->getId() === null) {
      throw new \Exception(
           "Can't update anything if we don't have a card id!"
      );
    }

    // Get the connection
    $sql = $this->connection();

    try {
      // Create the WHERE clause
      $where = new Where();
      $where->equalTo('id', $data->getId());

      // Create the update class
      $update = $sql->update();

      // Set the where clause
      $update->where($where);
      $update->set(array(
        'color' => $data->getColor(),
        'type' => $data->getType(),
        'value' => $data->getValue()
      ));

      // Create the statement
      $statement = $sql->prepareStatementForSqlObject($update);

      // Execute the statement
      $result = $statement->execute();

      // If more than 0 rows were updated return true, 
      // otherwise false
      return $result->getAffectedRows() > 0;
    } catch (\Exception $e) { 
      return false;
    }
  }
}
```

我们现在已经成功创建了一个映射器类，这也标志着我们的 DAO 的结束。现在我们可以通过服务管理器（例如，在控制器`/module/Cards/src/Cards/Controller/CardController.php`）轻松获取映射器，使用以下代码：

```php
<?php

namespace Cards\Controller;

use Zend\Mvc\Controller\AbstractActionController;

class CardsController extends AbstractActionController
{
  public function viewAction()
  {
    if (!$this->getParam('id'))
        throw new \Exception("Missing id");

    // Get the record to load from the query string
    $id = $this->params()->fromQuery('id');

    // Get the card mapper from the service manager
    $cardMapper = $this->getServiceLocator()
                       ->get('DAO_Mapper_Cards');

    // Load the requested card
    $card = $cardMapper->load($id);

    // Dump the loaded record to the screen
    echo '<pre>'. Print_r($card, true). '</pre>';
  }
}
```

由于我们创建了一个抽象类和接口，因此我们很容易创建新的映射器。显然，这需要我们保持一致性，但这是一件好事。

## 它是如何工作的…

### 关于 DAO

DAO（数据库访问对象）是一种设计模式，它为开发者创建了一个抽象的环境，以便他们可以访问数据库相关的方法。这意味着我们创建了一个标准化的工作环境，它不仅一致，而且非常稳定。因为，我们在处理数据库查询和创建的对象的方式上限制了自己，所以我们创建了一段非常容易工作的代码。

在这个食谱中，我们创建了一个非常简单的 DAO，这在个人看来是一个很好的基础，但可能不是创建 DAO 最有效的方法。我们只是举了一个例子，说明 DAO 可以如何实现，但我们绝不应该忽视实际上有数十种不同的实现方式。

### 关于食谱

由于我们的配置包含一个包含所有映射类名称的映射器数组（`DAO\Db\Mapper\Cards`在配置中简化为 cards），我们不可能出错。这将数据库环境的本地配置与代码分离。因此，如果我们想将表名更改为'books'，我们只需更改配置，代码仍然可以工作！

我们将创建一个 DTO（数据传输对象），这样我们就可以通过标准化的方式轻松地插入、更新并返回记录。因此，在我们的选择中，我们不再返回一个数组，而是返回一个包含我们所需一切内容的对象。这样我们确保我们的数据被过滤并且可以简单地传输。

正如我们在 Mapper 类中的`insert`方法中看到的，我们假设 DTO 对象包含我们插入记录所需的所有正确信息。尽管这个方法远非完美，但它是一种将我们的数据检查和验证分离到另一个对象（在我们的情况下是 DTO）的好方法，这样我们就可以专注于插入记录。这种分离对于良好的 DAO 工作至关重要。
