# 第四章：创建 RESTful API

如果有一个单一的核心功能可以展示 Laravel 的优越性，那就是快速轻松地创建 RESTful API 的能力。随着 Laravel 5 的到来，添加了几个新功能；然而，通过 Artisan 命令行工具创建应用程序模型和控制器的能力仍然是最有用的功能。

这个功能最初鼓励了我和其他许多人放弃诸如 CodeIgniter 之类的框架，因为在 Laravel 4 测试版时，它并没有原生具有相同的集成功能。Laravel 提供了基本的 CRUD 方法：创建，读取，更新，删除，并且还列出了所有内容。

通过 HTTP 到达 Laravel URL 的请求是通过它们的动词管理的，随后是`routes.php`文件，该文件位于`app/Http/routes.php`。请求处理有两种方式。一种方式是直接通过闭包处理请求，代码完全在`routes`文件中。另一种方式是将请求路由到控制器，其中将执行一个方法。

此外，使用的基本范式是约定优于配置，其中方法名称已准备好处理各种请求，而无需太多额外的努力。

# Laravel 中的 RESTful API

由 RESTful API 处理的 RESTful API 请求列表如下：

|   | HTTP VERB | Function | URL |
| --- | --- | --- | --- |
| 1 | `GET` | 列出所有住宿 | `/accommodations` |
| 2 | `GET` | 显示（读取）单个住宿 | `/accommodations/{id}` |
| 3 | `POST` | 创建新的住宿 | `/accommodations` |
| 4 | `PUT` | 完全修改（更新）住宿 | `/accommodations/{id}` |
| 5 | `PATCH` | 部分修改（更新）住宿 | `/accommodations/{id}` |
| 6 | `DELETE` | 删除住宿 | `/accommodations/{id}` |

大多数 RESTful API 最佳实践建议使用模型名称的复数形式。Laravel 的文档使用单数格式。大多数实践都同意一致的复数命名，即`/accommodations/{id}`指的是单个住宿，`/accommodations`指的是多个住宿，都使用复数形式，而不是混合的，但语法上正确的`/accommodation/{id}`（单数形式）和`/accommodations`（复数形式）。

# 基本 CRUD

为简单起见，我已经对每一行进行了编号。第一和第二项代表了 CRUD 的“读取”部分。

第一项是对模型名称的复数形式进行的`GET`调用，相当简单；它显示所有项目。有时，这被称为“列表”，以区别于单个记录的“读取”。因此，添加一个“列表”将扩展首字母缩写为 CRUDL。它们可以进行分页或需要授权。

第二项，也是`GET`调用，将模型的 ID 添加到 URL 的末尾，显示具有相应 ID 的单个模型。这也可能需要身份验证，但不需要分页。

第三项代表了 CRUD 的“创建”部分。它使用`POST`动词来创建一个新的模型。请注意，URL 格式与第一项相同；这展示了动词在区分操作中的重要性。

第四、第五和第六项使用了一些浏览器不支持的新的`HTTP`动词。无论这些动词是否受支持，JavaScript 库和框架（如 jQuery）都会以 Laravel 可以正确处理的方式发送动词。

第四项是 CRUD 的“更新”部分，使用`PUT`动词更新模型。请注意，它与第二项具有相同的 URL 格式，因为它需要知道要更新哪个模型。它也是幂等的，这意味着整个模型必须被更新。

第五项类似于第四项；它更新模型，但使用`PATCH`动词。这用于指示模型将被部分修改，这意味着一个或多个模型的属性必须被更改。

第六项删除一个单个模型，因此需要模型的 ID，使用不言自明的 `DELETE` 动词。

# 额外功能

Laravel 添加了两个通常不是标准 RESTful API 的额外方法。在模型 URL 上使用 `GET` 方法，添加 `create` 用于显示创建模型的表单。在带有 ID 的模型 URL 上使用 `GET` 方法，添加 `edit` 用于显示创建模型的表单。这两个功能对于提供将加载表单的 URL 非常有用，尽管这种类型的使用不是标准的 RESTful：

| HTTP VERB | Function | URL |   |
| --- | --- | --- | --- |
| `GET` | 这显示一个住宿创建表单 | `/accommodations/create` |   |
| `GET` | 这显示一个住宿修改/更新表单 | `/accommodations/{id}/edit` |   |

# 控制器创建

要为住宿创建一个控制器，使用以下 Artisan 命令：

```php
**$ php artisan make:controller AccommodationsController**

 **<?php namespace MyCompany\Http\Controllers;**

 **use MyCompany\Http\Requests;**
 **use MyCompany\Http\Controllers\Controller;**
 **use Illuminate\Http\Request;**

 **class AccommodationController extends Controller {**

 **/****
 *** Display a listing of the resource.**
 *** @return Response**
 ***/**
 **public function index()**
 **{**
 **}**

 **/****
 *** Show the form for creating a new resource.**
 *** @return Response**
 ***/**
 **public function create()**
 **{**
 **}**

 **/****
 *** Store a newly created resource in storage.**
 *** @return Response**
 ***/**
 **public function store()**
 **{**
 **}**

 **/****
 *** Display the specified resource.**
 *** @param  int  $id**
 *** @return Response**
 ***/**
 **public function show($id)**
 **{**
 **}**

 **/****
 *** Show the form for editing the specified resource.**
 *** @param  int  $id**
 *** @return Response**
 ***/**
 **public function edit($id)**
 **{**
 **}**

 **/****
 *** Update the specified resource in storage.**
 *****
 *** @param  int  $id**
 *** @return Response**
 ***/**
 **public function update($id)**
 **{**
 **}**

 **/****
 *** Remove the specified resource from storage.**
 *** @param  int  $id**
 *** @return Response**
 ***/**
 **public function destroy($id)**
 **{**
 **}**
 **}**

```

# 通过示例进行 CRUD(L)

我们之前看过这个控制器，但这里有一些示例。RESTful 调用的最简单示例将如下所示。

## cRudl – 读取

创建一个 `GET` 调用到 `http://www.hotelwebsite.com/accommmodations/1`，其中 `1` 将是房间的 ID：

```php
/**
 * Display the specified resource.
 *
 * @param  int  $id
 * @return Response
 */
public function show($id)
{
    return \MyCompany\Accommodation::findOrFail($id);
}
```

这将返回一个单个模型作为 JSON 编码对象：

```php
{
    "id": 1,
    "name": "Hotel On The Hill","description":"Lorem ipsum dolor sit amet, consectetur adipiscing elit.",
    "location_id": 1,
    "created_at": "2015-02-08 20:13:10",
    "updated_at": "2015-02-08 20:13:10",
    "deleted_at": null
}
```

## crudL – 列表

创建一个 GET 调用到 `http://www.hotelwebsite.com/accommmodations`。

这与前面的代码类似，但略有不同：

```php
/** Display a listing of the resource.
    * @return Response
 */
public function index()
{
    return Accommodation::all();
}
```

这将返回所有模型，自动编码为 JSON 对象；没有其他要求。已添加格式，以便 JSON 结果更易读，但基本上整个模型都会返回：

```php
[{ 
    "id": 1,
    "name": "Hotel On The Hill","description":"Lorem ipsum dolor sit amet, consectetur adipiscing elit.",
    "location_id": 1,
    "created_at": "2015-02-08 20:13:10",
    "updated_at": "2015-02-08 20:13:10",
    "deleted_at": null
} 
{   "id": 2,
    "name": "Patterson Place",
    "description": "Lorem ipsum dolor sit amet, consectetur adipiscing elit.",
    "location_id": 2,
    "created_at": "2015-02-08 20:15:02",
    "updated_at": "2015-02-08 20:15:02",
    "deleted_at": null
},
{
    "id": 3,
    "name": "Neat and Tidy Hotel",
    "description": "Lorem ipsum dolor sit amet, consectetur adipiscing elit.",
    "location_id": 3,
    "created_at": "2015-02-08 20:17:34",
    "updated_at": "2015-02-08 20:17:34",
    "deleted_at": null
}
]
```

### 提示

`deleted_at` 字段是软删除或回收站机制。对于未删除的情况，它要么是 `null`，要么是已删除的 `date`/`time` 时间戳。

### 分页

要添加分页，只需用 `paginate()` 替换 `all()`：

```php
public function index()
{
    return Accommodation::paginate();
}
```

现在结果看起来像这样。Eloquent 集合数组现在移动到 `date` 属性内：

```php
{"total":15,
"per_page":15,
"current_page":1,
"last_page":1,
"next_page_url":null,
"prev_page_url":null,
"from":1,
"to":15,
"data":[{"id":9,
"name":"Lovely Hotel",
"description":"Lovely Hotel Greater Pittsburgh",
….
```

## Crudl – 创建

创建一个 `POST` 调用到 `http://www.hotelwebsite.com/accommmodations`。

要创建一个新模型，将发送一个 `POST` 调用到 `/accommodations`。前端将发送一个 JSON 如下：

```php
{
    "name": "Lovely Hotel",
    "description": "Lovely Hotel Greater Pittsburgh",
    "location_id":1
}
```

`store` 函数可能看起来像这样：

```php
public function store()
{
    $input = \Input::json();
    $accommodation = new Accommodation;
    $accommodation->name = $input->get('name');
    $accommodation->description = $input->get('description');
    $accommodation->location_id = $input->get('location_id');
    $accommodation->save();
    return response($accommodation, 201)
;
}
```

### 提示

`201` 是 `created` 的 HTTP 状态码（`HTTP/1.1 201 created`）。

在这个例子中，我们将模型作为 JSON 编码对象返回。对象将包括插入的 ID：

```php
{
    "name":"Lovely Hotel",
    "description":"Lovely Hotel Greater Pittsburgh",
    "location_id":1,
    "updated_at":"2015-03-13 20:48:19",
    "created_at":"2015-03-13 20:48:19",
    "id":26
}
```

## crUdl – 更新

创建一个 `PUT` 调用到 `http://www.hotelwebsite.com/accommmodations/1`，其中 `1` 是要更新的 ID：

```php
/**
    * Update the specified resource in storage.
    *
    * @param  int  $id
    * @return Response
    */
    public function update($id)
    {
        $input = \Input::json();
        $accommodation = \MyCompany\Accommodation::findOrFail($id);
        $accommodation->name = $input->get('name');
        $accommodation->description = $input->get('description');
        $accommodation->location_id = $input->get('location_id');
        $accommodation->save();
        return response($accommodation, 200)
            ->header('Content-Type', 'application/json');
    }
```

要更新现有模型，代码与之前使用的完全相同，只是使用以下行来查找现有模型：

```php
$accommodation = Accommodation::find($id);
```

`PUT` 动词将发送到 `/accommodations/{id}`，其中 `id` 将是住宿表的数字 ID。

## cruDl – 删除

要删除一个模型，创建一个 `DELETE` 调用到 `http://www.hotelwebsite.com/accommmodation/1`，其中 `1` 是要删除的 ID：

```php
/**
 * Remove the specified resource from storage.
 *
 * @param  int  $id
 * @return Response
 */
public function destroy($id)
{
    $accommodation = Accommodation::find($id);
    $accommodation->delete();
    return response('Deleted.', 200)
;
}
```

### 提示

关于删除模型的适当状态码似乎存在一些分歧。

# 模型绑定

现在，我们可以使用一种称为*模型绑定*的技术来进一步简化代码：

```php
public function boot(Router $router)
{
    parent::boot($router);
    $router->model('accommodations', '\MyCompany\Accommodation');
}
```

在 `app/Providers/RouteServiceProvider.php` 中，添加接受路由作为第一个参数并将要绑定的模型作为第二个参数的 `$router->model()` 方法。

## 重新访问读取

现在，我们的 `show` 控制器方法看起来像这样：

```php
public function show(Accommodation $accommodation)
{
    return $accommodation;
}
```

当调用 `/accommodations/1` 时，例如，与该 ID 对应的模型将被注入到方法中，允许我们替换查找方法。

## 重新访问列表

同样，对于 `list` 方法，我们按照类型提示的模型注入如下：

```php
public function index(Accommodation $accommodation)
{
    return $accommodation;
}
```

## 更新重新访问

同样，`update` 方法现在看起来像这样：

```php
public function update(Accommodation $accommodation)
{
    $input = \Input::json();
    $accommodation->name = $input->get('name');
    $accommodation->description = $input->get('description');
    $accommodation->location_id = $input->get('location_id');
    $accommodation->save();
    return response($accommodation, 200)
    ->header('Content-Type', 'application/json');
}
```

## 删除重新访问

此外，`destroy` 方法看起来像这样：

```php
public function destroy(Accommodation $accommodation)
{
    $accommodation->delete();
    return response('Deleted.', 200)
        ->header('Content-Type', 'text/html');
}
```

# 超越 CRUD

如果软件应用的一个要求是能够搜索住宿，那么我们可以很容易地添加一个搜索函数。搜索函数将使用`name`字符串查找住宿。一种方法是将路由添加到`routes.php`文件中。这将把`GET`调用映射到`AccommodationsController`中包含的新的`search()`函数：

```php
Route::get('search', 'AccommodationsController@search');
Route::resource('accommodations', 'AccommodationsController');
```

### 提示

在这种情况下，`GET`方法比`POST`方法更可取，因为它可以被收藏并稍后调用。

现在，我们将编写我们的搜索函数：

```php
public function search(Request $request, Accommodation $accommodation)
{
    return $accommodation
        ->where('name',
          'like',
          '%'.$request->get('name').'%')
        ->get();
    }
```

这里有几种机制：

+   包含来自`GET`请求的变量的`Request`对象被类型提示，然后注入到搜索函数中

+   `Accommodation`模型被类型提示，然后注入到`search`函数中

+   在 Eloquent 模型`$accommodation`上调用了`where()`方法

+   从`request`对象中使用`name`参数

+   使用`get()`方法来执行实际的 SQL 查询

### 提示

请注意，查询构建器和 Eloquent 方法中的一些返回查询构建器的实例，而其他方法执行查询并返回结果。`where()`方法返回查询构建器的实例，而`get()`方法执行查询。

+   返回的 Eloquent 集合会自动编码为 JSON

因此，`GET`请求如下：

```php
http://www.hotelwebsite.com/search-accommodation?name=Lovely
```

生成的 JSON 看起来会像这样：

```php
[{"id":3,
"name":"Lovely Hotel",
"description":"Lovely Hotel Greater Pittsburgh",
"location_id":1,
"created_at":"2015-03-13 22:00:23",
"updated_at":"2015-03-13 22:00:23",
"deleted_at":null},
{"id":4,
"name":"Lovely Hotel",
"description":"Lovely Hotel Greater Philadelphia",
"location_id":2,
"created_at":"2015-03-11 21:43:31",
"updated_at":"2015-03-11 21:43:31",
"deleted_at":null}]
```

# 嵌套控制器

嵌套控制器是 Laravel 5 中的一个新功能，用于处理涉及关系的所有 RESTful 操作。例如，我们可以利用这个功能来处理住宿和客房之间的关系。

住宿和客房之间的关系如下：

+   一个住宿可以有一个或多个房间（一对多）

+   一个房间只属于一个住宿（一对一）

现在，我们将编写代码，使得我们的模型能够熟练处理 Laravel 的一对一和一对多关系。

## Accommodation 有许多 rooms

首先，我们将添加所需的代码到代表`accommodation`模型的`Accomodation.php`文件中：

```php
class Accommodation extends Model {
    public function rooms(){
        return $this->hasMany('\MyCompany\Accommodation\Room');
    }
}
```

`rooms()`方法创建了一种从住宿模型内部访问关系的简单方法。关系说明了“住宿*有许多*房间”。`hasMany`函数，当位于`Accommodation`类内部时，没有额外的参数，期望`Room`模型的表中存在一个名为`accommodation_id`的列，这在这种情况下是`rooms`。

## Room 属于 accommodation

现在，我们将添加`Room.php`文件所需的代码，该文件代表`Room`模型：

```php
class Room extends Model
{
    public function accommodation(){
        return $this->belongsTo('\MyCompany\Accommodation');
    }
}
```

这段代码说明了“一个房间*属于*一个住宿”。`Room`类中的*belongsTo*方法，没有额外的参数，期望`room`模型的表中存在一个字段；在这种情况下，名为`accommodation_id`的`rooms`。

### 提示

如果应用程序数据库中的表遵循了活动记录约定，那么大多数 Eloquent 关系功能将自动运行。所有参数都可以很容易地配置。

创建嵌套控制器的命令如下：

```php
**$php artisan make:controller AccommodationsRoomsController**

```

然后，以下行将被添加到`app/Http/routes.php`文件中：

```php
Route::resource('accommodations.rooms', 'AccommodationsRoomsController');
```

要显示创建的路由，应执行以下命令：

```php
**$php artisan route:list**

```

以下表列出了 HTTP 动词及其功能：

|   | HTTP 动词 | 功能 | URL |
| --- | --- | --- | --- |
| 1 | `GET` | 这显示了住宿和客房的关系 | `/accommodations/{accommodations}/rooms` |
| 2 | `GET` | 这显示了住宿和客房的关系 | `/accommodations/{accommodations}/rooms/{rooms}` |
| 3 | `POST` | 这创建了一个新的住宿和客房的关系 | `/accommodations/{accommodations}/rooms` |
| 4 | `PUT` | 这完全修改（更新）了住宿和客房的关系 | `/accommodations/{accommodations}/rooms/{rooms}` |
| 5 | `PATCH` | 部分修改（更新）住宿和房间关系 | `/accommodations/{accommodations}/rooms/{rooms}` |
| 6 | `DELETE` | 删除住宿和房间关系 | `/accommodations/{accommodations}/rooms/{rooms}` |

## 雄辩的关系

一个很好的机制用于直接在控制器内部说明雄辩关系，通过使用**嵌套关系**来执行，其中两个模型首先通过路由连接，然后通过它们的控制器方法的参数通过模型依赖注入连接。

## 嵌套更新

让我们调查`update`/`modify PUT`嵌套控制器命令。URL 看起来像这样：`http://www.hotelwebsite.com/accommodations/21/rooms/13`。

这里，`21`将是住宿的 ID，`13`将是房间的 ID。参数是类型提示的模型。这使我们可以轻松地更新关系，如下所示：

```php
public function update(Accommodation $accommodation, Room $room)
{
    $room->accommodation()->associate($accommodation);
    $room->save();
}
```

## 嵌套创建

同样，可以通过`POST`请求将嵌套的`create`操作执行到`http://www.hotelwebsite.com/accommodations/21/rooms`。`POST`请求的 body 是一个 JSON 格式的对象：

```php
{"roomNumber":"123"}
```

请注意，由于我们正在创建房间，因此不需要房间 ID：

```php
public function store(Accommodation $accommodation)
{
    $input = \Input::json();
    $room = new Room();
    $room->room_number = $input->get('roomNumber');
    $room->save();
    $accommodation->rooms()->save($room);
}
```

# 雄辩模型转换

模型以 JSON 格式返回，就像它们在数据库中表示的那样。通常，模型属性，其性质为布尔值，分别用`0`和`1`表示`true`和`false`。在这种情况下，更方便的是返回一个真正的`true`和`false`给 RESTful 调用的返回对象。

在 Laravel 4 中，这是使用**访问器**完成的。如果值是`$status`，则方法将定义如下：

```php
public function getStatusAttribute($value){
    //do conversion;
}
```

在 Laravel 5 中，由于有了一个称为模型转换的新功能，这个过程变得更加容易。要应用这种技术，只需将一个受保护的键和一个名为`$casts`的值数组添加到模型中，如下所示：

```php
class Room extends Model
{
    protected $casts = ['room_number'=>'integer','status'=>'boolean'];
    public function accommodation(){
        return $this->belongsTo('\MyCompany\Accommodation');
    }
}
```

在这个例子中，`room_number`是一个字符串，但我们想返回一个整数。状态是一个小整数，但我们想返回一个布尔值。在模型中对这两个值进行转换将以以下方式修改结果 JSON：

```php
{"id":1,
"room_number": "101",
"status": 1,
"created_at":"2015-03-14 09:25:59",
"updated_at":"2015-03-14 19:03:03",
"deleted_at":null,
"accommodation_id":2}
```

前面的代码现在将改变如下：

```php
{"id":1,
"room_number": 101,
"status": true,
"created_at":"2015-03-14 09:25:59",
"updated_at":"2015-03-14 19:03:03",
"deleted_at":null,
"accommodation_id":2}
```

# 路由缓存

Laravel 5 有一个新的机制用于缓存路由，因为`routes.php`文件很容易变得非常庞大，并且会迅速减慢请求过程。要启用缓存机制，输入以下`artisan`命令：

```php
**$ php artisan route:cache**

```

这将在`/storage/framework/routes.php`中创建另一个`routes.php`文件。如果该文件存在，则会使用它，而不是位于`app/Http/routes.php`中的`routes.php`文件。文件的结构如下：

```php
<?php

/*
|--------------------------------------------------------------------------
| Load The Cached Routes
|
…
*/

app('router')->setRoutes(
unserialize(base64_decode('TzozNDoiSWxsdW1pbmF0ZVxSb3V0aW5nXFJvdXRlQ29sbGVjdGlvbiI6NDp7czo5OiIAKgByb3V0ZXMiO2E6Njp7czozOiJHRVQiO2E6M
…
... VyQGluZGV4IjtzOjk6Im5hbWVzcGFjZSI7czoyNjoiTXlDb21wYWbXBhbnlcSHR0cFxDb250cm9sbGVyc1xIb3RlbENvbnRyb2xsZXJAZGVzdHJveSI7cjo4Mzg7fX0='))
);
```

请注意，这里使用了一个有趣的技术。路由被序列化，然后进行 base64 编码。显然，要读取路由，使用相反的方法，`base64_decode()`，然后`unserialize()`。

如果`routes.php`缓存文件存在，则每次对`routes.php`文件进行更改时，都必须执行路由缓存`artisan`命令。这将清除文件，然后重新创建它。如果以后决定不再使用这种机制，则可以使用以下`artisan`命令来消除该文件：

```php
**$ php artisan route:clear**

```

Laravel 对于构建几种完全不同类型的应用程序非常有用。在构建传统的 Web 应用程序时，控制器和视图之间通常有着紧密的集成。当构建可以在智能手机上使用的应用程序时，它也非常有用。在这种情况下，前端将使用另一种编程语言和/或框架为智能手机的操作系统创建。在这种情况下，可能只会使用控制器和模型。无论哪种情况，拥有一个良好文档化的 RESTful API 是现代软件设计的重要组成部分。

嵌套控制器帮助开发人员立即阅读代码——这是一种理解特定控制器处理“嵌套”或一个类与另一个相关联的概念的简单方法。

在控制器中对模型和对象进行类型提示也提高了可读性，同时减少了执行对象的基本操作所需的代码量。

此外，雄辩的模型转换为模型的属性提供了一种简单的方式，无需依赖外部包或繁琐的访问器函数，就像在 Laravel 4 中那样。

现在我们很清楚为什么 Laravel 正在成为许多开发人员的选择。学习并重复本章中所述的一些步骤将允许在一个小时内为一个中小型程序创建一个 RESTful API。

# 总结

RESTful API 为将来扩展程序提供了一种简单的方式，也与公司内部可能需要与应用程序通信的第三方程序和软件集成。RESTful API 是程序内部的最前端外壳，并提供了外部世界与应用程序本身之间的桥梁。程序的内部部分将是所有业务逻辑和数据库连接所在的地方，因此从根本上说，控制器只是连接路由和应用程序的工作。

Laravel 遵循 RESTful 最佳实践，因此文档化 API 对其他开发人员和第三方集成商来说应该足够容易理解。Laravel 5 为框架引入了一些功能，使代码更易读。

在未来的章节中，将讨论中间件。中间件在路由和控制器之间添加了各种“中间”层。中间件可以提供诸如身份验证之类的功能。中间件将丰富、保护并帮助将路由组织成逻辑和功能组。

我们还将讨论 DocBlock 注释。虽然 PHP 本身不支持注释，但可以通过 Laravel 社区包启用。然后，在控制器和控制器函数的 DocBlock 中，每个控制器的路由将自动创建，而无需实际修改`app/Http/routes.php`文件。这是 Laravel 轻松适应的另一个伟大的社区概念，就像 phpspec 和 Behat 一样。
