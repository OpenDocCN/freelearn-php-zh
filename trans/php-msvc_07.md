# 第七章 安全

当我们开发应用程序时，我们应该始终考虑如何使我们的微服务更加安全。有一些技术和方法，每个开发人员都应该了解，以避免安全问题。在本章中，您将发现如何在您的微服务中使用身份验证和授权，以及在用户登录后如何管理每个功能的权限。您还将发现可以用来加密数据的不同方法。

# 微服务中的加密

我们可以将加密定义为将信息转换为只有授权方能够阅读的过程。这个过程实际上可以在您的应用程序的任何级别进行。例如，您可以加密整个数据库，或者您可以在传输层使用 SSL/TSL 或**JSON Web Token**（**JWT**）进行加密。

如今，加密/解密过程是通过现代算法完成的，加密添加的最高级别是在传输层。在这一层中使用的所有算法至少提供以下功能：

+   **认证**：此功能允许您验证消息的来源

+   **完整性**：此功能可为您提供消息内容在从原始内容到目的地的过程中未更改的证据

加密算法的最终任务是为您提供一个安全层，以便您可以在不必担心有人窃取您的信息的情况下交换或存储敏感数据，但这并非免费。您的环境将使用一些资源来处理加密、解密或其他相关事项中的握手。

作为开发人员，您需要考虑到您将被部署到一个敌对的环境——生产环境是一个战区。如果您开始这样思考，您可能会开始问自己以下问题：

+   我们将部署到硬件还是虚拟化环境？我们将共享资源吗？

+   我们能相信我们应用程序的所有可能的邻居吗？

+   我们将把我们的应用程序分割成不同的和分离的区域吗？我们将如何连接我们的区域？

+   我们的应用程序是否符合 PCI 标准，或者由于我们存储/管理的数据，它是否需要非常高的安全级别？

当您开始回答所有这些问题（以及其他问题）时，您将开始确定应用程序所需的安全级别。

在本节中，我们将向您展示加密应用程序数据的最常见方法，以便您可以随后选择要实施的方法。

请注意，我们不考虑全盘加密，因为它被认为是保护数据的最弱方法。

## 数据库加密

当您处理敏感数据时，保护数据的最灵活且开销最低的方法是在应用程序层中使用加密。然而，如果由于某种原因您无法更改您的应用程序，接下来最强大的解决方案是加密您的数据库。

对于我们的应用程序，我们选择了*关系数据库*；具体来说，我们使用的是 Percona，一个 MySQL 分支。目前，您在该数据库中有两种不同的加密数据的选项：

+   通过 MariaDB 补丁启用加密（另一个与 Percona 非常相似的 MySQL 形式）。此补丁在 10.1.3 及更高版本中可用。

+   InnoDB 表空间级加密方法可从 Percona Server 5.7.11 或 MySQL 5.7.11 开始使用。

也许您想知道为什么我们在选择了 Percona 后还在谈论 MariaDB 和 MySQL。这是因为它们三者具有相同的核心，共享大部分核心功能。

### 提示

所有主要的数据库软件都允许您加密数据。如果您没有使用 Percona，请查看您的数据库的官方文档，找到允许加密所需的步骤。

作为开发人员，你需要了解在应用中使用数据库级加密的弱点。除其他外，我们可以强调以下几点：

+   特权数据库用户可以访问密钥环文件，因此在你的数据库中要严格控制用户权限。

+   数据在存储在服务器的 RAM 中时并不加密，只有在数据写入硬盘时才会加密。一个特权且恶意的用户可以使用一些工具来读取服务器内存，因此也可以读取你的应用数据。

+   一些工具，比如 GDB，可以用来更改根用户密码结构，从而允许你无任何问题地复制数据。

### MariaDB 中的加密

想象一下，如果你不想使用 Percona，而是想使用 MariaDB；由于`file_key_management`插件，数据库加密是可用的。在我们的应用示例中，我们正在使用 Percona 作为 secrets 微服务的数据存储，所以让我们添加一个新的 MariaDB 容器，以便以后尝试并交换这两个 RDBMS。

首先，在与数据库文件夹处于同一级别的 secrets 微服务内的 Docker 存储库中创建一个`mariadb`文件夹。在这里，你可以添加一个包含以下内容的`Dockerfile`：

```php
**FROM mariadb:latest

RUN apt-get update \
&& apt-get autoremove && apt-get autoclean \
&& rm -rf /var/lib/apt/lists/*

RUN mkdir -p /volumes/keys/
RUN echo 
"1;
C472621BA1708682BEDC9816D677A4FDC51456B78659F183555A9A895EAC9218" > 
/volumes/keys/keys.txt
RUN openssl enc -aes-256-cbc -md sha1 -k secret -in 
/volumes/keys/keys.txt -out /volumes/keys/keys.enc
COPY etc/ /etc/mysql/conf.d/**

```

在上述代码中，我们正在拉取最新的官方 MariaDB 镜像，更新它，并创建一些我们加密需要的证书。在`keys.txt`文件中保存的长字符串是我们自己生成的密钥，使用以下命令生成：

```php
**openssl enc -aes-256-ctr -k secret@phpmicroservices.com -P -md sha1**

```

我们`Dockerfile`的最后一个命令将我们定制的数据库配置复制到容器内。在`etc/encryption.cnf`中创建我们的自定义数据库配置，内容如下：

```php
    [mysqld]
    plugin-load-add=file_key_management.so
    file_key_management_filekey = FILE:/mount/keys/server-key.pem
    file-key-management-filename = /mount/keys/mysql.enc
    innodb-encrypt-tables = ON
    innodb-encrypt-log = 1
    innodb-encryption-threads=1
    encrypt-tmp-disk-tables=1
    encrypt-tmp-files=0
    encrypt-binlog=1
    file_key_management_encryption_algorithm = AES_CTR
```

在上述代码中，我们告诉我们的数据库引擎我们存储证书的位置，并启用了加密。现在，你可以编辑我们的`docker-compose.yml`文件，并添加以下容器定义：

```php
    microservice_secret_database_mariadb:
      build: ./microservices/secret/mariadb/
      environment:
        - MYSQL_ROOT_PASSWORD=mysecret
        - MYSQL_DATABASE=finding_secrets
        - MYSQL_USER=secret
        - MYSQL_PASSWORD=mysecret
      ports:
        - 7777:3306
```

从上述代码中可以看出，我们并没有定义任何新的内容；你现在可能已经有足够的 Docker 经验来理解我们正在定义`Dockerfile`的位置。我们设置了一些环境变量，并将本地的`7777`端口映射到容器的`3306`端口。一旦你做出所有的更改，一个简单的`docker-compose build microservice_secret_database`命令将生成新的容器。

构建完容器后，是时候检查一切是否正常运行了。使用`docker-compose up microservice_secret_database`启动新容器，并尝试将其连接到我们本地的`7777`端口。现在，你可以开始在这个容器中使用加密。考虑以下示例：

```php
    CREATE TABLE `test_encryption` (
      `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
      `text_field` varchar(255) NOT NULL,
      PRIMARY KEY (`id`)
    ) ENGINE=InnoDB `ENCRYPTED`=YES `ENCRYPTION_KEY_ID`=1;
```

在上述代码中，我们为我们的 SQL 添加了一些额外的标签；它们启用了表中的加密，并使用我们在`keys.txt`中存储的 ID 为`1`的加密密钥（我们用它来启动数据库的文件）。试一试，如果一切顺利，随时可以进行必要的更改，使用这个新的数据库容器代替我们项目中的另一个容器。

### InnoDB 加密

Percona 和 MySQL 5.7.11+版本自带一个新功能--支持**InnoDB**表空间级加密。有了这个功能，你可以在不需要太多麻烦或配置的情况下加密所有的 InnoDB 表。在我们的示例应用中，我们正在使用 Percona 5.7 来处理 secrets 微服务。让我们看看如何加密我们的表。

首先，我们需要对我们的 Docker 环境进行一些小的修改；首先，打开`microservices/secret/database/Dockerfile`，并用以下代码替换所有内容：

```php
    FROM percona:5.7
 **RUN mkdir -p /mount/mysql-keyring/ \**
 **&& touch /mount/mysql-keyring/keyring \**
 **&& chown -R mysql:mysql /mount/mysql-keyring**
**COPY etc/ /etc/mysql/conf.d/**

```

在本书的这一部分，你可能不需要解释我们在`Dockerfile`中做了什么，所以让我们创建一个新的`config`文件，稍后将其复制到我们的容器中。在`secret microservice`文件夹中，创建一个`etc`文件夹，并生成一个名为`encryption.cnf`的新文件，内容如下：

```php
    [mysqld]
    early-plugin-load=keyring_file.so
    keyring_file_data=/mount/mysql-keyring/keyring
```

在我们之前创建的配置文件中，我们正在加载`keyring`库，数据库可以在其中找到并存储用于加密数据的生成密钥环。

此时，您已经拥有了启用加密所需的一切，因此使用`docker-compose build microservice_secret_database`重新构建容器，并使用`docker-compose up -d`再次启动所有容器。

如果一切正常，您应该能够无问题地打开数据库，并且可以使用以下 SQL 命令更改我们存储的表：

```php
**ALTER TABLE `secrets` ENCRYPTION='Y'**

```

也许您会想知道为什么我们修改了`secrets`表，如果我们已经在数据库中启用了加密。背后的原因是因为加密不是默认启用的，因此您需要明确告诉引擎您想要加密哪些表。

### 性能开销

在数据库中使用加密将降低应用程序的性能。您的机器/容器将使用一些资源来处理加密/解密过程。在某些测试中，当您不使用表空间级加密（MySQL/Percona +5.7）时，这种开销可能超过 20%。我们建议您测量启用和未启用加密时应用程序的平均性能。这样，您可以确保加密不会对应用程序产生很大影响。

在本节中，我们向您展示了两种快速增加应用程序安全性的方法。使用这些功能的最终决定取决于您和您的应用程序的规格。

## TSL/SSL 协议

**传输层安全**（**TSL**）和**安全套接字层**（**SSL**）是用于在不受信任的网络中保护通信的加密协议，例如，互联网或 ISP 的局域网。 SSL 是 TSL 的前身，它们两者经常可以互换使用或与 TLS/SSL 一起使用。如今，SSL 和 TSL 实际上是一回事，如果您选择使用其中之一，选择另一个没有区别，您将使用服务器规定的相同级别的加密。例如，如果应用程序（例如电子邮件客户端）让您在 SSL 或 TSL 之间进行选择，您只是选择了安全连接的启动方式，没有其他区别。

这些协议的所有功能和安全性都依赖于我们所知的证书。TSL/SSL 证书可以定义为将加密密钥与组织或个人的详细信息数字绑定的小型数据文件。您可以找到各种公司出售 TSL/SSL 证书，但如果您不想花钱（或者您处于开发阶段），您可以创建自签名证书。这些类型的证书可用于加密数据，但客户端将不信任它们，除非您跳过验证。

### TSL/SSL 协议的工作原理

在您开始在应用程序中使用 TSL/SSL 之前，您需要了解它的工作原理。还有许多其他专门解释这些协议工作原理的书籍，因此我们只会给您一个初步了解。

以下图表总结了 TSL/SSL 协议的工作原理；首先，您需要知道 TSL/SSL 是一个 TCP 客户端-服务器协议，加密在经过几个步骤后开始：

![TSL/SSL 协议的工作原理](img/B06142_07_01.jpg)

TSL/SSL 协议

TSL/SSL 协议的步骤如下：

1.  我们的客户希望与使用 TSL/SSL 保护的服务器/服务建立连接，因此它要求服务器进行身份验证。

1.  服务器响应请求并向客户端发送其 TSL/SSL 证书的副本。

1.  客户端检查 TSL/SSL 证书是否是受信任的，如果是，则向服务器发送消息。

1.  服务器返回数字签名的确认以开始会话。

1.  在所有先前的步骤（握手）之后，加密数据在客户端和服务器之间共享。

正如你所能想象的，术语*客户端*和*服务器*是模棱两可的；客户端可以是试图访问你的页面的浏览器，也可以是试图与另一个微服务通信的微服务。

## TSL/SSL 终止

正如你之前学到的，为你的应用添加 TSL/SSL 层会给应用的整体性能增加一些开销。为了缓解这个问题，我们有所谓的 TSL/SSL 终止，一种 TSL/SSL 卸载形式，它将加密/解密的责任从服务器转移到应用的不同部分。

TSL/SSL 终止依赖于这样一个事实，即一旦所有数据被解密，你就信任你正在使用的所有通信渠道来传输这些解密后的数据。让我们以一个微服务为例；看一下下面的图片：

![TSL/SSL 终止](img/B06142_07_02.jpg)

微服务中的 TSL/SSL 终止

在上述图片中，所有的输入/输出通信都是使用我们微服务架构的特定组件加密的。这个组件将充当代理，并处理所有的 TSL/SSL 事务。一旦来自客户端的请求到来，它就会处理所有的握手并解密请求。一旦请求被解密，它就会被代理到特定的微服务组件（在我们的案例中，它是 NGINX），我们的微服务就会执行所需的操作，例如从数据库中获取一些数据。一旦微服务需要返回响应，它就会使用代理，其中我们所有的响应都是加密的。如果你有多个微服务，你可以扩展这个小例子并做同样的事情--加密不同微服务之间的所有通信，并在微服务内部使用加密数据。

## 使用 NGINX 进行 TSL/SSL

你可以找到多个软件，可以用来进行 TSL/SSL 终止。以下列出了一些最知名的：

+   **负载均衡器**：Amazon ELB 和 HaProxy

+   **代理**：NGINX、Traefik 和 Fabio

在我们的案例中，我们将使用 NGINX 来管理所有的 TSL/SSL 终止，但是请随意尝试其他选项。

你可能已经知道，NGINX 是市场上最多才多艺的软件之一。你可以将其用作反向代理或具有高性能水平和稳定性的 Web 服务器。

我们将解释如何在 NGINX 中进行 TSL/SSL 终止，例如对于 battle 微服务。首先，打开`microservices/battle/nginx/Dockerfile`文件，并在 CMD 命令之前添加以下命令：

```php
**RUN echo 01 > ca.srl \
&& openssl genrsa -out ca-key.pem 2048 \
&& openssl req -new -x509 -days 365 -subj "/CN=*" -key ca-key.pem -out ca.pem \
&& openssl genrsa -out server-key.pem 2048 \
&& openssl req -subj "/CN=*" -new -key server-key.pem -out server.csr \
&& openssl x509 -req -days 365 -in server.csr -CA ca.pem -CAkey ca-key.pem -out server-cert.pem \
&& openssl rsa -in server-key.pem -out server-key.pem \
&& cp *.pem /etc/nginx/ \
&& cp *.csr /etc/nginx/**

```

在这里，我们创建了一些自签名证书，并将它们存储在`nginx`容器的`/etc/nginx`文件夹中。

一旦我们有了证书，就是时候改变 NGINX 配置文件了。打开`microservices/battle/nginx/config/nginx/nginx.conf.ctmpl`文件，并添加以下服务器定义：

```php
    server {
      listen 443 ssl;
      server_name _;
      root /var/www/html/public;
      index index.php index.html;
      ssl on;
      ssl_certificate /etc/nginx/server-cert.pem;
      ssl_certificate_key /etc/nginx/server-key.pem;
      location = /favicon.ico { access_log off; log_not_found off; }
      location = /robots.txt { access_log off; log_not_found off; }
      access_log /var/log/nginx/access.log;
      error_log /var/log/nginx/error.log error;
      sendfile off;
      client_max_body_size 100m;
      location / {
        try_files $uri $uri/ /index.php?_url=$uri&$args;
      }
      location ~ /\.ht {
        deny all;
      }
      {{ if service $backend }}
      location ~ \.php$ {
        try_files $uri =404;
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass {{ $backend }};
        fastcgi_index /index.php;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME 
        $document_root$fastcgi_script_name;
        fastcgi_intercept_errors off;
        fastcgi_buffer_size 16k;
        fastcgi_buffers 4 16k;
      }
      {{ end }}
    }
```

上述代码片段在`nginx`服务器中设置了一个新的监听器，在`443`端口。正如你所看到的，它与默认服务器设置非常相似；不同之处在于端口和我们在上一步中创建的证书的位置。

为了使用这个新的 TSL/SSL 端点，我们需要对`docker-compose.yml`文件进行一些小的更改，并映射`443` NGINX 端口。你只需要去`microservice_battle_nginx`定义中添加一个新的端口声明行，如下所示：

```php
    - 8443:443
```

新的行将我们的`8443`端口映射到`nginx`容器的`443`端口，允许我们通过 TSL/SSL 连接。你现在可以用 Postman 试一试，但是由于它是一个自签名证书，默认情况下是不被接受的。打开**首选项**并禁用**SSL 证书验证**。作业时，你可以将我们所有的示例服务都改为只使用 TSL/SSL 层来相互通信。

在本章的这一部分，我们向您展示了如何为您的应用程序添加额外的安全层，加密数据和用于交换消息的通信渠道。现在我们确信我们的应用程序至少具有一定程度的加密，让我们继续讨论应用程序的另一个重要方面--认证。

# 认证

每个项目的起点是认证系统，通过它可以识别将使用我们的应用程序或 API 的用户或客户。有许多库可以实现不同的用户认证方式；在本书中，我们将看到两种最重要的方式：**OAuth 2**和**JWT**。

正如我们已经知道的，微服务是*无状态*的，这意味着它们应该使用*访问令牌*而不是 cookie 和会话与彼此和用户进行通信。因此，让我们看看使用它进行认证的工作流程是什么样的：

![认证](img/B06142_07_03-1.jpg)

通过令牌进行认证的工作流程

正如您在上图中所看到的，这应该是获取客户或用户所需的秘密列表的过程：

1.  **USER**向**FRONTEND LOGIN**请求秘密列表。

1.  **FRONTEND LOGIN**向**BACKEND**请求秘密列表。

1.  **BACKEND**向**FRONTEND LOGIN**请求用户访问令牌。

1.  **FRONTEND LOGIN**向**GOOGLE**（或任何其他提供者）请求访问令牌。

1.  **GOOGLE**向**USER**请求他们的凭据。

1.  **USER**向**GOOGLE**提供凭据。

1.  **GOOGLE**向**FRONTEND LOGIN**提供用户访问令牌。

1.  **FRONTEND LOGIN**提供**BACKEND**用户访问令牌。

1.  **BACKEND**与**GOOGLE**检查使用该访问令牌的用户是谁。

1.  **GOOGLE**告诉**BACKEND**用户是谁。

1.  **BACKEND**检查用户并告诉**FRONTEND LOGIN**秘密列表。

1.  **FRONTEND LOGIN**向**USER**显示秘密列表。

显然，在这个过程中，一切都是在用户不知情的情况下发生的。用户只需要向适当的服务提供他/她的凭据。在前面的例子中，服务是**GOOGLE**，但它甚至可以是我们自己的应用程序。

现在，我们将构建一个新的 docker 容器，以便使用 OAuth 2 和 JWT 来创建和设置一个用于认证用户的数据库。

在 docker 用户微服务的`docker/microservices/user/database/Dockerfile`数据库文件夹下创建一个`Dockerfile`，并添加以下行。我们将像我们为 secret 微服务所做的那样使用 Percona：

```php
    FROM percona:5.7
```

创建了`Dockerfile`之后，打开`docker-composer.yml`文件，并在用户微服务部分的末尾添加用户数据库微服务配置（就在源容器之前）。还要将`microservice_user_database`添加到`microservice_user_fpm`链接部分，以使数据库可见：

```php
    microservice_user_fpm:
    {{omitted code}}
    links:
    {{omitted code}}
 **- microservice_user_database**
 **microservice_user_database:**
 **build: ./microservices/user/database/**
 **environment:**
 **- CONSUL=autodiscovery**
 **- MYSQL_ROOT_PASSWORD=mysecret**
 **- MYSQL_DATABASE=finding_users**
 **- MYSQL_USER=secret**
 **- MYSQL_PASSWORD=mysecret**
 **ports:**
 **- 6667:3306**

```

一旦我们设置了配置，就该构建它了，所以在您的终端上运行以下命令来创建我们刚刚设置的新容器：

```php
**docker-compose build microservice_user_database**

```

这可能需要一些时间；当它完成时，我们必须通过运行以下命令再次启动容器：

```php
**docker-compose up -d**

```

您可以通过执行`docker ps`来检查用户数据库微服务是否正确创建，因此请检查其中的新`microservice_user_database`。

现在是时候设置用户微服务以便能够使用我们刚刚创建的数据库容器了，所以将以下行添加到`bootstrap/app.php`文件中：

```php
    $app->configure('database');
```

还要创建`config/database.php`文件，并添加以下配置：

```php
    <?php
      return [
        'default'     => 'mysql',
        'migrations'  => 'migrations',
        'fetch'       => PDO::FETCH_CLASS,
        'connections' => [
          'mysql' => [
            'driver'    => 'mysql',
            'host'      => env('DB_HOST','microservice_user_database'),
            'database'  => env('DB_DATABASE','finding_users'),
            'username'  => env('DB_USERNAME','secret'),
            'password'  => env('DB_PASSWORD','mysecret'),
            'collation' => 'utf8_unicode_ci',
          ]
        ]
      ];
```

请注意，在上述代码中，我们使用了与`docker-compose.yml`文件中用于连接到数据库容器的相同凭据。

就是这样。我们现在有一个新的数据库容器连接到用户微服务，它已经准备好使用了。通过创建迁移或在您喜爱的 SQL 客户端中执行以下查询来添加一个新的用户表：

```php
    CREATE TABLE `users` (
      `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
      `email` varchar(255) NOT NULL,
      `password` varchar(255) NOT NULL,
      `api_token` varchar(255) DEFAULT NULL,
      PRIMARY KEY (`id`)
    ) ENGINE=InnoDB AUTO_INCREMENT=3 DEFAULT CHARSET=latin1;
```

## OAuth 2

让我们介绍一种安全且特别适用于微服务的基于访问令牌的认证系统。

OAuth 2 是一种标准协议，允许我们将 API REST 的某些方法限制为特定用户，而无需要求用户提供用户名和密码。

这个协议非常常见，因为它更安全，可以避免在 API 之间通信时共享密码或敏感凭据。

OAuth 2 使用访问令牌，用户需要获取该令牌才能使用应用程序。令牌将具有过期时间，并且可以在无需再次提供用户凭据的情况下进行刷新。

### 如何在 Lumen 上使用 OAuth 2

现在，我们将解释如何在 Lumen 上安装、设置和尝试 OAuth 2 身份验证。这样做的目的是让微服务使用 OAuth 2 来限制方法；换句话说，在使用需要身份验证的方法之前，消费者需要提供一个令牌。

#### OAuth 2 安装

通过在 docker 文件夹上执行以下命令进入用户微服务：

```php
**docker-compose up -d
docker exec -it  docker_microservice_user_fpm_1 /bin/bash**

```

一旦我们进入用户微服务，就需要通过在`composer.json`文件的`require`部分中添加以下行来安装 OAuth 2：

```php
    "lucadegasperi/oauth2-server-laravel": "⁵.0"
```

然后，执行`composer update`，该包将在您的微服务上安装 OAuth 2。

#### 设置

安装完包后，我们必须设置一些重要的东西才能运行 OAuth 2。首先，我们需要将位于`/vendor/lucadegasperi/oauth2-server-laravel/config/oauth2.php`的 OAuth 2 配置文件复制到`/config/oauth2.php`；如果`config`文件夹不存在，则创建它。此外，我们需要将包含在`/vendor/lucadegasperi/oauth2-server-laravel/database/migrations`中的迁移文件复制到`/database/migrations`文件夹中。

不要忘记通过在`/bootstrap/app.php`中添加以下行来注册 OAuth 2：

```php
    $app-
    >register(\LucaDegasperi\OAuth2Server\Storage\
    FluentStorageServiceProvider::class);
    $app-    >register(\LucaDegasperi\OAuth2Server\
    OAuth2ServerServiceProvider::class);
    $app->middleware([
      \LucaDegasperi\OAuth2Server\Middleware\
      OAuthExceptionHandlerMiddleware::class
    ]);
```

在`app->withFacades();`行之前的文件顶部（如果没有取消注释，请这样做），添加以下行：

```php
    class_alias('Illuminate\Support\Facades\Config', 'Config');
    class_alias(\LucaDegasperi\OAuth2Server\Facades\Authorizer::class, 
    'Authorizer');
```

现在，我们将执行迁移以在数据库中创建必要的表：

```php
**composer dumpautoload
php artisan migrate**

```

### 提示

如果在执行迁移时遇到问题，请尝试将`'migrations' => 'migrations', 'fetch' => PDO::FETCH_CLASS,`添加到`config/database.php`文件中，然后执行`php artisan migrate:install --database=mysql`。

一旦我们创建了所有必要的表，可以使用 Lumen seeders 在`oauth_clients`表中插入一个注册，或者通过在您喜爱的 SQL 客户端上执行以下查询来执行：

```php
    INSERT INTO `finding_users`.`oauth_clients`
    (`id`, `secret`, `name`, `created_at`, `updated_at`)
    VALUES
    ('1', 'YouAreTheBestDeveloper007', 'PHPMICROSERVICES', NULL, NULL);
```

现在，我们必须在`/app/Http/routes.php`中添加一个新路由，以便为我们刚刚创建的用户获取有效的令牌。例如，路由可以是`oauth/access_token`：

```php
    $app->post('**oauth/access_token**', function() {
      return response()->json(Authorizer::issueAccessToken());
    });
```

最后，修改`/config/oauth2.php`文件中的`grant_types`值，将其更改为以下代码行：

```php
    'grant_types' => [
      'client_credentials' => [
        'class'            => '\League\OAuth2\Server\Grant\
        ClientCredentialsGrant',
        'access_token_ttl' => 0
      ]
    ],
```

#### 让我们尝试 OAuth2

现在，我们可以通过在 Postman 上对`http://localhost:8084/api/v1/oauth/access_token`进行 POST 调用来获取我们的令牌，包括在 body 中包含以下参数：

```php
 **grant_type:** client_credentials
 **client_id:** 1
 **client_secret:** YouAreTheBestDeveloper007
```

如果输入错误的凭据，将会得到以下响应：

```php
    {
      "error": "invalid_client",
      "error_description": "Client authentication failed."
    }
```

如果凭据正确，我们将在 JSON 中获得`access_token`：

```php
    {
      "access_token": "**anU2e6xgXiLm7UARSSV7M4Wa7u86k4JryKWrIQhu**",
      "token_type": "Bearer",
      "expires_in": 3600
    }
```

一旦我们获得有效的访问令牌，我们可以限制一些未注册用户的方法。这在 Lumen 上非常容易。我们只需在`/bootstrap/app.php`上启用路由中间件，因此在该文件中添加以下代码：

```php
    $app->routeMiddleware(
      [
        'check-authorization-params' => 
        \LucaDegasperi\OAuth2Server\Middleware\
        CheckAuthCodeRequestMiddleware::class,
        'csrf' => \Laravel\Lumen\Http\Middleware\
        VerifyCsrfToken::class,
        'oauth' => 
        \LucaDegasperi\OAuth2Server\Middleware\
        OAuthMiddleware::class,
        'oauth-client' => \LucaDegasperi\OAuth2Server\Middleware\
        OAuthClientOwnerMiddleware::class,
        'oauth-user' => \LucaDegasperi\OAuth2Server\Middleware\
        OAuthUserOwnerMiddleware::class,
      ]
    );
```

转到`UserController.php`文件并添加一个带有以下代码的`__construct()`函数：

```php
    public function __construct(){
      $this->middleware('oauth');
    }
```

这将影响控制器上的所有方法，但我们可以使用以下代码排除其中一些方法：

```php
    public function __construct(){
      $this->middleware('oauth', **['except' => 'index']**);
    }
    public function index()
    {
      return response()->json(['method' => 'index']);
    }
```

现在，我们可以通过在`http://localhost:8084/api/v1/user`上进行 GET 调用来测试 index 函数。不要忘记在`Authorization`标头中包含`Bearer anU2e6xgXiLm7UARSSV7M4Wa7u86k4JryKWrIQhu`值。

如果我们排除了 index 函数，或者如果我们正确输入了令牌，我们将获得状态码 200 的 JSON 响应：

```php
    {"method":"index"}
```

如果我们没有排除 index 方法并输入了错误的令牌，我们将收到错误代码 401 和以下消息：

```php
    {"error":"access_denied","error_description":"The resource owner or 
    authorization server denied the request."}
```

现在您有一个安全且更好的应用程序。请记住，您可以将上一章中学到的错误处理添加到您的授权方法中。

## JSON Web Token

**JSON Web Token**（**JWT**）是一组安全方法，用于 HTTP 请求和客户端与服务器之间的传输。JWT 令牌是使用 JSON Web 签名进行数字签名的 JSON 对象。

为了使用 JWT 创建令牌，我们需要用户凭据、秘密密钥和要使用的加密类型；可以是 HS256、HS384 或 HS512。

### 如何在 Lumen 上使用 JWT

可以使用 composer 在 Lumen 上安装 JWT。因此，一旦您在用户微服务容器中，就在终端中执行以下命令：

```php
**composer require tymon/jwt-auth:"¹.0@dev"**

```

安装该库的另一种方法是打开您的`composer.json`文件，并将`"tymon/jwt-auth": "¹.0@dev"`添加到所需库列表中。安装后，我们需要像在 OAuth 2 中注册服务提供程序一样在注册服务提供程序上注册 JWT。在 Lumen 上，可以通过在`bootstrap/app.php`文件中添加以下行来实现：

```php
    $app->register('Tymon\JWTAuth\Providers\JWTAuthServiceProvider');
```

还要取消以下行的注释：

```php
    $app->register(App\Providers\AuthServiceProvider::class);
```

您的`bootstrap/app.php`文件应如下所示：

```php
    <?php
      require_once __DIR__.'/../vendor/autoload.php';
      try {
        (new Dotenv\Dotenv(__DIR__.'/../'))->load();
      } catch (Dotenv\Exception\InvalidPathException $e) {
        //
      }
      $app = new Laravel\Lumen\Application(
        realpath(__DIR__.'/../')
      );
      // $app->withFacades();
 **$app->withEloquent();**
      $app->singleton(
        Illuminate\Contracts\Debug\ExceptionHandler::class,
        App\Exceptions\Handler::class
      );
      $app->singleton(
        Illuminate\Contracts\Console\Kernel::class,
        App\Console\Kernel::class
      );
 **$app->routeMiddleware([**
 **'auth' => App\Http\Middleware\Authenticate::class,**
 **]);**
      $app->register(App\Providers\AuthServiceProvider::class);
 **$app->register
      (Tymon\JWTAuth\Providers\LumenServiceProvider::class);**
      $app->group(['namespace' => 'App\Http\Controllers'], 
      function ($app) 
      {
           require __DIR__.'/../app/Http/routes.php';
      });
      return $app;
```

#### 设置 JWT

现在我们需要一个秘密密钥，因此运行以下命令以生成并将其放置在 JWT 配置文件中：

```php
**php artisan jwt:secret**

```

生成后，您可以在`.env`文件中看到放置的秘密密钥（您的秘密密钥将不同）。检查并确保您的`.env`如下所示：

```php
    APP_DEBUG=true
    APP_ENV=local
    SESSION_DRIVER=file
    DB_HOST=microservice_user_database
    DB_DATABASE=finding_users
    DB_USERNAME=secret
    DB_PASSWORD=mysecret
 **JWT_SECRET=wPB1mQ6ADZrc0ouxMCYJfiBbMC14IAV0**
    CACHE_DRIVER=file
```

现在，转到`config/jwt.php`文件；这是 JWT `config`文件，请确保您的文件如下所示：

```php
    <?php
      return [
        'secret' => env('JWT_SECRET'),
        'keys' => [
          'public' => env('JWT_PUBLIC_KEY'),
          'private' => env('JWT_PRIVATE_KEY'),
          'passphrase' => env('JWT_PASSPHRASE'),
        ],
        'ttl' => env('JWT_TTL', 60),
        'refresh_ttl' => env('JWT_REFRESH_TTL', 20160),
        'algo' => env('JWT_ALGO', 'HS256'),
        'required_claims' => ['iss', 'iat', 'exp', 'nbf', 'sub', 
        'jti'],
        'blacklist_enabled' => env('JWT_BLACKLIST_ENABLED', true),
        'blacklist_grace_period' => env('JWT_BLACKLIST_GRACE_PERIOD', 
        0),
        'providers' => [
          'jwt' => Tymon\JWTAuth\Providers\JWT\Namshi::class,
          'auth' => Tymon\JWTAuth\Providers\Auth\Illuminate::class,
          'storage' => 
          Tymon\JWTAuth\Providers\Storage\Illuminate::class,
        ],
      ];
```

还需要正确设置`config/app.php`。确保您正确输入了用户模型，它将定义 JWT 应该搜索用户和提供的密码的表：

```php
    <?php
      return [
        'defaults' => [
          'guard' => env('AUTH_GUARD', 'api'),
          'passwords' => 'users',
        ],
        'guards' => [
          'api' => [
            'driver' => 'jwt',
            'provider' => 'users',
          ],
        ],
        'providers' => [
          'users' => [
            'driver' => 'eloquent',
 **'model' => \App\Model\User::class,**
          ],
        ],
        'passwords' => [
          'users' => [
            'provider' => 'users',
            'table' => 'password_resets',
            'expire' => 60,
          ],
        ],
      ];
```

现在，我们准备通过编辑`/app/Http/routes.php`来定义需要身份验证的方法：

```php
    <?php
      $app->get('/', function () use ($app) {
        return $app->version();
      });
      use Illuminate\Http\Request;
      use Tymon\JWTAuth\JWTAuth;
      $app->post('login', function(Request $request, JWTAuth $jwt) {
        $this->validate($request, [
          'email' => 'required|email|exists:users',
          'password' => 'required|string'
        ]);
        if (! $token = $jwt->attempt($request->only(['email', 
        'password']))) {
          return response()->json(['user_not_found'], 404);
        }
        return response()->json(compact('token'));
      });
      $app->group(**['middleware' => 'auth']**, function () use ($app) {
        $app->post('user', function (JWTAuth $jwt) {
          $user = $jwt->parseToken()->toUser();
          return $user;
        });
      });
```

您可以在上述代码中看到，我们的中间件只影响我们在其中定义了中间件的组中包含的方法。我们可以创建所有我们想要的组，以便通过我们选择的中间件传递方法。

最后，编辑`/app/Providers/AuthServiceProvider.php`文件，并添加以下突出显示的代码：

```php
    <?php
      namespace App\Providers;
      use App\User;
      use Illuminate\Support\ServiceProvider;
      class AuthServiceProvider extends ServiceProvider
      {
        public function register()
        {
          //
        }
        public function boot()
        {
 **$this->app['auth']->viaRequest('api', function ($request) {**
 **if ($request->input('email')) {**
 **return User::where('email', $request->input('email'))-
              >first();**
 **}**
 **});**
        }
      }
```

最后，我们需要对用户模型文件进行一些更改，因此转到`/app/Model/User.php`并将以下行添加到类实现列表中的`JWTSubject`：

```php
    <?php
      namespace App\Model;
      use Illuminate\Contracts\Auth\Access\Authorizable as 
      AuthorizableContract;
      use Illuminate\Database\Eloquent\Model;
      use Illuminate\Auth\Authenticatable;
      use Laravel\Lumen\Auth\Authorizable;
      use Illuminate\Contracts\Auth\Authenticatable as 
      AuthenticatableContract;
 **use Tymon\JWTAuth\Contracts\JWTSubject;**
      class User extends Model implements **JWTSubject**, 
      AuthorizableContract, 
      AuthenticatableContract {
        use Authenticatable, Authorizable;
        protected $table = 'users';
        protected $fillable = ['email', 'api_token'];
        protected $hidden = ['password'];
   **public function getJWTIdentifier()**
 **{**
 **return $this->getKey();**
 **}**
 **public function getJWTCustomClaims()**
 **{**
 **return [];**
 **}**
      }
```

不要忘记添加`getJWTIdentifier()`和`getJWTCustomClaims()`函数，如上述代码所示。这些函数是实现`JWTSubject`所必需的。

#### 让我们尝试 JWT

为了测试这一点，我们必须在数据库的用户表中创建一个新用户。因此，通过执行迁移或在您喜欢的 SQL 客户端中执行以下查询来添加它：

```php
    INSERT INTO `finding_users`.`users`
    (`id`, `email`, `password`, `api_token`)
    VALUES
    (1,'john@phpmicroservices.com',
    '$2y$10$m5339OpNKEh5bL6Erbu9r..sjhaf2jDAT2nYueUqxnsR752g9xEFy',
    NULL,);
```

手动插入的哈希密码对应于'123456'。Lumen 会为安全原因保存您的用户密码的哈希值。

打开 Postman 并尝试通过对`http://localhost:8084/user`进行 POST 调用。您应该收到以下响应：

```php
    Unauthorized.
```

这是因为`http://localhost:8084/user`方法受到身份验证中间件的保护。您可以在`routes.php`文件中检查这一点。为了获取用户，需要提供有效的访问令牌。

获取有效访问令牌的方法是`http://localhost:8084/login`，因此使用与我们添加的用户对应的参数进行 POST 调用，`email = john@phpmicroservices.com`，密码为`123456`。如果它们是正确的，我们将获得有效的访问令牌：

```php
    {"token":"eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.
    eyJpc3MiOiJodHRwOi8vbG9jYWxob3N0OjgwODQvbG9naW4iLCJ
    pYXQiOjE0ODA4ODI4NTMsImV4cCI6MTQ4MDg4NjQ1MywibmJmIjox
    NDgwODgyODUzLCJqdGkiOiJVVnRpTExZTFRWcEtyWnhsIiwic3
    ViIjoxfQ.jjgZO_Lf4dlfwYiOYAOhzvcTQ4EGxJUTgRSPyMXJ1wg"}
```

现在，我们可以使用前面的访问令牌进行 POST 调用`http://localhost:8084/user`，就像以前一样。这次，我们将获得用户信息：

```php
    {"id":1,"email":"john@phpmicroservices.com","api_token":null}
```

如您所见，使用有效的访问令牌保护您的方法非常简单。这将使您的应用程序更加安全。

# 访问控制列表

这是所有应用程序中非常常见的系统，无论其大小如何。**访问控制列表**（**ACL**）为我们提供了一种简单的方式来管理和过滤每个用户的权限。让我们更详细地看一下。

## ACL 是什么？

应用程序用于识别应用程序的每个单个用户的方法是 ACL。这是一个系统，它告诉应用程序特定任务或操作的用户或用户组有什么访问权限或权限。

每个任务（函数或操作）都有一个属性来标识哪些用户可以使用它，ACL 是一个将每个任务与每个操作（如读取、写入或执行）关联的列表。

对于使用 ACL 的应用程序，ACL 具有以下两个特点优势：

+   **管理**：在我们的应用程序中使用 ACL 允许我们将用户添加到组中，并管理每个组的权限。此外，可以更容易地向许多用户或组添加、修改或删除权限。

+   **安全性**：为每个用户设置不同的权限对应用程序的安全性更好。它可以避免虚假用户或利用通过给予普通用户和管理员不同的权限来破坏应用程序。

对于我们基于微服务的应用程序，我们建议为每个微服务设置不同的 ACL；这样可以避免整个应用程序只有一个入口点。请记住，我们正在构建微服务，其中一个要求是微服务应该是隔离和独立的；因此，有一个微服务来控制其他微服务并不是一个好的做法。

这并不是一项困难的任务，这是有道理的，因为每个微服务应该有不同的任务，每个任务在权限方面对每个用户都是不同的。想象一下，我们有一个用户，该用户具有以下权限。该用户可以创建秘密并检查附近的秘密，但不允许创建战斗或新用户。在全局管理 ACL 将成为一个问题，因为当新的微服务添加到系统中时，甚至当新的开发人员加入团队并且他们必须理解全局 ACL 的复杂系统时，会出现可扩展性问题。正如你所看到的，最好为每个微服务设置一个 ACL 系统，这样当你添加一个新的微服务时，就不需要修改其余的 ACL。

## 如何使用 ACL

Lumen 为我们提供了一个身份验证过程，以便让用户注册、登录、注销和重置密码，并且还提供了一个名为`Gate`的 ACL 系统。

`Gate`允许我们知道特定用户是否有权限执行特定操作。这非常简单，可以在 API 的每个方法中使用。

要在 Lumen 上设置 ACL，必须通过从`app->withFacades();`行中删除分号来启用门面；如果您的文件中没有这一行，请添加它。

在`config/Auth.php`上创建一个新文件是必要的，文件中包含以下代码：

```php
    <?php
      return [
        'defaults' => [
          'guard' => env('AUTH_GUARD', 'api'),
        ],
        'guards' => [
          'api' => [
            'driver' => 'token',
            'provider' => 'users'
          ],
        ],
        'providers' => [
          'users' => [
            'driver' => 'eloquent',
            // We should get model name from JWT configuration
            'model' => app('config')->get('jwt.user'),
          ],
        ],
      ];
```

在我们的控制器上使用`Gate`类来检查用户权限，需要上述代码。

设置好这些后，我们必须定义特定用户可用的不同操作或情况。为此，打开`app/Providers/AuthServiceProvider.php`文件；在`boot()`函数中，我们可以通过编写以下代码来定义每个操作或情况：

```php
    <?php
      /* Code Omitted */
      use Illuminate\Contracts\Auth\Access\Gate;
      class AuthServiceProvider extends ServiceProvider
      {
        /* Code Omitted */
        public function boot()
        {
          Gate::define('update-profile', function ($user, $profile) {
            return $user->id === $profile->user_id;
          });
        }
```

一旦我们定义了情况，我们就可以将其放入我们的函数中。有三种不同的使用方法：*allows*、*checks*和*denies*。前两种是相同的，当定义的情况返回 true 时，它们返回 true，最后一种在定义的情况返回 false 时返回 true：

```php
    if (Gate::**allows**('update-profile', $profile)) {
      // The current user can update their profile...
    }
    if (Gate::**denies**('update-profile', $profile)) {
      // The current user can't update their profile...
    }
```

正如你所看到的，不需要发送`$user`变量，它会自动获取当前用户。

# 源代码的安全性

最有可能的情况是，您的项目将使用一些凭证连接到外部服务，例如数据库。您会把所有这些信息存储在哪里？最常见的方法是在源代码中有一个配置文件，您可以在其中放置所有凭证。这种方法的主要问题是您会提交凭证，任何有权访问源代码的人都将能够访问它们。不管您信任有权访问存储库的人，将凭证存储起来都不是一个好主意。

如果您不能在源代码中存储凭证，您可能想知道如何存储它们。您有两个主要选项：

+   环境变量

+   外部服务

让我们来看看每一个，这样您就可以选择哪个选项更适合您的项目。

## 环境变量

这种存储凭证的方式非常容易实现--您只需定义要存储在环境中的变量，稍后可以在源代码中获取它们。

我们选择的项目框架是 Lumen，使用这个框架非常容易定义您的环境变量，然后在代码中使用它们。最重要的文件是位于源代码根目录的`.env`文件。默认情况下，这个文件在`gitignore`中，以避免被提交，但框架附带了一个`.env.example`示例，以便您可以查看如何定义这些变量。在这个文件中，您可以找到以下定义：

```php
    DB_CONNECTION=mysql
    DB_HOST=localhost
    DB_PORT=3306
    DB_DATABASE=homestead
    DB_USERNAME=homestead
    DB_PASSWORD=secret
```

前面的定义将创建环境变量，您可以使用简单的`env('DB_DATABASE');`或`env('DB_DATABASE', 'default_value');`在代码中获取值。`env()`函数支持两个参数，因此您可以定义一个默认值，以防您要获取的变量未定义。

使用环境变量的主要好处是您可以拥有不同的环境，而无需更改源代码中的任何内容；您甚至可以在不对代码进行任何更改的情况下更改值。

## 外部服务

这种存储凭证的方式使用外部服务来存储所有凭证，它们的工作方式与环境变量差不多。当您需要任何凭证时，您必须向该服务请求。

这些天主流的凭证存储系统之一是 HashiCorp Vault 项目，这是一个开源工具，允许您创建一个安全的地方来存储您的凭证。它有多个好处，我们其中一些重点包括以下几点：

+   HTTP API

+   密钥滚动

+   审计日志

+   支持多个秘密后端

使用外部服务的主要缺点是您为应用程序增加了额外的复杂性；您将添加一个新组件来管理和保持最新状态。

# 跟踪和监控

当您处理应用程序中的安全性时，重要的是要跟踪和监视其中发生的事情。在第六章 *监控*中，我们实现了 Sentry 作为日志和监控系统，并且还添加了 Datadog 作为我们的 APM，因此您可以使用这些工具来跟踪发生的情况并发送警报。

然而，您想要跟踪什么？让我们想象一下，您有一个登录系统，这个组件是一个很好的地方来添加您的跟踪。如果您跟踪每次用户登录失败，您就可以知道是否有人试图攻击您的登录系统。

您的应用程序是否允许用户添加、修改和删除内容？跟踪内容的任何更改，以便您可以检测到不受信任的用户。

在安全方面，没有关于要跟踪什么和不要跟踪什么的标准，只需运用常识。我们的主要建议是创建一个敏感点列表，至少涵盖用户可以登录、创建内容或删除内容的地方，并将这些列表用作添加跟踪和监控的起点。

# 最佳实践

与应用程序的任何其他部分一样，当您处理安全性时，有一些众所周知的最佳实践需要遵循，或者至少要意识到以避免未来的问题。在这里，您可以找到与 Web 开发相关的最常见的最佳实践。

## 文件权限和所有权

文件/文件夹权限和所有权是最基本的安全机制之一。假设您正在使用 Linux/Unix 系统，主要建议是将您的源代码的所有权分配给 Web 服务器或 PHP 引擎用户。关于文件权限，您应该使用以下设置：

+   目录的 500 权限（dr-x------）：此设置防止意外删除或修改目录中的文件。

+   文件的 400 权限（-r--------）：此设置防止任何用户覆盖文件。

+   700 权限（drwx------）：这适用于任何可写目录。它给予所有者完全控制，并用于上传文件夹。

+   600 权限（-rw-------）：这个设置适用于任何可写文件。它避免了任何非所有者的用户对您的文件进行修改。

## PHP 执行位置

通过仅允许在选定路径上执行 PHP 脚本并拒绝在敏感（可写）目录中执行任何类型的执行，例如，任何上传目录，避免任何未来问题。

## 永远不要相信用户

作为一个经验法则，永远不要相信用户。过滤来自任何人的任何输入，您永远不知道表单提交背后的黑暗意图。当然，永远不要仅依赖于前端过滤和验证。如果您在前端添加了过滤和验证，请在后端再次进行过滤和验证。

## SQL 注入

没有人希望他们的数据被暴露或被未经许可的人访问，对您的应用程序的这种攻击是由于输入的过滤或验证不当。想象一下，您使用一个字段来存储未经正确过滤的用户名称，恶意用户可以使用此字段执行 SQL 查询。为了帮助您避免这个问题，当您处理数据库时，请使用 ORM 过滤方法或您喜欢的框架中可用的任何过滤方法。

## 跨站脚本 XSS

这是对您的应用程序的另一种攻击类型，是由于过滤不当。如果您允许用户在页面上发布任何类型的内容，一些恶意用户可能会未经您的许可向页面添加脚本。想象一下，您的页面上有评论部分，您的输入过滤不是最好的，恶意用户可以添加一个作为评论的脚本，打开垃圾邮件弹出窗口。记住我们之前告诉过您的--永远不要相信您的用户--过滤和验证一切。

## 会话劫持

在这种攻击中，恶意用户窃取另一个用户的会话密钥，使恶意用户有机会像其他用户一样。想象一下，您的应用程序涉及财务信息，一个恶意用户可以窃取管理员会话密钥，现在这个用户可以获得他们需要的所有信息。大多数情况下，会话是通过 XSS 攻击窃取的，所以首先要尽量避免任何 XSS 攻击。另一种减轻这个问题的方法是防止 JavaScript 访问会话 ID；您可以在`php.ini`中使用`session.cookie.httponly`设置来做到这一点。

## 远程文件

从您的应用程序包含远程文件可能非常危险，您永远无法 100%确定您包含的远程文件是否可信。如果在某个时刻，被包含的远程文件受到损害，攻击者可以为所欲为，例如，从您的应用程序中删除所有数据。

避免这种问题的简单方法是在您的`php.ini`中禁用远程文件。打开它并禁用以下设置：

+   `allow_url_fopen`：默认情况下启用

+   `allow_url_include`：默认情况下禁用；如果禁用`allow_url_fopen`设置，它会强制禁用此设置。

## 密码存储

永远不要以明文存储任何密码。当我们说永远不要，我们是指永远不要。如果你认为你需要检查用户的密码，那么你是错误的，任何恢复或补充丢失密码的操作都需要通过恢复系统进行。当你存储一个密码时，你存储的是与一些随机盐混合的密码哈希。

## 密码策略

如果你保留敏感数据，并且不希望你的应用程序因用户的密码而暴露，那么请制定非常严格的密码策略。例如，你可以创建以下密码策略来减少破解和字典攻击：

+   至少 18 个字符

+   至少 1 个大写字母

+   至少 1 个数字

+   至少 1 个特殊字符

+   以前未使用过

+   不是用户数据的串联，将元音变成数字

+   每 3 个月过期

## 源代码泄露

将源代码放在好奇的眼睛看不见的地方，如果你的服务器出了问题，所有的源代码都将以明文形式暴露出来。避免这种情况的唯一方法是只在 web 服务器根目录中保留所需的文件。另外，要小心特殊文件，比如`composer.json`。如果我们暴露了我们的`composer.json`，每个人都会知道我们每个库的不同版本，从而轻松地了解可能存在的任何错误。

## 目录遍历

这种攻击试图访问存储在 web 根目录之外的文件。大多数情况下，这是由于代码中的错误导致的，因此恶意用户可以操纵引用文件的变量。没有简单的方法可以避免这种情况；然而，如果你使用外部框架或库，保持它们最新将有所帮助。

这些是你需要注意的最明显的安全问题，但这并不是一个详尽的列表。订阅安全新闻通讯，并保持所有代码最新，以将风险降到最低。

# 总结

在这一章中，我们谈到了安全和认证。我们向您展示了如何加密数据和通信层；我们甚至向您展示了如何构建一个强大的登录系统，以及如何处理应用程序的秘密。安全是任何项目中非常重要的一个方面，所以我们给出了一个常见安全风险的小列表，当然，主要建议是——永远不要相信你的用户。
