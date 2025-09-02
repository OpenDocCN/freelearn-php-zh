# 第十二章：JavaScript 和 Ajax API

到目前为止，在这本书中，我们只讨论了可以被认为是与后端开发相关的话题。这意味着大量的 PHP 与 API 和数据库一起工作，等等。这是因为这本书面向的是模块开发者，而不是“主题开发者”。此外，这本书的作者承认自己不是 JavaScript 或任何前端开发者。

尽管如此，在本章中，我们将转换方向，简要讨论一下 *前端开发*，即如何在 Drupal 8 应用程序中与 JavaScript 一起工作。这是因为开发者可以在他们的模块中做很多事情，这些事情需要前端技术。在添加和使用 JavaScript 文件方面，Drupal 有一些特定的方法和技巧，我们将在本章中讨论这些内容。此外，我们还将证明 Drupal 8 在允许我们进行大量的 JavaScript 工作方面是多么强大，而实际上我们甚至不需要编写一行 JavaScript 代码。

因此，在本章中我们将介绍一些内容。

首先，我们将讨论在 Drupal 中编写 JavaScript 的方法。你已经在第四章，“主题”中学习了如何创建库并将它们附加到渲染数组、元素或页面上。基本上，通过使用库，我们可以在需要的时候加载我们的 JavaScript 文件。如果你不记得库是如何工作的，我建议你查看第四章，“主题”中的“资源和库”部分。因为在本章中，我们将从这里继续，并简要讨论那些 JavaScript 文件中实际包含的内容。

一个有用的资源是文档页面([`www.drupal.org/node/172169`](https://www.drupal.org/node/172169))，它列出了 Drupal 8 中 JavaScript 的编码标准，我们应该遵守这些标准。

在本章的第一部分，我们实际上不会编写很多 JavaScript 代码——只是足够让你开始。在第二部分，我们则一个字也不会写。相反，我们将讨论与 Drupal 一起提供的强大 Ajax API，它允许我们构建一些非常动态的功能，这些功能依赖于 JavaScript。为了展示事物是如何工作的，我们将回顾我们在第七章，“你的自定义实体和插件类型”中开始的功能导入，并使用 Ajax 来改进它。

最后，我们还将讨论表单 API 的状态系统，它允许我们以声明的方式使我们的表单元素动态化并依赖于其他元素。同样，我们甚至不需要了解任何 JavaScript 就可以完成相当复杂的客户端行为。

# Drupal 中的 JavaScript

Drupal 8 依赖于许多 JavaScript 库和插件来执行一些前端任务。例如，使用 *Backbone.js* 是 Drupal 相较于之前版本在采用现有库而不是重新发明轮子方面的进步。当然，正如我们已经看到的，无处不在的 *jQuery* 库在 Drupal 8 中仍然被使用。但当然，还有其他库。

另一件我已经提到过，但再次提及是有帮助的，那就是 Drupal 现在不再在所有页面上无谓地加载诸如 jQuery 或其 Ajax 框架之类的东西。例如，许多为匿名用户提供服务且不需要 jQuery 的页面甚至不会加载它。这可以大大提高性能。但这也意味着，当我们定义库以包含我们自己的 JavaScript 文件时，我们必须始终将这些声明为依赖项（如果我们需要它们）。例如，jQuery 是你经常依赖的东西。

# Drupal 行为

当你在 Drupal 中编写 JavaScript 文件时，你需要知道的一个重要概念是行为。但为了理解这一点，让我们先了解一下背景。

当使用 jQuery 编写 JavaScript 代码时，通常的标准是将我们的代码包裹在一个 `ready()` 方法语句中，如下所示：

```php
$(document).ready(function () { 
  // Essentially the entirety of your javascript code. 
}); 
```

这确保了您的代码只有在浏览器加载了整个 **文档对象模型**（**DOM**）之后才会运行。此外，使用 jQuery 来实现这一点在很大程度上有助于跨浏览器兼容性，并允许我们在页面上任何我们想要的位置放置此代码（例如头部或尾部）。

然而，在 Drupal 中，我们有一个不同的解决方案，这个方案在编写与 Drupal 一起工作的 JavaScript（而不仅仅是与 DOM 一起工作）的上下文中更好。这个解决方案就是 Drupal 行为。简而言之，行为是我们声明的、在 DOM 完全加载时被调用的方法，也就是说，当文档准备就绪时。然而，除此之外，它们还会在 Ajax 框架将新数据加载到页面上时被调用。即使在使用 BigPipe 和占位符替换时也是如此。

任何 Drupal 网站都有一个全局的 `Drupal` 对象，用于许多我们现在不会深入讨论的事情。然而，`Drupal.behaviours` 对象是我们声明行为的地方，通常我们想要运行的任何 JavaScript 代码都应该放在行为内部。所以，让我们看看一个例子，这样会更容易理解。

我们想要展示一个动态的 JavaScript 时钟，位于“Hello World”问候语旁边，如果消息不是来自配置而是依赖于一天中的时间。在编写我们功能性的代码时，我们将讨论 Drupal 行为以及它们是如何被使用的。

# 我们的库

为了使我们的 JavaScript 文件能够加载，它需要位于一个库中，并附加到*某个东西*上。正如你在第四章中学习的*主题化*，库文件名为 `hello_world.libraries.yml`，位于我们模块的根目录中：

```php
hello_world_clock: 
  version: 1.x 
  js: 
    js/hello_world_clock.js: {} 
  dependencies: 
    - core/jquery 
    - core/drupal 
    - core/jquery.once  
```

我们只需要一个 JavaScript 文件，它位于我们模块的 `js` 目录中，用于我们的目的。但我们确实有一些依赖项。首先，我们希望加载 jQuery，因为我们将会使用它。其次，我们希望有通用的 Drupal JavaScript 库，它处理许多事情，包括行为。我们将很快讨论最后一个依赖项，那时它会有更多的意义。

如果没有声明这些依赖项，在某些情况下（尤其是对于匿名用户），Drupal 不会在页面上加载它们，我们的 JavaScript 功能将无法工作。

现在，让我们将这个库附加到位于 `HelloWorldSalutation` 服务内部的 salutation 组件上。

在这两行之后：

```php
$time = new \DateTime(); 
$render['#target'] = $this->t('world');  
```

我们可以添加以下内容：

```php
$render['#attached'] = [ 
  'library' => [ 
    'hello_world/hello_world_clock' 
  ] 
];  
```

这对我们来说并不新鲜，但关键是我们只在我们需要显示的组件上附加库，这个动态问候消息依赖于一天中的时间。如果这个消息已被覆盖，我们甚至不想加载这些库，这就是全部。我们可以深入进去创建我们的 `hello_world_clock.js` 文件。

# JavaScript

在 JavaScript 文件中，我们首先需要做的是将我们在这个文件中编写的整个代码包裹在一个**立即调用的函数表达式**（**IIFE**）中。通过这样做，我们保护了我们编写的代码的作用域，使其不会与全局作用域冲突，甚至可以在我们自己的作用域中使用与变量名更常见的全局变量。这就是它的样子：

```php
(function (Drupal, $) {

  "use strict";

  // Our code here. 

}) (Drupal, jQuery);
```

这里最重要的事情是，现在我们可以在函数内部使用美元符号（`$`）作为对全局 jQuery 对象的引用，而不会干扰可能使用相同变量名的其他库。此外，我们还添加了 `use strict` 声明，以确保我们编写的代码语义正确（这也是 Drupal 8 的 JavaScript 编码标准的一部分）。

让我们现在添加我们功能的核心部分，并解释它是如何工作的：

```php
Drupal.behaviors.helloWorldClock = {
  attach: function (context, settings) {
    function ticker() {
      var date = new Date();
      $(context).find('.clock').html(date.toLocaleTimeString());
    }

    var clock = '<div>The time is <span class="clock"></span></div>';

    $(document).find('.salutation').append(clock);

    setInterval(function() {
      ticker();
    }, 1000);
  }
};  
```

首先，我们正在定义一个新的行为，这是一个位于 `Drupal.behaviours` 对象上的对象，并且需要有一个唯一的名字。你可以把一个行为看作是一块功能。我们只需要在这个对象上有一个名为 `attach` 的函数，它接收两个参数：`context`（正在加载的页面或页面的一部分）和 `settings`（包含从 PHP 传递的数据的变量）。

这个函数会在 Drupal 需要附加行为时被调用——`Drupal.attachBehaviors()`。这发生在页面首次加载时（在这种情况下，`context` 是整个 DOM），或者在 Ajax 请求或 BigPipe 替换之后（在这种情况下，`context` 只包含页面新加载的部分）。因此，使用 `context` 而不是整个文档来查找元素有时会更高效（尤其是在 Ajax 请求之后），并且可以防止其他副作用。

在 `attach` 函数内部，我们有创建时钟的逻辑。首先，我们定义一个简单的函数，用于查找具有 `.clock` 类的元素并将当前时间放入其中。你会注意到我们使用了 `context` 来查找元素。接下来，我们自行创建这个元素并将其附加到我们的问候信息元素上。最后，我们每秒设置一个间隔来持续调用我们的 `ticker()` 函数，本质上每秒更新一次时间，从而产生时钟的错觉。这一切都很标准。

请注意，我们通过 JavaScript 打印给用户的字符串没有经过翻译系统，这不是一个好的做法（即使网站不是多语言的）。在后面的章节中，我们将看到我们如何需要处理它。

清除缓存并导航到我们的 `/hello` 页面，我们就可以看到新的时钟已经出现（如果我们没有覆盖问候信息）。所以，我们完成了，对吧？嗯，其实并没有。

如果我们打开浏览器的开发者工具，即控制台，并尝试再次附加行为：

```php
Drupal.attachBehaviors();  
```

我们注意到我们的时钟元素被再次附加（它已经被复制了）。这显然是不对的，因为如果我们有 Ajax 请求，我们就有风险发生这种情况。这就是 `jQuery.once` 发挥作用的地方。

`jQuery.once` 库是 jQuery 的一个插件，它允许我们跟踪并确保我们只执行一次某个操作。实际上，它非常简单易用。我们只需替换这一行：

```php
$(context).find('.salutation').append(clock);  
```

通过这种方式：

```php
$(context).find('.salutation').once('helloWorldClock').append(clock);  
```

所以基本上，在执行实际操作之前，我们使用一个 ID 调用 `.once()` 方法来跟踪。这将确保链中接下来的任何操作只应用于尚未应用过该操作的元素。现在你也看到了为什么我们希望我们的库依赖于 `core/jquery.once`。

有了这个，我们的时钟就准备好了。

# Drupal 设置

我们还可以做另一件强大而常见的事情（我们经常需要做），那就是将值从 PHP 代码传递到 JavaScript 层。在自定义 PHP 应用程序中，这可能会变得很复杂，但 Drupal 有一个强大的 API，可以将 PHP 数组转换为 JavaScript 对象。这些对象可以在传递给行为 `attach()` 函数的 `settings` 对象中找到。

再次，理解这一点最简单的方法是通过一个例子。所以，假设我们想在问候语之后打印一条额外的消息，如果它是下午的话。当然，我们也可以使用 JavaScript 来确定这一点，但到目前为止，这一直是我们的 PHP 代码的责任，所以让我们保持这种方式。因此，我们需要一种方法来告诉我们的 JavaScript 它是下午，如果那样的话，我们可以通过设置一个标志来实现，如下所示：

```php
if ((int) $time->format('G') >= 12 && (int) $time->format('G') < 18) {
  $render['#salutation']['#markup'] = $this->t('Good afternoon');
  $render['#attached']['drupalSettings']['hello_world']['hello_world_clock']['afternoon'] = TRUE;
  return $render;
}

```

新的是 *if 条件* 中的第二行，即我们将其附加到渲染数组中的那一行。然而，在这种情况下，它不是一个库，而是一个大型的多维数组中的 `drupalSettings`。最佳实践是按照以下方式分层命名我们的设置：我们的模块名称 -> 设置所属的功能 -> 设置名称。在 JavaScript 中，这个数组将被转换成一个对象。

要使 `drupalSettings` 生效，我们需要确保已加载 `core/drupalSettings` 库。在我们的例子中，这是因为 `core/drupal` 库将其列为依赖项。

现在我们传递了这个标志（如果需要，它可能更加复杂），我们就可以在 JavaScript 中使用它：

```php
var clock = '<div>The time is <span class="clock"></span></div>'; 
if (settings.hello_world != undefined && settings.hello_world.hello_world_clock.afternoon != undefined) { 
  clock += 'Are you having a nice day?'; 
}  
```

就这样了。我们成功地轻松地将值从 PHP 传递到 JavaScript 并在客户端逻辑中使用它们。

# Ajax API

现在您已经准备好编写您应用程序所需的任何 JavaScript 代码，并且能够将其与 Drupal 后端 API 集成，让我们来看看 Ajax 框架。在不编写任何 JavaScript 代码的情况下，我们可以在客户端做很多事情。

Drupal Ajax API 是一个强大的系统，它允许我们通过 PHP 定义客户端交互。我们最常使用 Ajax 与表单交互——触发某些动作，改变 DOM 而无需重新加载页面。我们将通过扩展我们在 第七章，*您的自定义实体和插件类型* 中构建的导入功能来演示这一切是如何工作的。在深入探讨之前，让我们快速看一下 Drupal 8 中 Ajax 的简单用法。

# Ajax 链接

与 Drupal 的 Ajax API 交互的最简单方法是向任何链接添加 `use-ajax` 类。这将导致链接向链接的路径发送 Ajax 请求，而不是将浏览器移动到那里。类似的事情可以使用表单的提交按钮通过 `use-ajax-submit` 类来完成。这使得表单通过 Ajax 提交到表单的 action 中定义的路径。

然而，最重要的是我们在流程的另一端所做的工作。点击一个触发 Ajax 请求的链接，如果我们没有相应地处理该请求，那么什么也不会发生。我们必须做的是返回一个包含一些 jQuery *命令* 的 `AjaxResponse` 对象，这些命令指导浏览器对 DOM 所需进行的更改。所以，让我们看看一个例子。

记得在第二章，*创建你的第一个模块*中，当我们创建第一个仅渲染服务中的问候消息的块时？它没有使用我们在第四章，*主题化*中创建的主题钩子，而是简单地委托给`HelloWorldSalutation`服务的`getSalutation()`方法。假设我们想在消息后添加一个可以点击的链接，并且可以完全隐藏块。我们需要采取几个简单的步骤来实现这一点。

首先，我们需要修改块中的`build()`方法，使其看起来像这样：

```php
/** 
* {@inheritdoc} 
*/ 
public function build() { 
$build = []; 

$build[] = [ 
  '#theme' => 'container', 
  '#children' => [ 
    '#markup' => $this->salutation->getSalutation(), 
  ] 
]; 

$url = Url::fromRoute('hello_world.hide_block'); 
$url->setOption('attributes', ['class' => 'use-ajax']); 
$build[] = [ 
  '#type' => 'link', 
  '#url' => $url, 
  '#title' => $this->t('Remove'), 
]; 

return $build; 
}  
```

以及新的`use`声明：

```php
use Drupal\Core\Url;
```

我们首先做的事情是将我们的原始基于`#markup`的简单数组包装到 Drupal 核心的`container`主题钩子中，这样它就会用一些 div 将其包裹起来，我们就不必创建自己的主题钩子。毕竟，我们在这里做的是概念验证工作。接下来，在消息下方，我们打印出一个指向我们必须要定义的新路由的链接。正如我们之前讨论的，我们给这个链接添加了`use-ajax`类。你会注意到，我们可以直接将属性（参考第四章，*主题化*，了解更多关于这些属性的信息）添加到`Url`对象中，它们将被添加到渲染的链接元素中。

第二，我们需要定义这个新路由。这再简单不过了：

```php
hello_world.hide_block: 
  path: '/hide-block' 
  defaults: 
    _controller: '\Drupal\hello_world\Controller\HelloWorldController::hideBlock' 
  requirements: 
    _permission: 'access content'  
```

我们将其映射到我们一直在使用的相同控制器类的新方法上，并允许所有用户访问它。

第三（也是最后），我们需要定义控制器方法：

```php
/** 
 * Route callback for hiding the Salutation block. 
 * Only works for Ajax calls. 
 * 
 * @param \Symfony\Component\HttpFoundation\Request $request 
 * 
 * @return \Drupal\Core\Ajax\AjaxResponse 
 */ 
public function hideBlock(Request $request) { 
  if (!$request->isXmlHttpRequest()) { 
    throw new NotFoundHttpException(); 
  } 

  $response = new AjaxResponse(); 
  $command = new RemoveCommand('.block-hello-world'); 
  $response->addCommand($command); 
  return $response; 
}  
```

以及顶部的新`use`声明：

```php
use Drupal\Core\Ajax\AjaxResponse; 
use Drupal\Core\Ajax\RemoveCommand; 
use Symfony\Component\HttpFoundation\Request; 
use Symfony\Component\HttpKernel\Exception\NotFoundHttpException;  
```

你首先会注意到这个方法的`$request`参数，你可能想知道它从哪里来。Drupal 将当前请求对象传递给任何仅通过该类类型提示参数的控制器方法。因此，我们不需要将其注入到我们的控制器中。我们需要它的原因是我们可以检查对这条路由的请求是否是通过 Ajax 发起的。因为如果不是，我们不想处理它。也就是说，我们抛出一个`NotFoundHttpException`，这会导致常规的 404 错误。

然后是关于 Ajax API 的有趣内容，即构建一个充满命令的`AjaxResponse`，这些命令将返回到浏览器。在我们的例子中，只有一个命令，它指示浏览器在匹配传递给它的选择器的元素上运行 jQuery 的`remove()`方法。在我们的情况下，这是块包装器的类。有了这个，我们的功能就到位了。我们可以清除缓存，现在块应该会打印出一个通过 Ajax 删除块的链接。

你可能会想：为什么我们需要回到服务器去完成本可以在客户端独立完成的工作？答案是——实际上我们不需要。然而，这作为一个很好的例子，说明了 Ajax 响应是如何工作的。我鼓励你查看 Ajax API 的文档页面([`api.drupal.org/api/drupal/core!core.api.php/group/ajax/8.6.x`](https://api.drupal.org/api/drupal/core!core.api.php/group/ajax/8.6.x) )，在那里你可以找到所有可用命令的列表。例如，我们可以使用`ReplaceCommand`来用从服务器返回的另一个块替换块，或者使用`HtmlCommand`在页面上插入一些数据，甚至使用`AlertCommand`触发一个带有来自服务器的数据的 JavaScript 警告。酷的地方在于响应可以处理多个命令，所以我们不受仅使用一个的限制。

# 表单中的 Ajax

在 Drupal 中，Ajax 最常用的用途是通过表单 API，我们可以轻松地在服务器和客户端之间创建动态交互。为了演示这是如何工作的，我们将通过一个示例来展示。这将是对我们在第七章，“你自己的自定义实体和插件类型”中创建的导入器配置实体表单的重构。

如果你还记得，我们说过将某些配置值绑定到通用实体是没有意义的，因为导入插件可能不同。我们编写的第一个导入器从远程 URL 加载 JSON 文件。因此，可以合理地认为 URL 的配置值绑定到插件而不是配置实体（即使后者实际上存储它）。因为如果我们想创建一个 CSV 导入器，例如，我们不需要 URL。所以，让我们重构我们的工作来实现这一点。

这是我们需要采取的步骤的概述，以进行此重构：

1.  导入器插件需要提供它们自己的配置表单元素。

1.  导入器配置表单需要根据所选的插件读取这些元素（这就是 Ajax API 发挥作用的地方）。

1.  我们需要更改特定于插件的值的数据存储和配置架构。

让我们从为`ImporterInterface`插件类型提供一个新方法开始：

```php
/**
 * Returns the form array for configuring this plugin.
 *
 * @param \Drupal\products\Entity\ImporterInterface $importer
 *
 * @return array
 */
public function getConfigurationForm(\Drupal\products\Entity\ImporterInterface $importer);
```

这负责获取此插件所需的表单元素。作为一个参数，它接收导入器配置实体，可以检查默认值。

接下来，在配置实体的`ImporterInterface`上，我们需要移除`getUrl()`方法（因为这是针对`JsonImporter`插件的特定方法）并替换为用于检索与实体选择的插件相关的所有配置值的通用方法：

```php
/** 
 * Returns the configuration specific to the chosen plugin. 
 * 
 * @return array 
 */ 
public function getPluginConfiguration();  
```

当然，在导入器实体类中，我们也反映了这一变化（通过替换`$url`属性）：

```php
/** 
 * The configuration specific to the plugin. 
 * 
 * @var array 
 */ 
protected $plugin_configuration; 
```

实际的 getter 方法，与接口保持一致：

```php
/** 
 * {@inheritdoc} 
 */ 
public function getPluginConfiguration() { 
  return $this->plugin_configuration; 
}  
```

到目前为止，一切顺利，没有复杂的事情发生。我们正在用通用配置值替换特定于插件的配置值，其中将存储特定于所选插件的值。然而，由于我们的实体类型不再有`$url`字段，而是有一个`$plugin_configuration`字段，因此我们还需要调整注解中的`config_export`键以反映这一变化：

```php
*   config_export = { 
*     "id", 
*     "label", 
*     "plugin", 
*     "update_existing", 
*     "source", 
*     "bundle", 
*     "plugin_configuration" 
*   } 
```

现在，让我们转向`ImporterForm`并对其进行所有调整。但在我们这样做之前，让我们将`url`字段的表单元素移动到`JsonImporter`中，在那里我们必须实现新的`getConfigurationForm()`方法：

```php
/** 
 * {@inheritdoc} 
 */ 
public function getConfigurationForm(\Drupal\products\Entity\ImporterInterface $importer) { 
  $form = []; 
  $config = $importer->getPluginConfiguration(); 
  $form['url'] = [ 
    '#type' => 'url', 
    '#default_value' => isset($config['url']) ? $config['url'] : '', 
    '#title' => $this->t('Url'), 
    '#description' => $this->t('The URL to the import resource'), 
    '#required' => TRUE, 
  ]; 
  return $form; 
}  
```

你会注意到在获取默认值时有一些差异。我们不再在配置实体上调用已删除的`getUrl()`方法，而是使用新的`getPluginConfiguration()`方法并在结果数组内部进行检查。另外，由于我们使用`$this->t()`方法来确保字符串的翻译，我们还应该使用`StringTranslationTrait`（它可以放在父基类中，因为它是一个特质）：

```php
use StringTranslationTrait; 
```

让我们不要忘记，我们实际上在导入时使用了 URL，因此我们还需要对`getData()`方法做一些调整：

```php
/**
 * Loads the product data from the remote URL.
 *
 * @return \stdClass
 */
private function getData() {
  /** @var ImporterInterface $importer_config */
  $importer_config = $this->configuration['config'];
  $config = $importer_config->getPluginConfiguration();
  $url = isset($config['url']) ? $config['url'] : NULL;
  if (!$url) {
    return NULL;
  }
  $request = $this->httpClient->get($url);
  $string = $request->getBody();
  return json_decode($string);
}
```

在此基础上，我们可以继续调整我们的`ImporterForm`（其中我们不再有 URL 字段的表单元素）。

我们需要做的主要有两件事：

+   将插件选择元素暴露给 Ajax，即当用户做出选择时触发 Ajax 请求

+   根据选择的插件添加额外的元素到表单中

新的`plugin`元素看起来是这样的：

```php
$form['plugin'] = [ 
  '#type' => 'select', 
  '#title' => $this->t('Plugin'), 
  '#default_value' => $importer->getPluginId(), 
  '#options' => $options, 
  '#description' => $this->t('The plugin to be used with this importer.'), 
  '#required' => TRUE, 
  '#empty_option' => $this->t('Please select a plugin'), 
  '#ajax' => array( 
    'callback' => [$this, 'pluginConfigAjaxCallback'], 
    'wrapper' => 'plugin-configuration-wrapper' 
  ), 
];  
```

有两个显著的变化：我们添加了一个`#empty_option`键（用于在用户未做出选择时显示的选项）和一个`#ajax`键（我们将在下面更详细地讨论）。

我们所做的是相当简单的。我们声明了一个回调方法，当用户更改此表单元素时将被触发，并声明了应该用 Ajax 回调的结果替换的元素的 HTML ID。在后者（这是同一类的一个简单方法）中，我们只需做以下操作：

```php
/** 
 * Ajax callback for the plugin configuration form elements. 
 * 
 * @param $form 
 * @param \Drupal\Core\Form\FormStateInterface $form_state 
 * 
 * @return array 
 */ 
public function pluginConfigAjaxCallback($form, FormStateInterface $form_state) { 
  return $form['plugin_configuration']; 
}  
```

我们返回一个表单元素（我们仍然需要定义）。在这里的一个重要教训是，表单中的 Ajax 响应可以返回内容（以渲染数组或字符串的形式），这将用于替换 Ajax 声明中`wrapper`键指定的 ID 找到的 HTML。或者，也可以返回一个充满命令的`AjaxResponse`以执行更复杂的事情，就像我们在上一节中看到的那样。

在我们查看这个新的`plugin_configuration`表单元素之前，让我们看看可以在`#ajax`数组内部使用的其他一些选项：

+   `method`：这表示与`wrapper`元素交互时要使用的 jQuery 方法（如果指定）。默认是`replaceWith()`，但您也可以使用`append()`、`html()`等。

+   `event`: 这表示应该使用哪个事件来触发 Ajax 调用。默认情况下，相关的表单元素会做出这个决定。例如，当在选择元素中选择一个选项或向文本字段中输入内容时。

+   `progress`: 这定义了在 Ajax 请求进行时使用的指示器。

+   `url`: 如果未指定`callback`，则触发 Ajax 请求的 URL。通常，使用后者更强大，因为整个`$form`和`$form_state`作为参数传递，并可用于处理。

我建议你查看文档页面([`api.drupal.org/api/drupal/core%21core.api.php/group/ajax/8.7.x`](https://api.drupal.org/api/drupal/core%21core.api.php/group/ajax/8.7.x))，以获取有关这些选项和其他可用选项的更多信息。

在处理完这些之后，我们可以回到我们的表单定义，并在`plugin`元素之后添加我们缺失的部分：

```php
$form['plugin_configuration'] = [ 
  '#type' => 'hidden', 
  '#prefix' => '<div id="plugin-configuration-wrapper">', 
  '#suffix' => '</div>', 
]; 

$plugin_id = NULL; 
if ($importer->getPluginId()) { 
  $plugin_id = $importer->getPluginId(); 
} 
if ($form_state->getValue('plugin') && $plugin_id !== $form_state->getValue('plugin')) { 
  $plugin_id = $form_state->getValue('plugin'); 
} 

if ($plugin_id) { 
  /** @var \Drupal\products\Plugin\ImporterInterface $plugin */ 
  $plugin = $this->importerManager->createInstance($plugin_id, ['config' => $importer]); 
  $form['plugin_configuration']['#type'] = 'details'; 
  $form['plugin_configuration']['#tree'] = TRUE; 
  $form['plugin_configuration']['#open'] = TRUE; 
  $form['plugin_configuration']['#title'] = $this->t('Plugin configuration for <em>@plugin</em>', ['@plugin' => $plugin->getPluginDefinition()['label']]); 
  $form['plugin_configuration']['plugin'] = $plugin->getConfigurationForm($importer); 
}  
```

首先，我们将`plugin_configuration`表单元素定义为`hidden`类型。这意味着当页面首次加载时，用户将看不到它。然而，我们确实使用了`#prefix`和`#suffix`选项（与 Drupal 表单 API 的常见做法）来用具有我们指示的 ID 的 div 包装这个元素，作为我们的 Ajax 声明的包装器。所以，目标是每次进行 Ajax 请求时（即每次选择插件时）替换这个元素。

接下来，我们尝试获取所选插件的 ID。首先，如果我们在查看编辑表单，我们会从配置实体中加载它。然而，我们也会检查表单状态，看看是否已经选择了一个（并且与实体中的不同）。如果你想知道我们如何在表单状态中拥有插件，答案是：在用户选择插件后触发 Ajax 调用后，表单会被重建。现在，我们可以看到表单状态中的内容，并检索所选的插件 ID。

更重要的是，如果我们获得了插件 ID，我们可以完全更改`plugin_configuration`元素，这反过来又会被 Ajax 回调返回，用于替换我们的包装器。所以总结一下：

1.  页面首次加载（在新的表单中）。元素被隐藏。

1.  用户选择一个插件并触发 Ajax 请求，这会重建表单。

1.  当表单重建时，我们会检查所选插件，并修改`plugin_configuration`元素以反映所选插件。

1.  Ajax 响应会用新的、可能已更改的元素替换旧的元素。

新的`plugin_configuration`元素变成了一个`details`元素（一个可折叠的容器，用于多个元素），默认打开，并且有一个名为`plugin`的键，我们将所有来自插件的元素添加到这个键上。此外，我们使用`#tree`属性来指示，当表单提交时，元素的值会被发送并存储在一个反映表单元素的树中（基本上是一个多维数组）。否则，提交的表单状态值会被扁平化，我们就会失去它们与`plugin_configuration`元素（这也是我们想要存储数据的导入器配置实体字段名称）的关联。

我们几乎完成了。我们可以创建一个导入器实体，并且当我们选择 JSON 导入器时，包含 URL 字段的新的字段集应该会显示在下面。但我们仍然有一个问题。如果我们保存表单，URL 值将存储在`plugin_configuration`字段中，以`plugin`为键的数组内。因此，我们需要稍微整理一下，我们可以在`save()`方法中这样做。

在保存实体之前，我们可以这样做：

```php
$importer->set('plugin_configuration', $importer->getPluginConfiguration()['plugin']); 
```

因此，我们基本上将值向上移动一个数组，从数组中移除多余的`plugin`级别（这只是为了整齐地组织表单树）。

这样，我们就完成了。嗯，实际上还没有，因为我们仍然需要处理配置模式方面。是的，还记得第六章数据建模和存储和第七章自定义实体和插件类型中的那些吗？我们现在将看到我们如何处理自己的动态配置模式，类似于我们在第九章自定义字段中处理字段插件所需的方式。但为什么我们需要一个动态配置模式呢？

在这次重构之前，我们知道导入器配置实体的确切字段，并且可以轻松地为每个字段声明模式（就像我们做的那样）。然而，现在插件可以带有它们自己的单个字段，因此我们需要确保它们可以为相应数据提供自己的模式定义。那么我们如何做到这一点呢？

首先，在我们的`importer.schema.yml`文件中，我们需要移除`url`字段模式定义，因为它已经不存在了。然而，我们用我们创建的新字段替换它，即来自插件的`plugin_configuration`值数组：

```php
plugin_configuration: 
  type: products.importer.plugin.[%parent.plugin] 
```

这里事情变得有趣。我们不知道里面会有哪些字段，所以我们引用了另一个类型（我们自己的）。此外，类型的名称是动态的。我们有一个前缀（`products.importer.plugin.`）后面跟着由父插件字段（*主要配置实体*）的值给出的变量名。所以基本上，如果一个给定的配置实体使用了`json`插件，模式定义的类型将是`products.importer.plugin.json`。因此，现在，创建新插件的人也有责任为其自己的字段提供自己的模式定义（就像我们在第九章，*自定义字段*）中定义字段插件时做的那样）。

但在那之前，我们需要定义我们创建的新类型：

```php
products.importer.plugin.*: 
  type: mapping 
  label: 'Plugin configuration' 
```

所以，本质上，我们的新类型扩展自`mapping`并有一个简单的标签。当然，它适用于所有以该名称开头的东西（这就是我们之前遇到的通配符的原因）。

现在，我们可以为我们的单个`json`导入插件添加模式定义：

```php
products.importer.plugin.json: 
  type: mapping 
  label: Plugin configuration for the Json importer plugin 
  mapping: 
    url: 
      type: uri 
      label: Uri  
```

如您所见，我们现在有了`products.importer.plugin`类型的第一个实例，它包含`url`字段，位于配置实体的`plugin_configuration`字段中——反映了一个简单的数组层次结构。

但这个动态声明的目的是，其他定义新插件的模块现在也可以定义自己的`products.importer.plugin.*`模式定义的实例来映射它们自己的字段。不再是由配置实体（模式）来“猜测”每个插件正在使用哪些字段类型。

这样，我们的重构就完成了。Drupal 清楚地知道配置实体正在保存的数据类型，即使它部分与外部输入（选定的插件）相关。这意味着我们可以创建（如果我们想的话）另一个使用 CSV 文件的产品数据的导入插件。但我们将如何在后面的章节中讨论文件处理时看到如何做。

# 状态（表单）系统

在本章中，我们将最后探讨 Form API 的状态系统（不要与我们在第六章，*数据建模和存储*）混淆）。这允许我们根据用户与表单的交互动态地定义我们的表单元素。它不使用 Ajax，而是依赖于 JavaScript 来处理操作。这是另一个客户端行为的绝佳例子，我们不需要写一行 JavaScript。所以，让我们看看这是什么。

`#states`是我们可以添加到表单元素中的简单属性，它们根据其他元素的*状态*来改变它们。理解这个概念最好的方式是通过一些例子。想象这两个表单元素：

```php
$form['kids'] = [ 
  '#type' => 'checkbox', 
  '#title' => $this->t('Do you have kids?'), 
]; 

$form['kid_number'] = [ 
  '#type' => 'textfield', 
  '#title' => $this->t('How many kids do you have?'), 
]; 
```

在第一种情况下，我们询问用户是否有孩子（使用简单的复选框），而在第二种情况下，我们询问他们有多少孩子。但为什么如果用户没有孩子，他们实际上应该看到第二个元素呢？这就是`#states`属性发挥作用的地方，它的作用是根据另一个元素的*状态*来操纵一个元素。所以，我们可以这样：

```php
$form['kid_number'] = [ 
  '#type' => 'textfield', 
  '#title' => $this->t('How many kids do you have?'), 
  '#states' => [ 
    'visible' => [ 
      'input[name="kids"]' => ['checked' => TRUE], 
    ], 
  ], 
]; 
```

现在，指定孩子数量的元素只有在`kid`元素的*状态*被选中时才会可见。

`#states`属性是一个数组，其键是如果条件内部满足，需要应用到当前元素的实际情况。条件可以变化，但它们都依赖于一个 CSS 选择器（在我们的例子中是`input[name="kids"]`匹配另一个元素）。

我们的例子也可以用这种逆向逻辑来写：

```php
'#states' => array( 
  'invisible' => array( 
    'input[name="kids"]' => array('checked' => FALSE), 
  ), 
),  
```

除了`visible`和`invisible`之外，以下*状态*也可以应用到表单元素上：`enabled`、`disabled`、`required`、`optional`、`checked`、`unchecked`、`expanded`和`collapsed`。至于可以“触发”这些*状态*的条件，我们可以有以下几种（除了我们之前看到的`checked`）：`empty`、`filled`、`unchecked`、`expanded`、`collapsed`和`value`。

例如，我们甚至可以根据用户在另一个元素上选择的值来控制一个元素的*状态*。结合这些可能性可以极大地改善我们的表单，在用户体验、整理甚至构建逻辑表单树方面。

# 摘要

在本章中，我们关注了客户端，并讨论了 Drupal 8 中的 JavaScript 和客户端功能。我们首先讨论了在 Drupal 环境中编写 JavaScript 时需要采取的方法。我们学习了行为、为什么它们很重要以及如何使用它们。我们还看到了如何从服务器（Drupal）传递数据到客户端，并在 JavaScript 中使用它。

很有趣的是，我们随后将本章的其余部分改为不允许使用 JavaScript 的策略。我们这样做是为了证明 Drupal Ajax API 的强大功能，即使我们不是能够编写 JavaScript 代码的前端开发者，我们也能使用它来执行复杂的客户端到服务器的交互。为了展示这个 API，我们首先看了如何将简单的链接转换为 Ajax 请求。接着，我们对之前的产品导入功能进行了重要的重构，该功能依赖于 Ajax 来使导入配置实体表单动态化（依赖于所选的插件）。别忘了另一个信息亮点——动态配置模式，它允许我们将配置实体数据定义与其所选插件的数据定义解耦。

最后，我们通过查看表单 API 的 States 系统来结束讨论，该系统允许我们将客户端操作声明性地编码到我们的表单元素上，本质上使它们依赖于用户的表单交互。

在下一章中，我们将讨论国际化与翻译，以确保我们的应用程序可以在全球任何地方使用。
