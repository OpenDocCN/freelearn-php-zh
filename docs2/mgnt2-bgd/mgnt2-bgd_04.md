# 第四章。模型和集合

如同大多数现代框架和平台，如今 Magento 采用 **对象关系映射**（**ORM**）方法而非原始 SQL 查询。尽管底层机制仍然归结为 SQL，我们现在严格处理对象。这使得我们的应用程序代码更易读、更易于管理，并从供应商特定的 SQL 差异中隔离出来。模型、资源和集合是三种类型的类协同工作，使我们能够全面管理实体数据，从加载、保存、删除和列出实体。我们的大部分数据访问和管理将通过名为 Magento 模型的 PHP 类来完成。模型本身不包含与数据库通信的任何代码。

数据库通信部分被解耦成其自身的 PHP 类，称为资源类。然后每个模型都被分配一个资源类。在模型上调用 `load`、`save` 或 `delete` 方法会被委托给资源类，因为它们是实际从数据库读取、写入和删除数据的地方。理论上，有了足够的知识，可以编写针对各种数据库供应商的新资源类。

除了模型和资源类之外，我们还有集合类。我们可以将集合视为单个模型实例的数组。在基本层面上，集合扩展自 `\Magento\Framework\Data\Collection` 类，该类实现了来自 **标准 PHP 库**（**SPL**）的 `\IteratorAggregate` 和 `\Countable` 以及一些其他 Magento 特定的类。

更多的时候，我们将模型和资源视为一个单一统一的事物，因此简单地称之为模型。Magento 处理两种类型的模型，我们可以将它们分类为简单和 EAV 模型。

在本章中，我们将涵盖以下主题：

+   创建一个微型模块

+   创建一个简单模型

+   EAV 模型

+   理解模式和数据脚本流程

+   创建安装模式脚本（`InstallSchema.php`）

+   创建升级模式脚本（`UpgradeSchema.php`）

+   创建安装数据脚本（`InstallData.php`）

+   创建升级数据脚本（`UpgradeData.php`）

+   实体 CRUD 操作

+   管理集合

# 创建一个微型模块

为了本章的目的，我们将创建一个名为 `Foggyline_Office` 的小型模块。

该模块将定义两个实体，如下所示：

+   `Department`：一个具有以下字段的简单模型：

    +   `entity_id`：主键

    +   `name`：部门的名称，字符串值

+   `Employee`：一个具有以下字段和属性的 EAV 模型：

    +   **字段：**

        +   `entity_id`：主键

        +   `department_id`：外键，指向 `Department.entity_id`

        +   `email`：员工的唯一电子邮件，字符串值

        +   `first_name`：员工的姓名，字符串值

        +   `last_name`：员工的姓氏，字符串值

    +   **属性：**

        +   `service_years`：员工的工龄，整数值

        +   `dob`：员工的出生日期，日期时间值

        +   `salary` – 月薪，十进制值

        +   `vat_number`：增值税号，（短）字符串值

        +   `note`：关于员工的可能注释，（长）字符串值

每个模块都以 `registration.php` 和 `module.xml` 文件开始。为了我们本章模块的目的，让我们创建一个包含以下内容的 `app/code/Foggyline/Office/registration.php` 文件：

```php
<?php
\Magento\Framework\Component\ComponentRegistrar::register(
    \Magento\Framework\Component\ComponentRegistrar::MODULE,
    'Foggyline_Office',
    __DIR__
);
```

`registration.php` 文件可以看作是我们模块的入口点。

现在，让我们创建一个包含以下内容的 `app/code/Foggyline/Office/etc/module.xml` 文件：

```php
<config  xsi:noNamespaceSchemaLocation="urn:magento:framework:Module/ etc/module.xsd">
    <module name="Foggyline_Office" setup_version="1.0.0">
        <sequence>
            <module name="Magento_Eav"/>
        </sequence>
    </module>
</config>
```

我们将在后面的章节中更详细地介绍 `module.xml` 文件的结构。现在，我们只关注 `sequence` 中的 `setup_version` 属性和 `module` 元素。

`setup_version` 的值很重要，因为我们可能会在我们的模式安装脚本（`InstallSchema.php`）文件中使用它，有效地将安装脚本转换为更新脚本，正如我们很快将展示的那样。

`sequence` 元素是 Magento 为我们的模块设置依赖关系的方式。鉴于我们的模块将使用 EAV 实体，我们列出 `Magento_Eav` 作为依赖项。

## 创建一个简单的模型

根据要求，`Department` 实体被建模为一个简单的模型。我们之前提到，每当谈到模型时，我们隐含地想到 `model` 类、`resource` 类和 `collection` 类形成一个单元。

让我们先从创建一个 `model` 类开始，部分定义在 `app/code/Foggyline/Office/Model/Department.php` 文件中，如下所示：

```php
namespace Foggyline\Office\Model;

class Department extends \Magento\Framework\Model\AbstractModel
{
    protected function _construct()
    {
        $this-> _init('Foggyline\Office\Model \ResourceModel\Department');
    }
}
```

这里发生的一切就是，我们正在扩展 `\Magento\Framework\Model\AbstractModel` 类，并在 `_construct` 方法中触发 `$this->_init` 方法，传递我们的 `resource` 类。

`AbstractModel` 进一步扩展了 `\Magento\Framework\Object`。我们的 `model` 类最终从 `Object` 继承的事实意味着我们不需要在 `model` 类上定义属性名称。`Object` 为我们做的事情是，它使我们能够神奇地获取、设置、取消设置和检查属性上的值存在。为了给出一个比 `name` 更健壮的例子，想象我们的实体在以下代码中有一个名为 `employee_average_salary` 的属性：

```php
$department->getData('employee_average_salary');
$department->getEmployeeAverageSalary();

$department->setData('employee_average_salary', 'theValue');
$department->setEmployeeAverageSalary('theValue');

$department->unsetData('employee_average_salary');
$department->unsEmployeeAverageSalary();

$department->hasData('employee_average_salary');
$department->hasEmployeeAverageSalary();
```

这之所以有效，是因为 `Object` 实现了 `setData`、`unsetData`、`getData` 和魔法 `__call` 方法。魔法 `__call` 方法实现的美丽之处在于，它理解 `getEmployeeAverageSalary`、`setEmployeeAverageSalary`、`unsEmployeeAverageSalary` 和 `hasEmployeeAverageSalary` 这样的方法调用，即使它们不存在于 `Model` 类上。然而，如果我们选择在我们的 `Model` 类中实现一些这些方法，我们可以自由地这样做，并且当调用它们时，Magento 会捕获它们。

这是 Magento 的一个重要方面，有时对新来者来说可能有些令人困惑。

一旦我们有一个 `model` 类，我们创建一个模型 `resource` 类，部分定义在 `app/code/Foggyline/Office/Model/ResourceModel/Department.php` 文件中，如下所示：

```php
namespace Foggyline\Office\Model\ResourceModel;

class Department extends \Magento\Framework\Model\ResourceModel\Db\AbstractDb
{
    protected function _construct()
    {
        $this->_init('foggyline_office_department', 'entity_id');
    }
}
```

我们继承自`\Magento\Framework\Model\ResourceModel\Db\AbstractDb`的`resource`类在`_construct`方法中触发`$this->_init`方法的调用。`$this->_init`接受两个参数。第一个参数是表名`foggyline_office_department`，我们的模型将在此表中持久化其数据。第二个参数是表中的主键列名`entity_id`。

`AbstractDb`进一步扩展了`Magento\Framework\Model\ResourceModel\AbstractResource`。

### 注意

资源类是向数据库通信的关键。我们只需要命名表及其主键，我们的模型就可以保存、删除和更新实体。

最后，我们创建我们的`collection`类，部分定义在`app/code/Foggyline/Office/Model/ResourceModel/Department/Collection.php`文件中，如下所示：

```php
namespace Foggyline\Office\Model\ResourceModel\Department;

class Collection extends \Magento\Framework\Model\ResourceModel \Db\Collection\AbstractCollection
{
    protected function _construct()
    {
        $this->_init(
            'Foggyline\Office\Model\Department',
            'Foggyline\Office\Model\ResourceModel\Department'
        );
    }
}
```

`collection`类继承自`\Magento\Framework\Model\ResourceModel\Db\Collection\AbstractCollection`，并且类似于`model`和`resource`类，在`_construct`方法中调用`$this->_init`方法。这次，`_init`接受两个参数。第一个参数是完整的`model`类名`Foggyline\Office\Model\Department`，第二个参数是完整的资源类名`Foggyline\Office\Model\ResourceModel\Department`。

`AbstractCollection`实现了`Magento\Framework\App\ResourceConnection\SourceProviderInterface`，并扩展了`\Magento\Framework\Data\Collection\AbstractDb`。`AbstractDb`进一步扩展了`\Magento\Framework\Data\Collection`。

值得花些时间研究这些`collection`类的内部结构，因为这是我们处理获取符合某些搜索标准实体列表时的首选地方。

## 创建一个 EAV 模型

根据要求，`Employee`实体被建模为一个 EAV 模型。

让我们先从创建一个 EAV `model`类开始，部分定义在`app/code/Foggyline/Office/Model/Employee.php`文件中，如下所示：

```php
namespace Foggyline\Office\Model;

class Employee extends \Magento\Framework\Model\AbstractModel
{
    const ENTITY = 'foggyline_office_employee';

    public function _construct()
    {
        $this-> _init('Foggyline\Office \Model \ResourceModel\Employee');
    }
}
```

在这里，我们继承自`\Magento\Framework\Model\AbstractModel`类，这与之前描述的简单模型相同。这里唯一的区别是我们定义了一个`ENTITY`常量，但这只是为了后面的语法糖；它对实际的`model`类没有实际意义。

接下来，我们创建一个 EAV 模型`resource`类，部分定义在`app/code/Foggyline/Office/Model/ResourceModel/Employee.php`文件中，如下所示：

```php
namespace Foggyline\Office\Model\ResourceModel;

class Employee extends \Magento\Eav\Model\Entity\AbstractEntity
{

    protected function _construct()
    {
        $this->_read = 'foggyline_office_employee_read';
        $this->_write = 'foggyline_office_employee_write';
    }

    public function getEntityType()
    {
        if (empty($this->_type)) {
            $this->setType(\Foggyline\Office\Model \Employee::ENTITY);
        }
        return parent::getEntityType();
    }
}
```

我们的`resource`类继承自`\Magento\Eav\Model\Entity\AbstractEntity`，并通过`_construct`方法设置`$this->_read`和`$this->_write`类属性。这些可以自由分配为我们想要的任何值，最好遵循我们模块的命名模式。读取和写入连接需要命名，否则当使用我们的实体时，Magento 会产生错误。

`getEntityType` 方法内部将 `_type` 值设置为 `\Foggyline\Office\Model\Employee::ENTITY`，即字符串 `foggyline_office_employee`。这个相同的值存储在 `eav_entity_type` 表的 `entity_type_code` 列中。此时，`eav_entity_type` 表中没有这样的条目。这是因为安装模式脚本将会创建一个，正如我们很快将要演示的那样。

最后，我们创建我们的 `collection` 类，部分定义在 `app/code/Foggyline/Office/Model/ResourceModel/Employee/Collection.php` 文件中，如下所示：

```php
namespace Foggyline\Office\Model\ResourceModel\Employee;

class Collection extends \Magento\Eav\Model\Entity\Collection\AbstractCollection
{
    protected function _construct()
    {
        $this->_init('Foggyline\Office\Model\Employee', 'Foggyline\Office\Model\ResourceModel\Employee');
    }
}
```

`collection` 类继承自 `\Magento\Eav\Model\Entity\Collection\AbstractCollection`，并且与模型类类似，在 `_construct` 中调用 `$this->_init` 方法。`_init` 接受两个参数：完整的模型类名 `Foggyline\Office\Model\Employee` 和完整的资源类名 `Foggyline\Office\Model\ResourceModel\Employee`。

`AbstractCollection` 与简单模型集合类具有相同的父树，但自身实现了很多 EAV 集合特定的方法，如 `addAttributeToFilter`、`addAttributeToSelect`、`addAttributeToSort` 等。

### 注意

如我们所见，EAV 模型看起来与简单模型非常相似。区别主要在于 `resource` 类和 `collection` 类的实现及其一级父类。然而，我们需要记住，这里给出的例子是最简单的一种。如果我们查看数据库中的 `eav_entity_type` 表，我们可以看到其他实体类型使用了 `attribute_model`、`entity_attribute_collection`、`increment_model` 等。这些都是我们可以定义在 EAV 模型旁边的先进属性，使其更接近 `catalog_product` 实体类型的实现，这可能是 Magento 中最健壮的一种。这种高级 EAV 使用超出了本书的范围，因为它可能值得一本自己的书。

现在我们已经设置了简单和 EAV 模型，是时候考虑安装必要的数据库模式和可能预先填充一些数据了。这是通过模式和数据脚本完成的。

# 理解模式和数据脚本流程

简而言之，模式脚本的作用是创建支持您模块逻辑的数据库结构。例如，创建一个表，我们的实体将在此表中持久化其数据。数据脚本的作用是管理现有表中的数据，通常以在模块安装期间添加一些示例数据的形式。

如果我们回顾几步，我们可以注意到数据库中的 `schema_version` 和 `data_version` 与我们的 `module.xml` 文件中的 `setup_version` 数字相匹配。它们都意味着相同的事情。如果我们现在更改 `module.xml` 文件中的 `setup_version` 数字并再次运行 `php bin/magento setup:upgrade` 控制台命令，我们的数据库 `schema_version` 和 `data_version` 将更新到这个新版本号。

这是通过模块的 `install` 和 `upgrade` 脚本来实现的。如果我们快速查看 `setup/src/Magento/Setup/Model/Installer.php` 文件，我们可以看到一个名为 `getSchemaDataHandler` 的函数，其内容如下：

```php
private function getSchemaDataHandler($moduleName, $type)
{
    $className = str_replace('_', '\\', $moduleName) . '\Setup';
    switch ($type) {
        case 'schema-install':
            $className .= '\InstallSchema';
            $interface = self::SCHEMA_INSTALL;
            break;
        case 'schema-upgrade':
            $className .= '\UpgradeSchema';
            $interface = self::SCHEMA_UPGRADE;
            break;
        case 'schema-recurring':
            $className .= '\Recurring';
            $interface = self::SCHEMA_INSTALL;
            break;
        case 'data-install':
            $className .= '\InstallData';
            $interface = self::DATA_INSTALL;
            break;
        case 'data-upgrade':
            $className .= '\UpgradeData';
            $interface = self::DATA_UPGRADE;
            break;
        default:
            throw new \Magento\Setup\Exception("$className does not exist");
    }

    return $this->createSchemaDataHandler($className, $interface);
}
```

这告诉 Magento 从单个模块的 `Setup` 目录中挑选和运行哪些类。目前我们将忽略重复的情况，因为只有 `Magento_Indexer` 模块使用它。

第一次运行 `php bin/magento setup:upgrade` 命令针对我们的模块；尽管它仍然在 `setup_module` 表下没有条目，但 Magento 将按照以下顺序执行模块 `Setup` 文件夹中的文件：

+   `InstallSchema.php`

+   `UpgradeSchema.php`

+   `InstallData.php`

+   `UpgradeData.php`

注意，这与 `getSchemaDataHandler` 方法中从上到下的顺序相同。

每次后续模块版本号的变化，随后是控制台命令 `php bin/magento setup:upgrade`，都会导致以下文件按照列表中的顺序运行：

+   `UpgradeSchema.php`

+   `UpgradeData.php`

此外，Magento 还会在 `setup_module` 数据库下记录升级后的版本号。只有当数据库中的版本号小于 `module.xml` 文件中的版本号时，Magento 才会触发安装或升级脚本。

### 小贴士

如果需要，我们不必总是提供这些安装或升级脚本。只有在需要添加或编辑数据库中的现有表或条目时才需要它们。

如果我们仔细查看适当脚本中 `install` 和 `update` 方法的实现，我们可以看到它们都接受 `ModuleContextInterface $context` 作为第二个参数。由于升级脚本是在每次升级版本号时触发的，我们可以使用 `$context->getVersion()` 来针对特定于模块版本的更改。

# 创建安装模式脚本（InstallSchema.php）

现在我们已经了解了模式和数据脚本及其与模块版本号的关系，让我们继续组装我们的 `InstallSchema`。我们首先定义 `app/code/Foggyline/Office/Setup/InstallSchema.php` 文件，其（部分）内容如下：

```php
namespace Foggyline\Office\Setup;

use Magento\Framework\Setup\InstallSchemaInterface;
use Magento\Framework\Setup\ModuleContextInterface;
use Magento\Framework\Setup\SchemaSetupInterface;

class InstallSchema implements InstallSchemaInterface
{
    public function install(SchemaSetupInterface $setup, ModuleContextInterface $context)
    {
        $setup->startSetup();
        /* #snippet1 */
        $setup->endSetup();
    }
}
```

`InstallSchema` 遵循 `InstallSchemaInterface`，这要求实现接受两个类型为 `SchemaSetupInterface` 和 `ModuleContextInterface` 的参数的 `install` 方法。

在这里只需要安装方法。在这个方法中，我们会添加任何可能需要的代码来创建所需的表和列。

在代码库中查看，我们可以看到 `Magento\Setup\Module\Setup` 是扩展 `\Magento\Framework\Module\Setup` 并实现 `SchemaSetupInterface` 的一个。前面代码中看到的两个方法 `startSetup` 和 `endSetup` 用于在我们代码之前和之后运行额外的环境设置。

进一步来说，让我们将 `/* #snippet1 */` 部分替换为创建我们的 `Department` 模型实体表的代码，如下所示：

```php
$table = $setup->getConnection()
    ->newTable($setup->getTable('foggyline_office_department'))
    ->addColumn(
        'entity_id',
        \Magento\Framework\DB\Ddl\Table::TYPE_INTEGER,
        null,
        ['identity' => true, 'unsigned' => true, 'nullable' => false, 'primary' => true],
        'Entity ID'
    )
    ->addColumn(
        'name',
        \Magento\Framework\DB\Ddl\Table::TYPE_TEXT,
        64,
        [],
        'Name'
    )
    ->setComment('Foggyline Office Department Table');
$setup->getConnection()->createTable($table);
/* #snippet2 */
```

在这里，我们指示 Magento 创建一个名为`foggyline_office_department`的表，向其中添加`entity_id`和`name`列，并设置表的注释。假设我们使用的是 MySQL 服务器，当代码执行时，以下 SQL 将在数据库中执行：

```php
CREATE TABLE 'foggyline_office_department' (
  'entity_id' int(10) unsigned NOT NULL AUTO_INCREMENT COMMENT 'Entity ID',
  'name' varchar(64) DEFAULT NULL COMMENT 'Name',
  PRIMARY KEY ('entity_id')
) ENGINE=InnoDB AUTO_INCREMENT=3 DEFAULT CHARSET=utf8 COMMENT='Foggyline Office Department Table';
```

`addColumn`方法是这里最有趣的一个。它接受五个参数，从列名、列数据类型、列长度、附加选项数组到列描述。然而，只有列名和列数据类型是必须的！可接受的列数据类型可以在`Magento\Framework\DB\Ddl\Table`类下找到，如下所示：

```php
boolean     smallint    integer     bigint
float       numeric     decimal     date
timestamp   datetime    text        blob
varbinary
```

附加选项数组可能包含以下一些键：`unsigned`、`precision`、`scale`、`unsigned`、`default`、`nullable`、`primary`、`identity`、`auto_increment`。

在了解了`addColumn`方法之后，让我们继续创建`foggyline_office_employee_entity`表，用于`Employee`实体。我们通过替换前面代码中的`/* #snippet2 */`部分来实现这一点：

```php
$employeeEntity = \Foggyline\Office\Model\Employee::ENTITY;
$table = $setup->getConnection()
    ->newTable($setup->getTable($employeeEntity . '_entity'))
    ->addColumn(
        'entity_id',
        \Magento\Framework\DB\Ddl\Table::TYPE_INTEGER,
        null,
        ['identity' => true, 'unsigned' => true, 'nullable' => false, 'primary' => true],
        'Entity ID'
    )
    ->addColumn(
        'department_id',
        \Magento\Framework\DB\Ddl\Table::TYPE_INTEGER,
        null,
        ['unsigned' => true, 'nullable' => false],
        'Department Id'
    )
    ->addColumn(
        'email',
        \Magento\Framework\DB\Ddl\Table::TYPE_TEXT,
        64,
        [],
        'Email'
    )
    ->addColumn(
        'first_name',
        \Magento\Framework\DB\Ddl\Table::TYPE_TEXT,
        64,
        [],
        'First Name'
    )
    ->addColumn(
        'last_name',
        \Magento\Framework\DB\Ddl\Table::TYPE_TEXT,
        64,
        [],
        'Last Name'
    )
    ->setComment('Foggyline Office Employee Table');
$setup->getConnection()->createTable($table);
/* #snippet3 */
```

根据良好的数据库设计实践，我们可能会注意到这里的一个问题。如果我们同意每个员工可以分配一个单一的部门，我们应该向该表的`department_id`列添加一个外键。目前，我们将故意跳过这一部分，因为我们想通过稍后的更新模式脚本来演示这一点。

EAV 模型将数据分散在多个表中，至少有三个。我们刚刚创建的`foggyline_office_employee_entity`表就是其中之一。另一个是核心的 Magento `eav_attribute`表。第三个表不是一个单独的表，而是一系列多个表；每个 EAV 类型一个表。这些表是我们安装脚本的结果。

存储在核心 Magento `eav_attribute`表中的信息不是属性值或类似的东西；存储在那里的信息是属性的元数据。那么，Magento 是如何知道我们的`Employee`属性（`service_years`、`dob`、`salary`、`vat_number`、`note`）的呢？它不知道；还没有。我们需要自己将属性添加到该表中。我们将在稍后通过`InstallData`演示这样做。

根据 EAV 属性数据类型，我们需要创建以下表：

+   `foggyline_office_employee_entity_datetime`

+   `foggyline_office_employee_entity_decimal`

+   `foggyline_office_employee_entity_int`

+   `foggyline_office_employee_entity_text`

+   `foggyline_office_employee_entity_varchar`

这些属性值表的名字来自一个简单的公式，即*{实体表名称}+{_}+{eav_attribute.backend_type 值}*。如果我们看`salary`属性，我们需要它是一个小数值，因此它将被存储在`foggyline_office_employee_entity_decimal`中。

由于定义属性值表背后的代码量很大，我们将只关注一个单一的小数类型表。我们通过替换前面代码中的`/* #snippet3 */`部分来实现这一点：

```php
$table = $setup->getConnection()
    ->newTable($setup->getTable($employeeEntity . '_entity_decimal'))
    ->addColumn(
        'value_id',
        \Magento\Framework\DB\Ddl\Table::TYPE_INTEGER,
        null,
        ['identity' => true, 'nullable' => false, 'primary' => true],
        'Value ID'
    )
    ->addColumn(
        'attribute_id',
        \Magento\Framework\DB\Ddl\Table::TYPE_SMALLINT,
        null,
        ['unsigned' => true, 'nullable' => false, 'default' => '0'],
        'Attribute ID'
    )
    ->addColumn(
        'store_id',
        \Magento\Framework\DB\Ddl\Table::TYPE_SMALLINT,
        null,
        ['unsigned' => true, 'nullable' => false, 'default' => '0'],
        'Store ID'
    )
    ->addColumn(
        'entity_id',
        \Magento\Framework\DB\Ddl\Table::TYPE_INTEGER,
        null,
        ['unsigned' => true, 'nullable' => false, 'default' => '0'],
        'Entity ID'
    )
    ->addColumn(
        'value',
        \Magento\Framework\DB\Ddl\Table::TYPE_DECIMAL,
        '12,4',
        [],
        'Value'
    )
    //->addIndex
    //->addForeignKey
    ->setComment('Employee Decimal Attribute Backend Table');
$setup->getConnection()->createTable($table);
```

注意上述代码中的 `//->addIndex` 部分。让我们将其替换为以下内容。

```php
->addIndex(
    $setup->getIdxName(
        $employeeEntity . '_entity_decimal',
        ['entity_id', 'attribute_id', 'store_id'],
        \Magento\Framework\DB\Adapter\AdapterInterface::INDEX_TYPE_UNIQUE
    ),
    ['entity_id', 'attribute_id', 'store_id'],
    ['type' => \Magento\Framework\DB\Adapter\AdapterInterface::INDEX_TYPE_UNIQUE]
)
->addIndex(
    $setup->getIdxName($employeeEntity . '_entity_decimal', ['store_id']),
    ['store_id']
)
->addIndex(
    $setup->getIdxName($employeeEntity . '_entity_decimal', ['attribute_id']),
    ['attribute_id']
)
```

前面的代码在 `foggyline_office_employee_entity_decimal` 表上添加了三个索引，结果生成以下 SQL：

+   `UNIQUE KEY 'FOGGYLINE_OFFICE_EMPLOYEE_ENTT_DEC_ENTT_ID_ATTR_ID_STORE_ID' ('entity_id','attribute_id','store_id')`

+   `KEY 'FOGGYLINE_OFFICE_EMPLOYEE_ENTITY_DECIMAL_STORE_ID' ('store_id')`

+   `KEY 'FOGGYLINE_OFFICE_EMPLOYEE_ENTITY_DECIMAL_ATTRIBUTE_ID' ('attribute_id')`

同样，我们将前面代码中的 `//->addForeignKey` 部分替换为以下内容：

```php
->addForeignKey(
    $setup->getFkName(
        $employeeEntity . '_entity_decimal',
        'attribute_id',
        'eav_attribute',
        'attribute_id'
    ),
    'attribute_id',
    $setup->getTable('eav_attribute'),
    'attribute_id',
    \Magento\Framework\DB\Ddl\Table::ACTION_CASCADE
)
->addForeignKey(
    $setup->getFkName(
        $employeeEntity . '_entity_decimal',
        'entity_id',
        $employeeEntity . '_entity',
        'entity_id'
    ),
    'entity_id',
    $setup->getTable($employeeEntity . '_entity'),
    'entity_id',
    \Magento\Framework\DB\Ddl\Table::ACTION_CASCADE
)
->addForeignKey(
    $setup->getFkName($employeeEntity . '_entity_decimal', 'store_id', 'store', 'store_id'),
    'store_id',
    $setup->getTable('store'),
    'store_id',
    \Magento\Framework\DB\Ddl\Table::ACTION_CASCADE
)
```

前面的代码将外键关系添加到 `foggyline_office_employee_entity_decimal` 表中，结果生成以下 SQL：

+   `CONSTRAINT 'FK_D17982EDA1846BAA1F40E30694993801' FOREIGN KEY ('entity_id') REFERENCES 'foggyline_office_employee_entity' ('entity_id') ON DELETE CASCADE,`

+   `CONSTRAINT 'FOGGYLINE_OFFICE_EMPLOYEE_ENTITY_DECIMAL_STORE_ID_STORE_STORE_ID' FOREIGN KEY ('store_id') REFERENCES 'store' ('store_id') ON DELETE CASCADE,`

+   `CONSTRAINT 'FOGGYLINE_OFFICE_EMPLOYEE_ENTT_DEC_ATTR_ID_EAV_ATTR_ATTR_ID' FOREIGN KEY ('attribute_id') REFERENCES 'eav_attribute' ('attribute_id') ON DELETE CASCADE`

注意我们如何将 `store_id` 列添加到我们的 EAV 属性值表中。尽管我们的示例不会使用它，但使用 `store_id` 与 EAV 实体一起定义数据范围是一个好习惯。为了进一步说明，想象我们有一个多店铺设置，并且像前面的 EAV 属性表那样设置，我们就能为每个店铺存储不同的属性值，因为表中的唯一条目定义为 `entity_id`、`attribute_id` 和 `store_id` 列的组合。

### 小贴士

为了性能和数据完整性的原因，根据良好的数据库设计实践定义索引和外键非常重要。我们可以在定义新表时在 `InstallSchema` 中这样做。

# 创建升级模式脚本（UpgradeSchema.php）

在第一次模块安装期间，安装模式之后立即运行升级模式。我们在 `app/code/Foggyline/Office/Setup/UpgradeSchema.php` 文件中定义升级模式，其部分内容如下：

```php
namespace Foggyline\Office\Setup;

use Magento\Framework\Setup\UpgradeSchemaInterface;
use Magento\Framework\Setup\ModuleContextInterface;
use Magento\Framework\Setup\SchemaSetupInterface;

class UpgradeSchema implements UpgradeSchemaInterface
{
    public function upgrade(SchemaSetupInterface $setup, ModuleContextInterface $context)
    {
        $setup->startSetup();
          /* #snippet1 */
        $setup->endSetup();
    }
}
```

`UpgradeSchema` 遵循 `UpgradeSchemaInterface`，这要求实现接受两个参数 `SchemaSetupInterface` 和 `ModuleContextInterface` 的 `upgrade` 方法。

这与 `InstallSchemaInterface` 非常相似，除了方法名。当此模式被触发时，会运行 `update` 方法。在这个方法中，我们会添加任何我们可能想要执行的代码。

进一步来说，让我们将前面代码中的 `/* #snippet1 */` 部分替换为以下代码：

```php
$employeeEntityTable = \Foggyline\Office\Model\Employee::ENTITY. '_entity';
$departmentEntityTable = 'foggyline_office_department';

$setup->getConnection()
    ->addForeignKey(
        $setup->getFkName($employeeEntityTable, 'department_id', $departmentEntityTable, 'entity_id'),
        $setup->getTable($employeeEntityTable),
        'department_id',
        $setup->getTable($departmentEntityTable),
        'entity_id',
        \Magento\Framework\DB\Ddl\Table::ACTION_CASCADE
    );
```

在这里，我们指示 Magento 在 `foggyline_office_employee_entity` 表上创建一个外键，更确切地说是在其 `department_id` 列上，指向 `foggyline_office_department` 表及其 `entity_id` 列。

# 创建安装数据脚本（InstallData.php）

安装数据脚本是在升级架构后立即运行的。我们在`app/code/Foggyline/Office/Setup/InstallData.php`文件中定义安装数据架构，其内容如下（部分）：

```php
namespace Foggyline\Office\Setup;

use Magento\Framework\Setup\InstallDataInterface;
use Magento\Framework\Setup\ModuleContextInterface;
use Magento\Framework\Setup\ModuleDataSetupInterface;

class InstallData implements InstallDataInterface
{
    private $employeeSetupFactory;

    public function __construct(
        \Foggyline\Office\Setup\EmployeeSetupFactory $employeeSetupFactory
    )
    {
        $this->employeeSetupFactory = $employeeSetupFactory;
    }

    public function install(ModuleDataSetupInterface $setup, ModuleContextInterface $context)
    {
        $setup->startSetup();
        /* #snippet1 */
        $setup->endSetup();
    }
}
```

`InstallData`符合`InstallDataInterface`，这要求实现接受两个类型为`ModuleDataSetupInterface`和`ModuleContextInterface`的参数的`install`方法。

当此脚本被触发时，将运行`install`方法。在这个方法中，我们会添加任何可能想要执行的代码。

进一步来说，让我们用以下代码替换前面代码中的`/* #snippet1 */`部分：

```php
$employeeEntity = \Foggyline\Office\Model\Employee::ENTITY;

$employeeSetup = $this->employeeSetupFactory->create(['setup' => $setup]);

$employeeSetup->installEntities();

$employeeSetup->addAttribute(
    $employeeEntity, 'service_years', ['type' => 'int']
);

$employeeSetup->addAttribute(
    $employeeEntity, 'dob', ['type' => 'datetime']
);

$employeeSetup->addAttribute(
    $employeeEntity, 'salary', ['type' => 'decimal']
);

$employeeSetup->addAttribute(
    $employeeEntity, 'vat_number', ['type' => 'varchar']
);

$employeeSetup->addAttribute(
    $employeeEntity, 'note', ['type' => 'text']
);
```

使用`\Foggyline\Office\Setup\EmployeeSetupFactory`实例上的`addAttribute`方法，我们指示 Magento 向其实体添加多个属性（`service_years`、`dob`、`salary`、`vat_number`、`note`）。

我们很快就会进入`EmployeeSetupFactory`的内部，但现在请注意对`addAttribute`方法的调用。在这个方法中，有一个对`$this->attributeMapper->map($attr, $entityTypeId)`方法的调用。`attributeMapper`符合`Magento\Eav\Model\Entity\Setup\PropertyMapperInterface`，查看`vendor/magento/module-eav/etc/di.xml`，它对`Magento\Eav\Model\Entity\Setup\PropertyMapper\Composite`类有优先权，该类进一步初始化以下映射类：

+   `Magento\Eav\Model\Entity\Setup\PropertyMapper`

+   `Magento\Customer\Model\ResourceModel\Setup\PropertyMapper`

+   `Magento\Catalog\Model\ResourceModel\Setup\PropertyMapper`

+   `Magento\ConfigurableProduct\Model\ResourceModel\Setup\PropertyMapper`

由于我们正在定义自己的实体类型，我们主要感兴趣的映射类是`Magento\Eav\Model\Entity\Setup\PropertyMapper`。快速查看它，我们发现`map`方法中的以下映射数组：

```php
[
    'backend_model' => 'backend',
    'backend_type' => 'type',
    'backend_table' => 'table',
    'frontend_model' => 'frontend',
    'frontend_input' => 'input',
    'frontend_label' => 'label',
    'frontend_class' => 'frontend_class',
    'source_model' => 'source',
    'is_required' => 'required',
    'is_user_defined' => 'user_defined',
    'default_value' => 'default',
    'is_unique' => 'unique',
    'note' => 'note'
    'is_global' => 'global'
]
```

通过查看前面的数组键和值字符串，我们可以了解正在发生的事情。键字符串与`eav_attribute`表中的列名匹配，而值字符串与我们在`InstallData.php`中通过`addAttribute`方法传递给数组的键匹配。

让我们看看`app/code/Foggyline/Office/Setup/EmployeeSetup.php`文件中的`EmployeeSetupFactory`类，其定义如下（部分）：

```php
namespace Foggyline\Office\Setup;
use Magento\Eav\Setup\EavSetup;

class EmployeeSetup extends EavSetup
{
    public function getDefaultEntities()
    {
        /* #snippet1 */
    }
}
```

这里发生的事情是我们从`Magento\Eav\Setup\EavSetup`类扩展，从而有效地告诉 Magento 我们即将创建自己的实体。我们通过重写`getDefaultEntities`，用以下内容替换`/* #snippet1 */`来实现这一点：

```php
$employeeEntity = \Foggyline\Office\Model\Employee::ENTITY;
$entities = [
    $employeeEntity => [
        'entity_model' => 'Foggyline\Office\Model\ResourceModel\Employee',
        'table' => $employeeEntity . '_entity',
        'attributes' => [
            'department_id' => [
                'type' => 'static',
            ],
            'email' => [
                'type' => 'static',
            ],

            'first_name' => [
                'type' => 'static',
            ],
            'last_name' => [
                'type' => 'static',
            ],
        ],
    ],
];
return $entities;
```

`getDefaultEntities`方法返回一个我们想要与 Magento 注册的实体数组。在我们的`$entities`数组中，键`$employeeEntity`成为`eav_entity_type`表中的一个条目。鉴于我们的`$employeeEntity`的值为`foggyline_office_employee`，运行以下 SQL 查询应该会产生结果：

```php
SELECT * FROM eav_entity_type WHERE entity_type_code = "foggyline_office_employee";
```

为了使我们的新实体类型能够正常工作，只需要少量元数据值。`entity_model` 值应指向我们的 EAV 模型 `resource` 类，而不是 `model` 类。表值应等于数据库中我们的 EAV 实体表名称。最后，属性数组应列出我们想要在此实体上创建的任何属性。属性及其元数据在 `eav_attribute` 表中创建。

如果我们回顾一下我们创建的所有那些 `foggyline_office_employee_entity_*` 属性值表，它们并不是真正创建属性或注册新实体类型的。创建属性和新实体类型的是我们在 `getDefaultEntities` 方法下定义的数组。一旦 Magento 创建了属性并注册了新实体类型，它就会将实体保存过程路由到适当的属性值表，具体取决于属性类型。

# 创建升级数据脚本（UpgradeData.php）

升级数据脚本是在最后执行的。我们将使用它来演示为我们的 `Department` 和 `Employee` 实体创建示例条目的示例。

我们首先创建 `app/code/Foggyline/Office/Setup/UpgradeData.php` 文件，其（部分）内容如下：

```php
namespace Foggyline\Office\Setup;

use Magento\Framework\Setup\UpgradeDataInterface;
use Magento\Framework\Setup\ModuleContextInterface;
use Magento\Framework\Setup\ModuleDataSetupInterface;

class UpgradeData implements UpgradeDataInterface
{
    protected $departmentFactory;
    protected $employeeFactory;

    public function __construct(
        \Foggyline\Office\Model\DepartmentFactory $departmentFactory,
        \Foggyline\Office\Model\EmployeeFactory $employeeFactory
    )
    {
        $this->departmentFactory = $departmentFactory;
        $this->employeeFactory = $employeeFactory;
    }

    public function upgrade(ModuleDataSetupInterface $setup, ModuleContextInterface $context)
    {
        $setup->startSetup();
        /* #snippet1 */
        $setup->endSetup();
    }
}
```

`UpgradeData` 遵循 `UpgradeDataInterface`，该接口要求实现接受两个参数（类型为 `ModuleDataSetupInterface` 和 `ModuleContextInterface`）的升级方法。我们进一步添加了自己的 `__construct` 方法，将 `DepartmentFactory` 和 `EmployeeFactory` 传递给它，因为我们将在下一个示例中在升级方法中使用它们，通过将 `/* #snippet1 */` 替换为以下代码：

```php
$salesDepartment = $this->departmentFactory->create();
$salesDepartment->setName('Sales');
$salesDepartment->save();

$employee = $this->employeeFactory->create();
$employee->setDepartmentId($salesDepartment->getId());
$employee->setEmail('john@sales.loc');
$employee->setFirstName('John');
$employee->setLastName('Doe');
$employee->setServiceYears(3);
$employee->setDob('1983-03-28');
$employee->setSalary(3800.00);
$employee->setVatNumber('GB123456789');
$employee->setNote('Just some notes about John');
$employee->save();
```

上述代码创建了一个部门实体的实例并将其保存。然后创建了一个员工实例并保存，传递给它新创建的部门 ID 和其他属性。

### 小贴士

保存实体的一种更方便、更专业的做法可以是以下这样：

```php
$employee->setDob('1983-03-28')
    ->setSalary(3800.00)
    ->setVatNumber('GB123456789')
    ->save();
```

在这里，我们利用了每个实体设置方法都返回 `$this`（实体对象本身的实例）的事实，因此我们可以链式调用方法。

# 实体 CRUD 操作

到目前为止，我们已经学习了如何创建一个简单的模型、一个 EAV 模型，以及安装和升级类型的数据脚本。现在，让我们看看我们如何创建、读取、更新和删除我们的实体，这些操作通常被称为 CRUD。

虽然这一章是关于模型、集合和相关内容的，但为了演示的目的，让我们稍微偏离一下路由和控制器。想法是创建一个简单的 `Test` 控制器，我们可以通过 URL 触发其 `Crud` 动作。然后，在 `Crud` 动作中，我们将输出我们的 CRUD 相关代码。

要使 Magento 对应于我们在浏览器中输入的 URL，我们需要定义路由。我们通过创建包含以下内容的 `app/code/Foggyline/Office/etc/frontend/routes.xml` 文件来实现：

```php
<config  xsi:noNamespaceSchemaLocation="urn:magento:framework:App/ etc/routes.xsd">
    <router id="standard">
        <route id="foggyline_office" frontName="foggyline_office">
            <module name="Foggyline_Office"/>
        </route>
    </router>
</config>
```

路由定义需要唯一的 ID 和`frontName`属性值，在我们的案例中这两个都等于`foggyline_office`。`frontName`属性值成为我们 URL 结构的一部分。简单来说，访问`Crud`操作的 URL 公式如下：*{magento-base-url}/index.php/{route frontName}/{controller name}/{action name}*

### 注意

例如，如果我们的基础 URL 是`http://shop.loc/`，完整的 URL 将是`http://shop.loc/index.php/foggyline_office/test/crud/`。如果我们启用了 URL 重写，我们可以省略`index.php`部分。

一旦定义了路由，我们就可以继续创建`Test`控制器，它在`app/code/Foggyline/Office/Controller/Test.php`文件中定义（部分）代码如下：

```php
namespace Foggyline\Office\Controller;

abstract class Test extends \Magento\Framework\App\Action\Action
{
}
```

这真的是我们能够定义的最简单的控制器。这里唯一值得注意的事情是，控制器类需要定义为抽象类，并扩展`\Magento\Framework\App\Action\Action`类。控制器动作位于控制器外部，可以在同一级别的子目录下找到，命名为控制器。由于我们的控制器名为`Test`，我们将我们的`Crud`动作放在`app/code/Foggyline/Office/Controller/Test/Crud.php`文件中，内容如下：

```php
namespace Foggyline\Office\Controller\Test;

class Crud extends \Foggyline\Office\Controller\Test
{
    protected $employeeFactory;
    protected $departmentFactory;

    public function __construct(
        \Magento\Framework\App\Action\Context $context,
        \Foggyline\Office\Model\EmployeeFactory $employeeFactory,
        \Foggyline\Office\Model\DepartmentFactory $departmentFactory
    )
    {
        $this->employeeFactory = $employeeFactory;
        $this->departmentFactory = $departmentFactory;
        return parent::__construct($context);
    }

    public function execute()
    {
        /* CRUD Code Here */
    }
}
```

`Controller`动作类基本上是控制器定义`execute`方法的扩展。当我们在浏览器中点击 URL 时，会运行`execute`方法中的代码。此外，我们还有一个`__construct`方法，我们将`EmployeeFactory`和`DepartmentFactory`类传递给它，我们将在我们的 CRUD 示例中使用这些类。请注意，`EmployeeFactory`和`DepartmentFactory`不是我们创建的类。Magento 将在`var/generation/Foggyline/Office/Model`文件夹中的`DepartmentFactory.php`和`EmployeeFactory.php`文件下自动生成它们。这些是我们`Employee`和`Department`模型类的工厂类，在请求时生成。

有了这个，我们就结束了这个小插曲，重新关注我们的实体。

# 创建新实体

如果我们可以称它们为不同的风味，我们有三种方式可以设置实体（字段和属性）的属性值。它们都会产生相同的结果。以下几个代码片段可以复制粘贴到我们的`Crud`类的`execute`方法中进行测试，只需将`/* CRUD Code Here */`替换为以下代码片段之一：

```php
//Simple model, creating new entities, flavour #1
$department1 = $this->departmentFactory->create();
$department1->setName('Finance');
$department1->save();
//Simple model, creating new entities, flavour #2
$department2 = $this->departmentFactory->create();
$department2->setData('name', 'Research');
$department2->save();
//Simple model, creating new entities, flavour #3
$department3 = $this->departmentFactory->create();
$department3->setData(['name' => 'Support']);
$department3->save();
```

前面代码中的`flavour #1`方法可能是设置属性的首选方式，因为它使用了我们之前提到的魔术方法方法。`flavour #2`和`flavour #3`都使用了`setData`方法，只是方式略有不同。一旦在`object`实例上调用`save`方法，这三个示例都应该产生相同的结果。

现在我们知道了如何保存简单的模型，让我们快速看一下如何使用 EAV 模型做同样的事情。以下是对应的代码片段：

```php
//EAV model, creating new entities, flavour #1
$employee1 = $this->employeeFactory->create();
$employee1->setDepartment_id($department1->getId());
$employee1->setEmail('goran@mail.loc');
$employee1->setFirstName('Goran');
$employee1->setLastName('Gorvat');
$employee1->setServiceYears(3);
$employee1->setDob('1984-04-18');
$employee1->setSalary(3800.00);
$employee1->setVatNumber('GB123451234');
$employee1->setNote('Note #1');
$employee1->save();

//EAV model, creating new entities, flavour #2
$employee2 = $this->employeeFactory->create();
$employee2->setData('department_id', $department2->getId());
$employee2->setData('email', 'marko@mail.loc');
$employee2->setData('first_name', 'Marko');
$employee2->setData('last_name', 'Tunukovic');
$employee2->setData('service_years', 3);
$employee2->setData('dob', '1984-04-18');
$employee2->setData('salary', 3800.00);
$employee2->setData('vat_number', 'GB123451234');
$employee2->setData('note', 'Note #2');
$employee2->save();

//EAV model, creating new entities, flavour #3
$employee3 = $this->employeeFactory->create();
$employee3->setData([
    'department_id' => $department3->getId(),
    'email' => 'ivan@mail.loc',
    'first_name' => 'Ivan',
    'last_name' => 'Telebar',
    'service_years' => 2,
    'dob' => '1986-08-22',
    'salary' => 2400.00,
    'vat_number' => 'GB123454321',
    'note' => 'Note #3'
]);
$employee3->save();
```

如我们所见，用于持久化数据的 EAV 代码与简单模型相同。这里有一件事值得注意。`Employee`实体定义了一个指向部门的关联。忘记在新的`employee`实体保存时指定`department_id`会导致类似于以下错误信息的错误：

```php
SQLSTATE[23000]: Integrity constraint violation: 1452 Cannot add or update a child row: a foreign key constraint fails ('magento'.'foggyline_office_employee_entity', CONSTRAINT 'FK_E2AEE8BF21518DFA8F02B4E95DC9F5AD' FOREIGN KEY ('department_id') REFERENCES 'foggyline_office_department' ('entity_id') ON), query was: INSERT INTO 'foggyline_office_employee_entity' ('email', 'first_name', 'last_name', 'entity_id') VALUES (?, ?, ?, ?)
```

Magento 将其中的这些类型错误保存在其`var/report`目录下。

## 读取现有实体

根据提供的实体 ID 值读取实体归结为实例化实体，并使用传递实体 ID 的`load`方法，如下所示：

```php
//Simple model, reading existing entities
$department = $this->departmentFactory->create();
$department->load(28);

/*
    \Zend_Debug::dump($department->toArray());

    array(2) {
      ["entity_id"] => string(2) "28"
      ["name"] => string(8) "Research"
    }
 */
```

如下所示，加载简单模型或 EAV 模型之间实际上没有真正的区别：

```php
//EAV model, reading existing entities
$employee = $this->employeeFactory->create();
$employee->load(25);

/*
    \Zend_Debug::dump($employee->toArray());

    array(10) {
      ["entity_id"] => string(2) "25"
      ["department_id"] => string(2) "28"
      ["email"] => string(14) "marko@mail.loc"
      ["first_name"] => string(5) "Marko"
      ["last_name"] => string(9) "Tunukovic"
      ["dob"] => string(19) "1984-04-18 00:00:00"
      ["note"] => string(7) "Note #2"
      ["salary"] => string(9) "3800.0000"
      ["service_years"] => string(1) "3"
      ["vat_number"] => string(11) "GB123451234"
    }
 */
```

注意到 EAV 实体加载了所有的字段和属性值，这在我们通过 EAV 集合获取实体时并不总是如此，我们将在稍后展示。

## 更新现有实体

更新实体归结为使用`load`方法读取现有实体，重置其值，并在最后调用`save`方法，如下例所示：

```php
$department = $this->departmentFactory->create();
$department->load(28);
$department->setName('Finance #2');
$department->save();
```

无论实体是简单模型还是 EAV，代码都是相同的。

## 删除现有实体

在已加载的实体上调用`delete`方法将删除该实体从数据库中，或者在失败时抛出`Exception`。删除实体的代码如下所示：

```php
$employee = $this->employeeFactory->create();
$employee->load(25);
$employee->delete();
```

删除简单实体和 EAV 实体之间没有区别。我们在删除或保存实体时应该始终使用 try/catch 块。

# 管理集合

让我们从 EAV 模型集合开始。我们可以通过以下方式通过实体`factory`类实例化集合：

```php
$collection = $this->employeeFactory->create()
                   ->getCollection();
```

或者我们可以使用对象管理器来实例化集合，如下所示：

```php
$collection = $this->_objectManager->create(
    'Foggyline\Office\Model\ResourceModel\Employee\Collection's
);
```

还有第三种方法，这可能是一个更受欢迎的方法，但它要求我们定义 API，所以我们暂时跳过这个方法。

一旦我们实例化了集合对象，我们就可以遍历它并对单个`$employee`实体进行一些变量转储，以查看其内容，如下所示：

```php
foreach ($collection as $employee) {
    \Zend_Debug::dump($employee->toArray(), '$employee');
}
```

前面的操作会产生如下结果：

```php
$employee array(5) {
  ["entity_id"] => string(2) "24"
  ["department_id"] => string(2) "27"
  ["email"] => string(14) "goran@mail.loc"
  ["first_name"] => string(5) "Goran"
  ["last_name"] => string(6) "Gorvat"
}
```

注意到单个`$employee`只有字段，没有属性。让我们看看当我们想要通过使用`addAttributeToSelect`来指定要添加到集合中的单个属性时会发生什么，如下所示：

```php
$collection->addAttributeToSelect('salary')
           ->addAttributeToSelect('vat_number');
```

前面的操作会产生如下结果：

```php
$employee array(7) {
  ["entity_id"] => string(2) "24"
  ["department_id"] => string(2) "27"
  ["email"] => string(14) "goran@mail.loc"
  ["first_name"] => string(5) "Goran"
  ["last_name"] => string(6) "Gorvat"
  ["salary"] => string(9) "3800.0000"
  ["vat_number"] => string(11) "GB123451234"
}
```

虽然我们在取得进步，但想象一下如果我们有数十个属性，并且我们希望每个属性都包含在集合中。多次使用`addAttributeToSelect`会导致代码杂乱。我们可以通过将`'*'`作为参数传递给`addAttributeToSelect`，让集合获取每个属性，如下所示：

```php
$collection->addAttributeToSelect('*');
```

这会产生如下结果：

```php
$employee array(10) {
    ["entity_id"] => string(2) "24"
    ["department_id"] => string(2) "27"
    ["email"] => string(14) "goran@mail.loc"
    ["first_name"] => string(5) "Goran"
    ["last_name"] => string(6) "Gorvat"
    ["dob"] => string(19) "1984-04-18 00:00:00"
    ["note"] => string(7) "Note #1"
    ["salary"] => string(9) "3800.0000"
    ["service_years"] => string(1) "3"
    ["vat_number"] => string(11) "GB123451234"
}
```

虽然代码的 PHP 部分看起来似乎很简单，但在 SQL 层面上发生的事情相对复杂。虽然 Magento 在获取最终集合结果之前执行了几个 SQL 查询，但让我们关注下面显示的最后三个查询：

```php
SELECT COUNT(*) FROM 'foggyline_office_employee_entity' AS 'e'

SELECT 'e'.* FROM 'foggyline_office_employee_entity' AS 'e'

SELECT
  'foggyline_office_employee_entity_datetime'.'entity_id',
  'foggyline_office_employee_entity_datetime'.'attribute_id',
  'foggyline_office_employee_entity_datetime'.'value'
FROM 'foggyline_office_employee_entity_datetime'
WHERE (entity_id IN (24, 25, 26)) AND (attribute_id IN ('349'))
UNION ALL SELECT
            'foggyline_office_employee_entity_text'.'entity_id',
            'foggyline_office_employee_entity_text'.' attribute_id',
            'foggyline_office_employee_entity_text'.'value'
          FROM 'foggyline_office_employee_entity_text'
          WHERE (entity_id IN (24, 25, 26)) AND (attribute_id IN ('352'))
UNION ALL SELECT
            'foggyline_office_employee_entity_decimal'.' entity_id',
            'foggyline_office_employee_entity_decimal'.' attribute_id',
            'foggyline_office_employee_entity_decimal'.'value'
          FROM 'foggyline_office_employee_entity_decimal'
          WHERE (entity_id IN (24, 25, 26)) AND (attribute_id IN ('350'))
UNION ALL SELECT
            'foggyline_office_employee_entity_int'.'entity_id',
            'foggyline_office_employee_entity_int'.'attribute_id',
            'foggyline_office_employee_entity_int'.'value'
          FROM 'foggyline_office_employee_entity_int'
          WHERE (entity_id IN (24, 25, 26)) AND (attribute_id IN ('348'))
UNION ALL SELECT
            'foggyline_office_employee_entity_varchar'.' entity_id',
            'foggyline_office_employee_entity_varchar'.' attribute_id',
            'foggyline_office_employee_entity_varchar'.'value'
          FROM 'foggyline_office_employee_entity_varchar'
          WHERE (entity_id IN (24, 25, 26)) AND (attribute_id IN ('351'))
```

### 注意

在我们继续之前，了解这些查询不能直接复制粘贴是很重要的。原因是 `attribute_id` 值肯定会在不同的安装中有所不同。这里给出的查询是为了让我们在 PHP 应用程序级别使用 Magento 集合时，在 SQL 层面上获得对后台发生的事情的高级理解。

第一个查询只是简单地计算实体表中的条目数量，然后将这个信息传递到应用层。第二个查询从 `foggyline_office_employee_entity` 中检索所有条目，然后将这些信息传递到应用层，以便在第三个查询中将实体 ID 作为 `entity_id IN (24, 25, 26)` 的一部分传递。如果我们的实体和 EAV 表中有大量条目，这里的第二个和第三个查询可能会非常消耗资源。为了防止可能出现的性能瓶颈，我们应该始终在集合上使用 `setPageSize` 和 `setCurPage` 方法，如下所示：

```php
$collection->addAttributeToSelect('*')
           ->setPageSize(25)
           ->setCurPage(5);
```

这会导致第一个 `COUNT` 查询仍然保持不变，但第二个查询现在看起来如下所示：

```php
SELECT 'e'.* FROM 'foggyline_office_employee_entity' AS 'e' LIMIT 25 OFFSET 4
```

如果我们有成千上万或数万条条目，这将使得数据集更小，从而性能更轻。这里的要点是始终使用 `setPageSize` 和 `setCurPage`。如果我们需要处理一个非常大的集合，那么我们需要分页浏览它，或者逐个遍历它。

现在我们知道了如何限制结果集的大小并获取正确的页面，让我们看看我们如何进一步过滤集合以避免过度使用 PHP 循环来完成同样的目的。因此，有效地将过滤传递到数据库而不是应用层。为了过滤 EAV 集合，我们使用它的 `addAttributeToFilter` 方法。

让我们实例化一个干净的新集合，如下所示：

```php
$collection = $this->_objectManager->create(
    'Foggyline\Office\Model\ResourceModel\Employee\Collection'
);

$collection->addAttributeToSelect('*')
           ->setPageSize(25)
           ->setCurPage(1);

$collection->addAttributeToFilter('email', array('like'=>'%mail.loc%'))
           ->addAttributeToFilter('vat_number', array('like'=>'GB%'))
           ->addAttributeToFilter('salary', array('gt'=>2400))
           ->addAttributeToFilter('service_years', array('lt'=>10));
```

注意我们现在正在集合上使用 `addAttributeToSelect` 和 `addAttributeToFilter` 方法。我们已经看到了 `addAttributeToSelect` 对 SQL 查询的数据库影响。`addAttributeToFilter` 做的事情则完全不同。

使用 `addAttributeToFilter` 方法，计数查询现在被转换成以下 SQL 查询：

```php
SELECT COUNT(*)
FROM 'foggyline_office_employee_entity' AS 'e'
  INNER JOIN 'foggyline_office_employee_entity_varchar' AS 'at_vat_number'
    ON ('at_vat_number'.'entity_id' = 'e'.'entity_id') AND ('at_vat_number'.'attribute_id' = '351')
  INNER JOIN 'foggyline_office_employee_entity_decimal' AS 'at_salary'
    ON ('at_salary'.'entity_id' = 'e'.'entity_id') AND ('at_salary'.'attribute_id' = '350')
  INNER JOIN 'foggyline_office_employee_entity_int' AS 'at_service_years'
    ON ('at_service_years'.'entity_id' = 'e'.'entity_id') AND ('at_service_years'.'attribute_id' = '348')
WHERE ('e'.'email' LIKE '%mail.loc%') AND (at_vat_number.value LIKE 'GB%') AND (at_salary.value > 2400) AND
      (at_service_years.value < 10)
```

我们可以看到这比之前的计数查询要复杂得多，现在有 `INNER JOIN` 介入。注意我们如何有四个 `addAttributeToFilter` 方法调用，但只有三个 `INNER JOIN`。这是因为其中四个调用之一是用于电子邮件的，它不是一个属性，而是 `foggyline_office_employee_entity` 表中的一个字段。这就是为什么不需要 `INNER JOIN`，因为该字段已经存在。然后三个 `INNER JOIN` 简单地将所需信息合并到查询中，以获取选择。

第二个查询也变得更加健壮，如下所示：

```php
SELECT
  'e'.*,
  'at_vat_number'.'value'    AS 'vat_number',
  'at_salary'.'value'        AS 'salary',
  'at_service_years'.'value' AS 'service_years'
FROM 'foggyline_office_employee_entity' AS 'e'
  INNER JOIN 'foggyline_office_employee_entity_varchar' AS 'at_vat_number'
    ON ('at_vat_number'.'entity_id' = 'e'.'entity_id') AND ('at_vat_number'.'attribute_id' = '351')
  INNER JOIN 'foggyline_office_employee_entity_decimal' AS 'at_salary'
    ON ('at_salary'.'entity_id' = 'e'.'entity_id') AND ('at_salary'.'attribute_id' = '350')
  INNER JOIN 'foggyline_office_employee_entity_int' AS 'at_service_years'
    ON ('at_service_years'.'entity_id' = 'e'.'entity_id') AND ('at_service_years'.'attribute_id' = '348')
WHERE ('e'.'email' LIKE '%mail.loc%') AND (at_vat_number.value LIKE 'GB%') AND (at_salary.value > 2400) AND
      (at_service_years.value < 10)
LIMIT 25
```

这里，我们还可以看到 `INNER JOIN` 的使用。我们也有三个而不是四个 `INNER JOIN`，因为其中一个条件是对 `email` 字段的。查询的结果是一个扁平化的行块，其中包含 `vat_number`、`salary` 和 `service_years` 等属性。我们可以想象如果没有使用 `setPageSize` 限制结果集，性能会受到怎样的影响。

最后，第三个查询也受到影响，现在看起来类似于以下内容：

```php
SELECT
  'foggyline_office_employee_entity_datetime'.'entity_id',
  'foggyline_office_employee_entity_datetime'.'attribute_id',
  'foggyline_office_employee_entity_datetime'.'value'
FROM 'foggyline_office_employee_entity_datetime'
WHERE (entity_id IN (24, 25)) AND (attribute_id IN ('349'))
UNION ALL SELECT
            'foggyline_office_employee_entity_text'.'entity_id',
            'foggyline_office_employee_entity_text'.' attribute_id',
            'foggyline_office_employee_entity_text'.'value'
          FROM 'foggyline_office_employee_entity_text'
          WHERE (entity_id IN (24, 25)) AND (attribute_id IN ('352'))
```

注意这里 `UNION ALL` 已经减少到单个出现，从而有效地形成了两个选择。这是因为我们总共有五个属性（`service_years`、`dob`、`salary`、`vat_number`、`note`），其中三个是通过第二个查询获取的。在前面的三个查询示例中，Magento 主要从第二个和第三个查询中提取集合数据。这看起来像是一个相当优化和可扩展的解决方案，尽管我们真的应该仔细思考在创建集合时正确使用 `setPageSize`、`addAttributeToSelect` 和 `addAttributeToFilter` 方法。

在开发过程中，如果正在处理具有大量属性、过滤器和可能的大型数据集的集合，我们可能希望使用 SQL 日志记录实际击中数据库服务器的 SQL 查询。这可能会帮助我们及时发现可能的性能瓶颈并及时做出反应，无论是通过向 `setPageSize` 或 `addAttributeToSelect` 添加更多限制值，还是两者都添加。

在前面的示例中，使用 `addAttributeToSelect` 导致 SQL 层上的 `AND` 条件。如果我们想使用 `OR` 条件来过滤集合怎么办？如果 `$attribute` 参数以以下方式使用，`addAttributeToSelect` 也可以导致 SQL `OR` 条件：

```php
$collection->addAttributeToFilter([
    ['attribute'=>'salary', 'gt'=>2400],
    ['attribute'=>'vat_number', 'like'=>'GB%']
]);
```

这次不深入实际 SQL 查询的细节，只需说它们几乎与之前的示例相同，使用了 `addAttributeToFilter` 的 `AND` 条件。

使用 `addExpressionAttributeToSelect`、`groupByAttribute` 和 `addAttributeToSort` 等集合方法，集合提供了进一步的梯度过滤，甚至可以将一些计算从 PHP 应用层转移到 SQL 层。深入了解这些和其他集合方法超出了本章的范围，可能需要一本单独的书籍。

## 集合过滤器

回顾前面的 `addAttributeToFilter` 方法调用示例，人们可能会问在哪里可以看到所有可用的集合过滤器的列表。如果我们快速查看 `vendor/magento/framework/DB/Adapter/Pdo/Mysql.php` 文件，我们可以看到名为 `prepareSqlCondition` 的方法（部分）定义如下：

```php
public function prepareSqlCondition($fieldName, $condition)
{
    $conditionKeyMap = [
        'eq'            => "{{fieldName}} = ?",
        'neq'           => "{{fieldName}} != ?",
        'like'          => "{{fieldName}} LIKE ?",
        'nlike'         => "{{fieldName}} NOT LIKE ?",
        'in'            => "{{fieldName}} IN(?)",
        'nin'           => "{{fieldName}} NOT IN(?)",
        'is'            => "{{fieldName}} IS ?",
        'notnull'       => "{{fieldName}} IS NOT NULL",
        'null'          => "{{fieldName}} IS NULL",
        'gt'            => "{{fieldName}} > ?",
        'lt'            => "{{fieldName}} /* AJZELE */ < ?",
        'gteq'          => "{{fieldName}} >= ?",
        'lteq'          => "{{fieldName}} <= ?",
        'finset'        => "FIND_IN_SET(?, {{fieldName}})",
        'regexp'        => "{{fieldName}} REGEXP ?",
        'from'          => "{{fieldName}} >= ?",
        'to'            => "{{fieldName}} <= ?",
        'seq'           => null,
        'sneq'          => null,
        'ntoa'          => "INET_NTOA({{fieldName}}) LIKE ?",
    ];

    $query = '';
    if (is_array($condition)) {
        $key = key(array_intersect_key($condition, $conditionKeyMap));

    ...
}
```

这种方法是在 SQL 查询构建过程中的某个时刻最终被调用的。期望 `$condition` 参数具有以下（部分列出）形式之一：

+   `array("from" => $fromValue, "to" => $toValue)`

+   `array("eq" => $equalValue)`

+   `array("neq" => $notEqualValue)`

+   `array("like" => $likeValue)`

+   `array("in" => array($inValues))`

+   `array("nin" => array($notInValues))`

+   `array("notnull" => $valueIsNotNull)`

+   `array("null" => $valueIsNull)`

+   `array("gt" => $greaterValue)`

+   `array("lt" => $lessValue)`

+   `array("gteq" => $greaterOrEqualValue)`

+   `array("lteq" => $lessOrEqualValue)`

+   `array("finset" => $valueInSet)`

+   `array("regexp" => $regularExpression)`

+   `array("seq" => $stringValue)`

+   `array("sneq" => $stringValue)`

如果`$condition`作为整数或字符串传递，则将过滤确切值（`'eq'`条件）。如果没有匹配任何条件，则期望参数为一个顺序数组，并将使用前面的结构构建`OR`条件。

上述示例涵盖了 EAV 模型集合，因为它们稍微复杂一些。尽管过滤的方法在简单模型集合中也大致适用，但最显著的区别是没有`addAttributeToFilter`、`addAttributeToSelect`和`addExpressionAttributeToSelect`方法。简单模型集合使用`addFieldToFilter`、`addFieldToSelect`和`addExpressionFieldToSelect`等方法，以及其他细微的区别。

# 摘要

在本章中，我们首先学习了如何创建简单的模型、其资源以及集合类。然后我们对 EAV 模型也进行了同样的操作。一旦我们有了所需的模型、资源和集合类，我们就详细地研究了模式和数据脚本的类型和流程。动手实践，我们涵盖了`InstallSchema`、`UpgradeSchema`、`InstallData`和`UpgradeData`脚本。一旦脚本运行，数据库最终拥有了所需的表和样本数据，这些数据是我们基于实体 CRUD 示例的。最后，我们快速但专注地查看集合管理，这主要涉及过滤集合以获取所需的结果集。

完整的模块代码可以从[`github.com/ajzele/B05032-Foggyline_Office`](https://github.com/ajzele/B05032-Foggyline_Office)下载。
