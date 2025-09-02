# 应用程序

应用程序层是分隔领域模型（Domain Model）和查询或更改其状态的客户端的区域。应用程序服务是构建此类层的基本块。正如 Vaughn Vernon[所说](https://www.amazon.com/Implementing-Domain-Driven-Design-Vaughn-Vernon/dp/0321834577)：“应用程序服务是领域模型的直接客户端。”你可以将应用程序服务视为外部世界（HTML 表单、API 客户端、命令行、框架、UI 等）与领域模型本身的接触点。通过考虑系统向世界公开的最高级用例可能会有所帮助，例如：“作为访客，我想注册”，“作为已登录用户，我想购买产品”，等等。

在本章中，我们将探讨如何实现应用程序服务，了解命令模式的作用，并确定应用程序服务的责任。为此，让我们考虑*注册新用户*的用例。

从概念上讲，为了注册新用户，我们需要：

+   从客户端获取 email 和密码

+   检查 email 是否已被使用

+   创建一个新用户

+   将此新用户添加到现有用户集

+   返回我们刚刚创建的用户

让我们开始吧。

# 请求

我们需要将`email`和`password`发送到应用程序服务。从客户端（HTML 表单、API 客户端，甚至是命令行）有多种方法可以这样做。我们可以直接通过方法签名发送标准参数（email 和 password），或者构建并发送包含这些信息的数据结构。后一种方法，发送[DTO](http://martinfowler.com/eaaCatalog/dataTransferObject.html)，在桌面上带来了一些有趣的功能。通过发送对象，它将被序列化并排队在命令总线（Command Bus）上。它还可能添加类型安全和一些 IDE 帮助。

数据传输对象

DTO（数据传输对象）是一种在进程之间携带信息的数据结构。不要将其误认为是具有完整功能的对象。DTO 除了存储和检索自己的数据（访问器和修改器）之外，没有任何行为。DTO 是简单的对象，不应包含任何需要测试的业务逻辑。

如 Vaughn Vernon[所说](https://www.amazon.com/Implementing-Domain-Driven-Design-Vaughn-Vernon/dp/0321834577):

应用程序服务方法签名仅使用原始类型（`int`、`strings`等），以及可能的 DTO。然而，作为这些方法的替代方案，更好的方法可能是设计命令对象。这并不一定有对或错。这主要取决于你的品味和目标。

用于存储应用程序服务所需数据的 DTO 实现可能如下所示：

```php
namespace Lw\Application\Service\User;

class SignUpUserRequest
{
    private $email;
    private $password;

    public function __construct($email, $password)
    {
        $this->email = $email;
        $this->password = $password;
    }

    public function email()
    {
        return $this->email;
    }

    public function password()
    {
        return $this->password;
    }
}

```

如你所见，`SignUpUserRequest`没有行为，只有数据。这可以来自 HTML 表单或 API 端点，尽管我们不在乎它是哪一个。

# 构建应用程序服务请求

从交付机制、你最喜欢的框架中创建请求应该相当直接。在网络上，你可以从控制器请求中提取参数，并将它们传递到 DTO 中的服务内部。同样的原则也适用于 CLI 命令：读取输入参数，然后再发送下去。

使用 Symfony，我们可以从`HttpFoundation`组件中提取我们需要的请求对象数据：

```php
// ...
class UsersController extends Controller
{
    /**
     * @Route('/signup', name = 'signup')
     * @param Request $request
     * @return Response
     */
    public function signUpAction(Request $request)
    {
        // ...
        $signUpUserRequest = new SignUpUserRequest(
            $request->get('email'),
            $request->get('password')
        );
        // ...
    }
// ...

```

在一个更复杂的 Silex 应用程序中，它使用`Form`组件来捕获和验证参数，看起来会是这样：

```php
// ...
$app->match('/signup', function (Request $request) use ($app) {
    $form = $app['sign_up_form'];
    $form->handleRequest($request);

    if ($form->isValid()) {
        $data = $form->getData();

        try {
            $app['sign_in_user_application_service']->execute(
                new SignUpUserRequest(
                     $data['email'],
                     $data['password']
                )
            );

            return $app->redirect(
                $app['url_generator']->generate('login')
            );
        } catch (UserAlreadyExistsException $e) {
            $form
                ->get('email')
                ->addError(
                    new FormError(
                        'Email is already registered by another user'
                    )
                );
        } catch (Exception $e) {
            $form
                ->addError(
                    new FormError(
                      'There was an error, please get in touch with us'
                    )
                );
        }
    }

    return $app['twig']->render('signup.html.twig', [
        'form' => $form->createView(),
    ]);
});

```

# 请求设计

当设计你的请求对象时，你应该始终遵循以下原则：使用原始数据类型，设计为可序列化，并且不要在内部包含业务逻辑。这样，你将能够节省单元测试的成本。

# 使用原始数据类型

我们建议使用基本类型来构建你的请求对象——这意味着字符串、整数、布尔值等等。我们只是在抽象化输入参数。你应该能够独立于交付机制来消费应用程序服务。即使是相当复杂的 HTML 表单，在控制器级别也总是被转换成基本类型。你不希望混淆你的框架和业务逻辑。

在某些场景下，直接使用值对象可能很有诱惑力。不要这样做。值对象定义的更新将影响所有客户端，并且你将客户端与你的领域逻辑耦合起来。

# 可序列化

使用基本类型的一个酷炫副作用是，任何请求对象都可以轻松地序列化为字符串，通过网络发送，并存储在消息系统或数据库中。

# 没有业务逻辑

避免在请求对象中放置任何业务逻辑——甚至验证。验证应该在领域内部进行——这是在实体、值对象、领域服务等内部。验证是强制执行业务不变性和领域约束的一种方式。

# 没有测试

应用程序请求是数据结构，而不是对象。单元测试数据结构就像测试 getter 和 setter 一样。没有行为可以测试，所以尝试单元测试请求对象和 DTOs 的价值不大。这些结构将作为更复杂测试（如集成测试或验收测试）的副作用得到覆盖。

命令是请求对象的替代方案。我们可以设计一个具有多个应用程序方法的 Service，每个方法都有你会在请求中放入的参数。这对于简单应用程序来说是可行的，但我们会稍后再讨论这个话题。

# 应用程序服务的解剖结构

一旦我们将数据封装在请求中，就是业务逻辑的时间了。正如 Vaughn Vernon[所说](https://www.amazon.com/Implementing-Domain-Driven-Design-Vaughn-Vernon/dp/0321834577)：“保持应用程序服务瘦，只使用它们在模型上协调任务。”

首先要做的是从请求中提取必要的信息，即`email`和`password`。从高层次来看，我们需要检查是否存在具有特定电子邮件的现有用户。如果不是这种情况，那么我们创建并添加用户到`UserRepository`。在找到具有相同电子邮件的用户这一特殊情况下，我们抛出异常，以便客户端可以按自己的方式处理——通过显示错误、重试或简单地忽略它：

```php
namespace Lw\Application\Service\User;

use Ddd\Application\Service\ApplicationService;
use Lw\Domain\Model\User\User;
use Lw\Domain\Model\User\UserAlreadyExistsException;
use Lw\Domain\Model\User\UserRepository;

class SignUpUserService
{
    private $userRepository;

    public function __construct(UserRepository $userRepository) 
    {
        $this->userRepository = $userRepository;
    }

    public function execute(SignUpUserRequest $request)
    {
        $email = $request->email();
        $password = $request->password();

        $user = $this->userRepository->ofEmail($email);
        if ($user) {
            throw new UserAlreadyExistsException();
        }

        $this->userRepository->add(
            new User(
                $this->userRepository->nextIdentity(),
                $email ,
                $password
            )
        );
    }
}

```

很好！如果你想知道这个`UserRepository`在构造函数中做什么，我们将在下一部分向你展示。

**异常处理**

应用程序服务抛出的异常是向客户端传达异常情况和流程的一种方式。这一层的异常与业务逻辑（如找不到用户）有关，而不是与实现细节（如`PDOException`、`PredisException`或`DoctrineException`）有关。

# 依赖倒置

处理用户不是服务的责任。正如我们在第十章章节 10，“存储库”中看到的，有一个专门处理`User`集合的类：`User`存储库。这是应用程序服务对存储库的依赖。我们不希望应用程序服务与存储库的具体实现耦合，因为那样的话，我们的服务将与基础设施细节耦合。因此，我们依赖于具体实现所依赖的合同（接口），即`UserRepository`。

将`UserRepository`的具体实现构建并传递到运行时——例如，使用`DoctrineUserRepository`，这是一个使用 Doctrine 的具体实现。传递一个具体实现也可以在测试时工作。例如，`NotAvailableUserRepository`可以是一个具体实现，每次执行操作时都会抛出异常。这样，我们可以测试所有应用程序服务的所有行为，包括*悲伤*路径，即当应用程序必须正确行为，即使某些事情出错时。

应用程序服务也可以依赖于领域服务，如`GetBadgesByUser`。在运行时，此类服务的实现可能相当复杂。想象一下一个`HttpGetBadgesByUser`，它通过 HTTP 协议集成一个边界上下文。

根据抽象，我们将使我们的应用程序服务免受底层基础设施更改的影响。

# 实例化应用程序服务

实例化应用程序服务本身很容易，但构建依赖树可能很棘手，这取决于构建依赖的复杂性。为此目的，大多数框架都提供了一个依赖注入容器。如果没有，你最终会在你的控制器中的某个地方得到以下代码：

```php
$redisClient = new Predis\Client([
    'scheme' => 'tcp',
    'host' => '10.0.0.1',
    'port' => 6379
]);

$userRepository = new RedisUserRepository($redisClient);
$signUp = new SignUpUserService($userRepository);
$signUp->execute(new SignUpUserRequest(
    'user@example.com',
    'password'
));

```

我们决定使用 [Redis](http://redis.io/) 实现来为 `UserRepository`。在之前的代码示例中，我们构建了构建使用 Redis 内部实现的仓库所需的所有依赖项。这些依赖项包括：一个 [Predis](https://github.com/nrk/predis) 客户端，以及连接到我们的 Redis 服务器的所有参数。这不仅效率低下，而且还在控制器中传播了重复代码。

你可以将构建逻辑重构为一个工厂，或者你可以使用依赖注入容器——大多数现代框架都自带这个功能。

使用依赖注入容器是不是坏事？

完全不是。依赖注入容器只是一个工具。它们通过抽象出构建依赖项的复杂性来帮助你。它们在构建基础设施工件时非常有用。Symfony 提供了一个完整的解决方案。

请注意，将整个容器作为一个整体传递给某个服务是一种不良做法。这就像将应用程序的整个上下文与领域耦合在一起。如果一个服务需要特定的对象，从你的框架中构建它们，并将它们作为依赖项传递给服务，但不要让该服务了解整个上下文。

让我们看看如何在 Silex 中构建依赖项：

```php
$app = new \Silex\Application();
$app['redis_parameters'] = [
     'scheme' => 'tcp',
     'host' => '127.0.0.1',
     'port' => 6379
];

$app['redis'] = $app->share(function ($app) {
    return new Predis\Client($app['redis_parameters']);
});

$app['user_repository'] = $app->share(function($app) {
    return new RedisUserRepository(
        $app['redis']
    );
});

$app['sign_up_user_application_service'] = $app->share(function($app) {
    return new SignUpUserService(
        $app['user_repository']
    );
});

// ...

$app->match('/signup' ,function (Request $request) use ($app) {
    // ...
    $app['sign_up_user_application_service']->execute(
        new SignUpUserRequest(
            $request->get('email'),
            $request->get('password')
        )
    );
    // ...
});

```

正如你所见，`$app` 被用作服务容器。我们注册所有需要的组件及其依赖项。`sign_up_user_application_service` 依赖于上面定义的内容。更改 `user_repository` 的实现就像返回其他东西（MySQL、MongoDB 等）一样简单，所以我们根本不需要更改服务代码。

对于 Symfony 应用程序，其等效操作如下：

```php
<?xml version=" 1.0" ?>
<container 

    xsi:schemaLocation="
        http://symfony.com/schema/dic/services
        http://symfony.com/schema/dic/services/services-1.0.xsd">
    <services>
        <service
            id="sign_up_user_application_service"
            class="SignUpUserService">
            <argument type="service" id="user_repository" />
        </service>

        <service
            id="user_repository"
            class="RedisUserRepository">
            <argument type="service">
                <service class="Predis\Client" />
            </argument>
        </service>
    </services> 
</container>

```

现在你已经在 Symfony 服务容器中定义了你的应用程序服务，稍后获取它就非常简单了。所有交付机制——Web 控制器、REST 控制器，甚至是控制台命令——都共享相同的定义。服务在实现 `ContainerAware` 接口的任何类上都是可用的。获取服务就像调用 `$this->get('sign_up_user_application_service')` 一样简单。

总结一下，你如何构建你的服务（临时、使用服务容器、使用工厂等）并不重要。然而，保持你的应用程序服务设置在基础设施边界之外是很重要的。

# 定制应用程序服务

定制你的应用程序服务的主要方式是选择你传递的依赖项。根据你的服务容器功能，这可能会有些棘手，因此你还可以添加一个 setter 来动态更改依赖项。例如，你可能需要更改输出依赖项，以便你可以设置一个默认值，然后之后再更改它。如果逻辑变得过于复杂，你可以创建一个应用程序服务工厂来为你处理这种情况。

# 执行

调用应用服务有两种不同的方法：每个用例一个专用类，具有单个执行方法，以及同一类中的多个应用服务和用例。

# 每个应用服务一个类

这是我们首选的方法，可能也是适合所有场景的方法：

```php
class SignUpUserService 
{ 
    // ...
    public function execute(SignUpUserRequest $request)
    {
       // ...
    }
}

```

使用每个应用服务一个专用类可以使代码更健壮，抵御外部变化（单一职责原则）。由于服务只做一件事情，因此更改类的理由更少。由于应用服务做的事情较少，因此它将更容易进行测试。实现一个通用的应用服务合同更容易，这使得类装饰更容易（参见第十章的子节*事务*，*存储库*）。这还将导致更高的内聚性，因为所有依赖项都专门用于单个用例。

`execution`方法可以有一个更具表达力的名称，比如`signUp`。然而，执行[命令模式](http://martinfowler.com/bliki/DecoratedCommand.html)格式标准化了应用服务之间的通用合同，从而使得装饰变得容易，这在事务中很有用。

# 每个类中多个应用服务方法

有时候，将连贯的应用服务分组在同一类下可能是个好主意：

```php
class UserService
{
    // ...
    public function signUp(SignUpUserRequest $request)
    {
        // ...
    }

    public function signIn(SignUpUserRequest $request)
    {
        // ...
    }

    public function logOut(LogOutUserRequest $request)
    {
        // ...
    }
}

```

我们不推荐这种方法，因为并非所有应用服务都是 100%连贯的。一些服务将需要不同的依赖项，你最终会得到依赖它们不需要的东西的应用服务。另一个问题是这种类型的类会迅速增长。因为它违反了单一职责原则，所以会有多个理由去更改它，甚至可能破坏它。

# 返回值

在注册后，我们可能会考虑将用户重定向到个人资料页面。将所需信息直接返回给控制器的自然方式是从服务中返回用户实体：

```php
class SignUpUserService
{
    // ...

    public function execute(SignUpUserRequest $request)
    {
        $user = new User(
            $this->userRepository->nextIdentity(),
            $email,
            $password
        );

        $this->userRepository->add($user);

        return $user;
    }
}

```

然后，从控制器中，我们会获取`id`字段并重定向到其他地方。然而，仔细考虑我们刚才所做的事情。我们返回了一个功能齐全的实体给控制器，这将允许交付机制绕过应用层并直接与领域交互。

假设`User`实体提供了一个`updateEmailAddress`方法。你可以尝试阻止它，但在未来的某个时刻，有人可能会考虑使用它：

```php
$app-> match( '/signup' , function (Request $request) use ($app) {
   // ...
   $user = $app['sign_up_user_application_service']->execute(
       new SignUpUserRequest(
           $request->get('email'),
           $request->get('password'))
   );
   $user->updateEmailAddress('shouldnotupdate@email.com');
   // ...
});

```

不仅如此，表示层所需的数据与领域管理的数据并不相同。我们不希望领域层围绕表示层进行演变和耦合。相反，我们希望它们自由地演变。

要做到这一点，我们需要一种灵活的方式来解耦这两层。

# 从聚合实例中获取 DTO

我们可以返回带有表示层所需信息的无状态数据结构。正如我们之前所看到的，DTOs（数据传输对象）适合这种场景。我们只需要在应用服务中组合它们，然后返回给客户端：

```php
class UserDTO
{
    private $email ;
    // ...

    public function __construct(User $user)
    {
        $this->email = $user->email ();
        // ...
    }

    public function email ()
    {
        return $this->email ;
    }
}

```

`UserDTO`将暴露我们在表示层从`User`实体中需要的所有只读数据，从而避免暴露行为：

```php
class SignUpUserService
{
    public function execute(SignUpUserRequest $request)
    {
        // ...

        $user = // ...

        return new UserDTO($user);
    }
}

```

任务完成。现在我们可以向模板引擎传递参数，并将它们转换为小部件、标签或子模板，或者对表示层上的数据进行任何我们想要的操作：

```php
$app->match('/signup' , function (Request $request) use ($app) {
    /**
     * @var UserDTO $user
     */
    $userDto=$app['sign_up_user_application_service']->execute(
        new SignUpUserRequest(
            $request->get('email'),
            $request->get('password')
        )
    );

    // ...
});

```

然而，让应用服务决定如何构建 DTO 揭示了另一个限制。因为构建 DTO 完全取决于应用服务，所以适应不同客户端的 DTO 将非常困难。考虑 Web 控制器上的重定向所需的数据和同一用例的 REST 响应所需的数据。这根本不是相同的数据。

让客户端通过传递特定的 DTO 组装器来定义如何构建 DTO：

```php
class SignUpUserService
{
    private $userDtoAssembler;

    public function __construct(
        UserRepository $userRepository,
        UserDTOAssembler $userDtoAssembler
    ) {
        $this->userRepository = $userRepository;
        $this->userDtoAssembler = $userDtoAssembler;
    }

    public function execute(SignUpUserRequest $request)
    {
        $user = // ...

        return $this->userDtoAssembler->assemble($user);
    }
}

```

现在客户端可以通过传递特定的`UserDTOAssembler`来自定义响应。

# 数据转换器

在某些情况下，为更复杂的响应（如 JSON、XML、CSV 和 iCAL 联系人）生成中间 DTO 可能被视为不必要的开销。我们可以在缓冲区中输出表示，并在交付时再请求它。

转换器通过将高级领域概念转换为低级客户端细节来帮助减少这种开销。让我们看看一个例子：

```php
interface UserDataTransformer
{
    public function write(User $user);

    /**
     * @return mixed
     */
    public function read();
}

```

考虑为给定产品生成不同数据表示的情况。通常，产品信息通过 Web 界面（HTML）提供，但我们可能对提供其他格式（如 XML、JSON 或 CSV）感兴趣。这可能使与其他服务的集成成为可能。

考虑一个类似的博客案例。我们可能将我们的写作潜力通过 HTML 展示给世界，但有些人可能对通过 RSS 消费我们的文章感兴趣。用例——应用服务——保持不变。表示则不同。

DTOs（数据传输对象）是一个干净且简单的解决方案，可以传递给模板引擎以实现不同的表示，但这可能会使数据转换的最后一步的逻辑变得复杂，因为这样的模板的逻辑可能成为维护、测试和理解的问题。

在特定情况下，数据转换器可能是一个更好的方法。这些只是以领域概念（聚合、实体等）作为输入，以只读表示（XML、JSON、CSV 等）作为输出的黑盒。这些转换器可能非常容易测试：

```php
class JsonUserDataTransformer implements UserDataTransformer
{
    private $data;

    public function write(User $user)
    {
        // More complex logic could be placed here
        // As using JMSSerializer, native json, etc.
        $this->data = json_encode($user);
    }

    /**
     * @return string
     */
    public function read()
    {
        return $this->data;
    }
}

```

这很简单。想知道 XML 或 CSV 的样子吗？让我们看看如何将数据转换器集成到我们的应用服务中：

```php
class SignUpUserService
{
    private $userRepository;
    private $userDataTransformer;

    public function __construct(
        UserRepository $userRepository,
        UserDataTransformer $userDataTransformer
    ) {
        $this->userRepository = $userRepository;
        $this->userDataTransformer = $userDataTransformer;
    }

    public function execute(SignUpUserRequest $request)
    {
        $user = // ...
        $this->userDataTransformer()->write($user);
    }

    /**
     * @return UserDataTransformer
     */
    public function userDataTransformer()
    {
        return $this->userDataTransformer;
    } 
}

```

这与 DTO 组装器方法类似，但这次没有返回具体值。数据转换器被用来持有和交互数据。

DTO 的问题主要在于编写它们的开销。大多数时候，你的领域概念和 DTO 表示将呈现相同的结构。大多数时候，你可能会觉得花时间进行这种映射不值得。话虽如此，表示和聚合之间的关系不是 1:1。你可以在单个表示中一起表示两个聚合。你也可以用多种方式表示相同的聚合。你如何做总是取决于你的用例。

然而，根据 [马丁·福勒](http://www.martinfowler.com/books/eaa.html)：

有一种情况下使用类似 DTO 的东西是有用的，那就是**当你的表示层模型与底层领域模型之间存在显著不匹配时**。在这种情况下，创建一个针对表示层的特定外观/网关，它将领域模型映射到方便表示的接口是有意义的。它与表示模型很好地结合在一起。这样做是值得的，但只有在存在这种不匹配的屏幕上才值得（在这种情况下，这不是额外的工作，因为你本来也必须在屏幕上这样做。）

我们认为长期愿景将值得投资。在中到大型的项目中，界面表示和领域概念的变化节奏非常不同。你可能希望将它们彼此解耦，以降低更新的摩擦。使用 DTO 或数据转换器允许你自由地发展你的模型，而无需总是考虑破坏布局。

# 复合布局上的多个应用程序服务

大多数时候，没有布局像单个应用程序服务那样简单。我们的项目有相当复杂的界面。

考虑一个特定项目的首页。我们如何渲染这么多部分和用例？有几个选项，让我们来看看。

# AJAX 内容集成

你可以让浏览器直接请求不同的端点，并通过 AJAX 或 [Hijax](https://en.wikipedia.org/wiki/Hijax) 在布局中立即组合数据。这将避免在你的控制器中混合大量的应用程序服务，但可能会因为触发的请求数量而带来性能损失。

# ESI 内容集成

**边缘侧包含**（**ESI**）是一种类似于之前方法的微小标记语言，但它在服务器端。它需要额外的努力来配置额外的中间件，如 NGINX 或 Varnish，以使其工作。包含（ESI）是一种类似于之前方法的微小标记语言，但它在服务器端。它需要额外的努力来配置额外的中间件，如 NGINX 或 [Varnish](https://en.wikipedia.org/wiki/Edge_Side_Includes)，以使其工作。

# Symfony 子请求

如果你使用 Symfony，子请求可能是一个有趣的选择。根据 [Symfony 文档](http://symfony.com/doc/current/components/http_kernel/introduction.html#sub-requests)：

除了发送到`HttpKernel::handle`的主要请求之外，你还可以发送所谓的子请求。子请求看起来和表现得像任何其他请求一样，但通常只用于渲染页面的一小部分而不是整个页面。你通常会在控制器（或可能是在控制器渲染的模板内部）中创建子请求。这创建了一个新的完整请求-响应周期，其中这个新的请求被转换为一个响应。唯一的内部区别是，一些监听器（例如：安全）可能只对主请求起作用。每个监听器都会传递一些`KernelEvent`的子类，其中`isMasterRequest()`可以用来检查当前请求是主请求还是子请求。

这很棒，因为你可以获得调用单独应用程序服务的优势，而不受 AJAX 惩罚或复杂的 ESI 配置。

# 一个控制器，多个应用程序服务

最后一个选项可能是管理同一控制器内的多个应用程序服务，尽管控制器逻辑可能会变得有些复杂，因为它将处理和合并响应以传递给视图。

# 测试应用程序服务

由于你对测试应用程序服务本身的行为感兴趣，没有必要将其转换为与真实数据库进行复杂设置的集成测试。你不对测试低级细节感兴趣，所以大多数情况下，单元测试就足够了：

```php
class SignUpUserServiceTest extends \PHPUnit_Framework_TestCase
{
    /**
     * @var \Lw\Domain\Model\User\UserRepository
     */
    private $userRepository;

    /**
     * @var SignUpUserService
     */
    private $signUpUserService;

    public function setUp()
    {
        $this->userRepository = new InMemoryUserRepository();
        $this->signUpUserService = new SignUpUserService(
            $this->userRepository
        );
    }

    /**
     * @test
     * @expectedException   
     *     \Lw\Domain\Model\User\UserAlreadyExistsException
     */
    public function alreadyExistingEmailShouldThrowAnException()
    {
        $this->executeSignUp();
        $this->executeSignUp();
    }

    private function executeSignUp()
    {
        return $this->signUpUserService->execute(
            new SignUpUserRequest(
                'user@example.com',
                'password'
            )
        );
    }

    /**
     * @test
     */
    public function afterUserSignUpItShouldBeInTheRepository()
    {
        $user = $this->executeSignUp();

        $this->assertSame(
            $user,
            $this->userRepository->ofId($user->id())
        );
    }
}

```

我们已经为`User`存储库使用了一个内存实现。这被称为模拟：一个完全功能化的存储库实现，将使我们的单元测试工作正常。我们不需要访问数据库来测试这个类的行为。这会使我们的测试变慢且脆弱。

检查域事件提交可能也有趣。如果创建用户触发了用户注册事件，确保它已被触发可能是个好主意：

```php
class SignUpUserServiceTest extends \PHPUnit_Framework_TestCase
{
    // ...

    /**
     * @test
     */
    public function itShouldPublishUserRegisteredEvent()
    {
        $subscriber = new SpySubscriber();
        $id = DomainEventPublisher::instance()->subscribe($subscriber);

        $user = $this->executeSignUp();
        $userId = $user->id();

        DomainEventPublisher::instance()->unsubscribe($id);
        $this->assertUserRegisteredEventPublished(
            $subscriber, $userId
        );
    }  

    private function assertUserRegisteredEventPublished(
        $subscriber, $userId
    ) {
        $this->assertInstanceOf(
            'UserRegistered', $subscriber->domainEvent
        );
        $this->assertTrue(
            $subscriber->domainEvent->userId()->equals($userId)
        );
    }
}

class SpySubscriber implements DomainEventSubscriber
{
    public $domainEvent;

    public function handle($aDomainEvent)
    {
        $this->domainEvent = $aDomainEvent;
    }

    public function isSubscribedTo($aDomainEvent)
    {
        return true;
    }
}

```

# 事务

事务是与持久化机制相关的实现细节。领域层不应该意识到这个低级实现细节。在这个级别考虑开始、提交或回滚事务是一个很大的问题。这个级别的细节属于基础设施层。

处理事务的最佳方式是根本不处理它们。我们可以用处理事务会话的装饰器实现来包装我们的应用程序服务。

我们在我们的存储库中实现了一个解决方案来解决这个问题，你可以在这里查看[它](https://github.com/dddinphp/ddd)：

```php
interface TransactionalSession
{
    /**
     * @return mixed
     */
    public function executeAtomically(callable $operation);
}

```

此合约接受一段代码并原子性地执行它。根据你的持久化机制，你将得到不同的实现。

让我们看看我们如何使用 Doctrine ORM 来实现它：

```php
class DoctrineSession implements TransactionalSession
{
    private $entityManager;

    public function __construct(EntityManager $entityManager)
    {
        $this->entityManager = $entityManager;
    }

    public function executeAtomically(callable $operation)
    {
        return $this->entityManager->transactional($operation);
    }
}

```

这就是客户端如何使用之前代码的方式：

```php
/** @var EntityManager $em */
$nonTxApplicationService = new SignUpUserService(
    $em->getRepository('BoundedContext\Domain\Model\User\User')
);

$txApplicationService = new TransactionalApplicationService(
    $nonTxApplicationService,
    new DoctrineSession($em)
);

$response = $txApplicationService->execute(
    new SignUpUserRequest(
        'user@example.com',
        'password'
    )
);

```

现在我们有了事务性会话的 Doctrine 实现，创建一个用于我们的应用服务的装饰器将非常棒。采用这种方法，我们使事务性请求对领域透明：

```php
class TransactionalApplicationService implements ApplicationService
{
    private $session;
    private $service;

    public function __construct(
        ApplicationService $service, TransactionalSession $session
    ) {
        $this->session = $session;
        $this->service = $service;
    }

    public function execute(BaseRequest $request)
    {
        $operation = function () use ($request) {
            return $this->service->execute($request);
        };

        return $this->session->executeAtomically($operation);
    }
}

```

使用 Doctrine 会话的一个很好的副作用是它自动管理 flush 方法，因此你不需要在领域或基础设施中添加 flush。

# 安全

如果你想知道如何管理和处理用户凭证和一般安全，除非这是你的领域的责任，我们建议让框架来处理。用户会话是交付机制的关心点。将此类概念引入领域将使开发变得更加困难。

# 领域事件

在应用服务执行之前必须配置领域事件监听器，否则没有人会注意到。在某些情况下，你必须在执行应用服务之前明确配置监听器：

```php
// ...
$subscriber = new SpySubscriber();
DomainEventPublisher::instance()->subscribe($subscriber);

$applicationService = // ...
$applicationService->execute(...);

```

大多数情况下，这将通过配置依赖注入容器来完成。

# 命令处理器

通过命令总线库执行应用服务是一种有趣的方式。一个好的选择是[Tactician](https://tactician.thephpleague.com/)。从 Tactician 网站：

命令总线是什么？这个术语通常在我们将[命令模式](https://en.wikipedia.org/wiki/Command_pattern)与[服务层](http://martinfowler.com/eaaCatalog/serviceLayer.html)结合使用时使用。它的任务是取一个命令对象（描述用户想要做什么）并将其与处理器（执行它）匹配。这可以帮助你整洁地组织代码。

— 我们的应用服务是服务层，我们的请求对象看起来几乎像是命令。

没问题——我们的应用服务是服务层，我们的请求对象看起来几乎像是命令。如果我们有一个机制来链接所有应用服务，并根据请求执行正确的服务，那岂不是很好？实际上，这就是命令总线的作用。

# 战术家库和其他选项

Tactician 是一个命令总线库，它允许你为你的应用服务使用命令模式。它特别方便用于应用服务，但你也可以使用任何类型的输入。

让我们看看[Tactician](http://tactician.thephpleague.com/)网站上的一个例子：

```php
// You build a simple message object like this:
class PurchaseProductCommand
{
    protected $productId;
    protected $userId;

    // ...and constructor to assign those properties...
}

// And a Handler class that expects it:
class PurchaseProductHandler
{
    public function handle(PurchaseProductCommand $command)
    {
        // use command to update your models, etc
    }
}
// And then in your Controllers, you can fill in the command using your favorite
// form or serializer library, then drop it in the CommandBus and you're done!
$command = new PurchaseProductCommand(42, 29);
$commandBus->handle($command);

```

就这样。Tactician 是`$commandBus`服务。它负责找到正确的处理器和方法的所有管道工作，这可以避免大量的样板代码。在这里，命令和处理器只是普通类，但你可以根据你的应用更好地配置它们。

总结来说，我们可以得出结论，命令只是请求对象，命令处理器只是应用服务。

Tactician（以及命令总线一般）的一个酷特点是它们非常容易扩展。Tactician 为常见任务提供了插件，如日志记录和数据库事务。这样，你就可以忘记在每个处理器上设置连接了。

[Tactician](http://bernard.readthedocs.org/) 的另一个有趣的插件是 Bernard 集成。Bernard 是一个异步作业队列，允许你将一些任务留待稍后处理。重处理过程会阻塞响应。大多数时候，我们可以分支并延迟它们的执行。为了获得最佳体验，尽快回答客户的问题，并在分支过程完成后通知他们。

Matthias Noback 开发了一个类似的项目，名为 [SimpleBus](http://simplebus.github.io/MessageBus/)，它可以作为 Tactician 的替代品。主要区别在于 `SimpleBus` 命令处理器没有返回值。

# 总结

应用服务代表了你的边界上下文中的应用层。这些高级用例应该是相对简单和精简的，因为它们的目的主要围绕领域协调。应用服务是领域逻辑交互的入口点。我们注意到请求和命令有助于保持事物的组织性；DTOs（数据传输对象）和数据转换器使我们能够将数据表示与领域概念解耦；使用依赖注入容器构建应用服务相当直接；并且我们在复杂布局中组合应用服务有许多选择。
