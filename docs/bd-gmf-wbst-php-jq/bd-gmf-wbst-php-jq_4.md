# 第四章：玩家

在本章中，我们将概述我们系统/项目中的角色、用户和利益相关者（玩家）。在第二章中，*框架*，我们讨论了巴特尔游戏测试中概述的玩家类型（杀手、成就者、社交者和探险者）。我们需要更仔细地研究我们的用户/玩家及其动机，以便在设计我们的游戏化电子学习系统时最大程度地了解这一点。我们将在 MySQL 中完成一些初始数据库设计，以便我们有所有的技术来构建我们的系统。

# Bartle 游戏心理测试

尽管它可能需要更新，但巴特尔游戏心理测试已经成为讨论游戏化系统中玩家类型的标准。巴特尔在上世纪 90 年代中期设计了这个系统/测试，当时是在视频游戏设计的背景下，并已经被移植到了游戏化领域。这个模型可能需要一些调整，但它是朝着正确方向的良好开端。

巴特尔测试是一系列设计用于评估个性类型和偏好相对于游戏情景中的互动的 30 个问题。玩家通常在一个类别中有更多的倾向。让我们快速回顾一下它们。

## 杀手

杀手受到对系统中其他玩家的直接游戏和影响的激励。他们渴望赢得游戏。我们将在我们的系统中支持这些玩家，并讨论以下章节中的游戏元素。

### 排行榜

排行榜是游戏中得分最高的领导者的显著展示图形。以下是一个例子：

![排行榜](img/8119_04_01.jpg)

### 为辩护和批评帖子提供奖励

由于杀手受到直接竞争的激励，我们可以通过为玩家为其他玩家的帖子辩护以及攻击（批评）玩家的帖子提供额外积分来满足这种动机。

![为辩护和批评帖子提供奖励](img/8119_04_02.jpg)

## 成就者

与杀手一样，成就者希望对系统产生直接影响。然而，这种影响不需要以其他玩家为代价。玩家被驱使着赢得并实现目标，但不需要*赢*，因为达到目标就足够了。

我们可以通过以下游戏元素来实现这一点。

### 徽章

成就者受到他们的成就带来的认可的高度激励。我们可以通过排行榜在一定程度上满足这种动机。然而，排行榜将玩家置于彼此之间的竞争中，这对成就者来说并不像杀手那样有益。因此，他们需要奖杯。他们需要对他们的成就给予认可，而不是为了成就而成就。徽章使这成为可能。

徽章是玩家获得的图形图像，表示他们已经达到了一个里程碑。我们非常习惯用徽章来承认成就，从青少年侦察项目到对权威人物的认可。因此，这个概念在社会上是可以接受的。

将徽章计划纳入系统是游戏化运动的主要批评之一，因此我们需要小心地运用机制。糟糕的徽章通常会适得其反。

我们希望确保我们的徽章系统符合以下四个标准。徽章应该：

+   **用令人向往的、可预测的成就来惊喜玩家**：徽章通常在用于认可明确成就时效果最好。奖励本身有一定的惊喜因素。然而，它们不能太随机，以至于玩家无法将其与他们所要认可的进展或地位联系起来。另一方面，它们也不能太线性，以至于玩家对它们感到厌倦。我们的电子学习系统中的玩家将收到各种徽章，以表示他们与系统的参与程度。玩家达到一定阈值（比如 25 篇帖子）后，他们将获得参与徽章。玩家将能够在继续达到其他阈值时获得更多徽章。

+   **美观性**：徽章需要看起来好看，对玩家视觉上吸引力。玩家需要为徽章的外观感到自豪。

+   **稀缺性**：徽章系统的一个更有力的方面是稀缺性因素。如果某物“不易获得”，它往往会被认为具有更大的价值。因此，徽章应该很少发放。因此，我们将限制对任何成就发放的徽章数量为用户的预定百分比。此外，随着预定百分比的用户达到徽章，我们将提高未来徽章的门槛水平。这增加了获得这些徽章的难度水平。

+   **有意义**：我们需要将徽章系统与玩家会发现有意义的东西联系起来。我们不能仅仅为了发放徽章而发放。尽管对一些人来说收集徽章是一种动机，但收集的需求不应该盖过实现的需求。

一些示例徽章

### 级别/进度

我们已经在之前关于徽章的讨论中讨论了级别。级别是一种游戏机制，通过这种机制我们奖励并认可玩家在系统中达到一定的掌握水平。在许多情况下，达到更高的级别也会为玩家打开更多的游戏福利和功能。

我们已经讨论过给予我们的玩家批评或者捍卫帖子的能力（请参阅*捍卫和批评帖子的奖励*部分的截图）。然而，我们可以在玩家在系统中达到一定的参与水平之后才提供这个功能。我们可以通过积累的积分数量来衡量这种参与水平。

### 挑战

挑战特别吸引成就者类型的动机。挑战是玩家可以努力实现的预设目标。

在我们的系统中，我们允许玩家为自己设定目标，但我们也可以向玩家提供挑战。例如，我们可以挑战玩家在 7 天内回复一定数量的帖子。这是许多成就者会努力实现并且可以合理获得的目标。

## 社交者

这些玩家的动机与杀手和成就者的动机非常不同。社交者通过与其他玩家互动而受到驱动。像杀手一样，他们渴望与其他玩家互动，但他们的互动并不是具有攻击性的，而是更具合作性的。

我们将通过以下元素来支持社交者：

+   **分享**：我们对分享的概念非常熟悉。这是社交媒体的核心。我们将奖励玩家与他们在 Facebook 和 Twitter 上的朋友分享帖子的积分。但分享不仅限于玩家已建立的社交网络。分享的一个关键方面是与可能对分享感兴趣的其他人交友或关注。在我们的电子学习系统中，玩家将有选择关注那些他们倾向于有共同观点的其他玩家的选项。当玩家关注其他玩家或被其他玩家关注时，将奖励他们积分。

+   **团队合作**：团队合作考虑了队伍的积分累积。例如，玩家可以创建团队。我们将实施一个团队排行榜。而杀手和成就者会采取行动来提高他们在排行榜上的得分，社交者将确保更倾向于采取行动来提高他们团队的积分累积。作为个人，这个行动可能不是优先考虑的，但作为团队的贡献者，它是的。

## 探险者

通过审查探险者以及我们如何吸引这种游戏个性类型，我们完成了对我们游戏化系统潜在玩家的审视。探险者和社交者一样，寻求互动和参与，但更多的是与系统本身而不是其他玩家互动。像成就者一样，他们想要与系统互动，但不是为了实现目标。他们满足于享受游戏本身。对他们来说，旅程就是奖励。发现是他们的主要动机。

与探险者一起运作良好的游戏机制是任务（搜索）、谜题和收集。以下是我们可以在系统中实施的一些例子。

我们可以通过设置玩家随时可以参与的一系列任务来在系统中实施任务/搜索。我们可以为每次玩家查看帖子记录积分，但是每次帖子在不同类别中时，将帖子的积分值加倍。目标是创造一种冒险感。

另一个可能的情景是为创建新的讨论主题给予积分和奖励。探险者总是试图推动系统的极限。

# 我们的玩家

以下列表显示了各种玩家：

+   **发帖者**：这是一个回应另一个玩家观点的通用玩家。

+   **防守者**：这是支持玩家观点的发帖者。

+   **评论家**：这是一个不同意玩家观点的发帖者。

+   **队友**：这是一个与另一个玩家有团队关系的玩家。

+   **新手**：这是第一个在某个主题上发表观点的玩家。

+   **创建者**：这是一个创建讨论主题的玩家。

+   **新手**：这是一个有账户但没有回复或发帖的玩家。

+   **老手**：这是一个拥有超过 30 天和/或 30 次登录会话的玩家。

# 创建 MySQL 数据库

我们需要一种方式来保存关于我们的玩家和系统的所有信息。如果我们没有以合理的方式存储这些信息，玩家每次登录时都需要重新开始我们的系统。这不是一个好的系统。

在我们的环境中，我们选择使用开源数据库 MySQL。如果您在上一章中安装了 WAMP 服务器，您应该已经安装了 MySQL。在这里我们将开始使用它。

我们的 WAMP 安装包括 phpMyAdmin，如下截图所示。这是一个可以直接从 Web 浏览器创建和管理数据库的工具。

![创建 MySQL 数据库](img/8119_04_04.jpg)

点击 WAMP 服务器菜单上的**phpMyAdmin**选项后，我们应该期望看到 PHP Admin 工具的主屏幕：

![创建 MySQL 数据库](img/8119_04_05.jpg)

让我们创建主要的 Point 数据库。点击**数据库**菜单。将数据库命名为`VuPoint`，然后点击**创建**。您可以在左侧看到我们的数据库列表：

![创建 MySQL 数据库](img/8119_04_06.jpg)

我们可以在**PHPMyAdmin**主屏幕的左侧的数据库列表中找到我们新创建的数据库：

创建 MySQL 数据库

有许多基于 GUI 的 MySQL 工具，我们可以安装以使一些数据库表的创建和存储过程的创建更简单。我们将以两者为例。选择您最熟悉的那个。前往[www.mysql.com/downloads](http://www.mysql.com/downloads)查看工具选项。

## 创建我们的表

数据库由由列/字段组成的表组成。在最初创建表后，我们可以添加/修改/删除列，但最好将这些类型的更改保持在最低限度，因为它们可能会严重干扰已存储在表中的数据。

这是我们系统所需的一些表：

+   `玩家`：这个表将保存关于系统用户的所有信息

+   `玩家 ID`：这是每个玩家的内部唯一标识符

+   `用户名`：这是玩家在系统中将被称为的名称（外部标识符）

+   `密码`：玩家用于登录系统的密码

+   `积分`：玩家当前累积的积分

+   `获得徽章`：玩家通过徽章获得的活动 ID

+   `电子邮件地址`：玩家的唯一电子邮件地址

+   `登录次数`：玩家登录的总次数

+   `最后登录`：玩家上次登录系统的日期和时间

+   `当前登录`：该玩家是否登录的`true`/`false`值

+   `账户创建`：玩家创建此账户的日期和时间

请注意，`PlayerID`字段是使用`auto_increment`创建的，并且是主键（如上所示）。请注意，`AccountCreated`表是一个时间戳数据类型，其默认值为`current_timestamp`。我们将在创建的其他表中模仿这种方法。

我们需要创建其余的表，如下所示：

+   `帖子`：此表将包含系统中的所有帖子

+   `帖子 ID`：每个帖子的内部唯一标识符

+   `类型`：帖子的类型（`防御`、`批评`或`中立`）

+   `主题`：此帖子相关的讨论主题的 ID

+   `玩家`：发帖的玩家的 ID

+   `帖子`：发布的消息的实际文本

+   `日期时间`：帖子创建的日期和时间

+   `活动`：此表将包含玩家可以获得积分的所有可能事项及其积分值

+   `活动 ID`：活动的内部唯一标识符

+   `活动`：活动的描述

+   `值`：特定活动的积分值

+   `徽章`：与活动相关的徽章（如果存在）。

+   `主题`：此表包含有关讨论主题的所有信息

+   `主题 ID`：主题的内部唯一标识符

+   `主题`：讨论主题的实际文本

+   `创建者`：创建主题的玩家的 ID

+   `帖子数量`：主题上中立帖子的总数

+   `添加日期`：此主题添加到系统中的日期和时间

现在我们已经有了支持 VuPoint 电子学习应用的基本表结构，让我们开始向应用程序添加一些基本代码。我们有三个代码将执行的地方。我们可以编写在客户端（浏览器）中使用 JQuery/JavaScript 运行的代码。我们可以编写在服务器上使用 PHP 运行的代码。我们可以编写在数据库中使用 MySQL 存储过程运行的代码。理论上，我们可以编写所有代码在一个地方运行，但在大多数情况下，这不仅不切实际，而且还会使我们的一些任务变得更加困难。所以问题是，“在哪里编写我们的代码最好？”以下是我们将遵循的顺序：

1.  我们将编写在数据库中添加、修改或删除数据的任务部分。

1.  我们将编写生成在服务器上运行的 HTML 的任务部分。

1.  我们将编写与玩家交互的任务部分，使用 JQuery。

那么可能使用数据库的一些操作是什么？有几种，但以下是一些起步的操作：

| 操作 | 存储过程 | 输入 | 输出 |
| --- | --- | --- | --- |
| 检查用户是否经过身份验证 | `选择玩家` | `用户名``电子邮件地址``密码` | 恰好一条玩家记录 |
| 添加新玩家 | `插入玩家` | `用户名``电子邮件地址``密码` | N/A |
| 获取玩家的积分 | `选择玩家积分` | `用户名` | 总积分数量 |
| 记录帖子 | `插入帖子` | `用户名``类型``主题``玩家 ID``帖子` | N/A |
| 获取特定主题的所有帖子 | `选择主题帖子` | `主题` | 与该主题相关的所有帖子记录 |
| 获取所有主题 | `选择所有主题` | N/A | 所有主题 |
| 获取排行榜数据 | `选择领导者` | `多少` | 按积分累积数量排序的前几名玩家的记录（比如前 10 到 25 名），并且这由`多少`限制 |
| 获取在线玩家 | `SelectOnlinePlayers` | N/A | 所有`LoggedIn`列值为`true`的玩家记录 |
| 创建新的讨论话题 | `InsertATopic` | `Topic``Player` | N/A |
| 获取热门话题 | `SelectHotTopics` | `HowMany` | 按帖子累积数量排序的热门话题记录（比如前 10 到 25 个），由`HowMany`限制 |
| 获取玩家徽章 | `SelectPlayerBadges` | `Player` | 玩家获得的徽章的逗号分隔列表 |

以下是创建这些存储过程的代码：

```php
DELIMITER //
-- Procedure Name: InsertAPlayer
-- Procedure Purpose: Inserts a New Player into the system
CREATE PROCEDURE 'vupoint'.'InsertAPlayer'(IN _UserName varchar(20),

  _EmailAddress varchar(40),
  _Password varchar(20))
BEGIN
Insert into Player (UserName,Password,EmailAdddress) values(_UserName,_Password,_EmailAdddress);
END//

DELIMITER ;

DELIMITER //
-- Procedure Name: InsertAPost
-- Procedure Purpose: Inserts a posts into the VuPoint database
CREATE PROCEDURE 'vupoint'.'InsertAPost'(IN _UserName varchar(20),   _Type varchar(10),_Topic int,_Player int,_Post varchar(4000))
BEGIN
Insert into Posts (UserName,Type,Topic,Player,Post) values(_UserName,_Type,_Topic,_Player,_Post);
END//

DELIMITER ;

DELIMITER //
-- Procedure Name: InsertATopic
-- Procedure Purpose: Inserts a new topic into the system 
CREATE PROCEDURE 'vupoint'.'InsertATopic'(IN _Topic varchar(10), _Player int)
BEGIN
Insert into Topics (Topic,PlayerCreated) values (_Topic,_Player);
END//

DELIMITER ;

DELIMITER //
-- Procedure Name: SelectAPlayer
-- Procedure Purpose: Returns the record for a player
CREATE PROCEDURE 'vupoint'.'SelectAPlayer' (In _UserName varchar(20), 
  _EmailAddress varchar(40),
  _Password varchar(20) )
BEGIN
Select * from Player 
where (UserName=_UserName|| EmailAddress=_EmailAddress)
and Password=_Password;
END // 

DELIMITER ;

DELIMITER //
-- Procedure Name: SelectAllTopics
-- Procedure Purpose: Returns all the topics in the system
CREATE PROCEDURE 'vupoint'.'SelectAllTopics'()
BEGIN
Select * from Topics;
END//
DELIMITER ;

DELIMITER //
-- Procedure Name: SelectPlayerBadges
-- Procedure Purpose: Returns a list of the players badges
CREATE PROCEDURE 'vupoint'.'SelectPlayerBadges'(IN _UserName varchar(20),
  OUT _badges varchar(100))
BEGIN
Select BadgesEarned into _badges from Player 
where UserName=_UserName;
END//
DELIMITER ;

DELIMITER //
-- Procedure Name: SelectHotTopics
-- Procedure Purpose: Returns topics ordered by the number of posts and limited by how -- many
CREATE PROCEDURE 'vupoint'.'SelectHotTopics'(IN _HowMany int)
BEGIN
Select  * from Topics order by NumberOfPosts desc LIMIT 0,_HowMany ;
END//
DELIMITER ;

DELIMITER //
-- Procedure Name: SelectOnlinePlayers
-- Procedure Purpose: Returns a lists of players how currently logged in is true
CREATE PROCEDURE 'vupoint'.'SelectOnlinePlayers'()
BEGIN
Select  * from Player where CurrentlyLoggedIn=true ;
END//
DELIMITER ;

DELIMITER //
-- Procedure Name: SelectPostsByTopic
-- Procedure Purpose: Returns all the posts associated with a Topic
CREATE PROCEDURE 'vupoint'.'SelectPostsByTopic'(IN _Topic int)
BEGIN
Select * from posts where Topic =_Topic ;
END//
DELIMITER ;
```

# 总结

我们已经确定了系统中的玩家。我们知道是什么激励了每个玩家，以及我们如何给予他们需要的反馈。现在，我们已经有了数据库设计，这是我们游戏化系统的基础。现在，我们只需要为我们的骨架系统添加细节。
