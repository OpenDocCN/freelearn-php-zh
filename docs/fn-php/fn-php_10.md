# 第十章 PHP 框架和 FP

现在我们已经看到了功能编程如何可以用来解决常见的编程问题，是时候在使用框架开发时应用这些技术了。本章将介绍使用一些最常见的 PHP 框架的各种方法。

在我们开始之前，有一个小免责声明。我绝对不是我们将在这里讨论的每个框架的专家。我在不同层面上都与它们一起工作过，但这并不意味着我知道有关它们的一切。因此，尽管在撰写本章时进行了研究，但我可能不会呈现最新的最佳实践。

话虽如此，在本章中我们不会写很多代码。我们主要会看看如何将现有的功能代码与框架结构进行接口化，以及如何利用各种框架功能来帮助您以功能方式编写代码。我们还将讨论每个框架在功能编程方面的利弊。

在本章中，我们将看看以下框架：

+   Symfony

+   Laravel

+   Drupal

+   WordPress

我听到一些人在背景中低声说 Drupal 和 WordPress 不是框架，而是内容管理系统。我同意你的观点，但请记住，人们正在使用它们来创建具有电子商务和其他功能的完整应用程序，因此它们在这里有其位置。

此外，**CodeIgniter**框架没有出现在列表中，因为我从未使用过它。但是，您可能可以使用将在这里介绍的大部分建议与任何框架一起使用，包括 CodeIgniter。

实际上，每个部分中的大部分建议在各种情境中都是有用的。这就是为什么我强烈建议您阅读关于每个框架的部分。这将使我避免重复太多。

# Symfony

专注于依赖注入，Symfony 框架非常适合编写功能代码。Symfony 开发人员习惯于以一种明确定义其依赖关系的方式声明其控制器和服务。

我们可以争论整个容器的注入有点问题。在严格意义上，控制器和服务仍然可以是纯粹的，但显然认知负担会更重一些，因为您需要阅读代码才能确切知道使用了哪些依赖。

在本部分中，我们将讨论 Symfony 的哪些部分非常适合功能编程，以及您需要注意什么。我们无法覆盖所有内容，因为 Symfony 是一个非常完整的框架，具有��多组件，但它应该足以让您入门。

## 处理请求

原始的`Request`类不符合我们已经讨论过的`PSR-7 HTTP`消息接口。这意味着它不是不可变的，正如规范所建议的那样。但是，如果您使用的是`SensioFrameworkExtraBundle`框架的至少 3.0.7 版本，那么很容易获得请求的 PSR 版本。您只需要使用 Composer 安装所需的依赖项，并稍微更改您的控制器操作签名：

```php
**composer require sensio/framework-extra-bundle 
composer require symfony/psr-http-message-bridge 
composer require zendframework/zend-diactoros**

```

您的控制器需要在其方法签名中使用新的`ServerRequestInterface`类，而不是更传统的`Request`类：

```php
<?php 

namespace AppBundle\Controller; 

use Psr\Http\Message\ServerRequestInterface; 
use Symfony\Bundle\FrameworkBundle\Controller\Controller; 
use Zend\Diactoros\Response; 

class DefaultController extends Controller 
{ 
    public function indexAction(ServerRequestInterface $request) 
    { 
        return new Response(); 
    } 
} 
```

如果出于任何原因，您需要获取与 Symfony 接口兼容的`Request`或`Response`实例，您可以使用**Symfony PSR-7**桥接器。

如果您在控制器和服务中正确注入了您需要的依赖项，并且使用了新的符合 PSR-7 标准的 HTTP 消息，那么您现在已经准备好为 Symfony 应用程序编写功能代码了。

## 数据库实体

当编写完全纯函数代码时，您可能会遇到的一个挑战是数据库访问。通常，开发人员在 Symfony 中使用**Doctrine**，据我所知，Doctrine 目前还没有任何设施来帮助编写引用透明的代码。

在框架的上下文中使用像 IO 单子这样的东西也很麻烦，因为在函数的每个入口和出口点，你都必须封装参数或将结果转换为框架期望的格式。

我们将尝试看看如何使用各种技术来减轻这个问题。在此过程中，我们还将学习如何在使用 Doctrine 时利用`Maybe`类型。

### 可嵌入对象

虽然与函数式编程没有严格相关，但价值对象的概念可以用来实现实体的某种不可变性。这也是一个值得探索的想法，因为它本身也有一些好处。

这是我们在第二章中已经讨论过的想法，*纯函数、引用透明度和不可变性*。然而，我将借此机会给出一个有些不同的定义，这个定义来自领域驱动设计：

+   **实体**：具有与其属性无关的身份的东西。

+   **价值对象**：没有与其属性分开的身份的东西。

一个常见的例子是一个人有一个地址。人是一个实体，地址是一个值对象。如果你改变一个人的姓名、地址或任何其他属性，它仍然是同一个人。然而，如果你改变地址上的任何东西，它就成了完全不同的地址。

Doctrine 在**可嵌入对象**的名称下实现了这个想法。这个术语来自于价值对象总是与实体相关联或嵌入的事实，因为它本身没有存在的意义。你可以在官方网站上找到文档[`docs.doctrine-project.org/en/latest/tutorials/embeddables.html`](http://docs.doctrine-project.org/en/latest/tutorials/embeddables.html)。

虽然我不建议利用这个特性，来为你拥有的每个关系实现不可变性，但我强烈建议你在设计实体时考虑可嵌入对象，并在可以使用时使用它们。这将帮助你在功能上编码和提高数据模型的质量。

### 避免使用 setter

如果你开始寻找关于使用 Doctrine 和大多数 ORM 的最佳实践，很有可能有一天你会发现有人建议我们避免创建 setter 方法。通常会有许多很好的理由这样做。在我们的情况下，我们只会集中在其中一个上——我们希望不可变的实体帮助我们编写纯函数式代码。

在大多数情况下，摆脱 setter 的建议解决方案将是以任务为思考方式。例如，不是在`BlogPost`类上有一个`setState`和一个`setPublicationDate`方法，而是有一个`publish`方法，该方法将依次更改这两个字段。

这是一个很好的建议，因为它允许你将大部分业务逻辑放在实体内，避免因为开发人员没有采取所有必要的步骤而使对象处于一种奇怪的状态。一个传统的具有 setter 的类可能是以下的样子：

```php
<?php 

class BlogPost 
{ 
    private $status; 
    private $publicationDate; 

    public function setStatus(string $s) 
    { 
        $this->status = $s; 
    } 

    public function setPublicationDate(DateTime $d) 
    { 
        $this->publicationDate = $d; 
    } 
} 
```

它可以转换为以下实现：

```php
<?php 

class BlogPost2 
{ 
    private $status; 
    private $publicationDate; 

    public function publish(DateTime $d) 
    { 
        $this->status = 'published'; 
        $this->publicationDate = $d; 
    } 
} 
```

正如你所看到的，我们在原地修改值，留下了副作用。我们可能天真地认为，在`publish`方法中克隆当前对象并返回具有修改属性的新版本就足以获得我们方法的不可变版本；遗憾的是，这种解决方案并不起作用。

Doctrine 存储了哪些实体由其工作单元管理，克隆的实体不处于受控状态。我们可以使用一些技巧附加实体，但然后我们将处于以下两种情况之一：

+   两个实体都是受控的，这可能会导致 Doctrine 内部元数据的一致性问题

+   只有最新的实体是受控的，这意味着我们对`publish`方法的调用具有将先前的实体从 Doctrine 中分离的副作用

这是一个问题的关键在于目前没有可用的 API 来在实体内部执行此操作。这就是为什么我不建议在写作时使用当前 Doctrine 版本（即版本 2.5.5）追求不可变实体。

无论如何，避免在实体上创建 setter 已经是朝着引用透明代码库的方向迈出了一大步。这也将帮助您将所有业务逻辑集中在一个地方，而不会出现实体处于无效状态的可能性。

### 为什么要使用不可变实体？

不用多说，让我们用一个简单的例子来演示。Doctrine 在与日期和时间相关的任何事务中使用`DateTime`类的实例。由于`DateTime`类是可变的，这可能会导致非常难以准确定位的问题。

```php
<?php 

$date = $post->getPublicationDate(); 

// for any reason you modify the date 
$date->modify('+14 days'); 

var_dump($post->getPublicationDate() == $date); 
// bool(true) 

$entityManager->persist($post); 
$entityManager->flush(); 
// nothing changes in the database :( 
```

第一个问题是你在实体内部存储了对同一对象的引用。这意味着如果出于任何原因你对它进行了更改，日期也会在帖子内部发生变化。这可能是你想要的，但毫无疑问这是一个副作用。特别是如果你将`$date`变量返回给潜在的调用者。它怎么知道修改日期会导致修改实体呢？

第二个问题更加棘手。由于 Doctrine 使用对象标识而不是其值来确定是否发生了变化，它将不知道日期现在已经不同，将帖子保存回数据库将毫无意义。

GitHub 上有一个可用的包（[`github.com/VasekPurchart/Doctrine-Date-Time-Immutable-Types`](https://github.com/VasekPurchart/Doctrine-Date-Time-Immutable-Types)）来解决这个特定问题，但是，任何时候你使用可变实例而不是可嵌入的或任何其他类型的值对象，你都可能遇到类似的问题。请尽量使用不可变性。

### Symfony ParamConverter

我们已经讨论了修改已经实例化的实体并将其持久化到数据库中。但是首先如何获取它们呢？`SensioFrameworkExtraBundle`框架包含一个名为`@ParamConverter`的很好的小注解，它允许我们让框架来完成这项工作，并将从数据库获取实体的副作用放在我们的代码库之外。

这里有一个小例子，以便你了解如何使用这个注解（如果你想了解更多，你可以在 Symfony 网站的官方文档中阅读：[`symfony.com/doc/current/bundles/SensioFrameworkExtraBundle/annotations/converters.html`](http://symfony.com/doc/current/bundles/SensioFrameworkExtraBundle/annotations/converters.html)）：

```php
<?php 

use Sensio\Bundle\FrameworkExtraBundle\Configuration\ParamConverter; 

class PostController extends Controller 
{ 
    /** 
     * @Route("/blog/{id}") 
     * @ParamConverter("post", class="SensioBlogBundle:Post") 
     */ 
    public function showAction(Post $post) 
    { 
        // do something here 
    } 
} 
```

使用路由中的信息以及定义的参数转换，框架能够直接给你`Post`实例，或者在找不到时生成`404 错误`。

使用注解，你的方法不需要再执行数据库访问，因为它将直接接收数据。我们可能会争论说不纯的代码存在于其他地方，这是正确的，但 Symfony 本来就不应该是一个纯净的代码库。我们已经将不纯度从我们自己的代码中挤出去了，这对我们来说是最重要的。

在这种特殊情况下，类型提示已经足够与路由相关。`ParamConverter`注解将在函数签名引用实体类时自动进入操作。如果你觉得更清晰，保留这个注解也没有坏处，或者你可以决定只在更复杂的情况下使用它。

显然会有情况下这种机制不够强大。我知道一些包提供了类似的功能，更加灵活；你可能能够找到一个适合你需求的包。如果其他方法都不起作用，你仍然可以自己执行查询，或者使用 IO 单子来代替你执行查询。

### 也许有一个实体

Doctrine 可以很容易地适应返回 Collection 和 Maybe 单子的实例。第一步是创建一个新的存储库：

```php
<?php 

use Widmogrod\Monad\Maybe as m; 
use Widmogrod\Monad\Collection; 

class FunctionalEntityRepository extends EntityRepository 
{ 
    public function find($id, $lockMode = null, $lockVersion =  null) 
    { 
        return m\maybeNull(parent::find($id, $lockMode,  $lockVersion)); 
    } 

    public function findOneBy(array $criteria, array $orderBy =  null) 
    { 
        return m\maybeNull(parent::findOneBy($criteria,  $orderBy)); 
    } 

    public function findBy(array $criteria, array $orderBy = null,  $limit = null, $offset = null) 
    { 
        return Collection::of(parent::findBy($criteria, $orderBy,  $limit, $offset)); 
    } 

    public function findAll() 
    { 
        return Collection::of(parent::findAll()); 
    } 
} 
```

然后，您需要配置 Symfony 以使用这个新类作为默认存储库；这可以通过将以下键添加到您的 YAML 配置文件中轻松完成：

```php
doctrine:   
  orm:     
    entity_managers:       
      default_em:         
        default_repository_class: MyBundly\MyNamespace\FunctionalEntityRepository 
```

如果您不使用 Symfony，可以在 Doctrine 的`Configuration`类实例上使用`setDefaultRepositoryClassName`方法来实现相同的效果。

根据我们讨论的有关 Doctrine 的一切，当涉及到数据库时，您将无法拥有纯粹的功能性代码，但您已经准备好获得大部分的好处。

## 组织您的业务逻辑

官方 Symfony 最佳实践包含一些建议，指导如何以及在哪里编写业务逻辑。我们将对其进行扩展，以便更容易编写功能性代码。

第一个建议是避免在与框架本身相关的部分（路由和控制器）中编写任何逻辑。这些部分应尽可能简单明了。遵循这个建议是个好主意，因为这样做的话，如果您被迫在控制器中进行一些数据库访问，也不会那么重要。

我建议您将所有与数据库相关的操作都放在控制器内部，这样具有副作用的内容就会被隔离在那里，您的业务逻辑可以遵循适当的功能性技术。

此外，您应该只在控制器和服务中注入您需要的依赖项，而不是使用服务容器。这将大大减轻认知负担，因为您的方法和构造函数的签名足以确定业务逻辑的依赖关系。

我还建议您避免使用 setter 注入，因为调用 setter 将修改服务的状态，从而破坏不可变性，并且如果多次调用 setter 可能会导致潜在问题。

通过决定将副作用限制在控制器中，您可以集中精力在实体和服务中编写功能性代码。这将使实体和服务易于理解和测试。然后，由于控制器不应包含自己的逻辑，您可以使用集成和功能测试来测试最终部分，并迅速对应用程序获得信心。

## Flash 消息、会话和其他具有副作用的 API

Flash 消息通常用于以不显眼的方式向用户传达信息。Symfony 提出的管理 API 遗憾地不是引用透明的，因为您需要在控制器上调用一个方法来将消息添加到队列中。会话数据管理也是如此。

这个问题可以通过在`Response`对象内部以某种方式集成它们来解决。然而，这需要在框架级别完成。这些更改要么需要上游整合，要么需要大量的维护。

一种解决方案是在服务中利用 Writer 或 State 单子来保存信息，然后在控制器中持久化它们，因为我们已经决定在控制器中处理与数据库相关的副作用。

然而，我不建议使用 IO 单子，因为在没有框架级别的支持的情况下，它将变得复杂，特别是因为 PHP 缺乏 Haskell 中可用的*do notation*注释的良好替代方案。这只会使您的代码变得更加复杂，而没有真正的好处。

`Form` API 是另一个具有大量内部状态和不纯方法的实例。然而，它足够声明性，以至于这个事实不会带来太多问题。您可以将表单创建抽象成自己的类，这也有助于考虑它是无副作用的。

我强烈建议您在可能的情况下创建`Form`类型，并尽可能将生成的`Form`实例视为不可变对象。

## 结束语

尽管 Symfony 在面向对象设计方面有着很强的重点，但它为我们提供了一个很好的基础来编写函数式代码。特别是在涉及数据库访问和框架提供的非函数式 API 时，需要做出一些让步，但您自己编写的代码基本上可以从头到尾都是函数式的，除了控制器本身。

使用函数式方法的一个缺点可能是您发现自己创建了更多的服务和类，以将所有副作用隔离在请求生命周期的一个单独部分中。

然而，这样做的好处是拥有一个明确定义的代码库，甚至可以在 Symfony 之外得到重复使用，如果得到适当的关注。您还将能够更容易地分开测试每个部分。

在您的应用程序的某些部分和服务中逐渐应用函数式技术也没有任何阻碍。通过能够慢慢地将您的代码迁移到更加函数式的东西，您能够立即应用这些技术，这也有助于其他人的学习曲线。您可以使用以下资源更好地理解函数式技术的应用：

+   [`vimeo.com/177154259`](https://vimeo.com/177154259)

+   [`www.slideshare.net/boerdedavid/being-functional-in-php`](http://www.slideshare.net/boerdedavid/being-functional-in-php)

# Laravel

正如我们已经讨论过的，Laravel 的`collection` API 是一个很好的不可变数据结构的例子，它在其顶部具有很好的函数式方法。返回`Collection`类实例的数据库层真的有助于简化其使用。

然而，如果您想保持函数的纯净，框架提出的**Facade**模式是不可取的。一旦您使用一个 Facade，您就会使用一个未在函数签名中声明的外部依赖。

无论您对这种模式的看法如何，如果您想编写引用透明的代码，您必须摆脱它们。幸运的是，Laravel 为大多数常见任务提供了辅助函数和访问容器的方法。正如我们将看到的那样，因此使用不同的东西并不那么困难。

由于 Laravel 也是一个基于面向对象原则和 MVC 模式的框架，Symfony 部分的所有一般建议也适用，特别是关于解耦各个部分并尝试将副作用隔离在每个请求的一个唯一位置的建议，通常是控制器。

实现这一点的方式可能有所不同，但并不会有太大差异；这就是为什么我鼓励您阅读前一节，如果您还没有这样做的话，因为那些建议将不会在这里重复。

## 数据库结果

正如已经提到的，Laravel 有一个非常好的不可变集合实现。返回多个实体的所有数据库查询都将返回它的一个实例。有一本非常好的书详细介绍了您可以利用其功能的所有方法。您可以在作者的网站上找到它，以及相关的视频教程和其他教程，网址为[`adamwathan.me/refactoring-to-collections/`](https://adamwathan.me/refactoring-to-collections/)。集合可能是不可变的，但其中的对象不是。由于 Laravel 的 ORM，Eloquent，与 Doctrine 非常不同，因此我们有可能使它们成为不可变的。您只需在一个`Model`类上扩展方法，而不是使用`Repository`模式和`UnitOfWork`模式，这意味着可以以一种使实体不可变的方式实现您的方法，而不会遇到有关 Doctrine 内部状态的问题：

```php
<?php 

use Illuminate\Database\Eloquent\Model; 

class BlogPost extends Model 
{ 
    private $status; 
    private $publicationDate; 

    public function publish(DateTime $d) 
    { 
        $new = clone $this; 

        $new->status = 'published'; 
        $new->publicationDate = $d; 
        return $new; 
    } 
} 
```

只要不修改用于主键的字段，对对象的`save`方法的任何调用都将更新数据库中的当前行。但是，如果您的对象具有非标量属性，则可能需要在对象上添加`__clone`方法。 PHP 默认只执行浅复制，这意味着所有引用将保持不变。这可能是您想要的，但您需要确保它。

如果您想强制某些属性的不可变性，GitHub 上有一个可用的软件包([`github.com/davidmpeace/immutability`](https://github.com/davidmpeace/immutability))可以做到这一点。如果您严格遵守规定，则不需要，但在传统和功能部分都存在的遗留代码库中，这可能是一个不错的功能。

### 使用 Maybe

与 Doctrine 一样，也可以返回`Maybe`的实例，而不是`null`。由于结构有些不同，我们首先需要创建一个新的`Builder`类：

```php
<?php 

use Illuminate\Database\Eloquent\Builder as BaseBuilder; 
use Widmogrod\Monad\Maybe as m; 

class FunctionalBuilder extends BaseBuilder 
{ 
    public function first($columns = array('*')) 
    { 
        return m\maybeNull(parent::first($columns)); 
    } 

    public function firstOrFail($columns = array('*')) 
    { 
        return $this->first($columns)->orElse(function() { 
            throw (new ModelNotFoundException)- >setModel(get_class($this->model)); 
        }); 
    } 

    public function findOrFail($id, $columns = array('*')) 
    { 
        return $this->find($id, $columns)->orElse(function() { 
            throw (new ModelNotFoundException)- >setModel(get_class($this->model));         }); 
    } 

    public function pluck($column) 
    { 
        return $this->first([$column])->map(function($result) { 
            return $result->{$column}; 
        }); 
    } 
} 
```

由于`first`函数的使用，多个方法被重新定义，该函数现在返回`Maybe`类型的实例，而不是`null`。但是，无需返回`Collection`的实例，因为 Laravel 版本已经是一个很好的实现，即使它不是一个 monad。

现在我们需要我们自己的`Model`类，它将使用我们的`FunctionalBuilder`方法：

```php
{ 
        return $this->find($id, $columns)->orElse(function() { 
            throw (new ModelNotFoundException)- >setModel(get_class($this->model)); 
        }); 
    } 

    public function pluck($column) 
    { 
        return $this->first([$column])->map(function($result) { 
            return $result->{$column}; 
        }); 
    } 
} 
```

这两个新类在大多数情况下应该能够正常工作，但由于我们正在重新定义框架本身使用的方法，可能会遇到问题。如果是这种情况，我很乐意听取您的意见，以便改进实现以避免问题。

您可能还希望修改 Collection monad，以便在适当的各种方法中返回 Maybe monad 的实例，而不是`null`。但是，这将需要比我们迄今为止所做的更多的修改。据我所知，目前没有提供此功能的软件包。

## 摆脱 facade

Facades 的概念可能对新手减少学习曲线有所帮助，并且可能有助于使用 Laravel 提供的服务。但无论您对它们的看法如何，它们都不是一点功能，因为一旦使用它们，就会引入副作用。

通过在控制器和服务中注入依赖项，可以很容易地摆脱它们，这是 Symfony 世界中的常见做法。除了允许您编写功能代码之外，停止使用 facade 还有一个隐藏的好处-您的代码将与 Laravel 的联系更少，因此您可能能够更多地重用它。

Laravel 提供了一个名为**自动注入**的功能，它将允许您通过 Facade 非常轻松地获取各种组件。它使用类型提示在类实例化时自动注入所需的依赖项。它在多种上下文中都有效-例如控制器、事件监听器和中间件。

获取`UserRepository`类的实例就像下面这样简单：

```php
<?php 

namespace App\Http\Controllers; 

use App\Users\Repository as UserRepository; 

class UserController extends Controller 
{ 
    protected $users; 

    public function __construct(UserRepository $users) 
    { 
        $this->users = $users; 
    } 
} 
```

可以通过参考文档中提供的表格轻松找到要使用的类型提示[`laravel.com/docs/5.3/facades#facade-class-reference`](https://laravel.com/docs/5.3/facades#facade-class-reference)。

然而，这个巧妙的机制并没有真正将您与框架解耦，因为您需要使用正确的类型提示。通过项目的`bootstrap/start.php`手动注入依赖项的另一种方法在这篇文章中有描述[`programmingarehard.com/2014/01/11/stop-using-facades.html/`](http://programmingarehard.com/2014/01/11/stop-using-facades.html/)。

## HTTP 请求

与 Symfony 一样，使用 PSR-7 中定义的接口而不是框架中的接口非常容易。 Laravel 使用与 Symfony 相同的桥梁执行转换。您只需要使用 Composer 安装两个软件包：

```php
**composer require symfony/psr-http-message-bridge 
composer require zendframework/zend-diactoros**

```

然后，当你想要获取 PSR-7 版本的实例时，只需使用`ServerRequestInterface`类作为类型提示，而不是`Request`方法。如果你的控制器动作返回 PSR-7 版本，Laravel 会自行将`Response`方法转换为自己的格式。

## 结束语

Laravel 核心开发人员所做的实现决策有两面性。其中一些，比如不可变集合实现，在功能性编程方面非常出色。而另一些，比如使用 Facades，会让我们的生活变得有些困难。

然而，将我们的代码转换为更加功能性的方法是相当简单的。你可能会遇到的唯一困难是阅读文档或教程时，它们通常描述我们试图避免的模式和实践。

总的来说，当涉及到编写功能性代码时，Laravel 和 Symfony 一样出色。前面提到的关于其集合实现的书籍也是学习如何使用一些功能性技术与集合单子实现相关的绝佳方式。据我所知，Symfony 没有这种资源。

# Drupal

Drupal 模块直到版本 7 都依赖于钩子来执行操作。Drupal 钩子是遵循特定命名模式的函数，当 Drupal 在响应请求时发生各种事件时，会调用这些函数来修改生成的网页的各个方面。

在理想的世界中，所有的钩子都会接收到执行工作所需的所有信息，并且修改某些东西的方式将使用返回值。这在某些模块 API 的部分是基本正确的。不幸的是，有一些函数会接收通过引用传递的参数，比如`hook_block_list_alter`函数。此外，有时你需要访问全局变量，例如获取当前语言。

Drupal 8 转向了基于类的方法。现在应该在控制器中创建内容，以便更接近 Symfony 的术语。原因是这个新版本现在使用了 Symfony 的一些核心组件。这并不意味着不再可能使用功能性编程，只是事情有些不同。

本书的角色并不是详细解释从版本 7 到版本 8 发生了什么变化。有很多教程在做这方面的出色工作。这里将呈现的大部分内容都是足够通用并且对 Drupal 的两个版本都适用的。

## 数据库访问

在 Drupal 7 中，你可以使用多个函数来执行数据库查询并访问结果。通常，你会从`db_query`函数开始，该函数返回一个带有各种方法来检查和处理数据的结果对象。Drupal 8 的首选方式不是在模块或服务中注入数据库连接并以更面向对象的方式使用它。

这种变化实际上并不影响我们，因为在这两种情况下，都不可能以引用透明的方式查询数据库。此外，人们通常不会在 Drupal 中使用 ORM；大多数对数据库的请求都是直接使用 SQL 进行的。

这就是为什么我们不会在这个主题上停留，除了重复强调尽可能隔离数据库访问，以便代码的其余部分可以是功能性的。

## 处理需要副作用的钩子

最小的 Drupal 模块由两个文件组成，`info`文件和`module`文件。`info`文件的格式从 Drupal 7 中的特定文本文件变为 Drupal 8 中的 YAML 文件，但文件仍然包含有关模块的信息。`module`文件是模块的主要 PHP 文件。

正如我们所看到的，一些钩子需要副作用来执行它们的工作，几乎没有其他方法。我可以建议的是，将模块文件作为保存所有非严格功能性代码的文件，并将所有计算放在其他地方。

在 Drupal 8 的情况下，一些控制器方法可能也需要具有副作用。在这种情况下，我会给您与 Laravel 和 Symfony 相同的建议：将这些保留在控制器中，并使用外部服务/助手来执行引用透明计算。

我们如何为我们之前讨论过的`hook_block_list_alter`函数做到这一点？首先，这仅适用于 Drupal 7，因为在下一个版本中，块是通过类管理的，这在这种特定情况下解决了引用透明性的问题。

我的建议只是在您的模块中创建第二个 PHP 文件，其中只包含纯函数。例如，这个文件可以包含一个`new_blocks`函数，它只接受当前块和语言作为参数。

然后，在模块文件中，您可以执行以下操作：

```php
function my_module_block_list_alter(&$blocks) { 
    global $language; 

    $blocks = new_blocks($blocks, $language); 
} 
```

这个函数显然既有副作用又有副作用；我们对此无能为力。然而，`new_blocks`函数可以是纯的，这意味着您可以轻松地对其进行推理和测试，就像我们在前几章中看到的那样。

这种方法几乎可以应用于任何事物。一旦您遇到副作用或副作用，就在模块文件中执行这些操作，然后使用不同的文件来保存您的纯函数，这些函数将进行必要的处理和计算。如果您使用 Drupal 8，可以使用控制器，而不是使用`module`文件，就像我们已经讨论过的 Symfony 和 Laravel 一样。

## 钩子顺序

Drupal 的美妙之处在于所有可用的各种模块。这是如此真实，以至于有些人提出了苹果营销口号的变体之一：*有一个模块可以做到！*。然而，这也带来了一些困难。然而，当涉及到质量时，并非所有模块都是平等的，通常您会为任何给定的应用程序得到一堆模块。

其推论是，您自己的钩子接收到的信息可能已经被之前的模块修改过。比如，您正在编写一个模块来重新排列页面上的一些块；很可能您期望存在的一些块已经被移除了。或者您在关联数组中使用的键已经被注册或将被覆盖。

这可能会导致一些问题，有点难以准确地确定。由于您的函数将是纯净的，因此相对容易检测到它来自其他地方的事实，方法是通过明确添加一个测试来确保它针对给定数据集按预期工作。

关于这个问题的一个很好的建议是，不要假设您从 Drupal 接收的任何内容中可能存在或不存在的内容。始终应用某种检查来确保您接收的数据结构正确并且存在。

## 结束语

出于历史原因，一些 Drupal 钩子需要具有副作用才能执行其职责。此外，并非所有信息都作为参数传递给它们，这要求我们访问全局范围以获取它们。这个事实要求我们找到解决方法，以保持尽可能多的纯净代码。

通过引入更多的面向对象的方法以及服务注入，Drupal 8 使事情变得更容易。与我们在 Symfony 或 Laravel 中的经验相当，但事情仍然不完美。

如果您在至少两个文件中严格区分您的不纯函数和纯函数，那么您编写功能代码的体验将非常好。如果您想要纯函数，那么实现一个钩子需要创建两个函数可能看起来很麻烦，但这是您必须付出的代价，而且在我看来，这是值得的。

正如我们讨论过的，即使您的纯代码经过了彻底测试，由于一些钩子的调用顺序，您仍然可能在最终页面呈现时遇到问题，但是如果您对函数有信心，这些问题通常更容易发现。

Drupal 的功能开发者体验并不完美，但它接近。您将不得不做一些让步，但您可以将它们绑定到少数文件中，以限制它们对代码其余部分的影响。

# WordPress

WordPress 也有一个钩子系统，尽管与 Drupal 的不同。您不是创建具有特定名称的函数，而是将函数注册到特定的钩子上。遗憾的是，根据定义，大多数这些钩子需要具有副作用。例如，我们可以通过`wp_footer`钩子来实现这一点：

> *此钩子不提供参数。您可以通过让您的函数向浏览器输出内容，或者让它执行后台任务来使用此钩子。您的函数不应返回，也不应该带任何参数。*

没有返回值，没有参数；我们被迫创建一个具有副作用的函数。这意味着您将不得不在您的代码周围创建包装函数，甚至比我们刚刚在 Drupal 中演示的更多。

幸运的是，WordPress 还允许您在插件中拥有多个文件。因此，建议是一样的-将所有不纯的代码放在主文件中。从全局上下文中获取您需要的所有信息，并在那里执行任何类型的具有副作用的操作。一旦您获得所需的一切，调用您的纯函数进行处理和计算。

一些 WordPress 教程将面向对象编程呈现为开发人员掌握更为程序化的插件编写方式后的下一个进化阶段。如果您打算使用功能技术，这并不重要。您可以仅使用函数组织您的代码，也可以将它们分组到类中。我建议您坚持您更熟悉的方法。

## 数据库访问

在 WordPress 插件中，基本上有两种访问数据库的方式。您可以直接使用`WP_Query`方法及其面向对象的接口。或者您可以使用辅助函数，如`get_posts`和`get_pages`。

您可能已经在某处听说或读到，当使用类编写插件时，最好使用`WP_Query`函数，而在使用函数时使用辅助函数。从功能的角度来看，这并不重要。它们都不是引用透明的。您可以使用您喜欢的或更适合您需求的任何一个。

在 WordPress 代码库中，关于数据库访问并没有太多可说的。问题与其他框架相同-目前没有纯粹的方法来执行它们。

话虽如此，建议仍然是一样的-尽量将任何具有副作用的代码与副作用隔离到插件的单个文件中，然后将计算和处理委托给其他地方的纯函数。

## 功能方法的好处

我在本部分的介绍中说过，无论您使用函数还是对象来组织您的代码都无关紧要。这只是部分正确。WordPress 缺乏像 Symfony 或 Laravel 等框架的所有注入功能。这意味着，如果您使用对象，您将遇到在各处共享实例的困难。

如果您的对象仅用于保存不使用任何内部状态的纯方法，那么这并不是问题，但正如我们所看到的，有时需要做出让步。如果您需要与此类状态共享实例，您唯一的解决方案是将其全局可用。这样的变量的问题在于它可能被重新分配给其他内容，从而在以后引起问题。

相反，函数可以从任何地方使用，您无法重新定义它。这会导致更健壮的代码，因为您限制了副作用的可能性。

## 结束语

Drupal 的第一个版本可以追溯到 2000 年，使其成为这里介绍的最古老的工具。WordPress 诞生于 2003 年。然而，Drupal 自那时起已经被重写，而 WordPress 的代码库大多是在没有完全重写的情况下进行扩展。

为什么我要告诉你这些？因为在尝试在 WordPress 中编写函数式代码时，你遇到的大部分问题都与其遗留代码库有关。2000 年编写软件的方式与我们现在期望的最佳实践有些不同。

WordPress 进行了大量的现代化工作，但你能做的也只有这么多。特别是当重点不是使框架对函数式开发者友好时。然而，如果你愿意跳过一些障碍来隔离具有副作用的部分，仍然可以编写函数式代码。

WordPress 主要基于钩子，大部分 API 由函数组成。其中一些是引用透明的，而另一些则完全不是。你需要一些严谨性来清晰地将它们与你的其余代码隔离开来。好处总是一样的：

+   减少认知负担

+   促进代码重用

+   更容易的测试

缺点是，你的主要插件文件将主要由一些非常小的函数组成，这些函数只是作为 WordPress API 的不纯函数的包装器，然后调用你的引用透明函数。

如果你的包装器的名称与它们的 WordPress 对应物足够接近，那么阅读你的代码并在其中导航对于任何熟悉 WordPress 的人来说应该是相当容易的。最终，尽可能多地编写函数式代码仍然是一个好主意。

# 总结

正如我们之前讨论过的，没有一个主流框架在其核心具有函数式方法。在本章中，我们试图看到我们学到的技术如何更多或更少地应用于一些可用的框架和 CMS。

正如我们所看到的，至少在某个层面上总是可以使用函数式编程。遗憾的是，根据框架的不同，你将不得不在某个时候创建非引用透明的代码。

正如我在介绍中所说的，我并不是本章讨论的所有库的专家，所以要持保留态度。更有经验的开发者可能会有不同的做法。然而，这些示例为任何想尝试函数式编程的人提供了一个很好的起点。

此外，当这样做时，请记住，这首先是一种思维方式。最重要的是你解决问题的方式。如果在某个时候，你需要创建非纯代码来适应外部依赖或你正在使用的框架，那就这样吧。这不会改变你编写的函数式代码所能获得的好处。

现在我们已经看到了如何在现有框架或遗留代码库中使用函数式编程，下一章将涵盖使用一种称为函数响应式编程或 FRP 的范式来设计整个应用程序。
