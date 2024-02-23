# 第八章：使用 Eloquent ORM 查询数据库

在之前的章节中，您学习了如何构建应用程序的基本组件。在本章中，将介绍 Eloquent ORM，这是使 Laravel 如此受欢迎的最佳功能之一。

在本章中，我们将涵盖以下主题：

+   基本查询语句

+   一对一，一对多和多对多关系

+   多态关系

+   急切加载

ORM，或对象关系映射，在最简单的意义上解释，将表转换为类，将其列转换为属性，并将其行转换为该类的实例。它在开发人员和数据库之间创建了一个抽象层，并允许更容易的编程，因为它使用熟悉的面向对象范式。

我们假设有一个带有以下结构的帖子表：

| **id** | **contents** | **author_id** |   |
| --- | --- | --- | --- |

为了说明这个例子，以下将是帖子表的表示：

```php
<?php
namespace MyBlog;

class Post {
}
```

要添加`id`，`contents`和`author_id`属性，我们将在类中添加以下代码：

```php
class Post {
    private $id;
    private $contents;
    private $author_id;

    public function getId()
    {
        return $this->id;
    }

    public function setId($id)
    {
        $this->id = $id;
    }

    public function getContents()
    {
        return $this->contents;
    }

    public function setContents($contents)
    {
        $this->contents = $contents;
    }

    public function getAuthorId()
    {
        return $this->author_id;
    }

    public function setAuthorId($author_id)
    {
        $this->author_id = $author_id;
    }

}
```

这给我们一个关于如何用类表示表的概述：`Post`类表示一个具有**posts**集合的实体。

如果遵循了活动记录模式，那么 Eloquent 可以自动管理所有类名、键名和它们的相关关系。Eloquent 的强大之处在于它能够让程序员使用面向对象的方法来管理类之间的关系。

# 基本操作

现在我们将讨论一些基本操作。使用 Eloquent 有几乎无数种方式，当然每个开发人员都会以最适合其项目的方式使用 Eloquent。以下技术是更复杂查询的基本构建块。

## 查找一个

最基本的操作之一是执行以下查询：

```php
select from rooms where id=1;
```

这是通过使用`find()`方法实现的。

使用`find`方法调用`Room`外观，该方法接受 ID 作为参数：

```php
MyCompany\Accommodation\Room::find($id);
```

由于 Eloquent 基于流畅的查询构建器，任何流畅的方法都可以混合和匹配。一些流畅的方法是可链接的，而其他方法执行查询。

`find()`方法实际上执行查询，因此它总是需要在表达式的末尾。

如果未找到模型的 ID，则不返回任何内容。要强制`ModelNotFoundException`，然后可以捕获它以执行其他操作，例如记录日志，添加`OrFail`如下：

```php
MyCompany\Accommodation\Room::findOrFail($id);
```

## where 方法

要查询除 ID 以外的属性（列），请使用以下命令：

```php
select from accommodations where name='Lovely Hotel';
```

使用`where`方法后跟`get()`方法：

```php
MyCompany\Accommodation::where('name','Lovely Hotel')->get();
```

`like`比较器可以如下使用：

```php
MyCompany\Accommodation::where('name','like','%Lovely%')->get();
```

## 链接函数

多个 where 方法可以链接如下：

```php
MyCompany\Accommodation::where('name','Lovely Hotel')- >where('city','like','%Pittsburgh%')->get();
```

上述命令产生以下查询：

```php
select * from accommodations where name ='Lovely Hotel' and description like '%Pittsburgh%'
```

请注意，如果`where`比较器是`=`（相等），则不需要第二个参数（比较器），并且比较的第二部分传递到函数中。还要注意，在两个`where`方法之间添加了`and`操作。要实现`or`操作，必须对代码进行以下更改：

```php
MyCompany\Accommodation::where('name','Lovely Hotel')- >orWhere('description','like','%Pittsburgh%')->get();
```

请注意，`or`被添加到`where`创建`orWhere()`。

## 查找所有

要找到所有房间，使用`all()`方法代替`find`。请注意，此方法实际上执行查询：

```php
MyCompany\Accommodation\Room::all();
```

为了限制房间的数量，使用`take`方法代替`find`。由于`take`是可链接的，需要使用`get`来执行查询：

```php
MyCompany\Accommodation\Room::take(10)->get();
```

要实现分页，可以使用以下查询：

```php
MyCompany\Accommodation\Room::paginate();
```

默认情况下，上述查询将返回一个 JSON 对象，如下所示：

```php
{"total":15,        "per_page":15,
"current_page":1,      "last_page":1,
"next_page_url":null,   "prev_page_url":null,
"from":1,        "to":15,
"data":
{"id":9,"name":"LovelyHotel","description":"Lovely Hotel Greater Pittsburgh","location_id":1,"created_at":null,"updated_at": "2015-03-13 22:00:23","deleted_at":null,"franchise_id":1},{"id":12, "name":"Grand Hotel","description":"Grand Hotel Greater Cleveland","location_id":2,"created_at":"2015-02- 0820:09:35","updated_at":"2015-02- 0820:09:35","deleted_at":null,"franchise_id":1}
...
```

属性，如`total`，`per_page`，`current_page`和`last_page`，用于为开发人员提供一种简单的实现分页的方法，而数据数组则返回在名为`data`的数组中。

# 优雅的关系

诸如一对一、一对多（或多对一）和多对多之类的关系对于数据库程序员来说是熟悉的。Laravel 的 Eloquent 已经将这些概念带入了面向对象的环境中。此外，Eloquent 还有更强大的工具，比如多态关系，其中实体可以与多个其他实体相关联。在接下来的示例中，我们将看到住宿、房间和便利设施之间的关系。

![Eloquent 关系

## 一对一

第一个关系是一对一。在我们的示例软件中，我们可以使用我们住宿中的房间的例子。一个房间可能只（至少很容易）属于一个住宿，所以房间*属于*住宿。在`Room` Eloquent 模型中，以下代码告诉 Eloquent 房间属于`accommodation`函数：

```php
class Room extends Eloquent {
     public function accommodation()
     {
         return $this->belongsTo('MyCompany\Accommodation');
     }
}
```

有时，数据库表不遵循活动记录模式，特别是如果程序员继承了遗留数据库。如果数据库使用了一个名为`bedroom`而不是`rooms`的表，那么类将添加一个属性来指示表名：

```php
class Room extends Eloquent {
    protected $table = 'bedroom';
}
```

当执行以下路由代码时，`accommodation`对象将以 JSON 对象的形式返回：

```php
Route::get('test-relation',function(){
    $room = MyCompany\Accommodation\Room::find(1);
    return $room->accommodation;
});
```

响应将如下：

```php
{"id":9,"name":"LovelyHotel","description":"Lovely Hotel Greater Pittsburgh","location_id":1,"created_at":null,"updated_at": "2015-03-13 22:00:23","deleted_at":null}
```

### 提示

一个常见的错误是使用以下命令：

```php
return $room->accommodation();
```

在这种情况下，程序员期望返回模型。这将返回实际的`belongsTo`关系，在 RESTful API 的上下文中，将会抛出错误：

```php
Object of class Illuminate\Database\Eloquent\Relations\BelongsTo could not be converted to string
```

这是因为 Laravel 可以将 JSON 对象转换为字符串，但不能转换为关系。

运行的 SQL 如下：

```php
select * from rooms where rooms.id = '1' limit 1
select * from accommodations where accommodations.id = '9' limit 1
```

Eloquent 倾向于使用多个简单的查询，而不是进行更大的连接。

首先找到房间。然后，添加`limit 1`，因为`find`只用于查找单个实体或行。一旦找到`accommodation_id`，下一个查询将找到具有相应 ID 的住宿并返回对象。如果遵循了活动记录模式，Eloquent 生成的 SQL 非常易读。

## 一对多

第二个关系是一对多。在我们的示例软件中，我们可以使用住宿有许多房间的例子。因为房间可能属于一个住宿，那么住宿有*许多房间*。在`Accommodation` Eloquent 模型中，以下代码告诉 Eloquent 住宿有许多房间。

```php
class Accommodation {
    public function rooms(){
        return $this->hasMany('\MyCompany\Accommodation\Room');
    }
}
```

在类似的路由中，运行以下代码。这次，将以 JSON 格式的对象数组返回一组`rooms`对象：

```php
Route::get('test-relation',function(){
    $accommodation = MyCompany\Accommodation::find(9);
    return $accommodation->rooms;
});
```

响应将是以下数组：

```php
[{"id":1,"room_number":0,"created_at":null,"updated_at":null, "deleted_at":null,"accommodation_id":9},{"id":3,"room_number": 12,"created_at":"2015-03-14 08:52:25","updated_at":"2015-03-14  08:52:25","deleted_at":null,"accommodation_id":9},{"id":6, "room_number":12,"created_at":"2015-03-14  09:03:36","updated_at":"2015-03-14  09:03:36","deleted_at":null,"accommodation_id":9},{"id": 14,"room_number":12,"created_at":"2015-03-14  09:26:36","updated_at":"2015-03- 1409:26:36","deleted_at":null,"accommodation_id":9}]
```

运行的 SQL 如下：

```php
select * from accommodations where accommodations.id = ? limit 1
select * from rooms where rooms.accommodation_id = '9' and  rooms.accommodation_id is not null
```

与之前一样，找到住宿。第二个查询将找到属于该住宿的房间。添加了一个检查以确认`accommodation_id`不为空。

## 多对多

在我们的示例软件应用程序中，便利设施和房间之间的关系是多对多的。每个房间可以有许多便利设施，比如互联网接入和按摩浴缸，每个便利设施都在许多房间之间共享：*住宿中的每个房间都可以并且应该有互联网接入！*以下代码使用`belongsToMany`关系，使便利设施可以属于许多房间：

```php
class Amenity {
  public function rooms(){
        return $this- >belongsToMany('\MyCompany\Accommodation\Room');
    }
}
```

告诉我们每个房间都有某个便利设施的测试路由写成如下：

```php
Route::get('test-relation',function(){
    $amenity = MyCompany\Accommodation\Amenity::find(3);
    return $amenity->rooms;
});
```

返回一个房间列表：

```php
[{"id":1,"room_number":0,"created_at":2015-03-14 08:10:45,"updated_at":null,"deleted_at":null, "accommodation_id":9},{"id":5,"room_number":12, "created_at":"2015-03-14 09:00:38","updated_at":"2015-03-14", 09:00:38","deleted_at":null,"accommodation_id":12},
...]
```

执行的 SQL 如下：

```php
select * from amenities where amenities.id = ? limit 1
select rooms.*, amenity_room.amenity_id as pivot_amenity_id, amenity_room.room_id as pivot_room_id from rooms inner join amenity_room on rooms.id = amenity_room.room_id where amenity_room.amenity_id = 3
```

我们回忆一下`belongToMany`关系，它返回具有特定便利设施的房间：

```php
class Amenity {
   public function rooms(){
        return $this- >belongsToMany('\MyCompany\Accommodation\Room');
    }
}
```

Eloquent 巧妙地给了我们相应的`belongsToMany`关系，以确定特定房间有哪些便利设施。语法完全相同：

```php
class Room {
     public function amenities(){
         return $this- >belongsToMany('\MyCompany\Accommodation\Amenity');
     }
 }
```

测试路由几乎相同，只是用`rooms`替换`amenities`：

```php
Route::get('test-relation',function(){
    $room = MyCompany\Accommodation\Room::find(1);
    return $room->amenities;
});
```

结果是 ID 为 1 的房间的便利设施列表：

```php
[{"id":1,"name":"Wifi","description":"Wireless Internet Access","created_at":"2015-03-1409:00:38","updated_at":"2015-03-14 09:00:38","deleted_at":null},{"id":2,"name": "Jacuzzi","description":"Hot tub","created_at":"2015-03-14 09:00:38","updated_at":null,"deleted_at":null},{"id":3,"name": "Safe","description":"Safe deposit box for protecting valuables","created_at":"2015-03-1409:00:38","updated_at": "2015-03-1409:00:38","deleted_at":null}]
```

使用的查询如下：

```php
select * from rooms where rooms.id = 1 limit 1
select amenities.*, amenity_room.room_id as pivot_room_id, amenity_room.amenity_id as pivot_amenity_id from amenities inner join amenity_room on amenities.id = amenity_room.amenity_id where amenity_room.room_id = '1'
```

查询，用`room_id`替换`amenity_id`，用`rooms`替换`amenities`，显然是并行的。

## 有许多通过

Eloquent 的一个很棒的特性是“has-many-through”。如果软件的需求发生变化，并且我们被要求将一些住宿分组到特许经营店中，该怎么办？如果应用程序用户想要搜索一个房间，那么属于该特许经营店的任何住宿中的任何房间都可以被找到。将添加一个特许经营店表，并在住宿表中添加一个可空列，名为 `franchise_id`。这将可选地允许住宿属于特许经营店。房间已经通过 `accommodation_id` 列属于住宿。

一个房间通过其 `accommodation_id` 键属于一个 `住宿`，而一个住宿通过其 `franchise_id` 键属于一个特许经营店。

Eloquent 允许我们通过使用 `hasManyThrough` 来检索与特许经营店相关联的房间：

```php
<?php namespace MyCompany;

use Illuminate\Database\Eloquent\Model;

class Franchise extends Model {

    public function rooms()
    {
        return $this- >hasManyThrough('\MyCompany\Accommodation\Room', '\MyCompany\Accommodation');
    }
}
```

`hasManyThrough` 关系将目标或“拥有”作为其第一个参数（在本例中是房间），将“通过”作为第二个参数（在本例中是住宿）。

作为短语陈述的逻辑是：*这个特许经营店通过其住宿拥有许多房间*。

使用先前的测试路由，代码编写如下：

```php
Route::get('test-relation',function(){
    $franchise = MyCompany\Franchise::find(1);
    return $franchise->rooms;
});
```

返回的房间是一个数组，正如预期的那样：

```php
[{"id":1,"room_number":0,"created_at":null,"updated_at":null,"deleted_at":null,"accommodation_id":9,"franchise_id":1}, {"id":3,"room_number":12,"created_at":"2015-03-14 08:52:25","updated_at":"2015-03-14 08:52:25","deleted_at":null,"accommodation_id":9, "franchise_id":1},{"id":6,"room_number":12,"created_at":"2015-03-14 09:03:36","updated_at":"2015-03-14 09:03:36","deleted_at":null,"accommodation_id":9, "franchise_id":1},
]
```

执行的查询如下：

```php
select * from franchises where franchises.id = ? limit 1
select rooms.*, accommodations.franchise_id from rooms inner join accommodations on accommodations.id = rooms.accommodation_id where accommodations.franchise_id = 1
```

# 多态关系

Eloquent 的一个很棒的特性是拥有一个关系是多态的实体的可能性。这个词的两个部分，*poly* 和 *morphic*，来自希腊语。由于 *poly* 意味着 *许多*，*morphic* 意味着 *形状*，我们现在可以很容易地想象一个关系有多种形式。

## 设施关系

在我们的示例软件中，一个设施是与房间相关联的东西，比如按摩浴缸。某些设施，比如有盖停车场或机场班车服务，也可能与住宿本身相关。我们可以为此创建两个中间表，一个叫做 `amenity_room`，另一个叫做 `accommodation_amenity`。另一种很好的方法是将两者合并成一个表，并使用一个字段来区分两种类型或关系。

为了做到这一点，我们需要一个字段来区分 *设施和房间* 和 *设施和房间*，我们可以称之为关系类型。Laravel 的 Eloquent 能够自动处理这一点。

Eloquent 使用后缀 `-able` 来实现这一点。在我们的示例中，我们将创建一个具有以下字段的表：

+   `id`

+   `name`

+   `description`

+   `amenitiable_id`

+   `amenitiable_type`

前三个字段是熟悉的，但添加了两个新字段。其中一个将包含住宿或房间的 ID。

### 设施表结构

例如，给定 ID 为 5 的房间，`amenitiable_id` 将是 `5`，而 `amenitiable_type` 将是 `Room`。给定 ID 为 5 的住宿，`amenitiable_id` 将是 `5`，而 `amenitiable_type` 将是 `Accommodation`：

| id | name | description | amenitiable_id | amenitiable_type |
| --- | --- | --- | --- | --- |
| 1 | 无线网络 | 网络连接 | 5 | 房间 |
| 2 | 有盖停车场 | 车库停车 | 5 | 住宿 |
| 3 | 海景 | 房间内海景 | 5 | 房间 |

### 设施模型

在代码方面，`Amenity` 模型现在将包含一个 "amenitiable" 函数：

```php
<?php
namespace MyCompany\Accommodation;

use Illuminate\Database\Eloquent\Model;

class Amenity extends Model
{
    public function rooms(){
        return $this->belongsToMany('\MyCompany\Accommodation\Room');
    }
    public function amenitiable()
    {
        return $this->morphTo();
    }
```

### 住宿模型

`住宿` 模型将更改 `amenities` 方法，使用 `morphMany` 而不是 `hasMany`：

```php
<?php namespace MyCompany;

use Illuminate\Database\Eloquent\Model;

class Accommodation extends Model {
    public function rooms(){
        return $this->hasMany('\MyCompany\Accommodation\Room');
    }

    public function amenities()
    {
        return $this- >morphMany('\MyCompany\Accommodation\Amenity', 'amenitiable');
    }
}
```

### 房间模型

`Room` 模型将包含相同的 `morphMany` 方法：

```php
<?php
namespace MyCompany\Accommodation;

use Illuminate\Database\Eloquent\Model;

class Room extends Model
{
    protected $casts = ['room_number'=>'integer'];
    public function accommodation(){
        return $this->belongsTo('\MyCompany\Accommodation');
    }
    public function amenities() {
        return $this- >morphMany('\MyCompany\Accommodation\Amenity', 'amenitiable');
    }

}
```

现在，当要求为房间或住宿请求设施时，Eloquent 将自动区分它们：

```php
$accommodation->amenities();
$room->amenities();
```

这些函数中的每一个都返回了房间和住宿的正确类型的设施。

## 多对多多态关系

然而，一些设施可能在房间和住宿之间共享。在这种情况下，使用多对多多态关系。现在中间表添加了几个字段：

| amenity_id | amenitiable_id | amenitiable_type |
| --- | --- | --- |
| 1 | 5 | 房间 |
| 1 | 5 | 住宿 |
| 2 | 5 | 房间 |
| 2 | 5 | 住宿 |

正如所示，ID 为 5 的房间和 ID 为 5 的住宿都有 ID 为 1 和 2 的设施。

## 具有关系

如果我们想选择与特许经营连锁店关联的所有住宿，使用`has()`方法，其中关系作为参数传递：

```php
MyCompany\Accommodation::has('franchise')->get();
```

我们将得到以下 JSON 数组：

```php
[{"id":9,"name":"LovelyHotel","description":"Lovely Hotel Greater Pittsburgh","location_id":1,"created_at":null,"updated_at": "2015-03-13 22:00:23","deleted_at":null,"franchise_id":1}, {"id":12,"name": "Grand Hotel","description":"Grand Hotel Greater Cleveland","location_id":2,"created_at": "2015-02-0820:09:35","updated_at": "2015-02-0820:09:35","deleted_at":null,"franchise_id":1}]
```

请注意，`franchise_id`的值为 1，这意味着住宿与特许经营连锁店相关联。可选地，可以在`has`中添加`where`，创建一个`whereHas`函数。代码如下：

```php
MyCompany\Accommodation::whereHas('franchise',
                  function($query){
      $query->where('description','like','%Pittsburgh%'); 
      })->get();
```

请注意，`whereHas`将闭包作为其第二个参数。

这将仅返回描述中包含`匹兹堡`的住宿，因此返回的数组将只包含这样的结果：

```php
[{"id":9,"name":"LovelyHotel","description":"Lovely Hotel Greater Pittsburgh","location_id":1,"created_at":null,"updated_at": "2015-03-13 22:00:23","deleted_at":null,"franchise_id":1}]
```

## 贪婪加载

Eloquent 提供的另一个很棒的机制是贪婪加载。如果我们想要返回所有的特许经营连锁店以及它们的所有住宿，我们只需要在我们的`Franchise`模型中添加一个`accommodations`函数，如下所示：

```php
    public function accommodations()
    {
        return $this->hasMany('\MyCompany\Accommodation');
    }
```

然后，通过向语句添加`with`子句，为每个特许经营连锁店返回住宿：

```php
MyCompany\Franchise::with('accommodations')->get();
```

我们还可以列出与每个住宿相关的房间，如下所示：

```php
MyCompany\Franchise::with('accommodations','rooms')->get();
```

如果我们想要返回嵌套在住宿数组中的房间，则应使用以下语法：

```php
MyCompany\Franchise::with('accommodations','accommodations.rooms') ->get();
```

我们将得到以下输出：

```php
[{"id":1,"accommodations":
[
{"id":9,
"name":"Lovely Hotel",
"description":"Lovely Hotel Greater Pittsburgh",
"location_id":1,
"created_at":null,
"updated_at":"2015-03-13 22:00:23",
"deleted_at":null,
"franchise_id":1,
"rooms":[{"id":1,"room_number":0,"created_at":null,"updated_at": null,"deleted_at":null,"accommodation_id":9},
]},
{"id":12,"name":"GrandHotel","description":"Grand Hotel Greater Cleveland","location_id":2,"created_at":"2015-02-08…
```

在这个例子中，`rooms`包含在`accommodation`中。

# 结论

Laravel 的 ORM 非常强大。事实上，有太多类型的操作无法在一本书中列出。最简单的查询可以用几个按键完成。

Laravel 的 Eloquent 命令被转换为流畅的命令，因此如果需要更复杂的操作，可以使用流畅的语法。如果需要执行非常复杂的查询，甚至可以使用`DB::raw()`函数。这将允许在查询构建器中使用精确的字符串。以下是一个例子：

```php
$users = DB::table('accommodation')
                     ->select(DB::raw('count(*) as number_of_hotels'))->get();
```

这将只返回酒店的数量：

```php
[{"number_of_hotels":15}]
```

学习设计软件，从领域开始，然后考虑该领域涉及的实体，将有助于开发人员以面向对象的方式思考。拥有实体列表会导致表的创建，因此实际的模式创建将在最后执行。这种方法可能需要一些时间来适应。理解 Eloquent 关系对于能够生成表达性、可读性的查询数据库语句至关重要，同时隐藏复杂性。

Eloquent 极其有用的另一个原因是在遗留数据库的情况下。如果 ORM 应用在表名不符合标准、键名不相同或列名不易理解的情况下，Eloquent 提供了开发人员工具，实际上帮助使表名和字段名同质化，并通过提供属性的 getter 和 setter 来执行关系。

例如，如果字段名为`fname1`和`fname2`，我们可以在我们的模型中使用一个获取属性函数，语法是`get`后跟应用中要使用的所需名称和属性。因此，在`fname1`的情况下，函数将被添加如下：

```php
public function getUsernameAttribute($value)
{
  return $this->attributes['fname1'];
}
```

这些函数是 Eloquent 的真正卖点。在本章中，您学会了如何通过使用实体模型在数据库中查找数据，通过添加`where`、关系、强大的约定（如多态关系）以及辅助工具（如分页）来限制结果。

# 摘要

在本章中，详细演示了 Eloquent ORM。Eloquent 是一个面向对象的包装器，用于实际发生在数据库和代码之间的事情。由于 Fluent 查询构建器很容易访问，因此熟悉查询的编写方式非常重要。这将有助于调试，并且还涵盖了 Eloquent 不足的复杂情况。在本章中，讨论了大部分 Eloquent 的概念。然而，还有许多其他可用的方法，因此鼓励进一步阅读。

在下一章中，除了其他主题，您将学习如何扩展数据库以在更大规模上表现更好。
