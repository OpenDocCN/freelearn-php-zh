# 第十章：部署您的应用程序

> 在我们的应用程序上线并准备好供用户注册并创建帖子之前，我们还有一些步骤要完成。

在本章中，我们将做以下事情来让我们的应用程序运行起来：

+   我们将在 Cloudant 上建立一个帐户，用于存放我们应用的 CouchDB 数据库，并为我们的应用做好准备

+   我们将在我们的项目中添加一个配置类，使用环境变量来驱动我们应用程序的设置

+   我们将创建一个 PHP Fog 帐户来托管我们的应用程序

+   我们将配置 Git 连接到 PHP Fog 的 Git 存储库并部署我们的应用程序

正如您可能期望的那样，在本章中，我们将进行大量的帐户设置和代码调整。

# 在我们开始之前

对于任何应用程序或数据库部署，都有各种各样的选择。每个选项都有其优势和劣势。我想给你一些知识，而不是立即设置服务，以防有一天你想改用其他服务。

在过去的几年里，云已经成为技术行业中最常用和滥用的术语之一。要完全理解云这个术语，您需要阅读大量的研究论文和文章。但是，简单来说，云这个术语描述了从传统的单租户专用托管转变为可扩展的多租户和多平台主机。CouchDB 本身就是一个可扩展的数据库的完美例子，可以实现云架构。我们的应用程序也是云解决方案的一个很好的候选，因为我们没有本地存储任何东西，也没有任何特殊的依赖。

考虑到这一点，我们将使用云服务来托管我们的应用程序和数据库。其中一个额外的好处是，我们将能够让我们的应用程序免费运行，并且只有在我们的应用程序成功后才需要开始付费。这一点真的不错！

让我们快速讨论一下我们将如何处理我们的应用程序和 CouchDB 托管以及我们可用的选项。

## 应用程序托管

在云中托管 Web 应用程序时，有无数种方法可以实现。由于我们不是服务器设置专家，我们希望使用一种设置较少但回报较高的系统。考虑到这一点，我们将使用**平台即服务（PaaS）**。有很多 PaaS 解决方案，但目前，对于 PHP 开发人员来说，最好的选择是 Heroku 和 PHP Fog。

**Heroku** ([`www.heroku.com`](http://www.heroku.com)) 是将 PaaS 推向聚光灯下的创新者。他们使用 Cedar 堆栈支持 PHP 应用程序。但是，由于它不是一个特定于 PHP 的堆栈，对于我们来说，可能更明智选择另一个提供商。

**PHP Fog** ([`www.phpfog.com`](http://www.phpfog.com)) 在我看来，是一个非常专注于 PHP 开发的 PaaS，因为他们非常专注于 PHP。他们支持各种 PHP 应用框架，提供 MySQL 托管（如果您的应用需要），总体上，他们致力于为 PHP 开发人员提供一个稳固的开发环境。

考虑到这一切，PHP Fog 将是我们选择的应用托管解决方案。

## CouchDB 托管

与应用程序托管相比，CouchDB 托管的解决方案要少得多，但幸运的是，它们都是非常稳固的产品。我们将讨论的两种服务是 Cloudant 和 IrisCouch。

**Cloudant** ([`www.cloudant.com`](http://www.cloudant.com)) 是云中 CouchDB 最强大的解决方案之一。他们提供了我们在本书中使用过的熟悉工具，如 Futon 和命令行，还能够根据数据增长的需要进行扩展。Cloudant 特别独特的地方在于，当你的应用程序需要一些特殊功能时，他们提供定制解决方案，而且 Cloudant 是 CouchDB 本身的主要贡献者之一。

**Iris Couch** ([`www.iriscouch.com`](http://www.iriscouch.com)) 也允许在云中免费托管 CouchDB。不幸的是，他们刚刚开始提供 Couchbase 服务器作为他们的基础设施，这是建立在 CouchDB 核心之上的。虽然我非常喜欢 Couchbase 及其对核心 CouchDB 技术的增强，但我们的任务是在本书中只使用 CouchDB。但是，如果你将来需要 Couchbase 的增强功能，那么值得考虑一下 Iris Couch。

因为我以前使用过 Cloudant 并知道它能处理什么，我们将在这个项目中使用它。

总的来说，本章中我们将执行的设置与其他竞争性服务相对类似。因此，如果你决定以后转换，你应该能够很好地处理它，而不会有太多问题。

# 使用 Cloudant 进行数据库托管

在本节中，我们将设置一个 Cloudant 服务器，并准备让我们的应用程序连接到它。需要做的设置很少，而且希望这些步骤对我们在本书初期设置 CouchDB 数据库时所采取的步骤来说是熟悉的。

## 开始使用 Cloudant

创建 Cloudant 账户非常简单，但让我们一起走一遍，以确保我们都在同一页面上。

1.  首先去[`cloudant.com/sign-up/`](http://https://cloudant.com/sign-up/)，你会看到注册页面。![开始使用 Cloudant](img/3586_10_005.jpg)

1.  Cloudant 只需要一些基本信息来创建你的账户。首先输入一个用户名。这将被用作你的唯一标识符和你的 Cloudant 账户的链接。我建议选择像你的名字或公司名这样的东西。

1.  填写页面上的其余信息，当你准备好时，点击页面底部的注册按钮！

你已经完成了，现在你应该看到你的 Cloudant 仪表板。从这里，你可以管理你的账户并创建新的数据库。

![开始使用 Cloudant](img/3586_10_020.jpg)

## 创建一个 _users 数据库

现在我们有了全新的 Cloudant 账户，但我们还没有任何数据库。更糟糕的是，我们甚至还没有我们的`_users`数据库。我们只需要创建一个新的`_users`数据库，Cloudant 会处理剩下的。我们技术上可以通过 Cloudant 的界面完成这个过程，但让我们使用命令行，因为它更加通用。

1.  打开终端。

1.  运行以下命令，并替换两个用户名和一个密码的实例，这样 Cloudant 就知道你是谁以及你要使用的账户是什么：

```php
**curl -X PUT https://username:password@username.cloudant.com/_users** 

```

终端会通过返回成功消息来告诉你数据库已经创建：

```php
**{"ok":true}** 

```

太棒了！你的`_users`数据库现在已经创建。记住，我们还需要另一个叫做`verge`的数据库来存储我们的所有数据。让我们接下来创建`verge`数据库。

## 创建一个 verge 数据库

你需要在你的账户中创建另一个数据库，这次叫做`verge`。

## 来吧，英雄——自己试试看

现在，你应该很容易自己创建另一个数据库。按照我们创建`_users`数据库时所采取的相同步骤来尝试一下，但是将数据库名称改为`verge`。

如果你感到困惑，我马上会向你展示命令行语句。好的，进行得怎么样？让我们回顾一下创建`verge`数据库所需执行的步骤。

1.  打开终端。

1.  你应该运行以下命令，并替换两个用户名实例和一个密码实例，这样 Cloudant 就会知道你是谁，以及你要使用的账户是什么：

```php
**curl -X PUT https://username:password@username.cloudant.com/verge** 

```

当你看到一个熟悉的成功消息时，终端应该已经让你放心一切都进行得很顺利，如下所示：

```php
**{"ok":true}** 

```

## 在 Cloudant 上使用 Futon

通过命令行管理内容可能有点繁琐。幸运的是，Cloudant 还带来了我们的老朋友——Futon。要在 Cloudant 上使用 Futon，按照以下步骤进行：

1.  登录，并转到你的仪表板。![在 Cloudant 上使用 Futon](img/3586_10_025.jpg)

1.  点击你的数据库名称之一；在这个例子中，让我们使用`verge`。![在 Cloudant 上使用 Futon](img/3586_10_030.jpg)

+   这是数据库详细页面——当文档出现在你的数据库中时，它们将显示在这个页面上。

1.  点击 Futon 中的查看按钮继续。

看起来熟悉吗？这就是我们在本地一直在使用的伟大的 Futon。

![在 Cloudant 上使用 Futon](img/3586_10_032.jpg)

## 配置权限

现在我们的生产数据库已经上线，非常重要的是我们要配置权限以在我们的生产服务器上运行。如果我们不保护我们的数据库，那么我们的用户很容易就能读取，这是我们不想卷入的事情。

幸运的是，Cloudant 已经为我们解决了所有这些问题，具体做法如下：

+   因为我们已经创建了一个账户，数据库不再处于`Admin Party`模式

+   默认情况下，Cloudant 使`_users`数据库对我们的`admin`账户可管理，但其他账户无法访问它

我们很幸运，Cloudant 一直支持我们！但是，如果你决定自己部署 CouchDB 实例，一定要回头看看第三章，“使用 CouchDB 和 Futon 入门”，并按照我们用来保护本地环境的步骤进行操作。

然而，我们需要更新我们的`verge`数据库，以便用户可以在该数据库中读取、创建和写入。

1.  登录到你的 Cloudant 账户，并转到你的仪表板。[`cloudant.com/#!/dashboard`](http://https://cloudant.com/#!/dashboard)。

1.  点击`verge`数据库。

1.  点击**权限**来管理数据库权限。

1.  通过选中**读取、创建**和**写入**下的复选框来更新**其他人**的**权限**。确保不要选中**管理员**，这样普通用户就无法更改我们的数据库结构和设计文档。最终结果应该类似于以下截图：![配置权限](img/3586_10_035.jpg)

# 配置我们的项目

现在我们已经设置好了我们的生产数据库，我们的代码需要知道如何连接到它。我们可以只是修改我们在`Bones`库中硬编码的值，每次想要在本地开发或部署到生产环境时来回更改。但是，请相信我，你不想经历这样的麻烦，更重要的是，我们不想在我们的代码中存储任何用户名或密码；为此，我们将使用环境变量。**环境变量**是一组动态命名的值，允许你从应用程序的托管环境中定义变量。让我们创建一个类，使我们能够使用环境变量，这样我们的代码就不会包含敏感信息，我们的应用程序也容易配置。

# 行动时间——创建一个配置类

由于我们迄今为止编写的代码，插入一个简单的配置类实际上对我们来说非常容易。让我们一起来创建它。

1.  首先在我们的`lib`文件夹内创建一个名为`configuration.php`的新配置文件（`lib/configuration.php`）。

1.  现在，让我们为名为`Configuration`的类创建脚手架。

```php
<?php
class Configuration {
}

```

1.  让我们继续并创建一些描述性的配置变量。我们可以添加更多，但现在让我们只添加我们现在需要的。

```php
<?php
class Configuration {
**private $db_server = ';
private $db_port = '';
private $db_database = '';
private $db_admin_user = '';
private $db_admin_password = '';** 
}

```

1.  现在，复制你需要访问本地 CouchDB 实例的登录信息；我的看起来类似于以下内容：

```php
<?php
class Configuration {
**private $db_server = '127.0.0.1';
private $db_port = '5984';
private $db_database = 'verge';
private $db_admin_user = 'tim';
private $db_admin_password = 'test';** 
}

```

1.  让我们使用一个特殊的`__get`函数来检查并查看是否设置了环境变量，并返回该值，而不是默认值。如果没有，它将只返回我们在这个类中定义的默认值。

```php
<?php
class Configuration {
private $db_server = '127.0.0.1';
private $db_port = '5984';
private $db_database = 'verge';
private $db_admin_user = 'tim';
private $db_admin_password = 'test';
**public function __get($property) {
if (getenv($property)) {
return getenv($property);
} else {
return $this->$property;
}
}** 
}

```

## 刚刚发生了什么？

我们刚刚创建了一个名为`configuration.php`的简单配置类，并创建了一个名为`Configuration`的类的框架。接下来，我们为数据库的配置创建了一些变量，我们将其设置为`public`，因为我们可能需要在各种地方使用这些变量。然后，我们用访问本地 CouchDB 实例的信息填充了这些变量的默认值。然后，我们添加了这个类的魔力。我们创建了一个`__get`函数，它覆盖了类的标准`get`操作。这个函数使用`getenv`函数来检查服务器，看看环境变量中是否设置了该变量（我们将很快介绍如何做到这一点）。如果有一个同名的环境变量，我们将把它返回给调用函数；如果没有，我们将简单地返回默认值。

`Configuration`类是一个很好而简单的类，它可以在不过分复杂的情况下完成我们需要的一切。接下来，让我们确保我们的应用程序知道如何访问和使用这个类。

# 行动时间-将我们的配置文件添加到 Bones

将新的配置类添加到我们的应用程序中非常容易。现在，我们只需要将它添加到 Bones 的`__construct()`中，然后我们就可以在整个项目中开始使用这个类了。

1.  打开`lib/bones.php`，并查看文件开头，告诉我们的库在哪里查找其他`lib`文件。我们需要在这里添加我们的配置类。

```php
require_once ROOT . '/lib/bootstrap.php';
require_once ROOT . '/lib/sag/src/Sag.php';
**require_once ROOT . '/lib/configuration.php';** 

```

1.  让我们确保在 Bones 的公共变量中定义`$config`，这样我们在其他文件中也可以使用它们。

```php
class Bones {
private static $instance;
public static $route_found = false;
public $route = '';
public $method = '';
public $content = '';
public $vars = array();
public $route_segments = array();
public $route_variables = array();
public $couch;
**public $config;** 

```

1.  让我们看一下文件中稍后的`__construct()`方法。在这个方法中（就在实例化 Sag 之前），让我们创建一个`Configuration`类的新实例。

```php
public function __construct() {
...
**$this->config = new Configuration();** 
$this->couch = new Sag('127.0.0.1','5984');
$this->couch->setDatabase('verge');
}

```

1.  现在我们的代码知道了配置类，我们只需要把变量放在正确的位置，就可以运行起来了。让我们告诉 Sag 如何使用配置类连接到 CouchDB。

```php
public function __construct() {
$this->route = $this->get_route();
$this->route_segments = explode('/', trim($this->route, '/'));
$this->method = $this->get_method();
$this->config = new Configuration();
**$this->couch = new Sag($this->config->db_server, $this->config->db_port);
$this->couch->setDatabase($this->config->db_database);** 
}

```

1.  还有一些地方需要更新我们的代码，以便使用配置类。记住，我们在`classes/user.php`中有`admin`用户名和密码，用于创建和查找用户。让我们首先看一下`classes/user.php`中的注册函数，清理一下。一旦我们插入我们的配置类，该函数应该类似于以下内容：

```php
public function signup($password) {
$bones = new Bones();
$bones->couch->setDatabase('_users');
**$bones->couch->login($bones->config->db_admin_user, $bones->config->db_admin_password);** 

```

1.  我们需要调整的最后一个地方是`classes/user.php`文件末尾的`get_by_username`函数，以使用`config`类。

```php
public static function get_by_username($username = null) {
$bones = new Bones();
**$bones->couch->login($bones->config->db_admin_user, $bones->config->db_admin_password);** 
$bones->couch->setDatabase('_users');

```

1.  我们刚刚删除了`index.php`顶部定义的`ADMIN_USER`和`ADMIN_PASSWORD`的所有引用。我们不再需要这些变量，所以让我们切换到`index.php`，并从文件顶部删除`ADMIN_USER`和`ADMIN_PASSWORD`。

## 刚刚发生了什么？

我们刚刚写完了我们应用程序的最后几行代码！在这一部分中，我们确保 Bones 完全可以访问我们最近创建的`lib/configuration.php`配置文件。然后，我们创建了一个公共变量`$config`，以确保我们在应用程序的任何地方都可以访问我们的配置类。将我们的配置类存储在`$config`变量中后，我们继续查看我们的代码中那些硬编码了数据库设置的地方。

## 将更改添加到 Git

因为我们刚刚写完了我们的代码的最后几行，我要确保你已经完全提交了我们的所有代码到 Git。否则，当我们不久部署我们的代码时，你的所有文件可能都不会到达生产服务器。

1.  打开终端。

1.  使用通配符添加项目中的任何剩余文件。

```php
**git add .** 

```

1.  现在，让我们告诉 Git 我们做了什么。

```php
**git commit m 'Abstracted out environment specific variables into lib/configuration.php and preparing for launch of our site 1.0!'** 

```

# 使用 PHP Fog 进行应用程序托管

我们的代码已经更新完毕，准备部署。我们只需要一个地方来实际部署它。正如我之前提到的，我们将使用 PHP Fog，但请随意探索其他可用的选项。大多数 PaaS 提供商的设置和部署过程都是相同的。

## 设置 PHP Fog 账户

设置 PHP Fog 账户就像我们设置 Cloudant 账户一样简单。

1.  首先，访问[`www.phpfog.com/signup`](http://https://www.phpfog.com/signup)。![设置 PHP Fog 账户](img/3586_10_045.jpg)

1.  填写每个字段创建一个账户。完成后，点击“注册”。你将被引导创建你的第一个应用程序。![设置 PHP Fog 账户](img/3586_10_050.jpg)

1.  你会注意到有各种各样的起始应用程序和框架，可以让我们快速创建 PHP 应用程序的脚手架。我们将使用我们自己的代码，所以点击“自定义应用程序”。![设置 PHP Fog 账户](img/3586_10_060.jpg)

1.  我们的应用程序几乎创建完成了，我们只需要给 PHP Fog 提供更多信息。

1.  你会注意到 PHP Fog 要求输入 MySQL 的密码。由于我们在这个应用程序中没有使用 MySQL，我们可以输入一个随机密码或其他任何字符。值得一提的是，如果将来有一天你想在项目中使用 MySQL 来存储一些关系数据，只需点击几下，就可以在同一应用程序环境中进行托管。记住，如果正确使用，MySQL 和 CouchDB 可以成为最好的朋友！

1.  接下来，PHP Fog 会要求输入你的域名。每个应用程序都会有一个托管在[phpfogapp.com](http://phpfogapp.com)上的短 URL。这对我们来说在短期内是完全可以接受的，当我们准备使用完整域名推出我们的应用程序时，我们可以通过 PHP Fog 的“域名”部分来实现。在为应用程序创建域名时，PHP Fog 要求它是唯一的，所以你需要想出自己的域名。你可以使用类似`yourname-verge.phpfogapp.com`的形式，或者你可以特别聪明地创建一个以你最喜欢的神话生物命名的应用程序。这是一个常见的做法，这样在你还在修复错误和准备推出时，没有人会随机找到你的应用程序。![设置 PHP Fog 账户](img/3586_10_065.jpg)

1.  当你准备好时，点击“创建应用程序”，你的应用程序将被创建。![设置 PHP Fog 账户](img/3586_10_070.jpg)

1.  就是这样！你的应用程序已经准备就绪。你会注意到 PHP Fog 会在短暂的时间内显示“状态：准备应用...”，然后会变成“状态：运行”。![设置 PHP Fog 账户](img/3586_10_072.jpg)

## 创建环境变量

我们的 PHP Fog 应用程序已经启动运行，我们在将代码推送到服务器之前需要进行最后一项配置。记得我们在配置项目时设置的所有环境变量吗？好吧，我们需要在 PHP Fog 中设置它们，这样我们的应用程序就知道如何连接到 Cloudant 了。

为了管理你的环境变量，你需要首先转到你项目的“应用程序控制台”，这是你创建第一个应用程序后留下的地方。

点击“环境变量”，你将进入“环境变量管理”部分。

![创建环境变量](img/3586_10_075.jpg)

你会注意到 PHP Fog 为我们创建的 MySQL 数据库的环境变量已经设置好了。我们只需要输入 Cloudant 的环境变量。名称需要与我们在本章前面定义的配置类中的名称相同。

让我们从添加我们的`db_server`环境变量开始。我的`db_server`位于`https://timjuravich:password@timjuravich.cloudant.com`，所以我将这些详细信息输入到**名称**和**值**文本字段中。

![创建环境变量](img/3586_10_080.jpg)

让我们继续为配置文件中的每个变量进行此过程。回顾一下，这里是您需要输入的环境变量：

+   `db_server:` 这将是您的 Cloudant URL，同样，我的是`https://timjuravich:password@timjuravich.cloudant.com`

+   `db_port:` 这将设置为`5984`

+   `db_database:` 这是将存储所有内容的数据库，应设置为`verge`

+   `db_admin_user:` 这是`admin`用户的用户名。在我们的情况下，这是设置为 Cloudant 管理员用户名的值

+   `db_admin_password:` 这是上述`admin`用户的密码

当您完成所有操作后，点击**保存更改**，您的环境变量将被设置。有了这个，我们就可以部署到 PHP Fog 了。

## 部署到 PHP Fog

部署到 PHP Fog 是一个非常简单的过程，因为 PHP Fog 使用 Git 进行部署。很幸运，我们的项目已经使用 Git 设置好并准备就绪。我们只需要告诉 PHP Fog 我们的 SSH 密钥，这样它就知道如何识别我们。

### 将我们的 SSH 密钥添加到 PHP Fog

PHP Fog 使用 SSH 密钥来识别和验证我们，就像 GitHub 一样。由于我们在本书的早期已经创建了一个，所以我们不需要再创建一个。

1.  您可以从右上角点击**我的帐户**开始，然后在下一页上点击**SSH 密钥**。您将看到以下页面，您可以在其中输入您的 SSH 密钥:![将我们的 SSH 密钥添加到 PHP Fog](img/3586_10_082.jpg)

1.  输入**昵称**的值。您应该使用简单但描述性的内容，比如`Tim's Macbook`。将来，您会因为保持这种组织而感激自己，特别是如果您开始与其他开发人员合作开发这个项目。

您需要获取公钥以填入公钥文本框。幸运的是，我们可以在终端中用一个简单的命令来做到这一点。

1.  打开终端。

1.  运行以下命令，您的公钥将被复制到剪贴板。

```php
**pbcopy< ~/.ssh/id_rsa.pub** 

```

1.  将公钥复制到剪贴板后，只需点击文本框，粘贴值进去。

1.  最后，在表单底部有一个复选框，上面写着**给此密钥写入访问权限**。如果您希望计算机能够将代码推送到 PHP Fog（我们希望能够这样做），则需要勾选此复选框。

1.  点击**保存 SSH 密钥**，我们就可以继续进行部署应用程序的最后步骤了。

### 连接到 PHP Fog 的 Git 存储库

由于我们已经设置好并准备好使用的 Git 存储库，我们只需要告诉 Git 如何连接到 PHP Fog 上的存储库。让我们通过向我们的工作目录添加一个名为`phpfog`的远程存储库来完成这个过程。

#### 从 Php Fog 获取存储库

当我们在 PHP Fog 上创建应用程序时，我们还创建了一个独特的 Git 存储库，我们的应用程序由此驱动。在本节中，我们将获取此存储库的位置，以便告诉 Git 连接到它。

1.  登录到您的 PHP Fog 帐户。

1.  转到您的应用程序的应用控制台。

1.  点击**源代码**。

1.  在**源代码**页面上，您将看到一个部分，上面写着**克隆您的 git 存储库**。我的里面有以下代码（您的应该类似）:

```php
**git clone git@git01.phpfog.com:timjuravich-verge.phpfogapp.com** 

```

1.  因为我们已经有一个现有的 Git 存储库，所以我们不需要克隆他们的，但是我们需要应用程序的 Git 存储库的位置来进行下一步配置。使用这个例子，存储库位置将是`git@git01.phpfog.com:timjuravich-verge.phpfogapp.com`。将其复制到剪贴板上。

#### 从 Git 连接到存储库

现在我们知道了 PHP Fog 的 Git 存储库，我们只需要告诉我们的本地机器如何连接到它。

1.  打开终端。

1.  将目录更改为您的`工作`文件夹。

```php
**cd /Library/WebServer/Documents/verge** 

```

1.  现在，让我们将 PHP Fog 的存储库添加为一个名为`phpfog`的新远程存储库。

```php
**git remote add phpfog git@git01.phpfog.com:verge.phpfogapp.com** 

```

1.  清除跑道，我们准备启动这个应用程序！

### 部署到 PHP Fog

这就是我们一直在等待的时刻！让我们将我们的应用程序部署到 PHP Fog。

1.  打开终端。

1.  将目录更改为您的`working`文件夹。

```php
**cd /Library/WebServer/Documents/verge** 

```

1.  我们希望忽略 PHP Fog 的 Git 存储库中的内容，因为我们已经构建了我们的应用程序。因此，这一次，我们将在调用的末尾添加`--force`。

```php
**git push origin master --force** 

```

我希望这不会太令人失望，但恭喜，您的应用程序已经上线了！这是不是很简单？从现在开始，每当您对代码进行更改时，您只需要将其提交到 Git，输入命令`git push phpfog master`，并确保通过`git push origin master`将您的代码推送到 GitHub。

如果您开始对您的实时应用程序进行一些操作，您可能会感到沮丧，因为您的本地机器上的数据并不适合您查看。您很幸运；在下一节中，我们将使用 CouchDB 强大的复制功能将本地数据库推送到我们的生产数据库。

# 将本地数据复制到生产环境

复制的内部工作原理和背景信息将不会在本节中详细介绍，但您可以在 Packt Publishing 网站上的名为*复制您的数据*的奖励章节中找到完整的演练。

为了给您一个快速概述，**复制**是 CouchDB 在服务器之间传输数据的方式。复制由每个文档中的`_rev`字段驱动，`_rev`字段确保您的服务器知道哪个版本具有正确的数据可供使用。

在本节中，我们将复制`_users`和`verge`数据库，以便我们所有的本地数据都可以在生产服务器上使用。如果您的应用程序已经上线了几分钟甚至几天，您不必担心，因为复制的最大好处是，如果有人已经在使用您的应用程序，那么他们的所有数据将保持完整；我们只是添加我们的本地数据。

# 执行操作-将本地 _users 数据库复制到 Cloudant

让我们使用 Futon 将我们的本地`_users`数据库复制到我们在 Cloudant 上创建的`_users`数据库。

1.  在浏览器中打开 Futon，单击**复制器**，或者您可以直接导航到`http://localhost:5984/_utils/replicator.html`。

1.  确保您以`管理员`身份登录；如果没有，请单击**登录**，以`管理员`身份登录。

1.  在**从中复制更改**的下拉列表中选择`_users`数据库。

1.  在**To**部分中单击**远程数据库**单选按钮。![执行操作-将本地 _users 数据库复制到 Cloudant](img/3586_10_085.jpg)

1.  在**远程数据库**文本字段中，输入 Cloudant 数据库的 URL 以及凭据。 URL 的格式看起来类似于`https://username:password@username.cloudant.com/_users`。![执行操作-将本地 _users 数据库复制到 Cloudant](img/3586_10_090.jpg)

1.  单击**复制**，CouchDB 将推送您的本地数据库到 Cloudant。

1.  您将看到 Futon 的熟悉结果。![执行操作-将本地 _users 数据库复制到 Cloudant](img/3586_10_095.jpg)

## 刚刚发生了什么？

我们刚刚使用 Futon 将我们的本地`_users`数据库复制到了我们在 Cloudant 上托管的`_users`生产数据库。这个过程与我们之前做的完全相同，但是我们在**To**部分使用了**远程数据库**，并使用了数据库的 URL 以及我们的凭据。复制完成后，我们收到了一个冗长且令人困惑的报告，但要点是一切都进行得很顺利。让我们继续复制我们的`verge`数据库。

### 注意

值得一提的是，如果你尝试从命令行复制`_users`数据库，你将不得不在调用中包含用户名和密码。这是因为我们完全将用户数据库锁定为匿名用户。函数看起来类似于以下内容：

curl -X POST http://user:password@localhost:5984/_replicate -d '{"source":"_users",*"target":"https://username:password@username.cloudant.com/_users"}'* -H*"Content-Type:* application/json"

## 试一试英雄——将本地的 verge 数据库复制到 Cloudant

你认为你能根据我刚给你的提示找出将本地`verge`数据库复制到 Cloudant 上的`verge`数据库的命令吗？在游戏的这个阶段几乎不可能搞砸任何事情，所以如果第一次没有搞定，不要害怕尝试几次。

试一试。完成后，继续阅读，我们将讨论我使用的命令。

一切进行得如何？希望你能轻松搞定。如果你无法让它工作，这里有一个你可以使用的命令示例：

```php
curl -X POST http://user:password@localhost:5984/_replicate -d '{"source":"verge","target":"https://username:password@username .cloudant.com/verge"}' -H "Content-Type: application/json"

```

在这个例子中，我们使用我们的本地 CouchDB 实例将本地的`verge`数据库复制到目标 Cloudant`verge`数据库。对于本地数据库，我们可以简单地将名称设置为`verge`，但对于目标数据库，我们必须传递完整的数据库位置。

当你的所有数据都在生产服务器上并且在线时，你可以登录为你在本地创建的任何用户，并查看你创建的所有内容都已经准备好供全世界查看。这并不是你旅程的结束；让我们快速谈谈接下来的事情。

# 接下来是什么？

在我送你离开之前，让我们谈谈你的应用在野外的前景，以及你可以做些什么来使这个应用更加强大。

## 扩展你的应用

幸运的是，在利用 PHPFog 和 Cloudant 时，扩展你的应用应该是非常容易的。实际上，你唯一需要做的最紧张的事情就是登录 PHPFog 并增加我们的 Web 进程，或者登录 Cloudant 并升级到更大的计划。他们处理所有的艰苦工作；你只需要学会如何有效地扩展。这是无法超越的！

要了解更多关于有效扩展的信息，请浏览 PHPFog 和 Cloudant 的帮助文档，它们详细介绍了不同的扩展方式和需要避免的问题领域。

值得再次提到的是，我们在本章中并没有完全涵盖复制。要全面了解复制，请务必查看题为*复制数据*的奖励章节，该章节可在 Packt Publishing 网站上找到。

## 下一步

我希望你继续开发和改进 Verge，使其成为非常有用的东西，或者，如果不是，我希望你利用这本书学到的知识构建更伟大的东西。

如果你决定继续在 Verge 上构建功能，这个应用还有很多可以做的事情。例如，你可以：

+   添加用户之间相互关注的功能

+   允许用户过滤和搜索内容

+   添加一个消息系统，让用户可以相互交流

+   自定义 UI，使其成为真正独特的东西

我将继续在 GitHub 上的 Verge 存储库中逐步添加这样的功能和更多功能：[`github.com/timjuravich/verge`](http://https://github.com/timjuravich/verge)。所以，请确保关注存储库的更新，并在需要时进行分叉。

再次感谢你在这本书中花费的时间，请随时在 Twitter 上联系我`@timjuravich`，如果你有任何问题。

开心开发！

# 总结

在本章中，我们学会了如何与世界分享我们的应用。具体来说，我们注册了 Cloudant 和 PHP Fog 的帐户，并成功部署了我们的应用。你所要做的就是继续编码，将这个应用变成一些了不起的东西。
