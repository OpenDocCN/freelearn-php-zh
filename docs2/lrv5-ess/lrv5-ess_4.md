# 第四章 Eloquent ORM

在上一章中，我们提到了与 Laravel 一起提供的 **对象关系映射器**（ORM）Eloquent。Eloquent 在我们的应用程序中充当模型层（MVC 中的 M）。由于它是大多数在 Laravel 中构建的应用程序的一个重要部分，我们将更详细地探讨 Eloquent。

在本章中，我们将涵盖以下主题：

+   读取和写入数据库数据

+   模型之间的关系

+   查询作用域

+   模型事件和观察者

+   集合

# Eloquent 约定

Eloquent 有一些约定，遵循这些约定可以使你的生活更轻松。这种方法被称为 **约定优于配置**，这意味着如果你遵循这些约定，你将几乎不需要进行配置，事情就能“自然而然”地工作。

Eloquent 模型包含在一个单独的类中，并且是数据库表名称的单数“大驼峰式”版本。大驼峰式类似于驼峰式，但第一个字母也是大写。所以如果你有一个名为 `cats` 的数据库表，那么你的模型类将被称为 `Cat`。

在文件系统中没有固定的位置来放置你的 Eloquent 模型；你可以自由地按你的意愿组织它们。你可以使用 Artisan 命令来创建一个模型 **模板**（一个具有 Eloquent 模型基本结构的简单类）。命令如下：

```php
$ php artisan app:model Cat

```

默认情况下，Artisan 将新的模型类放在 `app` 目录中。你可以自由地移动你的模型类并将它们存储在你想要的任何目录中，只需确保更新文件顶部的命名空间声明以反映其新位置。

我们的模式模板类看起来像这样：

```php
<?php namespace App;
use Illuminate\Database\Eloquent\Model;
class Cat extends Model {
  //
}
```

这将默认尝试使用名为 `cats` 的表。

我们的模式类扩展了 Eloquent 的基本 `Model` 类，其中包含我们在本章中将使用到的所有功能。在创建模型后，你应该做的第一件事是定义它映射到的数据库表。在我们的例子中，数据库表将被称为 `cats`：

```php
class Cat extends Model {
  protected $table = 'cats';
}
```

这是一个简单的、可工作的 Eloquent 模型，你现在可以使用它来从你的数据库表中获取记录。

# 获取数据

Eloquent 提供了多种方式来从数据库中获取记录，每种方式都有其合适的用例。你可以一次性获取所有记录；基于主键获取单个记录；基于条件获取记录；或者获取所有或过滤记录的分页列表。

要获取所有记录，我们可以使用名为 `all` 的方法：

```php
use App\Cat;
$cats = Cat::all();
```

要通过主键获取记录，你可以使用 `find` 方法：

```php
$cat = Cat::find(1);
```

除了 `first` 和 `all` 方法之外，还有 **聚合** 方法。这些方法允许你从数据库表中检索聚合值（而不是记录集）：

```php
use App\Order;

$orderCount    = Order::count();
$maximumTotal  = Order::max('amount');
$minimumTotal  = Order::min('amount');
$averageTotal  = Order::avg('amount');
$lifetimeSales = Order::sum('amount');
```

## 过滤记录

Eloquent 还提供了一套功能丰富的查询构建器，允许你在代码中构建查询，而无需编写任何 SQL 代码。这个抽象层使得在需要时更容易切换数据库平台。使用 Laravel，你只需要更新你的数据库配置，你的应用程序将继续像以前一样运行。

Laravel 的查询构建器提供了常见的 SQL 类型的指令方法，如 `WHERE`、`ORDER` 和 `LIMIT`；以及更高级的概念，如连接。例如，之前的 `find` 示例可以表达为——尽管是冗长的——：

```php
$cat = Cat::where('id', '=', 1)->first();
```

这将检索第一个记录 `WHERE 'id' = 1`。当我们基于主键进行查询时，我们只期望一个记录，所以使用 `first` 方法。如果我们有一个更开放的 `WHERE` 子句，我们期望可能有多条记录，我们可以使用 `get` 方法，就像我们在第一个代码示例中所做的那样，它将只返回匹配该子句的记录。

条件也可以**链式**使用。这允许你通过添加条件来构建复杂的查询条件。考虑以下示例代码：

```php
use App\User;

$users = User::where('gender', '=', 'Male')
  ->where('birth_date', '>', '1989-02-12')
  ->all();
```

这将找到所有在 1989 年 2 月 12 日之后出生的男性用户。除了手动指定日期外，我们还可以使用 **Carbon**，一个日期和时间库。以下是一个使用 Carbon 查找所有 21 岁以上用户的示例：

```php
use App\User;
use Carbon\Carbon;

$users = User::where('birth_date', '<', Carbon::now()- >subYears(21))
  ->all();
```

### 注意

你可以在 Carbon 的官方 GitHub 仓库中找到更多关于 Carbon 及其可用函数的信息 [`github.com/briannesbitt/Carbon`](https://github.com/briannesbitt/Carbon)。Carbon 的常见方法也在 附录 *An Arsenal of Tools* 中进行了介绍。

除了通过 `WHERE` 条件过滤记录外，你还可以使用 `take` 方法通过范围限制记录数量：

```php
$women = User::where('gender', '=', 'Female')->take(5)->get();
```

这将获取前五个女性用户。你还可以使用 `skip` 方法指定偏移量：

```php
$women = User::where('gender', '=', 'Female')->take(5)->skip(10)->get();
```

在 SQL 中，这看起来类似于以下：

```php
SELECT * FROM users WHERE gender = 'Female' OFFSET 10 LIMIT 5
```

查询也可以通过使用 `orderBy` 方法进行排序：

```php
$rankings = Team::orderBy('rating', 'asc')->get();
```

这将对应于一个看起来像这样的 SQL 语句：

```php
SELECT * FROM teams ORDER BY rating ASC
```

# 保存数据

显示数据的应用程序很棒，但它们并不非常交互式。当允许用户提交数据时，乐趣才开始，无论这些用户是受信任的贡献者通过内容管理系统添加内容，还是来自像维基百科这样的网站的一般用户。

当你通过 Eloquent 获取记录时，你可以按照以下方式访问其属性：

```php
$cat = Cat::find(1);
print $cat->name;
```

我们也可以以同样的方式更新属性值：

```php
$cat->name = 'Garfield';

```

这将在模型实例中设置值，但我们需要将更改持久化到数据库中。我们通过之后调用 `save` 方法来完成此操作：

```php
$cat->name = 'Garfield';
$cat->save();

```

如果你有一个包含许多列的表格，那么手动为每个属性分配值将会变得非常繁琐。为此，Eloquent 允许你通过传递一个包含值的关联数组，并使用键来表示列名，来填充模型。你可以在创建或更新模型时填充模型：

```php
$data = [
  'name' => 'Garfield',
  'birth_date' => '1978-06-19',
  'breed_id' => 1,
];

$cat->create($data);
```

然而，这将抛出`MassAssignmentException`错误。

## 批量赋值

上述示例是一个**批量赋值**的例子。这就是模型属性被盲目地批量更新。如果前一个例子中的`$data`数组来自用户表单提交，那么它们可以更新数据库中相同表中的任何和所有值。

假设你有一个名为`users`的表，其中有一个名为`is_admin`的列，该列确定用户是否可以查看你的网站管理区域。还假设你的网站公共侧面的用户可以更新他们的个人资料。如果在表单提交期间，用户还包含一个名为`is_admin`且值为`1`的字段，那么这将更新数据库表中的列值，并授予他们访问你的超级秘密管理区域——这是一个重大的安全问题，这正是批量赋值保护所防止的。

为了标记可以通过批量赋值安全设置的列（例如`name`、`birth_date`等），我们需要通过提供一个名为`$fillable`的新属性来更新我们的 Eloquent 模型。这只是一个包含可以通过批量赋值安全设置的属性名称的数组：

```php
class Cat extends Model {

  protected $table = 'cats';
  protected $fillable = [
 'name',
 'birth_date',
 'breed_id',
 ];
}
```

现在，我们可以通过传递一个数据数组的方式创建和更新模型，就像之前一样，而不会遇到抛出`MassAssignmentException`异常的情况。

除了创建新记录之外，还有一些你可以使用的兄弟方法。有`firstOrCreate`，你可以传递一个数据数组——Eloquent 会首先尝试找到具有匹配值的模型。如果找不到匹配项，它将创建记录。

还有类似命名的`firstOrNew`方法。然而，它不会立即将记录保存到数据库中，而是会返回一个新的 Eloquent 实例，其中设置了属性值，允许你在手动保存之前先设置其他任何值。

使用这些方法的好时机是当允许用户通过使用第三方服务（如 Facebook 或 Twitter）登录时。这些服务通常会返回识别用户的个人信息，例如电子邮件地址，允许你检查数据库中是否存在匹配的用户。如果存在，你可以简单地让他们登录，否则你可以为他们创建一个新的用户账户。

# 删除数据

删除记录有两种方式。如果你已经从数据库中获取了一个模型实例，那么你可以在其上调用`delete`方法：

```php
$cat = Cat::find(1);
$cat->delete();
```

或者，你可以调用`destroy`方法，指定你想要删除的记录的 ID，而不需要先获取这些记录：

```php
Cat::destroy(1);
Cat::destroy(1, 2, 3, 4, 5);
```

## 软删除

默认情况下，Eloquent 将从你的数据库中**硬删除**记录。这意味着一旦删除，它就永远消失了。如果你需要保留已删除的数据（即，用于审计），那么你可以使用**软删除**。当删除模型时，记录将保留在数据库中，但会设置一个 `deleted_at` 时间戳，并且当查询数据库时，将不包括设置此时间戳的任何记录。

软删除可以轻松添加到你的 Eloquent 模型中。你只需要包含以下特性：

```php
use Illuminate\Database\Eloquent\SoftDeletes;
class Cat extends Model {
  use SoftDeletes;
 protected $dates = ['deleted_at'];
}
```

我们还指定了 `deleted_at` 列应被视为日期列。这将产生一个 Carbon 实例，并允许我们在需要时对其进行操作或以各种格式显示。

你还需要确保将 `deleted_at` 列添加到你的表迁移中。以下是一个此类迁移的示例：

```php
public function up() {
  $table->softDeletes();
}
```

### 包含已删除模型的结果

如果你发现需要在查询数据库时包含已删除的记录（例如，在管理区域），那么你可以使用 `withTrashed` 查询作用域。查询作用域只是你可以用于链式调用的方法：

```php
$cats = Cat::withTrashed()->get();
```

这将混合已删除的记录和非删除的记录。如果你发现你需要检索*仅*已删除的记录，那么你可以使用 `onlyTrashed` 查询作用域：

```php
$cats = Cat::onlyTrashed()->get();
```

如果你发现你需要“恢复”一条记录，那么 `SoftDeletes` 特性为你提供了一个新的 `restore` 方法来撤销此操作：

```php
$cat->restore();
```

最后，如果你发现你真的需要从数据库中删除一条记录，可以使用 `forceDelete` 方法。正如其名所示，一旦使用此方法删除记录，它就真的消失了。

```php
$cat->forceDelete();
```

# 查询作用域

上一节向您介绍了查询作用域的概念。这是基于查询构建器，允许您根据需要构建条件。然而，如果你需要某些条件适用于每个请求？或者一个实际上是多个 `WHERE` 子句组合的单个条件？这就是查询作用域发挥作用的地方。

查询作用域允许你在模型中一次性定义这些条件，然后无需手动定义构成该条件的子句即可重用它们。例如，假设我们需要在我们的应用程序的多个地方找到年龄超过 21 岁的用户。我们可以将此表示为一个查询作用域：

```php
class User extends Model {

 public function scopeOver21($query)
 {
 $date = Carbon::now()->subYears(21);
 return $query->where('birth_date', '<', $date);
 }
}
```

多亏了流畅的查询构建器，我们现在可以这样使用：

```php
$usersOver21 = User::over21()->get();
```

如您所见，查询作用域是那些以“scope”一词开头的方法，它们接受当前查询作为参数，以某种方式修改它，然后返回修改后的查询，以便在另一个子句中使用。这意味着你可以像使用任何其他查询表达式一样链式调用查询作用域：

```php
$malesOver21 = User::male()->over21()->get();
```

除了这些简单的查询作用域之外，我们还可以创建更“动态”的查询作用域，这些作用域接受参数，并且可以将它们传递给作用域的条件。考虑以下示例代码：

```php
class Cat extends Model {
  public function scopeOfBreed($query, $breedId)
 {
 return $query->where('breed_id', '=', $breedId);
 }
}
```

然后，我们可以按照以下方式找到特定品种的猫：

```php
$tabbyCats = Cat::ofBreed(1)->get();
```

# 关联

当我们在第三章中构建我们的应用程序时，*您的第一个应用程序*，我们使用了关系。在我们的应用程序中，每只猫都属于一个特定的品种。然而，我们并没有在每只猫旁边存储品种名称，并且可能重复多次，而是创建了一个单独的`breeds`表，每只猫的品种都是一个引用该表中记录 ID 的值。这为我们提供了一个关于两种关系类型的例子：猫*属于*一个品种，但一个品种可以*拥有*许多猫。这被定义为**一对多**关系。

Eloquent 为每种关系类型都提供了良好的支持：

+   一对一

+   多对多

+   通过多个模型关联

+   多态关系

+   多对多多态关系

在这里，我们将通过每个示例来查看它们。

## 一对一

有时，你可能想将数据分散在多个表中以便于管理，或者因为它们代表了一个实体的两个不同部分。一个常见的例子是用户及其个人资料。你可能有一个包含关于该用户核心信息的`users`表，例如他们的姓名、账户电子邮件地址和密码散列；然而，如果是一个社交网络网站，他们可能还有一个包含更多信息的个人资料，例如他们最喜欢的颜色。这些信息可以存储在一个单独的`profiles`表中，其中外键代表个人资料所属的用户。

在你的模型中，这种关系将看起来像这样：

```php
class User extends Model {

  public function profile()
 {
 return $this->hasOne('App\Profile');
 }
}
```

在`Profile`模型中，关系将看起来像这样：

```php
class Profile extends Model {

  public function user()
 {
 return $this->belongsTo('App\User');
 }
}
```

当查询`用户`时，我们也可以单独访问他们的个人资料：

```php
$profile = User::find(1)->profile;
```

使用在模型中定义它的方法的名称来访问关系。由于在`User`模型中我们在一个名为`profile`的方法中定义了关系，因此这是我们用来访问相关模型数据的属性名称。

## 多对多

多对多关系比一对一（一个模型恰好属于另一个模型）或一对多关系（许多模型可以属于另一个模型）更复杂。正如其名所示，许多模型可以属于许多其他模型。为了实现这一点，除了涉及两个表之外，还需要引入第三个表。这可能会相当难以理解，所以让我们通过一个例子来看看。

想象你正在构建一个权限系统来限制每个用户可以执行的操作。你不必为每个用户分配权限，而是有角色，每个用户根据他们被分配的角色获得一组子权限。在这个描述中，我们识别了两个实体：`用户`和`角色`。在这个场景中，一个用户可以有多个角色，一个角色可以属于多个用户。为了将角色映射到用户，我们创建了一个第三个表，称为连接表。Laravel 将这些表称为**枢纽表**，如果你之前使用过电子表格，你可能已经听说过这个术语。

默认情况下，Eloquent 预期连接表包含两个目标表的单一名称，按字母顺序排列并用下划线分隔。所以在我们这个场景中，这将是一个 `role_user`。该表本身只包含两列（除了主键之外）。这些列代表 `Role` 模型和它正在创建关系的 `User` 模型的外键。再次按照约定优于配置的原则，这些名称应该是小写、单数，并附加 `_id`，即 `role_id` 和 `user_id`。

我们在 `User` 和 `Role` 模型中使用 `belongsToMany` 方法定义了这种关系：

```php
class User extends Model {
  public function roles()
 {
 return $this->belongsToMany('App\Role');
 }
}

class Role extends Model {
  public function users()
 {
 return $this->belongsToMany('App\User');
 }
}
```

现在我们可以找出用户被分配了哪些角色：

```php
$roles = User::find(1)->roles;
```

我们还可以找出具有特定角色的所有用户：

```php
$admins = Role::find(1)->users;
```

如果你需要向用户添加新角色，你可以通过使用 `attach` 方法来实现：

```php
$user = User::find(1);
$user->roles()->attach($roleId);
```

当然，`attach` 的反义词是 `detach`：

```php
$user->roles()->detach($roleId);
```

`attach` 和 `detach` 方法也接受数组，允许你在一次操作中添加/删除多个关系。

或者，你也可以使用 `sync` 方法。与 `sync` 的区别在于，只有当操作完成后，传递的 ID 才会出现在连接表中，而不是从现有关系中添加/删除它们。

```php
$user->roles()->sync(1, 2, 3, 4, 5);
```

### 存储在枢纽表中的数据

除了在枢纽表中存储相关模型的两个主键之外，你还可以存储额外的数据。想象一下，在一个应用程序中，我们有用户和组。许多用户可以属于许多组，但用户也可以是组的调解员。为了指示哪些用户是某个组的调解员，我们可以在枢纽表上添加一个 `is_moderator` 列。为了指定在调用 `attach` 方法时应存储在枢纽表中的额外数据，我们可以在调用 `attach` 方法时指定第二个参数：

```php
$user->groups()->attach(1, ['is_moderator' => true]);
```

当使用 `sync` 方法时，我们也可以使用相同的方法：

```php
$user->groups()->sync([1 => ['is_moderator' => true]]);
```

## 一对多通过

当你需要从与你当前正在操作的直接相关的模型中获取数据时，有相关数据的情况下事情很简单；但如果你想要获取与你当前模型相隔两个**跳**的数据会发生什么呢？

考虑一个简单的电子商务网站。你可能有一个 `Product` 模型，一个 `Order` 模型，以及一个 `OrderItem` 模型，它属于产品也属于订单。你被分配的任务是找出包含特定产品的所有订单。如果 `Product` 与 `Order` 没有直接关联，你该如何做呢？幸运的是，在我们的场景中，它们有一个共同的关系——`OrderItem` 模型。

我们可以使用 "一对多通过" 关系通过中间的 `OrderItem` 模型来访问产品所属的订单。我们在 `Product` 模型中设置关系，如下所示：

```php
class Product extends Model {

  public function orders()
 {
 return $this->hasManyThrough('App\Order', 'App\OrderItem');
 }
}
```

`hasManyThrough` 方法中的第一个参数是目标模型，第二个参数是我们通过它到达的中间模型。现在我们可以轻松地列出产品所属的订单：

```php
$product = Product::find(1);
$orders = $product->orders;
```

## 多态关系

多态关系一开始很难理解；然而，一旦你理解了它们，它们就非常强大。它们允许一个模型在单个关联中属于多个其他模型。

多态关系的一个常见用例是创建一个图像库，然后允许你的其他模型通过链接到图像库表中的相关记录来包含图片。一个基本的`Image`模型看起来像这样：

```php
class Image extends Model {

  public function imageable()
 {
 return $this->morphTo();
 }
}
```

`morphTo`方法使得这个模型成为多态。现在，在我们的其他模型中，我们可以创建一个与`Image`模型的关联，如下所示：

```php
class Article extends Model {
  public function images()
 {
 $this->morphMany('App\Image', 'imageable');
 }
}
```

你现在可以通过你的`Article`模型获取任何相关的`Image`模型：

```php
$article = Article::find(1);

foreach ($article->images as $image) {
  // Do something with image
}
```

你可能会认为这和一对一关系没有区别，但当你从另一边看这个关系时，差异就变得明显了。当你检索一个`Image`实例时，如果你访问`imageable`关系，你会收到一个属于任何“拥有”该图片的模型的实例。这可能是一个`Article`、一个`Product`或者你应用中的另一个模型类型。Eloquent 通过不仅存储外键值，还存储模型类名来实现这一点。在我们的`Image`模型的情况下，列将是`imageable_id`和`imageable_type`。在创建迁移时，有一个方法可以创建这两个列：

```php
$table->morphs('imageable');
```

## 多对多多态关系

我们将要查看的最后一个关系类型是**多对多多态关系**，这是最复杂的。继续我们的图像库示例，我们可以看到它有一个缺点，一个`Image`一次只能属于另一个模型。所以，虽然我们可以看到我们应用中所有模型上传的图片，但我们不能像在真正的图像库中那样重复使用上传的图片。这就是多对多多态关系可以发挥作用的地方。

保持我们的`images`和`articles`表，我们需要引入第三个表，`imageables`。关系数据从`images`表中移除，并放置在这个新表中，这个新表还有一个外键列指向`Image`主键。这三个列是：

+   `image_id`

+   `imageable_id`

+   `imageable_type`

使用这个模式，一个`Image`可以有多个关系。也就是说，图片可以在多个模型中重复使用，无论是多个`Article`记录，还是不同类型的模型。我们的更新后的模型类现在是这样的：

```php
class Article extends Model {
  public function images()
 {
 return $this->morphedByMany('App\Image', 'imageable');
 }
}
```

`Image`模型也被更新，包含其每个关系的操作方法：

```php
class Image extends Model {
  public function articles()
 {
 return $this->morphToMany('App\Article', 'imageable');
 }
 public function products()
 {
 return $this->morphToMany('App\Product', 'imageable');
 }
 // And any other relations
}
```

你仍然可以像在正常的多态关系一样访问`images`关系。

# 模型事件

Eloquent 在不同的点触发许多事件，例如当模型正在保存或删除时。以下是一个 Eloquent 模型可以触发的方法列表：

+   `creating`

+   `created`

+   `updating`

+   `updated`

+   `saving`

+   `saved`

+   `deleting`

+   `deleted`

+   `restoring`

+   `restored`

这些名称是自解释的。过去和现在分词之间的区别在于，例如`creating`这样的事件在模型创建之前触发，而`created`在模型创建之后触发。因此，如果你在`creating`事件的处理器中停止执行，记录将不会被保存；而如果你在`created`事件的处理器中停止执行，记录仍然会被持久化到数据库中。

## 注册事件监听器

它非常开放，关于在哪里注册模型事件的监听器。一个地方是在`EventServiceProvider`类中的`boot`方法内：

```php
public function boot(DispatcherContract $events)
{
  parent::boot($events);

  User::creating(function($user)
 {
 // Do something
 });
}
```

确保在文件顶部导入`DispatcherContract`命名空间：

```php
use Illuminate\Contracts\Bus\Dispatcher as DispatcherContract;
```

优雅的模型为每个事件提供了一个方法，你可以传递一个匿名函数。这个匿名函数接收一个模型的实例，然后你可以对其执行操作。所以如果你想在每次`Article`模型保存时创建一个 URL 友好的文章标题表示，你可以通过监听保存事件来实现这一点：

```php
Article::saving(function($article)
{
  $article->slug = Str::slug($article->headline);
});
```

## 模型观察者

随着你向`EventServiceProvider`类添加更多的模型事件处理器，你可能会发现它变得越来越拥挤且难以维护。这就是处理模型事件的替代方案——模型观察者。

模型观察者是独立类，你可以将其附加到模型上，并实现你需要的任何事件的方法。所以我们的 slug 创建函数可以被重构为模型观察者，如下所示：

```php
use Illuminate\Support\Str;

class ArticleObserver {
  public function saving($article)
  {
    $article->slug = Str::slug($article->headline);
  }
}
```

我们可以在`EventServiceProvider`类中注册我们的观察者：

```php
public function boot(DispatcherContract $events)
{
  parent::boot($events);
  Article::observe(new ArticleObserver);
}
```

# 集合

从历史上看，其他带有自己 ORM 和查询构建器的框架返回的结果集要么是多维数组，要么是**普通的 PHP 对象**（**POPOs**）。Eloquent 从其他更成熟的 ORM 中汲取灵感，而不是返回结果集为集合对象的实例。

集合对象非常强大，因为它不仅包含从数据库返回的数据，还包含许多辅助方法，允许你在向用户显示之前操纵这些数据。

## 检查集合中是否存在键

如果你需要确定某个特定的键是否存在于集合中，你可以使用`contains`方法：

```php
$users = User::all();
if ($users->contains($userId))
{
  // Do something
}
```

当查询模型时，任何关系也会作为子集合返回，允许你在关系上使用完全相同的方法：

```php
$user = User::find(1);
if ($user->roles->contains($roleId))
{
  // Do something
}
```

默认情况下，模型返回`Illuminate\Database\Eloquent\Collection`的实例。然而，这可以被覆盖以使用不同的类。如果我们想向集合添加额外的功能，这会很有用。

假设有一个角色集合，我们想要确定管理员是否是这些角色之一。如果我们想象管理员角色有一个主键值为`1`，我们可以创建一个新的方法，如下所示：

```php
<?php namespace App;
use Illuminate\Database\Eloquent\Collection as EloquentCollection;
class RoleCollection extends EloquentCollection {
  public function containsAdmin()
 {
 return $this->contains(1);
 }
}
```

第二部分是告诉`Role`模型使用我们新的集合：

```php
use App\RoleCollection;

class Role extends Model {
  public function newCollection(array $models = array())
 {
 return new RoleCollection($models);
 }
}
```

而不是实例化默认的 Eloquent 集合，它将创建我们`RoleCollection`类的新实例，并用查询结果填充它。这意味着每次我们请求角色时，我们都可以使用我们新的`containsAdmin`方法：

```php
$user = User::find(1);

if ($user->roles->containsAdmin())
{
  // Let user administrate something
}
else
{
 // User does not have administrator role
}
```

Eloquent 集合还提供了许多其他有用的功能，允许您操作、过滤和迭代项目。您可以在[`laravel.com/docs/master/eloquent#collections`](http://laravel.com/docs/master/eloquent#collections)查看这些方法的更多信息。

# 摘要

尽管我们在本章中已经涵盖了大量内容，但 Eloquent 功能丰富，遗憾的是，没有足够的空间来介绍其每一个特性。不过，我们已经涵盖了 Eloquent 最重要的方面，这将为您在保存和检索数据、在模型之间创建不同复杂度的关系以及处理模型生命周期中引发的各种事件方面打下坚实的基础。

下一章我们将继续学习如何测试我们的应用程序，以确保其尽可能的坚不可摧。
