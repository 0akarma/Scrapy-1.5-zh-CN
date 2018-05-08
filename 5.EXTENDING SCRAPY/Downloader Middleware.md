下载器中间件是Scrapy的请求/响应处理的钩子框架。这是一个轻微的低级系统，用于全球改变Scrapy的请求和响应。

### 激活一个下载中间件

要激活下载器中间件组件，请将其添加到 [`DOWNLOADER_MIDDLEWARES`](https://doc.scrapy.org/en/latest/topics/settings.html#std:setting-DOWNLOADER_MIDDLEWARES)设置中，该设置是键是中间件类路径的字典，它们的值是中间件订单。

这是一个例子：

```python
DOWNLOADER_MIDDLEWARES  =  { 
    'myproject.middlewares.CustomDownloaderMiddleware' ： 543 ，
}
```

该[`DOWNLOADER_MIDDLEWARES`](https://doc.scrapy.org/en/latest/topics/settings.html#std:setting-DOWNLOADER_MIDDLEWARES)设置与[`DOWNLOADER_MIDDLEWARES_BASE`](https://doc.scrapy.org/en/latest/topics/settings.html#std:setting-DOWNLOADER_MIDDLEWARES_BASE)Scrapy中定义的设置合并 （并不意味着被覆盖），然后按顺序排序以获得最终的已启用中间件排序列表：第一个中间件是靠近引擎的中间件，最后一个是靠近引擎的中间件到下载器。换句话说，[`process_request()`](https://doc.scrapy.org/en/latest/topics/downloader-middleware.html#scrapy.downloadermiddlewares.DownloaderMiddleware.process_request) 每个中间件的方法将以增加中间件的顺序（100,200,300，...）[`process_response()`](https://doc.scrapy.org/en/latest/topics/downloader-middleware.html#scrapy.downloadermiddlewares.DownloaderMiddleware.process_response)被调用，并且每个中间件的方法将按降序调用。

要决定分配给中间件的顺序，请参阅 [`DOWNLOADER_MIDDLEWARES_BASE`](https://doc.scrapy.org/en/latest/topics/settings.html#std:setting-DOWNLOADER_MIDDLEWARES_BASE)设置并根据要插入中间件的位置选择一个值。顺序很重要，因为每个中间件都执行不同的操作，而您的中间件可能依赖于某些以前（或后续）正在应用的中间件。

如果要禁用内置中间件（[`DOWNLOADER_MIDDLEWARES_BASE`](https://doc.scrapy.org/en/latest/topics/settings.html#std:setting-DOWNLOADER_MIDDLEWARES_BASE)默认情况下定义和启用的中间件 ），则必须在项目[`DOWNLOADER_MIDDLEWARES`](https://doc.scrapy.org/en/latest/topics/settings.html#std:setting-DOWNLOADER_MIDDLEWARES)设置 中将其定义并将None指定为其值。例如，如果您想禁用用户代理中间件：

```python
DOWNLOADER_MIDDLEWARES  =  { 
    'myproject.middlewares.CustomDownloaderMiddleware' ： 543 ，
    'scrapy.downloadermiddlewares.useragent.UserAgentMiddleware' ： None ，
}
```

最后，请记住，某些中间件可能需要通过特定设置启用。有关更多信息，请参阅每个中间件文档。

### 编写你自己的下载中间件

每个中间件组件都是一个Python类，它定义了以下一种或多种方法：

*class*`scrapy.downloadermiddlewares.``DownloaderMiddleware`

>**！注意**
>
>任何下载器中间件方法也可能返回延迟。



>`process_request`（*请求*，*蜘蛛*）
>
>每个通过下载中间件的请求都会调用此方法。
>
>[`process_request()`](https://doc.scrapy.org/en/latest/topics/downloader-middleware.html#scrapy.downloadermiddlewares.DownloaderMiddleware.process_request)应该：返回`None`，返回一个 [`Response`](https://doc.scrapy.org/en/latest/topics/request-response.html#scrapy.http.Response)对象，返回一个[`Request`](https://doc.scrapy.org/en/latest/topics/request-response.html#scrapy.http.Request) 对象，或者提高[`IgnoreRequest`](https://doc.scrapy.org/en/latest/topics/exceptions.html#scrapy.exceptions.IgnoreRequest)。
>
>如果它返回`None`，Scrapy将继续处理此请求，执行所有其他中间件，直到最后，合适的下载器处理程序被称为执行的请求（及其响应下载）。
>
>如果它返回一个[`Response`](https://doc.scrapy.org/en/latest/topics/request-response.html#scrapy.http.Response)对象，Scrapy不会打扰调用*任何*其他[`process_request()`](https://doc.scrapy.org/en/latest/topics/downloader-middleware.html#scrapy.downloadermiddlewares.DownloaderMiddleware.process_request)或[`process_exception()`](https://doc.scrapy.org/en/latest/topics/downloader-middleware.html#scrapy.downloadermiddlewares.DownloaderMiddleware.process_exception)方法，或相应的下载功能; 它会返回该响应。[`process_response()`](https://doc.scrapy.org/en/latest/topics/downloader-middleware.html#scrapy.downloadermiddlewares.DownloaderMiddleware.process_response) 安装中间件的方法始终在每个响应中调用。
>
>如果它返回一个[`Request`](https://doc.scrapy.org/en/latest/topics/request-response.html#scrapy.http.Request)对象，Scrapy将停止调用process_request方法并重新安排返回的请求。一旦执行新返回的请求，将在下载的响应中调用适当的中间件链。
>
>如果引发[`IgnoreRequest`](https://doc.scrapy.org/en/latest/topics/exceptions.html#scrapy.exceptions.IgnoreRequest)异常，[`process_exception()`](https://doc.scrapy.org/en/latest/topics/downloader-middleware.html#scrapy.downloadermiddlewares.DownloaderMiddleware.process_exception)则会调用已安装的下载器中间件的 方法。如果它们都不处理异常，`Request.errback`则调用request（）的errback函数。如果没有代码处理引发的异常，它将被忽略并且不记录（不像其他异常）。
>
>参数：
>
>- **请求**（[`Request`](https://doc.scrapy.org/en/latest/topics/request-response.html#scrapy.http.Request)对象） - 正在处理的请求
>- **蜘蛛**（[`Spider`](https://doc.scrapy.org/en/latest/topics/spiders.html#scrapy.spiders.Spider)物体） - 这个请求所针对的蜘蛛
>
>
>
>`process_response`（*请求*，*响应*，*蜘蛛*）
>
>[`process_response()`](https://doc.scrapy.org/en/latest/topics/downloader-middleware.html#scrapy.downloadermiddlewares.DownloaderMiddleware.process_response)应该：返回一个[`Response`](https://doc.scrapy.org/en/latest/topics/request-response.html#scrapy.http.Response) 对象，返回一个[`Request`](https://doc.scrapy.org/en/latest/topics/request-response.html#scrapy.http.Request)对象或引发[`IgnoreRequest`](https://doc.scrapy.org/en/latest/topics/exceptions.html#scrapy.exceptions.IgnoreRequest)异常。
>
>如果它返回一个[`Response`](https://doc.scrapy.org/en/latest/topics/request-response.html#scrapy.http.Response)（它可能是相同的响应，或者是一个全新的响应），那么响应将继续与[`process_response()`](https://doc.scrapy.org/en/latest/topics/downloader-middleware.html#scrapy.downloadermiddlewares.DownloaderMiddleware.process_response)链中的下一个中间件一起处理。
>
>如果它返回一个[`Request`](https://doc.scrapy.org/en/latest/topics/request-response.html#scrapy.http.Request)对象，则中间件链将暂停，并且将返回的请求重新计划以备将来下载。这与返回请求的行为相同[`process_request()`](https://doc.scrapy.org/en/latest/topics/downloader-middleware.html#scrapy.downloadermiddlewares.DownloaderMiddleware.process_request)。
>
>如果引发[`IgnoreRequest`](https://doc.scrapy.org/en/latest/topics/exceptions.html#scrapy.exceptions.IgnoreRequest)异常，`Request.errback`则调用request（）的errback函数。如果没有代码处理引发的异常，它将被忽略并且不记录（不像其他异常）。
>
>参数：
>
>- **请求**（是一个[`Request`](https://doc.scrapy.org/en/latest/topics/request-response.html#scrapy.http.Request)对象） - 发起响应的请求
>- **响应**（[`Response`](https://doc.scrapy.org/en/latest/topics/request-response.html#scrapy.http.Response)对象） - 正在处理的响应
>- **蜘蛛**（[`Spider`](https://doc.scrapy.org/en/latest/topics/spiders.html#scrapy.spiders.Spider)物体） - 这个反应所针对的蜘蛛
>
>
>
>`process_exception`（*请求*，*例外*，*蜘蛛*）
>
>[`process_exception()`](https://doc.scrapy.org/en/latest/topics/downloader-middleware.html#scrapy.downloadermiddlewares.DownloaderMiddleware.process_exception)当下载处理程序或[`process_request()`](https://doc.scrapy.org/en/latest/topics/downloader-middleware.html#scrapy.downloadermiddlewares.DownloaderMiddleware.process_request)（从下载中间件）引发异常（包括[`IgnoreRequest`](https://doc.scrapy.org/en/latest/topics/exceptions.html#scrapy.exceptions.IgnoreRequest)异常）时，Scrapy会调用它，
>
>[`process_exception()`](https://doc.scrapy.org/en/latest/topics/downloader-middleware.html#scrapy.downloadermiddlewares.DownloaderMiddleware.process_exception)应该返回：要么`None`，一个[`Response`](https://doc.scrapy.org/en/latest/topics/request-response.html#scrapy.http.Response)对象或[`Request`](https://doc.scrapy.org/en/latest/topics/request-response.html#scrapy.http.Request)对象。
>
>如果它返回`None`，Scrapy将继续处理这个异常，执行任何其他[`process_exception()`](https://doc.scrapy.org/en/latest/topics/downloader-middleware.html#scrapy.downloadermiddlewares.DownloaderMiddleware.process_exception)已安装中间件的方法，直到没有中间件被遗留，并且默认的异常处理开始。
>
>如果它返回一个[`Response`](https://doc.scrapy.org/en/latest/topics/request-response.html#scrapy.http.Response)对象，那么[`process_response()`](https://doc.scrapy.org/en/latest/topics/downloader-middleware.html#scrapy.downloadermiddlewares.DownloaderMiddleware.process_response) 已安装的中间件的方法链就会启动，Scrapy也不会打扰调用任何其他[`process_exception()`](https://doc.scrapy.org/en/latest/topics/downloader-middleware.html#scrapy.downloadermiddlewares.DownloaderMiddleware.process_exception)中间件方法。
>
>如果它返回一个[`Request`](https://doc.scrapy.org/en/latest/topics/request-response.html#scrapy.http.Request)对象，则返回的请求将被重新计划以备将来下载。这会停止执行[`process_exception()`](https://doc.scrapy.org/en/latest/topics/downloader-middleware.html#scrapy.downloadermiddlewares.DownloaderMiddleware.process_exception)中间件的方法，就像返回响应一样。
>
>参数：
>
>- **请求**（是一个[`Request`](https://doc.scrapy.org/en/latest/topics/request-response.html#scrapy.http.Request)对象） - 生成异常的请求
>- **异常**（一个`Exception`对象） - 引发的异常
>- **蜘蛛**（[`Spider`](https://doc.scrapy.org/en/latest/topics/spiders.html#scrapy.spiders.Spider)物体） - 这个请求所针对的蜘蛛
>
>
>
>`from_crawler`（*CLS*，*履带*）
>
>如果存在，这个classmethod被称为从a创建一个中间件实例[`Crawler`](https://doc.scrapy.org/en/latest/topics/api.html#scrapy.crawler.Crawler)。它必须返回一个新的中间件实例。抓取工具对象提供对所有Scrapy核心组件的访问，如设置和信号; 这是中间件访问它们并将其功能挂接到Scrapy的一种方式。
>
>参数：
>
>- **爬虫**（[`Crawler`](https://doc.scrapy.org/en/latest/topics/api.html#scrapy.crawler.Crawler)对象） - 使用此中间件的爬虫

### 内置下载中间件参考

本页面描述了Scrapy附带的所有下载器中间件组件。有关如何使用它们以及如何编写自己的下载器中间件的信息，请参阅[下载器中间件使用指南](https://doc.scrapy.org/en/latest/topics/downloader-middleware.html#topics-downloader-middleware)。

有关默认启用的组件列表（及其订单），请参阅该 [`DOWNLOADER_MIDDLEWARES_BASE`](https://doc.scrapy.org/en/latest/topics/settings.html#std:setting-DOWNLOADER_MIDDLEWARES_BASE)设置。

#### CookiesMiddleware

>*class*`scrapy.downloadermiddlewares.cookies.``CookiesMiddleware`
>
>该中间件可以处理需要cookie的网站，例如那些使用会话的网站。它跟踪由Web服务器发送的cookie，并将其发回（随后从蜘蛛中获取），就像Web浏览器一样。
>
>以下设置可用于配置Cookie中间件：
>
>- [`COOKIES_ENABLED`](https://doc.scrapy.org/en/latest/topics/downloader-middleware.html#std:setting-COOKIES_ENABLED)
>- [`COOKIES_DEBUG`](https://doc.scrapy.org/en/latest/topics/downloader-middleware.html#std:setting-COOKIES_DEBUG)

#### 每个蜘蛛多个Cookie会话

新版本0.15。

通过使用[`cookiejar`](https://doc.scrapy.org/en/latest/topics/downloader-middleware.html#std:reqmeta-cookiejar)请求元密钥，支持每个蜘蛛保持多个Cookie会话 。默认情况下，它使用一个cookie jar（会话），但是你可以传递一个标识符来使用不同的标识符。

例如：

```Python
for i, url in enumerate(urls):
    yield scrapy.Request(url, meta={'cookiejar': i},
        callback=self.parse_page)
```

请记住，[`cookiejar`](https://doc.scrapy.org/en/latest/topics/downloader-middleware.html#std:reqmeta-cookiejar)元键不是“粘性”的。您需要继续在随后的请求中传递它。例如：

```python
def parse_page(self, response):
    # do some processing
    return scrapy.Request("http://www.example.com/otherpage",
        meta={'cookiejar': response.meta['cookiejar']},
        callback=self.parse_other_page)
```

#### COOKIES_ENABLED

默认： `True`

是否启用Cookie中间件。如果禁用，则不会将Cookie发送到Web服务器。

注意，如果[`Request`](https://doc.scrapy.org/en/latest/topics/request-response.html#scrapy.http.Request) 已经`meta['dont_merge_cookies']`评估`True`。尽管价值[`COOKIES_ENABLED`](https://doc.scrapy.org/en/latest/topics/downloader-middleware.html#std:setting-COOKIES_ENABLED)饼干将**不会**被发送到Web服务器，并收到饼干 [`Response`](https://doc.scrapy.org/en/latest/topics/request-response.html#scrapy.http.Response)将**不会**与现有的cookie合并。

有关更多详细信息，请参阅中的`cookies`参数 [`Request`](https://doc.scrapy.org/en/latest/topics/request-response.html#scrapy.http.Request)

#### COOKIES_DEBUG

默认： `False`

如果启用，Scrapy会记录在请求中发送的所有Cookie（即`Cookie` 标题）以及在响应中收到的所有Cookie（即`Set-Cookie`标题）。

以下是[`COOKIES_DEBUG`](https://doc.scrapy.org/en/latest/topics/downloader-middleware.html#std:setting-COOKIES_DEBUG)启用日志的示例：

```
2011-04-06 14:35:10-0300 [scrapy.core.engine] INFO: Spider opened
2011-04-06 14:35:10-0300 [scrapy.downloadermiddlewares.cookies] DEBUG: Sending cookies to: <GET http://www.diningcity.com/netherlands/index.html>
        Cookie: clientlanguage_nl=en_EN
2011-04-06 14:35:14-0300 [scrapy.downloadermiddlewares.cookies] DEBUG: Received cookies from: <200 http://www.diningcity.com/netherlands/index.html>
        Set-Cookie: JSESSIONID=B~FA4DC0C496C8762AE4F1A620EAB34F38; Path=/
        Set-Cookie: ip_isocode=US
        Set-Cookie: clientlanguage_nl=en_EN; Expires=Thu, 07-Apr-2011 21:21:34 GMT; Path=/
2011-04-06 14:49:50-0300 [scrapy.core.engine] DEBUG: Crawled (200) <GET http://www.diningcity.com/netherlands/index.html> (referer: None)
[...]
```

#### DefaultHeadersMiddleware

>*class*`scrapy.downloadermiddlewares.defaultheaders.``DefaultHeadersMiddleware`

该中间件设置设置中指定的所有默认请求标头 [`DEFAULT_REQUEST_HEADERS`](https://doc.scrapy.org/en/latest/topics/settings.html#std:setting-DEFAULT_REQUEST_HEADERS)。

#### DownloadTimeoutMiddleware

>*class*`scrapy.downloadermiddlewares.downloadtimeout.``DownloadTimeoutMiddleware`

该中间件为[`DOWNLOAD_TIMEOUT`](https://doc.scrapy.org/en/latest/topics/settings.html#std:setting-DOWNLOAD_TIMEOUT)设置或`download_timeout` spider属性中指定的请求设置下载超时。

>**！注意**
>
>任何下载器中间件方法也可能返回延迟。

#### HttpAuthMiddleware

>*class*`scrapy.downloadermiddlewares.httpauth.``HttpAuthMiddleware`

该中间件使用[基本访问验证](https://en.wikipedia.org/wiki/Basic_access_authentication)（又称HTTP验证）来[验证](https://en.wikipedia.org/wiki/Basic_access_authentication)从某些蜘蛛生成的所有请求。

为了能够从某些蜘蛛HTTP认证，设置`http_user` 和`http_pass`那些蜘蛛属性。

例：

```python
from scrapy.spiders import CrawlSpider

class SomeIntranetSiteSpider(CrawlSpider):

    http_user = 'someuser'
    http_pass = 'somepass'
    name = 'intranet.example.com'

    # .. rest of the spider code omitted ..
```

#### HttpCacheMiddleware

>*class*`scrapy.downloadermiddlewares.httpcache.``HttpCacheMiddleware`

该中间件为所有HTTP请求和响应提供低级缓存。它必须与缓存存储后端以及缓存策略结合使用。

Scrapy附带三个HTTP缓存存储后端：

> - [文件系统存储后端（默认）](https://doc.scrapy.org/en/latest/topics/downloader-middleware.html#httpcache-storage-fs)
> - [DBM存储后端](https://doc.scrapy.org/en/latest/topics/downloader-middleware.html#httpcache-storage-dbm)
> - [LevelDB存储后端](https://doc.scrapy.org/en/latest/topics/downloader-middleware.html#httpcache-storage-leveldb)

您可以使用该[`HTTPCACHE_STORAGE`](https://doc.scrapy.org/en/latest/topics/downloader-middleware.html#std:setting-HTTPCACHE_STORAGE) 设置更改HTTP缓存存储后端。或者你也可以实现你自己的存储后端。

Scrapy附带两个HTTP缓存策略：

> - [RFC2616政策](https://doc.scrapy.org/en/latest/topics/downloader-middleware.html#httpcache-policy-rfc2616)
> - [虚拟策略（默认）](https://doc.scrapy.org/en/latest/topics/downloader-middleware.html#httpcache-policy-dummy)

您可以使用该[`HTTPCACHE_POLICY`](https://doc.scrapy.org/en/latest/topics/downloader-middleware.html#std:setting-HTTPCACHE_POLICY) 设置更改HTTP缓存策略。或者你也可以实施你自己的政策。

您还可以避免使用[`dont_cache`](https://doc.scrapy.org/en/latest/topics/downloader-middleware.html#std:reqmeta-dont_cache)meta key等于True在每个策略上缓存响应。

#### 虚拟策略（默认）

该策略不了解任何HTTP Cache-Control指令。每个请求及其相应的响应都被缓存。当再次看到相同的请求时，将返回响应而不从Internet传输任何内容。

Dummy策略对于更快地测试蜘蛛（无需每次都等待下载）以及当Internet连接不可用时尝试使蜘蛛离线都很有用。我们的目标是能够像以前一样“重播”蜘蛛*跑*。

为了使用此策略，请设置：

- [`HTTPCACHE_POLICY`](https://doc.scrapy.org/en/latest/topics/downloader-middleware.html#std:setting-HTTPCACHE_POLICY) 至 `scrapy.extensions.httpcache.DummyPolicy`

#### RFC2616政策

该策略提供符合RFC2616的HTTP缓存，即HTTP缓存控制感知，旨在生产并用于连续运行，以避免下载未修改的数据（以节省带宽并加快爬网）。

实施内容：

- 不要尝试存储没有存储缓存控制指令集的响应/请求

- 如果即使对于新的响应设置了no-cache cache-control指令，也不要从高速缓存提供响应

- 根据最大年龄缓存控制指令计算新鲜度寿命

- 计算Expires响应标题的新鲜度生命周期

- 从Last-Modified响应标头计算新鲜生命期（Firefox使用的启发式）

- 从年龄响应标题计算当前年龄

- 从日期标题计算当前时间

- 根据Last-Modified响应标题重新验证陈旧的响应

- 基于ETag响应头重新验证陈旧的响应

- 为任何收到的响应设置日期标题丢失它

- 在请求中支持最大陈旧的缓存控制指令

  这使得蜘蛛可以使用完整的RFC2616缓存策略进行配置，但可以避免在逐个请求的基础上进行重新验证，同时保持与HTTP规范的一致性。

  例：

  添加缓存控制：max-stale = 600以请求标头接受超过其过期时间不超过600秒的响应。

  另见：RFC2616,14.9.3

什么不见了：

- Pragma：无缓存支持<https://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.9.1>
- 多种头文件支持<https://www.w3.org/Protocols/rfc2616/rfc2616-sec13.html#sec13.6>
- 更新或删除后无效<https://www.w3.org/Protocols/rfc2616/rfc2616-sec13.html#sec13.10>
- ...可能是其他人..

为了使用此策略，请设置：

- [`HTTPCACHE_POLICY`](https://doc.scrapy.org/en/latest/topics/downloader-middleware.html#std:setting-HTTPCACHE_POLICY) 至 `scrapy.extensions.httpcache.RFC2616Policy`

#### 文件系统存储后端（默认）

文件系统存储后端可用于HTTP缓存中间件。

为了使用此存储后端，请设置：

- [`HTTPCACHE_STORAGE`](https://doc.scrapy.org/en/latest/topics/downloader-middleware.html#std:setting-HTTPCACHE_STORAGE) 至 `scrapy.extensions.httpcache.FilesystemCacheStorage`

每个请求/响应对都存储在包含以下文件的不同目录中：

> - `request_body` - 简单的请求主体
> - `request_headers` - 请求标题（原始HTTP格式）
> - `response_body` - 简单的回应机构
> - `response_headers` - 请求标题（原始HTTP格式）
> - `meta`- Python `repr()`格式的这种缓存资源的一些元数据（grep-friendly格式）
> - `pickled_meta`- `meta`为了更有效的反序列化而腌制相同的元数据

目录名称由请求指纹（请参见参考资料`scrapy.utils.request.fingerprint`）制作而成 ，并且使用一级子目录以避免将太多文件创建到同一目录（这在许多文件系统中效率低下）。示例目录可以是：

````
/ path / to / cache / dir / example 。COM / 72 / 七二八一一f648e718090f041317756c03adb0ada46c7
````

#### DBM存储后端

0.13版本的新功能。

一个[DBM](https://en.wikipedia.org/wiki/Dbm)存储后端也可用于HTTP缓存中间件。

默认情况下，它使用[anydbm](https://docs.python.org/2/library/anydbm.html)模块，但可以使用该[`HTTPCACHE_DBM_MODULE`](https://doc.scrapy.org/en/latest/topics/downloader-middleware.html#std:setting-HTTPCACHE_DBM_MODULE)设置对其进行更改 。

为了使用此存储后端，请设置：

- [`HTTPCACHE_STORAGE`](https://doc.scrapy.org/en/latest/topics/downloader-middleware.html#std:setting-HTTPCACHE_STORAGE) 至 `scrapy.extensions.httpcache.DbmCacheStorage`

#### LevelDB存储后端

0.23版本的新功能。

一个[性LevelDB](https://github.com/google/leveldb)存储后端也可用于HTTP缓存中间件。

由于只有一个进程可以同时访问LevelDB数据库，因此不推荐将此后端用于开发，因此不能同时为同一个蜘蛛运行爬网并打开scrapy shell。

为了使用这个存储后端：

- 设置[`HTTPCACHE_STORAGE`](https://doc.scrapy.org/en/latest/topics/downloader-middleware.html#std:setting-HTTPCACHE_STORAGE)为`scrapy.extensions.httpcache.LeveldbCacheStorage`
- 像安装[LevelDB python绑定](https://pypi.python.org/pypi/leveldb)一样`pip install leveldb`

#### HTTPCache中间件设置

的[`HttpCacheMiddleware`](https://doc.scrapy.org/en/latest/topics/downloader-middleware.html#scrapy.downloadermiddlewares.httpcache.HttpCacheMiddleware)可以通过以下设置来配置：

#### HTTPCACHE_ENABLED

0.11版本的新功能。

默认： `False`

是否启用HTTP缓存。

在版本0.11中更改：在0.11之前，[`HTTPCACHE_DIR`](https://doc.scrapy.org/en/latest/topics/downloader-middleware.html#std:setting-HTTPCACHE_DIR)用于启用缓存。

#### HTTPCACHE_EXPIRATION_SECS

默认： `0`

缓存请求的到期时间，以秒为单位。

比这次更早的缓存请求将被重新下载。如果为零，则高速缓存的请求将永不过期。

在版本0.11中更改：在0.11之前，零意味着缓存的请求总是过期。

#### HTTPCACHE_DIR

默认： `'httpcache'`

用于存储（低级别）HTTP缓存的目录。如果为空，则HTTP缓存将被禁用。如果给出相对路径，则相对于项目数据目录进行。欲了解更多信息，请参阅：[Scrapy项目的默认结构](https://doc.scrapy.org/en/latest/topics/commands.html#topics-project-structure)。

#### HTTPCACHE_IGNORE_HTTP_CODES

0.10版本中的新功能。

默认： `[]`

不要用这些HTTP代码缓存响应。

#### HTTPCACHE_IGNORE_MISSING

默认： `False`

如果启用，则在缓存中找不到的请求将被忽略而不是下载。

#### HTTPCACHE_IGNORE_SCHEMES

0.10版本中的新功能。

默认： `['file']`

不要使用这些URI方案缓存响应。

#### HTTPCACHE_STORAGE

默认： `'scrapy.extensions.httpcache.FilesystemCacheStorage'`

实现高速缓存存储后端的类。

#### HTTPCACHE_DBM_MODULE

0.13版本的新功能。

默认： `'anydbm'`

用于[DBM存储后端](https://doc.scrapy.org/en/latest/topics/downloader-middleware.html#httpcache-storage-dbm)的数据库模块。此设置特定于DBM后端。

#### HTTPCACHE_POLICY

0.18版本中的新功能。

默认： `'scrapy.extensions.httpcache.DummyPolicy'`

实现缓存策略的类。

#### HTTPCACHE_GZIP

1.0版中的新功能。

默认： `False`

如果启用，将使用gzip压缩所有缓存的数据。该设置特定于文件系统后端。

#### HTTPCACHE_ALWAYS_STORE

版本1.1中的新功能

默认： `False`

如果启用，将无条件缓存页面。

例如，蜘蛛可能希望在缓存中提供所有响应，以便将来与Cache-Control一起使用：max-stale。DummyPolicy缓存所有响应，但从不重新验证它们，有时需要更细致的策略。

该设置仍然遵循Cache-Control：响应中的无存储指令。如果你不希望出现这种情况，过滤无店出的Cache-Control头的回应你feedto缓存中间件。

#### HTTPCACHE_IGNORE_RESPONSE_CACHE_CONTROLS

版本1.1中的新功能

默认： `[]`

缓存控制指令列表中被忽略的响应。

网站通常会设置“无存储”，“无缓存”，“必须重新验证”等，但如果遵守这些指令，蜘蛛可能产生的流量就会变得不安。这允许有选择地忽略已知对于被爬网的站点不重要的Cache-Control指令。

我们假设蜘蛛不会在请求中发出Cache-Control指令，除非它真的需要它们，所以请求中的指令不会被过滤。

#### HttpCompressionMiddleware

>*class*`scrapy.downloadermiddlewares.httpcompression.``HttpCompressionMiddleware`

该中间件允许从网站发送/接收压缩（gzip，deflate）流量。

只要安装了[brotlipy](https://pypi.python.org/pypi/brotlipy)，该中间件也支持解码[brotli压缩的](https://www.ietf.org/rfc/rfc7932.txt)响应。

#### HttpCompressionMiddleware设置

##### COMPRESSION_ENABLED

默认： `True`

压缩中间件是否启用。

#### HttpProxyMiddleware

0.8版新增功能

>*class*`scrapy.downloadermiddlewares.httpproxy.``HttpProxyMiddleware`

该中间件通过设置对象的`proxy`元值来设置HTTP代理以用于请求 [`Request`](https://doc.scrapy.org/en/latest/topics/request-response.html#scrapy.http.Request)。

像Python标准库模块[urllib](https://docs.python.org/2/library/urllib.html)和[urllib2一样](https://docs.python.org/2/library/urllib2.html)，它遵循以下环境变量：

- `http_proxy`
- `https_proxy`
- `no_proxy`

您还可以将`proxy`每个请求的元键设置为像`http://some_proxy_server:port`or 的值`http://username:password@some_proxy_server:port`。请记住，该值将优先于`http_proxy`/ `https_proxy` 环境变量，并且也会忽略`no_proxy`环境变量。

#### RedirectMiddleware


>*class*`scrapy.downloadermiddlewares.redirect.``RedirectMiddleware`

请求经过的URL（被重定向）可以在`redirect_urls` [`Request.meta`](https://doc.scrapy.org/en/latest/topics/request-response.html#scrapy.http.Request.meta)密钥中找到。

该[`RedirectMiddleware`](https://doc.scrapy.org/en/latest/topics/downloader-middleware.html#scrapy.downloadermiddlewares.redirect.RedirectMiddleware)可通过以下设置进行配置（详情参见设置文档）：

- [`REDIRECT_ENABLED`](https://doc.scrapy.org/en/latest/topics/downloader-middleware.html#std:setting-REDIRECT_ENABLED)
- [`REDIRECT_MAX_TIMES`](https://doc.scrapy.org/en/latest/topics/settings.html#std:setting-REDIRECT_MAX_TIMES)

如果[`Request.meta`](https://doc.scrapy.org/en/latest/topics/request-response.html#scrapy.http.Request.meta)将`dont_redirect` 键设置为True，则该中间件将忽略该请求。

如果您想在蜘蛛中处理一些重定向状态代码，您可以在`handle_httpstatus_list`spider属性中指定这些代码。

例如，如果您希望重定向中间件忽略301和302响应（并将它们传递给您的蜘蛛），您可以这样做：

```Python
class MySpider(CrawlSpider):
    handle_httpstatus_list = [301, 302]
```

所述`handle_httpstatus_list`的键[`Request.meta`](https://doc.scrapy.org/en/latest/topics/request-response.html#scrapy.http.Request.meta)也可以被用于指定的响应代码，以允许在每个请求基础。您还可以设置meta键 `handle_httpstatus_all`来`True`，如果你想以允许请求的任何响应代码。

#### RedirectMiddleware设置

##### REDIRECT_ENABLED

0.13版本的新功能。

默认： `True`

是否启用重定向中间件。

##### REDIRECT_MAX_TIMES

默认： `20`

单个请求将遵循的最大重定向次数。

#### MetaRefreshMiddleware

*class*`scrapy.downloadermiddlewares.redirect.``MetaRefreshMiddleware`


该[`MetaRefreshMiddleware`](https://doc.scrapy.org/en/latest/topics/downloader-middleware.html#scrapy.downloadermiddlewares.redirect.MetaRefreshMiddleware)可通过以下设置进行配置（详情参见设置文档）：

- [`METAREFRESH_ENABLED`](https://doc.scrapy.org/en/latest/topics/downloader-middleware.html#std:setting-METAREFRESH_ENABLED)
- [`METAREFRESH_MAXDELAY`](https://doc.scrapy.org/en/latest/topics/downloader-middleware.html#std:setting-METAREFRESH_MAXDELAY)

该中间件服从[`REDIRECT_MAX_TIMES`](https://doc.scrapy.org/en/latest/topics/settings.html#std:setting-REDIRECT_MAX_TIMES)设置，[`dont_redirect`](https://doc.scrapy.org/en/latest/topics/downloader-middleware.html#std:reqmeta-dont_redirect) 并[`redirect_urls`](https://doc.scrapy.org/en/latest/topics/downloader-middleware.html#std:reqmeta-redirect_urls)按照所述的要求请求元键[`RedirectMiddleware`](https://doc.scrapy.org/en/latest/topics/downloader-middleware.html#scrapy.downloadermiddlewares.redirect.RedirectMiddleware)

#### MetaRefreshMiddleware设置

##### METAREFRESH_ENABLED

0.17版本中的新功能。

默认： `True`

是否启用Meta Refresh中间件。

##### METAREFRESH_MAXDELAY

默认： `100`

遵循重定向的最大元刷新延迟（以秒为单位）。某些站点使用元刷新重定向到会话过期页面，因此我们将自动重定向限制为最大延迟。

#### RetryMiddleware

>*class*`scrapy.downloadermiddlewares.retry.``RetryMiddleware`


在抓取过程中收集失败页面，并在蜘蛛抓取完所有常规（非失败）页面后结束重新安排。一旦没有更多的失败页面重试，这个中间件发送一个信号（retry_complete），所以其他扩展可以连接到该信号。

该[`RetryMiddleware`](https://doc.scrapy.org/en/latest/topics/downloader-middleware.html#scrapy.downloadermiddlewares.retry.RetryMiddleware)可通过以下设置进行配置（详情参见设置文档）：

- [`RETRY_ENABLED`](https://doc.scrapy.org/en/latest/topics/downloader-middleware.html#std:setting-RETRY_ENABLED)
- [`RETRY_TIMES`](https://doc.scrapy.org/en/latest/topics/downloader-middleware.html#std:setting-RETRY_TIMES)
- [`RETRY_HTTP_CODES`](https://doc.scrapy.org/en/latest/topics/downloader-middleware.html#std:setting-RETRY_HTTP_CODES)

如果[`Request.meta`](https://doc.scrapy.org/en/latest/topics/request-response.html#scrapy.http.Request.meta)将`dont_retry`键设置为True，则该中间件将忽略该请求。

#### 重试中间件设置

##### RETRY_ENABLED

0.13版本的新功能。

默认： `True`

是否启用重试中间件。

##### RETRY_TIMES

默认： `2`

除了第一次下载之外，重试的最大次数。

也可以使用[`max_retry_times`](https://doc.scrapy.org/en/latest/topics/request-response.html#std:reqmeta-max_retry_times)属性为每个请求指定最大重试次数 [`Request.meta`](https://doc.scrapy.org/en/latest/topics/request-response.html#scrapy.http.Request.meta)。初始化时，[`max_retry_times`](https://doc.scrapy.org/en/latest/topics/request-response.html#std:reqmeta-max_retry_times)元键优先于[`RETRY_TIMES`](https://doc.scrapy.org/en/latest/topics/downloader-middleware.html#std:setting-RETRY_TIMES)设置。

##### RETRY_HTTP_CODES

默认： `[500, 502, 503, 504, 408]`

哪个HTTP响应代码要重试。其他错误（DNS查找问题，连接丢失等）总是重试。

在某些情况下，您可能需要添加400，[`RETRY_HTTP_CODES`](https://doc.scrapy.org/en/latest/topics/downloader-middleware.html#std:setting-RETRY_HTTP_CODES)因为它是用于指示服务器过载的通用代码。它没有默认包含，因为HTTP规范是这样说的。

#### RobotsTxtMiddleware

>*class*`scrapy.downloadermiddlewares.robotstxt.``RobotsTxtMiddleware`

如果[`Request.meta`](https://doc.scrapy.org/en/latest/topics/request-response.html#scrapy.http.Request.meta)将 `dont_obey_robotstxt`键设置为True，则即使[`ROBOTSTXT_OBEY`](https://doc.scrapy.org/en/latest/topics/settings.html#std:setting-ROBOTSTXT_OBEY)启用该中间件，该请求也将被忽略 。

#### DownloaderStats

> *class*`scrapy.downloadermiddlewares.stats.``DownloaderStats`

存储所有通过它的请求，响应和异常的统计信息的中间件。

要使用此中间件，您必须启用该[`DOWNLOADER_STATS`](https://doc.scrapy.org/en/latest/topics/settings.html#std:setting-DOWNLOADER_STATS) 设置。

#### UserAgentMiddleware

>*class*`scrapy.downloadermiddlewares.useragent.``UserAgentMiddleware`

允许蜘蛛覆盖默认用户代理的中间件。

为了使蜘蛛覆盖默认用户代理， 必须设置其user_agent属性。

#### AjaxCrawlMiddleware

>*class*`scrapy.downloadermiddlewares.ajaxcrawl.``AjaxCrawlMiddleware`

根据meta-fragment html标签查找“AJAX可抓取”页面变体的中间件。有关 更多信息，请参阅<https://developers.google.com/webmasters/ajax-crawling/docs/getting-started>。

>**！注意**
>
>Scrapy发现了“AJAX可抓取”网页， `'http://example.com/!#foo=bar'`即使没有这个中间件也是如此。当URL不包含时，AjaxCrawlMiddleware是必需的`'!#'`。这通常是'索引'或'主'网站页面的情况。

#### AjaxCrawlMiddleware设置

##### AJAXCRAWL_ENABLED

新版本0.21。

默认： `False`

是否启用AjaxCrawlMiddleware。您可能需要启用它以进行[广泛抓取](https://doc.scrapy.org/en/latest/topics/broad-crawls.html#topics-broad-crawls)。

#### HttpProxyMiddleware设置

##### HTTPPROXY_ENABLED

默认： `True`

是否启用`HttpProxyMiddleware`。

##### HTTPPROXY_AUTH_ENCODING

默认： `"latin-1"`

代理身份验证的默认编码`HttpProxyMiddleware`。