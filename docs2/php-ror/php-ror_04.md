

# Ruby 脚本与 PHP 脚本

就像我们在查看 Ruby 和 PHP 语法时发现的相似之处一样，我们将更进一步，深入研究 Ruby 脚本和 PHP 脚本之间的相似性。脚本是一段代码，它将执行一个任务然后停止执行。这个任务可能是简单的，也可能是复杂的，但它不被视为一个应用程序，因为它在任务完成后就会停止，并且只执行该任务。让我们一起迈出这一步，开始编写简单的脚本，以便我们最终能够编写完整的应用程序。

考虑到这一点，在本章中，我们将涵盖以下主题：

+   有用的脚本

+   文本处理

+   文件处理

+   命令行参数

# 技术要求

为了跟随本章内容，我们需要以下内容：

+   任何 IDE 来查看/编辑代码（例如，SublimeText，Visual Studio Code，Notepad++ Vim，Emacs 等）

+   对于 macOS 用户，您还需要安装 XCode 命令行工具

+   必须安装并准备好使用 Ruby 版本 2.6 或更高版本

本章中展示的代码可在[`github.com/PacktPublishing/From-PHP-to-Ruby-on-Rails/`](https://github.com/PacktPublishing/From-PHP-to-Ruby-on-Rails/)找到。

# 超越 Hello World

在上一章中，我们学习了如何运行（或执行）Ruby 代码。然而，我们只关注了语法，而没有关注代码的有用性。我们在任何语言中编写的著名的 Hello World 脚本本身是毫无用处的，至少从实用角度来看是这样。因此，让我们开始学习如何使用一些工具来使我们的脚本变得更有用。

任何一种语言中，有一个验证当前使用编程语言版本的方法是非常有用的。一旦我们获取了版本信息，如果使用的版本不正确，我们可以停止执行。因此，我们的第一步是获取当前的 Ruby 版本。让我们创建一个名为`version_verification.rb`的文件，并包含以下代码：

```php
# version_verification.rb
puts "We are running Ruby version #{RUBY_VERSION}"
```

我们可以通过在 shell 中输入以下命令来运行此脚本：

```php
ruby version_verifications.rb
```

它应该输出类似以下内容：

```php
We are running Ruby version 2.6.8
```

在这个脚本中，我们使用`RUBY_VERSION`常量来获取我们正在使用的 Ruby 当前版本，并将这个常量与一个字符串进行插值，以查看有关 Ruby 版本的整个消息。这个常量本身是没有用的，但让我们给它一些实际用途。假设我们想要与其他团队分享我们的脚本，该脚本将在不同的计算机和/或环境中使用。为了确保我们的脚本能够正常工作，我们必须为我们的脚本提供某些要求或条件。验证这些要求是否得到满足也会很有用。我们有几种方法可以实现这一点。我们可以简单地比较从`RUBY_VERSION`常量获取的版本与另一个字符串，例如`'2.6.8'`。这将是最直接的方法。然而，这种方法的问题是你必须在每个地方都有相同的 Ruby 版本，这很少见。我们几乎总是有版本的小幅变化。如果我们以`'2.6.8'`为例，在其他系统中，我们可能会得到`'2.6.5'`、`'2.6.7'`，甚至`'2.6.9'`。所有这些版本不仅等效，而且对于我们所要求的是有效的。所以，让我们说我们的要求是 2.6 及以上，这相当于任何高于`'2.5.9'`的版本。我们可以将`RUBY_VERSION`常量获取的版本分割，通过点分割其值，然后开始比较。然而，这太费事了；这正是 Ruby 发挥作用的地方。Ruby 附带一个名为`stdlib`的库，其中包含一些在遇到这类问题时非常有用的实用工具。具体来说，Ruby 有一个`Gem::Version`类，它将解决我们手头的这个问题。我们将在示例中包含它，但为了确保验证工作正常，我们将它与版本`'3.0'`进行比较。一旦我们测试了验证，我们就可以添加正确的版本（`'2.6'`）。我们的代码现在看起来是这样的：

```php
# version_verification.rb
puts "Incompatible Ruby version" if Gem::Version.new(RUBY_VERSION) < Gem::Version.new('3.0')
puts "We are running Ruby version #{RUBY_VERSION}"
```

如果我们在我们的 shell 上运行这个脚本，我们会得到以下输出：

```php
Incompatible Ruby version
We are running Ruby version 2.6.8
```

我们的验证成功了，但现在的问题是，如果没有正确的 Ruby 版本，我们没有停止执行。显示 Ruby 版本的提示信息不应该显示。如果我们用 PHP 编写脚本，我们可以简单地使用`die()`函数（它等同于`exit()`语言结构）然后脚本就会立即停止。然而，由于我们正在编写脚本，某些实践可以使我们的脚本更加有用。如果我们的程序在网络上运行，我们会依赖 HTTP 响应状态码（[`developer.mozilla.org/en-US/docs/Web/HTTP/Status`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status)）来告诉浏览器我们的页面已经渲染并且发生了错误。同样，在脚本中，我们依赖退出码（[`www.baeldung.com/linux/status-codes`](https://www.baeldung.com/linux/status-codes)）来告诉 shell 我们的程序失败了。考虑到这一点，我们将使用`Kernel::exit()`方法来停止执行并向 shell 发送一个信号，表明我们的脚本失败了。此方法接收一个参数，然后将其发送到 shell。这个参数是一个错误码，操作系统可以使用它。我们将使用错误码`1`，因为它表示一个一般错误。在做出这个调整后，我们的代码现在将如下所示：

```php
# version_verification.rb
Kernel::exit(1) if Gem::Version.new(RUBY_VERSION) < Gem::Version.new('3.0')
puts "We are running Ruby version #{RUBY_VERSION}"
```

如果我们在 shell 上运行此脚本，由于脚本在消息之前停止了执行，所以不会有输出。在基于 Unix 的系统上，在我们的脚本停止后，我们可以运行以下命令：

```php
echo $?
```

这将返回`1`，这与我们传递给`exit()`方法的参数相同。`$?`返回最后运行的命令的退出码。

Windows 用户注意事项

对于 Windows 用户，shell 的输出将根据您使用的 Windows shell 而有所不同。如果您使用的是 PowerShell，您可以通过在 PowerShell 中执行`echo $LastExitCode`命令来获得相同的输出。

有关此变量的更多信息，请参阅 Windows 文档：[`learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_automatic_variables?view=powershell-7.3`](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_automatic_variables?view=powershell-7.3)。

我们还需要对 Ruby 版本验证脚本进行最后一次调整才能使其完整。正如我之前提到的，我们只添加了版本`'3.0'`以确保我们的代码能够工作，但实际上，我们希望验证安装的版本大于`'2.6'`。因此，我们的最终验证将如下所示：

```php
# version_verification.rb
Kernel::exit(1) if Gem::Version.new(RUBY_VERSION) > Gem::Version.new('2.6')
puts "We are running Ruby version #{RUBY_VERSION}"
```

如果我们在 shell 上执行我们的脚本，我们会得到以下结果：

```php
We are running Ruby version 2.6.8
```

通过这样，我们确保如果我们的脚本在低于 `'2.6'` 版本的 Ruby（例如，`'2.5.7'` 或 `'2.2.1'`）上执行，则脚本将停止并发送错误信号。恭喜！我们已经创建了我们的第一个有用的代码片段。这种技术通常被经验丰富的 Ruby 开发者使用，他们对版本变化非常了解。现在，您可以根据需要改进这个片段，例如添加错误消息，并添加一个上限（例如，大于 `'2.5.9'` 但小于 `'3.0'`）。现在我们已经创建了第一个真正有用的脚本，让我们看看一些其他有用的 Ruby 工具，用于处理文本。

# 文本处理

在您成为 Ruby 开发者的旅程中，您很可能会遇到字符串（文本），因此了解如何处理和操作这类数据非常重要。无论您需要将文本大写、获取部分字符串，甚至修剪字符串，Ruby 都提供了一套丰富的工具来按需操作文本。大多数编程语言都有这类工具，Ruby 也不例外。例如，假设我们想要获取之前输入的名字并确保所有字母都是大写或小写。Ruby 有两种方法可以做到这一点：`upcase()` 和 `downcase()`。让我们通过创建一个名为 `string_cases.rb` 的文件并使用以下代码来尝试它们：

```php
first_name = "benjamin"
last_name = "BECKER"
puts "My full name is #{first_name} #{last_name}"
```

到目前为止，我们已经声明了两个变量并使用插值输出了全名。假设我们在壳中运行此脚本，如下所示：

```php
ruby string_cases.rb
```

输出结果如下：

```php
My full name is benjamin BECKER.
```

由于我们声明名字时使用了小写字母，而姓氏使用了大写字母，所以输出结果并不令人意外。然而，将名字和姓氏的大小写分开是不合理的。因此，我们可以将它们都改为大写或小写。让我们尝试这两种解决方案。

## 大写方法

要将它们都转换为大写，我们可以使用 `upcase()` 方法。我们的代码将如下所示：

```php
first_name = "benjamin"
last_name = "BECKER"
puts "My full name is #{first_name.upcase} #{last_name}"
```

如果我们在壳中再次运行此代码，我们会得到以下输出：

```php
My full name is BENJAMIN BECKER.
```

## 小写方法

同样，我们可以使用 `downcase()` 方法将所有字符转换为小写。在这种情况下，我们的代码将如下所示：

```php
first_name = "benjamin"
last_name = "BECKER"
puts "My full name is #{first_name} #{last_name.downcase}"
```

因此，通过这个最后的更改，如果我们运行脚本，我们会得到以下输出：

```php
My full name is benjamin becker
```

如您所见，我们可以使用 `upcase()` 和 `downcase()` 方法来更改变量的大小写。然而，我们也可以直接对字符串执行相同的操作，而不仅仅是变量。为了展示这一点，让我们将代码更改为以下内容：

```php
first_name = "benjamin"
last_name = "BECKER"
puts "My full name is #{first_name} #{last_name}".upcase
```

这次，我们将整个输出字符串转换为大写。输出结果如下：

```php
MY FULL NAME IS BENJAMIN BECKER
```

虽然这样做很有趣，但它仅适用于学习目的。因此，让我们通过添加一个额外的方法来增加我们脚本的实用性。对于用户来说，无论是全部大写还是全部小写地读取全名都没有意义，当然也不够专业。所以，让我们只将 `first_name` 和 `last_name` 变量的首字母大写。

## 大写方法

我们可以使用 `capitalize()` 方法来实现这一点。现在，我们的代码将看起来像这样：

```php
first_name = "benjamin"
last_name = "BECKER"
puts "My full name is #{first_name.capitalize} #{last_name.capitalize}"
```

如果我们运行这个示例，shell 上的输出将看起来像这样：

```php
My full name is Benjamin Becker
```

注意，Ruby “知道”应该将哪些字符转换为大写，哪些字符转换为小写，以获得这个“大写”的输出。Ruby 有许多其他方法来处理和操作文本。我们本可以花掉本章的剩余时间来查看这些方法中的许多，但我更想专注于其他工具，并挑战你自己去检查这些文本方法。这些方法的文档相当清晰，而且由于这些方法是在 Ruby 哲学和最佳实践的基础上构建的，这也很有帮助。我建议你查看 `strip`、`lstrip`、`rstrip`、`start_with?`、`end_with?`、`rindex`、`gsub`、`chomp` 和 `chop` 方法：

+   [`apidock.com/ruby/String/strip`](https://apidock.com/ruby/String/strip)

+   [`apidock.com/ruby/String/lstrip`](https://apidock.com/ruby/String/lstrip)

+   [`apidock.com/ruby/String/rstrip`](https://apidock.com/ruby/String/rstrip)

+   [`apidock.com/ruby/String/start_with%3F`](https://apidock.com/ruby/String/start_with%3F)

+   [`apidock.com/ruby/String/end_with%3F`](https://apidock.com/ruby/String/end_with%3F)

+   [`apidock.com/ruby/String/rindex`](https://apidock.com/ruby/String/rindex)

+   [`apidock.com/ruby/String/gsub`](https://apidock.com/ruby/String/gsub)

+   [`apidock.com/ruby/String/chomp`](https://apidock.com/ruby/String/chomp)

+   [`apidock.com/ruby/String/chop`](https://apidock.com/ruby/String/chop)

你可能对这组方法的名称更为熟悉，因为 PHP 有与我刚刚提到的类似的方法：`trim`、`ltrim`、`rtrim`、`str_starts_with`、`str_ends_with`、`strpos` 和 `str_replace`。`chomp` 和 `chop` 方法在 PHP 中非常不同，所以我建议你在 Ruby 中仔细查看它们，因为它们可以非常有用。

上述方法的易用性和实用性证明了为什么我们应该依赖 Ruby 的字符串方法来进行字符串操作。我们当然可以自己编写所有这些功能，但这只是重新发明轮子，我们会浪费时间和精力。如果你选择这样做，我当然不会阻止你，因为你在这个过程中可能会学到很多 Ruby。然而，在这个指导性的旅行中，我们将坚持学习 Ruby 为我们提供的更多工具。现在，让我们看看 Ruby 如何允许我们执行另一个强大的操作：处理文件。

# 文件操作

几十年前，保存信息的一个少数选项（如果不是唯一选项）是将信息存储在文件中。所有种类的数据都存储在这些文件中：密码、用户数据、配置数据等等。在当时，将信息保存在纯文本文件中是保存信息最可行的方法。但随着数据库（**DBs**）的出现和数据库的使用，这一切都结束了。数据库成为了一个更可行且更受欢迎的选项，并且现在有不同类型的数据库。尽管今天这仍然是事实，但使用数据库的成本相当高。我不仅是在谈论货币成本——我是在谈论内存、磁盘和处理器时间。因此，在某些用例中，使用纯文本文件来存储信息仍然是一个更好的选择。为此，大多数编程语言，包括 Ruby 和 PHP，都使这项任务变得简单直接。让我们看看我们如何利用 Ruby 伴随的文件操作工具。

假设我们想要从一个文件中获取用户的第一个名字。为此，我们必须创建一个文件。这个文件将被命名为 `name.txt`。我们也可以不使用文件扩展名（`.txt`）来命名它，但这不会对我们的脚本功能产生影响，但始终给我们的同事开发者一些提示，了解我们脚本的目的是一个好习惯。很容易假设一个名为 `name.txt` 的文件很可能包含文本，并且这些文本将是一个名字。所以，让我们创建这个文本文件并向其中添加一些文本：

```php
mary
```

现在，让我们专注于打开这个文件。在 Ruby 中打开文件有不同的模式，但就目前而言，我们将专注于从文件中读取数据。让我们创建一个名为 `reading_file.rb` 的文件，并向其中添加以下代码：

```php
# reading_file.rb
File.open("name.txt")
```

首先，我们必须让 Ruby 打开文件，以便它可以处理和操作它。`File.open()` 方法正是这样做的。但现在，我们需要获取文件内容以便在脚本中使用它。首先，我们将 `File.open()` 的结果分配给一个变量。我们的代码将如下所示：

```php
# reading_file.rb
file_instance = File.open("name.txt")
```

通过这样，我们将 `File.open()` 方法的输出保存到了 `file_instance` 变量中，这反过来又让我们能够访问文件的内容。Ruby 有一个非常直观的方法来获取文件内容：`read()` 方法。`read()` 方法获取文件内容并将其转换为字符串。所以，让我们获取这个字符串并将其输出，以确保我们的脚本正在正常工作。现在，我们的脚本看起来是这样的：

```php
# reading_file.rb
file_instance = File.open("name.txt")
user_name = file_instance.read
puts "The user's name is #{user_name}"
```

如果我们在 shell 中用 `ruby reading_file.rb` 运行我们的脚本，输出将如下所示：

```php
The user's name is mary
```

And voilà – we have successfully read a value from a text file. In the code, we got Ruby to open the `name.txt` file. Then, from the instance we got as a result, we obtained the original file’s contents as a string. Lastly, we used the value in a string to output something useful to the user. We can get fancy and capitalize the username with our already acquainted `capitalize()` method. We can also test that our script is reading from the `name.txt` file. Let’s open the `name.txt` file and change the name contained in the file to something else:

```php
nancy
```

Now, let’s run our script in the shell again with `ruby reading_file.rb`. The output should be as follows:

```php
full_name.txt with the following content:

```

paul smith

```php

 We’re going to manipulate this file with another Ruby script called `full_name.rb`. Initially, the script is going to be the same as the `reading_file.rb` script. We can even just copy the file, but we are going to make some tweaks to separate the full name into `name` and `last_name`. We’ll also change the `name.txt` parameter to `full_name.txt`. So, let’s look at the code in the `full_name.rb` file:

```

# full_name.rb

file_instance = File.open("full_name.txt")

user_name = file_instance.read

puts "The user's name is #{user_name}"

```php

 If we execute this script on the shell with `ruby full_name.rb`, the output will be as follows:

```

The user's name is paul smith

```php

 There’s nothing unexpected here as the functionality is pretty much the same as the first script, `reading_file.rb`. But what if we wanted to have the name and the last name capitalized? We could try using the `capitalize()` method on the `user_name` variable. Let’s do that. The line where we output `user_name` will look like this:

```

puts "The user's name is #{user_name.capitalize}"

```php

 However, when we run the script again, the output will be as follows:

```

The user's name is Paul smith

```php

 Unfortunately for us, the `capitalize()` method only changes the first letter of the first word to uppercase. But do not despair, as we can accomplish the correct upper casing with just a single line of code. Before we do that, we will look at three additional methods: `split()`, `map()`, and `join()`.
The split() method
We can use `split()` to divide a word by spaces into an array. Simply put, `split()` would turn `paul smith` into an array of `[ "paul", "smith" ]`, which we can use in our current situation. So, let’s incorporate it into our code:

```

# full_name.rb

file_instance = File.open("full_name.txt")

user_name = file_instance.read.split

puts "The user's name is #{user_name[0]} #{user_name[1]}"

```php

 In the preceding code, we took the string from the file and applied the `split()` method.
This method divided the `paul smith` into an array of two elements. In the end, we used the element on the `0` slot (`user_name[0]`) and the `1` slot (`user_name[0]`) and embedded them into the string. For now, the output is the same, but with the advantage that we have divided the name into two words. We could apply the `capitalize()` method to both elements and be done with our task at hand. But this is when we have to take a step back and think in more broad terms for our script. What would happen if someone had a middle name? Or how would our script behave if a user had two last names? Our script would truncate part of the name in both of these cases. It is our job, as developers, to create code that is generic and that will behave properly, even with some unexpected input. This is where the `map()` method proves useful.
The map() method
The `map()` method is equivalent to iterating through an array and applying a method to each element of the array in a single line. It receives a method that we want to apply to each element as a parameter. So, let’s have another rewrite of our script:

```

# full_name.rb

file_instance = File.open("full_name.txt")

user_name = file_instance.read.split.map(&:capitalize)

puts "The user's name is #{user_name}"

```php

 Now, the output is something slightly strange, but closer to what we’re looking for. If we run this script again, we will get the following output:

```

The user's name is ["Paul", "Smith"]

```php

 We are almost there. Here, we are reading the name from the file, then dividing it by spaces into words, and finally applying each word to the `capitalize()` method. The problem with the output is that, yes, we’ve capitalized each element of the array, but then we are printing the whole array as a string, so the square brackets (`'[ ]'`) are included on the string. We are missing one last step, which is where the `join()` method comes in handy.
The join() method
The `join()` method does the opposite of `split()`. The `join()` method takes an array, converts it into a string, and glues each element with what we set as a parameter. So, the last step is to make the `user_name` array a string, each element separated by a white space. So, let’s add that last touch:

```

# full_name.rb

file_instance = File.open("full_name.txt")

user_name = file_instance.read.split.map(&:capitalize).join(' ')

puts "The user's name is #{user_name}"

```php

 And with that, our generic script is done. Let’s take it out for a ride. If we were to run it on the shell, the output would be as follows:

```

The user's name is Paul Smith

```php

 Now, since we claim that our script is generic, it should not be an issue if we were to add a middle name. So, let’s change the name in the `full_name.txt` file:

```

paul isaac smith

```php

 If we were to run the script again, the output would be as follows:

```

The user's name is Paul Isaac Smith

```php

 We are still missing the other use case that I mentioned in which some people in some countries have two last names. So, let’s change the name one more time in our `full_name.txt` file:

```

benjamin eliseo pineda avendaño

```php

 As with the other examples, the script will run correctly and output the following:

```

The user's name is Benjamin Eliseo Pineda Avendaño

```php

 We’ve successfully made a truly generic piece of code. It will work whether we add a single name, a generic name (name and last name), or a special combination of first, middle, and two last names. While the code is not as readable as other snippets we’ve read, I can guarantee that you will encounter a combination of the `split()`, `map()`, and `join()` methods whenever you move into more advanced code. Once we get to using the Ruby on Rails framework, you will see and use both of these methods there.
So far, we’ve only written code in read-only mode. Now, let’s look at creating and modifying file contents.
Creating and modifying file contents
One practical use of reading and writing a file would be creating and modifying a counter value saved in a text file. What if we wanted to keep track of how many times a script has been executed? We could add a file with a number and each time we run the script, we could increment this value and simply output it to the user. We’ll start by creating a file called `counter.rb` with the following code:

```

# counter.rb

file_instance = File.open("counter.txt", "w")

counter = file_instance.read

puts "Time(s) script has been run: #{counter}"

```php

 Again, we are opening a file, but in this case, we’ve added an additional parameter (`"w"`) so that we can write contents to the file. Additionally, we are going to try to create the file with our script instead of creating it by ourselves. So, let’s run this script from the shell with `ruby counter.rb`. The output should be as follows:

```

counter.rb:3:in 'read': not opened for reading (IOError)

from counter.rb:3:in '<main>'

```php

 Unfortunately for us, this is an error. If we look closer at the error description, it reads `not opened for reading`. This is because we set `"w"` mode, which is a write-only mode. We can only write to the file in this mode. However, we need to both read and write the contents of the file. Also, notice that the `counter.txt` file has been created, and that’s an advantage of `"w"` mode. If the file we are trying to write doesn’t exist, it will create it for us. We want this behavior, but we also want to be able to read the contents of the file. So, let’s change the mode to `"a+"` in our script:

```

# counter.rb

file_instance = File.open("counter.txt", "a+")

counter = file_instance.read

puts "Time(s) script has been run: #{counter}"

```php

 Don’t forget to delete the `counter.txt` file and execute the script again. The output will now look like this:

```

Time(s) script has been run:

```php

 If we check the folder in which our script is, we will notice that the `counter.txt` file has been created. However, the value is empty, which is unintended. So, let’s tweak our script to convert that into a number:

```

# counter.rb

file_instance = File.open("counter.txt", "a+")

counter = file_instance.read.to_i

puts "Time(s) script has been run: #{counter}"

```php

 Notice that on line 3 of our script, we’ve added `.to_i` at the end of the line, which converts the contents of the string into a number. In this scenario, the file is empty and thus returns an empty string, which, in turn, is converted into a `0`. Let’s run this script again. The output will be as follows:

```

Time(s) script has been run: 0

```php

 So far, so good. However, if we run it again, the output will remain the same as we have not added the functionality to increment the number. Let’s do just that:

```

# counter.rb

file_instance = File.open("counter.txt", "a+")

counter = file_instance.read.to_i

puts "Time(s) script has been run: #{counter}"

counter += 1

File.write("counter.txt", counter)

```php

 With the last two lines, we’ve incremented the counter value by one and written the said value to the same `counter.txt` file. As a final test for this script, let’s delete the `counter.txt` file once more and run the script a few times. This should be the output:

```

Time(s) script has been run: 0

Time(s) script has been run: 1

Time(s) script has been run: 2

Time(s) script has been run: 3

```php

 With this output, we can confirm that our script has run correctly a couple of times. As I mentioned previously, even with the existence of DBs, file reading and writing can be useful, be it for saving configuration values or for logging, and it is fast and easy to implement. You can find additional examples and modes at [`www.rubyguides.com/2015/05/working-with-files-ruby/`](https://www.rubyguides.com/2015/05/working-with-files-ruby/).
Now that we’ve established how we can read and write to and from files, let’s take a look at the next feature that will help us give our scripts more usefulness: command-line arguments.
Command-line arguments
So far, we’ve added both variable and fixed (either numeric or string) values to our code. To make our scripts more generic and more usable for other folks, we can add parameters that won’t be hardcoded within the code. If you’re not familiar with the term, *hardcoded* is the practice of writing fixed variable values within code. In our previous examples, we added the filename that we were going to open as a fixed value – that is, to change it, we would have to change the source code. To avoid that, we could pass the script a value (a filename, in this case) that whoever runs the script can change. Passing values to a script is what we commonly refer to as command-line arguments. We can have multiple arguments, a single argument, or as we’ve done so far, no arguments. Let’s start with a simple example, then work our way up to more complex examples that will help us make our scripts more generic.
Let’s start by taking a string as a command-line argument on a script, format it, and output it to the shell. We will start by creating a script called `command_line.rb` with the following code:

```

# command_line.rb

input_arguments = ARGV

puts "Hello #{input_arguments[0]}"

```php

 In this script, we are using `ARGV`, which is an array that contains any parameters passed to our script, then assigns it to a variable, to finally pass its first value to a string to be outputted. Let’s try running the script. First, let’s try it with no arguments by running this on the shell:

```

ruby command_line.rb

```php

 This will output the following:

```

Hello

```php

 We received this output we have not passed any command-line arguments to our script. So, how do we pass arguments, you may ask? Well, it’s as simple as writing the value right after the filename when we run it. Now, let’s try this with a name value. On the shell, run the following:

```

ruby command_line.rb marco

```php

 We will now see the following output:

```

Hello marco

```php

 As we can see, Ruby detects a single value on the `ARGV` array, and as a result, the output shows the same value we passed to the script. Unlike the value we obtained through opening a file, the `ARGV` array works a bit differently. Let’s try adding a second argument to our script. Let’s run it with both a different name and a second parameter. Back in the shell, run the following:

```

ruby command_line.rb ben franco

```php

 The output will be as follows:

```

Hello ben

```php

 This is because we are only using the first value of the `input_arguments` array – that is, the value contained in `input_arguments[0]`. Ruby takes the string that is passed as an argument, automatically splits it by spaces, and then places each element in the `ARGV` array. Let’s use all of the arguments that are being passed and wrap up this example. We will take the `map()` and `join()` combo that we previously used in the file handling examples to glue and show all arguments passed to the script, and since it’s a name, we will capitalize it in the process. So, let’s tweak our script once more so that it does just that:

```

# command_line.rb

input_arguments = ARGV

puts "Hello #{input_arguments.map(&:capitalize).join(' ')}"

```php

 Now, let’s run it a couple of times with different names each time:

```

ruby command_line.rb ben aaron jones

```php

 This will output the following:

```

Hello Ben Aaron Jones

```php

 Let’s try it with more parameters:

```

ruby command_line.rb gaby audra luna WOODHOUSE

```php

 This will output the following:

```

Hello Gaby Audra Luna Woodhouse

```php

 The same goes for running it with fewer arguments. Let’s give this one more try with a single name:

```

ruby command_line.rb al

```php

 This will also run but with a single name, just like on the first iteration of the script. The output will be as follows:

```

Hello Al

```php

 Now that we understand the basics of command-line arguments, let’s start giving them a bit more usefulness. Let’s write a script that will take two arguments, the first being a name and the second being a digit. We will get our script to get the digit and print the name as many times as that digit. We will also add an error message if the script is run with fewer or more arguments than what we need for our script to work. So, let’s start by creating a file called `validate_arguments.rb` and add the following code to it:

```

# validate_arguments.rb

input_arguments = ARGV

name = input_arguments.first

cycle_times = input_arguments.last.to_i

cycle_times.times { 输出 name }

```php

 In this script, we get the command-line arguments and use the first one as the name and the last one as the counter for our cycle. With the `cycle_times` variable, we’re casting (or converting) the value of the last element of the `input_arguments` array from a string into an integer. Then, we’re using the `times` method to repeat a piece of code inside the curly brackets. Now, let’s try running our script with the following values:

```

ruby validate_arguments.rb gabriela 5

```php

 As expected, the output of the script is as follows:

```

gabriela

gabriela

gabriela

gabriela

gabriela

```php

 We may think our work here is done, but we’d be wrong. This is what we call the *happy path*, in which we feed our script values that the script is expecting and thus the script’s behavior is correct. However, this is utopic as this never happens in real life. In real life, the user will forget to feed the script both parameters or will feed the parameters in the wrong order. We, as coders, need to take this into account and code appropriately. We need to validate that the input we feed the script is either correct or that we need to tell the user that the parameters are incorrect. What happens if we invert the parameters? Let’s see:

```

ruby validate_arguments.rb 5 gabriela

```php

 This outputs nothing. Let’s try running the script with no arguments:

```

ruby validate_arguments.rb

```php

 This also outputs nothing. This is a mistake on our side because we know how our script works, but someone who might be using the script doesn’t. There is no documentation and the least we can do is output error messages that can guide the user as to the correct usage of the script. Additionally, we have to assume that the end user does not know how to program and is not going to open the script to view its usage. So, let’s start by validating that the script only receives two arguments. Also, let’s add an error message to help the user out. Let’s create our validation in our code. Our `validate_arguments.rb` script will now look like this:

```

# validate_arguments.rb

if ARGV.size != 2

输出错误。脚本执行失败！

end

input_arguments = ARGV

name = input_arguments.first

cycle_times = input_arguments.last.to_i

cycle_times.times { 输出 name }

```php

 Now, when we run it again without any arguments, we will see an error message:

```

错误。脚本执行失败！

```php

 However, even though we are seeing an error, the script is still going through the whole code, which is not what we want. To prove this, let’s add another message at the end of the code:

```

# validate_arguments.rb

if ARGV.size != 2

输出 "错误。脚本执行失败！"

end

input_arguments = ARGV

name = input_arguments.first

cycle_times = input_arguments.last.to_i

cycle_times.times { 输出 name }

输出 "但我们仍在运行脚本"

```php

 Let’s run the script again (with no arguments):

```

ruby validate_arguments.rb

```php

 The output will be as follows:

```

错误。脚本执行失败！

但我们仍在运行脚本。

```php

 As we learned previously, we need to stop the execution of the script once we’ve figured out that we have errors. So, let’s fix this and stop the execution with a `Kernel::exit()` call. Let’s also add a suggestion to fix the problem:

```

# validate_arguments.rb

if ARGV.size != 2

输出。错误。脚本执行失败！

输出。用法：ruby validate_arguments.rb name times_to_repeat

Kernel::exit(1)

end

input_arguments = ARGV

name = input_arguments.first

cycle_times = input_arguments.last.to_i

cycle_times.times { 输出 name }

输出 "但我们仍在运行脚本"

```php

 Let’s run the script once again without arguments:

```

ruby validate_arguments.rb

```php

 This time, we will get the correct output, and the execution will be stopped at the right point:

```

错误。脚本执行失败！

用法：ruby validate_arguments.rb name times_to_repeat

```php

 As you can see, we are no longer outputting the `But we are still running the script.` message. This is because once the script does not pass the validation block, its execution is stopped. This is great progress for our script. However, we are still missing one piece of validation. In previous examples, we inverted the arguments by passing the number and then the name. Even with our tweaks and validations, this is a use case that will still make our script behave erroneously. Let’s try it out:

```

ruby validate_arguments.rb 3 henry

```php

 This will output the following:

```

我们仍在运行脚本。

```php

 Again, we have no information as to why the output is empty, which can be very frustrating to the user. So, let’s add more validations to our script. We, as the creators of the script, know that the script is failing because the second argument should be a number – an integer, to be more precise. However, this validation can be slightly tricky because of the way a lot of programming languages behave, including Ruby and PHP. The behavior I’m referring to is the way Ruby converts a text string into an integer. As an example, a string such as `'22'` will be converted into an integer, `22`. However, the `'henry'` string will unexpectedly be converted into a valid `0`, and while this is not what we need, we can certainly take advantage of this behavior. For our script, we need the second argument to be larger than `0`. So, that’s exactly what we are going to validate:

```

# validate_arguments.rb

if ARGV.size != 2

输出 "错误。脚本执行失败！"

输出 "用法：ruby validate_arguments.rb name times_to_repeat"

Kernel::exit(1)

end

input_arguments = ARGV

name = input_arguments.first

cycle_times = input_arguments.last.to_i

if cycle_times < 1

输出 "错误。第二个参数必须是一个整数！"

输出 "用法：ruby validate_arguments.rb name times_to_repeat"

Kernel::exit(1)

end

cycle_times.times { 输出 name }

输出。我们仍在运行脚本

```php

 Let’s run the script with the incorrect arguments again:

```

ruby validate_arguments.rb 3 henry

```php

 We will get the following output:

```

错误。第二个参数必须是一个整数！

用法：ruby validate_arguments.rb name times_to_repeat

```php

 With that, we have successfully made our script validate that the input must be two arguments. If we feed the script either no arguments, one argument, or more than two arguments, it will fail and send an error message describing the correct usage of the script. Now that we’ve looked into different ways to add input to our script, let’s look at another way to interact with the user: user input.
User input
So far, we’ve made use of command-line arguments to make our scripts more generic, thus helping with what could be an automated script. Be it a shell script (as we’ve done so far) or a crontab script to be run at a designated time each day, we’ve learned the basic usage of these arguments that are fed to our scripts. But there is another type of argument that, while technically not a command-line argument, is closely related and is super useful when making scripts that interact with human users. In comes *user input*. User input helps us make a script more interactive with the user as it makes a pause in the execution of the script to wait for the user to type data and resume after the user has typed a carriage return (or the *Enter* key). This makes a more user-friendly interaction with the user. Let’s look at a simple example to see this interaction at play. We will create a file called `user_input.rb` and add the following code:

```

# user_input.rb

输出 "输入你的名字："

name = gets

输出 "Hello #{name}"

```php

 Now, let’s run it on our shell:

```

ruby user_input.rb

```php

 We will notice that the output shows the following message:

```

输入你的名字：

```php

 We will also notice that the shell looks slightly different as it is expecting input from us. So, let’s do that and type a name, hitting the *Enter* key right after entering it:

```

输入你的名字：

brandon

```php

 Immediately, the execution of our script continues and outputs the greeting that we included in our code:

```

输入你的名字：

brandon

Hello brandon

```php

 Notice how interactive our script has become. It asks for your name; we type the name and immediately we are greeted. While this is friendlier and more interactive, we still need to fix a couple of things in our code. The method we are currently using (`Kernel::gets()`; see [`ruby-doc.org/2.7.7/Kernel.html#method-i-gets`](https://ruby-doc.org/2.7.7/Kernel.html#method-i-gets)) not only includes the name, but it also includes the return of carriage character (`\n`) or in layman’s terms, the *Enter* key character. So, if we tried to compare the input to a string, we would be surprised to see that it wouldn’t behave as we would expect it to. Let’s try it with the following code:

```

# user_input.rb

输出。输入你的名字：

name = gets

输出 "Hello #{name}" if name == "brandon"

```php

 Now, let’s re-run our script:

```

ruby user_input.rb

```php

 Let’s input the name `brandon`:

```

输入你的名字：

brandon

```php

 This time, notice that we don’t see the greeting. And this is not because of a typo. This is because the `Kernel::gets()` method is capturing a character at the end of the name. In comes another method to save the day: `chomp()`. The `chomp()` method removes carriage return characters and trailing new lines from the string. Please refer to [`apidock.com/ruby/String/chomp`](https://apidock.com/ruby/String/chomp) for more details regarding this method. Essentially, it cleans up our string and leaves the original text. So, let’s modify our code so that it includes this method. Our code will now look like this:

```

# user_input.rb

输出。输入你的名字：

name = gets.chomp

输出 "Hello #{name}" if name == "brandon"

```php

 Let’s run it once more:

```

ruby user_input.rb

```php

 If we input the name `brandon` again, we will get the following result:

```

输入你的名字：

brandon

Hello brandon

```php

 We will finally get the correct greeting. So, from now on, we will get input from the user with `gets.chomp` for safe measure. Now, let’s fetch an integer from the user and run some code multiple times, depending on the number the user typed:

```

# user_input.rb

输出。输入你的名字：

name = gets.chomp

输出 "Hello #{name}" if name == "brandon"

输出 "输入尝试该过程的次数"

repeat_n = gets.chomp.to_i

repeat_n.times do

输出。尝试中…

sleep(1)

end

```php

 Let’s run it on the shell again:

```

ruby user_input.rb

```php

 Now, let’s input the name `brandon` and enter `3` afterwards. This will be the whole sequence:

```

输入你的名字：

brandon

Hello brandon

输入你想尝试该过程的次数：

3

尝试中…

尝试中…

尝试中…

```php

 After entering `brandon`, we now output instructions to enter a digit, and right after that, we fetch an integer from the user. With this digit, we will print a `trying…` message and use the `sleep()` method to pause the execution for 1 second. If you would like more detailed information regarding the `sleep()` method, please check out [`apidock.com/ruby/Kernel/sleep`](https://apidock.com/ruby/Kernel/sleep). You will notice that the script will show the `trying…` message, pause for a second, show the second message, pause again, and finally show the last message, pause one last time, and finish the execution of the script. The `sleep()` method is useful when we are waiting for a process to finish. It’s especially useful when working with API calls, which may take some time to finish. As a final exercise, let’s dive into a script’s friendliness and usefulness.
Putting it all together
Reading and understanding someone else’s code is essential to learning Ruby. With that intent, we will now look at the following example, which was written with some of the techniques we learned about in this chapter, and figure out what the script is doing:

```

# main.rb

# 第一部分：Ruby 版本验证

if Gem::Version.new(RUBY_VERSION) < Gem::Version.new('2.6')

输出 "请验证 Ruby 版本！"

Kernel::exit(1)

end

# 第二部分：打开或创建 user_name 文件

file_instance = File.open("user_name.txt", "a+")

user_name = file_instance.read

# 第三部分：空名称验证

if user_name.empty?

puts "输入您的名字："

name = gets.chomp

File.write("user_name.txt", name)

# 第四部分：程序主日志

File.write("main.log", "Writing #{name} as the entry to user_name.txt at #{Time.now}\n", mode: "a")

user_name =  name

end

# 第五部分：程序标题

puts "Hello #{user_name.capitalize}"

puts "欢迎来到第四章"

puts "请输入您希望创建日志条目次数"

# 第六部分：程序周期

repeat_n = gets.chomp.to_i

repeat_n.times do

puts "正在添加日志条目..."

File.write("main.log", "Adding entry to log at #{Time.now}\n", mode: "a")

sleep(1)

end

```php

 For us to understand the intent of the script, we will divide it into six sections that we have commented on the code itself. This is not compulsory when writing code, but I’ve taken the liberty to do this for teaching purposes. So, let’s take a look at the first section:

```

…

# 第一部分：Ruby 版本验证

if Gem::Version.new(RUBY_VERSION) < Gem::Version.new('2.6')

puts "请验证 Ruby 版本！"

Kernel::exit(1)

end

…

```php

 Here, we are comparing our currently installed Ruby version and making sure that the version is higher than 2.6\. If the version is lower than that, we print an error message and exit the program. This is something you might find very often in scripts as major versions tend to differ in terms of functionality and sometimes in syntax.
Let’s move on to the next section of our script:

```

…

# 第二部分：打开或创建 user_name 文件

file_instance = File.open("user_name.txt", "a+")

user_name = file_instance.read

…

```php

 In this section, we are opening a file, but as we’ve seen in previous examples, it’s using `"a+"` mode so that if the file does not exist, it creates it. If the file already exists, it reads its contents. The script’s intent in this section is to read a user’s name from this file, but if the file is empty, the name will be empty too. This may seem slightly different from what we’ve been doing so far. However, let’s look at the next section, where this will make more sense. In sections 3 and 4, we can see the following:

```

…

# 第三部分：空名称验证

if user_name.empty?

puts "输入您的名字："

name = gets.chomp

File.write("user_name.txt", name)

# 第四部分：程序主日志

File.write("main.log", "Writing #{name} as the entry to user_name.txt at #{Time.now}\n", mode: "a")

user_name =  name

end

…

```php

 In section 3, we can see that if the username fetched from the file is not empty, the script moves on to the next section. But if the username is empty, then we prompt the user to provide a username. Once the user types a username, the script will write the name to the `user_name.txt` file and move to section 4.
In section 4, the script simply writes a log entry to a `main.log` file in which it writes the name obtained from the user and the time in which the user did this. Lastly, the script assigns the `user_name` variable to be used later on in the script.
In section 5, we have this code:

```

…

# 第五部分：程序标题

puts "Hello #{user_name.capitalize}"

puts "欢迎来到第四章"

puts "请输入您希望创建日志条目次数"

…

```php

 In this section, we greet the user by capitalizing the name, print a welcoming message, and then print another aiding message so that the user knows what they’ll do next, which is enter a number.
In our last section, we are doing something similar to what we did in our previous looping example. Let’s take a look:

```

…

# 第六部分：程序周期

repeat_n = gets.chomp.to_i

repeat_n.times do

puts "正在添加日志条目..."

File.write("main.log", "Adding entry to log at #{Time.now}\n", mode: "a")

sleep(1)

end

…

```php

 In section 6, which is the last section of the script, we’re getting a number from the user, then taking this number and doing a cycle to execute a code the number of times the user provided. The code to be executed simply shows a message for the user and adds multiple entry logs to the same `main.log` file. In this case, the script is also using the append writing mode so that when the script writes to this file, it will write contents at the end of the file instead of replacing the previous contents. This type of logging is both common and useful in the scripting and programming realms. It helps other users debug the functionality of the script, especially when things start failing. We are only missing one thing now: running the script. Let’s run it:

```

ruby main.rb

```php

 The first time we run this file, we will get the following output on the shell:

```

输入您的名字：

```php

 Let’s enter `daniel`. After we enter this name, we will get the following output:

```

Hello Daniel

欢迎来到第四章

请输入您希望创建日志条目次数

```php

 Here, the script requires that we enter a number. Let’s type `2`. The script will respond with the following output:

```

正在添加日志条目...

正在添加日志条目...

```php

 Initially, this seems simple enough. However, if we take a look at the contents of the folder where our script resides, we’ll notice two new files: `main.log` and `user_name.txt` . If we open the `user_name.txt` file, its contents will coincide with the name we typed:

```

daniel

```php

 And if we look at the `main.log` file, we will see the following output:

```

将 daniel 写入 user_name.txt 作为条目于 2022-12-25 16:33:24 -0600

正在日志中添加条目于 2022-12-25 16:34:53 -0600

正在日志中添加条目于 2022-12-25 16:34:54 -0600

```php

 This content coincides with what happened on the initial run of the script. It wrote `daniel` to the log and added two entries to the same log. Now, let’s run the script once more:

```

ruby main.rb

```php

 This time, we will notice that the script does not ask for a user, instead using the previous username we entered. The output will look like this:

```

Hello Daniel

欢迎来到第四章

请输入您希望创建日志条目次数

```php

 This is very practical as we don’t need to enter the name every time we run the script. Lastly, it prompts us for a number to run the log process again. Let’s type `1` and wait for the response, which should look like this:

```

正在添加日志条目...

```php

 This time, as we typed `1`, the cycle only ran the code once, which is what we expected. I hope you found this reading exercise useful and I hope you make the habit of reading other developer’s code to both learn good practices and understand how to do things in Ruby.
Summary
In this chapter, we learned how to write more useful scripts that we will probably reuse in the future. We also got a glimpse at some of the tools that Ruby has to handle text. We also learned how to open, read, and write content to and from a file and how this may come in handy when writing scripts. Lastly, we were exposed to Ruby’s command-line arguments, which make our automation work easier. We also learned of Ruby’s user input arguments, which make our scripts more interactive for users. Having learned this, we are now ready to undercover some misconceptions about PHP, Ruby, Ruby on Rails, and other frameworks.

```
