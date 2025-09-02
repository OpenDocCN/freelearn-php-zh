# 附录

# 常见问题解答

经过多年的帮助开发者适应 Liquid，出现了许多特定的情况和问题，对于这些问题的答案并不容易找到。现在我们将列出最常见的情况，并提供一些建议，这些建议将有助于提高你对 Shopify 和 Liquid 的了解，并使你的工作变得更加容易：

1.  *是否可以在代码编辑器中找到特定代码的位置，而无需手动搜索每个模板、部分或代码片段？*

    开发者在开始使用**Shopify**时遇到的最大问题之一是，他们很难在不同的模板、部分和代码片段中找到特定的代码片段。虽然经过一段时间，我们确实会习惯于更有效地查找东西，但一开始可能会相当麻烦。

    为了帮助我们解决这个问题，我们可以使用一个叫做**Shopify Theme Search by Bold**的**Chrome**插件。这个小插件在代码编辑器内提供了一个搜索输入框，我们可以输入任何想要查找的字符串。几秒钟后，它会高亮显示包含搜索字符串的每一个目录和文件。你可以在[`chrome.google.com/webstore/detail/shopify-theme-search-by-b/epbnmkionkpliaiogpemfkclmcnbdfle`](https://chrome.google.com/webstore/detail/shopify-theme-search-by-b/epbnmkionkpliaiogpemfkclmcnbdfle)了解更多信息。

1.  *无论我们创建多少新的副本主题，出于某种原因，副本主题都没有保存任何之前所做的自定义设置。* *在这种情况下我们应该怎么办呢？*

    第二个最棘手的问题发生在主题复制过程中。在创建主题副本后，我们可能会注意到副本主题已经完全重置。它不包含我们在主题编辑器中之前设置的任何自定义设置。

    问题出在`settings_data.json`文件中，如果我们打开它，我们会看到它是空的，在某些情况下，文件可能完全缺失。这种发生的原因很明显——Shopify 没有正确地复制文件内容——但问题是**为什么**？如果我们导航到最初创建副本的主题，打开`settings_data.json`文件，并尝试将其内容复制到新的副本主题中，由于 JSON 文件中的错误，我们无法保存文件。如果我们尝试将复制的内容保存到新文件中，Shopify 将不允许这样做，并给我们提供一个错误代码，指示有问题的值。

    为了解决这个问题，我们需要手动搜索错误中收到的每个值，并相应地更新它。可能出现的问题包括，我们在删除特定部分块之前没有从主题编辑器内部删除该块，就完全删除了该部分。Shopify 会继续尝试加载该块。然而，由于我们已经删除了它，它将无法加载，并会破坏 JSON 文件。第二个最常见的问题是使用`range`输入类型时，输入的当前值超过了最小值和最大值之间的范围。只有在解决所有这些问题之后，我们才能正确保存文件。

1.  *是否可以恢复已删除的主题？*

    每个商店允许我们保留最多 20 个重复的主题作为备份，在**Shopify Plus**计划中最多可以保留 50 个重复的主题。如果我们因任何原因删除主题文件，并且没有在 GitHub 或本地保存备份文件，该文件将永久丢失，我们将无法以任何方式恢复它。

1.  *我们如何找到特定商店正在使用的应用程序？*

    虽然我们无法看到商店使用的每种应用程序，但我们可以通过使用**Fera.ai 的 Shopify App Detector**或**Koala Inspector - 检查 Shopify 商店**Chrome 插件来检索大多数应用程序的名称。您可以通过以下链接获取有关 Fera.ai Shopify App Detector 的更多信息：[`chrome.google.com/webstore/detail/shopify-app-detector-by-f/lhfdhjladfcmghahdbcmlceajdlbkale`](https://chrome.google.com/webstore/detail/shopify-app-detector-by-f/lhfdhjladfcmghahdbcmlceajdlbkale).

    您可以通过以下链接获取有关 Koala Inspector - 检查 Shopify 商店的更多信息：[`chrome.google.com/webstore/detail/koala-inspector-inspect-s/hjbfbllnfhppnhjdhhbmjabikmkfekgf`](https://chrome.google.com/webstore/detail/koala-inspector-inspect-s/hjbfbllnfhppnhjdhhbmjabikmkfekgf).

1.  *我们如何轻松识别导致商店变慢的 Liquid 代码？*

    为了帮助我们，Shopify 推出了一款名为**Shopify Theme Inspector for Chrome**的插件，我们可以使用它来获取详细的分析渲染配置文件，并帮助我们缩小导致最大延迟的文件。您可以通过以下链接获取有关 Shopify Theme Inspector for Chrome 的更多信息：[`chrome.google.com/webstore/detail/shopify-theme-inspector-f/fndnankcflemoafdeboboehphmiijkgp`](https://chrome.google.com/webstore/detail/shopify-theme-inspector-f/fndnankcflemoafdeboboehphmiijkgp).

1.  *当我们尝试使用 Shopify 合作伙伴平台向客户发送协作账户请求时，由于某种原因，我们始终收到消息说商店 URL 未与商店关联。* *我们该如何解决这个问题？*

    原因在于，在大多数情况下，店主已经设置了一个自定义域名，其 URL 与我们要求发送协作账户请求的 [myshopify.com](http://myshopify.com) URL 相差甚远，所以将 `.com` 替换为 `.myshopify.com` 并不能解决我们的问题。我们可以通过简单地导航到他们提供给我们的 URL，打开检查器并在控制台中输入 `Shopify` 来轻松解决这个问题，这将提示一系列关于商店的各种信息。其中，我们将找到一个正确的 [myshopify.com](http://myshopify.com) URL。请注意，`Shopify` 关键字是区分大小写的。

1.  *Shopify 代码编辑器有暗黑模式吗？*

    答案是肯定的！但它相当隐蔽。为了揭示它，导航到代码编辑器并点击按钮以展开编辑器屏幕。我们可以在编辑器屏幕的右上角找到它。一旦我们展开了编辑器屏幕，我们会注意到现在我们的右侧有两个滚动条。如果我们滚动到页面底部，我们会找到允许我们更改编辑器颜色模式的 **白色** 和 **黑色** 按钮。然而，请注意，当我们关闭展开的编辑器视图时，编辑器将恢复到默认颜色。

1.  *有没有一种方法可以自动格式化 Liquid 文件中的代码呢？*

    *答案是肯定的!* 我们可以使用 *CTRL + A* 在代码编辑器中突出显示整个 Liquid 文件，然后按 *Shift + Tab*，自动根据整个文件进行格式化。然而，虽然自动格式化对大多数文件都有帮助，但请注意，自动格式化在 `section` 架构标签及其内容上不适用。
