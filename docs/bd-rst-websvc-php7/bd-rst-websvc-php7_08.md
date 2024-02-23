# API 测试-守卫在大门上

在上一章中，我们解决了我们识别出的问题，并完成了 RESTful web 服务中剩下的事情。然而，为了确保质量，我们需要测试我们的端点，手动测试是不够的。在现实世界的项目中，我们无法重复测试每个端点，因为在现实世界中有更多的端点。因此，我们转向自动化测试。我们编写测试用例并以自动化的方式执行它们。事实上，首先编写测试用例，运行它们，然后编写代码来满足该测试的要求更有意义。这种开发方法称为 TDD（测试驱动开发）。

TDD 是好的，并确保我们按照我们的测试用例进行工作。然而，在这本书中，我们没有使用 TDD，因为有很多东西要理解，我们不想同时包括一件事。所以现在，当我们完成了概念、理解和在 Lumen 中编写 RESTful web 服务（对我们许多人来说也是新的）时，现在我们可以做这个缺失的事情，也就是测试。TDD 并非必不可少，但测试是。如果我们迄今为止没有为了理解其他东西而编写测试，那么现在我们应该这样做。以下是本章将涵盖的主题：

+   自动化 API 测试的需求

+   测试类型：

+   单元测试

+   集成测试

+   功能测试

+   验收测试

+   我们将编写什么类型的测试？

+   测试框架：

+   介绍 CodeCeption

+   设置和配置

+   编写 API 测试

+   总结和更多资源

# 自动化测试的需求

正如我们之前讨论过的，在现实世界中，我们无法在每个主要功能或更改后重复测试每个端点。我们可以尝试，但我们是人类，我们可能会错过。更大的问题是，我们有时可能会认为我们已经测试过了，但却错过了，因为没有记录我们测试过什么，我们无法知道。如果我们有一个单独的质量保证团队或人员，他们很可能会测试并记录下来。然而，在 RESTful web 服务的情况下，这将占用更多的时间，或者可能的情况是 QA 人员将作为一个整体测试最终产品，而不是 RESTful web 服务。

就像 RESTful web 服务作为产品的一个组件或一个方面一样，RESTful web 服务还有更多低级组件。不仅仅是端点，而是这些端点依赖于更低级的代码。因此，为了使我们的调试更容易，我们也为这些低级组件编写测试。此外，这样我们可以确保这些低级组件运行良好，并且按照其意图进行操作。在出现任何问题的情况下，我们可以运行测试，并确切地知道哪些地方出了问题。

尽管一开始编写测试需要时间，但从长远来看是有好处的。首先，它节省了在每次更改后重复测试相同端点的时间。然后，在重构某些东西时，它在很大程度上有所帮助。它让我们知道涟漪效应在哪里，以及由于我们的重构受到了什么影响。尽管一开始编写测试需要时间，但如果我们打算做一些长期存在的东西，那么它是值得的。事实上，软件开发成本比维护成本要低。这是因为它只会开发一次，但要维护和做更改，将会消耗更多的时间。因此，如果我们编写了自动化测试，它将确保一切都按照要求正常工作，因为维护代码的人很可能不是第一次编写代码的人。

没有一种测试可以提供所有这些好处，但有不同类型的测试，每种测试都有其自己的好处。既然我们知道了自动化测试和编写测试用例的重要性，让我们了解一下不同类型的测试。

# 测试类型

在不同的上下文中有不同类型的测试。在我们的情况下，我们将讨论四种主要类型的自动化测试。这些是不同类型的测试，基于我们测试的方式和内容：

+   单元测试

+   集成测试

+   功能测试

+   验收测试

# 单元测试

在单元测试中，我们分别测试不同的单元。所谓的单元，指的是非常小的独立组件。显然，组件彼此依赖，但我们考虑的是一个非常小的单元。在我们的情况下，这个小单元是类。这个类是一个应该具有单一职责的单元，它应该与其他类或组件抽象，并且依赖于最少数量的其他类或组件。无论如何，我们可以说在单元测试中，我们通过创建该类的对象来测试类，而不管它是否满足所需的行为。

一个重要的事情要理解的是，在单元测试期间，我们的测试不应该触及除了测试类/代码之外的代码。如果在单元测试期间，这段代码与其他对象或外部内容进行交互，我们应该模拟这些对象，而不是与实际对象进行交互，以便其他对象的方法的结果不会影响我们正在测试的单元/类的结果。你可能想知道我们所说的模拟是什么意思？**模拟**意味着提供一个虚假对象，并根据所需的行为设置它。

例如，如果我们正在使用`User`类进行测试，并且它依赖于`Role`类，那么`Role`类中的问题不应该导致`User`类的测试失败。为了实现这一点，我们将模拟`Role`对象并将其注入`User`类对象中，然后为`User`类使用的`Role`类的方法设置固定的返回值。因此，接下来，它将实际调用`Role`类，并且不会依赖于它。

单元测试的好处：

+   它将让我们知道一个类是否没有达到其意图。一段时间后，当项目处于维护阶段时，另一个开发人员将能够理解这个类的意图。它的方法的意图是什么。因此，它将像是由知道为什么编写了该类的开发人员编写的类的手册。

+   另外，正如我们刚刚讨论的，我们应该模拟对象，测试类依赖的对象，我们应该能够将模拟对象注入到测试类的对象中。如果我们能够做到这一点，并且能够在不调用外部对象的情况下进行管理，那么我们才能称我们的代码为可测试代码。如果我们的代码不可测试，那么我们就无法为其编写单元测试。因此，单元测试帮助我们使我们的代码可测试，这实际上是更松散耦合的代码。因此，具有可测试代码是一种优势（因为它是松散耦合的，更易于维护），即使我们不编写测试。

+   如果我们遇到任何问题，它将让我们调试出问题所在。

+   由于单元测试不与外部对象交互，因此它们比其他一些测试类型更快。

编写单元测试的开发人员被认为是更好的开发人员，因为带有测试的代码被认为是更干净的代码，因为开发人员已经确保了单元级组件不是紧密耦合的。单元测试可以作为类提供的手册以及如何使用它。

# 验收测试

验收测试是单元测试的完全相反。单元测试是在最低级别进行的，而验收测试是在最高级别进行的。验收测试是最终用户将如何看待产品以及最终用户将如何与产品进行交互的方式。在网站的情况下，在验收测试中，我们编写测试以从外部访问 URL。测试工具模拟浏览器或外部客户端来访问 URL。事实上，一些测试工具还提供使用 Web 浏览器（如 Chrome 或 Firefox）的选项。

大多数情况下，这些测试是由 QA 团队编写的。这是因为他们是确保系统对最终用户正常工作的人，正如预期的那样。此外，这些测试的执行速度很慢。对于用户界面，有时需要测试很多细节，因此需要一个单独的 QA 团队来进行此类测试。但这只是一个常见的做法，因此根据情况可能会有例外。

验收测试的好处：

+   验收测试让您看到最终用户如何从外部看到和与您的软件交互

+   它还可以让您捕捉将在任何特定的 Web 浏览器中发生的问题，因为它使用真实的 Web 浏览器来执行测试

+   由于验收测试是为了从外部执行而编写的，所以无论您要测试哪个系统以及用于编写系统的技术或框架是什么都无关紧要

例如，如果您正在使用 PHP 编写测试用例的工具，那么您也可以将其用于其他语言编写的系统。因此，开发语言是 PHP、Python、Golang 还是.Net 都无关紧要。这是因为验收测试从外部击中系统，而不需要了解系统的任何内部知识。它是这四种测试中唯一一个在不考虑任何内部细节的情况下测试系统的测试类型。

验收测试非常有用，因为它们与您的系统使用真实浏览器进行交互。因此，如果某些内容在特定浏览器中无法正常工作，那么这些问题可以被识别出来。但请记住，使用真实浏览器，这些测试需要时间来执行。如果使用浏览器模拟，速度也会很慢，但仍然比真实浏览器快。请注意，验收测试被认为是这四种测试中最慢和最耗时的。

# 功能测试

功能测试与验收测试类似；但是，它是从不同的角度。功能测试是关于测试功能需求。它测试功能需求并从系统外部进行测试。但是，它具有内部可见性，并且可以在测试用例中执行系统的一些代码。

与验收测试类似，它击中 URL；然而，即使要击中 URL，它也会执行浏览器或外部客户端将在特定 URL 上执行的代码。但实际上并不是从外部击中 URL。测试实际上并没有击中 URL，它只是模拟了它。这是因为与验收测试不同，我们对最终用户如何与之交互并不感兴趣，而是如果代码从该 URL 执行，我们想要知道响应。

我们更感兴趣的是我们的功能需求是否得到满足，如果没有得到满足，问题出在哪里？

功能测试的好处：

+   通过功能测试，测试工具可以访问系统，因此显示的错误细节比验收测试更好。

+   功能测试实际上不会打开浏览器或外部客户端，因此速度更快。

+   在功能测试中，我们还可以通过测试工具直接执行系统代码，因此在某些情况下，我们可以这样做来节省测试用例编写时间，或者使测试用例执行更快。有许多可用的测试工具。我们将很快使用其中一个名为 CodeCeption。

# 集成测试

集成测试在某种程度上与单元测试非常相似，因为在这两种测试中，我们通过使它们的对象调用它们的方法来测试类。但它们在测试类的方式上有所不同。在单元测试的情况下，我们不会触及我们要测试的类与之交互的其他对象。但在集成测试中，我们想要看到它们如何一起工作。我们让它们相互交互。

有时，一切都按照单元测试的要求正常运行，但在更高级别的测试（功能测试或验收测试）中却不正常，我们会根据需求通过访问 URL 进行测试。因此，在某些情况下，高级别测试失败，而单元测试通过，为了缩小问题的范围，集成测试非常有用。因此，可以认为集成测试处于功能测试和单元测试之间。功能测试是关于测试功能需求，而单元测试是关于测试单个单元。因此，集成测试处于两者之间，它测试这些单个单元如何一起工作；然而，它通过测试代码中的小组件进行测试，同时也让它们相互交互。

一些开发人员编写集成测试并将其称为单元测试。实际上，集成测试只是让测试中的代码与其他对象进行交互，因此它可以测试这些类在与系统组件交互时的工作方式。因此，如果测试中的代码非常简单且需要与系统交互进行测试，有些人会编写集成测试。然而，并不一定只编写一个单元测试或集成测试，如果有时间，可以同时编写两者。

集成测试的好处：

+   当单元测试不足以捕捉错误，高级别测试不断告诉您有问题时，集成测试非常有用，可以帮助调试问题。

+   由于集成测试的性质，在重构时非常有帮助，可以告诉您新更改受到了什么影响。

# 我们将进行哪种类型的测试？

每种类型的测试都有其重要性，尤其是单元测试。然而，我们主要进行 API 测试，将测试我们的 RESTful Web 服务端点。这并不意味着单元测试不重要，只是我们在本章主要关注 API 测试，因为本书侧重于 RESTful Web 服务。实际上，测试是一个大课题，你将能够看到关于测试和 TDD 的完整书籍。

如今，“BDD”（行为驱动开发）是一个更流行的术语。它与 TDD 并没有完全不同。它只是陈述测试用例的一种不同方式。实际上，在 BDD 中，没有测试用例，而是规范。它们具有相同的目的，但 BDD 以更友好的方式解决问题，即通过陈述规范并实现它们，这就是 TDD 的工作方式。因此，TDD 和 BDD 并没有不同，只是解决同一个问题的不同方式。

我们可以以功能测试和验收测试的方式进行 API 测试。然而，将 API 测试编写为功能测试更有意义。因为功能测试将更快，并且对我们的代码库有洞察力。这也更有意义，因为验收测试是为最终用户而设计的，而最终用户不使用 API。最终用户使用用户界面。

# 测试框架

就像我们有用于编写软件的框架一样，我们也有用于编写测试用例的框架。由于我们是 PHP 开发人员，我们将使用一个用 PHP 编写的测试框架，或者我们可以在其中用 PHP 编写测试用例，以便我们可以轻松使用它。

首先，无论我们用于应用开发的开发框架是什么，我们都可以在 PHP 中使用不同的测试框架。然而，Laravel 和 Lumen 也带有测试工具。我们也可以使用它们来编写测试用例。实际上，为 Lumen 编写测试用例会更容易，但它将是特定于 Lumen 和 Laravel 的。在这里，我们将使用一个框架，您将能够在 Lumen 和 Laravel 生态系统之外以及任何 PHP 项目中使用它，无论您使用哪个开发框架来编写代码。

PHP 中有许多不同的测试框架，那么我们如何决定使用哪一个？我们将使用一个不太低级的框架，因为我们不打算编写单元测试，而是功能测试，所以我们选择了一个稍微高级的框架。一个著名的单元测试框架是 PHPUnit：[`phpunit.de/`](https://phpunit.de/)。

还有另一个以**BDD**（行为驱动开发）风格命名的单元测试框架，名为 PHPSpec：[`www.phpspec.net`](http://www.phpspec.net)，如果您想学习或编写单元测试，PHPSpec 也很棒。然而，在这里，我们将使用一个既适用于功能测试又适用于单元测试的框架。尽管我们不写单元测试，但我们希望考虑一个稍后也可以用于单元测试的框架。我选择的框架是 CodeCeption：[`codeception.com/`](http://codeception.com/)，因为它在 API 测试方面似乎非常出色。另一个 BDD 风格的选择可能是 Behat：[`behat.org/en/latest/`](http://behat.org/en/latest/)。这是一个高级测试框架，但如果我们进行验收测试，甚至更好的是如果我们有一个专门的 QA 团队，他们将用 Gherkin 语法（[`github.com/cucumber/cucumber/wiki/Gherkin`](https://github.com/cucumber/cucumber/wiki/Gherkin)）编写许多测试用例，这非常接近自然语言。然而，对于 PHP 开发人员来说，Behat 和 Gherkin 可能有更多的学习曲线，而 CodeCeption 只是简单的 PHP（尽管如果需要，它也可以使用 Gherkin），因此许多读者将是新手编写测试用例，我将保持简单并贴近 PHP。然而，这是我两年前写的关于选择 API 测试框架的详细比较，虽然有些过时，但对于大部分内容仍然有效。如果您感兴趣，可以看一下[`haafiz.me/programming/api-testing-selecting-testing-framework`](http://haafiz.me/programming/api-testing-selecting-testing-framework)。

# CodeCeption 简介

CodeCeption 是用 PHP 编写的，并由 PHPUnit 支持。CodeCeption 声称*CodeCeption 使用 PHPUnit 作为运行其测试的后端。因此，任何 PHPUnit 测试都可以添加到 CodeCeption 测试套件中，然后执行*。

除了验收测试之外的其他测试需要一个具有对测试代码的洞察或连接的测试框架。如果我们使用的是开发框架，那么测试框架应该具有某种针对该框架的模块或插件。CodeCeption 在这方面做得很好。它为不同的框架和 CMS 提供了模块，例如 Symfony、Joomla、Laravel、Lumen、Yii2、WordPress 和 Zend 框架。只是让您知道，这些只是一些框架。CodeCeption 还支持许多其他模块，可以在不同情况下提供帮助。

# 设置和理解结构

安装 CodeCeption 有不同的方法，但我更喜欢 composer，这是安装不同 PHP 工具的标准方式。所以让我们安装它：

```php
composer require "codeception/codeception" --dev
```

正如您所看到的，我们正在使用`--dev`标志，这样它就会将 CodeCeption 添加到`composer.json`文件中的`require-dev`块中。因此，在生产环境中，当您运行`composer install --no-dev`时，它将不会安装在`require-dev`块中的依赖项。如果有疑惑，请查看与 composer 相关的章节，即第五章，*使用 Composer 加载和解决问题，一个进化*。

安装完成后，我们需要设置它以编写测试用例，并使其成为我们项目的一部分。安装只意味着它现在在`vendors`目录中，现在我们可以通过 composer 执行 CodeCeption 命令。

要设置，我们需要运行 CodeCeption 引导命令：

```php
composer exec codecept bootstrap
```

`codecept`是 CodeCeption 在`vendor/bin`目录中的可执行文件，所以我们通过 composer 执行它，并给它一个参数来运行`bootstrap`命令。因此，在执行这个命令之后，一些文件和目录将被添加到你的项目中。

以下是它们的列表：

```php
codeception.yml

tests/_data/
tests/_output/

tests/acceptance/
tests/acceptance.suite.yml
tests/_support/AcceptanceTester.php
tests/_support/Helper/Acceptance.php
tests/_support/_generated/AcceptanceTesterActions.php

tests/functional/
tests/functional.suite.yml
tests/_support/FunctionalTester.php
tests/_support/Helper/Functional.php
tests/_support/_generated/FunctionalTesterActions.php

tests/unit/
tests/unit.suite.yml
tests/_support/UnitTester.php
tests/_support/Helper/Unit.php
tests/_support/_generated/UnitTesterActions.php
```

如果你看一下提到的文件列表，那么你会注意到我们在根目录下有一个文件，即`codeception.yml`，其中包含了 CodeCeption 测试的基本配置。它告诉我们关于路径和基本设置。如果你阅读这个文件，你将能够很容易地理解它。如果你不理解某些东西，现在先忽略它。除了这个文件，其他的都在`tests/`目录下。这些对我们来说更重要。

首先，在`tests/`目录下有两个空目录。`_output`包含测试用例的输出，如果失败的话，`_data`包含数据库查询，如果我们想在运行测试之前和之后设置一个默认的数据库。

除此之外，你可以看到有三组文件，这些文件具有相似的文件，只是测试类型不同。在 CodeCeption 中，我们知道这些组是测试套件。所以，默认情况下，CodeCeption 带有三个测试套件。接受、功能和单元套件。所有这三个套件都包含四个文件和一个空目录。所以，让我们来看看每个文件和目录的目的。

# tests/{suite-name}/

在这里，`{suite-name}`将被套件的实际名称替换；比如说单元套件，它将是`tests/unit/`。

无论如何，这个目录将用于保存我们将要编写的测试用例。

# tests/{suite-name}.suite.yml

这个文件是特定套件的配置文件。它包含了这个特定套件的`ActorName`。Actor 实际上就是具有特定设置和能力的人。根据 actor 的设置，它以不同的方式运行测试。设置包括模块的配置和启用模块。

# tests/_support/_generated/{suite-name}TesterActions.php

这个文件是基于`tests/{suite-name}.suite.yml`中的设置自动生成的文件。

# tests/_support/{suite-name}Tester.php

这个文件使用了`_generated`目录中生成的文件，开发人员可以根据需要进行更多的自定义。然而，通常情况下是不需要的。

# tests/_support/Helper/{suite-name}.php

套件中的这个文件是辅助文件。在这里，你可以在类中添加更多的方法，并在你的测试用例中使用它。就像其他代码有库和辅助程序一样，你的测试用例代码也可以在套件的辅助类中有辅助方法。

请注意，如果你需要不同的辅助类，你可以添加更多的文件。

# 创建 API 套件

在我们的情况下，我们需要单元测试和 API 测试。虽然我们可以使用功能测试套件进行 API 测试，因为这些测试处于功能测试级别，但为了清晰和理解起见，我们可以通过这个命令创建一个单独的 API 套件：

```php
composer exec codecept g:suite api
```

在这个命令中，`g`是`generate`的缩写，它将生成一个 API 套件。`api`只是另一个套件的名称，这个命令已经创建了这些文件和目录：

```php
tests/api/
tests/api.suite.yml
tests/_support/ApiTester.php
tests/_support/Helper/Api.php
tests/_support/_generated/ApiTesterActions.php
```

`api.suite.yml`文件将具有基本设置，但没有太多细节。这是因为`api.suite.yml`文件将具有基本设置，但没有太多细节。这是因为这里的`api`只是一个名称。你甚至可以说：

```php
composer exec codecept g:suite abc
```

它应该已经创建了`abc`套件，具有相同的文件结构和设置。所以，我们的 API 套件只是另一个我们为了清晰和理解而单独创建的测试套件。

# 配置 API 套件

API 需要 REST 客户端来获取 RESTful 网络服务端点。除此之外，它还依赖于 Lumen。我们说 Lumen，因为它将与我们的代码集成，我们正在编写功能级别的测试而不是接受测试。因此，我们的测试框架应该对 Lumen 有洞察力和交互。我们的配置还需要什么？我们需要设置测试的`.env`文件。因此，这就是我们的配置文件的样子：

```php
class_name: ApiTester
modules:
 enabled:  - REST:
 url: /api/v1
            depends: Lumen
        - \Helper\Api
    config:
  - Lumen:
 environment_file: .env.testing
```

在继续之前，请注意，我们在`config/Lumen`下指定了一个不同的环境文件选项，即`environment_file: .env.testing`。因此，如果我们在这里指定了`.env.testing`，那么我们应该有一个`.env.testing`。没什么大不了的，只需复制并粘贴您的`.env`文件。从命令行执行以下操作：

```php
cp .env .env.testing
```

更改数据库凭据，使其指向不同的数据库，该数据库具有您当前数据库架构和数据的副本，基于这些数据，您想编写测试用例。尽管在 Laravel/Lumen 中进行测试时与数据库相关的内容将会回滚，并且不会影响我们的实际数据库，因此在开发中使用相同的数据库也是可以的。但是，在暂存环境中不建议，事实上是禁止使用相同的数据库进行测试；因此，最好从一开始就保持不同的数据库和配置。

我们不会在生产环境中运行测试。我们甚至不会在生产环境中安装与测试相关的工具，正如您所看到的，我们使用`--dev`标志安装了 CodeCeption。因此，当我们的代码在生产环境中，并且我们想要部署一个新功能时，我们的测试用例会在不同的服务器上运行，然后将代码部署到生产服务器上。有几种**CI**（**持续集成**）工具可用于此。

# 编写测试用例

现在，是时候编写测试用例了。首先要了解的是，我们如何决定应该测试什么。我们应该先测试每个端点，然后再测试每个类吗？

首先要理解的是，我们应该只测试我们自己编写的代码。我所说的我们，是指我们团队的某个人。我们不打算测试第三方代码、框架代码或包代码。此外，我们也不想测试每个类和每个方法。在理想情况下，我们可以测试每个细微功能的细节，但这也有其缺点。首先，在现实世界中，我们没有时间这样做。我们打算测试大部分但不是全部部分。另一个原因是，我们编写的所有测试也是一种负担。随着时间的推移，我们还需要维护这些测试。因此，我们只对有意义的部分进行单元测试；只有在实际上执行一些复杂操作的地方才进行测试。如果您有一个函数，它的功能就是调用另一个函数并返回结果，那么我认为这样的代码片段不应该有自己的测试。

另一件事是，如果我们既要进行单元测试又要进行 API 测试，那么我们应该从哪里开始编写测试呢？我们应该先为所有端点编写测试，然后再为所有类编写测试，还是相反，先测试所有类，然后再测试所有端点？我们该如何做呢？我们显然打算测试我们的端点。我们也打算测试这些端点下的代码。这是不同的人可以以不同方式做的事情，但我和许多其他人，我见过的人，都是同时编写 API 测试和单元测试。我更喜欢编写 API 测试并继续为一个资源编写测试。之后，我们将转向控制器的单元测试。在我们的情况下，模型中除了从 Eloquent 或关系继承的内容之外，没有太多东西。在对资源进行 API 测试时，如果我们需要更多细节来修复错误，那么我们可以开始为该类编写单元测试。但这并没有硬性规定。这只是一种偏好问题。

# 针对 post 资源的 API 测试

我们可以以结构化方法编写测试用例，也可以以类的方式编写。两种方式都可以，我建议使用类，这样您可以在某个时候利用面向对象的概念。因此，让我们为此创建一个文件：

```php
composer exec codecept generate:cest api CreatePost
```

这将在`tests/api/CreatePostCest.php`中创建一个类，内容类似于：

```php
<?php   class CreatePostCest {
  public function _before(ApiTester $I)
 { }    public function _after(ApiTester $I)
 { }    // tests
  public function tryToTest(ApiTester $I)
 {  } } 
```

`_before()`方法是为了让您可以在测试用例之前编写任何代码，而`_after()`方法是为了在测试用例之后执行。接下来的方法只是一个示例，我们将对其进行修改。

在这里，我们将编写两种类型的测试。一种是在尝试未登录时创建一个帖子，这应该返回未经授权的错误，另一种是在登录后创建一个帖子，这应该是成功的。

在写之前，让我们设置我们的数据库工厂，以便为帖子获取随机内容，这样我们在测试期间可以使用它。让我们修改`app/database/factories/ModelFactory.php`，使其看起来像这样：

```php
<?php   /* |-------------------------------------------------------------------------- | Model Factories |-------------------------------------------------------------------------- | | Here you may define all of your model factories. Model factories give | you a convenient way to create models for testing and seeding your | database. Just tell the factory how a default model should look. | */   $factory->define(App\User::class, function (Faker\Generator $faker) {
  return [
  'name' => $faker->name,
  'email' => $faker->email,
 ]; });   $factory->define(App\Post::class, function (Faker\Generator $faker) {
    return [
        'title' => $faker->name,
        'content' => $faker->text(),
        'status' => $faker->randomElement(['draft', 'published']),
    ]; **});** 
```

我刚刚添加了粗体标记的代码。所以，我们告诉它返回一个基于`Faker\Generator`类对象生成的参数数组，标题、内容和状态。所以，根据我们在这里定义的不同字段，我们可以通过`ModelFactory`为帖子用户生成随机内容，这样数据将是随机和动态的，而不是静态内容。在测试期间，最好在测试用例中使用随机数据来进行测试。

好的，现在让我们在`CreatePostCest.php`文件中编写我们的测试用例，这是我们将要编写的函数：

```php
// tests if it let unauthorized user to create post public function tryToCreatePostWithoutLogin(ApiTester $I) {
  //This will be in console like a comment but effect nothing
  $I->wantTo("Send sending Post Data to create Post to test if it let it created without login?");    //get random data generated through ModelFactory
  $postData = factory(App\Post::class, 1)->make();    //Send Post data through Post method
  $I->sendPost("/posts", $postData);    //This one will also be like a comment in console
  $I->expect("To receive a unauthorized error resposne");    //Response code of unauthorized request should be 401
  $I->seeResponseCodeIs(401);  }
```

正如你所看到的，注释已经解释了一切，所以除了我们使用`sendPost()`方法发送一个 Post 请求之外，没有必要明确说明任何事情，我们也可以说`sendGet()`或`sendPut()`来使用不同的 HTTP 方法，等等。所以，现在我们需要运行这个测试。

我们可以通过以下方式运行它：

```php
composer exec codecept run api
```

它不会在控制台上给我们清晰的输出。我们可以添加`-v`，`-vv`或`-vvv`来使输出更加详细，但在这个命令中，它会使 composer exec 相关的信息变得越来越详细。所以让我们这样执行：

```php
vendor/bin/codecept run api
```

随意添加`-v`，最多三次，以获得更加详细的输出：

```php
vendor/bin/codecept run api -vv
```

我们可以为路径`vendor/bin/codecept`创建一个别名，在控制台的会话中，我们可以使用这样的简写：

```php
alias codecept=vendor/bin/codecept
codecept run api -vv
```

所以，执行它，你会在控制台中看到很多细节。根据你的需要使用`-v`，`-vv`或`-vvv`。现在让我们这样执行它：

```php
codecept run api -v
```

在我们的情况下，我们的第一个测试应该已经通过了。现在，我们应该编写我们的第二个测试用例，也就是在登录后创建一个帖子。这涉及到更多我们需要理解的东西。所以让我们先看一下这个测试用例的代码，然后再进行审查：

```php
// tests if it let unauthorized user to create post public function tryToCreatePostAfterLogin(ApiTester $I) {
  //This will be in console like a comment but effect nothing
 $I->wantTo("Sending Post Data to create Post after login"**);**
 $user = App\User::first();
    $token = JWTAuth::fromUser($user**);**    //get random data generated through ModelFactory
  $postData = factory(App\Post::class, 1)->make()->first()->toArray();    //Send Post data through Post method
 $I->amBearerAuthenticated($token);   $I->sendPost("/posts", $postData);    //This one will also be like a comment in console
  $I->expect("To receive a unauthorized error resposne");    //Response code of unauthorized request should be 401
 $I->seeResponseCodeIs(200**);**   }
```

如果你看这个测试用例，你会发现它和之前的测试用例代码非常相似，除了一些语句。所以，我已经用粗体标出了这些语句。第一件事是，由于测试用例不同，所以我们的`wantTo()`参数也不同。

然后，我们从数据库中获取第一个用户，并基于用户对象生成一个令牌。在这里，我们调用我们的应用程序代码，因为我们使用了在`api.suite.yml`文件中配置的 Lumen 模块。然后，我们使用 CodeCeption 的`$I->amBearerAuthenticated($token)`方法与我们生成的`$token`一起。这意味着我们发送了一个有效的令牌，所以服务器将把它视为已登录用户。这次响应代码将是 200，所以通过`$I->seeResponseCodeIs(200)`，我们告诉它应该有 200 的响应代码，否则测试应该失败。这段代码就是这样做的。

实际上，还可以有很多类似的测试用例，比如测试如果在请求不完整的情况下返回`400 Bad Request`响应。

运行测试后，你会在控制台的最后看到这个：

```php
OK (2 tests, 2 assertions)
```

这表明我们断言了两件事。<q>断言意味着陈述我们希望为真的期望或事实。</q>简单来说，这是我们在响应中检查的内容。就像现在我们只测试响应代码一样。但在现实世界中，我们会用更多的测试用例来测试整个响应。CodeCeption 也为我们提供了测试这些内容的方法。所以，让我们用更多的断言修改我们当前的两个测试用例。以下是我们将要测试的两个内容：

+   将断言我们得到了响应中的 JSON。

+   将断言我们根据我们的输入得到了正确的响应数据。

所以这是我们的代码：

```php
<?php   use Tymon\JWTAuth\Facades\JWTAuth;   class CreatePostCest {
  public function _before(ApiTester $I)
 { }    public function _after(ApiTester $I)
 { }    // tests if it let unauthorized user to create post
  public function tryToCreatePostWithoutLogin(ApiTester $I)
 {  //This will be in console like a comment but effect nothing
  $I->wantTo("Send sending Post Data to create Post to test if it let it created without login?");    //get random data generated through ModelFactory
  $postData = factory(App\Post::class, 1)->make()->first()->toArray();    //Send Post data through Post method
  $I->sendPost("/posts", $postData);    //This one will also be like a comment in console
  $I->expect("To receive a unauthorized error resposne");    //Response code of unauthorized request should be 401
  $I->seeResponseCodeIs(401);
     // Response should be in JSON format
  $I->seeResponseIsJson();
 }    // tests if it let unauthorized user to create post
  public function tryToCreatePostAfterLogin(ApiTester $I)
 {  //This will be in console like a comment but effect nothing
  $I->wantTo("Sending Post Data to create Post after login");    $user = App\User::first();
  $token = JWTAuth::fromUser($user);    //get random data generated through ModelFactory
  $postData = factory(App\Post::class, 1)->make()->first()->toArray();    //Send Post data through Post method
  $I->amBearerAuthenticated($token);
  $I->sendPost("/posts", $postData);    //This one will also be like a comment in console
  $I->expect("To receive a 200 response");    //Response code of unauthorized request should be 200
  $I->seeResponseCodeIs(200);
        // Response should be in JSON format
 $I->seeResponseIsJson();        //Response should contain data that matches with request $I->seeResponseContainsJson($postData**);**  } } 
```

正如你在上述代码片段中看到的，我们添加了三个额外的断言，你可以看到它是多么简单。实际上，在我们不知道对象可能具有的值时，检查响应中的内容可能会有些棘手。例如，如果我们想要请求并查看帖子列表，那么当我们不知道值时，我们该如何断言呢？在这种情况下，你可以使用基于 JSON 路径的断言，文档在这里：[`codeception.com/docs/modules/REST#seeResponseJsonMatchesJsonPath`](http://codeception.com/docs/modules/REST#seeResponseJsonMatchesJsonPath)。

你会像这样使用它：

```php
$I->seeResponseJsonMatchesJsonPath('$.data[*].title');
```

这也是你在响应中看到的，但甚至有一个方法可以测试该记录是否现在也存在于数据库中。你应该自己尝试一下。你可以在这里找到它的文档：[`codeception.com/docs/modules/Db#seeInDatabase`](http://codeception.com/docs/modules/Db#seeInDatabase)。

# 其他测试用例

还有很多与其他帖子操作（端点）相关的测试用例。然而，编写测试用例的方式将保持不变。所以，我会跳过这部分，这样你就可以自己编写这些测试用例。不过，作为提示，以下是一些你应该练习编写的测试用例：

```php
tryToDeletePostWithWrongId()  and it should return 404 response.

tryToDeletePostWithCorrectId() and it should return 200 with JSON we set there in PostController delete() method.

tryToDeletePostWithIdBelongsToOtherUserPost() it should return 403 Forbidden response because a user is only allowed to delete his/her own Post.

tryToDeletePostWithoutLogin() it should return 401 Unauthenticated because only a logged in user is allowed to delete his/her Post.
```

然后关于更新帖子：

```php
tryToUpdatePostWithWrongId()  and it should return 404 response.

tryToUpdatePostWithCorrectId() and it should return 200 with JSON having that Post data.

tryToUpdatePostWithIdBelongsToOtherUserPost() it should return 403 Forbidden response because a user is only allowed to update his/her own Post.
tryToUpdatePostWithoutLogin() and it should return 401 unauthorized.
```

然后关于帖子列表：

```php
tryToListPosts() and it should return 200 response code with Post list having data and meta indices in JSON.
```

然后关于获取单个帖子：

```php
tryToSeePostWithId() and it should return 200 response code with Post data in JSON.

tryToSeePostWithInvalidId() and it should return 404 Not Found error.
```

因此，我强烈建议你编写这些测试用例。如果你需要更多示例，或者想要查看与身份验证相关的端点测试示例，那么你可以在这里找到一些示例，以便更好地理解：[`github.com/Haafiz/REST-API-for-basic-RPG/tree/master/tests/api`](https://github.com/Haafiz/REST-API-for-basic-RPG/tree/master/tests/api)。

有关 CodeCeption 的更多信息，请参阅 CodeCeption 文档：[`codeception.com/`](http://codeception.com/)。

# 总结

在本章中，我们学习了测试类型，自动化测试的重要性，并为我们的 RESTful Web 服务端点编写了 API 测试。我再次想说的一件事是，我们只编写了 API 测试，以保持专注在我们的主题上，但单元测试同样重要。然而，测试是一个庞大的主题，单元测试有其自身的复杂性，所以无法在这一章中讨论。

# 更多资源

如果你想了解更多关于 PHP 自动化测试的信息，那么这里有一些重要的资源。

《Test Driven Laravel》（Adam Wathan 的视频课程）[`adamwathan.me/test-driven-Laravel/`](https://adamwathan.me/test-driven-laravel/)，然而这主要是关注于 Laravel 的。但是，这也会教给你一些重要的东西。

同样地，Jeffrey Way 的旧书《Laravel Testing Decoded》可以在[`leanpub.com/Laravel-testing-decoded`](https://leanpub.com/laravel-testing-decoded)找到。

再次强调，这是一本专门针对 Laravel 的书，但在一般情况下也会教给你很多东西。Jeffrey Way 即将推出的新书是关于 PHP 测试的，名为《Testing PHP》：[`leanpub.com/testingphp`](https://leanpub.com/testingphp)

前面提到的关于 PHP 的书还没有完成，所以你可以从 Jeffrey Way 的精彩的视频测试中学习：[`laracasts.com/skills/testing`](https://laracasts.com/skills/testing)。事实上，Laracasts 不仅适用于测试，还适用于全面学习 PHP 和 Laravel。

无论你选择哪个来源，重要的是你要练习。这对于开发和测试都是如此。事实上，如果你以前没有进行过测试，那么练习测试就更加重要。起初，你可能会感到有些不知所措，但这是值得的。
