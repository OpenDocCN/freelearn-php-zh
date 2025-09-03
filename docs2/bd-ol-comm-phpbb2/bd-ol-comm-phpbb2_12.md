# 附录 D. phpBB 数据库结构

本附录描述了所有 phpBB 数据库表，列出了它们的字段、关系以及简要说明每个表字段的注释。正如您所知，phpBB 数据库表可以有前缀，这是在安装时由您指定的。在本附录中，使用了默认的 `phpbb_` 前缀。本附录是在 phpMyAdmin 的帮助下创建的（[`www.phpmyadmin.net`](http://www.phpmyadmin.net)）。

# phpbb_auth_access

此表定义了组（和用户）权限。

| Field | Type | Null | Default | Comments |
| --- | --- | --- | --- | --- |
| group_id | mediumint(8) | No | 0 | 来自用户组的组 ID。每个用户都是某个组的成员，即使该用户是该组的唯一成员。从这个意义上讲，只有组权限在此表中设置。**`链接到：phpbb_groups -> group_id`** |
| forum_id | smallint(5) | No | 0 | 来自论坛表的论坛 ID。**`链接到：phpbb_forums -> forum_id`** |
| auth_view | tinyint(1) | No | 0 | 此组是否允许查看具有指定论坛 ID 的论坛？“查看”意味着看到该论坛存在。值是 `0` 表示“否”，`1` 表示“是”。其余字段使用相同的值。 |
| auth_read | tinyint(1) | No | 0 | 此组是否允许阅读论坛？ |
| auth_post | tinyint(1) | No | 0 | 该组是否允许在论坛中创建主题？ |
| auth_reply | tinyint(1) | No | 0 | 此组是否允许回复现有主题？ |
| auth_edit | tinyint(1) | No | 0 | 此组是否允许编辑自己的帖子？ |
| auth_delete | tinyint(1) | No | 0 | 此组是否允许删除自己的帖子？ |
| auth_sticky | tinyint(1) | No | 0 | 此组是否允许创建置顶主题？ |
| auth_announce | tinyint(1) | No | 0 | 此组是否允许发布公告？ |
| auth_vote | tinyint(1) | No | 0 | 此组是否允许在现有投票中投票？ |
| auth_pollcreate | tinyint(1) | No | 0 | 此组是否允许创建新的投票？ |
| auth_attachments | tinyint(1) | No | 0 | 此组是否允许将文件附加到帖子中（在标准 phpBB 版本中未使用）。 |
| auth_mod | tinyint(1) | No | 0 | 此组是否允许管理论坛？ |

# phpbb_banlist

此表包含被禁止的 IP 地址、用户 ID 或电子邮件地址。

| Field | Type | Null | Default | Comments |
| --- | --- | --- | --- | --- |
| ban_id | mediumint(8) | No | - | 自动递增。 |
| ban_userid | mediumint(8) | No | 0 | 根据用户表定义的被禁止用户的 ID。**`链接到：phpbb_users -> user_id`** |
| ban_ip | varchar(8) | No | - | 被禁止的 IP 地址（加密）。 |
| ban_email | varchar(255) | Yes | NULL | 被禁止的电子邮件地址。 |

# phpbb_categories

此表存储论坛类别。

| Field | Type | Null | Default | Comments |
| --- | --- | --- | --- | --- |
| cat_id | mediumint(8) | No | - | 自动递增。 |
| cat_title | varchar(100) | Yes | NULL | 类别的标题。 |
| cat_order | mediumint(8) | No | 0 | 类别的显示顺序。 |

# phpbb_config

此表存储了论坛配置。

| 字段 | 类型 | 可空 | 默认值 | 备注 |
| --- | --- | --- | --- | --- |
| config_name | 可变字符(255) | 否 | - | 配置变量的名称。 |
| config_value | 可变字符(255) | 否 | - | 配置变量的值。 |

# phpbb_confirm

在此 phpBB 版本中未使用。

| 字段 | 类型 | 可空 | 默认值 | 备注 |
| --- | --- | --- | --- | --- |
| confirm_id | 字符(32) | 否 | - | - |
| session_id | 字符(32) | 否 | - | - |
| code | 字符(6) | 否 | - | - |

# phpbb_disallow

在此表中定义的用户名在注册时是不允许的。

| 字段 | 类型 | 可空 | 默认值 | 备注 |
| --- | --- | --- | --- | --- |
| 不允许 ID | 中型整型(8) | 否 |  | 自动递增。 |
| 不允许的用户名 | 可变字符(25) | 否 |  | 不允许作为用户名的字符串。 |

# phpbb_forum_prune

此表定义了论坛修剪的规则。

| 字段 | 类型 | 可空 | 默认值 | 备注 |
| --- | --- | --- | --- | --- |
| prune_id | 中型整型(8) | 否 |  | 自动递增。 |
| forum_id | 小型整型(5) | 否 | 0 | 来自论坛表的 ID。**`链接到：phpbb_forums -> forum_id`** |
| prune_days | 小型整型(5) | 否 | 0 | 主题应有多旧才能被删除。 |
| prune_freq | 小型整型(5) | 否 | 0 | phpBB 应多久检查一次要删除的主题。`1`的值表示“每天”。 |

# phpbb_forums

此表包含有关论坛的信息。

| 字段 | 类型 | 可空 | 默认值 | 备注 |
| --- | --- | --- | --- | --- |
| 论坛 ID | 小型整型(5) | 否 | 0 | 自动递增。 |
| cat_id | 中型整型(8) | 否 | 0 | 来自分类表的 ID。此论坛所属的分类。**`链接到：phpbb_categories -> cat_id`** |
| forum_name | 可变字符(150) | 是 | NULL | 论坛标题。 |
| 论坛描述 | 文本 | 是 | NULL | 论坛描述。 |
| 论坛状态 | 小型整型(4) | 否 | 0 | 论坛状态（0 为启用，`1`表示禁用）。 |
| forum_order | 中型整型(8) | 否 | 1 | 相对于同一类别的其他论坛的显示顺序。 |
| forum_posts | 中型整型(8) | 否 | 0 | 到目前为止此论坛中的帖子数。 |
| 论坛主题 | 中型整型(8) | 否 | 0 | 本论坛的主题总数。 |
| forum_last_post_id | 中型整型(8) | 否 | 0 | 来自帖子表的最后帖子的 ID。**`链接到：phpbb_posts -> post_id`** |
| prune_next | 整型(11) | 是 | NULL | 下次论坛修剪的日期（Unix 时间戳）。 |
| prune_enable | 小型整型(1) | 否 | 0 | 是否为该论坛启用修剪？ |
| auth_view | 小型整型(2) | 否 | 0 | 允许查看的权限级别（谁可以查看）。`0`是全部，`1`是注册用户，`2`是私有，`3`是版主。这些值适用于本表中其余列。 |
| auth_read | 小型整型(2) | 否 | 0 | 谁可以阅读论坛？查看`auth_view`以获取允许的值。 |
| auth_post | 小型整型(2) | 否 | 0 | 谁可以在论坛中创建主题？查看`auth_view`以获取允许的值。 |
| auth_reply | tinyint(2) | No | 0 | 谁可以在论坛中回复现有主题？请参阅`auth_view`以获取允许的值。 |
| auth_edit | tinyint(2) | No | 0 | 谁可以在论坛中编辑帖子？请参阅`auth_view`以获取允许的值。 |
| auth_delete | tinyint(2) | No | 0 | 谁可以删除论坛中的帖子？请参阅`auth_view`以获取允许的值。 |
| auth_sticky | tinyint(2) | No | 0 | 谁可以在论坛中创建置顶主题？请参阅`auth_view`以获取允许的值。 |
| auth_announce | tinyint(2) | No | 0 | 谁可以在论坛中创建公告主题？请参阅`auth_view`以获取允许的值。 |
| auth_vote | tinyint(2) | No | 0 | 谁可以在论坛中的投票中投票？请参阅`auth_view`以获取允许的值。 |
| auth_pollcreate | tinyint(2) | No | 0 | 谁可以在论坛中创建新的投票？请参阅`auth_view`以获取允许的值。 |
| auth_attachments | tinyint(2) | No | 0 | 谁可以在论坛中的帖子中附加文件？在此库存 phpBB 版本中未使用。 |

# phpbb_groups

此表包含用户组信息。

| Field | Type | Null | Default | 评论 |
| --- | --- | --- | --- | --- |
| group_id | mediumint(8) | No | - | 自增。 |
| group_type | tinyint(4) | No | 1 | 组类型。`0`是公开组，`1`是封闭组，`2`是隐藏组。 |
| group_name | varchar(40) | No | - | 组名称。 |
| group_description | varchar(255) | No | - | 组描述。 |
| group_moderator | mediumint(8) | No | 0 | 组管理员的 ID。对于个人用户组为`0`。**`链接到：phpbb_users -> user_id`** |
| group_single_user | tinyint(1) | No | 1 | 如果此组是单个用户的个人组，则为`1`；如果是成员更多的正常组，则为`0`。 |

# phpbb_posts

此表包含关于帖子的信息。

| Field | Type | Null | Default | 评论 |
| --- | --- | --- | --- | --- |
| post_id | mediumint(8) | No | - | 自增。 |
| topic_id | mediumint(8) | No | 0 | 来自主题表的主题 ID。**`链接到：phpbb_topics -> topic_id`** |
| forum_id | smallint(5) | No | 0 | 来自论坛表的论坛 ID。**`链接到：phpbb_forums -> forum_id`** |
| poster_id | mediumint(8) | No | 0 | 来自用户表的用户 ID。**`链接到：phpbb_users -> user_id`** |
| post_time | int(11) | No | 0 | 帖子时间作为 Unix 时间戳。 |
| poster_ip | varchar(8) | No | - | 发布帖子的用户的编码 IP 地址。 |
| post_username | varchar(25) | Yes | NULL | 发布者的用户名。仅适用于匿名发布者。如果发布者已登录，则值为 NULL。此外，当您删除用户时，所有帖子都会显示为匿名发布，phpBB 会在用户删除时为所有用户帖子填充此字段。 |
| enable_bbcode | tinyint(1) | No | 1 | 发帖者设置的选项，用于确定是否应处理并替换为 HTML 的任何 BBcode。 |
| enable_html | tinyint(1) | No | 0 | 与上述类似，但针对的是启用/禁用 HTML 选项。 |
| enable_smilies | tinyint(1) | No | 1 | 与上面相同，但与是否解析表情符号或仅保留表情符号代码有关。 |
| enable_sig | tinyint(1) | No | 1 | 是否将发帖者的签名附加到帖子中。 |
| post_edit_time | int(11) | Yes | NULL | 帖子最后编辑的时间（Unix 时间戳）。 |
| post_edit_count | smallint(5) | No | 0 | 该帖子被编辑的次数。 |

# phpbb_posts_text

该表包含帖子的实际文本。

| 字段 | 类型 | 可空 | 默认值 | 备注 |
| --- | --- | --- | --- | --- |
| post_id | mediumint(8) | No | 0 | 从帖子表中获取的帖子 ID。**`链接到：phpbb_posts -> post_id`** |
| bbcode_uid | varchar(10) | No | - | BBCode 处理器使用的标识字符串。 |
| post_subject | varchar(60) | Yes | NULL | 帖子的主题。 |
| post_text | text | Yes | NULL | 帖子的文本。 |

# phpbb_privmsgs

该表包含关于私人消息的数据。

| 字段 | 类型 | 可空 | 默认值 | 备注 |
| --- | --- | --- | --- | --- |
| privmsgs_id | mediumint(8) | No | - | 自动递增。 |
| privmsgs_type | tinyint(4) | No | 0 | 消息的类型。从`0`到`5`的整数值，其中`0`是已读消息，`1`是新消息，`2`是已发送消息，`3`是已保存，`4`是已保存的发送消息，`5`是未读。 |
| privmsgs_subject | varchar(255) | No | 0 | 消息的主题。 |
| privmsgs_from_userid | mediumint(8) | No | 0 | 发送者的用户 ID。**`链接到：phpbb_users -> user_id`** |
| privmsgs_to_userid | mediumint(8) | No | 0 | 收件人的用户 ID。**`链接到：phpbb_users -> user_id`** |
| privmsgs_date | int(11) | No | 0 | 消息发送时的 Unix 时间戳。 |
| privmsgs_ip | varchar(8) | No | - | 发送者的编码 IP 地址。 |
| privmsgs_enable_bbcode | tinyint(1) | No | 1 | 发送者是否为这条消息启用了 BBCode？ |
| privmsgs_enable_html | tinyint(1) | No | 0 | 发送者是否为这条消息启用了 HTML 代码？ |
| privmsgs_enable_smilies | tinyint(1) | No | 1 | 发送者是否为这条消息启用了表情符号？ |
| privmsgs_attach_sig | tinyint(1) | No | 1 | 发送者是否勾选了将签名附加到消息中的字段？ |

# phpbb_privmsgs_text

该表包含私人消息的文本。

| 字段 | 类型 | 可空 | 默认值 | 备注 |
| --- | --- | --- | --- | --- |
| privmsgs_text_id | mediumint(8) | No | 0 | 从私人消息表中获取的 ID。**`链接到：phpbb_privmsgs -> privmsgs_id`** |
| privmsgs_bbcode_uid | varchar(10) | No | 0 | BBCode 解析器所需的标识字符串。 |
| privmsgs_text | text | Yes | NULL | 私人消息的文本。 |

# phpbb_ranks

该表包含用户等级的定义。

| 字段 | 类型 | 可空 | 默认值 | 备注 |
| --- | --- | --- | --- | --- |
| rank_id | smallint(5) | No | - | 自动递增。 |
| rank_title | varchar(50) | No | - | 等级的标题。 |
| rank_min | mediumint(8) | No | 0 | 该等级所需的帖子最小数量。 |
| rank_special | tinyint(1) | Yes | 0 | 对于特殊等级为 `1`，否则为 `0`。 |
| rank_image | varchar(255) | Yes | NULL | 等级图像文件名和路径，相对于 phpBB 安装的位置。 |

# phpbb_search_results

此表包含搜索结果数据。

| 字段 | 类型 | Null | 默认 | 备注 |
| --- | --- | --- | --- | --- |
| search_id | int(11) | No | 0 | 搜索标识符。 |
| session_id | varchar(32) | No | - | 会话标识符，来自 sessions 表。**`链接到: phpbb_sessions -> session_id`** |
| search_array | text | No | - | 序列化的搜索结果数据数组。 |

# phpbb_search_wordlist

此表包含论坛中使用的词汇。

| 字段 | 类型 | Null | 默认 | 备注 |
| --- | --- | --- | --- | --- |
| word_text | varchar(50) | No | - | 实际的词。 |
| word_id | mediumint(8) | No | - | 自动递增。 |
| word_common | tinyint(1) | No | 0 | 对于常用词为 `1`，否则为 `0`。 |

# phpbb_search_wordmatch

此表存储有关词使用位置的信息。

| 字段 | 类型 | Null | 默认 | 备注 |
| --- | --- | --- | --- | --- |
| post_id | mediumint(8) | No | 0 | 来自 postings 表的帖子 ID。**`链接到: phpbb_posts -> post_id`** |
| word_id | mediumint(8) | No | 0 | 来自搜索词列表表的词 ID。**`链接到: phpbb_search_wordslist -> word_id`** |
| title_match | tinyint(1) | No | 0 | 如果词出现在帖子标题中则为 `1`，否则为 `0`。 |

# phpbb_sessions

此表存储有关当前会话的数据。

| 字段 | 类型 | Null | 默认 | 备注 |
| --- | --- | --- | --- | --- |
| session_id | char(32) | No | - | 唯一会话 ID。 |
| session_user_id | mediumint(8) | No | 0 | 来自 users 表的用户 ID，如果会话属于未登录用户则为 `-1`。**`链接到: phpbb_users -> user_id`** |
| session_start | int(11) | No | 0 | 会话开始时的 Unix 时间戳。 |
| session_time | int(11) | No | 0 | 最后访问页面的 Unix 时间戳。 |
| session_ip | char(8) | No | 0 | 用户的编码 IP 地址。 |
| session_page | int(11) | No | 0 | 用户当前查看的页面标识符。页面在 `includes/constants.php` 文件中定义为带有减号的常量。具有非负整数值的页面实际上是论坛 ID。 |
| session_logged_in | tinyint(1) | No | 0 | 如果会话属于已登录用户则为 `1`，否则为 `0`。 |

# phpbb_smilies

表情符号存储库。

| 字段 | 类型 | Null | 默认 | 备注 |
| --- | --- | --- | --- | --- |
| smilies_id | smallint(5) | No | - | 自动递增。 |
| code | varchar(50) | Yes | NULL | 表情符号代码（如 “:)”）。 |
| smile_url | varchar(100) | Yes | NULL | 表情图像的文件名。 |
| emoticon | varchar(75) | Yes | NULL | 表情符号代表的情感类型，例如快乐、烦躁等。 |

# phpbb_themes

此表包含主题设置。

| 字段 | 类型 | Null | 默认 | 备注 |
| --- | --- | --- | --- | --- |
| themes_id | mediumint(8) | No | - | 自动递增。 |
| template_name | varchar(30) | 否 | - | 模板的名称。 |
| style_name | varchar(30) | 否 | - | 样式的名称。 |
| head_stylesheet | varchar(100) | 是 | NULL | 样式表的文件名（.css）。 |
| body_background | varchar(100) | 是 | NULL | 作为页面背景使用的任何图像的文件名。 |
| body_bgcolor | varchar(6) | 是 | NULL | 页面背景的十六进制（Hex）颜色代码，例如‘ff00ff’。注意颜色代码前没有井号（#）。 |
| body_text | varchar(6) | 是 | NULL | 页面上使用的文本颜色的十六进制代码。 |
| body_link | varchar(6) | 是 | NULL | 链接的颜色。 |
| body_vlink | varchar(6) | 是 | NULL | 访问链接的颜色。 |
| body_alink | varchar(6) | 是 | NULL | 激活链接的颜色。 |
| body_hlink | varchar(6) | 是 | NULL | 鼠标悬停链接的颜色（当鼠标悬停在链接上时）。 |
| tr_color1 | varchar(6) | 是 | NULL | 这些字段的描述在`phpbb_themes_name`表中。 |
| tr_color2 | varchar(6) | 是 | NULL |   |
| tr_color3 | varchar(6) | 是 | NULL |   |
| tr_class1 | varchar(25) | 是 | NULL |   |
| tr_class2 | varchar(25) | 是 | NULL |   |
| tr_class3 | varchar(25) | 是 | NULL |   |
| th_color1 | varchar(6) | 是 | NULL |   |
| th_color2 | varchar(6) | 是 | NULL |   |
| th_color3 | varchar(6) | 是 | NULL |   |
| th_class1 | varchar(25) | 是 | NULL |   |
| th_class2 | varchar(25) | 是 | NULL |   |
| th_class3 | varchar(25) | 是 | NULL |   |
| td_color1 | varchar(6) | 是 | NULL |   |
| td_color2 | varchar(6) | 是 | NULL |   |
| td_color3 | varchar(6) | 是 | NULL |   |
| td_class1 | varchar(25) | 是 | NULL |   |
| td_class2 | varchar(25) | 是 | NULL |   |
| td_class3 | varchar(25) | 是 | NULL |   |
| fontface1 | varchar(50) | 是 | NULL |   |
| fontface2 | varchar(50) | 是 | NULL |   |
| fontface3 | varchar(50) | 是 | NULL |   |
| fontsize1 | tinyint(4) | 是 | NULL |   |
| fontsize2 | tinyint(4) | 是 | NULL |   |
| fontsize3 | tinyint(4) | 是 | NULL |   |
| fontcolor1 | varchar(6) | 是 | NULL | 这些字段的描述在`phpbb_themes_name`表中。 |
| fontcolor2 | varchar(6) | 是 | NULL |   |
| fontcolor3 | varchar(6) | 是 | NULL |   |
| span_class1 | varchar(25) | 是 | NULL |   |
| span_class2 | varchar(25) | 是 | NULL |   |
| span_class3 | varchar(25) | 是 | NULL |   |
| img_size_poll | smallint(5) | 是 | NULL | 未使用。 |
| img_size_privmsg | smallint(5) | 是 | NULL | 未使用。 |

# phpbb_themes_name

在主题表中设置的元素名称。

| 字段 | 类型 | 可空 | 默认值 | 备注 |
| --- | --- | --- | --- | --- |
| themes_id | smallint(5) | 否 | 0 | - |
| tr_color1_name | char(50) | 是 | NULL | - |
| tr_color2_name | char(50) | 是 | NULL | - |
| tr_color3_name | char(50) | 是 | NULL | - |
| tr_class1_name | char(50) | 是 | NULL | - |
| tr_class2_name | char(50) | 是 | NULL | - |
| tr_class3_name | char(50) | 是 | NULL | - |
| th_color1_name | char(50) | Yes | NULL | - |
| th_color2_name | char(50) | Yes | NULL | - |
| th_color3_name | char(50) | Yes | NULL | - |
| th_class1_name | char(50) | Yes | NULL | - |
| th_class2_name | char(50) | Yes | NULL | - |
| th_class3_name | char(50) | Yes | NULL | - |
| td_color1_name | char(50) | Yes | NULL | - |
| td_color2_name | char(50) | Yes | NULL | - |
| td_color3_name | char(50) | Yes | NULL | - |
| td_class1_name | char(50) | Yes | NULL | - |
| td_class2_name | char(50) | Yes | NULL | - |
| td_class3_name | char(50) | Yes | NULL | - |
| fontface1_name | char(50) | Yes | NULL | - |
| fontface2_name | char(50) | Yes | NULL | - |
| fontface3_name | char(50) | Yes | NULL | - |
| fontsize1_name | char(50) | Yes | NULL | - |
| fontsize2_name | char(50) | Yes | NULL | - |
| fontsize3_name | char(50) | Yes | NULL | - |
| fontcolor1_name | char(50) | Yes | NULL | - |
| fontcolor2_name | char(50) | Yes | NULL | - |
| fontcolor3_name | char(50) | Yes | NULL | - |
| span_class1_name | char(50) | Yes | NULL | - |
| span_class2_name | char(50) | Yes | NULL | - |
| span_class3_name | char(50) | Yes | NULL | - |

# phpbb_topics

此表包含有关主题的数据。

| Field | Type | Null | Default | Comments |
| --- | --- | --- | --- | --- |
| topic_id | mediumint(8) | No | - | 自增。 |
| forum_id | smallint(8) | No | 0 | 来自论坛表的论坛 ID。**`链接到: phpbb_forums -> forum_id`** |
| topic_title | char(60) | No | - | 主题标题。 |
| topic_poster | mediumint(8) | No | 0 | 开始主题的用户 ID。**`链接到: phpbb_users -> user_id`** |
| topic_time | int(11) | No | 0 | 主题开始时的 Unix 时间戳。 |
| topic_views | mediumint(8) | No | 0 | 主题被查看的次数。 |
| topic_replies | mediumint(8) | No | 0 | 主题中的回复数量。 |
| topic_status | tinyint(3) | No | 0 | 主题的状态。`0`为正常，`1`为锁定，`2`为已移动。 |
| topic_vote | tinyint(1) | No | 0 | 如果主题附加了投票，则为`1`，否则为`0`。 |
| topic_type | tinyint(3) | No | 0 | `0`为普通主题，`1`为置顶主题，`2`为公告。 |
| topic_first_post_id | mediumint(8) | No | 0 | 本主题第一帖子的 ID。**`链接到: phpbb_posts -> post_id`** |
| topic_last_post_id | mediumint(8) | No | 0 | 本主题最后帖子的 ID。**`链接到: phpbb_posts -> post_id`** |
| topic_moved_id | mediumint(8) | No | 0 | 如果这是一个仅作为占位符保留在论坛中的服务主题，则此字段包含已移动主题的 ID。**`链接到: phpbb_topics -> topic_id`** |

# phpbb_topics_watch

此表包含正在监视特定主题的用户列表。“监视”意味着用户已请求在监视的主题中发布回复时通知他们。

| Field | Type | Null | Default | Comments |
| --- | --- | --- | --- | --- |
| topic_id | mediumint(8) | No | 0 | 正在关注的主题 ID（在主题表中定义）。**`链接到: phpbb_topics -> topic_id`** |
| user_id | mediumint(8) | No | 0 | 来自用户表的用户 ID，观察者。**`链接到: phpbb_users -> user_id`** |
| notify_status | tinyint(1) | No | 0 | 如果用户被通知，则为`1`，否则为`0`。 |

# phpbb_user_group

包含组和组用户的交换板表。

| 字段 | 类型 | Null | 默认值 | 备注 |
| --- | --- | --- | --- | --- |
| group_id | mediumint(8) | No | 0 | 来自组表的组 ID。**`链接到: phpbb_groups -> group_id`** |
| user_id | mediumint(8) | No | 0 | 来自用户表的用户 ID。**`链接到: phpbb_users -> user_id`** |
| user_pending | tinyint(1) | Yes | NULL | 如果用户处于待定状态，则为`1`，否则为`0`。 |

# phpbb_users

用户仓库。

| 字段 | 类型 | Null | 默认值 | 备注 |
| --- | --- | --- | --- | --- |
| user_id | mediumint(8) | No | 0 | 自增。 |
| user_active | tinyint(1) | Yes | 1 | 用户是否活跃？`1`为是，`0`为否。 |
| username | varchar(25) | No | - | 用户名。 |
| user_password | varchar(32) | No |  | 密码。 |
| user_session_time | int(11) | No | 0 | 用户最后访问的页面时间戳。 |
| user_session_page | smallint(5) | No | 0 | 用户最后查看的页面。 |
| user_lastvisit | int(11) | No | 0 | 最后登录时间戳。 |
| user_regdate | int(11) | No | 0 | 注册日期的 Unix 时间戳。 |
| user_level | tinyint(4) | Yes | 0 | `0`为普通用户，`1`为管理员，`2`为版主。 |
| user_posts | mediumint(8) | No | 0 | 该用户发布的帖子总数。 |
| user_timezone | decimal(5,2) | No | 0.00 | 用户的时区。 |
| user_style | tinyint(4) | Yes | NULL | 从主题表中选择的样式 ID。**`链接到: phpbb_themes -> themes_id`** |
| user_lang | varchar(255) | Yes | NULL | 用户选择的语言。 |
| user_dateformat | varchar(14) | No | d M Y H:i | 在论坛上显示的所有日期的格式。 |
| user_new_privmsg | smallint(5) | No | 0 | 新私信总数。 |
| user_unread_privmsg | smallint(5) | No | 0 | 未读私信总数。 |
| user_last_privmsg | int(11) | No | 0 | 收到最新私信的 Unix 时间戳。 |
| user_emailtime | int(11) | Yes | NULL | 使用 phpBB 电子邮件功能发送给用户的最后电子邮件的 Unix 时间戳。 |
| user_viewemail | tinyint(1) | Yes | NULL | 如果此用户决定公开电子邮件，则为`1`，否则为`0`。 |
| user_attachsig | tinyint(1) | Yes | NULL | 如果用户的签名默认附加，则为`1`，否则为`0`。 |
| user_allowhtml | tinyint(1) | Yes | 1 | 如果用户决定在帖子中默认启用 HTML，则为`1`，否则为`0`。 |
| user_allowbbcode | tinyint(1) | Yes | 1 | 如果用户决定在帖子中默认启用 BBCode，则为`1`，否则为`0`。 |
| user_allowsmile | tinyint(1) | 是 | 1 | 如果用户决定在帖子中默认启用表情符号，则为`1`，否则为`0`。 |
| user_allowavatar | tinyint(1) | 否 | 1 | 如果允许此用户拥有头像，则为`1`。 |
| user_allow_pm | tinyint(1) | 否 | 1 | 如果允许此用户使用私人消息系统，则为`1`。 |
| user_allow_viewonline | tinyint(1) | 否 | 1 | 如果用户同意使其状态（在线/离线）公开，则为`1`。 |
| user_notify | tinyint(1) | 否 | 1 | 如果用户选择通过电子邮件通知回复，则为`1`，否则为`0`。 |
| user_notify_pm | tinyint(1) | 否 | 0 | 如果用户选择通过电子邮件通知任何新的私人消息，则为`1`，否则为`0`。 |
| user_popup_pm | tinyint(1) | 否 | 0 | 如果应通过新弹出窗口通知用户新的私人消息，则为`1`，否则为`0`。 |
| user_rank | int(11) | 是 | 0 | 分配给用户的等级 ID。**`链接到：phpbb_ranks -> rank_id`** |
| user_avatar | varchar(100) | 是 | NULL | 用户的头像。用户头像图像的 URL 或服务器路径和文件名。 |
| user_avatar_type | tinyint(4) | 否 | 0 | 头像类型。`0`表示无，`1`表示上传，`2`表示远程，`3`表示相册。 |
| user_email | varchar(255) | 是 | NULL | 用户的电子邮件地址。 |
| user_icq | varchar(15) | 是 | NULL | 用户的 ICQ 号码。 |
| user_website | varchar(100) | 是 | NULL | 用户网站的 URL。 |
| user_from | varchar(100) | 是 | NULL | 用户的位置。 |
| user_sig | text | 是 | NULL | 用户的签名。 |
| user_sig_bbcode_uid | varchar(10) | 是 | NULL | 签名 BBCode 标识符。 |
| user_aim | varchar(255) | 是 | NULL | 用户 AOL 即时通讯 ID。 |
| user_yim | varchar(255) | 是 | NULL | 用户 Yahoo!即时通讯 ID。 |
| user_msnm | varchar(255) | 是 | NULL | 用户 MSN 即时通讯 ID。 |
| user_occ | varchar(100) | 是 | NULL | 用户的职业。 |
| user_interests | varchar(255) | 是 | NULL | 用户的兴趣。 |
| user_actkey | varchar(32) | 是 | NULL | 激活密钥。 |
| user_newpasswd | varchar(32) | 是 | NULL | 新的临时密码。 |

# phpbb_vote_desc

此表包含与投票相关的数据。

| Field | Type | Null | Default | 备注 |
| --- | --- | --- | --- | --- |
| vote_id | mediumint(8) | 否 | - | 自动递增。 |
| topic_id | mediumint(8) | 否 | 0 | 来自主题表的主题 ID。**`链接到：phpbb_topics -> topic_id`** |
| vote_text | text | 否 |  | 投票问题。 |
| vote_start | int(11) | 否 | 0 | 投票开始时的 Unix 时间戳。 |
| vote_length | int(11) | 否 | 0 | 此投票应活跃的时间（以毫秒为单位）。 |

# phpbb_vote_results

投票选项答案和投票结果。

| Field | Type | Null | Default | 备注 |
| --- | --- | --- | --- | --- |
| vote_id | mediumint(8) | 否 | 0 | 来自`vote_desc`表的投票 ID。**`链接到：phpbb_vote_desc -> vote_id`** |
| vote_option_id | tinyint(4) | 否 | 0 | 投票可能答案的连续编号。 |
| vote_option_text | varchar(255) | 否 | - | 投票答案文本。 |
| vote_result | int(11) | 否 | 0 | 此投票答案的投票数。 |

# phpbb_vote_voters

关于谁在哪个投票中投票的数据。

| 字段 | 类型 | 可空 | 默认值 | 备注 |
| --- | --- | --- | --- | --- |
| vote_id | mediumint(8) | 否 | 0 | 来自`vote_desc`表的投票 ID。**`链接到: phpbb_vote_desc -> vote_id`** |
| vote_user_id | mediumint(8) | 否 | 0 | 来自用户表的用户 ID。**`链接到: phpbb_users -> user_id`** |
| vote_user_ip | char(8) | 否 | - | 投票时投票者的编码 IP 地址。 |

# phpbb_words

被审查词汇列表。

| 字段 | 类型 | 可空 | 默认值 | 备注 |
| --- | --- | --- | --- | --- |
| word_id | mediumint(8) | 否 | - | 自动递增。 |
| word | char(100) | 否 | - | 不雅词汇。 |
| replacement | char(100) | 否 | - | 替换词汇。 |
