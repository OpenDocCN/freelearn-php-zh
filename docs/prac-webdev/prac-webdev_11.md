# 第十一章。MongoDB

在本章中，我们将首先比较传统的关联数据库和所谓的**NoSQL 数据库**，并快速介绍**MongoDB**，一个**文档数据库**。你将了解它是什么，如何获取和安装它，以及如何创建数据库以插入文档。然后，你将学习如何使用 PHP 作为语言与之交互。

# 关系数据库管理系统

在第六章中，我们介绍了 MySQL 数据库以及如何使用 PHP 语言与之交互。这并不是真正与数据库对话的语言。我们使用 PHP 来编写查询字符串，这些字符串是另一种语言的代码行，这种语言是数据库的本地语言**SQL**。它用于查询数据库，从中提取数据，替换数据或插入数据。我们提到，SQL 是那些存在多年且预计还会存在很长时间的非常少数语言之一。它被用于许多不同的数据库，开源或商业的，小或大的，我们统称为**关系数据库管理系统**（**RDBMS**）。

关系数据库使用表格。列代表存储在其中的数据类型，行代表数据本身。你可以有一个包含客户数据的表格，包含如*姓名*、*名*、*地址*、*邮编*、*城市*、*电话*、*账户号*等字段。所有表格都包含索引，可以在其他表格中使用以显示它们之间的关系。索引本身的值永远不应该改变。因此，如果客户更改地址，地址需要在客户表中更改，并且只在该客户表中更改，就像在其他所有表中一样，客户将通过他的或她的`customer_id`被引用。这是一件好事，但随着你的数据变得更加复杂，表格的数量也会增加。

我在使用这个系统的经验是，一旦需要记录包含两种相同类型的信息，比如当某人有两个地址时，你会觉得需要创建另一个表格。

访问数据以组成网页 HTML 的查询可能会变得相当复杂，一个接一个的内部连接（INNER JOIN）。一旦需要添加表格或向现有表格中添加列，你也必须更改你的代码。

在使用关系数据库管理系统（RDBMS）的项目中，这是一个常见的做法：首先定义数据库的表格，然后开始编写代码。表格的定义被放在一个模式或图中。因此，每次需要添加或更改表格时，你的模式也需要相应地更改。

总结来说，随着数据库的增长和数据的复杂化，可以创建大量的开销：添加表格或列、编写复杂的查询或不断更改数据库的模式。如果能够使用不同类型的数据库，其中每个记录都包含你需要的所有信息，而没有其他信息，那岂不是很好？是的，会很好。

# NoSQL 数据库

近年来，出现了几种新的数据库技术，它们不是关系型数据库管理系统（RDBMS）。其中大多数，如果不是全部，都试图解决一个特定的问题（而不是想要成为适合所有人的通用解决方案）。它们被统称为 NoSQL 数据库。NoSQL 这一部分可以有两种解释：**NO SQL**，意味着没有可用的查询语言，或者**Not Only SQL**，意味着这项技术支持不仅仅是 SQL。NoSQL 数据库在 Web 开发中已经变得相当流行。

有几种类型。对我们感兴趣的 NoSQL 数据库组是所谓的文档数据库。存储在数据库中的是文档。在这个上下文中，文档不是，比如说，Word 或 PDF 文档，而是一个 JSON 对象。文档数据库提供了在文档的任何字段上进行查询的能力，因此它们无疑是 Not Only SQL 数据库。

我们在这里所做的是与 RDBMS 中发生的事情相反。每个记录或文档将包含我们想要的所有信息，没有其他信息。如果需要向一个以前从未使用过的文档中添加项目，我们只需添加即可。不需要更改模式或表。如果一个记录现在需要两种相同类型的东西，我们按照 JSON 的方式行事：我们使用数组而不是单个对象。

令人惊讶的是，这些数据库即使在拥有数千个文档的情况下也能表现出色，并且具有良好的扩展性。我们为项目以及本书所选用的 NoSQL 数据库是 MongoDB。它似乎拥有最多的功能，被最广泛采用——**《纽约时报》**也在使用它！——并且具有良好的扩展性和性能。

# MongoDB

使用 MongoDB，我们只需用文档数据库替换 MySQL，这可能成为我们项目的更好解决方案，并且以 JSON 作为共同的基础。MongoDB 使用集合和文档。这使得它更适合，例如，在线书店应用程序。

由于 MongoDB 中的文档是一个 JSON 对象，因此从 PHP 中向其传输数据就像在 PHP 数组中存储数据一样简单。不再需要多个表和内部连接。我们将解释如何连接到 MongoDB，创建数据库，以及添加和更新文档。我们将从 PHP 程序以及 MongoDB shell 中这样做。

## 安装 MongoDB

获取和安装 MongoDB 取决于您所使用的平台。请访问[www.mongodb.org](http://www.mongodb.org)以获取软件和文档。这将为您提供基本上两个程序。第一个是**mongod**（被称为**MongoDB 守护进程**），一个需要一直运行和开启的程序，就像 MySQL 服务器一样。第二个程序简单地称为**mongo**，它是**MongoDB shell**。它是一个命令解释器，允许您创建数据库、集合和文档，并对其进行修改。我喜欢将其视为 MongoDB 的**phpMyAdmin**。

如果你想在程序内部访问和操作你的 MongoDB 数据库，你还需要下载并安装一个适用于你选择编程语言的 MongoDB 驱动程序。所以，对于 PHP，我们需要一个 MongoDB 的 PHP 驱动程序。驱动程序可以从源代码构建，但也有一些来自不断变化的发行版的二进制文件。所以，使用你喜欢的搜索引擎，查找 MongoDB 的 PHP 驱动程序。在撰写本文时，`php.net/manual/en/book.mongo.php`有一个。

## MongoDB shell

一旦安装好所有这些工具，就是时候开始用文档填充数据库了。在 MySQL 中，我们首先创建所有表格，可能最初是在一张纸上，然后通过使用 phpMyAdmin 来实际操作。在 MongoDB 中，我们使用 MongoDB shell 来首次填充我们的数据库。然而，没有必要创建表格。我们只需添加文档。正如他们所说的法语：*简单得就像你好*。

为了做到这一点，我们在数据库、集合和文档之间做出区分。每个 MongoDB 实例可以包含一个或多个数据库。虽然你可以用单个数据库做所有事情，但建议在适当的时候使用单独的数据库，例如，每个项目一个数据库。

虽然你可以像那样将所有文档放入数据库，但最好将它们组织为几个集合。与 RDBMS 相比，文档相当于表行，而集合相当于表。

在本节的剩余部分，我们将向您展示如何使用 MongoDB shell 进行基本的**创建、读取、更新和删除**（**CRUD**）操作。对于更详细和高级的操作，请参考优秀的 MongoDB 文档，网址为[mongodb.org](http://mongodb.org)，或者一些关于这个主题的书籍。

要启动 MongoDB shell，只需在命令行中输入`mongo`。你会看到类似以下的内容：

```php
MongoDB shell version: 2.6.6
connecting to: test
>

```

你现在正在与命令解释器对话，它基本上读取 JavaScript 代码。默认情况下，mongoDB 会连接到一个名为`test`的数据库。`>`符号是你的命令提示符。为了展示它的工作，让我们立即更改它：

```php
> prompt="mongoDB "
mongoDB

```

### 创建数据库、集合和文档

我们已经将命令提示符更改为字符串 mongoDB。让我们连接到另一个数据库：

```php
mongoDB   use california

```

你现在已经切换到了名为`california`的数据库。你认为我忘记了什么，但我告诉你我没有，我只是想保持悬念。现在，让我们向`people`集合添加一个文档。只需输入：

```php
mongoDB db.people.insert( {
 "name":"Adams",
 "first":"Ansel",
 "profession":"photographer",
 "born":"San Francisco"
 });

```

让我们找出这里是否真的发生了什么。有一个方法可以找到集合中的所有文档，不出所料，它被称为`find()`：

```php
mongoDB db.people.find()

{ "_id" : ObjectId("54ad4336227ad31227c3902d"), "name" : "Adams", "first" : "Ansel", "profession" : "photographer", "born" : "San Francisco" }

```

哇，我们现在创建了一个名为`california`的数据库，在其中，一个名为`people`的集合，以及它里面的第一个文档。所有这些事情都发生了；如果数据库或集合尚未存在，MongoDB 将创建它们。与 MySQL 不同，在创建数据库或表之前不需要任何 SQL，也不需要指定数据类型。

如果你输入以下两个命令，你确实会看到它们现在存在：

```php
mongoDB show dbs
...
california
...
mongoDB show collections
...
people
...

```

有一个`find`的替代方法，它提供了一个更友好的输出，称为`findOne()`：

```php
mongoDB db.people.findOne()
{
 "_id" : ObjectId("54ad4336227ad31227c3902d"),
 "name" : "Adams",
 "first" : "Ansel",
 "profession" : "photographer",
 "born" : "San Francisco"
}

```

当然，我们只找到一件事情，但随着文档数量的增加，我们希望更加具体，所以我们可以这样说：

```php
mongoDB db.people.findOne({"first" : "Ansel"})

```

这看起来越来越像是一个查询，因为它就是！你可能注意到了一个名为`"_id"`的关键字和一个非常长的值。让我们花几行来讨论这个问题。

#### `_id`和 ObjectId

存储在 MongoDB 中的每个文档都必须有一个`"_id"`键。它可以任何类型，但默认为`ObjectId`。在一个集合中，每个文档都必须有一个唯一的`"_id"`值，这确保了集合中的每个文档都可以唯一标识。ObjectId 使用 12 字节存储空间。如前所述，如果插入文档时没有`"_id"`键，MongoDB 将自动将其添加到插入的文档中。

#### 加载脚本

mongoDB shell 可以解析 JavaScript，并且还有内置函数。一些你习惯使用的东西可能不起作用，比如`alert()`，因为它属于 Windows 对象的方法，而且只有在浏览器中运行时才可用。

`load()`是一个非常有用的函数。你可以在事先准备所有命令并将它们存储在一个文件中。为你的项目创建一个文件夹，进入它，并将以下内容（与上一章中我们的一些 JSON 示例非常相似）存储在一个名为`californiapeople.js`的文件中：

```php
db.people.insert({
    "name":"Adams",
     "first":"Ansel",
     "profession":"photographer",
     "born":"San Francisco"

  });

db.people.insert({  "name":"Muir",
     "first":"John",
     "profession":"naturalist",
     "born":"Scotland"

  });
db.people.insert(   {
    "name":"Schwarzenegger",
     "first":"Arnold",
     "profession":"governator",
     "born":"Germany"

  });
db.people.insert( {
    "name":"Rowell",
     "first":"Galen",
     "profession":"photographer",
     "born":"Oakland CA"

  });
db.people.insert(   {
    "name":"Wellens",
     "first":"Paul",
     "profession":"author",
     "born":"Belgium"

  });
```

然后，输入以下命令：

```php
mongoDB      load("californiapeople.js")

```

这将向`people`集合中插入五个更多文档。为了验证它，输入以下命令：

```php
mongoDB db.people.find()

```

结果看起来可能像这样：

```php
{ "_id" : ObjectId("54ad4336227ad31227c3902d"), "name" : "Adams", "first" : "Ansel", "profession" : "photographer", "born" : "San Francisco" }
{ "_id" : ObjectId("54ad46f6812ce43efb530a07"), "name" : "Adams", "first" : "Ansel", "profession" : "photographer", "born" : "San Francisco" }
{ "_id" : ObjectId("54ad46f6812ce43efb530a08"), "name" : "Muir", "first" : "John", "profession" : "naturalist", "born" : "Scotland" }
{ "_id" : ObjectId("54ad46f6812ce43efb530a09"), "name" : "Schwarzenegger", "first" : "Arnold", "profession" : "governator", "born" : "Germany" }
{ "_id" : ObjectId("54ad46f6812ce43efb530a0a"), "name" : "Rowell", "first" : "Galen", "profession" : "photographer", "born" : "Oakland CA" }
{ "_id" : ObjectId("54ad46f6812ce43efb530a0b"), "name" : "Wellens", "first" : "Paul", "profession" : "author", "born" : "Belgium" }

```

#### 删除文档

注意现在有两个包含`Ansel Adams`的文档，它们通过其`"_id"`明显不同。但我们只需要一个，所以现在是时候删除一个了。这是通过`remove()`函数完成的。当心，如果你在以下命令中不指定任何参数，集合中的所有文档都将被删除，一旦文档被删除，就无法恢复：

```php
mongoDB db.people.remove( { "_id" : ObjectId("54ad46f6812ce43efb530a07" )} )

```

#### 更新文档

我们最后的 CRUD 操作是更新。在这个例子中，我们向包含有关已故自然摄影师和攀岩家加伦·罗威尔信息的文档中添加一个键值对。在这种情况下，我们为字段`"died"`设置了一个值。

```php
db.people.update ({"name" : "Rowell"}, { "$set": {"died" : "Bishop"}})

```

`$set`是一个**修饰符**的例子。它允许我们指定我们想要如何更改文档。在上一个例子中，如果该字段`"died"`尚未存在，其值将被更新或创建。

其他一些修飾符包括`$unset`，用於刪除字段，`$inc`和`$dec`用於增加和減少字段的值，以及`$push`和`$pull`用於向數組中添加元素或刪除它：

```php
mongoDB db.people.findOne({"name" : "Rowell"})
{
 "_id" : ObjectId("54ad46f6812ce43efb530a0a"),
 "name" : "Rowell",
 "first" : "Galen",
 "profession" : "photographer",
 "born" : "Oakland CA",
 "died" : "Bishop"
}

```

#### MongoDB 數據類型

在 MongoDB 中，我們不需要事先指定數據類型。你只需使用它們，到現在為止，應該已經從我們的例子中認識到構成 JSON 對象的相同數據類型。然而，在 MongoDB 中我們有一些額外的功能。

#### 基礎數據類型

我们可以使用与 JSON 相同的基礎數據類型：`null`、`true`、`false`、`string`、`number`和`array`。

#### 日期

日期總是難以處理，不僅在生活中，而且在數據庫中也是如此。幸運的是，MongoDB 支持 JavaScript 的`Date()`類，當使用日期時給我們一個數據類型。日期以自 epoch 以來經過的毫秒數存儲，不包含任何時區信息。當然，如果你喜歡，你可以將時區作為一個獨立的 key:value 對來存儲：

```php
{ "today" : new Date() }

```

在 MySQL 中，上述代碼的等價操作是使用`NOW()`。如果你在添加了`"today"`的文檔上執行`find()`，你將看到如下內容：

```php
"today" : ISODate("2015-01-14T08:37:17.086Z")

```

#### 嵌入式文檔

文檔中字段的值也可以是一個完整的文檔。我們稱這些為**嵌入式文檔**。

```php
{ "key" : { "name":"Schwarzenegger","first":"Arnold",
"profession":"governator" } }

```

### 再舉一個例子

現在讓我們用一個例子來結束 MongoDB shell 這一節。讓我們從第十章的`practical.json`文件中取我們的`practical.json`文件，*XML 和 JSON*，以及它包含的 JSON 對象。只需將它們包圍在`db.junelake.insert()`中，我們就可以將它轉換為 MongoDB 命令：

```php
db.junelake.insert ({
  "photocollection": {
    "title": "June Lake",
    "overview": "The June Lake Loop begins  ...",
    "photo": [
      {
        "scaption": "June Lake in the Fall",
        "caption": "June Lake and Carson Peak in the fall",
        "story": "Each time that unfortunate day  ...",
        "smallimg": "imagessmall/junelakefall.jpg",
        "largeimg": "imagespng/junelakeinthefall.png"
      },
      {
        "scaption": "Aspen by Silver Lake",
        "caption": "Aspen trees by Silver Lake",
        "story": "In 1998, I hit the right week of the ...",
        "smallimg": "imagessmall/silverlakeaspenfall98.jpg",
        "largeimg": "imagespng/silverlakeaspenfall98.png"
      },
      {
        "scaption": "Gull Lake in the Fall",
        "caption": "Gull Lake in the Fall - Happy fishermen !",
        "story": "If you take the north shore road around ...",
        "smallimg": "imagessmall/gulllake648.jpg",
        "largeimg": "imageslarge/gulllake648.jpg"
      },
      {
        "scaption": "Silver Lake",
        "caption": "Silver Lake - June Lake Loop",
        "story": "Any time of the year, there ...",
        "smallimg": "imagessmall/silver2.jpg",
        "largeimg": "imagespng/silver2.png"
      }
    ]
  }
});
```

一旦執行，無論是直接輸入還是先存儲在文件中，然後使用`load()`函數，我們現在已經將一份文件插入到我們圖庫數據庫的`junelake`集合中。我們甚至可以把它作為一個 JSON 對象來訪問並遍歷它：

```php
var json = db.junelake.findOne()

print (json.photocollection.title)
June Lake
```

我們可以使用`print()`函數來顯示這些值。我們甚至可以進行循環，就像我們在第十章中做的那樣，*XML 和 JSON*。這也是我選擇`json`作為變量名稱的原因，在那裡我們有`json`作為變量，它包含了 Ajax 調用返回的內容：

```php
for (var i in json.photocollection.photo) {
print(json.photocollection.photo[i].scaption) }
This will produce the following output:
June Lake in the Fall
Aspen by Silver Lake
Gull Lake in the Fall
Silver Lake
```

因此，既然我們可以直接從 MongoDB 數據庫中獲取 JSON，我們現在可以使用完全相同的 JavaScript 代碼來生成我們照片相冊的 HTML 頁面，對吧？錯誤！我們的數據庫位於服務器上，也是我們執行 MongoDB shell 的服務器（即使它在我的開發機器上，它仍然具有服務器的角色）。

為了讓生成我們 HTML 的 JavaScript 代碼有作用，它必須在客戶端執行，並由一個不同的 JavaScript 解釋器來解釋——從這本書開始我們一直在使用的：*瀏覽器*。

用一句著名电影台词来概括：*JSON，我们究竟如何才能把数据传到这里来？* 好吧，这将会和之前一样发生。我们将使用 Ajax 调用在服务器上执行代码以从我们的数据库中提取数据，这次是一个 MongoDB 数据库，然后客户端处理这些数据以生成我们的 HTML。

到目前为止，我们知道我们可以在服务器端使用的唯一编程语言是 PHP，因此我们需要一种方法来使用 PHP 访问我们的 MongoDB 数据库。

## MongoDB 和 PHP

我们将直接进入主题。我们在上一节通过将包含我们照片画廊页面所需数据的文档插入数据库来结束。然后我们发现我们无法立即使用它。然而，如果我们已经安装了 PHP 驱动程序并在`php.ini`文件中指定了 mongo 扩展，我们就可以从服务器上的 PHP 程序访问我们的 MongoDB 数据库。它可以非常简短。

### 获取我们的画廊数据

这就是全部所需：

```php
<?php
 $connection = new Mongo();
 $db = $connection->gallery;
 $junelake = $db->junelake;
 $gallery = $junelake->findone();
 $jsongallery =  json_encode($gallery);
 echo $jsongallery;
?>
```

多亏了驱动程序，我们有了对`Mongo()`类的访问权限。在第一行，我们连接到 MongoDB，在第二行，我们选择我们想要使用的数据库，在第三行，我们指定我们正在处理的集合。

接下来，我们使用`findone()`方法，这是我们之前章节中提到的。由于 PHP 不知道 JSON 对象，这将返回其他内容：一个数组。现在，记住有一个 PHP 函数可以将数组转换为 JSON：`json_encode()`。就在那里。只需`echo`它，以便 Ajax 调用能够捕获它，并在客户端完成剩余的工作。这里还有其他部分。我没有在这里重复 CSS 文件。

这是 HTML 文件：

```php
<!doctype html>
<html lang="en">
<head>
<meta charset="utf-8" />
<title>June Lake Gallery</title>
<link href="styles/practical.css" rel="stylesheet" type="text/css" media="screen"/>
</head>
<script src="img/jquery-1.10.1.js"></script>
<body>
<div id="mysite">
</div>
</body>
</html>
<script src="img/practicalmongo.js"></script>
```

这是 JavaScript 文件：

```php
$(document).ready (function () {
$.post( "mongogallery.php", function( json ) {

var html = '<div id="overview"><h1>' + json.photocollection.title + '</h1>';
html += '<p>' + json.photocollection.overview + '</p></div>';
html += '<div id="gallery">';

for (var i in json.photocollection.photo)
{
html += '<div class="storybook"><div class="smallmat"><div class="simage"><a href="';
var scaption = json.photocollection.photo[i].scaption;
var caption = json.photocollection.photo[i].caption;
var largeimg = json.photocollection.photo[i].largeimg;
var smallimg = json.photocollection.photo[i].smallimg;
var story = json.photocollection.photo[i].story;
html += largeimg;
html += '"><img class="sphoto" src="';
html += smallimg;
html += '" title="';
html +=  caption;
html += '"></img></a></div><div class="scaption">';
html +=  scaption;
html += '</div></div><div class="story">';
html +=  story;
html +=  '</div></div>';
}
html += '</div>';
$('#mysite').html(html);
}, "json");
});
```

我们使用了`.post()` jQuery 方法在服务器上执行我们的 PHP 脚本并收集 JSON 数据。注意`.post()`语句中的额外`"json"`参数。这非常重要，因为它告诉 Ajax 调用预期数据将以 JSON 格式提供，并且不要尝试将其作为 HTML 处理。如果您忘记这部分，可能会发生奇怪的事情，或者更糟，什么也不发生。

### 使用 MongoDB 和 PHP 进行 CRUD 操作

我们用 PHP 和 MongoDB 执行一些基本 CRUD 操作的概述来结束这一章。

正如我们提到的，PHP 不支持 JSON 对象，所以我们必须使用数组。在之前的例子中，我们从数据库中获取数据，所以我们甚至不需要查看数组，我们只需将其转换为 JSON。唯一令人失望的部分可能是在处理包含 JSON 对象的文档数据库时，我们获取数据，它显示为数组，我们必须将其转换回 JSON。这看起来似乎不是很高效，但如果您已经熟悉 PHP，编写起来很容易。

我们将在本书的最后一章讨论一个 JSON（以及 JavaScript）的替代方案。

#### 插入文档

让我们在 `california` 数据库的 `people` 集合中添加另一个著名的加利福尼亚人。那些熟悉威尼斯海滩的人都知道我在说什么。他是那个戴着头巾在滑板上上下滑行的人。30 年来他几乎没有变化，除了现在他卖 T 恤而不是卡带。

它与 MongoDB shell 中的 `insert()` 几乎相同，只是我们的 *key:value* 对现在是作为关联数组的一部分的 *key=>value* 对：

```php
<?php
 $connection = new Mongo();
 $db = $connection->california;
 $people = $db->people;
 $californiadude = array(
'first' => 'Harry', 'name' => 'Perry', 'address' => 'Boardwalk', 'city' => 'Venice', 'state' => 'CA','zip' => '90291', 'song' => 'Invaders from another Planet'
);
$people->insert($californiadude);
?>
```

#### 更新文档

让我们通过添加一个额外的键值对来更新我们的文档：

```php
$people->update(array('name' => 'Perry'),
array('$set' => array('vehicle' => 'rollerblades')));
```

到目前为止，在我们的例子中，我们可以使用双引号而不是单引号，但对于 `$set` 修改符，我们只能使用单引号，因为，正如你所知，`$` 在 PHP 中有特殊含义——如果没有单引号，`$set` 就会被解释为一个 PHP 变量。

#### 带条件的查询

就像我们在 shell 中做的那样，我们可以使用 `findone()` 函数来查询一个文档：

```php
$result = $people->findone(array('name' => 'Perry'));
print_r($result);  // in case we want to check our insert was succesful.
```

再次强调，我们使用数组而不是键值对。但我们可以使这些查询更加复杂。存在几个关键字可以用来细化我们的查询。这里有一个额外的例子：

```php
$result = $people->findone(array('first' => 'John' , 'age' => array ( '$gt' => 50)));
// look for all people named John that are over 50\. This is just an example. Nothing will
// be returned.
```

`findone()` 只返回一个文档，并以关联数组的形式返回，就像我们最初的例子一样。如果我们使用 `find()`，则会返回多个文档的数据。在 shell 中，我们得到了一个很好的小输出，默认为前 20 个文档，但在这里，使用 PHP，我们将得到一个称为 MongoDB **游标**的东西。

#### MongoDB 游标对象

MongoDB 的 `db.collection.find()` 返回所有文档作为一个可迭代的对象，或者游标。在 PHP 中，`find()` 返回所有文档及其所有数据，这可能不是你想要的。但因为我们现在没有太多数据，这里有一种方法可以获取所有数据并在 PHP 中遍历它：

```php
<?php
 $connection = new Mongo();
 $db = $connection->california;
 $people = $db->people;
$cursor = $people->find();
foreach ($cursor as $document)
{
    echo $document['name'];  // we could do more of course
}
?>
```

在例子中，`$cursor` 是 MongoDB 游标类的对象。这个类有几个方法可用，为你提供了遍历游标的不同方式。如果你想将游标转换为数组，可以使用 `iterator_to_array()` 函数。

在例子中，我们使用 `foreach()` 循环遍历 `$cursor`，这样我们就可以访问每个文档并将其作为数组来处理。我们做的是 `echo` 名称。

为了总结，我们将向你展示如何细化我们的查询。想象一下，你想执行相同的查询，在 SQL 中它看起来像：

```php
SELECT first, last FROM people WHERE age > 50 LIMIT 10;

```

在 MongoDB PHP 术语中，这将是：

```php
$cursor =          // the variable that will contain the cursor we can iterate through
$people->find(  // our collection, equivalent to a table
array('age' => array ('$gt '=> 50 ),  // criteria for our query
array ('name' => 1, 'first' => 1, '_id' => 0 ))
 // MongoDB calls this a projection:  which fields we want
)->limit(10);                         // cursor modifier
```

到现在为止，这应该对你来说已经很直接了。游标修改符部分允许你限制或影响返回的数据，例如，有一个 **排序** 修改符。

`projection` 数组允许你指定你感兴趣的哪些字段。默认情况下，`id` 的值总是返回，所以如果你不需要它，请在你的投影数组中添加 `_id => 0`。

# 概述

在本章中，你学习了 MongoDB，这是一种在 NoSQL 数据库中非常受欢迎的文档数据库。存储在数据库中的文档基本上就是 JSON 对象。它们可以被分组到集合中，这相当于 RDBMS 中的表。

使用 MongoDB shell，我们可以从命令行填充数据库和集合。在 Web 应用程序内部，我们可以使用 PHP 访问服务器上的 MongoDB 数据库。这真的很简单，因为 JSON 对象可以写成关联数组。

在最后几章中，有很多示例代码，我承诺你会有一本教科书，这样你就可以在电脑外学习。在下一章，将有更多的阅读内容。

当前 Web 开发的新趋势正在被讨论，这是由于人们使用 Web 的方式与几年前完全不同所引发的。我们将更多地关注你的网站外观，而较少关注其工作方式。
