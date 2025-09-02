# 第十四章 Node.js

恭喜！你现在已经到达了这本书的最后一章。在了解了古典和现代网络开发中使用的多数技术之后，我们现在将讨论我想要称之为网络开发的**前卫**（请原谅我的法语），的基本知识：node.js 及其朋友。

在我们迄今为止讨论的所有内容中，我们使用了通常所说的**LAMP**（或**MAMP**或**WAMP**，取决于服务器上的操作系统）堆栈：**Linux** **Apache MySQL PHP**。即使我们用**MongoDB**替换 MySQL，这个缩写仍然适用。或者我们可以称之为**LANP**，其中的 N 代表**NoSQL**。

因此，我们必须学习所有这些语言：客户端的 JavaScript，由浏览器进行解析；PHP，由 Apache 网络服务器进行解析；还有更多。想象一下，如果你可以替换掉所有东西，甚至是你不会想到替换的东西——网络服务器——而用 JavaScript 来替代？这正是 node.js 为你做的事情。在我看来，node.js 对网络开发的影响，就像 Schoenberg 和 Webern 对古典音乐的影响，Picasso 和 Braque 对绘画的影响，哦，为什么不呢，威尼斯海滩的说唱歌手对流行音乐的影响。

这里有好消息和坏消息。坏消息是我们有重新开始一切并白费很多功夫的风险，但好消息是，基于 node.js 的解决方案**性能**良好，**可扩展**，对于那些在我们之后的人来说，只需要了解一种编程语言：JavaScript。这个缩写的名称仍然有待确定：**LJMJ**或**LNMJ**（J 代表**JavaScript**或 N 代表**node.js**）或**MEN**（**MongoDB**，**Express**，**node.js**）？已经被广泛使用的一个是**MEAN**。随着时间的推移，人们将达成一致。谁发明了单词**立体主义**？

在这一章中，我们将尝试以 node.js 的方式重复你迄今为止所学的所有内容。所以，我们已经知道我们的代码将用哪种语言编写：JavaScript。我们将要编写的代码可能会让你感到惊讶。

# Node.js

让我们回顾一下；在第一章《万维网》中，你学习了关于万维网以及人们如何通过发送 HTTP 协议请求到网络服务器的浏览器来访问数百万个页面。嗯，人们仍然会使用发送 HTTP 请求的浏览器，但我们刚刚丢弃了网络服务器，现在我们该怎么办？我们编写一个。可怕吗？不，这将会非常有趣。编写底层代码不是你的强项？没关系，有人已经为你做了。有一个整个社区正在为 node.js 编写代码，其他人都可以使用。这些代码被提供为所谓的**模块**，当然，我们也有一个可用的 HTTP 模块。

Web 服务器为我们做的另一件事是分析用户输入的 URL，并探索**文件系统**以查看是否存在物理文件，例如，一个`hello.html`文件，并将其发送回客户端。我们也必须编写这个。这很酷，因为它将让我们完全控制我们的 Web 服务器应该能够处理什么，不应该处理什么。正如预期的那样，node.js 也有`url`和`fs`模块。

我们需要一个数据库，但我们已经知道我们喜欢的一个：MongoDB。我们能否用 node.js 使用它？是的，我们可以。有一个模块或驱动程序可以从 node.js 访问现有的 MongoDB 服务器。由于 MongoDB 是一个文档数据库，而文档实际上是 JSON 对象，所以在全 JavaScript 生态系统中这是一个完美的匹配。

逐渐地，我们开始意识到，在写出一个应用之前，这个 node 应用实际上是要成为一个包含特定代码的 web 服务器，还是一个包含 web 服务器的应用；任你选择。

当我带你通过我们的第一个示例时，你就会意识到我们以前从未需要为简单的`Hello, World`程序编写这么多代码。想象一下，如果需要用这么多底层代码编写一个功能齐全的单页 Web 应用会怎样？这正是*Express*发挥作用的地方。它是一个 node.js 框架，将帮助我们编写更干净、更紧凑的代码。这是服务器端的 jQuery。一旦我们的示例变得过于冗长，我们就会切换到 Express。

我们还丢弃了另一件事：Apache 作为一个**应用服务器**，这部分让我们在服务器上有了 PHP 语言。我们一直使用 PHP 在服务器上动态生成 HTML，等到浏览器读取时，它已经变成了全部的 HTML。

使用 PHP 的好处是，我们可以在纯 HTML 中嵌入 PHP 代码，在`<?php`和`?>`之间。用`<script>`标签填充 HTML 文件以包含 JavaScript 代码并不吸引人。我们也会看看这个问题的解决方案。

# 安装 node.js

我们不再拖延，现在就在你的电脑上安装 node。安装方式会根据你运行的操作系统而有所不同。访问[`nodejs.org/`](http://nodejs.org/)并获取正确的下载。结果在所有地方都是一样的：它给了我们两个程序：**node**和**npm**。

## npm

npm，**node 打包管理器**，是您用来查找和安装模块的工具。每次您编写需要模块的代码时，您可以通过在代码中放入类似以下内容来指定：

```php
var module = require('module');
```

如果尚未安装，需要使用以下命令进行安装：

```php
npm install

```

或者你也可以使用：

```php
npm -g install

```

后者会尝试在全局范围内安装模块，前者在命令执行的目录中安装模块。它通常会在名为**node_modules**的文件夹中安装模块。

## node

`node`命令是用来启动你的 node.js 程序的命令，例如：

```php
node myprogram.js

```

Node 将启动并解释你的代码。输入 *Ctrl* + *C* 来停止 node。让我们立即开始我们的第一个程序。

我们不可避免的 `Hello, world` 示例是最小的可能网络服务器：

```php
var http = require('http');
   http.createServer(function (req, res) {
     res.writeHead(200, {'Content-Type': 'text/plain'});
     res.end('Hello World\n');
   }).listen(8080, 'localhost');
   console.log('Server running at http://localhost:8080');
```

将此文件保存为 `hello.js`，或者从 Packt Publishing 网站获取，然后在终端窗口中输入：

```php
node hello.js

```

此命令将使用 `node` 运行程序。在终端窗口中，它将成为你的控制台，你会看到文本 `Server running`。

接下来，当你启动浏览器并输入 `http://localhost:8080` 作为 URL 时，会出现一个看起来像网页的东西，包含著名的两字句子 `Hello World`。实际上，如果你访问 `http://localhost:8080/it/does/not/matterwhat`，会出现相同的内容。可能不是非常有用，但它是一个网络服务器。

### 添加 HTML

这是一个略有不同的版本，我们明确指定发送 HTML 而不是纯文本：

```php
var http = require('http');
   http.createServer(function (req, res) {
     res.writeHead(200, {'Content-Type': 'text/html'});
     res.end('<h1>Hello World\</h1>');
   }).listen(8080, 'localhost');
   console.log('Server running at http://localhost:8080');
```

### 提供静态内容

我们不习惯无论我们给出什么路径作为 URL，都会出现相同的内容。通常，URL 指向一个文件（或一个文件夹，在这种情况下，服务器会查找 `index.html` 文件），`foo.html` 或 `bar.php`，并且当存在时，它会被提供给客户端。

那么，如果我们想用 node.js 来做这个呢？我们需要一个模块。有几种方法可以完成这项工作。在我们的例子中，我们使用 `node-static`。首先，我们需要安装它：

```php
npm install node-static

```

你通常可以在 Github 和其他酷炫的地方找到关于方法和属性的帮助文档。在我们的应用程序中，我们不仅创建了一个网络服务器，还创建了一个 `fileserver`。它将为本地目录 `public` 中的所有文件提供服务。将所有所谓的 `静态内容` 放在一起，放在一个单独的文件夹中是很好的。这基本上是所有将被客户端接收并解释的文件。由于我们最终会得到客户端代码和服务器代码的混合，因此将它们分开是一种良好的实践。当你使用 Express 框架时，它会为你创建这些。

1.  在我们的项目文件夹中创建 `hello.js`，我们的 node.js 应用程序：

    ```php
    var http = require('http');
    var static = require('node-static');
    var fileServer = new static.Server('./public');
       http.createServer(function (req, res) {
         fileServer.serve(req,res);
       }).listen(8080, 'localhost');
       console.log('Server running at http://localhost:8080');
    ```

1.  接下来，在子文件夹 `public` 中，我们创建 `hello.html`：

    ```php
    <!DOCTYPE html>
    <html>
    <head>
    <meta charset="UTF-8" />
    <title>Hello world document</title>
    <link href="./styles/hello.css" rel="stylesheet">
    </head>
    <body>
    <h1>Hello, World</h1>
    </body>
    </html>
    ```

1.  我们可以使用 `hello.css` 创建背景，如下所示：

    ```php
    body {
      background-color:#FFDEAD;
    }
    h1
    {
      color:teal;
      margin-left:30px;
    }
    .bigbutton {
      height:40px;
      color: white;
      background-color:teal;
      margin-left:150px;
      margin-top:50px;
      padding:15 15 25 15;
      font-size:18px;
    }
    ```

因此，如果我们现在访问 `http://localhost:8080/hello.html`，我们将看到我们至今仍然熟悉的 `Hello World` 消息，并带有一些基本的样式，这证明了我们的文件服务器也传递了 CSS 文件。

1.  现在，我们将更进一步，实际上在我们的 html 文件（`hellobutton.html`，仅 body 部分）中添加 JavaScript。我们将重用之前的 CSS 文件，创建一个略有不同的 HTML 文件，并添加一个 JavaScript 文件。我假设你 somewhere 有一个 jQuery 的副本。

    ```php
    <body>
    <div id="content">
    <button type="button" id="hellobutton" class="bigbutton">Click here for a message</button>
    </div>
    <script src="img/jquery.js"></script>
    <script src="img/hellobutton.js"></script>
    </body>
    ```

1.  要添加按钮，让我们创建 `hellobutton.js`：

    ```php
    $(document).ready(function(){
    $("#content").on("click", "#hellobutton", function(data){
    $('#hellobutton').text("Hello World");
      } );
    });
    ```

因此，当我们访问 `http://localhost:8080/hellobutton.html` 时，我们会看到一个带有按钮的网页；当我们点击它时，其文本变为 `Hello World`。这意味着我们的客户端 jQuery 和 JavaScript 正在正常工作。

在 `public` 文件夹中，创建一个文件 `index.html`：

```php
<!DOCTYPE html >
<html>
<body>
<h1>It works!</h1>
</body>
</html>
```

如果我们访问`http://localhost:8080`，我们会看到*It Works !* 就像我们点击 Apache Web 服务器的文档根目录一样。这是因为我们的`node-static`模块已经将这个文件配置为默认。

但是，还有一些事情我们没有习惯的方式工作。如果我们输入`hellobutton`而不是`hellobutton.html`，什么也不会发生，因为我们没有编程我们的 Web 服务器去寻找`hellobutton.something`。甚至不要想处理`hello.html?key=value`。

另一方面，如果你在`./public`中放置一个图片文件，例如`baywatchstation.jpg`，并输入`http://localhost:8080/baywatchstation.jpg`，你将在你的浏览器中看到这张图片。所有这些操作只需要很少的代码和两个酷炫的 node.js 模块。

### 两个（JavaScript）城市的传说

我们已经到达了一个重要的阶段：我们有两个不同的 JavaScript 文件，它们都位于我们的服务器上，但一个是通过 node.js 解释的，另一个则是由 node.js 提供并由浏览器解释的，换句话说，是客户端。

尝试这个：`http://localhost:8080/js/hellobutton.js`。你将在浏览器中看到你的 JavaScript 文件的代码。现在插入`alert("Here's Johnny!");`并将其放在`<script>`标签中，保存并刷新浏览器。`Johnny`会出现，然后 JavaScript 什么也不做，不会给你任何错误信息。

因为我们已经将`public`（好吧，是`node-static`）配置为我们的迷你 Web 服务器的文档根目录，所以我们甚至无法访问`hello.js`，这使我们免于更大的困惑。我确信到现在你已经理解了 JavaScript 文件和 JavaScript 文件之间的区别。这就是为什么一些开发者养成了使用不同扩展名（例如`.njs`用于服务器端 JS 文件）的习惯。我相信，像我们开始做的那样，将不同类型的文件放在不同的文件夹中要清晰得多。

但是，到目前为止，在这么短的时间内，仅仅几行代码，我们就能够用 node.js 的方式做到我们在这本书中讨论的几乎所有事情：我们可以处理 HTML、CSS、JavaScript 和 jQuery。我们放弃了 PHP，并用 MongoDB 替换了 MySQL。这让我们只剩下后者和 Ajax，然后我们就可以用 node.js 的方式重写我们的书了。

## node.js 和 MongoDB

在第十一章中，我们介绍了 MongoDB，一个文档数据库，并学习了如何从命令行以及 PHP 程序内部访问它。在 node.js 中这样做甚至更容易。首先，让我们不要忘记在一个单独的终端窗口中启动 MongoDB 服务器：

```php
mongodb

```

接下来，我们当然需要一个 node.js 模块，`mongodb`：

```php
npm install mongodb

```

下面是一个简单的程序，它连接到 MongoDB 服务器，具体来说是`california`数据库，并在`people`集合中查找一个文档。

```php
var MongoClient = require('mongodb').MongoClient;
   MongoClient.connect('mongodb://localhost:27017/california',
   function(err, db) {
       console.log('Connected to MongoDB!');
       var collection = db.collection('people');
       collection.findOne({name: 'Adams'}, function(err, doc) {
               console.log(doc.first + ' - ' + doc.name);
               db.close();
      });
    });
```

### Déjà vu … 再次

当我刚开始使用 node.js 时，我有一种似曾相识的感觉。用 Grace Jones 基于 Astor Piazolla 的《Libertango》改编的歌曲来比喻：*奇怪，我以前见过这种情况*。

使用 node.js，你只需添加你需要的东西，所以它默认不包括厨房用具。这只能意味着你在性能方面会从中受益。

我是一个 UNIX 用户，但这个故事要追溯到当 Linus 还没有将其重写为 Linux，Mac OS X 还不存在的时候。内存和磁盘空间都很昂贵，UNIX 也是如此，因为制造商必须支付版税。

我曾经是 PC UNIX 产品的自豪的产品经理，我们最酷的价值增值之一是一个名为**kconfig**的工具，它允许人们自定义 UNIX 内核中的内容，使其只包含所需的内容。这就是 node.js 让我想起来的。而且它是用 C 语言编写的，就像 UNIX 一样。*似曾相识*。

虽然当时很酷，但今天就不会那么酷了，因为 UNIX 已经添加了太多东西：它将无法管理。

如果我们想用纯 node.js 来模拟 Apache 网络服务器能处理的所有功能，也是同样的道理。只需看看 PHP 的`phpinfo()`函数的输出。它显示了所有加载到 Apache 中的模块。如果我们想只用 node.js 支持所有这些，我们需要太多的模块，最终代码将难以阅读。电影《莫扎特传》中的场景浮现在脑海中，皇帝的随从们对莫扎特的《费加罗的婚礼》达成一致意见（我不这么认为）：音符太多！

# Express

使用 Express 框架是完成工作而无需过多笔记的好方法。在[expressjs.com](http://expressjs.com)网站上，它被称为*最小和灵活的 node.js 网络应用程序框架，提供了一套强大的功能，用于构建网络应用程序*。

描述 Express 能为你做什么的最好方法可能没有。它是最小的，因此框架本身的开销很小。它是灵活的，因此你可以添加你需要的东西。因为它提供了一套强大的功能，这意味着你不必自己创建它们，而且它们已经由不断增长的社区进行了测试。

## 安装 Express

当然，Express 也是一个 node 模块，所以我们像安装模块一样安装它。在撰写本文时，我们使用了**Express 4**。在你的应用程序项目目录中，输入：

```php
npm install express

```

或者你也可以使用：

```php
npm install —save express

```

如果你指定了`—save`选项，`npm`将会更新`package.json`文件。你会注意到在`node_modules`内部会创建一个名为`express`的文件夹，并在其中还有一个 node-modules 的集合。这些都是被称为**中间件**的例子。

在接下来的几个例子中，我们假设`app.js`是你的 node.js 应用程序的名称，`app`是你在该文件中使用的变量，用于 Express 的实例。这样做是为了简洁。最好使用一个与你的项目名称匹配的字符串。

## 我们的第一个 Express 应用程序

当然，我们还将进行更多的 `Hello, World` 示例。这是我们第一个 Express 应用程序：

```php
var express = require('express');
var app = express();
app.set('port', process.env.PORT || 3000);
app.get('/', function (req, res) {
  res.send('<h1>Hello World!</h1>');
});

app.listen(app.get('port'),  function () {
console.log('Express started on http://localhost:' +
  app.get('port') + '; press Ctrl-C to terminate.' );

});
```

嗯，与我们的第二个 node.js 示例相比，行数差不多。但它看起来干净得多，而且为我们做了更多。你不再需要显式包含 HTTP 模块，你也不再需要指定要发送哪个头，而且当你指定不同的 URL 时，你不会得到 `Hello, World`，而是一个合理的错误消息。我们使用 `app.set` 和 `app.get` 来设置端口。当环境变量 `PORT` 被设置时，端口将被设置为它的值。

包含 `app.get` 的另一行告诉我们，当服务器接收到 `GET` 模式下的 URL 时，我们希望发生什么。就像在 node.js 中一样，有一个以 `request` 和 `respond` 对象为参数的函数。在 `express` 中，它们被扩展了；你可以用它们做更多创造性的事情，因为你可以使用更多方法。

例如，你可以访问 `req.body`，它将包含一个对象，包含使用 `POST` 方法在表单中发送的所有值（使用 `app.post`）。

## 一个使用中间件的示例

我们现在将使用 Express 重写 `hello button` 示例。`public` 目录中的所有静态资源都可以保持不变。唯一的变化是在节点 `app` 本身：

```php
var express = require('express');
var path = require('path');
var app = express();

app.set('port', process.env.PORT || 3000);

var options = {
  dotfiles: 'ignore',
  extensions: ['htm', 'html'],
  index: false
};

app.use(express.static(path.join(__dirname, 'public') ,
options    ));

app.listen(app.get('port'),  function () {
console.log('Hello express started on http://localhost:' +
app.get('port') + '; press Ctrl-C to terminate.' );
});
```

这段代码使用了与 `express` 一起提供的所谓中间件（`static`）。第三方提供了更多。在前面提到的 `req.body` 中，有可用的中间件来解析表单数据（`body-parse`）。你也可以编写自己的中间件。在其最简单的形式中，它是一个以 `req` 和 `res` 为参数的函数：

```php
app.use (function(req,res) {
   res.status(404);
   res.send(" Oops, you have tried to reach a page that does not exist");
});
```

这是你的最小 404 处理器，当人们输入错误的 URL 时，你可以给他们一些有意义的内容在屏幕上阅读。你将这个放在你的 app.js 文件中，在代表成功场景的代码之后。

# 模板和 handlebars.js

还有另一个 `Hello, world` 示例要展示！在整本书中，我们大部分时间都在使用 PHP。我们用它来动态生成网页，或者网页的一部分。所以 PHP 代码，通常嵌入到以 `.php` 为扩展名的文件中的 HTML 代码中，在服务器上执行，浏览器渲染的是纯 HTML。你也学习了如何从单独的 PHP 文件或客户端的 JavaScript 中生成 HTML，使用来自服务器的数据，然后将它注入到网页的一部分（Ajax）中。

通过 `<script>` 标签和将 PHP 代码放在 `<?php` 和 `?>` 之间，将 PHP 和 HTML 以及甚至一小块客户端 JavaScript 结合在一个文件中成为可能。这就是为什么他们有时把 PHP 称为 *模板* 语言。

现在想象一个全部由 JavaScript 构成的生态系统。是的，我们仍然可以将客户端 JavaScript 代码放在 `<script>` 标签之间，但服务器端的 JavaScript 代码怎么办？没有 `<?javascript ?>` 这样的东西，因为这不是 node.js 的工作方式。

Node.js 和 Express 支持多种模板语言，这些语言允许你分离布局和内容，并由模板系统完成获取内容并将其注入 HTML 的工作。由于我们不再想学习另一种语言，我们决定选择**handlebars.js**，因为它使用你已经在 12 章前学过的纯 HTML 来定义布局。Express 的默认模板语言似乎是**Jade**，它使用自己的、尽管更紧凑的、因为没有标签的格式。使用 handlebars.js 的另一个优点是它也支持客户端模板。

我们以一个如何使用 handlebars.js 的示例来结束本章。

本章中的所有示例都是 node.js 示例，我们需要使用模块。要在 node 和 Express 中使用 handlebars，有几个模块可供选择。我喜欢名字容易记住的**express-handlebars**。如果你在网上搜索 handlebars.js，你会找到用于客户端模板的库。

使用以下命令为 Express 获取`handlebars`模块：

```php
npm install express-handlebars

```

## 创建布局

在包含`public`的项目文件夹内，创建一个名为`views`的文件夹，并在其中创建一个名为`layouts`的子目录。将`public`中可能有的其他静态内容复制到`views`中。在`layouts`子文件夹内，创建一个名为`main.handlebars`的文件。这是你的默认布局。把它想象成几乎所有页面的通用布局：

```php
<!doctype html>
    <html>
    <head>
<title>Handlebars demo</title> </head>
<link href="./styles/hello.css" rel="stylesheet">
<body>
{{{body}}}
    </body>
    </html>
```

注意到`{{{body}}}`部分。这个标记将被 HTML 替换。在`views`文件夹中创建一个名为`hello.handlebars`的文件，内容如下。这将是一个（许多）HTML 示例之一，它将被替换为：

```php
<h1>Hello, World</h1>
```

# 我们最后的 Hello, World 示例

现在在项目文件夹中创建一个名为`lasthello.js`的文件。为了方便，我们在之前的 Express 示例中添加了相关代码。之前所有工作正常，但如果你输入`http://localhost:3000/`，你会看到一个页面，其中包含布局文件中的布局，并且`{{{body}}}`被替换为（你猜对了）：

```php
var express = require('express');
var path = require('path');
var app = express();
var handlebars = require('express-handlebars') .create({ defaultLayout:'main' });
    app.engine('handlebars', handlebars.engine);
    app.set('view engine', 'handlebars');
app.set('port', process.env.PORT || 3000);

var options = {
  dotfiles: 'ignore',
  etag: false,
  extensions: ['htm', 'html'],
  index: false
};

app.use(express.static(path.join(__dirname, 'public') , options    ));
app.get('/', function(req, res)
{
res.render('hello');   // this is the important part
});
app.listen(app.get('port'),  function () {
  console.log('Hello express started on http://localhost:' +
  app.get('port') + '; press Ctrl-C to terminate.' );

});
```

# 摘要

在本章的最后，我们概述了 node.js 和 Express。得益于 node.js，你可以在客户端和服务器端全面使用 JavaScript。你甚至可以用几行代码就编写自己的网络服务器。因为你只包含真正需要的东西，所以你可以通过这种*前卫*的网页开发方式获得更好的性能。

当你在代码中将网络服务器和服务器应用程序结合起来时，可能需要编写的代码比你预期的要多。这时 Express 就派上用场了：一个轻量级的框架，它生成的代码既紧凑又健壮。

总结来说，我们通过介绍 handlebars.js 触及了模板冰山的一角。这是一种更好的方法来分离布局和动态内容，让框架将它们结合起来，以便浏览器可以将其渲染为视图。为此，我们通过编写 HTML 布局来结束本章。

这让我想起了安娜·拉塞尔对瓦格纳的*尼伯龙根的指环*的演绎，她用 20 分钟完成了（通常《指环》需要 16 小时），结论是故事以它开始的方式结束。它有点像这样：

> *这里有莱茵河，河里有莱茵河女神，河底有……金子……*

因此，经过 14 章，我们了解了网络开发的许多方面，我们回到了一切开始的地方：HTML。我希望你们阅读它的时候和我写作它的时候一样享受。
