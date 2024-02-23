# 第四章。合并类和函数

现在我们已经有了一个自动加载程序，我们可以开始删除所有只加载类和函数定义的`include`调用。完成后，剩下的唯一`include`调用将是执行逻辑的。这将使我们更容易看到哪些`include`调用正在形成我们遗留应用程序中的逻辑路径，哪些仅仅提供定义。

我们将从一个代码库结构相对良好的场景开始。之后，我们将回答一些与不太适合修改的布局相关的问题。

### 注意

在本章中，我们将使用术语`include`来覆盖不仅仅是`include`，还包括`require`、`include_once`和`require_once`。

# 合并类文件

首先，我们将所有应用程序类合并到我们在上一章确定的中心目录位置。这样做将使它们放在我们的自动加载程序可以找到它们的地方。以下是我们将遵循的一般流程：

1.  找到一个`include`语句，用于引入一个类定义文件。

1.  将该类定义文件移动到我们的中心类目录位置，确保它被放置在符合 PSR-0 规则的子路径中。

1.  在原始文件*和代码库中的所有其他文件*中，如果有一个`include`引入了该类定义，删除该`include`语句。

1.  抽查以确保所有文件现在都自动加载该类，通过浏览它们或以其他方式运行它们。

1.  提交、推送并通知 QA。

1.  重复直到没有更多引入类定义的`include`调用。

对于我们的示例，我们假设我们有一个遗留应用程序，其部分文件系统布局如下：

```php
/path/to/app/
```

```php
classes/          # our central class directory location
Mlaphp/
Autoloader.php    # A hypothetical autoloader class
foo/ bar/ baz.php # a page script
includes/         # a common "includes" directory
setup.php         # setup code
index.php         # a page script
lib/              # a directory with some classes in it
sub/ Auth.php     # class Auth { ... }
Role.php          # class Role { ... }
User.php          # class User { ... }
```

你自己的遗留应用程序可能不完全匹配这个，但你明白了。

## 找到一个候选包括

我们首先选择一个文件，任何文件，然后我们检查其中的`include`调用。其中的代码可能如下所示：

```php
1 <?php
2 require 'includes/setup.php';
3 require_once 'lib/sub/User.php';
4
5 // ...
6 $user = new User();
7 // ...
8 ?>
```

我们可以看到有一个新的`User`类被实例化。在检查`lib/sub/User.php`文件时，我们可以看到它是其中唯一定义的类。

## 移动类文件

已经确定了一个`include`语句，用于加载类定义，现在我们将该类定义文件移动到中心类目录位置，以便我们的自动加载程序函数可以找到它。现在的文件系统布局如下（请注意，`User.php`现在在`classes/`中）：

```php
**/path/to/app/**
classes/                 # our central class directory location
Mlaphp/ Autoloader.php   # A hypothetical autoloader class
User.php                 # class User { ... }
foo/ bar/ baz.php        # a page script
includes/                # a common "includes" directory
setup.php                # setup code
db_functions.php         # a function definition file
index.php                # a page script
lib/                     # a directory with some classes in it
sub/
Auth.php                 # class Auth { ... }
Role.php                 # class Role { ... } ~~
```

## 删除相关的包括调用

现在问题是，我们的原始文件试图从其旧位置`include`类文件，而这个位置已经不存在了。我们需要从代码中删除这个调用：

```php
**index.php**
1 <?php
2 require 'includes/setup.php';
3
4 // ...
5 // the User class is now autoloaded
6 $user = new User();
7 // ...
8 ?>
```

然而，代码可能还有其他地方尝试加载现在已经缺失的`lib/sub/User.php`文件。

这就是项目范围搜索工具派上用场的地方。这里我们有不同的选择，取决于你选择的编辑器/IDE 和操作系统。

+   在 GUI 编辑器中，如 TextMate、SublimeText 和 PHPStorm，通常有一个**在项目中查找**的菜单项，我们可以用它来一次性搜索所有应用程序文件中的字符串或正则表达式。

+   在 Emacs 和 Vim 等其他编辑器中，通常有一个键绑定，可以搜索特定目录及其子目录中的所有文件，以查找字符串或正则表达式。

+   最后，如果你是老派的，你可以在命令行中使用`grep`来搜索特定目录及其子目录中的所有文件。

重点是找到所有引用`lib/sub/User.php`的`include`调用。因为`include`调用可以以不同的方式形成，我们需要使用这样的正则表达式来搜索`include`调用：

```php
**^[ \t]*(include|include_once|require|require_once).*User\.php**

```

如果你不熟悉正则表达式，这里是我们要寻找的内容的分解：

```php
**^**               Starting at the beginning of each line,
**[ \t]***          followed by zero or more spaces and/or tabs,
**(include|...)**   followed by any of these words,
**.***             followed by any characters at all,
**User\.php**      followed by User.php, and we don't care what comes after.
```

（正则表达式使用`.`表示`任何字符`，所以我们必须指定`User\.php`来表示我们指的是一个字面上的点，而不是任何字符。）

如果我们使用正则表达式搜索来查找遗留代码库中的这些字符串，我们将得到所有匹配行及其对应的文件列表。不幸的是，我们需要检查每一行，看它是否真的是对`lib/sub/User.php`文件的引用。例如，这行可能会出现在搜索结果中：

```php
**include_once("/usr/local/php/lib/User.php");**

```

然而，显然我们不是在寻找`User.php`文件。

### 注意

我们可以对我们的正则表达式更严格，这样我们就可以专门搜索`lib/sub/User.php`，但这更有可能错过一些`include`调用，特别是那些在`lib/`或`sub/`目录下的文件。例如，在`sub/`文件中的`include`可能是这样的：

```php
include 'User.php';
```

因此，最好宽松一点地搜索以获得每个可能的候选项，然后手动处理结果。

检查每个搜索结果行，如果是引入`User`类的`include`，则删除它并保存文件。保留每个修改后的文件的列表，因为我们以后需要对它们进行测试。

最终，我们将删除整个代码库中该类的所有`include`调用。

## 抽查代码库

在删除给定类的`include`语句之后，我们现在需要确保应用程序正常工作。不幸的是，因为我们没有建立测试流程，这意味着我们需要通过浏览或以其他方式调用修改后的文件来进行伪测试或抽查。实际上，这通常并不困难，但很繁琐。

当我们进行抽查时，我们特别寻找*文件未找到*和*类未定义*错误。这意味着分别尝试`include`缺失的类文件，或者自动加载程序无法找到类文件。

为了进行测试，我们需要设置 PHP 错误报告，以便直接显示错误，或将错误记录到我们在测试代码库时检查的文件中。此外，错误报告级别需要足够严格，以便我们实际上看到错误。一般来说，`error_reporting(E_ALL)`是我们想要的，但因为这是一个遗留的代码库，它可能显示比我们能忍受的更多的错误（特别是*变量未定义*通知）。因此，将`error_reporting(E_WARNING)`设置为更有成效。错误报告值可以在设置或引导文件中设置，也可以在正确的`php.ini`文件中设置。

## 提交、推送、通知 QA

测试完成并修复所有错误后，将代码提交到源代码控制，并（如果需要）将其推送到中央代码存储库。如果您有一个质量保证团队，现在是通知他们需要进行新一轮测试并提供测试文件列表的时候了。

## 做...直到

这是将单个类从`include`转换为自动加载的过程。回顾代码库，找到下一个`include`引入类文件并重新开始该过程。一直持续下去，直到所有类都已合并到中央类目录位置，并且相关的`include`行已被删除。是的，这是一个乏味、繁琐和耗时的过程，但这是现代化我们遗留代码库的必要步骤。

# 将函数合并到类文件中

并非所有的遗留应用程序都使用大量的类。通常，除了类之外，还有大量用户定义的核心逻辑函数。

使用函数本身并不是问题，但这意味着我们需要`include`定义函数的文件。但自动加载只适用于类。找到一种方法自动加载函数文件以及类文件将是有益的。这将帮助我们删除更多的`include`调用。

这里的解决方案是将函数移到类文件中，并在这些类上调用函数作为静态方法。这样，自动加载程序可以为我们加载类文件，然后我们可以调用该类中的方法。

这个过程比我们合并类文件时更复杂。以下是我们将遵循的一般流程：

1.  找到一个`include`语句，引入一个函数定义文件。

1.  将该函数定义文件转换为一组静态方法的类文件；我们需要为该类选择一个唯一的名称，并且可能需要将函数重命名为更合适的方法名称。

1.  在原始文件*和代码库中的所有其他文件*中，如果使用了该文件中的任何函数，将这些函数的调用更改为静态方法调用。

1.  通过浏览或以其他方式调用受影响的文件来检查新的静态方法调用是否有效。

1.  将类文件移动到中央类目录位置。

1.  在原始文件*和代码库中的所有其他文件*中，如果有`include`引入该类定义，删除相关的`include`语句。

1.  再次进行抽查，确保所有文件现在都通过自动加载该类来浏览或运行它们。

1.  提交、推送并通知 QA。

1.  重复，直到没有更多的`include`调用引入函数定义文件。

## 寻找一个包括候选者

我们选择一个文件，任何文件，并查找其中的`include`调用。我们选择的文件中的代码可能如下所示：

```php
1 <?php
2 require 'includes/setup.php';
3 require_once 'includes/db_functions.php';
4
5 // ...
6 $result = db_query('SELECT * FROM table_name');
7 // ...
8 ?>
```

我们可以看到有一个`db_query()`函数被使用，并且在检查`includes/db_functions.php`文件时，我们可以看到其中定义了该函数以及其他几个函数。

## 将函数文件转换为类文件

假设`db_functions.php`文件看起来像这样：

```php
**includes/db_functions.php**
1 <?php
2 function db_query($query_string)
3 {
4 // ... code to perform a query ...
5 }
6
7 function db_get_row($query_string)
8 {
9 // ... code to get the first result row
10 }
11
12 function db_get_col($query_string)
13 {
14 // ... code to get the first column of results ...
15 }
16 ?>
```

要将此函数文件转换为类文件，我们需要为即将创建的类选择一个唯一的名称。在这种情况下，从文件名和函数名称来看，这似乎很明显，这些都是与数据库相关的调用。因此，我们将称这个类为 Db。

现在我们有了一个名称，我们将创建这个类。这些函数将成为类中的静态方法。我们暂时不会移动文件；将其保留在当前文件名的位置。

然后我们进行更改，将文件转换为类定义。如果我们更改函数名称，我们需要保留旧名称和新名称的列表以供以后使用。更改后，它将看起来像下面这样（注意更改后的方法名称）：

```php
**includes/db_functions.php**
1 <?php
2 class Db
3 {
4 public static function query($query_string)
5 {
6 // ... code to perform a query ...
7 }
8
9 public static function getRow($query_string)
10 {
11 // ... code to get the first result row
12 }
13
14 public static function getCol($query_string)
15 {
16 // ... code to get the first column of results ...
17 }
18 }
19 ?>
```

更改非常温和：我们将函数包装在一个唯一的类名中，标记为`public static`，并对函数名称进行了轻微更改。我们对函数签名或函数本身的代码没有做任何更改。

### 将函数调用更改为静态方法调用

我们已经将`db_functions.php`的内容从函数定义转换为类定义。如果我们现在尝试运行应用程序，它将因为"未定义的函数"错误而失败。因此，下一步是找到应用程序中所有相关的函数调用，并将它们重命名为我们新类的静态方法调用。

没有简单的方法可以做到这一点。这是另一种情况，项目范围的搜索和替换非常方便。使用我们首选的项目范围搜索工具，搜索`old`函数调用，并将其替换为`new`静态方法调用。例如，使用正则表达式，我们可能会这样做：

搜索：

```php
db_query\s*\(
```

替换为：

```php
Db::query(
```

正则表达式表示的是开括号，而不是闭括号，因为我们不需要在函数调用中查找参数。这有助于区分可能以我们正在搜索的函数名为前缀的函数名称，例如`db_query_raw()`。正则表达式还允许在函数名和开括号之间有可选的空格，因为一些样式指南建议这样的间距。

对旧函数文件中的每个`old`函数名称执行此搜索和替换，将每个函数转换为新类文件中的`new`静态方法调用。

### 检查静态方法调用

当我们完成将旧的函数名称重命名为新的静态方法调用后，我们需要遍历代码库以确保一切正常。同样，这并不容易。你可能需要浏览或以其他方式调用在这个过程中被更改的每个文件。

### 移动类文件

此时，我们已经用类定义替换了函数定义文件的内容，并且“测试”表明新的静态方法调用按预期工作。现在我们需要将文件移动到我们的中央类目录位置，并正确命名。

目前，我们的类定义在`includes/db_functions.php`文件中。该文件中的类名为`Db`，所以将文件移动到其新的可自动加载位置`classes/Db.php`。之后，文件系统将看起来像这样：

```php
**/path/to/app/**
classes/          # our central class directory location
Db.php            # class Db { ... }
Mlaphp/
Autoloader.php    # A hypothetical autoloader class
User.php          # class User { ... }
foo/
bar/
baz.php           # a page script
includes/         # a common "includes" directory
setup.php         # setup code
index.php         # a page script
lib/              # a directory with some classes in it
sub/
Auth.php          # class Auth { ... }
Role.php          # class Role { ... }
```

## 做...直到

最后，我们遵循与移动类文件相同的结束过程：

+   在整个代码库中删除与函数定义文件相关的`include`调用

+   抽查代码库

+   提交，推送，通知 QA

现在对我们在代码库中找到的每个函数定义文件重复这个过程。

# 常见问题

## 我们应该删除自动加载器的 include 调用吗？

如果我们将我们的自动加载器代码放在一个类中作为静态或实例方法，我们搜索`include`调用将会显示该类文件的包含。如果你移除了那个`include`调用，自动加载将会失败，因为类文件没有被加载。这是一个鸡生蛋蛋生鸡的问题。解决方法是将自动加载器的`include`保留在我们的引导或设置代码中。如果我们完全勤奋地删除`include`调用，那很可能是代码库中唯一剩下的`include`。

## 我们应该如何选择候选的 include 调用文件？

有几种方法可以解决这个问题。我们可以这样做：

+   我们可以手动遍历整个代码库，逐个文件处理。

+   我们可以生成一个类和函数定义文件的列表，然后生成一个`include`这些文件的文件列表。

+   我们可以搜索每个`include`调用，并查看相关文件是否有类或函数定义。

## 如果一个 include 定义了多个类？

有时一个类定义文件可能有多个类定义。这可能会影响自动加载过程。如果一个名为`Foo.php`的文件定义了`Foo`和`Bar`类，那么`Bar`类将永远不会被自动加载，因为文件名是错误的。

解决方法是将单个文件拆分成多个文件。也就是说，为每个类创建一个文件，并根据 PSR-0 命名和自动加载期望命名每个文件。

## 如果每个文件一个类的规则是令人不快的呢？

有时我会听到关于每个文件一个类的规则在检查文件系统时有些浪费或者在审美上不够美观的抱怨。加载那么多文件会影响性能吗？如果有些类只有在某些其他类的情况下才需要，比如只在一个地方使用的`Exception`类呢？我有一些回应：

+   当然，加载两个文件而不是一个会减少性能。问题是*减少了多少*，*与什么相比*？我断言，与我们遗留应用程序中其他更可能的性能问题相比，加载多个文件所带来的影响微乎其微。更有可能的是我们有其他更大的性能问题。如果这真的是一个问题，使用像 APC 这样的字节码缓存将减少或完全消除这些相对较小的性能损失。

+   一致性，一致性，一致性。如果有时一个类文件中只有一个类，而其他时候一个类文件中有多个类，这种不一致性将在项目中的所有人中后来成为认知摩擦的源头。遗留应用程序中的一个主要主题就是不一致性；让我们通过遵守每个文件一个类的规则来尽可能减少这种不一致性。

如果我们觉得某些类自然地属于一起，将从属或子类放在主或父类的子目录中是完全可以接受的。子目录应该根据 PSR-0 命名规则以更高的类或命名空间命名。

例如，如果我们有一系列与`Foo`类相关的`Exception`类：

```php
Foo.php                      # class Foo { ... }
Foo/
NotFoundException.php        # class Foo_NotFoundException { ... }
MalformedDataException.php   # class Foo_MalformedDataException { ... }
```

以这种方式重命名类将改变代码库中实例化或引用它们的相关类名。

## 如果一个类或函数是内联定义的呢？

我见过页面脚本中定义一个或多个类或函数的情况，通常是当这些类或函数只被特定页面脚本使用时。

在这些情况下，从脚本中删除类定义，并将其放在中央类目录位置的单独文件中。确保根据 PSR-0 自动加载规则为它们的类名命名文件。同样，将函数定义移动到它们自己的相关类文件中作为静态方法，并将函数调用重命名为静态方法调用。

## 如果一个定义文件也执行逻辑会怎么样？

我也见过相反的情况，即类文件中有一些逻辑会在文件加载时执行。例如，一个类定义文件可能如下所示：

```php
**/path/to/foo.php**
1 <?php
2 echo "Doing something here ...";
3 log_to_file('a log entry');
4 db_query('UPDATE table_name SET incrementor = incrementor + 1');
5
6 class Foo
7 {
8 // the class
9 }
10 ?>
```

在上述情况下，即使类从未被实例化或以其他方式调用，类定义之前的逻辑也将在文件加载时执行。

这种情况比在页面脚本中内联定义类要难处理得多。类应该可以在不产生副作用的情况下加载，其他逻辑也应该可以执行，而不必加载类。

一般来说，处理这种情况最简单的方法是修改我们的重定位过程。从原始文件中剪切类定义，并将其放在中央类目录位置的单独文件中。保留原始文件及其可执行代码，并保留所有相关的`include`调用。这样我们就可以提取类定义，以便自动加载，但是`include`原始文件的脚本仍然可以获得可执行的行为。

例如，给定上述组合的可执行代码和类定义，我们可能会得到这两个文件：

```php
**/path/to/foo.php**
1 <?php
2 echo "Doing something here ...";
3 log_to_file('a log entry');
4 db_query('UPDATE table_name SET incrementor = incrementor + 1');
5 ?>
```

```php
**/path/to/app/classes/Foo.php**
1 <?php
2 class Foo
3 {
4 // the class
5 }
6 ?>
```

这很混乱，但它保留了现有的应用行为，同时也允许自动加载。

### 如果两个类有相同的名称会怎么样？

当我们开始移动类时，我们可能会发现`应用流 A`使用`Foo`类，而`应用流 B`也使用`Foo`类，但是同名的两个类实际上是在不同文件中定义的不同类。它们永远不会发生冲突，因为这两个不同的应用流永远不会交叉。

在这种情况下，当我们将它们移动到中央类目录位置时，我们必须重命名一个或两个类。例如，将其中一个命名为`FooOne`，另一个命名为`FooTwo`，或者选择更好的描述性名称。将它们分别放在根据 PSR-0 自动加载规则命名的各自类名的单独类文件中，并在整个代码库中重命名对这些类的所有引用。

## 第三方库呢？

当我们合并我们的类和函数时，我们可能会在旧应用程序中找到一些第三方库。我们不想移动或重命名第三方库中的类和函数，因为这样会使以后升级库变得太困难。我们将不得不记住哪些类被移动到哪里，哪些函数被重命名为什么。

带着一些运气，第三方库已经使用了某种自动加载。如果它带有自己的自动加载器，我们可以将该自动加载器添加到 SPL 自动加载器注册表堆栈中，放在我们的设置或引导代码中。如果它的自动加载由另一个自动加载器系统管理，比如 Composer 中的自动加载器，我们可以将*那个*自动加载器添加到 SPL 自动加载器注册表堆栈中，同样是在我们的设置或引导代码中。

如果第三方库不使用自动加载，并且在其自身代码和旧应用程序中都依赖于`include`调用，我们就有点为难了。我们不想修改库中的代码，但同时又想从旧应用程序中删除`include`调用。这里的两个解决方案都是*最不坏*的选择：

+   修改我们应用程序的主自动加载器，以允许一个或多个第三方库

+   为第三方库编写额外的自动加载器，并将其添加到 SPL 自动加载器注册表堆栈中。

这两个选项都超出了本书的范围。您需要检查相关的库，确定其类命名方案，并自行编写适当的自动加载器代码。

最后，在如何组织旧应用程序中的第三方库方面，将它们全部整合到自己的中心位置可能是明智的选择。例如，这可能是在一个名为`3rdparty/`或`external_libs/`的目录下。如果我们移动一个库，我们应该移动整个包，而不仅仅是它的类文件，这样我们以后可以正确地升级它。这还将使我们能够从我们不想修改的文件中排除中心第三方目录，以免得到额外的`include`调用搜索结果。

### 那么系统范围的库呢？

系统范围的库集合，比如 Horde 和 PEAR 提供的库，是第三方库的特例。它们通常位于服务器文件系统*外部*，以便可以供运行在该服务器上的所有应用程序使用。与这些系统范围库相关的`include`语句通常依赖于`include_path`设置，或者是通过绝对路径引用的。

当试图消除仅引入类和函数定义的`include`调用时，这些选项会带来特殊的问题。如果我们足够幸运地使用了 PEAR 安装的库，我们可以修改现有的自动加载器，使其在两个目录而不是一个目录中查找。这是因为 PSR-0 命名约定源自 Horde/PEAR 约定。尾随的自动加载器代码从这个：

```php
1 <?php
2 // convert underscores in the class name to directory separators
3 $subpath .= str_replace('_', DIRECTORY_SEPARATOR, $class);
4
5 // the path to our central class directory location
6 $dir = '/path/to/app/classes'
7
8 // prefix with the central directory location and suffix with .php,
9 // then require it.
10 require $dir . DIRECTORY_SEPARATOR . $subpath . '.php';
11 ?>
```

变成这样：

```php
1 <?php
2 // convert underscores in the class name to directory separators
3 $subpath .= str_replace('_', DIRECTORY_SEPARATOR, $class);
4
5 // the paths to our central class directory location and to PEAR
6 $dirs = array('/path/to/app/classes', '/usr/local/pear/php');
7 foreach ($dirs as $dir) {
8 $file = $dir . DIRECTORY_SEPARATOR . $subpath . '.php';
9 if (file_exists($file)) {
10 require $file;
11 }
12 }
13 ?>
```

## 对于函数，我们可以使用实例方法而不是静态方法吗？

当我们将用户定义的全局函数合并到类中时，我们将它们重新定义为静态方法。这并没有改变它们的全局范围。如果我们感到特别勤奋，我们可以将它们从静态方法更改为实例方法。这需要更多的工作，但最终可以使测试变得更容易，也是一种更清晰的技术方法。考虑到我们之前的`Db`示例，使用实例方法而不是静态方法会是这样的：

```php
**classes/Db.php**
1 <?php
2 class Db
3 {
4 public function query($query_string)
5 {
6 // ... code to perform a query ...
7 }
8
9 public function getRow($query_string)
10 {
11 // ... code to get the first result row
12 }
13
14 public function getCol($query_string)
15 {
16 // ... code to get the first column of results ...
17 }
18 }
19 ?>
```

当使用实例方法而不是静态方法时，唯一增加的步骤是在调用其方法之前需要实例化该类。也就是说，不是这样：

```php
1 <?php
2 Db::query(...);
3 ?>
```

我们会这样做：

```php
1 <?php
2 $db = new Db();
3 $db->query(...);
4 ?>
```

尽管在开始时需要更多的工作，但我建议使用实例方法而不是静态方法。除其他外，这使我们可以在实例化时调用构造方法，并且在许多情况下使测试变得更容易。

如果愿意，您可以首先转换为静态方法，然后再将静态方法转换为实例方法，以及所有相关的方法调用。但是，您的时间表和偏好将决定您选择哪种方法。

### 我们能自动化这个过程吗？

正如我之前所指出的，这是一个乏味、繁琐和耗时的过程。根据代码库的大小，可能需要数天或数周的努力才能完全合并类和函数以进行自动加载。如果有某种方法可以自动化这个过程，使其更快速和更可靠，那将是很好的。

不幸的是，我还没有发现任何可以简化这个过程的工具。据我所知，这种重构最好还是通过细致的手工操作来完成。在这里，有强迫倾向和长时间的不间断专注可能会有所帮助。

# 回顾和下一步

在这一点上，我们已经在现代化我们的传统应用程序中迈出了一大步。我们已经开始从*包含导向*的架构转变为*类导向*的架构。即使以后发现了我们遗漏的类或函数，也没关系；我们可以根据需要多次遵循上述过程，直到所有定义都被移动到中央位置。

我们的应用程序中可能仍然有很多`include`语句，但剩下的那些与应用程序流程有关，而不是拉入类和函数定义。任何剩下的`include`调用都在执行逻辑。我们现在可以更好地看到应用程序的流程。

我们已经为新功能建立了一个结构。每当我们需要添加新的行为时，我们可以将其放入一个新的类中，该类将在我们需要时自动加载。我们可以停止编写新的独立函数；相反，我们将在类上编写新的方法。这些新方法将更容易进行单元测试。

然而，我们为自动加载而合并的*现有*类可能在其中具有全局变量和其他依赖关系。这使它们彼此紧密联系，并且很难为其编写测试。考虑到这一点，下一步是检查我们现有类中的依赖关系，并尝试打破这些依赖关系，以提高我们应用程序的可维护性。
