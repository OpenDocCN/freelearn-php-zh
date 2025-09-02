# 第十章：XML 和 JSON

到目前为止，我们一直在使用 HTML。它是网页的格式，因此我们用它来创建静态网页。因此，HTML 是服务器和客户端在**.html**文件页面加载期间交换的数据的格式。

当我们动态创建网页时，我们的 PHP 代码会在数据发送时组合 HTML 行，这将是我们真正交换的数据，而不是你**.php**文件的原始内容。当我们使用 Ajax 时，我们没有在我们的 jQuery Ajax 方法`.post()`或`.ajax()`中指定数据格式作为参数，因为这被隐含地理解为将是 HTML。

在本章中，我们介绍了两种不同的数据格式，XML 和 JSON。在网络开发中，我们可以以不同的方式使用它们。首先，它们可以成为你数据库的替代品。对于简单的项目，你的数据可以存储在文本文件中，而不是完整的数据库中。你甚至可以将其存储为纯文本。

下一个，也是更好的水平，将是使用 XML 文件或 JSON 作为你的数据。它们同样是纯文本，但以特殊的方式格式化。以该格式存储的数据可以轻松转换为 HTML，这使我们回到了之前的场景。

另一方面，我们可以首先将数据发送到客户端，然后在客户端进行处理以创建我们页面的内容。在本章中，我们将涵盖这两种场景。

# XML

**可扩展标记语言** (**XML**) 是一种被广泛采用的格式，用于公司、文档和程序之间交换信息。它使用与 HTML 类似的标记符号。实际上，HTML 是 XML 的一个特例。在 HTML 中，标签具有意义，并且旨在由浏览器进行解释。在 XML 中，标签可以代表任何东西，也可以什么都不代表，但规则更为严格。当然，如果你是 XML 文件的创建者，我们希望标签对你来说有一定的意义。

作为一名网络开发者，你可能永远不会使用 XML，但了解它的存在很重要。它被创建成一种既适合机器读取又适合人类阅读的数据格式。人类将能够阅读其中的内容，因为 XML 文件基本上是文本文件；计算机程序可以读取它们，因为它们结构良好。XML 格式在多个领域得到应用。我们在这里只提及其中的一些：

+   **网络服务**：有些人或公司通过提供网络服务来允许访问他们的数据库中的信息。如何创建网络服务超出了本书的范围。网络服务通常提供给你的能力是能够访问一个特定的网络地址，其中查询字符串表明你正在寻找的信息类型。访问这个（虚拟）网站的结果是 XML 格式的输出流，然后你可以对其进行处理。我们将教你如何进行处理，而不是创建服务。

+   **数据库的替代方案**：对于简单的应用程序或其部分，使用完整的数据库可能有些过度。正如我们提到的，你可以使用纯文本文件，或者更常见的是 `.csv` 文件。使用 XML 文件来做这件事会更好，因为它有更多的结构，而且很可能正是客户想要的那个结构。

## XML 格式

XML 文件的格式实际上非常简单。它是一个以类似以下行开始的文本文件：

```php
<?xml version="1.0" encoding="utf-8"?>
```

这行提到了 XML 的版本，到目前为止只有 1.0 或 1.1。规则如此简单，几乎没有空间进行更改和编码。这被称为**XML 声明**。随后的所有内容都是实际的 XML 文档，它由标签或元素组成，每个开标签都有一个对应的闭标签，以及它们之间的文本。看看这个第一个例子：

```php
<?xml version="1.0"encoding="UTF-8"?>
<!— A list of people that like California —>
<californiapeople>
  <person id="1">
    <name>Adams</name>
    <first>Ansel</first>
    <profession>photographer</profession>
    <born>San Francisco</born>
    <picture/>
  </person>
  <person>
    <name>Muir</name>
    <first>John</first>
    <profession>photographer</profession>
    <born>Scotland</born>
  </person>
  <person>
    <name>Schwarzenegger</name>
    <first>Arnold</first>
    <profession>governator</profession>
    <born>Germany</born>
  </person>
  <person>
    <name>Rowell</name>
    <first>Galen</first>
    <profession>photographer</profession>
    <born>Oakland CA</born>
  </person>
  <person>
    <name>Wellens</name>
    <first>Paul</first>
    <profession>travel guide</profession>
    <born>Antwerp Belgium</born>
  </person>
</californiapeople>
```

如你所见，每个开标签都有一个对应的闭标签，并且有一个标签 `<californiapeople>` 首次出现且仅出现一次。这是**根标签**。标签之间可以有其他标签，但它们必须正确嵌套。我已经给第一个人的标签添加了一个**属性**，并且还包含了一个**空标签**。就像 HTML 中的 `<br/>` 标签一样，`<picture/>` 是 `<picture></picture>` 的缩写表示。这样的元素被称为**自闭合元素**。

由于我时不时地使用 XML，我养成了在 HTML 文件中始终使用闭标签的习惯，即使在不强制的情况下也是如此，例如在 HTML5 中。其他明显的区别是，在 XML 文件中，空白不会截断，元素名称的大小写敏感，所以 `<Person>` 与 `<person>` 不同。

最后，正如预期的那样，有一些字符你不能在 XML 文件的文本中使用：`<` 和 `&`。XML 会认为它已经到达了开标签的开始。如果你需要 `<` 符号，请使用 `&lt;` 代替。你应该记得我们在几章前讨论 HTML 实体时提到的这一点。XML 中使用的其他实体还有 `&amp`，它解决了“先有鸡还是先有蛋”的问题，即如果 `&lt;` 表示 `<`，那么单独使用 `&` 也是不允许的。`&gt;`、`&apos;` 和 `&quot;` 也可以使用，尽管 `>`、`'` 和 `"` 是合法字符。如果你需要它们作为文本的一部分，我们建议你使用实体。顺便说一句，你注意到注释的语法也与 HTML 相同吗？还有更多相似之处，我们甚至可以给 XML 文件添加样式！

## 显示 XML 文件

第一次查看 XML 文件时，你可能看不到你期望的内容。显示 XML 文件的最佳方式是使用浏览器，浏览器会像处理 HTML 文件一样处理该文件；因此，你将看不到标签，只能看到它们之间的文本。这也是我喜欢使用 Firefox 作为浏览器的原因之一。

Firefox 和 Chrome 会显示整个 XML 文件，包括标签。标签名称以不同的颜色显示，并且所有内容都进行了适当的缩进。Firefox 还会检查它是否是一个格式良好的 XML 文件，如果不是，会生成一个有意义的错误消息，指向文件中的问题行。

有时 Firefox 会显示：“此 XML 文件似乎没有与之关联的任何样式信息。以下显示文档树。”嗯，这个 Firefox 消息暗示了 XML 文件可能有样式表，不是吗？那甚至可能是一个 CSS 文件吗？是的，可以。只需在 XML 声明之后添加以下行：

```php
<?xml-stylesheet type="text/css" href="california.css"?>
```

然后，创建一个包含以下内容的 CSS 文件。现在，Firefox 将仅显示您文件的正文，而不是标签，并使用一些奇特的颜色。在本章的后面部分，您将了解到可能存在其他不是 CSS 文件的样式表。

由于 XML 的特性，以下的所有代码示例都相当长。您可能需要考虑访问 Packt Publishing 网站上本书的在线页面，并在那里获取它们：

```php
californiapeople {
  background-color: #FFDEAD;
}
name, first {
  color:teal;
  font-size:20px;
  margin-top:30px;
}
name {
margin-left:25px;
}
profession, born
{
  display:block;
  margin-left:50px;
  font-size:16px;
  font-family:Baskerville, "Times New Roman",  serif;
  color:blue;
}
```

## XML 编辑器

由于 XML 文件是文本文件，你可以使用任何文本编辑器来创建和编辑它们。**Textastic**是一个出色的编辑器，只需花费几美元，我很喜欢使用，同时也有适用于 MacOS 和 iOS 的版本。**Dreamweaver**用户将受益于其验证 XML 文件正确语法的功能。

然而，在 XML 的世界里，验证可以超越简单的语法。

## XML Schema

假设您正在与几个需要向您提供数据的客户合作，例如员工记录，您要求他们使用 XML 格式来这样做。您已经给自己找了一个大麻烦！随着数据的到来，您意识到一个客户使用了`<name>`元素来表示姓氏，而另一个客户则决定使用`<last>`。不仅如此，他们可能使用了不同的顺序和组合。

您首先应该做的是给出您希望 XML 文件看起来是什么样的定义。然后，您就可以开始编写应用程序，在您收到数据之前处理这些数据。

在定义 XML 文件结构时，使用了几个约定。其中一个被称为**文档类型定义**（**DTD**），另一个则是**XML 模式**。XML 模式格式的优点在于，描述你的 XML 文件结构的文件本身也是一个 XML 文件。本书的范围不包括介绍这两种方法。不过，这里提供了一些 XML 模式文件的示例。

作为开始，这里是我们之前示例中结构相似的 XML 模式定义：

```php
<xs:schema >
  <xs:element name="californiapeople">
    <xs:complexType>
      <xs:sequence>
        <xs:element name="person" maxOccurs="unbounded">
          <xs:complexType>
            <xs:sequence>
              <xs:element type="xs:string" name="name"/>
              <xs:element type="xs:string" name="first"/>
              <xs:element type="xs:string" name="profession"/>
              <xs:element type="xs:string" name="born"/>
              <xs:element type="xs:string" name="photograph"/>
            </xs:sequence>
           <xs:attribute type="xs:byte" name="id" use="optional"/>
          </xs:complexType>
        </xs:element>
      </xs:sequence>
    </xs:complexType>
  </xs:element>
```

如您可能已经推断出的，这是对 XML 文件中可以包含的所有元素的描述，它们可以有哪些属性，它们应该以何种顺序出现，等等。

## SimpleXML

假设我们在服务器上有一个包含我们想要用于动态生成网页的数据的 XML 文件，就像之前在 PHP 程序中从 MySQL 数据库中提取数据一样。这该如何实现？

有几种方法。在这个例子中，使用了 **SimpleXML**。它由几个 PHP 类组成，具有处理现有 XML 文件或创建新文件的方法。以下示例取自我的网站的一个旧版本，其中包含许多照片画廊。照片信息存储在一个 XML 文件中，每个画廊一个。每次需要将照片添加到网站上时，只需向适当的 XML 文件中添加一个 `node` 即可。

我已经简化了 XML 文件内的文本，否则这部分章节就会变成一本旅行指南。

### 这是 XML 文件

使用你喜欢的编辑器创建 `practical.xml`：

```php
<?xml version="1.0" encoding="utf-8"?>
<photocollection>
<title>June Lake</title>
<overview>
The June Lake Loop begins just five miles south of Lee Vining, on US 395, ...
</overview>
<photo>
<scaption>June Lake in the Fall</scaption>
<caption>June Lake and Carson Peak in the fall</caption>
<id>Juneinthefall-31-21</id>
<story>
Each time that unfortunate day arrives that I have to leave June Lake ...
</story>
<thumbnail></thumbnail>
<smallimg>imagessmall/junelakefall.jpg</smallimg>
<largeimg>imagespng/junelakeinthefall.png</largeimg>
<photoshop></photoshop>
<date></date>
<camera>Nikon F6</camera>
</photo>
<photo>
<scaption>Aspen by Silver Lake</scaption>
<caption>Aspen trees by Silver Lake</caption>
<id>silverlakeaspenfall98</id>
<story>In 1998, I hit the right week of the year for fall colors. I parked by Silver Lake ...
</story>
<smallimg>imagessmall/silverlakeaspenfall98.jpg</smallimg>
<largeimg>imagespng/silverlakeaspenfall98.png</largeimg>
<photoshop></photoshop>
<date></date>
<camera>Hasselblad</camera>
</photo>
<photo>
<scaption>Gull Lake in the Fall</scaption>
<caption>Gull Lake in the Fall - Happy fishermen !</caption>
<id>gullake-648</id>
<story>
If you take the north shore road around June Lake there is a turnoff, ...
</story>
<smallimg>imagessmall/gulllake648.jpg</smallimg>
<largeimg>imageslarge/gulllake648.jpg</largeimg>
<photoshop></photoshop>
<date></date>
<camera>Nikon D1</camera>
</photo>
<photo>
<scaption>Silver Lake</scaption>
<caption>Silver Lake - June Lake Loop</caption>
<id>silver2</id>
<story>
Any time of the year, there can be snow in June Lake. That year there was fresh snow ...
</story>
<smallimg>imagessmall/silver2.jpg</smallimg>
<largeimg>imagespng/silver2.png</largeimg>
<photoshop></photoshop>
<date></date>
<camera>Hasselblad</camera>
</photo>
</photocollection>
```

### XML 架构文件

这是 XML 架构文件 `photocollection.xs`，如果你对它的样子感兴趣的话：

```php
<xs:schema >
  <xs:element name="photocollection">
    <xs:complexType>
      <xs:sequence>
        <xs:element type="xs:string" name="title"/>
        <xs:element type="xs:string" name="overview"/>
        <xs:element name="photo" maxOccurs="unbounded"/>
          <xs:complexType>
            <xs:sequence>
              <xs:element type="xs:string" name="scaption"/>
              <xs:element type="xs:string" name="caption"/>
              <xs:element type="xs:string" name="id"/>
              <xs:element type="xs:string" name="story"/>
              <xs:element type="xs:string" name="thumbnail"/>
              <xs:element type="xs:string" name="smallimg"/>
              <xs:element type="xs:string" name="largeimg"/>
              <xs:element type="xs:string" name="photoshop"/>
              <xs:element type="xs:string" name="date"/>
              <xs:element type="xs:string" name="camera"/>
            </xs:sequence>
          </xs:complexType>
        </xs:element>
      </xs:sequence>
    </xs:complexType>
  </xs:element>
</xs:schema>
```

### CSS 文件

这是 `practical.css`，用于从 XML 文件生成的 HTML 的样式表：

```php
@charset "utf-8";
body {
background-color:#FFDEAD;
margin-top:10px;
color: teal;
}
#overview h1
{
  text-align:center;
}
div.simage img
{
border:3px white solid;
align:center;
}
#mysite
{
margin:auto;
width:980px;
}
div.smallmat {

  width:300px;
  float: left;
}
div.scaption {
  padding:5px;
}
div.story {
  width:500px;
  float: left;
}

div.storybook {
  width:850px;
  float:left;
  margin:20px;
}
```

### PHP 文件

这是生成包含与 XML 中引用的照片数量一样多的照片的小图片画廊的 HTML 的 PHP 文件：

```php
<!DOCTYPE html>
<html >
<head><meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
<title>June Lake Gallery</title>
<link href="styles/practical.css" rel="stylesheet" type="text/css" media="screen"/>
</head>
<?php
$xmlfile = "practical.xml";
if (!file_exists($xmlfile)) {
   exit('Failed to open practical.xml.');
}
$xml =  simplexml_load_file($xmlfile);
?>
<body><div id="mysite"><div id="overview"><h1>
<?php
echo $xml->title;
?>
</h1><p>
<?php echo $xml->overview; ?>
</p></div><div id="gallery">
<?php
$photos = $xml->xpath("//photo");
$photocount = count($photos);
$count = 0;
while ($count < $photocount) {
$htmlstring = '<div class="storybook"><div class="smallmat"><div class="simage"><a href="';
$htmlstring .= $photos[$count]->largeimg;
$htmlstring .= '"><img class="sphoto" src="';
$htmlstring .= $photos[$count]->smallimg;
$htmlstring .= '" title="';
$htmlstring .=  $photos[$count]->caption;
$htmlstring .= '"></img></a></div><div class="scaption">';
$htmlstring .=  $photos[$count]->scaption;
$htmlstring .= '</div></div><div class="story">';
$htmlstring .=  $photos[$count]->story;
$htmlstring .=  '</div></div>';
echo $htmlstring;
$count++;   }
?>
</div></div></body></html>
```

如其名所示，SimpleXML 使用简单。`simplexml_load_file()` 函数返回一个包含传递给参数的文件整个 XML 树的对象。还有一个配套的方法，称为 `simplexml_load_string()`，它接受一个包含 XML 的字符串作为参数。这个树中的每个元素都可以使用简单的语法进行访问。

要访问所有照片节点集合，使用了 **xpath()** 方法。它支持 XPath 语法，这是一种用于 XML 文件的查询语言。你在这里需要记住的是 `//photo` 的含义。它返回所有照片节点，即使它们不在树的顶层。

### 使用 SimpleXML 创建 XML 文件

你也可以使用 SimpleXML 在程序内部创建 XML 文件。你从一个字符串形式的根元素开始，然后通过使用 `addChild()` 方法添加节点来构建你的 XML 文件，该方法接受一个或两个参数。第一个是节点的名称，可选的第二个参数是它的值。当你完成所有操作后，使用 `asXML()` 方法结束，它将返回包含所有 XML 的字符串，或者如果你提供了参数，它会将其存储到文件中。在下面的示例中，我们将其称为 `xmlfiles/new.xml`。

为了总结 SimpleXML 这一部分，这里有一个小程序，它重新创建我们的 XML 文件，但只包含一个照片节点，并将其保存为 `xmlfiles/new.xml`：

```php
<?php
$xml =  simplexml_load_string('<photocollection></photocollection>');
$xml->addChild("title", "June Lake");
$xml->addChild("overview", "The June Lake Loop begins just five miles south of Lee Vining, on US 395, ...");
$photo = $xml->addChild("photo");
$photo->addChild("scaption", "June Lake in the Fall");
$photo->addChild("caption", "June Lake and Carson Peak");
$photo->addChild("story", "Each time that unfortunate day  ...");
$photo->addChild("smallimg", "imagessmall/junelakefall.jpg");
$photo->addChild("largeimg", "imagespng/junelakeinthefall.png");
$xml->asXML("xmlfiles/new.xml");
?>
```

### 在客户端生成我们的 HTML

当我们使用 XML 或 JSON 作为数据格式并使用 Ajax 调用时不一定需要始终在服务器上生成最终的 HTML 并将其发送到客户端。随着越来越多的强大设备被用来访问网络并在其中运行浏览器，生成 HTML 代码也可以在客户端完成。

我们可以使用 `.post()` jQuery Ajax 调用来执行 PHP 程序，使用 SimpleXML 将整个 XML 文件作为字符串传递，然后进行处理。即使这部分也不是必需的，因为 jQuery Ajax 方法已经知道 XML 和 JSON 是什么。我们只需将 XML 文件的路径指定给 `.ajax()` 方法，jQuery 就会处理剩下的部分。这是一个非常适合使用 `.get()` 方法的例子，因为我们只需要从服务器检索数据，即我们的 XML，而不需要向其发送任何数据。

以下示例将生成完全相同的相册，但它全部是通过 Ajax 和 JavaScript 实现的，没有涉及任何 PHP 代码。HTML 文件如下：

```php
<!doctype html>
<html lang="en">
<head>
<meta charset="utf-8" />
<title>June Lake Gallery</title>
<link href="styles/practical.css" rel="stylesheet" type="text/css" media="screen"/>
</head>
<script src="img/jquery-1.10.1.js"></script>
<body>
<div id="mysite">
</div>
</body>
</html>
<script src="img/practical.js"></script>
```

JavaScript 文件如下：

```php
$(document).ready (function () {
$.get( "practical.xml", function( xml ) {
var jQueryxml = $(xml);  // Turn XML object into a jQuery object
var html = '<div id="overview"><h1>' + jQueryxml.find('title').text() + '</h1>';
html += '<p>' + jQueryxml.find('overview').text() + '</p></div>';
html += '<div id="gallery">';
jQueryxml.find('photo').each(function(){
html += '<div class="storybook"><div class="smallmat"><div class="simage"><a href="';
var scaption = $(this).find('scaption').text();
var caption = $(this).find('caption').text();
var largeimg = $(this).find('largeimg').text();
var smallimg = $(this).find('smallimg').text();
var story = $(this).find('story').text();
html += largeimg;
html += '"><img class="sphoto" src="';
html += smallimg;
html += '" title="';
html +=  caption;
html += '"></img></a></div><div class="scaption">';
html +=  scaption;
html += '</div></div><div class="story">';
html +=  story;
html +=  '</div></div>';
    }); //close each()
html += '</div>';
$('#mysite').html(html);
}, "xml");
});
```

## XSLT

稍等一下。JSON 就在眼前。只需再添加一种 XML 技术，就可以充实我们的工具库！

想象两家银行，它们像银行一样产生大量数据。两家银行都使用 XML，并且都制作了一个 XML Schema 文件，以确保 XML 文件的格式定义得非常严格。现在，想象一家银行收购了另一家，并继承了所有其他 XML 格式的数据：这正在酝酿一场灾难。不用担心，XSLT 来拯救！

**可扩展样式表语言转换**（**XSLT**）是一种可以将 XML 文件转换为另一个 XML 文件的编程语言。所需的是一个 XSLT 引擎和一个 XSL 文件，该文件描述了如何将一个文件转换成另一个文件。在转换过程中可以发生各种计算，因为它是一个完整的编程语言。我关于单一网络开发主题最厚的书是一本 XSLT 指南。

XSL 文件被称为样式表。它们不是 CSS 文件，但“样式表”这个术语一点也不错。如果我们将一个 XML 文件转换成 HTML 文件，那么我们实际上为 XML 文件创建了一个视图，这超出了仅使用 CSS 文件所能做到的。而不是为 `california.xml` 示例创建一个 XSL 文件，这里有一个将 `practical.xml` 文件转换为与 SimpleXML 示例完全相同的 HTML 的 XSL 样式表。如果你分析其内部内容，你可以将其分解为三个部分：哪些文件部分要应用规则（模板），规则是什么，以及所有这些规则中，纯 HTML。

这里是一个 XSLT 示例：

```php
<?xml version="1.0"?>
<xsl:stylesheet version="1.0"
  >
  <xsl:output method="html"/>
  <xsl:template match="/">
  <div id="overview">
       <xsl:apply-templates select="/photocollection"/>
  </div>
  <div id="gallery">
  <xsl:apply-templates select="/photocollection/photo"/>
   </div>
  </xsl:template>
  <xsl:template match="photocollection">
  <h1>
  <xsl:value-of select="title"/>
  </h1>
  <p>
  <xsl:value-of select="overview"/>
  </p>
  </xsl:template>
  <xsl:template match="photo">
  <div class="storybook">
  <div class="smallmat">
    <div class="simage">
    <a>
    <xsl:attribute name="href">
    <xsl:value-of select="largeimg"/>
    </xsl:attribute>
    <img class="sphoto">
    <xsl:attribute name="src">
    <xsl:value-of select="smallimg"/>
    </xsl:attribute>
     <xsl:attribute name="title">
    <xsl:value-of select="caption"/>
    </xsl:attribute>
    </img>
    </a>
    </div>
    <div class="scaption">
    <xsl:value-of select="scaption"/>
    </div>
    </div>
    <div class="story">
    <xsl:value-of select="story"/>
    </div>
    </div>
  </xsl:template>
</xsl:stylesheet>
```

这是处理转换的 PHP 文件：

```php
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8" />
<title>June Lake Gallery</title>
<link href="styles/practical.css" rel="stylesheet" type="text/css" media="screen"/>
</head>
<body>
<div id="mysite">
<?php
$xslt = new XSLTProcessor;
$xmlfile = "practical.xml";
$xslfile = "practical.xsl";
if (!file_exists($xmlfile)) {

    exit('Failed to open practical.xml.');
}
$xml = new DOMdocument;
$xml->load($xmlfile);
$xsl = new DOMdocument;
$xsl->load($xslfile);
$xslt->importStyleSheet($xsl);
printf("%s",$xslt->transformToXML($xml));
?>
</div>
</body>
</html>
```

# JSON

**JavaScript 对象表示法**（**JSON**）是本章讨论的另一种数据交换格式。像 XML 一样，它是基于文本的，可读性强，但它比其对应物更轻量。以 JSON 格式发送的数据在网络连接中传输时将占用更少的带宽。它在当今的 Web 应用程序中越来越受欢迎。XML 文件更重——只需注意我们提供了多少页的例子，并且需要某种类型的解析器来处理数据。

JSON 是从 JavaScript 衍生出来的，JSON 代码看起来很像 JavaScript 对象，但也有一些细微的差别。然而，JavaScript 可以用来处理 JSON 数据，所以你不需要一个单独的解析器，就像处理 XML 那样。这里是从 `california.xml` 示例中相同的资料，但以 JSON 格式呈现：

```php
[
   {
    "name":"Adams",
     "first":"Ansel",
     "profession":"photographer",
     "born":"San Francisco"
  },
  {
    "name":"Muir",
     "first":"John",
     "profession":"naturalist",
     "born":"Scotland"
  },
  {
    "name":"Schwarzenegger",
     "first":"Arnold",
     "profession":"governator",
     "born":"Germany"

  },
  {
    "name":"Rowell",
     "first":"Galen",
     "profession":"photographer",
     "born":"Oakland CA"
  },
  {
    "name":"Wellens",
     "first":"Paul",
     "profession":"author",
     "born":"Belgium"

  }
]
```

首先要注意的是，与它的 XML 对应物相比，这要容易阅读得多。它看起来像是一个包含键：值对的 JavaScript 对象数组，这几乎就是它所代表的。在 第四章 中关于对象的讨论中，我们有一个包含 John Williams 名字的例子，以及对 JSON 格式的提示。关键的区别（这里的 *key* 是一个关键词）是，在 JSON 中，键也必须被双引号包围。所以，在 JSON 的领域中，`name:"Williams"` 是不正确的。让我们回顾一下 JSON 数据格式的简单结构。你可能想要拿一张纸，为每条规则画一个小图。

## JSON 语法

JSON 数据是纯文本。JSON 文本由一系列字符组成，这些字符以 Unicode 编码，并符合 **JSON 值语法**。所以简单地说，每条 JSON 数据都必须是一个有效的 JSON 值。在 JSON 中有六个具有特殊意义的令牌：`[`、`{`、`]`、`}`、`:` 和 `,`。还有三个具有特殊意义的名称令牌：`true`、`false` 和 `null`。这些令牌用于组成正确的 JSON 值。

### JSON 值

一个 JSON **值** 可以是一个对象、数组、数字或字符串，或者是 true、false 或 null 之一。这些都是格式良好的 JSON 数据的构建块。

### JSON 对象

一个 JSON **对象** 以左花括号开始，以右花括号结束。在其内部，将有零个或多个由逗号分隔的键：值对。键或名称是一个 JSON 字符串。键和值之间有一个分号。值本身可以是任何有效的 JSON 值。

### JSON 字符串

一个 JSON **字符串** 以双引号开始和结束。不能使用单引号来界定字符串。然而，字符串中间可以有一个单引号。如果需要双引号，你必须用反斜杠（`\`）转义它。其他两个字符的转义序列不可避免地是 `\\`、`\/`、`\b`、`\f`、`\n`、`\r` 和 `\t`。你也可以使用任何字符的 4 位 Unicode 码，只需在该字符前加上 `\u`。

### JSON 数组

JSON 的 **数组** 是一对方括号，包围着零个或多个值，这些值由逗号分隔。之前看到的例子实际上是一个包含四个 JSON 对象的 JSON 数组，每个对象又包含四个键值对。

### JSON 数字

JSON 的 **数字** 总是十进制，没有前导零。前面可以有一个负号和一个作为浮点数的 `.`。它可以有一个十的指数，前面有 `e` 或 `E`。

## JSON 和 PHP

如果你使用 PHP 在服务器端，例如，从 MySQL 数据库中检索数据，并且你想将数据转换为 JSON 格式，这可以很容易地完成。只需创建一个包含数据的 PHP 数组。有一个非常方便的函数 `json_encode()`，它接受一个 PHP 数组并将其转换为包含 JSON 数据的字符串（注意我没有写 JSON 字符串）。还有一个函数执行相反的操作，将 JSON 数据转换为 PHP 数组：`json_decode()`。

这里有一些示例 PHP 代码，它将 PHP 数组转换为包含与我们的 `californiapeople` 示例中相同的 JSON 数组的 PHP 字符串。如果这是从 jQuery Ajax 调用访问的 `.php` 文件的一部分，你可以在客户端将其作为 JSON 处理。只需确保指定 `json` 作为数据类型：

```php
$californiapeople =  array  (
     array
     (
     "name"=>"Adams",
     "first"=>"Ansel",
    "profession" => "photographer",
    "born" => "San Francisco"),
     array
    (
     "name"=>"Muir",
     "first"=>"John",
     "profession" => "naturalist",
    "born" => "Scotland"),
    array
    (
     "name"=>"Schwarzenegger",
     "first"=>"Arnold",
     "profession" => "governator",
     "born" => "Germany"),
    array
    (
     "name"=>"Wellens",
     "first"=>"Paul",
     "profession" => "author",
    "born" => "Belgium")
  );
echo   json_encode($californiapeople);
```

## 使用 Ajax 和 jQuery 的 JSON

为了结束这一章，我们将重新创建我们之前用来演示如何在客户端动态生成 HTML 的照片库示例。这次，我们将使用 JSON 作为数据格式。让我们从我们的更简单、更精简的数据开始，这些数据将驻留在服务器上的 `practical.json` 文件中。请注意，它包含一个具有单个键值对的 JSON 对象，其值是另一个具有四个键值对的对象，其中最后一个是一个数组。

再次强调，这比它的 XML 对应物更容易阅读：

```php
{
  "photocollection": {
    "title": "June Lake",
    "overview": "The June Lake Loop begins  ...",
    "photo": [
      {
        "scaption": "June Lake in the Fall",
        "caption": "June Lake and Carson Peak in the fall",
        "story": "Each time that unfortunate day  ...",
        "smallimg": "imagessmall/junelakefall.jpg",
        "largeimg": "imagespng/junelakeinthefall.png"
      },
      {
        "scaption": "Aspen by Silver Lake",
        "caption": "Aspen trees by Silver Lake",
        "story": "In 1998, I hit the right week of the ...",
        "smallimg": "imagessmall/silverlakeaspenfall98.jpg",
        "largeimg": "imagespng/silverlakeaspenfall98.png"
      },
      {
        "scaption": "Gull Lake in the Fall",
        "caption": "Gull Lake in the Fall - Happy fishermen !",
        "story": "If you take the north shore road around ...",
        "smallimg": "imagessmall/gulllake648.jpg",
        "largeimg": "imageslarge/gulllake648.jpg"
      },
      {
        "scaption": "Silver Lake",
        "caption": "Silver Lake - June Lake Loop",
        "story": "Any time of the year, there ...",
        "smallimg": "imagessmall/silver2.jpg",
        "largeimg": "imagespng/silver2.png"
      }
    ]
  }
}
```

这个 HTML 文件与 XML 示例相同：我们只需要更改 JavaScript 文件中的内容。这里我们将使用一个方便的 jQuery 函数 `.getJSON()`，它是 `.ajax()` 的另一个快捷方式。它返回一个 JSON 对象，我们可以用常规 JavaScript 进一步处理，从而使代码本身更简单：

```php
$(document).ready (function () {
$.getJSON( "practical.json", function( json ) {
// For debugging:
//var jsonString = JSON.stringify(json)
//alert(jsonString);
var html = '<div id="overview"><h1>' + json.photocollection.title + '</h1>';
html += '<p>' + json.photocollection.overview + '</p></div>';
html += '<div id="gallery">';
// Use jQuery .each() function to loop through array
$(json.photocollection.photo).each(function()
{
html += '<div class="storybook"><div class="smallmat"><div class="simage"><a href="';
var scaption = this.scaption;
var caption = this.caption;
var largeimg = this.largeimg;
var smallimg = this.smallimg;
var story = this.story;
html += largeimg;
html += '"><img class="sphoto" src="';
html += smallimg;
html += '" title="';
html +=  caption;
html += '"></img></a></div><div class="scaption">';
html +=  scaption;
html += '</div></div><div class="story">';
html +=  story;
html +=  '</div></div>';
}
);
html += '</div>';
$('#mysite').html(html);
});
});
```

而不是使用 jQuery 的 `.each()` 函数，我们也可以使用普通的 JavaScript `for` 循环：

```php
for (var i in json.photocollection.photo)
{
. . .
var scaption = json.photocollection.photo[i].scaption;
. . .
}
```

## 两个有用的 JSON 方法

总结来说，我们将提到两个属于 JSON 对象的 JavaScript 方法，并且大多数浏览器都支持这些方法。

将 JSON 字符串转换为 JavaScript 对象，你只需将 JSON 字符串传递给 `JSON.parse()` 方法，如下所示：

```php
var jsObject = JSON.parse(jsonString);
```

`JSON.stringify()` 可以用来执行相反的操作，将 JavaScript 对象转换为 JSON 字符串。在我们的例子中，我们可以使用以下代码，这对于调试很有用：

```php
var jsonString = JSON.stringify(json)
alert(jsonString);
```

# 摘要

在本章中，我们介绍了两种在网页开发中使用的数据格式。XML 在整个行业中用于交换人类和计算机都能读取的数据格式。我们可以将 XML 文件用作小型数据库，并且存在一些基于 XML 的技术来帮助我们，例如 XML Schema，它可以用来严格定义 XML 文件的结构，以及 XSLT，它可以创建样式表将 XML 文件转换为另一个 XML 文件，这可能是网页。使用 SimpleXML，我们还可以在 PHP 中处理和创建 XML 文件。

JSON 是当前网页开发者中流行的更简洁、更高效的格式。它所需的带宽比 XML 少，当数据量增加时这一点极为有益。由于 JSON 数据可以像 JavaScript 对象一样处理，所以在使用 JSON 时不需要额外的处理代码或引擎。

现在，如果我们的数据也能以 JSON 格式存储在服务器上，那么就无需进行这些从一种格式到另一种格式的转换，这不是很好吗？是的，这样很好。我们能这样做吗？是的，我们可以。只需翻到下一章。
