# 第三章：利用 PEAR 安装程序的全应用支持

在上一章中，我们学习了关于`package.xml`内部的大量知识。在这一章中，我们将强度提高一个档次，探索使我们能够轻松分发 PHP 应用程序并管理其安装和安装后定制的令人兴奋的新功能。

如果你曾经想要使在多个平台、PHP 版本和用户设置上定制 PHP 应用程序变得容易，那么这一章就是为你准备的。

# package.xml 版本 2.0：你的新性感伙伴

本节标题已经说明了所有内容。`package.xml 2.0`比`package.xml 1.0`有重大改进。在 PEAR 安装程序中实现的一些重要新功能，如自定义文件角色/任务、企业级依赖关系和通道，在`package.xml 2.0`中的新标签中得到了反映。此外，结构设计得易于使用其他工具进行验证。

# PEAR 通道：PHP 安装的革命

`package.xml 2.0`最小的增加是`<channel>`标签。但不要被骗了，通道是 PEAR 安装程序中实现的最重要的新功能。通道对于 PEAR 来说，就像依赖关系对于团队开发一样。通过将 PEAR 安装程序的便利性扩展到[pear.php.net](http://pear.php.net)以外的网站，为 PHP 用户提供了许多免费选择。第一次，可以设计一个依赖于`pear.php.net`、`pear.example.com`和任何数量网站的程序，并且所有这些都可以通过单个命令自动下载、安装和轻松升级到最终用户的计算机上。尽管第五章讨论了通道的细节和`channel.xml`通道定义文件，但在设计你的包时，了解通道的工作原理是很好的。

通道有效地解决了两个问题：

+   在多个开发团队之间分配应用程序开发

+   防止冲突的包相互覆盖

一个用户的 PEAR 安装程序可以了解一个通道，在这个例子中，通过[channelserver.example.com](http://channelserver.example.com)了解通道：

```php
$ pear channel-discover channelserver.example.com 

```

一旦用户的 PEAR 安装程序有了这方面的知识，就可以简单地安装来自[channelserver.example.com](http://channelserver.example.com)的包，例如一个假设的名为[Packagename](http://Packagename)的包，使用以下命令：

```php
$ pear install channelserver.example.com/Packagename 

```

用户还可以安装依赖于`channelserver.example.com/Packagename`的包。在 PEAR 通道出现之前，这是不可能的。

当用户简单地输入以下命令时：

```php
$ pear install Package 

```

就像在 PEAR 版本 1.3.6 及更早版本中一样，安装程序使用`default_channel`配置变量，通常为`pear.php.net`或`pecl.php.net`（对于`pecl`命令），然后就像用户输入了以下内容一样操作：

```php
$ pear install pear.php.net/Package 

```

事实上，现在每个现有的 PEAR 包 Foo 都变成了`pear.php.net/Foo`，实际上充当了一个命名空间，将其与`channelserver.example.com/Foo`区分开来。这是频道用来防止冲突包相互覆盖的机制。由于`pear.php.net/Foo`与`channelserver.example.com/Foo`不是同一个包，因此不可能从`pear.php.net/Foo`升级到`channelserver.example.com/Foo`。因此，现在是时候介绍一个重要的概念了：

### 小贴士

尽管频道名称是服务器名称，但它们也充当了一个分类命名方案，用于区分来自不同来源的包。

要理解这一点，我们需要研究一些频道背后的历史。在描述频道的原始草案提案中，频道名称（用于安装和依赖项）和用于访问频道的服务器是不同的。例如，[pear.php.net](http://pear.php.net)频道最初被命名为 PEAR 频道，这样用户就会输入：

```php
$ pear install PEAR/Foo 

```

经过几个 PEAR 的开发版本发布后，很明显，这有几个原因是不好的，其中一个原因就是如果你不知道频道的位置，它就根本无法找到。因此，在 PEAR 的第一个 alpha 版本 1.4.0a1 中，频道的名称与服务器名称相同。

### 小贴士

**我们是否总是必须输入 pear.php.net/Package？**

不，事实上，使使用服务器名称作为频道名称合理化的创新是频道别名的想法。从命令行，我们可以输入以下内容，PEAR 就会安装[pear.php.net/Package](http://pear.php.net/Package)。

```php
$ pear install pear/Package 

```

此外，如果我们愿意，我们可以通过使用`channel-alias`命令将其更改为我们想要的任何名称，如下所示：

```php
$ pear channel-alias pear.php.net p $ pear install p/Package 

```

然而，在`package.xml`的`<channel>`和其他标签中，必须始终使用完整的频道服务器名称，不允许使用别名。

使用服务器名称作为频道名称的转换还有一个期望的结果。最初，可以透明地更改与频道关联的服务器。这在很多层面上都是个坏主意！首先，这意味着一个恶意程序员可以通过简单地更改 PEAR 频道使用的服务器来绕过 PEAR 频道的冲突保护。其次，通过提供与[pear.php.net](http://pear.php.net)上可用的同名包相同名称的包，并在其中隐藏恶意代码，甚至可能欺骗用户在不了解的情况下使用恶意代码。

使用服务器名称作为频道名称，这不再可能，除非对原始频道服务器进行老式的黑客攻击，而这种攻击很快就会被发现。

简而言之，通道的强大之处不仅在于其对最终用户的易用性和对开发者的灵活性，还在于其设计中考虑到的广泛安全性。正如最近在主要 PHP 包（如 XML_RPC 和 phpBB）中出现的安全漏洞所证明的那样，我们不能过于小心。在 PEAR 中，安全性是至关重要的，开发者们已经竭尽全力确保 PEAR 无懈可击。

# 应用支持

到目前为止，我们已经了解到 PEAR 安装程序最初是为了支持库而设计的，然后在 PEAR 版本 1.4.0 中添加了应用支持。让我们通过检查四个令人兴奋的新功能来更详细地了解这意味着什么：自定义文件角色、自定义文件任务、安装后脚本以及将多个包捆绑成一个单一存档的能力。

在本节中，我们将通过创建一个新的自定义文件角色`chiaramdb2schema`、一个自定义文件任务`chiara-managedb`、一个用于填充所需数据的安装后脚本和一个示例应用程序来探索这些功能的细节。然后我们将把角色和任务捆绑在一个单一存档中分发。

### 小贴士

在我们开始之前，你需要检查一个重要的点：

你在解决什么问题？你应该使用自定义文件角色、自定义文件任务、安装后脚本还是其他什么？

自定义文件角色旨在将相关的文件类分组在一起。例如，如果你希望以不同于基于 Web 的 JavaScript 文件的方式安装所有基于 Web 的图像文件，自定义文件角色是完成这一点的最佳方式。

自定义文件任务旨在在安装之前或打包之前操作单个文件的内容。如果你需要将通用模板转换为特定机器的文件（例如，将通用数据库创建 SQL 文件转换为 MySQL 特定或 Oracle 特定 SQL 文件），自定义任务是很好的选择。

安装后脚本旨在允许用户在包准备就绪使用之前执行任何其他高级配置。

我们的示例文件角色和任务是为单用户情况设计的。在共享主机上，这必须通过安装后脚本来完成，因此我们将提供一个脚本，以便系统管理员可以维护多个数据库安装的包。

### 小贴士

**自定义文件角色/任务的命名约定**

对于所有扩展 PEAR 安装程序的功能，使用自定义前缀是一个非常不错的选择。在我们的示例中，如果我们把角色命名为`sql`而不是`chiara_sql`，任务命名为`updatedb`而不是`chiara_updatedb`，那么存在与官方自定义角色或任务冲突的风险，这些角色或任务是从[pear.php.net](http://pear.php.net)分发的。特别是，如果任何角色或任务被认为足够有用，足以成为 PEAR 安装程序默认部分，那么使用你自定义角色/任务的用户将无法升级他们的 PEAR 安装，除非他们卸载该角色及其依赖的所有包。

## 自定义文件角色的介绍

文件角色用于将相关文件分组在一起以处理安装。标准文件角色有 `php, data, doc, test, script, src` 和 `ext`。安装程序会以不同的方式处理每个角色。在名为`My_Package`的包中指定此类标签的文件将被安装到`My/Package/foo.php`。

```php
<file name="foo.php" role="php" baseinstalldir="My/Package"/>

```

然而，具有`data`角色的相同标签会提示安装程序采取非常不同的行动。这个文件不会安装到`My/Package/foo.php`，而是安装到`My_Package/foo.php`。

```php
<file name="foo.php" role="data" baseinstalldir="My/Package"/>

```

`baseinstalldir`属性被`data, doc`和`test`角色忽略，这些角色将根据`package.xml`中定义的相对路径安装到`<package name>/path/to/file`。

此外，每个角色都通过不同的`config`变量确定文件安装的位置。`role: configuration`变量映射如下：

+   `php: php_dir`

+   `data: data_dir`

+   `doc: doc_dir`

+   `test: test_dir`

+   `script: bin_dir`

+   `ext: ext_dir`

+   `src: <none>`（不安装）

通常来说，配置变量与带有`_dir`后缀的文件角色相同，但`role="script"`除外，它应该附加`bin_dir`。此外，注意带有`role="src"`的文件实际上并没有安装。相反，这些文件会被提取出来，然后编译成扩展二进制文件，之后被丢弃。每个角色都有一组特性，使其与其他角色区分开来：

+   有些适用于 PHP 包，而有些适用于扩展包

+   有些被安装，而有些则没有

+   可安装的角色有一个配置变量，用于确定它们应该安装的位置

+   有些遵守`baseinstalldir`属性，而有些则不遵守

+   有些角色安装到`<packagename>/path`，而有些则不是

+   有些代表 PHP 脚本

+   有些代表可执行文件（如脚本）

+   有些代表 PHP 扩展二进制文件

这些特性就是定义自定义文件角色所需的所有内容。实际上，现有的文件角色就是使用这些特性和特殊对象定义的。例如，定义 PHP 角色的代码如下：

```php
<?php
/**
* PEAR_Installer_Role_Php
*
* PHP versions 4 and 5
*
* LICENSE: This source file is subject to version 3.0 of the PHP
* license
* that is available through the world-wide-web at the following URI:
* http://www.php.net/license/3_0.txt. If you did not receive a copy
* of the PHP License and are unable to obtain it through the web,
* please send a note to license@php.net so we can mail you a copy
* immediately.
*
* @category pear
* @package PEAR
* @author Greg Beaver <cellog@php.net>
* @copyright 1997-2005 The PHP Group
* @license http://www.php.net/license/3_0.txt PHP License 3.0
* @version CVS: $Id: Php.php,v 1.5 2005/07/28 16:51:53 cellog Exp
$
* @link http://pear.php.net/package/PEAR
* @since File available since Release 1.4.0a1
*/
/**
* @category pear
* @package PEAR
* @author Greg Beaver <cellog@php.net>
* @copyright 1997-2005 The PHP Group
* @license http://www.php.net/license/3_0.txt PHP License 3.0
* @version Release: @package_version@
* @link http://pear.php.net/package/PEAR
* @since Class available since Release 1.4.0a1
*/
class PEAR_Installer_Role_Php extends PEAR_Installer_Role_Common {}
?>

```

对于大多数角色来说，这可能是唯一需要定义的代码！然而，除了这段 PHP 代码之外，还应安装一个 XML 文件，用于记录角色的属性。PHP 角色的 XML 文件如下：

```php
<role version="1.0">
<releasetypes>php</releasetypes>
<releasetypes>extsrc</releasetypes>
<releasetypes>extbin</releasetypes>
<installable>1</installable>
<locationconfig>php_dir</locationconfig>
<honorsbaseinstall>1</honorsbaseinstall>
<unusualbaseinstall />
<phpfile>1</phpfile>
<executable />
<phpextension />
<config_vars />
</role>

```

各种标签如下：

+   `<releasetypes>:` 这个标签的作用类似于数组，其内容定义了哪些发布类型可以包含此角色。可能的发布类型列表为 `php, extsrc, extbin` 或 `bundle`。

+   `<installable>:` 这个布尔值确定角色是否安装到磁盘上。

+   `<locationconfig>:` 对于可安装的角色，这个字符串值确定用于安装文件的配置变量。

+   `<honorsbaseinstall>:` 这个布尔值（表示为 1 或空标签）确定是否在计算最终安装位置时使用`baseinstalldir`。

+   `<unusualbaseinstall>:` 这个布尔值（表示为 1 或一个空标签）决定了包名是否被添加到安装路径之前。

+   `<phpfile>:` 这个布尔值（表示为 1 或一个空标签）决定了文件是否被视为 PHP 文件（在打包时分析有效的 PHP/类名/函数名）。

+   `<executable>:` 这个布尔值（表示为 1 或一个空标签）决定了在基于 UNIX 的系统上是否将文件与可执行属性一起安装。

+   `<phpextension>:` 这个布尔值（表示为 1 或一个空标签）决定了在覆盖现有扩展二进制文件失败（由于文件锁定）时，安装程序是否会显示一个有用的错误消息。

### 创建 PEAR_Installer_Role_Chiaramdb2schema 自定义角色

首先，重要的是要理解这个角色在`package.xml`中的使用方式。为了实现自定义角色，`package.xml`验证应该能够告诉用户在哪里下载和安装它，因为依赖验证仅在`package.xml`文件从基本结构角度验证之后才会发生。

```php
<package>
<name>Role_Chiaramdb2schema</name>
<channel>pear.chiaraquartet.net</channel>
</package>

```

因此，除了包依赖`package.xml`外，还应包含`<usesrole>`标签，描述所使用的自定义文件角色名称以及包含此角色的包的远程位置。在我们的例子中，如下所示：

```php
<usesrole>
<role>chiaramdb2schema</role>
<package>Role_Chiaramdb2schema</package>
<channel>pear.chiaraquartet.net</channel>
</usesrole>

```

此标签将提示安装程序首先检查`pear.chiaraquartet.net/Role_Chiaramdb2schemaql`包是否已安装。如果没有，安装程序将发出警告：

```php
This package contains role "chiaramdb2schema" and requires package "pear.chiaraquartet.net/Role_Chiaramdb2schema" to be used 

```

### 小贴士

**为什么在依赖关系之外还要使用<usesrole>/<usestask>？**

一旦开始安装，PEAR 安装程序就无法成功配置角色或任务。它们必须在尝试安装使用它们的包之前安装和配置。因此，自定义角色或任务的安装必须在与使用它们的包分开的过程中进行。

在`<file>`标签内使用自定义角色与任何常规角色没有区别。

```php
<file name="dbcontents.xml" role="chiaramdb2schema"/>

```

所有自定义文件角色都实现在一个 PHP 文件中，该文件安装到`PEAR/Installer/Role/`目录。例如，`data`角色位于`PEAR/Installer/Role/Data.php`。与自定义任务不同，自定义文件角色不能在子目录中，因此前缀应该不带下划线，以匹配 PEAR 命名约定。此外，每个自定义角色都必须扩展`PEAR_Installer_Role_Common`类，该类位于`PEAR/Installer/Role/Common.php`。

我们的定制文件角色使用`data_dir`配置变量来确定安装位置，因此在安装方面，它表现得就像`data`角色一样。然而，它通过`Chiaramdb2schema.xml`文件中的此 XML 执行了神奇的事情：

```php
<config_vars>
<chiaramdb2schema_driver>
<type>string</type>
<default />
<doc>MDB2 database driver used to connect to the database</doc>
<prompt>Database driver type. This must be a valid MDB2 driver.
Example drivers are mysql, mysqli, pgsql, sqlite, and so on</prompt>
<group>Database</group>
</chiaramdb2schema_driver>
<chiaramdb2schema_dsn>
<type>string</type>
<default />
<doc>PEAR::MDB2 dsn string[s] for database connection, separated
by ;.
This must be of format:
[user@]host/dbname[;[Package[#schemafile]::]dsn2...]
One default database connection must be specified, and package-
specific databases
may be specified. The driver type and password should be excluded.
Passwords
are set with the chiaramdb2schema_password config variable
</doc>
<prompt>Database connection DSN[s] (no driver/password)</prompt>
<group>Database</group>
</chiaramdb2schema_dsn>
<chiaramdb2schema_password>
<type>string</type>
<default />
<doc>PEAR::MDB2 dsn password[s] for database connection.
This must be of format: password[:password...]
Each DSN in chiaramdb2schema_dsn must match with a password in this
list, or
none will be used. To use no password, simply put another :: like
::::
</doc>
<prompt>Database connection password[s]</prompt>
<group>Database</group>
</chiaramdb2schema_password>
</config_vars>

```

通过这种方式定义`<config_vars>`标签，将向 PEAR 配置添加三个全新的配置变量。它们以与其他配置变量相同的方式操作，并提供将使我们的`chiaramdb2schema`角色变得特殊的信息。

我们的角色利用了基于 MDB2 的架构文件是数据文件的特殊子类这一事实，通过直接扩展`PEAR_Installer_Role_Data`类。以下是我们的示例角色的完整源代码：

```php
<?php
/**
* Custom file role for MDB2_Schema-based database setup files
*
* This file contains the PEAR_Installer_Role_Chiaramdb2schema file * role
*
* PHP versions 4 and 5
*
* @package Role_Chiaramdb2schema
* @author Greg Beaver <cellog@php.net>
* @copyright 2005 Gregory Beaver
* @license http://www.opensource.org/licenses/bsd-license.php BSD
* License
* @version Release: 0.2.0
* @link
http://pear.chiaraquartet.net/index.php?package=Role_Chiaramdb2schema
*/
/**
* Contains the PEAR_Installer_Role_Data class
*/
require_once 'PEAR/Installer/Role/Data.php';
/**
* chiaramdb2schema Custom file role for MDB2_Schema-based database
* setup files
*
* This file role provides the <var>chiaramdb2schema_driver</var>,
* <var>chiaramdb2schema_dsn</var>, and
<var>chiaramdb2schema_password</var>
* configuration variables for use by the chiara-managedb custom task
* to set up and initialize database files
*
* PHP versions 4 and 5
*
* @package Role_Chiaramdb2schema
* @author Greg Beaver <cellog@php.net>
* @copyright 2005 Gregory Beaver
* @license http://www.opensource.org/licenses/bsd-license.php BSD
* License
* @version Release: 0.2.0
* @link http://pear.chiaraquartet.net/index.php?package=Role_Chiaramdb2schema
*/
class PEAR_Installer_Role_Chiaramdb2schema extends
PEAR_Installer_Role_Data
{
}
?>

```

伴随此角色的`Chiaramdb2schema.xml`文件：

```php
<role version="1.0">
<releasetypes>php</releasetypes>
<releasetypes>extsrc</releasetypes>
<releasetypes>extbin</releasetypes>
<installable>1</installable>
<locationconfig>data_dir</locationconfig>
<honorsbaseinstall />
<unusualbaseinstall />
<phpfile />
<executable />
<phpextension />
<config_vars>
<chiaramdb2schema_driver>
<type>string</type>
<default />
<doc>MDB2 database driver used to connect to the database</doc>
<prompt>Database driver type. This must be a valid MDB2 driver.
Example drivers are mysql, mysqli, pgsql, sqlite, and so on</prompt>
<group>Database</group>
</chiaramdb2schema_driver>
<chiaramdb2schema_dsn>
<type>string</type>
<default />
<doc>PEAR::MDB2 dsn string[s] for database connection, separated
by ;.
This must be of format:
[user@]host/dbname[;[Package[#schemafile]::]dsn2...]
One default database connection must be specified, and package-
specific databases
may be specified. The driver type and password should be excluded.
Passwords
are set with the chiaramdb2schema_password config variable
</doc>
<prompt>Database connection DSN[s] (no driver/password)</prompt>
<group>Database</group>
</chiaramdb2schema_dsn>
<chiaramdb2schema_password>
<type>string</type>
<default />
<doc>PEAR::MDB2 dsn password[s] for database connection.
This must be of format: password[:password...]
Each DSN in chiaramdb2schema_dsn must match with a password in this
list, or
none will be used. To use no password, simply put another :: like
::::
</doc>
<prompt>Database connection password[s]</prompt>
<group>Database</group>
</chiaramdb2schema_password>
</config_vars>
</role>

```

就这些了！现在我们已经看到了如何实现一个简单的角色，让我们来探讨自定义文件角色设计内置的可能性的范围。

### 可能的自定义文件角色全范围

大多数自定义文件角色只需要指定如前几节所述的配置变量和属性。然而，有时这还不够，需要一些不寻常的设置。基类`PEAR_Installer_Role_Common`提供的受保护的`setup()`方法专门用于允许文件角色执行任何所需的不寻常设置功能。方法签名如下：

```php
/**
* Do any unusual setup here
* @param PEAR_Installer
* @param PEAR_PackageFile_v2
* @param array file attributes
* @param string file name
*/
function setup(&$installer, $pkg, $atts, $file)

```

参数相当直接：

+   `PEAR_Installer $installer:` 这允许通过`PEAR_Installer`类的公共 API 完成任何专门的安装任务。

+   `PEAR_PackageFile_v2 $pkg:` 这允许从`package.xml`检索对自定义角色可能有用的任何信息。请注意，`PEAR_PackageFile_v2`类的公共 API 是只读的。

+   `array $atts:` 这是从`package.xml`解析出的文件属性，格式类似于以下内容：

    ```php
    array(
    'name' => 'Full/Path/To/File.php',
    'role' => 'customrolename',
    'baseinstalldir' => 'Whatever',
    );

    ```

+   `string $file:` 这是文件名。

注意，`setup()`方法在计算任何安装位置之前为每个角色调用。此外，当前`PEAR_Config`配置对象可通过`$this->config`成员访问。

还需要探索的是自定义文件角色的配置变量定义方式。

`<config_vars>`标签定义配置变量。每个配置变量都使用其名称的标签进行声明。如果您想创建一个名为`foo`的简单配置变量，您将使用以下 XML：

```php
<config_vars>
<foo>
<type>string</type>
<default />
<doc>Foo configuration</doc>
<prompt>Foo protocol login</prompt>
<group>Auth</group>
</foo>
</config_vars>

```

合法的配置类型是`string, directory, file, set`和`password`。如果您希望将可能的输入限制到指定的值，您还需要使用`<valid_set>`标签定义有效值的集合：

```php
<config_vars>
<foo>
<type>set</type>
<default />
<doc>Foo configuration</doc>
<valid_set>bar</valid_set>
<valid_set>baz</valid_set>
<valid_set>gronk</valid_set>
<prompt>Foo protocol type</prompt>
<group>Auth</group>
</foo>
</config_vars>

```

在`PEAR/Config.php`文件中查看现有配置变量组的示例。此变量仅用于信息目的，可以是您想要的任何内容。

另一方面，`<default>`标签有大量的可能性。可以访问三种类型的值来设置配置变量的默认值：

+   现有配置变量的默认值

+   PHP 常量

+   任何文本

为了检索`php_dir`配置变量的默认值，您将使用此标签：

```php
<default><php_dir/></default>

```

只有内置配置变量可以访问其默认值。要访问像`PHP_OS`这样的 PHP 常量，请使用此标签：

```php
<default><constant>PHP_OS</constant></default>

```

注意，在`PEAR/Common.php`或`PEAR/Config.php`中定义的任何常量也将可用作默认值。最后，可以直接使用纯文本，如下所示：

```php
<default><text>hello world</text></default>

```

为了组合这些任务中的几个，只需按所需顺序使用它们即可：

```php
<default><php_dir/><constant>DIRECTORY_SEPARATOR</constant> <text>foo</text></default>

```

如果你希望使用多个常量或多个文本，可以在标签名末尾附加一个数字，如下所示：

```php
<default><text1>.</text1><constant>PATH_SEPARATOR</constant> <text2>mychannel</text2></default>

```

## 自定义文件任务简介

PEAR 随带三个自定义文件任务和一个脚本任务（下一节将讨论安装后脚本）。任务如下：

+   `<tasks:replace/>:` 在已安装或打包的文件上执行基本的`str_replace`操作。可能的替换值来自`package.xml`的信息，来自 PEAR 的配置信息，如`php_dir`的值，或 PHP 常量如`PHP_OS`。

+   `<tasks:windowseol/>:` 这将所有行结束转换为 Windows 的`"\r\n"`行结束符。

+   `<tasks:unixeol/>:` 这将所有行结束转换为 UNIX 的`"\n"`行结束符。

在本节中，我们将检查这些任务如何在 PEAR 代码内部定义，以及如何创建你自己的自定义文件任务。

文件任务通常用于在安装前操作文件的內容。然而，这仅受限于你的想象力。在我们的例子中，我们将使用一个任务来创建和更新数据库结构，在升级时使用我们之前创建的`chiaramdb2schema`文件角色。这个任务是一个非常高级的任务，执行复杂的处理，因此展示了这种系统的多功能性。

自定义任务的 XML 内容的唯一约束是任务的命名空间（通常为`tasks`）必须作为每个标签的前缀。验证由每个任务的 PHP 代码控制。自定义文件任务必须扩展`PEAR_Task_Common`，并且必须位于 PEAR 的`PEAR/Task/`子目录中。与自定义文件角色不同，自定义文件任务可以直接通过使用下划线支持子目录。在我们的示例文件任务`chiara-managedb`中，类名为`PEAR_Task_Chiara_Managedb`，这可以在文件`PEAR/Task/Chiara/Managedb.php`中找到。

有三种自定义文件任务：单个、**多个和脚本**。单个任务在其操作上对单个文件执行，并在文件安装前执行。多个任务在包含任务的每个文件上操作，并在安装完成后执行。脚本任务在安装后使用`run-scripts`命令执行，将在下一节详细说明安装后脚本。此外，任务在文件标签中出现的顺序也很重要。以下可能的但逻辑上不合理的任务顺序会导致`foo.php`中的`@blah@`出现被替换为`data_dir`配置变量的内容。

```php
<file name="foo.php" role="php">
<tasks:replace from="@blah@" to="data_dir" type="pear-config"/>
<tasks:replace from="@blah@" to="version" type="package-info"/>
</file>

```

然而，相反的顺序会导致`foo.php`中的`@blah@`出现，并用`package.xml`中的`<version>`标签的内容替换。

```php
<file name="foo.php" role="php">
<tasks:replace from="@blah@" to="version" type="package-info"/>
<tasks:replace from="@blah@" to="data_dir" type="pear-config"/>
</file>

```

此外，在打包过程中可以执行单个任务。换句话说，有些任务不需要依赖于客户端机器的状态来执行。一个例子是`replace`任务。`package-info`替换仅依赖于`package.xml`文件的内容，该内容在`pear package`时已知。任务执行的时间被称为任务的安装阶段。目前识别的安装阶段有安装和打包。自定义任务可以使用`$phase`属性来控制其安装阶段。定义了三个常量：

+   `PEAR_TASK_INSTALL:` 安装阶段

+   `PEAR_TASK_PACKAGE:` 打包阶段

+   `PEAR_TASK_PACKAGEANDINSTALL:` 安装和打包阶段

因此，例如，`windowseol`任务的阶段声明如下：

```php
var $phase = PEAR_TASK_PACKAGEANDINSTALL;

```

实际的安装阶段由`PEAR_Task_Common`的构造函数设置，可以通过`$installphase`属性访问。唯一合法的值是`PEAR_TASK_INSTALL`和`PEAR_TASK_PACKAGE`。此成员用于确定哪些替换应该发生。例如，如果`$this->installphase`是`PEAR_TASK_PACKAGE`，则不会执行`pear-config`和`php-const`替换。

也许最好的自定义文件任务的介绍是使用在 PEAR 包本身中分发的某些简单任务。最简单的任务是`<tasks:windowseol/>`和`<tasks:unixeol/>`任务。这些将处理其文件的内容并将行结束转换为 Windows 格式或 UNIX 格式。以下是`windowseol`任务的完整源代码：

```php
<?php
/**
* <tasks:windowseol>
*
* PHP versions 4 and 5
*
* LICENSE: This source file is subject to version 3.0 of the PHP
* license that is available through the world-wide-web at the * following URI:
* http://www.php.net/license/3_0.txt. If you did not receive a copy * of the PHP License and are unable to obtain it through the web,
* please send a note to license@php.net so we can mail you a copy
immediately.
*
* @category pear
* @package PEAR
* @author Greg Beaver <cellog@php.net>
* @copyright 1997-2005 The PHP Group
* @license http://www.php.net/license/3_0.txt PHP License 3.0
* @version CVS: $Id: Windowseol.php,v 1.6 2005/10/02 06:29:39
cellog Exp $
* @link http://pear.php.net/package/PEAR
* @since File available since Release 1.4.0a1
*/
/**
* Base class
*/
require_once 'PEAR/Task/Common.php';
/**
* Implements the windows line endings file task.
* @category pear
* @package PEAR
* @author Greg Beaver <cellog@php.net>
* @copyright 1997-2005 The PHP Group
* @license http://www.php.net/license/3_0.txt PHP License 3.0
* @version Release: @package_version@
* @link http://pear.php.net/package/PEAR
* @since Class available since Release 1.4.0a1
*/
class PEAR_Task_Windowseol extends PEAR_Task_Common
{
var $type = 'simple';
var $phase = PEAR_TASK_PACKAGE;
var $_replacements;
/**
* Validate the raw xml at parsing-time.
* @param PEAR_PackageFile_v2
* @param array raw, parsed xml
* @param PEAR_Config
* @static
*/
function validateXml($pkg, $xml, &$config, $fileXml)
{
if ($xml != '') {
return array(PEAR_TASK_ERROR_INVALID, 'no attributes allowed');
}
return true;
}
/**
* Initialize a task instance with the parameters
* @param array raw, parsed xml
* @param unused
*/
function init($xml, $attribs)
{
}
/**
* Replace all line endings with windows line endings
*
* See validateXml() source for the complete list of allowed
fields
* @param PEAR_PackageFile_v1|PEAR_PackageFile_v2
* @param string file contents
* @param string the eventual final file location (informational
only)
* @return string|false|PEAR_Error false to skip this file,
PEAR_Error to fail
* (use $this->throwError), otherwise return the new contents
*/
function startSession($pkg, $contents, $dest)
{
$this->logger->log(3, "replacing all line endings with \\r\\n in $dest");
return preg_replace("/\r\n|\n\r|\r|\n/", "\r\n", $contents);
}
}
?>

```

如您所见，主要操作是在`startSession()`方法中执行的。对于大多数任务来说，这已经足够了。接下来，让我们创建我们自己的自定义文件任务！

### 创建 PEAR_Task_Chiara_Managedb 自定义任务

创建我们的任务的第一步是确定任务期望的目的。在我们的情况下，我们可以用一个需要解决的问题来总结期望的目的。

### 小贴士

问题：安装和更新包使用的数据库是一个繁琐的过程，应该自动化。

更具体地说，我们需要一个能够执行以下任务的解决方案：

+   在包的新安装上从头创建数据库

+   在升级包时更新现有的数据库结构以反映包新版本中的任何更改

+   能够操作大量不同的数据库，并轻松管理在未来日期迁移到不同的数据库

+   根据用户的控制，为不同的包操作不同的数据库

为了满足这些约束，我们将利用可从[`pear.php.net/MDB2_Schema`](http://pear.php.net/MDB2_Schema)获取的 MDB2_Schema 包。此包提供了一些相对于我们从头开始设计的任何自定义解决方案的独特优势：

+   MDB2 支持广泛的数据库驱动程序。

+   用于描述数据库结构的 XML 模式格式是数据库无关的，允许使用 MDB2 支持的任何数据库的用户使用使用此任务的包。

+   有一个广泛的用户基础和几个活跃的维护者，他们帮助确保该包按预期运行。

+   `MDB2_Schema::updateDatabase()`方法能够通过比较两个模式文件来执行数据库的复杂更新。

此外，我们将依赖于一个不太理想的解决方案来满足每个包需要不同数据库的需求：我们将使用`chiaramdb2schema`角色提供的配置变量的所需格式。

为了确定任务是否需要为包提供一个唯一的数据库，我们将在 XML 中添加一个名为`unique`的可选属性。因此，在`package.xml`中，我们的任务有三个合法的可能性：

```php
<tasks:chiara-managedb/>
<tasks:chiara-managedb unique="0"/>
<tasks:chiara-managedb unique="1"/>

```

### 小贴士

**需要 1.5.0a1 或更高版本的 PEAR 才能运行此任务**

不幸的是，在 1.5.0a1 之前的 PEAR 版本中的一个严重错误阻止了此任务的正确使用，因此如果您想尝试它，请确保您已安装了最新的 PEAR 版本。

此外，由于我们需要`chiaramdb2schema`角色以确保我们的配置变量已安装并准备好使用，因此我们将要求任务包含一个名为`chiaramdb2schema`的角色的文件，如下所示：

```php
<file name="blah.xml" role="chiaramdb2schema">
<tasks:chiara-managedb/>
</file>

```

这是我们的任务的 XML 验证方法：

```php
/**
* Validate the raw xml at parsing-time.
* @param PEAR_PackageFile_v2
* @param array raw, parsed xml
* @param PEAR_Config
* @static
*/
function validateXml($pkg, $xml, &$config, $fileXml)
{
if ($fileXml['role'] !='chiaramdb2schema') {
return array(PEAR_TASK_ERROR_INVALID,
'chiara_managedb task can only be ' .
'used with files whose role is chiaramdb2schema.
File is role "' .
$fileXml['role'] . '"');
}
if (isset($xml['attribs'])) {
if (!isset($xml['attribs']['unique'])) {
return array(PEAR_TASK_ERROR_MISSING_ATTRIB, 'unique');
}
if (!in_array($xml['attribs']['unique'], array('0', '1'))) {
return array (PEAR_TASK_ERROR_WRONG_ATTRIB_VALUE, 'unique',
$xml['attribs']['unique'], array('0', '1'));
}
}
return true;
}

```

在运行任务时，我们将使用`unique`属性的值来控制用于连接数据库的数据库 DSN（数据源名称）。因此，以下是我们的初始化方法：

```php
/**
* Initialize a task instance with the parameters
* @param array raw, parsed xml
* @param unused
*/
function init($xml, $attribs)
{
if (isset($xml['attribs']['unique']) &&
$xml['attribs']['unique']) {
$this->_unique = true;
} else {
$this->_unique = false;
}
}

```

到目前为止，这很简单，不是吗？下一步是确定要使用哪个数据库以及如何连接。为此，我们将结合使用`chiaramdb2schema_driver`配置变量、`chiaramdb2schema_dsn`变量和`chiara_mdb2schema_password`变量。

首先，我们将定义一个方法来从这些配置变量中构建数据源名称（DSN）。在分析源代码之前，让我们看看它的全貌：

```php
/**
* parse the chiaramdb2schema_dsn config variable and the
* password variable to determine an actual DSN that should be * used for this task.
* @return string|PEAR_Error
* @access private
*/
function _parseDSN($pkg)
{
// get channel-specific configuration for this variable
$driver = $this->config->get('chiaramdb2schema_driver', null, $pkg->getChannel());
if (!$driver) {
return PEAR::raiseError('Error: no driver set. use
"config-set ' . 'chiaramdb2schema_driver <drivertype>" before installing');
}
$allDSN = $this->config->get('chiaramdb2schema_dsn', null, $pkg->getChannel());
if (!$allDSN) {
return $this->throwError('Error: no dsn set. use
"config-set ' . 'chiaramdb2schema_dsn <dsn>" before installing');
}
$allPasswords = $this->config->get('chiaramdb2schema_password', null, $pkg->getChannel());
$allDSN = explode(';', $allDSN);
$badDSN = array();
$allPasswords = explode(':', $allPasswords);
for ($i = 0; $i < count($allDSN); $i++) {
if ($i && strpos($allDSN[$i], '::')) {
$allDSN[$i] = explode('::', $allDSN[$i]);
$password = (isset($allPasswords[$i]) &&
$allPasswords[$i]) ? $allPasswords[$i] : '';
if (!strpos($allDSN[$i][1], '@')) {
$password = '';
} elseif ($password) {
// insert password into DSN
$a = explode('@', $allDSN[$i][1]);
$allDSN[$i][1] = $a[0] . ':' . $password . '@';
unset($a[0]);
$allDSN[$i][1] .= implode('@', $a);
}
} elseif (!$i && !strpos($allDSN[0], '::')) {
$password = (isset($allPasswords[0]) &&
$allPasswords[0]) ? $allPasswords[0] : '';
if (!strpos($allDSN[0], '@'))
{$password = '';
} elseif ($password) {
// insert password into DSN
$a = explode('@', $allDSN[0]);
$allDSN[0] = $a[0] . ':' . $password . '@';
unset($a[0]);
$allDSN[0] .= implode('@', $a);
}
} else {
// invalid DSN
$badDSN[$i] = $allDSN[$i];
$allDSN[$i] = false;
}
}
if ($this->_unique) {
$lookfor = array($pkg->getPackage(), $pkg->getPackage() . '#' . $this->_file);
foreach ($allDSN as $i => $dsn) {
if (!$i) {
continue;
}
if (strcasecmp($dsn[0], $lookfor[0]) === 0) {
return $driver . '://' . $dsn[1];
}
if (strcasecmp($dsn[0], $lookfor[1]) === 0) {
return $driver . '://' . $dsn[1];
}
}
return $this->throwError('No valid DSNs for package "' .
$pkg->getPackage() . '" were found in config variable
chiaramdb2schema_dsn');
} else {
if (!$allDSN[0]) {
return $this->throwError('invalid default DSN "' .
$badDSN[0] . '" in config variable chiaramdb2schema_dsn');
}
return $driver . '://' . $allDSN[0];
}
}

```

首先，使用构造函数中设置的`$config`成员检索配置变量：

```php
// get channel-specific configuration for this variable
$driver = $this->config->get('chiaramdb2schema_driver', null, $pkg->getChannel());
if (!$driver) {
return PEAR::raiseError('Error: no driver set. use
"config-set ' . 'chiaramdb2schema_driver <drivertype>" before installing');
}
$allDSN = $this->config->get('chiaramdb2schema_dsn', null, $pkg->getChannel());
if (!$allDSN) {
return $this->throwError('Error: no dsn set. use
"config-set ' . 'chiaramdb2schema_dsn <dsn>" before installing');
}
$allPasswords = $this->config->get('chiaramdb2schema_password', null, $pkg->getChannel());

```

为了在用户端简化事情，我们还将尝试检索包的通道配置数据，然后默认使用[pear.php.net](http://pear.php.net)通道配置。

接下来，我们将根据其分隔符";"拆分`DSN`变量，并根据其分隔符":"拆分`Passwords`变量。通过遍历`DSN`变量，我们可以为每个 DSN 插入适当的密码。例如，对于 DSN "user:pass@localhost/databasename"，DSN 将存储为"user@localhost/databasename"，因此我们需要在"@"之前插入":pass"。此外，第一个 DSN 是默认 DSN，如果找到非包特定的 DSN，则使用该 DSN（通过`$allDSN[0]`找到），所以这是一个特殊情况。

```php
$allDSN = explode(';', $allDSN);
$badDSN = array();
$allPasswords = explode(':', $allPasswords);
for ($i = 0; $i < count($allDSN); $i++) {
if ($i && strpos($allDSN[$i], '::')) {
$allDSN[$i] = explode('::', $allDSN[$i]);
$password = (isset($allPasswords[$i]) &&
$allPasswords[$i]) ?
$allPasswords[$i] : '';
if (!strpos($allDSN[$i][1], '@')) {
$password = '';
} elseif ($password) {
// insert password into DSN
$a = explode('@', $allDSN[$i][1]);
$allDSN[$i][1] = $a[0] . ':' . $password . '@';
unset($a[0]);
$allDSN[$i][1] .= implode('@', $a);
}
} elseif (!$i && !strpos($allDSN[0], '::')) {
$password = (isset($allPasswords[0]) &&
$allPasswords[0]) ?
$allPasswords[0] : '';
if (!strpos($allDSN[0], '@')) {
$password = '';
} elseif ($password) {
// insert password into DSN
$a = explode('@', $allDSN[0]);
$allDSN[0] = $a[0] . ':' . $password . '@';
unset($a[0]);
$allDSN[0] .= implode('@', $a);
}
} else {
// invalid DSN
$badDSN[$i] = $allDSN[$i];
$allDSN[$i] = false;
}
}

```

最后，我们将确定包是否需要使用`$this->_unique`指定的特定数据库连接字符串，正如你回忆的那样，这是在`init()`方法中设置的。包特定的 DSN 以包名称为前缀，例如"Packagename::user:password@localhost/databasename"，或者在一个包内的特定文件中，例如"Packagename#file::user:password@localhost/databasename"，因此我们将搜索解析的 DSN，直到找到或失败。

最后，在确定要使用哪个 DSN 之后，我们需要在前面加上应该连接到的数据库类型。例如，这可以是 MySQL、MySQLi、OCI、Firebird、pgSQL 等等。

```php
if ($this->_unique) {
$lookfor = array($pkg->getPackage(), $pkg->getPackage() . '#' . $this->_file);
foreach ($allDSN as $i => $dsn) {
if (!$i) {
continue;
}
if (strcasecmp($dsn[0], $lookfor[0]) === 0) {
return $driver . ':/' . $dsn[1];
}
if (strcasecmp($dsn[0], $lookfor[1]) === 0) {
return $driver . ':/' . $dsn[1];
}
}
return $this->throwError('No valid DSNs for package "' .
$pkg->getPackage() .
'" were found in config variable
chiaramdb2schema_dsn');
} else {
if (!$allDSN[0]) {
return $this->throwError('invalid default DSN "' .
$badDSN[0] . '" in config variable
chiaramdb2schema_dsn');
}
return $driver . ':/' . $allDSN[0];
}

```

如果你发现你的眼睛开始发花，不要害怕。重要的是要意识到，在体验结束时，该方法将返回一个包含详细错误信息的`PEAR_Error`，或者一个像**"mysqli://user:pass@localhost/databasename"**这样的字符串。

我们自定义任务的最后一部分是`startSession()`方法，它实际上执行任务，因为这是一个类型为单的任务。

```php
/**
* Update the database.
*
* First, determine which DSN to use from the
* chiaramdb2schema_dsn config variable
* with {@link _parseDSN()}, then determine whether the database
* already exists based
* on the contents of a previous installation, and finally use
* {@link MDB2_Schema::updateDatabase()} * to update the database itself
*
* PEAR_Error is returned on any problem.
* See validateXml() source for the complete list of allowed fields
* @param PEAR_PackageFile_v2
* @param string file contents
* @param string the eventual final file location * (informational only)
* @return string|false|PEAR_Error false to skip this file,
* PEAR_Error to fail
* (use $this->throwError), otherwise return the new contents
*/
function startSession($pkg, $contents, $dest)
{
$this->_file = basename($dest);
$dsn = $this->_parseDSN($pkg);
if (PEAR::isError($dsn)) {
return $dsn;
}
require_once 'MDB2/Schema.php';
require_once 'System.php';
$tmp = System::mktemp(array('foo.xml'));
if (PEAR::isError($tmp)) {
return $tmp;
}
$fp = fopen($tmp, 'wb');
fwrite($fp, $contents);
fclose($fp);
$schema = &MDB2_Schema::factory($dsn);
$reg = &$this->config->getRegistry();
if ($installed && file_exists($dest)) {
// update existing database
$res = $schema->updateDatabase($tmp, $dest);
if (PEAR::isError($res)) {
return PEAR::raiseError($res->getMessage() . $res->getUserInfo());
}
} else {
// create new database
$res = $schema->updateDatabase($tmp);
if (PEAR::isError($res)) {
return PEAR::raiseError($res->getMessage() . $res->getUserInfo());
}
}
// unmodified
return $contents;
}

```

`MDB2_Schema::updateDatabase()`需要两个模式文件才能升级数据库。在升级数据库时，我们将使用最终的安装目标`$dest`来确定我们是否正在替换现有的模式文件。如果是这样，则将其传递给`updateDatabase()`。否则，我们只需调用`updateDatabase()`来创建新的数据库结构。

注意，在此阶段，文件内容尚未写入磁盘，因为任务是在安装之前对文件进行操作的。因此，我们将使用包含在 PEAR 包中的`System`类创建的临时位置写入模式文件。

任务的大部分工作由`MDB2_Schema`类执行。在完成任务后，用户的数据库将在安装和升级时自动配置。

### 可能的所有自定义文件任务的全范围

可用于自定义任务的方法有：

+   `true|array validXml($pkg, $xml, &$config, $fileXml):` 验证任务`XML`

+   `void init($xml, $fileAttributes, $lastVersion):` 初始化任务

+   `true|PEAR_Error startSession($pkg, $contents, $dest):` 开始（通常完成）任务处理

+   `true|PEAR_Error run($tasks):` 仅对类型为"multiple"的任务，处理所有任务并执行所需操作

#### `validXml($pkg, $xml, &$config, $fileXml)`

这种方法在`package.xml`验证期间用于所有三种类型的任务，以验证特定任务的 XML。`$pkg`是一个表示包含任务的`package.xml`的`PEAR_PackageFile_v2`对象。这是只读的，应该仅用于检索信息。`$xml`是文件任务的解析内容，`$config`是一个表示当前配置的`PEAR_Config`对象，而`$fileXml`是`package.xml`文件标签的解析内容。

这里是一些示例任务 XML 和`$xml`变量内容的简单映射：

| XML | 解析内容 |
| --- | --- |
| `<tasks:something/>` | `''` |
| `<tasks:something att="blah"/>` | `array('attribs' => array('att' => 'blah'))` |
| `<tasks:something>blah</tasks:something>` | `'blah'` |
| `<tasks:something att="blah">blah2</tasks:something>` | `array('attribs' => array('att' => 'blah'), '_content' => 'blah2')` |
| `<tasks:something> <tasks:subtag>hi</tasks:subtag> </tasks:something>` | `array('tasks:subtag' => 'hi')` |
| `<tasks:something> <tasks:subtag>hi</tasks:subtag> <tasks:subtag att="blah">again</tasks:subtag> </tasks:something>` | `array('tasks:subtag' => array(0 => 'hi', 1 => array('attribs' => array('att' => 'blah'), '_content' => 'again'))))` |

`$fileXml` 参数将包含一个此格式的数组，其中包含在 `<file>` 标签中定义的所有属性。

```php
array('attribs' => array('name' => 'Filename', 'role' =>
'filerole',...));

```

错误应返回为数组。第一个索引必须是以下错误代码之一：

+   `PEAR_TASK_ERROR_NOATTRIBS:` 数组应返回为：`array(PEAR_TASK_ERROR_NOATTRIBS)`;

+   `PEAR_TASK_ERROR_MISSING_ATTRIB:` 数组应返回为：`array(PEAR_TASK_ERROR_MISSING_ATTRIB, 'attributename')`;

+   `PEAR_TASK_ERROR_WRONG_ATTRIB_VALUE:` 数组应返回为：`array(PEAR_TASK_ERROR_WRONG_ATTRIB_VALUE, 'attributename', 'actualvalue', ['expectedvalue'|array('expectedvalue1', 'expectedvalue2',...)])`;

+   `PEAR_TASK_ERROR_INVALID:` 数组应返回为：`array(PEAR_TASK_ERROR_INVALID, 'unusual error message')`;

#### init($xml, $fileAttributes, $lastVersion)

`init()` 方法被调用以初始化所有非脚本任务，并且可用于任何目的。三个参数是：

+   `mixed $xml:` 表示 `package.xml` 中任务的 XML 的数组。这与传递给 `validXml()` 的 `$xml` 参数的格式相同。

+   `array $fileAttributes:` 表示文件属性的数组。这与传递给 `validXml()` 的 `$fileXml` 参数的格式相同。

+   `string|NULL $lastVersion:` 如果正在升级包，则为包的最后一个安装版本，如果这是第一次安装此包，则为 `NULL`。这可以用于依赖于先前安装配置的任务。

`init()` 的任何返回值都将被丢弃。

#### startSession($pkg, $contents, $dest)

`startSession()` 方法被调用以执行任务，并在 `init()` 方法之后调用。需要注意的是，此方法预期返回文件的精确内容，因为它应该被安装到磁盘上。不应在磁盘上修改文件。如果在运行任务时发生任何错误，应返回一个包含描述问题的清晰错误消息的 `PEAR_Error` 对象，并包含包含任务的文件信息。

如果任务确定此文件不应安装，返回 `FALSE` 将提示安装程序跳过此文件的安装。请注意，只有字面量 `FALSE` 会导致跳过安装；空字符串、数字 0 和任何其他可以用作假条件的字面量都不会影响安装。

在任务成功执行后，必须返回完整的文件内容。返回值用于将文件内容写入磁盘。例如，`windowseol` 任务在将所有新行转换为 `\r\n` 后返回 `$contents` 的值。

传递给 `startSession()` 的参数是：

+   `PEAR_PackageFile_v2 $pkg:` 代表包含此任务的完整 `package.xml` 的包文件对象。

+   `string $contents:` 文件的完整内容，可以在任务成功完成后对其进行操作并返回。

+   `string $dest:` 文件最终安装位置的完整路径。这仅用于信息用途。

#### run($tasks)

此方法仅对类型为多重的任务调用。`$tasks` 参数是 `package.xml` 中每个多重任务的数组。例如，如果 `package.xml` 包含类型为多重的 `<tasks:foo/>` 和 `<tasks:bar/>` 任务，`run()` 方法将为所有 `foo` 任务调用，并且 `$tasks` 参数将包含每个 `foo` 任务的数组。然后，相同的程序将重复用于 `bar` 任务。

`run()` 方法在安装成功完成后被调用，因此可以操作包的已安装内容。

在出错时，`run()` 方法应返回一个包含有关任务失败原因的详细信息的 `PEAR_Error` 对象。所有其他返回值都被忽略。

## 适用于终极定制的安装后脚本

第三个也是最后一个任务是安装后脚本。这些是最强大和可定制的任务，可以字面意义上用于执行安装所需的任何定制。PEAR 安装程序通过在 `package.xml` 文件中定义一系列问题来实施安装后脚本，并通过传递用户给出的答案到一个特殊的 PHP 文件中来执行。以下是一组简单的问题和相应的安装后脚本：

首先，`package.xml` 中的 XML：

```php
<file name="rolesetup.php" role="php">
<tasks:postinstallscript>
<tasks:paramgroup>
<tasks:id>setup</tasks:id>
<tasks:param>
<tasks:name>channel</tasks:name>
<tasks:prompt>Choose a channel to modify configuration
values from</tasks:prompt>
<tasks:type>string</tasks:type>
<tasks:default>pear.php.net</tasks:default>
</tasks:param>
</tasks:paramgroup>
<tasks:paramgroup>
<tasks:id>driver</tasks:id>
<tasks:instructions>
In order to set up the database, please choose a database
driver.
This should be a MDB2-compatible driver name, such as mysql, mysqli,
Pgsql, oci8, etc.
</tasks:instructions>
<tasks:param>
<tasks:name>driver</tasks:name>
<tasks:prompt>Database driver?</tasks:prompt>
<tasks:type>string</tasks:type>
</tasks:param>
</tasks:paramgroup>
<tasks:paramgroup>
<tasks:id>choosedsn</tasks:id>
<tasks:param>
<tasks:name>dsnchoice</tasks:name>
<tasks:prompt>%sChoose a DSN to modify, or to add a new
dsn, type
&quot;new&quot;. To remove a DSN prepend with
&quot;!&quot;</tasks:prompt>
<tasks:type>string</tasks:type>
<tasks:default>new</tasks:default>
</tasks:param>
</tasks:paramgroup>
<tasks:paramgroup>
<tasks:id>deletedsn</tasks:id>
<tasks:param>
<tasks:name>confirm</tasks:name>
<tasks:prompt>Really delete &quot;%s&quot; DSN? (yes to
delete)</tasks:prompt>
<tasks:type>string</tasks:type>
<tasks:default>no</tasks:default>
</tasks:param>
</tasks:paramgroup>
<tasks:paramgroup>
<tasks:id>modifydsn</tasks:id>
<tasks:name>choosedsn::dsnchoice</tasks:name>
<tasks:conditiontype>!=</tasks:conditiontype>
<tasks:value>new</tasks:value>
<tasks:param>
<tasks:name>user</tasks:name>
<tasks:prompt>User name</tasks:prompt>
<tasks:type>string</tasks:type>
</tasks:param>
<tasks:param>
<tasks:name>password</tasks:name>
<tasks:prompt>Database password</tasks:prompt>
<tasks:type>password</tasks:type>
</tasks:param>
<tasks:param>
<tasks:name>host</tasks:name>
<tasks:prompt>Database host</tasks:prompt>
<tasks:type>string</tasks:type>
<tasks:default>localhost</tasks:default>
</tasks:param>
<tasks:param>
<tasks:name>database</tasks:name>
<tasks:prompt>Database name</tasks:prompt>
<tasks:type>string</tasks:type>
</tasks:param>
</tasks:paramgroup>
<tasks:paramgroup>
<tasks:id>newpackagedsn</tasks:id>
<tasks:param>
<tasks:name>package</tasks:name>
<tasks:prompt>Package name</tasks:prompt>
<tasks:type>string</tasks:type>
</tasks:param>
<tasks:param>
<tasks:name>host</tasks:name>
<tasks:prompt>Database host</tasks:prompt>
<tasks:type>string</tasks:type>
<tasks:default>localhost</tasks:default>
</tasks:param>
<tasks:param>
<tasks:name>user</tasks:name>
<tasks:prompt>User name</tasks:prompt>
<tasks:type>string</tasks:type>
<tasks:default>root</tasks:default>
</tasks:param>
<tasks:param>
<tasks:name>password</tasks:name>
<tasks:prompt>Database password</tasks:prompt>
<tasks:type>password</tasks:type>
</tasks:param>
<tasks:param>
<tasks:name>database</tasks:name>
<tasks:prompt>Database name</tasks:prompt>
<tasks:type>string</tasks:type>
</tasks:param>
</tasks:paramgroup>
<tasks:paramgroup>
<tasks:id>newdefaultdsn</tasks:id>
<tasks:param>
<tasks:name>host</tasks:name>
<tasks:prompt>Database host</tasks:prompt>
<tasks:type>string</tasks:type>
<tasks:default>localhost</tasks:default>
</tasks:param>
<tasks:param>
<tasks:name>user</tasks:name>
<tasks:prompt>User name</tasks:prompt>
<tasks:type>string</tasks:type>
<tasks:default>root</tasks:default>
</tasks:param>
<tasks:param>
<tasks:name>password</tasks:name>
<tasks:prompt>Database password</tasks:prompt>
<tasks:type>password</tasks:type>
</tasks:param>
<tasks:param>
<tasks:name>database</tasks:name>
<tasks:prompt>database name</tasks:prompt>
<tasks:type>string</tasks:type>
</tasks:param>
</tasks:paramgroup>
</tasks:postinstallscript>
</file>

```

然后，安装后脚本（`rolesetup.php` 的内容）：

```php
<?php
/**
* Post-installation script for the Chiara_Managedb task.
*
* This script takes user input on DSNs and sets up DSNs, allowing
* the addition of one custom DSN per iteration.
* @version @package_version@
*/
class rolesetup_postinstall
{
/**
* object representing package.xml
* @var PEAR_PackageFile_v2
* @access private
*/
var $_pkg;
/**
* Frontend object
* @var PEAR_Frontend
* @access private
*/
var $_ui;
/**
* @var PEAR_Config
* @access private
*/
var $_config;
/**
* The actual DSN value as will be saved to the configuration file
* @var string
*/
var $dsnvalue;
/**
* The actual password value as will be saved to the * configuration file
* @var string
*/
var $passwordvalue;
/**
* The channel to modify configuration values from
*
* @var string
*/
var $channel;
/**
* The task object used for dsn serialization/unserialization
* @var PEAR_Task_Chiara_Managedb
*/
var $managedb;
/**
* An "unserialized" array of DSNs parsed from the chiaramdb2schema
* configuration variables.
* @var array
*/
var $dsns;
/**
* The index of the DSN in $this->dsns we will be modifying
* @var string
*/
var $choice;
/**
* Initialize the post-installation script
*
* @param PEAR_Config $config
* @param PEAR_PackageFile_v2 $pkg
* @param string|null $lastversion Last installed version. * Not used in this script
* @return boolean success of initialization
*/
function init(&$config, &$pkg, $lastversion)
{
require_once 'PEAR/Task/Chiara/Managedb.php';
$this->_config = &$config;
$this->_ui = &PEAR_Frontend::singleton();
$this->managedb = new PEAR_Task_Chiara_Managedb($config,
$this->_ui, PEAR_TASK_INSTALL);
$this->_pkg = &$pkg;
if (!in_array('chiaramdb2schema_dsn', $this->_config->getKeys())) {
// fail: role was not installed?
return false;
}
$this->channel = $this->_config->get('default_channel');
$this->dsns = PEAR::isError( $e = $this->managedb->unserializeDSN($pkg)) ? array() : $e;
return true;
}
/**
* Set up the prompts properly for the script
*
* @param array $prompts
* @param string $section
* @return array
*/
function postProcessPrompts($prompts, $section)
{
switch ($section) {
case 'driver' :
if ($this->driver) {
$prompts[0]['default'] = $this->driver;
}
break;
case 'deletedsn' :
$count = 1;
foreach ($this->dsns as $i => $dsn) {
$text = ($i ? "(Package $i) " : '') . $dsn;
if ($count == $this->choice) {
break;
}
$count++;
}
$prompts[0]['prompt'] = sprintf($prompts[0]['prompt'], $text);
break;
case 'choosedsn' :
$text = '';
$count = 1;
foreach ($this->dsns as $i => $dsn) {
$text .= "[$count] " . ($i ? "(Package $i) " : '') . $dsn . "\n";
$count++;
}
$prompts[0]['prompt'] =
sprintf($prompts[0]['prompt'], $text);
break;
case 'modifydsn' :
$count = 1;
$found = false;
foreach ($this->dsns as $i => $dsn) {
if ($count == $this->choice) {
$found = true;
break;
}
$count++;
}
if ($found) {
$dsn = MDB2::parseDSN($this->dsns[$i]);
// user
$prompts[0]['default'] = $dsn['username'];
// password
if (isset($dsn['password'])) {
$prompts[1]['default'] = $dsn['password'];
}
// host
$prompts[2]['default'] = $dsn['hostspec'];
if (isset($dsn['port'])) {
$prompts[2]['default'] .= ':' . $dsn['port'];
}
// database
$prompts[3]['default'] = $dsn['database'];
}
break;
}
return $prompts;
}
/**
* Run the script itself
*
* @param array $answers
* @param string $phase
*/
function run($answers, $phase)
{
switch ($phase) {
case 'setup' :
return $this->_doSetup($answers);
break;
case 'driver' :
require_once 'MDB2.php';
PEAR::pushErrorHandling(PEAR_ERROR_RETURN);
if (PEAR::isError($err =
MDB2::loadFile('Driver' . DIRECTORY_SEPARATOR .
$answers['driver']))) {
PEAR::popErrorHandling();
$this->_ui->outputData( 'ERROR: Unknown MDB2 driver "' .
$answers['driver'] . '": ' .
$err->getUserInfo() . '. Be sure you have
installed ' . 'MDB2_Driver_' .
$answers['driver']);
return false;
}
PEAR::popErrorHandling();
$ret = $this->_config->set('chiaramdb2schema_driver',
$answers['driver'],
'user', $this->channel);
return $ret && $this->_config->writeConfigFile();
break;
case 'choosedsn' :
if ($answers['dsnchoice'] && $answers['dsnchoice']{0} == '!') {
// delete a DSN
$answers['dsnchoice'] =
substr($answers['dsnchoice'], 1);
} else {
$this->_ui->skipParamgroup('deletedsn');
}
if ($answers['dsnchoice'] > count($this->dsns)) {
$this->_ui->outputData('ERROR: No suchdsn "' .
$answers['dsnchoice'] . '"');
return false;
}
$this->choice = $answers['dsnchoice'];
break;
case 'deletedsn' :
$this->_ui->skipParamgroup('modifydsn');
$this->_ui->skipParamgroup('newpackagedsn');
$this->_ui->skipParamgroup('newdefaultdsn');
if ($answers['confirm'] == 'yes') {
$count = 1;
foreach ($this->dsns as $i => $dsn) {
if ($count == $this->choice) {
unset($this->dsns[$i]);
break;
}
$count++;
}
$this->_ui->outputData('DSN deleted');
$this->managedb->serializeDSN($this->dsns, $this->channel);
return true;
} else {
$this->_ui->outputData('No changes performed');
}
break;
case 'modifydsn' :
$count = 1;
$found = false;
foreach ($this->dsns as $i => $dsn) {
if ($count == $this->choice) {
$found = true;
break;
}
$count++;
}
if (!$found) {
$this->_ui->outputData('ERROR: DSN "' . $this->choice . '" not found!');
return false;
}
$dsn = $answers['user'] . ':' . $answers['password'] . '@' .
$answers['host'] . '/' . $answers['database'];
$this->dsns[$i] = $dsn;
$this->managedb->serializeDSN($this->dsns, $this->channel);
$this->_ui->skipParamgroup('newpackagedsn');
$this->_ui->skipParamgroup('newdefaultdsn');
break;
case 'newpackagedsn' :
$dsn = $answers['user'] . ':' . $answers['password'] . '@' .
$answers['host'] . '/' . $answers['database'];
$this->dsns[$answers['package']] = $dsn;
$this->managedb->serializeDSN($this->dsns, $this->channel);
$this->_ui->skipParamgroup('newdefaultdsn');
break;
case 'newdefaultdsn' :
$dsn = $answers['user'] . ':' . $answers['password'] . '@' .
$answers['host'] . '/' . $answers['database'];
$this->dsns[0] = $dsn;
$this->managedb->serializeDSN($this->dsns, $this->channel);
break;
case '_undoOnError' :
// answers contains paramgroups that succeeded in
// reverse order foreach ($answers as $group) {
}
break;
}
return true;
}
/**
* Run the setup paramgroup
*
* @param array $answers
* @return boolean
* @access private
*/
function _doSetup($answers)
{
$reg = &$this->_config->getRegistry();
if (!$reg->channelExists($answers['channel'])) {
$this->_ui->outputData('ERROR: channel "' .
$answers['channel'] . '" is not registered, use the channel-discover command');
return false;
}
$this->channel = $answers['channel'];
$this->driver = $this->_config->get('chiaramdb2schema_driver', null, $this->channel);
$this->dsnvalue = $this->_config->get('chiaramdb2schema_dsn',
null, $this->channel);
$this->passwordvalue = $this->_config->get('chiaramdb2schema_dsn', null,
$this->channel);
if (!$this->dsnvalue) {
// magically skip the "choosedsn", "deleteDSN" and
// "modifydsn" <paramgroup>s,
// and only create a new, default DSN
$this->_ui->skipParamgroup('choosedsn');
$this->_ui->skipParamgroup('deletedsn');
$this->_ui->skipParamgroup('modifydsn');
$this->_ui->skipParamgroup('newpackagedsn');
}
return true;
}
}
?>

```

安装后脚本与 PEAR 提供的不同前端紧密交互。脚本有许多可用可能性。除了使用用户提供的数据外，安装后脚本可以基于用户的先前答案交互式地修改提示，并且可以动态地跳过整个 `<tasks:paramgroup>` 部分。这些功能允许对实际脚本的显著定制。

#### 安装后脚本的组成部分

每个安装后脚本必须定义两个方法，`init()` 和 `run()`。`init()` 方法应定义得类似于这样：

```php
/**
* Initialize the post-installation script
*
* @param PEAR_Config $config
* @param PEAR_PackageFile_v2 $pkg
* @param string|null $lastversion Last installed version. * Not used in this script
* @return boolean success of initialization
*/
function init(&$config, &$pkg, $lastversion)
{
require_once 'PEAR/Task/Chiara/Managedb.php';
$this->_config = &$config;
$this->_ui = &PEAR_Frontend::singleton();
$this->managedb = new PEAR_Task_Chiara_Managedb($config, $this->_ui,
PEAR_TASK_INSTALL);
$this->_pkg = &$pkg;
if (!in_array('chiaramdb2schema_dsn', $this->_config->getKeys())) {
// fail: role was not installed?
return false;
}
$this->channel = $this->_config->get('default_channel');
$this->dsns = PEAR::isError($e = $this->managedb->unserializeDSN($pkg)) ? array() : $e;
return true;
}

```

注意使用`$this->_ui = &PEAR_Frontend::singleton():` 这行代码打开了一个巨大的可能性。除了公开可用的整个公共 API 以显示文本外，还包括：

+   `Void outputData(string $text):` 向用户显示信息

+   `string bold(string $text):` 接受文本并返回该文本的粗体转换版本，然后可以将其传递给`outputData()`

这使得`skipParamGroup(string $id)`方法可用。`$id`参数应该是尚未执行的 paramgroup 的 ID（来自`<tasks:paramgroup>`标签的`<tasks:id>`标签的内容）。

通过创建一个名为`postProcessPrompts()`的方法来修改提示或参数的默认值，如下所示：

```php
/**
* Set up the prompts properly for the script
*
* @param array $prompts
* @param string $section
* @return array
*/
function postProcessPrompts($prompts, $section)
{
switch ($section) {
case 'driver' :
if ($this->driver) {
$prompts[0]['default'] = $this->driver;
}
break;
case 'deletedsn' :
$count = 1;
foreach ($this->dsns as $i => $dsn) {
$text = ($i ? "(Package $i) " : '') . $dsn;
if ($count == $this->choice) {
break;
}
$count++;
}
$prompts[0]['prompt'] =
sprintf($prompts[0]['prompt'], $text);
break;
case 'choosedsn' :
$text = '';
$count = 1;
foreach ($this->dsns as $i => $dsn) {
$text .= "[$count] " . ($i ? "(Package $i) " :
'') . $dsn . "\n";
$count++;
}
$prompts[0]['prompt'] =
sprintf($prompts[0]['prompt'], $text);
break;
case 'modifydsn' :
$count = 1;
$found = false;
foreach ($this->dsns as $i => $dsn) {
if ($count == $this->choice) {
$found = true;
break;
}
$count++;
}
if ($found) {
$dsn = MDB2::parseDSN($this->dsns[$i]);
// user
$prompts[0]['default'] = $dsn['username'];
// password
if (isset($dsn['password'])) {
$prompts[1]['default'] = $dsn['password'];
}
// host
$prompts[2]['default'] = $dsn['hostspec'];
if (isset($dsn['port'])) {
$prompts[2]['default'] .= ':' . $dsn['port'];
}
// database
$prompts[3]['default'] = $dsn['database'];
}
break;
}
return $prompts;
}

```

`$prompts`参数将是`<tasks:paramgroup>`标签的解析内容。

```php
<tasks:paramgroup>
<tasks:id>databaseSetup</tasks:id>
<tasks:param>
<tasks:name>database</tasks:name>
<tasks:prompt>%s database name</tasks:prompt>
<tasks:type>string</tasks:type>
<tasks:default>pear</tasks:default>
</tasks:param>
<tasks:param>
<tasks:name>user</tasks:name>
<tasks:prompt>%s database username</tasks:prompt>
<tasks:type>string</tasks:type>
<tasks:default>%s_pear</tasks:default>
</tasks:param>
</tasks:paramgroup>

```

对于这个`paramgroup`，`$prompts`变量如下所示：

```php
array(
'id' => 'databaseSetup';
'param' =>
array(
array(
'name' => 'database',
'prompt' => '%s database name',
'type' => 'string',
'default' => 'pear',
),
array(
'name' => 'user',
'prompt' => '%s database username',
'type' => 'string',
'default' => '%s_pear',
),
),
);

```

`postProcessPrompts()`方法应该返回经过修改的`$prompts`数组，仅修改提示和默认字段。如果修改了其他内容，将导致安装后脚本直接失败。

例如，在确定用户正在使用 pgSQL 驱动程序后，`postProcessPrompts()`的返回值可能是：

```php
array(
'id' => 'databaseSetup';
'param' =>
array(
array(
'name' => 'database',
'prompt' => 'Postgresql database name',
'type' => 'string',
'default' => 'pear',
),
array(
'name' => 'user',
'prompt' => 'Postgresql database username',
'type' => 'string',
'default' => 'pgsql_pear',
),
),
);

```

此外，还可以替换整个提示。这可能是处理国际化的简单方法。例如：

```php
array(
'id' => 'databaseSetup';
'param' =>
array(
array(
'name' => 'database',
'prompt' => 'Nom de la base de données Postgresql',
'type' => 'string',
'default' => 'pear',
),
array(
'name' => 'user',
'prompt' => 'Nom d'utilisateur de la base de données Postgresql',
'type' => 'string',
'default' => 'pgsql_pear',
),
),
);

```

`run()`方法应接受两种类型的参数。在正常操作中，第一个参数将是一个包含用户答案的数组，第二个参数是 paramgroup 的 ID。对于这个`<tasks:paramgroup>`，示例值可能如下：

```php
array(
'database' => 'huggiepear',
'user' => 'killinator',
);

```

如你所想，ID 将是`'databaseSetup'`。

除了这些旨在成功的功能外，有时在出错时需要中止安装后脚本。在这些情况下，`run()`方法也带有两个参数，但第二个是`'_undoOnError'`，第一个是按逆序排列的已完成的 paramgroup ID 数组，以方便迭代回滚安装后脚本所做的更改。

### 小贴士

**_undoOnError 是错误头而不是另一个 paramgroup ID 吗？**

Paramgroup ID 不能以下划线开头，它只能包含字母数字字符。因此，`_undoOnError`是错误头，而不是另一个 paramgroup ID。

# 将多个包打包成一个单独的存档

通常，将包及其依赖项打包成一个可安装的存档是一个期望的功能。有两种方法可以实现这一点。最简单的方法是使用一个类似于下面的`package.xml`文件：

```php
<?xml version="1.0" encoding="UTF-8"?>
<package version="2.0" 

xsi:schemaLocation="http://pear.php.net/dtd/tasks-1.0
http://pear.php.net/dtd/tasks-1.0.xsd
http://pear.php.net/dtd/package-2.0
http://pear.php.net/dtd/package-2.0.xsd">
<name>PEAR_all</name>
<channel>pear.php.net</channel>
<summary>PEAR Base System</summary>
<description>
The PEAR package and its dependencies
</description>
<lead>
<name>Greg Beaver</name>
<user>cellog</user>
<email>cellog@php.net</email>
<active>yes</active>
</lead>
<date>2005-09-25</date>
<version>
<release>1.4.2</release>
<api>1.0.0</api>
</version>
<stability>
<release>stable</release>
<api>stable</api>
</stability>
<license uri="http://www.php.net/license">PHP License</license>
<notes>
This contains PEAR version 1.4.2 and its dependencies
</notes>
<contents>
<bundledpackage>PEAR-1.4.2.tgz</bundledpackage>
<bundledpackage>Archive_Tar-1.3.1.tgz</bundledpackage>
<bundledpackage>Console_Getopt-1.2.tgz</bundledpackage>
<bundledpackage>XML_RPC-1.4.3.tgz</bundledpackage>
</contents>
<dependencies>
<required>
<php>
<min>4.2</min>
</php>
<pearinstaller>
<min>1.4.0a12</min>
</pearinstaller>
</required>
</dependencies>
<bundle/>
</package>

```

这个简单的`package.xml`可以打包成`PEAR_all-1.4.2.tgz`，并作为单个存档分发，用户可以使用它从非互联网位置升级所有包：

```php
$ pear upgrade PEAR_all-1.4.2.tgz 

```

分发依赖项的另一种方法是旧版 bundle-all-dependencies 方法和 PEAR 分发依赖项方法的巧妙混合。

# 向后兼容性：使用 package.xml 1.0 和 2.0

PEAR 版本 1.4.0 及更高版本最重要的新特性之一是，随着 `package.xml 2.0` 的出现，能够使包适用于较旧的 PEAR 版本。使用以下命令调用的包命令，将 `package.xml` 作为输入，并输出一个 GZIP 压缩的 tar 文件（`.tgz`）。

```php
$ pear package 

```

如果包名为 `Foo`，版本为 `1.0.0`，`.tgz` 文件将被命名为 `Foo-1.0.0.tgz`。从版本 1.4.0 开始，如果有第二个名为 `package2.xml` 的 `package.xml`，包命令将尝试将其包含在存档中。当 PEAR 下载包进行安装时，它首先寻找一个 `package2.xml` 文件，该文件始终以 2.0 格式，然后回退到 `package.xml`。这样，就支持了较旧的 PEAR 版本，因为它们总是首先寻找 `package.xml`。

为了使此功能正常工作，PEAR 对 `package.xml` 文件的内容进行非常严格的比较。`package.xml` 版本 1.0 和 `package.xml` 版本 2.0 必须满足以下约束列表才能被认为是等效的，否则验证将失败：

+   相同的包名称

+   相同的包摘要

+   相同的包描述

+   相同的包版本（发布版本）

+   相同的包稳定性（发布稳定性/状态）

+   相同的许可证

+   相同的发布说明

+   相同的维护者

+   `package.xml 1.0` 中的所有文件都必须存在于 `package.xml 2.0 <contents>`

注意，由于 `package.xml 2.0` 允许使用 `<ignore>` 标签在安装期间忽略文件的存在，因此 `package.xml 2.0` 可以用来提供既可由 PEAR 安装，又可解压后直接使用的存档。

# 为什么支持过时的 `package.xml` 1.0？

这在 PHP 界是一个常见的争议。为什么支持向后兼容性？这些是过时且存在错误的 PEAR 版本，对吧？是的，它们是过时且存在错误的版本，任何使用它们的人都在寻求麻烦，但人们可能找不到一个令人信服的理由去升级他们的 PEAR 安装器，仅仅是为了使用你的包，因为对他们来说“已经足够好”。作为包分发者，你的目标应该是使升级过程尽可能无痛。只有在你实际上在 PHP 代码中使用 PEAR 的新功能，或者你的包是一个没有安装用户基础的全新包时，你才应该停止支持 `package.xml` 版本 1.0。

PEAR 的发展速度非常快，但新安装器的采用不会一夜之间发生。像 Linux 发行版这样的大型软件项目需要时间来评估新功能，并在采用新版本之前确保一切正常工作。作为 PEAR 开发者，我们必须尊重这一需求。

一旦安装的用户不再使用过时且存在错误的 PEAR 版本，升级安装器依赖项应尽快进行，以使用户的利益为重。话虽如此，PEAR 用户应尽快升级以避免在较旧的 PEAR 安装器版本中发现的安全漏洞。

### 小贴士

**PEAR 1.4.3 及更早版本的安全问题**

在撰写本章的前几个月，PEAR 中发现了两个主要的安全漏洞。基本上，如果你正在使用 PEAR 1.4.3 或更早版本，你需要尽快升级。

详细信息可在：[`pear.php.net/advisory-20051104.txt`](http://pear.php.net/advisory-20051104.txt) 和 [`pear.php.net/advisory-20060108.txt`](http://pear.php.net/advisory-20060108.txt) 查找。

# 案例研究：PEAR 包

PEAR 是一个始终需要支持`package.xml 1.0`的包的完美例子。我们总会有一批用户从早期版本升级到最新版本，而 PEAR 1.3.6 及更早版本根本不了解`package.xml 2.0`。如果我们不能使 PEAR 升级成为可能，那么让代码可用就没有太多意义。

然而，与此同时，新的依赖特性以及`package.xml 2.0`的任务对于 PEAR 包来说非常重要，因此需要同时使用`package.xml 1.0`和`package.xml 2.0`。例如，`pear`命令本身在 UNIX 上是一个 shell 脚本（带有 UNIX `\n` 行结束符），在 Windows 上是一个批处理文件（带有 Windows `\r\n` 行结束符）。在`package.xml 2.0`之前，必须将这些脚本作为二进制文件添加到 CVS 中，以确保行结束符不会被打包者的系统行结束符替换。现在，通过使用`<tasks:windowseol/>`和`<tasks:unixeol/>`任务，这不再是必要的，因为正确的行结束符在打包时已经设置好了。此外，由于 PEAR 1.4.0 及更早版本与 PEAR_Frontend_Web 的早期版本以及较老的 PEAR_Frontend_Gtk（已被 PEAR_Frontend_Gtk2 取代）之间的不兼容性，有必要检查这些版本的存在，如果版本正确，则静默成功。`package.xml 2.0`通过在包依赖中使用`<conflicts/>`标签提供了这一功能。

# PEAR_PackageFileManager

虽然在有限的情况下可以使用`convert`和`pickle`命令来管理`package.xml`的 1.0 和 2.0 版本，但这些命令也可能很危险。通过使用 PEAR 包 PEAR_PackageFileManager，从单一位置维护两个版本的`package.xml`是一个更安全的方法。这个包提供了一个简单的接口，可以从中导入现有的`package.xml`文件并更新为当前信息，或者从头开始创建一个新的`package.xml`文件。此外，它非常简单地将现有的`package.xml 2.0`（无论其多么复杂）转换为等效的`package.xml 1.0`，并绝对控制每个`package.xml`的内容。

## 获取 PEAR_PackageFileManager

可以通过使用 PEAR 安装程序轻松获取 PEAR_PackageFileManager。在撰写本书时，版本 1.6.0b1 可用。要安装它，您必须通过以下方式设置`preferred_state`配置变量为`beta`：

```php
$ pear config-set preferred_state beta 

```

或者，您可以运行：

```php
$ pear install PEAR_PackageFileManager-beta 

```

当然，始终确定最新版本在 [`pear.php.net/package/PEAR_PackageFileManager`](http://pear.php.net/package/PEAR_PackageFileManager) 并安装该版本是一个好习惯。

## PEAR_PackageFileManager 脚本及其生成的 package.xml 文件

对于我们的示例 `PEAR_PackageFileManager` 脚本，我们将为 `Chiara_Managedb` 任务生成一个 `package.xml` 文件。在编写 `package.xml` 脚本之前，让我们确保我们理解了我们希望 `package.xml` 包含的组件。在我们的例子中，我们将有三个文件需要打包，包含任务实际代码的 `Managedb.php` 文件，包含可读/可写 `PEAR_Task_Chiara_Managedb_rw` 类的 `rw.php` 文件，用于通过 `PEAR_PackageFile_v2_rw` API 将任务添加到 `package.xml`，以及 `rolesetup.php`，它是初始化 `chiaramdb2schema` 配置变量的安装后脚本。

在深入研究源代码之前，有几个重要的细节需要注意。首先，这个脚本是从头开始生成 `package.xml` 文件的。当使用 `importOptions()` 方法时，大多数脚本不需要这种详细程度。此外，需要注意的是，`PEAR_PackageFileManager2` 类扩展了 PEAR 自身提供的 `PEAR_PackageFile_v2_rw` 类。这允许使用如

使用 `setPackage()` 等方法来调整 `package.xml` 的内容。让我们看看如何生成一个包含安装后脚本的复杂 `package.xml 2.0`。

```php
<?php
/**
* package.xml generation script for Task_Chiara_Managedb package
* @author Gregory Beaver <cellog@php.net>
*/
require_once 'PEAR/PackageFileManager2.php';
PEAR::setErrorHandling(PEAR_ERROR_DIE);
$pfm = &PEAR_PackageFileManager2::importOptions('package.xml',
array(
// set a subdirectory everything is installed into
'baseinstalldir' => 'PEAR/Task/Chiara',
// location of files to package
'packagedirectory' => dirname(__FILE__),
// what method is used to glob files? cvs, svn, perforce
// and file are options
'filelistgenerator' => 'file',
// don't distribute this script
'ignore' => array('package.php', 'package2.xml', 'package.xml'),
// put the post-installation script in a
// different location from the task itself
'installexceptions' =>
array(
'rolesetup.php' => 'Chiara/Task/Managedb',
),
// make the output human-friendly
'simpleoutput' => true,
));
$pfm->setPackage('PEAR_Task_Chiara_Managedb');
$pfm->setChannel('pear.chiaraquartet.net');
$pfm->setLicense('BSD license', 'http://www.opensource.org/licenses/bsd-license.php');
$pfm->setSummary('Provides the <tasks:chiara-managedb/> file task for
managing ' . 'databases on installation');
$pfm->setDescription('Task_Chiara_Managedb provides the code to
implement the <tasks:chiara-managedb/> task, as well as a post- installation script to manage the configuration variables it needs.
This task works in conjunction with the chiaramdb2schema file role
(package PEAR_Installer_Role_Chiaramdb2schema) to create databases
used by a package on installation, and to upgrade the database structure automatically on upgrade. To do this, it uses MDB2_Schema\'s
updateDatabase() functionality.
The post-install script must be run with "pear run-scripts"
to initialize configuration variables');
// initial release version should be 0.1.0
$pfm->addMaintainer('lead', 'cellog', 'Greg Beaver',
'cellog@php.net', 'yes');
$pfm->setAPIVersion('0.1.0');
$pfm->setReleaseVersion('0.1.0');
// our API is reasonably stable, but may need tweaking
$pfm->setAPIStability('beta');
// the code is very new, and may change dramatically
$pfm->setReleaseStability('alpha');
// release notes
$pfm->setNotes('initial release');
// this is a PHP script, not a PECL extension source/binary or a
// bundle package
$pfm->setPackageType('php');
$pfm->addRelease();
// set up special file properties
$pfm->addGlobalReplacement('package-info', '@package_version@',
'version');
$script = &$pfm->initPostinstallScript('rolesetup.php');
// add paramgroups to the post-install script
$script->addParamGroup(
'setup',
$script->getParam('channel', 'Choose a channel to modify
configuration values from',
'string', 'pear.php.net'));
$script->addParamGroup(
'driver',
$script->getParam('driver', 'Database driver?'),
'In order to set up the database, please choose a database
driver. This should be a MDB2-compatible driver name, such as mysql, mysqli, Pgsql, oci8, etc.');
$script->addParamGroup(
'choosedsn',
$script->getParam('dsnchoice', '%sChoose a DSN to modify, or to add a' . ' new dsn, type "new". To remove a DSN prepend with "!"'));
$script->addParamGroup(
'deletedsn',
$script->getParam('confirm', 'Really delete "%s" DSN? (yes to delete)', 'string', 'no'));
$script->addConditionTypeGroup(
'modifydsn',
'choosedsn', 'dsnchoice', 'new', '!=',
array(
$script->getParam('user', 'User name', 'string', 'root'),
$script->getParam('password', 'Database password',
'password'),
$script->getParam('host', 'Database host', 'string',
'localhost'),
$script->getParam('database', 'Database name'),
));
$script->addParamGroup(
'newpackagedsn',
array(
$script->getParam('package', 'Package name'),
$script->getParam('user', 'User name', 'string', 'root'),
$script->getParam('password', 'Database password',
'password'),
$script->getParam('host', 'Database host', 'string',
'localhost'),
$script->getParam('database', 'Database name'),
));
$script->addParamGroup(
'newdefaultdsn',
array(
$script->getParam('user', 'User name', 'string', 'root'),
$script->getParam('password', 'Database password',
'password'),
$script->getParam('host', 'Database host', 'string',
'localhost'),
$script->getParam('database', 'Database name'),
));
$pfm->addPostinstallTask($script, 'rolesetup.php');
// start over with dependencies
$pfm->clearDeps();
$pfm->setPhpDep('4.2.0');
// we use post-install script features fixed in PEAR 1.4.3
$pfm->setPearinstallerDep('1.4.3');
$pfm->addPackageDepWithChannel('required', 'PEAR', 'pear.php.net',
'1.4.3');
$pfm->addPackageDepWithChannel('required', 'MDB2_Schema',
'pear.php.net', '0.3.0');
$pfm->addPackageDepWithChannel('required',
'PEAR_Installer_Role_Chiaramdb2schmea',
'pear.chiaraquartet.net', '0.1.0');
// create the <contents> tag
$pfm->generateContents();
// create package.xml 1.0 to gracefully tell PEAR 1.3.x users they have
// to upgrade to use this package
$pfm1 = $pfm->exportCompatiblePackageFile1(array(
// set a subdirectory everything is installed into
'baseinstalldir' => 'PEAR/Task/Chiara',
// location of files to package
'packagedirectory' => dirname(__FILE__),
// what method is used to glob files? cvs, svn, perforce
// and file are options
'filelistgenerator' => 'file',
// don't distribute this script
'ignore' => array('package.php', 'package.xml', 'package2.xml', 'rolesetup.php'),
// put the post-installation script in a
// different location from the task itself
// make the output human-friendly
'simpleoutput' => true,
));
// display the package.xml by default to allow "debugging" by eye,
// and then create it if explicitly asked to
if (isset($_GET['make']) || (isset($_SERVER['argv']) &&
@$_SERVER['argv'][1] == 'make')) {
$pfm1->writePackageFile();
$pfm->writePackageFile();
} else {
$pfm1->debugPackageFile();
$pfm->debugPackageFile();
}
?>

```

它生成的 `package.xml` 如下：

```php
<?xml version="1.0" encoding="UTF-8"?>
<package packagerversion="1.4.3" version="2.0"

xsi:schemaLocation="http://pear.php.net/dtd/tasks-1.0
http://pear.php.net/dtd/tasks-1.0.xsd
http://pear.php.net/dtd/package-2.0
http://pear.php.net/dtd/package-2.0.xsd">
<name>PEAR_Task_Chiara_Managedb</name>
<channel>pear.chiaraquartet.net</channel>
<summary>Provides the &lt;tasks:chiara-managedb/&gt; file task for
managing databases on installation</summary>
<description>Task_Chiara_Managedb provides the code to implement the
&lt;tasks:chiara-managedb/&gt; task, as well as a post-installation
script to manage the configuration variables it needs.
This task works in conjunction with the chiaramdb2schema file role
(package PEAR_Installer_Role_Chiaramdb2schema) to create databases
used by a package on installation, and to upgrade the database structure automatically on upgrade. To do this, it uses MDB2_Schema&apos;s
updateDatabase() functionality.
The post-install script must be run with &quot;pear run-scripts&quot;
to initialize configuration variables</description>
<lead>
<name>Greg Beaver</name>
<user>cellog</user>
<email>cellog@php.net</email>
<active>yes</active>
</lead>
<date>2005-10-18</date>
<time>23:55:29</time>
<version>
<release>0.1.0</release>
<api>0.1.0</api>
</version>
<stability>
<release>alpha</release>
<api>beta</api>
</stability>
<license uri="http://www.opensource.org/licenses/bsd-
license.php">BSD license</license>
<notes>initial release</notes>
<contents>
<dir baseinstalldir="PEAR/Task/Chiara" name="/">
<dir name="Managedb">
<file name="rw.php" role="php">
<tasks:replace from="@package_version@" to="version"
type="package-info" />
</file>
</dir> <!-- //Managedb -->
<file name="Managedb.php" role="php">
<tasks:replace from="@package_version@" to="version"
type="package-info" />
</file>
<file name="rolesetup.php" role="php">
<tasks:postinstallscript>
<tasks:paramgroup>
<tasks:id>setup</tasks:id>
<tasks:param>
<tasks:name>channel</tasks:name>
<tasks:prompt>Choose a channel to modify configuration values
from</tasks:prompt>
<tasks:type>string</tasks:type>
<tasks:default>pear.php.net</tasks:default>
</tasks:param>
</tasks:paramgroup>
<tasks:paramgroup>
<tasks:id>driver</tasks:id>
<tasks:instructions>In order to set up the database, please choose a database driver.
This should be a MDB2-compatible driver name, such as mysql, mysqli, Pgsql, oci8, etc. </tasks:instructions>
<tasks:param>
<tasks:name>driver</tasks:name>
<tasks:prompt>Database driver?</tasks:prompt>
<tasks:type>string</tasks:type>
</tasks:param>
</tasks:paramgroup>
<tasks:paramgroup>
<tasks:id>choosedsn</tasks:id>
<tasks:param>
<tasks:name>dsnchoice</tasks:name>
<tasks:prompt>%sChoose a DSN to modify, or to add a new dsn, type &quot;new&quot;. To remove a DSN prepend with &quot;!&quot; </tasks:prompt>
<tasks:type>string</tasks:type>
</tasks:param>
</tasks:paramgroup>
<tasks:paramgroup>
<tasks:id>deletedsn</tasks:id>
<tasks:param>
<tasks:name>confirm</tasks:name>
<tasks:prompt>Really delete &quot;%s&quot; DSN? (yes to delete)</tasks:prompt>
<tasks:type>string</tasks:type>
<tasks:default>no</tasks:default>
</tasks:param>
</tasks:paramgroup>
<tasks:paramgroup>
<tasks:id>modifydsn</tasks:id>
<tasks:name>choosedsn::dsnchoice</tasks:name>
<tasks:conditiontype>!=</tasks:conditiontype>
<tasks:value>new</tasks:value>
<tasks:param>
<tasks:name>user</tasks:name>
<tasks:prompt>User name</tasks:prompt>
<tasks:type>string</tasks:type>
<tasks:default>root</tasks:default>
</tasks:param>
<tasks:param>
<tasks:name>password</tasks:name>
<tasks:prompt>Database password</tasks:prompt>
<tasks:type>password</tasks:type>
</tasks:param>
<tasks:param>
<tasks:name>host</tasks:name>
<tasks:prompt>Database host</tasks:prompt>
<tasks:type>string</tasks:type>
<tasks:default>localhost</tasks:default>
</tasks:param>
<tasks:param>
<tasks:name>database</tasks:name>
<tasks:prompt>Database name</tasks:prompt>
<tasks:type>string</tasks:type>
</tasks:param>
</tasks:paramgroup>
<tasks:paramgroup>
<tasks:id>newpackagedsn</tasks:id>
<tasks:param>
<tasks:name>package</tasks:name>
<tasks:prompt>Package name</tasks:prompt>
<tasks:type>string</tasks:type>
</tasks:param>
<tasks:param>
<tasks:name>user</tasks:name>
<tasks:prompt>User name</tasks:prompt>
<tasks:type>string</tasks:type>
<tasks:default>root</tasks:default>
</tasks:param>
<tasks:param>
<tasks:name>password</tasks:name>
<tasks:prompt>Database password</tasks:prompt>
<tasks:type>password</tasks:type>
</tasks:param>
<tasks:param>
<tasks:name>host</tasks:name>
<tasks:prompt>Database host</tasks:prompt>
<tasks:type>string</tasks:type>
<tasks:default>localhost</tasks:default>
</tasks:param>
<tasks:param>
<tasks:name>database</tasks:name>
<tasks:prompt>Database name</tasks:prompt>
<tasks:type>string</tasks:type>
</tasks:param>
</tasks:paramgroup>
<tasks:paramgroup>
<tasks:id>newdefaultdsn</tasks:id>
<tasks:param>
<tasks:name>user</tasks:name>
<tasks:prompt>User name</tasks:prompt>
<tasks:type>string</tasks:type>
<tasks:default>root</tasks:default>
</tasks:param>
<tasks:param>
<tasks:name>password</tasks:name>
<tasks:prompt>Database password</tasks:prompt>
<tasks:type>password</tasks:type>
</tasks:param>
<tasks:param>
<tasks:name>host</tasks:name>
<tasks:prompt>Database host</tasks:prompt>
<tasks:type>string</tasks:type>
<tasks:default>localhost</tasks:default>
</tasks:param>
<tasks:param>
<tasks:name>database</tasks:name>
<tasks:prompt>Database name</tasks:prompt>
<tasks:type>string</tasks:type>
</tasks:param>
</tasks:paramgroup>
</tasks:postinstallscript>
<tasks:replace from="@package_version@" to="version"
type="package-info" />
</file>
</dir> <!-- / -->
</contents>
<dependencies>
<required>
<php>
<min>4.2.0</min>
</php>
<pearinstaller>
<min>1.4.3</min>
</pearinstaller>
<package>
<name>PEAR</name>
<channel>pear.php.net</channel>
<min>1.4.3</min>
</package>
<package>
<name>MDB2_Schema</name>
<channel>pear.php.net</channel>
<min>0.3.0</min>
</package>
<package>
<name>PEAR_Installer_Role_Chiaramdb2schema</name>
<channel>pear.chiaraquartet.net</channel>
<min>0.1.0</min>
</package>
</required>
</dependencies>
<phprelease />
<changelog>
<release>
<version>
<release>0.1.0</release>
<api>0.1.0</api>
</version>
<stability>
<release>alpha</release>
<api>beta</api>
</stability>
<date>2005-10-18</date>
<license>BSD license</license>
<notes>initial release</notes>
</release>
</changelog>
</package>

```

## PEAR_PackageFileManager 如何让艰难的生活变得容易

聪明的读者可能已经注意到，`package.xml` 生成脚本相当广泛且长。好消息是，在许多情况下，这将是多余的。事实上，`package.xml` 的初始生成通常不是 PEAR_PackageFileManager 完成的最重要的功能。更重要的要数是发布数据的维护。维护 `package.xml` 文件，在许多情况下 `package.xml` 和 `package2.xml` 是一个严重的问题。尽管 PEAR 安装程序通过在包文件之间进行仔细的等价性比较来使其变得容易一些，但这个过程并不完美。

PEAR_PackageFileManager 使用相同的数据，并使用显式逻辑来生成 `package.xml` 的元数据，从而保证创建的 `package.xml` 文件是等效的。此外，数据的集中化意味着您只需在更新发布说明时修改脚本。此外，由于 PEAR 内置的 `package.xml` 验证用于验证生成的 `package.xml` 文件——与打包和安装时使用的相同验证，因此不可能生成无效的 `package.xml`。

### 为 package.xml 查找文件

PEAR_PackageFileManager 执行的最重要功能之一是创建文件列表。我们的简单包中只有几个文件，但对于像 PhpDocumentor（[`pear.php.net/PhpDocumentor`](http://pear.php.net/PhpDocumentor)）这样的大型、复杂包来说，管理 `package.xml` 就变成了一项越来越困难的任务。PhpDocumentor 不仅包含了几百个文件，而且由于使用了 Smarty 模板，它们在版本之间也往往会发生巨大的变化。

通过关闭 `simpleoutput` 选项，可以轻松检测到已修改的文件，并从版本到版本进行监控，而无需依赖于外部工具。

### 小贴士

**为什么会出现 PEAR_PackageFileManager？**

最初，PEAR_PackageFileManager 是一个用于生成 PhpDocumentor 的 `package.xml` 的单一脚本。随着时间的推移，随着对该脚本的请求越来越多，它得到了改进，最终变得明确，它应该成为一个独立的项目。

最初，PEAR_PackageFileManager 简单地使用 shell 通配符的 `'ignore'` 选项来排除文件，遍历当前文件列表中的所有文件。例如，我们可以使用这个通配符 `"*test*"` 来忽略所有名称中包含 "test" 的文件。此外，通过在模式后附加一个 "/"，可以忽略整个目录及其所有内容，包括子目录，例如 `"CVS/"`。

上述示例突出了这种方法的其中一个问题：在一个基于 CVS 的包中，可能存在不属于项目的文件在包内部，并且不会随着 `cvs export` 命令一起导出。因此，PEAR_PackageFileManager 有几个文件遍历驱动程序，或文件列表生成器。选择使用哪个文件列表生成器驱动程序由 `setOptions()/importOptions()` 方法族中的 `'filelistgenerator'` 选项控制。最简单的是 `file` 生成器。

其他驱动程序有 `'cvs'`, `'svn'`, 和 `'perforce'`。这些驱动程序与 `'file'` 驱动程序相同，只是它们不是简单地遍历目录及其所有子目录中的每个文件，而是将文件列表限制为远程版本控制源代码库的本地签出中的文件。并发版本控制系统（CVS）、Subversion 和 Perforce 都是版本控制源代码库系统。如果你不知道它们是什么，调查 Subversion 和 CVS 会是一个好主意，因为它们都是免费的开源解决方案。Subversion 比 CVS 功能更全面，而且更新，而 CVS 是一个经过考验的可靠系统。

### 管理变更日志

PEAR_PackageFileManager 默认会自动为当前发布版本生成一个变更日志，从旧到新。有一些选项可以控制这一点。首先，如果`'changelogoldtonew'`选项设置为 false，将重新排序变更日志，使较新的条目更靠近文件顶部。此外，如果变更日志要使用不同于发布说明的笔记集，请使用`'changelognotes'`选项来控制这一点。

### 同步 package.xml 版本 1.0 和 package.xml 版本 2.0

在某些情况下，可能需要生成等效的`package.xml`版本 1.0。例如，我们可能希望允许 PEAR 1.3.x 用户在出现“requires PEAR 1.4.3 or newer”错误消息时优雅地失败。使用 PEAR_PackageFileManager 来做这件事非常简单。将`importOptions`行从：

```php
$pfm = &PEAR_PackageFileManager2::importOptions('package.xml',

```

更改为：

```php
$pfm = &PEAR_PackageFileManager2::importOptions('package2.xml',

```

然后，将脚本的最后几行更改为：

```php
// create the <contents> tag
$pfm->generateContents();
// create package.xml 1.0 to gracefully tell PEAR 1.3.x users they
// have to upgrade to use this package
$pfm1 = $pfm->exportCompatiblePackageFile1(array(
// set a subdirectory everything is installed into
'baseinstalldir' => 'PEAR/Task/Chiara',
// location of files to package
'packagedirectory' => dirname(__FILE__),
// what method is used to glob files? cvs, svn, perforce
// and file are options
'filelistgenerator' => 'file',
// don't distribute this script
'ignore' => array('package.php', 'package.xml',
'package2.xml', 'rolesetup.php'),
// put the post-installation script in a
// different location from the task itself
// make the output human-friendly
'simpleoutput' => true,
));
// display the package.xml by default to allow "debugging" by eye,
// and then create it if explicitly asked to
if (isset($_GET['make']) || (isset($_SERVER['argv']) &&
@$_SERVER['argv'][1] == 'make')) {
$pfm1->writePackageFile();
$pfm->writePackageFile();
} else {
$pfm1->debugPackageFile();
$pfm->debugPackageFile();
}
?>

```

然后，脚本将输出`package.xml`和`package2.xml`。

# 使用 PEAR Installer 安装包

处理过程的最后一步是创建一个包。一旦生成了一个`package.xml`文件，就可以使用它来创建包含包内容的文件。为此，应使用`package`命令：

```php
$ pear package 

```

此命令应在包含`package.xml`文件的目录中执行。这将创建一个`.tgz`文件，例如`Package-version.tgz`，其中`Package`是包名，`version`是发布版本。如果你的包名为`Foo`且版本为`1.2.3`，则包命令将创建一个名为`Foo-1.2.3.tgz`的文件。此文件可以安装为：

```php
$ pear install Foo-1.2.3.tgz 

```

或者也可以上传到频道服务器进行公开发布（见第五章）。

包命令还可以使用`--uncompress`或`-Z`选项创建一个未压缩的`.tar`文件：

```php
$ pear package Z 

```

在某些情况下，你可能已经重命名了包文件。在这种情况下，有必要明确指定用于打包的`package.xml`，如下所示：

```php
$ pear package package-PEAR.xml package2.xml 

```

这是创建用于发布的 PEAR 包的实际命令行。请注意，传入哪个`package.xml`（版本 1.0 或版本 2.0）无关紧要，以下命令行序列是相同的。

```php
$ pear package package2.xml package-PEAR.xml 

```

然而，如果两个`package.xml`版本相同，打包将失败。此外，两个`package.xml`文件之间会进行严格的比较。如果`<description>`标签、`<summary>`标签或`<notes>`标签的文本之间有任何细微的差异，验证将失败。实际上，`package.xml 1.0`中的每个文件都必须包含在`package.xml 2.0`中。维护者的数量和他们的角色必须完全相同。

然而，允许存在一些差异。例如，`package.xml 1.0` 的依赖关系无需与 `package.xml 2.0` 的依赖关系相匹配，这是因为 `package.xml 2.0` 简单地代表了一个比 `package.xml 1.0` 更广泛的可能依赖集。此外，`package.xml 2.0` 中引入的 `<ignore>` 标签使得能够分发被 PEAR 安装程序忽略的文件。通过这种方式，一个即插即用的应用程序也可以通过分发用于即插即用运行的文件，并要求 PEAR 安装程序忽略它们，而轻松地使用 PEAR 进行安装。这些文件将不会出现在 `package.xml 1.0` 中，因为 PEAR 1.3.x 没有这个功能。

# 摘要

在这个阶段，我们已经深入探讨了 PEAR 安装程序和 `package.xml` 的内部工作原理——可以肯定地说，你现在已经成为了一个 `package.xml` 专家。
