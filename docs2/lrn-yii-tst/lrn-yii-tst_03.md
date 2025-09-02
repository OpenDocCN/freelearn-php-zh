# 第三章。进入 Codeception

在上一章讨论的 Yii 2 安装之后，在这一章中，我们将介绍 Codeception 套件的安装（[`codeception.com`](http://codeception.com)）并遍历文件夹结构，描述 Codeception 是如何工作的，它的扩展，模块化，语法以及使用的术语。

我们需要对其概念和细节有一个很好的掌握，因为 Codeception 将成为我们在本书剩余部分与测试交互的主要工具。在这一章中，我们将涵盖以下主题：

+   开始使用 Codeception

+   在 Yii 2 中安装 Codeception

+   在 Codeception 中找到你的路径

+   与 Codeception 交互

### 注意

请记住，当 Yii 2 达到稳定版本时（可能是在本书发布之后），其文件夹结构可能会发生变化，以及用于组织测试的结构。始终尝试做笔记并理解你所看到的内容，因为 Codeception 的工作方式和与 Yii 的交互方式不会发生重大变化，如果不是改善的话。

# 开始使用 Codeception

并非每个人都接触过测试。真正接触过的人都知道他们所使用的测试工具的怪癖和限制。有些可能比其他更有效率，但在任何情况下，你都必须依赖于你所面对的情况：遗留代码、难以测试的架构、没有自动化、工具没有支持，以及其他设置问题，仅举几例。只有某些公司，因为它们拥有正确的技能集或预算，才会投资于测试，但大多数公司没有能力看到质量保证的重要性。在让开发者对自己的代码负责并进行测试之后，建立测试基础设施和工具是紧接着的下一步。

即使在编程世界中，测试并不是什么特别新的东西，PHP 在这方面一直有一个弱点。它的历史并不是一个纯种编程语言，没有所有那些细微的细节，而且 PHP 直到最近才找到了一个更好的位置，开始受到更多的重视。

因此，唯一且最重要的工具就是 PHPUnit，它是在 10 年前，2004 年发布的，归功于 Sebastian Bergmann 的努力。

PHPUnit 曾经是，有时至今仍然难以掌握和理解。它需要时间和投入，尤其是如果你是从非测试经验过来的。PHPUnit 仅仅提供了一个低级框架来实施单元测试，以及在一定范围内，集成测试，并在需要时能够创建模拟和伪造对象。

尽管它仍然是发现 bug 的最快方式，但鉴于我们在前几章中看到的限制，它并没有涵盖所有内容，使用它来创建大型集成测试最终将是一项几乎不可能完成的任务。

此外，从 3.7 版本开始，PHPUnit 切换到不同的自动加载机制，并远离了 PEAR，这导致了许多头痛的问题，使得大多数安装变得无法使用。

自那时以来开发的其他工具大多来自其他环境和需求，编程语言和框架。其中一些工具非常强大且构建良好，但它们带来了自己声明测试和与应用程序交互的方式，一套规则和配置细节。

## 一个模块化框架，而不仅仅是另一个工具

显然，掌握所有这些工具需要一定的理解，而且学习曲线并不保证在所有工具中都是相同的。

那么，如果这是当前的格局，为什么还要创建另一个工具，如果你最终会陷入我们之前所处的相同境地呢？

好吧，关于 Codeception 需要理解的最重要的事情之一是，它不仅仅是一个工具，而是一个完整的栈，正如 Codeception 网站所注明的，一套框架，或者如果你愿意更元地看待它，一个框架的框架。

Codeception 通过尽可能使用相同的语义和逻辑来设计不同类型的测试，从而提供了一种使整个测试基础设施更加一致和易于接近的方法。

## 阐述 Codeception 背后的概念

Codeception 的创建是基于以下基本概念：

+   **易于阅读**：通过使用接近自然语言的声明性语法，测试可以很容易地阅读和解释，这使得它们成为用作应用程序文档的理想候选者。任何接近项目的利益相关者和工程师都可以确保测试被正确编写并覆盖所需的场景，而无需了解任何特殊术语。它还可以从代码测试用例生成 BDD 风格的测试场景。

+   **易于编写**：正如我们之前强调的，每个测试框架都使用自己的语法或语言来编写测试，这导致在从一个套件切换到另一个套件时存在一定程度的难度，而没有考虑到每个套件的学习曲线。Codeception 试图通过使用一种常见的声明性语言来弥合这种知识差距。此外，抽象提供了一个舒适的环境，使得维护变得简单。

+   **易于调试**：Codeception 天生具有在不乱动配置文件或在你代码周围随机使用`print_r`的情况下查看幕后情况的能力。

在所有这些之上，Codeception 还考虑了模块化和可扩展性，这使得组织代码变得简单，同时也在测试中促进了代码的重用。

但让我们更详细地看看 Codeception 提供了什么。

## 测试类型

正如我们所见，Codeception 提供了三种基本的测试类型：

+   单元测试

+   功能测试

+   接受测试

每一个都包含在其自己的文件夹中，你可以在这里找到所需的一切，从配置和实际测试到任何有价值的信息，例如夹具、数据库快照或要提供给测试的特定数据。

为了开始编写测试，你需要初始化所有必要的类，这将允许你运行测试，你可以通过使用带有`build`参数的`codecept`来做到这一点：

```php
$ cd tests
$ ../vendor/bin/codecept build
Building Actor classes for suites: functional, acceptance, unit
\FunctionalTester includes modules: Filesystem, Yii2
FunctionalTester.php generated successfully. 61 methods added
\AcceptanceTester includes modules: PhpBrowser
AcceptanceTester.php generated successfully. 47 methods added
UnitTester includes modules:
UnitTester.php generated successfully. 0 methods added
$

```

### 注意

每次当你修改 Codeception 拥有的任何配置文件，添加或删除任何模块时，都需要运行`codecept build`命令，换句话说，每次你修改`/tests`文件夹中可用的`.suite.yml`文件中的任何内容时。

你可能已经注意到的前一个输出中存在一个非常独特的测试类命名系统。

Codeception 引入了被 Yii 术语重命名为**测试者**的**Guys**，如下所示：

+   `AcceptanceTester`：这用于验收测试

+   `FunctionalTester`：这用于功能测试

+   `UnitTester`：这用于单元测试

这些将成为你与（大多数）测试的主要交互点，我们将看到原因。通过使用这样的命名法，Codeception 将注意力从代码本身转移到那些将要*执行*你将要编写的测试的人。

这样我们就会更加熟练地以 BDD 思维模式思考，而不是试图找出所有可能被覆盖的解决方案，同时失去我们试图实现的目标的焦点。

再次强调，BDD 比 TDD 是一个改进，因为它以更详细的方式声明了需要测试的内容以及*不需要测试的内容*。

### AcceptanceTester

`AcceptanceTester`可以看作是一个对所使用的技术一无所知的人，试图验证最初定义的验收标准。

如果我们想要以前定义的验收测试以更标准化的 BDD 方式进行重写，我们需要记住所谓的*用户故事*的结构。故事应该有一个清晰的标题，一个简短介绍，指定参与获得一定*结果*或*效果*的*角色*，以及这将反映的*价值*。随后，我们需要指定各种场景或*验收标准*，这些通过概述初始*场景*、*触发事件*和*预期结果*在一个或多个子句中定义。

让我们讨论使用模态窗口进行登录，这是我们将在应用程序中实现的两个功能之一。

**故事标题 – 成功的用户登录**

我，作为一个验收测试者，希望从任何页面登录到应用程序。

+   **场景 1**：从主页登录

    1.  我在主页上。

    1.  我点击登录链接。

    1.  我输入我的用户名。

    1.  我输入我的密码。

    1.  我点击提交。

    1.  登录链接现在显示为“注销 (<用户名>)”，而我仍然在主页上。

+   **场景 2**：从次要页面登录

    1.  我在二级页面上。

    1.  我点击了登录链接。

    1.  我输入我的用户名。

    1.  我输入我的密码。

    1.  我按下提交按钮。

    1.  登录链接现在显示为“注销 (<用户名>)”，我仍然在二级页面上。

正如你可能注意到的，我正在将前面的例子限制在成功案例中。还有更多，我们将在本书进一步实现实际功能之前，更详细地讨论所有相关的故事和场景。

前面的故事可以立即翻译成以下代码：

```php
// SuccessfulLoginAcceptanceTest.php

$I = new AcceptanceTester($scenario);
$I->wantTo("login into the application from any page");

// scenario 1
$I->amOnPage("/");
$I->click("login");
$I->fillField("username", $username);
$I->fillField("password", $password);
$I->click("submit");
$I->canSee("logout (".$username.")");
$I->seeInCurrentUrl("/");

// scenario 2
$I->amOnPage("/");
$I->click("about");
$I->seeLink("login");
$I->click("login");
$I->fillField("username", $username);
$I->fillField("password", $password);
$I->click("submit");
$I->canSee("logout (".$username.")");
$I->amOnPage("about");
```

如你所见，这完全直截了当且易于阅读，以至于业务中的任何人都能编写任何案例场景（这是一个夸张的说法，但你应该明白这个意思）。

显然，我们需要理解的是`AcceptanceTester`能够做什么：由`codecept build`命令生成的类可以在`tests/codeception/acceptance/AcceptanceTester.php`中找到，其中包含所有可用方法。如果你需要了解如何断言特定条件或对页面执行操作，你可能想浏览一下。[`codeception.com/docs/04-AcceptanceTests`](http://codeception.com/docs/04-AcceptanceTests)上的在线文档也会以更易读的方式提供这些信息。

不要忘记，在最后`AcceptanceTester`只是一个类的名称，它在 YAML 文件中定义为特定测试类型：

```php
$ grep class tests/codeception/acceptance.suite.yml
class_name: AcceptanceTester

```

验收测试是测试的最高级别，作为一种高级用户导向的集成测试。正因为如此，验收测试最终会使用几乎真实的测试环境，其中不需要任何模拟或伪造。显然，我们需要某种初始状态，我们可以回退到这个状态，尤其是如果我们执行的动作会修改数据库的状态。

根据 Codeception 文档，我们本可以使用数据库快照在每次测试开始时加载。不幸的是，我没有找到这个功能正常工作的。所以后来，我们将被迫使用固定值。然后一切都会更有意义。

当我们编写验收测试时，我们还将探索你可以与之一起使用的各种模块，例如 PHPBrowser 和 Selenium WebDriver 及其相关配置选项。

### FunctionalTester

正如我们之前所说的，`FunctionalTester`代表我们在处理功能测试时的角色。

你可以将功能测试视为从更高角度利用实现正确性的方式。

实现功能测试的方式与验收测试的结构相同，以至于我们为 Codeception 中的验收测试编写的代码大多数时候可以轻松地与功能测试的代码交换，所以你可能自己会问：“差异在哪里？”

必须注意的是，功能测试的概念是 Codeception 的特定概念，可以被认为是与应用程序中中间层集成测试几乎相同。

最重要的是，功能测试不需要 web 服务器来运行，并且被称为 **无头测试**：因此，它们不仅比验收测试更快，而且由于在特定环境中运行的所有影响，它们也更不“真实”。并且默认情况下由基本应用程序提供的验收测试几乎与功能测试相同。

由于这个原因，并且正如在第二章中强调的，“为测试做准备”，我们将最终拥有更多的功能测试，这些测试将涵盖我们应用程序特定部分更多的用例。

`FunctionalTester` 以某种方式设置了 `$_GET`、`$_POST` 和 `$_REQUEST` 变量，并在测试中运行应用程序。因此，Codeception 随带模块，允许它与底层框架交互，无论是 Symfony2、Laravel4、Zend，还是在我们的案例中，Yii 2。

在配置文件中，你会注意到为 Yii 2 启用的模块：

```php
# tests/functional.suite.yml

class_name: FunctionalTester
modules:
    enabled:
      - Filesystem
      - Yii2
# ...
```

`FunctionalTester` 对所使用的技术的理解更好，尽管他可能对将要测试的各种功能如何详细实现一无所知；他只知道规格说明。

这正是功能测试应由开发者或任何接近了解如何将各种功能公开给大众的人拥有或编写的一个完美案例。

通过 API 暴露的 REST 应用程序的基本功能也将进行大量测试，在这种情况下，我们将有以下场景：

+   我可以使用 POST 发送正确的认证数据，并将收到包含成功认证的 JSON

+   我可以使用 POST 发送错误的认证数据，并将收到包含失败认证的 JSON

+   经过正确的认证后，我可以使用 GET 来检索用户数据

+   经过正确的认证后，当进行 GET 请求以获取用户信息并指出是我时，我会收到一个错误信息

+   我可以使用 POST 发送我的更新后的哈希密码

+   没有正确的认证，我无法执行前面的任何操作

需要记住的最重要的事情是，在每个测试结束时，你有责任保持内存清洁：PHP 应用程序在处理请求后不会终止。在同一个内存容器中发生的所有请求都不是隔离的。

### 小贴士

如果你看到你的测试在某些未知原因下失败，而它们本不应该失败，尝试单独执行一个测试。

### 单元测试器

我将`UnitTester`放在最后，因为它是一个非常特殊的存在。据我们所知，到目前为止，Codeception 可能已经使用了一些其他框架来覆盖单元测试，我们相当确信 PHPUnit 是唯一能够实现这一目标的候选框架。如果你已经使用过 PHPUnit，你会记得学习曲线以及理解其语法和执行最简单任务时的初始问题。

我发现大多数开发者对 PHPUnit 有着爱恨交加的关系：要么你学会了它的语法，要么你花了一半的时间查阅手册才能找到一点线索。我不会责怪你。

我们将看到，如果我们遇到测试难题时，Codeception 将再次伸出援手：记住，这些单元测试是我们将要测试的工作中最简单、最原子化的部分。与之相伴的是集成测试，它们覆盖了不同组件的交互，很可能是使用模拟数据和固定值。

正如我们在第四章中将要看到的，*使用 PHPUnit 进行隔离组件测试*，如果你习惯于使用 PHPUnit，你不会在编写测试时遇到任何特别的问题；否则，你可以使用`UnitTester`并通过使用 Verify 和 Specify 语法来实现相同的测试。

`UnitTester`假设对签名和基础设施以及框架的工作有深入的理解，因此这些测试可以被认为是测试的基石。

与其他任何类型的测试相比，它们运行得超级快，而且它们也应该相对容易编写。

你可以从足够简单的断言开始，在需要处理固定值之前转向数据提供者。更多内容将在下一章中介绍。

## Codeception 提供的其他功能

除了测试类型之外，Codeception 还提供了一些辅助工具，帮助你组织、模块化并扩展你的测试代码。

正如我们所看到的，功能测试和验收测试具有非常简单和声明性的结构，所有与特定验收标准相关的代码和场景都保存在同一文件中，并且这些测试是线性执行的。

在大多数情况下，就像在我们的例子中一样，这已经足够好了，但是当你的代码开始增长，组件和功能的数量变得越来越复杂时，执行验收或功能测试的场景列表和步骤可能会相当长。

此外，一些测试可能最终会依赖于其他测试，因此你可能需要开始考虑编写更紧凑的场景，并在你的测试中推广代码重用，或者将测试拆分为两个或更多个测试。

如果你觉得你的代码需要更好的组织结构，你可能想要开始生成`CEST`类而不是普通的测试，这些普通的测试被称为`CEPT`。

`CEST`类将所有场景作为一个方法分组，如下面的代码片段所示：

```php
<?php
// SuccessfulLoginCest.php

class SuccessfulLoginCest
{
    public function _before(\Codeception\Event\TestEvent $event) {}

    Codeception\Event\TestEvent $event 

     public function _fail(Codeception\Event\TestEvent $event) {}

    // tests
    public function loginIntoTheApplicationTest(\AcceptanceTester $I)
    {
        $I->wantTo("login into the application from any page");
        $I->amOnPage("/");
        $I->click("login");
        $I->fillField("username", $username);
        $I->fillField("password", $password);
        $I->click("submit");
        $I->canSee("logout (".$username.")");
        $I->seeInCurrentUrl("/");
        // ...
    }
}
?>
```

任何不以下划线开头的函数都被视为测试，而保留方法`_before`和`_after`分别在测试类中测试列表的开始和结束时执行，而`_fail`方法在失败时用作清理方法。

仅此可能还不够，你可以使用文档注释来创建可重用的代码，在测试前后使用`@before <methodName>`和`@after <methodName>`来运行这些代码。

你也可以更加严格，要求在运行任何其他测试之前先通过特定的测试，可以使用文档注释`@depends <methodName>`来实现。

我们将使用其中的一些文档注释，但在开始安装 Codeception 之前，我想强调两个额外的功能：**PageObjects**和**StepObjects**。

+   PageObject 是测试自动化工程师中的一种常见模式。它将网页表示为一个类，其中其 DOM 元素是类的属性，而方法则提供与页面的一些基本交互。使用 PageObjects 的主要原因是为了避免在测试中硬编码 CSS 和 XPATH 定位器。Yii 在`/tests/codeception/_pages`中提供了一些 PageObjects 的示例实现。

+   StepObject 是提高测试中代码复用性的另一种方式：它将定义一些可以在多个测试中使用的常见操作。与 PageObjects 一起，StepObjects 可以变得相当强大。StepObject 扩展了`Tester`类，并可以用来与 PageObject 交互。这样，你的测试将减少对特定实现的依赖，并在标记和与页面中每个组件交互的方式发生变化时节省重构的成本。

为了将来参考，你可以在 Codeception 文档的“高级使用”部分找到所有这些内容，包括其他功能，如分组和一个交互式控制台，你可以用它来在运行时测试你的场景。[`codeception.com/docs/07-AdvancedUsage`](http://codeception.com/docs/07-AdvancedUsage)

# 在 Yii 2 中安装 Codeception

现在我们已经看到了理论上我们可以用 Codeception 做什么，让我们继续并安装它。

Yii 自带了它的 Codeception 扩展，该扩展提供了一个单元测试的基础类(`yii\codeception\TestCase`)，一个需要数据库交互的测试类(`yii\codeception\DbTestCase`)，以及 Codeception 页面对象的基础类(`yii\codeception\BasePage`)。

如同往常，我们首选的方法是使用 Composer：

```php
$ composer require "codeception/codeception: 2.0.*" --prefer-dist --dev

```

使用`–prefer-dist`有特定的原因；如果你使用 Git，你可能会因为 Git 子模块而陷入困境（但再次排除`/vendor`文件夹应该解决这些问题）。为了避免每次使用 Composer 时都重复，只需将以下内容添加到你的`composer.json`文件中：

```php
// composer.json

{
    "config": {
        "preferred-install": "dist"
    }
}
```

此外，请记住，如果您已手动将组件添加到您的 `composer.json` 文件中，则使用 `composer install` 将不会工作，因为它会将其视为不匹配并引发错误。要安装包，您需要运行 `composer update`，无论是针对您安装的所有包，还是专门针对此包：

```php
$ composer update codeception/codeception

```

如前所述，您可能还对两个额外的包 `codeception/specify` 和 `codeception/verify` 感兴趣。这两个包提供了一层额外的抽象，允许您使用面向业务的语言编写更易于阅读的测试，接近 BDD 定义的外观。

您的 `composer.json` 文件将包含以下内容：

```php
// composer.json

{
    "require-dev": {
        "yiisoft/yii2-codeception": "*",
        "yiisoft/yii2-debug": "*",
        "yiisoft/yii2-gii": "*",
        "codeception/codeception": "2.0.*",
        "codeception/specify": "*",
        "codeception/verify": "*"
    }
}
```

## 在 Codeception 中找到您的路径

我们的所有测试都位于 `/tests/codeception` 文件夹中。在 2.0 版本中，此文件夹直接包含所有套件及其所需的配置文件以及 Codeception 本身。以下配置步骤基于此结构。

通过列出 `/tests` 文件夹的内容，我们将看到主要的 Codeception 配置文件，而每个单独的套件在 `/tests/codeception` 文件夹内都有自己的配置文件，我们可以相应地修改以覆盖或进一步配置我们的测试。从我们的 `/tests` 文件夹开始，以下是我们将处理的配置文件：

+   `codeception.yml`：这是用于所有套件和 Codeception 的通用配置

+   `codeception/acceptance.suite.yml`：这是用于验收测试的

+   `codeception/functional.suite.yml`：这是用于功能测试的

+   `codeception/unit.suite.yml`：这是用于单元测试的

与这些文件一起，还有一些额外的配置文件，这些文件主要用于 Yii：`_bootstrap.php` 和 `config/` 文件夹的内容。一些文件使用下划线前缀仅是为了让 Codeception 忽略它们。如果您需要在各种套件文件夹中创建新文件，请记住这一点。

在 `/tests/codeception` 文件夹内，您将找到包含每个单独测试套件测试的文件夹，`unit/`、`functional/` 和 `acceptance/`。每个文件夹都将包含套件的自定义 `_bootstrap.php` 文件、实际测试以及其他用于示例的文件夹。

`/tests/codeception` 中包含的其他几个文件夹如下：

+   `bin/`：它包含用于对测试数据库运行迁移的测试-bound `yii` CLI 命令，我们将使用它。

+   `_data/`：它包含数据库的快照（`dump.sql`），通常用于将数据库恢复到初始状态以进行验收测试，但它可以包含任何内容，例如，此文件夹将由 Codeception 使用，如果您希望它从您用纯英语创建的测试中生成（并发布）各种场景（运行 `codecept help generate:scenarios` 命令以获取更多信息）。

+   `_output/`：这个文件夹将非常有用，因为它将包含在您的验收或功能测试失败时获取的页面输出，这为您提供了另一种检查和理解问题的方式。

+   `_pages/`：这是 Codeception 页面对象存储的地方。基本应用程序已经提供了三个页面对象，分别是 `AboutPage.php`、`ContactPage.php` 和 `LoginPage.php`。我们将在稍后进一步探讨这部分，因为它们将证明极其有用，因为它们极大地简化了我们的生活，并促进了代码的模块化和重用。

+   `_support/`：这部分用于存放额外的支持文件，目前包含用于用提供的 fixtures 填充数据库的 `FixtureHelper` 类。

## 配置 Codeception

现在，我们基本上应该知道所有配置文件的位置，因此在我们开始与 Codeception 交互并首先运行所有提供的测试以及我们自己的测试之前，我们将审查它们的内容并对其进行调整。

让我们从不同套件的 YAML 配置文件开始。

默认情况下，验收测试配置为使用 PHPBrowser。我们将看到如何调整以使用 Selenium WebDriver，但一般来说，这两个工具都需要至少一个 URL 来访问我们的应用程序：

```php
# tests/codeception/acceptance.suite.yml

class_name: AcceptanceTester
modules:
    enabled:
        - PhpBrowser
    config:
        PhpBrowser:
            url: 'http://basic.yii2.sandbox'

```

默认 URL 是 `http://localhost:8080`，当您使用 Vagrant 或 PHP 内置服务器时，您不需要更改它。在先前的示例中，我已经设置了一个自定义域名；为了运行您的测试，这并不是必需的，因为它可能需要额外的配置步骤，而这些步骤通常是不需要的，除非您在一个更大的环境中，并且您的配置稍微复杂一些（例如，如果您的测试是在远程执行的）。我们将在最后一章中看到更多关于这方面的内容。

请注意，您不需要指定入口文件`index-test.php`，因为您希望 Yii 为您解析路由。

对于我们的功能测试，我们的应用程序的基本 URL 不是必需的，正如我之前指出的。事实上，Codeception 对于我们的功能测试所关心的是应用程序的入口脚本：所有功能都是由 `yii2-codeception` 包提供的（这个包应该已经预安装在您的应用程序中），因此，在配置文件中，您只有一个指向测试应用程序配置的引用：

```php
# tests/codeception/functional.suite.yml
...
modules:
    config:
        Yii2:
            configFile: 'codeception/config/functional.php'

```

转到这个文件，我们会发现一开始就设置了一些 `$_SERVER` 变量：

```php
// tests/codeception/config/functional.php
...
// set correct script paths
$_SERVER['SCRIPT_FILENAME'] = YII_TEST_ENTRY_FILE;
$_SERVER['SCRIPT_NAME'] = YII_TEST_ENTRY_URL;

```

这两个常量已在 Yii 启动文件中定义：

```php
// tests/codeception/_bootstrap.php

defined('YII_TEST_ENTRY_URL') or define('YII_TEST_ENTRY_URL', parse_url(\Codeception\Configuration::config()['config']['test_entry_url'], PHP_URL_PATH));

defined('YII_TEST_ENTRY_FILE') or define('TEST_TEST_ENTRY_FILE', dirname(__DIR__) . '/web/index-test.php');
```

换句话说，入口文件始终是位于 `/web/index-test.php` 的那个，而 URL 可以在主配置文件中进行配置：

```php
// tests/codeception.yml
...
config:
    test_entry_url: https://basic.yii2.sandbox/index-test.php

```

### 注意

请记住将主机名调整为您将使用的名称，或者保留默认值，即 `localhost:8080`。

对于单元测试，没有太多需要配置的，因为 Codeception 只是围绕 PHPUnit 进行封装，而 *verify* 和 *specify* 这两个包将直接使用。

剩下的唯一要更新的是数据库的配置：正如我们之前所说的，目前，你只需在`/tests/codeception/config/config.php`中更新 DSN，就像定义主 Yii 数据库配置一样。

## Yii 2 中可用的测试

与 Yii 1 提供的相比，Yii 2 现在为任何测试套件提供了工作测试的示例。这是一件好事，因为它将帮助我们了解如何构建和实现我们的测试。

一旦服务器运行正常并且所有配置都设置妥当，我们就可以运行以下命令：

```php
$ cd tests
$ ../vendor/bin/codecept run

```

这将运行所有测试并查看它们通过。最后，你将看到一个漂亮的总结：

```php
...
Time: 6.92 seconds, Memory: 35.75Mb

OK (12 tests, 60 assertions)
$

```

提供的验收和功能测试的测试相当直观；它们基本上确保了四个页面，即主页、关于页面、联系页面和登录页面，在基本应用程序中按预期工作。

这些测试完全相同，唯一的区别是验收测试考虑了您通过 Selenium 运行测试的能力，并包含针对它的特定指令：

```php
// tests/functional/ContactCept.php
...
if (method_exists($I, 'wait')) {
    $I->wait(3); // only for selenium
}
...
```

这只是一个例子，现在对你来说可能并不重要，因为我们将在第七章中详细探讨 Selenium WebDriver 的工作原理，*享受浏览器测试的乐趣*。

Yii 2 提供的单元测试与预期不同；它们主要覆盖各种组件之间的集成测试，例如登录表单和联系表单，而将用户测试的实现负担留给了我们。我们将在下一章中实现这一点。

在单元测试中唯一值得注意的事情是，它们使用*Specify*以更声明性的方式编写单元测试，而不是更常见的 PHPUnit 语法。再次强调，这只是一个语法糖，它可能对你开始时更容易。

# 与 Codeception 交互

到目前为止，我们已经看到了`codecept`命令的两个参数：

+   `build`: 这个命令用于构建“测试器”以及在使用任何附加模块时所需的任何附加代码

+   `run`: 这个命令用于执行测试

有几个参数你可以与`run`一起使用，我想提醒你注意，因为这些在运行和调试测试时将很有用。`run`命令的语法如下：

```php
$ vendor/bin/codecept run [options] [suite] [test]

```

首先，你可以运行特定的测试套件，例如单元测试、验收测试或功能测试，或者更具体地运行单个测试文件，例如：

```php
$ ../vendor/bin/codecept run acceptance LoginCept.php
…
Time: 3.35 seconds, Memory: 13.75Mb

OK (1 test, 5 assertions)

```

在前面的命令中，你还可以使用`--steps`选项，这是一种更详细地显示测试运行时所有单个步骤的方法。

或者，你也可以使用`--debug`选项，它不仅会显示应用程序执行的步骤，还会显示幕后发生的事情，例如向特定 URL 发送数据 POST 请求、页面加载或设置的 cookie 列表。

## 创建测试

当您编写了测试并看到它们通过时，您可能只会关心这些，但在编写测试之前，您首先需要编写它们。

Codeception 通过在命令行上提供代码生成参数来帮助我们开始：

+   `generate:cept`：此命令用于生成 CEPT 测试

+   `generate:cest`：此命令用于生成 CEST 测试

+   `generate:phpunit`：此命令用于生成 PHPUnit 测试，不包含 Codeception 的附加功能

+   `generate:test`：此命令用于生成单元测试

所有的先前参数都需要参数套件名称和要创建的文件名称：

```php
$ ../vendor/bin/codecept generate:cept acceptance ModalLoginCept
Test was created in ModalLoginCept.php

```

您可以通过运行不带参数的`codecept`来查看这些命令以及更多命令。

## 测试数据库上的迁移

我发现特别方便并且我们将广泛使用的一项功能是能够在您的测试数据库上运行与我们为应用程序创建的相同迁移。

### 注意

迁移这个概念并不仅限于 Yii，您可以在[`www.yiiframework.com/doc-2.0/guide-db-migrations.html`](http://www.yiiframework.com/doc-2.0/guide-db-migrations.html)的文档中了解更多相关信息。

在`/tests/codeception/bin/`文件夹中，您将找到可以用于之前配置的测试数据库的`yii` CLI 命令行，以运行迁移。

假设您位于项目的根目录，以下命令序列将向您展示如何运行迁移：

```php
$ cd tests/codeception
$ php bin/yii migrate/up
Yii Migration Tool (based on Yii v2.0.0-dev)

Creating migration history table "migration"...done.
No new migration found. Your system is up-to-date.

```

`yii` CLI 与位于项目根目录的主程序完全相同，唯一的区别是它将读取测试配置，特别是关于数据库的那部分。

# 摘要

在本章中，我们体会到了 Codeception 的广度和质量。

我们已经看到了三种测试类型，即单元测试、功能测试和验收测试，这些测试我们将贯穿整本书。我们还接触了一些由工具和 Yii 2 Codeception 模块提供的附加功能。我们学习了如何与之交互、生成测试以及处理调试和保持测试数据库与主应用程序数据库同步。

在下一章中，我们将开始重构我们的`User`类，首先添加测试，然后逐步通过所有最重要的 PHPUnit 特性。

### 小贴士

**下载示例代码**

您可以从[`www.packtpub.com`](http://www.packtpub.com)下载您购买的所有 Packt Publishing 书籍的示例代码文件。如果您在其他地方购买了这本书，您可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)并注册，以便将文件直接通过电子邮件发送给您。
