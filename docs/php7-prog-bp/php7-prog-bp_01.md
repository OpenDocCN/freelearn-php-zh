# 第一章：创建用户配置文件系统并使用空合并运算符

为了开始这一章，让我们来看看 PHP 7 中的新**空合并**。我们还将学习如何构建一个简单的配置文件页面，其中列出了可以单击的用户，并创建一个简单的类似 CRUD 的系统，这将使我们能够注册新用户到系统中，并删除用户以进行封禁。

我们将学习使用 PHP 7 空合并运算符，以便我们可以在有数据时显示数据，或者如果没有数据，只显示一个简单的消息。

让我们创建一个简单的`UserProfile`类。自 PHP 5 以来，创建类的能力就已经可用了。

PHP 中的一个类以`class`开头，后面是类的名称：

```php
class UserProfile { 

  private $table = 'user_profiles'; 

} 

} 

```

我们已经将表格设为私有，并添加了一个`private`变量，我们在其中定义它将与哪个表相关联。

让我们添加两个函数，也称为类中的方法，来简单地从数据库中获取数据：

```php
function fetch_one($id) { 
  $link = mysqli_connect(''); 
  $query = "SELECT * from ". $this->table . " WHERE `id` =' " .  $id "'"; 
  $results = mysqli_query($link, $query); 
} 

function fetch_all() { 
  $link = mysqli_connect('127.0.0.1', 'root','apassword','my_dataabase' ); 
  $query = "SELECT * from ". $this->table . "; 
 $results = mysqli_query($link, $query); 
} 

```

# 空合并运算符

我们可以使用 PHP 7 的空合并运算符来允许我们检查我们的结果是否包含任何内容，或者返回一个我们可以在视图上检查的定义文本，这将负责显示任何数据。

让我们把这个放在一个文件中，其中包含所有的定义语句，并称之为：

```php
//definitions.php 
define('NO_RESULTS_MESSAGE', 'No results found'); 

require('definitions.php'); 
function fetch_all() { 
   ...same lines ... 

   $results = $results ??  NO_RESULTS_MESSAGE; 
   return $message;    
} 

```

在客户端，我们需要设计一个模板来显示用户配置文件的列表。

让我们创建一个基本的 HTML 块，以显示每个配置文件可以是一个`div`元素，其中包含几个列表项元素来输出每个表。

在下面的函数中，我们需要确保所有的值都至少填写了姓名和年龄。然后当函数被调用时，我们只需返回整个字符串：

```php
function profile_template( $name, $age, $country ) { 
 $name = $name ?? null; 
  $age = $age ?? null; 
  if($name == null || $age === null) { 
    return 'Name or Age need to be set';  
   } else { 

    return '<div> 

         <li>Name: ' . $name . ' </li> 

         <li>Age: ' . $age . '</li> 

         <li>Country:  ' .  $country . ' </li> 

    </div>'; 
  } 
} 

```

# 关注点分离

在一个适当的 MVC 架构中，我们需要将视图与获取数据的模型分开，控制器将负责处理业务逻辑。

在我们的简单应用程序中，我们将跳过控制器层，因为我们只想在一个公共页面中显示用户配置文件。前面的函数也被称为 MVC 架构中的模板渲染部分。

虽然有一些可用于 PHP 的框架可以直接使用 MVC 架构，但现在我们可以坚持我们已经拥有的东西并使其工作。

PHP 框架可以从空合并运算符中受益很多。在我曾经使用的一些代码中，我们经常使用三元运算符，但仍然需要添加更多的检查来确保值不是虚假的。

此外，三元运算符可能会令人困惑，并需要一些时间来适应。另一种选择是使用`isSet`函数。然而，由于`isSet`函数的性质，一些虚假的值将被 PHP 解释为已设置。

# 创建视图

现在我们的模型已经完成，有一个模板渲染函数，我们只需要创建一个视图，通过它我们可以查看每个配置文件。

我们的视图将放在一个`foreach`块中，并且我们将使用我们编写的模板来渲染正确的值：

```php
//listprofiles.php 

<html> 
<!doctype html> 
<head> 
<link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.6/css/bootstrap.min.css"> 
</head> 
<body> 

<?php 
foreach($results as $item) { 
  echo profile_template($item->name, $item->age, $item->country; 
} 
?> 
</body> 
</html> 

```

让我们把上面的代码放到`index.php`中。

虽然我们可以安装 Apache 服务器，配置它来运行 PHP，安装新的虚拟主机和其他必要的功能，并将我们的 PHP 代码放入 Apache 文件夹中，但这需要时间。因此，为了测试这一点，我们可以只运行 PHP 的服务器进行开发。

要运行内置的 PHP 服务器（在[`php.net/manual/en/features.commandline.webserver.php`](http://php.net/manual/en/features.commandline.webserver.php)上阅读更多信息），我们将使用我们正在运行的文件夹，在终端内：

```php
**php -S localhost:8000**

```

如果我们打开浏览器，我们应该还看不到任何东西，**没有找到结果**。这意味着我们需要填充我们的数据库。

如果您的数据库连接出现错误，请确保将我们提供的正确数据库凭据替换到我们所做的每个`mysql_connect`调用中。

1.  为了向我们的数据库提供数据，我们可以创建一个简单的 SQL 脚本，就像这样：

```php
INSERT INTO user_profiles ('Chin Wu', 30, 'Mongolia'); 
INSERT INTO user_profiles ('Erik Schmidt', 22, 'Germany'); 
INSERT INTO user_profiles ('Rashma Naru', 33, 'India'); 

```

1.  让我们把它保存在一个名为`insert_profiles.sql`的文件中。在与 SQL 文件相同的目录中，通过以下命令登录 MySQL 客户端：

```php
 **mysql -u root -p**

```

1.  然后输入使用<数据库名称>：

```php
 **mysql>  use <database>;**

```

1.  通过运行 source 命令导入脚本：

```php
 **mysql> source insert_profiles.sql**

```

现在我们的用户资料页面应该显示如下：

![创建视图](img/image_01_001.jpg)

# 创建个人资料输入表单

现在让我们为用户创建 HTML 表单来输入他们的个人资料。

如果我们没有一个简单的方法让用户输入他们的用户资料细节，我们的个人资料应用就没有用处。

我们将创建个人资料输入表单如下：

```php
//create_profile.php 

<html> 
<body> 
<form action="post_profile.php" method="POST"> 

  <label>Name</label><input name="name"> 
  <label>Age</label><input name="age"> 
  <label>Country</label><input name="country"> 

</form> 
</body> 
</html> 

```

在这个个人资料帖子中，我们需要创建一个 PHP 脚本来处理用户发布的任何内容。它将从输入值创建一个 SQL 语句，并输出它们是否被插入。

我们可以再次使用空合并运算符来验证用户是否输入了所有值，并且没有留下未定义或空的值：

```php
$name = $_POST['name'] ?? ""; 

$age = $_POST['country'] ?? ""; 

$country = $_POST['country'] ?? ""; 

```

这可以防止我们在向数据库插入数据时累积错误。

首先，让我们创建一个变量来保存每个输入的数组：

```php
$input_values =  [ 
 'name' => $name, 
 'age' => $age, 
 'country' => $country 
]; 

```

上面的代码是一种新的 PHP 5.4+写数组的方式。在 PHP 5.4+中，不再需要使用实际的`array()`；作者个人更喜欢新的语法。

我们应该在我们的`UserProfile`类中创建一个新的方法来接受这些值：

```php
Class UserProfile { 

 public function insert_profile($values)  { 

 $link =  mysqli_connect('127.0.0.1', 'username','password', 'databasename'); 

 $q = " INSERT INTO " . $this->table . " VALUES ( '".$values['name']."', '".$values['age'] . "' ,'".$values['country']. "')"; 
   return mysqli_query($q); 

 } 
} 

```

与我们的个人资料模板渲染函数一样，我们不需要在函数中创建一个参数来保存每个参数，我们可以简单地使用一个数组来保存我们的值。

这样，如果需要向我们的数据库插入一个新字段，我们只需在 SQL `insert`语句中添加另一个字段。

趁热打铁，让我们创建编辑个人资料部分。

目前，我们假设使用此编辑个人资料的人是网站的管理员。

我们需要创建一个页面，假设`$_GET['id']`已经设置，那么我们将从数据库中获取用户并在表单上显示。代码如下：

```php
<?php 
require('class/userprofile.php');//contains the class UserProfile into 

$id = $_GET['id'] ?? 'No ID'; 
//if id was a string, i.e. "No ID", this would go into the if block 
if(is_numeric($id)) { 
  $profile =  new UserProfile(); 
  //get data from our database 
  $results =   $user->fetch_id($id); 
  if($results && $results->num_rows > 0  ) { 
     while($obj = $results->fetch_object()) 
   { 
          $name = $obj->name; 
          $age = $obj->age; 
       $country = $obj->country; 
      } 
        //display form with a hidden field containing the value of the ID 
?> 

  <form action="post_update_profile.php" method="post"> 

  <label>Name</label><input name="name" value="<?=$name?>"> 
  <label>Age</label><input name="age" value="<?=$age?>"> 
  <label>Country</label><input name="country" value="<?=country?>"> 

</form> 

  <?php 

  } else { 
         exit('No such user'); 
  } 

} else { 
  echo $id; //this  should be No ID'; 
 exit; 
}   

```

请注意，我们在表单中使用了所谓的快捷`echo`语句。这使我们的代码更简单，更易读。由于我们使用的是 PHP 7，这个功能应该是开箱即用的。

一旦有人提交表单，它就会进入我们的`$_POST`变量，我们将在我们的`UserProfile`类中创建一个新的`Update`函数。

# 管理员系统

最后，让我们创建一个简单的*网格*，用于与我们的用户资料数据库一起使用的管理员仪表板门户。我们对此的要求很简单：我们只需设置一个基于表格的布局，以行显示每个用户资料。

从网格中，我们将添加链接以便能够编辑个人资料，或者删除它，如果我们想要的话。在我们的 HTML 视图中显示表格的代码如下：

```php
<table> 
 <tr> 
  <td>John Doe</td> 
  <td>21</td> 
  <td>USA</td> 
  <td><a href="edit_profile.php?id=1">Edit</a></td> 
  <td><a href="profileview.php?id=1">View</a> 
  <td><a href="delete_profile.php?id=1">Delete</a> 
 </tr> 
</table> 
This script to this is the following: 
//listprofiles.php 
$sql = "SELECT * FROM userprofiles LIMIT $start, $limit ";  
$rs_result = mysqli_query ($sql); //run the query 

while($row = mysqli_fetch_assoc($rs_result) { 
?> 
    <tr> 
           <td><?=$row['name'];?></td> 
           <td><?=$row['age'];?></td>  
        <td><?=$row['country'];?></td>       

         <td><a href="edit_profile.php?id=<?=$id?>">Edit</a></td> 
          <td><a href="profileview.php?id=<?=$id?>">View</a> 
          <td><a href="delete_profile.php?id=<?=$id?>">Delete</a> 
           </tr> 

<?php 
} 

```

有一件事我们还没有创建：`delete_profile.php`页面。查看和编辑页面已经讨论过了。

`delete_profile.php`页面将如下所示：

```php
<?php 

//delete_profile.php 
$connection = mysqli_connect('localhost','<username>','<password>', '<databasename>'); 

$id = $_GET['id'] ?? 'No ID'; 

if(is_numeric($id)) { 
mysqli_query( $connection, "DELETE FROM userprofiles WHERE id = '" .$id . "'"); 
} else { 
 echo $id; 
} 
i(!is_numeric($id)) {  
exit('Error: non numeric \$id');  
 } else { 
echo "Profile #" . $id . " has been deleted"; 

?> 

```

当然，由于我们的数据库中可能有很多用户资料，我们必须创建一个简单的分页。在任何分页系统中，你只需要找出总行数和每页要显示的行数。我们可以创建一个函数，它将能够返回一个包含页码和每页要查看的数量的 URL。

从我们的查询数据库中，我们首先创建一个新的函数，让我们只选择数据库中的总项目数：

```php
class UserProfile{ 
 // .... Etc ... 
function count_rows($table) { 
      $dbconn = new mysqli('localhost', 'root', 'somepass', 'databasename');  
  $query = $dbconn->query("select COUNT(*) as num from '". $table . "'"); 

   $total_pages = mysqli_fetch_array($query); 

   return $total_pages['num']; //fetching by array, so element 'num' = count 
} 

```

对于我们的分页，我们可以创建一个简单的`paginate`函数，它接受页面的`base_url`，每页的行数（也称为每页要显示的记录数），以及找到的记录的总数：

```php
require('definitions.php'); 
require('db.php'); //our database class 

Function paginate ($base_url, $rows_per_page, $total_rows) { 
  $pagination_links = array(); //instantiate an array to hold our html page links 

   //we can use null coalesce to check if the inputs are  null   
  ( $total_rows || $rows_per_page) ?? exit('Error: no rows per page and total rows);  
     //we exit with an error message if this function is called incorrectly  

    $pages =  $total_rows % $rows_per_page; 
    $i= 0; 
       $pagination_links[$i] =  "<a href="http://". $base_url  . "?pagenum=". $pagenum."&rpp=".$rows_per_page. ">"  . $pagenum . "</a>"; 
      } 
    return $pagination_links; 

} 

```

这个函数将帮助在表格中显示上面的页面链接：

```php
function display_pagination($links) {
      $display = '<div class="pagination">
                  <table><tr>';
      foreach ($links as $link) {
               echo "<td>" . $link . "</td>";
      }

       $display .= '</tr></table></div>';

       return $display;
    }
```

请注意，我们遵循的原则是函数内部应该很少有`echo`语句。这是因为我们希望确保这些函数的其他用户在调试页面上出现神秘输出时不会感到困惑。

通过要求程序员回显函数返回的内容，可以更容易地调试我们的程序。此外，我们遵循关注点分离，我们的代码不输出显示，它只是格式化显示。

因此，任何未来的程序员都可以更新函数的内部代码并返回其他内容。这也使我们的函数可重用；想象一下，将来有人使用我们的函数，这样，他们就不必再次检查我们的函数中是否有一些错放的`echo`语句。

### 提示

**关于替代短标签的说明**

如你所知，另一种`echo`的方法是使用`<?= `标签。你可以这样使用：`<?="helloworld"?>`。这些被称为短标签。在 PHP 7 中，替代 PHP 标签已被移除。RFC 声明`<%`、`<%=`、`%>`和`<script language=php>`已被弃用。RFC 在[`wiki.php.net/rfc/remove_alternative_php_tags`](https://wiki.php.net/rfc/remove_alternative_php_tags)中表示，RFC 并未移除短开标签（`<?`）或带有`echo`的短开标签（`<?= `）。

由于我们已经铺好了创建分页链接的基础，现在我们只需要调用我们的函数。以下脚本就是使用前面的函数创建分页页面所需的全部内容：

```php
$mysqli = mysqli_connect('localhost','<username>','<password>', '<dbname>'); 

   $limit = $_GET['rpp'] ?? 10;    //how many items to show per page default 10; 

   $pagenum = $_GET['pagenum'];  //what page we are on 

   if($pagenum) 
     $start = ($pagenum - 1) * $limit; //first item to display on this page 
   else 
     $start = 0;                       //if no page var is given, set start to 0 
/*Display records here*/ 
$sql = "SELECT * FROM userprofiles LIMIT $start, $limit ";  
$rs_result = mysqli_query ($sql); //run the query 

while($row = mysqli_fetch_assoc($rs_result) { 
?> 
    <tr> 
           <td><?php echo $row['name']; ?></td> 
           <td><?php echo $row['age']; ?></td>  
        <td><?php echo $row['country']; ?></td>            
           </tr> 

<?php 
} 

/* Let's show our page */ 
/* get number of records through  */ 
   $record_count = $db->count_rows('userprofiles');  

$pagination_links =  paginate('listprofiles.php' , $limit, $rec_count); 
 echo display_pagination($paginaiton_links); 

```

我们页面链接的 HTML 输出在`listprofiles.php`中会看起来像这样：

```php
<div class="pagination"><table> 
 <tr> 
        <td> <a href="listprofiles.php?pagenum=1&rpp=10">1</a> </td> 
         <td><a href="listprofiles.php?pagenum=2&rpp=10">2</a>  </td> 
        <td><a href="listprofiles.php?pagenum=3&rpp=10">2</a>  </td> 
    </tr> 
</table></div> 

```

# 总结

正如你所看到的，我们对 null 合并有很多用例。

我们学习了如何创建一个简单的用户配置文件系统，以及如何在从数据库获取数据时使用 PHP 7 的 null 合并功能，如果没有记录，则返回 null。我们还了解到，null 合并运算符类似于三元运算符，只是如果没有数据，默认返回 null。

在下一章中，我们将有更多用例来使用其他 PHP 7 功能，特别是在为我们的项目创建数据库抽象层时。
