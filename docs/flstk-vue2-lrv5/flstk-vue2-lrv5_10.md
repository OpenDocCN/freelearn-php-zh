# 第十章：将全栈应用程序部署到云端

现在 Vuebnb 的功能已经完成，最后一步是将其部署到生产环境。我们将使用两个免费服务，Heroku 和 KeyCDN，与世界分享 Vuebnb。

本章涵盖的主题：

+   Heroku 云平台服务简介

+   将 Vuebnb 作为免费应用程序部署到 Heroku

+   CDN 如何提高全栈应用程序的性能

+   将免费 CDN 与 Laravel 集成

+   为了提高性能和安全性，在生产模式下构建资产

# Heroku

Heroku 是一个用于 web 应用程序的云平台服务。由于其提供的简单性和经济性，它在开发者中非常受欢迎。

Heroku 应用程序可以用各种语言创建，包括 PHP、JavaScript 和 Ruby。除了 web 服务器，Heroku 还提供各种附加组件，如数据库、电子邮件服务和应用程序监控。

Heroku 应用程序可以免费部署，但有一定的限制，例如，应用程序在长时间不活动后会*休眠*，使其响应速度变慢。如果升级到付费服务，这些限制将被解除。

现在我们将 Vuebnb 部署到 Heroku 平台。第一步是通过访问以下 URL 创建一个账户：[`signup.heroku.com`](https://signup.heroku.com)。

# CLI

使用 Heroku 最方便的方式是通过命令行。访问以下 URL 并按照安装步骤进行安装：[`devcenter.heroku.com/articles/heroku-cli`](https://devcenter.heroku.com/articles/heroku-cli)。

安装了 CLI 之后，从终端登录到 Heroku。验证了你的凭据之后，你就可以使用 CLI 来创建和管理你的 Heroku 应用程序了：

```php
$ heroku login

# Enter your Heroku credentials:
# Email: anthony@vuejsdevelopers.com
# Password: ************
# Logged in as anthony@vuejsdevelopers.com
```

# 创建一个应用程序

现在让我们创建一个新的 Heroku 应用程序。新的应用程序需要一个唯一的名称，所以在下面的命令中用你自己的选择替换`vuebnbapp`。这个名称将成为应用程序的 URL 的一部分，所以确保它简短且易记：

```php
$ heroku create vuebnbapp
```

应用程序创建后，你将得到 URL，例如：[`vuebnbapp.herokuapp.com/`](https://vuebnbapp.herokuapp.com/)。在浏览器中输入它，你将看到这个默认消息：

![](img/1df365a5-0d94-4368-b8e6-d0a053a05f7c.png)图 10.1\. Heroku 默认消息新的 Heroku 应用程序被分配了一个免费的域名，例如：`appname.herokuapp.com`，但你也可以使用自己的自定义域名。在 Heroku Dev Center 上查看更多信息：[`devcenter.heroku.com`](https://devcenter.heroku.com)。

# 源代码

要将代码部署到你的 Heroku 应用程序，你可以使用 Heroku 的 Git 服务器。当你用 CLI 创建应用程序时，一个新的远程存储库会自动添加到你的 Git 项目中。用以下命令确认：

```php
$ git remote -v

heroku  https://git.heroku.com/vuebnbapp.git (fetch) heroku  https://git.heroku.com/vuebnbapp.git (push) origin  git@github.com:fsvwd/vuebnb.git (fetch) origin  git@github.com:fsvwd/vuebnb.git (push)
```

一旦我们完成了应用程序的配置，我们将进行第一次推送。Heroku 将使用这段代码来构建应用程序。

# 环境变量

Heroku 应用程序具有一个短暂的文件系统，只包括最近的 Git 推送的代码。这意味着 Vuebnb 将不会有`.env`文件，因为这个文件没有提交到源代码中。

环境变量是由 Heroku CLI 设置的，使用`heroku config`命令。让我们从设置应用程序密钥开始。用你自己的应用程序密钥替换以下值：

```php
$ heroku config:set APP_KEY=base64:mDZ5lQnC2Hq+M6G2iesFzxRxpr+vKJSl+8bbGs=
```

# 创建数据库

我们的生产应用程序需要一个数据库。Heroku 的 ClearDB 附加组件提供了一个易于设置和连接的 MySQL 云数据库。

这个附加组件每个月有限次数的交易是免费的。然而，在你添加数据库之前，你需要验证你的 Heroku 账户，这意味着你需要提供信用卡信息，即使你使用免费计划。

要验证你的 Heroku 账户，前往此 URL：[`heroku.com/verify`](https://heroku.com/verify)。

一旦你完成了这些，用这个命令创建一个新的 ClearDB 数据库：

```php
$ heroku addons:create cleardb:ignite
```

# 默认字符串长度

在撰写本文时，ClearDB 使用的是 MySQL 版本 5.5，而我们的 Homestead 数据库是 MySQL 5.7。MySQL 5.5 中的默认字符串长度对于 Passport 授权密钥来说太短，因此我们需要在运行生产应用程序中的数据库迁移之前，在应用程序服务提供程序中手动设置默认字符串长度。

`app/Providers/AppServiceProvider.php`：

```php
<?php

...

use Illuminate\Support\Facades\Schema;

class AppServiceProvider extends ServiceProvider
{
  ...

  public function boot()
  { Schema::defaultStringLength(191);
  }

  ...
}
```

# 配置

当您安装 ClearDB 附加组件时，会自动设置一个新的环境变量`CLEARDB_DATABASE_URL`。让我们使用`heroku config:get`命令读取它的值：

```php
$ heroku config:get CLEARDB_DATABASE_URL

# mysql://b221344377ce82c:398z940v@us-cdbr-iron-east-03.cleardb.net/heroku_n0b30ea856af46f?reconnect=true
```

在 Laravel 项目中，通过设置`DB_HOST`和`DB_DATABASE`的值来连接数据库。我们可以从`CLEARDB_DATABASE_URL`变量中提取这些值，其格式为：

```php
mysql://[DB_USERNAME]:[DB_PASSWORD]@[DB_HOST]/[DB_DATABASE]?reconnect=true
```

一旦您提取了这些值，就在 Heroku 应用程序中设置适用的环境变量：

```php
$ heroku config:set \
DB_HOST=us-cdbr-iron-east-03.cleardb.net \
DB_DATABASE=heroku_n0b30ea856af46f \
DB_USERNAME=b221344377ce82c \
DB_PASSWORD=398z940v
```

# 配置 Web 服务器

Heroku 的 Web 服务器配置是通过一个名为`Procfile`（无文件扩展名）的特殊文件完成的，它位于项目目录的根目录中。

现在让我们创建该文件：

```php
$ touch Procfile
```

`Procfile`的每一行都是一个声明，告诉 Heroku 如何运行应用程序的各个部分。现在让我们为 Vuebnb 创建一个`Procfile`并添加这个单一声明。

`Procfile`：

```php
web: vendor/bin/heroku-php-apache2 public/
```

冒号左侧的部分是进程类型。`web`进程类型定义了应用程序中 HTTP 请求的发送位置。右侧部分是要运行或启动该进程的命令。我们将把请求路由到指向我们应用程序的*public*目录的 Apache 服务器。

# Passport 密钥

在第九章中，*使用 Passport 添加用户登录和 API 身份验证*，我们使用`php artisan passport:install`命令为 Passport 创建了加密密钥。这些密钥存储在文本文件中，可以在`storage`目录中找到。

加密密钥不应该在版本控制下，因为这会使它们不安全。相反，我们需要在每次部署时重新生成这些密钥。我们可以通过向我们的 composer 文件添加一个 post-install 脚本来实现这一点。

`composer.json`：

```php
"scripts": {
 ...
 "post-install-cmd": [
    "Illuminate\\Foundation\\ComposerScripts::postInstall",
    "php artisan optimize",
```

```php
    "php artisan passport:install"
  ],
} 
```

# 部署

我们已经完成了所有必要的设置和配置，现在我们准备部署 Vuebnb。确保将任何文件更改提交到您的 Git 存储库，并推送到 Heroku Git 服务器的主分支：

```php
$ git add --all
$ git commit -m "Ready for deployment!" $ git push heroku master
```

在推送过程中，您将看到类似以下的输出：

![](img/ad3ba19b-a5b8-411e-a8a2-b4e2dac68549.png)图 10.2\. 推送到 Heroku 后的 Git 输出有问题？`heroku logs --tail`将显示您的 Heroku 应用程序的终端输出。您还可以设置`APP_DEBUG=true`环境变量来调试 Laravel。不过，当您完成后记得将其设置回`false`。

# 迁移和填充

部署完成后，我们将迁移我们的表并填充数据库。您可以通过在 Heroku CLI 中使用`heroku run`来在生产应用程序上运行 Artisan 和其他应用程序命令：

```php
$ heroku run php artisan migrate --seed
```

一旦迁移和填充完成，我们可以尝试通过浏览器查看应用程序。页面应该可以访问，但您会看到这些混合内容错误：

![](img/f0f43db3-8576-44f5-a20d-4e2f75127fb5.png)图 10.3\. 控制台错误

修复这些错误不会有太大帮助，因为所引用的文件实际上并不在服务器上。让我们首先解决这个问题。

# 提供静态资产

由于我们的静态资产，即 CSS、JavaScript 和图像文件，不在版本控制中，它们还没有部署到我们的 Heroku 应用服务器上。

不过，更好的选择是通过 CDN 提供它们。在本章的这一部分，我们将注册 KeyCDN 账户并从那里提供我们的静态资产。

# 内容分发网络

当服务器收到传入的 HTTP 请求时，通常会响应两种类型的内容：动态或静态。动态内容包括包含特定于该请求的数据的网页或 AJAX 响应，例如，通过 Blade 插入用户数据的网页。

静态内容包括图片、JavaScript 和 CSS 文件，在请求之间不会改变。使用 Web 服务器提供静态内容是低效的，因为它不必要地占用服务器资源来简单地返回一个文件。

**内容传送网络**（**CDN**）是一个服务器网络，通常位于世界各地不同位置，专门用于更快、更便宜地传送静态资产。

# KeyCDN

有许多不同的 CDN 服务可用，但在本书中，我们将使用 KeyCDN，因为它提供了一个易于使用的服务，并且有免费使用层。

通过访问此链接并按照说明进行注册：[`app.keycdn.com/signup`](https://app.keycdn.com/signup)。

一旦你创建并确认了一个新的 KeyCDN 账户，通过访问以下链接添加一个新的区域。*区域*只是资产的集合；你可能为你用 KeyCDN 管理的每个网站创建一个不同的区域。将你的新区域命名为*vuebnb*，并确保它是*推送*区域类型，这将允许我们使用 FTP 添加文件：[`app.keycdn.com/zones/add`](https://app.keycdn.com/zones/add)。

# 使用 FTP 上传文件

现在我们将使用 FTP 将静态资产推送到 CDN。你可以使用 FTP 实用程序（如 Filezilla）来完成这个任务，但我已经在项目中包含了一个 Node 脚本`scripts/ftp.js`，可以让你用一个简单的命令来完成。

脚本需要一些 NPM 包，所以首先安装这些包：

```php
$ npm i --save-dev dotenv ftp recursive-readdir
```

# 环境变量

为了连接到你的 KeyCDN 账户，FTP 脚本需要设置一些环境变量。让我们创建一个名为`.env.node`的新文件，将这个配置与主要的 Laravel 项目分开：

```php
$ touch .env.node
```

用于 FTP 到 KeyCDN 的 URL 是[ftp.keycdn.com](http://ftp.keycdn.com)。用户名和密码将与你创建账户时相同，所以确保在以下代码的值中替换它们。远程目录将与你创建的区域名称相同。

`.env.node`：

```php
FTP_HOST=ftp.keycdn.com
FTP_USER=anthonygore
FTP_PWD=*********
FTP_REMOTE_DIR=vuebnb
FTP_SKIP_IMAGES=0
```

# 跳过图片

我们需要传输到 CDN 的文件位于`public/css`、`public/js`、`public/fonts`和`public/images`目录中。FTP 脚本已配置为递归复制这些文件。

然而，如果将`FTP_SKIP_IMAGES`环境变量设置为 true，脚本将忽略`public/images`中的任何文件。你应该在第一次运行脚本后这样做，因为图片不会改变，传输需要相当长的时间。

`.env.node`：

```php
FTP_SKIP_IMAGES=1
```

你可以在`scripts/ftp.js`中看到这是如何生效的：

```php
let folders = [
  'css',
  'js',
  'fonts'
];

if (process.env.FTP_SKIP_IMAGES == 0) { folders.push('images');
}
```

# NPM 脚本

为了方便使用 FTP 脚本，将以下脚本定义添加到你的`package.json`文件中。

`package.json`：

```php
"ftp-deploy-with-images": "cross-env node ./ftp.js",
"ftp-deploy": "cross-env FTP_SKIP_IMAGES=1 node ./ftp.js"
```

# 生产构建

在运行 FTP 脚本之前，确保首先使用`npm run prod`命令为生产构建你的应用程序。这将使用`NODE_ENV=production`环境变量进行 Webpack 构建。

生产构建确保你的资产被优化为生产环境。例如，当 Vue.js 在生产模式下捆绑时，它将不包括警告和提示，并且将禁用 Vue Devtools。你可以从`vue.runtime.common.js`模块的这一部分看到这是如何实现的。

`node_modules/vue/dist/vue.runtime.common.js`：

```php
/**
 * Show production mode tip message on boot? */
productionTip: process.env.NODE_ENV !== 'production',

/**
 * Whether to enable devtools
 */
devtools: process.env.NODE_ENV !== 'production',
```

Webpack 在生产构建过程中还会运行某些仅限于生产环境的插件，以确保你的捆绑文件尽可能小和安全。

# 运行 FTP 脚本

第一次运行 FTP 脚本时，你需要复制所有文件，包括图片。这将需要一些时间，可能需要 20 到 30 分钟，具体取决于你的互联网连接速度：

```php
$ npm run prod && npm run ftp-deploy-with-images
```

一旦传输完成，上传的文件将在区域 URL 上可用，例如，`http://vuebnb-9c0f.kxcdn.com`。文件的路径将相对于`public`文件夹，例如，`public/css/vue-style.css`将在`[ZONE_URL]/css/vue-style.css`上可用。

测试一些文件以确保传输成功：

>![](img/92cca9a8-5fe1-4fc2-a271-2c46b40445b0.png)图 10.4 测试 CDN 文件

后续的传输可以通过使用这个命令跳过图像：

```php
$ npm run prod && npm run ftp-deploy
```

# 从 CDN 读取

我们现在希望在生产环境中，Vuebnb 从 CDN 加载任何静态资产，而不是从 Web 服务器加载。为了做到这一点，我们将创建我们自己的 Laravel 辅助方法。

目前，我们使用 `asset` 辅助程序引用应用中的资产。这个辅助程序返回该资产在 Web 服务器上位置的完全合格的 URL。例如，在我们的应用视图中，我们像这样链接到 JavaScript 捆绑文件：

```php
<script type="text/javascript" src="{{ asset('js/app.js') }}"></script>
```

我们的新辅助程序，我们将其称为 `cdn`，将返回一个指向 CDN 上资产位置的 URL：

```php
<script type="text/javascript" src="{{ cdn('js/app.js') }}"></script>
```

# CDN 辅助程序

让我们开始创建一个名为 `helpers.php` 的文件。这将声明一个新的 `cdn` 方法，目前不会做任何事情，只会返回 `asset` 辅助方法。

`app/helpers.php`:

```php
<?php

if (!function_exists('cdn'))
{
  function cdn($asset)
  {
    return asset($asset);
  }
}
```

为了确保这个辅助程序可以在我们的应用中的任何地方使用，我们可以使用 Composer 的 *autoload* 功能。这使得一个类或文件可以在所有其他文件中使用，而不需要手动 *include* 或 *require* 它。

`composer.json`:

```php
... "autoload": {
  "classmap": [ ... ],
  "psr-4": { ... },
  "files": [
    "app/helpers.php"
  ]
},

...
```

每次修改 Composer 的自动加载声明时，您都需要运行 `dump-autoload`：

```php
$ composer dump-autoload
```

完成后，`cdn` 辅助程序将可以在我们的应用中使用。让我们用 Tinker 测试一下：

```php
$ php artisan tinker >>>> cdn('js/app.js')
=> "http://vuebnb.test/js/app.js"
```

# 设置 CDN URL

`cdn` 辅助程序需要知道 CDN 的 URL。让我们设置一个 `CDN_URL` 环境变量，该变量将被分配给 Vuebnb 的区域 URL，减去协议前缀。

在这个过程中，让我们添加另一个变量 `CDN_BYPASS`，它可以用于在我们不需要 CDN 的本地开发环境中绕过 CDN。

`.env`:

```php
... CDN_URL=vuebnb-9c0f.kxcdn.com
CDN_BYPASS=0
```

现在让我们在应用配置文件中注册这些新变量。

`config/app.php`:

```php
<?php

return [
  ... // CDN

  'cdn' => [
    'url' => env('CDN_URL'),
    'bypass' => env('CDN_BYPASS', false),
  ],
];
```

现在我们可以完成我们的 `cdn` 辅助程序的逻辑。

`app/helpers.php`:

```php
<?php

use Illuminate\Support\Facades\Config;

if (!function_exists('cdn'))
{
  function cdn($asset)
  {
    if (Config::get('app.cdn.bypass') || !Config::get('app.cdn.url')) {
      return asset($asset);
    } else {
      return  "//" . Config::get('app.cdn.url') . '/' . $asset;
    }
  }
}
```

如果您仍然打开了 Tinker，请退出并重新进入，并测试更改是否按预期工作：

```php
>>>> exit
$ php artisan tinker >>>> cdn('js/app.js')
=> "//vuebnb-9c0f.kxcdn.com/js/app.js"
```

# 在 Laravel 中使用 CDN

现在让我们用 `cdn` 辅助程序替换我们的 Laravel 文件中 `asset` 辅助程序的用法。

`app/Http/Controllers/ListingController.php`:

```php
<?php

...

class ListingController extends Controller
{
  private function get_listing($listing)
  {
    ...
    for($i = 1; $i <=4; $i++) {
      $model['image_' . $i] = cdn( 'images/' . $listing->id . '/Image_' . $i . '.jpg' );
    }
    ...
  }

  ...

  private function get_listing_summaries()
  {
    ...
    $collection->transform(function($listing) {
      $listing->thumb = cdn(
        'images/' . $listing->id . '/Image_1_thumb.jpg'
      );
      return $listing;
    });
    ...
  }

  ...
}
```

`resources/views/app.blade.php`:

```php
<html>
  <head>
    ... <link rel="stylesheet" href="{{ cdn('css/style.css') }}" type="text/css">
    <link rel="stylesheet" href="{{ cdn('css/vue-style.css') }}" type="text/css">
    ... </head>
  <body>
    ... <script src="{{ cdn('js/app.js') }}"></script>
  </body>
</html>
```

# 在 Vue 中使用 CDN

在我们的 Vue 应用中，我们也加载一些静态资产。例如，在工具栏中我们使用 logo。

`resources/assets/components/App.vue`:

```php
<img class="icon" src="/images/logo.png">
```

由于这是一个相对 URL，默认情况下它将指向 Web 服务器。如果我们将其改为绝对 URL，我们将不得不硬编码 CDN URL，这也不理想。

让我们让 Laravel 在文档的头部传递 CDN URL。我们只需调用空字符串的 `cdn` 辅助程序即可实现这一点。

`resources/views/app.blade.php`:

```php
<head>
  ... <script type="text/javascript">
     ...
```

```php
 window.cdn_url = "{{ cdn('') }}";
   </script>
</head>
```

现在我们将使用一个计算属性来构建绝对 URL，使用这个全局值。

`resources/assets/components/App.vue`:

```php
<template>
  ... <router-link :to="{ name: 'home' }">
    <img class="icon" :src="logoUrl">
    <h1>vuebnb</h1>
  </router-link>
  ... </template>
<script>
  export default {
    computed: {
      logoUrl() {
        return `${window.cdn_url || ''}images/logo.png`;
      }
    },
    ... }
</script>
<style>...</style>
```

我们将在页脚中使用相同的概念，灰色的 logo 被使用。

`resources/assets/components/CustomFooter.vue`:

```php
<template>
... <img class="icon" :src="logoUrl">
... </template>
<script>
  export default {
    computed: {
      containerClass() { ... },
      logoUrl() {
        return `${window.cdn_url || ''}images/logo_grey.png`;
      }
    },
  }
</script>
```

# 部署到 Heroku

完成后，提交任何文件更改到 Git 并再次推送到 Heroku 以触发新的部署。您还需要重建您的前端资产并将其传输到 CDN。

最后，设置 CDN 环境变量：

```php
$ heroku config:set \
CDN_BYPASS=0 \
CDN_URL=vuebnb-9c0f.kxcdn.com
```

# 终曲

您现在已经完成了本书的案例研究项目，一个复杂的全栈 Vue.js 和 Laravel 应用。恭喜！

一定要向你的朋友和同事展示 Vuebnb，因为他们肯定会对你的新技能印象深刻。我也会很感激，如果你把你的项目链接发给我，这样我也可以欣赏你的工作。我的 Twitter 账号是 `@anthonygore`。

# 回顾

在这本书中，我们走了很长的路，让我们回顾一下我们取得的一些成就：

+   在第一章，*你好 Vue - Vue.js 简介*，我们介绍了 Vue.js

+   在第二章中，*原型设计 Vuebnb，您的第一个 Vue.js 项目*，我们学习了 Vue.js 的基础知识，包括安装、数据绑定、指令和生命周期钩子。我们创建了 Vuebnb 列表页面的原型，包括图像模态框

+   在第三章中，*建立 Laravel 开发环境*，我们安装了主要的 Vuebnb 项目，并设置了 Homestead 开发环境

+   在第四章中，*使用 Laravel 构建 Web 服务*，我们创建了一个 Laravel Web 服务，为 Vuebnb 提供数据

+   在第五章中，*使用 Webpack 集成 Laravel 和 Vue.js*，我们将原型迁移到主项目，并使用 Laravel Mix 将我们的资产编译成捆绑文件

+   在第六章中，*使用 Vue.js 组件组合小部件*，我们学习了组件。我们利用这些知识在列表页面的模态框中添加了图像轮播，并重构了前端以整合单文件组件

+   在第七章中，*使用 Vue Router 构建多页面应用*，我们向项目添加了 Vue Router，允许我们添加一个带有列表摘要滑块的主页

+   在第八章中，*使用 Vuex 管理应用程序状态*，我们介绍了 Flux 架构，并将 Vuex 添加到我们的应用程序中。然后我们创建了一个保存功能，并将页面状态移到了 Vuex 中

+   在第九章中，*使用 Passport 添加用户登录和 API 认证*，我们向项目添加了用户登录。我们通过经过身份验证的 AJAX 调用将用户保存的列表返回到数据库。

+   在第十章中，*将全栈应用部署到云端*，我们将应用部署到 Heroku 云服务器，并将静态资产转移到 CDN

# 下一步

您可能已经读到了本书的结尾，但作为全栈 Vue 开发人员，您的旅程才刚刚开始！接下来应该做什么呢？

首先，您仍然可以向 Vuebnb 添加许多功能。自己设计和实现这些功能将极大地增加您的技能和知识。以下是一些开始的想法：

+   完成用户认证流程。添加注册页面和重置密码的功能

+   添加用户个人资料页面。在这里，用户可以上传头像，在登录时会显示在工具栏中

+   在列表页面创建一个表单，允许预订房间。包括一个下拉式日期选择器小部件，用于选择开始和结束日期

+   通过在服务器上运行 Vue 来对应用进行服务器渲染。这样用户在加载网站时就能看到完整的页面内容

其次，我邀请您查看*Vue.js Developers*，这是一个我创建的 Vue.js 爱好者的在线社区。在这里，您可以阅读有关 Vue.js 的文章，通过我们的通讯订阅了解 Vue.js 的最新消息，并与我们的 Facebook 小组中的其他开发人员分享技巧和诀窍。

在此网址查看：[`vuejsdevelopers.com`](https://vuejsdevelopers.com)。

# 总结

在本章中，我们学习了如何将全栈应用部署到 Heroku 云服务器。为此，我们使用 Heroku CLI 设置了一个新的 Heroku 应用，然后使用 Heroku 的 Git 服务器进行部署。

我们还使用 KeyCDN 创建了一个 CDN，并使用 FTP 将静态资产部署到 CDN。

最后，我们了解到在部署之前以生产模式构建 JavaScript 资产对性能和安全性的重要性。

这是本书的最后一章。感谢您的阅读，祝您在网页开发之旅中好运！
