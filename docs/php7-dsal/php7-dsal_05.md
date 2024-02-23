# 应用递归算法 - 递归

解决复杂问题总是很困难的。即使对于程序员来说，解决复杂问题也可能更加困难，有时需要特殊的解决方案。递归是计算机程序员用来解决复杂问题的一种特殊方法。在本章中，我们将介绍递归的定义、属性、不同类型的递归以及许多示例。递归并不是一个新概念；在自然界中，我们看到许多递归元素。分形展现了递归行为。以下图像显示了自然递归：

![](img/Image00032.jpg)

# 理解递归

递归是通过将大问题分解为小问题来解决更大问题的一种方法。换句话说，递归是将大问题分解为更小的相似问题来解决它们并获得实际结果。通常，递归被称为函数调用自身。这可能听起来很奇怪，但事实是当函数递归时，函数必须调用自身。这是什么样子？让我们看一个例子，

在数学中，“阶乘”这个术语非常流行。数字*N*的阶乘被定义为小于等于*N*的所有正整���的乘积。它总是用*!*（感叹号）表示。因此，*5*的阶乘可以写成如下形式：

*5! = 5 X 4 X 3 X 2 X 1*

同样，我们可以写出给定数字的以下阶乘：

*4! = 4 X 3 X 2 X 1*

*3! = 3 X 2 X 1*

*2! = 2 X 1*

*1! = 1*

如果我们仔细观察我们的例子，我们可以看到我们可以用*4*的阶乘来表示*5*的阶乘，就像这样：

*5! = 5 X 4!*

同样，我们可以写成：

*4! = 4 X 3!*

*3! = 3 X 2!*

*2! = 2 X 1!*

*1! = 1 X 0!*

*0! = 1*

或者，我们可以简单地说一般来说：

*n! = n * (n-1)!*

这代表了递归。我们将每个步骤分解成更小的步骤，并解决实际的大问题。这里有一张图片展示了如何计算 3 的阶乘：

![](img/Image00033.jpg)

因此，步骤如下：

1.  *3! = 3 X 2!*

1.  *2! = 2 X 1!*

1.  *1! = 1 X 0!*

1.  *0! = 1*

1.  *1! = 1 X 1 = 1*

1.  *2! = 2 X 1 = 2*

1.  *3! = 3 X 2 = 6*

# 递归算法的属性

现在，问题可能是，“如果一个函数调用自身，那么它如何停止或知道何时完成递归调用？”当我们编写递归解决方案时，我们必须确保它具有以下属性：

1.  每个递归调用都应该是一个更小的子问题。就像阶乘的例子，6 的阶乘是用 6 和 5 的阶乘相乘来解决的，依此类推。

1.  它必须有一个基本情况。当达到基本情况时，将不会有进一步的递归，并且基本情况必须能够解决问题，而不需要进一步的递归调用。在我们的阶乘示例中，我们没有从 0 进一步。所以，在这种情况下，0 是我们的基本情况。

1.  不应该有任何循环。如果每个递归调用都调用同一个问题，那么将会有一个永无止境的循环。经过一些重复后，计算机将显示堆栈溢出错误。

因此，如果我们现在使用 PHP 7 编写我们的阶乘程序，那么它将如下所示：

```php
function factorial(int $n): int {

   if ($n == 0)

    return 1;

   return $n * factorial($n - 1);

}

```

在前面的示例代码中，我们可以看到我们有一个基本条件，当$n$的值为$0$时，我们返回`1`。如果不满足这个条件，那么我们返回$n$的乘积和$n-1$的阶乘。所以，它满足 1 和 3 这两个数字的属性。我们避免了循环，并确保每个递归调用都创建了一个更大的子问题。我们将像这样编写递归行为的算法：

![](img/Image00034.jpg)

# 递归与迭代算法

如果我们分析我们的阶乘函数，我们可以看到它可以使用简单的迭代方法来编写，使用`for`或`while`循环，如下所示：

```php
function factorial(int $n): int { 

    $result = 1; 

    for ($i = $n; $i > 0; $i--) {

      $result *= $i; 

    } 

    return $result; 

}

```

如果这可以写成一个简单的迭代形式，那么为什么要使用递归呢？递归用于解决更复杂的问题。并非所有问题都可以如此轻松地迭代解决。例如，我们需要显示某个目录中的所有文件。我们可以通过运行循环来列出所有文件来简单地做到这一点。但是，如果里面还有另一个目录呢？那么，我们必须运行另一个循环来获取该目录中的所有文件。如果该目录中还有另一个目录，依此类推呢？在这种情况下，迭代方法可能根本无济于事，或者可能会产生一个复杂的解决方案。在这里最好选择递归方法。

递归管理一个调用堆栈来管理函数调用。因此，与迭代相比，递归将需要更多的内存和时间来完成。此外，在迭代中，每一步都可以得到一个结果，但对于递归，我们必须等到基本情况执行才能得到任何结果。如果我们考虑阶乘的迭代和递归示例，我们可以看到有一个名为`$result`的局部变量来存储每一步的计算。然而，在递归中，不需要局部变量或赋值。

# 使用递归实现斐波那契数

在数学中，斐波那契数是特殊的整数序列，其中一个数字由过去两个数字的求和组成，如下所示的表达式：

![](img/Image00035.gif)

如果我们使用 PHP 7 来实现，它将如下所示：

```php
function fibonacci(int $n): int { 

    if ($n == 0) { 

    return 1; 

    } else if ($n == 1) { 

    return 1; 

    } else { 

    return fibonacci($n - 1) + fibonacci($n - 2); 

    } 

}

```

如果我们考虑前面的实现，可以看到它与以前的示例有些不同。现在，我们从一个函数调用中调用两个函数。我们将很快讨论不同类型的递归。

# 使用递归实现 GCD 计算

递归的另一个常见用途是实现两个数字的**最大公约数**（**GCD**）计算。在 GCD 计算中，我们会一直进行下去，直到余数变为 0。可以表示如下：

![](img/Image00036.jpg)

现在，如果我们使用 PHP 7 进行递归实现，它将如下所示：

```php
function gcd(int $a, int $b): int { 

    if ($b == 0) { 

     return $a; 

    } else { 

     return gcd($b, $a % $b); 

    } 

}

```

这个实现的另一个有趣部分是，与阶乘不同，我们不是从基本情况返回到调用堆栈中的其他步骤。基本情况将返回计算出的值。这是递归的一种优化方式。

# 不同类型的递归

到目前为止，我们已经看到了一些递归的示例案例以及它的使用方式。尽管术语是递归，但有不同类型的递归。我们将逐一探讨它们。

# 线性递归

在编程世界中最常用的递归之一是线性递归。当一个函数在每次运行中只调用自身一次时，我们将其称为线性递归。就像我们的阶乘示例一样，当我们将大的计算分解为较小的计算，直到达到基本条件时，我们称之为缠绕。当我们从基本条件返回到第一个递归调用时，我们称之为展开。在本章的后续部分中，我们将研究不同的线性递归。

# 二进制递归

在二进制递归中，函数在每次运行中调用自身两次。因此，计算取决于对自身的两个不同递归调用的结果。如果我们看看我们的斐波那契序列生成递归函数，我们很容易发现它是一个二进制递归。除此之外，在编程世界中，我们还有许多常用的二进制递归，如二分查找、分治、归并排序等。下图显示了一个二进制递归：

![](img/Image00037.jpg)

# 尾递归

当返回时没有待处理的操作时，递归方法是尾递归。例如，在我们的阶乘代码中，返回的值用于与前一个值相乘以计算阶乘。因此，这不是尾递归。斐波那契数列递归也是如此。如果我们看看我们的最大公约数递归，我们会发现在返回后没有要执行的操作。因此，最终返回或基本情况返回实际上就是答案。因此，最大公约数是尾递归的一个例子。尾递归也是线性递归的一种形式。

# 相互递归

可能会出现这样的情况，我们可能需要从两个不同的方法中交替地递归调用两个不同的方法。例如，函数 `A()` 调用函数 `B()`，函数 `B()` 在每次调用中调用函数 `A()`。这被称为相互递归。

# 嵌套递归

当递归函数调用自身作为参数时，它被称为嵌套递归。嵌套递归的一个常见例子是 Ackermann 函数。看看下面的方程：

![](img/Image00038.gif)

如果我们看最后一行，我们可以看到函数 `A()` 被递归调用，但第二个参数本身是另一个递归调用。因此，这是嵌套递归的一个例子。

尽管有不同类型的递归可用，但我们只会根据我们的需求使用那些必需的。现在，我们将看到递归在我们的项目中的一些实际用途。

# 使用递归构建 N 级类别树

构建多级嵌套的类别树或菜单总是一个问题。许多 CMS 和网站只允许一定级别的嵌套。为了避免由于多次连接而导致的性能问题，一些只允许最多 3-4 级的嵌套。现在，我们将探讨如何利用递归创建一个 N 级嵌套的类别树或菜单，而不会影响性能。以下是我们解决方案的方法：

1.  我们将为数据库中的类别定义表结构。

1.  我们将在不使用任何连接或多个查询的情况下获取表中的所有类别。这将是一个带有简单选择语句的单个数据库查询。

1.  我们将构建一个类别数组，以便我们可以利用递归来显示嵌套的类别或菜单。

让我们假设我们的数据库中有一个简单的表结构来存储我们的类别，它看起来像这样：

```php
CREATE TABLE `categories` ( 

  `id` int(11) NOT NULL, 

  `categoryName` varchar(100) NOT NULL, 

  `parentCategory` int(11) DEFAULT 0, 

  `sortInd` int(11) NOT NULL 

) ENGINE=InnoDB DEFAULT CHARSET=utf8;

```

为简单起见，我们假设表中不需要其他字段。此外，我们的表中有一些数据如下：

| **Id** | **类别名称** | **父类别** | **排序索引** |
| --- | --- | --- | --- |
| 1 | 第一 | 0 | 0 |
| 2 | 第二 | 1 | 0 |
| 3 | 第三 | 1 | 1 |
| 4 | 第四 | 3 | 0 |
| 5 | 第五 | 4 | 0 |
| 6 | 第六 | 5 | 0 |
| 7 | 第七 | 6 | 0 |
| 8 | 第八 | 7 | 0 |
| 9 | 第九 | 1 | 0 |
| 10 | 第十 | 2 | 1 |

现在，我们已经为我们的数据库创建了一个结构化的表，并且我们也假设输入了一些示例数据。让我们构建一个查询来检索这些数据，以便我们可以转移到我们的递归解决方案：

```php
$dsn = "mysql:host=127.0.0.1;port=3306;dbname=packt;"; 

$username = "root"; 

$password = ""; 

$dbh = new PDO($dsn, $username, $password); 

$result = $dbh->query("Select * from categories order by parentCategory asc, sortInd asc", PDO::FETCH_OBJ); 

$categories = []; 

foreach($result as $row) { 

    $categories[$row->parentCategory][] = $row;

}

```

上述代码的核心部分是我们如何将我们的类别存储在数组中。我们根据它们的父类别存储结果。这将帮助我们递归地显示类别的子类别。这看起来非常简单。现在，基于类别数组，让我们编写递归函数以分层显示类别：

```php
function showCategoryTree(Array $categories, int $n) {

    if(isset($categories[$n])) { 

      foreach($categories[$n] as $category) {        

          echo str_repeat("-", $n)."".$category->categoryName."\n"; 

          showCategoryTree($categories, $category->id);          

      }

    }

    return;

}

```

上述代码实际上显示了所有类别及其子类别的递归。我们取一个级别，首先打印该级别上的类别。接着，我们将检查它是否有任何子级别的类别，使用代码 `showCategoryTree($categories, $category->id)`。现在，如果我们用根级别（级别 0）调用递归函数，那么我们将得到以下输出：

```php
showCategoryTree($categories, 0);

```

这将产生以下输出：

```php
First

-Second

--Tenth

-Third

---Fourth

----fifth

-----Sixth

------seventh

-------Eighth

-Nineth

```

正如我们所看到的，不需要考虑类别级别的深度或多个查询，我们可以只用一个简单的查询和递归函数构建嵌套类别或菜单。如果需要动态显示和隐藏功能，我们可以使用`<ul>`和`<li>`来创建嵌套菜单。这对于在不涉及实现阻碍的情况下获得问题的有效解决方案非常重要，比如具有固定级别的连接或固定级别的类别。前面的示例是尾递归的完美展示，在这里我们不需要等待递归返回任何东西，随着我们的前进，结果已经显示出来了。

# 构建嵌套的评论回复系统

我们经常面临的挑战是以适当的方式显示评论回复。按时间顺序显示它们有时不符合我们的需求。我们可能需要以这样的方式显示它们，即每条评论的回复都在实际评论本身下面。换句话说，我们可以说我们需要一个嵌套的评论回复系统或者线程化评论。我们想要构建类似以下截图的东西：

![](img/Image00039.jpg)

我们可以按照嵌套类别部分所做的相同步骤进行。但是，这一次，我们将有一些 UI 元素，使其看起来更真实。假设我们有一个名为`comments`的表，其中包含以下数据和列。为简单起见，我们不涉及多个表关系。我们假设用户名存储在与评论相同的表中：

| **Id** | **评论** | **用户名** | **日期时间** | **父 ID** | **帖子 ID** |
| --- | --- | --- | --- | --- | --- |
| 1 | 第一条评论 | Mizan | 2016-10-01 15:10:20 | 0 | 1 |
| 2 | 第一条回复 | Adiyan | 2016-10-02 04:09:10 | 1 | 1 |
| 3 | 第一条回复的回复 | Mikhael | 2016-10-03 11:10:47 | 2 | 1 |
| 4 | 第一条回复的回复的回复 | Arshad | 2016-10-04 21:22:45 | 3 | 1 |
| 5 | 第一条回复的回复的回复的回复 | Anam | 2016-10-05 12:01:29 | 4 | 1 |
| 6 | 第二条评论 | Keith | 2016-10-01 15:10:20 | 0 | 1 |
| 7 | 第二篇帖子的第一条评论 | Milon | 2016-10-02 04:09:10 | 0 | 2 |
| 8 | 第三条评论 | Ikrum | 2016-10-03 11:10:47 | 0 | 1 |
| 9 | 第二篇帖子的第二条评论 | Ahmed | 2016-10-04 21:22:45 | 0 | 2 |
| 10 | 第二篇帖子的第二条评论的回复 | Afsar | 2016-10-18 05:18:24 | 9 | 2 |

现在让我们编写一个准备好的语句来从帖子中获取所有评论。然后，我们可以构建一个类似嵌套类别的数组：

```php
$sql = "Select * from comments where postID = :postID order by parentID asc, datetime asc"; 

$stmt = $dbh->prepare($sql, array(PDO::ATTR_CURSOR => PDO::CURSOR_FWDONLY)); 

$stmt->setFetchMode(PDO::FETCH_OBJ); 

$stmt->execute(array(':postID' => 1)); 

$result = $stmt->fetchAll(); 

$comments = []; 

foreach ($result as $row) { 

    $comments[$row->parentID][] = $row;

}

```

现在，我们有了数组和其中的所有必需数据；我们现在可以编写一个函数，该函数将递归调用以正确缩进显示评论：

```php
function displayComment(Array $comments, int $n) { 

   if (isset($comments[$n])) { 

      $str = "<ul>"; 

      foreach ($comments[$n] as $comment) { 

          $str .= "<li><div class='comment'><span class='pic'>

            {$comment->username}</span>"; 

          $str .= "<span class='datetime'>{$comment->datetime}</span>"; 

          $str .= "<span class='commenttext'>" . $comment->comment . "

            </span></div>"; 

          $str .= displayComment($comments, $comment->id); 

          $str .= "</li>"; 

       } 

      $str .= "</ul>"; 

      return $str; 

    } 

    return ""; 

} 

echo displayComment($comments, 0); 

```

由于我们在 PHP 代码中添加了一些 HTML 元素，因此我们需要一些基本的 CSS 来使其工作。这是我们编写的 CSS 代码，用于创建清晰的设计。没有花哨的东西，只是纯 CSS 来创建级联效果和对评论每个部分的基本样式：

```php
  ul { 

      list-style: none; 

      clear: both; 

  }

  li ul { 

      margin: 0px 0px 0px 50px; 

  } 

  .pic { 

      display: block; 

      width: 50px; 

      height: 50px; 

      float: left; 

      color: #000; 

      background: #ADDFEE; 

      padding: 15px 10px; 

      text-align: center; 

      margin-right: 20px; 

  }

  .comment { 

      float: left; 

      clear: both; 

      margin: 20px; 

      width: 500px; 

  }

  .datetime { 

      clear: right; 

      width: 400px; 

      margin-bottom: 10px; 

      float: left; 

  }

```

正如前面提到的，我们在这里并不试图做一些复杂的东西，只是响应式的，设备友好的等等。我们假设您可以在应用程序的不同部分集成逻辑而不会出现任何问题。

这是数据和前面代码的输出：

![](img/Image00040.jpg)

从前面的两个示例中，我们可以看到，很容易创建嵌套内容，而无需多个查询或对嵌套的连接语句有限制。我们甚至不需要自连接来生成嵌套数据。

# 使用递归查找文件和目录

我们经常需要找到目录中的所有文件。这包括其中所有的子目录，以及这些子目录中的目录。因此，我们需要一个递归解决方案来找到给定目录中的文件列表。以下示例将展示一个简单的递归函数来列出目录中的所有文件：

```php
function showFiles(string $dirName, Array &$allFiles = []) { 

    $files = scandir($dirName); 

    foreach ($files as $key => $value) { 

      $path = realpath($dirName . DIRECTORY_SEPARATOR . $value); 

      if (!is_dir($path)) { 

          $allFiles[] = $path; 

      } else if ($value != "." && $value != "..") { 

          showFiles($path, $allFiles); 

          $allFiles[] = $path; 

      } 

   } 

    return; 

} 

$files = []; 

showFiles(".", $files);

```

`showFiles` 函数实际上接受一个目录，并首先扫描目录以列出其中的所有文件和目录。然后，通过 `foreach` 循环，它遍历每个文件和目录。如果是一个目录，我们再次调用 `.` 函数以列出其中的文件和目录。这将继续，直到我们遍历所有文件和目录。现在，我们有了 `$files` 数组下的所有文件。现在，让我们使用 `foreach` 循环顺序显示文件：

```php
foreach($files as $file) {

    echo $file."\n";

}

```

这将在命令行中产生以下输出：

```php
/home/mizan/packtbook/chapter_1_1.php

/home/mizan/packtbook/chapter_1_2.php

/home/mizan/packtbook/chapter_2_1.php

/home/mizan/packtbook/chapter_2_2.php

/home/mizan/packtbook/chapter_3_.php

/home/mizan/packtbook/chapter_3_1.php

/home/mizan/packtbook/chapter_3_2.php

/home/mizan/packtbook/chapter_3_4.php

/home/mizan/packtbook/chapter_4_1.php

/home/mizan/packtbook/chapter_4_10.php

/home/mizan/packtbook/chapter_4_11.php

/home/mizan/packtbook/chapter_4_2.php

/home/mizan/packtbook/chapter_4_3.php

/home/mizan/packtbook/chapter_4_4.php

/home/mizan/packtbook/chapter_4_5.php

/home/mizan/packtbook/chapter_4_6.php

/home/mizan/packtbook/chapter_4_7.php

/home/mizan/packtbook/chapter_4_8.php

/home/mizan/packtbook/chapter_4_9.php

/home/mizan/packtbook/chapter_5_1.php

/home/mizan/packtbook/chapter_5_2.php

/home/mizan/packtbook/chapter_5_3.php

/home/mizan/packtbook/chapter_5_4.php

/home/mizan/packtbook/chapter_5_5.php

/home/mizan/packtbook/chapter_5_6.php

/home/mizan/packtbook/chapter_5_7.php

/home/mizan/packtbook/chapter_5_8.php

/home/mizan/packtbook/chapter_5_9.php

```

这些是我们在开发过程中面临的一些常见挑战的解决方案。然而，还有其他地方我们将大量使用递归，比如二进制搜索、树、分治算法等。我们将在接下来的章节中讨论它们。

# 分析递归算法

递归算法的分析取决于我们使用的递归类型。如果是线性的，复杂度将不同；如果是二进制的，复杂度也将不同。因此，递归算法没有通用的复杂度。我们必须根据具体情况进行分析。在这里，我们将分析阶乘序列。首先，让我们专注于阶乘部分。如果我们回忆一下这一节，我们对阶乘递归有这样的东西：

```php
function factorial(int $n): int { 

    if ($n == 0) 

    return 1; 

    return $n * factorial($n - 1); 

} 

```

假设计算阶乘（`$n`）需要 `T(n)`。我们将专注于如何使用大 O 符号表示这个 `T(n)`。每次调用阶乘函数时，都涉及某些步骤：

1.  每次，我们都在检查基本情况。

1.  然后，我们在每个循环中调用阶乘（`$n-1`）。

1.  我们在每个循环中用 `$n` 进行乘法。

1.  然后，我们返回结果。

现在，如果我们用 `T(n)` 表示这个，那么我们可以说：

*T(n) = 当 n = 0 时，a*

*T(n) = 当 n > 0 时，T(n-1) + b*

在这里，*a* 和 *b* 都是一些常数。现在，让我们用 *n* 生成 *a* 和 *b* 之间的关系。我们可以轻松地写出以下方程：

*T(0) = a*

*T(1) = T(0) + b = a + b*

*T(2) = T(1) + b = a + b + b = a + 2b*

*T(3) = T(2) + b = a + 2b + b = a + 3b*

*T(4) = T(3) + b = a + 3b + b = a + 4b*

我们可以看到这里出现了一个模式。因此，我们可以确定：

*T(n) = a + (n) b*

或者，我们也可以简单地说 `T(n) = O(n)`。

因此，阶乘递归具有线性复杂度 `O(n)`。

具有递归的斐波那契序列大约具有 `O(2^n)` 的复杂度。计算非常详细，因为我们必须考虑大 O 符号的下界和上界。在接下来的章节中，我们还将分析二进制递归，如二进制搜索和归并排序。我们将在这些章节中更多地关注递归分析。

# PHP 中的最大递归深度

由于递归是函数调用自身的过程，我们可以心中有一个有效的问题，比如“这个递归可以有多深？”。让我们为此编写一个小程序：

```php
function maxDepth() {

    static $i = 0;

    print ++$i . "\n";

    maxDepth();

}

maxDepth();

```

我们能猜测最大深度水平吗？在耗尽内存限制之前，深度达到了 917,056 级。如果启用了**XDebug**，那么限制将比这个小得多。这也取决于您的内存、操作系统和 PHP 设置，如内存限制和最大执行时间。

虽然我们有选择深入进行递归，但始终重要的是要记住，我们必须控制好我们的递归函数。我们应该知道基本条件以及递归必须在何处结束。否则，可能会产生一些错误的结果或突然结束。

# 使用 SPL 递归迭代器

标准 PHP 库 SPL 有许多内置的迭代器，用于递归目的。我们可以根据需要使用它们，而不必费力实现它们。以下是迭代器及其功能的列表：

+   **RecursiveArrayIterator**：这个递归迭代器允许迭代任何类型的数组或对象，并修改键或值，或取消它们。它还允许迭代当前迭代器条目。

+   递归回调过滤迭代器：如果我们希望递归地将回调应用于任何数组或对象，这个迭代器可以非常有帮助。

+   递归目录迭代器：这个迭代器允许迭代任何目录或文件系统。它使得目录列表非常容易。例如，我们可以很容易地使用这个迭代器重新编写本章中编写的目录列表程序：

```php
$path = realpath('.'); 

$files = new RecursiveIteratorIterator( 

   new RecursiveDirectoryIterator($path), RecursiveIteratorIterator::SELF_FIRST); 

foreach ($files as $name => $file) { 

    echo "$name\n"; 

}

```

+   递归过滤迭代器：如果我们在迭代过程中递归地寻找过滤选项，我们可以使用这个抽象迭代器来实现过滤部分。

+   递归迭代迭代器：如果我们想要迭代任何递归迭代器，我们可以使用这个。它已经内置，我们可以很容易地应用它。在`RecursiveDirectoryIterator`部分中显示了它的使用示例。

+   递归正则表达式迭代器：如果您想要应用正则表达式来过滤迭代器，我们可以使用这个迭代器以及其他迭代器。

+   递归树迭代器：递归树迭代器允许我们为任何目录或多维数组创建类似树的图形表示。例如，以下足球队列表数组将产生树结构：

```php
$teams = array( 

    'Popular Football Teams', 

    array( 

  'La Lega', 

  array('Real Madrid', 'FC Barcelona', 'Athletico Madrid', 'Real  

    Betis', 'Osasuna') 

    ), 

    array( 

  'English Premier League', 

  array('Manchester United', 'Liverpool', 'Manchester City', 'Arsenal',   

    'Chelsea') 

    ) 

); 

$tree = new RecursiveTreeIterator( 

  new RecursiveArrayIterator($teams), null, null, RecursiveIteratorIterator::LEAVES_ONLY 

); 

foreach ($tree as $leaf) 

    echo $leaf . PHP_EOL;

```

输出将如下所示：

```php
|-Popular Football Teams

| |-La Lega

|   |-Real Madrid

|   |-FC Barcelona

|   |-Athletico Madrid

|   |-Real Betis

|   \-Osasuna

 |-English Premier League

 |-Manchester United

 |-Liverpool

 |-Manchester City

 |-Arsenal

 \-Chelsea

```

# 使用 PHP 内置函数 array_walk_recursive

`array_walk_recursive`可以是 PHP 中非常方便的内置函数，因为它可以递归地遍历任何大小的数组并应用回调函数。无论我们想要找出多维数组中是否存在元素，还是获取多维数组的总和，我们都可以毫无问题地使用这个函数。

执行以下代码示例将产生输出**136**：

```php
function array_sum_recursive(Array $array) { 

    $sum = 0; 

    array_walk_recursive($array, function($v) use (&$sum) { 

      $sum += $v; 

    }); 

    return $sum; 

} 

$arr =  

[1, 2, 3, 4, 5, [6, 7, [8, 9, 10, [11, 12, 13, [14, 15, 16]]]]]; 

echo array_sum_recursive($arr); 

```

PHP 中的另外两个内置递归数组函数是`array_merge_recursive`和`array_replace_recursive`。我们可以使用它们来合并多个数组到一个数组中，或者从多个数组中替换，分别。

# 总结

到目前为止，我们讨论了递归的不同属性和实际用途。我们已经看到了如何分析递归算法。计算机编程和递归是两个不可分割的部分。递归的使用几乎无处不在于编程世界中。在接下来的章节中，我们将更深入地探讨它，并在适用的地方应用它。在下一章中，我们将讨论另一个特殊的数据结构，称为“树”。
