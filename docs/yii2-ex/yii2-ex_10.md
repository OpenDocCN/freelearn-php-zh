# 第十章。本地化应用

本章解释了如何编写多语言应用。本地化，也称为国际化（I18N），负责确保软件应用可以在不同的语言中适应和渲染，而无需更改源代码。这在用户说不同语言的 Web 应用中尤为重要。

Yii 提供了强大的工具来处理这项任务，可以选择文件或数据库方法（根据应用的复杂性）。我们将涵盖以下主题：

+   设置默认语言

+   基于文件的翻译

    +   示例 - 使用基于文件的翻译为整个网站

+   占位符格式化

+   基于数据库的翻译

    +   示例 - 使用数据库翻译房间描述

# 设置默认语言

Yii 应用使用两种语言：源语言和目标语言。

源语言指定编写源代码所使用的语言；默认设置是 `en-US`，建议不要更改此值，因为英语是软件开发中最常用和最知名的语言。另一方面，目标语言用于向最终用户显示内容，我们将专门讨论这个方面。

可以使用配置文件中的 `language` 属性设置此语言：

```php
return [
    // set target language to be Italian
    'language' => 'it-IT',

      ....
      ....
];
```

或者，你可以使用以下代码：

```php
// change target language to Italian
\Yii::$app->language = 'it-IT';
```

现在，让我们看看如何在实践中处理应用本地化。

# 基于文件的翻译

这是将文本消息从一种语言翻译到另一种语言的最简单方法。基本上，每个语言都有一个或多个包含关键词文本表示的文件；我们将把这些关键词放入源代码中，框架将用文本替换它们。

关键词-文本翻译的成对组合按类别分组，这些类别代表存储它们的文件名。这些成对组合是数组键值对，其中键表示关键词，值表示文本翻译。

默认情况下，包含特定语言翻译的路径文件夹位于 `@app/messages/<language>/<category>.php`。因此，如果我们正在为 `app` 类别和 `en-US` 语言编写翻译，例如，翻译文件的完整路径将在 `@app/messages/en-US/app.php`。

在源代码中，使用接受四个参数的 `Yii::t()` 静态方法激活翻译，但只需要前两个参数；第一个是类别，第二个是要翻译的消息。

现在，我们想要举一个例子，在这个例子中我们将用两种语言：英语和意大利语来编写一个经典的 `Hello World!`。然而，将其翻译成任何其他语言也同样简单。

在之前的基本模板项目上，在 `basic/controllers/FileTranslatorController.php` 中编写一个新的控制器 `FileTranslatorController`，内容如下：

```php
<?php

namespace app\controllers;

use Yii;
use yii\web\Controller;

class FileTranslatorController extends Controller
{
    public function actionIndex()
    {
        \Yii::$app->language = 'en-US';
        $englishText = \Yii::t('app', 'Hello World!');

        \Yii::$app->language = 'it-IT';
        $italianText = \Yii::t('app', 'Hello World!');

        return $this->render('index', ['englishText' => $englishText, 'italianText' => $italianText]);
    }
}
```

`actionIndex()`中的前两行源代码将设置应用程序语言为`en-US`，然后它们将`basic/messages/en-US/app.php`文件中`Hello World!`键的内容存储在`$englishText`变量中。

同样，`actionIndex()`中的最后两行源代码将设置应用程序语言为`it-IT`，然后它们将`basic/messages/it-IT/app.php`文件中`Hello World!`键的内容存储在`$italianText`变量中。

`basic/views/file-translator/index.php`中的视图内容如下所示：

```php
<b>Display Hello World! in two language: English and Italian</b>

<br /><br />

In English:
<?= $englishText ?>

<br /><br />
In Italian:
<?= $italianText ?>
```

现在，我们需要定义英语和意大利语翻译的文件语言。

如果`basic/messages`中不存在`messages`文件夹，我们将创建它；然后，创建两个新的文件夹，分别命名为`en-US`和`it-IT`。在每个文件夹中，添加一个名为`app.php`的新文件。

对于包含英文翻译的文件`basic/messages/en-US/app.php`，让我们写下：

```php
<?php

return [
    'Hello World!' => 'Hello world!',
];

?>
```

而对于`basic/messages/it-IT/app.php`中的意大利语翻译，让我们写下：

```php
<?php

return [
    'Hello World!' => 'Ciao Mondo!',
];

?>
```

您可以浏览到`http://hostname/basic/file-translator/index`来查看输出。

## 示例 - 使用基于文件的翻译为整个网站

将翻译应用到整个网站是繁琐的，而且最重要的是，你可能会错过一些翻译。Yii 提供了一个强大的工具，可以自动为所有我们想要的语种生成消息的 PHP 文件。

### 注意

这个强大的工具是一个名为`message`的控制台命令；因此，我们需要控制台访问权限。

此命令需要两个步骤：

1.  创建一个配置文件，我们将在这里指定`languages`属性，即我们想要在项目中支持的语言，以及`messagePath`属性，或者说，翻译消息的存储位置。

1.  启动`message`命令。

对于第一步，前往控制台，在项目的根目录下，即`yii`文件所在的位置。

如果我们正在处理基本模板，我们将启动以下命令：

```php
$ ./yii message/config config/i18n.php

```

第一个参数`message/config`是在`message`控制器上调用的`config`动作，第二个参数是我们想要保存配置的文件路径（在这种情况下，`config/i18n.php`，但我们可以写任何内容）。

如果我们正在处理高级模板，我们将启动以下命令：

```php
./yii message/config common/config/i18n.php

```

唯一的区别是，在最后一个命令中，我们指定了消息命令翻译的配置文件位于`common/config`而不是`config`文件夹中。

现在，如果我们打开`config/i18n.php`，我们应该看到`message`命令的默认配置文件，其外观应如下所示：

```php
<?php

return [
    // string, required, root directory of all source files
    'sourcePath' => __DIR__ . DIRECTORY_SEPARATOR . '..',
    // array, required, list of language codes that the extracted messages
    // should be translated to. For example, ['zh-CN', 'de'].
    'languages' => ['de'],
    // string, the name of the function for translating messages.
    // Defaults to 'Yii::t'. This is used as a mark to find the messages to be
    // translated. You may use a string for single function name or an array for
    // multiple function names.
    'translator' => 'Yii::t',
    // boolean, whether to sort messages by keys when merging new messages
    // with the existing ones. Defaults to false, which means the new (untranslated)
    // messages will be separated from the old (translated) ones.
    'sort' => false,
    // boolean, whether to remove messages that no longer appear in the source code.
    // Defaults to false, which means each of these messages will be enclosed with a pair of '@@' marks.
    'removeUnused' => false,
    // array, list of patterns that specify which files/directories should NOT be processed.
    // If empty or not set, all files/directories will be processed.
    // A path matches a pattern if it contains the pattern string at its end. For example,
    // '/a/b' will match all files and directories ending with '/a/b';
    // the '*.svn' will match all files and directories whose name ends with '.svn'.
    // and the '.svn' will match all files and directories named exactly '.svn'.
    // Note, the '/' characters in a pattern matches both '/' and '\'.
    // See helpers/FileHelper::findFiles() description for more details on pattern matching rules.
    'only' => ['*.php'],
    // array, list of patterns that specify which files (not directories) should be processed.
    // If empty or not set, all files will be processed.
    // Please refer to "except" for details about the patterns.
    // If a file/directory matches both a pattern in "only" and "except", it will NOT be processed.
    'except' => [
        '.svn',
        '.git',
        '.gitignore',
        '.gitkeep',
        '.hgignore',
        '.hgkeep',
        '/messages',
    ],

    // 'php' output format is for saving messages to php files.
    'format' => 'php',
    // Root directory containing message translations.
    'messagePath' => __DIR__,
    // boolean, whether the message file should be overwritten with the merged messages
    'overwrite' => true,

    /*
    // 'db' output format is for saving messages to database.
    'format' => 'db',
    // Connection component to use. Optional.
    'db' => 'db',
    // Custom source message table. Optional.
    // 'sourceMessageTable' => '{{%source_message}}',
    // Custom name for translation message table. Optional.
    // 'messageTable' => '{{%message}}',
    */

    /*
    // 'po' output format is for saving messages to gettext po files.
    'format' => 'po',
    // Root directory containing message translations.
    'messagePath' => __DIR__ . DIRECTORY_SEPARATOR . 'messages',
    // Name of the file that will be used for translations.
    'catalog' => 'messages',
    // boolean, whether the message file should be overwritten with the merged messages
    'overwrite' => true,
    */
];
```

配置非常易于阅读，因此我们只需解释其主要属性：`languages`、`messagePath`和`except`。

`languages`属性定义了在 Web 项目中支持哪些语言。例如，我们可以写：

```php
'languages' => ['en', 'it', 'fr'],
```

前面的命令支持并自动生成英语、意大利语和法语的消息。

`messagePath` 属性定义了自动生成的消息应该保存的位置。建议指向 `messages` 文件夹（如果不存在，必须创建）；这样我们可以在基本模板中写入以下内容：

```php
'messagePath' =>  __DIR__ . DIRECTORY_SEPARATOR . '..' . DIRECTORY_SEPARATOR . 'messages',
```

在这里，`__DIR__` 指的是 `config` 文件夹，而在基本模板中，它是 `basic/config` 文件夹。

一旦我们启动了 `message` 命令，它将查找所有包含 `.php` 文件的文件夹和子文件夹，如 `only` 属性中所示（只有 `.php` 文件将被处理）。

因此，在项目的根文件夹中，有一些文件夹，如 `vendor`，与我们无关。

因此，我们将 `/vendor` 值添加到 `except` 属性中，以指示 `message` 命令不会查看此文件夹，这样：

```php
    'except' => [
        '.svn',
        '.git',
        '.gitignore',
        '.gitkeep',
        '.hgignore',
        '.hgkeep',
        '/messages',
        '/vendor'
    ],
```

对于步骤 2，我们现在将尝试启动以下命令：

```php
$ ./yii message config/i18n.php

```

它将在 `sourcePath` 属性指定的文件夹和子文件夹中的所有文件中找到 `Yii::t` 标记，考虑 `except` 属性以排除我们不想查找的文件和文件夹。

翻译后的消息将（如果不存在）创建在 `messagePath` 文件夹中，在我们的例子中，是从项目的根文件夹开始的 `messages` 文件夹。

如果所有搜索的文件中都没有 `Yii::t` 标记，则相对语言子文件夹将为空。

例如，打开 `basic/controller/SiteController.php` 中的 `SiteController` 并按以下方式更改 `actionIndex` 内容：

```php
    public function actionIndex()
    {
        $message = \Yii::t('app', 'this message must be translated!');

        return $this->render('index');
    }
```

现在，重新启动 `message` 命令：

```php
$ ./yii message config/i18n.php

```

然后，检查 `basic/messages/en` 文件夹。我们将找到一个包含 `this message must be translated` 键的 `app.php` 文件，我们必须填写值以指定翻译。

# 占位符格式

`Yii:t` 方法不仅限于用其他语言的翻译替换字符串，它还处理源字符串的特定格式化以支持许多类型的泛化。

首先，`Yii:t()` 支持以下两种格式的占位符：

+   `{nameOfPlaceholder}` 格式的字符串

+   整数在 `{0}` 格式，这种类型的占位符是从零开始的

要替换占位符的值数组作为 `Yii:t()` 方法的第三个参数传递。

例如，我们想要通过添加自定义名字到文本来显示一个只包含 `Hello World, I'm ...` 的页面。

创建 `basic/controllers/FileTranslatorController.php`：

```php
    public function actionHelloWorldWithName($name='')
    {
        $text = \Yii::t('app', 'Hello World! I\'m {name}', ['name' => $name]);

        return $this->render('helloWorldWithName', ['text' => $text]);        
    }
```

现在，在 `basic/views/file-translator/helloWorldWithName.php` 中创建视图，只需使用以下命令：

```php
<?= $text ?>
```

它将显示从控制器传递的 `$text` 值。

通过将浏览器指向 `http://hostname/basic/web/file-translator/hello-world-with-name` 并传递 `?name=` 参数来测试它，否则文本末尾将没有名字。

可以使用我们刚刚看到的 `message` 命令来准备翻译：

```php
$ ./yii message config/i18n.php

```

这将自动在 `basic/messages` 子文件夹中创建一个新的标记 `Hello World! I\'m {name}`。

可以使用两个其他属性`ParameterType`和`ParameterStyle`来专门化占位符，在`PlaceholderName`后添加逗号。因此，指定占位符的完整形式如下：

```php
{PlaceholderName, ParameterType, ParameterStyle}
```

在这里，`ParameterType`可以是：

+   `number`：参数样式可以是整数、货币、百分比或自定义模式（例如，000）

+   `date`：参数样式可以是短格式、中格式、长格式、完整格式或自定义模式（例如，dd/mm/yyyy）

+   `time`：参数样式可以是短格式、中格式、长格式、完整格式或自定义模式（例如，hh:mm）

+   `spellout`：没有参数样式

+   `ordinal`：没有参数样式

+   `duration`：没有参数样式

最常用的消息格式可能是`plural`，这允许我们根据传递的参数数量指定不同的键字符串。

以下代码作为示例：

```php
// if $n = 0, it shows "There are no books!"
// if $n = 1, it shows "There is one book!"
// if $n = 4, it shows "There are 4 books!"

echo \Yii::t('app', 'There {n, plural, =0{are no books} =1{is one book} other{are # books}}!', ['n' => $n]);
```

在这里，`=0`代表当`$n`为`0`时显示的消息，`=1`代表当`$n`为`1`时显示的消息，而`other`代表当`$n`不是`0`和`1`时显示的消息。

# 基于数据库的翻译

Yii 还支持将数据库作为消息翻译的存储选项。

如果我们在基本模板中工作，必须在`config/web.php`文件中显式配置，如果我们在高级模板中工作，则必须在`common/config/main.php`中配置。

接下来，我们需要添加两个额外的数据库表来管理消息源和消息翻译。

首先根据 Yii 官方文档中的建议创建数据库表，文档地址为[`www.yiiframework.com/doc-2.0/yii-i18n-dbmessagesource.html`](http://www.yiiframework.com/doc-2.0/yii-i18n-dbmessagesource.html)：

```php
CREATE TABLE source_message (
    id INTEGER PRIMARY KEY AUTO_INCREMENT,
    category VARCHAR(32),
    message TEXT
);

CREATE TABLE message (
    id INTEGER,
    language VARCHAR(16),
    translation TEXT,
    PRIMARY KEY (id, language),
    CONSTRAINT fk_message_source_message FOREIGN KEY (id)
        REFERENCES source_message (id) ON DELETE CASCADE ON UPDATE RESTRICT
);
```

### 注意

可以在配置文件中自定义表名。

`source_message`表将存储所有用源语言编写的消息；`message`表将存储所有翻译；这两个表通过`id`字段连接在一起。

在下一个示例中，让我们为每个表插入一条记录：

```php
INSERT INTO `source_message` (`id`, `category`, `message`) VALUES
(1, 'app', 'Hello World from Database!');

INSERT INTO `message` (`id`, `language`, `translation`) VALUES
(1, 'it', 'Ciao Mondo dal Database!');
```

现在，是时候对配置进行一些更改了。我们需要在`config/web.php`配置文件的`components`部分插入`i18n`属性（基于基本模板）：

```php
'components' => [
    // ...
    'i18n' => [
        'translations' => [
            'app' => [
                    'class' => 'yii\i18n\DbMessageSource',
                    //'messageTable' => 'message,
                    //'sourceMessageTable' => 'source_message,

            ],
        ],
    ],
],
```

此组件，i18n，默认使用`yii\i18n\PhpMessageSource`类，并已用于基于文件的翻译。

现在，我们想显示意大利语的消息。在`basic/controllers/FileTranslatorController.php`中创建一个新的操作`actionHelloWorldFromDatabase()`，内容如下：

```php
     public function actionHelloWorldFromDatabase()
    {
        \Yii::$app->language = 'it';
        $text = \Yii::t('app', 'Hello World from Database!');

        return $this->render('helloWorldFromDatabase', ['text' => $text]);        
    }
```

`basic/views/file-translator/helloWorldFromDatabase`视图将显示`$text`内容：

```php
<?= $text ?>
```

通过将浏览器指向`http://hostname/basic/web/file-translator/hello-world-from-database`来测试它。如果一切正常，我们应该看到`Ciao Mondo dal Database!`，这是`Hello World from Database!`的意大利语版本。

## 示例 - 使用数据库翻译房间描述

本例将向您展示如何使用数据库作为存储选项来翻译房间的描述。我们将为`message`和`source_message`数据库表创建模型，因为我们打算使用 ActiveRecord 来管理所有控制翻译的表的记录。

首先，我们将使用 Gii 创建`message`和`source_message`数据库表的模型。在基本模板中，将浏览器指向`http://hostname/basic/web/gii`，然后转到模型生成器。Gii 将在`basic/models`文件夹中创建`Message`和`SourceMessage`模型。

接下来，我们想要创建一个包含原始语言和所有其他翻译的描述的表单。

为此目的，我们将在`basic/views/rooms/indexWithTranslatedDescriptions.php`中创建一个视图，如下所示：

```php
<?php
use yii\helpers\Url;
use yii\widgets\ActiveForm;
?>

<div class="row">
    <div class="col-md-4">
        <legend>Rooms with translated descriptions</legend>

        <?php $form = ActiveForm::begin([]); ?>
        <table class="table">
            <tr>
                <th>#</th>
                <th>Floor</th>
                <th>Room number</th>
                <th>Description - English</th>
                <th>Description - Italian</th>
                <th>Description - French</th>
            </tr>
            <?php for($k=0;$k<count($rooms);$k++) : ?>
                <?php $room = $rooms[$k]; ?>
                <input type="hidden" name="Room[<?= $k ?>][id]" value="<?= $room->id ?>" />
                <tr>
                    <td><?php echo $k+1 ?></td>
                    <td><?php echo $room->floor ?></td>
                    <td><?php echo $room->room_number ?></td>
                    <td><input type="text" name="Room[<?= $k ?>][description][en]" value="<?= $room->description ?>" /></td>
                    <td><input type="text" name="Room[<?= $k ?>][description][it]" value="<?= Yii::$app->i18n->translate('app', $room->description, [], 'it') ?>" /></td>
                    <td><input type="text" name="Room[<?= $k ?>][description][fr]" value="<?= Yii::$app->i18n->translate('app', $room->description, [], 'fr') ?>" /></td>
                </tr>
            <?php endfor; ?>
        </table>
        <br />
        <input type="submit" class="btn btn-primary" value="Submit descriptions" />
        <?php ActiveForm::end(); ?>
    </div>
</div>
```

我们将使用`Yii::$app->i18n->translate`方法检查其他语言的翻译，该方法接受：

+   分类

+   待翻译的消息

+   消息参数

+   语言

现在是时候在`basic/controllers/RoomsController.php`中添加`actionIndexWithTranslatedDescriptions()`了：

```php
    public function actionIndexWithTranslatedDescriptions()
    {
        if(isset($_POST['Room']))
        {
            $roomsInput = $_POST['Room'];
            foreach($roomsInput as $item)
            {
                $sourceMessage = \app\models\SourceMessage::findOne(['message' => $item['description']]);

                // If null, I need to create source message
                if($sourceMessage == null)
                {
                    $sourceMessage = new \app\models\SourceMessage();
                }
                $sourceMessage->category = 'app';
                $sourceMessage->message = $item['description']['en'];
                $sourceMessage->save();

                $otherLanguages = ['it', 'fr'];

                foreach($otherLanguages as $otherLang)
                {
                    $message = \app\models\Message::findOne(['id' => $sourceMessage->id, 'language' => $otherLang]);
                    if($message == null)
                    {
                        $message = new \app\models\Message();
                    }
                    $message->id = $sourceMessage->id;
                    $message->language = $otherLang;
                    $message->translation = $item['description'][$otherLang];
                    $message->save();
                }

                // Room to update
                $roomToUpdate = \app\models\Room::findOne($item['id']);
                $roomToUpdate->description = $item['description']['en'];
                $roomToUpdate->save();
            }
        }

        $rooms = Room::find()
        ->all();

        return $this->render('indexWithTranslatedDescriptions', ['rooms' => $rooms]);
    }
```

### 注意

如果我们无法访问 URL，请检查此控制器`behaviors()`方法返回的`access`属性，以确保此操作被允许。

在此代码之上，我们将检查`$_POST`数组是否已填充；在这种情况下，我们将从视图中传递的描述中获取`$sourceMessage`对象。接下来，我们可以为任何我们想要的语种创建或更新消息模型。最后，我们还将保存房间对象，最终更改其描述字段。

使用此解决方案，每次我们想要更改描述时，由于文本已更改，都会创建一个新的记录。

# 摘要

在本章中，我们看到了如何在我们的应用程序中配置多种语言。我们发现处理国际化有两种存储选项：文件和数据库。对于小型项目建议使用文件，对于大型项目建议使用数据库。

我们已经发现如何通过控制台中的'message'命令从整个网站中抓取占位符，以及如何创建包含格式化信息的占位符。

最后，我们已经将数据库配置为翻译的存储目标，并创建了一个完整的示例来处理不同语言的房间描述。

在下一章中，我们将学习如何使用 Yii 2 的新集成管理创建 RESTful 网络服务。
