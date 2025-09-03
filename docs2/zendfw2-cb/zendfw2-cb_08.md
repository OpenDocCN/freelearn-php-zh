# 第八章. 优化性能

在本章中，我们将涵盖：

+   缓存，以及何时缓存

+   理解和使用存储插件

+   设置缓存系统

# 简介

在一个我们希望立即获取数据的社会中，确保我们的网站和应用能够尽快提供数据是非常重要的。当我们排除了任何明显的减速原因，例如网络基础设施或服务器配置后，我们就可以开始考虑缓存。这一章全部关于应该缓存什么以及如何缓存，从而使我们的生活变得更加快捷。

# 缓存和何时缓存

缓存，每个人都知道它，每个人都在谈论它，但什么是缓存呢？在它的最纯粹的本质上，缓存就是尽可能快地向用户提供应用。这就是我们在本食谱中将要讨论的，何时以及如何进行缓存。

## 准备工作

我们将再次使用 Zend Framework 骨架应用，因此明智的做法是先设置好它。

## 如何操作…

在开发应用时，缓存可能不是设计阶段立即考虑的事情，而且很可能是当应用上线后一段时间，你发现应用的响应速度比你最初上线时要慢。

这正是考虑实现缓存的理想时间（虽然不是完美的，因为显然应该在设计阶段进行），以加快应用的响应速度。

当我们谈论缓存时，一个常见的误解是我们仅仅在谈论缓存 HTML 输出。这完全不是事实，因为我们有几种强大的 PHP 缓存方法。

以下列表是我们可用于缓存应用不同部分的一些方法的集合：

+   缓存 ZF2 配置

+   缓存渲染后的输出

+   缓存类映射

我们现在将更深入地探讨前面列表中提到的各种方法。

### 缓存配置

在你的应用中，最静态的代码部分可能就是配置。哦，但我们需要配置来正确加载我们的应用，但与此同时，我们可能因为所有需要合并的操作而讨厌它，直到我们得到配置的最终版本。

但不必担心，因为我们只需缓存合并后的配置，这样你的应用就不必再解析所有内容了！实际上，这个过程如此简单，以至于给出示例几乎令人捧腹（`/config/application.config.php`）：

```php
<?php
return array(
  // Look for this ke y in the configuration array.
  'module_listener_options' => array(

    // Enable the config cache.
    'config_cache_enabled' => true,

    // If we want to give the cache a special filename
    // we can just type a name here.
    'config_cache_key' => 'configuration'

    // The directory where we want to write the cache 
    // to. Don't forget that we need read/write access 
    // to this directory by the process running the app, which in 
    // most cases is the web server process!
    'cache_dir' => 'data/cache/',
  ),
);
```

就这样。不需要任何花哨的东西来使这个工作正常，因为使这个工作正常所需的一切都已经内置在 Zend Framework 2 中。

这是一个非常有效的方法来开始缓存所有静态内容，尽管它可能不会给应用带来巨大的速度提升（除非我们真的有成百上千个模块），但它将是一个不应该被遗忘的方法。

### 缓存输出

缓存输出在当我们有很多静态文件通常不会或很少改变时很有用。当我们谈论变化不大的内容时，我们可以想到博客文章或新闻条目，因为它们通常只生成一次并无限期地发布。显然还有更多有用的输出类型可以缓存，但我们将只给出一个例子来展示我们认为静态的输出是如何容易缓存的。

首先，我们需要在我们的模块中创建配置，以确保在`ServiceManager`中启用缓存（`/module/SomeModule/config/module.config.php`）：

```php
<?php
return array(
  // We need to define the ServiceManager
  'service_manager' => array(
    // We will call it cache-service
    'cache-service' => function () {
      // Return a new cache adapter
      return \Zend\Cache\StorageFactory::factory(array(
        'adapter' => array(
          // We want to use the cache that is being 
          // stored on the filesystem
          'name' => 'filesystem',
          'options' => array(
            'cache_dir' => 'data/cache/',

            // This is the amount in minutes the cache is valid
            'ttl' => 100
          ),
        ),
      ));
    },
  },
);
```

现在我们创建了配置，让我们继续在我们的`/module/SomeModule/Module.php`文件的`onBootstrap`方法中控制缓存：

```php
<?php
// Don't forget the namespace (obviously)
namespace SomeModule;

// We need this event for the onBootstrap event
use Zend\Mvc\MvcEvent;

// Begin our module class
class Module
{
  // This is going to be run at bootstrap, and will thus 
  // create our events that will create our cached 
  // output
  public function onBootstrap(MvcEvent $e)
  {
    // We will need a list of routes that we deem 
    // cacheable
    $routes = array('blog/pages', 'blog/archives');

    $eventManager = $e->getApplication()->getEventManager();
    $serviceManager = $e->getApplication()->getServiceManager();

    $eventManager->attach(
         MvcEvent::EVENT_ROUTE, 
         function($e) use ($serviceManager)
    {
      $route = $e->getRouteMatch()
                 ->getMatchedRouteName();

      // Check if this is a page that we want to cache, 
      // if not then just exit this method
      if (!in_array($route, $routes)) {
        return;
      }

      // Get the cache-service from the configuration
      $cache = $serviceManager->get('cache-service');

      // Define a unique key that we use for the route
      $key = 'route-'. $route;

      // Check if our cache has the key with our route 
      // content 
      if ($cache->hasItem($key)) {
        // Handle response
        $response = $e->getResponse();

        // Set the content to our cached content
        $response->setContent($cache->getItem($key));

        // Return the response, because when we return 
        // the response from a route event, the 
        // application will output that response.
        return $response;
      }
    }, 
    // Make this priority super low to make sure this 
    // route has already happened
    -1000); 

    // Now we create an trigger for the render event 
    // which will come after the route event. This means 
    // that we didn't have a valid cache, and we will 
    // now use this opportunity to create a cache of our 
    // rendered content.
    $eventManager->attach(
      MvcEvent::EVENT_RENDER, 
      function($e) use ($serviceManager, $routes) 
    {
      // Get the current route name 
      $route = $e->getRouteMatch()
                 ->getMatchedRouteName();

      // Check if this is a page that we want to cache, 
      // if not then just exit this method
      if (!in_array($route, $routes))
        return;

      // Apparently we want to cache the content, so 
      // here we go!
      $response = $e->getResponse(); 

      // Get the cache service from the ServiceManager
      $cache = $serviceManager->get('cache-service');

      // Build up our unique cache key
      $key = 'route-'. $route;

      // And now set the cache item
      $cache->setItem($key, $response->getContent());
    }, 
    // Again the lowest priority to make sure rendering 
    // already has happened.
    -1000);
  }
}
```

现在，每次我们访问我们的应用时，路由事件都会检查我们是否可能有一个特定路由的缓存，如果有，它将返回缓存（当然，如果没有过期的话）。如果路由还没有被缓存，一旦渲染事件被触发，它将根据需要执行缓存。

这个例子的功劳归功于*Jurian Sluiman* (*jurian-sluiman*)，他是[stackoverflow.com](http://stackoverflow.com)网站的用户，也是 Zend Framework 2 的重要贡献者。

### 缓存类映射

类映射文件是那些一旦应用完成合并后就会变得很大的文件之一，基本上是静态的。它之所以是静态的，这显然为我们提供了一个很好的机会，我们可以缓存它，并从应用的合并中减轻一些负担。至于我们缓存的第一个方法，这也只需要我们在配置文件中添加几个属性。

让我们从这个例子开始（`/config/application.config.php`）：

```php
<?php
return array(
  // Look for this key in the configuration array.
  'module_listener_options' => array(

    // Enable the module map cache.
    'module_map_cache_enabled' => true,

    // If we want to give the cache a special filename
    // we can just type a name here.
    'module_map_cache_key' => 'classmap

    // The directory where we want to write the cache 
    // to. Don't forget that we need read/write access  
    // to this directory!
    'cache_dir' => 'data/cache/',
  ),
);
```

再次强调，尽管这可能在整体性能上可能没有显著提升，但我们确信每一丝帮助都是有益的，它肯定会帮助减轻自动加载过程的工作负担。

## 它是如何工作的…

所有这些缓存所做的只是通过在特定时间（`ttl`，也称为生存时间）内保持一切准备就绪来加快应用的速度。它通过在应用需要时提供所需数据，而不需要应用连接到数据库或重新编译模板等，从而加快了应用的速度。

缓存通常是在文件系统中进行的，因为它被认为是一个非常快的选项，而不是通过数据库等。然而，从技术上讲，缓存的最快选项是在内存中（这是因为内存或 RAM 是 CPU 最近的数据存储，因此是最快的）。尽管内存缓存是一种很好的缓存方法，但如果缓存的数据太多，它也可能变成最糟糕的一种。

因此，在仅仅使用一种方法之前，考虑不同的缓存方法（例如，仅对博客文章和应用配置使用文件系统缓存，例如，内存缓存）是明智的。

# 理解和使用存储插件

与自定义一切不同，Zend Framework 2 提供了一个出色的接口，可以通过存储插件来操作存储、删除和检索缓存数据。

## 如何做到这一点……

存储插件用于在开发者觉得需要向适配器添加更多功能时补充存储适配器，而不必 necessarily 制作一个自定义适配器。因此，当我们要修改我们的存储适配器处理缓存的方式时，插件是最方便的工具。

在 Zend Framework 2 中，有几个存储插件可供使用，因此让我们开始进一步解释它们。

### 使用 ClearExpiredByFactor 插件

`ClearExpiredByFactor` 插件会偶尔清除过期的缓存项，这些项由一个设置的因子决定。因子整数越高，缓存清除过期项的可能性就越小。但别忘了；这是一个（伪）随机过程，所有机会都可能是它每次都会被调用。我们理解这非常不符合直觉，所以也许这个从插件中提取的代码片段可以澄清一些问题。

```php
if ($factor && mt_rand(1, $factor) == 1) {
     $storage->clearExpired();
}
```

我们还应该注意，此插件仅在需要写入缓存时才会触发，当读取缓存时不会触发。

可以设置的 `PluginOptions` 是 `setClearingFactor`，它设置清除因子。

### 小贴士

此插件要求存储适配器必须是 `ClearExpiredInterface` 的实例，否则它将不会做任何事情（而且我们永远不会知道，因为它不会记录这个错误）。只有文件系统和内存存储适配器支持此接口。

### 使用 ExceptionHandler 插件

`ExceptionHandler` 插件会捕获在获取/设置缓存时抛出的任何异常，并将其转发到开发者定义的回调函数。

可以设置的 `PluginOptions` 包括：

+   `setExceptionCallback`：这是一个在发生异常时调用的回调函数

+   `setThrowExceptions`：这是一个布尔值（默认 `true`），告诉插件重新抛出它捕获的异常

### 使用 IgnoreUserAbort 插件

`IgnoreUserAbort` 插件确保脚本在写入缓存完成之前不会被中止。这样我们就可以确保我们的缓存中不会有任何损坏的数据。

可以设置的 `PluginOptions` 是 `setExitOnAbort`，它是一个布尔值（默认 `true`），告诉我们我们是否可以随时中止脚本，或者如果我们需要等待我们完成写入。

### 使用 OptimizeByFactor 插件

你想要按因子清除吗？我敢肯定你也不想按因子优化！此插件（伪）随机优化缓存。因子决定了它实际优化的机会，数字越低（在 1 和较大数字之间），机会越大，数字越高，机会越小。我们理解这非常不符合直觉，所以也许这个从插件中提取的代码片段可以澄清一些问题：

```php
if ($factor && mt_rand(1, $factor) == 1) {
     $storage->clearExpired();
}
```

我们还应该注意，此插件仅在需要移除缓存时才会触发，当缓存被读取或写入时不会触发。

可以设置的`PluginOptions`是`setOptimizingFactor`，它设置优化因子。

### 小贴士

此插件仅在具有`OptimizableInterface`实例的存储适配器上工作。如果此接口不可用，它不会抛出错误，所以我们永远不会知道。目前支持此接口的适配器是 Dba 和 Filesystem。

### 使用序列化器插件

`Serializer`插件将在设置和从缓存获取数据时序列化和反序列化数据。

可以设置的`PluginOptions`：

+   `setSerializer`: 这将设置我们想要使用的序列化器，它需要是一个实现了`Zend\Serializer\Adapter\AdapterInterface`类的类

+   `setSerializerOptions`: 如果在`setSerializer`选项（作为字符串的全类名）中给出了字符串，则需要在选项中设置实例化选项

### 使用任何插件

幸运的是，插件很容易使用，我们只需要将它们添加到存储适配器即可使其工作。

我们知道有几种方法可以实例化插件，但我们将只显示一种方法来展示它基本上是如何工作的：

```php
<?php

// Use the following libraries for our example
use Zend\Cache\Storage\Plugin\Serializer;
use Zend\Cache\Storage\Adapter\FileSystem;

// Initialize our Serializer plugin
$plugin = new Serializer();

// Initialize our FileSystem adapter
$adapter = new FileSystem();

// Now bind the two together
$adapter->addPlugin($plugin);
```

这就是所有需要配置以使其协同工作的内容。在一个 MVC 应用程序（我们可能将使用 Zend Framework 2）中，插件可以位于非常不同的位置。通常，我们希望在配置或引导事件中配置它，如果我们打算在整个应用程序中持续使用它，因为这样可以节省时间，与多次实例化相比。

## 它是如何工作的…

插件附加到存储适配器上，并且因为它们将自己附加到存储适配器的事件上，所以它们才会工作。当这些事件被触发时，功能也会被触发。这真的很简单，而且对此没有真正的进一步解释所需的。

# 设置缓存系统

学习新技术的最佳方式总是最好的例子。这就是为什么我们将向您展示如何在应用程序的不同部分实现缓存系统。

## 准备工作

在这个菜谱中，我们将展示一个简单的系统，该系统利用了缓存。我们还将展示一些基准测试，以便我们可以清楚地看到没有缓存和有缓存的系统之间的差异。此项目的代码也可以在书中找到，其中包含一些示例类，以便我们可以更好地测量性能。我们不会讨论任何示例类（所有这些类都可以在`/module/Application/src/Application`目录中找到），但我们将参考它们在示例中。

## 如何做…

设置一个简单的缓存系统很容易，但大多数时候的问题是，从哪里开始。

### 在缓存之前基准测试我们的应用程序

对于基准测试，我们将使用一个名为`ab`的应用程序，它是 ApacheBench 的缩写。这是一个标准工具，包含在 Apache 网络服务器中，无论是 Microsoft Windows 版本还是 Linux 版本；在我们的食谱中，我们将使用基准测试工具的 Linux 版本，不用担心，因为两个版本都完全一样。

对于我们的基准测试，我们将不使用任何缓存，并在`Application\Controller\IndexController`（`/module/Application/src/Application/Controller/IndexController.php`）中使用以下代码来生成我们的荒谬长输出：

```php
<?php

// Don't forget to set our namespace
namespace Application\Controller;

// Use the following classes 
use Zend\Mvc\Controller\AbstractActionController;
use Zend\View\Model\ViewModel;

// Define our class name and extend
class IndexController extends AbstractActionController
{
  // We will just use the index for this
  public function indexAction()
  {
    // Initialize our LongOutput class
    $output = new \Application\Model\LongOutput();

    // echo our stupidly long output 
    echo '<!-- '. $output->run(1500). ' -->';

    // Just return a view model, it doesn't affect us
    return new ViewModel();
  }
}
```

这个操作将输出一个非常长的字符串，非常复杂，但我们并不真的关心这一点，因为我们只想测量创建这样一个字符串需要多长时间。现在我们可以开始第一个基准测试。

以下命令将用于进行基准测试：

```php
$ ab -c 4 -n 10 http://localhost/

```

命令表示并发数为四（`-c 4`），我们想在`localhost`上运行测试十次（`-n 10`），作为我们的网站。这意味着我们的页面总共将被访问 40 次，这将给我们一个相当清晰的平均响应时间视图。

以下是对基准测试最重要的结果的回顾。显然，其余的结果也有一定的兴趣，但我们目前只对响应时间感兴趣。

```php
Time taken for tests:   18.111 seconds

```

我们将使用 18.111 秒作为比较所有其他结果的基础。

### 实现配置/类映射缓存

首先，我们将实现配置缓存，因为这是所有缓存的基础（至少我喜欢这样想）。

我们可以通过向`/config/application.config.php`文件添加以下配置来实现这一点：

```php
<?php
// Lets add our options to the configuration array, 
// please be aware that we don't show any other options 
// here that could very well be in the configuration 
// already.
return array(
  // We should add our options inside this array key
  'module_listener_options' => array(
    // Enable the config cache
    'config_cache_enabled' => true,

    // Give the config cache a file name like module-
    // config-cache.config.php
    'config_cache_key' => 'config',

    // Enable the class map caching
    'module_map_cache_enabled' => true,

    // Give the class map cache a file name like module-
    // classmap-cache.classmap.php
    'module_map_cache_key' => 'classmap',

    // Use our data/cache as the cache directory 
    // (remember this directory need to be writeable for 
    // the web server).
    'cache_dir' => 'data/cache',
    // We don't want to check the module dependencies as 
    // that is the job of the developer, it just takes 
    // time to do this and is pretty much useless.
    'check_dependencies' => false,
  ),
);
```

现在，我们已经启用了配置/类映射缓存，这应该会给我们带来一个非常（非常）小的响应时间增加。当然，当我们的应用程序更大，有更多模块时，这个差异会更大。

让我们再次进行基准测试，看看有什么区别：

```php
Time taken for tests:   15.428 seconds

```

如我们所见，我们的结果已经显著不同，实际上快了 14.2%，令人震惊。然而，我们不应忘记，我们的应用程序非常小，如果未来应用程序规模扩大，这个百分比可能实际上会更小。尽管如此，这仍然是一个明显的迹象，表明缓存配置和类映射是一个好的实践。

### 小贴士

我们应该注意配置缓存系统中的一个小错误是，我们不能使用闭包（也称为匿名函数）。如果我们这样做，我们会得到一个 PHP 致命错误，如下所示：

```php
Call to undefined method Closure::__set_state() in your_configuration_cache.php on line XX
```

### 实现类缓存

由于我们有这个非常长的输出，使用`ClassCache`适配器来缓存生成此输出的单个方法输出是很有趣的。我们还知道我们的`LongOutput`模型没有改变输出的内容，我们可以安全地缓存输出。

为了使这种缓存方法工作，我们需要确保配置缓存已被关闭，否则它将导致 PHP 错误。

我们首先将更改`Application`模块中的`module.config.php`，以初始化我们的缓存存储适配器。之后，我们将更改`Application\Controller\IndexController`，以便我们可以使用我们的模式。我们只需将以下代码添加到`/module/Application/config/module.config.php`：

```php
<?php
return array(
  // We are configuring the service manager
  'service_manager' => array(
    'factories' => array(
      // Initialize our file system storage
      'Zend\Cache\StorageFactory' => function() {
        return Zend\Cache\StorageFactory::factory(
          array(
            'adapter' => array(
              'name' => 'filesystem',
              'options' => array(
                // Define the directory to store the 
                // cache in 
                'cacheDir' => 'data/cache',
              ),
            ),
            // For the file system storage we need to 
            // have the serializer plugin enabled, 
            // otherwise thing just go wrong when we  
            // want to storage a class or so
            'plugins' => array('serializer'),
          ),
        );
      }
    ),
    // We want to call our cache with the 'cache' key
    'aliases' => array(
      'cache' => 'Zend\Cache\StorageFactory',
    ),
  ),
);
```

现在我们已经初始化了缓存，我们需要确保我们的输出也被缓存。这将在我们的`Application`模块的`IndexController`（`/module/Application/src/Application/Controller/IndexController.php`）中完成：

```php
<?php

// Set the namespace
namespace Application\Controller;

// Define the imports
use Zend\Mvc\Controller\AbstractActionController;
use Zend\View\Model\ViewModel;

// Define the class name and extend
class IndexController extends AbstractActionController
{

  // Begin our index action again
  public function indexAction()
  {
    // This time we want to make sure our class is 
    // loaded in to the ClassCache pattern so that we 
    // can eventually cache the output of our class 
    // method    
    $pattern = \Zend\Cache\PatternFactory::factory(
           'class', array(
      'storage' => $this->getServiceLocator()->get('cache'),
      'class' => '\Application\Model\LongOutput'
    ));

    // Now call our method through the ClassCache 
    // pattern with the same arguments as the previous 
    // test
    echo '<!-- '. $pattern->call('run', array(1500)). '-->';

    // Return the view model again because we don't 
    // actually do anything with it
    return new ViewModel();
  }
}
```

如果我们现在查看基准测试，我们可以看到以下缓存导致了以下性能提升：

```php
Time taken for tests:   14.956 seconds

```

如我们所见，这几乎是在原始基准测试上提高了 17.4%，这显然是一个巨大的改进。它也比配置/类映射缓存提高了 3.2%的响应速度。我们知道这听起来并不太令人印象深刻，我们也理解你的失望。然而，请理解，在现实生活中，数据库调用或服务调用可能比这长得多，因此改进的百分比会更大！

这个方法只有一个小问题；那就是我们不会以这种方式缓存配置/类映射。因为我们想尽可能优化我们的应用程序，这显然不是好的做法。但是，不要慌张，这个问题有一个解决方案，它以`StorageCacheFactory`的形式出现！

我们没有立即讨论这个问题，因为最好看到不止一种编码方式，至少这是我的个人选择。

我们将要移除在`/module/Application/config/module.config.php`中添加的配置，并添加以下配置：

```php
<?php
// We need to assume that we have stripped the previous 
// configuration out of here and it is back to the 
// default configuration file
return array(
  'service_manager' => array(
    // Instantiate the cache through our storage cache 
    // factory. It will look for the 'cache' key to 
    // initialize the cache
    'factories' => array(
        'cache' => '\Zend\Cache\Service\StorageCacheFactory',
    ),
  ),

  // And here we go, initializing the cache
  'cache' => array(
    // We want to use the filesystem adapter
    'adapter' => 'Filesystem',
    'options' => array(
      // Of course we need to set the directory to cache 
      // in
      'cache_dir' => 'data/cache'
    ),

    // We also want the serializer otherwise it will 
    // throw an exception
    'plugins' => array('Serializer'),
  ),
);
```

如果我们现在回过头来对配置和类映射缓存进行基准测试，我们会得到以下结果。

```php
Time taken for tests:   14.303 seconds

```

如我们所见，这次在两个缓存系统都启用的情况下，与原始版本相比，我们得到了 21%的速度提升。

## 它是如何工作的...

总是缓存我们经常使用且确信其持久性的东西是很好的。如果我们知道一个类的输出不会改变，但例如仅仅做一些我们知道它会进行的计算，那么它将是一个很好的缓存候选者。不要忘记，依赖于第三方输入的缓存方法，如数据库，更难缓存，因为它们需要一定的存活时间，在这个时间内缓存知道它们缓存的这些数据已经过时。

另一件需要注意的事情是缓存过多，这样实际上你的应用程序会变慢，而不是加快，因为缓存太忙于刷新/获取和设置缓存，而不是实际输出它。然而，设置一个周期性的`cron`（类似于 Windows 用户的计划任务）过程来进行自动清理和自动优化是一个好方法。
