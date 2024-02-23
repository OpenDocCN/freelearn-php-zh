# 第十二章。替换类中的包含

即使现在我们已经有了模型视图控制器分离，我们的类中可能仍然有许多包含调用。我们希望我们的遗留应用程序摆脱其包含导向遗产的痕迹，仅仅包含一个文件就会导致逻辑被执行。为了做到这一点，我们需要在整个类中用方法调用替换包含调用。

### 注意

在本章的目的是，我们将使用术语包含来覆盖不仅仅是`include`，还包括`require`，`include_once`和`require_once`。

# 嵌入式包含调用

假设我们提取了一些嵌入式`include`的动作逻辑到一个*Controller*方法中。代码接收一个新用户的信息，调用一个`include`来执行一些常见的验证功能，然后处理验证的成功或失败：

```php
**classes/Controller/NewUserPage.php**
1 <?php
2 public function __invoke()
3 {
4 // ...
5 $user = $this->request->post['user'];
6 include 'includes/validators/validate_new_user.php';
7 if ($user_is_valid) {
8 $this->user_transactions->addNewUser($user);
9 $this->response->setVars('success' => true);
10 } else {
11 $this->response->setVars(array(
12 'success' => false,
13 'user_messages' => $user_messages
14 ));
15 }
16
17 return $this->response;
18 }
19 ?>
```

以下是包含文件可能的示例：

```php
includes/validators/validate_new_user.php
1 <?php
2 $user_messages = array();
3 $user_is_valid = true;
4
5 if (! Validate::email($user['email'])) {
6 $user_messages[] = 'Email is not valid.';
7 $user_is_valid = false;
8 }
9
10 if (! Validate::strlen($foo['username'], 6, 8)) {
11 $user_messages[] = 'Username must be 6-8 characters long.';
12 $user_is_valid = false;
13 }
14
15 if ($user['password'] !== $user['confirm_password']) {
16 $user_messages[] = 'Passwords do not match.';
17 $user_is_valid = false;
18 }
19 ?>
```

暂时忽略验证代码的具体内容。这里的重点是`include`文件和使用它的任何代码都紧密耦合在一起。使用该文件的任何代码都必须在包含它之前初始化一个`$user`变量。使用该文件的任何代码也都期望在其范围内引入两个新变量（`$user_messages`和`$user_is_valid`）。

我们希望解耦这个逻辑，使得`include`文件中的逻辑不会侵入其使用的类方法的范围。我们通过将`include`文件的逻辑提取到一个独立的类中来实现这一点。

# 替换过程

提取包含到它们自己的类中的难度取决于我们的类文件中剩余的`include`调用的数量和复杂性。如果包含很少，并且相对简单，那么这个过程将很容易完成。如果有许多复杂的相互依赖的包含，那么这个过程将相对难以完成。

总的来说，这个过程如下：

1.  在一个类中搜索`classes/`目录中的`include`调用。

1.  对于该`include`调用，搜索整个代码库，找出包含的文件被使用的次数。

1.  如果包含的文件只被使用一次，并且只在一个类中使用：

1.  将包含文件的内容直接复制到`include`调用上。

1.  测试修改后的类，并删除包含文件。

1.  重构复制的代码，使其遵循我们现有的所有规则：没有全局变量，没有`new`，注入依赖项，返回而不是输出，没有`include`调用。

1.  如果包含的文件被使用多次：

1.  将包含文件的内容直接复制到一个新的类方法中。

1.  用新类的内联实例化和新方法的调用替换发现的`include`调用。

1.  测试替换了`include`的类，找到耦合的变量；通过引用将这些变量添加到新方法的签名中。

1.  搜索整个代码库，查找对同一文件的`include`调用，并用内联实例化和调用替换每个调用；抽查修改后的文件并测试修改后的类。

1.  删除原始的`include`文件；对整个遗留应用程序进行单元测试和抽查。

1.  为新类编写单元测试，并重构新类，使其遵循我们现有的所有规则：没有全局变量，没有超全局变量，没有`new`，注入依赖项，返回而不是输出，没有包含。 

1.  最后，在我们的每个类文件中，用依赖注入替换新类的每个内联实例化，并在此过程中进行测试。

1.  提交，推送，通知 QA。

1.  重复，直到我们的任何类中都没有`include`调用。

## 搜索包含调用

首先，就像我们在更早的章节中所做的那样，使用我们的项目范围搜索工具来查找`include`调用。在这种情况下，只在`classes/`目录中搜索以下正则表达式：

```php
**^[ \t]*(include|include_once|require|require_once)**

```

这应该给我们一个`classes/`目录中候选`include`调用的列表。

我们选择一个要处理的单个`include`文件，然后搜索整个代码库，查找同一文件的其他包含。例如，如果我们找到了这个候选`include`...

```php
1 <?php
2 require 'foo/bar/baz.php';
3 ?>
```

我们将搜索整个代码库，查找文件名为`baz.php`的`include`调用：

```php
**^[ \t]*(include|include_once|require|require_once).*baz\.php**

```

我们只搜索文件名，因为根据`include`调用的位置不同，相对目录路径可能会指向同一个文件。我们需要确定这些`include`调用中哪些引用了同一个文件。

一旦我们有了我们知道指向同一文件的`include`调用列表，我们就计算包含该文件的调用次数。如果只有一个调用，我们的工作相对简单。如果有多个调用，我们的工作就更复杂了。

## 替换单个 include 调用

如果一个文件作为`include`调用的目标仅被调用一次，删除`include`相对容易。

首先，我们复制整个`include`文件的内容。然后，我们返回到包含`include`的类中，删除`include`调用，并将整个`include`文件的内容粘贴到其位置。

接下来，我们运行该类的单元测试，以确保它仍然正常工作。如果测试失败，我们会感到高兴！我们发现了需要在继续之前纠正的错误。如果测试通过，我们同样会感到高兴，并继续前进。

现在`include`调用已经被替换，文件内容已经成功移植到类中，我们删除`include`文件。它不再需要了。

最后，我们可以返回到包含新移植代码的类文件中。我们根据迄今为止学到的所有规则进行重构：不使用全局变量或超全局变量，不在工厂之外使用`new`关键字，注入所有需要的依赖项，返回值而不是生成输出，以及（递归地）不使用`include`调用。我们一路上运行单元测试，以确保我们不会破坏任何预先存在的功能。

## 替换多个 include 调用

如果一个文件作为多个`include`调用的目标，替换它们将需要更多的工作。

## 将 include 文件复制到类方法中

首先，我们将`include`代码复制到一个独立的类方法中。为此，我们需要选择一个与包含文件目的相适应的类名。或者，我们可以根据包含文件的路径命名类，以便跟踪代码的原始来源。

至于方法名，我们再次选择与`include`代码目的相适应的内容。就个人而言，如果类只包含一个方法，我喜欢将`__invoke()`方法用于此目的。但是，如果最终有多个方法，我们需要为每个方法选择一个合理的名称。

一旦我们选择了一个类名和方法，我们就在正确的文件位置创建新的类，并将`include`代码直接复制到新的方法中。（我们暂时不删除包含文件本身。）

## 替换原始 include 调用

现在我们有了一个要处理的类，我们回到我们在搜索中发现的`include`调用，用新类的内联实例化替换它，并调用新方法。

例如，假设原始调用代码如下：

```php
**Calling Code**
1 <?php
2 // ...
3 include 'includes/validators/validate_new_user.php';
4 // ...
5 ?>
```

如果我们将`include`代码提取到`Validator\NewUserValidator`类作为其`__invoke()`方法体，我们可以用以下代码替换`include`调用：

```php
**Calling Code**
1 <?php
2 // ...
3 $validator = new \Validator\NewUserValidator;
4 $validator->__invoke();
5 // ...
6 ?>
```

### 注意

在类中使用内联实例化违反了我们关于依赖注入的规则之一。我们不希望在工厂类之外使用`new`关键字。我们在这里这样做只是为了便于重构过程。稍后，我们将用注入替换这种内联实例化。

## 通过测试发现耦合的变量

现在我们已经成功地将调用代码与`include`文件解耦，但这给我们留下了一个问题。因为调用代码内联执行了`include`代码，新提取的代码所需的变量不再可用。我们需要将新类方法所需的所有变量传递进去，并在方法完成时使其变量对调用代码可用。

为了做到这一点，我们运行调用`include`的类的单元测试。测试将向我们展示新方法需要哪些变量。然后我们可以通过引用将这些变量传递给方法。使用引用可以确保两个代码块操作的是完全相同的变量，就好像`include`仍然在内联执行一样。这最大程度地减少了我们需要对调用代码和新提取的代码进行的更改数量。

例如，假设我们已经将代码从一个`include`文件提取到了这个类和方法中：

```php
**classes/Validator/NewUserValidator.php**
1 <?php
2 namespace Validator;
3
4 class NewUserValidator
5 {
6 public function __invoke()
7 {
8 $user_messages = array();
9 $user_is_valid = true;
10
11 if (! Validate::email($user['email'])) {
12 $user_messages[] = 'Email is not valid.';
13 $user_is_valid = false;
14 }
15
16 if (! Validate::strlen($foo['username'], 6, 8)) {
17 $user_messages[] = 'Username must be 6-8 characters long.';
18 $user_is_valid = false;
19 }
20
21 if ($user['password'] !== $user['confirm_password']) {
22 $user_messages[] = 'Passwords do not match.';
23 $user_is_valid = false;
24 }
25 }
26 }
27 ?>
```

当我们测试调用这段代码的类时，测试将失败，因为新方法中的`$user`值不可用，并且调用代码中的`$user_messages`和`$user_is_valid`变量也不可用。我们为失败而欢欣鼓舞，因为它告诉我们接下来需要做什么！我们通过引用将每个缺失的变量添加到方法签名中：

```php
**classes/Validator/NewUserValidator.php**
1 <?php
2 public function __invoke(&$user, &$user_messages, &$user_is_valid)
3 ?>
```

然后我们从调用代码将变量传递给方法：

```php
**classes/Validator/NewUserValidator.php**
1 <?php
2 $validator->__invoke($user, $user_messages, $user_is_valid);
3 ?>
```

我们继续运行单元测试，直到它们全部通过，根据需要添加变量。当所有测试都通过时，我们欢呼！所有需要的变量现在在两个范围内都可用，并且代码本身将保持解耦和可测试。

### 注意

提取的代码中并非所有变量都可能被调用代码需要，反之亦然。我们应该让单元测试的失败指导我们哪些变量需要作为引用传递。

## 替换其他包括调用和测试

现在我们已经将原始调用代码与`include`文件解耦，我们需要将所有其他剩余的代码也从同一个文件中解耦。根据我们之前的搜索结果，我们去每个文件，用新类的内联实例化替换相关的`include`调用。然后我们添加一行调用新方法并传入所需的变量。

请注意，我们可能正在替换类中的代码，也可能在视图文件等非类文件中替换代码。如果我们在一个类中替换代码，我们应该运行该类的单元测试，以确保替换不会出现问题。如果我们在一个非类文件中替换代码，我们应该运行该文件的测试（如果存在的话，比如视图文件测试），否则抽查该文件是否存在测试。

## 删除 include 文件并测试

一旦我们替换了所有对该文件的`include`调用，我们就删除该文件。现在我们应该运行所有的测试和抽查整个遗留应用程序，以确保我们没有漏掉对该文件的`include`调用。如果测试或抽查失败，我们需要在继续之前解决它。

### 编写测试和重构

现在遗留应用程序的工作方式与我们将`include`代码提取到自己的类之前一样，我们为新类编写一个单元测试。

一旦我们为新类编写了一个通过的单元测试，我们根据迄今为止学到的所有规则重构该类中的代码：不使用全局变量或超全局变量，不在工厂之外使用`new`关键字，注入所有需要的依赖项，返回值而不是生成输出，以及（递归地）不使用`include`调用。我们继续运行我们的测试，以确保我们不会破坏任何已有的功能。

### 转换为依赖注入并测试

当我们新重构的类的单元测试通过时，我们继续用依赖注入替换所有内联实例化。我们只在我们的类文件中这样做；在我们的视图文件和其他非类文件中，内联实例化并不是什么大问题。

例如，我们可能在一个类中看到这样的内联实例化和调用：

```php
**classes/Controller/NewUserPage.php**
1 <?php
2 namespace Controller;
3
4 class NewUserPage
5 {
6 // ...
7
8 public function __invoke()
9 {
10 // ...
11 $user = $this->request->post['user'];
12
13 $validator = new \Validator\NewUserValidator;
14 $validator->__invoke($user, $user_messages, $u
15
16 if ($user_is_valid) {
17 $this->user_transactions->addNewUser($user
18 $this->response->setVars('success' => true
19 } else {
20 $this->response->setVars(array(
21 'success' => false,
22 'user_messages' => $user_messages
23 ));
24 }
25
26 return $this->response;
27 }
28 }
29 ?>
```

我们将`$validator`移到通过构造函数注入的属性中，并在方法中使用该属性：

```php
**classes/Controller/NewUserPage.php**
1 <?php
2 namespace Controller;
3
4 class NewUserPage
5 {
6 // ...
7
8 public function __construct(
9 \Mlaphp\Request $request,
10 \Mlaphp\Response $response,
11 \Domain\Users\UserTransactions $user_transactions,
12 \Validator\NewUserValidator $validator
13 ) {
14 $this->request = $request;
15 $this->response = $response;
16 $this->user_transactions = $user_transactions;
17 $this->validator = $validator;
18 }
19
20 public function __invoke()
21 {
22 // ...
23 $user = $this->request->post['user'];
24
25 $this->validator->__invoke($user, $user_messages, $user_is_valid);
26
27 if ($user_is_valid) {
28 $this->user_transactions->addNewUser($user);
29 $this->response->setVars('success' => true);
30 } else {
31 $this->response->setVars(array(
32 'success' => false,
33 'user_messages' => $user_messages
34 ));
35 }
36
37 return $this->response;
38 }
39 }
40 ?>
```

现在我们需要搜索代码库，并替换每个修改后的类的实例化以传递新的依赖对象。我们在进行这些操作时运行我们的测试，以确保一切继续正常运行。

### 提交，推送，通知 QA

此时，我们要么替换了单个`include`调用，要么替换了同一文件的多个`include`调用。因为我们一直在测试，现在我们可以提交我们的新代码和测试，将它们全部推送到公共存储库，并通知 QA 我们有新的工作需要他们审查。

### Do ... While

我们再次开始搜索类文件中的下一个`include`调用。当所有的`include`调用都被类方法调用替换后，我们就完成了。

## 常见问题一个类可以从多个 include 文件中接收逻辑吗？

在示例中，我们展示了`include`代码被提取到一个独立的类中。如果我们有许多相关的`include`文件，将它们收集到同一个类中，每个都有自己的方法名，可能是合理的。例如，*NewUserValidator*逻辑可能只是许多与用户相关的验证器之一。我们可以合理地想象将该类重命名为*UserValidator*，并具有诸如`validateNewUser()`、`validateExistingUser()`等方法。

## 那么在非类文件中发起的 include 调用呢？

在寻找`include`调用时，我们只在`classes/`目录中寻找原始调用。很可能还有`include`调用来自其他位置，比如`views/`。

对于我们重构的目的，我们并不特别关心`include`调用是否来自我们类外部。如果一个`include`只从非类文件中调用，我们可以放心地保留该`include`的现有状态。

我们的主要目标是从类文件中删除`include`调用，而不一定是整个遗留应用程序。此时，很可能我们类外的大多数或所有`include`调用都是呈现逻辑的一部分。

# 审查和下一步

在我们从类中提取了所有的 include 调用之后，我们最终删除了遗留架构的最后一个主要部分。我们可以加载一个类而不产生任何副作用，并且逻辑只有在调用方法时才执行。这对我们来说是一个重要的进步。

现在我们可以开始关注我们遗留应用程序的整体架构。

目前为止，整个遗留应用程序仍然位于 Web 服务器文档根目录中。用户直接浏览每个页面脚本。这意味着 URL 与文件系统耦合在一起。此外，每个页面脚本都有相当多的重复逻辑：加载设置脚本，使用依赖注入实例化控制器，调用控制器，并发送响应。

因此，我们下一个主要目标是在我们的遗留应用程序中开始使用前端控制器。前端控制器将由一些引导逻辑、路由器和调度器组成。这将使我们的应用程序与文件系统解耦，并允许我们开始完全删除我们的页面脚本。

但在这样做之前，我们需要将应用程序中的公共资源与非公共资源分开。
