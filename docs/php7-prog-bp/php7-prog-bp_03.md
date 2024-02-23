# 第三章：构建社交通讯服务

根据可靠的词典，通讯是定期发布给社会、企业或组织成员的公告。

在本章中，我们将构建一个电子邮件通讯，允许会员订阅和取消订阅，接收特定类别的更新，并允许营销人员检查有多少人访问了某个链接。

我们将为用户构建一个身份验证系统，以便登录和退出通讯管理系统，这是一个社交登录系统，供订阅会员轻松检查其订阅以及为订阅者和管理员提供简单的仪表板。

# 身份验证系统

在本章中，我们将实现一个新的身份验证系统，以允许通讯通讯的管理员进行身份验证。自 PHP5 以来，PHP 已经改进并添加了一个功能，面向对象的开发人员已经用来分隔命名空间。

让我们首先定义一个名为`Newsletter`的命名空间，如下所示：

```php
<?php 
namespace Newsletter;  
//this must always be in every class that will use namespaces 
class Authentication { 
} 
?> 

```

在上面的示例中，我们的`Newsletter`命名空间将有一个`Authentication`类。当其他类或 PHP 脚本需要使用`Newsletter`的`Authentication`类时，他们可以简单地使用以下代码声明它：

```php
Use Newsletter\Authentication; 

```

在我们的`Newsletter`类中，让我们使用**bcrypt**创建一个简单的用户检查，这是一种流行且安全的创建和存储散列密码的方法。

### 注意

自 PHP 5.5 以来，bcrypt 已内置到`password_hash()` PHP 函数中。PHP 的`password_hash()`函数允许密码成为散列。相反，当您需要验证散列是否与原始密码匹配时，可以使用`password_verify()`函数。

我们的类将非常简单-它将有一个用于验证输入的电子邮件地址和散列密码是否与数据库中的相同的函数。我们必须创建一个只有一个方法`verify()`的简单类，该方法接受用户的电子邮件和密码。我们将使用`bcrypt`来验证散列密码是否与我们数据库中的相同：

```php
Class Authorization { 
     public function verify($email, $password) { 
         //check for the $email and password encrypted with bcrypt 
         $bcrypt_options = [ 
            'cost' => 12, 
            'salt' => 'secret' 
         ]; 
         $password_hash = password_hash($password, PASSWORD_BCRYPT, $bcrypt_options); 
         $q= "SELECT * FROM users WHERE email = '". $email. "' AND password = '".$password_hash. "'"; 
         if($result = $this->db->query($q)) { 
                     while ($obj = results->fetch_object()) { 
                           $user_id = $obj->id; 
} 

         } else { 
   $user_id = null; 
} 
         $result->close(); 
         $this->db->close(); 
         return $user_id; 

    } 
} 

```

然而，我们需要让`DB`类能够在我们的数据库中执行简单的查询。对于这个简单的一次性项目，我们可以在我们的`Authentication`类中简单使用依赖注入的概念。

我们应该创建一个相当简单的 IOC 容器类，它允许我们实例化数据库。

让我们称之为`DbContainer`，它允许我们将类（例如`Authentication`）连接到`DB`类：

```php
Namespace Newsletter; 
use DB; 
Class DbContainer { 
   Public function getDBConnection($dbConnDetails) {  
   //connect to database here: 
    $DB = new \DB($server, $username, $password, $dbname); 
       return $DB; 
  } 
} 

```

但是，如果您立即使用此函数，将会出现一个错误，指出找不到文件并将加载`DB`类。

以前，我们使用了`use`系统来要求类。为了使其工作，我们需要创建一个自动加载程序函数来加载我们的`DB`类，而无需使用`require`语句。

在 PHP 中，我们可以创建`spl_autoload_register`函数，它将自动处理所需的文件。

以下是基于 PHP 手册中的示例的示例实现：

```php
<?php 
/** 
 * After registering this autoload function with SPL, the following line 
 * would cause the function to attempt to load the \Newsletter\Qux class 
 * from /path/to/project/src/Newsletter/Qux.php: 
 *  
 *      new \Newsletter\Qux; 
 *       
 * @param string $class The fully-qualified class name. 
 * @return void 
 */ 
spl_autoload_register(function ($class) { 
    // project-specific namespace prefix 
    $prefix = 'Newsletter'; 
    // base directory for the namespace prefix 
    $base_dir = __DIR__ . '/src/'; 
    // does the class use the namespace prefix? 
    $len = strlen($prefix); 
    if (strncmp($prefix, $class, $len) !== 0) { 
        // no, move to the next registered autoloader 
        return; 
    } 
    // get the relative class name 
    $relative_class = substr($class, $len); 
    // replace the namespace prefix with the base directory,               //replace namespace 
    // separators with directory separators in the relative class      //name, append 
    // with .php 
    $file = $base_dir . str_replace('', '/', $relative_class) . '.php'; 
    // if the file exists, require it 
    if (file_exists($file)) { 
        require $file; 
    } 
}); 

```

使用上述代码，我们现在需要创建一个`src`目录，并在应用程序中使用此分隔符`\\`约定来分隔文件夹结构。

使用此示例意味着我们需要将数据库类文件`DB.class.php`放在`src`文件夹中，并将文件名重命名为`DB.php`。

这样做是为了当您在另一个 PHP 脚本中指定要使用`DB`类时，PHP 将在后台自动执行`require src/DB.php`。

继续使用我们的示例`DbContainer`，我们需要以某种方式将所有配置信息（即数据库名称、用户名和密码）传递到`DbContainer`中。

让我们简单地创建一个名为`dbconfig.php`的文件，其中包含数据库详细信息并将其作为对象返回，并要求它：

```php
//sample dbconfig.php 
return array('server' => 'localhost', 
  'username' => 'root', 
  'password => '', 
  'dbname' => 'newsletterdb' 
); 

```

在我们的`DbContainer`类中，让我们创建一个`loadConfig()`函数，从`dbconfig.php`文件中读取，并实例化一个数据库连接：

```php
Class DbContainer { 
public function  loadConfig ($filePath) { 

   if($filePath) { 
     $config = require($filePath); 
     return $config; //contains the array  
   } 

} 

```

现在我们需要创建一个`connect()`方法，这将使我们能够简单地连接到 MySQL 数据库并仅返回连接：

```php
Class DB { 
 //... 
public function connect($server, $username, $password, $dbname) { 
   $this->connection = new MySQLI($server, $username, $password, $dbname); 
     return $this->connection; 
} 
} 

```

通过不将文件名硬编码到我们的函数中，我们使我们的函数更加灵活。在调用`loadConfig()`时，我们需要将`config`文件的路径放入。

我们还使用了`$this`关键字，这样每当我们需要引用`DB`类中的其他函数时，我们只需在自动加载程序加载并实例化`DB`类后调用`$DB->nameOfMethod(someParams)`。

有了这个，我们现在可以轻松地更改`config`文件的路径，以防我们将`config`文件移动到其他路径，例如，到一个通过 Web 直接访问的文件夹。

然后，我们可以轻松地使用这个函数，并在一个单独的类中生成一个数据库实例，例如，在我们的`Newsletter`类中，我们现在可以引用`DB`类连接的一个实例，并在`Newsletter`类中实例化它。

现在我们完成了这一步，我们应该简单地创建一个 Bootstrap 文件，加载`spl_autoload_register`函数和使用`dbContainer`连接到数据库。让我们将文件命名为`bootstrap.php`，它应该包含以下内容：

```php
require('spl_autoloader_function.php'); 

$dbContainer = new \DBContainer; //loads our DB from src folder, using the spl_autoload_functionabove. 

$dbConfig = $db->getConfig('dbconfig.php'); 

$dbContainer = getDB($dbConfig); //now contains the array of database configuration details 

```

下一步是使用以下代码连接到数据库：

```php
$DB = new \DB;  
$DBConn = $DB->connect($dbContainer['server'],$dbContainer['username'],$dbContainer['password'],$dbContainer['dbname']); 

```

当我们都连接到数据库之后，我们需要重写我们的授权查询，以使用新初始化的类。

让我们在我们的`DB`类中创建一个简单的`select_where`方法，然后从`Authorization`类中调用它：

```php
public function select_where($table, $where_clause) { 
   return $this->db->query("SELECT * FROM ". $table." WHERE " . $where_clause); 
} 

```

`Authorization`类现在如下所示：

```php
Class Authorization { 
    //this is used to get the database class into Authorization  
    Public function instantiateDB($dbInstance){ 
       $this->db = $dbInstance; 
    } 

    public function verify($email, $password) { 
         //check for the $email and password encrypted with bcrypt 
         $bcrypt_options = [ 
            'cost' => 12, 
            'salt' => 'secret' 
         ]; 
         $password_hash = password_hash($password, PASSWORD_BCRYPT, $bcrypt_options); 
         //select with condition 
         $this->db->select_where('users', "email = '$email' AND password = '$password_hash'"); 
         if($result = $this->db->query($q)) { 
                     while ($obj = results->fetch_object()) { 
                           $user_id = $obj->id; 
} 

         } else { 
   $user_id = null; 
} 
         $result->close(); 
         $this->db->close(); 
         return $user_id; 

    } 
} 

```

## 为会员创建社交登录

为了让更多人轻松订阅，我们将实现一种方式，让 Facebook 用户可以简单地登录并订阅我们的通讯，而无需输入他们的电子邮件地址。

通过**Oauth**登录 Facebook 通过生成应用程序认证令牌开始。第一步是转到[`developers.facebook.com/`](https://developers.facebook.com/)。

您应该看到您的应用程序列表，或者点击应用程序进行创建。您应该看到类似以下截图的内容：

![为会员创建社交登录](img/image_03_001.jpg)

您应该首先创建一个应用程序，并且可以通过访问应用程序创建页面来获取您的应用程序 ID 和应用程序密钥，类似于以下截图：

![为会员创建社交登录](img/image_03_002.jpg)

在创建新应用程序时，Facebook 现在包括了一种测试应用程序 ID 的方法。

它看起来像这样：

![为会员创建社交登录](img/image_03_003.jpg)

这是为了测试应用程序 ID 是否有效。这是可选的，您可以跳过这一步，只需将应用程序 ID 和应用程序密钥的值插入到前面截图中显示的代码中。

现在让我们创建`fbconfig.php`文件，其中将包含一种使用 Facebook SDK 库启用会话的方法。

`fbconfig.php`脚本将包含以下内容：

```php
<?php 
session_start(); 
$domain = 'http://www.socialexample.info'; 
require_once 'autoload.php'; 

use FacebookFacebookSession; 
use FacebookFacebookRedirectLoginHelper; 
use FacebookFacebookRequest; 
use FacebookFacebookResponse; 
use FacebookFacebookSDKException; 
use FacebookFacebookRequestException; 
use FacebookFacebookAuthorizationException; 
use FacebookGraphObject; 
use FacebookEntitiesAccessToken; 
use FacebookHttpClientsFacebookCurlHttpClient; 
use FacebookHttpClientsFacebookHttpable; 

// init app with app id and secret (get from creating an app) 
$fbAppId = '123456382121312313'; //change this. 
$fbAppSecret = '8563798aasdasdasdweqwe84'; 
FacebookSession::setDefaultApplication($fbAppId, $fbAppSecret); 
// login helper with redirect_uri 
    $helper = new FacebookRedirectLoginHelper($domain . '/fbconfig.php' ); 
try { 
  $session = $helper->getSessionFromRedirect(); 
} catch( FacebookRequestException $ex ) { 
echo "Hello, sorry but we've encountered an exception and could not log you in right now"; 
} catch( Exception $ex ) { 
  // Tell user something has happened 
  echo "Hello, sorry but we could not log you in right now";       
} 
// see if we have a session 
if ( isset( $session ) ) { 
  // graph api request for user data 
  $request = new FacebookRequest( $session, 'GET', '/me' ); 
  $response = $request->execute(); 
  // get response 
//start a graph object with the user email 
  $graphObject = $response->getGraphObject(); 
  $id = $graphObject->getProperty('id');  
  $fullname = $graphObject->getProperty('name');  
  $email = $graphObject->getProperty('email'); 

     $_SESSION['FB_id'] = $id;            
     $_SESSION['FB_fullname'] = $fullname; 
     $_SESSION['FB_email'] =  $email; 

//save user to session 
     $_SESSION['UserName'] = $email; //just for demonstration purposes 
//redirect user to index page        
    header("Location: index.php"); 
} else { 
  $loginUrl = $helper->getLoginUrl(); 
 header("Location: ".$loginUrl); 
} 
?> 

```

在这里，我们基本上通过`session_start()`开始一个会话，并通过将其保存到一个变量中设置我们网站的域。然后自动加载 FB SDK，这将需要 Facebook 访问其 API 所需的文件和类来访问。

然后，我们使用`use`关键字在其他 Facebook SDK 类上设置了几个依赖项。我们使用我们的应用程序 ID 和应用程序密钥设置了`facebookSession`类，然后尝试通过调用`getSessionfromRedirect()`方法启动会话。

如果有任何错误被捕获尝试启动会话，我们只需让用户知道我们无法登录他，但如果一切顺利进行，我们将以用户的电子邮件开始一个图形对象。

为了演示目的，我们保存一个用户名，实际上是用户的电子邮件地址，一旦我们通过 Facebook 图表获取了电子邮件。

无论如何，我们将通过检查他们的电子邮件地址对每个人进行身份验证，并且为了让用户更容易登录，让我们只将他们的电子邮件存储为用户名。

我们需要用`index.php`完成我们的网站，向用户展示我们网站内部的内容。我们在从 Facebook 页面登录后，将用户重定向到`index.php`页面。

现在我们将保持简单，并从登录的用户的 Facebook 个人资料中显示全名。我们将添加一个注销链接，以便用户有注销的选项：

```php
<?php 
session_start();  
?> 
<!doctype html> 
<html > 
  <head> 
    <title>Login to SocialNewsletter.com</title> 
<link href=" https://maxcdn.bootstrapcdn.com/bootstrap/3.3.6/css/bootstrap.min.css" rel="stylesheet">  
 </head> 
  <body> 
  <?php if ($_SESSION['FB_id']): ?>      <!--  After user login  --> 
<div class="container"> 
<div class="hero-unit"> 
  <h1>Hello <?php echo $_SESSION['UserName']; ?></h1> 
  <p>How to login with PHP</p> 
  </div> 
<div class="span4"> 
 <ul class="nav nav-list"> 
<li class="nav-header">FB ID: <?php echo $_SESSION['FB_id']; ?></li> 
<li> Welcome <?php echo $_SESSION['FB_fullName']; ?></li> 
<div><a href="logout.php">Logout</a></div> 
</ul></div></div> 
    <?php else: ?>     <!-- Before login -->  
<div class="container"> 
<h1>Login with Facebook</h1> 
           Not Connected with Facebook. 
<div> 
      <a href="fbconfig.php">Login with Facebook</a></div> 
      </div> 
    <?php endif ?> 
  </body> 
</html> 

```

登录后，我们只需为用户显示仪表板。我们将在下一节讨论如何为用户创建基本仪表板。

## 会员仪表板

最后，当会员登录我们的应用程序时，他们现在可以使用会员订阅页面订阅通讯。让我们首先构建用于存储会员详细信息和他们订阅的数据库。`member_details`表将包括以下内容：

+   `firstname`和`lastname`：用户的真实姓名

+   `email`：能够给用户发送电子邮件

+   `canNotify`：布尔值（true 或 false），如果他们同意通过电子邮件接收有关其他优惠的通知

### 提示

关于 MySQL 中的布尔类型有趣的是，当您创建使用布尔值（true 或 false）的字段时，MySQL 实际上只将其别名为`TINYINT(1)`。布尔基本上是 0 表示 false，1 表示 true。有关更多信息，请参阅[`dev.mysql.com/doc/refman/5.7/en/numeric-type-overview.html`](http://dev.mysql.com/doc/refman/5.7/en/numeric-type-overview.html)。

`member_details`表将处理此事，并将使用以下 SQL 代码创建：

```php
CREATE TABLE member_details(  
  id INT(11) PRIMARY KEY AUTO_INCREMENT, 
  firstname VARCHAR(255), 
  lastname VARCHAR(255), 
  email VARCHAR(255), 
  canNotify TINYINT(1), 
  member_id INT(11) 
); 

```

登录时，我们的会员将存储在`users`表中。让我们使用以下 SQL 代码创建它：

```php
CREATE TABLE users ( 
   id INT(11) PRIMARY KEY AUTO_INCREMENT 
   username VARCHAR(255), 
   password VARCHAR(255), 
); 

```

现在，构建一个视图，向我们的会员展示我们拥有的所有不同订阅。我们通过检查`subscriptions`表来实现这一点。`subscriptions`表模式定义如下：

+   `id Int(11)`：这是`subscriptions`表的主键，并设置为`AUTO_INCREMENT`

+   `newsletter_id Int(11)`：这是他们订阅的`newsletter_id`

+   `active BOOLEAN`：这表示用户当前是否订阅（默认为 1）

使用 SQL，它将如下所示：

```php
CREATE TABLE subscriptions ( 
  `id` INT(11) PRIMARY KEY AUTO_INCREMENT, 
  `newsletter_id` INT(11) NOT NULL, 
  `member_id` INT(11) NOT NULL, 
  `active` BOOLEAN DEFAULT true 
); 

```

我们还需要创建`newsletters`表，其中将以 JSON 格式保存所有通讯、它们的模板和内容。通过在我们的数据库中使用 JSON 作为存储格式，现在应该很容易从数据库中获取数据并将 JSON 解析为适当的值插入到我们的模板中。

由于我们的通讯将存储在数据库中，我们需要为其创建适当的 SQL 模式。设计如下：

+   `Id INT(11)`：在数据库中索引我们的通讯

+   `newsletter_name（文本）`：我们通讯的标题

+   `newsletter_count INT(11)`：记录我们特定通讯的版本

+   `Status（字符串）`：记录我们的通讯的状态，是否已发布、未发布或待发布

+   `Slug（字符串）`：能够在我们的社交通讯网站上使用浏览器查看通讯

+   `Template（文本）`：存储 HTML 模板

+   `Content（文本）`：存储将进入我们的 HTML 模板的数据

+   发布日期（日期）：记录发布日期

+   `Created_at（日期）`：记录通讯首次创建的时间

+   `Updated_at（日期）`：记录上次有人更新通讯的时间

其 SQL 如下：

```php
CREATE TABLE newsletters ( 
id INT(11) PRIMARY KEY AUTO_INCREMENT, 
newsletter_name (TEXT), 
newsletter_count INT(11) NOT NULL DEFAULT '0', 
marketer_id INT(11) NOT NULL, 
is_active TINYINT(1), 
created_at DATETIME, 

); 

```

当用户取消订阅时，这将帮助指示他们先前订阅了此通讯。这就是为什么我们将存储一个`active`字段，以便当他们取消订阅时，而不是删除记录，我们只需将其设置为 0。

`marketer_id`将在未来的管理部分中使用，其中我们提到将负责管理新闻通讯订阅的人员。

新闻通讯也可能有许多出版物，这些出版物将是实际发送给每个订阅的新闻通讯。以下 SQL 代码是用来创建出版物的：

```php
CREATE TABLE publications ( 
  newsleterId INT(11) PRIMARY KEY AUTO_INCREMENT, 
  status VARCHAR(25), 
  content TEXT, 
  template TEXT, 
  sent_at DATETIME, 
  created_at DATETIME, 
); 

```

现在让我们在我们的`Newsletter`类中构建方法，以选择已登录会员的订阅以显示到我们的仪表板：

```php
Class Dashboard { 
  public function getSubscriptions($member_id) { 
  $query = $db->query("SELECT * FROM subscriptions, newsletters WHERE subscriptions.member_id ='". $member_id."'"); 
  if($query->num_rows() > 0) { 
      while ($row = $result->fetch_assoc()) { 
          $data  = array(  
            'name' => $row->newsletter_name,  
            'count' => $row->newsletter_count, 
            'mem_id' => $row->member_id,  
            'active' => $row->active 
         ); 
      } 
      return $data; 
  }  
} 
} 

```

从上述代码中，我们只是创建了一个函数，用于获取给定会员 ID 的订阅。首先，我们创建了`"SELECT * FROM subscriptions, newsletters WHERE subscriptions.member_id ='". $member_id."`查询。之后，我们使用 MySQLi 结果对象的`fetch_assoc()`方法循环遍历查询结果。现在我们已经将其存储在`$data`变量中，我们返回该变量，并在以下代码中通过调用以下函数在表中显示数据：

```php
 $member_id = $_SESSION['member_id']; 
 $dashboard = new Dashboard; 
 $member_subscriptions = $dashboard->getSubscriptions($member_id); 
 ?> 
  <table> 
    <tr> 
      <td>Member Id</td><td>Newsletter Name</td><td>Newsletter count</td><td>Active</td> 
     </tr> 
<?php 
 foreach($member_subscriptions as $subs) { 
    echo '<tr> 
     <td>'. $subs['mem_id'] . '</td>' .  
     '<td>' . $subs['name'].'</td>' .  
     '<td>' . $subs['count'] . '</td>'. 
     '<td>' . $subs['active'] . '</td> 
     </tr>'; 
 } 
 echo '</table>'; 

```

## 营销人员仪表盘

我们的营销人员，他们管理自己拥有的每个新闻通讯，将能够登录到我们的系统，并能够看到有多少会员订阅了他们的电子邮件地址。

这将是一个管理员系统，使营销人员能够更新会员记录，查看最近的订阅，并允许营销人员向其新闻通讯的任何会员发送自定义电子邮件。

我们将有一个名为`marketers`的表，其中将包含以下字段：

+   `id`：用于存储索引

+   营销人员的姓名：用于存储营销人员的姓名

+   营销人员的电子邮件：用于存储营销人员的电子邮件地址

+   营销人员的密码：用于存储营销人员的登录密码

我们用于创建上述字段的 SQL 很简单：

```php
CREATE TABLE marketers ( 
id INT(11) AUTO_INCREMENT, 
marketer_name VARCHAR(255) NOT NULL, 
marketer_email VARCHAR(255) NOT NULL, 
marketer_password VARCHAR(255) NOT NULL, 

PRIMARY KEY `id`  
); 

```

在另一个表中，我们将定义营销人员及其管理的新闻通讯的多对多关系。

我们需要一个`id`作为索引，拥有新闻通讯的营销人员的 ID，以及营销人员拥有的新闻通讯的 ID。

创建此表的 SQL 如下：

```php
CREATE TABLE newsletter_admins ( 
  Id INT(11) AUTO_INCREMENT, 
  marketer_id INT(11) , 
  newsletter_id INT(11), 
  PRIMARY KEY `id`, 
); 

```

现在让我们构建一个查询，以获取他们拥有的新闻通讯的管理员。这将是一个简单的类，我们将引用所有我们的数据库函数：

```php
<?php  
class NewsletterDb { 
public $db; 

function __construct($dbinstance) { 
$this->db = $dbinstance; 
} 

//get admins = marketers 
public function get_admins ($newsletter_id) { 
$query = "SELECT * FROM newsletter_admins LEFT JOIN marketers ON marketers.id = newsletter_admins.admin_id.WHERE newsletters_admins.newsletter_id = '".$newsletter_id."'"; 
  $this->db->query($query); 
} 
} 

```

### 管理营销人员的管理系统

我们需要一种方法让营销人员登录并通过密码进行身份验证。我们需要一种方法让管理员创建帐户并注册营销人员及其新闻通讯。

让我们首先构建这部分。

在我们的管理视图中，我们将需要设置一个默认值，并要求对执行的每个操作进行身份验证密码。这是我们不需要存储在数据库中的东西，因为只会有一个管理员。

在我们的`config`/`admin.php`文件中，我们将定义用户名和密码如下：

```php

<?php 
$admin_username = 'admin'; 
$password = 'test1234'; 
?> 

```

然后我们只需在我们的登录页面`login.php`中包含文件。我们将简单地检查它。登录页面的代码如下：

```php
<html> 
<?php  
if(isset($_POST['username']) && isset($_POST['password'])) { 
  //check if they match then login 
  if($_POST['username'] == $admin_username  
    && $_POST['password'] == $password) { 
   //create session and login 
   $_SESSION['logged_in'] = true; 
   $_SESSION['logged_in_user'] = $admin_username; 
      header('http://ourwebsite.com/admin/welcome_dashboard.php'); 
 } 
 ?> 
} 
</html> 

```

请注意，我们必须根据我们正在开发的位置正确设置我们的网站 URL。在上面的示例中，页面将在登录后重定向到[`ourwebsite.com/admin/welcome_dashboard.php`](http://ourwebsite.com/admin/welcome_dashboard.php)。我们可以创建变量来存储域和要重定向到的 URL 片段，以便这可以是动态的；请参阅以下示例：

```php
$domain = 'http://ourwebsite.com'; 
$redirect_url = '/admin/welcome_dashboard.php'; 
header($domain . $redirect_url); 

```

一旦登录，我们将需要构建一个简单的 CRUD（创建、读取、更新、删除）系统来管理将管理他们的新闻通讯的营销人员。

以下是能够获取营销人员和他们管理的新闻通讯列表的代码：

```php
Function get_neewsletter_marketers() { 
  $q = "SELECT * FROM marketers LEFT JOIN newsletters '; 
  $q .= "WHERE marketers.id = newsletters.marketer_id"; 

  $res = $db->query($q); 

  while ($row = $res->fetch_assoc()) { 
   $marketers = array( 
     'name' => $row['marketer_name'], 
     'email' => $row['marketer_email'], 
     'id' => $row['marketer_id'] 
    ); 
  } 
  return $marketers; 
} 

```

我们需要添加一种方法来编辑、创建和删除营销人员。让我们创建一个`dashboard`/`table_header.php`来包含在我们脚本的顶部。

以下是`table_header.php`代码的样子：

```php
<table> 
<tr> 
 <th>Marketer Email</th> 
  <th>Edit</th> 
 <th>Delete</th> 
</tr> 

```

现在我们将创建一个`for()`循环来循环遍历每个营销人员。让我们创建一种方法来选择我们数据库中的所有营销人员。首先，让我们调用我们的函数来获取数据：

```php
$marketrs = get_newsletter_marketers(); 

```

然后让我们使用`foreach()`循环来循环遍历所有营销人员：

```php
foreach($marketers as $marketer) { 
  echo '<tr><td>'. $marketer['email'] .'</td> 
   <td><a href="edit_marketer.php?id='. $marketer['id'].'">Edit</a></td> 
  <td><a href="delete_marketer.php">delete</td> 
  </tr>'; 
} 
echo '</table>'; 

```

然后我们用`</table>`为表结束代码。

让我们创建`delete_marketer.php`脚本和`edit_marketer.php`脚本。以下将是删除脚本：

```php
function delete_marketer($marketer_id) { 
  $q = "DELETE FROM marketers WHERE marketers.id = '" .   $marketer_id . "'"; 
   $this->db->query($q); 
} 
$marketer_id = $_GET['id']; 
delete_marketer($marketer_id); 

```

这是由一个表单组成的编辑脚本，一旦提交将更新数据：

```php
if(empty($_POST['submit'])) { 
  $marketer_id = $_GET['id']; 
  $q = "SELECT * FROM marketers WHERE id = '" . $marketer_id."'"; 

 $res = $db->query($q); 

  while ($row = $res->fetch_assoc()) { 
   $marketer = array( 
     'name' => $row['marketer_name'], 
     'email' => $row['marketer_email'], 
     'id' => $row['id'] 
    ); 
  } 

  ?> 
  <form action="update_marketer.php" method="post"> 
   <input type="hidden" name="marketer_id" value="<?php echo $marketer['id'] ?>"> 
   <input type="text" name="marketer_name" value="<?php echo $marketer['name'] ?>"> 
   <input type="text" name="marketer_email" value="<?php echo $marketer['email'] ?>"> 
  <input type="submit" name="submit" /> 
</form> 
  <?php 

  } else { 
     $q = "UPDATE marketers SET marketer_name='" . $_POST['marketer_name'] . ", marketer_email = '". $_POST['marketer_email']."' WHERE id = '".$_POST['marketer_id']."'"; 
   $this->db->query($q); 
   echo "Marketer's details has been updated"; 
  } 
?> 

```

## 我们的通讯的自定义模板

每个营销人员都需要制定他们的通讯。在我们的情况下，我们可以允许他们创建一个简单的侧边栏通讯和一个简单的自上而下的通讯。为了构建一个简单的侧边栏，我们可以创建一个 HTML 模板，看起来像下面这样：

```php
<html> 
<!doctype html> 

<sidebar style="text-align:left"> 
{{MENU}} 
</sidebar> 

<main style="text-align:right"> 
   {{CONTENT}} 
</main> 
</html> 

```

在之前的代码中，我们使用内联标签样式化 HTML 电子邮件，因为一些电子邮件客户端不会渲染从我们 HTML 外部引用的样式表。

我们可以使用**正则表达式**来替换`{{MENU}}`和`{{CONTENT}}`模式，以填充数据。

我们的数据库将以 JSON 格式存储内容，一旦解析 JSON，我们将得到内容和菜单数据，然后插入到它们各自的位置。

在我们的数据库中，我们需要添加`newsletter_templates`表。以下是我们将如何创建它：

```php
CREATE TABLE newsletter_templates ( 
 Id INT(11) PRIMARY KEY AUTO_INCREMENT, 
Newsletter_id INT(11) NOT NULL, 
   Template TEXT NOT NULL, 
   Created_by INT(11) NOT NULL   
) ENGINE=InnoDB; 

```

有了模板之后，我们需要一种方法让营销人员更新模板。

从仪表板上，我们显示通讯的模板列表。

让我们按照以下方式创建表单：

```php
$cleanhtml = htmlentities('<html> 
<!doctype html> 

<sidebar style="text-align:left"> 
{{MENU}} 
</sidebar> 

<main style="text-align:right"> 
   {{CONTENT}} 
</main> 
</html> 
'); 
<form> 
   <h2>Newsletter Custom Template</h2> 
  <textarea name="customtemplate"> 
<?php echo $cleanhtml; ?> 
</textarea> 
  <input type="submit" value="Save Template" name="submit"> 
  </form> 

```

我们还通过向`textarea`添加值来填充它。请注意，在之前的代码中，我们需要首先使用`htmlentities`清理模板的 HTML 代码。这是因为我们的 HTML 可能被解释为网页的一部分，并在浏览器渲染时引起问题。

我们现在已经准备好发送实际的通讯了。为了发送通讯，我们需要创建一个脚本，循环遍历通讯中的所有成员，然后简单地使用 PHP 邮件功能发送给他们。

使用 PHP 邮件功能，我们只需要循环遍历我们数据库中的所有通讯成员。

这就是那个脚本的样子：

```php
$template = require('template.class.php'); 
$q = "SELECT * FROM newsletter_members WHERE newsletter_id = 1"; //if we're going to mail newsletter #1  
$results = $db->query($q); 
While ($rows =$results->fetch_assoc() ) { 
  //gather data  
  $newsletter_title = $row['title']; 
  $member_email = $row['template']; 
  $menu = $row['menu']; //this is a new field to contain any menu html 
  $content = $row['content']; 
  $content_with_menu = $template->replace_menu($menu, $content); 
  $emailcontent = $template->         replace_contents($content,$content_with_menu); 
  //mail away! 
  mail($member_email, 'info@maillist.com', $newsletter_title ,$email_content); 
} 

```

我们需要完成`replace_menu`和`replace_contents`函数。让我们简单地构建文本替换函数，用于替换我们在之前的代码中已经获取的内容。数据来自数据库中的通讯表：

```php
class Template { 
   public function replace_menu($menu, $content) { 
     return  str_replace('{{MENU}}', $menu, $content); 
   } 
   public function replace_contents ($actualcontent, $content) { 
    return str_replace('{{CONTENT}}', $actualcontent,  $content); 
   }  
} 

```

请注意，我们修改了我们的表，为通讯中添加了菜单。这个菜单必须由用户创建，并使用 HTML 标记。它基本上是一个 HTML 链接列表。菜单的正确标记应该如下所示：

```php
<ul> 
  <li><a href="http://someUrl.com">some URL</a></li> 
<li><a href="http://someNewUrl.com">some new URL</a></li> 
<li><a href="http://someOtherUrl.com">some other URL</a></li> 
</ul> 

```

## 链接跟踪

对于我们的链接跟踪系统，我们需要允许营销人员嵌入链接，实际上通过我们的系统传递，以便我们跟踪链接的点击次数。

我们将创建一个服务，自动将我们输入的链接缩短为随机哈希。URL 看起来像`http://example.com/link/xyz123`，哈希`xyz123`将存储在我们的数据库中。当用户访问链接时，我们将匹配链接。

让我们创建链接表，并创建一个函数来帮助我们生成缩短链接。至少，我们需要能够存储链接的标题、实际链接、缩短链接，以及创建链接的人，以便我们可以将其放在营销人员的仪表板上。

链接表的 SQL 如下所示：

```php
CREATE TABLE links ( 
   id INT(11) PRIMARY KEY AUTO_INCREMENT, 
   link_title TEXT NOT NULL, 
   actual_link TEXT, 
   shortened_link VARCHAR(255), 
   created DATETIME, 
   created_by INT(11) 
); 

```

现在让我们创建以下函数，它将生成一个随机哈希：

```php
public function createShortLink($site_url,$title, $actual_url,$created_by) { 
    $created_date = date('Y-m-d H:i:s'); 
  $new_url = $site_url . "h?=" . md5($actual_url); 
  $res = $this->db->query("INSERT INTO links VALUES (null, $title ,'". $actual_url. "', '". $new_url.", '". $created_date."','".$created_by."'"),; 
  )); 
   return $res; 
} 

```

我们还需要存储链接的点击次数。我们将使用另一个表，将`link_id`链接到点击次数，每当有人使用缩短链接时，我们将更新该表：

```php
CREATE TABLE link_hits ( 
   link_id INT(11), 
   num_hits INT(11) 
); 

```

我们不需要对之前的 SQL 表进行索引，因为我们不需要在其上进行快速搜索。每次生成新的 URL 时，我们应该将表填充为`num`默认为 0：

在`createShortLink`函数中添加以下函数：

```php
$res = $this->db->query("INSERT INTO links VALUES (null, '$actual_url',$title, '$new_url', '$created_date', '$created_by'"); 

$new_insert_id = $this->db->insert_id; 

$dbquery = INSERT INTO link_hits VALUES($new_insert_id,0); 

$this->db->query($dbquery); 

```

`insert_id`是 MySQL 最后插入记录的 ID。它是一个函数，每次添加新行时都会返回新生成的 ID。

让我们生成包含两个函数的链接点击类，一个用于初始化数据库，另一个用于在用户点击链接时更新`link_hits`表：

```php
Class LinkHit {       

     Public function __construct($mysqli) { 
          $this->db = $mysqli; 
      } 

   public function  hitUpdate ($link_id) { 

  $query = "UPDATE link_hits SET num_hits++ WHERE link_id='".    $link_id. "'"; 

   //able to update 
     $this->db->query($query)       
   } 

   Public function checkHit ($shorturl) { 
   $arrayUrl = parse_url($shortUrl); 
parse_str($parts['query'],$query); 
$hash = $query['h'];  

   $testQuery = $this->db->query("SELECT id FROM links WHERE shortened_link LIKE '%$hash%'"); 
   if($this->db->num_rows > 0) { 
         while($row = $testQuery->fetch_array() ) { 
   return $row['id']; 
          } 
   } else { 
     echo "Could not find shorted link"; 
     return null; 
  } 
} 

//instantiating the function: 
$mysqli = new mysqli('localhost','test_user','test_password','your_database'); 
$Link = new LinkHit($mysqli); 
$short_link_id = $Link->checkHit("http://$_SERVER[HTTP_HOST]$_SERVER[REQUEST_URI]"); 

if($short_link_id !== null) { 
  $link->hitUpdate($isShort); 
} 

```

为了让我们的营销人员查看链接，我们需要在我们的门户网站上的`links`页面上显示他们的链接。

我们创建用于检查链接及其点击次数的功能，这是归因于已登录的管理员用户：

```php
$user_id = $_SESSION['user_id']; 
$sql = "SELECT * FROM links LEFT JOIN link_hits ON links.id = link_hits.link_id WHERE links.created_by='" . $user_id. "'"; 
$query = $mysqli->query($sql); 
?> 
<table> 
<tr> 
<td>Link id</td><td>Link hits</td></tr> 
<?php 
while($obj = $query->fetch_object()) { 
  echo '<tr><td>'.$obj->link.'</td> 
<td>' . $obj->link_hits.'</td></tr></tr>'; 
} 
?> 
</table> 

```

在上述代码中，我们只是通过检查变量`$_SESSION['user_id']`获取了已登录用户的 ID。然后我们通过执行字符串变量`$SQL`执行了一个 SQL 查询。之后，我们循环遍历结果，并将结果显示在 HTML 表中。请注意，当我们显示永久的 HTML 标记时，例如表的开头、标题和`</table>`标记的结束时，我们退出 PHP 代码。

PHP 在不使用 echo 语句时性能略有提高，这就是 PHP 脚本的美妙之处，您真的可以进入 PHP 部分，然后进入代码中的 HTML 部分。您对这个想法的美感可能有所不同，但我们只是想展示 PHP 在这个练习中可以做什么。

## 支持的 AJAX 套接字聊天

该系统允许订阅者联系特定通讯组的管理员。它将只包含一个联系表单。此外，我们需要实现一种实时向管理员发送通知的方式。

我们将基本上为管理员添加一个套接字连接，以便每当有人发送查询时，它将在营销人员的仪表板上闪烁通知。

这对于**socket.io**和一个名为 WebSockets 的浏览器技术来说非常简单。

### socket.io 简介

使用 socket.io，我们不需要创建用于定期检查服务器是否有事件的代码。我们只需通过 AJAX 传递用户输入的数据，并通过发出事件来触发套接字的监听器。它提供了长轮询和通过 WebSockets 进行通信，并得到了现代 Web 浏览器的支持。

### 注意

WebSockets 扩展了通过浏览器建立套接字连接的概念。要了解有关 WebSockets 的更多信息，请访问[`www.html5rocks.com/en/tutorials/websockets/basics/`](http://www.html5rocks.com/en/tutorials/websockets/basics/)。

socket.io 网站上的示例代码只包括`socket.io.js`脚本：

```php
<script src="socket.io/socket.io.js"></script> 

```

我们的 PHP Web 服务器将使用一个名为**Ratchet**的东西，它在[`socketo.me`](http://socketo.me)上有一个网站。它基本上允许我们为 PHP 使用 WebSockets。

这是他们的网站：

![socket.io 简介](img/image_03_004.jpg)

Ratchet 只是一个工具，允许 PHP 开发人员“*在 WebSockets 上创建实时的、双向的应用程序*”。通过创建双向数据流，它允许开发人员创建实时聊天和其他实时应用程序等东西。

让我们通过按照他们在[`socketo.me/docs/hello-world`](http://socketo.me/docs/hello-world)上的教程开始。

使用 Ratchet，我们需要安装**Composer**并将以下内容添加到我们项目目录中的`composer.json`文件中：

```php
{ 
    "autoload": { 
        "psr-0": { 
            "MyApp": "src" 
        } 
    }, 
    "require": { 
        "cboden/ratchet": "0.3.*" 
    } 
} 

```

如果您之前有使用 Composer 的经验，基本上它所做的就是在编写需要自动加载的脚本的路径时使用`psr-0`标准。然后我们在同一目录中运行`composer install`。在设置好 Ratchet 之后，我们需要设置处理某些事件的适当组件。

我们需要创建一个名为`SupportChat`的文件夹，并将`Chat.php`放在其中。这是因为在之前的`composer.json`文件中使用 psr-0 时，它期望`src`目录内有一个目录结构。

让我们创建一个包含我们需要实现的存根函数的类：

```php
namespace SupportChat; 
use Ratchet\MessageComponentInterface; 
use Ratchet\ConnectionInterface; 

class SupportChat implements MessageComponentInterface { 
  Protected $clients; 
  Public function __construct() { 
    $this->clients = new \SplObjectStorage; 
  } 
} 

```

我们需要声明`$clients`变量来存储将连接到我们聊天应用程序的客户端。

让我们实现客户端打开连接时的接口：

```php
Public function onOpen(ConnectionInterface $conn) { 
  $this->clients->attach($conn); 
  echo "A connection has been established"; 
} 

```

现在让我们创建`onMessage`和`onClose`方法如下：

```php
Public function onMessage (ConnectionInterface $from, $msg) { 
 foreach ($this->clients as $client) { 
        if ($from !== $client) { 
            $client->send($msg); 
        } 
    } 
} 

public function onClose(ConnectionInterface $conn) { 
$this->clients->detach($conn); 
} 

```

让我们也创建一个用于处理错误的`onError`方法如下：

```php
public function onError (ConnectionInterface $conn) { 
$this->clients->detach($conn); 
} 

```

现在我们需要实现应用程序的客户端（浏览器）部分。

在您的`htdocs`或`public`文件夹中创建一个名为`app.js`的文件，其中包含以下代码：

```php
var messages = []; 

// connect to the socket server 
var conn = new WebSocket('ws://localhost:8088'); 
conn.onopen = function(e) { 
   console.log('Connected to server:', conn); 
} 

conn.onerror = function(e) { 
   console.log('Error: Could not connect to server.'); 
} 

conn.onclose = function(e) { 
   console.log('Connection closed'); 
} 

// handle new message received from the socket server 
conn.onmessage = function(e) { 
   // message is data property of event object 
   var message = JSON.parse(e.data); 
   console.log('message', message); 

   // add to message list 
   var li = '<li>' + message.text + '</li>'; 
   $('.message-list').append(li); 
} 

// attach onSubmit handler to the form 
$(function() { 
   $('.message-form').on('submit', function(e) { 
         // prevent form submission which causes page reload 
         e.preventDefault(); 

         // get the input 
         var input = $(this).find('input'); 

         // get message text from the input 
         var message = { 
               type: 'message', 
               text: input.val() 
         }; 

         // clear the input 
         input.val(''); 

         // send message to server 
         conn.send(JSON.stringify(message)); 
   }); 
}); 

```

我们需要创建用于上述代码的 HTML。我们应该将文件命名为`app.js`。现在，让我们实现一个简单的输入文本，让用户输入他们的消息：

```php
<!DOCTYPE html> 
<html> 
<head> 
   <title>Chat with Support</title> 
   <script src="https://cdnjs.cloudflare.com/ajax/libs/jquery/2.2.3/jquery.js"></script> 
   <script src="app.js"></script> 
</head> 
<body> 

   <h1>Chat with Support</h1> 

   <h2>Messages</h2> 
   <ul class="message-list"></ul> 
   <form class="message-form"> 
         <input type="text" size="40" placeholder="Type your message here" /> 
         <button>Send message</button> 
   </form> 
</body> 
</html> 

```

`App.js`是我们之前编写的 JavaScript 代码应该放置的地方。我们还需要创建一个 WebSocket 服务器来处理端口`8088`上的 WebSocket：

```php

<?php 
// import namespaces 
use Ratchet\Server\IoServer; 
use Ratchet\WebSocket\WsServer; 
use SupportChat\Chat; 

// use the autoloader provided by Composer 
require dirname(__DIR__) . '/vendor/autoload.php'; 

// create a websocket server 
$server = IoServer::factory( 
    new WsServer( 
        new Chat() 
    ) 
    , 8088 
); 

$server->run(); 

```

我们的聊天应用现在已经准备好供公众使用。但是，我们需要启动我们的聊天服务器，通过`php bin/server.php`启动它来处理 WebSockets。

请注意，在 Windows 上，它会提示有关正在使用的网络：

![socket.io 简介](img/image_03_005.jpg)

只需单击**允许访问**，然后单击**确定**。

现在当我们访问`http://localhost/client.html`时，我们应该看到以下内容：

![socket.io 简介](img/image_03_006.jpg)

但是，我们需要通过添加用户名和电子邮件来改进联系表单，以便支持人员在没有支持人员回复用户的情况下通过电子邮件回复他。

我们的表单现在如下所示：

```php
<form class="message-form" id="chatform"> 
         <input type="text" name="firstname" size="25" placeholder="Your Name"> 
         <input type="text" name="email" size="25" placeholder="Email"> 

         <input type="text" name="message" size="40" placeholder="Type your message here" /> 
         <button>Send message</button> 
   </form> 

```

由于我们添加了这些细节，我们需要将它们存储在我们的数据库中。我们可以通过将所有数据转发到另一个 PHP 脚本来执行发送。在 JavaScript 中，代码将向处理程序添加一种从表单发送到`sendsupportmessage.php`的方式。

以下是使用 jQuery 编写的 JavaScript 代码的样子：

```php
<script> 
$(document).ready(function() { 
   $('submit').on('click', function() { 
     $.post('sendsupportmessage.php', $("#chatform").serialize()) 
       .done(function(data) { 
         alert('Your message has been sent'); 
      }); 
   }); 
}); 
</script> 

```

在将接收消息的脚本`sendsupportmessage.php`中，我们需要解析信息并创建一封发送到支持电子邮件`contact@yoursite.com`的电子邮件；请参考以下示例：

```php
<?php 
  if( !empty($_POST['message'])) { 
    $message = htmlentities($_POST['message']); 
  } 

  if( !empty($_POST['email'])) { 
    $email = htmlentities($_POST['email']); 
  } 

  if( !empty($_POST['firstname']) ) { 
    $firstname = htmlentities($_POST['firstname']); 
  }  

  $emailmessage = 'A support message from ' . $firstname . '; 
  $emailmessage .=  ' with email address: ' . $email . '; 
  $emailmessage .= ' has been received. The message is '. $message; 

   mail('contact@yoursite.com', 'Support message', $emailmessage);  

  echo "success!"; 
?> 

```

该脚本只是检查提交的值是否不为空。根据经验，使用`!empty()`而不是使用`isset()`函数检查设置的值更好，因为 PHP 可能会将空字符串（''）评估为已设置：

```php
$foo = ''; 
if(isset($foo)) { print 'But no its empty'; } 
else { print 'PHP7 rocks!'; } 

```

现在我们需要向用户显示，因为我们使用 AJAX 将消息发送到服务器，并更新 AJAX 框。在 JavaScript 代码中，我们应该将`.done()`回调代码更改为以下内容：

```php
.done(function(data) { 
   if(data === 'succcess!') { 
     var successHtml = '<li>Your message was sent</li>'; 
     $('.message-list').append(successHtml); 

   } 
      } 

```

太棒了！请注意，我们更改了警报框的调用，而是将消息`您的消息已发送`附加回消息列表中。我们的支持表单现在发送了消息的发送者，并且我们的支持团队可以在他们的电子邮件中收到消息。

# 总结

在本章中，您学到了很多。总之，我们建立了一个简单的管理系统来管理我们的营销人员。此外，我们还为新闻通讯的成员创建了一个登录方式，这将引导用户到主页。

然后我们回顾了如何使用简单的模板系统发送电子邮件，这允许用户添加自己的菜单和内容到布局中。我们还能够使用 Facebook PHP SDK 和其认证过程添加 Facebook 社交登录。

在本章的后半部分，我们建立了一个简单的聊天系统，它将立即发送电子邮件到我们网站的支持电子邮件地址。我们查看了 Ratchet，这是一个 PHP 库，可以帮助我们在 PHP 中处理实时消息，并使用 AJAX 异步发送数据到另一个将发送电子邮件到支持电子邮件的脚本。

我们现在已经创建了一个令人印象深刻的新闻通讯应用程序，它不仅具有社交登录功能和支持聊天框，还允许其他新闻通讯营销人员通过网站管理其内容。
