# 第九章. 扩展核心

在本章中，你将学习：

+   使用 CodeIgniter Sparks

+   使用 DOMPDF Spark 创建 PDF

+   在 CodeIgniter 中创建钩子

+   从数据库中清除无效会话

+   扩展你的控制器

+   使用 FTP 上传文件

+   创建自己的配置文件并使用设置

+   创建库并为它们提供对 CodeIgniter 资源的访问

+   使用语言类 – 在路上切换语言

# 简介

CodeIgniter 几乎可以做到你需要的所有事情，直接从盒子里出来；但总会有时候你必须扩展或更改标准设置——无论是创建钩子以便你不必修改核心（你真的不想修改核心），还是安装 Sparks 以添加额外的功能和新特性——有许多方法可以扩展和构建 CodeIgniter 以获得更多功能。

# 使用 CodeIgniter Sparks

不久以前，如果你想为 CodeIgniter 安装一个扩展或一些外部软件，你必须四处寻找。你需要在互联网上搜索你想要的东西，如果你很幸运，你可能会找到一个博客或某个人的个人网站，也许甚至是一个 GitHub 账户或 Google 代码仓库，你可以从中下载并安装一个包。有时它有效，有时则无效，而且你下载的几乎总是需要某种程度的编辑或重写。

快进到现在！幸运的是，我们有 Sparks。将 Sparks 视为可以与 CodeIgniter 一起使用的扩展。它们被保存在一个地方（这样你就不必在互联网上四处寻找），在 [`www.getsparks.org`](http://www.getsparks.org)。

我们在第一章，*CodeIgniter 基础*中提到了 Sparks，但是，让我们更深入地探讨，并指导你安装和使用一些我在一段时间内发现很有用的 Sparks。

因此，如果你之前还没有安装它，现在就在你的 CodeIgniter 实例中安装 Sparks。你可以前往第一章，*CodeIgniter 基础*，获取如何操作的说明，或者查看 GetSparks 网站上的说明。

## 准备工作

首先，你需要导航到你的 CodeIgniter 目录的顶级（或根）目录。

## 如何做到这一点...

如果你使用的是 MAC 或 Linux，按照以下方式下载 CodeIgniter Sparks：

1.  在命令行中，导航到你的 CodeIgniter 应用程序的根目录，然后输入：

    ```php
    php -r "$(curl -fsSL http://getsparks.org/go-sparks)"
    ```

    这将下载并安装 CodeIgniter Sparks 到你的特定 CodeIgniter 实例。

如果你使用的是 Windows，那么你需要手动下载并解压 Sparks。请按照以下说明操作，或者查看 GetSparks 网站上的说明以获取最新版本：

1.  在你的 CodeIgniter 目录的顶级（根）目录下创建一个名为 `tools` 的文件夹。

1.  访问以下 URL：[`getsparks.org/install`](http://getsparks.org/install)。

1.  前往**常规安装**部分，下载 Sparks 包。

1.  将下载的内容解压到你在第一步中创建的 `tools` 文件夹中。

1.  从以下链接下载 `Loader` 类扩展：[`getsparks.org/static/install/MY_Loader.php.txt`](http://getsparks.org/static/install/MY_Loader.php.txt)。

1.  将 `MY_Loader.php.txt` 文件重命名为 `MY_Loader.php`，并将其移动到你的 CodeIgniter 应用程序的 `application/core/MY_Loader.php` 目录中。

## 它是如何工作的...

Sparks 已下载到你的 CodeIgniter 实例的 `/path/to/codeigniter/tools` 文件夹中，准备使用。你可以通过以下命令安装任何你想要的 Spark：

```php
php tools/spark install [version] spark-name
```

这里，`[version]` 是 Spark 的特定版本。它是可选的，如果你省略它，Sparks 将自动选择最新版本。`spark-name` 是你想要下载的 Spark 的名称。

# 使用 DOMPDF Spark 创建 PDF

DOMPDF Spark 是一个非常好的工具；它设置简单，可以处理大多数事情。它通过获取 HTML 模板文件的输出（你之前创建的）来工作——你可以像使用普通视图文件一样向 HTML 传递变量——DOMPDF 将从格式化的 HTML 代码创建一个 PDF 文件。

## 准备工作

由于这个菜谱需要 DOMPDF Spark，我们在做其他任何事情之前需要将其安装到我们的 CodeIgniter 实例中。要安装它，请执行以下步骤：

1.  从 [getsparks.org](http://getsparks.org) 获取 DOMPDF Spark 并安装它。打开一个终端窗口，导航到你的 CodeIgniter 应用程序目录，并输入以下代码：

    ```php
    php tools/spark install dompdf
    ```

    这将下载 DOMPDF Spark。DOMPDF Spark 可能已经被下载，但你需要做一些安装才能使其运行。

    在 `/path/to/codeigniter/sparks/dompdf/[version-number]/helpers` 文件夹中（其中 `[version-number]` 是 Spark 的版本），有一个文件夹和一个文件。具体如下（加粗）：

    +   `/path/to/codeigniter/sparks/dompdf/0.5.3/helpers/dompdf/`

    +   `/path/to/codeigniter/sparks/dompdf/0.5.3/helpers/dompdf_helper.php`

1.  将此文件夹和文件复制到你的 CodeIgniter 应用程序的 `helpers` 文件夹中（`/path/to/codeigniter/application/helpers`）。

1.  我们还需要一个数据库表，以便我们的模型能够工作，所以请继续，并在你的数据库中输入以下代码：

    ```php
    CREATE TABLE `users` (
      `user_id` int(11) NOT NULL AUTO_INCREMENT,
      `user_firstname` varchar(255) NOT NULL,
      `user_lastname` varchar(255) NOT NULL,
      `user_email` varchar(255) NOT NULL,
      PRIMARY KEY (`user_id`)
    ) ENGINE=InnoDB  DEFAULT CHARSET=latin1 AUTO_INCREMENT=3 ;

    INSERT INTO `users` (`user_id`, `user_firstname`, `user_lastname`, `user_email`) VALUES
    (1, 'Rob', 'Foster', 'rf@domain.com'),
    (2, 'Lucy', 'Welsh', 'lw@domain.com');
    ```

## 如何操作...

我们将创建以下三个文件：

+   `/path/to/codeigniter/application/controllers/makepdf.php`：此控制器将加载 DOMPDF 扩展并输出我们的 PDF。

+   `/path/to/codeigniter/application/models/makepdf_model.php`：我们将使用此模型从数据库获取传递给视图文件 `view_all_users` 所必需的信息。

+   `/path/to/codeigniter/application/views/makepdf/view_all_users.php`：此文件包含我们希望 PDF 看起来的 HTML 和标记。它还包含一些 PHP 代码，这些代码会输出从模型收集的一些数据。

1.  首先添加以下代码到 `makepdf` 控制器中：

    ```php
    <?php if (! defined('BASEPATH')) exit('No direct script access allowed');

    class Makepdf extends CI_Controller {
        function __construct() {
            parent::__construct();
            $this->load->helper('file');
            $this->load->helper('dompdf');
            $this->load->model('Makepdf_model');
        }

        function index() {
            $data['query'] = $this->Makepdf_model->get_all_users();
            $filename = 'List-of-users';
            $html = $this->load->view('makepdf/view_all_users', $data, true);
            pdf_create($html, $filename);
        }
    }
    ```

1.  然后将以下代码添加到 `makepdf_model` 模型中：

    ```php
    <?php if (! defined('BASEPATH')) exit('No direct script access allowed');

    class Makepdf_model extends CI_Model {

        function __construct() {
            parent::__construct();
        }

        function get_all_users() {
            $query = $this->db->get('users');
            return $query;
        }
    }
    ```

1.  最后，将以下代码添加到`makepdf/view_all_users.php`视图中：

    ```php
    <table>
    <tr>
        <td>First Name</td>
        <td>Last Name</td>
        <td>Email</td>
    </tr>
        <?php foreach($query->result() as $row) : ?>
            <tr>
                <td><?php echo $row->user_firstname ; ?></td>
                <td><?php echo $row->user_lastname ; ?></td>
                <td><?php echo $row->user_email ; ?></td>
            </tr>
        <?php endforeach ; ?>
    </table>
    ```

## 它是如何工作的……

在我们的`makepdf`控制器构造函数中，我们加载了`dompdf`辅助工具。我们将`dompdf_helper.php`文件和`dompdf`文件夹放置在我们之前在“准备就绪”部分安装的 CodeIgniter 应用程序的`helpers`文件夹中。

`private function index()`然后调用`Makepdf_model`函数，`get_all_users()`。这个模型函数在用户表上运行一个简单的选择操作，并将其存储在`$data['query']`数组中，我们稍后会回到这一点。`$filename`变量被分配字符串`List-of-users`，但如果你愿意，你可以将其更改为任何你想要的，你甚至可以添加今天的日期，代码片段如下：

```php
$filename = 'List-of-users-for'.date("d-m-Y", time())
```

无论如何，回到故事中。我们然后使用标准的 CodeIgniter 方法一个视图文件，并将`$data`数组传递给它：`$html = $this->load->view('makepdf/view_all_users', $data, true)`。记住这个`$data`数组包含我们的用户数据库查询。我们不是让 CodeIgniter 像通常那样渲染视图，而是将现在处理过的 HTML 捕获到`$html`变量中，该变量被传递到`dompdf`函数`pdf_createalong`，以及`$filename`变量。

如果你将控制器加载到浏览器中，你应该会自动提示下载 PDF。

你可以看到，这可以很容易地适应输出各种广泛的 HTML 设计，从发票到采购订单，再到地址联系人。只需修改模型查询，从数据库中提取你需要知道的信息，并更改 HTML 以显示这些信息，然后就可以完成了。

# 在 CodeIgniter 中创建钩子

你就在那里，快乐地使用某个框架或 CMS 来运行一个网站，现在有人要求你让它做它最初没有设计的事情。你很不情愿地同意了，并开始研究如何实现所要求的功能。你决定修改你所遇到的任何平台的系统核心文件，因为它既快又简单，而且你的老板使用了诸如“快速胜利”和“成本效益”之类的术语。

无论如何，请求的功能是有效的，每个人都感到高兴——除了那个可怜的人（可能就是你）几个月后回来升级平台到最新版本时……*砰*！你对该系统核心所做的修改已经丢失，因为升级时被新文件覆盖了。现在，你被要求实现的那件疯狂的事情不再工作，你只能试图回忆你是如何让它工作的。然后，在任何人注意到之前，它就消失了，你只能想着。“哦，上帝，我为什么就不能回家！”

好吧，振作起来！你正在使用 CodeIgniter，这种情况永远不会发生在你身上（当然，前提是你使用钩子）。如果你愿意，你仍然可以修改核心——去做吧，随意修改，看看我是否在乎——但当你需要升级时，你会后悔的。我只是在考虑你的压力水平。

Hooks 允许你在 CodeIgniter 执行的特定点覆盖其行为，而无需修改 CodeIgniter 核心文件。太棒了！这意味着使用 CodeIgniter，你可以实现任何数量的荒谬的管理变更请求，而且升级当天一点也不会受到影响。Hooks 按特定顺序工作。这意味着什么？也就是说，CodeIgniter 按特定顺序工作，也就是说，当 CodeIgniter 运行时，它会按顺序加载其特定的部分。这个顺序始终相同，并且你可以设置一个 Hook 在任何一步执行。

## 准备工作

首先，你需要告诉 CodeIgniter 它应该允许 Hooks 运行。打开 `/application/config/config.php` 文件，确保 `enable_hooks` 的设置是 `TRUE`，如下面的代码片段所示：

```php
$config['enable_hooks'] = TRUE;
```

你还必须决定你的 Hook 运行的最佳时间。在 CodeIgniter 的执行过程中有七个点可以设置你的 Hook 运行，如下所示：

+   `pre_system`：这是你可以设置 Hook 运行的最早位置。在这个阶段，只有基准测试和 Hooks 被引入

+   `pre_controller`：这在你调用任何控制器之前执行

+   `post_controller_constructor`：这使你的 Hooks 在控制器构造函数之后但调用任何控制器函数之前运行

+   `post_controller`：这使得你的 Hook 在控制器执行完毕后运行

+   `display_override`：这将覆盖 CodeIgniter 的 `display()` 函数--这是 CodeIgniter 尝试将视图文件或其他输出渲染到屏幕上的时候

+   `cache_override`：这覆盖了 `_display_cache()` 函数，如果你想要实现自定义缓存功能，可以使用这个

+   `post_system`：这是在正常操作完成后要调用的（即，在系统完成当前请求的执行后）

## 如何操作...

1.  为你的 Hook 定义一个执行点。一旦你决定了运行 Hook 的点，你将需要告诉 CodeIgniter 何时运行钩子。你通过在 `config/hooks.php` 文件中定义 `$hook` 数组中的正确信息来完成此操作。

    `$hook` 数组的键指定了执行点--即你希望钩子运行的时候（参见本食谱中“准备工作”部分提到的列表）。在下面的示例中，数组键是 `post_controller`，这意味着钩子在控制器执行完毕后执行。现在的 `$hook` 数组看起来如下。

    ```php
    $hook['post_controller'] = array(
        'class'    => 'Class_name',
        'function' => 'function_name',
        'filename' => 'file_name_of_hook.php',
        'filepath' => 'path_to_hook',
        'params'   => array(param1, param2, param3 ... etc)
        );
    ```

    那么，前面的所有代码意味着什么呢？让我们看一下下面的表格，以更好地理解前面的代码：

    | 数组元素 | 描述 |
    | --- | --- |
    | `class` | 它是 `filename` 元素中定义的文件中的类的名称。 |
    | `function` | 它是类中函数的名称。 |
    | `filename` | 它是包含类的文件的名称。 |
    | `filepath` | 这是数组元素`filename`中定义的文件的路径，通常是`hooks`文件夹，但如果你愿意，可以添加子文件夹或将它移动到不同的位置。 |
    | `params` | 这是你要传递给`Hook`类`function`元素的任何参数。用逗号分隔每个参数。 |

1.  然后，你需要创建你的钩子类，如下面的代码片段所示：

    ```php
    <?php  if ( ! defined('BASEPATH')) exit('No direct script access allowed');

    class Class_name {
      function function_name () {
      }
    }
    ```

    你将你的钩子代码放在这个类中，显然需要将`Class_name`改为对你更有用的名称，而函数`function_name()`显然是一个示例——这个名称也需要更改。

如果你需要在钩子中访问 CodeIgniter 资源，你可以通过使用以下代码中的 CodeIgniter get_instance()函数来访问主 CodeIgniter 对象：

```php
function function_name () {
  $CI = & get_instance();
  $CI->load->thing_to_load();
}
```

在这里，`thing_to_load`是你的模型、库等的名称。在那里，你就可以看到钩子实际上很简单，确定一个执行点，创建一个包含你的钩子代码的类，然后就可以开始了！

# 从数据库中清除无效会话

你可能发现 CodeIgniter 有时无法成功从数据库中的会话表中删除会话。未使用的会话会清除其`user_data`字段，但行仍然保留在会话表中。

我经常需要编写特定的数据库查询来清除会话。我通常将这些查询放在`My_Controller`类中（参考本章中的*扩展你的控制器*配方）；然而，我最近开始使用钩子来完成这个任务。这样它总是在后台运行，我无需再考虑它。

## 准备工作

我们将要修改以下文件：

`/path/to/codeigniter/application/config/config.php`

打开`/application/config/config.php`文件，确保`enable_hooks`的设置是`TRUE`，如下面的代码片段所示：

```php
$config['enable_hooks'] = TRUE;
```

## 如何操作...

我们将创建并修改以下文件：

`/path/to/codeigniter/application/hooks/clear_sessions.php`

1.  创建前面的文件，并向其中添加以下代码：

    ```php
    <?php  if ( ! defined('BASEPATH')) exit('No direct script access allowed');

    class Clear_sessions {
      function clear_now() {
        $CI =& get_instance();
        $query = $CI->db->query("DELETE FROM `ci_sessions` WHERE `user_data` = '' ");
      }
    }
    ```

    ### 小贴士

    我们使用的是默认的 CodeIgniter 会话表名称，即`ci_sessions`。在你的应用程序中，你需要将这个值替换为你会话表的名称（假设你已经将其从`ci_sessions`更改了）。

1.  打开`/path/to/codeigniter/application/config/hooks.php`文件，并向其中添加以下代码：

    ```php
    $hook['post_controller'] = array(
        'class'    => 'Clear_sessions',
        'function' => 'clear_now',
        'filename' => 'clear_sessions.php',
        'filepath' => 'hooks'
        );
    ```

## 它是如何工作的...

我们在`clear_sessions.php`文件中定义了一个类（`Clear_sessions`），我们将把所有需要用于钩子执行任务的逻辑放入这个类中。在这个类中，我们有一个`clear_now()`函数。CodeIgniter 知道它需要在`clear_sessions.php`文件中的`Clear_sessions`类中运行`clear_now()`函数，因为我们已经在`hooks.php`配置文件中定义了这一点，如下面的代码片段所示：

```php
$hook['post_controller'] = array(
    'class'    => 'Clear_sessions',
    'function' => 'clear_now',
    'filename' => 'clear_sessions.php',
    'filepath' => 'hooks'
    );
```

我们通过前一个数组中的`filepath`元素告诉 CodeIgniter`clear_sessions.php`文件的路径。CodeIgniter 将在执行`post_controller`阶段运行此钩子，也就是说，在运行控制器之后。因此，这就是 CodeIgniter 知道何时执行什么钩子；但钩子的功能是什么？它做了什么？

让我们看看以下代码行：

```php
$CI =& get_instance();
```

这使我们能够访问主 CodeIgniter 对象，该对象定义为`$CI`（代表 CodeIgniter）。使用此对象，我们可以访问数据库功能和设置。我们将使用此`$CI`对象来运行数据库查询，如下所示代码片段所示：

```php
$query = $CI->db->query("DELETE FROM `ci_sessions` WHERE `user_data` = '' ");
```

上一行代码会删除表中（在此处命名为默认的 CodeIgniter 会话表 ci_sessions）任何`user_data`为空的会话（*不是*非空，而是空）。

所有 CodeIgniter 未正确清理的已消耗会话现在应从会话表中消失，而留下活跃的会话。

# 扩展您的控制器

继承是一件美妙的事情；它允许您定义概念层或应用程序中的级别，也就是说，允许子类共享父类的特性和属性，这使得应用程序能够比标准过程式编程更准确、更直观地模拟现实世界的例子。

在 CodeIgniter 中，您应用程序的默认设计结构是您创建的控制器扩展主 CodeIgniter 控制器。例如，在以下代码中，我们有一个`Signin`控制器，它扩展了主 CodeIgniter 控制器`CI_Controller`：

```php
class Signin extends CI_Controller {
}
```

您的`Signin`控制器将继承`CI_Controller`的属性。然而，CodeIgniter 还允许您在`CI_Controller`和您创建的控制器之间插入一个阶段，您可以插入一个中间层。这个中间步骤被命名为`My_Controller`。新的 MY_Controller 将扩展 CI_Controller（在扩展过程中继承一切）以及您创建的任何普通应用程序特定控制器（用户、登录、发票等），它们都将继承自新的 MY_Controller。它定义如下代码片段所示：

```php
class MY_Controller extends CI_Controller {
}
```

现在，您的`Signin`类必须修改为从您的`MY_Controller`继承，而不是从 CodeIgniter 的`CI_Controller`继承，如下所示：

```php
class Signin extends MY_Controller {
}
```

## 如何操作...

我们将创建以下两个文件：

+   `/path/to/codeigniter/application/core/MY_Controller.php`

+   `/path/to/codeigniter/application/controllers/days.php`

1.  创建`/path/to/codeigniter/application/core/MY_Controller.php`文件，并将以下代码添加到其中：

    ```php
    <?php if (! defined('BASEPATH')) exit('No direct script access allowed');
    class MY_Controller extends CI_Controller {
        function __construct() {
            parent::__construct();
            $this->load->helper('array');
        }
    }
    ```

1.  接下来，创建`/path/to/codeigniter/application/controllers/days.php`文件，并将以下代码添加到其中：

    ```php
    <?php if (! defined('BASEPATH')) exit('No direct script access allowed');

    class Days extends MY_Controller {
        function __construct() {
            parent::__construct();
        }

        function index() {
            $days = array("Monday","Tuesday","Wednesday","Thursday","Friday","Saturday","Sunday");
            echo random_element($days);
        }
    }
    ```

## 它是如何工作的...

这主要是继承。你的子类正在从父类继承属性。然而，具体来说，我们在这里展示了如何在 `MY_Controller 类` 中加载数组助手，并让 `Days` 控制器继承助手资源以执行数组助手函数 `random_element()`。

这有很多用途，例如，你可以检查用户是否在 `MY_Controller 类` 中登录，这意味着你只需要写一次，而不是在每个控制器中。或者你可以更进一步，有一个经过身份验证的控制器和一个未经身份验证的控制器扩展 `MY_Controller`。`MY_Controller` 文件将调用任何文件并执行对经过身份验证和未经身份验证用户都通用的任何任务，而经过身份验证的控制器将执行登录检查等。

# 使用 FTP 上传文件

不时地，你会被要求生成一个文件——可能是从数据库导出，或者是一些系统日志。大多数时候，你会被要求将文件通过电子邮件发送到某个位置，但有时，你会被要求将其上传到客户端可以访问的 FTP 文件夹。我们将从一个之前的章节的食谱中提取信息（第六章中的“从数据库结果生成 CSV”，*与数据库一起工作*），并对其进行调整，以便将其上传到 FTP 位置而不是流式传输输出。

## 准备工作

1.  我们将从一个数据库中提取一些值。为了做到这一点，我们需要一个表来提取数据。以下是该表的架构。将以下内容复制到您的数据库中：

    ```php
    CREATE TABLE `users` (
      `user_id` int(11) NOT NULL AUTO_INCREMENT,
      `user_firstname` varchar(255) NOT NULL,
      `user_lastname` varchar(255) NOT NULL,
      `user_email` varchar(255) NOT NULL,
      PRIMARY KEY (`user_id`)
    ) ENGINE=InnoDB  DEFAULT CHARSET=latin1 AUTO_INCREMENT=3 ;# MySQL returned an empty result set (i.e. zero rows).

    INSERT INTO `users` (`user_id`, `user_firstname`, `user_lastname`, `user_email`) VALUES
    (1, 'Rob', 'Foster', 'rf@domain.com'),
    (2, 'Lucy', 'Welsh', 'lw@domain.com');# 2 rows affected.
    ```

1.  前往第六章, *与数据库一起工作*，并复制出 *从数据库结果生成 CSV* 食谱中的代码；然后返回这里获取进一步的指示。

## 如何做到...

我们将修改 *从数据库结果生成 CSV* 食谱中的一个文件，即 `/path/to/codeigniter/application/controllers/export.php`。这是第六章, *与数据库一起工作*中的导出控制器。我们将将其作为本食谱的基础。它将从数据库中获取一些数据，最终为我们生成 CSV。

1.  打开 `export.php` 进行编辑，并修改它以反映以下代码片段（更改已突出显示）：

    ```php
    <?php if (!defined('BASEPATH')) exit('No direct script access allowed');

    class Export extends CI_Controller {
      function __construct() {
        parent::__construct();
        $this->load->helper('url');
        $this->load->helper('file');
        $this->load->library('ftp');
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
        $filename = 'myfile.csv';
        $path = './'.$filename;	

        if (! write_file($path, $data)) {
          echo 'Cannot write file - permissions maybe?';
          exit;
        }

        $config['hostname'] = 'your-hostname';
        $config['username'] = 'your-username';
        $config['password'] = 'your-password';
        $config['debug'] = TRUE;

        $this->ftp->connect($config);
        $this->ftp->upload($path, '/dir_on_ftp/'.$filename, 'ascii', 0755);
        $this->ftp->close();
      }
    }
    ```

    这里有一些变量你需要更改以反映你的 FTP 环境设置，例如：

    ```php
    $config['hostname'] = 'ftp.example.com';
    $config['username'] = 'your-username';
    $config['password'] = 'your-password';
    ```

    ...并且字符串 `dir_on_ftp` 在：

    ```php
    $this->ftp->upload($path, '/dir_on_ftp/'.$filename, 'ascii', 0755);
    ```

## 它是如何工作的...

正如你所期望的新食谱一样，我们在第六章, *与数据库一起工作*中使用的正常导出控制器中做了一些更改。这些更改是必要的，以支持 FTP 助手在将文件上传到 FTP 服务器的工作中的功能。让我们看看发生了什么。

首先，我们在导出控制器构造函数中加载所需的辅助工具--URL、文件和 FTP--以确保这个导出控制器拥有进行文件传输所需的正确支持。

```php
function __construct() {
    parent::__construct();
    $this->load->helper('url');
    $this->load->helper('file');
    $this->load->library('ftp');    
    $this->load->dbutil();    
  }
```

然后一切都将遵循先前控制器（来自第六章，*与数据库一起工作*）的功能，即从数据库表获取结果集，并使用 CodeIgniter 的`dbutil`函数`csv_from_result()`为我们生成文件。它通过`write_file()`函数使用在`$path`中定义的位置放置在服务器上。

然后 FTP 功能开始启动。我们为要将文件写入的 FTP 服务器定义登录设置--你可以，也可能应该将这些放在你自己的配置文件中（本章也有解释，见*制作自己的配置文件并使用设置*配方），但为了现在更容易解释，我们将在这里定义它们。设置相当明显，直到你看到`$config['debug']`数组设置。`debug`允许错误报告显示给你，即开发者。显然，在实时生产环境中，你肯定希望将其设置为`FALSE`以防止任何敏感和重要的信息被显示。

无论如何，我们使用在`$config`数组中定义的登录设置，尝试连接到 FTP 服务器并尝试上传文件，如下面的代码片段所示：

```php
$this->ftp->connect($config);
$this->ftp->upload($path, '/dir_on_ftp/'.$filename, 'ascii', 0755);
```

如果一切顺利，文件应该已经上传到你的服务器。使用 FTP 客户端登录并查看它是否在那里--如果不在，请检查你是否使用了正确的 FTP 设置，以及你在 FTP 服务器上写入的路径是否可写并且实际上存在，也就是说，这里用粗体定义的路径确实存在：

```php
$this->ftp->upload($path, '/dir_on_ftp/'.$filename, 'ascii', 0755);
```

一个有趣的观点是前面`upload()`函数末尾的两个函数参数：`ascii`和`0755`。这些表示我们将文件传输编码为`ascii`（即纯文本）并设置其文件权限为`0755`。如果你愿意，这些也可以在配置数组中定义。

# 创建库并让它们访问 CodeIgniter 资源

CodeIgniter 允许你在不需要或不需要在控制器或模型中放置代码的情况下创建自己的库和辅助工具。为什么要把代码放在库中而不是辅助工具中呢？嗯，有些人对这一点相当激动，我相信如果你足够深入地思考这个问题，你就能想出一套严格的规则来定义何时一段代码是辅助工具或库。但是生活太短暂了。只要代码有良好的文档，并且是可维护的、稳定的和安全的，你可以随心所欲。然而，作为一个一般规则：

*图书馆是用于需要访问其他资源的代码，例如需要访问数据库或外部系统（可能通过 cURL），而辅助器是较小的一段代码，用于执行特定任务（例如检查字符串是否为有效的电子邮件或*URL，例如）。*

我相信有更好的定义，但这个对我来说是有效的；而且我相信你可能会想要一个辅助器能够访问数据库或其他资源；而且我相信在有些时候，库可能不需要访问这些资源。我的观点是，确保代码有文档和可维护性，不要陷入概念设计细节。

话虽如此，让我们看看如何创建一个库并给它访问 CodeIgniter 资源（因为默认情况下它不会）。

## 准备工作

我们将通过库访问数据库；然而，为了做到这一点，我们需要一个可以访问的数据库（当然），所以将以下代码复制到你的数据库中：

```php
CREATE TABLE IF NOT EXISTS `person` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `first_name` varchar(50) NOT NULL,
  `last_name` varchar(50) NOT NULL,
  `email` varchar(255) NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB  DEFAULT CHARSET=latin1 AUTO_INCREMENT=5 ;

INSERT INTO `person` (`id`, `first_name`, `last_name`, `email`) VALUES
(1, 'Rob', 'Foster', 'rfoster@dudlydog.com'),
(2, 'Lucy', 'Welsh', 'lwelsh@cocopopet.com'),
(3, 'Chloe', 'Graves', 'cgraves@mia-cat.com'),
(4, 'Claire', 'Strickland', 'cstrickland@an-other-domain.com');
```

## 如何做...

我们将创建以下三个文件：

+   `/path/to/codeigniter/application/controllers/call_lib.php`

+   `/path/to/codeigniter/application/libraries/lib_to_call.php`

+   `/path/to/codeigniter/application/models/lib_model.php`

1.  创建`call_lib.php`控制器，并将以下代码添加到其中：

    ```php
    <?php if (! defined('BASEPATH')) exit('No direct script access allowed');

    class Call_lib extends CI_Controller {
      function __construct() {
        parent::__construct();
        $this->load->library('lib_to_call');
        $this->load->database();
      }

      public function index() {
        $result = $this->lib_to_call->get_users();

        echo '<pre>';
          var_dump($result->result());
        echo '</pre>';
      }
    }
    ```

1.  创建`lib_to_call.php`库，并将以下代码添加到其中：

    ```php
    <?php if (! defined('BASEPATH')) exit('No direct script access allowed'); 

    class Lib_to_call {
        public function get_users() {
     $CI =& get_instance();
     $CI->load->model('Lib_model'); 
     return $CI->Lib_model->get();
        }
    }
    ```

1.  然后创建`lib_model.php`模型，并将以下代码添加到其中：

    ```php
    <?php if ( ! defined('BASEPATH')) exit('No direct script access allowed');

    class Lib_model extends CI_Model {
        function __construct() {
            parent::__construct();
        }

        function get() {
            $query = $this->db->get('person');
            return $query;
        }
    }
    ```

## 它是如何工作的...

在查看如何给你的库访问 CodeIgniter 资源时，我们将运行前面的配方。它是连接到一个模型；然而，它可以连接到任何类型的 CodeIgniter 资源，如钩子或辅助器等。

首先，`call_lib.php`控制器在其构造函数中加载了`lib_to_call`库，如下所示：

```php
    function __construct() {
        parent::__construct();
        $this->load->library('lib_to_call');
    }
```

这使得库在整个控制器中可用。

然后我们调用`public function index()`，它调用库函数`get_users()`，并将返回的结果存储在`$result`变量中。让我们看看库函数`get_users()`正在做什么，这是允许库访问 CodeIgniter 资源的地方。

看看`lib_to_call`库中以下突出显示的行：

```php
class Lib_to_call {
    public function get_users() {
 $CI =& get_instance();
 $CI->load->model('Lib_model');
 return $CI->Lib_model->get();
    }
}
```

我们使用 PHP 函数`get_instance()`通过引用获取一个副本（这意味着我们使用的是原始对象而不是它的副本）或 CodeIgniter 对象。此对象可以访问整个 CodeIgniter 系统和其资源。我们将此对象存储在名为`$CI`（代表 CodeIgniter）的局部变量中。现在我们可以调用任何我们喜欢的 CodeIgniter 资源，就像从控制器中调用一样，只不过我们使用的是`$CI`（而不是在控制器中使用`$this`）。因此，要在一个控制器中调用模型或辅助器，我们将执行以下操作：

```php
$this->load->helper('helper_name');
$this->load->model('model_name');
```

但要在库中调用辅助器和模型，我们现在这样做：

```php
$CI->load->helper('helper_name');
$CI->load->model('model_name');
```

我们现在加载的模型将使用 Active Record 查询数据库，将表中的所有行返回到库中，然后库再将它返回到控制器进行处理。在这种情况下（为了证明它工作），我们使用 `var_dump()` 输出结果。如果一切顺利，这个菜谱应该会输出如下：

```php
array(4) {
  [0]=>
  object(stdClass)#19 (4) {
    ["id"]=>
    string(1) "1"
    ["first_name"]=>
    string(3) "Rob"
    ["last_name"]=>
    string(6) "Foster"
    ["email"]=>
    string(20) "rfoster@dudlydog.com"
  }
  [1]=>
  object(stdClass)#20 (4) {
    ["id"]=>
    string(1) "2"
    ["first_name"]=>
    string(4) "Lucy"
    ["last_name"]=>
    string(5) "Welsh"
    ["email"]=>
    string(20) "lwelsh@cocopopet.com"
  }
  [2]=>
  object(stdClass)#21 (4) {
    ["id"]=>
    string(1) "3"
    ["first_name"]=>
    string(5) "Chloe"
    ["last_name"]=>
    string(6) "Graves"
    ["email"]=>
    string(19) "cgraves@mia-cat.com"
  }
  [3]=>
  object(stdClass)#22 (4) {
    ["id"]=>
    string(1) "4"
    ["first_name"]=>
    string(6) "Claire"
    ["last_name"]=>
    string(10) "Strickland"
    ["email"]=>
    string(31) "cstrickland@an-other-domain.com"
  }
}
```

前面的菜谱通过访问数据库来展示如何在库中从内部访问 CodeIgniter 超级对象。然而，辅助工具、钩子和其他元素也可以访问。通过使用 `$CI =& get_instance()`，我们可以访问主要的 CodeIgniter 对象。通过使用 `$CI` 而不是 `$this`，这将使我们能够访问 CodeIgniter 的所有资源，如下面的代码片段所示：

```php
$CI =& get_instance();
$CI->load->model('Lib_model');
return $CI->Lib_model->get();
```

# 创建自己的配置文件并使用设置

在同一文件中拥有配置设置是一个很好的主意——好处显而易见——因此，而不是将设置隐藏在控制器、模块、辅助工具、库中，或者（天哪）在视图中，你可以将它们放在一个位置，并从那里引用。CodeIgniter 在 `config` 文件夹中自带了自己的配置文件；然而，你可以在 `config` 文件夹中添加自己的文件，并在你的代码中引用它们。这非常方便且易于操作；让我们看看。

## 如何做到这一点...

我们将创建以下两个文件：

+   `/path/to/codeigniter/application/controllers/config_settings.php`

+   `/path/to/codeigniter/application/config/my_config_file.php`

1.  创建 `config_settings.php` 控制器，打开它进行编辑，并添加以下代码：

    ```php
    <?php if (!defined('BASEPATH')) exit('No direct script access allowed');

    class Config_settings extends CI_Controller {
      function __construct() {
        parent::__construct();
     $this->config->load('my_config_file');
      }

      public function index() {
        echo $this->config->item('first_config_item');

        for($i = 0; $i < $this->config->item('stop_at'); $i++) {
          echo $i . '<br />';
        }
      }
    }
    ```

1.  创建配置文件，`my_config_file.php`，打开它进行编辑，并添加以下代码：

    ```php
    <?php if (!defined('BASEPATH')) exit('No direct script access allowed');

    $config['first_config_item'] = "This is my config setting, there are many like it but this one is mine<br />";
    $config['stop_at'] = 10;
    ```

## 它是如何工作的...

说实话，有那么多情况将信息放在配置文件中是有用的，以至于为你们编写一个特定的菜谱是毫无意义的，因为你们需要这个菜谱的可能性极小。所以，这只是一个概念证明；它作为基本两个文件的一个指南存在：一个配置文件和另一个文件（一个控制器）来从中获取信息。

看看控制器中的构造函数，`Config_settings`。这就是我们定义配置文件名称的地方。你可以随意命名（只要名称没有被另一个配置文件占用）。在这里，我将其命名为 `my_config_file`；虽然有点像 1995 年，但对于解释来说足够好了，我相信你们已经明白了这个意思。

接下来发生的事情是执行 `public function index()`，它做两件事：打印出一段文本（文本设置在我们的配置文件中）并遍历一个 `for()` 循环，直到达到我们在 `my_config_file` 中指定的值。

这两种方法展示了如何处理配置值：要么在屏幕上输出，要么在某种结构中使用该值。

# 使用语言类——随时切换语言

一旦您开发了一个支持多种语言的网站，您显然希望允许人们在这些语言之间切换。例如，从英语切换到法语，或从法语切换到德语，或为任何您正在开发的地区或语言。这可以通过几种方式处理，但在这个例子中，我们将使用 CodeIgniter 的 `Session` 类在不同的语言之间切换。

## 准备工作

我们将使用 `Session` 类来存储用户的语言偏好，这意味着我们需要使用 CodeIgniter 会话。这需要一些配置。

我们将编辑以下文件：

+   `path/to/codeigniter/application/config/config.php`

+   `path/to/codeigniter/application/config/database.php`

以下是一些配置设置：

1.  在 `path/to/codeigniter/application/config/config.php` 文件中找到以下配置值，并修改它们以反映以下值：

    | 配置项 | 数据类型 | 更改为值 | 描述 |
    | --- | --- | --- | --- |
    | `$config['sess_cookie_name']` | `字符串` | `ci_session` | 这是写入用户计算机的 cookie 的名称。 |
    | `$config['sess_expiration']` | `整型` | `7200` | 这是指定会话在无用户活动后应保持活跃的秒数，在此之后变为无效。 |
    | `$config['sess_expire_on_close']` | `布尔型 (True/False)` | `TRUE` | 这指定了如果用户关闭浏览器，会话将变为无效。 |
    | `$config['sess_encrypt_cookie']` | `布尔型 (True/False)` | `TRUE` | 这指定了 cookie 是否应在用户计算机上加密。出于安全考虑，应将其设置为 `TRUE`。 |
    | `$config['sess_use_database']` | `布尔型 (True/False)` | `TRUE` | 这指定了是否将会话存储在数据库中。出于安全考虑，应将其设置为 `TRUE`。您还需要创建会话表，其代码可在下一页找到。 |
    | `$config['sess_table_name']` | `字符串` | `sessions` | 这指定了用于存储会话数据的数据库表名称。在这个菜谱中，我称会话表为简单会话；然而，您可以保留原始的 i_sessions--只需确保相应地修改 SQL 即可。 |
    | `$config['sess_match_ip']` | `布尔型 (True/False)` | `TRUE` | 这指定了 CodeIgniter 是否应监控请求的 IP 地址与 `session_id` 的对比。如果传入请求的 IP 与之前的值不匹配，则不允许会话。 |
    | `$config['sess_match_useragent']` | `布尔型 (True/False)` | `TRUE` | 这指定了 CodeIgniter 是否应监控请求的用户代理地址与 `session_id` 的对比。如果传入请求的用户代理与之前的值不匹配，则不允许会话。 |

1.  在 `path/to/codeigniter/application/config/database.php` 文件中找到以下配置值，并修改它们以反映正确的设置，以便您能够连接到您的数据库：

    | 配置项 | 数据类型 | 更改为值 | 描述 |
    | --- | --- | --- | --- |
    | `$db['default']['hostname']` | `String` | `localhost` | 您数据库的主机名。这通常是 `localhost` 或 IP 地址 |
    | `$db['default']['username']` | `String` |   | 您用于连接数据库的用户名 |
    | `$db['default']['password']` | `String` |   | 连接到您的数据库时使用的密码 |
    | `$db['default']['database']` | `String` |   | 您希望连接到的数据库名称 |

1.  我们将在会话中存储用户的语言偏好，会话存储在数据库表 `sessions` 中。以下为会话表的架构。使用您选择的方法（命令行、`phpMyAdmin` 等），将以下 MySQL 架构输入到您的数据库中：

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
    ```

## 如何做到这一点...

我们将创建以下语言文件：

`/path/to/codeigniter/application/language/french/fr_lang.php`

...并修改以下两个文件：

+   `/path/to/codeigniter/application/controllers/lang.php`

+   `/path/to/codeigniter/application/views/lang/lang.php`

1.  修改 `/path/to/codeigniter/application/controllers/lang.php` 以反映以下代码片段：

    ```php
    <?php if (!defined('BASEPATH')) exit('No direct script access allowed');
    class Lang extends CI_Controller {
        function __construct() {
            parent::__construct();
            $this->load->helper('form');
            $this->load->helper('url');
            $this->load->helper('language');
            $this->load->helper('security');

            // Check for empty language values in the session
            if ($this->session->userdata('filename') == '' || $this->session->userdata('language') == '') {
                $change_lang = array(
                   'language'   => 'english',
                   'filename'   => 'en',
                );

                $this->session->set_userdata($change_lang);
                $this->lang->load($this->session->userdata('filename'), $this->session->userdata('language'));
            } else { // Default language
                $this->lang->load($this->session->userdata('filename'), $this->session->userdata('language'));
            }
        }

        public function index() {
            redirect('lang/submit');
        }

        public function submit() {
            $this->load->library('form_validation');
            $this->form_validation->set_error_delimiters('', '<br />');

            // Set validation rules
            $this->form_validation->set_rules('email', $this->lang->line('form_email'), 'required|min_length[1]|max_length[50]|valid_email');

            // Begin validation
            if ($this->form_validation->run() == FALSE) {
                $this->load->view('lang/lang');
            }
        }

    public function change_language() {
            $lang = xss_clean($this->uri->segment(3));

            switch ($lang) {
                    case "en":
                            $language = 'english';
                            $filename = 'en';

                            break;
                    case "fr":
                            $language = 'french';
                            $filename = 'fr';
                            break;
                    default:
                        break;
            }

            $change_lang = array(
               'language'   => $language,
               'filename'   => $filename,
            );

            $this->session->set_userdata($change_lang);

            redirect('lang/index');
        }
    }
    ```

1.  修改 `/path/to/codeigniter/application/views/lang/lang.php` 以反映以下代码片段：

    ```php
    <html>
        <body>

            <?php echo anchor('lang/change_language/fr', 'French'); ?>
            &nbsp;
            <?php echo anchor('lang/change_language/en', 'English'); ?>

            <h2><?php echo $this->lang->line('form_title'); ?></h2>
            <?php echo form_open('lang/submit'); ?>
            <?php echo $this->lang->line('form_email'); ?>
            <?php echo form_input(array('name' => 'email', 'id' => 'email', 'value' => '', 'maxlength' => '100', 'size' => '50', 'style' => 'width:10%')); ?>

            <?php echo form_submit('', $this->lang->line('form_submit_button')); ?>
            <?php echo form_close(); ?>

        </body>
    </html>
    ```

1.  将以下代码复制到 `/path/to/codeigniter/application/system/language/french/fr_lang.php` 文件中：

    ```php
    <?php if (!defined('BASEPATH')) exit('No direct script access allowed');
    $lang['form_title'] = "Titre du formulaire en anglais";
    $lang['form_email'] = "Votre E-Mail: ";
    $lang['form_submit_button'] = "Suggérer";
    ```

1.  最后，将以下代码添加到 `/path/to/codeigniter/application/system/language/english/en_lang.php` 文件中：

    ```php
    <?php if (!defined('BASEPATH')) exit('No direct script access allowed');
    $lang['form_title'] = "Form title in English";
    $lang['form_email'] = "Email: ";
    $lang['form_submit_button'] = "Submit";
    ```

## 它是如何工作的...

这是对早期语言菜谱的扩展。我们唯一不同的地方是添加了对 `$this->lang->load('', '');` 中值的切换支持。我们使用 CodeIgniter 的会话功能来存储切换的值。因此，当我们通过 URL 中的值传递所需的语言时，我们将想要使用 CodeIgniter 的安全方法 `xss_clean()` 来减轻跨站脚本攻击的风险。

因此，在构造函数中，我们有以下代码：

```php
// Check for empty language values in the session
if ($this->session->userdata('filename') == '' || $this->session->userdata('language') == '') {
     $change_lang = array(
     'language'   => 'english',
     'filename'   => 'en',
     );

    $this->session->set_userdata($change_lang);
    $this->lang->load($this->session->userdata('filename'), $this->session->userdata('language'));
} else { // Default language
    $this->lang->load($this->session->userdata('filename'), $this->session->userdata('language')); 
}
```

这将检查是否有任何语言变量在会话中设置。如果没有，我们在 `$change_lang` 数组中定义语言并将其传递给 `$this->lang->load('', '');` 函数，如下所示：

```php
$this->lang->load($this->session->userdata('filename'), $this->session->userdata('language'));
```

从那里，`public function index()` 被加载并立即重定向到 `public function submit()`，显示 HTML 表单。

我们已修改了 HTML 表单，添加了以下两个锚点标签：

```php
<?php echo anchor('lang/change_language/fr', 'French'); ?>
&nbsp;
<?php echo anchor('lang/change_language/en', 'English'); ?>
```

如果用户在浏览器中点击这些链接（法语或英语）中的任何一个，将执行 `public function change_language()`。然后我们通过 `$lang = xss_clean($this->uri->segment(3));` 获取并清理 URL 的第三个参数，并通过 PHP Switch/Case 语句查找 `en` 或 `fr`，在过程中将 `$language` 和 `$filename` 变量分配为正确的详细信息（默认加载英语）。

接下来，我们将 `$language` 和 `$filename` 变量加载到 `$change_lang` 数组中，并使用 `$this->session->set_userdata($change_lang);` 写入会话。最后，我们通过重定向回 `public function index()` 来结束，这将带我们回到起点，并加载新的语言。
