# 第五章. 调用测试替身

在本章中，我们将仔细研究测试替身，以便更精确地控制我们的测试，并避免依赖于我们一无所知的接口。

我们将完成上一章开始的工作，并了解如何处理外部依赖，特别是存根和模拟之间的区别。

然后，我们将花费本章的剩余时间来了解如何组织我们的测试，以提高可读性和可维护性，使用我们在书中早期介绍的一些面向 BDD 的工具，例如**Specify**和**Verify**。

在高层次上，这些是我们将在本章中涉及的主题：

+   处理外部依赖

+   使用存根隔离组件

+   使用观察者监听调用

+   编写可维护的单元测试

# 处理外部依赖

我们将我们的单元测试套件留在了几乎完成的状态。我们剩下要测试的是来自我们的`User`模型类的`validatePassword()`方法。

这个方法给我们带来的问题是，我们计划使用我们喜爱的由 Yii 提供的、友好提供的安全组件来加密和解密密码，并验证其正确性。该组件在整个应用程序的生命周期中通过`Yii::$app->getSecurity()`可用。

`yii\base\Security`类公开了一系列方法来帮助您加强您的应用程序。我们将对其的使用相当有限，但我建议您阅读一下官方文档，该文档可在[`www.yiiframework.com/doc-2.0/guide-security-authentication.html`](http://www.yiiframework.com/doc-2.0/guide-security-authentication.html)找到，并阅读以下部分，这些部分将涵盖所有关于身份验证、加密等方面的内容。

让我们定义我们认为这个方法应该如何工作的实现方式。验证密码的文档化用途如下：

```php
public function testValidatePasswordReturnsTrueIfPasswordIsCorrect() {
    $expectedPassword = 'valid password';

    $this->_user->password = Yii::$app->getSecurity()->generatePasswordHash($expectedPassword);

    $this->assertTrue($this->_user->validatePassword($expectedPassword));
}
```

这意味着我们首先需要使用前面提到的辅助类来创建密码散列，然后在用户中设置它，然后可以使用`$user->validatePassword()`方法来检查实际传入的明文密码是否与内部密码匹配。幕后应该发生某种加密/解密操作，理想情况下是通过使用安全组件中的`Security::validatePassword()`。

用户模型中`User::validatePassword()`的一个可能实现可以是以下内容：

```php
// models/User.php

/**
 * Validates password
 *
 * @param  string  $password password to validate
 * @return boolean if password provided is valid for current user
 */
public function validatePassword($password)
{
    return Yii::$app->getSecurity()->validatePassword($password, $this->password);
}
```

如果我们尝试运行测试，这个特定的方法将无问题通过。

这可能是一个不错的解决方案，但我们需要非常清楚，这并不是一个真正的单元测试；它更像是一个集成测试，因为我们仍然依赖于安全组件。

# 使用存根隔离组件

我们目前面临的问题是我们真的不想使用实际的安全组件，因为它不是测试本身的一部分。记住，我们正在一个黑盒环境中工作，我们不知道安全组件未来可能有什么其他依赖。我们只需要确保我们的实现方法在（假的）对象接口按预期工作的情况下将表现正确。我们可以在以后添加一个集成方法来确保安全组件实际上可以工作，但这完全是另一回事。

为了做到这一点，PHPUnit 提供了一个有趣的系统来模拟和存根类，并将它们注入到你的应用程序中以提供一个更受控的环境。一般来说，这些通常被称为**测试替身**，创建它们的方法是通过 Mock Builder。

PHPUnit 的最新版本（4.x 或更高版本）建议使用 Mock Builder 来配置存根和行为。以前，这是通过传递给它的一系列长参数来完成的。我不会沉迷于说如果你正在使用 PHPUnit 3.x 或更早版本，可能到了升级的时候了！

### 注意

请注意，`final`、`private`和`static`方法*不能*被存根或模拟，因为 PHPUnit 测试替身功能不支持这一点。

特别是**存根**指的是用可能返回配置值的测试替身替换对象的实践。

那么，我们是如何做到这一点的呢？我决定采用使用一个单独的私有函数来将存根逻辑委托给可重用代码块的方法：

```php
/**
 * Mocks the Yii Security module,
 * so we can make it return what we need.
 *
 * @param string $expectedPassword the password used for encoding
 *                                 also used for validating if the
 *                                 second parameter is not set
 */
private function _mockYiiSecurity($expectedPassword)
{
    $security = $this->getMockBuilder(
'yii\base\Security')
        ->getMock();
```

我们首先通过使用`getMockBuilder()`创建我们的安全类的存根。默认情况下，Mock Builder 将用返回`null`的测试替身替换所有类方法。

我们也可以选择性地指定要替换哪些方法，通过在数组中指定它们，然后传递给`setMethods()`；例如：`setMethods(['validatePassword', 'generatePasswordHash'])`。

我们还可以传递`null`给它；我们可以避免任何方法有测试替身，但没有它，我们无法设置任何期望。

### 提示

如果你正在模拟的类在`__constructor()`方法中执行了不必要的初始化，你可以通过使用`disableOriginalConstructor()`或传递自定义参数使用`setConstructorArguments()`来禁用它。还有更多方法可以应用于修改 Mock Builder 的行为；请参阅以下文档：[`phpunit.de/manual/current/en/test-doubles.html#test-doubles.stubs`](https://phpunit.de/manual/current/en/test-doubles.html#test-doubles.stubs)。

任何是测试替身的方法都可以使用`expects()`进行配置和监控：

```php
    $security->expects($this->any())
        ->method('validatePassword')
        ->with($expectedPassword)
        ->willReturn(true);

    $security->expects($this->any())
        ->method('generatePasswordHash')
        ->with($expectedPassword)
        ->willReturn($expectedPassword);

    Yii::$app->set('security', $security);
}
```

这看起来相当容易阅读：任何（`any()`）时候方法（`method()`）`'validatePassword'`被调用（`with()`）带有`$expectedPassword`，它将返回（`willReturn()`）true。

你可以以多种方式配置你的替换函数：让它们在连续调用中返回特定的值，或者不同的值，或者当被调用时抛出异常。

### 注意

更多信息和文档可以在官方 PHPUnit 文档中找到，该文档位于[`phpunit.de/manual/current/en/test-doubles.html`](https://phpunit.de/manual/current/en/test-doubles.html)。

然后，我们可以实现负测试（negative test）来覆盖传递给`validatePassword()`方法的错误密码，使用我们想要的逻辑：

```php
/**
 * @expectedException yii\base\InvalidParamException
 */
public function testValidatePasswordThrowsInvalidParamExceptionIfPasswordIsIncorrect() {
    $password = 'some password';
    $wrongPassword = 'some other password';
    $this->_mockYiiSecurity($password, $wrongPassword);

    $this->_user->password = $password;
    $this->_user->validatePassword($wrongPassword);
}
```

为了实现这一点，我们需要稍微重构我们刚刚实现的私有方法：

```php
/**
 * Mocks the Yii Security module,
 * so we can make it returns what we need.
 *
 * @param string $expectedPassword the password used for encoding
 *                                 also used for validating if the
 *                                 second parameter is not set
 * @param mixed $wrongPassword  if passed, validatePassword will
 *                              throw an InvalidParamException
 *                              when presenting this string.
 */
private function _mockYiiSecurity($expectedPassword, $wrongPassword = false)
{
    $security = $this->getMockBuilder(
'yii\base\Security')
        ->getMock()
    );
    if ($wrongPassword) {
        $security->expects($this->any())
            ->method('validatePassword')
            ->with($wrongPassword)
            ->willThrowException(new InvalidParamException());
    } else {
        $security->expects($this->any())
            ->method('validatePassword')
            ->with($expectedPassword)
            ->willReturn(true);
    }
    $security->expects($this->any())
        ->method('generatePasswordHash')
        ->with($expectedPassword)
        ->willReturn($expectedPassword);

    Yii::$app->set('security', $security);
}
```

在这里，我们终于可以看到如何配置我们替换的方法，使用`willThrowException()`抛出异常。有了它，我们可以确保方法抛出了异常；因此，检查异常的测试应该与其他测试分开。

### 注意

官方文档提供了对 Mock Builder API 使用的更详细解释，并且可以在[`phpunit.de/manual/current/en/test-doubles.html`](https://phpunit.de/manual/current/en/test-doubles.html)找到。

# 使用观察者监听调用

由于`User::validatePassword()`现在以透明的方式在其实现中使用了`Security::validatePassword()`，我们不想在设置密码时向使用 User 模型的人暴露任何这些内容。

因此，我们希望认为在设置密码时，我们的实现将以某种方式使用`Security::generatePasswordHash()`，这样当调用`User::validatePassword()`时，我们就能闭合这个循环，并且一切应该都能正常工作，无需过多担心加密方案等问题。

实现一个可以覆盖这部分内容的测试的一个直接、逻辑上合理但相当滥用的方法是以下内容：

```php
public function testSetPasswordEncryptsThePasswordCorrectly()
{
    $clearTextPassword = 'some password';
    $encryptedPassword = 'encrypted password';

    // here, we need to stub our security component

    $this->_user->setPassword($clearTextPassword);

    $this->assertNotEquals(
        $clearTextPassword, $this->_user->password
    );
    $this->assertEquals(
        $encryptedPassword, $this->_user->password
    );
}
```

让我们停下来片刻，理解我们在做什么：理想情况下，我们想要设置一个密码，并加密后读取它，进行相关的和必要的断言。这意味着我们在同一个测试中同时测试密码的设置器和获取器，这再次违反了单元测试的基本原则。

作为旁注，无论我们如何实现安全组件的存根（stubbing），我们的逻辑都不会与本章开头时的初始实现有很大不同。

## 引入模拟

模拟（Mocking）指的是用一个测试替身（test double）替换对象，以验证预期行为，例如确保某个方法已被调用。这似乎正好符合我们的需求。

在一个合适的黑盒场景中，我们不知道`setPassword()`做什么，我们需要完全依赖代码覆盖率来理解我们是否覆盖了编程流程的任何可能分支，正如在第四章中所述，*使用 PHPUnit 进行隔离组件测试*。

仅作为我们目的的示例，我们想确保在调用`setPassword()`时，至少调用一次`Security::generatePasswordHash()`，并且将参数无修改地传递给该方法。

让我们尝试以下方法来测试这个：

```php
public function testSetPasswordCallsGeneratePasswordHash()
{
    $clearTextPassword = 'some password';

    $security = $this->getMockBuilder(
'yii\base\Security')

        ->getMock(
);
    $security->expects($this->once())
        ->method('generatePasswordHash')
        ->with($this->equalTo($clearTextPassword));
    Yii::$app->set('security', $security);

    $this->_user->setPassword($clearTextPassword);
}
```

如你所注意到的，这个测试中没有任何特定的断言。我们的模拟类将在其方法被调用并传入指定值后，简单地标记测试为通过。

### 了解 Yii 虚拟属性

在我们刚才讨论的例子中，如果我们能从用户那里隐藏将明文密码转换为哈希的功能，那将是非常棒的。

这没有发生的原因有很多，但其中最重要的原因是 Yii 已经提供了一个非常有趣且做得很好的虚拟属性系统。这个系统自 Yii 1 以来就存在了，一旦你习惯了它，你就可以取得令人满意的结果。

通过实现一个继承自`yii\base\Component`的模型，例如`ActiveRecord`或`Model`，你也将继承已经实现的魔法函数`__get()`和`__set()`，这些函数可以帮助你处理虚拟属性。当你需要不费吹灰之力创建额外属性时，这尤其有用。

让我们看看一个更实际的例子。

假设我们的用户模型在数据库中有一个名字字段和一个姓氏字段，但我们需要创建全名属性，而不在用户表中添加新列：

```php
namespace app\models;

class User extends ActiveRecord
{
    /**
     * Getter for fullname
     */
    public function getFullname()
    {
        return $this->firstname . ' ' . $this->lastname;
    }

    // rest of the class
}
```

因此，我们可以像访问类的正常属性一样访问字段：

```php
public function testGetFullnameReturnsTheCorrectValue()
{
    $user = new User;
    $user->firstname = 'Rainer';
    $user->lastname = 'Wolfcastle';

    $this->assertEquals(
        $user->firstname . ' ' . $user->lastname,
        $user->fullname
    );
}
```

简单明了的公共属性按预期工作。在下面的代码片段中，我引入了一个新的类`Dog`，仅用于示例目的，它继承自`Model`：

```php
namespace app\models;

use Yii;
use yii\base\Model

class Dog extends Model
{
    public $age;
}
```

因此，我们的测试将毫无问题地通过：

```php
public function testDogAgeIsRecordedCorrectly()
{
    $expectedAge = 7;
    $dog = new Dog;
    $dog->age = $expectedAge;

    $this->assertEquals($expectedAge, $dog->age);
}
```

这一点对你来说应该毫不奇怪，但让我们看看如果我们同时有：

```php
namespace app\models;

class Dog extends ActiveRecord
{
    const AGE_MULTIPLIER = 7;
    public $age;

    public function setAge($age){
        // let's record it in dog years
        $this->age = $age * self::AGE_MULTIPLIER;
    }

    // rest of the class
}
```

现在，我们期待在赋值时触发`setAge()`，而直接读取公共属性的值：

```php
public function testDogAgeIsRecordedInDogYears()
{

    $dog = new Dog;
    $dog->age = 8;

    $this->assertEquals(
        56, 
        $dog->age
    );
}
```

然而，运行这个测试只会揭示一个令人沮丧的事实：

```php
$ ../vendor/bin/codecept run unit models/DogTest.php

1) tests\codeception\unit\models\DogTest::testAgeIsRecordedInDogYears
Failed asserting that 8 matches expected 56.

```

是的，测试确实是相同的。

让我们的类自动处理 getter 和 setter 是有代价的。魔法 setter 执行的检查顺序可以用以下内容来概括：

```php
BaseActiveRecord::__set($name, $value)
  if (BaseActiveRecord::hasAttribute($name))
      $this->_attributes[$name] = $value;
  else
      Component::__set($name, $value)
        if (method_exists($this, 'set'.$name))
            $this->'set'.$name($value);
        if (method_exist($this, 'get'.$name))
            throw new InvalidCallException(...);
        else
            throw new UnknownPropertyException(...);
```

如果你实现了一个继承自`yii\base\ActiveRecord`的模型，其基类将首先检查该属性是否已经作为表列存在；如果没有，它将调用`Component::__set()`。这个方法不仅适用于继承自`yii\base\Model`的模型，也适用于任何隐式继承自`yii\base\Component`的其他类，例如行为和事件。

接着，我们可以看到 setter 将确保我们的类中存在`'set'.$name`方法，如果只有 getter，则将引发异常。

在我们最初定义的 `firstname` 获取器中，我们可以有以下的附加测试：

```php
/**
 * @expectedException yii\base\InvalidCallException
 */
public function testSetFullnameThrowsException()
{
    $user = new User;
    $user->firstname = 'Fido';
    $user->lastname = 'Smith';

    // setter not available
    $user->fullname = 'Something Else';
}
```

关于在设置器中执行的事件和行为，有一些事情需要注意，但我们现在不会触及它们。

因此，回到我们的 `setPassword()` 困境，我们需要意识到，如果我们通过使用 `$user->password` 进行左侧赋值来触发魔法方法，这将不会触发我们的方法。

因此，理想的最佳解决方案可能是以更声明性的方式调用存储的密码，例如 `password_hash`。

# 编写可维护的单元测试

在离开单元测试之前，作为最后一部分，我想展示一些 Codeception 提供的附加功能，这些功能已经在 第一章 中介绍过，*测试心态*。

Codeception 是以模块化和灵活性为设计理念创建的，因此其他任何事情都是你的责任。特别是，你可能已经注意到我们的 `UserTest` 类已经增长了很多。

那么，如果接口或我们的模型工作方式的变化破坏了我们的测试，会发生什么呢？

这非常清楚，特别是如果你在一个团队中工作，或者如果你的代码被其他人接手维护，那么你需要明确的规则，以便每个人都同意如何编写代码，作为一个起点，以及测试。

我已经在 第四章 中突出显示过，*使用 PHPUnit 进行隔离组件测试*，我与我合作过的团队以及我自己的代码开始做的非常基本的事情之一，就是定义精确且简单的规则，这些规则旨在提高代码的清晰度和可读性。我见过太多的“开发者摇滚明星”炫耀他们如何擅长编写压缩代码、嵌套变量和隐藏多个赋值。如果代码是废弃的，编写最终变得难以理解的代码可能很有趣。

代码可读性最终成为我看到公司选择候选人的方式之一，一个非常快速的测试就是让某人阅读你的代码，并能够理解它做什么，而无需询问。

测试不应该比你的应用程序代码得到更少的关注：如果做得恰当，测试代表了一种记录事物应该如何工作以及应该如何使用的方法。一旦你的类提供了更多和更多的功能，你的测试类将开始增长，你需要准备好面对重构并在应用程序发生变化或引入错误时引入回归测试。

## 使用 BDD 规范测试

Codeception 提供了一个叫做 **Specify** 的好用的紧凑工具，我们之前已经介绍过。

仅使用 PHPUnit，我们只有方法来分割我们的测试；使用 Specify，我们有了另一层组织：方法成为我们的 *故事*，我们的规范块成为我们的 *场景*。

仅出于文档目的，应指出 PHPUnit 具有自己的 BDD 兼容的语法糖，即`given()`、`when()`和`then()`方法，如[`phpunit.de/manual/3.7/en/behaviour-driven-development.html`](https://phpunit.de/manual/3.7/en/behaviour-driven-development.html)中所述。如果您更习惯于这种语法，您仍然可以使用它。

作为更清晰的例子，我们可以在同一个测试中分组所有验证规则，并通过使用 Specify 块来分割我们正在执行的定义，如下所示：

```php
use Specify;

public function testValidationRules()
{
    $this->specify(
        'user should not validate if no attribute is set',
        function () {
 verify_not($this->_user->validate());
 }
    );

    $this->specify(
        'user should validate if all attributes are set', 
        function () {
            $this->_user->attributes = [
                'username'=>'valid username',
                'password'=>'valid password',
                'authkey' =>'valid authkey'
            ];
            verify_that($this->_user->validate());
        }
    );
}
```

如我们所见，我们现在正在将两个先前的测试聚合到一个单独的方法中，并将它们分组在两个`specify()`调用块内。

Specify 被定义为一种特质；这就是为什么您需要在最外层的全局作用域中使用命名空间，并在测试类中加载它的原因：

```php
namespace tests\codeception\unit\models;

use Codeception\Specify;
use yii\codeception\TestCase;
// other imported namespaces

class UserTest extends TestCase
{
    use Specify;

    // our test methods will follow
    // we can now use $this->specify()
}
```

如您所见，`specify()`只需要两个参数：即将定义的场景的简单描述，以及包含我们想要执行的断言的匿名函数。

到目前为止，我们可以继续使用我们一直使用的 PHPUnit 经典断言，或者尝试启用 BDD 风格的断言。**Verify**是一个小巧而实用的包，它将提供这种能力，让您可以使用`verify()`、`verify_that()`和`verify_not()`等方法。

从先前提到的场景中，考虑以下代码行：

```php
verify_not($this->_user->validate());
```

这与使用 PHPUnit 断言完全相同：

```php
$this->assertFalse($this->_user->validate());
```

为了执行更复杂的断言，我们可以使用以下方式使用`verify()`：

```php
$this->specify(
    'user with username too long should not validate',
    function () {
        $this->_user->username = 'this is a username longer than 24 characters';

        verify_not($this->_user->validate('username'));
        verify($this->_user->getErrors('username'))->notEmpty();
    }
);
```

### 小贴士

在项目主页[`github.com/Codeception/Verify`](https://github.com/Codeception/Verify)上，还有许多其他断言可以使用，并且可以找到。

# 摘要

在本章中，我们介绍了备受期待的模拟和存根，这将允许您执行适当的组件测试。在最后一部分，我们更好地探讨了测试的代码组织，以及通过使用 Specify 和 Verify 来编写类似 BDD 的方式。

在下一章中，我们将探讨实现功能测试的下一步，这些测试应该定义我们用户的 REST 接口。
