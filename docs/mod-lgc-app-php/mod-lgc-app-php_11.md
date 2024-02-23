# 第十一章：将动作逻辑提取到控制器中

到目前为止，我们已经提取了我们的模型领域逻辑和视图呈现逻辑。我们的页面脚本中只剩下两种逻辑：

+   使用应用程序设置创建对象的依赖逻辑

+   使用这些对象执行页面动作的动作逻辑（有时称为业务逻辑）

在本章中，我们将从我们的页面脚本中提取出一层`Controller`类。这些类将单独处理我们遗留应用程序中的剩余动作逻辑，与我们的依赖创建逻辑分开。

# 嵌入式动作逻辑

作为嵌入式动作逻辑与依赖逻辑混合的示例，我们可以查看上一章末尾的示例代码，在附录 G 中可以找到，*响应视图文件后的代码*。在其中，我们做了一些设置工作，然后检查一些条件并调用我们领域`Transactions`的不同部分，最后我们组合了一个`Response`对象来将我们的响应发送给客户端。

与混合呈现逻辑的问题一样，我们无法单独测试动作逻辑，而无法轻松更改依赖创建逻辑以使页面脚本更易于测试。

我们解决了嵌入式动作逻辑的问题，就像解决嵌入式呈现逻辑一样。我们必须将动作代码提取到自己的类中，以将页面脚本的各种剩余关注点分开。这也将使我们能够独立于应用程序的其余部分测试动作逻辑。

## 提取过程

现在，从我们的页面脚本中提取动作逻辑应该对我们来说是一个相对容易的任务。因为领域层已经被提取出来，以及呈现层，动作逻辑应该是显而易见的。工作本身仍然需要注意细节，因为主要问题将是从动作逻辑本身中分离出依赖设置部分。

一般来说，流程如下：

1.  找到一个页面脚本，其中动作逻辑仍然与其余代码混合在一起。

1.  在该页面脚本中，重新排列代码，使所有动作逻辑位于其自己的中心块中。抽查重新排列的代码，确保它仍然正常工作。

1.  将动作逻辑的中心块提取到一个新的`Controller`类中，并修改页面脚本以使用新的`Controller`。使用*Controller*对页面脚本进行抽查。

1.  为新的`Controller`类编写单元测试，并再次进行抽查。

1.  提交新代码和测试，将它们推送到共享存储库，并通知质量保证团队。

1.  查找另一个包含嵌入式动作逻辑的页面脚本，并重新开始；当所有页面脚本都使用`Controller`对象时，我们就完成了。

## 搜索嵌入式动作逻辑

此时，我们应该能够找到动作逻辑，而无需使用项目范围的搜索功能。我们遗留应用程序中的每个页面脚本可能都至少有一点动作逻辑。

## 重新排列页面脚本并进行抽查

当我们有一个候选页面脚本时，我们继续重新排列代码，使所有设置和依赖创建工作位于顶部，所有动作逻辑位于中间，`$response->send()`调用位于底部。在这里，我们将使用上一章末尾的代码作为起始示例，该代码可以在附录 G 中找到，*响应视图文件后的代码*。

### 识别代码块

首先，我们转到脚本的顶部，在第一行（或者在包含设置脚本之后）放置一个`/* 依赖 */`注释。然后我们转到脚本的最末尾，到`$response->send()`行，并在其上方放置一个`/* 完成 */`注释。

现在我们达到了一个必须使用我们的专业判断的时刻。在页面脚本中设置和依赖工作之后的某一行，我们会发现代码开始执行某种动作逻辑。我们对这个转变发生的确切位置的评估可能有些随意，因为动作逻辑和设置逻辑很可能仍然交织在一起。即便如此，我们必须选择一个我们认为动作逻辑真正开始的时间点，并在那里放置一个`/* 控制器 */`注释。

### 将代码移动到相关块

一旦我们在页面脚本中确定了这三个块，我们就开始重新排列代码，以便只有设置和依赖创建工作发生在`/* 依赖 */`和`/* 控制器 */`之间，只有动作逻辑发生在`/* 控制器 */`和`/* 完成 */`之间。

一般来说，我们应该避免在依赖块中使用条件或循环，并且避免在控制器块中创建对象。依赖块中的代码应该只创建对象，控制器块中的代码应该只操作在依赖块中创建的对象。

鉴于我们在附录 G 中的起始代码，*响应视图文件后的代码*，我们可以在附录 H 中看到一个示例重新排列的结果，*控制器重新排列后的代码*。值得注意的是，我们将`$user_id`声明移到了控制器块，将`Response`对象创建移到了依赖块。中央控制器块中的原始动作逻辑在其他方面保持不变。

### 抽查重新排列后的代码

最后，在重新排列页面脚本之后，我们需要抽查我们的更改，以确保一切仍然正常工作。如果我们有特征测试，我们应该运行这些测试。否则，我们应该浏览或以其他方式调用页面脚本。如果它没有正确工作，我们需要撤消并重新进行重新排列，以修复我们引入的任何错误。

当我们的抽查运行成功时，我们可能希望提交到目前为止的更改。这将给我们一个已知工作的状态，如果将来的更改出现问题，我们可以回滚到这个状态。

### 提取一个控制器类

现在我们有一个正确工作的重新排列页面脚本，我们可以将中央控制器块提取到一个独立的类中。这并不困难，但我们将分几个子步骤来确保一切顺利进行。

### 选择一个类名

在我们可以提取到一个类之前，我们需要为我们将要提取到的类选择一个名称。

对于我们的领域层类，我们选择了顶层命名空间*Domain*。因为这是一个控制器层，我们将使用顶层命名空间*Controller*。我们使用的命名空间并不像一致地为所有控制器使用相同的命名空间那样重要。就个人而言，我更喜欢*Controller*，因为它足够广泛，可以包含不同类型的控制器，比如应用控制器。

该命名空间中的类名应该反映页面脚本在 URL 层次结构中的位置，其中在路径中有目录分隔符的地方使用命名空间分隔符。这种方法可以清楚地显示原始页面脚本目录路径，并且可以在类结构中很好地组织子目录。我们还在类名后缀加上`Page`以表明它是一个页面控制器。

例如，如果页面脚本位于`/foo/bar/baz.php`，那么类名应该是`Controller\Foo\Bar\BazPage`。然后，类文件本身将被放置在我们的中央类目录下的`classes/Controller/Foo/Bar/BazPage.php`。

### 创建一个骨架类文件

一旦我们有了一个类名，我们就可以为其创建一个骨架类文件。我们添加两个空方法作为以后的占位符：`__invoke()`方法将接收页面脚本的动作逻辑，构造函数最终将接收类的依赖项。

```php
**classes/Controller/Foo/Bar/BazPage.php**
1 <?php
2 namespace Controller\Foo\Bar;
3
4 class BazPage
5 {
6 public function __construct()
7 {
8 }
9
10 public function __invoke()
11 {
12 }
13 }
14 ?>
```

### 注意

**为什么是 __invoke()?**

就我个人而言，我喜欢利用`__invoke()`魔术方法来实现这个目的，但您可能希望使用`exec()`或其他适当的术语来指示我们正在执行或以其他方式运行控制器。无论我们选择什么方法名，我们都应该保持一致使用。

## 移动动作逻辑并进行抽查

现在我们准备将动作逻辑提取到我们的新`Controller`类中。

首先，我们从页面脚本中剪切控制器块，并将其原样粘贴到`__invoke()`方法中。我们在动作逻辑的末尾添加一行`return $response`，将*Response*对象发送回调用代码。

接下来，我们回到页面脚本。在提取的动作逻辑的位置，我们创建一个新的`Controller`实例并调用其`__invoke()`方法，得到一个*Response*对象。

我们应该在所有页面脚本中`始终`使用相同的变量名来表示*Controller*对象。这里的所有示例都将使用名称`$controller`。这不是因为名称`$controller`很特别，而是因为在后面的章节中，这种一致性将非常重要。

在这一点上，我们已经成功地将动作逻辑与页面脚本解耦。然而，这种解耦基本上破坏了动作逻辑，因为*Controller*依赖于页面脚本中的变量。

考虑到这一点，我们开始进行抽查和修改循环。我们浏览或以其他方式调用页面脚本，发现特定变量对*Controller*不可用。我们将其添加到`__invoke()`方法签名中，并再次进行抽查。我们继续向`__invoke()`方法添加变量，直到*Controller*拥有所需的一切，我们的抽查运行完全成功。

### 注意

在这个过程的这一部分，最好设置`error_reporting(E_ALL)`。这样我们将得到每个动作逻辑中未初始化变量的 PHP 通知。

在附录 H 中给出了我们重新排列的页面脚本，*Controller 重排后的代码*，我们初始提取到*Controller*的结果可以在附录 I 中看到，*Controller 提取后的代码*。原来提取的动作逻辑需要四个变量：`$request`、`$response`、`$user`和`$article_transactions`。

## 将 Controller 转换为依赖注入并进行抽查。

一旦我们在`__invoke()`方法中有一个可用的动作逻辑块，我们将把方法参数转换为构造函数参数，以便*Controller*可以使用依赖注入。

首先，我们剪切`__invoke()`参数，并将它们整体粘贴到`__construct()`参数中。然后编辑类定义和`__construct()`方法以将参数保留为属性。

接下来，我们修改`__invoke()`方法，使用类属性而不是方法参数。这意味着在每个所需变量前加上`$this->`。

然后，我们回到页面脚本。我们剪切`__invoke()`调用的参数，并将它们粘贴到*Controller*的实例化中。

现在我们已经将*Controller*转换为依赖注入，我们需要再次抽查页面脚本，确保一切正常运行。如果不正常，我们需要撤销并重新进行转换，直到测试通过。

在这一点上，我们可以删除`/* DEPENDENCY */`、`/* CONTROLLER */`和`/* FINISHED */`注释。它们已经达到了它们的目的，不再需要。

鉴于附录 I 中对`__invoke()`的使用，我们可以看到在附录 J 中将*Controller*转换为依赖注入的样子。我们将*Controller*的`__invoke()`参数移到`__construct()`中，将它们保留为属性，在`__invoke()`方法体中使用新属性，并修改页面脚本以在`new`时而不是`__invoke()`时传递所需的变量。

一旦我们有一个可工作的页面脚本，我们可能希望再次提交我们的工作，以便我们有一个已知正确的状态，以便以后可以恢复。

### 编写 Controller 测试

即使我们已经测试了我们的页面脚本，我们仍需要为我们提取的*Controller*逻辑编写单元测试。当我们编写测试时，我们需要将所有所需的依赖项注入到我们的*Controller*中，最好是作为测试替身，如伪造对象或模拟对象，这样我们就可以将*Controller*与系统的其余部分隔离开来。

当我们进行断言时，它们可能应该针对从`__invoke()`方法返回的*Response*对象。我们可以使用`getView()`来确保设置了正确的视图文件，使用`getVars()`来检查要在视图中使用的变量，使用`getLastCall()`来查看最终可调用的（如果有的话）是否已经正确设置。

### 提交，推送，通知 QA

一旦我们通过了单元测试，并且我们对原始页面脚本的测试也通过了，我们就可以提交我们的新代码和测试。然后我们推送到公共存储库，并通知质量保证团队，让他们审查我们的工作。

### Do ... While

现在我们继续下一个包含嵌入式动作逻辑的页面脚本，并重新开始提取过程。当我们所有的页面脚本都使用依赖注入的*Controller*对象时，我们就完成了。

## 常见问题

### 我们可以向 Controller 方法传递参数吗？

在这些示例中，我们从`__invoke()`方法中删除了所有参数。但是，有时我们希望将参数作为最后一刻的信息传递给控制器逻辑。

一般来说，在我们的现代化过程中，我们应该避免这样做。这不是因为这是一种不好的做法，而是因为我们需要在稍后的现代化步骤中对我们的控制器调用具有非常高的一致性水平。最一致的做法是`__invoke()`根本不带参数。

如果我们需要向*Controller*传递额外的信息，我们应该通过构造函数来实现。特别是当我们要传递请求值时。

例如，而不是这样：

```php
**page_script.php**
1 <?php
2 /* DEPENDENCY */
3 // ...
4 $response = new \Mlaphp\Response('/path/to/app/views');
5 $foo_transactions = new \Domain\Foo\FooTransactions(...);
6 $controller = new \Controller\Foo(
7 $response,
8 $foo_transactions
9 );
10
11 /* CONTROLLER */
12 $response = $controller->__invoke('update', $_POST['user_id']);
13
14 /* FINISHED */
15 $response->send();
16 ?>
```

我们可以这样做：

```php
**page_script.php**
1 <?php
2 /* DEPENDENCY */
3 // ...
4 $response = new \Mlaphp\Response('/path/to/app/views');
5 $foo_transactions = new \Domain\Foo\FooTransactions(...);
6 $request = new \Mlaphp\Request($GLOBALS);
7 $controller = new \Controller\Foo(
8 $response,
9 $foo_transactions,
10 $request
11 );
12
13 /* CONTROLLER */
14 $response = $controller->__invoke();
15
16 /* FINISHED */
17 $response->send();
18 ?>
```

`__invoke()`方法体将使用`$this->request->get['item_id']`。

## 一个 Controller 可以有多个动作吗？

在这些示例中，我们的*Controller*对象执行单个动作。但是，通常情况下，页面控制器可能包含多个动作，例如插入和更新数据库记录。

我们首次提取页面脚本中的动作逻辑应该保持代码基本完整，允许使用属性而不是局部变量等。但是，一旦代码在类中，将逻辑拆分为单独的动作方法是完全合理的。然后`__invoke()`方法可以变得不过是一个选择正确动作方法的`switch`语句。如果我们这样做，我们应该确保更新我们的*Controller*测试，并继续抽查页面脚本，以确保我们的更改不会破坏任何东西。

请注意，如果我们创建额外的*Controller*动作方法，我们需要避免从我们的页面脚本中调用它们。为了在稍后的现代化步骤中需要的一致性，`__invoke()`方法应该是页面脚本在其控制器块中调用的唯一*Controller*方法。

## 如果 Controller 包含 include 调用怎么办？

不幸的是，当我们重新排列页面脚本时，我们可能会发现我们的控制器块中仍然有几个`include`调用。（为设置和依赖目的而进行的`include`调用并不是什么大问题，特别是如果它们在每个页面脚本中都是相同的。）

在控制器块中使用`include`调用是我们遗留应用开始时采用的基于包含的架构的遗留物。这是一个特别难以解决的问题。我们希望将动作逻辑封装在类中，而不是在我们`include`它们时立即执行行为的文件中。

目前，我们必须接受在页面脚本的控制器块中使用`include`调用是丑陋但必要的想法。如果需要的话，我们应该避开视线，并将它们与页面脚本中的其余控制器代码一起复制到`Controller`类中。

作为安慰，我们将在下一章解决这些嵌入的`include`调用的问题。

# 回顾和下一步

将动作逻辑提取到*Controllers*层完成了我们遗留应用的一个巨大的现代化目标。现在我们已经建立了一个完整的模型视图控制器系统：模型的领域层，视图的表示层，以及连接两者的控制器层。

我们应该对我们的现代化进展感到非常满意。每个页面脚本中剩下的代码都是其原始形式的阴影。大部分逻辑是创建带有其依赖关系的*Controller*的连接代码。剩下的逻辑在所有页面脚本中都是相同的；它调用*Controller*并发送返回的*Response*对象。

然而，我们需要处理一个重要的遗留物件。为了完成对控制器逻辑的完全提取和封装，我们需要移除在我们的*Controller*类中嵌入的任何剩余的`include`调用。