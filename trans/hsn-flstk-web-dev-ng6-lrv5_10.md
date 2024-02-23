# 使用 Bootstrap 4 和 NgBootstrap 的前端视图

在本章中，我们将看看如何使用 Angular CLI 的新`add`功能在运行的 Angular 应用程序中包含 Bootstrap 框架。

Bootstrap 框架是最重要的 UI 框架之一，结合 Angular 指令/组件，我们可以在 Angular 应用程序中拥有 Bootstrap 的全部功能。

我们还将看看如何将我们的 Angular 服务与组件连接起来，以及如何使用后端 API 将它们整合在一起。最后，我们将学习如何在后端 API 上配置**跨源资源共享**（**CORS**）以及如何在我们的 Angular 客户端应用程序中使用它。

在本章中，我们将涵盖以下主题：

+   安装 Bootstrap CSS 框架

+   使用 Bootstrap 编写 Angular 模板

+   如何在 Laravel 后端设置 CORS

+   将 Angular 服务与应用程序组件连接起来

+   处理 Angular 管道、表单和验证

# 准备基线代码

现在，我们需要准备我们的基线代码，这个过程与我们在上一章中执行的非常相似。让我们按照以下步骤进行：

1.  复制`chapter-9`文件夹中的所有内容。

1.  重命名`chapter-10`文件夹。

1.  删除`storage-db`文件夹。

1.  现在，让我们对`docker-compose.yml`文件进行一些更改，以便我们可以适应新的数据库和服务器容器。打开`docker-compose.yml`并用以下代码替换其内容：

```php
 version: "3.1"
 services:
     mysql:
       image: mysql:5.7
       container_name: chapter-10-mysql
       working_dir: /application
       volumes:
         - .:/application
         - ./storage-db:/var/lib/mysql
       environment:
         - MYSQL_ROOT_PASSWORD=123456
         - MYSQL_DATABASE=chapter-10
         - MYSQL_USER=chapter-10
         - MYSQL_PASSWORD=123456
       ports:
         - "8083:3306"
     webserver:
       image: nginx:alpine
       container_name: chapter-10-webserver
       working_dir: /application
       volumes:
         - .:/application
         -./phpdocker/nginx/nginx.conf:/etc/nginx/
           conf.d/default.conf
        ports:
          - "8081:80"
     php-fpm:
       build: phpdocker/php-fpm
       container_name: chapter-10-php-fpm
       working_dir: /application
       volumes:
         - ./Server:/application
         - ./phpdocker/php-fpm/php-ini-
           overrides.ini:/etc/php/7.2/fpm/conf.d/99-overrides.ini
```

请注意，我们更改了容器名称、数据库和 MySQL 用户：

+   `container_name: chapter-10-mysql`

+   `container_name: chapter-10-webserver`

+   ``container_name: chapter-10-php-fpm``

+   `MYSQL_DATABASE=chapter-10`

+   `MYSQL_USER=chapter-10`

1.  使用连接字符串更新`.env`文件：

```php
 DB_CONNECTION=mysql
 DB_HOST=mysql
 DB_PORT=3306
 DB_DATABASE=chapter-10
 DB_USERNAME=chapter-10
 DB_PASSWORD=123456
```

1.  将我们所做的更改添加到 Git 源代码控制中。打开您的终端窗口，输入以下命令：

```php
 git add .
 git commit -m "Initial commit chapter 10"
```

1.  现在，让我们使用以下命令启动我们的 Docker 容器：

```php
 docker-compose up -d
```

# 安装 Bootstrap CSS 框架

在本节中，我们将再次使用 Angular CLI 6 中可用的最新功能：`add`命令。使用这个命令，我们将向我们的应用程序添加 Bootstrap 4：

1.  在`chapter-10`的`Client`文件夹中，打开您的终端窗口并输入以下命令：

```php
 ng add @ng-bootstrap/schematics
```

1.  上一个命令将创建并更新以下文件：

```php
+ @ng-bootstrap/schematics@2.0.0-alpha.1
added 3 packages in 26.372s
Installed packages for tooling via npm.
UPDATE package.json (1589 bytes)
UPDATE src/app/app.module.ts (1516 bytes)
UPDATE angular.json (3706 bytes)
```

1.  在`package.json`文件中，我们将添加以下依赖项：

```php
     "@ng-bootstrap/schematics": "².0.0-alpha.1",
        "@ng-bootstrap/ng-bootstrap": "².0.0-alpha.0",
        "bootstrap": "⁴.0.0"
```

1.  在`src/app/app.module.ts`文件中，我们将添加以下行：

```php
     import { NgbModule } from  '@ng-bootstrap/ng-bootstrap';

        imports: [
                ...
                NgbModule.forRoot()
        ],
```

1.  在`angular.json`文件中，我们将添加以下行：

```php
     "styles": [
                "src/styles.scss",
                {
                        "input": "./node_modules/bootstrap/dist/css/bootstrap.css"
                }
        ],
```

在这里，我们可以看到 Angular CLI 的全部功能，因为所有这些更改都是自动完成的。

但是，我们可以看到`bootstrap.css`文件的使用方式使应用程序冻结，使其难以定制。

在下一节中，我们将探讨一种更灵活使用 Bootstrap 的方法。

# 移除 Bootstrap CSS 导入

首先，我们将删除通过`NgBootstrap`安装命令注入到我们的`angular.json`文件中的从 Bootstrap 编译的 CSS。

打开`angular.json`文件并删除`input`标签。只保留`styles`标签，如下所示：

```php
     "styles": [
                "src/styles.scss"
        ],
```

# 添加 Bootstrap SCSS 导入

现在，我们将使用`node_modules`文件夹中安装的文件作为我们主样式表`./Client/src/style.scss`的导入：

1.  打开`./Client/src/style.scss`并在文件顶部添加以下代码：

```php
/*! * Bootstrap v4.1.1 (https://getbootstrap.com/) * Copyright 2011-2018 The Bootstrap Authors * Copyright 2011-2018 Twitter, Inc. * Licensed under MIT (https://github.com/twbs/bootstrap/blob/master/LICENSE) */ @import "../node_modules/bootstrap/scss/functions"; @import "../scss/bootstrap/_variables.scss"; @import "../node_modules/bootstrap/scss/_variables.scss"; @import "../node_modules/bootstrap/scss/mixins"; @import "../node_modules/bootstrap/scss/root"; @import "../node_modules/bootstrap/scss/reboot"; @import "../node_modules/bootstrap/scss/type"; @import "../node_modules/bootstrap/scss/images"; @import "../node_modules/bootstrap/scss/code"; @import "../node_modules/bootstrap/scss/grid"; @import "../node_modules/bootstrap/scss/tables"; @import "../node_modules/bootstrap/scss/forms"; @import "../node_modules/bootstrap/scss/buttons"; @import "../node_modules/bootstrap/scss/transitions"; @import "../node_modules/bootstrap/scss/dropdown"; @import "../node_modules/bootstrap/scss/button-group"; @import "../node_modules/bootstrap/scss/input-group"; @import "../node_modules/bootstrap/scss/custom-forms"; @import "../node_modules/bootstrap/scss/nav"; @import "../node_modules/bootstrap/scss/navbar"; @import "../node_modules/bootstrap/scss/card"; @import "../node_modules/bootstrap/scss/breadcrumb"; @import "../node_modules/bootstrap/scss/pagination"; @import "../node_modules/bootstrap/scss/badge"; @import "../node_modules/bootstrap/scss/jumbotron"; @import "../node_modules/bootstrap/scss/alert"; @import "../node_modules/bootstrap/scss/progress"; @import "../node_modules/bootstrap/scss/media"; @import "../node_modules/bootstrap/scss/list-group"; @import "../node_modules/bootstrap/scss/close"; @import "../node_modules/bootstrap/scss/modal"; @import "../node_modules/bootstrap/scss/tooltip"; @import "../node_modules/bootstrap/scss/popover"; @import "../node_modules/bootstrap/scss/carousel"; @import "../node_modules/bootstrap/scss/utilities"; @import "../node_modules/bootstrap/scss/print";
```

请注意，我们保留了文件顶部的 Bootstrap 注释，只是为了在易于找到的地方记录 Bootstrap 版本。

1.  如果您愿意，您可以复制`node_modules/bootstrap/scss/bootstrap.scss`文件的内容，并只需调整导入路径为`../node_modules/bootstrap/scss`。

现在，我们的应用程序直接从`bootstrap/scss`文件夹编译 SCSS 代码。

这样做的一些优势包括：

+   我们可以根据应用程序使用的组件选择要导入的 SCSS 模块。

+   我们减少将不会使用的 SCSS 代码。

+   我们可以轻松地覆盖 Bootstrap 变量。

# 覆盖 Bootstrap 变量

在这一步中，我们将看到如何在我们的应用程序中覆盖`Boostrap`变量：

1.  在`Client`文件夹的根目录下创建一个名为`scss`的新文件夹。

1.  在`./Client/scss`文件夹中，添加一个名为`bootstrap`的新文件夹。

1.  在`./Client/scss/bootstrap`中，添加一个名为`_variable.scss`的新文件。

1.  从`node_modules/bootstrap/scss/_variables.scss`中复制内容，并粘贴到`./Client/scss/bootstrap/_variables.scss`中。

非常简单；恭喜！我们已经准备好覆盖 Bootstrap 变量。

最后一步是将新的`_variables.scss`文件导入到我们的主要`style.scss`文件中。

1.  打开`./Client/style.scss`文件，并用以下内容替换行`@import "../node_modules/bootstrap/scss/_variables.scss"`：

```php
 <pre>Error: ENOENT: no such file or directory, open '/Users/fernandomonteiro/_bitbucket/scss/bootstrap/_variables.scss'</pre>  
```

我们还有一个选项，即只使用我们将要覆盖的变量，而不使用关键字`Default`放置这个变量文件。这样，文件会变得更短，因为我们不会覆盖这样一个小项目中的所有变量。让我们看看我们如何做到这一点。

1.  假设我们只想覆盖所有组件的`border-radius`并删除`box-shadow`。我们只能使用这些变量，因此我们的`_variables.scss`文件将如下所示：

```php
     // Variables
        //
        // Removing border-radius and box-shadow from components

        $border-radius: 0;
        $border-radius-lg: 0;
        $border-radius-sm: 0;

        $box-shadow-sm: none;
        $box-shadow: none;
        $box-shadow-lg: none;
```

1.  为了使这些更改生效，我们需要对`./Client/style.scss`进行一些小调整，并在 Bootstrap`variables`文件之前添加新的变量文件，如下所示：

```php
/*!
 * Bootstrap v4.1.1 (https://getbootstrap.com/) * Copyright 2011-2018 The Bootstrap Authors * Copyright 2011-2018 Twitter, Inc. * Licensed under MIT (https://github.com/twbs/bootstrap/blob/master/LICENSE) */  @import  "../node_modules/bootstrap/scss/functions";
  @import  "../scss/bootstrap/_variables.scss";
  @import  "../node_modules/bootstrap/scss/_variables.scss";
```

# 使用 Bootstrap 编写 Angular 模板

此刻，我们的应用程序已经可以使用 Bootstrap CSS 进行可视化，这是我们在上一节中所做的。回想一下，在之前的章节中，我们已经向一些模板中添加了 HTML 标记。

它们都已经包含了 Bootstrap 类，我们已经可以在浏览器窗口中可视化到目前为止的内容。让我们来看看：

1.  在`./Client`文件夹中打开您的终端窗口，并键入以下命令：

```php
 npm start
```

1.  打开您的默认浏览器，转到`http://localhost:4200/`。

您将看到以下结果：

![](img/1bbae730-312d-4937-8aae-5934f803713c.png)

哇！现在，我们有了一个 Web 应用。您会注意到我们已经有一个完美运行的应用程序。

1.  让我们点击`bikes`链接，看看我们到目前为止有什么：

随意浏览应用程序的其余部分并检查其他页面。

然而，我们目前只有占位符，所以现在是学习如何在我们的模板中应用 Angular 模板语法的时候了。

# 向导航组件添加模板绑定

现在，让我们在模板中做一些更改，以便我们可以使用 Angular 语法：

1.  打开`./Client/src/app/layout/nav/nav.component.html`，并用以下代码替换其内容：

```php
<header> 
<nav class="navbar navbar-expand-md navbar-dark fixed-top bg-dark"> 
<a class="navbar-brand" [routerLink]="['/']" (click)="setTitle('Custom Bikes Garage')">Custom
Bikes Garage</a> 
<button class="navbar-toggler" type="button" data-toggle="collapse" data-
target="#navbarCollapse" aria-controls="navbarCollapse" aria-expanded="false" aria- label="Toggle
navigation"> 
<span class="navbar-toggler-icon"></span> 
</button> 
<div class="collapse navbar-collapse" id="navbarCollapse"> 
<ul class="navbar-nav ml-auto"> <li class="nav-item"> <a class="nav-link" [routerLink]="['/bikes']" routerLinkActive="active" (click)="setTitle('Bikes')">Bikes</a> 
</li> <li class="nav-item"> <a class="nav-link" [routerLink]="['/builders']" routerLinkActive="active" (click)="setTitle('Builders')">Builders</a> </li> 
<li *ngIf="!auth.isAuthenticated()" class="nav-item"> <a class="nav-link" [routerLink]="['/login']" routerLinkActive="active" (click)="setTitle('Login')">Login</a> </li> 
<li *ngIf="!auth.isAuthenticated()" class="nav-item"> <a class="nav-link" [routerLink]="['/register']" routerLinkActive="active" (click)="setTitle('Register')">Register</a> </li>
 <li *ngIf="auth.isAuthenticated()" class="nav-item"> 
<div ngbDropdown class="d-inline-block">
<button class="btn btn-secondary" id="dropdownBasic1" ngbDropdownToggle>{{ auth.currentUser?.name }}</button 
<div ngbDropdownMenu aria-labelledby="dropdownBasic1"> 
<button class="dropdown-item" (click)="onLogout();">Logout</button>
</div> 
</div>
</li>
</ul>
</div>
</nav> 
</header>
```

请注意，在上面的代码中，我们使用了`ngbDropdown`组件，并且还使用`auth.isAuthenticated()`来确定用户是否已登录。还要注意，我们在下拉菜单中包含了注销链接。

现在，让我们调整登录和注册的模板。

# 向登录页面添加模板绑定

在第七章*，* *使用 Angular-cli 创建渐进式 Web 应用*中，我们已经为应用的所有视图/模板添加了 HTML 标记，但是，我们需要向模板添加 Angular 绑定和模型，以便一切都能正常工作：

1.  打开`./Client/src/app/auth/login/login.component.html`。

1.  将以下绑定函数添加到标签中：

```php
 (ngSubmit)="onSubmit(loginForm)" #loginForm="ngForm"
```

现在，我们将在`./Client/src/app/auth/login/login.component.html`中为电子邮件和密码输入添加`ngModel`。

1.  将以下代码添加到`email`输入中：

```php
 <input  type="email" [(ngModel)]="user.email" name="email" #email="ngModel" class="form-control"  id="email"  aria-describedby="emailHelp"  placeholder="Enter email">
```

1.  将以下代码添加到`password`输入中：

```php
 <input  type="password" [(ngModel)]="user.password" name="password" #password="ngModel" class="form-control"  id="password"  placeholder="Password">
```

# 向注册页面添加模板绑定

现在，让我们在注册页面模板上重复相同的操作：

1.  打开`./Client/src/app/auth/register/register.component.html`。

1.  将以下绑定函数添加到标签中：

```php
 [formGroup]="registerForm" (ngSubmit)="onSubmit()"  class="form-signin"  novalidate
```

注意`formGroup`属性的使用。它是 Angular 响应式表单的一部分，但现在不用担心这个；在本书的后面，我们将讨论模板驱动表单和响应式表单。

现在，在`./Client/src/app/auth/register/register.component.html`中，我们将为`name`、`email`和`password`输入添加`formControlName`。

1.  将以下代码添加到`name`输入：

```php
 <input type="name"  formControlName="name"  class="form-control"  id="name"  aria-describedby="nameHelp"  placeholder="Enter your name">
```

1.  将以下代码添加到`email`输入：

```php
 <input type="email"  formControlName="email" class="form-control"  id="email"  aria-describedby="emailHelp"  placeholder="Enter email">
```

1.  将以下代码添加到`password`输入：

```php
 <input  formControlName="password"  type="password"  name="password" class="form-control"  id="password"  placeholder="Password">
```

# 向 bike-detail 页面添加模板绑定

现在，让我们对`bike-detail`页面模板进行一些调整：

1.  打开`./Client/src/app/bikes/bike-detail/bike-detail.component.html`。

1.  用以下代码替换其内容：

```php
 <main role="main">
        <div class="py-5">
        <div class="container">
        <div *ngIf="isLoading" class="spinner">
                <div class="double-bounce1"></div>
                <div class="double-bounce2"></div>
        </div>
        <ngb-tabset type="pills" *ngIf="!isLoading">
                <ngb-tab title="Bike Detail">
                        <ng-template ngbTabContent>
                        <br>
                        <div class="row">
                                <div class="col-md-4">
                                <img class="card-img-top" src="{{ bike?.picture }}" alt="Card image cap">
                                </div>
                                <div class="col-md-8">
                                <div class="card">
                                        <div class="card-body">
                                        <h5 class="card-title">{{ bike?.model }} | {{ bike?.year }} | Ratings: {{ bike?.average_rating }}
                                                <span *ngIf="userVote">| Your Vote: {{ userVote }}</span>
                                        </h5>
                                        <p class="card-text">{{ bike?.mods }}</p>
                                        </div>
                                        <div *ngIf="bike?.builder" class="card-header">
                                        <strong>Builder</strong>:
                                        <a routerLink="/builders/{{bike?.builder['id']}}">{{ bike?.builder['name'] }}</a>
                                        </div>
                                        <div *ngIf="bike?.items" class="card-header">
                                        <strong>Featured items</strong>:
                                        </div>
                                        <ul class="list-group list-group-flush">
                                        <li *ngFor="let item of bike?.items" class="list-group-item">
                                                <strong>Type</strong>: {{ item.type }} |
                                                <strong>Name</strong>: {{ item.name }} |
                                                <strong>Company</strong>: {{ item.company }}
                                        </li>
                                        </ul>
                                        <div class="card-body">
                                        <ul class="list-unstyled list-inline">
                                                <li class="list-inline-item">Vote: </li>
                                                <li class="list-inline-item">
                                                <a (click)="onVote('1')" class="btn btn-outline-secondary">1</a>
                                                </li>
                                                <li class="list-inline-item">
                                                <a (click)="onVote('2')" class="btn btn-outline-primary">2</a>
                                                </li>
                                                <li class="list-inline-item">
                                                <a (click)="onVote('3')" class="btn btn-outline-success">3</a>
                                                </li>
                                        </ul>
                                        </div>
                                </div>
                                </div>
                        </div>
                        </ng-template>
                </ngb-tab>
                <ngb-tab>
                        <ng-template ngbTabTitle *ngIf="checkBikeOwner()">Edit bike</ng-template>
                        <ng-template ngbTabContent>
                        <br>
                        <form (ngSubmit)="onSubmit(bikeAddForm)" #bikeAddForm="ngForm" name=bikeAddForm class="bg-light px-4 py-4">
                                <div class="form-group">
                                <label for="make">Make</label>
                                <input type="text" [(ngModel)]="bike.make"  name="make" class="form-control" id="make" placeholder="Enter make">
                                </div>
                                <div class="form-group">
                                <label for="model">Model</label>
                                <input type="text" [(ngModel)]="bike.model" name="model" class="form-control" id="model" placeholder="Enter model">
                                </div>
                                <div class="form-group">
                                <label for="year">Year</label>
                                <input type="text" [(ngModel)]="bike.year" name="year" class="form-control" id="year" placeholder="Enter year, ex: 1990, 2000">
                                </div>
                                <div class="form-group">
                                <label for="mods">Mods</label>
                                <textarea type="text" [(ngModel)]="bike.mods" name="mods" class="form-control" id="mods" placeholder="Enter modifications"></textarea>
                                </div>
                                <div class="form-group">
                                <label for="picture">Picture</label>
                                <input type="text" [(ngModel)]="bike.picture" name="picture" class="form-control" id="picture" placeholder="Enter picture url">
                                </div>
                                <div class="form-group">
                                <label for="inputState">Builder</label>
                                <select [(ngModel)]="bike.builder.id" name="builder_id" class="form-control">
                                        <option *ngFor="let builder of builders" [(ngValue)]="builder['id']">{{builder['name']}}</option>
                                </select>
                                </div>
                                <button type="submit" class="btn btn-primary">Submit</button>
                        </form>
                        </ng-template>
                </ngb-tab>
                </ngb-tabset>
        </div>
 </div>
 </main>
```

请注意，我们使用`*ngIf`指令来隐藏我们的自行车，直到自行车对象可用为止。我们还使用点击绑定函数`(click)="onVote('1')"`对自行车进行投票，我们使用`*ngFor="let item of bike?.items"`来列出自行车项目。

我们还使用了来自`NgBootstrap`的`ngb-tab`、`ngb-tabset`指令在此页面上创建两个视图：一个用于显示自行车的详细信息，另一个用于显示编辑表单，以便我们可以编辑自行车的详细信息。请注意，我们使用了一个名为`checkBikeOwner()`的函数来进行简单的检查，以查看登录的用户是否是自行车的所有者。否则，我们会隐藏该选项卡。

`(?)`符号被称为安全导航运算符。

预期结果是我们在下面的截图中看到的：

![](img/c38958b9-0c49-4084-881e-9209111afbe3.png)

现在不用担心表单，因为我们将在本章末尾详细讨论它。

# 向 bike-list 页面添加模板绑定

好了，现在是时候创建`bike-list`模板绑定了：

1.  打开`./Client/src/app/bikes/bike-list/bike-list.component.html`。

1.  用以下代码替换其内容：

```php
<main role="main">
  <div class="py-5 bg-light">
    <div class="container">
      <form>
        <div class="form-group row">
          <label for="search" class="col-sm-2 col-form-label">Bike List</label>
          <div class="col-sm-8">
            <input [(ngModel)]="searchText" [ngModelOptions]="{standalone: true}" placeholder="buscar" type="text" class="form-control"
              id="search" placeholder="Search">
          </div>
          <div class="col-sm-2">
            <div ngbDropdown class="d-inline-block">
              <button class="btn btn-primary" id="dropdownBasicFilter" ngbDropdownToggle>Filter</button>
              <div ngbDropdownMenu aria-labelledby="dropdownBasicFilter">
                <button class="dropdown-item">Year</button>
              </div>
            </div>
          </div>
        </div>
      </form>
      <div *ngIf="isLoading" class="spinner">
        <div class="double-bounce1"></div>
        <div class="double-bounce2"></div>
      </div>
      <div class="row">
        <div class="col-md-4" *ngFor="let bike of bikes | bikeSearch: searchText ">
          <div class="card mb-4 box-shadow">
            <img class="card-img-top" src="{{ bike.picture }}" alt="{{ bike.model }}">
            <div class="card-body">
              <p>{{ bike.model }} | {{ bike.year }}</p>
              <p class="card-text">{{ bike.mods }}</p>
              <a routerLink="/bikes/{{ bike.id }}" class="card-link">Vote</a>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</main>
```

请注意，我们正在使用`ngbDropdown`、`ngbDropdownToggle`和`ngbDropdownMenu`组件，并且我们还使用`*ngFor="let bike of bikes"`来列出`bikes 数组`中的所有自行车，并使用`*ngIf`来显示和隐藏加载消息。

现在，我们可以看到 Angular 的强大之处。通过一些更改，我们的静态模板已经准备好与后端进行交互。但我们仍然需要编写组件的逻辑来将所有内容整合在一起。

在我们这样做之前，让我们调整构建者模板。

# 向 builder-detail 页面添加模板绑定

让我们添加`builder-detail`页面：

1.  打开`./Client/src/app/builders/builder-detail/builder-detail.component.html`。

1.  用以下代码替换其内容：

```php
<main role="main">
  <div class="py-5">
  <div class="container">
  <div *ngIf="isLoading" class="spinner">
    <div class="double-bounce1"></div>
    <div class="double-bounce2"></div>
  </div>
  <ngb-tabset type="pills" *ngIf="!isLoading">
    <ngb-tab title="Bike Detail">
      <ng-template ngbTabContent>
      <br>
      <div class="row">
        <div class="col-md-4">
        <img class="card-img-top" src="{{ bike?.picture }}" alt="Card image cap">
        </div>
        <div class="col-md-8">
        <div class="card">
          <div class="card-body">
          <h5 class="card-title">{{ bike?.model }} | {{ bike?.year }} | Ratings: {{ bike?.average_rating }}
            <span *ngIf="userVote">| Your Vote: {{ userVote }}</span>
          </h5>
          <p class="card-text">{{ bike?.mods }}</p>
          </div>
          <div *ngIf="bike?.builder" class="card-header">
          <strong>Builder</strong>:
          <a routerLink="/builders/{{bike?.builder['id']}}">{{ bike?.builder['name'] }}</a>
          </div>
          <div *ngIf="bike?.items" class="card-header">
          <strong>Featured items</strong>:
          </div>
          <ul class="list-group list-group-flush">
          <li *ngFor="let item of bike?.items" class="list-group-item">
            <strong>Type</strong>: {{ item.type }} |
            <strong>Name</strong>: {{ item.name }} |
            <strong>Company</strong>: {{ item.company }}
          </li>
          </ul>
          <div class="card-body">
          <ul class="list-unstyled list-inline">
            <li class="list-inline-item">Vote: </li>
            <li class="list-inline-item">
            <a (click)="onVote('1')" class="btn btn-outline-secondary">1</a>
            </li>
            <li class="list-inline-item">
            <a (click)="onVote('2')" class="btn btn-outline-primary">2</a>
            </li>
            <li class="list-inline-item">
            <a (click)="onVote('3')" class="btn btn-outline-success">3</a>
            </li>
          </ul>
          </div>
        </div>
        </div>
      </div>
      </ng-template>
    </ngb-tab>
    <ngb-tab>
      <ng-template ngbTabTitle *ngIf="checkBikeOwner()">Edit bike</ng-template>
      <ng-template ngbTabContent>
      <br>
      <form (ngSubmit)="onSubmit(bikeAddForm)" #bikeAddForm="ngForm" name=bikeAddForm class="bg-light px-4 py-4">
        <div class="form-group">
        <label for="make">Make</label>
        <input type="text" [(ngModel)]="bike.make"  name="make" class="form-control" id="make" placeholder="Enter make">
        </div>
        <div class="form-group">
        <label for="model">Model</label>
        <input type="text" [(ngModel)]="bike.model" name="model" class="form-control" id="model" placeholder="Enter model">
        </div>
        <div class="form-group">
        <label for="year">Year</label>
        <input type="text" [(ngModel)]="bike.year" name="year" class="form-control" id="year" placeholder="Enter year, ex: 1990, 2000">
        </div>
        <div class="form-group">
        <label for="mods">Mods</label>
        <textarea type="text" [(ngModel)]="bike.mods" name="mods" class="form-control" id="mods" placeholder="Enter modifications"></textarea>
        </div>
        <div class="form-group">
        <label for="picture">Picture</label>
        <input type="text" [(ngModel)]="bike.picture" name="picture" class="form-control" id="picture" placeholder="Enter picture url">
        </div>
        <div class="form-group">
        <label for="inputState">Builder</label>
        <select [(ngModel)]="bike.builder.id" name="builder_id" class="form-control">
          <option *ngFor="let builder of builders" [(ngValue)]="builder['id']">{{builder['name']}}</option>
        </select>
        </div>
        <button type="submit" class="btn btn-primary">Submit</button>
      </form>
      </ng-template>
    </ngb-tab>
    </ngb-tabset>
    </div>
  </div>
</main>
```

在构建者模板中，我们使用了与之前模板相同的技术。

# 向 builder-list 页面添加模板绑定

现在，是时候添加`builder-list`模板了：

1.  打开`./Client/src/app/builders/builder-list/builder-list.component.html`。

1.  用以下代码替换其内容：

```php
 <main  role="main">
        <div  class="py-5 bg-light">
                <div  class="container">
                        <div *ngIf="isLoading"  class="spinner">
                                <div  class="double-bounce1"></div>
                                <div  class="double-bounce2"></div>
                        </div>
                        <div  class="row">
                                <div  class="col-md-4" *ngFor="let builder of builders">
                                        <div  class="card mb-4 box-shadow">
                                                <div  class="card-header">
                                                        <h4  class="my-0 font-weight-normal">{{ builder?.name }}</h4>
                                                </div>
                                                <div  class="card-body">
                                                        <p  class="mt-3 mb-4">{{ builder?.description }</p>
                                                        <button  routerLink="/builders/{{ builder?.id }}"  type="button"  class="btn btn-lg btn-block btn-outline-primary">View Bike</button>
                                                </div>
                                                <div  class="card-footer text-muted">
                                                        {{ builder?.location }}
                                                </div>
                                        </div>
                                </div>
                        </div>
                </div>
        </div>
 </main>
```

现在，我们有足够的代码让我们的模板呈现后端的内容。为此，我们只需要对后端进行一些微小的调整，并在组件中编写逻辑。

# 在 Laravel 后端设置 CORS

在我们的后端进行必要的更改之前，让我们谈谈今天现代 Web 应用中非常重要且非常常见的一个主题，即 CORS。

当我们使用`XMLHttpRequest`或`Fetch API`从给定服务器获取数据时，这个调用通常是从另一个应用程序和其他地方执行的。

出于安全原因，浏览器限制跨源 HTTP 请求。

理解 CORS 工作原理的一个简单例子是：想象一个在特定域中运行的前端应用，例如`http://mysimpledomain.com`，向另一个域中的另一个应用`http://myanothersimpledomain.com`发送请求。

CORS 是一种机制，它使用额外的 HTTP 头来告诉浏览器允许一个 Web 应用程序在一个起源`http://mysimpledomain.com`上运行，并且有权限从不同起源的服务器`http://myanothersimpledomain`访问选定的资源。

您可以在[`www.w3.org/TR/cors/`](https://www.w3.org/TR/cors/)上阅读有关 CORS 的更多信息。

# 设置 Laravel CORS

Laravel 在其应用程序中使用 CORS 具有出色的支持。让我们看看如何使用一个名为`barryvdh/laravel-cors`的库来配置它：

1.  在`chapter-10`文件夹中打开您的终端窗口。

1.  输入以下命令：

```php
 docker-compose up -d
```

1.  现在，在`php-fpm`容器中，输入以下命令：

```php
 docker-compose exec php-fpm bash
```

这一步非常重要。如果您忘记了这个命令，很可能会出现错误，或者您可能会冒着使用本地 composer 版本来执行以下命令的风险。

1.  在容器的 bash 中，输入以下命令：

```php
 composer require barryvdh/laravel-cors
```

由于最新版本的 Laravel（5.6），我们的新库已经准备好使用。让我们只做一个小小的改变。

1.  打开`./Server/app/Http/Kernel.php`文件并将以下代码添加到`middlewareGroup` API 中：

```php
 protected $middlewareGroups = [

        'web'  => [
                ...
        ],
        'api'  => [
                \Barryvdh\Cors\HandleCors::class,
                'throttle:60,1',
                'bindings',
        ],
```

非常重要的一点是，我们在 API 标签的依赖项的第一行中添加了`\Barryvdh\Cors\HandleCors :: class`。这非常重要，因为我们避免在前端应用程序上获得状态码 0 的错误。

我们已经准备好了！

# 将 Angular 服务与应用程序组件连接起来

现在，我们将连接我们在本书中创建的所有 Angular 服务和模板。为此，我们将创建我们将在组件中使用的逻辑和函数。

在开始之前，让我们将 API 的端点设置为 Angular 环境文件中的一个变量。

# 添加环境配置

正如其名称所示，此文件用于在我们的应用程序中设置环境变量。最好的部分是，Angular 默认配置了一个 dev 和 prod 环境，并且非常简单易用。我们还可以设置各种变量。

在此示例中，我们正在使用开发文件来设置后端 URL。

打开`./Client/src/environments/environment.ts`文件并添加以下 URL：

```php
     export  const  environment  = {
                production: false,
                apiUrl: 'http://localhost:8081/api'
        };
```

如您所见，`environments`文件夹中还有一个名为`environment.prod.ts`的文件。

现在不要担心这个文件，因为我们将在书中稍后使用它。

# 创建导航方法

现在，是时候在`nav.component.ts`中创建导航行为了，让我们看看我们可以如何做到这一点：

1.  打开`./Client/src/layout/nav/nav.component.ts`并在核心导入之后添加以下导入：

```php
 import { Router } from  '@angular/router';
 import { Title } from  '@angular/platform-browser';

 // App imports
 import { AuthService } from  '../../pages/auth/_services/auth.service';
```

1.  仍然在`./Client/src/layout/nav/nav.component.ts`中，让我们创建`constructor()`函数：

```php
 public  constructor(
        private  titleTagService:  Title,
        public  auth:  AuthService,
        private  router:  Router ) {}
```

在这里，我们使用内置的 Angular 服务`Title`来在导航模板之间更新页面的标题标签。请记住，我们的应用程序是一个 SPA，我们不希望在所有页面上保持相同的标题。

此外，我们将使用身份验证服务来显示已登录到应用程序的用户的名称，并且我们还将使用此服务的注销功能来注销用户。因此，让我们创建这个函数。

1.  在析构函数之后添加以下代码：

```php
 public  setTitle( pageTitle:  string) {
        this.titleTagService.setTitle( pageTitle );
 }
```

1.  现在，在`ngOnInit()`函数中，添加以下代码：

```php
 if (this.auth.getToken()) {
        this.auth.getUser().subscribe();
 }
```

1.  最后一步是在`ngOnInit()`函数之后添加`logout()`函数。添加以下代码：

```php
 onLogout() {
        this.auth.onLogout().subscribe();
 }
```

现在，我们的应用程序导航已经准备好使用。预期结果如下截图所示：

![](img/8b9295c5-18ee-42c2-9f66-9c64e5451a9e.png)

导航视图

# 创建 bike-detail 方法

让我们创建`bike-detail`组件：

1.  打开`./Client/src/pages/bikes/bike-detail/bike-detail.component.ts`并在核心导入之后添加以下导入：

```php
 import { ActivatedRoute } from  '@angular/router';

 // App imports
 import { Bike } from  '../bike';
 import { BikesService } from  '../_services/bikes.service';
 import { AuthService } from  '../../auth/_services/auth.service';
 import { User } from  './../../auth/user';
```

1.  在`BikeDetailComponent`类声明后添加以下属性：

```php
 bike:  Bike;
 isLoading:  Boolean  =  false;
 userVote:  number;
 builders: Array<Object> = [
        {id: 1, name: 'Diamond Atelier'},
        {id: 2, name: 'Deus Ex Machina\'s'},
        {id: 3, name: 'Rough Crafts'},
        {id: 4, name: 'Roldand Sands'},
        {id: 5, name: 'Chopper Dave'}
 ];
```

请注意，我们正在使用`Bike`模型作为我们的`bike`属性的类型，并创建一个简单的数组来保存我们的构建者。

请注意，在真实的网络应用程序中，最好从服务器获取构建者列表，以避免在组件内部变得硬编码。

1.  在`./Client/src/pages/bikes/bike-detail/bike-detail.component.ts`中，让我们创建`constructor()`函数：

```php
 constructor(
        private  bikeService:  BikesService,
        private  route:  ActivatedRoute,
        private  auth:  AuthService ) {}
```

我们将使用`ActivatedRoute`来在本节后面获取自行车 ID。

1.  在`ngOnInit()`函数中，添加以下代码：

```php
 // Get bike details
 this.getBikeDetail();
```

现在，让我们创建`getBikeDetail()`函数。

1.  在`ngOnInit()`函数之后添加以下代码：

```php
 getBikeDetail():  void {
        this.isLoading  =  true;
        const  id  =  +this.route.snapshot.paramMap.get('id');
        this.bikeService.getBikeDetail(id)
                .subscribe(bike  => {
                        this.isLoading  =  false;
                        this.bike  =  bike['data'];
        });
 }
```

1.  现在，让我们添加`onVote()`函数。在`getBikeDetail()`函数之后添加以下代码：

```php
 onVote(rating:  number, id:  number):  void {
        // Check if user already vote on a bike
        if (this.checkUserVote(this.bike.ratings)) {
                alert('you already vote on this bike');
                return;
        }
        // Get bike id
        id  =  +this.route.snapshot.paramMap.get('id');
        // post vote
        this.bikeService.voteOnBike(rating, id)
                .subscribe(
                        (response) => {
                                this.userVote  =  response.data.rating;
                                // Update the average rating and rating object on bike
                                this.bike['average_rating'] =  response.data.average_rating;
                                // Update ratings array
                                this.bike.ratings.push(response.data);
                        }
                );
 }
```

1.  现在，我们将创建一个函数，检查已登录用户是否已经对所选自行车进行了投票。请记住，`RatingController.php`正在使用`firstOrCreate`方法：

```php
     public  function  store(Request $request, Bike $bike)
        {
                $rating =  Rating::firstOrCreate(
                        [
                        'user_id'  => $request->user()->id,
                        'bike_id'  => $bike->id,
                        ],
                        ['rating'  => $request->rating]
                );
                return  new  RatingResource($rating);
        }
```

我们只会注册第一次投票。因此，我们需要向用户显示一个简单的消息作为`Vote`函数的反馈。

1.  在`onVote()`函数之后添加以下代码：

```php
 checkUserVote(ratings:  any[]):  Boolean {
        const  currentUserId  =  this.auth.currentUser.id;
        let  ratingUserId:  number;
        Object.keys(ratings).forEach( (i) => {
                ratingUserId  =  ratings[i].user_id;
        });
        if ( currentUserId  ===  ratingUserId ) {
                return  true;
        } else {
                return  false;
        }
 }
```

1.  以下方法使用提交函数来更新`bike`记录。在`checkUserVote()`函数之后添加以下代码：

```php
 onSubmit(bike) {
        this.isLoading = true;
        const id = +this.route.snapshot.paramMap.get('id');
        this.bikeService.updateBike(id, bike.value)
        .subscribe(response => {
                this.isLoading = false;
                this.bike = response['data'];
        });
 }
```

请注意，在此步骤中，我们正在使用`bikeService`的`updateBike`方法。

1.  最后一个方法是一个简单的函数，用于检查自行车所有者。请记住，用户只能编辑自己的自行车。在`onSubmit()`函数之后添加以下代码：

```php
 checkBikeOwner(): Boolean {
        if (this.auth.currentUser.id === this.bike.user.id) {
                return true;
        } else {
                return false;
        }
 }
```

在此代码中，我们使用`authService`来获取`User.id`，然后与`bike.user.id`进行比较。

当我们访问`http://localhost:4200/bikes/3` URL 时，此页面的预期结果将类似于以下屏幕截图：

![](img/c22e87ff-c17c-4e1e-98f0-3022a69dec71.png)

自行车详细信息屏幕

请注意，我们可以在此自行车上看到编辑按钮，因为我们的应用程序种子已经用一些示例信息填充了数据库。

因此，如果我们点击`编辑自行车`按钮，我们将看到类似于以下的东西：

![](img/fc0b5edf-fba8-4ec2-aa1e-08dfa1cd5192.png)

编辑自行车表单

# 创建自行车列表方法

让我们创建`bike-list`组件：

1.  打开`./Client/src/pages/bikes/bike-list/bike-list.component.ts`并在核心导入之后添加以下导入：

```php
 import { NgbDropdown } from '@ng-bootstrap/ng-bootstrap/dropdown/dropdown.module';

 // App imports
 import { Bike } from '../bike';
 import { BikesService } from '../_services/bikes.service';
```

1.  在`bike-list.component`类声明之后添加以下属性：

```php
 // Using Bike Model class
 bikes: Bike[];
 isLoading: Boolean = false;
 public searchText: string;
```

1.  在`./Client/src/pages/bikes/bike-list/bike-list.component.ts`中，让我们创建`constructor()`函数：

```php
 constructor(
        private bikeService: BikesService) {}
```

1.  在`ngOnInit()`函数中，添加以下代码：

```php
 // Get bike list
 this.getBikes();
```

现在，让我们创建`this.getBikes()`函数。

1.  在`ngOnInit()`函数之后添加以下代码：

```php
 getBikes(): void {
 this.isLoading = true;
 this.bikeService.getBikes()
        .subscribe(
        response => this.handleResponse(response),
        error => this.handleError(error));
 }
```

请注意，在此代码中，我们使用两个函数来处理成功和错误响应。可以将所有内容写在`subscribe()`函数内，但更好的组织技术是将它们分开。

1.  在`getBikes()`函数之后添加以下代码：

```php
 protected handleResponse(response: Bike[]) {
        this.isLoading = false,
        this.bikes = response;
 }

 protected handleError(error: any) {
        this.isLoading = false,
        console.error(error);
 }
```

在受保护的`handleError`方法中，我们只是使用`console.log()`来显示错误。

当我们访问`http://localhost:4200/bikes` URL 时，此页面的预期结果将类似于以下屏幕截图：

![](img/00dc1a31-cc36-47fc-9898-07c50840d2e5.png)

自行车列表页面

# 创建 builder-detail 方法

现在，是时候创建`builder-detail`组件了。让我们看看：

1.  打开`./Client/src/pages/builders/builder-detail/builder-detail.component.ts`并在核心导入之后添加以下导入：

```php
 import { ActivatedRoute } from '@angular/router';

 // App imports
 import { Builder } from './../builder';
 import { BuildersService } from '../_services/builders.service';
```

1.  在`builder-detail.component`类声明之后添加以下属性：

```php
   builder: Builder;
   isLoading: Boolean = false;
```

1.  在`./Client/src/pages/builders/builder-detail/builder-detail.component.ts`中，让我们创建`constructor()`函数：

```php
 constructor(
        private buildersService: BuildersService,
        private route: ActivatedRoute) { }
```

1.  在`ngOnInit()`函数中，添加以下代码：

```php
 ngOnInit() {
        // Get builder detail
        this.getBuilderDetail();
 }
```

现在，让我们创建`this.getBuilderDetail()`函数。

1.  在`ngOnInit()`函数之后添加以下代码：

```php
 getBuilderDetail(): void {
        this.isLoading = true;
        const id = +this.route.snapshot.paramMap.get('id');
        this.buildersService.getBuilderDetail(id)
                .subscribe(builder => {
                this.isLoading = false;
                this.builder = builder['data'];
        });
 }
```

当我们访问`http://localhost:4200/builders/4`URL 时，此页面的预期结果将类似于以下屏幕截图：

![](img/bc81c670-a648-45f3-a5da-292174dcbe32.png)

建筑师详细页面

# 创建`builder-list`方法

现在，让我们创建`builder-list`方法来列出所有建筑师：

1.  打开`./Client/src/pages/builders/builder-list/builder-list.component.ts`并在核心导入后添加以下导入：

```php
 // App imports
 import { Builder } from './../builder';
 import { BuildersService } from '../_services/builders.service';
```

1.  在`BuilderListComponent`类声明后添加以下属性：

```php
 // Using Builder Model class
 builders: Builder[];
 isLoading: Boolean = false;
```

1.  仍然在`./Client/src/pages/builders/builder-list/builder-list.component.ts`中，让我们创建`constructor()`函数：

```php
 constructor(private builderService: BuildersService) { }
```

1.  在`ngOnInit()`函数内添加以下代码：

```php
 ngOnInit() {
        // Get builder detail
        this.getBuilders();
 }
```

1.  在`ngOnInit()`函数后面添加以下代码：

```php
 getBuilders(): void {
 this.isLoading = true;
 this.builderService.getBuilders()
        .subscribe(
        response => this.handleResponse(response),
        error => this.handleError(error));
 }
```

请注意，在此代码中，我们使用两个函数来处理成功和错误响应。可以将所有内容写在`subscribe()`函数内，但更好的组织技术是将它们分开。

1.  在`getBuilders()`函数后面添加以下代码：

```php
 protected handleResponse(response: Builder[]) {
        this.isLoading = false,
        this.builders = response;
 }
 protected handleError(error: any) {
        this.isLoading = false,
        console.error(error);
 }
```

最后，我们已经准备好所有组件。

当我们访问`http://localhost:4200/builders`URL 时，此页面的预期结果将类似于以下屏幕截图：

![](img/43dcaa69-1ad8-44b6-83b1-96a5a4d47b52.png)

建筑师列表页面

# 处理 Angular 管道、表单和验证

在本节中，我们将看到如何在自行车列表页面内创建一个简单的搜索组件，使用新的管道功能。我们还将看看如何以两种方式创建 Angular 表单：使用模板驱动表单和响应式表单。最后，我们将向您展示如何在 Bootstrap CSS 中使用表单验证。

# 创建管道过滤器

在 Angular 中，管道是一种简单的过滤和转换数据的方式，与旧的 AngularJS 过滤器非常相似。

在 Angular 中，我们有一些默认的管道（`DatePipe`、`UpperCasePipe`、`LowerCasePipe`、`CurrencyPipe`和`PercentPipe`），我们也可以创建自己的管道。

要创建自定义管道，我们可以使用 Angular CLI 为我们生成脚手架。让我们看看它是如何工作的：

1.  打开您的终端窗口，并在`./Client/src/app`内输入以下命令：

```php
 ng g pipe pages/bikes/_pipes/bikeSearch
```

像往常一样，Angular CLI 会负责创建文件和适当的导入。

1.  打开`./Client/src/app/pages/bikes/_pipes/bike-search.pipe.ts`并在`BikeSearchPipe`类内添加以下代码：

```php
 transform(items: any, searchText: string): any {
 if (searchText) {
        searchText = searchText.toLowerCase();
        return items.filter((item: any) => item.model.toLowerCase().indexOf(searchText) > -1);
 }
 return items;
 }
```

先前的`transform`函数接收两个参数：来自自行车列表页面搜索框的输入字段的列表和搜索字符串。因此，让我们看看如何在`bike-list`模板内使用它们。

1.  打开`./Client/src/app/pages/bikes/bike-list/bike-list.component.ts`并在搜索输入字段内添加以下属性：

```php
 <input [(ngModel)]="searchText" [ngModelOptions]="{standalone: true}" placeholder="buscar" type="text" class="form-control"
       id="search" placeholder="Search">
```

既然我们已经有了搜索模型，让我们在`*ngFor`循环上添加管道过滤器。

1.  在`*ngFor`属性内添加以下代码：

```php
 <div class="col-md-4" *ngFor="let bike of bikes | bikeSearch: searchText ">...</div>
```

因此，当我们在搜索输入框中输入自行车型号时，我们将看到以下屏幕截图：

![](img/0ab1fae1-2b66-4c09-8f9d-e306e199c530.png)

搜索字段工作

现在，让我们看看如何实现 Angular 表单。

# 介绍 Angular 表单

众所周知，表单是任何现代 Web 应用程序的重要组成部分，用于登录用户到应用程序，添加产品，并向博客发送评论。有些表单非常简单，但其他表单可能有一系列字段，甚至有许多步骤和页面，带有大量输入字段。

在 Angular 中，我们可以实现两种类型的表单：

+   模板驱动表单

+   响应式表单或模型驱动表单

两者同样强大，并属于`@angular/forms`库。它们基于相同的表单控件类。然而，它们有不同的哲学、编程风格和技术，验证也不同。在下一节中，我们将看到每种技术的独特之处。

# 理解 Angular 模板驱动表单

正如我们之前解释的，模板驱动表单非常类似于 AngularJS 表单，并使用诸如`ngModel`和可能`required`、`minlength`、`maxlength`等指令。当我们使用这些表单指令时，我们让模板在幕后完成工作。

# 审查登录表单模板和组件

一个很好的例子来理解模板驱动表单是登录表单。让我们看一下`login.component.html`和`login.component.ts`：

1.  打开`./Client/src/app/pages/auth/login/login.component.html`并审查模板输入标签：

```php
 [(ngModel)]="user.email"  name="email"
 [(ngModel)]="user.password" name="password"
```

请注意，我们正在使用`ngModel = [(ngModel)]`的双向数据绑定语法。这意味着我们可以从登录组件类设置初始数据，但也可以更新它。

请记住，Angular 的`ngModel`可以以三种不同的方式使用：

+   `ngModel`：没有绑定或赋值，并且依赖于 name 属性

+   `[ngModel]`：单向数据绑定语法

+   `[(ngModel)]`：双向数据绑定语法

对于提交按钮事件，我们只是使用了`(ngSubmit)="onSubmit(loginForm)" #loginForm="ngForm"`指令，传递`loginForm`。

现在我们的`login.component.ts`完整了，我们唯一需要的是`onSubmit`函数。

1.  现在，让我们通过用以下代码替换`login.component.ts`来编辑它：

```php
 import { Component, OnInit } from '@angular/core';
 import { Router, ActivatedRoute } from '@angular/router';

 // App imports
 import { AuthService } from '../_services/auth.service';
 import { User } from '../user';

 @Component({
 selector: 'app-login',
 templateUrl: './login.component.html',
 styleUrls: ['./login.component.scss']
 })
 export class LoginComponent implements OnInit {
        user: User = new User();
        error: any;
        returnUrl: string;

        constructor(
                private authService: AuthService,
                private router: Router,
                private route: ActivatedRoute) { }

        ngOnInit() {
                //  Set the return url
                this.returnUrl = this.route.snapshot.queryParams['returnUrl'] || '/';
        }

        onSubmit(loginForm): void {
                this.authService.onLogin(this.user).subscribe(
                (response) => {
                        // get return url from route parameters or default to '/'
                        this.router.navigate([this.returnUrl]);
                },
                (error) => {
                        this.error = error.error;
                }
                );
                // Clear form fields
                loginForm.reset();
        }

 }
```

请注意，我们将`loginForm`传递给`onSubmit(loginForm)`函数，并使用`authService`将数据发送到端点。

# 理解 Angular 响应式/模型驱动表单

响应式/模型驱动表单和模板驱动表单之间的一个区别是使用诸如`ngModel`之类的指令。

这背后的原则是，我们使用表单 API 将指令负责地传递到`component.ts`代码中。这具有更大的能力，对于工作来说非常高效，将所有逻辑保留在同一个地方，我们很快就会看到。

# 审查注册表单模板和组件

一个很好的例子来理解模型驱动表单是注册表单。让我们看一下`register.component.html`和`register.component.ts`：

1.  打开`./Client/src/app/pages/auth/register/register.component.html`并审查模板输入标签：

```php
 formControlName="name"
 formControlName="email"
 formControlName="password"
```

这几乎是我们在模板驱动表单中使用的相同符号，但更清晰一些。在这里，我们不需要`name`属性。

对于提交按钮事件，我们只是使用了`[formGroup]="registerForm" (ngSubmit)="onSubmit()"`属性和绑定函数。

1.  现在，让我们创建`register.component.ts`。用以下代码替换它的代码：

```php
 import { Component, OnInit } from '@angular/core';
 import { Router } from '@angular/router';
 import { FormBuilder, FormGroup, Validators } from '@angular/forms';

 // App imports
 import { User } from '../user';
 import { AuthService } from '../_services/auth.service';

 @Component({
 selector: 'app-register',
 templateUrl: './register.component.html',
 styleUrls: ['./register.component.scss']
 })
 export class RegisterComponent implements OnInit {

        user: User = new User();
        error: any;
        registerForm: FormGroup;

        constructor(private authService: AuthService, private router: Router, private fb: FormBuilder) {
                this.createForm();
        }

        ngOnInit() {}

        createForm() {
                this.registerForm = this.fb.group({
                name: [this.user.name, Validators.compose([Validators.required])],
                email: [this.user.email, Validators.compose([Validators.required, Validators.email ])],
                password: [this.user.password, Validators.compose([Validators.required, Validators.minLength(6)])],
                });
        }

        onSubmit(): void {

                this.authService.onRegister(this.registerForm.value).subscribe(
                (response) => {
                        this.router.navigate(['bikes']);
                },
                (response) => {
                        if (response.status === 422) {
                        Object.keys(response.error).map((err) => {
                                this.error = `${response.error[err]}`;
                        });

                        } else {
                        this.error = response.error;
                        }
                }
                );
        }

 }
```

请注意，在这段代码中，我们正在处理提交函数上的错误消息。在接下来的示例中，我们将看看如何在两种表单上实现表单验证，但现在让我们回顾一些重要的要点。

1.  打开`./Client/src/app/pages/auth/register/register.component.ts`；让我们回顾`registerComponent`类。

我们可以注意到的第一个区别是文件顶部的`FormBuilder`、`FormGroup`和`Validators`的导入：

```php
     import { FormBuilder, FormGroup, Validators } from
     '@angular/forms';
```

我们还需要在`auth.module.ts`内导入`ReactiveFormsModule`：

```php
     import { FormsModule, ReactiveFormsModule } from '@angular/forms';
```

我们可以使用`FormBuilder` API 在`createForm()`函数内创建表单：

```php
     createForm() {
                this.registerForm = this.fb.group({
                        name: [this.user.name, Validators.compose([Validators.required])],
                        email: [this.user.email, Validators.compose([Validators.required, Validators.email ])],
                        password: [this.user.password, Validators.compose([Validators.required, Validators.minLength(6)])],
                });
        }
```

在这里，我们使用`Validators`直接从`component.ts`代码中添加表单验证。很棒，对吧？

请记住，`fb`变量保存了我们放在构造函数中的`FormBuilder`：private `fb:FormBuilder`。我们还将`registerForm`设置为`RegisterClass`内的`FormGroup`。

# 添加前端表单验证

正如我们今天所知道的，当谈到用户体验时，向最终用户显示持续的反馈是一个很好的做法，因此在将表单发送到后端之前验证表单是一个很好的做法。

在本节中，我们将看看如何向登录和注册表单添加表单验证。

# 处理模板驱动表单上的表单验证

打开`./Client/src/app/pages/auth/login/login.component.html`并用以下代码替换表单标记：

```php
 <form class="form-signin" (ngSubmit)="onSubmit(loginForm)" #loginForm="ngForm">
        <div class="text-center mb-4">
                <h1 class="h3 mt-3 mb-3 font-weight-normal">Welcome</h1>
                <p>Motorcycle builders and road lovers</p>
                <hr>
        </div>
        <div class="form-group" [ngClass]="{ 'has-error': !email.valid && (email.dirty || email.touched) }">
                <label for="email">Email address</label>
                <input type="email" [(ngModel)]="user.email"  name="email" #email="ngModel" required class="form-control" id="email" aria-describedby="emailHelp" placeholder="Enter email">
                <div *ngIf="email.invalid && (email.dirty || email.touched)" class="form-feedback">
                        <div *ngIf="email?.errors.required">Email is required</div>
                        <div *ngIf="email?.errors.email">Email must be a valid email address</div>
                </div>
        </div>
        <div class="form-group" [ngClass]="{ 'has-error': !password.valid && (password.dirty || password.touched) }">
                <label for="password">Password</label>
                <input type="password" [(ngModel)]="user.password" name="password" #password="ngModel" required minlength="6" class="form-control" id="password" placeholder="Password">
                <div *ngIf="password.invalid && (password.dirty || password.touched)" class="form-feedback">
                        <div *ngIf="password?.errors.required">Password is required</div>
                        <div *ngIf="password?.errors.minlength">Password must be at least 6 characters</div>
                </div>
        </div>
        <div  *ngIf="error" class="alert alert-danger" role="alert">
                Ops: {{ error.error }}
        </div>
        <button [disabled]="!loginForm.valid" class="btn btn-lg btn-primary btn-block mt-5" type="submit">Login</button>
 </form>
```

让我们回顾一下之前的代码。

请注意，我们正在使用内置在 Angular 指令中的`[ngClass]`将错误类应用于`div`表单组，如果表单无效：

```php
     // Email field
        class="form-group" [ngClass]="{ 'has-error': !email.valid && (email.dirty || email.touched) }"
        // Password field
        class="form-group" [ngClass]="{ 'has-error': !password.valid && (password.dirty || password.touched) }"
```

为了显示错误消息，我们将在输入字段后创建两个新的 div：

```php
     // Email validation
        <div *ngIf="email.invalid && (email.dirty || email.touched)" class="form-feedback">
                <div *ngIf="email?.errors.required">Email is required</div>
                <div *ngIf="email?.errors.email">Email must be a valid email address</div>
        </div>
        // Password validation
        <div *ngIf="password.invalid && (password.dirty || password.touched)" class="form-feedback">
                <div *ngIf="password?.errors.required">Password is required</div>
                <div *ngIf="password?.errors.minlength">Password must be at least 6 characters</div>
        </div>
```

借助`ngIf`和表单状态（脏、触摸），我们可以看到每个错误，如果输入字段符合此条件。

下一个规则是，以下`div`显示可能发生的后端错误：

```php
     <div  *ngIf="error" class="alert alert-danger" role="alert">
                Ops: {{ error.error }}
        </div>
```

最后，使用`[disabled]`指令在提交按钮上设置验证：

```php
     <button [disabled]="!loginForm.valid" class="btn btn-lg btn-primary btn-block mt-5" type="submit">Login</button>
```

我们表单的最终结果将类似于以下内容：

![](img/df2d3413-38bb-4bba-a188-f946432f8bb4.png)

登录表单验证

# 处理基于模型的表单验证

打开`./Client/src/app/pages/auth/register/register.component.html`并用以下代码替换表单标签：

```php
 <form [formGroup]="registerForm" (ngSubmit)="onSubmit()"  class="form-register" novalidate>
        <div class="text-center mb-4">
                <h1 class="h3 mt-3 mb-3 font-weight-normal">Welcome</h1>
                <p>Motorcycle builders and road lovers</p>
                <hr>
        </div>
        <div class="form-group" [ngClass]="{ 'has-error': !registerForm.get('name').valid && (registerForm.get('name').dirty || registerForm.get('name').touched) }">
                <label for="name">Name</label>
                <input type="name" formControlName="name" class="form-control" id="name" aria-describedby="nameHelp" placeholder="Enter your name">
                <div class="form-feedback"
                        *ngIf="registerForm.get('name').errors && (registerForm.get('name').dirty || registerForm.get('name').touched)">
                        <div *ngIf="registerForm.get('name').hasError('required')">Name is required</div>
                </div>
        </div>
        <div class="form-group" [ngClass]="{ 'has-error': !registerForm.get('email').valid && (registerForm.get('email').dirty || registerForm.get('email').touched) }">
                <label for="email">Email address</label>
                <input type="email" formControlName="email" class="form-control" id="email" aria-describedby="emailHelp" placeholder="Enter email">
                <div class="form-feedback"
                *ngIf="registerForm.get('email').errors && (registerForm.get('email').dirty || registerForm.get('email').touched)">
                        <div *ngIf="registerForm.get('email').hasError('required')">Email is required</div>
                        <div *ngIf="registerForm.get('email').hasError('email')">Email must be a valid email address</div>
                </div>
        </div>
        <div class="form-group" [ngClass]="{ 'has-error': !registerForm.get('password').valid && (registerForm.get('password').dirty || registerForm.get('password').touched) }">
                <label for="password">Password</label>
                <input type="password" formControlName="password"  class="form-control" id="password" placeholder="Password">
                <div class="form-feedback"
                *ngIf="registerForm.get('password').errors && (registerForm.get('password').dirty || registerForm.get('password').touched)">
                        <p *ngIf="registerForm.get('password').hasError('required')">Password is required</p>
                        <p *ngIf="registerForm.get('password').hasError('minlength')">Password must be 6 characters long, we need another {{registerForm.get('password').errors['minlength'].requiredLength - registerForm.get('password').errors['minlength'].actualLength}} characters </p>
                </div>
        </div>
        <div  *ngIf="error" class="alert alert-danger" role="alert">
                Ops: {{ error }}
        </div>
        <button [disabled]="!registerForm.valid" class="btn btn-lg btn-primary btn-block mt-5" type="submit">Register</button>
 </form>
```

让我们回顾一下之前的代码。

请注意，我们正在使用内置在 Angular 中的`[ngClass]`将`error`类应用于`div`表单组，如果表单无效：

```php
     // Name field
        class="form-group" [ngClass]="{ 'has-error': !registerForm.get('name').valid && (registerForm.get('name').dirty || registerForm.get('name').touched) }"
        // Email field
        class="form-group" [ngClass]="{ 'has-error': !registerForm.get('email').valid && (registerForm.get('email').dirty || registerForm.get('email').touched) }"
        // Password field
        class="form-group" [ngClass]="{ 'has-error': !registerForm.get('password').valid && (registerForm.get('password').dirty || registerForm.get('password').touched) }"
```

在这里，您可以注意到我们正在使用`registerForm.get()`方法，使输入字段与登录表单有所不同。

为了显示错误消息，我们将在输入字段后创建三个新的`div`：

```php
     // Name validation
        <div class="form-feedback"
                *ngIf="registerForm.get('name').errors && (registerForm.get('name').dirty || registerForm.get('name').touched)">
                <div *ngIf="registerForm.get('name').hasError('required')">Name is required</div>
        </div>

        // Email validation
        <div class="form-feedback"
                *ngIf="registerForm.get('email').errors && (registerForm.get('email').dirty || registerForm.get('email').touched)">
                <div *ngIf="registerForm.get('email').hasError('required')">Email is required</div>
                <div *ngIf="registerForm.get('email').hasError('email')">Email must be a valid email address</div>
        </div>

        // Password validation
        <div class="form-feedback"
                *ngIf="registerForm.get('password').errors && (registerForm.get('password').dirty || registerForm.get('password').touched)">
                <p *ngIf="registerForm.get('password').hasError('required')">Password is required</p>
                <p *ngIf="registerForm.get('password').hasError('minlength')">Password must be 6 characters long, we need another {{registerForm.get('password').errors['minlength'].requiredLength - registerForm.get('password').errors['minlength'].actualLength}} characters </p>
        </div>
```

下一个规则是，以下`div`用于显示可能发生的后端错误：

```php
     <div  *ngIf="error" class="alert alert-danger" role="alert">
                Ops: {{ error }}
        </div>
```

最后，使用`[disabled]`指令在提交按钮上设置验证：

```php
     <button [disabled]="!registerForm.valid" class="btn btn-lg btn-
      primary btn-block mt-5" type="submit">Register</button>
```

我们表单的最终结果将类似于以下内容：

![](img/2f67b8b7-9c41-4218-a4b7-5991f3bcfd9f.png)

注册表单验证

在下一个截图中，我们可以看到后端错误，这是我们尝试插入一个已经在使用中的电子邮件地址的地方：

![](img/ea9e5c96-3cbc-456e-8a02-48134be9257c.png)后端错误消息

# 总结

我们完成了另一个章节，我们的示例应用程序具有现代 Web 应用程序的所有关键要点。我们学会了如何安装、定制和扩展 Bootstrap CSS 框架，并学会了如何使用`NgBootstrap`组件。

我们还了解了如何设置组件和服务，表单验证以及许多其他非常有用的技术。

在下一章中，我们将看到如何为 SCSS 和 TS 文件设置 linter，以及如何使用 Docker 镜像进行部署。
