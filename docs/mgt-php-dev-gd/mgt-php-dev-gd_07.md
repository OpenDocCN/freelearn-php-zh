# 第七章：测试和质量保证

到目前为止，我们已经涵盖了：

+   Magento 基础知识

+   前端开发

+   后端开发

+   扩展和使用 API

然而，我们忽略了任何扩展或自定义代码开发的关键步骤：测试和质量保证。

尽管 Magento 是一个非常复杂和庞大的平台，但在 Magento2 之前的版本中没有包含/集成的单元测试套件。

因此，适当的测试和质量保证经常被大多数 Magento 开发人员忽视，要么是因为缺乏信息，要么是因为一些测试工具的大量开销，虽然没有太多可用于运行 Magento 的适当测试的工具，但现有的工具质量非常高。

在本章中，我们将看看测试 Magento 代码的不同选项，并为我们的自定义扩展构建一些非常基本的测试。

因此，让我们来看看本章涵盖的主题：

+   Magento 可用的不同测试框架和工具

+   测试我们的 Magento 代码的重要性

+   如何设置、安装和使用 Ecomdev PHPUnit 扩展

+   如何设置、安装和使用 Magento Mink 来运行功能测试

# 测试 Magento

在我们开始编写任何测试之前，重要的是我们了解与测试相关的概念，尤其是每种可用方法论。

## 单元测试

单元测试的理念是为我们代码的某些区域（单元）编写测试，以便我们可以验证代码是否按预期工作，并且函数是否返回预期值。

> *单元测试是一种方法，通过该方法测试源代码的单个单元，以确定它们是否适合使用，其中包括一个或多个计算机程序模块以及相关的控制数据、使用程序和操作程序。*

编写单元测试的另一个优势是，通过执行测试，我们更有可能编写更容易测试的代码。

这意味着随着我们不断编写更多的测试，我们的代码往往会被分解成更小但更专业的功能。我们开始构建一个测试套件，可以在引入更改或功能时针对我们的代码库运行；这就是回归测试。

## 回归测试

回归测试主要是指在进行代码更改后重新运行现有测试套件的做法，以检查新功能是否也引入了新错误。

> 回归测试是一种软件测试，旨在在对现有系统的功能和非功能区域进行更改（如增强、补丁或配置更改）后，发现新的软件错误或回归。

在 Magento 商店或任何电子商务网站的特定情况下，我们希望对商店的关键功能进行回归测试，例如结账、客户注册、添加到购物车等。

## 功能测试

功能测试更关注的是根据特定输入返回适当输出的应用程序，而不是内部发生的情况。

> *功能测试是一种基于被测试软件组件的规范的黑盒测试类型。通过向它们提供输入并检查输出来测试功能，很少考虑内部程序结构。*

这对于像我们这样的电子商务网站尤为重要，我们希望测试网站与客户的体验一致。

## TDD

近年来变得越来越受欢迎的一种测试方法，现在也正在 Magento 中出现，被称为**测试驱动开发**（**TDD**）。

> *测试驱动开发（TDD）是一种依赖于非常短的开发周期重复的软件开发过程：首先开发人员编写一个（最初失败的）自动化测试用例，定义所需的改进或新功能，然后生成最少量的代码来通过该测试，最后将新代码重构为可接受的标准。*

TDD 背后的基本概念是首先编写一个失败的测试，然后编写代码来通过测试；这会产生非常短的开发周期，并有助于简化代码。

理想情况下，您希望通过在 Magento 中使用 TDD 来开始开发您的模块和扩展。我们在之前的章节中省略了这一点，因为这会增加不必要的复杂性并使读者困惑。

### 注意

有关从头开始使用 Magento 进行 TDD 的完整教程，请访问`http://magedevguide.com/getting-started-with-tdd`。

# 工具和测试框架

如前所述，有几个框架和工具可用于测试 PHP 代码和 Magento 代码。让我们更好地了解每一个：

+   `Ecomdev_PHPUnit`：这个扩展真是太棒了；Ecomdev 的开发人员创建了一个集成了 PHPUnit 和 Magento 的扩展，还向 PHPUnit 添加了 Magento 特定的断言，而无需修改核心文件或影响数据库。

+   `Magento_Mink`：Mink 是 Behat 框架的 PHP 库，允许您编写功能和验收测试；Mink 允许编写模拟用户行为和浏览器交互的测试。

+   `Magento_TAF`：`Magento_TAF`代表 Magento 测试自动化框架，这是 Magento 提供的官方测试工具。`Magento_TAF`包括超过 1,000 个功能测试，非常强大。不幸的是，它有一个主要缺点；它有很大的开销和陡峭的学习曲线。

## 使用 PHPUnit 进行单元测试

在`Ecomdev_PHPUnit`之前，使用 PHPUnit 测试 Magento 是有问题的，而且从可用的不同方法来看，实际上并不实用。几乎所有都需要核心代码修改，或者开发人员必须费力地设置基本的 PHPUnits。

### 安装 Ecomdev_PHPUnit

安装`Ecomdev_PHPUnit`的最简单方法是直接从 GitHub 存储库获取副本。让我们在控制台上写下以下命令：

```php
**git clone git://github.com/IvanChepurnyi/EcomDev_PHPUnit.git**

```

现在将文件复制到您的 Magento 根目录。

### 注意

Composer 和 Modman 是可用于安装的替代选项。有关每个选项的更多信息，请访问[`magedevguide.com/module-managers`](http://magedevguide.com/module-managers)。

最后，我们需要设置配置，指示 PHPUnit 扩展使用哪个数据库；`local.xml.phpunit`是`Ecomdev_PHPUnit`添加的新文件。这个文件包含所有特定于扩展的配置，并指定测试数据库的名称。

文件位置为`app/etc/local.xml.phpunit`。参考以下代码：

```php
<?xml version="1.0"?>
<config>
    <global>
        <resources>
            <default_setup>
                <connection>
                   <dbname><![CDATA[magento_unit_tests]]></dbname>
                </connection>
            </default_setup>
        </resources>
    </global>
    <default>
        <web>
            <seo>
                <use_rewrites>1</use_rewrites>
            </seo>
            <secure>
                <base_url>[change me]</base_url>
            </secure>
            <unsecure>
                <base_url>[change me]</base_url>
            </unsecure>
            <url>
                <redirect_to_base>0</redirect_to_base>
            </url>
        </web>
    </default>
    <phpunit>
        <allow_same_db>0</allow_same_db>
    </phpunit>
</config>
```

您需要为运行测试创建一个新的数据库，并在`local.xml.phpunit`文件中替换示例配置值。

默认情况下，这个扩展不允许您在同一个数据库上运行测试；将测试数据库与开发和生产数据库分开允许我们有信心地运行我们的测试。

### 为我们的扩展设置配置

现在我们已经安装并设置了 PHPUnit 扩展，我们需要准备我们的礼品注册扩展来运行单元测试。按照以下步骤进行：

1.  打开礼品注册扩展的`config.xml`文件

1.  添加以下代码（文件位置为`app/code/local/Mdg/Giftregistry/etc/config.xml`）：

```php
…
<phpunit>
        <suite>
            <modules>
                    <Mdg_Giftregistry/>
            </modules>
         </suite>
</phpunit>
…
```

这个新的配置节点允许 PHPUnit 扩展识别扩展并运行匹配的测试。

我们还需要创建一个名为`Test`的新目录，我们将用它来放置所有的测试文件。使用`Ecomdev_PHPUnit`相比以前的方法的一个优点是，这个扩展遵循 Magento 的标准。

这意味着我们必须在`Test`文件夹内保持相同的模块目录结构：

```php
Test/
Model/
Block/
Helper/
Controller/
Config/
```

基于此，每个`Test`案例类的命名约定将是`[Namespace]_[Module Name]_Test_[Group Directory]_[Entity Name]`。

每个`Test`类必须扩展以下三个基本`Test`类中的一个：

+   `EcomDev_PHPUnit_Test_Case`：这个类用于测试助手、模型和块

+   `EcomDev_PHPUnit_Test_Case_Config`：这个类用于测试模块配置

+   `EcomDev_PHPUnit_Test_Case_Controller`：这个类用于测试布局渲染过程和控制器逻辑

### 测试案例的解剖

在跳入并尝试创建我们的第一个测试之前，让我们分解`Ecomdev_PHPUnit`提供的一个示例：

```php
<?php
class EcomDev_Example_Test_Model_Product extends EcomDev_PHPUnit_Test_Case
{
    /**
     * Product price calculation test
     *
     * @test
     * @loadFixture
     * @doNotIndexAll
     * @dataProvider dataProvider
     */
    public function priceCalculation($productId, $storeId)
    {
        $storeId = Mage::app()->getStore($storeId)->getId();
        $product = Mage::getModel('catalog/product')
            ->setStoreId($storeId)
            ->load($productId);
        $expected = $this->expected('%s-%s', $productId, $storeId);
        $this->assertEquals(
            $expected->getFinalPrice(),
            $product->getFinalPrice()
        );
        $this->assertEquals(
            $expected->getPrice(),
            $product->getPrice()
        );
    }
}
```

在示例`test`类中要注意的第一件重要的事情是注释注释：

```php
…
/**
     * Product price calculation test
     *
     * @test
     * @loadFixture
     * @doNotIndexAll
     * @dataProvider dataProvider
     */
…
```

这些注释被 PHPUnit 扩展用来识别哪些类函数是测试，它们还允许我们为运行每个测试设置特定的设置。让我们来看一下一些可用的注释：

+   `@test`：这个注释将一个类函数标识为 PHPUnit 测试

+   `@loadFixture`：这个注释指定了固定的使用

+   `@loadExpectation`：这个注释指定了期望的使用

+   `@doNotIndexAll`：通过添加这个注释，我们告诉 PHPUnit 测试在加载固定后不应该运行任何索引

+   `@doNotIndex [index_code]`：通过添加这个注释，我们可以指示 PHPUnit 不运行特定的索引

所以现在，你可能有点困惑。固定？期望？它们是什么？

以下是对固定和期望的简要描述：

+   **固定**：固定是**另一种标记语言**（**YAML**）文件，代表数据库或配置实体

+   **期望**：期望对我们的测试中不想要硬编码的值很有用，也是在 YAML 值中指定的

### 注意

有关 YAML 标记的更多信息，请访问`http://magedevguide.com/resources/yaml`。

所以，正如我们所看到的，固定提供了测试处理的数据，期望用于检查测试返回的结果是否是我们期望看到的。

固定和期望存储在每个`Test`类型目录中。按照之前的例子，我们将有一个名为`Product/`的新目录。在里面，我们需要一个期望的新目录和一个我们的固定的新目录。

让我们来看一下修订后的文件夹结构：

```php
Test/
Model/  
  Product.php
  Product/
    expectations/
    fixtures/
Block/
Helper/
Controller/
Config/
```

![测试案例的解剖](img/3060OS_07_01.jpg)

### 创建一个单元测试

对于我们的第一个单元测试，让我们创建一个非常基本的测试，允许我们测试之前创建的礼品注册模型。

正如我们之前提到的，`Ecomdev_PHPUnit`使用一个单独的数据库来运行所有的测试；为此，我们需要创建一个新的固定，为我们的测试用例提供所有的数据。按照以下步骤：

1.  打开`Test/Model`文件夹。

1.  创建一个名为`Registry`的新文件夹。

1.  在`Registry`文件夹中，创建一个名为`fixtures`的新文件夹。

1.  创建一个名为`registryList.yaml`的新文件，并将以下代码粘贴到其中（文件位置为`app/code/local/Mdg/Giftregistry/Test/Model/fixtures/registryList.yaml`）：

```php
  website: # Initialize websites
    - website_id: 2
      code: default
      name: Test Website
      default_group_id: 2
  group: # Initializes store groups
    - group_id: 2
      website_id: 2
      name: Test Store Group
      default_store_id: 2
      root_category_id: 2 # Default Category
  store: # Initializes store views
    - store_id: 2
      website_id: 2
      group_id: 2
      code: default
      name: Default Test Store
      is_active: 1
eav:
   customer_customer:
     - entity_id: 1
       entity_type_id: 3
       website_id: 2
       email: test@magentotest.com
       group_id: 2
       store_id: 2
       is_active: 1
   mdg_giftregistry_entity:
     - entity_id: 1
       customer_id: 1
       type_id: 2
       website_id: 2
       event_date: 12/12/2012
       event_country: Canada
       event_location: Dundas Square
       created_at: 21/12/2012
     - entity_id: 2
       customer_id: 1
       type_id: 3
       website_id: 2
       event_date: 01/01/2013
       event_country: Canada
       event_location: Eaton Center
       created_at: 21/12/2012
```

它可能看起来不像，但我们通过这个固定添加了很多信息。我们将创建以下固定数据：

+   一个网站范围

+   一个商店组

+   一个商店视图

+   一个客户记录

+   两个礼品注册

通过使用固定，我们正在创建可用于我们的测试用例的数据。这使我们能够多次运行相同的数据测试，并灵活地进行更改。

现在，你可能想知道 PHPUnit 扩展如何将`Test`案例与特定的固定配对。

扩展加载固定有两种方式：一种是在注释注释中指定固定，或者如果没有指定固定名称，扩展将搜索与正在执行的`Test`案例函数相同名称的固定。

知道这一点，让我们创建我们的第一个`Test`案例：

1.  导航到`Test/Model`文件夹。

1.  创建一个名为`Registry.php`的新`Test`类。

1.  添加以下基本代码（文件位置为`app/code/local/Mdg/Giftregistry/Test/Model/Registry.php`）：

```php
<?php
class Mdg_Giftregistry_Test_Model_Registry extends EcomDev_PHPUnit_Test_Case
{
    /**
     * Listing available registries
     *
     * @test
     * @loadFixture
     * @doNotIndexAll
     * @dataProvider dataProvider
     */
    public function registryList()
    {

    }
}
```

我们刚刚创建了基本函数，但还没有添加任何逻辑。在这之前，让我们先看看什么构成了一个`Test`案例。

一个`Test`案例通过使用断言来评估和测试我们的代码。断言是我们的`Test`案例从父`TestCase`类继承的特殊函数。在默认可用的断言中，我们有：

+   `assertEquals()`

+   `assertGreaterThan()`

+   `assertGreaterThanOrEqual()`

+   `assertLessThan()`

+   `assertLessThanOrEqual()`

+   `assertTrue()`

现在，如果我们只使用这些类型的断言来测试 Magento 代码，可能会变得困难甚至不可能。这就是`Ecomdev_PHPUnit`发挥作用的地方。

这个扩展不仅将 PHPUnit 与 Magento 整合得很好，遵循他们的标准，还在 PHPUnit 测试中添加了 Magento 特定的断言。让我们来看看扩展添加的一些断言：

+   `assertEventDispatched()`

+   `assertBlockAlias()`

+   `assertModelAlias()`

+   `assertHelperAlias()`

+   `assertModuleCodePool()`

+   `assertModuleDepends()`

+   `assertConfigNodeValue()`

+   `assertLayoutFileExists()`

这些只是可用的一些断言，正如你所看到的，它们为构建全面的测试提供了很大的力量。

现在我们对 PHPUnit 的`Test`案例有了更多了解，让我们继续创建我们的第一个 Magento `Test`案例：

1.  导航到之前创建的`Registry.php`测试案例类。

1.  在`registryList()`函数内添加以下代码（文件位置为`app/code/local/Mdg/Giftregistry/Test/Model/Registry.php`）：

```php
    /**
     * Listing available registries
     *
     * @test
     * @loadFixture
     * @doNotIndexAll
     * @dataProvider dataProvider
     */
    public function registryList()
    {
        $registryList = Mage::getModel('mdg_giftregistry/entity')->getCollection();
        $this->assertEquals(
            2,
            $registryList->count()
        );
    }
```

这是一个非常基本的测试；我们所做的就是加载一个注册表集合。在这种情况下，所有的注册表都是可用的，然后他们运行一个断言来检查集合计数是否匹配。

然而，这并不是很有用。如果我们能够只加载属于特定用户（我们的测试用户）的注册表，并检查集合大小，那将更好。因此，让我们稍微改变一下代码：

文件位置为`app/code/local/Mdg/Giftregistry/Test/Model/Registry.php`。参考以下代码：

```php
    /**
     * Listing available registries
     *
     * @test
     * @loadFixture
     * @doNotIndexAll
     * @dataProvider dataProvider
     */
    public function registryList()
    {
        $customerId = 1;
        $registryList = Mage::getModel('mdg_giftregistry/entity')
->getCollection()
->addFieldToFilter('customer_id', $customerId);
        $this->assertEquals(
            2,
            $registryList->count()
        );
    }
```

仅仅通过改变几行代码，我们创建了一个测试，可以检查我们的注册表集合是否正常工作，并且是否正确地链接到客户记录。

在你的 shell 中运行以下命令：

```php
**$ phpunit**

```

如果一切如预期般进行，我们应该看到以下输出：

```php
**PHPUnit 3.4 by Sebastian Bergmann**
**.**
**Time: 1 second**
**Tests: 1, Assertions: 1, Failures 0**

```

### 注意

您还可以运行`$phpunit`—colors 以获得更好的输出。

现在，我们只需要一个测试来验证注册表项是否正常工作：

1.  导航到之前创建的`Registry.php`测试案例类。

1.  在`registryItemsList()`函数内添加以下代码（文件位置为`app/code/local/Mdg/Giftregistry/Test/Model/Registry.php`）：

```php
    /**
     * Listing available items for a specific registry
     *
     * @test
     * @loadFixture
     * @doNotIndexAll
     * @dataProvider dataProvider
     */
    public function registryItemsList()
    {
        $customerId = 1;
        $registry   = Mage::getModel('mdg_giftregistry/entity')
->loadByCustomerId($customerId);

        $registryItems = $registry->getItems();
        $this->assertEquals(
            3,
            $registryItems->count()
        );
    }
```

我们还需要一个新的 fixture 来配合我们的新`Test`案例：

1.  导航到`Test/Model`文件夹。

1.  打开`Registry`文件夹。

1.  创建一个名为`registryItemsList.yaml`的新文件（文件位置为`app/code/local/Mdg/Giftregistry/Test/Model/fixtures/ registryItemsList.yaml`）：

```php
  website: # Initialize websites
    - website_id: 2
      code: default
      name: Test Website
      default_group_id: 2
  group: # Initializes store groups
    - group_id: 2
      website_id: 2
      name: Test Store Group
      default_store_id: 2
      root_category_id: 2 # Default Category
  store: # Initializes store views
    - store_id: 2
      website_id: 2
      group_id: 2
      code: default
      name: Default Test Store
      is_active: 1
eav:
   customer_customer:
     - entity_id: 1
       entity_type_id: 3
       website_id: 2
       email: test@magentotest.com
       group_id: 2
       store_id: 2
       is_active: 1
   mdg_giftregistry_entity:
     - entity_id: 1
       customer_id: 1
       type_id: 2
       website_id: 2
       event_date: 12/12/2012
       event_country: Canada
       event_location: Dundas Square
       created_at: 21/12/2012
   mdg_giftregistry_item:
     - item_id: 1
       registry_id: 1
       product_id: 1
     - item_id: 2
       registry_id: 1
       product_id: 2
     - item_id: 3
       registry_id: 1
       product_id: 3 
```

让我们运行我们的测试套件：

```php
**$phpunit --colors**

```

我们应该看到两个测试都通过了：

```php
PHPUnit 3.4 by Sebastian Bergmann
.
Time: 4 second
Tests: 2, Assertions: 2, Failures 0
```

最后，让我们用正确的期望值替换我们的硬编码变量：

1.  导航到`Module Test/Model`文件夹。

1.  打开`Registry`文件夹。

1.  在`Registry`文件夹内，创建一个名为`expectations`的新文件夹。

1.  创建一个名为`registryList.yaml`的新文件（文件位置为`app/code/local/Mdg/Giftregistry/Test/Model/expectations/registryList.yaml`）。

```php
count: 2
```

是不是很容易？好吧，它是如此容易，以至于我们将再次为`registryItemsList`测试案例做同样的事情：

1.  导航到`Module Test/Model`文件夹。

1.  打开`Registry`文件夹。

1.  在`expectations`文件夹中创建一个名为`registryItemsList.yaml`的新文件（文件位置为`app/code/local/Mdg/Giftregistry/Test/Model/expectations/registryItemsList.yaml`）：

```php
count: 3
```

最后，我们需要做的最后一件事是更新我们的`Test`案例类以使用期望。确保更新文件具有以下代码（文件位置为`app/code/local/Mdg/Giftregistry/Test/Model/Registry.php`）：

```php
<?php
class Mdg_Giftregistry_Test_Model_Registry extends EcomDev_PHPUnit_Test_Case
{
    /**
     * Product price calculation test
     *
     * @test
     * @loadFixture
     * @doNotIndexAll
     * @dataProvider dataProvider
     */
    public function registryList()
    {
        $customerId = 1;
        $registryList = Mage::getModel('mdg_giftregistry/entity')
                ->getCollection()
                ->addFieldToFilter('customer_id', $customerId);
        $this->assertEquals(
            $this->_getExpectations()->getCount(),$this->_getExpectations()->getCount(),
            $registryList->count()
        );
    }
    /**
     * Listing available items for a specific registry
     *
     * @test
     * @loadFixture
     * @doNotIndexAll
     * @dataProvider dataProvider
     */
    public function registryItemsList()
    {
        $customerId = 1;
        $registry   = Mage::getModel('mdg_giftregistry/entity')->loadByCustomerId($customerId);

        $registryItems = $registry->getItems();
        $this->assertEquals(
            $this->_getExpectations()->getCount(),
            $registryItems->count()
        );
    }
}
```

这里唯一的变化是，我们用期望值替换了断言中的硬编码值。如果我们需要进行任何更改，我们不需要更改我们的代码；我们只需更新期望和固定装置。

## 使用 Mink 进行功能测试

到目前为止，我们已经学会了如何对我们的代码运行单元测试，虽然单元测试非常适合测试代码和逻辑的各个部分，但对于像 Magento 这样的大型应用程序来说，从用户的角度进行测试是很重要的。

### 注意

功能测试主要涉及黑盒测试，不关心应用程序的源代码。

为了做到这一点，我们可以使用 Mink。Mink 是一个简单的 PHP 库，可以虚拟化 Web 浏览器。Mink 通过使用不同的驱动程序来工作。它支持以下驱动程序：

+   `GoutteDriver`：这是 Symfony 框架的创建者编写的纯 PHP 无头浏览器

+   `SahiDriver`：这是一个新的 JS 浏览器控制器，正在迅速取代 Selenium

+   `ZombieDriver`：这是一个在`Node.js`中编写的浏览器仿真器，目前只限于一个浏览器（Chromium）

+   `SeleniumDriver`：这是目前最流行的浏览器驱动程序；原始版本依赖于第三方服务器来运行测试

+   `Selenium2Driver`：Selenium 的当前版本在 Python、Ruby、Java 和 C#中得到了充分支持

### Magento Mink 安装和设置

使用 Mink 与 Magento 非常容易，这要归功于 Johann Reinke，他创建了一个 Magento 扩展，方便了 Mink 与 Magento 的集成。

我们将使用 Modgit 来安装这个扩展，Modgit 是一个受 Modman 启发的模块管理器。Modgit 允许我们直接从 GitHub 存储库部署 Magento 扩展，而无需创建符号链接。

安装 Modgit 只需三行代码即可完成：

```php
**wget -O modgit https://raw.github.com/jreinke/modgit/master/modgit**
**chmod +x modgit**
**sudo mv modgit /usr/local/bin**

```

是不是很容易？现在我们可以继续安装 Magento Mink，我们应该感谢 Modgit，因为这样甚至更容易：

1.  转到 Magento 根目录。

1.  运行以下命令：

```php
**modgit init**
**modgit -e README.md clone mink https://github.com/jreinke/magento-mink.git**

```

就是这样。Modgit 将负责直接从 GitHub 存储库安装文件。

# 创建我们的第一个测试

`Mink`测试也存储在`Test`文件夹中。让我们创建`Mink`测试类的基本骨架：

1.  导航到我们模块根目录下的`Test`文件夹。

1.  创建一个名为`Mink`的新目录。

1.  在`Mink`目录中，创建一个名为`Registry.php`的新 PHP 类。

1.  复制以下代码（文件位置为`app/code/local/Mdg/Giftregistry/Test/Mink/Registry.php`）：

```php
<?php
class Mdg_Giftregistry_Test_Mink_Registry extends JR_Mink_Test_Mink 
{   
    public function testAddProductToRegistry()
    {
        $this->section('TEST ADD PRODUCT TO REGISTRY');
        $this->setCurrentStore('default');
        $this->setDriver('goutte');
        $this->context();

        // Go to homepage
        $this->output($this->bold('Go To the Homepage'));
        $url = Mage::getStoreConfig('web/unsecure/base_url');
        $this->visit($url);
        $category = $this->find('css', '#nav .nav-1-1 a');
        if (!$category) {
            return false;
        }

        // Go to the Login page
        $loginUrl = $this->find('css', 'ul.links li.last a');
        if ($loginUrl) {
            $this->visit($loginUrl->getAttribute('href'));
        }

        $login = $this->find('css', '#email');
        $pwd = $this->find('css', '#pass');
        $submit = $this->find('css', '#send2');

        if ($login && $pwd && $submit) {
            $email = 'user@example.com';
            $password = 'password';
            $this->output(sprintf("Try to authenticate '%s' with password '%s'", $email, $password));
            $login->setValue($email);
            $pwd->setValue($password);
            $submit->click();
            $this->attempt(
                $this->find('css', 'div.welcome-msg'),
                'Customer successfully logged in',
                'Error authenticating customer'
            );
        }

        // Go to the category page
        $this->output($this->bold('Go to the category list'));
        $this->visit($category->getAttribute('href'));
        $product = $this->find('css', '.category-products li.first a');
        if (!$product) {
            return false;
        }

        // Go to product view
        $this->output($this->bold('Go to product view'));
        $this->visit($product->getAttribute('href'));
        $form = $this->find('css', '#product_registry_form');
        if ($form) {
            $addToCartUrl = $form->getAttribute('action');
            $this->visit($addToCartUrl);
            $this->attempt(
                $this->find('css', '#btn-add-giftregistry'),
                'Product added to gift registry successfully',
                'Error adding product to gift registry'
            );
        }
    }
}
```

仅仅乍一看，你就可以看出这个功能测试与我们之前构建的单元测试有很大不同，尽管看起来代码很多，但实际上很简单。之前的测试已经在代码块中完成了。让我们分解一下之前的测试在做什么：

+   设置浏览器驱动程序和当前商店

+   转到主页并检查有效的类别链接

+   尝试以测试用户身份登录

+   转到类别页面

+   打开该类别上的第一个产品

+   尝试将产品添加到客户的礼品注册表

### 注意

这个测试做了一些假设，并期望在现有的礼品注册表中有一个有效的客户。

在创建`Mink`测试时，我们必须牢记一些考虑因素：

+   每个测试类必须扩展`JR_Mink_Test_Mink`

+   每个测试函数必须以 test 关键字开头

最后，我们唯一需要做的就是运行我们的测试。我们可以通过进入命令行并运行以下命令来实现这一点：

```php
**$ php shell/mink.php**

```

如果一切顺利，我们应该看到类似以下输出：

```php
---------------------- SCRIPT START ---------------------------------
Found 1 file
-------------- TEST ADD PRODUCT TO REGISTRY -------------------------
Switching to store 'default'
Now using Goutte driver
----------------------------------- CONTEXT ------------------------------------
website: base, store: default
Cache info:
config            Disabled  N/A       Configuration
layout            Disabled  N/A       Layouts
block_html        Disabled  N/A       Blocks HTML output
translate         Disabled  N/A       Translations
collections       Disabled  N/A       Collections Data
eav               Disabled  N/A       EAV types and attributes
config_api        Disabled  N/A       Web Services Configuration
config_api2       Disabled  N/A       Web Services Configuration
ecomdev_phpunit   Disabled  N/A       Unit Test Cases

Go To the Homepage [OK]
Try to authenticate user@example.com with password password [OK]
Go to the category list [OK]
Go to product view [OK]
Product added to gift registry successfully

```

# 总结

在本章中，我们介绍了 Magento 测试的基础知识。本章的目的不是构建复杂的测试或深入讨论，而是让我们初步了解并清楚地了解我们可以做些什么来测试我们的扩展。

本章我们涵盖了几个重要的主题，通过拥有适当的测试套件和工具，可以帮助我们避免未来的头痛，并提高我们代码的质量。

在下一章，我们将学习如何打包和分发自定义代码和扩展。
