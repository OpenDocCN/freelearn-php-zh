# 第八章.还不够！扩展 Eloquent，高级概念

通过这本书，你学习了你可以用 Eloquent 做许多令人惊叹的事情。它是一个出色的活动记录实现；它易于使用，非常灵活，并且提供了许多工具来提高代码库的质量。

开发者通常必须面对两种类型的项目：*应用*和*应用*，如下所示：

+   对于*应用*，我打算做一些你可以做的事情，可能是一种快速的方式，一些在这里和那里的解决方案和技巧。而且，我知道你知道我在说什么。你为朋友做的那个网站，一个小博客，等等。

+   让我们明确并严肃，你不能用同样的完美关怀来制作每个应用。说实话，可能没有人能做到。那么，你必须处理*应用*。事情可能会变得非常严重，你必须能够构建一个可维护、出色的应用结构。

这不仅仅是关于一些能工作的事情；在这种情况下，我在想的是一些可以轻松扩展且代码质量良好的事情。这不仅仅是关于从控制器中调用模型。这还不够。你可能会遇到许多问题：可测试性、可维护性，以及遵循某些原则。

在本章中，我们将探讨两种不同的方法来更认真地扩展 Eloquent。

在第一部分，你将学习如何扩展 Eloquent 模型类。实际上，模型做了很多事情，但如果我们需要更多呢？没问题：站在巨人的肩膀上，扩展现有的模型类将会变得容易。

你听说过 Ardent 项目吗？这是一个为 Laravel 4 设计的包，它扩展了模型类，添加了一些超级功能：自我验证的模型和从请求输入数据中自动填充。

你将为 Laravel 5 Eloquent 模型做类似的事情，我会一步步地教你如何做。此外，它受到了菲利普·布朗在他的博客中的工作（[`culttt.com/2013/08/05/extending-eloquent-in-laravel-4/`](http://culttt.com/2013/08/05/extending-eloquent-in-laravel-4/)）的启发。

然而，正如本书前面提到的，Laravel 主要关于自由，尤其是在组织项目方面的自由。现在，一个真正有趣的趋势是**仓库模式**。这是一种以更好的方式抽象你的代码并分离责任的方法。对于比面包店的博客更大的项目，这是你知识库中必须拥有的。

此外，仓库模式不仅仅与 Laravel 有关。这意味着你将学习一些新东西，你将来可以在其他语言和产品中重用。

好吧，不再闲聊。现在是时候最后一次动手实践了。

来吧，英雄！我们将涵盖以下主题：

+   扩展模型：Aweloquent !

+   深入到仓库模式

+   摘要

# 扩展模型：Aweloquent！

Eloquent Model 实际上可以非常聪明且容易地完成很多事情。然而，在每次想要执行特定操作时，代码的编写方面可能需要改进。

通常，在创建新的模型实例时，你可能会使用用户之前输入到表单中的某些数据。

向我们的数据库添加新作者可以是一个完美的例子。你所要做的就是将姓名和姓氏输入到表单中，然后按下**保存**。

然后，在专用的路由（或相对控制器方法）中，你将执行以下类似操作：

```php
<?php

public function postAdd(Request $request)
{
  $author = new Author;

  $author->first_name = $request->input('first_name');
  $author->last_name = $request->input('last_name');

  $author->save();
}
```

这相当不错。然而，你可能还必须验证用户输入。

所以，假设你仍然在控制器中，你可以添加一个控制器验证调用，就像这样：

```php
<?php

public function postAdd(Request $request)
{
  $this->validate($request, [
    'first_name' => 'required',
    'last_name' => 'required'
  ], [
    'first_name.required' => 'You forgot the first name!',
    'last_name.required' => 'You forgot the last name!'
  ]);

  $author = new Author;

  $author->first_name = $request->input('first_name');
  $author->last_name = $request->input('last_name');

  $author->save();
}
```

再次，救了场！

现在，开发者们经常就单个类的职责进行辩论，讨论这个类应该做什么，不应该做什么。每个人对这个话题都有自己的看法。

**单一职责原则**，SOLID 原则的一部分，对此非常明确——简单来说，这个原则指出，一个类应该只做一件事情。

另一方面，然而，你经常会发现非常大的类。Eloquent Model 就是其中之一。在撰写本文时，Illuminate\Database\Eloquent\Model 有 3,399 行代码。这绝对不是一个小数字！

显然，Model 并不执行单一操作；它填充自己的属性，处理关系，并序列化自己的属性。是的，它远远超出了你刚才读到的原则。

那么，这到底是怎么回事呢？

好吧，即使它非常大，这样的 Model 也允许你使用一个单独的类执行许多操作。

一个完美的例子是，你可以如何将 Model 用作模型，如下所示：

```php
<?php

$user = new User;

// using magic methods...
$user->first_name = 'Francesco';
$user->last_name = 'Malatesta';

...
```

你还可以使用它作为工厂（一个用于以更优雅和更好的方式创建实例的类）使用`create()`方法：

```php
<?php

$user = User::create([
  'first_name' => 'Francesco',
  'last_name' => 'Malatesta',
]);
```

如果这还不够，Model 还处理与实例持久化相关的一切：

```php
<?php

$user = new User;

// some assignments...
$user->first_name = 'Francesco';
// ...

// and then save!
$user->save();
```

所有这些都可以使用一个单独的类——这是主要优势。

你可能正在问自己，所有这些话他到底想说什么。答案其实很简单：并不总是只有一个正确的解决方案。有些人讨厌 Eloquent Model，有些人则非常喜欢它。

所以，在这种情况下，我将向现有的 Model 类添加新功能，创建一个新的 Eloquent Model… **Aweloquent**！

### 注意

在我们继续前进之前，这里有一个澄清。我会再次重复，但我也想现在就说明。在本章的后续部分，我们将扩展 Model 类，添加一个在 Laravel 中由`Validator`类处理的功能。我之所以教你们这个，并不是因为我想要你们这样做，而是因为我想要展示如何扩展 Model 类。

例如，你可能觉得**智能密码哈希**功能很愚蠢，但这只是一个例子。扩展模型和使用仓库是两种完全不同的技术，它们是完全分开的。我只是给你提供知识，然后你可以选择做什么，我相信你会做出正确的选择，英雄！

## Aweloquent 模型

正如我之前提到的，我们的增强 Eloquent 模型将非常类似于 Ardent Laravel 4 包改进的模型。我还会从 Philip Brown 的 Magniloquent 项目中借鉴一些想法。

更精确地说，我们的改进模型将具有以下特性：

+   自动填充

+   模型自我验证

+   智能密码哈希

+   确认字段的自动清除

### 自动填充

而不是逐个分配属性，或者将它们作为一个数组传递，Aweloquent 模型将能够读取当前请求并自动填充其属性，而无需任何其他代码行。

这意味着你将能够使用以下内容：

```php
<?php

$user = new User;
$user->save();
```

而不是更经典的：

```php
<?php

$user = new User;

$user->first_name = 'Francesco';
$user->last_name = 'Malatesta';
// other assignments here...

$user->save();
```

### 模型自我验证

你将能够指定验证规则和消息作为模型的静态属性。然后，模型将自动执行你需要的验证操作，而无需使用任何外部类或控制器验证器。你还将能够将特定的规则分配给特定的操作（`'create'`或`'update'`操作，或者两者都分配）。

所以，在你的模型中，你将会有类似以下的内容：

```php
<?php namespace App;

use App\Aweloquent\AweloquentModel;

class Author extends AweloquentModel {

  protected $fillable = [
    'first_name', 'last_name', 'bio'
  ];

  protected static $rules = [
    'everytime' => [
      'first_name' => 'required'
    ],

     'create' => [
      'last_name' => 'required'
    ],

    'update' => [
      'bio' => 'required'
    ],
  ];

  protected static $messages = [
    'first_name.required' => 'You forgot the first name!',
    'last_name.required' => 'You forgot the last name!',
    'bio.required' => 'You forgot the biography!'
  ];

}
```

### 智能密码哈希

另一件你经常需要做的事情是**哈希密码**。通常，你会从`password`属性中获取值。因此，Aweloquent 模型会自动在存在`password`字段的情况下执行哈希操作。

### 确认字段的自动清除

Laravel 验证器有一个基于`x_confirmation`属性（其中`x`是字段的名称）的确认规则。你可能已经用它来为密码确认字段使用过了。Aweloquent 模型的自动清除功能会在验证后（当然）自动删除每个`_confirmation`字段。

然而，这还没有结束！Aweloquent 模型将自动排除由**跨站请求伪造**（**CSRF**）保护中间件使用的`'_token'`字段。

好的，这就结束了！现在你可以编写一些代码了。

## 扩展类

你首先必须做的是创建一个新的类，即扩展现有 Eloquent 模型的`AweloquentModel`类。

在我的特定情况下，我做了件非常简单的事情：我在`app`文件夹中创建了一个名为`Aweloquent`的新文件夹，然后在其中创建了一个`AweloquentModel.php`文件。

这是你必须放入此文件的代码：

```php
<?php

namespace App\Aweloquent;

use Illuminate\Database\Eloquent\Model;

class AweloquentModel extends Model {}
```

太好了！作为一个开始，我们有了新的`AweloquentModel`类。

如果你愿意，你可以将其用作未来模型的基础。这里没有变化，只是简单的扩展。

让我们添加第一个功能：自动填充。

## 自动填充功能

在我们实现这个第一个功能之前，让我们先思考一下我们想要的结果。

实际上，当你创建一个新模型时，你可以快速通过构造函数传递其属性：

```php
<?php

$user = new User([
  'first_name' => 'Francesco',
  'last_name' => 'Malatesta'
]);
```

这些参数是从构造函数传递到另一个名为 `fill()` 的方法：

```php
/**
   * Create a new Eloquent model instance.
   *
   * @param  array  $attributes
   * @return void
   */

  public function __construct(array $attributes = array())
  {
    $this->bootIfNotBooted();

    $this->syncOriginal();

    $this->fill($attributes);
  }
```

作为逻辑上的后果，如果我们想实现这个自动填充功能，我们必须编写一个新的构造函数来处理那里的自动填充并调用父类。所以，让我们回到我们的 `AweloquentModel` 类。这是第一个实现：

```php
<?php

namespace App\Aweloquent;

use Illuminate\Database\Eloquent\Model;

class AweloquentModel extends Model {

  public function __construct(array $attributes = [])
  {
    $attributes = $this->autoHydrate($attributes);

    parent::__construct($attributes);
  }

  private function autoHydrate(array $attributes)
  {
    // getting the request instance using the service container
    $request = app('Illuminate\Http\Request');

    // getting the request form data, except the token
    $requestData = $request->except('_token');

    foreach($requestData as $name => $value)
    {
      // manually specified attribute has priority over auto- 
	  hydrated one.
      if(!isset($attributes[$name]))
      $attributes[$name] = $value;
    }

    return $attributes;
  }
}
```

`autoHydrate` 方法创建了以下内容：

+   当前请求的一个实例以获取所需数据

+   之后，并且对于每个循环，它将请求数据数组中的每个元素（排除 CSRF 的 `'_token'`）添加到属性数组中

注意，显式指定的属性（你可以在模型构造函数中放入的属性）具有优先级，高于请求数据数组。因此，你仍然可以自由地处理模型并决定定义什么，不定义什么，也许可以添加一些你从表单中未获取到的额外数据。

如果你尝试通过设置基本表单来创建新用户，自动填充功能已经起作用了。

让我们继续前进！

## Aweloquent 模型自验证功能 – 基本版本

是时候实现我们 Aweloquent 模型的自验证功能了。这个想法很简单：对于每个模型，你将能够声明（作为属性）规则和相关消息。所以，它应该看起来像这样：

```php
<?php namespace App;

use App\Aweloquent\AweloquentModel;

class Author extends AweloquentModel {

  protected $fillable = [
    'first_name', 'last_name', 'bio'
  ];

  protected static $rules = [
    'first_name' => 'required',
    'last_name' => 'required'
  ];

  protected static $messages = [
    'first_name.required' => 'You forgot the first name!',
    'last_name.required' => 'You forgot the last name!'
  ];

}
```

这些规则和消息将被 `validate()` 专用方法自动使用。我想实现的是类似这样的效果：

```php
<?php

$user = new User;

if(!$user->validate())
{
  dd($user->errors);
}
```

所以，让我们打开 `AweloquentModel.php` 文件并添加一些代码：

```php
<?php

namespace App\Aweloquent;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Support\Facades\Validator;

class AweloquentModel extends Model {

  protected static $rules = [];
  protected static $messages = [];

  public $errors;

  public function __construct(array $attributes = [])
  {
    $attributes = $this->autoHydrate($attributes);

    parent::__construct($attributes);
  }

  public function validate()
  {
    $validator = Validator::make($this->attributes, static::$rules, static::$messages);

    if($validator->fails())
    {
      $this->errors = $validator->messages();
      return false;
    }

    return true;
  }

  private function autoHydrate(array $attributes)
  {
    // auto hydrate method here...
  }

}
```

太好了！`Validator` `Facadec` 用于实例化一个新的验证器。静态 `$rules` 和 `$message` 属性用于 `make()` 方法。

然后，`$validator->fails()` 调用确定给定的模型是否有效。如果不有效，则使用验证错误 `MessageBag` 对象填充 `$errors` 属性。

显然，这是一个非常基础的验证系统。然而，我们可以做更多。例如，基于操作的验证实现会很好。

如果你想尝试，那就继续吧！它已经起作用了！

## Aweloquent 模型自验证功能 – 操作版本

为了实现自验证系统的先进版本，我们必须为每个模型定义规则格式。

在这个特定的版本中，我选择了类似这样的方法：

```php
<?php namespace App;

use App\Aweloquent\AweloquentModel;

class Author extends AweloquentModel {

  protected $fillable = [
    'first_name', 'last_name', 'bio'
  ];

  protected static $rules = [
    'everytime' => [
      'first_name' => 'required'
    ],

    'create' => [
      'last_name' => 'required'
    ],

    'update' => [
      'bio' => 'required'
    ],
  ];

  protected static $messages = [
    'first_name.required' => 'You forgot the first name!',
    'last_name.required' => 'You forgot the last name!',
    'bio.required' => 'You forgot the biography!'
  ];

}
```

`$message` 属性保持不变。唯一需要修改的是 `$rules`，正如你可以想象的那样。

在这个 `$rules` 的新版本中，你可以为单个操作 `'create'` 或两者都定义规则。如果你想在这两者中都使用规则，有一个专门的 `'everytime'` 项来避免规则重复。

当然，我们必须再次编辑我们的 `AweloquentModel`。这次，我们必须定义一个方法，该方法必须与现有的验证方法一起工作，并理解我们是创建还是更新它。

然后，将正确的规则合并到一个数组中，并验证模型是否符合这些规则。

让我们看看我们能做什么！考虑以下代码：

```php
<?php

class AweloquentModel extends Model {

  ...

  public function __construct(array $attributes = [])
  {
    // constructor remains the same…
  }

  public function validate()
  {
    static::$rules = $this->mergeValidationRules();

    $validator = Validator::make($this->attributes, static::$rules, static::$messages);

    if($validator->fails())
    {
      $this->errors = $validator->messages();
      return false;
    }

    return true;
  }

  private function mergeValidationRules()
  {
    // if updating, use "update" rules, "create" otherwise.
    if($this->exists)
      $mergedRules = array_merge_recursive(static::$rules['everytime'], static::$rules
      ['update']);
  else
    $mergedRules = array_merge_recursive(static::$rules['everytime'], static::$rules

    ['create']);

  $finalRules = [];

  foreach($mergedRules as $field => $rules){
    if(is_array($rules))
      $finalRules[$field] = implode("|", $rules);
    else
      $finalRules[$field] = $rules;
    }

    return $finalRules;
  }

}
```

太好了，我们做到了！

`validate()`方法变化不大。唯一大的不同在于新的一行：

```php
static::$rules = $this->mergeValidationRules();
```

基本上，我们说的是：“好的，现在将`$rules`属性分配给这个`mergeValidationRules()`方法的返回结果。”

然后，在`mergeValidationRules()`方法中，首先使用的指令是：

```php
if($this->exists)
```

这用于确定当前操作是插入还是更新。从这个值开始，我们可以获取正确的规则数组，并将它们与`everytime`规则合并。

你新的复杂自验证模型几乎可以使用了。

## 智能密码哈希和确认字段自动清除方法

我们必须实现的最后两个功能是智能密码哈希和确认字段自动清除方法。

第一件事非常简单直观：

```php
private function smartPasswordHashing()
{
  if($this->attributes['password'])
    $this->attributes['password'] = Hash::make($this- >attributes['password']);
}
```

如果存在`'password'`字段，则对其进行哈希处理。仅此而已！

即使它稍微长一点，`purgeConfirmationFields()`也不难理解：

```php
private function purgeConfirmationFields()
{
  foreach($this->attributes as $name => $value)
  {
    if(Str::endsWith($name, '_confirmation'))
      unset($this->attributes[$name]);
  }
}
```

这次，我使用了`Str`字符串实用类来使用`endsWith()`方法，该方法用于确定字符串是否以某个字符序列结束。每个`'_confirmation'`字段都被移除。

## 修复 save()模型方法

现在，我们需要修复的最后一件事情是`save()`方法。实际上，`save()`方法完全忽略了验证过程，这是不行的。所以，这是`AweloquentModel`类的最终版本：

```php
<?php

namespace App\Aweloquent;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Support\Facades\Hash;
use Illuminate\Support\Facades\Validator;
use Illuminate\Support\Str;

class AweloquentModel extends Model {

  protected static $rules = [];
  protected static $messages = [];

  public $errors;

  public function __construct(array $attributes = [])
  {
    $attributes = $this->autoHydrate($attributes);

    parent::__construct($attributes);
  }

  public function save(array $options = [])
  {
    if($this->validate())
    {
      $this->smartPasswordHashing();
      $this->purgeConfirmationFields();

      return parent::save($options);
    }
    else
      return false;
  }

  public function validate()
  {
    static::$rules = $this->mergeValidationRules();

    $validator = Validator::make($this->attributes, static::$rules, static::$messages);

    if($validator->fails())
    {
      $this->errors = $validator->messages();
      return false;
    }

    return true;
  }

  private function autoHydrate(array $attributes)
  {
    // getting the request instance using the service container
    $request = app('Illuminate\Http\Request');

    // getting the request form data, except the token
    $requestData = $request->except('_token');

    foreach($requestData as $name => $value)
    {
      // manually specified attribute has priority over auto- 
	  hydrated one.
      if(!isset($attributes[$name]))
        $attributes[$name] = $value;
    }

    return $attributes;
  }

  private function mergeValidationRules()
  {
    // if updating, use "update" rules, "create" otherwise.
    if($this->exists)
      $mergedRules = array_merge_recursive(static::$rules['everytime'], static::$rules

      ['update']);
    else
      $mergedRules = array_merge_recursive(static::$rules['everytime'], static::$rules

    ['create']);

    $finalRules = [];

    foreach($mergedRules as $field => $rules){
      if(is_array($rules))
        $finalRules[$field] = implode("|", $rules);
      else
        $finalRules[$field] = $rules;
    }

    return $finalRules;
  }

  private function smartPasswordHashing()
  {
    if($this->attributes['password'])
      $this->attributes['password'] = Hash::make($this- >attributes['password']);
  }

  private function purgeConfirmationFields()
  {
    foreach($this->attributes as $name => $value)
    {
      if(Str::endsWith($name, '_confirmation'))
        unset($this->attributes[$name]);
    }
  }
}
```

让我们详细分析`save()`方法：

```php
public function save(array $options = [])
{
  if($this->validate())
  {
    $this->smartPasswordHashing();
    $this->purgeConfirmationFields();

    return parent::save($options);
  }
  else
    return false;
}
```

首先要做的是验证整个输入。之后，如果一切正常，密码将被哈希处理，确认字段将被清除，因为我们不再需要它们了。

最后，调用`parent::save()`方法，操作完成。

### 注意

为了与父类保持完美的连续性，我声明了与父类相同的签名（包括`$options`数组参数）的`save()`方法。

就这样！`AweloquentModel`类已经完成，你可以在你的项目中随意使用它。你还学会了如何深入到`Model`类中并扩展它，以便添加新的方法、行为和功能。

# 深入了解仓库模式

如果你了解一些关于良好开发和最佳实践的知识，你可能听说过软件设计模式。

你可以将其定义为针对某种问题的有用解决方案模板，或者更准确地说：

> *"在软件工程中，设计模式是在软件设计给定上下文中对常见问题的一般可重用解决方案。设计模式不是可以直接转换为源代码或机器代码的最终设计。它是对如何解决问题的描述或模板，可以在许多不同情况下使用。模式是程序员在设计和应用或系统时解决常见问题的最佳实践。"* 

### 注意

这个摘录来自维基百科上的软件设计模式页面([`en.wikipedia.org/wiki/Software_design_pattern`](http://en.wikipedia.org/wiki/Software_design_pattern))。

现在，让我们专注于第二句话。

设计模式不是可以直接转换为源代码的东西。

这是最重要的部分，因为它解释了许多事情。这不是你为了 Laravel 或可能为了某种特定语言而专门学习的东西。

绝对地，一旦你了解了设计模式，它就会伴随你一生！

在本章的前一部分，你学习了如何创建 Eloquent 模型的改进版本。你通过向现有模型添加内容达到了目标。

然而，许多人不喜欢这种做法。他们坚信单一职责原则，所以每个类必须只做一件事情。没有更多！

让我们明确一点；我不想通过在这个长时间的辩论中添加我的无用观点来让你感到无聊。

在本章的最后部分，我将介绍一种可以真正有助于改进 Laravel 应用的特定设计模式——仓库模式。

## 嗨，仓库模式！

你知道我喜欢从例子开始解释一个概念。这也不例外。

想象一下，你正在为仓库构建一个应用程序。在这个仓库中，你可以存储任何你想要的东西，你可能有一个像这样的模型：

```php
<?php namespace Warehouse;

use Illuminate\Database\Eloquent\Model;

class Item extends Model {

  // properties and methods here...

}
You are probably using this model in a controller, like this one:
<?php namespace Warehouse\Http\Controllers;

class ItemsController extends Controller {

  public function getIndex()
  {
    $items = \Warehouse\Item::orderBy('created_at', 'DESC')- >paginate(30);

    return view('item.list', compact('items'));
  }

}
```

没什么好说的；它有效，你知道这一点。

这是一个简单项目的酷解决方案，你可以称之为**应用程序**。然而，如果我们没有**应用程序**，会发生什么呢？

想象一下，你正在帮助的业务在增长，管理层决定创建一个必须内部使用的移动应用程序。你可能需要为其他开发者实现一个 API。

如果你不知道如何处理这类事情，你很快就会开始编写重复的代码。在你的 Rest API 中，肯定会有一个 `\items` 端点，它执行你在控制器方法中执行的操作，例如：

```php
$items = \Warehouse\Item::orderBy('created_at', 'DESC')->paginate(30);
```

重复相同的代码很多次是不安全的。你知道这一点，对吧？

但别担心，英雄！解决方案被称为**仓库模式**。

这个概念的最佳定义可以在 Martin Fowler 的网站上找到([`martinfowler.com/eaaCatalog/repository.html`](http://martinfowler.com/eaaCatalog/repository.html))：

> *"仓库在领域和数据映射层之间进行调解，就像内存中的领域对象集合。客户端对象以声明性方式构建查询规范，并将它们提交给仓库以满足需求。"*
> 
> *对象可以被添加到和从仓库中移除，就像它们可以从一个简单的对象集合中添加和移除一样，由仓库封装的映射代码将在幕后执行适当的操作。"*

因此，想象一个仓库就像介于中间，抽象出所有必要的。

回到之前的例子，想象我们有一个包含专用方法`getRecent($perPage, $pageNumber)`的仓库。

我们将在控制器中使用相同的方法：

```php
 <?php namespace Warehouse\Http\Controllers;

class ItemsController extends Controller {

  public function getIndex(ItemsRepository $itemsRepository)
  {
    // this is an example...
    $items = $itemsRepository->getRecent(30, 1);	

    return view('item.list', compact('items'));
  }

}
```

在 Rest API 中，使用以下方法：

```php
<?php

Route::get('api/v1/items', function(ItemsRepository $repo){

  return $repo->getRecent(30, 1);

});
```

同样的代码被两次使用，但只写了一次。然而，还有更多；让我们看看如何在 Laravel 项目中实现仓库。

## 介绍仓库——一个具体的实现

开始使用仓库的最佳方式是实现一个具体的实现。正如我之前提到的，仓库位于控制器和模型之间。它是中间某个东西。

当你构建一个仓库时，你必须考虑到你将需要从仓库中获取什么。让我们为我们的`Author`模型想象一个例子。

我可能需要以下方法：

+   `getAll($perPage, $pageNumber)`: 这将返回数据源中每个作者的分页列表

+   `find($authorId)`: 这将返回具有特定主键的特定作者

+   `search($firstName, $lastName)`: 这将返回从第一个名字到最后一个名字的结果数组

足够的搜索和获取记录了！然而，我们还需要其他方法，这些方法专门用于数据持久化：

+   `create($authorData)`: 这将在数据源中保存一个新的作者

+   `save($authorData, $authorId)`: 这将使用特定的主键更新数据源中现有的作者

让我们开始！

首先，在`app`文件夹中创建一个新的目录。命名为`Repositories`。在其内部，创建一个名为`DbAuthorsRepository.php`的新文件。

这里是内容：

```php
<?php

namespace App\Repositories;

use App\Author;

class DbAuthorsRepository {

  private $model;

  public function __construct(Author $model)
  {
    $this->model = $model;
  }

  public function getAll($perPage, $pageNumber)
  {
    $authors = $this->model->skip(($pageNumber - 1) * $perPage)- >take($perPage)->get();
    return $authors->toArray();
  }

  public function find($authorId)
  {
    return $this->model->find($authorId)->toArray();
  }

  public function search($firstName, $lastName)
  {
    return $this->model
    ->where('first_name', 'LIKE', '%'.$firstName.'%')
    ->where('last_name', 'LIKE', '%'.$lastName.'%')
    ->get()
    ->toArray();
  }

  public function create($authorData)
  {
    return $this->model->create($authorData);
  }

  public function update($authorData, $authorId)
  {
    return $this->model->find($authorId)->update($authorData);
  }

}
```

这就是你可以在一些测试路由中使用它的方法：

```php
<?php

Route::get('authors', function(\App\Repositories\DbAuthorsRepository $repository){

  return $repository->getAll(10, 1);

});

Route::get('create_author', function(\App\Repositories\DbAuthorsRepository $repository){

  $repository->create([
    'first_name' => 'Francesco',
    'last_name' => 'Malatesta',
    'bio' => 'Lorem ipsum...'
  ]);

});

Route::get('update_author', function(\App\Repositories\DbAuthorsRepository $repository){

  $repository->update([
    'first_name' => 'Frank',
    'last_name' => 'Smith',
    'bio' => 'Other ipsum...'
  ], 6);

});
```

没有更多了！

通过创建一个仓库，你学习了如何改进你的软件架构和解决方案的抽象级别。此外，与本章前面提到的 Aweloquent 相比，这次你可以感受到职责的极大分离。

然而，这还没有结束；仓库模式还没有展示出它的全部力量。

## 在抽象上编码

我已经在本章的早期介绍了你 SOLID 原则。我提到了单一职责原则，SOLID 的*S*。现在我们接近尾声，我将介绍*D*。

依赖倒置原则是我最喜欢的之一，因为它真正强调了尽可能抽象你的代码库的重要性。

它的定义是：

> “A. 高级模块不应该依赖于低级模块。两者都应该依赖于抽象。”
> 
> “B. 抽象不应该依赖于细节。细节应该依赖于抽象。”
> 
> “-维基百科 ([`en.wikipedia.org/wiki/Dependency_inversion_principle`](http://en.wikipedia.org/wiki/Dependency_inversion_principle))”

简而言之，这个概念是你必须编写抽象，而不应该依赖于具体类。否则，更好的说法是：你必须依赖于抽象，而不是具体类。

在 PHP 中，谈论抽象是指使用接口和契约。

在某种意义上，Laravel 本身广泛使用这个概念。基本的 Laravel 包是`Contracts`，它由几个接口组成，这些接口指定了每个组件必须做什么以及如何做。

然而，这不仅仅是一个大型框架的问题。你可以在日常开发中应用这个原则。更具体地说，你可以将这个概念应用到仓库中。

我会向你展示如何！

## 仓库 – 一个完整的实现

在我们深入探讨之前，让我们向我们的软件引入一个小问题。

实际上，我们的情况是这样的：我们的控制器和路由使用`DbAuthorsRepository`类来获取数据，如下所示：

```php
Route::get('authors', function(\App\Repositories\DbAuthorsRepository $repository){

  return $repository->getAll(10, 1);

});
```

然后，`DbAuthorsRepository`类使用作者模型从物理存储中获取所需的数据：

```php
<?php

namespace App\Repositories;

use App\Author;

class DbAuthorsRepository {

  private $model;

  public function __construct(Author $model)
  {
    $this->model = $model;
  }

  public function getAll($perPage, $pageNumber)
  {
    $authors = $this->model->skip(($pageNumber - 1) * $perPage)- >take($perPage)->get();
    return $authors->toArray();
  }

}
```

现在，让我们假设我们的数据源发生了变化。出于某种原因（我知道，这相当矛盾），管理层想要切换到基于文件的存储。

你有两种处理这个问题的方法：

+   你尖叫并因恐惧而瘫痪

+   决定以更好的方式组织你的代码库，在你的仓库工作流程中引入接口

这里是计划：

使用 Laravel 服务容器，你可以决定将某个接口绑定到特定的实现。

因此，如果你为每个仓库创建一个接口，你将能够一次性编写代码，然后编写你需要的每个具体仓库，最后，要从一个仓库切换到另一个仓库，你只需更改一行代码。

然而，让我们一步一步来：

1.  首先，让我们为我们的作者仓库定义一个标准行为，定义一个`AuthorRepository`接口。在`app/Repositories/Contracts`中创建一个新的`AuthorRepository.php`文件。我将使用`Contracts`文件夹来存储接口。

    这里是新鲜文件的目录：

    ```php
    <?php

    namespace App\Repositories\Contracts;
    interface AuthorsRepository {

      public function getAll($perPage, $pageNumber);

      public function find($authorId);

      public function search($firstName, $lastName);

      public function create($authorData);

      public function update($authorData, $authorId);

    }
    ```

    接口是纯粹的抽象。我们在这里所说的就是，“当我构建一个新的作者仓库时，我不关心底层的实现。我不在乎我是使用 NoSQL 数据库还是平面文件驱动器。我只想每个仓库都实现所有这些方法。”

    以这种方式工作意味着我们可以为未来使用的每个组件（或在这个案例中是仓库）定义一个标准格式。

1.  现在我们可以更新我们的`DbAuthorsRepository`类以实现我们的接口。考虑以下行：

    ```php
    class DbAuthorsRepository {
    It now becomes:
    class DbAuthorsRepository implements AuthorsRepository {
    ```

    好的。现在，让我们看看整个机制在实际中的威力。

1.  首先，打开`app/Providers/AppServiceProvider.php`文件，并将此绑定添加到`register()`方法中：

    ```php
    public function register()
    {
      $this->app->bind(
        'App\Repositories\Contracts\AuthorsRepository',
        'App\Repositories\DbAuthorsRepository'
      );
    }
    ```

    Laravel 现在知道，每次你请求`AuthorsRepository`的实例时，它都必须使用服务容器创建一个`DbAuthorsRepository`的实例。

1.  为了测试我们的假设，打开路由文件并添加以下内容：

    ```php
    <?php

    Route::get('authors', function(\App\Repositories\Contracts\AuthorsRepository $repository){

        return $repository->getAll(10, 1);

    });
    ```

    结果将正好是我们所期望的。此外，使用方法注入技术，我们不需要显式调用服务容器。

1.  事实上，这个语法的替代方案可以是以下这样：

    ```php
    Route::get('authors', function(){

        $repository = app('App\Repositories\Contracts\AuthorsRepository');

        return $repository->getAll(10, 1);

    });
    ```

### 添加新的仓库

最后，我们接近了尾声。让我们回到我们的主要问题：我们必须实现一个新的基于文件的作者仓库。

到目前为止，这相当简单：

1.  首先，在`app/Repositories`中创建一个新的文件，命名为`FileAuthorsRepository`。它将是一个新的类，当然。

1.  它将实现`AuthorsRepository`接口，这是显而易见的。

    这里是类的内容：

    ```php
    <?php

    namespace App\Repositories;

    use App\Repositories\Contracts\AuthorsRepository;

    class FileAuthorsRepository implements AuthorsRepository {

      public function getAll($perPage, $pageNumber)
      {
        dd('getting all records from flat file driver...');
      }

      public function find($authorId)
      {
        dd('searching by id: ' . $authorId);
      }

      public function search($firstName, $lastName)
      {
        dd('searching by first and last name...', $firstName, $lastName);
      }

      public function create($authorData)
      {
        dd('creating new author ', $authorData);
      }

      public function update($authorData, $authorId)
      {
        dd('updating author ' . $authorId, $authorData);
      }

    }
    ```

    如您所容易看到的，我已经从接口实现了所有必需的方法。这意味着我们的应用程序将能够以与 DbAuthorsRepository 相同的方式使用 FileAuthorsRepository。

1.  我在方法体中添加了一些`dd`指令，只是为了展示这个概念是如何工作的。为了我们的最后一步，前往`AppServiceProvider`类并更新之前的绑定到以下内容：

    ```php
    public function register()
    {
      $this->app->bind(
        'App\Repositories\Contracts\AuthorsRepository',
        'App\Repositories\FileAuthorsRepository'
      );
    }
    ```

1.  现在，浏览到`/authors`路由。是的，输出现在是：

    **"从平面文件驱动程序获取所有记录..."**

是的，它工作了！

我们的路由文件永远不会知道它正在使用哪个仓库：接口定义了你需要的所有方法。

这真是太棒了，因为例如，如果你想在将来将 NoSQL 仓库添加到你的应用程序中，你只需要创建一个新的`NoSQLAuthorsRepository`类，该类实现了`AuthorsRepository`接口。然后，在`AppServiceProvider`中，你将切换到所需的绑定。

简单、酷炫，而且可测试！

也许我有点重复，但请关注这个具体点：使用提到的结构，你可以将处理数据的方式与访问数据的方式抽象出来。我知道一遍又一遍地读同样的事情很无聊，但我需要你理解这个概念。

很可能，你第一次读到关于仓库的内容时，会想“我在这里做什么？见鬼了？”我也做过同样的事情，所以我完全理解你的疑惑。然而，当你处理更复杂的项目时，你将完全感受到这种差异。

这就是仓库的魔力！

# 摘要

我们完成了。在本章的最后，你学习了以两种不同且独立的方式增强你的应用程序：一方面，向现有实体添加功能。在这种情况下，是 Eloquent 模型。

在另一方面，你学习了如何使用仓库以不同的方式来结构你的应用程序，以便获得更好的代码可测试性、可维护性，以及目的的分离，而不是将一切委托给单个类。

现在你已经拥有了构建优秀应用程序所需的所有工具，使用 Eloquent 和 Laravel 吧。

你还在等什么？继续吧，让我为你感到骄傲！
