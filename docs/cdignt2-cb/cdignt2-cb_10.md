# 第十章. 处理图片

在本章中，你将学习：

+   在 MAC 上使用 Cactuslab 安装 ImageMagick

+   使用 CodeIgniter 上传图片

+   生成缩略图 – 调整大小

+   旋转图片

+   裁剪图片

+   使用文字添加水印

+   使用图像叠加添加水印

+   使用 CodeIgniter CAPTCHA 提交表单

# 简介

CodeIgniter 提供了一系列有用的工具来帮助你以图像处理类的方式操作和修改图片；它不是 Photoshop，但对于日常 Web 开发中的大多数需求来说已经足够好了。它具有帮助您上传图片、调整大小、创建缩略图、添加水印、裁剪和旋转的功能——这些都是非常有用的事情，正是你在开发环境中所追求的。以下是操作方法。

# 在 MAC 上使用 Cactuslab 安装 ImageMagick

CodeIgniter 图像处理类的某些功能需要 GD2，然而，其他功能需要 ImageMagick。如果你在 MAC 上使用 MAMP，那么默认情况下你很可能没有安装它。Cactuslab 已经制作了一个安装程序，它会为你完成这项工作。

## 如何操作...

1.  访问 URL [`www.cactuslab.com/imagemagick`](http://www.cactuslab.com/imagemagick)。

1.  下载安装程序。在撰写本文时，最新版本是适用于 Mac OS X 10.5 - 10.8 的 ImageMagick 6.8.6-3。

1.  运行安装程序，如果一切顺利，你现在应该已经安装了 ImageMagick。你需要将`$config['library_path']`的值设置为`/opt/ImageMagick/bin`，如下所示：

    ```php
    'library_path' => '/opt/ImageMagick/bin'
    ```

## 它是如何工作的...

安装程序会为你处理所有事情：这是巫术！

# 使用 CodeIgniter 上传图片

这与书中前面提到的文件上传配方类似；然而，它与它不同，因为我们正在使 CodeIgniter 上传图片（而不是上传任何文件类型）并在图片上执行特定任务，这些任务与书中其他上传示例不相关。因此，请将其视为一个单独的上传脚本。此脚本是该章中其他配方的基础脚本（除了 CAPTCHA 配方之外），也就是说，旋转、水印、调整大小等配方需要此基础脚本才能运行。

## 如何操作...

我们将创建以下两个文件：

+   `/path/to/codeigniter/application/controllers/upload.php`

+   `/path/to/codeigniter/application/views/upload/upload.php`

1.  创建控制器文件，`upload.php`，并将以下代码添加到其中：

    ```php
    <?php if (! defined('BASEPATH')) exit('No direct script access allowed');

    class Upload extends CI_Controller {
      function __construct() {
        parent::__construct();
        $this->load->helper('form');
        $this->load->helper('url');
      }

      function index() {
        $this->load->view('upload/upload', array('error' => ' ' ));
      }

      function do_upload() {
        $config['upload_path'] = '/path/to/upload/dir/';
        $config['allowed_types'] = 'gif|jpg|png';
        $config['max_size'] = '10000';
        $config['max_width'] = '1024';
        $config['max_height'] = '768';

        $this->load->library('upload', $config);

        if (! $this->upload->do_upload()) {
          $error = array('error' => $this->upload->display_errors());
          $this->load->view('upload/upload', $error);
        } else {
    $data = array('upload_data' => $this->upload->data());
          echo '<pre>';
          var_dump($data);
          echo '</pre>';
             }
      }  
    }
    ```

1.  创建`/path/to/codeigniter/application/views/upload/upload.php`文件，并将以下代码添加到其中：

    ```php
    <?php if ($error) : ?>
    <?php echo $error ; ?>
    <?php endif ; ?>

    <?php echo form_open_multipart('upload/do_upload');?>

    <input type = "file" name = "userfile" size = "20" />

    <br /><br />

    <input type = "submit" value = "upload" />

    <?php echo form_close() ; ?>
    ```

## 它是如何工作的...

当在浏览器中运行`upload`控制器时，用户会看到表单（该表单位于视图文件`views/ipload/upload.php`中）。用户选择一个图片并按下**提交**按钮，然后调用公共函数`do_upload()`。我们立即定义一些设置，上传的图片将根据这些设置进行检查，例如允许的图片类型、最大大小和尺寸，这些是：

```php
$config['upload_path'] = '/path/to/upload/dir';
$config['allowed_types'] = 'gif|jpg|png';
$config['max_size'] = '10000';
$config['max_width'] = '1024';
$config['max_height'] = '768';
```

如果上传的图片符合这些要求，图片可以被移动到由 `$config['upload_path']` 指定的地方进行存储，以便在需要时使用。

# 生成缩略图 – 调整大小

显然，拥有生成缩略图的功能是非常有用的。大多数网络开发者都曾有过在当前上传的图片或之前上传的图片中生成缩略图的需求。通常，这种处理会直接使用 PHP 或你可能使用的任何编程语言来完成；但 CodeIgniter 给你提供了轻松创建缩略图的能力，这就是你如何做到的。

## 准备工作

我们将使用自己的库来完成这项工作。如果你还没有这样做（在本章的其他菜谱中），创建以下文件：

+   `/path/to/codeigniter/application/libraries/image_manip.php`

1.  确保按照以下方式定义 `Image_manip` 库类：

    ```php
    <?php if (! defined('BASEPATH')) exit('No direct script access allowed');
    class Image_manip {
    }
    ```

1.  还要确保你已经安装了图像库 GD2，并且已经复制并准备好了本章的“基础”菜谱——即 *使用 CodeIgniter 上传图片*——因为本菜谱使用 *使用 CodeIgniter 上传图片* 的代码作为基础菜谱。

## 如何操作...

我们将修改之前菜谱中的以下文件，*使用 CodeIgniter 上传图片*：

+   `/path/to/codeigniter/application/controllers/upload.php`

+   `/path/to/codeigniter/application/libraries/image_manip.php`

1.  将以下行（粗体）添加到构造函数中，使整个构造函数看起来像以下代码片段：

    ```php
    function __construct() {
    	parent::__construct();
    	$this->load->helper('form');
    	$this->load->helper('url');
    	$this->load->library('image_manip');
    }
    ```

1.  修改 `do_upload()` 函数，将其更改为以下内容：

    ```php
    if ( ! $this->upload->do_upload()) {
    	$error = array('error' => $this->upload->display_errors());
    	$this->load->view('upload/upload', $error);
    } else {
    	$data = array('upload_data' => $this->upload->data());

    	$result = $this->upload->data();
    	$origional_image = $result['full_path'];

    	$data = array(
    		'image_library' => 'gd2',
    		'source_image' => $origional_image,
    		'create_thumb' => TRUE,
    		'maintain_ratio' => TRUE,
    		'width' => '75',
    		'height' => '50'
    		);

    	if ($this->image_manip->resize_image($data)) {
    		echo 'Image successfully resized<br /><pre>';
    		var_dump($result);
    		echo '</pre>';
    	} else {
    		echo 'There was an error with the image processing.';
    	}
    }
    ```

1.  修改 `image_manip` 库，添加以下函数：

    ```php
      function resize_image($data) {
        $CI =& get_instance();
        $CI->load->library('image_lib', $data);
        if ($CI->image_lib->resize()) {
          return true;
        } else {
          echo $CI->image_lib->display_errors();
        }
      }
    ```

## 它是如何工作的...

当在浏览器中运行 `upload` 控制器时，用户会看到一个表单（该表单位于视图文件 `views/ipload/upload.php` 中）。用户选择一张图片，然后按下 **提交** 按钮，之后调用 `public function do_upload()`。我们立即定义一些设置，用于检查上传的图片，例如允许的图片类型、最大尺寸和维度——就像我们在 *使用 CodeIgniter 上传图片* 菜谱中所做的那样。假设上传成功且没有错误，我们调用 `image_manip` 库中的 `resize_image()` 函数：

```php
  $this->image_manip->resize_image($data);
```

`resize_image()` 函数获取主 CodeIgniter 对象 `$CI` 并加载其自己的 `image_lib` 库，如下面的代码片段所示：

```php
  $CI = & get_instance();
  $CI->load->library('image_lib', $data);
```

`image_lib` 库将由 CodeIgniter 使用，通过我们在 `$data` 数组中提供的参数对图片进行更改。

我们调用 `$CI->image_lib->resize()`，检查返回的 `TRUE` 值。如果返回 `FALSE`，我们则返回操作中的任何错误消息。否则，将创建一个缩略图，如下面的代码片段所示：

```php
  if ($CI->image_lib->resize()) {
    return true;
  } else {
    echo $CI->image_lib->display_errors();
  }
```

# 旋转图片

CodeIgniter 允许旋转图片；如果你需要垂直或任何其他方向的翻转，这很有用。以下是操作方法。

## 准备工作

我们将要使用我们自己的库来完成这个任务。如果你还没有这样做（在本章的其他菜谱中），创建以下文件：

+   `/path/to/codeigniter/application/libraries/image_manip.php`

1.  确保将`image_manip`库类定义为以下形式：

    ```php
    <?php if ( ! defined('BASEPATH')) exit('No direct script access allowed');
    class Image_manip {
    }
    ```

1.  还要确保你已经安装了图像库 ImageMagik，并且已经将本章的“基础”菜谱——即*使用 CodeIgniter 上传图片*——复制并准备好，因为这个菜谱使用*使用 CodeIgniter 上传图片*的代码作为基础菜谱。

## 如何操作...

我们将要修改以下两个文件：

+   `/path/to/codeigniter/application/controllers/upload.php`

+   `/path/to/codeigniter/application/libraries/image_manip.php`

1.  将以下函数添加到`image_manip.php`库文件中：

    ```php
      function rotate($data) {
        $CI = & get_instance();
        $CI->load->library('image_lib', $data);

        $CI->image_lib->initialize($data); 

        if ($CI->image_lib->rotate()) {
          return true;
        } else {
          echo $CI->image_lib->display_errors();
        }
      }
    ```

1.  将以下行（加粗）添加到构造函数中，使整个构造函数看起来如下：

    ```php
    function __construct() {
      parent::__construct();
      $this->load->helper('form');
      $this->load->helper('url');
      $this->load->library('image_manip');
    }
    ```

1.  修改`do_upload()`函数，将其修改为以下内容：

    ```php
    if ( ! $this->upload->do_upload()) {
    	$error = array('error' => $this->upload->display_errors());
    	$this->load->view('upload/upload', $error);
    } else {
    	$data = array('upload_data' => $this->upload->data());

    	$result = $this->upload->data();
    	$original_image = $result['full_path'];

    	$data = array(
    		'image_library' => 'imagemagick',
    		'library_path' => '/opt/ImageMagick/bin',
    		'source_image' => $original_image,			
    		'rotation_angle' => 'vrt'
    		);
    		if ($this->image_manip->rotate($data)) {
    		echo 'Image successfully rotated<br /><pre>';
    		var_dump($result);
    		echo '</pre>';
    	} else {
    		echo 'There was an error with the image processing.';
    	}
    }
    ```

## 它是如何工作的...

当在浏览器中运行上传控制器时，用户会看到一个表单（该表单位于视图`views/ipload/upload.php`文件中）。用户选择一个图片并点击**提交**按钮，然后调用`public function do_uplaod()`。我们立即定义一些设置，用于检查上传的图片，例如允许的图片类型、最大大小和尺寸——就像我们在*使用 CodeIgniter 上传图片*菜谱中所做的那样。假设上传成功且没有错误，我们从上传数据中获取`full_path`：

```php
	$original_image = $result['full_path'];
```

将其分配为局部变量`$original_image`。然后我们定义一个数组（`$data`），包含所有 CodeIgniter 需要裁剪图片的配置设置（确保获取`library_path`正确）。我们将这个`$data`数组传递给库函数`rotate`：

```php
	$this->image_manip->rotate($data);
```

这将在图片上执行旋转操作。

# 裁剪图片

这将最有用且相关，当与前端机制结合使用时，允许用户选择图片的一个区域；然而，我包括这段代码在这里，因为你可能需要它。你永远不知道！

## 准备工作

我们将要使用我们自己的库来完成这个任务。如果你还没有这样做（在本章的其他菜谱中），创建以下文件：

+   `/path/to/codeigniter/application/libraries/image_manip.php`

1.  确保将`image_manip`库类定义为以下形式：

    ```php
    <?php if (! defined('BASEPATH')) exit('No direct script access allowed');
    class Image_manip {
    }
    ```

1.  还要确保你已经安装了图像库 ImageMagik。

    ### 注意

    如果您在 MAC 上使用 MAMP，那么您默认可能没有安装 ImageMagick。有一个过程可以在 MAMP 上安装 ImageMagick；然而，有一个更快的方法。Cactuslab 提供了一个安装程序，网址为[`www.cactuslab.com/imagemagick`](http://www.cactuslab.com/imagemagick)，它工作得很好。本章中也有一个*使用 Cactuslab 在 MAC 上安装 ImageMagick*的食谱，它解释了安装过程。

1.  确保你已经复制并准备好这个章节的“基础”食谱，即*使用 CodeIgniter 上传图片*，因为这个食谱使用*使用 CodeIgniter 上传图片*作为基础食谱。

## 如何做...

我们将修改以下两个文件：

+   `/path/to/codeigniter/application/controllers/upload.php`

+   `/path/to/codeigniter/application/libraries/image_manip.php`

1.  将以下行（加粗）添加到构造函数中，使整个构造函数看起来如下：

    ```php
    function __construct() {
    	parent::__construct();
    	$this->load->helper('form');
    	$this->load->helper('url');
    	$this->load->library('image_manip');
    }
    ```

1.  修改`do_upload()`函数，改为以下内容：

    ```php
    if ( ! $this->upload->do_upload()) {
    	$error = array('error' => $this->upload->display_errors());
    	$this->load->view('upload/upload', $error);
    } else {
    	$data = array('upload_data' => $this->upload->data());

    	$result = $this->upload->data();
    	$original_image = $result['full_path'];

    	$data = array(
    		'source_image' => $original_image,
    		'image_library' => 'imagemagick',
    		'library_path' => '/opt/ImageMagick/bin',
    		'x_axis' => '100',
    		'y_axis' => '60'	
    	);

    	if ($this->image_manip->crop_image($data)) {
    		echo 'Image successfully cropped<br /><pre>';
    		var_dump($result);
    		echo '</pre>';
    	} else {
    		echo 'There was an error with the image processing.';
    	}
    }
    ```

1.  修改`image_manip`库中的以下函数：

    ```php
      function crop_image($data) {
        $CI =& get_instance();
        $CI->load->library('image_lib', $data);

        $CI->image_lib->initialize($data); 

        if ($CI->image_lib->crop()) {
          return true; 
        } else {
          echo $CI->image_lib->display_errors();
        }
      } 
    ```

## 它是如何工作的...

当在浏览器中运行`upload`控制器时，用户会看到一个表单（位于视图`views/ipload/upload.php`文件中）。用户选择一个图片并点击**提交**按钮，然后调用`public function do_uplaod()`。我们立即定义一些设置，用于检查上传的图片，例如允许的图片类型、最大大小和尺寸——就像我们在*使用 CodeIgniter 上传图片*食谱中所做的那样。假设上传成功且没有错误，我们从上传数据中获取`full_path`：

```php
  $original_image = $result['full_path'];
```

我们将其分配为一个局部变量`$original_image`。然后我们定义一个数组`$data`，包含所有 CodeIgniter 要求裁剪图片的配置设置（确保获取`library_path`变量正确）。我们将这个`$data`数组传递给库函数`crop_image`，如下面的代码片段所示：

```php
 $this->image_manip->crop_image($data);

```

这将在图片上执行裁剪操作。

### 可能的错误

在编写这个食谱的过程中，您可能会看到一些或所有（或者可能是完全不同的）错误信息。以下是一些错误及其可能的解决方案（如果其他方法都失败了，请尝试 Google 搜索）：

**错误**：图片处理失败。请验证您的服务器支持所选协议，并且您的图片库路径正确。

**可能的解决方案**：可能是您的图片库路径不正确或图片库的配置设置错误。请验证您是否安装了正确的库并且路径正确。为此，请执行以下步骤：

1.  在您的终端中输入`cd /usr/X11R6/bin`。

1.  然后输入`ls`（或者在 Windows 上输入`dir`）。

1.  在那里寻找 ImageMagick 库。如果您看不到它，那么它可能尚未安装，您需要安装它才能执行裁剪操作。

你如何安装 ImageMagick？嗯，互联网上有许多说明和教程可以帮助你。然而，如果你使用 MAMP，请访问本章中的*在 MAC 上使用 Cactuslab 安装 ImageMagick*配方，并使用安装程序来帮助安装 ImageMagick 并完成配置工作。

然而，请注意，库路径不会像 CodeIgniter 文档中那样是`/use/X11R6/bin/`（带有尾随斜杠），而是`/opt/ImageMagick/bin`（不带尾随斜杠）。

# 使用文本添加水印

添加水印可以是一个有用的方式来标记图片上的版权（只是确保你是版权所有者）。CodeIgniter 提供了一个简单的方法来给图片添加水印。水印可以是文本或图像叠加，并且可以放置在原始图片的任何位置。以下是如何添加文本水印的描述。

## 准备工作

我们将使用我们自己的库来完成这项工作。如果您还没有这样做（在本章的其他配方中），请创建以下文件：

+   `/path/to/codeigniter/application/libraries/image_manip.php`

1.  确保将`image_manip`库类定义为以下形式：

    ```php
    <?php if (! defined('BASEPATH')) exit('No direct script access allowed');
    class Image_manip {
    }
    ```

1.  还要确保您已经安装了图像库，GD2。

1.  确保您已经复制并准备好了本章的基础配方*使用 CodeIgniter 上传图片*，因为这个配方使用*使用 CodeIgniter 上传图片*作为基础配方。

## 如何实现...

我们将修改以下两个文件：

+   `/path/to/codeigniter/application/controllers/uplod.php`

+   `/path/to/codeigniter/application/libraries/image_manip.php`

1.  将以下函数添加到`image_manip.php`库文件中：

    ```php
      function do_watermark($data) {
        $CI =& get_instance();
        $CI->load->library('image_lib', $data);

        $CI->image_lib->initialize($data); 
        if ($CI->image_lib->watermark()) {
          return true;
        } else {
          echo $CI->image_lib->display_errors();
        }
      }
    ```

1.  将以下行（加粗）添加到构造函数中，使整个构造函数看起来如下：

    ```php
    function __construct() {
      parent::__construct();
      $this->load->helper('form');
      $this->load->helper('url');
     $this->load->library('image_manip');
    }
    ```

1.  修改`do_upload()`函数，将其更改为以下内容：

    ```php
    if ( ! $this->upload->do_upload()) {
      $error = array('error' => $this->upload->display_errors());
      $this->load->view('upload/upload', $error);
    } else {
      $data = array('upload_data' => $this->upload->data());

      $result = $this->upload->data();
      $original_image = $result['full_path'];

      $data = array(
      'image_library' => 'GD2','source_image' => $original_image,'wm_text' => 'Copyright (c) ' . date("Y",time()) . ' - YOUR NAME','wm_type' => 'text','wm_font_path' => './system/fonts/texb.ttf',
        'wm_font_size' => '16','wm_font_color' => 'ffffff','wm_vrt_alignment' => 'middle','wm_hor_alignment' => 'left','wm_padding' => '20'
      );

      if ($this->image_manip->do_watermark($data)) {
        echo 'Image successfully watermarked<br /><pre>';
        var_dump($result);
        echo '</pre>';
      } else {
        echo 'There was an error with the image processing.';
      }
    }
    ```

## 工作原理...

当在浏览器中运行`upload`控制器时，用户会看到一个表单（该表单位于视图文件`views/ipload/upload.php`中）。用户选择一张图片并点击**提交**按钮，随后调用`public function do_uplaod()`。我们立即定义一些设置，用于检查上传的图片，例如允许的图片类型、最大尺寸和维度，就像我们在*使用 CodeIgniter 上传图片*配方中所做的那样。假设上传成功且没有错误，我们从上传数据中获取`full_path`：

```php
  $original_image = $result['full_path'];
```

将其分配为局部变量，`$original_image`。接下来，我们将定义一个数组（`$data`），包含所有必要的设置，以允许 CodeIgniter 在我们的上传图片上执行水印叠加。以下表格中我将介绍一些有趣的设置：

| 设置 | 选项 | 我们如何应用它 |
| --- | --- | --- |
| `wm_type` | `text`, `overlay` | 在这个菜谱中，它设置为 text，这告诉 CodeIgniter 它必须在图像上写文本，而不是调用图像作为叠加。在下一个菜谱中，我们将查看叠加水印。 |
| `wn_vrt_alignment` | `top`, `middle`, `bottom` | 我们告诉 CodeIgniter 它应该将文本放置在上传图像的中间。 |
| `wn_hor_alignment` | `left`, `center`, `right` | 我们告诉 CodeIgniter 它应该将文本放置在上传图像的左侧。 |
| `wm_font_color` | 任何十六进制值（有关有用 URL 的提示，请参阅以下内容） | 我们将文本写成白色，没有其他原因，只是因为我用来测试这段代码的图像相当暗——一个浪漫的日落（啊）——但您当然可以将其更改为您想要的任何十六进制值。 |
| `wm_font_path` | `./system/fonts/texb.ttf` | 这是 CodeIgniter 附带的自带字体；它有点工业风格，您可能想更换另一个，要么将不同的真型字体复制到`./system/fonts/`目录中，要么链接到该目录之外的字体。 |

以下 URL 有一个十六进制颜色值的列表：[`www.w3schools.com/html/html_colors.asp`](http://www.w3schools.com/html/html_colors.asp)。

# 添加图像叠加水印

CodeIgniter 可以根据前面的菜谱以文本形式添加水印，但 CodeIgniter 还可以通过在基础图像上叠加水印图像来实现。以下是操作方法...

## 准备工作

我们将使用自己的库来完成这个任务。如果您还没有这样做（在本章的其他菜谱中），请创建以下文件：

+   `/path/to/codeigniter/application/libraries/image_manip.php`

1.  确保将`image_manip`库类定义为以下内容：

    ```php
    <?php if ( ! defined('BASEPATH')) exit('No direct script access allowed');
    class Image_manip {
    }
    ```

1.  这个菜谱基于*添加文本水印*菜谱。请确保您已经遵循了那个菜谱。我们将对其进行一些代码更改，以帮助我们实现水印叠加。

## 如何做...

我们将要修改以下文件：

+   `/path/to/codeigniter/application/controllers/uplod.php`

+   `/path/to/codeigniter/application/libraries/image_manip.php`

1.  将以下函数添加到库文件`image_manip.php`中：

    ```php
      function do_watermark($data) {
        $CI =& get_instance();
        $CI->load->library('image_lib', $data);

        $CI->image_lib->initialize($data); 
        if ($CI->image_lib->watermark()) {
          return true;
        } else {
          echo $CI->image_lib->display_errors();
        }
      }
    ```

1.  将以下行（加粗）添加到构造函数中，以便整个构造函数看起来像这样：

    ```php
    function __construct() {
      parent::__construct();
      $this->load->helper('form');
      $this->load->helper('url');
     $this->load->library('image_manip');
    }
    ```

1.  修改`do_upload()`函数，将其更改为以下内容：

    ```php
    if (! $this->upload->do_upload()) {
      $error = array('error' => $this->upload->display_errors());
        $this->load->view('upload/upload', $error);
      } else {
        $data = array('upload_data' => $this->upload->data());

        $result = $this->upload->data();
        $original_image = $result['full_path'];

        // Create Watermark
        $data = array(
        'image_library' => 'gd2','source_image' => $original_image, 'wm_type' => 'overlay','wm_overlay_path' => $config['upload_path'] . '/overlay_image_name',
        'wm_font_path' => './system/fonts/texb.ttf','wm_font_size' => '16','wm_font_color' => 'ffffff','wm_vrt_alignment' => 'middle','wm_hor_alignment' => 'left','wm_padding' => '20'
        );

        $this->image_manip->do_watermark($data);
      }
    ```

## 它是如何工作的...

这与*添加文本水印*菜谱的基本功能相同。然而，与`wm_type`设置为`text`不同，我们将其设置为`overlay`。我们添加了配置数组项`wm_overlay_path`，并将其设置为存储叠加图像的位置（在这种情况下，我们将叠加图像放置在上传文件夹中；当然，您可以在系统上的任何位置移动它，但这里是为了保持简单）。我们还删除了数组项`wm_text`，现在它不再需要（然而，如果您愿意，可以保留它，它不会干扰图像叠加）。

# 使用 CodeIgniter CAPTCHA 提交表单

有时，除了转义和验证用户输入之外，还需要在表单中添加更多的安全性；有时你可能希望确保输入数据并提交表单的是人类而不是某些脚本或机器人。

一种经过验证和测试的方法是**CAPTCHA**。CAPTCHA 有其他替代方案；例如，一个数学问题（比如 10 + 7 是什么）在你的应用程序中相当容易构建。一种新方法是让用户玩一个简短的游戏。根据他们的表现，他们会被评估为人类或机器人；[areyouahuman.com](http://areyouahuman.com) 是一个很好的资源。但到目前为止，我们将专注于 CodeIgniter 的 CAPTCHA 功能，为我们创建一个受 CAPTCHA 保护的表单。

## 准备工作

我们将把 CodeIgniter 为我们生成的 CAPTCHA 信息存储在数据库中的一个表中。为了做到这一点，我们首先需要创建这个表。以下是在数据库中执行此操作的 MySQL 代码。将以下内容复制到您的数据库中：

```php
CREATE TABLE `captcha` (`captcha_id` bigint(13) unsigned NOT NULL AUTO_INCREMENT,`captcha_time` int(10) unsigned NOT NULL,`ip_address` varchar(16) NOT NULL DEFAULT '0',`word` varchar(20) NOT NULL,PRIMARY KEY (`captcha_id`),KEY `word` (`word`)) ENGINE=InnoDB  DEFAULT CHARSET=latin1 AUTO_INCREMENT=1 ;
```

## 如何实现...

我们将创建以下四个文件：

+   `/path/to/codeigniter/application/controllers/comments.php`: 这是一个控制器，它帮助处理用户提交的姓名、电子邮件地址和评论，并且还会调用一个助手来处理 CAPTCHA 数据

+   `/path/to/codeigniter/application/helpers/make_captcha_helper.php`: 这个助手包含了生成 CAPTCHA 图像所需的代码

+   `/path/to/codeigniter/application/models/captcha_model.php`: 这个模型用于检查用户提交的 CAPTCHA 值与数据库中存储的值是否匹配

+   `/path/to/codeigniter/application/views/comments/post_form.php`: 这个视图文件将显示表单（姓名、电子邮件、评论等）和 CAPTCHA 图像

1.  创建 `controllers/comments.php` 控制器文件，并将以下代码添加到其中：

    ```php
    <?php if ( ! defined('BASEPATH')) exit('No direct script access allowed');

    class Comments extends CI_Controller {
      function __construct() {
        parent::__construct();
          $this->load->helper('form');
          $this->load->helper('url');
          $this->load->helper('make_captcha');
          $this->load->model('Captcha_model');
      }

      public function index() {
          $this->load->library('form_validation');
          $this->form_validation->set_error_delimiters('', '<br />');

          $this->form_validation->set_rules('name', 'Name', 'required|max_length[225]');
          $this->form_validation->set_rules('email', 'Email', 'required|max_length[225]');
          $this->form_validation->set_rules('message', 'Message', 'required|max_length[225]');
          $this->form_validation->set_rules('captcha', 'Captcha', 'required|max_length[225]');

          // Begin validation
          if ($this->form_validation->run() == FALSE) {
            $data['img'] = make_captcha();
            $this->load->view('comments/post_form', $data);
          } else {
            $expiration = time() - 7200;

            $this->Captcha_model->delete_expired($expiration);

            $data = array(
              'captcha' => $this->input->post('captcha'),
              'ip_address' => $this->input->ip_address(),
              'expiration' => $expiration);

            $num_rows = 
              $this->Captcha_model->does_exist($data);

            if ($num_rows == 0) {
              $data['error'] = "Type the word in the image.";
              $data['img'] = make_captcha();
              $this->load->view('comments/post_form', $data);
            } else {
              echo 'CAPTCHA OKAY - HERE IS YOUR POST:' . '<br />';
              echo $this->input->post('name') . '<br />';
              echo $this->input->post('email') . '<br />';
              echo $this->input->post('message') . '<br />';
          }
        }
      }
    }
    ```

1.  接下来创建 `helpers/make_captcha_helper.php` 助手文件，并将以下代码添加到其中：

    ```php
    <?php if (! defined('BASEPATH')) exit('No direct script access allowed');

    function make_captcha() {
      $CI =& get_instance();
      $CI->load->helper('captcha');

      $vals = array(
          'img_path' => '/file/path/to/image/captcha/',
          'img_url' => 'http://url/to/image/captcha/',
          'font_path' => './system/fonts/texb.ttf',
          'img_width' => '150',
          'img_height' => 30,
          'expiration' => 7200
          );

      $cap = create_captcha($vals);
      $data = array(
          'captcha_time' => $cap['time'],
          'ip_address' => $CI->input->ip_address(),
          'word' => $cap['word']
          );

      $query = $CI->db->insert_string('captcha', $data);
      $CI->db->query($query);

      return $cap['image'];
    }
    ```

    ### 小贴士

    `img_path` 和 `img_url` 是这里两个有趣的设置。`img_path` 应该是文件系统上图片文件夹的路径，而 `img_url` 应该是图片在网页浏览器中显示的路径。CodeIgniter 使用 `img_url` 来构建一个 HTML `img` 标签，并且这个标签会被发送到 `post_form` 作为 `$data['img']`。

1.  创建 `models/captcha_model.php` 模型文件，并将以下代码添加到其中：

    ```php
    <?php if (! defined('BASEPATH')) exit('No direct script access allowed');

    class Captcha_model extends CI_Model {
        function __construct() {
            parent::__construct();
        }

        public function delete_expired($expiration) {
            $this->db->where('captcha_time < ',$expiration);
            $this->db->delete('captcha');
        }

        public function does_exist($data) {
            $this->db->where('word', $data['captcha']);
            $this->db->where('ip_address', $data['ip_address']);
            $this->db->where('captcha_time > ', $data['expiration']);
            $query = $this->db->get('captcha');
            return $query->num_rows();
        }
    }
    ```

1.  然后创建 `comments/post_form.php` 视图文件，并将以下代码添加到其中：

    ```php
    <?php echo form_open() ; ?>
      <?php echo validation_errors() ; ?>
      <?php if (isset($errors)) { echo $errors ; }  ?>
      <br />
      Name <input type = "text" name = "name" size = "5" value = "<?php echo set_value('name') ; ?>"/><br />
      Email <input type = "text" name = "email" size = "5" value = "<?php echo set_value('email') ; ?>"/><br />
      Message <br />
      <textarea name = "message" rows = "4" cols = "20" /><?php echo set_value('message') ; ?></textarea><br />

      Please enter the string you see below
      <input type = "text" name = "captcha" value = "" />
      <br />
      <?php echo $img ; ?>
      <br />
      <input type = "submit" value = "Submit" />
    <?php echo form_close() ; ?>
    ```

## 它是如何工作的...

评论控制器加载 `public function index()`，它设置了当用户提交表单时的验证环境。由于 `$this->form_validation->run()` 将等于 `FALSE`（因为表单尚未提交），所以会调用 `make_captcha_helper` 函数，即 `make_captcha()`，并将它的返回值发送到 `comments/post_form` 视图，如下面的代码片段所示：

```php
        if ($this->form_validation->run() == FALSE) {
          $data['img'] = make_captcha();
          $this->load->view('comments/post_form', $data);
        } else {
  ... etc
```

`make_captcha_helper` 函数，`make_captcha()`，将获取主要的 CodeIgniter 对象，如下面的代码片段所示：

```php
  $CI = & get_instance();
  $CI->load->helper('captcha');
```

它定义了构建 CAPTCHA 图像所需的价值，如下面的代码片段所示：

```php
  $vals = array(
    'img_path' => '/file/path/to/image/',
    'img_url' => 'http://url/to/image/',
    'font_path' => './system/fonts/texb.ttf',
    'img_width' => '150',
    'img_height' => 30,
    'expiration' => 7200
    );
```

这些值存储在 `$data` 数组中，该数组传递给 CodeIgniter 辅助程序 `captcha`，它返回给我们 `$cap` 数组。`$cap` 传递给数据库函数 `insert_string()`，以便将 CAPTCHA 信息保存到数据库中，如下面的代码片段所示：

```php
  $cap = create_captcha($vals);
  $data = array(
    'captcha_time' => $cap['time'],
    'ip_address' => $CI->input->ip_address(),
    'word' => $cap['word']
    );

  $query = $CI->db->insert_string('captcha', $data);
```

我们将使用数据库中的这一行与用户提交表单时输入的数据进行比较。

最后，`make_captcha` 辅助程序返回一个 HTML `img` 标签字符串到我们的评论控制器。这被保存在 `$data` 数组中，并传递到 `post_form` 视图文件。

然后用户会看到一个 HTML 表单，其中包含姓名、电子邮件、评论输入，以及一个 CAPTCHA 图像和一个用于输入看到的 CAPTCHA 字符串的文本框。用户完成表单，仔细输入他们的数据以及 CAPTCHA 图像中的字符串，然后点击 **提交** 按钮。

假设验证通过（没有表单错误），则评论控制器将开始将用户输入的 CAPTCHA 字符串与由 `make_captcha` 辅助程序在数据库中创建的字符串进行比较。

它通过首先清理数据库中的旧 CAPTCHA 行（在这个例子中，旧的是任何超过两小时的数据）来启动这个过程；它是通过定义当前时间（作为一个 Unix 时间戳）减去两小时（或 `7200` 秒）来做到这一点的，这被设置为 `$expiration` 时间，如下面的代码片段所示：

```php
  $expiration = time() - 7200;

  $this->Captcha_model->delete_expired($expiration);
```

调用 `Captcha_model` 函数 `delete_expired()`，并将过期时间传递给它。这个模型函数将删除数据库中 `captcha_time` 小于过期时间的行，如下面的代码片段所示：

```php
    public function delete_expired($expiration) {
        $this->db->where('captcha_time < ',$expiration);
        $this->db->delete('captcha');
    }
```

一旦从数据库中删除了旧的 CAPTCHA，就创建并填充了包含用户的 CAPTCHA 输入、他们的 IP 地址以及再次的 `$expiration` 时间（我们用来删除旧行的那个）的 `$data` 数组。这个 `$data` 数组传递给 `Captcha_model` 函数 `does_exist()`。这个模型函数将检查用户输入的 CAPTCHA 字符串是否存在于数据库中，如果是，则有效（即小于两小时且与提供的 IP 地址匹配）。模型函数返回找到的行数，如下面的代码片段所示

```php
    public function does_exist($data) {
        $this->db->where('word', $data['captcha']);
        $this->db->where('ip_address', $data['ip_address']);
        $this->db->where('captcha_time > ', $data['expiration']);
        $query = $this->db->get('captcha');
        return $query->num_rows();
    }
```

如果没有行存在，则 `$data['errors']` 被赋予一个错误消息。再次调用 `make_captcha()`，生成一个新的 CAPTCHA 图像并发送到 `post_form` 视图，错误消息显示在新的 CAPTCHA 图像上方。然后系统等待用户再次填写表格并再次尝试。

然而，如果结果不是零，那么用户输入的 CAPTCHA 字符串就是正确的，因此我们会向他们显示一条快速消息并回显他们的输入。实际上，你可以在这里做你喜欢的事情，比如处理他们的消息并将其保存到博客订阅源，或者将他们重定向到网站上的另一个区域，无论你想要什么。
