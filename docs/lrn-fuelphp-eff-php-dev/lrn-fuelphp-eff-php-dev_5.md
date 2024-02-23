# 第五章：包

在上一章中，我们使用包来使我们的生活更轻松。我们使用了 Oil 来快速启动和运行，以及帮助用户认证的 Auth 包。

在本章中，我们将涵盖以下主题：

+   什么是包？

+   推荐的包

+   使用 Auth 包进行用户认证

+   什么是 Composer 以及如何使用它？

+   构建自己的包的介绍

# 什么是包？

作为开发人员，我们经常有一些我们在多个项目中使用的代码。它可能只是简单的字符串操作，但如果没有我们熟悉的代码，我们会感到迷失。

这就是包派上用场的地方。它们提供了一个很好的组织、重用和共享代码的方式。包可以包含各种代码，比如模型、配置，甚至第三方库。

由于 FuelPHP 和其他包的类结构，可以扩展其他包和 FuelPHP 核心。这一切都可以在不更改核心文件的情况下实现，使升级更加容易和简单。

尽管包可以做很多事情，但也有一些包做不到的事情。例如，它们无法映射到 URL；这是应用程序或项目代码的作用。如果有一个功能在多个项目中需要重复使用，并且还需要 URL 访问，建议使用模块。这将在下一章中介绍。

运行一些推荐的包是展示使用包时可能发生的事情的好方法。

# 推荐的包

尽管您的需求可能不同，在本节中我们将介绍一些经常有用的包。

## OAuth

如今的互联网充满了像 Facebook 和 Twitter 这样的大玩家，每个玩家都有不同的结构和用途；但它们都支持用户认证的**OAuth**的一个版本。这意味着许多项目可能需要使用 OAuth 进行用户认证和单一登录。

一个高度推荐的包是从 Kohana PHP 框架（[`kohanaframework.org`](http://kohanaframework.org)）移植而来的。它可以在[`github.com/fuel-packages/fuel-oauth`](https://github.com/fuel-packages/fuel-oauth)找到。

这个包处理与第三方服务的认证，比如：

+   Dropbox

+   Flickr

+   领英

+   Tumblr

+   推特

+   Vimeo

在使用这个包时，您需要将配置文件复制到应用程序配置目录。这将允许您为第三方系统添加您的消费者密钥和密钥。

## OAuth2

您可能已经注意到，Facebook 在 OAuth 包的可用交互中缺失了。这主要是因为 Facebook 使用**OAuth2**标准（[`oauth.net/2`](http://oauth.net/2)）。尽管它与第一个包执行的工作类似，但您可能会发现 OAuth2 包有用。这个包可以在[`github.com/fuel-packages/fuel-oauth2`](https://github.com/fuel-packages/fuel-oauth2)找到，并支持以下第三方服务：

+   脸书

+   Foursquare

+   GitHub

+   谷歌

+   PayPal

+   Instagram

+   Soundcloud

+   Windows Live

+   YouTube

与之前的包一样，您需要创建控制器和应用程序代码，以利用来自 OAuth 的认证数据。

## Mandrill

有时，我们需要发送电子邮件。尽管使用默认的`mail()` PHP 函数肯定更容易，但这种方法并不总是可靠的。有时，`mail()`函数会悄悄失败。这就是第三方系统派上用场的地方。有几个大型的电子邮件服务，比如**Campaign Monitor**和**MailChimp**。这些通常是电子邮件活动和邮件列表。幸运的是，MailChimp 已经以**Mandrill**（[`mandrill.com`](http://mandrill.com)）的形式开放了他们的基础设施。

Fuel Mandrill 包是一个非官方包，可以从[`github.com/Izikd/fuel-mandrill`](https://github.com/Izikd/fuel-mandrill)获取。它是官方 Mandrill API 库的包装器，因此您可以放心它会正常工作。

## Sprockets

前端开发和 HTML 正开始变得更加结构化，使用诸如 LESS、Sass 和 Compass 之类的编译器。通常，我们需要使用外部库或工具编译 Web 上的资产。Sprockets 包受到 Ruby on Rails 资产管道的启发（[`guides.rubyonrails.org/asset_pipeline.html`](http://guides.rubyonrails.org/asset_pipeline.html)）。它处理了使用 Sass、LESS 和 CoffeeScript 编译器的端口进行编译。更多信息和安装说明可在[`github.com/vesselinv/fuel-sprockets`](https://github.com/vesselinv/fuel-sprockets)找到。

# 使用 Auth 包进行用户身份验证

尽管我们在第四章中提到了 Auth 包，*演示应用程序*，让我们详细地检查一下这个包。

`Auth`包不是 FuelPHP 核心的一部分，它为 FuelPHP 中的用户身份验证提供了一个标准化的接口。这使开发人员可以编写自己的驱动程序，并轻松地集成新的驱动程序以与旧代码一起工作，但仍然保持方法一致。

`Auth`包包括三个基本驱动程序：登录、组和**ACL**（**访问控制列表**）。值得注意的是，登录驱动程序可以同时处理多个登录驱动程序。例如，用户可以使用用户名和密码对进行登录，或者通过第三方系统（如 Twitter）进行身份验证。

包括的三个驱动程序是**SimpleAuth**、**ORMAuth**和**OPAuth**。

## SimpleAuth

这是一个简单的身份验证系统，它使用数据库表来存储用户信息。它在配置文件中存储有关组、角色和 ACL 的信息。它被用作管理系统的一部分。正如我们在第四章中所看到的，*演示应用程序*，SimpleAuth 包括创建存储用户信息的数据库表所需的迁移。只需配置应用程序以使用 SimpleAuth，然后运行以下命令即可创建表格：

```php
**oil r migrate --package=auth**

```

有关配置 SimpleAuth 的更多信息以及使用示例可以在以下链接找到：

[`fuelphp.com/docs/packages/auth/simpleauth/intro.html`](http://fuelphp.com/docs/packages/auth/simpleauth/intro.html)

## ORMAuth

在许多方面，这与 SimpleAuth 类似，只是它将所有配置存储在数据库中，而不是在配置文件中。

用户控件和角色可以更加精细化，并分配给个别用户。与 SimpleAuth 不同，用户属性存储在元数据表中，而不是在用户表中的序列化数组中。另一个不错的功能是保留用户登录时间的历史记录。

要启用 ORMAuth 包，您需要将`ORMAuth`添加到`app/config/config.php`文件中的`always load`代码部分。

更多信息可以在以下链接找到：

[`fuelphp.com/docs/packages/auth/ormauth/usage.html`](http://fuelphp.com/docs/packages/auth/ormauth/usage.html)

## OPAuth

这是提供的三个驱动程序中最复杂的一个。它支持对 OAuth 和 OpenID 提供程序进行身份验证。它还支持单一登录；因此，当用户登录到第三方网站（如 Twitter）时，OPAuth 将能够检测到会话并透明地登录用户。

有关 OPAuth 驱动程序的更多信息可以在以下链接找到：

[`fuelphp.com/docs/packages/auth/opauth/intro.html`](http://fuelphp.com/docs/packages/auth/opauth/intro.html)

正如您在日志管理系统的脚手架中所看到的，身份验证方法将需要在我们的应用程序中实现。Auth 方法有很好的文档，并且命名如下：

+   `Auth::check()`: 此方法检查用户是否已经认证，并返回一个布尔值 true 或 false

+   `Auth::remember_me()`: 此方法创建一个“记住我”cookie

+   `Auth::dont_remember_me()`: 此方法删除“记住我”cookie

+   `Auth::logout()`: 此方法注销用户

+   `Auth::create_user( array())`: 此方法注册一个用户

每个 Web 应用程序都是不同的，但至少 FuelPHP Auth 软件包为用户认证提供了一个很好的起点。对于大多数用途来说，这通常已经足够了。

# Composer

目前有很多种方法来管理代码和安装第三方功能。Ruby 世界有 Gem 打包系统。正如在第一章中提到的，FuelPHP 正在采用 PHP 编码和互操作标准。其中之一是能够在不重写为 FuelPHP 软件包的情况下使用其他框架的代码。

在项目的生命周期中，软件包可能会随着新功能和安全修复而发生变化。就像 Ruby on Rails 的 Bundler 一样，PHP 有一个名为 Composer 的依赖管理器。

Composer 允许您声明要在项目中安装哪些库的版本，并将为您安装它们。在开发和测试时非常有用，因为您知道确切安装了哪些代码。它还允许您对这些库的任何更改进行源代码控制。

尽管 FuelPHP 软件包和 Composer 仍处于早期阶段，但可以在以下链接找到一些软件包：

[`packagist.org/search/?q=fuel-`](https://packagist.org/search/?q=fuel-)

要向项目添加更多依赖项，只需更改`~/Sites/journal/composer.json`中找到的`composer.json`文件。

一旦找到要在 Composer 中使用的软件包，就需要在您的`composer.json`文件中添加类似以下代码：

```php
"require" : {
    "monolog/monolog": "1.2.*"
    }
```

这将确保您拥有 Monolog 1.2 软件包的最新点版本。

# 构建自己的软件包简介

到目前为止，您已经看到软件包非常有用，而且创建起来也很简单。在本节中，我们将通过创建一个文本操作软件包来介绍一些基础知识。

## 设置存储库

首先要做的是设置一个存储库。就像在上一章中一样，我们将使用 GitHub。

我们将创建一个名为**Journal String**的软件包；这将有一个名为`journal-string`的存储库名称。通常建议使用类似 Fuel String 的名称，但由于这只是一个简单的例子，因此不需要在软件包标题中包含 Fuel。

我们将使用新的存储库作为日志项目中的一个子模块，因此请确保记下存储库地址，类似于`git@github.com:digitales/journal-string.git`。

## 将软件包作为子模块使用

我们需要将这个新的子模块添加到我们的项目中，所以现在是加载控制台/终端窗口的时候了。在终端中，导航到日志项目的顶层，然后添加子模块。确保它被克隆到软件包目录中。在这个例子中，我们将子模块检出到一个名为 string 的目录中，而不是`journal-string`；这样做是为了节省输入并使自动加载更容易。

```php
**$ cd ~/Sites/journal**
**$ git submodule add git@github.com:digitales/journal-text.git fuel/packages/string**
**$ cd fuel/packages/string**

```

最后一个命令将我们带入`string`软件包目录。Git 子模块充当主项目存储库中封装的完全独立的存储库。这意味着对`journal-text`软件包所做的任何更改都需要提交到其自己的存储库，然后需要更新主存储库。

## 构建软件包

由于您将与您的团队或自己或社区共享软件包，因此将来目录和文件的结构非常重要。在使用软件包时，这将帮助您快速熟悉。建议的目录结构如下所示：

```php
**/packages**
 **/package**
 **/bootstrap.php**
 **/classes**
 **/class1.php**
 **/second-class.php**
 **/config**
 **/packageconfig.php**

```

每个包都应该在包的顶级目录中包含一个 `bootstrap.php` 文件。这个文件可以用来为包添加命名空间，并为了更好的性能和自动加载目的添加包类。如果您想要覆盖核心类，您需要将包命名空间添加到核心命名空间中，例如：

```php
Autoloader::add_core_namespace( 'String', true );
```

让我们创建我们的包结构，如下所示的层次结构：

```php
**packages/**
 **/journal/**
 **/classes/**
 **string.php**
 **/config/**
 **string.php**
 **bootstrap.php**
 **readme.md**

```

在这个例子中，我们将为一串文本创建一些基本的双向加密，并给它命名空间 `String`。我们将有一个配置文件，允许我们对加密后的字符串进行加盐。所以，让我们首先创建示例配置文件。加载位于 `/journal/config/` 的 `string.php` 文件，并添加以下代码片段：

```php
<?php
/**
 * NOTICE:
 *
 * If you need to make modifications to the default configuration, copy
 * this file to your app/config folder, and make them in there.
 *
 * This will allow you to upgrade without losing your custom config.
 */
return array(
    'active' => Fuel::$env,
    'development' => array(
        'salt' => 'put_your_salt_here',
    ),
    'production' => array(
        'salt' => 'put_your_salt_here',
    ),  
);
```

在 `string.php` 文件中，有一些示例环境。在配置文件中，我们还动态设置环境为 `Fuel::$env`。这将在主字符串类中用于加载正确环境的配置。然后，正确的值将被分配给一个名为 `$_config` 的静态类变量。

在 `encode` 和 `decode` 函数中，我们使用了 FuelPHP **Crypt** 功能。我们还包括了一些大写和小写字符串操作函数，以供后续演示目的使用。不再拖延，以下是字符串操作类的示例：

```php
<?php
/**
 * String manipulation Package
 *
 * This is a simple set of methods for string manipulation
 *
 * @license    MIT License
 */
namespace String;
```

这个类将包括一个 `StringExeption` 异常，它扩展了 FuelPHP 的 `Exception` 类，允许我们在需要时自定义异常：

```php
class StringException extends \Exception {}

class String {
    protected static $config;

    /**
     * @var  object  PHPSecLib hash object
     */
    protected static $hasher;
```

以下函数用于从应用程序配置目录中获取 `string.php` 文件：

```php
    protected static function get_config()
    {
        if ( !static::$config ):
            $config = \Config::load('string', true);
            static::$config = $config[ $config['active'] ];
        endif;
        return static::$config;    
    }

    /**
     * Encode a string using the core encode function
     *
     *  This will retrieve the salt and use that for the encoding.
     *
     *  @param string $string The string to be encoded
     *  @param null | string $salt If the salt is null, the config will be used instead.
     *  @return string
     */
    public static function encode( $string, $salt = null )
    {   
        if ( ! $salt) {
             $config = self::get_config();
    $salt = $config['salt']; 
        }
        return \Crypt::encode( $string, $salt );
    }
```

`encode()` 函数将对字符串进行编码并返回加密后的字符串。接下来，我们将在 `decode()` 函数中解码字符串：

```php

    /**
     * Decode a string using the core decode function
     *
     *  This will retrieve the salt and use that for the decoding
     *
     *  @param string $string The string to be decoded
     *  @param null | string $salt If the salt is null, the config will be used instead.
     *  @return string
     */
    public static function decode( $string, $salt = null )
    {
        if ( ! $salt) {
            $config = self::get_config();
            $salt = $config['salt'];   
        }    
        return \Crypt::decode ( $string, $salt );
    }
```

现在，让我们介绍一个快速的**密码哈希**方法。值得注意的是，在 PHP 5.5 中，我们可以使用新的密码哈希算法：

```php

    /**
     * Default password hash method
     *
     * @param   string
     * @return  string
     */
    public static function hash_password($password)
    {    
        $config = self::get_config();       
            $salt = $config['salt'];
        return base64_encode(self::hasher()->pbkdf2($password, $config['salt'], 10000, 32));
    }

    /**
     * Returns the hash object and creates it if necessary
     *
     * @return  PHPSecLib\Crypt_Hash
     */
    public static function hasher()
    {
        if ( !static::$hasher ):
            if ( ! class_exists('PHPSecLib\\Crypt_Hash', false))
            {
                import('phpseclib/Crypt/Hash', 'vendor');
            }
            $hasher = new \PHPSecLib\Crypt_Hash();
            return $hasher;
        endif;

        return static::$hasher;
    }
```

在接下来的几个函数中，我们将执行一些简单的文本字符串操作。首先，让我们将字符串改为全部小写：

```php

    /**
     * Convert the string to lowercase.
     *
     * @param   string  $str       required
     * @param   string  $encoding  default UTF-8
     * @return  string
     */
    public static function lower($str, $encoding = null)
    {
        $encoding = \Fuel::$encoding;

        if ( function_exists('mb_strtolower') ){
            return mb_strtolower($str, $encoding);
        } else {
            return strtolower($str);
        }

    }
```

现在，让我们将字符串转换为大写：

```php
    /**
     * Covert the string to uppercase.
     *
     * @param   string  $str       required
     * @param   string  $encoding  default UTF-8
     * @return  string
     */
    public static function upper($str, $encoding = null)
    {
        $encoding or $encoding = \Fuel::$encoding;

        if ( function_exists('mb_strtoupper') {
            return mb_strtoupper($str, $encoding);
        } else {
            return strtoupper($str);
        }

    }
```

现在，让我们将每个单词的第一个字符变为小写：

```php
    /**
     * lcfirst
     *
     * Does not strtoupper first
     *
     * @param   string  $str       required
     * @param   string  $encoding  default UTF-8
     * @return  string
     */
    public static function lcfirst($str, $encoding = null)
    {
        $encoding or $encoding = \Fuel::$encoding;

        if(function_exists('mb_strtolower')){
            return mb_strtolower(mb_substr($str, 0, 1, $encoding), $encoding).
                mb_substr($str, 1, mb_strlen($str, $encoding), $encoding);
        }else{
            return lcfirst($str);
        }

    }
```

现在，让我们将每个单词的第一个字符变为大写：

```php
    /**
     * ucfirst
     *
     * Does not strtolower first
     *
     * @param    string $str       required
     * @param    string $encoding  default UTF-8
     * @return   string
     */
    public static function ucfirst($str, $encoding = null)
    {
        $encoding or $encoding = \Fuel::$encoding;

        if(function_exists('mb_strtoupper')_{
            return mb_strtoupper(mb_substr($str, 0, 1, $encoding), $encoding).
                mb_substr($str, 1, mb_strlen($str, $encoding), $encoding);
        }else{
            return ucfirst($str);
        }

    }
```

现在，让我们将每个单词的第一个字符大写：

```php
    /**
     * ucwords
     *
     * First strtolower then ucwords
     *
     * ucwords normally doesn't strtolower first
     * but MB_CASE_TITLE does, so ucwords now too
     *
     * @param   string   $str       required
     * @param   string   $encoding  default UTF-8
     * @return  string
     */
    public static function ucwords($str, $encoding = null)
    {
        $encoding or $encoding = \Fuel::$encoding;

        if ( function_exists('mb_convert_case') ){
            return mb_convert_case($str, MB_CASE_TITLE, $encoding);
        } else {
            return ucwords(strtolower($str));
        }

    }
}
```

现在我们已经解决了基本功能，应用程序代码需要能够访问它。为此，我们可以使用日志包目录顶层的 `bootstrap.php` 文件。

加载位于 `packages/journal-string/` 的 `bootstrap.php` 文件：

```php
<?php
Autoloader::add_core_namespace('String');

Autoloader::add_classes(array(
    'String\\String' => __DIR__.'/classes/string.php',
    'String\\StringException' => __DIR__.'/classes/string.php'
));
```

然后，我们就可以使用类似以下的方式调用功能：

```php
String::decode( $the_string );
```

## 配置您的包

使用该包时的第一件事是创建一个特定于项目的包配置版本。要做到这一点，在您的终端中运行以下命令：

```php
**$ cp ~/Sites/journal/fuel/packages/journal-string/config/string.php  ~/Sites/journal/fuel/app/config/string.php**

```

我们需要添加一些自定义的 `salt` 文本字符串，这些将作为新复制的 `string.php` 配置中的键使用：

```php
return array(
    'active' => Fuel::$env,
    'development' => array(
        'salt' => '(my awesome salt)',
    ),
    'production' => array(
        'salt' => '(my awesome salt)',
    ),  
);
```

## 使用您的包

现在，您已经配置了包并创建了字符串函数，是时候演示如何使用新包了。首先，让我们将 `String` 包添加到我们的 `config.php` 应用程序文件中：

由于我们添加了一个核心命名空间 `String`，我们可以使用以下方法调用我们的字符串函数：

```php
**$encoded_string = String::encode( 'something to encode');**
**$decoded_string = String::decode();**
**echo $decoded_string;**

```

您可以在控制器中测试功能，然后在视图中显示结果。创建包是一个简单的过程，您应该熟悉它。一旦您创建了您的包，您可能希望与他人分享。

# 让人们了解您的包

所以，您已经创建了您的包，现在是时候发布它了。首先，检查所有函数是否都有注释，并且您已经在`Readme.md`文件（或`Readme.txt`文件）中记录了如何使用该包的方法是个好主意。如果您在 GitHub 上编写代码，他们提供了一个快速创建网页来宣传您的包或项目的方法。在 GitHub 上创建页面时，他们将使用 Readme 文件作为起点，然后让您自定义关于您的包的任何信息。更多信息可以在[`help.github.com/categories/20/articles`](https://help.github.com/categories/20/articles)中找到。

一旦您确定代码已经准备好分享，就发送一条推文到 FuelPHP 的 Twitter 账号（[`twitter.com/FuelPHP`](https://twitter.com/FuelPHP)）。他们经常会“转推”您的消息给他们的关注者。除此之外，您还可以在 FuelPHP 论坛上分享您的包链接，网址是[`fuelphp.com/forums/categories/codeshare`](http://fuelphp.com/forums/categories/codeshare)。

# 总结

在本章中，我们已经涵盖了一些包的基础知识，以及一些有用的包的示例，这些包可以让我们的开发工作更加轻松。有了一系列可靠的包，我们可以集中精力去创建应用程序，并交付客户想要的东西。我们已经创建了一个包，配置了它，并演示了它的使用。

在下一章中，我们将涵盖一些更高级的主题，包括功能可移植性、单元测试和在 FuelPHP 中进行性能分析。
