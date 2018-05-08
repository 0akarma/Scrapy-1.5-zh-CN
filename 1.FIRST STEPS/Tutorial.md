# Scrapy指南

## 前言

当你接着往下看这个文档的时候，我们将默认您已经在电脑上成功安装好了Scrapy，如果还没有安装，请看[安装指南](https://doc.scrapy.org/en/latest/intro/install.html#intro-install)

我们将会从爬取一个罗列了一些著名作家的名言的网站[quotes.toscrape.com](http://quotes.toscrape.com/)开始

这份指南将会陪你你一起学习下列知识：

1. 创建一个新的Scrapy项目
2. 写一个Spider去爬取网站以及从中提取数据
3. 用命令来输出爬取的数据
4. 让spider递归爬取所有相关网站
5. 会使用spider的参数

Scrapy是用Python写的，所以在这之前你没有学过Python的话，你或许想知道，Python是个什么东西？除了爬虫之外，Python还能拿来干嘛？

如果你在这之前已经掌握了一门或多门其他计算机语言，然后想快速学习Python，我们推荐你阅读一些Python教程如：[Dive Into Python 3](http://www.diveintopython3.net/)，或者你也可以直接看这篇Scrapy文档。

如果你是一个编程小白，然后想用Python来入门编程。那么你可以在网上找到一些有用的教程如：[Learn Python The Hard Way](https://learnpythonthehardway.org/book/)，你也可以看一下这篇文章[this list of Python resources for non-programmers](https://wiki.python.org/moin/BeginnersGuide/NonProgrammers)

## 创建一个项目

在开始爬虫之前，你要先建立一个Scrapy项目。进入一个你想存放代码的目录，然后输入如下命令：

```bash
scrapy startproject tutorial
```

然后就创建了一个叫**tutorial**且带有如下文件的目录

```bash
tutorial/
    scrapy.cfg            # deploy configuration file
    tutorial/             # project's Python module, you'll import your code from here
        __init__.py
        items.py          # project items definition file
        middlewares.py    # project middlewares file
        pipelines.py      # project pipelines file
        settings.py       # project settings file
        spiders/          # a directory where you'll later put your spiders
            __init__.py
```

## 第一个爬虫

Spiders是一群听你指挥的士兵（因为她由你来定义），他们会按照你的命令，去爬取一个或者一群网站。他们是你Scrapy军团的一个分支，你要告诉他们首先打哪里，，线索在哪里，下面要继续打哪里，还有怎么搜刮城中的战利品。

下面是我们第一个爬虫的代码，把如下代码储存在你项目/tutorial/spiders的目录下，并命名为**quotes_spider.py **

```python
import scrapy


class QuotesSpider(scrapy.Spider):
    name = "quotes"

    def start_requests(self):
        urls = [
            'http://quotes.toscrape.com/page/1/',
            'http://quotes.toscrape.com/page/2/',
        ]
        for url in urls:
            yield scrapy.Request(url=url, callback=self.parse)

    def parse(self, response):
        page = response.url.split("/")[-2]
        filename = 'quotes-%s.html' % page
        with open(filename, 'wb') as f:
            f.write(response.body)
        self.log('Saved file %s' % filename)
```

正如你所看到的那样，我们的爬虫有一个[`scrapy.Spider`](https://doc.scrapy.org/en/latest/topics/spiders.html#scrapy.spiders.Spider) （一个已经定义了一些属性和方法的类）：

- [`name`](https://doc.scrapy.org/en/latest/topics/spiders.html#scrapy.spiders.Spider.name): 为了便于辨认各个爬虫，所以就必须要求每个爬虫互不重名。
- [`start_requests()`](https://doc.scrapy.org/en/latest/topics/spiders.html#scrapy.spiders.Spider.start_requests): 
- [`parse()`](https://doc.scrapy.org/en/latest/topics/spiders.html#scrapy.spiders.Spider.parse): 这是一个拿来处理来自每个request请求的回应。回应的参数，是一个带有网页内容文本实例（[`TextResponse`](https://doc.scrapy.org/en/latest/topics/request-response.html#scrapy.http.TextResponse)），这个实例，我们后续有一些实用的方法可以处理它

[`parse()`](https://doc.scrapy.org/en/latest/topics/spiders.html#scrapy.spiders.Spider.parse) 这个方法，经常用来处理回应，将储存在字典中的爬取数据提取出来，然后寻找下一个新的目标url，继续发送新的请求。

## 如何开始我们的爬虫

要让我们的爬虫运行起来，只需要cd到我们的项目根目录，然后执行如下命令：

```bash
scrapy crawl quotes
```

这个命令就是执行我们上面建立的名叫**quotes**的爬虫文件，它会发送请求到`quotes.toscrape.com`这个域名上，然后你的命令行上就有类似如下输出：

```shell
... (omitted for brevity)
2016-12-16 21:24:05 [scrapy.core.engine] INFO: Spider opened
2016-12-16 21:24:05 [scrapy.extensions.logstats] INFO: Crawled 0 pages (at 0 pages/min), scraped 0 items (at 0 items/min)
2016-12-16 21:24:05 [scrapy.extensions.telnet] DEBUG: Telnet console listening on 127.0.0.1:6023
2016-12-16 21:24:05 [scrapy.core.engine] DEBUG: Crawled (404) <GET http://quotes.toscrape.com/robots.txt> (referer: None)
2016-12-16 21:24:05 [scrapy.core.engine] DEBUG: Crawled (200) <GET http://quotes.toscrape.com/page/1/> (referer: None)
2016-12-16 21:24:05 [scrapy.core.engine] DEBUG: Crawled (200) <GET http://quotes.toscrape.com/page/2/> (referer: None)
2016-12-16 21:24:05 [quotes] DEBUG: Saved file quotes-1.html
2016-12-16 21:24:05 [quotes] DEBUG: Saved file quotes-2.html
2016-12-16 21:24:05 [scrapy.core.engine] INFO: Closing spider (finished)
...
```

现在，我们可以发现，目录下多了两个html文件：*quotes-1.html* 和 *quotes-2.html* ，他们根据我们**parse**方法的定义，各自储存着自己网页上的内容。

>**!Note**
>
>如果你想问，为什么我们还没开始分析我们的html？不用急。。我们待会就会讲到:)

## 刚刚发生了些什么？

## 条条大路通start_requests(方法)

除了用[`start_requests()`](https://doc.scrapy.org/en/latest/topics/spiders.html#scrapy.spiders.Spider.start_requests)方法来请求网页之外，你 还可以定义一个[`start_urls`](https://doc.scrapy.org/en/latest/topics/spiders.html#scrapy.spiders.Spider.start_urls) 类带有一堆url的属性。这样子，就可以将它们排列起来，然后传递给[`start_requests()`](https://doc.scrapy.org/en/latest/topics/spiders.html#scrapy.spiders.Spider.start_requests) 帮你的爬虫完成网页请求操作。

```python
import scrapy


class QuotesSpider(scrapy.Spider):
    name = "quotes"
    start_urls = [
        'http://quotes.toscrape.com/page/1/',
        'http://quotes.toscrape.com/page/2/',
    ]

    def parse(self, response):
        page = response.url.split("/")[-2]
        filename = 'quotes-%s.html' % page
        with open(filename, 'wb') as f:
            f.write(response.body)
```

[`parse()`](https://doc.scrapy.org/en/latest/topics/spiders.html#scrapy.spiders.Spider.parse) 这个方法，会被用来处理每一个url的请求。即使我们还没有明确的告诉Scrapy要这样做，但是，因为[`parse()`](https://doc.scrapy.org/en/latest/topics/spiders.html#scrapy.spiders.Spider.parse) 是Scrapy的默认回调方法，即是不用妈妈吩咐，就能自动自觉地去做家务的“乖乖仔”噢！！

## 提取数据

要问怎么学习用Scrapy提取数据最有用，那就是尝试结合Scrapy的shell命令去学习利用选择器（CSS、Xpath），运行如下命令：

```shell
scrapy shell 'http://quotes.toscrape.com/page/1/'
```

> !Note
>
> 记得url地址要用   **'**  单引号闭合，不然会导致一些不必要的错误。
>
> 如果是windows下，请把单引号换成双引号，如下：
>
> ```shell
> scrapy shell "http://quotes.toscrape.com/page/1/"
> ```

你就会看到如下响应：

```python
[ ... Scrapy log here ... ]
2016-09-19 12:09:27 [scrapy.core.engine] DEBUG: Crawled (200) <GET http://quotes.toscrape.com/page/1/> (referer: None)
[s] Available Scrapy objects:
[s]   scrapy     scrapy module (contains scrapy.Request, scrapy.Selector, etc)
[s]   crawler    <scrapy.crawler.Crawler object at 0x7fa91d888c90>
[s]   item       {}
[s]   request    <GET http://quotes.toscrape.com/page/1/>
[s]   response   <200 http://quotes.toscrape.com/page/1/>
[s]   settings   <scrapy.settings.Settings object at 0x7fa91d888c10>
[s]   spider     <DefaultSpider 'default' at 0x7fa91c8af990>
[s] Useful shortcuts:
[s]   shelp()           Shell help (print this help)
[s]   fetch(req_or_url) Fetch request (or URL) and update local objects
[s]   view(response)    View response in a browser
>>>
```

运用如上的一些命令，你就可以尝试运用[CSS选择器](https://www.w3.org/TR/selectors/)的一些知识寻找出响应对象中的一些元素：

```python
>>> response.css('title')
[<Selector xpath='descendant-or-self::title' data='<title>Quotes to Scrape</title>'>]

```

运行如上命令的结果就是得到一个叫选择器表（[`SelectorList`](https://doc.scrapy.org/en/latest/topics/selectors.html#scrapy.selector.SelectorList)）的表对象，这就意味着这是一个包含了XML\HTML元素的可以用于接下来进一步提取的选择器。

要提取上面title中的内容，你可以输入如下命令：

```python
>>> response.css('title::text').extract()
['Quotes to Scrape']
```

这里有两点值得注意：

1. 我们要在CSS查找指令中加上`::text` ，因为这样才能表明我们只要<title>中的文本元素，如果我们不加`::text` ，我们就会得到含title标签的文本：

```python
>>> response.css('title').extract()
['<title>Quotes to Scrape</title>']
```

2. `.extract()`的返回结果是一个列表，因为我们正在处理一个选择器表，如果你知道自己所要的内容就是第一个内容，你就可以这样：

```python
>>> response.css('title::text').extract_first()
'Quotes to Scrape'
```

当然，你也可以这样：

```python
>>> response.css('title::text')[0].extract()
'Quotes to Scrape'
```

## 简单介绍XPath

除了CSS选择器之外，Scrapy还支持Xpath选择器：

```python
>>> response.xpath('//title')
[<Selector xpath='//title' data='<title>Quotes to Scrape</title>'>]
>>> response.xpath('//title/text()').extract_first()
'Quotes to Scrape'
```

Xpath选择器十分的高效，同时她也是Scrapy选择器的基础。实际上，当你在命令行中看网页的源代码的时候，你就会发现CSS选择器其实是由Xpath选择器演变而来的。

虽然Xpath的使用人数没有CSS多，但是Xpath在定位网站结构的时候更好用，她同样可以获取网站内容。而已用Xpath，你可以搜索一些东西如：*select the link that contains the text “Next Page”*，这就让Xpath十分适合用于爬虫的选择器。所以我们推荐你学习Xpath即使你已经知道如何使用CSS选择器，但是。Xpath会让爬虫更简单一些。

这里我们不过多介绍Xpath，但是你可以自己搜索相关文章学习

这里我们也推荐一些关于Xpath的文章：

 [using XPath with Scrapy Selectors here](https://doc.scrapy.org/en/latest/topics/selectors.html#topics-selectors).

深入学习：

 [this tutorial to learn XPath through examples](http://zvon.org/comp/r/tut-XPath_1.html), 

 [this tutorial to learn “how to think in XPath”](http://plasmasturm.org/log/xpath101/).

## 如何提取引文和作者

既然你已经了解了一些筛选和提取数据的知识，那就让我们开始往我们的spider里面添加一些提取数据的代码吧！

```python
<div class="quote">
    <span class="text">“The world as we have created it is a process of our
    thinking. It cannot be changed without changing our thinking.”</span>
    <span>
        by <small class="author">Albert Einstein</small>
        <a href="/author/Albert-Einstein">(about)</a>
    </span>
    <div class="tags">
        Tags:
        <a class="tag" href="/tag/change/page/1/">change</a>
        <a class="tag" href="/tag/deep-thoughts/page/1/">deep-thoughts</a>
        <a class="tag" href="/tag/thinking/page/1/">thinking</a>
        <a class="tag" href="/tag/world/page/1/">world</a>
    </div>
</div>
```

打开Scrapy 命令行，然后自己尝试如何提取我们想要的信息：

```bash
$ scrapy shell 'http://quotes.toscrape.com'
```

我们首先要用选择器，初步提取我们所要的html元素，命令如下：

```python
>>> response.css("div.quote")
```

每一个选择器都会返回页面中所有你查询的元素，这样便于我们接着继续查询他们的子类。所以让我们先把第一个选择器返回值存在变量quote中，这样我们就可以在特定的选择器中继续用css查找我们想要的信息。

```Python
>>> quote = response.css("div.quote")[0]
```

现在，让我们从我们刚刚创建的变量中提取`title`, `author` 和一些其他标签：

```python
>>> title = quote.css("span.text::text").extract_first()
>>> title
'“The world as we have created it is a process of our thinking. It cannot be changed without changing our thinking.”'
>>> author = quote.css("small.author::text").extract_first()
>>> author
'Albert Einstein'
```

上述标签都是存在于数组中的字符串，我们可以用`.extract()` 方法获取他们：

```Python
>>> tags = quote.css("div.tags a.tag::text").extract()
>>> tags
['change', 'deep-thoughts', 'thinking', 'world']
```

在学会如何提取一小部分数据之后，我们可以迭代获取所有quote中的元素，然后把他们全部存放在一个字典中，便于后续使用：

```python
>>> for quote in response.css("div.quote"):
...     text = quote.css("span.text::text").extract_first()
...     author = quote.css("small.author::text").extract_first()
...     tags = quote.css("div.tags a.tag::text").extract()
...     print(dict(text=text, author=author, tags=tags))
{'tags': ['change', 'deep-thoughts', 'thinking', 'world'], 'author': 'Albert Einstein', 'text': '“The world as we have created it is a process of our thinking. It cannot be changed without changing our thinking.”'}
{'tags': ['abilities', 'choices'], 'author': 'J.K. Rowling', 'text': '“It is our choices, Harry, that show what we truly are, far more than our abilities.”'}
    ... a few more of these, omitted for brevity
>>>
```

## 在我们的爬虫中提取数据

看回我们的爬虫代码，发现我们好像还没有开始提取任何数据，只是把整个html网页存在了一个本地文件中。那就让我们把我们的提取数据操作融入到我们的爬虫中吧！

一个Scrapy爬虫通常会创建多个用于存取从网页上爬取的数据的字典。所以我们就要用到`yield`这个Python关键词来回调，如下：

```python
import scrapy


class QuotesSpider(scrapy.Spider):
    name = "quotes"
    start_urls = [
        'http://quotes.toscrape.com/page/1/',
        'http://quotes.toscrape.com/page/2/',
    ]

    def parse(self, response):
        for quote in response.css('div.quote'):
            yield {
                'text': quote.css('span.text::text').extract_first(),
                'author': quote.css('small.author::text').extract_first(),
                'tags': quote.css('div.tags a.tag::text').extract(),
            }

```

如果你跑这个spider，爬取的数据就会从log日志中输出出来。

```python
2016-09-19 18:57:19 [scrapy.core.scraper] DEBUG: Scraped from <200 http://quotes.toscrape.com/page/1/>
{'tags': ['life', 'love'], 'author': 'André Gide', 'text': '“It is better to be hated for what you are than to be loved for what you are not.”'}
2016-09-19 18:57:19 [scrapy.core.scraper] DEBUG: Scraped from <200 http://quotes.toscrape.com/page/1/>
{'tags': ['edison', 'failure', 'inspirational', 'paraphrased'], 'author': 'Thomas A. Edison', 'text': "“I have not failed. I've just found 10,000 ways that won't work.”"}
```

## 如何储存我们所爬取的数据

最简单的存储方式就是用[Feed exports](https://doc.scrapy.org/en/latest/topics/feed-exports.html#topics-feed-exports)，可以通过如下命令实现：

```bash
scrapy crawl quotes -o quotes.json
```

这会产生一个`quotes.json` 文件，文件中包含所有通过 [JSON](https://en.wikipedia.org/wiki/JSON)加载的爬取数据。

由于一些历史原因，Scrapy通常只会在文件末尾写入数据，而不是覆盖她的所有内容。即如果你运行这条命令后没有删除其中的内容，再运行一次，就会发生一些错误。

你也可以用其他的方式，如[JSON Lines](http://jsonlines.org/)：

```bash
scrapy crawl quotes -o quotes.jl
```

这个方法十分有用因为她是一行一行保存数据的，所以你可以轻松添加新的内容。而且在运行两次的时候，她不会出现JSON的问题。因为每一条数据都储存在独立的一行，因此您可以处理大文件而不必将所有内容都放在内存中，但有像[JQ](https://stedolan.github.io/jq)这样的工具可以帮助在命令行执行该操作。

在小型项目中（如这个指南中的项目），JSON Lines这个存储方式足矣。但是如果你要处理爬来的更多更复杂的东西，那你就可以写一个 [Item Pipeline](https://doc.scrapy.org/en/latest/topics/item-pipeline.html#topics-item-pipeline)来实现。这个文件在你创建项目的时候已经创建好了`tutorial/pipelines.py`。如果你只是需要存储数据，那你大可不必管这个文件。

## 如何利用Scrapy进行多页爬取

爬虫难道只能爬前面两页吗？当我们需要连续迭代爬取所有网页时，我们该如何做？

既然你已经知道如何从网页中提取数据，接下来让我们学习如何寻找“下一页”目标吧！

首先要做的就是找到并提取“下一页”的地址在哪里。在我们的示例中，我们可以看到有一个链接指向下一页，源码如下：

```html
<ul class="pager">
    <li class="next">
        <a href="/page/2/">Next <span aria-hidden="true">&rarr;</span></a>
    </li>
</ul>
```

我们可以首先在命令行中手动提取她：

```python
>>> response.css('li.next a').extract_first()
'<a href="/page/2/">Next <span aria-hidden="true">→</span></a>'
```

这样我们就得到了a锚点元素，但是我们想要的是 `href`中的内容。庆幸的是，Scrapy提供了一个CSS的拓展，可以让你搜索到属性中的内容，代码如下：

```python
>>> response.css('li.next a::attr(href)').extract_first()
'/page/2/'
```

让我们修改一下我们的代码，让她可以迭代到下一个网页，并从中提取数据：

```python
import scrapy


class QuotesSpider(scrapy.Spider):
    name = "quotes"
    start_urls = [
        'http://quotes.toscrape.com/page/1/',
    ]

    def parse(self, response):
        for quote in response.css('div.quote'):
            yield {
                'text': quote.css('span.text::text').extract_first(),
                'author': quote.css('small.author::text').extract_first(),
                'tags': quote.css('div.tags a.tag::text').extract(),
            }

        next_page = response.css('li.next a::attr(href)').extract_first()
        if next_page is not None:
            next_page = response.urljoin(next_page)
            yield scrapy.Request(next_page, callback=self.parse)
```

提取完数据之后， `parse()` 方法用来寻找下一页的链接，通过 [`urljoin()`](https://doc.scrapy.org/en/latest/topics/request-response.html#scrapy.http.Response.urljoin) 方法（因为链接都是相关的）就可以重新向下一页发送请求，callback寄存器自己就会自动处理提取出下一页链接的数据然后一直爬取所有网页。

这里就是告诉你Scrapy的迭代爬虫机制：当您在回调方法中产生请求时，Scrapy会安排发送请求并注册一个回调方法，以便在请求结束时执行。

使用这种方法，您可以根据您定义的规则构建复杂的抓取工具，并根据所访问的页面提取不同类型的数据。

在我们的例子中，它会创建一个循环，跟随下一页的所有链接，直到找不到用于分页搜索博客，论坛和其他网站的链接。

## 条条大路通请求(Requests方法)

如果你想用一个简单的方法来发送请求，你可以用 [`response.follow`](https://doc.scrapy.org/en/latest/topics/request-response.html#scrapy.http.TextResponse.follow):

```python
import scrapy


class QuotesSpider(scrapy.Spider):
    name = "quotes"
    start_urls = [
        'http://quotes.toscrape.com/page/1/',
    ]

    def parse(self, response):
        for quote in response.css('div.quote'):
            yield {
                'text': quote.css('span.text::text').extract_first(),
                'author': quote.css('span small::text').extract_first(),
                'tags': quote.css('div.tags a.tag::text').extract(),
            }

        next_page = response.css('li.next a::attr(href)').extract_first()
        if next_page is not None:
            yield response.follow(next_page, callback=self.parse)
```

不像 scrapy.Request， `response.follow` 同样支持相关网站的迭代请求，但是她不用调用urljoin

但是需要注意的就是 `response.follow`只是返回一个响应请求，你同样需要继续提取数据。

你也可以在`response.follow` 直接调用选择器中返回的内容，省去存取在另外一个变量的步骤。

```python
for href in response.css('li.next a::attr(href)'):
    yield response.follow(href, callback=self.parse)
```

对于 `<a>` 元素，有一个捷径： `response.follow`会自动用他们的href属性，所以代码可以简化成：

```python
for a in response.css('li.next a'):
    yield response.follow(a, callback=self.parse)
```

>**!Note**
>
>`response.follow(response.css('li.next a'))` 不是一个变量，因为 `response.css` 返回的是一个包含所有结果的选择器列表对象，并不是单一的选择器。一个在栗子中的 `for`，或者`response.follow(response.css('li.next a')[0])`就可以解决这个问题。 

## 另外一些栗子和爬虫模式

这是另外一个爬虫栗子来演示回调和迭代爬取，如下代码是爬取作者信息的：

```python
import scrapy


class AuthorSpider(scrapy.Spider):
    name = 'author'

    start_urls = ['http://quotes.toscrape.com/']

    def parse(self, response):
        # follow links to author pages
        for href in response.css('.author + a::attr(href)'):
            yield response.follow(href, self.parse_author)

        # follow pagination links
        for href in response.css('li.next a::attr(href)'):
            yield response.follow(href, self.parse)

    def parse_author(self, response):
        def extract_with_css(query):
            return response.css(query).extract_first().strip()

        yield {
            'name': extract_with_css('h3.author-title::text'),
            'birthdate': extract_with_css('.author-born-date::text'),
            'bio': extract_with_css('.author-description::text'),
        }
```

这个Spider会从主页开始，她会通过`parse_author` 回调来爬取所有作者的页面。就像我们之前用`parse` 来回调页码一样。

这里我们用`response.follow` 来代替 `scrapy.Request`，因为这样能让代码更短（其实二者都可以实现功能）。

`parse_author` 回调定义了一个辅助函数，可以从CSS查询中提取并清除一些无用的数据，并生成带有作者数据的字典。

这个蜘蛛演示的另一个有趣的事情是，即使有来自同一作者的许多引用，我们也不必担心多次访问相同的作者页面。默认情况下，Scrapy会将重复的请求过滤到已访问的URL中，避免因编程错误而导致服务器过多的问题。这可以通过设置进行配置 [`DUPEFILTER_CLASS`](https://doc.scrapy.org/en/latest/topics/settings.html#std:setting-DUPEFILTER_CLASS)。

希望现在，你已经很好地理解如何运用Scrapy的迭代爬取和回调机制。

还有另一个运用了迭代爬取机制的爬虫栗子：[`CrawlSpider`](https://doc.scrapy.org/en/latest/topics/spiders.html#scrapy.spiders.CrawlSpider) ，这是一个集成小规则引擎的通用爬虫，你可以在她的基础上，改写成一个自己的爬虫。

当然，人们都是从多个页面中爬取数据，然后整合到一个项目中处理的，可以尝试一下 [trick to pass additional data to the callbacks](https://doc.scrapy.org/en/latest/topics/request-response.html#topics-request-response-ref-request-callback-arguments).

## 如何运用爬虫一些命令行参数

在你跑你的spider时，你可以加上 `-a`选项：

```bash
scrapy crawl quotes -o quotes-humor.json -a tag=humor
```

这些参数会传递到Spider的 `__init__` 方法，然后成为spider的默认属性。

In this example, the value provided for the `tag` argument will be available via `self.tag`. You can use this to make your spider fetch only quotes with a specific tag, building the URL based on the argument:

```Python
import scrapy


class QuotesSpider(scrapy.Spider):
    name = "quotes"

    def start_requests(self):
        url = 'http://quotes.toscrape.com/'
        tag = getattr(self, 'tag', None)
        if tag is not None:
            url = url + 'tag/' + tag
        yield scrapy.Request(url, self.parse)

    def parse(self, response):
        for quote in response.css('div.quote'):
            yield {
                'text': quote.css('span.text::text').extract_first(),
                'author': quote.css('small.author::text').extract_first(),
            }

        next_page = response.css('li.next a::attr(href)').extract_first()
        if next_page is not None:
            yield response.follow(next_page, self.parse)
```



## 接下来要干什么？

这份指南仅仅介绍了Scrapy的一些基础知识，但是Scrapy还有其他这里没有提及的特色。阅读完  [Scrapy at a glance](https://doc.scrapy.org/en/latest/intro/overview.html#intro-overview)章节中的[What else?](https://doc.scrapy.org/en/latest/intro/overview.html#topics-whatelse) 小节，你就会对Scrapy最重要的特点有一个大概的认识。

你可以继续阅读[Basic concepts](https://doc.scrapy.org/en/latest/index.html#section-basics) 章节去了解更多关命令行工具，spider，选择器和其他这份指南没有讲全的知识入爬取的数据。如果你更喜欢通过栗子来学习，那 [Examples](https://doc.scrapy.org/en/latest/intro/examples.html#intro-examples) 章节会让你受益匪浅。