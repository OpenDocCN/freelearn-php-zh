# 第十章. 进一步探索

在本章中，我们将尝试涵盖一些在这本书中没有使用的内容。2015 年 4 月，Phalcon 发布了 2.0 版本。你不必担心，因为它与之前学到的内容完全兼容。

与之前版本相比，2.0 版本完全用 Zephir 语言（[`www.zephir-lang.com/`](http://www.zephir-lang.com/)）重写。如果你想升级到 2.0.*版本。

本章我们将涵盖以下主题：

+   使用 Phalcon 上传文件

+   使用注解路由器

# 使用 Phalcon 上传文件

使用 Phalcon 上传文件非常简单。我们只需要检查请求对象是否有文件，并将其移动到我们的上传目录。让我们在`Backoffice`模块中创建以下控制器：

```php
<?php
namespace App\Backoffice\Controllers;

use App\Core\Forms\MediaForm;

class MediaController extends BaseController {
  public function addAction() {
    $this->view->form = new MediaForm();
  }

  public function uploadAction() {
    if (true == $this->request->hasFiles() && $this->request->isPost()) {
      $upload_dir = __DIR__ . '/../../../public/uploads/';

      if (!is_dir($upload_dir)) {
        mkdir($upload_dir, 0755);
      }
      foreach ($this->request->getUploadedFiles() as $file) {
        $file->moveTo($upload_dir . $file->getName());
        $this->flashSession->success($file->getName().' has been successfully uploaded.');
      }

      $this->response->redirect('media/add');
    }
  }
}
```

`uploadAction()`方法首先检查请求对象是否有文件，并且请求方法是`POST`。我们将上传目录的路径分配给`$upload_dir`变量。然后我们检查这个目录是否在 public 中存在，如果不存在，则创建它。接下来，我们将每个上传的文件移动到`public/uploads/`。你可以在这个章节的源代码中找到这个示例的表单和视图。`file`对象有一些内置方法，非常有帮助：

+   `$file->getSize();`

+   `$file->getRealType();`

+   `$file->getName()`

使用这些方法，我们可以为图像实现一个简单的验证器。假设我们只接受不超过 1MB 的 JPEG 文件。下面是`uploadAction()`方法改进版本的样子：

```php
<?php

class MediaController extends BaseController {
  private $valid_mime = [
    'image/jpeg'
  ];

  private $max_size = 125000;

  public function uploadAction() {
    if (true == $this->request->hasFiles() && $this->request->isPost()) {
      $upload_dir = __DIR__ . '/../../../public/uploads/';

      if (!is_dir($upload_dir)) {
        mkdir($upload_dir, 0755);
      }

      foreach ($this->request->getUploadedFiles() as $file) {

        if (!in_array($file->getRealType(), $this->valid_mime)) {
          $this->flashSession->error($file->getName().' is invalid');
          continue;
        }

        if ($file->getSize() > $this->max_size) {
          $this->flashSession->error($file->getName().' is too big');
          continue;
        }

        $file->moveTo($upload_dir . $file->getName());
        $this->flashSession->success($file->getName().' has been successfully uploaded.');
      }

      $this->response->redirect('media/add');
    }
  }
}
```

Phalcon 还支持图像处理。不幸的是，这部分内容没有文档说明，但你可以查看官方仓库[`github.com/phalcon/cphalcon/tree/master/ext/phalcon/image`](https://github.com/phalcon/cphalcon/tree/master/ext/phalcon/image)以了解可用方法，或者查看 IDE 存根的源代码[`github.com/phalcon/phalcon-devtools/tree/master/ide/1.3.4/Phalcon/Image`](https://github.com/phalcon/phalcon-devtools/tree/master/ide/1.3.4/Phalcon/Image)。

图像处理的简单示例可以如下所示：

```php
$image = new Phalcon\Image\Adapter\GD($file);
$image->resize(200, 200)
if ($image->save()) {
  $this->flashSession->success('Image has been successfully resized');
}
```

我们还可以使用外部库，例如[`github.com/avalanche123/Imagine`](https://github.com/avalanche123/Imagine)，你可以在[`imagine.readthedocs.org/en/latest/usage/introduction.html`](http://imagine.readthedocs.org/en/latest/usage/introduction.html)找到它，那里有很好的文档说明。

# 使用注解路由器

在这本书中，我们使用了配置文件来设置路由器。如果你来自 Symfony，例如，你可能想使用注解。为此，你需要更改 DI 中的路由器信息：

```php
<?php

use Phalcon\Mvc\Router\Annotations;

$di['router'] = function() {
    $router = new Annotations(false);
    $router->addResource('Articles', '/api/v1/articles');

    return $router;
};
```

然后，你必须修改`ArticlesController`使其看起来像这样：

```php
<?php
namespace App\Api\Controllers;

/**
 * @RoutePrefix("/api/v1/articles")
 */
class ArticlesController extends BaseController {
  /**
  * @Get("/")
  */
  public function listAction() {

  }
}
```

你可以在[`docs.phalconphp.com/en/latest/reference/routing.html#annotations-router`](http://docs.phalconphp.com/en/latest/reference/routing.html#annotations-router)了解更多关于注解路由器的信息。如果你需要/想要，你也可以开发自己的路由器，实现`Phalcon\Mvc\RouterInterface`。

# 摘要

在本章中，我们看到了如何使用 Phalcon 上传文件，也看到了如何使用注解路由器。

Phalcon 是一个完全解耦的框架。没有真正的“最佳实践”，因此作为开发者，您可以构建自己的管道。我还建议您查看 Phalcon 的 Vegas CMF 项目[`github.com/vegas-cmf`](https://github.com/vegas-cmf)，尤其是如果您打算与大型团队合作。

感谢您阅读这本书，我真心希望它对您有所帮助。现在您可以开始开发自己的应用程序了。
