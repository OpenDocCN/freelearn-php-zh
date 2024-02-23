# 第四章。演示应用程序

现在我们已经介绍了 FuelPHP 框架，是时候开始构建一些东西了。我们将通过 Oil 命令行工具迁移，将所有内容整合到示例应用程序中。在本章中，我们将创建一个类似 WordPress 的简单博客应用程序。

用他们自己的话来说，WordPress 正在成为网络的操作系统。根据创始人 Matt Mullenweg 的说法：

> *"WordPress 现在占据了网络的 18.9%，已经有超过 4600 万次下载"*

访问以下链接以获取有关文章的更多信息：

[`thenextweb.com/insider/2013/07/27/wordpress-now-powers-18-9-of-the-web-has-over-46m-downloads-according-to-founder-matt-mullenweg/`](http://thenextweb.com/insider/2013/07/27/wordpress-now-powers-18-9-of-the-web-has-over-46m-downloads-according-to-founder-matt-mullenweg/)

现在，WordPress 的焦点远远超出了博客或简单的期刊。尽管是一个简单的应用程序或网站，创建一个简单的博客将演示 FuelPHP 的许多功能，从数据库迁移、扭转思想和代码到存储期刊条目修订的完整时间模型。

在开始编码之前，让我们先考虑要构建什么，以及最小可行产品是什么。

在本章中，我们将涵盖以下主题：

+   创建和运行数据库迁移

+   使用 Oil 创建模型

+   使用 Oil 创建控制器

+   安装和使用 HTML5 Boilerplate

+   将所有内容整合在一起，创建一个带有脚手架的管理系统

+   使用 Oil 命令行控制台

# 入门

就像 WordPress 一样，我们将从小处开始，演示一些 FuelPHP 工具，这些工具在使用 FuelPHP 开发项目时可以让您的生活更轻松。

在创建项目之前，首先创建一个源代码控制存储库。在这个示例中，我们将使用 GitHub（[`github.com`](https://github.com)），但是其他选择，如**Bitbucket**（[`bitbucket.org`](https://bitbucket.org)）或**beanstalk**（[`beanstalkapp.com`](http://beanstalkapp.com)）可能更适合您。

1.  首先，登录您的帐户，然后创建一个新的项目/存储库。对于这个应用程序，我们可以选择公共选项，因为它在 GitHub 上是免费的。在这个示例中，我们将称我们的项目为`journal`。确保记录页面右侧的存储库 URL，您很快会需要它。

1.  现在，让我们在开发机器上创建项目，但首先您需要使用以下命令行导航到您的`home`文件夹：

```php
$ cd ~/
```

1.  然后，使用以下命令导航到`Sites`文件夹：

```php
$ cd ~/Sites
```

1.  如果您没有`Sites`文件夹，让我们创建一个：

```php
$ mkdir ~/Sites
```

1.  在您的 Sites 文件夹中，运行以下 Oil 命令：

```php
$ php oil create journal
```

### 注

请注意，在这些示例中，$表示终端中新行的开始。运行命令时，您只需要`$`后面的文本，例如`php oil create journal`。

如果您在本地开发机器上使用 Apache，下一步将是为新站点创建虚拟主机（**vhosts**），然后修改 hosts 文件。您需要有一个带有`sites-available`文件夹和`sites-enabled`文件夹的 Apache 配置。然后，来自朋友的一个小脚本将证明是非常宝贵的。可以使用以下命令安装它：

```php
**$ sudo cd /usr/local/bin && sudo git clone git://github.com/maartenJacobs/quickhost.git quickhost files && sudo mv quickhost files/* . && sudo rm -Rf quickhost_files**

```

### 注

这个脚本是一组 PHP 函数，用于在您的计算机上设置 Apache vhosts 和 hosts 文件。所有操作都很简单，但它们可以让您更快地设置它。代码可以在以下链接公开查看：

[`github.com/maartenJacobs/quickhost`](https://github.com/maartenJacobs/quickhost)

或者更多信息可以在[`github.com/digitales/quickhost`](https://github.com/digitales/quickhost)找到。

安装脚本后，我们将能够运行以下一组命令来设置 Apache，以便我们可以使用本地域`http://journal.dev`：

```php
**$ cd ~/Sites/journal/public**
**$ sudo quickhost journal.dev**

```

使用`sudo`的原因是允许 Apache 优雅地重新启动并修改主机文件。这将将域名映射到本地主机 IP 地址（`127.0.0.1`）。在此示例中，我们选择了`.dev`顶级域（TLD）来区分它与生产和暂存环境。我们也可以选择`.local`，但这可能会在某些操作系统上（尤其是在 Mac OS X 上）与 Active Directory（[`en.wikipedia.org/wiki/Active_Directory`](http://en.wikipedia.org/wiki/Active_Directory)）发生冲突。

### 注意

这已在 Mac OS X 上进行了测试，应该可以在*nix 上运行，但在 Windows 环境中可能需要更改脚本才能运行。

我们需要一个数据存储来存储日志条目，所以现在让我们进行配置。加载`fuel/app/config/development/db.php`中找到的`db.php`配置。

如前所述，FuelPHP 具有环境的概念，因此我们可以将开发数据库配置添加到源代码控制中，而不会影响其他环境。因此，您的`db.php`文件应该如下所示：

```php
<?php
return array(
    'default' => array(
        'connection' => array(
            'dsn' => 'mysql:host=localhost;dbname=fue_dev',
            'username' => 'root',
            'password' => 'root',
        ),
        'profiling' => true,
    ),
);
```

### 注意

我倾向于为每个项目设置单独的数据库用户。这确保了没有项目可以触及另一个项目的数据库。在我的 db.php 中，我正在使用 journal_dev 的 dbname、用户名和密码，所以您可以更改您的 db.php 文件版本。

虽然不建议在生产环境中使用，但启用`profiling`可以在开发项目时提供帮助。示例配置已启用它。

我们有代码和存储库，现在让我们将它们链接起来：

```php
**$ cd ~/Sites/journal**
**$ git remote rm origin**
**$ git remote add origin <repository url>**
**$ git pull origin**
**$ git add ./**
**$ git commit -m 'Initial commit'**
**$ git push origin master**

```

用您的存储库 URL 替换上面的`<repository URL>`。现在，也是设置几个不同环境的分支的好时机：`develop`、`staging`和`production`：

```php
**$ git checkout -b develop**
**$ git push origin develop**
**$ git checkout -b staging**
**$ git push origin staging**
**$ git checkout -b production**
**$ git push origin production**
**$ git checkout develop**

```

继续设置主题，让我们设置数据库表。

# 创建数据库表

在数据库中创建表之前，让我们首先详细说明将存储哪些数据以及表中将有哪些初始字段。

对于博客，我们将有博客文章或条目，这些文章将有发布日期。它们将被分类以便对类似主题进行分组。每篇文章将与作者（用户）相关联，并且将具有内容和摘录。我们可以稍后添加自由形式的标记。

## 条目

`Entries`表中存在以下字段：

+   `id`（整数）：这将是主要标识符

+   `name`（varchar）

+   `slug`（varchar）

+   `excerpt`（文本）

+   `content`（文本）

+   `published_at`（整数）

+   `created_at`（时间戳）：这将是记录创建时的时间戳

+   `updated_at`（时间戳）- 这将是记录上次更新的时间戳

## 类别

`Categories`表中存在以下字段：

+   `id`（整数）：这将是主要标识符

+   `name`（varchar）

+   `slug`（varchar）

+   `created_at`（时间戳）：这将是记录创建时的时间戳

+   `updated_at`（时间戳）：这将是记录上次更新的时间戳

## 用户

`Users`表中存在以下字段：

+   `id`（整数）：这将是主要标识符

+   `name`（varchar）

+   `username`（varchar）

+   `password`（varchar）

+   `email`（varchar）

+   `last_login`（时间戳）

+   `login_hash`（文本）

+   `profile_fields`（文本）

+   `created_at`（时间戳）：这将是记录创建时的时间戳

+   `updated_at`（时间戳）：这将是记录上次更新的时间戳

我们还需要一个链接表，以便多个条目可以分类到相同的类别中。

## categories_entries

`categories_entries`表中存在以下字段：

+   `id`（整数）：这将是主要标识符

+   `category_id`（整数）：这将是类别的主要标识符

+   `entry_id`（整数）：这将是条目的主要标识符

+   `created_at`（时间戳）：这将是记录创建时的时间戳

+   `updated_at`（时间戳）：这将是记录上次更新的时间戳

你可能已经注意到每个表都有一个`id`、一个`created_at`表和一个`updated_at`字段。这些都是由 FuelPHP Oil 工具自动添加的，当链接或形成数据对象之间的关系时，它们可能会很有用。它们确实会增加一些额外的开销，但在这个阶段这不应该成为问题，因为额外的存储需求是很小的。

让我们使用 FuelPHP Oil 工具创建一些迁移：

```php
**$ php oil generate migration create_entries name:string slug:string excerpt:text content:text published_at:timestamp**

```

这将创建迁移以组装 posts 数据库表。不需要详细说明`id`、`created_at`或`updated_at`字段，因为这些将自动生成。如果你不需要这些额外的基于时间的字段，你可以在`generate`命令的末尾添加`--no-timestamp`。

### 提示

我们在这里运行迁移示例，并且在本章后面生成模型时将使用正确的 entries 表。

# 迁移和石油

在快速创建项目并组织基本结构时，FuelPHP Oil 工具可以处理一些更加重复的任务，让你可以集中精力处理项目的其他方面。这包括设置数据库表（如之前演示的）以及创建模型和控制器。FuelPHP Oil 工具还可以用来搭建一个管理系统，该系统已经准备好用于管理内容，并带有完整的用户认证系统。它将提供起点；虽然结果不总是美观的，但它是功能性的。

这使你可以快速测试想法并迭代向完美的代码。我们将逐个介绍 Oil 的各个功能，以创建模型、控制器和数据库迁移。这将给我们更多时间来调整和创建一个更加完善的项目/日志。

Oil 最有用的功能之一是迁移数据库结构的能力。这些可以用于确保不同环境之间的一致性。在部署代码时，也可以在数据库上运行迁移。

迁移可以用于重命名表、添加和删除字段甚至表。Oil 有魔术迁移的概念。这些以关键字前缀开始，使得很容易看出它们将要做什么：

```php
$ php oil generate migration create_users name:text
$ php oil generate migration rename_table_users_to_people
$ php oil generate migration add_profile_to_people profile:text
$ php oil generate migration delete_profile_from_people profile:text
$ php oil generate migration drop_people
```

### 提示

在创建迁移时，你需要注意关键字。关键字包括`create_`、`rename_`、`add_`、`delete_`、`rename_`和`drop_`。

迁移存储在`fuel/app/migrations`中，已执行的迁移列表存储在`fuel/app/config/<environment>/migrations.php`中。

此外，迁移的数据库表记录了项目的当前状态。运行迁移就像输入以下命令一样简单：

```php
**$ oil refine migrate**

```

### 提示

你可以简单地输入`r`和`g`来运行 refine 和 generate Oil 操作。

如果你想要迁移上升或下降，请运行以下命令：

```php
**$ oil r migrate:up**
**$ oil r migrate:down**

```

这些命令可以被整合到部署脚本中，这样迁移就可以在将新代码添加到不同环境时自动运行。这对避免在生产环境中出现错误特别有用。

在继续生成模型及其数据库表之前，让我们删除之前创建的 posts 表：

```php
**$ php oil g migration drop_posts**
**$ php oil r migrate**

```

# 模型

现在我们对迁移和 Oil 有了简要的了解，让我们开始使用 Oil。我们将使用以下命令创建一个模型以及相应的数据库表细节：

```php
**$ php oil generate model entry name:string slug:string excerpt:text content:text published_at:timestamp**

```

这将创建模型和迁移。让我们打开新创建的`entry.php`模型，位于`fuel/app/classes/model/`：

```php
<?php
class Model_Entry extends \Orm\Model
{
    protected static $_properties = array(
        'id',
        'name',
        'slug',
        'excerpt',
        'content',
        'published_at',
        'created_at',
        'updated_at',
    );

    protected static $_observers = array(
        'Orm\Observer_CreatedAt' => array(
            'events' => array('before_insert'),
            'mysql_timestamp' => false,
        ),
        'Orm\Observer_UpdatedAt' => array(
            'events' => array('before_save'),
            'mysql_timestamp' => false,
        ),
    );

    protected static $_table_name = 'entries'
}
```

你会注意到新生成的模型正在扩展 ORM 模型。为了使其工作，我们需要在自动加载器中包含 ORM 包。这是一个很好的机会来介绍 FuelPHP 自动加载器配置。加载位于`fuel/app/config/config.php`的配置文件。查找`'always load'`部分，并取消注释`'always_load'`数组，如下所示：

```php
'always_load' => array(
    'packages' => array(
        'orm',
    )
),
```

当您安装新的包时，只需将它们添加到`always_load`数组中以自动加载它们。有时，一个包只在某个方法或类中需要。然而，在这种情况下，我们可以只在需要时加载包。关于这方面的更多信息可以在以下链接找到：

[`fuelphp.com/docs/classes/package.html#method_load`](http://fuelphp.com/docs/classes/package.html#method_load)

现在，让我们回到`entry`模型。您会注意到`$_properties`数组，这将使从模型层调用变量变得更容易。如果您添加了新的数据库字段，它将需要添加到`properties`数组中。ORM 包括观察者的概念，这些是可以在不同时间执行方法或函数的操作。模型中使用的是`before_insert`和`before_update`。目前正在使用它们来在添加或更新条目时创建时间戳。

其他观察者在文档中列出；其中一个对我们有用的是`Observer_Slug`。这个观察者将处理将条目标题转换为 URL 安全，并避免重写现有功能的需要。让我们通过将以下片段添加到`observers`数组中，将这个观察者添加到我们的 entry 模型中：

```php
'Orm\Observer_Slug' => array(
    'events' => array( 'before_insert', 'before_update'),
    'source' => 'name',
    'property' => 'slug',

),
```

现在，当我们保存一个条目时，slug 将自动更新。

我们还需要创建一些其他数据库表。让我们现在这样做，然后我们可以创建它们的模型之间的关系，以使我们的生活更轻松：

```php
**$ php oil generate model category name:varchar slug:varchar**
**$ php oil generate model categories_entries category_id:int entry_id:int**

```

对于`entry`模型，将`Observer_Slug`添加到新的类别模型中。

对于`user`表，我们将使用 Auth 包，因为它包括用户认证功能。为此，我们需要将默认的 Auth 包配置复制到我们的应用程序配置中。

```php
**$ cp ~/Sites/journal/fuel/packages/auth/config/auth.php ~/Sites/journal/fuel/app/config/auth.php**

```

在这个阶段，我们应该在`auth.php`配置文件中更改盐值。完成后，我们必须运行以下命令来创建必要的`auth`表：

```php
**$ php oil refine migrate --packages=auth**

```

让我们运行迁移工具，确保数据库已更新：

```php
**$ php oil r migrate**

```

现在我们已经建立了数据库结构，我们可以将模型之间的关系联系起来。这是由 ORM 模型实现的，它原生支持以下关系：

+   `belongs_to`：该模型具有关系的主键，它属于一个相关对象。

+   `has_one`：它的主键存储在属于该模型的另一个表中。它只有一个关系。

+   `has_many`：它的主键保存在另一个模型的多个结果中。每个其他模型都需要属于这个模型。它可以有许多关系。

+   `many_to_many`：这使用一个链接表跟踪多对多关系。这个表记录了被链接表的主键对。它可以有，并且属于，许多模型。例如，一个博客文章可以有许多类别，一个类别可以链接到许多博客文章。

对于日志，我们将使用`belongs_to`，`has_one`和`many_to_many`关系。充分利用关系只需要快速添加到已创建的模型中。在您喜欢的文本编辑器中加载 entry 模型。该模型可以在`fuel/app/classes/model/entry.php`中找到。

```php
<?php

class Model_Entry extends \Orm\Model
{
    protected static $_properties = array(
        'id',
        'name',
        'slug',
        'excerpt',
        'content',
        'published_at',
        'created_at',
        'updated_at',
    );

    protected static $_observers = array(
        'Orm\Observer_CreatedAt' => array(
            'events' => array('before_insert'),
            'mysql_timestamp' => false,
        ),
        'Orm\Observer_UpdatedAt' => array(
            'events' => array('before_update'),
            'mysql_timestamp' => false,
        ),
        'Orm\Observer_Slug' => array(
            'events' => array('before_insert'),
            'source' => 'name',
            'property' => 'slug',
        ),
    );
    protected static $_table_name = 'entries';
}
```

在关闭大括号之前，将以下代码添加到模型中：

```php
 protected static $_belongs_to = array(
            'category' => array(
                'key_from' => 'id',
                'model_to' => 'Model_Category_Entry',
                'key_to' => 'entry_id',
                'cascade_save' => false,
                'cascade_delete' => false,
            ),
        );

    protected static $_has_many = array(
        'categories' => array(
            'key_from' => 'id',
            'model_to' => 'Model_Category_Entry',
            'key_to' => 'entry_id',
            'cascade_save' => false,
            'cascade_delete' => false,
        )
    );
```

我们并不总是需要包含完整的`$_belongs_to`声明。在大多数情况下，我们可以简单地使用以下内容：

```php
protected static $_belongs_to = array('post');
```

通过这些添加，我们正在配置 ORM 模型，将条目和类别之间的关系视为多对多关系。我们不是直接将类别链接到条目，而是使用`category_entry`链接表。这个表的命名是使用要链接的表的单数版本按字母顺序排列；在这种情况下，`categories`和`entries`的连接给我们一个表名`category_entry`。我们想要使用链接表的原因是因为我们希望能够为每个条目分配多个类别，这样就可以重复使用类别而不需要重复。例如，如果我们检索一个条目，类别列表将以以下方式获取：

```php
**$entry = Model_Entry::find(1)**

**$categories =$entry->categories;**

```

在这个例子中，我们正在寻找 ID 设置为`1`的条目。如果找到，将返回完整的条目对象，否则将返回 false。

除了多对多的关系，让我们将用户链接到他们撰写的条目。

首先，我们需要将`user_id`添加到条目表中：

```php
**$ php oil g migration add_user_id_to_entry user_id:int**
**$ php oil r migrate**

```

我们需要将`user_id`添加到条目模型的`$_properties`中：

```php
protected static $_properties = array(
        'id',
        'name',
        'slug',
        'excerpt',
        'content',
        'published_at',
        'created_at',
        'updated_at',
    );
```

由于关系从`user_id`映射到`user`表中的主`id`键，我们可以将用户定义为`$_belongs_to`数组的一部分。一旦将互补关系添加到`Model_User`模型中，其他所有内容都会就位：

```php
$_belongs_to = array( 
    'user',
    'category' => ...
);
```

谈到`Model_User`，需要创建它：

```php
**$ php oil g model user username:string password:string group:string email:string last_login:int profile_fields:text --no-migration**

```

尽管大多数用户功能都是通过 Auth 包内完成的，但添加用户模型对于与用户数据的`Model_Entry`关系是必要的。因此，加载位于`fuel/app/classes/model/`的`user.php`用户模型，并添加以下数组：

```php
protected static $_has_many = array('entry');
```

`category`和`category_entry`模型也需要创建关系。要做到这一点，导航到位于`fuel/app/class/mode/`的`category.php`文件：

```php
<?php

class Model_Category extends \Orm\Model
{
    protected static $_properties = array(
        'id',
        'name',
        'slug',
        'created_at',
        'updated_at',
    );

    protected static $_observers = array(
        'Orm\Observer_CreatedAt' => array(
            'events' => array('before_insert'),
            'mysql_timestamp' => false,
        ),
        'Orm\Observer_UpdatedAt' => array(
            'events' => array('before_update'),
            'mysql_timestamp' => false,
        ),
    );
    protected static $_table_name = 'categories';

    protected static $_belongs_to = array(
            'entry' => array(
                'key_from' => 'id',
                'model_to' => 'Model_Category_Entry',
                'key_to' => 'category_id',
                'cascade_save' => false,
                'cascade_delete' => false,
            ),
        );

    protected static $_has_many = array(
        'entries' => array(
            'key_from' => 'id',
            'model_to' => 'Model_Category_Entry',
            'key_to' => 'category_id',
            'cascade_save' => false,
            'cascade_delete' => false,
        )
    );
}
```

以下是位于`fuel/app/classes/model/category/`的`entry.php`文件的列表：

```php
<?php

class Model_Category_Entry extends \Orm\Model
{
    protected static $_properties = array(
        'id',
        'category_id',
        'entry_id',
        'created_at',
        'updated_at',
    );

    protected static $_observers = array(
        'Orm\Observer_CreatedAt' => array(
            'events' => array('before_insert'),
            'mysql_timestamp' => false,
        ),
        'Orm\Observer_UpdatedAt' => array(
            'events' => array('before_update'),
            'mysql_timestamp' => false,
        ),
    );
    protected static $_table_name = 'category_entries';

    protected static $_belongs_to = array(
        'category',
        'entry',
    );
}
```

现在我们有了模型和迁移，将它们添加到源代码控制是个好主意。一旦完成，让我们开始创建控制器来使用模型，然后创建视图（包括表单）。

# 控制器

控制器和模型一样，可以使用 Oil 创建。主要的控制器将用于管理条目和类别。这将有两种类型，公开可见的站点控制器和条目和类别表的管理系统。

我们的控制器可能最初不需要处理 RESTful 请求。因此，我们应该扩展`Controller_Template`，幸运的是，这正是 Oil 工具设置运行的方式。当我们使用 Oil 创建控制器时，它将被创建为扩展模板控制器。我们只需要考虑我们需要做什么操作和方法。因为我们主要将显示博客的条目和类别信息，所以让我们从`index`和`view`操作开始：

```php
**$ php oil g controller entry index view**

```

这将创建控制器、模板文件和所需方法的视图——`index`和`view`。

让我们看一下位于`fuel/app/classes/controller/`的`entry.php`：

```php
<?php

class Controller_Entry extends Controller_Template { 
    public function action_index()
    {
        $data['entries'] = Model_Entry::find('all');
        $this->template->title = "Entries";
        $this->template->content = View::forge('entry/index', $data);

    }

    public function action_view($id = null)
    {
        is_null($id) and Response::redirect('entry');

        if ( ! $data['entry'] = Model_Entry::find($id))
        {
            Session::set_flash('error', 'Could not find entry #'.$id);
            Response::redirect('entry');
        }
        $this->template->title = "Entry";
        $this->template->content = View::forge('entry/view', $data);

    }
}
```

在生成的控制器中有一些有趣的事情需要注意。首先是每个直接与 URL 相关的方法都以`action_`为前缀。这样可以更容易地看出哪些方法直接与 URL 相关，哪些方法涵盖了控制器内的其他功能

您还会注意到活动视图被传递到`$data`数组中的视图中。这用于将变量传递给视图，允许在视图渲染之前进行逻辑处理。这样视图就纯粹用于呈现，就像它们应该的那样。`data`数组中的键将被转换为视图中的变量，例如，`$data['subnav']`将通过视图中的`$subnav`变量调用。

`Controller_Template`类还有另外两个方法——`before()`和`after()`——这两个方法非常有用。`before()`方法可用于管理系统的用户身份验证，这将在本章后面进行演示。我们将在管理系统中使用它，以确保只有经过身份验证的用户才能访问管理系统。在控制器中使用`before()`方法时，请确保调用`parent::before()`方法，例如：

```php
class Controller_Entry extends Controller_Template
{
    // Essential, don't forget this method.
    public function before()
    {
        parent::before();

        // Your code
    }    
    .....
}
```

# 视图

使用 Oil 工具创建控制器时，还会创建模板和视图。让我们来看看其中一个视图，加载`fuel/app/views/entry/index.php`。

在视图中，你会注意到缺少开头和结尾的 HTML body。`template.php`文件位于 views 文件夹的根目录中，负责处理演示元素的测试。在查看`template.php`之前，让我们讨论一下`index.php`文件的一些部分：

```php
Arr::get( $subnav, "index");
```

如前面的代码行所示，它使用了核心 FuelPHP `Arr`类，这是一组用于处理数组的辅助函数。在这种情况下，使用了`get`方法。这允许您检查数组中是否存在给定的键，如果找不到键，则返回 false。视图正在使用此功能来输出活动页面/视图的`'active'`样式类。

第二个正在使用的核心类是`Html::anchor()`。这个类提供了大量的 HTML 标签，并确保所有使用的标签都符合`Doctype`声明。我们使用`anchor`方法来输出到视图的链接。第一个值是链接，第二个是显示标题（在`a`标签之间的部分）。使用这个辅助方法的一个原因是确保链接在任何环境的 URL 下都能正常工作，无论应用程序是安装在子文件夹还是子域中。现在，让我们快速看一下`fuel/app/views/template.php`中的`template.php`文件。

默认情况下，模板使用 Twitter bootstrap 为项目提供快速起点。FuelPHP 期望项目的 public 文件夹中有 CSS 和 JS 文件夹。加载 CSS 和 JavaScript 文件就像在模板中使用以下代码一样简单：

```php
<?php echo Asset::css('bootstrap.css');?>
<?php echo Asset::js('bootstrap.js'); ?>
```

与 Ruby on Rails 等框架一样，FuelPHP 实现了 flash 会话，用于在页面加载之间传递变量和值。默认情况下，FuelPHP 有两个 flash 会话，默认命名为`success`和`error`。当然，您可以根据需要向 FuelPHP 添加任意数量的 flash 会话。以下是一个示例：

```php
**<?php Session::set_flash('warning', 'Warning Message') ?>**

```

以下示例是模板中输出 flash 会话的两种方式：

```php
<?php if (Session::get_flash('success')): ?>
    <div class="alert alert-success">
        <strong>Success</strong>
        <p>
            <?php echo implode('</p><p>', e((array) Session::get_flash('success'))); ?>
        </p>
    </div>
<?php endif; ?>
```

`template.php`的其余部分是相当简单的 HTML。创建能够响应访问者屏幕尺寸的网站，正在成为支持移动设备的一种非常流行的方式。因此，在为桌面添加布局之前，我们应该首先考虑移动尺寸的屏幕。为了帮助实现这一点，我们将使用 HTML5 Boilerplate ([`html5boilerplate.com`](http://html5boilerplate.com))。我们将自定义 Boilerplate 的版本，可以通过[`www.initializr.com/`](http://www.initializr.com/)伴侣网站完成。要做到这一点，请执行以下步骤：

1.  在 Initializr 网站上，应该有 3 个选项：Classic H5BP、Responsive 和 Bootstrap。选择**Classic H5BP**选项（灰色按钮），然后选择**Responsive Bootstrap** 2.3.2 模板选项。最后点击**Download it!**。

1.  将下载的文件复制到您的日志应用程序的 public 文件夹中。将`js`、`css`和`img`文件夹复制到您的 assets 文件夹中。

1.  现在，打开文本编辑器中的`index.html`文件并进行编辑。下一步是从`index.html`文件中提取元素并将其添加到`template.php`文件中。

以下是最终的示例：

```php
<!DOCTYPE html>
<!--[if lt IE 7]>      <html class="no-js lt-ie9 lt-ie8 lt-ie7"> <![endif]-->
<!--[if IE 7]>         <html class="no-js lt-ie9 lt-ie8"> <![endif]-->
<!--[if IE 8]>         <html class="no-js lt-ie9"> <![endif]-->
<!--[if gt IE 8]><!--> <html class="no-js"> <!--<![endif]-->
    <head>
        <meta charset="utf-8">
        <meta http-equiv="X-UA-Compatible" echo Asset::css('bootstrap.min.css'); ?>
content="IE=edge,chrome=1">
        <title><?php echo $title; ?></title>
        <meta name="description" content="">
        <meta name="viewport" content="width=device-width">
        <?php
        <style>
            body {
                padding-top: 60px;
                padding-bottom: 40px;
            }
        </style>
        <?php echo Asset::css('bootstrap-responsive.min.css'); ?>
        <?php echo Asset::css('main.css'); ?>
        <?php Asset::add_path('assets/js/vendor/', 'js'); ?>
        <?php echo Asset::js('modernizr-2.6.2.min.js'); ?>
    </head>
    <body>
        <!--[if lt IE 7]>
            <p class="chromeframe">You are using an <strong>outdated</strong> browser. Please <a href="http://browsehappy.com/">upgrade your browser</a> or <a href="http://www.google.com/chromeframe/?redirect=true">activate Google Chrome  Frame</a> to improve your experience.</p>
        <![endif]-->
        <div class="navbar navbar-inverse navbar-fixed-top">
            <div class="navbar-inner">
                <div class="container">
                    <a class="btn btn-navbar" data-toggle="collapse" data-target=".nav-collapse">
                        <span class="icon-bar"></span>
                        <span class="icon-bar"></span>
                        <span class="icon-bar"></span>
                    </a>
                    <a class="brand" href="/">Journal</a>
                    <div class="nav-collapse collapse">
                        <ul class="nav">
                          <li class="active"><a href="/">Home</a></li>
                          <li><a href="/entry">Entries</a></li>
                          <li><a href="/category">Categories</a></li>
                        </ul>
                    </div><!--/.nav-collapse -->
                </div>
            </div>
        </div>
        <div class="container">
            <h1><?php echo $title; ?></h1>
            <?php if (Session::get_flash('success')): ?>
                <div class="alert alert-success">
                    <strong>Success</strong>
                    <p>
                    <?php echo implode('</p><p>', e((array) Session::get_flash('success'))); ?>
                    </p>
                </div>
            <?php endif; ?>
            <?php if (Session::get_flash('error')): ?>
                <div class="alert alert-error">
                    <strong>Error</strong>
                    <p>
                    <?php echo implode('</p><p>', e((array) Session::get_flash('error'))); ?>
                    </p>
                </div>
            <?php endif; ?>
            <hr>
            <div class="span12">
                <?php echo $content; ?>
            </div>

            <footer>
                <p>&copy; Journal <?php echo date( 'Y' ); ?></p>
            </footer>

        </div> <!-- /container -->

        <script src="//ajax.googleapis.com/ajax/libs/jquery/1.10.1/jquery.min.js"></script>
        <script>window.jQuery || document.write('<script src="js/vendor/jquery-1.10.1.min.js"><\/script>')</script>
        <?php echo Asset::js('bootstrap.min.js'); ?>
        <?php echo Asset::js('plugins.js'); ?>
        <?php echo Asset::js('main.js'); ?>
    </body>
</html>
```

### 提示

完成`index.html`文件后，请确保从公共文件夹中删除它；否则，您可能会在加载日志网站时遇到问题。

`template.php`文件的大部分内容已经在前面提到过，除了`Asset::set_path()`。HTML5 Boilerplate 包括`modernizr` JavaScript 文件的`vendor`文件夹。默认情况下，Asset 加载器将查找位于`public/assets`的`css`和`js`文件夹，但不会查看这些目录中的文件夹。我们使用`set_path()`方法将`vendor`文件夹包含在 JavaScript 加载中。`set_path()`中的第二个变量让 Asset 加载器知道新路径是一个 JavaScript 文件夹。

现在，我们有机会在浏览器中检查网站，例如`http://journal.dev/entry`或`http://localhost/Site/journal/entry`或`http://127.0.0.1/Site/journal/entry`，具体取决于您的开发机器配置。

目前，访问者将看到的第一个网页是 FuelPHP 欢迎页面。我们应该更改这一点，使得访问者首先看到条目列表页面。这样做很快，可以很容易地通过 FuelPHP 中的路由来完成。加载位于`fuel/app/config/`的`routes.php`配置文件。

```php
<?php

return array(
    '_root_'  => 'welcome/index',  // The default route
    '_404_'   => 'welcome/404',    // The main 404 route
    'hello(/:name)?' => array('welcome/hello', 'name' => 'hello'),
);
```

您可以通过将`'welcome/index'`更改为`'entry/index'`来将欢迎页面更改为条目列表页面。

回到浏览器，在日志网站的顶层导航到，例如`http://journal.dev`或`http://localhost/Site/journal`或`http://127.0.0.1/Site/journal`。您会注意到您的条目索引页面正在显示。

到目前为止，我们已经涵盖了控制器和视图的基本部分，并安装了 HTML5 Boilerplate。现在，我们应该看一下条目和类别的管理。

在查看管理系统之前，公共网站上还有一件事要做——类别。让我们生成类别索引和查看控制器：

```php
**$ php oil g controller category index view**

```

您可能认为能够显示条目和类别很好，但我们实际上如何处理管理系统呢？下一节将教会我们如何做到这一点。

# 使用 Oil 生成管理系统

我们将使用 Oil 工具快速构建一个条目和类别的管理系统。

除了创建迁移和控制器之外，Oil 还可以用来为您搭建功能。这有两种风格：一种是用于前端，就像我们已经看到的那样，另一种是具有完全控制的管理系统。

在继续之前，我们需要重命名`category`和`entry`模型。这是因为它们将作为管理系统脚手架的一部分重新创建。我们将它们重命名，以便我们有之前在这些模型中设置的关系的副本：

```php
**$ mv ~/Sites/journal/fuel/app/classes/model/entry.php ~/Sites/journal/fuel/app/classes/model/entry.bak.php**
**$ mv ~/Sites/journal/fuel/app/classes/model/category.php ~/Sites/journal/fuel/app/classes/model/category.php**

```

现在，让我们运行管理脚手架 Oil 命令：

```php
**$ php oil g admin entry name:varchar slug:varchar excerpt:text content:text published_at:int user_id:int**
**$ php oil g admin category name:varchar slug:varchar -force**

```

`-force`元素是必需的，以便重新运行管理系统中的共享文件，以便创建类别文件。

Oil 已经为您创建了一些文件，但不幸的是，它不知道类别和条目的关系，也不知道`Observer_Slug`属性。因此，我们需要关联回`Entry 和 Category`模型，我们还需要添加回`Observer_Slug`。这个过程将与最近重命名的条目和类别模型一样简单。

在编辑模型时，我们需要删除或注释掉以下内容：

```php
**$val->add_field( 'slug', 'Slug', 'required|max_length[255]');**

```

我们还需要在管理条目和类别控制器中做同样的事情。在模型中定义的`Observer_Slug`将处理 slug。

因此，您需要检查位于`fuel/app/classes/controller/admin/`的`category.php`的第 32、66 和 86 行，并且还需要检查位于相同文件夹中的`entry.php`的第 32、69 和 92 行。

作为脚手架过程的一部分，已经创建了一些表单，并在条目和类别视图的创建和编辑之间共享。加载`fuel/app/views/admin/category/_form.php`和`fuel/app/views/admin/entry/_form.php`。在两个表单中，注释掉对 slug 的引用，因为它不需要。

在浏览器中，导航到`http://journal.dev/admin`或`http://localhost/Site/journal/admin`或`http://127.0.0.1/Site/journal/admin`，您将被重定向到登录表单。我们还没有设置用户帐户，但 Oil 可以帮助解决问题。

加载您的终端并运行：

```php
**$ php oil console**

```

这将启动一个交互式控制台，可以利用核心 FuelPHP 和包代码。在这种情况下，我们需要调用 Auth 包来创建一个新用户。

```php
>>> Auth::create_user('admin', 'password', 'email@domain.com', 100 );
1
>>> exit;
```

此命令将输出新创建帐户的`user_id`。`100`代表 Auth 包中的最高默认用户特权和角色。要退出控制台会话，只需输入`exit`并运行。

现在，您将能够登录到您新创建的管理系统。从那里，您将能够添加新的类别和条目。

# 总结

本章简要介绍了项目结构以及 FuelPHP Oil 命令行工具如何帮助您快速设置。

Oil 充当脚手架结构，但您仍然需要填补空白。例如，可以从博客中检索链接的条目和类别，但我们仍然需要构建必要的管理控件来设置关系。

在下一章中，我们将更详细地了解包，并创建我们自己的包，可以与社区共享。
