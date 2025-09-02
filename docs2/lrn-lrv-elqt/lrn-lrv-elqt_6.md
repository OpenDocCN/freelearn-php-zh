# 第六章。使用事件和观察者全面控制一切

你听说过单一职责原则吗？我希望你听说过。它是编程中的 SOLID 原则之一，它基本上说一个类只有一个且只有一个职责。换句话说，每个类都必须做*一件事*，而不是其他任何事情。

通常，当你构建软件的第一个版本时，一切都很顺利。然后，事情发生了。你的老板打电话来：是时候介绍一个新功能了，开发者！特别是如果*更新*意味着*在这里插入这个小小的额外行为*，你的代码库很容易变得庞大而杂乱。

非常马虎！然后，你就要与截止日期、测试、问答等等作斗争，这简直是一场 Odyssey。这不是一个好的做法，对吧？

现在，在软件开发的世界里，你可以找到许多技术和方法以优雅的方式向你的软件添加新功能。你可能听说过编程中的*事件*。

简而言之，可以说它包含这样的逻辑：*当 X 做这件事时，Y 必须做那件事*。

想象一下在你的应用程序中类似的情景：你刚刚完成你的应用程序，然后你说，“哦，我刚刚忘记给新用户发送一封电子邮件了！”

使用 Eloquent，你可以以两种方式处理这种情况。第一种方式是使用非常有趣的概念：模型事件。第二种方式是基于一个更高级的概念：模型观察者。

在这一章中，首先，你将学习有关 Eloquent 模型中事件的所有内容。然后，我将介绍模型事件：它们是什么，以及你会在什么情况下使用它们。

然后，我将为模型观察者做同样的事情。你将学习所有这些差异，以及它们的优缺点。显然，对于这两个概念，我将使用一个实际例子来展示如何在现实世界中使用它们。

你准备好了吗，英雄？

+   我应该在模型中使用事件吗？

+   模型事件

+   模型事件的例子

+   模型观察者

+   模型观察者的一个例子

# 我应该在模型中使用事件吗？

什么是事件？如果你在谷歌上搜索这个术语，你会得到多个结果。

例如，它可以定义为*发生或被认为发生的事情；一个事件，尤其是某个重要的事件*。它也可以定义为*在特定时间间隔内发生在特定地点的事情*。

我喜欢这两个定义，因为它们在这个上下文中非常合适。事实上，你可以将这个*特定的时间间隔*视为模型的生命周期，从某种意义上说。

你可以创建一个新的实例，更新现有的实例，或者删除它。

你能做的每个操作都与两个事件相关。

从基础开始：我刚刚创建了那条记录，我删除了那条记录，或者我正在更新那条记录，听起来很自然，对吧？

好的。现在，Eloquent 在模型生命周期中发生某些事情时触发一些事件。更准确地说，它们如下：

| 创建 | 已保存 |
| --- | --- |
| 创建 | 删除 |
| 更新 | 已删除 |
| 已更新 | 恢复 |
| 保存 | 恢复 |

对于每个操作，你都有两个独立的事件。正如你可能想象的那样，它们指的是不同的时刻。让我们以*创建*操作为例。

你有*创建*事件，你可以将其读作“创建操作即将发生。”然后，你有*已创建*，这意味着“创建操作刚刚发生”。

就像科学家说的：

| 操作 | 描述 |
| --- | --- |
| 创建 | 关于*t - 1*时刻 |
| 已创建 | 与*t + 1*时刻相关 |

因此，对于三个基本操作：创建、更新和删除，你有两个事件。

你还可以看到两个更多操作：保存和恢复。但是，不用担心，它们并不复杂，事实上：

+   **保存**：你所需要知道的是，保存操作与*创建*和*更新*都有关。假设你想添加一个行为，无论应用程序是创建记录还是保存现有记录。为什么要在两次声明相同的事情上浪费时间？只需使用“保存”通用操作即可完成。

+   **恢复**：当你在某个模型上启用了软删除功能并撤销对其的删除操作时，使用恢复操作。

好的，我知道你在想什么：关于深入概念呢？

# 模型事件

我们将要查看的第一个事件技术被称为**模型事件**。基本概念非常简单：

+   在`EventServiceProvider`类中，你可以添加一个特殊的事件监听器并将其绑定到某个闭包

+   在这个闭包中，你将能够指定你的新行为，而无需修改模型代码

+   这种绑定必须放在类的`boot()`方法中

这里是一个将*创建*用户事件与闭包绑定的简单示例，该闭包作为调用方法的参数传递。

闭包的`$user`参数包含相关用户的实例：

```php
  public function boot(DispatcherContract $events)
  {
      parent::boot($events);

      User::created(function($user)
      {
          // doing something here, after User creation...
      });
  }
```

如你所想，每个模型都有这些方法，这些方法以我们之前看到的事件命名。所以，如果你想将某个操作绑定到*已保存*的事件，例如，你必须使用以下方法：

```php
  User::saved(function($user)
    {
        // doing something here, after User save operation (both create and update)...
    });
```

另一个有趣的功能是使用*pre*方法停止当前操作的可能性。实际上，如果你使用以下任何一种：

+   `创建`

+   `更新`

+   `保存`

+   `恢复`

+   `删除`

如果你想要中止操作，可以决定返回布尔值`false`。

假设你想在用户电子邮件以`@deniedprovider.com`字符串结尾时中止创建操作。你可以这样做：

```php
  User::creating(function($user)
    {
      if(ends_with($user->email, '@deniedprovider.com'))
      {
        return false;
      }
    });
```

显然，你不能在创建、更新、保存、恢复和删除事件上做同样的事情；事件已经发生，你不能回到过去！

# 模型事件的示例

我们现在可以看看一些模型事件在实际中的应用示例。

让我们从伟大的经典开始。一位新用户加入了我们，我们想通过欢迎邮件向他们打招呼。这很简单！让我们打开`EventServiceProvider`并在调用`parent::boot()`之后添加此代码：

```php
  User::created(function($user){

    Mail::send('emails.welcome', ['user' => $user], function($message) use ($user)
    {
        $message->to($user->email, $user->first_name . ' ' . $user->last_name)->subject('Welcome to My Awesome App, '.$user->first_name.'!');
    });

  });
```

完成了！不是很容易吗？

### 注意

我假设你有一个 `welcome.blade.php` 视图在 `resources/views/emails` 文件夹下。我也假设你了解 Laravel 中发送电子邮件的基础知识。如果你需要更多信息，请访问 [`laravel.com/docs/5.0/mail#basic-usage`](http://laravel.com/docs/5.0/mail#basic-usage)。

这里是模型事件在行动中的另一个例子：

让我们假设我们有一个类（为了这个示例的目的），它被委托给向每个想要知道某个作者新书被添加的用户发送电子邮件。这个类的名字是 `NewBookNotifier`，方法名为 `forAuthor($authorId)`，其中 `$authorId` 是所需作者的键。

我们可以做一些类似的事情：

```php
  Book::created(function($book){

    $newBookNotifier = new NewBookNotifier();
    $newBookNotifier->forAuthor($book->author->id);

  });
```

完成！主要观点是这非常简单。正如我之前提到的，最重要的是每个模型都保持不变。你甚至可以添加非常复杂的行为，但你不会触及模型中的任何东西。这是一个很大的优势，因为如果你测试这个模型并且没有触及它，你可以确信它永远不会出错。

现在，我将谈论*更复杂的事情*。

# 事件观察者

模型事件很酷，我同意。然而，有时你可能需要更高级的功能。

当你使用 Laravel 时，你主要是在进行面向对象编程，并且可能希望对你的模型事件也这样做。你问题的答案是模型观察者，它是模型事件的更高级版本。

要使用它们，你只需要声明一个新的类，如下所示（可能在一个名为 `observers` 的专用文件夹中）：

```php
  class BookObserver {

    public function creating($book)
    {
      // I want to create the $book book, but first...
    }

      public function saving($book)
      {
          // I want to save the $book book, but first...
      }

      public function saved($book)
      {
          // I just saved the $book book, so....
      }

  } 
```

然后，你可以在 `EventServiceProvider` 的 `boot()` 方法中注册它：

```php
  Book::observe(new BookObserver);
```

在 `EventServiceProvider` 类的 `boot()` 方法中。

没有更多的事情了，概念完全相同。使用观察者，你还可以使用之前在模型事件中学到的每一个概念。你可以声明你想要的每一个方法，要绑定特定的事件，只需使用事件标识符作为方法名。所以，*创建* 事件将与 `creating()` 方法相关，依此类推。

显然，如果你使用*预*方法，如*创建*或*更新*，你也可以中止操作：

```php
  class BookObserver {

    public function creating($book)
    {
      $somethingGoesWrong = true;

      if($somethingGoesWrong)
      {
        return false;
      }
    }

  }
```

好的，现在让我们看看几个使用模型观察者的例子！

# 模型观察者的一个例子

首先，这是你如何使用观察者做与第一个模型事件示例中相同的事情的方法。

在 `app/Observers` 下创建一个名为 `WelcomeUserObserver.php` 的新文件。现在，输入以下内容：

```php
  <?php

  namespace App\Observers;

  class WelcomeUserObserver {

    public function created($user){

      Mail::send('emails.welcome', ['user' => $user], function($message) use ($user)
      {
          $message->to($user->email, $user->first_name . ' ' . $user->last_name)->subject('Welcome to My Awesome App, '.$user->first_name.'!');
      });

    }

  }
```

然后，你可以在 `EventServiceProvider` 的 `boot()` 方法中注册观察者：

```php
  /**
   * Register any other events for your application.
   *
   * @param  \Illuminate\Contracts\Events\Dispatcher  $events
   * @return void
   */
  public function boot(DispatcherContract $events)
  {
      parent::boot($events);

      User::observe(new WelcomeUserObserver);
  }
```

嘿！你完成了。你的观察者现在已经附加到你的模型上了。

现在，让我们想象另一种情况。在开发者会议之后，我们发现图书管理员需要对代码库做一些小的介绍：

+   当系统添加新作者时，向每个用户发送通知

+   每次添加或删除作者时发送的电子邮件

最后，每次删除一本书时，图书管理员必须知道在数据库中没有相关书籍的作者数量。

好的。让我们开始。我们将构建三个独立的类：记住，我的朋友，这是*单一职责原则*。

我们将会有：

+   `CustomerNewAuthorObserver`

+   `LibrarianAuthorObserver`

+   `AuthorsWithoutBooksObservers`

### 注意

你可以按你的喜好命名你的类。我只是用这个约定作为一个例子，以便轻松地将行为与选定的名称联系起来。

然后，让我们创建三个独立的类：

```php
  <?php

  // file: app/Observers/CustomerNewAuthorObserver

  namespace App\Observers;

  class CustomerNewAuthorObserver {

    public function created($author)
    {

    }

  }

  <?php

  // file: app/Observers/LibrarianAuthorObserver

  namespace App\Observers;

  class LibrarianAuthorObserver {

    public function created($author)
    {

    }

    public function deleted($author)
    {

    }

  }

  <?php

  // file: app/Observers/AuthorsWithoutBooksObservers

  namespace App\Observers;

  class AuthorsWithoutBooksObservers {

    public function deleted($author)
    {

    }

  }
```

好的。现在，是时候添加一些逻辑了。

首先，让我们添加`CustomerNewAuthorObserver`：

```php
  <?php

  // file: app/Observers/CustomerNewAuthorObserver

  namespace App\Observers;

  class CustomerNewAuthorObserver {

    public function created($author)
    {
      // getting all users...
      $users = \App\User::all();

      foreach($users as $user)
      {
        Mail::send('emails.created_author_customer', ['author' => $author], function($message) use ($user)
        {
            $message->to($user->email, $user->first_name . ' ' . $user->last_name)->subject('New Author Added!');
        });
      }
    }

  }
```

### 注意

我知道这是一种非常粗鲁的方法。像往常一样，这只是为了教学目的。不要在家里尝试这样做！

然后，我们的`LibrarianAuthorObserver`类如下：

```php
  <?php

  // file: app/Observers/LibrarianAuthorObserver

  namespace App\Observers;

  class LibrarianAuthorObserver {

    public function created($author) {
      Mail::send('emails.created_author_librarian', ['author' => $author], function($message) use ($author)
      {
          $message->to('librarian@awesomelibrary.com', 'The Librarian')->subject('New Author: ' . $author->first_name . ' ' . $author->last_name);
      });
    }

    public function deleted($author) {
      Mail::send('emails.deleted_author_librarian', ['author' => $author], function($message) use ($author)
      {
          $message->to('librarian@awesomelibrary.com', 'The Librarian')->subject('New Author: ' . $author->first_name . ' ' . $author->last_name);
      });
    }

  }
```

最后，我们有以下内容：

```php
  <?php

  // file: app/Observers/AuthorsWithoutBooksObservers

  namespace App\Observers;

  class AuthorsWithoutBooksObservers {

    public function deleted($author) {
      $authorsWithoutBooks = \App\Author::has('books', '=', 0)->get();

      if(count($authorsWithoutBooks) > 0){
        Mail::send('emails.author_without_books_librarian', ['authorsWithoutBooks' => $authorsWithoutBooks], function($message)
        {
            $message->to('librarian@awesomelibrary.com', 'The Librarian')->subject('Authors without Books! A check is required!');
        });
      }
    }

  }
```

### 注意

如前所述，我假设你们都已经拥有了所有需要的视图并且知道如何处理电子邮件。如果没有，请查看[`laravel.com/docs/5.0/mail#basic-usage`](http://laravel.com/docs/5.0/mail#basic-usage)页面。

这还没有结束。你可以使用观察者和事件来解决大量的案例和场景。仅举一个例子，想象你正在写一个博客，并且每次你创建或编辑文章时都想重新生成你的网站地图。观察者是答案，或者你可能想在添加新书时记录一些东西——再次使用事件观察者！

# 摘要

太棒了！你现在能够处理各种形式的事件，从非常基础的概念到更高级的观察者概念。你刚刚为你的 Eloquent 知识增添了另一个小片段：你走得越远，你将越多地了解如何制作复杂的应用程序。此外，我们也在尊重一些 SOLID 原则！

还不错，不是吗？然而，不要把观察者和事件用于所有事情。有时候，它们并不是最佳选择，你必须使用其他工具。所以，要小心，分析你想要解决的个别问题。一个好的技术并不总是适用于所有事情。

好吧，现在是时候向前迈出另一步了。如果你想，可以休息一下；我们工作的*中间部分*已经完成了。实际上，在接下来的两个章节中，你将学习一些高级内容。

你准备好了吗？太好了！

翻到下一页，学习如何在没有 Laravel 的情况下使用 Eloquent！
