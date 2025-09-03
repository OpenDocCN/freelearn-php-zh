# 第四章：库

本章涵盖了 CI 库的主题，以及不同类型的库及其不同的使用类别，并提供了几个 Web 应用程序的代码示例。CI 开发平台为我们提供了内置库，使我们能够通过简单的程序集成第三方库，还允许我们开发自己的库并与社区共享，如果我们愿意的话。

CI 库正在推动效率、代码重用性、分离和简单性。所获得的好处如下：

+   **效率**：意味着以最少的资源负载。这一特性是通过以下事实实现的，即 CI 库可能只由特定的 CI 项目控制器（或甚至只由特定的方法）加载，在这些方法中需要库的服务。因此，在执行时间中，库资源（内存）的负载在每个控制器操作状态下都最小化了。

+   **重用性**：重用性意味着编写一次函数代码并在项目资源中重复使用它。任何项目控制器、模型或辅助程序（在辅助程序中，我们将使用之前讨论多次的`&get_instance()`方法）都可以加载库以在 CI 项目的任何地方重复使用其代码。不仅如此，控制器渲染的视图还可以调用这些已加载的库方法。因此，实现了极高的代码重用性。

+   **分离**：分离防止了与项目其他地方相同名称的参数或函数的意外重叠。库类方法和参数有自己的命名空间，这样它们就不会在库外部被错误地覆盖，如果开发者在服务模块（如控制器/视图）中使用相同的参数。

+   **简单性**：这使得代码文本尽可能简单，易于理解和维护。从服务资源（如控制器、模型和辅助程序）调用的库方法使代码看起来更加简单，并使其易于维护和导航。因此，这简化了代码的扩展和维护。

这些库为我们提供了开发力量和效率，在特定项目方面具有丰富的功能，还使我们能够通过调用库方法而不是在控制器中本地放置服务代码，以简单和简洁的方式在服务控制器中编写代码。库应该由使用它们的代码（如控制器、模型或辅助程序）初始化，或者如果几乎所有的控制器都使用它，可以使用自动加载机制来加载模型。第二章配置和命名约定讨论了如何自动加载库。

一旦通过自动加载或控制器构造函数实例化，库就可以通过控制器方法或渲染的视图使用。此外，任何模型、辅助程序或其他库都可以使用 `&get_instance()`，如前几章所述，来使用我们项目安装的库。

这些库为 CI 模型-视图-控制器实例化组件的代码提供支持（有关功能扩展和项目控制器、模型、辅助程序和视图的可重用性更多信息，请访问网站 [`en.wikipedia.org/wiki/Model-view-controller`](http://en.wikipedia.org/wiki/Model-view-controller))。

本章将主要关注以下主题：

+   CI 库的作用域和用法：

    +   使用类别

    +   使用库

    +   将库添加到项目中

    +   实例化一个库

    +   使用库方法

+   可用的 CI 库

+   示例：

    +   示例 1：使用内置库

    +   示例 2：使用第三方库，例如 Google Maps CI 库包装器

    +   示例 3：构建自己的库，例如 Flickr API 包装器

    +   示例 4：构建自己的库，例如 LinkedIn API 包装器

我们将首先简要回顾 CI 框架中的库是什么，以及我们如何在我们项目的代码资源中用它来满足我们的需求。

# CI 库的作用域和用法

CI 库默认情况下无法访问控制器资源，除非调用 CI `$ci = &get_instance()` 并使用 `$ci` 而不是 `$this` 来访问 CI 资源，例如，而不是 `$this->db->query ($sql)`，我们应该使用 `$ci->db->query ($sql)`，依此类推。我们可以通过 CI echo 系统（全球 CI 开发者社区分享代码和开放问题的知识、资源和解决方案）使用第三方库来扩展 CI 库，或者从头开始开发自己的库或扩展其他库。

任何应用程序库都将位于项目目录下的 `application/libraries/` 目录中。此外，可选资源，如库配置文件，这些文件对于库配置是必需的，可以放置在项目根目录或其他位置。一个良好的做法是将它们放置在项目根目录下，以增强 CI 提供的安全性。例如，`<PROJECT_ROOT>application/config/<LIB_NAME>_config.php`，或者甚至可能还需要在另一个 `application/<LIB_ADDITIONAL_RESOURCES>` 目录下添加额外的资源，例如 `application/assets` 下的 `images/CSS/HTML/additional` 类库。

库集成和 CI 项目中的使用方法如下：

+   将库代码资源添加到 `application/libraries/my_lib.php`，如果有的话，还可以添加相关的资源，例如库配置文件，以及/或其他库资产到之前提到的位置。

+   通过配置自动加载或控制器实例化库类。

    +   自动加载整个 CI 项目的库 `my_lib`：

        ```php
        $autoload['libraries'] = array('database','my_lib');
        ```

    +   特别是在某些控制器（s）、构造函数（s）或方法（s）中：

        ```php
        $this->load->library('my_lib');
        ```

+   使用库方法：

    ```php
    $result=$this->my_lib->called_method ($param1, $param2);
    ```

+   我们可以将库范围视为整个项目代码资源模型、视图、辅助工具和库的终极面向对象重用启用器，它们共同处理所有执行请求，或计划请求。

如前所述，CI 库为我们提供了极大的分离和简单性。例如，以下代码：

```php
// Library class
class my_handler {
  private $my_lib_param;
  // Can't be accessed outside the class directly
  // but we can provide the read only service as follows:

  public function get_my_lib_param () {
    return $this->my_lib_param;}
  // The following is a library function that can't 
  // be called from outside the class!

  private function my_private_function () { }
  }
```

## 可用的 CI 库

CI 和 CI 开发者回声系统提供了许多涵盖丰富主题的库。我们将回顾 CI 库以及已知的第三方 CI 库资源。

我们还鼓励构建自己的库，供他人使用，并与社区分享，例如：

+   Git 社区在 [`github.com`](https://github.com)

+   CI Sparks 在 [`getsparks.org`](http://getsparks.org)

+   CI 论坛在 [`codeigniter.com/forums`](http://codeigniter.com/forums)

+   Packagist 在 [`packagist.org`](https://packagist.org)

要调用内置库，例如，调用名为 `CI_Xxxx` 的内置库，如下所示：`$this->load->library (xxxx)`；这样，就不使用 CI_ 前缀，而是使用大写库名称 `Xxxx` 的小写库名称 `xxxx`。在库 `CI_Xxxx` 内调用库函数 `yyyy` 时，应编写 `$this->xxxx->yyyy()`;。

以下是一个内置和常用 CI 库的列表（截至版本 2.1.4）：

| `CI_Benchmark` | `CI_Encrypt` | `CI_Migration` | `CI_Unit_test` |
| --- | --- | --- | --- |
| `CI_Cache` | `CI_Exceptions` | `CI_Model` | `CI_Upload` |
| `CI_Cache_apc` | `CI_Form_validation` | `CI_Output` | `CI_URI` |
| `CI_Cache_dummy` | `CI_FTP` | `CI_Pagination` | `CI_User_agent` |
| `CI_Cache_file` | `CI_Hooks` | `CI_Parser` | `CI_Utf8` |
| `CI_Cache_memcached` | `CI_Image_lib` | `CI_Profiler` | `CI_Xmlrpc` |
| `CI_Calendar` | `CI_Input` | `CI_Router` | `CI_Xmlrpcs` |
| `CI_Cart` | `CI_Javascript` | `CI_Security` | `CI_Zip` |
| `CI_Config` | `CI_Jquery` | `CI_Session` |   |
| `CI_Controller` | `CI_Lang` | `CI_SHA1` |   |
| `CI_Driver` | `CI_Loader` | `CI_Table` |   |
| `CI_Driver_Library` | `CI_Log` | `CI_Trackback` |   |

在本章中，我们将提供一个 Google Maps 第三方库包装器的使用示例，可在 [`github.com/ianckc/CodeIgniter-Google-Maps-Library`](https://github.com/ianckc/CodeIgniter-Google-Maps-Library) 找到。

在 CI 论坛 [`codeigniter.com/forums`](http://codeigniter.com/forums) 可以找到更多第三方库。

# 示例 1 – 使用内置库

在这个初始示例中，我们将看到如何使用 CI 内置库。在这里，我们将使用 CI 库`CI_Table`以及`CI_db`库，对于给定的数据库表/视图和一些可选的 CSS 设置，它将使我们能够在单行代码中渲染出所有 HTML 表格标签和 CSS 设置。在本例中，我们将使用与第三章中控制器示例相同的用户表，*控制器使用和范围*。

本示例将由以下控制器和视图构建：

+   `application/controllers/builtins.php`：此控制器加载内置 CI 库`table`以及`db`库，它是自动加载的（更多信息，请参阅第二章，*配置和命名约定*），以获取用户的表内容，并设置表格以使用`table`库渲染。

    ```php
    $this->load->library('table');

    ```

    控制器准备地图设置的向量以及每个地点和可能的控制器列表，以缩放进入每个地点，并渲染一个名为`google_map_view`的视图。

+   `application/views/users_view.php`：此视图将使用`table`库服务渲染从`db`加载的格式良好的表格，并由控制器配置。

    ### 注意

    让我们假设项目根 URL 为`http://mydomain.com/myproject`，`http://mydomain.com/myproject/builtins`。

    （源代码通过 URL 提供。）

## 控制器文件

以下是对每个操作控制器代码的逐步示例：

```php
<?php 
/** Use CI built In libraries
class Builtins extends CI_Controller{
  function __construct(){
    parent::__construct();
    // Load the table library that generates the HTML tags for// showing the table structure within a view
    $this->load->library('table');
    }
  public function index(){
    // Load the users list from DB into the view 
    $data['users'] = $this->db->get('users');
    // Create custom header for the table 
    $header = array
    ('id', 'User Name', 'Hashed Password', 'Position' );
    // Set the headings
    $this->table->set_heading($header);
    // Set table formatting 
    $table_format = array ( 'table_open'  => '<table border="1"cellpadding="2" cellspacing="1" class="mytable">' );
    $this->table->set_template($table_format);
    // Load the view and send the results
    $this->load->view('users_view', $data);
    }
  }
```

## 视图文件

为了完成操作，我们将完成视图文件的工作。

```php
<!DOCTYPE html">
<meta http-equiv="Content-type" content="text/html;charset=utf-8"/>
<html>
<head>
<title>
  Showing Users Table Using CI Build-In table Library
</title>
</head>
<body>
  <div id='results'>
  <!—All The Formatted Table is rendered by the table libraryinstance using the controller defined settings and the tableof users we have fetched from the DB >
  <?php echo $this->table->generate($users); ?>
  </div>
</body>
</html>
```

# 示例 2 – 使用第三方库，如 Google Maps CI 库包装器

在本例中，我们将看到如何安装和使用 Google Maps CI 库以及一些酷炫的服务。首先，我们需要从[`biostall.com/codeigniter-google-maps-v3-api-library`](http://biostall.com/codeigniter-google-maps-v3-api-library)下载库文件。

在下载的 TAR 文件中，我们将找到以下库：

+   `Googlemaps.php`：这是 CI 的 Google Maps API 库。我们将将其放置在`application/libraries/`。

+   `Jsmin.php`：这是库的辅助代码，用于生成启用智能 Google Maps UI 交互的 JavaScript 生成代码。我们也将将其放置在`application/libraries/`。

+   Google Maps V3 API：这是一个 PDF 文件，用于深入了解可能的库设置和使用。

在本例中，我们将提供一个初始页面，在创建的应用程序中，我们将在这个 Google 地图窗口上一起显示几个标记的地点。在那个可视视图中，我们将启用用户使用 CI 锚点 URL 助手缩放进入我们在地图上标记的预定义选定地点。

本示例将由以下库、控制器和视图构建：

+   `application/libraries/`：这是我们下载的 Google Maps CI 包装器库。请参阅 CI 库贡献者网站[`biostall.com`](http://biostall.com)。

+   `application/controllers/gmaps.php`：此控制器加载`googlemaps`库，并为 Google 地图上显示的几个地点构建几个视图，并放大到每个地点。

    ```php
    $this->load->library('googlemaps');
    ```

    控制器准备地图设置向量、地点列表以及可能的控制器，以便放大到每个地点，并渲染名为`google_map_view`的视图。

+   `application/views/google_map_view.php`：这是最初显示 Google 地图上所有地点的渲染视图，并允许用户通过菜单选项放大到列出的放大位置，或者回到放大地图上所有地点的视图。

假设项目根 URI 为`http://mydomain.com/myproject`。`http://mydomain.com/myproject/gmaps`。

### 小贴士

本书通过 URL 提供源代码。

## 控制器文件

控制器文件`controllers/gmaps.php`最初将加载 CI Google Maps 库，然后设置地图设置和要标记并显示在不同视图中的地点，控制器将具有`__construct()`和`index()`方法，并设置在定义的地点上放大。

```php
<?php
/** Use The Google Maps CI Library Wrapper for severalmarked places altogether and zoom-in*/
class Gmaps extends CI_Controller {
  function __construct()
  {  parent::__construct();
    $this->load->library('googlemaps');
    // Set the map window sizes:
    $config['map_width']	= "1000px";
    // map window width
    $config['map_height'] = "1000px";
    // map window height
    $this->googlemaps->initialize($config);
  }
  function index() {
    /* Initialize and setup Google Maps for our App startingwith 3 marked places
    London, UK, Bombai, India, Rehovot, Israel
    */
    // Initialize our map for this use case of show 3// places altogether.
    // let the zoom be automatically decided by Google for showing// the several places on one view.
    $config['zoom'] = "auto";
    $this->googlemaps->initialize($config);
    //Define the places we want to see marked on Google Map!
    $this->add_visual_flag ('London, UK');
    $this->add_visual_flag ('Bombai, India');
    $this->add_visual_flag ('Rehovot, Israel');
    $data = $this->load_map_setting ();
    // Load our view, passing the map data that has just been//created.
    $this->load->view('google_map_view', $data);
  }
  //The class Gmaps continued with several more functions as//follows:
  function london() {
    // Initialize our map
    //Here you can also pass in additional parameters for// customizing the map (see the following code:)
    // Define the address we want to be on the map center
    $config['center'] = 'London, UK'; to be on the map center
    // Set Zoom Level - Zoom 0: World – 18 Street Level
    $config['zoom'] = "16";
    $this->googlemaps->initialize($config);
    // Add visual flag
    $this->add_visual_flag ($config['center']);
    $data = $this->load_map_setting ();
    // Load our view passing the map data that has just been//created
    $this->load->view('google_map_view', $data);
  }
  functionBombay() {
  //Initialize our map.
  //Here you can also pass in additional parameters for//customizing the map (see the following code)
  //Define the address we want to see as the map center
  $config['center'] = 'Bombay, India';
  $config['zoom'] = "16";  // City Level Zoom 
  $this->googlemaps->initialize($config);
  // Add visual flag
  $this->add_visual_flag ($config['center']);
  $data = $this->load_map_setting ();
  // Load our view passing the map data that has just been created
  $this->load->view('google_map_view', $data);
}
```

类`Gmaps`继续添加以下几个函数：

```php
function rehovot()
{
  // Initialize our map.
  //Here you can also pass in additional parameters for //customizing the map (see the following code)
  $config['center'] = 'Rehovot, Israel';
  $config['zoom'] = "16";
  // City Level Zoom
  $this->googlemaps->initialize($config);
  // Add visual flag
  $this->add_visual_flag ($config['center']);
  $data = $this->load_map_setting ();
  // Load our view, passing the map data that has just been//created.
  $this->load->view('google_map_view', $data);
}
function load_map_setting ( ) {
  $data = array();
  $locations = array();
  $controllers = array();
  // Set controllers list for zoom in
  $locations[] = 'London, UK';
  $locations[] = 'Bombai, India';
  $locations[] = 'Rehovot, Israel';
  // Set controllers list for zoom in 
  $controllers[] = "london";
  $controllers[] = "bombai";
  $controllers[] = "rehovot";
  $data['map'] = $this->googlemaps->create_map();
  $data['locations'] = $locations;
  $data['controllers'] = $controllers;
  $data['map'] = $this->googlemaps->create_map();
  return $data;
}
```

类`Gmaps`继续添加以下几个函数：

```php
function add_visual_flag ( $place ) {
  $marker = array();
  // Setup Marker for the place and the title as the place name
  $marker['position'] = $place;
  $marker['title'] = $place;
  $this->googlemaps->add_marker($marker);
  }
}
```

## 视图文件

视图文件将渲染提供的 Google Maps JavaScript 和 HTML 部分，以及渲染地点列表。它还提供控制器支持的地点的放大和缩小导航选项。

```php
<!DOCTYPE html">
<meta http-equiv="Content-type"
content="text/html; charset=utf-8" />
<html>
<head>
  <script src = http://code.jquery.com/jquery-latest.js ></script>
  <!--Render all the map JS provided by rendering controller-->
  <?php echo $map['js']; ?>
</head>
<body>
<H3>CodeIgniter Powered CI Google Maps Library : <H3>
<HR/><ul>
<!—Let the User Always Get Back to the default Zoom out -->
<li><?php  echo anchor("index.php/gmaps",
'<B>See All Locations</B>' ); ?>
</li>
<?PHP 
$i=0;
foreach ($locations as $location ) {
  // Show to the user all the possible Zoom Ins defined places by//the controller, so that user may zoom in by issuing the// anchor.
  $controller = $controllers["$i"];
  $i++; ?>
  <li>
  <?php echo anchor
  ("index.php/gmaps/$controller", "Zoom-In to ".$location ) ?>
  </li>
  <?PHP } ?>
  }
</ul>
<HR></HR>
<?php echo $map['html']; ?>
</body>
</html>
```

# 示例 3 – 构建如 Flickr API 包装器之类的库

Yahoo!提供的[flickr.com](http://flickr.com)网站为社区上传的公共照片库提供 API 访问。该 API 非常丰富，其文档可在[`www.flickr.com/services/api/`](http://www.flickr.com/services/api/)找到，被称为**应用花园**。

该 API 支持多种编程语言和访问方法。我们将构建一个包装器解决方案，使其能够扩展以获取任何 Flickr API 服务，使用 PHP REST 访问方法。

本示例将由以下库、控制器和视图构建：

+   `application/libraries/flickr_wrapper.php`：这是一个 CI 包装器库，它通过 CI 实现流畅的 Flickr API 访问。这个基本服务库可以扩展以支持整个 Flickr 应用花园。

+   `application/controllers/flickr_recent.php`：这是使用我们编写的`flickr_wrapper`库的控制器，它抓取了带有 EXIF 照片信息和摄影师相关信息的最近上传的公共照片。

+   `application/views/flickr_recent_view.php`：这是显示最近照片和摄影师收集信息的视图。

假设项目根 URI 为 `http://mydomain.com/myproject`，因此执行自动登录控制器的 URI 将是 `http://mydomain.com/myproject/flickr_recent`。

## flickr_wrapper.php 库文件

`application/libraries/flickr_wrapper.php` 库文件包含我们正在构建并用于访问 Flickr App Garden API 的 `flickr_wrapper` 类库。必须使用有效的 Flickr `api_key` 加载此库，您可以通过遵循 Flickr App Garden 文档来获取该 `api_key`。库将使用 PHP REST API 访问，这样我们就可以在以后将任何 Flickr API 服务扩展到我们的库中。库的每个方法都返回一个多维键数组的结果数据。

以下是对应的代码：

```php
/**
* CodeIgniter Flickr API wrapper Library Class
*
* Enable Simple Flickr API usage 
*
* @package        CodeIgniter
* @category    Libraries
* @author        Eli Orr
* Usage:
* Via CI controller: 
* $this->load->library( 'flickr_wrapper',
* array(   'api_key'     => '<YOUR_FLICKR_API_KEY>',
* 'DEFAULT_RES' => '3000',
// filter 3000 pix 
* 'GPS_ENABLED' => FALSE ));
* $this->flickr_wrapper->set_params ( $keyed_array );
* $recent_photos = 
* $this->flickr_wrapper->flickrPhotosGetRecent ();
* $filter_photos = 
* $this->flickr_handler->
* filter_photos ($photos_to_filter);
* $user_info        = 
* $this->flickr_wrapper->flickrUserInfo ($uid);
* // $uid e.g. 72095130@N00
//.PRIVATE 
//We will use the following private functions:
private function _file_get_contents_curl($url);
private function _flickrRestAPI ($params);
private function _is_filtered_photo ($photo_rec );
*/
```

以下是我们正在构建的 `Flickr_wrapper` 类：

```php
class Flickr_wrapper {
  // parameters as part of the library instance
  private $DEFAULT_RES = 2000;
  // Width in Pixels 
  private $GPS_ENABLED = TRUE;
  // total shown photos 
  private $RECENT_PHOTOS = 500;
  // how many photos in each poll ?
  // CI instance 
  private $CI;
  // Flickr api_key to use 
  private $api_key = "" ;
  function __construct( $params = array())
  {
    // Make sure we got the api_key – otherwise exit!
    if (!isset ($params['api_key']))
    exit ('FATAL - must be constructed with api_key!');
    $this->set_params ($params);
    // Just for debugging needs, we may drop those later
    error_reporting(E_ALL);
    ini_set('display_errors', '1');
  }
  // change settings on the fly
  function set_params ( $key_array ) {
    // sets array of instance parameters 
    foreach ($key_array as $key => $val ){ 
      switch ($key) {
        case 'DEFAULT_RES': $this->DEFAULT_RES 	= $val; break;
        case 'GPS_ENABLED': $this->GPS_ENABLED 	= $val; break;
        case 'RECENT_PHOTOS': $this->RECENT_PHOTOS = $val; break;
       case 'api_key' : $this->api_key = $val; break;
       // We can add many more here.
       default: exit ("FATAL! - set_params invalid param: $key");
     }
  }
}
```

类代码继续，同时将我们的重点转向访问最新的公开照片。

```php
// Pulls recent public photos as multi-dimensional array
function flickrPhotosGetRecent () {
  #
  # build the Params for API
  #
  $params = array(
    'api_key' => $this->api_key,
    'method' => 'flickr.photos.getRecent',
    'extras' => 'o_dims,owner_name,date_taken,media,
    path_alias,url_sq,geo',
    'per_page' => $this->RECENT_PHOTOS,
    'format' => 'php_serial'
  );
  $rsp_obj = $this->_flickrRestAPI ($params);
  #
  # check if ok or successful result :
  #
  if ($rsp_obj['stat'] == 'ok') {
    # Get the array of all the photo records in this cycle 
    return $recent_photos = $rsp_obj['photos']['photo'];
  }
  else 
  # Query failed we shall return NULL to the caller 
  return NULL;
}
// Get the Photo EXIF that has a lot of info related to the// photo for a given photo id
```

类代码继续，我们将看到如何访问与图像相关的附加信息。

```php
function GetPhotoExif ($photo_id) {
  #
  # build the API URL to call
  #
  $params = array(
    'api_key' => $this->api_key,
    'method' => 'flickr.photos.getExif',
    'photo_id' => $photo_id,
    'format' => 'php_serial',
  );
  $rsp_obj = $this->_flickrRestAPI ($params);
  #
  # display the photo title (or an error if it failed)
  #
  if ($rsp_obj['stat'] == 'ok') {
    /*
    Array ([photo] => Array ([id] => 8002716747 [secret] => 559f87aea0
    [server] => 8030
    [farm] => 9
    [camera] => Casio EX-H20G
    [exif] => ... A LOT OF EXTRA INFO
    */

    $photo_camera = $rsp_obj['photo']['camera'];
    // We can add more interesting items for our app here
    $params = array
    ( 'camera'    => $photo_camera,
    'full_exif' => $rsp_obj
    // All EXIF info for the photo_id
  );
  return $params;
  }
  else // Request Failed We shall return error:
  return NULL;
}
```

让我们看看如何使用以下代码应用照片过滤：

```php
// apply photos filtering on a provided photos array
// based on the current settings
function filter_photos ($photos) {
  $filtered_photos = array();
  foreach ($photos  as $photo) {
    if ($this->_is_filtered_photo ($photo) )
    $filtered_photos[] = $photo;
  }
  return $filtered_photos;
}
function flickrUserInfo ($uid) {
  // UID e.g. : 72095130@N00
  // find info for this User
  #
  # build the API URL to call
  #
  $params = array(
    'api_key'	=> $this->api_key,
    'method'	=> 'flickr.people.getInfo',
    'user_id' 	=> $uid,
    'extras'  	=> 'contact,friend,family',
    'format'	=> 'php_serial',
  );
  $rsp_obj = $this-> _flickrRestAPI ($params);
  #
  # Check if response is OK
  #
  if ($rsp_obj['stat'] == 'ok'){ 
    // Yes! We have a good result .. let's load it to the // keyed array structure
    $real_name = @urlencode($rsp_obj['person']['realname']['_content']);
    $location = @urlencode (strtolower ($rsp_obj['person']['location']['_content']));
    $photos = @$rsp_obj['person']['photos']['count']['_content'];
    // more can be added
```

类代码继续如下：

```php
    $params = array ( 
      'name' => $real_name,
      'uid' => $uid,
      'photos' => $photos,
      'location' => $location,
      'full_info' => $rsp_obj
    );
    return $params;
  }
  else // Response failed return NULL
  return NULL;
}
// PRIVATE SECTION OF ALL PRIVATE LIBRARY METHODS 
// THAT CANNOT BE CALLED DIRECTLY FROM THE LIBRARY USER 
// This is the heart of our wrapper library that makes it easy to get 
// The Flickr API access via simple keyed array based calls and response
private function _flickrRestAPI ($params) {
  $encoded_params = array();
  foreach ($params as $k => $v){
    $encoded_params[] = urlencode($k).'='.urlencode($v);
  }
  #
  # call the API and decode the response
  #
  $url = "http://api.flickr.com/services/rest/?".implode('&', $encoded_params);
  // This will create get query URI …?param1=val1&param2=val2// and so on
  $rsp = $this->_file_get_contents_curl($url);
  return $rsp_obj = unserialize($rsp);
}
```

类代码继续如下：

```php
// This function assure we can get a url content into a buffer// it requires that a PHP curl library is installed!
private function _file_get_contents_curl($url) {
  if (! function_exists('curl_init') )
  exit ('PHP curl library is not enabled please fix!');
  $ch = curl_init();
  curl_setopt($ch, CURLOPT_HEADER, 0);
  curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1);
  curl_setopt($ch, CURLOPT_URL, $url);
  $data = curl_exec($ch);
  curl_close($ch);
  return $data;
}
private function _is_filtered_photo ($photo_rec ) {
  /*
  [o_width]   => 4416
  [latitude] => 0
  //More can be added 
  */
  // Photo width  shall be larger than  $this->DEFAULT_RES ?
  if ( 	(int) (@$photo_rec['o_width'] )  <
  (int)  $this->DEFAULT_RES  )	 return FALSE;
  // GPS info required & Found ?
  if (( $this->GPS_ENABLED && ! @$photo_rec['latitude'] ))
  return FALSE;
  // if we are here the filtered photo passed successfully
  return TRUE;
  }
}
```

## flickr_recent.php 控制器文件

`application/controllers/flickr_recent.php` 控制器文件将加载 `flickr_wrapper` API，调用其服务以获取新上传的公开照片和摄影师，并渲染一个视图来显示结果。

为了执行控制器，您应该将浏览器指向以下 URI：`http://mydomain.com/myproject/flickr_recent`。

以下是对应的控制器代码：

```php
<?php
/**
 * Flickr Recent Controller
 *
 * Provide recent uploaded public photos in flickr community
 * Enable to apply several settings and filtering
 * Enable to get photographer user profile for each photo
 * 
 * @author Eli Orr
*/
class Flickr_recent extends CI_Controller{
  function __construct()
  {
    parent::__construct();
    /* 
    Standard Libraries, database, & helper url are included in theconfigs/autoload.php
    */
    // This lines are only for debugging needs we may drop them// if things are going good
    error_reporting(E_ALL);
    ini_set('display_errors', '1');
    /* ------Loading User Defined Library------------ */
    $this->load->library
    ( 'flickr_wrapper',
    array('api_key' => '<YOUR_FLICKR_API>',
    'DEFAULT_RES' => '3000',
    // filter 3000 pix
    'GPS_ENABLED' => FALSE
  )
  );
}
```

类代码继续如下：

```php
function index () {
  $settings = array(
    'DEFAULT_RES' => '4000',  // Only 4000 pix and better
    'GPS_ENABLED' => FALSE,  // GPS Info is not mandatory
    'RECENT_PHOTOS' => 50,  // Latest 100 photo uploads
  );
  $this->flickr_wrapper->set_params ( $settings );
  $photos_to_filter = 
  $this->flickr_wrapper->flickrPhotosGetRecent ();
  $filter_photos = 
  $this->flickr_wrapper->filter_photos ($photos_to_filter);
  $data = Array();
  $data['photos'] = $filter_photos;
  $data['settings'] = $settings;
  $this->load->view('flickr_recent_view.php',$data );
  }
}
```

## flickr_recent_view.php 视图文件

`flickr_recent_view.php` 视图文件由我们之前定义的 `Flickr_recent` 控制器渲染。此控制器使用我们开发的 `flickr_wrapper` 库来获取带有相关信息的最新 Flickr 上传照片。

视图文件位于 `application/views/flickr_recent_view.php`。此视图使用 CI 解析器通过 `<?=$param ?>` 符号插入 PHP 参数。

以下是对应的代码：

```php
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8" />
<div>
<H1>Flickr Recent Uploads</H1>
<p>
<!--  Show the applied filter settings first -->
<table border="1" style='background-color:#b0c4de;' >
<tr>
  <td>Photos in Poll</td><td><?=$settings['RECENT_PHOTOS'];?></td>
</tr>
<tr>
  <td>Min. Width Filter</td><td><?=$settings['DEFAULT_RES'];?>Px</td>
</tr>
<tr>
  <td>GPS Filter</td><td><?=$settings['GPS_ENABLED'] ? "With GPS" : "With/Without GPS"; ?></td>
</tr>
</p>
<!--  For each photo show the User name, how many photos they took till now, the original size in MP (Mega Pixels) of the photos and the Time stamp when the photo was taken by the camera (mostly loaded days or even weeks/months later)
<table border="1"  style='background-color:#009900;'  >
<tr>
  <th>User Uploaded</th><th>User photos Count</th>
  <th>Photo ID</th><th>Original Size MP</th><th>Was Taken</th>
</tr>
```

类代码继续如下：

```php
<?PHP foreach($photos as $photo )
{
  // get the owner id
  $uid = $photo['owner'];
  // Get User Info
  $user_info = $this->flickr_wrapper->flickrUserInfo ($uid);
  $photos = number_format ($user_info["photos"]);
  $mp_res = (int) ((( $photo['o_width' ] * $photo['o_height'] )/ 1000000)  +  1);
  ?> 
  <tr>
    <td> <?=$photo['ownername'] ?></td>
    <td> <?=$photos ?></td>
    <td> <?=$photo['id'] ?></td>
    <td> <?=$mp_res ?></td>
    <td> <?=$photo['datetaken'] ?></td>
  </tr>
  <?PHP      } ?>
  </table>
</div>
</body>
</html>
```

# 示例 4 – LinkedIn API 包装器

在本例中，我们将构建 CI 库包装器以与 LinkedIn API 集成，以便从其中查询 LinkedIn 信息。

在这样做时存在几个挑战，其中之一是获取访问 LinkedIn 资源的令牌并访问如下数据对象：

+   LinkedIn 用户的详细信息

+   LinkedIn 用户的连接

+   LinkedIn 公司的详细信息

+   LinkedIn 公司的更新

## 要求

+   PHP 扩展 `oauth` 库必须从 [`il1.php.net/manual/en/book.oauth.php`](http://il1.php.net/manual/en/book.oauth.php) 安装。

+   我们应在 LinkedIn 开发者网络主页注册应用程序以接收 API 密钥，[`developer.linkedin.com`](http://developer.linkedin.com)。此唯一的 API 密钥用于识别我们的应用程序，以便从 LinkedIn 授权对我们的 API 调用进行访问。一旦我们注册了 LinkedIn 应用程序，我们将获得一个 API 密钥和一个密钥。为了我们应用程序的安全，我们不共享我们的密钥。有关更多信息，请参阅[`developer.linkedin.com/`](http://developer.linkedin.com/)。

## 身份验证流程图

为了认证我们的 LinkedIn 应用程序以授予访问权限，需要以下步骤。我们将把这个项目称为 LinkedIn 应用程序。

1.  LinkedIn API 客户端向 LinkedIn 发送请求。客户端通过`oauth`对象将请求发送到 LinkedIn 请求令牌 URL [`api.linkedin.com/uas/oauth/requestToken`](https://api.linkedin.com/uas/oauth/requestToken)，其中`callback URL`作为 LinkedIn API 的参数。`callback URL`参数是从 LinkedIn 授权 URL 返回的 URL，LinkedIn 用户应在其中确认 LinkedIn 应用程序所需的权限。LinkedIn 服务器响应并返回`oauth`令牌（公钥）和`oauth`令牌密钥。

    ```php
    Client > Server request token URL
    parameter: callback URL < Server returns oauth token, ouath token secret
    ```

1.  客户端使用从[`api.linkedin.com/uas/oauth/authorize ?oauth_token = oauth_token`](https://api.linkedin.com/uas/oauth/authorize%20?oauth_token%20=%20oauth_token)服务器在第一阶段返回的`oauth_token`令牌，向 LinkedIn 服务器身份验证 URL 发送请求。

    ```php
    Client > Server auth URL
    $_GET parameter: oauth token
    ```

1.  LinkedIn 服务器将 oauth 令牌、oauth 令牌密钥和`oauth_verifier`返回给客户端。

    ```php
    Client < Server
    oauth token, oauth token secret, oauth_verifier
    ```

1.  客户端向 LinkedIn 服务器访问令牌路径[`api.linkedin.com/uas/oauth/accessToken`](https://api.linkedin.com/uas/oauth/accessToken)发送请求。

    ```php
    Client > Server access token path
    parameter: oauth_verifier (from phase 2) < Server returns oauth token, ouath token secret
    ```

此示例将由以下控制器、库和视图构建：

+   `application/controllers/linkedinfo.php`: 使用 LinkedIn 库进行身份验证并显示库返回输出的控制器

+   `application/libraries/linkedin_handeler.php`: `linkedin_handler`库，它使访问 LinkedIn 资源成为可能，例如 LinkedIn 用户的详情和连接，以及公司的详情

+   `application/views/linkedin-me.php`: 显示 LinkedIn 用户详情的视图

+   `application/views/linked-connections.php`: 显示 LinkedIn 用户的连接的视图

+   `application/views/linked-company.php`: 显示公司详情的视图

+   `application/views/linked-company-updates.php`: 显示公司更新的视图

假设项目根的 URI 是`http://mydomain.com/myproject`。

因此，执行登录身份验证控制器的 URI 将是`http://mydomain.com/myproject/linkedinfo`。

## linkedin_handler.php 库文件

库文件`application/libraries/linkedin_handler.php`包含类库`linkedin_handler`。

该库包含用于验证应用程序和访问 LinkedIn 资源的函数。

以下是对应的代码：

```php
<?php

if (!defined('BASEPATH')) exit('No direct script access allowed');

// The php oauth extension is required
// For more information refer to 
// http://il1.php.net/manual/en/book.oauth.php
if(!extension_loaded('oauth')) {
  throw new Exception('Simple-LinkedIn: library not compatible with installed PECL oauth extension. Please disable this extension to use the Simple-LinkedIn library.');
  }
/*
 *   CodeIgniter LinkedIn API
 *
 *   @package CodeIgniter
 *
 *   @author  Yehuda Zadik
 *
 *
 *   Enable Simple LinkedIn API usage
 */
class Linkedin_handler {
  const LINKEDIN_API_URL = 'https://api.linkedin.com';

  private $api_key;
  private $secret_key;
  private $on_failure_url;

  // Oauth class
  public $oauth_consumer;

  // The url to return to from LinkedIn
  // authorize url in our case is// http://mydomain.com/return_from_provider
  private $callback_url;

  // The request token URL
  private $request_token_url;

  // LinkedIn authorize URL for getting the LinkedIn user // confirmation for required permissions
  private $authorize_path;

  // LinkedIn URL for getting the tokens to access // the LinkedIn URL resources
  private $access_token_path;

  // accessory variable for accessing the LinkedIn resources
  private $api_url;

  // CI instance
  private $CI;

  /*
   *  Set the class variables
   */
  private function set_varaiables() {
    $this->request_token_url = self::LINKEDIN_API_URL . '/uas/oauth/requestToken';
    $this->authorize_path = self::LINKEDIN_API_URL . '/uas/oauth/authorize';
    $this->access_token_path = self::LINKEDIN_API_URL . '/uas/oauth/accessToken';

    $this->api_url = array('people' => 'http://api.linkedin.com/v1/people/~' , 'connections' => 'http://api.linkedin.com/v1/people/~/connections', 'companies' => 'http://api.linkedin.com/v1/companies/');

    $this->CI = &get_instance();
    }
  /*
   *  Library constructor
   *
   *  Initializes the library variables
   *  and initializes oauth consumer object
   *
   *  @param $config array of the Linked configuration variables
   */
  public function __construct($config) {
    // Setting the handler's variables;
    foreach ($config as $k => $v) {
      $this->$k = $v;
      }

    // Setting all the class variables
    $this->set_varaiables();

    // Initializing the oauth consumer object
    $this->oauth_consumer = new oauth($this->api_key, $this->secret_key);

    // Enabling Oauth debug
    $this->oauth_consumer->enableDebug();

    // Checking if returned from the LinkedIn UI permission // conformation window
    if ($this->CI->input->get('oauth_verifier') || $this->CI->input->get('oauth_problem')) {
      $this->on_success();
      } elseif (!$this->CI->session->userdata('oauth_token') && !$this->CI->session->userdata('oauth_token_secret')) {
      // if session variables are not set: oauth_token, // oauth_token_secret
      // call auth to start the process of getting the tokens from // LinkedIn via the oauth consumer object
      $this->auth();
      } elseif ($this->CI->session->userdata('oauth_token') && $this->CI->session->userdata('oauth_token_secret')) {
      // if session variables are set: oauth_token, // oauth_token_secret initialize the oauth consumer with // $oauth_token, $oauth_token_secret
      $oauth_token = $this->CI->session->userdata('oauth_token');
      $oauth_token_secret = $this->CI->session->userdata('oauth_token_secret');

      // initialize oauth consumer with $oauth_token, // $oauth_token_secret
      $this->setToken($oauth_token, $oauth_token_secret);
      }
    }
  /*
   * Start the process of getting oauth token & oauth token * secret so that the user
   * redirects to a LinkedIn UI permission conformation window
   */
  public function auth()  {
    // Start communication with the LinkedIn server
    $request_token_response = $this->getRequestToken();

    $this->CI->session->set_userdata('oauth_token_secret', $request_token_response['oauth_token_secret']);

    // Get the token for the LinkedIn authorization url
    $oauth_token = $request_token_response['oauth_token'];

    $log_message = 'yuda auth getRequestToken oauth_token: : ' . $oauth_token;
    $log_message = "oauth_token_secret: " . $request_token_response['oauth_token_secret'] . "\n";
    log_message('debug', $log_message) ;

    // Redirect to the LinkedIn authorization url for getting // permissions for the app
    header("Location: " . $this->generateAuthorizeUrl($oauth_token));
    }
  /*
   * This is the method called after returning
   * from the LinkedIn authorization URL
   * The returned values from the LinkedIn authorization URL are: * oauth_token, oauth_token_secret, oauth_verifier
   * Those values are used to retrieve oauth_token, * oauth_token_secret for accessing the LinkedIn resources
   *
   */
  public function on_success() {
    if ($this->CI->input->get('oauth_problem')) {
      redirect($this->on_failure_url);
      }

    // Set the oauth consumer tokens
    $this->setToken($this->CI->input->get('oauth_token'), $this->CI->session->userdata('oauth_token_secret'));

    // Sending request to the LinkedIn access_token_path to // receive the array, which it's keys are tokens: oauth_token, // oauth_token_secret for accessing the LinkedIn resources
    $access_token_reponse = $this->getAccessToken($this->CI->input->get('oauth_verifier'));

    // Setting the session variables with the tokens: oauth_token, // oauth_token_secret for accessing the LinkedIn resources
    $this->CI->session->set_userdata('oauth_token', $access_token_reponse['oauth_token']);
    $this->CI->session->set_userdata('oauth_token_secret',$access_token_reponse ['oauth_token_secret']);

    // Redirecting to the main page
    redirect('');
    }

  /*
   * This method sends the request token to LinkedIn
   *
   * @return array keys: oauth_token, oauth_token_secret
   */
  public function getRequestToken() {
    // The LinkedIn request token url
    $request_token_url = $this->request_token_url;

    // The LinkedIn app permissions
    $request_token_url = "?scope = r_basicprofile+r_emailaddress+r_network";

    // Getting the response from the LinkedIn request token URL.
    // The method returns the response, which is an array// with the following keys: oauth_token, oauth_token_secret
    return $this->oauth_consumer->getRequestToken($request_token_url, $this->callback_url);
    }
  /*
   * This method returns the LinkedIn authorize URL
   *
   * @param $oauth_token string oauth token for the LinkedIn * authorzation URL
   *
   * @return string URL of the LinkedIn authorization URL
   */
  public  function generateAuthorizeUrl($oauth_token) {
    return $this->authorize_path . "?oauth_token = " . $oauth_token;
    }
  /*
   * This method sets the token and secret keys of
   * the oauth object of the oauth protocol
   *
   * @param $oauth_token string oauth token
   * @param $oauth_token_secret oauth_token_secret
   *
   */
  public function setToken($oauth_token, $oauth_token_secret) {
    $this->oauth_consumer->setToken($oauth_token, $oauth_token_secret);
    }
  /*
   * This method requests the LinkedIn tokens for
   * accessing the LinkedIn resources
   * It returns an array with the following keys: oauth_token, * oauth_token_secret
   *
   * @param $oauth_verifier string
   *
   * @return array Array with the following keys: *  oauth_token, oauth_token_secret,
   * which are used to access the LinkedIn resources URL
   */
  public function getAccessToken($oauth_verifier) {
    try {
      // Returns an array with the following keys: // oauth_token, oauth_token_secret
      // These keys are used to access the LinkedIn // resources URL
      return $this->oauth_consumer->getAccessToken($this->access_token_path, '', $oauth_verifier);
      } catch(OAuthException $E) {
      echo "<pre>";var_dump($this->oauth_consumer);
      echo "</pre><br><br>";
      echo "Response: ". $E->lastResponse;
      exit();
      }
    }
  /*
   * This function returns a LinkedIn user's details
   * It returns a JSON string containing these values
   *
   * @return $json string String containing user's details
   */
  public function me() {
    $params = array();
    $headers = array();
    $method = OAUTH_HTTP_METHOD_GET;
    $api_url = $this->api_url['people'] . '?format = json';

    try {
      // Request for a LinkedIn user's details
      $this->oauth_consumer->fetch ($api_url, $params, $method, $headers);

      // Receiving the last response with json // containing the user's details
      $s_json = $this->oauth_consumer->getLastResponse();
      return $s_json;
      } catch(OAuthException $E) {
      echo "<pre>";var_dump($this->oauth_consumer);
      echo "</pre><br><br>";
      echo "Response: ". $E->lastResponse;
      exit();
      }
    }
  /*
   * This function returns a LinkedIn user's connections
   * It returns a JSON string containing these values
   *
   * @return $json string String containing user's connections
   */
  public function connections() {
    $params = array();
    $headers = array();
    $method = OAUTH_HTTP_METHOD_GET;
    $api_url = $this->api_url['connections'] . '?count = 10&format = json';

    try {
      // Request for a LinkedIn user's connections
      $this->oauth_consumer->fetch($api_url, $params, $method, $headers);

      // Receiving the last response with json containing the user's // connections
      $s_json = $this->oauth_consumer->getLastResponse();
      return $s_json;
      } catch(OAuthException $E) {
      echo "<pre>";var_dump($this->oauth_consumer);
      echo "</pre><br><br>";
      echo "Response: ". $E->lastResponse;
      exit();
      }
    }
  /*
   * This function returns a LinkedIn company' details
   * It returns a JSON string containing these values
   *
   * @param Integer $company_id - company id
   *
   * @return $json string String containing a company' details
   */
  public function company($company_id) {
    $params = array();
    $headers = array();
    $method = OAUTH_HTTP_METHOD_GET;
    $api_url = $this->api_url['companies'] . $company_id;

    // The following company's details are required: // company_id, number of employees, foundation year, // number of the company's followers
    $api_url = ':(id, name, website-url, twitter-id, employee-count-range, specialties, founded-year, num-followers)?format = json';

    try {
      // Request for a LinkedIn company's details
      $this->oauth_consumer->fetch($api_url, $params, $method, $headers);

      // Receiving the last response with json containing the // company's details
      $s_json = $this->oauth_consumer->getLastResponse();
      return $s_json;
      } catch(OAuthException $E) {
      echo "<pre>";var_dump($this->oauth_consumer);
      echo "</pre><br><br>";
      echo "Response: ". $E->lastResponse;
      exit();
      }
    }
  /*
   * This function returns a LinkedIn company' three updates
   * It returns a JSON string containing these values
   *
   * @param Integer $company_id - company id
   *
   * @return $json string String containing company's three updates
   */
  public function company_updates($company_id) {
    $params = array();
    $headers = array();
    $method = OAUTH_HTTP_METHOD_GET;
    $api_url = $this->api_url[ 'companies'] . $company_id . '/updates?start = 0 & count = 3 & format = json';

    try {
      // Request for a LinkedIn company's three updates
      $this->oauth_consumer->fetch($api_url, $params, $method, $headers);

      // Receiving the last response with json // containing company's three updates
      $s_json = $this->oauth_consumer->getLastResponse();
      return $s_json;
      } catch(OAuthException $E) {
      echo "<pre>"; var_dump($this->oauth_consumer);
      echo "</pre><br><br>";
      echo "Response: ". $E->lastResponse;
      exit();
      }
    }
  }
// Class closing tags
/*  End of file linkedin.php */
/* Location: ./application/libraries/linkedin_handler.php */
```

## 链接 info.php 控制器文件

控制器文件`application/controllers/linkedinfo.php`将加载 LinkedIn API，调用其服务，并渲染视图以显示结果。

以下是对应的控制器代码：

```php
<?php
if (!defined('BASEPATH')) exit('No direct script access allowed');

/**
 * *
 * The controller is loading our developed library * LinkedIn (wrapper)
 * Next, the following process will occur in the loaded library.
 * 1 – get oauth token & oauth token secret so that the user * will be redirected to a LinkedIn UI permission conformation * window to approve our requested permission.
 * 2 – If user confirms the permissions we requested, * the method onSuccess is called with the * oauth token & oauth token secret as $_GET parameters.* The tokens will be stored as session parameters. * Else we cannot proceed querying LinkedIn and the onFailure.
 *
 * Now we can access the LinkedIn resources using the retrieved .*.tokens.
 * Here are the methods that query LinkedIn resources: * me() – Get the Info of the User who confirmed the permissions
 * connections() - Get the preceding user connection records JSON * formatted
 * company() – We just gave an example how to retrieve any company * by company id we got from the results or query company * id by company id or search criteria
 * company_updates() – Let us get the latest updates of this * company
 */
class Linkedinfo extends CI_Controller {
  // array of LinkedIn configuration variables
  private $linkedin_config;

  // callback url from the LinkedIn authorization URL
  private $callback_url;
  /*
   * Controller constructor
   *
   * Checks if session variables are set: oauth_token, * oauth_token_secret
   * If they are set, then it initializes the oauth consumer
   * else it will call the method auth() to start the * process of setting the token
   * It also loads the LinkedIn library
   */
  public function __construct() {

    parent::__construct();

    $linked_config = array(
      // Application keys registered in the // LinkedIn developer app
      ‹api_key› => ‹esq76cosbm9x›, ‹secret_key› => ‹TyUQ2FzRRzWz9bHk›,// The url to return from the // LinkedIn confirmation URL
        ‹callback_url› => base_url() . ‹linkedinfo/on_success›,// The URL when the failure occurs
          ‹on_failure_url› => ‹linkedinfo/on_failure›);

    // Load the LinkedIn library
    $this->load->library(‹linkedin_handler›, $linked_config);
    }
  /*
   * Load the main menu of the application
   */
  public function index() {
    $this->load->view(‹linkedin-menu›);
    }
  /*
   * This is the method called after returning* from the LinkedIn authorization URL
   * The returned values from the LinkedIn authorization URL are: * oauth_token, oauth_token_secret, oauth_verifier
   * Those values are used to retrieve oauth_token, * oauth_token_secret for accessing the LinkedIn resources
   *
   *
   */
  public function onSucess() {
    // Set the oauth consumer tokens
    $this->linkedin->setToken($this->input->get(‹oauth_token›), $this->session->userdata(‹oauth_token_secret›));

    // Sending the request to the LinkedIn access_token_path to 
    // receive the array, which it's keys
    // are tokens: oauth_token, oauth_token_secret for // accessing the LinkedIn resources
    $access_token_reponse = $this->linkedin->getAccessToken($this->input->get('oauth_verifier'));

    // Setting the session variables with the tokens: oauth_token, // oauth_token_secret for accessing the LinkedIn resources
    $this->session->set_userdata(‹oauth_token›, $access_token_reponse[‹oauth_token›]);
    $this->session->set_userdata(‹oauth_token_secret›, $access_token_reponse[‹oauth_token_secret›]);

    // Redirecting to the main page
    redirect(‹›);
    }
  /*
   *
   */
  /*
   * This function calls the library method me to get
   * the LinkedIn user›s details
   */
  public function me() {
    // Get the LinkedIn user›s details
    $s_json = $this->linkedin->me();
    $o_my_details = json_decode($s_json);
    $prodile_url = $o_my_details->siteStandardProfileRequest->url;

    $view_params[‹my_details›] = $o_my_details;
    $view_params[‹profile_url›] = $prodile_url;

    // Load the view for displaying the LinkedIn user›s details
    $this->load->view(‹linkedin-me›, $view_params);
    }
  /*
   * This function calls the library method me to get
   * the LinkedIn user›s connections
   */
  public function connections() {
    // Get the LinkedIn user›s connections
    $s_json = $this->linkedin->connections();
    $o_json = json_decode($s_json);

    // Processing data received from the LinkedIn library
    $a_connections = array();
    for ($index = 0; $index < $o_json->_count; $index++) {
      if ($o_json->values[$index]->id == ‹private›) {
        continue;
        }

      if (isset($o_json->values[$index]->pictureUrl)) {
        $picture_url = $o_json->values[$index]->pictureUrl;
        } else {
        $picture_url = ‹› ;
        }

      $a_connections[] = array(‹picture_url› => $picture_url, ‹name› => $o_json->values[$index]->firstName . « «. $o_json->values[$index]->lastName, ‹headline› => $o_json->values[$index]->headline, ‹industry› => $o_json->values[$index]->industry, ‹url› => $o_json->values[$index]->siteStandardProfileRequest->url);
      }

    $view_params[‹connections_count›] = $o_json->_total;
    $view_params[‹connections›] = $a_connections;

    // Load the view for displaying the LinkedIn user›s // connections
    $this->load->view(‹linked-connections›, $view_params);
    }
  /*
   * This function the calls library method me to get
   * the LinkedIn company›s details
   *
   * @param $company_id integer - Linkedin company id
   */
  public function companies($company_id) {
    // Get the LinkedIn company›s details
    $s_json = $this->linkedin->company($company_id);
    $o_company_details = json_decode( $s_json);

    $a_company_details = array (‹id› => $company_id, ‹name› => $o_company_details->name, ‹specialties› => $o_company_details->specialties->values, ‹websiteUrl› => $o_company_details->websiteUrl, ‹employeeCountRange› => $o_company_details->employeeCountRange->name, ‹foundedYear› => $o_company_details->foundedYear, ‹numFollowers› => $o_company_details->numFollowers);

    // Load the view for displaying the LinkedIn company›s // details
    $view_params = $a_company_details;
    $this->load->view(‹linked-company›, $view_params);
    }
  /*
   * This function calls the library method me to get
   * the LinkedIn company›s updates
   *
   *  @param $company_id integer - Linkedin company id
   */
  public function company_updates($company_id) {
    // Get the LinkedIn company›s updates
    $s_json = $this->linkedin->company_updates($company_id);
    $o_company_updates = json_decode( $s_json);

    // Processing the data received from the LinkedIn library
    $a_updates = array();
    $a_json_updates = $o_company_updates->values;
    for ($index = 0; $index < count($a_json_updates);$index++) {
        $o_update = $a_json_updates[$index];

        if (isset($o_update->updateContent->companyJobUpdate)) {
          $a_updates[] = array(‹type› => ‹Job Update›, ‹position› => $o_update->updateContent->companyJobUpdate->job->position->title, ‹url› => $o_update->updateContent->companyJobUpdate->job->siteJobRequest->url);
        }
      }

    // Load the view for displaying the LinkedIn // company›s updates
    $view_params[‹updates›] = $a_updates;
    $this->load->view(‹linked-company-updates›, $view_params);
    }
  } // class closing tags
/* End of the file linkedinfo.php */
/* Location: ./application/controllers/linkedinfo.php */
```

## 链接 linkedin-me.php 视图文件

此视图文件显示 LinkedIn 用户的详细信息。

以下是对应的视图代码：

```php
<!DOCTYPE html>
<html lang = "en">
<head>
  <meta charset = "utf-8">
  <title>My Details</title>
</head>
<body>
<table>
<tr>
  <td>Full Name:</td>
  <td><?php echo $my_details->firstName . « « . $my_details->lastName ; ?></td>
</tr>
<tr>
  <td>Title</td>
  <td><?php echo $my_details->headline ; ?></td>
</tr>
<tr>
  <td>My LinkedIn profile</td>
  <td><a href = «<?php echo $profile_url ?>» target = «_blank»>Link</a> </td>
</tr>
</table>

<div>
  <p><a href = «<?php echo site_url(‹›) ; ?>»>Back to Menu</a> </p>
</div>
</body>
</html>
```

视图文件 linked-connections.php

此视图文件显示 LinkedIn 用户的联系。

以下是对应的视图代码：

```php
<!DOCTYPE html>
<html lang = "en">
<head>
  <meta charset = "utf-8">
  <title>My Connections</title>
</head>
<body>
<h1>My Total connections: <?php echo $connections_count ; ?></h1>
<div>
  <p><a href = «<?php echo site_url(‹›) ; ?>»>Back to Menu</a> </p></div>
<table>
<tr>
  <td>Picture</td>
  <td>Name</td>
  <td>Headline</td>
  <td>Industry</td>
</tr>
  <?php foreach ($connections as $connection): ?>
<tr>
  <td><img src = «<?php echo $connection[‹picture_url›]; ?>»> </td>
  <td><a href = «<?php echo $connection[‹url›];?>» target = «_blank»><?php echo $connection[‹name›] ?></a></td>
  <td><?php echo $connection[‹headline›]; ?></td>
  <td><?php echo $connection[‹industry›]; ?></td>
</tr>
<?php endforeach; ?>
</table>
</body>
</html>
```

视图文件 linked-company.php

此视图文件显示 LinkedIn 公司的详细信息。

以下是对应的视图代码：

```php
<!DOCTYPE html>
<html lang = "en">
<head>
  <meta charset = "utf-8">
  <title>Company</title>
</head>
<body>

<div>
  <p><a href = «<?php echo site_url(‹›); ?>»>Back to Menu</a></p>
</div>

<table>
<tr>
  <td>Name</td>
  <td><?php echo $name; ?></td>
</tr>
<tr>
  <td>Founded</td>
  <td><?php echo $foundedYear; ?></td>
</tr>
<tr>
  <td>employeeCountRange</td>
  <td><?php echo $employeeCountRange; ?></td>
</tr>
<tr>
  <td>Specialties<td>
  <td>
    <ul>
      <?php foreach ($specialties as $specialty): ?>
      <li><?php echo $specialty; ?></li>
      <?php endforeach; ?>
    </ul>
  </td>
</tr>
<tr>
  <td>Website</td>
  <td><a href = «<?php echo $websiteUrl; ?>»>Website</a></td>
</tr>
<tr>
  <td>numFollowers</td>
  <td><?php echo $numFollowers; ?></td>
</tr>
</table>
<div style = «margin-top: 10px;»>
  <a href = «<?php echo site_url(‹linkedinfo/company_updates/7919›); ?>»>Updates</a>
</div>
</body>
</html>
```

视图文件 linked-company-updates.php

此视图文件显示 LinkedIn 公司的三个更新。

以下是对应的视图代码：

```php
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset = "utf-8">
    <title>Company</title>
</head>
<body>
<div>
  <p><a href = "<?php echo site_url('') ; ?>">Back to Menu</a> </p>
</div>
<table>
  <?php foreach ($updates as $update): ?>
  <tr>
    <td>
      <ul>
        <?php foreach ($update as $key => $val): ?>
        <li><?php echo $key; ?>: <?php echo $val; ?></li>
        <?php endforeach; ?>
      </ul>
    </td>
  </tr>
  <?php endforeach; ?>
</table>
</body>
</html>
```

# 摘要

在本章中，我们回顾了 CI 库的作用域、不同类型的内置 CI 输出系统第三方库，以及如何在我们的项目中构建自己的库。我们还回顾了在项目中加载和使用库资源的方法。最终，我们创建了几个使用示例。
