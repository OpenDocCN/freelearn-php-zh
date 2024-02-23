# 高效查询现代 SQL 数据库

现在，我们将学习如何使用现代 SQL 高效地查询 SQL 数据库。在本章中，我们将定义现代 SQL 是什么以及如何使用它。我们将从定义现代 SQL 的概念开始，了解它与传统 SQL 的区别，并描述许多其特点。因此，我们将了解如何将某些传统 SQL 查询转换为现代查询以及何时最好这样做。此外，通过这样做，我们将更好地了解现代 SQL 如何帮助我们以多种方式优化服务器性能。

因此，我们将涵盖以下内容：

+   了解现代 SQL 及其特点

+   学习如何使用`WITH`和`WITH RECURSIVE`、`CASE`、`OVER AND PARTITION BY`、`OVER AND ORDER BY`、GROUPING SETS、JSON 子句和函数、`FILTER`和`LATERAL`查询。

# 现代 SQL

现代 SQL 是什么，它与传统 SQL 有何不同？它的主要特点是什么？让我们从定义概念开始。

# 定义

正如 Markus Winand 在他的网站[`modern-sql.com`](https://modern-sql.com)上所述，现代 SQL 可以被定义为“*一种国际标准化、广泛可用且图灵完备的数据处理语言，支持关系和非关系数据模型*。”这个定义指的是 ISO 和 ANSI 组织多年来推广的一系列标准，并为 SQL 编程语言添加了新功能。自 SQL-92 以来，SQL 标准的许多新版本被采纳，并且这些标准引入了许多基于关系和非关系模型的新功能。以下是这些功能的简要列表，以及确认它们被采纳到 SQL 语言中的相应标准：

+   `WITH`和`WITH RECURSIVE`（SQL：1999）

+   `CASE`（SQL：1999 和 SQL：2003）

+   `OVER AND PARTITION BY`（SQL：2003 和 SQL：2011）

+   `OVER AND ORDER BY`（SQL：2003 和 SQL：2011）

+   GROUPING SETS（SQL：2011）

+   JSON 子句和函数（SQL：2016）

+   `FILTER`（SQL：2003）

+   `LATERAL`查询（SQL：1999）

值得注意的是，大多数这些功能直到最近才被大多数关系数据库管理系统（RDBMS）实现。大多数 RDBMS 仅为其用户提供了基于老化的 SQL-92 标准所推广的关系模型的传统 SQL 语言。直到最近几年，许多 RDBMS 才开始实现现代 SQL 功能。

此外，让我们提出警告：使用这些功能不会立即为您的数据库服务器带来巨大的性能提升。那么，在您的代码库中使用这些功能的意义是什么？目的是使您的代码库与未来的数据库引擎优化兼容，并避免大多数与慢查询执行相关的问题。

但在进一步了解新的 SQL 功能之前，我们将在我们的 Linux 中为 PHP 容器安装`phpMyAdmin`，以便以用户友好的方式查看我们查询的结果。为此，请在容器的 CLI 上输入以下命令：

```php
# rm /srv/www
# ln -s /srv/fasterweb/chapter_6 /srv/www
# cd /srv
# wget -O phpMyAdmin-4.7.7-all-languages.zip https://files.phpmyadmin.net/phpMyAdmin/4.7.7/phpMyAdmin-4.7.7-all-languages.zip
# unzip phpMyAdmin-4.7.7-all-languages.zip
# cp phpMyAdmin-4.7.7-all-languages/config.sample.inc.php phpMyAdmin-4.7.7-all-languages/config.inc.php
# sed -i "s/AllowNoPassword'] = false/AllowNoPassword'] = true/" phpMyAdmin-4.7.7-all-languages/config.inc.php
# cd fasterweb/chapter_6
# ln -s ../../phpMyAdmin-4.7.7-all-languages ./phpmyadmin  
```

这些命令应该可以让您通过 Web 界面访问数据库服务器，网址为`http://localhost:8181/phpmyadmin`。在您喜欢的浏览器中访问此地址时，您应该看到以下屏幕：

![](img/a0aac6ea-6f8d-4787-a740-eb10c29e4428.png)在 phpMyAdmin 的登录页面上输入用户名和密码

安装`phpMyAdmin`后，您可以使用用户名`root`和空密码登录到数据库服务器。

现在，让我们更详细地了解每一个新的 SQL 功能。

# WITH 和 WITH RECURSIVE

第一个功能是所谓的**公共表达式**（**CTE**）。CTE 是一个临时结果集，允许您多次将相同的数据连接到自身。有两种类型的 CTE：非递归（`WITH`）和递归（`WITH RECURSIVE`）。

非递归类型的 CTE 就像派生表一样，允许您从临时结果集中`SELECT`。一个简单的例子，使用一个虚构的员工表，将是：

```php
WITH accountants AS (
  SELECT id, first_name, last_name
  FROM staff
  WHERE dept = 'accounting'
)
SELECT id, first_name, last_name
FROM accountants;
```

递归类型的 CTE 由两部分组成。查询的第一部分称为 CTE 的锚成员。锚的结果集被认为是基本结果集（T[0]）。第二部分是递归成员，它将以 T[i]作为输入和 T[i+1]作为输出运行，直到返回一个空的结果集。查询的最终结果集将是递归结果集（T[n）和锚（T[0]）之间的`UNION ALL`。

为了更好地理解递归 CTE 以及它们可以有多么有用，让我们举个例子。但在开始之前，让我们先将以下表加载到测试数据库中。在容器的 CLI 中，输入此命令：

```php
# mysql -uroot test < /srv/www/employees.sql 
```

完成后，您可以通过使用`phpMyAdmin`打开数据库来确保一切都加载正确，如下所示：

![](img/5f8e9316-5aeb-4914-9e5d-428303e40b4c.png)测试数据库中员工表中找到的所有行

为了更好地理解 CTE，我们将从使用具有多个连接的基本查询开始，以获得分层结果集。为了仅基于数据库中员工记录中经理 ID 的存在来获取员工的整个层次结构，我们必须考虑使用多个连接到同一表的查询。在 SQL 选项卡中，输入此查询：

```php
SELECT CONCAT_WS('->', t1.last_name, t2.last_name, t3.last_name, t4.last_name, t5.last_name, t6.last_name) AS path
FROM employees AS t1
RIGHT JOIN employees AS t2 ON t2.superior = t1.id
RIGHT JOIN employees AS t3 ON t3.superior = t2.id
RIGHT JOIN employees AS t4 ON t4.superior = t3.id
RIGHT JOIN employees AS t5 ON t5.superior = t4.id
RIGHT JOIN employees AS t6 ON t6.superior = t5.id
WHERE t1.superior IS NULL
ORDER BY path;
```

您将获得这个结果集：

![](img/c5516de5-df0b-41d3-8bff-c9321342231e.png)使用连接语句生成的所有员工的分层树

首先要注意的是，这个查询假设我们事先知道这个层次结构中的级别数量，这意味着我们之前做了一个查询来确认关于我们数据集的这个事实。第二件事是，为了检索整个结果集，必须重复`JOIN`子句的笨拙。递归 CTE 是优化这种查询的完美方式。要使用递归 CTE 获得完全相同的结果集，我们必须运行以下查询：

```php
WITH RECURSIVE hierarchy_list AS (
  SELECT id, superior, CONVERT(last_name, CHAR(100)) AS path
  FROM employees
  WHERE superior IS NULL
  UNION ALL
  SELECT child.id, child.superior, CONVERT(CONCAT(parent.path, '->', child.last_name), CHAR(100)) AS path
  FROM employees AS child
  INNER JOIN hierarchy_list AS parent ON (child.superior = parent.id)
)
SELECT path
FROM hierarchy_list
ORDER BY path;
```

如果我们通过运行它们针对 MariaDB 的性能模式来比较前两个查询，即使它们不提供有关我们层次结构中级别动态发现的相同功能，我们也会更好地了解底层发生了什么。

首先，让我们使用`EXPLAIN`语句运行多个连接查询：

![](img/f4eb1ecc-c582-40ef-8fb8-f9e4e112e77a.png)MariaDB 的连接语句查询执行计划

现在来看一下 RDBMS 的性能模式：

![](img/8fc22b94-631c-4e12-a209-4d3447747a98.png)多个连接导致数据库引擎中的 65 个操作

其次，让我们按照相同的步骤进行，但使用递归 CTE：

![](img/f055c2ca-730c-4af5-b8e7-f9504a5793b2.png)MariaDB 的递归 CTE 查询执行计划

而性能模式应该产生以下结果：

![](img/b0996943-fe99-42b8-8948-837863f9a63a.png)CTE 在数据库引擎中引起了 47 个操作

尽管这个递归 CTE 在我的电脑上比基本的多重连接查询慢了一点，但当所有选择的列都被索引时，它确实生成了更少的引擎操作。那么，为什么这更有效率呢？递归 CTE 将允许你避免创建存储过程或类似的东西，以便递归地发现你的层次树中的级别数量，例如。如果我们将这些操作添加到主查询中，多重连接查询肯定会慢得多。此外，递归 CTE 可能是一种派生表，不比视图快多少，比基本的多重连接查询稍慢一点，但在查询数据库时非常可扩展和有用，以便在休息时基于小的结果子集修改表内容，同时确保你更复杂的查询将免费受益于未来的引擎优化。此外，它将使你的开发周期更加高效，因为它将使你的代码对其他开发人员更易读，保持**DRY**（“**不要重复自己**”）。

让我们继续下一个特性，`CASE`表达式。

# 案例

尽管`CASE`表达式似乎让我们想起了诸如`IF`、`SWITCH`之类的命令式结构，但它仍不允许像这些命令式结构那样进行程序流控制，而是允许根据某些条件对值进行声明性评估。让我们看下面的例子，以更好地理解这个特性。

请在`phpMyAdmin`界面的测试数据库的 SQL 选项卡中输入以下查询：

```php
SELECT id, COUNT(*) as Total, COUNT(CASE WHEN superior IS NOT NULL THEN id END) as 'Number of superiors'
FROM employees
WHERE id = 2;
```

这个查询应该产生以下结果集：

![](img/777459b1-42ce-4b2f-bf6f-7b9c17dbe087.png)包含 CASE 语句的查询结果集

正如结果所示，具有`id`值为`2`的行被从第二个`COUNT`函数的输入中过滤掉，因为`CASE`表达式应用了条件，即上级列必须不具有`NULL`值才能计算 id 列。使用这个现代 SQL 的特性，在很大程度上不是为了提高性能，而是为了尽可能地避免存储过程和控制执行流程，同时保持代码清晰、易读和可维护。

# OVER 和 PARTITION BY

`OVER`和`PARTITION BY`是窗口函数，允许对一组行进行计算。与聚合函数不同，窗口函数不会对结果进行分组。为了更好地理解这两个窗口函数，让我们花点时间在`phpMyAdmin`的 Web 界面上运行以下查询：

```php
SELECT DISTINCT superior AS manager_id, (SELECT last_name FROM employees WHERE id = manager_id) AS last_name, SUM(salary) OVER(PARTITION BY superior) AS 'payroll per manager'
FROM employees
WHERE superior IS NOT NULL
ORDER BY superior;
```

运行这个查询后，你应该看到以下结果：

![](img/4c735583-b305-467f-9f6d-026e2f01ca5a.png)每个经理的工资单列表

正如我们所看到的，结果集显示了每个经理的工资单列，而不是对结果进行分组。这就是为什么我们必须使用`DISTINCT`语句，以避免对同一个经理出现多行。显然，窗口函数允许在当前行与某种关系的行子集上进行高效的查询和优化性能的聚合计算。

# OVER 和 ORDER BY

`OVER AND ORDER BY`窗口函数在对行子集进行排名、计算累积总数或简单地避免自连接时非常有用。

为了说明何时使用这个最有用的特性，我们将采用前面的例子，并通过执行这个查询来确定每个经理的每个工资单上薪水最高的员工：

```php
SELECT id, last_name, salary, superior AS manager_id, (SELECT last_name FROM employees WHERE id = manager_id) AS manager_last_name, SUM(salary) OVER(PARTITION BY superior ORDER BY manager_last_name, salary DESC, id) AS payroll_per_manager
FROM employees
WHERE superior IS NOT NULL
ORDER BY manager_last_name, salary DESC, id;
```

执行这个查询将得到以下结果集：

![](img/7db91a77-1ede-437a-bbd5-7cad5cb5a5b1.png)每个经理的每个工资单上薪水最高的员工列表

返回的结果集使我们能够查看每个工资单的细分，并对每个子集中的每个员工进行排名。那么，允许我们获取关于这些数据子集的所有这些细节的底层执行计划是什么？答案是一个`SIMPLE`查询！在我们的查询中，存在一个依赖子查询，但这是因为我们正在获取每个经理的姓氏，以使结果集更有趣。

在删除依赖子查询后，这将是生成的查询：

```php
SELECT id, last_name, salary, superior AS manager_id, SUM(salary) OVER(PARTITION BY superior ORDER BY manager_id, salary DESC, id) AS payroll_per_manager
FROM employees
WHERE superior IS NOT NULL
ORDER BY manager_id, salary DESC, id;
```

这是相同查询版本的底层执行计划：

![](img/cd90e410-83c9-43db-b954-62c9bf09cf54.png)避免获取经理姓氏时，查询执行计划是简单的

通过在没有返回每个经理姓氏的依赖子查询的情况下运行查询，我们的查询执行计划的`select_type`是`SIMPLE`。这将产生一个高效的查询，未来易于维护。

# GROUPING SETS

GROUPING SETS 使得可以在一个查询中应用多个`GROUP BY`子句。此外，这一新功能引入了`ROLLUP`的概念，它是结果集中添加的额外行，以先前返回的值的超级聚合形式给出结果的摘要。让我们在测试数据库的 employees 表中给出一个非常简单的例子。在`phpMyAdmin`的 Web 界面中执行以下查询：

```php
SELECT superior AS manager_id, SUM(salary)
FROM employees
WHERE superior IS NOT NULL
GROUP BY manager_id, salary;
```

执行后，您应该会看到这个结果：

![](img/65ce923d-82e4-43dc-80bd-0ab65ea65110.png)GROUPING SETS 使得可以在一个查询中应用多个 GROUP BY 子句

多个`GROUP BY`子句使我们能够快速查看每个经理监督下每个员工的工资。如果现在将`ROLLUP`操作符添加到`GROUP BY`子句中，我们将获得这个结果：

![](img/b0c388e6-2ed6-4a28-9db9-1cae7aa20bbf.png)在 GROUP BY 子句中添加 ROLLUP 操作符后的结果集

`ROLLUP`操作符添加了额外的行，其中包含每个子集和整个结果集的超级聚合结果。执行计划显示，底层的`select_type`再次是`SIMPLE`，而不是在此功能存在之前我们将使用`UNION`操作符将多个查询合并。再次，现代 SQL 为我们提供了一个高度优化的查询，将在未来多年内保持高度可维护性。

# JSON 子句和函数

SQL 语言的最新增强之一是 JSON 功能。这一系列新功能使得更容易从 SQL 本机函数中受益，将某些类型的非结构化和无模式数据（如 JSON 格式）以非常结构化和关系方式存储。这允许许多事情，例如对 JSON 文档中包含的某些 JSON 字段应用完整性约束，对某些 JSON 字段进行索引，轻松地将非结构化数据转换并返回为关系数据，反之亦然，并通过 SQL 事务的可靠性插入或更新非结构化数据。

为了充分欣赏这一系列新功能，让我们通过执行查询将一些数据插入测试数据库，将 JSON 数据转换为关系数据。

首先，请在容器的 CLI 上执行以下命令：

```php
# mysql -uroot test < /srv/www/json_example.sql 
```

新表加载到数据库后，您可以执行以下查询：

```php
SELECT id,
   JSON_VALUE(json, "$.name") AS name,
   JSON_VALUE(json, "$.roles[0]") AS main_role,
   JSON_VALUE(json, "$.active") AS active
FROM json_example
WHERE id = 1;
```

执行后，您应该会看到这个结果：

![](img/902782e2-c7c8-4e12-8aaf-d1238df20da2.png)JSON 函数会自动将 JSON 数据转换为关系数据

正如我们所看到的，使用新的 JSON 函数将 JSON 非结构化数据转换为关系和结构化数据非常容易。将非结构化数据插入结构化数据库同样容易。此外，添加的约束将验证要插入的 JSON 字符串是否有效。为了验证这一功能，让我们尝试将无效的 JSON 数据插入我们的测试表中：

```php
INSERT INTO `json_example` (`id`, `json`) VALUES (NULL, 'test');
```

尝试执行查询时，我们将收到以下错误消息：

![](img/ca660695-dede-40df-932e-b66f89a12210.png)JSON 约束确保要插入的 JSON 字符串是有效的

因此，现代 SQL 使得在 SQL 环境中轻松处理 JSON 格式的数据。这将极大地优化应用程序级别的性能，因为现在可以消除每次应用程序需要检索或存储 JSON 格式数据到关系数据库时都需要`json_encode()`和`json_decode()`的开销。

还有许多现代 SQL 功能，我们可以尝试更好地理解，但并非所有 RDBMS 都已实现，并且其中许多功能需要我们分析实现细节。我们将简单地看一下两个在 MariaDB 服务器中未实现但在 PostgreSQL 服务器中实现的功能。要启动和使用包含在 Linux for PHP 容器中的 PostgreSQL 服务器，请在容器的 CLI 上输入以下命令：

```php
# /etc/init.d/postgresql start 
# cd /srv 
# wget --no-check-certificate -O phpPgAdmin-5.1.tar.gz https://superb-sea2.dl.sourceforge.net/project/phppgadmin/phpPgAdmin%20%5Bstable%5D/phpPgAdmin-5.1/phpPgAdmin-5.1.tar.gz 
# tar -xvf phpPgAdmin-5.1.tar.gz 
# sed -i "s/extra_login_security'] = true/extra_login_security'] = false/" phpPgAdmin-5.1/conf/config.inc.php 
# cd fasterweb/chapter_6 
# ln -s ../../phpPgAdmin-5.1 ./phppgadmin # cd /srv/www
```

输入这些命令后，您应该能够通过`phpPgAdmin` Web 界面访问 PostgreSQL 服务器，网址为`http://localhost:8181/phppgadmin`。将浏览器指向此地址，并点击屏幕右上角的服务器图标，以查看以下界面：

![](img/1c4e010d-a76d-4d51-9676-cae9bb368916.png)列出唯一可用的 PostgreSQL 服务器，并通过端口 5432 访问

在这里，点击页面中央的 PostgreSQL 链接，在登录页面上，将用户名输入为`postgres`，密码留空：

![](img/c770642c-08c9-4ddb-a5a5-9b3468636b4f.png)在登录页面上，输入用户名'postgres'，并将密码框留空

然后，点击“登录”按钮，您应该能够访问服务器：

![](img/6bf40f18-277d-4e62-ab10-6505c96eb23b.png)服务器显示 postgres 作为唯一可用的数据库

最后，我们将创建一个数据库，以便学习如何使用本书中将要介绍的最后两个现代 SQL 功能。在`phpPgAdmin`界面中，点击“创建数据库”链接，并填写表单以创建 test 数据库：

![](img/b9e3abd9-2616-4dd0-8f9c-f17aa5ce963d.png)使用 template1 模板和 LATIN1 编码创建名为 test 的数据库

点击“创建”按钮，您将创建 test 数据库并将其与 postgres 数据库一起创建：

![](img/347ae761-f294-471d-98b6-393bcdaa8114.png)现在，服务器显示 test 数据库和 postgres 数据库

完成后，在容器的 CLI 上输入以下命令：

```php
# su postgres 
# psql test < sales.sql 
# exit 
```

现在我们准备尝试`FILTER`子句。

# FILTER

现代 SQL 的另一个非常有趣的功能是`FILTER`子句。它可以将`WHERE`子句添加到聚合函数中。让我们通过在`phpPgAdmin`界面的 test 数据库的 SQL 选项卡中执行以下查询来尝试`FILTER`子句：

```php
SELECT
   SUM(total) as total,
   SUM(total) FILTER(WHERE closed IS TRUE) as transaction_complete,
   year
FROM sales
GROUP BY year;
```

您应该会得到以下结果：

![](img/a84067cb-b25f-4409-bf2b-cdf8e447095a.png)包含 FILTER 语句的查询结果集

`FILTER`子句非常适合在查询的`WHERE`子句中生成报表而不会增加太多开销。

此外，`FILTER`子句非常适合数据透视表，其中按年和月进行分组更加复杂，因为必须生成一个跨越月份和年份的报表，这在两个不同的轴上（例如月份=*x*和年份=*y*）变得更加复杂。

让我们继续讨论最后一个现代 SQL 功能，`LATERAL`查询。

# LATERAL 查询

`LATERAL`查询允许您在相关子查询中选择多个列和一行以上。在创建 Top-N 子查询并尝试将表函数连接在一起时，这非常有用，从而使得解除嵌套成为可能。然后，可以将`LATERAL`查询视为一种 SQL `foreach`循环。

让我们举一个简单的例子来说明`LATERAL`查询是如何工作的。假设我们有两个假想的包含有关电影和演员数据的表：

```php
SELECT
    film.id,
    film.title,
    actor_bio.name,
    actor_bio.biography
FROM film,
    LATERAL (SELECT 
                 actor.name,
                 actor.biography
             FROM actor
             WHERE actor.film_id = film.id) AS actor_bio;
```

正如我们所看到的，`LATERAL`子查询从演员表中选择了多列（actor.name 和 actor.biography），同时仍然能够与电影表（film.id）相关联。许多优化，无论是性能优化还是代码可读性和可维护性，都成为了使用`LATERAL`查询的真正可能性。

有关现代 SQL 的更多信息，我邀请您查阅 Markus Winand 的优秀网站（https://modern-sql.com），并收听 Elizabeth Smith 在 Nomad PHP 上关于这个主题的精彩演讲（https://nomadphp.com/product/modern-sql/）。

# 总结

在本章中，我们学习了如何使用现代 SQL 高效地查询 SQL 数据库。我们定义了现代 SQL 是什么，以及我们如何使用它。我们学会了如何将某些传统的 SQL 查询转换为现代查询，以及何时最好这样做。此外，通过这样做，我们现在更好地理解了现代 SQL 如何帮助我们以多种方式优化服务器的性能。

在下一章中，我们将介绍一些 JavaScript 的优点和缺点，特别是与代码效率和整体性能有关的部分，以及开发人员应该如何始终编写安全、可靠和高效的 JavaScript 代码，主要是通过避免“危险驱动开发”。
