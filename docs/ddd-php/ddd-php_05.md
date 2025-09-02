# 第五章：服务

您已经看到了实体和值对象是什么。作为基本构建块，它们应该包含任何应用程序的大部分业务逻辑。然而，在某些场景中，实体和值对象并不是最佳解决方案。让我们看看埃里克·埃文斯在他的书中对此有何看法，[领域驱动设计：软件核心的复杂性处理](http://www.amazon.com/Domain-Driven-Design-Tackling-Complexity-Software/dp/0321125215)：

当领域中的某个重要过程或转换不是实体或值对象的自然职责时，将操作添加到模型中，作为独立接口声明的服务。用模型的语言定义接口，并确保操作名称是通用语言的一部分。使服务无状态。

因此，当有需要表示的操作，但实体和值对象不是最佳选择时，您应该考虑将这些操作建模为服务。在领域驱动设计中，通常会遇到三种不同类型的服务：

+   **应用服务**: 在标量类型上操作，将它们转换为领域类型。标量类型可以被认为是领域模型中未知的任何类型。这包括原始类型和不属于领域的类型。我们将在本章中提供一个概述，但若要深入了解此主题，请参阅第十一章应用，*应用*。

+   **领域服务**: 仅在属于领域的类型上操作。它们包含可以在通用语言中找到的有意义的概念。它们包含不适合值对象或实体的操作。

+   **基础设施服务**: 是满足基础设施关注点的操作，例如发送电子邮件和记录有意义的数据。在六边形架构中，它们位于领域边界之外。

# 应用服务

应用服务是外部世界和领域逻辑之间的中间件。这种机制的目的是将来自外部世界的命令转换为有意义的领域指令。

让我们考虑*用户注册到我们的平台*用例。从外部到内部的方法开始：从交付机制，我们需要为我们的领域操作组合输入请求。使用像 Symfony 这样的框架作为交付机制，代码看起来可能像这样：

```php
class SignUpController extends Controller
{
    public function signUpAction(Request $request)
    {
        $signUpService = new SignUpUserService(
            $this->get('user_repository')
        );

        try {
            $response = $signUpService->execute(new SignUpUserRequest(
                $request->request->get('email'),
                $request->request->get('password')
            ));
        } catch (UserAlreadyExistsException $e) {
            return $this->render('error.html.twig', $response);
        }

        return $this->render('success.html.twig', $response);
    }
}

```

正如您所看到的，我们为我们的应用程序服务创建了一个新的实例，传递了所有需要的依赖项——在这个例子中，是一个`UserRepository`。`UserRepository`是一个可以由任何特定技术实现的接口（例如：MySQL、Redis、Elasticsearch）。然后，我们为我们的应用程序服务构建一个请求对象，以便抽象化交付机制——在这个例子中，是一个网络请求——从业务逻辑中。最后，我们执行应用程序服务，获取响应，并使用该响应来渲染结果。在领域一侧，让我们检查一个可能的实现，该实现协调满足“用户注册”用例的逻辑：

```php
class SignUpUserService
{
    private $userRepository;

    public function __construct(UserRepository $userRepository)
    {
        $this->userRepository = $userRepository;
    }

    public function execute(SignUpUserRequest $request)
    {
        $user = $this->userRepository->userOfEmail($request->email);
        if ($user) {
            throw new UserAlreadyExistsException();
        }

        $user = new User(
            $this->userRepository->nextIdentity(),
            $request->email,
            $request->password
        );

        $this->userRepository->add($user);

        return new SignUpUserResponse($user);
    }
}

```

代码中的每一件事都是关于我们想要解决的领域问题，而不是我们用来解决它的特定技术。采用这种方法，我们可以将高级策略与低级实现细节解耦。交付机制与领域之间的通信由称为 DTO 的数据结构携带，我们在第二章中介绍了它，*架构风格*：

```php
class SignUpUserRequest
{
    public $email;
    public $password;

    public function __construct($email, $password)
    {
        $this->email = $email;
        $this->password = $password;
    }
}

```

有不同的策略用于返回内容，但到目前为止，请考虑我们不应该返回我们的实体，这样它们就不能从我们的应用程序服务外部被修改。这就是为什么通常返回另一个包含信息的 DTO，而不是整个实体。让我们看看一个简单的例子：

```php
class SignUpUserResponse
{
    public $id;
    public $email;

    public function __construct(User $user)
    {
        $this->id = $user->id();
        $this->email = $user->email();
    }
}

```

对于创建您的响应，您可以使用 getter 或公共实例变量。应用程序服务应负责事务范围和安全。然而，您将在第十一章中深入了解这些以及其他与应用程序服务相关的内容，*应用程序*。

# 领域服务

在与领域专家的整个对话中，您会遇到一些在通用语言中无法整洁地表示为实体或值对象的概念，例如：

+   用户能够自行登录系统

+   一个购物车能够自行变成订单

上述示例是两个具体概念，它们都不能自然地绑定到实体或值对象。进一步突出这种异常，我们可以尝试如下建模行为：

```php
class User
{
    public function signUp($aUsername, $aPassword)
    {
        // ...
    }
}

class Cart
{
    public function createOrder()
    {
        // ...
    }
}

```

在第一种实现的情况下，我们无法知道给定的用户名和密码与被调用的用户实例相关联。显然，这个操作不适合这个实体；相反，它应该被提取到一个单独的类中，使其意图明确。

考虑到这一点，我们可以创建一个仅负责验证用户的领域服务：

```php
class SignUp
{
    public function execute($aUsername, $aPassword)
    {
        // ...
    }
}

```

类似地，正如第二个示例的情况，我们可以创建一个专门从提供的购物车创建订单的领域服务：

```php
class CreateOrderFromCart
{
    public function execute(Cart $aCart)
    {
        // ...
    }
}

```

领域服务可以被定义为执行领域任务的操作，并且自然不适合实体或值对象。作为代表领域操作的概念，领域服务应该由客户端使用，无论它们的运行历史如何。领域服务本身不持有任何状态，因此领域服务是无状态的操作。

# 领域服务和基础设施服务

在建模领域服务时，遇到基础设施依赖是很常见的——例如，在需要处理密码散列的认证机制的情况下。在这种情况下，你可以使用[分离接口](http://martinfowler.com/eaaCatalog/separatedInterface.html)，这允许定义多个散列机制。使用这种模式仍然在领域和基础设施之间提供了清晰的关注点分离：

```php
namespace Ddd\Auth\Domain\Model;

interface SignUp
{
    public function execute($aUsername, $aPassword);
}

```

使用在领域中找到的先前接口，我们可以在基础设施层创建一个实现，如下所示：

```php
namespace Ddd\Auth\Infrastructure\Authentication;

class DefaultHashingSignUp implements Ddd\Auth\Domain\Model\SignUp
{
    private $userRepository;

    public function __construct(UserRepository $userRepository)
    {
        $this->userRepository = $userRepository;
    }

    public function execute($aUsername, $aPassword)
    {
        if (!$this->userRepository->has($aUsername)) {
            throw UserDoesNotExistException::fromUsername($aUsername);
        }

        $aUser = $this->userRepository->byUsername($aUsername);

        if (!$this->isPasswordValidForUser($aUser, $aPassword)) {
            throw new BadCredentialsException($aUser, $aPassword);
        }

        return $aUser;
    }

    private function isPasswordValidForUser(
        User $aUser, $anUnencryptedPassword
    ) {
        return password_verify($anUnencryptedPassword,$aUser->hash());
    }
}

```

这里是基于 MD5 算法的另一个实现：

```php
namespace Ddd\Auth\Infrastructure\Authentication;

use Ddd\Auth\Domain\Model\SignUp

class Md5HashingSignUp implements SignUp
{
    const SALT = 'S0m3S4lT' ;

    private $userRepository;

    public function __construct(UserRepository $userRepository)
    {
        $this->userRepository = $userRepository;
    }

    public function execute($aUsername, $aPassword)
    {
        if (!$this->userRepository->has($aUsername)) {
            throw new InvalidArgumentException(
                sprintf('The user "%s" does not exist.', $aUsername)
            );
        }

        $aUser = $this->userRepository->byUsername($aUsername);

        if ($this->isPasswordInvalidFor($aUser, $aPassword)) {
            throw new BadCredentialsException($aUser, $aPassword);
        }

        return $aUser;
    }

    private function salt()
    {
        return md5(self::SALT);
    }

    private function isPasswordInvalidFor(
        User $aUser, $anUnencryptedPassword
    ) {
        $encryptedPassword = md5(
            $anUnencryptedPassword . '_' .$this->salt()
        );

        return $aUser->hash() !== $encryptedPassword;
    }
}

```

选择这种方案允许我们在基础设施层拥有多个领域服务接口的实现。换句话说，我们最终会得到几个基础设施领域服务。每个基础设施服务将负责处理不同的散列机制。根据实现方式，用户可以通过依赖注入容器轻松管理使用，例如，通过 Symfony 的依赖注入组件：

```php
<?xml version="1.0"?>
<container

    xsi:schemaLocation="
        http://symfony.com/schema/dic/services
        http://symfony.com/schema/dic/services/services-1.0.xsd">

    <services>

        <service id="sign_in" alias="sign_in.default" />

        <service id="sign_in.default"
            class="Ddd\Auth\Infrastructure\Authentication
            \DefaultHashingSignUp">
            <argument type="service" id="user_repository"/>
        </service>

        <service id="sign_in.md5"
            class="Ddd\Auth\Infrastructure\Authentication
                \Md5HashingSignUp">
            <argument type="service" id="user_repository"/>
        </service>

    </services>
</container>

```

如果在未来，我们希望处理新的散列类型，我们只需从实现领域服务接口开始。然后，只需在依赖注入容器中声明服务，并用新创建的服务别名依赖项替换即可。

# 代码复用问题

虽然之前描述的实现清楚地定义了关注点分离，但我们每次想要实现一个新的散列机制时，都需要重复密码验证算法。解决此问题的另一种方法，它可以提高代码复用性，是将这两个责任分离出来。我们可以将这些密码散列逻辑提取到一个专门的类中，使用[策略模式](http://en.wikipedia.org/wiki/Strategy_pattern)对所有定义的散列算法进行处理。这使设计可以扩展，但修改是封闭的：

```php
namespace Ddd\Auth\Domain\Model;

class SignUp 
{
    private $userRepository; 
    private $passwordHashing;

    public function __construct(
        UserRepository $userRepository, PasswordHashing $passwordHashing
    ) {
        $this->userRepository = $userRepository;
        $this->passwordHashing = $passwordHashing;
    }

    public function execute($aUsername, $aPassword)
    {
        if (!$this->userRepository->has($aUsername)) {
            throw new InvalidArgumentException(
                sprintf('The user "%s" does not exist.', $aUsername)
            );
        }

        $aUser = $this->userRepository->byUsername($aUsername);

        if ($this->isPasswordInvalidFor($aUser, $aPassword)) {
            throw new BadCredentialsException($aUser, $aPassword);
        }

        return $aUser;
    }

    private function isPasswordInvalidFor(User $aUser, $plainPassword)
    {
        return !$this->passwordHashing->verify(
            $plainPassword,
            $aUser->hash()
        );
    }
}

interface PasswordHashing 
{
    /**
     * @param string $password
     * @param string $hash 
     * @return boolean 
     */
    public function verify($plainPassword, hash);
}

```

定义不同的散列策略就像实现`PasswordHashing`接口一样简单：

```php
namespace Ddd\Auth\Infrastructure\Authentication;

class BasicPasswordHashing
    implements \Ddd\Auth\Domain\Model\PasswordHashing
{
    public function verify($plainPassword, $hash)
    {
        return password_verify($plainPassword, $hash);
    }
}

class Md5PasswordHashing
    implements Ddd\Auth\Domain\Model\PasswordHashing
{
    const SALT = 'S0m3S4lT' ;

    public function verify($plainPassword, $hash)
    {
        return $hash === $this-> calculateHash($plainPassword);
    }

    private function calculateHash($plainPassword)
    {
        return md5($plainPassword . '_' .$this-> salt());
    }

    private function salt()
    {
        return md5(self::SALT);
    }
}

```

# 测试领域服务

在多个领域服务实现的用户身份验证示例中，能够轻松测试服务是非常有益的。然而，通常情况下，测试模板方法实现可能会很棘手。因此，我们将使用一个简单的密码散列实现来进行测试：

```php
class PlainPasswordHashing implements PasswordHashing
{
    public function verify($plainPassword, $hash)
    {
        return $plainPassword === $hash;
    }
}

```

现在，我们可以在领域服务中测试所有情况：

```php
class SignUpTest extends PHPUnit_Framework_TestCase
{
    private $signUp;
    private $userRepository;

    protected function setUp()
    {
        $this->userRepository = new InMemoryUserRepository();
        $this->signUp = new SignUp(
            $this->userRepository,
            new PlainPasswordHashing()
        );
    }

    /**
     * @test
     * @expectedException InvalidArgumentException
     */
    public function itShouldComplainIfTheUserDoesNotExist()
    {
        $this->signUp->execute('test-username', 'test-password');
    }

    /**
     * @test
     * @expectedException BadCredentialsException
     */
    public function itShouldTellIfThePasswordDoesNotMatch()
    {
        $this->userRepository->add(
            new User(
                'test-username',
                'test-password'
            )
        );

        $this->signUp->execute('test-username', 'no-matching-password')
    }

    /**
     * @test
     */
    public function itShouldTellIfTheUserMatchesProvidedPassword()
    {
        $this->userRepository->add(
            new User(
                'test-username',
                'test-password'
            )
        );

        $this->assertInstanceOf(
            'Ddd\Domain\Model\User\User',
            $this->signUp->execute('test-username', 'test-password')
        );
    }
}

```

# 贫血型领域模型与丰富型领域模型

你必须小心不要在系统中过度使用领域服务抽象。走这条路可能会导致实体和值对象失去所有行为，仅仅成为数据容器。这与面向对象编程的目标相悖，面向对象编程可以被视为将数据和行为聚集到称为对象的语义单元中，目的是表达现实世界概念和问题。过度使用领域服务可以被视为一种反模式，被称为贫血领域模型。

通常，在开始一个新的项目或功能时，很容易陷入首先建模数据的陷阱。这通常包括认为每个数据库表都有一个直接的一对一对象形式表示。然而，这种想法可能并不总是完全正确。

假设我们被要求建模一个订单处理系统。如果我们首先建模数据，我们可能会得到一个像这样的 SQL 脚本：

```php
CREATE TABLE `orders` (
    `ID` INTEGER NOT NULL AUTO_INCREMENT,
    `CUSTOMER_ID` INTEGER NOT NULL,
    `AMOUNT` DECIMAL(17, 2) NOT NULL DEFAULT '0.00',
    `STATUS` TINYINT NOT NULL DEFAULT 0,
    `CREATED_AT` DATETIME NOT NULL,
    `UPDATED_AT` DATETIME NOT NULL, 
    PRIMARY KEY (`ID`)
) ENGINE=INNODB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

```

从这个角度来看，创建一个`Order`类表示相对容易。这个表示包括所需的访问器方法，用于设置或获取从底层订单数据库表的数据：

```php
class Order 
{ 
    const STATUS_CREATED   = 10;
    const STATUS_ACCEPTED  = 20;
    const STATUS_PAID      = 30;
    const STATUS_PROCESSED = 40;

    private $id;
    private $customerId;
    private $amount;
    private $status;
    private $createdAt;
    private $updatedAt;

    public function __construct(
        $customerId,
        $amount,
        $status,
        DateTimeInterface $createdAt,
        DateTimeInterface $updatedAt
    ) {
        $this->customerId = $customerId;
        $this->amount = $amount;
        $this->status = $status;
        $this->createdAt = $createdAt;
        $this->updatedAt = $updatedAt;
    }

    public function setId($id)
    {
        $this->id = $id;
    }

    public function getId()
    {
        return $this->id;
    }

    public function setCustomerId($customerId)
    {
        $this->customerId = $customerId;
    }

    public function getCustomerId()
    {
        return $this->customerId;
    }

    public function setAmount($amount)
    {
        $this->amount = $amount;
    }

    public function getAmount()
    {
        return $this->amount;
    }

    public function setStatus($status)
    {
        $this->status = $status;
    }

    public function getStatus()
    {
        return $this->status;
    }

    public function setCreatedAt(DateTimeInterface $createdAt)
    {
        $this->createdAt = $createdAt;
    }

    public function getCreatedAt()
    {
        return $this->createdAt;
    }

    public function setUpdatedAt(DateTimeInterface $updatedAt)
    {
        $this->updatedAt = $updatedAt;
    }

    public function getUpdatedAt()
    {
        return $this->updatedAt;
    }
}

```

这种实现的示例用例可能是更新订单状态如下：

```php
// Fetch an order from the database
$anOrder = $orderRepository->find( 1 );

// Update order status
$anOrder->setStatus(Order::STATUS_ACCEPTED);

// Update updatedAt field
$anOrder->setUpdatedAt(new DateTimeImmutable());

// Save the order to the database
$orderRepository->save($anOrder);

```

关于代码复用，这段代码存在与初始用户认证解决方案类似的问题。为了解决这个问题，这种做法的支持者建议使用一个[服务层](http://martinfowler.com/eaaCatalog/serviceLayer.html)，从而使操作变得明确且可重用。现在的先前实现现在可以封装到一个单独的类中：

```php
class ChangeOrderStatusService
{
    private $orderRepository;

    public function __construct(OrderRepository $orderRepository)
    {
        $this->orderRepository = $orderRepository;
    }

    public function execute($anOrderId, $anOrderStatus)
    {
        // Fetch an order from the database
        $anOrder = $this->orderRepository->find($anOrderId);

        // Update order status
        $anOrder->setStatus($anOrderStatus);

        // Update updatedAt field
        $anOrder->setUpdatedAt(new DateTimeImmutable());

        // Save the order to the database
        $this->orderRepository->save($anOrder);
    }
}

```

或者，在更新订单金额的情况下，考虑以下情况：

```php
class UpdateOrderAmountService
{
    private $orderRepository;

    public function __construct(OrderRepository $orderRepository)
    {
        $this->orderRepository = $orderRepository;
    }

    public function execute( $orderId, $amount)
    {
        $anOrder = $this->orderRepository->find(1);

        $anOrder->setAmount($amount);
        $anOrder->setUpdatedAt(new DateTimeImmutable());
        $this->orderRepository->save($anOrder);
    }
}

```

在进行明确意图的操作后，客户端代码将大大减少。

```php
$updateOrderAmountService = new UpdateOrderAmountService(
    $orderRepository
);

$updateOrderAmountService->execute(1, 20.5);

```

实施这种方法可以带来很高的代码复用度。如果有人想要更新订单金额，只需检索一个`UpdateOrderAmountService`实例，并使用适当的参数调用 execute 方法。

然而，选择这条路径打破了讨论的面向对象设计原则，并承担了构建领域模型而不利用任何好处所带来的成本。

# 贫血领域模型破坏封装

如果我们回顾定义服务层中服务的代码，我们可以看到，作为一个使用订单实体的客户端，我们需要了解其实际表示的每一个细节。这个发现违反了面向对象编程的基本规则，即结合数据与随后的行为

# 贫血领域模型带来代码复用的假象

假设有一个实例，客户端绕过`UpdateOrderAmountService`，而是直接从`OrderRepository`获取、更新和持久化。那么，`UpdateOrderAmountService`服务可能拥有的所有额外业务逻辑都不会被执行。这可能导致订单以不一致的状态存储。因此，不变量应该得到正确的保护，而最好的方式是让真正的领域模型来处理。在这个例子中，订单实体将是确保这一点的最佳位置：

```php
class Order 
{ 
    // ...
    public function changeAmount($amount)
    {
        $this->amount = $amount;
        $this->setUpdatedAt(new DateTimeImmutable());
    }
}

```

注意，通过将此操作推入实体并以通用语言命名，系统实现了代码的重用。现在任何希望更改订单数量的人都必须直接调用`Order::changeAmount`方法。

这导致产生了更丰富的类，其中行为是代码重用的目标。这通常被称为丰富的领域模型。

# 如何避免贫血领域模型

避免陷入贫血领域模型的方法是在开始一个新项目或功能时，首先考虑行为。数据库、ORM 等只是实现细节，我们应该努力将使用这些工具的决定推迟到开发过程的后期。这样做，我们可以专注于真正重要的一个属性：行为。

正如与实体一样，领域服务也可以触发第六章，*领域事件*。然而，当事件主要由领域服务而非实体触发时，这又是一个你可能正在创建贫血领域模型的指标。

# 总结

正如我们所见，服务代表了我们系统内的操作，我们可以区分它们的三个版本：

+   **应用服务**：帮助协调来自外部世界的请求到领域。这些服务不应包含领域逻辑。事务在应用级别处理；将你的服务包裹在事务装饰器中会使你的代码对事务不可知。

+   **领域服务**：仅使用领域概念进行操作，这些概念通过通用语言表达。记住推迟实现细节，首先考虑行为，因为滥用领域服务会导致贫血领域模型和糟糕的面向对象设计。

+   **基础设施服务**：在基础设施上操作，例如发送电子邮件或记录信息。

我们最重要的建议是在决定创建领域服务之前，考虑所有选项。首先尝试将业务逻辑移入实体或值。与一些同事讨论。再次审查。如果经过不同的方法，最佳选项是创建领域服务，那么就去做吧。
