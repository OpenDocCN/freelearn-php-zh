# 第十二章。性能和安全

默认情况下，Yii2 既是高性能的也是高效的 PHP 框架。它被设计得尽可能快，同时仍然提供功能丰富的工具箱以供使用。有许多因素会影响我们应用程序的性能，这些因素可能会对我们的应用程序性能产生负面影响，例如长时间运行的查询和数据生成。在本章中，我们将介绍几种优化和微调 Yii2 的方法，以确保我们的应用程序保持高性能。我们还将介绍保护我们代码的几个重要方面。

# 缓存

提高我们应用程序性能的最简单方法之一是实现缓存。通过在我们的应用程序中实现缓存，我们可以减少生成和向最终用户交付数据所需的时间。使用 Yii2，我们可以缓存从生成数据、数据库查询到整个页面和页面片段的一切。我们还可以指示我们的浏览器为我们缓存页面。在本节中，我们将介绍几种不同的缓存技术，我们可以在 Yii2 中实现这些技术，以提高我们应用程序的性能。

## 缓存数据

数据缓存主要是关于存储常见生成数据，以便我们可以为给定时间段生成一次，而不是每次请求都生成，在 Yii2 中，它是通过应用程序的缓存组件实现的。Yii2 提供了多种不同的类，我们可以使用这些类来缓存数据，所有这些类都遵循并使用一致的 API，通过实现 `yii\caching\Cache` 抽象类。

这个一致的 API 允许我们无需修改应用程序内部的代码，就可以用以下表格中列出的任何缓存替换我们的缓存组件：

| 缓存名称 | 描述 | 类引用 |
| --- | --- | --- |
| `yii\caching\ApcCache` | 使用 APC PHP 扩展的缓存。在单个服务器配置中，APC 缓存性能非常好，但如果启用了 PHP Opcache，则存在兼容性问题。 | [`www.yiiframework.com/doc-2.0/yii-caching-apccache.html`](http://www.yiiframework.com/doc-2.0/yii-caching-apccache.html) |
| `yii\caching\DbCache` | 使用数据库表来存储信息的缓存。 | [`www.yiiframework.com/doc-2.0/yii-caching-dbcache.html`](http://www.yiiframework.com/doc-2.0/yii-caching-dbcache.html) |
| `yii\caching\DummyCache` | 一个占位符缓存，不执行任何缓存操作，但在开发期间用作真实缓存的代表，以确保我们的应用程序能够与真实缓存一起工作。 | [`www.yiiframework.com/doc-2.0/yii-caching-dummycache.html`](http://www.yiiframework.com/doc-2.0/yii-caching-dummycache.html) |
| `yii\caching\FileCache` | 一种将数据存储在文件存储中的缓存，适用于存储页面或页面片段。 | [`www.yiiframework.com/doc-2.0/yii-caching-filecache.html`](http://www.yiiframework.com/doc-2.0/yii-caching-filecache.html) |
| `yii\caching\MemCache` | 使用 PHP memcache 或 memcached 扩展存储数据的内存缓存。 | [`www.yiiframework.com/doc-2.0/yii-caching-memcache.html`](http://www.yiiframework.com/doc-2.0/yii-caching-memcache.html) |
| `yii\caching\WinCache` | 使用 WinCache PHP 扩展的缓存。 | [`www.yiiframework.com/doc-2.0/yii-caching-wincache.html`](http://www.yiiframework.com/doc-2.0/yii-caching-wincache.html) |
| `yii\redis\Cache` | 实现 Redis 键值存储的缓存。 | [`www.yiiframework.com/doc-2.0/yii-redis-cache.html`](http://www.yiiframework.com/doc-2.0/yii-redis-cache.html) |
| `yii\caching\XCache` | 使用 XCache PHP 扩展的缓存。 | [`www.yiiframework.com/doc-2.0/yii-caching-xcache.html`](http://www.yiiframework.com/doc-2.0/yii-caching-xcache.html) |

### 小贴士

虽然列出的每个缓存都实现了 `yii\caching\Cache` API，但一些缓存，例如 `yii\redis\Cache` 和 `yii\caching\MemCache`，需要一些额外的配置。请确保您参考您在应用程序中决定使用的缓存类的参考文档。

以 `yii\caching\FileCache` 为例，我们可以在应用程序配置文件中添加以下内容来实现应用程序内的缓存：

```php
<?php return [
    // [...],
    'components' => [
        // [...]
        'cache' => [
 'class' => 'yii\caching\FileCache',
 ]
    ]
];
```

在实现特定的缓存系统之后，我们可以在应用程序代码中通过引用 `Yii::$app->cache` 来使用我们的缓存。

如前所述，每个缓存都实现了由 `yii\caching\Cache` 抽象类定义的一致 API。因此，每个缓存都提供了以下方法，我们可以使用这些方法来操作缓存中的数据。

| 方法 | 说明 |
| --- | --- |
| `yii\caching\Cache::add()` | 如果缓存中不存在，则使用给定键存储值。如果缓存的项已存在，则不会执行任何操作。 |
| `yii\caching\Cache::get()` | 从缓存中检索具有给定键的项。 |
| `yii\caching\Cache::set()` | 将具有给定键的项设置到缓存中，并可选择指定过期日期。设置了过期日期的缓存项将自动由底层缓存机制或 Yii2 本身清除。 |
| `yii\caching\Cache::madd()` | 将多个项作为键值数组存储到缓存中。如果给定的缓存键已存在，则不会发生任何操作。在 Yii 2.1 中，此方法将被标记为已弃用，并由 `yii\caching\Cache::multiAdd()` 取代。 |
| `yii\caching\Cache::mget()` | 同时从缓存中检索多个数据键。在 Yii 2.1 中，此方法将被标记为已弃用，并由 `yii\caching\Cache::multiGet()` 取代。 |
| `yii\caching\Cache::mset()` | 将多个缓存项（以键值对的形式）同时设置到缓存中。带有过期时间的缓存项将被底层缓存机制或 Yii2 自动清除。在 Yii 2.1 中，此方法将被标记为弃用，并由`yii\caching\Cache::multiSet()`方法取代。 |
| `yii\caching\Cache::exists()` | 返回一个布尔值，表示给定的缓存键是否存在于缓存中。 |
| `yii\caching\Cache::delete()` | 从缓存中删除给定的缓存键。 |
| `yii\caching\Cache::flush()` | 清除缓存中的所有数据。 |

### 小贴士

关于每个方法和其使用的更多信息，请参阅`yii\caching\Cache`抽象类中描述的非继承的公共方法，链接为[`www.yiiframework.com/doc-2.0/yii-caching-cache.html`](http://www.yiiframework.com/doc-2.0/yii-caching-cache.html)。

通常，我们可以通过调用`Yii::$app->cache`组件的任何这些方法来使用我们的缓存，如下面的示例所示：

```php
$cache = Yii::$app->cache;
if ($cache->exists('example'))
    $data = $cache->get('example');
else
{
    // Generate data here...
    $data = [];
    // Cache the $data for 100 seconds
    $cache->set('example', $data, 100);
}
return $data;
```

### 缓存依赖项

除了设置具有给定过期时间的缓存外，我们还可以使用某些依赖项（如文件的最后修改时间或某种表达式的表达式）来缓存数据，如果依赖项发生变化，我们的数据将自动过期。Yii2 提供了几个我们可以使用的依赖项。

| 方法 | 说明 | 类引用 |
| --- | --- | --- |
| `yii\caching\ChainedDependency` | 允许我们将多个依赖项链接在一起，如果任何依赖项失败，则使缓存项过期。 | [`www.yiiframework.com/doc-2.0/yii-caching-chaineddependency.html`](http://www.yiiframework.com/doc-2.0/yii-caching-chaineddependency.html) |
| `yii\caching\DbDependency` | 基于给定 SQL 查询的一个依赖项。如果查询的结果发生变化，缓存将被失效。 | [`www.yiiframework.com/doc-2.0/yii-caching-dbdependency.html`](http://www.yiiframework.com/doc-2.0/yii-caching-dbdependency.html) |
| `yii\caching\FileDependency` | 基于文件的最后修改时间的一个依赖项。 | [`www.yiiframework.com/doc-2.0/yii-caching-filedependency.html`](http://www.yiiframework.com/doc-2.0/yii-caching-filedependency.html) |
| `yii\caching\ExpressionDependency` | 由布尔表达式表示的依赖项。 | [`www.yiiframework.com/doc-2.0/yii-caching-expressiondependency.html`](http://www.yiiframework.com/doc-2.0/yii-caching-expressiondependency.html) |
| `yii\caching\TagDependency` | 基于可管理的标签数组的一个依赖项。 | [`www.yiiframework.com/doc-2.0/yii-caching-tagdependency.html`](http://www.yiiframework.com/doc-2.0/yii-caching-tagdependency.html) |

### 小贴士

查看每个依赖项的类引用，以获取有关其可用属性和方法的信息。

在我们之前的示例基础上，我们可以添加一个缓存依赖，如下面的示例所示。在下面的代码中，我们创建了一个对名为`data.csv`的文件的依赖，该文件可以包含报告或其他我们希望生成或导入到我们的应用程序中的数据：

```php
$cache = Yii::$app->cache;
if ($cache->exists('example'))
    $data = $cache->get('example');
else
{
    // Generate data here...
    $data = [];

    $dependency = new \yii\caching\FileDependency(['fileName' => 'data.csv']);
    // Cache $data for 100 seconds using the key "example" with a FileDependency
   $cache->set('example', $data, 100, $dependency);
}
return $data;
```

### 数据库查询缓存

使用 Yii2，我们还可以缓存数据库查询的结果。要启用查询缓存，我们需要在我们的数据库组件中设置三个属性：`$enableQueryCache`，用于切换查询缓存的开启和关闭；`$queryCacheDuration`，用于设置查询应该缓存的持续时间；以及`$queryCache`，用于指定应使用的缓存组件。

以下连接示例说明了如何启用查询缓存：

```php
<?php return [
    // [...],
    'components' => [
        // [...]
        'db' => [
            'class'        => 'yii\db\Connection',
            'dsn'          => 'mysql:host='127.0.0.1;dbname=masteringyii',
            'username'     => '<username>,
            'password'	=> '<password>',
            'charset' 	=> 'utf8',

            'queryCacheEnabled'  => true,
 // 0 = Never expires
 'queryCacheDuration' => 0,
 'queryCache'      => 'cache'
        ],

    'cache' => [
            'class' => 'yii\caching\FileCache',
        ]
    ]
];
```

在配置数据库查询缓存后，我们可以通过添加或链接缓存方法到我们的查询上来缓存单个 DAO 查询的结果，如下面的示例所示：

```php
$duration = 100; // 100 seconds
$results = $db->createCommand('SELECT * FROM users WHERE id=1;')->cache($duration)->queryOne();
```

或者，如果我们想缓存多个查询，我们可以直接调用`yii\db\Connection::cache()`函数：

```php
$result = $db->cache(function ($db) {
    $result = $db->createCommand('SELECT * FROM users WHERE id=1;')->queryOne();
    return $result;
}, $duration, $dependency);
```

`ActiveRecord`也可以通过从`ActiveRecord`模型获取数据库组件来利用查询缓存，如下面的示例所示：

```php
$result = User::getDb()->cache(function ($db) {
    return User::find()->where(['id' => 5])->one();
}, $duration, $dependency);
```

此外，在查询缓存中，我们可以通过将`noCache()`方法链接到我们的查询上来排除某些查询的缓存，如下面的示例所示：

```php
$result = $db->cache(function ($db) {
     // Cache queries in this block

    $db->noCache(function ($db) {
 // Do not cache queries in this block
 });

    // Don't cache this query either
    $customer = $db->createCommand('SELECT * FROM users WHERE id=1')->noCache()->queryOne();
    return $result;
});
```

### 提示

一些数据库，如 MySQL，在其软件层中实现了自己的内置缓存。同时实现 MySQL 的本地查询缓存和 Yii2 的查询缓存可能会导致确保正确数据呈现的问题。此外，任何作为资源处理器返回的数据都不能由 Yii2 缓存。此外，一些缓存，如`Memcache`，限制了可以与特定键关联的数据量。在使用查询缓存时，请注意这些限制。

## 片段缓存

片段缓存建立在数据缓存之上。Yii2 中的片段缓存允许我们缓存页面的一部分，并在每次请求时呈现该缓存片段，而不是重新生成页面的全部内容。通常，我们可以通过以下代码块使用片段缓存：

```php
// $id = ...a unique key...
// $this = ...instance of yii\web\View...;
// Begin our cache and check to see if the data is already cached.
// If content is found, beginCache will output data, otherwise
// the conditional will execute.
if ($this->beginCache($id))
{
    // Our cached content goes here
    $this->endCache();
}
```

与数据缓存一样，片段缓存也支持多个条件，如持续时间、依赖项、变化和切换片段缓存的开启和关闭。这些条件可以作为键值数组添加到`beginCache()`方法的第二个参数中，如下面的示例所示：

```php
if ($this->beginCache($id, [
    // Time we want the fragment cache valid for
    'duration' => 100,

    // Any dependencies we want to add
    'dependency' => [
        'class' => 'yii\caching\DbDependency',
        'sql' => 'SELECT MAX(updated_at) FROM user',
    ],

    // Conditionally enable the cache for any boolean value
    'enabled' => Yii::$app->request->isGet,

    // Have a variation of this page for every language
    'variations' => Yii::$app->language	

]))
{
    // Our cached content goes here
    $this->endCache();
}
```

## 页面缓存

作为仅缓存网页片段的替代方案，使用 Yii2，我们还可以缓存整个页面，并在每次页面加载时提供缓存的副本而不是生成页面。这对于我们有一个读密集型应用程序，如博客来说非常有用。在 Yii2 中，通过将`yii\filters\PageCache`过滤器添加到我们的控制器中的`behaviors()`方法来实现页面缓存，如下面的示例所示。与片段缓存一样，我们可以为我们的页面指定变体，指定内容应该无效化的依赖项，以及缓存的时间长度。与其他过滤器一样，我们还可以使用`only`和`except`参数指定我们想要我们的缓存应用到的操作。以下示例说明了页面缓存的使用：

```php
public function behaviors()
{
    return [
        [
            'class' => 'yii\filters\PageCache',
            'only' => ['article'],
            'duration' => 60,
            'variations' => [
                Yii::$app->language,
                Yii::$app->user->isGuest
            ],
            'dependency' => [
                'class' => 'yii\caching\DbDependency',
                'sql' => 'SELECT MAX(updated_at) FROM articles',
            ],
        ],
    ];
}
```

## HTTP 缓存

数据、片段和页面缓存都是我们可以用来优化我们应用程序服务器端性能的策略。为了进一步提高我们应用程序的性能，我们还可以将带有我们应用程序的标题发送出去，以指示我们希望客户端的浏览器缓存我们页面的输出。这三个标题是`Last-Modified`、`ETag`和`Cache-Control`。通过将这三个标题与我们的应用程序一起发送，我们可以显著减少客户端对我们应用程序发送的 HTTP 请求的数量，对于不经常更改的页面。在 Yii2 中，HTTP 缓存是通过`yii\filtersHttpCache`过滤器实现的：

+   第一个标题`Last-Modified`通知客户端页面最后更改的时间。如果客户端向服务器发送 HEAD 请求并看到`Last-Modified`标题与它目前拥有的不同，它将重新请求页面并缓存它。否则，它将从客户端的缓存中加载页面。

+   `ETag`标题用于表示标签的哈希。与`Last-Modified`标题一样，如果`ETag`哈希发生变化，浏览器就知道它必须重新下载页面。

+   最后，`Cache-Control`标题指示页面应该存储在哪种类型的缓存中，以及存储多长时间。默认情况下，Yii2 将为该标题发送`public; max-age: 3600`，这将指示客户端应将内容缓存 3600 秒或 1 小时。

    ### 小贴士

    更多关于 Cache-Control 头部的信息可以在 w3c 规范参考指南中找到，链接为[`www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.9`](http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.9)。

下面是一个说明如何结合使用这三个标题的示例：

```php
public function behaviors()
{
    return [
        [
            'class' => 'yii\filters\HttpCache',
            'only' => ['index'],
            'lastModified' => function ($action, $params) {
                $q = new \yii\db\Query();
                return $q->from('articles')->max('updated_at');
            },
            'etag' => function($action, $params) {
                $article = Article::find()->where(['id' => \Yii::$app->request->get('id')])->one();
                return serialize([$article->title, $article->content]);
            },
            'cacheControlHeader' => 'public; max-age:3600'
        ],
    ];
}
```

### 小贴士

注意，对于 HTTP 缓存，您只需要指定您想要发送的标题。指定多个标题可以为您提供更精细的控制，以确定缓存何时过期。

## 缓存数据库模式

为了使`ActiveRecord`模型自动工作，Yii2 将在每次查询的开始自动查询数据库以确定应用程序的模式。虽然这在开发环境中很有用，但在我们的模式很少改变的生产环境中，这个操作是不必要的。我们可以通过启用数据库组件的三个属性来告诉 Yii2 缓存我们的数据库模式，以改善数据库操作的性能：`$schemeCache`，它代表我们想要使用的缓存组件；`$schemaCacheDuration`，它定义了 Yii2 缓存我们的模式的时间长度；以及`$enableSchemaCache`，它启用或禁用模式缓存。

以下 MySQL 数据库组件说明了模式缓存属性的使用：

```php
<?php return [
    // [...],
    'components' => [
        // [...]
        'db' => [
            'class'        => 'yii\db\Connection',
            'dsn'         => 'mysql:host='127.0.0.1;dbname=masteringyii',
            'username'     => '<username>,
            'password'	=> '<password>',
            'charset' 	=> 'utf8',

            'enableSchemaCache'   => true,
 'schemaCacheDuration' => 0,
 'schemaCache' 	      => 'cache'
        ],

        'cache' => [
            'class' => 'yii\caching\FileCache',
        ]
    ]
];
```

### 小贴士

当启用模式缓存时，在应用新的迁移之后运行`cache/flush`命令，以便 Yii2 能够获取你的新数据库结构。

# 通用性能提升

为了获得显著的性能提升，你可以对你的应用程序以及你的 Web 服务器环境进行一些更改，这些更改可以显著提高应用程序的性能。

## 启用 OPCache

与 C 和 C++等编译型语言不同，PHP 是一种解释型脚本语言。因此，每次我们的 Web 服务器请求一个新页面，或者每次我们从命令行运行一个命令时，PHP 都需要将我们的代码解释成服务器可以实际运行的机器码。即使我们的源代码没有改变，PHP 也会在每次请求时自动执行这一步骤。在我们的开发环境中，这允许我们简单地更改源代码，保存文件，然后在页面上重新加载以查看我们的更改。然而，在生产环境中，这一步骤是不必要的，因为我们的代码只有在进行部署时才会改变。

从 PHP 5.5 开始，Zend Framework Technologies Ltd 发布了一个名为 OPCache 的新工具，并将其集成到 PHP 核心中。一旦启用，OPCache 将缓存由我们的 PHP 代码生成的编译和优化后的操作码，并将其存储在共享内存存储中。如果我们的代码再次运行，OPCache 将查找那个共享内存存储中的代码并执行它，而不是重新解释我们的原始源代码文件。根据我们应用程序的大小，启用 OPCache 可以对我们的应用程序性能产生重大影响。此外，由于 OPCache 现在已集成到 PHP 中，启用它相对简单。

### 小贴士

注意，Zend OPCache 和 APCCache 都可以配置为缓存 PHP 的操作码。强烈建议你不要同时运行 Zend OPCache 和 APCCache，因为这可能会在 PHP 中引起不稳定。由于 Zend OPCache 由 PHP 维护者维护，建议你使用它而不是 APC。

根据你的包管理器，OPCache 可能已经内置到你的 PHP 实例中，或者作为外部扩展提供。一个简单的方法是检查 OPCache 是否已安装，可以通过从你的命令行运行以下命令来完成：

```php
$ php –m

```

如果 OPCache 已安装，你应该在输出中看到**Zend OPcache**。如果你没有看到这个输出，你需要从你的包管理器安装 OPCache。一旦安装了 OPCache，你可以通过将以下内容添加到你的`php.ini`文件或 PHP INI 包含文件夹中的文件，并重新启动你的 Web 服务器来启用它：

```php
zend_extension=opcache.so
opcache.enable = true
opcache.enable_cli = true
opcache.save_comments = false
opcache.enable_file_override = true
```

### 小贴士

当你执行部署时，你需要清除 OPCache 以使新代码生效。通常，这是通过重新启动你的 Web 服务器或 PHP 进程来完成的。或者，你可以使用像`cachetool`（可在[`github.com/gordalina/cachetool`](https://github.com/gordalina/cachetool)）这样的工具来清除缓存。使用像`cachetool`这样的工具是有益的，因为它允许你在不重新启动 Web 服务器并面临潜在的中断的情况下清除 OPCache。

## 优化 Composer 依赖项

作为部署的一部分，你可以进行另一种性能改进，即从你的生产部署中排除开发依赖项：

```php
$ composer install --no-dev

```

由于我们的开发依赖项在开发中使用，将此代码加载并注册到我们的应用程序中只会给我们的应用程序增加额外的开销。

此外，我们可以在安装 composer 依赖项时通过运行以下命令来指示 Composer 优化它生成的自动加载器：

```php
$ composer install –o

```

或者，我们可以在安装依赖项后通过运行以下命令生成一个优化的自动加载文件：

```php
$ composer dumpautoload -o

```

通过优化 Composer 的自动加载文件，我们可以减少在源代码中加载类时所需的文件和磁盘查找次数，这反过来会使我们的应用程序运行更快。

## 升级到 PHP 7

在发布时，PHP 7 已经发布，它包含了一个重构的 PHP 引擎，能够以显著较少的指令解释、编译和执行相同的 PHP 代码。通过减少 CPU 指令和内存使用，PHP 7 比 PHP 5.6 快得多。为了获得显著的性能提升，考虑将你的 PHP 实例从 5.6 升级到 7。

## 切换到 Facebook 的 HHVM

作为升级到 PHP 7 的替代方案，你可以考虑保留 PHP 引擎并切换到 HHVM，这是 Facebook 创建的针对 PHP 的重构引擎。与 PHP 7 一样，HHVM 比 PHP 5.6 快得多，对于高流量应用程序，它可以显著降低托管高流量应用程序的相关成本。然而，与 PHP 7 不同的是，HHVM 不支持你可能习惯的所有 PHP 模块。此外，虽然 Yii2 完全兼容 HHVM，但第三方 Composer 包可能不兼容，如果不进行彻底测试，可能会出现问题。有关 HHVM 的更多信息，请参阅[`docs.hhvm.com/manual/en/index.php`](http://docs.hhvm.com/manual/en/index.php)的 HHVM 文档。

# 安全考虑

当使用 Yii2 时，重要的是要记住遵循安全最佳实践，以确保应用程序、运行它们的服务器、我们收集的数据以及将此信息托付于我们的最终用户的安全。在之前的章节中，我们探讨了如何使用`yii\base\Security`类安全地加密和散列数据，以及如何使用 Bcrypt 等哈希算法来保护密码。在本节中，我们将介绍一些在构建我们的应用程序时可以应用的其他安全最佳实践。

## 证书

在几乎每个 Yii2 将提供后端支持的应用程序中，我们的客户端（浏览器或本地客户端）将通过 HTTP（超文本传输协议）与我们的应用程序通信。确保我们的客户端从客户端提交的信息以相同的状态到达我们的服务器的一个简单方法是通过 TLS（传输层安全性）协议传输由受信任的证书颁发机构签名的证书来加密客户端和服务器之间的流量。

### 备注

TLS（传输层安全性）是 SSL（安全套接字层）的后继者，两者通常统称为 SSL 证书。截至 2014 年，所有版本的 SSL（1.0、2.0 和 3.0）由于 SSL 协议本身存在的已知安全问题而被弃用。其继任者，TLS 1.1 和 1.2 版本不受影响，并且在通过 HTTP 在客户端和服务器之间加密数据时是推荐的协议。

将已签名和受信任的证书添加到我们的服务器具有几个主要优点：

+   在传输过程中加密数据可以防止第三方查看和操纵数据。健康信息、信用卡信息、用户名和密码都可以通过在传输过程中加密数据来得到保护。

+   客户端可以锁定我们发布的证书，这样他们就知道只有当我们的证书与他们锁定的证书匹配时才与我们通信。这防止了中间人攻击（MITM）并阻止他人了解我们的数据。此外，当使用锁定证书时，我们的客户端将知道不要与冒充我们的服务器通信。再次，这保护了我们和我们的用户。

+   搜索引擎如 Google 和 Bing 会给使用 TLS 的网站更高的排名

+   在我们的网络服务器中实施 TLS 是一个简单的任务，在现代计算机上，它几乎不产生开销

当实施 TLS 时，您可以使用多种资源来确定最安全的加密套件，并验证您的配置是否安全。例如，[`cipherli.st`](https://cipherli.st)网站提供了一系列现代加密套件，适用于各种网络服务器和配置。Qualys 的 SSL Labs 网站([`www.ssllabs.com/ssltest/`](https://www.ssllabs.com/ssltest/))也可以为您提供完整的 TLS 配置报告，并验证您的网络服务器配置。结合这些工具，可以帮助更好地保护您的应用程序和基础设施。

## Cookie

当使用`yii\web\Request`和`yii\web\Response`从 cookie 中检索数据时，Yii2 会自动使用您的 cookie 验证密钥对您的 cookie 信息进行加密：

```php
return [
    // [...],
    'components' => [
        // [...]
        'request' => [
            'cookieValidationKey' => '<your secret key here>',
        ],
    ],
];
```

当处理 cookie 和会话 cookie 时，我们可以通过向我们的 cookie 添加额外的属性来采取额外的保护措施，例如`yii\web\Cookie::$secure`和`yii\web\Cookie::$httpOnly`。通过将我们的 cookie 标记为`secure`，我们可以确保我们的 cookie 只通过安全连接发送，如前所述。此外，通过将我们的 cookie 设置为`httpOnly`，我们可以确保 JavaScript 和其他网络脚本语言无法读取我们的 cookie。通过配置这两个标志，我们可以显著提高我们应用程序的安全性。

## 防止跨站脚本攻击

作为网络开发的一般规则，每次我们展示由最终用户提交的信息时，我们都应该对其进行编码，以便保护我们的网站和用户免受跨站脚本（XSS）攻击。XSS 发生在用户提交的数据，当显示在我们的页面上时，可以被我们的浏览器解释。这可能是一些无害的操作，例如在我们的标记中添加`<em>`或`<b>`标签，或者可能是更危险的操作，例如注入一个`<script>`标签来跟踪用户信息或将他们重定向到另一个网站。幸运的是，Yii2 提供了两种处理最终用户提交数据并显示的方法。

我们可以用来保护我们的网站免受 XSS 攻击的第一种方法是通过使用`yii\helpers\Html::encode()`方法对最终用户数据进行编码，如下面的示例所示：

```php
<?php echo \yii\helpers\Html::encode($data); ?>
```

当使用这种方法对数据进行编码时，Yii2 会将`<`和`>`等标签转换为现代浏览器知道如何显示和解释的 HTML 编码实体。

在我们确实希望将最终用户数据以 HTML 形式显示的情况下，我们可以使用`yii\web\HtmlPurifier::purify()`来正确解析我们想要的数据，同时不允许注入 JavaScript 代码：

```php
<?php \yii\helpers\HtmlPurifier::process($longData);
```

### 注意

即使配置得当，HtmlPurifier 也可能非常慢。在部署代码之前，请确保您正确理解和配置 HtmlPurifier，因为它可能会显著影响您应用程序的性能。有关如何在 Yii2 中配置 HtmlPurifier 的更多信息，请参阅[`www.yiiframework.com/doc-2.0/yii-helpers-htmlpurifier.html`](http://www.yiiframework.com/doc-2.0/yii-helpers-htmlpurifier.html)，HtmlPurifier 的完整文档可以在[`htmlpurifier.org/`](http://htmlpurifier.org/)找到。

## 启用跨站请求伪造保护

**CSRF**（**跨站请求伪造**）是许多网站处理的一种常见漏洞，Yii2 可以帮助我们保护自己免受其侵害。在处理客户端请求时，我们通常假设请求来自用户本人。然而，使用 JavaScript，我们可以在用户不知情的情况下在后台发送虚假请求。这些请求可能很简单，比如在用户不知情的情况下将其从特定服务中注销，或者抓取有关用户信息的特定页面并将其传输到恶意服务器。Yii2 自动保护我们免受 CSRF 攻击。您唯一可以执行的其他保护措施是遵循 HTTP 规范（例如，不允许在 GET 请求上更改状态）。

### 小贴士

注意，可能会有很多次出于各种原因需要禁用 CSRF。在我们的控制器中，我们可以通过在动作中添加以下代码来为特定动作禁用 CSRF：将`Yii::$app->controller->enableCsrfValidation`设置为`false`。

# 摘要

在本章中，我们介绍了多种提高和探索我们应用程序性能的方法，并学习了如何提高我们应用程序的安全性。我们探讨了如何使用数据、页面、片段、HTTP、数据库和模式缓存来提高我们应用程序的性能。我们还发现了我们可以对 Yii2 和 PHP 进行的一般改进，以使我们的应用程序运行得更快。最后，我们发现了通过使用证书、启用某些 cookie 属性以及保护我们的网站免受 XSS 和 CSRF 攻击的几种方法来提高我们应用程序安全性的几种方法。

在我们的最后一章中，我们将介绍如何使用 Yii2 加快我们已有的快速开发时间，学习如何通过日志记录探索我们的应用程序，并发现快速且安全的方法来部署我们的应用程序，几乎不会出现停机或服务中断。
