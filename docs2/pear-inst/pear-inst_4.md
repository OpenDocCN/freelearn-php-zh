# 第四章. 使用 PEAR 安装程序进行巧妙的网站协调

在最后两章中，我们学习了如何使用 PEAR 安装程序的功能来管理公共分发用的库和应用程序。在本章中，我们将学习如何使用 PEAR 安装程序使管理复杂且快速发展的网站内容变得容易。实际上，PEAR 安装程序可以用来提供额外的保险，确保网站按预期运行，甚至使诊断问题更容易。

在本章中，我们将首先从高层次了解我们问题的细节，并了解 PEAR 安装程序如何帮助我们解决问题。然后我们将查看解决方案的第一步，设置源代码控制系统，最后我们将以一个实际网站为例，完成多段网站复杂性的管理。

# 问题概述

最重要的任务之一是保持复杂且动态网站的结构在信息表示演变时保持一致和最新。在许多情况下，重新组织可能会导致不再使用的文件充斥着目录布局。更糟糕的是，在删除未使用的文件时，可能会不小心删除一个重要的文件而未意识到这一点。此外，协调多开发者模块化网站显然是一个挑战：如何防止更新和添加时的冲突？

与如 CVS（并发版本系统）之类的版本控制系统结合使用时，PEAR 安装程序提供了一个独特且经过实战检验的解决方案，以有效地管理所有这些问题。版本控制系统提供了一种冗余和灵活性的组合，这是简单的文件系统无法比拟的。能够在不担心破坏主代码库的情况下检查出一个个人**沙盒**进行开发的能力是任何严肃代码开发的一个基本部分。

传统上，分支（CVS 和 Subversion）和标签（仅限 CVS）用于记录“发布点”，即程序已准备好使用。例如，可以通过以下命令直接从 `cvs.php.net` 获取 PEAR 版本 1.4.5：

```php
$ cvs -d :pserver:anoncvs@cvs.php.net:/respository login
Password:<enter your email address>
$ cvs -d :pserver:anoncvs@cvs.php.net:/respository co r RELEASE_1_4_5 pear-core 

```

此序列检查 PEAR 安装程序的源代码，该源代码位于 `/repository/pear-core` 中的 `cvs.php.net`，然后检索在 PEAR 版本 1.4.5 发布时设置的标签 `RELEASE_1_4_5`，这是通过方便的 `cvstag` 命令（本章后面将详细介绍）完成的：

```php
$ pear cvstag package2.xml package-PEAR.xml tests

```

上述示例是最复杂的可能命令。在大多数情况下，开发者可以使用以下命令简单地标记一个包：

```php
$ pear cvstag package.xml 

```

当然，PEAR 安装程序和版本控制系统只与它们背后的协调计划一样有效，因此制定一个结合这两个工具开发网站的良好策略是等式中的一个基本部分，并在本章的最后一段中进行了介绍。

使用**路线图**或开发时间表，描述新功能将何时添加以及旧功能将何时移除，这是一个很好的第一步。定义开发者应该如何协调和同步他们的努力，这也是另一个方面。使用设计工具和策略，如 UML 和极限编程（测试驱动开发及其相关技术）也可能有用，但最终，网站架构师设计目标中的思维清晰度通常更为重要，并且将导致最佳解决方案，无论选择哪种工具来实现。

### 小贴士

我们将解决的主要问题是如何协调一个复杂的网站，具体来说，如何从开发机器安全且系统地更新实时网站。

# 理解问题

要理解问题的解决方案，从高层次理解问题是有意义的。在我们的案例中，理解围绕协调主要网站开发的主要问题非常重要。一个好的网站将吸引用户尽可能频繁地访问，并发展成一个社区。这只有在有令人兴奋的内容和令人愉悦的视觉和逻辑布局的情况下才会发生。通常，内容可以更新而不需要更改任何代码（想想博客或内容管理系统），但更改网站的视觉布局和逻辑结构需要更广泛的网站内部更改。

即使你在第一次尝试时就设计出了完美的网站（恭喜！）并且有简单的方法来调整网站的内容甚至逻辑结构，这也可能导致从小型网站过渡到每分钟有成千上万的访问量的巨大成功网站的最大挑战：**可扩展性**。通常在这个阶段，可能需要进行全面的重设计来满足意外的需求。成功不可避免地会导致需要聘请新的开发者，他们可能不熟悉网站结构或设计目标。不断重构以改进事物的威胁也可能给甚至最善意的网络团队带来意外的混乱。

所有这些不确定性都将导致代码损坏的可能性更大，目录结构混乱，以及其他问题。

我们将探讨四个典型的问题领域，并更好地了解 PEAR 安装程序将如何帮助我们：

+   管理代码损坏和回滚到先前版本

+   管理缺失或多余的文件

+   与一组开发者协调开发

+   备份代码：冗余作为必要的预防措施

## 管理代码损坏和回滚到先前版本

有时，尽管经过精心设计和更加仔细的测试，但最近的一次更新中仍然可能破坏了网站的关键部分。更糟糕的是，安全漏洞可能导致恶意黑客破坏精心构建的网站结构。

在过去，这可能意味着希望从备份中恢复能够解决问题。在某些情况下，一个漏洞可能长时间未被注意到，需要从早期的备份中恢复。这可能导致在确定要传输和删除的正确文件时遇到极大的困难。

PEAR 安装程序以极其高效的方式管理这些问题。无需花费狂热的时间手动检查每个目录，只需两个命令就足以完全删除并恢复最新的网站结构：

```php
$ rm -rf /path/to/htdocs
$ pear upgrade --force WebSite-1.2.3.tgz 

```

与“*一切是否都正常？哦，我需要恢复那个文件并删除这个文件*”的慌乱相比，这种复杂性之间的差异是惊人的。

### 小贴士

**PEAR 安装程序或 rsync？**

经验丰富的网络开发者也应该了解 `rsync` 命令，它促进了相隔甚远的机器上目录的远程同步。在许多情况下，这是一种确保本地和远程存储库同步的非常有效的方法。然而，如果您在协调几个开发者之间，或者开发网站的某些部分而其他部分稳定时，它可能比便利性造成更多困难。在这种情况下，您将更多地受益于 PEAR 安装程序提供的版本控制和简单回滚的优势。

## 管理缺失或多余的文件

理论上，如果你在上传之前在一个开发机器上完全测试了网站，那么缺失或额外文件的问题永远不会发生，但有时在 `rsync` 传输中发生错误，删除了不应该删除的文件。在最佳情况下，这将导致网站立即损坏，并且可以轻松追踪并修复。然而，在最坏的情况下，损坏可能很微妙，实际上可能直到你的网站的一个最终用户执行了一个罕见但关键的任务，或者黑客发现了一个未使用的文件中的安全漏洞时才变得明显。这可能导致吓跑用户，商业网站的利润损失，甚至如果您的网站被黑客用来犯罪，并且可以证明您的疏忽，还可能导致法律问题。

当使用 PEAR 安装程序管理网站时，这些问题有何不同？PEAR 安装程序有两个特性使其与传统解决方案区别开来：

+   版本控制

+   文件事务

通过版本控制的概念，可以通过检查 `package.xml` 的内容或运行类似 `list-files` 的命令来确定哪些文件存在：

```php
$ pear list-files mychannel/PackageName 

```

使用这个系统可以轻松检测到缺失或多余的文件，无需进行缓慢的递归检查实际目录。此外，能够快速回滚到早期版本，即使暂时如此，然后恢复更新的修复版本，这也非常简单，如下所示示例命令序列：

```php
$ pear upgrade --force mychannel/PackageName-1.0.2
<after fixing things>
$ pear upgrade mychannel/PackageName 

```

PEAR 安装程序利用关系数据库中的一个概念，并实现了基于事务的文件安装和删除。这意味着只有在所有操作都成功完成后，目录结构才会发生变化。此外，尽可能进行原子文件操作。文件的生命周期相当简单，包括以下步骤：

+   文件`path/to/foo.php`被安装为`/path/to/pear/path/to/.tmpfoo.php`

+   如果存在，文件`/path/to/pear/path/to/foo.php`将被重命名为`/path/to/pear/path/to/foo.php.bak`。

+   文件`/path/to/pear/path/to/.tmpfoo.php`被重命名为`/path/to/pear/path/to/foo.php`。

安装完成后，所有创建的`.bak`文件都会被删除。

然而，如果在任何步骤中出现问题，已安装的文件将被删除，`.bak`文件将被重命名为原始文件名。这样，就可以非常安全地管理文件升级。还会执行额外的检查，以确保安装程序能够写入安装目录，并且文件确实按照预期安装。所有这些额外的工作有助于保证安装的成功。

## 与开发者团队协调开发

PEAR 安装程序以两种方式帮助协调一组网络开发者：

+   离散打包

+   文件冲突解决

**离散打包**简单来说，就是每个开发者或子团队的文件集可以占用自己的包，并且可以独立于其他包进行安装/升级。此外，它们都可以通过使用依赖关系从中央包进行管理。

**文件冲突解决**解决了意外覆盖其他团队文件的可能性，并使得安全共享目录空间成为可能。PEAR 安装程序不允许不同包之间冲突任何其他包的文件。这个简单的事实将为你的团队文件命名约定增加一个额外的错误检查层。

即使指定了`--force`选项，也会使用文件冲突解决机制。只有危险的`--ignore-errors`选项可以覆盖文件冲突检查。

## 代码备份：作为必要预防措施的冗余

使用 PEAR 安装程序管理网站提供了一种可能不那么明显的利益，即通过复制代码。通常人们只从保留整个站点的副本的角度考虑从网站备份代码的需求。这很重要，但可能还不够。如果只有少数文件损坏，使用 PEAR 安装程序快速恢复网站损坏部分的能力仍然相当简单；只需运行：

```php
$ pear upgrade --force mychannel/MyPackage 

```

此外，将网站代码作为打包存档存储在频道服务器上，在源控制和传统完整备份方法提供的冗余之上提供了额外的冗余级别。

既然我们已经知道了问题区域以及 PEAR 安装程序如何帮助我们处理这些问题，让我们详细探讨解决方案。

# 解决方案，第一部分：至关重要的源代码控制

在使用 PEAR 安装程序之前，设置一个源代码控制系统非常重要。有许多优秀的商业软件程序可以用于执行源代码控制，包括**Perforce**和**Visual SourceSafe**，但我们将关注经过验证的、免费的开放源代码版本控制系统：*CVS*和**Subversion**。

**CVS**（**并发版本控制系统**）是最早的源代码控制产品之一，基于更早的**RCS**（**版本控制系统**）源代码控制程序。CVS 通过使用客户端-服务器模型来实现其源代码控制。服务器包含最终代码，组织成目录和文件。然而，在服务器上，每个文件实际上都包含该文件的完整版本历史。在客户端，用户检出本地**沙盒**——服务器代码的副本，然后可以独立于其他开发者进行开发。当代码准备好提交到服务器时，用户向服务器发送一个特殊命令。那时，服务器会检查是否有其他用户更改了仓库，如果有，将防止可能发生的冲突。用户提交之间的冲突解决完全支持，以及合并兼容的更改。

虽然 CVS 做得非常好，但也有一些限制促使 Subversion 开发团队开始开发新的模型。与 CVS 一样，Subversion 提供了相同的协作工具。区别在于 Subversion 存储信息的方式。使用伯克利数据库文件来记录更改和版本（或在最新版本中，基于文件的 FSFS 数据库），Subversion 能够跟踪文件组和目录以及单个文件的变化，这是 CVS 难以做到的。此外，Subversion 在客户端沙盒中存储服务器代码的完整副本，这使得在执行检查更改和制作补丁等操作时，能够非常有效地使用带宽。

## 提供冗余和版本历史

源代码控制系统的主要好处是冗余和版本历史的结合。源代码控制系统是围绕人类易犯错误的原则设计的：我们有时会犯错误，能够尽可能容易地从这些错误中恢复过来非常重要。通过将开发者的沙盒与服务器仓库分离提供的冗余，可以快速从开发者的机器上的错误中恢复过来。版本历史的存在，以及 CVS/Subversion 从特定时间点、特定标签或分支检查代码的复杂能力，意味着也可以简单地回滚错误的提交或更改到服务器仓库。

## 安装 CVS 或 Subversion

在大多数情况下，设置版本控制系统是容易的。本文不是安装的最佳资源，但涵盖了基础知识。对于扩展支持，最好是直接咨询 CVS 或 Subversion 的支持资源。

在接下来的几节中，你将学习如何在 CVS 或 Subversion 中初始化仓库，以及如何在仓库内创建新项目，以及如何创建用于开发的本地沙盒。设置版本控制系统有一些先决条件。重要的是你能够访问你打算设置版本控制系统的主机上的 shell。如果你没有文件系统访问权限，修复仓库中的任何问题将会极其困难。

如果你没有远程主机的 shell 访问权限，并且没有资源切换到提供 shell 访问的互联网服务提供商，事情还没有完全失去希望。

在基于 Windows 的系统上，使用免费软件 TortoiseCVS 或 TortoiseSVN 程序设置本地仓库非常容易，而在像 Mac OS X 或 Linux 这样的基于 Unix 的系统上，你可以直接编译并使用 CVS 和 Subversion 工具来初始化仓库。这种方法唯一的缺点是，你失去了一些远程仓库的故障安全优势。

在任何情况下，无论是远程仓库还是本地仓库，你都需要定期备份系统，以避免在硬件故障或其他墨菲定律的令人不快的事实发生时的麻烦。

### 小贴士

**以下是墨菲定律：**

“如果有可能出错，它就会出错。”

总是计划可能出现的问题——硬件故障、数据损坏和安全漏洞只是困扰我们工作的诸多问题中的开始。

### 并发版本控制系统

并发版本控制系统（CVS）托管在[`www.nongnu.org/cvs/`](http://www.nongnu.org/cvs/)，是版本控制的最古老形式。CVS 非常稳定，已经稳定了多年。因此，它是版本控制的老将，经受了时间的考验。CVS 基于一个非常简单的基于文件的版本控制。在仓库中，每个文件都包含内容和包含修订之间差异的元数据。仓库不应被用于直接访问或工作。相反，为了开发，需要检出目录的完整副本。除非你签入或提交代码，否则不会将任何更改保存到主仓库中。

CVS 旨在协调多个开发者的工作，因此具有拒绝提交的能力，如果两个不同开发者的工作可能存在冲突。让我们看看一个例子。

PHP 仓库由`cvs.php.net`托管了几个模块，它们像文件系统一样组织。要检出模块，你必须首先登录到 CVS 服务器。有账户的用户将使用他们的账户名，但 PHP 还提供了匿名只读 CVS 访问。要登录到 CVS 服务器，输入以下命令：

```php
$ cvs -d :pserver:cvsread@cvs.php.net:/repository login 

```

到目前为止，`cvs` 命令将提示输入密码。对于匿名 CVS 访问，请输入您的电子邮件地址。一旦您登录，请检出模块。例如，要查看 PEAR_PackageFileManager 包的源代码，您将输入：

```php
$ cvs -d :pserver:cvsread@cvs.php.net:/repository checkout pear/ PEAR_PackageFileManager 

```

此命令创建 `pear` 目录和子目录 `PEAR_PackageFileManager`，并将项目中的所有文件和目录填充到其中。此外，它还创建了名为 `CVS` 的特殊目录，其中包含有关仓库本地副本状态的详细信息。每个目录必须包含名为 `Entries, Root` 和 `Repository` 的文件。根据仓库的状态，还可能有其他文件。这些文件在极端情况下不应手动修改，但了解控制您的 CVS 检出的是什么非常重要。

关于使用 CVS 的更多信息和技术支持，最好从阅读 `man cvs` 的 *Unix 手册页* 开始，并阅读由 O'Reilly 出版的 CVS 书籍，该书籍在 [`cvsbook.red-bean.com/`](http://cvsbook.red-bean.com/) 上以 GNU 通用公共许可证在线分发。

在 Windows 上，使用像 **TortoiseCVS** 这样的工具可能最简单，它可以从 [`www.tortoisecvs.org`](http://www.tortoisecvs.org) 获取。这个免费工具为 Windows 资源管理器添加了一个扩展，允许通过鼠标右键单击文件或目录来直接操作 CVS 检出和仓库。它非常直观且功能强大。

#### 设置 CVS 仓库

设置 CVS 仓库的第一步是确定您打算将仓库放在哪里。CVS 有几种远程连接到仓库的方法。在大多数情况下，最好要求通过安全外壳（**SSH**）通过 `ext` 方法访问，如下所示：

```php
$ cvs -d :ext:cellog@cvs.phpdoc.org:/opt/cvsroot 

```

进一步解析此命令行，`-d` 选项告诉 CVS 在哪里定位 CVS 的根目录，或 `CVSROOT`。在这种情况下，它告诉 CVS 通过 `pserver` 协议连接到远程 CVSROOT `cvs.phpdoc.org`，使用 `cellog` 用户名。此外，它通知远程 CVS 守护进程 CVSROOT 位于 `/opt/cvsroot`。在 `cvs.php.net` 上，CVSROOT 位于 `/repository`。

如果您在共享主机上，假设您的远程用户名为 `youruser`，最好将 `cvsroot` 放在 `/home/youruser/cvs` 或类似的位置。否则，您可能无法访问初始化 CVS 仓库的目录。显然，写入访问非常重要；否则，从开发沙盒中提交代码是没有办法的。

一旦您决定将 CVS 仓库放在哪里，下一步就是初始化它。这是直截了当的：

```php
$ cvs -d /home/youruser/cvs init 

```

这将创建 `/home/youruser/cvs/CVSROOT` 目录。

下一步是创建您将用于网站的模块。在导入网站之前，首先创建网站的模块。

```php
$ mkdir /home/youruser/website
$ cd /home/youruser/website
$ echo "hi" >> README
$ cvs -d /home/youruser/cvs imoort website tcvs-vendor tcvs-release 

```

这将创建一个名为`website`的模块，可以检出。为了确保成功，检出`website`模块的一个副本：

```php
$ cd
$ cvs -d /home/youruser/cvs checkout website 

```

如果一切顺利，这将导致在`/home/youruser/website`目录和`/home/youruser/website/README`文件中创建目录。为了测试 CVS 是否可以从远程访问，可以通过类似以下方式检出`website`模块的副本：

```php
$ cvs -d :ext:youruser@example.com:/home/youruser/cvs checkout website 

```

下一步是将网站内容添加到 CVS 仓库中。只需将所需目录层次结构的所有网站文件复制到本地`website`模块的检出中即可。下一步是最复杂的。

大多数网站都包含文本文件（如我们的 PHP 脚本）和二进制文件，如图像或声音剪辑。CVS 将二进制文件和文本文件处理方式不同。文本文件会被处理，并且像`$Id$`或`$Revision$`这样的特殊 CVS 内文件标签会被根据文件在仓库中的状态替换为特殊值。像"$Id$"这样的标签必须由开发者手动添加到文件中，CVS 不会自动创建它们。二进制文件被视为一个单一实体，其内容不会被修改。

当向 CVS 模块添加文件时，它们必须首先通过`cvs add`命令添加，然后通过`cvs commit`命令提交。文本文件和目录只需这样添加：

```php
$ cvs add file 

```

使用`-kb`开关添加二进制文件：

```php
$ cvs add -kb file

```

可以使用通配符添加文件，但务必非常小心，确保不要将图像文件作为文本文件添加，或将文本文件作为二进制文件添加！在最坏的情况下，可以在提交之前使用以下命令删除文件：

```php
$ cvs remove file 

```

注意，在删除之前必须先删除已提交到仓库的文件：

```php
$ rm file
$ cvs remove file

```

如果你使用的是 Windows，TortoiseCVS 可以使添加文件变得容易得多，因为它会递归地这样做，并隐藏实现细节。

### Subversion

Subversion 是在几年前开发的，用以解决 CVS 的一些不足。具体来说，Subversion 使用数据库存储仓库信息，因此支持按提交而不是按文件分组更改。Subversion 较新，因此没有像 CVS 那样经过长时间的实战检验，但两者都已经用于生产多年。

### 小贴士

[`svnbook.red-bean.com`](http://svnbook.red-bean.com) 包含了由 O'Reilly 出版的同一本书的几种不同格式。Subversion 书籍包含了设置、配置和管理 Subversion 仓库所需的一切。

Subversion 在几个重要方面与 CVS 不同：

+   远程仓库的完整副本存储在本地，简化了差异比较，并使得离线操作成为可能。

+   标签存储为分支，与 CVS 不同。在 CVS 中，标签是只读的，很难意外修改标签。修改分支相当简单，由于在 Subversion 中标签是分支，这使得实现只读标签更加困难。

+   大文件更容易管理。因为沙盒包含仓库模块当前状态的完整副本，这意味着提交大文本文件只需发送 diff。最终，这可以在服务器上节省带宽和处理器周期，这非常重要。（我曾经通过在 CVS 仓库中提交一个 133MB 数据库转储的微小更改而锁定整个实时服务器，需要重启。这很糟糕。）

+   关键字（`$Id$`、`$Revision$`等）默认不替换；所有 Subversion 中的文件都被视为二进制文件。要为文件设置关键字替换，您需要设置一个属性，例如：

    ```php
    $ svn propset svn:keywords "Id" blah.php 

    ```

更多信息请参阅[`svnbook.red-bean.com/nightly/en/svn.advanced.props.html#svn.advanced.props.special.keywords`](http://svnbook.red-bean.com/nightly/en/svn.advanced.props.html#svn.advanced.props.special.keywords)

#### 设置 Subversion 仓库

与 CVS 一样，设置仓库相对简单。只需运行：

```php
$ svnadmin create /path/to/subversion 

```

这将在当前目录下创建一个 Subversion 仓库作为子目录。要将您的网站代码导入到仓库中，首先设置标准的 Subversion 目录：

```php
$ mkdir ~/tmp
$ cd ~/tmp
$ mkdir website
$ cd website
$ mkdir trunk
$ mkdir tags
$ mkdir branches 

```

接下来，将您当前网站的完整内容复制到`website/trunk`目录中。最后执行：

```php
$ cd ~/tmp
$ svn import . file:///path/to/subversion -m "initial import" 

```

成功导入后，通过检出模块进行测试：

```php
$ svn checkout file:///path/to/subversion/website/trunk website 

```

### 小贴士

**警告：检出 website/trunk，而不是 website。**

如果您检出整个模块，您将获得所有分支和标签。这最终会消耗所有可用的磁盘空间。

远程访问`website`模块需要`svnserve`守护进程正在运行，或者`mod_svn`作为 Apache web 服务器模块正在运行。如果您没有设置，请参考[`svnbook.red-bean.com`](http://svnbook.red-bean.com)中的 Subversion 书籍以获取设置此环境的详细信息。

如果您正在运行`svnserve`，可以通过以下方式检出：

```php
$ svn checkout svn://yourwebsite.example.com/path/to/subversion/website/trunk website 

```

或者，如果您支持高度推荐的 SSH 隧道：

```php
$ svn checkout
svn+ssh://yourwebsite.example.com/path/to/subversion/website/ trunk website

```

相反，如果您有`mod_svn`运行，检出操作就简单多了：

```php
$ svn checkout http://yourwebsite.example.com/subversion/website/ trunk website 

```

或者，如果您有一个安全的服务器：

```php
$ svn checkout https://yourwebsite.example.com/subversion/website/ trunk website 

```

这些命令将创建一个名为`website`的目录，其中包含您网站的代码。

在 Subversion 仓库中添加和删除文件非常直接，类似于 CVS。只需使用此格式从`website`目录中添加文件或目录：

```php
$ svn add file.php 

```

使用此格式来删除文件或目录：

```php
$ svn delete file.php 

```

与 CVS 不同，使用`move`和`copy`命令可以移动文件或复制它们，同时保留它们的修订历史：

```php
$ svn copy file.php newfile.php
$ svn move oldfile.php anotherfile.php 

```

如果您希望替换像`$Id$`或`$Revision$`这样的关键字，您需要手动告诉 Subversion 执行此替换：

```php
$ svn propset svn:keywords "Id Revision" file.php 

```

这应该足以让您开始使用您选择的仓库。

## 智能源代码控制

好的，现在你已经设置并配置了版本控制仓库。太好了！接下来是什么？智能地使用版本控制系统是一个非常重要的步骤。应该遵循基本原理以确保这一点发生。尽管许多是常识，但保持警惕并坚持它们并不容易。

+   定期备份你的仓库，并将它们存储在独立于托管仓库的机器上的媒体上。如果你记得其他任何事情，请记住这一点！

+   只将可工作的代码提交到仓库——在提交前进行测试，以避免像语法错误这样的明显错误。

+   使用标签标记将要部署到实时服务器的可工作代码的点发布。

+   使用分支来同时支持创新和稳定的代码库。

+   如果你有多名开发者，定义一些基本的编码标准（如 PEAR 的编码标准），以便修订之间的差异不包含对空白和其他噪声的虚假更改。

+   如果你有多名开发者，设置一个专门用于提交到仓库的邮件列表。有许多优秀的程序可用于在提交后脚本中使用，将差异发送到邮件列表。确保每个开发者都订阅了该邮件列表。

### 维护分支以支持复杂版本控制

**分支**允许同时开发同一软件的多个版本。例如，当软件达到稳定性，并将添加主要新功能时，从软件中分出一个副本，以便在开发继续的同时，在稳定版本中修复小错误。最佳实践是将稳定版本分支出来，并在 HEAD 上继续开发。

要在分支中使用 CVS 开发稳定的版本 1.2.X：

```php
$ cvs tag -b VERSION_1_2
$ cvs update -b DEVEL_1_2 

```

`update` 命令是一个不直观的要求，但非常重要；如果不更新你的源代码，你将不会编辑分支代码，任何更改最终都会出现在 HEAD 上。

有两个独立的目录是个好主意。例如，当开发 PEAR 版本 1.5.0 时，我使用了两个目录，它们创建的方式如下：

```php
$ cvs -d :pserver:cellog@cvs.php.net:/repository co -r PEAR_1_4 d pear1.4 pear-core
$ cvs -d :pserver:cellog@cvs.php.net:/repository co pear-core 

```

因为我在 `checkout` 命令（缩写为 `co`）中指定了 `-d pear1.4` 选项，所以文件将被检出到 `pear1.4` 目录。`-r PEAR_1_4` 选项检索 PEAR_1_4 分支以修复 PEAR 版本 1.4.X 中的错误。在第二种情况下，默认 HEAD 分支的文件被检出到 `pear-core` 目录。

使用 Subversion 执行相同任务的命令类似于：

```php
$ svn checkout http://yourwebsite.example.com/subversion/website/ trunk website 

```

### 使用标签标记点发布

标签之所以重要，原因很多，其中最重要的是在数据灾难性丢失的情况下能够重建旧版本。使用 CVS 创建标签有两种方法。推荐的方法非常简单：

```php
$ pear cvstag package.xml 

```

此命令解析 `package.xml`，并为每个找到的文件添加 `RELEASE_X_Y_Z` 标签，其中 X_Y_Z 是版本号。版本 1.2.3 将被标记为 `RELEASE_1_2_3`。

从模块检出中手动标记可以通过以下方式完成：

```php
$ cvs tag -r RELEASE_1_2_3 

```

Subversion 不区分标签和分支，因此创建标签和分支之间的唯一区别在于你使用`svn`复制命令将其复制到何处。默认情况下，标签应该使用`svn`复制命令复制到 trunk/tags 分支：

```php
$ svn copy trunk tags/RELEASE_1_2_3
$ svn commit -m "tag release version 1.2.3" 

```

# 解决方案，第二部分：使用 PEAR 安装程序更新网站

到目前为止，你应该熟悉 PEAR 安装程序和源代码控制系统的基本用法。现在我们将把这种知识提升到下一个层次，并了解如何利用这两个工具的优势来管理具有相互依赖关系的多段网站复杂性。首先，重要的是要从离散包和依赖关系的角度考虑网站代码。为此，使用图表通常很有帮助。对于复杂的系统，使用**统一建模语言**（**UML**）来描述系统是非常有意义的，因为这是描述的通用标准。

让我们考察一个现实世界的例子：Chiara 弦乐四重奏网站，[`www.chiaraquartet.net`](http://www.chiaraquartet.net)。截至 2006 年初，这是一个由单个开发者设计的中型网站，但原则可以很好地扩展到多开发者的情况。该网站由多个子站点以及主站点组成。

### 小贴士

在出版前几天，Chiara 弦乐四重奏的网站现在由一个独立开发者管理，不再由作者直接维护，如前所述。作为一个使用`package.xml`管理的网站的例子，在出版时，`pear.php.net`正在迁移到这种方法。查看`cvs.php.net`中的 pearweb 模块，以及`package.php`脚本和相应的`package.xml`以及安装后脚本，这些可以用来设置 MySQL 数据库并配置`http.conf`文件，以开发`pear.php.net`代码的副本。

公共站点包括：

+   [`www.chiaraquartet.net`](http://www.chiaraquartet.net)（主站）

+   [`music.chiaraquartet.net`](http://music.chiaraquartet.net)（MP3s/音频样本）

+   [`calendar.chiaraquartet.net`](http://calendar.chiaraquartet.net)（日程信息）

私人后端站点包括：

+   [`addressbook.chiaraquartet.net`](http://addressbook.chiaraquartet.net)（联系数据库的数据录入）

+   [`database.chiaraquartet.net`](http://database.chiaraquartet.net)（管理通用后端数据）

这些站点各自拥有独立的代码库，但它们在图像和模板等元素上存在相互链接的依赖关系，这些元素用于统一不同站点的外观。此外，随着四重奏事业的成长，网站的需求发生了巨大变化，能够添加新的子站点和删除过时的站点非常重要。

还需要注意的是，需要非常大的文件，例如高分辨率的新闻图片和 MP3 音频剪辑，这些文件不会像 PHP 代码那样频繁更改。

因此，网站可以被分组为几个简单的包：

+   网站

+   网站数据库后端

+   网站联系数据输入后端

+   网站图片

+   网站 MP3 文件

+   网站图片

+   网站 PDF 文件

主要网站包本身可以在稍后日期拆分为单独的包，例如主要网站和网站博客包，而不会受到任何惩罚，这可以通过我们将在本章稍后考察的一些方法实现。

一旦我们确定了网站逻辑分区为包，所需做的就是创建一个私有通道，为每个包生成适当的 `package.xml` 文件，并安装网站。每个组件都可以独立升级——和降级——这使得维护和跟踪更改远不如一个神奇的仪式。

备份数据库的一个美妙技巧不仅是要保存完整的转储，还要将这些转储提交到 Subversion 仓库。这样，你存储的是更小的版本，并且有在远程机器上为开发和测试目的检出数据库转储的能力。

相比 CVS，Subversion 更受欢迎，因为它处理极大文件的方式更为优雅。CVS 在计算两个 150MB 数据库转储之间的差异时，很容易使整个机器崩溃。我是通过艰难的方式学到这一点的。Subversion 在这方面远胜一筹，因为它使用本地副本来计算差异，因此只有差异会被来回发送，从而指数级地减少网络流量。

### 小贴士

重要的一点是，如果数据库中存在任何私有数据，应限制访问为 `svn+ssh`，以减少意外将敏感数据提供给错误人员的可能性。

无需多言，如果存在任何至关重要的敏感数据，例如信用卡号码或身份盗贼渴望获取的数据，绝对不允许对数据进行任何远程访问；相反，应使用像 rdiff-backup（[`www.nongnu.org/rdiff-backup/`](http://www.nongnu.org/rdiff-backup/)）这样的工具。

当移植现有网站时，最好将其添加到 CVS/Subversion 中，保持现有网站的源布局。

开发远程网站的一个困难是需要代码理解它将在不同的 IP 地址上运行，可能还有不同的主机名。有三种处理这个问题的方法：

+   将实时主机名添加到 `/etc/hosts` 文件中，作为本地主机的别名，这样所有请求都会发送到本地主机。

+   使用主机中立的代码，例如依赖于 Apache 网络服务器使用的 `$_SERVER['HTTP_HOST']` 变量，或 `$_SERVER['PHP_SELF']`。

+   使用 PEAR 的替换设施和自定义文件角色来定义每台机器的主机信息。

将实时主机名添加到 `/etc/hosts` 文件中（在 Microsoft Windows 系统上通常是 `C:\WINDOWS\system32\drivers\etc\hosts`），将使得从开发机器实际上无法访问实时网络服务器——或 FTP 服务器——成为不可能，因此这根本不是解决方案。

使用主机中立的代码看起来是个好主意，但正如最近的安全担忧所显示的，跨站脚本（XSS）攻击利用了通过使用这些工具创建的漏洞。虽然避免安全问题是容易的，但它确实增加了相当大的复杂性，使得引入另一个错误或安全问题的可能性高于舒适水平。

第三个选项涉及创建自定义 PEAR 安装器文件角色，定义特殊的配置变量，然后与 PEAR 的替换任务耦合，以自动为每台机器定制文件。

具体来说，我们的示例网站 [`www.chiaraquartet.net`](http://www.chiaraquartet.net)，需要在开发机上设置虚拟主机 `www.chiaraquartet.net, database.chiaraquartet.net, music.chiaraquartet.net` 和 `calendar.chiaraquartet.net`。真麻烦！相反，我创建了两个自定义包，定义了五个配置变量：

+   `root_url:` 这定义了网站的基础 URL。

+   `music_url` : 这定义了网站音乐音频部分的基础 URL。

+   `calendar_url:` 这定义了音乐会和活动日程的基础 URL。

+   `addressbook_url:` 这定义了后端联系列表的基础 URL。

+   `database_url:` 这定义了后端数据库的基础 URL。

在 *开发* 服务器上，我使用以下方式设置这些配置变量的所需值：

```php
$ pear config-set root_url http://localhost
$ pear config-set music_url http://localhost/music
$ pear config-set calendar_url http://localhost/calendar
$ pear config-set addressbook_url http://localhost/addressbook
$ pear config-set database_url http://localhost/database 

```

在 *实时* 服务器上，我使用以下方式设置配置变量的所需值：

```php
$ pear config-set root_url http://www.chiaraquartet.net
$ pear config-set music_url http://music.chiaraquartet.net
$ pear config-set calendar_url http://calendar.chiaraquartet.net
$ pear config-set addressbook_url http://addressbook.chiaraquartet.net
$ pear config-set database_url http://database.chiaraquartet.net 

```

然后，为了直接在源代码中设置这些信息，我使用如下方式：

```php
$ajaxHelper->serverUrl = '@DATABASE-URL@/rest/rest_server.php';

```

如果在 `package.xml` 中指定了这个标签，`@DATABASE-URL@` 的值将被替换为 `database_url` 配置变量的值：

```php
<file name="blah.php" role="php">
<tasks:replace from="@DATABASE-URL@" to="database_url"
type="pear-config" />
</file>

```

在这项工作完成后，当安装在本地 *开发* 机器上时，代码将是：

```php
$ajaxHelper->serverUrl =
'http://localhost/database/rest/rest_server.php';

```

并且在 *实时* 服务器上，代码将是：

```php
$ajaxHelper->serverUrl = 'http://database.chiaraquartet.net/rest/ rest_server.php';

```

最好的是，URL 在两台机器上都能保证正确无误，无需额外工作。这个基本原理也可以应用于开发服务器和实时服务器之间任何重要的差异。

标准库包和网站之间的另一个具体区别是，网站应该安装到公开可访问的目录中，而标准库包文件应该安装到不可访问的位置。为此，我们有默认配置变量，如 `php_dir, data_dir` 和 `test_dir`。对于网页文件没有默认角色。幸运的是，`pearified.com` 通道上确实存在一个自定义文件角色包。要获取此包，请按照以下步骤操作：

```php
$ pear channel-discover pearified.com
$ pear install pearified/Role_Web
$ pear run-scripts pearified/Role_Web 

```

然后，要在 `package.xml` 中使用它，只需简单地写下：

```php
<file name="foo.html" role="web" />

```

此外，您应该指定对 `pearified.com/Role_Web` 的必需依赖关系，以及如 第二章 中所述的 `<usesrole>` 标签。

在这些细节确定之后，是时候生成 PEAR 安装程序所需的 `package.xml` 文件，以便管理网站的安装。

## 从源代码检出生成 package.xml

要生成 `package.xml`，有几种选项可用。对于复杂的网站，最古老和最简单的方法是使用 PEAR_PackageFileManager 包创建一个 `package.xml` 生成脚本。该脚本应生成所需的每个 `package.xml` 文件，使其更新变得简单。此外，它应正确忽略无关的文件和子包。

我们真实网站的生成脚本 [`www.chiaraquartet.net`](http://www.chiaraquartet.net) 是使用这种方法维护的：

```php
<?php
require_once 'PEAR/PackageFileManager2.php';
PEAR::setErrorHAndling(PEAR_ERROR_DIE);

```

下一个部分简单地设置每个子包的 `package.xml` 的发布说明和版本，位于文件顶部的集中位置，这使得编辑文件更容易。

```php
$imageversion = '0.1.0';
$imagenotes = <<<EOT
initial release
EOT;
$mp3version = '0.1.0';
$mp3notes = <<<EOT
initial release
EOT;
$photoversion = '0.1.0';
$photonotes = <<<EOT
initial release
EOT;
$pressversion = '0.1.0';
$pressnotes = <<<EOT
initial release
EOT;
$dataversion = '0.10.3';
$datanotes = <<<EOT
fix saving multiple program items
EOT;
$version = '0.10.0';
$apiversion = '0.1.0';
$notes = <<<EOT
split off database from main package
EOT;

```

接下来，我们通过从每个子包现有的 `package.xml` 中导入来创建每个 `package.xml` 文件。为了简洁起见，我们将省略一些子包。以下是一个典型的例子（网站图片）：

```php
$package_images =
PEAR_PackageFileManager2::importOptions(dirname(__FILE__) .
DIRECTORY_SEPARATOR . 'images' . DIRECTORY_SEPARATOR .
'package.xml',
$options = array(
'ignore' => array('package.xml'),
'filelistgenerator' => 'cvs', // other option is 'file'
'changelogoldtonew' => false,
'baseinstalldir' => 'images',
'packagedirectory' => dirname(__FILE__) . DIRECTORY_SEPARATOR .
'images',
'simpleoutput' => true,
'roles' => array('*' => 'web'),
));
$package_images->setPackageType('php');
$package_images->setReleaseVersion($imageversion);
$package_images->setAPIVersion($imageversion);
$package_images->setReleaseStability('alpha');
$package_images->setAPIStability('alpha');
$package_images->setNotes($imagenotes);
$package_images->clearDeps();
$package_images->resetUsesRole();
$package_images->addUsesRole('web_dir', 'Role_Web', 'pearified.com');
$package_images->addPackageDepWithChannel('required', 'Role_Web',
'pearified.com');
$package_images->setPhpDep('5.1.0');
$package_images->setPearinstallerDep('1.4.3');
$package_images->generateContents();
$package_images->addRelease();
// snip 
$package_data =
PEAR_PackageFileManager2::importOptions(dirname(__FILE__) .
DIRECTORY_SEPARATOR . 'database' . DIRECTORY_SEPARATOR .
'package.xml',
$options = array(
'ignore' => array('package.xml'),
'filelistgenerator' => 'cvs', // other option is 'file'
'changelogoldtonew' => false,
'baseinstalldir' => 'database',
'packagedirectory' => dirname(__FILE__) . DIRECTORY_SEPARATOR .
'database',
'simpleoutput' => true,
'roles' => array('*' => 'web'),
));
$package_data->setPackageType('php');
$package_data->setReleaseVersion($dataversion);
$package_data->setAPIVersion($dataversion);
$package_data->setReleaseStability('alpha');
$package_data->setAPIStability('alpha');
$package_data->setNotes($datanotes);
$package_data->clearDeps();
$package_data->resetUsesRole();
$package_data->addPackageDepWithChannel('required', 'HTML_AJAX',
'pear.php.net');
$package_data->addPackageDepWithChannel('required',
'HTML_Javascript', 'pear.php.net');
$package_data->addPackageDepWithChannel('required', 'XML_RPC2',
'pear.php.net');
$package_data->addUsesRole('web_dir', 'Role_Web', 'pearified.com');
$package_data->addUsesRole('root_url', 'Role_Chiara',
'pear.chiaraquartet.net/private');
$package_data->addUsesRole('music_url', 'Role_Chiara',
'pear.chiaraquartet.net/private');
$package_data->addUsesRole('calendar_url', 'Role_Chiara',
'pear.chiaraquartet.net/private');
$package_data->addUsesRole('database_url', 'Role_Chiara2',
'pear.chiaraquartet.net/private');
$package_data->addPackageDepWithChannel('required', 'Role_Web',
'pearified.com');
$package_data->setPhpDep('5.1.0');
$package_data->setPearinstallerDep('1.4.3');

```

在这里，我们将向所有文件添加替换任务，展示我们针对开发与生产机器所需的定制：

```php
$package_data->addGlobalReplacement('pear-config', '@ROOT-URL@',
'root_url');
$package_data->addGlobalReplacement('pear-config', '@MUSIC-URL@',
'music_url');
$package_data->addGlobalReplacement('pear-config', '@CALENDAR-URL@',
'calendar_url');
$package_data->addGlobalReplacement('pear-config', '@DATABASE-URL@',
'database_url');
$package_data->addGlobalReplacement('pear-config', '@WEB-DIR@',
'web_dir');
$package_data->generateContents();
$package_data->addRelease();
$package_website =
PEAR_PackageFileManager2::importOptions(dirname(__FILE__) .
DIRECTORY_SEPARATOR . 'package.xml',

```

这下一行至关重要；我们需要忽略所有子包的内容，否则它们将被重复，并与父包冲突！

```php
$options = array(
'ignore' => array('package.php', 'package.xml', '*.bak',
'chiaraqu_Chiara', '*.tgz', 'README', 'Chiara_Role/',
'images/', 'photos/', 'music/mp3/', 'press/', 'database/'),
'filelistgenerator' => 'cvs', // other option is 'file'
'changelogoldtonew' => false,
'baseinstalldir' => '/',
'packagedirectory' => dirname(__FILE__),
'simpleoutput' => true,
'roles' => array('*' => 'web'),
));
$package_website->setPackageType('php');
$package_website->setReleaseVersion($version);
$package_website->setAPIVersion($apiversion);
$package_website->setReleaseStability('alpha');
$package_website->setAPIStability('alpha');
$package_website->setNotes($notes);
$package_website->clearDeps();
$package_website->resetUsesRole();
$package_website->addUsesRole('web_dir', 'Role_Web',
'pearified.com');
$package_website->addUsesRole('root_url', 'Role_Chiara',
'pear.chiaraquartet.net/private');
$package_website->addUsesRole('music_url', 'Role_Chiara',
'pear.chiaraquartet.net/private');
$package_website->addUsesRole('calendar_url', 'Role_Chiara',
'pear.chiaraquartet.net/private');
$package_website->addUsesRole('database_url', 'Role_Chiara2',
'pear.chiaraquartet.net/private');
$package_website->addPackageDepWithChannel('required', 'PEAR',
'pear.php.net', '1.4.3');
$package_website->addPackageDepWithChannel('required', 'Role_Web',
'pearified.com');
$package_website->addPackageDepWithChannel('required', 'Role_Chiara',
'pear.chiaraquartet.net/private');
$package_website->addPackageDepWithChannel('required',
'Role_Chiara2', 'pear.chiaraquartet.net/private');

```

在这里，我们为每个子包添加依赖项：

```php
$package_website->addPackageDepWithChannel('required',
'website_photos', 'pear.chiaraquartet.net/private');
$package_website->addPackageDepWithChannel('required',
'website_mp3s', 'pear.chiaraquartet.net/private');
$package_website->addPackageDepWithChannel('required',
'website_press', 'pear.chiaraquartet.net/private');
$package_website->addPackageDepWithChannel('required',
'website_images', 'pear.chiaraquartet.net/private');
$package_website->setPhpDep('5.1.0');
$package_website->setPearinstallerDep('1.4.3');
$package_website->addGlobalReplacement('pear-config', '@ROOT-URL@',
'root_url');
$package_website->addGlobalReplacement('pear-config', '@MUSIC-URL@',
'music_url');
$package_website->addGlobalReplacement('pear-config',
'@CALENDAR-URL@', 'calendar_url');
$package_website->addGlobalReplacement('pear-config',
'@DATABASE-URL@', 'database_url');
$package_website->addGlobalReplacement('pear-config', '@WEB-DIR@',
'web_dir');
$package_website->generateContents();
$package_website->addRelease();

```

最后，我们创建每个 `package.xml` 文件或显示它们以进行错误检查：

```php
if (isset($_SERVER['argv'][1]) && $_SERVER['argv'][1] == 'commit') {
$package_press->writePackageFile();
$package_images->writePackageFile();
$package_mp3s->writePackageFile();
$package_photos->writePackageFile();
$package_website->writePackageFile();
$package_data->writePackageFile();
} else {
$package_press->debugPackageFile();
$package_mp3s->debugPackageFile();
$package_images->debugPackageFile();
$package_photos->debugPackageFile();
$package_website->debugPackageFile();
$package_data->debugPackageFile();
}
?>

```

每个子包都有自己的 PEAR_PackageFileManager2 对象，并从现有的 `package.xml` 中导入选项，仅修改必要的部分。为了创建 `package.xml`，我从 PEAR 包（在这种情况下，是 PEAR 的 `package2.xml`）中复制了一个现有的文件，并修改了 `summary`、`description`、`license` 部分以及维护者列表，以适应网站包。

要使用此脚本，我将其保存为 `package.php`，现在使用 PHP 5.1.0 或更高版本运行它，如下所示：

```php
$ php package.php 

```

这允许我查看 `package.xml` 文件并检查错误。要保存对 `package.xml` 的更改，我运行：

```php
$ php package.php commit 

```

哇！`package.xml` 已在 `website/`、`website/database`、`website/calendar`、`website/press` 和 `website/music` 中创建。

在这个阶段，我们准备开始发布代码并将其部署到测试服务器。

## 打包：协调标签和分支的发布版本

达到这个阶段意味着我们准备开始将我们的网站打包成 PEAR 包以进行安装。在这个时候，过程开始与我们在上一章中学到的打包过程合并。最终，这就是为什么使用 PEAR 安装程序打包网站是一个好主意。安装、升级，甚至回滚网站的过程与安装、升级和回滚任何 PEAR 包没有区别。这个过程使得管理变得极其简单。

当从管理网站的老方法转换为 PEAR 方法时，必须采取几个重要的步骤：

1.  在本地开发服务器上测试发布版本。

1.  如果可能，在部署前立即备份 Live 服务器。

1.  在远程服务器上部署 PEAR 包。

一旦网站成功部署，升级远程服务器上的相应包就是一个简单的过程。

当发布一个版本时，在源代码控制系统中使用标签标记该版本非常重要。如果你选择使用 CVS，标记过程很简单：

```php
$ pear cvstag package.xml 

```

这将自动扫描 `package.xml` 以查找发布中包含的文件，并使用发布版本创建一个名为 `RELEASE_X_Y_Z` 的标签，并将其应用于所有文件。如果发布版本是 0.10.0，则标签将是 `RELEASE_0_10_0`；如果发布版本是 1.2.3，则标签将是 `RELEASE_1_2_3`，依此类推。

使用 Subversion 标记并不是完全自动的，但可以通过以下方式简单完成：

```php
$ svn copy trunk tags/RELEASE_X_Y_Z
$ svn commit -m "tag release X.Y.Z" 

```

按照这些步骤，版本 X.Y.Z 将被标记。

## 在上传前测试发布版本

当你准备部署网站时，创建一个测试包并在本地安装它以确保一切正常非常重要。打包的输出将类似于以下内容：

```php
$ pear package
Analyzing addressbook/data.php
Analyzing addressbook/form.php
Analyzing addressbook/index.php
Analyzing addressbook/list.php
Analyzing addressbook/login.php
Analyzing addressbook/nameparser.php
Analyzing addressbook/usps.php
Analyzing calendar/chiaraSchedule.php
Analyzing calendar/HTMLController.php
Analyzing calendar/HTMLView.php
Analyzing calendar/index.php
Analyzing calendar/schedulelogin.php
Analyzing css/default.css
Analyzing css/history.css
Analyzing css/jonah.css
Analyzing css/music.css
Analyzing css/newindex.css
Analyzing css/news.css
Analyzing css/schedule.css
Analyzing domLib/Changelog
Analyzing domLib/domLib.js
Analyzing domLib/LICENSE
Analyzing domTT/alphaAPI.js
Analyzing domTT/domLib.js
Analyzing domTT/domTT.js
Analyzing domTT/domTT_drag.js
Analyzing music/index.php
Analyzing rest_/list/composers.php
Analyzing rest_/list/concerts.php
Analyzing rest_/list/halls.php
Analyzing rest_/list/pieces.php
Analyzing templates/bio.tpl
Analyzing templates/genericheader.tpl
Analyzing templates/header.tpl
Analyzing templates/history.tpl
Analyzing templates/index.tpl
Analyzing templates/jonahheader.tpl
Analyzing templates/jonahindex.tpl
Analyzing templates/musicbody.tpl
Analyzing templates/news.tpl
Analyzing templates/schedulebody.tpl
Analyzing bio.php
Analyzing editconcerts.html
Analyzing history.php
Analyzing index.php
Analyzing news.php
Warning: in pieces.php: class "getPieces" not prefixed with package name "website"
Warning: in halls.php: class "getHalls" not prefixed with package name "website"
Warning: in concerts.php: class "getConcerts" not prefixed with package name "website"
Warning: in composers.php: class "getComposers" not prefixed with package name "website"
Warning: in index.php: function "err" not prefixed with package name "website"
Warning: in index.php: class "schedulebody" not prefixed with package name "website"
Warning: in news.php: class "test" not prefixed with package name "website"
Warning: in HTMLView.php: class "HTMLView" not prefixed with package name "website"
Warning: in HTMLController.php: class "HTMLController" not prefixed with package name "website"
Warning: in chiaraSchedule.php: class "chiaraSchedule" not prefixed with package name "website"
Warning: in usps.php: class "Services_USPS" not prefixed with package name "website"
Warning: in nameparser.php: class "Spouses" not prefixed with package name "website"
Warning: in nameparser.php: class "CareOf" not prefixed with package name "website"
Warning: in nameparser.php: class "Name" not prefixed with package name "website"
Warning: in nameparser.php: class "NameParser" not prefixed with package name "website"
Warning: in schedulelogin.php: class "Login" not prefixed with package name "website"
Warning: in index.php: class "Page3Display" not prefixed with package name "website"
Warning: in index.php: class "Page1Display" not prefixed with package name "website"
Warning: in index.php: class "MyDisplay" not prefixed with package name "website"
Warning: in index.php: class "Recycle" not prefixed with package name "website"
Warning: in index.php: class "Cancel" not prefixed with package name "website"
Warning: in index.php: class "FinalPage" not prefixed with package name "website"
Warning: in index.php: class "VerifyPage" not prefixed with package name "website"
Warning: in index.php: class "FirstPage" not prefixed with package name "website"
Warning: in form.php: class "HTML_QuickForm_Action_Next" not prefixed with package name "website"
Warning: in data.php: class "ContactAddress" not prefixed with package name "website"
Warning: in data.php: class "Address" not prefixed with package name "website"
Warning: in data.php: class "Data" not prefixed with package name "website"
Warning: Channel validator warning: field "date" - Release Date "2006-04-08" is not today
Package website-0.10.0.tgz done
Tag the released code with `pear cvstag package.xml'
(or set the CVS tag RELEASE_0_10_0 by hand)
$ pear upgrade website-0.10.0.tgz
upgrade ok: channel://pear.chiaraquartet.net/private/website-0.10.0 

```

### 小贴士

**关于所有那些 "Warning: in index.php..." 的事情怎么办？**

这条警告是针对那些正在为官方 PEAR 仓库 [`pear.php.net`](http://pear.php.net) 开发的开发者，旨在帮助捕捉到类命名错误的情况。我们可以通过创建一个自定义通道验证器来轻松消除这些警告，正如在第五章（Chapter 5. 发布到世界：PEAR 通道）中讨论的那样，但这并不是必需的，因为我们知道这些警告是多余的（编写软件或阅读这本书的一个优点！）

接下来，浏览每个页面并点击链接（或者如果你有一个测试套件，运行它）以确保其正常工作。一旦你确信它已经准备好并且可以正常工作，那么就是时候升级 Live 服务器了。

## 升级 Live 服务器

虽然实际上可以在不关闭服务器的情况下升级 Live 服务器，但这通常不是一个好主意。实际上，最好创建一个包含 `index.html` 和 `404.html` 文件的测试目录；如下所示：

```php
<html>
<head>
<title>Upgrading site check back later</title>
</head>
<body>
We are currently upgrading our server, please check back later.
</body>
</html>

```

然后，保存一个类似以下的 `.htaccess` 文件（假设你正在使用 Apache）：

```php
RewriteEngine On
RewriteRule .+ test/index.html

```

这假设你的网络主机在 Apache 中启用了 `mod_rewrite`（并非所有主机都这样做）。更好的是，如果你可以访问 `httpd.conf`，只需将 `DocumentRoot` 的目录更改为测试目录，并添加 `ErrorDocument` 引用。

### 小贴士

由于 PEAR 安装器使用原子文件事务，因此几乎不可能出现半完成的安装。临时网站的目的在于避免用户在出现任何重大问题时看到网站。你可以通过添加一个 RewriteCond 规则来测试网站，该规则指定你的计算机 IP 将忽略该规则，这样你就可以看到完整的网站并检测需要修复的任何问题。

一旦你正确地将网站隐藏在 RewriteRule 之后，就是时候真正升级网站了。首先，使用我们在本章开头学到的`cvs checkout`或`svn checkout`从源代码控制中检出网站的副本。接下来，我们需要创建包并安装它。在我们能够这样做之前，我们需要安装我们创建的所有必要的自定义文件角色：

```php
$ pear channel-discover pearified.com
$ pear install pearified/Role_Web
$ pear channel-discover pear.chiaraquartet.net/private
$ pear install priv/Role_Chiara
$ pear install priv/Role_Chiara2 

```

接下来，我们需要初始化自定义文件角色：

```php
$ pear run-scripts pearified/Role_Web
$ pear config-set root_url http://www.chiaraquartet.net/
$ pear config-set calendar_url http://calendar.chiaraquartet.net/
$ pear config-set database_url http://database.chiaraquartet.net/
$ pear config-set music_url http://music.chiaraquartet.net/
$ pear config-set addressbook_url http://addressbook.chiaraquartet.net/ 

```

一旦完成这些，我们就可以开始安装了：

```php
$ cvs checkout...etc.
$ cd website
$ pear package
$ pear install website-0.10.0.tgz 

```

就这样！现在我们已经完成了初始安装，当需要时升级到下一个版本变得非常简单。

### 使用 pear upgrade 命令

对于本节，我们将使用 CVS 作为我们的源代码控制示例，但如果你使用的是 Subversion 仓库，请替换我们学到的关于 Subversion 的内容。

当你需要修复一个错误或添加一个新功能时，只需修改`package.php package.xml`生成文件中的发布说明：

```php
$version = '0.11.0';
$apiversion = '0.1.0';
$notes = <<<EOT
Add a doohickey to the main page
EOT;

```

然后，创建`package.xml`文件：

```php
$ php package.php commit 

```

最后，提交你的工作：

```php
$ cvs commit -m "add a doohickey to the main page, and prepare for 0.11.0 release" 

```

再次，通过以下方式在本地服务器上进行测试：

```php
$ pear upgrade package.xml 

```

当你在远程服务器上确定它工作正常后，只需运行这些命令：

```php
$ cd website
$ cvs upd -P -d
$ pear package
$ pear cvstag package.xml
$ pear upgrade website-0.11.0.tgz 

```

到这一阶段，你已经成功升级到版本 0.11.0。

如果你发现版本 0.11.0 中存在版本 0.10.0 中不存在的关键错误，会发生什么？幸运的是，修复问题的顺序简单而优雅：

```php
$ cd website
$ pear upgrade -f website-0.10.0.tgz

```

只需两行输入即可。这是假设你保留了 0.10.0 版本的 tarball。即使你没有，这个过程仍然非常简单：

```php
$ cd website
$ cvs upd -r RELEASE_0_10_0 -P -d
$ pear upgrade -f package.xml
$ cvs upd -r HEAD -P -d

```

从技术上讲，返回 CVS 的`HEAD`的最后一条命令对于恢复网站来说并不是真正必要的，但它会在你忘记已经检出`RELEASE_0_10_0`标签时节省未来的麻烦。在 Subversion 中，这个过程同样简单：

```php
$ cd website
$ svn switch http://yourwebsite.example.com/subversion/website/tags/RELEASE_0_10_0
$ pear upgrade -f package.xml
$ svn switch http://yourwebsite.example.com/subversion/website/trunk

```

现在是时候回答那个价值 1000 万美元的问题了：如果你在远程服务器上没有 shell 访问权限，你该如何执行这些任务？（提示：阅读第一章中关于*PEAR_RemoteInstaller*的部分，你将找到答案）。

### 使用梨来解决问题的关键之美

对于网页开发者来说，最糟糕的时刻是在网站上发现严重缺陷。通常很难快速回滚到旧版本。PEAR 安装器使这个过程变得简单。如果测试确定问题是在版本 1.2.3 中引入的，而在版本 1.2.2 中不存在，那么回滚到版本 1.2.2 就像这样：

```php
$ pear upgrade --force website-1.2.2.tgz 

```

或者如果你已经设置了一个频道（我们假设你已经将其别名为`private`）：

```php
$ pear upgrade --force private/website-1.2.2 

```

简称：

```php
$ pear up f private/website-1.2.2 

```

此外，如果你传递了`-o`命令行选项，任何所需的依赖项也将被降级。

# 摘要

在本章中，我们看到了如何使用 PEAR 安装程序来管理一个复杂且快速发展的网站。我们看到了在网站开发中涉及的一些问题，例如代码冲突、文件缺失或多余，以及 PEAR 安装程序如何帮助我们解决这些问题。

我们还看到了如何设置版本控制系统，无论是 CVS 还是 Subversion。最后，我们看到了如何使用 PEAR 安装程序和源代码控制系统来更新一个网站。

在下一章中，你将学习如何在互联网上公开分发你的库和应用程序的方法。
