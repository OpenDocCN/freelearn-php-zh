# 前言

PHP 社区在几十年来面临着一个巨大的问题：性能。无论他们拥有多么强大的硬件，最终 PHP 本身都成为了瓶颈。随着 PHP 5.4.x、5.5.x 和 5.6.x 的推出，PHP 的性能开始改善，但在高负载应用中仍然是一个巨大的问题。社区开发了诸如**Alternative PHP Cache**（**APC**）和 Zend OpCache 之类的缓存工具，这些工具对性能产生了良好的影响。

为了解决 PHP 的性能问题，Facebook 构建了他们自己的开源工具**HHVM**（**HipHop 虚拟机**）。根据他们的官方网站，HHVM 使用即时（JIT）编译来实现卓越的性能，同时保持 PHP 提供的开发灵活性。与 PHP 相比，HHVM 的性能非常出色，并且广泛用于像 Magento 这样的重型应用的生产环境。

PHP 通过**PHP Next Generation**（**PHPNG**）与 HHVM 展开了竞争。PHPNG 的整个目的是提高性能，并专注于重写和优化 Zend 引擎内存分配和 PHP 数据类型。世界各地的人开始对 PHPNG 和 HHVM 进行基准测试，据他们称，PHPNG 的性能优于 HHVM。

最后，PHPNG 与 PHP 的主分支合并，经过大量优化和完全重写，PHP 7 发布，性能大幅提升。PHP 7 仍然不是 JIT，但其性能很好，与 HHVM 类似。这是与旧版本 PHP 相比的巨大性能提升。

# 本书涵盖的内容

第一章，*设置环境*，介绍了如何设置不同的开发环境，包括在 Windows、不同的 Linux 发行版上安装 NGINX、PHP 7 和 Percona Server，以及为开发目的设置 Vagrant 虚拟机。

第二章，*PHP 7 的新特性*，介绍了 PHP 7 引入的主要新特性，包括类型提示、组使用声明、匿名类和新操作符，如太空船操作符、空合并操作符和统一变量语法。

第三章，*改善 PHP 7 应用程序性能*，介绍了不同的技术来增加和扩展 PHP 7 应用程序的性能。在本章中，我们涵盖了 NGINX 和 Apache 的优化、CDN 和 CSS/JavaScript 的优化，如合并和最小化它们，全页面缓存以及安装和配置 Varnish。最后，我们讨论了应用开发的理想基础架构设置。

第四章，*改善数据库性能*，介绍了优化 MySQL 和 Percona Server 配置以实现高性能的技术。还介绍了不同的工具来监控数据库的性能。还介绍了用于缓存对象的 Memcached 和 Redis。

第五章，*调试和性能分析*，介绍了调试和性能分析技术，包括使用 Xdebug 进行调试和性能分析，使用 Sublime Text 3 和 Eclipse 进行调试，以及 PHP DebugBar。

第六章，*压力/负载测试 PHP 应用程序*，介绍了不同的工具来对应用程序进行压力和负载测试。涵盖了 Apache JMeter、ApacheBench 和 Siege 用于负载测试。还介绍了如何在 PHP 7 和 PHP 5.6 上对 Magento、Drupal 和 WordPress 等不同开源系统进行负载测试，并比较它们在 PHP 7 和 PHP 5.6 上的性能。

第七章，*PHP 编程的最佳实践*，介绍了一些生产高质量标准代码的最佳实践。涵盖了编码风格、设计模式、面向服务的架构、测试驱动开发、Git 和部署。

附录 A, *使生活更轻松的工具*，更详细地讨论了其中三种工具。我们将讨论的工具是 Composer、Git 和 Grunt watch。

附录 B, *MVC 和框架*，涵盖了 PHP 开发中使用的 MVC 设计模式和最流行的框架，包括 Laravel、Lumen 和 Apigility。

# 您需要为本书准备什么

任何符合运行以下软件的最新版本的硬件规格都应足以完成本书的学习：

+   操作系统：Debian 或 Ubuntu

+   软件：NGINX、PHP 7、MySQL、PerconaDB、Redis、Memcached、Xdebug、Apache JMeter、ApacheBench、Siege 和 Git

# 这本书适合谁

这本书适合那些具有 PHP 编程基础经验的人。如果您正在开发性能关键的应用程序，那么这本书适合您。

# 约定

在本书中，您会发现一些文本样式，用于区分不同类型的信息。以下是这些样式的一些示例及其含义的解释。

文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 用户名显示如下：“我们可以通过使用`include`指令来包含其他上下文。”

代码块设置如下：

```php
location ~ \.php$ {
  fastcgi_pass    127.0.0.1:9000;
  fastcgi_param    SCRIPT_FILENAME complete_path_webroot_folder$fastcgi_script_name;
  include    fastcgi_params;
}
```

当我们希望引起您对代码块的特定部分的注意时，相关行或项会以粗体显示：

```php
server {
  …
  …
 **root html;**
 **index index.php index.html index.htm;**
  …
```

任何命令行输入或输出都以以下方式编写：

```php
**php-cgi –b 127.0.0.1:9000**

```

**新术语**和**重要单词**以粗体显示。例如，在屏幕上看到的单词，比如菜单或对话框中的单词，会在文本中出现，就像这样：“点击**下一步**按钮会将您移动到下一个屏幕。”

### 注意

警告或重要说明会以这样的方式出现在一个框中。

### 提示

提示和技巧会以这种方式出现。
