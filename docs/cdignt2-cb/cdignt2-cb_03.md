# 第三章：创建电子商务功能

在本章中，我们将涵盖：

+   修改配置设置以在数据库中运行会话

+   创建基本购物车

+   通过产品分类添加和搜索

+   将购物车保存到数据库中

# 简介

CodeIgniter 中的`Cart`类提供了基本的购物车功能，例如添加项目、修改购物车、显示购物车详情以及从购物车中删除项目。在本章中，我们将探讨如何使用 CodeIgniter 的`Cart`类创建一个简单的商店。

# 修改配置设置以在数据库中运行会话

“为什么我需要这样做；我现在没有使用会话，而且，我已经在其他菜谱中涵盖了会话？”我知道；我听到了你的话；但是 CodeIgniter 的`Cart`类使用会话来为顾客或用户构建购物车。我们将在这里考虑`Cart`类来讲解会话。如果你已经实现了会话，你可能可以跳过这个菜谱。

## 准备工作

在我们开始购物车示例之前，我们需要做一些准备工作。首先，我们将修改一些配置设置，这将允许 CodeIgniter 将购物车信息保存到数据库中，然后我们将创建一个简单的数据库模式来处理产品等。

## 如何操作...

在我们开始之前，我们需要应用一些配置更改，如下所示：

1.  打开文本编辑器中的`application/config/config.php`文件，并进行以下修改：

    | 配置变量 | 值 | 说明 |
    | --- | --- | --- |
    | `$config['encryption_key'] = ''` | `Alphanumeric` | 指定 CodeIgniter 在加密会话时应使用的加密密钥。该值可以是字母数字的，你必须决定一个字符串作为其值。 |
    | `$config['sess_encrypt_cookie']` | `TRUE/FALSE` | 指定是否在数据库中加密会话；对于本例，应设置为`TRUE`。 |
    | `$config['sess_use_database]` | `TRUE/FALSE` | 指定是否将会话存储在数据库中；对于本例，应设置为`TRUE`。 |
    | `$config['sess_table_name']` | `Alphanumeric` | 数据库中用于存储会话的表名。对于本例，应设置为`sess_cart`。 |

1.  为了支持这个菜谱，需要一些表和一些虚拟数据。将以下 SQL 代码复制到你的数据库中：

    ```php
    CREATE TABLE IF NOT EXISTS `sessions` (
      `session_id` varchar(40) COLLATE utf8_bin NOT NULL DEFAULT '0',
      `ip_address` varchar(16) COLLATE utf8_bin NOT NULL DEFAULT '0',
      `user_agent` varchar(120) COLLATE utf8_bin DEFAULT NULL,
      `last_activity` int(10) unsigned NOT NULL DEFAULT '0',
      `user_data` text COLLATE utf8_bin NOT NULL,
      PRIMARY KEY (`session_id`),
      KEY `last_activity_idx` (`last_activity`)
    ) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin;

    CREATE TABLE IF NOT EXISTS `products` (
      `product_id` int(11) NOT NULL AUTO_INCREMENT,
      `product_name` varchar(255) NOT NULL,
      `product_code` int(11) NOT NULL,
      `product_description` varchar(255) NOT NULL,
      `category_id` int(11) NOT NULL,
      `product_price` int(11) NOT NULL,
      PRIMARY KEY (`product_id`)
    ) ENGINE=InnoDB  DEFAULT CHARSET=latin1 AUTO_INCREMENT=3;

    INSERT INTO `products` (`product_id`, `product_name`, `product_code`, `product_description`, `category_id`, `product_price`) VALUES
    (1, 'Running Shoes', 423423, 'These are some shoes', 2, 50),
    (2, 'Hawaiian Shirt', 34234, 'This is a shirt', 1, 25);

    CREATE TABLE IF NOT EXISTS `categories` (
      `cat_id` int(11) NOT NULL AUTO_INCREMENT,
      `cat_name` varchar(50) NOT NULL,
      PRIMARY KEY (`cat_id`)
    ) ENGINE=InnoDB  DEFAULT CHARSET=latin1 AUTO_INCREMENT=3;

    INSERT INTO `categories` (`cat_id`, `cat_name`) VALUES
    (1, 'Shirts'),
    (2, 'Footware');
    ```

## 工作原理...

好吧，我们刚才做了什么？以下是对两个表的描述：

### 分类表

分类表存储有关每个产品（在产品表中）关联的分类组的信息。它们如下所示：

| 项目名称 | 属性 | 描述 |
| --- | --- | --- |
| `cat_id` | `INTEGER(11)` | 表的主键。 |
| `cat_name` | `VARCHAR(50)` | 分类名称。 |

### 产品表

产品表存储有关购物车中每个销售产品的信息。它通过外键（`category_id`）与分类表相关联。以下是产品表：

| 项目名称 | 属性 | 描述 |
| --- | --- | --- |
| `product_id` | `INTEGER(11)` | 表的主键 |
| `product_name` | `VARCHAR(125)` | 产品名称 |
| `product_code` | `INTEGER(11)` | 产品代码，可以用作内部库存代码 |
| `product_description` | `VARCHAR(255)` | 产品详细描述 |
| `category_id` | `INTEGER(11)` | 产品分类 ID（作为从分类表的外键） |
| `product_price` | `INTEGER(11)` | 产品价值。 |

# 创建基本购物车

在本节中，我们将创建运行购物车所需的基本文件。在本章的后面部分，我们将添加更多的功能，但首先让我们做好准备。我们将创建以下四个文件：

+   `path/to/codeigniter/application/controllers/shop.php`: 此控制器将处理视图和模型之间的任何客户交互，例如处理任何表单和控制客户通过购物车的旅程。

+   `path/to/codeigniter/application/models/shop_model.php`: 此模型将处理控制器与数据库之间的任何数据库交互。它将包含获取产品和产品分类的函数，并在本章的后面部分，将购物车保存到数据库中。

+   `path/to/codeigniter/application/views/shop/display_cart.php`: 此视图将显示任何时刻客户的购物车摘要，并允许该客户修改购物车中项目的数量。

+   `path/to/codeigniter/application/views/shop/view_products.php`: 此视图将显示数据库中 `cart.products` 表的产品，并提供选项允许客户将项目添加到他们的购物车中。

## 如何操作...

我们将创建以下四个文件：

+   `/path/to/codeigniter/application/controllers/shop.php`

+   `/path/to/codeigniter/application/models/shop_model.php`

+   `/path/to/codeigniter/application/views/shop/display_cart.php`

+   `/path/to/codeigniter/application/views/shop/display_products.php`

1.  将以下代码复制到 `/path/to/codeigniter/application/controllers/shop.php` 文件中：

    ```php
    <?php if (!defined('BASEPATH')) exit('No direct script access allowed');

    class Shop extends CI_Controller {
        function __construct() {
            parent::__construct();
            $this->load->library('cart');
            $this->load->helper('form');
            $this->load->helper('url');
            $this->load->helper('security');
            $this->load->model('Shop_model');
        }

        public function index() {
            $data['query'] = $this->Shop_model->get_all_products();
            $this->load->view('shop/display_products', $data);
        }

        public function add() {
            $product_id = $this->uri->segment(3);
            $query = $this->Shop_model->get_product_details($product_id);
            foreach($query->result() as $row) {
                $data = array(
                    'id'   => $row->product_id,
                    'qty' => 1,
                    'price'  => $row->product_price,
                    'name' => $row->product_name,
                 );
            }

            $this->cart->insert($data);
            $this->load->view('shop/display_cart', $data);
        }

        public function update_cart() {
            $data = array();
            $i = 0;

            foreach($this->input->post() as $item) {
                    $data[$i]['rowid']  = $item['rowid'];
                    $data[$i]['qty']    = $item['qty'];
                    $i++;
            }

            $this->cart->update($data);
            redirect('shop/display_cart');
        }

        public function display_cart() {
            $this->load->view('shop/display_cart');
        }

        public function clear_cart() {
            $this->cart->destroy();
            redirect('index');
        }
    }
    ```

1.  然后将以下代码复制到 `/path/to/codeigniter/application/models/shop_model.php` 文件中：

    ```php
    <?php if ( ! defined('BASEPATH')) exit('No direct script access allowed');

    class Shop_model extends CI_Model {

        function __construct() {
            parent::__construct();
            $this->load->helper('url');
        }

        public function get_product_details($product_id) {
            $this->db->where('product_id', $product_id);
            $query = $this->db->get('products');
            return $query;
        }

        public function get_all_products() {
            $query = $this->db->get('products');
            return $query;
        }
    }
    ```

1.  然后将以下代码复制到 `path/to/codeigniter/application/views/shop/display_cart.php` 文件中：

    ```php
    <?php echo form_open('shop/update_cart'); ?>

    <table cellpadding="6" cellspacing="1" style="width:50%" border="1">

      <tr>
        <th>Quantity</th>
        <th>Description</th>
        <th>Item Price</th>
        <th>Sub-Total</th>
      </tr>

      <?php $i = 1; ?>

      <?php foreach ($this->cart->contents() as $items): ?>

        <?php echo form_hidden($i . '[rowid]', $items['rowid']); ?>

        <tr>
          <td><?php echo form_input(array('name' => $i . '[qty]', 'value' => $items['qty'], 'maxlength' => '3', 'size' => '5')); ?></td>
          <td>
            <?php echo $items['name']; ?>

            <?php if ($this->cart->has_options($items['rowid']) == TRUE): ?>

              <p>
                <?php foreach ($this->cart->product_options($items['rowid']) as $option_name => $option_value): ?>

                  <strong><?php echo $option_name; ?>:</strong> <?php echo $option_value; ?><br/>

                <?php endforeach; ?>
              </p>

            <?php endif; ?>

          </td>
          <td><?php echo $this->cart->format_number($items['price']); ?></td>
          <td>$<?php echo $this->cart->format_number($items['subtotal']); ?></td>
        </tr>

        <?php $i++; ?>

      <?php endforeach; ?>

      <tr>
        <td colspan="2"> </td>
        <td><strong>Total</strong></td>
        <td>$<?php echo $this->cart->format_number($this->cart->total()); ?></td>
      </tr>

    </table>

    <p><?php echo form_submit('', 'Update Cart'); ?></p>
    <?php echo form_close() ; ?>
    ```

1.  最后，将以下代码复制到 `path/to/codeigniter/application/views/shop/display_products.php` 文件中：

    ```php
    <body>
        <table>
        <?php foreach ($query->result() as $row) : ?>
            <tr>
            <td><?php echo $row->product_id ; ?></td>
            <td><?php echo $row->product_name ; ?></td>
            <td><?php echo $row->product_description ; ?></td>
            <td><?php echo anchor('shop/add/'.$row->product_id, 'Add to cart') ; ?></td>
            </tr>
        <?php endforeach ; ?>
        </table>
    </body>
    ```

## 工作原理...

好吧，这里有很多内容；所以我们不会讨论每个文件中的内容，而是将其分解为用户操作。当用户与购物车交互时，可以执行几种操作。最常见的是以下操作：

+   用户浏览目录

+   用户将项目添加到购物车

+   用户更新或删除购物车中的项目

### 用户浏览目录

`controllers/shop.php`中的`public function index()`通过调用`Shop_model`中的`$this->Cart_model->get_all_products()`函数从数据库中加载所有产品，并通过`$this->load->view('shop/display_products', $data)`将其传递给`views/shop/display_products.php`视图。

### 用户将项目添加到购物车

用户点击`views/shop/display_products.php`文件中他们想要购买的项目旁边的“**添加到购物车**”链接，这将在`controllers/shop.php`中调用`public function add()`。`public function add()`通过`$this->uri->segment(3)`从 URL 获取传递给它的产品 ID。使用这个，`$product_id`通过`$this->Shop_model->get_product_details($product_id)`从数据库中查找产品详情。然后，通过`$this->cart->insert($data)`将这些数据写入购物车。最后，用户通过`$this->load->view('shop/display_cart', $data)`被重定向到购物车，以便他们可以查看购物车中的项目，并根据需要做出任何修改。

### 用户更新或删除购物车中的项目

这涵盖了两个动作：向购物车添加更多项目或完全删除购物车中的项目或所有项目。它们如下所述：

+   **向购物车添加或减去项目**：当用户查看他们的购物车时，他们会看到左侧一个表格，显示购物车中每个项目的数量。用户可以通过更改此文本框中的值来更改项目的数量。如果用户增加或减少特定项目的当前数量并按下“**更新购物车**”按钮，那么我们将运行商店控制器函数`update_cart()`。`update_cart()`遍历 POST 数组，查看每个项目和它的新或期望的数量。它将使用`$data`数组中的行值跟踪每个项目，确保正确地更新了正确的数量。

+   **从购物车中删除项目**：此功能与之前描述的添加或删除项目的方式相同。然而，区别在于如果用户选择的数量是`0（零）`，那么 CodeIgniter 将完全从购物车中删除该项目。

# 添加和按产品类别搜索

从客户的角度来看，能够通过查看类别（如鞋子、衬衫、外套等）来缩小您的目录是有用的。如果您想添加此功能，您需要修改数据库。如果您需要此功能，请复制以下“准备就绪”部分中的代码。

## 准备就绪

为了支持按类别搜索和筛选，我们需要添加一个类别表。如果您在章节前面的“准备就绪”部分还没有这样做，请在您的数据库中创建以下表格：

```php
CREATE TABLE IF NOT EXISTS `categories` (
  `cat_id` int(11) NOT NULL AUTO_INCREMENT,
  `cat_name` varchar(50) NOT NULL,
  PRIMARY KEY (`cat_id`)
) ENGINE=InnoDB  DEFAULT CHARSET=latin1 AUTO_INCREMENT=3;

INSERT INTO `categories` (`cat_id`, `cat_name`) VALUES
(1, 'Shirts'),
(2, 'Footware');
```

## 如何操作...

我们现在需要对以下文件进行修改：

+   `/path/to/codeigniter/views/shop/display_products.php`：添加了一个小菜单，允许用户点击不同的类别；结果将相应地过滤。

+   `/path/to/codeigniter/views/shop/display_cart.php`：添加了一个小菜单，允许用户点击不同的类别；结果将相应地过滤。

+   `/path/to/codeigniter/application/controllers/shop.php`：这段添加的代码将显示类别，并在表单提交中查找用户的类别选择。

+   `/path/to/codeigniter/application/models/shop_model.php`：添加了从数据库根据类别 ID 获取类别的代码。

1.  在`path/to/codeigniter/application/views/shop/display_cart.php`文件的顶部复制以下代码：

    ```php
    <?php echo form_open('shop/index') ; ?>
      <select name="cat">
        <?php foreach ($cat_query->result() as $cat_row) : ?>
           <option value="<?php echo $cat_row->cat_id;?>"><?php echo $cat_row->cat_name;?></option>
        <?php endforeach ; ?>
     </select>
    <?php echo form_submit('', 'Search') ; ?>
        <?php echo form_close() ; ?>
    ```

1.  在`path/to/codeigniter/application/views/shop/display_products.php`文件的顶部复制以下代码：

    ```php
        <?php echo form_open('shop/index') ; ?>
            <select name="cat">
                <?php foreach ($cat_query->result() as $cat_row) : ?>
                    <option value="<?php echo $cat_row->cat_id;?>"><?php echo $cat_row->cat_name;?></option>
                <?php endforeach ; ?>
            </select>
        <?php echo form_submit('', 'Search') ; ?>
        <?php echo form_close() ; ?>
    ```

1.  将商店控制器中的`public function index()`替换为以下内容：

    ```php
        public function index() {
            $this->load->library('form_validation');
            $this->form_validation->set_error_delimiters('', '<br />');

            if ($this->input->post()) {
                $category_id = $this->input->post('cat');
            } else {
                $category_id = null;
            }

            $this->form_validation->set_rules('cat', 'Category', 'required|min_length[1]|max_length[125]|integer');

            if ($this->form_validation->run() == FALSE) {
                $data['query'] = $this->Shop_model->get_all_products($category_id);
                $data['cat_query'] = $this->Shop_model->get_all_categories();
                $this->load->view('shop/display_products', $data); 
            } else {
                $data['query'] = $this->Shop_model->get_all_products($category_id);
                $data['cat_query'] = $this->Shop_model->get_all_categories();
                $this->load->view('shop/display_products', $data);
            }
        }
    ```

1.  将商店控制器函数`add()`替换为以下代码（更改已突出显示）：

    ```php
    public function add() {
            $product_id = $this->uri->segment(3);
            $query = $this->Shop_model->get_product_details($product_id);
            foreach($query->result() as $row) {
                $data = array(
                    'id' => $row->product_id,
                    'qty' => 1,
                    'price' => $row->product_price,
                    'name' => $row->product_name,
                 );
            }

            $this->cart->insert($data);
     $data['cat_query'] = $this->Shop_model->get_all_categories();

            $this->load->view('shop/display_cart', $data);
        }
    ```

1.  将控制器函数`displat_cart()`替换为以下代码（更改已突出显示）：

    ```php
    public function display_cart() {
     $data['cat_query'] = $this->Shop_model->get_all_categories();
            $this->load->view('shop/display_cart', $data);
        }
    ```

1.  将`shop_model`中的`public function get_all_products()`替换为以下代码：

    ```php
    public function get_all_products($category_id = null) {
            if ($category_id) {
                $this->db->where('category_id', $category_id);
            }
            $query = $this->db->get('products');
            return $query;
        }
    ```

1.  将以下函数`get_all_categories()`添加到`shop_model`中，如下所示：

    ```php
    public function get_all_categories() {
            $query = $this->db->get('categories');
            return $query;
        }
    ```

## 它是如何工作的...

我们已经修改了`public function index()`，以便我们可以通过`$category_id`（这是数据库中每个类别的主键）过滤用户的浏览结果。如果页面是首次加载（不是作为提交），则代码：

```php
  if ($this->input->post()) { 
            $category_id = $this->input->post('cat'); 
        } else { 
            $category_id = null; 
        }
```

将自动将`$category_id`设置为 null，因此`$this->Shop_model->get_all_products($category_id)`将返回所有产品，无论`category_id`如何。然而，如果通过表单提交传递了`$category_id`，则`$this->Shop_model->get_all_products($category_id)`将只返回分配给该类别的产品。

# 将购物车保存到数据库

在您的客户准备好进行支付之前，您需要收集他们的支付、配送和您的记录详情，然后将会话表中的购物车移动到将存储订单的特定表中。

一旦订单已保存并且提供了客户详细信息，就会生成一个唯一的订单代码并存储在`orders.order_fulfilment_code`中。这可以被支付提供商（例如，PayPal、GoCardless、Stripe 等）用来跟踪通过他们的系统到您系统的支付处理。

## 如何操作...

1.  首先，在您的数据库中创建以下表：

    ```php
    CREATE TABLE IF NOT EXISTS `customer` (
      `cust_id` int(11) NOT NULL AUTO_INCREMENT,
      `cust_first_name` varchar(125) NOT NULL,
      `cust_last_name` varchar(125) NOT NULL,
      `cust_email` varchar(255) NOT NULL,
      `cust_created_at` int(11) NOT NULL,
      `cust_address` text NOT NULL COMMENT 'card holder address',
      PRIMARY KEY (`cust_id`)
    ) ENGINE=InnoDB DEFAULT CHARSET=latin1 AUTO_INCREMENT=1 ;

    CREATE TABLE `orders` (
      `order_id` int(11) NOT NULL AUTO_INCREMENT,
      `cust_id` int(11) NOT NULL,
      `order_details` text NOT NULL,
      `order_created_at` int(11) NOT NULL,
      `order_closed` int(1) NOT NULL COMMENT '0 = open, 1 = closed',
      `order_fulfilment_code` varchar(255) NOT NULL COMMENT 'the unique code sent to a payment provider',
      `order_delivery_address` text NOT NULL,
      PRIMARY KEY (`order_id`)
    ) ENGINE=InnoDB  DEFAULT CHARSET=latin1 AUTO_INCREMENT=1;
    ```

1.  现在我们已经创建了存储您的客户和订单的数据库表，让我们创建支持这个新功能的文件。我们将创建以下两个文件：

    +   `/path/to/codeigniter/application/controllers/cust.php`

    +   `/path/to/codeigniter/application/models/cart_model.php`

1.  在`/path/to/codeigniter/application/controllers/cust.php`控制器中创建控制器并添加以下代码：

    ```php
    <?php if (!defined('BASEPATH')) exit('No direct script access allowed');

    class Cust extends CI_Controller {
        function __construct() {
            parent::__construct();
            $this->load->library('cart');
            $this->load->helper('form');
            $this->load->helper('url');
            $this->load->helper('security');
            $this->load->model('Shop_model');
        }

        public function index() {
            redirect('cust/user_details');
        }

        public function user_details() {
            $this->load->library('form_validation');
            $this->form_validation->set_error_delimiters();

            // Set validation rules
            $this->form_validation->set_rules('first_name', 'First Name', 'required|min_length[1]|max_length[125]');
            $this->form_validation->set_rules('last_name', 'Last Name', 'required|min_length[1]|max_length[125]');
            $this->form_validation->set_rules('email', 
              'Email Address', 'required|min_length[1]|max_length[255]|valid_email');
            $this->form_validation->set_rules('email_confirm', 'Comfirmation Email Address', 
                'required|min_length[1]|max_length[255]|valid_email|matches[email]');
            $this->form_validation->set_rules('payment_address', 'Payment Address', 'required|min_length[1]|max_length[1000]');
            $this->form_validation->set_rules('delivery_address', 'Delivery Address', 'min_length[1]|max_length[1000]');

            // Begin validation
            if ($this->form_validation->run() == FALSE) {
                $this->load->view('shop/user_details');
            } else {
                $cust_data = array(
                'cust_first_name' => $this->input->post('cust_first_name'),
                'cust_last_name' => $this->input->post('cust_last_name'),
                'cust_email'=> $this->input->post('cust_email'),
                'cust_address'  => $this->input->post('payment_address'),
                'cust_created_at' => time());

                $payment_code = mt_rand();

                $order_data = array(
                'order_details' => serialize($this->cart->contents()),
                'order_delivery_address' => $this->input->post('delivery_address'),
                'order_created_at' => time(),
                'order_closed' => '0',
                'order_fulfilment_code' => $payment_code,
                'order_delivery_address' => $this->input->post('payment_address'));

                if ($this->Shop_model->save_cart_to_database($cust_data, $order_data)) {
                    echo 'Order and Customer saved to DB';
                } else {
                    echo 'Could not save to DB';
                }
            }
        }
    }
    ```

1.  在`models/shop_model.php`模型中添加`public function save_cart_to_db()`，如下所示：

    ```php
        public function save_cart_to_database($cust_data, $order_data) {
            $this->db->insert('customer', $cust_data);
            $order_data['cust_id'] = $this->db->insert_id();
            if ($this->db->insert('orders', $order_data)) {
                return true;
            } else {
                return false;
            }
        }
    ```

1.  修改`views/shop/display_cart.php`文件。在页面顶部添加以下行：

    ```php
    <?php echo anchor('cust/user_details', 'Proceed to checkout') ; ?>
    ```

1.  创建`filepath/to/codeigniter/application/views/shop/user_details.php`文件，并将以下代码添加到其中：

    ```php
    <body>
        <?php echo validation_errors(); ?>
        <?php echo form_open('/cust/user_details') ; ?>
            <?php echo form_input(array('name' => 'first_name', 'value' => 'First Name', 'maxlength' => '125', 'size' => '50')); ?><br />
            <?php echo form_input(array('name' => 'last_name', 'value' => 'Last Name', 'maxlength' => '125', 'size' => '50')); ?><br />
            <?php echo form_input(array('name' => 'email', 'value' => 'Email Address', 'maxlength' => '255', 'size' => '50')); ?><br />
            <?php echo form_input(array('name' => 'email_confirm', 'value' => 'Confirm Email', 'maxlength' => '255', 'size' => '50')); ?><br />
            <?php echo form_textarea(array('name' => 'payment_address', 'value' => 'Payment Address', 'rows' => '6', 'cols' => '40', 'size' => '50')); ?><br />
            <?php echo form_submit('', 'Enter') ; ?><br />
            <?php echo form_close() ; ?>
        </form>
    </body>
    ```

## 它是如何工作的...

为了解释这里发生的事情，让我们从顾客的角度来看待这个问题。顾客在商店里四处看了看，挑选了几件商品并将它们放入购物车。下一步是将购物车转换为订单。因此，用户点击**查看购物车**来查看他们想要订购的产品（我们已经讨论过这一点，所以不会再深入讨论。）当用户点击**前往结账**时，会调用公共函数`user_details()`，它将`views/shop/user_details.php`显示给顾客。这要求他们输入一些信息；在这种情况下，他们的名字、电子邮件地址、支付方式、送货地址等等。在成功提交表单（即没有验证错误）后，他们的订单将从购物车移动到数据库，并与他们提交的用户详细信息匹配。

还通过行`$payment_code = mt_rand()`创建了一个跟踪码，该跟踪码可用于通过支付提供商系统跟踪支付。
