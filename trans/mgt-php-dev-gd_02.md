# 第二章。开发人员的 Magento 基础知识

在本章中，我们将介绍与 Magento 一起工作的基本概念。我们将了解 Magento 的结构，并将介绍 Magento 灵活性的来源，即其模块化架构。

Magento 是一个灵活而强大的系统。不幸的是，这也增加了一定程度的复杂性。目前，Magento 的干净安装大约有 30,000 个文件和超过 120 万行代码。

拥有如此强大和复杂的功能，Magento 对于新开发人员可能会令人望而生畏；但不用担心。本章旨在教新开发人员所有他们需要使用和扩展 Magento 的基本概念和工具，在下一章中，我们将深入研究 Magento 的模型和数据集。

# Zend Framework – Magento 的基础

您可能知道，Magento 是市场上最强大的电子商务平台；您可能不知道的是，Magento 还是一个基于 Zend Framework 开发的**面向对象**（**OO**）PHP 框架。

Zend 的官方网站描述了该框架为：

> *Zend Framework 2 是一个使用 PHP 5.3+开发 Web 应用程序和服务的开源框架。Zend Framework 2 使用 100%面向对象的代码，并利用了 PHP 5.3 的大多数新特性，即命名空间、后期静态绑定、lambda 函数和闭包。*
> 
> *Zend Framework 2 的组件结构是独特的；每个组件都设计为对其他组件的依赖较少。ZF2 遵循 SOLID 面向对象设计原则。这种松散耦合的架构允许开发人员使用他们想要的任何组件。我们称之为“随意使用”设计。*

但是 Zend Framework 究竟是什么？Zend Framework 是一个基于 PHP 开发的面向对象框架，实现了**模型-视图-控制器**（**MVC**）范式。当 Varien，现在的 Magento 公司，开始开发 Magento 时，决定在 Zend 的基础上进行开发，因为以下组件：

+   `Zend_Cache`

+   `Zend_Acl`

+   `Zend_Locale`

+   `Zend_DB`

+   `Zend_Pdf`

+   `Zend_Currency`

+   `Zend_Date`

+   `Zend_Soap`

+   `Zend_Http`

总的来说，Magento 使用了大约 15 个不同的 Zend 组件。Varien 库直接扩展了先前提到的几个 Zend 组件，例如`Varien_Cache_Core`是从`Zend_Cache_Core`扩展而来的。

使用 Zend Framework，Magento 是根据以下原则构建的：

+   **可维护性**：通过使用代码池来将核心代码与本地定制和第三方模块分开

+   **可升级性**：Magento 的模块化允许扩展和第三方模块独立于系统的其他部分进行更新

+   **灵活性**：允许无缝定制并简化新功能的开发

虽然使用 Zend Framework 甚至理解它并不是开发 Magento 的要求，但至少对 Zend 组件、用法和交互有基本的了解，在我们开始深入挖掘 Magento 的核心时，可能会是非常宝贵的信息。

### 注意

您可以在[`framework.zend.com/`](http://framework.zend.com/)了解更多关于 Zend Framework 的信息。

# Magento 文件夹结构

Magento 的文件夹结构与其他 MVC 应用程序略有不同；让我们来看看目录树，以及每个目录及其功能：

+   `app`：这个文件夹是 Magento 的核心，分为三个导入目录：

+   `code`：这包含了我们的应用程序代码，分为`core`、`community`和`local`三个代码池

+   `design`：这包含了我们应用程序的所有模板和布局

+   `locale`：这包含了商店使用的所有翻译和电子邮件模板文件

+   `js`：这包含了 Magento 中使用的所有 JavaScript 库

+   `media`：这包含了我们产品和 CMS 页面的所有图片和媒体文件，以及产品图片缓存

+   `lib`：这包含 Magento 使用的所有第三方库，如 Zend 和 PEAR，以及 Magento 开发的自定义库，这些库位于 Varien 和 Mage 目录下

+   `皮肤`：这包含对应主题使用的所有 CSS 代码、图像和 JavaScript 文件

+   `var`：这包含我们的临时数据，如缓存文件、索引锁文件、会话、导入/导出文件，以及企业版中的完整页面缓存文件夹

Magento 是一个模块化系统。这意味着应用程序，包括核心，被划分为较小的模块。因此，文件夹结构在每个模块核心的组织中起着关键作用；典型的 Magento 模块文件夹结构看起来像下面的图：

![Magento 文件夹结构](img/3060OS_02_01.jpg)

让我们更详细地审查每个文件夹：

+   `块`：这个文件夹包含 Magento 中形成控制器和视图之间的额外逻辑的块

+   `controllers`：`controllers`文件夹由处理 Web 服务器请求的操作组成

+   `控制器`：这个文件夹中的类是抽象类，由`controllers`文件夹下的`controller`类扩展

+   `etc`：在这里，我们可以找到以 XML 文件形式的模块特定配置，例如`config.xml`和`system.xml`

+   `助手`：这个文件夹包含封装常见模块功能并使其可用于同一模块的类和其他模块类的辅助类

+   `模型`：这个文件夹包含支持模块中控制器与数据交互的模型

+   `sql`：这个文件夹包含每个特定模块的安装和升级文件

正如我们将在本章后面看到的那样，Magento 大量使用工厂名称和工厂方法。这就是为什么文件夹结构如此重要的原因。

# 模块化架构

Magento 不是一个庞大的应用程序，而是由较小的模块构建，每个模块为 Magento 添加特定功能。

这种方法的优势之一是能够轻松启用和禁用特定模块功能，以及通过添加新模块来添加新功能。

## 自动加载程序

Magento 是一个庞大的框架，由近 30000 个文件组成。在应用程序启动时需要每个文件将使其变得非常缓慢和沉重。因此，Magento 使用自动加载程序类来在每次调用工厂方法时找到所需的文件。

那么，自动加载程序到底是什么？PHP5 包含一个名为`__autoload()`的函数。在实例化类时，`__autoload()`函数会自动调用；在这个函数内部，定义了自定义逻辑来解析类名和所需文件。

让我们仔细看看位于`app/Mage.php`的 Magento 引导代码：

```php
… 
Mage::register('original_include_path', get_include_path());
if (defined('COMPILER_INCLUDE_PATH')) {
    $appPath = COMPILER_INCLUDE_PATH;
    set_include_path($appPath . PS . Mage::registry('original_include_path'));
    include_once "Mage_Core_functions.php";
    include_once "Varien_Autoload.php";
} else {
    /**
     * Set include path
     */
    $paths[] = BP . DS . 'app' . DS . 'code' . DS . 'local';
    $paths[] = BP . DS . 'app' . DS . 'code' . DS . 'community';
    $paths[] = BP . DS . 'app' . DS . 'code' . DS . 'core';
    $paths[] = BP . DS . 'lib';

    $appPath = implode(PS, $paths);
    set_include_path($appPath . PS . Mage::registry('original_include_path'));
    include_once "Mage/Core/functions.php";
    include_once "Varien/Autoload.php";
}

Varien_Autoload::register();
```

引导文件负责定义`include`路径和初始化 Varien 自动加载程序，后者将定义自己的`autoload`函数作为默认调用函数。让我们来看看 Varien `autoload`函数的内部工作：

```php
    /**
     * Load class source code
     *
     * @param string $class
     */
    public function autoload($class)
    {
        if ($this->_collectClasses) {
            $this->_arrLoadedClasses[self::$_scope][] = $class;
        }
        if ($this->_isIncludePathDefined) {
            $classFile =  COMPILER_INCLUDE_PATH . DIRECTORY_SEPARATOR . $class;
        } else {
            $classFile = str_replace(' ', DIRECTORY_SEPARATOR, ucwords(str_replace('_', ' ', $class)));
        }
        $classFile.= '.php';
        //echo $classFile;die();
        return include $classFile;
    }
```

`autoload`类接受一个名为`$class`的参数，这是工厂方法提供的别名。这个别名被处理以生成一个匹配的类名，然后被包含。

正如我们之前提到的，Magento 的目录结构很重要，因为 Magento 从目录结构中派生其类名。这种约定是我们将在本章后面审查的工厂方法的核心原则。

## 代码池

正如我们之前提到的，在我们的`app/code`文件夹中，我们的应用程序代码分为三个不同的目录，称为代码池。它们如下：

+   `核心`：这是 Magento 核心模块提供基本功能的地方。Magento 开发人员之间的黄金法则是，绝对不要修改`core`代码池下的任何文件。

+   `community`：这是第三方模块放置的位置。它们要么由第三方提供，要么通过 Magento Connect 安装。

+   `本地`：这是专门为 Magento 实例开发的所有模块和代码所在的位置。

代码池确定模块来自何处以及它们应该被加载的顺序。如果我们再看一下`Mage.php`引导文件，我们可以看到代码池加载的顺序：

```php
    $paths[] = BP . DS . 'app' . DS . 'code' . DS . 'local';
    $paths[] = BP . DS . 'app' . DS . 'code' . DS . 'community';
    $paths[] = BP . DS . 'app' . DS . 'code' . DS . 'core';
    $paths[] = BP . DS . 'lib';
```

这意味着对于每个类请求，Magento 将首先查找`local`，然后是`community`，然后是`core`，最后是`lib`文件夹内的内容。

这也导致了一个有趣的行为，可以很容易地用于覆盖`core`和`community`类，只需复制目录结构并匹配类名。

### 提示

毋庸置疑，这是一个糟糕的做法，但了解这一点仍然是有用的，以防将来有一天你不得不处理利用这种行为的项目。

# 路由和请求流程

在更详细地了解构成 Magento 一部分的不同组件之前，重要的是我们了解这些组件如何相互交互以及 Magento 如何处理来自 Web 服务器的请求。

与任何其他 PHP 应用程序一样，我们有一个单一文件作为每个请求的入口点；在 Magento 的情况下，这个文件是`index.php`，负责加载`Mage.php`引导类并启动请求周期。然后它经历以下步骤：

1.  Web 服务器接收请求，并通过调用引导文件`Mage.php`来实例化 Magento。

1.  前端控制器被实例化和初始化；在控制器初始化期间，Magento 搜索 web 路由并实例化它们。

1.  然后 Magento 遍历每个路由器并调用匹配。`match`方法负责处理 URL 并生成相应的控制器和操作。

1.  Magento 然后实例化匹配的控制器并执行相应的操作。

路由器在这个过程中尤其重要。前端控制器使用`Router`对象将请求的 URL（路由）与模块控制器和操作进行匹配。默认情况下，Magento 带有以下路由器：

+   `Mage_Core_Controller_Varien_Router_Admin`

+   `Mage_Core_Controller_Varien_Router_Standard`

+   `Mage_Core_Controller_Varien_Router_Default`

然后动作控制器将加载和渲染布局，然后加载相应的块、模型和模板。

让我们分析一下 Magento 如何处理对类别页面的请求；我们将使用`http://localhost/catalog/category/view/id/10`作为示例。Magento 的 URI 由三部分组成 - */FrontName/ControllerName/ActionName*。

这意味着对于我们的示例 URL，拆分将如下所示：

+   **FrontName**：`catalog`

+   **ControllerName**：`category`

+   **ActionName**：`view`

如果我看一下 Magento 路由器类，我可以看到`Mage_Core_Controller_Varien_Router_Standard`匹配函数：

```php
public function match(Zend_Controller_Request_Http $request)
{
  …
   $path = trim($request->getPathInfo(), '/');
            if ($path) {
                $p = explode('/', $path);
            } else {
                $p = explode('/', $this->_getDefaultPath());
            }
  …
}
```

从前面的代码中，我们可以看到路由器尝试做的第一件事是将 URI 解析为数组。根据我们的示例 URL，相应的数组将类似于以下代码片段：

```php
$p = Array
(
    [0] => catalog
    [1] => category
    [2] => view
)
```

函数的下一部分将首先尝试检查请求是否指定了模块名称；如果没有，则尝试根据数组的第一个元素确定模块名称。如果无法提供模块名称，则函数将返回`false`。让我们看看代码的这一部分：

```php
      // get module name
        if ($request->getModuleName()) {
            $module = $request->getModuleName();
        } else {
            if (!empty($p[0])) {
                $module = $p[0];
            } else {
                $module = $this->getFront()->getDefault('module');
                $request->setAlias(Mage_Core_Model_Url_Rewrite::REWRITE_REQUEST_PATH_ALIAS, '');
            }
        }
        if (!$module) {
            if (Mage::app()->getStore()->isAdmin()) {
                $module = 'admin';
            } else {
                return false;
            }
        }
```

接下来，匹配函数将遍历每个可用模块，并尝试匹配控制器和操作，使用以下代码：

```php
…
        foreach ($modules as $realModule) {
            $request->setRouteName($this->getRouteByFrontName($module));

            // get controller name
            if ($request->getControllerName()) {
                $controller = $request->getControllerName();
            } else {
                if (!empty($p[1])) {
                    $controller = $p[1];
                } else {
                    $controller = $front->getDefault('controller');
                    $request->setAlias(
                        Mage_Core_Model_Url_Rewrite::REWRITE_REQUEST_PATH_ALIAS,
                        ltrim($request->getOriginalPathInfo(), '/')
                    );
                }
            }

            // get action name
            if (empty($action)) {
                if ($request->getActionName()) {
                    $action = $request->getActionName();
                } else {
                    $action = !empty($p[2]) ? $p[2] : $front->getDefault('action');
                }
            }

            //checking if this place should be secure
            $this->_checkShouldBeSecure($request, '/'.$module.'/'.$controller.'/'.$action);

            $controllerClassName = $this->_validateControllerClassName($realModule, $controller);
            if (!$controllerClassName) {
                continue;
            }

            // instantiate controller class
            $controllerInstance = Mage::getControllerInstance($controllerClassName, $request, $front->getResponse());

            if (!$controllerInstance->hasAction($action)) {
                continue;
            }

            $found = true;
            break;
        }
...
```

现在看起来代码量很大，所以让我们进一步分解。循环的第一部分将检查请求是否有一个控制器名称；如果没有设置，它将检查我们的参数数组（$p）的第二个值，并尝试确定控制器名称，然后它将尝试对操作名称做同样的事情。

如果我们在循环中走到了这一步，我们应该有一个模块名称，一个控制器名称和一个操作名称，Magento 现在将使用它们来尝试通过调用以下函数获取匹配的控制器类名：

```php
$controllerClassName = $this->_validateControllerClassName($realModule, $controller);
```

这个函数不仅会生成一个匹配的类名，还会验证它的存在；在我们的例子中，这个函数应该返回`Mage_Catalog_CategoryController`。

由于我们现在有了一个有效的类名，我们可以继续实例化我们的控制器对象；如果你一直关注到这一点，你可能已经注意到我们还没有对我们的操作做任何事情，这正是我们循环中的下一步。

我们新实例化的控制器带有一个非常方便的函数叫做`hasAction()`；实质上，这个函数的作用是调用一个名为`is_callable()`的 PHP 函数，它将检查我们当前的控制器是否有一个与操作名称匹配的公共函数；在我们的例子中，这将是`viewAction()`。

这种复杂的匹配过程和使用`foreach`循环的原因是，可能有几个模块使用相同的 FrontName。

![路由和请求流程](img/3060OS_02_03.jpg)

现在，`http://localhost/catalog/category/view/id/10`不是一个非常用户友好的 URL；幸运的是，Magento 有自己的 URL 重写系统，允许我们使用`http://localhost/books.html`。

让我们深入了解一下 URL 重写系统，看看 Magento 如何从我们的 URL 别名中获取控制器和操作名称。在我们的`Varien/Front.php`控制器分发函数中，Magento 将调用：

```php
Mage::getModel('core/url_rewrite')->rewrite();
```

在实际查看`rewrite`函数的内部工作之前，让我们先看一下`core/url_rewrite`模型的结构：

```php
Array (
  ["url_rewrite_id"] => "10"
  ["store_id"]       => "1"
  ["category_id"]    => "10"
  ["product_id"]     => NULL
  ["id_path"]        => "category/10"
  ["request_path"]   => "books.html"
  ["target_path"]    => "catalog/category/view/id/10"
  ["is_system"]      => "1"
  ["options"]        => NULL
  ["description"]    => NULL
)
```

正如我们所看到的，重写模块由几个属性组成，但其中只有两个对我们特别感兴趣——`request_path`和`target_path`。简而言之，重写模块的工作是修改请求对象路径信息，使其与`target_path`的匹配值相匹配。

# Magento 的 MVC 版本

如果您熟悉传统的 MVC 实现，比如 CakePHP 或 Symfony，您可能知道最常见的实现被称为基于约定的 MVC。使用基于约定的 MVC，要添加一个新模型或者说一个控制器，你只需要创建文件/类（遵循框架约定），系统就会自动接收它。

Magento，另一方面，使用基于配置的 MVC 模式，这意味着创建我们的文件/类是不够的；我们必须明确告诉 Magento 我们添加了一个新类。

每个 Magento 模块都有一个`config.xml`文件，位于模块的`etc/`目录下，包含所有相关的模块配置。例如，如果我们想要添加一个包含新模型的新模块，我们需要在配置文件中定义一个节点，告诉 Magento 在哪里找到我们的模型，比如：

```php
<global>
…
<models>
     <group_classname>
          <class>Namespace_Modulename_Model</class>
     <group_classname>
</models>
...
</global>
```

虽然这可能看起来像是额外的工作，但它也给了我们巨大的灵活性和权力。例如，我们可以使用`rewrite`节点重写另一个类：

```php
<global>
…
<models>
     <group_classname>
      <rewrite>
               <modulename>Namespace_Modulename_Model</modulename>
      </rewrite>
     <group_classname>
</models>
...
</global>
```

Magento 然后会加载所有的`config.xml`文件，并在运行时合并它们，创建一个单一的配置树。

此外，模块还可以有一个`system.xml`文件，用于在 Magento 后台指定配置选项，这些选项又可以被最终用户用来配置模块功能。`system.xml`文件的片段如下所示：

```php
<config>
  <sections>
    <section_name translate="label">
      <label>Section Description</label>
      <tab>general</tab>
      <frontend_type>text</frontend_type>
      <sort_order>1000</sort_order>
      <show_in_default>1</show_in_default>
      <show_in_website>1</show_in_website>
      <show_in_store>1</show_in_store>
      <groups>
       <group_name translate="label">
         <label>Demo Of Config Fields</label>
         <frontend_type>text</frontend_type>
         <sort_order>1</sort_order>
         <show_in_default>1</show_in_default>
         <show_in_website>1</show_in_website>
         <show_in_store>1</show_in_store>  
   <fields>
          <field_name translate="label comment">
             <label>Enabled</label>
             <comment>
               <![CDATA[Comments can contain <strong>HTML</strong>]]>
             </comment>
             <frontend_type>select</frontend_type>
             <source_model>adminhtml/system_config_source_yesno</source_model>
             <sort_order>10</sort_order>
             <show_in_default>1</show_in_default>
             <show_in_website>1</show_in_website>
             <show_in_store>1</show_in_store>
          </field_name>
         </fields>
        </group_name>
       </groups>
    </section_name>
  </sections>
</config>
```

让我们分解每个节点的功能：

+   `section_name`：这只是一个我们用来标识配置部分的任意名称；在此节点内，我们将指定配置部分的所有字段和组。

+   `group`：组，顾名思义，用于对配置选项进行分组，并在手风琴部分内显示它们。

+   `label`：这定义了字段/部分/组上要使用的标题或标签。

+   `tab`：这定义了应在其中显示部分的选项卡。

+   `frontend_type`：此节点允许我们指定要为自定义选项字段使用的渲染器。一些可用的选项包括：

+   `button`

+   `checkboxes`

+   `checkbox`

+   `date`

+   `file`

+   `hidden`

+   `image`

+   `label`

+   `link`

+   `multiline`

+   `multiselect`

+   `password`

+   `radio`

+   `radios`

+   `select`

+   `submit`

+   `textarea`

+   `text`

+   `time`

+   `sort_order`：它指定字段、组或部分的位置。

+   `source_model`：某些类型的字段，如`select`字段，可以从源模型中获取选项。Magento 已经在`Mage/Adminhtml/Model/System/Config/Source`下提供了几个有用的类。我们可以找到一些类：

+   `YesNo`

+   `Country`

+   `Currency`

+   `AllRegions`

+   `Category`

+   `Language`

仅通过使用 XML，我们就可以在 Magento 后端为我们的模块构建复杂的配置选项，而无需担心设置模板来填充字段或验证数据。

Magento 还提供了大量的表单字段验证模型，我们可以在`<validate>`标签中使用。在以下字段验证器中，我们有：

+   `validate-email`

+   `validate-length`

+   `validate-url`

+   `validate-select`

+   `validate-password`

与 Magento 的任何其他部分一样，我们可以扩展`source_model`，`frontend_type`和`validator`函数，甚至创建新的函数。我们将在后面的章节中处理这个任务，在那里我们将创建每种新类型。但现在，我们将探讨模型、视图、文件布局和控制器的概念。

## 模型

Magento 使用 ORM 方法；虽然我们仍然可以使用`Zend_Db`直接访问数据库，但我们大多数时候将使用模型来访问我们的数据。对于这种类型的任务，Magento 提供了以下两种类型的模型：

+   **简单模型**：这种模型实现是一个简单的将一个对象映射到一个表，意味着我们的对象属性与每个字段匹配，表结构

+   **实体属性值（EAV）模型**：这种类型的模型用于描述具有动态属性数量的实体

Magento 将模型层分为两部分：处理业务逻辑的模型和处理数据库交互的资源。这种设计决策使 Magento 最终能够支持多个数据库平台，而无需更改模型内部的任何逻辑。

Magento ORM 使用 PHP 的一个魔术类方法来提供对对象属性的动态访问。在下一章中，我们将更详细地了解模型、Magento ORM 和数据集合。

### 注意

Magento 模型不一定与数据库中的任何类型的表或 EAV 实体相关。稍后我们将要审查的观察者就是这种类型的 Magento 模型的完美例子。

## 视图

视图层是 Magento 真正使自己与其他 MVC 应用程序区分开的领域之一。与传统的 MVC 系统不同，Magento 的视图层分为以下三个不同的组件：

+   **布局**：布局是定义块结构和属性（如名称和我们可以使用的模板文件）的 XML 文件。每个 Magento 模块都有自己的布局文件集。

+   **块**：块在 Magento 中用于通过将大部分逻辑移动到块中来减轻控制器的负担。

+   **模板**：模板是包含所需 HTML 代码和 PHP 标记的 PHTML 文件。

布局为 Magento 前端提供了令人惊讶的灵活性。每个模块都有自己的布局 XML 文件，告诉 Magento 在每个页面请求上包含和渲染什么。通过使用布局，我们可以在不担心改变除了我们的 XML 文件之外的任何其他内容的情况下，移动、添加或删除我们商店的块。

## 解剖布局文件

让我们来看看 Magento 的一个核心布局文件，比如`catalog.xml`：

```php
<layout version="0.1.0">
<default>
    <reference name="left">
        <block type="core/template" name="left.permanent.callout" template="callouts/left_col.phtml">
            <action method="setImgSrc"><src>images/media/col_left_callout.jpg</src></action>
            <action method="setImgAlt" translate="alt" module="catalog"><alt>Our customer service is available 24/7\. Call us at (555) 555-0123.</alt></action>
            <action method="setLinkUrl"><url>checkout/cart</url></action>
        </block>
    </reference>
    <reference name="right">
        <block type="catalog/product_compare_sidebar" before="cart_sidebar" name="catalog.compare.sidebar" template="catalog/product/compare/sidebar.phtml"/>
        <block type="core/template" name="right.permanent.callout" template="callouts/right_col.phtml">
            <action method="setImgSrc"><src>images/media/col_right_callout.jpg</src></action>
            <action method="setImgAlt" translate="alt" module="catalog"><alt>Visit our site and save A LOT!</alt></action>
        </block>
    </reference>
    <reference name="footer_links">
        <action method="addLink" translate="label title" module="catalog" ifconfig="catalog/seo/site_map"><label>Site Map</label><url helper="catalog/map/getCategoryUrl" /><title>Site Map</title></action>
    </reference>
    <block type="catalog/product_price_template" name="catalog_product_price_template" />
</default>
```

布局块由三个主要的 XML 节点组成，如下所示：

+   `handle`：每个页面请求将具有几个唯一的句柄；布局使用这些句柄告诉 Magento 在每个页面上加载和渲染哪些块。最常用的句柄是`default`和`[frontname]_[controller]_[action]`。

`default`句柄特别适用于设置全局块，例如在页眉块上添加 CSS 或 JavaScript。

+   `reference`：`<reference>`节点用于引用一个块。它用于指定嵌套块或修改已经存在的块。在我们的示例中，我们可以看到在`<reference name="left">`内指定了一个新的子块。

+   `block`：`<block>`节点用于加载我们的实际块。每个块节点可以具有以下属性：

+   `type`：这是实际块类的标识符。例如，`catalog`/`product_list`指的是`Mage_Catalog_Block_Product_List`。

+   `name`：其他块用这个名称来引用这个块。

+   `before`/`after`：这些属性可用于相对于其他块的位置定位块。这两个属性都可以使用连字符作为值，以指定模块是应该出现在最顶部还是最底部。

+   `template`：此属性确定将用于渲染块的`.phtml`模板文件。

+   `action`：每个块类型都有影响前端功能的特定操作。例如，`page`/`html_head`块具有用于添加 CSS 和 JavaScript（`addJs`和`addCss`）的操作。

+   `as`：用于指定我们将在模板中调用的块的唯一标识符，例如使用`getChildHtml('block_name')`调用子块。

块是 Magento 实现的一个新概念，以减少控制器的负载。它们基本上是直接与模型通信的数据资源，模型操作数据（如果需要），然后将其传递给视图。

最后，我们有我们的 PHTML 文件；模板包含`html`和`php`标记，并负责格式化和显示来自我们模型的数据。让我们来看一下产品视图模板的片段：

```php
<div class="product-view">
...
    <div class="product-name">
        <h1><?php echo $_helper->productAttribute($_product, $_product->getName(), 'name') ?></h1>
    </div>
...           
    <?php echo $this->getReviewsSummaryHtml($_product, false, true)?>
    <?php echo $this->getChildHtml('alert_urls') ?>
    <?php echo $this->getChildHtml('product_type_data') ?>
    <?php echo $this->getTierPriceHtml() ?>
    <?php echo $this->getChildHtml('extrahint') ?>
...

    <?php if ($_product->getShortDescription()):?>
        <div class="short-description">
            <h2><?php echo $this->__('Quick Overview') ?></h2>
            <div class="std"><?php echo $_helper->productAttribute($_product, nl2br($_product->getShortDescription()), 'short_description') ?></div>
        </div>
    <?php endif;?>
...
</div>
```

以下是 MVC 的块图：

![解剖布局文件](img/3060OS_02_02.jpg)

## 控制器

在 Magento 中，MVC 控制器被设计为薄控制器；薄控制器几乎没有业务逻辑，主要用于驱动应用程序请求。基本的 Magento 控制器动作只是加载和渲染布局：

```php
    public function viewAction()
    {
        $this->loadLayout();
        $this->renderLayout();
    }
```

从这里开始，块的工作是处理显示逻辑，从我们的模型中获取数据，准备数据，并将其发送到视图。

# 网站和商店范围

Magento 的一个核心特性是能够使用单个 Magento 安装处理多个网站和商店；在内部，Magento 将这些实例称为范围。

![网站和商店范围](img/3060OS_02_06.jpg)

某些元素的值，如产品、类别、属性和配置，是特定范围的，并且在不同的范围上可能不同；这使得 Magento 具有极大的灵活性，例如，一个产品可以在两个不同的网站上设置不同的价格，但仍然可以共享其余的属性配置。

作为开发人员，我们在使用范围最多的领域之一是在处理配置时。Magento 中可用的不同配置范围包括：

+   **全局**：顾名思义，这适用于所有范围。

+   **网站**：这些由域名定义，由一个或多个商店组成。网站可以设置共享客户数据或完全隔离。

+   **商店**：商店用于管理产品和类别，并分组商店视图。商店还有一个根类别，允许我们每个商店有单独的目录。

+   **商店视图**：通过使用商店视图，我们可以在商店前端设置多种语言。

Magento 中的配置选项可以在三个范围（全局、网站和商店视图）上存储值；默认情况下，所有值都设置在全局范围上。通过在我们的模块上使用`system.xml`，我们可以指定配置选项可以设置的范围；让我们重新审视一下我们之前的`system.xml`：

```php
…
<field_name translate="label comment">
    <label>Enabled</label>
    <comment>
         <![CDATA[Comments can contain <strong>HTML</strong>]]>
     </comment>
     <frontend_type>select</frontend_type>
     <source_model>adminhtml/system_config_source_yesno</source_model>
     <sort_order>10</sort_order>
     <show_in_default>1</show_in_default>
     <show_in_website>1</show_in_website>
     <show_in_store>1</show_in_store>
</field_name>
…
```

# 工厂名称和函数

Magento 使用工厂方法来实例化`Model`、`Helper`和`Block`类。工厂方法是一种设计模式，允许我们实例化一个对象而不使用确切的类名，而是使用类别名。

Magento 实现了几种工厂方法，如下所示：

+   `Mage::getModel()`

+   `Mage::getResourceModel()`

+   `Mage::helper()`

+   `Mage::getSingleton()`

+   `Mage::getResourceSingleton()`

+   `Mage::getResourceHelper()`

这些方法中的每一个都需要一个类别名，用于确定我们要实例化的对象的真实类名；例如，如果我们想要实例化一个`product`对象，可以通过调用`getModel()`方法来实现：

```php
$product = Mage::getModel('catalog/product'); 
```

请注意，我们正在传递一个由`group_classname/model_name`组成的工厂名称；Magento 将解析这个工厂名称为`Mage_Catalog_Model_Product`的实际类名。让我们更仔细地看看`getModel()`的内部工作：

```php
public static function getModel($modelClass = '', $arguments = array())
    {
        return self::getConfig()->getModelInstance($modelClass, $arguments);
    }

getModel calls the getModelInstance from the Mage_Core_Model_Config class.

public function getModelInstance($modelClass='', $constructArguments=array())
{
    $className = $this->getModelClassName($modelClass);
    if (class_exists($className)) {
        Varien_Profiler::start('CORE::create_object_of::'.$className);
        $obj = new $className($constructArguments);
        Varien_Profiler::stop('CORE::create_object_of::'.$className);
        return $obj;
    } else {
        return false;
    }
}
```

`getModelInstance()`又调用`getModelClassName()`方法，该方法以我们的类别名作为参数。然后它尝试验证返回的类是否存在，如果类存在，它将创建该类的一个新实例并返回给我们的`getModel()`方法：

```php
public function getModelClassName($modelClass)
{
    $modelClass = trim($modelClass);
    if (strpos($modelClass, '/')===false) {
        return $modelClass;
    }
    return $this->getGroupedClassName('model', $modelClass);
}
```

`getModelClassName()`调用`getGroupedClassName()`方法，实际上负责返回我们模型的真实类名。

`getGroupedClassName()`接受两个参数 - `$groupType`和`$classId`；`$groupType`指的是我们正在尝试实例化的对象类型（目前只支持模型、块和助手），`$classId`是我们正在尝试实例化的对象。

```php
public function getGroupedClassName($groupType, $classId, $groupRootNode=null)
{
    if (empty($groupRootNode)) {
        $groupRootNode = 'global/'.$groupType.'s';
    }
    $classArr = explode('/', trim($classId));
    $group = $classArr[0];
    $class = !empty($classArr[1]) ? $classArr[1] : null;

    if (isset($this->_classNameCache[$groupRootNode][$group][$class])) {
        return $this->_classNameCache[$groupRootNode][$group][$class];
    }
    $config = $this->_xml->global->{$groupType.'s'}->{$group};
    $className = null;
    if (isset($config->rewrite->$class)) {
        $className = (string)$config->rewrite->$class;
    } else {
        if ($config->deprecatedNode) {
            $deprecatedNode = $config->deprecatedNode;
            $configOld = $this->_xml->global->{$groupType.'s'}->$deprecatedNode;
            if (isset($configOld->rewrite->$class)) {
                $className = (string) $configOld->rewrite->$class;
            }
        }
    }
    if (empty($className)) {
        if (!empty($config)) {
            $className = $config->getClassName();
        }
        if (empty($className)) {
            $className = 'mage_'.$group.'_'.$groupType;
        }
        if (!empty($class)) {
            $className .= '_'.$class;
        }
        $className = uc_words($className);
    }
    $this->_classNameCache[$groupRootNode][$group][$class] = $className;
    return $className;
}
```

正如我们所看到的，`getGroupedClassName()`实际上正在做所有的工作；它抓取我们的类别名`catalog`/`product`，并通过在斜杠字符上分割字符串来创建一个数组。

然后，它加载一个`VarienSimplexml_Element`的实例，并传递我们数组中的第一个值（`group_classname`）。它还会检查类是否已被重写，如果是，我们将使用相应的组名。

Magento 还使用了`uc_words()`函数的自定义版本，如果需要，它将大写类别名的第一个字母并转换分隔符。

最后，该函数将返回真实的类名给`getModelInstance()`函数；在我们的例子中，它将返回`Mage_Catalog_Model_Product`。

![工厂名称和函数](img/3060OS_02_04.jpg)

# 事件和观察者

事件和观察者模式可能是 Magento 更有趣的特性之一，因为它允许开发人员在应用程序流的关键部分扩展 Magento。

为了提供更多的灵活性并促进不同模块之间的交互，Magento 实现了事件/观察者模式；这种模式允许模块之间松散耦合。

这个系统有两个部分 - 一个是带有对象和事件信息的事件分发，另一个是监听特定事件的观察者。

![事件和观察者](img/3060OS_02_05.jpg)

## 事件分发

使用`Mage::dispatchEvent()`函数创建或分派事件。核心团队已经在核心的关键部分创建了几个事件。例如，模型抽象类`Mage_Core_Model_Abstract`在每次保存模型时调用两个受保护的函数——`_beforeSave()`和`_afterSave()`；在这些方法中，每个方法都会触发两个事件：

```php
protected function _beforeSave()
{
    if (!$this->getId()) {
        $this->isObjectNew(true);
    }
    Mage::dispatchEvent('model_save_before', array('object'=>$this));
    Mage::dispatchEvent($this->_eventPrefix.'_save_before', $this->_getEventData());
    return $this;
}

protected function _afterSave()
{
    $this->cleanModelCache();
    Mage::dispatchEvent('model_save_after', array('object'=>$this));
    Mage::dispatchEvent($this->_eventPrefix.'_save_after', $this->_getEventData());
    return $this;
}
```

每个函数都会触发一个通用的`mode_save_after`事件，然后根据正在保存的对象类型生成一个动态版本。这为我们通过观察者操作对象提供了广泛的可能性。

`Mage::dispatchEvent()`方法接受两个参数：第一个是事件名称，第二个是观察者接收的数据数组。我们可以在这个数组中传递值或对象。如果我们想要操作对象，这将非常方便。

为了理解事件系统的细节，让我们来看一下`dispatchEvent()`方法：

```php
public static function dispatchEvent($name, array $data = array())
{
    $result = self::app()->dispatchEvent($name, $data);
    return $result;
}
```

这个函数实际上是位于`Mage_Core_Model_App`中的`app`核心类内部的`dispatchEvent()`函数的别名：

```php
public function dispatchEvent($eventName, $args)
{
    foreach ($this->_events as $area=>$events) {
        if (!isset($events[$eventName])) {
            $eventConfig = $this->getConfig()->getEventConfig($area, $eventName);
            if (!$eventConfig) {
                $this->_events[$area][$eventName] = false;
                continue;
            }
            $observers = array();
            foreach ($eventConfig->observers->children() as $obsName=>$obsConfig) {
                $observers[$obsName] = array(
                    'type'  => (string)$obsConfig->type,
                    'model' => $obsConfig->class ? (string)$obsConfig->class : $obsConfig->getClassName(),
                    'method'=> (string)$obsConfig->method,
                    'args'  => (array)$obsConfig->args,
                );
            }
            $events[$eventName]['observers'] = $observers;
            $this->_events[$area][$eventName]['observers'] = $observers;
        }
        if (false===$events[$eventName]) {
            continue;
        } else {
            $event = new Varien_Event($args);
            $event->setName($eventName);
            $observer = new Varien_Event_Observer();
        }

        foreach ($events[$eventName]['observers'] as $obsName=>$obs) {
            $observer->setData(array('event'=>$event));
            Varien_Profiler::start('OBSERVER: '.$obsName);
            switch ($obs['type']) {
                case 'disabled':
                    break;
                case 'object':
                case 'model':
                    $method = $obs['method'];
                    $observer->addData($args);
                    $object = Mage::getModel($obs['model']);
                    $this->_callObserverMethod($object, $method, $observer);
                    break;
                default:
                    $method = $obs['method'];
                    $observer->addData($args);
                    $object = Mage::getSingleton($obs['model']);
                    $this->_callObserverMethod($object, $method, $observer);
                    break;
            }
            Varien_Profiler::stop('OBSERVER: '.$obsName);
        }
    }
    return $this;
}
```

`dispatchEvent()`方法实际上是在事件/观察者模型上进行所有工作的：

1.  它获取 Magento 配置对象。

1.  它遍历观察者节点的子节点，检查定义的观察者是否正在监听当前事件。

1.  对于每个可用的观察者，分派事件将尝试实例化观察者对象。

1.  最后，Magento 将尝试调用与特定事件相映射的相应观察者函数。

## 观察者绑定

现在，分派事件是方程式的唯一部分。我们还需要告诉 Magento 哪个观察者正在监听每个事件。毫不奇怪，观察者是通过`config.xml`指定的。正如我们之前所看到的，`dispatchEvent()`函数会查询配置对象以获取可用的观察者。让我们来看一个示例`config.xml`文件：

```php
<events>
    <event_name>
        <observers>
            <observer_identifier>
                <class>module_name/observer</class>
                <method>function_name</method>
            </observer_identifier>
        </observers>
    </event_name>
</events>
```

`event`节点可以在每个配置部分（admin、global、frontend 等）中指定，并且我们可以指定多个`event_name`子节点；`event_name`必须与`dispatchEvent()`函数中使用的事件名称匹配。

在每个`event_name`节点内，我们有一个单一的观察者节点，可以包含多个观察者，每个观察者都有一个唯一的标识符。

观察者节点有两个属性，如`<class>`，指向我们的观察者模型类，和`<method>`，依次指向观察者类内部的实际方法。让我们分析一个示例观察者类定义：

```php
class Namespace_Modulename_Model_Observer
{
    public function methodName(Varien_Event_Observer $observer)
    {
        //some code
    }
}  
```

### 注意

关于观察者模型的一个有趣的事情是，它们不继承任何其他 Magento 类。

# 摘要

在本章中，我们涵盖了许多关于 Magento 的重要和基本主题，如其架构、文件夹结构、路由系统、MVC 模式、事件和观察者以及配置范围。

虽然乍一看可能会让人感到不知所措，但这只是冰山一角。关于每个主题和 Magento，还有很多值得学习的地方。本章的目的是让开发人员了解从配置对象到事件/对象模式的实现方式的所有重要组件。

Magento 是一个强大而灵活的系统，它远不止是一个电子商务平台。核心团队在使 Magento 成为一个强大的框架方面付出了很多努力。

在后面的章节中，我们不仅会更详细地回顾所有这些概念，还会通过构建我们自己的扩展来实际应用它们。
