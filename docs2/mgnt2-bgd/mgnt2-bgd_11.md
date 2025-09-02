# 第十章。主要功能区域

Magento 平台包含各种模块，提供各种功能。开发者通常更熟悉某一组功能而不是其他功能。一些最常用的功能示例包括与 CMS 块和页面、分类、产品、客户、导入、自定义产品类型、自定义支付和运输模块相关的功能。这并不是说其他功能就不重要。在本章中，我们将快速查看 Magento 管理区域、PHP 代码和 API 调用中的功能。本章分为以下部分：

+   CMS 管理

+   目录管理

+   客户管理

+   产品和客户导入

+   自定义产品类型

+   自定义离线运输方法

+   自定义离线支付方法

目的不是深入了解每个功能区域，而是展示管理界面以及相应的程序性和 API 方法，以实现基本管理。

# CMS 管理

内容是帮助区分一个商店与另一个商店的因素。优质内容可以提高商店在搜索引擎中的可见性，为购买产品的客户提供信息洞察，并提供信誉和信任。Magento 提供了一个强大的内容管理系统，可用于为商店创建丰富内容。我们还可以用它来管理块和页面。

## 手动管理块

一个 CMS 块是内容的一个小型模块化单元，可以在页面的几乎任何位置定位。它们甚至可以被调用到另一个块中。块支持 HTML 和 JavaScript 作为其内容。因此，它们能够显示静态信息，如文本、图像和嵌入的视频，以及动态信息。

块可以通过管理界面、API 或代码创建。

以下步骤概述了从管理界面内创建块的过程：

1.  登录到 Magento 管理区域。

1.  在**内容** | **元素** | **块**菜单中，点击**添加新块**。这将打开一个类似于以下截图的屏幕：![手动管理块](img/00070.jpeg)

1.  填写所需字段的值（**块标题**、**标识符**、**商店视图**、**状态**和**内容**）并点击**保存块**按钮。

保存块后，您将在浏览器中看到**您已保存块**的成功消息。CMS 块存储在数据库中的`cms_block`和`cms_block_store`表中。

**标识符**值可能是这里最有趣的部分。我们可以在 CMS 页面、另一个 CMS 块或某些代码中使用它来获取我们刚刚创建的块。

假设我们已经创建了一个具有**标识符**值为`foggyline_hello`的块，我们可以通过以下表达式在 CMS 页面或另一个块中调用它：

```php
{{widget type="Magento\\Cms\\Block\\Widget\\Block" template="widget/static_block/default.phtml" block_id="foggyline_hello"}}
```

我们也可以将块的实际整数 ID 值传递给前面的表达式，如下所示：

```php
{{widget type="Magento\\Cms\\Block\\Widget\\Block" template="widget/static_block/default.phtml" block_id="2"}}
```

然而，这种方法要求我们知道块的实际整数 ID。

前面的表达式表明，块通过小部件（也称为前端应用）包含在页面或另一个块中。`Magento\Cms\Block\Widget\Block`类类型的小部件正在使用`widget/static_block/default.phtml`模板文件来渲染实际的 CMS 块。

## 通过代码管理块

除了通过管理界面手动创建块之外，我们还可以使用代码创建 CMS 块，如下面的代码片段所示：

```php
$model = $this->_objectManager->create('Magento\Cms\Model\Block');
$model->setTitle('Test block');
$model->setIdentifier('test_block');
$model->setContent('Test block!');
$model->setIsActive(true);
$model->save();
```

在这里，我们使用了实例管理器来创建`Magento\Cms\Model\Block`类的新模型实例。然后，我们通过定义的方法设置了一些属性，最后调用了`save`方法。

我们可以使用类似于以下代码的代码片段加载和更新现有块：

```php
$model = $this->_objectManager->create('Magento\Cms\Model\Block');
//$model->load(3);
$model->load('test_block');
$model->setTitle('Updated Test block');
$model->setStores([0]);
$model->save();
```

块的`load`方法接受一个整数块 ID 或一个字符串块标识符。

最后，我们可以通过可用的 API 方法来管理块创建和更新的操作。以下代码片段显示了如何通过控制台 cURL REST API 调用创建 CMS 块：

```php
curl -X POST "http://magento2.ce/index.php/rest/V1/cmsBlock" \
 -H "Content-Type:application/json" \
 -H "Authorization: Bearer lcpnsrk4t6al83lymhfs86jabbi9mmt8" \
 -d '{"block": {"identifier": "test_api_block", "title": "Test API Block", "content": "API Block Content"}}'

```

携带者字符串只是一个登录令牌，我们通过首先运行前面章节中描述的认证 API 调用来获取。一旦我们有了认证令牌，我们就可以发送一个`V1/cmsBlock` `POST`请求，传递一个 JSON 对象作为数据。

## 通过 API 管理块

我们可以通过执行类似于以下代码的代码片段通过 API 获取新创建的 CMS 块：

```php
curl -X GET "http://magento2.ce/index.php/rest/V1/cmsBlock/4" \
 -H "Content-Type:application/json" \
 -H "Authorization: Bearer lcpnsrk4t6al83lymhfs86jabbi9mmt8"

```

我们可以通过使用 API 并执行类似于以下代码的代码片段来更新现有的 CMS 块：

```php
curl -X PUT "http://magento2.ce/index.php/rest/V1/cmsBlock/4" \
 -H "Content-Type:application/json" \
 -H "Authorization: Bearer lcpnsrk4t6al83lymhfs86jabbi9mmt8" \
 -d '{"block": {"title": "Updated Test API Block"}}'

```

在这里，我们使用了 HTTP PUT 方法，并将整数`4`作为`V1/cmsBlock/4` URL 的一部分传递。数字 4 代表数据库中块的 ID 值。

## 手动管理页面

CMS 页面是健壮的内容单元，与简单地嵌入到某些页面中的 CMS 块不同。CMS 页面可以有自己的 URL。CMS 页面的例子包括**404 未找到**、**主页**、**启用 Cookies**和**隐私和 Cookies 政策**。在处理 CMS 页面时，我们的想法是我们可以控制页面内容区域，而不会影响网站范围的元素，如页眉、页脚或侧边栏。除了前面列出的之外，Magento 并没有提供很多开箱即用的 CMS 页面。

与块一样，页面也可以通过管理界面、API 或代码创建。

以下步骤概述了从管理界面内部创建页面的过程：

1.  登录到 Magento 管理区域。

1.  在**内容** | **元素** | **页面菜单**中，点击**添加新页面**。这将打开一个与以下截图类似的屏幕：![手动管理页面](img/00071.jpeg)

1.  填写所需字段的值（**页面标题**、**商店视图**、**状态**和**内容**）并点击**保存块**按钮。

页面保存后，你将在浏览器中看到**你已保存此页面**的成功消息。CMS 页面存储在数据库中的 `cms_page` 和 `cms_page_store` 表中。

假设我们已经创建了一个**页面标题**值为**信息**的页面，我们可以通过类似 `http://magento2.ce/info` 的 URL 在浏览器中访问此页面。尽管我们可能需要在**新页面**编辑屏幕中指定**URL 键**值，但 Magento 会自动分配与**页面标题**匹配的**URL 键**。

## 通过代码管理页面

除了通过管理界面手动创建之外，我们还可以通过以下代码片段创建 CMS 页面：

```php
$model = $this->_objectManager->create('Magento\Cms\Model\Page');
$model->setTitle('Test page');
$model->setIdentifier('test-page');
$model->setPageLayout('1column');
$model->setContent('Test page!');
$model->setIsActive(true);
$model->setStores([0]);
$model->save();
```

在这里，我们使用了实例管理器来创建 `Magento\Cms\Model\Page` 类的新模型实例。然后，我们通过定义的方法设置了一些属性，并最终调用了 `save` 方法。通过管理界面设置的**URL 键**实际上是通过 `setIdentifier` 方法调用设置的标识符。

## 通过 API 管理页面

我们可以使用类似于以下代码段的代码片段来加载和更新现有的页面：

```php
$model = $this->_objectManager->create('Magento\Cms\Model\Page');
//$model->load(6);
$model->load('test-page');
$model->setContent('Updated Test page!');
$model->save();
```

页面模型 `load` 方法接受页面标识符（**URL 键**）的整数 ID 值。

最后，我们可以通过可用的 API 方法来管理页面的创建和更新。以下代码片段显示了如何通过控制台 cURL REST API 调用来创建 CMS 页面：

```php
curl -X POST "http://magento2.ce/index.php/rest/V1/cmsPage" \
 -H "Content-Type:application/json" \
 -H "Authorization: Bearer lcpnsrk4t6al83lymhfs86jabbi9mmt8" \
 -d '{"page": {"identifier": "test-api-page", "title": "Test API Page", "content": "API Block Content"}}'

```

```php
 similar to the following one:
```

```php
curl -X GET "http://magento2.ce/index.php/rest/V1/cmsPage/7" \
 -H "Content-Type:application/json" \
 -H "Authorization: Bearer lcpnsrk4t6al83lymhfs86jabbi9mmt8"

```

我们可以通过执行类似于以下代码段的代码来通过 API 更新现有的 CMS 页面：

```php
curl -X PUT "http://magento2.ce/index.php/rest/V1/cmsPage/7" \
 -H "Content-Type:application/json" \
 -H "Authorization: Bearer lcpnsrk4t6al83lymhfs86jabbi9mmt8" \
 -d '{"page": {"content": "Updated Test API Page", "identifier":"updated-page"}}'

```

在这里，我们使用了 HTTP PUT 方法，将整数 `7` 作为 `V1/cmsPage/7` URL 的一部分传递。数字 7 代表数据库中页面的 ID 值。

# 目录管理

`Magento_Catalog` 模块是整个 Magento 平台的骨架之一。它为各种产品类型的库存管理提供强大的支持。该模块负责管理产品、类别及其属性、前端显示以及许多其他事情。

## 手动管理类别

我们可以通过导航到**产品** | **库存** | **目录**或**产品** | **库存** | **类别**来访问 Magento 管理区域内的目录功能。

如果我们从空白的 Magento 安装开始，我们可能会首先创建类别作为要创建的第一个实体之一。我们可以通过以下步骤手动创建类别：

1.  登录到 Magento 管理区域。

1.  前往**产品** | **库存** | **类别**菜单。这将打开一个类似于以下截图的屏幕：![手动管理类别](img/00072.jpeg)

1.  在屏幕的左侧点击**默认类别**。然后，当页面重新加载时，点击**添加子类别**按钮。

1.  虽然看起来好像没有发生任何变化，因为屏幕内容没有改变，但现在我们应该在**一般信息**选项卡中填写所需的选项，将**名称**设置为某个字符串值，并将**是否激活**设置为是。

1.  最后，点击**保存类别**按钮。

新类别现在应该已创建。在左侧屏幕区域，如果您点击新创建的类别的名称，您将在**一般信息**选项卡上方看到其 ID 值，如图所示：

![手动管理类别](img/00073.jpeg)

### 注意

知道类别 ID 后，您可以直接在浏览器中打开一个类似于`http://magento2.ce/index.php/catalog/category/view/id/3`的 URL 来测试它，其中数字`3`是类别的 ID。您将看到一个加载的类别页面，可能显示**找不到与选择匹配的产品**的消息，这是好的，因为我们还没有将产品分配给类别。

虽然我们不会深入探讨其细节，但值得注意的是，我们在这里只是触及了表面，因为类别使我们能够通过**显示设置**、**自定义设计**选项卡提供许多额外的选项。

由于类别是 EAV 实体，它们的数据存储在数据库的多个表中，如下所示：

+   `catalog_category_entity`

+   `catalog_category_entity_datetime`

+   `catalog_category_entity_decimal`

+   `catalog_category_entity_int`

+   `catalog_category_entity_text`

+   `catalog_category_entity_varchar`

有几个额外的表将类别链接到产品：

+   `catalog_category_product`

+   `catalog_category_product_index`

+   `catalog_category_product_index_tmp`

+   `catalog_url_rewrite_product_category`

## 通过代码管理类别

除了通过管理界面手动创建之外，我们还可以通过以下代码片段所示的方式通过代码创建类别：

```php
$parentId = \Magento\Catalog\Model\Category::TREE_ROOT_ID;

$parentCategory = $this->_objectManager
                       ->create('Magento\Catalog\Model\Category')
                       ->load($parentId);

$category = $this->_objectManager
                ->create('Magento\Catalog\Model\Category');

$category->setPath($parentCategory->getPath());
$category->setParentId($parentId);
$category->setName('Test');
$category->setIsActive(true);

$category->save();
```

这里特别的是，在创建新类别时，我们首先创建了一个`$parentCategory`实例，它代表根类别对象。我们使用`Category`模型的`TREE_ROOT_ID`常量作为父类别 ID 的 ID 值。然后，我们创建了一个类别实例，设置了其`path`、`parent_id`、`name`和`is_active`值。

## 通过 API 管理类别

我们可以通过可用的 API 方法进一步管理类别创建。以下代码片段显示了通过控制台 cURL REST API 调用创建类别：

```php
curl -X POST "http://magento2.ce/index.php/rest/V1/categories" \
 -H "Content-Type:application/json" \
 -H "Authorization: Bearer lcpnsrk4t6al83lymhfs86jabbi9mmt8" \
 -d '{"category": {"parent_id": "1", "name": "Test API Category", "is_active": true}}'

```

承载字符串只是一个登录令牌，我们通过首先运行上一章中描述的认证 API 调用来获取它。一旦我们有了认证令牌，我们就可以发出一个`/V1/categories POST`请求，传递一个 JSON 对象作为数据。

我们可以通过执行以下类似代码片段的代码片段，通过 API 获取新创建的类别作为一个 JSON 对象：

```php
curl -X GET "http://magento2.ce/index.php/rest/V1/categories/9" \
 -H "Content-Type:application/json" \
 -H "Authorization: Bearer lcpnsrk4t6al83lymhfs86jabbi9mmt8"

```

## 手动管理产品

现在，让我们看看如何创建一个新的产品。我们可以通过以下步骤手动创建产品：

1.  登录到 Magento 管理区域。

1.  在**产品** | **库存** | **目录**菜单中，点击**添加产品**按钮。这将打开一个类似于以下截图的屏幕：![手动管理产品](img/00074.jpeg)

1.  现在，在**产品详情**选项卡中填写所需的选项。

1.  最后，点击**保存**按钮。

如果保存成功，页面将重新加载并显示**您已保存产品**的消息。

与类别一样，我们在这里只是触及了产品的表面。查看其他可用的选项卡，有许多其他选项可以分配给产品。只需分配所需的选项就足以让我们在商店的前端 URL（如`http://magento2.ce/index.php/catalog/product/view/id/4`）上看到产品，其中数字`4`是产品的 ID 值。

产品也是 EAV 实体，其数据存储在数据库的多个表中，如下所示：

+   `catalog_product_entity`

+   `catalog_product_entity_datetime`

+   `catalog_product_entity_decimal`

+   `catalog_product_entity_gallery`

+   `catalog_product_entity_group_price`

+   `catalog_product_entity_int`

+   `catalog_product_entity_media_gallery`

+   `catalog_product_entity_media_gallery_value`

+   `catalog_product_entity_text`

+   `catalog_product_entity_tier_price`

+   `catalog_product_entity_varchar`

还有大量其他引用产品的表，例如`catalog_product_bundle_selection`，但这些主要用于链接功能片段。

## 通过代码管理产品

除了通过管理界面手动创建外，我们还可以通过代码创建产品，如下面的代码片段所示：

```php
$catalogConfig = $this->_objectManager
    ->create('Magento\Catalog\Model\Config');

$attributeSetId = $catalogConfig->getAttributeSetId(4, 'Default');

$product = $this->_objectManager
    ->create('Magento\Catalog\Model\Product');

$product
    ->setTypeId(\Magento\Catalog\Model\Product\Type::TYPE_SIMPLE)
    ->setAttributeSetId($attributeSetId)
    ->setWebsiteIds([$this->storeManager->getWebsite()->getId()])
    ->setStatus(\Magento\Catalog\Model\Product\Attribute \Source\Status::STATUS_ENABLED)
    ->setStockData(['is_in_stock' => 1, 'manage_stock' => 0])
    ->setStoreId(\Magento\Store\Model\Store::DEFAULT_STORE_ID)
    ->setVisibility(\Magento\Catalog\Model\Product \Visibility::VISIBILITY_BOTH);

$product
    ->setName('Test API')
    ->setSku('tets-api')
    ->setPrice(19.99);

$product->save();
```

## 通过 API 管理产品

以下示例使用 REST API 创建一个新的简单产品：

```php
curl -X POST "http://magento2.ce/index.php/rest/V1/products" \
 -H "Content-Type:application/json" \
 -H "Authorization: Bearer lcpnsrk4t6al83lymhfs86jabbi9mmt8" \
 -d '{"product":{"sku":"test_api_1","name":"Test API #1","attribute_set_id":4,"price":19.99,"status":1, "visibility":4,"type_id":"simple","weight":1}}'

```

应该通过使用身份验证请求预先获取`Bearer`令牌。响应应该是一个包含所有公开产品数据的 JSON 对象。

我们可以通过执行以下代码片段的 API 获取现有产品信息：

```php
curl -X GET "http://magento2.ce/index.php/rest/V1/products /product_dynamic_125" \
 -H "Content-Type:application/json"

```

在前面的 URL 中，`product_dynamic_125`部分代表这个特定的产品 SKU 值。响应是一个包含所有公开产品数据的 JSON 对象。

可用的整个目录 API 列表可以在`vendor/magento/module-catalog/etc/webapi.xml`文件中查看。

# 客户管理

管理客户是 Magento 平台的重要方面之一。大多数情况下，客户创建是由新客户自己完成的。访问商店的新客户启动注册过程，并最终创建客户账户。一旦注册，客户就可以在**我的账户**页面上进一步编辑他们的账户详情，该页面通常在类似`http://magento2.ce/index.php/customer/account/index/`的链接上可用。

作为本节的一部分，我们感兴趣的是通过使用管理区域、代码和 API 来管理客户账户的可能性。

## 手动管理客户

以下步骤概述了从管理界面内部创建客户账户的过程：

1.  登录到 Magento 管理区域。

1.  在**客户** | **所有客户**菜单中，点击**添加新客户**按钮。这将打开一个类似于以下截图的屏幕：![手动管理客户](img/00075.jpeg)

1.  填写所需字段的值（**关联到网站**、**组**、**名**、**姓**和**电子邮件**）并点击**保存客户**按钮。

一旦客户被保存，你将在浏览器中看到**您已保存客户**的成功消息。

对于此类情况，**关联到网站**值可能是最重要的值，在这种情况下，客户账户是由非客户用户间接创建的。

### 注意

由于 Magento 支持设置多个网站，客户账户可以根据**商店** | **设置** | **配置** | **客户** | **客户配置** | **账户共享选项** | **共享客户账户**选项设置为**全局**或**按网站**。因此，如果**共享客户账户**选项已设置为**按网站**，将**关联到网站**值指向正确的网站至关重要。否则，将创建客户账户，但客户将无法在店面登录。

`Magento_Customer`模块使用 EAV 结构来存储客户数据。因此，没有单个表存储客户信息。相反，根据客户属性及其数据类型存在多个表。

以下列表包含存储客户实体的表：

+   `customer_entity`

+   `customer_entity_datetime`

+   `customer_entity_decimal`

+   `customer_entity_int`

+   `customer_entity_text`

+   `customer_entity_varchar`

没有客户地址的客户账户将不会真正完整。地址可以通过在管理区域客户编辑屏幕下的**地址**选项卡添加，如下面的截图所示：

![手动管理客户](img/00076.jpeg)

注意，Magento 允许我们设置其中一个地址为**默认发货地址**和**默认账单地址**。

与客户实体类似，客户地址实体也使用 EAV 结构来存储其数据。

以下列表包含存储客户地址实体的表：

+   `customer_address_entity`

+   `customer_address_entity_datetime`

+   `customer_address_entity_decimal`

+   `customer_address_entity_int`

+   `customer_address_entity_text`

+   `customer_address_entity_varchar`

## 通过代码管理客户

除了通过管理界面手动创建之外，我们还可以通过以下代码片段创建客户：

```php
$model = $this->_objectManager-> create('Magento\Customer\Model\Customer');
$model->setWebsiteId(1);
$model->setGroupId(1);
$model->setFirstname('John');
$model->setLastname('Doe');
$model->setEmail('john.doe@mail.com');
$model->save();
```

在这里，我们使用实例管理器来创建`Magento\Customer\Model\Customer`类的新模型实例。然后我们可以通过定义的方法设置一些属性，并最终调用`save`方法。

我们可以使用类似于以下代码片段的代码片段来加载和更新现有客户：

```php
$model = $this->_objectManager-> create('Magento\Customer\Model\Customer');
$model->setWebsiteId(1);
//$model->loadByEmail('john.doe@mail.com');
$model->load(1);
$model->setFirstname('Updated John');
$model->save();
```

我们可以使用`load`或`loadByEmail`方法调用。`load`方法接受现有客户实体的整数 ID 值，而`loadByEmail`接受一个字符串电子邮件地址。值得注意的是，必须在任何加载方法之前调用`setWebsiteId`。否则，我们将收到一个错误消息，指出**在使用网站范围时必须指定客户网站 ID**。

## 通过 API 管理客户

最后，我们可以使用可用的 API 方法来管理客户信息的创建和更新。以下代码片段显示了如何通过控制台 cURL REST API 调用创建客户：

```php
curl -X POST "http://magento2.ce/index.php/rest/V1/customers" \
 -H "Content-Type:application/json" \
     -H "Authorization: Bearer r9ok12c3wsusrxqomyxiwo0v7etujw9h" \
 -d '{"customer": {"website_id": 1, "group_id": 1, "firstname": "John", "lastname": "Doe", "email": "john.doe@mail.com"}, "password":"abc123"}'

```

一旦我们有了认证令牌，我们就可以发送一个`V1/customers` `POST`请求，传递一个 JSON 对象作为数据。

我们可以通过执行一个类似于以下代码片段的代码，通过 API 获取新创建的客户：

```php
curl -X GET "http://magento2.ce/index.php/rest/V1/customers/24" \
 -H "Content-Type:application/json" \
 -H "Authorization: Bearer lcpnsrk4t6al83lymhfs86jabbi9mmt8"

```

我们可以通过执行类似于以下代码片段的代码片段通过 API 更新现有客户：

```php
curl -X PUT "http://magento2.ce/index.php/rest/V1/customers/24" \
 -H "Content-Type:application/json" \
 -H "Authorization: Bearer r9ok12c3wsusrxqomyxiwo0v7etujw9h" \
 -d '{"customer": {"id":24, "website_id": 1, "firstname": "John Updated", "lastname": "Doe", "email": "john2@mail.com"}, "password_hash":"cda57c7995e5f03fe07ad52d99686ba130e0d3e fe0d84dd5ee9fe7f6ea632650:cEf8i1f1ZXT1L2NwawTRNEqDWGyru6h3:1"}'

```

在这里，我们使用了 HTTP PUT 方法，将整数`24`作为`V1/customers/24`的一部分以及作为 URL 的主体部分。数字 24 代表数据库中客户的 ID 值。此外，请注意`password_hash`值；如果没有它，更新将失败。

## 通过代码管理客户地址

与客户类似，我们可以使用代码创建客户地址，如下面的代码片段所示：

```php
$model = $this->_objectManager-> create('Magento\Customer\Model\Address');
//$model->setCustomer($customer);
$model->setCustomerId(24);
$model->setFirstname('John');
$model->setLastname('Doe');
$model->setCompany('Foggyline');
$model->setStreet('Test street');
$model->setCity('London');
$model->setCountryId('GB');
$model->setPostcode('GU22 7PY');
$model->setTelephone('112233445566');
$model->setIsDefaultBilling(true);
$model->setIsDefaultShipping(true);
$model->save();
```

在这里，我们使用了实例管理器创建`Magento\Customer\Model\Address`类的新模型实例。然后，我们通过定义的方法设置一些属性，并最终调用`save`方法。

我们可以通过类似于以下代码片段的代码片段加载和更新现有的客户地址：

```php
$model = $this->_objectManager-> create('Magento\Customer\Model\Address');
$model->load(22);
$model->setCity('Update London');
$model->save();
```

在这里，我们使用了`load`方法通过其 ID 值加载现有的地址。然后，我们调用`setCity`方法并传递更新的字符串。在执行`save`方法后，地址应该反映这一变化。

## 通过 API 管理客户地址

令人惊讶的是，客户地址不能通过 API 调用直接创建或更新，因为没有定义`POST`或`PUT REST` API。然而，我们仍然可以通过以下方式使用 API 获取现有的客户地址信息：

```php
curl -X GET "http://magento2.ce/index.php/rest/V1/customers /addresses/22" \
 -H "Content-Type:application/json" \
 -H "Authorization: Bearer lcpnsrk4t6al83lymhfs86jabbi9mmt8"

```

可用的所有客户 API 列表可以在`vendor/magento/module-customer/etc/webapi.xml`文件中看到。

# 产品和客户导入

Magento 通过以下模块提供开箱即用的批量导入和导出功能：

+   `AdvancedPricingImportExport`

+   `BundleImportExport`

+   `CatalogImportExport`

+   `ConfigurableImportExport`

+   `CustomerImportExport`

+   `GroupedImportExport`

+   `ImportExport`

+   `TaxImportExport`

导入功能的核心实际上在于`ImportExport`模块，而其他模块通过`vendor/magento/module-{partialModuleName}-import-export/etc/import.xml`和`vendor/magento/module-{partialModuleName}-import-export/etc/export.xml`文件提供单独的导入和导出实体。

这些功能可以从 Magento 管理区域中的**系统** | **数据传输**菜单访问。它们使我们能够导出和导入多个实体类型，例如**高级定价**、**产品**、**主客户文件**和**客户地址**。

以下截图显示了**导入设置**屏幕的**实体类型**选项：

![产品和客户导入](img/00077.jpeg)

在**导入设置**旁边，当我们为导入选择**实体类型**时，会出现**导入行为**部分，如下面的截图所示：

![产品和客户导入](img/00078.jpeg)

大多数实体类型都有类似的**导入行为**选项。大多数时候，我们将对**添加/更新**行为感兴趣。

由于导入过程比导出过程复杂一些，我们将重点关注导入和 CSV 文件格式。更具体地说，我们的重点是**产品**、**主客户文件**和**客户地址**的导入。

当与干净的 Magento 安装一起工作时，以下列在产品导入期间是必需的，以便在之后使产品在店面可见：

+   `sku`（例如，“test-sku”）：这可以具有几乎任何值，只要它在 Magento 中是唯一的。

+   `attribute_set_code`（例如，“默认”）：这可以具有在执行`SELECT DISTINCT attribute_set_name FROM eav_attribute_set;`查询时在数据库中找到的任何值。

+   `product_type`（例如，“简单”）：这可以具有`simple`、`configurable`、`grouped`、`virtual`、`bundle`或`downloadable`的值。此外，如果我们创建或安装了一个添加新产品类型的第三方模块，我们也可以使用该模块。

+   `categories`（例如，“根/鞋子”）：使用“根类别名称/子类别名称/子子类别名称”语法创建完整的类别路径。如果有多个类别，则使用竖线（“|”）分隔它们。例如，“根类别名称/子类别名称/子子类别名称| 根类别名称/子 _2 类别名称”。

+   `product_websites`（例如，“基础”）：这可以具有在执行`SELECT DISTINCT code FROM store_website;`查询时在数据库中找到的值。

+   `name`（例如，“测试”）：这可以具有几乎任何值。

+   `product_online`（例如，`1`）：这可以是`1`表示“可见”或`0`表示“不可见”。

+   `visibility`（例如，“目录，搜索”）：这可以具有“单独不可见”、“目录”、“搜索”或“目录，搜索”的值。

+   `price`（例如，“9.99”）：这可以是一个整数或小数值。

+   `qty`（例如，“100”）：这可以是一个整数或小数值。

虽然产品将仅通过包含一组列的前列列表导入，但我们通常希望为它们分配额外的信息，例如描述和图片。我们可以通过以下列来实现：

+   `description`（例如，“描述”）：这可以包含任何字符串值。支持 HTML 和 JavaScript。

+   `short_description`（例如，“简短描述”）：这可以包含任何字符串值。支持 HTML 和 JavaScript。

+   `base_image`（例如，`butterfly.jpg`）：这是最终的导入图像名称。

+   `small_image`（例如，`galaxy.jpg`）

+   `thumbnail_image`（例如，`serenity.jpg`）

关于图像的导入，只要在导入期间设置了**图像文件目录**路径，我们只需要提供最终的图像名称。我们可以使用相对路径，例如 Magento 安装的`var/export`、`var/import`、`var/export/some/dir`。

导入完成后，建议通过控制台运行`php bin/magento indexer:reindex`命令。否则，直到运行索引器之前，产品在店面上将不可见。

重新索引完成后，我们可以尝试打开店面 URL，它看起来像`http://magento2.ce/index.php/catalog/product/view/id/1`。在这种情况下，数字`1`是新导入的产品 ID。

当与干净的 Magento 安装一起工作时，在客户主要文件导入期间需要以下列出的列，以便我们的客户之后能够成功登录到店面：

+   `email`（例如，`<john.doe@fake.mail>`）：作为字符串值的电子邮件地址

+   `_website`（例如，基础）：这可以包含在执行`SELECT DISTINCT code FROM store_website;`查询时在数据库中找到的任何值

+   `firstname`（例如，约翰）：一个字符串值

+   `lastname`（例如，多）：一个字符串值

+   `group_id`（例如，1）：这可以包含在执行`SELECT customer_group_id code FROM customer_group WHERE customer_group_id != 0;`查询时在数据库中找到的任何值

尽管客户只需使用之前列出的列集就能登录到店面，但我们通常希望分配其他相关的信息。我们可以通过以下列来实现：

+   `gender`（例如，男性）：这可以是男性或女性

+   `taxvat`（例如，HR33311122299）：任何有效的增值税号，尽管导入将接受无效的增值税号

+   `dob`（例如，1983-01-16）：出生日期

+   `prefix`（例如，先生）：任何字符串值

+   `middlename`（例如，开发人员）：任何字符串值

+   `suffix`（例如，工程师）：任何字符串值

+   `password`（例如，123abc）：任何长度至少为 6 个字符的字符串值，如通过`\Magento\CustomerImportExport\Model\Import\Customer::MIN_PASSWORD_LENGTH`定义。

我们需要特别注意`password`列。这是一个明文密码。因此，我们需要小心不要以非安全的方式分发 CSV 文件。理想情况下，我们可以提供`password_hash`列而不是`password`。然而，`password_hash`列下的条目需要通过`Magento\Customer\Model\Customer`类中的`hashPassword`方法调用的相同算法进行哈希处理。这进一步在`Magento\Framework\Encryption\Encryptor`类的一个实例上调用`getHash`方法，最终解析为`md5`或`sha256`算法。

当与干净的 Magento 安装一起工作时，在客户地址导入期间需要以下列，以便我们的客户之后能够成功地在店面使用地址：

+   `_website` (例如，base)：这可以具有在执行`SELECT DISTINCT code FROM store_website;`查询时在数据库中找到的任何值

+   `_email` (例如，`<john@change.me>`)：作为字符串值的电子邮件地址

+   `_entity_id`

+   `firstname` (例如，约翰)：任何字符串值

+   `lastname` (例如，多伊)：任何字符串值

+   `street` (例如，阿什顿巷)：任何字符串值

+   `city` (例如，奥斯汀)：任何字符串值

+   `telephone` (例如，00 385 91 111 000)：任何字符串值

+   `country_id` (例如，GB)：ISO-2 格式的国家代码

+   `postcode` (例如，TX 78753)：任何字符串值

尽管客户只需列出的一组列就可以在店面使用地址，但我们通常希望分配其他相关的信息。我们可以通过以下列来实现：

+   `region` (例如，加利福尼亚)：这可以是空白、自由形式的字符串，或者与在执行`SELECT DISTINCT default_name FROM directory_country_region;`查询时在数据库中找到的任何值匹配的特定字符串。当运行`SELECT DISTINCT country_id FROM directory_country_region;`时，显示 13 个不同的国家代码，这些代码在`directory_country_region`表中都有条目——`AT`、`BR`、`CA`、`CH`、`DE`、`EE`、`ES`、`FI`、`FR`、`LT`、`LV`、`RO`、`US`。这意味着具有该代码的国家需要分配一个适当的地区名称。

+   `company` (例如，雾线)：这可以是任何字符串值。

+   `fax` (例如，00 385 91 111 000)：这可以是任何字符串值。

+   `middlename` (例如，开发者)：这可以是任何字符串值。

+   `prefix` (例如，先生)：这可以是任何字符串值。

+   `suffix` (例如，工程师)：这可以是任何字符串值。

+   `vat_id` (例如，HR33311122299)：这可以是任何有效的增值税号，尽管导入将接受甚至非有效的那些。

+   `_address_default_billing_` (例如，"1")：这可以是"1"作为是或"0"作为否，以标记地址为默认账单地址。

+   `_address_default_shipping_`（例如，“1”）：这可以是“1”表示是，或者“0”表示否，以标记该地址为默认的配送地址。

虽然 CSV 导入是一种非常好且相对快速的大规模导入产品、客户及其地址的方法，但它也有一些局限性。CSV 只是平面数据。我们无法对其应用任何逻辑。根据数据有多干净和有效，CSV 导入可能做得很好。否则，我们可能需要选择 API。我们需要记住，CSV 导入比产品和服务器的 API 创建要快得多，因为 CSV 导入直接在数据库上批量插入，而 API 则实例化完整的模型，尊重事件观察者等。

# 定制产品类型

Magento 提供以下六种开箱即用的产品类型：

+   简单产品

+   可配置产品

+   组合产品

+   虚拟产品

+   组合产品

+   可下载的产品

每个产品都有其特定属性。例如，虚拟和可下载的产品没有 `weight` 属性。因此，它们被排除在标准配送计算之外。通过围绕内置产品类型进行自定义编码，使用观察器和插件，我们可以实现几乎任何功能。然而，有时这还不够，或者没有解决方案来满足需求。在这种情况下，我们可能需要创建自己的产品类型，以便以更简洁的方式满足项目需求。

让我们创建一个名为 `Foggyline_DailyDeal` 的小型模块，它将为 Magento 添加一个新的产品类型。

首先，创建一个名为 `app/code/Foggyline/DailyDeal/registration.php` 的模块注册文件，其中包含以下部分内容：

```php
\Magento\Framework\Component\ComponentRegistrar::register(
    \Magento\Framework\Component\ComponentRegistrar::MODULE,
    'Foggyline_DailyDeal',
    __DIR__
);
```

然后，创建一个包含以下内容的 `app/code/Foggyline/DailyDeal/etc/module.xml` 文件：

```php
<config  xsi:noNamespaceSchemaLocation="urn:magento:framework:Module /etc/module.xsd">
    <module name="Foggyline_DailyDeal" setup_version="1.0.0">
        <sequence>
            <module name="Magento_Catalog"/>
        </sequence>
    </module>
</config>
```

现在，创建一个包含以下内容的 `app/code/Foggyline/DailyDeal/etc/product_types.xml` 文件：

```php
<config  xsi:noNamespaceSchemaLocation="urn:magento:module: Magento_Catalog:etc/product_types.xsd">
    <type name="foggylinedailydeal"
          label="Daily Deal"
          modelInstance="Foggyline\DailyDeal\Model\Product\Type \DailyDeal"
          composite="false"
          isQty="true"
          canUseQtyDecimals="false">
        <priceModel instance="Foggyline\DailyDeal\Model \Product\Price"/>
        <indexerModel instance="Foggyline\DailyDeal\Model \ResourceModel\Indexer\Price"/>
        <stockIndexerModel instance="Foggyline\DailyDeal\Model \ResourceModel\Indexer\Stock"/>
        <!-- customAttributes parsed by Magento\Catalog\Model\ProductTypes\Config -->
        <customAttributes>
            <attribute name="is_real_product" value="true"/>
            <attribute name="refundable" value="false"/>
            <attribute name="taxable" value="true"/>
        </customAttributes>
    </type>
</config>
```

`customAttributes` 元素由 `vendor/magento/module-catalog/Model/ProductTypes/Config.php` 解析。

创建一个包含部分内容的 `app/code/Foggyline/DailyDeal/Model/Product/Type/DailyDeal.php` 文件，如下所示：

```php
namespace Foggyline\DailyDeal\Model\Product\Type;

class DailyDeal extends \Magento\Catalog\Model\Product\Type\AbstractType
{
    const TYPE_DAILY_DEAL = 'foggylinedailydeal';

    public function deleteTypeSpecificData (\Magento\Catalog\Model\Product $product)
    {
        // TODO: Implement deleteTypeSpecificData() method.
    }
}
```

现在，创建一个包含部分内容的 `app/code/Foggyline/DailyDeal/Model/Product/Price.php` 文件，如下所示：

```php
namespace Foggyline\DailyDeal\Model\Product;

class Price extends \Magento\Catalog\Model\Product\Type\Price
{

}
```

完成此操作后，创建一个包含部分内容的 `app/code/Foggyline/DailyDeal/Model/ResourceModel/Indexer/Price.php` 文件，如下所示：

```php
namespace Foggyline\DailyDeal\Model\ResourceModel\Indexer;

class Price extends \Magento\Catalog\Model\ResourceModel\Product \Indexer\Price\DefaultPrice
{
}
```

然后，创建一个包含部分内容的 `app/code/Foggyline/DailyDeal/Model/ResourceModel/Indexer/Stock.php` 文件，如下所示：

```php
namespace Foggyline\DailyDeal\Model\ResourceModel\Indexer;

class Stock extends \Magento\CatalogInventory\Model\ResourceModel \Indexer\Stock\DefaultStock
{

}
```

最后，创建一个包含部分内容的 `app/code/Foggyline/DailyDeal/Setup/InstallData.php` 文件，如下所示：

```php
namespace Foggyline\DailyDeal\Setup;

class InstallData implements \Magento\Framework\Setup\InstallDataInterface
{
    private $eavSetupFactory;

    public function __construct(\Magento\Eav\Setup\EavSetupFactory $eavSetupFactory)
    {
        $this->eavSetupFactory = $eavSetupFactory;
    }

    public function install(
        \Magento\Framework\Setup\ModuleDataSetupInterface $setup,
        \Magento\Framework\Setup\ModuleContextInterface $context
    )
    {
        // the "foggylinedailydeal" type specifics
    }
}
```

通过在 `InstallData` 类中添加以下 `foggylinedailydeal` 类型特定内容来扩展 `install` 方法：

```php
$eavSetup = $this->eavSetupFactory->create(['setup' => $setup]);
$type = \Foggyline\DailyDeal\Model\Product\Type\ DailyDeal::TYPE_DAILY_DEAL;

$fieldList = [
    'price',
    'special_price',
    'special_from_date',
    'special_to_date',
    'minimal_price',
    'cost',
    'tier_price',
    'weight',
];

// make these attributes applicable to foggylinedailydeal products
foreach ($fieldList as $field) {
    $applyTo = explode(
        ',',
        $eavSetup->getAttribute (\Magento\Catalog\Model\Product::ENTITY, $field, 'apply_to')
    );

    if (!in_array($type, $applyTo)) {
        $applyTo[] = $type;
        $eavSetup->updateAttribute(
            \Magento\Catalog\Model\Product::ENTITY,
            $field,
            'apply_to',
            implode(',', $applyTo)
        );
    }
}
```

现在，从控制台运行 `php bin/magento setup:upgrade`。

如果你现在在管理区域打开**产品** | **库存** | **目录**菜单，并单击**添加产品**按钮旁边的下拉图标，你将在列表中看到**Daily Deal**产品类型，如下面的截图所示：

![自定义产品类型](img/00079.jpeg)

在下拉列表中单击**Daily Deal**产品类型应该会打开产品编辑页面，如下面的截图所示：

![自定义产品类型](img/00080.jpeg)

自定义产品类型编辑屏幕与内置产品类型之一之间没有明显的区别。

假设我们已经将产品命名为`Daily Deal Test Product`并保存，我们应该能够在店面中看到它，如下面的截图所示：

![自定义产品类型](img/00081.jpeg)

如果我们将产品添加到购物车并执行结账，应该会创建一个订单，就像任何其他产品类型一样。在管理区域中，在订单查看页面上，在**已订购项目**下，我们应该能够在列表中看到该产品，如下面的截图所示：

![自定义产品类型](img/00082.jpeg)

再次，自定义产品类型与在**已订购项目**部分下渲染的内置产品类型之间没有明显的区别。

最后，我们应该在控制台上运行`php bin/magento indexer:reindex`命令。即使我们实际上没有在索引器中实现任何代码，这也只是为了确保现有的索引器没有损坏。

整个模块代码可以从[`github.com/ajzele/B05032-Foggyline_DailyDeal`](https://github.com/ajzele/B05032-Foggyline_DailyDeal)下载。

# 自定义离线配送方法

Magento 提供了几种开箱即用的离线配送方式，例如`Flatrate`、`Freeshipping`、`Pickup`和`Tablerate`。我们可以在`vendor/magento/module-offline-shipping/Model/Carrier`目录中看到这些。

然而，项目需求通常是这样的，我们需要一个自定义编码的配送方法，其中应用了特殊业务逻辑。因此，我们可以控制配送价格的计算。在这种情况下，了解如何编写我们自己的离线配送方法可能会很有用。

让我们继续创建一个名为`Foggyline_Shipbox`的小模块，为 Magento 提供额外的离线配送方法。

首先，创建一个名为`app/code/Foggyline/Shipbox/registration.php`的模块注册文件，内容如下所示：

```php
\Magento\Framework\Component\ComponentRegistrar::register(
    \Magento\Framework\Component\ComponentRegistrar::MODULE,
    'Foggyline_Shipbox',
    __DIR__
);
```

然后，创建一个包含以下内容的`app/code/Foggyline/Shipbox/etc/module.xml`文件：

```php
<config  xsi:noNamespaceSchemaLocation="urn:magento:framework:Module /etc/module.xsd">
    <module name="Foggyline_Shipbox" setup_version="1.0.0">
        <sequence>
            <module name="Magento_OfflineShipping"/>
        </sequence>
    </module>
</config>
```

现在，创建一个包含以下内容的`app/code/Foggyline/Shipbox/etc/config.xml`文件：

```php
<config  xsi:noNamespaceSchemaLocation="urn:magento:module:Magento_Store :etc/config.xsd">
    <default>
        <carriers>
            <shipbox>
                <active>0</active>
                <sallowspecific>0</sallowspecific>
                <model> Foggyline\Shipbox\Model\Carrier\Shipbox</model>
                <name>Shipbox</name>
                <price>4.99</price>
                <title>Foggyline Shipbox</title>
                <specificerrmsg>This shipping method is not available. To use this shipping method, please contact us.</specificerrmsg>
            </shipbox>
        </carriers>
    </default>
</config>
```

完成此操作后，创建一个包含以下内容的`app/code/Foggyline/Shipbox/etc/adminhtml/system.xml`文件，如下所示：

```php
<config  xsi:noNamespaceSchemaLocation="urn:magento:module: Magento_Config:etc/system_file.xsd">
    <system>
        <section id="carriers">
            <group id="shipbox" translate="label" type="text" sortOrder="99" showInDefault="1" showInWebsite="1" showInStore="1">
                <label>Foggyline Shipbox</label>
                <field id="active" translate="label" type="select" sortOrder="1" showInDefault="1" showInWebsite="1" showInStore="0">
                    <label>Enabled</label>
                    <source_model> Magento\Config\Model\Config\Source\Yesno </source_model>
                </field>
                <field id="name" translate="label" type="text" sortOrder="3" showInDefault="1" showInWebsite="1" showInStore="1">
                    <label>Method Name</label>
                </field>
                <field id="price" translate="label" type="text" sortOrder="5" showInDefault="1" showInWebsite="1" showInStore="0">
                    <label>Price</label>
                    <validate>validate-number validate-zero-or-greater</validate>
                </field>
                <field id="title" translate="label" type="text" sortOrder="2" showInDefault="1" showInWebsite="1" showInStore="1">
                    <label>Title</label>
                </field>
                <field id="sallowspecific" translate="label" type="select" sortOrder="90" showInDefault="1" showInWebsite="1" showInStore="0">
                    <label>Ship to Applicable Countries</label>
                    <frontend_class>shipping-applicable-country </frontend_class>
                    <source_model> Magento\Shipping\Model\Config\Source \Allspecificcountries </source_model>
                </field>
                <field id="specificcountry" translate="label" type="multiselect" sortOrder="91" showInDefault="1" showInWebsite="1" showInStore="0">
                    <label>Ship to Specific Countries</label>
                    <source_model> Magento\Directory\Model \Config\Source\Country </source_model>
                    <can_be_empty>1</can_be_empty>
                </field>
            </group>
        </section>
    </system>
</config>
```

现在，创建一个包含以下内容的`app/code/Foggyline/Shipbox/Model/Carrier/Shipbox.php`文件：

```php
namespace Foggyline\Shipbox\Model\Carrier;

use Magento\Quote\Model\Quote\Address\RateRequest;

class Shipbox extends \Magento\Shipping\Model\Carrier\AbstractCarrier
    implements \Magento\Shipping\Model\Carrier\CarrierInterface
{
    protected $_code = 'shipbox';
    protected $_isFixed = true;
    protected $_rateResultFactory;
    protected $_rateMethodFactory;

    public function __construct(
        \Magento\Framework\App\Config\ScopeConfigInterface $scopeConfig,
        \Magento\Quote\Model\Quote\Address\RateResult\ErrorFactory $rateErrorFactory,
        \Psr\Log\LoggerInterface $logger,
        \Magento\Shipping\Model\Rate\ResultFactory $rateResultFactory,
        \Magento\Quote\Model\Quote\Address\RateResult \MethodFactory $rateMethodFactory,
        array $data = []
    )
    {
        $this->_rateResultFactory = $rateResultFactory;
        $this->_rateMethodFactory = $rateMethodFactory;
        parent::__construct($scopeConfig, $rateErrorFactory, $logger, $data);
    }

    public function collectRates(RateRequest $request)
    {
        //implement business logic
    }

    public function getAllowedMethods()
    {
        return ['shipbox' => $this->getConfigData('name')];
    }
}
```

在`Carrier\Shipbox`类中扩展`collectRates`方法，如下所示：

```php
public function collectRates(RateRequest $request)
{
    if (!$this->getConfigFlag('active')) {
        return false;
    }

    //Do some filtering of items in cart
    if ($request->getAllItems()) {
        foreach ($request->getAllItems() as $item) {
            //$item->getQty();
            //$item->getFreeShipping()
            //$item->isShipSeparately()
            //$item->getHasChildren()
            //$item->getProduct()->isVirtual()
            //...
        }
    }

    //After filtering, start forming final price
    //Final price does not have to be fixed like below
    $shippingPrice = $this->getConfigData('price');
    $result = $this->_rateResultFactory->create();

    $method = $this->_rateMethodFactory->create();

    $method->setCarrier('shipbox');
    $method->setCarrierTitle($this->getConfigData('title'));

    $method->setMethod('shipbox');
    $method->setMethodTitle($this->getConfigData('name'));

    $method->setPrice($shippingPrice);
    $method->setCost($shippingPrice);

    $result->append($method);

    return $result;
}
```

在 Magento 管理区域，如果你现在查看**商店** | **设置** | **配置** | **销售** | **运费方法**，你将在列表中看到**Foggyline Shipbox**，如下截图所示：

![自定义离线运费方法](img/00083.jpeg)

将**启用**选项设置为**是**，然后点击**保存配置**按钮。

如果你现在在 MySQL 服务器上运行`SELECT * FROM core_config_data WHERE path LIKE "%shipbox%";`查询，你将看到如下截图所示的类似结果：

![自定义离线运费方法](img/00084.jpeg)

注意，在前面的截图中的代码片段中没有直接与**适用于的国家**和**特定国家**选项相关的代码，因为这些选项的处理已经内置在父`AbstractCarrier`类中。因此，只需在`config.xml`和`system.xml`中添加`sallowspecific`选项，我们就可以启用一个功能，即运费方法可以显示或隐藏在某些国家。

实现的核心在于`collectRates`方法。这是我们实现自己的业务逻辑的地方，该逻辑应该根据购物车中的商品计算运费。我们可以在`collectRates`方法中使用`$request->getAllItems()`来获取所有购物车商品的集合，遍历它们，根据各种条件形成最终的运费，等等。

现在，让我们继续跳转到店面以测试结账流程。我们应该能在结账页面上看到我们的方法，如下截图所示：

![自定义离线运费方法](img/00085.jpeg)

如果我们完成一个订单，我们还应该在订单本身上看到运费方法详情。在管理区域中，在**销售** | **操作** | **订单**下，如果我们查看**支付与运费方法**部分的**查看**订单，我们应该能看到运费方法，如下截图所示：

![自定义离线运费方法](img/00086.jpeg)

类似地，在**订单总额**部分，我们应该看到**运费及处理费**中的运费金额，如下截图所示：

![自定义离线运费方法](img/00087.jpeg)

通过这种方式，我们完成了自定义离线运费方法模块。完整的模块可以在[`github.com/ajzele/B05032-Foggyline_Shipbox`](https://github.com/ajzele/B05032-Foggyline_Shipbox)找到。

# 自定义离线支付方法

Magento 提供了一些现成的离线支付方法，例如`Banktransfer`、`Cashondelivery`、`Checkmo`和`Purchaseorder`。你可以在`vendor/magento/module-offline-payments/Model`目录中看到它们。

当涉及到支付方式时，更常见的是使用在线支付服务提供商（网关），例如 PayPal 或 Braintree。有时，项目需求可能要求我们可能需要一个自定义编码的支付方式。你需要考虑程序化产品导入和订单创建脚本，这些脚本可能专门针对某些特别标记的支付方式。因此，支付过程将由我们控制。

在这种情况下，了解如何编写我们自己的离线支付方法可能很有用。值得注意的是，虽然我们可以创建一个离线支付，它会抓取用户的信用卡信息，但除非我们的基础设施符合 PCI 标准，否则这样做并不真正建议。

让我们继续创建一个名为 `Foggyline_Paybox` 的小模块，为 Magento 提供额外的离线支付方式。

首先，创建一个名为 `app/code/Foggyline/Paybox/registration.php` 的模块注册文件，部分内容如下：

```php
\Magento\Framework\Component\ComponentRegistrar::register(
    \Magento\Framework\Component\ComponentRegistrar::MODULE,
    'Foggyline_Paybox',
    __DIR__
);
```

然后，创建一个名为 `app/code/Foggyline/Paybox/etc/module.xml` 的文件，内容如下：

```php
<config  xsi:noNamespaceSchemaLocation="urn:magento:framework:Module /etc/module.xsd">
    <module name="Foggyline_Paybox" setup_version="1.0.0">
        <sequence>
            <module name="Magento_OfflinePayments"/>
        </sequence>
    </module>
</config>
```

完成此操作后，创建一个名为 `app/code/Foggyline/Paybox/etc/config.xml` 的文件，内容如下：

```php
<config  xsi:noNamespaceSchemaLocation="urn:magento:module: Magento_Store:etc/config.xsd">
    <default>
        <payment>
            <paybox>
                <active>0</active>
                <model>Foggyline\Paybox\Model\Paybox</model>
                <order_status>pending</order_status>
                <title>Foggyline Paybox</title>
                <allowspecific>0</allowspecific>
                <group>offline</group>
            </paybox>
        </payment>
    </default>
</config>
```

然后，创建一个名为 `app/code/Foggyline/Paybox/etc/payment.xml` 的文件，内容如下：

```php
<payment  xsi:noNamespaceSchemaLocation="urn:magento:module: Magento_Payment:etc/payment.xsd">
    <methods>
        <method name="paybox">
            <allow_multiple_address>1</allow_multiple_address>
        </method>
    </methods>
</payment>
```

现在，创建一个名为 `app/code/Foggyline/Paybox/etc/adminhtml/system.xml` 的文件，内容如下：

```php
<config  xsi:noNamespaceSchemaLocation="urn:magento:module: Magento_Config:etc/system_file.xsd">
    <system>
        <section id="payment">
            <group id="paybox" translate="label" type="text" sortOrder="30" showInDefault="1" showInWebsite="1" showInStore="1">
                <label>Paybox</label>
                <field id="active" translate="label" type="select" sortOrder="1" showInDefault="1" showInWebsite="1" showInStore="0">
                    <label>Enabled</label>
                    <source_model> Magento\Config\Model\Config\Source\Yesno </source_model>
                </field>
                <field id="order_status" translate="label" type="select" sortOrder="20" showInDefault="1" showInWebsite="1" showInStore="0">
                    <label>New Order Status</label>
                    <source_model> Magento\Sales\Model\Config \Source\Order\Status\NewStatus </source_model>
                </field>
                <field id="sort_order" translate="label" type="text" sortOrder="100" showInDefault="1" showInWebsite="1" showInStore="0">
                    <label>Sort Order</label>
                    <frontend_class> validate-number</frontend_class>
                </field>
                <field id="title" translate="label" type="text" sortOrder="10" showInDefault="1" showInWebsite="1" showInStore="1">
                    <label>Title</label>
                </field>
                <field id="allowspecific" translate="label" type="allowspecific" sortOrder="50" showInDefault="1" showInWebsite="1" showInStore="0">
                    <label>Payment from Applicable Countries </label>
                    <source_model> Magento\Payment\Model\ Config\Source\Allspecificcountries </source_model>
                </field>
                <field id="specificcountry" translate="label" type="multiselect" sortOrder="51" showInDefault="1" showInWebsite="1" showInStore="0">
                    <label>Payment from Specific Countries</label>
                    <source_model> Magento\Directory\Model \Config\Source\Country </source_model>
                    <can_be_empty>1</can_be_empty>
                </field>
                <field id="payable_to" translate="label" sortOrder="61" showInDefault="1" showInWebsite="1" showInStore="1">
                    <label>Make Check Payable to</label>
                </field>
                <field id="mailing_address" translate="label" type="textarea" sortOrder="62" showInDefault="1" showInWebsite="1" showInStore="1">
                    <label>Send Check to</label>
                </field>
                <field id="min_order_total" translate="label" type="text" sortOrder="98" showInDefault="1" showInWebsite="1" showInStore="0">
                    <label>Minimum Order Total</label>
                </field>
                <field id="max_order_total" translate="label" type="text" sortOrder="99" showInDefault="1" showInWebsite="1" showInStore="0">
                    <label>Maximum Order Total</label>
                </field>
                <field id="model"></field>
            </group>
        </section>
    </system>
</config>
```

创建一个名为 `app/code/Foggyline/Paybox/etc/frontend/di.xml` 的文件，内容如下：

```php
<config  xsi:noNamespaceSchemaLocation="urn:magento:framework: ObjectManager/etc/config.xsd">
    <type name="Magento\Checkout\Model\CompositeConfigProvider">
        <arguments>
            <argument name="configProviders" xsi:type="array">
                <item name= "offline_payment_paybox_config_provider" xsi:type="object">
                    Foggyline\Paybox\Model\PayboxConfigProvider
                </item>
            </argument>
        </arguments>
    </type>
</config>
```

完成此操作后，创建一个名为 `app/code/Foggyline/Paybox/Model/Paybox.php` 的文件，内容如下：

```php
namespace Foggyline\Paybox\Model;

class Paybox extends \Magento\Payment\Model\Method\AbstractMethod
{
    const PAYMENT_METHOD_PAYBOX_CODE = 'paybox';
    protected $_code = self::PAYMENT_METHOD_PAYBOX_CODE;

    protected $_isOffline = true;

    public function getPayableTo()
    {
        return $this->getConfigData('payable_to');
    }

    public function getMailingAddress()
    {
        return $this->getConfigData('mailing_address');
    }
}
```

现在，创建一个名为 `app/code/Foggyline/Paybox/Model/PayboxConfigProvider.php` 的文件，内容如下：

```php
namespace Foggyline\Paybox\Model;

class PayboxConfigProvider implements \Magento\Checkout\Model\ConfigProviderInterface
{
    protected $methodCode = \Foggyline\Paybox\Model\Paybox::PAYMENT_METHOD_PAYBOX_CODE;
    protected $method;
    protected $escaper;

    public function __construct(
        \Magento\Payment\Helper\Data $paymentHelper
    )
    {
        $this->method = $paymentHelper->getMethodInstance($this-> methodCode);
    }

    public function getConfig()
    {
        return $this->method->isAvailable() ? [
            'payment' => [
                'paybox' => [
                    'mailingAddress' => $this-> getMailingAddress(),
                    'payableTo' => $this->getPayableTo(),
                ],
            ],
        ] : [];
    }

    protected function getMailingAddress()
    {
        $this->method->getMailingAddress();
    }

    protected function getPayableTo()
    {
        return $this->method->getPayableTo();
    }
}
```

将整个 `vendor/magento/module-offline-payments/view/frontend/layout/checkout_index_index.xml` Magento 核心文件复制到 `app/code/Foggyline/Paybox/view/frontend/layout/checkout_index_index.xml` 模块中。然后，通过替换整个 `<item name="offline-payments" xsi:type="array">` 元素及其子元素，编辑模块的 `checkout_index_index.xml` 文件，如下所示：

```php
<item name="foggline-offline-payments" xsi:type="array">
    <item name="component" xsi:type="string"> Foggyline_Paybox/js/view/payment/foggline-offline-payments </item>
    <item name="methods" xsi:type="array">
        <item name="paybox" xsi:type="array">
            <item name="isBillingAddressRequired" xsi:type="boolean">true</item>
        </item>
    </item>
</item>
```

然后，创建一个名为 `app/code/Foggyline/Paybox/view/frontend/web/js/view/payment/offline-payments.js` 的文件，内容如下：

```php
/*browser:true*/
/*global define*/
define(
    [
        'uiComponent',
        'Magento_Checkout/js/model/payment/renderer-list'
    ],
    function (
        Component,
        rendererList
    ) {
        'use strict';
        rendererList.push(
            {
                type: 'paybox',
                component: 'Foggyline_Paybox/js/view/payment/method- renderer/paybox'
            }
        );
        return Component.extend({});
    }
);
```

完成此操作后，创建一个名为 `app/code/Foggyline/Paybox/view/frontend/web/js/view/payment/method-renderer/paybox.js` 的文件，内容如下：

```php
/*browser:true*/
/*global define*/
define(
    [
        'Magento_Checkout/js/view/payment/default'
    ],
    function (Component) {
        'use strict';

        return Component.extend({
            defaults: {
                template: 'Foggyline_Paybox/payment/paybox'
            },

            getMailingAddress: function () {
                return window.checkoutConfig.payment. paybox.mailingAddress;
            },

            getPayableTo: function () {
                return window.checkoutConfig.payment. paybox.payableTo;
            }
        });
    }
);
```

现在，创建一个名为 `app/code/Foggyline/Paybox/view/frontend/web/template/payment/paybox.html` 的文件，内容如下：

```php
<div class="payment-method" data-bind="css: {'_active': (getCode()  == isChecked())}">
    <div class="payment-method-title field choice">
        <input type="radio"
               name="payment[method]"
               class="radio"
               data-bind="attr: {'id': getCode()}, value: getCode(), checked: isChecked, click: selectPaymentMethod, visible: isRadioButtonVisible()"/>
        <label data-bind="attr: {'for': getCode()}" class="label"><span data-bind="text: getTitle()"></span></label>
    </div>
    <div class="payment-method-content">
        <div class="payment-method-billing-address">
            <!-- ko foreach: $parent.getRegion(getBillingAddressFormName()) -->
            <!-- ko template: getTemplate() --><!-- /ko -->
            <!--/ko-->
        </div>
        <!-- ko if: getMailingAddress() || getPayableTo() -->
        <dl class="items check payable">
            <!-- ko if: getPayableTo() -->
            <dt class="title"><!-- ko i18n: 'Make Check payable toooooo:' --><!-- /ko --></dt>
            <dd class="content"><!-- ko i18n: getPayableTo() --> <!-- /ko --></dd>
            <!-- /ko -->
            <!-- ko if: getMailingAddress() -->
            <dt class="title"><!-- ko i18n: 'Send Check toxyz:' -- ><!-- /ko --></dt>
            <dd class="content">
                <address class="paybox mailing address" data-bind ="html: $t(getMailingAddress())"></address>
            </dd>
            <!-- /ko -->
        </dl>
        <!-- /ko -->
        <div class="checkout-agreements-block">
            <!-- ko foreach: $parent.getRegion('before-place- order') -->
            <!-- ko template: getTemplate() --><!-- /ko -->
            <!--/ko-->
        </div>
        <div class="actions-toolbar">
            <div class="primary">
                <button class="action primary checkout"
                        type="submit"
                        data-bind="
                        click: placeOrder,
                        attr: {title: $t('Place Order')},
                        css: {disabled: !isPlaceOrderActionAllowed()},
                        enable: (getCode() == isChecked())
                        "
                        disabled>
                    <span data-bind="i18n: 'Place Order'"></span>
                </button>
            </div>
        </div>
    </div>
</div>
```

通过这种方式，我们完成了自定义离线支付方法模块。整个模块可以在 [`github.com/ajzele/B05032-Foggyline_Paybox`](https://github.com/ajzele/B05032-Foggyline_Paybox) 找到。

# 摘要

在本章中，我们讨论了一些开发者最常接触到的功能点。我们学习了在管理区域中查找信息的位置，以及如何编程管理这些功能背后的实体。因此，我们能够有效地手动和编程创建和获取 CMS 页面、块、分类和产品。我们还学习了如何创建产品和客户导入脚本。最后，我们研究了如何创建我们自己的自定义产品类型、简单支付和运输模块。

以下章节将引导我们了解 Magento 内置测试的使用方法，以及如何利用它们有效地进行应用程序的质量保证，以保持其健康状态。
