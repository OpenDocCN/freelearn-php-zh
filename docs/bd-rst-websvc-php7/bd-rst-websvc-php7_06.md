# 使用 Lumen 照亮 RESTful Web 服务

到目前为止，我们已经在核心 PHP 中创建了一个非常基本的 RESTful Web 服务，并发现了设计和安全方面的缺陷。我们还看到，为了改进，我们不需要从头开始创建所有东西。事实上，使用经过时间考验的开源代码更有意义，以基于更干净的代码构建更好的 Web 服务。

在上一章中，我们已经看到 Composer 是 PHP 项目的依赖管理器。在本章中，我们将使用一个开源的微框架来编写 RESTful Web 服务。我们将在一个活跃开发、经过时间考验并在 PHP 社区中广为人知的开源微框架中完成相同的工作。使用框架而不是几个组件的原因是，一个合适的框架可以为我们的代码提供良好的结构，并且它带有一些基本所需的组件。我们选择的微框架是 Lumen，它是全栈框架 Laravel 的微框架版本。

这是我们打算在本章中涵盖的内容：

+   介绍 Lumen

+   Lumen 提供了什么

+   Lumen 与 Laravel 有什么共同之处

+   Lumen 与 Laravel 有何不同

+   安装和配置

+   数据库迁移

+   在 Lumen 中编写 REST API

+   路由

+   控制器

+   REST 资源

+   Eloquent（模型）

+   关系

+   用户访问和基于令牌的身份验证和会话

+   API 版本控制

+   速率限制

+   用户的数据库种子

+   使用 Lumen 包进行 REST API

+   审查基于 Lumen 的 REST API

+   加密的需求

+   不同的 SSL 选项

+   总结和更多资源

# 介绍 Lumen

Lumen 是全栈框架 Laravel 的微框架版本。在 PHP 社区中，Laravel 是一个非常知名的框架。因此，通过使用 Lumen，我们可以随时将我们的项目转换为 Laravel，并开始使用其全栈功能。

# 为什么使用微框架？

一切都有代价。我们选择了微框架而不是全栈框架，因为尽管全栈框架提供了更多的功能，但为了拥有这些功能，它必须加载更多的东西。因此，为了提供更多功能的奢侈，与微框架相比，全栈框架在性能上必须做出一些妥协。另一方面，微框架放弃了一些构建 Web 服务所不需要的功能，比如视图等，这使得它更快。

# 为什么选择 Lumen？

Lumen 并不是 PHP 社区中唯一的微框架。那么为什么选择 Lumen？有三个主要原因：

+   Lumen 是 Laravel 的微框架，因此我们可以通过一点努力将其转换为 Laravel，并利用其全栈功能。

+   由于 Lumen 是 Laravel 的微框架，它像 Laravel 一样拥有出色的社区支持。一个良好的社区总是一个非常重要的因素。同时，Lumen 能够使用许多与 Laravel 相同的包。

+   除了与 Laravel 的关系，Lumen 在性能方面也非常出色。基于性能，其他替代的微框架可能是 Slim 和 Selex。

# Lumen 提供了什么

正如我们所知，Lumen 是 Laravel 的微框架版本，它提供了许多 Laravel 提供的功能。例如，它是一个 MVC 框架。然而，了解 Lumen 和 Laravel 的共同之处以及 Lumen 没有或具有不同的地方是很重要的。这将让我们对 Lumen 为我们提供了什么有一个很好的了解。

# Lumen 与 Laravel 有什么共同之处

在这里，我没有说 Laravel 和 Lumen 之间的相似之处，因为 Lumen 并不是一个完全不同的框架。我说的是它们之间的共同之处，因为它们有共同的包和组件：这意味着它们在许多情况下共享相同的代码库。

实际上，Lumen 是一种小型、精简的 Laravel。它只是放弃了一些组件，并对一些任务使用不同的组件，比如路由。然而，你总是可以在同一个安装中打开很多组件。有时，甚至不需要在配置中编写一些代码。相反，你只需转到配置文件，取消注释一些代码行，它就开始使用这些组件。

事实上，Lumen 具有相同的版本。例如，如果有 Laravel 的 5.4 版本，Lumen 将具有相同的版本。因此，这些不是两个不同的东西。它们彼此之间有很多相似之处。Lumen 只是为了性能而放弃了一些不必要的东西。然而，如果你只想将为 Lumen 编写的应用程序代码转换为 Laravel，你只需将该代码放入 Laravel 安装中，它应该大部分工作。不需要对应用程序代码进行重大更改。

# Lumen 与 Laravel 有何不同

由于 Lumen 是为微服务和 API 构建的，与前端相关的组件，如 elixir、身份验证 bootstrap、会话和视图等，不是 Lumen 的默认组件，但如果需要的话，可以稍后包含：它在这方面非常灵活。

Lumen 中的路由不同。事实上，它不使用 Symfony 路由器；而是使用一个速度更快但功能较少的不同路由器。这是因为 Lumen 为了速度而牺牲了功能。同样，像 Laravel 一样，没有单独的配置文件。相反，一些配置在`.env`文件中完成，而其他与注册提供程序或别名等相关的配置在`bootstrap/app.php`文件中完成，可能是为了避免为了速度而加载不同的文件。

Lumen 和 Laravel 都有很多包，其中很多都适用于两者。但仍然有一些包主要是为 Laravel 构建的，如果没有一些更改，就无法在 Lumen 中使用。因此，如果你打算安装一个包，请确保它支持 Lumen。对于 Laravel，大多数包都适用于 Laravel，因为 Laravel 更受欢迎，大多数包都是为 Laravel 构建的。

# Lumen 到底提供了什么

你可能认为这就是 Lumen 和 Laravel 之间的区别，但 Lumen 到底提供了什么，让我们能够构建 API？我们将研究一下，但不会详细讨论，因为 Lumen 的文档[`lumen.laravel.com/docs/5.4`](https://lumen.laravel.com/docs/5.4)已经涵盖了这一目的。文档中没有涵盖的是我们如何使用 Lumen 制作 RESTful Web 服务，以及我们可以利用哪些包来使我们的工作和生活更轻松。我们将深入研究这一点。

首先，我们将讨论 Lumen 提供了什么，以便我们了解其不同的组件和工作方式。

# 良好的结构

Lumen 具有良好的结构。由于它源自 Laravel，后者遵循 MVC（模型视图控制器）模式，Lumen 也有模型和控制器层。它没有视图层，因为它不需要视图：它是用于 Web 服务的。如果你不知道什么是 MCV，可以将其视为一种架构模式，其中责任分布在三个层中。模型是一个数据库层，有时也用作业务逻辑层（我们将在后面的章节中讨论模型中应该包含什么）。视图层用于模板相关的内容。控制器可以被认为是一个处理请求的层，同时从模型获取数据并渲染视图。在 Lumen 的情况下，只有模型和控制器层。

Lumen 为我们提供了一个良好的结构，因此我们不需要自己制作。事实上，Laravel 不仅提供 MVC 结构，还提供了**服务容器**，可以很好地解决依赖关系。Lumen 和 Laravel 的结构不仅仅是一个设计模式，而是很好地利用了不同的设计模式。因此，让我们看看 Lumen 还提供了什么，并深入了解服务容器和许多其他主题。

# 单独的配置

在第四章中，*审查设计缺陷和安全威胁*，我们看到配置应该与实现分开，所以 Lumen 为我们做到了这一点。它有单独的配置文件。事实上，它有一个单独的`.env`文件，可以在不同的环境中不同。除了`.env`文件，还有一个配置文件，其中存储了与不同包相关的配置，比如包注册或别名等。

请注意，你可能一开始在 Mac 或 Linux 上看不到`.env`文件，因为它以点开头，所以它会被隐藏。你需要显示隐藏文件，然后你就会看到`.env`文件。

# 路由器

Lumen 具有更好的路由能力。它不仅可以让你告诉哪些 URL 应该由哪个控制器提供，还可以让你告诉哪些 URL 以及使用哪种 HTTP 方法应该由哪个控制器的哪个方法提供。事实上，Lumen 可以指定 RESTful 约定中大多数我们使用的 HTTP 方法。

在为我们的博客示例创建 RESTful web 服务时，我们将看到代码示例。

# 中间件

中间件是在控制器提供请求之前或之后执行的一些操作。中间件可以执行许多任务，比如身份验证中间件、验证中间件等。

Lumen 自带一些中间件，同时我们也可以编写自己的中间件来实现我们的目的。

# 服务容器和依赖注入

服务容器是一个用于依赖注入和依赖解析的工具。开发人员只需告诉哪个类应该在哪里注入，服务容器就会解析并注入该依赖。

如果通过应用程序服务容器创建对象，而不是通过应用程序中的`new`关键字，依赖注入可以用于解析类的任何依赖关系。

例如，Lumen 服务容器用于解析所有 Lumen 控制器。因此，如果它们需要任何依赖项，服务容器负责解析它们。为了更好地理解，考虑以下示例：

```php
class PostController extends Post
{
    public function __construct(Post $post){
        //do something with $post
    }
}
```

在前面的示例中，我只是提到了简单的`Controller`类，在`PostController`构造函数中注入了`Post`类。如果我们已经有另一个对象，我们希望注入而不是实际的`Post`对象，我们也可以这样做。

你可以在解析依赖项之前的任何地方使用以下代码来简单地做到这一点：

```php
$ourCustomerPost = new OurCustomPost();
$this->app->instance("\Post", $ourCustomerPost);
```

现在，如果在类的构造函数或方法中对`Post`进行类型提示，那么`OurCustomPost`类的对象将被注入其中。这是因为`$this->app->instance("\Post", $ourCustomerPost)`告诉服务容器，如果有人要求一个`\Post`的实例，就给他们`$ourCustomerPost`。

请注意，除了控制器解析，如果我们希望服务容器注入依赖项，我们还可以以以下方式创建对象：

```php
$postController = $this->app->make('PostController');
```

因此，在这里，`PostController`将以与 Lumen 本身解析控制器相同的方式解析。请注意，我们使用术语*Lumen*，因为我们正在谈论 Lumen，但大部分内容在 Lumen 和 Laravel 中都是相同的。

如果这听起来有点令人不知所措，不要担心，一旦你开始使用 Lumen 或 Laravel 并在其中进行实际工作，你就会开始理解这一点。

# HTTP 响应

Lumen 内置支持发送不同类型的响应、HTTP 状态码和响应头。这是我们之前讨论过的重要内容。对于 Web 服务来说，这更加重要，因为 Web 服务是由机器使用的，而不是人类。机器应该能够知道响应类型和状态码是什么。这不仅有助于判断是否出现错误或成功，还有助于判断发生了什么类型的错误。您可以在[`lumen.laravel.com/docs/5.4/responses`](https://lumen.laravel.com/docs/5.4/responses)中更详细地了解这一点。

# 验证

Lumen 还提供了对验证的支持；不仅是验证支持，还有内置的验证规则可以开始使用。但是，如果您需要为某个字段编写一些自定义验证逻辑，也可以随时进行编写。在创建我们的 RESTful Web 服务时，我们将深入研究这一点。

# Eloquent ORM

Lumen 带有一个名为 Eloquent 的 ORM 工具。为了便于理解，您可以将其视为与数据库相关的高级库，通过它可以在不涉及关系的大量细节的情况下获取数据。在我们使用它时，我们将很快详细了解它。

# 数据库迁移和填充

如今，开发人员并不总是应该使用 SQL 或数据库工具来创建数据库。代码中应该有一些东西可以在版本控制系统下运行，并且团队中的每个开发人员都可以在自己的系统或服务器上运行。这个东西现在被称为迁移。编写迁移的另一个好处是它不是针对一个特定的数据库。相同的迁移可以在 MySQL 和 PostgreSQL 上工作。迁移涉及数据库的结构变化。

迁移是用于创建或修改数据库表，或创建不同的约束或索引。同样，种子数据用于向数据库中插入数据。

# 单元测试

单元测试也是确保代码质量的非常重要的部分，Lumen 也提供了对此的支持。我们不会在本章中编写测试，但我们将在以后的章节中编写测试。

请注意，我们还没有看到 Lumen 提供的每一项功能，我们只看到了一些我们可能需要了解的组件，以便在 Lumen 中创建 RESTful Web 服务。有关 Lumen 的更多详细信息，您可以简单地查阅其文档：[`lumen.laravel.com/docs/5.4`](https://lumen.laravel.com/docs/5.4)。

# Lumen 的安装

要安装 Lumen，如果您已经安装了 composer，只需运行以下命令：

```php
composer create-project --prefer-dist laravel/lumen blog
```

这将创建一个名为`blog`的目录，其中包含 Lumen 的安装。如果您遇到任何困难，请参阅此处的 Lumen 安装文档：[`lumen.laravel.com/docs/5.4`](https://lumen.laravel.com/docs/5.4)。

我建议在安装后，您去查看名为 blog 的这个 Lumen 项目的目录结构，因为当我们执行不同的任务时，这将更有意义。

# 配置

如果您查看我们安装 Lumen 的安装目录，在我们的案例中是`blog`，您会看到一个`.env`文件。Lumen 将配置保存在`.env`文件中。您可以看到有一个选项`APP_KEY=`，如果在`.env`文件中尚未设置，请设置它。这只需要设置为一个具有 32 个字符长度的随机字符串。

由于`.env`文件以点开头，在 Linux 或 Mac 中，此文件可能是隐藏的。为了查看此文件，您需要查看隐藏文件。

然后，要运行 Lumen，只需使用以下命令：

```php
php -S localhost:8000 -t public
```

如您所见，我们正在使用 PHP 内置服务器，并在项目中给出`public`目录的路径。这是因为入口点是`public/index.php`。然后，在[`localhost:8000/`](http://_wp_link_placeholder/)上，您应该看到`Lumen (5.4.6) (Laravel Components 5.4.*)`。

如果您看到错误`Class 'Memcached' not found`，这意味着您没有安装 Memcached，而 Lumen 正在尝试在某个地方使用它。如果您不需要 Memcached，您可以简单地转到`.env`文件并更改`CACHE_DRIVER=file`。

现在我们已经安装和配置了 Lumen，我们将在 Lumen 中为博客示例创建相同的 RESTful Web 服务。

您还需要在`bootstrap/app.php`中取消注释以下内容。

```php
//$app->withFacades();   //$app->withEloquent();
```

正如先前所述，Lumen 在这些功能不可用时可能会更快。但我们取消了注释，因为我们还需要利用 Lumen 的一些功能。那么，这两行代码到底是做什么的呢？第一行代码启用了 Facades 的使用。我们启用它是因为我们将需要一些需要 Facade 的包。第二行代码启用了 Laravel 和 Lumen 附带的 Eloquent ORM 的使用。出于性能考虑，默认情况下不启用 Eloquent。但是，Eloquent 是一个非常重要的组件，我们不应该忽视它，即使是出于性能考虑，除非性能对我们非常重要并且由于 Eloquent 而变慢。在我看来，除非情况紧急，否则我们不应该为了性能而牺牲清晰度。

# 设置数据库

我们需要为博客设置数据库。实际上，我们在第三章中已经设置了这一点，*创建 RESTful 端点*。我们可以在这里使用该数据库。实际上，我们将拥有相同的数据库结构，因此我们可以轻松地使用相同的数据库，但这并不推荐。在 Lumen 中，我们使用迁移来创建数据库结构。这不是强制性的，但很有用，因此您可以编写一次迁移并在任何地方使用它来创建数据库结构。SQL 文件也可以实现这个目的，但是迁移的美妙之处在于它也可以跨不同的关系型数据库管理系统工作。因此，手动创建一个名为`blog`的数据库。现在，我们将为结构编写迁移。

# 编写迁移

要在 Lumen 中创建迁移文件，我们可以在`blog`目录中使用以下命令创建迁移文件：

```php
php artisan make:migration create_users_table
```

您将看到类似于这样的内容：

```php
Created Migration: 2017_06_23_180043_create_users_table
```

并且将在`/blog/database/migrations`目录中创建一个具有此名称的文件。在此文件中，我们可以为用户表编写迁移代码。如果您打开文件并查看其中，您会发现其中有两个方法：`up()`和`down()`。`up()`方法在运行迁移时执行，而`down()`在回滚迁移时执行。

这是用户表创建迁移文件的内容：

```php
<?php   use Illuminate\Database\Migrations\Migration; use Illuminate\Database\Schema\Blueprint;   class CreateUsersTable extends Migration {    /**
 * Run the migrations. * * @return void
 */  public function up()
 { Schema::create('users', function(Blueprint $table)
 {  $table->integer('id', true);
  $table->string('name', 100);
  $table->string('email', 50)->unique('email_unique');
  $table->string('password', 100);
 $table->timestamps();  }); }      /**
 * Reverse the migrations. * * @return void
 */  public function down()
 { Schema::drop('users');
 }   } 
```

在`up()`方法中，我们调用了 create 方法，并传递了一个函数。该函数包含添加字段的代码。如果您想了解有关通过迁移创建字段和表的更多信息，可以查看[`laravel.com/docs/5.4/migrations#tables`](https://laravel.com/docs/5.4/migrations#tables)。

但是，在运行从数据库生成迁移的命令之前，您应该转到`.env`文件并添加您的数据库名称和凭据。为了运行迁移，请运行以下命令：

```php
php artisan migrate
```

这将运行迁移，并创建两个表：迁移表和用户表。用户表是由先前提到的代码创建的，而迁移表是由 Laravel/Lumen 创建的，用于记录运行的迁移。这个表是第一次创建的，每次运行迁移时都会有更多的数据。

请注意，在运行迁移之前，您应该在`.env`文件中安装和配置 MySQL 或其他数据库。否则，如果没有安装或设置数据库，则迁移将无法工作。

现在，您可以以相同的方式创建帖子和评论表创建迁移文件。以下是帖子和评论表创建迁移文件的内容。

帖子迁移文件内容：

```php
<?php   use Illuminate\Database\Migrations\Migration; use Illuminate\Database\Schema\Blueprint;   class CreatePostsTable extends Migration {    /**
 * Run the migrations. * * @return void
 */  public function up()
 { Schema::create('posts', function(Blueprint $table)
 {  $table->integer('id', true);
  $table->string('title', 100);
  $table->enum('status', array('draft','published'))->default('draft');
  $table->text('content', 65535);
  $table->integer('user_id')->index('user_id_foreign');

         $table->timestamps();
 }); }      /**
 * Reverse the migrations. * * @return void
 */  public function down()
 { Schema::drop('posts');
 }   } 
```

这是评论表创建迁移文件：

```php
<?php   use Illuminate\Database\Migrations\Migration; use Illuminate\Database\Schema\Blueprint;   class CreateCommentsTable extends Migration {    /**
 * Run the migrations. * * @return void
 */  public function up()
 { Schema::create('comments', function(Blueprint $table)
 {  $table->integer('id', true);
  $table->string('comment', 250);
  $table->integer('post_id')->index('post_id');
  $table->integer('user_id')->index('user_id');

         $table->timestamps();
 }); }      /**
 * Reverse the migrations. * * @return void
 */  public function down()
 { Schema::drop('comments');
 }   } 
```

在拥有上述两个文件之后，再次运行以下命令：

```php
php artisan migrate
```

上述命令将只执行尚未执行的新迁移文件。有了这样，你将在数据库中拥有这三个表。而且，由于我们也将有这些迁移，所以只需再次运行迁移，就可以在数据库中拥有这个模式。你可能会想，“编写迁移有什么好处呢？”。好处在于，迁移使得在任何 RDBMS 上部署变得更容易，因为代码是 Laravel 迁移代码，而不是 SQL 代码。此外，将这样的东西放在代码中总是更容易的，这样多个开发人员就可以获取彼此的迁移并立即运行它们。

如果你还记得，我们还做了一些索引和外键约束。所以，这就是我们如何在迁移中做到的。

使用与之前相同的命令创建一个新的迁移文件：

```php
php artisan make:migration add_foreign_keys_to_comments_table
```

这将为评论表索引创建一个迁移文件。让我们给这个文件添加内容：

```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;

class AddForeignKeysToCommentsTable extends Migration {

    /**
     * Run the migrations.
     *
     * @return void
     */
    public function up()
    {
        Schema::table('comments', function(Blueprint $table)
        {
            $table->foreign('post_id', 'post_id_comment_foreign')->references('id')->on('posts')->onUpdate('RESTRICT')->onDelete('RESTRICT');
            $table->foreign('post_id', 'post_id_foreign')->references('id')->on('posts')->onUpdate('RESTRICT')->onDelete('RESTRICT');
            $table->foreign('user_id', 'user_id_comment_foreign')->references('id')->on('users')->onUpdate('RESTRICT')->onDelete('RESTRICT');
        });
    }

    /**
     * Reverse the migrations.
     *
     * @return void
     */
    public function down()
    {
        Schema::table('comments', function(Blueprint $table)
        {
            $table->dropForeign('post_id_comment_foreign');
            $table->dropForeign('post_id_foreign');
            $table->dropForeign('user_id_comment_foreign');
        });
    }

}
```

同样地，为帖子索引创建一个迁移文件。以下是文件的内容：

```php
<?php   use Illuminate\Database\Migrations\Migration; use Illuminate\Database\Schema\Blueprint;   class AddForeignKeysToPostsTable extends Migration {    /**
 * Run the migrations. * * @return void
 */  public function up()
 { Schema::table('posts', function(Blueprint $table)
 {  $table->foreign('user_id', 'user_id_foreign')->references('id')->on('users')->onUpdate('RESTRICT')->onDelete('RESTRICT');
 }); }      /**
 * Reverse the migrations. * * @return void
 */  public function down()
 { Schema::table('posts', function(Blueprint $table)
 {  $table->dropForeign('user_id_foreign');
 }); }   } 
```

在上述索引文件代码中，有一些代码有点复杂，需要我们的注意：

```php
  $table->foreign('user_id', 'user_id_foreign')->references('id')->on('users')->onUpdate('RESTRICT')->onDelete('RESTRICT');
```

在这里，`foreign()` 方法接受字段名和索引的名称。然后，`references()` 方法接受父表中外键字段的名称，`on()` 方法的参数是被引用的表名（在我们的例子中，是用户表）。然后，另外两个方法 `onUpdate()` 和 `onDelete()` 告诉用户在更新和删除时该做什么。如果你对迁移语法不太熟悉，没关系；你只需要查看 Lumen/Laravel 迁移文档。事实上，我建议你停下来，看一下与迁移相关的文档：[`laravel.com/docs/5.4/migrations`](https://laravel.com/docs/5.4/migrations)。

现在，为了使这些迁移在数据库中生效，我们需要再次运行迁移，以便新的迁移可以执行，并且我们可以在数据库中看到变化。所以运行：

```php
php artisan migrate
```

有了这个，我们就完成了迁移。现在我们可以通过种子在这些表中插入一些数据，但现在我们不需要，所以暂时跳过编写种子。

# 编写 RESTful web service 端点

现在，是时候真正开始编写我们在第一章“RESTful Web Services, Introduction and Motivation”中讨论过的端点，并在第三章中用纯粹的 Vanilla PHP 编写的端点了。所以让我们开始吧。

由于它有控制器和模型层，我们将从控制器层开始编写 API，该层将为不同的端点提供服务。对于第一个控制器，我们要编写的是 `PostController`。

# 编写第一个控制器

从技术上讲，这不是第一个控制器，因为 Lumen 自带了 2 个控制器，你可以在 `/<our blog project path>/app/Http/Controllers/` 目录中找到。但这是我们要编写的第一个控制器。在 Laravel（Lumen 的大哥）中，我们不需要去创建控制器，因为有相应的命令，但是在 Lumen 中这些命令是不可用的。由于这些命令不是强制性的，但非常方便，最好是让这些命令可用。

为了使用我们在 Lumen 中没有的额外功能（其中一些已经包含在 Laravel 中），我们需要安装一个包。现在，我们需要安装的包是 `flipbox/lumen-generator`。关于这个包的更多信息可以在 [`packagist.org/packages/flipbox/lumen-generator`](https://packagist.org/packages/flipbox/lumen-generator) 找到。

正如我们在前一章中所看到的，我们通过 composer 安装包，所以让我们安装它：

```php
composer require --dev flipbox/lumen-generator
```

你可以看到我在那里添加了一个 `--dev` 标志。我这样做是为了避免在生产环境中使用它，因为这样它将被添加到 `composer.json` 中的 `require --dev` 部分。

无论如何，一旦安装了这个，你可以在 `bootstrap/app.php` 中注册它的 `ServiceProvider`。

```php
if ($app->environment() !== 'production') {
  $app->register(Flipbox\LumenGenerator\LumenGeneratorServiceProvider::class); }
```

现在，你可以看到我们有更多的命令可用。你可以通过运行以下命令来查看：

```php
php artisan migrate
```

因此，让我们使用命令创建一个控制器。请注意，我们不仅仅是为了创建控制器而安装它，但当你使用它时，它将非常方便。无论如何，让我们使用以下命令创建一个控制器：

```php
php artisan make:controller PostController --resource
```

它将在`app/Http/Controllers/PostController.php`创建一个控制器。这个命令不仅会创建`PostController`，还会添加与 REST 资源相关的方法。打开一个文件并查看它。

这是它生成的内容：

```php
<?php   namespace App\Http\Controllers;   use Illuminate\Http\Request;   class PostController extends Controller {
  /**
 * Display a listing of the resource. * * @return \Illuminate\Http\Response
 */  public function index()
 {  //
  }    /**
 * Store a newly created resource in storage. * * @param \Illuminate\Http\Request  $request
 * @return \Illuminate\Http\Response
 */  public function store(Request $request)
 {  //
  }    /**
 * Display the specified resource. * * @param int  $id
 * @return \Illuminate\Http\Response
 */  public function show($id)
 {  //
  }    /**
 * Update the specified resource in storage. * * @param \Illuminate\Http\Request  $request
 * @param int  $id
 * @return \Illuminate\Http\Response
 */  public function update(Request $request, $id)
 {  //
  }    /**
 * Remove the specified resource from storage. * * @param int  $id
 * @return \Illuminate\Http\Response
 */  public function destroy($id)
 {  //
  } } 
```

这些方法是因为我们添加了`--resource`标志而生成的。如果你想知道我是从哪里获取到这个标志的知识，因为它没有列在包页面上，我是从 Laravel 的控制器文档中获取的[`laravel.com/docs/5.4/controllers#resource-controllers`](https://laravel.com/docs/5.4/controllers#resource-controllers)。然而，由于这些命令是由第三方包工作的，所以 Laravel 文档和这些命令的实际行为可能会有所不同，但由于这些是为了复制 Laravel 命令而在 Lumen 中完成的，它们很可能会非常相似。

无论如何，我们有`PostController`及其方法。让我们逐个实现这些方法。

但是，请注意，在 Lumen 和 Laravel 中，与其他 PHP MVC 框架不同，每个 URL 都应该在路由中告知，否则它将无法访问。路由是一种唯一的入口点，不像其他框架如`CodeIgniter`，路由是可选的。在 Lumen 中，路由是必需的。换句话说，控制器的每个方法只能通过路由访问。

因此，在继续使用`PostController`之前，让我们为帖子端点添加路由，否则`PostController`将毫无用处。

# Lumen 路由

在 Lumen 中，默认情况下，路由位于`/routes/web.php`。我说默认是因为这个路径可以更改。无论如何，进入`routes/web.php`并查看它。你会看到它自己返回一个响应，而不是指向任何控制器。所以，你应该知道它是由路由决定是否返回响应或使用控制器。但是，请注意，只有在路由闭包中返回响应才有意义，如果涉及的逻辑不多。在我们的情况下，我们将主要使用控制器。

当我们添加第一个路由时，我们的路由将如下所示：

```php
<?php   /* |---------------------------------------------------------------- | Application Routes |---------------------------------------------------------------- | | Here is where you can register all of the routes for an application. | It is a breeze. Simply tell Lumen the URIs it should respond to | and give it the Closure to call when that URI is requested. | */   $app->get('/', function () use ($app) {
  return $app->version(); });  $app->get('/api/posts', [
    'uses' => 'PostController@index',
    'as' => 'list_posts'
]);
```

粗体字中的代码是我们编写的。在这里，`$app->get()`中的`get`用于指定 HTTP 方法。它可以是`$app->post()`，但我们使用了`$app->get()`来指定接受`GET`方法。然后，在这个方法中有 2 个参数，你可以在前面的代码中看到。第一个是路由模式，而第二个参数是一个关联数组，其中`uses`键中有控制器和方法，`as`键中有路由名称：意味着在域或项目 URL 之后，如果`api/posts/`是一个 URL，它应该由`PostController`的`index()`方法提供服务。而路由名称只是在那里，如果你想在代码中按名称指定路由 URL，那么它是有用的。

现在，为了检查我们的路由是否正确，并从控制器的`index`方法获取响应，让我们向`PostController`的`index`方法添加一些内容。这是我们现在添加的内容，只是为了测试我们的路由：

```php
public function index() {
 return ['response' =>
        [
            'id' => 1,
            'title' => 'Some Post',
            'body' => 'Here is post body'
        ] **];** }
```

现在，尝试运行这段代码。在任何其他操作之前，你需要使用 PHP 内置服务器。进入整个代码所在的`blog`目录并运行：

```php
php -S localhost:8000 -t public
```

然后，从浏览器中输入：`http://localhost:8000/api/posts`，你将看到以下响应：

```php
{"response":{"id":1,"title":"Some Post","body":"Here is post body"}}
```

正如你所看到的，我们的路由起作用并从`PostController`的`index()`方法提供服务，如果你返回一个数组，Lumen 会将其转换为 JSON 并作为 JSON 返回。

要进一步查看特定 URL 映射到特定控制器特定方法的路由列表，只需运行：

```php
php artisan route:list
```

你将看到路由的详细信息，告诉你哪个 URL 模式与哪段代码相关联。

# REST 资源

这是一个非常基本的路由示例，由 `PostController` 方法提供。然而，如果你查看 `PostController`，它还有 4 个方法，我们需要为第一章中讨论的 4 个端点提供服务，并在第三章中实现*创建 RESTful 端点*。因此，我们需要在 Lumen 中为其他 4 个方法做同样的事情。为了将这 4 个方法映射到 4 个端点，我们不需要再添加 4 个路由。我们可以简单地添加一个基于资源的路由，它将将 REST 基础的 URL 模式映射到`PostController`的所有方法。

通过命令行创建`PostController`时，它创建了一个资源控制器，这意味着它具有提供 RESTful 端点所需的方法。因此，在`routes/web.php`文件中，我们应该用资源路由替换之前编写的代码。现在，我们应该能够通过在路由文件中添加这个语句来将所有 RESTful 端点映射到`PostController`的方法：

```php
$app->resource('api/posts', 'PostController');
```

不幸的是，这个资源路由在 Laravel 中可用，但在 Lumen 中不可用。Lumen 使用不同的路由器以获得更好的性能。然而，这个资源方法也非常方便，如果我们有 4-5 个以上的 RESTful 资源，我们可以只用 4-5 个语句来映射它们的所有端点，而不是 16-20 个语句。因此，这里有一个小技巧，可以在 Lumen 中使用这种资源路由的方法。你可以将这个自定义方法添加到同一个路由文件中。

```php
function resource($uri, $controller) {
  //$verbs = ['GET', 'HEAD', 'POST', 'PUT', 'PATCH', 'DELETE'];
  global $app;

  $app->get($uri, $controller.'@index');
  $app->post($uri, $controller.'@store');

  $app->get($uri.'/{id}', $controller.'@show');
  $app->put($uri.'/{id}', $controller.'@update');
  $app->patch($uri.'/{id}', $controller.'@update');

  $app->delete($uri.'/{id}', $controller.'@destroy'); }  
```

因此，我们的路由文件将如下所示：

```php
<?php   /* |---------------------------------------------------------------- | Application Routes |---------------------------------------------------------------- | | Here is where you can register all of the routes for an application. | It is a breeze. Simply tell Lumen the URIs it should respond to | and give it the Closure to call when that URI is requested. | */    function resource($uri, $controller)
{
    //$verbs = ['GET', 'HEAD', 'POST', 'PUT', 'PATCH', 'DELETE'];

    global $app;

    $app->get($uri, $controller.'@index');
    $app->post($uri, $controller.'@store');

    $app->get($uri.'/{id}', $controller.'@show');
    $app->put($uri.'/{id}', $controller.'@update');
    $app->patch($uri.'/{id}', $controller.'@update');

    $app->delete($uri.'/{id}', $controller.'@destroy'); **}**     $app->get('/', function () use ($app) {
  return $app->version(); });   resource('api/posts', 'PostController'**);** 
```

粗体字中的代码是我们添加的。因此，你可以看到我们在路由文件中一次定义了`resource()`函数，并且我们可以将其用于所有 REST 资源路由。然后在最后一行，我们使用资源函数将所有`api/posts`端点映射到`PostController`的相应方法。

现在你可以通过访问 `http://localhost:8000/api/posts` 来测试它。我们现在无法测试其他端点，因为我们还没有在 `PostController` 的其他方法中编写任何代码。但是，你可以使用以下命令来查看存在的路由：

```php
php artisan route:list
```

在我们的情况下，这个命令将在命令行上产生类似这样的结果：

![](img/d2d15438-6b99-4d46-9620-e339b0e5651f.png)

在这里，我们可以看到这是根据我们在[第一章](https://cdp.packtpub.com/building_restful_web_services_with_php_7/wp-admin/post.php?post=260&action=edit&save=save#post_412)中讨论的 RESTful 资源约定将路径映射到`PostController`方法。因此，对于帖子端点，我们已经完成了路由。现在我们需要在控制器中添加代码，以便它可以向数据库添加数据并从数据库中获取数据。下一步是创建一个模型层，并在控制器中使用它并返回适当的响应。

# Eloquent ORM（模型层）

Eloquent 是 Laravel 和 Lumen 附带的 ORM。它负责数据库相关操作以及数据库关系。**ORM**（**对象关系映射**）基本上将对象与数据库中的关系（表）进行映射。不仅如此，基于关系，你可以在不涉及底层细节的情况下获取另一个表的数据。这不仅节省了我们的时间，还使我们的代码更加清晰。

# 创建模型

我们现在要创建模型。模型层与数据库相关，因此我们也会在其中提及数据库关系。让我们为我们拥有的三个表创建模型。模型名称将是`User`、`Post`和`Comment`，分别对应`users`、`posts`和`comments`表。

我们不需要创建一个用户模型，因为它已经包含在 Lumen 中。要创建帖子和评论模型，让我们运行以下命令，这些命令是通过使用`flipbox/lumen-generator`包而变得可用的。运行以下命令来创建模型：

这将在`app`目录中创建一个`Post`模型：

```php
php artisan make:model Post
```

这将在`app`目录中创建一个`Comment`模型：

```php
php artisan make:model Comment
```

如果你查看这些模型文件，你会发现这些都是继承自 Eloquent Model 的类；因此，这些模型都是基于 Eloquent Model 的模型，并具有 Eloquent Model 的特性。

注意，根据 Eloquent 的约定，如果模型的名称是 Post，表的名称将是 posts，即模型名称的复数形式。同样，对于 Comment 模型，它将是 comments 表。如果我们的表名不同，我们可以覆盖这一点，但我们没有这样做，因为在我们的情况下，我们的表和模型名称都符合相同的约定。

Eloquent 是一个大的讨论话题，但我们只是用它来制作我们的 API，所以我将限制讨论 Eloquent 在服务我们目的方面的使用。我认为这是有道理的，因为 Eloquent 的文档中已经有很多细节，所以有关 Eloquent 的更多细节，请参阅这里的 Eloquent 文档：[`laravel.com/docs/5.4/eloquent`](https://laravel.com/docs/5.4/eloquent)。

# Eloquent 关系

在模型层，特别是在从 ORM 继承时，有两个重要的事情：

+   我们应该有模型，这样我们可以通过它们访问数据

+   我们应该指定关系，这样我们可以利用 ORM 的全部功能

只需访问数据而不编写查询，我们也可以使用查询构建器。但是，关系的优势在于它仅与 ORM 使用一起出现。因此，让我们指定所有模型的关系。

首先，让我们指定用户的关系。由于用户可以有多篇帖子和多条评论，用户模型将与帖子和评论模型都有`hasMany`关系。在指定关系后，用户模型将如下所示：

```php
<?php   namespace App;   use Illuminate\Auth\Authenticatable; use Laravel\Lumen\Auth\Authorizable; use Illuminate\Database\Eloquent\Model; use Illuminate\Contracts\Auth\Authenticatable as AuthenticatableContract; use Illuminate\Contracts\Auth\Access\Authorizable as AuthorizableContract;   class User extends Model implements AuthenticatableContract, AuthorizableContract {
  use Authenticatable, Authorizable;    /**
 * The attributes that are mass assignable. * * @var array
 */  protected $fillable = [
  'name', 'email',
 ];    /**
 * The attributes excluded from the model's JSON form. * * @var array
 */  protected $hidden = [
  'password',
 ]; public function posts(){
        return $this->hasMany('App\Post');
    }

    public function comments(){
        return $this->hasMany('App\Comment'); **}** } 
```

我们在 User Model 中添加的唯一的东西是这两个用粗体标出的方法，它们是`posts()`和`comments()`，指定了关系。基于这些方法，我们可以访问用户的帖子和评论数据。这两个方法告诉我们，用户与`Post`和`Comment`模型都有多对多的关系。

现在，让我们在`Post`模型中添加一个关系。由于帖子可以有多条评论，`Post`模型与`Comment`模型有多个关系。同时，`Post`模型与用户模型有多个反向关系，该反向关系是`belongsTo`关系。在添加关系信息后，`Post`模型代码如下。

```php
<?php   namespace App;   use Illuminate\Database\Eloquent\Model;   class Post extends Model {
 public function comments(){
        return $this->hasMany('App\Comment');
    }

    public function user(){
        return $this->belongsTo('App\User'); **}** } 
```

正如你所看到的，我们已经指定了帖子与`User`和`Comment`模型的关系。现在，这是带有关系的`Comment`模型。

```php
<?php   namespace App;   use Illuminate\Database\Eloquent\Model;   class Comment extends Model {
 public function post(){
        return $this->belongsTo("App\Post");
    }

    public function user(){
        return $this->belongsTo("App\User"); **}** } 
```

正如你所看到的，对于`Post`和`User`模型，评论都有一个`belongsTo`关系，这是`hasMany()`的反向关系。

所以，现在我们已经指定了关系。是时候实现`PostController`的方法了。

# 控制器实现

让我们首先在`PostController`的`index()`方法中添加适当的代码，以返回实际数据。但是为了查看响应中的数据，最好在用户、帖子和评论表中插入一些虚拟数据。更好的方法是为此编写种子。但是，如果你不想了解如何编写种子，那么现在可以手动插入。

以下是`index()`方法的实现：

```php
public function index(\App\Post $post) {
  return $post->paginate(20); }
```

在这里，`paginate(20)`表示它将返回一个带有 20 个限制的分页结果。正如你所看到的，我们使用了依赖注入来获取`Post`对象。这是我们在本章中已经讨论过的内容。

同样，我们将在这里实现`PostController`的其他方法。`PostController`代码将如下所示：

```php
<?php   namespace App\Http\Controllers;   use Illuminate\Http\Request;   class PostController extends Controller {   public function __construct(\App\Post $post)
    {
        $this->post = $post; **}**    /**
 * Display a listing of the resource. * * @return \Illuminate\Http\Response
 */  public function index()
 { return $this->post->paginate(20**);**
 }    /**
 * Store a newly created resource in storage. * * @param \Illuminate\Http\Request  $request
 * @return \Illuminate\Http\Response
 */  public function store(Request $request)
 { $input = $request->all();
        $this->post->create($input);

        return [
            'data' => $input **];**
 }    /**
 * Display the specified resource. * * @param int  $id
 * @return \Illuminate\Http\Response
 */  public function show($id)
 {  return $this->post->find($id**);**
 }    /**
 * Update the specified resource in storage. * * @param \Illuminate\Http\Request  $request
 * @param int  $id
 * @return \Illuminate\Http\Response
 */  public function update(Request $request, $id)
 { $input = $request->all();
        $this->post->where('id', $id)->update($input);

        return $this->post->find($id**);**
 }    /**
 * Remove the specified resource from storage. * * @param int  $id
 * @return \Illuminate\Http\Response
 */  public function destroy($id)
 { $post = $this->post->destroy($id);

        return ['message' => 'deleted successfully', 'post_id' => $post**];**
 } } 
```

正如你所看到的，我们正在使用 Post 模型并使用它的方法执行不同的操作。Lumen 的变量和函数名称使我们更容易理解发生了什么，但如果你想知道可以使用哪些 Eloquent 方法，请查看 Eloquent：[`laravel.com/docs/5.4/eloquent`](https://laravel.com/docs/5.4/eloquent)。

如果您在那里找不到 Eloquent 方法的文档，请注意，我们使用的许多函数都是查询构建器的函数。因此，还要查看查询构建器的文档，网址为[`laravel.com/docs/5.4/queries`](https://laravel.com/docs/5.4/queries)。

由于`CommentController`的实现将类似，我建议您自己实现`CommentController`，因为您只有自己动手才能真正学会。

# 我们错过了什么？

我们是否已经创建了为 RESTful 资源端点提供服务的控制器？实际上没有，我们错过了很多东西。我们只是创建了基本的 RESTful 网络服务，可以让您了解如何使用 Lumen 制作它，但我们错过了很多东西。因此，让我们看看它们，逐个完成它们。

# 验证和负面情况？

首先，我们只处理积极的情况：这意味着我们不考虑如果请求不符合我们的假设会发生什么。如果用户使用错误的方法发送数据会发生什么？如果用户传递的 ID 不存在记录会发生什么？

简而言之，我们还没有处理所有这些，但是 Lumen 已经为我们处理了一些事情。

如果您尝试使用`POST`方法命中端点 URL，`http://localhost:8000/api/posts/1`，那么这是一种无效的方法。在这些 URL 上，我们只能使用`GET`，`PUT`或`PATCH`发送请求。使用`GET`将触发`PostController`的`show()`方法，而`PUT`或`PATCH`将触发`update()`方法。但是不应该允许使用`POST`方法。实际上，如果您尝试使用`POST`方法在这些 URL 上发送请求，它将不起作用，您还将收到`Method Not Allowed`错误，就像应该的那样。因此，通过一次定义我们的路由，Lumen 将自行处理此类错误。

同样，Lumen 将使错误的 URL 或错误的 HTTP 方法和 URL 组合无效。

除此之外，我们不打算处理每一种情况，但让我们看看我们必须处理的重要事情；如果没有这些东西，我们的工作就无法完成。

因此，让我们看看`PostController`每个方法中我们错过了什么关于验证或缺失的用例。

# 使用 GET 方法的/api/posts

以下是`/api/posts`端点的响应（在我的情况下，数据库中只有一条记录）：

```php
{
   "current_page":1,
   "data":[
      {
         "id":1,
         "title":"test",
         "status":"draft",
         "content":"test post",
         "user_id":2
      }
   ],
   "from":1,
   "last_page":1,
   "next_page_url":null,
   "path":"http:\/\/localhost:8000\/api\/posts",
   "per_page":20,
   "prev_page_url":null,
   "to":1,
   "total":1
}
```

如果您回忆一下我们在第一章中看到的响应，*RESTful Web Services, Introduction and Motivation*，您会发现我们得到了我们讨论的大部分信息，但是以不同的格式得到了。尽管这完全没问题，因为它提供了足够的信息，但是这是如何使其与我们决定的内容相似。

以下是`index()`方法将变成什么样子：

```php
public function index() {
 $posts = $this->post->paginate(20);
    $data = $posts['data'];

    $response = [
        'data' => $data,
        'total_count' => $posts['total'],
        'limit' => $posts['per_page'],
        'pagination' => [
            'next_page' => $posts['next_page_url'],
            'current_page' => $posts['current_page']
        ]
    ];

    return $response**;** }
```

最重要的是，响应的根级别的所有信息都不清晰。我们应该消除损害清晰度的内容，因为如果程序员需要花更多时间才能理解某些内容，程序员的生产力可能会受到影响。我们应该将分页相关的信息放在一个单独的属性下，可以是 pagination 或 meta，这样程序员就可以轻松地查看数据和其他属性。

我们做到了，但是我们是手动做的。现在，让我们暂时告一段落。在下一章中，我们将看看这其中有什么问题，为什么我们要手动调用它，以及我们可以做些什么。

# 使用 POST 方法的/api/posts

这将触发`PostController::store()`方法。我们错过的是验证。实际上，Lumen 还为我们提供了验证支持以及一些内置的验证规则。Lumen 的验证与 Laravel 非常相似，但也有一些不同之处。我建议您查看 Laravel 的验证文档，网址为[`laravel.com/docs/5.4/validation`](https://laravel.com/docs/5.4/validation)，以及 Lumen 与 Laravel 的验证差异：[`lumen.laravel.com/docs/5.4/validation`](https://lumen.laravel.com/docs/5.4/validation)。

在这里，我们在`store()`中添加了验证，因此在添加验证后查看代码，然后我们将讨论它：

```php
public function store(Request $request) {
  $input = $request->all();   $validationRules = [
        'content' => 'required|min:1',
        'title' => 'required|min:1',
        'status' => 'required|in:draft,published',
        'user_id' => 'required|exists:users,id'
    ];

    $validator = \Validator::make($input, $validationRules);
    if ($validator->fails()) {
        return new \Illuminate\Http\JsonResponse(
            [
                'errors' => $validator->errors()
            ], \Illuminate\**Http\**Response::HTTP_BAD_REQUEST
        );
 **}**   $post = $this->post->create($input);    return [
  'data' => $post
  ]; }
```

在这里，我们正在做 3 件事：

1.  首先，我们设置了以下验证规则：

1.  对于`content`和`title`，这些字段将是必需的，并且至少为 1 个字符长。

1.  对于`status`，它是必需的，其值可以是已发布或草稿，因为它在数据库中设置为 ENUM。

1.  `user_id`是必需的，并且应存在于`users`表的`id`字段中。

1.  然后，我们根据验证规则和输入创建了一个验证器对象，并检查验证器是否失败。否则，我们将继续进行。

1.  如果验证器失败，它将手动返回错误。它返回与验证器获得的相同错误描述，同时手动返回适当的响应代码。这就是为什么我们使用了`\Illuminate\Http\JsonResponse`。第一个参数是响应主体，而第二个参数是响应代码。而不是编写 400 错误代码，我们可以在`\Illuminate\Http\Response`中使用一个常量。因此，我们将不需要记住响应代码，阅读我们的代码的人也不需要知道 400 状态代码是什么。

请注意，错误代码、响应代码和 HTTP 代码表示相同的事情。因此，如果您看到它们，不要感到困惑，它们在本书中是可以互换使用的。

# 使用 GET 方法的/api/posts/1

这将由`show($id)`方法提供。在我们的 show 方法中，我们只是获取记录并返回，但是如果传递给 URL 中的`show()`方法的 ID 不正确，或者记录表明该 ID 不存在，该怎么办？因此，我们只需要放置一个检查以确保它返回 404 错误，如果找不到具有该 ID 的帖子。我们的`show()`方法的代码将如下所示：

```php
public function show($id) {
  $post = $this->post->find($id);
 if(!$post) {
        abort(404);
    }

    return $post**;** }
```

`abort()`方法将使用传递给它的错误代码停止执行。在这种情况下，它将简单地给出 404 Not Found 错误。

# 使用 PATCH/PUT 方法的/api/posts/1

它将由`update()`方法提供。同样，它是基于提供的 ID，因此我们需要检查该 ID 是否有效。因此，这就是我们将要做的事情：

```php
public function update(Request $request, $id) {
  $input = $request->all();    $post = $this->post->find($id);    if(!$post) {
 abort(404);
 }   $post->fill($input);
  $post->save();    return $post; }
```

在这里，我们使用了模型的`fill()`方法，它将使用$input 中的字段和值分配给 Post 模型，然后使用`save()`方法保存它。在 Laravel 文档中，您可以看到使用 Eloquent 以不同方式插入和更新的方法，这在不同的地方可能很方便：[`laravel.com/docs/5.4/eloquent#inserting-and-updating-models`](https://laravel.com/docs/5.4/eloquent#inserting-and-updating-models)。

有时，您会看到 Laravel 文档链接，而不是 Lumen。这是因为 Lumen 大多使用与 Laravel 相同的代码。所有这些组件的文档大多是在 Laravel 的文档中编写的，并且没有在 Lumen 文档中复制，因此 Lumen 文档在 Lumen 与 Laravel 不同的地方是很好的。

这就是我们在`update()`方法中要做的事情。

# 使用 DELETE 方法的/api/posts/1

删除操作将由`destroy($id)`提供。同样，它取决于来自 API 用户的 ID，因此我们需要放置与我们为`update()`和`show()`放置的类似检查。它将如下所示：

```php
public function destroy($id) {
 $post = $this->post->find($id);

    if(!$post) {
        abort(404);
    }

    $post**->delete();**    return ['message' => 'deleted successfully', 'post_id' => $id]; }
```

有了这个，我们的`PostController`将如下所示：

```php
<?php   namespace App\Http\Controllers;   use Illuminate\Http\Request; use Illuminate\Http\Response; use Illuminate\Http\JsonResponse;   class PostController extends Controller {    public function __construct(\App\Post $post)
 {  $this->post = $post;
 }    /**
 * Display a listing of the resource. * * @return \Illuminate\Http\Response
 */  public function index()
 {  $posts = $this->post->paginate(20);
  $data = $posts['data'];    $response = [
  'data' => $data,
  'total_count' => $posts['total'],
  'limit' => $posts['per_page'],
  'pagination' => [
  'next_page' => $posts['next_page_url'],
  'current_page' => $posts['current_page']
 ] ];    return $response;
 }    /**
 * Store a newly created resource in storage. * * @param \Illuminate\Http\Request  $request
 * @return \Illuminate\Http\Response
 */  public function store(Request $request)
 {  $input = $request->all();    $validationRules = [
  'content' => 'required|min:1',
  'title' => 'required|min:1',
  'status' => 'required|in:draft,published',
  'user_id' => 'required|exists:users,id'
  ];    $validator = \Validator::make($input, $validationRules);
  if ($validator->fails()) {
  return new JsonResponse(
 [  'errors' => $validator->errors()
 ], Response::HTTP_BAD_REQUEST
  );
 }   $post = $this->post->create($input);    return [
  'data' => $post
  ];
 }    /**
 * Display the specified resource. * * @param int  $id
 * @return \Illuminate\Http\Response
 */  public function show($id)
 {  $post = $this->post->find($id);    if(!$post) {
 abort(404);
 }    return $post;
 }    /**
 * Update the specified resource in storage. * * @param \Illuminate\Http\Request  $request
 * @param int  $id
 * @return \Illuminate\Http\Response
 */  public function update(Request $request, $id)
 {  $input = $request->all();    $post = $this->post->find($id);    if(!$post) {
 abort(404);
 }    $post->fill($input);
  $post->save();    return $post;
 }    /**
 * Remove the specified resource from storage. * * @param int  $id
 * @return \Illuminate\Http\Response
 */  public function destroy($id)
 {  $post = $this->post->find($id);    if(!$post) {
 abort(404);
 }    $post->delete();    return ['message' => 'deleted successfully', 'post_id' => $id];
 } } 
```

我们现在已经完成了返回适当的响应代码和验证等工作。

# 用户身份验证

到目前为止，我们还缺少用户身份验证。我们在输入中传递了`user_id`，这是错误的。我们之所以这样做是因为我们没有用户身份验证。因此，我们需要有一个身份验证机制。但是，除了身份验证之外，我们还需要有一个令牌生成机制。实际上，我们还需要刷新令牌。虽然我们可以自己做到这一点，但我们将安装另一个外部包。

在本章末尾开始用户身份验证并没有太多意义，因此我们将在下一章中处理用户身份验证，因为与之相关的事情有很多。

# 其他缺失的元素

我们现在缺少的其他东西如下：

+   API 版本控制

+   速率限制或节流

+   加密的需求

+   转换器或序列化

（这是为了避免在控制器内部进行硬编码手动返回格式）

在下一章中，我们将处理用户认证和前面提到的元素，并进行一些其他改进。

# 评论资源实现

我把评论端点的实现留给了你，因为它与帖子端点的实现非常相似。但是，由于评论的两个路由与其他路由不同，为了让你了解你需要实现什么，我将告诉你在`routes`文件中添加什么，以便你可以相应地实现`CommentController`。这是`routes/web.php`文件：

```php
<?php   /* |---------------------------------------------------------------- | Application Routes |---------------------------------------------------------------- | | Here is where you can register all of the routes for an application. | It is a breeze. Simply tell Lumen the URIs it should respond to | and give it the Closure to call when that URI is requested. | */     function resource($uri, $controller, $except **= []**) {
  //$verbs = ['GET', 'HEAD', 'POST', 'PUT', 'PATCH', 'DELETE'];    global $app;     if(!in_array('index', $except**)){**
  $app->get($uri, $controller.'@index');
 **}**   if(!in_array('store', $except)) {        $app->post($uri, $controller . '@store'); }

    if(!in_array('show', $except)) {        $app->get($uri . '/{id}', $controller . '@show'); };

    if(!in_array('udpate', $except)) {
        $app->put($uri . '/{id}', $controller . '@update');
        $app->patch($uri . '/{id}', $controller . '@update'); }

    if(!in_array('destroy', $except)) {        $app->delete($uri . '/{id}', $controller . '@destroy'); **}** }     $app->get('/', function () use ($app) {
  return $app->version(); });     resource('api/posts', 'PostController');  resource('api/comments', 'CommentController', ['store','index'**]);**   $app->post('api/posts/{id}/comments', $controller . '@store');
$app->get('api/posts/{id}/comments', $controller . '@index'**);**  
```

正如你所看到的，我们在`resource()`中添加了`$except`数组作为可选的第三个参数，这样如果我们不想为某个资源生成特定的路由，我们可以将其传递给`$except`数组。

在`CommentController`中，代码将与`PostController`非常相似，只是对于`store()`和`index()`，`post_id`将是第一个参数并将被使用。

# 总结

到目前为止，我们在一个名为 Lumen 的微框架中创建了 RESTful web 服务端点。我们创建了迁移、模型和路由。我实现了`PostController`，但留下了`CommentController`的实现给你。因此，我们可以看到，我们在第三章中讨论的许多与实现相关的问题，*创建 RESTful 端点*，已经得到解决，因为使用了一个框架。我们能够很容易地解决许多其他问题。因此，使用正确的框架和包，我们能够工作得更快。

我们还确定了一些缺失的元素，包括用户认证。我们将在下一章中解决它们。在下一章中，我们还将从代码方面改进我们的工作。

在本章中，我们主要使用 Lumen。我们看了它，但我们试图继续制作我们的 API，所以我们无法详细查看 Lumen 及其代码的每一个部分。因此，查看 Lumen 的文档是一个好主意：[`lumen.laravel.com/docs/5.4/validation`](https://lumen.laravel.com/docs/5.4)。

为了更好地理解，您应该查看 Laravel 的文档，因为一些常见组件在 Laravel 的文档中有详细解释：[`laravel.com/docs/5.4`](https://laravel.com/docs/5.4)。除了 Laravel 和 Lumen 的文档之外，建议去[`laracasts.com/`](http://laracasts.com/)观看关于 Laravel 的视频。如果在 Lumen 上找不到太多内容，不要担心，它与 Laravel 非常相似。除了一些变化，它们基本上是一样的。要了解 Laravel 和/或 Lumen，Lara casts 是一个非常好的资源，在 Laravel 社区非常受欢迎。Lara casts 主要由 Jeffrey Way 制作。我从他那里学到了很多东西，希望你也能学到。它不仅会教你 Laravel，还会教你如何开发，以及你应该如何进行开发。
