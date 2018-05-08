#蜘蛛
蜘蛛是一个定义怎样爬取一个特定站点（或一组站点）的类，包括如何执行抓取(即跟踪链接)以及如何从它们的页面中提取结构化数据(即抓取项目)。换句话说，在这里，您可以为特定的站点(或者一组站点)定义爬行和解析页面的自定义行为。

对于蜘蛛来说，抓取周期是这样的:

1.首先生成初始请求以抓取第一个url，并指定一个回调函数，由这些请求下载的响应来调用。

执行的第一个请求是通过调用`start_request()`方法获得的，该方法(默认情况下)为`start_urls`中指定的url生成的请求，并将`parse`方法作为请求的回调函数。

2.在回调函数中，您将解析响应(web页面)，并返回提取的数据、`Item`对象、`Request`对象或这些对象的迭代字典来返回这些命令。这些请求还将包含一个回调(可能是相同的)，然后将被Scrapy下载，然后由指定的回调函数处理它们的响应。

3.在回调函数中，您可以解析页面内容，通常使用[选择器]()(但您也可以使用Beautifulsoup、lxml或任何您喜欢的机制)，并生成带有解析数据的项目。

4.最后，从爬行器返回的项通常会保存到数据库(在某些[项目管道]()中)或使用[Feed输出]()写入文件。

尽管这一循环(或多或少)适用于任何种类的蜘蛛，但有不同类型的默认爬行器被绑定到不同的目的。我们将在这里讨论这些类型。
##scrapy.Spider
>class scrapy.spiders.Spider

这是最简单的蜘蛛，也是其他蜘蛛必须继承的蜘蛛（包括与Scrapy捆绑在一起的蜘蛛，以及自己写的蜘蛛）。它不提供任何特殊功能。它只是提供了一个默认的`start_requests()`实现，它通过spider的`start_urls` 属性发送请求，并为每个结果响应调用`parse`方法。
###name
定义此蜘蛛名称的字符串。蜘蛛名称是Scrapy定位（并实例化）蜘蛛的关键词，因此它必须是唯一的。然而，没有什么能够阻止你为同一个蜘蛛类实例化多个实例。这是最重要的蜘蛛属性，它是必需的。

如果蜘蛛刮取单个域名，通常的做法用域名命名蜘蛛。例如，爬行`mywebsite.com`的蜘蛛通常会被命名为`mywebsite`。
>注意

>在Python 2中，这只能是ASCII。

###allowed_domains
包含允许此蜘蛛抓取的域的可选字符串列表。如果`OffsiteMiddleware`启用，则不会遵循不属于此列表中指定的域名（或其子域）的URL的请求 。

假设你的目标网址是`https://www.example.com/1.html`，则添加`'example.com'`到列表中。
###start_urls
当没有指定特定网址时，蜘蛛将从哪个网址开始抓取的网址列表。所以，下载的第一个页面将在这里列出。随后的URL将从包含在起始URL中的数据中连续生成。
###custom_settings
运行此蜘蛛时设置字典将被项目范围的配置覆盖。它必须被定义为类属性，因为设置在实例化之前会被更新。

有关可用内置设置的列表，请参阅： [内置设置参考]()。
###crawler
在初始化该类后，`class`属性由`from_crawler()` 方法设置，并链接`Crawler`到此spider实例绑定的 对象。

爬虫在项目中封装了大量组件，以便进行单一入口访问（例如扩展，中间件，信号管理器等）。请参阅[Crawler API]()以了解更多关于它们的信息。
###settings
运行这个蜘蛛的配置。这是一个Settings实例，请参阅[设置]()主题以获取关于此主题的详细介绍。
###logger
Python记录器是用蜘蛛的`name`创建的。您可以按照[蜘蛛日志]()中所述，使用它来发送日志消息 。
###from_crawler（crawler，* args，** kwargs ）
这是Scrapy用来创建蜘蛛的类方法。

你可能不需要直接覆盖它，因为默认的实例化方法作为`__init__()`方法的代理，用给定参数`args`和命名参数`kwargs`调用它。

尽管如此，这个方法 在新实例中设置`crawler`和`settings`属性，以便蜘蛛在之后的代码中访问它们。


**参数**：
	
- 爬虫（`Crawler`实例） - 蜘蛛将被绑定到的爬虫
- args（`list`） - 传递给`__init__()`方法的参数
- `kwargs（dict）` - 传递给`__init__()`方法的关键字参数
###start_requests（）
此方法必须返回一个可抓取的，可迭代的，第一个请求。在蜘蛛被打开时，此方法将被Scrapy调用。Scrapy只会调用它一次，因此`start_requests()`可作为生成器。

默认实例会在`start_urls`中为每个url生成`Request(url, dont_filter=True)`
如果您想更改用于开始抓取域的请求，则需要覆盖此方法。例如，如果您需要使用POST请求登录，则可以执行以下操作：

	class MySpider(scrapy.Spider):
	    name = 'myspider'
	
	    def start_requests(self):
	        return [scrapy.FormRequest("http://www.example.com/login",
	                                   formdata={'user': 'john', 'pass': 'secret'},
	                                   callback=self.logged_in)]
	
	    def logged_in(self, response):
	        # here you would extract links to follow and return Requests for
	        # each of them, with another callback
	        pass


###parse(response)
这是Scrapy用来处理下载响应的默认回调，当它们的请求没有指定回调时。

`parse`方法负责处理响应并返回抓取的数据以及更多的URL。其他请求回调与`Spider`类有相同的要求。

这个方法以及任何其他的Request回调都必须返回可迭代的`Request`或`dicts` 或`Item`对象。

**参数**：	

	response（Response） - 解析的响应
###log(message[, level, component])
通过Spider的`logger`方法发送日志消息的包装器，保持向后兼容性。有关更多信息，请参阅[蜘蛛日志]()。
###closed（reason）
当蜘蛛关闭时调用。此方法为`signal.connect（）`提供了快捷方式，作为`spider_closed`的信号。

我们来看一个例子：

	import scrapy
	
	class MySpider(scrapy.Spider):
	    name = 'example.com'
	    allowed_domains = ['example.com']
	    start_urls = [
	        'http://www.example.com/1.html',
	        'http://www.example.com/2.html',
	        'http://www.example.com/3.html',
	    ]
	
	    def parse(self, response):
	        self.logger.info('A response from %s just arrived!', response.url)

从单个回调中返回多个请求和项目：

	import scrapy
	class MySpider(scrapy.Spider):
	    name = 'example.com'
	    allowed_domains = ['example.com']
	    start_urls = [
	        'http://www.example.com/1.html',
	        'http://www.example.com/2.html',
	        'http://www.example.com/3.html',
	    ]
	
	    def parse(self, response):
	        for h3 in response.xpath('//h3').extract():
	            yield {"title": h3}
	
	        for url in response.xpath('//a/@href').extract():
	            yield scrapy.Request(url, callback=self.parse)

相较于`start_urls`，你可以直接使用`start_requests()`给数据更多的结构。你可以阅读[Items]()：

	import scrapyfrom myproject.items import MyItem
	class MySpider(scrapy.Spider):
    name = 'example.com'
    allowed_domains = ['example.com']

    def start_requests(self):
        yield scrapy.Request('http://www.example.com/1.html', self.parse)
        yield scrapy.Request('http://www.example.com/2.html', self.parse)
        yield scrapy.Request('http://www.example.com/3.html', self.parse)

    def parse(self, response):
        for h3 in response.xpath('//h3').extract():
            yield MyItem(title=h3)

        for url in response.xpath('//a/@href').extract():
            yield scrapy.Request(url, callback=self.parse)
##蜘蛛的参数
蜘蛛可以接收修改其行为的参数。蜘蛛参数的一些常见用途是定义起始URL或限制爬取到站点的某些部分，但实际上它们可用于配置蜘蛛的任何功能。
蜘蛛参数通过使用`crawl`命令`-a`选项传递。例如：

	scrapy crawl myspider -a category=electronics
蜘蛛可以在`__init__`方法中访问参数：

	import scrapy
	class MySpider(scrapy.Spider):
    name = 'myspider'

    def __init__(self, category=None, *args, **kwargs):
        super(MySpider, self).__init__(*args, **kwargs)
        self.start_urls = ['http://www.example.com/categories/%s' % category]
        # ...
默认的__init__方法将采用任何蜘蛛参数并将其作为属性复制到蜘蛛中。上面的例子也可以写成如下：

	import scrapy
	class MySpider(scrapy.Spider):
    name = 'myspider'

    def start_requests(self):
        yield scrapy.Request('http://www.example.com/categories/%s' % self.category)
请记住，蜘蛛参数只是字符串。蜘蛛不会自行解析。如果要从命令行设置`start_urls`属性，则必须使用`ast.literal_eval` 或`json.loads`之类的东西将它自己解析为列表 ，然后将其设置为属性。否则，会导致对一个start_urls字符串进行迭代（一个非常常见的python陷阱），导致每个字符被视为一个单独的url。

有效的用例是设置由`HttpAuthMiddleware`所使用的http认证凭证或`UserAgentMiddleware`所使用的用户代理

	scrapy crawl myspider -a http_user=myuser -a http_pass=mypassword -a user_agent=mybot
蜘蛛参数也可以通过Scrapyd `schedule.json` API 传递。请参阅[Scrapyd文档]()。
##通用蜘蛛
Scrapy附带了一些有用的通用蜘蛛，您可以使用它们对蜘蛛进行子类化。他们的目标是为一些常见的刮取案例提供便利，比如遵循特定规则的网站上的所有链接，从`Sitemaps`抓取或解析XML/CSV供稿。

对于下面的蜘蛛中使用的例子，我们假设你有一个在`myproject.items`模块中声明的`TestItem`项目：

	import scrapy
	class TestItem(scrapy.Item):
	    id = scrapy.Field()
	    name = scrapy.Field()
	    description = scrapy.Field()

###CrawlSpider
>class scrapy.spiders.CrawlSpider


这是抓取常规网站最常用的蜘蛛，因为它提供了一个通过定义一组规则来跟踪链接的便捷机制。它可能不是特别适合您特定网站或项目，但它对于多种情况是足够通用的，所以您可以从它开始并根据需要，覆盖它以获得更多自定义功能，或者只是实现您自己的蜘蛛。

除了从Spider继承的属性（必须指定）之外，该类还支持一个新的属性：

`rules`

它是一个（或多个）Rule对象的列表。每个Rule 都定义了用于爬网的特定行为。Rules对象如下所述。如果多个规则匹配相同的链接，则会根据它在此属性中定义的顺序使用第一个规则。

这个蜘蛛也暴露了一个可覆盖的方法：

`parse_start_url（response）`

这个方法被称为`start_urls`响应。它允许解析初始响应，并且必须返回一个`Item`对象，一个`Request`对象或包含它们的迭代器。
###抓取规则
>class scrapy.spiders.Rule（link_extractor，callback = None，cb_kwargs = None，follow = None，process_links = None，process_request = None ）

`link_extractor`是一个链接提取器对象，它定义了如何从每个已爬网页中提取链接。

`callback`是一个可调用的方法或字符串（在这种情况下，表示具有该字符串名称的蜘蛛对象的方法）由使用`link_extractor`指定的每个被提取出来的链接调用。这个回调接收到一个响应作为它的第一个参数，并且必须返回一个包含`Item`或`Request`对象（或者它们的任何子类）的列表。
>警告
>
编写爬网规则时，避免使用`parse`回调函数，因为`CrawlSpider`会使用`parse`方法本身来实现其逻辑。因此，如果您重写该`parse`方法，抓取蜘蛛将不再起作用。

`cb_kwargs`是一个包含要传递给回调函数的关键字参数的字典。

`follow`是一个布尔值，用于指定是否应该遵循通过此规则提取的每个响应之间的链接。如果`callback`是无`follow`默认值`True`，则默认为`False`。

`process_links`是可调用的方法或字符串（在这种情况下，将使用具有该字符串名称的蜘蛛对象的方法），将使用指定的对每个响应中提取的每个链接列表调用该方法`link_extractor`。这主要用于过滤目的。

`process_request`是一个可调用的方法或字符串（在这种情况下，将使用具有该字符串名称的spider对象的方法），该方法将在此规则提取的每个请求中调用，并且必须返回请求或None（以过滤请求） 。

###CrawlSpider示例

现在我们来看一个带有规则的CrawlSpider示例：

	import scrapyfrom scrapy.spiders import CrawlSpider, Rulefrom scrapy.linkextractors import LinkExtractor
	class MySpider(CrawlSpider):
	    name = 'example.com'
	    allowed_domains = ['example.com']
	    start_urls = ['http://www.example.com']
	
	    rules = (
	        # Extract links matching 'category.php' (but not matching 'subsection.php')
	        # and follow links from them (since no callback means follow=True by default).
	        Rule(LinkExtractor(allow=('category\.php', ), deny=('subsection\.php', ))),
	
	        # Extract links matching 'item.php' and parse them with the spider's method parse_item
	        Rule(LinkExtractor(allow=('item\.php', )), callback='parse_item'),
	    )
	
	    def parse_item(self, response):
	        self.logger.info('Hi, this is an item page! %s', response.url)
	        item = scrapy.Item()
	        item['id'] = response.xpath('//td[@id="item_id"]/text()').re(r'ID: (\d+)')
	        item['name'] = response.xpath('//td[@id="item_name"]/text()').extract()
	        item['description'] = response.xpath('//td[@id="item_description"]/text()').extract()
	        return item
这个蜘蛛将开始抓取example.com的主页，收集分类链接和项目链接，并用`parse_item`方法解析后者。对于每个项目响应，将使用XPath从HTML中提取一些数据，并填充到`Item`中
###XMLFeedSpider
>Calss scrapy.spiders.XMLFeedSpider

XMLFeedSpider旨在通过以特定节点名称遍历它们来解析XML提要。迭代器可以选自：`iternodes`，`xml`，和`html`。因为性能方面的原因，我们推荐使用`iternodes`，因为`xml`和`html`迭代器会一次生成整个DOM来解析。但是，`html`在解析具有错误标记的`XML`时，用作迭代器可能会很有用。

要设置迭代器和标签名称，您必须定义以下类属性：
>iterator

一个定义要使用的迭代器的字符串。它可以是：

- 'iternodes' - 基于正则表达式的快速迭代器
- 'html'- 一个使用`Selector`的迭代器。请记住，这使用DOM解析，并且必须加载内存中的所有DOM，这可能是大型提要的问题
- 'xml'- 一个使用`Selector`的迭代器。请记住，这使用DOM解析，并且必须加载内存中的所有DOM，这可能是大型提要的问题

它默认为：'iternodes'。

>itertag

一个字符串，其中包含要迭代的节点（或元素）的名称。示例：

	itertag = 'product'

>namespaces

定义该文档中可用的名称空间的`(prefix, uri)`元组列表，该列表将与该蜘蛛一起处理。`prefix`和`uri`将被用于自动注册使用的命名空间的`register_namespace()`方法。

然后，您可以在`itertag`属性中指定具有名称空间的节点。

例：

	class YourSpider(XMLFeedSpider):
	    namespaces = [('n', 'http://www.sitemaps.org/schemas/sitemap/0.9')]
	    itertag = 'n:url'
	    # ...
除了这些新的属性外，这个蜘蛛还有以下可覆盖的方法：
>adapt_response（response）

一旦响应到达爬虫中间件，在蜘蛛开始分析它之前，该方法会拦截响应。它可以用来修改响应主体。这个方法接收一个响应，并返回一个响应（它可能是相同的或另一个）。

>parse_node（response，selector）

为与提供的标签名称（`itertag`）匹配的节点调用此方法。接收响应并为每个`Selector`节点分配一个响应 。覆盖此方法是强制性的。否则，你的蜘蛛将无法工作。此方法必须返回一个`Item`对象，一个`Request`对象或包含它们的迭代器。

>process_results（response，results）

该方法针对蜘蛛所返回的每个结果（项目或请求）进行调用，并且它将在将结果返回给框架核心之前执行所需的最后一次处理，例如设置项目ID。它会收到一份结果清单和源自这些结果的回复。它必须返回结果列表（项目或请求）。
###XMLFeedSpider示例
这些蜘蛛很容易使用，让我们看看一个例子：

	from scrapy.spiders import XMLFeedSpiderfrom myproject.items import TestItem
	class MySpider(XMLFeedSpider):
	    name = 'example.com'
	    allowed_domains = ['example.com']
	    start_urls = ['http://www.example.com/feed.xml']
	    iterator = 'iternodes'  # This is actually unnecessary, since it's the default value
	    itertag = 'item'
	
	    def parse_node(self, response, node):
	        self.logger.info('Hi, this is a <%s> node!: %s', self.itertag, ''.join(node.extract()))
	
	        item = TestItem()
	        item['id'] = node.xpath('@id').extract()
	        item['name'] = node.xpath('name').extract()
	        item['description'] = node.xpath('description').extract()
	        return item
基本上我们在那里做的是创建一个蜘蛛，从给定的`start_urls`下载数据，然后遍历每个`item`标签，打印出来，并在`Item`中随机存储一些数据。
###CSVFeedSpider
>class scrapy.spiders.CSVFeedSpider

这个蜘蛛与XMLFeedSpider非常相似，只是它遍历行而不是节点。在每次迭代中调用的方法是`parse_row()`。
>delimiter

包含CSV文件中每个字段的分隔符的字符串默认为`','`（逗号）。
>quotechar

包含CSV文件中每个字段的外壳字符的字符串默认为`'"'`（引号）。
>headers

CSV文件中列名称的列表。
>parse_row（响应，行）

接收每个提供（或检测）的CSV文件标题的响应和字典（代表每行）。这个蜘蛛也提供了覆盖的机会`adapt_response`以及`process_results`用于预处理和后处理的方法。
###CSVFeedSpider示例
我们来看一个和上一个类似的例子，但是使用 `CSVFeedSpider`：
	
	from scrapy.spiders import CSVFeedSpiderfrom myproject.items import TestItem
	class MySpider(CSVFeedSpider):
	    name = 'example.com'
	    allowed_domains = ['example.com']
	    start_urls = ['http://www.example.com/feed.csv']
	    delimiter = ';'
	    quotechar = "'"
	    headers = ['id', 'name', 'description']
	
	    def parse_row(self, response, row):
	        self.logger.info('Hi, this is a row!: %r', row)
	
	        item = TestItem()
	        item['id'] = row['id']
	        item['name'] = row['name']
	        item['description'] = row['description']
	        return item
###SitemapSpider
>Class scrapy.spiders.SitemapSpider

SitemapSpider允许您通过使用`Sitemaps`发现网址来抓取网站。

它支持嵌套站点地图并从robots.txt中发现站点地图网址 。
>sitemap_urls

指向您要抓取的网址的站点地图的网址列表。
您也可以指向一个`robots.txt`，并将其解析为从中提取网站地图网址。
>sitemap_rules

元组列表，其中元祖为：`(regex, callback)`

- `regex`是一个正则表达式，用于匹配从站点地图提取的网址。 `regex`可以是str或编译的正则表达式对象。
- `Callback`是用于处理匹配正则表达式的url的回调。`callback`可以是一个字符串（表示蜘蛛方法的名称）或可调用的字符串。

例如：

	sitemap_rules = [('/product/', 'parse_product')]

规则按顺序应用，只有匹配的第一个将被使用。
如果你忽略这个属性，所有在站点地图中找到的URL都将被`parse`回调处理。
>sitemap_follow

应遵循的站点地图正则表的列表。这仅适用于使用指向其他站点地图文件的站点地图索引文件的站点。

默认情况下，遵循所有站点地图。

`sitemap_alternate_links`

指定一个`url`是否应该遵循替代链接。这些是在同一个`url`区块内传递的另一种语言的同一网站的链接。

例如：

	<url>
	    <loc>http://example.com/</loc>
	    <xhtml:link rel="alternate" hreflang="de" href="http://example.com/de"/>
	</url>
有了`sitemap_alternate_links`集，这将同时检索的网址。随着`sitemap_alternate_links`禁用，只`http://example.com/`将被检索。
默认`sitemap_alternate_links`是禁用的。
###SitemapSpider的例子
最简单的例子：使用parse回调处理通过站点地图发现的所有网址 ：

	from scrapy.spiders import SitemapSpider
	class MySpider(SitemapSpider):
	    sitemap_urls = ['http://www.example.com/sitemap.xml']
	
	    def parse(self, response):
	        pass # ... scrape item here ...
使用某些回调处理一些url以及使用其他回调处理其他url：

	from scrapy.spiders import SitemapSpider
	class MySpider(SitemapSpider):
	    sitemap_urls = ['http://www.example.com/sitemap.xml']
	    sitemap_rules = [
	        ('/product/', 'parse_product'),
	        ('/category/', 'parse_category'),
	    ]
	
	    def parse_product(self, response):
	        pass # ... scrape product ...
	
	    def parse_category(self, response):
	        pass # ... scrape category ...
遵循`robots.txt`文件中定义的站点地图，并且只跟随其网址包含以下内容的站点地图`/sitemap_shop`：

	from scrapy.spiders import SitemapSpider
	class MySpider(SitemapSpider):
	    sitemap_urls = ['http://www.example.com/robots.txt']
	    sitemap_rules = [
	        ('/shop/', 'parse_shop'),
	    ]
	    sitemap_follow = ['/sitemap_shops']
	
	    def parse_shop(self, response):
	        pass # ... scrape shop here ...
将SitemapSpider与其他网址来源相结合：
	
	from scrapy.spiders import SitemapSpider
	class MySpider(SitemapSpider):
	    sitemap_urls = ['http://www.example.com/robots.txt']
	    sitemap_rules = [
	        ('/shop/', 'parse_shop'),
	    ]
	
	    other_urls = ['http://www.example.com/about']
	
	    def start_requests(self):
	        requests = list(super(MySpider, self).start_requests())
	        requests += [scrapy.Request(x, self.parse_other) for x in self.other_urls]
	        return requests
	
	    def parse_shop(self, response):
	        pass # ... scrape shop here ...
	
	    def parse_other(self, response):
	        pass # ... scrape other here ...
