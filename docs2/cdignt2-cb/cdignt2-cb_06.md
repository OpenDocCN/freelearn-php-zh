# 第六章. 与数据库一起工作

在本章中，我们将涵盖：

+   配置 CodeIgniter 数据库

+   连接到多个数据库

+   Active Record – 创建（插入）

+   Active Record – 读取（选择）

+   Active Record – 更新

+   Active Record – 删除

+   遍历数据库结果

+   使用 num_rows() 计算返回结果的数量

+   使用 count_all_results() 计算返回结果的数量

+   计算返回结果的数量

+   查询绑定

+   查找最后插入的 ID

+   查找受影响行数

+   查找最后的数据库查询

+   使用 CodeIgniter 数据库迁移

+   使用 current() 移动到当前版本

+   使用 version() 回滚/前进

+   从数据库结果生成 XML

+   从数据库结果生成 CSV

# 简介

几乎您构建的任何应用程序都需要数据库访问和功能，从基本的 **创建**、**读取**、**更新** 和 **删除** （**CRUD**）操作到更复杂的方法。在本章中，我们将查看一些相当简单的食谱（例如，简单的 CRUD 操作），然后是一些更强大的食谱，例如连接到多个数据库、数据库缓存和生成输出文件。

一些食谱相当简单，所以我不提供所有文件供您复制（在某些情况下，这可能是不必要的）；相反，许多食谱都是小块代码，您可以在需要时将其放入实际场景中。

# 配置 CodeIgniter 数据库

如果您已经配置了 CodeIgniter 以与数据库连接，您可以跳过这部分，因为我们将要做的只是确保我们可以连接到数据库；为此，我们将修改以下两个文件：

+   `/path/to/codeigniter/application/config/database.php`

+   `/path/to/codeigniter/application/config/autoload.php`

## 如何做到这一点...

1.  在 `database.php` 配置文件中，查找以下行并相应修改：

    ```php
    $db['default']['hostname'] = 'localhost'; 
    $db['default']['username'] = 'Replace with database username'; 
    $db['default']['password'] = 'Replace with database password'; 
    $db['default']['database'] = 'Replace with database name'; 
    ```

    很可能您不需要更改 `$db['default']['hostname']` 从 `'localhost'`，并将其他值（`username`、`password` 和 `database`）替换为您的环境中的特定值。

1.  在 `autoload.php` 配置文件中，查找以下行（大约在第 55 行）：

    ```php
    $autoload['libraries'] = array(); 
    ```

1.  确保数据库正在通过将其添加到 `$autoload` 数组中自动加载，如下所示：

    ```php
    $autoload['libraries'] = array('database'); 
    ```

    ### 小贴士

    确保用逗号分隔您正在自动加载的每个库，例如，`$autoload['libraries'] = array('database', 'session', 'javascript')` 等等。

## 它是如何工作的...

实际上并没有太多；这只是设置配置设置，但一个有趣的观点是自动加载库。通过在自动加载配置文件中将库名放入此数组中，您就不需要在应用程序中的控制器中显式加载库了。

# 连接到多个数据库

有时候你可能需要你的应用程序连接到多个数据库或数据库服务器。例如，想象你管理一个在线商店，你可能希望有一个数据库来处理客户订单、账单、发票等，而另一个数据库用于存储和维护产品库存信息。CodeIgniter 可以配置为使用多个数据库实例，以下部分展示了如何操作。

## 准备工作

为了让 CodeIgniter 与两个或多个数据库交互，我们需要在以下 `config` 文件中修改一些设置：

+   `/path/to/codeigniter/application/config/database.php`

滚动到文件底部并将以下内容复制进去。请记住，用你的设置中的正确细节替换 `hostname`、`username`、`password` 和 `database`。

```php
$db['database1']['hostname'] = ''; 
$db['database1']['username'] = ''; 
$db['database1']['password'] = ''; 
$db['database1']['database'] = 'database1'; 
$db['database1']['dbdriver'] = 'mysql'; 
$db['database1']['dbprefix'] = ''; 
$db['database1']['pconnect'] = FALSE; 
$db['database1']['db_debug'] = FALSE; 
$db['database1']['cache_on'] = FALSE; 
$db['database1']['cachedir'] = ''; 
$db['database1']['char_set'] = 'utf8'; 
$db['database1']['dbcollat'] = 'utf8_general_ci'; 
$db['database1']['swap_pre'] = ''; 
$db['database1']['autoinit'] = TRUE; 
$db['database1']['stricton'] = FALSE; 

$db['database2']['hostname'] = ''; 
$db['database2']['username'] = ''; 
$db['database2']['password'] = ''; 
$db['database2']['database'] = 'database2'; 
$db['database2']['dbdriver'] = 'mysql'; 
$db['database2']['dbprefix'] = ''; 
$db['database2']['pconnect'] = FALSE; 
$db['database2']['db_debug'] = FALSE; 
$db['database2']['cache_on'] = FALSE; 
$db['database2']['cachedir'] = ''; 
$db['database2']['char_set'] = 'utf8'; 
$db['database2']['dbcollat'] = 'utf8_general_ci'; 
$db['database2']['swap_pre'] = ''; 
$db['database2']['autoinit'] = TRUE; 
$db['database2']['stricton'] = FALSE;
```

仔细查看粗体行。每个数据库组的前四行详细说明了你希望使用的每个数据库的标准主机、用户名、密码和数据库名。但，也要看看以下行：

```php
$db['database1']['pconnect'] = FALSE; 
$db['database2']['pconnect'] = FALSE; 

```

数据库配置设置 `'pconnect'` 告诉 CodeIgniter 你是否希望有持久连接。将此值设置为 `False` 在每个数据库组中允许 CodeIgniter 与多个数据库通信。我们将创建两个数据库，每个数据库中有一个表。显然，你的需求可能不同，但你可以根据需要调整配方。将以下代码复制到你的数据库中：

```php
CREATE DATABASE  `database1` ;
USE `database1`;

CREATE TABLE `table1` (
  `t1_id` int(11) NOT NULL AUTO_INCREMENT,
  `t1_first_name` varchar(255) NOT NULL,
  `t1_last_name` varchar(255) NOT NULL,
  PRIMARY KEY (`t1_id`)
) ENGINE=InnoDB  DEFAULT CHARSET=latin1 AUTO_INCREMENT=3 ;

INSERT INTO `table1` (`t1_id`, `t1_first_name`, `t1_last_name`) VALUES
(1, 'Lucy', 'Welsh'),
(2, 'Rob', 'Foster');

CREATE DATABASE  `database2` ;
USE `database2`;

CREATE TABLE `table2` (
  `t2_id` int(11) NOT NULL AUTO_INCREMENT,
  `t2_first_name` varchar(255) NOT NULL,
  `t2_last_name` varchar(255) NOT NULL,
  PRIMARY KEY (`t2_id`)
) ENGINE=InnoDB  DEFAULT CHARSET=latin1 AUTO_INCREMENT=3 ;

INSERT INTO `table2` (`t2_id`, `t2_first_name`, `t2_last_name`) VALUES
(1, 'Oliver', 'Welsh'),
(2, 'Chloe', 'Graves');
```

## 如何操作...

现在，我们有两个数据库可以工作，并且已经配置了 `database.php` 配置文件以与它们通信，因此现在我们可以依次访问每个数据库。

我们将创建控制器文件 `'database1'` 和 `'database2'`，它们将位于：

+   `/path/to/codeigniter/application/controllers/multi_database.php`: 这是一个控制器文件；它将调用两个数据库 `'database1'` 和 `'database2'` 的模型。

+   `/path/to/codeigniter/application/models/multi_database_model_db_1.php`: 此模型将与第一个数据库 `'database1'` 通信。

+   `/path/to/codeigniter/application/models/multi_database_model_db_2.php`: 此模型将与第二个数据库 `'database2'` 通信。

以下步骤将帮助我们访问每个数据库：

1.  确保 `/config/database.php` 中的 `$db['users']['pconnect']` 设置为 `FALSE`，并且你已经为每个数据库输入了正确的访问信息。

1.  创建文件 `/path/to/codeigniter/application/controllers/multi_database.php`。此控制器将调用两个数据库模型并输出每个模型的结果。将以下代码添加到控制器文件 `multi_database.php` 中：

    ```php
    <?php if (!defined('BASEPATH')) exit('No direct script access allowed');

    class Multi_database extends CI_Controller {

      function __construct() {
        parent::__construct();
        $this->load->helper('url'); 
      }

      public function index() {
        redirect('multi_database/select');
      }

      public function select() {   
        $this->load->model('Multi_database_model_db_1');
        $query1 = $this->Multi_database_model_db_1->select_1();

        $this->load->model('Multi_database_model_db_2');
        $query2 = $this->Multi_database_model_db_2->select_2();

        foreach ($query1->result() as $row1) {
          echo $row1->t1_first_name . ' ' . $row1->t1_last_name;
          echo '<br />';
        }

        foreach ($query2->result() as $row2) {
          echo $row2->t2_first_name . ' ' . $row2->t2_last_name;
          echo '<br />';      
        }
      }
    }
    ```

1.  创建模型 `/path/to/codeigniter/application/models/multi_database_model_db_1php`。此模型将与 `'database1'` 通信。将以下代码添加到模型中：

    ```php
    <?php if ( ! defined('BASEPATH')) exit('No direct script access allowed');
    class Multi_database_model_db_1 extends CI_Model {
      function __construct() {
        parent::__construct();
      }

      function select_1() {
        $DBconn1 = $this->load->database('database1', TRUE);	    
        $query1 = $DBconn1->query("SELECT * FROM `table1`");
        return $query1;
      }
    }
    ```

1.  创建模型 `/path/to/codeigniter/application/models/multi_database_model_db_2.php`。此模型将与 `'database2'` 进行通信。将以下代码添加到模型中：

    ```php
    <?php if ( ! defined('BASEPATH')) exit('No direct script access allowed');

    class Multi_database_model_db_2 extends CI_Model {
      function __construct() {
        parent::__construct();
      }

      function select_2() {
        $DBconn2 = $this->load->database('database2', TRUE);
        $query2 = $DBconn2->query("SELECT * FROM `table2`");
        return $query2;
      }
    }
    ```

## 它是如何工作的...

首先，让我们关注在文件 `/path/to/codeigniter/application/config/database.php` 中为我们的每个数据库定义的设置。这些数据库设置是针对我们想要连接的每个数据库特定的。我们还为我们的每个数据库设置了配置变量 `'pconnect'` 为 `FALSE`（见前面的粗体文本）。当我们通过浏览器运行 `Multi_database` 控制器时，控制器将加载我们的两个数据库模型，为了便于解释，命名为 `'Multi_database_model_db_1'` 和 `'Multi_database_model_db_2'`。`Multi_database` 控制器将随后调用每个模型中的一个函数，再次为了便于解释，命名为 `select_1` 和 `select_2`。以下代码显示了相同的内容：

```php
    $this->load->model('Multi_database_model_db_1');
    $query1 = $this->Multi_database_model_db_1->select_1();

    $this->load->model('Multi_database_model_db_2');
    $query2 = $this->Multi_database_model_db_2->select_2();
```

好的！到目前为止一切顺利。这里没有什么新内容——只是调用一些数据库模型；然而，有趣的事情发生在这些模型内部。让我们看看模型 `Multi_database_model_db_1` 的代码：

```php
    function select_1() {
      $DBconn1 = $this->load->database('database1', TRUE);      
      $query1 = $DBconn1->query("SELECT * FROM `table1`");
      return $query1;
    }
```

我们正在加载数据库 `'database1'`——这意味着，我们正在使用在 `database.php` 配置文件中为 `'database1'` 定义的设置来连接一个名为 `'database1'` 的数据库，并将其存储在名为 `$DBconn1` 的对象中：

```php
      $DBconn1 = $this->load->database('database1', TRUE);
```

接下来，我们使用数据库对象 `$DBconn1` 来运行一个查询，并将数据库结果对象存储在变量 `$query1` 中：

```php
      $query1 = $DBconn1->query("SELECT * FROM `table1`");
```

然后，我们将 `$query` 返回给调用控制器。`Multi_database` 控制器随后遍历 `$query1` 结果对象，边走边输出：

```php
    foreach ($query1->result() as $row1) {
      echo $row1->t1_first_name . ' ' . $row1->t1_last_name;
      echo '<br />';
    }
```

# 活动记录 – 创建（插入）

使用 CodeIgniter 活动记录将数据插入数据库有几种方法；例如，`$this->db->insert()` 和 `$this->db->insert_batch()`。第一个一次只插入一条记录，而第二个将数据数组作为单独的行插入数据库；如果你知道需要一次插入多条记录，这可以非常有用，从而避免多次调用 `insert()`。

## 准备工作

这是支持此菜谱所需的 SQL 代码；你需要根据你的情况对其进行调整。将以下 SQL 代码复制到你的数据库中：

```php
CREATE TABLE IF NOT EXISTS `ch6_users` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `firstname` varchar(50) NOT NULL,
  `lastname` varchar(50) NOT NULL,
  `username` varchar(20) NOT NULL,
  `password` varchar(20) NOT NULL,
  `created_date` int(11) NOT NULL,
  `is_active` varchar(3) NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB  DEFAULT CHARSET=latin1 AUTO_INCREMENT=1 ;
```

## 如何操作...

我们将创建以下两个文件（或者如果你已经创建了这些文件，则修改它们）：

+   `/path/to/codeigniter/application/controllers/database.php`

+   `/path/to/codeigniter/application/models/database_model.php`

以下步骤将演示如何使用 CodeIgniter 活动记录将数据插入数据库：

1.  将以下代码添加到控制器 `database.php` 中：

    ```php
    <?php if (!defined('BASEPATH')) exit('No direct script access allowed'); 

    class Database extends CI_Controller { 

      function __construct() { 
        parent::__construct();
      }

      public function index() { 
        redirect('database/create');
      } 

      public function create() { 
        $data = array( 
          'firstname' => 'Lucy', 
          'lastname' => 'Welsh', 
          'username' => 'lucywelsh', 
          'password' => 'password', 
          'created_date' => time(), 
          'is_active' => 'yes' 
        ); 

        $this->load->model('Database_model');
        if ($this->Database_model->insert_data($data) ) { 
          echo 'Success';  
        } 
        else 
        {
          echo 'Cannot insert to database';
        }     

      }

      public function create_batch() { 
        $data = array( 
          array( 
            'firstname' => 'Lucy', 
            'lastname' => 'Welsh', 
            'username' => 'lwelsh', 
            'password' => 'password', 
            'created_date' => time(), 
            'is_active' => 'yes'), 
          array( 
            'firstname' => 'claire', 
            'lastname' => 'Strickland', 
            'username' => 'cstrickland', 
            'password' => 'password', 
            'created_date' => time(), 
            'is_active' => 'yes'), 
           array( 
            'firstname' => 'Douglas', 
            'lastname' => 'Morrisson', 
            'username' => 'dmorrisson', 
            'password' => 'password', 
            'created_date' => time(), 
            'is_active' => 'yes') 
          ); 

        $this->load->model('Database_model'); 
        if ($this->Database_model->insert_batch_data($data)) { 
          echo 'Success';  
        } 
        else
        {
          echo 'Cannot insert to database';
        }     
      }
    }
    ```

1.  将以下代码添加到模型 `database_model.php` 中：

    ```php
    <?php if ( ! defined('BASEPATH')) exit('No direct script access allowed'); 
    class Database_model extends CI_Model { 

      function __construct() { 
        parent::__construct(); 
      } 

      function insert_data($data) { 
        $this->db->insert('ch6_users', $data);
      } 

      function insert_batch_data($data) { 
        $this->db->insert_batch('ch6_users',$data);
      } 
    } 
    ```

## 它是如何工作的...

这里使用了两种方法：`create()` 和 `create_batch()`。让我们逐一查看每个函数的工作原理。

### 公共函数 create()

`create()` 函数应该相当熟悉；我们正在创建一个数组（命名为 `$data`）并用一个用户的数据或相当于一行插入的数据填充它。然后 `create()` 方法将 `$data` 数组传递给模型函数 `insert_data()`，如下所示：

```php
    $this->load->model('Database_model'); 
    $this->Database_model->insert_data($data); 
```

模型将随后向表 `ch6_users` 插入一行：

```php
    function insert_data($data) { 
      $this->db->insert('ch6_users', $data); 
    } 
```

### 公共函数 `create_batch()`

与前面的 `create()` 功能类似的 `create_batch()` 公共函数，但不是传递包含一组项目的数组，而是创建一个包含多行数据的二维数组，如下所示：

```php
    $data = array( 
      array( 
        'firstname' => 'Lucy', 
        'lastname' => 'Welsh', 
        'username' => 'lwelsh', 
        'password' => 'password', 
        'created_date' => time(), 
        'is_active' => 'yes'), 
      array( 
        'firstname' => 'claire', 
        'lastname' => 'Strickland', 
        'username' => 'cstrickland', 
        'password' => 'password', 
        'created_date' => time(), 
        'is_active' => 'yes'), 
       array( 
        'firstname' => 'Douglas', 
        'lastname' => 'Morrisson', 
        'username' => 'dmorrisson', 
        'password' => 'password', 
        'created_date' => time(), 
        'is_active' => 'yes') 
    ); 
```

然后，我们将该数组发送到新的模型函数 `create_batch()`：

```php
    function insert_batch_data($data) { 
      $this->db->insert_batch('ch6_users',$data); 
    } 
```

`function create_batch()` 函数使用 CodeIgniter 的 `function insert_batch()` 将每一行插入到数据库中。

# Active Record – 读取（选择）

CRUD 中的 R 代表从数据库中选择数据的过程。CodeIgniter 使用 `$this->db->get()` 数据库函数从数据库中检索行。其用法将在以下章节中解释。

## 准备工作

以下是需要支持此菜谱的 SQL 代码；您需要根据您的具体情况对其进行调整。

```php
CREATE TABLE IF NOT EXISTS `ch6_users` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `firstname` varchar(50) NOT NULL,
  `lastname` varchar(50) NOT NULL,
  `username` varchar(20) NOT NULL,
  `password` varchar(20) NOT NULL,
  `created_date` int(11) NOT NULL,
  `is_active` varchar(3) NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB  DEFAULT CHARSET=latin1 AUTO_INCREMENT=1 ;

INSERT INTO `ch6_users` (`firstname`, `lastname`, `username`, `password`, `created_date`, `is_active`) VALUES
('claire', 'Strickland', 'cstrickland', 'password', 1366114115, 'yes'),
('Douglas', 'Morrisson', 'dmorrisson', 'password', 1366114115, 'yes'),
('Jessica', 'Welsh', 'jesswelsh', 'password', 1366114115, 'yes');
```

## 如何操作...

我们将创建以下两个文件（或者如果您已经创建了这些文件，则修改它们）：

+   `/path/to/codeigniter/application/controllers/database.php`

+   `/path/to/codeigniter/application/models/database_model.php`

以下步骤将演示如何使用 CodeIgniter Active Record 将数据读取到数据库中：

1.  将以下代码添加到文件 `/path/to/codeigniter/application/controllers/database.php` 中：

    ```php
      public function select_row() { 
        $id = 1; 
        $this->load->model('Database_model'); 
        $result = $this->Database_model->select_row($id);  
        echo '<pre>'; 

        var_dump($result->result()); 
      } 
    ```

1.  将以下代码添加到文件 `/path/to/codeigniter/application/models/database_model.php` 中：

    ```php
    function select_row($id) {	 
      $this->db->where('id', $id); 
      $query = $this->db->get('ch6_users'); 
      return $query; 
    } 
    ```

如果您看到以下输出，则表示操作成功：

```php
array(1) { 
  [0]=> 
  object(stdClass)#20 (7) { 
    ["id"]=> 
    string(1) "1" 
    ["firstname"]=> 
    string(4) "Lucy" 
    ["lastname"]=> 
    string(5) "Welsh" 
    ["username"]=> 
    string(6) "lwelsh" 
    ["password"]=> 
    string(8) "password" 
    ["created_date"]=> 
    string(10) "1366114115" 
    ["is_active"]=> 
    string(3) "yes" 
  } 
} 
```

## 它是如何工作的...

在前面的控制器中，`public function select_row()` 将 `$id` 赋值为 `1`——然而，这也可以从 **post**、**get**、**session** 或其他来源完成——并加载数据库模型，如下所示将变量 `$id` 传递给它：

```php
    $this->load->model('Database_model'); 
    $this->Database_model->insert_batch_data($data);  
```

模型函数 `select_row()` 从表 `'ch6_users'` 中提取匹配的记录并将其返回给调用控制器。

# Active Record – 更新

CRUD 中的 U 代表在数据库中更新数据记录的过程。CodeIgniter 使用数据库函数 `$this->db->update()` 更新数据库记录；本菜谱将解释如何进行。

## 准备工作

以下是需要支持此菜谱的 SQL 代码；您需要根据您的具体情况对其进行调整。

```php
CREATE TABLE IF NOT EXISTS `ch6_users` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `firstname` varchar(50) NOT NULL,
  `lastname` varchar(50) NOT NULL,
  `username` varchar(20) NOT NULL,
  `password` varchar(20) NOT NULL,
  `created_date` int(11) NOT NULL,
  `is_active` varchar(3) NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB  DEFAULT CHARSET=latin1 AUTO_INCREMENT=1 ;

INSERT INTO `ch6_users` (`firstname`, `lastname`, `username`, `password`, `created_date`, `is_active`) VALUES ('Jessica', 'Welsh', 'jesswelsh', 'password', 1366114115, 'yes');
```

## 如何操作...

我们将创建以下两个文件（或者如果您已经创建了这些文件，则修改它们）：

+   `/path/to/codeigniter/application/controllers/database.php`

+   `/path/to/codeigniter/application/models/database_model.php`

以下步骤将演示如何使用 CodeIgniter Active Record 更新数据库中的数据：

1.  将以下代码添加到文件：`/path/to/codeigniter/application/controllers/database.php`

    ```php
    public function update_row() {
      $id = 1; 

      $data = array( 
        'firstname' => 'Jessica', 
        'lastname' => 'Welsh', 
        'username' => 'jesswelsh', 
        'password' => 'password', 
        'created_date' => time(), 
        'is_active' => 'yes' 
      ); 

      $this->load->model('Database_model'); 
      $result = $this->Database_model->update_row($id, $data);  

      redirect('database/select_row');
    } 

    Add the following code into the file: /path/to/codeigniter/application/models/database_model.php:::

    function update_row($id, $data) { 
      $this->db->where('id', $id); 
      $this->db->update('ch6_users', $data); 
    }

    array(1) { 
      [0]=> 
      object(stdClass)#20 (7) { 
        ["id"]=> 
        string(1) "1" 
        ["firstname"]=> 
        string(7) "Jessica" 
        ["lastname"]=> 
        string(5) "Welsh" 
        ["username"]=> 
        string(9) "jesswelsh" 
        ["password"]=> 
        string(8) "password" 
        ["created_date"]=> 
        string(10) "1366117753" 
        ["is_active"]=> 
        string(3) "yes" 
      } 
    } 
    ```

## 它是如何工作的...

在我们刚才看到的控制器中，`public function update_row()`将`$id`赋值为`1`——然而，这可以来自 post、get、session 或另一个来源——并将数据库模型加载进来，如下将变量`$id`传递给它：

```php
  $this->load->model('Database_model'); 
  $result = $this->Database_model->update_row($id, $data);  
```

模型函数`update_row()`按照以下方式更新表中的匹配记录：

```php
function update_row($id, $data) { 
  $this->db->where('id', $id); 
  $this->db->update('ch6_users', $data); 
} 
```

# ActiveRecord – 删除

CRUD 中的 D 用于在数据库表中删除数据行。CodeIgniter 使用`$this->db->delete()`数据库函数从数据库中删除行；它将在以下部分中使用。

## 准备工作

以下是需要支持此食谱的 SQL 代码；你需要根据你的情况对其进行调整：

```php
CREATE TABLE IF NOT EXISTS `ch6_users` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `firstname` varchar(50) NOT NULL,
  `lastname` varchar(50) NOT NULL,
  `username` varchar(20) NOT NULL,
  `password` varchar(20) NOT NULL,
  `created_date` int(11) NOT NULL,
  `is_active` varchar(3) NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB  DEFAULT CHARSET=latin1 AUTO_INCREMENT=1 ;

INSERT INTO `ch6_users` (`firstname`, `lastname`, `username`, `password`, `created_date`, `is_active`) VALUES ('Jessica', 'Welsh', 'jesswelsh', 'password', 1366114115, 'yes');
```

## 如何操作...

我们将创建以下两个文件（或者如果你已经创建了这些文件，则修改它们）：

+   `/path/to/codeigniter/application/controllers/database.php`

+   `/path/to/codeigniter/application/models/database_model.php`

以下步骤将演示如何使用 CodeIgniter Active Record 从数据库中删除数据：

1.  将以下代码添加到文件`/path/to/codeigniter/application/controllers/database.php`中：

    ```php
    public function delete_row() { 
      $id = 1; 

      $this->load->model('Database_model'); 
      $result = $this->Database_model->delete_row($id);    

      redirect('database/select_row');   
    } 
    ```

1.  将以下代码添加到文件`/path/to/codeigniter/application/models/database_model.php`中：

    ```php
    function delete_row($id) { 
      $this->db->where('id', $id); 
      $this->db->delete('ch6_users');
    } 
    ```

## 工作原理...

在前面的控制器中，`public function delete_row()`将`$id`赋值为`1`——然而，这可以来自 post、get、session 或另一个来源——并将数据库模型加载进来，如下将变量`$id`传递给它：

```php
  $this->load->model('Database_model'); 
  $result = $this->Database_model->delete_row($id);    
```

模型函数`delete_row()`从表中删除匹配的记录：

```php
function delete_row($id) { 
  $this->db->where('id', $id); 
  $this->db->delete('ch6_users'); 
} 
```

# 遍历数据库结果

在任何具有数据库连接的应用程序中，你可能需要显示数据库中的记录；遍历查询返回的数据行是你在编程中将要执行的最常见任务之一。CodeIgniter 使用 PHP 中的每个语句来处理遍历数据库结果。在这个食谱中，我们将一次遍历每条记录，输出相关信息。

## 准备工作

为了支持这个食谱，我们将创建一个数据库表并向其中写入一些数据。如果你已经有了数据，你可以跳过这个食谱；如果没有，请将以下代码复制到你的数据库中：

```php
CREATE TABLE IF NOT EXISTS `loop_table` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `first_name` varchar(255) NOT NULL,
  `last_name` varchar(255) NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB  DEFAULT CHARSET=latin1 AUTO_INCREMENT=3 ;

INSERT INTO `loop_table` (`id`, `first_name`, `last_name`) VALUES
(1, 'Lucy', 'Welsh'),
(2, 'Rob', 'Foster');
```

## 如何操作...

1.  将以下代码添加或修改到你的控制器中：

    ```php
    public function loop_through_data() {
      $this->load->model('Some_model');
      $data['query'] = $this->Some_model->select_data();
      $this->load->view('some_view', $data);
    }
    ```

1.  将以下代码添加或修改到你的模型中：

    ```php
    function select_data() {
      $query = $this->db->get('loop_table');
      return $query;
    }
    ```

1.  将以下代码添加或修改到你的视图中：

    ```php
    foreach ($query->result() as $row) {
      echo $row->first_name . ' ' . $row->last_name;
      echo '<br />';
    }
    ```

## 工作原理...

首先，让我们看看 SQL 代码；如果你使用了前面的 SQL 代码，我们所做的只是创建一个非常简单的表，并用两行数据填充它。

接下来，我们调用控制器函数`loop_through_data()`，它加载一个模型；在这种情况下，将`Some_model`重命名为与你的应用程序相关的模型。我们调用模型函数`select_data()`，将返回的结果存储在`$data`数组中，或者更具体地说，在`$data`数组的一个部分，我们称之为`'query'`：

```php
  $data['query'] = $this->Some_model->select_data();
```

模型函数`select_data()`从数据库表`loop_table`中检索所有行，并将其返回给调用控制器函数。

返回到我们的控制器，现在我们已经将数据库结果存储在`$data`数组中，我们可以调用`view`文件`some_view.php`——显然，您需要将其重命名为您应用程序中的其他名称——并将`$data`数组传递给它：

```php
  $this->load->view('some_view', $data);
```

`view`文件随后使用简单的`foreach()`循环遍历`$query`中的每个结果。让我们更仔细地看看这个`foreach()`循环。看看以下代码行：

```php
foreach ($query->result() as $row) {
```

记得我们是如何将数据库结果存储在`$data['query']`中的吗？那么，我们将使用`$data`数组中的`'query'`部分，它已存储数据库结果，并将使用 CodeIgniter 函数`result()`来处理它。我听到你问，“`result()`函数做什么？”`result()`函数将接受一个对象或数组，并允许您遍历每一行，允许您对那一行中的单个数据项进行操作。

因此，我们使用`result()`将`$query`拆分为每一行，将那一行传递给`$row`（因为这是显而易见的），并允许我们执行如下操作：

```php
  echo $row->first_name . ' ' . $row->last_name;
```

这是在`$row`中显示每个人的姓名和姓氏。

# 使用`num_rows()`计算返回的结果数量

计算返回的结果数量是有用的——如果一段代码期望至少有一行，但传递了零行，则可能会出现错误。如果不处理零结果的可能性，应用程序可能会变得不可预测地不稳定，并可能向恶意用户提供有关应用程序架构的线索。确保正确处理零结果是我们在这里要关注的问题。

## 如何做...

1.  我们将创建一个模型和控制器代码块。您可能已经在控制器、模型或视图中编写了执行以下所有或部分操作的代码——显然，您可以跳过您不需要的任何步骤。将以下代码添加或修改到您的控制器中：

    ```php
    $this->load->model('Some_model');
    $data['query'] = $this->Some_model->some_model_function();
    $this->load->view('some_view', $data);
    ```

1.  将以下代码添加或修改到您的模型中：

    ```php
    function some_model_function() {	
           $query = $this->db->get('database_table_name');
           return $query;
    }
    ```

1.  将以下代码添加或修改到您的视图中：

    ```php
    if ($query->num_rows() > 0) {
      foreach ($query->result() as $row) {
        echo $row->item1;
        echo $row->item2;
      }
    } else {
      echo 'No results returned';
    }
    ```

## 它是如何工作的...

这相当常见；控制器加载所需的模型并调用该模型中的函数；该模型的结果存储在数组中。然后将其传递到视图中。正是在这里，我们会计算行数。看看加粗的行。我们使用 CodeIgniter 函数`num_rows()`来查看`$query`结果，并计算模型返回的行数。我们询问行数是否大于零。如果是，则意味着模型至少有一个结果——然后我们像通常一样遍历`$query`数组。然而，如果结果的数量不大于零，这意味着模型没有返回任何结果。因此，我们使用 else 语句显示一条简短的消息，说明没有返回结果。

# 使用`count_all_results()`计算返回的结果数量

计算返回的结果数量是有用的——如果代码部分期望至少有一行，而传递了零行，则可能会出现错误。如果不处理零结果的可能性，应用程序可能会变得不可预测地不稳定，并可能向恶意用户提供有关应用程序架构的线索。确保正确处理零结果是我们在这里要关注的内容。

## 如何操作...

1.  将以下代码添加或修改到您的控制器中：

    ```php
    $this->load->model('Some_model');
    $data['num_results'] = $this->Some_model->some_model_function();
    $this->load->view('some_view', $data);
    ```

1.  将以下代码添加或修改到您的模型中：

    ```php
    function some_model_function() {	
      $this->db->from('table');
      return $num_rows = $this->db->count_all_results();
    }
    ```

1.  将以下代码添加或修改到您的视图中：

    ```php
    if (isset($num_results)) {
      echo 'There are ' . $num_results . ' returned';
    }
    ```

这与上面 `num_rows()` 的配方相当类似，但有一些关键的区别。我们首先调用一个控制器，该控制器加载所需的模型并在其中调用一个函数。看看加粗的代码：`$this->db->count_all_results();`。这将返回给定查询返回的结果数量。此代码的结果存储在数组中，并传递到视图中，我们测试变量 `$num_results` 是否已设置；如果是，我们输出一个简短的消息，指示结果数量。

# 查询绑定

绑定查询是另一个有用的安全过程；如果您在查询中使用绑定，CodeIgniter 会自动转义值，您无需手动进行。

## 准备工作

将以下 SQL 代码复制到您的数据库中：

```php
CREATE TABLE IF NOT EXISTS `users` (
  `user_id` int(11) NOT NULL AUTO_INCREMENT,
  `user_first_name` varchar(125) NOT NULL,
  `user_last_name` varchar(125) NOT NULL,
  `user_email` varchar(255) NOT NULL,
  `user_created_date` int(11) NOT NULL COMMENT 'unix timestamp',
  `user_is_active` varchar(3) NOT NULL COMMENT 'yes or no',
  PRIMARY KEY (`user_id`)
) ENGINE=InnoDB  DEFAULT CHARSET=latin1 AUTO_INCREMENT=1 ;

INSERT INTO `users` (`user_first_name`, `user_last_name`, `user_email`, `user_created_date`, `user_is_active`) VALUES
('Chloe', 'Graves', 'cgraves@domain.com', 1366114115, 'yes'),
('Mark', 'Brookes', 'mbrookes@domain.com', 1366114115, 'yes');
```

## 如何操作...

在您的任何模型中，调整您的查询代码以反映以下内容：

```php
$query = "SELECT * FROM users WHERE users.is_active = ? AND users.created_date > ?";
$this->db->query($query, 'yes', '1366114114');
```

## 它是如何工作的

以名为 `users` 的表为例，查询将尝试获取所有 `users.is_active` 等于 `Y` 且 `users.created_date` 大于 `1359706809 (02/01/2013 – 03:20)` 的记录。但，您会注意到查询中有两个问号，每个问号代表 `$data array` 中的一个项目。`$data` 数组中的值按顺序传递到查询中，通过这一行 `$this->db->query($query, $data);`。因此，查询中的第一个问号将被数组中的第一个项目替换，查询中的第二个问号将被数组中的第二个项目替换，依此类推。

# 查找最后插入的 ID

返回最后插入行的主键在您可能希望将数据写入多个表且数据可能通过键相关联的情况下非常有用。CodeIgniter 提供了返回最后插入键的支持。

## 如何操作...

1.  将以下代码添加或修改到模型中：

    ```php
      function insert($data) { 
     if ($this->db->insert($data, 'table_name')) { 
     return $this->db->last_id();
     } else { 
     return false; 
     } 
    } 
    ```

## 它是如何工作的...

看看加粗的行。我们测试 `$this->db->insert($data);` 返回的值，如果成功则返回 true，如果出错则返回 false。如果返回值是 true，我们获取此连接最后插入记录的主键；此值与 `return $this->db->insert_id();` 一起从模型返回到调用函数的代码。如果数据库插入失败，它将返回 false。您可以轻松地调整上述配方；只需将加粗的行放入您的模型中。

# 查找受影响行数

查找受影响行数可以在几种方式下有用——也许你想要更新一些记录，并且只有当一定数量的记录被更新时才继续，或者也许你只是想显示被查询删除或更新的行数。

## 如何操作...

1.  将以下代码添加或修改到你的模型中：

    ```php
      function update($id, $data) {
        $this->db->where('id', $id); 
        if ($data->db->update($data, 'table_name')) {
          return $this->db->affected_rows(); 
        } else { 
          return false; 
        } 
      } 
    ```

## 它是如何工作的...

模型`function update()`接受两个参数：一个`$data`数组和我们希望更新的数据库行的`$id`数组。

接下来，我们测试`$this->db->update($data);`返回的值，如果成功将返回 true，如果出错将返回 false。如果返回值是 true，我们将使用以下行获取更新的受影响行数：

```php
return $this->db->affected_rows();  
```

如果更新没有发生，返回值将是 false。

# 查找最后数据库查询

有时，了解对数据库执行的最后查询是有用的，无论是为了调试目的，还是出于你希望有数据库每次交互的审计记录的原因——你会惊讶于你需要这样做多少次。CodeIgniter 提供了一个非常实用的函数，你可以用它来记录 CodeIgniter 最近发送到数据库的查询。

## 如何操作...

1.  将以下行代码添加或修改到你的控制器或模型中：

    ```php
    $this->db->last_query();
    ```

## 它是如何工作的...

简单来说，这个函数将返回发送到数据库的最后查询；你可以将它放在控制器或模型中（如果你愿意，甚至可以放在视图中，但最好将其保留在应用程序的逻辑部分而不是视图中）。它将以字符串的形式返回你可以用作审计的查询；例如，考虑以下代码行：

```php
log_message('level', $this->db->last_query())
```

上一行代码将最后查询写入到你的日志文件中，其中`'level'`表示消息类型。我们将在第九章中介绍一些错误报告和日志记录配方，*扩展核心*。

# 使用 CodeIgniter 数据库迁移

假设你在一个由其他开发者组成的团队中工作，每个人都忙于工作，对代码和数据库结构进行更改。跟上所有这些数据库更改可能成为一个挑战，尤其是当许多人几乎在项目的同一区域工作时。

CodeIgniter 迁移为你提供了安装（或回滚）可能支持代码更改的数据库结构更改的选项。例如，如果你正在对用户注册脚本进行编码更改——这个更改需要在数据库表中添加一个列；你可以在你的版本控制提交中包含一个 CodeIgniter 数据库迁移脚本（假设你在使用版本控制）——其他开发者现在将知道，为了你的代码更改能够工作，他们必须运行迁移，这将修改他们的数据库。

迁移还允许你回滚更改。这不应与数据库事务回滚的概念混淆；将使用迁移回滚想象成卸载之前安装的更改。

## 准备工作

在进行此操作之前，我们需要更改一些配置设置，因此打开 `/path/to/codeigniter/application/config/migration.php` 并找到以下选项：

| Preference | Default Value | Options | Description |
| --- | --- | --- | --- |
| `migration_enabled` | FALSE | TRUE/FALSE | 指定你是否希望启用迁移；TRUE 是启用，FALSE 是禁用。 |
| `migration_version` | 0 | None | 指定数据库当前使用的迁移版本，或者更确切地说，是你希望工作的最合适的迁移版本。我们将在本章后面详细讨论这个问题。通过使用 `current()`，我们将安装 `'migration_version'` 中设置的最新值。 |
| `migration_path` | APPPATH.'migrations/' | None | 指定存储迁移文件的文件夹路径。迁移文件是 PHP 脚本，其中包含定义必要数据库更改的查询。请确保已将 `migrations` 文件夹设置为可写。 |

确保使用以下行在你的控制器中加载迁移库：

```php
$this->load->library('migration');
```

在这个菜谱中，我们将创建一个简单的用户表，并使用迁移库添加和删除该表中的一个列。将以下 SQL 语句输入到你的数据库中：

```php
CREATE TABLE IF NOT EXISTS `users` (
  `user_id` int(11) NOT NULL AUTO_INCREMENT,
  `user_first_name` varchar(125) NOT NULL,
  `user_last_name` varchar(125) NOT NULL,
  `user_email` varchar(255) NOT NULL,
  PRIMARY KEY (`user_id`)
) ENGINE=InnoDB DEFAULT CHARSET=latin1 AUTO_INCREMENT=1 ;
```

## 如何操作...

首先，所有你的数据库迁移文件都应该放在 `/path/to/codeigniter/application/migrations/` 路径下的 `migrations` 文件夹中。

如果文件夹尚不存在，你需要在 `/path/to/codeigniter/application/` 文件夹中创建它——确保给它写入权限。

1.  我们将要创建以下两个文件：

    +   `/path/to/codeigniter/application/migrations/001_add_icon.php`

    +   `/path/to/codeigniter/application/controllers/migrate.php`

        ### 小贴士

        注意文件名 `001_add_icon.php`。第一部分（001）是迁移号；每次添加新的迁移文件时，它都会递增。第二部分（add_icon）是对迁移文件目的的描述性指示。将以下代码添加到文件 `001_add_icon.php` 中。此迁移文件定义了要运行的查询以实现迁移更改或从该更改中回滚。

        ```php
        <?php defined('BASEPATH') OR exit('No direct script access allowed');
        class Migration_Add_icon extends CI_Migration {
          public function up() {
            $this->db->query("ALTER TABLE `users` ADD COLUMN `user_icon`  TEXT NULL AFTER `user_email`;");
          }
          public function down() {
            $this->db->query("ALTER TABLE `users` DROP COLUMN `user_icon`;");
          }
        }
        ```

1.  将以下代码添加到 `/path/to/codeigniter/application/controllers/migrate.php`；迁移控制器为我们提供了访问 CodeIgniter 迁移函数的权限。

    ```php
    <?php if (!defined('BASEPATH')) exit('No direct script access allowed');

    class Migrate extends CI_Controller {
      function __construct() {
        parent::__construct();

        if ( ! $this->input->is_cli_request() ) {
          echo 'Only access via command line.';
          exit;
        }

        $this->load->library('migration');
      }

      public function index() {
        echo 'Config: ' . $config['migration_version'];
      }

      public function current() {
        if ( ! $this->migration->current()) {
          show_error($this->migration->error_string());
        }        
      }

      public function latest() {
        if ( ! $this->migration->latest()) {
        show_error($this->migration->error_string());
      }        
    }    

      public function version() {
        if ( $this->uri->segment(3) == '') {
          echo 'You must specify a migration version number';
        } else {
          if ( ! $this->migration->version($this->uri->segment(3)) ) {
            show_error($this->migration->error_string());
          }       
       } 
      }
    }
    ```

好的，到目前为止我们做了什么？我们已经配置了 CodeIgniter 中的迁移，我们创建了第一个迁移文件（注意正确命名），并且我们有两个文件：控制器 `migrate.php` 和迁移文件 `001_add_icon.php`。

在迁移文件`001_add_icon.php`中，有 222 个函数；在这些函数中，`up()`和`down()`是定义与你的代码更改一起的 SQL 语句的函数。`down()`函数是定义如果有人（可能是另一个开发者）希望撤销你可能做出的代码更改时，将定义用于删除更改的 SQL 语句的地方；因此，它支持 SQL。

在控制器`migrate.php`中，我们创建了一些供我们操作的迁移函数，例如`current()`和`latest()`。以下两个食谱将向你展示这些迁移的一些基本用法。

# 使用 current()移动到当前版本

要简单地更改你的数据库，使其与`$config['migration_version']`中的版本号相对应，你应该使用`current()`函数。

## 准备工作

确保你已经遵循了前面的食谱，*使用 CodeIgniter 数据库迁移*。

## 如何操作...

1.  使用你的命令行（终端应用程序），导航到你的 CodeIgniter 安装根目录（`index.php`文件所在的位置）并输入以下内容：

    ```php
    php index.php migrate current
    ```

## 它是如何工作的...

考虑以下命令行：

```php
php index.php migrate current 
```

我们首先应该牢记的是迁移控制器中的构造函数。构造函数正在查看迁移控制器是如何被访问的；如果它不是通过命令行访问的，它将拒绝访问迁移控制器——这是一个有用的安全措施。

通过输入我们刚才看到的命令，你会运行`public function current()`。该函数不接受任何参数。CodeIgniter 会检查迁移文件夹中与配置文件`/path/to/codeigniter/application/config/migration.php`中设置的`$config['migration_version']`值对应的文件编号的文件。

# 使用 version()回滚/前进版本

你可能希望故意通过指向特定的迁移编号来更改数据库。这可以通过在 CodeIgniter 中使用`version()`函数来实现。

## 准备工作

确保你已经遵循了前面的食谱，*使用 CodeIgniter 数据库迁移*。

## 如何操作...

1.  使用你的命令行（终端应用程序），导航到你的 CodeIgniter 安装根目录（`index.php`文件所在的位置）并输入以下内容：

    ```php
    php index.php migrate version 1
    ```

## 它是如何工作的...

考虑以下命令行：

```php
php index.php migrate version number

```

`number`被突出显示，因为它指定了要移动到的迁移文件编号，即`1`、`2`、`3`等等。

我们首先应该牢记的是迁移控制器中的构造函数。构造函数正在查看迁移控制器是如何被访问的；如果它不是通过命令行访问的，它将拒绝访问迁移控制器——这是一个有用的安全措施。

通过输入前面的命令，您将运行 `public function version()`，传递给它第三个参数（其值为 `1`）。CodeIgniter 将检查迁移文件夹，寻找与第三个参数（1）对应的文件编号，令人惊讶的是，这个编号正是我们创建的迁移文件编号——谁知道呢？

CodeIgniter 将加载迁移文件 `001_add_icon.php` 并立即运行 `public function up()`，这将向数据库表 `'users'` 添加 `user_icon` 列。

通过在命令行中输入以下内容，我们可以撤销创建 `user_icon` 列的操作：

```php
php index.php migrate version 0

```

然后，CodeIgniter 将在迁移文件中运行公共函数 `down()`，这将删除 `user_icon` 列。

# 从数据库结果生成 XML

从数据库生成 XML 可能以多种方式有用，也许您希望使用 SOAP 请求通过网络发送查询数据，或者也许您正在使用它来构建一些用于网络服务的数据。无论您的目的如何，这就是如何操作——我们还将探讨一些实际应用——例如，我们将从数据库查询生成 XML 输出。

## 准备工作

首先，我们需要创建一个表并输入一些示例数据，这样您将看到一些以 CSV 格式显示的数据，所以考虑到这一点，将以下代码复制到 SQL：

```php
CREATE TABLE IF NOT EXISTS `users` (
  `user_id` int(11) NOT NULL AUTO_INCREMENT,
  `user_first_name` varchar(125) NOT NULL,
  `user_last_name` varchar(125) NOT NULL,
  `user_email` varchar(255) NOT NULL,
  `user_created_date` int(11) NOT NULL COMMENT 'unix timestamp',
  `user_is_active` varchar(3) NOT NULL COMMENT 'yes or no',
  PRIMARY KEY (`user_id`)
) ENGINE=InnoDB  DEFAULT CHARSET=latin1 AUTO_INCREMENT=1 ;

INSERT INTO `users` (`user_first_name`, `user_last_name`, `user_email`, `user_created_date`, `user_is_active`) VALUES
('Chloe', 'Graves', 'cgraves@domain.com', 1366114115, 'yes'),
('Mark', 'Brookes', 'mbrookes@domain.com', 1366114115, 'yes');
```

## 如何操作...

我们将创建以下文件：

+   `/path/to/codeigniter/application/controllers/export.php`

1.  创建控制器 `export.php`。此控制器将加载 CodeIgniter 的 `dbutil`（数据库工具）类，该类将提供对各种数据库特定操作的支持并生成 XML。将以下代码添加到您的 `export.php` 控制器中：

    ```php
    <?php if (!defined('BASEPATH')) exit('No direct script access allowed');

    class Export extends CI_Controller {

      function __construct() {
        parent::__construct();
        $this->load->helper('url');
        $this->load->dbutil();
      }

      public function index() {
        redirect('export/xml');
      }

      public function xml() {
        $config = array ('root'    => 'root',
                         'element' => 'element', 
                         'newline' => "\n", 
                         'tab'    => "\t"
                        );

        $query = $this->db->query("SELECT * FROM users");

        echo $this->dbutil->xml_from_result($query, $config);
      }
    }
    ```

## 它是如何工作的...

好的，看看加粗的那一行——我们在控制器构造函数中加载数据库工具类。这个工具类包含一些用于处理数据库的出色函数。我们使用它来提供对 `xml_from_result()` 函数的访问。

`export.php` 控制器的 `index()` 函数将重定向我们到 `public function xml()`，该函数执行数据库查询。当然，您可以使用任何数据源，但我们在这里调用数据库并将结果存储在数组 `$query` 中。这被传递给 CodeIgniter 函数 `xml_from_result()`。`xml_from_result()` 函数接受以下两个参数：

+   `$query`：这是 XML 的数据；在这种情况下，我们数据库查询的输出。

+   `$config`：这是配置参数；在这种情况下，XML 格式化选项。

我们随后将 `xml_from_result()` 的结果输出到屏幕上——您可以通过在浏览器中查看页面源代码来查看结果。您不必输出它；如果您需要将 XML 用于其他目的，您可以将其存储在变量中。

请确保将数据库查询单独放入其自己的模型中——查询在先前的控制器中显示，用于说明目的。

# 从数据库结果生成 CSV

可能你被要求做的最常见的事情之一，尤其是如果你正在构建一个可能包含用户、产品、订单和各种其他指标复杂的应用程序，就是提供某种形式的数据报告。可能你会被要求生成一个 CSV 文件，以下部分展示了如何操作。

## 准备工作

首先，我们需要创建一个表并输入一些示例数据，这样你就能看到一些以 CSV 格式的数据，所以带着这个想法，将以下代码复制到 SQL 中：

```php
CREATE TABLE IF NOT EXISTS `users` (
  `user_id` int(11) NOT NULL AUTO_INCREMENT,
  `user_first_name` varchar(125) NOT NULL,
  `user_last_name` varchar(125) NOT NULL,
  `user_email` varchar(255) NOT NULL,
  `user_created_date` int(11) NOT NULL COMMENT 'unix timestamp',
  `user_is_active` varchar(3) NOT NULL COMMENT 'yes or no',
  PRIMARY KEY (`user_id`)
) ENGINE=InnoDB  DEFAULT CHARSET=latin1 AUTO_INCREMENT=1 ;

INSERT INTO `users` (`user_first_name`, `user_last_name`, `user_email`, `user_created_date`, `user_is_active`) VALUES
('Chloe', 'Graves', 'cgraves@domain.com', 1366114115, 'yes'),
('Mark', 'Brookes', 'mbrookes@domain.com', 1366114115, 'yes');
```

现在数据库已经准备好了，我们需要确保你调用了数据库实用工具类；确保你使用以下行调用它：

```php
$this->load->dbutil();
```

你可以将此行放在你控制器的构造函数中，或者在你的控制器函数中调用它。

此外，由于我们将要创建一个文件，我们需要`'file'`辅助函数的支持，所以请确保你使用以下行调用辅助函数：

```php
$this->load->helper('file');
```

## 如何操作...

我们将要创建以下文件：

+   `/path/to/codeigniter/application/controllers/export.php`

### 强制下载

将以下代码添加到你的`export.php`控制器中：

```php
<?php if (!defined('BASEPATH')) exit('No direct script access allowed');

class Export extends CI_Controller {

  function __construct() {
    parent::__construct();
    $this->load->helper('download');
    $this->load->dbutil();    
    $this->load->helper('url');
  }

  public function index() {
    redirect('export/csv');
  }

  public function csv() {
    $query = $this->db->query("SELECT * FROM users");
    $delimiter = ",";
    $newline = "\r\n";    

    force_download('myfile.csv', $this->dbutil->csv_from_result($query, $delimiter, $newline));
  }
```

## 它是如何工作的...

好的，看看加粗的行——我们在控制器构造函数中加载了 CodeIgniter 的`'download'`辅助函数和数据库实用工具类。这将帮助我们完成这个配方。

`export.php`控制器函数`index()`将我们重定向到`public function csv()`，它执行一个数据库查询。当然，这里可以是任何数据源，但我们调用数据库并将结果存储在数组`$query`中。这被传递给接受以下两个参数的 CodeIgniter 函数`force_download()`：

+   要创建的文件名和扩展名（或在这种情况下，下载的文件）

+   将进入文件的数据；在这种情况下，我们使用 CodeIgniter 函数`csv_from_result()`，它将从数据库查询中取出一行数据并将其转换为以分隔符分隔的文本字符串。`csv_from_result()`函数接受以下三个参数：

    +   `$query`: 这是 CSV 中的数据；在这种情况下，我们数据库查询的输出

    +   `$delimiter`: 这是数据分隔符，即它指定了我们如何分隔每个单元格的数据；这通常是逗号（`,`）。

    +   `$newline`: 这是新行字符；它通常是 '`\n\n`'

如果一切按计划进行，`force_download()`将像其名称所暗示的那样，强制下载 CSV 文件。

### 保存到文件

将以下代码添加到你的`export.php`控制器中：

```php
<?php if (!defined('BASEPATH')) exit('No direct script access allowed');

class Export extends CI_Controller {

  function __construct() {
    parent::__construct();
 $this->load->helper('url');
 $this->load->helper('file');
 $this->load->dbutil(); 
  }

  public function index() {
    redirect('export/csv');
  }

  public function csv() {
    $query = $this->db->query("SELECT * FROM users");

    $delimiter = ",";
    $newline = "\r\n";

    $data = $this->dbutil->csv_from_result($query, $delimiter, $newline);
 $path = '/path/to/write/to/myfile.csv';

 if ( ! write_file($path, $data)) {
 echo 'Cannot write file - permissions maybe?';
 } else {
 echo 'File write OK';
 }
  }
```

## 它是如何工作的...

好的，看看加粗的行；我们在控制器构造函数中加载了 CodeIgniter 的`'download'`、`'file'`辅助函数和数据库实用工具类。这将帮助我们完成这个配方。我们还在写入磁盘时添加了 CodeIgniter 特定的语法。

`export.php` 控制器的 `index()` 函数将我们重定向到 `public function csv()`，该函数执行一个数据库查询。当然，这里可以是任何数据源，但我们在这里调用数据库查询并将结果存储在 `$query` 数组中。然后我们调用 CodeIgniter 的 `csv_from_result()` 函数，其中 `csv_from_result()` 接受以下三个参数：

+   `$query`: CSV 的数据；在这种情况下，是我们数据库查询的输出

+   `$delimiter`: 数据分隔符，即它指定了我们如何分隔每个数据单元格

+   `$newline`: 新行字符；通常设置为 `'\n\n'`

`csv_from_result()` 函数将它的输出存储在变量 `$data` 中。然后我们尝试运行 CodeIgniter 的 `write_file()` 函数，该函数接受以下两个参数：

+   要写入文件的路径，包括文件名和扩展名；请记住，此路径应该是可写的

+   要写入文件的数据

如果一切按计划进行，该食谱将返回消息 `文件写入 OK`——当然，你应该根据需要替换它。如果失败，它将返回错误消息，并在必要时用你自己的代码替换。

## 还有更多...

如果文件没有写入，那么你很可能没有足够的权限写入目标文件夹。你需要修改目标文件夹的权限，使其达到允许 CodeIgniter 写入的水平。例如，在 Linux/Mac 中，你会在终端中使用 `chmod` 命令。
