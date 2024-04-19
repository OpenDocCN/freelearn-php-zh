# 第九章：创建服务和用户认证

在本章中，我们有很多工作要做。我们将创建许多新东西，并对一些东西进行重构。这是以一种规律和渐进的方式学习东西的好方法。

我们将深入研究 Angular 的 HTTP 模块的操作和使用，该模块被称为`HttpClient`。

此外，我们将看到如何使用拦截器和处理错误。

Angular 的新版本提供了非常有用的工具来创建现代 Web 应用程序，在本章中，我们将使用其中许多资源。

在本章中，我们将涵盖以下主题：

+   处理模型和类

+   使用新的`HttpModule`和`HttpModuleClient`来处理 XHR 请求

+   处理`HttpErrorHandler`服务

+   如何使用授权头

+   如何使用路由守卫保护应用程序路由

# 准备基线代码

现在，我们需要准备我们的基线代码，这个过程与我们在上一章中所做的非常相似。让我们按照以下步骤进行：

1.  复制`chapter-08`文件夹中的所有内容。

1.  将文件夹重命名为`chapter-09`。

1.  删除`storage-db`文件夹。

现在，让我们对`docker-compose.yml`文件进行一些更改，以使其适应新的数据库和服务器容器。

1.  打开`docker-compose.yml`并用以下代码替换其内容：

```php
 version: "3.1"
 services:
     mysql:
       image: mysql:5.7
       container_name: chapter-09-mysql
       working_dir: /application
       volumes:
         - .:/application
         - ./storage-db:/var/lib/mysql
       environment:
         - MYSQL_ROOT_PASSWORD=123456
         - MYSQL_DATABASE=chapter-09
         - MYSQL_USER=chapter-09
         - MYSQL_PASSWORD=123456
       ports:
         - "8083:3306"
     webserver:
       image: nginx:alpine
       container_name: chapter-09-webserver
       working_dir: /application
       volumes:
         - .:/application
         -./phpdocker/nginx/nginx.conf:/etc/nginx/conf.d/default
         .conf
       ports:
         - "8081:80"
     php-fpm:
       build: phpdocker/php-fpm
       container_name: chapter-09-php-fpm
       working_dir: /application
       volumes:
         - ./Server:/application
         - ./phpdocker/php-fpm/php-ini-
           overrides.ini:/etc/php/7.2/fpm/conf.d/99-overrides.ini
```

请注意，我们更改了容器名称、数据库和 MySQL 用户：

+   `container_name: chapter-09-mysql`

+   `container_name: chapter-09-webserver`

+   `container_name: chapter-09-php-fpm`

+   `MYSQL_DATABASE=chapter-09`

+   `MYSQL_USER=chapter-09`

1.  将我们所做的更改添加到 Git 源代码控制中。打开您的终端窗口并输入以下命令：

```php
 git add .
 git commit -m "Initial commit chapter 09"
```

# 处理模型和类

由 Angular 开发者社区认为是良好实践（我们认为是必不可少的）的是创建类以将其用作模型。这些也被称为**领域模型**。

我们认为创建类来存储我们的模型是创建大型应用程序甚至小型应用程序的一个非常重要的资源。这有助于保持代码的组织性。

想象一下，如果我们的项目规模更大——如果所有数据都存储在普通对象中，那么新开发人员将很难找到数据存储的位置。

这也是使用类来存储我们的模型信息的一个很好的理由。

# 创建用户类模型

让我们首先创建一个类来存储我们的用户信息。按照惯例，我们将把这个文件命名为`user.ts`：

1.  打开您的终端窗口。

1.  转到`./Client/src/app`并输入以下命令：

```php
 ng g class pages/auth/user
```

1.  上一个命令将在`./app/pages/auth/auth.ts`中创建一个新文件。打开此文件并添加以下代码：

```php
 export  class  User {
        name?:  string;
        email?:  string;
        password?:  string;
        constructor() {}
 }
```

# 创建构建者类模型

现在，让我们为构建者创建模型，并更好地理解类作为模型的操作。在此之前，我们将观察当我们对`api/builders/1`端点进行 GET 请求时 API 的返回，如下面的屏幕截图所示：

![](img/08cc1ca5-aaed-4f32-b5df-38b529601d7d.png)构建者详细 JSON 结果

在先前的屏幕截图中，我们已经在构建者详细请求中包含了自行车信息。让我们看看如何使用`builders`类来实现这一点：

1.  仍然在您的终端中，输入以下命令：

```php
 ng g class pages/builders/builder
```

1.  上一个命令将在`./app/pages/builders/builder.ts`中创建一个新文件。打开此文件并添加以下代码：

```php
 import { Bike } from  '../bikes/bike';

 export  class  Builder {
        id:  number;
        name:  string;
        description:  string;
        location:  string;
        bike?:  Bike;

        constructor() {}
 }
```

请注意，在先前的代码中，我们添加了一个可选的`bike`属性，并将其类型设置为`Bike`模型。

# 创建 Bike 类模型

现在，是时候创建自行车模型类了，但首先让我们检查一下我们在自行车详细端点`api/bikes/2`上的 JSON 格式，如下面的屏幕截图所示：

![](img/b1af74f8-64cb-4259-aae0-7dee28e2c45f.png)自行车详细 JSON 结果

在这里，我们可以注意到`bike-detail`结果指向`garages`、`items`、`builder`、`user`和`ratings`。对于我们正在构建的示例应用程序，我们将只使用构建者和用户模型。不用担心其他的；我们在这里使用的示例足以理解模型领域：

1.  仍然在您的终端中，输入以下命令：

```php
 ng g class pages/bikes/bike
```

1.  上一个命令将在`./app/pages/bikes/bike.ts`中创建一个新的文件。打开这个文件并添加以下代码：

```php
 import { User } from  './../auth/user';
 import { Builder } from  '../builders/builder';

 export  class  Bike {
        id:  number;
        make:  string;
        model:  string;
        year:  string;
        mods:  string;
        picture:  string;
        user_id:  number;
        builder_id:  number;
        average_rating?: number;
        user?:  User;
        builder?:  Builder;
        items?:  any;
        ratings?:  any;

        constructor() {}
 }
```

请注意，在上一个代码中，我们使用了上一个截图中的所有属性，包括`items`和`ratings`，作为类型为`any`的可选属性，因为我们没有为这些属性创建模型。

# 使用新的 HttpClient 处理 XHR 请求

如今，绝大多数 Web 应用程序都使用`XMLHttpRequest`（XHR）请求，而使用 Angular 制作的应用程序也不例外。为此，我们有`HTTPClient`模块取代了以前版本中的旧 HTTP 模块。

在这个会话中，我们将了解如何在我们的 Angular 服务中使用 XHR 请求。

强烈建议您使用 Angular 服务来处理这种类型的请求，以便组件的代码更有组织性和易于维护。

您可以在[`developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest`](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest)上阅读更多关于 XHR 请求的信息。

# 创建认证服务

让我们创建一个将存储我们认证模块所需代码的文件：

1.  仍然在您的终端中，输入以下命令：

```php
 ng g service pages/auth/_services/auth
```

上一个命令将在`./app/pages/auth/_services/auth.service.ts`中创建一个新的文件夹和文件。现在，让我们添加一些代码。

1.  打开`./app/pages/auth/_services/auth.service.ts`并在文件顶部添加以下导入：

```php
 import { HttpClient, HttpParams, HttpErrorResponse } from  '@angular/common/http';
 import { HttpHeaders } from  '@angular/common/http';
 import { Router } from  '@angular/router';
 import { Observable, throwError } from  'rxjs';
 import { catchError, map, tap } from  'rxjs/operators';

 // App imports
 import { environment } from  './../../../environments/environment';
 import { User } from  './user';
```

现在，我们将使用`HttpHeaders`来设置我们的 XHR 请求的内容类型。

1.  在导入文件后添加以下代码：

```php
 // Setup headers
 const httpOptions  = {
```

```php
        headers: new  HttpHeaders({
                'Content-Type': 'application/json'
        })
 };
```

上一个代码示例将使用`HttpHeaders`为我们的请求添加一个新的头。

1.  在`AuthService`类内部，添加以下代码：

```php
 public  currentUser:  User;
 private  readonly  apiUrl  =  environment.apiUrl;
 private  registerUrl  =  this.apiUrl  +  '/register';
 private  loginUrl  =  this.apiUrl  +  '/login';
```

您一定会问为什么`currentUser`是`public`而其他的是`private`，对吧？

嗯，`currentUser`属性是`public`的，因为我们将在其他文件中访问它，正如我们将在本节后面看到的那样。因此，其他属性将不会在`AuthService`之外可用。

1.  现在，让我们创建我们的`constructor`函数。在`constructor`函数内部，添加以下代码：

```php
 private  http:  HttpClient, private  router:  Router
```

1.  `constructor`类将如下代码所示：

```php
     constructor(
                private  http:  HttpClient,
                private  router:  Router) {}
```

请注意，我们在这里使用了`HttpClient`和`Router`模块，所以现在是时候编写我们的函数来看看这个模块的实际应用了。

# 创建注册函数

让我们创建`Register`函数。在`constructor`函数之后，添加以下代码：

```php
     onRegister(user: User): Observable<User> {
                const request  =  JSON.stringify(
                        { name: user.name, email: user.email, password:
                 user.password }
                );
                return  this.http.post(this.registerUrl, request,
                httpOptions)
                .pipe(
                        map((response:  User) => {
                                // Receive jwt token in the response
                                const  token: string  =
                                response['access_token'];
                                // If we have a token, proceed
                                if (token) {
                                        this.setToken(token);
                                        this.getUser().subscribe();
                                }
                                return  response;
                        }),
                catchError(error  =>  this.handleError(error))
                );
        }
```

请注意，我们在这里使用了**Reactive Extensions Library for JavaScript**（**RxJS**）中包含的`pipe()`、`map()`和`catchError()`函数。

在使用 RxJS 库之前，在 AngularJS 应用程序中使用一个叫做 Lodash 的库来操作结果是非常常见的。

您可以在官方文档链接[`rxjs-dev.firebaseapp.com/api`](https://rxjs-dev.firebaseapp.com/api)中阅读更多关于 RxJS 库的信息。

我们使用`pipe()`函数，它允许我们链接其他函数，当我们使用可观察对象时，这是非常有趣的。在`pipe()`函数内部，这正是我们在`map()`和`catchError()`函数中所做的。

此外，我们还使用了三个名为`setToken()`、`getUser()`和`handleError()`的本地函数，我们稍后会看到它们。

请记住，函数名非常重要。尽量使用像我们在`setToken`和`getUser`中所做的那样自解释的名称。

# 创建登录函数

`Login`函数的结构几乎与`Register`函数相同。不同之处在于我们只是将电子邮件地址和密码发送到服务器。

在`onRegister()`函数之后添加以下代码：

```php
     onLogin(user: User): Observable<User> {
                const request  =  JSON.stringify(
                        { email: user.email, password: user.password }
                );
                return  this.http.post(this.registerUrl, request,
                httpOptions)
                .pipe(
                        map((response:  User) => {
                                // Receive jwt token in the response
                                const  token: string  = 
                                response['access_token'];
                                // If we have a token, proceed
                                if (token) {
                                        this.setToken(token);
                                        this.getUser().subscribe();
                                }
                                return  response;
                        }),
                catchError(error  =>  this.handleError(error))
                );
        }
```

请注意，我们使用`setToken()`函数保存用户令牌，并使用`getUser()`函数获取用户的详细信息。我们将在本节后面详细介绍这一点。

# 创建注销函数

对于注销函数，我们将使用不同的方法。我们将使用`tap()`操作符，而不是使用`map()`操作符。

在`onLogin()`函数之后添加以下代码：

```php
onLogout():  Observable<User> {
        return  this.http.post(this.apiUrl  +  '/logout',
          httpOptions).pipe(
                tap(
                        () => {
                                localStorage.removeItem('token');
                                this.router.navigate(['/']);
                                }
                        )
                );
}
```

在上述代码中，我们只是从`localStorage`中删除令牌，并将用户重定向到主页。现在，是时候创建处理数据的本地函数了。

# 创建设置令牌和获取令牌函数

我们几乎已经完成了我们的身份验证服务，但我们仍然需要创建一些辅助函数，这些函数将在其他应用程序块中使用。

让我们创建处理用户令牌的函数。重新创建我们在 Laravel 后端中使用的`jwt-auth`库来进行调用，用于验证我们的用户。

在本示例中，我们使用`localStorage`来存储用户的令牌。因此，让我们创建两个非常简单的函数来写入和检索此令牌。

在`logout()`函数之后，添加以下代码块：

```php
setToken(token:  string):  void {
        return  localStorage.setItem('token', token );
}

getToken():  string {
        return  localStorage.getItem('token');
}
```

# 创建获取用户函数

现在，我们将看到如何获取已登录用户的信息。请记住，我们的 API 有一个端点，根据认证令牌为我们提供已登录用户的信息。

让我们看看如何以简单的方式做到这一点。

在`getToken()`函数之后添加以下代码：

```php
getUser():  Observable<User> {
        return  this.http.get(this.apiUrl  +  '/me').pipe(
                tap(
                        (user: User) => {
                                this.currentUser  =  user;
                        }
                )
        );
}
```

上述代码从 API 接收用户信息，并将其应用于`currentUser`属性。

# 创建 isAuthenticated 函数

现在，我们将创建一个额外的函数。这个函数将帮助我们确定用户是否已登录。

在`getUser()`函数之后添加以下代码：

```php
  isAuthenticated():  boolean { // get the token
  const  token:  string  =  this.getToken();
  if (token) {
  return  true;
 }  return  false;
 }
```

现在，我们可以在任何地方使用`AuthService.currentUser`和`AuthService.isAuthenticated`方法来使用这些信息。

# 创建 handleError 函数

您应该已经注意到`login()`和`register()`函数具有指向另一个名为`handleError`的函数的`catchError`函数。此刻，我们将创建这个函数，负责显示我们的请求可能出现的错误。

在**`getUser()`**函数之后添加以下代码：

```php
private  handleError(error:  HttpErrorResponse) {
        if (error.error  instanceof  ErrorEvent) {
                // A client-side error.
                console.error('An error occurred:',
                error.error.message);
        } else {
                // The backend error.
                return  throwError(error);
        }
        // return a custom error message
        return  throwError('Ohps something wrong happen here; please try again later.');
}
```

我们将错误消息记录到浏览器控制台，仅供本示例使用。

# 创建自行车服务

现在，我们将创建一个服务来保存所有自行车操作。请记住，对于自行车和建造者，我们的服务必须具有用于列出、详细信息、创建、更新和删除的方法：

1.  仍然在您的终端中，键入以下命令：

```php
 ng g service pages/bikes/_services/bike
```

上述命令将在`./app/pages/bikes/_services/bike.service.ts`中创建一个新的文件夹和文件。现在，让我们添加一些代码片段。

1.  打开`./app/pages/bikes/_services/bike.service.ts`并将以下导入添加到文件顶部：

```php
 import { Injectable } from  '@angular/core';
 import { HttpClient, HttpParams, HttpErrorResponse } from  '@angular/common/http';
 import { HttpHeaders } from  '@angular/common/http';
 import { Observable, throwError } from  'rxjs';
 import { catchError } from  'rxjs/operators';

 // App import
 import { environment } from  '../../../../environments/environment';
 import { Bike } from  '../bike';
```

1.  在`bikesService`类中，添加以下属性：

```php
 private  readonly  apiUrl  =  environment.apiUrl;
 private  bikesUrl  =  this.apiUrl  +  '/bikes';
```

1.  现在，让我们创建我们的`constructor`函数。在`constructor`函数中，添加以下代码：

```php
 constructor(private  http:  HttpClient) {}
```

现在，我们准备创建我们的自行车服务的函数。

# 创建 CRUD 函数

正如我们之前提到的，**CRUD**代表`Create`，`Read`，`Update`和`Delete`。我们将一次性添加操作的代码，然后进行必要的注释。

在`constructor()`函数之后添加以下代码块：

```php
 /** GET bikes from bikes endpoint */
 getBikes ():  Observable<Bike[]> {
        return  this.http.get<Bike[]>(this.bikesUrl)
        .pipe(
                catchError(error  =>  this.handleError(error))
        );
 }

 /** GET bike detail from bike-detail endpoint */
 getBikeDetail (id:  number):  Observable<Bike[]> {
        return  this.http.get<Bike[]>(this.bikesUrl  +  `/${id}`)
        .pipe(
                catchError(error  =>  this.handleError(error))
        );
 }

 /** POST bike to bikes endpoint */
 addBike (bike:  Bike):  Observable<Bike> {
        return  this.http.post<Bike>(this.bikesUrl, bike)
        .pipe(
                catchError(error  =>  this.handleError(error))
        );
 }

 /** PUT bike to bikes endpoint */
 updateBike (bike:  Bike, id:  number):  Observable<Bike> {
        return  this.http.put<Bike>(this.bikesUrl  +  `/${id}`, bike)
        .pipe(
                catchError(error  =>  this.handleError(error))
        );
 }

 /** DELETE bike bike endpoint */
 deleteBike (id:  number):  Observable<Bike[]> {
        return  this.http.delete<Bike[]>(this.bikesUrl  +  `/${id}`)
        .pipe(
                catchError(error  =>  this.handleError(error))
        );
 }

 /** Vote on bike */
 voteOnBike (vote:  any, bike:  number):  Observable<any> {
        const  rating  =  vote;
        return  this.http.post(this.bikesUrl  +  `/${bike}/ratings`, {rating})
        .pipe(
                catchError(error  =>  this.handleError(error))
        );
 }
```

上述代码与我们在身份验证服务中使用的内容没有特别不同，除了使用模板字符串：

```php
this.bikesUrl  +  `/${id}`
this.bikesUrl  +  `/${bike}/ratings`, {rating}
```

这些由反引号（`` ` ``）字符包围，而不是单引号或双引号，以及以美元符号开头的表达式。

# 创建 `voteOnBike` 函数

我们的服务仍然有一个功能，我们将用它来发送用户对特定自行车的投票。请记住，每当需要使用 `HTTPClient` 模块时，请在服务中执行此操作。这在 Angular 开发中被认为是一个良好的实践。

在 `deleteBike()` 函数之后添加以下代码：

```php
     /** Vote on bike */
        voteOnBike (vote:  number, bike:  number):  Observable<any> {
                const  rating  =  vote;
                return  this.http.post(this.bikesUrl  + 
                `/${bike}/ratings`, {rating})
                .pipe(
                        catchError(error  =>  this.handleError(error))
                );
        }

```

# 创建 `handleError` 函数

现在，让我们为自行车服务添加错误处理。在 `voteOnBike()` 函数之后添加以下代码：

```php

     /** Error handler */
        private  handleError(error:  HttpErrorResponse) {
                if (error.error  instanceof  ErrorEvent) {
                        // A client-side error.
                        console.error('An error occurred:', 
                error.error.message);
                } else {
                        // The backend error.
                        return  throwError(error);
                }
                // return a custom error message
                return  throwError('Something bad happened; please try
                again later.');
        }
```

正如我们所看到的，在自行车服务中的 `handleError()` 函数与认证服务相同，并且在构建者服务上也是一样的。每当需要多次编写相同的代码时，强烈建议使用服务来避免代码的重复。

之后，我们将创建一个解决这个问题的服务，但现在我们将创建构建者服务。

# 创建构建者服务

现在，我们将创建 `builder` 服务，其中包括 `Create`、`Read`、`Update` 和 `Delete` 方法：

1.  仍然在你的终端中，输入以下命令：

```php

ng g service pages/builders/_services/builder

```

前述命令将在 `./app/pages/builders/_services/builder.service.ts` 中创建一个新的文件夹和文件。现在，让我们添加一些代码片段。

1.  打开 `./app/pages/builders/_services/builder.service.ts`，并将其代码替换为以下代码块：

```php

     import { Injectable } from  '@angular/core';

        import { HttpClient, HttpParams, HttpErrorResponse } from
       '@angular/common/http';
        import { HttpHeaders } from  '@angular/common/http';
        import { Observable, throwError } from  'rxjs';
        import { catchError } from  'rxjs/operators';

        // App import
        import { environment } from
        '../../../../environments/environment';
        import { Builder } from  '../builder';
        @Injectable({
                providedIn: 'root'
        })

        export  class  BuildersService {
                private  readonly  apiUrl  =  environment.apiUrl;
                private  buildersUrl  =  this.apiUrl  +
         '/builders';
                
                constructor(private  http:  HttpClient) { }

                /** GET builders from builders endpoint */
                getBuilders ():  Observable<Builder[]> {
                        return  this.http.get<Builder[]>
      (this.buildersUrl)
                                .pipe(
                                        catchError(error  =>
       this.handleError(error))
                                );
                }

                /** GET builder detail from builder-detail endpoint
        */
                getBuilderDetail (id:  number):
        Observable<Builder[]> {
                return  this.http.get<Builder[]>(this.buildersUrl  +  
        `/${id}`)
                        .pipe(
                                catchError(error  => 
        this.handleError(error))
                        );
                }

                /** POST builder to builders endpoint */
                addBuilder (builder:  Builder):  Observable<Builder> 
         {
                        return  this.http.post<Builder>
           (this.buildersUrl, builder)
                                .pipe(
                                        catchError(error  =>
           this.handleError(error))
                                );
                }

                /** PUT builder to builders endpoint */
                updateBuilder (builder:  Builder, id:  number):
           Observable<Builder> {
                        return  this.http.put<Builder>
           (this.buildersUrl  +  `/${id}`, builder)
                                .pipe(
                                        catchError(error  =>
            this.handleError(error))
                                );
                }

                /** DELETE builder builder endpoint */
                deleteBuilder (id:  number):  Observable<Builder[]>
            {
                        return  this.http.delete<Builder[]>
            (this.buildersUrl  +  `/${id}`)
                                .pipe(
                                        catchError(error  =>
            this.handleError(error))
                                );
                }

                /** Error handler */
                private  handleError(error:  HttpErrorResponse) {
                        if (error.error  instanceof  ErrorEvent) {
                                // A client-side error.
                                console.error('An error occurred:',
             error.error.message);
                        } else {
                                // The backend error.
                                return  throwError(error);
                        }
                        // return a custom error message
                        return  throwError('Something bad happened;
             please try again later.');
                }
        }

```

前述代码与自行车服务几乎相同，我们可以注意到最后一个函数是 `handleError()` 函数，因此现在是学习如何创建错误服务的时候了。

# 处理 HttpErrorHandler 服务

如前所述，在现代 Web 应用程序中重复代码并不是一个好的实践，因此我们可以使用许多资源来避免这种实践。在 Angular 开发中，我们可以使用共享服务在一个地方处理应用程序错误。

# 创建错误处理服务

如本章早些时候提到的，让我们创建我们的错误处理程序服务：

1.  在 `./Client/src/app` 内打开你的终端窗口，然后输入以下命令：

```php

ng g service pages/shared/_services/httpHandleError

```

上述命令将在 `pages/shared` 文件夹内创建一个名为 `_services` 的新文件夹，原因很简单：我们将在 `bikes`、`builders` 和 `auth` 模块中创建的所有服务之间共享此服务。上述命令还创建了一个名为 `http-handle-error.service.ts` 的文件。

1.  打开 `./Client/src/app/shared/_services/http-handle-error.service.ts` 并添加以下导入：

```php

import { HttpErrorResponse } from  '@angular/common/http';
import { Observable, of } from  'rxjs';

```

1.  让我们为我们的错误创建一个 Angular `type`。在导入之后添加以下代码：

```php

export  type  HandleError  =
        <T> (operation?:  string, result?:  T) => (error:  HttpErrorResponse) =>  Observable<T>;

```

上述代码创建了一个名为 `HandleError` 的新类型，并且我们将在接下来的行中使用它。

请记住，Angular 有许多类型，如数组、空、任何更多。我们在第三章 *理解 Angular 6 的核心概念* 中已经看到了这一点。

1.  让我们添加错误函数。在 `constructor()` 函数之后添加以下代码块：

```php

     /** Pass the service name to map errors */
        createHandleError  = (serviceName  =  '') => <T>
                (operation  =  'operation', result  = {} as  T) =>
        this.handleError(serviceName, operation, result)
        handleError<T> (serviceName  =  '', operation  =
       'operation', result  = {} as  T) {
                return (response:  HttpErrorResponse):
                Observable<T> => {
                        // Optionally send the error to a third part
                      error logging service
                        console.error(response);
                        
                        // Show a simple alert if error
                        const  message  = (response.error
                        instanceof  ErrorEvent) ?
                        response.error.message  :
                        `server returned code ${response.status}
                        with body "${response.error.error}"`;
                        
                        // We are using alert just for example, on
                        real world avoid this pratice
                        alert(message);
                        
                        // Keep running and returning a safe result.
                        return  of( result );
                };
        }
```

上面的代码创建了一个名为`handleError`的函数，接收三个参数——`serviceName`、`operation`和`result`——并返回一个名为`HandleError`的可观察类型。

我们还使用基本内置的 JavaScript 函数来向用户显示警报，如果出现错误，则使用`console.log()`函数显示所有 HTTP 响应。

如今，使用付费日志记录服务来监视 Web 应用程序并向用户发出静默错误已经非常普遍。

一些私人服务，例如 Rollbar、TrackJS、Bugsnag 和 Sentry。它们都提供了一个强大的 API，用于在生产模式下跟踪错误，并将其发送到一个易于使用的仪表板面板，而不会引起应用程序用户的警报或搜索应用程序日志。

我们还建议，对于测试版和内部测试应用程序，可以在[`www.bugsnag.com/platforms/javascript/`](https://www.bugsnag.com/platforms/javascript/)上免费注册一个 bugsnag 账户。

# 将 HttpErrorHandler 导入到 app.module.ts

现在，我们需要将我们的服务添加到应用程序的中央模块中。请记住，我们正在使用一个名为`shared`的目录；将我们的服务放在`app.module.ts`文件中的适当位置：

1.  打开`./Client/src/app/app.module.ts`文件，并在`NavComponent`导入之后添加以下代码：

```php

import { HttpErrorHandler } from  './shared/_services/http-handle-error.service';

```

1.  在`./Client/src/app/app.module.ts`中，将`HttpErrorHandler`属性添加到`providers`数组中的`Title`属性之后：

```php

 providers: 
        Title,
        HttpErrorHandler,

```

在这一步结束时，我们的应用程序中有以下目录结构：

![共享服务文件夹

# 重构构建者服务

现在我们已经创建了错误处理服务，我们需要重构我们的构建者和自行车服务以使用新的错误处理。

打开`./app/pages/builders/_services/builder.service.ts`，并用以下代码替换其内容：

```php

     import { Injectable } from  '@angular/core';
        import { HttpClient, HttpParams, HttpErrorResponse } from  
        '@angular/common/http';
        import { HttpHeaders } from  '@angular/common/http';
        import { Observable, throwError } from  'rxjs';
        import { catchError } from  'rxjs/operators';
        // App import
        import { environment } from
        '../../../../environments/environment';
        import { Builder } from  '../builder';
        import { HttpErrorHandler, HandleError } from
        '../../../shared/_services/http-handle-error.service';

        @Injectable({
                providedIn: 'root'
        })

        export  class  BuildersService {
                private  readonly  apiUrl  =  environment.apiUrl;
                private  buildersUrl  =  this.apiUrl  +
                '/builders';
                private  handleError:  HandleError;

                constructor(
                        private  http:  HttpClient,
                        httpErrorHandler:  HttpErrorHandler ) {
                        this.handleError  =
  httpErrorHandler.createHandleError('BuildersService');
                }
                
                /** GET builders from builders endpoint */
                getBuilders ():  Observable<Builder[]> {
                        return  this.http.get<Builder[]>
                (this.buildersUrl)
                                .pipe(
                                         
                catchError(this.handleError('getBuilders', []))
                                );
                }

                /** GET builder detail from builder-detail endpoint
                 */
                getBuilderDetail (id:  number): 
                Observable<Builder[]> {
                        return  this.http.get<Builder[]>
                (this.buildersUrl  +  `/${id}`)
                                .pipe(
                                 
                catchError(this.handleError('getBuilderDetail', []))
                                );
                }

                /** POST builder to builders endpoint */
                addBuilder (builder:  Builder):  Observable<Builder> {
                        return  this.http.post<Builder> 
               (this.buildersUrl, builder)
                                .pipe(
                                       
            catchError(this.handleError('addBuilder', builder))
                                );
                }

                /** PUT builder to builders endpoint */
                updateBuilder (builder:  Builder, id:  number):
                Observable<Builder> {
                        return  this.http.put<Builder>(this.buildersUrl
           +  `/${id}`, builder).pipe(                            
             catchError(this.handleError('updateBuilder', builder))
                                );
                }

                /** DELETE builder builder endpoint */
                deleteBuilder (id:  number):  Observable<Builder[]> {
                        return  this.http.delete<Builder[]>
                (this.buildersUrl  +  `/${id}`)
                                .pipe(
                          catchError(this.handleError('deleteBuilder'))
                                );
                }
        }
```

在上面的代码中，我们替换了本地错误函数以使用新的错误服务。我们添加了一个名为`handleError`的新属性，并创建了一个名为`BuildersService`的新处理程序，代码如下：

```php

this.handleError = httpErrorHandler.createHandleError ('BuildersService');

```

每个处理程序都接收`serviceName`，如`getBuilders`、`getBuilderDetail`、`addBuilder`、`updateBuilder`和`deleteBuilder`。

现在，我们将为自行车服务执行相同的操作。

# 重构自行车服务

现在，让我们为自行车服务添加新的错误处理。

打开`./app/pages/bikes/_services/bike.service.ts`，并用以下代码替换其内容：

```php

     import { Injectable } from  '@angular/core';
        import { HttpClient, HttpParams, HttpErrorResponse } from  '@angular/common/http';
        import { HttpHeaders } from  '@angular/common/http';
        import { Observable, throwError } from  'rxjs';
        import { catchError } from  'rxjs/operators';
        // App import
        import { environment } from  '../../../../environments/environment';
        import { Bike } from  '../bike';
        import { HttpErrorHandler, HandleError } from  '../../../shared/_services/http-handle-error.service';

        @Injectable({
                providedIn: 'root'
        })

        export  class  BikesService {
                private  readonly  apiUrl  =  environment.apiUrl;
                private  bikesUrl  =  this.apiUrl  +  '/bikes';
                private  handleError:  HandleError;
                
                constructor(
                        private  http:  HttpClient,
                        httpErrorHandler:  HttpErrorHandler ) {
                        this.handleError  = 
                httpErrorHandler.createHandleError('BikesService');
                }

                /** GET bikes from bikes endpoint */
                getBikes ():  Observable<Bike[]> {
                        return  this.http.get<Bike[]>(this.bikesUrl)
                                .pipe(
                   
                 catchError(this.handleError('getBikes', []))
                                );
                }

                /** GET bike detail from bike-detail endpoint */
                getBikeDetail (id:  number):  Observable<Bike[]> {
                        return  this.http.get<Bike[]>(this.bikesUrl  +  
                `/${id}`)
                                .pipe(
                                         
                catchError(this.handleError('getBikeDetail', []))
                                );
                }

                /** POST bike to bikes endpoint */
                addBike (bike:  Bike):  Observable<Bike> {
                        return  this.http.post<Bike>(this.bikesUrl, 
                bike)
                                .pipe(
                                         
               catchError(this.handleError('addBike', bike))
                                );
                }

                /** PUT bike to bikes endpoint */
                updateBike (bike:  Bike, id:  number):  
                Observable<Bike> {
                        return  this.http.put<Bike>(this.bikesUrl  +  
                `/${id}`, bike)
                                .pipe(
                                        
                catchError(this.handleError('updateBike', bike))
                                );
                }

                /** DELETE bike bike endpoint */
                deleteBike (id:  number):  Observable<Bike[]> {
                        return  this.http.delete<Bike[]>(this.bikesUrl  
                +  `/${id}`)
                                .pipe(
                                        
                catchError(this.handleError('deleteBike'))
                                );
                }
                
                /** Vote on bike */
                voteOnBike (vote:  number, bike:  number):  
                Observable<any> {
                        const  rating  =  vote;
                        return  this.http.post(this.bikesUrl  +  
                `/${bike}/ratings`, {rating})
                                .pipe(
                                        
                 catchError(this.handleError('voteOnBike', []))
                                );
                        }
                }
```

在上面的代码中，我们与构建者服务中所做的一样，并添加了每个处理程序，其中`serviceName`为`getBikes`、`getBikeDetail`、`addBike`、`updateBike`和`deleteBike`。

# 如何使用授权头

当我们谈论头部授权时，基本上是在讨论对应用程序头部进行一些修改以发送某种授权。在我们的情况下，我们具体讨论的是由我们的 API 后端生成的授权令牌。

最好的方法是使用 Angular 拦截器。拦截器正如其名称所示，允许我们简单地拦截和配置请求，然后再将其发送到服务器。

这使我们能够做很多事情。其中一个示例是在任何请求上配置令牌验证，或者突然添加我们的应用程序可能需要的自定义标头，直到我们在完成请求之前处理答案。

当 JWT 令牌被发送到后端时，请记住我们在我们的 Laravel API 上使用了 `jwt-auth` 库：它预期在 HTTP 请求的授权标头中。

在 Angular 中添加授权标头到 HTTP 请求的最常见方法是创建一个拦截器类，并通过将 JWT（或其他形式的访问令牌）作为授权标头附加到请求中来让拦截器对请求进行修改，就像我们之前解释的那样。

# 创建一个 HTTP 拦截器。

让我们看看如何使用 Angular 的 `HttpInterceptor` 接口来进行身份验证的 HTTP 请求。

当我们在 Angular 应用中处理身份验证时，大多数情况下，最好将所需的一切都放在一个专用的服务中，就像我们之前做的那样。

任何身份验证服务都应该有几个基本方法，允许用户登录和退出。它还应该包括一种获取 JSON Web Token 并将其放入 `localStorage` 中的方法（就像我们之前所做的那样），在客户端，并确定用户是否经过身份验证的方式，我们的情况下，使用 `auth.service.ts` 上的 `isAuthenticated()` 函数。

因此，让我们创建 HTTP 拦截器：

1.  在你的终端窗口中打开 `./Client/src/app`，并输入以下命令：

```php

ng g service shared/_services/http-interceptor

```

上一条命令将生成以下文件：`./Client/src/app/shared/_services/app-http-interceptor.service.ts`。再次，我们正在创建一个文件在我们的 `shared` 目录中，因为我们可以在应用程序中的任何地方使用这个服务。

1.  打开 `./Client/src/app/shared/_services/app-http-interceptor.service.ts` 文件，并添加以下代码：

```php

     import { Injectable, Injector } from  '@angular/core';
        import { HttpEvent, HttpHeaders, HttpInterceptor, HttpHandler, HttpRequest, HttpErrorResponse, HttpResponse } from  '@angular/common/http';
        import { Observable } from  'rxjs';
        import { catchError, map, tap } from  'rxjs/operators';
        import { Router } from  '@angular/router';
        // App import
        import { AuthService } from 
  '../../pages/auth/_services/auth.service';
        
        @Injectable()
        export  class  AppHttpInterceptorService  implements
        HttpInterceptor {
          
        constructor(public  auth:  AuthService, private  router:
        Router ) { }

        intercept(req:  HttpRequest<any>, next:  HttpHandler):
        Observable<HttpEvent<any>> {
                console.log('interceptor running');
                
                // Get the token from auth service.
                const  authToken  =  this.auth.getToken();
                if (authToken) {
                        // Clone the request to add the new header.
                        const  authReq  =  req.clone(
                                { headers:
         req.headers.set('Authorization', `Bearer ${authToken}`)}
                        );                      
                        console.log('interceptor running with new
         headers');
                        
                        // send the newly created request
                        return  next.handle(authReq).pipe(
                                tap((event:  HttpEvent<any>) => {
                                        if (event instanceof
          HttpResponse) {
                                        // Response wiht
          HttpResponse type
                                        console.log('TAP function',
          event);
                                        }
                                }, (err:  any) => {
                                console.log(err);
                                if (err  instanceof 
          HttpErrorResponse) {
                                        if (err.status ===  401) {
                                      
          localStorage.removeItem('token');
                                 
          this.router.navigate(['/']);
                                        }
                                }
                                })
                        );
                } else {
                        console.log('interceptor without changes');
                        return  next.handle(req);
                }
        }
```

1.  在前面的代码中，首先我们检查 `localStorage` 中是否有一个令牌，使用 `AuthService` 的 `this.auth.getToken();` 函数。所以，如果我们有一个令牌，我们添加它作为一个新的标头，使用以下方式：

```php

     const  authReq  =  req.clone(
                { headers: req.headers.set('Authorization', `Bearer ${authToken}`)}
                );
```

1.  如果令牌无效，或者 API 返回了 401 错误，我们将使用以下方式将用户发送到主路由：

```php

this.router.navigate(['/']);

```

# 将 AppHttpInterceptorService 添加到主模块中。

现在我们已经配置好了拦截器，并准备好使用了，我们需要将其添加到主应用程序模块中：

1.  打开 `./Client/src/app/app.module.ts` 文件，并在 `HttpErrorHandler` 导入之后添加以下导入：

```php

import { AppHttpInterceptorService } from  './shared/_services/app-http-interceptor.service';

```

1.  在 `providers` 数组中添加以下代码，在 `HttpErrorHandler` 属性之后：

```php

{
 {
        provide: HTTP_INTERCEPTORS,
        useClass: AppHttpInterceptorService ,
        multi: true
 }

```

1.  在前面的步骤结束时，我们的主应用程序模块将包含以下代码：

```php

     import { BrowserModule, Title } from  '@angular/platform-browser';
        import { NgModule } from  '@angular/core';
        import { HttpClientModule, HTTP_INTERCEPTORS } from  '@angular/common/http';
        import { AppRoutingModule } from  './app-routing.module';
        import { ServiceWorkerModule } from  '@angular/service-worker';
        
        // Application modules
        import { AppComponent } from  './app.component';
        import { environment } from  '../environments/environment';
        import { HomeModule } from  './pages/home/home.module';
        import { BikesModule } from  './pages/bikes/bikes.module';
        import { BuildersModule } from  './pages/builders/builders.module';
        import { AuthModule } from  './pages/auth/auth.module';
        import { NavComponent } from  './layout/nav/nav.component';
        import { HttpErrorHandler } from  './shared/_services/http-handle-error.service';
        import { AppHttpInterceptorService } from  './shared/_services/app-http-interceptor.service';
          
        @NgModule({
        declarations: [
                AppComponent,
                NavComponent
        ],
        imports: [
                BrowserModule,
                AppRoutingModule,
                HttpClientModule,
                HomeModule,
                BikesModule,
                BuildersModule,
                AuthModule,
                ServiceWorkerModule.register('/ngsw-worker.js', { enabled: environment.production })
        ],
        providers: [
                Title,
                HttpErrorHandler,
                {
                provide: HTTP_INTERCEPTORS,
                useClass: AppHttpInterceptorService ,
                multi: true
                }
        ],
        bootstrap: [AppComponent]
        })
        export  class  AppModule { }
```

请注意，我们将 Angular 导入与应用程序导入分开。这是一个好的做法，有助于保持代码的组织。

恭喜！现在，我们可以拦截应用程序中的每个请求。

# 如何使用路由守卫保护应用程序路由

在本节中，我们将讨论 Angular 框架的另一个强大功能。我们称之为守卫，甚至更好地称之为路由守卫。

它在 Angular CLI 中可用，正如我们将在下面的代码行中看到的那样，但首先让我们更深入地了解一下守卫。

当构建现代 Web 应用程序时，保护路由是一项非常常见的任务，因为我们希望防止用户访问他们不被允许访问的区域，在我们的情况下是自行车的详细信息。请记住，我们在`./Server/app/Http/Controllers/API/BikeController.php`中定义了对自行车详细信息的访问：

```php

     /**
        * Protect update and delete methods, only for authenticated
        users.
        *
        * @return  Unauthorized
        */
        public  function  __construct()
        {
                $this->middleware('auth:api')->except(['index']);
        }
```

前面的代码表示只有索引路由不应受到保护。

我们可以使用四种不同的守卫类型来保护我们的路由：

+   `CanActivate`：选择是否可以激活路由

+   `CanActivateChild`：选择是否可以激活路由的子路由

+   `CanDeactivate`：选择是否可以停用路由

+   `CanLoad`：选择是否可以延迟加载模块

在下一个示例中，我们将使用`CanActivate`功能。

# 创建自行车详细信息的路由守卫

守卫是作为服务实现的，因此我们通常使用 Angular CLI 创建一个守卫类：

1.  打开您的终端窗口，并输入以下命令：

```php

ng g guard pages/auth/_guards/auth

```

前面的代码将生成以下文件：`./Client/src/app/pages/auth/_guards/auth.guard.ts`。

1.  打开`./Client/src/app/pages/auth/_guards/auth.guard.ts`文件，并在 observable 导入之后添加以下导入：

```php

import { AuthService } from  '../_services/auth.service';

```

1.  现在，让我们在`constructor()`函数内添加`Router`和`AuthService`，如下所示的代码中：

```php

 constructor(
        private  router:  Router,
        private  auth:  AuthService) {}

```

1.  在`return`属性之前，添加以下代码块到`canActivate()`函数内：

```php

 if (this.auth.isAuthenticated()) {
 // logged in so return true
        return  true;
 }
 // not logged in so redirect to login page with the return url
 this.router.navigate(['/login'], { queryParams: { returnUrl: state.url }});
```

在上面的代码中，我们使用`AuthService`中的`auth.isAuthenticated()`函数来检查用户是否已经认证。这意味着，如果用户未经身份验证/登录，我们将重定向他们到登录屏幕。

我们还使用`queryParams`和`returnUrl`函数将用户发送回他们来自的位置。

这意味着，如果用户点击查看自行车的详细信息，而他们没有登录到应用程序，他们将被重定向到登录屏幕。登录后，用户将被重定向到他们打算查看的自行车的详细信息。

最后一步是将`AuthGuard`添加到`bike-detail`路由。

1.  打开`./Client/src/app/bikes/bikes-routing.module.ts`，并在路由导入之后添加以下导入：

```php

import { AuthGuard } from '../auth/_guards/auth.guard';

```

1.  现在，在`bikeDetailComponent`之后添加`canActivate`属性，如下所示的代码中：

```php

 {
        path: ':id',
        component: BikeDetailComponent,
        canActivate: [AuthGuard]
 }
```

看！我们的`bike-detail`路由现在受到了保护。

# 总结

现在，我们离看到我们的应用程序处于工作状态非常接近。然而，我们仍然需要执行一些步骤，我们将在接下来的章节中进行讨论。

与此同时，我们已经学习了一些构建现代 Web 应用程序的重要要点，比如创建服务来处理 XHR 请求，学习如何保护我们的路由，以及创建路由拦截器和处理错误的服务。

在下一章中，我们将深入探讨如何在我们的组件中使用我们刚刚创建的服务，并且我们还将为我们的应用程序应用一个视觉层。