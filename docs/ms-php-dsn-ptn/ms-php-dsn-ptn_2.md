# 第二章 反模式

这就是我们开始讨论反模式的地方；在你满怀希望地认为我将告诉你一些了不起的东西，可以在不使用设计模式的情况下奇妙地简化你的代码之前，我在这里不会这样做（我是否提到过我擅长粉碎希望和梦想？）。简而言之，反模式是你不想在你的代码中出现的东西。

说到粉碎希望和梦想，如果你曾经有过初级开发人员，反模式也是教授应该避免的方法论的好方法。学习反模式还可以提高代码审查的效率；你可以有一个外部来源来咨询代码质量，而不是基于个人意见来辩论代码质量。

反模式构成了一种解决经常出现的问题的可怕方法，通常是无效的，并且有很高的反生产力风险。它们可能会产生技术债务，因为开发人员必须后来努力重构以解决最初的问题，但希望使用更具弹性的设计模式。

我们都遇到过意大利面代码；我和一个合同开发人员一起工作时，他在面对高技术债务的产品负责人时大声喊道：“意大利面太多了，我可能还不如开一家餐厅！”意大利面代码是指程序的控制结构几乎无法理解，因为它太混乱和过于复杂，可以被描述为一种反模式。在 PHP 5.3.0 中的一个主要批评是语言中 goto 操作符的实现。事实上，批评它们的实施的人声称，goto 操作符将为 PHP 中的意大利面代码提供另一个借口。

在 PHP 中，goto 语句曾引起了很大的争议，甚至有人将其报告为一个 bug，并表示：“PHP 5.3 包括 goto。这是一个问题。说真的，PHP 在没有 goto 的情况下已经走到了这一步，为什么要把这种语言变成公共威胁呢？”

除此之外，bug 报告的提交者将预期结果列为：“世界将会结束”，实际结果是“世界已经结束”。尽管如此，在 PHP 中，goto 操作符受到严格限制，因此你不能随意跳入和跳出函数。有些人还认为它们在有限状态机中很有用（基本上是基于多个输入的二进制输出），但这也是有争议的；所以我会让你自己对它们做出判断。

你可能也经历过复制粘贴编程，整个代码块被复制并粘贴到程序中；这是另一个糟糕软件设计的例子。实际上，开发人员应该设计他们的软件，以创造通用的解决方案来解决问题，而不是复制、重构和粘贴代码来适应某种情况。

我将在本章中介绍为什么学习反模式很重要。在本章中，我将讨论传统的与反模式相关的软件设计，还有与反模式相关的网络基础设施和管理风格。除此之外，我想讨论一些 PHP 特定的反模式，或者 PHP 中的缺陷，这可能需要你在自己的代码中进行补偿。

本书包含了一个专门的章节，讲述了重构的过程；如果重构的过程对你感兴趣，这一章将帮助你打下你可能想要开始思考的理念基础；除此之外，专门讲述设计模式的章节可能会帮助你意识到你最终可能要达到的代码。在专门讲述重构的章节中，我们还将涵盖一些代码异味，这可以帮助你发现你正在维护的代码库中的反模式。

# 为什么反模式很重要

大多数程序员都来自采用某种反模式的背景，直到最终意识到它无法扩展或效果不佳。当我 17 岁时，在我的第一份学徒开发人员工作中，我会被送到伦敦，从周一到周五，以某种方式把我的西装和完全黑色的衣服压缩成一个令人惊讶的小手提箱，然后学习软件开发。周五，我们经常在中午 12:00 放半天假，但我会提前预订公司的火车票，所以我会在快餐店或咖啡店里工作简单的项目。每周，当我回来尝试扩展其中一个解决方案时，我会意识到新的可扩展性问题和代码质量问题。当然，我以前也做过开发，但这些主要是处理全新的、非常简短的编程任务，使用预先制作的框架，或者处理已经完成架构的遗留代码（或者，我现在意识到，是用非常迟钝的刀切割的）。扩展自己代码的这个学习过程很棒；我迅速教会自己如何更好地设计软件。作为人类，我们经常对某个主题了解不够，以至于不知道自己知道的有多少（我发现这在那些管理软件开发人员但从未自己编写过任何代码的人身上非常真实）；在牢记这一点的同时，我们应该记住，我们永远不会超越从自己的错误中学习。虽然这非常重要，但教会自己记录的反模式也是为了从他人的错误中学习至关重要。

我曾经是一位开发人员的技术负责人和导师，他从犯错中学习受到了最严厉的对待。在我和这位开发人员的第一次评估中，我的人力资源对手告诉我，每次他犯错时，之前的技术负责人和人力资源负责人都会把他拉到会议室进行正式的纪律程序。这两个人的技术知识非常有限，而且在管理开发人员方面完全无能（以至于他们是那种生活在自己的泡泡中，对更成功的环境中人们的工作方式一无所知，基本上陷入了没有前途的职业生涯，没有知识去做任何有意义的事情）。等他们离开时，这位可怜的开发人员的自信已经被压垮到了一个程度，以至于他对网络没有真正的职业抱负，也不急于学习。在你的职位上快乐并没有错。正如我的一位前上司在我告诉他一些非常个人的事情后所说的，“最重要的是你快乐”。是的，让世界运转需要很多人，但一旦你把自己置于指导或管理其他开发人员的位置，你就有义务使自己保持在游戏的前沿。如果你是一个经理，你应该知道如何有效地做好你的工作。我遇到的最好的人事经理是那些拥有丰富知识的人，他们不断更新最新的管理方法，就像我喜欢不断更新 PHP 核心和社区最新最伟大的方法一样。在写这本书的过程中，我的项目和人事管理知识有所提高，但仍有很长的路要走，因此我不会在没有先教育自己的情况下接受这样的线路管理职责。在我工作的公司，实际上以恐吓作为管理策略，我曾经向部门负责人提到过这一点，他回答说“我们并不是说这是最好的做法”；如果这是真的，那么为了公司的利益，肯定应该采取一些措施来解决这个问题！这并不是整个公司都是这样；其他部门有着非常不同的态度，事实上，技术总监曾经就这个问题和知道自己不知道的重要性做了一个技术讲座。公司的 CEO 也和我开始了类似的对话，说他知道自己不知道。旧的做法很难改变，但至少他们已经开始迎接变革的风。

所以除了发牢骚（我确实喜欢发牢骚），我为什么要谈论这个？我的观点是你的态度很重要。关于这个问题，我最喜欢的一句话是“如果你把你的开发人员当作白痴，他们很快就会变成白痴”。让我进一步说一下：

+   学生的糟糕表现往往反映了老师的糟糕表现。

+   每个人都会犯错，但错误失控是管理者愚蠢行为的错，而不是开发人员的错。

+   白痴会吸引白痴。如果你在自己的专业知识领域是个白痴，那么你很可能会招募更多的白痴。

+   如果你在工作场所实行恐惧统治，你就是个白痴，害怕被发现。

+   如果你不知道自己知道的有多少，也不寻求有效地消除自己的无知，你就是个白痴。

+   如果你把你的开发人员当作白痴，那么你就是个白痴。

简而言之，了解自己知道的有多少，然后成长。听起来很残酷，但这是事实。我们都是无知的，我们无法知道一切。有效地利用我们自己的知识与他人的知识是成功的关键。认识到自己的无知是关键。例如，去年我决定，我在基础计算机科学方面的知识不够广泛，无法满足自己对它的需求，也无法满足我指导的人的需求；因此，我决定去读计算机科学的兼职硕士学位。学习过程非常棒，教会了我之前不知道存在的计算机科学领域。

一些软件开发人员使用其他人的工作，没错，WordPress 或 Drupal 的开发可以给你一个快乐和富有成效的职业，但你会发现为自己建立和设计东西是一次很好的学习经历。在传统的工程环境中工作过后，我已经被说服，计算机科学的坚实理论基础对软件工程师是非常有益的。事实上，理解计算机科学基本原理所需的知识体系实际上是相当容易掌握的。当然，在很多方面，我是在对已经信奉的人说教；如果你正在阅读这本书，你可能理解需要更深入的理论计算机科学知识基础，但请不要读完这本书就停止积极学习。继续制定计划来提高你的知识，努力改进我们头脑中存储的信息。

经常有人说“在盲人国度，有一只眼睛的人是国王”；较小的开发团队在良好的软件开发方面可能经常缺乏基础（也许是因为没有必要），而一些陷入过去的较大的开发环境最终可能陷入同样的境地。在这方面，知识变得更加珍贵，对开发人员了解软件开发同样重要。

反模式不仅仅是你的团队可以学会避免的东西；良好的软件开发需要对编程语言和软件开发的理论理解有坚实的了解。

最后，让我从 SourceMaking 的一篇文章中引用这句话：

> “以架构为驱动的软件开发是构建系统的最有效方法。架构驱动方法优于需求驱动、文档驱动和方法论驱动方法。项目通常成功是尽管方法论，而不是因为它。”

情绪发泄完毕。让我们来谈谈一些反模式。

# 非自行开发综合症

密码学可以教给我们关于软件的一个非常重要的教训；这对于克尔克霍夫原则尤其如此。该原则陈述了这一点：

> “即使系统的一切都是公开知识，一个加密系统也应该是安全的，除了密钥。”

这是由克劳德·香农改编的，称为香农定律：

> “应该在假设敌人立即完全熟悉它们的情况下设计系统。”

简而言之，为了拥有一个安全的系统，它不应该只因为没有人知道它是如何实现的而被视为安全（“安全通过模糊性”）。如果你通过模糊性来保护你的钱，你会把它埋在树下，希望没有人会找到它。而当你使用真正的安全机制，比如把你的钱放在银行的保险柜里，你可以把安全系统的每一个细节都公开，只需要保密保险柜的钥匙，其他细节都可以是公开的信息。如果有人找到了你的保险柜的钥匙，你只需要改变组合，而如果有人真的找到了你的钱埋在树下，你就必须挖出钱，找别的地方放。

只通过模糊性来保护安全是一个坏主意（尽管并不总是坏主意）。正如你可能知道的，当你把密码存储在数据库中时，你应该使用一种单向的加密算法，称为**哈希算法**，以确保如果数据库被盗，没有人能够使用数据库中的数据找到用户的原始密码。当然，在现实中，你不应该只是对密码进行哈希处理，你应该对其进行盐处理，并使用诸如 PBKDF2 或 BCrypt 的算法，但本书不是关于密码安全的。

然而，情况的现实是，有时候，当开发人员真的费心去对密码进行哈希处理时，他们决定创建自己的密码哈希函数，这些函数很容易被逆向，并且只有通过不知道算法的模糊性来保护。这是*非本地研发*（NIH）综合症的一个完美例子；开发人员没有使用备受尊重的密码哈希库，而是决定自己创建，假装自己是一个密码学家，却不理解这样的决定的安全影响。

值得庆幸的是，PHP 现在让对密码进行哈希处理变得非常容易；`password_hash`函数和`password_verify`函数使这变得非常容易，而`password_needs_rehash`函数甚至可以告诉你何时需要重新计算哈希。尽管如此，我岔开了话题。

那么，什么是 NIH 综合症？NIH 综合症是指对组织或个人开发者自身能力的虚假自豪感导致他们建立自己的解决方案，而不是采用更优秀的第三方解决方案。重新发明轮子不仅成本高昂、不必要，并且会增加不必要的维护开销；它也可能非常不安全。

也就是说，如果解决方案是封闭源和封锁的，那么最好避免使用它们。这样做也可以避免供应商锁定和对业务灵活性的限制。

NIH 综合症依赖于现有解决方案的良好性能和达到预期。使用第三方库并不是不检查其代码质量的借口。

为开源解决方案做贡献是缓解这些问题的好方法。对现有库有改进的空间？分叉它，提出合并的修改。没有符合你需求的功能的库？那么你可能需要考虑编写自己的库并发布它。

我将以这一节结束，说世界已经变得多元化；人们不再寻求一个技术堆栈来满足他们所有的需求；如今人们追求的是最适合工作的最佳工具。值得考虑如何利用这一事实来使自己受益。

## 使用 Composer 的第三方依赖

Composer 使管理第三方依赖变得非常容易。在第一章中，*为什么“优秀的 PHP 开发人员”不是一个自相矛盾的词*，我简要描述了如何使用 Composer 进行自动加载。自动加载从 PHP 5.1.2 以来就作为核心功能得到支持，但 Composer 的伟大之处在于你还可以用它进行依赖管理。Composer 可以根据你指定的版本约束有效地获取你需要的依赖项。

让我们从以下`composer.json`文件开始：

```php
{ 
  "autoload": { 
    "psr-4": { 
      "IcyApril\\ChapterOne": "src/" 
    } 
  } 
} 

```

所以让我们拉取一个依赖项：

```php
{ 
  "autoload": { 
    "psr-4": { 
      "IcyApril\\ChapterOne": "src/" 
    } 
  }, 
  "require": { 
    "guzzlehttp/guzzle": "⁶.1" 
  } 
} 

```

请注意，我们所做的只是添加了一个`require`参数，指定我们想要的软件。没有手动将文件粘贴到你的项目或根目录，或者使用 Git 中的子模块！

在这种情况下，我们拉取了 Guzzle，一个用于 PHP 的 HTTP 库。

Composer 默认从一个名为**Packergist**的中央仓库查询仓库，该仓库汇总了你可以从各种版本控制系统（如 GitHub、BitBucket 或其他仓库主机）安装的软件包。如果你愿意，Packergist 就像一个电话簿，将 Composer 对代码仓库的软件包请求连接起来。

也就是说，Composer 不仅支持 Packergist 仓库。为了支持开源精神，它支持来自各种 VCS 系统（如 Git/SVN）的仓库，无论它们托管在何处。

让我们看一下以下`composer.json`文件：

```php
{ 
  "autoload": { 
    "psr-4": { 
      "IcyApril\\ChapterTwo": "src/" 
    } 
  } 
} 

```

让我演示一下如何在没有在 Packergist 上的情况下包含一个来自 BitBucket 的仓库：

```php
{ 
  "autoload": { 
    "psr-4": { 
      "IcyApril\\ChapterOne": "src/" 
    } 
  }, 
  "require": { 
    "IcyApril/my-private-repo": "dev-master" 
  }, 
  "repositories": [ 
    { 
      "type": "vcs", 
      "url": "git@bitbucket.org:IcyApril/my-private-repo.git" 
    } 
  ] 
} 

```

就是这么简单！你只需指定你想要从中拉取的仓库，Composer 就会完成剩下的工作。使用其他版本控制系统也同样简单：

```php
{ 
  "autoload": { 
    "psr-4": { 
      "IcyApril\\ChapterOne": "src/" 
    } 
  }, 
  "require": { 
    "IcyApril/myLibrary": "@dev" 
  }, 
  "repositories": [ 
    { 
      "type": "vcs", 
      "url": "http://svn.example.com/path/to/myLibrary" 
    } 
  ] 
} 

```

有点厚颜无耻，Composer 甚至可以支持 PEAR PHP 仓库：

```php
{ 
  "autoload": { 
    "psr-4": { 
      "IcyApril\\ChapterOne": "src/" 
    } 
  }, 
  "require": { 
    "pear-pear2.php.net/PEAR2_Text_Markdown": "*", 
    "pear-pear2/PEAR2_HTTP_Request": "*" 
  }, 
  "repositories": [ 
    { 
      "type": "pear", 
      "url": "https://pear2.php.net" 
    } 
  ] 
} 

```

在你对`composer.json`文件进行更改后更新依赖项的简单方法就是运行`composer update`。

请注意，你不能仅使用`composer dump-autoload`来更新外部依赖项。原因是`dump-autoload`将仅更新你的自动加载器的类映射。它实质上是更新它需要自动加载的类的列表；它不会去拉取新的依赖项。

偶尔在使用 Composer 并拉取依赖项时，Git 可能会说你需要生成一个 GitHub 身份验证密钥。这是因为如果你在本地机器上安装了 Git，Composer 将会通过版本控制系统克隆依赖项；然而，偶尔，如果它从 GitHub 克隆仓库，你可能会遇到它的速率限制。如果发生这种情况，没有必要惊慌。Composer 会给你关于如何实际获取 API 密钥的指示，这样你就可以在没有速率限制的情况下继续进行。

解决这个问题的一个简单方法就是生成一个本地 SSH 密钥，然后将你的公钥放入你的 GitHub 账户。这样，当你从 GitHub 克隆到你的本地机器时，你就不会面临任何速率限制，也不需要设置 API 密钥。

为了在 Linux/Mac OS X 机器上生成 SSH 密钥，你可以使用`ssh-keygen`命令，它将创建一个你可以用于 SSH 身份验证的公钥和私钥，包括 Github 或 BitBucket。这些密钥（通常）将存储在`~/.ssh`目录中，注意波浪号（`~`代表你的主目录）。因此，为了将你的密钥打印到你的终端窗口中，运行`cat ~/.ssh/id_rsa.pub`命令。注意`.pub`后缀表示`id_rsa.pub`是你可以公开分享的公钥。你不应该分享你的私钥，通常只命名为`id_rsa`。在 Windows 上，你可以使用一个名为**PuttyGen**的 GUI 工具来生成公钥和私钥。

一旦您获得了公钥和私钥，您可以简单地将它们放在 GitHub 上，方法是访问 GitHub 网站，转到设置菜单中的 SSH 密钥页面，粘贴您的密钥，然后保存。

对于后续更新，`composer update`将根据`composer.json`中定义的所有依赖项的最新版本进行更新。如果您不想这样做，还有另一种选择；运行 Composer `dump-autoload`将仅重新生成需要包含在项目中的 PSR-0/PSR-4 类的列表（例如，您添加、删除或重命名了一些类）。

Composer 还支持私有存储库，允许您有效地管理跨多个项目的代码重用。另一个关键好处是 Composer 会自动生成一个锁定文件，您可以将其与项目一起提交。这使您能够有效地管理在特定时间点安装的依赖项的确切版本。

Composer 使管理第三方依赖项变得简单而有效。一些关键库已经通过 Composer 可用，例如 PHPUnit，但还有一些其他很棒的库可以让您的生活更轻松。在 Composer 上，我最喜欢的两个数据库库是 Eloquent（来自 Laravel 的数据库 ORM 系统，您可以在`illuminate`/`database`找到）和 Phinx（一个数据库迁移/填充系统，您可以在`robmorgan`/`phinx`找到）。除此之外，还有一些来自 Packergist 的各种 API 的 SDK 可用（Google 发布了一些其 SDK，还有一些更具体的 SDK，例如用于从您的 PHP 应用程序发送短信的 Twilio SDK）。

Composer 允许您为特定环境指定依赖项；假设您只想在开发环境中引入 PHPUnit...那就没问题！

# 上帝对象

上帝对象是糟糕的软件设计和糟糕的对象导向的诱人结果。

本质上，**上帝对象**是一个具有太多方法或太多属性的对象；本质上，它是一个知识过多或做得过多的类。上帝对象很快就会与应用程序中的许多其他代码紧密耦合。

那么这到底有什么问题呢？简而言之，当您的一小段代码与每一小段其他代码都紧密联系在一起时，您很快就会发现维护成为一场灾难。如果您为上帝对象中的一个用例调整了方法的逻辑，您可能会发现它对另一个元素产生了意想不到的后果。

在计算机科学中，采用分而治之的策略通常是一个好主意。通常，大问题只是一系列小问题。通过解决这一系列小问题，您可以迅速解决整体问题。对象通常应该是自包含的；它们只应该了解自己的问题，并且只应该解决一组问题，即自己的问题。任何与此目标无关的东西都不应该属于该类。

可以说，与物理对象相关的对象应该被实例化，而与物理对象无关的对象应该是抽象类。

上帝对象作为反模式的反面是在开发嵌入式系统时。嵌入式系统用于处理从计算器到 LED 标识的任何数据；它们是基本上是自包含计算机且成本相当低的小芯片。在这种用例中，由于计算能力受限，您经常会发现编程优雅和可维护性变得边缘化。轻微的性能提升和控制的集中化可能更重要，这意味着使用上帝对象可能是合理的。幸运的是，PHP 极少用于编程嵌入式系统，因此您极不可能陷入这种特殊情况。

处理这些类的最有效方法是手动将它们拆分为单独的类。

另一个反模式，称为*害怕添加类*，也可能在其中发挥作用，以及未能加以缓解。这是开发人员不愿意创建必要的类。

所以，这是一个 God 类的例子：

```php
<?php 
class God 
{ 
  public function getTime(): int 
  { 
    return time(); 
  } 

  public function getYesterdayDate(): string 
  { 
    return date("F j, Y", time() - 60 * 60 * 24); 
  } 

  public function getDaysInMonth(): int 
  { 
    return cal_days_in_month(CAL_GREGORIAN, date('m'), date('Y')); 
  } 

  public function isCacheWritable(): bool 
  { 
    return is_writable(CACHE_FILE); 
  } 

  public function writeToCache($data): bool 
  { 
    return (file_put_contents(CACHE_FILE, $data) !== false); 
  } 

  public function whatIsThisClass(): string 
  { 
    return "Pure technical debt in the form of a God Class."; 
  } 
} 

```

因此，正如你所看到的，在这个类中，我们基本上结合了许多不相关的方法。为了解决这个问题，我们可以将这个类分成两个子类，一个是`Watch`类，另一个是`CacheManager`类。

这是`Watch`类；这个类只是用来以各种格式显示时间：

```php
<?php 

class Watch 
{ 
  public function getTime(): int 
  { 
    return time(); 
  } 

  public function getYesterdayDate(): string 
  { 
    return date("F j, Y", time() - 60 * 60 * 24); 
  } 

  public function getDaysInMonth(): int 
  { 
    return cal_days_in_month(CAL_GREGORIAN, date('m'), date('Y')); 
  } 
} 

```

这是`CacheManager`类；这个类将所有缓存功能分离出来，因此它与`Watch`类完全分离：

```php
<?php 
class CacheManager 
{ 
  public function isCacheWritable(): bool 
  { 
    return is_writable(CACHE_FILE); 
  } 

  public function writeToCache($data): bool 
  { 
    return (file_put_contents(CACHE_FILE, $data) !== false); 
  } 
} 

```

# PHP 源中的环境变量

经常你会在 GitHub 上遇到一个项目，你会注意到原始开发人员留下了一个包含（在最好的情况下）无用的数据库信息或（在最坏的情况下）非常重要的 API 密钥的`config.php`文件。

当这些文件不小心被版本化时，它们通常会被塞进一个`.gitignore`文件中，并附上一个示例文件供开发人员根据需要修改。一个这样做的平台的例子是 WordPress。

有一些小的改进，比如将核心配置放在一个 XML 文件中，这个文件被埋在一些不相关的配置中。

我发现在 PHP 中管理环境变量通常有两种好方法。第一种方法是将它们放在`root`文件夹中的一个文件中，格式可以是 YML，并根据需要读取这些变量。

第二种方法，我个人更喜欢的方法，是一个名为`dotenv`的库实现的方法。基本上，发生的情况是创建一个`.env`文件并将其放在项目的房间里。为了从这个文件中读取配置，你只需要调用`env()`函数。然后，你可以将这个文件添加到你的`.gitignore`文件中，这样当你从开发环境推送并拉到各种其他服务器配置时，这个过程会变得更容易。除此之外，你还可以在 Web 服务器级别指定环境变量，从而确保额外的安全级别，也使管理变得更容易。

所以，例如，如果我的`.env`文件有一个`DB_HOST`属性，那么我可以使用`env('DB_HOST');`来访问它。

如果你选择了`dotenv`的路线，请确保你的`.env`文件不会从文档根目录公开可见。要么将它放在公共 HTTP 目录之外（例如，在上一级），要么在 Web 服务器级别限制对它的访问（例如，限制权限，或者如果你使用 Apache，使用你的`.htaccess`文件限制对它的访问）。

在撰写本文时，你可以通过简单运行以下命令来要求这个库：

```php
**composer require vlucas/phpdotenv**

```

**软代码**也经常是一个反模式，通过使用配置文件来采用。这是你开始将业务逻辑放在配置文件中而不是源代码中；因此，值得提醒自己要考虑什么时候真正需要配置导向。

# 单例（以及为什么你应该使用依赖注入）

单例是只能被实例化一次的类。在一个应用程序中，你实际上只能有一个`Singleton`类的对象。如果你以前从未听说过单例，你可能会跳起来想“是的！我有一百万个用例可以用这个！”好吧，请不要。单例只是糟糕透了，可以有效地避免使用。

因此，在 PHP 中，`Singleton`类看起来像这样：

```php
<?php 

class Singleton 
{ 

  private static $instance; 

  public static function getInstance() 
  { 
    if (null === static::$instance) { 
      static::$instance = new static(); 
    } 

    return static::$instance; 
  } 

  protected function __construct() 
  { 
  } 

  private function __clone() 
  { 
  } 

  private function __wakeup() 
  { 
  } 
} 

```

因此，以下是应该避免这样做的原因：

+   它们本质上是紧密耦合的，这意味着它们很难进行测试，例如使用单元测试。它们甚至在应用程序的生命周期中保持它们的状态。

+   它们通过控制自己的创建和生命周期来违反单一责任原则。

+   从根本上讲，这会导致你将应用程序的依赖关系隐藏在一个`global`实例中。你再也不能有效地跟踪你的代码中的依赖关系，因为你无法跟踪它们被注入为函数参数的位置。如果需要分析依赖链，这将使其变得无效。

也就是说，有些人认为它们可以是资源争用的有效解决方案（在这种情况下，你只需要一个资源的单个实例，并且需要管理该单个资源）。

## 依赖注入

依赖注入是对单例模式的解药。所以，假设你有一个名为`Transaction`的类。作为类的构造函数，它接受名为`$creditCardNumber`和`$clientID`的参数，因此我们可以构造对象如下：

```php
**$order = new Transaction('1234 5678 9012 3456', 26);**

```

使用依赖注入，我们将传入`$creditCard`和`$client`的对象，它们将是信用卡和客户的类的实例。如果你使用 ORM，这可能是一个数据库模型类：

```php
**$order = new Transaction($clientCreditCard, $client);**

```

# 数据库作为 IPC

在写作时，我目前正在大西洋上空，从伦敦飞往旧金山，这可能是一件好事，因为这意味着我之前与之合作过的一些开发人员已经无法接触到我的颈部。

让我为你澄清一下；你的数据库不是一个消息队列系统。你不要用它来安排作业或排队等待完成的任务。如果你需要做这样的事情，使用一个队列系统。你的数据库是用来存储数据的...提示就在名字里；不要把临时消息塞进去。

有很多原因说明这是一个坏主意。一个主要问题是，在数据库中，没有真正的方法来不执行一个策略，以确保不会发生双重读取，这是通过利用行锁来实现的。这反过来导致进程（无论是传入还是传出）被阻塞，这又导致处理只能以串行方式进行。

此外，为了检查是否有工作要做，你最终基本上要计算数据库中的数据行数，看是否有工作要做；你要持续进行这个操作。MySQL 不支持推送通知；不像 PostgreSQL 那样有`NOTIFY`命令来配合`LISTEN`通道。

还要注意的是，当你将作业队列与存储真实数据的数据库表合并时，每次完成作业并更新标志时，都会使缓存失效，从而使 MySQL 变得更慢。

简而言之，这会导致你的数据库性能变差，并可能迫使它将关键消息放缓到停滞状态。你必须小心，不要让这个功能悄悄地将你的数据库变成一个作业队列；相反，只使用数据库来存储数据，并在扩展数据库时牢记这一点。

RabbitMQ 提供了一个带有一些出色的 PHP SDK 的开源队列系统。

# 自增数据库 ID

数据库自增是我非常沮丧的事情；几乎每个 PHP/MySQL 初学者教程都教人们这样做，但你真的不应该这样做。

我有尝试对自增数据库 ID 进行分片的经验，这很混乱。假设你将数据库分片，使数据集分布在两个数据库服务器上...你怎么能指望有人来扩展自增 ID 呢？

MySQL 现在甚至提供了 UUID 函数，允许你生成具有强熵的良好 ID，这意味着它在`int`数据类型的表上也具有更高的理论限制。

为了使用 UUID 函数，数据库表理想上应该是 CHAR(20)。

# 模拟服务的 Cronjob

这是我个人的憎恶。开发人员需要一个无限运行的服务，所以他们只需启用一个永不结束的 cronjob，或者只需设置一个运行非常频繁的 cronjob（比如每隔几秒运行一次）。

cronjob 是在预定时间运行的计划任务。这不仅从架构的角度看很混乱，而且扩展性很差，监控起来也很糟糕。

一个不断处理的任务应该被视为守护进程，而不是基于 cronjob 运行的东西。

**Monit**是 Linux 系统中的一个工具，允许您模拟服务。

您可以使用`apt-get`命令安装 Monit：

```php
**sudo apt-get install monit**

```

安装 Monit 后，您可以将进程添加到其配置文件中：

```php
**sudo nano /etc/monit/monitrc**

```

然后可以通过运行`monit`命令来启动 Monit。它还有一个`status`命令，因此您可以验证它是否仍在运行：

```php
**monitmonit status**

```

您可以在[`www.mmonit.com`](http://www.mmonit.com)了解更多关于 Monit 的信息，并了解如何配置它。对于每个专注于 DevOps 的开发人员来说，这是一个非常有价值的工具。

# 软件代替架构

通常，开发人员会试图在软件开发层面纠正系统的架构问题。虽然这有用，但我非常赞成在不必要的情况下避免这种做法。将问题从软件架构层移至基础架构层具有其优势。

例如，假设您需要代理特定 URL 端点的请求到另一台服务器。我认为最好在 Web 服务器级别完成这项工作，而不是编写一个 PHP 代理脚本。Apache 和 Nginx 都可以处理反向代理，但编写一个库来做这个可能意味着您会遇到一些未知的问题。您有没有考虑过如何处理`HTTP PUT`/`DELETE`请求？错误处理呢？假设您的库很完美，性能如何？一个 PHP 代理脚本真的比使用低级系统工程语言编写的 Web 服务器级别代理更快吗？在 Web 服务器配置中写一两行肯定比在 PHP 中编写整个代理脚本更容易实现。

以下是在 VirtualHost 中创建代理的示例。以下配置作为 Apache VirtualHost 将允许您将`test.local/api`中的所有内容重定向到`api.local`（在 Nginx 中更容易）：

```php
<VirtualHost *:80> 
    ServerName test.local 
    DocumentRoot /var/www/html/ 
    ProxyPass /api http://api.local 
    ProxyPassReverse /api http://api.local 
</VirtualHost> 

```

这比在 PHP 库中维护成千上万行代码来模拟 ProxyPass Apache 模块中已经可用的功能要容易得多。

我听说过对微服务的批评，认为它们试图将问题从软件开发层移至基础架构层，但我们真的在说这总是一件坏事吗？

是的，软件开发人员有兴趣在软件开发层面做事情，但通常值得让自己了解一下链条上更高层面可用的功能，并看看是否可以纠正您遇到的任何问题。

以奥卡姆剃刀的观点来思考：最简单的解决方案通常是最好的，因为它的字面意思是“不应该使用比必要更多的东西”。

# 界面膨胀

我遇到过多次人们认为他们在进行出色的架构，但结果证明他们的努力是适得其反。界面膨胀是这种情况的常见后果。

有一次，当我与一个 Scrum Master 讨论在 PHP 中进行多态时接口的重要性时，他告诉我他曾在一个环境中工作过，那里有一个工程师花了几个月时间开发接口，并认为自己在进行出色的架构工作。不幸的是，事实证明他并没有做出出色的基础架构工作，他实际上是在实现界面膨胀。

界面膨胀，正如其名，是指界面过度膨胀。界面可能膨胀到几乎不可能以其他方式实现类。

接口应该节俭使用；如果类只会被实现一次（实际上，没有人永远不需要修改这样的代码），那么你真的需要一个接口吗？如果需要，你可能要考虑在这种情况下避免使用接口。

接口不应该被用作测试单元功能的手段。在这种情况下，你真的应该使用单元测试，例如通过 PHPUnit。即使如此，单元测试应该测试一个单元的功能，而不是用作确保没有人编辑你的代码的工具。

因此，让我给你举一个接口膨胀的实现。让我们看看 Pheanstalk 开源库中的`Pheanstalk`接口类（注意我已经删除了注释以使其更易读）：

```php
<?php 

namespace Pheanstalk; 

interface PheanstalkInterface 
{ 
    const DEFAULT_PORT = 11300; 
    const DEFAULT_DELAY = 0; 
    const DEFAULT_PRIORITY = 1024; 
    const DEFAULT_TTR = 60; 
    const DEFAULT_TUBE = 'default'; 

    public function setConnection(Connection $connection); 

    public function getConnection(); 

    public function bury($job, $priority = self::DEFAULT_PRIORITY); 

    public function delete($job); 

    public function ignore($tube); 

    public function kick($max); 

    public function kickJob($job); 

    public function listTubes(); 

    public function listTubesWatched($askServer = false); 

    public function listTubeUsed($askServer = false); 

    public function pauseTube($tube, $delay); 

    public function resumeTube($tube); 

    public function peek($jobId); 

    public function peekReady($tube = null); 

    public function peekDelayed($tube = null); 

    public function peekBuried($tube = null); 

    public function put($data, $priority = self::DEFAULT_PRIORITY, $delay = self::DEFAULT_DELAY, $ttr = self::DEFAULT_TTR); 

    public function putInTube($tube, $data, $priority = self::DEFAULT_PRIORITY, $delay = self::DEFAULT_DELAY, $ttr = self::DEFAULT_TTR); 

    public function release($job, $priority = self::DEFAULT_PRIORITY, $delay = self::DEFAULT_DELAY); 

    public function reserve($timeout = null); 

    public function reserveFromTube($tube, $timeout = null); 

    public function statsJob($job); 

    public function statsTube($tube); 

    public function stats(); 

    public function touch($job); 

    public function useTube($tube); 

    public function watch($tube); 

    public function watchOnly($tube); 
} 

```

呸！注意即使常量也被放在了实现中，这可能是你真正想要更改的唯一事物。显然，这是一个只能以一种方式实现的类的接口，使接口变得无用。

在编写面向对象的代码时，接口提供了很高程度的结构；一旦实现，它们作为保证，确保了实现它的类中的方法已经被实现。

然而，像大多数好事一样，它可能是一把双刃剑。有人曾经用一个极其天真的论点反对架构设计；他引用了他以前的一位同事，后者花了数月时间仅仅编写了非常详细的接口，并认为这是很好的架构。事实上，他是在制造接口膨胀。

接口不应该是强制实现的一种方式；事实上，有一些接口的例子导致某人面临的问题是永远无法以其他方式将接口实现到类中。

接口不应包含数千个引用类内部操作的方法。它们应该是轻量级的，并被视为一种保证，当查询某些内容时，它一定存在。

有一个反模式被称为**瑞士军刀**（或**厨房水槽**），围绕着人们试图设计接口以适应类的每种可能的用例的想法。这可能会导致调试、文档编制和维护困难。

# 本末倒置

像大多数开发人员一样，我偶尔会对一些项目管理策略感到困惑；*本末倒置*也不例外。

本末倒置是一个反模式，即被设计出来却永远不需要被构建的功能，从而浪费时间。这种情况让我感到恼火的是在技术会议上讨论长期技术计划时，项目经理会讨论一个功能，然后立即要求提供这个功能如何实现的技术细节。

首先，重要的是要注意，优秀的开发人员应该离开并有研究时间来提出解决方案。开发人员只有通过研究他们打算的解决方案，与开发团队一起讨论，在线查找其他人面临类似问题的情况，然后提出一个统一的、良好架构的解决方案，才能变得更加强大。

我曾在伦敦的首届领导开发者大会上发言，有一句话让我印象深刻。这句话源自非洲谚语，但在软件工程环境中尤为真实：

> *“如果你想走快，就一个人走。如果你想走远，就一起走。”*

我曾经与各种公司的董事总经理和首席执行官交谈过，他们喜欢在董事会上拥有各种不同性格的人。首席财务官（CFO）可能是一个无情的完美主义者，只有在所有数字都完美无缺时才感到满意，而首席运营官可能在按时交付方面是一个强硬的实用主义者。在开发团队中也可能是如此；拥有广泛的专业知识和性格的输入，提出经过激烈讨论的想法，以得出一个全面的解决方案，对于需要做出大决策的情况是有益的，一个单独的开发人员无法期望做出决策。是的，你可能需要一个过滤器，甚至说只有开发团队的一小部分可能与某个特定决策相关，但总的来说，你的开发人员需要资源和时间来做出架构决策。此外，做出这种架构决策的最佳地点是在最相关的时候，当有必要做出这些决策时。

地平派是指相信地球是一个平坦圆盘的人。当他们面对重力概念时，他们宣称重力不存在，并声称这个平坦的地球实际上只是以每秒 9.8 米的速度在太空中上升。当面对更多的科学理论时，他们反而创造出自己的不合逻辑的物理宇宙观。当然，这样的理论是荒谬的。我在这里要说的是，你应该基于扎实的计算机科学（例如已发表的 RFC）来做出决策，而不是根据临时基础上的自己的计算机科学。

# 开发和运营的分离

我曾经遇到过开发环境，开发人员明令禁止进行任何操作，传统的开发结构在 21 世纪的网络环境中受到了严重打击。有着固定的工作角色；你要么是开发人员，要么是负责托管。尽管两个部门有着明显的共同命运，但它们有着独立的预算。

这种设置的结果是开发人员和运营技术人员从未共享知识。通过结合开发和运营（如果你愿意，就叫做 DevOps），不仅可以通过共享知识库有效提高工作质量，而且通过赋予开发人员更多的权力，还可以提高效率。

在我提到的例子中，当公司服务器上托管的网站被黑客入侵或破坏时，所有运营部门所做的就是从备份中恢复。将开发工作整合到这一过程中不仅导致了漏洞的修补，还采取了有效措施来纠正这些问题（无论是暴力插件还是网络应用防火墙）。

# 过分分离的开发责任

过分明显地分割开发责任可能对团队有害。

有些分离是必要的。例如，与物联网（IoT）平台合作的团队不能指望他们既能保持强大的电子工程知识，又能保持强大的前端网页开发知识。也就是说，开发人员应该期望学习他们遇到的其他技能，并且可以通过鼓励知识共享来帮助他们。拥有多学科团队成员并不是商业劣势，事实上这是一个优势。

# 错误抑制运算符

PHP 中的错误抑制运算符确实是一个非常危险的工具。只需在语句前面加上一个`@`符号，就可以抑制由此产生的任何错误，包括导致脚本停止执行的致命错误。

不幸的是，目前在 PHP 中还不能废弃这一点；与 PHP 内部组的成员交谈后得知，首先需要做大量的先决工作，因为一些 PHP 函数没有伴随的错误函数来产生在执行 PHP 脚本时的错误。因此，唯一的方法是捕获在特定函数操作期间抛出的非致命错误，这样就不会停止脚本的执行。

不幸的是，PHP 核心本身包含相当多的技术债务。不幸的是，一个优秀的 PHP 开发人员应该擅长发现 PHP 核心中的技术债务。事实上，Facebook 试图通过自己重写 PHP 核心并称其为**Hak**来规避这个问题；我将让你决定是否应该考虑采用它。

在 Go 中我非常喜欢的一个特性（这是 Google 编写的一种系统语言）是你可以进行多返回类型（例如，你可以从一个函数返回两个值）。这样做的额外好处是，你可以简单地在一个函数调用中返回任何错误，而不是需要一个返回错误消息的伴随函数。

我在 Go 中也喜欢的一点是所有警告都被视为错误。你赋值给一个变量，然后不使用它？程序将无法运行（除非你将变量赋值给下划线`_`，这是一个空赋值运算符，意味着变量不会被存储在任何地方）。将警告视为错误的结果是，当开发人员遇到错误时，他们知道这是严重的。

所以是的，PHP 可以从诸如 Go 之类的语言中学到很多东西，但基本上，很明显 PHP 核心已经需要做很多工作，而且除此之外，PHP 社区可能需要进行文化转变，更加开放，少一些政治色彩。**PHP RFC: 采用行为准则**提出 PHP 应该采用*行为准则*。不用说，如果以某种形式采用，PHP 社区应该会受益。

回到手头的问题，应该避免使用错误抑制运算符，除非绝对必要，以便使开发人员更容易进行调试。

# 盲目信任

我大约 11 岁的时候，在一节物理课上，我们只有有限数量的量角器，我们慢慢地把它们传递下去，以便画出一个角度。作为一个年轻时的狡猾捷径者，我决定不等待，而是直接复制了别人画的图。当时我的物理老师惊呆了，大声喊道：“不行！物理是关于精确的！”

他说得有道理，这在编程世界中也是非常真实的。

为了避免*盲目信任*，你应该注意以下错误：

+   未检查返回类型

+   未检查你的数据模型

+   假设你的数据库中的数据是正确的，或者是你期望的格式

让我们把这个问题提升到更极端的程度；看看这段代码：

```php
<?php 

$isAdmin = false; 
extract($_GET); 

if ($isAdmin === true) { 
  echo "Hey ".$name."; here, have some secret information!"; 
} 

```

在上面的代码中，有两个关键错误。第一个错误是我们直接提取`GET`变量；我们将远程定义的变量导入到当前符号表中，有效地允许任何人覆盖在提取之前定义的任何变量。

此外，显然存在 XSS 漏洞，因为我们在返回`GET`变量时没有对其进行消毒处理。

所以我们可以这样改进：

```php
<?php 

$isAdmin = false; 

if ($isAdmin === true) { 
    echo "Hey ".htmlspecialchars($_GET['name'])."; here, have some secret information!"; 
} 

```

# 顺序耦合

**顺序耦合**是指创建一个类，该类具有必须按特定顺序调用的方法。以`init`、`begin`或`start`开头的方法名称可能表明这种行为；根据上下文，这可能表明一种反模式。有时，工程师使用汽车来解释抽象概念，在这里我也会这样做。

例如，看下面的类：

```php
<?php 

class BadCar 
{ 
  private $started = false; 
  private $speed = 0; 

  private $topSpeed = 125; 

  /** 
   * Starts car. 
   * @return bool 
   */ 
  public function startCar(): bool 
  { 
    $this->started = true; 

    return $this->started; 
  } 

  /** 
   * Changes speed, increments by 1 if $accelerate is true, else decrease by 1\. 
   * @param $accelerate 
   * @return bool 
   * @throws Exception 
   */ 
  public function changeSpeed(bool $accelerate): bool 
  { 
    if ($this->started !== true) { 
      throw new Exception('Car not started.'); 
    } 

    if ($accelerate == true) { 
      if ($this->speed > $this->topSpeed) { 
        return false; 
      } else { 
        $this->speed++; 
        return true; 
      } 
    } else { 
      if ($this->speed <= 0) { 
        return false; 
      } else { 
        $this->speed--; 
        return true; 
      } 
    } 
  } 

  /** 
   * Stops car. 
   * @return bool 
   * @throws Exception 
   */ 
  public function stopCar(): bool 
  { 
    if ($this->started !== true) { 
      throw new Exception('Car not started.'); 
    } 

    $this->started = false; 

    return true; 
  } 
} 

```

正如您可能注意到的，我们必须在使用其他函数之前运行`startCar`函数，否则会抛出异常。实际上，如果您尝试加速未启动的汽车，它不应该做任何事情，但是为了论证的目的，我已经更改了它，以便汽车首先会启动。在停止汽车的下一个示例中，我已更改了类，以便如果您尝试在汽车未运行时停止汽车，该方法将返回`false`：

```php
<?php 
class GoodCar 
{ 
  private $started = false; 
  private $speed = 0; 

  private $topSpeed = 125; 

  /** 
   * Starts car. 
   * @return bool 
   */ 
  public function startCar(): bool 
  { 
    $this->started = true; 

    return $this->started; 
  } 

  /** 
   * Changes speed, increments by 1 if $accelerate is true, else decrease by 1\. 
   * @param bool $accelerate 
   * @return bool 
   */ 
  public function changeSpeed(bool $accelerate): bool 
  { 
    if ($this->started !== true) { 
      $this->startCar(); 
    } 

    if ($accelerate == true) { 
      if ($this->speed > $this->topSpeed) { 
        return false; 
      } else { 
        $this->speed++; 
        return true; 
      } 
    } else { 
      if ($this->speed <= 0) { 
        return false; 
      } else { 
        $this->speed--; 
        return true; 
      } 
    } 
  } 

  /** 
   * Stops car. 
   * @return bool 
   */ 
  public function stopCar(): bool 
  { 
    if ($this->started !== true) { 
      return false; 
    } 

    $this->started = false; 

    return true; 
  } 
} 

```

# 大改写

开发人员的一个诱惑是重写整个代码库。您需要权衡利弊，是的，阅读现有代码通常比编写新代码更困难；但请记住，重写需要时间，对您的业务可能造成巨大成本。

请记住，任何项目的技术债务总和永远不会超过从头开始启动项目。

Maiz Lulkin 在一篇博客文章中写道：

> “大改写的问题在于它们是对文化问题的技术解决方案。”

大改写非常低效，特别是当您无法保证开发人员现在会更好时。在截止日期内设计新系统并迁移数据可能是一项艰巨的任务。

此外，部署大改写可能会带来巨大问题；将这样的更改部署到应用程序的整个代码库可能是致命的。尝试定期在频繁的间隔内部署代码。尝试一次更改一件事。

您现有的软件就是您现有的规范。通过进行重写，您正在基于遗留代码构建代码。

幸运的是，还有一种替代方法；在周期中快速改进您当前的代码库。您可以采取三个主要步骤来改进您的代码库：

+   测试（单元测试、行为测试等）

+   服务拆分

+   完美的分阶段迁移

本书有一章专门讲述重构以及我们如何改变遗留代码的设计。

## 自动化测试

您需要测试；是的，编写自动化测试可能会很慢，但对于确保重写或重构时不会出现问题至关重要。

测试和开发发生在尽可能接近生产环境的环境中也至关重要。Web 服务器软件或数据库权限的微小变化可能会产生灾难性后果。

使用自动化部署系统，如 Vagrant 与 Puppet 或 Docker，可能是一个很好的解决方案。

在使用 PHPUnit 和 Composer 进行单元测试时，您可以将其包含在`composer.json`文件中以引入：

```php
{ 
  "autoload": { 
    "psr-4": { 
      "IcyApril\\Example": "src/" 
    } 
  }, 
  "require": { 
    "illuminate/database": "*", 
 **"phpunit/phpunit": "*",** 
    "robmorgan/phinx": "*" 
  } 
} 

```

除此之外，`phpunit.xml`文件也可能很有用，这样 PHPUnit 就知道测试在哪里，还知道 Composer 自动加载器在哪里（这样它就可以继续引入类）：

```php
<?xml version="1.0" encoding="UTF-8"?> 
<phpunit colors="true" bootstrap="./vendor/autoload.php"> 
  <testsuites> 
    <testsuite name="Application Test Suite"> 
      <directory>./tests/</directory> 
    </testsuite> 
  </testsuites> 
</phpunit> 

```

然后，您可以像在 PHPUnit 中一样编写测试：

```php
<?php 
class App extends PHPUnit_Framework_TestCase 
{ 
  public function testApp() 
  { 
$this->assertTrue(true); 
  } 
} 

```

当然，您还可以在需要时将 PHP 类引入自动加载器，从而获得额外的好处。

并非所有测试都需要是单元测试。编写外部测试脚本来测试 API 也可能很有益。一个名为**Selenium**（[`www.seleniumhq.org`](http://www.seleniumhq.org)）的工具甚至可以帮助您进行浏览器自动化。

## 服务拆分

将您的单体应用程序拆分为小的独立松耦合服务是减少技术债务的好方法。

具有技术债务的大型单体应用程序可能很难处理，因为技术债务根植于应用程序的核心。在这样不稳定的基础上构建可能很难以后拆分。然而，有一个解决方案；通过构建独立服务的新功能，您可以有效地在稳定的基础上构建新的核心，与旧的脆弱基础设施分道扬镳。然后，您可以使用 RESTful 结构与旧的单体和这样的新服务进行互联。

这种结构允许您在迁移到新的微服务架构的同时继续开发新功能。

马丁·福勒提出了一种称为**分支抽象**的系统，它允许您逐渐对系统进行大规模的更改，同时可以继续发布。

第一步是捕捉客户端代码的一个部分与其供应商之间的交互；然后我们可以更改代码的这一部分，使其通过一个抽象层进行相互通信。

然后我们对与供应商的交互进行同样的操作。在这样做的同时，我们有机会提高单元测试覆盖率。一旦一个供应商完全不再使用，我们可以将客户端迁移到使用该供应商，然后删除旧的供应商。

## 完美的分阶段迁移

将您的单体架构拆分为小型独立松耦合的服务是减少技术债务的好方法，但在这个过程中，您显然增加了对架构层面的额外负担。

在迁移数据或托管环境时，您可能会在这个过程中遇到困难。当部署过程不重复并且对每个部署都是独特的时，这一点尤为真实（例如在不使用持续集成的环境中）。

使用诸如 Docker 之类的容器技术可以让您更好地进行快速应用程序部署，使您能够更快地部署，同时增加可移植性并简化维护。一些人可能会发现其他技术，如 Vagrant，对他们更有利；不过，所有这些技术都有一个共同的因素：基础设施即代码。

**基础设施即代码**是通过代码管理和配置计算基础设施的过程，而不是通过交互式配置工具；然而，我们在这里追求的比这更基本。我们想要能够在事实之前分阶段和测试任何类型的迁移，并在执行迁移时重新运行确切的过程。

通过脚本化迁移，您可以像代码一样事先测试它们。您可以确保在实际服务器上完成时，与在暂存服务器上相比，减少了任何错误的机会。

除此之外，迁移后可以用于在部署中出现问题时逆向工程该过程，或者可以看到决策的理由。它实质上充当了软件部署过程的工件。

在可能的情况下，应尽可能多地提供资源；这包括部署代码的人员、组装项目的开发人员，以及在极端情况下，一个沟通人员来及时更新客户。这些资源可以快速调试问题，但至关重要的是，部署代码的个人必须主导并协调这些资源的使用，以防止分散注意力。

按照正式的预先计划的程序工作，同时也留有纠正任何问题的空间，通常可以帮助使部署尽可能无痛。

# 测试驱动开发

这是对**测试驱动开发**（**TDD**）的一种半开玩笑式引用。TDD 是一种软件开发策略，主要围绕使用开发测试来驱动实现以满足需求。

然而，测试驱动开发是指需求是一种捷径，软件团队开始通过错误报告来指定需求。测试驱动开发也可以称为**错误驱动开发**，因为它实质上导致错误报告被用来指定开发人员应该实现的操作和功能。

例如，开发人员构建了一个工具，用于将数据库中的数据导出到电子表格中。它运行得很完美，但测试人员仍然回来提出了一个问题，说产品中存在一个错误；他们说它没有包含导出到 PDF 的功能。如果这不在需求中，就不应该被提出作为错误。是的，您应该有需求。

QA 团队和测试人员的存在是为了验证软件是否符合要求。他们的存在并不是为了规定要求本身。

# 臃肿的优化

通常，开发人员可能会在试图过度优化他们的代码或设计工件时自相矛盾，甚至在他们的代码甚至没有执行基本功能之前，甚至在任何代码被创建之前。这可能会迅速在生产中出现问题。

在这一部分，我想专门讨论与这个主题相关的三种反模式：

+   分析瘫痪

+   无关紧要的争论

+   过早的优化

## 分析瘫痪

简而言之，这就是一个策略被过度分析到进展被减缓甚至在极端情况下完全停滞的地方。这样的解决方案不仅可能迅速过时，而且可能是在教育不足的情况下做出的，例如，在一个过度分析的老板试图在允许他们的开发人员实际进行一些研究之前就过于深入细节的会议上。

过度分析问题并试图提前寻求完美解决方案是行不通的；程序员应该寻求完善他们的解决方案，而不是提前想出完善的解决方案。

## 无关紧要的争论

基本上，这就是分析瘫痪可能发生的地方，基于一些非常琐碎的决定，例如登录页面的颜色。唯一需要修复的是不要浪费时间在琐碎的决定上。尽量避免委员会决策，因为大多数人，无论他们认为自己的设计技能有多好，在设计上都是相当无能的。

## 过早的优化

到目前为止，在这一部分，我主要是在批评项目经理；没有时间去批评开发人员。通常，开发人员会试图过早地优化他们的代码，而没有经过教育的数据驱动的结论来驱动何时何地进行优化。

编写清晰易读的代码是你的首要任务；然后你可以使用一些很棒的性能分析工具来确定你的瓶颈在哪里。XDebug 和 New Relic 只是一些擅长这方面的工具。

也就是说，有些情况下必须进行优化，特别是在一些长时间的计算任务上，从 O(N2)时间减少到 O(N)可能是至关重要的。也就是说，大多数简单的 PHP Web 应用程序不需要考虑这一点。

# 未受教育的经理综合症

你的经理有没有亲自构建过 Web 应用程序？我发现这是一个经理必须具备的相当重要的特征。就像初级医生会向曾经经历过初级医生过程的医生汇报一样，或者老师会向曾经是老师的校长汇报一样，软件开发人员应该向曾经经历过这个过程的人汇报。

显然，在小团队中（例如，一个小型设计公司在业余时间进行 Web 开发），可能并不严格需要工程经理。这在经理明白必要时需要把决策推迟给程序员的情况下运作良好。然而，一旦事情扩大，就需要有结构。

诸如谁来雇佣，谁来解雇，如何解决技术债务，哪些元素需要最关注等决定，需要由开发人员来做；此外，有时候这些决定不应该民主地做，因为这样做会导致委员会决策。在这种情况下，需要一个工程经理。

在大规模团队中，应该总有一个开发人员花费超过 90%的时间不是在编写代码。

我将进一步深入；一个 Web 工程经理不应该只有技术背景，他们应该有 Web 背景。开发 Java 应用程序开发人员可能与构建 PHP Web 应用程序完全不同，因此这样的工程经理应该通过一些 Web 经验来理解这样的学科（尽管不一定要在一种特定的语言中）。

# 错误的基础

SensioLabs Insight 工具被用来评估各种项目中的技术债务，并且他们评估并发布了响应。SensioLabs 在他们的博客上回应说，结果没有考虑项目的年龄或项目的大小，但无论如何，它确实显示了在使用某些框架作为基础时你所面临的技术债务：

![错误的基础](img/image_02_001.jpg)

不要误会：WordPress 是一个很棒的 CMS；是的，它在核心部分有一些怪癖，并且来自 OOP 之前的时代，但它是一个很棒的博客平台。通常情况下，你不应该去摆弄它的核心代码，所以你不需要担心它。当然，你不应该编写自己的博客平台或 CMS，但与此同时，WordPress 也不适合构建营销资产系统或保险报价生成器（是的，这两个都是我最初被要求在 WordPress 中完成的真实项目）。

简而言之：为你的任务使用最佳基础。

# 长方法

在某些情况下，PHP 中的方法可能过于复杂；例如，在下面的类中，我故意省略了一些有意义的注释，并且构造函数过长：

```php
<?php 
class TaxiMeter 
{ 
  const MIN_RATE = 2.50; 
  const secondsInDay = 60 * 60 * 24; 
  const MILE_RATE = 0.2; 

  private $timeOfDay; 
  private $baseRate; 
  private $miles; 
  private $dob; 

  /** 
   * TaxiMeter constructor. 
   * @param int $timeOfDay 
   * @param float $baseRate 
   * @param string $driverDateOfBirth 
   * @throws Exception 
   */ 
  public function __construct(int $timeOfDay, float $baseRate, string $driverDateOfBirth) 
  { 
    if ($timeOfDay > self::SECONDS_IN_DAY) { 
      throw new Exception('There can only be ' . self::SECONDS_IN_DAY . ' seconds in a day.'); 
    } else if ($timeOfDay < 0) { 
      throw new Exception('Value cannot be negative.'); 
    } else { 
      $this->timeOfDay = $timeOfDay; 
    } 

    if ($baseRate < self::MIN_RATE) { 
      throw new Exception('Base rate below minimum.'); 
    } else { 
      $this->baseRate = $baseRate; 
    } 

    $dateArr = explode('/', $driverDateOfBirth); 
    if (count($dateArr) == 3) { 
      if ((checkdate($dateArr[0], $dateArr[1], $dateArr[2])) !== true) { 
        throw new Exception('Invalid date, please use mm/dd/yyyy.'); 
      } 
    } else { 
      throw new Exception('Invalid date formatting, please use simple mm/dd/yyyy.'); 
    } 
    $this->dob = $driverDateOfBirth; 

    $this->miles = 0; 

  } 

  /** 
   * @param int $miles 
   * @return bool 
   */ 
  public function addMilage(int $miles): bool 
  { 
    $this->miles += $miles; 
    return true; 
  } 

  /** 
   * @return float 
   * @throws Exception 
   */ 
  public function getRate(): float 
  { 
    $dynamicRate = $this->miles * self::MILE_RATE; 

    $totalRate = $dynamicRate + $this->baseRate; 

    if (is_numeric($totalRate)) { 
      return $totalRate; 
    } else { 
      throw new Exception('Invalid rate output.'); 
    } 
  } 
} 

```

现在，让我们做两个小改变；让我们将一些方法提取到它们自己的函数中，并添加一些 DocBlock 注释。这仍然并不完美，但请注意所做的区别：

```php
<?php 

class TaxiMeter 
{ 
  const MIN_RATE = 2.50; 
  const SECONDS_IN_DAY = 60 * 60 * 24; 
  const MILE_RATE = 0.2; 

  private $timeOfDay; 
  private $baseRate; 
  private $miles; 

  /** 
   * TaxiMeter constructor. 
   * @param int $timeOfDay 
   * @param float $baseRate 
   * @param string $driverDateOfBirth 
   * @throws Exception 
   */ 
  public function __construct(int $timeOfDay, float $baseRate, string $driverDateOfBirth) 
  { 
    $this->setTimeOfDay($timeOfDay); 

    $this->setBaseRate($baseRate); 

    $this->validateDriverDateOfBirth($driverDateOfBirth); 

    $this->miles = 0; 

  } 

  /** 
   * Set timeOfDay class variable. 
   * Only providing it doesn't exceed the maximum seconds in a day (const secondsInDay) and is greater than 0\. 
   * @param $timeOfDay 
   * @return bool 
   * @throws Exception 
   */ 
  private function setTimeOfDay($timeOfDay): bool 
  { 
    if ($timeOfDay > self::SECONDS_IN_DAY) { 
      throw new Exception('There can only be ' . self::SECONDS_IN_DAY . ' seconds in a day.'); 
    } else if ($timeOfDay < 0) { 
      throw new Exception('Value cannot be negative.'); 
    } else { 
      $this->timeOfDay = $timeOfDay; 
      return true; 
    } 
  } 

  /** 
   * Sets the base rate variable providing it's over the MIN_RATE class constant. 
   * @param $baseRate 
   * @return bool 
   * @throws Exception 
   */ 
  private function setBaseRate($baseRate): bool 
  { 
    if ($baseRate < self::MIN_RATE) { 
      throw new Exception('Base rate below minimum.'); 
    } else { 
      $this->baseRate = $baseRate; 
      return true; 
    } 
  } 

  /** 
   * Validates 
   * @param $driverDateOfBirth 
   * @return bool 
   * @throws Exception 
   */ 
  private function validateDriverDateOfBirth($driverDateOfBirth): bool 
  { 
    $dateArr = explode('/', $driverDateOfBirth); 
    if (count($dateArr) == 3) { 
      if ((checkdate($dateArr[0], $dateArr[1], $dateArr[2])) !== true) { 
        throw new Exception('Invalid date, please use mm/dd/yyyy.'); 
      } 
    } else { 
      throw new Exception('Invalid date formatting, please use simple mm/dd/yyyy.'); 
    } 

    return true; 
  } 

  /** 
   * Adds given milage to the milage class variable. 
   * @param int $miles 
   * @return bool 
   */ 
  public function addMilage(int $miles): bool 
  { 
    $this->miles += $miles; 
    return true; 
  } 

  /** 
   * Calculates rate of trip. 
   * Times class constant mileRate against the current milage in miles class variables and adds the base rate. 
   * @return float 
   * @throws Exception 
   */ 
  public function getRate(): float 
  { 
    $dynamicRate = $this->miles * self::MILE_RATE; 

    $totalRate = $dynamicRate + $this->baseRate; 

    if (is_numeric($totalRate)) { 
      return $totalRate; 
    } else { 
      throw new Exception('Invalid rate output.'); 
    } 
  } 
} 

```

长方法是代码异味的指标；它们指的是代码中可能存在更深层问题的症状。其他例子包括重复的代码和人为的复杂性（在更简单的方法可以满足的情况下使用高级设计模式）。

# 魔术数字

请注意，在前面的例子中，我总是将我的常量数值变量放在类常量中，而不是直接放在代码中：

```php
  const minRate = 2.50; 
  const secondsInDay = 60 * 60 * 24; 
  const mileRate = 0.2; 

```

我这样做的原因是为了避免一种被称为**魔术数字**或**未命名的数值常量**的反模式。使用类常量可以使代码更易于阅读、理解和维护；当然，在 PSR 标准下，它们应该以大写字母分隔下划线的形式声明。

# 摘要

在本章中，我们介绍了一些基本的反模式，供你避免；有些是架构上的，有些是与 PHP 相关的，还有一些是在管理层上的。

基本上，反模式会导致技术债务。所谓技术债务，指的是代码难以扩展，以至于以后更改变得更加困难。

以下是我希望你做的事情清单：

+   在开始编码之前制定计划

+   做注释，并在你的代码目的不明显时添加注释

+   确保你的代码有结构。

+   尽量避免将太多的代码放在一个方法中

+   使用 DocBlock

+   使用常识方法来处理 PHP

在本章中，我们学习了一些常见的设计问题，这些问题可能导致严重的问题；这些原则可以帮助你在以后避免重大问题。编写可扩展的代码是设计的一个重要因素。在其核心，这需要理解约束。使用适当的进程间通信策略可以帮助你的服务扩展，编写松耦合的代码可以增加代码重用和调试。最后，在部署这个令人敬畏的代码时，自动化测试和完美的分阶段迁移可以确保一切顺利进行。

在接下来的章节中，我们将继续介绍一些设计模式（大概是你一直在等待的）。

如果你有兴趣了解如何改进现有代码库的设计，你可能会发现本书中关于重构的专门章节特别有趣；但是在阅读其他设计模式之前，了解我们试图朝着重构的模式是值得的。
