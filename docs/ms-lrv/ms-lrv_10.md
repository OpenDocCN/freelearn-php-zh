# 第十章。使用 Elixir 构建、编译和测试

本章将涵盖以下主题：

+   安装 Node.js，Gulp 和 Elixir

+   运行 Elixir

+   使用 Elixir 合并 CSS 和 JavaScript 文件

+   设置通知

+   使用 Elixir 运行测试

+   扩展 Elixir

# 自动化 Laravel

在整本书中，已经构建了示例应用程序的许多部分。我们讨论了创建应用程序涉及的步骤。然而，关于帮助搭建、样板模板和为 CRUD 应用程序构建 RESTful API 的工具还有更多信息可用。直到最近，关于自动化开发过程和部署过程的一些部分并没有太多的资料。

在 PHP 领域，近年来出现了一个新的领域，即持续集成和构建工具的概念。持续集成和持续交付的流行使开发团队能够不断发布许多小的改进，每天多次发布他们的应用程序。在本章中，您将了解到 Laravel 具有一套新的工具集，可以使团队快速轻松地部署他们的软件版本，并自动构建和组合软件的许多组件。

持续集成和持续交付在开发过程中引起了相当大的变革，大大改变了软件构建的方式。然而，不久之前，标准的部署过程只涉及将代码放在服务器上。大多数早期采用 PHP 的人只是需要添加功能，比如*论坛*或*联系我们*表单的网页设计师。由于他们大多不是程序员，因此网页设计和图形设计中使用的大多数实践也被用于 PHP 部署。这些实践通常涉及使用诸如 FileZilla 之类的应用程序，将文件从左侧面板（用户的计算机）拖放到右侧（服务器的目录）。对于更有经验的人来说，使用终端仿真器（如 PuTTY）执行当时晦涩的 UNIX 命令。

使用不安全的文件传输端口 21，并且所有内容都未经压缩，只是简单地复制到服务器上。通常，所有文件都会被覆盖，而且部署大型网站的过程通常需要将近一个小时，因为有很多图片和文件。

最终，源代码控制系统变得普遍。在最近几年，SVN 和 Git 已成为大多数软件项目的行业标准。这些工具允许直接从代码仓库部署。

最近，composer 的到来为简单地将整个软件包包含到软件应用程序中添加功能创造了一种简单的方式。开发人员只需向配置文件添加一行代码即可轻松实现！

自动化开发和部署过程可能涉及许多步骤，以下是其中一些。

## 部署

以下是部署过程的一些功能：

+   复制与生产环境相关的某些配置设置

+   处理或编译使用快捷语法或预处理器编写的任何**层叠样式表**（**CSS**）或 JavaScript 文件

+   将各种资产（源代码或图像）复制到镜像、集群服务器或内容交付网络中

+   修改某些文件或目录的读/写/执行权限和/或所有权

+   将多个文件合并为一个文件，以减少执行多个 HTTP 调用所需的开销

+   减少文件中的无用空格和注释（缩小和/或混淆）以减小文件大小

+   将服务器上的现有文件与本地环境中的文件进行比较，以确定是否覆盖它们

+   对源代码进行标记和/或版本控制，以便可能进行代码回滚

## 开发或部署

以下是开发或部署过程的一些功能：

+   验证代码是否通过了编写的所有单元、功能和验收测试，以确保其质量

+   运行执行各种操作的脚本

+   执行任何迁移、种子播种或对数据库表的其他修改

+   从托管的源代码控制系统（如 GitHub）获取源代码控制

很明显，现代开发非常复杂。软件开发的更加困难的方面是在开发过程中不断重新创建生产或最终环境。

# 朝着自动化的方向

诸如文件监视器之类的工具可以在每次文件被修改时运行脚本或执行操作。此外，诸如 PHPStorm 之类的 IDE 将识别文件扩展名，并提供监视文件更改并允许开发人员执行某些操作的选项。虽然这种方法是可以接受的，但它并不是非常便携，每个开发人员都必须创建和共享一个包含 IDE 或文本编辑器中各种监视器的配置文件。这会产生依赖性，依赖于整个团队的一个单一 IDE。

此外，还可以创建其他方法，例如 Bash-shell 脚本，以在特定时间间隔运行。但是，使用这些脚本需要 UNIX-shell 编码知识。正如先前所示，像 artisan 这样的工具有助于自动化许多手动任务。但是，大多数默认的 artisan 命令是设计为手动执行的。

幸运的是，出现了两个使用 Node.js JavaScript 平台的工具：*Grunt*和*gulp*。Grunt 和 gulp 都取得了相当大的成功，但 gulp 最近变得更加流行。然而，对于可能不熟悉 JavaScript 语法的 PHP 开发人员来说，学习如何快速编写 gulp 任务并不容易。

考虑以下示例代码，摘自 gulp 的文档：

```php
gulp.task('scripts', ['clean'], function() {
  // Minify and copy all JavaScript (except vendor scripts)
  // with sourcemaps all the way down
  return gulp.src(paths.scripts)
    .pipe(sourcemaps.init())
      .pipe(coffee())
      .pipe(uglify())
      .pipe(concat('all.min.js'))
    .pipe(sourcemaps.write())
    .pipe(gulp.dest('build/js'));
});
```

# 从 Gulp 到 Elixir

幸运的是，Laravel 社区一直秉承着前瞻性思维，专注于减少复杂性。一个名为**Elixir**的官方社区工具已经出现，以便于使用 gulp。Gulp 是建立在 Node.js 之上的，而 Elixir 是建立在 gulp 之上的，创建了一个包装器：

![从 Gulp 到 Elixir](img/B04559_10_01.jpg)

### 注意

Laravel Elixir 不应与同名的动态功能语言混淆。另一个 Elixir 使用 Erlang 虚拟机，而 Laravel Elixir 使用 gulp 和 Node.js

# 入门

第一步是在开发计算机上安装 Node.js（如果尚未安装）。

### 注意

可以在以下网址找到说明：

[`nodejs.org`](https://nodejs.org)

## 安装 Node.js

对于像 Ubuntu 这样的基于 Debian 的操作系统，安装 Node.js 可能就像使用`apt`软件包管理器一样简单。从命令行使用以下命令：

```php
**$ sudo apt-get install -y nodejs**

```

请参考 Node.js 网站（[`nodejs.org`](https://nodejs.org)）上的正确操作系统的安装说明。

## 安装 Node.js 包管理器

下一步涉及安装 gulp，Elixir 将使用它来运行其任务。对于这一步，需要**Node.js 包管理器**（**npm**）。如果尚未安装`npm`，则应使用`apt`软件包安装程序。以下命令将用于安装`npm`：

```php
**$ sudo apt-get install npm**

```

npm 使用一个`json`文件来管理项目的依赖关系：`package.json`。该文件位于 Laravel 项目目录的根目录中，格式如下：

```php
{
  "devDependencies": {
    "gulp": "³.8.8",
    "laravel-elixir": "*"
  }
}
```

安装 gulp 和 Laravel Elixir 作为依赖项。

## 安装 Gulp

以下命令用于安装`gulp`：

```php
**$ sud onpm install --global gulp**

```

## 安装 Elixir

一旦安装了 Node.js、npm 和 gulp，下一步是安装 Laravel Elixir。通过运行`npm` install 而不带任何参数，`npm`将读取其配置文件并安装 Laravel Elixir：

```php
**$ npm install**

```

# 运行 Elixir

默认情况下，Laravel 包含一个`gulpfile.js`文件，该文件由 gulp 用于运行其任务。该文件包含一个`require`方法，用于包含运行任务所需的一切：

```php
var elixir = require('laravel-elixir');

/*
 |----------------------------------------------------------------
 | Elixir Asset Management
 |----------------------------------------------------------------
 |
 | Elixir provides a clean, fluent API for defining some basic gulp tasks
 | for your Laravel application. By default, we are compiling the Sass
 | file for our application, as well as publishing vendor resources.
 |
 */

elixir(function(mix) {
    mix.less('app.less');
});
```

第一个混合示例显示为：`app.less`。要运行 gulp，只需在命令行中输入`gulp`，如下所示：

```php
**$  gulp**

```

输出如下所示：

```php
**[21:23:38] Using gulpfile /var/www/laravel.example/gulpfile.js**
**[21:23:38] Starting 'default'...**
**[21:23:38] Starting 'less'...**
**[21:23:38] Running Less: resources/assets/less/app.less**
**[21:23:41] Finished 'default' after 2.35 s**
**[21:23:43] gulp-notify: [Laravel Elixir] Less Compiled!**
**[21:23:43] Finished 'less' after 4.27 s**

```

第一行表示已加载 gulp 文件。接下来的行显示每个任务的运行情况。`less`任务处理层叠样式表预处理器`Less`。

# 设置通知

如果您的开发环境是 Vagrant Box，则安装`vagrant-notify`将允许 Laravel Elixir 直接与主机交互，并在操作系统中直接显示本机消息。要安装它，应从主机操作系统运行以下命令：

```php
**$ vagrant plugin install vagrant-notify**

```

以下是通知的截图，显示 PHPUnit 测试失败了：

![设置通知](img/B04559_10_02.jpg)

安装说明取决于每个操作系统。

### 注意

有关更多信息，请访问[`github.com/fgrehm/vagrant-notify`](https://github.com/fgrehm/vagrant-notify)。

# 使用 Elixir 合并 CSS 和 JavaScript 文件

可能，部署过程中最重要的一步是合并和缩小 CSS 和 JavaScript 文件。缩小和合并五个 JavaScript 文件和三个 CSS 文件意味着不再有八个 HTTP 请求，而只有一个。此外，通过去除空格、换行符、注释和其他技术（例如缩短变量名）来缩小文件大小，文件大小将减少到原始大小的一小部分。尽管有这些优势，仍然有许多网站继续使用未缩小和未合并的 CSS 和 JavaScript 文件。

Elixir 提供了一种简单的方法来轻松合并和缩小文件。以下代码说明了这个示例：

```php
elixir(function(mix) {
    mix.scripts().styles();
});
```

`scripts()`和`styles()`两种方法将所有 JavaScript 和 CSS 文件合并为单个文件，分别为`all.js`和`all.css`。默认情况下，这两个函数期望文件位于`/resources/assets/js`和`/resources/assets/css`。

当 gulp 命令完成时，输出将如下所示：

```php
**[00:36:20] Using gulpfile /var/www/laravel.example/gulpfile.js**
**[00:36:20] Starting 'default'...**
**[00:36:20] Starting 'scripts'...**
**[00:36:20] Merging: resources/assets/js/**/*.js**
**[00:36:20] Finished 'default' after 246 ms**
**[00:36:20] Finished 'scripts' after 280 ms**
**[00:36:20] Starting 'styles'...**
**[00:36:20] Merging: resources/assets/css/**/*.css**
**[00:36:21] Finished 'styles' after 191 ms**

```

请注意输出方便地说明了扫描了哪些目录。内容被合并，但没有被缩小。这是因为在开发过程中，在缩小文件上进行调试太困难。如果只有某个文件需要合并，则可以将文件名作为第一个参数传递给函数：

```php
mix.scripts('app.js');
```

如果要合并多个文件，则可以将文件名数组作为第一个参数传递给函数：

```php
mix.scripts(['app.js','lib.js']);
```

在生产环境中，希望有缩小的文件。要让 Elixir 缩小 CSS 和 JavaScript，只需在 gulp 命令中添加`--production`选项，如下所示：

```php
**$ gulp --production**

```

这将产生所需的缩小输出。默认输出目录位于：

```php
/public/js
/public/css
```

# 使用 Laravel Elixir 编译

Laravel Elixir 非常擅长执行通常需要学习脚本语言的例行任务。以下各节将演示 Elixir 可以执行的各种编译类型。

## 编译 Sass 和 Less

层叠样式表预处理器`Less`和`Sass`出现是为了增强 CSS 的功能。例如，它不包含任何变量。`Less`和`Sass`允许前端开发人员利用变量和其他熟悉的语法特性。以下代码是标准 CSS 的示例。DOM 元素`p`和`li`（分别表示段落和列表项），以及具有`post`类的任何元素将具有`font-family` Arial，sans-serif 作为回退，并且颜色为黑色：

```php
p, li, .post {
  font-family: Arial, sans-serif;
  color: #000;
}
```

接下来，使用`Sass` CSS 预处理器，将字体族和文本颜色替换为两个变量：`$text-font`和`$text-color`。这样在需要更改时可以轻松维护。而且，这些变量可以共享。代码如下：

```php
$text-font:    Arial, sans-serif;
$text-color: #000;

p, li, .post {
  font: 100% $text-font;
  color: $text-color;
}
h2 {
  font: 2em $text-font;
  color: $text-color;
}
```

`Less`预处理器使用`@`而不是`$`；因此，它的语法看起来更像是注释而不是`php`变量：

```php
@text-font:    Arial, sans-serif;
@text-color: #000;

p, li, .post {
  font: 100% @text-font;
  color: @text-color;
}
h2 {
  font: 2em @text-font;
  color: @text-color;
}
```

还需要执行一个额外的步骤，因为它不会被浏览器引擎解释。增加的步骤是将`Less`或`Sass`代码编译成真正的 CSS。这在开发阶段会增加额外的时间；因此，Elixir 通过自动化流程来帮助。

在之前的 Laravel Elixir 示例中，`less`函数只接受文件名`app.less`作为其唯一参数。现在，示例应该更清晰一些。此外，`less`可以接受一个将被编译的参数数组。

`less`方法在`/resources/assets/less`中搜索，默认情况下输出将放在`public/css/`中：

```php
elixir(function(mix) {
    mix.less([
        'style.less',
        'style-rtl.less'
    ]);
});
```

## 编译 CoffeeScript

CoffeeScript 是一种编译成 JavaScript 的编程语言。与 Less 和 Sass 一样，它的目标是简化或扩展它所编译的语言的功能。在 CoffeeScript 的情况下，它通过减少按键次数来简化 Javascript。在下面的 JavaScript 代码中，创建了两个变量——一个数组和一个对象：

```php
var available, list, room;

room = 14;

available = true;

list = [101,102,311,421];

room = { 
  id: 1,
  number: 102,
  status: "available"
}
```

在下面的 CoffeeScript 代码中，语法非常相似，但不需要分号，也不需要`var`来创建变量。此外，缩进用于定义对象的属性。代码如下：

```php
room = 14

available = true 

list = [101,102,311,421]

room = 
  id: 1
  number: 102
  status: "available"
```

在这个 CoffeeScript 示例中，字符较少；然而，对于程序员来说，减少按键次数可以帮助提高速度和效率。要将 coffee 编译器添加到 Elixir 中，只需使用`coffee`函数，如下面的代码所示：

```php
elixir(function(mix) {
    mix.coffee([
        'app.coffee'
    ]);
});
```

## 编译器命令摘要

下表显示了预处理器、语言、函数以及每个函数期望源文件的位置。右侧的最后一列显示了结果合并文件的目录和/或名称。

| processor | Language | function | Source directory | Default Output Location |
| --- | --- | --- | --- | --- |
| Less | CSS | `less()` | `/resources/assets/less/file(s).less` | `/public/css/file(s).css` |
| Sass | CSS | `sass()` | `/resources/assets/sass/file(s).scss` | `/public/css/file(s).css` |
| N/A | CSS | `styles()` | `/resources/assets/css/` | `/public/css/all.css` |
| N/A | JavaScript | `scripts()` | `/resources/assets/js/` | `/public/js/all.js` |
| CoffeeScript | JavaScript | `coffee()` | `/resources/assets/coffee/` | `/public/js/app.js` |

> 使用不同的名称保存

可选地，每个方法都可以接受第二个参数，该参数将覆盖默认位置。要使用不同的目录（在本例中是一个名为`app`的目录），只需将该目录作为第二个参数添加：

```php
mix.scripts(null,'public/app/js').styles(null,'public/app/css');
```

在这个例子中，文件将保存在`public/app/js`和`public/app/css`。

## 把所有东西放在一起

最后，让我们把所有东西放在一起得出一个有趣的结论。由于 CoffeeScript 脚本和`less`和`sass`文件不是合并而是直接复制到目标中，我们首先将 CoffeeScript、`less`和`sass`文件保存到 Elixir 期望 JavaScript 和 CSS 文件的目录中。然后，我们指示 Elixir 将所有 JavaScript 和 CSS 文件合并和压缩成两个合并和压缩的文件。代码如下：

```php
elixir(function(mix) {
    mix.coffee(null,'resources/assets/js')
        .sass(null,'resources/assets/css')
        .less(null,'resources/assets/css')
        .scripts()
        .styles();
});
```

### 提示

非常重要的一点是，Elixir 会覆盖文件而不验证文件是否存在，因此需要为每个文件选择一个唯一的名称。命令完成后，`all.js`和`all.css`将合并和压缩在`public/js`和`public/css`目录中。

# 使用 Elixir 运行测试

除了编译和发送通知之外，Elixir 还可以用于自动化测试的启动。接下来的部分将讨论 Elixir 如何用于 PHPSpec 和 PHPUnit。

## PHPSpec

第一步是运行 PHPSpec 测试以自动化代码测试。通过将`phpSpec()`添加到我们的`gulpfile.js`中，PHPSpec 测试将运行：

```php
elixir(function(mix) {
    mix.less('app.less').phpSpec();
});
```

以下截图显示了输出。PHPSpec 输出被保留，因此测试输出非常有用：

![PHPSpec](img/B04559_10_03.jpg)

当 PHPSpec 测试失败时，结果很容易阅读：

![PHPSpec](img/B04559_10_04.jpg)

Laravel Elixir 输出的截图

在这个例子中，phpspec 在**it creates a reservation test**一行遇到了错误，如前面的截图所示。

## PHPUnit

同样，我们可以通过将`phpUnit`添加到任务列表中来将 PHPUnit 添加到我们的测试套件中，如下所示：

```php
elixir(function(mix) {
    mix.less('app.less').phpSpec().phpUnit();
});
```

## 创建自定义任务

Elixir 使我们能够创建自定义任务来几乎做任何事情。我们可以编写一个扫描控制器注释的自定义任务的一个例子。所有自定义任务都需要`gulp`和`laravel-elixir`。重要的是要记住所使用的编程语言是 JavaScript，因此语法可能或可能不熟悉，但很容易快速学习。如果命令将从命令行界面执行，那么我们还将导入 gulp-shell。代码如下：

```php
var gulp = require('gulp');
var elixir = require('laravel-elixir');
var shell = require('gulp-shell');

/*
 |----------------------------------------------------------------
 | Route Annotation Scanner
 |----------------------------------------------------------------
 |
 | We'll run route:scan Artisan to scan for changed files.
 | Output is written to storage/framework/routes.scanned.php
 | 
*/

 elixir.extend('routeScanning', function() {
                 gulp.task('routeScanning', function() {
                         return gulp.src('').
      pipe(shell('php artisan route:scan'));
                 });

     return this.queueTask('routeScanning');
 });
```

在这段代码中，我们首先扩展 Elixir 并给方法一个名称，例如`routeScanning`。然后，定义了一个 gulp 任务，`task`方法的第一个参数是命令的名称。第二个命令是包含将被执行和返回的代码的闭包。

最后，通过将命令的名称传递给`queueTask`方法，将任务排队执行。

将此脚本添加到我们的链中，如下所示：

```php
elixir(function(mix) {
    mix.routeScanning();
});
```

输出将如下所示：

```php
**$ gulp**
**[23:24:19] Using gulpfile /var/www/laravel.example/gulpfile.js**
**[23:24:19] Starting 'default'...**
**[23:24:19] Starting 'routeScanning'...**
**[23:24:19] Finished 'default' after 12 ms**
**[23:24:20] Finished 'routeScanning' after 1 s**

```

由于`pipe`函数允许命令链接，很容易添加一个通知，以警报通知系统，如下所示：

```php
var gulp = require('gulp');
var elixir = require('laravel-elixir');
var shell = require('gulp-shell');
var Notification = require('./commands/Notification');

 elixir.extend('routeScanning', function() {
                 gulp.task('routeScanning', function() {
                         return gulp.src('').
                             pipe(shell('php artisan route:scan')).
                             pipe(new Notification().message('Annotations scanned.'));
                 });
     return this.queueTask('routeScanning');

 });
```

在这里，`Notification`类被引入，并创建了一个新的通知，以将消息`Annotations scanned.`发送到通知系统。

运行代码会产生以下输出。请注意，已添加了`gulp-notify`：

```php
**$ gulp**
**[23:46:59] Using gulpfile /var/www/laravel.example/gulpfile.js**
**[23:46:59] Starting 'default'...**
**[23:46:59] Starting 'routeScanning'...**
**[23:46:59] Finished 'default' after 38 ms**
**PHP Warning:  Module 'xdebug' already loaded in Unknown on line 0**
**Routes scanned!**
**[23:47:00] gulp-notify: [Laravel Elixir] Annotations scanned**
**[23:47:00] Finished 'routeScanning' after 1.36 s**

```

# 设置文件监视器

显然，每次我们想要编译层叠样式表或扫描注释时运行 gulp 是很繁琐的。幸运的是，Elixir 内置了一个监视机制。要调用它，只需运行以下命令：

```php
**$ gulp watch**

```

这将允许将任务自动运行到`gulpfile.js`链中的任何任务在发生某些更改时。启用此功能的必要代码在注释任务中如下：

```php
 this.registerWatcher("routeScanning", "app/Http/Controllers/**/*.php");
```

上面的代码注册了一个监视器。第一个参数是`routeScanning`任务。第二个命令是将被监视以进行修改的目录模式。

由于我们知道路由注释将在控制器内部，我们可以设置路径仅在`app/Http/Controllers/`目录内查找。正则表达式样式语法将匹配位于控制器下的任何一个目录中具有`php`扩展名的文件。

现在，每当修改与模式匹配的文件时，`routeScanning`任务以及任何其他监视匹配相同模式的文件的任务都将被执行。

# 额外的 Laravel Elixir 任务

npm 网站提供了超过 75 个任务，涉及测试、JavaScript、CSS 等。`npm`网站位于[`npmjs.com`](http://npmjs.com)。

![额外的 Laravel Elixir 任务](img/B04559_10_05.jpg)

npm 网站的截图包含了许多有用的 Laravel Elixir 任务

# 总结

在本章中，您了解了 Elixir 不断增长的任务列表如何帮助全栈开发人员以及开发团队。一些任务与前端开发相关，例如编译、合并和压缩 CSS 和 JavaScript。其他任务与后端开发相关，例如行为驱动开发。将这些任务集成到日常开发工作流程中，将使整个团队能够理解在持续集成服务器中执行的步骤，其中 Elixir 将执行其任务，例如测试和编译，以准备将文件从开发转换为生产。

由于 Elixir 是建立在 gulp 之上的，随着 gulp 和 Elixir 社区的持续增长和新的贡献者继续为 Elixir 做出贡献，Elixir 的未来将继续丰富。
