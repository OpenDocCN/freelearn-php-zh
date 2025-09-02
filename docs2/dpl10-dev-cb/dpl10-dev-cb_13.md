# 13

# 在 Drupal 中编写自动化测试

在前面的章节中，我们回顾了如何使用控制器、路由、响应、自定义模块、自定义实体类型、钩子等来为 Drupal 添加自定义功能。即使只有一点点的代码，你也可以为你的 Drupal 应用程序添加很多功能。

但你怎么知道它真的“工作”了？当然，你可以点击并尝试一些事情——但这并不能保证事情在底层按预期工作。你添加的代码和功能越多，验证现有功能是否仍然完整而不提供测试就越困难。

通过实施自动化测试，你可以确保你添加的代码和功能确实按预期工作。最重要的是，自动化测试可以显著减少错误和回归到你的生产网站，这反过来将有助于增强作为开发者的信心。

Drupal 提供了几种类别的工具来帮助你为你的应用程序提供测试。本章涵盖了以下内容：

+   安装 PHPUnit 测试套件

+   运行 PHPUnit

+   编写单元测试

+   编写内核测试

+   编写功能测试

+   编写功能 JavaScript 测试

+   编写 NightwatchJS 测试

# 测试类型

你可以在 Drupal 中运行和编写的测试类型有五种——**单元测试**、**内核测试**、**功能测试**、**功能 JavaScript 测试**和**NightwatchJS 测试**。你编写哪些将取决于你正在创建的功能类型以及你愿意接受的测试覆盖率水平来证明你的代码正在工作。让我们来看看这些测试类型。你可以在 GitHub 上找到本章中使用的完整代码：[`github.com/PacktPublishing/Drupal-10-Development-Cookbook/tree/main/chp13`](https://github.com/PacktPublishing/Drupal-10-Development-Cookbook/tree/main/chp13)

## 单元测试

单元测试是不需要 Drupal 安装即可评估的测试，因为它们只测试执行代码。这是你可以编写的最低级别的测试。单元测试对于测试插件、服务或其他不要求与数据库交互的代码非常有用。如果你需要编写需要数据库或 Drupal 环境的测试，你将编写内核测试。

## 内核测试

内核测试是单元测试的下一步。内核测试是一种可以执行和测试需要一定数据库交互级别的功能的测试。如果你想测试在保存实体时触发的功能、测试字段格式化器、检查用户对路由的访问权限，或者测试控制器及其响应的功能，内核测试就是你需要编写的测试。在执行内核测试时，会在测试执行的地方安装一个 Drupal 实例。这种隔离级别允许你在不影响当前工作数据库的情况下测试交互式功能。你可以指定要安装的模块和在内核测试中包含的配置，这使得它成为测试模块的绝佳方式。这些是在 Drupal 中最常见的测试。

## 功能性测试

功能性测试是你能编写的最高级别的测试。像内核测试一样，功能性测试将安装一个可工作的 Drupal 副本来运行测试。这些测试使用无头浏览器进行评估，并且从用户的角度测试功能非常有用。这类测试适用于测试用户工作流程、用户权限，以及评估页面内容是否符合预期。例如，如果你有一个功能，用户导航到网站，登录，导航到 Drupal 管理界面，并看到其他角色不应看到的部分，你可以通过功能性测试来评估这一点。

## 功能性 JavaScript 测试

功能性 JavaScript 测试是在真实浏览器中执行的，例如 Chrome 或 Firefox。常规的功能性测试在底层使用名为 Mink 的无头 PHP 浏览器模拟器中运行。由于它不是一个完整的浏览器，因此无法测试像 Chrome 这样的真实浏览器所拥有的任何 JavaScript 相关功能。如果你想在浏览器中测试与 AJAX、cookies 或 DOM 相关的 JavaScript 事件，你需要编写一个功能性 JavaScript 测试。功能性 JavaScript 测试将需要存在一个浏览器，如 Chrome，以及像 Selenium 这样的工具来编排测试中的 Chrome。

## NightwatchJS 测试

与功能性 JavaScript 测试类似，可以使用 `yarn` 运行和与之交互。如果你是一位倾向于编写比 PHP 更多 JavaScript 的开发者，你可能对使用 NightwatchJS 而不是 **PHPUnit** 感兴趣。另一个额外的优点是，你可以为自定义编写的 JavaScript 编写单元测试，这是你不能用 PHPUnit 来测试的。

功能性 JavaScript 和 NightwatchJS 测试需要最多的努力来设置和实施，但它们非常有价值，因为它们在相同的设置、条件、用户角色（们）和浏览器下运行。它们执行时间最长，但仍然只占任何人类完成相同任务时间的很小一部分。

# 安装 PHPUnit 测试套件

所有的先前测试类型（除了 NightwatchJS 测试）都由一个测试框架 PHPUnit 执行。我们可以向我们的自定义模块或自定义主题添加任意数量的测试，并使用 PHPUnit 运行它们——我们只需安装它并配置它指向我们的测试文件。

## 准备工作

在继续之前，首先要安装所有测试依赖项，以便您实际上可以在 Drupal 中运行测试。Drupal 有一个特定的 Composer 包，它带来了 PHPUnit 和其所需的依赖项。

安装时，请按照以下步骤操作：

在项目根目录打开一个终端（命令行）。

运行以下 Composer 命令：

```php
composer require --dev drupal/core-dev:¹⁰
```

`drupal/core-dev` 包将把 PHPUnit 和 Drupal 中编写和运行测试所需的各个依赖项引入到您的项目中。

--dev 标志

注意，在安装时，我们使用 `--dev` 标志。这告诉 Composer 在 `composer.json` 的 `require-dev` 部分列出这些包。`drupal/core-dev` 包不是您想在生产环境中拥有的东西；它仅用于测试。

## 如何操作...

接下来，我们需要配置 PHPUnit，使其知道我们的测试所在的位置。Drupal 核心在 `core` 目录中提供了一个示例 `phpunit.xml.dist` 文件。这是 PHPUnit 执行时读取的文件。您不需要逐行理解它，但有一些区域我们需要调整，以便它可以在您的自定义模块目录中执行。

将 `phpunit.xml.dist` 文件从 `core` 目录复制出来，并将其放置在项目根目录中。将文件重命名为 `phpunit.xml` 并进行以下编辑：

1.  在文档顶部 `<phpunit>` 标签上编辑 bootstrap 属性，使其指向核心测试的 `bootstrap` 文件：

    ```php
    bootstrap="./web/core/tests/bootstrap.php"
    ```

1.  为了运行内核或功能测试，请编辑 `<php>` 部分中的以下环境设置：

    ```php
    <env name="SIMPLETEST_BASE_URL"
    ```

    ```php
        value="http://localhost"/>
    ```

    ```php
    <env name="SIMPLETEST_DB" value="
    ```

    ```php
        mysql://database:database@database/database"/>
    ```

    ```php
    <env name="BROWSERTEST_OUTPUT_DIRECTORY" value=""/>
    ```

    ```php
    <env name="SYMFONY_DEPRECATIONS_HELPER" value="weak"/>
    ```

    ```php
    <env name="MINK_DRIVER_ARGS_WEBDRIVER" value="
    ```

    ```php
         ["chrome", {"browserName":"chrome",
    ```

    ```php
         "chromeOptions":{"args":["--disable-gpu","--
    ```

    ```php
         headless"]}}, "http://chrome:9515"] "/>
    ```

DDEV、Lando、Docksal 或 Docker？

如果您使用 DDEV、Lando、Docksal 或其他本地运行 Drupal 的现成工具，请检查它们的文档以及服务名称。根据您使用的工具，`SIMPLETEST_BASE_URL`、`SIMPLETEST_DB` 和 `MINK_DRIVER_ARGS_WEBDRIVER` 的先前值可能会有所不同。

1.  接下来，编辑 `testsuites` 部分，使其指向您的自定义模块目录。您需要为每种测试类型指定一个 `testsuite` 条目：

    ```php
      <testsuites>
    ```

    ```php
        <testsuite name="unit">
    ```

    ```php
          <directory>
    ```

    ```php
            web/modules/custom/*/tests/src/Unit
    ```

    ```php
          </directory>
    ```

    ```php
        </testsuite>
    ```

    ```php
        <testsuite name="kernel">
    ```

    ```php
          <directory>
    ```

    ```php
            web/modules/custom/*/tests/src/Kernel
    ```

    ```php
          </directory>
    ```

    ```php
        </testsuite>
    ```

    ```php
        <testsuite name="functional">
    ```

    ```php
          <directory>
    ```

    ```php
            web/modules/custom/*/tests/src/Functional
    ```

    ```php
          </directory>
    ```

    ```php
        </testsuite>
    ```

    ```php
        <testsuite name="functional-javascript">
    ```

    ```php
          <directory>
    ```

    ```php
       web/modules/custom/*/tests/src/FunctionalJavascript
    ```

    ```php
          </directory>
    ```

    ```php
        </testsuite>
    ```

    ```php
    </testsuites>
    ```

这些更改将告诉 PHPUnit 我们为自定义模块（位于 `web/modules/custom`）编写的测试的位置。测试位于模块的 `tests/src` 目录中。

每种测试类型都位于该目录下的独立目录中：

单元测试放入 `tests/src/Unit`

内核测试放入 `tests/src/Kernel`

功能测试放入 `tests/src/Functional`

函数 JavaScript 测试放入 `tests/src/FunctionalJavascript`

通过在`phpunit.xml`中将每种测试类型定义为单独的测试套件，我们可以选择使用 PHPUnit 执行哪种类型的测试，我们将在下一节中探讨这一点。你还可以设置多个`testsuite`条目，或者一个`testsuite`条目内的多个目录。例如，如果你想运行 PHPUnit 时包含所有贡献模块的所有单元测试，你可以添加另一个`directory`条目：

```php
<testsuite name="unit">
   <directory>
      web/modules/contrib/*/tests/src/Unit
   </directory>
   <directory>
      web/modules/custom/*/tests/src/Unit
   </directory>
</testsuite>
```

我们创建了一个`phpunit.xml`文件并将其放置在项目的根目录下，原因在于该文件可以防止在下次更新 Drupal 时覆盖或删除项目。这样，你可以将配置提交到你的仓库中，并与团队共享，同时还能在 GitHub、GitLab 或 CircleCI 提供的持续集成服务中运行测试。

重要的是要知道，这仅仅是一种设置 PHPUnit 配置的可能方式。一旦你对 PHPUnit 更加熟悉，你还可以在这里配置更多设置。

还有一些其他设置可以输出测试覆盖率结果，保存测试失败截图的位置，额外的测试监听器，日志输出等等。请务必查看在线文档（[`phpunit.readthedocs.io/en/9.5/configuration.html`](https://phpunit.readthedocs.io/en/9.5/configuration.html)），以获取更多关于如何配置 PHPUnit 的信息。

## 它是如何工作的...

当你执行 PHPUnit 时，它会使用`phpunit.xml`文件来告知如何执行。它将扫描所有列出的目录以查找测试文件，并相应地执行它们，在命令行窗口（或 IDE）中提供测试反馈。为每个你编写测试的项目，都需要一个像这样的`phpunit.xml`文件才能运行任何测试。

## 还有更多...

如果你计划编写功能 JavaScript 测试，你还需要 Chrome、Chrome WebDriver 和 Selenium。安装和配置这些工具可能因你使用 Lando、DDEV、Docksal、Docker Compose 或其他工具来运行 Drupal 而异。请查阅有关如何安装这些工具以正确执行功能 JavaScript 测试的最佳信息的文档。假设你正在使用`.lando.yml`文件中的`services`部分：

```php
  chrome:
     type: compose
     services:
       image: drupalci/webdriver-chromedriver:production
       command: chromedriver --log-path=/tmp/
           chromedriver.log --verbose
```

在`phpunit.xml`中的`MINK_DRIVER_ARGS_WEBDRIVER`设置中，`chrome`成为了一种新的有线服务，这使得所有这些功能都能为功能 JavaScript 测试工作）。如果你想安装并使用 Firefox 作为测试的替代浏览器，你也可以这样做，但 Chrome 是测试中最常用的浏览器。

这些工具的添加方式可能因你所使用的工具而异。请查阅有关如何添加 Chrome 和 Chrome WebDriver 的最佳方法的文档。由于涉及太多特定于堆栈的配置，本书无法涵盖。

# 运行 PHPUnit

首先，我们可以通过执行`phpunit`命令来验证我们是否正确设置了。目前还没有测试，但这没关系。这一步只是为了验证工具已安装，并且我们的配置文件被正确读取。

## 如何做到这一点...

咨询文档

从这里开始，我们将提供使用 PHPUnit 执行测试的裸骨、详尽的命令。如果你使用 Lando、DDEV、Docksal 或其他工具，请查阅它们的文档，了解如何最佳运行 PHPUnit。它们通常有小型且方便的命令包装器。

在你的命令行中，执行以下操作：

```php
phpunit
```

这是您将运行的主要命令。PHPUnit 将自动检测项目目录中的`phpunit.xml`配置。

找不到 phpunit？

如果你收到关于找不到`phpunit`命令的错误，你可能需要使用项目根目录中的完整路径来代替，使用`vendor/bin/phpunit`。Composer 应该会自动为你别名命令，但根据项目的设置，这可能是必需的。

你应该看到类似以下输出的内容：

```php
PHPUnit 9.5.26 by Sebastian Bergmann and contributors.
No tests executed!
```

你也可以通过向`phpunit`添加`--testsuite`和`--filter`参数来运行特定的`testsuites`或单个测试：

```php
phpunit --testsuite unit
```

上述命令将在`phpunit.xml`文件中列出的目录中运行所有找到的单元测试：

```php
phpunit --testsuite unit --filter FooBarTest
```

上述命令将仅运行`FooBarTest`单元测试。这两种方法在测试特定`testsuites`或测试本身以获得更快反馈时非常有用，尤其是在测试、调试和迭代代码时。

## 它是如何工作的…

当你执行 PHPUnit 时，它会扫描`phpunit.xml`文件中列出的所有目录，并在命令行窗口中提供测试反馈。

在这种情况下，PHPUnit 表示没有找到或执行任何测试，这是正确的，因为我们还没有任何测试。我们现在可以编写第一个单元测试了。

# 编写单元测试

让我们编写第一个测试。如前所述，单元测试是你能编写的最低级别的测试类型。它只执行和测试没有依赖服务（如连接的数据库或其他集成 API）的原始 PHP 代码。这意味着测试可以使用 PHP 单独执行。

## 准备工作

让我们创建一个场景，帮助你思考何时以及如何应用单元测试。想象一下，你需要向一个前端组件提供数据。前端开发者要求你在 API 响应中提供所有 JSON 键，格式为`camelCase`。`camelCase`会将字符串如`field_event_date`转换为`fieldEventDate`。

在 Drupal 的许多地方使用`snake_case`；你最常见的用途是在机器名（如前一个事件日期字段）上。Drupal 中的所有机器名都在`snake_case`格式。

这是一个非常简单的例子，但完美地说明了我们如何使用单元测试来测试我们的类。

## 如何做呢...

现在创建一个名为`chapter13`的自定义模块。这是我们将在本章的其余部分编写示例代码及其测试的地方。如果你需要复习如何在 Drupal 中创建自定义模块，请参考*第四章*，*使用自定义代码扩展 Drupal*。

自定义模块创建完成后，让我们在 `chapter13` 模块的 `src` 目录下添加 `CamelCase` 类：

```php
<?php
declare(strict_types=1);
namespace Drupal\chapter13;
/**
 * Class CamelCase
 * @package Drupal\chapter13
 */
class CamelCase {
  /**
   * Convert snake_case to camelCase.
   *
   * @param string $input
   * @return string
   */
  public static function convert(string $input): string {
    $input = strtolower($input);
    return str_replace('_', '', lcfirst(ucwords
        ($input, '_')));
  }
}
```

看起来它工作得很好，对吧？让我们通过一个测试来证明它。

在你的自定义模块中，创建一个 `tests` 目录。在这个目录下，创建一个 `src` 目录，然后在 `src` 目录中，创建一个 `Unit` 目录。现在你应该有一个路径看起来像 `chapter13/tests/src/Unit`。请注意，你的 `tests` 目录中的 `src` 目录与 `chapter13` 模块根目录下的 `src` 目录不同，`CamelCase` 类就位于该目录。

在 `chapter13/tests/src/Unit` 目录下，创建一个新文件并将其命名为 `CamelCaseTest.php`。所有测试文件都必须以 `Test` 结尾，以便被 PHPUnit 发现。

我们的测试文件需要做两件事：

+   使用 `CamelCase` 类提供测试数据

+   断言转换方法返回我们期望的结果

在你的测试类中，任何以 `test` 开头的方法都将由 PHPUnit 进行评估。考虑到这一点，让我们继续在我们的 `CamelCaseTest.php` 文件中填写我们的单元测试：

```php
<?php
namespace Drupal\Tests\chapter13\Unit;
use Drupal\Tests\UnitTestCase;
use Drupal\chapter13\CamelCase;
/**
 * Class CamelCaseTest
 * @package Drupal\Tests\chapter13\Unit
 */
class CamelCaseTest extends UnitTestCase {
  /**
   * Data provider for testToCamel().
   *
   * @return array
   *   An array containing input values and expected output
            values.
   */
  public function exampleStrings() {
    return [
      ['button_color', 'buttonColor'],
      ['snake_case_example', 'snakeCaseExample'],
      ['ALL_CAPS_LOCK', 'allCapsLock'],
    ];
  }
  /**
   * Tests the ::convert method.
   *
   * @param $input
   *   The input values.
   *
   * @param bool $expected
   *   The expected output.
   *
   * @param bool $separator
   *   The string separator.
   *
   * @dataProvider exampleStrings()
   */
  public function testCamelCaseConversion($input,
    $expected) {
    $output = CamelCase::convert($input);
    $this->assertEquals($expected, $output);
  }
}
```

好的，运行 `phpunit`：

```php
phpunit --testsuite unit --filter CamelCaseTest
```

这将导致以下结果：

```php
Testing
...  3 / 3 (100%)
Time: 00:00.044, Memory: 10.00 MB
OK (3 tests, 3 assertions)
```

这个输出表明测试被调用了三次（一次对应于 `@dataProvider` 中的每一组数据）并且每次都通过了。换句话说，`$input` 等于 `$expected`，这是 `assertEquals` 评估的内容。如果测试失败，这个输出将表明哪个测试失败以及在哪一行。

## 它是如何工作的…

PHPUnit 报告说所有三个测试都通过了。但是，等等——我们不是只写了 `testCamelCaseConversion` 测试方法吗？

当测试执行时，PHPUnit 检测到我们测试方法中的一个特殊注释，称为 `@dataProvider`，它标记了 `exampleStrings` 方法。`exampleStrings` 方法提供了一个数据数组。每个数组包含要发送的值和我们期望得到的值。PHPUnit 将循环数据提供者的值，因此我们的测试方法被调用三次（一次对应于每一组值）并评估测试。这意味着当执行时，PHPUnit 看到的测试方法调用如下：

```php
testCamelCaseConversion("button_color", "buttonColor")
```

三个值测试确保该方法对两个或更多单词有效，并返回正确的格式。

在测试中，我们将输入传递给 `CamelCase::convert` 方法。从该方法中，我们将返回的值传递给 `assertEquals`，这是 PHPUnit 的许多值断言方法之一。

你添加到数据提供者中的每一个项目都将由使用它们的测试方法进行断言。`dataProviders` 是测试我们代码在多种场景下的一个极好的方式。因此，我们知道传递给转换的任何字符串都将从 `foo_bar` 转换为 `fooBar` 并返回。这个测试证明了这一点。

那么，自动化测试是如何帮助这里的呢？在我们的设置中，团队中的任何开发者都可以运行 `testsuites` 并查看它们的输出。如果它们失败了，他们可以看到哪些代码失败以及在哪里。这为在代码部署到生产之前修复代码提供了机会。

在持续集成设置下，你可以防止代码在所有测试通过之前合并到你的主分支。正如我们提到的，这是减少在生产环境中向用户暴露错误和缺陷的绝佳方式。

## 还有更多…

随着时间的推移，需求可能会改变，而且经常是这样。假设你部署了这段代码后，你的前端开发者回来要求将任何类似`foo-bar`的字符串也转换为驼峰格式。

我们的测试已经到位，我们可以将这个新案例添加到我们的数据提供者中：

```php
  public function exampleStrings() {
    return [
      ['button_color', 'buttonColor'],
      ['snake_case_example', 'snakeCaseExample'],
      ['ALL_CAPS_LOCK', 'allCapsLock'],
      ['foo-bar', 'fooBar'],
    ];
  }
```

现在我们运行测试，得到以下结果：

```php
Testing
...F     4 / 4 (100%)
Time: 00:00.046, Memory: 10.00 MB
There was 1 failure:
1) Drupal\Tests\chapter13\Unit\CamelCaseTest::
    testCamelCaseConversion with data set #3 ('foo-bar',
        'fooBar')
Failed asserting that two strings are equal.
--- Expected
+++ Actual
@@ @@
-'fooBar'
+'foo-bar'
FAILURES!
Tests: 4, Assertions: 4, Failures: 1.
```

这没问题——我们还没有更新我们的实现。首先添加新案例到测试中，让我们可以迭代和改进实现代码，直到测试通过。这提供了一个快速的反馈循环，因为我们可以在`CamelCase`类中随意修改实现。如果所有测试都通过，我们知道我们已经满足了要求，我们的功能是有效的，我们可以继续处理模块中的其他功能。

## 参见

在编写任何实现代码之前先编写测试被称为**测试驱动开发**。这意味着测试驱动规范或需求，实现代码使它们通过。关于你是否应该先编写所有测试，有许多不同的观点。

如果你刚开始接触测试，如果你需要在 Drupal 模块中澄清一个想法，写一些初始代码作为概念验证是完全可以的。你可以在开发过程中的任何时候添加测试。重要的是要为你的代码添加测试；不必太担心何时添加。

你练习得越多，写测试也越多，你的技能就会越好，最终你会转向先写测试。

继续实验`CamelCase`类及其`CamelCaseTest`类。看看你是否可以发明一些新的需求，更改代码，运行测试，并使它们通过。

Drupal 核心有数百个优秀的单元测试示例。最受欢迎的贡献模块也很有用。如果你卡住了或需要指导，一定要查看它们。

现在你已经了解了 Drupal 中单元测试的基础，让我们看看内核测试。

# 编写内核测试

让我们基于之前的例子继续前进。现在假设利益相关者要求你在屏幕上以驼峰格式输出字段值。好消息是我们有一个工作的实现和单元测试，所以我们可以轻松地在 Drupal 中完成这个任务。

在这种情况下，我们需要为字符串字段创建一个字段格式化类。当格式化器用于字段以显示输出时，我们希望将用户输入通过我们现有的`CamelCase`类。如果你需要关于字段格式化和管理实体显示的复习，请参阅*第二章*，*内容* *构建经验*。

这提供了一个很好的例子，可以进入内核测试。之前，我们提到内核测试创建了一个具有你测试中指定的设置的 Drupal 最小安装，以便运行和评估它们的测试方法。运行内核测试没有危险，因为它们以任何方式都不会接触或干扰你的当前站点数据库。测试完成后，它将被*拆除*，或从数据库中删除，不留痕迹。

## 如何做到这一点...

在你的`tests/src`目录下创建一个新的目录，命名为`Kernel`。这是你的`chapter13`模块的内核测试将驻留的地方。这次，我们将先编写测试。

在`chapter13/tests/src/Kernel`下创建`CamelCaseFormatterTest.php`文件。接下来，我们将用以下代码填充它。设置部分有相当多的模板代码，但我们将回顾正在发生的事情：

```php
<?php
namespace Drupal\Tests\chapter13\Kernel;
use Drupal\Core\Entity\Entity\EntityViewDisplay;
use Drupal\field\Entity\FieldConfig;
use Drupal\field\Entity\FieldStorageConfig;
use Drupal\KernelTests\KernelTestBase;
use Drupal\Tests\node\Traits\ContentTypeCreationTrait;
use Drupal\Tests\node\Traits\NodeCreationTrait;
/**
 * Tests the formatting of string fields using the Camel
        Case field formatter.
 *
 * @package Drupal\Tests\chapter13\Kernel
 */
class CamelCaseFormatterTest extends KernelTestBase {
  use NodeCreationTrait,
    ContentTypeCreationTrait;
  protected $strictConfigSchema = FALSE;
  /**
   * Modules to enable.
   *
   * @var array
   */
  protected static $modules = [
    'field',
    'text',
    'node',
    'system',
    'filter',
    'user',
    'chapter13',
  ];
  /**
   * {@inheritdoc}
   */
  protected function setUp(): void {
    parent::setUp();
    // Install required module schema and configs.
    $this->installEntitySchema('node');
    $this->installEntitySchema('user');
    $this->installConfig(['field', 'node', 'filter',
        'system']);
    $this->installSchema('node', ['node_access']);
    // Create a vanilla content type for testing.
    $this->createContentType(
      [
        'type' => 'page'
      ]
    );
    // Create and store the field_chapter13_test field.
    FieldStorageConfig::create([
      'field_name' => 'field_chapter13_test',
      'entity_type' => 'node',
      'type' => 'string',
      'cardinality' => 1,
      'locked' => FALSE,
      'indexes' => [],
      'settings' => [
        'max_length' => 255,
        'case_sensitive' => FALSE,
        'is_ascii' => FALSE,
      ],
    ])->save();
    FieldConfig::create([
      'field_name' => 'field_chapter13_test',
      'field_type' => 'string',
      'entity_type' => 'node',
      'label' => 'Chapter13 Camel Case Field',
      'bundle' => 'page',
      'description' => '',
      'required' => FALSE,
      'settings' => [
        'link_to_entity' => FALSE
      ],
    ])->save();
    // Set the entity display for testing to use our
        camel_case formatter.
    $entity_display = EntityViewDisplay::load
         ('node.page.default');
    $entity_display->setComponent('field_chapter13_test',
    [
      'type' => 'camel_case',
      'region' => 'content',
      'settings' => [],
      'label' => 'hidden',
      'third_party_settings' => []
    ]);
    $entity_display->save();
  }
  /**
   * Tests that the field formatter camel_case formats the
        value
   * as expected.
   */
  public function testFieldIsFormatted() {
    $node = $this->createNode(
      [
        'type' => 'page',
        'field_chapter13_test' => 'A user entered string'
      ]
    );
    $build = $node->field_chapter13_test->view('default');
    $this->assertSame('aUserEnteredString',
        $build[0]['#context']['value']);
  }
}
```

不要因为测试中的代码量而气馁。大多数代码是设置我们运行测试本身所需的条件：

+   安装提供字段和`Node`实体的所需模块

+   安装模块配置和实体模式

+   创建一个`page`内容类型

+   创建`field_chapter13_test`并将其分配给页面节点类型

+   修改节点默认视图模式并将`field_chapter13_test`设置为使用`camel_case`字段格式化器

另一种方法是，在 Drupal 中创建所有这些项目（内容类型、字段、设置格式化器）并将该配置导出到仅用于此测试的模块中。最终结果大致相同，但你可能会发现，完全用代码创建条件在长期来看比几十个 YAML 文件更易于维护。

测试中的命名

在内核测试中命名内容类型、实体或字段时，你可以随意命名。它们不必与你的系统完全匹配，特别是如果你只需要一个可以包含字段的随机内容类型。我们测试的是返回的输出，而不是名称。

如果我们现在运行测试，它将失败；现在，我们可以实现使测试通过。

为了解决这个问题，我们需要创建一个使用我们现有的`CamelCase`类的`FieldFormatter`插件。这个练习很简单，因为将输入转换为驼峰格式的大部分工作已经存在。

在`chapter13`模块的`src`目录下添加一个名为`Plugin/Field/FieldFormatter`的目录。在该目录中，添加一个名为`CamelCaseFormatter.php`的文件。我们的实现将如下所示：

```php
<?php
declare(strict_types = 1);
namespace Drupal\chapter13\Plugin\Field\FieldFormatter;
use Drupal\chapter13\CamelCase;
use Drupal\Core\Field\FieldItemInterface;
use Drupal\Core\Field\FieldItemListInterface;
use Drupal\Core\Field\Plugin\Field\FieldFormatter\
    StringFormatter;
/**
 * Plugin implementation of the 'camel_case' field
      formatter.
 *
 * @FieldFormatter(
 *   id = "camel_case",
 *   label = @Translation("Camel case"),
 *   field_types = {
 *     "string"
 *   }
 * )
 */
class CamelCaseFormatter extends StringFormatter {
  /**
   * {@inheritdoc}
   */
  public function viewElements(FieldItemListInterface
    $items, $langcode) : array {
    $elements = [];
    foreach ($items as $delta => $item) {
      $view_value = $this->viewValue($item);
      $elements[$delta] = $view_value;
    }
    return $elements;
  }
  /**
   * {@inheritdoc}
   */
  protected function viewValue(FieldItemInterface $item) {
    return [
      '#type' => 'inline_template',
      '#template' => '{{ value|nl2br }}',
      '#context' => ['value' => CamelCase::convert($item
          ->value)],
    ];
  }
}
```

通过从核心 Drupal 扩展`StringFormatter`类，我们可以利用我们的`CamelCase`类进行修改。

现在，让我们运行我们的内核测试：

```php
phpunit --testsuite kernel --filter CamelCaseFormatterTest
```

这导致以下结果：

```php
Testing
F     1 / 1 (100%)
Time: 00:01.500, Memory: 10.00 MB
There was 1 failure:
1) Drupal\Tests\chapter13\Kernel\CamelCaseFormatterTest::
    testFieldIsFormatted
Failed asserting that two strings are identical.
--- Expected
+++ Actual
@@ @@
-'aUserEnteredString'
+'a user entered string'
FAILURES!
Tests: 1, Assertions: 9, Failures: 1.
```

失败了！发生了什么？记得我们之前提到过需求可能会改变吗？我们的测试用例将`用户输入的字符串`作为测试数据。我们的`CamelCase`类目前无法处理包含空格、破折号或逗号的字符串。由于这是 Drupal 中的一个字段，用户可以输入几乎所有内容。我们需要考虑到这一点。

修改`CamelCase`类以适应这个新的要求：

```php
  public static function convert(string $input): string {
    $input = strtolower($input);
    $input = preg_replace('/[, -]/', '_', $input);
    return str_replace('_', '', lcfirst(ucwords
        ($input, '_')));
  }
```

替换逗号、空格和破折号字符的添加现在应该能满足新的用例。让我们再次运行内核测试：

```php
Testing
.       1 / 1 (100%)
Time: 00:01.491, Memory: 10.00 MB
OK (1 test, 9 assertions)
```

然而，我们不要忘记之前损坏的单元测试——让我们将两个新的示例字符串添加到`exampleStrings`数据提供者方法中：

```php
  public function exampleStrings() {
    return [
      ['button_color', 'buttonColor'],
      ['snake_case_example', 'snakeCaseExample'],
      ['ALL_CAPS_LOCK', 'allCapsLock'],
      ['foo-bar', 'fooBar'],
      ['This is a basic string', 'thisIsABasicString'],
    ];
  }
```

现在，当你运行 PHPUnit 时，单元测试和内核测试都应该通过：

```php
Testing
......   6 / 6 (100%)
Time: 00:01.419, Memory: 10.00 MB
OK (6 tests, 14 assertions)
```

我们现在网站新增了一个新的、经过测试的功能，我们可以有信心地部署它。当然，当我们需要时，我们可以自由地在我们 Drupal 应用程序中重用这个功能。

## 它是如何工作的...

当运行内核测试时，Drupal 将安装一个额外的副本来运行测试。内核测试将使用测试中所需的安装配置文件和模块。这样，你可以确保你在最干净的可能设置中测试你的代码，没有任何不必要的贡献模块或其他代码的干扰。当测试运行完成后，Drupal 将自动清理并删除这个第二个安装，包括测试本身创建的任何数据。

到目前为止，我们已经编写了代码和测试来证明它们确实做了我们需要的——但它们无法测试网站上的用户是否看到了正确的内容。让我们深入进去，添加一个功能测试来做到这一点！

# 编写功能测试

现在我们知道我们的代码是有效的，让我们证明用户在访问节点时可以看到格式化的字符串。功能测试使用一个模拟浏览器，允许我们模拟用户在网站上导航。功能测试安装 Drupal 并在隔离模式下进行测试，因此没有风险会破坏你当前的 Drupal 安装。

## 如何做到这一点...

使用功能测试，我们可以像真实用户一样导航浏览器，执行所有各种断言，这是单元测试和内核测试无法做到的。像之前一样，我们需要设置一些配置才能进行测试。在你的`tests/src`目录中，创建一个名为`Functional`的新目录，然后在其中创建一个名为`CamelCaseFormatterDisplayTest.php`的文件。

在这个新的测试文件中，我们将借鉴之前内核测试的一些设置：

```php
<?php
namespace Drupal\Tests\chapter13\Functional;
use Drupal\Core\Entity\Entity\EntityViewDisplay;
use Drupal\field\Entity\FieldConfig;
use Drupal\field\Entity\FieldStorageConfig;
use Drupal\Tests\BrowserTestBase;
/**
 * Class CamelCaseFormatterDisplayTest
 *
 * @package Drupal\Tests\chapter13\Functional
 */
class CamelCaseFormatterDisplayTest extends
    BrowserTestBase {
  /**
   * @var bool Disable schema checking.
   */
  protected $strictConfigSchema = FALSE;
  /**
   * @var string The theme to use during test.
   */
  protected $defaultTheme = 'stark';
  /**
   * Modules to enable.
   *
   * @var array
   */
  protected static $modules = [
    'field',
    'text',
    'node',
    'system',
    'filter',
    'user',
    'chapter13',
  ];
  /**
   * {@inheritdoc}
   */
  protected function setUp(): void {
    parent::setUp();
    // Create a vanilla content type for testing.
    $this->createContentType(
      [
        'type' => 'page'
      ]
    );
    // Create and store the field_chapter13_test field.
    FieldStorageConfig::create([
      'field_name' => 'field_chapter13_test',
      'entity_type' => 'node',
      'type' => 'string',
      'cardinality' => 1,
      'locked' => FALSE,
      'indexes' => [],
      'settings' => [
        'max_length' => 255,
        'case_sensitive' => FALSE,
        'is_ascii' => FALSE,
      ],
    ])->save();
    FieldConfig::create([
      'field_name' => 'field_chapter13_test',
      'field_type' => 'string',
      'entity_type' => 'node',
      'label' => 'Chapter13 Camel Case Field',
      'bundle' => 'page',
      'description' => '',
      'required' => FALSE,
      'settings' => [
        'link_to_entity' => FALSE
      ],
    ])->save();
    // Set the entity display for testing to use our
        camel_case formatter.
    $entity_display = EntityViewDisplay::load
        ('node.page.default');
    $entity_display->setComponent('field_chapter13_test',
      [
        'type' => 'camel_case',
        'region' => 'content',
        'settings' => [],
        'label' => 'hidden',
        'third_party_settings' => []
      ]);
    $entity_display->save();
  }
  /**
   * Test that a site visitor can see a string formatted
         with our custom
   * field CamelCaseFieldFormatter.
   *
   * @return void
   */
  public function testUserCanSeeFormattedString() {
    $this->drupalCreateNode(
      [
        'type' => 'page',
        'field_chapter13_test' => 'A user entered string'
      ]
    );
    $this->drupalGet('/node/1');
    $this->getSession()->getPage()->hasContent
        ('aUserEnteredString');
  }
}
```

在功能测试中这里有一些差异。

在功能测试中，您可以指定两个属性，`$profile` 和 `$defaultTheme`。`$profile` 指定用于此测试的安装配置文件，而 `$defaultTheme` 指定用于此测试要激活的主题。默认情况下，使用 *testing* 配置文件，这是安装 Drupal 所需的最小绝对值。您还可以指定 *minimal* 或 *standard*，这将安装这些配置文件之一以用于测试。

安装配置文件和测试

如果您使用带有配置或额外内容的安装配置文件，例如创建内容类型和角色，请确保不要尝试在测试中再次使用 `createContentType` 等方法创建它们。在这些情况下，我们将收到错误，因为安装配置文件已经创建了它们。

最后，您还可以创建自己的安装配置文件进行测试，其中包含您需要的 Drupal 配置文件 – 如果您不想在测试的 `setUp` 方法中编写代码，这将很有用。

为了使此测试运行，我们只需要默认设置，所以运行 PHPUnit：

```php
phpunit --testsuite functional --filter
  CamelCaseFormatterDisplayTest
```

这导致以下结果：

```php
Testing
.           1 / 1 (100%)
Time: 00:07.390, Memory: 10.00 MB
OK (1 test, 3 assertions)
```

它通过了，但请注意，执行功能测试的总时间远高于单元测试或内核测试。这是由于必须安装 Drupal 的完整版本来运行功能测试。在此机器上，使用核心 *testing* 配置文件需要 7 秒。如果我们切换到安装更多内容的核心 *standard* 配置文件，我们可以看到时间的增加：

```php
protected $profile = 'standard';
Testing
.          1 / 1 (100%)
Time: 00:18.441, Memory: 10.00 MB
OK (1 test, 2 assertions)
```

所用时间比较小的安装配置文件翻倍。这不一定是一个问题，但在编写测试时请记住这一点。最终，有价值的测试和具体的反馈值得花一点额外的时间来确保我们用自定义代码正确地完成事情。

## 它是如何工作的...

功能测试比内核测试更健壮。与内核测试一样，Drupal 将安装另一个副本来运行测试，但安装比内核测试更 *完整* 或更完整。这些测试使用名为 Mink 的浏览器模拟器运行，这就是它能够导航 URL、检查 HTML 元素以及执行类似真实用户导航网站的操作的原因。

## 参见

您可以使用功能测试做很多事情（请参阅本章末尾关于如何扩展我们之前所涵盖的内容的想法）。然而，Mink 并不是一个真正的浏览器，并且没有运行或对 JavaScript 交互做出反应的能力。例如，如果我们需要在 5 秒后隐藏页面上的字段文本，或者使用 AJAX 显示字段值，我们需要使用功能 JavaScript 测试来完成。

# 编写功能 JavaScript 测试

假设您现在有一个最后的请求。您被要求使用 AJAX 在用户访问页面 3 秒后显示 Camel Case 格式化器的值。

测试这需要使用实际的浏览器，例如 Google Chrome。检查涉及 AJAX、cookies、用户交互或 DOM 的任何内容都不能用常规功能测试完成。

幸运的是，编写功能 JavaScript 测试并没有太大的不同；我们只是扩展了不同的基类进行测试——`WebDriverTestBase` 而不是 `BrowserTestBase`。

需要 Chrome 和 Selenium

如果你使用 DDEV、Lando、Docksal 或其他现成的工具在本地运行 Drupal，请检查它们的文档，了解如何最佳地集成 Chrome 和 Selenium 以进行功能 JavaScript 测试。它们在安装方法上都有所不同。

## 如何做…

在你的 `tests/src` 目录中，创建一个名为 `FunctionalJavascript` 的新目录。在该新目录中，创建一个名为 `CamelCaseFormatterDisplayAjaxTest.php` 的文件。你可以将之前的函数测试代码复制到这个文件中，并进行以下更改：

+   将 `namespace` 从 `Functional` 更改为 `FunctionalJavascript`

+   使用 `WebDriverTestBase` 而不是 `BrowserTestBase` 扩展

+   类名应更改为 `CamelCaseFormatterDisplayAjaxTest`

由于我们现在需要在页面显示此文本周围添加 JavaScript，因此可以删除旧的功能测试，因为它将失败。文本最初不会出现在页面上，旧测试无法监听或等待 AJAX，所以我们不再需要这个测试。

现在，我们可以更新测试方法以适应新的要求。假设 JavaScript 已经编写好，并且字段值是通过 AJAX 获取和输出的。实现细节在这里并不重要，但我们可以更改测试方法以 *等待* AJAX 完成后再检查页面上的文本：

```php
  public function testUserCanSeeFormattedString() {
    $this->drupalCreateNode(
      [
        'type' => 'page',
        'field_chapter13_test' => 'A user entered string'
      ]
    );
    $this->drupalGet('/node/1');
    $this->assertSession()->assertWaitOnAjaxRequest();
    $this->assertSession()->
        pageTextContains('aUserEnteredString');
  }
```

在运行我们的新测试后，这是结果：

```php
Testing
.          1 / 1 (100%)
Time: 00:19.721, Memory: 10.00 MB
OK (1 test, 2 assertions)
```

## 它是如何工作的…

功能 JavaScript 测试是功能测试，增加了在真实浏览器（如 Google Chrome）中运行的功能。这使得可以对 HTML 元素、交互性、AJAX、cookies 以及其他需要使用真实浏览器执行操作的功能进行更多测试。

# 编写 NightwatchJS 测试

与功能 JavaScript 测试类似，NightwatchJS 使用 Google Chrome 来评估测试。然而，测试文件完全是用 JavaScript 编写的，而不是 PHP 文件，并且需要 NodeJS 和 `yarn` 来运行和交互。

## 准备工作

NightwatchJS 不是作为 PHPUnit 的一部分包含的，因此你需要使用 `yarn` 安装它：

1.  在 `/core` 文件夹中，运行 `yarn install`。

1.  配置 `.env.example` 为 `.env` 并根据需要编辑。这些设置将大致与你的本地环境设置相同。`DRUPAL_TEST_BASE_URL` 将是本地站点的 URL 值，例如。

## 如何做…

NightwatchJS 在以下目录中查找测试：

+   `mymodule/tests/src/Nightwatch/Tests`

+   `mymodule/tests/src/Nightwatch/Commands`

+   `mymodule/tests/src/Nightwatch/Assertions`

+   `mymodule/tests/src/Nightwatch/Pages`

在这里，`mymodule` 是你的自定义模块名称，例如 `chapter13`，我们一直在本章的所有代码中使用它。

你可以使用 `yarn` 运行 NightwatchJS 测试：

```php
yarn test:nightwatch (args)
```

或者，您可以运行单个测试——例如，您自定义模块中的测试：

```php
yarn test:nightwatch mymodule/tests/src/Nightwatch/Tests/
  exampleTest.js
```

让我们将`CamelCase` PHP 类转换为 JavaScript 函数，并使用`nightwatch`进行测试。

在`chapter13/js`目录下创建一个名为`js`的目录。在其内部，创建一个名为`camelCase.js`的文件。该文件将包含以下代码：

```php
module.exports = (inputText) => {
  let text = inputText.toLowerCase();
  text = text.replace(/[, -]/g, '_');
  let extractedText = text.split('_').map(function(word,
    index) {
    if (index !== 0) {
      return word.charAt(0).toUpperCase() +
         word.slice(1).toLowerCase();
    } else {
      text = word;
    }
  }).join('');
  text = text.toLowerCase() + extractedText;
  return text.replace('_', '');
}
```

我们可以像在章节开头对 PHP 类进行单元测试一样对这个 JavaScript 函数进行单元测试。为此，在`chapter13/tests/src/Nightwatch`目录下创建一个新的目录，并在该目录内创建一个名为`Tests`的目录。

在`Tests`目录下创建一个名为`CamelCaseTest.js`的文件。它看起来像这样，得到与之前功能 JavaScript 测试相同的结果：

```php
const assert = require('assert');
const camelCase = require('../../../js/camelCase');
const dataProvider = [
  {input: 'button_color', expected: 'buttonColor'},
  {input: 'snake_case_example', expected:
    'snakeCaseExample'},
  {input: 'ALL_CAPS_LOCK', expected: 'allCapsLock'},
  {input: 'foo-bar', expected: 'fooBar'},
];
module.exports = {
  '@tags': ['chapter13'],
  '@unitTest' : true,
  'Strings are converted to camelCase' : function (done) {
    dataProvider.forEach(function (values) {
      assert.strictEqual(camelCase(values.input),
          values.expected);
    });
    setTimeout(function() {
      done();
    }, 10);
  }
};
```

现在执行测试，使用以下命令：

```php
yarn test:nightwatch chapter13/tests/src/Nightwatch/Tests/
  CamelCaseTest.js
```

Nightwatch 将执行 JavaScript 函数，遍历`dataProvider`值，并断言函数返回我们期望的结果：

```php
yarn test:nightwatch ../modules/custom/chapter13/tests/src/
  Nightwatch/CamelCaseTest.js
yarn run v1.22.19
[Nightwatch/CamelCaseTest]
✔ Strings are converted to camelCase
Done in 3.77s.
```

这对于测试您为自定义模块或主题编写的 JavaScript 函数非常有用。手动测试 JavaScript 函数或行为可能是一项耗时的工作。NightwatchJS 可以方便地自动化这项任务。

## 它是如何工作的...

NightwatchJS 也能够执行基于浏览器的测试断言，例如我们在本章前面讨论的功能 JavaScript 测试。然而，也有一些限制，因为 JavaScript 的作用域比 PHP 等语言要有限得多——根据您试图测试的内容，测试设置可能比在 PHP 中要长得多。您当然可以通过在测试中使用 JavaScript 来登录为管理员并导航 Drupal 来创建字段、节点和元素来编写脚本，但这需要大量的 JavaScript，在这种情况下，创建自己的安装配置文件可能更合适。

话虽如此，Drupal 核心中有各种使用 NightwatchJS 的测试，因此请务必参考它们以获取提示、示例和想法。

### 参见

您并不严格限于使用 PHPUnit 或 NightwatchJS 为 Drupal 编写测试。有许多第三方测试框架可用于在 Drupal 中编写和运行针对不同用例的测试。请务必查看以下内容：

+   Behat Drupal 扩展

+   Drupal 测试特性

+   [Cypress.io](https://Cypress.io)
