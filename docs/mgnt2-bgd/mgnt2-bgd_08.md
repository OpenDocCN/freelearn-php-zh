# 第八章：索引器

索引是通过对数据进行简化以减少数据库表数来转换数据的过程。这个过程用于产品、类别等，以提高网店性能。由于数据不断变化，这不仅仅是一个一次性过程，而是一个周期性的过程。`Magento_Indexer`模块是`Magento`索引功能的基础。

`Magento`控制台工具支持以下索引器命令。

```php
indexer
 indexer:info        Shows allowed Indexers
 indexer:reindex     Reindexes Data
 indexer:set-mode    Sets index mode type
 indexer:show-mode   Shows Index Mode
 indexer:status      Shows status of Indexer

```

当运行`php bin/magento indexer:info`时，你会得到所有 Magento 索引器的列表；默认的索引器如下：

```php
catalog_category_product    Category Products
catalog_product_category    Product Categories
catalog_product_price       Product Price
catalog_product_attribute   Product EAV
foggyline_office_employee   Employee Flat Data
cataloginventory_stock      Stock
catalogrule_rule            Catalog Rule Product
catalogrule_product         Catalog Product Rule
catalogsearch_fulltext      Catalog Search

```

你将在`系统` | `工具` | `索引管理`菜单中的`Magento`管理后台看到所有索引器。

在管理区域内部，我们只能更改索引器模式。索引器有两种模式：

+   **保存时更新**：索引表在字典数据更改后立即更新

+   **按计划更新**：索引表根据配置的计划由`cron`作业更新

由于无法从管理后台手动运行索引器，我们只能依赖它们的手动执行或`cron`执行。

手动执行是通过以下控制台命令完成的：

```php
php bin/magento indexer:reindex

```

前面的命令会一次性运行所有索引器。我们可以进一步微调，通过运行一个类似于以下代码的命令来执行单个索引器：

```php
php bin/magento indexer:reindex catalogsearch_fulltext

```

通过`Magento_Indexer`模块定义了 Cron 执行的索引器，如下所示：

+   `indexer_reindex_all_invalid`：这将每天每小时每分钟执行一次。它会在`Magento\Indexer\Model\Processor`类的一个实例上运行`reindexAllInvalid`方法。

+   `indexer_update_all_views`：这将每天每小时每分钟执行一次。它会在`Magento\Indexer\Model\Processor`类的一个实例上运行`updateMview`方法。

+   `indexer_clean_all_changelogs`：这将每天每小时的第 0 分钟执行一次。它会在`Magento\Indexer\Model\Processor`类的一个实例上运行`clearChangelog`方法。

这些`cron`作业使用操作系统`cron`作业设置，使得`Magento`的`cron`作业每分钟被触发一次。

以下三个状态是索引器可能具有的状态：

+   `valid`：数据已同步，无需重新索引

+   `invalid`：原始数据已更改，索引应更新

+   `working`：索引过程正在运行

尽管我们不会在本章中详细介绍创建自定义索引器的细节，但值得注意的是，`Magento` 在 `vendor/magento/module-*/etc/indexer.xml` 文件中定义了其索引器。这可能在我们需要深入了解单个索引器内部工作原理的情况下派上用场。例如，`catalog_product_flat` 索引器是通过 `Magento\Catalog\Model\Indexer\Product\Flat` 类实现的，该类在 `vendor/magento/module-catalog/etc/indexer.xml` 文件中定义。通过深入研究 `Flat` 类的实现，你可以了解数据是如何从 `EAV` 表中提取并扁平化成简化结构的。

# 摘要

在本章中，我们涵盖了 Magento 的许多最相关方面，这些方面超出了模型和类，涉及后端开发。我们查看了一下 `crontab.xml`，它帮助我们安排 `jobs`（命令），以便它们可以定期运行。然后，我们处理了通知消息，这使得我们可以通过浏览器向用户推送样式化的消息。*会话和 cookies* 部分让我们了解了 `Magento` 如何从浏览器跟踪用户信息到会话。日志和性能分析展示了跟踪性能和潜在问题的简单而有效的方法。*事件和观察者* 部分介绍了一种强大的模式，`Magento` 在代码中实现，我们可以触发自定义代码执行，当某个事件被触发时。缓存部分引导我们了解可用的缓存类型，并研究了如何创建和使用我们自己的缓存类型。通过前端应用（小工具）部分，我们学习了如何创建自己的小型应用，这些应用可以被调用到 CMS 页面和块中。自定义变量让我们了解了一个简单而有趣的功能，我们可以通过管理界面定义一个变量，然后在 CMS 页面、块或电子邮件模板中使用它。*国际化* 部分展示了如何使用 Magento 翻译功能在三个不同级别上翻译任何字符串，即模块 CSV 文件、主题 CSV 文件和内联翻译。最后，我们查看了一下索引器和它们的模式和状态；我们学习了如何控制它们的执行。

下一章将探讨前端开发。我们将学习如何创建自己的主题，并使用块和布局来影响输出。
