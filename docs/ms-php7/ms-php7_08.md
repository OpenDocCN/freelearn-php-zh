# 第八章：无服务器之前

“无服务器”一词可能是最近软件行业中最热门的术语之一。它可以被描述为部分或完全抽象出运行软件所需的基础架构的架构风格。这种抽象通常由各种第三方服务提供商提供。

将其放在 Web 应用程序开发的背景下，让我们考虑单页应用程序（SPA）。如今，我们可以轻松地在完全托管的基础架构上开发整个 SPA，比如 AWS。这样的 SPA 可以用 Angular 编写，客户端组件可以从 S3 存储桶中提供，通过 Amazon Cognito 服务管理用户，同时使用 DynamoDB 作为应用程序数据存储。托管的基础架构将我们从任何主机或服务器交易中抽象出来，使我们能够将精力集中在应用程序上。我们最终得到的是一种无服务器应用程序，这取决于我们定义的范围有多窄。

像任何架构风格一样，无服务器远非“解决方案”。虽然某些类型的应用程序可能会从中受益，但其他类型的应用程序可能会发现它完全不匹配。例如，长时间运行的应用程序可能很容易成为无服务器框架的昂贵解决方案，而不是在专用服务器上运行工作负载。关键是找到合适的平衡。

无服务器的更严格和狭窄的定义是纯代码/函数托管，通常称为函数即服务（FaaS）。这样的基础设施提供高并发、可扩展、成本效益的解决方案，因为它们大多是按“按执行付费”的模式定价。AWS Lambda 和 Iron.io 是两个完美体现这一概念的平台。

在本章中，我们将更仔细地看看如何利用 AWS Lambda 和 Iron.io 平台来部署我们代码的块：

+   使用无服务器框架

+   使用 Iron.io IronWorker

# 使用无服务器框架

AWS Lambda 是由亚马逊网络服务（AWS）提供的计算服务。它的特点是可以在不提供或管理任何服务器的情况下运行代码。自动扩展功能使其能够承受每秒数千个请求。加上按执行付费的额外好处，这项服务在开发人员中引起了一些关注。随着时间的推移，无服务器框架被开发出来，以使 AWS Lambda 服务的使用变得容易。

无服务器框架可在[`serverless.com`](https://serverless.com)上找到。

假设我们已经创建了一个 AWS 账户，并且手头上有一个干净的 Ubuntu 服务器安装，让我们继续概述设置和利用无服务器框架所需的步骤。

在我们可以在 AWS Lambda 上部署应用程序之前，我们需要确保我们有一个具有正确权限集的用户。AWS 权限非常强大，我们可以根据资源对其进行调整。无服务器框架除了 AWS Lambda 本身之外，还使用了其他几个 AWS 资源，如 S3、API Gateway 等。为了使我们的演示简单，我们将首先创建一个具有管理员访问权限的 IAM 用户：

1.  我们首先登录到[`aws.amazon.com/console/`](https://aws.amazon.com/console/)的 AWS 控制台。登录后，我们需要在“我的安全凭证”|“用户”屏幕下继续：

![](img/26e8dfc0-bee9-4725-9426-068b56ebcace.png)

1.  要添加新用户，我们点击“添加用户”按钮。这将触发一个四步过程，如下截图所示：

![](img/2b71f9fd-594c-4e29-a045-f80cebfa993c.png)

1.  我们在这里提供两条信息，用户名和访问类型。我们的无服务器集成需要编程访问类型。单击“下一步：权限”按钮将我们带到以下屏幕：

![](img/afed7936-66d3-43d5-b95c-200800c9b0ac.png)

1.  这里有几种方法可以在这里为用户附加权限。 为了保持简单，我们点击“直接附加现有策略”框，并在“策略类型”字段过滤器中输入 AdministratorAccess。 然后我们简单地勾选 AdministratorAccess 策略，然后点击“下一步：审阅”按钮，这将带我们到以下屏幕：

![](img/e4ce7578-bcdc-45a8-9e40-d3c2c4cf9a61.png)

1.  在这里，我们仅仅回顾了当前的进展，最后点击“创建用户”按钮，这将带我们到以下屏幕：

![](img/f8227089-6927-46fa-a6a4-195feb1ce937.png)

1.  现在我们有了 Access key ID 和 Secret access key，这是 serverless 框架所需的两个信息。

通常认为，创建具有完整管理权限的用户是一个不好的安全实践。 通常，我们会创建具有所需权限的最低限度的用户。

完成了这些步骤，我们可以继续设置 serverless 框架本身。

serverless 框架运行在 Node.js 之上。 假设我们有一个干净的 Ubuntu 服务器实例，我们可以通过以下步骤进行设置：

1.  使用以下控制台命令安装 Node.js：

```php
curl -sL https://deb.nodesource.com/setup_7.x | sudo -E bash -
sudo apt-get install -y nodejs

```

1.  一旦安装了 Node.js，`npm`控制台工具就可用了。 服务器框架本身作为一个`npm`包可在[`www.npmjs.com/package/serverless`](https://www.npmjs.com/package/serverless)上获得。 运行以下控制台命令应该可以在我们的服务器上安装它：

```php
sudo npm install -g serverless
serverless --version

```

![](img/3ff41a29-4d16-41cf-bff0-4bf06d7490e2.png)

1.  现在安装了 serverless 框架，我们需要设置控制台环境变量：`AWS_ACCESS_KEY_ID`和`AWS_SECRET_ACCESS_KEY`。 这些在部署期间由 serverless 使用：

```php
export AWS_ACCESS_KEY_ID=<--AWS_ACCESS_KEY_ID-->
export AWS_SECRET_ACCESS_KEY=<--AWS_SECRET_ACCESS_KEY--> 

```

1.  现在我们可以处理与 PHP 相关的细枝末节了。 官方 serverless 框架示例使用运行 PHP 函数的 AWS Lambda，可以在[`github.com/ZeroSharp/serverless-php`](https://github.com/ZeroSharp/serverless-php)找到。 我们可以通过以下控制台命令安装它：

```php
serverless install --url https://github.com/ZeroSharp/serverless-php

```

这应该给我们一个类似以下截图的输出：

![](img/ecebec78-dabf-4a68-b734-acdfb7542473.png)

serverless 安装命令只是将 Git 存储库的内容拉到本地目录中。 在新创建的`serverless-php`目录中，有一个`index.php`文件，其中包含我们的 PHP 应用程序代码。 令人奇怪的是，这里有一些东西，乍一看似乎与 PHP 无关，比如`handler.js`。 快速查看`handler.js`揭示了一些有趣的东西，即 AWS Lambda 服务实际上并不直接运行 PHP 代码。 它的工作方式是`handler.js`，这是一个 Node.js 应用程序，生成一个带有包含的`php`二进制文件的进程。 简而言之，`index.php`是我们的应用程序文件，其余的是必要的样板。

作为一个快速的健全检查，让我们触发以下两个命令：

```php
php index.php
serverless invoke local --function hello

```

这些应该给我们以下输出，表明 serverless 能够看到并执行我们的函数：

![](img/43fdf30e-4b1c-4deb-a925-3602a44b81e1.png)

最后，我们准备将我们的 PHP 应用程序部署到 AWS Lambda 服务。 我们通过执行以下命令来实现这一点：

```php
serverless deploy

```

![](img/95344457-6c57-4980-b8b5-1142d0972798.png)

这个简单的命令启动了一系列事件，导致在 AWS 控制台中利用了几种不同的 AWS 服务。

打开在端点下列出的链接显示我们的应用程序是公开可用的：

![](img/9ce30662-5947-40ef-ab1f-c83d254b707e.png)

这是由 Amazon API Gateway 服务下自动创建的 API 入口所实现的，如下截图所示：

![](img/e369c9b8-3397-46dd-b7c2-5981dca8800f.png)

API Gateway 将`GET /hello` URL 操作与 AWS Lambda `serverless-php-dev-hello`应用程序连接起来。 在 AWS Lambda 屏幕下查看这个应用程序：

![](img/7f7e212a-3c65-41bd-bc8b-fd69a845e979.png)

CloudFormation 堆栈也已创建，如下截图所示：

![](img/b167b8cc-6d54-4f29-8577-ba77cdb709ba.png)

S3 存储桶也已创建，如下所示：

![](img/7f0a9b21-a5be-4b4f-b51d-9775fb452b9a.png)

CloudWatch 日志组也已创建，如下截图所示：

![](img/e52db856-4769-46ea-aa17-0b1c01f8e784.png)

简而言之，`serverless deploy`为我们启动了许多服务，因此我们有更多时间专注于实际的应用程序开发。尽管 AWS Lambda 只在运行代码时收费，但混合使用的其他一些服务可能是不同的。这就是为什么重要的是要密切关注自动触发的一切。

幸运的是，无服务器还提供了一个清理命令，写成如下：

```php
serverless remove

```

![](img/8a1807f7-94d9-4e90-bcab-e513dec0fa22.png)

此命令通过删除先前创建的所有服务和资源来进行总体清理。

# 使用 Iron.io IronWorker

Iron.io 是一个为高性能和并发设计的无服务器作业处理平台。该平台围绕 Docker 容器构建，本身是与语言无关的。我们可以使用它来运行几乎任何编程语言，包括 PHP。Iron.io 平台的三个主要特点是：

+   **IronWorker**：这是一个弹性的任务/队列式工作服务，可扩展处理

+   **IronMQ**：这是为分布式系统设计的消息队列服务

+   **IronCache**：这是一个弹性和耐用的键/值存储

虽然我们不能在 Iron.io 平台上运行实时 PHP，但我们可以利用其 IronWorker 功能来进行任务/队列式类型的应用程序。

假设我们已经打开了一个 Iron.io 账户并且在 Ubuntu 服务器上安装了 Docker，我们就能够按照下面的步骤来了解 IronWorker 的工作流程。

我们首先点击 Iron.io 仪表板下的 New Project 按钮。这将打开一个简单的屏幕，我们只需要输入项目名称：

![](img/3893fc43-cfa5-4ed0-b4ba-048ea945b652.png)

项目创建后，我们可以点击项目设置链接。这将打开一个屏幕，显示包括认证/配置参数在内的多个信息：

![](img/271ab3b9-79d2-4e1b-817a-c2bfd145e822.png)

我们稍后将配置`iron.json`文件，因此需要这些参数。有了这些信息，我们就可以继续进行应用程序的配置。

在应用程序方面，我们首先安装`iron`控制台工具：

```php
curl -sSL https://cli.iron.io/install | sh

```

安装完成后，`iron`命令应该可以通过控制台使用，如下截图所示：

![](img/456cb0f0-3aea-4c2d-afc4-61428794c65a.png)

现在我们准备启动我们的第一个 Iron 应用。

假设我们有一个干净的目录，我们想要放置我们的应用程序文件，我们首先添加`composer.json`，内容如下：

```php
{
  "require": {
    "iron-io/iron_worker": "2.0.4",
    "iron-io/iron_mq": "2.*",
    "wp-cli/php-cli-tools": "~0.10.3"
  }
}

```

在这里，我们只是告诉 Composer 要拉取哪些库：

+   `iron_worker`：这是 IronWorker 的客户端库（[`packagist.org/packages/iron-io/iron_worker`](https://packagist.org/packages/iron-io/iron_worker)）

+   `iron_mq`：这是 IronMQ 的客户端绑定（[`packagist.org/packages/iron-io/iron_mq`](https://packagist.org/packages/iron-io/iron_mq)）

+   `php-cli-tools`：这些是用于 PHP 的控制台实用程序（[`packagist.org/packages/wp-cli/php-cli-tools`](https://packagist.org/packages/wp-cli/php-cli-tools)）

然后我们创建`Dockerfile`，内容如下：

```php
FROM iron/php

WORKDIR /app
ADD . /app

ENTRYPOINT ["php", "greet.php"]

```

这些`Dockerfile`指令帮助 Docker 自动为我们构建必要的镜像。

然后我们添加`greet.payload.json`文件及其内容如下：

```php
{
  "name": "John"
}

```

这实际上并不是流程中必要的一部分，但我们正在使用它来模拟我们的应用程序接收到的有效载荷。

然后我们添加`greet.php`文件及其内容如下：

```php
<?php

require 'vendor/autoload.php';

$payload = IronWorker\Runtime::getPayload(true);

echo 'Welcome ', $payload['name'], PHP_EOL;

```

`greet.php`文件是我们的实际应用程序。在 IronWorker 服务上创建的作业将排队并执行此应用程序。应用程序本身很简单；它只是简单地获取名为`name`的负载变量的值，并将其输出。这对于我们的 IronWorker 演示足够了。

然后创建`iron.json`文件，内容类似如下：

```php
{
  "project_id": "589dc552827e8d00072c7e11",
  "token": "Gj5vBCht0BP9MeBUNn5g"
}

```

确保我们粘贴了从 Iron.io 仪表板的项目设置屏幕获取的`project_id`和`token`。

有了这些文件，我们已经定义了我们的应用程序，现在准备开始 Docker 相关的任务。总体思路是，我们将首先创建一个本地 Docker 镜像用于测试。一旦测试完成，我们将把 Docker 镜像推送到 Docker 仓库，然后配置 Iron.io 平台使用 Docker 仓库中的镜像来驱动其 IronWorker 作业。

我们现在可以通过运行以下命令将我们的工作程序依赖项安装到 Docker 中，如`composer.json`文件所设定的。

```php
docker run --rm -v "$PWD":/worker -w /worker iron/php:dev composer install

```

输出应该显示 Composer 正在安装依赖项，如下图所示：

![](img/54528b74-a093-4227-9bfe-d67ebba7aa26.png)

一旦 Composer 安装完依赖项，我们应该测试一下我们的应用程序是否在执行。我们可以通过以下命令来做到这一点：

```php
docker run --rm -e "PAYLOAD_FILE=greet.payload.json" -v "$PWD":/worker -w /worker iron/php php greet.php

```

前面命令的输出应该是一个“欢迎 John”字符串，如下图所示：

![](img/773c3783-468f-4602-a540-ec2c766777bf.png)

这证实了我们的 Docker 镜像正常工作，现在我们准备构建并部署它到[`hub.docker.com`](https://hub.docker.com)。

Docker Hub，位于[`hub.docker.com`](https://hub.docker.com)，是一个基于云的服务，提供了集中的容器镜像管理解决方案。虽然它是一个商业服务，但也有一个免费的*一个仓库*计划。

假设我们已经打开了 Docker Hub 账户，通过控制台执行以下命令将标记我们已登录：

```php
docker login --username=ajzele

```

其中`ajzele`是用户名，应该用我们自己的替换：![](img/4443de04-07d7-4d4c-961e-ed0ed8e90bd3.png)

我们现在可以通过执行以下命令打包我们的 Docker 镜像：

```php
docker build -t ajzele/greet:0.0.1 .

```

这是一个标准的构建命令，将创建一个带有版本`0.0.1`标记的`ajzele/greet`镜像

![](img/249c6260-3677-4b42-ab8a-c29131084018.png)

现在创建了镜像，我们应该先测试它，然后再将其推送到 Docker Hub。执行以下命令确认我们新创建的`ajzele/greet`镜像工作正常：

```php
docker run --rm -it -e "PAYLOAD_FILE=greet.payload.json" ajzele/greet:0.0.1

```

![](img/27fcdeea-f539-4b2d-81d6-48e901398b95.png)

生成的欢迎 John 输出确认我们的镜像现在已准备好部署到 Docker Hub，可以使用以下命令完成：

```php
docker push ajzele/greet:0.0.1

```

![](img/0e266f2c-0cb6-44f1-9a97-7b6870f8e86c.png)

一旦推送过程完成，我们应该能在 Docker Hub 仪表板下看到我们的镜像：

![](img/962b5d25-bf1d-45e3-a699-78234cf300d2.png)

到目前为止有相当多的步骤，但我们快要完成了。现在我们的应用程序在 Docker Hub 仓库中可用作为 Docker 镜像，我们可以把重点转回 Iron.io 平台。我们在过程中早已安装的`iron`控制台工具能够在 Iron.io 仪表板下注册 Docker Hub 镜像为一个新的工作程序：

```php
iron register ajzele/greet:0.0.1

```

以下截图显示了此命令的输出：

![](img/aa2d7445-b1b8-4ee6-9c08-382268bd66e8.png)

此时，我们应该在 Iron.io 仪表板的 TASKS 选项卡下看到`ajzele/greet`工作程序：

![](img/12bdfd6d-417a-4f1a-933e-43f3ef7edd37.png)

虽然工作程序已注册，但此时尚未执行。Iron.io 平台允许我们将工作程序作为定时或排队任务执行。

如下截图所示的定时任务允许我们选择注册的 Docker 镜像以及执行时间和其他一些选项：

![](img/afdb4665-59f0-4bb3-8d05-d22a9fafeec3.png)

如下截图所示，排队任务还允许我们选择注册的 Docker 镜像，但这次没有任何特定的时间配置：

![](img/195b6831-ff13-400e-a34a-6c9e1a1a0812.png)

使用`iron`控制台工具，我们可以基于`ajzele/greet` worker 创建计划和排队任务。

以下命令创建了一个基于`ajzele/greet` worker 的计划任务：

```php
iron worker schedule --payload-file greet.payload.json -start-at="2017-02-12T14:16:28+00:00" ajzele/greet

```

`start-at`参数以 RFC3339 格式定义了一个时间。

有关 RFC3339 格式的更多信息，请查看[`tools.ietf.org/html/rfc3339`](https://tools.ietf.org/html/rfc3339)。

以下截图显示了前面命令的输出：

![](img/62b5908c-ed94-43e7-8f1c-5040177b6f1c.png)

Iron.io 仪表板现在应该将其显示为 SCHEDULED TASKS 部分下的新条目：

![](img/46ff3923-9a05-4496-a2fa-8a5c4a2d698b.png)

当计划的时间到来时，Iron.io 平台将执行此计划任务。

以下命令创建了一个基于`ajzele/greet` worker 的排队任务：

```php
iron worker queue --payload-file greet.payload.json --wait ajzele/greet

```

以下截图显示了此命令的输出：

![](img/14d527b1-f84d-4879-9f90-b6e337065ab5.png)

Iron.io 仪表板通过增加 TASKS 部分下的 Complete 计数器（在下面的截图中当前显示为*3*）记录每个执行的任务：

![](img/1cc021bf-9195-40a4-95a2-0588d956d8bc.png)

进入`ajzele/greet` worker 可以查看每个作业的详细信息，包括计划和排队的作业。

![](img/6ec8a8ac-57dd-4ce4-a397-37237b31fc67.png)

到目前为止，您已经学会了如何创建 PHP 应用程序 Docker 镜像，将其推送到 Docker Hub，将其注册到 Iron.io 平台，并开始调度和排队任务。关于调度和排队任务的部分可能有点棘手，因为我们是从控制台而不是从 PHP 代码中进行的。

幸运的是，`composer.json`文件引用了我们需要的所有库，以便能够从 PHP 代码中调度和排队任务。假设我们抓取了`iron.json`和`composer.json`文件，并移动到完全不同的服务器，甚至是我们的本地开发机器。在那里，我们只需要在控制台上运行`composer install`，并创建内容如下的`index.php`文件：

```php
<?php

require './vendor/autoload.php';

$worker = new IronWorker\IronWorker();

$worker->postScheduleAdvanced(
  'ajzele/greet',
  ['name' => 'Mariya'],
  '2017-02-12T14:33:39+00:00'
);

$worker->postTask(
  'ajzele/greet',
  ['name' => 'Alice']
);

```

一旦这段代码被执行，它将创建一个已计划和一个排队的任务，就像`iron`控制台工具一样。

虽然我们可能不会使用它来托管整个 PHP 应用程序，但 Iron.io 平台使得创建和运行各种隔离作业变得轻松和无忧。

# 总结

在本章中，我们采用了两个流行的无服务器平台--AWS 和 Iron.io 的实际操作方法。使用无服务器框架，我们能够快速将我们的代码部署到 AWS Lambda 服务。实际的部署涉及了一些 AWS 服务，将我们的小代码块作为一个 REST API 端点暴露出来，后台调用 AWS Lambda。由于所有服务都由 AWS 管理，我们得到了真正的无服务器体验。如果我们考虑一下，这是一个非常强大的概念。除了 AWS，Iron.io 是另一个有趣的无服务器平台。与 AWS Lambda 上的实时代码执行不同，Iron.io 上的代码执行是作为已计划/排队的任务（并不是说 AWS 没有自己的排队解决方案）。虽然 AWS Lambda 原生支持 Node.js、Java、Python 和.NET Core 运行时，但 Iron.io 通过使用 Docker 容器来抽象语言。尽管如此，我们仍然能够通过 Node.js 来包装 PHP 二进制文件，甚至在 AWS Lambda 上运行 PHP。

无服务器方法确实具有吸引力。虽然它可能不是我们某些应用程序的完整解决方案，但它确实可以处理资源密集型的部分。无需费力的使用和按执行付费的模式对某些人来说可能是一个改变游戏规则的因素。

接下来，我们将看一下 PHP 在流行的响应式编程范式方面提供了什么。
