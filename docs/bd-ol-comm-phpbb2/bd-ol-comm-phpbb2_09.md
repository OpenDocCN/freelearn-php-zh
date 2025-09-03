# 附录 A. 目录结构

phpBB 2.0 的完整下载包含大约三百个文件，分为二十一目录。为了在您的 phpBB 论坛上进行某些类型的更高级工作，您需要知道哪些目录包含重要文件以及 phpBB 如何使用每个文件。一些目录和文件是可选的，而其他目录和文件对于论坛正确运行是绝对必需的。本附录提供了每个目录及其包含文件的快速总结。

在大多数情况下，您不应重命名任何目录或文件。当然，这也有例外，可能包括安装修改、自定义论坛的某些方面或本书其他地方提到的其他情况。当您更改名称时，请确保您了解您正在做什么。在不进行其他更改的情况下更改目录或文件名通常会导致论坛出现错误，因为它找不到必要的文件。

# 必需目录

当您从下载的压缩文件中提取 phpBB 时创建的大多数目录必须在您的网站上存在，以便论坛能够无错误地运行。这些基本必需的目录也包含论坛的大部分必要文件，但您会发现所需目录中的一些文件实际上是可选的。

## 根目录

您的论坛主目录，其中包含 phpBB 论坛的所有相关文件和其他目录，被称为**根目录**。默认情况下，此目录名为`phpBB2`。重命名`phpBB2`是一种常见做法；一些更流行的名称是`forums`和`boards`。不需要特定的名称；phpBB 的内部代码将始终使用特殊快捷符号表示法引用此名称。

根目录内的目录以及存储在这些目录中的文件将被单独检查。根目录本身包含许多重要文件，其中大部分文件被论坛成员直接频繁访问。这些文件包括其他文件，或将它们的内容添加到当前代码中，以执行某些必要任务。

有三个例外是`extension.inc`、`common.php`和`config.php`文件，这些文件被根目录中的其他文件包含。以下表格列出了根目录中包含的其他文件及其用途，之后我们将详细检查这三个更重要的文件。

| 文件名 | 文件功能 |
| --- | --- |
| `faq.php` | 列出关于论坛操作的常见问题。 |
| `groupcp.php` | 列出并管理用户组。 |
| `index.php` | 显示论坛主页，列出分类和论坛。 |
| `login.php` | 记录用户在论坛中的登录和登出操作。 |
| `memberlist.php` | 以多种方式列出所有论坛成员。 |
| `modcp.php` | 管理员控制面板；用于管理主题和帖子。 |
| `posting.php` | 允许创建、编辑和删除帖子。 |
| `privmsg.php` | 发送、接收和显示私密消息。 |
| `profile.php` | 用户账户管理的界面。调用其他文件进行大部分工作。 |
| `search.php` | 以多种方式搜索论坛。 |
| `viewforum.php` | 论坛中的主题列表。 |
| `viewonline.php` | 查看当前正在访问论坛的所有用户，以及他们查看的页面。 |
| `viewtopic.php` | 列出主题内的帖子。 |

### extension.inc

作为所有论坛页面调用的第一个文件，`extension.inc`似乎是最重要的 phpBB 文件之一。然而，它是一个非常简单的文件，几乎从未被修改过。`extension.inc`的唯一目的是定义一个变量，`$phpEx`，它包含与服务器上 PHP 代码文件关联的文件扩展名。这个扩展名通常是`.php`，这也是`extension.inc`中的默认设置，因此该文件几乎不需要修改。一些使用 PHP 3 的较老服务器可能会使用扩展名`.php3`而不是`.php`。如果出于某种原因必须使用这些扩展名之一，则需要在此文件中更改文件扩展名，并将所有以`.php`结尾的文件重命名，使其以`.php3`结尾。

### common.php

此文件实际上是通过初始化重要变量、调用几乎所有用于正常论坛功能的文件以及设置论坛配置设置来“启动”phpBB 的。在这里还执行了一些与用户数据工作的特殊安全措施。在大多数情况下，`common.php`将是 phpBB 页面调用的第二个文件。

### config.php

论坛需要访问 SQL 数据库的所有信息都存储在`config.php`文件中。这包括 SQL 服务器的地址或名称、SQL 用户名和密码以及数据库表名称的前缀。所有这些数据在安装 phpBB 时输入，以后可以通过编辑此文件进行修改。

## 管理文件

通常，您将在根目录内看到的第一个目录称为`admin`。`admin`目录包含创建 phpBB 管理控制面板的文件。任何以`admin_`开头（例如`admin_styles.php`）的`admin`目录内的文件也将用于创建在管理控制面板中显示的导航菜单。许多修改通过在此处放置新的`admin_`文件来添加新的管理部分。以下表格描述了所有默认的`admin_`文件（管理控制面板文件）：

| 管理文件名 | 文件用途 |
| --- | --- |
| `admin_board.php` | 通用配置管理。 |
| `admin_db_utilities.php` | 基本的 phpBB 数据库备份和恢复。 |
| `admin_disallow.php` | 定义不允许在论坛上使用的用户名。 |
| `admin_forum_prune.php` | 从论坛中删除旧帖子。 |
| `admin_forumauth.php` | 管理论坛的一般权限设置。 |
| `admin_forums.php` | 创建和管理论坛和分类。 |
| `admin_groups.php` | 创建、编辑和删除用户组。 |
| `admin_mass_email.php` | 向所有注册用户发送电子邮件消息。 |
| `admin_ranks.php` | 用户等级管理。 |
| `admin_smilies.php` | 添加、删除或修改表情符号图片。 |
| `admin_styles.php` | 执行所有样式和主题管理。 |
| `admin_ug_auth.php` | 定义特定用户和用户组的权限设置。 |
| `admin_user_ban.php` | 禁用用户、电子邮件地址或 IP 地址。 |
| `admin_users.php` | 管理用户资料。 |
| `admin_words.php` | 指定将在消息中自动被审查的单词。 |

`admin`目录中任何没有以`admin_`前缀命名的原始 phpBB 文件都以某种方式参与了其他页面的显示。`index.php`文件有三个功能：创建控制面板的框架、从`admin_`文件构建导航菜单，以及显示入口页面。每个管理控制面板页面都使用一个名为`pagestart.php`的文件，该文件处理面板的用户权限检查。它还可能包含`page_header_admin.php`文件，该文件创建管理页面的最顶部。匹配的`page_footer_admin.php`用于终止每个管理页面的处理，并确保其显示。

## 数据库抽象层文件

phpBB 的一个特性是能够与多种不同类型的 SQL 数据库软件包协同工作。这个特性是通过使用**数据库抽象层**来实现的，它是一组用于创建一个通用接口的文件，该接口能够访问每个支持的软件包。这些文件位于名为`db`的目录中。每个支持的软件包都有一个文件。您的论坛只使用特定数据库的文件，因此您可以删除其他文件以节省一些服务器空间。如果删除了这些文件中的任何一个，您在决定稍后更改数据库软件包时将需要恢复它们。下表列出了文件及其对应的数据库软件。`db`目录中的`index.htm`文件只是一个占位符，以防止服务器列出目录的内容。这些占位符文件存在于 phpBB 的几个目录中，并且总是命名为`index.htm`。

| 文件名 | 支持的数据库软件 |
| --- | --- |
| `db2.php` | 此文件在 phpBB 2.0 中未使用。 |
| `msaccess.php` | 通过 ODBC 使用 Microsoft Access。 |
| `mssql.php` | Microsoft SQL Server 7/2000。 |
| `mssql-odbc.php` | 通过 ODBC 使用 Microsoft SQL Server。 |
| `mysql.php` | MySQL 3.2。 |
| `mysql4.php` | MySQL 4。 |
| `oracle.php` | Oracle 7 和 8（在 phpBB 2.0.14 中已删除）。 |
| `postgres7.php` | PostgreSQL 7。 |

### 包含文件

如前所述，论坛访客访问的大部分页面将包括其他 PHP 文件，并将这些文件的代码作为当前页面的部分进行处理。这些包含的文件中的大量文件存储在论坛的`includes`目录中。除了一个`index.htm`占位符外，这里的每个文件实际上都是为其他文件包含而设计的。这些文件中没有任何一个可能会被您的用户直接访问。如果有人尝试直接访问这些文件，他们将只会看到一个空白页面或根据他们选择的文件显示的**黑客尝试**消息。这些文件中包含的几乎所有代码都是以函数或类形式存在的，这些函数或类将被包含它们的文件引用。以下表格显示了包含的文件：

| 包含文件名 | 文件的使用或内容 |
| --- | --- |
| `auth.php` | 获取用户权限设置。 |
| `bbcode.php` | 处理消息中的 BBCode 标签。 |
| `constants.php` | 定义在其他文件中使用的大量常量。 |
| `db.php` | 包含基于论坛配置的数据库抽象层文件。 |
| `emailer.php` | 包含用于发送电子邮件的类。 |
| `functions.php` | 包含在许多页面上使用的通用函数。 |
| `functions_admin.php` | 由管理控制和版主控制面板使用的函数。 |
| `functions_post.php` | 处理用于显示和数据库存储的消息。 |
| `functions_search.php` | 执行许多与搜索相关的任务。 |
| `functions_selects.php` | 创建下拉列表的 HTML 选择代码。 |
| `functions_validate.php` | 检查资料信息中的无效或不允许的细节。 |
| `page_header.php` | 显示论坛页眉。 |
| `page_tail.php` | 显示论坛页脚并结束脚本的执行。 |
| `prune.php` | 包含用于删除旧论坛主题的函数。 |
| `sessions.php` | 创建、更新和删除用户会话。 |
| `smtp.php` | 通过 SMTP 发送电子邮件，通常连接到另一个服务器。 |
| `sql_parse.php` | 将 SQL 查询的文本文件转换为数据库管理中使用的形式。 |
| `template.php` | 包含一个将模板文件解析为论坛页面的类。 |
| `topic_review.php` | 在发帖表单上显示主题回顾框。 |
| `usercp_activate.php` | 激活不活跃的用户账户。 |
| `usercp_avatar.php` | 管理头像画廊和上传。 |
| `usercp_confirm.php` | 在注册表单中创建视觉确认图像。 |
| `usercp_email.php` | 允许论坛用户通过论坛界面给其他人发电子邮件。 |
| `usercp_register.php` | 处理注册和资料编辑，包括表单创建。 |
| `usercp_sendpasswd.php` | 重置用户账户密码。 |
| `usercp_viewprofile.php` | 显示用户资料信息。 |

## 安装文件

`install` 目录包含用于安装或升级 phpBB 的文件。如果安装完成后此目录仍然存在，phpBB 将显示错误消息，因此您很可能在安装完成后已从论坛中删除了该目录。在升级 phpBB 时，您需要重新创建 `install` 目录并在其中放置升级安装程序文件。升级安装程序文件的名称会随着每个 phpBB 版本的发布而变化，但通常格式为 `update_to_2xxx.php`，其中 `2xxx` 被版本号替换，或者为 `update_to_latest.php`。一个例子是 `update_to_2011.php`。升级后，再次删除 `install` 目录。

`install` 目录内可能还包含一个名为 `schemas` 的目录。`schemas` 文件包含在安装期间用于创建论坛数据库表和默认数据的 SQL 查询。phpBB 2.0 的早期版本将 `schemas` 放置在 `db` 目录内。

## 语言包

提供 phpBB 多语言支持的文件位于 `language` 目录中。每种语言都由一个以语言命名的子目录表示，并以前缀 `lang_` 开头，例如 `lang_english, lang_dutch, lang_french` 等。首次下载时，phpBB 仅包含 `lang_english` 用于英文语言文件。每个语言子目录包含定义整个论坛中使用的文本的几个文件。请查看以下表格以了解所有默认语言文件的描述。新语言文件通常在大型修改后添加。

| 文件名 | 文件内容 |
| --- | --- |
| `lang_admin.php` | 所有仅在管理控制面板中出现的文本。 |
| `lang_bbcode.php` | 创建 BBCode 指南常见问题解答页面。 |
| `lang_faq.php` | 创建主论坛常见问题解答页面。 |
| `lang_main.php` | 论坛主要区域中出现的所有文本。 |
| `search_stopwords.txt` | 列出在论坛搜索和索引中省略的单词。 |
| `search_synonyms.txt` | 列出用作搜索词同义词的单词。 |

每个语言目录内还可能存在一个名为 `email` 的子目录，结构类似于 `language/lang_english/email`。`email` 中的文件是论坛可以发送给用户和管理员的电子邮件模板。以下表格给出了每个电子邮件模板的描述：

| 文件名 | 收件人 | 在...时使用 |
| --- | --- | --- |
| `admin_activate.tpl` | 管理员 | 用户账户必须被激活。 |
| `admin_send_email.tpl` | 用户 | 管理员向论坛成员发送大量电子邮件。 |
| `admin_welcome_activated .tpl` | 用户 | 用户账户由管理员激活。 |
| `admin_welcome_inactive .tpl` | 用户 | 用户注册且管理员必须激活账户。 |
| `coppa_welcome_inactive .tpl` | 用户 | 用户注册且年龄小于 13 岁。 |
| `group_added.tpl` | 用户 | 用户被添加到用户组中。 |
| `group_approved.tpl` | 用户 | 用户请求加入一个群组并被添加。 |
| `group_request.tpl` | 群组管理员 | 用户请求加入管理员用户组。 |
| `privmsg_notify.tpl` | 用户 | 收到一条新的私人消息。 |
| `profile_send_email.tpl` | 用户 | 用户通过论坛给另一位用户发送电子邮件。 |
| `topic_notify.tpl` | 用户 | 用户关注的主题收到新的回复。 |
| `user_activate.tpl` | 用户 | 用户必须激活其账户。 |
| `user_activate_passwd.tpl` | 用户 | 用户请求新的密码。 |
| `user_welcome.tpl` | 用户 | 用户注册且账户激活已关闭。 |
| `user_welcome_inactive.tpl` | 用户 | 用户注册并必须激活其账户。 |

## 模板存储

最后必需的目录是`templates`，其中包含与 phpBB 样式系统相关的文件。这是另一个包含重要文件的子目录。在这种情况下，子目录以包含的模板文件集命名。phpBB 附带一套模板文件，因此`templates`中只有一个目录，即`subSilver`。

这些模板目录包含用于显示 phpBB 论坛每一页的图像、样式表、配置文件和模板文件。模板文件是文本文件，通常以`.tpl`扩展名，包含如 HTML 之类的标记代码。以下表格描述了主要的`.tpl`文件。通常，还会包含一些其他文件，如层叠样式表（`.css`文件）和模板配置（`.cfg`文件），这些文件通常以模板命名，例如`subSilver.css`和`subSilver.cfg`。

| 文件名 | 用于显示 |
| --- | --- |
| `agreement.tpl` | 注册协议页面。 |
| `bbcode.tpl` | 无；用于特殊内部 BBCode 处理。 |
| `confirm_body.tpl` | 确认（做某事？：是/否）页面。 |
| `error_body.tpl` | 表单中显示的错误消息。 |
| `faq_body.tpl` | 常见问题解答页面。 |
| `groupcp_info_body.tpl` | 用户组的信息。 |
| `groupcp_pending_info.tpl` | 用户组的待处理成员。 |
| `groupcp_user_body.tpl` | 用户的用户组成员资格。 |
| `index_body.tpl` | 论坛主页；index.php。 |
| `jumpbox.tpl` | 论坛导航下拉菜单。 |
| `login_body.tpl` | 登录表单。 |
| `memberlist_body.tpl` | 会员列表页面。 |
| `message_body.tpl` | 大多数错误或成功页面。 |
| `modcp_body.tpl` | 管理员控制面板中的主题列表。 |
| `modcp_move.tpl` | 主题移动确认和论坛列表。 |
| `modcp_split.tpl` | 主题拆分帖子列表。 |
| `modcp_viewip.tpl` | 与帖子相关的 IP 地址列表和用户。 |
| `overall_footer.tpl` | 正常页面页脚，例如版权详情。 |
| `overall_header.tpl` | 正常页面页眉和网站菜单。 |
| `posting_body.tpl` | 发布表单。 |
| `posting_poll_body.tpl` | 发布表单的区域。 |
| `posting_preview.tpl` | 发布表单的预览区域。 |
| `posting_smilies.tpl` | 从发布表单可访问的弹出表情窗口。 |
| `posting_topic_review.tpl` | 发布表单的主题回顾区域。 |
| `privmsgs_body.tpl` | 私信文件夹列表。 |
| `privmsgs_popup.tpl` | 私信弹出通知窗口。 |
| `privmsgs_preview.tpl` | 私信发布表单的预览区域。 |
| `privmsgs_read_body.tpl` | 私信文本。 |
| `profile_add_body.tpl` | 注册和编辑个人资料表单。 |
| `profile_avatar_gallery.tpl` | 头像画廊子分类。 |
| `profile_send_email.tpl` | 发送电子邮件表单。 |
| `profile_send_pass.tpl` | 失去密码重置表单。 |
| `profile_view_body.tpl` | 用户个人资料页面。 |
| `search_body.tpl` | 完整的搜索表单。 |
| `search_results_posts.tpl` | 以帖子形式显示的搜索结果。 |
| `search_results_topics.tpl` | 以主题形式显示的搜索结果。 |
| `search_username.tpl` | 弹出用户搜索窗口。 |
| `simple_footer.tpl` | 较小的页面页脚，通常在弹出窗口中。 |
| `simple_header.tpl` | 较小的页面页眉，不带网站菜单。 |
| `viewforum_body.tpl` | 论坛中的主题列表。 |
| `viewonline_body.tpl` | 当前在论坛上的用户列表及其访问的页面。 |
| `viewtopic_body.tpl` | 主题内的帖子。 |
| `viewtopic_poll_ballot.tpl` | 主题中的投票表单。 |
| `viewtopic_poll_result.tpl` | 主题中的投票结果列表。 |

每个模板的目录也可以包含模板作者希望包含的任意数量的子目录。唯一必需的子目录是`admin`，例如`templates/subSilver/admin`。此目录将包含用于显示管理控制面板的模板文件，如下表所示。

| 文件名 | 用于显示 |
| --- | --- |
| `admin_message_body.tpl` | 错误和成功消息。 |
| `auth_forum_body.tpl` | 论坛权限：权限编辑器。 |
| `auth_select_body.tpl` | 权限：选择要编辑的论坛或组。 |
| `auth_ug_body.tpl` | 编辑用户或用户组的权限。 |
| `board_config_body.tpl` | 通用分类：配置页面。 |
| `category_edit_body.tpl` | 编辑论坛分类的名称和其他详细信息。 |
| `db_utils_backup_body.tpl` | 数据库实用工具备份表单。 |
| `db_utils_restore_body.tpl` | 数据库实用工具恢复表单。 |
| `disallow_body.tpl` | 禁止使用的用户名页面。 |
| `forum_admin_body.tpl` | 基本论坛详细信息编辑页面。 |
| `forum_delete_body.tpl` | 删除论坛或分类。 |
| `forum_edit_body.tpl` | 编辑论坛的名称和其他详细信息。 |
| `forum_prune_body.tpl` | 论坛修剪区域。 |
| `forum_prune_result_body.tpl` | 完成论坛修剪的详细信息。 |
| `forum_prune_select_body.tpl` | 选择要修剪的论坛。 |
| `group_edit_body.tpl` | 编辑或创建用户组。 |
| `group_select_body.tpl` | 选择要编辑的用户组。 |
| `index_body.tpl` | 管理面板入口页面。 |
| `index_frameset.tpl` | 管理面板框架布局。 |
| `index_navigate.tpl` | 管理面板导航菜单。 |
| `page_footer.tpl` | 页面页脚。 |
| `page_header.tpl` | 页面标题和样式表。 |
| `ranks_edit_body.tpl` | 编辑用户排名的设置或创建排名。 |
| `ranks_list_body.tpl` | 列出所有当前排名。 |
| `smile_edit_body.tpl` | 添加或修改表情符号的设置。 |
| `smile_import_body.tpl` | 从 `pak` 文件中安装表情符号。 |
| `smile_list_body.tpl` | 列出所有已安装的表情符号。 |
| `styles_addnew_body.tpl` | 安装新样式；列出所有未安装的样式。 |
| `styles_edit_body.tpl` | 创建或编辑主题/样式。 |
| `styles_exporter.tpl` | 导出模板的主题设置。 |
| `styles_list_body.tpl` | 列出所有已安装的样式。 |
| `user_avatar_gallery.tpl` | 头像画廊；用于编辑用户的个人资料。 |
| `user_ban_body.tpl` | 禁用用户工具。 |
| `user_edit_body.tpl` | 编辑用户的个人资料信息。 |
| `user_email_body.tpl` | 向用户发送大量电子邮件。 |
| `user_select_body.tpl` | 选择一个用户进行编辑。 |
| `words_edit_body.tpl` | 添加或编辑被屏蔽的词。 |
| `words_list_body.tpl` | 列出被屏蔽的词。 |

另一个常见的子目录是 `images`，它可能还包含更多子目录，通常以语言包命名，例如 `lang_english`。模板目录内的语言目录通常包含带有文本的图像，如回复或发帖按钮。

# 可选目录

除了你刚刚学到的必需目录外，phpBB 论坛还包括一些可选目录。其中一些可能对论坛本身有益或无用。一般来说，如果你不打算在你的网站上使用它们，删除这些目录及其内容是个好主意。

## 缓存页面

加速论坛的一种流行方法是添加缓存系统。**缓存系统**在页面被查看时存储由 phpBB 模板系统生成的代码，然后在页面再次被查看时使用该存储的代码。速度提升来自于引用存储的代码；这比每次查看页面时重新生成代码要快。存储的代码通常保存在 `cache` 目录下的文件中。phpBB 默认不使用缓存系统，但包含了一些可选的系统，你稍后会看到。为使用这些选项之一，提供了一个 `cache` 目录。

## phpBB 文档

phpBB 的 `docs` 目录包含文档文件。通过查看这些文件，你可以找到诸如安装和升级说明、phpBB 发布的许可证副本以及有关使用 phpBB 的常见问题解答等信息。这些文件在论坛中没有任何用途，因此可以安全地删除此目录。保留你自己的电脑上的文件副本，以防将来需要引用它们。

## 图片

`images`目录通常用作存储表情符号、等级和头像的图片文件的地方。你可以在论坛目录结构中的其他位置放置这些文件，但这样做通常需要在管理控制面板中更改一些配置设置。phpBB 在`images`目录内提供了`smilies`（包含默认表情符号）和`avatars`（头像）子目录。

# 其他值得注意的目录

在 phpBB 的结构中，除了可选的和必需的目录之外，还有一些其他的目录。这些目录有些独特。phpBB 并不绝对需要它们，但有时你真的没有选择是否包含它们。它们绝对值得仔细研究。

## 贡献的额外内容

phpBB 附带了一个特殊的`contrib`目录。它包含了一些你可以选择安装到论坛上的额外插件。要了解这些插件，请阅读浏览器中的`README.html`文件。包含的插件随 phpBB 的每个版本而变化，但你可以期待看到之前提到的模板缓存系统等。除非你主动将这些插件安装到论坛上，否则 phpBB 不会使用此目录中的任何插件或其他文件。

与`install`目录类似，如果在你网站上保留`contrib`目录，phpBB 将会显示错误信息。为了让人们使用你的论坛，你必须删除`contrib`目录及其中的所有文件。

## 修改文件

到目前为止，你已经了解了 phpBB 附带的所有目录和文件。在阅读过程中，你可能已经注意到一些部分提到修改可能会在特定区域添加文件。事实是，修改可以对论坛的目录结构做任何事情，包括添加新的目录或文件，移动现有的文件，甚至完全删除一些文件。

许多修改作者已经采用了一个共同的标准，即将新文件放置在根目录内添加的`mods`目录中。phpBB 在下载时并不包含`mods`目录；这是一个社区创建的概念。`mods`目录通常为每个添加文件的修改都有一个子目录，例如`prillian`用于 Prillian 消息传递修改，`phpbb_fetch_all`用于 phpBB Fetch All 聚合修改。使用`mods`来存放新文件可以使它们紧密分组并易于查找。

并非所有修改创建者都会使用`mods`。有些人会在论坛结构中的其他位置放置新的目录。如果你创建修改，你应该将新文件和目录放置在你感到舒适的位置。除非你安装使用`mods`目录的修改，否则你不需要创建它。如果这些目录不存在，你需要创建安装的任何修改所使用的目录。

### 注意

新的语言文件几乎总是放置在`language`目录中，即使在那些使用`mods`目录的作者中也是如此。同样，模板文件通常也放在`templates`目录中。当这些文件存储在这些区域时，跟踪和使用这些文件会更加容易。
