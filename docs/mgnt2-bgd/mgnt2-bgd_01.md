# 第一章：理解平台架构

**Magento**是一个强大、高度可扩展和高度可定制的电子商务平台，可用于构建网店，如果需要，还可以用于一些非电子商务网站。它提供了大量开箱即用的电子商务功能。

产品库存、购物车、支持多种支付和运输方式、促销规则、内容管理、多种货币、多种语言、多个网站等功能使其成为商家的绝佳选择。另一方面，开发者享受与商人相关的完整功能集以及与实际开发相关的所有事物。本章将涉及强大的 Web API 支持、可扩展的管理界面、模块、主题、嵌入式测试框架等内容。

在本章中，以下部分提供了对 Magento 的高级概述：

+   技术栈

+   架构层

+   最高级别的文件系统结构

+   模块文件系统结构

# 技术栈

Magento 高度模块化的结构是几个开源技术嵌入到堆栈中的结果。这些开源技术由以下组件组成：

+   **PHP**：PHP 是一种服务器端脚本语言。本书假设您对 PHP 的面向对象方面有深入的了解，这通常被称为**PHP OOP**。

+   **编码规范**：Magento 非常重视编码规范。这包括**PSR-0**（自动加载标准）、**PSR-1**（基本编码规范）、**PSR-2**（编码风格指南）、**PSR-3**和**PSR-4**。

+   **Composer**：Composer 是一个 PHP 的依赖管理包。它用于拉取所有供应商库的要求。

+   **HTML**：HTML5 是开箱即支持的。

+   **CSS**：Magento 通过其内置的**LESS CSS**预处理器支持 CSS3。

+   **jQuery**：jQuery 是一个成熟的跨平台 JavaScript 库，旨在简化 DOM 操作。它是当今最受欢迎的 JavaScript 框架之一。

+   **RequireJS**：RequireJS 是一个 JavaScript 文件和模块加载器。使用如 RequireJS 这样的模块化脚本加载器有助于提高代码的速度和质量。

+   **第三方库**：Magento 内置了许多第三方库，其中最显著的是**Zend Framework**和**Symfony**。值得注意的是，Zend Framework 有两个不同的主要版本，即 1.x 版本和 2.x 版本。Magento 内部使用这两个版本。

+   **Apache 或 Nginx**：Apache 和 Nginx 都是 HTTP 服务器。每个都有自己的优缺点。说一个比另一个好是不公平的，因为它们的性能广泛取决于整个系统的设置和使用。Magento 与 Apache 2.2 和 2.4 以及 Nginx 1.7 兼容。

+   **MySQL**：MySQL 是一个成熟且广泛使用的**关系数据库管理系统**（**RDBMS**），它使用**结构化查询语言**（**SQL**）。MySQL 既有免费社区版本，也有商业版本。Magento 至少需要**MySQL 社区版**5.6 版本。

+   **MTF**：**Magento 测试框架**（**MTF**）提供了一套自动化测试套件。它涵盖了各种类型的测试，如性能测试、功能测试和单元测试。整个 MTF 都可在 GitHub 上找到，可以通过访问[`github.com/magento/mtf`](https://github.com/magento/mtf)作为一个独立的项目进行查看。

不同的技术可以粘合到各种架构中。从模块开发者、系统集成商或商家，或者从其他角度看待 Magento 架构的方式有很多。

# 架构层

从上到下，Magento 可以分为四个架构层，即**表示层**、**服务层**、**领域层**和**持久层**。

**表示层**是我们通过浏览器直接与之交互的那一层。它包含布局、块、模板，甚至控制器，这些控制器处理用户界面的命令。jQuery、RequireJS、CSS 和 LESS 等客户端技术也是这一层的一部分。通常，三种类型的用户与这一层交互，即网络用户、系统管理员和进行 Web API 调用的用户。由于 Web API 调用可以通过与用户使用浏览器相同的方式进行 HTTP 调用，因此两者之间有一条很细的界限。虽然网络用户和 Web API 调用按原样消耗表示层，但系统管理员有权对其进行更改。这种更改以设置活动主题和更改**CMS**（即**内容管理系统**）页面、块和产品本身的内容的形式体现。

当与表示层的组件进行交互时，它们通常会调用底层的服务层。

**服务层**是表示层和领域层之间的桥梁。它包含服务合约，这些合约定义了实现行为。**服务合约**基本上是一个 PHP 接口的别称。这一层是我们可以找到 REST/SOAP API 的地方。大多数用户在店面上的交互都是通过服务层路由的。同样，进行 REST/SOAP API 调用的外部应用程序也与这一层进行交互。

当与服务层的组件进行交互时，它们通常会调用底层的领域层。

*域名*层实际上是 Magento 的业务逻辑。这一层完全是关于组成业务逻辑的通用数据对象和模型。域名层模型本身不参与数据持久化，但它们包含一个指向资源模型的引用，该模型用于从 MySQL 数据库检索和持久化数据。一个模块的域名层代码可以通过使用*事件观察者*、*插件*和*di.xml*定义与另一个模块的域名模块代码进行交互。我们将在后面的章节中探讨这些细节。鉴于插件和 di.xml 的强大功能，重要的是要注意，这种交互最好通过服务合同（PHP 接口）来建立。

当与域名层的组件进行交互时，它们通常会调用底层的持久化层。

*持久化*层是数据被持久化的地方。这一层负责所有的**CRUD**（即**创建、读取、更新和删除**）请求。Magento 使用持久化层的活动记录模式策略。模型对象包含一个资源模型，它将一个对象映射到一个或多个数据库行。在这里，区分简单资源模型和**实体-属性-值**（**EAV**）资源模型的情况很重要。简单资源模型映射到单个表，而 EAV 资源模型将它们的属性分散在多个 MySQL 表中。例如，`Customer` 和 `Catalog` 资源模型使用 EAV 资源模型，而新闻通讯的 `Subscriber` 资源模型使用简单资源模型。

# 顶级文件系统结构

以下列表描述了根 Magento 文件系统结构：

+   `.htaccess`

+   `.htaccess.sample`

+   `.php_cs`

+   `.travis.yml`

+   `CHANGELOG.md`

+   `CONTRIBUTING.md`

+   `CONTRIBUTOR_LICENSE_AGREEMENT.html`

+   `COPYING.txt`

+   `Gruntfile.js`

+   `LICENSE.txt`

+   `LICENSE_AFL.txt`

+   `app`

+   `bin`

+   `composer.json`

+   `composer.lock`

+   `dev`

+   `index.php`

+   `lib`

+   `nginx.conf.sample`

+   `package.json`

+   `php.ini.sample`

+   `phpserver`

+   `pub`

+   `setup`

+   `update`

+   `var`

+   `vendor`

`app/etc/di.xml` 文件是我们可能在开发过程中经常查看的最重要文件之一。它包含各种类映射或单个接口的偏好设置。

`var/magento/language-*` 目录是注册语言所在的位置。尽管每个模块都可以在 `app/code/{VendorName}/{ModuleName}/i18n/` 下声明自己的翻译，但如果在自定义模块或主题目录中找不到翻译，Magento 最终会回退到其自己的名为 `i18n` 的单独模块。

`bin`目录是我们可以找到`magento`文件的地方。`magento`文件是一个旨在从控制台运行的脚本。一旦通过`php bin/magento`命令触发，它将运行`Magento\Framework\Console\Cli`应用程序的一个实例，向我们提供相当多的控制台选项。我们可以使用`magento`脚本来启用/禁用缓存，启用/禁用模块，运行索引器，以及执行许多其他操作。

`dev`目录是我们可以找到 Magento 测试脚本的地方。我们将在后面的章节中了解更多关于这些脚本的内容。

`lib`目录包含两个主要子目录，即位于`lib/internal`下的服务器端 PHP 库代码和位于`lib/web`中的客户端 JavaScript 库。

`pub`目录是公开文件所在的位置。这是我们在设置 Apache 或 Nginx 时应将其设置为根目录的目录。当在浏览器中打开店面时，会触发`pub/index.php`文件。

`var`目录是动态生成的文件类型，如缓存、日志等文件创建的地方。我们应该能够随时删除此文件夹的内容，并且让 Magento 自动重新创建它。

`vendor`目录是大多数代码所在的地方。这是我们可以找到各种第三方供应商代码、Magento 模块、主题和语言包的地方。进一步查看`vendor`目录，你会看到以下结构：

+   `.htaccess`

+   `autoload.php`

+   `bin`

+   `braintree`

+   `composer`

+   `doctrine`

+   `fabpot`

+   `justinrainbow`

+   `league`

+   `lusitanian`

+   `magento`

+   `monolog`

+   `oyejorge`

+   `pdepend`

+   `pelago`

+   `phpmd`

+   `phpseclib`

+   `phpunit`

+   `psr`

+   `sebastian`

+   `seld`

+   `sjparkinson`

+   `squizlabs`

+   `symfony`

+   `tedivm`

+   `tubalmartin`

+   `zendframework`

在供应商目录中，我们可以找到来自各种供应商的代码，例如`phpunit`、`phpseclib`、`monolog`、`symfony`等。Magento 本身也可以在这里找到。Magento 代码位于`vendor/magento`目录下，部分列表如下：

+   `composer`

+   `framework`

+   `language-en_us`

+   `magento-composer-installer`

+   `magento2-base`

+   `module-authorization`

+   `module-backend`

+   `module-catalog`

+   `module-customer`

+   `module-theme`

+   `module-translation`

+   `module-ui`

+   `module-url-rewrite`

+   `module-user`

+   `module-version`

+   `module-webapi`

+   `module-widget`

+   `theme-adminhtml-backend`

+   `theme-frontend-blank`

+   `theme-frontend-luma`

你会发现目录的进一步结构遵循某种命名模式，其中`theme-*`目录存储主题，`module-*`目录存储模块，而`language-*`目录存储已注册的语言。

# 模块文件系统结构

Magento 将自己定位为一个高度模块化的平台。这意味着模块被放置的目录位置是实际存在的。现在让我们看一下单个模块的结构。以下结构属于一个较简单的核心 Magento 模块——可以在`vendor/magento/module-contact`中找到的`Contact`模块：

+   `Block`

+   `composer.json`

+   `Controller`

+   `etc`

    +   `acl.xml`

    +   `adminhtml`

        +   `system.xml`

    +   `config.xml`

    +   `email_templates.xml`

    +   `frontend`

        +   `di.xml`

        +   `page_types.xml`

        +   `routes.xml`

    +   `module.xml`

+   `Helper`

+   `i18n`

+   `LICENSE_AFL.txt`

+   `LICENSE.txt`

+   `Model`

+   `README.md`

+   `registration.php`

+   `Test`

    +   `Unit`

        +   `Block`

        +   `Controller`

        +   `Helper`

        +   `Model`

+   `view`

    +   `adminhtml`

    +   `frontend`

        +   `layout`

        +   `contact_index_index.xml`

        +   `default.xml`

    +   `templates`

        +   `form.phtml`

尽管前面的结构是针对一个较简单的模块，但你可以看到它仍然相当广泛。

`Block`目录是存储与视图相关的 PHP 类的地方。

`Controller`目录是存储与控制器相关的 PHP 类的地方。这是响应店面`POST`和`GET HTTP`操作的代码。

`etc`目录是模块配置文件所在之处。在这里，我们可以看到诸如`module.xml`、`di.xml`、`acl.xml`、`system.xml`、`config.xml`、`email_templates.xml`、`page_types.xml`、`routes.xml`等文件。`module.xml`文件是一个实际的模块声明文件。我们将在后面的章节中查看这些文件的内容。

`Helper`目录是各种辅助类所在之处。这些类通常用于将各种商店配置值抽象到获取方法中。

`i18n`目录是存储模块翻译包 CSV 文件的地方。

`Module`目录是实体、资源实体、集合和各种其他业务类可以找到的地方。

测试目录存储模块单元测试。

`view`目录包含所有模块管理员和店面模板文件（`.phtml`和`.html`）和静态文件（`.js`和`.css`）。

最后，`registration.php`是一个模块注册文件。

# 摘要

在本章中，我们快速浏览了在 Magento 中使用的技术栈。我们讨论了作为一个开源产品的 Magento 如何广泛使用其他开源项目和服务，如 MySQL、Apache、Nginx、Zend Framework、Symfony、jQuery 等。然后我们学习了这些库是如何组织到目录中的。最后，我们探索了一个现有的核心模块，并简要地查看了一个模块结构的示例。

在下一章中，我们将处理环境设置，以便我们可以安装并准备开发 Magento。
