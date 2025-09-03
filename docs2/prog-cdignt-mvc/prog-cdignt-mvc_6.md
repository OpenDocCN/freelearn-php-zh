# 第六章 模型

本章涵盖了 CI 模型、它们的作用以及它们在多个 Web 应用程序代码示例中的使用。该模型负责处理它存储和检索的数据库对象，并包含应用程序实现的逻辑。

大部分数据都是应用程序持久状态的一部分（无论这种持久状态是存储在文件中还是数据库中），在数据加载到应用程序后应位于模型对象中。因为模型对象代表与特定主题相关的知识和专业知识，它们可以在应用程序中重复使用。

模型代表应用程序数据服务，也可以服务于应用程序逻辑（通常称为业务逻辑）。通常，模型负责以下操作：

+   **添加、修改、删除和搜索应用程序数据库对象**：通常，这包括数据库操作，但实现相同的操作并调用外部 Web 服务或 API 并不罕见。

+   **封装应用程序逻辑**：例如，模型可以在存储数据对象之前进行数据验证，并可以向调用应用程序模块报告问题。

CI 数据库类的最常见误用是直接从控制器、视图或辅助程序中使用它。一个好的做法是开发模型类来处理所有应用程序的数据库服务。

因此，所有其他应用程序模块都可以从中受益，并重用这些模型。

CI 模型是专门设计来处理数据库或外部信息资源（如 Facebook）的特殊类（我们将在本章中看到一个示例）。

CI 模型是为与数据库中的信息一起工作而设计的 PHP 类。

本章将主要关注以下主题：

+   CI 模型的作用范围：

    +   模型资源路径

    +   加载模型

    +   使用模型方法

    +   连接到数据库

    +   业务逻辑

+   对象关系映射（ORM）

+   示例 1：CRUD 示例

+   示例 2：业务逻辑示例

+   示例 3：从 Facebook 检索数据

我们将首先简要回顾 CI 模型的作用范围，然后通过几个使用示例继续，涵盖在真实项目中结合的不同用例。

# CI 模型的作用范围

CI 模型为所有应用程序模块提供了一种面向对象的方式，以访问应用程序数据库（或外部信息资源）。通常，模型类将包含帮助我们检索、插入和更新数据库中的信息的函数。

本节将专注于 CI 模型语法和用法指南，作为以下使用代码示例的序言。

## 模型资源路径

模型文件位于文件夹`application/models/`中，遵循模式`application/models/<MODEL_NAME>.php`。

## 加载模型

加载模型可以是自动的，也可以通过控制器完成。更具体地说，可以在特定控制器的构造函数或任何控制器方法中完成。

+   如果模型只在少数控制器方法中使用，建议在这些方法中加载模型。在这种情况下，模型的范围仅限于那些方法的项目，并将引用`application/models/mymodel.php`。

+   如果模型在大多数控制器方法中使用，建议在控制器构造函数中加载模型。在这种情况下，模型的范围是所有控制器方法的项目，并将引用`application/models/mymodel.php`。

    ```php
    $this->load->model('mymodel');
    ```

    它自动为所有 CI 项目加载模型`mymodel`。

+   如果模型在 CI 项目的大部分控制器中使用，建议在`application/config/autoload.php`中自动加载它。在这种情况下，模型的范围是所有 CI 项目，并将引用`application/models/mymodel.php`。

    `$autoload['model'] = array('users', 'mymodel');`

## 使用模型方法

一旦 CI 模型被加载，我们将使用以模型名称作为类的对象来访问模型函数。模型的函数被调用以执行数据库操作，例如从数据库检索、插入和更新数据。

```php
// Loading the model mymodel in the controller's method
$this->load->model('mymodel');
// Calling the model's method my_function 
$this->mymodel->my_function();
```

例如，让我们加载模型`users`并访问其函数`get_users`。

```php
// Loading the model class
$this->load->model('usermodel');
// Calling the model to retrieve the users from the database
$view_params['users'] = $this->usermodel->get_users();
```

## 连接到数据库

更多信息，请参阅第二章，*配置和命名约定*。

在此示例中，我们将手动连接到数据库。以下设置在`application/config/database.php`中完成：

```php
$config['hostname'] = '127.0.0.1';
$config['username'] = 'db_username';
$config['password'] = 'db_password';
$config['database'] = 'db_database';
$config['port'] = 'db_port';
$config['dbdriver'] = 'mysql';
$config['dbprefix'] = '';
$config['pconnect'] = TRUE;
$config['db_debug'] = TRUE;
$config['cache_on'] = FALSE;
$config['cachedir'] = '';
$config['char_set'] = 'utf8';
$config['dbcollat'] = 'utf8_general_ci';
$config['swap_pre'] = '';
$config['autoinit'] = TRUE;
$config['stricton'] = FALSE;
// Loading the database with the configuration manually
this->load->database($config);
```

## 业务逻辑

业务逻辑是为特定信息对象主题或数据库对象定义的一组验证规则和决策标准。

模型可以对它处理的数据库和信息对象应用业务逻辑。

在社交网络的情况下，模型层将负责诸如保存用户数据、保存朋友关联、存储和检索用户照片、寻找新朋友以供推荐等任务。

# 对象关系映射 (ORM)

当 CI 为开发者提供模型类以扩展面向对象的数据库**CRUD**（**创建、读取、更新和删除**）验证，并为定义的项目数据库提供业务逻辑时，还有一个选项可以启用自动模型服务。在本节中，我们将讨论**对象关系映射**（**ORM**）。ORM 是将数据库模式定义转换为面向对象数据库类 API 的新概念。它为给定数据库提供数据库 CRUD 服务，因此所需的代码量最小，而不是完整的模型开发。更重要的是，还启用了对 CRUD 操作的定制验证。使用 ORM 插件可以减少我们自行开发 CI 模型的需求，这样只剩下特殊业务逻辑活动需要实现。

今天，ORM 插件提供了预定义的验证服务，以及用户定义的服务，以强制执行来自请求数据库 CRUD 服务的应用程序控制器、库或助手中的 CRUD 请求的验证。

使用 ORM 既有优点也有缺点。一方面，它大大简化了数据库模型开发。另一方面，它对数据库模式定义施加了各种规则，例如为 ORM 对象用户定义用户表，或者定义自动递增主键字段名，如 ID 等。

CI 有几个 ORM 插件；最著名和最完善的，拥有庞大的社区开发者网络，如下所示：

+   **Doctrine ORM** ([docs.doctrine-project.org](http://docs.doctrine-project.org)): 此 ORM 插件具有良好的 CI 集成指南文档，可在[`docs.doctrine-project.org/en/2.0.x/cookbook/integrating-with-codeigniter.html`](http://docs.doctrine-project.org/en/2.0.x/cookbook/integrating-with-codeigniter.html)找到。

+   **DataMapper CodeIgniter ORM 库** ([datamapper.wanwizard.eu](http://datamapper.wanwizard.eu))：它提供了 CI 库，例如用户指南网络导航器。

+   这两个 ORM 库不仅提供基于表的 CRUD 服务，还可以配置以处理外键字段之间的跨表关系。它们可以支持一对一、一对多和多对多关系，甚至多个数据库表之间的更复杂关系。

ORM 插件还提供了对处理数据库字段进行验证和操作的服务，例如在将字符串字段保存到数据库之前对其进行修剪。

验证服务包括内置验证，如有效的电子邮件字段，或者必须与另一个字段具有相同值的字段，例如具有账户创建密码重输要求的字段。ORM 的完整范围和用法超出了本 CI 书籍的范围。然而，强烈建议您了解更多关于 ORM 的信息，并尝试使用所提到的 ORM 插件，并考虑在 CI 项目中使用它们。

当然，我们在下一节中提供了一个简单的使用示例，演示如何使用 ORM 向数据库添加记录，以及如何使用 ORM 检索数据库记录：

## ORM 简单操作示例

例如，假设我们有一个以 ID 作为主键自动递增的用户数据库表。用户名、电子邮件和密码是其他字段，如果我们想向数据库添加一个新的用户记录，我们可以使用以下代码来完成：

```php
<?PHP
// We shall define the database table named users// with ID as auto-increment, username, password, and e-mail as // the other fields.
// ORM will create an user objet based on the users // table scheme. We can set the variable to this object, and use // the operational services provided by ORM for actions, such // as save, delete, update, and add.
$u = new User();
$u->username = 'A new User';
$u->password = 'shhnew1';
$u->email = 'user@mail.com';
// To add a new user record
if ($u->save()) {
  // if saved we have a new echo 'New User Id Opened having'$u->id. 'User Id <br/>';
  }
else {// Show why we failed to save echo
  $u->error->string;
  }
// Getting the first three users from the database
$u = new User();
$u->limit(3)->get();
// Showing the fetched users
foreach ($u as $user_rec)
{
  echo 'User Id: '. $user_rec->id . '<br/>';
  echo 'User Name: '. $user_rec->username . '<br />';
  echo 'User Email: '. $user_rec->email. '<br/>';
  }
// Get the user with Uid = 10 if any
$u = new user();
$seek_uid = 10;

$u->where('id', $seek_uid)->get();
// Check if found
if (exist ($u)){
  echo 'User Id:'.$u->id.' Name is'.$u->username. '<br />';
  }
else echo 'No user found for user ID'. $seek_uid. '<br />';
```

这只是一个非常简单的使用示例，而 ORM 今天提供了丰富的 CRUD 和验证服务。请参阅提供的 ORM 插件链接以获取更多信息。

# 示例 1 – CRUD 示例

在这个示例中，我们将看到如何使用 CI 模型。对于这个示例，我们将使用一个在数据库上执行以下操作的模型：`SELECT`、`INSERT`和`UPDATE`。

示例显示，在数据库主页中由模型`productmodel`检索的所有产品。

假设项目根 URI 为`http://mydomain.com/myproject`和`http://mydomain.com/myproject/product`。

### 注意

源代码通过 URL 提供。

主页有添加和编辑产品的链接。这些链接会生成用于编辑和添加产品的表单。

假设项目根 URI 为`http://mydomain.com/myproject`和`http://mydomain.com/myproject/product/add`。

假设我们想编辑并更新`product_id 1`的产品，链接将是`http://mydomain.com/myproject/product/edit/1`。此示例将由以下控制器、模型和视图构建：

+   `application/controllers/product.php`：此控制器加载模型 product。

    ```php
    $this->load->model('productmodel');
    ```

    此控制器渲染以下视图：

    +   `productsview`：此视图显示所有产品，并提供编辑和添加产品的链接

    +   `productform`：此视图包含添加和编辑产品的表单

+   `application/models/productmodel.php`：此模型包含执行以下数据库操作的函数：`SELECT`、`INSERT`和`UPDATE`。

+   `application/views/productsview.php`：显示产品的视图。

+   `application/views/productform.php`：包含表单的视图。

## 控制器文件

控制器 PHP 文件位于`application/controllers/product.php`。控制器处理产品的操作，如添加、编辑、更新和显示产品表。

控制器创建用于添加和编辑产品的表单。

更多信息请参阅第七章，*视图*。

以下为代码和内联说明：

```php
<?php
if (!defined('BASEPATH')) exit('No direct script access allowed');
class Product extends CI_Controller {
// Accessory method for generating forms called by the methods add // and edit.
private function load_form($form_action, $a_values = array())
{
  // Loading the form helper
  $this->load->helper('form');
  // Loading the form_validation library
  $this->load->library('form_validation');
  $view_params['form']['attributes'] = array('id' => 'productform');
  $view_params['form']['action'] = $form_action;
  $product_id = isset($a_values['product_id']) ?
  $a_values['product_id']: 0;
  $view_params['form']['hidden_fields'] = array('product_id' => $product_id);
  // product name details
  $view_params['form']['product_name']['label'] = array('text' => 'Product name:', 'for' => 'product_name');
  $view_params['form']['product_name']['field'] = array('name' => 'product_name', 'id' => 'product_name', 'value' => isset($a_values['product_name']) ? $a_values['product_name']: '', 'maxlength' => '100', size' => '30', 'class' => 'input');
  // product sku details
  $view_params['form']['product_sku']['label'] = array('text' => 'Product SKU:', 'for' => 'product_sku');
  $view_params['form']['product_sku']['field'] = array('name' => 'product_sku', 'id' => 'product_sku', 'value' => isset($a_values['product_sku']) ? $a_values['product_sku']: '', 'maxlength' => '100', 'size' => '30', 'class' => 'input');
  // product quantity details
  $view_params['form']['product_quantity']['label'] = array('text' => 'Product Quantity:', 'for' => 'product_quantity');
  $view_params['form']['product_quantity']['field'] = array('name' => 'product_quantity', 'id' => 'product_quantity', 'value' => isset($a_values['product_quantity']) ? $a_values['product_quantity']: '', 'maxlength' => '100', 'size' => '30', 'class' => 'input');
  // Form attributes validation rules
  $config_form_rules = array(array('field' => 'product_name', 'label' => 'Product Name','rules' => 'trim|required'), array('field' => 'product_sku', 'label' => 'Product SKU', 'rules' => 'trim|required'), array('field' => 'product_quantity', 'label' => 'Product Quantity', 'rules' => 'trim|required|integer'));
  $this->form_validation->set_rules($config_form_rules);
  return $view_params;
  }
// This controller method retrieves the products list calling the // model productmodel's method get_products() renders the results // in the view productsview.
public function index()
{
  // Loading the url helper
  $this->load->helper('url');

  // Manually loading the database
  $this->load->database();

  // Loading the model class
  $this->load->model('productmodel');

  // Calling the model productmodel's method get_products()to // retrieve the products from the database.
  $view_params['products'] = $this->productmodel->get_products();
  // Rendering the products list in the view productsview.
  $this->load->view('productsview', $view_params);
  }
// This method handles the operation of adding a product to the // database.
public function add()
{
  // Loading the url helper
  $this->load->helper('url');

  // Manually loading the database
  $this->load->database();

  // Loading the model class
  $this->load->model('productmodel');

  $a_post_values = $this->input->post();
  $view_params = $this->load_form('product/add', $a_post_values);

  // Validating the form
  if ($this->form_validation->run() == FALSE) {
    // Validation failed
    $this->load->view('productform', $view_params);
    } else {
    $data = $a_post_values;
    array_pop($data);
    $this->productmodel->addProduct($data);

    redirect('product');
    }
  }
// This method handles the operation of editing a product
public function edit($product_id)
{
  // Loading the url helper
  $this->load->helper('url');
  // Manually loading the database
  $this->load->database();

  // Loading the model class
  $this->load->model('productmodel');

  $a_post_values = $this->input->post();
  // Checking if a form was submitted
  if ($a_post_values) {
    $a_form_values = $a_post_values;
    } else {
    // Get the values of the database
    $a_db_values = $this->productmodel->get_product($product_id);
    $a_form_values = array('product_id' => $a_db_values[0]->product_id, 'product_name' => $a_db_values[0]->product_name, product_sku' => $a_db_values[0]->product_sku, 'product_quantity' => $a_db_values[0]->product_quantity);
    }

  $view_params = $this->load_form('product/edit/' . $product_id, $a_form_values);
  // Validating the form
  if ($this->form_validation->run() == FALSE) {
    // Validation failed
    $this->load->view('productform', $view_params);
    } else {
    $a_fields = array('product_name', 'product_sku', 'product_quantity');
    for ($index = 0; $index < count($a_fields); $index++)
    {
      $s_field = $a_fields[$index];
      $data[$s_field] = $this->input->post($s_field)
      }
    $this->productmodel->updateProduct($product_id, $data);
    redirect('product');
    }
  }
}
/* End of file product.php */
/* Location: /application/controllers/product.php */
```

## 模型文件

模型 PHP 文件位于`application/models/productmodel.php`。

在此示例中，调用 CI 对象`db`的方法来生成和执行 SQL 查询。

请参阅 CI 数据库库[`ellislab.com/codeigniter/user-guide/database/index.html`](http://ellislab.com/codeigniter/user-guide/database/index.html)。

以下为代码和内联说明：

```php
<?php
class Productmodel extends CI_Model {
  // The model's constructor method
  public function __construct()
  {
    // Call the Model's parent constructor
    parent::__construct();
    }
  // This method retrieves the products list and returns an array of // objects each containing product details.
  public function get_products()
  {
    // Calling the CI's db object's method for generating SQL // queries.
    $query = $this->db->get('products');
    // returns an array of products objects
    return $query->result();
    }
  // This method retrieves a specific product's details identified by // $product_id as a parameter
  public function get_product($product_id)
  {
    // Calling the CI's db object's methods for generating SQL // queries.
    $this->db->select('*');
    $this->db->from('products');
    $this->db->where('product_id', $product_id);

    // Calling the CI's db object method for executing the query
    $query = $this->db->get();
    // Returning array of one object element containing product // details.
    return $query->result();
    }

  // This method adds a product to the products table Parameters // $data - The data to insert into the table
  public function addProduct($data)
  {
    // Calling the CI's db object method for inserting a product data // into the products table.
    $this->db->insert('products', $data);
    }
  // This method updates a product row in the products table // parameters $product_id - The product id, $data - The updated // data
  public function updateProduct($product_id, $data)
  {
  // Calling the CI's db object's methods for generating SQL queries
  $this->db->where('product_id', $product_id);
  // Calling the CI's db object method for updating the product data // in the products table
  $this->db->update('products', $data);
  }
}
```

## 视图文件

视图 PHP 文件位于`application/views/productsview.php`。

此视图文件显示包含产品列表的表格。以下为代码和内联说明：

```php
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="utf-8">
<title>Products List</title>
</head>
<body>
<table>
<tr>
  <td>ID</td>
  <td>Name</td>
  <td>SKU</td>
  <td>Quantity</td>
  <td>Actions</td>
</tr>

<?php foreach ($products as $product): ?>
<tr>
  <td><?php echo $product->product_id; ?></td>
  <td><?php echo $product->product_name; ?></td>
  <td><?php echo $product->product_sku ; ?></td>
  <td><?php echo $product->product_quantity ; ?></td>
  <td><a href="<?php echo site_url("product/edit/" . $product->product_id); ?>">Edit Product</a></td>
</tr>
<?php endforeach; ?>
</table>

<p>
  <a href="<?php echo site_url('product/add'); ?>">Add Product</a>
</p>
</body>
</html>
```

# 示例 2 - 业务逻辑示例

在此示例中，我们将演示业务逻辑。订购产品将触发模型更新产品的数量并检查是否小于某个数量。

此示例将由以下控制器、模型和视图构建：

+   `application/controllers/order.php`：此控制器加载模型`productmodel`

+   `$this->load->model('productmodel')`：此控制器渲染视图`orderview`，显示所有产品，并且每个产品都有订购产品的链接

+   `application/models/productmodel.php`: 此模型包含检索产品、更新其数量和检查其数量的函数

+   `application/views/orderview.php`: 视图以表格形式显示所有产品，其中每行都有一个订购产品的链接

假设项目根目录的 URI 为 `http://mydomain.com/myproject` 和 `http://mydomain.com/myproject/order`。

### 注意

代码源文件通过 URL 提供。

## 控制器文件

控制器 PHP 文件位于 `application/controllers/order.php`。

此控制器负责显示产品并更新每个产品。如果产品的数量达到限制，它将生成错误消息。

代码和内联说明如下：

```php
<?php
if (!defined('BASEPATH')) exit('No direct script access allowed');
class Order extends CI_Controller
{
  // This method retrieves the products list and returns an array // of objects each containing product details
  public function index()
  {
    // Loading the url helper
    $this->load->helper('url');

    // Manually loading the database
    $this->load->database();

    // Loading the model class
    $this->load->model('productmodel');

    $view_params['products'] = $this->productmodel->get_products();

    $this->load->view('orderview', $view_params);
    }
  // This method checks the product's quantity.
  // It updates the product row in the database or generates an // error message
  public function product($product_id)
  {
    // Loading the url helper
    $this->load->helper('url');

    // Manually loading the database
    $this->load->database();

    // Loading the model class
    $this->load->model('productmodel');

    if (!$this->productmodel->update_quantity($product_id)) {
      mail($user_mail, 'product' . $product_id . "reached it's limit", 'Order product' . $product_id);
      }
    redirect('product');
  }
}
```

## 模型文件

模型 PHP 文件位于 `application/models/productmodel.php`。

在此示例中，调用 CI 对象 `db` 的方法来生成和执行 SQL 查询。

请参阅 CI 数据库库，可在[`ellislab.com/codeigniter/user-guide/database/index.html`](http://ellislab.com/codeigniter/user-guide/database/index.html)找到。

代码和内联说明如下：

```php
<?php
class Productmodel extends CI_Model
{
  // The model's constructor method
  public function __construct()
  {
    // Call the model constructor
    parent::__construct();
    }
  // This method retrieves the products list and returns an array of // objects each containing product details.
  public function get_products()
  {
    // Calling the CI's db object's method for generating the // SQL queries.
    $query = $this->db->get('products');
    // returns an array of products objects
    return $query->result();
    }
  // This method retrieves a specific product's details // identified by $product_id as a parameter.
  public function get_product($product_id)
  {
  // Calling the CI's db object's methods for generating the // SQL queries.
  $this->db->select('*');
  $this->db->from('products');
  $this->db->where('product_id', $product_id);
  // Calling the CI's db object method for executing the query
  $query = $this->db->get();
  // Returning array of one object element containing the product // details.
  return $query->result();
  }
// This method adds a product to the products table parameters.
// $data - The data to insert into the table
public function addProduct($data)
{
  // Calling the CI's db object method for inserting a product data // into the products table.
  $this->db->insert('products', $data);
  }
// This method updates a product row in the products table // parameters.
// $product_id - The product id
// $data - The updated data
public function updateProduct($product_id, $data)
{
  // Calling the CI's db object's methods for generating the // SQL queries.
  $this->db->where('product_id', $product_id);
  // Calling the CI's db object method for updating the product data // in the products table.
  $this->db->update('products', $data);
  }

// This method checks whether the quantity exceeds it's limit.
private function check_quantity($product_id) {
  // Calling the CI's db object's methods for generating the // SQL queries.
  $this->db->select('product_quantity');
  $this->db->from('products');
  $this->db->where('product_id', $product_id);
  // Calling the CI's db object method for executing the query.
  $query = $this->db->get();
  // Calling the result's method row, which returns the SQL query // result row.
  $row = $query->row();
  if ($row->product_quantity < 7) {
    return false;
    } else {
    return true;
    }
  }

// This method updates a product quantity and return true or false, // if quantity reaches it's limit.
public function update_quantity($product_id)
{
  $sql = "UPDATE products SET product_quantity = product_quantity - 1 WHERE product_id=" $product_id;

  $this->db->query($sql);

  // Checking if the quantity reached it's limit.
  if ($this->check_quantity($product_id)) {
    return true;
    } else {
    return false;
    }
  }
}
```

## 视图文件

PHP 视图文件位于 `application/views/orderview.php`。此视图文件显示一个包含产品列表的表格。

以下为代码和内联说明：

```php
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="utf-8">
<title>Products List</title>
</head>
<body>
<table>
<tr>
  <th>ID</th>
  <th>Name</th>
  <th>SKU</th>
  <th>Quantity</th>
  <th>Actions</th>
</tr>
<?php foreach ($products as $product): ?>
<tr>
  <td><?php echo $product->product_id; ?></td>
  <td><?php echo $product->product_name;  ?></td>
  <td><?php echo $product->product_sku ; ?></td>
  <td><?php echo $product->product_quantity ; ?></td>
  <td><a href="<?php echo site_url("order/product/" . $product->product_id); ?>">Order Product</a></td>
</tr>
<?php endforeach; ?>
</table>
</body>
</html>
```

# 示例 3 – 从 Facebook 获取数据

在此示例中，我们将使用 CI 内置模型从 Facebook 获取数据。

此示例将显示 Facebook 用户名和图片，并显示用户的 Facebook 朋友。

此示例使用 Facebook PHP SDK 作为 CI 库。可以从[`github.com/facebook/php-sdk`](https://github.com/facebook/php-sdk)下载。更多信息，请参阅第四章，*库*。

此示例将由以下控制器、模型和视图构建：

+   `application/controllers/fbpage.php`: 此控制器加载模型 `fbmodel`

+   `$this->load->model('fbmodel')`: 此控制器渲染视图 `fbview`，显示用户的 Facebook 图片和姓名，以及包含用户朋友姓名和其个人资料链接的表格

+   `application/models/fbmodel.php`: 此模型包含从 Facebook 检索数据的函数

+   `application/views/fbview.php`: 此视图显示 Facebook 数据

假设项目根目录的 URI 为 `http://mydomain.com/myproject` 和 `http://mydomain.com/myproject/fbpage`。

### 注意

代码源文件通过 URL 提供。

## 控制器文件

控制器 PHP 文件位于 `application/controllers/fbpage.php`。

控制器负责从 Facebook 获取访问令牌，并将 Facebook 用户重定向到 Facebook 登录页面以确认 Facebook 应用的权限。

控制器还负责通过模型获取 Facebook 用户的详细信息和朋友，并相应地渲染视图页面。

如需了解更多关于 Facebook API 使用和开发的信息，请参阅位于[`developers.facebook.com/`](http://developers.facebook.com/)的 Facebook 开发者页面。

以下为代码和内联解释：

```php
<?php
class Fbpage extends CI_Controller {
  public function __construct() {
    parent::__construct();
    // Extremely important!!!
    // Due to the fact that the CI handles classes for // $_GET, $_POST, and $_COOKIE parse_str is called to // copy the variables sent by Facebook to the $_REQUEST var, // so that the Facebook SDK can do its checks.
    // This is done in order to avoid infinite redirect loop.
    parse_str($_SERVER['QUERY_STRING'], $_REQUEST);
    }
  // This method retrieves Facebook data of a Facebook user and // displays personal details and some of his  friends.
  // It checks if a Facebook token is valid, if it's valid, // then it displays his details, otherwise it produces // the token.
  public function index() {
    $a_config = array('appId' => $fb_API, 'secret'=> $fb_secret, 'cookie' => true);
    $this->load->library('facebook', $a_config);
    // Checking if the user is logged in and confirms // the app's permissions.
    if ($user = $this->facebook->getUser()) {
      // Get the Facebook token
      $access_token = $this->facebook->getAccessToken();
      // Loading the fbmodel
      $this->load->model('fbmodel');
      // Updating the token
      $this->fbmodel->set_token($access_token);
      // Get a Facebook user's profile details
      $user_profile = $this->fbmodel->get_user_profile();
      // Getting the Facebook user ID
      $uid = $user_profile['id'];

      // Retrieving a Facebook user's details
      $me = $this->fbmodel->get_me_by_fql($uid);
      // Get a Facebook user's friends
      $friends = $this->fbmodel->get_friends();
      $view_params = array('me' => $me, 'friends' => $friends);
      // Loading the view
      $this->load->view("fbview", $view_params);
      } else {
      // The Facebook parameters for the Facebook login URL, // where scope consists the Facebook app's permissions.
      $a_params = array ('fbconnect' => 0, 'scope' => offline_access, publish_stream', 'cookie' => true);
      // The Facebook login URL page
      $login_url= $this->facebook->getLoginUrl($a_params);
      // Redirecting the Facebook user to the login URL.
      // After the Facebook user confirms the permissions // required by the app; he is redirected back to the // index page.
      header('Location:'. $login_url);
      }
    }
  }
```

## 模型文件

模型 PHP 文件位于`application/models/fbmodel.php`。

模型负责与 Facebook SDK 交互，并检索 Facebook 用户的详细信息和朋友列表。该模型使用 Facebook FQL 机制。

如需了解更多关于 Facebook API 使用和开发的信息，请参阅位于[`developers.facebook.com/`](http://developers.facebook.com/)的 Facebook 开发者页面。

代码和内联解释如下：

```php
<?php
class fbmodel extends CI_Model {
  // The Facebook app's token
  private $token;
  public function __construct() {
    // Call the model constructor
    parent::__construct();
    }

  // This method sets the model class's private token value
  public function set_token($token) {
    $this->token = $token;
    }

  // This method returns an array, which contains the Facebook user // profile.
  public function get_user_profile() {
    // Getting the CI main class to get access to the Facebook // library.
    $ci =& get_instance();

    // Getting the Facebook user's profile
    $user_profile = $ci->facebook->api('/me');
    return $user_profile;
    }

  // This method returns an array, which contains a Facebook user's // details.
  public function get_me_by_fql($uid) {
    // Getting the CI main class to get access to the Facebook // library.
    $ci =& get_instance();
    // The SQL query to send to Facebook $fql = SELECT uid, name, // pic_big FROM user WHERE uid=" $uid;
    $param = array('method' => 'fql.query', 'query' => $fql, 'callback' => '');

    // Getting the Facebook user's details
    $fqlResult = $ci->facebook->api($param);
    // Returning an array, which contains the required details
    return $fqlResult;
    }

  // This method returns an array of a Facebook user's friend.
  public function get_friends() {
    // Getting the CI main class to get access to the Facebook // library
    $ci =& get_instance();
    // Getting the Facebook user's friends
    $friends = $ci->facebook->api('/me/friends');

    // Returning an array, which contains a Facebook user's friend
    return $friends;
    }
  }
```

## 视图文件

视图 PHP 文件位于`application/views/fbview.php`。

此视图文件显示了一个 Facebook 用户的详细信息以及他们的朋友详情表格。

代码和内联解释如下：

```php
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>My facebook details</title>
</head>
<body>

<div id="my_details">
  <div id="picture"><img src="img/<?=$me[0]['pic_big'] ?>"></div>
  <div id="my_name"><?=$me[0]['name'] ?></div>
</div>
<table>
<tr>
  <th>Name</th>
  <th>Link to friend</th>
</tr>
<?php foreach ($friends['data'] as $friend): ?>
<tr>
  <td><?=$friend['name']?></td>
  <td><a href='http://www.facebook.com/<?=$friend["id"]?>'>To friend</a></td>
</tr>
<?php endforeach; ?>
</table>
</body>
</html>
```

# 摘要

在本章中，我们回顾了 CodeIgniter 模型的作用域、业务逻辑和 ORM。在本章中，我们制作了以下示例：

+   示例 1：一个 CRUD 示例

+   示例 2：一个业务逻辑示例

+   示例 3：从 Facebook 检索数据
