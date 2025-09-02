# 第十二章：创建控制台应用程序以自动化周期性任务

在本章中，我们将学习如何编写控制台应用程序，并会发现 Web 应用程序和控制台应用程序之间的主要区别。

然后，我们将创建我们的第一个控制台控制器，使用一个实际示例来说明如何更新数据库表。

在最后几段中，我们将看到如何设置输出颜色和文本格式，以及如何实现一个完整的周期性任务，例如使用每日预订发送电子邮件。在本章中，我们将涵盖以下主题：

+   与控制台应用程序交互

+   创建控制台控制器

    +   示例 - 为已过期的预订设置警报标志

+   格式化控制台输出

+   实施和执行 cron 作业

    +   示例 - 发送包含当日新预订的电子邮件

# 与控制台应用程序交互

控制台是高级模板默认安装的第三个应用程序。

此应用程序配置为通过控制台访问启动命令，并且它具有与之前章节中看到的相同的应用程序结构。因此，在本节中，我们需要对主机进行控制台访问。

与到目前为止使用的 Web 和 API 应用程序相比，有一些区别。

控制器的`public`属性实际上在命令行上作为`option`可见。需要扩展控制器的`option()`方法以使这些属性可用。此外，根据特定操作，操作参数作为命令行参数传递。

最后，控制台控制器操作可以返回一个退出代码，这是一个数字，其中 0 表示一切正常，这是控制台应用程序开发的最佳实践。

下面是一个从 shell 开始使用控制台应用程序的典型用法：

```php
yii <route> [--option1=value1 --option2=value2 ... argument1 argument2 ...]

```

上述代码的元素解释如下：

+   `route`：这表示要调用的`controller/action`路径

+   `option`：这表示控制器针对该特定操作的`public`属性的可访问性；我们只能访问控制器`options()`方法返回的公共属性

+   `argument`：这表示要传递给控制器操作的参数

    ### 注意

    总有一个始终可用的选项，`appconfig`，用于指示必须使用哪个配置文件路径。如果没有设置，将采用默认配置文件。

Yii 提供了一套核心控制台应用程序，我们可以通过调用`help`控制器（作为一个 Web 应用程序，默认操作将是`index`）来访问它们，以便显示有关可用控制台控制器列表或单个控制器或操作控制器详细信息的所有内容。

让我们考虑一个示例；打开命令行（在这种情况下，Linux shell）并从项目根目录输入以下内容：

```php
$ ./yii help

```

这将显示类似于以下内容的输出（部分显示）：

```php
This is Yii version 2.0.4.

The following commands are available:

- asset                         Allows you to combine and compress your JavaScript and CSS files.
 asset/compress (default)    Combines and compresses the asset files according to the given configuration.
 asset/template              Creates template of configuration file for [[actionCompress]].

- cache                         Allows you to flush cache.
 cache/flush                 Flushes given cache components.
 cache/flush-all             Flushes all caches registered in the system.
 cache/flush-schema          Clears DB schema cache for a given connection component.
 cache/index (default)       Lists the caches that can be flushed.
…
…

```

在这里，第一级分组表示控制器名称（右侧有相对描述），第二级包括相关控制器的操作。当我们传递控制器名称时，我们将需要更详细的信息来帮助它：

```php
$ ./yii help message

```

要显示控制器描述和操作列表，我们还可以通过输入完整路由（控制器/操作）来获取有关完整路由的帮助：

```php
$ ./yii help message/config

```

这返回了一个包含操作描述、用法和可用选项的输出：

```php
DESCRIPTION

Creates a configuration file for the "extract" command.

The generated configuration file contains detailed instructions on
how to customize it to fit for your needs. After customization,
you may use this configuration file with the "extract" command.

USAGE

yii message/config <filePath> [...options...]

- filePath (required): string
 output file name or alias.

OPTIONS

--appconfig: string
 custom application configuration file path.
 If not set, default application configuration is used.

--color: boolean, 0 or 1
 whether to enable ANSI color in the output.
 If not set, ANSI color will only be enabled for terminals that support it.

--interactive: boolean, 0 or 1 (defaults to 1)
 whether to run the command interactively.

```

# 创建控制台控制器

控制台控制器与之前创建的 Web 控制器完全相同。它扩展了 `\yii\console\Controller` 基类，并可以返回一个整数值，表示操作的响应状态（0 表示操作成功执行），也称为 `退出码`。

只有当控制器的 `public` 属性名称由接受 `actionID` 作为参数的 `options()` 方法返回时，它们才能作为选项提供；因此，响应可以根据 `actionID` 进行定制。

`options()` 方法的响应是一个表示控制器公共属性名的文本字符串数组。

从我们之前在 `yiiadv` 文件夹中安装的高级模板应用程序开始，让我们在 `console/controllers/MyExampleController.php` 中创建一个名为 `MyExampleController` 的新控制台控制器，其内容如下：

```php
<?php

namespace console\controllers;

use \yii\console\Controller;

/**
 * This is an example controller
 */
class MyExampleController extends Controller
{
    public $option1;
    public $option2;

    public function options($action)
    {
        return ['option1'];
    }

    /**
     * Simply return a welcome text
     */
    public function actionTest($param1)
    {
        echo 'this is my first controller using console application';
        echo "\n";
        echo "You have passed param1 with value: ".$param1;
        echo "\n";
        echo "Value of option1 is: ".$this->option1;
        echo "\n";

        // equivalent to return 0;
        return Controller::EXIT_CODE_NORMAL;
    }

}

?>
```

此控制器包含两个公共属性，但只有 `option1` 可以从控制台使用，因为它是由 `options()` 方法返回的。我们将显示以下命令的结果：

```php
$ ./yii help my-example

```

前面的命令将返回以下输出：

```php
DESCRIPTION

This is an example controller

SUB-COMMANDS

- my-example/test  Simply return a welcome text

To see the detailed information about individual sub-commands, enter:

 yii help <sub-command>

```

如果我们需要有关 `test` 操作的其他详细信息，我们可以通过指定完整路由来启动前面的命令：

```php
$ ./yii help my-example/test

```

现在，尝试使用路由 `my-example/test` 启动命令，不添加任何参数：

```php
$ ./yii my-example/test

```

我们将收到一个关于缺少 `param1` 的错误。以下是正确的语法：

```php
$ ./yii my-example/test "this is value for param1"

```

前面的命令将返回以下输出，没有 `option1` 的值：

```php
this is my first controller using console application
You have passed param1 with value: this is value for param1
Value of option1 is:.

```

我们也可以通过在命令后附加 `--option1` 来传递值 `option1`，如下所示：

```php
$ ./yii my-example/test "this is value for param1" --option1="this is value for option1"

```

前面的命令将返回完整的输出，如下所示：

```php
this is my first controller using console application
You have passed param1 with value: this is value for param1
Value of option1 is: this is value for option1

```

## 示例 – 为已过期的预订设置警报标志

现在，让我们通过一个示例来说明如何使用控制台命令来执行维护操作。

在控制台控制器中，我们可以访问项目中所有可用的模型、组件和扩展，以及我们在 Web 应用程序中所做的。因此，我们将以与 Web 应用程序相同的方式操作数据。

从前几章中使用的预订数据库表开始，我们将添加一个新的布尔字段，命名为 expired，用于设置哪些预订已超出结束日期。

这是存储在 MySQL 服务器中的 `reservation` 表的结构：

```php
CREATE TABLE `reservation` (
 `id` int(11) NOT NULL AUTO_INCREMENT,
 `room_id` int(11) NOT NULL,
 `customer_id` int(11) NOT NULL,
 `price_per_day` decimal(20,2) NOT NULL,
 `date_from` date NOT NULL,
 `date_to` date NOT NULL,
 `reservation_date` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP,
 `expired` int(1) NOT NULL DEFAULT '0',
 PRIMARY KEY (`id`)
 )
```

现在，让我们插入一些记录以进行模拟。如果今天是 `date_to value` 之后，我们将更新 `expired` 字段的值为 `1`；否则，它将是 `0`。

这些是要插入到 `reservation` 数据库表中的记录：

```php
INSERT INTO `reservation` (`id`, `room_id`, `customer_id`, `price_per_day`, `date_from`, `date_to`, `reservation_date`, `expired`) VALUES
(1, 2, 1, 90.00, '2015-02-10', '2015-05-23', '2015-05-24 22:45:37', 0),
(2, 2, 1, 48.00, '2019-08-27', '2019-08-31', '2015-05-24 22:45:37', 0),
(3, 1, 2, 105.00, '2015-09-24', '2015-10-06', '2015-06-03 00:21:14', 0),
(4, 1, 2, 150.00, '2015-06-22', '2015-06-28', '2015-06-21 22:24:25', 0),
(5, 1, 2, 150.00, '2015-07-22', '2015-08-28', '2015-06-21 22:24:34', 0);
```

### 注意

确保用户存在于用户数据库表中

现在，在 `console/controllers/ReservationsController.php` 中创建一个新的控制台控制器，内容如下：

```php
<?php

namespace console\controllers;

use \yii\console\Controller;

/**
 * Manage reservations
 */
class ReservationsController extends Controller
{
    /**
     * Update 'expired' field of reservations
     */
    public function actionUpdateExpired()
    {
        $models = \common\models\Reservation::find()->all();

        foreach($models as $m)
        {
            echo sprintf('Check reservation #%d - date_to = %s - status : %s', $m->id, $m->date_to, (strtotime($m->date_to)<=time())?'OK':'Expired');
            echo "\n";
            // Set expired field. I'll for every model because if we could have changed 'date_to' value.
            $m->expired = (strtotime($m->date_to)<=time())?0:1;
            $m->save();
        }
        // equivalent to return 0;
        return Controller::EXIT_CODE_NORMAL;
    }
}
?>
```

在 `actionUpdateExpired` 中，我们向控制台显示每个模型的一些数据，例如 `id`、`date_to` 和 `status`。然后，我们将根据 `date_to` 值为每个模型设置 `expired` 字段的值。

最后，我们将运行以下命令：

```php
$ ./yii reservations/update-expired

```

这将返回以下输出：

```php
Check reservation #1 - date_to = 2015-05-23 - status : OK
Check reservation #2 - date_to = 2019-08-31 - status : Expired
Check reservation #3 - date_to = 2015-10-06 - status : Expired
Check reservation #4 - date_to = 2015-06-28 - status : OK
Check reservation #5 - date_to = 2015-08-28 - status : OK

```

# 格式化来自控制台的输出

基础类控制台控制器 `yii\console\Controller` 支持显示彩色和格式化输出的方法。

有两种标准方法来显示输出，具体如下：

+   `stdout`：这会将字符串打印到 `STDOUT`

+   `strerr`：这会将字符串打印到 `STDERR`

这两种方法都支持更多参数：第一个是要显示的文本字符串，另一个包括可以传递的格式化选项，以生成美观的输出。

有颜色和打字格式的格式化选项；这些是由 `\yii\helpers\Console` 中的常量定义的；例如，`BG_CYAN` 用于青色背景，`BG_RED` 用于红色背景，`UNDERLINE` 用于下划线文本。

让我们使用以下代码看看一个例子：

```php
$this->stdout("Hello?\n", Console::BOLD);
```

这将显示 `Hello?`（带有回车符），字体加粗。有时，可能不会显示任何效果，因为我们的终端不支持颜色。

在这种情况下，控制台控制器的一个方法将帮助我们验证我们的终端功能：`isColorEnabled()` 返回一个布尔值，指示终端是否支持 ANSI 颜色。

两种方法 `strout` 和 `strerr` 都应用于整个文本字符串，并且作为第一个参数传递。如果我们只想将一些功能应用于文本的某个部分，我们必须使用返回 ANSI 格式化字符串的 `ansiFormat` 方法。

让我们举一个例子。创建一个控制器来检查控制台是否支持 ANSI，并尝试打印彩色文本，如果这个功能被支持。

然后，在 `console/controllers/ColorController.php` 中创建一个名为 `ColorController` 的新控制器，内容如下：

```php
<?php

namespace console\controllers;

use \yii\console\Controller;
use \yii\helpers\Console;

/**
 * Colors dedicated controller
 */
class ColorController extends Controller
{
    /**
     * Simply return a welcome text
     */
    public function actionIsClientEnabled()
    {
        if($this->isColorEnabled())
        {
            $this->stdout('OK, terminal supports colors!');
        }
        else
        {
            $this->stdout('NOT OK, terminal does not support colors!'); 

        }

        $this->stdOut("\n");        

        // equivalent to return 0;
        return Controller::EXIT_CODE_NORMAL;
    }

    public function actionPrintColouredText()
    {
        $colouredText = $this->ansiFormat('This text is coloured', Console::FG_RED);
        $normalText ="This text is normal";

        $this->stdout(sprintf("%s - %s\n", $normalText, $colouredText));
    }

}

?>
```

我们调用 `launch` 来检查客户端是否支持 ANSI 颜色：

```php
$ ./yii color/is-client-enabled

```

并且为了显示彩色文本（如果客户端支持的话）：

```php
$ ./yii color/print-coloured-text

```

`\yii\helpers\` 下的 `Console` 类包含许多其他有用的方法来格式化文本和输出，例如 `confirm()` 或 `prompt()` 从用户获取输入，或者 `progress` 创建进度条以显示执行状态。

# 实现和执行 cron 作业

控制台应用程序的主要用途在于使用 cron 作业（在 Linux 或 Unix 机器上）执行周期性任务。

我们可以使用控制台应用程序发送大量电子邮件以执行系统维护或检查应用程序的特定状态。

在下一个示例中，我们将看到如何发送包含当日预订摘要的电子邮件。

## 示例 - 发送包含当日新预订的电子邮件

此示例说明了如何发送包含新每日预订摘要的电子邮件。

首先，让我们在`console/config/main.php`中配置`mailer`组件，如果尚未配置。

只需传递几个参数给组件：

```php
    'components' => [
    ..
    ..

        'mailer' => [
            'class' => 'yii\swiftmailer\Mailer',
            'viewPath' => '@common/mail',
            // send all mails to a file by default. You have to set
            // 'useFileTransport' to false and configure a transport
            // for the mailer to send real emails.
            'useFileTransport' => true,
        ],
..
..
    ],
];
```

`class`参数表示处理组件的类，`viewPath`参数表示电子邮件或电子邮件模板的视图存储位置；最后一个参数`useFileTransport`表示电子邮件发送方法。

现在，在`ReservationsController`中，在`console/controllers/ReservationsController.php`下添加方法`actionReservationsOfTheDay`，该方法发送每日预订的内容：

```php
    public function actionReservationsOfTheDay($currentDate=null)
    {
        if($currentDate == null) $currentDate = date('Y-m-d');
        $models = \common\models\Reservation::find()->where('DATE(reservation_date) = "'.$currentDate.'"')->all(); 
        \Yii::$app->mailer->compose(['html' => 'reservationsOfTheDay-html', 'text' => 'reservationsOfTheDay-text'], ['models' => $models, 'currentDate' => $currentDate])
            ->setFrom('myemail@example.com')
            ->setTo('administrator@example.com')
            ->setSubject('Reservations of the day: '.$currentDate)
            ->send();

    }
```

### 注意

建议将`from`电子邮件参数，例如，放在`params.php`文件中，该文件包含整个应用程序中可用的所有全局参数。

此方法简单地从输入中获取`currentDate`参数，以便我们可以根据需要更改评估日期；动作体找到输入日期的预订并将它们传递给`html`和`text`格式的电子邮件视图`reservationsOfTheDay`。

现在，我们必须在`common/mail`中创建电子邮件格式的內容，创建两个文件：`reservationsOfTheDay-html.php`和`reservationsOfTheDay-text.php`。

这是 HTML 版本的內容：

```php
There are <?= count($models) ?> reservations for the date <?= $currentDate ?>

<br /><br />

<?php if(count($models)>0) { ?>
    <b>This is a summary:</b>

    <br />

    <table>
        <tr>
            <td>Reservation #</td>
            <td>Room</td>
            <td>Customer</td>
            <td>Price per day</td>
            <td>Date from</td>
            <td>Date to</td>
        </tr>

        <?php foreach($models as $m) { ?>
        <tr>
            <td><?= $m->id ?></td>
            <td><?= $m->room->floor.' '.$m->room->number ?></td>
            <td><?= $m->customer->surname.' '.$m->customer->name ?></td>
            <td><?= $m->price_per_day ?></td>
            <td><?= $m->date_from ?></td>
            <td><?= $m->date_to ?></td>
        </tr>    
        <?php } ?>

    </table>
<?php } else { ?>
    <i>There is no summary for current date</i>
<?php } ?>    
```

这是文本格式的对应内容（对于 HTML 电子邮件客户端不是必需的）：

```php
There are <?= count($models) ?> reservations for the date <?= $currentDate ?>

<?php if(count($models)>0) { ?>
    This is a summary

    <?php foreach($models as $m) { ?>
        Reservation #: <?= $m->id ?> - Room: <?= $m->room->floor.' '.$m->room->number ?> - Customer: <?= $m->customer->surname.' '.$m->customer->name ?> - Price per day: <?= $m->price_per_day ?> - Date from: <?= $m->date_from ?> - Date to: <?= $m->date_to ?>
    <?php } ?>
<?php } else { ?>
    There is no summary for the current date
<?php } ?>
```

可以通过启动以下命令来执行：

```php
$ ./yii reservations/reservations-of-the-day

```

我们还可以调用传递日期参数来更改要检查的日期，例如，检查`2015-08-05`的预订：

```php
$ ./yii reservations/reservations-of-the-day "2015-08-05"

```

最后，根据操作系统，将此命令附加到周期性任务调度程序，例如 Linux 或 Unix 环境中的 cron。

# 摘要

在本章中，我们讨论了与 Yii 的高级模板一起安装的第三种默认应用程序，即控制台应用程序。

我们已经看到了控制台应用程序和 Web 应用程序之间的主要区别，我们学习了如何创建我们的第一个控制台控制器，处理传递给动作的选项和参数。然后，我们通过一个具体的示例应用了控制台应用程序，例如对预订表进行维护操作，以更新预订的状态为已过期。

然后，我们关注了控制台应用程序如何使用颜色和文本格式化功能生成漂亮的输出。

最后，我们掌握了如何使用控制台控制器动作创建一个完整的周期性任务，以发送包含当日预订的每日总结电子邮件。

在最后一章，我们将看到我们发展的最终阶段，在这个阶段我们必须使代码可重用，但尤其是可维护的。
