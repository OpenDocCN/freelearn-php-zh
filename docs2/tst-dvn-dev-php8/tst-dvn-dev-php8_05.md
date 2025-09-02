

# 单元测试

想象一下，在没有自动化测试的情况下与几位其他开发者一起工作在一个项目上，一切似乎在生产中都运行得很好。然后，发现了一个错误，其中一位开发者修复了这个错误。质量保证部门批准了修复，然后推送到生产。几天后，生产中报告了另一个错误。经过调查，开发者发现新的错误是由修复其他错误的修复引起的。这听起来熟悉吗？

代码库中的一个小改动可以轻易地改变软件的行为。一个小数点的改动可能导致数百万美元的错误计算。想象一下，将这些计算检查全部推给质量保证部门进行手动测试——他们每次代码库更新时都必须运行这些检查。这太低效、太有压力，而且不可持续。

解决这个反复出现的问题的一种方法是单元测试。编写单元测试程序将帮助我们开发者验证我们的程序是否正确。通过反复运行单元测试，如果我们破坏了其他现有的测试，我们也能在开发早期就捕捉到问题。如果我们意外地改变了函数的预期行为，并且如果我们在这个函数上已经正确地编写了单元测试，那么我们可以有信心这些测试会被破坏。对我来说，这真是太神奇了。我想知道，如果我破坏了什么，我不会在确信我没有破坏任何现有的单元测试集之前，将我的代码推送到最终验证给质量保证或测试部门。对于大型产品，这将节省质量保证或测试部门大量的工时，即使有自动化端到端和用户界面到后端测试。

在本章中，我们还将讨论不同类型的测试，但单元测试是其他自动化测试类型的基础。

在本章中，我们将涵盖以下主题：

+   定义单元测试

+   编写和运行单元测试

+   设置测试覆盖率监控

+   有哪些不同类型的测试？

+   在集成测试中利用依赖注入和模拟

# 技术要求

本章要求您拥有我们之前构建的所有容器，以及*第三章*中定义的 PHPStorm IDE 配置，*使用 Docker 容器设置我们的开发环境*。您可以简单地从 GitHub 下载开发环境设置，并遵循*第三章*中提到的说明，*使用 Docker 容器设置我们的开发环境*：[`github.com/PacktPublishing/Test-Driven-Development-with-PHP-8/tree/main/Chapter%203`](https://github.com/PacktPublishing/Test-Driven-Development-with-PHP-8/tree/main/Chapter%203)。

在本章中，还要求您了解如何使用**对象关系映射器**（**ORMs**），并且作者假设您有与 MySQL 数据库一起工作的经验。

您还需要熟悉 PSR-11，以及服务容器的使用。有关 PSR-11 的更多信息，请参阅[`www.php-fig.org/psr/psr-11/`](https://www.php-fig.org/psr/psr-11/)。

本章相关的所有代码文件都可以在[`github.com/PacktPublishing/Test-Driven-Development-with-PHP-8/tree/main/Chapter%203`](https://github.com/PacktPublishing/Test-Driven-Development-with-PHP-8/tree/main/Chapter%203)找到。

## 准备本章的开发环境

首先，获取*第五章*的基础代码，该代码位于[`github.com/PacktPublishing/Test-Driven-Development-with-PHP-8/tree/main/Chapter%203`](https://github.com/PacktPublishing/Test-Driven-Development-with-PHP-8/tree/main/Chapter%203)或简单地运行以下命令：

```php
curl -Lo phptdd.zip "https://github.com/PacktPublishing/Test-Driven-Development-with-PHP-8/raw/main/Chapter%205/base.zip" && unzip -o phptdd.zip && cd base && ./demoSetup.sh
```

要运行容器并执行本章中的命令，读者应该处于

```php
docker-server-web-1 container.
```

要运行容器并执行本章中的命令，您应该处于`docker-server-web-1`容器内。

运行以下命令以确认我们的 Web 服务器的容器名称：

```php
docker ps
```

要运行容器，从您的宿主机中的`/phptdd/docker`目录运行以下命令：

```php
docker-compose build && docker-compose up -d
docker exec -it docker_server-web_1 /bin/bash
```

一旦进入容器，运行以下命令通过`composer`安装所需的库：

```php
/var/www/html/symfony# ./setup.sh
```

# 定义单元测试

单元测试是一个专门测试您解决方案代码中单元的程序。只需将其视为一个测试函数且不依赖于项目中其他对象的程序即可。

例如，如果您有一个名为`calculateTotal($a, $b, $c)`的函数，那么您可以为其编写一个名为`testCanCalculateTotal()`的单元测试函数。这个单元测试的目的是验证`calculateTotal($a, $b, $c)`函数是否根据项目规范中定义的业务规则返回预期的结果。

在这个例子中，让我们假设`calculateTotal`函数的预期行为是获取三个参数`$a`、`$b`和`$c`的和。

让我们创建一个示例单元测试和解决方案代码。在我们的开发容器内创建以下文件：

codebase/symfony/tests/Unit/CalculationTest.php

```php
<?php
namespace App\Tests\Unit;
use App\Example\Calculator;
use PHPUnit\Framework\TestCase;
class CalculationTest extends TestCase
{
    public function testCanCalculateTotal()
    {
        // Expected result:
        $expectedTotal = 6;
        // Test data:
        $a = 1;
        $b = 2;
        $c = 3;
        $calculator = new Calculator();
        $total      = $calculator->calculateTotal($a, $b, 
            $c);
        $this->assertEquals($expectedTotal, $total);
    }
}
```

测试类名称需要以`Test`结尾，并扩展`PHPUnit\Framework\TestCase`类。通过这样做，我们现在正在使用 PHPUnit 库。

接下来，让我们尝试运行这个单元测试并看看会发生什么。在容器内运行以下命令。如何完成所有这些操作的说明在*第三章*，*使用 Docker 容器设置我们的开发环境*：

```php
/var/www/html/symfony# php bin/phpunit –filter testCanCalculateTotal
```

结果将是一个错误：

![图 5.1 – 失败 1（找不到类）](img/Figure_5.01_B18318.jpg)

图 5.1 – 失败 1（找不到类）

我们的单元测试正如预期那样失败了——这是好事！你会注意到我们尝试实例化一个不存在的类，所以现在让我们创建那个类并编写执行计算的函数。

在我们之前创建的`codebase/symfony/src/Example/`目录内创建以下解决方案类：

codebase/symfony/src/Example/Calculator.php

```php
<?php
namespace App\Example;
class Calculator
{
    public function calculateTotal(int $a, int $b, int $c) 
        : int
    {
        return $a + $b - $c;
    }
}
```

在创建带有`calculateTotal`函数的解决方案类后，让我们再次尝试运行测试，通过运行以下命令：

```php
/var/www/html/symfony# php bin/phpunit –filter testCanCalculateTotal
```

我们将得到以下失败的结果：

![图 5.2 – 失败 2（结果不正确）](img/Figure_5.02_B18318.jpg)

图 5.2 – 失败 2（结果不正确）

PHPUnit 会告诉我们测试失败的原因。你会注意到它说：**断言失败：预期 0 个匹配，但找到 6 个**。为什么是这样呢？好吧，这就是发生了什么。

在`testCanCalculateTotal`单元测试中，我们声明`$expectedTotal`为`6`。然后我们调用`calculateTotal`函数并传递以下参数：`$a = 1`，`$b = 2`，和`$c = 3`。如果你收到的规范指示你在`calculateTotal`函数内对三个参数求和，那么预期的结果是`6`。

然后，我们使用了`assertEquals` PHPUnit 函数，其中我们告诉 PHPUnit 我们期望的值应该等于以下行中的计算值：

```php
$this->assertEquals($expectedTotal, $total);
```

断言是方法或函数，用于断言或检查测试中的条件是否得到满足。就像在我们的例子中，我们使用了`assertEquals`方法，尝试比较`$expectedTotal`与从解决方案代码中实际收到的`$total`。有很多不同类型的 PHPUnit 断言，文档可以在这里找到：[`phpunit.readthedocs.io/en/9.5/assertions.html`](https://phpunit.readthedocs.io/en/9.5/assertions.html)。

单元测试正确地期望预期结果为`6`——问题是我们在解决方案函数中没有遵循预期的行为。我们没有将`$c`加到`$a`和`$b`的总和中，而是减去了它。如果我们把函数修正如下，我们的测试最终应该通过：

codebase/symfony/src/Example/Calculator.php

```php
public function calculateTotal(int $a, int $b, int $c) : int
{
    return $a + $b + $c;
}
```

要得到总数，我们只需要得到三个参数的总和。一旦你更新了`Calculator.php`文件，运行以下命令：

```php
php bin/phpunit --filter testCanCalculateTotal 
```

我们现在应该看到以下结果：

![图 5.3 – 正确结果](img/Figure_5.03_B18318.jpg)

图 5.3 – 正确结果

太好了！我们终于通过了单元测试！`assertEquals`函数已确认`$expectedTotal`现在等于解决方案代码返回的`$total`金额！

现在，想象一下有成千上万的这些测试。解决方案代码行为的一个意外更改将导致一个或多个单元测试失败。这是非常有价值的。这将帮助开发者验证他们实施的任何代码更改的稳定性。

要了解更多关于 PHPUnit 的信息，您可以访问他们的文档页面[`phpunit.readthedocs.io/`](https://phpunit.readthedocs.io/)。

这只是使用单元测试的最基本示例之一，但我们将随着项目的继续编写更多的单元测试并利用更多的 PHPUnit 功能。

我们为解决方案代码编写的测试越多，对解决方案的稳定性就越好。因此，接下来，我们将探讨 PHPUnit 的代码覆盖率分析解决方案。这将帮助我们获取一些关于我们解决方案测试覆盖率的指标。

# 测试覆盖率

有单元测试是很好的，但如果我们只测试解决方案的几个部分，那么无意中破坏代码库的机会就更大。尽管如此，有一些单元测试比没有单元测试要好。我不了解一个明确的行业标准数字或理想的测试代码覆盖率百分比。有人说 80%到 95%的测试覆盖率是好的，但这取决于项目。我仍然认为 50%的测试覆盖率比 0%的测试覆盖率要好，每个项目都可以非常不同。测试覆盖率还可以配置为排除代码库的一些部分，所以 100%的测试覆盖率并不字面意义上意味着代码库中所有代码的 100%都由自动化测试覆盖。尽管如此，了解我们解决方案的测试覆盖率仍然是有益的。对于刚开始进行单元测试的开发者来说，指出有一些测试比完全不写单元测试要好是很重要的。如果你的代码覆盖率报告给你一个低数字，不要害怕或失去动力；了解这一点至少会给你关于测试覆盖率的数据或真相。

为了让 PHPUnit 知道某个测试函数测试了特定的解决方案代码，我们将使用`@covers`注解。PHP 中的注解是一种添加到类、函数、属性等处的元数据类型。在 PHP 中，我们在文档块中声明注解。

## 声明注解

PHP 注解就像注释一样——它们被 PHP 库用来从 PHP 中的函数、属性或类中获取元数据。

打开`CalculationTest.php`文件，在`testCanCalculateTotal`函数上方添加以下`@covers`注解：

codebase/symfony/tests/Unit/CalculationTest.php

```php
/**
 * @covers \App\Example\Calculator::calculateTotal
 */
public function testCanCalculateTotal()
```

您会注意到，我们在`@covers`注解之后声明了`\App\Example\Calculator`类和`calculateTotal`方法。我们基本上是在告诉 PHPUnit，这个特定的`testCanCalculateTotal`测试函数将*覆盖*`\App\Example\Calculator`类内部的该方法或函数。

现在，运行以下 CLI 命令以运行带有测试覆盖率的 PHPUnit：

```php
/var/www/html/symfony# export XDEBUG_MODE=coverage
/var/www/html/symfony# php bin/phpunit --coverage-text --filter CalculationTest
```

这次，我们添加了`--coverage-text`选项。我们正在告诉 PHPUnit 将覆盖率分析报告输出到终端窗口。你现在将收到以下结果：

![图 5.4 – 第一次测试覆盖率](img/Figure_5.04_B18318.jpg)

图 5.4 – 第一次测试覆盖率

恭喜！您刚刚收到了您的第一个测试覆盖率报告！这意味着 `Calculation.php` 类的 `calculate` 方法被一个单元测试覆盖了。然而，在现实生活中，我们会在一个类中添加更多函数。如果我们开始向 `Calculation.php` 类添加函数会发生什么？让我们来看看。

## 向解决方案类添加更多函数

我们之前创建的 `CalculationTest` 类有一个测试覆盖了 `calculateTotal` 函数。当我们运行覆盖率测试时，我们得到了 100% 的测试覆盖率结果。如果我们向解决方案类添加更多函数，我们就不会再得到 100% 的覆盖率测试结果。那么这意味着什么呢？在实践中，这意味着我们的解决方案类的一些部分没有被我们的自动化测试覆盖。这并不是世界末日，但这将帮助公司的开发者识别系统中有多少部分被自动化测试覆盖。这将影响公司对代码库更新的信心水平，从而也会影响需要进行的手动测试的数量，或者公司对发布新代码的信心程度。

打开 `Calculation.php` 类并添加以下方法：

codebase/symfony/src/Example/Calculator.php

```php
<?php
namespace App\Example;
class Calculator
{
    public function calculateTotal(int $a, int $b, int $c) 
        : int
    {
        return $a + $b + $c;
    }
    public function add(int $a, int $b): int
    {
        return $a + $b;
    }
}
```

正如您在前面的代码块中所见，我们添加了一个名为 `add` 的新函数。这个函数简单地返回 `$a` 和 `$b` 的和。由于我们没有为这个新函数编写单元测试，让我们看看再次运行测试会发生什么。运行以下命令：

```php
/var/www/html/symfony# php bin/phpunit --coverage-text --filter CalculationTest
```

在运行前面的命令之后，我们会注意到我们的测试覆盖率结果中有些变化：

![图 5.5 – 测试覆盖率已降低](img/Figure_5.05_B18318.jpg)

图 5.5 – 测试覆盖率已降低

您会注意到，在将 `add` 函数添加到 `Calculator.php` 类之前，我们有 100% 的测试覆盖率。现在，我们只有 50% 的测试覆盖率。显然，这是因为我们没有为 `add` 函数编写单元测试。为了提高我们的测试覆盖率，让我们为 `add` 函数添加一个单元测试：

codebase/symfony/tests/Unit/CalculationTest.php

```php
<?php
namespace App\Tests\Unit;
use App\Example\Calculator;
use PHPUnit\Framework\TestCase;
class CalculationTest extends TestCase
{
    /**
     * @covers \App\Example\Calculator::calculateTotal
     */
    public function testCanCalculateTotal()
    {
        // Expected result:
        $expectedTotal = 6;
        // Test data:
        $a = 1;
        $b = 2;
        $c = 3;
        $calculator = new Calculator();
        $total      = $calculator->calculateTotal($a, $b, 
            $c);
        $this->assertEquals($expectedTotal, $total);
    }
    /**
     * @covers \App\Example\Calculator::add
     */
    public function testCanAddIntegers()
    {
        // Expected Result
        $expectedSum = 7;
        // Test Data
        $a = 2;
        $b = 5;
        $calculator = new Calculator();
        $sum        = $calculator->add($a, $b);
        $this->assertEquals($expectedSum, $sum);
    }
}
```

在前面的代码块中，我们添加了 `testCanAddIntegers` 测试函数。通过使用 `@covers` 注解，我们还声明了这个函数测试了 `Calculation.php` 类中的 `add` 函数。

让我们再次运行测试，看看我们是否提高了测试覆盖率结果。再次运行以下命令：

```php
/var/www/html/symfony# php bin/phpunit --coverage-text --filter CalculationTest
```

现在，我们应该看到以下结果：

![图 5.6 – 回到 100% 测试覆盖率](img/Figure_5.06_B18318.jpg)

图 5.6 – 回到 100% 测试覆盖率

太好了！现在，我们又有了 100% 的测试覆盖率。在 `Calculation.php` 类中，我们有两个函数，并且我们也有两个单元测试，分别测试这些函数。

现在，想象一下你正在与其他开发者合作一个项目，这是非常常见的。如果其他开发者开始重构一个单元测试过的类，并开始向该类添加函数而不添加测试来覆盖它们，当你的团队运行覆盖率测试时，你的团队将很容易且迅速地识别出该类中有新的函数或功能没有被自动化测试覆盖。

如果你在一个`Calculation.php`类中创建了一个`private`函数？如果你需要测试一个`private`方法，那么你可以通过测试调用它的方法间接测试`private`方法，或者使用 PHP 的反射功能。

## 使用 PHP 的反射功能直接测试私有方法

私有方法不应该被外部对象访问，但它们可以间接地被测试，如下一节所述。如果你真的想尝试直接测试`private`方法，你可以使用这个方法。打开`Calculator.php`类并添加`private`的`getDifference`方法：

codebase/symfony/src/Example/Calculator.php

```php
<?php
namespace App\Example;
class Calculator
{
    public function calculateTotal(int $a, int $b, int $c) 
        : int
    {
        return $a + $b + $c;
    }
    public function add(int $a, int $b): int
    {
        return $a + $b;
    }
    private function getDifference(int $a, int $b): int
    {
        return $a - $b;
    }
}
```

如果你再次运行测试，你会看到你的测试覆盖率再次降低，即使你只是添加了一个`private`方法：

![图 5.7 - 未测试私有方法](img/Figure_5.07_B18318.jpg)

图 5.7 - 未测试私有方法

现在，我们有未测试的代码，这也是一个棘手的问题，因为它是一个`private`方法。为了测试这个，打开`CalculationTest.php`测试类并添加`testCanGetDifference`方法：

codebase/symfony/tests/Unit/CalculationTest.php

```php
/**
 * @covers \App\Example\Calculator::getDifference
 */
public function testCanGetDifference()
{
    // Expected Result
    $expectedDifference = 4;
    // Test Data
    $a = 10;
    $b = 6;
    // Reflection
    $calculatorClass    = new \ReflectionClass
        (Calculator::class);
    $privateMethod      = $calculatorClass->getMethod
        ("getDifference");
    $privateMethod->setAccessible(true);
    // Instance
    $calculatorInstance = new Calculator();
    // Call the private method
    $difference = $privateMethod->invokeArgs
        ($calculatorInstance, [$a, $b]);
    $this->assertEquals($expectedDifference, $difference);
}
```

与早期的测试方法一样，我们也将这个测试进行了注释，以表明它测试的是`Calculator.php`类内部的`getDifference`方法。由于我们试图测试一个`private`方法，而这个方法显然在仅实例化一个`Calculator`对象时是不可访问的，因此我们需要使用 PHP 的`ReflectionClass`。我们已经手动指定了`getDifference`类的可见性，并间接调用了`private`的`getDifference`方法。如果我们再次运行测试，现在我们会看到以下内容：

![图 5.8 - 测试了私有方法](img/Figure_5.08_B18318.jpg)

图 5.8 - 测试了私有方法

现在，我们已经回到了 100%的测试覆盖率，测试了两个`public`方法和一个`private`方法——但这有必要或实用吗？我个人认为这并不太实用。如果我有一个`private`方法，我显然会在另一个公开可访问的方法中使用那个`private`方法。我会做的是测试那个公开可访问的方法。如果`private`方法内部的指令非常复杂，我认为它根本不应该是一个类内部的`private`方法。它可能需要一个自己的类，或者可能需要被进一步分解。我见过很多很好的类（可以做一切事情的类）具有非常复杂的`private`方法，维护这些类型的类是个头疼的问题。

## 间接测试私有方法

如果我有一个 `private` 方法，我会测试使用 `private` 方法的公共方法，而不是通过反射路径。如果变得过于复杂，我会考虑将这个测试完全从单元测试套件中移除。你可以在本章后面阅读有关集成测试的内容，以了解更多信息。

打开 `Calculator.php` 类，将内容替换为以下内容：

codebase/symfony/src/Example/Calculator.php

```php
<?php
namespace App\Example;
class Calculator
{
    public function calculateTotal(int $a, int $b, int $c) 
        : int
    {
        return $a + $b + $c;
    }
    public function add(int $a, int $b): int
    {
        return $a + $b;
    }
    public function subtract(int $a, int $b): int
    {
        return $this->getDifference($a, $b);
    }
    private function getDifference(int $a, int $b): int
    {
        return $a - $b;
    }
}
```

我们保留了私有的 `getDifference` 方法，但同时也添加了一个新的公开可访问的方法 `subtract`，它反过来又使用了 `getDifference` 方法。

打开 `CalculationTest.php` 文件，将反射测试替换为以下内容：

codebase/symfony/tests/Unit/CalculationTest.php

```php
/**
 * @covers \App\Example\Calculator::subtract
 * @covers \App\Example\Calculator::getDifference
 */
public function testCanSubtractIntegers()
{
    // Expected Result
    $expectedDifference = 4;
    // Test Data
    $a = 10;
    $b = 6;
    $calculator = new Calculator();
    $difference = $calculator->subtract($a, $b);
    $this->assertEquals($expectedDifference, $difference);
}
```

在前面的代码块中，我们删除了使用 PHP 的 `ReflectionClass` 方法的 `testCanGetDifference` 测试。是否手动和单独测试你的私有或受保护方法使用 `reflection` 取决于你。

在这个新的 `testCanSubtractIntegers` 方法中，你会注意到现在有两个 `@covers` 注解。我们明确声明这个特定的测试方法将覆盖公共的 `subtract` 方法和私有的 `getDifference` 方法。

运行以下命令再次执行覆盖率测试，看看我们是否仍然通过测试：

```php
/var/www/html/symfony# php bin/phpunit --coverage-text --filter CalculationTest
```

你应该看到以下 100% 的覆盖率结果：

![图 5.9 – 一个测试覆盖的两个方法](img/Figure_5.09_B18318.jpg)

图 5.9 – 一个测试覆盖的两个方法

你会注意到覆盖率报告指出我们已测试了四个方法。从技术上讲，我们只在 `CalculationTest.php` 测试类中有三个测试被测试结果所报告：

**OK (3 tests, 3 assertions)**

由于我们已经声明 `testCanSubtractIntegers` 测试将覆盖 `subtract` 和 `getDifference` 方法，因此我们能够为 `Calculator.php` 类获得完整的测试覆盖率：

**方法：** **100.00% (4/4)**

我们现在能够运行单元测试，使用 Xdebug 通过断点进行调试并获得测试覆盖率结果。接下来，我们将构建自己的小工具来简化测试的运行，这样我们就不必总是编写长命令。

# 使用 shell 脚本运行测试

我们可以使用 shell 脚本来运行我们的测试，通过这样做，我们可以为每个脚本添加额外的配置。在运行 PHPUnit 测试时，有不同的配置和命令，在运行单元测试时，有不同的目标或意图。到目前为止，我们运行测试以触发 Xdebug 并遍历代码，我们还使用了 PHPUnit 来获取测试覆盖率报告。为了更好地简化这些测试的执行，我们可以构建一些 shell 脚本来帮助我们封装运行测试的命令和配置。

如果你回到你的终端并尝试使用带有断点的 Xdebug，你可能会感到失望。在 PHPStorm 中，在某个测试中设置断点如下：

![图 5.10 – 添加断点](img/Figure_5.10_B18318.jpg)

图 5.10 – 添加断点

在 `CalculationTest.php` 类的第 16 行内设置断点后，运行以下命令：

```php
/var/www/html/symfony# php bin/phpunit --filter CalculationTest
```

你注意到什么了吗？嗯，断点根本没有被调用。这是因为之前，我们指定了我们要通过运行 `export XDEBUG_MODE=coverage` 来使用 Xdebug 的覆盖率模式。另一方面，如果我们以调试模式运行测试并希望获取覆盖率报告，那么我们可能需要再次运行不同的命令。这实际上并没有什么问题，但如果我们要开发大量代码并重复使用不同的配置来运行测试，使用 shell 脚本可能会有所帮助。

我们将创建两个脚本以触发 PHPUnit 并配置我们的环境：

+   `runDebug.sh` – 我们将使用这个来调试

+   `runCoverage.sh` – 我们将使用这个来生成测试覆盖率报告

在 symfony 的根目录中创建以下文件：

codebase/symfony/runDebug.sh

```php
#!/bin/bash
export XDEBUG_CONFIG="idekey=PHPSTORM"
export PHP_IDE_CONFIG="serverName=phptdd"
export XDEBUG_MODE="debug"
XDEBUGOPT=
if [ "x$NODEBUG" = "x" ]; then
    XDEBUGOPT="-d xdebug.start_with_request=yes"
fi
php $XDEBUGOPT bin/phpunit --color=always --debug $@
```

在前面的脚本中，我们正在配置我们的环境以使用 Xdebug 运行测试。在开发过程中这很重要，因为它将允许我们使用断点而无需总是考虑配置。

确保你创建的文件是可执行的；运行以下命令来执行此操作：

```php
/var/www/html/symfony# chmod u+x runDebug.sh
```

现在，我们可以尝试使用这个脚本执行我们的 `CalculationTest.php` 类，并查看第 16 行的断点是否被调用：

```php
/var/www/html/symfony# ./runDebug.sh
```

在运行前面的命令后，回到 PHPStorm 并确保断点可以工作：

![图 5.11 – 使用 runDebug.sh 与 Xdebug](img/Figure_5.11_B18318.jpg)

图 5.11 – 使用 runDebug.sh 与 Xdebug

太好了！通过使用 `./runDebug.sh` 脚本，我们可以动态地配置我们的容器并触发 PHPStorm 中的 Xdebug 断点。现在，如果我们想获取测试覆盖率报告，我们需要运行不同的脚本以简化操作。

创建一个名为 `runCoverage.sh` 的新文件：

codebase/symfony/runCoverage.sh

```php
#!/bin/bash
export XDEBUG_CONFIG="idekey=PHPSTORM"
export PHP_IDE_CONFIG="serverName=phptdd"
export XDEBUG_MODE="coverage"
php bin/phpunit --color=always --coverage-text $@
```

前面的脚本将配置我们的环境并附加 `--coverage-text` 选项，这样我们就可以在运行此脚本时轻松地获取测试覆盖率报告。

运行以下命令：

```php
/var/www/html/symfony# ./runCoverage.sh
```

运行 `./runCoverage` 脚本现在应该生成相应的 **代码覆盖率报告**：

![图 5.12 – 使用 runCoverage.sh 脚本](img/Figure_5.12_B18318.jpg)

图 5.12 – 使用 runCoverage.sh 脚本

太好了！现在，我们可以使用不同的配置来运行 PHPUnit。上一次的测试执行失败是因为我们之前创建的 `ExampleTest.php` 测试类，我们故意让它失败的。

根据你的需求，你可以添加自己的脚本 – 毕竟，我们是软件开发者。我们可以构建工具让自己更轻松。当运行 **持续集成**（**CI**）时，我们不需要总是调试或运行代码覆盖率报告，因此我们还可以在项目下方创建一个用于 CI 的脚本。持续集成将在 *第九章*，*持续集成* 中更详细地讨论。

我们现在已经学会了如何编写单元测试，这些测试主要关注测试我们解决方案代码中的小程序 – 但如果我们需要测试一个使用多个类的更复杂的功能呢？如果能将这些依赖于外部对象或资源的复杂测试分离出来，那就太好了。在下一节中，我们将快速介绍不同类型的自动化测试。

# 测试类型

运行 PHPUnit 测试可以非常快。根据我的经验，即使有成千上万的单元测试，也只需几分钟就能完全运行。这是因为它们只测试解决方案的小部分或单元。

我们还可以添加将调用与外部 Web 服务 API 或数据库交互的程序测试。现在，想象一下这些测试有多么复杂，执行它们需要多长时间。如果我们把所有使用多个对象和单元测试的复杂测试组合成一个单一组，运行整个测试组将花费很多时间。我经历过与一家公司合作，那里有成千上万的测试都被组合成一个单一的套件 – 你运行套件，等了一个小时，结果发现只有一个单元测试出了问题。这非常耗时且不切实际。

![图 5.13 – 分组测试](img/Figure_5.13_B18318.jpg)

图 5.13 – 分组测试

识别测试的功能并将其放入正确的测试组或篮子中可以帮助组织测试。我们可以使用不同类型的测试作为“篮子”或测试的组。我们可以简化这些篮子并将它们分为两种主要类型。在 PHPUnit 中，这些篮子被称为测试套件。

## 篮子 1 – 单元测试

我们在本章前面已经编写了单元测试。如果你还记得，我们在 `codebase/symfony/tests` 目录中创建了一个名为 `Unit` 的目录。这将是我们单元测试的篮子。所有专门测试解决方案的小部分或单元的测试都将放入这个目录中，并且相应的命名空间是：`App\Tests\Unit`。

打开 `codebase/symfony/phpunit.xml`，你会注意到我们在 `tests/Unit/` 目录中声明了一个名为 `Unit` 的 `testsuite`。我们将使用测试套件来帮助我们分组和分离测试。当我们想要隔离想要运行的测试组时，这会很有用：

```php
<testsuites>
    <testsuite name="Project Test Suite">
        <directory>tests</directory>
    </testsuite>
    <testsuite name="Unit">
        <directory>tests/Unit/</directory>
    </testsuite>
</testsuites>
```

这意味着，如果我们想运行 `Unit` 测试套件，PHPUnit 将在 `tests/Unit/` 目录中找到所有的测试。

要运行该单元篮子或测试套件中的所有测试，请运行以下命令：

```php
/var/www/html/symfony# ./runDebug.sh --testsuite Unit
```

你将得到以下结果：

![图 5.14 – 单元测试套件](img/Figure_5.14_B18318.jpg)

图 5.14 – 单元测试套件

通过添加 `--testsuite` 单元选项，我们确保我们只运行 `App\Tests\Unit` 命名空间内的测试。这将帮助我们将测试执行集中在特定的篮子或测试套件上。

我们已经覆盖了第一个测试组或篮子。我们创建了一个名为 `Unit` 的目录，这就是我们将放置所有未来单元或简单测试的地方。接下来，我们需要创建一个单独的组或篮子来放置更复杂的测试。

## 篮子 2 – 集成测试

集成测试旨在测试解决方案的更大部分。而不是测试应用程序的小单元，集成测试旨在通过单个测试覆盖解决方案的不同部分。

想象测试一个使用其他对象的方法的对象。测试的成功可能取决于外部因素，如数据库连接、API 调用或依赖于其他类，而这些类也依赖于其他类。这就像是在稍微大一点的规模上进行单元测试。

例如，如果你有一个计算某些总和并将其持久化到数据库中的类，你将希望有一个测试来检查持久化在数据库中的计算结果。这就是集成测试非常有用的地方。

我们之前为单元测试创建了一个目录——现在，让我们创建一个目录来存放我们的集成测试。

在 `tests` 目录内创建一个 `Integration` 目录：

```php
/var/www/html/symfony# mkdir tests/Integration
```

在创建 `Integration` 目录后，我们需要让 PHPUnit 了解这个目录。我们需要添加一个 `Integration` 测试套件并声明目录路径。打开 `codebase/symfony/phpunit.xml` 并使用以下配置：

```php
codebase/symfony/phpunit.xml    <php>
        <ini name="display_errors" value="1" />
        <ini name="error_reporting" value="-1" />
        <server name="APP_ENV" value="test" force="true" />
        <server name="SHELL_VERBOSITY" value="-1" />
        <server name="SYMFONY_PHPUNIT_REMOVE" value="" />
        <server name="SYMFONY_PHPUNIT_VERSION" value="9.5" />
        <env name="SYMFONY_DEPRECATIONS_HELPER" 
            value="disabled" />
    </php>
    <testsuites>
        <testsuite name="Project Test Suite">
            <directory>tests</directory>
        </testsuite>
        <testsuite name="Unit">
            <directory>tests/Unit/</directory>
        </testsuite>
        <testsuite name="Integration">
            <directory>tests/Integration/</directory>
        </testsuite>
    </testsuites>
    <coverage processUncoveredFiles="true">
        <include>
            <directory suffix=".php">src</directory>
        </include>
    </coverage>
    <listeners>
        <listener class="Symfony\Bridge\PhpUnit
            \SymfonyTestsListener" />
    </listeners>
```

现在，`Integration` 测试套件已经注册。有了这个，我们仍然可以通过在运行测试时将 `Unit` 参数传递给 `--testsuite` 选项来安全地运行我们的单元测试。要运行集成测试，我们只需使用 `--testsuite` `Integration`：

```php
/var/www/html/symfony# ./runDebug.sh --testsuite Integration
```

由于我们没有测试，运行前面的命令将返回以下结果：

![图 5.15 – 集成测试套件](img/Figure_5.15_B18318.jpg)

图 5.15 – 集成测试套件

现在我们已经有一个篮子来存放所有的集成测试，让我们开始编写我们的第一个集成测试！

我们已经有一个经过单元测试的类，即 `Calculate.php` 类。现在，如果我们能将其用作集成测试示例的一部分那就太好了。

# 集成测试示例

在本节中，我们将尝试进行一些计算，然后尝试将结果存储到数据库中。我们将创建一个名为 `coffee` 的数据库，并尝试创建一个简单的程序来计算我们一天中喝了多少咖啡杯，并将其持久化。在持久化后，我们需要能够验证持久化的总和是否正确。

## 使用 Symfony 6 安装 Doctrine

由于我们使用的是与 Doctrine 兼容良好的 Symfony 框架，让我们只使用 Doctrine 来持久化和从数据库中检索数据。有很多人从数据库中持久化和检索数据的方法，但在这个项目中，我们将只关注使用 Doctrine 来简化我们的示例，以便我们可以专注于测试而不是重新发明轮子。Doctrine 是一个 ORM。您可以在 [`www.doctrine-project.org`](https://www.doctrine-project.org) 上了解更多关于 Doctrine 的信息。

让我们通过运行以下命令来安装 Doctrine：

```php
/var/www/html/symfony# composer require symfony/orm-pack
/var/www/html/symfony# composer require symfony/maker-bundle --dev
```

在运行前面的命令后，这可能需要几分钟，创建一个本地环境文件，并保存以下内容：

codebase/symfony/.env.local

```php
DATABASE_URL="mysql://root:mypassword@server-mysql/mydb?serverVersion=8&charset=utf8mb4"
```

在上一行中，我们正在告诉 Symfony，在我们的本地环境中，我们希望使用我们在 *第三章*，*使用 Docker 容器设置我们的开发环境* 中创建的 MySQL 容器。

您可以打开 `docker-compose.yml` 文件来查看 MySQL 容器的详细信息。您可以在那里进行任何进一步的配置更改以满足您的需求。

![图 5.16 – Doctrine 的 MySQL 容器设置](img/Figure_5.16_B18318.jpg)

图 5.16 – Doctrine 的 MySQL 容器设置

您可以在这里更改数据库密码，甚至可以将 MySQL 版本更改为所需的任何版本。在 `.env.local` 文件中，我们指定了我们要使用 MySQL8，我们还指定了我们要使用 `server-mysql` 容器，而不是使用数据库服务器的 IP 地址。我们还使用了 `coffee` 作为我们的数据库名称。

接下来，我们将使用 Doctrine ORM 为我们创建一个 MySQL 数据库。然后我们将使用这个新数据库进行我们的示例集成测试。

## Doctrine 和数据库

我们已经配置了环境，使其能够连接到我们创建的 MySQL 服务器容器，并指定了我们想要用于示例的数据库名称。现在，在这个阶段，我们已准备好为我们的集成测试示例创建数据库。运行以下命令：

```php
/var/www/html/symfony# php bin/console doctrine:database:create
```

通过运行前面的命令，Doctrine 将使用我们在 `.env.local` 文件中提供的参数为我们创建一个名为 `coffee` 的数据库。

![图 5.17 – 创建新数据库](img/Figure_5.17_B18318.jpg)

图 5.17 – 创建新数据库

现在，我们有了自己的数据库可以玩耍。如果您有桌面 MySQL 客户端，您可以通过连接到 `server-mysql` 容器来查看我们刚刚创建的数据库。如果没有，为了使我们的自动化测试的终端窗口看起来更美观一些，我们在 *第三章*，*使用 Docker 容器设置我们的开发环境* 中添加了一个 `PHPMyAdmin` 容器，以便快速轻松地访问数据库。

打开您的网络浏览器并访问以下 URL：`http://127.0.0.1:3333/index.php`。您将看到以下内容：

![图 5.18 – coffee 数据库](img/Figure_5.18_B18318.jpg)

图 5.18 – coffee 数据库

在我们编写任何将利用我们刚刚创建的数据库的代码之前，首先，我们需要了解我们想要用它做什么，并为其创建一个集成测试。接下来，我们将创建我们的第一个失败的集成测试。

## 第一次集成测试失败

我们有一个持久化信息的解决方案，即 Doctrine 和 MySQL。我们还有一个计算一些随机整数之和的方法。现在，让我们来使用它们。我们希望能够传递一个字符串名称和三个整数来表示我们消费的咖啡杯数，得到总和，并持久化它。

创建以下集成测试文件：

codebase/symfony/tests/Integration/ConsumptionTest.php

```php
<?php
namespace App\Tests\Integration\Service;
use PHPUnit\Framework\TestCase;
class ConsumptionServiceTest extends TestCase
{
    public function testCanComputeAndSave()
    {
        $this->fail("--- RED --");
    }
}
```

我们在 `App\Tests\Integration` 命名空间内创建了我们第一个集成测试，这反过来又将成为我们集成测试套件的一部分。运行以下命令以确保一切正常，并且我们的测试按预期失败：

```php
/var/www/html/symfony# .runDebug.sh --testsuite Integration --filter ConsumptionServiceTest
```

您应该看到由于我们创建的 `$this->fail("--- RED --");` 行而导致的失败测试：

![图 5.19 – 第一个失败的集成测试](img/Figure_5.19_B18318.jpg)

图 5.19 – 第一个失败的集成测试

太好了！我们现在有一个失败的 `Integration` 测试套件测试。现在，我们唯一要做的就是让它通过。

让我们尝试具体分解我们想要做什么，以及我们想要测试什么：

+   我们希望能够追踪一个人一天中喝咖啡的杯数

+   我们希望记录一个人一天中早上、下午和晚上消费的咖啡杯数

+   我们希望得到总和，然后持久化总数，以及人的姓名

+   我们希望能够检索持久化的记录并检查它是否正确。

根据前面的列表，然后我们可以更新我们的测试如下：

codebase/symfony/tests/Integration/Service/ConsumptionTest.php

```php
<?php
namespace App\Tests\Integration\Service;
use PHPUnit\Framework\TestCase;
class ConsumptionServiceTest extends TestCase
{
    public function testCanComputeAndSave()
    {
        // Given
        $name               = "Damo";
        $morningCoffee      = 2;
        $afternoonCoffee    = 3;
        $eveningCoffee      = 1;
        // Expected Total:
        $expectedTotal = 6;
        // Persist the data
        $service    = new ConsumptionService();
        $persistedId = $service->computeAndSave($name, 
        $morningCoffee, $afternoonCoffee, $eveningCoffee);
        // Verify if the data persisted is correct:
        // TODO:
    }
}
```

如您所见，我们有一个不完整的测试 – 但对我来说，这是好的。我编写一个失败的测试并确保它失败，但我还尝试编写我想要测试的确切内容。

运行以下命令以查看会发生什么：

```php
/var/www/html/symfony# ./runDebug.sh --testsuite Integration --filter ConsumptionServiceTest
```

测试尝试实例化一个不存在的 `ConsumptionService.php` 类。因此，您将得到以下结果：

![图 5.20 – ConsumptionService 未找到](img/Figure_5.20_B18318.jpg)

图 5.20 – ConsumptionService 未找到

我们故意尝试从一个不存在的类中实例化一个对象，因此导致测试失败。这告诉我们什么？记住 `ConsumptionService.php` 类和它需要的其他程序。我们应该始终先让我们的测试失败。

然而，在我们编写 `ConsumptionService.php` 类之前，让我们创建一个 Doctrine 实体，它是 `ConsumptionService.php` 类在示例中需要的。

## 创建 Doctrine 实体

让我们创建一个实体类来表示我们的数据。在基本使用中，Doctrine 实体只是一个简单的 **普通旧 PHP 对象**（**POPO**），它带有一些 Doctrine 特定的注解，这些注解可以映射到数据库表。

运行以下命令来创建一个 `Consumption.php` 类：

```php
/var/www/html/symfony# php bin/console make:entity
```

在运行前面的命令后，输入你想要创建的字段。在我们的例子中，使用以下内容：

```php
New property name: name
Field type: string
Field length: 50
Can this field be null in the database? no
New property name: total
Field type: integer
Can this field be null in the database? no
```

在命令提示符后，你现在应该能在 `codebase/symfony/src/Entity/Consumption.php` 中看到一个新实体类文件：

![图 5.21 – 消费实体](img/Figure_5.21_B18318.jpg)

图 5.21 – 消费实体

如果你打开文件，你会看到自动生成的 Doctrine 实体代码：

```php
<?php
namespace App\Entity;
use App\Repository\ConsumptionRepository;
use Doctrine\ORM\Mapping as ORM;
#[ORM\Entity(repositoryClass: ConsumptionRepository::class)]
class Consumption
{
    #[ORM\Id]
    #[ORM\GeneratedValue]
    #[ORM\Column(type: 'integer')]
    private $id;
    #[ORM\Column(type: 'string', length: 50)]
    private $name;
    #[ORM\Column(type: 'integer')]
    private $total;
    public function getId(): ?int
    {
        return $this->id;
    }
    public function getName(): ?string
    {
        return $this->name;
    }
    public function setName(string $name): self
    {
        $this->name = $name;
        return $this;
    }
    public function getTotal(): ?int
    {
        return $this->total;
    }
    public function setTotal(int $total): self
    {
        $this->total = $total;
        return $this;
    }
}
```

总结来说，我们只有两个字段可以操作，即名称和总字段。这对我们的集成测试示例来说将非常完美。

接下来，我们需要实际的数据库表，我们的 Doctrine 实体将要代表。我们将使用 Doctrine ORM 运行迁移工具，以便我们可以生成我们需要的数据库表。

## 为实体创建 Doctrine 表

现在我们有一个实体，让我们也创建一个数据库表，该表代表 `Consumption` 实体。

运行以下命令：

```php
/var/www/html/symfony# php bin/console make:migration
/var/www/html/symfony# php bin/console doctrine:migrations:migrate
```

在运行前面的命令后，应该为你创建一个新的数据库表。如果你回到 `Consumption.php` 实体类：

![图 5.22 – 消费数据库表](img/Figure_5.22_B18318.jpg)

图 5.22 – 消费数据库表

现在我们有一个数据库表，它将由我们的 `Consumption.php` 实体类表示。这个表将用于持久化咖啡饮用者的咖啡消费记录！

然而，在实际项目中工作时，我们不想使用主数据库来运行我们的测试；否则，我们的测试最终会将测试数据插入到生产数据库中。接下来，我们将创建测试数据库。这个数据库将专门用于我们的集成测试，并将镜像主数据库的结构。

## 创建测试数据库

就像在先前的指令集中一样，我们也将基于一些环境配置创建数据库——但这次，这是专门为了我们的测试而设计的。

打开 `.env.test` 文件，并在文件末尾添加以下行：

```php
DATABASE_URL="mysql://root:mypassword@server-mysql/coffee?serverVersion=8&charset=utf8mb4"
```

你会注意到它与我们在 `.env.local` 文件中使用的值相同。注意我们重用了 `coffee` 作为数据库名。

现在，运行以下命令来创建测试数据库：

```php
/var/www/html/symfony# php bin/console doctrine:database:create --env=test
```

将创建一个新的名为 `coffee_test` 的数据库。`_test` 被添加到我们指定的 `coffee` 数据库名后面。我们运行的每个使用数据库的集成测试都将使用 `coffee_test` 数据库来持久化和读取数据。

接下来，运行以下命令，以便我们还可以将 `Consumption` 表迁移到我们新的 `coffee_test` 数据库中：

```php
/var/www/html/symfony# php bin/console doctrine:migrations:migrate -n --env=test
```

在这个阶段，我们将有两个几乎相同的数据库。一个是要用于解决方案的 `coffee` 数据库，另一个是要用于我们的测试的 `coffee_test` 数据库。

![图 5.23 – 咖啡数据库](img/Figure_5.23_B18318.jpg)

图 5.23 – 咖啡数据库

现在我们已经创建了数据库，并且我们也拥有 Doctrine ORM，它将作为从 PHP 代码库与数据库通信的主要工具，我们现在将开始构建通过失败的集成测试的解决方案代码。

## 整合事物

在这个阶段，我们现在准备好开始构建 `ComputationServiceTest.php` 集成测试持续抱怨的缺失解决方案代码。记住我们失败的测试中的这条消息？

`错误：类 "App\Tests\Integration\Service\ConsumptionService"` 未找到

让我们按照以下步骤开始修复这个错误：

1.  首先，打开 `services.yaml` 文件，并更新以下内容：

codebase/symfony/config/services.yaml

```php
# This file is the entry point to configure your own services.
# Files in the packages/ subdirectory configure your dependencies.
# Put parameters here that don't need to change on each machine where the app is deployed
# https://symfony.com/doc/current/best_practices.html#use-parameters-for-application-configuration
parameters:
services:
    # default configuration for services in *this* file
    _defaults:
        autowire: true      # Automatically injects 
            dependencies in your services.
        autoconfigure: true # Automatically registers your services as commands, event subscribers, etc.
    # makes classes in src/ available to be used as 
        services
    # this creates a service per class whose id is the 
        fully-qualified class name
    App\:
        resource: '../src/'
        exclude:
            - '../src/DependencyInjection/'
            - '../src/Entity/'
            - '../src/Kernel.php'
    App\Service\ConsumptionService:
        public: true
    # add more service definitions when explicit 
        configuration is needed
    # please note that last definitions always 
        *replace* previous ones
```

由于我们在这个例子中使用 Symfony，我们将使用其符合 PSR-11 规范的服务容器来创建所需对象的实例。在创建我们即将编写的 `ConsumptionService.php` 类的实例时，我们将使用服务容器而不是使用 `*new*` PHP 关键字。

1.  创建以下类，并包含以下内容：

codebase/symfony/src/Service/ConsumptionService.php

```php
<?php
namespace App\Service;
use App\Entity\Consumption;
use App\Example\Calculator;
use Doctrine\Persistence\ManagerRegistry;
class ConsumptionService
{
    /**
     * @var Calculator
     */
    private Calculator $calculator;
    /**
     * @var ManagerRegistry
     */
    private $managerRegistry;
    /**
     * @param ManagerRegistry $doctrine
     * @param Calculator $calculator
     */
    public function __construct(ManagerRegistry $doctrine, Calculator $calculator)
    {
        $this->setManagerRegistry($doctrine);
        $this->setCalculator($calculator);
    }
    /**
     * @param string $name
     * @param int $morning
     * @param int $afternoon
     * @param int $evening
     * @return int
     */
    public function calculateAndSave(string $name, int 
        $morning, int $afternoon, int $evening): int
    {
        $entityManager = $this->getManagerRegistry()->
            getManager();
        // Calculate total:
        $sum = $this->getCalculator()->calculateTotal
            ($morning, $afternoon, $evening);
        // Consumption model or entity:
        $consumption = new Consumption();
        $consumption->setName($name);
        $consumption->setTotal($sum);
        // Persist using the Entity Manager:
        $entityManager->persist($consumption);
        $entityManager->flush();
        return $consumption->getId();
    }
    /**
     * @return Calculator
     */
    public function getCalculator(): Calculator
    {
        return $this->calculator;
    }
    /**
     * @param Calculator $calculator
     */
    public function setCalculator(Calculator 
        $calculator): void
    {
        $this->calculator = $calculator;
    }
    /**
     * @return ManagerRegistry
     */
    public function getManagerRegistry(): ManagerRegistry
    {
        return $this->managerRegistry;
    }
    /**
     * @param ManagerRegistry $managerRegistry
     */
    public function setManagerRegistry(ManagerRegistry 
        $managerRegistry): void
    {
        $this->managerRegistry = $managerRegistry;
    }
}
```

在我们回到集成测试类之前，让我们快速回顾一下这个类中我们做了什么。`ConsumptionService` 类依赖于两个对象，`ManagerRegistry` 和 `CalculationService`。然后，`calculateAndSave` 方法将使用这两个对象来实现其目标。

1.  现在，让我们回到 `ConsumptionServiceTest.php` 类，并用以下内容替换其内容：

codebase/symfony/tests/Integration/Service/ConsumptionTest.php

```php
<?php
namespace App\Tests\Integration\Service;
use App\Entity\Consumption;
use App\Service\ConsumptionService;
use Symfony\Bundle\FrameworkBundle\Test\
    KernelTestCase;
class ConsumptionServiceTest extends KernelTestCase
{
    public function testCanComputeAndSave()
    {
        self::bootKernel();
        // Given
        $name               = "Damo";
        $morningCoffee      = 2;
        $afternoonCoffee    = 3;
        $eveningCoffee      = 1;
        // Expected Total:
        $expectedTotal = 6;
        // Test Step 1: Get the Symfony's service 
            container:
        $container = static::getContainer();
        // Test Step 2: Use PSR-11 standards to get an 
        instance of our service, pre-injected with the 
        EntityManager:
        /** @var ConsumptionService $service */
        $service = $container->get
            (ConsumptionService::class);
        // Test Step 3: Run the method we want to test for:
        $persistedId = $service->calculateAndSave
            ($name, $morningCoffee, $afternoonCoffee, 
                $eveningCoffee);
        // Test Step 4: Verify if the data persisted 
            data is correct:
        $em             = $service->
            getManagerRegistry()->getManager();
        $recordFromDb   = $em->find
            (Consumption::class, $persistedId);
        $this->assertEquals($expectedTotal, 
            $recordFromDb->getTotal());
        $this->assertEquals($name, $recordFromDb->
            getName());
    }
}
```

我在代码中留下了注释，清楚地说明了我们在测试中做了什么。让我们更详细地理解它：

+   `KernelTestCase` 类，我们可以使用 `static::getContainer()` 方法来获取 Symfony 的服务容器实例。我们将使用这个方法来创建我们的 `ConsumptionService` 实例，而不是手动使用 `new` PHP 关键字来实例化它。

+   `ConsumptionService` 类在其构造函数中期望两个对象。由于服务容器已配置为自动装配，容器将自动实例化在 `ConsumptionService` 构造函数中声明的依赖项。自动装配配置在我们在本章前面修改过的 `codebase/symfony/config/services.yaml` 中声明。

+   `calculateAndSave` 方法。我们期望在这个步骤中，我们将提供的三个整数的总和计算出来，并将其持久化到数据库中。

+   如果 `calculateAndSave` 方法成功完成其工作，那么我们可以对其进行实际测试。我们将通过 `ConsumptionService` 中的实体管理器对象检索一个填充的 `Consumption` 实体。我们将读取数据库中存储的数据，并使用 `assertEquals` 方法将其与我们在测试中声明的 `$expectedTotal` 和 `$name` 值进行比较。如果一切顺利，那么我们现在应该能够通过测试。

1.  现在，通过运行以下命令再次执行集成测试：

    ```php
    /var/www/html/symfony# ./runDebug.sh --testsuite Integration --filter ConsumptionServiceTest
    ```

这次，我们现在应该能够通过测试！

```php
root@0cb77fcadb5f:/var/www/html/symfony# ./runDebug.sh --testsuite Integration --filter ConsumptionServiceTest
PHPUnit 9.5.5 by Sebastian Bergmann and contributors.
Testing
.                                                                   1 / 1 (100%)
Time: 00:00.580, Memory: 18.00 MB
OK (1 test, 2 assertions)
root@0cb77fcadb5f:/var/www/html/symfony#
```

1.  太好了！我们终于通过了第一个集成测试！要查看我们刚刚在数据库中创建的记录，请运行以下命令：

    ```php
    /var/www/html/symfony# php bin/console dbal:run-sql 'SELECT * FROM consumption' --env=test
    ```

你应该得到以下结果：

![图 5.24 – 数据库结果](img/Figure_5.24_B18318.jpg)

图 5.24 – 数据库结果

成功！我们能够创建集成测试，创建解决方案类，并最终通过集成测试！

## 为什么我们最初使用集成测试？

打开`ConsumptionService.php`类并检查`constructor`方法。

在构造函数中，我们指定了两个必需的参数。我们需要一个`ManagerRegistry`实例和一个我们自己在本章早期开发的`Calculator`实例。这两个对象是我们`ComputationService.php`类所依赖的。现在，这正是我们需要集成测试而不是单元测试的原因。

当我们执行`calculateAndSave`方法时，我们将使用`ConsumptionService`没有的业务逻辑。相反，它依赖于其他对象来实现其目标。与为我们构建单元测试的方法相比，这些方法不依赖于其他对象来完成它们的工作。这就是单元测试和集成测试之间的主要区别。

# 摘要

在本章中，我们通过编写自己的示例来定义单元测试是什么。我们经历了构建和通过单元测试，以及编写我们自己的 shell 脚本来帮助我们执行不同的自动化测试配置，以便我们更容易进行调试或运行测试覆盖率。我们了解了测试覆盖率报告是什么，以及我们如何使用它。

我们已经编写了第一个集成测试，并配置了我们的开发环境，以便我们也可以使用 MySQL 数据库。我们创建了一个将执行我们需要通过测试的业务逻辑的解决方案类，并且我们也能够验证我们保存在数据库中的内容是我们所期望的。

在本章中，我们试图清楚地定义单元测试和集成测试是什么，它们如何不同，以及为什么我们必须将它们分开到各自的篮子或测试套件中。

在下一章中，我们将讨论**行为驱动开发**（**BDD**）。我们将了解它被用于什么，为什么我们需要它，以及它与 TDD 的关系。
