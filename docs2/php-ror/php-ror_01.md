

# 第一章：理解 Ruby 思维方式和文化

Ruby 自 Yukihiro Matsumoto 创立以来已经有了一段相当长的历史。社区对语言的采用当然影响了 Ruby 的关注方向。但，在本质上，Ruby 在你“可以”和“应该”使用它编写程序/脚本的方式上非常直接。每种语言都有其独特的特性，社区将这些特性用于定义什么是良好的实践，什么是被认为是“坏”代码。虽然这可能是完全主观的，但这种主观性为作者最初创建语言的原意铺平了道路，使其成为社区希望语言成为的样子。

Ruby 的设计理念是极其易于阅读、灵活且面向对象。同样，这也适用于由于 Ruby 而产生的技术。我指的是在 Ruby 中创建的框架，例如 Ruby on Rails ([`rubyonrails.org`](https://rubyonrails.org)) 和 Sinatra ([`sinatrarb.com/`](http://sinatrarb.com/))。但我还指的是那些具有相同思维方式的工具，例如 Chef ([`www.chef.io/`](https://www.chef.io/))。所有这些工具都有共同的特性，但最突出的特性是可读性。一旦你进入 Ruby 的“领域”，你就能阅读和理解为纯 Ruby、Ruby on Rails 应用程序 API 或甚至 Chef 脚本（用于管理和配置基础设施）编写的代码。Ruby 并不会自动使你的代码更易于理解或阅读，但它为你提供了使代码更容易阅读的工具。使代码易于理解是专注于手头业务（或爱好）并减少试图理解某些代码在做什么的关键。

但在我们到达那里之前，我们需要转换到 Ruby 思维方式。在本章中，我们将通过涵盖以下主题开始我们的思维之旅：

+   创建可读的 Ruby 代码

+   面向对象的 Ruby

+   编写 Ruby 风格的代码

你下定决心学习一门新的编程语言。恭喜！至少对我来说，我很想赞扬这个决定，并希望它没有花你像我第一次对另一种编程语言产生浓厚兴趣时那么多的时间。一开始，我有点固执和犹豫，看到了 Ruby 的每一个缺点。我最喜欢的说法是，“我可以用 PHP 更容易做到那件事。”但后来，有一天，它突然变得清晰，我再也没有回头。Ruby 已经是我很长时间以来的首选语言。而且我不会试图过分夸大这一点。我拒绝说 Ruby 是最好的编程语言，因为这会回答一个有争议的问题。没有一种编程语言在所有语言中都是普遍优于其他语言的。我能做的是尝试向你展示我为什么喜欢 Ruby。

# 技术要求

要跟随这本书，你需要：

+   Git 客户端

+   rbenv（Ruby 版本管理器，可启用多个 Ruby 版本）

+   Ruby（版本 2.6.10 和 3.0 或更高版本）

+   您可以选择安装任何 IDE 来编辑代码（Sublime, Visual Studio Code, vim, Emacs, Rubymine 等）

所有代码示例都已编写以与 Ruby 版本 2.6.10 和 3.0（或更高版本）兼容。其中一些示例可能需要以前的 Ruby 版本（即 2.6.10）才能与以前的 Ruby on Rails 版本（例如 Ruby on Rails 5）兼容，但为了确保它们与本章中的方式相同，您应该尝试安装最新的 Ruby 版本。您可以从这里获取不同操作系统的安装程序：[`www.ruby-lang.org/en/downloads/`](https://www.ruby-lang.org/en/downloads/).

此外，为了能够使用不同版本的 Ruby，我建议您从以下 GitHub 仓库安装 rbenv：[`github.com/rbenv/rbenv`](https://github.com/rbenv/rbenv).

本书中的代码可在[`github.com/PacktPublishing/From-PHP-to-Ruby-on-Rails`](https://github.com/PacktPublishing/From-PHP-to-Ruby-on-Rails)找到。

# Ruby 的设计意图是像句子一样阅读

说了这么多关于 Ruby 的话，让我们动手实践，从最基本的概念开始。您已经了解了 PHP 变量。变量存储信息，可以在我们的程序中使用和引用。此外，PHP 是一种动态类型语言，这意味着解释我们的 PHP 代码的“引擎”将自动推断该变量内的内容类型。也就是说以下两点：

+   我们不需要定义变量包含的内容类型

+   变量可以在不失败的情况下改变类型

来自 PHP 的您不需要费尽脑筋去学习 Ruby 的新用法或定义变量的新方式，因为 Ruby 的行为完全相同。然而，请注意，在其他强类型语言中，如 Java，变量必须用其将包含的类型定义，并且其类型不能随时间改变。

因此，让我们在 PHP 中玩一些变量：

```php
<?php
$name = "bernard";
$age = 40;
$height_in_cms = 177.5;
$chocolate_allergy = true;
$travel_bucket_list = ["Turkey", "Japan", "Canada"];
```

在这个场景中，Ruby 并没有太大的不同：

```php
name = "bernard";
age = 40;
height_in_cms = 177.5;
chocolate_allergy = true;
travel_bucket_list = ["Turkey", "Japan", "Canada"];
```

对于那些阅读此文档且可能因我没有使用`$`符号而眼睛发红的经验丰富的 PHP 开发者。PHP 和 Ruby 之间的另一个区别是，我们不需要任何标签来表示 PHP 代码，而在 PHP 中，我们使用开头的 PHP 标签（`<?php`）。因此，我们片段之间的主要区别（到目前为止）是我们调用 PHP 代码的方式和引用变量的方式。虽然这是一段有效的 Ruby 代码，但我故意将其写成 PHP 风格，以便让您也一窥 Ruby 的灵活性。Ruby 非常灵活，甚至可以弯曲 Ruby 自己的行为。这种灵活性的一个例子是，虽然我们可以在每行末尾添加分号（`;`），但 Ruby 的最佳实践是省略它们。如果您对 Ruby 的灵活性感兴趣，您可能想查看 Ruby 的元编程。这是一份很好的起点指南：

[`www.rubyguides.com/2016/04/metaprogramming-in-the-wild/`](https://www.rubyguides.com/2016/04/metaprogramming-in-the-wild/)

但我们不要急于求成，因为这个主题真的很复杂——至少对于一个初学者 Ruby 程序员来说是这样。

基于前面的 PHP 代码，现在让我们确定名字是否为空。在 PHP 中，你会使用`empty`内部函数。我们用另一个内部函数`var_dump`包围它，以显示`empty`函数的结果内容：

```php
$name = "Bernard";
var_dump( empty($name) );
```

这将输出以下内容：

```php
bool(false)
```

根据`empty`函数的文档，这是`false`，因为名字不是一个空字符串。现在，让我们在 Ruby 中尝试一下：

```php
name = "bernard";
puts name.empty?;
```

在这里，有几个需要注意的地方。首先想到的是，这几乎就像一个句子一样被阅读。这是 Ruby 社区如何聚集在一起并使用 Ruby 编写人类可读代码的关键点之一。在大多数情况下，你应该避免在代码中添加注释，除非它是版权声明和/或确实需要解释。Ruby 甚至有一种奇怪的方式来编写多行注释。如果我要在我的代码上写多行注释，我必须查找语法，因为我从未使用过那种符号。这并不是说你不能或你不应该这样做。它有它的原因。这仅仅意味着 Ruby 社区很少使用那种符号。要在 Ruby 中添加注释，你只需在行首添加井号符号（`#`）：

```php
# This is a comment
```

正如您从代码片段中的注释中知道的那样，这一行将被 Ruby 忽略。请记住，编程语言，就像 spoken language，会因使用而演变。最好的工具可能因为没有人使用而丢失。学习语言的一部分也涉及到学习工具的使用和最佳实践。这包括了解 Ruby 社区决定不利用什么以及使用什么。因此，尽管社区很少使用多行注释，但所有 Ruby 开发者都会利用其最强大的工具之一：对象。

# 一切都是对象

在阅读之前的代码时，我脑海中浮现的第二件事是我们正在对一个字符串调用一个方法。现在，让我们稍微退后一点，这就是我们开始用新手的眼光审视 Ruby 代码的地方。我们的变量名包含一个字符串。这意味着我们的名字是一个对象吗？嗯，简短的答案是*是的*。在 Ruby 中，几乎所有东西都是一个对象。我知道这可能会感觉像是我们跳过了几章，但请耐心等待。我们将在*第五章*中看到 Ruby 的面向对象语法。现在，让我们通过以下行进一步在我们的代码中探索我们的变量具有哪种类型的对象：

```php
puts name.class();
```

这将返回我们对象所属的类类型（在这个特定案例中，`String`）。我们能够用其他变量做同样的事情，我们会得到类似的价值（`Integer`、`Float`、`TrueClass`或`Array`）。为了进一步证明我的观点，即 Ruby 中几乎一切都是对象，让我们阅读以下示例：

```php
puts "benjamin".class();
```

这也将返回一个`String`类型。所以，当你编写 Ruby 代码时，请记住这一点。现在，让我们回到最初的带有`empty`函数的例子：

```php
name = "bernard";
puts name.empty?();
```

我们还注意到第三点是，我们实际上是在提问。这让我第一次看到它时感到困惑。你如何知道何时提问？Ruby 是否真的如此直观，以至于你可以提问？这是什么魔法？不幸的是，事实远没有代码本身那么神秘。在 Ruby 中，我们可以将函数或方法命名为包含问号符号的一部分，仅为了提高可读性。它对 Ruby 解释器没有特殊的执行或含义。我们只是能够以这种方式命名方法/函数。话虽如此，按照惯例，Ruby 开发者使用问号来暗示我们将返回一个布尔值。在这种情况下，它仅仅回答了关于变量名称是否为空的提问。简单来说，如果名称为空，则提问将返回`true`值，反之亦然。这种命名技术是 Ruby 哲学的一部分，旨在使我们的整个代码可读。此外，这种代码风格贯穿于 Ruby 的许多内部类中。一些附加到数字对象和数组对象的方法就是这种风格的例子。以下是一些示例：

+   `.``odd?`

+   `.``even?`

+   `.``include?`

所有这些示例都是以这种命名方式命名的，仅为了提高可读性，没有其他原因。其中一些甚至在不同的类之间共享，但每个类型都有自己的实现。当我们目前正在查看问号符号时，让我们看一下一个类似的符号：感叹号（`!`）。在 Ruby 开发者中，它有一个稍微不同的含义。让我们用一个例子来看看。

让我们展示名称的大写字母形式。在 PHP 中，我们会写出以下代码：

```php
$name = "bernard";
echo strtoupper($name);
```

在 Ruby 中，可以通过以下代码实现相同的功能：

```php
name = "bernard";
puts(name.upcase());
```

在这两种情况下，这将返回名称的大写形式（`BERNARD`）。然而，如果我们对`name`变量进行任何其他引用，变量将保持不变：

```php
name = "bernard";
puts(name.upcase());
puts(name);
```

这将返回以下结果：

```php
BERNARD
bernard
```

但如果我们添加感叹号（`!`）会发生什么呢？

```php
name = "bernard";
puts(name.upcase!());
puts(name);
```

这将返回名称的大写形式两次：

```php
BERNARD
BERNARD
```

事实上，感叹号符号会永久地修改变量内容。使用感叹号命名的函数被称为`String`和`Array`类：

+   `.``downcase!`

+   `.``reverse!`

+   `.``strip!`

+   `.``flatten!`

我们可以通过阅读它们来推断它们的功能，但现在我们知道了在这个上下文中感叹号符号的含义。使用时要小心，但如果使用场景需要，也不要害羞地使用它们。现在，当你阅读 Ruby 代码时，你会意识到问号（`?`）和感叹号（`!`）符号的存在。

# 转向 Ruby

到目前为止，我们已经看到了一些例子，我们的代码看起来非常类似于 PHP。正如我之前提到的，我故意这样做是为了展示 Ruby 的灵活性。这使得将代码转换为 Ruby 比其他语法与 Ruby 差异很大的语言要容易得多。然而，这仅仅是我们成为 Ruby 开发者旅程的开始。如果我们想要能够像经验丰富的 Ruby 开发者一样阅读和编写 Ruby 代码和代码片段，我们需要了解社区是如何使 Ruby 代码的。简而言之，虽然我们可以将代码编写得类似于其他语言，但我们应该避免这种做法，在这个过程中，了解 Ruby 可以提供什么来使我们的代码越来越易于阅读。

我们将采取的第一个步骤是移除代码中的不必要的语法。为了做到这一点，我们也必须理解我们正在移除的内容的用途。

让我们以我们的原始代码为例：

```php
name = "bernard";
age = 40;
height_in_cms = 177.5;
chocolate_allergy = true;
travel_bucket_list = ["Turkey", "Japan", "Canada"];
```

在 Ruby 中，分号可以用来将多行代码合并成一行，每行用分号分隔。如果我们把名字和例子转换成大写，我们会得到以下内容：

```php
name = "bernard"; name.upcase!(); puts(name);
```

这工作得非常好。但请记住，我们正在努力使我们的代码更易于阅读。这并不更易于阅读。相反。而且，如果我们不打算将整个代码写在一行中，那么让我们保留原始代码片段（多行），并移除每个分号。这开始看起来更像 Ruby：

```php
name = "bernard"
age = 40
height_in_cms = 177.5
chocolate_allergy = true
travel_bucket_list = ["Turkey", "Japan", "Canada"]
```

通过移除未使用的字符，这确实稍微提高了可读性，但我们还没有完成。让我们用另一个例子来实践。让我们编写一个示例，如果 `$chocolate_allergy` 变量的值为 `true`，则将打印出字符串 `This person is allergic to chocolate`。由于我们的 PHP 背景，我们可能会被促使编写类似于 PHP 的代码。在 PHP 中，我们会编写以下内容：

```php
$chocolate_allergy = true;
if($chocolate_ allergy)
{
  echo "This person is allergic to chocolate";
}
```

考虑到这一点，我们会在 Ruby 中编写以下代码：

```php
chocolate_allergy = true
if(chocolate_allergy)
  puts("This person is allergic to chocolate")
end
```

这工作得很好，但它仍然看起来很像 PHP。一个中级 Ruby 开发者可能会写类似以下的内容：

```php
chocolate_allergy = true
puts "This person is allergic to chocolate"
  if chocolate_allergy
```

这使得代码的可读性每秒都在提高。但它也带来了一些新的实践。首先，`puts` 语句没有被括号包围。这是因为，在 Ruby 中，对于函数和方法，括号的使用是可选的。这非常有用，因为它开始看起来像普通的英语。它也适用于具有多个参数的函数。例如，一个实现的函数可能看起来像这样：

```php
add_locations "location 1", "location 2"
```

当然，如果我们需要调用嵌套函数，这会变得很繁琐。让我们看看以下两个函数的例子：

```php
def concatenate( text1, text2 )
  puts text1 + " " + text2
end
def to_upper( text )
  return text.upcase()
end
```

`concatenate` 函数接受两个字符串，并将它们之间用空格连接的字符串打印出来。第二个函数只是将输入字符串转换成大写字符串并返回值。如果我们没有使用括号，这可能会成为问题。如果我们想将两个字符串连接起来，并将每个字符串转换成大写字符串，我们可以尝试以下操作：

```php
concatenate to_upper "something", to_upper "else"
```

但我们会失败得很惨，因为 Ruby 解释器不知道 `"something"` 是 `to_upper` 函数的参数。我们可以通过括号轻松解决这个问题：

```php
concatenate to_upper("something"), to_upper("else")
```

注意这个知识，就像其他所有事情一样，如果过度使用，可能会损害我们代码的可读性。在决定是否使用括号时，我们需要考虑两个额外的点。第一个是，这些规则也适用于函数的定义。因此，`concatenate` 函数可以这样定义：

```php
def concatenate text1, text2
  puts text1 + " " + text2
end
```

第二点是，这个规则也适用于没有参数的函数——也就是说，我们也可以从它们中移除括号。让我们以下面的例子作为参考：

```php
return text.upcase()
```

现在将变成以下内容：

```php
return text.upcase
```

更重要的是，使用带问号的方法和破坏性方法（`!`）现在对可读性来说非常合理。

让我们看看以下内容：

```php
name.empty?()
```

这变成如下所示：

```php
name.empty?
```

作为另一个例子，让我们看看以下内容：

```php
name.upcase!()
```

现在变成如下所示：

```php
name.upcase!
```

我们现在要讨论的最后一点与 Ruby 在返回值方面的行为有关。虽然方法可以使用 `return` 关键字显式返回一个值，但 Ruby 不需要这个关键字。在函数和方法内部，Ruby 会自动返回最后引用的值。让我们用以下例子来说明这一点：

```php
def to_upper text
  return text.upcase
end
```

那个例子将变成这样：

```php
def to_upper text
  text.upcase
end
```

你会看到很多这样的 Ruby 代码。一开始可能会觉得令人畏惧和困惑，但一旦你理解了 Ruby 在做什么，它就会变得更有意义。

如你现在可能已经意识到的，Ruby 的创造者非常重视这些工具，以便更容易编写可读的代码。社区也采纳了这种理念并将其付诸实践。我们不仅看到使用这些约定和规则来提高可读性的代码，还看到 Ruby 程序员采用其他约定，虽然它们本身不是 Ruby 规则的一部分，但在特定上下文中使用时却非常合理。我指的是变量和方法命名。因为 Ruby 开发者会努力使他们的代码看起来像普通英语，所以他们会在如何命名方法和变量以使代码更易读上花费大量时间。因此，在 Ruby 中更常使用蛇形命名法，因为它有助于提高可读性。考虑到这一点，让我们看看这个例子：

```php
chocolate_allergy = true
puts "This person is allergic to chocolate"
  if chocolate_allergy
```

我们仍然可以提升其可读性。这不仅仅是一个语法上的改变；它还涉及到变量名甚至定义一个方法，只是为了提高可读性。因此，一位经验丰富的 Ruby 开发者可能会为这个例子编写以下最终的代码片段：

```php
def say text
  text
end
is_allergic = true
say "This person is allergic to chocolate" if is_allergic
```

正如你所见，Ruby 开发者会尽力使代码尽可能像普通英语一样易于阅读。当然，这并不总是可行的，有时它可能并不实用，因为它有时需要投入大量的努力来编写即使是简单的东西，但大部分情况下，只要它是可读的，遵循这些指南，其他开发者会感谢你，而不仅仅是 Ruby 开发者。

# 摘要

在本章中，我们介绍了 PHP 和 Ruby 之间的语法差异和相似之处，Ruby 的可读性工具，以及 Ruby 的语法灵活性。我们还学习了问号（`?`）和感叹号或叹号符号（`!`）。走到这一步意味着你确实在尝试重用你之前的编程技能，但使用的是一门新语言：Ruby。这是一个很好的开始，因为你可以跳过学习一门新语言中最困难的部分之一：逻辑部分。虽然我们只看到了 Ruby 的表面，更重要的是，我们清楚地看到了 Ruby 开发者编写代码时的思维方式。我们了解到，对于 Ruby 开发者来说，可读性是最重要的。我们不仅使用语法和语言结构来实现这一点，我们还使用对象来增加代码的可读性。它读起来越像句子，就越好。我们查看了一些简单的 Ruby 示例，虽然你可以跟随，但这并不是练习的目的。它更多的是为了激发你的兴趣。

为了继续学习这条学习路径，我们现在需要适当的工具来开始编写和运行 Ruby 代码。在下一章中，我们将探讨安装 Ruby 的不同方法以及设置我们的本地环境，这样我们就可以开始学习 Ruby 的真实示例，并最终跟随这个过程。
