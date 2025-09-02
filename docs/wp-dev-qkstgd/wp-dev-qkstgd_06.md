# 第六章：WordPress APIs 的实际应用

在现代网站上，应用程序编程接口的使用很常见。我们使用 Google Maps、Google Analytics 和 Facebook 等 API 来利用第三方服务的功能。基本上，API 是由开发者创建的，以使他们的功能对其他开发者和平台可用。同样，WordPress APIs 让我们能够以标准方式使用核心功能来改变和扩展功能。我们还可以使用这些功能在 WordPress 之上创建自己的 API，并将它们暴露给第三方服务和平台。

在本章中，我们将探讨可用的 API、它们的功能以及在开发中的应用。我们将更多地关注在自定义开发中起主要作用但尚未在前几章中介绍的三个 API。开发者经常使用 WordPress 短代码来提供可重用功能，客户也熟悉使用这些短代码。因此，我们研究了创建短代码的技术以及识别限制。接下来，我们探讨 Rewrite API 在构建不影响核心功能的功能中的重要性。最后，我们查看 REST API 的使用，通过标准接口使功能对第三方开发者可用。

本章我们将涵盖以下主题：

+   WordPress APIs 的简要概述

+   介绍短代码 API

+   创建自定义短代码

+   理解短代码的使用

+   使用 Rewrite API 管理自定义路由

+   使用 REST API 建立远程连接

在本章结束时，你应该了解开发任务所需的 API 函数。此外，你应该能够使用短代码构建可重用功能，并通过 REST API 将你的功能暴露给第三方应用程序。

# 技术要求

遵循此程序需要安装 WordPress 4.9.8。即使你有 WordPress 的更高版本，描述的示例也应该没有重大问题地运行。

本章的代码文件可以在 GitHub 上找到：

[`github.com/PacktPublishing/WordPress-Development-Quick-Start-Guide/tree/master/Chapter06`](https://github.com/PacktPublishing/WordPress-Development-Quick-Start-Guide/tree/master/Chapter06)

查看以下视频，以查看代码的实际运行情况：

[`bit.ly/2EQa24y`](http://bit.ly/2EQa24y)

# WordPress APIs 的简要概述

WordPress API 是一组协同工作的子 API，允许开发者在核心功能之上构建。单个 API 涵盖一个或多个核心功能，而一些 API 可以用于 WordPress 核心功能之外。REST API、Rewrite API 和 Shortcode API 是我们可以用于 WordPress 核心功能以及自定义功能的一些 API。与构建自己的函数相比，使用 WordPress API 可以减少你的开发工作量。让我们看看在 WordPress 中使用 API 的优势：

+   **使用动作和过滤器**：WordPress API 函数在过程中执行必要的动作和过滤器，允许开发者自定义功能。使用我们自己的自定义函数会省略这些动作和过滤器。因此，将许多插件组合或一起使用将变得困难，因为一些插件可能依赖于 API 函数中的这些钩子。

+   **输入验证和安全**：内置的 API 函数是由世界上最优秀的 WordPress 开发者开发和测试的。因此，数据验证和过程中的安全性非常可靠。使用自定义构建的函数而不是 API 函数会增加开发者的工作量，以实现这些安全功能和验证。

+   **效率**：内置的 API 函数经过优化，以提高与数据库交互以及处理核心功能和文件时的性能。因此，使用这些函数可以提高您解决方案的效率。

+   **向后兼容性**：WordPress 团队高度重视与旧版本的兼容性，因此很少移除对旧函数和功能的支持。因此，使用 API 函数保证了当前 WordPress 版本以及未来版本中功能的兼容性。

上述列表解释了为什么开发者应尽可能使用 API 来节省开发时间，并构建无风险的解决方案。

# 使用内置 API

根据最新版本，WordPress 提供了十八个独立的 API，用于处理核心的不同部分。我们已经在之前的章节中使用了一些 API 函数。在本章中，我们将详细介绍几个 API。让我们来看看可用的 API 以及它们在开发中的作用：

+   **仪表板小部件 API**：它用于添加、编辑或修改管理仪表板上的小部件。仪表板是用户从后端登录后重定向到的位置。默认仪表板小部件包括**概览**、**快速草稿**和**活动**。此 API 通过删除不必要的 widget 并添加特定插件的 widget，有助于构建项目特定的仪表板。

+   **数据库 API**：它用于简化并优化核心数据库表中的操作。我们已经在第二章，*管理数据库结构、存储和检索*中使用过核心数据库 API 功能，通过`global $wpdb`对象和查询函数。

+   **HTTP API**：它用于安全地执行 HTTP 操作，例如发送数据请求或检索数据。API 函数检查 PHP 中各种可用的连接类型中服务器支持的连接类型。我们之前章节中没有使用 HTTP API。

+   **文件头 API**：它用于在主题和插件中处理文件头。该 API 的函数通过文件的头部注释部分识别主题和插件。从开发者的角度来看，该 API 的功能在开发任务中不太常用。

+   **文件系统 API**：它用于在多种托管环境中读取和写入本地文件到文件系统。从开发者的角度来看，该 API 的功能在开发任务中不太常用。

+   **元数据 API**：它用于以标准方式处理 WordPress 元对象类型。WordPress 为帖子、用户和评论提供了元数据表。这些函数允许您以最佳性能添加、编辑、修改和删除帖子元数据。我们在第五章，*通过插件扩展、过滤器、和动作*中，使用`update_user_meta`和`get_user_meta`等帖子元数据函数与元数据 API 进行了交互。从开发者的角度来看，该 API 的功能非常重要，并且在开发任务中常用。

+   **选项 API**：它用于处理 WordPress 中的选项表，以存储网站范围的设置以及主题和插件特定的设置。这些 API 函数允许您以最佳性能添加、编辑、修改和删除选项。我们在第四章，*使用插件开发构建自定义模块*中，使用`get_option`和`update_option`等函数与选项 API 进行了交互。从开发者的角度来看，该 API 的功能非常重要，并且在开发任务中常用。

+   **插件 API**：它在插件开发中使用，因此对于您作为开发者来说非常重要。该 API 包含用于管理 WordPress 钩子的函数。我们已通过使用`add_filter`、`apply_filters`、`add_action`函数与该 API 进行了交互。

+   **快速标签 API**：它用于允许开发者向 WordPress 内容编辑器的**文本**模式添加额外的按钮。我们没有使用此 API，因为它仅关注修改内容编辑器。从开发者的角度来看，在内容创建是网站主要功能的情况下，此 API 很重要。

+   **设置 API**：它用于创建和管理您网站的设置以及主题或插件的设置。您可以使用此 API 向现有设置页面添加设置，或将您自己的自定义设置部分添加到 WordPress 设置菜单中。这是添加和保存设置的最简单方法之一，因为大部分功能都由 API 处理。从开发者的角度来看，此 API 对于构建任何类型的网站都非常重要。

+   **主题修改 API**：它旨在让主题开发者添加和检索主题的设置作为 WordPress 选项。除非您正在构建自定义主题，否则该 API 在开发中不扮演关键角色。

+   **主题定制 API**：它用于定制主题设置、小工具、样式，并可以立即预览更改。您可以通过外观 | 定制部分访问由该 API 生成的功能。开发者可以使用此 API 添加他们自己的设置、部分和控制。

+   **Transients API**：它用于在`wp_options`表中临时存储或缓存信息。与 WordPress 选项不同，transients 有一个过期时间，因此仅用于短期保存信息。API 函数允许您使用各种选项存储、检索和删除 transients。

+   **Widgets API**：它用于在 WordPress 中创建和管理小工具。在 WordPress 中，小工具是一个可重用的组件，主要用于网站的侧边栏。我们将在接下来的章节中更详细地讨论小工具和 API 函数。

+   **XML – RPC API**：它用于将网站功能暴露给第三方服务和应用程序。这是提供 API 功能的老方法，随着 REST API 的出现，它正在逐渐被弃用。

在本节中，我们对所有 WordPress API 进行了简要概述，除了三个。短代码、重写规则和 REST API 在定制开发中起着重要作用，因此我们将在接下来的章节中详细讨论它们。您可以在[`codex.wordpress.org/WordPress_API%27s`](https://codex.wordpress.org/WordPress_API%27s)查看有关先前 API 的更多详细信息。

# 介绍短代码

WordPress 中的短代码是一段代码，它根据各种要求为网站生成动态内容或更改给定内容。这个功能是为了允许用户向帖子或页面添加动态内容而引入的。然而，现代网站将短代码用于其预期目的之外。我们可以看到短代码被用于小工具、模板文件，甚至插件内部以提高可重用性。短代码 API 是一组函数，允许开发者根据他们的偏好轻松创建和使用短代码。现代网站使用高级主题以及众多插件来提供具有吸引力的设计的高级功能。因此，您将看到短代码的使用越来越多。短代码是向网站添加功能的最简单方法之一，因此了解 API 函数非常重要。

# 短代码的结构

在本节中，我们将从不同用户类型的视角来理解 WordPress 短代码。作为一个开发者，理解所有三种用户视角对于在开发任务中构建和使用短代码非常重要。让我们看一下以下图表：

![图片](img/9d20fe00-9482-4e47-808e-5580c98294ec.png)

上述图表基于使用短代码过程中涉及的三种用户视角。

短代码的内容使用是可选的。我们可以找到许多自闭合短代码，它们在不需要使用任何内容的情况下生成输出。

让我们了解这三个用户视角中的每一个。

# 短代码的开发者视角

开发者负责构建短代码，除非你只计划使用 WordPress 核心中可用的短代码。短代码需要使用 API 函数创建，并通过插件或主题注册到 WordPress 中。前面的图示显示了从开发者视角的基本短代码语法。

# 短代码的内容创作者视角

通常，网站管理员为大多数网站创建内容。然而，你也可能有其他用户角色或具有创建内容权限的特定用户。这些用户负责在网站开发过程中使用带有必要属性和内容的短代码。前面的图示说明了短代码的主要语法，这些用户将在帖子或页面中使用。

他们可以使用 WordPress 核心、主题或网站上提供的短代码。内容创作者不必知道短代码的来源。然而，在你想切换主题或插件的情况下，了解短代码是否由主题、插件或 WordPress 核心生成变得非常有用。在这些情况下，缺乏对短代码来源的了解可能会导致冲突，甚至导致网站完全崩溃。除非他们想探索主题或插件内部的代码，否则这些用户不会意识到短代码的源代码。

# 短代码的用户视角

用户是任何访问网站的人，包括访客和成员。他们将只看到由短代码生成的输出。这些用户不知道短代码的存在，因为短代码在浏览器视图或页面源代码中是不可见的。此外，短代码输出没有预定义的 ID 或类，因此这些用户无法从前端跟踪短代码。

现在你应该对 WordPress 短代码的创建和使用以及如何生成动态输出有一个基本的了解。关于短代码的创建和使用将在后续章节中详细讨论。

# 使用内置短代码

WordPress 核心包含一组内置短代码，主要专注于向帖子或页面添加各种内容类型。内容创作者可以直接在帖子、页面或任何支持的位置使用这些短代码，无需任何修改。让我们看看现有的短代码：

+   **音频**：用于嵌入和播放音频文件

+   **标题**：用于添加内容标题。主要用于图像

+   **嵌入**：用于嵌入由 WordPress 支持的网站的内容

+   **相册**：通过传递图像 ID 来显示图像相册

+   **视频**：用于嵌入和播放视频文件

+   **播放列表**：用于显示音频和视频文件的集合

正如你所见，这些短代码不使用 WordPress 网站的主要数据，如文章、评论、用户。因此，我们将不得不与外部插件中的短代码或创建自定义短代码来满足涉及 WordPress 数据库的功能。

# 构建自定义短代码

构建自定义短代码的过程并不像许多人想象的那样复杂。短代码的最小实现只需要几行代码，如最后部分的图中所示。一个短代码由四个主要部分组成：

+   **开标签和闭标签**：开标签和闭标签类似于 HTML，WordPress 短代码使用方括号的开标签和闭标签。这些标签由内容创建者用于向文章或页面添加功能。另一方面，开发者需要使用这个`标签`来注册短代码作为唯一元素。一些短代码没有闭标签，并使用自闭合的开标签。

+   **属性**：属性是一组数据设置，用于在短代码函数内部处理功能并返回输出。属性列表从短代码名称在开标签之后开始。每个属性都有一个键和值，如短代码图像的结构所示。属性应该由一个或多个空格分隔。在短代码内部，开发者可以使用属性变量的数组键访问传递的属性。

+   **内容**：内容是我们添加到短代码开标签和闭标签之间的内容。一旦内容被使用，短代码将作为 WordPress 过滤器工作，在那里我们检索、处理和返回修改后的内容。我们也可以通过使用自闭合的开标签来定义没有内容短代码。

+   **输出**：输出是基于传递的属性和内容，由短代码返回的值。输出将是短代码内部修改的内容，或者内容将被用来捕获和显示网站数据。许多开发者倾向于直接使用 PHP `echo`语句将短代码输出打印到浏览器。然而，短代码应该始终返回输出而不是直接打印到浏览器。

现在我们可以继续创建自定义短代码的过程，使用这些部分。

# 创建自定义短代码

在本节中，我们将考虑两种场景来构建自定义短代码，并解释短代码的不同用途。让我们列出为每个构建短代码的两个要求：

+   根据用户角色限制内容

+   显示带有附件的文章列表

让我们开始吧。

# 使用短代码限制内容

这种情况解释了使用之前章节中讨论的四个短代码组件创建短代码的过程。基本上，我们需要一个短代码，它接受内容，并且只对授权用户显示输出，同时为未授权用户提供自定义输出。让我们考虑这个场景所需的短代码：

```php
[wpquick_restrict_content  role=subscriber]  Content to be protected [/wpquick_restrict_content]
```

之前的短代码有一个属性用于指定不允许查看内容的用户角色和需要保护的内容，现在我们必须将之前的短代码与`短代码结构`图中的代码相匹配。考虑以下代码来实现这个短代码：

```php
add_shortcode( 'wpquick_restrict_content', 'wpquick_restrict_content_display' );
function wpquick_restrict_content_display( $atts, $content ){
  $sh_attr = shortcode_atts( array( 'role' => '' ), $atts );

  if( $sh_attr['role'] != '' && ! current_user_can( $sh_attr['role'] ) ){
    $content = __('You don\'t have permission to view this content','wqsa');
  } 
  return $content;
}
```

在这种情况下，一个回调函数接受短代码的属性和内容。WordPress 提供了一个名为`shortcode_atts`的函数，用于合并传递的属性和默认属性，并创建一个用于处理短代码所需的属性数组。在`shortcode_atts`函数内部定义允许的属性和默认值作为一个数组是一个好的做法。在这种情况下，我们为`role`参数传递了一个值，因此它将覆盖默认参数。

如果我们没有指定属性的值，`shortcode_atts`将查找默认值。接下来，我们使用`$sh_attr`数组访问属性并应用条件。最后，根据条件返回原始内容或消息。这是构建短代码所需的最基本的代码。

# 显示带有附件的帖子

这种情况从开发者和内容创作者的角度解释了短代码的不同用法。在这里，我们需要显示至少有一个附件的帖子列表。与之前的场景不同，我们不是检索和修改内容。相反，我们根据短代码属性生成动态内容。让我们看看这个场景所需的短代码：

```php
[wpquick_attachment_posts/]
```

如您所见，没有关闭标签，关闭部分是在开标签内完成的。这类短代码被称为*自闭合*短代码。我们不需要在这些短代码中使用任何内容。在这种情况下，我们正在显示所有带有附件的帖子，因此不需要短代码属性。如果我们需要显示特定类别的帖子附件，短代码将如下所示：

```php
[wpquick_attachment_posts category='1' /]
```

现在我们可以通过将其与我们在短代码图中使用的语法相匹配来实现之前的短代码：

```php
add_shortcode( 'wpquick_attachment_posts', 'wpquick_attachment_posts_display' );
function wpquick_attachment_posts_display( $atts, $content ){
  global $wpdb;
  $post_attachments_table = $wpdb->prefix.'wpqpa_post_attachments'; 
  $sql = "SELECT P.post_title,P.guid from $post_attachments_table as PA inner join $wpdb->posts as P on P.ID=PA.post_id group by PA.post_id ";

  $result = $wpdb->get_results($sql);
  $html = '';
  if($result){
    foreach ( $result as $key => $value ) {
      $html .= '<a href="'. $value->guid .'">'. $value->post_title .'</a><br/>'; 
    }
  } 
  return $html;
}
```

在这种情况下，我们没有属性，因此我们通过将其与帖子表连接来查询自定义表以生成结果。`$content`变量将为空，因为我们没有通过开闭标签传递任何数据。最后，我们返回一个 HTML 字符串以显示帖子列表。

# 理解短代码的使用

我们在上一节中讨论了构建短代码的技术。现在我们需要了解在不同网站部分使用这些短代码的技术。让我们看看我们可以使用短代码的不同位置：

+   **帖子与页面**：这是短代码最常见的使用方式，内容创建者直接在帖子或页面编辑器中添加短代码。然后，WordPress 执行短代码，并在前端帖子或页面上显示结果。

+   **小部件**：我们可以在 WordPress 小部件中使用短代码来提供类似于帖子或页面的动态内容。然而，我们必须根据所使用的小部件使用额外的代码。WordPress 的**文本小部件**允许你直接在内容编辑器中添加短代码，并在前端显示输出。有时，你可能想使用短代码与 HTML 元素一起，因此需要使用**HTML 小部件**而不是**文本小部件**。HTML 小部件默认不支持短代码。因此，我们必须在主题的`functions.php`文件或任何插件中以下代码来执行短代码：

```php
     add_filter( 'widget_text', 'do_shortcode' );
```

WordPress 有一个名为`widget_text`的过滤器，它在所有小部件的内容上执行。我们可以使用这个过滤器来按需更改内容。到目前为止，我们使用了`add_filter`语句和一个回调函数。这里，我们还有一个名为`do_shortcode`的回调函数。然而，你可能想知道为什么实现缺失。`do_shortcode`函数是内置在 WordPress 中的，因此我们可以直接调用它，而无需为该函数添加实现。此函数执行内容中的任何短代码。

+   **模板文件**：有时，我们可能需要在标题、页脚或任何模板文件中调用短代码。当你正在自定义现有主题以集成其他插件的功能时，这非常有用。在这种情况下，将短代码添加到模板文件中不会起作用。我们必须使用 PHP 脚本并使用`do_shortcode`函数来执行短代码。考虑以下代码，用于在模板文件中使用短代码：

```php
      <?php echo do_shortcode('wpquick_attachment_posts'); ?> 
```

我们还需要使用`echo`语句将输出打印到浏览器，因为在之前的场景中它不是自动打印的。

+   **插件与主题**：有时，我们希望将短代码功能与其他插件或主题集成。在这种情况下，我们可以直接调用短代码，而不是复制短代码的代码。假设我们想在每篇帖子内容之后显示带有附件的帖子。在这种情况下，我们可以在我们的插件内部调用短代码，生成带有附件的帖子列表，而不是编写单独的函数或复制代码。考虑以下用于在主题或插件代码中使用短代码的语句：

这些是使用短代码的一些常见方法和位置。

# 使用短代码的技巧

短代码是一种简单灵活的方法，可以将可重用的功能添加到网站中。然而，使用短代码既有优点也有缺点。作为一名开发者，你需要了解何时使用短代码以及如何构建高质量的短代码。以下提示将帮助你改进短代码，同时避免不必要的麻烦：

+   **过滤短代码输出**：通常，我们在短代码内创建一些内容，并将其直接作为输出返回。有些插件会在短代码的输出上使用过滤器。这是一种增加灵活性的好方法，开发者可以控制其他插件的短代码输出。

+   **使用嵌套短代码**：我们可以在其他短代码内添加短代码以获取输出。然而，在使用嵌套短代码时，您需要在主短代码中使用`do_shortcode`来执行其内容内的短代码。

+   **验证短代码属性**：属性是由内容创建者添加的，因此需要将其视为用户输入。由于我们不应信任任何类型的用户输入，因此对属性值进行验证是必须的。

+   **使用唯一的短代码名称**：许多插件使用通用的短代码名称，如 `[product]`、`[event]` 等。这可能导致多个插件之间的冲突，因此您应该始终使用插件特定的前缀来使其独特。

+   **过度使用短代码**：短代码是添加大量内容的一种简单方法。然而，在文章或页面中使用过多的短代码可能导致后期维护噩梦。由于文章或页面依赖于短代码，更改内容将变得困难。

我们已经介绍了创建和使用短代码的过程。现在，让我们看看使用短代码的插件，并识别各种用途和开发技术。

# 使用 Rewrite API 管理自定义路由

默认的 WordPress URL 结构使用查询参数来加载必要的文章和页面。因此，文章的 URL 将看起来像[`www.example.com/?p=130`](http://www.example.com/?p=130)。这并不理想，因为很难记住文章 ID，搜索引擎也不会给予此类 URL 更高的优先级。因此，我们需要一个更好的 URL 结构，不使用查询参数，并提供 SEO 友好的 URL。因此，我们使用 WordPress 的永久链接部分来更改 URL 结构。然后，WordPress 将内部将这些 SEO 友好的 URL 转换为默认的 URL 结构。我们可以从设置 | 永久链接访问永久链接设置部分。让我们选择*文章名称*作为我们网站的 URL 结构。然后，前面的 URL 将转换为`http:// www.example.com/sample-post/`。

如您所见，这是一个更符合用户和 SEO 友好的 URL 结构。一旦设置好永久链接，WordPress 前端的所有核心功能都将使用这种结构。然而，在自定义开发中，我们可能需要自定义 URL 结构来处理功能。在这种情况下，我们可以使用重写 API 来定义我们自己的路由并管理功能，而不会干扰默认的 WordPress 功能。

**什么是重写 API？**

WordPress 重写 API 是一组函数，允许您使用标签、规则和端点来管理自定义路由。让我们确定用于实现重写 API 的函数：

+   `add_rewrite_rule`: 这个函数用于向 WordPress 注册新的重写规则以生成自定义模板。

+   `add_rewrite_tag`: 这个函数用于注册新的查询变量。我们需要与`add_rewrite_rule`一起使用来为自定义模板创建重写规则。

+   `add_rewrite_endpoint`: 这个函数用于为 WordPress 核心组件创建额外的重写规则。

+   `flush_rewrite_rules`: 这个函数用于删除重写规则然后重新创建重写规则。

在本节中，我们将使用这些函数（除了`add_rewrite_endpoint`）来创建和管理自定义规则。您可以在[`codex.wordpress.org/Rewrite_API/add_rewrite_endpoint`](https://codex.wordpress.org/Rewrite_API/add_rewrite_endpoint)查看关于重写端点的更多详细信息。

# 理解自定义路由的需求

在开发超出默认 WordPress 功能的插件时，自定义路由的使用变得至关重要。我们将考虑两个场景来理解自定义重写的需求：

+   假设我们有一个用于显示用户个人资料详情的短代码，并且我们在名为**Profile**的 WordPress 页面中使用它。因此，该页面的 URL 将是[`www.example.com/profile`](http://www.example.com/profile)。现在，我们需要使用相同的页面来显示每个用户的详细信息。目前，这是不可能的，因为我们没有在访问此页面时识别用户的方法。因此，我们需要以某种方式修改 URL，以便我们可以识别每个用户。让我们假设我们有一个类似[`www.example.com/profile/john123`](http://www.example.com/profile/john123)的 URL，用户名位于页面之后。然后，我们可以使用用户名组件来识别用户并显示个人资料详情。在这种情况下，我们必须使用重写 API 函数来匹配 URL 与必要的查询参数并执行我们的功能。

+   正如我们已经讨论过的，将短代码添加到如此重要的功能中可能会在管理员不小心删除或移除短代码时成为一个问题。因此，依靠帖子或页面来处理此类功能可能被视为一种风险。在这种情况下，我们可以通过使用不使用帖子或页面的自定义 URL 来避免风险。假设我们想在[`www.example.com/user/login`](http://www.example.com/user/login)显示登录表单，在[`www.example.com/user/register`](http://www.example.com/user/register)显示注册。默认情况下，当你没有名为`user`的帖子或页面时，这些 URL 将重定向到*404 未找到页面*。因此，我们需要使用重写 API 来注册自定义路由并管理这些 URL，以提供不干扰核心 WordPress 功能的自定义功能。

在本节中，我们将实现第二个场景，因为处理没有帖子或页面的自定义 URL 是自定义功能的理想解决方案。

# 创建路由规则

我们可以使用`add_rewrite_rule`函数在 WordPress 中注册新的自定义路由。这个函数可以在 WordPress 的许多动作中实现。然而，我们通常使用`init`动作来处理重写规则。让我们添加一个自定义重写规则来处理用户登录和注册的场景：

```php
add_action('init','wqraf_manage_user_routes');
function wqraf_manage_user_routes() {
  add_rewrite_rule( '^user/([^/]+)/?', 'index.php?wpquick_action=$matches[1]', 'top' );
}
```

你可以在源代码目录中的`wpquick-rewrite-api`插件中找到本节的代码。`add_rewrite_rule`函数接受两个必需参数和一个可选参数。让我们看看这些参数及其作用：

+   `regex`：这是第一个参数，它使用正则表达式将 URL 与规则匹配。

+   `redirect`：这是第二个参数，它指定当当前 URL 与正则表达式匹配时请求的 URL。正如你所见，我们通过传递带有唯一标识我们功能的自定义参数将请求传递给`index.php`。

+   `after`：这个可选参数决定何时使用这个重写规则。如果使用`top`值，重写规则将优先于其他默认 WordPress 重写规则。如果使用`bottom`值，将优先考虑现有规则。

现在，你可能认为我们可以使用`$_GET`查询参数访问`wpquick_action`并实现登录和注册的功能。然而，你不会在`$_GET`参数中找到`wpquick_action`。WordPress 不允许你在查询字符串中使用任何类型的变量。它将检查现有列表中的查询变量，而所有其他变量将被忽略。因此，在我们实际上可以在代码中使用它们之前，我们必须指定自定义查询变量。

# 添加查询变量

WordPress 注册了超过 40 个参数以供查询使用。在添加自定义查询变量之前，让我们检查现有的查询变量。将以下代码添加到`wpquick-rewrite-api`插件的主体文件中，并打开你网站的首页：

```php
add_filter( 'query_vars', 'wqraf_manage_user_query_vars' );
function wqraf_manage_user_query_vars( $query_vars ) {
  echo "<pre>";print_r($query_vars);exit;
}
```

你将看到所有现有的查询参数作为一个数组。现在我们需要创建我们自己的查询参数来处理用户登录和注册。因此，我们在`wpquick-rewrite-api`插件中添加以下代码：

```php
add_filter( 'query_vars', 'wqraf_manage_user_query_vars' );
function wqraf_manage_user_query_vars( $query_vars ) {
  $query_vars[] = 'wpquick_action';
  return $query_vars;
}
```

`query_vars`是一个内置的过滤器，允许我们在请求处理之前添加或删除查询变量。在这种情况下，我们向现有的查询参数数组中添加了一个名为`wpquick_action`的新查询参数。

WordPress 核心和其他插件注册了相当数量的查询变量。因此，最好使用带有前缀的唯一名称以避免与其他插件的查询参数冲突。

现在我们有能力从 URL 中访问`wpquick_action`变量。在这种情况下，我使用了`query_vars`过滤器来添加新的查询变量，并用于识别内置的查询变量。重写 API 提供了一个名为`add_rewrite_tag`的函数来实现相同的功能，这是推荐的方式。因此，我们可以从我们的插件中移除`query_vars`过滤器，并将`wqraf_manage_user_routes`函数修改如下以获得相同的功能：

```php
function wqraf_manage_user_routes() {
  add_rewrite_rule( '^user/([^/]+)/?', 'index.php?wpquick_actions=$matches[1]', 'top' );
  add_rewrite_tag('%wpquick_actions%', '([^&]+)');
}
```

第一个参数接受一个由`%`包围的**动作名称**，第二个参数定义了一个`regex`来验证值。与使用`query_vars`过滤器相比，添加重写标签相对容易。

我们通过添加新的规则修改了 WordPress 的重写规则。因此，在新的规则生效之前，我们必须刷新重写规则。有两种刷新重写规则的方法。首先，我们可以访问设置 | 永久链接部分来自动刷新重写规则。然而，在开发中这并不理想，因为用户需要手动进入**设置**部分。因此，我们需要在我们的插件中自动刷新重写规则的方法，而不需要用户的输入。

# 刷新重写规则

我们可以很容易地通过在`init`动作中调用 WordPress 的`flush_rewrite_rules`函数来刷新重写规则。然而，它会在每次请求时刷新重写规则，产生不必要的性能开销。因此，我们需要使用插件的激活处理程序来刷新规则。让我们看看以下激活处理程序中的实现：

```php
register_activation_hook( __FILE__, 'wqraf_activate' );
function wqraf_activate(){
  flush_rewrite_rules(); 
}
```

现在，转到管理面板，停用插件，然后再次激活插件。然后，转到 URL [`www.example.com/user/login`](http://www.example.com/user/login) 并检查它是否工作。不幸的是，你仍然会得到 404 错误。你可能想知道出了什么问题。让我们回到并思考这个过程，以了解问题所在。我们在插件激活时刷新了规则。因此，新规则应该成功持久化。然而，我们在`init`动作中定义了规则，这个动作只有在插件激活后才执行。因此，新规则在刷新时不可用。所以，我们必须更新函数如下以使其工作：

```php
register_activation_hook( __FILE__, 'wqraf_activate' );
function wqraf_activate(){
  wqraf_manage_user_routes();
  flush_rewrite_rules(); 
}
```

现在请先禁用插件，然后再重新激活它。接着，访问网址 [`www.example.com/user/login`](http://www.example.com/user/login)。这次，你不会遇到 404 错误，因为重写规则已经被正确添加并刷新。现在我们已经准备好了用户功能的路由规则。

# 显示自定义路由的内容

我们必须构建一个自定义路由来管理站点的登录和注册。现在我们需要从自定义路由访问自定义查询变量，并生成登录和注册的屏幕。以下代码用于实现登录和注册表单的实现：

```php
add_action( 'template_redirect', 'wqraf_front_controller' );
function wqraf_front_controller() {
  global $wp_query;
  $wpquick_action = isset ( $wp_query->query_vars['wpquick_action'] ) ? $wp_query->query_vars['wpquick_action'] : '';
  switch ( $wpquick_action ) {
    case 'register':
      echo "<h1>REgistration Form</h1>";exit;
      break;
    case 'login':
      echo "<h1>Login Form</h1>";exit;
      break;
  }
}
```

WordPress 在决定加载哪个模板之前执行`template_redirect`动作。因此，我们可以使用这个动作来拦截请求并加载我们自己的模板。首先，我们使用全局对象`$wp_query`访问`wpquick_action`值。在这个场景中，这个变量将包含注册或登录。然后，我们使用`switch`语句来过滤动作并加载必要的模板。在这里，我们只打印标题以供说明。你必须生成登录或注册表单而不是标题。

由于我们正在拦截请求，我们的内容将会被加载而不是 WordPress 模板。在这里，它只会打印标题，而不会包含头部、底部或其他 WordPress 模板的部分。因此，你必须通过包含头部、底部、内容区域和侧边栏来生成一个完整的模板。

我们已经探讨了使用重写 API 函数来创建自定义路由和为插件加载自定义模板的过程。当开发具有非 WordPress 特定屏幕的高级插件时，这将非常有用。

# 使用 REST API 建立远程连接

WordPress REST API 正在超越旧的 XML-RPC API，并成为网站开发的行业标准。该 API 允许开发者将 WordPress 站点与其他站点以及第三方应用程序连接起来。现代网站正朝着基于 JS 框架的前端发展，以优化性能并简化用户体验。在这种情况下，开发者可以使用 WordPress 作为核心功能的后端，并通过 REST API 公开数据来构建不使用 WordPress 的前端。REST API 很可能是 WordPress 开发的未来。

REST API 最初是在一个外部插件中引入用于测试的。最终，它在 WordPress 4.7 版本中被包含到 WordPress 核心中，现在它完全支持 WordPress 中所有主要数据模型的 REST API 端点。

让我们确定一些在 REST API 操作中常用的术语：

+   **路由**：这是一个可以映射到 HTTP 方法的明确定义的 URL

+   **端点**：将特定路由与 HTTP 方法匹配的过程

+   **请求**：带有必要数据的 API 端点调用

+   **响应**：API 对特定请求提供的数据

+   **模式（Schema**）: 这用于结构化 API 数据并提供关于 API 请求中可用属性和输入参数的信息

# 介绍核心 REST API 端点

WordPress 为处理大多数主要核心功能提供了内置的 API 端点。文档定义了端点、属性和示例请求，简化了开发者的学习曲线。在本节中，我们将探讨核心 REST API 端点以及使用测试请求来理解这个过程。您可以在 WordPress REST API 文档中查看所有可用端点和附加信息，网址为[`developer.wordpress.org/rest-api/reference/`](https://developer.wordpress.org/rest-api/reference/)。WordPress REST API 默认启用。您可以通过在浏览器中访问[`www.example.com/wp-json`](http://www.example.com/wp-json)来检查您的网站上是否启用了 API。如果 API 已启用，您将获得一个包含可用设置和端点的大型 JSON 字符串。

# 理解和测试核心 API 端点

我们必须识别和理解 API 端点、参数和返回数据格式，以使用 API 功能构建功能。测试 API 请求是理解过程的最佳方式。我们可以使用现有的工具来简化 API 请求测试过程。有许多这样的工具，我们将使用 Google Chrome 浏览器的 Postman 扩展。

**Postman** 是一个通过提供测试 API 请求、构建 API 文档和监控 API 使用等特性来简化管理 API 过程的工具。此工具在免费和 PRO 版本中提供了许多额外的 API 相关功能。免费扩展对于处理基本 API 测试项目来说已经足够。您可以使用 PRO 版本的高级功能，例如团队协作、创建模拟服务器以及与许多第三方 API 的集成。

您可以从[`www.getpostman.com/`](https://www.getpostman.com/)在您的 Chrome 浏览器中安装此扩展。一旦安装，您将获得一个独立的程序，如下面的截图所示：

![图片](img/8e87c0d8-6c90-4076-b11a-6423bed1fbb7.png)

现在我们准备使用 WordPress REST API 测试 API 请求。主要来说，我们在开发任务中主要使用 HTTP `GET` 和 `POST` 请求。然而，这个工具提供了使用各种 HTTP 请求类型的能力，例如 `PUT` 和 `DELETE`。REST API 包含端点，允许我们直接使用 GET 请求以及需要认证的`POST`请求。因此，我们将检查这两种类型的请求。

让我们从测试一个 GET 请求开始。基本上，这些端点暴露了 WordPress 中现有的数据。首先，你必须在**输入请求 URL**字段中输入端点的 URL，并将 HTTP 请求选择为`GET`。端点需要添加到 URL 中的[`www.example.com/wp-json/`](http://www.example.com/wp-json/)部分之后。因此，通过 REST API 访问帖子列表的请求看起来与以下截图所示的参数相似：

![截图](img/f12f53cc-9f6a-4a6c-92c1-1b3b54d3f9d7.png)

底部的屏幕显示了我们选择的请求结果的格式。我们必须选择 JSON 作为结果类型，因为成功的 API 请求返回 JSON 数据。由于我们正在使用 GET 请求，类型也被设置为无认证。

现在我们可以继续测试用于检索数据以显示以及通过 API 请求修改数据库的 POST 请求。与使用 GET 请求相比，测试 POST 请求需要更多的工作。这些请求始终需要某种认证，并且在没有有效认证的情况下返回错误。因此，首先，你必须在**输入请求 URL**中添加 URL，并将方法选择为`POST`。接下来，转到**授权**标签页，并选择基本认证作为类型。然后添加具有创建和编辑帖子能力的网站用户的用户名和密码。

通常，POST 请求需要额外的参数和值。在这里，我们正在尝试创建一个帖子，因此你可以在“主体”标签页的表单数据部分添加帖子详情作为键值对。最后，点击**发送**按钮发送 API 请求。然而，你将得到以下错误消息而不是创建帖子：

```php
{ "code": "rest_cannot_create", "message": "Sorry, you are not allowed to create posts as this user.", "data": { "status": 401 } }
```

你可能不明白为什么即使我们提供了基本的认证信息，它仍然返回权限错误。WordPress 认为基本认证是一种不安全的处理 REST API 请求的方式，因此，**基本认证**默认是禁用的。所以，我们必须使用 WordPress API 团队开发的名为**JSON 基本认证**的插件。此插件包含在本章的源代码文件夹中。一旦此插件被激活，你就可以再次发送请求，并看到响应和表单参数，如下面的截图所示：

![截图](img/7954b0a4-50e6-41e2-a2d3-c0d9d688f641.png)

响应将包含关于我们请求中创建的新帖子的详细信息。你可以通过更改请求类型并提供必要的参数值，使用`UPDATE`和`DELETE`操作类似地使用。

由于我们必须在每次请求中以纯文本格式传输用户名和密码详情，因此基本认证不是推荐用于认证 REST API 的方式。WordPress 团队建议我们仅将基本认证用于测试目的或在高度安全的私有网络中。

# 管理自定义路由和端点

REST API 的真正力量来自于自定义路由和端点的使用。在调整现有路由和端点以满足我们的要求方面存在某些限制。在某些情况下，我们可能必须创建自己的类并扩展 WordPress 核心类来实现自定义功能。因此，自定义路由和端点允许我们以最少的努力实现自定义 REST API 功能，而不会干扰 WordPress 核心 REST API 功能。

在本节中，我们将实现以下功能，以学习自定义 REST API 路由和端点的基础：

1.  禁用默认帖子端点并使用自定义路由仅在选择类别中选择帖子数据公开帖子

1.  通过列出帖子中的可用附件来提供自定义表数据的 REST API 访问

让我们开始实现这两个功能。

# 为帖子创建自定义路由

默认帖子相关端点可通过`/wp/v2/posts`访问。我们可以对该端点执行不带任何参数的`GET`请求，以检索包含所有主要帖子细节的帖子列表。假设我们想要使用自定义路由为该端点的每个功能提供更多控制，因此我们必须在定义自己的路由之前阻止现有端点。

我们将使用一个名为**WQ REST API**的新插件来实现本章 REST API 部分的所需功能。您可以使用前几章中讨论的插件创建技术，通过创建一个主文件来创建该插件。让我们通过在插件的主文件中使用以下代码来阻止默认的帖子端点：

```php
add_filter( 'rest_endpoints', 'wpquick_rest_endpoints');
function wpquick_rest_endpoints( $endpoints ){
    if ( isset( $endpoints['/wp/v2/posts'] ) ) {
        unset( $endpoints['/wp/v2/posts'] );
    }
    return $endpoints;
}        
```

WordPress 提供了一个名为`rest_endpoints`的过滤器，用于过滤现有端点并通过删除或创建新的端点来自定义它们。在这种情况下，我们通过从端点数组中取消设置来禁用了默认的帖子相关端点。这将移除列出、创建、更新和删除帖子的能力。因此，我们必须为每个这些功能定义自定义路由，或者使用相同的自定义路由来支持所有与帖子相关的操作。让我们首先添加一个自定义路由来为第三方源提供帖子数据：

```php
function wqra_routes() {
  register_rest_route( 'wqra/v1', '/read_posts', 
        array( 'methods'  => WP_REST_Server::READABLE, 
                'callback' => 'wqra_read_posts_handler',                                       'permission_callback' => function () {
                         return true;
               }
    ) );
}
add_action( 'rest_api_init', 'wqra_routes' );
```

REST API 包括一个名为`rest_api_init`的操作，它在执行 API 请求之前被触发。因此，将新路由引入 API 的推荐操作是它。在回调函数内部，我们使用`register_rest_route`函数为我们应用程序创建新路由。让我们确定`register_rest_route`函数中使用的参数的基本参数：

+   `namespace`：这将是在主 REST API URL 之后的第一个 URL 段。在这里，我们使用了`wqra/v1`，因此它将替换我们用于访问核心 WordPress 路由的`wp/v2`。

+   `route`：这是路由的基本 URL。在这里，我们使用了`read_posts`作为路由，它将替换帖子的默认路由。

+   `methods`：在这里，我们使用的方法是`WP_REST_Server::READABLE`。这确保了当`WP_REST_Server`更改我们的可读端点时，我们的自定义路由可以正常工作。`WP_REST_Server`类的`readable`常量是`GET`方法的别名。

+   `callback`：这是当路由与给定的端点匹配时执行的函数。我们定义了一个名为`wqra_read_posts_handler`的回调函数。所有应在 REST API 响应中发送的帖子数据都在此函数中准备。

+   `permission_callback`：这是在执行回调函数之前检查请求权限的函数。我们返回 TRUE 以提供权限。在实际场景中，我们需要使用`current_user_can`函数检查用户权限，并根据用户权限级别返回布尔值。

现在，我们可以查看我们生成特定于网站的帖子列表的回调函数的实现。考虑以下`wqra_read_posts_handler`函数的实现：

```php
function wqra_read_posts_handler() {
  $posts_query = new WP_Query(array('post_type' => 'post','post_status' =>'publish', 'order' => 'desc', 'orderby' => 'date', 'category__not_in' => array( 20 ),'posts_per_page'=>-1 ));
  $data = array();
  if($posts_query->have_posts()){
    while($posts_query->have_posts()) : $posts_query->the_post();
      array_push($data, array("ID" => get_the_ID(), "title" => get_the_title() ));
    endwhile;
  }

  return rest_ensure_response(($data));
}
```

首先，我们使用`WP_Query`对象传递自定义设置并生成基于我们要求的帖子列表。在这种情况下，我们只允许 API 用户查看帖子，除了所选分类的帖子。然后，我们只添加帖子的 ID 和标题，因为我们只想为 API 用户提供这两条数据。默认的帖子列表提供了对帖子所有主要数据的访问权限，因此它不适合我们的要求。最后，我们将结果数据数组传递给`rest_ensure_response`函数。此函数将我们的数据转换为 JSON，并返回与`WP_REST_Response`兼容的响应。

# 为自定义表数据创建自定义路由和端点

在上一节中，我们探讨了禁用现有的 API 端点并创建我们自己的端点来处理核心 WordPress 数据的过程。默认的 WordPress REST API 没有用于处理自定义表数据的任何路由或端点。因此，我们必须始终使用我们自己的自定义路由和端点来操作自定义表数据。

在前面的章节中，我们创建了一个插件来添加附件到帖子。现在，我们将创建一个自定义 API 端点，允许我们检索给定帖子的附件。让我们从向我们的 REST API 插件添加另一个自定义路由开始，如下面的代码所示：

```php
function wqra_routes() {
  // Custom route for read posts
  register_rest_route( 'wqra/v1', '/list_post_attachments/(?P<id>\d+)', array(
     'methods' => 'POST','callback' => 'wqra_list_post_attachments_handler',
     'args' => array( 'id' => array( 'validate_callback' => function($param, $request, $key) {
                 return is_numeric($param);
     }
     ) ) ) );
}
```

我们使用与上次相同的命名空间，并使用`list_post_attachments`作为新的路由。此外，我们在括号中添加了一个额外的参数，其中我们可以传递帖子的 ID。我们还已将方法设置为`POST`而不是使用`GET`。由于我们正在检索记录并在 URL 中传递 ID，我们可以使用`GET`作为方法。然而，我们已启用`POST`方法，以便我们可以传递额外的参数并过滤结果。我们已使用一个新的回调函数`wqra_list_post_attachments_handler`来处理新的自定义路由。

在这种情况下，与之前的自定义路由相比，还有一个额外的参数称为`args`。这个`args`数组用于指定路由的附加参数。在这种情况下，我们使用它来验证传递给端点的参数。帖子 ID 是一个数值，因此我们需要在执行 REST API 请求之前对其进行验证。因此，我们执行 ID 参数的`validate_callback`函数。如果您为帖子 ID 传递字符串值，您将得到以下无效参数错误：

```php
{"code":"rest_invalid_param","message":"Invalid parameter(s):id","data":{"status":400,"params":{"id":"Invalid parameter."}}}
```

我们可以使用这项技术来验证具有不同条件的多个参数，并确保请求只包含请求的格式中的数据。现在我们可以进入实现`wqra_list_post_attachments_handler`函数，该函数用于处理对新路由的请求：

```php
function wqra_list_post_attachments_handler($data){
  global $wpdb;
  $data = $data->get_params();      
  $post_id = isset($data['id']) ? $data['id'] : 0;
  $post_attachments_table = $wpdb->prefix.'wpqpa_post_attachments';
  $sql  = $wpdb->prepare( "SELECT * from $post_attachments_table where post_id = %d ", $post_id);
  $result = $wpdb->get_results($sql);
  $post_attachments = array();
 if($result){
   foreach ($result as $key => $value) {
     $post_attachments[] = array('ID'=> $value->id, 'file_name' => $value->file_name, 'file_path' => $value->file_path);
   }
  }

  return rest_ensure_response(($post_attachments));
}
```

将 REST API 请求对象作为参数传递给此函数。因此，我们使用`$data`对象的`get_params`函数来检索请求中可用的参数。接下来，我们使用`$data`对象的`id`属性捕获帖子 ID。接下来，我们在`wpqpa_post_attachments`表上执行一个自定义查询。结果将包含给定帖子的文件附件。然后，我们遍历结果并将附件详情分配到一个数组中。最后，我们使用`rest_ensure_response`函数将附件数据以 JSON 格式传递到响应中。

在我们有能力使用现有路由和端点的情况下，使用自定义路由和端点也有一些缺点。对于自定义端点，我们需要实现自己的分页和排序，而 WordPress 默认为现有端点提供这些功能。我们可以在 WordPress 核心端点中默认使用用户权限检查，而我们需要传递额外的 nonce 参数来检查用户权限。

现在我们可以调用这个新路由，并检索给定帖子的文件附件列表。

# 构建 REST API 客户端

API 客户端是使用 REST API 提供的数据的用户或服务。我们使用客户端来访问其他站点的 REST API。客户端站点可以是 WordPress 或非 WordPress 站点。客户端也可以在不使用 PHP 的情况下用不同的编程语言实现。让我们确定 API 客户端的用法。在本章中，我们将探讨构建 REST API 客户端以从外部站点访问我们的数据的过程，因为这是 API 最重要的用途之一。

# 来自外部站点的 REST API 客户端

即使我们将 REST API 用于构建同一站点的功能，它也在这里达到了最大潜力。创建 REST API 的主要目的是为第三方 WordPress 和非 WordPress 基于的应用程序提供访问权限。因此，API 客户端可以用任何编程语言实现。在本节中，我们将构建一个基于 PHP 的 API 客户端，以支持 CURL 访问其他应用程序的 REST API。

让我们添加使用 CURL 的 API 客户端实现。你应该将此代码放置在一个 PHP 文件中，并从应用程序外部访问它：

```php
function wpquick_rest_api_client( $url, $post_data = '' ){
  $api_route = $url;
  $ch = curl_init( $api_route );
  $headers = array(
    'Authorization:Basic YWRtaW46dTh1OHU4dTg='// <---
    );
  curl_setopt( $ch, CURLOPT_HTTPHEADER, $headers );
  curl_setopt( $ch, CURLOPT_SSL_VERIFYHOST, false );
  curl_setopt( $ch, CURLOPT_RETURNTRANSFER, 1 );
  curl_setopt( $ch, CURLOPT_POST, true );
  curl_setopt( $ch, CURLOPT_SSL_VERIFYPEER, false );
  if( $post_data != '' ){
    curl_setopt( $ch, CURLOPT_POSTFIELDS, $post_data );
  }
  curl_setopt( $ch, CURLOPT_VERBOSE, 1 );
  $return = curl_exec( $ch );
  echo "<pre>";
  print_r( $return );
  exit;
}

$post_data = array();
$post_data = json_encode( $post_data );
$api_route ="http://www.example.com/wp-json/wqra/v1/list_post_attachments/5";
wpquick_rest_api_client( $api_route, $post_data );
```

我们创建了一个名为`wpquick_rest_api_client`的函数，用于接受 API 端点的两个参数：URL 和文章数据。然后，我们通过在指定路由上执行`curl_init`函数来初始化一个`curl`请求。接下来，我们添加一个包含`username:password`组合的 base-64 编码字符串的授权头。我们需要某种认证方法来执行 POST 和 DELETE 请求。我们应该只为测试目的使用授权头。我们将在下一节讨论各种认证技术。然后，我们将指定必要的`curl`参数，并在请求中添加文章数据（如果提供）。最后，我们执行并打印从服务器返回的 API 响应。

现在我们有一个基本的 API 客户端函数，我们可以使用必要的参数调用该函数来执行 REST API 请求，如前述代码所示。在这种情况下，我们正在以 JSON 字符串的形式检索 ID 为 5 的论坛的文件附件。如果我们请求 ID 作为字符串，我们将得到一个无效参数错误，因为它是经过验证和限制的。因此，我们已经确定了从外部来源构建 API 客户端以访问网站的过程。

REST API 正在成为现代 Web 应用程序的趋势，因此 WordPress 似乎提供了越来越多的功能来构建功能齐全的 REST API。所以，在未来，我们将看到 REST 成为必须使用的 WordPress 功能。

# 摘要

WordPress API 是为了简化并标准化 WordPress 核心功能的使用而构建的。我们可以使用这些 API 函数来构建未来证明的解决方案，而无需在开发上浪费不必要的时间。

在本章中，我们探讨了使用 WordPress API 的重要性，并对每个 API 的功能进行了简要介绍。我们选择了三个在开发中扮演重要角色的 API，并对其进行了详细讨论。我们首先了解了短代码的结构和使用方法。然后，我们讨论了使用短代码的各种技术和位置。接下来，我们转向 Rewrite API，它管理着超出 WordPress 核心功能的自定义路由。最后，我们研究了 REST API，用于开发 API 函数并将应用程序数据暴露给第三方服务。

在第七章，“管理自定义文章类型和处理表单”，我们将通过教授用户如何构建各种功能和使用自定义表单来捕获和显示高级数据，来探讨 WordPress 开发中最重要两个方面。
