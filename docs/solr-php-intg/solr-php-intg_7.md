# 第七章。Solr 中的拼写检查

拼写检查组件可用于根据我们在索引中拥有的数据提供拼写更正的建议。在本章中，我们将看到如何在我们的索引中启用拼写检查，并使用 PHP 获取和显示拼写更正。本章将涵盖以下主题：

+   拼写检查的 Solr 配置

+   Solr 中可用的拼写检查器实现

+   使用 PHP 运行拼写检查查询

+   显示建议和整理

+   构建自动完成功能

### 注意

拼写检查适用于索引词。如果我们的索引中有拼写错误，建议也可能拼写错误。

拼写检查可用于向用户提供拼写更正的建议，提供*您是不是想要*的功能。这类似于 Google 提供的**显示结果**功能。它可用于为自动完成用户输入文本提供一系列建议。PHP 也有一个类似的功能，称为**pspell**，但这个拼写检查是建立在我们在 Solr 中创建的索引之上的。这意味着它更加定制化，适用于索引中存在的文档类型，并且可以调整以获得更符合我们喜好的结果。

# 拼写检查的 Solr 配置

Solr 安装中附带的演示模式和配置已经配置了拼写检查。让我们看看它的设置：

1.  打开`<solr_dir>/example/solr/collection1/conf`中的`solrconfig.xml`。

1.  通过名称为`spellcheck`的`searchComponent`进行搜索。

1.  在`spellcheck`组件内部有多个拼写检查器。这是 Solr 附带的`default`拼写检查器：

```php
    <lst name="spellchecker">
    <str name="name">default</str>
    <str name="field">text</str>
    <str name="classname">solr.DirectSolrSpellChecker</str>
    <float name="accuracy">0.5</float>
    <int name="maxEdits">2</int>
    <int name="minPrefix">1</int>
    <int name="maxInspections">5</int>
    <int name="minQueryLength">4</int>
    <float name="maxQueryFrequency">0.01</float>
    <float name="thresholdTokenFrequency">.01</float>
    </lst>
    ```

1.  上面的代码块显示了拼写检查中使用的各种变量。让我们来看看拼写检查配置中的重要变量，并了解它们的含义：

+   `name`：此变量指定 Solr 拼写检查器的拼写检查配置的名称。在我们的配置中，名称是`default`。

+   `field`：此变量指定用于拼写检查的字段。我们使用文本字段来加载拼写检查的标记。

+   `classname`：此变量指定正在使用的 Solr 拼写检查器的实现。我们使用`DirectSolrSpellChecker`，它直接使用 Solr 索引，不需要我们构建或重建拼写检查索引。我们还将查看其他实现。

+   `accuracy`：此变量的范围为`0.0`到`1.0`，`1.0`表示最准确。Solr 拼写检查实现使用此准确度值来决定是否可以使用结果。

+   `maxQueryFrequency`：此变量指定查询术语必须出现在文档中的最大阈值，才能被视为建议。这里设置为`0.01`。较小的阈值对于较小的索引更好。

+   `thresholdTokenFrequency`：此变量指定术语必须出现在百分之一的文档中，才能被考虑为拼写建议。这可以防止低频率的术语被提供为建议。但是，如果您的文档基数很小，您可能需要进一步减少这个值以获得拼写建议。

# Solr 中可用的拼写检查器实现

让我们来看看 Solr 提供的不同拼写检查器实现：

+   `DirectSolrSpellChecker`：此实现不需要为拼写检查构建单独的索引。它使用主 Solr 索引进行拼写建议。

+   `IndexBasedSpellChecker`：此实现用于创建和维护基于 Solr 索引的拼写词典。由于需要创建和维护一个单独的索引，我们需要在主索引发生变化时构建/重建索引。这可以通过在配置中启用`buildOnCommit`或`buildOnOptimize`来自动完成。此外，我们需要使用我们的 Solr 拼写检查组件配置中的`spellcheckIndexDir`变量来指定要创建的索引的位置。

### 注意

`buildOnCommit`组件非常昂贵。建议使用`buildOnOptimize`或在 Solr URL 中使用`spellcheck.build=true`进行显式构建。

+   基于文件的拼写检查器：此实现使用一个平面文件在 Solr 中构建拼写检查索引。由于没有可用的频率信息，使用此组件创建的索引不能用于提取基于频率的信息，例如阈值或最受欢迎的建议。文件的格式是每行一个单词，例如：

```php
    Java
    PHP
    MySQL
    Solr
    ```

索引需要使用`spellcheck.build=true`参数在我们的 Solr URL 中构建。除了`spellcheckIndexDir`位置来构建和存储索引外，`FileBasedSpellChecker`组件还需要`sourceLocation`变量来指定拼写文件的位置。

+   `WordBreakSolrSpellChecker`：此拼写检查组件通过组合相邻单词或将单词分解为多个部分来生成建议。它可以与前面的拼写检查器之一一起配置。在这种情况下，结果将被合并，整理可能包含来自两个拼写检查器的结果。

拼写检查器通常会提供通过字符串距离计算的分数排序的建议，然后按照索引中建议的频率进行排序。可以通过在配置文件中提供不同的距离计算实现使用`distanceMeasure`变量或通过提供不同的单词频率实现使用`comparatorClass`变量来调整这些参数。一些可用的`comparatorClass`实现是`score`（默认）和`freq`。类似地，`org.apache.lucene.search.spell.JaroWinklerDistance`是距离计算的实现，它在 Solr 中可用。

# 使用 PHP 运行拼写检查查询

让我们配置 Solr，使拼写检查发生在两个字段上，名称和作者：

1.  更改`schema.xml`文件的内容。创建一个新的字段，拼写检查将在该字段上进行，并使用以下代码将`name`和`author`字段复制到新字段中：

```php
    <field name="spellfld" type="text_general" indexed="true" stored="false" multiValued="true"/>
    <copyField source="name" dest="spellfld"/>
    <copyField source="author" dest="spellfld"/>
    ```

1.  在`solrconfig.xml`中更改默认拼写检查器的拼写检查字段为我们刚刚创建的新字段。默认的拼写检查器使用 Solr 提供的拼写检查器`DirectSolrSpellChecker`实现。

```php
    <lst name="spellchecker">
    <str name="name">default</str>
    <str name="field">spellfld</str>
    ```

1.  默认情况下，Solr 配置中的`/select`请求处理程序没有拼写检查设置和结果。因此，让我们在名为`/select`的`requestHandler`中添加这些变量。在这里，我们指定要使用的拼写检查词典为**default**，这是我们之前配置的，并将拼写检查组件添加为输出的一部分。

```php
    <requestHandler name="/select" class="solr.SearchHandler">
    <lst name="defaults">
    .....  
    <!-- spell check settings -->
    <str name="spellcheck.dictionary">default</str>
    <str name="spellcheck">on</str>
    </lst>

    <arr name="last-components">
    <str>spellcheck</str>
    </arr>
    ```

1.  现在重新启动 Solr，并在`exampledocs`文件夹中重新索引`books.csv`文件，以及在第五章中提供的`books.csv`文件，*使用 PHP 和 Solr 突出显示结果*。我们需要重新索引我们的书籍的原因是因为我们已经改变了我们的模式。每当模式更改并添加新字段时，需要重新索引文档以在新字段中填充数据。有关在 Solr 中索引这些 CSV 文件，请参阅第二章中的*向 Solr 索引添加示例文档*部分。

让我们使用 PHP 对作者*Stephen King*进行拼写检查，并查看 Solr 建议的更正：

1.  首先使用以下代码从选择查询中获取拼写检查组件：

```php
    $spellChk = $query->getSpellcheck();
    $spellChk->setCount(10);
    $spellChk->setCollate(true);
    $spellChk->setExtendedResults(true);
    $spellChk->setCollateExtendedResults(true);
    ```

1.  我们已经通过`setCount()`函数设置了要返回的建议数量。通过将`setCollate()`设置为`true`，我们告诉 Solr 建议原始查询字符串，并用最佳建议替换原始拼写错误的单词。`setExtendedResults()`和`setCollateExtendedResults()`函数告诉 Solr 提供有关建议和返回的整理的附加信息。如果需要，可以用于分析。

1.  执行查询后，我们需要从查询结果集中获取拼写检查组件，并用它获取建议和整理。我们使用`getCorrectlySpelled()`函数来检查查询是否拼写正确。

```php
    $resultset = $client->select($query);
    $spellChkResult = $resultset->getSpellcheck();
    if ($spellChkResult->getCorrectlySpelled()) {
    echo 'yes';
    }else{
    echo 'no';
    }
    ```

1.  接下来，我们循环遍历拼写检查结果，并针对查询中的每个术语获取建议和相关详细信息，例如建议的数量、原始术语的频率以及建议的单词及其出现频率。

```php
    foreach($spellChkResult as $suggestion) {
    echo 'NumFound: '.$suggestion->getNumFound().'<br/>';
    echo 'OriginalFrequency: '.$suggestion->getOriginalFrequency().'<br/>';    
    foreach ($suggestion->getWords() as $word) {
    echo 'Frequency: '.$word['freq'].'<br/>';
    echo 'Word: '.$word['word'].'<br/>';
    }
    }
    ```

1.  同样，我们获取整理并循环遍历它以获取更正后的查询和命中。我们还可以获取查询中每个术语的更正详细信息。

```php
    $collations = $spellChkResult->getCollations();
    echo '<h1>Collations</h1>';
    foreach($collations as $collation) {
    echo 'Query: '.$collation->getQuery().'<br/>';
    echo 'Hits: '.$collation->getHits().'<br/>';
    foreach($collation->getCorrections() as $input => $correction) {
    echo $input . ' => ' . $correction .'<br/>';
    }
    }
    ```

# 使用 PHP 和 Solr 实现自动完成功能

可以通过在 Solr 中创建一个 Suggester 并使用 Solarium 中可用的 Suggester 来构建自动完成功能。自动完成的目的是根据不完整的用户输入建议查询术语。Suggester 的工作方式与拼写检查功能非常相似。它可以在主索引或任何其他字典上工作。

首先让我们更改`schema.xml`文件，添加一个名为`suggest`的拼写检查组件：

```php
<searchComponent name="suggest" class="solr.SpellCheckComponent">
<lst name="spellchecker">
<str name="name">suggest</str>
<str name="field">suggestfld</str>
<str name="classname">org.apache.solr.spelling.suggest.Suggester</str>
<str name="lookupImpl">org.apache.solr.spelling.suggest.tst.TSTLookup</str>
<str name="storeDir">suggest_idx</str>
<float name="threshold">0.005</float>
<str name="buildOnCommit">true</str>
</lst>
</searchComponent>
```

我们已经指定了用于建议的字段为`suggestfld`。用于构建 Suggester 的 Solr 组件在类名中被称为`org.apache.solr.spelling.suggest.Suggester`。阈值是一个介于 0 和 1 之间的值，指定了术语应出现在多少文档中才能添加到查找字典中的最小分数。我们将索引存储在`suggest_idx`文件夹中。`lookupImpl`组件提供了用于创建建议的`inmemory`查找实现。Solr 中可用的查找实现有：

+   `JaspellLookup`：这是基于 Jaspell 的基于树的表示。Jaspell 是一个创建拼写校正的复杂基于树的结构的 Java 拼写检查包。它使用一种称为`trie`的数据结构。

+   `TSTLookup`：这是一种简单而紧凑的三叉树表示，能够立即更新数据结构。它还使用`trie`数据结构。

+   `FSTLookup`：这是基于自动机的表示。它构建速度较慢，但在运行时消耗的内存要少得多。

+   `WFSTLookup`：这是加权自动机表示，是`FSTLookup`的另一种更精细的排名方法。

您可以更改查找实现并查看建议的变化。由于建议是基于索引的，因此索引越大，建议就越好。

让我们在 Solr 中为建议创建一个单独的请求处理程序，并将我们的建议拼写检查作为其中的一个组件添加进去。提供建议的默认配置选项已经合并在请求处理程序本身中。

```php
<requestHandler class="org.apache.solr.handler.component.SearchHandler" name="/suggest">
<lst name="defaults">
<str name="spellcheck">true</str>
<str name="spellcheck.dictionary">suggest</str>
<str name="spellcheck.onlyMorePopular">true</str>
<str name="spellcheck.count">5</str>
<str name="spellcheck.collate">true</str>
</lst>
<arr name="components">
<str>suggest</str>
</arr>
</requestHandler>
```

接下来，我们需要在我们的`schema.xml`中创建一个单独的字段，该字段被索引。我们将书名、作者和标题复制到该字段中，以便对它们进行建议。

```php
<field name="suggestfld" type="text_general" indexed="true" stored="false" multiValued="false"/>

<copyField source="name" dest="suggestfld"/>
<copyField source="author" dest="suggestfld"/>
<copyField source="title" dest="suggestfld"/>
```

完成后，重新启动 Apache Tomcat Web 服务器，并使用以下 URL 构建拼写检查索引：

```php
http://localhost:8080/solr/collection1/suggest/?spellcheck.build=true
```

### 注意

我们创建了一个名为 suggest 的单独请求处理程序，因此我们的 URL 为/`suggest`/而不是/`select`/。

现在让我们看看 Solarium 库提供的用于与 PHP 集成的 Suggester。首先，我们需要从 Solarium 客户端创建一个 Suggester 查询，而不是普通查询。

```php
$client = new Solarium\Client($config);
$suggestqry = $client->createSuggester();
```

接下来，我们必须设置要使用的请求处理程序。请记住，我们创建了一个名为**suggest**的单独请求处理程序来提供建议。还要设置我们要使用的字典。我们可以创建多个字典，并使用以下函数在运行时更改它们：

```php
$suggestqry->setHandler('suggest');
$suggestqry->setDictionary('suggest');
```

现在提供 Suggester 的查询。设置要返回的建议数量。打开`collation`标志和`onlyMorePopular`标志。

```php
$suggestqry->setQuery('ste');
$suggestqry->setCount(5);
$suggestqry->setCollate(true);
$suggestqry->setOnlyMorePopular(true);
```

使用`suggester()`函数执行查询，然后循环遍历结果集以获取所有术语及其建议。可以使用`getQuery()`函数显示原始查询。

```php
$resultset = $client->suggester($suggestqry);
echo "Query : ".$suggestqry->getQuery();
foreach ($resultset as $term => $termResult) {
    echo '<strong>' . $term . '</strong><br/>';
    echo 'Suggestions:<br/>';
    foreach($termResult as $result){
        echo '-> '.$result.'<br/>';
    }
}
```

最后，使用以下代码获取并显示整理：

```php
echo 'Collation: '.$resultset->getCollation();
```

这段代码可以用来创建一个 AJAX 调用，并提供 JSON 或 XML 字符串作为自动完成建议。

# 摘要

我们从理解 Solr 上的拼写检查是如何工作开始。我们通过配置 Solr 来创建拼写检查索引，并看到了 Solr 提供的不同拼写检查实现。我们了解了 Solr 中拼写检查的一些微调选项。接下来，我们在 Solr 中创建了一个用于对书名和作者进行拼写检查的字段，并配置 Solr 来使用这个字段提供拼写建议。我们看到了一种可以用于提供自动完成拼写建议的拼写检查变体。我们为自动完成建议创建了一个单独的 Solr 索引，并看到了一个 PHP 代码，它接受一个三个字符的单词，并从索引中提供建议。
