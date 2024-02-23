# 第二章：构建一个数据库类和简单的购物车

对于我们以前的应用程序，只是用户配置文件，我们只创建了一个简单的**创建-读取-更新-删除（CRUD）**数据库抽象层 - 基本的东西。在本章中，我们将创建一个更好的数据库抽象层，它将允许我们做的不仅仅是基本的数据库功能。

除了简单的 CRUD 功能之外，我们将在数据库抽象类中添加结果操作。我们将在我们的数据库抽象类中构建以下功能：

+   将整数转换为其他更准确的数字类型

+   数组转对象

+   `firstOf()`方法：允许我们选择数据库查询结果的第一个结果

+   `lastOf()`方法：允许我们选择数据库查询结果的最后一个结果

+   `iterate()`方法：允许我们迭代结果并以我们将发送到此函数的格式返回它

+   `searchString()`方法：在结果列表中查找字符串

我们可能会根据需要添加更多的功能。在本章的末尾，我们将应用数据库抽象层来构建一个简单的**购物车**系统。

购物车很简单：已经登录的用户应该能够点击一些出售的物品，点击**添加到购物车**，并获取用户的详细信息。用户验证了他们的物品后，然后点击购买按钮，我们将把他们的购物车物品转移到购买订单中，他们将填写交货地址，然后保存到数据库中。

# 构建数据库抽象类

在 PHP 中，创建一个类时，有一种方法可以在每次初始化该类时调用某个方法。这称为类的构造函数。大多数类都有构造函数，所以我们将有自己的构造函数。构造函数的函数名是用两个下划线和`construct()`关键字命名的，就像这样：`function __construct()`。两个下划线的函数也被称为魔术方法。

在我们的数据库抽象类中，我们需要创建一个构造函数，以便能够返回`mysqli`生成的`link`对象：

```php
 Class DB { 

  public $db; 

  //constructor 
  function __construct($server, $dbname,$user,$pass) { 
    //returns mysqli $link $link = mysqli_connect(''); 
    return $this->db = mysqli_connect($server, $dbname, $user, $pass); 
  } 
} 

```

## 原始查询方法

`query`方法将执行传递给它的任何查询。我们将在`query`方法中调用 MySQLi 的`db->query`方法。

它是什么样子的：

```php
public function query($sql) { 
 $results =   $this->db->query($sql); 
 return $results; 
} 

```

## 创建方法

对于我们的数据库层，让我们创建`create`方法。通过这个方法，我们将使用 SQL 语法将项目插入到数据库中。在 MySQL 中，语法如下：

```php
INSERT INTO [TABLE] VALUES ([val1], [val2], [val3]); 

```

我们需要一种方法将数组值转换为以逗号分隔的字符串：

```php
 function create ($table, $arrayValues) { 
  $query = "INSERT INTO  `" . $table . " ($arrayVal);  //TODO: setup arrayVal 
  $results = $this->db->query($link, $query); 
} 

```

## 读取方法

对于我们的`db`层，让我们创建`read`方法。通过这个方法，我们将只使用 SQL 语法查询我们的数据库。

MySQL 中的语法如下：

```php
SELECT * FROM [table] WHERE [key] = [value] 

```

我们需要创建一个能够接受括号中的前置参数的函数：

```php
public function read($table, $key, $value){ 
         $query  = SELECT * FROM $table WHERE `". $key . "` =  " . $value; 
     return $this->db->query($query); 
} 

```

## 选择所有方法

我们的`read`方法接受一个`key`和`value`对。然而，可能有些情况下我们只需要选择表中的所有内容。在这种情况下，我们应该创建一个简单的方法来选择表中的所有行，它只接受要选择的`table`作为参数。

在 MySQL 中，您只需使用以下命令选择所有行：

```php
SELECT * FROM [table]; 

```

我们需要创建一个能够接受括号中的前置参数的函数：

```php
public function select_all($table){ 
         $query  = "SELECT * FROM " . $table; 
     return $this ->query($query); 
} 

```

## 删除方法

对于我们的`db`层，让我们创建`delete`方法。通过这个方法，我们将使用 SQL 语法删除数据库中的一些项目。

MySQL 语法很简单：

```php
DELETE FROM [table] WHERE [key] = [val]; 

```

我们还需要创建一个能够接受括号中的前置参数的函数：

```php
public function delete($table, $key, $value){ 
         $query  = DELETE FROM $table WHERE `". $key . "` =  " . $value; 
     return $this->query($query); 
} 

```

## 更新方法

对于我们的数据库层，让我们创建一个`update`方法。通过这个方法，我们将能够使用 SQL 语法更新数据库中的项目。

MySQL 语法如下：

```php
UPDATE [table] SET [key1] = [val1], [key2] => [val2]  WHERE [key] = [value] 

```

### 注意

请注意，`WHERE`子句可以比一个键值对更长，这意味着您可以向语句添加`AND`和`OR`。这意味着，除了使第一个键动态化之外，`WHERE`子句需要能够接受`AND`/`OR`作为其参数。

例如，您可以为`$where`参数编写以下内容，以选择`firstname`为`John`且`lastname`为`Doe`的人：

```php
firstname='John' AND lastname='Doe' 

```

这就是为什么我们在函数中将条件作为一个字符串参数的原因。我们数据库类中的`update`方法最终将如下所示：

```php
public function update($table, $updateSetArray, $where){ 
     Foreach($updateSetArray as $key => $value) { 
         $update_fields .= $key . "=" . $value . ","; 
     } 
      //remove last comma from the foreach loop above 
     $update_fields = substr($update_fields,0, str_len($update_fields)-1); 
    $query  = "UPDATE " . $table. " SET " . $updateFields . " WHERE " $where; //the where 
    return $this->query($query); 
} 

```

## first_of 方法

在我们的数据库中，我们将创建一个`first_of`方法，它将过滤掉其余的结果，只获取第一个结果。我们将使用 PHP 的`reset`函数，它只获取数组中的第一个元素：

```php
//inside DB class  
public function first_of($results) { 
  return reset($results); 
} 

```

## last_of 方法

`last_of`方法类似；我们可以使用 PHP 的`end`函数：

```php
//inside DB class  
public function last_of($results) { 
  Return end($results); 
} 

```

## iterate_over 方法

`iterate_over`方法将是一个简单添加格式的函数 - 在 HTML 代码之前和之后 - 例如，对于我们从数据库中获得的每个结果：

```php
public function iterate_over($prefix, $postfix, $items) { 
    $ret_val = ''; 
    foreach($items as $item) { 
        $ret_val .= $prefix. $item . $postfix; 
    } 
    return $ret_val; 
} 

```

## searchString 方法

给定一组结果，我们将查找某个字段中的内容。这样做的方法是生成类似于以下的 SQL 代码：

```php
    SELECT * FROM {table} WHERE {field} LIKE '%{searchString}%';
```

该函数将接受表和字段，以检查表中的搜索字符串`needle`：

```php
public function search_string($table, $column, $needle) { 
 $results = $this->query("SELECT * FROM `".$table."` WHERE " .    $column . " LIKE '%" . $needle. "%'"); 
   return $results; 
} 

```

## 使用 convert_to_json 方法实现一个简单的 API

有时我们希望数据库的结果以特定格式呈现。一个例子是当我们将结果作为 JSON 对象而不是数组处理时。这在您构建一个简单的 API 以供移动应用程序使用时非常有用。

这可能是可能的，例如，在另一个需要以特定格式（例如 JSON 格式）的系统中，我们可以将对象转换为 JSON 并发送它。

在 PHP 中，有一个`json_encode`方法，它将任何数组或对象转换为 JSON 表示。我们类的方法将只是将传递给它的值返回为`json`：

```php
function convertToJSON($object) { 
   return json_encode($object); 
   } 

```

# 购物车

现在我们将构建一个简化的购物车模块，它将利用我们新建的数据库抽象类。

让我们来规划一下购物车的功能：

+   **购物清单页面**：

+   购物者应该看到几个带有名称和价格的物品

+   购物者应该能够点击每个物品旁边的复选框，将其添加到购物车中

+   **结账页面**：

+   物品清单及其价格

+   总计

+   **确认页面**：

+   输入详细信息，如账单地址、账单信用卡号，当然还有名字

+   购物者还应该能够指定将商品发送到哪个地址

## 构建购物清单

在这个页面中，我们将创建基本的 HTML 块，以显示购物者可能想要购买的物品清单。

我们将使用与之前相同的模板系统，但是不再将整个代码放在一个页面中，而是将页眉和页脚分开，并简单地在我们的文件中包含它们使用`include()`。我们还将使用相同的 Bootstrap 框架来使我们的前端看起来漂亮。

### 物品模板渲染函数

我们将创建一个物品渲染函数，它将在`div`中渲染所有我们的购物物品。该函数将简单地返回一个带有物品价格、名称和图片的 HTML 标记：

```php
//accepts the database results as an array and calls db functions render_shopping_items($items) 
{ 
$db->iterate_over("<td>", "</td>", $item_name); 
    foreach($items as $item) { 
     $item->name.  ' ' .$item->price . ' ' . $item->pic; 

   } 
$resultTable .= "</table>"; 
} 

```

在上面的代码中，我们使用了我们新创建的`iterate_over`函数，该函数格式化数据库的每个值。最终结果是我们有了一个我们想要购买的物品的表格。

让我们创建一个简单的布局结构，每个页面都会得到页眉和页脚，并且从现在开始，只需包含它们：

在`header.php`中：

```php
<html> 
<!doctype html> 
<head> 
<link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.6/css/bootstrap.min.css"> 
</head> 
<body> 

```

在`footer.php`中：

```php
<div class="footer">Copyright 2016</div></body> 
</html> 

```

在`index.php`中：

```php
<?php 
require('header.php'); 
//render item_list code goes here 
require('itemslist.php'); //to be coded  
require('footer.php'); 
?> 

```

现在让我们创建`itemslist.php`页面，该页面将包含在`index.php`中：

```php
<?php  
include('DB.php'); 
$db = new DB(); 
$table = 'shopping_items'; 
$results = $db->select_all($table); 
//calling the render function created earlier: 
foreach(as $item) { 
  echo render_shopping_items($results);  
} 

?> 
//shopping items list goes here. 

```

我们的函数已经准备好了，但是我们的数据库还不存在。我们还需要填充我们的数据库。

通过在我们的 MySQL 数据库中创建`shopping_items`表来创建一些购物物品：

```php
CREATE TABLE shopping_items ( 
    id INT(11) NOT NULL AUTO_INCREMENT, 
    name VARCHAR(255) NOT NULL, 
    price DECIMAL(6,2) NOT NULL, 
   image VARCHAR(255) NOT NULL, 
PRIMARY KEY  (id)  
); 

```

让我们运行 MySQL，并将以下物品插入到我们的数据库中：

```php
INSERT INTO `shopping_items` VALUES (NULL,'Tablet', '199.99', 'tablet.png'); 
INSERT INTO `shopping_items` VALUES (NULL, 'Cellphone', '199.99', 'cellphone.png'); 
INSERT INTO `shopping_items` (NULL,'Laptop', '599.99', 'Laptop.png'); 
INSERT INTO `shopping_items` (NULL,'Cable', '14.99', 'Cable.png'); 
INSERT INTO `shopping_items` (NULL, 'Watch', '100.99', 'Watch.png'); 

```

将其保存在一个名为`insert_shopping_items.sql`的文件中。然后，在与`insert_shopping_items.sql`文件相同的目录中：

1.  登录到 MySQL 客户端并按照以下步骤进行：

```php
**mysql -u root -p**

```

1.  然后键入`use <数据库名称>`：

```php
**mysql>  use <database>;**

```

1.  使用`source`命令导入脚本：

```php
**mysql> source insert_shopping_items.sql**

```

当我们运行`SELECT * FROM shopping_items`时，我们应该看到以下内容：

![项目模板渲染函数](img/B05285_02_01.jpg)

## 向购物清单页面添加复选框

现在让我们为用户创建 HTML 复选框，以便能够选择购物物品。我们将创建以下形式来插入数据：

```php
//items.php 

<html> 
<body> 

<form action="post_profile.php" method="POST"> 
<table> 
  <input type="checkbox" value="<item_id>"> <td><item_image></td> 
  <td><item_name></td><td> 
 </table> 
</form> 
</body> 
</html> 

```

为此，我们需要修改我们的`render_items`方法以添加复选框：

```php
public function render_items($itemsArray)  { 

foreach($itemsArray as $item) { 
  return '<tr> 
           <td><input type="checkbox" name="item[]" value="' . $item->id. '">' .  . '</td><td>' . $item->image .'</td> 
<td>'. $item->name . '</td> 
'<td>'.$item->price . '</td> 
'</tr>'; 
} 
} 

```

在下一页上，当用户单击**提交**时，我们将需要获取所有 ID 并存储到一个数组中。

由于我们将复选框命名为`item[]`，我们应该能够通过`$_POST['item']`作为数组获取值。基本上，所有被选中的物品将作为数组存储在 PHP 的`$_POST`变量中，这将允许我们获取所有值以保存数据到我们的数据库中。让我们循环遍历结果的 ID，并在我们的数据库中获取每个物品的价格并将每个物品保存在名为`itemsArray`的数组中，其中键是物品的名称，值是物品的价格。

```php
$db = new DB(); 
$itemsArray= []; //to contain our items - since PHP 5.4, an array can be defined with []; 
foreach($_POST['item'] as $itemId) { 

   $item = $db->read('shopping_items', 'id', $itemId); 
   //this produces the equivalent SQL code: SELECT * FROM shopping_items WHERE id = '$itemId'; 
   $itemsArray[$item->name] = $item-price;  

} 

```

我们将首先与用户确认购买的物品。现在我们只会将物品和总金额保存到 cookie 中。我们将在结账页面上访问 cookie 的值，该页面将接受用户的详细信息，并在提交结账页面时将其保存到我们的数据库中。

### 提示

PHP 会话与 cookie：对于不太敏感的数据，例如用户购买的物品清单，我们可以使用 cookie，它实际上将数据（以纯文本形式！）存储在浏览器中。如果您正在构建此应用程序并在生产中使用它，建议使用会话。要了解有关会话的更多信息，请访问[`php.net/manual/en/features.sessions.php`](http://php.net/manual/en/features.sessions.php)。

## PHP 中的 Cookies

在 PHP 中，要启动一个 cookie，只需调用`setcookie`函数。为了将我们购买的物品保存到 cookie 中，我们必须对数组进行序列化，原因是 cookie 只能将值存储为字符串。

在这里，我们将物品保存到 cookie 中：

```php
setcookie('purchased_items', serialize($itemsArray), time() + 900); 

```

前面的 cookie 将在`purchased_items` cookie 中将物品存储为数组。它将在 15 分钟后过期（900 秒）。但是，请注意`time()`函数的调用，它返回当前时间的 Unix 时间戳。在 PHP 中，当达到最后一个参数中设置的时间时，cookie 将会过期。

### 注意

调试基于 cookie 的应用程序有时会令人沮丧。确保`time()`生成的时间戳确实显示当前时间。

例如，可能您最近重新格式化了您的计算机，由于某种原因无法正确设置时间。要测试`time()`，只需运行一个带有`time()`调用的 PHP 脚本，并检查[`www.unixtimestamp.com/`](http://www.unixtimestamp.com/)是否几乎相同。

## 构建结账页面

最后，我们将创建一个表单，用户可以在结账后输入他们的详细信息。

首先，我们需要为客户建立数据库表。让我们称这个表为`purchases`。我们需要存储客户的姓名、地址、电子邮件、信用卡、购买的物品和总额。我们还应该存储购买交易的时间，并使用唯一的主键来索引每一行。

以下是要导入到我们的 MySQL 数据库中的表的架构：

```php
CREATE TABLE purchases ( 
    id INT(11) NOT NULL AUTO_INCREMENT, 
    customer_name VARCHAR(255) NOT NULL, 
    address DECIMAL(6,2) NOT NULL, 
    email DECIMAL(6,2) NOT NULL, 
    credit_card VARCHAR(255) NOT NULL, 
    items TEXT NOT NULL, 
    total DECIMAL(6,2) NOT NULL, 
    created DATETIME NOT NULL, 
    PRIMARY KEY (id) 
); 

```

导入的一种方法是创建一个名为`purchases.sql`的文件，然后登录到您的 MySQL 命令行工具。

然后，您可以选择要使用的数据库：

```php
**USE <databasename>**

```

最后，假设您在与`purchases.sql`相同的目录中，您可以运行：

```php
**SOURCE purchases.sql** 

```

最后，通过创建一个简单的表单，包括地址、信用卡和买家姓名等详细信息的输入字段来完成：

```php
<form action="save_checkout.php" method="post"> 
<table> 
  <tr> 
   <td>Name</td><td><input type="text" name="fullname"></td>  
  </tr>  

 <tr> 
<td>Address</td><td><input type="text" name="address"></td> 
</tr> 
<tr> 
<td>Email</td><td><input type="text" name="email"></td> 
</tr> 

<tr>  
  <td>Credit Card</td><td><input type="text" name="credit_card"></td> 
 </tr> 
<tr>  
  <td colspan="2"><input type="submit" name="submit" value="Purchase"></td> 
 </tr> 

</table> 
</form> 

```

这是它的样子：

![构建结账页面](img/B05285_02_02.jpg)

最后，我们将像往常一样将所有内容保存到我们的数据库中的另一个表中，使用我们的`DB`类。为了计算总金额，我们将查询数据库的价格，并使用 PHP 的`array_sum`来获得总金额：

```php
$db = new DB($server,$dbname,$name,$password); 

//let's get the other details of the customer 
$customer_name = $_POST['fullname']; 
$address = $_POST['address']; 
$email = $_POST['email']; 
$credit_card = $_POST['credit_card]; 
$time_now = date('Y-m-d H:i:s'); 

foreach($purchased_items as $item) { 
  $prices[] = $item->price; 
} 

//get total using array_sum 
$total = array_sum($prices); 

$db->insert('purchases', [ 
   'address' => $address, 
'email' => $email, 
'credit_card' => $credit_card, 
 **'items' => //<all the items and prices>//,** 
   'total' => $total, 
    'purchase_date' => $timenow 
  ]);  
?> 

```

为了保持简单，正如您在突出显示的代码中所看到的，我们需要将所有购买的物品收集到一个长字符串中，以保存在我们的数据库中。以下是如何连接每个物品和它们的价格：

```php
foreach($purchased_items as $item) { 
   $items_text .= $item->name ":" . $item->price .  "," 
} 

```

然后我们可以将这些数据保存到变量`$items_text`中。我们将更新前面突出显示的代码，并将文本`<所有物品和价格>`更改为`$items_text`：

```php
... 
  'items' => $items_text 
 ... 

```

在我们的代码中，`foreach`循环应该放在调用`$db->insert`方法之前。

## 感谢页面

最后，我们已经将数据保存到我们的`purchased_items`表中。现在是时候向我们的客户说声谢谢并发送一封电子邮件了。在我们的`thankyou.php`的 HTML 代码中，我们将只写一张感谢便条，并让用户知道他们的购买情况即将收到一封电子邮件。

这是一个屏幕截图：

![感谢页面](img/B05285_02_03.jpg)

我们将文件命名为`thankyou.php`，它的 HTML 代码非常简单：

```php
<!DOCTYPE html> 
<html> 
<head> 
   <!-- Latest compiled and minified CSS --> 
   <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.6/css/bootstrap.min.css" integrity="sha384-1q8mTJOASx8j1Au+a5WDVnPi2lkFfwwEAa8hDDdjZlpLegxhjVME1fgjWPGmkzs7" crossorigin="anonymous"> 
   <title>Thank you for shopping at example.info</title> 
</head> 
<body> 
 <div class="container"> 
        <div class="row"> 
            <div class="col-lg-12 text-center"> 
                <h1>Thank you for shopping at example.info</h1> 
                     <p>Yey! We're really happy for choosing us to shop online. We've sent you an email of your purchases. </p> 
                     <p>Let us know right away if you need anything</p> 
            </div> 
        </div> 
    </div> 
</body> 
</html> 

```

使用 PHP 发送电子邮件是使用`mail()`函数完成的：

```php
mail("<to address>", "Your purchase at example.com","Thank you for purchasing...", "From: <from address>"); 

```

第三个参数是我们电子邮件的消息。在代码中，我们仍然需要添加购买的细节。我们将循环遍历我们之前制作的 cookie 和价格，然后输出总金额，并发送消息：

```php
$mail_message = 'Thank you for purchasing the following items'; 
$prices = []; 
$purchased_items = unserialize($_COOKIE['purchased_items']); 
foreach($purchased_items as $itemName => $itemPrice) { 
  $mail_message .=  $itemName . ": " .$itemPrice . "\r\n \r\n"; 
  //since this is a plain text email, we will use \r\n - which are escape strings for us to add a new line after each price. 
  $prices[] = $itemPrice; 
} 

$mail_message .= "The billing total of your purchases is " . array_sum($prices); 

mail($_POST['email'], "Thank you for shopping at example.info here is your bill", $mail_message, "From: billing@example.info"); 

```

我们可以将前面的代码添加到我们的`thankyou.php`文件的最后。

# 安装 TCPDF

您可以从 sourceforge 下载 TCPDF 库，[`sourceforge.net/projects/tcpdf/`](https://sourceforge.net/projects/tcpdf/)

TCPDF 是用于编写 PDF 文档的 PHP 类。

一个带有 TCPDF 示例的 PHP 示例代码如下：

```php
//Taken from http://www.tcpdf.org/examples/example_001.phps 

// Include the main TCPDF library (search for installation path). 
require_once('tcpdf_include.php'); 

// create new PDF document 
$pdf = new TCPDF(PDF_PAGE_ORIENTATION, PDF_UNIT, PDF_PAGE_FORMAT, true, 'UTF-8', false); 

// set document information 
$pdf->SetCreator(PDF_CREATOR); 
$pdf->SetAuthor('Nicola Asuni'); 
$pdf->SetTitle('TCPDF Example 001'); 
$pdf->SetSubject('TCPDF Tutorial'); 
$pdf->SetKeywords('TCPDF, PDF, example, test, guide'); 

// set default header data 
$pdf->SetHeaderData(PDF_HEADER_LOGO, PDF_HEADER_LOGO_WIDTH, PDF_HEADER_TITLE.' 001', PDF_HEADER_STRING, array(0,64,255), array(0,64,128)); 
$pdf->setFooterData(array(0,64,0), array(0,64,128)); 

// set header and footer fonts 
$pdf->setHeaderFont(Array(PDF_FONT_NAME_MAIN, '', PDF_FONT_SIZE_MAIN)); 
$pdf->setFooterFont(Array(PDF_FONT_NAME_DATA, '', PDF_FONT_SIZE_DATA)); 

// set default monospaced font 
$pdf->SetDefaultMonospacedFont(PDF_FONT_MONOSPACED); 

// set margins 
$pdf->SetMargins(PDF_MARGIN_LEFT, PDF_MARGIN_TOP, PDF_MARGIN_RIGHT); 
$pdf->SetHeaderMargin(PDF_MARGIN_HEADER); 
$pdf->SetFooterMargin(PDF_MARGIN_FOOTER); 

// set auto page breaks 
$pdf->SetAutoPageBreak(TRUE, PDF_MARGIN_BOTTOM); 

// set image scale factor 
$pdf->setImageScale(PDF_IMAGE_SCALE_RATIO); 

// set some language-dependent strings (optional) 
if (@file_exists(dirname(__FILE__).'/lang/eng.php')) { 
    require_once(dirname(__FILE__).'/lang/eng.php'); 
    $pdf->setLanguageArray($l); 
} 

// --------------------------------------------------------- 

// set default font subsetting mode 
$pdf->setFontSubsetting(true); 

// Set font 
// dejavusans is a UTF-8 Unicode font, if you only need to 
// print standard ASCII chars, you can use core fonts like 
// helvetica or times to reduce file size. 
$pdf->SetFont('dejavusans', '', 14, '', true); 

// Add a page 
// This method has several options, check the source code documentation for more information. 
$pdf->AddPage(); 

// set text shadow effect 
$pdf->setTextShadow(array('enabled'=>true, 'depth_w'=>0.2, 'depth_h'=>0.2, 'color'=>array(196,196,196), 'opacity'=>1, 'blend_mode'=>'Normal')); 

// Set some content to print 
$html = <<<EOD 
<h1>Welcome to <a href="http://www.tcpdf.org" style="text-decoration:none;background-color:#CC0000;color:black;">&nbsp;<span style="color:black;">TC</span><span style="color:white;">PDF</span>&nbsp;</a>!</h1> 
<i>This is the first example of TCPDF library.</i> 
<p>This text is printed using the <i>writeHTMLCell()</i> method but you can also use: <i>Multicell(), writeHTML(), Write(), Cell() and Text()</i>.</p> 
<p>Please check the source code documentation and other examples for further information.</p> 
<p style="color:#CC0000;">TO IMPROVE AND EXPAND TCPDF I NEED YOUR SUPPORT, PLEASE <a href="http://sourceforge.net/donate/index.php?group_id=128076">MAKE A DONATION!</a></p> 
EOD; 

// Print text using writeHTMLCell() 
$pdf->writeHTMLCell(0, 0, '', '', $html, 0, 1, 0, true, '', true); 

// --------------------------------------------------------- 

// Close and output PDF document 
// This method has several options, check the source code documentation for more information. 
$pdf->Output('example_001.pdf', 'I'); 

```

有了这个例子，我们现在可以使用前面的代码并稍微修改一下，以便创建我们自己的发票。我们只需要相同的 HTML 样式和我们总计生成的值。让我们使用相同的代码并更新值到我们需要的值。

在这种情况下，我们将设置作者为网站的名称`example.info`。并将我们的主题设置为`发票`。

首先，我们需要获取主要的 TCPDF 库。如果您将其安装在不同的文件夹中，我们可能需要提供一个相对路径，指向`tcpdf_include.php`文件：

```php
require_once('tcpdf_include.php'); 

```

这将使用类的默认方向和默认页面格式实例化一个新的 TCPDF 对象：

```php
$pdf = new TCPDF(PDF_PAGE_ORIENTATION, PDF_UNIT, PDF_PAGE_FORMAT, true, 'UTF-8', false); 

$pdf = new TCPDF(PDF_PAGE_ORIENTATION, PDF_UNIT, PDF_PAGE_FORMAT, true, 'UTF-8', false); 

// set document information 
$pdf->SetCreator(PDF_CREATOR); 
$pdf->SetAuthor('Example.Info'); 
$pdf->SetTitle('Invoice Purchases'); 
$pdf->SetSubject('Invoice'); 
$pdf->SetKeywords('Purchases, Invoice, Shopping'); 
s 

$html = <<<EOD 
<h1>Example.info Invoice </h1> 
<i>Invoice #0001.</i> 
EOD; 

```

现在，让我们使用 HTML 来创建一个客户购买的 HTML 表格：

```php
$html .= <<<EOD 
<table> 
  <tr> 
    <td>Item Purchases</td> 
    <td>Price</td> 
  </tr> 
EOD; 

```

### 注意

这种多行字符串的写法被称为 heredoc 语法。

让我们通过实例化我们的`DB`类来创建与数据库的连接：

```php
$db = new DBClass('localhost','root','password', 'databasename'); 
We shall now query our database with our database class: 

$table = 'purchases'; 
$column = 'id';  
$findVal = $_GET['purchase_id']; 

   $result = $db->read ($table, $column, $findVal); 

foreach($item = $result->fetch_assoc()) { 
$html .=   "<tr> 
         <td>". $item['customer_name']. "</td> 
         <td>" . $item['items'] . " 
</tr>"; 

$total = $items['total']; //let's save the total in a variable for printing in a new row 

} 

$html .= '<tr><td colspan="2" align="right">TOTAL: ' ".$total. " ' </td></tr>'; 

$html .= <<<EOD 
</table> 
EOD; 

$pdf->writeHTML($html, true, false, true, false, ''); 

$pdf->Output('customer_invoice.pdf', 'I'); 

```

在创建 PDF 时，重要的是要注意，大多数 HTML 到 PDF 转换器都是简单创建的，并且可以解释简单的内联 CSS 布局。我们使用表格打印出每个项目，这对于表格数据来说是可以的。它为布局提供了结构，并确保事物被正确对齐。

# 管理购买的管理员

我们将建立管理系统来处理所有的购买。这是为了跟踪每个从我们网站购买东西的客户。它将包括两个页面：

+   所有购买了东西的客户的概述

+   能够查看客户购买的物品

我们还将在这些页面上添加一些功能，以便管理员更容易地更改客户的信息。

我们还将创建一个简单的**htaccess apache 规则**，以阻止其他人访问我们的管理站点，因为它包含非常敏感的数据。

让我们首先开始选择我们`purchases`表中的所有数据：

```php
<?php 
//create an html variable for printing the html of our page: 
$html = '<!DOCTYPE html><html><body>'; 

$table = 'purchases'; 
$results = $db->select_all($table); 

//start a new table structure and set the headings of our table: 
$html .= '<table><tr> 
    <th>Customer name</th> 
    <th>Email</th> 
    <th>Address</th> 
    <th>Total Purchase</th> 
</tr>'; 

//loop through the results of our query: 
while($row = $results->fetch_assoc()){ 
    $html .= '<tr><td>'$row['customer_name'] . '</td>'; 
    $html .= '<td>'$row['email'] . '</td>'; 
    $html .= '<td>'$row['address'] . '</td>'; 
    $html .= '<td>'$row['purchase_date'] . '</td>'; 
    $html .= '</tr>'; 
} 

$html .= '</table>'; 
$html .= '</body></html>; 

//print out the html 
echo $html; 

```

现在，我们将添加一个链接到我们客户数据的另一个视图。这个视图将使管理员能够查看他们所有的购买。我们可以通过在客户姓名上添加一个链接，将第一个页面链接到客户购买的详细视图，方法是将我们添加客户姓名到`$html`变量的行更改为：

```php
    $html .= '<tr><td><a href="view_purchases.php?pid='.$row['id'] .'">'$row['customer_name'] . '</a></td>'; 

```

请注意，我们已经将`$row['id']`作为 URL 的一部分。现在我们可以通过`$_GET['pid']`值访问我们将要获取的数据的 ID 号。

让我们在一个新文件`view_purchases.php`中创建查看客户购买物品的代码：

```php
<?php 
//create an html variable for printing the html of our page: 
$html = '<!DOCTYPE html><html><body>'; 

$table = 'purchases; 
$column = 'id'; 
**$purchase_id = $_GET['pid'];** 
$results = $db->read($table, $column, $purchase_id);  
//outputs: 
// SELECT * FROM purchases WHERE id = '$purchase_id'; 

//start a new table structure and set the headings of our table: 
$html .= '<table><tr><th>Customer name</thth>Total Purchased</th></tr>'; 
//loop through the results of our query: 
while($row = $results->fetch_assoc()){ 
    $html .= '<tr><td>'$row['customer_name'] . '</td>'; 
    $html .= '<tr><td>'$row['email'] . '</td>'; 
    $html .= '<tr><td>'$row['address'] . '</td>'; 
    $html .= '<tr><td>'$row['purchase_date'] . '</td>'; 
    $html .= '</tr>'; 
} 
$html .= '</table>'; 
echo $html; 

```

在上面的代码中，我们使用了`$_GET['id']`变量来查找客户的确切购买记录。虽然我们可以只使用客户姓名来查找表`purchases`中客户的购买记录，但这将假定客户只通过我们的系统购买了一次。此外，我们没有使用客户姓名来确定我们是否有时有相同姓名的客户。

通过使用表`purchases`的主要 ID，在我们的情况下，通过选择`id`字段来确保我们选择了特定的唯一购买。请注意，由于我们的数据库很简单，我们可以只查询数据库中的一个表 - 在我们的情况下是`purchases`表。

也许一个更好的实现方法是将“购买”表分成两个表 - 一个包含客户的详细信息，另一个包含已购买物品的详细信息。这样，如果同一个客户返回，他们的详细信息可以在下次自动填写，我们只需要将新购买的物品链接到他们的账户上。

在这种情况下，“购买”表将简单地称为“已购买物品”表，每个物品将与客户 ID 相关联。客户的详细信息将存储在一个包含其唯一地址、电子邮件和信用卡详细信息的“客户”表中。

然后，您将能够向客户展示他们的购买历史。每次客户从商店购买物品时，交易日期将被记录下来，您需要按照每笔交易的日期和时间对历史记录进行排序。

# 总结

太好了，我们完成了！

我们刚刚学会了如何构建一个简单的数据库抽象层，以及如何将其用于购物车。我们还学习了关于 cookies 和使用 TCPDF 库构建发票。

在下一章中，我们将构建一个完全不同的东西，并使用会话来保存用户的当前信息，以构建基于 PHP 的聊天系统。
