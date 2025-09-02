

# 库和类语法

到目前为止，我们只使用过 Ruby 及其核心组件，而没有利用 Ruby 中可用的其他库的优势。此外，我们只对 Ruby 对象和类进行了部分了解。在本章中，我们将与您一起进行一次测试驾驶，了解 Ruby 库（gem）以及如何利用 Gemfile 来利用它们。最后，我们还将学习类语法的基础知识，以帮助我们迈向更高级的工具，例如 Ruby on Rails。

考虑到这一点，在本章中，我们将涵盖以下主题：

+   让我们准备好打包吧！！！

+   Gemfile 与 `composer.json` 的比较

+   在 Ruby 中将库集成到代码中

+   在 Ruby 中声明类

+   Ruby 中的对象

+   Ruby 中的继承

# 技术要求

要跟随本章内容，我们需要以下内容：

+   任何 IDE 用于查看/编辑代码（例如，SublimeText，Visual Studio Code，Notepad++，Vim，Emacs 等）

+   对于 macOS 用户，您还需要安装 Xcode 命令行工具

+   已安装 Ruby 版本 2.6 或更高版本，并准备好使用

本章中展示的代码可在[`github.com/PacktPublishing/From-PHP-to-Ruby-on-Rails/`](https://github.com/PacktPublishing/From-PHP-to-Ruby-on-Rails/)找到。

# 让我们准备好打包吧！！！

编程语言本身，虽然有用，但无法考虑到程序员可能遇到的每一个单一用例。语言的核心包括许多有用的库，因此开箱即用，语言本身就非常实用。这适用于大多数编程语言。然而，当我们需要超越核心库并使用其他库来解决手头的问题时，就会出现这种情况。在 Ruby 中，社区已经创建了许多库，这些库被亲切地称为 *gems*。为了跟踪这些 gems，Ruby 社区想出了一个名为 *bundler* 的工具。对于 PHP 开发者来说，bundler 的 PHP 对应物是 Composer ([`getcomposer.org/`](https://getcomposer.org/))。这两个工具都服务于相同的目的（管理库），但 bundler 在安装库到计算机上时有所不同，而 Composer 只是使它们对项目可用。但是等等——我们甚至还没有安装一个库。让我们退一步，安装一个 Ruby gem。

## 安装 gem

在 bundler 接管之前，我们应该了解在我们的系统上安装 gem 的什么和如何操作。Ruby 有解析 JSON 对象的能力，但有一个名为 `oj` 的 gem 在处理 JSON 字符串并将它们转换为 Ruby 哈希方面更为高效。让我们首先安装 `oj` gem。在 shell 中，让我们输入以下命令：

```php
gem install oj
```

在按下 *Enter* 键后运行前面的命令，我们应该得到以下输出：

```php
Fetching oj-3.14.2.gem
Building native extensions. This could take a while...
Successfully installed oj-3.14.2
Parsing documentation for oj-3.14.2
Installing ri documentation for oj-3.14.2
Done installing documentation for oj after 2 seconds
1 gem installed
```

恭喜！我们已经安装了我们的第一个 gem，现在可以在代码中使用它了。让我们通过创建一个名为 `reading_json.rb` 的文件并添加以下代码来测试这个 gem：

```php
json_text = '{"name":"Sarah Kerrigan", "age":23, "human":
  true}'
ruby_hash = Oj.load(json_text)
puts ruby_hash
puts ruby_hash["name"]
```

现在，让我们回到我们的 shell 中，尝试执行以下脚本：

```php
ruby reading_json.rb
```

我们将得到以下错误：

```php
reading_json.rb:5:in `<main>': uninitialized constant Oj (NameError)
```

为什么我们的脚本会失败？嗯，我们安装了我们的 gem，并且在安装过程中没有出现任何错误。实际上，我们不仅需要安装我们的 gem，还需要在我们的代码中导入它。让我们在文件的开始处做这件事，这样我们的代码现在看起来就像这样：

```php
require 'oj'
json_text = '{"name":"Sarah Kerrigan", "age":23, "human":
  true}'
ruby_hash = Oj.load(json_text)
puts ruby_hash
puts ruby_hash["name"]
```

然后，让我们再次执行我们的脚本，使用以下命令：

```php
ruby reading_json.rb
```

现在，我们将得到正确的输出：

```php
{"name"=>"Sarah Kerrigan", "age"=>23, "human"=>true}
Sarah Kerrigan
```

如您所见，我们已经将 JSON 字符串转换成了 Ruby 哈希，现在它已经准备好在我们的脚本中使用。这在从 JSON 文件加载配置或处理返回 JSON 响应的 API 调用时非常有用。然而，我们还没有处理一个问题。我们如何确保使用我们脚本的其他人会得到相同的结果？嗯，这就是 bundler 和 Gemfile 进入场景的提示。

# Gemfile 与 composer.json 的比较

如我之前提到的，bundler 帮助我们处理我们程序的所有依赖项——也就是说，我们需要安装的所有内容，以便我们的程序能够正确运行。为了完成这项任务，bundler 使用一个文本文件，我们将称之为`Gemfile`。Composer 以非常相似的方式工作，通过让我们创建一个名为`composer.json`的文件，但与 Composer 将所需的库下载到文件夹不同，bundler 将它们安装到我们的系统上。如果 bundler 确定缺少依赖项，它将自动尝试安装它。Ruby 的方式有点更神奇（或自动）。让我们通过测试 bundler 来进一步了解这个过程。我们将首先使用 shell 中的以下命令卸载之前安装的`oj`gem：

```php
gem uninstall oj
```

之前的命令将确认当 gem 从我们的系统中移除时：

```php
Successfully uninstalled oj-3.14.2
```

现在，如果我们再次尝试运行我们的`reading_json.rb`，我们将得到以下错误：

```php
…kernel_require.rb:54:in `require': cannot load such file -- oj (LoadError)
  from...kernel_require.rb:54:in `require'
  from reading_json.rb:1:in `<main>'
```

我们回到了起点，但现在我们将用 Gemfile 来解决我们的库（或 gem）安装问题。让我们创建一个名为`Gemfile`的文件，内容如下：

```php
source 'https://rubygems.org'
gem 'oj'
```

通过这种方式，我们告诉 bundler 在哪里下载依赖项以及我们的依赖项是什么。我们可以告诉 bundler 其他事情，比如 gem 版本或甚至要使用的 Ruby 版本（或等效版本），但现在我们将保持简单。现在，让我们试试。在 shell 中，让我们输入以下命令：

```php
bundle install
```

根据您所使用的操作系统，您可能需要输入您的 root 或管理员密码，但一旦您完成，命令的输出应该如下所示：

```php
Fetching gem metadata from https://rubygems.org/..
Resolving dependencies...
Using bundler 1.17.2
Installing oj 3.14.2 with native extensions
Bundle complete! 1 Gemfile dependency, 2 gems now installed.
Use `bundle info [gemname]` to see where a bundled gem is installed.
```

现在我们已经安装了 gem，我们可以安全地再次使用以下命令运行我们的脚本：

```php
ruby reading_json.rb
```

我们应该得到与之前完全相同的输出：

```php
{"name"=>"Sarah Kerrigan", "age"=>23, "human"=>true}
Sarah Kerrigan
```

然而，现在我们会注意到一些细微的差别。如果我们查看我们脚本和 Gemfile 所在的文件夹的内容，我们还有一个新文件叫做`Gemfile.lock`。此外，如果我们查看内容，应该会有类似以下的内容：

```php
GEM
  remote: https://rubygems.org/
  specs:
    oj (3.14.2)
PLATFORMS
  ruby
DEPENDENCIES
  oj
BUNDLED WITH
   1.17.2
```

`Gemfile.lock`文件充当依赖关系的地图。如果没有地图，bundler 必须从头开始构建它，这个过程有时会花费一些时间。然而，如果有锁文件，即使依赖项尚未安装，安装它们的过程也会更加高效。

我们已经从仅仅安装 gem 发展到现在为我们的脚本运行创建理想的场景。在下一节中，我们将探讨我们可以通过 Gemfile 设置的额外选项（例如使用 gem 的特定版本），以进一步明确我们的 Ruby 环境。

# 在 Ruby 中将库集成到你的代码中

在你成为资深 Ruby 开发者的道路上，应该掌握的一项最有用的技能是将其他 gem 集成到你的代码中。正如我们之前所看到的，这是通过使用 Gemfile 来实现的，但我们将探讨一些可以添加到其中的额外选项，并将它们集成到我们自己的脚本中。让我们编写一个脚本，它使用 GitHub 公共 API 并列出用户`@PacktPublishing`的所有公共仓库。我们可以用几种方法来做这件事，但在这个例子中，我选择了一个名为**Faraday**的 gem。你可以在这里查看源代码：[`github.com/lostisland/faraday`](https://github.com/lostisland/faraday)。

Faraday 是一个客户端库，可以帮助我们使用 Ruby 附带的自带的`Net::HTTP`库。让我们创建一个名为`integrating_gems`的文件夹，并导航到该文件夹：

```php
mkdir integrating_gems
cd integrating_gems
```

现在，让我们创建一个名为`Gemfile`的文件，其内容如下：

```php
source 'https://rubygems.org'
gem 'oj'
gem 'faraday'
```

在安装这些 gem 之前，我想使用 Gemfile 语法中可用的更多选项。我们将锁定 Faraday 的版本为 2.5，这意味着我们安装一个特定的版本。所以，让我们将 Faraday 行更改为以下内容：

```php
gem 'faraday', '2.5'
```

将版本锁定当然有其优点和缺点。优点之一是你可以确信脚本将以相同的方式，通常使用相同的语法，在多个环境中工作。缺点可能是你可能会被锁定在某个 Ruby 版本上，你可能会陷入无法升级 Ruby 直到升级 gem 的场景。对于`oj`gem，我们将使用`~`运算符并将其设置为以下内容：

```php
gem 'oj', '~> 3.13.0'
```

`~`运算符用于界定版本范围。在这个特定的情况下，我们告诉 bundler 获取 3.13.0 和 3.14 之间的最高发布版本，不包括 3.14 - 换句话说，我们需要一个大于或等于 3.13 但小于 3.14 的版本。那么，为什么使用这个运算符呢？简而言之，`~`运算符用于在我们依赖关系中增加稳定性。我不会深入解释为什么我们会使用这种语法。只需知道你迟早会遇到它。

如果你想要深入了解这个话题，请参考以下链接：

+   [`thoughtbot.com/blog/rubys-pessimistic-operator`](https://thoughtbot.com/blog/rubys-pessimistic-operator)

+   [`guides.rubygems.org/patterns/#declaring_dependencies`](https://guides.rubygems.org/patterns/#declaring_dependencies)

现在的 Gemfile 将看起来像这样：

```php
source 'https://rubygems.org'
gem 'oj', '~> 3.13.0'
gem 'faraday', '2.5'
```

现在，让我们再次使用 bundler 安装我们的依赖项。在 shell 中输入以下命令：

```php
bundle install
```

我们将得到类似以下的输出：

```php
Fetching gem metadata from https://rubygems.org/........
Resolving dependencies…
Using bundler 1.17.2
Using faraday-net_http 2.1.0
Following files may not be writable, so sudo is needed:
  /usr/local/bin
Using ruby2_keywords 0.0.5
Using faraday 2.5.0
Using oj 3.13.23
Bundle complete! 2 Gemfile dependencies, 5 gems now installed.
Use `bundle info [gemname]` to see where a bundled gem is installed.
```

现在，有了我们的 gem 准备就绪，我们可以在脚本中使用它们。让我们创建一个名为`faraday_example.rb`的脚本，内容如下：

```php
require 'faraday'
require 'oj'
conn = Faraday.new(
  url: 'https://api.github.com',
  headers: {'Content-Type' '> 'application/json'}
)
```

让我们退一步看看我们在做什么。我们正在导入`faraday`和`oj`gem。然后，我们创建了一个将连接到 GitHub API 的客户端。客户端对象需要一个 URL 和一些头信息，我们已经提供了。到目前为止，我们还没有调用 API。让我们这样做。在文件末尾添加对 API 的调用，并使用以下内容输出响应：

```php
response = conn.get('/users/PacktPublishing/repos')
puts response.body
```

现在，让我们尝试使用以下内容执行此脚本：

```php
ruby faraday_example.rb
```

这将输出大量文本，所以我只会包括一个摘录：

```php
[{"id":184740404,"node_id":"MDEwOlJlcG9zaXRvcnkxODQ3NDA0MDQ=","name":"-.NET-Core-Microservices","full_name":"PacktPublishing/-.NET-Core-Microservices","private":false,"owner":{"login":"PacktPublishing","id":10974906,"node_id":"MDEyOk9yZ2FuaXphdGlvbjEwOTc0OTA2"
…
```

通过这个输出，我们可以确认我们已经成功连接到 GitHub API。对于练习的最后部分，让我们使用`oj`gem 将 JSON 文本转换为 Ruby 哈希，并输出 API 返回的所有响应的名称。所以，让我们删除`puts`命令，并用`oj`对象替换它。我们的代码现在将如下所示：

```php
require 'faraday'
require 'oj'
conn = Faraday.new(
  url: 'https://api.github.com',
  headers: {'Content-Type' => 'application/json'}
)
response = conn.get('/users/PacktPublishing/repos')
repo_hash = Oj.load(response.body)
```

因此，我们已经从 GitHub API 服务中获取了响应并将其转换为 Ruby 哈希。最后，让我们通过在脚本末尾添加以下内容来输出 repo 的名称：

```php
repo_hash.each { |repo| puts repo['name'] }
```

如果我们再次运行脚本，输出的一部分将看起来像这样：

```php
-.NET-Core-Microservices
-Accelerate-Deep-Learning-on-Raspberry-Pi
-Accelerate-Deep-Learning-on-Raspberry-Pi-
…
```

有了这个，我们已经成功使用 Faraday gem 连接到 API，并使用`oj`gem 处理了输出。为了参考，代码最终应该看起来像这样：

```php
require 'faraday'
require 'oj'
conn = Faraday.new(
  url: 'https://api.github.com',
  headers: {'Content-Type' => 'application/json'}
)
response = conn.get('/users/PacktPublishing/repos')
repo_hash = Oj.load(response.body)
repo_hash.each { |repo| puts repo['name'] }
```

这是如何在其他环境中运行代码的起点。如果我们想让其他人能够运行我们的脚本，我们应该将脚本和 Gemfile 一起包含在我们的共享代码中，这样其他人就知道在运行脚本之前需要安装什么。

此外，只是让您知道，这仅仅是一个练习，目的是学习如何在代码中包含和使用 gem。Gemfile 上的语法只是为了学习目的而获取的。您应该始终追求拥有 gem 的最新版本。然而，锁定到一个版本在您想要控制升级的时间和地点时是有用的。

既然我们已经了解了 gem 的工作原理及其使用方法，现在我们可以继续学习 Ruby 中的面向对象编程。

# Ruby 中声明类

PHP 和 Ruby 都是使用**面向对象编程**（**OOP**）范式的语言，Ruby 是设计如此，PHP 则是通过其自身的发展。到目前为止，严肃的开发者应该非常熟悉这种范式。在 PHP 中，所有框架都使用 OOP。虽然我们不会深入探讨这种范式在 Ruby 中的实现方式，但我们会介绍类语法的基础知识。

一个类基本上是对现实世界实体的抽象。它是这个抽象的蓝图。让我们先创建一个简单的类来表示一个人，为这个人定义一些属性，以及为他们定义一个动作（或方法）。让我们创建一个名为 `class_syntax.rb` 的文件，并包含以下内容：

```php
class Person
end
```

这是最简单的了，但仅凭这一点并不很有用。为了使其有用，我们需要添加代表人的特征的属性。所以，让我们添加一些属性，比如他们的名字和姓氏。我们的代码现在将看起来像这样：

```php
class Person
  @first_name = nil
  @last_name = nil
end
```

注意，我们通过在它们前面加上 `@` 来定义我们的属性。这些被称为实例变量，并且只能通过方法来访问。现在，让我们定义一个方法来打印出全名：

```php
class Person
  @first_name = nil
  @last_name = nil
  def full_name
    puts "#{@first_name} #{@last_name}"
  end
end
```

除了实例变量之外，注意我们之前没有做过任何我们没有学过的事情。我们定义了一个方法（或函数），并在方法要打印的字符串上包含了名字和姓氏变量。我们最后要做的最后一件事是添加一个构造函数。在面向对象编程中，构造函数是一个可以在使用类定义创建对象时自定义以添加行为和属性的方法。换句话说，当我们拿蓝图时，我们定义并使用它来创建一个特定的对象，我们可以在对象创建时控制某些值。在 PHP 中，这个方法是通过简单地命名方法为 `__constructor()` 来实现的。在 Ruby 中的等效方法是命名方法为 `initialize()`。现在让我们将其包含在我们的类定义中。我们的类定义应该看起来像这样：

```php
class Person
  def initialize
   @first_name = 'James'
   @last_name = 'Raynor'
  end
  def full_name
    puts "#{@first_name} #{@last_name}"
  end
end
```

注意，我们不再需要实例变量定义，因为这是在构造函数（初始化器）中完成的。恭喜！我们的第一个类已经准备好使用了。现在让我们继续到下一节，使用这个类定义来创建我们的第一个对象。

# Ruby 中的对象

在前面的章节中，我们定义了我们的抽象人应该是什么样子。这是一个将有一个名字和一个姓氏的人，我们将能够打印出他们的全名。与建筑业务并行，因为我们现在有了蓝图，我们现在可以按照这些规格建造我们的建筑了。这种创建就是我们所说的类的实例或对象。类定义是通用的，而实例是具体的。不深入探讨类定义和对象之间的关系，我们将看看代码中这种关系看起来是什么样子，以及这将如何帮助我们编写更好、更易读的代码。让我们用我们之前的代码创建我们的第一个对象：

```php
# class_syntax.rb
class Person
  def initialize
   @first_name = 'James'
   @last_name = 'Raynor'
  end
  def full_name
    puts "#{@first_name} #{@last_name}"
  end
end
person = Person.new
```

我们现在根据我们的类创建了一个特定的人，我们现在可以调用这个人的某些方法了。我们现在可以调用 `full_name()` 方法。让我们通过在文件的末尾添加以下行来实现这一点：

```php
person.full_name
```

让我们在 shell 中再次运行我们的脚本，如下所示：

```php
ruby class_syntax.rb
```

然后，我们应该得到以下输出：

```php
James Raynor
```

哇！我们终于给了我们的类一个更实用的用途。现在，我们可以使用类定义来创建任意数量的对象（人）。然而，我们仍然有一个问题。我们的类只允许我们创建名为 James Raynor 的人。我们想要的是具体性，但结果却过于具体。我们需要修改我们的类，以便我们可以创建更通用的对象。所以，让我们通过向构造函数添加参数来实现这一点。我们的代码现在将看起来像以下这样：

```php
# class_syntax.rb
class Person
  def initialize( first_name, last_name )
   @first_name = first_name
   @last_name = last_name
  end
  def full_name
    puts "#{@first_name} #{@last_name}"
  end
end
person = Person.new
person.full_name
```

我们已经向构造函数方法添加了两个参数。我们传递给类的第一个名字将被分配给我们的实例变量`@first_name`，这样它就可以供其他方法使用。对于姓氏也是如此。我们现在必须做出的额外调整是将第一个和最后一个名字传递给构造函数。所以，让我们这么做：

```php
# class_syntax.rb
class Person
  def initialize( first_name, last_name )
   @first_name = first_name
   @last_name = last_name
  end
  def full_name
    puts "#{@first_name} #{@last_name}"
  end
end
jim = Person.new( 'James', 'Raynor' )
jim.full_name
```

现在，我们可以创建任意数量的实例（或对象）。让我们再添加两个人：

```php
# class_syntax.rb
class Person
  def initialize( first_name, last_name )
   @first_name = first_name
   @last_name = last_name
  end
  def full_name
    puts "#{@first_name} #{@last_name}"
  end
end
jim = Person.new( 'James', 'Raynor' )
sarah = Person.new( 'Sarah', 'Kerrigan' )
arcturus = Person.new( 'Arcturus', 'Mengsk' )
jim.full_name
sarah.full_name
arcturus.full_name
```

现在，让我们在我们的 shell 上运行我们的脚本：

```php
ruby class_syntax.rb
```

输出将是以下内容：

```php
James Raynor
Sarah Kerrigan
Arcturus Mengsk
```

我们已经使我们的蓝图更通用，现在我们可以从这个蓝图创建不同的角色。

在我们继续到下一个主题继承之前，我想让我们看看你将来会遇到的一个非常有用的类工具——属性访问器。

## 属性访问器

在我们的类定义中，我们拥有`first_name`和`last_name`属性，同时还有一个`full_name()`方法。然而，如果我们想输出一个人的名字呢？我们可能会尝试以下做法：

```php
jim.first_name
```

然而，这会以以下错误惨败：

```php
class_syntax.rb:23:in `<main>': undefined method `first_name' for #<Person:0x000000014d935460> (NoMethodError)
```

这就是 Ruby 与 PHP 在外观或行为上的不同之处。Ruby 显然可以有一个方法和一个具有相同名称的属性。虽然这从技术上讲并不正确，但为了辩论的需要，让我们假设这是真的，这样我们就可以暂时继续这个练习。让我们创建一个名为`first_name`的方法：

```php
…
  def first_name
    @first_name
  end
…
```

这个方法看起来很奇怪，但让我们记住，Ruby 不需要我们显式地返回一个值，因为它会自动完成。所以，这个方法只是返回`@first_name`中包含的值。虽然很有用，但我们必须为每个定义的属性都这样做。此外，我们只创建了方法来获取值。我们还需要创建一个方法来设置值。然而，我有好消息要告诉你。Ruby 已经通过属性访问器解决了这个问题。属性访问器会自动创建获取和设置值的方法。我们只需要指出我们想要这个“魔法”作用于哪个属性。让我们定义属性访问器，然后利用它们。我们的最终代码应该看起来像这样：

```php
# class_syntax.rb
class Person
  attr_accessor :first_name, :last_name
  def initialize( first_name, last_name )
   @first_name = first_name
   @last_name = last_name
  end
  def full_name
    puts "#{first_name} #{last_name}"
  end
end
jim = Person.new( 'James', 'Raynor' )
sarah = Person.new( 'Sarah', 'Kerrigan' )
arcturus = Person.new( 'Arcturus', 'Mengsk' )
jim.full_name
sarah.full_name
arcturus.full_name
puts jim.first_name
puts sarah.first_name
puts arcturus.last_name
```

让我们再次使用以下命令运行它：

```php
ruby class_syntax.rb
```

我们应该得到以下输出：

```php
James Raynor
Sarah Kerrigan
Arcturus Mengsk
James
Sarah
Mengsk
```

注意我们不需要定义`last_name`方法，但它仍然可用。在你学习 Ruby 的道路上，我保证你会在某个时候遇到`attr_accessor`工具。Ruby 还有`attr_reader`和`attr_writer`，它们将`attr_accessor`本身做的两件事分开成两个方法。如果你想更深入地了解属性访问器，了解它们的确切作用，并查看其他示例，你可能想访问[`www.rubyguides.com/2018/11/attr_accessor/`](https://www.rubyguides.com/2018/11/attr_accessor/)。

你准备好创建更强大的类了吗？那么，让我们跳到下一节。

# Ruby 中的继承

到目前为止，我们已经探讨了 Ruby 实现面向对象范式的一些特性，但我们忽略了一个核心特性，它有助于我们重用代码。继承可以简化为将一个类的特性传递给创建一个新的子类。有了这个新类，我们可以使用父类中的任何特性，创建新的特性，或者自定义从父类继承的特性。继承的语法可能和 PHP 中的语法大不相同，但行为却非常相似。考虑到这一点，让我们看看一些用例，并看看它是如何工作的。

假设我们想要一个可以让我们连接到数据库的类。而不是必须编写所有连接到数据库的功能，我们可以获取一个已经创建的数据库类，创建一个新的类，它继承了所有的数据库功能，然后只专注于创建我们需要的特性。这是使用继承重用代码的一种方式，但让我们用一个更简单的例子来展示继承的实际应用。假设我们想要创建一个用户抽象。用户必须具有名字、姓氏、年龄和电子邮件详情。我们可以从上一节中定义的`Person`类继承特性，并在新的`User`类中只关注缺失的部分。

那么，让我们将我们的`Person`类取名为`inheritance_example.rb`并创建一个包含以下内容的文件：

```php
# inheritance_example.rb
class Person
  attr_accessor :first_name, :last_name
  def initialize( first_name, last_name )
   @first_name = first_name
   @last_name = last_name
  end
  def full_name
    puts "#{first_name} #{last_name}"
  end
end
```

现在，让我们在原始类下面创建一个新的类，名为`User`，并从`Person`类继承。我们将使用`<`操作符来完成这个操作。让我们将以下内容添加到`Person`类的末尾：

```php
# inheritance_example.rb
…
class User < Person
end
```

只用两行代码，我们就创建了一个全新的类，它现在（暂时）的行为与`Person`类完全相同。让我们通过创建一个新的`User`对象来确认这一点。现在，让我们将以下内容添加到文件的末尾：

```php
# inheritance_example.rb
…
user = User.new
```

现在，让我们尝试在我们的 shell 中运行这个脚本：

```php
ruby inheritance_example.rb
```

这应该会输出以下内容：

```php
inheritance_example.rb:4:in `initialize': wrong number of arguments (given 0, expected 2) (ArgumentError)
  from inheritance_example.rb:18:in `new'
  from inheritance_example.rb:18:in `<main>'
```

我们的脚本失败了，但为什么呢？当我们阅读错误信息时，它指出构造函数期望两个参数，但一个都没有提供。从我们之前的执行示例中，我们可以推断出我们必须给构造函数提供名字和姓氏的参数。所以，让我们添加这些参数，同时也要调用`full_name()`方法。我们的代码现在看起来是这样的：

```php
class Person
  attr_accessor :first_name, :last_name
  def initialize( first_name, last_name )
   @first_name = first_name
   @last_name = last_name
  end
  def full_name
    puts "#{first_name} #{last_name}"
  end
end
class User < Person
end
user = User.new
user = User.new("Amit", "Seth")
user.full_name
```

让我们运行这个脚本：

```php
ruby inheritance_example.rb
```

脚本将输出以下内容：

```php
Amit Seth
```

因此，我们已经确认，这个新创建的类已经从`Person`类继承了所有功能。我们根本不需要定义`full_name()`方法，因为它已经可用。此外，构造函数自动将名字和姓氏分配给了我们的`@first_name`和`@last_name`实例变量。再次强调，我们只需要让这个类从`Person`类继承。然而，使用本节开头提供的示例，我们想要添加一个额外的属性，称为`email`。所以首先，我们将为`email`属性添加一个属性访问器。现在我们的`User`类看起来是这样的：

```php
class User < Person
  attr_accessor :email
end
```

我们现在可以使用以下方式为我们的对象分配`em.ail`属性：

```php
user.email = "my@fakemail.com"
```

然而，我们想要做的是将这个分配包含在新的`User`构造函数中。这并不像看起来那么容易，但也不是那么困难。所以，我们首先必须为我们的新`User`类定义一个构造函数。让我们就这样做。现在我们的`User`类看起来是这样的：

```php
class User < Person
  attr_accessor :email
  def initialize( email )
    @email = email
  end
end
```

然而，通过这样做，我们已经覆盖了原始的构造函数方法（初始化器）。但不要绝望，因为原始构造函数仍然可以通过`super()`方法访问。`super()`方法调用原始构造函数，但你必须提供原始的参数数量。所以，为了完成这个例子，我们再次将名字、姓氏和电子邮件添加到我们的构造函数中，并最终调用`super()`方法。我们的最终代码将如下所示：

```php
class Person
  attr_accessor :first_name, :last_name
  def initialize( first_name, last_name )
   @first_name = first_name
   @last_name = last_name
  end
  def full_name
    puts "#{first_name} #{last_name}"
  end
end
class User < Person
  attr_accessor :email
  def initialize( first_name, last_name, email )
    @email = email
    super( first_name, last_name )
  end
end
user = User.new
user = User.new("Amit", "Seth")
user.full_name
puts user.email
```

我们已经成功使用继承重用了`Person`类的功能，并创建了一个名为`User`的新类。当谈论使用继承的类时，你会听到术语“层次结构”。当我们谈论层次结构时，我们指的是我们类在想象中的结构中的位置，在最上面，我们将有最通用的类，在最下面是最具体的类。对于这个例子，`Person`类和`User`类之间的层次关系可能开始变得有意义，其中`Person`类是最通用的类，因此位于顶部。换句话说，用户是人；因此，人必须具有用户和人的属性。反之则不然。一个人不一定是用户。一个人可能是客户，并且有一个不同的用例，我们不需要电子邮件属性。在设计你的 Ruby 类时，如果你考虑到这个层次结构，这将使你更容易确定应该将哪些功能放在哪里，以编写更少的代码，并且不会出现重复不必要的代码的问题。虽然这个例子非常简化，但它向我们展示了构建可重用类是多么容易。

# 摘要

在本章中，我们学习了如何安装、使用和命名（gem）Ruby 库。我们还学习了如何使用 bundler 工具安装 gem，为我们的脚本和程序创建正确运行的环境。最后，我们学习了最基本的面向对象编程（OOP）语法，用于创建、实例化和继承类。现在，我们已准备好开始调试。
