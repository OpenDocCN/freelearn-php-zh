# 第十章：将演示逻辑提取到视图文件中

在传统应用程序中的页面脚本方面，很常见看到业务逻辑与演示逻辑交织在一起。例如，页面脚本做一些设置工作，然后包含一个头部模板，调用数据库，输出结果，计算一些值，打印计算出的值，将值写回数据库，并包含一个页脚模板。

我们已经采取了一些步骤，通过提取传统应用程序的域层，来解耦这些关注点。然而，在页面脚本中对域层的调用和其他业务逻辑仍然与演示逻辑混合在一起。除其他外，这种关注点的交织使得难以测试我们传统应用程序的不同方面。

在这一章中，我们将把所有的演示逻辑分离到自己的层中，这样我们就可以单独测试它，而不受业务逻辑的影响。

# 嵌入式演示逻辑

作为嵌入式演示逻辑的示例，我们可以看一下附录 E*收集演示逻辑之前的代码*。

演示逻辑。该代码显示了一个已经重构为使用域*Transactions*的页面脚本，但仍然在其余代码中存在一些演示逻辑。

### 注意

**演示逻辑和业务逻辑之间有什么区别？**

对于我们的目的，演示逻辑包括生成发送给用户（如浏览器或移动客户端）的任何和所有代码。这不仅包括`echo`和`print`，还包括`header()`和`setcookie()`。每个都会生成某种形式的输出。另一方面，“业务逻辑”是其他所有内容。

将演示逻辑与业务逻辑解耦的关键是将它们的代码放入单独的范围中。脚本应首先执行所有业务逻辑，然后将结果传递给演示逻辑。完成后，我们将能够单独测试我们的演示逻辑，而不受业务逻辑的影响。

为了实现这种范围的分离，我们将朝着在我们的页面脚本中使用`Response`对象的方向发展。我们所有的演示逻辑将在`Response`实例内执行，而不是直接在页面脚本中执行。这样做将为我们提供我们需要的范围分离，包括 HTTP 头和 cookie 在内的所有输出生成，与页面脚本的其余部分分离开来。

### 注意

**为什么使用 Response 对象？**

通常，当我们想到演示时，我们会想到一个视图或模板系统，为我们呈现内容。然而，这些类型的系统通常不会封装将发送给用户的完整输出集。我们不仅需要输出 HTTP 主体，还需要输出 HTTP 头。此外，我们需要能够测试是否设置了正确的头部，并且内容已经生成正确。因此，在这一点上，`Response`对象比单独的视图或模板系统更合适。对于我们的`Response`对象，我们将使用[`mlaphp.com/code`](http://mlaphp.com/code)提供的类。请注意，我们将在*Response*上下文中包含文件，这意味着该对象上的方法将对在该对象“内部”运行的`include`文件可用。

## 提取过程

提取演示逻辑并不像提取域逻辑那么困难。然而，它需要仔细的注意和大量的测试。

一般来说，流程如下：

1.  找到一个包含演示逻辑混合在其余代码中的页面脚本。

1.  在那个脚本中，重新排列代码，将所有演示逻辑收集到文件中所有其他逻辑之后的一个单独的块中，然后对重新排列的代码进行抽查。

1.  将演示逻辑块提取到视图文件中，通过`Response`进行交付，并再次对脚本进行抽查，以确保脚本能够正确地与新的`Response`一起工作。

1.  对演示逻辑进行适当的转义并再次进行抽查。

1.  提交新代码，推送到公共存储库，并通知 QA。

1.  重新开始包含演示逻辑混合在其他非演示代码中的下一个页面脚本。

### 搜索嵌入式演示逻辑

一般来说，我们应该很容易找到我们遗留应用程序中的演示逻辑。在这一点上，我们应该对代码库足够熟悉，以便大致知道页面脚本生成的输出在哪里。

如果我们需要一个快速启动，我们可以使用项目范围的搜索功能来查找所有`echo`、`print`、`printf`、`header`、`setcookie`和`setrawcookie`的出现。其中一些可能出现在类方法中；我们将在以后解决这个问题。现在，我们将集中精力在页面脚本上，这些调用发生在这些调用发生的地方。

### 重新排列页面脚本并进行抽查

现在我们有了一个候选的页面脚本，我们需要重新排列代码，以便演示逻辑和其他所有内容之间有一个清晰的分界线。在这个例子中，我们将使用附录 E 中的代码，*收集之前的代码*。

首先，我们转到文件底部，并在最后一行添加一个`/* PRESENTATION */`注释。然后我们回到文件顶部。逐行和逐块地工作，将所有演示逻辑移动到文件末尾，在我们的`/* PRESENTATION */`注释之后。完成后，`/* PRESENTATION */`注释之前的部分应该只包含业务逻辑，之后的部分应该只包含演示逻辑。

鉴于我们在附录 E 中的起始代码，*收集之前的代码*，我们应该最终得到类似附录 F 中的代码，*收集之后的代码*。特别要注意的是，我们有以下内容：

+   将业务逻辑未使用的变量，如`$current_page`，移到演示块下

+   将`header.php`包含移到演示块下

+   将仅对演示变量起作用的逻辑和条件，如设置`$page_title`的`if`，移到演示块中

+   用一个`$action`变量替换`$_SERVER['PHP_SELF']`

+   用一个`$id`变量替换`$_GET['id']`

### 注意

在创建演示块时，我们应该小心遵循我们从早期章节中学到的所有课程。即使演示代码是文件中的一个块（而不是一个类），我们也应该将该块视为类方法。除其他事项外，这意味着不使用全局变量、超全局变量或`new`关键字。这将使我们在以后将演示块提取到视图文件时更容易。

现在我们已经重新排列了页面脚本，使得所有演示逻辑都集中在最后，我们需要进行抽查，以确保页面脚本仍然正常工作。通常情况下，我们通过运行我们预先存在的特性测试来做到这一点。如果没有，我们必须浏览或以其他方式调用已更改的代码。

如果页面生成的输出与以前不同，我们的重新排列在某种程度上改变了逻辑。我们需要撤消并重新进行重新排列，直到页面按照应该的方式工作。

一旦我们的抽查成功，我们可能希望提交到目前为止的更改。如果我们接下来的一系列更改出现问题，我们可以将代码恢复到这一点作为已知的工作状态。

## 提取演示到视图文件并进行抽查

现在我们有了一个带有所有演示逻辑的工作页面脚本，我们将把整个块提取到自己的文件中，然后使用`Response`来执行提取的逻辑。

### 创建一个 views/目录

首先，我们需要一个地方来放置我们传统应用程序中的视图文件。虽然我更喜欢将呈现逻辑保持在业务逻辑附近，但这种安排将给我们在以后的现代化步骤中带来麻烦。因此，我们将在我们的传统应用程序中创建一个名为`views/`的新目录，并将我们的视图文件放在那里。该目录应该与我们的`classes/`和`tests/`目录处于同一级别。例如：

```php
**/path/to/app/**
1 classes/
2 tests/
3 views/
```

#### 选择一个视图文件名称

现在我们有一个保存视图文件的地方，我们需要为即将提取的呈现逻辑选择一个文件名。视图文件应该以页面脚本命名，在`views/`下的路径应与页面脚本路径匹配。例如，如果我们从`/foo/bar/baz.php`页面脚本中提取呈现，目标视图文件应保存在`/views/foo/bar/baz.php`。

有时，除了`.php`之外，使用其他扩展名对于我们的视图文件也是有用的。我发现使用一个指示视图格式的扩展名可能会有所帮助。例如，生成 HTML 的视图可能以`.html.php`结尾，而生成 JSON 的视图可能以`.json.php`结尾。

#### 将呈现块移动到视图文件中

接下来，我们从页面脚本中剪切呈现块，并将其原样粘贴到我们的新视图文件中。

然后，在页面脚本中原始的呈现块的位置，我们在新的视图文件中创建一个`Response`对象，并用`setView()`指向我们的视图文件。我们还为以后设置了一个空的`setVars()`调用，最后调用了`send()`方法。

### 注意

我们应该*始终*在所有页面脚本中使用相同的变量名来表示*Response*对象。这里的所有示例都将使用名称`$response`。这不是因为名称`$response`很特别，而是因为这种一致性在以后的章节中将非常重要。

例如：

```php
foo/bar/baz.php
1 <?php
2 // ... business logic ...
3
4 /* PRESENTATION */
5 $response = new \Mlaphp\Response('/path/to/app/views');
6 $response->setView('foo/bar/baz.html.php');
7 $response->setVars(array());
8 $response->send();
9 ?>
```

此时，我们已成功将呈现逻辑与页面脚本解耦。我们可以删除`/* PRESENTATION */`注释。它已经达到了它的目的，不再需要。

然而，这种解耦基本上破坏了呈现逻辑，因为视图文件依赖于页面脚本中的变量。考虑到这一点，我们开始进行抽查和修改周期。我们浏览或以其他方式调用页面脚本，并发现特定变量对于呈现不可用。我们将其添加到`setVars()`数组中，并再次进行抽查。我们继续向`setVars()`数组添加变量，直到视图文件拥有所需的一切，我们的抽查运行变得完全成功。

### 注意

在这个过程的这一部分，最好设置`error_reporting(E_ALL)`。这样我们将得到每个未初始化变量在呈现逻辑中的 PHP 通知。

鉴于我们之前在附录 E 中的示例，*收集之前的代码*和附录 F 中的示例，*收集之后的代码*，我们最终到达附录 G，*响应视图文件之后的代码*。我们可以看到`articles.html.php`视图文件需要四个变量：`$id, $failure`, `$input`, 和 `$action`：

```php
1 <?php
2 // ...
3 $response->setVars(array(
4 'id' => $id,
5 'failure' => $article_transactions->getFailure(),
6 'input' => $article_transactions->getInput(),
7 'action' => $_SERVER['PHP_SELF'],
8 ));
9 // ...
10 ?>
```

一旦我们有一个工作的页面脚本，我们可能希望再次提交我们的工作，以便以后如果需要，我们有一个已知正确的状态可以回滚。

### 添加适当的转义

不幸的是，大多数传统应用程序很少或根本不关注输出安全性。最常见的漏洞之一是**跨站脚本**（**XSS**）。

### 注意

什么是 XSS？

跨站脚本攻击是一种可能是由用户输入导致的攻击。例如，攻击者可以在表单输入或 HTTP 标头中输入恶意构造的 JavaScript 代码。如果该值然后在未经逃逸的情况下传递回浏览器，浏览器将执行该 JavaScript 代码。这有可能使客户端浏览器暴露于进一步的攻击。有关更多信息，请参阅*OWASP 关于 XSS 的条目* ([`www.owasp.org/index.php/Cross-site_Scripting_%28XSS%29`](https://www.owasp.org/index.php/Cross-site_Scripting_%28XSS%29))。

防御 XSS 的方法是始终为使用的上下文逃逸所有变量。如果一个变量用作 HTML 内容，它需要作为 HTML 内容进行逃逸；如果一个变量用作 HTML 属性，它需要逃逸为 HTML 属性，依此类推。

防御 XSS 需要开发人员的勤奋。如果我们记住逃逸输出的一件事，那就应该是`htmlspecialchars()`函数。适当使用此函数将使我们免受大多数 XSS 攻击的侵害。

使用`htmlspecialchars()`时，我们必须确保每次传递引号常量和字符集。因此，仅调用`htmlspecialchars($unescaped_text)`是不够的。我们必须调用`htmlspecialchars($unescaped_text, ENT_QUOTES, 'UTF-8')`。因此，输出看起来像这样：

```php
**unescaped.html.php**
1 <form action="<?php
2 echo $request->server['PHP_SELF'];
3 ?>" method="POST">
```

这需要像这样进行逃逸：

```php
**escaped.html.php**
1 <form action="<?php
2 echo htmlspecialchars(
3 $request->server['PHP_SELF'],
4 ENT_QUOTES,
5 'UTF-8'
6 );
7 ?>" method="POST">
```

每当我们发送未经逃逸的输出时，我们需要意识到我们很可能会打开一个安全漏洞。因此，我们必须对我们用于输出的每个变量应用逃逸。

以这种方式重复调用`htmlspecialchars()`可能很麻烦，因此`Response`类提供了一个`esc()`方法，作为`htmlspecialchars()`的别名，并带有合理的设置：

```php
**escaped.php**
1 <form action="<?php
2 echo $this->esc($request->server['PHP_SELF']);
3 ?>" method="POST">
```

请注意，通过`htmlspecialchars()`进行逃逸只是一个起点。虽然逃逸本身很简单，但很难知道特定上下文的适当逃逸技术。

很遗憾，本书的范围不包括提供逃逸和其他安全技术的全面概述。有关更多信息以及一个很好的独立逃逸工具，请参阅*Zend\Escaper* ([`framework.zend.com/manual/2.2/en/modules/zend.escaper`](https://framework.zend.com/manual/2.2/en/modules/zend.escaper)) 库。

在我们逃逸`Response`视图文件中的所有输出之后，我们可以继续进行测试。

## 编写视图文件测试

为视图文件编写测试提出了一些独特的挑战。在本章之前，我们所有的测试都是针对类和类方法的。因为我们的视图文件是*文件*，所以我们需要将它们放入稍微不同的测试结构中。

### tests/views/目录

首先，我们需要在我们的`tests/`目录中创建一个`views/`子目录。之后，我们的`tests/`目录应该看起来像这样：

```php
**/path/to/app/tests/**
1 bootstrap.php
2 classes/
3 phpunit.xml
4 views/
```

接下来，我们需要修改`phpunit.xml`文件，以便它知道要扫描新的`views/`子目录进行测试：

```php
**tests/phpunit.xml**
1 <phpunit bootstrap="./bootstrap.php">
2 <testsuites>
3 <testsuite>
4 <directory>./classes</directory>
5 <directory>./views</directory>
6 </testsuite>
7 </testsuites>
8 </phpunit>
```

#### 编写视图文件测试

现在我们有了视图文件测试的位置，我们需要编写一个。

尽管我们正在测试一个文件，但是 PHPUnit 要求每个测试都是一个类。因此，我们将为正在测试的视图文件命名我们的测试，并将其放在`tests/views/`目录下，该目录模仿原始视图文件的位置。例如，如果我们有一个视图文件位于`views/foo/bar/baz.html.php`，我们将在`tests/views/foo/bar/`创建一个测试文件`BazHtmlTest.php`。是的，这有点丑陋，但这将帮助我们跟踪哪些测试与哪些视图相对应。

在我们的测试类中，我们将创建一个`Response`实例，就像我们页面脚本末尾的那个一样。我们将传递视图文件路径和所需的变量。最后，我们将要求视图，然后检查输出和标头，以查看视图是否正常工作。

考虑到我们的`articles.html.php`文件，我们的初始测试可能如下所示：

```php
**tests/views/ArticlesHtmlTest.php**
1 <?php
2 class ArticlesHtmlTest extends \PHPUnit_Framework_TestCase
3 {
4 protected $response;
5 protected $output;
6
7 public function setUp()
8 {
9 $this->response = new \Mlaphp\Response('/path/to/app/views');
10 $this->response->setView('articles.html.php');
11 $this->response->setVars(
12 'id' => '123',
13 'failure' => array(),
14 'action' => '/articles.php',
15 'input' => array(
16 'title' => 'Article Title',
17 'body' => 'The body text of the article.',
18 'max_ratings' => 5,
19 'credits_per_rating' => 1,
20 'notes' => '...',
21 'ready' => 0,
22 ),
23 );
24 $this->output = $this->response->requireView();
25 }
26
27 public function testBasicView()
28 {
29 $expect = '';
30 $this->assertSame($expect, $this->output);
31 }
32 }
33 ?>
```

### 注意

**为什么使用 requireView()而不是 send()?**

如果我们使用`send()`，`Response`将输出视图文件的结果，而不是将它们留在缓冲区供我们检查。调用`requireView()`会调用视图文件，但返回结果而不是生成输出。

当我们运行这个测试时，它会失败。我们会感到高兴，因为`$expect`的值为空，但输出应该有很多内容。这是正确的行为。（如果测试通过，可能有什么地方出错了。）

#### 断言内容的正确性

现在我们需要我们的测试来查看输出是否正确。

最简单的方法是转储实际的`$this->output`字符串，并将其值复制到`$expect`变量中。如果输出字符串相对较短，使用`assertSame($expect, $this->output)`来确保它们是相同的应该完全足够。

然而，如果我们主视图文件包含的任何其他文件发生了变化，那么测试将失败。失败不是因为主视图已经改变，而是因为相关视图已经改变。这不是对我们有帮助的失败。

对于大型输出字符串，我们可以查找预期的子字符串，并确保它在实际输出中存在。然后，当测试失败时，它将与我们正在测试的特定子字符串相关，而不是整个输出字符串。

例如，我们可以使用`strpos()`来查看特定字符串是否在输出中。如果`$this->output`的大堆中不包含`$expect`针，`strpos()`将返回布尔值`false`。任何其他值都表示`$needle`存在。（如果我们编写自己的自定义断言方法，这种逻辑更容易阅读。）

```php
1 <?php
2 public function assertOutputHas($expect)
3 {
4 if (strpos($this->output, $expect) === false) {
5 $this->fail("Did not find expected output: $expect");
6 }
7 }
8
9 public function testFormTag()
10 {
11 $expect = '<form method="POST" action="/articles.php">';
12 $this->assertOutputHas($expect);
13 }
14 ?>
```

这种方法的好处是非常直接，但可能不适用于复杂的断言。我们可能希望计算元素出现的次数，或者断言 HTML 具有特定的结构而不引用该结构的内容，或者检查元素是否出现在输出的正确位置。

对于这些更复杂的内容断言，PHPUnit 有一个`assertSelectEquals()`断言，以及其他相关的`assertSelect*()`方法。这些方法通过使用 CSS 选择器来检查输出的不同部分，但可能难以阅读和理解。

或者，我们可能更喜欢安装`Zend\Dom\Query`来更精细地操作 DOM 树。这个库也通过使用 CSS 选择器来拆分内容。它返回`DOM`节点和节点列表，这使得它非常适用于以细粒度的方式测试内容。

不幸的是，我无法就哪种方法对您最好给出具体建议。我建议从上面的`assertOutputHas()`方法类似的方法开始，当明显需要更强大的系统时，再转向`Zend\Dom\Query`方法。

在我们编写了确认演示工作正常的测试之后，我们继续进行流程的最后一部分。

### 提交，推送，通知 QA

在这一点上，我们应该对页面脚本和提取的演示逻辑进行了测试。现在我们提交所有的代码和测试，将它们推送到公共存储库，并通知 QA 我们已经准备好让他们审查新的工作。

### Do ... While

我们继续在页面脚本中寻找混合业务逻辑和演示逻辑。当我们通过`Response`对象将所有演示逻辑提取到视图文件中时，我们就完成了。

## 常见问题

### 关于头部和 Cookies 呢？

在上面的例子中，我们只关注了`echo`和`print`的输出。然而，通常情况下，页面脚本还会通过`header()`、`setcookie()`和`setrawcookie()`设置 HTTP 头部。这些也会生成输出。

处理这些输出方法可能会有问题。`Response`类使用`输出缓冲`将`echo`和`print`捕获到返回值中，但对于`header()`和相关函数的调用，没有类似的选项。因为这些函数的输出没有被缓冲，我们无法轻松地测试看到发生了什么。

这是一个`Response`对象真正帮助我们的地方。该类带有缓冲`header()`和相关本机 PHP 函数的方法，但直到`send()`时才调用这些函数。这使我们能够捕获这些调用的输入并在它们实际激活之前进行测试。

例如，假设我们在一个虚构的视图文件中有这样的代码：

```php
**foo.json.php**
1 <?php
2 header('Content-Type: application/json');
3 setcookie('baz', 'dib');
4 setrawcookie('zim', 'gir');
5 echo json_encode($data);
6 ?>
```

除其他事项外，我们无法测试头部是否符合预期。PHP 已经将它们发送给客户端。

在使用*Response*对象的视图文件时，我们可以使用`$this->`前缀来调用*Response*方法，而不是本机 PHP 函数。*Response*方法缓冲本机调用的参数，而不是直接进行调用。这使我们能够在它们作为输出之前检查参数。

```php
**foo.json.php**
1 <?php
2 $this->header('Content-Type: application/json');
3 $this->setcookie('baz', 'dib');
4 $this->setrawcookie('zim', 'gir');
5 echo json_encode($data);
6 ?>
```

### 注意

因为视图文件是在*Response*实例内执行的，所以它可以访问`$this`来获取`Response`属性和方法。`Response`对象上的`header()`、`setcookie()`和`setrawcookie()`方法具有与本机 PHP 方法完全相同的签名，但是它们将输入捕获到属性中以便稍后输出，而不是立即生成输出。

现在我们可以测试`Response`对象来检查 HTTP 正文以及 HTTP 头部。

```php
**tests/views/FooJsonTest.php**
1 <?php
2 public function test()
3 {
4 // set up the response object
5 $response = new \Mlaphp\Response('/path/to/app/views');
6 $response->setView('foo.json.php');
7 $response->setVars('data', array('foo' => 'bar'));
8
9 // invoke the view file and test its output
10 $expect_body = '{"foo":"bar"}';
11 $actual_body = $response->requireView();
12 $this->assertSame($expect_output, $actual_output);
13
14 // test the buffered HTTP header calls
15 $expect_headers = array(
16 array('header', 'Content-Type: application/json'),
17 array('setcookie', 'baz', 'dib'),
18 array('setrawcookie', 'zim', 'gir'),
19 );
20 $actual_headers = $response->getHeaders();
21 $this->assertSame($expect_output, $actual_output);
22 }
23 ?>
```

### 注意

*Response*的`getHeaders()`方法返回一个子数组的数组。每个子数组都有一个元素 0，表示要调用的本机 PHP 函数名称，其余元素是函数的参数。这些是将在`send()`时调用的函数调用。

## 如果我们已经有一个模板系统呢？

许多时候，遗留应用程序已经有一个视图或模板系统。如果是这样，保持使用现有的模板系统可能就足够了，而不是引入新的`Response`类。

如果我们决定保留现有的模板系统，则本章的其他步骤仍然适用。我们需要将所有模板调用移动到页面脚本末尾的一个位置，将所有模板交互与其他业务逻辑分离。然后我们可以在页面脚本末尾显示模板。例如：

```php
**foo.php**
1 <?php
2 // ... business logic ...
3
4 /* PRESENTATION */
5 $template = new Template;
6 $template->assign($this->getVars());
7 $template->display('foo.tpl.php');
8 ?>
```

如果我们不发送 HTTP 头部，这种方法与使用`Response`对象一样具有可测试性。然而，如果我们混合调用`header()`和相关函数，我们的可测试性将更受限制。

为了未来保护我们的遗留代码，我们可以将模板逻辑移到视图文件中，并在页面脚本中与`Response`对象交互。例如：

```php
**foo.php**
1 <?php
2 // ... business logic ...
3
4 /* PRESENTATION */
5 $response = new Response('/path/to/app/views');
6 $response->setView('foo.html.php');
7 $response->setVars(array('foo' => $foo));
8 $response->send();
9 ?>
```

```php
**foo.html.php**
1 <?php
2 // buffer calls to HTTP headers
3 $this->setcookie('foo', 'bar');
4 $this->setrawcookie('baz', 'dib');
5
6 // set up the template object with Response vars
7 $template = new Template;
8 $template->assign($this->getVars());
9
10 // display the template
11 $template->display('foo.tpl.php');
12 ?>
```

这使我们能够继续使用现有的模板逻辑和文件，同时通过`Response`对象为 HTTP 头部添加可测试性。

为了保持一致，我们应该使用现有的模板系统或者通过`Response`对象在视图文件中包装所有模板逻辑。我们不应该在一些页面脚本中使用模板系统，在其他页面脚本中使用`Response`对象。在后面的章节中，我们在页面脚本中与呈现层交互的方式将变得很重要。

### 流式内容怎么办？

大多数情况下，我们的呈现内容足够小，可以由 PHP 缓冲到内存中，直到准备发送。然而，有时我们的遗留应用程序可能需要发送大量数据，比如几十或几百兆字节的文件。

将大文件读入内存，以便我们可以将其输出给用户通常不是一个好的方法。相反，我们流式传输文件：我们读取文件的一小部分并将其发送给用户，然后读取下一小部分并将其发送给用户，依此类推，直到整个文件被传送。这样，我们就不必将整个文件保存在内存中。

到目前为止，示例只处理了将视图缓冲到内存中，然后一次性输出，而不是流式传输。对于视图文件来说，将整个资源读入内存然后输出是一个不好的方法。与此同时，我们需要确保在任何流式内容之前传送标头。

`Response`对象有一个处理这种情况的方法。`Response`方法`setLastCall()`允许我们设置一个用户定义的函数（可调用的），以在需要视图文件并发送标头后调用。有了这个，我们可以传递一个类方法来为我们流式传输资源。

例如，假设我们需要流式传输一个大图像文件。我们可以编写一个类来处理流逻辑，如下所示：

```php
**classes/FileStreamer.php**
1 <?php
2 class FileStreamer
3 {
4 public function send($file, $dest = STDOUT)
5 {
6 $fh = fopen($file, 'rb');
7 while (! feof($fh)) {
8 $data = fread($fh, 8192);
9 fwrite($dest, $data);
10 }
11 fclose($fh);
12 }
13 }
14 ?>
```

这里还有很多需要改进的地方，比如错误检查和更好的资源处理，但它完成了我们示例的目的。

我们可以在页面脚本中创建一个*FileStreamer*的实例，视图文件可以将其用作`setLastCall()`的可调用参数：

```php
**foo.php**
1 <?php
2 // ... business logic ...
3 $file_streamer = new FileStreamer;
4 $image_file = '/path/to/picture.tiff';
5 $content_type = 'image/tiff';
6
7 /* PRESENTATION */
8 $response = new Response('/path/to/app/views');
9 $response->setView('foo.stream.php');
10 $response->setVars(array(
11 'streamer' => $file_streamer,
12 'file' => $image_file,
13 'type' => $content_type,
14 ));
15 ?>
```

```php
**views/foo.stream.php**
1 <?php
2 $this->header("Content-Type: {$type}");
3 $this->setLastCall(array($streamer, 'send'), $file);
4 ?>
```

在`send()`时，`Response`将需要视图文件，设置一个标头和最后一个调用的参数。然后，`Response`发送标头和视图的捕获输出（在这种情况下是空的）。最后，它调用`setLastCall()`中的可调用和参数，流式传输文件。

## 如果我们有很多演示变量怎么办？

在本章的示例代码中，我们只有少数变量需要传递给演示逻辑。不幸的是，更有可能的情况是需要传递 10 个、20 个或更多的变量。这通常是因为演示由几个`include`文件组成，每个文件都需要自己的变量。

这些额外的变量通常用于诸如站点标题、导航和页脚部分之类的内容。因为我们已经将业务逻辑与演示逻辑解耦，并在一个单独的范围内执行演示逻辑，所以我们必须传递所有`include`文件所需的变量。

比如说我们有一个视图文件，其中包括一个`header.php`文件，就像这样：

```php
**header.php**
1 <html>
2 <head>
3 <title><?php
4 echo $this->esc($page_title);
5 ?></title>
6 <link rel="stylesheet" href="<?php
7 echo $this->esc($page_style);
8 ?>"></link>
9 </head>
10 <body>
11 <h1><?php echo $this->esc($page_title); ?></h1>
12 <div id="navigation">
13 <ul>
14 <?php foreach ($site_nav as $nav_item) {
Extract Presentation Logic To View Files 117
15 $href = $this->esc($nav_item['href']);
16 $name = $this->esc($nav_item['name']);
17 echo '<li><a href="' . $href
18 . '"/a>' . $name
19 . '</li>' . PHP_EOL;
20 }?>
21 </ul>
22 </div>
23 <!-- end of header.php -->
```

我们的页面脚本将不得不传递`$page_title`、`$page_style`和`$site_nav`变量，以便页眉正确显示。这是一个相对温和的情况；可能会有更多的变量。

一个解决方案是将常用变量收集到一个或多个自己的对象中。然后我们可以将这些常用对象传递给`Response`供视图文件使用。例如，特定于页眉的显示变量可以放在`HeaderDisplay`类中，然后传递给`Response`。

```php
classes/HeaderDisplay.php
1 <?php
2 class HeaderDisplay
3 {
4 public $page_title;
5 public $page_style;
6 public $site_nav;
7 }
8 ?>
```

然后我们可以修改`header.php`文件以使用*HeaderDisplay*对象，页面脚本可以传递*HeaderDisplay*的实例，而不是所有单独的与页眉相关的变量。

### 提示

一旦我们开始将相关变量收集到类中，我们将开始看到如何将演示逻辑收集到这些类的方法中，从而减少视图文件中的逻辑量。例如，我们应该很容易想象在*HeaderDisplay*类上有一个`getNav()`方法，它返回我们导航小部件的正确 HTML。

### 那么生成输出的类方法怎么办？

在本章的示例代码中，我们集中在页面脚本中的呈现逻辑。然而，可能情况是，领域类或其他支持类使用`echo`或`header()`来生成输出。因为输出生成必须限制在呈现层，我们需要找到一种方法来移除这些调用，而不破坏我们的遗留应用程序。即使是用于呈现目的的类也不应该自行生成输出。

这里的解决方案是将每个`echo`、`print`等的使用转换为`return`。然后我们可以立即输出结果，或者将结果捕获到一个变量中，稍后再输出。

例如，假设我们有一个类方法看起来像这样：

```php
1 <?php
2 public function namesAndRoles($list)
3 {
4 echo "<p>Names and roles:</p>";
5 foreach ($list as $item) {
6 echo "<dl>";
7 echo "<dt>Name</dt><dd>{$item['name']}</dd>";
8 echo "<dt>Role</dt><dd>{$item['role']}</dd>";
9 echo "</dl>";
10 }
11 }
12 ?>
```

我们可以将其转换为类似于这样的东西（并记得添加转义！）：

```php
1 <?php
2 public function namesAndRoles($list)
3 {
4 $html = "<p>Names and roles:</p>";
5 foreach ($list as $item) {
6 $name = htmlspecialchars($item['name'], ENT_QUOTES, 'UTF-8');
7 $role = htmlspecialchars($item['role'], ENT_QUOTES, 'UTF-8');
8 $html .= "<dl>";
9 $html .= "<dt>Name</dt><dd>{$name}</dd>";
10 $html .= "<dt>Role</dt><dd>{$role}</dd>";
11 $html .= "</dl>";
12 }
13 return $html;
14 }
15 ?>
```

## 业务逻辑混入呈现逻辑怎么办？

当重新排列页面脚本以将业务逻辑与呈现逻辑分开时，我们可能会发现呈现代码调用*Transactions*或其他类或资源。这是一种混合关注点的恶劣形式，因为呈现依赖于这些调用的结果。

如果被调用的代码专门用于输出，那么就没有问题；我们可以保留调用。但是，如果被调用的代码与数据库或网络连接等外部资源进行交互，那么我们就需要分离关注点。

解决方案是从呈现逻辑中提取出一组等效的业务逻辑调用，将结果捕获到一个变量中，然后将该变量传递给呈现。

举个假设的例子，以下混合代码进行数据库调用，然后在一个循环中呈现它们：

```php
1 <?php
2 /* PRESENTATION */
3 foreach ($post_transactions->fetchTopTenPosts() as $post) {
4 echo "{$post['title']} has "
5 . $comment_transactions->fetchCountForPost($post['id'])
6 . " comments.";
7 }
8 ?>
```

暂时忽略我们需要解决示例中提出的 N+1 查询问题，以及这可能更好地在*Transactions*级别解决。我们如何将呈现与数据检索分离？

在这种情况下，我们构建了一组等效的代码来捕获所需的数据，然后将该数据传递给呈现逻辑，并应用适当的转义。

```php
1 <?php
2 // ...
3 $posts = $post_transactions->fetchTopTenPosts();
4 foreach ($posts as &$post) {
5 $count = $comment_transactions->fetchCountForPost($post['id']);
6 $post['comment_count'] = $count;
7 }
8 // ...
9
10 /* PRESENTATION */
11 foreach ($posts as $post) {
12 $title = $this->esc($post['title']);
13 $comment_count = $this->esc($post['comment_count']);
14 echo "{$title} has {$comment_count} comments."
15 }
16 ?>
```

是的，我们最终会两次循环相同的数据——一次在业务逻辑中，一次在呈现逻辑中。虽然从某些方面来说，这可能被称为低效，但效率不是我们的主要目标。关注点的分离是我们的主要目标，这种方法很好地实现了这一点。

### 如果一个页面只包含呈现逻辑呢？

我们遗留应用程序中的一些页面可能主要或完全由呈现代码组成。在这些情况下，似乎我们不需要*Response*对象。

然而，即使这些页面脚本也应该转换为使用*Response*和视图文件。我们现代化过程中的后续步骤将需要一个一致的接口来处理我们的页面脚本的结果，我们的*Response*对象是确保这种一致性的方法。

# 审查和下一步

我们现在已经浏览了所有的页面脚本，并将呈现逻辑提取到一系列单独的文件中。呈现代码现在在一个完全独立于页面脚本的范围内执行。这使我们非常容易看到脚本的剩余逻辑，并独立测试呈现逻辑。

将呈现逻辑提取到自己的层中后，我们的页面脚本正在减小。它们中所剩的只是一些设置工作和准备响应所需的操作逻辑。

那么，我们的下一步是将页面脚本中剩余的操作逻辑提取到一系列控制器类中。
