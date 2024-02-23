# 第五章：使用 PHP 和 Solr 突出显示结果

Solr 提供的高级功能之一是在搜索返回的结果中突出显示匹配的关键字。除了突出显示的匹配项之外，还可以指定我们希望 Solr 每个字段返回的突出显示片段的数量。在本章中，我们将使用 PHP 和 Solarium 库探索 Solr 的所有突出显示功能。我们将涵盖的主题包括：

+   Solr 突出显示配置

+   使用 PHP 和 Solarium 在 Solr 中进行突出显示

+   为不同字段使用不同的突出显示标记

+   使用快速向量突出显示器进行突出显示

### 注意

需要在 Solr 中存储需要进行突出显示的字段。

# Solr 高亮配置

Solr 有两种类型的突出显示器——**常规突出显示器**和**快速向量突出显示器**。常规突出显示器适用于大多数查询类型，但不适用于大型文档。另一方面，快速向量突出显示器非常适用于大型文档，但支持的查询类型较少。尽管我个人还没有遇到快速向量突出显示器无法工作的情况。

### 注意

快速向量突出显示器需要设置`termVectors`，`termPositions`和`termOffsets`才能工作。

让我们看一下用于突出显示的 Solr 配置。打开`<solr_directory>/example/solr/collection1/conf/solrconfig.xml`中的 Solr 配置。搜索具有属性`class="solr.HighlightComponent"`和`name="highlight"`的 XML 元素`searchComponent`。我们可以看到文件中定义了多个**fragmenters**，一个 HTML **formatter**和一个 HTML **encoder**。我们还在文件中定义了多个`fragmentsBuilders`，多个`fragListBuilders`和多个`boundaryScanners`，如下列表所述：

+   **Fragmenter:** 它是用于突出显示文本的文本片段生成器。默认的片段生成器是由`default="true"`标记的间隙。

+   **格式化程序**：用于格式化输出，并指定要用于突出显示输出的 HTML 标记。标记是可定制的，并且可以在 URL 中传递。

+   **fragListBuilder:** 仅与`FastVectorHighlighter`一起使用。用于定义由`FastVectorHighlighter`创建的片段的大小（以字符为单位）。默认的`fragListBuilder`是`single`，可以用来指示应使用整个字段而不进行任何分段。

+   **fragmentsBuilder**：与`FastVectorHighlighter`一起使用，用于指定用于突出显示的标记。可以通过使用`hl.tag.pre`和`hl.tag.post`参数进行覆盖。

+   **boundaryScanner**：仅为`FastVectorHighlighter`定义边界如何确定。默认的`boundaryScanner`将边界字符定义为`.,!?\t\n`和空格。

### 注意

可以从以下 URL 获取有关突出显示参数的更多详细信息：[`cwiki.apache.org/confluence/display/solr/Standard+Highlighter`](https://cwiki.apache.org/confluence/display/solr/Standard+Highlighter)

# 使用 PHP 和 Solarium 在 Solr 中进行突出显示

让我们尝试使用 PHP 进行常规突出显示。在我们的索引中搜索`harry`，并突出显示两个字段——`name`和`series_t`，如以下代码所示：

```php
  $query->setQuery('harry');
  $query->setFields(array('id','name','author','series_t','score','last_modified'));
```

首先从以下查询中获取突出显示组件：

```php
  $hl = $query->getHighlighting();
```

使用以下查询设置我们想要突出显示的字段：

```php
  $hl->setFields('name,series_t');
```

使用以下查询将突出显示的 HTML 标记设置为粗体：

```php
  $hl->setSimplePrefix('<strong>');
  $hl->setSimplePostfix('</strong>');
```

设置要为每个字段生成的突出显示片段的最大数量。在这种情况下，可以生成从 0 到 2 个突出显示片段，如以下查询所示：

```php
  $hl->setSnippets(2);
```

设置要考虑进行突出显示的片段的字符大小。0 使用整个字段值而不进行任何分段，如以下查询所示：

```php
  $hl->setFragSize(0);
```

将`mergeContiguous`标志设置为将连续的片段合并为单个片段，如以下代码所示：

```php
  $hl->setMergeContiguous(true);
```

将`highlightMultiTerm`标志设置为启用范围、通配符、模糊和前缀查询的高亮显示，如下面的查询所示：

```php
  $hl->setHighlightMultiTerm(true);
```

一旦查询运行并接收到结果集，我们将需要从结果集中检索高亮显示的结果，使用以下查询：

```php
  $hlresults = $resultSet->getHighlighting();
```

对于结果集中的每个文档，我们将需要从高亮结果集中获取高亮文档。我们将需要在`getResult()`函数中传递唯一 ID 作为标识符，以获取高亮文档，如下面的代码所示：

```php
foreach($resultSet as $doc)
{
  $hldoc = $hlresults->getResult($doc->id);
  $hlname = implode(',',$hldoc->getField('name'));
  $hlseries = implode(',',$hldoc->getField('series_t'));
}
```

这里为每个文档的高亮字段，我们使用`getField()`方法获取，返回为一个数组。这就是为什么我们必须在显示之前将其 implode。我们可以看到在输出中，字段使用加粗的`<strong>`和`</strong>`标记进行高亮显示。

在 Solr 日志中，我们可以看到我们在 PHP 代码中指定的所有参数，如下所示：

```php
  336647163 [http-bio-8080-exec-1] INFO  org.apache.solr.core.SolrCore  – [collection1] webapp=/solr path=/select params={**hl.fragsize=0&hl.mergeContiguous=true&hl.simple.pre=**    **<strong>&hl.fl=name,series_t**&wt=json&**hl=true**    &rows=25&**hl.highlightMultiTerm=true**&omitHeader=true&fl=id,name,author,series_t,score,last_modified&**hl.snippets=2**&start=0&q=harry&**hl.simple.post=</strong>**} hits=7 status=0 QTime=203
```

传递给启用高亮的参数是`hl=true`，要高亮显示的字段指定为`hl.fl=name,series_t`。

# 为不同字段使用不同的高亮标记

我们可以为不同的字段使用不同的高亮标记。让我们使用`bold`标记为`name`添加高亮显示，使用`italics`标记为`series`添加高亮显示。在我们的代码中设置`per`字段标记，如下所示：

```php
  $hl->getField('name')->setSimplePrefix('<strong>')->setSimplePostfix('</strong>');
  $hl->getField('series_t')->setSimplePrefix('<em>')->setSimplePostfix('</em>');
```

输出显示，字段`name`被加粗标记，而字段`series`被斜体标记，如下面的截图所示：

![为不同字段使用不同的高亮标记](img/4920_05_01.jpg)

使用不同的标记高亮显示不同的字段。

我们还可以使用`setQuery()`函数为高亮结果设置单独的查询，而不是正常的查询。在之前的程序中，让我们将高亮显示更改为在搜索`harry`时发生在`harry potter`上，如下面的代码所示：

```php
  $hl->setQuery('harry potter');
```

在检查 Solr 日志时，可以看到用于高亮显示的查询作为`hl.q`参数传递给 Solr，如下面的代码所示：

```php
  344378867 [http-bio-8080-exec-9] INFO  org.apache.solr.core.SolrCore  – [collection1] webapp=/solr path=/select params={f.series_t.hl.simple.pre=<i>&f.name.hl.simple.post=</b>&f.name.hl.simple.pre=<b>&hl.fl=name,series_t&wt=json&hl=true&rows=25&omitHeader=true&hl.highlightMultiTerm=true&fl=id,name,author,series_t,score,last_modified&f.series_t.hl.simple.post=</i>&hl.snippets=2&start=0&q=harry&**hl.q=harry+potter**} hits=7 status=0 QTime=27
```

# 使用快速向量高亮显示

让我们更改`schema.xml`，并为两个字段`name`和`*_t`启用**termVectors**、**termPositions**和**termOffsets**（这将匹配所有以`_t`结尾的字段-`series_t`）。

```php
  <field name="name" type="text_general" indexed="true" stored="true" termVectors="true" termPositions="true" termOffsets="true"/>
  <dynamicField name="*_t"  type="text_general" indexed="true"  stored="true" termVectors="true" termPositions="true" termOffsets="true"/>
```

重新启动 Tomcat。根据您的系统（Windows 或 Linux）和安装类型，重新启动 Tomcat 的机制将有所不同。请查看 Tomcat 文档以了解如何重新启动 Tomcat。

由于模式现在已更改，我们需要重新索引我们在第二章中索引的所有文档，*从 Solr 插入、更新和删除文档*。还要索引本章的`books.csv`文件。在代码中，启用快速高亮显示，并设置用于高亮显示的`fragmentsBuilder`（HTML 标记），如下面的查询所示：

```php
  $hl->setUseFastVectorHighlighter(true);
  $hl->setFragmentsBuilder('colored');
```

在输出中，我们可以看到`harry`被高亮显示。要更改默认的高亮显示，我们需要在`solrconfig.xml`文件中添加一个新的**fragmentsBuilder**。浏览`solrconfig.xml`文件，并搜索带有名称 colored 的`fragmentsBuilder`标记。这有两个属性——`hl.tag.pre`和`hl.tag.post`。我们可以在这里为快速向量高亮显示指定前置和后置标记。在它之后创建一个名为`fasthl`的新`fragmentsbuilder`，如下面的代码所示：

```php
  <fragmentsBuilder name="fasthl" class="solr.highlight.ScoreOrderFragmentsBuilder">
  <lst name="defaults">
  <str name="hl.tag.pre"><![CDATA[<b style="background:cyan">]]></str>
  <str name="hl.tag.post"><![CDATA[</b>]]></str>
  </lst>
  </fragmentsBuilder>
```

重新启动 Tomcat，并更改 PHP 代码以使用这个新的`fragmentbuilder`进行高亮显示，如下面的查询所示：

```php
  $hl->setFragmentsBuilder('fasthl');
```

现在输出将包含以浅蓝色高亮显示的`harry`。

还可以使用`setTagPrefix()`和`setTagPostfix()`函数在运行时更改高亮显示标记。在下面的代码中，我们正在将快速向量高亮显示的标记更改为石灰色：

```php
  $hl->setTagPrefix('<b style="background:lime">')->setTagPostfix('</b>');
```

配置文件用于设置默认的高亮显示标记。标记可以通过 PHP 函数调用在运行时进行更改，以进行格式化。

以下是 Solarium 中的一些其他可用函数，可根据您的要求进行突出显示：

+   `setUsePhraseHighlighter(boolean $use)`: 设置为`true`，只有当短语项出现在文档的查询短语中时才进行突出显示。默认值为`true`。

+   `setRequireFieldMatch(boolean $require)`: 设置为`true`，只有在此特定字段中查询匹配时才突出显示字段。默认情况下，这是 false，因此无论哪个字段匹配查询，都会在所有请求的字段中突出显示项。需要`setUsePhraseHighlighter(true)`。

+   `setRegexPattern(string $pattern)`: 仅在常规突出显示器中使用。用于设置片段的正则表达式。

+   `setAlternateField(string $field)`: 如果没有匹配的项，也无法生成摘要，我们可以设置一个备用字段来生成摘要。

+   `setMaxAlternateFieldLength(int $length)`: 仅当设置了备用字段时使用。它指定要返回的备用字段的最大字符数。默认值为“无限制”。

# 摘要

我们看到了如何使用 PHP 代码向 Solr 请求突出显示的搜索结果。我们看到了常规和快速向量突出显示器。我们看到了用于更改常规和快速向量突出显示器的突出显示标记的函数和参数。我们还通过一些函数和 Solr 配置和模式更改来调整突出显示和生成的摘要。

在下一章中，我们将深入探讨评分机制。我们将探索调试和统计组件，这将使我们能够改进相关性排名并从索引中获取统计信息。
