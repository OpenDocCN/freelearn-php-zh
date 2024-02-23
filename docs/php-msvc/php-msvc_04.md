# 第四章：测试和质量控制

在本章中，我们将看一下在开发过程之前、期间和之后可以使用的不同测试方法。正如你所知，测试你的应用程序可以避免未来出现问题，并为你提供更好的项目概述。

# 在你的应用程序中使用测试的重要性

在我们的应用程序中使用测试非常重要，因为这些步骤可以避免（或至少减少）未来可能出现的问题或错误，因为我们是人类，在开发过程中可能会犯错误，或者因为项目结构不正确，甚至开发人员的理解与客户的要求不符。

测试过程将有助于提高代码质量和功能的理解，进行回归测试以避免在持续集成中包含旧问题，并减少完成项目所需的时间。

测试用于减少应用程序中的失败或错误。开发团队花费大量时间进行错误修复，根据错误的发现时间不同，影响可能会更大或更小。以下图片显示了与开发阶段相关的错误修复的相对成本：

![在应用程序中使用测试的重要性](img/B06142_04_01.jpg)

在开发中使用测试方法的原因是我们可以在开发的早期阶段发现代码中的错误，这样我们将花费更少的时间进行错误修复。

## 微服务测试

基于微服务的应用程序测试的挑战不在于测试每个单独的微服务，而在于集成和数据一致性。基于微服务的应用程序将需要开发人员对微服务架构及其工作流程有更好的理解，以便能够在其上进行测试。这是因为需要检查每个微服务的信息和功能，以及微服务之间的通信点。

在微服务上使用的测试如下：

+   **单元测试**：在所有基于微服务的应用程序中，甚至在单体应用程序中，都需要使用单元测试。通过使用它，我们将检查方法或代码模块的必要功能。

+   **集成测试**：单元测试仅检查孤立的组件，因此我们还需要检查方法之间的行为。我们将使用集成测试来检查同一微服务中方法之间的行为，因此微服务之间的调用将需要被模拟。

+   **API 测试**：微服务架构依赖于它们之间的通信。对于每个微服务，需要建立一个 API；这就像使用该微服务的*合同*。通过这种测试，我们将检查每个微服务的合同是否有效，并且所有微服务是否互相配合。

+   **端到端测试**：这些测试保证了应用程序的质量，没有任何模拟方法或调用。将运行测试来评估所有必需微服务之间的功能。在这些测试期间有一些规则可以避免问题：

+   端到端测试很难维护，因此只测试最重要的功能；其余功能使用单元测试

+   通过模拟对微服务的调用，可以测试用户功能

+   测试环境必须保持清洁，因为测试非常依赖数据，因此先前的测试可能会操纵数据，然后下一个测试

一旦我们知道如何根据微服务进行应用程序测试，我们将看一些在开发过程中进行测试的策略。

# 测试驱动开发

**测试驱动开发**（**TDD**）是敏捷哲学的一部分，它似乎解决了应用程序不断发展和成长以及代码变得混乱时常见的开发人员问题。开发人员修复问题以使其运行，但我们添加的每一行代码都可能是一个新的错误，甚至可能破坏其他功能。

TDD 是一种学习技术，它帮助开发人员以迭代、增量和建构主义的方式了解他们将构建的应用程序的领域问题：

+   **迭代**，因为该技术始终重复相同的过程以获得价值

+   **增量**，因为对于每个迭代，我们有更多的单元测试可供使用

+   **建构主义**，因为我们可以立即测试我们正在开发的一切，以便我们可以获得即时反馈

此外，当我们完成每个单元测试或迭代开发后，我们可以忘记它，因为它将在整个开发过程中保留，帮助我们通过单元测试记住领域问题。这对健忘的开发人员是一个很好的方法。

非常重要的是要理解 TDD 包括四个方面：分析、设计、开发和测试。换句话说，进行 TDD 就是理解领域问题并正确分析问题，良好设计应用程序，良好开发和测试。需要明确的是，TDD 不仅仅是实现单元测试，而是整个软件开发过程。

TDD 完全匹配基于微服务的项目，因为在大型项目中使用微服务是将其分成小微服务，我们的功能就像由通信渠道连接的小项目的聚合。项目的大小与使用 TDD 无关，因为在这种技术中，您将每个功能划分为小示例，为此，项目的大小无关紧要，甚至当我们的项目被微服务分割时更是如此。此外，微服务仍然比单块项目更好，因为单元测试的功能是在微服务中组织的，这将帮助开发人员知道他们可以从哪里开始使用 TDD。

## 如何进行 TDD？

进行 TDD 并不难，我们只需要遵循一些步骤并通过改进我们的代码来重复它们，并检查我们没有破坏任何东西：

1.  **编写单元测试**：它需要是可能的最简单和最清晰的测试，并且一旦完成，它必须失败；这是强制性的，如果它没有失败，那意味着我们做错了什么。

1.  **运行测试**：如果有错误（测试失败），这是开发最小代码以通过测试的时刻；只需做必要的事情，不要编写额外的代码。一旦开发了最小代码以通过测试，再次运行测试；如果通过了，就进入下一步，如果没有，修复它并再次运行测试。

1.  **改进测试**：如果您认为可以改进您编写的代码，那就去做并再次运行测试。如果您认为它是完美的，那就编写一个新的单元测试。

以下图片说明了 TDD 的口号--**红**，**绿**，**重构**：

![如何进行 TDD？](img/B06142_04_02.jpg)

要进行 TDD，需要在实现函数之前编写测试；如果先开始实现然后再编写测试，那就不是 TDD，只是测试。

如果我们在开始开发应用程序后创建单元测试，那么我们正在进行经典测试，并且没有充分利用 TDD 的好处。编写单元测试将帮助您确保在开发过程中您对领域问题的抽象理解是正确的。

显然，进行测试总比不进行测试要好，但进行 TDD 仍然比仅进行经典测试要好。

## 为什么我应该使用 TDD？

TDD 是对问题的答案，比如“我应该从哪里开始？”，“我该如何做？”，“我如何编写可以修改而不会破坏任何东西的代码？”，以及“我如何知道我必须实现什么？”。

目标不是毫无意义地编写许多单元测试，而是根据要求正确设计 TDD。在 TDD 中，我们不是考虑实现功能，而是考虑与域问题相关的功能的良好示例，以消除域问题造成的歧义。

换句话说，我们应该通过 TDD 在 X 个示例中复制特定功能或用例，直到我们得到必要的示例来描述该功能或任务，而不会产生歧义或误解。

### 提示

TDD 可能是记录应用程序的最佳方法。

使用其他软件开发方法时，我们开始思考架构将是什么样子，将使用什么模式，微服务之间的通信将如何进行，但如果一旦我们计划了所有这些，我们意识到这是不必要的呢？我们要花多长时间才能意识到？我们将花费多少精力和金钱？

TDD 通过在许多迭代中创建小示例来定义我们应用程序的架构，直到我们意识到什么是架构。这些示例将逐渐向我们展示应该遵循的步骤，以便定义最佳结构、模式或要使用的工具，从而避免在应用程序的最初阶段浪费资源。

这并不意味着我们在没有架构的情况下工作。显然，我们必须知道我们的应用程序是网站还是移动应用，并使用适当的框架（您可以在第二章中了解哪种框架适合您的需求，*开发环境*），还要知道应用程序中的互操作性将是什么；在我们的情况下，它将是基于微服务的应用程序。因此，它将支持我们开始创建第一个单元测试。TDD 将成为我们开发应用程序的指南，并且将通过单元测试产生一个没有歧义的架构。

TDD 并非万能良药；换句话说，它对资深开发人员和初级开发人员的效果并不相同，但对整个团队都是有用的。让我们看看使用 TDD 的一些优势：

+   代码重用：这样可以仅使用必要的代码来通过第二阶段（绿色）的测试来创建每个功能。它可以让你看到是否有更多的功能使用相同的代码结构或特定功能的部分；因此，它可以帮助你重用先前编写的代码。

+   团队合作更容易：它让你对团队同事充满信心。一些架构师或资深开发人员不信任经验不足的开发人员，他们需要在提交更改之前检查他们的代码，从而在这一点上造成瓶颈，因此 TDD 帮助我们相信经验较少的开发人员。

+   增加沟通：增加团队同事之间的沟通。沟通更加流畅，因此团队可以分享他们在单元测试中反映的对项目的知识。

+   避免过度设计：不要在最初阶段过度设计应用程序。正如我们之前所说，进行 TDD 可以让你逐渐了解应用程序的概况，避免在项目中创建无用的结构或模式，也许在将来的阶段会被废弃。

+   单元测试是最好的文档：了解特定功能的最佳方法是阅读其单元测试，这有助于我们理解它的工作原理，而不是人类的语言。

+   在设计阶段发现更多用例：在每个测试中，您都将了解功能应该如何更好地工作，以及功能可能具有的所有可能阶段。

+   **增加了工作完成的感觉**：在每次提交代码时，您会感到它被正确地完成了，因为其余的单元测试都通过了，所以您不必担心破坏其他功能。

+   **提高软件质量**：在重构步骤中，我们努力使代码更高效和可维护，验证在更改后整个项目仍然正常工作。

## TDD 算法

遵循 TDD 算法的技术概念和步骤是简单明了的，通过实践来改进实现它的正确方式。正如我们之前所看到的，只有三个步骤：红、绿和重构。

### 红 - 编写单元测试

即使代码尚未编写，也可以编写测试，您只需考虑是否可以在实现之前编写规范。因此，在第一步中，您应该考虑开始编写的单元测试不像单元测试，而像功能的示例或规范。

在 TDD 中，这个示例或规范并不是固定的；换句话说，单元测试可以在将来进行修改。在开始编写第一个单元测试之前，需要考虑**被测试软件**（**SUT**）将是什么样子，以及它将如何工作。我们需要考虑 SUT 代码将是什么样子，以及我们将如何检查它是否按我们想要的方式工作。

TDD 的工作方式首先让我们设计更舒适和清晰的东西，如果它符合要求的话。

### 绿 - 使代码工作

一旦示例编写完成，我们必须编写最少的代码使其通过测试；换句话说，设置单元测试为绿色。代码是否丑陋且未经优化并不重要，这将是我们在接下来的步骤和迭代中的任务。

在这一步中，重要的是只编写满足要求的必要代码，而不是不必要的东西。这并不意味着不考虑功能性地编写，而是考虑到它的高效性。看起来很容易，但您会意识到第一次会写出额外的代码。

如果您专注于这一步，您将考虑到关于 SUT 行为的不同输入的新问题。然而，您应该坚定不移地避免编写与当前功能相关的其他功能的额外代码。作为一个经验法则，不要编写新功能，而是做笔记，以便在未来的迭代中将它们转换为功能。

### 重构 - 消除冗余

重构不同于重写代码。您应该能够在不改变行为的情况下改变设计。

在这一步中，您应该消除代码中的重复，并检查代码是否符合良好实践的原则，考虑效率、清晰度和代码的未来可维护性。这部分取决于每个开发人员的经验。

### 提示

良好重构的关键是采取小步骤。

重构功能的最佳方式是改变一小部分并执行所有可用的测试，如果它们通过了，继续进行下一个小部分，直到你对得到的结果满意。

# 行为驱动开发

**行为驱动开发**（**BDD**）是一种扩展 TDD 技术并将其与其他设计思想和业务分析相结合的过程，以便提供给开发人员以改进软件开发。

在 BDD 中，我们测试场景和类的行为，以满足可以由许多类组成的场景。

使用 DSL 非常有用，以便客户、项目所有者、业务分析师或开发人员使用共同的语言。目标是拥有一个像我们在第三章中看到的那样的普遍语言，*应用设计*，在领域驱动设计部分。

## 什么是 BDD？

正如我们之前所说，BDD 是一种基于 TDD 和 ATDD 的敏捷技术，促进了项目整个团队之间的协作。

BDD 的目标是让整个团队了解客户的需求，让客户知道团队其他成员从他们的规范中理解了什么。大多数情况下，当项目开始时，开发人员和客户的观点并不相同，在开发过程中客户意识到也许他们没有解释清楚，或者开发人员没有正确理解，因此需要更多时间来更改代码以满足客户的需求。

因此，BDD 是使用规则或通用语言以人类语言编写测试用例，以便客户和开发人员能够理解。它还为测试定义了 DSL。

## 它是如何工作的？

需要将功能定义为用户故事（我们将在本章的 ATDD 部分解释这是什么），并检查它们的验收标准。

一旦用户故事被定义，我们必须专注于使用 DSL 描述项目行为的可能场景。步骤是：给定（上下文），当（事件发生），然后（结果）。

总之，为用户故事定义的场景为验收标准提供了检查功能是否完成的依据。

### Cucumber 作为 BDD 的 DSL

Cucumber 是一个 DSL 工具，它执行以纯文本形式制作的示例作为自动测试，利用 BDD 的好处，将业务层和技术结合在一起，以了解用户最看重的功能，并在定义用例测试和记录项目的同时开发它们。

### 提示

Cucumber 最重要的是让开发人员和客户有相同的理解。

**Gherkin**是 Cucumber 使用的语言，它允许您将项目的规范翻译成接近人类语言，以便客户或其他没有技术技能的人能够理解。这个工具和语言可以用于 BDD 和 ATDD。让我们看一个样本代码：

```php
    Feature: Search secrets 
     In order to find secrets 
     Users should be able to search for near secrets 

     Scenario: Search secrets by distance 
       Given there are 996 secrets in the game which are no closer than 100 
       meters from me 
       And there are 4 secrets SEC001, SEC005, SEC054, SEC121 that are 
       within 100 
       meters from me 
       When I search for closer secrets 
       Then I should see the following secrets: 
         | Secret code | 
         | SEC001      | 
         | SEC005      | 
         | SEC054      | 
         | SEC121      | 

```

这样可以让我们定义软件行为，而不用说出它是如何实现的。同时，它也让我们能够在编写自动测试用例的同时记录功能。使用 Cucumber 的优势如下：

+   易于阅读

+   易于理解

+   易于解析

+   易于讨论

DSL 在代码中有三个工具可以理解和处理的步骤；它们如下：

1.  **给定**：这是将系统设置为适当状态以检查测试的必要步骤。

1.  **当**：这是用户必须执行的必要步骤来激活功能。

1.  **然后**：这指的是系统中发生变化的事物。在这里，我们能够看到它是否符合我们的期望。

此外，还有两个可选的步骤：**And**和**But**，它们可以在**Given**或**Then**中使用，当您需要超过一句话来满足要求时。

在本章中，我们将看到如何使用一个名为 Selenium 的工具来进行 BDD。这是另一个 DSL 工具，但是它是面向 Web 开发而不是纯文本的。

# 验收测试驱动开发

也许项目中最重要的方法是**验收测试驱动开发**（**ATDD**）或**故事测试驱动开发**（**STDD**）；它是 TDD，但在不同的层面上。

验收（或客户）测试是项目满足客户需求的业务要求的书面标准。它们是由项目所有者编写的示例（就像 TDD 中的示例）。它是每次迭代开发的开始，是 Scrum 和敏捷开发之间的桥梁。

在 ATDD 中，我们以一种与传统方法不同的方式开始项目的实施。用人类语言编写的业务需求被一些团队成员和客户商定的可执行文件所取代。这并不是要替换整个文档，而只是部分需求。

使用 ATDD 的优势如下：

+   它提供了真实的例子和一个团队可以理解领域的共同语言

+   它使我们能够正确识别领域规则

+   可以在每次迭代中知道用户故事是否完成

+   工作流程从最初的步骤开始

+   开发直到团队定义并接受了测试才开始

## 用户故事

ATDD 的用户故事在名称或描述方面类似于用例，但工作方式不同。用户故事不定义需求，避免了人类语言的歧义问题。目标是让团队的其他成员能够无问题地理解这个想法。

每个用户故事都是关于客户对应用程序的期望的清晰简洁的例子列表。故事的名称是一个人类语言的句子，定义了功能必须做什么。考虑以下例子：

+   搜索我们位置周围的可用秘密

+   检查我们已经存储的秘密

+   检查战斗中谁是赢家

他们的目标是倾听客户并帮助他们定义他们对应用程序的期望。用户故事应该清晰明了，没有歧义，并且应该用人类语言而不是技术语言编写；客户应该能够理解他们所说的话。

一旦我们定义了一个用户故事，就会出现一些问题，这些问题应该通过为每个故事关联验收测试来回答。例如，对于*检查战斗中谁是赢家*的故事，一些可能的问题如下所列：

+   如果他们平局了会发生什么？

+   赢家会得到什么？

+   输家会失去什么？

+   一场战斗需要多长时间？

可能的验收测试如下：

+   如果他们平局了，没有人会赢或输任何东西；他们会保留他们的秘密

+   赢家将获得 10 分，并从输家的口袋里得到一个秘密

+   输家将给赢家一个秘密

+   一场战斗需要掷三次骰子

也许一个用户故事的问题和答案会产生新的用户故事，添加到待办列表中。

## ATDD 算法

ATDD 的算法类似于 TDD，但覆盖的人员比开发人员更多。换句话说，进行 ATDD 时，每个故事的测试是在一个会议上编写的，该会议包括项目所有者、开发人员和 QA 技术人员，因为团队必须理解需要做什么以及为什么需要这样做，以便他们可以看到代码应该做什么。以下图片显示了 ATDD 的循环：

![ATDD 算法](img/B06142_04_03.jpg)

### 讨论

ATDD 算法的起点是讨论。在这一步中，业务与客户进行会议，澄清应用程序应该如何工作，分析师应该从对话中创建用户故事。此外，他们应该能够解释每个用户故事的满意条件，以便被翻译成例子，就像我们在用户故事部分解释的那样。

会议结束时，例子应该清晰简洁，这样我们就可以得到一个用户故事的例子列表，以满足客户审查和理解的所有需求。此外，团队将对项目有一个概览，以便理解用户故事的业务价值，如果用户故事太大，可以将其分成小的用户故事，获得第一个迭代的第一个用户故事。

### 提炼

高级验收测试由客户和开发团队编写。在这一步中，从讨论步骤中得到的测试用例的编写开始，并且团队可以参与讨论并帮助澄清信息或指定其真实需求。

测试应该覆盖在讨论步骤中发现的所有示例，并在这个过程中可以添加额外的测试；一点一点地我们正在更好地理解功能。

在这一步结束时，我们将获得以人类语言编写的必要测试，以便团队（包括客户）能够理解他们在下一步将要做什么。这些测试可以用作文档。

### 开发

在开发步骤中，验收测试用例开始由开发团队和项目所有者开发。在这一步骤中要遵循的方法与 TDD 相同--开发人员应该创建一个测试并观察它失败（红色），然后开发最少的代码行以通过测试（绿色）。一旦验收测试通过，就应该进行验证和测试，准备好交付。

在这个过程中，开发人员可能会发现需要添加到测试中的新场景，甚至，如果需要大量工作，它可能会被推到用户故事中。

在这一步结束时，我们将拥有一个通过验收测试的软件，甚至可能还有更全面的测试。

### 演示

通过运行验收测试用例并手动探索新功能的特性来展示创建的功能。演示完毕后，团队讨论用户故事是否做得恰当，是否满足产品所有者的需求，并决定是否可以继续下一个故事。

# 工具

现在您已经更多地了解了 TDD 和 BDD，是时候解释一些可以在开发工作流程中使用的工具了。有很多可用的工具，但我们只会解释最常用的工具。

## Composer

Composer 是一个用于管理软件依赖关系的 PHP 工具。您只需要声明项目所需的库，Composer 将管理它们，在必要时安装和更新它们。这个工具只有一些要求--如果您有 PHP 5.3.2+，您就可以开始了。如果缺少某个要求，Composer 会提醒您。

您可以在开发机器上安装这个依赖管理器，但由于我们使用的是 Docker，我们将直接在我们的**PHP-FPM**（**FastCGI 进程管理器**）容器中安装它。在 Docker 中安装 Composer 非常容易；您只需要向 Dockerfile 添加以下规则：

```php
    RUN curl -sS https://getcomposer.org/installer 
    | php -- --install-dir=/usr/bin/ --filename=composer 

```

## PHPUnit

我们项目中需要的另一个工具是 PHPUnit，一个单元测试框架。在我们的情况下，我们将使用 4.0 版本。与之前一样，我们将把这个工具添加到我们的 PHP-FPM 容器中，以保持我们的开发机器干净。如果您想知道为什么除了 Docker 之外我们不在开发机器上安装任何东西，答案很明确--将所有东西放在容器中将帮助您避免与其他项目的冲突，并且可以灵活地更改版本而不必过于担心。

作为一个快速的方法，您可以在您的`PHP-FPM` `Dockerfile`中添加以下`RUN`命令，这样您就可以安装并准备使用最新的 PHPUnit 版本了：

```php
    RUN curl -sSL https://phar.phpunit.de/phpunit.phar -o 
    /usr/bin/phpunit && chmod +x /usr/bin/phpunit 

```

上述命令将在您的容器中安装最新的 Composer 版本，但推荐的安装方式是通过 Composer。打开您的`composer.json`文件并添加以下行：

```php
    "phpunit/phpunit": "4.0.*",
```

一旦您更新了`composer.json`文件，您只需要在容器命令行中执行 Composer 更新，Composer 就会为您安装 PHPUnit。

既然我们的要求都准备好了，现在是时候安装我们的 PHP 框架并开始做一些 TDD 的工作了。稍后，我们将继续更新我们的 Docker 环境，加入新的工具。

在前几章中，我们谈到了一些 PHP 框架，并选择了 Lumen 作为我们的示例。请随意将所有示例调整为您喜欢的框架。我们的源代码将存储在容器中，但在开发的这一阶段，我们不希望容器是不可变的。我们希望我们对代码所做的每一次更改都能立即在我们的容器中使用，因此我们将使用容器作为存储卷。

要创建一个包含我们的源代码并将其用作存储卷的容器，我们只需要编辑我们的`docker-compose.yml`文件，并为我们的每个微服务创建一个源容器，如下所示：

```php
    source_battle: 
       image: nginx:stable 
       volumes: 
           - ../source/battle:/var/www/html 
       command: "true" 

```

上述代码片段创建了一个名为`source_battle`的容器映像，并存储了我们的 battle 源代码（位于`docker-compose.yml`文件当前路径的`../source/battle`）。一旦我们有了我们的源代码容器，我们可以编辑每个服务并分配一个卷。例如，我们可以将以下行添加到我们的`microservice_battle_fpm`和`microservice_battle_nginx`容器描述中：

```php
    volumes_from: 
               - source_battle 

```

我们的 battle 源代码将在我们的源容器的`/var/www/html`路径中可用，安装 Lumen 的剩下步骤是执行一个简单的 Composer 命令。首先，您需要确保您的基础设施已经准备好了：

```php
**$ docker-compose up**

```

上述命令启动我们的容器并将日志输出到标准 IO。现在我们确信一切都已经准备就绪，我们需要进入我们的 PHP-FPM 容器并安装 Lumen。

### 提示

如果您需要知道每个容器分配的名称，可以在终端上执行`docker ps`并复制容器名称。例如，我们将输入以下命令进入 battle PHP-FPM 容器：

```php
**$ docker exec -it docker_microservice_battle_fpm_1 /bin/bash**

```

上述命令在您的容器中打开一个交互式 shell，以便您可以做任何您想做的事情。让我们用一个命令安装 Lumen：

```php
**# cd /var/www/html && composer create-project --prefer-dist laravel/lumen .**

```

对每个微服务重复上述命令。

现在您已经准备好开始进行单元测试并编写应用程序代码了。

### 单元测试

单元测试是一小段代码，它在已知的上下文中使用其他代码，以便我们可以检查我们正在测试的代码是否有效。Lumen 默认带有 PHPUnit；因此，我们只需要将所有测试添加到 tests 文件夹中。框架安装默认带有一个非常小的示例文件--`ExampleTest.php`--您可以在其中尝试单元测试。为了开始单元测试，直到您更加熟悉创建单元测试，选择一个您的微服务源代码，并创建`app/Dummy.php`文件，内容如下：

```php
<?php 
namespace App;

class Dummy 
{ 
} 

```

### 提示

开始单元测试的最简单方法是每次在代码中创建一个新类时，您都可以为测试创建一个新类。以这种方式工作，您将记住您的新类需要用单元测试进行覆盖。例如，想象一下您需要一个`Battle`类；因此，当您创建该类时，您还可以在`tests`文件夹中创建一个以`Test`为前缀的新类。

在理想的情况下，所有代码都应该由单元测试覆盖，但我们知道这是一个奇怪的情况。大多数情况下，如果幸运的话，您的代码覆盖率将达到 70%或 80%。我们鼓励您保持代码完全覆盖，但如果不可能，至少覆盖核心功能。有两种创建单元测试的方法：

+   **先测试，后编码：**在我们看来，当您有足够的时间开发项目时，这种工作流程更好。首先，创建测试，以确保您真正了解每个新功能。在测试就位后，您将编写必要的最小代码以通过测试。以这种方式编码，您将思考什么使您的代码有效，以及什么可能使您的代码失败。

+   **先写代码，后写测试：** 当您没有太多时间进行单元测试时，这是一个非常常见的工作流程。您像往常一样创建您的代码，一旦完成，就创建单元测试。这种方法会创建一个不太健壮的代码，因为您是将单元测试适应已创建的代码，而不是相反。

请记住，测试代码的时间非常重要；这是一项长期投资。在开始时花费时间将使您的代码更加健壮，并消除未来的错误。

### 运行测试

您可能想知道如何运行和检查您的测试。别担心，这很简单。您只需要进入您的 PHP-FPM 容器之一。例如，要进入 Battle PHP-FPM 容器，请打开终端并执行以下命令：

```php
**$ docker exec -it docker_microservice_battle_fpm_1 /bin/bash**

```

执行上述命令后，您将进入容器。现在是时候确保您的当前路径是`/var/www/html`文件夹。完成上一步后，您可以在该文件夹中执行 phpunit。所有这些操作都可以使用以下命令完成：

```php
**# cd /var/www/html**
**# ./vendor/bin/phpunit**

```

`phpunit`命令将读取`phpunit.xml`文件。这个 XML 描述了我们的测试存储在哪里，并执行所有测试。执行此命令将为我们提供一个漂亮的屏幕，显示我们的测试通过或失败的结果。

### 断言

断言是在已知上下文中的语句，我们期望在代码中的某个时刻为真，并且这是单元测试的核心。断言用于测试用例内，一个测试用例可以包含多个断言。在 PHPUnit 中，创建测试非常简单，您只需要在方法名称前添加`test`前缀。简单吧？为了澄清所有这些概念，让我们看一些您可以在单元测试中使用的断言及其示例。随时创建更复杂的测试，直到您熟悉 PHPUnit 为止。

### assertArrayHasKey

`assertArrayHasKey(mixed $key, array $array[, string $message = ''])`断言检查`$array`是否具有`$key`元素。想象一下，您有一个生成并返回某种配置数据的方法，并且有一个特定由`storage`标识的元素，您需要确保它始终存在。将以下方法添加到我们的`Dummy`类中以模拟配置生成：

```php
    public static function getConfigArray() 
    { 
      return [ 
           'debug'   => true, 
           'storage' => [ 
               'host' => 'localhost', 
               'port' => 5432, 
               'user' => 'my-user', 
               'pass' => 'my-secret-password' 
           ] 
       ]; 
    } 

```

现在我们可以以任何方式测试此`getConfigArray`的响应：

```php
    public function testFailAssertArrayHasKey() 
    { 
       $dummy = new App\Dummy(); 

       $this->assertArrayHasKey('foo', $dummy::getConfigArray()); 
    } 

```

上面的测试将检查`getConfigArray`返回的数组是否具有由`foo`标识的元素，在我们的示例中失败了：

```php
    public function testPassAssertArrayHasKey() 
    { 
       $dummy = new App\Dummy(); 

       $this->assertArrayHasKey('storage', $dummy::getConfigArray()); 
    } 

```

在这种情况下，此测试将确保`getConfigArray`返回由`storage`标识的元素。如果由于某种原因您将来更改`getConfigArray`方法的实现，此测试将帮助您确保您至少继续接收由`storage`标识的数组元素。

你可以使用`assertArrayNotHasKey()`作为`assertArrayHasKey`的反向操作；它使用相同的参数。

### assertClassHasAttribute

`assertClassHasAttribute(string $attributeName, string $className[, string $message = ''])`断言检查我们的`$className`是否已定义`$attributeName`。修改我们的`Dummy`类并添加一个新属性，如下所示：

```php
    public $foo; 

```

现在我们可以使用以下测试来测试此公共属性的存在：

```php
    public function testAssertClassHasAttribute() 
    { 
      $this->assertClassHasAttribute('foo', App\Dummy::class); 
      $this->assertClassHasAttribute('bar', App\Dummy::class); 
    } 

```

上面的代码将通过`foo`属性的检查，但在检查`bar`属性时将失败。

您可以使用`assertClassNotHasAttribute()`作为`assertClassHasAttribute`的反向操作；它使用相同的参数。

### assertArraySubset

`assertArraySubset(array $subset, array $array[, bool $strict = '', string $message = ''])`断言检查给定的`$subset`是否在我们的`$array`中可用：

```php
    public function testAssertArraySubset() 
    { 
       $dummy = new App\Dummy(); 

       $this->assertArraySubset(['storage' => 'failed-test'], 
       $dummy::getConfigArray()]); 
    } 

```

上面的示例测试将失败，因为`['storage' => 'failed-test']`子集不存在于我们的`getConfigArray`方法的响应中。

### assertClassHasStaticAttribute

`assertClassHasStaticAttribute(string $attributeName, string $className[, string $message = ''])`断言检查给定`$className`中静态属性的存在。我们可以向我们的`Dummy`类添加一个静态属性，如下所示：

```php
    public static $availableLocales = [ 
           'en_GB', 
           'en_US', 
           'es_ES', 
           'gl_ES' 
    ]; 

```

有了这个静态属性，我们可以自由地测试`$availableLocales`的存在：

```php
    public function testAssertClassHasStaticAttribute() 
    { 
      $this->assertClassHasStaticAttribute('availableLocales', 
      App\Dummy::class); 
    } 

```

如果需要断言相反的情况，可以使用`assertClassNotHasStaticAttribute();`它使用相同的参数。

### assertContains()

有时您需要检查一个集合是否包含特定元素。您可以使用`assertContains()`函数来实现：

+   `assertContains(mixed $needle, Iterator|array $haystack[, string $message = ''])`

+   `assertNotContains(mixed $needle, Iterator|array $haystack[, string $message = ''])`

+   `assertContainsOnly(string $type, Iterator|array $haystack[, boolean $isNativeType = null, string $message = ''])`

+   `assertNotContainsOnly(string $type, Iterator|array $haystack[, boolean $isNativeType = null, string $message = ''])`

+   `assertContainsOnlyInstancesOf(string $classname, Traversable|array $haystack[, string $message = ''])`

### assertDirectory()和 assertFile()

PHPUnit 不仅允许您测试应用程序的逻辑，还可以测试文件夹和文件的存在和权限。所有这些都可以通过以下断言实现：

+   `assertDirectoryExists(string $directory[, string $message = ''])`

+   `assertDirectoryNotExists(string $directory[, string $message = ''])`

+   `assertDirectoryIsReadable(string $directory[, string $message = ''])`

+   `assertDirectoryNotIsReadable(string $directory[, string $message = ''])`

+   `assertDirectoryIsWritable(string $directory[, string $message = ''])`

+   `assertDirectoryNotIsWritable(string $directory[, string $message = ''])`

+   `assertFileEquals(string $expected, string $actual[, string $message = ''])`

+   `assertFileNotEquals(string $expected, string $actual[, string $message = ''])`

+   `assertFileExists(string $filename[, string $message = ''])`

+   `assertFileNotExists(string $filename[, string $message = ''])`

+   `assertFileIsReadable(string $filename[, string $message = ''])`

+   `assertFileNotIsReadable(string $filename[, string $message = ''])`

+   `assertFileIsWritable(string $filename[, string $message = ''])`

+   `assertFileNotIsWritable(string $filename[, string $message = ''])`

+   `assertStringMatchesFormatFile(string $formatFile, string $string[, string $message = ''])`

+   `assertStringNotMatchesFormatFile(string $formatFile, string $string[, string $message = ''])`

您的应用程序是否依赖于可写文件才能工作？别担心，PHPUnit 会帮你解决。您可以在测试中添加`assertFileIsWritable()`，这样下次有人删除您在断言中指定的文件时，测试将失败。

### assertString()

在某些情况下，您需要检查一些字符串的内容。例如，如果您的代码生成序列码，您可以检查生成的代码是否符合您的规格。您可以使用以下断言来处理字符串：

+   `assertStringStartsWith(string $prefix, string $string[, string $message = ''])`

+   `assertStringStartsNotWith(string $prefix, string $string[, string $message = ''])`

+   `assertStringMatchesFormat(string $format, string $string[, string $message = ''])`

+   `assertStringNotMatchesFormat(string $format, string $string[, string $message = ''])`

+   `assertStringEndsWith(string $suffix, string $string[, string $message = ''])`

+   `assertStringEndsNotWith(string $suffix, string $string[, string $message = ''])`

### assertRegExp()

`assertRegExp(string $pattern, string $string[, string $message = ''])`断言对您非常有用，因为您可以在一个断言中使用所有的正则表达式功能。让我们向我们的 Dummy 类添加一个静态函数：

```php
    public static function getRandomCode() 
    { 
      return 'CODE-123A'; 
    } 

```

这个新函数返回一个静态字符串代码。随意增加生成的复杂性。要测试这个生成的字符串代码，您现在可以在测试类中做如下操作：

```php
    public function testAssertRegExp() 
    { 
       $this->assertRegExp('/^CODE\-\d{2,7}[A-Z]$/', 
       App\Dummy::getRandomCode()); 
    } 

```

正如您所看到的，我们正在使用简单的正则表达式来检查`getRandomCode`生成的输出。

### assertJson()

在使用微服务时，您可能会与 JSON 请求和响应密切合作。因此，非常重要的是您有能力测试我们的 JSON。您可以将 JSON 作为文件或字符串：

+   `assertJsonFileEqualsJsonFile()`

+   `assertJsonStringEqualsJsonFile()`

+   `assertJsonStringEqualsJsonString()`

### 布尔断言

可以使用以下方法检查布尔结果或类型：

+   `assertTrue(bool $condition[, string $message = ''])`

+   `assertFalse(bool $condition[, string $message = ''])`

### 类型断言

有时您需要确保元素是特定类的实例或具有特定的内部类型。您可以在测试中使用以下断言：

+   `assertInstanceOf($expected, $actual[, $message = ''])`

+   `assertInternalType($expected, $actual[, $message = ''])`

### 其他断言

PHPUnit 具有大量的断言，如果没有以下一些断言应用于您的功能的结果或对象状态，您的测试将无法完成：

+   `assertCount($expectedCount, $haystack[, string $message = ''])`

+   `assertEmpty(mixed $actual[, string $message = ''])`

+   `assertEquals(mixed $expected, mixed $actual[, string $message = ''])`

+   `assertGreaterThan(mixed $expected, mixed $actual[, string $message = ''])`

+   `assertGreaterThanOrEqual(mixed $expected, mixed $actual[, string $message = ''])`

+   `assertInfinite(mixed $variable[, string $message = ''])`

+   `assertLessThan(mixed $expected, mixed $actual[, string $message = ''])`

+   `assertLessThanOrEqual(mixed $expected, mixed $actual[, string $message = ''])`

+   `assertNan(mixed $variable[, string $message = ''])`

+   `assertNull(mixed $variable[, string $message = ''])`

+   `assertObjectHasAttribute(string $attributeName, object $object[, string $message = ''])`

+   `assertSame(mixed $expected, mixed $actual[, string $message = ''])`

您可以在 PHPUnit 网站上找到有关您可以在其中使用的断言的更多信息，即[`phpunit.de/`](https://phpunit.de/)。

### 从头开始的单元测试

此时，您可能对单元测试感到更加舒适，并且希望尽快开始编写您的应用程序，因此让我们开始测试吧！

我们的微服务应用程序使用地理定位来查找秘密和其他玩家。这意味着您的位置微服务将需要一种方法来计算两个地理空间点之间的距离。我们还需要根据起始点获取最接近的存储点的列表（它们可以是最接近的用户或秘密）。由于这是一个核心功能，您需要确保我们描述的内容经过充分测试。

在我们的应用程序中，定位有自己的服务。因此，使用您的 IDE 打开位置微服务的源代码，并创建`app/Http/Controllers/LocationController.php`文件，内容如下：

```php
    <?php 

    namespace App\Http\Controllers; 

    use Illuminate\Http\Request; 

    class LocationController extends Controller 
    { 

    } 

```

上述代码已在 Lumen 中创建了我们的位置控制器，并且正如我们之前提到的，一旦我们创建了这个类，我们需要为我们的单元测试创建一个类似的类。为了做到这一点，您只需要创建`tests/app/Http/Controllers/LocationControllerTest.php`文件。正如您所看到的，我们甚至在复制文件夹结构；这是最好的方法，可以轻松知道我们正在为哪个类进行测试。

我们希望开始为距离计算和允许我们根据特定地理位置点获取最接近的秘密的功能创建测试。一种方法是创建两个不同的测试。因此，请使用以下代码填充您的`LocationControllerTest.php`：

```php
    <?php 

    use Laravel\Lumen\Testing\DatabaseTransactions; 

    class LocationControllerTest extends TestCase 
    { 
      public function testDistance() 
      { 
      } 

      public function testClosestSecrets() 
      { 
      } 
    } 

```

我们没有向我们的测试类添加任何特殊内容，我们只声明了两个测试。

让我们从`testDistance()`开始。在这个测试中，我们希望确保给定两个地理空间点之间的计算距离对我们的目的来说足够准确。在单元测试中，你需要开始描述已知的场景——作为点，我们选择了伦敦（纬度：`51.50`，经度：`-0.13`）和阿姆斯特丹（纬度：`52.37`，经度：`4.90`）。这两个城市之间的已知距离大约为 358.06 公里，这是我们希望从我们的距离计算器中得到的结果。让我们用以下代码填充我们的测试：

```php
    public function testDistance() 
    { 
      $realDistanceLondonAmsterdam = 358.06; 

      $london = [ 
           'latitude'  => 51.50, 
           'longitude' => -0.13 
      ]; 

      $amsterdam = [ 
           'latitude'  => 52.37, 
           'longitude' => 4.90 
      ]; 

      $location = new App\Http\Controllers\LocationController(); 

      $calculatedDistance = $location->getDistance($london, $amsterdam); 

      $this->assertClassHasStaticAttribute('conversionRates', 
      App\Http\Controllers\LocationController::class); 
      $this->assertEquals($realDistanceLondonAmsterdam, 
                          $calculatedDistance); 
    } 

```

在上述代码片段中，我们定义了已知的场景，我们两点的位置和它们之间的已知距离。一旦我们准备好了已知的场景，我们创建了一个`LocationController`的实例，并使用定义的`getDistance`函数来获得我们想要测试的结果。一旦我们得到了结果，我们测试我们的`LocationController`是否有一个`conversionRate`静态属性，我们可以用它来将距离转换为不同的单位。我们最后并且最重要的断言检查了计算出的距离与这两点之间的已知距离之间的匹配。我们已经准备好了基本的测试，现在是时候开始编写我们的`getDistance`函数了。

两个地理空间点之间的距离计算可以用非常不同的方式计算。你可以在这里使用策略模式，但为了保持示例简单，我们将在控制器内的不同函数中编写不同的计算算法。

打开你的`LocationController`并添加一些辅助代码：

```php
    const ROUND_DECIMALS = 2; 

    public static $conversionRates = [ 
       'km'    => 1.853159616, 
       'mile'  => 1.1515 
    ]; 

    protected function convertDistance($distance, $unit = 'km') 
    { 
      switch (strtolower($unit)) { 
        case 'mile': 
          $distance = $distance * self::$conversionRates['mile']; 
          break; 
        default : 
          $distance = $distance * self::$conversionRates['km']; 
          break; 
      } 

      return round($distance, self::ROUND_DECIMALS); 
    } 

```

在上述代码中，我们定义了我们的转换率、一个我们可以用来四舍五入结果的常量，以及一个简单的转换函数。我们将稍后使用这个`convertDistance`函数。

我们计算距离的第一个方法是使用欧几里得函数来得到我们的结果。一个简单的实现如下所示：

```php
public function getEuclideanDistance($pointA, $pointB, $unit = 'km') 
    { 
       $distance = sqrt( 
           pow(abs($pointA['latitude'] - $pointB['latitude']), 2) + pow(abs($pointA['longitude'] - $pointB['longitude']), 2) 
       ); 

       return $this->convertDistance($distance, $unit); 
    } 

```

现在我们的算法准备好了，我们可以将其添加到我们的`getDistance`函数中，如下所示：

```php
    public function getDistance($pointA, $pointB, $unit = 'km') 
    { 
      return $this->getEuclideanDistance($pointA, $pointB, $unit); 
    } 

```

此时，你已经准备好了一切，可以开始测试了。进入位置容器，在`/var/www/html`中运行 PHPUnit。这是我们的第一次尝试；PHPUnit 的结果将是失败，应用程序的输出将告诉你问题所在。在我们的情况下，失败的主要原因是我们使用的算法对我们的应用程序来说不够准确。我们不能部署这个版本的应用程序，因为它未通过测试，我们必须更改我们的测试或实现测试的代码。

正如我们之前提到的，有多种计算两点之间距离的方法，每种方法都可能更或者更少准确。我们尝试的第一个实现失败了，因为它用于平面，而我们的世界是一个球体。

再次打开你的`LocationController`，并使用 haversine 计算创建一个新的距离实现：

```php
    public function getHaversineDistance($pointA, $pointB, $unit = 'km') 
    { 
      $distance = rad2deg( 
           acos( 
               (sin(deg2rad($pointA['latitude'])) * 
               sin(deg2rad($pointB['latitude']))) + 
               (cos(deg2rad($pointA['latitude'])) * 
               cos(deg2rad($pointB['latitude'])) * 
               cos(deg2rad($pointA['longitude'] - 
               $pointB['longitude']))) 
           ) 
       ) * 60; 

      return $this->convertDistance($distance, $unit); 
    } 

```

如你所见，这个距离计算函数稍微复杂一些，考虑了我们世界的球形形式。更改`getDistance`函数以使用我们的新算法：

```php
    return $this->getHaversineDistance($pointA, $pointB, $unit); 

```

现在再次运行 PHPUnit，一切应该没问题；测试将通过，我们的代码已经准备好投入生产。

使用单元测试和 TDD，流程总是一样的：

1.  创建测试。

1.  让你的代码通过测试。

1.  运行测试，如果测试失败，从第 2 步重新开始。

我们想要在我们的位置微服务中拥有的另一个功能是获取我们当前位置附近最近的秘密。打开`LocationControllerTest`文件并添加以下代码：

```php
    public function testClosestSecrets() 
    { 
      $currentLocation = [ 
           'latitude'  => 40.730610, 
           'longitude' => -73.935242 
      ]; 

      $location = new App\Http\Controllers\LocationController(); 

      $closestSecrets = $location->getClosestSecrets($currentLocation); 
      $this->assertClassHasStaticAttribute('conversionRates', 
      App\Http\Controllers\LocationController::class); 
      $this->assertContainsOnly('array', $closestSecrets); 
      $this->assertCount(3, $closestSecrets); 

       // Checking the first element 
       $currentElement = array_shift($closestSecrets); 
       $this->assertArraySubset(['name' => 'amber'], $currentElement); 

       // Second 
       $currentElement = array_shift($closestSecrets); 
       $this->assertArraySubset(['name' => 'ruby'], $currentElement); 

       // Third 
       $currentElement = array_shift($closestSecrets); 
       $this->assertArraySubset(['name' => 'diamond'], $currentElement); 
    } 

```

在上述代码片段中，我们定义了我们的当前位置（纽约），并要求我们的实现给我们一个最近秘密的列表。我们的位置实现将有一个秘密的缓存列表，我们知道它们的位置（这将帮助我们知道正确的顺序）。

打开`LocationController.php`，首先添加一个秘密的缓存列表。在现实世界中，我们没有硬编码的值，但对于测试目的来说，这已经足够了：

```php
    public static $cacheSecrets = [ 
       [ 
           'id' => 100, 
           'name' => 'amber', 
           'location' => ['latitude'  => 42.8805, 'longitude' => -8.54569, 
           'name'      => 'Santiago de Compostela'] 
       ], 
       [ 
           'id' => 100, 
           'name' => 'diamond', 
           'location' => ['latitude'  => 38.2622, 'longitude' => -0.70107,
           'name'      => 'Elche'] 
       ], 
       [ 
           'id' => 100, 
           'name' => 'pearl', 
           'location' => ['latitude'  => 41.8919, 'longitude' => 12.5113, 
           'name'      => 'Rome'] 
       ], 
       [ 
           'id' => 100, 
           'name' => 'ruby', 
           'location' => ['latitude'  => 53.4106, 'longitude' => -2.9779, 
           'name'      => 'Liverpool'] 
       ], 
       [ 
           'id' => 100, 
           'name' => 'sapphire', 
           'location' => ['latitude'  => 50.08804, 'longitude' => 14.42076, 
           'name'      => 'Prague'] 
       ], 
    ]; 

```

一旦我们准备好秘密列表，我们可以添加我们的`getClosestSecrets`函数如下：

```php
    public function getClosestSecrets($originPoint) 
    { 
      $closestSecrets    = [];
      $preprocessClosure = function($item) use($originPoint) { 
        return $this->getDistance($item['location'], $originPoint); 
      };  

       $distances = array_map($preprocessClosure, self::$cacheSecrets); 

       asort($distances); 

       $distances = array_slice($distances, 0, 
         self::MAX_CLOSEST_SECRETS, true); 

       foreach ($distances as $key => $distance) { 
         $closestSecrets[] = self::$cacheSecrets[$key]; 
       } 

       return $closestSecrets; 
    } 

```

在这段代码中，我们使用我们的缓存秘密列表来计算原点与我们每个秘密点之间的距离。一旦我们有了距离，我们就对结果进行排序并返回最接近的三个。

在我们的位置容器中运行 PHPUnit 将显示所有测试都已通过，这让我们有信心将代码部署到生产环境。

未来的提交可能会对距离计算或最接近功能进行更改，并且可能会破坏我们的测试。幸运的是，有一个单元测试覆盖它们，PHPUnit 会发出警报，因此您可以开始重新思考代码实现。

让您的想象力飞翔，并测试一切--从简单和小的情况到您能想象到的任何奇怪和模糊的情况。想法是您的应用程序将会崩溃，并且会非常严重，在半夜或者在您度假期间。除了尽可能添加尽可能多的测试以确保您在生产中的发布足够稳定以减少破坏风险之外，您无能为力。

## Behat

Behat 是一个开源的行为驱动开发框架。所有 Behat 测试都是用简单的英语编写，并包装成可读的场景。该框架使用 Gherkin 语法，受到了 Ruby 工具 Cucumber 的启发。Behat 的主要优势在于，大多数测试场景都可以被任何人理解。

### 安装

使用 Composer 可以轻松安装 Behat。您只需要编辑每个微服务的`composer.json`，并添加一行新的`"behat/behat" : "3.*"`。您的`require-dev`定义将如下所示：

```php
    "require-dev": { 
      "fzaninotto/faker": "~1.4", 
      "phpunit/phpunit": "~4.0", 
      "behat/behat": "3.*" 
    }, 

```

一旦您更新了`dev`要求，您需要进入每个 PHP-FPM 容器并运行 Composer：

```php
**# cd /var/www/html && composer update**

```

### 测试执行

运行 Behat 就像运行 PHPUnit 一样简单。您只需要进入 PHP-FPM 容器，转到`/var/www/html`文件夹，并运行以下命令：

```php
**# vendor/bin/behat**

```

### 从头开始的 Behat 示例

我们微服务应用程序的关键功能之一是查找秘密。用户应该能够保存这些秘密，为此，他们需要一个钱包。因此，让我们在用户微服务中编写我们的用户故事：

```php
Feature: Secrets wallet 
 In order to play the game 
 As a user 
 I need to be able to put found secrets into a wallet 

 Scenario: Finding a single secret 
    Given there is an "amber" 
    When I add the "amber" to the wallet 
    Then I should have 1 secret in the wallet 

 Scenario: Finding two secrets 
    Given there is an "amber" 
    And there is a "diamond" 
    When I add the "amber" to the wallet 
    And I add the "diamond" to the wallet 
    Then I should have 2 secrets in the wallet 

```

正如你所看到的，该测试可以被项目中的任何人理解，从开发人员到利益相关者。每个测试场景总是具有相同的格式：

```php
Scenario: Some description of the scenario 
 Given some context 
 When some event 
 Then the outcome 

```

您可以向上述基本模板添加一些修饰词（and 或 but）以增强场景描述。在这一点上，您的场景准备就绪后，可以将其保存为`features/wallet.feature`文件。

第一次在项目中开始编写 Behat 测试时，您需要使用以下命令初始化套件：

```php
**# vendor/bin/behat --init**

```

上述命令将创建 Behat 运行场景测试所需的文件。我们将使用的主要文件是`features/bootstrap/FeatureContext.php`；这个文件将成为我们的测试环境。

一旦我们的`FeatureContext`文件就位，就该开始创建我们的场景步骤了。例如，将以下方法放入您的`FeatureContext`中：

```php
    /** 
    * @Given there is a(n) :arg1 
    */ 
    public function thereIsA($arg1) 
    { 
       throw new PendingException(); 
    } 

```

### 提示

Behat 使用文档块来定义步骤、步骤转换和钩子。

在上述代码片段中，我们告诉 Behat，`thereIsA()`函数匹配每个`Given there is a(n)`步骤。在我们的示例中，该定义将匹配以下情况中的步骤：

+   假设有一块琥珀

+   有一颗钻石

我们需要映射每个场景步骤，以便我们的`FeatureContext`最终如下：

```php
    <?php 

     use Behat\Behat\Context\Context; 
     use Behat\Behat\Tester\Exception\PendingException; 
     use Behat\Gherkin\Node\PyStringNode; 
     use Behat\Gherkin\Node\TableNode; 

    /** 
    * Defines application features from the specific context. 
    */ 
    class FeatureContext implements Context 
    { 
      private $secretsCache; 
      private $wallet; 

      public function __construct() 
      { 
        $this->secretsCache = new SecretsCache(); 
        $this->wallet = new Wallet($this->secretsCache); 
      } 

      /** 
      * @Given there is a(n) :secret 
      */ 
      public function thereIsA($secret) 
      { 
        $this->secretsCache->setSecret($secret); 
      } 

      /** 
      * @When I add the :secret to the wallet 
      */ 
      public function iAddTheToTheWallet($secret) 
      { 
        $this->wallet->addSecret($secret); 
      } 

      /** 
      * @Then I should have :count secret(s) in the wallet 
      */ 
      public function iShouldHaveSecretInTheWallet($count) 
      { 
         PHPUnit_Framework_Assert::assertCount( 
           intval($count), 
           $this->wallet 
         ); 
      } 
    } 

```

我们的测试使用需要定义的外部类。这些类实现了我们的逻辑，并且例如故意创建了`features/bootstrap/SecretsCache.php`，其中包含以下内容：

```php
    <?php 
    final class SecretsCache 
    { 
      private $secretsMap = []; 

      public function setSecret($secret) 
      { 
         $this->secretsMap[$secret] = $secret; 
      } 

      public function getSecret($secret) 
      { 
        return $this->secretsMap[$secret]; 
      } 
    } 

```

您还需要创建`features/bootstrap/Wallet.php`，其中包含以下示例代码：

```php
    <?php 
    final class Wallet implements \Countable 
    { 
      private $secretsCache; 
      private $secrets; 

      public function __construct(SecretsCache $secretsCache) 
      { 
        $this->secretsCache = $secretsCache; 
      } 

      public function addSecret($secret) 
      { 
        $this->secrets[] = $secret; 
      } 

      public function count() 
      { 
        return count($this->secrets); 
      } 
    } 

```

前两个类是我们测试的实现，正如你所看到的，它们具有在钱包中存储秘密的逻辑。现在，如果你在控制台上运行 `vendor/bin/behat`，这个工具将检查所有我们的测试场景，并让我们确信我们的代码将按我们想要的方式运行。

这是使用 Behat 测试应用程序的一个简单示例。在我们的 GitHub 存储库中，你可以找到更具体的示例。另外，随时探索 Behat 生态系统；你可以找到多个工具和扩展，可以帮助你测试你的应用程序。

## Selenium

Selenium 是一套用于自动化多平台上的 Web 浏览器的工具，并且可以作为浏览器扩展使用，或者可以安装在服务器上来运行我们的浏览器测试。Selenium 的主要优势在于你可以轻松地记录完整的用户旅程并从记录中创建测试。这个测试可以稍后添加到你的流水线中，以便在每次提交时执行，以发现回归。

### Selenium WebDriver

WebDriver 是你可以用来从其他工具运行浏览器测试的 API。它是一个强大的测试环境，通常放置在专用服务器上，等待运行浏览器测试。

### Selenium IDE

Selenium IDE 是一个 Firefox 扩展，允许你记录、编辑和调试浏览器测试。这个插件不仅是一个录制工具，还是一个带有自动完成功能的完整 IDE。你甚至可以使用 IDE 记录和创建测试，然后用 WebDriver 稍后运行它们。

大多数情况下，Selenium 被用作补充测试工具，从另一个测试框架执行。例如，你可以通过 Mink 项目（[`mink.behat.org/en/latest/`](http://mink.behat.org/en/latest/)）来改进你的 Behat 测试。这个项目是不同浏览器驱动程序的包装器，所以你可以在 BDD 工作流中使用它们。

我们将在第七章 *Security*中讨论我们应用程序的部署。我们将学习如何自动化所有这些测试，并将它们集成到我们的 CI/CD 工作流中。

# 总结

在本章中，你学习了在应用程序中使用测试的重要性，诸如 Behat 和 Selenium 之类的工具，以及关于实现驱动开发。在下一章中，你将学习错误处理、依赖管理和微服务框架。
