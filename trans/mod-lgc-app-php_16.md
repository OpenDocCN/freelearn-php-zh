# 第十六章。添加依赖注入容器

我们已经完成了现代化过程的最后一步。我们将通过将剩余逻辑移入依赖注入容器来删除页面脚本的最后痕迹。容器将负责协调应用程序中的所有对象创建活动。这样做，我们将再次修改我们的前端控制器，并开始添加指向控制器类而不是文件路径的路由。

### 注意

在现代化过程的最后一步中，最好安装 PHP 5.3 或更高版本。这是因为我们需要闭包来实现应用程序逻辑的关键部分。如果我们没有访问 PHP 5.3，还有一种不太可行但仍然可行的选项来实现依赖注入容器。我们将在本章的“常见问题”中解决这种情况。

# 什么是依赖注入容器？

依赖注入作为一种技术，是我们从本书的早期就开始练习的。重申一下，依赖注入的理念是将依赖从外部推入对象。这与通过 new 关键字在类内部创建依赖对象，或者通过`globals`关键字从当前范围外部引入依赖的做法相反。

### 注意

要了解控制反转的概述以及具体的依赖注入，请阅读 Fowler 在[`martinfowler.com/articles/injection.html`](http://martinfowler.com/articles/injection.html)上关于容器的文章。

为了完成我们的依赖注入活动，我们一直在页面脚本中手动创建必要的对象。对于任何需要依赖的对象，我们首先创建了该依赖，然后创建了依赖它的对象并传入依赖。这个创建过程有时会非常复杂，比如当依赖有依赖时。无论复杂程度和深度如何，目前做法的逻辑都嵌入在页面脚本中。

依赖注入容器的理念是将所有对象创建逻辑放在一个地方，这样我们就不再需要使用页面脚本来设置我们的对象。我们可以将每个对象创建逻辑放在容器中，使用一个唯一的名称，称为服务。

然后我们可以告诉容器返回任何定义的服务对象的新实例。或者，我们可以告诉容器创建并返回该服务对象的共享实例，这样每次获取它时，它都是同一个实例。精心组合容器服务的新实例和共享实例将允许我们简化依赖创建逻辑。

### 注意

在任何时候，我们都不会将容器传递给需要依赖的任何对象。这样做将使用一种称为服务定位器的模式。我们避免服务定位器活动，因为这样做违反了范围。当容器在一个对象内部，并且该对象使用它来检索依赖时，我们只是离我们开始的地方一步之遥；也就是说，使用`global`关键字。因此，我们不会传递容器 -- 它完全留在创建对象的范围之外。

PHP 领域中有许多不同的容器实现，每种实现都有其自身的优势和劣势。为了使事情与我们的现代化过程相适应，我们将使用*Mlaphp\Di*。这是一个精简的容器实现，非常适合我们的过渡需求。

## 添加 DI 容器

一般来说，添加 DI 容器的过程如下：

1.  添加一个新的`services.php`包含文件来创建容器并管理其服务。

1.  在容器中定义一个`router`服务。

1.  修改前端控制器以包含`services.php`文件并使用`router`服务，然后对应用程序进行抽查。

1.  从每个页面脚本中提取创建逻辑到容器中：

1.  在容器中为页面脚本控制器类命名一个服务。

1.  将页面脚本中的逻辑复制到容器服务中。根据需要重命名变量以使用 DI 容器属性。

1.  将页面 URL 路径路由到容器服务名称（即控制器名称）。

1.  检查并提交更改。

1.  继续，直到所有页面脚本都已提取到容器中。

1.  删除空的`pages/`目录，提交，推送，并通知 QA。

## 添加 DI 容器包含文件

为了防止我们现有的设置文件变得更大，我们将引入一个新的`services.php`设置文件。是的，这意味着在前端控制器中添加另一个`include`，但是如果我们一直很勤奋，我们的应用程序中几乎没有剩余的`include`。这个`include`的重要性将会很小。

首先，我们需要选择一个适当的位置放置文件。最好是与我们已有的任何其他设置文件一起，可能是在现有的`includes/`目录中。

然后我们创建了以下行的文件。（随着我们的继续，我们将在这个文件中添加更多内容。）因为这个文件将作为我们设置文件的最后一个加载，我们可以假设自动加载将会生效，所以没有必要加载`Di`类文件：

```php
**includes/services.php**
1 <?php
2 $di = new \Mlaphp\Di($GLOBALS);
3 ?>
```

结果是新的`$di`实例加载了所有现有的全局变量值。这些值作为容器的属性被保留。例如，如果我们的设置文件创建了一个`$db_user`变量，现在我们还可以通过`$di->db_user`访问该值。这些是副本，而不是引用，因此对一个的更改不会影响另一个。

### 注意

**为什么我们保留现有变量作为属性？**

目前，我们的页面脚本直接访问全局变量来进行创建工作。然而，在后续步骤中，创建逻辑将不再在全局范围内。它将在 DI 容器中。因此，我们将 DI 容器填充了原本可以使用的变量的副本。

### 添加一个路由器服务

现在我们已经有了一个 DI 容器，让我们添加我们的第一个服务。

回想一下，DI 容器的目的是为我们创建对象。目前，前端控制器创建了一个*Router*对象，因此我们将在容器中添加一个`router`服务。（在下一步中，我们将让前端控制器使用这个服务，而不是自己创建一个*Router*。）

在`services.php`文件中，添加以下行：

```php
**includes/services.php**
1 <?php
2 // set a container service for the router
3 $di->set('router', function () use ($di) {
4 $router = new \Mlaphp\Router('/path/to/app/pages');
5 $router->setRoutes(array());
6 return $router;
7 });
8 ?>
```

让我们稍微检查一下服务定义。

+   服务名称是`router`。我们将用全小写名称来命名那些预期作为共享实例创建的服务对象，并使用完全限定的类名来命名那些预期每次创建新实例的服务对象。因此，在这种情况下，我们的意图是通过容器只提供一个共享的`router`。（这是一个约定，而不是容器强制执行的规则。）

+   服务定义是一个可调用的。在这种情况下，它是一个闭包。闭包不接收任何参数，但它确实使用了当前作用域中的`$di`对象。这使得定义代码可以在构建服务对象时访问容器属性和其他容器服务。

+   我们创建并返回由服务名称表示的对象。我们不需要检查对象是否已经存在于容器中；如果我们要求一个共享实例，容器内部将为我们执行这项工作。

有了这段代码，容器现在知道如何创建一个`router`服务。这是一个懒加载的代码，只有当我们调用`$di->newInstance()`（获取服务对象的新实例）或者`$di->get()`（获取服务对象的共享实例）时才会执行。

### 修改前端控制器

现在我们有了一个 DI 容器和一个`router`服务定义，我们修改前端控制器来加载容器并使用`router`服务。

```php
docroot/front.php
1 <?php
2 require dirname(__DIR__) . '/includes/setup.php';
3 require dirname(__DIR__) . '/includes/services.php';
4
5 // get the shared router service
6 $router = $di->get('router');
7
8 // match against the url path
9 $path = parse_url($_SERVER['REQUEST_URI'], PHP_URL_PATH);
10 $route = $router->match($path);
11
12 // container service, or page script?
13 if ($di->has($route)) {
14 // create a new $controller instance
15 $controller = $di->newInstance($route);
16 } else {
17 // require the page script
18 require $route;
19 }
20
21 // invoke the controller and send the response
22 $response = $controller->__invoke();
23 $response->send();
24 ?>
```

我们从之前的实现中做了以下更改：

+   我们在设置包含的最后添加了对`services.php`容器文件的`require`。

+   我们不直接创建*Router*对象，而是从`$di`容器中`get()`一个共享实例的`router`服务对象。

+   我们已经在调度逻辑上做了一些更改。在我们从`$router`获取`$route`之后，我们检查看看`$di`容器是否`has()`匹配的服务。如果是，它将`$route`视为新`$controller`实例的服务名称；否则，它将`$route`视为在`pages/`中创建`$controller`的文件。无论哪种方式，代码都会调用控制器并发送响应。

在这些更改之后，我们会抽查应用程序，以确保新的`router`服务能够正常工作。如果不能正常工作，我们会撤消并重做到这一点，直到应用程序像以前一样正常工作。

一旦应用程序正常工作，我们可能希望提交我们的更改。这样，如果将来的更改出现问题，我们就有一个已知工作状态可以恢复。

### 将页面脚本提取到服务中

现在是现代化我们的遗留应用程序的最后一步。我们将逐个删除页面脚本，并将它们的逻辑放入容器中。

#### 创建一个容器服务

选择任何页面脚本，并确定它使用哪个类来创建其`$controller`实例。然后，在 DI 容器中，为该类名创建一个空的服务定义。

例如，如果我们有这个页面脚本：

```php
**pages/articles.php**
1 <?php
2 $db = new Database($db_host, $db_user, $db_pass);
3 $articles_gateway = new ArticlesGateway($db);
4 $users_gateway = new UsersGateway($db);
5 $article_transactions = new ArticleTransactions(
6 $articles_gateway,
7 $users_gateway
8 );
9 $response = new \Mlaphp\Response('/path/to/app/views');
10 $controller = new \Controller\ArticlesPage(
11 $request,
12 $response,
13 $user,
14 $article_transactions
15 );
16 ?>
```

我们实例化的控制器类是`Controller\ArticlesPage`。在我们的`services.php`文件中，我们创建了一个空的服务定义：

```php
**includes/services.php**
1 <?php
2 $di->set('Controller\ArticlesPage', function () use ($di) {
3 });
4 ?>
```

接下来，我们将页面脚本设置逻辑移到服务定义中。当我们这样做时，我们应该注意到我们期望从全局范围获得的任何变量，并使用`$di->`前缀来引用适当的容器属性。（回想一下，这些是在`services.php`文件的早期从`$GLOBALS`加载的。）我们还在定义的最后返回控制器实例。

完成后，服务定义将看起来像这样：

```php
**includes/services.php**
1 <?php
2 $di->set('Controller\ArticlesPage', function () use ($di) {
3 // replace `$variables` with `$di->` properties
4 $db = new Database($di->db_host, $di->db_user, $di->db_pass);
5 // create dependencies
6 $articles_gateway = new ArticlesGateway($db);
7 $users_gateway = new UsersGateway($db);
8 $article_transactions = new ArticleTransactions(
9 $articles_gateway,
10 $users_gateway
11 );
12 $response = new \Mlaphp\Response('/path/to/app/views');
13 // return the new instance
14 return new \Controller\ArticlesPage(
15 $request,
16 $response,
17 $user,
18 $article_transactions
19 );
20 });
21 ?>
```

一旦我们将逻辑复制到容器中，我们就从`pages/`中删除原始页面脚本文件。

#### 将 URL 路径路由到容器服务

现在我们已经删除了页面脚本，转而使用容器服务，我们需要确保*Router*指向容器服务，而不是现在缺失的页面脚本。我们通过向`setRoutes()`方法参数添加一个数组元素来实现这一点，其中键是 URL 路径，值是服务名称。

例如，如果 URL 路径是`/articles.php`，我们的新容器服务命名为`Controller\ArticlesPage`，我们将修改我们的`router`服务如下：

```php
**includes/services.php**
1 <?php
2 // ...
3 $di->set('router', function () use ($di) {
4 $router = new \Mlaphp\Router($di->pages_dir);
5 $router->setRoutes(array(
6 // add a route that points to a container service name
7 '/articles.php' => 'Controller\ArticlesPage',
8 ));
9 return $router;
10 });
11 ?>
```

#### 抽查和提交

最后，我们检查页面脚本转换为容器服务是否按我们的预期工作。我们通过浏览或以其他方式调用 URL 路径来抽查旧页面脚本。如果它起作用，那么我们知道容器服务已成功取代现在已删除的页面脚本。

如果不是，我们需要撤消并重做我们的更改，看看哪里出了问题。我在这里看到的最常见的错误是：

+   未能将页面脚本中的`$var`变量替换为服务定义中的`$di->var`属性

+   未能从服务定义中返回对象

+   控制器服务名称与映射的路由值之间的不匹配

一旦我们确定应用程序将 URL 路由到新的容器服务，并且服务正常工作，我们就提交我们的更改。

### 做...直到

我们继续下一个页面脚本，并重新开始这个过程。当所有页面脚本都转换为容器服务然后被删除时，我们就完成了。

### 删除 pages/，提交，推送，通知 QA

在我们将所有页面脚本提取到 DI 容器之后，`pages/`目录应该是空的。我们现在可以安全地将其删除。

有了这个，我们提交我们的工作，推送到共同的存储库，并通知 QA 我们有新的更改需要他们审查。

## 常见问题

### 我们如何完善我们的服务定义？

当我们完成将对象创建逻辑提取到容器后，每个服务定义可能会变得相当长，而且可能重复。我们可以通过进一步将对象创建逻辑的每个部分提取到自己的服务中来减少重复并完善服务定义，使其变得简短而简洁。

例如，如果我们有几个服务使用*Request*对象，我们可以将对象创建逻辑提取到自己的服务中，然后在其他服务中引用该服务。我们可以命名它以显示我们的意图，即它可以被用作共享服务（`request`）或新实例（`Mlaphp\Request`）。其他服务可以使用`get()`或`newInstance()`而不是在内部创建请求。

考虑到我们之前的`Controller\ArticlesPage`服务，我们可以将其拆分为几个可重用的服务，如下所示：

```php
includes/services.php
1 <?php
2 // ...
3
4 $di->set('request', function () use ($di) {
5 return new \Mlaphp\Request($GLOBALS);
6 });
7
8 $di->set('response', function () use ($di) {
9 return new \Mlaphp\Response('/path/to/app/views');
10 });
11
12 $di->set('database', function () use ($di) {
13 return new \Database(
14 $di->db_host,
15 $di->db_user,
16 $di->db_pass
17 );
18 });
19
20 $di->set('Domain\Articles\ArticlesGateway', function () use ($di) {
21 return new \Domain\Articles\ArticlesGateway($di->get('database'));
22 });
23
24 $di->set('Domain\Users\UsersGateway', function () use ($di) {
25 return new \Domain\Users\UsersGateway($di->get('database'));
26 });
27
28 $di->set('Domain\Articles\ArticleTransactions', function () use ($di) {
29 return new \Domain\Articles\ArticleTransactions(
30 $di->newInstance('Domain\Articles\ArticlesGateway'),
31 $di->newInstance('Domain\Users\UsersGateway'),
32 );
33 });
34
35 $di->set('Controller\ArticlesPage', function () use ($di) {
36 return new \Controller\ArticlesPage(
37 $di->get('request'),
38 $di->get('response'),
39 $di->user,
40 $di->newInstance('Domain\Articles\ArticleTransactions')
41 );
42 });
43 ?>
```

注意`Controller\ArticlesPage`服务现在引用容器中的其他服务来构建自己的对象。当我们获得`Controller\ArticlesPage`服务对象的新实例时，它会访问`$di`容器以获取共享的请求和响应对象、`$user`属性以及*ArticleTransactions*服务对象的新实例。这反过来又会递归地访问`$di`容器以获取该服务对象的依赖关系，依此类推。

### 如果页面脚本中有包含文件怎么办？

尽管我们已经尽力删除它们，但我们的页面脚本中可能仍然存在一些包含文件。当我们将页面脚本逻辑复制到容器时，我们别无选择，只能一并复制它们。然而，一旦我们所有的页面脚本都转换为容器，我们就可以寻找共同点，并开始将包含逻辑提取到设置脚本或单独的类中（如果需要，这些类本身可以成为服务）。

### 我们能减小 services.php 文件的大小吗？

根据我们应用程序中页面脚本的数量，我们的 DI 容器可能会有数十个或数百个服务定义。这可能会使单个文件难以管理或浏览。

如果愿意，将容器拆分为多个文件，并使`services.php`成为包含各种定义的一系列调用也是完全合理的。

### 我们能减小 router 服务的大小吗？

作为 DI 容器文件长度的子集，`router`服务特别可能会变得非常长。这是因为我们将应用程序中的每个 URL 映射到一个服务；如果有数百个 URL，就会有数百行`router`。

作为一种替代方案，我们可以创建一个单独的`routes.php`文件，并让它返回一个路由数组。然后我们可以在`setRoutes()`调用中包含该文件：

```php
**includes/routes.php**
1 <?php return array(
2 '/articles.php' => 'Controller\ArticlesPage',
3 ); ?>
```

```php
**includes/services.php**
1 <?php
2 // ...
3 $di->set('router', function () use ($di) {
4 $router = new \Mlaphp\Router($di->pages_dir);
5 $router->setRoutes(include '/path/to/includes/routes.php');
6 return $router;
7 });
8 ?>
```

至少这将减小`services.php`文件的大小，尽管它并不会减小路由数组的大小。

### 如果我们无法升级到 PHP 5.3 怎么办？

本章的示例显示了一个使用闭包封装对象创建逻辑的 DI 容器。闭包只在 PHP 5.3 中才可用，因此如果我们卡在较早版本的 PHP 上，使用 DI 容器似乎根本不是一个选择。

事实证明这并不正确。通过一些额外的努力和更大的容忍度，我们仍然可以为 PHP 5.2 及更早版本构建 DI 容器。

首先，我们需要扩展 DI 容器，以便我们可以向其添加方法。然后，我们不再将服务定义为闭包，而是将它们创建为我们扩展容器上的方法：

```php
**classes/Di.php**
1 <?php
2 class Di extends \Mlaphp\Di
3 {
4 public function database()
5 {
6 return new \Database(
7 $this->db_host,
8 $this->db_user,
9 $this->db_pass
10 );
11 }
12 }
13 ?>
```

（注意我们在方法中使用`$this`而不是`$di`。）

然后在我们的`services.php`文件中，可调用的内容变成了对这个方法的引用，而不是内联闭包。

```php
**includes/services.php**
1 <?php
2 $di->set('database', array($di, 'database'));
3 ?>
```

这有些混乱但可行。它也可能变得非常冗长。我们之前将`Controller\ArticlesPage`拆分的示例最终看起来更像这样：

```php
**includes/services.php**
1 <?php
2 // ...
3 $di->set('request', array($di, 'request'));
4 $di->set('response', array($di, 'response'));
5 $di->set('database', array($di, 'database'));
6 $di->set('Domain\Articles\ArticlesGateway', array($di, 'ArticlesGateway'));
7 $di->set('Domain\Users\UsersGateway', array($di, 'UsersGateway'));
8 $di->set(
9 'Domain\Articles\ArticleTransactions',
10 array($di, 'ArticleTransactions')
11 );
12 $di->set('Controller\ArticlesPage', array($di, 'ArticlesPage'));
13 ?>
```

```php
**classes/Di.php**
1 <?php
2 class Di extends \Mlaphp\Di
3 {
4 public function request()
5 {
6 return new \Mlaphp\Request($GLOBALS);
7 }
8
9 public function response()
10 {
11 return new \Mlaphp\Response('/path/to/app/views');
12 }
13
14 public function database()
15 {
16 return new \Database(
17 $this->db_host,
18 $this->db_user,
19 $this->db_pass
20 );
21 }
22
23 public function ArticlesGateway()
24 {
25 return new \Domain\Articles\ArticlesGateway($this->get('database'));
26 }
27
28 public function UsersGateway()
29 {
30 return new \Domain\Users\UsersGateway($this->get('database'));
31 }
32
33 public function ArticleTransactions()
34 {
35 return new \Domain\Articles\ArticleTransactions(
36 $this->newInstance('ArticlesGateway'),
37 $this->newInstance('UsersGateway'),
38 );
39 }
40
41 public function ArticlesPage()
42 {
43 return new \Controller\ArticlesPage(
44 $this->get('request'),
45 $this->get('response'),
46 $this->user,
47 $this->newInstance('ArticleTransactions')
48 );
49 }
50 }
51 ?>
```

不幸的是，为了使服务名称看起来像它们相关的方法名称，我们可能不得不打破一些我们的风格约定。我们还必须将用于新实例的服务方法名称缩短为它们的结束类名，而不是它们的完全限定名称。否则，我们会发现自己有着过长和令人困惑的方法名称。

这可能会很快让人困惑，但它确实有效。总的来说，如果我们能升级到 PHP 5.3 或更高版本，那真的会更好。

# 回顾和下一步

我们终于完成了现代化的过程。我们不再有任何页面脚本。我们所有的应用逻辑都已转换为类，剩下的唯一包含文件是引导和设置过程的一部分。我们所有的对象创建逻辑都存在于一个容器中，我们可以直接修改它，而不必干扰我们对象的内部。

在这之后可能的下一步是什么呢？答案是持续改进，这将持续到你的职业生涯的最后。
