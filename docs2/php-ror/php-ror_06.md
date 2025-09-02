

# Ruby 的调试

就像任何其他重要的编程语言一样，Ruby 有许多工具可以帮助我们分析、修复和改进我们的代码。根据多年前我遇到的一位软件架构师的说法，拥有调试技能的程序员与缺乏调试技能的程序员之间的差距，相当于初级程序员与高级程序员之间的区别。通过使用正确的工具来调试我们的代码，我们不仅可以改进我们的代码，还可以提高我们对语言如何被解释的理解，从长远来看，这对我们作为开发者的道路是有益的。让我们看看 Ruby 自带的一些工具，以及 Ruby 社区开发的用于调试代码的 gem。

考虑到调试，在本章中，我们将涵盖以下主题：

+   Ruby 与 PHP 中的调试函数

+   调试 gem

+   理解**交互式 Ruby** (IRB)的有用性

# 技术要求

要跟随本章内容，你需要以下内容：

+   任何 IDE 来查看/编辑代码（例如，Sublime Text，Visual Studio Code，Notepad++，Vim，Emacs 等）

+   对于 macOS 用户，你还需要安装 Xcode 命令行工具

+   已安装并准备好使用的 Ruby 版本 2.6 或更高版本

本章中展示的代码可在[`github.com/PacktPublishing/From-PHP-to-Ruby-on-Rails/`](https://github.com/PacktPublishing/From-PHP-to-Ruby-on-Rails/)找到。

# Ruby 与 PHP 中的调试函数

到目前为止，我们已经编写了确保代码每次都能正确运行的脚本和代码片段。然而，在现实世界中，我们会接触到别人编写的代码，这些代码可能没有经过测试，或者没有在之前未曾出现过的场景下进行过测试。这种情况很常见，我们应该准备好动手解决这些问题。在 PHP 中，我们有几个函数可以帮助我们以最简单的方式调试。你可以只阅读以下示例，不必跟着操作。现在，让我们看看 PHP 的`var_dump()`函数。我们可以打开命令行 shell 并创建一个包含以下内容的文件：

```php
<?php //buggy_code.php
$person['firSt'] = 'Thomas';
$person['last'] = 'Anderson';
echo "Hi {$person['first']} {$person['last']}";
```

假设我们在 shell 上运行了以下 PHP 脚本：

```php
php buggy_code.php
```

应该输出类似以下内容：

```php
PHP Warning:  Undefined array key "first"…
Warning: Undefined array key "first" in…
Hi  Anderson
```

初看时，我们可能看不到代码中的错误。如果你一眼就看到了错误，恭喜你，你有一个良好的开端。但对于那些一眼没看到错误的人来说，让我们使用`var_dump()`函数。让我们注释掉最后一行，并将其添加到代码的结尾行：

```php
<?php //buggy_code.php
…
//echo "Hi {$person['first']} {$person['last']}";
var_dump($person);
```

然后让我们再次从我们的 shell 中运行这个脚本：

```php
php buggy_code.php
```

这应该会输出以下内容：

```php
…
array(2) {
  ["firSt"]=>
  string(6) "Thomas"
  ["last"]=>
  string(8) "Anderson"
}
```

在 PHP 中，`var_dump()`函数非常有用，因为它输出变量、其内容及其内容类型。注意，在这种情况下，`var_dump($person)`告诉我们我们有一个数组，并且这个数组有两个元素。这两个元素都是字符串。我们还注意到数组中的第一个关联元素（`["firSt"]`）看起来很奇怪。它在中间有一个大写的“S”。所以，让我们进行更正并移除我们的调试代码。我们的代码现在应该看起来像这样：

```php
<?php //buggy_code.php
$person['first'] = 'Thomas';
$person['last'] = 'Anderson';
echo "Hi {$person['first']} {$person['last']}";
```

我们在 shell 上再次运行它：

```php
php buggy_code.php
```

它返回以下输出：

```php
Hi Thomas Anderson
```

虽然这可能不是最好的调试工具，但它确实是一种非常直接的方式来调试 PHP 中的代码。我个人非常喜欢这个函数，因为它是我用来调试人生中第一个函数。此外，这是一个过于简化的例子，但本质上描绘了真实场景，尤其是在进行网页开发调试时，有时你不知道什么被发送到你的脚本中。如果你对这个函数更感兴趣，你应该查看该函数的文档页面：[`www.php.net/manual/en/function.var-dump.php`](https://www.php.net/manual/en/function.var-dump.php)。

你可能还想看看`print_r()`和`var_export()`函数。

在 Ruby 中，我们实际上没有`var_dump()`函数，而是每个对象都有一个名为`inspect()`的方法。`inspect()`方法将对象的内容转换为字符串并返回它。让我们用 Ruby 重做相同的例子。让我们创建一个名为`buggy_code.rb`的文件，并添加以下代码：

```php
# buggy_code.rb
person = Hash.new
person['firSt'] = 'Thomas'
person['last'] = 'Anderson'
print "Hi #{person['first']} #{person['last']}"
```

假设我们将在 shell 上使用以下内容运行此脚本：

```php
ruby buggy_code.rb
```

我们将得到以下输出：

```php
Hi  Anderson
```

我们事先就知道错误是什么，但为了教育目的，让我们通过在我们的代码中添加以下内容来调试`person`对象，使其现在看起来像这样：

```php
# buggy_code.rb
person = Hash.new
person['firSt'] = 'Thomas'
person['last'] = 'Anderson'
#print "Hi #{person['first']} #{person['last']}"
print person.inspect
```

我们使用以下命令在 shell 上运行此脚本：

```php
ruby buggy_code.rb
```

这将返回以下结果：

```php
{"firSt"=>"Thomas", "last"=>"Anderson"}
```

虽然我们可以通过仅使用`print()`函数来实现相同的效果，但这只适用于我们在这个例子中分析哈希的内容。如果前面的例子有一个类对象，`print()`函数将不会显示对象的所有内容。`inspect()`方法更通用，因为它输出任何类型对象的内容。

由于我们已经知道解决方案，我们可以直接修复示例。我们的代码现在应该看起来像这样：

```php
# buggy_code.rb
person = Hash.new
person['first'] = 'Thomas'
person['last'] = 'Anderson'
print "Hi #{person['first']} #{person['last']}"
```

最后，我们可以再次运行它，以验证我们的脚本是否正确运行，通过在 shell 上运行以下命令：

```php
ruby buggy_code.rb
```

这将返回以下结果：

```php
Hi Thomas Anderson
```

通过这种方式，我们已经学习了一种非常 PHP 风格的调试脚本方法。幸运的是，我们不会继续这条路。相反，让我们看看其他更 Ruby 风格的调试方法。

# 调试用的宝石

如你所猜，Ruby 社区遇到了调试问题，就像其他编程社区一样，因此想出了库（或 gem）来封装不同的调试行为。我们将具体讨论三个 gem，它们可以使你的调试体验与仅仅在代码中输出值完全不同：

+   Debug

+   Pry

+   Byebug

所有这些 gem 都是为了同一个目的而设计的，但每个 gem 都有自己的特性，你将根据自己的偏好选择在项目中使用哪一个。让我们先看看第一个。

## debug gem

这个 gem 作为传统`lib/debug.rb`标准库的替代品（和改进）。虽然有多种使用这个 gem 的方法，但让我们先安装我们的 gem 并创建一个简单的可调试示例。首先，我们必须做一些设置。让我们使用在前面章节中学到的 Gemfile。在一个空文件夹中创建一个名为`Gemfile`的文件，并向其中添加以下代码：

```php
# Gemfile
gem "debug", ">= 1.0.0"
```

接下来，我们将通过在 shell 中运行以下命令来安装此文件中列出的 gem：

```php
bundle install
```

这应该输出类似以下的内容：

```php
Resolving dependencies...
Using bundler 2.4.6
Using debug 1.7.1
Bundle complete! 1 Gemfile dependency, 2 gems now installed.
Use `bundle info [gemname]` to see where a bundled gem is installed.
```

通过这种方式，我们在系统中安装了 debug gem。现在让我们测试一下。为了节省时间，你应该从 GitHub 仓库下载示例：

[`github.com/PacktPublishing/From-PHP-to-Ruby-on-Rails/blob/main/chapter06/debuggable_example.rb`](https://github.com/PacktPublishing/From-PHP-to-Ruby-on-Rails/blob/main/chapter06/debuggable_example.rb)

你也可以输入整个代码，尽管我不推荐这么做，因为它包含一个非常长的字符串：

```php
# debuggable_example.rb
require 'digest'
user = Hash.new
user['name'] = 'admin'
user['password'] = 'secret'
SECRET_SHA2_PASSWORD = '32b363908ba2382c892800589d6aa3104dc41e6789d2d6a12512c34ec0632834'
user_sha2_input = Digest::SHA2.hexdigest(user['password'])
if user_sha2_input == SECRET_SHA2_PASSWORD
  print "Your password is CORRECT"
else
  print "Your password is INCORRECT"
end
```

上述代码一开始可能看起来有点复杂，但就目前而言，让我们专注于运行它和输出，我们将在后面解释脚本的输出结果。

让我们用以下命令从 shell 中运行我们的代码：

```php
ruby debuggable_example.rb
```

这将输出以下内容：

```php
Your password is INCORRECT
```

我们的代码基本上是用 SHA2 算法加密密码，并与使用相同算法加密的已加密字符串进行比较。我们使用`digest`模块，在文件开头导入，用`Digest::SHA2`加密密码。如果密码匹配，我们应该得到消息`您的密码是正确的`。然而，我们可以从输出中推断出密码不匹配。为了调试，而不是用`print()`函数在这里那里打印值，让我们使用调试 gem。因为我们已经安装了 gem，所以我们可以直接将这个 gem 导入到我们的脚本中。让我们这么做。让我们在第三行代码中添加`require`语句，这样前四行代码现在看起来是这样的：

```php
# debuggable_example.rb
require 'digest'
require 'debug'
user = Hash.new
…
```

现在 gem 已经导入，我们可以创建一个断点了。断点是一行代码，调试程序会在这里暂停执行，让我们分析特定行内的程序。一旦我们尝试一下，就会更清楚。所以，让我们在为`name`设置的 hash 值之后添加一个断点。现在我们的代码应该看起来是这样的：

```php
# debuggable_example.rb
require 'digest'
require 'debug'
user = Hash.new
user['name'] = 'admin'
user['password'] = 'secret'
SECRET_SHA2_PASSWORD = '32b363908ba2382c892800589d6aa3104dc41e6789d2d6a12512c34ec0632834'
binding.break
user_sha2_input = Digest::SHA2.hexdigest(user['password'])
if user_sha2_input == SECRET_SHA2_PASSWORD
  print "Your password is CORRECT"
else
  print "Your password is INCORRECT"
end
```

现在，就像其他脚本一样运行我们的脚本：

```php
ruby debuggable_example.rb
```

但你会注意到一些新东西。我们的脚本现在会在执行过程中暂停。而不是输出文本，我们的 shell 现在看起来像这样：

```php
[4, 13] in debuggable_example.rb
     4|
     5| user = Hash.new
     6| user['name'] = 'admin'
     7| user['password'] = 'admin'
     8| SECRET_SHA2_PASSWORD = '32b3639…'
=>   9| binding.break
    10| user_sha2_input = Digest::SHA2.hexdigest
        (user['password'])
    11|
    12| if user_sha2_input == SECRET_SHA2_PASSWORD
    13|   print "Your password is CORRECT"
=>#0  <main> at debuggable_example.rb:9
(rdbg)
```

我已经截断了加密值（`'32b3639…'`），但在你的屏幕上，它应该显示完整的值。注意执行已经暂停。这是什么魔法？这不过是 debug gem 在起作用。现在我们甚至可以测试一些事情。例如，让我们看看用户哈希中有什么。在我们的调试 shell 中，让我们输入以下内容：

```php
user
```

在按下*Enter*后，shell 将返回分配给`user`的值：

```php
(rdbg) user
{"name"=>"admin", "password"=>"secret"}
(rdbg)
```

注意我们的`user`变量既有`name`也有`password`索引。但如果我们尝试获取`user_sha2_input`变量的值会发生什么？让我们试试。在调试 shell 中，输入以下内容：

```php
user_sha2_input
```

这返回一个空值：

```php
(ruby) user_sha2_input
nil
(rdbg)
```

这是因为程序还没有到达设置该变量的代码。然而，这个特殊的 shell 已经通过显示与执行相关的各种数据提供了有用的信息，例如我们正在运行的脚本（`debuggable_example.rb`）和当前所在的行（`=> 9| binding.break`）。我们还可以与此 shell 交互。让我们输入`next`然后按*Enter*：

```php
(rdbg) next
```

这将逐行前进到断点。shell 将再次渲染，因此现在显示以下内容：

```php
[5, 14] in debuggable_example.rb
     5| user = Hash.new
     6| user['name'] = 'admin'
     7| user['password'] = 'admin'
     8| SECRET_SHA2_PASSWORD = '32b3639…'
     9| binding.break
=>  10| user_sha2_input = Digest::SHA2.hexdigest
        (user['password'])
    11|
    12| if user_sha2_input == SECRET_SHA2_PASSWORD
    13|   print "Your password is CORRECT"
    14| else
=>#0  <main> at debuggable_example.rb:10
(rdbg)
```

注意我们已经从第 9 行移动到第 10 行。让我们再移动一行代码，以便允许程序分配密码。这次我们将使用`next`的快捷键，即第一个字母，`n`：

```php
(rdbg) n
```

当它再次渲染时，我们现在看到我们的断点已经移动到了`if`语句：

```php
[5, 14] in debuggable_example.rb
     5| user = Hash.new
     6| user['name'] = 'admin'
     7| binding.break
     8| user['password'] = 'admin'
     9|
=>  10| if user_sha2_input == SECRET_SHA2_PASSWORD
    11|   print "Your password is CORRECT"
    12| else
    13|   print "Your password is INCORRECT"
    14| end
=>#0  <main> at debuggable_example.rb:10
(rdbg)
```

现在我们可以查看密码的内容。再次，在这个调试 shell 中，输入`user_sha2_input`变量：

```php
(rdbg) user_sha2_input
```

我们确认值现在已经在那里，因为输出现在是以下内容：

```php
(rdbg) user_sha2_input
2bb80d537b1da3e38bd30361aa855686bde0eacd7162fef6a25fe97bf527a25b
(rdbg)
```

这就是我们的 shell 变得极其方便的地方。我们可以在执行之前测试`if`子句。让我们复制`if`子句，但不包括`if`关键字，并将其粘贴到调试 shell 中：

```php
user_sha2_input == SECRET_SHA2_PASSWORD
```

这返回`if`句子正在评估的内容：

```php
false
```

显然，有人更改了我们的脚本的密码，但犯了一个错误。你看，SHA2 算法的问题在于它不是一个你可以加密和解密的加密算法，以便你可以得到原始值。SHA2 是一个散列算法，这意味着你只能加密一个值。如果你需要用它作为密码，那么你所做的就是保存这个加密值，并将其与用户的加密输入进行比较。这样，甚至程序本身也无法获得原始密码，从而保护用户的密码免受程序本身的侵害。在这种情况下，我们可以快速确认密码不匹配。让我们调试这个问题。让我们比较加密值。首先，让我们输出`user_sha2_input`值：

```php
user_sha2_input
```

这返回以下内容：

```php
2bb80d537b1da3e38bd30361aa855686bde0eacd7162fef6a25fe97bf527a25b
```

如果我们将它与我们的代码中的 `SECRET_SHA2_PASSWORD` 值进行比较，我们会看到以下内容：

```php
32b363908ba2382c892800589d6aa3104dc41e6789d2d6a12512c34ec0632834
```

看到它们靠近，我们可以很容易地只从前四个数字看出它们有不同的值。但在我们修复代码之前，让我们通过在 debug 控制台中输入 `continue` 调试命令来完成脚本执行：

```php
(rdbg) continue
```

这将继续执行我们的脚本，直到它找到另一个断点（在这种情况下没有更多断点）或者直到它到达程序的末尾（就像这个例子一样）。此外，它将输出与我们第一次运行脚本时相同的文本：

```php
(rdbg) continue    # command
Your password is INCORRECT
```

现在 shell 恢复正常，我们可以开始修复问题。不幸的是，我们无法获取原始密码（至少在我们能够使用量子计算机之前是这样）。幸运的是，我们已经从之前的调试会话中获得了正确的值：

```php
2bb80d537b1da3e38bd30361aa855686bde0eacd7162fef6a25fe97bf527a25b
```

我们可以将这个值直接复制到我们的 `SECRET_SHA2_PASSWORD` 变量中，并移除断点，因此我们的代码现在看起来像这样：

```php
# debuggable_example.rb
require 'digest'
require 'debug'
user = Hash.new
user['name'] = 'admin'
user['password'] = 'secret'
SECRET_SHA2_PASSWORD = '2bb80d537b1da3e38bd30361aa855686bde0eacd7162fef6a25fe97bf527a25b'
user_sha2_input = Digest::SHA2.hexdigest(user['password'])
if user_sha2_input == SECRET_SHA2_PASSWORD
  print "Your password is CORRECT"
else
  print "Your password is INCORRECT"
end
```

让我们再次运行它，通过在 shell 中输入以下内容来确认修复是否正确：

```php
ruby debuggable_example.rb
```

输出应该是以下内容：

```php
Your password is CORRECT
```

通过这种方式，我们已经成功地使用 debug 钩子调试了我们的脚本。正如我们之前看到的，使用 debug 钩子，我们可以逐行运行脚本，并添加多个断点，这只是一个简单的例子。我建议你更多地尝试这个钩子，并查看它支持的其它命令：[`github.com/ruby/debug`](https://github.com/ruby/debug)。

## pry 钩子

debug 钩子是我们现在可用的用于调试代码的选项之一。我们将要查看的第二个用于调试的钩子是 pry 钩子。正如其名称明显暗示的那样，pry 是一个能够深入观察 Ruby 程序的交互式控制台钩子。从其 GitHub 页面，我们了解到 pry 是尝试替代经典的 **交互式 Ruby**（或 **IRB**）。尽管它与 debug 钩子有相似之处，但它对调试范式的处理有自己的方法。让我们试驾一下，以便你能看到我在说什么。我们将从简单地通过在命令行中运行以下命令来安装钩子开始：

```php
gem install pry
```

我们应该看到以下输出：

```php
Fetching pry-0.14.2.gem
Successfully installed pry-0.14.2
Parsing documentation for pry-0.14.2
Installing ri documentation for pry-0.14.2
Done installing documentation for pry after 0 seconds
1 gem installed
```

现在，你可以从 [`github.com/PacktPublishing/From-PHP-to-Ruby-on-Rails/blob/main/chapter06/pryable_example.rb`](https://github.com/PacktPublishing/From-PHP-to-Ruby-on-Rails/blob/main/chapter06/pryable_example.rb) 下载示例，或者简单地输入代码。我们下载或创建的文件应该命名为 `pryable_example.rb`，它看起来像这样：

```php
# pryable_example.rb
class Person
  attr_accessor :first_name, :last_name
  def full_name
    puts "#{first_name}#{last_name}"
  end
end
person = Person.new
person.first_name = "Zach"
person.last_name = "Smith"
person.full_name
```

现在让我们从 shell 中执行我们刚刚创建的脚本：

```php
ruby pryable_example.rb
```

它应该只输出一个名字：

```php
ZachSmith
```

当然，我们故意犯了一个拼写错误（在名字和姓氏之间没有包括空格），但现在让我们看看 pry 的魔法。让我们导入 `pry` 并添加一个断点。我们的代码现在应该看起来像这样：

```php
# pryable_example.rb
require 'pry'
class Person
  attr_accessor :first_name, :last_name
  def full_name
    puts "#{first_name}#{last_name}"
  end
end
person = Person.new
person.first_name = "Zach"
person.last_name = "Smith"
binding.pry
person.full_name
```

让我们再次运行我们的脚本：

```php
ruby pryable_example.rb
```

就像 debug gem 一样，我们也将进入一个交互式 shell：

```php
     9: end
    10:
    11: person = Person.new
    12: person.first_name = "Zach"
    13: person.last_name = "Smith"
 => 14: binding.pry
    15: person.full_name
[1] pry(main)>
```

我们必须考虑的是，命令将会不同。我们将在这个 shell 中开始一个简单的命令。让我们输入`ls`（这是一个小写的 L 和一个小写的 S），就像 Linux 命令一样：

```php
ls
```

提示符将返回类似以下内容：

```php
[6] pry(main)> ls
self.methods: inspect  to_s
locals: _  __  _dir_  _ex_  _file_  _in_  _out_  person
pry_instance
[7] pry(main)>
```

Pry 列出我们程序（在内存中）的内容，并且 pry 将其列出，就像它是一个文件系统。如果我们想“pry”到`person`对象中，我们可以像导航文件夹一样导航。让我们用以下命令来做这件事：

```php
cd person
```

然后，紧接着，我们再次执行`ls`命令：

```php
ls
```

现在提示符将显示`person`的内容：

```php
[7] pry(main)> cd person
[8] pry(#<Person>):1> ls
Person#methods: first_name  first_name=  full_name  last_name  last_name=
self.methods: __pry__
instance variables: @first_name  @last_name
locals: _  __  _dir_  _ex_  _file_  _in_  _out_
  pry_instance
[9] pry(#<Person>):1>
```

根据你可能安装的 Ruby 版本，你可能会看到之前的提示或略有不同的类似提示，但无论如何，它们都会显示`person`对象的全部内容。如果我们仔细观察，我们会看到`first_name`、`last_name`和`full_name`方法，以及`@first_name`和`@last_name`变量。这就是`pry`的一个优点，因为它允许我们深入挖掘对象。我们还可以做的另一件有用的事情是修复并重新加载我们的代码，这意味着我们可以更改代码的一部分，然后将其重新加载到内存中，而无需停止当前执行。如果我们回到我们的源代码，并在第七行中在`first_name`和`last_name`之间添加一个空格，我们的代码现在应该看起来像这样：

```php
# pryable_example.rb
require 'pry'
class Person
  attr_accessor :first_name, :last_name
  def full_name
    puts "#{first_name} #{last_name}"
  end
end
person = Person.new
person.first_name = "Zach"
person.last_name = "Smith"
binding.pry
person.full_name
```

最后，在我们交互式 shell 中，我们运行以下命令：

```php
reload-method
```

这应该使提示符再次显示我们的断点位置：

```php
     9: end
    10:
    11: person = Person.new
    12: person.first_name = "Zach"
    13: person.last_name = "Smith"
 => 14: binding.pry
    15: person.full_name
```

现在，让我们再次通过输入以下内容来获取全名：

```php
person.full_name
```

我们随后在输出中看到修正后的完整名称：

```php
[1] pry(main)> person.full_name
Zach Smith
=> nil
[2] pry(main)>
```

现在，我们可以使用 pry 的退出别名（`!!!`）退出调试 shell：

```php
!!!
```

这将退出调试 shell。所以，正如你所看到的，我们有很多优点，比如我们观察对象深度的水平。然而，我们确实有一个主要的缺点。在使用 pry 时，我们只能看到特定执行时刻的内容。我们不能像使用 debug 那样移动到另一行代码。我们可以做的是添加多个断点，因为 pry 允许这样做。无论如何，不要让这个缺点阻止你尝试它。

## The byebug gem

另一个对我们可用的工具是 byebug gem。Byebug 是另一个不依赖于内部核心资源的调试器。它的工作方式与我们之前审查的两个选项类似。此外，这个 gem 支持流行的 IDE，如 Sublime、Atom 和 VS Code。

让我们快速了解一下这个工具。我们将安装 byebug gem。在 shell 中输入以下命令：

```php
gem install byebug
```

我们应该在 shell 上看到以下输出：

```php
Fetching byebug-11.1.3.gem
Building native extensions. This could take a while...
Successfully installed byebug-11.1.3
Parsing documentation for byebug-11.1.3
Installing ri documentation for byebug-11.1.3
Done installing documentation for byebug after 4 seconds
1 gem installed
```

既然这个 gem 已经可用，让我们创建一个名为`byeable_example.rb`的文件，并包含以下内容：

```php
# byeable_example.rb
require 'byebug'
[1,5,7,9].each do |index|
  not_label = index ? "NOT":""
  output = "#{index} is #{not_label} larger than 6"
  puts output
end
```

让我们先在没有断点的情况下运行我们的程序。在我们的 shell 中，让我们输入以下内容：

```php
ruby byeable_example.rb
```

输出将会是以下内容：

```php
1 is NOT larger than 6
5 is NOT larger than 6
7 is NOT larger than 6
9 is NOT larger than 6
```

我们的项目为每个数字显示相同的消息，这并不是我们预期的。由于我们的 gem 已经安装并加载，我们只需要添加一个断点。我们通过添加`byebug`文本来实现这一点。让我们在我们的代码中输出变量之前添加这一行。我们的代码现在应该看起来像这样：

```php
# byeable_example.rb
require 'byebug'
[1,5,7,9].each do |index|
  not_label = index ? "NOT":""
  output = "#{index} is #{not_label} larger than 6"
  byebug
  puts output
end
```

再次，让我们在 shell 中运行我们的脚本：

```php
ruby byeable_example.rb
```

就像其他调试器一样，我们看到我们的 shell 变成了一个交互式调试 shell：

```php
   1: # byeable_example.rb
   2: require 'byebug'
   3:
   4: [1,5,7,9].each do |index|
   5:   not_label = index ? "NOT":""
   6:   output = "#{index} is #{not_label} larger than 6"
   7:   byebug
=> 8:   puts output
   9: end
(byebug)
```

从这个 shell 中，我们可以编写命令并查看变量的内容，就像我们使用 debugger 和 pry 时做的那样。我们看到文件和我们在第 7 行定义的断点。在这个时候，我们可以访问`index`变量。让我们在 byebug shell 中输入以下内容：

```php
index
```

我们立即看到这个变量的值：

```php
1
```

由于我们处于`each`循环中，`index`变量的值已经被数组（`[1,5,7,9]`）的第一个值填充，在这个例子中是`1`。我们也可以查看`not_label`变量内部的内容。让我们在 byebug shell 中输入它：

```php
not_label
```

现在，我们看到该变量的内容：

```php
"NOT"
```

这是因为`not_label`变量正在获取`index`值，而不是将其与 6 进行比较，我们只是将其传递给三元运算符（`?`符号），因此`not_label`变量将始终具有文本`"NOT"`作为其值。我们可以通过输入关键字`continue`来验证这一点：

```php
continue
```

`continue`命令将继续执行程序，直到找到另一个断点。由于我们处于循环中，我们将移动到数组的下一个元素，直到再次达到断点。让我们再次在 shell 中查看`index`值：

```php
index
```

这将输出以下内容：

```php
5
```

这确认我们已经移动到数组的下一个元素。让我们再次查看`not_label`变量内部的内容：

```php
not_label
```

我们应该看到相同的值：

```php
"NOT"
```

这将发生在分配给`index`变量的每个值上。所以，让我们通过在 byebug shell 中输入`exit`命令来退出：

```php
exit
```

我们的 shell 应该恢复正常。现在让我们通过在`not_label`变量内添加比较并移除我们的断点来修复我们的代码。我们的代码现在应该看起来像这样：

```php
# byeable_example.rb
require 'byebug'
[1,5,7,9].each do |index|
  not_label = index < 6 ? "NOT":""
  output = "#{index} is #{not_label} larger than 6"
  puts output
end
```

现在让我们运行我们的脚本的最终版本：

```php
ruby byeable_example.rb
```

这应该正确输出文本消息：

```php
1 is NOT larger than 6
5 is NOT larger than 6
7 is  larger than 6
9 is  larger than 6
```

恭喜！我们的代码已经被调试并修复。再次强调，这是一个大大简化的例子，但我希望你能理解 byebug 有多有用。byebug 有额外的命令、配置选项，甚至还有我们命令的历史记录。请查看 byebug 的 GitHub 页面以获取更多信息：[`github.com/deivid-rodriguez/byebug`](https://github.com/deivid-rodriguez/byebug)。

如你可能注意到的，所有这些 gem 都以非常相似的方式工作。正如我们将在下一节中看到的，这是因为它们都是基于 IRB 构建的。

# 理解 IRB 的有用性

在*第三章*中，我们简要地看到了**IRB** shell，我希望你注意到了 IRB 和我们的调试工具之间的一些相似之处。基本上，这些 gem 的作用是增强 IRB，使我们能够查看内存中的变量，移动执行点，并在程序的“冻结”状态下工作。最终，你将能够选择这些工具中哪一个更适合你的日常使用。我可以引导你选择一个，或者你也可以简单地使用内置的 IRB。我看到一位开发者使用的一个技巧是，他在 IRB 中编写了大部分代码，一旦代码运行无误，他就会将代码从 IRB 复制到 IDE 中保存工作。这节省了他很多本可以用于测试的时间。另一位同事使用了一个非常流行的 IDE，叫做 RubyMine。这个工具允许我们通过点击按钮添加断点（以及其他许多功能）。此外，使用调试器的缺点之一是在设置断点时添加调试代码。如果你忘记在将代码提交到代码库时移除断点，这可能会破坏你的代码。RubyMine 消除了这种风险，因为断点不是添加到你的代码中，而是由 IDE 管理的。

现在，大多数 IDE 都集成了某种调试机制，以便于你的使用。由于这可能超出了本书的范围，你可能需要自己检查这些 IDE 及其对调试的实现：

+   [Sublime Text 官网](https://www.sublimetext.com/)

+   [Visual Studio Code 官网](https://code.visualstudio.com/)

+   [JetBrains Ruby 官网](https://www.jetbrains.com/ruby/)

无论你选择哪条道路，完全取决于你自己。我只是想让你了解 IRB 是什么，以及它在 Ruby 开发中可以有多么强大。

来自 Laravel 背景的 PHP 程序员会对 tinker 工具很熟悉，这个工具基本上是一个加载了 Laravel 组件的 PHP 交互式 shell。我必须承认，tinker（[`github.com/laravel/tinker`](https://github.com/laravel/tinker)）看起来很像 Ruby on Rails 的一个工具：Rails 控制台。Rails 控制台基本上是加载了 Ruby on Rails 组件的 IRB，以便于使用。但让我们不要跑题。我们首先必须了解 Ruby on Rails。

# 摘要

在本章中，我们学习了在 PHP 中可以使用的不同调试功能及其在 Ruby 中的等效功能，以及如何通过使用三种易于配置和安装的工具来调试我们的代码。虽然我不想强推我偏爱的调试宝石，但我可以说，在 byebug 出现之前，我使用了几年 pry。我建议你不仅尝试 byebug，还要留意新的调试宝石。我们还学习了如何将断点添加到我们的调试代码中，以及这些断点在开发过程中的有用性和强大功能。最后，我们了解到所有这些宝石基本上都是增强版的 IRB，因此我们可以轻松地使用它们，因为它们的行为非常相似。

在看到所有这些之后，我们现在已经准备好登上 Ruby on Rails 的列车。

# 第二部分：Ruby 与 Web

在本部分，你将通过最流行的 Ruby 框架 Ruby on Rails 了解 Ruby 在 Web 开发中的应用。除此之外，你还将学习数据库处理和视图的基础知识，最后，将概述 Ruby on Rails 应用程序与 PHP 应用程序在托管方面的差异。

本部分包含以下章节：

+   *第七章*, *理解约定优于配置*

+   *第八章*, *模型、数据库和活动记录*

+   *第九章*, *整合一切*

+   *第十章*, *托管 Rails 应用程序与 PHP 应用程序的考虑因素*
