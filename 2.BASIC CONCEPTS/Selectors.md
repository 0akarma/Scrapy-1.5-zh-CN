#选择器
在抓取网页时，需要执行的最常见任务是从HTML源中提取数据。有几个库可以实现这一点：

- `BeautifulSoup`是Python程序员中非常流行的网页抓取库，它基于HTML代码的结构构建了一个Python对象，同时也很好地处理了糟糕的标记，但它有一个缺点：速度很慢。
- `lxml`是一个基于`ElementTree`的pythonic API的XML解析库（它也解析HTML）。（lxml不是Python标准库的一部分。）

Scrapy带有自己的提取数据的机制。它们被称为选择器，因为它们“选择”由`XPath`或`CSS`表达式指定的HTML文档的某些部分。

`XPath`是一种用于选择XML文档中的节点的语言，它也可以用于HTML。`CSS`是一种将样式应用于HTML文档的语言。它定义选择器将这些样式与特定的HTML元素相关联。

Scrapy选择器是在`lxml`库上构建的，这意味着它们在速度和分析准确性方面非常相似。

本页解释了选择器如何工作和描述其非常小且简单的API ，不像更大的`lxml` API，因为`lxml`库除了选择标记文档之外还可以用于许多其他任务。

有关选择器API的完整参考，请参阅[选择器参考]()
##选择器的使用
###构建选择器
Scrapy选择器是`Selector`通过传递文本或`TextResponse`对象构造的类的实例。它会根据输入类型自动选择最佳的解析规则（XML vs HTML）：

	>>> from scrapy.selector import Selector
	>>> from scrapy.http import HtmlResponse
从文本构建：

	>>> body = '<html><body><span>good</span></body></html>'
	>>>> Selector(text=body).xpath('//span/text()').extract()
	[u'good']
从响应构建：

	>>> response = HtmlResponse(url='http://example.com', body=body)
	>>>> Selector(response=response).xpath('//span/text()').extract()
	[u'good']
为方便起见，响应对象在.selector属性上公开选择器，在可能的情况下完全可以使用此快捷方式：

	>>> response.selector.xpath('//span/text()').extract()
	[u'good']
###使用选择器
为了解释如何使用选择器，我们将使用Scrapy shell（提供交互式测试）以及位于Scrapy文档服务器中的示例页面：
[https://doc.scrapy.org/en/latest/_static/selectors-sample1.html](https://doc.scrapy.org/en/latest/_static/selectors-sample1.html)

这里是它的HTML代码：
	
	<html>
	 <head>
	  <base href='http://example.com/' />
	  <title>Example website</title>
	 </head>
	 <body>
	  <div id='images'>
	   <a href='image1.html'>Name: My image 1 <br /><img src='image1_thumb.jpg' /></a>
	   <a href='image2.html'>Name: My image 2 <br /><img src='image2_thumb.jpg' /></a>
	   <a href='image3.html'>Name: My image 3 <br /><img src='image3_thumb.jpg' /></a>
	   <a href='image4.html'>Name: My image 4 <br /><img src='image4_thumb.jpg' /></a>
	   <a href='image5.html'>Name: My image 5 <br /><img src='image5_thumb.jpg' /></a>
	  </div>
	 </body>
	</html>
首先，我们打开shell：

`scrapy shell https://doc.scrapy.org/zh/latest/_static/selectors-sample1.html`

然后，在加载shell之后，您将有可用的响应作为`response` shell变量，并将其附加的选择器作为`response.selector`属性。

由于我们正在处理HTML，选择器将自动使用HTML解析器。

因此，通过查看该页面的HTML代码，我们构建一个用于选择标题标签内的文本的XPath：

	>>> response.selector.xpath('//title/text()')
	[<Selector (text) xpath=//title/text()>]
使用XPath和CSS查询响应非常常见，以至于响应包含两个便捷快捷键：`response.xpath()`和`response.css()`：

	>>> response.xpath('//title/text()')
	[<Selector (text) xpath=//title/text()>]
	>>> response.css('title::text')
	[<Selector (text) xpath=//title/text()>]
正如你所看到的，`.xpath()`和`.css()`方法返回一个`SelectorList`实例，这是新的选择列表。该API可用于快速选择嵌套数据：

	>>> response.css('img').xpath('@src').extract()
	[u'image1_thumb.jpg', u'image2_thumb.jpg', u'image3_thumb.jpg', u'image4_thumb.jpg', u'image5_thumb.jpg']
要实际提取文本数据，您必须调用选择器`.extract() `方法，如下所示：

	>>> response.xpath('//title/text()').extract()
	[u'Example website']
如果你只想提取第一个匹配的元素，你可以调用选择器`.extract_first()`

	>>> response.xpath('//div[@id="images"]/a/text()').extract_first()
	u'Name: My image 1 '
如果没有找到元素，它会返回None：
	
	>>> response.xpath('//div[@id="not-exists"]/text()').extract_first() is NoneTrue
默认返回值可以作为参数提供，用来代替None：
	
	>>> response.xpath('//div[@id="not-exists"]/text()').extract_first(default='not-found')
	'not-found'
请注意，CSS选择器可以使用CSS3伪元素选择文本或属性节点：

	>>> response.css('title::text').extract()
	[u'Example website']
现在我们要获取基本URL和一些图像链接：

	>>> response.xpath('//base/@href').extract()
	[u'http://example.com/']
	>>> response.css('base::attr(href)').extract()
	[u'http://example.com/']
	>>> response.xpath('//a[contains(@href, "image")]/@href').extract()
	[u'image1.html', u'image2.html', u'image3.html', u'image4.html', u'image5.html']
	>>> response.css('a[href*=image]::attr(href)').extract()
	[u'image1.html', u'image2.html', u'image3.html', u'image4.html', u'image5.html']
	>>> response.xpath('//a[contains(@href, "image")]/img/@src').extract()
	[u'image1_thumb.jpg', u'image2_thumb.jpg', u'image3_thumb.jpg', u'image4_thumb.jpg', u'image5_thumb.jpg']
	>>> response.css('a[href*=image] img::attr(src)').extract()
	[u'image1_thumb.jpg', u'image2_thumb.jpg', u'image3_thumb.jpg', u'image4_thumb.jpg', u'image5_thumb.jpg']
###嵌套选择器
选择方法（`.xpath()`或`.css()`）返回相同类型的选择器列表，因此您也可以调用这些选择器的选择方法。这是一个例子：

	>>> links = response.xpath('//a[contains(@href, "image")]')>>> links.extract()
	[u'<a href="image1.html">Name: My image 1 <br><img src="image1_thumb.jpg"></a>', 
	u'<a href="image2.html">Name: My image 2 <br><img src="image2_thumb.jpg"></a>', 
	u'<a href="image3.html">Name: My image 3 <br><img src="image3_thumb.jpg"></a>', 
	u'<a href="image4.html">Name: My image 4 <br><img src="image4_thumb.jpg"></a>', 
	u'<a href="image5.html">Name: My image 5 <br><img src="image5_thumb.jpg"></a>']
	>>> for index, link in enumerate(links):
	...     args = (index, link.xpath('@href').extract(), link.xpath('img/@src').extract())
	...     print 'Link number %d points to url %s and image %s' % args
	Link number 0 points to url [u'image1.html'] and image [u'image1_thumb.jpg']
	Link number 1 points to url [u'image2.html'] and image [u'image2_thumb.jpg']
	Link number 2 points to url [u'image3.html'] and image [u'image3_thumb.jpg']
	Link number 3 points to url [u'image4.html'] and image [u'image4_thumb.jpg']
	Link number 4 points to url [u'image5.html'] and image [u'image5_thumb.jpg']
###使用正则表达式的选择器
`Selector`也有`.re()`使用正则表达式提取数据的方法。但是，与使用`.xpath()`或`.css()`方法不同，`.re()`返回一个unicode字符串列表。所以你不能构建嵌套`.re()`调用。

以下是一个用于从上面的HTML代码中提取图像名称的示例：

	>>> response.xpath('//a[contains(@href, "image")]/text()').re(r'Name:\s*(.*)')
	[u'My image 1', u'My image 2', u'My image 3', u'My image 4', u'My image 5']
###使用相对的XPath
请记住，如果您嵌套选择器并使用以XPath开头的XPath `/`，那么XPath对于文档而言是绝对的，而不是相对于 Selector您从中调用它。

例如，假设你想提取`<p>`元素内的所有`<div>` 元素。首先，你会得到所有`<div>`元素：

	>>> divs = response.xpath('//div')
起初，您可能会使用以下方法，这是错误的，因为它实际上会`<p>`从文档中提取所有元素，而不仅仅是`<div>`元素中的所有元素：

	>>> for p in divs.xpath('//p'):  # this is wrong - gets all <p> from the whole document
	...     print p.extract()
这是做到这一点的正确方法（注意前缀的点：`.//p`XPath ）：
	
	>>> for  p  in  divs 。xpath （'.//p' ）：  ＃提取所有<p>内部
	...     print  p 。提取物（）
另一个常见的情况是提取所有`<p>`的子节点：

	>>> for p in divs.xpath('.//p'):  # extracts all <p> inside
	...     print p.extract()
有关相关XPath的更多详细信息，请参阅XPath规范中的[位置路径]()部分。
###XPath表达式中的变量
XPath允许使用`$somevariable`语法在XPath表达式中引用变量。这与SQL世界中的参数化查询或预准备语句有些类似，您可以使用占位符替换查询中的某些参数，`?`然后用占位符传递的值替换。

下面是一个基于其“id”属性值匹配元素的示例，不用对其进行硬编码（以前显示过）：

	>>> # `$val` used in the expression, a `val` argument needs to be passed>>> response.xpath('//div[@id=$val]/a/text()', val='images').extract_first()
	u'Name: My image 1 '
下面是另一个例子，为了找到`<div>`包含五个`<a>`子的标签的“id”属性（这里我们将该值`5`作为整数传递）：
	
	>>> response.xpath('//div[count(a)=$cnt]/@id', cnt=5).extract_first()
	u'images'
在引用`.xpath()` 时，所有变量调用时都必须有一个绑定值（否则你会得到一个`ValueError: XPath error:`异常）。这需要通过传递许多命名参数来完成。
`parsel`，为Scrapy选择器提供动力的库，有更多关于XPath变量的细节和例子。
##使用EXSLT扩展
构建在`lxml`之上，Scrapy选择器还支持一些`EXSLT`扩展，并附带这些预先注册的名称空间以用于XPath表达式：


| 前缀 | 命名空间|  用法  |
| :--------| :-----   | :----: |
| re |http://exslt.org/regular-expressions|regular expressions|
| set|http://exslt.org/sets|set manipulation|
###常用表达
`test()`方法：例如，当XPath `starts-with()`或者`contains()`不够用时，这个函数是非常有用的 。
使用以数字结尾的“class”属性选择列表项中链接的示例：

	>>> from scrapy import Selector>>> doc = """
	... <div>
	...     <ul>
	...         <li class="item-0"><a href="link1.html">first item</a></li>
	...         <li class="item-1"><a href="link2.html">second item</a></li>
	...         <li class="item-inactive"><a href="link3.html">third item</a></li>
	...         <li class="item-1"><a href="link4.html">fourth item</a></li>
	...         <li class="item-0"><a href="link5.html">fifth item</a></li>
	...     </ul>
	... </div>
	... """
	>>> sel = Selector(text=doc, type="html")
	>>> sel.xpath('//li//@href').extract()[u'link1.html', u'link2.html', u'link3.html', u'link4.html', u'link5.html']
	>>> sel.xpath('//li[re:test(@class, "item-\d$")]//@href').extract()[u'link1.html', u'link2.html', u'link4.html', u'link5.html']
	>>>
>警告
C库libxslt本身不支持EXSLT正则表达式，所以lxml的实现在Python re模块中使用钩子。因此，在XPath表达式中使用正则表达式函数可能会增加一点性能损失。
###设置操作
例如，在提取文本元素之前，这些可以方便地排除文档树的部分内容。

使用itemscopes组和相应的itemprops 提取微数据（从`http://schema.org/Product`获取样本内容）的示例：

	>>> doc = """
	... <div itemscope itemtype="http://schema.org/Product">
	...   <span itemprop="name">Kenmore White 17" Microwave</span>
	...   <img src="kenmore-microwave-17in.jpg" alt='Kenmore 17" Microwave' />
	...   <div itemprop="aggregateRating"
	...     itemscope itemtype="http://schema.org/AggregateRating">
	...    Rated <span itemprop="ratingValue">3.5</span>/5
	...    based on <span itemprop="reviewCount">11</span> customer reviews
	...   </div>
	...
	...   <div itemprop="offers" itemscope itemtype="http://schema.org/Offer">
	...     <span itemprop="price">$55.00</span>
	...     <link itemprop="availability" href="http://schema.org/InStock" />In stock
	...   </div>
	...
	...   Product description:
	...   <span itemprop="description">0.7 cubic feet countertop microwave.
	...   Has six preset cooking categories and convenience features like
	...   Add-A-Minute and Child Lock.</span>
	...
	...   Customer reviews:
	...
	...   <div itemprop="review" itemscope itemtype="http://schema.org/Review">
	...     <span itemprop="name">Not a happy camper</span> -
	...     by <span itemprop="author">Ellie</span>,
	...     <meta itemprop="datePublished" content="2011-04-01">April 1, 2011
	...     <div itemprop="reviewRating" itemscope itemtype="http://schema.org/Rating">
	...       <meta itemprop="worstRating" content = "1">
	...       <span itemprop="ratingValue">1</span>/
	...       <span itemprop="bestRating">5</span>stars
	...     </div>
	...     <span itemprop="description">The lamp burned out and now I have to replace
	...     it. </span>
	...   </div>
	...
	...   <div itemprop="review" itemscope itemtype="http://schema.org/Review">
	...     <span itemprop="name">Value purchase</span> -
	...     by <span itemprop="author">Lucas</span>,
	...     <meta itemprop="datePublished" content="2011-03-25">March 25, 2011
	...     <div itemprop="reviewRating" itemscope itemtype="http://schema.org/Rating">
	...       <meta itemprop="worstRating" content = "1"/>
	...       <span itemprop="ratingValue">4</span>/
	...       <span itemprop="bestRating">5</span>stars
	...     </div>
	...     <span itemprop="description">Great microwave for the price. It is small and
	...     fits in my apartment.</span>
	...   </div>
	...   
	...
	... </div>
	... """
	>>> sel = Selector(text=doc, type="html")>>> for scope in sel.xpath('//div[@itemscope]'):
	...     print "current scope:", scope.xpath('@itemtype').extract()
	...     props = scope.xpath('''
	...                 set:difference(./descendant::*/@itemprop,...                                .//*[@itemscope]/*/@itemprop)''')
	...     print "    properties:", props.extract()
	...     print
	current scope: [u'http://schema.org/Product']    properties: [u'name', u'aggregateRating', u'offers', u'description', u'review', u'review']
	current scope: [u'http://schema.org/AggregateRating']    properties: [u'ratingValue', u'reviewCount']
	current scope: [u'http://schema.org/Offer']    properties: [u'price', u'availability']
	current scope: [u'http://schema.org/Review']    properties: [u'name', u'author', u'datePublished', u'reviewRating', u'description']
	current scope: [u'http://schema.org/Rating']    properties: [u'worstRating', u'ratingValue', u'bestRating']
	current scope: [u'http://schema.org/Review']    properties: [u'name', u'author', u'datePublished', u'reviewRating', u'description']
	current scope: [u'http://schema.org/Rating']    properties: [u'worstRating', u'ratingValue', u'bestRating']
	>>>
在这里，我们首先迭代`itemscope`元素，并为每个`itemprops`元素寻找所有元素并排除那些在另一个元素中的元素`itemscope`。
##一些XPath技巧
以下是一些技巧，您可能会发现在使用Scrapy选择器的XPath时有用，基于`ScrapingHub博客的这篇文章`。如果您还不太熟悉XPath，那么您可能需要先看看这个`XPath`教程。
###在条件中使用文本节点
当您需要使用文本内容作为`XPath字符串函数的参数`时，请避免使用`.//text()`和使用`.`。
这是因为表达式`.//text()`产生了一组文本元素 - 一个节点集。当一个节点集被转换为一个字符串时，当它作为参数传递给一个字符串函数如`contains() `or `starts-with()`时，会产生第一个元素的文本。

例：

	>>> from scrapy import Selector
	>>> sel = Selector(text='<a href="#">Click here to go to the <strong>Next Page</strong></a>')
将节点集转换为字符串：

	>>> sel.xpath('//a//text()').extract() # take a peek at the node-set[u'Click here to go to the ', u'Next Page']
	>>> sel.xpath("string(//a[1]//text())").extract() # convert it to string[u'Click here to go to the ']
一个节点转换为字符串时会将其本身及其后代的文本拼接在一起:

	>>> sel.xpath("//a[1]").extract() # select the first node
	[u'<a href="#">Click here to go to the <strong>Next Page</strong></a>']
	>>> sel.xpath("string(//a[1])").extract() # convert it to string
	[u'Click here to go to the Next Page']
因此，`.//text()`在这种情况下使用节点集将不会选择任何内容：

	>>> sel.xpath("//a[contains(.//text(), 'Next Page')]").extract()
	[]
但使用`.`意味着节点，工作：

	>>> sel.xpath("//a[contains(., 'Next Page')]").extract()[u'<a href="#">Click here to go to the <strong>Next Page</strong></a>']
###当心 //node[1] and (//node)[1]之间的区别
`//node[1]` 选择在其各自父母下首先发生的所有节点。
`(//node)[1]` 选择文档中的所有节点，然后只获取其中的第一个节点。

例：

	>>> from scrapy import Selector>>> sel = Selector(text="""
	....:     <ul class="list">
	....:         <li>1</li>
	....:         <li>2</li>
	....:         <li>3</li>
	....:     </ul>
	....:     <ul class="list">
	....:         <li>4</li>
	....:         <li>5</li>
	....:         <li>6</li>
	....:     </ul>""")
	>>> xp = lambda x: sel.xpath(x).extract()
无论它是否是它的父级，它都会获得所有的第一个`<li> `元素：

	>>> xp("//li[1]")
	[u'<li>1</li>', u'<li>4</li>']
这就得到了整个文档的第一个`<li>`元素

	>>> xp("(//li)[1]")
	[u'<li>1</li>']
这将获得所有父项<ul>下的第一个`<li> `元素：

	>>> xp("//ul/li[1]")
	[u'<li>1</li>', u'<li>4</li>']
这将获得整个文档中父项`<ul>`下的第一个`<li>`元素：

	>>> xp("(//ul/li)[1]")
	[u'<li>1</li>']

###按类查询时，请考虑使用CSS
由于一个元素可以包含多个CSS类，因此按类选择元素的XPath方法相当冗长：

	*[contains(concat(' ', normalize-space(@class), ' '), ' someclass ')]
如果你使用了，`@class='someclass'`你最终可能会丢失具有其他类的元素，如果你只是用`contains(@class, 'someclass')someclass`弥补这一点，你可能会得到更多的元素，如果他们有不同的类名则共享字符串someclass.

事实证明，Scrapy选择器允许您链接选择器，所以大多数情况下，您可以使用CSS按类选择，然后在需要时切换到XPath：

	>>> from scrapy import Selector
	>>> sel = Selector(text='<div class="hero shout"><time datetime="2014-07-23 19:00">Special date</time></div>')
	>>> sel.css('.shout').xpath('./time/@datetime').extract()
	[u'2014-07-23 19:00']
这比使用上面显示的详细XPath技巧更清晰。只要记住 . 要在XPath表达式后使用。
##内置选择器参考
###选择器对象
>classscrapy.selector.Selector（response = None，text = None，type = None ）

`Selector`的一个实例是一个包装器，用于选择其内容的某些部分。

`response`是将用于选择和提取数据的一个`HtmlResponse`对象或一个 `XmlResponse`对象。

当`response`不可用时，`text`是一个unicode字符串或utf-8编码文本。一起使用`tex`t和`response`，是一种未定义的行为。

`type`定义选择器类型，它可以是`"html"`，`"xml"`或`None`（默认）。

如果`type`是`None`，选择器将根据`response`类型自动选择最佳类型（请参见下文），或者默认为`"html"`与`text`一起使用。

如果`type` 是 `None`并且传递了一个`response`，则从响应类型推断选择器类型如下:

- "html"为`HtmlResponse`类型
- "xml"为`XmlResponse`类型
- "html" 为其他任何类型
否则，如果`type`被设置，选择器类型将被强制定义并且不会进行检测。
>xpath（query）

查找与xpath匹配的节点`query`，并将结果作为SelectorList实例返回， 并将所有元素展平。列表元素也实现了Selector接口。

`query` 是一个包含要应用的XPATH查询的字符串。
>注意
>
>为了方便，这个方法可以被称为 response.xpath()

**css（query）**

应用给定的CSS选择器并返回一个`SelectorList`实例。

`query` 是一个包含要应用的CSS选择器的字符串。
在后台，使用cssselect库和run .xpath()方法将CSS查询转换为XPath查询 。
>注意
>
>为了方便起见，这个方法可以称为 response.css()

**extract（）**

序列化并返回匹配的节点作为unicode字符串列表。百分比编码的内容是没有引用的。

**re（regex）**

应用给定的正则表达式并返回包含匹配的unicode字符串列表。

`regex` 可以是已编译的正则表达式，也可以是将使用正则表达式编译为正则表达式的字符串` re.compile(regex)`
>注意
>
>请注意，re()并且re_first()都解码HTML实体（除了&lt;和&amp;）。

**register_namespace（prefix，uri ）**

注册在此使用的给定名称空间`Selector`。如果不注册名称空间，则无法从非标准名称空间中选择或提取数据。看下面的例子。

**remove_namespaces（）**

删除所有名称空间，允许使用不含名称空间的xpaths来遍历文档。见下面的例子。

**__nonzero__（）**

返回`True`是否有任何选定的真实内容或`False` 其他。换句话说，a的布尔值Selector由它选择的内容给出。
###SelectorList对象
>Class scrapy.selector.SelectorList

这个`SelectorList`类是内建`list` 类的一个子类，它提供了一些额外的方法。
>xpath（query）

调用`.xpath()`此列表中每个元素的方法并将其结果展平为另一个`SelectorList`。

`query` 与`Selector.xpath()`参数是一样的 `Selector.xpath()`
>css（query）

调用`.css()`此列表中每个元素的方法并将其结果展平为另一个`SelectorList`。

`query` 与在`Selector.css()`中的参数是一样的
>extract（）

调用`.extract()`此列表中每个元素的方法并将其结果展平，作为unicode字符串列表。
>re（）

调用`.re()`此列表中每个元素的方法并将其结果展平，作为unicode字符串列表。
###HTML响应的选择器示例
这里有几个`Selector`例子来说明几个概念。在所有情况下，我们都假设已经有一个有着`HtmlResponse`对象的`Selector`实例：

	sel = Selector(html_response)

1.从HTML响应主体中选择所有`<h1>`元素，返回`Selector`对象列表 （即`SelectorList`对象）：

	sel.xpath("//h1")
2.从HTML响应主体中提取所有`<h1>`元素的文本，返回一个unicode字符串列表：

	sel.xpath("//h1").extract()         # this includes the h1 tag
	sel.xpath("//h1/text()").extract()  # this excludes the h1 tag
3.遍历所有`<p>`标签并打印它们的类属性：

	for node in sel.xpath("//p"):
	    print node.xpath("@class").extract()
###XML响应的选择器示例
这里有几个例子来说明几个概念。在这两种情况下，我们都假设已经`Selector`有一个`XmlResponse`像这样的对象实例化了 ：

	sel = Selector(xml_response)
1.从XML响应主体中选择所有`<product>`元素，返回`Selector`对象列表（即`SelectorList`对象）：

	sel.xpath("//product")
2.从需要注册名称空间的`Google Base XML Feed`中提取所有值：

	sel.register_namespace("g", "http://base.google.com/ns/1.0")
	sel.xpath("//g:price").extract()
###删除命名空间
在处理爬虫项目时，通常完全摆脱名称空间并仅使用元素名称来编写更简单/方便的XPath是非常方便的。你可以使用 `Selector.remove_namespaces()`方法。

我们来看一个用GitHub博客atom feed来说明的例子。

首先，我们用我们想要刮取的url打开shell：

	$ scrapy shell https://github.com/blog.atom
一旦进入shell，我们可以尝试选择所有<link>对象并查看它不起作用（因为Atom XML名称空间正在混淆这些节点）：

	>>> response.xpath("//link")
	[]
但是一旦我们调用这个`Selector.remove_namespaces()`方法，所有的节点都可以直接用它们的名字来访问：
	
	>>> response.selector.remove_namespaces()
	>>> response.xpath("//link")
	[<Selector xpath='//link' data=u'<link xmlns="http://www.w3.org/2005/Atom'>, <Selector xpath='//link' data=u'<link xmlns="http://www.w3.org/2005/Atom'>, 
	...
如果您想知道为什么命名空间删除过程并不总是被默认调用，而不必手动调用它，这是因为两个原因，按照相关性顺序，这两个原因是：

1.删除名称空间需要迭代和修改文档中的所有节点，这对于Scrapy搜索的所有文档执行操作来说是相当昂贵的操作

2.在某些情况下，实际上需要使用名称空间，以防某些元素名称在名称空间之间发生冲突。这些案例虽然非常罕见。