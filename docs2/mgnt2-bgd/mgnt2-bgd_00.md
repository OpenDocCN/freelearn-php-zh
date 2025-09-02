# 前言

构建基于 Magento 的商店可能是一项具有挑战性的任务。它需要一系列与 PHP/JavaScript 编程语言、开发和生产环境以及众多 Magento 特定功能相关的技术技能。本书将提供关于 Magento 构建块所需的知识。

到这本书的结尾，你应该熟悉配置文件、依赖注入、模型、集合、块、控制器、事件、观察者、插件、定时任务、配送方式、支付方式以及一些其他内容。所有这些都应该为你后续的开发旅程打下坚实的基础。

# 本书涵盖内容

第一章, *理解平台架构*，对技术堆栈、架构层、顶级系统结构和单个模块结构进行了高级概述。

第二章, *管理环境*，介绍了 VirtualBox、Vagrant 和 Amazon AWS 作为设置开发和生产环境的平台。它还提供了设置/脚本 Vagrant 和 Amazon EC2 盒子的实践示例。

第三章, *编程概念和约定*，向读者介绍了几个看似不相关但重要的 Magento 部分，如 composer、服务合约、代码生成、var 目录，以及最终的编码标准。

第四章, *模型和集合*，探讨了模型、资源、集合、架构和数据脚本。它还展示了应用于实体的实际 CRUD 操作以及过滤集合。

第五章, *使用依赖注入*，引导读者了解依赖注入机制。它解释了对象管理器的角色，如何配置类偏好，以及如何使用虚拟类型。

第六章, *插件*，深入探讨了名为插件的新概念。它展示了如何通过 before/after/around 监听器轻松扩展或添加现有功能。

第七章, *后端开发*，通过实践方法介绍通常被认为是后端相关开发的部分。这些包括定时任务、通知消息、会话、Cookies、日志、性能分析器、事件、缓存、小部件等。

第八章 *前端开发* 采用更高级的方法，引导读者了解大多数被认为是前端相关开发的内容。它涉及到在 Magento 中渲染流程、视图元素、块、模板、布局、主题、CSS 和 JavaScript。

第九章 *Web API* 详细介绍了 Magento 提供的强大 Web API。它提供了实际操作示例，以创建和使用 REST 和 SOAP，无论是通过 PHP cURL 库还是从控制台。

第十章 *主要功能区域* 采用了高级方法，向读者介绍 Magento 中一些最常见的部分。这包括 CMS、目录和客户管理、以及产品和客户导入。它甚至展示了如何创建自定义产品类型和运输及支付方式。

第十一章 *测试* 概述了在 Magento 中可用的测试类型。它进一步展示了如何编写和执行自定义测试。

第十二章 *从头开始构建模块* 展示了开发模块的整个过程，该模块使用了前几章中介绍的大多数功能。最终结果是具有管理后台和店面界面、管理配置区域、电子邮件模板、已安装的架构脚本、测试等的模块。

# 你需要这本书的内容

为了成功运行本书中提供的所有示例，你需要自己的网络服务器或第三方网络托管解决方案。高级技术堆栈包括 PHP、Apache/Nginx 和 MySQL。Magento 2 社区版平台本身附带了一份详细的系统要求列表，可以在[`devdocs.magento.com/guides/v2.0/install-gde/system-requirements.html`](http://devdocs.magento.com/guides/v2.0/install-gde/system-requirements.html)找到。实际的环境设置在第二章 *管理环境* 中有详细说明。

# 这本书面向的对象

本书主要面向对 Magento 2 开发感兴趣的初级到专业 PHP 开发者。对于后端开发者，本书涵盖了多个主题，将帮助你修改和扩展你的 Magento 店铺。前端开发者也会找到一些关于如何在前端定制网站外观的覆盖内容。

由于代码和结构发生了大量变化，Magento 2.x 版本可以描述为一个与其前身显著不同的平台。考虑到这一点，本书将不会假设或要求读者具备对 Magento 1.x 的先验知识。

# 惯例

在本书中，您将找到许多文本样式，用于区分不同类型的信息。以下是一些这些样式的示例及其含义的解释。

文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 昵称显示如下：“`AbstractProductPlugin1`类不需要从另一个类扩展，插件才能工作。”

代码块设置如下：

```php
<config  xsi:noNamespaceSchemaLocation="urn:magento:framework: ObjectManager/etc/config.xsd">
    <type name="Magento\Catalog\Block\Product\AbstractProduct">
        <plugin name="foggyPlugin1" type="Foggyline\Plugged\Block\Catalog\Product\ AbstractProductPlugin1" disabled="false" sortOrder="100"/>
        <plugin name="foggyPlugin2" type="Foggyline\Plugged\Block\Catalog\Product\ AbstractProductPlugin2" disabled="false" sortOrder="200"/>
        <plugin name="foggyPlugin3" type="Foggyline\Plugged\Block\Catalog\Product\ AbstractProductPlugin3" disabled="false" sortOrder="300"/>
    </type>
</config>
```

任何命令行输入或输出都按如下方式编写：

```php
php bin/magento setup:upgrade

```

**新术语**和**重要词汇**以粗体显示。屏幕上看到的单词，例如在菜单或对话框中，在文本中显示如下：“在**商店视图**下拉字段中，我们选择要应用主题的商店视图。”

### 注意

警告或重要注意事项以如下框显示。

### 小贴士

小贴士和技巧显示如下。

# 读者反馈

我们欢迎读者的反馈。请告诉我们您对这本书的看法——您喜欢或不喜欢什么。读者反馈对我们来说很重要，因为它帮助我们开发出您真正能从中获得最大价值的标题。

要向我们发送一般反馈，只需发送电子邮件至 `<feedback@packtpub.com>`，并在邮件主题中提及本书的标题。

如果您在某个领域有专业知识，并且对撰写或参与一本书籍感兴趣，请参阅我们的作者指南，网址为 [www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

现在，您已经成为 Packt 书籍的骄傲拥有者，我们有一些事情可以帮助您从购买中获得最大收益。

## 下载示例代码

您可以从您在 [`www.packtpub.com`](http://www.packtpub.com) 的账户下载示例代码文件，适用于您购买的所有 Packt 出版物。如果您在其他地方购买了这本书，您可以访问 [`www.packtpub.com/support`](http://www.packtpub.com/support) 并注册，以便将文件直接通过电子邮件发送给您。

## 错误清单

尽管我们已经尽一切努力确保内容的准确性，但错误仍然可能发生。如果您在我们的书中发现错误——可能是文本或代码中的错误——如果您能向我们报告这一点，我们将不胜感激。通过这样做，您可以节省其他读者的挫败感，并帮助我们改进本书的后续版本。如果您发现任何勘误，请通过访问 [`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)，选择您的书籍，点击**勘误****提交****表单**链接，并输入您的勘误详情。一旦您的勘误得到验证，您的提交将被接受，勘误将被上传到我们的网站或添加到该标题的勘误部分下的现有勘误列表中。

要查看之前提交的勘误表，请访问 [`www.packtpub.com/books/content/support`](https://www.packtpub.com/books/content/support)，并在搜索字段中输入书籍名称。所需信息将在**勘误**部分显示。

## 海盗行为

互联网上对版权材料的盗版是一个跨所有媒体的持续问题。在 Packt，我们非常重视我们版权和许可证的保护。如果您在互联网上发现任何形式的我们作品的非法副本，请立即提供位置地址或网站名称，以便我们可以寻求补救措施。

请通过 `<copyright@packtpub.com>` 联系我们，并提供涉嫌盗版材料的链接。

我们感谢您在保护我们作者和我们为您提供有价值内容的能力方面的帮助。

## 询问

如果您对本书的任何方面有问题，您可以通过 `<questions@packtpub.com>` 联系我们，我们将尽力解决问题。
