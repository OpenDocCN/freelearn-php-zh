# 第五章。使用集合增强结果

到目前为止，我告诉你了关于模型以及如何创建它们之间关系的一切。我解释了如何查询你的数据和关系，甚至如何指定复杂的条件和约束。然而，我从未告诉你关于 Eloquent 输出的一切。是的，有时我提到了一个*数组*或者只是说*结果*。别担心，这并不错，但还有更多隐藏的东西。

好吧，在本章中，我将讨论集合。当你从查询（例如使用`get()`或`all()`）检索结果时，你得到的是一个集合。这就是正确的术语。

实际上，你可以将集合视为结果数组，但具有一些额外的实用方法。事实上，当你使用集合时，你正在使用`Illuminate\Database\Eloquent`中的`Collection`类的一个实例。

这个类实现了`AggregateIterator`接口，允许你将集合当作数组来处理。你可以使用集合执行许多操作，有时甚至是复杂的操作。首先，你将看到如何使用集合执行一些基本的研究操作和检查。

然后，我们将看到一些结果转换方法。你还记得吗，在第三章中，我提到了模型结果在 JSON 中的自动转换？太棒了！这是这些方法之一。

接下来，我们将深入探讨；毕竟，集合是由元素组成的。我们将与这些元素一起工作。显然，使用集合，你可以遍历其元素。有一些专门的方法用于迭代。此外，你还将学习如何以简单的方式过滤集合，就像 Eloquent 中的许多事情一样简单。最后，我们将讨论集合上的排序操作以及如何处理它们。所以，这些内容并不是真正必要的，但它将帮助你更好地理解 Eloquent 以各种方式的工作。

你准备好了吗？以下是我们要讨论的主题：

+   基本集合操作

+   转换集合

+   迭代和过滤

+   排序

# 基本集合操作

让我们从一些真正基本的方法开始。为了更好地理解你将要做什么，我**强烈建议**你在以下列表中的每个方法都在你的项目上尝试一次：

+   第一个是`contains()`。如果集合中包含具有特定 ID 的记录，它将返回 true 或 false。

    这里有一个例子：

    ```php
      $books = \App\Book::all();

      if($books->contains(3))
      {
        return 'yeah, book 3 is here!';
      }
    ```

    在这里，你只需要指定 ID 作为参数。

+   如我之前所说，你可以将集合当作数组来使用。所以，如果你想获取集合中的第三个元素，你可以这样做：

    ```php
      $book = $books[2];
    ```

+   然而，如果你出于某种原因不喜欢这种语法，你可以以这种方式使用`get()`作为替代，如下所示：

    ```php
      $book = $books->get(2);
    ```

+   使用这种*增强*语法，你还可以指定一个默认值，如果所需的索引不存在：

    ```php
      $book = $books->get(2, "Not Found!");
    ```

+   显然，如果你想检查特定元素的存在，你可以使用 `has()`：

    ```php
      if($books->has(3))
      {
        $book = $books->get(3);
      }
    ```

+   与 `get()` 相反，你可以使用 `put()` 添加一个具有特定索引的元素：

    ```php
      $book = new Book;
      // other attributes assignment here...

      $books->put(2, $book);
    ```

    如你所想，第一个参数是期望的索引，第二个参数是值。

+   另一个很酷的方法是 `prepend()`，你可以用它将一个元素添加到特定的集合中。以下是语法：

    ```php
      $firstBook = new Book;
      // other attributes assignment here...

      $books->prepend($book);
    ```

+   如果你想要获取包含所有主键的数组，你可以使用专门的 `modelKeys()` 方法！

    ```php
      $books = \App\Book::all();

      $primaryKeys = $books->modelKeys();
    ```

    还没有结束；实际上，有许多你可以用于许多不同事情的方法。

+   例如，`random()` 方法可以从指定的集合中提取一个随机项：

    ```php
      $books = \App\Book::all();

      $randomBook = $books->random();
    ```

+   此外，你可以使用 `keys()` 或 `values()` 来获取仅包含键或值的数组。

    ```php
      $books = \App\Book::all();

      $keysArray = $books->keys();
      $valuesArray = $books->values();
    ```

+   什么？你想把你的集合当作栈来处理？没问题，`pop()` 和 `push()` 就在这里帮助你！

    ```php
      // let's get the last item!
      $books = \App\Books::all();
      $lastBook = $books->pop();

      // lets' add a new item ad the end!
      $book = new Book;
      // other attributes assignment here...

      $books->push($book);
    ```

+   现在，关于使用类似于你在模型上调用的 `here()` 语法在集合中搜索一个项，看看这个：

    ```php
        $books = \App\Book::all();

        $book = $books->where('title', 'Michael Strogoff');
    ```

    注意，这个方法返回另一个分类。这意味着你可以使用多个 `where()` 的链式调用。以下是一个更好的例子：

    ```php
      $books = \App\Book::all();

      $book = $categories->where('year', 1876)->where('page_count', 254);
    ```

+   让我们用 `perPage` 来结束本章的第一部分，这是一个非常直观的方法，它获取一定数量的项目，而你只需要指定你想要每页显示的项目数量。

    语法类似于这样：

    ```php
      $books = \App\Book::all();

      $secondPageBooks = $books->perPage(2, 10);
    ```

    通过这个简单的调用，你正在获取从第二页开始的 10 本书。我认为这是一个 Eloquent（以及 Laravel）提供的方法表达性语法的绝佳例子。

### 注意

如果你想了解更多关于 `Collection` 类及其提供的内容，请查看 [`laravel.com/api/5.0/Illuminate/Database/Eloquent/Collection.html`](http://laravel.com/api/5.0/Illuminate/Database/Eloquent/Collection.html) 或直接查看 `Illuminate\Database\Eloquent\Collection` 和 `Illuminate\Support\Collection` 类中的代码。

# 转换集合

很常见，Eloquent 会自动将集合转换成你可以以更好的方式输出的东西。例如，以下是我用来显示杂志网站新闻分类列表的代码：

```php
  $categories = \App\Category::all();
   return $categories;
```

这是相应的输出：

```php
  [
    {
      id: 1,
      name: "Editorial",
      slug: "editorial",
      description: "Qui est quo asperiores aliquid vitae possimus. Dolor consequuntur similique voluptatem a laborum dolorem ea repellendus. Aspernatur ducimus quis dolorum consequatur vel nam at. Aut omnis rem laborum.",
      created_at: "2015-04-21 10:14:37",
      updated_at: "2015-04-21 10:14:37"
    },
    {
      id: 2,
      name: "Interview",
      slug: "interview",
      description: "Rerum deleniti rerum aliquid laudantium id non voluptatum. Aut quia distinctio consequatur velit natus inventore sunt iusto. Non totam quis quam sint et.",
      created_at: "2015-04-21 10:14:37",
      updated_at: "2015-04-21 10:14:37"
    },
    {
      id: 3,
      name: "Reportage",
      slug: "reportage",
      description: "Adipisci et veritatis excepturi ullam explicabo. Eos dolore quas a vero. Optio voluptatem accusamus ex optio. Rerum rem quaerat qui maiores.",
      created_at: "2015-04-21 10:14:37",
      updated_at: "2015-04-21 10:14:37"
    }
  ]
```

然而，现在考虑以下代码：

```php
  $categories = \App\Category::all();
   return $categories;
```

然后，让我们将代码更改为这个：

```php
$categories = \App\Category::all();
dd($categories);
```

这就是发生的事情：

```php
  Collection {#152
    #items: array:3 [
      0 => Category {#155
        #connection: null
        #table: null
        #primaryKey: "id"
        #perPage: 15
        +incrementing: true
        +timestamps: true
        #attributes: array:6 [
          "id" => 1
          "name" => "News"
          "slug" => "news"
          "description" => "Qui est quo asperiores aliquid vitae possimus. Dolor consequuntur similique voluptatem a laborum dolorem ea repellendus. Aspernatur ducimus quis dolorum consequatur vel nam at. Aut omnis rem laborum."
          "created_at" => "2015-04-21 10:14:37"
          "updated_at" => "2015-04-21 10:14:37"
        ]
        #original: array:6 [
          "id" => 1
          "name" => "News"
          "slug" => "news"
          "description" => "Qui est quo asperiores aliquid vitae possimus. Dolor consequuntur similique voluptatem a laborum dolorem ea repellendus. Aspernatur ducimus quis dolorum consequatur vel nam at. Aut omnis rem laborum."
          "created_at" => "2015-04-21 10:14:37"
          "updated_at" => "2015-04-21 10:14:37"
        ]
        #relations: []
        #hidden: []
        #visible: []
        #appends: []
        #fillable: []
        #guarded: array:1 [
          0 => "*"
        ]
        #dates: []
        #casts: []
        #touches: []
        #observables: []
        #with: []
        #morphClass: null
        +exists: true
      }
      1 => Category {#156 ...}
      2 => Category {#157 ...}
    ]
  }
```

等等，等等；什么？只是改变一个 `dd()` 调用并返回？嗯，你可以使用两个特殊方法 `toArray` 和 `toJSON` 来看到这个魔法。如果你需要，你也可以手动使用它们，就像这样：

```php
  $books = \App\Book::all();

  $toArray = $books->toArray();
  $toJson = $books->toJson();
```

很酷，对吧？

### 注意

我之前使用的 `dd()` 函数是 Laravel 的一个实用工具。它是原生 PHP 的 `var_dump()` 和 `die()` 的混合。更准确地说，它会显示某个对象或变量的值，然后停止脚本。

# 迭代和过滤

有时候，你需要做的不只是将集合传递给视图，或者进行简单的 `toArray()` 调用。Eloquent 集合有许多你可以用来过滤和遍历其元素的方法。让我们看看实际操作！

## 迭代

首先，让我们从简单的迭代开始。你可以调用 `each()` 方法来遍历某个集合的元素：

```php
  $books = \App\Book::all();

  $books->each(function($book)
  {
    echo $book->title;
  });
```

你所要做的就是传递一个闭包作为第一个（也是唯一一个）参数，该闭包有一个参数：将被使用的单个项。在这个例子中，我只是打印了所有的标题。

## 过滤

如果你想要更复杂地过滤你的集合，你可以使用 `filter()`。让我们来看一个例子：我想选择所有在 1840 年之后打印的书籍。

```php
  $books = \App\Book:all();

  $books->filter(function($book)
  {
    if($book->year > 1840)
      return true;
    else
      return false;
  });
```

语法与上一个示例非常相似。你有一个闭包作为参数，传递一个单一参数；即集合项。

然而，这次你必须检查你的条件，如果你想包含（或不包含）这个项在 `result` 集合中，你需要返回 true 或 false。

因此，在这个特定的情况下，当前的 `$book` 值是在 1840 年之后打印的吗？太好了，请进。不是在 1840 年之后打印的？再见！

# 排序

最后，你可以使用某个字段对数据进行排序。这次你必须使用的方法是 `sortBy` 和 `sortByDesc`。我想你足够聪明，能够理解它们的作用，对吧？

然而，这里有一些示例：

```php
  // ordering books by title, ascending
  $books = $books->sortBy(function($book)
  {
      return $book->title;
  });

  // ordering books by creation date, descending;
  $books = $books->sortByDesc(function($book)
  {
      return $book->created_at;
  });
```

此外，如果你的闭包逻辑非常简单，你可以使用快捷方式，例如之前的示例：

```php
  $books = $books->sortBy('title');
  $books = $books->sortByDesc('created_at');
```

# 摘要

让我们明确一下；在我看来，了解集合的每一个方法并不是真的必不可少。然而，在某些需要特定方法来完成非常具体任务的情况下，它可能非常有用。我该如何表达呢？你知道的事情越多，你就越优秀！

现在，让我们继续前进！在这个短暂的休息之后，是时候进入事件的世界了！
