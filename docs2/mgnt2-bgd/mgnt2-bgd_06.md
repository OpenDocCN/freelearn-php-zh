# 第六章。插件

在本章中，我们将探讨 Magento 中的一个称为 **插件** 的功能。在我们开始使用插件之前，我们首先需要了解拦截这个术语，因为在处理 Magento 时，这两个术语有时是互换使用的。

**拦截** 是一种软件设计模式，当我们要动态地插入代码而不必改变原始类的行为时使用。这是通过在调用代码和目标对象之间动态插入代码来实现的。

Magento 中的拦截模式是通过插件实现的。它们提供了 `before`、`after` 和 `around` 监听器，这些监听器帮助我们扩展观察方法的行为。

在本章中，我们将涵盖以下主题：

+   创建一个插件

+   使用 `before` 监听器

+   使用 `after` 监听器

+   使用 `around` 监听器

+   插件排序

在我们开始创建插件之前，值得注意它们的限制。插件不能为任何类或方法创建，因为它们不适用于以下情况：

+   最终类

+   最终方法

+   没有依赖注入创建的类

让我们继续创建一个名为 `Foggyline_Plugged` 的简单模块的插件。

# 创建一个插件

首先，创建包含部分内容的 `app/code/Foggyline/Plugged/registration.php` 文件，如下所示：

```php
\Magento\Framework\Component\ComponentRegistrar::register(
    \Magento\Framework\Component\ComponentRegistrar::MODULE,
    'Foggyline_Plugged',
    __DIR__
);
```

然后，创建包含部分内容的 `app/code/Foggyline/Plugged/etc/module.xml` 文件，如下所示：

```php
<config  xsi:noNamespaceSchemaLocation="urn:magento:framework:Module/ etc/module.xsd">
    <module name="Foggyline_Plugged" setup_version="1.0.0">
        <sequence>
            <module name="Magento_Catalog"/>
        </sequence>
    </module>
</config>
```

前面的文件只是一个新的模块声明，其依赖项针对 `Magento_Catalog` 模块，因为我们将会观察其类。我们现在不会深入讨论模块声明，因为这将在接下来的章节中介绍。

现在，创建包含部分内容的 `app/code/Foggyline/Plugged/etc/di.xml` 文件，如下所示：

```php
<config  xsi:noNamespaceSchemaLocation="urn:magento:framework: ObjectManager/etc/config.xsd">
    <type name="Magento\Catalog\Block\Product\AbstractProduct">
        <plugin name="foggyPlugin1" type="Foggyline\Plugged\Block\Catalog\Product\ AbstractProductPlugin1" disabled="false" sortOrder="100"/>
        <plugin name="foggyPlugin2" type="Foggyline\Plugged\Block\Catalog\Product\ AbstractProductPlugin2" disabled="false" sortOrder="200"/>
        <plugin name="foggyPlugin3" type="Foggyline\Plugged\Block\Catalog\Product\ AbstractProductPlugin3" disabled="false" sortOrder="300"/>
    </type>
</config>
```

插件定义在模块 `di.xml` 文件中。要定义一个插件，我们首先使用 `type` 元素及其 `name` 属性映射我们想要观察的类。在这种情况下，我们正在观察 `Magento\Catalog\Block\Product\AbstractProduct` 类。请注意，尽管文件和类名暗示了一个抽象类型的类，但 `AbstractProduct` 类并不是抽象的。

在 `type` 元素中，我们随后使用 `plugin` 元素定义一个或多个插件。

`plugin` 元素分配了以下四个属性：

+   `name`: 使用此属性，您可以提供一个独特且易于识别的名称值，该值特定于插件

+   `sortOrder`: 此属性确定当多个插件观察同一方法时的执行顺序

+   `disabled`: 此属性的默认值设置为 `false`，但如果设置为 `true`，则将禁用插件

+   `type`: 此属性指向我们将要使用的类来实现 `before`、`after` 或 `around` 监听器

在完成此操作后，创建包含部分内容的 `app/code/Foggyline/Plugged/Block/Catalog/Product/AbstractProductPlugin1.php` 文件，如下所示：

```php
namespace Foggyline\Plugged\Block\Catalog\Product;

class AbstractProductPlugin1
{
    public function beforeGetAddToCartUrl(
        $subject,
        $product, $additional = []
    )
    {
        var_dump('Plugin1 - beforeGetAddToCartUrl');
    }

    public function afterGetAddToCartUrl($subject)
    {
        var_dump('Plugin1 - afterGetAddToCartUrl');
    }

    public function aroundGetAddToCartUrl(
        $subject,
        \Closure $proceed,
        $product,
        $additional = []
    )
    {
        var_dump('Plugin1 - aroundGetAddToCartUrl');
        return $proceed($product, $additional);
    }
}
```

根据`di.xml`文件中的类型定义，该插件观察`Magento\Catalog\Block\Product\AbstractProduct`类，并且这个类有一个名为`getAddToCartUrl`的方法，其定义如下：

```php
public function getAddToCartUrl($product, $additional = [])
{
    //method body here...
}
```

`AbstractProductPlugin1`类不需要从另一个类扩展，我们可以通过使用命名约定定义`before`、`after`和`around`监听器来为`getAddToCartUrl`方法工作，如下所示：

```php
<before> + <getAddToCartUrl> => beforeGetAddToCartUrl
<after> + <getAddToCartUrl> => afterGetAddToCartUrl
<around> + <getAddToCartUrl> => aroundGetAddToCartUrl
```

我们将在稍后详细说明每个监听器。现在，我们需要通过创建`AbstractProductPlugin2.php`和`AbstractProductPlugin3.php`文件来完成模块，这些文件是`AbstractProductPlugin1.php`的副本，并且简单地将其代码中的所有数字值从`1`更改为`2`或`3`。

将监听器组织成与被观察类位置结构相匹配的文件夹是一个好习惯。例如，如果一个模块名为`Foggyline_Plugged`，并且我们在`Magento\Catalog\Block\Product\AbstractProduct`类中观察方法，我们应该考虑将插件类放入`Foggyline/Plugged/Block/Catalog/Product/AbstractProductPlugin.php`文件中。这并不是一个要求。相反，这是一个好的约定，以便其他开发者可以轻松地管理代码。

一旦模块到位，我们需要在控制台执行以下命令：

```php
php bin/magento module:enable Foggyline_Plugged
php bin/magento setup:upgrade
```

这将使模块对 Magento 可见。

如果我们现在在浏览器中打开一个分类页面的店面，我们将看到所有`var_dump`函数调用的结果。

让我们详细查看每个监听器方法。

# 使用`before`监听器

`before`监听器用于我们想要更改原始方法的参数或在原始方法被调用之前添加一些行为时。

回顾`beforeGetAddToCartUrl`监听方法定义，你会看到它按顺序分配了三个属性——`$subject`、`$product`和`$additional`。

使用`before`方法监听器，第一个属性总是`$subject`属性，它包含被观察的对象类型的实例。在`$subject`属性之后的属性按照顺序匹配被观察的`getAddToCartUrl`方法的属性。

用于转换的简单规则如下：

```php
getAddToCartUrl($product, $additional = [])
beforeGetAddToCartUrl($subject, $product, $additional = [])
```

`before`监听方法不需要有返回值。

如果我们在之前看到的`beforeGetAddToCartUrl`监听方法中运行`get_class($subject)`，我们将得到以下结果：

```php
\Magento\Catalog\Block\Product\ListProduct\Interceptor
    extends \Magento\Catalog\Block\Product\ListProduct
        extends \Magento\Catalog\Block\Product\AbstractProduct
```

这表明，尽管我们正在观察`AbstractProduct`类，但`$subject`属性并不是直接那种类型。相反，它是`ListProduct\Interceptor`类型。这是你在开发过程中应该记住的事情。

# 使用`after`监听器

`after`监听器用于我们想要更改原始方法返回的值或在原始方法被调用后添加一些行为时。

回顾一下 `afterGetAddToCartUrl` 拦截器方法定义，你会看到它只分配了一个 `$subject` 属性。

使用 `after` 方法拦截器时，第一个且唯一的属性始终是 `$subject` 属性，它包含被观察的对象类型的实例，而不是被观察方法的返回值。

用于转换的简单规则如下：

```php
getAddToCartUrl($product, $additional = [])
afterGetAddToCartUrl($subject)
```

`after` 拦截器方法不需要有返回值。

与 `before` 拦截器方法类似，在这种情况下，`$subject` 属性不是直接属于 `AbstractProduct` 类型。相反，它是父类 `ListProduct\Interceptor` 类型。

# 使用 `around` 拦截器

当我们想要更改原始方法的参数和返回值，或者在调用原始方法前后添加一些行为时，使用 `around` 拦截器。

回顾一下 `aroundGetAddToCartUrl` 拦截器方法定义，你会看到它按顺序分配了四个属性——`$subject`、`$proceed`、`$product` 和 `$additional`。

使用 `after` 方法拦截器时，第一个属性始终是 `$subject` 属性，它包含被观察的对象类型的实例，而不是被观察方法的返回值。第二个属性始终是 `\Closure` 的 `$proceed` 属性。在 `$subject` 和 `$proceed` 之后跟随的属性与被观察的 `getAddToCartUrl` 方法的属性顺序相匹配。

用于转换的简单规则如下：

```php
getAddToCartUrl($product, $additional = [])
aroundGetAddToCartUrl(
    $subject,
    \Closure $proceed,
    $product,
    $additional = []
)
```

`around` 拦截器方法必须有一个返回值。返回值以这种方式形成，即 `around` 拦截器方法定义中 `$closure` 参数之后的参数按顺序传递给 `$closure` 函数调用，如下所示：

```php
return $proceed($product, $additional);
//or
$result = $proceed($product, $additional);
return $result;
```

# 插件排序顺序

回顾一下，当我们定义 `di.xml` 文件中的插件时，为每个插件定义设置的属性之一是 `sortOrder`。它被设置为 `100`，`200` 到 `300` 分别为 `foggyPlugin1`、`foggyPlugin2` 和 `foggyPlugin3`。

上述插件代码执行的流程如下：

+   `Plugin1 - beforeGetAddToCartUrl`

+   `Plugin1 - aroundGetAddToCartUrl`

+   `Plugin2 - beforeGetAddToCartUrl`

+   `Plugin2 - aroundGetAddToCartUrl`

+   `Plugin3 - beforeGetAddToCartUrl`

+   `Plugin3 - aroundGetAddToCartUrl`

+   `Plugin3 - afterGetAddToCartUrl`

+   `Plugin2 - afterGetAddToCartUrl`

+   `Plugin1 - afterGetAddToCartUrl`

换句话说，如果有多个插件监听同一个方法，将使用以下执行顺序：

+   按照排序顺序从低到高的 `before` 插件功能

+   具有最低 `sortOrder` 值的 `around` 插件功能

+   按照排序顺序从低到高的 `before` 插件功能

+   `around` 插件功能遵循 `sortOrder` 值从低到高

+   具有最高 `sortOrder` 值的 `after` 插件功能

+   `after` 插件函数按照 `sortOrder` 值从高到低执行

### 注意

在处理 `around` 监听器时需要特别注意，因为它是唯一需要返回值的监听器。如果我们省略返回值，可能会以这种方式破坏执行流程，导致同一方法的其它 `around` 插件无法执行。

# 摘要

在本章中，我们探讨了 Magento 中的一个强大功能——插件。我们创建了一个包含三个插件的模块；每个插件都有不同的排序顺序。这使得我们能够追踪观察同一方法的多个插件的执行流程。我们详细探讨了 `before`、`after` 和 `around` 监听器方法，同时特别强调了参数顺序。本章中使用的最终模块可以在[`github.com/ajzele/B05032-Foggyline_Plugged`](https://github.com/ajzele/B05032-Foggyline_Plugged)找到。

在下一章中，我们将深入探讨后端开发。
