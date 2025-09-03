# 第七章. 处理身份验证

在本章中，我们将涵盖：

+   理解身份验证方法

+   设置简单的数据库身份验证

+   编写自定义身份验证方法

# 简介

在本章中，我们将讨论不同的身份验证方法，并展示一些如何进行身份验证以及如何创建自己的身份验证方法的示例。

# 理解身份验证方法

在互联网安全如此重要的情况下，对强大身份验证方法的需求是不可或缺的。因此，Zend Framework 2 提供了一系列适合每个人需求的身份验证方法。

## 准备工作

为了充分利用这个配方，我建议设置一个可工作的 Zend Framework 2 骨架应用程序。

## 如何实现...

以下是在 Zend Framework 2 中 readily 可用的身份验证方法列表——或称为适配器。我们将提供适配器的小概览以及如何使用它的说明。

### DbTable 适配器

构建`DbTable`适配器相当简单，如果我们看看以下构造函数：

```php
public function __construct(
  // The Zend\Db\Adapter\Adapter
  DbAdapter $zendDb,

  // The table table name to query on
  $tableName = null,

  // The column that serves as 'username'
  $identityColumn = null,

  // The column that serves as 'password'
  $credentialColumn = null,

  // Any optional treatment of the password before 
  // checking, such as MD5(?), SHA1(?), etcetera
  $credentialTreatment = null
);
```

### Http 适配器

在构建对象之后，我们需要定义`FileResolver`以确保确实解析了用户详情。

根据我们在`accept_schemes`选项中配置的内容，`FileResolver`可以设置为`BasicResolver`、`DigestResolver`或两者兼而有之。

让我们快速了解一下如何将`FileResolver`设置为`DigestResolver`或`BasicResolver`（我们在`/module/Application/src/Application/Controller/IndexController.php`文件中这样做）：

```php
<?php

namespace Application;

// Use the FileResolver, and also the Http 
// authentication adapter.
use Zend\Authentication\Adapter\Http\FileResolver;
use Zend\Authentication\Adapter\Http;
use Zend\Mvc\Controller\AbstractActionController;

class IndexController extends AbstractActionController
{
  public function indexAction()
  {
    // Create a new FileResolver and read in our file to use 
    // in the Basic authentication
    $basicResolver = new FileResolver();
    $basicResolver->setFile(
      '/some/file/with/credentials.txt'
    );

    // Now create a FileResolver to read in our Digest file
    $digestResolver = new FileResolver();
    $digestResolver->setFile(
      '/some/other/file/with/credentials.txt'
    );

    // Options doesn't really matter at this point, we can 
    // fill them in to anything we like
    $adapter = new Http($options);

    // Now set our DigestResolver/BasicResolver, depending 
    // on our $options set
    $adapter->setBasicResolver($basicResolver);
    $adapter->setDigestResolver($digestResolver);
  }
}
```

## 它是如何工作的...

在两个简短的示例之后，让我们看看其他可用的适配器。

### DbTable 适配器（再次）

让我们从所有适配器中最常用的一个开始，即`DbTable`适配器。这个适配器连接到数据库，并从表中提取所需的用户名/密码组合，如果一切顺利，它将返回一个身份，这不过是一个与用户名详情匹配的记录。

要实例化适配器，它需要在构造函数中提供一个`Zend\Db\Adapter\Adapter`以连接到数据库并使用用户详情；还有一些其他选项可以设置。让我们看看构造函数的定义：

第二个（`tableName`）选项不言自明，因为它只是表名，我们需要用它来获取我们的用户，第三个和第四个（`identityColumn`，`credentialColumn`）选项是逻辑上的，它们代表我们表中的用户名和密码（或我们使用的）列。然而，最后一个选项`credentialTreatment`可能不太容易理解。

`credentialTreatment`告诉适配器在尝试查询之前用函数处理`credentialColumn`。如果是在 MySQL 数据库中，这可以是`MD5(?)`函数、`PASSWORD(?)`或`SHA1(?)`函数，但显然这也可以根据数据库的不同而有所不同。为了给出一个关于 SQL 如何看起来的小例子（实际的适配器会以不同的方式构建这个查询），请看以下示例：

有凭证处理：

```php
SELECT * FROM `users` WHERE `username` = 'some_user' AND `password` = MD5('some_password');
```

没有凭证处理：

```php
SELECT * FROM `users` WHERE `username` = 'some_user' AND `password` = 'some_password';
```

当定义处理时，我们应该始终包括一个问号，表示密码需要出现的位置，例如，`MD5(?)`将创建`MD5('some_password')`，但没有问号则不会插入密码。

最后，我们不仅可以通过构造函数提供选项，还可以使用属性设置器方法：`setTableName()`、`setIdentityColumn()`、`setCredentialColumn()`和`setCredentialTreatment()`。

### Http 适配器（再次）

HTTP 认证适配器是我们可能在互联网生活中至少遇到过一次的适配器。当我们访问一个网站并且弹出一个窗口显示我们可以填写用户名和密码以继续时，我们可以识别出这种认证。

这种认证方式非常基础，但在某些实现中仍然非常有效，因此它是 Zend Framework 2 的一部分。然而，这种认证有一个很大的问题，那就是（在使用基本认证时）它可以通过浏览器发送用户名和密码的明文（ouch!）。

然而，有一个解决方案，那就是使用摘要认证，这种认证也由这个适配器支持。

如果我们查看这个适配器的构造函数，我们会看到以下代码行：

```php
public function __construct(array $config);
```

构造函数接受其`config`参数中的大量键，如下所示：

+   `accept_schemes`：这指的是我们想要接受的认证方式；这可以是`basic`、`digest`或`basic digest`。

+   `realm`：这是我们对所在领域的描述，例如“会员区域”。这是针对用户的，仅用于描述用户登录的目的。

+   `digest_domains`：这是认证正在工作的 URLs。所以如果用户在任何定义的 URL 上使用他的详细信息登录，它们将有效。URLs 应该定义在一个空格分隔的（奇怪，对吧？）列表中，例如 `/members/area /members/login`。

+   `nonce_timeout`：这将设置 nonce（当我们使用摘要认证时，用户登录时使用的哈希）有效的秒数。请注意，然而，nonce 跟踪和过时支持在 2.2 版本中尚未实现，这意味着每次 nonce 超时时都会重新认证。

+   `use_opaque`：这是一个布尔值（默认为 true），告诉我们的适配器向客户端发送不透明头。不透明头是服务器发送的字符串，需要在身份验证时返回。不过，在 Microsoft Internet Explorer 浏览器上有时不起作用，因为它们似乎忽略了该头。理想情况下，不透明头应该是一个不断变化的字符串，以减少可预测性，但 ZF 2 不随机化字符串，总是返回相同的哈希。

+   `algorithm`：这包括用于身份验证的算法，它需要是一个在 `supportedAlgos` 属性中定义的支持算法。目前只有 MD5。

+   `proxy_auth`：这个布尔值（默认为 false）告诉我们所使用的身份验证是否是代理身份验证。

应该注意的是，在使用 Digest 或 Basic 时，文件之间略有差异。尽管这两个文件具有相同的布局，但它们不能互换使用，因为 Digest 需要将凭证进行 MD5 哈希，而 Basic 需要将凭证作为纯文本。在每条凭证之后都应该有一个新行，这意味着凭证文件中的最后一行应该是空的。

凭证文件的布局如下：

```php
username:realm:credentials
```

例如：

```php
some_user:My Awesome Realm:clear text password
```

除了 `FileResolver`，还可以使用 `ApacheResolver`，它可以用来读取 `htpasswd` 生成的文件，这在已经有此类文件的情况下非常有用。

### Digest 适配器

`Digest` 适配器基本上是没有任何基本身份验证的 `Http` 适配器。由于它的理念与 `Http` 适配器相同，我们只需继续讨论构造函数，因为它的实现略有不同：

```php
public function __construct($filename = null, $realm = null, $identity = null, $credential = null);
```

如我们所见，在构建对象时可以设置以下选项：

+   `filename`：这是用于 Digest 凭证的直接文件名，因此无需使用 `FileResolver`。

+   `realm`：这用于向用户标识他/她正在登录的系统，例如 `My Awesome Realm` 或 `The Dragonborn's lair`。由于我们在构建此文件时立即尝试登录，因此它确实需要与凭证文件（参见 *Http 适配器* 中的凭证文件布局）相匹配。

+   `identity`：这是我们尝试登录的用户名，并且它需要与凭证文件中定义的用户相似才能正常工作。

+   `credential`：这是我们尝试登录的 Digest 密码，并且它需要与凭证文件中的密码完全匹配。

然后，例如，我们可以运行 `$digestAdapter->getIdentity()` 来找出我们是否成功进行了身份验证，如果没有成功，则返回 `NULL`，如果成功，则返回身份列的值。

### LDAP 适配器

使用 LDAP 认证显然要难解释一些，所以我们不会全面介绍，因为这会花费相当长的时间。我们将展示 `LDAP` 适配器的构造函数并解释其各种选项。然而，如果我们想了解更多关于设置 LDAP 连接的信息，我们应该查看 ZF2 的文档，因为那里解释得很好：

```php
public function __construct(array $options = array(), $identity = null, $credential = null);
```

构造函数中的选项参数指的是一个与 `Zend\Ldap\Ldap` 配置兼容的配置选项数组。这里可以设置的实际选项有数十种，所以我们建议查看 ZF2 的 LDAP 文档以了解更多信息。接下来的两个参数 identity 和 credential 分别是用户名和密码，这一点很容易理解。

一旦与 LDAP 建立了连接，就没有太多的事情要做，只剩下获取身份并查看我们是否成功验证。

### 关于认证

在 Zend Framework 2 中，认证通过特定的适配器实现，这些适配器始终是 `Zend\Authentication\Adapter\AdapterInterface` 的实现，因此始终提供了那里定义的方法。然而，认证的方法各不相同，对之前显示的方法有深入了解始终是必需的。有些方法通过浏览器工作，如 `Http` 和 `Digest` 适配器，而有些则要求我们创建整个实现，如 `LDAP` 和 `DbTable` 适配器。

# 设置简单的数据库认证

在看到所有可用的认证方法后，是时候看看当设置了数据库认证时它实际上会如何工作了。这个菜谱将解释这个特定方法的方方面面。

## 准备工作

一个带有 PHP sqlite 扩展已加载和启用的有效 Zend Framework 2 框架应用骨架。

## 如何操作…

数据库认证可能是最广泛使用的认证方法。在这个菜谱中，我们将设置自己的数据库认证。

### 设置模块初始化

我们将在模块初始化后尽快创建数据库，因此我们将它附加到名为 route 或 `MvcEvent::EVENT_ROUTE` 的事件。作为 `Module.php` 的模板，我们只需复制 `Application/Module.php` 文件并更改命名空间；我们无论如何都会在 `onBootstrap` 方法中工作，`Module` 类的其余部分可以保持不变（但别忘了更改命名空间！）。

让我们看看我们的 `/module/Authentication/Module.php` 文件的代码：

```php
// We can assume the rest of the Module class file is 
// exactly the same as the default 
// Application/Module.php file, except of course the 
// namespace.
public function onBootstrap(MvcEvent $e)
{
  // This is also default    
  $eventManager = $e->getApplication()->getEventManager();
  $moduleRouteListener = new ModuleRouteListener();
  $moduleRouteListener->attach($eventManager);
  // And now we let the magic happen (this is the bit we 
  // will insert)
  $eventManager->attach(
    // We want to attach to the route event, which means   
    // it happens before our controllers are initialized 
    // (because that would mean we already found the 
    // route)
    MvcEvent::EVENT_ROUTE,

    // We are using this function as our callback 
    function (MvcEvent $event) 
    {
     // Get the database adapter from the configuration
     $dbAdapter = $event->getApplication()
                         ->getServiceManager()
                         ->get('db');

     // Our example is an in memory database, so the 
     // table never exists, but better sure than sorry
     $result = $dbAdapter->query("
             SELECT name 
            FROM sqlite_master 
           WHERE type='table' AND name='users'
     ")->execute();

      // If we couldn't find a users table, we will 
      // create one now (with an in memory db this is 
      // always the case)
     if ($result->current() === false) {
       try {
         // The user table doesn't exist yet, so let's 
         // just create some sample data
         $result = $dbAdapter->query("
            CREATE TABLE `users` (
              `id` INT(10) NOT NULL,
              `username` VARCHAR(20) NOT NULL,
              `password` CHAR(32) NOT NULL,
            PRIMARY KEY (`id`)
            )
           ")->execute();

         // Now insert some users
         $dbAdapter->query("
         INSERT INTO `users` VALUES 
           (1, 'admin', '". md5("adminpassword"). "')
          ")->execute();

         $dbAdapter->query("
           INSERT INTO `users` VALUES 
             (2, 'test', '". md5("testpassword"). "')
             ")->execute();		
        } catch (\Exception $e) {
        \Zend\Debug\Debug::dump($e->getMessage());
      }
    }
  });
}
```

我们现在创建了一个事件，当开始路由时会触发。如果我们仔细观察，我们可以找到一个肯定会导致代码崩溃的大错误。问题当然在于 `ServiceManager` 中的 db key，因为我们引用了一个尚未创建的服务。所以让我们开始动手，创建那个 `/module/Authentication/config/module.config.php` 文件…

```php
<?php

return array(
  // Let's initialize the ServiceManager
  'service_manager' => array(
    'factories' => array(
      // Create a Db Adapter on initialization of the 
      // ServiceManager
      'Zend\Db\Adapter\Adapter' =>
          'Zend\Db\Adapter\AdapterServiceFactory',
    ),

    // Let's give this Db Adapter the alias db
    'aliases' => array(
      'db' => 'Zend\Db\Adapter\Adapter',
    ),
  ),

  // We will now configure our Sqlite database, for 
  // which we only need these two lines
  'db' => array(
    'driver' => 'Pdo_Sqlite',
    'database' => ':memory:',
  ),
);
```

就这样；我们的基本配置已经完成，以便数据库开始运行，如果我们现在运行代码，我们可以确信数据库已经创建。

### 创建认证服务

我们接下来要做的事情是创建我们的认证服务，这个服务将帮助我们的应用程序完成所有的认证功能。让我们在`Authentication\Service`命名空间中创建这个服务，并将其命名为`Authentication`（文件位于`/module/Authentication/src/Authentication/Service/Authentication.php`）。

```php
<?php

// Set the namespace
namespace Authentication\Service;

use Zend\ServiceManager\ServiceLocatorAwareInterface;

// We give this one an alias, because otherwise 
// DbTable might confuse us in thinking that it is  
// an actual db table
use Zend\Authentication\Adapter\DbTable as AuthDbTable;
use Zend\Authentication\Storage\Session;

// We want to make a service, so we implement the 
// ServiceLocatorAwareInterface for that as well
class Authentication implements ServiceLocatorAwareInterface
{
  // Storage for our service locator
  private $servicelocator;

  // Get the ServiceManager
  public function getServiceLocator() 
  {
    return $this->servicelocator;
  }

  // Set the ServiceManager
  public function setServiceLocator(\Zend\ServiceManager\ServiceLocatorInterface $serviceLocator) 
  {
    $this->servicelocator = $serviceLocator;
  }
```

好的，这很简单；我们只是创建了一个服务……但目前它实际上什么都没做。让我们首先创建一个检查我们是否已经认证的方法。我们通过检查认证会话，看它是否为空来完成这个操作。假设在这种情况下，我们只有当实际上已经认证时才有一个（认证！）会话，我们可以安全地同意我们将登录；

```php
  /**
   * Lets us know if we are authenticated or not.
   * 
   * @return boolean
   */
  public function isAuthenticated()
  {
    // Check if the authentication session is empty, if 
    // not we assume we are authenticated
    $session = new Session();

    // Return false if the session IS empty, and true if 
    // the session ISN'T empty
    return !$session->isEmpty();
  }
```

我们可以轻松地打开一个会话，因为会话的命名空间仅用于认证目的。

让我们现在创建我们的认证服务，它将验证用户名和密码，并返回一个布尔值，表示我们是否成功认证：

```php
  /**
   * Authenticates the user against the Authentication 
   * adapter.
   * 
   * @param string $username
   * @param string $password
   * @return boolean
   */
  public function authenticate($username, $password)
  {
    // Create our authentication adapter, and set our 
    // DbAdapter (the one we created before) by getting 
    // it from the ServiceManager. Also tell the adapter 
    // to use table 'users', where 'username' is the 
    // identity and 'password' is the credential column
    $authentication = new AuthDbTable(
      $this->getServiceLocator()->get('db'),
      'users',
      'username',
      'password'
    );

    // We use md5 in here because SQLite doesn't have 
    // any functionality to encrypt strings
    $result = $authentication->setIdentity($username)
                             ->setCredential(md5($password))
                             ->authenticate();

    // Check if we are successfully authenticated or not
    if ($result->isValid() === true) {
      // Now save the identity to the session
      $session = new Session();
      $session->write($result->getIdentity());
    }

    return $result->isValid();
  }
```

正如我们在之前的代码片段中看到的，我们创建了一个简单的认证方法，它返回 true 或 false，这取决于我们是否已经认证。它还做的一件事是将身份保存到认证会话中，这样我们就可以在我们的上一个方法中看到我们是否已经认证。当我们想要从登录用户那里获取用户名时，我们也需要在会话中获取身份，这可以通过以下方法实现：

```php
  /**
   * Gets the identity of the user, if available, 
   * otherwise returns false.
   * @return array
   */
  public function getIdentity()
  {
    // Clear out the session, we are done here
    $session = new Session();

    // Check if the session is empty, if not return the 
    // identity of the logged in user
    if ($session->isEmpty() === false) {
      return $session->read();
    } else {
      return false;
    }
  }
```

现在我们已经获取了我们的身份，我们能够注销也同样重要。在我们的情况下，这很简单，只需清除会话即可，因为我们为什么要让它比这更复杂呢？

```php
  /**
   * Logs the user out by clearing the session.
   */
  public function logout()
  {
    // Clear out the session, we are done here
    $session = new Session();
    $session->clear();
  }

  // This is our last method, close the bracket for the 
  // class as well!
}
```

我们现在已经创建了一个简单的认证服务，现在剩下的唯一部分就是将其注册到服务管理器中，以便在启动时自动实例化。我们可以像往常一样在`/module/Authentication/config/module.config.php`文件中完成这项操作，因为我们已经有一个`service_manager`配置在那里，我们只需将可调用的实例放入其中：

```php
<?php
return array(
  'service_manager' => array(
    // [The rest of the service manager configuration 
    // comes here]

    // And our new invokable can be put here
    'invokables' => array(
    'AuthService' => 'Authentication\Service\Authentication',
    ),
  ),
);
```

对于服务来说这就结束了！现在我们唯一要做的就是创建`login/logout`操作，然后检查我们是否已经登录。让我们从`login/logout`操作开始，这样我们实际上能够登录！

### 设置控制器和操作

让我们在那里修改`/module/Authentication/config/module.config.php`文件，这样我们就可以访问我们的`login/logout`操作，这对我们来说非常重要：

```php
<?php
return array(
  // [The configuration that we have now resides here..]

  // And our route configuration comes here..
  'router' => array(
    'routes' => array(
      'authentication' => array(
        'type'    => 'Literal',
        'options' => array(
          'route'    => '/authentication',
          'defaults' => array(
          '__NAMESPACE__' => 
                  'Authentication\Controller',
          'controller'    => 'Index',
          'action'        => 'login',
        ),
      ),
      'may_terminate' => true,
      'child_routes' => array(
        'default' => array(
          'type'    => 'Segment',
          'options' => array(
            'route'    => '[/:action]',
            'constraints' => array(
              'action'     => '[a-zA-Z][a-zA-Z0-9_-]*',
            ),
            'defaults' => array(),
          ),
        ),
      ),
    ),
  ),
),

// Make our controller invokable
'controllers' => array(
  'invokables' => array(
    'Authentication\Controller\Index' =>   
          'Authentication\Controller\IndexController'
    ),
  ),

  // Make sure our template path is set correctly
  'view_manager' => array(
    'template_path_stack' => array(
      __DIR__ . '/../view',
    ),
  ),

);
```

这个基本的路由只是将 `/authentication` 重定向到我们的 `loginAction`，由于段路由，我们可以简单地通过 `/authentication/logout` 重定向到我们的 `logoutAction`；如果需要更多关于路由的解释，我们可以回顾第一章，*Zend Framework 2 基础*，并查看 *Handling routines* 菜单。

让我们继续在 `Authentication\Controller` 命名空间中创建我们的 `/module/Authentication/src/Authentication/Controller/IndexController`：

```php
<?php

namespace Authentication\Controller;

use Zend\Mvc\Controller\AbstractActionController;
use Zend\View\Model\ViewModel;

class IndexController extends AbstractActionController
{
}
```

我们已经简单地声明了我们的控制器；现在让我们添加 `logoutAction`（我们将从它开始，因为它非常简单）和 `loginAction`：

```php
public function logoutAction()
{
  // Log out the user
  $this->getServiceLocator()
       ->get('AuthService')
       ->logout();

  // Redirect the user back to the login screen
  $this->redirect()
       ->toRoute('authentication');
}
```

如我们所见，这几乎是太简单了，但如果它能工作，我们不会抱怨。现在让我们创建我们的 `loginAction`，它基本上检查是否有帖子，如果有，就尝试登录，否则显示登录表单。登录成功后，我们将被重定向到 `/application` 路由，如果没有成功，我们只显示一个错误消息：

```php
public function loginAction()
{
  // See if we are trying to authenticate
  if ($this->params()->fromPost('username') !== null) {
    // Try to authenticate with our post variables from 
    // the form we just send
    $done = $this->getServiceLocator()
                 ->get('AuthService')
                 ->authenticate(
        $this->params()->fromPost('username'),
        $this->params()->fromPost('password')
    );

    if ($done === true) {
      $this->redirect()
           ->toRoute('application');
    } else {
      \Zend\Debug\Debug::dump(
        "Username/password unknown!"
      );
    }
  }

  // On an unsuccessful attempt or just a get request 
  // show the form.
  return new ViewModel();
}
```

如我们所见，`loginAction` 只是在检查是否有任何帖子，如果有，它就允许 `AuthService` 处理它。这种方式并不完美，因为它不检查恶意参数或任何东西，但它确实展示了控制器应该是多么干净，除了必要的变量解析之外，没有登录。

`logoutAction` 不包含视图脚本，因为这个动作只是重定向用户，并且永远不会有自己的响应。然而，`loginAction` 确实包含视图脚本，因为它需要显示一个表单。现在让我们快速为 `loginAction` 建立一个视图脚本（文件位于 `/module/Authentication/view/authentication/index/login.phtml`）：

```php
<form action="/authentication" method="post">
  <label for="username">Username:</label>
  <input type="text" name="username" />

  <label for="password">Password:</label>
  <input type="password" name="password" />

  <button type="submit">Login</button>
</form>
```

一个简单的登录表单，在我看来不需要任何解释。

我们最后想要确保的是，如果用户未登录，他们不能访问应用程序中的任何内容，除了认证。我们可以通过在 Authentication 模块的 Module（文件位于 `/module/Authentication/Module.php`）类中添加一个新的事件来实现这一点，该事件将检查我们是否已登录，如果没有，则在屏幕输出任何内容之前将我们重定向：

```php
public function onBootstrap(MvcEvent $e)
{
  // Get the event manager from the event
  $eventManager = $e->getApplication()->getEventManager();

  // Attach the module route listeners
  $moduleRouteListener = new ModuleRouteListener();
  $moduleRouteListener->attach($eventManager);

  // Do this event when dispatch is triggered, on the 
  // highest priority (1)
  $eventManager->attach(
      MvcEvent::EVENT_DISPATCH, 
      function (MvcEvent $event) {
      // We don't have to redirect if we are in a 
      // 'public' area, so don't even try
      if ($event->getRouteMatch()->getMatchedRouteName() 
                  === 'authentication') return;

      // See if we are authenticated, if not lets 
      // redirect to our login page
      if ($event->getApplication()->getServiceManager()
                ->get('AuthService')->isAuthenticated() === 
          false) 
      {
        // Get the response from the event
        $response = $event->getResponse();

        // Clear current headers and add our Location 
        // redirection
        $response->getHeaders()
                 ->clearHeaders()
                 ->addHeaderLine(
             'Location', '/authentication'
        );

        // Set the status code to redirect
        $response->setStatusCode(302)
                 ->sendHeaders();

        // Don't forget to exit the application, as we 
        // don't want anything to overrule at this point
        exit;
      }
  }, 

  // Give this event priority 1
  1);
}
```

需要的事件就是这样的，发生的情况是，每次我们尝试访问非认证路由时，我们都会被重定向到登录页面。

## 它是如何工作的…

我们将要创建的是一个简单的数据库认证，它通过内存中的 SQLite 数据库工作。这意味着数据库不会被存储，每次我们请求页面时，所有表和记录都需要重新构建。显然，这在生产环境中使用非常不方便，但它确实可以很好地展示它是如何工作的，并且对于快速启动非常有用。

假设我们正在使用默认的 Zend Skeleton 应用程序，让我们创建一个新的认证模块。这个新模块将包含数据库连接、认证本身以及登录和注销操作。当我们为新的模块创建目录时，我们还应该小心地将新模块添加到`application.config.php`文件中，否则我们可能会遇到麻烦，不知道为什么它不起作用（哦，是的，我是在从经验中说的）。

首先，我们在`Module.php`中为认证构建了我们的内存数据库。然后我们创建了一个名为`users`的表，包含一个唯一的 ID、用户名和密码。ID 由一个整数组成，用户名是一个 20 位的可变字符，密码将是一个 32 个字符，因为这是 MD5 加密字符串的大小。

因为我们已经设置了一个用户表，并将其连接到认证适配器，所以我们能够简单地验证用户名和密码。作为额外的措施，我们确保用户在没有登录的情况下不能访问除登录页面之外的任何页面，这是通过使用在输出发送给用户之前发生的事件来实现的。

# 编写自定义认证方法

有时候，标准方法可能不够用，这是完全可以接受的。这就是为什么这个食谱清楚地说明了如何创建我们自己的认证方法。

## 准备工作

对于这个食谱，如果有一个启用了 SSL 的 Web 环境会更好。配置这样的环境超出了范围，但它对执行这个食谱会有好处。

这样的一个环境示例将是正确配置了`mod_ssl`的 Apache 2 Web 服务器。要在 Apache2 上启用证书验证，需要在他们的`public/.htaccess`文件中放置以下代码：

```php
# Only execute the following code when mod_ssl is 
# enabled
<IfModule mod_ssl.c>
  # This means the client can present their 
  # certificate, but it doesn't need to be verifiable 
  # by the server
  SSLVerifyClient optional_no_ca

  # This depth means the certificate can only be self-
  # signed otherwise it will be denied
  SSLVerifyDepth 0

  # We want to export the standard variables but also 
  # the certificate data as well to use in PHP
  SSLOptions +StdEnvVars +ExportCertData 
</IfModule>
```

另一个需要提到的重要事情是，PHP 应该配置（和编译）带有`--with-openssl`参数，否则解析证书的代码将不存在，因此我们无法使用该代码。有关如何做到这一点的更多信息，请参阅[`www.php.net/manual/en/book.openssl.php`](http://www.php.net/manual/en/book.openssl.php)。

## 如何做到这一点...

证书认证可能很少见，但有时当安全性略高于普通 Web 应用时，会使用它。在这个食谱中，我们将展示一个基于证书的认证示例。

### 创建我们的适配器

让我们开始创建我们的新适配器，这将是我们想要尽可能与当前认证适配器集成的`Zend\Authentication\Adapter\AdapterInterface`的实现。

由于我们已经有了一个来自上一个食谱的认证模块，我们只需将其作为我们将要工作的命名空间即可；就像之前描述的那样简单。

首先，让我们创建适配器（文件位于`/module/Authentication/src/Authentication/Adapter/Certificate.php`）。

### 适配器概述

我们首先从我们的基本类轮廓开始：

```php
// Set the right namespace
namespace Authentication\Adapter;

// We will use this to implement the right methods
use Zend\Authentication\Adapter\AdapterInterface;

// Out class name, not to forget the implementation
class Certificate implements AdapterInterface
{
  // Currently authenticate is the only method required 
  // for the AdapterInterface; lucky us!
  public function authenticate() {}
}
```

### 为任何错误消息创建 getter 和 setter

通常我们会在开发过程中逐步想出错误消息。但因为这个代码已经完成，我们已经有定义好的错误消息。没有很好的方法来描述这些 getter 和 setter，所以我们将在以下代码片段中展示它们，以便至少可以清楚地了解发生了什么（文件位于`/module/Authentication/src/Authentication/Adapter/Certificate.php`）：

```php
// After coding the adapter we found the following 
// errors that need to be relayed to the user/developer

// Invalid certificate, there is no certificate set
const AUTH_FAIL_INV_CERT = 0;

// Insecure connection, no HTTPS
const AUTH_FAIL_NO_HTTPS = 1;

// Couldn't parse the certificate, invalid certificate
const AUTH_FAIL_PARSE_CERT = 2;

// Certificate is expired
const AUTH_FAIL_EXP_CERT = 3;

// Not all the required fields we need are in the 
// certificate, thus rendering it invalid
const AUTH_FAIL_NOT_ALL_FIELDS = 4;

// No Database adapter was provided
const AUTH_FAIL_NO_DB_ADAPTER = 5;

// An error occurred in the SQL
const AUTH_FAIL_SQL_ERR = 6;

// The user requested couldn't be found
const AUTH_FAIL_NO_USER = 7;

// By default we have no error
private $error = -1;
```

这些是我们考虑到的错误消息，稍后将在代码的某个地方使用。现在让我们创建这些错误消息的 setter，以便 getter 可以轻松地在以后检索它们：

```php
/**
 * Sets an error.
 * 
 * @param int $error
 */
private function setError($error) 
{
  $this->error = $error;
}
```

好吧，那真是有趣。现在让我们创建 getter，它稍微复杂一些，但只是稍微复杂一点：

```php
/**
 * Gets the latest error message back.
 * 
 * @return string
 */
public function getErrorMessage() 
{
  switch ($this->error) {
    case self::AUTH_FAIL_SQL_ERR:
      $retval = "SQL error occurred while checking " 
              . "for the user.";
      break;
    case self::AUTH_FAIL_INV_CERT:
      $retval = "Certificate provided is invalid.";
      break;
    case self::AUTH_FAIL_PARSE_CERT:
      $retval = "Certificate provided couldn't be "
              . "parsed.";
      break;
    case self::AUTH_FAIL_EXP_CERT:
      $retval = "Certificate has expired.";
      break;
    case self::AUTH_FAIL_NO_DB_ADAPTER:
      $retval = "No Database adapter set.";
      break;
    case self::AUTH_FAIL_NOT_ALL_FIELDS:
      $retval = "Not all the fields required are " 
              . "available.";
      break;
    case self::AUTH_FAIL_NO_USER:
      $retval = "The user could not be found.";
      break;
    case self::AUTH_FAIL_NO_HTTPS:
      $retval = "Connection is not secure.";
      break;
    case -1:
      $retval = "No error occurred.";
      break;
    default:
      $retval = "Unknown error occurred.";
      break;
  }

  // Reset the error
  $this->error = -1;

  // Return the string with the error message
  return $retval;
}
```

### 确保我们有一个安全的连接

虽然证书只有在我们有 SSL 连接时才会发送，但额外的检查并不坏，因为我们想确保用户正在使用安全的连接。

```php
/**
 * Returns true if the current connection is through 
 * HTTPS.
 * 
 * @return boolean
 */
private function isHTTPS()
{
  return isset($_SERVER['HTTPS']) ? true : false;
} 
```

哇，那肯定是有史以来最好的方法！开个玩笑，它相当简单，因为`HTTPS`键在`$_SERVER`变量中给出，每当通过 HTTPS 建立安全连接时。当键存在时，我们可以假设存在一个安全连接。

### 检查证书是否为实际证书

接下来是检查证书是否有效，但在我们可以这样做之前，我们也应该确保有方法设置证书：

```php
// This property will store our certificate array
private $certificate;

/**
 * Sets (and parses) a certificate, returns false if the 
 * certificate couldn't be parsed.
 * 
 * @param string $certificateContent
 * @return boolean
 */
public function setCertificate($certificateContent) 
{
  // This function is part of the OpenSSL extension in 
  // PHP. This means that if OpenSSL is not installed 
  // into PHP this function will not exist and thus give 
  // a fatal error. This function deciphers the 
  // information received in the certificate to a great 
  // array with variables.
  $certificate = openssl_x509_parse(
               $certificateContent
  );

  // If the certificate can't be parsed (i.e. it is 
  // invalid) the function above will return false
  if ($certificate !== false) {
    // We can be sure the certificate is valid at least 
    // in raw state now
    $this->certificate = $certificate;

    // Done here
    return true;
  } else {
    // Use the failure to parse certificate here to make 
    // sure the developer/user will know what is going 
    // on
    $this->setError(self::AUTH_FAIL_PARSE_CERT);
    return false;
  }
}
```

此方法进行了基本检查，以查看我们是否真的得到了至少有效的证书，即使它已过期或没有我们的任何字段。

### 检查我们是否拥有所有证书字段

因为我们想检查证书中的电子邮件地址，我们需要确保我们确实有一个电子邮件地址在那里。同时，我们还将检查几个与我们认证无关但仍然很不错的字段：

```php
/**
 * Checks if all our fields (issuer, issuer[O], 
 * issuer[CN], issuer[emailAddress], serialNumber) are 
 * in the certificate.
 * 
 * @return boolean
 */
private function checkRequiredFields()
{
  // First get our certificate
  $certificate = $this->getCertificate();

  // Check if our certificate at least is valid
  if ($certificate !== false) {
    // We want to check if the following fields (and 
    // subfields) are in the certificate
    $required = array(
      'issuer' => array('O', 'CN', 'emailAddress'), 
      'serialNumber' => null
    );

    // Loop through the primary fields
    foreach ($required as $field=>$value) {
      if (in_array($field, $certificate) === true) {
        // The primary field is in there, check if 
        // there are any secondary fields we need to 
        // check
        if (is_array($value && is_array($certificate[$field) {
          // Loop through the secondary fields
          foreach ($value as $key) {
            // Now check of our values are in there
            if (in_array(
              $key, 
              array_keys(
                $certificate[$field])) === false) 
              {
                return false;
              }
            }
          }
        } else {
          return false;
        }
      }

      // If we reach this point, we are always ok to go
      $retval = true;

      unset($required);
    }

  unset($certificate);

  return isset($retval) ? $retval : false;
}
```

我们检查字段是否在其中，如果字段不在其中，我们将返回 false。

### 检查证书尚未过期

现在我们想知道证书在时间上是否仍然有效，因为证书通常在设定的时间后过期（这可能是一月、一周、一年，实际上可以是任何时间）：

```php
/**
 * Checks if the current certificate is valid or not.
 * 
 * @return boolean
 */
private function isCertificateValid()
{
  // Get our certificate again
  $certificate = $this->getCertificate();

  // Again make sure it is not false (highly unlikely 
  // here, but hey, never be sure
  if ($certificate !== false) {
    // Check if the valid from and to fields are set, 
    // because if they are not, we won't be able to 
    // check if the certificate is valid or not
    if (isset($certificate['validFrom_time_t']) === true && isset($certificate['validTo_time_t']) === true)  
    { 
      // Check if the from time is smaller than our 
      // current time and the to time is bigger than the 
      // current time
    if (time() >= $certificate['validFrom_time_t'] && time() < $certificate['validTo_time_t']) 
      {
        $retval = true;
      }
    }
  }

  unset($certificate);

  return isset($retval) ? $retval : false;
}
```

如果此方法返回 true，我们可以确信我们有一个未过期的证书。

### 为数据库适配器创建 getter 和 setter

现在我们需要一个简单的 getter 和 setter 来处理我们的数据库适配器，在我们实际上进行认证之前：

```php
/**
 * Our Database adapter property.
 * 
 * @var \Zend\Db\Adapter\Adapter
 */
private $dbAdapter;

/**
 * Sets the Db adapter.
 *	
 * @param \Zend\Db\Adapter\Adapter $db
 */
public function setDbAdapter(\Zend\Db\Adapter\Adapter $db) 
{
  $this->dbAdapter = $db;
}

/**
 * Returns the Db adapter.
 * 
 * @return \Zend\Db\Adapter\Adapter
 */
private function getDbAdapter()
{
  return $this->dbAdapter;
}
```

当然，这又很简单，因为它根本不需要任何逻辑。现在我们已经设置了数据库适配器，我们实际上可以开始认证用户了。

### 创建认证方法

此方法将实现我们之前定义的所有方法，如果它们都成功了，它将通过数据库进行身份验证，看看我们的用户是否在那里（或者不在）。但是首先，我们需要另一种方法来从我们的证书中获取字段，这是一种更整洁的方法，以及一种在认证后获取我们身份的方法：

```php
// We will store our identity in here, once 
// authenticated
private $identity;

/**
 * Retrieves a variable from the certificate, returns 
 * null if not found.
 * 
 * @param string $variable
 * @return string
 */
private function getCertificateVariable($variable)
{
  if (is_array($this->certificate) === true && isset($this->certificate[$variable]) === true) 
  {
    return $this->certificate[$variable];
  } else if (is_array($this->certificate) === true && isset($this->certificate['issuer'][$variable) 
  {
    return $this->certificate['issuer'][$variable];
  } else {
    return null;
  }
}

/**
 * Retrieves the identity of the user.
 * 
 * @return array
 */
public function getIdentity() 
{
  return $this->identity;
}
```

现在是最高潮的时刻，经过漫长的等待，终于到了`authenticate`方法！

```php
/**
 * Tries to authenticate the user through the 
 * certificate.
 * 
 * @return boolean
 */
public function authenticate() 
{
  $continue = true;

  if ($this->getDbAdapter() !== null) {
    // Check if we are on a secure connection
    if ($this->isHTTPS() === true) {
      // Check if the certificate is valid
      if ($this->getCertificate() !== false) {
        // Check if the fields we require are available
        if ($this->checkRequiredFields() === true) {
          // Check if the certificate isn't expired
          if ($this->isCertificateValid() === false) {
            // Certificate is expired!
            $this->setError(self::AUTH_FAIL_EXP_CERT);
            $continue = false;
          }
        } else {
          // Not all the fields are available
          $this->setError(
              self::AUTH_FAIL_NOT_ALL_FIELDS
          );
          $continue = false;
        }
      } else {
        // This is an invalid certificate
        $this->setError(self::AUTH_FAIL_INV_CERT);
        $continue = false;
      }
    } else {
      // Oh, oh, no secure connection
      $this->setError(self::AUTH_FAIL_NO_HTTPS);
      $continue = false;
    }
  } else {
    // We don't have a db adapter
    $this->setError(self::AUTH_FAIL_NO_DB_ADAPTER);
    $continue = false;
  }

  if ($continue === true) {
    // Now we are going to check with the database if 
    // the email address is in there
    $statement = $this->getDbAdapter()->createStatement(
      "SELECT * FROM users WHERE email = :email"
    );

    try { 
      // Input the email address in the statement and 
      // execute it on the database adapter
      $result = $statement->execute(array(
        'email' => $this->getCertificateVariable(
            'emailAddress'
        )
      ));

      // Check if we have one result
      if ($result->count() === 1) {
        // One result found, put it in the identity kit
        $this->identity = $result->current();

        // Because we are super-cool add some of our 
        // certificate variables as well
        $this->identity['serialNumber'] = 
          $this->getCertificateVariable('serialNumber');

        $this->identity['organization'] = 
          $this->getCertificateVariable('O');

        $this->identity['commonName'] = 
          $this->getCertificateVariable('CN');

        // We successfully found our user
        $retval = true;
      } else {
        $this->setError(self::AUTH_FAIL_NO_USER);
      }
    } catch (\Exception $e) {
      $this->setError(self::AUTH_FAIL_SQL_ERR);
      error_log($e->getMessage());
    }
  }

  // Return the retval is we have one, otherwise just 
  // false
  return isset($retval) ? $retval : false;
}
```

就这样！`authenticate`方法将在认证成功时返回 true，或者在失败时返回 false，同时设置一个错误，这样我们就可以看到到底出了什么问题！

## 它是如何工作的…

现在我们已经创建了我们的`authentication`适配器，是时候坐下来回顾我们刚才所做的一切了。

### 我们试图实现什么

在某些网站上，访问被禁止到了这种程度，以至于用户名和密码已成为过去式。在我们希望检查每个进入的顾客而不需要他们自己输入用户名和密码的环境中，我们可能会使用证书认证。

证书认证之所以有效，是因为客户端将在每次向服务器发送请求时发送一个证书。这个证书随后会向服务器显示用户是谁，谁正在尝试浏览他们的页面。通常，证书中的一个或多个字段被用来识别用户。在我们的例子中，我们将使用电子邮件地址，这是一个常见的用于身份验证的字段。

我们首先要做的是创建一个适配器，它将从手动输入（这样测试起来更简单）或浏览器中获取证书，哪个可行就用哪个。然后我们将检查电子邮件地址是否存在于我们的数据库中，如果是的话，我们就认为用户已经登录。显然，我们的服务器不会配置得那么严格，以至于不允许任何证书，因为在这一阶段，基本上所有带有正确电子邮件地址的证书都能获得访问权限。如果我们想知道如何防止用户使用任何证书，我们可以查看“更多内容…”部分，在那里我们将进一步探讨如何保护您的服务器，并限制证书的使用。

然而，从应用程序的角度来看，我们只是假设我们得到的所有证书都是有效的。

`AdapterInterface`只要求我们有一个认证方法。但在我们继续之前，我们想要确保以下项目已被检查：

+   我们想确保用户是通过一个安全的连接（HTTPS）来的

+   我们还想要确保证书是有效的（显然）

+   在检查的过程中，我们将确保我们的证书具有我们用于认证所需的字段

+   我们还需要知道证书是否仍然有效且未过期

+   最后但同样重要的是，我们还需要确保我们有一个数据库适配器来检查值

### 关于证书

通常，在证书到达应用程序之前，服务器会对证书进行验证。验证通常是对某种 CA，即证书颁发机构进行的，这基本上是服务器端的一个实体，它颁发证书，因此可以为带有其签名的任何证书作证。当然，在现实生活中，这比描述的要复杂得多，但基本思想是相同的。所以当服务器上进行某种程度的检查以验证证书的身份，并且如果它与服务器提供的 CA 有效时，它将解析它并通过到我们的应用程序。

当它到达我们的应用程序时，我们通常假设用户已经从我们这里获得了证书，因此应该被允许进入，因为他知道门上的密码。但尽管他知道密码，这并不意味着我们知道他是谁！这就是为什么第二次认证（我们刚刚进行的认证）验证用户实际上是否属于我们的应用程序，也就是说，证书是否有效！

## 还有更多...

保护服务器是这种验证最重要的部分，因为我们真的（真的，真的）需要确保携带证书的用户是有效的。通常，构建这样的复杂服务器是由服务器工程师完成的，而不是开发者的任务，但如果这样的话，先了解一下这个主题会是个好主意。

个人来说，我是 Apache 的粉丝，并建议任何人都去了解`mod_ssl`配置，因为它在保护服务器方面非常全面，并且有很多资源可以帮助你正确配置它。

但最终，如果没有适当的了解，配置 SSL 是一个非常繁琐且容易出错的流程，而且很可能正确配置服务器超出了开发者的能力范围。在这种情况下，让服务器工程师为我们完成这项工作是最好的，也是最偷懒的方法，这样我们就可以专注于我们的工作了！
