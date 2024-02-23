# 持续分析和监控

在本章中，我们将学习如何安装和配置分析和监控工具，这将帮助您在**持续集成**（**CI**）和**持续部署**（**CD**）环境中轻松优化 PHP 代码。

我们将从安装和配置基本的`Blackfire.io`设置开始，以便在提交到存储库时轻松自动地对代码进行分析。我们还将学习如何安装 TICK Stack，以便在将代码部署到实时生产服务器后持续监视我们代码的性能。

因此，在本章中，我们将涵盖以下几点：

+   安装和配置`Blackfire.io`代理，客户端和 PHP 扩展

+   将`Blackfire.io`客户端与 Google Chrome 集成

+   将`Blackfire.io`客户端集成到像 Travis 这样的已知 CI 工具

+   安装和配置完整的 TICK Stack 与 Grafana

# 什么是 Blackfire.io？

正如官方 Blackfire 网站所述（[`blackfire.io`](https://blackfire.io)），*Blackfire 赋予所有开发人员和 IT/Ops 持续验证和改进其应用程序性能的能力，通过在适当的时刻获取正确的信息。因此，它是一种性能管理解决方案，允许您在整个应用程序生命周期中自动对代码进行分析，并通过断言设置性能标准，特别是在开发阶段*。`Blackfire.io`是一种工具，使 Fabien Potencier 所说的*性能作为特性*成为可能，通过使性能测试成为项目从一开始就开发周期的一部分。

# 安装和配置 Blackfire.io

安装和配置`Blackfire.io`意味着设置三个组件：代理，客户端和 PHP 探针。在本书的背景下，我们将在 Linux 的 PHP 容器中安装`Blackfire.io`。要获取有关在其他操作系统上安装`Blackfire.io`的更多信息，请参阅以下说明：[`blackfire.io/docs/up-and-running/installation`](https://blackfire.io/docs/up-and-running/installation)。

我们将从安装 Blackfire 代理开始。在容器的命令行界面上，输入以下命令：

```php
# rm /srv/www
# ln -s /srv/fasterweb/chapter_2 /srv/www
# cd /srv/www
# wget -O blackfire-agent https://packages.blackfire.io/binaries/blackfire-agent/1.17.0/blackfire-agent-linux_static_amd64
```

下载完成后，您应该看到以下结果：

![](img/65c9d459-6f78-4931-b408-136785016f85.png)Blackfire 代理下载完成

如果是这样，请继续输入以下命令：

```php
# mv blackfire-agent /usr/local/bin/ 
# chmod +x /usr/local/bin/blackfire-agent 
```

现在，我们将把一个基本的代理配置文件复制到我们的`etc`目录：

```php
# mkdir -p /etc/blackfire 
# cp agent /etc/blackfire/ 
```

这是我们刚刚复制的文件的内容。这是一个基本的配置文件，正如 Blackfire 团队建议的那样：

```php
[blackfire] 
; 
; setting: ca-cert 
; desc   : Sets the PEM encoded certificates 
; default: 
ca-cert= 

; 
; setting: collector 
; desc   : Sets the URL of Blackfire's data collector 
; default: https://blackfire.io 
collector=https://blackfire.io/ 

; 
; setting: log-file 
; desc   : Sets the path of the log file. Use stderr to log to stderr 
; default: stderr 
log-file=stderr 

; 
; setting: log-level 
; desc   : log verbosity level (4: debug, 3: info, 2: warning, 1: error) 
; default: 1 
log-level=1 

; 
; setting: server-id 
; desc   : Sets the server id used to authenticate with Blackfire API 
; default: 
server-id= 

; 
; setting: server-token 
; desc   : Sets the server token used to authenticate with Blackfire 
API. It is unsafe to set this from the command line 
; default: 
server-token= 

; 
; setting: socket 
; desc   : Sets the socket the agent should read traces from. Possible 
value can be a unix socket or a TCP address 
; default: unix:///var/run/blackfire/agent.sock on Linux, 
unix:///usr/local/var/run/blackfire-agent.sock on MacOSX, and 
tcp://127.0.0.1:8307 on Windows. 
socket=unix:///var/run/blackfire/agent.sock 

; 
; setting: spec 
; desc   : Sets the path to the json specifications file 
; default: 
spec= 
```

然后，创建一个空文件，将用作代理的套接字：

```php
# mkdir -p /var/run/blackfire 
# touch /var/run/blackfire/agent.sock 
```

最后，我们将注册我们的代理到 Blackfire 服务：

```php
# blackfire-agent -register 
```

一旦您输入了最后一个命令，您将需要提供您的 Blackfire 服务器凭据。这些可以在您的 Blackfire 帐户中找到：[`blackfire.io/account#server`](https://blackfire.io/account#server)。输入凭据后，您可以通过输入以下命令启动代理：

```php
# blackfire-agent start & 
```

启动代理后，您应该看到代理的 PID 号。这告诉您代理正在监听我们之前创建的默认 UNIX 套接字。在本例中，代理的 PID 号为 8：

![](img/e7dcf06d-e85a-4ffc-8f14-27769d0ba0dc.png)Blackfire 代理进程 ID 号显示

安装和配置代理后，您可以安装 Blackfire 客户端。我们将通过以下命令安装和配置客户端。让我们首先下载二进制文件：

```php
# wget -O blackfire https://packages.blackfire.io/binaries/blackfire-agent/1.17.0/blackfire-cli-linux_static_amd64 
```

下载完成后，您应该看到以下消息：

![](img/81e2991b-5479-46af-8597-b0842fd7c96d.png)Blackfire 客户端下载完成

现在您可以继续配置客户端。输入以下命令：

```php
# mv blackfire /usr/local/bin/ 
# chmod +x /usr/local/bin/blackfire 
# blackfire config 
```

在输入最终命令后，您将需要提供 Blackfire 客户端凭据。这些也可以在以下 URL 的 Blackfire 帐户中找到：[`blackfire.io/account#client`](https://blackfire.io/account#client)。

为了在我们的服务器上运行`Blackfire.io`，最后一步是将 Blackfire 探针安装为 PHP 扩展。为了做到这一点，请首先下载库：

```php
# wget -O blackfire.so https://packages.blackfire.io/binaries/blackfire-php/1.20.0/blackfire-php-linux_amd64-php-71.so
```

下载完成后，您应该会收到以下确认消息：

![](img/dbbc8f29-df0d-45ed-b35a-b66504dc1821.png)Blackfire 探针下载完成

然后，您可以将共享库文件复制到 PHP 扩展目录中。如果您不确定该目录的位置，可以在将库文件移动到该目录之前发出以下命令：

```php
# php -i | grep 'extension_dir' 
# mv blackfire.so $( php -i | grep extensions | awk '{print $3}' )
```

在本例中，扩展的目录是`/usr/lib/php/extensions/no-debug-non-zts-20160303`。

现在可以在`PHP.INI`文件中配置扩展。激活 Blackfire 探针时，建议停用其他调试和分析扩展，如 xdebug。请运行以下命令（或者，您可以复制并粘贴我们存储库中已包含这些修改的`PHP.INI`文件）：

```php
# sed -i 's/zend_extension=\/usr\/lib\/php\/extensions\/no-debug-non-zts-20160303\/xdebug.so/;zend_extension=\/usr\/lib\/php\/extensions\/no-debug-non-zts-20160303\/xdebug.so/' /etc/php.ini
# sed -i 's/^xdebug/;xdebug/' /etc/php.ini
# cat >>/etc/php.ini << 'EOF'

[blackfire]
extension=blackfire.so
; On Windows use the following configuration:
; extension=php_blackfire.dll

; Sets the socket where the agent is listening.
; Possible value can be a unix socket or a TCP address.
; Defaults to unix:///var/run/blackfire/agent.sock on Linux,
; unix:///usr/local/var/run/blackfire-agent.sock on MacOSX,
; and to tcp://127.0.0.1:8307 on Windows.
;blackfire.agent_socket = unix:///var/run/blackfire/agent.sock

blackfire.agent_timeout = 0.25

; Log verbosity level (4: debug, 3: info, 2: warning, 1: error)
;blackfire.log_level = 1

; Log file (STDERR by default)
;blackfire.log_file = /tmp/blackfire.log

;blackfire.server_id =

;blackfire.server_token =
EOF 
```

请通过重新启动 PHP-FPM 来完成扩展的安装和配置：

```php
# /etc/init.d/php-fpm restart 
```

让我们从命令行对我们的第一个脚本进行分析。您现在可以通过在容器的 CLI 上输入以下命令来运行客户端：

```php
# blackfire curl http://localhost/index.php 
```

分析完成后，您将获得一个 URL 和一些分析统计信息。如果浏览到该 URL，您将看到分析的调用图，并获得有关分析脚本的更详细信息：

![](img/61a6dd68-99c9-46e6-8447-9012886ceadb.png)Blackfire 客户端返回一个初步的分析报告和一个 URL，以查看脚本的调用图

您还可以选择将客户端安装为浏览器插件。在本例中，我们将使用 Blackfire Companion，一个 Google Chrome 扩展程序。要安装该扩展，请使用 Chrome 访问以下 URL 并单击安装按钮：[`blackfire.io/docs/integrations/chrome`](https://blackfire.io/docs/integrations/chrome)。安装完成后，可以通过浏览到页面并单击工具栏中的 Blackfire Companion 图标，然后单击 Profile 按钮来对服务器上的资源进行分析：

![](img/6f6c1807-4dee-4cea-bf6e-0f5525a9485c.png)Chrome 的 Blackfire Companion 允许您直接从浏览器对 PHP 脚本进行分析

# 使用 Blackfire.io 手动进行分析

我们将首先手动对两个 PHP 脚本进行分析，以更好地了解 Blackfire 工具的用途和功能。我们将使用以下脚本，可以在我们的存储库（`chap2pre.php`）中找到：

```php
<?php 

function getDiskUsage(string $directory) 
{ 
    $handle = popen("cd $directory && du -ch --exclude='./.*'", 'r'); 

    $du = stream_get_contents($handle); 

    pclose($handle); 

    return $du; 
} 

function getDirList(string $directory, string &$du) 
{ 
    $result = getDiskUsage($directory); 

    $du = empty($du) 
        ? '<br />' . preg_replace('/\n+/', '<br />', $result) 
        : $du; 

    $fileList = []; 

    $iterator = new RecursiveDirectoryIterator($directory, FilesystemIterator::SKIP_DOTS); 

    foreach($iterator as $entry) { 

        if (!$entry->isDir() && $entry->getFilename()[0] != '.') { 
            $fileList[$entry->getFilename()] = 'size is ' . $entry->getSize(); 
        } else { 
            if ($entry->isDir() && $entry->getFilename()[0] != '.') { 
                $fileList[$entry->getFilename()] = getDirList( 
                    $directory . DIRECTORY_SEPARATOR . $entry->getFilename(), 
                    $du 
                );
 }
        } 

    } 

    return $fileList; 
} 

$du = ''; 

$baseDirectory = dirname(__FILE__); 

$fileList = getDirList($baseDirectory, $du); 

echo '<html><head></head><body><p>'; 

echo 'Disk Usage : ' . $du . '<br /><br /><br />'; 

echo 'Directory Name : ' . $baseDirectory . '<br /><br />'; 

echo 'File listing :'; 

echo '</p><pre>'; 

print_r($fileList); 

echo '</pre></body></html>'; 

```

该脚本基本上列出了存储库中包含的所有文件（目录及其子目录），并计算了每个文件的大小。此外，它还给出了每个目录大小的汇总结果。请使用 Chrome 浏览到以下 URL 以查看脚本的输出并使用 Blackfire Companion 启动分析：`http://localhost:8181/chap2pre.php`：

![](img/171e5bab-1f71-40f1-9f7b-125aa6bd05f3.png)单击右上方工具栏中的 Blackfire 图标将允许您启动分析会话

单击 Profile 按钮并等待几秒钟后，您应该可以单击 View Call Graph 按钮：

![](img/f7030d4d-e30b-437b-820e-6c5d3e56b125.png)您可以单击“查看调用图”按钮查看脚本的调用图

结果应该如下：

![](img/55a58fb4-3274-4c85-bd77-17660c01d532.png)该脚本执行完成所需的时间为 14.3 毫秒，并且使用'popen'函数创建了五个进程

结果显示，这个脚本的实际时间（墙时间[1]）为 14.3 毫秒，而唯一具有重要独占时间的函数是`stream_get_contents`和`popen`。这是合理的，因为脚本必须处理磁盘访问和可能大量的 I/O 延迟。不太合理的是，脚本似乎要创建五个子进程来获取一个简单的文件列表。

此外，如果我们向下滚动，我们会注意到`SplInfo::getFilename`被调用了六十七次，几乎是目录中文件数量的两倍：

![](img/6be50866-01b2-444c-8ac4-7ff1716958e1.png)SplFileInfo::getFilename 函数被调用了 67 次

从分析器获得的信息使我们能够快速确定我们代码库的哪些部分应该成为代码审查的候选项，以及在审查它们时要寻找什么。快速查看我们的代码表明，我们在每个目录迭代中都调用了`popen`，而不是只在开始时调用一次。一个简单的修复方法是用以下两行代码替换：

```php
function getDirList(string $directory, string &$du) 
{ 
    $result = getDiskUsage($directory); 

    $du = empty($du) 
        ? '<br />' . preg_replace('/\n+/', '<br />', $result) 
        : $du;  
[...]  
```

然后，以下代码行可以插入到它们的位置：

```php
function getDirList(string $directory, string &$du) 
{ 
    $du = empty($du) 
        ? '<br />' . preg_replace('/\n+/', '<br />', getDiskUsage($directory)) 
        : $du;

[...]
```

最后的调整是用包含函数调用结果的变量替换所有对`SplInfo::getFilename()`的调用。修改后的脚本如下所示：

```php
<?php 

function getDiskUsage(string $directory) 
{ 
    $handle = popen("cd $directory && du -ch --exclude='./.*'", 'r'); 

    $du = stream_get_contents($handle); 

    pclose($handle); 

    return $du; 
} 

function getDirList(string $directory, string &$du) 
{ 
    $du = empty($du) 
        ? '<br />' . preg_replace('/\n+/', '<br />', getDiskUsage($directory)) 
        : $du; 

    $fileList = []; 

    $iterator = new RecursiveDirectoryIterator($directory, FilesystemIterator::SKIP_DOTS); 

    foreach($iterator as $entry) { 

        $fileName = $entry->getFilename(); 

        $dirFlag = $entry->isDir(); 

        if (!$dirFlag && $fileName[0] != '.') { 
            $fileList[$fileName] = 'size is ' . $entry->getSize(); 
        } else { 
            if ($dirFlag && $fileName[0] != '.') { 
                $fileList[$fileName] = getDirList( 
                    $directory . DIRECTORY_SEPARATOR . $fileName, 
                    $du 
                ); 
            } 
        } 

    } 

    return $fileList; 
} 

$du = ''; 

$baseDirectory = dirname(__FILE__); 

$fileList = getDirList($baseDirectory, $du); 

echo '<html><head></head><body><p>'; 

echo 'Disk Usage : ' . $du . '<br /><br /><br />'; 

echo 'Directory Name : ' . $baseDirectory . '<br /><br />'; 

echo 'File listing :'; 

echo '</p><pre>'; 

print_r($fileList); 

echo '</pre></body></html>'; 
```

让我们尝试对新脚本（`chap2post.php`）进行分析，以衡量我们的改进。同样，请使用 Chrome 浏览到以下网址查看脚本的输出，并使用 Blackfire Companion 启动分析：`http://localhost:8181/chap2post.php`。

结果应该如下：

![](img/da3260d9-14de-48b4-b1ec-735d990cd3d2.png)现在，脚本只需要 4.26 毫秒来完成执行，并且只使用'popen'函数创建了一个进程

结果显示，这个脚本现在的墙时间为 4.26 毫秒，而`popen`函数只创建了一个子进程。此外，如果我们向下滚动，我们现在注意到`SplInfo::getFilename`只被调用了三十三次，比之前少了两倍：

![](img/4bcc5dee-e4c1-47c0-b10e-3387d621b989.png)现在，SplFileInfo::getFilename 函数只被调用了 33 次

这些都是重大的改进，特别是如果这个脚本要在不同的目录结构上每分钟被调用数千次。确保这些改进不会在应用程序开发周期的未来迭代中丢失的一个好方法是通过性能测试自动化分析器。现在我们将快速介绍如何使用`Blackfire.io`自动化性能测试。

# 使用 Blackfire.io 进行性能测试

在开始之前，请注意，此功能仅适用于高级和企业用户，因此需要付费订阅。

为了自动化性能测试，我们将首先在我们的存储库中创建一个非常简单的`blackfire.yml`文件。这个文件将包含我们的测试。一个测试应该由一个名称、一个正则表达式和一组断言组成。最好避免创建易变的时间测试，因为这些测试很容易变得非常脆弱，可能会导致从一个分析会话到下一个分析会话产生非常不同的结果。强大的性能测试示例包括检查 CPU 或内存消耗、SQL 查询数量或通过配置比较测试结果。在我们的情况下，我们将创建一个非常基本和易变的时间测试，只是为了举一个简短和简单的例子。以下是我们`.blackfire.yml`文件的内容：

```php
tests: 
    "Pages should be fast enough": 
        path: "/.*" # run the assertions for all HTTP requests 
        assertions: 
            - "main.wall_time < 10ms" # wall clock time is less than 10ms 
```

最后一步是将这个性能测试与持续集成工具集成。要选择您喜欢的工具，请参阅以下网址的文档：[`blackfire.io/docs/integrations/index`](https://blackfire.io/docs/integrations/index)。

在我们的情况下，我们将与*Travis CI*集成。为此，我们必须创建两个文件。一个将包括我们的凭据，并且必须加密（`.blackfire.travis.ini.enc`）。另一个将包括我们的 Travis 指令（`.travis.yml`）。

这是我们的`.blackfire.travis.ini`文件在加密之前的内容（用您自己的凭据替换）：

```php
[blackfire] 

server-id=BLACKFIRE_SERVER_ID 
server-token=BLACKFIRE_SERVER_TOKEN 
client-id=BLACKFIRE_CLIENT_ID 
client-token=BLACKFIRE_CLIENT_TOKEN 
endpoint=https://blackfire.io/ 
collector=https://blackfire.io/ 
```

然后，必须在提交到存储库之前对该文件进行加密。为此，请在 Linux for PHP 容器内部发出以下命令：

```php
# gem install travis
# travis encrypt-file /srv/www/.blackfire.travis.ini -r [your_Github_repository_name_here] 
```

这是我们的`.travis.yml`文件的内容：

```php
language: php 

matrix: 
    include: 
        - php: 5.6 
        - php: 7.0 
          env: BLACKFIRE=on 

sudo: false 

cache: 
    - $HOME/.composer/cache/files 

before_install: 
    - if [[ "$BLACKFIRE" = "on" ]]; then 
        openssl aes-256-cbc -K [ENCRYPT_KEY_HERE] -iv [ENCRYPT_IV_HERE] -in .blackfire.travis.ini.enc -out ~/.blackfire.ini -d 
        curl -L https://blackfire.io/api/v1/releases/agent/linux/amd64 | tar zxpf - 
        chmod 755 agent && ./agent --config=~/.blackfire.ini --socket=unix:///tmp/blackfire.sock & 
      fi 

install: 
    - travis_retry composer install 

before_script: 
    - phpenv config-rm xdebug.ini || true 
    - if [[ "$BLACKFIRE" = "on" ]]; then 
        curl -L https://blackfire.io/api/v1/releases/probe/php/linux/amd64/$(php -r "echo PHP_MAJOR_VERSION . PHP_MINOR_VERSION;")-zts | tar zxpf - 
        echo "extension=$(pwd)/$(ls blackfire-*.so | tr -d '[[:space:]]')" > ~/.phpenv/versions/$(phpenv version-name)/etc/conf.d/blackfire.ini 
        echo "blackfire.agent_socket=unix:///tmp/blackfire.sock" >> ~/.phpenv/versions/$(phpenv version-name)/etc/conf.d/blackfire.ini 
      fi 

script: 
    - phpunit 
```

一旦提交，此配置将确保性能测试将在每次 git 推送到您的 Github 存储库时运行。因此，性能成为一个特性，并且像应用程序的其他任何特性一样持续测试。下一步是在生产服务器上部署代码后监视代码的性能。让我们了解一些可用的工具，以便这样做。

# 使用 TICK 堆栈监控性能

TICK 堆栈是由 InfluxData（*InfluxDB*）开发的，由一系列集成组件组成，允许您轻松处理通过时间生成的不同服务的时间序列数据。TICK 是一个首字母缩写词，由监控套件的每个主要产品的首字母组成。T 代表 Telegraf，它收集我们希望在生产服务器上获取的信息。I 代表 InfluxDB，这是一个包含 Telegraf 或任何其他配置为这样做的应用程序收集的信息的时间序列数据库。C 代表 Chronograf，这是一个图形工具，可以让我们轻松地理解收集的数据。最后，K 代表 Kapacitor，这是一个警报自动化工具。

监控基础设施性能不仅对于确定应用程序和脚本是否按预期运行很重要，而且还可以开发更高级的算法，如故障预测和意外行为模式识别，从而使得可以自动化性能监控的许多方面。

当然，还有许多其他出色的性能监控工具，比如 Prometheus 和 Graphite，但我们决定使用 TICK 堆栈，因为我们更感兴趣的是事件日志记录，而不是纯粹的指标。有关 TICK 堆栈是什么，内部工作原理以及用途的更多信息，请阅读 Gianluca Arbezzano 在 Codeship 网站上发表的这篇非常信息丰富的文章：[`blog.codeship.com/infrastructure-monitoring-with-tick-stack/`](https://blog.codeship.com/infrastructure-monitoring-with-tick-stack/)。

现在，为了查看我们的`Blackfire.io`支持的分析有多有用，以及我们的代码变得更加高效，我们将再次运行这两个脚本，但是这次使用官方 TICK Docker 镜像的副本，以便我们可以监视优化后的 PHP 脚本部署到 Web 服务器上后，Web 服务器的整体性能是否有所改善。我们还将用 Grafana 替换 Chronograf，这是一个高度可定制的图形工具，我们不会设置 Kapacitor，因为配置警报略微超出了我们当前目标的范围。

让我们开始激活 Apache 服务器上的`mod_status`。从我们的 Linux for PHP 的 CLI 中，输入以下命令：

```php
# sed -i 's/#Include \/etc\/httpd\/extra\/httpd-info.conf/Include \/etc\/httpd\/extra\/httpd-info.conf/' /etc/httpd/httpd.conf 
# sed -i 's/Require ip 127/Require ip 172/' /etc/httpd/extra/httpd-info.conf 
# /etc/init.d/httpd restart 
```

完成后，您应该能够通过 Chrome 浏览器浏览以下 URL 来查看服务器的状态报告：`http://localhost:8181/server-status?auto`。

下一步是启动 TICK 套件。请打开两个新的终端窗口以执行此操作。

在第一个终端窗口中，输入此命令：

```php
# docker run -d --name influxdb -p 8086:8086 andrewscaya/influxdb 
```

然后，在第二个新打开的终端窗口中，通过发出此命令获取我们两个容器的 IP 地址：

```php
# docker network inspect bridge 
```

这是我在我的计算机上运行此命令的结果：

![](img/ce8758b7-ec84-4df8-88cd-455cec441844.png)两个容器的 IP 地址

请保留这两个地址，因为配置 Telegraf 和 Grafana 时将需要它们。

现在，我们将使用一个简单的命令生成 Telegraf 的示例配置文件（此步骤是可选的，因为示例文件已经包含在本书的存储库中）。

首先，将目录更改为我们项目的工作目录（Git 存储库），然后输入以下命令：

```php
# docker run --rm andrewscaya/telegraf -sample-config > telegraf.conf 
```

其次，用您喜欢的编辑器打开新文件，并取消注释`inputs.apache`部分中的以下行。不要忘记在`urls`行上输入我们 Linux *for PHP*容器的 IP 地址：

![](img/e8c183a7-d96c-49b5-8d1b-1120d6acac48.png)配置 Telegraf 以监视在另一个容器中运行的 Apache 服务器

在终端窗口中，现在可以使用以下命令启动 Telegraf（请确保您在我们项目的工作目录中）：

```php
# docker run --net=container:influxdb -v ${PWD}/telegraf.conf:/etc/telegraf/telegraf.conf:ro andrewscaya/telegraf
```

在第二个新生成的终端窗口中，使用以下命令启动 Grafana：

```php
# docker run -d --name grafana -p 3000:3000 andrewscaya/grafana
```

使用 Chrome 浏览到`http://localhost:3000/login`。您将看到 Grafana 的登录页面。请使用用户名 admin 和密码 admin 进行身份验证：

![](img/56f9895a-06a5-4188-8875-ac3e1220d599.png)显示 Grafana 登录页面

然后，添加新数据源：

![](img/2a0326db-90f7-4ce5-b350-936ee3e6acfd.png)将 Grafana 连接到数据源

请选择 InfluxDB 数据源的名称。选择 InfluxDB 作为类型。输入 InfluxDB 容器实例的 URL，其中包括您在之前步骤中获得的 IP 地址，后跟 InfluxDB 的默认端口号 8086。您可以选择直接访问。数据库名称是 telegraf，数据库用户和密码是 root:

![](img/efe9ed1a-b23c-411e-bff9-69b844c5a244.png)配置 Grafana 的数据源

最后，单击添加按钮：

![](img/01c2e253-e68f-4b92-9070-5da430a891c8.png)添加数据源

现在数据源已添加，让我们添加一些从 Grafana 网站导入的仪表板。首先点击仪表板菜单项下的导入：

![](img/aaa308c4-dd9a-4ba5-9a9d-0891eac34f73.png)单击导入菜单项开始导入仪表板

我们将添加的两个仪表板如下：

+   Telegraf 主机指标（[`grafana.com/dashboards/1443`](https://grafana.com/dashboards/1443)）：

![](img/f33d00fa-ffd4-457c-8190-0a263d1a5105.png)Telegraf 主机指标仪表板的主页

+   Apache 概览（[`grafana.com/dashboards/331`](https://grafana.com/dashboards/331)）：

![](img/2356dfeb-5bca-458a-aba8-8bd40c185f83.png)Apache 概览仪表板的主页

在导入屏幕上，只需输入仪表板的编号，然后单击加载：

![](img/5f6f1fe5-7540-4412-a155-4989ae0098a7.png)加载 Telegraf 主机指标仪表板

然后，确认新仪表板的名称并选择我们的本地 InfluxDB 连接：

![](img/96777062-1a40-4a25-aba2-3e1101f5239d.png)将 Telegraf 主机指标仪表板连接到 InfluxDB 数据源

现在您应该看到新的仪表板：

![](img/d04fec9b-f9c4-4ddb-9df5-8d6c708a9156.png)显示 Telegraf 主机指标仪表板

现在，我们将重复最后两个步骤，以导入 Apache 概览仪表板。单击仪表板菜单项下的导入按钮后，输入仪表板的标识符（`331`），然后单击加载按钮：

![](img/e0c0faa6-3265-4e1d-ae5a-2afac962ffdc.png)加载 Apache 概览仪表板

然后，确认名称并选择我们的本地 InfluxDB 数据源：

![](img/a207e4f0-7695-4895-856e-3a6d5a821eec.png)将 Apache 概览仪表板连接到 InfluxDB 数据源

现在您应该在浏览器中看到第二个仪表板：

![](img/bf9d26a3-d85b-4964-a48c-3a462ffb0c93.png)显示 Apache 概览仪表板

所有 TICK 套件仪表板都允许更高级的图形配置和自定义。因此，通过执行自定义 cron 脚本，可以收集一组自定义的时间序列数据点，然后配置仪表板以按您的要求显示这些数据。

在我们当前的例子中，TICK 套件现在已安装和配置。因此，我们可以开始测试和监视使用`Blackfire.io`在本章第一部分中进行优化的 PHP 脚本，以测量其性能的变化。我们将首先部署、进行基准测试和监视旧版本。在 Linux 上的 PHP CLI 中，输入以下命令以对旧版本的脚本进行基准测试：

```php
# siege -b -c 3000 -r 100 localhost/chap2pre.php 
```

基准测试应该产生类似以下结果：

![](img/845dd1b4-a034-4c57-bfe1-6dd3fda39694.png)显示了原始脚本的性能基准测试结果

然后，等待大约十分钟后，通过输入以下命令开始对新版本的脚本进行基准测试：

```php
# siege -b -c 3000 -r 100 localhost/chap2post.php 
```

这是我电脑上最新基准测试的结果：

![](img/23505778-9583-41ba-be5d-b6a440b2c7f0.png)显示了优化脚本的性能基准测试结果

结果已经显示出性能上的显著改善。事实上，新脚本每秒允许的交易数量是原来的三倍多，失败交易的数量也减少了三分之一以上。

现在，让我们看看我们的 TICK Stack 收集了关于这两个版本的 PHP 脚本性能的哪些数据：

![](img/148e9e94-8227-4989-8ddb-1de83c5f8a0b.png)监控图表中清楚地显示了性能的提升

我们 Grafana 仪表板中的图表清楚地显示了与基准测试结果本身相同数量级的性能提升。在 08:00 之后对新版本脚本进行的基准测试明显使服务器负载减少了一半，输入（I/O）减少了一半以上，并且总体上比之前在 7:40 左右进行基准测试的旧版本快了三倍以上。因此，毫无疑问，我们的`Blackfire.io`优化使得新版本的 PHP 脚本更加高效。

# 总结

在本章中，我们学习了如何安装和配置基本的`Blackfire.io`设置，以便在提交到存储库时轻松自动地对代码进行分析。我们还探讨了如何安装 TICK Stack，以便在将代码部署到实时生产服务器后持续监视其性能。因此，我们已经了解了如何安装和配置分析和监视工具，这些工具可以帮助我们在**持续集成**（**CI**）和**持续部署**（**CD**）环境中轻松优化 PHP 代码。

在下一章中，我们将探讨如何更好地理解 PHP 数据结构并使用简化的函数可以帮助应用程序在其关键执行路径上的全局性能。我们将首先分析项目的关键路径，然后微调其某些数据结构和函数。

# 参考资料

[1] 关于这些性能测试术语的进一步解释，请访问以下网址：[`blackfire.io/docs/reference-guide/time`](https://blackfire.io/docs/reference-guide/time)。
