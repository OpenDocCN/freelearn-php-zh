# 第六章 资产管理

现代 Web 应用程序由许多不同的组件组成。除了功能之外，我们应用程序的展示可能被认为是其最重要的方面。用户界面的展示和相应的用户体验对于构建优秀的 Web 应用程序至关重要。在 Web 应用程序中，展示和体验通常由**层叠样式表**（**CSS**）和 JavaScript 文件定义。使用原始 HTML，我们可以包含所需的任何脚本和样式，但通常我们需要以编程方式处理我们的资产（例如，当使用模块、组件或小部件时）。为了帮助管理我们的资产，我们可以使用第三方工具和 Yii2 内置的资产管理器的组合。在本章中，我们将介绍如何使用 Yii2 的资产管理工具，以及介绍我们可以使用的几个第三方工具，以简化资产文件的管理。

# 资产包

Yii2 中通过资产包管理资源。在 Yii2 中，资产包简单来说是一个声明我们希望在应用程序中使用的所有资源的类，它位于我们应用程序的`assets/`目录中，通常位于声明`AppAsset`类的`AppAsset.php`文件中，该类扩展了`yii\web\AssetBundle`。由于我们的默认应用程序包含一个预定义的`AppAsset`类，让我们看看该文件中已经定义了什么。

```php
<?php

namespace app\assets;
use yii\web\AssetBundle;

class AppAsset extends AssetBundle
{
    public $basePath = '@webroot';
    public $baseUrl = '@web';

    public $css = [
        'css/site.css',
        '//ajax.googleapis.com/ajax/libs/jquery/2.1.1/jquery.min.js'
    ];

    public $js = [ ];

    public $depends = [
        'yii\web\YiiAsset',
        'yii\bootstrap\BootstrapAsset',
    ];
}
```

我们的示例资产包文件声明了几个公共属性。第一个属性是应用程序的基路径和基 URL，它们定义了我们的资产应该从哪里加载。第二个属性是 CSS 和 JavaScript 文件的数组，它们定义了应该与我们的资产包注册哪些资产。最后，我们的资产包定义了当前资产包依赖于哪些资产包。以下是最常见属性的详细说明：

| 属性 | 说明 |
| --- | --- |
| `basePath` | 包含资产文件的 Web 服务器公共目录的字符串或路径别名。 |
| `baseUrl` | JS 或 CSS 属性中列出的相对资源的基 URL。 |
| `css` | 要包含在资产包中的 CSS 文件数组。 |
| `cssOptions` | 将与生成的`<link>`标签一起渲染的选项和条件数组。 |
| `depends` | 依赖于此资产包的资产包数组。 |
| `js` | 要包含在资产包中的 JavaScript 文件数组。 |
| `jsOptions` | 将与生成的`<script>`标签一起渲染的选项和条件数组。 |
| `publishOptions` | 要传递给`yii\web\AssetManager`的`publish()`方法的选项。 |
| `sourcePath` | 定义包含我们想要包含在包中的资产文件的目录。设置此属性将覆盖`basePath`和`baseUrl`。 |

## 使用资产包

在定义了我们的资源包之后，我们还需要将它们包含到我们的布局文件中。我们可以通过在主布局文件的开头添加以下内容来实现（在我们的例子中这是`views/layouts/main.php`）。

```php
<?php
use app\assets\AppAsset;
AppAsset::register($this);
```

在页面加载时，我们的资源包将注册所有依赖的资源包，并将所有非 Web 可访问的文件发布到 Web 可访问的目录。然后在视图渲染阶段，它将生成所有必要的 HTML 标记以包含在我们的视图中。

### 小贴士

在前面的例子中，`$this`是`yii\web\View`的一个实例。当在组件或小部件中工作时，您可以通过使用`$this->view`在组件或小部件中检索视图对象。

## 配置

内部，Yii2 通过`assetManager`应用程序组件来管理资源包及其配置，该组件由`yii\web\AssetManager`类实现。通过配置此组件的`$bundles`属性，我们可以自定义资源包的行为。以`yii\web\JQueryAsset`资源包为例；默认情况下，它提供从**Bower**（我们将在本章后面讨论的第三方资源依赖管理器）提供的 jQuery 版本，当我们的 Yii2 项目安装时。如果我们想使用这个资源包的不同版本的 jQuery，或者想通过使用第三方 CDN 来提高性能，我们可以像以下这样覆盖 jQuery 资源包选项。

```php
// config/web.php
return [
    // [...],
    'components' => [
        // [...],
        'assetManager' => [
            'bundles' => [
                'yii\web\JqueryAsset' => [
                    // Prevents the asset bundle from publishing this file
                    'sourcePath' => null,
                    'js' => [
                        'https://cdnjs.cloudflare.com/ajax/libs/jquery/2.1.4/jquery.js',
                    ]
                ],
            ],
        ],
    ],
];
```

在这个例子中，我们通过将`js`参数设置为 CloudFlare CDN，并告诉我们的`JQueryAsset`资源包不要将资源作为从第三方 CDN 渲染的内容来重新定义资源包的 JavaScript 文件。

或者，我们也可以有条件地重新定义正在渲染的文件，比如在我们有一个想要在生产环境中显示的脚本压缩版本，但在其他环境中我们希望使用非压缩版本的情况下。

```php
// config/web.php
return [
    // [...],
    'components' => [
        // [...],
        'assetManager' => [
            'bundles' => [
                'yii\web\JqueryAsset' => [
                    'js' => [
                        APPLICATION_ENV == 'prod' ? 
'jquery.min.js' : 'jquery.js'
                    ]
                ],
            ],
        ],
    ],
];
```

### 小贴士

作为提醒，我们的`APPLICATION_ENV`常量依赖于我们在第一章中建立的多环境设置，*Composer, 配置, 类和路径别名*。

此外，我们可以通过将特定的资源包设置为`false`来禁用特定的资源包，如下面的示例所示。

```php
// config/web.php
return [
    // [...],
    'components' => [
        // [...],
        'assetManager' => [
            'bundles' => [
                'yii\web\BootstrapAsset' => false
            ],
        ],
    ],
];
```

此外，我们可以通过将`bundles`属性设置为`false`来完全禁用应用程序中包含的所有资源包。

```php
// config/web.php
return [
    // [...],
    'components' => [
        // [...],
        'assetManager' => [
            'bundles' => false
        ],
    ],
];
```

### 资源映射

在某些情况下，多个资源包可能定义了同一脚本的不同版本。例如，一个资源包可能包含 jQuery 版本 2.1.3，另一个可能定义 2.1.4。为了解决这些冲突，我们可以将配置文件的`assetMap`属性设置为将任何命名的资源文件实例解析为单个依赖项，该依赖项将被包含在我们的视图中。

```php
// config/web.php
return [
    // [...],
    'components' => [
        // [...],
        'assetManager' => [
            'assetMap' => [
                'jquery.js' => 'https://cdnjs.cloudflare.com/ajax/libs/jquery/2.1.4/jquery.js',
                'jquery.min.js' => 'https://cdnjs.cloudflare.com/ajax/libs/jquery/2.1.4/jquery.min.js'
            ]
        ],
    ],
];
```

在这种情况下，任何在资产包的`js`部分定义了`jquery.js`和`jquery.min.js`实例的资产包，都将将该资产重新映射到我们的 CloudFlare CDN 资产。

### 小贴士

`assetMap`属性将资产文件在包中的最后一部分作为键值对进行匹配。

## 资产类型和位置

根据它们的位置，Yii2 将资产分为三种不同的方式。资产将被分类为源资产、发布资产或外部资产。源资产是混合在我们源代码中的资产文件，并且不在可访问的 Web 目录中。这类资产通常包含在模块、小部件、扩展或组件中。任何 Yii2 定义为源资产的资产都需要由 Yii2 发布到可访问的 Web 目录。发布资产是已发布到可访问的 Web 目录的源资产。最后，外部资产是位于可访问位置上的资产，例如在我们的当前服务器上、另一个服务器或 CDN 上。与发布资产不同，Yii2 不会将这些资产发布到我们的资产目录，而是直接将它们作为外部资源引用。

当与资产包一起工作时，如果指定了`sourcePath`属性，Yii2 会将任何带有相对路径列出的资产视为源资产，并在运行时尝试发布这些资产。如果没有指定`sourcePath`属性，Yii2 将假设列出的资产位于一个可访问的 Web 目录中，并且已发布。在这种情况下，有必要指定`basePath`属性或`baseUrl`属性，以告诉 Yii2 资产所在的位置。

### 小贴士

不要使用`@webroot/assets`别名作为`sourcePath`属性，因为这个目录由资产管理器用于保存从其源位置发布的资产文件。存储在这个目录中的任何数据都可能随时被 Yii2 删除。

## 资产选项

与`yii\web\View`方法`registerJsFile()`和`registerCssFile()`一样，我们可以通过设置资产包的相应`$jsOptions`和`$cssOptions`属性，以一组给定的选项来渲染资产包。

例如，我们可以在视图中的`<body>`标签末尾包含我们的列出的 JavaScript 文件，使我们的资产包包含在内。

```php
public $jsOptions = ['position' => \yii\web\View::POS_END];
```

### 小贴士

`yii\web\View`类还提供了定位方法，用于身体的开头（`yii\web\View::POS_BEGIN`），身体的末尾（`yii\web\View::POS_END`），在`jQuery(window).load()`事件中（`yii\web\View::POS_LOAD`），以及在`jQuery(window).ready()`事件中（`yii\web\View::POS_READY`）。

在 CSS 中，我们还可以定义如下`<noscript>`块：

```php
public $cssOptions = ['noscript' => true];
```

此外，我们还可以在条件语句中包裹我们的 CSS 块：

```php
public $cssOptions = ['condition' => 'IE 11'];
```

这将导致以下 HTML 被渲染：

```php
<!--[if IE11]>
<link rel="stylesheet" href="path/to/ie11.css">
<![endif]-->
```

### 小贴士

设置 `$jsOptions` 或 `$cssOptions` 属性将把指定的选项应用到资产包中定义的所有 CSS 和 JavaScript 文件上。为了对每个资产应用不同的条件，您需要创建一个包含这些条件的单独资产包，或者使用 `theregisterCssFile()` 或 `registerJsFile()` 方法在视图中内联资产。

## 资产发布

如前所述，如果资产包引用的资产位于一个从网络浏览器无法公开访问的目录中（或者设置了 `sourcePath` 属性），其资产将作为资产管理器在将包注册到视图时执行的自动发布过程的一部分被复制到 `@webroot/assets`（对应于 `@web/assets` 的网络路径）。如前所述，可以通过设置资产包的 `baseUrl` 和 `basePath` 属性来更改发布路径。

如您所预期的那样，在 Web 请求上复制文件的过程可能相当昂贵，如果允许其持续运行，可能会在生产环境中引起与性能相关的问题。为了帮助减轻这个问题，Yii2 提供了两种替代方案。

而不是复制文件，Yii 的资产管理器可以通过设置 `assetManager` 的 `linkAssets` 属性来配置在原始资产文件和可访问的网页目录之间创建一个符号链接，如下所示：

```php
// config/web.php
return [
    // [...], 
    'components' => [
        'assetManager' => [
            'linkAssets' => true,
        ],
    ],
    // [...], 
];
```

### 小贴士

发布过程通常只发生一次。一旦 Yii2 发布了我们的资产，它就不会再次发布，除非我们删除我们的资产目录或告诉 Yii2 重新发布我们的资产。

默认情况下，Yii2 将在 `sourcePath` 属性中列出的每个文件上运行发布过程，这意味着如果您有一个大目录，那么无论文件是否实际使用，每个文件都会被复制。为了使 Yii2 的资产管理器只复制您需要的文件，您可以修改资产包的 `publishOptions` 属性。

以我们使用雅虎流行的 CSS 库 `purecss` 为例。要从源代码构建 `purecss`，我们需要运行 Bower、NPM 和 Grunt，这将留下我们不希望发布到我们的网页目录中的构建文件。

通过设置如以下示例所示的 `publishOptions` 属性，我们可以确保只发布构建文件，这可以在初始发布期间显著提高性能。

```php
<?php
namespace app\assets;

use yii\web\AssetBundle;

class PureCssAsset extends AssetBundle 
{
    public $sourcePath = '@bower/purecss'; 

    public $css = [ 
        'build/base-min.css', 
    ];

    public $publishOptions = [
        'only' => [
        	'build/'
        ]
    ];
}
```

## 资产包的客户端缓存管理

在生产环境中运行应用程序时，我们通常会在我们的 JavaScript 和 CSS 资产上设置长期缓存过期日期，以提高性能。当推送新代码时，我们的资产通常会发生变化，但它们的文件位置不会变，这将在我们进行更改时阻止客户端接收我们的更新资产。克服这个问题的最简单方法是在我们的资产末尾附加一个版本号或时间戳，这样浏览器就可以缓存我们资产的特定版本，并且在我们推送新资产时能够重新缓存。

使用 Yii2，我们可以通过设置 `assetManager` 的 `appendTimestamp` 属性来配置资产管理器，使其自动将最后修改时间戳附加到我们的资产上，如下所示：

```php
// config/web.php
return [
    // [...],
    'components' => [
        'assetManager' => [
            'appendTimestamp' => true,
        ],
    ],
    // [...],
];
```

## 使用资产包的预处理器

为了使资产开发更加简单且易于管理，许多开发者已经转向使用扩展语法语言，如 LESS 和 CoffeeScript，并依赖它们相应的工具将这些资产转换为 CSS 和 JavaScript 文件。Yii2 可以通过启用资产管理器来帮助简化此过程，使其为您处理构建过程。使用 Yii2 的资产包，您可以直接在资产包中列出 LESS、SCSS、Stylus、CoffeeScript 和 TypeScript 文件，Yii2 将识别它们，并自动通过相应的预处理器运行它们。以下是一个资产包的示例：

```php
<?php
namespace app\assets;

use yii\web\AssetBundle;

class AppAsset extends AssetBundle
{
    public $basePath = '@webroot';
    public $baseUrl = '@web';
    public $css = [
        'css/app.less',
    ];
    public $js = [
        'js/app.ts'
    ];
    public $depends = [
        'yii\web\YiiAsset'
    ];
}
```

当我们的资产包注册到我们的视图中时，Yii2 将自动运行适当的预处理器工具，将资产转换为 CSS 和 JavaScript 以包含在我们的视图中。

### 注意

Yii2 依赖于您的计算机上安装的相应预处理器软件才能使此功能正常工作。

当与预处理器一起工作时，可能需要为您的资产指定额外的参数，以确保它们能够正确生成。在 Yii2 中，您可以通过以下方式设置 `assetManager` 实例的 `converter` 属性来实现这一点。

```php
// config/web.php
return [
    // [...],
    'components' => [
        'assetManager' => [
            'converter' => [
                'class' => 'yii\web\AssetConverter',
                'commands' => [
                    'less' => ['css', 'lessc {from} {to}'],
                    'ts' => ['js', 'tsc --out {to} {from}'],
                ],
            ],
        ],
    ],
    // [...],
];
```

### 小贴士

虽然使用起来很方便，但在生产环境中让 Yii2 构建我们的资产文件通常不是一个好主意，因为它会将不必要的软件引入生产环境，这些软件可能与您的开发环境不匹配或存在安全漏洞，并且可能会严重阻碍应用程序的性能，因为 Yii2 需要在其首次运行时构建资产文件。在生产环境中工作，通常在将应用程序推送到生产之前，在构建服务器上构建所有资产文件是一个更好的选择。我们将在本章后面介绍如何使用 Grunt、NodeJS 和 Bower 构建资产文件，并在第十三章调试和部署中介绍一些基本的部署策略。

## 资产命令行工具

对于 HTTP/1.1 应用程序，为了节省带宽和请求，通常最好将多个资产文件合并并压缩在一起。Yii2 可以通过 `asset` 命令帮助简化此过程，该命令可以帮助您使用 Yii2 以及一些第三方 Java 工具来压缩和合并您的资产文件。

### 小贴士

由于 HTTP/2 协议的变化，通常单独提供资产文件比合并它们更有益。随着更多像 Nginx 和 Apache 这样的网络服务器开始支持 HTTP/2 协议，您应该进行自己的实验，以确定对于您的应用程序来说，合并资产文件是否是最佳选择。

`asset`命令行工具提供了两个选项`asset/template`，该选项用于生成一个名为`asset.php`的指令文件，供第二个命令`asset/compress`使用，该命令用于压缩文件。第一个工具`asset/template`的调用方式如下：

```php
./yii asset/template config/assets.php

```

运行此命令后，将在我们应用程序的`config`目录中生成一个名为`assets.php`的文件，默认输出如下。

```php
<?php
/**
 * Configuration file for the "yii asset" console command.
 */

// In the console environment, some path aliases may not exist. Please define these:
// Yii::setAlias('@webroot', __DIR__ . '/../web');
// Yii::setAlias('@web', '/');

return [
    // Adjust command/callback for JavaScript files compressing:
    'jsCompressor' => 'java -jar compiler.jar --js {from} --js_output_file {to}',
    // Adjust command/callback for CSS files compressing:
    'cssCompressor' => 'java -jar yuicompressor.jar --type css {from} -o {to}',
    // The list of asset bundles to compress:
    'bundles' => [
        // 'app\assets\AppAsset',
        // 'yii\web\YiiAsset',
        // 'yii\web\JqueryAsset',
    ],

    // Asset bundle for compression output:
    'targets' => [
        'all' => [
            'class' => 'yii\web\AssetBundle',
            'basePath' => '@webroot/assets',
            'baseUrl' => '@web/assets',
            'js' => 'js/all-{hash}.js',
            'css' => 'css/all-{hash}.css',
        ],
    ],

    // Asset manager configuration:
    'assetManager' => [
        //'basePath' => '@webroot/assets',
        //'baseUrl' => '@web/assets',
    ],  
];
```

### 小贴士

为了压缩资产，Yii2 默认会尝试使用 Closure Compiler（[`developers.google.com/closure/compiler/`](https://developers.google.com/closure/compiler/)）和 YUI Compressor（[`github.com/yui/yuicompressor/`](https://github.com/yui/yuicompressor/)）。您需要安装这两个工具，以便`asset`命令按预期工作。

此配置文件定义了几个不同的选项。前两个选项`jsCompressor`和`cssCompressor`定义了压缩 JavaScript 和 CSS 文件应运行的命令。默认情况下，这些工具将尝试使用 Closure Compile 和 YUI Compressor；如果您希望使用其他工具，这两个工具都可以按需进行配置。

第二个选项`bundles`定义了您希望一起压缩的资产包。第三个选项`assetManager`定义了资产管理器组件应使用的一些基本选项，例如压缩资产的`basePath`和`baseUrl`。最后，`targets`选项定义了将生成的输出资产包。默认情况下，Yii2 将创建一个名为`all`的目标，并为列出的所有资产包生成压缩资产。

在许多情况下，我们通常会将资产分散在几个不同的资产包中，例如共享的、前端的和后端的工具。由于前端资产不需要与我们的后端资产一起包含，我们可以定义多个目标，压缩后生成单独的资产，这样我们就可以专门包含这些资产，从而为我们的最终用户节省带宽。以下是一个示例：

```php
<?php
/**
 * Configuration file for the "yii asset" console command.
 */

// In the console environment, some path aliases may not exist. Please define these:
// Yii::setAlias('@webroot', __DIR__ . '/../web');
// Yii::setAlias('@web', '/');

return [
    // [...],
    'targets' => [
        'shared' => [
            'js' => 'js/shared-{hash}.js',
            'css' => 'css/shared-{hash}.css',
            'depends' => [
                'yii\web\YiiAsset',
                'app\assets\AppAsset',
            ],
        ],
        'backend' => [
            'js' => 'js/backend-{hash}.js',
            'css' => 'css/backend-{hash}.css',
            'depends' => [
                'yii\web\YiiAsset',
                'app\assets\AdminAsset'
            ],
        ],
        'frontend' => [
            'js' => 'js/frontend-{hash}.js',
            'css' => 'css/frontend-{hash}.css',
            'depends' => [], 
        ],
    ] 
];
```

在编写我们的资产配置文件后，我们可以通过运行资产命令来生成我们的压缩资产文件，如下所示：

```php
./yii asset/compress config/asset.php

```

### 小贴士

资产配置文件作为便利性提供，以便尽可能地将所有内容保持在 Yii2 中。虽然 Closure Compiler 和 YUI Compressor 是很好的工具，但像 Grunt 和 NodeJS 这样的工具通常可以提供更易于使用和开发的解决方案，同时消除在 Yii2 中编译和压缩资产所需的大部分配置。在处理资产时，务必找到最适合您的开发工作流程、团队和构建过程的工具。

# 第三方资产工具

当处理现代 Web 应用程序时，我们经常需要从各种来源包含许多不同类型的资产。直接将这些资产包含在我们的应用程序中可能会导致几个问题，具体如下：

+   第三方资产的许可

+   版本和安全管理

+   仓库大小

+   构建过程

而不是直接在我们的应用程序中包含资产，我们可以利用第三方资产管理系统，如 NodeJS 和 Bower，这可以缓解之前概述的所有问题。

使用 Yii2，我们可以直接使用 Node 和 Bower 包。对于简单的应用程序，我们可以在 `composer.json` 文件中直接包含这些包，通过在 `require` 部分包含 `bower-asset/PackageName` 和 `npm-asset/PackageName`。Yii2 的后置脚本将自动处理将这些资产包含在 `@bower` 文件夹和 `@npm` 文件夹中，然后我们可以在我们的资产包中引用它们。在一个典型的 Yii2 实例中，这分别对应于 `vendor/bower` 和 `vendor/npm`。

对于更复杂的项目，直接在我们的应用程序中利用这些第三方工具可能更有意义，稍后包含必要的 CSS 和 JavaScript 文件。在下一节中，我们将探讨三个工具：NodeJS、Bower 和 Grunt，并探讨我们如何与 Yii2 结合使用它们。

## NodeJS

我们经常用来管理我们的资产的第一和最重要的工具被称为 **NodeJS**，这是一个我们可以用来安装其他两个包（Bower 和 Grunt）的工具。要开始使用 NodeJS，我们首先需要从 [`nodejs.org/download/`](https://nodejs.org/download/) 下载软件并在我们的系统上安装它。

对于我们的目的，NodeJS 将为我们提供自动下载和构建我们的资产文件所需的工具和包。要开始使用 NodeJS，我们首先需要在我们的应用程序中包含一个 `package.json` 文件。此文件将定义我们想要使用的所有依赖项。一个典型的用于资产管理 NodeJS 文件将如下所示：

```php
{
  "name": "masteringyii-ch6",
  "description": "Chapter 6 source code for the book 'Mastering Yii'",
  "repository": {
    "type": "git",
    "url": "https://www.github.com/masteringyii/chapter6"
  },
  "dependencies": {
    "ansi-styles": "¹.1.0",
    "bower": "1.3.12",
    "grunt": "⁰.4.4",
    "grunt-cli": "⁰.1.13",
    "grunt-contrib-concat": "⁰.4.0",
    "grunt-contrib-cssmin": "0.6.1",
    "grunt-contrib-uglify": "0.2.0"
  }
}
```

### 小贴士

在 NodeJS 中与其他包（如 Bower 和 Grunt）一起工作的有两种不同的方式。第一种方式是在我们的 `package.json` 文件中将它们作为依赖项包含。这样做的好处是我们可以将构建工具的版本锁定到我们的应用程序中。或者，我们可以全局安装这些工具，这样我们就可以直接通过命令行运行它们。当与许多开发者和团队一起工作时，通常最好使用 `package.json` 文件中定义的工具。

在我们的 `package.json` 文件中，我们定义了一些关于我们的存储库的详细信息，例如名称、描述和存储库详细信息，以及我们想要使用的几个工具，例如 Bower、Grunt 和一些用于连接和压缩 CSS 和 JavaScript 文件的 Grunt 工具。

在我们的 NodeJS 配置文件设置完成后；我们现在可以使用 NodeJS 通过运行以下命令将这些工具添加到我们的存储库中：

```php
npm install

```

这将安装我们的构建工具到 `node_modules` 目录。

### 小贴士

由于此目录包含构建工具，我们应该通过将其添加到我们的 `.gitignore` 文件中来排除它。

## Bower

要管理 CSS 和 JavaScript 库，我们可以利用一个名为 Bower 的资产依赖管理工具。要开始使用 Bower，我们首先需要在应用程序的根目录中创建一个 `bower.json` 文件，并填充我们想要包含的库。例如，让我们在我们的应用程序中包含流行的 CSS 库 PureCSS。我们可以通过编写以下基本的 `bower.json` 文件来实现这一点：

```php
{
  "name": "masteringyii-ch6",
  "dependencies": {
    "pure": "~0.6.0"
  }
}
```

### 提示

包名称的完整列表可以在 [`bower.io/`](http://bower.io/) 查找。

要安装这些包，然后我们可以从我们的 `node_modules` 目录运行 Bower，如下所示：

```php
./node_modules/.bin/bower install

```

这将把我们的库和 CSS 添加到应用程序根目录下的 `vendor/bower` 目录。

### 提示

默认情况下，Bower 会将其自身安装到 `bower_components` 目录。然而，由于 Yii2 已经定义了安装目录，它被重新映射到 `vendor/bower`。

## Grunt

由于我们已经知道如何使用 YUI Compressor 和 Closure Compiler 以及 Yii2 的 `asset` 命令，此时我们有一个选项是将我们的资产包和资产配置文件直接指向 `node_modules` 和 `bower_components` 目录。虽然这消除了之前列出的许多问题，但我们还可以使用另一个名为 Grunt 的第三方工具来处理压缩和合并我们的文件。

简而言之，Grunt 是一个 JavaScript 任务运行器，旨在帮助自动化许多需要重复执行的任务，例如构建资产文件。使用像 Grunt 这样的工具的主要好处是，您可以自动化开发和构建服务器的整个工作流程。

要开始使用 Grunt，我们首先需要创建一个名为 `Gruntfile.js` 的文件，它将包含我们应用程序的所有构建指令。

1.  创建我们的 `Gruntfile.js` 文件的第一个步骤是声明我们正在使用 Grunt，并指定我们想要使用的 Grunt 模块（这些名称我们在 `package.json` 文件中指定）。

    ```php
    module.exports = function(grunt) {

        // Register the NPM tasks we want
        grunt.loadNpmTasks('grunt-contrib-concat');
        grunt.loadNpmTasks('grunt-contrib-cssmin');
        grunt.loadNpmTasks('grunt-contrib-uglify');

    };
    ```

1.  在本节中，我们将通过指定在运行 Grunt 时想要运行的任务来声明我们的默认任务。在我们的例子中，我们想要连接我们的 JavaScript 和 CSS 文件，然后最小化我们的 JavaScript 和 CSS 文件。

    ```php
    // Register the tasks we want to run
    grunt.registerTask('default', [
        'concat',
        'cssmin:css',
        'uglify:js'
    ]);
    ```

1.  然后，我们开始配置我们的 Grunt 任务，告诉 Grunt 它可以在哪里找到我们的 `package.json` 文件，并设置一些基本的路径别名。

    ```php
    grunt.initConfig({
        pkg: grunt.file.readJSON('package.json'),

        paths: {
            assets: 'web',
            bower: 'vendor/bower',
            css : '<%= paths.assets %>/css',
            js: '<%= paths.assets %>/js',
            dist: '<%= paths.assets %>/dist',
        },
    }
    ```

1.  在本节中，我们定义我们的任务以连接我们的 JavaScript 和 CSS 文件。

    ```php
    concat: {
        css: {
            src: [
                '<%= paths.bower %>/pure/pure-min.css',
                '<%= paths.css %>/*'
            ],
            dest: '<%= paths.dist %>/app.css'
        },
        js : {
            src: [
                '<%= paths.js %>/*.js'
            ],
            dest: '<%= paths.dist %>/app.js'
        }
    },
    ```

1.  我们的任务是在连接我们的 CSS 资产后最小化它们。

    ```php
    cssmin : {
        css:{
            src: '<%= paths.dist %>/app.css',
            dest: '<%= paths.dist %>/app.min.css'
        }
    },
    ```

1.  最后，压缩我们的 JavaScript 文件的任务。

    ```php
    uglify: {
        js: {
            files: {
                '<%= paths.dist %>/app.min.js' : ['<%= paths.dist %>/app.js']
            }
        }
    },
    ```

在我们的 `Gruntfile.js` 文件配置完成后，我们可以通过以下方式运行 Grunt 来构建我们的资产文件：

```php
./node_modules/.bin/grunt

```

如果一切顺利，我们应该看到以下输出：

```php
Running "concat:css" (concat) task
File web/dist/app.css created.

Running "concat:js" (concat) task
File web/dist/app.js created.

Running "cssmin:css" (cssmin) task
File web/dist/app.min.css created.

Running "uglify:js" (uglify) task
File "web/dist/app.min.js" created.

Done, without errors.

```

如 Grunt 输出所示，我们为我们生成了四个文件，包括压缩和解压缩的 JavaScript 和 CSS 文件，这些文件包含了我们想要包含在我们网站中的所有资源。从这一点出发，我们就可以有条件地将我们的资源文件包含在我们的资源包中，并切换掉我们的`APPLICATION_ENV`或`YII_ENV_<ENV>`环境，以便在生产环境中使用压缩版本，在非生产环境中使用非压缩版本。

### 小贴士

NodeJS、Bower 和 Grunt 各自提供了强大的工具来自动完成某些任务，并且与 Yii2 配合良好。然而，在决定使用特定技术之前，务必咨询您的团队，以确定对他们来说什么是最有效的。

# 摘要

在本章中，我们介绍了资源在 Yii2 中的工作方式和管理工作。我们探讨了资源包文件的基本知识及其与 Yii2 资源管理器的集成。我们还探讨了如何使用`asset`命令来构建配置文件，以及如何组合和压缩我们的资源。最后，我们探讨了三个第三方工具：NodeJS、Bower 和 Grunt，并说明了如何结合我们的资源包使用这些工具来自动化我们的资源文件构建。

在探索了 Yii 的前端方面之后，在下一章中，我们将回到后端，学习我们如何在应用程序中处理用户认证和授权，以及如何在我们的应用程序中设置访问控制过滤器以及基于规则的认证。
