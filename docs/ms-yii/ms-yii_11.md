# 第十一章。国际化与本地化

在开发现代网络应用程序时，我们经常发现需要确保我们的语言对说和读写与我们不同的用户来说是可读的。为了帮助实现这一点，Yii2 提供了对国际化（i18n）和本地化（l10n）的内置支持。**国际化**是指规划和实施消息和视图的过程，以便它们可以轻松地适应其他语言。另一方面，**本地化**是指将我们的应用程序适应特定的语言或文化，包括我们的应用程序的外观和感觉，以符合特定语言或地区或市场的信息接受展示。在本章中，我们将了解如何使用 Yii2 的内置功能将我们的应用程序翻译和本地化为多种语言。

### 小贴士

i18n 和 l10n 是数字缩写词，而不是首字母缩略词。国际化简写为 i18n，因为它以字母 "I" 开头，后面跟着 18 个字符，并以字母 "N" 结尾。同样，本地化简写为 l10n，因为它以字母 "L" 开头，有 10 个额外的字母，然后以字母 "N" 结尾。这些缩写词仅仅是为了缩短单词，没有其他含义。在本章中，我们将使用全称和缩写形式来指代这两个词。

# 配置 Yii2 和 PHP

在我们开始使用 Yii2 的本地化功能之前，我们首先需要确保 `intl` PHP 扩展已安装。此扩展用于为 Yii2 提供大多数 i18n 功能，包括 Yii2 的消息和日期格式化器。虽然 Yii2 在此扩展未安装的情况下有一些内置的回退机制，但强烈建议您事先安装它。

## Intl 扩展

许多默认的 PHP 安装都包含在 PHP 包中构建的 `intl` 扩展，但许多没有。幸运的是，有几种方法可以检查 `intl` 扩展是否已安装。对于那些更喜欢在网页浏览器中查看这些信息的人来说，只需在您的 webroot 中创建一个包含以下内容的空白 PHP 文件，并扫描输出以检查 `intl` 扩展是否存在并已启用：

```php
<?php phpinfo();

```

如果你更喜欢使用命令行，你可以运行以下命令来检查你的 PHP 实例是否已安装 intl 扩展：

```php
php –m | grep intl

```

如果 `intl` 扩展在任何输出中都没有出现，您可以通过您的系统包管理器（根据您的操作系统使用 `apt` 或 `yum`）安装它，或者您可以手动安装它。一般来说，可以通过 `pecl` 命令手动编译和安装扩展：

```php
sudo pecl install intl

```

### 小贴士

如果你从源代码安装`intl`扩展，你需要确保你已经安装了`intl`库，最好是 49 或更高版本。如果你的系统有一个过时的`intl`库版本，你可以从[`site.icu-project.org/download`](http://site.icu-project.org/download)下载并编译一个新版本。此外，随`intl`库提供的时区数据可能已过时。确保你参考`intl`文档以获取有关如何更新你的`intl`时区数据的详细信息，链接为[`userguide.icu-project.org/datetime/timezone#TOC-Updating-the-Time-Zone-Data`](http://userguide.icu-project.org/datetime/timezone#TOC-Updating-the-Time-Zone-Data)。

编译完成后，你可以在你的`php.ini`配置文件中添加以下内容：

```php
extension=intl.so

```

在重启你的 Web 服务器和 PHP 进程后，你应该会看到`intl`扩展通过之前列出的其中一个命令出现。

### 提示

关于如何安装`intl`扩展的更多信息可以在 PHP 手册页面上找到，链接为[`secure.php.net/manual/en/intl.installation.php`](https://secure.php.net/manual/en/intl.installation.php)。

# 应用程序语言

在我们开始使用 Yii2 的翻译功能之前，我们需要定义应用程序所使用的语言。Yii2 中的应用程序语言由一个唯一的 ID 定义，该 ID 由 ISO-639 格式定义的语言 ID 和一个由 ISO-3166 格式定义的区域 ID 组成。例如，`en-US`代表在美国使用的英语。

### 提示

关于 ISO-639 的详细信息可以在[`www.loc.gov/standards/iso639-2/`](http://www.loc.gov/standards/iso639-2/)找到，关于 ISO-3166 的详细信息可以在[`www.iso.org/obp/ui/#search`](https://www.iso.org/obp/ui/#search)找到。

Yii2 在我们的配置文件中定义了两个语言属性，我们可以进行定义。第一个`sourceLanguage`属性代表我们的应用程序所使用的语言或区域，通常在应用程序请求生命周期内不会改变。第二个`language`属性代表用户正在使用的语言或区域，用户可以在任何时间点更改它（通常通过在页面上某个位置放置一个`language`选择小部件）。这两个配置选项结合使用，使我们能够通知 Yii2 如何处理我们希望被翻译的消息。在我们的`config/web.php`或`config/console.php`配置文件中，这两个选项可以设置如下：

```php
return [
    'language' => 'ru-RU',
    'sourceLanguage' => 'en-US',
];
```

### 提示

默认情况下，Yii2 会将`sourceLanguage`属性设置为`en-US`。

## 以编程方式设置应用程序语言

如果你正在开发一个多语言网站，而不是指定一个默认语言，你可能希望允许用户从下拉列表中选择他们的语言，并通过编程方式更改语言。为此，只需在你的代码中定义`Yii::$app->language`属性，并使用你选择的语言代码即可。

当以编程方式设置语言属性时，你通常会希望将用户的语言设置与其用户信息一起存储，或者作为一个会话变量。此外，你还需要确保在 Yii2 开始处理你的消息之前应用语言设置。设置此设置的好地方是在你的控制器流程的早期，例如在我们的控制器的`init()`方法中。

## 动态设置应用程序语言

除了在控制器中手动设置应用程序语言之外，我们还可以使用内容协商器过滤器（`yii\filters\ContentNegotiator`）从用户浏览器发送的`Accept-Language`头中确定用户的语言。要使用内容协商器过滤器，我们只需将`yii\filters\ContentNegotiator`添加到`config/web.php`配置文件的`bootstrap`部分，并指定我们想要自动支持的语言：

```php
return [
    // [...],
    'bootstrap' => [
        [
 'class' => 'yii\filters\ContentNegotiator',
 'languages' => [
 'en',
 'de',
 ],
 ],
    ],
];
```

### 小贴士

语言属性指定了当它们出现在`Accept-Language`头中时，Yii2 将自动将`Yii::$app->language`设置为哪些语言。在先前的例子中，我们只设置了语言为`en`或`de`。如果我们的`Accept-Language`头中出现的是我们应用程序配置中未列出的语言，我们将默认使用`sourceLanguage`属性中指定的语言。

除了全局设置之外，我们还可以在我们的控制器的`behaviors()`方法中设置内容协商器，并指定我们想要在该控制器中支持的语言。当你有一个可能支持比你的基础应用程序更多或不同语言的模块时，这很有益。在我们的控制器中，我们可以按照以下方式配置`yii\filters\ContentNegotiator`：

```php
public function behaviors()
{
    return [
        [
            'class' => 'yii\filters\ContentNegotiator',
            'languages' => [
                'en',
                'de',
            ],
        ],
    ];
}
```

### 小贴士

这个`yii\filters\ContentNegotiator`路径可以提供比仅设置应用程序语言更多的功能。有关内容协商过滤器的更多信息，请确保查看 Yii2 文档在[`www.yiiframework.com/doc-2.0/yii-filters-contentnegotiator.html`](http://www.yiiframework.com/doc-2.0/yii-filters-contentnegotiator.html)。

# 消息翻译

Yii2 的消息翻译服务通过在消息源文件中查找要翻译的消息，将给定的文本消息从源语言翻译到另一种语言。如果在目标语言的源中找到了消息，则返回该字符串而不是原始消息。如果找不到翻译文本，Yii2 将返回原始消息。

使用 Yii2 的消息翻译服务非常简单。在 Yii2 中翻译消息的第一步是将任何想要翻译的消息用`Yii::t()`静态方法包装，该方法的调用方式如下：

```php
Yii::t('app', 'My message to be translated');
```

第一个参数表示我们想要存储消息的类别，第二个参数表示我们想要翻译的消息。

## 消息源

然而，在 Yii2 可以翻译我们的消息之前，我们首先需要定义一个消息源，该消息源将存储我们的基础消息和翻译后的消息文件。Yii2 提供了三种不同的消息源选项：

+   `yii\i18n\PhpMessageSource`以键值数组格式存储消息文件

+   `yii\i18n\DbMessageSource`将消息存储在数据库表中

+   `yii\i18n\GettextMessageSource`使用 GNU Gettext MO 或 PO 文件来存储翻译后的消息

我们希望使用的消息源可以在应用程序配置文件中的组件部分声明，如下所示：

```php
<?php return [
    // [...],
    'components' => [
        // [...],
        'i18n' => [
 'translations' => [
 'app*' => [
 'class' => 'yii\i18n\PhpMessageSource',
 ],
 ],
        ],
    ]
];
```

在前面的代码块中，消息源由`yii\i18n\PhpMessageSource`提供。`app*`模式表示所有以`app`开头的消息都应该由指定的消息源处理。默认情况下，Yii2 将在`@app/messages`文件夹中存储消息，并将源语言默认设置为`en-US`；但是，可以通过在类别块中指定`basePath`和`sourceLanguage`属性来更改此行为，如下所示：

```php
<?php return [
    // [...],
    'components' => [
        // [...],
        'i18n' => [
            'translations' => [
                'app*' => [
                    'class' => 'yii\i18n\PhpMessageSource',
                    //'basePath' => '@app/messages',
                    //'sourceLanguage' => 'en-US',
                ],
            ],
        ],
    ]
];
```

此外，Yii2 将为类别创建与类别同名的消息文件。此行为可以通过在类别配置中指定`fileMap`属性来更改。除非使用`fileMap`属性另行指定，否则所有消息都将存储在`@app/messages/<language>/<category>.php`中。

## 默认翻译

Yii2 还允许我们为不匹配其他翻译的类别创建回退消息。这可以通过在配置文件中声明一个`*`类别来实现，如下面的示例所示：

```php
<?php return [
    // [...],
    'components' => [
        // [...],
        'i18n' => [
            'translations' => [
 '*' => [
 'class' => 'yii\i18n\PhpMessageSource'
 ],
            ],
        ],
    ]
];
```

## 框架消息

除了指定默认消息外，我们还可以修改 Yii2 原生提供的内置消息。默认情况下，Yii2 自带了诸如验证错误和其他基本字符串的几种翻译，所有这些都存储在`yii`类别中。由于默认的 Yii 消息可能不合适或不准确，您可以通过在配置文件中设置`yii`类别来重新定义默认消息：

```php
<?php return [
    // [...],
    'components' => [
        // [...],
        'i18n' => [
            'translations' => [
                'yii' => [
 'class' => 'yii\i18n\PhpMessageSource',
 'sourceLanguage' => 'en-US',
 'basePath' => '@app/messages'
 ],
            ],
        ],
    ]
];
```

## 处理缺失的翻译

如果源文件中缺少消息翻译，Yii2 将默认显示原始消息内容。虽然确保我们的网站至少显示一些内容是方便的，但这种行为可能会在调试和识别时造成麻烦。此外，我们可能希望在缺少翻译的情况下执行额外的处理。幸运的是，我们可以通过为`yii\i18n\MessageSource`触发的`missingTranslation`事件创建事件处理器来实现这一点，如下面的示例所示：

```php
<?php return [
    // [...],
    'components' => [
        // [...],
        'i18n' => [
            'translations' => [
                'app*' => [
                    'class' => 'yii\i18n\PhpMessageSource',
                'on missingTranslation' => ['app\components\TranslationEventHandler', 'handleMissingTranslation']
                ],
            ],
        ],
    ]
];
```

例如，我们可以编写一个事件处理器来输出一些值得注意的内容：

```php
<?php

namespace app\components;

use yii\i18n\MissingTranslationEvent;

class TranslationEventHandler
{
    public static function handleMissingTranslation(MissingTranslationEvent $event)
    {
        $event->translatedMessage = "@{$event->category}.{$event->message}-{$event->language}@";
    }
}
```

### 小贴士

事件处理器仅处理该类别中的消息。如果您希望处理多个类别的相同事件，必须将事件处理器分配给每个类别，或者，作为替代，将其分配给`*`类别。

## 生成消息文件

在配置我们的消息源之后，我们需要生成我们的消息文件。为此，我们将使用 `message` 命令：

1.  生成我们的消息文件的第一步是创建一个配置文件，该文件将定义我们想要支持的语言以及消息应存储的特定路径。这可以通过运行以下命令来完成：

    ```php
    ./yii message/config path/to/messagesconfig.php

    ```

1.  根据我们在 Web 或控制台配置文件中先前指定的语言，这将生成类似以下的内容：

    ```php
    <?php return [
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
     '/vendor,
     ],

        // 'php' output format is for saving messages to php files.
        'format' => 'php',

        // Root directory containing message translations.
        'messagePath' => __DIR__,

        // boolean, whether the message file should be overwritten with the merged messages
        'overwrite' => true
    ];
    ```

1.  在大多数情况下，Yii2 在此文件中提供的默认值应该足够。您应该考虑更改的唯一值是 `languages` 选项和 `format` 选项。在继续之前，请确保您已适当地设置了这些值。

1.  在对 `messagesconfig.php` 文件进行必要的更改后，我们可以通过直接运行消息命令来生成消息文件，如下例所示：

    ```php
    ./yii message path/to/messagesconfig.php

    ```

1.  `message` 命令是一个非常强大的工具，它允许我们快速生成可以交给翻译者的消息文件。配置文件中有几个选项可以使消息翻译更容易。例如，可以将 `removedUnused` 参数设置为 `true`，以自动从我们的消息文件中删除不再列在我们的源代码中的字符串。此外，通过将 `overwrite` 参数设置为 `true`，我们可以多次运行 `message` 命令来重新生成我们的翻译文件。

    ### 小贴士

    注意，`message` 命令不支持所有路径别名。当处理消息文件时，建议您使用绝对路径。此外，建议您将 `messagesconfig.php` 文件存储在应用程序的 `messages/` 目录中。

## 消息格式化

在翻译消息时，您可能希望将变量或数据从您的模型注入到消息中。为此，我们只需在我们的消息中嵌入一个 `placeholder`，然后在 `Yii::t()` 方法的第三个属性中定义 `placeholder` 的参数。例如，如果我们想使用用户的名字来问候用户，我们可以这样做：

```php
<?php 
// $model = User::find(1)->one();
echo Yii::t('app', 'Good Morning {name}', [
    'name' => $model->first_name
]);
```

作为命名参数的替代，我们还可以使用位置参数，如下例所示：

```php
$price = 500;
$count = 2;
$subtotal = 1000;
echo \Yii::t('app', 'Price: ${0}, Count: {1}, Subtotal: ${2}', [
    $price, 
    $count, 
    $subtotal
]);
```

### 小贴士

Yii2 还支持数字、货币、日期、时间、原始和复数数据的参数格式化。更多信息可以在 Yii2 API 的 [`www.yiiframework.com/doc-2.0/yii-i18n-formatter.html`](http://www.yiiframework.com/doc-2.0/yii-i18n-formatter.html) 以及 Yii2 指南的参数格式化部分 [`www.yiiframework.com/doc-2.0/guide-tutorial-i18n.html#parameter-formatting`](http://www.yiiframework.com/doc-2.0/guide-tutorial-i18n.html#parameter-formatting) 中找到。

# 查看文件翻译

作为翻译单个消息的替代方案，我们还可以通过在`views`文件夹的子目录中保存翻译的视图文件来翻译整个视图文件。例如，假设我们有一个位于`views/site/login.php`的视图脚本，我们可以通过在`views/site/es-MX/login.php`中放置一个翻译的消息文件来创建一个针对`es-MX`的西班牙语视图文件。假设我们的目标语言和源语言设置适当，当目标语言设置为`es-MX`时，Yii2 将自动渲染翻译的文件而不是基本文件。

### 小贴士

注意，如果源语言和目标语言相同，则无论是否存在翻译视图文件，都将渲染原始视图。

此外，视图文件翻译的使用并不遵循我们在整本书中强调的 DRY 模式。此外，将包含 PHP 代码的完整 HTML 文件交给翻译者可能会使这些文件的翻译变得困难，因为翻译行业基于字符串翻译，而不是代码中的字符串翻译。为了保持您的应用程序 DRY 并避免在翻译过程中可能出现的任何问题，强烈建议您使用之前提到的消息翻译方法，而不是视图文件翻译。

# 模块翻译

作为独立的实体，模块应包含它们自己的消息文件，而不是您的应用程序消息文件。在模块中使用消息的推荐方式如下：

1.  在模块的`init()`方法中，为模块定义一个新的翻译部分：

    ```php
    parent::init();Yii::$app->i18n->translations['modules/mymodule*'] = [
        'class' => 'yii\i18n\PhpMessageSource',
        'sourceLanguage' => 'en-US',
        'basePath' => '@app/modules/mymodule/messages'
    ];
    ```

1.  为`Yii::t()`创建一个静态方法包装器：

    ```php
    public static function t($category, $message, $params = [], $language = null)
    {
        return Yii::t('modules/mymodule/' . $category, $message, $params, $language);
    }
    ```

1.  最后，在模块的`messages/`目录中创建一个单独的消息配置文件，指定翻译器为`<ModuleName>::t`：

    ```php
    <?php return [
        'sourcePath' => __DIR__ . DIRECTORY_SEPARATOR . '..',
        'languages' => ['de'],
        'translator' => 'MyModule::t',
        'sort' => false,
        'removeUnused' => false,
        'only' => ['*.php'],
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
        'format' => 'php',
        'messagePath' => __DIR__,
        'overwrite' => true
    ];
    ```

我们模块中的消息可以通过调用`MyModule::t()`进行翻译。此外，可以通过运行以下命令生成翻译的消息文件：

```php
./yii message modules/mymodule/messages/messages.php

```

# 小部件翻译

类似地，小部件也可以通过遵循为模块概述的过程来拥有它们自己的消息翻译文件。使用我们在第五章中创建的`GreetingWidget`类，*模块、小部件和助手*将如下所示：

```php
<?php
namespace app\components;

use yii\base\Widget;
use yii\helpers\Html;

use Yii;
class GreetingWidget extends Widget
{
    public $name = null;

    public $greeting;

    public function init()
    {
        parent::init();

        Yii::$app->i18n->translations['widgets/GreetingWidget*'] = [
 'class' => 'yii\i18n\PhpMessageSource',
 'sourceLanguage' => 'en-US',
 'basePath' => '@app/components/widgets/GreetingWidget'
 ];

        $hour = date('G');

        if ( $hour >= 5 && $hour <= 11 )
           $this->greeting = GreetingWidget::t("Good Morning");
        else if ( $hour >= 12 && $hour <= 18 )
           $this->greeting = GreetingWidget::t("Good Afternoon");
        else if ( $hour >= 19 || $hours <= 4 )
           $this->greeting = GreetingWidget::t("Good Evening");
    }

    public function run()
    {
        if ($this->name === null)
            return HTML::encode($this->greeting);
        else
            return HTML::encode($this->greeting . ', ' . $this->name);
    }

    public static function t($category, $message, $params = [], $language = null)
 {
 return Yii::t('widgets/GreetingWidget/' . $category, $message, $params, $language);
 }
} 
```

因此，调用`GreetingWidget::t()`将渲染针对我们小部件的特定翻译消息。此外，由于小部件支持视图渲染，它们还可以通过遵循之前概述的过程来支持完全翻译的视图文件。

# 摘要

Yii2 为我们应用提供了强大的工具来支持国际化与本地化。在本章中，我们介绍了如何生成和存储消息源文件，如何生成消息和视图翻译，以及如何在模块和小部件中支持翻译。在下一章中，我们将介绍 Yii2 的性能特性，以及探索 Yii2 提供的几个内置安全特性。
