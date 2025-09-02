# 3

# 将基本 Ruby 语法与 PHP 比较

在 PHP 和 Ruby 中运行脚本相似，尽管每种语言都有其独特之处。同样，Ruby 和 PHP 的语法有时也会出奇地相似，正如我们在*第一章*中看到的。然而，如果我们认真想要成为 Ruby 程序员，我们就需要学习 Ruby 带来的差异和增强。

让我们开始这段旅程，利用我们从知道 PHP 获得的资源来创建、执行和调试我们自己的 Ruby 脚本，并看看 Ruby 语言相对于其他语言的改进。我们不仅会以 Ruby 的思维方式思考，还会以“Ruby 方式”编程。

因此，在本章中，我们将涵盖以下主题：

+   从命令行运行 Ruby 代码

+   探索变量类型

+   使用条件语句

+   使用循环重复代码

+   使用 Ruby 语言增强功能

# 技术要求

要跟随本章内容，我们需要以下内容：

+   任何 IDE 来查看/编辑代码（例如，SublimeText，Visual Studio Code，Notepad++ Vim，Emacs 等）

+   对于 macOS 用户，您还需要安装 Xcode 命令行工具

+   Ruby 版本 2.6 或更高版本需要安装并准备好使用

本章中展示的代码可在[`github.com/PacktPublishing/From-PHP-to-Ruby-on-Rails/`](https://github.com/PacktPublishing/From-PHP-to-Ruby-on-Rails/)找到。

# 从命令行运行 Ruby 代码

在学习 Ruby 时，我们需要了解的第一件事之一是如何在我们的屏幕上直接运行我们的代码并查看输出。有不同方法可以实现这一点，但我们将以最简单的方式来做。虽然从命令行加载代码有多种方式，但我们将从一个文件开始。

## 运行一个简单的代码文件

正如我在引言中提到的，在 Ruby 中运行脚本简单且容易。与在 PHP 中运行脚本类似，我们可以创建一个文件，向其中添加 Ruby 代码，然后用 Ruby 执行它。运行或执行代码简单来说就是 Ruby 将读取（也称为解析）我们的源代码，然后将其翻译成计算机可以理解和处理的语言。

让我们从创建一个名为 `ruby_syntax` 的文件夹开始，这个文件夹位于我们的桌面上。在那个文件夹中，创建我们的源代码文件，命名为 `running_ruby.rb`，使用您选择的 IDE。

现在，让我们向我们的文件中添加一些代码：

```php
# running_ruby.rb
print('I am running a Ruby script');
```

现在，让我们打开一个 shell 并转到我们刚刚创建的相同文件夹：

```php
cd path-to-our-desktop/ruby_syntax
```

一旦我们在 shell 中的这个文件夹里，我们可以用 Ruby 运行我们刚刚创建的脚本：

```php
ruby running_ruby.rb
```

这应该会输出以下内容：

```php
I am running a Ruby script
```

正如我在*第一章*中提到的，这种语法与 PHP 的语法非常相似。如果我们比较两者，我们会得到以下 PHP 等价代码：

```php
<?php # running_php.php
print('I am running a PHP script');
```

我们将按照与 Ruby 一样的方式运行示例，但使用 PHP 可执行文件，如下所示：

```php
php running_php.php
```

结果将与 Ruby 一样，但字符串为 PHP。

回到 Ruby 的例子，就像我们在 *第一章* 中的例子一样，让我们修改我们的 Ruby 代码，使其稍微易于阅读：

```php
# running_ruby.rb
print 'I am running a Ruby script without parenthesis'
```

尽管语法略有相似，但结果相同，而且它的优点是读起来更自然。虽然 Ruby 中有 `print` 用于向用户输出文本，但你也可以使用 `puts` 或简单地 `p`。你会在 Ruby 中经常看到这一点。

## 使用 `load` 方法加载源代码文件

到目前为止，你已经知道了如何在单个文件中执行 Ruby 代码。然而，随着源代码的增长，拥有一个单独的源代码文件将变得不切实际。这就是为什么 Ruby 允许我们从其他源文件加载源代码。

让我们看看如何做到这一点。首先，我们必须创建一个要加载的文件——在这个例子中，文件名为 `my_library.rb`，并包含一些简单的内容：

```php
# my_library.rb
print 'I am a library.'
```

虽然我们现在有了可运行的 Ruby 代码，但我们不会直接运行 `my_library.rb`，而是让另一个脚本加载其代码。这就是 `load` 方法的作用。`load` 方法接受一个文件名及其代码，并将其包含在我们的执行中。所以，让我们创建一个名为 `load_library.rb` 的文件，并包含以下内容：

```php
# load_library.rb
load 'my_library.rb'
```

现在，使用以下命令运行代码：

```php
ruby load_library.rb
```

脚本的输出应该是这样的：

```php
I am a library.
```

`load_library.rb` 文件注入（或者正如其名称所暗示的，*加载*）了 `my_library.rb` 文件中的代码并执行了它。这样，我们可以轻松地将大块代码分成更小、更易读的部分。

现在，如果我们多次在 Ruby 中调用 `load` 方法会发生什么？让我们试试看：

```php
# load_library.rb
load 'my_library.rb'
load 'my_library.rb'
```

再次运行命令后，输出如下：

```php
I am a library. I am a library.
```

从这种行为中，我们可以推断出每次我们加载一个文件，其代码都会被执行，这在我们的文件在执行期间多次更改时非常有用——在这些情况下，代码被认为是动态更改的。

作为动态更改代码的例子，假设我们需要一个脚本来包含代码的新部分，但又不能因此停止。在这种情况下，我们需要代码不断刷新，这就是 `load` 方法的作用。每次 `load` 方法遇到正在更改的文件时，它都会刷新代码——也就是说，每次 Ruby 引擎检测到更改时，它都会更新我们的代码。否则，每次我们需要代码中的新更改时，我们都需要停止执行。

Ruby 的 `load` 方法类似于 PHP 的 `include` 和 `require` 函数。然而，Ruby 的 `require` 方法与 PHP 中的略有不同，正如我们接下来将要看到的。

## 使用 `require` 方法加载源代码文件

与 `load` 方法相反，有时我们只需要代码执行一次，为此我们可以使用 `require` 方法。

让我们通过创建一个名为 `require_library.rb` 的文件并包含以下内容来实际看看：

```php
# require_library.rb
require './my_library.rb'
```

现在，让我们以与运行 `load_library.rb` 脚本相同的方式运行它：

```php
ruby require_library.rb
```

初始输出将与我们运行`load_library.rb`脚本时的输出相同：

```php
I am a library.
```

然而，请注意，在这个例子中，我们在文件名前包含了点斜杠前缀（`./`）。`require`函数需要文件的绝对路径或相对路径——因为`my_library.rb`示例位于同一文件夹中，我们使用当前文件夹的相对路径，即`./`。简单来说，`require_library.rb`的源代码将读取为“注入位于同一文件夹中的`my_library.rb`文件中的代码。”

现在，让我们尝试多次调用`require`方法：

```php
# require_library.rb
require './my_library.rb'
require './my_library.rb'
```

让我们再次用以下代码运行它：

```php
ruby require_library.rb
```

对于经验丰富的 PHP 开发者来说，输出如下应该不会令人惊讶：

```php
I am a library.
```

Ruby 的 `require` 方法几乎与 PHP 中的 `include_once` 和 `require_once` 一样工作。Ruby 解释器只加载代码一次，无论你何时尝试再次加载它，引擎都会注意到你已经加载了那段代码，因此不会再次加载，节省内存和资源。

使用`require`方法时，我们可能还想考虑的一件事是，不强制包含文件扩展名，所以这也适用：

```php
# require_library.rb
require './my_library'
```

通过不使用文件扩展名，我们的代码看起来稍微干净一些。现在，你可能想知道`require`方法有什么用？好吧，无论何时你正在编写将在代码的多个部分中使用的库，你不必担心多次加载你的库，因为`require`方法只会加载一次，从而节省内存资源。我们现在保持简单，但如果你对导入代码的更多选项感兴趣，还有`require_relative`方法：[`apidock.com/ruby/Kernel/require_relative`](https://apidock.com/ruby/Kernel/require_relative)。

## Ruby 类和模块

在 Ruby 中，模块是一种将代码分组以便重用的方式。具体来说，模块是方法集合和常量的集合，可以注入到类中。模块本身并没有什么用，因为不能以独立的方式使用它们。我们使用模块向类添加功能。

类，正如你可能知道的，是从现实世界抽象到编程世界的蓝图。一个类可以有不同的方法来表示动作，以及属性来表示值。一个类可以通过包含来自模块的额外方法和常量来增强。所以，简单来说，一个类可以附加来自模块的方法和常量——这正是`include`方法所做的事情。

让我们通过一个简化的例子来看看。创建一个名为`include_module.rb`的文件，并包含以下代码：

```php
# include_module.rb
class MyClass end
```

这里，我们创建了一个简单的空类`MyClass`，没有方法。现在，让我们创建这个类的实例：

```php
my_class_instance = MyClass.new
```

由于这个类是空的，我们无法用它做很多事情，所以让我们向它添加一个`Utilities`模块：

```php
module Utilities end
```

就像之前的类（`MyClass`）一样，这只是一个空模块。让我们给我们的模块添加一个名为`debug`的方法，这样我们的类和模块最终看起来就像这样：

```php
# include_module.rb
class MyClass end
module Utilities
  def debug
    puts 'We are debugging'
  end
end
my_class_instance = MyClass.new
```

使用这段代码，我们声明了一个名为`debug`的方法，它会打印文本“我们正在调试。”我们能将这个方法添加到`MyClass`中吗？嗯，这就是`include`方法发挥作用的地方。

## 包含方法

到目前为止，我们应该将这个新构建的调试方法与`include`方法结合起来，因为现在，`debug`方法尚未被使用。它仅仅是被定义了。通过`include`方法，我们现在可以通过在文件末尾添加以下内容来“附加”`debug`方法到我们的类中：

```php
MyClass.include(Utilities)
```

这仅仅意味着`debug`方法现在对`my_class_instance`可用。让我们从我们的类实例中调用新添加的`debug`方法。您的最终代码应该看起来像这样：

```php
# include_module.rb
class MyClass end
module Utilities
  def debug
    puts 'We are debugging'
  end
end
my_class_instance = MyClass.new
MyClass.include(Utilities)
my_class_instance.debug
```

现在，到了关键时刻——让我们运行它：

```php
ruby require_library.rb
```

这将输出以下内容：

```php
We are debugging
```

这意味着我们已经成功地将调试方法附加到我们的空类中。每当我们需要在整个代码中重复使用的模块时，这会非常有用。

PHP 中与 Ruby 的`include`方法等效的资源称为`traits`。如果您对这个主题感兴趣，请查看[`www.php.net/manual/en/language.oop5.traits.php`](https://www.php.net/manual/en/language.oop5.traits.php)。由于我们迄今为止还没有在 Ruby 中查看面向对象编程，这可能会一开始有点难以理解，但请不要担心您现在还没有完全理解它——我们将会达到那个水平。

## 交互式 Ruby Shell (IRB)

有时候，您可能想快速测试一小段代码，并且创建文件、添加代码然后运行它可能看起来很麻烦。也许我们只想测试一行代码的语法。好吧，我们不需要创建一个文件来测试一行代码。就像在其他语言中一样，例如 Python 或 PHP，Ruby 中有一个专门为此的工具：它就是**交互式 Ruby Shell**，也被称为**IRB**。让我们看看它。

在 shell 中运行以下命令：

```php
irb
```

这将使您的 shell 看起来像这样：

```php
irb(main):001:0>
```

这个 shell 作为一个实时 Ruby 解释器工作——也就是说，在`>`符号之后，我们可以开始输入 Ruby 命令，这些命令将被立即解释和执行。作为一个简单的例子，让我们加 1 和 1：

```php
irb(main):001:0> 1+1
```

这将返回以下结果：

```php
irb(main):001:0> 1+1
=> 2
```

当您想快速测试语法和操作，或者甚至查看类的内容时，这非常有用。它的工作方式与我们在前面的例子中使用过的 Ruby 二进制文件相同。

所以，作为一个最后的例子，让我们加载我们创建的`include_module`文件，但现在使用这个交互式 shell：

```php
irb(main):001:0> require './include_module'
We are debugging
=> true
```

现在我们已经使用`require`方法将代码包含到我们的 shell 中，我们可以在`irb`会话中使用这个加载的代码。由于我们有`MyClass`的定义可用，我们可以将其用作蓝图来创建`MyClass`的一个实例：

```php
irb(main):001:0> another_instance = MyClass.new
```

这将返回一个 Ruby 内部使用的唯一标识符，以确定实例在计算机内存中的位置：

```php
=> #<MyClass:0x000000014a17ed50>
```

注意，我们也可以在我们的新创建的实例上调用`debug`方法：

```php
irb(main):001:0> another_instance.debug
```

就像之前一样，我们得到了相同的输出：

```php
We are debugging
=> nil
```

要退出这个交互式 shell，只需输入`exit`，使你的 shell 恢复正常。如果你想知道，PHP 中也有一个类似的 shell，称为交互式 shell。如果你对这个主题感兴趣，请务必查看这个页面：[`www.php.net/manual/en/features.commandline.interactive.php`](https://www.php.net/manual/en/features.commandline.interactive.php)。

到目前为止，我们可以自豪地说，我们现在知道如何以几种不同的方式运行 Ruby 代码。我们还知道如何使用单个源代码文件，或者从单独的源代码文件中加载代码。无论哪种方式，无论是使用 Ruby 二进制文件还是使用交互式 shell 运行你的代码，你都需要一种方式来存储和使用值，这把我们带到了下一个主题：变量。

# 探索变量类型

Ruby 中的变量与其他编程语言中的变量具有相同的效用：它们是用于存储值的可变容器。简单来说，变量用于保存值以供以后使用。这些值可能会随时间变化，甚至改变它们包含的数据类型。

就像 PHP 一样，Ruby 是动态类型的（或称为鸭子类型），这意味着解释器在运行时会推断我们正在处理的数据类型。我们不需要告诉 Ruby 或 PHP 一个变量是字符串、数字还是布尔值。然而，与 PHP 的一个区别是，在 PHP 的后续版本中，你可以指定要使用的数据类型，尤其是在面向对象的 PHP 中。然而，即使有这种“增强”，语言的大部分仍然是鸭子类型。

这对我们作为开发者有何影响？好吧，让我们看看一个简单的例子。首先，打开一个 IRS。输入以下命令：

```php
irb
```

正如我们之前看到的，一旦我们输入这个命令，shell 将看起来像这样：

```php
irb(main):001:0>
```

现在，输入以下声明：

```php
name = "Oscar"
age = 35
is_married = true
books_read_this_week = 2.5
```

注意，尽管前一个代码块中的信息是你应该输入的内容，但在提示符中它看起来是这样的：

```php
irb(main):001:0> name = "Oscar"
=> "Oscar"
irb(main):002:0> age = 35
=> 35
irb(main):003:0> is_married = true
=> true
irb(main):004:0>books_read_this_week = 2.5
=> 2.5
```

Ruby 语言的一个核心特性是，每行代码，Ruby 都会尝试返回一个值。如果你声明一个变量，Ruby 将返回分配的值。所以，当我们添加`name`变量时，Ruby 返回了变量的值——即`"Oscar"`。当我们使用这种行为来获取变量所持有的数据类型时，这很有用。Ruby“知道”这个`name`变量包含的数据类型。

要做到这一点，我们可以使用内部的`class`方法。只需输入`name.class`；`irb`应该返回类似以下的内容：

```php
irb(main):005:0> name.class
=> String
```

Ruby 解释器确定`name`变量是一个字符串，或者简单地说，是文本。同样的方法也可以用于我们刚刚声明的其他变量，例如`age`变量：

```php
irb(main):006:0> age.class
=> Integer
```

同样地，`is_married`变量是一个布尔变量。我们可以通过获取该变量的类来确认这一点：

```php
irb(main):006:0> is_married.class
=> TrueClass
```

这意味着`is_married`变量有一个布尔值`true`。我们可以用同样的方法处理`books_read_this_week`变量：

```php
irb(main):006:0> books_read_this_week.class
=> Float
```

我们没有明确告诉 Ruby 我们将使用什么类型的变量，但 Ruby“自动”知道了这一点。这意味着我们不必担心在运行时告诉 Ruby 我们将使用的数据类型。这在大多数情况下是实用的，但我必须承认，我确实遇到过在运行前知道数据类型会很有帮助的情况。

到目前为止，在我们的前几个例子中，我们已经查看了 Ruby 中的四种不同类型的变量：字符串、整数、布尔值（true 或 false）和浮点值。然而，还有三种其他类型的变量我们应该深入了解：数组、哈希和符号。我们现在将逐一介绍它们。

## 数组

数组是一种将常见变量分组的方法。从 PHP 背景来看，数组在概念和语法上都可以很容易理解。从概念上讲，我们将相似或相关的值组合在一起。一个例子可能是将一个人的电话号码组合在一个变量中。另一个例子可能是通过保存街道、门牌号、城市、邮编等来存储物理地址，所有这些都在同一个变量中。正如我们接下来将要看到的，Ruby 的语法与 PHP 非常相似。

数组允许我们将相关的值保存在一个单独的变量中。例如，假设我们想要保存我兄弟姐妹的年龄。我们可以创建一个表示我兄弟姐妹年龄的整数数组。让我们在 PHP 和 Ruby 中这样做，并比较两种语法。这将 PHP 语法：

```php
$siblings_ages = [ 42, 31, 25 ];
```

而这将 Ruby 语法：

```php
siblings_ages = [ 42, 31, 25 ]
```

除了`$`和`;`之外，两段代码是相同的；这表明，如果你有 PHP 背景，理解 Ruby 数组的概念不应该很难。虽然还有其他声明数组的方法，但我们现在将保持语法简单。

现在我们已经了解了如何声明数组，让我们看看它们的实际用途。

Ruby 中的数组可以包含其他类型的值（不仅仅是数字）。假设我想列出某人会演奏的乐器。我们可以创建一个字符串列表并命名为`instruments_played`：

```php
instruments_played = ["guitar", "drums", "bass", "ukulele"]
```

如您所见，我们将相关的值（在这种情况下是乐器）组合在一个单独的变量中。我们创建了一个列表，可以在我们的代码中使用。这种类型的数组，正如它所声明的，有一个内部计数器来引用每个值。这个内部计数器从 0 开始，所以第一个值（`guitar`）将包含在`instruments_played[0]`中，第二个值（`drums`）将包含在`instruments_played[1]`中，以此类推。

如果我们想在屏幕上打印出所有的乐器，我们可以逐个打印每个乐器：

```php
puts instruments_played[0]
puts instruments_played[1]
puts instruments_played[2]
puts instruments_played[3]
```

然而，这样做既繁琐又不实用，正如你可能已经猜到的，我们有一个更好的编程方式来做这件事。我们不是逐个计数，而是可以使用 `do` 语句遍历数组的所有值：

```php
for i in 0…4 do
puts instruments_played[i]
end
```

三点符号 (`…`) 可能对来自 PHP 的人来说既新又奇怪，但如果你只是阅读代码，它几乎是有道理的。这种符号被称为范围，你会在 Ruby 中经常看到它的使用。这个符号创建了一个计数器，从 0 开始，到小于 4 的值（在这个例子中是 3），然后每次增加 1。然后这个计数器被分配给 `i` 变量。这个例子可以读作，“对于 `i` 变量，创建一个从 0 开始，到 3 结束，每次增加 1，并逐个打印数组值的循环。” 输出将看起来像这样：

```php
guitar
drums
bass
ukulele
=> 0...4
```

你注意到 `…` 符号排除了数字 4，相当于数学符号中的 [0,4] 吗？那么，如果我们想使范围包含最后一个数字——例如，[0,3]——会发生什么呢？Ruby 也有一个两点符号 (`..`)，它确实包含其范围内的最后一个数字。因此，我们可以将示例重写如下：

```php
for i in 0..3 do
puts instruments_played[i]
end
```

输出将与上一个例子相同：

```php
guitar
drums
bass
ukulele
=> 0..3
```

你可以根据需要使用三点或两点符号。如果你对范围的主题更感兴趣，我建议你查看有关此主题的文档：[`ruby-doc.org/core-2.5.1/Range.html`](https://ruby-doc.org/core-2.5.1/Range.html)。

现在，让我们回到数组。就像 PHP 一样，Ruby 也有一些内部方法来处理数组。作为一个并行设计的例子，PHP 有一个函数可以告诉我们数组的大小。这个函数叫做 `count()`，在 Ruby 中有一个等效的函数叫做 `size()`。只需记住，Ruby 中的所有东西都是对象，所以你不会像在 PHP 中那样使用 `size(instruments_played)`。相反，为了打印我们数组中的元素数量，我们会调用 `size()` 方法作为 `instruments_played` 数组的方法：

```php
puts instruments_played.size
```

由于数组有四个元素，我们会得到一个输出为 `4`。此外，还有一个做同样事情的方法叫做 `length`。

我发现另外两个内部方法极其有用，它们是 `first` 和 `last`。这些方法（如我们从它们的名字中推断出的）允许我们分别获取数组的第一个和最后一个元素。让我们尝试使用一些变量插值来使用 `first` 方法：

```php
puts "I learned how to play the #{instruments_played.first} first"
```

这将输出以下内容：

```php
I learned how to play the guitar first
```

`last` 方法以相同的方式工作：

```php
puts "I learned how to play the #{instruments_played.last} last"
```

这将按预期输出：

```php
I learned how to play the ukulele last
```

如你所见，我们已经将字符串和变量的内容结合起来创建了一个新的字符串。这种组合被称为变量插值。

### 变量插值

变量插值（这是一个你在编程中会经常听到的术语）涉及用变量的值替换变量。当打印消息和/或向用户展示数据时，这非常实用。当正确使用时，变量插值允许我们在字符串中嵌入变量值。让我们看看我们之前的示例中的代码：

```php
instruments_played = ["guitar", "drums", "bass", "ukulele"]
puts "I learned how to play the #{instruments_played.first} first"
```

字符串插值功能有几个层次，所以让我们分部分分析它。

首先，在代码的第二行，我们可以看到，在字符串内部，我们添加了一个特殊的块，它以`#`符号开头，后跟一系列花括号（`#{ }`）。当在字符串中使用时，这个块确定我们将使用插值。

其次，花括号内的所有内容都将被解释并返回。在这个例子中，花括号内的代码是`instruments_played.last`，它包含数组的最后一个元素。这个数组的最后一个元素将作为字符串的一部分返回，从而完成插值。这仅当字符串用双引号定义时才有效。

### 组合数组类型

到目前为止，我们看到了包含相同类型数据的数组——我们有一个仅由字符串组成的数组，另一个仅由整数组成的数组。但关于 Ruby 数组，还有一个值得注意的最后一个特性，即它们可以在同一个数组中组合不同的变量类型。作为一个随机示例，让我们向一个数组添加一些无关的值：

```php
random_values = [25, "drums", false, 3.8]
```

在这个数组中，我们正在将不同类型的数据（整数、字符串、布尔值和浮点数）组合成一个单一数组。并非所有语言都支持数组内的这种行为，但 Ruby 和 PHP 都支持。

正如我们在开始查看变量类型时提到的，Ruby 是动态类型的。动态类型语言的一个特性是数组可以组合它们所持有的数据类型。相比之下，在强类型语言（如 Java）中，数组被迫在每个元素上具有相同类型的数据——也就是说，你只能有一个整数数组或只能有一个字符串数组。这并不是说 Ruby 比 Java 好，或者一般来说，强类型语言比动态类型语言好。它们只是有不同的设计。

如果你对这个主题的数组感兴趣，请查看官方 Ruby 文档：[`ruby-doc.org/core-2.7.0/Array.html`](https://ruby-doc.org/core-2.7.0/Array.html)。

## Hashes

现在，让我们看看另一种 PHP 开发者也会很容易理解的变量类型：hashes。hash 是一个数组，但主要区别在于它有文本索引而不是数字索引。hashes 在行为上与数组非常相似，但不同之处在于我们使用字符串来引用某些值。在 PHP 中，这些被称为关联数组。

让我们看看一个实际例子。这里，我们有一个索引为英语、值为西班牙语的 hash：

```php
numbers = { "one" => "uno", "two" => "dos", "three" => "tres" }
```

类似于我们可以用数组做的，要获取单个索引的值，我们可以输入以下内容：

```php
puts numbers["one"]
```

我们会得到以下输出：

```php
uno
```

如您所见，索引是一个字符串，对人类来说更易读。正是在这种可读性上，哈希表可以派上用场。让我们重写我们在*探索变量类型*部分开始时使用的第一个例子，使其使用哈希表：

```php
person = { "name" => "Oscar", "age" => 35, "is_married" => true, "books_read_this_week" => 2.5 }
```

我们不需要为`name`、`age`、`is_married`和`books_read_this_week`分别设置单独的变量，我们可以用一个哈希表将这些值组合成一个名为`person`的单个变量。现在，我们可以像以下这样引用每个索引：

```php
person["name"]
person["age"]
person["is_married"]
person["books_read_this_week"]
```

此外，我们还可以使用以下方式打印一个非常易读的消息：

```php
puts "#{person["name"]} is #{person["age"]} years old"
```

即使对于一个刚开始学习 Ruby 的开发者来说，这不仅是可读的，而且从代码的意图来看也是可理解的。正如预期的那样，它会输出以下内容：

```php
Oscar is 35 years old
```

当处理需要由人类读取的映射数据时，哈希表非常有用。当你处理正在变化的数据时，它也很有用。

这把我们带到了我们将要看到的最后一种变量类型，我必须说它比我希望的要复杂。我们迄今为止看到的变量类型是可变的，这意味着它们可以被更改。然而，有时我们不需要某些值改变——我们需要一个存储这个值的位置。符号正是这样做的。

## 符号

符号是高度优化的标识符，它们将不可变的字符串映射到固定的内部值。它们本身也是不可变的字符串——也就是说，它们的值不会改变。

这个概念有点复杂，但我相信通过一个例子会更容易理解。让我们用一个简单的字符串来看看它的值指向什么。所以，运行以下代码：

```php
"name".object_id
```

当你创建一个字符串时，Ruby 会将在内存中某个地方保存字符串对象。`object_id`方法保存这个内部唯一标识符。注意当你多次调用同一行时会发生什么：

```php
irb(main):126:0> "name".object_id
=> 2391267760
irb(main):127:0> "name".object_id
=> 2391332180
irb(main):128:0> "name".object_id
=> 2391359800
```

第一个字符串指向的地址与第二个和第三个字符串不同。所以，每次我们输入`name`字符串时，Ruby 都会创建并存储一个新的字符串。即使它们的值相同，它们仍然是不同的。作为一个类比，这就像有相同名称但位于不同文件夹的文件。即使文件有相同的内容，它们仍然是不同的文件。

这与符号不同。符号指的是相同的内存位置。让我们用符号尝试相同的例子：

```php
:name.object_id
```

这也应该返回一个唯一的标识符号码。然而，当我们多次调用相同的代码时会发生什么呢？

```php
Irb(main):129:0> :name.object_id
=> 88028
irb(main):130:0> :name.object_id
=> 88028
irb(main):131:0> :name.object_id
=> 88028
```

与返回不同的随机数不同，这次我们得到的是相同的（尽管仍然是随机的）数字。这是因为每次我们调用 `:name` 时，Ruby 解释器都在查看内存中的同一位置。使用相同的类比，这就像创建一个唯一的文件，然后每次我们需要该文件时，我们都会创建指向原始文件的链接。所以，即使链接在不同的文件夹中，它们也会指向同一个文件。

我们现在不会深入探讨这个主题，因为只需要理解其基本知识就足够了，但我们在未来的章节中会看到更多示例，特别是 Ruby on Rails 的示例。目前，只需记住这个好的经验法则：当对象的身份很重要时使用符号。如果内容更重要，则使用字符串。

如果你现在想了解更多关于符号的信息，以下网站是一个很好的资源：[`medium.com/@lcriswell/ruby-symbols-vs-strings-248842529fd9`](https://medium.com/@lcriswell/ruby-symbols-vs-strings-248842529fd9)。

到目前为止，我们已经学习了所有关于变量的知识。但如果我们不能对变量做出决策，变量有什么用呢？这是我们下一个话题。

# 使用条件语句

现在我们已经知道了在 Ruby 中可以使用哪些类型的变量，让我们给这些变量一些更实用的用途。

## if 语句

到现在为止，我们都应该熟悉 `if` 语句及其结构：如果句子为真，代码应该执行或返回某些内容。

让我们以前面章节中使用的 person 哈希作为基础：

```php
person = { "name" => "Oscar", "age" => 35, "is_married" => true, "books_read_this_week" => 2.5 }
```

使用它，我们可以创建一个基本的 `if` 语句：

```php
if person["is_married"] == true
  puts "Person is married"
end
```

这基本上是自我解释的。这会读作：“如果 `person["married"]` 中的值等于 true，那么打印 `Person is married`。” `end` 关键字限制了 `if` 语句何时完成——也就是说，`end` 关键字之后的内容不是代码块的一部分。你会在 Ruby 中经常看到 `end` 关键字——只需记住，它用于界定特定的代码块。

虽然前面的代码很有用，但还有更好的写法——即“Ruby 方式”。首先，我们移除 `== true`，如果我们只打算执行一个动作，我们可以将其写在一行中：

```php
puts "Person is married" if person["is_married"]
```

这读起来就像一个句子，你会在很多 Ruby 程序员中使用这个有用的单行代码。

## if-else 语句

如果在值不为 true 的情况下需要不同的操作，那么你应该使用 `if-else` 结构：

```php
if person["age"] < 31
  puts "This person is under 30"
else
  puts "This person is over 30"
end
```

这将输出以下内容：

```php
This person is over 30
```

只需记住，`if` 语句评估条件。如果条件为真，Ruby 执行下一行的代码。然而，如果条件为假，Ruby 跳过第一个代码块，转到 `else` 语句，并执行 `else` 关键字之后的代码。

## 三元运算符

作为 `if` 语句的最后一个例子，我们还有三元运算符，它在其他编程语言中也有；尽管它不是那么易读，但仍然很有用。让我们看一个例子：

```php
over_or_under = person["age"] > 31 ? "over" : "under"
puts "This person is #{over_or_under} 30"
```

使用三元运算符时，将在`=`符号和`?`符号之间的条件进行评估。如果条件被认为是真的，则返回`:`符号左侧的值。如果条件被认为是假的，则返回`:`符号右侧的值。在这种情况下，存储在`person["age"]`中的值是 35。由于 35 大于 30，`over`字符串将被存储在`over_or_under`变量中。第二行代码将简单地插值这个值并应该返回以下内容：

```php
This person is over 30
```

虽然这不如之前的`if`语句可读，但代码仍然是有效的，并且在大多数编程语言中都是可用的。三元运算符在 PHP 中也是一样的，当你需要存储一个依赖于条件的值时很有用。

`if`语句可能是编程中最常用的资源之一，因此了解其语法、用法以及它解决的不同用例是个好主意。现在，让我们看看另一个使用真/假值来运行代码的资源。

# 使用循环重复代码

我们来到了下一个主题，即循环。Ruby 与其他语言一样，有不同的方式来重复执行相同的代码。当我们讨论数组时，特别是包含乐器名称的数组，我们看到了一个`for`循环的例子，它被用来打印数组中包含的每个乐器。但让我们看看另一种更常用的循环类型：`while`循环。

`while`循环让我们可以重复执行由真/假条件确定的代码。假设我们想要打印一个从一到三的数字。我们可以创建一个`print`语句，并简单地重复它三次，同时增加值。但是，让我们尝试一种更简洁的方法。首先创建一个`counter`变量：

```php
counter = 1
```

现在，我们可以开始`while`循环周期：

```php
while counter <= 3
  puts counter
  counter++
end
```

这可能看起来像是有效的代码，但我们将从 Ruby 解释器得到一个错误：

```php
Traceback (most recent call last):
        3: from /usr/bin/irb:23:in `<main>'
        2: from /usr/bin/irb:23:in `load'
        1: from /Library/Ruby/Gems/2.6.0/gems/irb-1.0.0/exe/irb:11:in `<top (required)>'
SyntaxError ((irb):205: syntax error, unexpected end)
```

这是因为大多数 Ruby 新手都会犯的一个常见错误假设。如果你使用 PHP 或 JavaScript，你会习惯于`++`运算符，它相当于给变量加 1。这就像写以下内容：

```php
counter = counter + 1
```

然而，Ruby 中不存在`++`运算符（这也适用于`––`运算符，它将值减 1）。因此，我们不得不用重写我们的代码，使其看起来像这样：

```php
while counter <= 3
  puts counter
  counter += 1
end
```

这将输出以下内容：

```php
1
2
3
=> nil
```

`while`语句的结构与`if`语句非常相似，但它不是只执行一行代码，而是评估条件，如果条件仍然满足，则执行代码。

在这种情况下，在进入循环之前，`counter` 变量的值为 `1` - 因为继续循环的条件是值小于或等于 `3`，条件得到了满足。由于条件得到了满足，Ruby 将执行 `end` 关键字之前的代码，所以将打印数字 `1` 并将 `1` 添加到 `counter` 变量中。由于这是一个循环，Ruby 将再次读取条件，但这次 `counter` 的值为 `2`，所以它将再次执行整个块。一旦 `counter` 的值达到 `4`，Ruby 将确定 `while` 条件不再满足，并且将中断循环，而不再执行其内部的代码。

当我们与数组一起工作时，循环的一个更有用的例子是。我们已经看到了使用计数器遍历数组的一种方法，但我们还有一个名为 `each` 的方法来遍历数组的每个元素。让我们再次使用 `instruments_played` 数组：

```php
instruments_played = ["guitar", "drums", "bass", "ukulele"]
```

我们可以使用 `each` 来遍历每个元素，如下所示：

```php
instruments_played.each do |instrument|
puts instrument
end
```

这段代码将遍历数组，这样我们就不必为该数组的每个元素重复代码。这正是循环的作用：编写更少的代码。对于数组中的每个元素，Ruby 将打印该元素的值（我们为了可读性将其称为 `instrument`，并且仅在这个示例中如此，但我们可以将其命名为任何我们想要的）。

此外，我们可以通过使用花括号将此代码压缩成一行，如下所示：

```php
instruments_played.each { |instrument| puts instrument  }
```

这将产生与上一个示例相同的输出，但如您所见，它更简洁且易于阅读。此外，`each` 循环帮助我们编写能够适应内容大小的代码。如果我们向数组中添加另一个元素，我们就不必修改循环中的任何内容来打印添加的乐器。循环将自动完成此操作。

最后，当我们想要访问数组的索引时会发生什么？我们有一个专门为此而存在的方法。`each_with_index` 方法将使数组的索引可用，正如您在这个示例中可以看到的那样：

```php
instruments_played.each_with_index do |instrument, index|
puts "#{index}: #{instrument}"
end
```

此代码将输出以下内容：

```php
0: guitar
1: drums
2: bass
3: ukulele
=> ["guitar", "drums", "bass", "ukulele"]
```

再次强调，`instrument` 和 `index` 都是别名 - 我们可以将其命名为任何我们想要的 - 但我们键入它们的顺序将决定哪个值将被存储在其中。数组元素的值将存储在第一个变量（`instrument`）中，而数组计数器将存储在第二个变量（`index`）中。

我们完全可以像这样重写示例，并且仍然得到相同的输出：

```php
instruments_played.each_with_index do |array_element, array_index|
puts "#{array_index}: #{array_element}"
end
```

新的代码将产生与之前相同的输出，但这次我们将 `instrument` 和其 `index` 重命名为 `array_element` 和 `array_index`，这仅仅是我个人为了使代码对我更有意义而做出的选择。这表明，作为程序员，我们决定如何命名变量以保护可读性（请相信我 - 随着你作为程序员的成长，你将花更多的时间尝试命名变量）。

到目前为止，我们知道如何通过使用循环和遍历数组来重复代码。我们不是多次编写相同的代码，而是利用 Ruby 的`while`语句和`each`方法来提高代码的效率和可读性。但我们还没有完成。Ruby 还有一些其他的技巧来进一步提高可读性。我们将在下一节中查看这些技巧。

# 使用 Ruby 语言增强功能

对于我们大多数开发者来说，我们应该始终努力提高代码的可读性，因为这将在长远上帮助到每个人。我遇到过一些情况，当我回顾自己的代码时，很难理解代码在做什么。这意味着我的代码写得不好。想象一下，这种写得不好的代码可能会对下一个使用它或，更糟糕的是，改进它的开发者或团队造成多大的负担。相比之下，如果我的代码写得很好，我们就不会遇到这个问题。我想说的是：请编写可读的代码，我无法强调得更多，Ruby 开发者为了使代码可读而付出的努力，超过了对代码的任何其他增强。Ruby 附带了一些额外的工具来实现这一点。

## `unless`语句

这些选项之一是一个名为`unless`语句的语言增强功能。`unless`语句是一个否定`if`语句——也就是说，它只会在条件不满足时执行代码。让我们看看它在示例中的使用情况。

让我们假设以下场景：我们有一个针对未婚个人的产品。为了简单起见，如果这个人是未婚的，我们将只打印出“单身促销”的消息。让我们尝试编写这段代码。让我们以我们之前的人的详细信息哈希为例：

```php
person = { "name" => "Oscar", "age" => 35, "is_married" => true, "books_read_this_week" => 2.5 }
```

现在，让我们将`is_married`的值改为`false`：

```php
person = { "name" => "Oscar", "age" => 35, "is_married" => false, "books_read_this_week" => 2.5 }
```

一旦我们声明了这个哈希，我们就可以尝试打印一条消息，如果这个人是单身的话：

```php
puts "Promo for singles" if person["is_married"] == false
```

因为这个人没有结婚，所以输出如下：

```php
Promo for singles
```

虽然代码能正常工作，但看起来并不好。我们可以使用感叹号(`!`)运算符将布尔值从`true`反转到`false`：

```php
puts "Show promotion" if !person["is_married"]
```

虽然这段代码仍然能工作，但看起来仍然不好。让我们看看 Ruby 有哪些选项可以解决这个问题。

在大多数编程语言中，你会看到很多读起来像“if not”的句子。当然，这很难读，也违反了 Ruby 的可读性原则。为了解决这个问题，Ruby 的创造者添加了确切的句子，使其更具可读性：`unless`。它的工作方式与`if`语句类似，但会在条件被认为是`false`时执行代码。

在这种情况下，当我们需要执行的代码只有当这个人没有结婚时，这很有帮助。所以，我们不是写一个否定条件语句（如果一个人是*不*结婚的），`if !person["is_married"]`，我们可以将示例重写如下（除非一个人结婚）：

```php
unless person["is_married"]
  puts "Show promotion"
end
```

看起来已经好多了，但就像`if`语句一样，我们可以将其转换为单行代码：

```php
puts "Show promotion" unless person["is_married"]
```

这是一个非常 Ruby 风格的句子，读起来就像它的行为一样：“除非这个人已婚，否则打印“Show promotion”。”这几乎是可读性的极致。

`unless`语句非常有用，以至于今天最常用的 PHP 框架之一，名为 Laravel，已经以指令的形式借用了这个功能：[`laravel.com/docs/9.x/blade#if-statements`](https://laravel.com/docs/9.x/blade#if-statements)。

## `until`循环

就像`unless`语句一样，`until`循环解决了与负条件相同的问题。

与其写“while not”，这听起来很糟糕，`until`语句接受一个假命题，并在条件变为真之前执行循环。

`until`语句接受一个假命题，并在条件变为真之前执行循环。让我们再次看看我们之前的`while`示例：

```php
counter = 1
while counter <= 3
  puts counter
  counter += 1
end
```

使用`until`，我们可以将其重写如下：

```php
counter = 1
until counter > 3
  puts counter
  counter += 1
end
```

输出与`while`语句相同：

```php
1
2
3
=> nil
```

我们的代码将读作“打印计数器，直到计数器大于 3。”你选择使用`while not`语句还是`until`语句将取决于你，因为它们都看起来很易读，但 Ruby 提供了这两种语句的事实告诉我们，Ruby 的设计是为了可读性，而不仅仅是编程。

## 自动返回

当你在 IRB（交互式 Ruby 解释器）中工作时，你可能已经注意到，每当你输入变量时，IRB 都会输出你刚刚输入的值。即使在我们最后的`until`语句示例中，shell 首先输出了三个数字，然后是一个最终的`=> nil`值。如果你仔细观察其他示例，你会发现类似的行为。这是因为 Ruby 总是尝试返回一个值——无论是声明、方法还是字符串，Ruby 都会尝试自动返回一个值。

如果你还不信，让我们使用 IRB 来更明确地看到它。所以，输入以下内容：

```php
"This is a string"
```

我们没有声明字符串或将其赋值给一个值；我们只是在 IRB shell 中输入了一个字符串。那么 shell 会做什么呢？它会返回我们刚刚输入的值：

```php
=> "this is a string"
```

来自 PHP 背景（以及其他语言），理解“自动返回”功能对于理解更复杂的 Ruby 代码至关重要。重要的是要知道 PHP（以及大多数语言）不会这样做。在 PHP（以及其他语言）中，我们需要显式地返回值，而 Ruby 默认就会这样做。在 PHP 中，这是通过使用`return`语句来实现的。话虽如此，有时你会在 Ruby 代码中遇到显式的`return`语句，因为它有时会增加可读性。为了进一步理解这个特性，在接下来的几个示例中，我们将退出 IRB，继续通过创建源代码文件并运行 Ruby 来执行它们。你仍然可以在 IRB 中跟随下一个示例，但我强烈建议你用源代码文件而不是 IRB 来跟随它们。

假设我们想要创建一个在屏幕上打印消息的方法。我们可以通过创建一个名为 `methods.rb` 的文件来实现。这个文件将包含以下代码：

```php
# methods.rb
def message()
  return "This is a message"
end
```

现在，我们定义一个名为 `message` 的方法，它返回一个字符串。在 Ruby 中，我们使用 `def` 保留关键字来定义一个方法，并用 `end` 关键字来限制定义。

现在，让我们添加另一个名为 `say()` 的方法，并在该方法内部调用 `message()` 方法：

```php
 # methods.rb
...
def say()
  message()
end
```

到目前为止，我们并没有做任何特别的事情——我们只是有一个方法调用了另一个方法。如果我们打开一个 shell 并执行这个脚本，它看起来就像什么都没做：

```php
ruby methods.rb
```

这个脚本什么都没有输出，但在幕后，它现在有两个定义好的方法，并且可以随时使用。`message` 方法由于使用了 `return` 关键字而显式（不是自动）地返回一个字符串。这段代码看起来仍然很熟悉，但不会持续太久。

现在，让我们打印出 `say()` 方法的这个最后一条代码的内容：

```php
# methods.rb
...
def say()
  return message()
end
puts say()
```

如果我们再次运行它，我们将在屏幕上看到以下消息：

```php
This is a message
```

这就是 Ruby 与 PHP 表现不同的地方。虽然你可以在 Ruby 中显式使用 `return` 函数，但 Ruby 不需要 `return` 语句，因为它已经自动作为其默认行为的一部分执行了。所以，让我们通过从 `message()` 和 `say()` 方法中移除 `return` 语句来试一试。你的最终代码应该看起来像这样：

```php
# methods.rb
def message()
  "This is a message"
end
def say()
  message()
end
puts say()
```

诚然，这看起来很奇怪，尤其是对于有 PHP 背景的人来说。我的建议是，你只需尝试适应这种语法。你会在 Ruby 中非常频繁地看到它。为了更容易学习这个规则，我们可以概括地说，“Ruby 中的每一句话都会返回一个值。”有一些明显的例外，但对于所有 Ruby 语句来说，这是真的。

## 可选的括号

另一个奇怪但有用的语法增强是，Ruby 方法的括号完全是可选的——因此，你可以选择是否包含括号。并且就像我们迄今为止所学的每一个 Ruby 资源一样，我们应该尝试使用它来使我们的代码更容易阅读，但也应该尽量避免过度使用。过度使用这个特性可能会让我们把代码格式化成这样：

```php
method1 method2 parmeter1, parameter2
```

这个片段的问题是我们不知道逗号是用来分隔 `method1` 或 `method2` 的参数的。在这种情况下，我们应该使用括号：

```php
method1( method2( parameter1, parameter2 )
```

现在，很明显 `method2` 接收了两个参数，而 `method1` 只接收一个参数。让我们通过从之前的例子中移除括号来查看一个更简化的例子。我们的例子现在看起来是这样的：

```php
# methods.rb
def message
  "This is a message"
end
def say
  puts message
end
say
```

输出与之前相同，但现在，代码看起来更像正确的句子而不是代码语法。你将看到很多类似的代码，尤其是在你开始使用 Ruby on Rails 时。由于缺少括号，Ruby 允许我们有一个方法和一个具有相同名称的变量。在这种情况下，我们有一个名为 `message` 的方法和一个名为 `message` 的变量。

这种情况，如果未进行解释，可能会在以后导致很多困惑。为了达到这个效果，让我们拿我们之前的例子稍作修改，以便更好地理解这种命名行为。首先，我们将向 `say()` 方法添加一个参数，使得打印的消息是动态的。这个参数将被命名为 `message`：

```php
# methods.rb
...
def say message
  puts message
end
say "Now we can say anything"
```

在其他编程语言中，如果我们尝试运行这段代码，我们预期会出错，这就是为什么当你刚开始使用 Ruby 时，它有时会让人感到不知所措。我们故意将参数命名为 `message`，这意味着我们现在有一个 `message` 方法和一个 `message` 本地变量。当我们到达 `puts message` 这一行时，我们不确定我们是调用带括号的 `message` 参数还是不带括号的 `message` 方法。不幸的是，这种混淆经常发生，至少在我开始更专业地使用 Ruby 时是这样的经验。

因此，我的建议是，在调用方法时尽量使用括号，即使语法不需要我们这样做。出于教学目的，我们在这个例子中不会这样做。所以，我们的最终源代码应该看起来像这样：

```php
# methods.rb
def message
  return "This is a message"
end
def say message
  puts message
end
say "Now we can say anything"
```

正如你所预期（或不是），当在壳上执行脚本时，我们会得到以下输出：

```php
Now we can say anything
```

为什么它打印了那个字符串而不是 `"This is a message"` 字符串呢？嗯，这是因为 `message` 变量优先于 `message` 方法。

虽然这个特性可能看起来不太美观（我并不太喜欢它），但我保证你时不时地会遇到它，你应该为此做好准备。

## 有疑问的感叹号方法名称

作为 Ruby 语言增强的甜点，其创造者也包含了一个命名特性，以增加我们方法的可读性：感叹号（`!`）（也称为感叹号）和问号（`?`）。它们在行为上没有任何改变，但允许一行代码读作一个问题或感叹。用感叹号命名的方法被称为危险方法，因为它们会修改它们被调用的对象。用问号命名的方法被称为谓词方法，并且按照惯例，返回一个布尔值。

要看到这个功能在实际中的运用，我们将使用带问号的创建一个方法。让我们创建一个名为 `enhanced_naming.rb` 的新文件，并添加以下代码：

```php
# enhanced_naming.rb
$married_status = false
def is_married
  $married_status
end
```

`$married_status`变量是一个全局变量，这仅仅意味着我们可以在方法内或方法外修改或访问其内容。在这种情况下，我们定义了一个获取`$married_status`值的方法。然而，知道我们可以将`?`添加到这个方法的名称中，让我们将`is_married`方法重命名为如下：

```php
# enhanced_naming.rb
$married_status = false
def is_married?
  $married_status
end
```

现在，让我们使用一个已经熟悉的单行代码来打印已婚人的消息：

```php
puts "Promo for married people" if is_married?
```

虽然将`?`添加到方法名称中不会影响其行为，但它确实将句子变成了一个明显的问题。我们将在 Ruby 中非常频繁地看到这种语法。

同样地，我们可以在方法名称中使用感叹号（`!`）作为一部分。再次强调，仅仅将感叹号添加到名称中本身并不会影响行为，但它告诉阅读代码的人我们正在做不同于仅仅调用方法的事情。作为一个例子，让我们将我们的`marry`方法重命名为`marry!`并看看它看起来像什么：

```php
def marry!
  $married_status = true
end
```

作为 Ruby 社区采用的一种约定，感叹号（`!`）将告诉代码的读者我们正在对象内部进行更改。没有感叹号的方法将简单地返回一个值，但不会影响对象本身。因此，在这种情况下，我们正在将`$married_status`的值更改为`true`。这就是代码现在的样子：

```php
# enhanced_naming.rb
$married_status = false
def is_married?
  $married_status
end
def marry!
  $married_status = true
end
puts "Promo for married people" if is_married?
```

很遗憾，当我们运行这个例子时，我们没有看到任何输出。这是因为全局变量`$married_status`的初始值是`false`，并且我们的代码只有在值是`true`时才会打印一条消息。现在，让我们调用`marry!`方法，并在代码末尾再次复制单行代码：

```php
puts "Promo for married people" if is_married?
marry!
puts "Promo for married people" if is_married?
```

现在，我们可以再次运行代码：

```php
ruby enhanced.rb
```

输出将看起来像这样：

```php
Promo for married people
```

这里发生了什么？我们有一个初始值为`false`的全局变量`$married_status`。然后，我们有两个方法——一个用于获取`$married_status`的值，另一个将其更改为`true`。最后，我们尝试打印消息，但由于初始值是`false`，消息没有被打印出来。通过调用`marry!`方法，我们将`$married_status`更改为`true`，这使得脚本的最后一条打印出消息。

Ruby 通过提高代码可读性来为编程带来语言增强。我见过写得如此漂亮的代码，这强化了不写代码注释而是让代码为自己说话的想法。一旦你开始经常使用它们，你会越来越欣赏它们，并希望所有语言都有这些增强。

# 摘要

在本章中，我们学习了如何使用 Ruby 二进制文件编写、执行和 require 脚本，以及如何使用 IRB 在命令行上直接执行 Ruby 代码，而无需编写源代码。

此外，我们还回顾了 Ruby 编写变量的语法、`if`语句的语法以及如何遍历循环和数组。最后，我们学习了 Ruby 的一些语言增强，而 PHP 没有，这样我们就可以阅读和理解更复杂的 Ruby 代码。

现在，我们已经准备好编写 Ruby 代码来解决现实生活中的例子了。我们将在下一章开始这样做。
