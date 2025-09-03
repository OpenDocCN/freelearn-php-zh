# 第一章. CodeIgniter 基础知识

在本章中，我们将涵盖：

+   下载和安装 CodeIgniter

+   基本配置选项

+   在不同环境中管理 CodeIgniter

+   在不同环境中管理数据库设置

+   保护系统文件

+   使用 .htaccess 移除地址栏中的 index.php

+   安装和使用 Sparks

# 简介

CodeIgniter 是一个易于使用、易于设置的基于 PHP 的框架，您可以使用它构建您能想到的几乎所有基于 Web 的应用程序。在我们开始使用 CodeIgniter 之前，需要做一些配置；然而，本章将指导您下载、安装和理解 CodeIgniter 的基本配置，以帮助您快速启动并运行。

# 下载和安装 CodeIgniter

首先，您需要一份 CodeIgniter 的副本才能开始使用。有几个选择：您可以下载夜间构建版本、旧版本或当前稳定版本。然而，建议您选择最新的稳定版本。

## 如何操作...

您可以通过以下链接获取 CodeIgniter 的最新稳定版本：

[`codeigniter.com/downloads/`](http://codeigniter.com/downloads/)

CodeIgniter 将以压缩存档文件的形式提供。一旦下载了 CodeIgniter，将其包复制到您的 Web 文件夹中，并像通常在系统上解压缩存档一样解压缩它。完成此操作后，您需要设置一些配置选项，我们将在下一节中讨论。

# 基本配置选项

配置 CodeIgniter 比许多其他可用的 Web 框架要容易得多，并且不需要您求助于使用命令行。您只需要快速启动并运行，就需要访问`application/config/`文件夹中的几个文件。以下是一些建议的设置，这些设置将使您的 CodeIgniter 安装准备就绪而无需太多麻烦。

## 如何操作...

在您的本地主机或开发环境中的`/path/to/codeigniter/application/config/config.php`文件中打开文件，并找到以下行：

```php
$config["base_url"]:
```

该值应该是到 CodeIgniter 安装的完整网络地址（在浏览器地址栏中写入的地址）。因此，如果您在本地主机上工作，该值应该是：`http://localhost/path/to/codeigniter/`。

### 小贴士

请记住以`http://`开始，并且始终放置尾随的斜杠`/`。

如果您已修改主机文件以使用域名而不是 localhost，那么请确保将 localhost 替换为该域名：

```php
$config["encryption_key"]
```

如果您想在您的应用程序中使用`Session`或`Encryption`类，则必须设置加密密钥。加密密钥是 CodeIgniter 用于加密各种通信的字符字符串：

```php
$config["global_xss_filtering"]
```

上一行代码指定是否应将跨站脚本过滤应用于`Get`、`Post`或`Cookie`变量。出于安全考虑，这应设置为`TRUE`，尤其是在生产环境中：

```php
$config["csrf_protection"]
```

上一行代码指定是否设置 cookie 令牌，如果设置为 `TRUE`，则每次从客户端提交表单时都会进行检查。在实时环境中，它应该设置为 `TRUE`：

```php
$config["log_threshold"]
```

上一行代码指定是否要将信息写入日志，如果是，则写入日志的信息类型。例如：

+   `0`: 不将错误写入日志，因为日志已禁用

+   `1`: 仅错误消息（这将包括 PHP 错误）

+   `2`: 仅调试消息

+   `3`: 仅信息性消息

+   `4`: 所有类型的消息

以下代码行是您希望保存日志文件的文件夹路径：

```php
$config["log_path"] = "/path/to/log/file"
```

### 它是如何工作的...

CodeIgniter 现在将根据提供的设置做出响应并运行。

## 在不同环境中管理 CodeIgniter

在某些情况下，可能需要调整您的配置文件，以便它们可以在多个服务器或环境中运行，而无需每次移动时都编辑或维护。例如，您在本地主机上可能有的配置设置与在实时或生产服务器上的设置很可能不同。正确设置配置文件将为您节省大量时间，而不是手动在两者之间切换。

### 如何操作...

打开 `/path/to/codeigniter/application/config/config.php` 文件，并将 `$config["base_url"]` 行替换为以下内容：

```php
switch($_SERVER["SERVER_NAME"]) {
    case "localhost":
        $config["base_url"] = "http://localhost/path/to/codeigniter/";
        break;
    case "mydomain.com":
        $config["base_url"] = "http://www.mydomain.com/";
        break;
}
```

### 它是如何工作的...

这是一个简单的 `case/switch` 语句，带有 `SERVER_NAME` 检查。`base_url` 值根据 CodeIgniter 应用程序或项目运行的服务器设置。

## 在不同环境中管理数据库设置

如果您计划为您的 CodeIgniter 应用程序使用数据库，那么您需要维护正确的连接设置。CodeIgniter 将这些设置保存在 `database.php` 配置文件中。

### 如何操作...

1.  打开 `/path/to/codeigniter/application/config/database.php` 文件。可能需要更改的唯一值是数据库服务器的标准主机名、用户名、密码和数据库名称。

1.  找到定义 `$active_group` 的行，它指定了特定托管环境要使用的特定数据库设置。您可以通过类似于之前使用的 `case`/`switch` 测试来切换设置，例如，以下代码测试特定服务器并加载适当的设置：

    ```php
    switch($_SERVER["SERVER_NAME"]) {
        case "localhost":
            $active_group = "testing";
            break;
        case "mydomain.com":
            $active_group = "default"
        break;
    }

    $db["default"]["hostname"] = "localhost";
    $db["default"]["username"] = "root";
    $db["default"]["password"] = "";
    $db["default"]["database"] = "database_name";
    $db["default"]["dbdriver"] = "mysql";
    $db["default"]["dbprefix"] = "";
    $db["default"]["pconnect"] = TRUE;
    $db["default"]["db_debug"] = FALSE;
    $db["default"]["cache_on"] = FALSE;
    $db["default"]["cachedir"] = "";
    $db["default"]["char_set"] = "utf8";
    $db["default"]["dbcollat"] = "utf8_general_ci";
    $db["default"]["swap_pre"] = "";
    $db["default"]["autoinit"] = TRUE;
    $db["default"]["stricton"] = FALSE;

    $db["testing"]["hostname"] = "localhost";
    $db["testing"]["username"] = "root";
    $db["testing"]["password"] = "";
    $db["testing"]["database"] = "database_name";
    $db["testing"]["dbdriver"] = "mysql";
    $db["testing"]["dbprefix"] = "";
    $db["testing"]["pconnect"] = TRUE;
    $db["testing"]["db_debug"] = TRUE;
    $db["testing"]["cache_on"] = FALSE;
    $db["testing"]["cachedir"] = "";
    $db["testing"]["char_set"] = "utf8";
    $db["testing"]["dbcollat"] = "utf8_general_ci";
    $db["testing"]["swap_pre"] = "";
    $db["testing"]["autoinit"] = TRUE;
    $db["testing"]["stricton"] = FALSE;

    $active_record – Specifies if you require active record support.  By default it is set to TRUE.
    ```

### 它是如何工作的...

我们所做的一切只是定义网站运行的环境。在上面的示例中，我们指定了两个环境：`default` 或 `testing`，并为它们应用了特定的设置。因此，让我们看看一些变量定义。

#### 常见值

以下表格显示了标准的数据库访问选项：

| 选项名称 | 有效选项 | 描述 |
| --- | --- | --- |
| $db["default"]["hostname"] | 通常为 localhost | 这是数据库所在的服务器 |
| $db["default"]["username"] |   | 数据库访问用户名 |
| $db["default"]["password"] |   | 数据库的密码 |
| $db["default"]["database"] |   | 数据库的名称 |

#### 其他值

以下表格显示了通常保持默认设置不变但在此处提供以便您更改的选项：

| 选项名称 | 有效选项 | 描述 |
| --- | --- | --- |
| $db["default"]["dbdriver"] | mysql | 这是您使用的 DBMS 类型——本书中的配方使用 MySQL。值必须全部小写。 |
| $db["default"]["dbprefix"] | 默认：mysql，但也可能是 postgre、odbc 等 | 有时您可能希望向数据库表名添加前缀，例如，一个博客应用可能会将表名前缀为“blog”，这样帖子表就会变成 blog_posts。 |
| $db["default"]["pconnect"] | TRUE/FALSE | 指定您是否希望保持与数据库的持久连接。 |
| $db["default"]["db_debug"] | TRUE/FALSE | 指定您是否希望在屏幕上显示数据库错误。默认为空白，但出于安全考虑，在实际环境中应设置为`FALSE`，在开发时应设置为`TRUE`。 |
| $db["default"]["cache_on"] | TRUE/FALSE | 指定您是否希望启用数据库查询缓存。 |
| $db["default"]["cachedir"] |   | 指定数据库查询缓存的绝对文件路径。 |
| $db["default"]["char_set"] | utf8 | 指定 CodeIgniter 将用于数据库的字符集。 |
| $db["default"]["dbcollat"] | utf8_general_ci | 指定 CodeIgniter 将用于数据库的字符排序。 |
| $db["default"]["port"] | 3306 | 默认 MySQL 端口。此选项默认不包含，如果您希望为数据库连接使用特定端口，您需要手动写入此行并设置值。 |

我们将在第六章*与数据库一起工作*中更详细地探讨从数据库访问数据。

## 保护系统文件

在实际环境中，强烈建议您将系统文件夹移出网站根目录，以防止恶意访问。

### 如何操作...

1.  通过命令行或使用计算机的 GUI 将系统文件夹移动到公开可访问的网站文件夹之外。执行此操作的方法将取决于您使用的系统，但我相信您知道如何移动文件夹，所以我们在这里不讨论这一点。

1.  在您移动系统文件夹后，您需要更新`path/to/codeigniter/index.php`文件中的`$system_path`变量。查找并找到以下行：

    ```php
    $system_path = "path/to/system/folder";
    ```

    修改该行以反映系统文件夹的新位置。例如，如果您将系统文件夹从网站根目录向上移动一个级别，您应该写下以下行：

    ```php
    $system_path = "../system";
    ```

### 它是如何工作的...

通过将系统文件夹移出 Web 根目录，你可以尽可能保护它免受通过互联网的访问。系统文件夹在 Web 根目录外被访问的可能性比在内部要小得多。

## 使用`.htaccess`从地址栏中删除 index.php

当 CodeIgniter 运行时，有可能从网络浏览器地址栏中删除`index.php`文件。

### 如何操作...

创建或打开一个`.htaccess`文件。如果还没有`.htaccess`文件，你可以按照以下步骤创建一个：

**Linux/Mac**

1.  打开一个终端窗口并输入：`touch/path/to/CodeIgniter/.htaccess`。

**Windows**

1.  在你的 CodeIgniter 根目录中创建一个文本文件，命名为`file.htaccess`。

1.  按*Windows* + *R*打开运行对话框。

1.  输入以下命令并点击**确定**：

    ```php
    ren "C:\path\to\CodeIgniter\file.htaccess" .htaccess
    ```

1.  一旦你的`.htaccess`文件被打开，在文件顶部写下以下行：

    ```php
    <IfModule mod_rewrite.c>
    RewriteEngine on
    RewriteCond $1 !^(index\.php|images|robots\.txt)
    RewriteRule ^(.*)$ index.php/$1 [L]
    </IfModule>
    ```

### 它是如何工作的...

`.htaccess`文件中的此规则将从浏览器的地址栏中删除`index.php`文件。CodeIgniter 的`index.php`文件仍然被调用，但它不会在浏览器的地址栏中显示给用户。

## 安装和使用 Sparks

很长一段时间以来，为了找到和使用 CodeIgniter 的扩展、库和其他有用的代码片段，你必须在网上搜索，并从博客、代码仓库等各种地方下载代码。有用的 CodeIgniter 安装散布在互联网上，因此可能很难找到。Sparks 作为 CodeIgniter 扩展的单一点参考。它简单易安装和使用，包含数千个有用的 CodeIgniter 附加组件。

### 如何操作...

如果你正在使用 MAC 或 Linux，那么命令行界面对你来说是开放的。

1.  使用系统上的终端应用程序，导航到你的 CodeIgniter 应用程序的根目录，并输入以下行：

    ```php
    php -r "$(curl -fsSL http://getsparks.org/go-sparks)"
    ```

    如果你的安装成功，你应该会看到类似以下的内容：

    ```php
    user@server:/path/to/codeigniter$ php -r "$(curl -fsSL http://getsparks.org/go-sparks)"
    Pulling down spark manager from http://getsparks.org/static/install/spark-manager-0.0.9.zip ...
    Pulling down Loader class core extension from http://getsparks.org/static/install/MY_Loader.php.txt ...
    Extracting zip package ...
    Cleaning up ...
    Spark Manager has been installed successfully!
    Try: `php tools/spark help`
    ```

    如果你正在使用 Windows，那么你需要手动下载 Sparks 并解压。为此，执行以下说明或查看 GetSparks 网站上的最新版本说明：

1.  在你的 CodeIgniter 目录的最高层（根目录）中创建一个名为`tools`的文件夹。

1.  访问以下 URL：[`getsparks.org/install`](http://getsparks.org/install)。

1.  前往**正常安装**部分并下载 Sparks 包。

1.  将下载解压到你在步骤 1 中创建的`tools`文件夹中。

1.  从[`getsparks.org/static/install/MY_Loader.php.txt`](http://getsparks.org/static/install/MY_Loader.php.txt)下载`Loader`类扩展。

1.  将`MY_Loader.php.txt`文件重命名为`MY_Loader.php`，并将其移动到你的 CodeIgniter 实例中的`application/core/MY_Loader.php`目录。

1.  现在 Sparks 已经安装在你的 CodeIgniter 实例中，你可以开始安装扩展和包了。

    要从 Sparks 安装一个包，请在命令行中输入以下内容：

    ```php
    php tools/spark install [Package Version] Spark Name
    ```

    在这里，`Package Version`是你希望安装的 Spark 的具体版本。你不需要声明版本，如果不指定，Spark 将默认下载最新版本。`Spark Name`是你希望安装的 Spark 的名称，例如，要安装默认安装中包含的`example-spark`（版本 1.0.0），请在命令行中输入以下命令：

    ```php
    php tools/spark install -v1.0.0 example-spark
    ```

    如果安装成功，你应该会看到类似以下内容：

    ```php
    user@server:/path/to/codeigniter$ php tools/spark install -v1.0.0 example-spark 
    [ SPARK ] Retrieving spark detail from getsparks.org
    [ SPARK ] From Downtown! Retrieving spark from Mercurial repository at https://url/of/the/spark/repo
    [ SPARK ] Spark installed to ./sparks/example-spark/1.0.0 - You're on fire!
    ```

### 它是如何工作的...

你现在应该准备好开始使用你的 Spark 了。请确保阅读随 Spark 一起提供的`Readme`文件或文档，以了解其正确使用方法。
