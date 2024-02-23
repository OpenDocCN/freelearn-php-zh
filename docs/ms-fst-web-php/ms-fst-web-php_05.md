# 测量和优化数据库性能

在本书的第一章中，我们使用 mysqlslap 工具学习了如何进行基本的 MySQL 基准测试。在本章中，我们将使用这个工具和其他工具来对我们的 MariaDB（MySQL）服务器进行更高级的基准测试。但首先，我们将学习查询优化技术，这些技术将使用 MySQL 的一些内置功能，以便更好地分析我们的 SQL 查询。

因此，我们将学习如何通过使用简单的测量技术来测量和优化数据库性能，例如查询优化。此外，我们将看到如何使用高级数据库基准测试工具，如 DBT2 和 SysBench。

因此，我们将涵盖以下几点：

+   测量和优化 SQL 查询性能

+   安装、配置和使用高级数据库基准测试工具

# SQL 查询性能

为了更好地理解 SQL 查询性能，我们必须首先了解索引是什么以及它们是如何构建的。

# 索引的结构

索引是表元素的有序列表。这些元素首先存储在物理上无序的双向链表中。该列表通过指向表条目和存储索引值的第二个结构的指针双向链接到表，以逻辑顺序、平衡树或 b 树存储索引值。因此，索引具有对数算法复杂度，平均读操作为 O(log n)，这意味着即使表中有大量条目，数据库引擎也应该保持速度。实际上，索引查找涉及三个步骤：

+   树遍历

+   搜索叶节点链

+   从表中获取数据

因此，当仅从 b 树中读取时，索引查找是很好的，因为你避免了线性的 O(n)完整表扫描。尽管如此，你永远无法避免由于在写入表时保持索引最新而引起的开销复杂性。

这带我们来到了关于查询优化的第一个考虑因素：表的数据的最终目的是什么？我们只是记录信息还是存储用户的购物车商品？我们查询的表大多是读取还是写入？这很重要，因为优化一个 SELECT 查询可能会减慢对同一表的整个系列其他 INSERT 或 UPDATE 查询的速度。

第二个考虑因素是表的数据的性质。例如，我们是否试图索引生成等价性的值，从而迫使数据库引擎在 b 树的叶节点中进行进一步查找，以确定真正满足特定查询期望的所有值？当等价性是一个问题时，我们可能会得到一个“慢索引”或者通常被称为“退化索引”。

第三个考虑因素是围绕查询表的效率的经济性。底层计算机有多强大？平均有多少用户在给定时间查询表？可伸缩性重要吗？

最后一个考虑因素是数据的存储大小。重要的是要知道，一般规则是，索引的大小平均增长到原始表大小的约 10%。因此，当表的大小很大时，预计表的索引也会更大。当然，索引越大，由于 I/O 延迟，等待的时间就越长。

这些考虑因素将决定要优化哪个查询以及如何优化它。有时，优化就是什么都不做。

现在我们更好地理解了索引，让我们开始分析简单的 SQL 查询，以了解数据库引擎执行计划。

# 执行计划

我们将通过分析简单的`WHERE`子句来开始理解执行计划。为此，我们将使用我们的第一个 Linux for PHP Docker 容器。在第一章中，我们将 Sakila 数据库加载到 MariaDB（MySQL）服务器中。现在我们将使用它来学习执行计划的工作原理以及何时使用查询优化技术。在容器的 CLI 上，输入以下命令：

```php
# mysql -uroot 
# MariaDB > USE sakila; 
# MariaDB > SELECT * FROM actor WHERE first_name = 'AL'; 
```

这些命令应该产生以下结果：

![](img/c6ff4e46-8bc9-4631-a7e4-89980e967449.png)SELECT 语句的结果

乍一看，这个查询似乎很好，执行时间为 0.00 秒。但是，这真的是这样吗？要进一步分析这个查询，我们需要查看数据库引擎的执行计划。为此，在查询开头输入关键字`EXPLAIN`：

```php
 # MariaDB > EXPLAIN SELECT * FROM actor WHERE first_name = 'AL'; 
```

以下结果为我们提供了一些执行计划的信息：

![](img/1958748c-a572-4ccb-b21f-ed4b2bf1d60b.png)相同 SELECT 语句的执行计划

让我们花时间来定义结果集的每一列：

+   `id`列告诉我们表的连接顺序。在这种情况下，只有一个表。

+   `select_type`是`SIMPLE`，这意味着执行此查询时没有子查询、联合或依赖查询类型。

+   `table`列给出了查询对象的表的名称。如果它是一个临时物化表，我们会在这一列看到表达式`<subquery#>`。

+   `type`列对于查询优化非常重要。它提供了关于表访问以及如何从表中找到和检索行的信息。在这种情况下，我们看到这一列的值是`ALL`，这是一个警告信号。要进一步了解这个非常重要的列的不同可能值，请参阅 MariaDB 手册[`mariadb.com/kb/en/library/explain/`](https://mariadb.com/kb/en/library/explain/)。

+   `possible_keys`列通知我们表中可以用来回答查询的键。在这个例子中，值为`NULL`。

+   `key`列指示实际使用的键。在这里，值再次为`NULL`。

+   `key_len`列中的值表示完成查询查找所使用的多列键的特定字节数。

+   `ref`列告诉我们用于与使用的索引进行比较的列或常量。当然，由于没有使用索引来执行此查询，因此这一列的值也是`NULL`。

+   `rows`列表示数据库引擎需要检查的行数以完成其执行计划。在这个例子中，引擎需要检查 200 行。如果表很大并且必须与前一个表连接，性能会迅速下降。

+   最后一列是`Extra`列。这一列将为我们提供有关执行计划的更多信息。在这个例子中，数据库引擎使用`WHERE`子句，因为它必须进行全表扫描。

# 基本查询优化

为了开始优化这个查询，我们必须经历我之前所说的查询优化的*初始考虑*。举例来说，让我们假设这个表将成为`READ`查询的对象，而不是`WRITE`查询，因为一旦写入表中，数据将保持相当静态。此外，重要的是要注意，在`actor`表的`first_name`列上创建索引将使索引容易产生由于该列中的非唯一值而产生的模糊性。此外，让我们假设可伸缩性很重要，因为我们打算每小时让许多用户查询这个表，并且表的大小应该在长期内保持可管理。

鉴于这一点，我们将在`actor`表的`first_name`列上创建一个索引：

```php
 # MariaDB > CREATE INDEX idx_first_name ON actor(first_name); 
```

完成后，MariaDB 确认了索引的创建：

![](img/da6584c8-040d-4242-aecf-9778747dbeb5.png)确认索引已创建

现在索引已创建，当我们要求数据库引擎“解释”其执行计划时，我们得到了这个结果：

执行计划现在已经优化

`type`列的值现在是`ref`，`possible_keys`是`idx_first_name`，`key`是`idx_first_name`，`ref`是`const`，`rows`是`1`，`Extra`是`Using index condition`。正如我们所看到的，引擎现在已经将我们新创建的索引识别为可能使用的关键，并继续使用它。它使用查询中给定的常量值在索引中执行查找，并在访问表时只考虑一行。所有这些都很好，但正如我们在最初的考虑中所期望的那样，索引由非唯一值组成。表列的值之间可能的等价性可能会导致随着时间的推移出现退化的索引，因此访问类型为`ref`，额外信息指示引擎正在“使用索引条件”，这意味着`WHERE`子句被推送到表引擎以在索引级别进行优化。在这个例子中，根据我们最初的考虑，这是在绝对意义上我们可以做的最佳查询优化，因为在“actor”表的`first_name`列中不可能获得唯一值。但实际上，根据领域使用情况，可能存在一种优化。如果我们只希望使用演员的名字，那么我们可以通过只选择适当的列来进一步优化`Extra`列中的`Using index condition`，从而允许数据库引擎只访问索引：

```php
 # MariaDB > EXPLAIN SELECT first_name FROM actor WHERE first_name = 'AL'; 
```

数据库引擎随后确认它只在`Extra`列中使用索引：

“Extra”列现在包含信息“使用 where; 使用索引”

这一切又如何转化为整体性能呢？让我们运行一些基准测试，以衡量我们的更改的影响。

首先，我们将在没有索引的情况下运行基准测试。在容器的 CLI 上运行以下命令：

```php
# mysqlslap --user=root --host=localhost --concurrency=1000 --number-of-queries=10000 --create-schema=sakila --query="SELECT * FROM actor WHERE first_name = 'AL';" --delimiter=";" --verbose --iterations=2 --debug-info;
```

以下是不使用索引的结果：

不使用索引的基准测试结果

以及，使用索引的结果：

使用索引的基准测试结果

最后，让我们只选择适当的列运行相同的命令，从而将查找限制为仅使用索引：

```php
# mysqlslap --user=root --host=localhost --concurrency=1000 --number-of-queries=10000 --create-schema=sakila --query="SELECT first_name FROM actor WHERE first_name = 'AL';" --delimiter=";" --verbose --iterations=2 --debug-info; 
```

这是最后一个基准测试的结果：

使用索引的基准测试结果

基准测试结果清楚地显示，我们的查询优化确实满足了我们最初的可扩展性假设，特别是如果我们看到表的大小增长，随着时间的推移，我们的数据库变得更受欢迎，用户数量也在增长。

# 性能模式和高级查询优化

查询优化的艺术可以通过使用 MariaDB（MySQL）的性能模式来进一步推进。查询分析允许我们看到发生在幕后的情况，并进一步优化复杂的查询。

首先，让我们在数据库服务器上启用性能模式。为此，请在 Linux 的 PHP 容器的 CLI 上输入以下命令：

```php
# sed -i '/myisam_sort_buffer_size =/a performance_schema = ON' /etc/mysql/my.cnf  
# sed -i '/performance_schema =/a performance-schema-instrument = "stage/%=ON"' /etc/mysql/my.cnf 
# sed -i '/performance-schema-instrument =/a performance-schema-consumer-events-stages-current = ON' /etc/mysql/my.cnf 
# sed -i '/performance-schema-consumer-events-stages-current =/a performance-schema-consumer-events-stages-history = ON' /etc/mysql/my.cnf 
# sed -i '/performance-schema-consumer-events-stages-history =/a performance-schema-consumer-events-stages-history-long = ON' /etc/mysql/my.cnf 
# /etc/init.d/mysql restart 
# mysql -uroot 
# MariaDB > USE performance_schema; 
# MariaDB > UPDATE setup_instruments SET ENABLED = 'YES', TIMED = 'YES'; 
# MariaDB > UPDATE setup_consumers SET ENABLED = 'YES'; 
```

数据库引擎将确认在`performance_schema`数据库中已修改了一些行：

“performance_schema”数据库已被修改

我们现在可以检查性能模式是否已启用：

```php
# MariaDB > SHOW VARIABLES LIKE 'performance_schema'; 
```

数据库引擎应返回以下结果：

确认性能模式现已启用

现在，性能分析已经启用并准备就绪，让我们在 Sakila 数据库上运行一个复杂的查询。在使用`NOT IN`子句的子查询时，引擎通常会迭代地对主查询进行额外的检查。这些查询可以使用`JOIN`语句进行优化。我们将使用以下查询在我们的数据库服务器上运行：

```php
# MariaDB > SELECT film.film_id 
          > FROM film 
          > WHERE film.rating = 'G' 
          > AND film.film_id NOT IN ( 
              > SELECT film.film_id 
              > FROM rental 
              > LEFT JOIN inventory ON rental.inventory_id = inventory.inventory_id 
              > LEFT JOIN film ON inventory.film_id = film.film_id 
          > ); 
```

运行查询会产生以下结果：

SELECT 语句的结果

并且，在上一个查询上使用`EXPLAIN`语句时的结果如下：

同一 SELECT 语句的执行计划

正如我们所看到的，引擎正在进行全表扫描，并使用一个物化子查询来完成其查找。要了解底层发生了什么，我们将不得不查看分析器记录的有关此查询的事件。为此，请输入以下查询：

```php
# MariaDB > SELECT EVENT_ID, TRUNCATE(TIMER_WAIT/1000000000000,6) as Duration, SQL_TEXT 
          > FROM performance_schema.events_statements_history_long WHERE SQL_TEXT like 
 '%NOT IN%'; 
```

运行此查询后，您将获得原始查询的唯一标识符：

原始查询的标识符

这些信息允许我们运行以下查询，以获取运行原始查询时发生的底层事件列表：

```php
# MariaDB > SELECT event_name AS Stage, TRUNCATE(TIMER_WAIT/1000000000000,6) AS Duration 
          > FROM performance_schema.events_stages_history_long WHERE NESTING_EVENT_ID=43; 
```

这是 MariaDB 性能模式中关于我们原始查询的内容：

查询的概要显示了一个特别长的操作

这个结果显示`NOT IN`子句导致数据库引擎创建了一个物化子查询，因为内部查询被优化为半连接子查询。因此，在运行查询和物化子查询之前，引擎必须进行一些优化操作。此外，结果显示物化子查询是最昂贵的操作。

优化这些子查询的最简单方法是将它们替换为主查询中的适当`JOIN`语句，如下所示：

```php
# MariaDB > SELECT film.film_id 
#         > FROM rental 
#         > INNER JOIN inventory ON rental.inventory_id = inventory.inventory_id 
#         > RIGHT JOIN film ON inventory.film_id = film.film_id 
#         > WHERE film.rating = 'G' 
#         > AND rental.rental_id IS NULL 
#         > GROUP BY film.film_id; 
```

通过运行此查询，我们从数据库中获得相同的结果，但是`EXPLAIN`语句揭示了一个全新的执行计划，以获得完全相同的结果：

新的执行计划只显示了“SIMPLE”选择类型

子查询已经消失，变成了简单的查询。让我们看看性能模式这次记录了什么：

```php
# MariaDB > SELECT EVENT_ID, TRUNCATE(TIMER_WAIT/1000000000000,6) as Duration, SQL_TEXT 
          > FROM performance_schema.events_statements_history_long WHERE SQL_TEXT like '%GROUP BY%'; 
# MariaDB > SELECT event_name AS Stage, TRUNCATE(TIMER_WAIT/1000000000000,6) AS Duration 
          > FROM performance_schema.events_stages_history_long WHERE NESTING_EVENT_ID=22717; 
```

分析器记录了以下结果：

新查询的概要显示了相当大的性能改进

结果清楚地显示，在执行计划的初始化阶段发生的优化操作较少，并且查询执行本身大约快了七倍。并非所有物化子查询都可以以这种方式进行优化，但是，在优化查询时，物化子查询、依赖子查询或不可缓存的子查询应该总是激励我们问自己是否可以做得更好。

有关查询优化的更多信息，您可以收听 Michael Moussa 在*Nomad PHP*上关于此主题的出色演示（[`nomadphp.com/product/mysql-analysis-understanding-optimization-queries/`](https://nomadphp.com/product/mysql-analysis-understanding-optimization-queries/)）。

# 高级基准测试工具

到目前为止，我们使用了`mysqlslap`基准测试工具。但是，如果您需要更彻底地测试数据库服务器，还存在其他更高级的基准测试工具。我们将简要介绍其中两个工具：DBT2 和 SysBench。

# DBT2

这个基准测试工具用于针对 MySQL 服务器运行自动化基准测试。它允许您模拟大量的数据仓库。

要下载、编译和安装 DBT2，请在容器的 CLI 上输入以下命令：

```php
# cd /srv/www
# wget -O dbt2-0.37.tar.gz https://master.dl.sourceforge.net/project/osdldbt/dbt2/0.37/dbt2-0.37.tar.gz 
# tar -xvf dbt2-0.37.tar.gz 
# cd dbt2-0.37.tar.gz 
# ./configure --with-mysql 
# make 
# make install 
# cpan install Statistics::Descriptive 
# mkdir -p /srv/mysql/dbt2-tmp-data/dbt2-w3 
# ./src/datagen -w 3 -d /srv/mysql/dbt2-tmp-data/dbt2-w3 --mysql 
```

一旦数据仓库被创建，您应该看到以下消息：

确认数据库仓库已经创建。

现在，您需要使用 vi 编辑器修改文件`scripts/mysql/mysql_load_db.sh`：

```php
# vi scripts/mysql/mysql_load_db.sh 
```

进入编辑器后，输入`/LOAD DATA`并按*Enter*。将光标定位在此行的末尾，按`*I*`并输入大写的`IGNORE`。编辑完成后，您的文件应该是这样的：

![](img/eaf14be8-ed22-41b0-8fa6-7e582f14ca29.png)在'mysql_load_db.sh'脚本的'LOAD DATA'行上插入字符串"Ignore"

完成后，按*Esc*键，然后输入`:wq`。这将保存更改并关闭 vi 编辑器。

现在，输入以下命令将测试数据加载到数据库中：

```php
# ./scripts/mysql/mysql_load_db.sh -d dbt2 -f /srv/mysql/dbt2-tmp-data/dbt2-w3 -s /run/mysqld/mysqld.sock -u root 
```

一旦数据加载到数据库中，您应该会看到以下消息：

![](img/a892a632-2f5b-4892-8cce-6008167b2eb8.png)确认数据正在加载到数据库中

要启动测试，输入以下命令：

```php
# ./scripts/run_mysql.sh -n dbt2 -o /run/mysqld/mysqld.sock -u root -w 3 -t 300 -c 20 
```

输入命令后，您将首先看到这条消息：

![](img/e1b03e63-3a1a-4d00-8d98-029e48929aac.png)确认测试已开始

您还会收到以下消息：

![](img/c7de45f4-984c-4ee5-876c-e1da8f1af596.png)确认测试正在运行

大约五分钟后，您将得到基准测试的结果：

![](img/c43b7135-035f-4634-b870-c451b74939a6.png)确认测试已完成

从给定的结果中，我们可以看到在大型数据仓库的背景下，我们可以对数据库服务器的性能有一个很好的了解。通过边缘案例测试，额外的测试可以轻松确认服务器的极限。让我们使用 SysBench 运行这样的测试。

# SysBench

SysBench 是另一个非常流行的开源基准测试工具。这个工具不仅允许您测试开源 RDBMS，还可以测试您的硬件（CPU、I/O 等）。

要下载、编译和安装 SysBench，请在 Linux for PHP Docker 容器中输入以下命令：

```php
# cd /srv/www
# wget -O sysbench-0.4.12.14.tar.gz https://downloads.mysql.com/source/sysbench-0.4.12.14.tar.gz 
# tar -xvf sysbench-0.4.12.14.tar.gz 
# cd sysbench-0.4.12.14 
# ./configure 
# make 
# make install 
```

现在，输入以下命令将创建一个包含 100 万行的表作为测试数据加载到数据库中：

```php
# sysbench --test=oltp --oltp-table-size=1000000 --mysql-db=test --mysql-user=root prepare 
```

一旦数据加载到数据库中，您应该会看到以下消息：

![](img/0eb6eee6-2020-4ae0-8e10-7ac10fbe7a04.png)确认测试数据已加载到数据库中

现在，运行测试，输入以下命令：

```php
# sysbench --test=oltp --oltp-table-size=1000000 --mysql-db=test --mysql-user=root --max-time=60 --oltp-read-only=on --max-requests=0 --num-threads=8 run 
```

输入上一个命令后，您将首先收到以下消息：

![](img/69a93d24-c499-4d1e-ae12-1e31dbf2de7d.png)确认测试正在运行

几分钟后，您应该会得到类似于以下结果：

![](img/9914961f-d1d6-405e-9167-a12a46df3bb6.png)SysBench 测试结果

结果显示，我的计算机上的 MariaDB 服务器大约可以处理每秒约 2300 次事务和每秒约 33000 次读/写请求。这些边缘案例测试可以让我们对硬件和数据库服务器的一般性能水平有一个很好的了解。

# 摘要

在本章中，我们已经学会了如何通过简单的测量技术（如查询优化）来衡量和优化数据库性能。此外，我们还看到了如何使用高级数据库基准测试工具，如 DBT2 和 SysBench。

在下一章中，我们将看到如何使用现代 SQL 技术来优化非常复杂的 SQL 查询。
