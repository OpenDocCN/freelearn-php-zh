# 第三章. 编程概念和约定

凭借多年的经验，Magento 平台发展起来，实现了许多行业概念、标准和约定。在本章中，我们将探讨几个在日常与 Magento 开发互动中突出的独立部分。

在本章中，我们将讨论以下部分：

+   Composer

+   服务合同

+   代码生成

+   `var`目录

+   编码标准

# Composer

**Composer**是一个处理 PHP 依赖管理的工具。它不像 Linux 系统中的**Yum**和**Apt**那样是一个包管理器。尽管它处理库（包），但它是在每个项目级别上处理的。它不会全局安装任何东西。Composer 是一个多平台工具。因此，它在 Windows、Linux 和 OS X 上运行得同样好。

在机器上安装 Composer 就像在项目目录中运行安装程序一样简单，使用以下命令即可：

```php
curl -sS https://getcomposer.org/installer | php

```

更多关于 Composer 安装的信息可以在其官方网站上找到，可以通过访问[`getcomposer.org`](https://getcomposer.org)查看。

Composer 用于获取 Magento 及其使用的第三方组件。如前一章所见，以下`composer`命令是将所有内容拉入指定目录的命令：

```php
composer create-project --repository-url=https://repo.magento.com/ magento/project-enterprise-edition <installation directory name>

```

一旦下载并安装了 Magento，你可以在其目录中找到许多`composer.json`文件。假设`<安装目录名称>`是`magento2`，如果我们执行如`find magento2/ -name 'composer.json'`之类的快速搜索命令，那么将产生超过 100 个`composer.json`文件。其中一些文件（部分）如下所示：

```php
/vendor/magento/module-catalog/composer.json
/vendor/magento/module-cms/composer.json
/vendor/magento/module-contact/composer.json
/vendor/magento/module-customer/composer.json
/vendor/magento/module-sales/composer.json
/...
/vendor/magento/theme-adminhtml-backend/composer.json
/vendor/magento/theme-frontend-blank/composer.json
/vendor/magento/theme-frontend-luma/composer.json
/vendor/magento/language-de_de/composer.json
/vendor/magento/language-en_us/composer.json
/...
/composer.json
/dev/tests/...
/vendor/magento/framework/composer.json
```

最相关的文件可能是位于`magento`目录根目录下的`composer.json`文件。其内容看起来像这样：

```php
{
    "name": "magento/project-community-edition",
    "description": "eCommerce Platform for Growth (Community Edition)",
    "type": "project",
    "version": "2.0.0",
    "license": [
        "OSL-3.0",
        "AFL-3.0"
    ],
    "repositories": [
        {
            "type": "composer",
            "url": "https://repo.magento.com/"
        }
    ],
    "require": {
        "magento/product-community-edition": "2.0.0",
        "composer/composer": "@alpha",
        "magento/module-bundle-sample-data": "100.0.*",
        "magento/module-widget-sample-data": "100.0.*",
        "magento/module-theme-sample-data": "100.0.*",
        "magento/module-catalog-sample-data": "100.0.*",
        "magento/module-customer-sample-data": "100.0.*",
        "magento/module-cms-sample-data": "100.0.*",
        "magento/module-catalog-rule-sample-data": "100.0.*",
        "magento/module-sales-rule-sample-data": "100.0.*",
        "magento/module-review-sample-data": "100.0.*",
        "magento/module-tax-sample-data": "100.0.*",
        "magento/module-sales-sample-data": "100.0.*",
        "magento/module-grouped-product-sample-data": "100.0.*",
        "magento/module-downloadable-sample-data": "100.0.*",
        "magento/module-msrp-sample-data": "100.0.*",
        "magento/module-configurable-sample-data": "100.0.*",
        "magento/module-product-links-sample-data": "100.0.*",
        "magento/module-wishlist-sample-data": "100.0.*",
        "magento/module-swatches-sample-data": "100.0.*",
        "magento/sample-data-media": "100.0.*",
        "magento/module-offline-shipping-sample-data": "100.0.*"
    },
    "require-dev": {
        "phpunit/phpunit": "4.1.0",
        "squizlabs/php_codesniffer": "1.5.3",
        "phpmd/phpmd": "@stable",
        "pdepend/pdepend": "2.0.6",
        "sjparkinson/static-review": "~4.1",
        "fabpot/php-cs-fixer": "~1.2",
        "lusitanian/oauth": "~0.3 <=0.7.0"
    },
    "config": {
        "use-include-path": true
    },
    "autoload": {
        "psr-4": {
            "Magento\\Framework\\": "lib/internal/Magento/Framework/",
            "Magento\\Setup\\": "setup/src/Magento/Setup/",
            "Magento\\": "app/code/Magento/"
        },
        "psr-0": {
            "": "app/code/"
        },
        "files": [
            "app/etc/NonComposerComponentRegistration.php"
        ]
    },
    "autoload-dev": {
        "psr-4": {
            "Magento\\Sniffs\\": "dev/tests/static/framework/Magento/Sniffs/",
            "Magento\\Tools\\": "dev/tools/Magento/Tools/",
            "Magento\\Tools\\Sanity\\": "dev/build/publication/sanity/ Magento/Tools/Sanity/",
            "Magento\\TestFramework\\Inspection\\": "dev/tests/static/framework/Magento/ TestFramework/Inspection/",
            "Magento\\TestFramework\\Utility\\": "dev/tests/static/framework/Magento/ TestFramework/Utility/"
        }
    },
    "minimum-stability": "alpha",
    "prefer-stable": true,
    "extra": {
        "magento-force": "override"
    }
}
```

Composer 的 JSON 文件遵循一定的模式。你可以在[`getcomposer.org/doc/04-schema.md`](https://getcomposer.org/doc/04-schema.md)找到这个模式的详细文档。遵循该模式确保了 composer 文件的正确性。我们可以看到，所有列出的键，如`name`、`description`、`require`、`config`等，都是由该模式定义的。

让我们看看单个模块的`composer.json`文件。一个具有最少依赖的简单模块是`Contact`模块，其`vendor/magento/module-contact/composer.json`内容如下所示：

```php
{
    "name": "magento/module-contact",
    "description": "N/A",
    "require": {
        "php": "~5.5.0|~5.6.0|~7.0.0",
        "magento/module-config": "100.0.*",
        "magento/module-store": "100.0.*",
        "magento/module-backend": "100.0.*",
        "magento/module-customer": "100.0.*",
        "magento/module-cms": "100.0.*",
        "magento/framework": "100.0.*"
    },
    "type": "magento2-module",
    "version": "100.0.2",
    "license": [
        "OSL-3.0",
        "AFL-3.0"
    ],
    "autoload": {
        "files": [
            "registration.php"
        ],
        "psr-4": {
            "Magento\\Contact\\": ""
        }
    }
}
```

你会看到模块定义了对 PHP 版本和其他模块的依赖。此外，你还会看到 PSR-4 用于自动加载以及直接加载`registration.php`文件。

接下来，让我们看看`en_us`语言模块中的`vendor/magento/language-en_us/composer.json`文件的内容：

```php
{
    "name": "magento/language-en_us",
    "description": "English (United States) language",
    "version": "100.0.2",
    "license": [
        "OSL-3.0",
        "AFL-3.0"
    ],
    "require": {
        "magento/framework": "100.0.*"
    },
    "type": "magento2-language",
    "autoload": {
        "files": [
            "registration.php"
        ]
    }
}
```

最后，让我们看看`luma`主题中的`vendor/magento/theme-frontend-luma/composer.json`文件的内容：

```php
{
    "name": "magento/theme-frontend-luma",
    "description": "N/A",
    "require": {
        "php": "~5.5.0|~5.6.0|~7.0.0",
        "magento/theme-frontend-blank": "100.0.*",
        "magento/framework": "100.0.*"
    },
    "type": "magento2-theme",
    "version": "100.0.2",
    "license": [
        "OSL-3.0",
        "AFL-3.0"
    ],
    "autoload": {
        "files": [
            "registration.php"
        ]
    }
}
```

如前所述，在 Magento 中散布着许多更多的 composer 文件。

# 服务合同

**服务合约**是一组由模块定义的 PHP 接口。此合约包括数据接口和服务接口。

数据接口的作用是保持数据完整性，而服务接口的作用是隐藏业务逻辑细节，使其不被服务消费者看到。

**数据接口**定义了各种功能，如验证、实体信息、搜索相关功能等。它们定义在单个模块的`Api/Data`目录中。为了更好地理解其真正含义，让我们看看`Magento_Cms`模块的数据接口。在`vendor/magento/module-cms/Api/Data/`目录中，定义了四个接口，如下所示：

```php
BlockInterface.php
BlockSearchResultsInterface.php
PageInterface.php
PageSearchResultsInterface.php
```

`CMS`模块实际上处理两个实体，一个是`Block`，另一个是`Page`。查看前面代码中定义的接口，我们可以看到我们有一个针对实体的单独数据接口和针对搜索结果的单独数据接口。

让我们更仔细地看看`BlockInterface.php`文件的内容（已去除），其定义如下：

```php
namespace Magento\Cms\Api\Data;

interface BlockInterface
{
    const BLOCK_ID      = 'block_id';
    const IDENTIFIER    = 'identifier';
    const TITLE         = 'title';
    const CONTENT       = 'content';
    const CREATION_TIME = 'creation_time';
    const UPDATE_TIME   = 'update_time';
    const IS_ACTIVE     = 'is_active';

    public function getId();
    public function getIdentifier();
    public function getTitle();
    public function getContent();
    public function getCreationTime();
    public function getUpdateTime();
    public function isActive();
    public function setId($id);
    public function setIdentifier($identifier);
    public function setTitle($title);
    public function setContent($content);
    public function setCreationTime($creationTime);
    public function setUpdateTime($updateTime);
    public function setIsActive($isActive);
}
```

前面的接口定义了当前实体的所有获取器和设置器方法，以及表示实体字段名的常量值。这些数据接口不包括管理操作，如`delete`。这个特定接口的实现可以在`vendor/magento/module-cms/Model/Block.php`文件中看到，其中这些常量被使用，如下（部分）所示：

```php
public function getTitle()
{
    return $this->getData(self::TITLE);
}

public function setTitle($title)
{
    return $this->setData(self::TITLE, $title);
}
```

服务接口包括管理、仓库和元数据接口。这些接口直接定义在模块的`Api`目录中。回顾`Magento Cms`模块，其`vendor/magento/module-cms/Api/`目录中有两个服务接口，定义如下：

```php
BlockRepositoryInterface.php
PageRepositoryInterface.php
```

快速查看`BlockRepositoryInterface.php`的内容，揭示以下（部分）内容：

```php
namespace Magento\Cms\Api;

use Magento\Framework\Api\SearchCriteriaInterface;

interface BlockRepositoryInterface
{
    public function save(Data\BlockInterface $block);
    public function getById($blockId);
    public function getList(SearchCriteriaInterface $searchCriteria);
    public function delete(Data\BlockInterface $block);
    public function deleteById($blockId);
}
```

在这里，我们看到用于保存、检索、搜索和删除实体的方法。

这些接口随后通过 Web API 定义实现，正如我们将在第九章中看到的，结果是定义良好且持久的 API，其他模块和第三方集成者可以消费这些 API。

# 代码生成

Magento 应用程序的一个巧妙特性是代码生成。正如其名称所暗示的，**代码生成**会生成不存在的类。这些类在 Magento 的`var/generation`目录中生成。

`var/generation`目录内的目录结构在一定程度上类似于核心的`vendor/magento/module-*`和`app/code`目录。更精确地说，它遵循模块结构。代码是为所谓的**Factory**、**Proxy**和**Interceptor**类生成的。

工厂类创建一个类型的实例。例如，由于在`vendor/magento`目录及其代码中某处调用了`Magento\Catalog\Model\ProductFactory`类，因此已创建了一个包含`Magento\Catalog\Model\ProductFactory`类的`var/generation/Magento/Catalog/Model/ProductFactory.php`文件。在运行时，当代码中调用`{someClassName}Factory`时，如果不存在，Magento 会在`var/generation`目录下创建一个工厂类。以下代码是`ProductFactory`类的（部分）示例：

```php
namespace Magento\Catalog\Model;

/**
* Factory class for @see \Magento\Catalog\Model\Product
*/
class ProductFactory
{
    //...

    /**
    * Create class instance with specified parameters
    *
    * @param array $data
    * @return \Magento\Catalog\Model\Product
    */
    public function create(array $data = array())
    {
        return $this->_objectManager->create($this->_instanceName, $data);
    }
}
```

注意`create`方法，它创建并返回`Product`类型实例。还要注意生成的代码是如何*类型安全*的，为**集成开发环境**（**IDEs**）提供`@return`注解以支持自动完成功能。

**工厂**用于将对象管理器与业务代码隔离开来。与业务对象不同，工厂可以依赖于对象管理器。

代理类是对某些**基类**的包装。代理类比基类提供更好的性能，因为它们可以在不实例化基类的情况下被实例化。基类仅在调用其方法时才被实例化。这对于将基类用作依赖项且实例化耗时且仅在执行路径的某些部分使用其方法的情况非常方便。

与工厂类似，代理类也生成在`var/generation`目录下。

如果我们查看包含`Magento\Catalog\Model\Session\Proxy`类的`var/generation/Magento/Catalog/Model/Session/Proxy.php`文件，我们会看到它实际上扩展了`Magento\Catalog\Model\Session`。包装的代理类在过程中实现了几个魔法方法，例如`__sleep`、`__wakeup`、`__clone`和`__call`。

拦截器是另一种由 Magento 自动生成的类类型。它与插件功能相关，将在第六章中详细讨论，*插件*。

为了触发代码生成，我们可以使用控制台上可用的代码编译器。我们可以运行*单租户*编译器或*多租户*编译器。

*单租户*意味着一个网站和商店，并且通过以下命令执行。

```php
magento setup:di:compile

```

*多租户*意味着不止一个独立的 Magento 应用程序，并且通过以下命令执行。

```php
magento setup:di:compile-multi-tenant

```

代码编译生成工厂、代理、拦截器和`setup/src/Magento/Setup/Module/Di/App/Task/Operation/`目录中列出的其他几个类。

# 变量目录

Magento 执行大量的缓存和某些类类型的自动生成。这些缓存和生成的类都位于 Magento 根目录的`var`目录中。`var`目录的常规内容如下：

```php
cache
composer_home
generation
log
di
view_preprocessed
page_cache
```

在开发过程中，我们很可能需要定期清除这些，以便我们的更改能够生效。

我们可以按照以下方式发出控制台命令以清除单个目录：

```php
rm -rf {Magento root dir}/var/generation/*

```

或者，我们可以使用内置的 `bin/magento` 控制台工具来触发命令，自动删除相应的目录，如下所示：

+   `bin/magento setup:upgrade`: 这将更新 Magento 数据库模式和数据。在此过程中，它将截断 `var/di` 和 `var/generation` 目录。

+   `bin/magento setup:di:compile`: 这将清除 `var/generation` 目录。在此之后，它将再次编译其中的代码。

+   `bin/magento deploy:mode:set {mode}`: 这将模式从开发模式更改为生产模式，反之亦然。在此过程中，它将截断 `var/di`、`var/generation` 和 `var/view_preprocessed` 目录。

+   `bin/magento cache:clean {type}`: 这将清理 `var/cache` 和 `var/page_cache` 目录。

在开发过程中，始终要记住 `var` 目录。否则，代码可能会遇到异常并无法正常工作。

# 编码标准

`代码文档。其想法是统一所有文件中代码 DocBlocks 的使用，无论使用哪种编程语言。然而，特定语言的 DocBlock 标准可能会覆盖它。`

`` 这是 Google JavaScript 风格指南和 JSDoc 的一个子集，可以在 [`usejsdoc.org`](http://usejsdoc.org) 找到。` ``

`` `The **LESS**` **编码标准定义了在处理 LESS 和 CSS 文件时的格式化和编码风格。** ``

### `**注意**`

`**您可以在 [`devdocs.magento.com`](http://devdocs.magento.com) 上阅读有关每个标准的实际细节，因为它们过于广泛，无法在本书中涵盖。**`

``**# 摘要    在本章中，我们探讨了 Composer，这是我们安装 Magento 时将首先与之交互的东西之一。然后，我们转向服务合同，它是 Magento 架构中最强大的部分之一，结果是我们使用的是老式的 PHP 接口。此外，我们还介绍了一些关于 Magento 代码生成功能的内容。因此，我们对 Factory 和 Proxy 类有了基本的了解。然后，我们查看 `var` 目录并探讨了其在开发过程中的作用。最后，我们简要介绍了 Magento 中使用的编码标准。    在下一章中，我们将讨论依赖注入，这是 Magento 最重要的架构部分之一。**```
