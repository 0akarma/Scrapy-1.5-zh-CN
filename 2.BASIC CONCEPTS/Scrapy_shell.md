#Scrapy外壳
Scrapy shell是一个交互式shell，您可以非常快速地尝试并调试您的scraping代码，而无需运行蜘蛛。它旨在用于测试数据提取代码，但实际上它可以用于测试任何类型的代码，因为它也是一个常规的Python shell。

该shell用于测试XPath或CSS表达式，并查看它们的工作方式以及它们从您尝试抓取的网页中提取的数据。它可以让你在写蜘蛛的时候交互地测试你的表达，而不必运行蜘蛛来测试每一个变化。

一旦熟悉Scrapy shell，您会发现它是开发和调试您的蜘蛛的宝贵工具。
##配置shell
如果你安装了IPython，Scrapy shell将会使用它（而不是标准的Python控制台）。该IPython的控制台功能更强大，并提供智能自动完成和彩色输出，等等。

我们强烈建议您安装IPython，特别是如果您在Unix系统上工作（IPython擅长）。有关 更多信息，请参阅IPython安装指南。

Scrapy也支持bpython，并会尝试在IPython 不可用的情况下使用它。

通过scrapy的设置，您可以配置为使用中的任何一个 `ipython`，`bpython`或标准的`python`外壳，安装不管是哪个。这是通过设置`SCRAPY_PYTHON_SHELL`环境变量来完成的; 或者通过在`scrapy.cfg`中定义它：

	[settings]
	shell = bpython
##启动外壳
要启动Scrapy shell，您可以使用如下`shell`命令：

	scrapy  shell  < url >
`<url>`是你想要刮的URL。

`shell`也适用于本地文件。如果你想玩一个网页的本地副本，这可以很方便。`shell`了解本地文件的以下语法：
	
	# UNIX-style
	scrapy shell ./path/to/file.html
	scrapy shell ../other/path/to/file.html
	scrapy shell /absolute/path/to/file.html
	# File URI
	scrapy shell file:///absolute/path/to/file.html
>注意
>
在使用相对文件路径时，应明确并预置`./`（或`../`）在相关时。 `scrapy shell index.html`不会像人们所期望的那样工作（这是通过设计而非错误）。由于`shell`支持文件URI上的HTTP URL，并且`index.html`在语法上类似`example.com`， `shell`因此将`index.html`视为域名并触发DNS查找错误：

	$ scrapy shell index.html
	[ ... scrapy shell starts ... ]
	[ ... traceback ... ]
	twisted.internet.error.DNSLookupError: DNS lookup failed:
	address 'index.html' not found: [Errno -5] No address associated with hostname.
>如果当前目录中存在调用的文件`index.html` ，则shell不会事先进行测试。再次，请明确。
##使用shell
Scrapy shell只是一个普通的Python控制台（如果有的话，也可以是`IPython`控制台），它提供了一些额外的便捷功能。
###可用的快捷方式
- `shelp()` - 用可用对象和快捷方式的列表打印帮助
- `fetch(url[, redirect=True])` - 从给定的URL获取新的响应并相应地更新所有相关的对象。您可以选择要求HTTP 3xx重定向，以免传递`redirect=False`
- `fetch(request)` - 从给定的请求中获取新的响应，并相应地更新所有相关的对象。
- `view(response)` - 在本地网络浏览器中打开给定的响应，以供检查。这将在响应主体中添加一个<base>标记，以便正确显示外部链接（如图像和样式表）。但请注意，这将在您的计算机中创建一个临时文件，该文件不会被自动删除。
###可用的Scrapy对象
Scrapy外壳会自动从下载的页面创建一些便利的对象，如`Response`对象和 `Selector`对象（用于HTML和XML内容）。

这些对象是：

- `crawler`- 当前`Crawler`对象。
- `spider`- 已知处理URL的Spider，或者`Spider`当前URL中没有发现蜘蛛的 对象
- `request` -`Request`最后一次抓取页面的对象。您可以使用`replace()` 或通过使用`fetch` 快捷方式获取新请求（不离开shell）来修改此请求。
- `response` -`Response`包含上次获取的页面的对象
- `settings` -当前的Scrapy设置
##shell会话的例子
下面是一个典型的shell会话示例，我们首先抓取[https://scrapy.org]()页面，然后继续刮取[https://reddit.com]() 页面。最后，我们将（Reddit）请求方法修改为POST并重新获取错误。我们通过在Windows中键入Ctrl-D（在Unix系统中）或Ctrl-Z来结束会话。

请记住，这里提取的数据在尝试时可能不尽相同，因为这些页面不是静态的，并且在测试时可能会发生变化。这个例子的唯一目的是让你熟悉Scrapy shell的工作原理。

首先，我们启动shell：

	scrapy shell 'https://scrapy.org' --nolog
然后，shell获取URL（使用Scrapy下载器）并打印可用对象列表和有用的快捷方式（您会注意到这些行都以`[s]`前缀开头）：
	
	[s] Available Scrapy objects:
	[s]   scrapy     scrapy module (contains scrapy.Request, scrapy.Selector, etc)
	[s]   crawler    <scrapy.crawler.Crawler object at 0x7f07395dd690>
	[s]   item       {}
	[s]   request    <GET https://scrapy.org>
	[s]   response   <200 https://scrapy.org/>
	[s]   settings   <scrapy.settings.Settings object at 0x7f07395dd710>
	[s]   spider     <DefaultSpider 'default' at 0x7f0735891690>
	[s] Useful shortcuts:
	[s]   fetch(url[, redirect=True]) Fetch URL and update local objects (by default, redirects are followed)
	[s]   fetch(req)                  Fetch a scrapy.Request and update local objects
	[s]   shelp()           Shell help (print this help)[s]   view(response)    View response in a browser
	>>>
之后，我们可以开始玩对象：

	>>> response.xpath('//title/text()').extract_first()
	'Scrapy | A Fast and Powerful Scraping and Web Crawling Framework'
	>>> fetch("https://reddit.com")
	>>> response.xpath('//title/text()').extract()
	['reddit: the front page of the internet']
	>>> request = request.replace(method="POST")
	>>> fetch(request)
	>>> response.status404
	>>> from pprint import pprint
	>>> pprint(response.headers)
	{'Accept-Ranges': ['bytes'], 
	'Cache-Control': ['max-age=0, must-revalidate'],
	 'Content-Type': ['text/html; charset=UTF-8'], 
	'Date': ['Thu, 08 Dec 2016 16:21:19 GMT'], 
	'Server': ['snooserv'], 
	'Set-Cookie':
		 ['loid=KqNLou0V9SKMX4qb4n; Domain=reddit.com; Max-Age=63071999; Path=/; expires=Sat, 08-Dec-2018 16:21:19 GMT; secure',                
		'loidcreated=2016-12-08T16%3A21%3A19.445Z; Domain=reddit.com; Max-Age=63071999; Path=/; expires=Sat, 08-Dec-2018 16:21:19 GMT; secure',                
		'loid=vi0ZVe4NkxNWdlH7r7; Domain=reddit.com; Max-Age=63071999; Path=/; expires=Sat, 08-Dec-2018 16:21:19 GMT; secure',                
		'loidcreated=2016-12-08T16%3A21%3A19.459Z; Domain=reddit.com; Max-Age=63071999; Path=/; expires=Sat, 08-Dec-2018 16:21:19 GMT; secure'], 
	'Vary': ['accept-encoding'],
	 'Via': ['1.1 varnish'], 
	'X-Cache': ['MISS'], 'X-Cache-Hits': ['0'], 
	'X-Content-Type-Options': ['nosniff'], 
	'X-Frame-Options': ['SAMEORIGIN'],
	 'X-Moose': ['majestic'], 
	'X-Served-By': ['cache-cdg8730-CDG'], 
	'X-Timer': ['S1481214079.394283,VS0,VE159'], 'X-Ua-Compatible': ['IE=edge'], 
	'X-Xss-Protection': ['1; mode=block']}
	>>>
##从蜘蛛中调用外壳来检查响应
有时候你想检查一下蜘蛛某一点正在处理的反应，如果只是为了检查你期望的反应是否到达那里。
这可以通过使用该`scrapy.shell.inspect_response`功能来实现。
以下是您如何从您的蜘蛛中调用它的示例：
	
	import scrapy
	
	class MySpider(scrapy.Spider):
	    name = "myspider"
	    start_urls = [
	        "http://example.com",
	        "http://example.org",
	        "http://example.net",
	    ]
	
	    def parse(self, response):
	        # We want to inspect one specific response.
	        if ".org" in response.url:
	            from scrapy.shell import inspect_response
	            inspect_response(response, self)
	
	        # Rest of parsing code.
当你运行蜘蛛时，你会得到类似于这样的东西：
	
	2014-01-23 17:48:31-0400 [scrapy.core.engine] DEBUG: Crawled (200) <GET http://example.com> (referer: None)
	2014-01-23 17:48:31-0400 [scrapy.core.engine] DEBUG: Crawled (200) <GET http://example.org> (referer: None)
	[s] Available Scrapy objects:
	[s]   crawler    <scrapy.crawler.Crawler object at 0x1e16b50>
	...
	>>> response.url
	'http://example.org'
然后，你可以检查提取代码是否工作：

	>>> response.xpath('//h1[@class="fn"]')[]
不，它没有。因此，您可以在Web浏览器中打开响应，看看它是否是您期望的响应：

	>>> view(response)
	True
最后，您按Ctrl-D（或Windows中的Ctrl-Z）以退出shell并恢复爬网：
	
	>>> ^D
	2014-01-23 17:50:03-0400 [scrapy.core.engine] DEBUG: Crawled (200) <GET http://example.net> (referer: None)
	...
请注意，`fetch`由于Scrapy引擎被shell阻塞，因此您无法使用此快捷方式。但是，在离开壳后，蜘蛛将继续爬行，如上所示。
