# TypeScript 的好处

TypeScript 使您能够编写 JavaScript 代码。它包括静态类型和其他在面向对象语言中非常常见的特性。此外，使用 TypeScript，您可以使用 ECMAScript 6 的所有特性，因为编译器将它们转换为当前浏览器可读的代码。

TypeScript 的一个特性是用户可以创建类型化的变量，就像在 Java 或 C#中一样（例如，`const VARIABLE_NAME: Type = Value`），不仅如此，TypeScript 还帮助我们编写干净、组织良好的代码。这就是为什么 Angular 团队为当前版本的框架采用了 TypeScript 的原因之一。

在开始之前，让我们看一下官方 TypeScript 文档中的内容：

"TypeScript 是 JavaScript 的一种有类型的超集，可以编译为普通的 JavaScript。

任何浏览器。任何主机。

在本章中，我们将在我们的环境中全局安装 TypeScript，以了解 TypeScript 文件在转换为 JavaScript 时会发生什么。不用担心；Angular 应用程序已经为我们提供了内置的 TypeScript 编译器。

在本章中，我们将涵盖以下内容：

+   安装 TypeScript

+   使用 TypeScript 的好处

+   如何将 TypeScript 文件转译为 JavaScript 文件

+   使用静态类型编写 JavaScript 代码

+   理解 TypeScript 中的接口、类和泛型

# 安装 TypeScript

安装和开始使用 TypeScript 非常简单。您的机器上必须安装 Node.js 和 Node 包管理器（NPM）。

如果您还没有它们，请前往[`nodejs.org/en/download/`](https://nodejs.org/en/download/)，并按照您的平台的逐步安装说明进行操作。

让我们按照以下步骤安装 TypeScript：

1.  打开终端并输入以下命令以安装 TypeScript 编译器：

```php
npm install -g typescript
```

请注意，`-g`标志表示在您的机器上全局安装编译器。

1.  让我们检查一下可用的 TypeScript 命令。在终端中输入以下命令：

```php
tsc --help
```

上述命令将提供有关 TypeScript 编译器的大量信息；我们将看到一个简单的示例，演示如何将 TypeScript 文件转译为 JavaScript 文件。

示例：

` tsc hello.ts`

` tsc --outFile file.js file.ts`

前面几行的描述如下：

+   `tsc`命令编译`hello.ts`文件。

+   告诉编译器创建一个名为`hello.js`的输出文件。

# 创建一个 TypeScript 项目

一些文本编辑器，如 VS Code，让我们有能力将 TS 文件作为独立单元处理，称为文件范围。尽管这对于孤立的文件（如下面的示例）非常有用，但建议您始终创建一个 TypeScript 项目。然后，您可以模块化您的代码，并在将来的文件之间使用依赖注入。

使用名为`tsconfig.json`的文件在目录的根目录创建了一个 TypeScript 项目。您需要告诉编译器哪些文件是项目的一部分，编译选项以及许多其他设置。

一个基本的`tsconfig.json`文件包含以下代码：

```php
{ "compilerOptions":
  { "target": "es5",
   "module": "commonjs"
  }
}
```

尽管前面的代码非常简单和直观，我们只是指定了我们将在项目中使用的编译器，以及使用的模块类型。如果代码片段指示我们使用 ECMAScript 5，所有 TypeScript 代码将被转换为 JavaScript，使用 ES5 语法。

现在，让我们看看如何可以借助`tsc`编译器自动创建此文件：

1.  创建一个名为`chapter-02`的文件夹。

1.  在`chapter-02`文件夹中打开您的终端。

1.  输入以下命令：

```php
tsc --init
```

我们将看到由`tsc`编译器生成的以下内容：

```php
{
"compilerOptions": {
/* Basic Options */
/* Specify ECMAScript target version: 'ES3' (default), 'ES5', 'ES2015', 'ES2016', 'ES2017','ES2018' or 'ESNEXT'. */
"target": "es5",
/* Specify module code generation: 'none', 'commonjs', 'amd', 'system', 'umd', 'es2015', or 'ESNext'. */
"module": "commonjs",
...
/* Strict Type-Checking Options */
/* Enable all strict type-checking options. */
"strict": true,
...
/* Enables emit interoperability between CommonJS and ES Modules via creation of namespace objects for all imports. Implies 'allowSyntheticDefaultImports'. */
"esModuleInterop": true
/* Source Map Options */
...
/* Experimental Options */
...
}
}
```

请注意，我们省略了一些部分。您应该看到所有可用的选项；但是，大多数选项都是被注释掉的。现在不用担心这一点；稍后，我们将更详细地查看一些选项。

现在，让我们创建一个 TypeScript 文件，并检查一切是否顺利。

1.  在`chapter-02`文件夹中打开 VS Code，创建一个名为`sample-01.ts`的新文件。

1.  将以下代码添加到`sample-01.ts`中：

```php
console.log('First Sample With TypeScript');
```

1.  回到你的终端，输入以下命令：

```php
tsc sample-01.ts
```

在 VS Code 中，你可以使用集成终端；在顶部菜单栏上，点击 View | Integrate Terminal [ˆ`]。

请注意，另一个文件出现了，但扩展名是`.js`。

如果你比较这两个文件，它们完全相同，因为我们的例子非常简单，我们使用的是一个简单的`console.log()`函数。

由于 TypeScript 是 JavaScript 的超集，这里也提供了所有的 JS 功能。

# TypeScript 的好处

以下是使用 TypeScript 的好处的一个小列表：

+   TypeScript 是强大的、安全的，易于调试。

+   TypeScript 代码在转换为 JavaScript 之前被编译，因此我们可以在运行代码之前捕捉各种错误。

+   支持 TypeScript 的 IDE 具有改进代码完成和检查静态类型的能力。

+   TypeScript 支持面向对象编程（OOP），包括模块、命名空间、类等。

TypeScript 受欢迎的一个主要原因是它已经被 Angular 团队采用；而且，由于 Angular 是用于开发现代 Web 应用程序的最重要的前端框架之一，这激励了许多开发人员从 AngularJS 的 1.x 版本迁移到 2/4/5/6 版本学习它。

这是因为大多数 Angular 教程和示例都是用 TypeScript 编写的。

1.  打开`sample-01.ts`，在`console.log()`函数之后添加以下代码：

```php
class MyClass {
  public static sum(x:number, y: number) {
  console.log('Number is: ', x + y);
  return x + y;
 }
}
MyClass.sum(3, 5);
```

1.  回到你的终端，输入以下代码：

```php
tsc sample-01.ts
```

1.  现在，当你打开`sample-01.js`文件时，你会看到以下截图中显示的结果：

使用 TypeScript 与生成的 JavaScript 进行比较

请注意，sum 类参数`(x:number, y:number)`被赋予了类型 number。这是 TypeScript 的一个优点；然而，由于我们根据类型和在函数调用`MyClass.sum(3, 5)`中使用数字，我们无法看到它的强大之处。

让我们做一个小改变，看看区别。

1.  将`MyClass.sum()`函数调用更改为`MyClass.sum('a', 5)`。

1.  回到你的终端，输入以下命令：

```php
tsc sample-01.ts
```

请注意，我们收到了一个 TypeScript 错误：

```php
error TS2345: Argument of type '"a"' is not assignable to parameter of type 'number'.
```

如果你使用 VS Code，你会在执行编译文件之前看到以下截图中的消息：

![](img/d90105da-eda8-4321-ad41-4a252b0c0842.png)编译错误消息

如前所述，VS Code 是 TypeScript 语言的强大编辑器；除了具有集成终端外，我们还能清楚地看到编译错误。

我们可以对 TS 文件进行一些修改，而不是每次都输入相同的命令。我们可以使用`--watch`标志，编译器将自动运行我们对文件所做的每一次更改。

1.  在你的终端中，输入以下命令：

```php
tsc sample-01.ts --watch
```

1.  现在，让我们修复它；回到 VS Code，用以下代码替换`MyClass.sum()`函数：

```php
MyClass.sum(5, 5);
```

要停止 TS 编译器，只需按下*Ctrl +* *C*。

# 使用静态类型编写 JavaScript 代码

在使用 TypeScript 时，你会注意到的第一件事是它的静态类型，以及下表中指示的所有 JavaScript 类型：

| **基本类型** | **对象** |
| --- | --- |
| 字符串 | 函数 |
| 数字 | 数组 |
| 空 | 原型 |
| 未定义 |  |
| 布尔 |  |
| 符号 |  |

这意味着你可以声明变量的类型；给变量分配类型非常简单。让我们看一些例子，只使用 JavaScript 类型：

```php

function Myband () {
  let band: string;
  let active: boolean;
  let numberOfAlbuns: number;
}
```

使用 TypeScript，我们有更多的类型，我们将在以下部分中看到。

# 创建一个元组

元组就像一个有组织的类型数组。让我们创建一个看看它是如何工作的：

1.  在`chapter-02`文件夹中，创建一个名为`tuple.ts`的文件，并添加以下代码：

```php
const organizedArray: [number, string, boolean] = [0, 'text',
      false];
let myArray: [number, string, boolean];
myArray = ['text', 0, false]
console.log(myArray);
```

前面的代码在 JavaScript 中看起来很好，但在 TypeScript 中，我们必须尊重变量类型；在这里，我们试图传递一个字符串，而我们必须传递一个数字。

1.  在你的终端中，输入以下命令：

```php
tsc tuple.ts
```

你将看到以下错误消息：

```php
tuple.ts(4,1): error TS2322: Type '[string, number, false]' is not assignable to type '[number, string, boolean]'.
 Type 'string' is not assignable to type 'number'.
```

在 VS Code 中，你会在编译文件之前看到错误消息。这是一个非常有用的功能。

当我们用正确的顺序修复它（`myArray = [0, 'text', false]`）时，错误消息消失了。

还可以创建一个元组类型，并将其用于分配一个变量，就像我们在下一个例子中看到的那样。

1.  返回到你的终端，并将以下代码添加到`tuple.ts`文件中：

```php
// using tuple as Type
type Tuple = [number, string, boolean];
let myTuple: Tuple;
myTuple = [0, 'text', false];
console.log(myTuple);
```

这时，你可能会想知道为什么前面的例子有`console.log`输出。

借助我们之前安装的 Node.js，我们可以运行示例并查看`console.log()`函数的输出。

1.  在终端中，输入以下命令：

```php
node tuple.js
```

请注意，你需要运行 JavaScript 版本，就像前面的例子一样。如果你尝试直接运行 TypeScript 文件，你可能会收到错误消息。

# 使用 void 类型

在 TypeScript 中，定义函数的返回类型是强制的。当我们有一个没有返回值的函数时，我们使用一个叫做`void`的类型。

让我们看看它是如何工作的：

在`chapter-02`文件夹内创建一个名为`void.ts`的新文件，并添加以下代码：

```php
function myVoidExample(firstName: string, lastName: string): string {
    return firstName + lastName;
}
console.log(myVoidExample('Jhonny ', 'Cash'));
```

在前面的代码中，一切都很好，因为我们的函数返回一个值。如果我们删除返回函数，我们将看到以下错误消息：

```php
void.ts(1,62): error TS2355: A function whose declared type is neither 'void' nor 'any' must return a value.
```

在 VS Code 中，你会看到以下内容：

![](img/1c7652c8-7333-4dbd-a38e-9247c6f90b4a.png)VS Code 输出错误

要修复它，用`void`替换类型`string`：

```php
function myVoidExample(firstName: string, lastName: string): void {
const name = firstName + lastName;
}
```

这非常有用，因为我们的函数并不总是返回一个值。但请记住，我们不能在*返回*值的函数中声明`void`。

# 选择退出类型检查 - any

当我们不知道从函数中期望什么时（换句话说，当我们不知道我们将返回哪种类型时），`any`类型非常有用：

1.  在`chapter-02`文件夹中创建一个名为`any.ts`的新文件，并添加以下代码：

```php
let band: any;
band = {
    name: "Motorhead",
    description: "Heavy metal band",
    rate: 10
}
console.log(band);
band = "Motorhead";
console.log(band);
```

请注意，第一个`band`赋值是一个对象，而第二个是一个字符串。

1.  返回到你的终端，编译并运行这段代码；输入以下命令：

```php
tsc any.ts
```

1.  现在，让我们看一下输出。输入以下命令：

```php
node any.js
```

你将在终端看到以下消息：

```php
{ name: 'Motorhead', description: 'Heavy metal band', rate: 10 }
 Motorhead
```

在这里，我们可以将*任何东西*赋给我们的`band`变量。

# 使用枚举

`enum`允许我们使用更直观的名称对值进行分组。有些人更喜欢称枚举列表为其他名称。让我们看一个例子，以便更容易理解这在实践中是如何工作的：

1.  在`chapter-02`文件夹中创建一个名为`enum.js`的文件，并添加以下代码：

```php
enum bands {
    Motorhead,
    Metallica,
    Slayer
}
console.log(bands);
```

1.  在你的终端中，输入以下命令以转换文件：

```php
tsc enum.ts
```

1.  现在，让我们执行这个文件。输入以下命令：

```php
node enum.js
```

你将在终端看到以下结果：

```php
{ '0': 'Motorhead',
 '1': 'Metallica',
 '2': 'Slayer',
 Motorhead: 0,
 Metallica: 1,
 Slayer: 2 }
```

现在，我们可以通过名称而不是位置来获取值。

1.  在`console.log()`函数之后添加以下代码行：

```php
let myFavoriteBand = bands.Slayer;
console.log(myFavoriteBand);
```

现在，执行*步骤 2*和*步骤 3*中的命令以检查结果。你将在终端中看到以下输出：

```php
{ '0': 'Motorhead',
 '1': 'Metallica',
 '2': 'Slayer',
 Motorhead: 0,
 Metallica: 1,
 Slayer: 2 }
 My Favorite band is:  Slayer
```

请注意，`band`对象中声明的所有值（乐队名称）都被转换为字符串，放在一个索引对象中，就像你在前面的例子中看到的那样。

# 使用 never 类型

`never`类型是在 TypeScript 2.0 中引入的；它意味着永远不会发生的值。乍一看，它可能看起来很奇怪，但在某些情况下可以使用它。

让我们看看官方文档对此的解释：

`never`类型表示永远不会发生的值的类型。具体来说，`never`是永远不会返回的函数的返回类型，也是永远不会为`type`保护下的变量为真的类型。

假设在另一个函数内调用的消息传递函数指定了回调。

它看起来会像以下代码：

```php
const myMessage = (text: string): never => {
    throw new Error(text);
}
const myError = () => Error('Some text here');
```

另一个例子是检查同时是字符串和数字的值，例如以下代码：

```php
function neverHappen(someVariable: any) {
    if (typeof someVariable === "string" && typeof someVariable ===
     "number") {
    console.log(someVariable);
    }
}
neverHappen('text');
```

# 类型：未定义和空

在 TypeScript 中，`undefined`和`null`本身就是类型；这意味着 undefined 是一种类型(`undefined`)，null 是一种类型(`null`)。令人困惑？undefined 和 null 不能是类型变量；它们只能被分配为变量的值。

它们也是不同的：null 变量意味着变量被设置为 null，而 undefined 变量没有分配值。

```php
let A = null;
    console.log(A) // null
    console.log(B) // undefined
```

# 理解 TypeScript 中的接口、类和泛型

**面向对象编程**（**OOP**）是一个非常古老的编程概念，用于诸如 Java、C#和许多其他语言中。

使用 TypeScript 的优势之一是能够将其中一些概念带入您的 JavaScript Web 应用程序中。除了能够使用类、接口等，我们还可以轻松扩展导入类和导入模块，正如我们将在接下来的示例中看到的那样。

我们知道在纯 JavaScript 中使用类已经是一个选项，使用 ECMAScript 5。虽然它很相似，但也有一些区别；我们不会在本章中讨论它们，以免混淆我们的读者。我们只会专注于 TypeScript 中采用的实现。

# 创建一个类

理解 TypeScript 中的类的最佳方法是创建一个。一个简单的类看起来像以下代码：

```php
class Band {
    public name: string;
    constructor(text: string) {
    this.name = text;
    }
}
```

让我们创建我们的第一个类：

1.  打开您的文本编辑器，创建一个名为`my-first-class.ts`的新文件，并添加以下代码：

```php
class MyBand {
    // Properties without prefix are public
    // Available is; Private, Protected
    albums: Array<string>;
    members: number;
    constructor(albums_list: Array<string>, total_members: number) {
        this.albums = albums_list;
        this.members = total_members;
    }
    // Methods
    listAlbums(): void {
        console.log("My favorite albums: ");
        for(var i = 0; i < this.albums.length; i++) {
            console.log(this.albums[i]);
        }
    }
}
// My Favorite band and his best albums
let myFavoriteAlbums = new MyBand(["Ace of Spades", "Rock and Roll", "March or Die"], 3);
// Call the listAlbums method.
console.log(myFavoriteAlbums.listAlbums());
```

我们在以前的代码中添加了一些注释以便理解。

一个类可以有尽可能多的方法。在前一个类的情况下，我们只给出了一个方法，列出我们最喜欢的乐队专辑。您可以在终端上测试这段代码，将任何您想要的信息传递给新的`MyBand()`构造函数。

这很简单，如果您已经接触过 Java、C#甚至 PHP，您可能已经看到了这个类结构。

在这里，我们可以将继承（OOP）原则应用于我们的类。让我们看看如何做到这一点：

1.  打开`band-class.ts`文件，并在`console.log()`函数之后添加以下代码：

```php
/////////// using inheritance with TypeScript ////////////
class MySinger extends MyBand {
    // All Properties from MyBand Class are available inherited here
    // So we define a new constructor.
    constructor(albums_list: Array<string>, total_members: number) {
        // Call the parent's constructor using super keyword.
        super(albums_list, total_members);
    }
    listAlbums(): void{
        console.log("Singer best albums:");
        for(var i = 0; i < this.albums.length; i++) {
            console.log(this.albums[i]);
        }
    }
}
// Create a new instance of the YourBand class.
let singerFavoriteAlbum = new MySinger(["At Falson Prision", "Among out the Stars", "Heroes"], 1);
console.log(singerFavoriteAlbum.listAlbums());
```

在 Angular 中，类非常有用于定义组件，正如我们将在第三章中看到的那样，*理解 Angular 6 的核心概念*。

# 声明一个接口

在使用 TypeScript 时，接口是我们的盟友，因为它们在纯 JavaScript 中不存在。它们是一种有效的方式来对变量进行分组和类型化，确保它们始终在一起，保持一致的代码。

让我们看一个声明和使用接口的实际方法：

1.  在您的文本编辑器中，创建一个名为`band-interface.ts`的新文件，并添加以下代码：

```php
interface Band {
    name: string,
    total_members: number
}
```

要使用它，请将接口分配给函数类型，就像以下示例中那样。

1.  在`band-interface.ts`文件中的接口代码之后添加以下代码：

```php
interface Band {
    name: string,
    total_members: number
}
function unknowBand(band: Band): void {
    console.log("This band: " + band.name + ", has: " +                 band.total_members + " members");
}
```

请注意，在这里，我们使用`Band`接口来为我们的`function`参数命名。因此，当我们尝试使用它时，我们需要在新对象中保持相同的结构，就像以下示例中的那样：

```php
// create a band object with the same properties from Band interface:
let newband = {
    name: "Black Sabbath",
    total_members: 4
}
console.log(unknowBand(newband));
```

请注意，您可以通过键入以下命令来执行所有示例文件

在您的终端中键入`tsc band-interface.ts`和`band-interface.js`节点。

因此，如果您遵循前面的提示，您将在终端窗口中看到相同的结果：

```php
This band: Black Sabbath, has: 4 members
```

正如您所看到的，TypeScript 中的接口非常棒；我们可以用它们做很多事情。在本书的课程中，我们将看一些更多使用接口在实际 Web 应用程序中的例子。

# 创建泛型函数

**泛型**是创建灵活类和函数的非常有用的方式。它们与 C#中使用的方式非常相似。它非常有用，可以在多个地方使用。

我们可以通过在函数名称后添加尖括号并封装数据类型来创建泛型函数，就像以下示例中的示例一样：

```php
function genericFunction<T>( arg: T ): T [] {
    let myGenericArray: T[] = [];
    myGenericArray.push(arg);
    return myGenericArray;
}
```

请注意，尖括号内的`t`（`<t>`）表示`genericFunction()`是通用类型。

让我们看看实际操作：

1.  在您的代码编辑器中，创建一个名为`generics.ts`的新文件，并添加以下代码：

```php
function genericFunction<T>( arg: T ): T [] {
    let myGenericArray: T[] = [];
    myGenericArray.push(arg);
    return myGenericArray;
}
let stringFromGenericFunction = genericFunction<string>("Some string goes here");
console.log(stringFromGenericFunction[0]);
let numberFromGenericFunction = genericFunction(190);
console.log(numberFromGenericFunction[0]);
```

让我们看看我们的通用函数会发生什么。

1.  回到您的终端并输入以下命令：

```php
tsc generics.ts
```

1.  现在，让我们使用以下命令执行文件：

```php
node generics.js
```

我们将看到以下结果：

```php
Some string goes here
 190
```

请注意，编译器能够识别我们作为`function`参数传递的数据类型。在第一种情况下，我们明确将参数作为字符串传递，而在第二种情况下，我们不传递任何东西。

尽管编译器能够识别我们使用的参数类型，但始终确定我们要传递的数据类型是非常重要的。例如：

```php
let numberFromGenericFunction = genericFunction<number>(190);
console.log(numberFromGenericFunction[0]);
```

# 使用模块

在使用 TypeScript 开发大型应用程序时，模块非常重要。它们允许我们导入和导出代码、类、接口、变量和函数。这些函数在 Angular 应用程序中非常常见。

然而，它们只能通过使用库来实现，这可能是浏览器的 Require.js，或者是 Node.js 的 Common.js。

在接下来的章节中，我们将说明如何在实践中使用这些特性。

# 使用类导出功能

任何声明都可以被导出，正如我们之前提到的；要这样做，我们只需要添加`export`关键字。在下面的例子中，我们将导出`band`类。

在您的文本编辑器中，创建一个名为`export.ts`的文件，并添加以下代码：

```php
export class MyBand {
    // Properties without prefix are public
    // Available is; Private, Protected
    albums: Array<string>;
    members: number;
    constructor(albums_list: Array<string>, total_members: number) {
        this.albums = albums_list;
        this.members = total_members;
    }
    // Methods
    listAlbums(): void {
        console.log("My favorite albums: ");
        for(var i = 0; i < this.albums.length; i++) {
            console.log(this.albums[i]);
        }
    }
}
```

现在我们的`Myband`类可以被导入到另一个文件中了。

# 导入和使用外部类

使用关键字`import`可以实现导入，并且可以根据您使用的库的不同方式进行声明。使用 Require.js 的示例如下：

+   回到您的文本编辑器，创建一个名为`import.ts`的文件，并添加以下代码：

```php
import MyBand = require('./export');
console.log(Myband());
```

使用 Common.js 的示例如下：

```php
import { MyBand } from './export';
console.log(new Myband(['ZZ Top', 'Motorhead'], 3));
```

+   第二种方法已被 Angular 团队采用，因为 Angular 使用 Webpack，这是一个构建现代 Web 应用程序的模块捆绑器。

# 摘要

在本章中，您看到了 TypeScript 的基本原则。我们只是触及了表面，但是我们为您提供了一个处理使用 TypeScript 开发 Angular 应用程序的坚实基础。

在本书的过程中，随着我们创建 Web 应用程序的进展，我们将增强您的理解。
