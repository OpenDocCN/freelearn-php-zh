# 第八章：将 SQL 语句提取到网关

现在我们已经将所有基于类的功能移动到一个中央目录位置（并且对这些类有一个合理的测试套件），我们可以开始从我们的页面脚本中提取更多的逻辑并将该逻辑放入类中。这将有两个好处：首先，我们将能够保持应用程序的各种关注点分开；其次，我们将能够测试提取的逻辑，以便在部署到生产环境之前很容易注意到任何故障。

这些提取中的第一个将是将所有与 SQL 相关的代码移动到自己的一组类中。对于我们的目的，SQL 是任何读取和写入数据存储系统的代名词。这可能是一个无 SQL 系统，一个 CSV 文件，一个远程资源或其他任何东西。我们将在本章集中讨论 SQL 导向的数据存储，因为它们在遗留应用程序中是如此普遍，但这些原则适用于任何形式的数据存储。

# 嵌入式 SQL 语句

目前，我们的页面脚本（可能还有一些我们的类）直接与数据库交互，使用嵌入式 SQL 语句。例如，一个页面脚本可能有一些类似以下的逻辑：

```php
**page_script.php**
1 <?php
2 $db = new Db($db_host, $db_user, $db_pass);
3 $post_id = $_GET['post_id'];
4 $stm = "SELECT * FROM comments WHERE post_id = $post_id";
5 $rows = $db->query($stm);
6 foreach ($rows as $row) {
7 // output each row
8 }
9 ?>
```

使用嵌入式 SQL 字符串的问题很多。除其他事项外，我们希望：

+   在与代码的其余部分隔离的情况下测试 SQL 交互

+   减少代码库中重复的 SQL 字符串数量

+   收集相关的 SQL 命令以进行概括和重用

+   隔离并消除诸如 SQL 注入之类的安全漏洞

这些问题和更多问题使我们得出结论，我们需要将所有与 SQL 相关的代码提取到一个 SQL 层，并用对我们的 SQL 相关类方法的调用替换嵌入式 SQL 逻辑。我们将通过创建一系列“网关”类来实现这一点。这些“网关”类唯一要做的事情就是从我们的数据源获取数据，并将数据发送回去。

本章中的“网关”类在技术上更像是表数据网关。然而，您可以选择设置适合您数据源的任何类型的“网关”。

## 提取过程

一般来说，这是我们将要遵循的过程：

1.  搜索整个代码库以查找 SQL 语句。

1.  对于每个尚未在“网关”中的语句，将语句和相关逻辑移动到相关的“网关”类方法中。

1.  为新的“网关”方法编写测试。

1.  用“网关”类方法的调用替换原始文件中的语句和相关逻辑。

1.  测试、提交、推送并通知 QA。

1.  重复上述步骤，直到下一个 SQL 语句不在“网关”类之外。

## 搜索 SQL 语句

与前几章一样，在这里我们使用项目范围的搜索功能。使用类似以下的正则表达式来识别代码库中 SQL 语句关键字的位置：

搜索：

```php
**(SELECT|INSERT|UPDATE|DELETE)**

```

我们可能会发现我们的代码库还使用其他 SQL 命令。如果是这样，我们应该将它们包括在搜索表达式中。

如果代码库在 SQL 关键字的大小写方面不一致，对我们来说更容易的是代码库始终只使用一个大小写，无论是大写还是小写。然而，这在遗留代码中并不总是约定俗成的。如果我们的代码库在 SQL 关键字的大小写方面不一致，并且我们的项目范围搜索工具有不区分大小写的选项，我们应该在这次搜索中使用该选项。否则，我们需要扩展搜索项，以包括 SQL 关键字的小写（也许是混合大小写）变体。

最后，搜索结果可能会包括误报。例如，叙述文本如“选择以下选项之一”将出现在结果列表中。我们需要逐个检查结果，以确定它们是否是 SQL 语句还是仅仅是叙述文本。

### 将 SQL 移动到网关类

将 SQL 提取到“网关”的任务是细节导向的，具体情况具体分析。遗留代码库本身的结构将决定这项任务的一个或多个正确方法。

首先，提取一个普通的 SQL 语句如下似乎很简单：

```php
1 <?php
2 $stm = "SELECT * FROM comments WHERE post_id = $post_id";
3 $rows = $db->query($stm);
4 ?>
```

但事实证明，即使在这个简单的例子中，我们也需要做出很多决定：

+   我们应该如何命名`Gateway`类和方法？

+   我们应该如何处理查询的参数？

+   我们如何避免安全漏洞？

+   适当的返回值是什么？

### 命名空间和类名

为了确定我们的命名空间和类名，我们需要首先决定是按层还是按实体进行组织。

+   如果我们按照实现层进行组织，我们类的顶层命名空间可能是`Gateway`或`DataSource\Gateway`。这种命名安排将根据代码库中的操作目的来结构化类。

+   如果我们按领域实体进行组织，顶层命名空间可能是`Comments`，甚至是`Domain\Comments`。这种命名安排将根据业务逻辑领域内的目的来结构化类。

遗留的代码库很可能会决定前进的方向。如果已经有按照某种方式组织的代码，那么最好继续使用已建立的结构，而不是重新做现有的工作。我们希望避免在代码库中设置冲突或不一致的组织结构。

在这两者之间，我建议按领域实体进行组织。我认为将与特定领域实体类型相关的功能集中在其相关的命名空间内更有意义，而不是将操作实现分散在几个命名空间中。我们还可以在特定领域功能内进一步分隔实现部分，这在按层进行组织时不容易做到。

为了反映我的领域实体偏见，本章其余部分的示例将按照领域的方式进行结构化，而不是按照实现层进行结构化。

一旦我们为我们的`Gateway`类确定了一个组织原则，我们就可以很容易地找到好的类名。例如，我们在 PHP 5.3 及更高版本中与评论相关的`Gateway`可能被命名为`Domain\Comments\CommentsGateway`。如果我们使用的是 PHP 5.2 或更早版本，我们将需要避免使用正确的命名空间，并在类名中使用下划线；例如，`Domain_Comments_CommentsGateway`。

### 方法名

然而，选择适当的方法名可能会更加困难。再次，我们应该寻找现有遗留代码库中的惯例。常见的习语可能是`get()`数据，`find()`数据，`fetch()`数据，`select()`数据，或者完全不同的其他内容。

我们应该尽可能坚持任何现有的命名约定。虽然方法名本身并不重要，但命名的一致性很重要。一致性将使我们更容易查看对`Gateway`对象的调用，并理解发生了什么，而无需阅读底层方法代码，并在代码库中搜索数据访问调用。

如果我们的遗留代码库没有显示出一致的模式，那么我们就需要为新的`Gateway`方法选择一致的命名约定。因为`Gateway`类应该是一个简单的层，用于包装 SQL 调用，本章的示例将使用诸如`select`、`insert`等方法名来标识被包装的行为。

最后，方法名可能应该指示正在执行的`select()`的类型。我们是选择一个记录还是所有记录？我们是按特定标准选择？查询中还有其他考虑吗？这些和其他问题将给我们一些提示，告诉我们如何命名`Gateway`方法。

## 一个初始的 Gateway 类方法

在将逻辑提取到类方法时，我们应该小心遵循我们在之前章节中学到的关于依赖注入的所有教训。除其他事项外，这意味着：不使用全局变量，用`Request`对象替换超全局变量，不在`Factory`类之外使用`new`关键字，以及（当然）根据需要通过构造函数注入对象。

根据上述命名原则和原始的`SELECT`语句来检索评论行，我们可以构建一个类似于这样的`Gateway`：

```php
**classes/Domain/Comments/CommentsGateway.php**
1 <?php
2 namespace Domain\Comments;
3
4 class CommentsGateway
5 {
6 protected $db;
7
8 public function __construct(Db $db)
9 {
10 $this->db = $db;
11 }
12
13 public function selectAllByPostId($post_id)
14 {
15 $stm = "SELECT * FROM comments WHERE post_id = {$post_id}";
16 return $this->db->query($stm);
17 }
18 }
19 ?>
```

这实际上是原始页面脚本的逻辑的几乎完全复制。但是，它至少留下了一个主要问题：它直接在查询中使用输入参数。这使我们容易受到 SQL 注入攻击。

### 注意

**什么是 SQL 注入**

关于小鲍比表的经典 XKCD 漫画应该有助于说明问题。恶意形成的输入参数直接用于数据库查询，以更改查询，从而损害或利用数据库。

### 击败 SQL 注入

当我们创建我们的`Gateway`方法时，我们不应假设参数值是安全的。无论我们是否期望参数在每次调用时都被硬编码为常量值，或者以其他方式保证是安全的。在某个时候，有人会更改调用`Gateway`方法的代码的一部分，我们将会有安全问题。相反，我们需要将每个参数值视为不安全，并相应处理。

因此，为了击败 SQL 注入尝试，我们应该在我们的`Gateway`方法中执行每个查询的三件事中的一件（实际上，在代码库中的任何 SQL 语句中）：

1.  最好的解决方案是使用准备语句和参数绑定，而不是查询字符串插值。

1.  第二好的解决方案是在将其插入查询字符串之前，对每个参数使用数据库层的“引用和转义”机制。

1.  第三好的解决方案是在将其插入查询字符串之前转义每个输入参数。

### 提示

或者，我们可以通过将预期的数值转换为`int`或`float`来完全避免字符串的问题。

让我们首先检查第三好的解决方案，因为它更有可能已经存在于我们的遗留代码库中。我们使用数据库的`escape`功能来转义每个参数，然后在查询字符串中使用它，并为数据库适当地引用它。因此，我们可以像这样重写`selectAllByPostId()`方法，假设使用 MySQL 数据库：

```php
<?php
2 public function selectAllByPostId($post_id)
3 {
4 $post_id = "'" . $this->db->escape($post_id) . "'";
5 $stm = "SELECT * FROM comments WHERE post_id = {$post_id}";
6 return $this->db->query($stm);
7 }
8 ?>
```

对值进行转义以插入字符串是第三好的解决方案，原因有几个。主要原因是转义逻辑有时不够。像`mysql_escape_string()`函数对我们的目的来说根本不够好。甚至`mysql_real_escape_string()`方法也有一个缺陷，这将允许攻击者根据当前字符集成功进行 SQL 注入尝试。然而，这可能是底层数据库驱动程序可用的唯一选项。

第二好的解决方案是一种称为引用和转义的转义的变体。这个功能只能通过`PDO::quote()`方法使用，比转义更安全，因为它还会自动将值包装在引号中，并处理适当的字符集。这避免了仅仅转义和自己添加引号时固有的字符集不匹配问题。

一个重写的`selectAllByPostId()`方法可能看起来像这样，使用暴露`PDO::quote()`方法的`Db`对象：

```php
<?php
2 public function selectAllByPostId($post_id)
3 {
4 $post_id = $this->db->quote($post_id);
5 $stm = "SELECT * FROM comments WHERE post_id = {$post_id}";
6 return $this->db->query($stm);
7 }
8 ?>
```

当我们记得使用它时，这是一种安全的方法。当然，问题在于，如果我们向方法添加参数，可能会忘记引用它，然后我们又容易受到 SQL 注入攻击。

最后，最好的解决方案：准备语句和参数绑定。这些只能通过 PDO（几乎适用于所有数据库）和`mysqli`扩展使用。每个都有自己的处理语句准备的变体。我们将在这里使用`PDO`样式的示例。

我们使用命名占位符而不是将值插入查询字符串，以指示参数应放置在查询字符串中的位置。然后，我们告诉`PDO`将字符串准备为`PDOStatement`对象，并在通过准备的语句执行查询时将值绑定到命名占位符。`PDO`自动使用参数值的安全表示，使我们免受 SQL 注入攻击。

以下是使用公开`PDO`语句准备逻辑和执行的`Db`对象进行重写的示例：

```php
1 <?php
2 public function selectAllByPostId($post_id)
3 {
4 $stm = "SELECT * FROM comments WHERE post_id = :post_id";
5 $bind = array('post_id' => $post_id);
6
7 $sth = $this->db->prepare($stm);
8 $sth->execute($bind);
9 return $sth->fetchAll(PDO::FETCH_ASSOC);
10 }
11 ?>
```

这里的巨大好处是我们从不在查询字符串中使用参数变量。我们总是只使用命名占位符，并将占位符绑定到准备好的语句中的参数值。这种习惯用法使我们清楚地知道何时不正确地使用了插入的变量，而且`PDO`会自动投诉如果有额外或缺少的绑定值，因此意外进行不安全的更改的机会大大减少了。

### 编写一个测试

现在是时候为我们的新类方法编写测试了。我们此时编写的测试可能不够完美，因为我们需要与数据库交互。然而，一个不完美的测试总比没有测试好。正如《测试之道》所告诉我们的，我们在能够的时候编写测试。

我们的新`Gateway`方法的测试可能看起来像这样：

```php
**tests/classes/Domain/Comments/CommentsGatewayTest.php**
1 <?php
2 namespace Domain\Comments;
3
4 use Db;
5
6 class CommentsGatewayTest
7 {
8 protected $db;
9
10 protected $gateway;
11
12 public function setUp()
13 {
14 $this->db = new Db('test_host', 'test_user', 'test_pass');
15 $this->gateway = new CommentsGateway($this->db);
16 }
17
18 public function testSelectAllByPostId()
19 {
20 // a range of known IDs in the comments table
21 $post_id = mt_rand(1,100);
22
23 // get the comment rows
24 $rows = $this->gateway->selectAllByPostId($post_id);
25
26 // make sure they all match the post_id
27 foreach ($rows as $row) {
28 $this->assertEquals($post_id, $row['post_id']);
29 }
30 }
31 }
32 ?>
```

现在我们运行我们的测试套件，看看测试是否通过。如果通过，我们会庆祝并继续前进！如果没有通过，我们将继续完善`Gateway`方法和相关测试，直到两者都正常工作。

### 提示

**完善我们的测试**

正如前面所述，这是一个非常不完美的测试。除其他事项外，它取决于一个可用的数据库连接，并且首先需要在数据库中种子数据。通过依赖数据库，我们依赖它处于正确的状态。如果数据库中没有正确的数据，那么测试将失败。失败不是来自我们正在测试的代码，而是来自大部分超出我们控制的数据库。改进测试的一个机会是将`Gateway`类更改为依赖于`DbInterface`而不是具体的`Db`类。然后，我们将为测试目的创建一个实现`DbInterface`的`FakeDb`类，并将一个`FakeDb`实例注入到`Gateway`中，而不是一个真正的`Db`实例。这样做将使我们更深入地了解 SQL 查询字符串的正确性，以及对返回给`Gateway`的数据具有更大的控制。最重要的是，它将使测试与对可用数据库的依赖解耦。目前，出于迅速进行的考虑，我们将使用不完美的测试。

### 替换原始代码

现在我们有一个可工作且经过测试的`Gateway`方法，我们用调用`Gateway`方法替换原始代码。而旧代码看起来像这样：

```php
**page_script.php (before)**
1 <?php
2 $db = new Db($db_host, $db_user, $db_pass);
3 $post_id = $_GET['post_id'];
4 $stm = "SELECT * FROM comments WHERE post_id = $post_id";
5 $rows = $db->query($stm);
6 foreach ($rows as $row) {
7 // output each row
8 }
9 ?>
```

新版本将如下所示：

```php
**page_script.php (after)**
1 <?php
2 $db = new Db($db_host, $db_user, $db_pass);
3 $comments_gateway = new CommentsGateway($db);
4 $rows = $comments_gateway->selectAllByPostId($_GET['post_id']);
5 foreach ($rows as $row) {\
6 // output each row
7 }
8 ?>
```

请注意，我们几乎没有修改操作逻辑。例如，我们没有添加以前不存在的错误检查。我们修改的最远程度是通过准备好的语句来保护查询免受 SQL 注入。

### 测试，提交，推送，通知 QA

与之前的章节一样，现在我们需要抽查旧应用程序。虽然我们对新的`Gateway`方法有一个单元测试，但我们仍然需要抽查我们修改过的应用程序的部分。如果我们之前准备了一个表征测试来覆盖我们遗留应用程序的这一部分，我们现在可以运行它。否则，我们可以通过浏览或以其他方式调用应用程序的更改部分来进行此操作。

一旦我们确信已成功用调用我们的新`Gateway`方法替换了嵌入式 SQL，我们就将更改提交到版本控制，包括我们的测试。然后我们推送到中央仓库，并通知 QA 团队我们的更改。

### 做...直到

完成后，我们再次搜索代码库，查找 SQL 关键字以指示嵌入式查询字符串的用法。如果它们存在于`Gateway`类之外，我们将继续将查询提取到适当的`Gateway`中。一旦所有 SQL 语句都已移动到`Gateway`类中，我们就完成了。

## 常见问题

### 那么插入、更新和删除语句呢？

到目前为止，我们只看了`SELECT`语句，因为它们很可能是我们传统代码库中最常见的情况。然而，还会有大量的`INSERT`，`UPDATE`，`DELETE`，以及其他语句。在提取到`Gateway`时，这些基本上与`SELECT`相同，但也有一些细微的差异。

特别是`INSERT`和`UPDATE`语句可能包含大量的参数，指示要插入或更新的列值。将太多的参数添加到提取的`Gateway`方法签名中将使其难以处理。

在这些情况下，我们可以使用数据数组来指示列名及其对应的值。但我们需要确保只插入或更新正确的列。

例如，假设我们从页面脚本中开始，保存一个新的评论，包括评论者的姓名、评论内容、评论者的 IP 地址以及评论所附加的帖子 ID：

```php
**page_script.php**
1 <?php
2 $db = new Db($db_host, $db_user, $db_pass);
3
4 $name = $db->escape($_POST['name']);
5 $body = $db->escape($_POST['body']);
6 $post_id = (int) $_POST['id'];
7 $ip = $db->escape($_SERVER['REMOTE_ADDR']);
8
9 $stm = "INSERT INTO comments (post_id, name, body, ip) "
10 .= "VALUES ($post_id, '{$name}', '{$body}', '{$ip}'";
11
12 $db->query($stm);
13 $comment_id = $db->lastInsertId();
14 ?>
```

当我们将这些提取到`CommentsGateway`中的方法时，我们可以为每个要插入的列值设置一个参数。在这种情况下，只有四列，但如果有十几列，方法签名将更难处理。

作为每列一个参数的替代方案，我们可以将数据数组作为单个参数传递，然后在方法内部处理。这个使用数据数组的示例包括一个带有占位符的预处理语句，以防止 SQL 注入攻击：

```php
1 <?php
2 public function insert(array $bind)
3 {
4 $stm = "INSERT INTO comments (post_id, name, body, ip) "
5 .= "VALUES (:post_id, :name, :body, :ip)";
6 $this->db->query($stm, $bind);
7 return $this->db->lastInsertId();
8 }
9 ?>
```

一旦我们在`CommentsGateway`中有了这样的方法，我们可以修改原始代码，使其更像下面这样：

```php
**page_script.php**
1 <?php
2 $db = new Db($db_host, $db_user, $db_pass);
3 $comments_gateway = new CommentsGateway($db);
4
5 $input = array(
6 'name' => $_POST['name'],
7 'body' => $_POST['body'],
8 'post_id' => $_POST['id'],
9 'ip' => $_SERVER['REMOTE_ADDR'],
10 );
11
12 $comment_id = $comments_gateway->insert($input);
13 ?>
```

### 重复的 SQL 字符串怎么办？

在这个过程中，我们可能会遇到的一件事是，在我们的传统应用程序中，查询字符串中存在大量的重复，或者是带有变化的重复。

例如，我们可能会在传统应用程序的其他地方找到一个类似于这样的与评论相关的查询：

```php
1 <?php
2 $stm = "SELECT * FROM comments WHERE post_id = $post_id LIMIT 10";
3 ?>
```

查询字符串与本章开头的示例代码相同，只是附加了一个`LIMIT`子句。我们应该为这个查询创建一个全新的方法，还是修改现有的方法？

这是需要专业判断和对代码库的熟悉。在这种情况下，修改似乎是合理的，但在其他情况下，差异可能足够大，需要创建一个全新的方法。

如果我们选择修改`CommentsGateway`中的现有方法，我们可以重写`selectAllByPostId()`以包括一个可选的`LIMIT`：

```php
1 <?php
2 public function selectAllByPostId($post_id, $limit = null)
3 {
4 $stm = "SELECT * FROM comments WHERE post_id = :post_id";
5 if ($limit) {
6 $stm .= " LIMIT " . (int) $limit;
7 }
8 $bind = array('post_id' => $post_id);
9 return $this->db->query($stm, $bind);
10 }
11 ?>
```

现在我们已经修改了应用程序类，我们需要运行现有的测试。如果测试失败，那我们就庆幸！我们发现了我们的改变有缺陷，而测试阻止了这个 bug 进入生产。如果测试通过，我们也庆幸，因为事情仍然像改变之前一样工作。

最后，在现有测试通过后，我们修改`CommentsGatewayTest`，以检查新的`LIMIT`功能是否正常工作。这个测试仍然不完美，但它传达了要点。

```php
tests/classes/Domain/Comments/CommentsGatewayTest.php
1 <?php
2 public function testSelectAllByPostId()
3 {
4 // a range of known IDs in the comments table
5 $post_id = mt_rand(1,100);
6
7 // get the comment rows
8 $rows = $this->gateway->selectAllByPostId($post_id);
9
10 // make sure they all match the post_id
11 foreach ($rows as $row) {
12 $this->assertEquals($post_id, $row['post_id']);
13 }
14
15 // test with a limit
16 $limit = 10;
17 $rows = $this->gateway->selectAllByPostId($post_id, $limit);
18 $this->assertTrue(count($rows) <= $limit);
19 }
20 }
21 ?>
```

我们再次运行测试，以确保我们的新的`LIMIT`功能正常工作，并不断完善代码和测试，直到通过为止。

然后我们继续用`Gateway`的调用替换原始的嵌入式 SQL 代码，进行抽查，提交等等。

### 注意

我们需要谨慎处理。在看到一个查询的变体之后，我们将能够想象出许多其他可能的查询变体。由此产生的诱惑是在实际遇到这些变体之前，就预先修改我们的`Gateway`方法来适应想象中的变体。除非我们实际在遗留代码库中看到了特定的变体，否则我们应该克制自己，不要为那种变体编写代码。我们不希望超前于代码库当前实际需要的情况。目标是在可见的路径上小步改进，而不是在想象的迷雾中大步跨越。

### 复杂的查询字符串怎么办？

到目前为止，示例都是相对简单的查询字符串。这些简单的示例有助于保持流程清晰。然而，在我们的遗留代码库中，我们可能会看到非常复杂的查询。这些查询可能是由多个条件语句构建而成，使用多个不同的参数在查询中使用。以下是一个复杂查询的示例，摘自附录 A，*典型的遗留页面脚本*：

```php
1 <?php
2 // ...
3 define("SEARCHNUM", 10);
4 // ...
5 $page = ($page) ? $page : 0;
6
7 if (!empty($p) && $p!="all" && $p!="none") {
8 $where = "`foo` LIKE '%$p%'";
9 } else {
10 $where = "1";
11 }
12
13 if ($p=="hand") {
14 $where = "`foo` LIKE '%type1%'"
15 . " OR `foo` LIKE '%type2%'"
16 . " OR `foo` LIKE '%type3%'";
17 }
18
19 $where .= " AND `bar`='1'";
20 if ($s) {
21 $s = str_replace(" ", "%", $s);
22 $s = str_replace("'", "", $s);
23 $s = str_replace(";", "", $s);
24 $where .= " AND (`baz` LIKE '%$s%')";
25 $orderby = "ORDER BY `baz` ASC";
26 } elseif ($letter!="none" && $letter) {
27 $where .= " AND (`baz` LIKE '$letter%'"
28 . " OR `baz` LIKE 'The $letter%')";
29 $orderby = "ORDER BY `baz` ASC";
30 } else {
31 $orderby = "ORDER BY `item_date` DESC";
32 }
33 $query = mysql_query(
34 "SELECT * FROM `items` WHERE $where $orderby
35 LIMIT $page,".SEARCHNUM;
36 );
37 ?>
```

对于这种复杂的安排，我们需要非常注意细节，将相关的查询构建逻辑提取到我们的`Gateway`中。主要考虑因素是确定查询构建逻辑中使用了哪些变量，并将其设置为我们新的`Gateway`方法的参数。然后我们可以将查询构建逻辑移动到我们的`Gateway`中。

首先，我们可以尝试将嵌入的与 SQL 相关的逻辑提取到`Gateway`方法中：

```php
1 <?php
2 namespace Domain\Items;
3
4 class ItemsGateway
5 {
6 protected $mysql_link;
7
8 public function __construct($mysql_link)
9 {
10 $this->mysql_link = $mysql_link;
11 }
12
13 public function selectAll(
14 $p = null,
15 $s = null,
16 $letter = null,
17 $page = 0,
18 $searchnum = 10
19 ) {
20 if (!empty($p) && $p!="all" && $p!="none") {
21 $where = "`foo` LIKE '%$p%'";
22 } else {
23 $where = "1";
24 }
25
26 if ($p=="hand") {
Extract SQL Statements To Gateways 84
27 $where = "`foo` LIKE '%type1%'"
28 . " OR `foo` LIKE '%type2%'"
29 . " OR `foo` LIKE '%type3%'";
30 }
31
32 $where .= " AND `bar`='1'";
33 if ($s) {
34 $s = str_replace(" ", "%", $s);
35 $s = str_replace("'", "", $s);
36 $s = str_replace(";", "", $s);
37 $where .= " AND (`baz` LIKE '%$s%')";
38 $orderby = "ORDER BY `baz` ASC";
39 } elseif ($letter!="none" && $letter) {
40 $where .= " AND (`baz` LIKE '$letter%'"
41 . " OR `baz` LIKE 'The $letter%')";
42 $orderby = "ORDER BY `baz` ASC";
43 } else {
44 $orderby = "ORDER BY `item_date` DESC";
45 }
46
47 $stm = "SELECT *
48 FROM `items`
49 WHERE $where
50 $orderby
51 LIMIT $page, $searchnum";
52
53 return mysql_query($stm, $this->mysql_link);
54 }
55 }
56 ?>
```

### 注意

尽管我们已经删除了一些依赖项（例如对`mysql_connect()`链接标识符的隐式全局依赖），但这第一次尝试仍然存在许多问题。其中，它仍然容易受到 SQL 注入的影响。我们需要在查询中使用`mysql_real_escape_string()`对每个参数进行转义，并将`LIMIT`值转换为整数。

一旦我们完成了提取及其相关的测试，我们将把原始代码更改为以下内容：

```php
1 <?php
2 // ...
3 define("SEARCHNUM", 10);
4 // ...
5 $page = ($page) ? $page : 0;
6 $mysql_link = mysql_connect($db_host, $db_user, $db_pass);
7 $items_gateway = new \Domain\Items\ItemsGateway($mysql_link);
8 $query = $items_gateway->selectAll($p, $s, $letter, $page, SEARCHNUM);
9 ?>
```

### 非 Gateway 类内的查询怎么办？

本章的示例显示了嵌入在页面脚本中的 SQL 查询字符串。同样可能的是，我们也会在非 Gateway 类中找到嵌入的查询字符串。

在这些情况下，我们遵循与页面脚本相同的流程。一个额外的问题是，我们将不得不将`Gateway`依赖项传递给该类。例如，假设我们有一个`Foo`类，它使用`doSomething()`方法来检索评论：

```php
1 <?php
2 class Foo
3 {
4 protected $db;
5
6 public function __construct(Db $db)
7 {
8 $this->db = $db;
9 }
10
11 public function doSomething($post_id)
12 {
13 $stm = "SELECT * FROM comments WHERE post_id = $post_id";
14 $rows = $this->db->query($stm);
15 foreach ($rows as $row) {
16 // do something with each row
17 }
18 return $rows;
19 }
20 }
21 ?>
```

我们提取 SQL 查询字符串及其相关逻辑，就像我们在页面脚本中所做的那样。然后我们修改`Foo`类，将`Gateway`作为依赖项，而不是`Db`对象，并根据需要使用`Gateway`：

```php
1 <?php
2 use Domain\Comments\CommentsGateway;
3
4 class Foo
5 {
6 protected $comments_gateway;
7
8 public function __construct(CommentsGateway $comments_gateway)
9 {
10 $this->comments_gateway = $comments_gateway;
11 }
12
13 public function doSomething($post_id)
14 {
15 $rows = $this->comments_gateway->selectAllByPostId($post_id);
16 foreach ($rows as $row) {
17 // do something with each row
18 }
19 return $rows;
20 }
21 }
22 ?>
```

### 我们可以从基类 Gateway 类扩展吗？

如果我们有许多具有类似功能的`Gateway`类，将一些功能收集到`AbstractGateway`中可能是合理的。例如，如果它们都需要`Db`连接，并且都有类似的`select*()`方法，我们可以做如下操作：

```php
classes/AbstractGateway.php
1 <?php
2 abstract class AbstractGateway
3 {
4 protected $table;
5
6 protected $primary_key;
7
8 public function __construct(Db $db)
9 {
10 $this->db = $db;
11 }
12
13 public function selectOneByPrimaryKey($primary_val)
14 {
15 $stm = "SELECT * FROM {$this->table} "
16 .= "WHERE {$this->primary_key} = :primary_val";
17 $bind = array('primary_val' => $primary_val);
18 return $this->db->query($stm, $bind);
19 }
20 }
21 ?>
```

然后我们可以从基类`AbstractGateway`扩展一个类，并调整特定表的扩展属性：

```php
1 <?php
2 namespace Domain\Items;
3
4 class ItemsGateway extends \AbstractGateway
5 {
6 protected $table = 'items';
7 protected $primary_key = 'item_id';
8 }
9 ?>
```

基本的`selectOneByPrimaryKey()`方法可以与各种`Gateway`类一起使用。根据需要，我们仍然可以在特定的`Gateway`类上添加其他具体的方法。

### 注意

对于这种方法要谨慎。我们应该只抽象出已经存在于我们已经提取的行为中的功能。抵制提前创建我们在遗留代码库中实际上还没有看到的功能的诱惑。

### 多个查询和复杂的结果结构怎么办？

本章中的示例已经显示了针对单个表的单个查询。我们可能会遇到使用多个查询针对几个不同的表，然后将结果合并为复杂领域实体或集合的逻辑。以下是一个例子：

```php
1 <?php
2 // build a structure of posts with author and statistics data,
3 // with all comments on each post.
4 $page = (int) $_GET['page'];
5 $limit = 10;
6 $offset = $page * $limit; // a zero-based paging system
7 $stm = "SELECT *
8 FROM posts
9 LEFT JOIN authors ON authors.id = posts.author_id
10 LEFT JOIN stats ON stats.post_id = posts.id
11 LIMIT {$limit} OFFSET {$offset}"
12 $posts = $db->query($stm);
13
14 foreach ($posts as &$post) {
15 $stm = "SELECT * FROM comments WHERE post_id = {$post['id']}";
16 $post['comments'] = $db->query($stm);
17 }
18 ?>
```

### 注意

这个例子展示了一个经典的 N+1 问题，其中为主集合的每个成员发出一个查询。获取博客文章的第一个查询将跟随 10 个查询，每个博客文章一个，以获取评论。因此，总查询数为 10，加上初始查询为 1。对于 50 篇文章，总共将有 51 个查询。这是遗留应用程序中性能拖慢的典型原因。有关 N+1 问题的详细讨论和解决方案，请参见*Solving The N+1 Problem in PHP* ([`leanpub.com/sn1php`](https://leanpub.com/sn1php))

第一个问题是确定如何将查询拆分为`Gateway`方法。有些查询必须一起进行，而其他查询可以分开。在这种情况下，第一个和第二个查询可以分开到不同的`Gateway`类和方法中。

下一个问题是确定哪个`Gateway`类应接收提取的逻辑。当涉及多个表时，有时很难确定，因此我们必须选择查询的主要主题。上面的第一个查询涉及到文章、作者和统计数据，但从逻辑上看，我们主要关注的是文章。

因此，我们可以将第一个查询提取到`PostsGateway`中。我们希望尽可能少地修改查询本身，因此我们保留连接和其他内容不变：

```php
1 <?php
2 namespace Domain\Posts;
3
4 class PostsGateway
5 {
6 protected $db;
7
8 public function __construct(Db $db)
9 {
10 $this->db = $db;
11 }
12
13 public function selectAllWithAuthorsAndStats($limit = null, $offset = null)
14 {
15 $limit = (int) $limit;
https://leanpub.com/sn1php
16 $offset = (int) $offset;
17 $stm = "SELECT *
18 FROM posts
19 LEFT JOIN authors ON authors.id = posts.author_id
20 LEFT JOIN stats ON stats.post_id = posts.id
21 LIMIT {$limit} OFFSET {$offset}"
22 return $this->db->query($stm);
23 }
24 }
25 ?>
```

完成后，我们继续根据第一个查询编写新功能的测试。我们修改代码并进行测试，直到测试通过。

第二个查询，与评论相关的查询，与我们之前的例子相同。

在完成提取及其相关测试后，我们可以修改页面脚本，使其如下所示：

```php
1 <?php
2 $db = new Database($db_host, $db_user, $db_pass);
3 $posts_gateway = new \Domain\Posts\PostsGateway($db);
4 $comments_gateway = new \Domain\Comments\CommentsGateway($db);
5
6 // build a structure of posts with author and statistics data,
7 // with all comments on each post.
8 $page = (int) $_GET['page'];
9 $limit = 10;
10 $offset = $page * $limit; // a zero-based paging system
11 $posts = $posts_gateway->selectAllWithAuthorsAndStats($limit, $offset);
12
13 foreach ($posts as &$post) {
14 $post['comments'] = $comments_gateway->selectAllByPostId($post['id']);
15 }
16 ?>
```

### 如果没有数据库类会怎么样？

许多遗留代码库没有数据库访问层。相反，这些遗留应用程序直接在其页面脚本中使用`mysql`扩展。对`mysql`函数的调用分散在整个代码库中，并未收集到单个类中。

如果我们可以升级到`PDO`，我们应该这样做。然而，由于各种原因，可能无法从`mysql`升级。`PDO`的工作方式与`mysql`不完全相同，从`mysql`习语更改为`PDO`习语可能一次性做得太多。此时进行迁移可能会使测试变得比我们想要的更加困难。

另一方面，我们可以将`mysql`调用按原样移入我们的`Gateway`类中。起初这样做似乎是合理的。然而，`mysql`扩展内置了一些全局状态。任何需要链接标识符（即服务器连接）的`mysql`函数在没有传递链接标识符时会自动使用最近的连接资源。这与依赖注入的原则相违背，因为如果可能的话，我们宁愿不依赖全局状态。

因此，我建议我们不直接迁移到 PDO，也不将`msyql`函数调用保持原样，而是将`mysql`调用封装在一个类中，该类代理方法调用到`mysql`函数。然后，我们可以使用类方法而不是`mysql`函数。类本身可以包含链接标识符，并将其传递给每个方法调用。这将为我们提供一个数据库访问层，我们的`Gateway`对象可以使用，而不会太大地改变`mysql`的习惯用法。

这样一个包装器的一个操作示例实现是`MysqlDatabase`类。当我们创建一个`MysqlDatabase`的实例时，它会保留连接信息，但实际上不会连接到服务器。只有在我们调用实际需要服务器连接的方法时才会连接。这种延迟加载的方法有助于减少资源使用。此外，`MysqlDatabase`类明确添加了链接标识参数，这在相关的`mysql`函数中是可选的，这样我们就不会依赖于`mysql`扩展的隐式全局状态。

要用`MysqlDatabase`调用替换`mysql`函数调用：

1.  在整个代码库中搜索`mysql_`前缀的函数调用。

1.  在每个文件中，如果有带有`mysql_`函数前缀的函数调用...

+   创建或注入一个`MysqlDatabase`的实例。

+   用`MysqlDatabase`对象变量和一个箭头操作符(`->`)替换每个`mysql_`函数前缀。如果我们对风格很挑剔，我们还可以将剩余的方法名部分从`snake_case()`转换为`camelCase()`。

1.  抽查，提交，推送，并通知 QA。

1.  继续搜索`mysql_`前缀的函数调用，直到它们都被替换为`MysqlDatabase`方法调用。

例如，假设我们有这样一个遗留代码：

```php
**Using mysql functions**
1 <?php
2 mysql_connect($db_host, $db_user, $db_pass);
3 mysql_select_db('my_database');
4 $result = mysql_query('SELECT * FROM table_name LIMIT 10');
5 while ($row = mysql_fetch_assoc($result)) {
6 // do something with each row
7 }
8 ?>
```

使用上述过程，我们可以将代码转换为使用`MysqlDatabase`对象：

**使用 MysqlDatabase 类**

```php
1 <?php
2 $db = new \Mlaphp\MysqlDatabase($db_host, $db_user, $db_pass);
3 $db->select_db('my_database'); // or $db->selectDb('my_database')
4 $result = $db->query('SELECT * FROM table_name LIMIT 10');
5 while ($row = $db->fetch_assoc($result)) {
6 // do something with each row
7 }
8 ?>
```

这段代码，反过来可以使用一个注入的`MysqlDatabase`对象提取到一个`Gateway`类中。

### 注意

对于我们的页面脚本，最好在现有的设置文件中创建一个`MysqlDatabase`实例并使用它，而不是在每个页面脚本中单独创建一个。实现的延迟连接性意味着如果我们从未对数据库进行调用，就永远不会建立连接，因此我们不需要担心不必要的资源使用。现有的遗留代码库将帮助我们确定这是否是一个合理的方法。

一旦我们的`Gateway`类使用了一个注入的`MysqlDatabase`对象，我们就可以开始计划从封装的`mysql`函数迁移到具有不同习惯用法和用法的`PDO`。因为数据库访问逻辑现在由`Gateway`对象封装，所以迁移和测试将比如果我们替换了遍布整个代码库的`mysql`调用要容易。

# 审查和下一步

当我们完成了这一步，我们所有的 SQL 语句将在`Gateway`类中，而不再在我们的页面脚本或其他非`Gateway`类中。我们还将对我们的`Gateway`类进行测试。

从现在开始，每当我们需要向数据库添加新的调用时，我们只会在`Gateway`类中这样做。每当我们需要获取或保存数据时，我们将使用`Gateway`方法，而不是编写嵌入式 SQL。这使我们在数据库交互和未来的模型层和实体对象之间有了明确的关注点分离。

现在我们已经将数据库交互分离到了它们自己的层中，我们将检查整个遗留应用程序中对`Gateway`对象的所有调用。我们将检查页面脚本和其他类如何操作返回的结果，并开始提取定义我们模型层的行为。
