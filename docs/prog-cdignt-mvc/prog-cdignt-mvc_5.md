# 第五章。助手

本章涵盖了 CI 助手主题，不同类型的助手及其不同的使用类别，并提供了几个网络应用程序的代码示例。CI 为我们提供了内置助手，使我们能够集成第三方助手，并允许我们开发新的助手，如果愿意，还可以与社区分享。CI 助手通过允许所有其他 CI 控制器使用相同的代码而不是在本地定义助手函数，从而提高了 CI 的效率和代码的可重用性。

助手文件是一组特定类别的独立过程函数集合。每个助手函数执行一个特定的任务，不依赖于其他函数。本章将通过几个示例详细阐述 CI 助手的理念、定义和用法。

文件夹 `system/helpers` 包含 CI 系统的内置助手。文件夹 `application/helpers` 包含 CI 助手的所有附加文件。它们可以是第三方助手或由开发者创建。

本章将主要关注：

+   CI 助手的范围和用法

    +   使用类别

    +   使用助手

    +   将助手添加到项目中

    +   加载助手

    +   使用助手方法

+   可用的 CI 助手

+   示例

    +   示例 1：使用内置助手

    +   示例 2：使用第三方助手——SSL 助手

    +   示例 3：构建我们自己的助手——`my_download` 助手

我们将首先简要回顾 CI 框架中助手的定义，以及我们如何在整个项目代码资源中利用它。

# CI 助手的范围和用法

CI 助手默认情况下无法访问控制器资源，除非调用并使用 CI 和 `get_instance()` 来访问 CI 资源。

我们可以使用 CI 系统的第三方助手扩展 CI 助手，或者我们可以开发自己的助手。

任何应用程序助手都应位于项目目录下的 `application/helpers/` 下。

助手文件必须采用以下格式：

```php
  application/helpers/<HELPER_NAME>_helper.php
```

例如，SSL 助手文件应显示为 `application/helpers/ssl_helper.php`。

在 CI 项目中，助手集成和使用的如下：

+   将助手代码资源添加到 `application/helpers/myhelper_helper.php`

+   自动加载助手或通过控制器加载

    +   自动为所有 CI 项目加载助手 `myhelper` 如下：

        ```php
        $autoload['helper'] = array('url','myhelper');

        ```

    +   对于加载特定控制器、构造函数或方法，请使用以下方法：

        ```php
        $this->load->helper('myhelper'); 

        ```

+   使用以下助手方法：

    ```php
    $result = $this->myhelper->called_method($param1, aram2);

    ```

## 可用的 CI 助手

CI 和 CI 开发者社区网络提供了许多助手，覆盖了丰富的主题。我们将回顾 CI 助手以及第三方 CI 助手的常用资源。

我们还被鼓励构建自己的助手，供他人使用，并与以下社区分享：

+   `Git` 社区：[`github.com`](https://github.com)

+   CI 论坛 [`codeigniter.com/forums/`](http://codeigniter.com/forums/)

### CI 系统助手

CI 内置助手的列表如下（它们可以在 `CI 目录树` 中找到，通过访问 `system/helpers/`）：

+   数组助手

+   验证码助手

+   Cookie 助手

+   日期助手

+   目录助手

+   下载助手

+   邮件助手

+   文件助手

+   表单助手

+   HTML 助手

+   变词助手

+   语言助手

+   数字助手

+   路径助手

+   安全助手

+   表情符号助手

+   字符串助手

+   文本助手

+   字体助手

+   URL 助手

+   XML 助手

### CI 第三方助手

+   `ssl_helper.php`

+   `html_manipulator_helper.php`

# 示例 1 – 使用内置助手

在本例中，我们将看到如何使用 CI 内置助手。对于本例，我们将使用 URL 助手来生成链接。URL 助手文件包含辅助处理 URL 的函数。我们将使用 URL 助手函数 `site_url()`，它返回在配置文件中指定的站点 URL。

本例将由以下任一控制器构建：

+   `application/controllers/ helperexample1.php`

    此控制器加载了内置的 CI URL 助手。

    ```php
    $this->load->helper('url'); 

    ```

    控制器渲染了一个名为 **helper-example1-view** 的视图

+   `application/views/helper-example1-view.php`

    此视图将使用 URL 助手在视图文件中生成链接

    假设项目根目录的 URL 如下所示：`http://mydomain.com/myproject`。`http://mydomain.com/myproject/helperexample1`

    ### 小贴士

    本书通过 URL 提供源代码。

## 控制器文件

现在，我们将看到控制器如何加载内置的 CI URL 助手，以便视图文件能够使用 URL 助手函数 `site_url`，该函数生成链接。

```php
class Helperexample1 extends CI_Controller {
  /**
   * Index Page for this controller.
   *
   * Maps to the following URL
   *     http://example.com/index.php/welcome
   *  - or -  
   *     http://example.com/index.php/welcome/index
   *  - or -
   * Since this controller is set as the default controller in 
   * config/routes.php, it's displayed at http://example.com/
   *
   * So any other public methods not prefixed with an underscore
       *  will
   * map to /index.php/welcome/<method_name>
   * @see http://codeigniter.com/user_guide/general/urls.html
   */
    public function index()
    {
  	      	  // Loading the url helper
  	      	  $this->load->helper('url');
         $this->load->view('helper-example1-view');
    }
  }
/* End of file helperexample1.php */
/* Location: ./application/controllers/helperexample1 */

```

## 视图文件

视图文件调用 URL 助手函数 `site_url`。由于控制器已加载 URL 助手，因此它被视图识别。

```php
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="utf-8">
  <title>Menu</title>
</head>
<body>
<table>
<tr>
<td><a href="<?php echo site_url('welcome'); ?>">Welcome</a></td>
</tr>
<tr>
<td><a href="<?php echo site_url('example2/more/1/2/3');      ?>">Example2</a></td>
</tr>
</table>
</body>
</html>
```

# 示例 2 – SSL 助手

在本例中，我们将使用 CI 第三方 SSL 助手来强制 CI 和浏览器之间的 https 或 HTTP URI 请求和响应。本例将由以下助手构建：

+   `application/helpers/ssl_helper.php`：实现 SSL 的 CI 助手。

+   `application/controllers/helpersslexample.php`：此控制器加载助手并在链接上实现 SSL。助手在构造函数中加载。

    ```php
    $this->load->helper('ssl'); 

    ```

+   `application/views/helper-ssl-view.php`：这是实现了 SSL 的渲染视图。

假设项目根目录的 URI 为 `http://mydomain.com/myproject`。`http://mydomain.com/myproject/helpersslexample`

### 小贴士

本书通过 URL 提供源代码。

## 助手文件

此 CI 助手文件实现了前面章节中描述的服务。此助手使用内置的 CI URL 库和 URL 助手，通过使用重定向 CI URL 助手函数。

```php
  <?php if ( ! defined('BASEPATH')) exit('No direct script access  allowed');
  if (!function_exists('force_ssl')) {function force_ssl(){ // get the CI instance to access the CI resources  $CI =& get_instance();// Change the base_url to have https prefix  
      $CI->config->config['base_url'] =str_replace('http://', 'https://',$CI->config->config['base_url']);
            if ($_SERVER['SERVER_PORT'] != 443){     // redirect CI to use https URI
        // so that ($CI->uri->uri_string() return 
        // the current URI with https prefix
              redirect($CI->uri->uri_string());}}}
  if (!function_exists('remove_ssl')) {function remove_ssl(){$CI =& get_instance();
      // Change the base_url to have http prefix  
      $CI->config->config['base_url'] =str_replace('https://', 'http://',$CI->config->config['base_url']);
      if ($_SERVER['SERVER_PORT'] != 80){// redirect CI to use http URI
        // so that ($CI->uri->uri_string() return 
        // the current URI with http prefix
              redirect($CI->uri->uri_string());}}}
```

## 控制器文件

现在，我们将看到控制器如何加载 SSL 助手并调用其函数 `force_ssl` 以强制浏览器进行 HTTPS URI 请求和响应。

```php
class Helpersslexample extends CI_Controller {
    public function __construct() {
      parent::__construct();
      // Loading the ssl helper
      $this->load->helper('ssl');		
      // Enforce URI request of https 
      force_ssl();
    }  
    /**
      * Index Page for this controller.
      *
      */
    public function index()
    {
      $this->load->helper('url');
      $this->load->view('helper-ssl-view');
    }
  }
/* End of file helpersslexample.php */
/* Location: ./application/controllers/helpersslexample */

```

## 视图文件

视图文件代码如下：

```php
  <!DOCTYPE html>
  <html lang="en">
  <head>
  <meta charset="utf-8">
    <title>Menu</title>
  </head>
  <body>
  <table>
  <tr>
    <td>
    <a href="<?php echo site_url('welcome'); ?>">
    Welcome - You are using https
    </a>
    </td>
  </tr>
  <tr>
     <td><a href="<?php echo site_url('example2/more/1/2/3'); 	   
           ?>">Example2</a></td>
  </tr>
  </table>
  </body>
  </html>
```

# 示例 3 – 创建自己的助手

此示例使用辅助函数下载一个非常大的文件，大小为 200 MB，该文件无法一次性读取。

此示例将由以下辅助函数构建：

+   `application/helpers/my_download_helper.php`：这是用于下载一个非常大文件的 CI 辅助函数

+   `application/controllers/classg2.php`：这是使用 `my_download` 辅助函数的控制文件

+   `application/views/classg2view.php`：这是包含文件下载链接的视图

假设项目根目录的 URI 为 `http://mydomain.com/myproject`。因此，执行登录的 `auth` 控制器的 URI 将是 `http://mydomain.com/myproject/classg2`。

### 小贴士

本书通过 URL 提供了源代码。

## 辅助函数文件

此辅助函数用于下载非常大的文件，这些文件无法一次性读取。其函数 `download_large_files` 在每次循环中读取 1 MB，直到下载整个文件。

```php
  <?php  if ( ! defined('BASEPATH')) exit('No direct script access           allowed');
  /*** CodeIgniter Download Helpers** @package  CodeIgniter* @subpackage Helpers* @category	Helpers* @author Yehuda Zadik*/ 
    // --------------------------------------------------------------
  	/*** Download large files** Generates headers that force a download to happen** @access  public* @param	string	$fullPath* @return	void*/ 
    function download_large_files($fullPath){
    // File Exists? if( file_exists($fullPath) ){ 
      // Parse Info / Get Extension $fsize = filesize($fullPath); $path_parts = pathinfo($fullPath); $ext = strtolower($path_parts["extension"]); 
      // Determine Content Type 
      switch ($ext) { 
        case "pdf": $ctype = "application/pdf"; break; 
        case "exe": $ctype = "application/octet-stream";
         break; 
        case "zip": $ctype = "application/zip";
          break; 
        case "doc": $ctype = "application/msword"; break; 

        case "xls":$ctyp = "application/vnd.ms-excel";
             break;
        case "ppt": $ctype = "application/vnd.ms-powerpoint";
           break; 

        case "wmv": $ctype = "video/x-ms-wmv";
             break;
               case "gif": $ctype = "image/gif";
             break;
               case "png": $ctype = "image/png";
               break;

        case "jpeg":case "jpg": $ctype = "image/jpg"; 
                break;

        default: $ctype = "application/force-download"; 
      } 
      $file_handle = fopen($fullPath, "rb");          
      header('Content-Description: File Transfer');
        header("Content-Type: " . $ctype);            header('Content-Length: ' . $fsize);

        header('Content-Disposition: attachment; filename=' .
                 basename($fullPath));
      while(!feof($file_handle)) {$buffer = fread($file_handle, 1*(1024*1024));
        echo $buffer;          ob_flush();flush();    //These two flush commands seem to             have helped with performance
      }    
      fclose($file_handle);
    } else{die('File Not Found');
    } 
  }
  /* End of file my_download_helper.php *//* Location: ./application/helpers/my_download_helper.php */
```

## 控制器文件

控制器加载了辅助函数 `my_download` 并调用其函数 `download_large_files,` 以便用户能够使用 `my_download` 辅助函数下载原本无法下载的大文件。

```php
  <?php 
  class Classg2 extends CI_Controller {
    public function index()
    {
      $this->load->helper('url');  
      $this->load->view('classg2view');
      }  
    function download(){
      // Loading the helpers url, my_download
      $this->load->helper(array('url', 'my_download'));        // FCPATH is a constant that Codeigniter sets which       // contains the absolute path to index.php 
      $fullPath = FCPATH . 'files/movie-classg2.wmv';
      // Using the helper my_download function to download       // a very large filedownload_large_files($fullPath);
    }
  }  
  /* End of file classg2.php *//* Location: ./application/controllers/classg2.php */
```

## 视图文件

视图文件显示包含下载非常大型文件链接的数据。

```php
<!DOCTYPE html>
<html lang="en">
<head>
 <meta charset="utf-8">
 <title>Download large file</title>
</head>
<body>
<div id="container">
 <a href="<?php echo base_url("classg2/download")              ?>">Download large file</a>
</div>
</body>
</html>

```

# 摘要

在本章中，我们回顾了 CI 辅助函数、作用域、不同类型的内置 CI 系统辅助函数、第三方辅助函数以及如何在我们的项目中构建自己的辅助函数。我们还回顾了在项目中加载和使用辅助函数的步骤。最后，我们看到了几个相关使用示例，如下：

+   示例 1：使用内置辅助函数

+   示例 2：使用第三方辅助函数— SSL 辅助函数

+   示例 3：构建我们自己的辅助函数—`my_download` 辅助函数
