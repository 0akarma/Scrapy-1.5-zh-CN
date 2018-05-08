# Settings
Scrapy设定(settings)提供了定制Scrapy组件的方法。您可以控制包括核心(core)，插件(extension)，pipeline及spider组件。

设定为代码提供了提取以key-value映射的配置值的的全局命名空间(namespace)。 设定可以通过下面介绍的多种机制进行设置。

设定(settings)同时也是选择当前激活的Scrapy项目的方法(如果您有多个的话)。

内置设定列表请参考 内置设定参考手册 。

## 指定设定(Designating the settings)
当您使用Scrapy时，您需要声明您所使用的设定。这可以通过使用环境变量: SCRAPY_SETTINGS_MODULE 来完成。

SCRAPY_SETTINGS_MODULE 必须以Python路径语法编写, 如 myproject.settings 。 注意，设定模块应该在 Python import search path 中。

## 获取设定值(Populating the settings)

设定可以通过多种方式设置，每个方式具有不同的优先级。 下面以优先级降序的方式给出方式列表:

命令行选项(Command line Options)(最高优先级)
项目设定模块(Project settings module)
命令默认设定模块(Default settings per-command)
全局默认设定(Default global settings) (最低优先级)
这些设定(settings)由scrapy内部很好的进行了处理，不过您仍可以使用API调用来手动处理。 详情请参考 设置(Settings) API.

这些机制将在下面详细介绍。

### 1. 命令行选项(Command line options)
命令行传入的参数具有最高的优先级。 您可以使用command line 选项 -s (或 --set) 来覆盖一个(或更多)选项。


样例:
```
scrapy crawl myspider -s LOG_FILE=scrapy.log
```

### 2. 设置每个蜘蛛
蜘蛛（请参阅蜘蛛章节以供参考）可以定义它们自己的设置，这些设置将优先考虑并覆盖项目的设置。 他们可以通过设置他们的custom_settings属性来实现：
```
class MySpider(scrapy.Spider):
    name = 'myspider'

    custom_settings = {
        'SOME_SETTING': 'some value',
    }

```

### 3. 项目设定模块(Project settings module)
项目设置模块是Scrapy项目的标准配置文件，它是大多数自定义设置将被填充的地方。 对于标准Scrapy项目，这意味着您将添加或更改为项目创建的settings.py文件中的设置。

### 4. 命令默认设定(Default settings per-command)
每个 Scrapy tool 命令拥有其默认设定，并覆盖了全局默认的设定。 这些设定在命令的类的 default_settings 属性中指定。

### 5. 默认全局设定(Default global settings)
全局默认设定存储在 scrapy.settings.default_settings 模块， 并在 内置设定参考手册 部分有所记录。

## 如何访问设定(How to access settings)
在蜘蛛中，这些设置可以通过self.settings获得：
```
class MySpider(scrapy.Spider):
    name = 'myspider'
    start_urls = ['http://example.com']

    def parse(self, response):
        print("Existing settings: %s" % self.settings.attributes.keys())
```

>Note

设置属性在蜘蛛初始化后在基本Spider类中设置。 如果你想在初始化之前使用设置（例如，在你的蜘蛛的__init __()方法中），你需要重写 from_crawler() 方法

## 设定名字的命名规则
设定的名字以要配置的组件作为前缀。 例如，一个robots.txt插件的合适设定应该为 ROBOTSTXT_ENABLED, ROBOTSTXT_OBEY, ROBOTSTXT_CACHEDIR 等等。

## 内置设定参考手册
这里以字母序给出了所有可用的Scrapy设定及其默认值和应用范围。

如果给出可用范围，并绑定了特定的组件，则说明了该设定使用的地方。 这种情况下将给出该组件的模块，通常来说是插件、中间件或pipeline。 同时也意味着为了使设定生效，该组件必须被启用。

## AWS_ACCESS_KEY_ID
默认: None

连接 Amazon Web services 的AWS access key。 S3 feed storage backend 中使用.

## AWS_SECRET_ACCESS_KEY
默认: None

连接 Amazon Web services 的AWS secret key。 S3 feed storage backend 中使用。

## BOT_NAME
默认: 'scrapybot'

Scrapy项目实现的bot的名字(也未项目名称)。 这将用来构造默认 User-Agent，同时也用来log。

当您使用 startproject 命令创建项目时其也被自动赋值。

## CONCURRENT_ITEMS
默认: 100

Item Processor(即 Item Pipeline) 同时处理(每个response的)item的最大值。

## CONCURRENT_REQUESTS
默认: 16

Scrapy downloader 并发请求(concurrent requests)的最大值。

## CONCURRENT_REQUESTS_PER_DOMAIN
默认: 8

对单个网站进行并发请求的最大值。

## CONCURRENT_REQUESTS_PER_IP
默认: 0

对单个IP进行并发请求的最大值。如果非0，则忽略 CONCURRENT_REQUESTS_PER_DOMAIN 设定， 使用该设定。 也就是说，并发限制将针对IP，而不是网站。

该设定也影响 DOWNLOAD_DELAY: 如果 CONCURRENT_REQUESTS_PER_IP 非0，下载延迟应用在IP而不是网站上。

## DEFAULT_ITEM_CLASS
默认: 'scrapy.item.Item'

the Scrapy shell 中实例化item使用的默认类。

## DEFAULT_REQUEST_HEADERS
默认:
```

{
    'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8',
    'Accept-Language': 'en',
}
Scrapy HTTP Request使用的默认header。由 DefaultHeadersMiddleware 产生。

## DEPTH_LIMIT
默认: 0
```

爬取网站最大允许的深度(depth)值。如果为0，则没有限制。

## DEPTH_LIMIT
Default: 0
Scope: scrapy.spidermiddlewares.depth.DepthMiddleware
最大的 depth 被允许爬去任意网站。如果为零，则不会施加限制。

## DEPTH_PRIORITY
Default: 0

Scope: scrapy.spidermiddlewares.depth.DepthMiddleware
根据深度调整请求优先级的整数：

* 如果为零（默认），则不会从深度进行优先级调整
* 正值会降低优先级，即更高深度的请求将在稍后处理; 这是广泛进行抓取时常用的（BFO）
* 负值会增加优先级，即更高深度的请求将被更快地处理（DFO）

另请参见：Scrapy是否以广度优先或深度优先的顺序进行爬网？ 关于调整Scrapy为BFO或DFO。

>Note
与其他优先级设置REDIRECT_PRIORITY_ADJUST和RETRY_PRIORITY_ADJUST相比，此设置以相反的方式调整优先级。

## DEPTH_STATS

Default: True

Scope: scrapy.spidermiddlewares.depth.DepthMiddleware
是否收集最大的 DEPTH_STATS 。

## DEPTH_STATS_VERBOSE
Default: False

Scope: scrapy.spidermiddlewares.depth.DepthMiddleware
是否收集详细深度统计信息。 如果启用此功能，则会在统计信息中收集每个深度的请求数量。

## DNSCACHE_ENABLED
Default: True
是否启用DNS内存中缓存。

## DNSCACHE_SIZE
Default: 10000
DNS内存中缓存大小。

## DNS_TIMEOUT
Default: 60
以秒为单位处理DNS查询超时。 支持浮点型。

## DOWNLOADER
Default: 'scrapy.core.downloader.Downloader'

Defines a Twisted protocol.ClientFactory class to use for HTTP/1.0 connections (for HTTP10DownloadHandler).

>Note
HTTP/1.0 is rarely used nowadays so you can safely ignore this setting, unless you use Twisted<11.1, or if you really want to use HTTP/1.0 and override DOWNLOAD_HANDLERS_BASE for http(s) scheme accordingly, i.e. to 'scrapy.core.downloader.handlers.http.HTTP10DownloadHandler'.

## DOWNLOADER_CLIENTCONTEXTFACTORY
Default: 'scrapy.core.downloader.contextfactory.ScrapyClientContextFactory'
在这里，“ContextFactory”是SSL / TLS上下文的Twisted术语，定义要使用的TLS / SSL协议版本，是否执行证书验证，甚至启用客户端身份验证（以及其他各种各样的事情）。

>Note

Scrapy默认上下文工厂不执行远程服务器证书验证。 这通常适用于网页抓取。

如果您确实需要启用远程服务器证书验证，Scrapy还有另一个可以设置的上下文工厂类，'scrapy.core.downloader.contextfactory.BrowserLikeContextFactory'，它使用平台的证书来验证远程端点。 这仅在使用Twisted> = 14.0时可用。

如果您使用自定义ContextFactory，请确保它在init处接受方法参数（这是OpenSSL.SSL方法映射DOWNLOADER_CLIENT_TLS_METHOD）。

## DOWNLOADER_CLIENT_TLS_METHOD
Default: 'TLS'

使用此设置可以自定义默认的HTTP / 1.1下载器使用的TLS / SSL方法。该设置必须是以下字符串值之一：

* 'TLS': maps to OpenSSL’s TLS_method() (a.k.a SSLv23_method()), which allows protocol negotiation, starting from the highest supported by the platform; default, recommended
* 'TLSv1.0': this value forces HTTPS connections to use TLS version 1.0 ; set this if you want the behavior of Scrapy<1.1
* 'TLSv1.1': forces TLS version 1.1
* 'TLSv1.2': forces TLS version 1.2
* 'SSLv3': forces SSL version 3 (not recommended)

>Note
我们推荐你使用 PyOpenSSL>=0.13 and Twisted>=0.13 or above (Twisted>=14.0 if you can).

## DOWNLOADER_MIDDLEWARES
Default:: {}
一个包含您项目中启用的下载器中间件的字典及其他们的命令。 有关更多信息，请参阅[激活下载中间件](https://doc.scrapy.org/en/latest/topics/downloader-middleware.html#topics-downloader-middleware-setting)。

## DOWNLOADER_MIDDLEWARES_BASE
Default:
```
{
    'scrapy.downloadermiddlewares.robotstxt.RobotsTxtMiddleware': 100,
    'scrapy.downloadermiddlewares.httpauth.HttpAuthMiddleware': 300,
    'scrapy.downloadermiddlewares.downloadtimeout.DownloadTimeoutMiddleware': 350,
    'scrapy.downloadermiddlewares.defaultheaders.DefaultHeadersMiddleware': 400,
    'scrapy.downloadermiddlewares.useragent.UserAgentMiddleware': 500,
    'scrapy.downloadermiddlewares.retry.RetryMiddleware': 550,
    'scrapy.downloadermiddlewares.ajaxcrawl.AjaxCrawlMiddleware': 560,
    'scrapy.downloadermiddlewares.redirect.MetaRefreshMiddleware': 580,
    'scrapy.downloadermiddlewares.httpcompression.HttpCompressionMiddleware': 590,
    'scrapy.downloadermiddlewares.redirect.RedirectMiddleware': 600,
    'scrapy.downloadermiddlewares.cookies.CookiesMiddleware': 700,
    'scrapy.downloadermiddlewares.httpproxy.HttpProxyMiddleware': 750,
    'scrapy.downloadermiddlewares.stats.DownloaderStats': 850,
    'scrapy.downloadermiddlewares.httpcache.HttpCacheMiddleware': 900,
}
```

包含Scrapy中默认启用的下载器中间件的字典。 低订单靠近引擎，高订单靠近下载器。 您不应该在您的项目中修改此设置，而是修改DOWNLOADER_MIDDLEWARES。 有关更多信息，请参阅激活下载中间件。	


## DOWNLOADER_STATS
Default: True

是否收集下载器数据。

## DOWNLOAD_DELAY
Default: 0

下载器在下载同一个网站下一个页面前需要等待的时间。该选项可以用来限制爬取速度， 减轻服务器压力。同时也支持小数:

DOWNLOAD_DELAY = 0.25    # 250 ms of delay
该设定影响(默认启用的) RANDOMIZE_DOWNLOAD_DELAY 设定。 默认情况下，Scrapy在两个请求间不等待一个固定的值， 而是使用0.5到1.5之间的一个随机值 * DOWNLOAD_DELAY 的结果作为等待间隔。

当 CONCURRENT_REQUESTS_PER_IP 非0时，延迟针对的是每个ip而不是网站。

另外您可以通过spider的 download_delay 属性为每个spider设置该设定。

## DOWNLOAD_HANDLERS
默认: {}

保存项目中启用的下载处理器(request downloader handler)的字典。 例子请查看 DOWNLOAD_HANDLERS_BASE 。

## DOWNLOAD_HANDLERS_BASE
Default:
```
{
    'file': 'scrapy.core.downloader.handlers.file.FileDownloadHandler',
    'http': 'scrapy.core.downloader.handlers.http.HTTPDownloadHandler',
    'https': 'scrapy.core.downloader.handlers.http.HTTPDownloadHandler',
    's3': 'scrapy.core.downloader.handlers.s3.S3DownloadHandler',
    'ftp': 'scrapy.core.downloader.handlers.ftp.FTPDownloadHandler',
}
```

保存项目中默认启用的下载处理器(request downloader handler)的字典。 永远不要在项目中修改该设定，而是修改 DOWNLOADER_HANDLERS 。

您可以通过将DOWNLOAD分配给DOWNLOAD_HANDLERS中的URI方案来禁用任何这些下载处理程序。 例如，要禁用内置的FTP处理程序（无需替换），请将其置于settings.py中：

```
DOWNLOAD_HANDLERS = {
    'ftp': None,
}		

```

## DOWNLOAD_TIMEOUT
Default: 180

下载器超时时间(单位: 秒)。

>note
可以使用download_timeout spider属性和每个请求使用download_timeout Request.meta项设置每个蜘蛛的超时值。

## DOWNLOAD_MAXSIZE

Default: 1073741824 (1024MB)
下载器将下载的最大响应大小（以字节为单位）。

如果你想禁用它，设置为0。

>note
可以使用download_maxsize spider属性和每个请求使用download_maxsize Request.meta键为每个蜘蛛设置此大小。
此功能需要Twisted> = 11.1。

## DOWNLOAD_WARNSIZE
Default: 33554432 (32MB)

下载器将开始发出警告的响应大小（以字节为单位）。

如果你想禁用它，设置为0。

>note
可以使用download_warnsize spider属性和每个请求使用download_warnsize Request.meta键为每个蜘蛛设置此大小。
此功能需要Twisted> = 11.1。

## DOWNLOAD_FAIL_ON_DATALOSS
Default: True
无论是否在破坏的响应上失败，即声明的Content-Length与服务器发送的内容不匹配，或者分块的响应没有正确完成。 如果为True，这些响应会引发ResponseFailed（[_DataLoss]）错误。 如果为False，则通过这些响应并将标志数据库添加到响应中，即：response.flags中的'dataloss'为True。

或者，可以通过使用download_fail_on_dataloss Request.meta键为False来设置每个请求的基础。

>note
在几种情况下，从服务器配置错误，网络错误到数据损坏，都可能发生错误的响应或数据丢失错误。 用户可以决定是否有理由处理破坏的响应，因为它们可能包含部分或不完整的内容。 如果RETRY_ENABLED为True并且此设置设置为True，则ResponseFailed（[_ DataLoss]）失败将照常重试。

## DUPEFILTER_CLASS
Default: 'scrapy.dupefilters.RFPDupeFilter'
该类用于检测和过滤重复的请求。

基于请求指纹的默认（RFPDupeFilter）过滤器使用scrapy.utils.request.request_fingerprint函数。 为了改变重复检查的方式，你可以继承RFPDupeFilter并覆盖它的request_fingerprint方法。 该方法应接受scrapy Request对象并返回其指纹（字符串）。

您可以通过将DUPEFILTER_CLASS设置为'scrapy.dupefilters.BaseDupeFilter'来禁用对重复请求的过滤。 不过要小心，因为你可以进入爬行循环。 在不应过滤的特定请求上将dont_filter参数设置为True通常是一个更好的主意。

## DUPEFILTER_DEBUG
Default: False
默认情况下，RFPDupeFilter只记录第一个重复请求。 将DUPEFILTER_DEBUG设置为True将使其记录所有重复的请求。

## EDITOR
Default: vi (on Unix systems) or the IDLE editor (on Windows)

编辑器用于使用编辑命令编辑蜘蛛。 此外，如果EDITOR环境变量已设置，则编辑命令将优先于默认设置。

## EXTENSIONS
Default:: {}
一个包含在您的项目中启用的扩展的字典，以及它们的命令。

## EXTENSIONS_BASE
Default:
```
{
    'scrapy.extensions.corestats.CoreStats': 0,
    'scrapy.extensions.telnet.TelnetConsole': 0,
    'scrapy.extensions.memusage.MemoryUsage': 0,
    'scrapy.extensions.memdebug.MemoryDebugger': 0,
    'scrapy.extensions.closespider.CloseSpider': 0,
    'scrapy.extensions.feedexport.FeedExporter': 0,
    'scrapy.extensions.logstats.LogStats': 0,
    'scrapy.extensions.spiderstate.SpiderState': 0,
    'scrapy.extensions.throttle.AutoThrottle': 0,
}
```
包含Scrapy中默认可用扩展名的字典及其订单。 该设置包含所有稳定的内置扩展。 请记住，其中一些需要通过设置启用。

有关更多信息，请参阅附加用户指南和可用扩展的列表。

## FEED_TEMPDIR

Feed Temp目录允许您设置一个自定义文件夹，以便在使用FTP提要存储和Amazon S3上传之前保存抓取工具临时文件。


## FTP_PASSIVE_MODE
Default:True
启动FTP传输时是否使用被动模式。

## FTP_PASSWORD
Default：“guest”

请求元中没有“ftp_password”时用于FTP连接的密码。
>Note
解释RFC 1635，虽然通常使用匿名FTP的密码“guest”或电子邮件地址，但某些FTP服务器明确要求用户的电子邮件地址，并且不允许使用“guest”密码登录。

## FTP_USER
Default: "anonymous"
当请求meta中没有“ftp_user”时用于FTP连接的用户名。

## ITEM_PIPELINES
Default: {}
包含要使用的物品管道的字典及其订单。 顺序值是任意的，但习惯上将它们定义在0-1000范围内。 在下单之前处理下单。
Example:

```ITEM_PIPELINES = {
    'mybot.pipelines.validate.ValidateMyItem': 300,
    'mybot.pipelines.validate.StoreMyItem': 800,
}
```

## ITEM_PIPELINES_BASE
Default: {}
包含Scrapy中默认启用的管道的字典。 您不应该在您的项目中修改此设置，而是修改ITEM_PIPELINES。

## LOG_ENABLED
默认: True

是否启用logging

## LOG_ENCODING
默认: 'utf-8'

logging使用的编码。

## LOG_FILE
默认: None

logging输出的文件名。如果为None，则使用标准错误输出(standard error)。

## LOG_FORMAT
Default: '%(asctime)s [%(name)s] %(levelname)s: %(message)s'

用于格式化日志消息的字符串。 有关可用占位符的整个列表，请参阅Python日志记录文档。

## LOG_DATEFORMAT
Default: '%Y-%m-%d %H:%M:%S'

用于格式化日期/时间的字符串，在LOG_FORMAT中扩展％（asctime）的占位符。 有关可用指令的整个列表，请参阅Python日期时间文档。

## LOG_LEVEL
Default: 'DEBUG'

log的最低级别。可选的级别有: CRITICAL、 ERROR、WARNING、INFO、DEBUG。更多内容请查看 Logging 。

## LOG_STDOUT
默认: False

如果为 True ，进程所有的标准输出(及错误)将会被重定向到log中。例如， 执行 print 'hello' ，其将会在Scrapy log中显示。

## LOG_SHORT_NAMES
Default: False

如果为True，则日志将只包含根路径。 如果它设置为False，则它显示负责日志输出的组件

## MEMDEBUG_ENABLED
默认: False

是否启用内存调试(memory debugging)。
## MEMDEBUG_NOTIFY
默认: []

如果该设置不为空，当启用内存调试时将会发送一份内存报告到指定的地址；否则该报告将写到log中。

样例:

>MEMDEBUG_NOTIFY = ['user@example.com']

## MEMUSAGE_ENABLED
默认: False

Scope: scrapy.contrib.memusage

是否启用内存使用插件。当Scrapy进程占用的内存超出限制时，该插件将会关闭Scrapy进程， 同时发送email进行通知。

See 内存使用扩展(Memory usage extension).

## MEMUSAGE_LIMIT_MB
默认: 0

Scope: scrapy.contrib.memusage

在关闭Scrapy之前所允许的最大内存数(单位: MB)(如果 MEMUSAGE_ENABLED为True)。 如果为0，将不做限制。

See 内存使用扩展(Memory usage extension).

## MEMUSAGE_CHECK_INTERVAL_SECONDS
版本1.1中的新功能

默认值：60.0

范围：scrapy.extensions.memusage

内存使用扩展以固定的时间间隔检查当前的内存使用情况，而不是MEMUSAGE_LIMIT_MB和MEMUSAGE_WARNING_MB设置的限制。

这设置这些间隔的长度，以秒为单位。

请参阅内存使用扩展。

## MEMUSAGE_NOTIFY_MAIL
默认: False

Scope: scrapy.contrib.memusage

达到内存限制时通知的email列表。

Example:

>MEMUSAGE_NOTIFY_MAIL = ['user@example.com']

## MEMUSAGE_WARNING_MB
默认: 0

Scope: scrapy.contrib.memusage

在发送警告email前所允许的最大内存数(单位: MB)(如果 MEMUSAGE_ENABLED为True)。 如果为0，将不发送警告。

## NEWSPIDER_MODULE
默认: ''

使用 genspider 命令创建新spider的模块。

样例:

NEWSPIDER_MODULE = 'mybot.spiders_dev'

## RANDOMIZE_DOWNLOAD_DELAY
默认: True

如果启用，当从相同的网站获取数据时，Scrapy将会等待一个随机的值 (0.5到1.5之间的一个随机值 * DOWNLOAD_DELAY)。

该随机值降低了crawler被检测到(接着被block)的机会。某些网站会分析请求， 查找请求之间时间的相似性。

随机的策略与 wget --random-wait 选项的策略相同。

若 DOWNLOAD_DELAY 为0(默认值)，该选项将不起作用。

## REACTOR_THREADPOOL_MAXSIZE
默认值：10

Twisted Reactor线程池大小的最大限制。 这是各种Scrapy组件使用的常见多用途线程池。 Threaded DNS Resolver，BlockingFeedStorage，S3FilesStore等等。 如果您遇到阻塞IO不足的问题，请增加此值。

## REDIRECT_MAX_TIMES
默认: 20

定义request允许重定向的最大次数。超过该限制后该request直接返回获取到的结果。 对某些任务我们使用Firefox默认值。

## REDIRECT_PRIORITY_ADJUST
默认值：+2

范围：scrapy.downloadermiddlewares.redirect.RedirectMiddleware

调整重定向请求相对于原始请求的优先级：

积极的优先级调整（默认）意味着更高的优先级。
否定优先级调整意味着优先级较低。

## RETRY_PRIORITY_ADJUST
	默认值：-1

范围：scrapy.downloadermiddlewares.retry.RetryMiddleware

调整相对于原始请求的重试请求优先级：

积极的优先调整意味着更高的优先。
负优先级调整（默认）意味着较低的优先级。

## ROBOTSTXT_OBEY
默认值：False

范围：scrapy.downloadermiddlewares.robotstxt

如果启用，Scrapy将尊重robots.txt策略。 欲了解更多信息，请参阅RobotsTxtMiddleware。

>注意
虽然由于历史原因默认值为False，但此选项在scrapy startproject命令生成的settings.py文件中默认启用。

## SCHEDULER
默认值：'scrapy.core.scheduler.Scheduler'

调度程序用于抓取。

## SCHEDULER_DEBUG
默认值：False

设置为True将记录有关请求调度程序的调试信息。 如果请求无法序列化到磁盘，则此操作当前会记录（仅一次）。 统计计数器（调度程序/不可序列化）跟踪发生这种情况的次数。

日志中的示例条目：
>1956-01-31 00:00:00+0800 [scrapy.core.scheduler] ERROR: Unable to serialize request:
<GET http://example.com> - reason: cannot serialize <Request at 0x9a7c7ec>
(type Request)> - no more unserializable requests will be logged
(see 'scheduler/unserializable' stats counter)

## SCHEDULER_DISK_QUEUE
默认：'scrapy.squeues.PickleLifoDiskQueue'

调度程序将使用的磁盘队列的类型。 其他可用的类型是scrapy.squeues.PickleFifoDiskQueue，scrapy.squeues.MarshalFifoDiskQueue，scrapy.squeues.MarshalLifoDiskQueue。

## SCHEDULER_MEMORY_QUEUE
默认：'scrapy.squeues.LifoMemoryQueue'

调度程序使用的内存中队列的类型。 其他可用的类型是：scrapy.squeues.FifoMemoryQueue。

## SCHEDULER_PRIORITY_QUEUE
默认值：'queuelib.PriorityQueue'

调度程序使用的优先级队列的类型。

## SPIDER_CONTRACTS
默认:: {}

一个包含您的项目中启用的蜘蛛协议的字典，用于测试蜘蛛。 欲了解更多信息，请参阅蜘蛛合同。

## SPIDER_CONTRACTS_BASE
Default:
```
{
    'scrapy.contracts.default.UrlContract' : 1,
    'scrapy.contracts.default.ReturnsContract': 2,
    'scrapy.contracts.default.ScrapesContract': 3,
}```

包含Scrapy默认启用的scrapy合约的字典。 您不应该在您的项目中修改此设置，而是修改SPIDER_CONTRACTS。 欲了解更多信息，请参阅蜘蛛合同。

您可以通过将SPIDER_CONTRACTS中的类路径分配为无效来禁用任何这些合同。 例如，要禁用内置的ScrapesContract，请将其置于settings.py中：
```
SPIDER_CONTRACTS = {
    'scrapy.contracts.default.ScrapesContract': None,
}
```

## SPIDER_LOADER_CLASS
默认：'scrapy.spiderloader.SpiderLoader'

将用于加载蜘蛛的类，它必须实现SpiderLoader API。

## SPIDER_LOADER_WARN_ONLY
1.3.3版本中的新功能。

默认值：False

默认情况下，当scrapy尝试从SPIDER_MODULES导入spider类时，如果存在任何ImportError异常，它将大声失败。 但是，您可以通过设置SPIDER_LOADER_WARN_ONLY = True来选择将此异常消除并将其变为简单警告。
>注意
一些scrapy命令已经将此设置运行为True（即它们只会发出警告并且不会失败），因为它们实际上并不需要加载spider类才能工作：scrapy runspider，scrapy设置，scrapy startproject，scrapy版本。

## SPIDER_MIDDLEWARES
默认:: {}

一个包含您的项目中启用的蜘蛛中间件的字典，以及他们的订单。 有关更多信息，请参阅激活蜘蛛中间件。

## SPIDER_MIDDLEWARES_BASE
Default:
```
{
    'scrapy.spidermiddlewares.httperror.HttpErrorMiddleware': 50,
    'scrapy.spidermiddlewares.offsite.OffsiteMiddleware': 500,
    'scrapy.spidermiddlewares.referer.RefererMiddleware': 700,
    'scrapy.spidermiddlewares.urllength.UrlLengthMiddleware': 800,
    'scrapy.spidermiddlewares.depth.DepthMiddleware': 900,
}
```
包含Scrapy中默认启用的蜘蛛中间件的字典及其订单。 低订单靠近引擎，高订单靠近蜘蛛。 有关更多信息，请参阅激活蜘蛛中间件。

## SPIDER_MODULES
默认值：[]

Scrapy将寻找蜘蛛的模块列表。

例：

>SPIDER_MODULES = ['mybot.spiders_prod'，'mybot.spiders_dev']

## STATS_CLASS
默认：'scrapy.statscollectors.MemoryStatsCollector'

用于收集统计信息的类，他们必须实现Stats Collector API。

## STATS_DUMP
默认值：True

一旦蜘蛛完成，将Scrapy统计信息（Scrapy日志）转储。

欲了解更多信息，请参阅：Stats Collection。

## STATSMAILER_RCPTS
默认值：[]（空列表）

在蜘蛛完成刮取后发送Scrapy数据。 请参阅StatsMailer了解更多信息。

## TELNETCONSOLE_ENABLED

默认: True

表明 telnet 终端 (及其插件)是否启用的布尔值。

## TELNETCONSOLE_PORT
默认: [6023, 6073]

telnet终端使用的端口范围。如果设置为 None 或 0 ， 则使用动态分配的端口。更多内容请查看 Telnet终端(Telnet Console) 。

## TEMPLATES_DIR
默认: scrapy模块内部的 templates

使用 startproject 命令创建项目时查找模板的目录。

## URLLENGTH_LIMIT
默认: 2083

Scope: contrib.spidermiddleware.urllength

爬取URL的最大长度。更多关于该设定的默认值信息请查看: http://www.boutell.com/newfaq/misc/urllength.html

## USER_AGENT
默认: "Scrapy/VERSION (+http://scrapy.org)"

爬取的默认User-Agent，除非被覆盖。

## Settings documented elsewhere:
以下设置在其他地方有记录，请检查每个特定情况以了解如何启用和使用它们。
[AJAXCRAWL_ENABLED]()
AUTOTHROTTLE_DEBUG
AUTOTHROTTLE_ENABLED
AUTOTHROTTLE_MAX_DELAY
AUTOTHROTTLE_START_DELAY
AUTOTHROTTLE_TARGET_CONCURRENCY
CLOSESPIDER_ERRORCOUNT
CLOSESPIDER_ITEMCOUNT
CLOSESPIDER_PAGECOUNT
CLOSESPIDER_TIMEOUT
COMMANDS_MODULE
COMPRESSION_ENABLED
COOKIES_DEBUG
COOKIES_ENABLED
FEED_EXPORTERS
FEED_EXPORTERS_BASE
FEED_EXPORT_ENCODING
FEED_EXPORT_FIELDS
FEED_EXPORT_INDENT
FEED_FORMAT
FEED_STORAGES
FEED_STORAGES_BASE
FEED_STORE_EMPTY
FEED_URI
FILES_EXPIRES
FILES_RESULT_FIELD
FILES_STORE
FILES_STORE_S3_ACL
FILES_URLS_FIELD
GCS_PROJECT_ID
HTTPCACHE_ALWAYS_STORE
HTTPCACHE_DBM_MODULE
HTTPCACHE_DIR
HTTPCACHE_ENABLED
HTTPCACHE_EXPIRATION_SECS
HTTPCACHE_GZIP
HTTPCACHE_IGNORE_HTTP_CODES
HTTPCACHE_IGNORE_MISSING
HTTPCACHE_IGNORE_RESPONSE_CACHE_CONTROLS
HTTPCACHE_IGNORE_SCHEMES
HTTPCACHE_POLICY
HTTPCACHE_STORAGE
HTTPERROR_ALLOWED_CODES
HTTPERROR_ALLOW_ALL
HTTPPROXY_AUTH_ENCODING
HTTPPROXY_ENABLED
IMAGES_EXPIRES
IMAGES_MIN_HEIGHT
IMAGES_MIN_WIDTH
IMAGES_RESULT_FIELD
IMAGES_STORE
IMAGES_STORE_S3_ACL
IMAGES_THUMBS
IMAGES_URLS_FIELD
MAIL_FROM
MAIL_HOST
MAIL_PASS
MAIL_PORT
MAIL_SSL
MAIL_TLS
MAIL_USER
MEDIA_ALLOW_REDIRECTS
METAREFRESH_ENABLED
METAREFRESH_MAXDELAY
REDIRECT_ENABLED
REDIRECT_MAX_TIMES
REFERER_ENABLED
REFERRER_POLICY
RETRY_ENABLED
RETRY_HTTP_CODES
RETRY_TIMES
TELNETCONSOLE_HOST
TELNETCONSOLE_PORT