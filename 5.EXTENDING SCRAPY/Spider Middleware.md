蜘蛛中间件是Scrapy的蜘蛛处理机制的钩子框架，您可以插入自定义功能来处理发送给[蜘蛛](https://doc.scrapy.org/en/latest/topics/spiders.html#topics-spiders)进行处理的响应，并处理从蜘蛛生成的请求和项目。

### 激活蜘蛛中间件

要激活蜘蛛中间件组件，将其添加到 [`SPIDER_MIDDLEWARES`](https://doc.scrapy.org/en/latest/topics/settings.html#std:setting-SPIDER_MIDDLEWARES)设置中，该设置是一个字典，其键是中间件类路径，它们的值是中间件命令。

这是一个例子：

```python
SPIDER_MIDDLEWARES  =  { 
    'myproject.middlewares.CustomSpiderMiddleware' ： 543 ，
}
```

该[`SPIDER_MIDDLEWARES`](https://doc.scrapy.org/en/latest/topics/settings.html#std:setting-SPIDER_MIDDLEWARES)设置与[`SPIDER_MIDDLEWARES_BASE`](https://doc.scrapy.org/en/latest/topics/settings.html#std:setting-SPIDER_MIDDLEWARES_BASE)Scrapy中定义的设置合并 （并不意味着被覆盖），然后按顺序排序以获得最终的已启用中间件排序列表：第一个中间件是靠近引擎的中间件，最后一个是靠近引擎的中间件到蜘蛛。换句话说，[`process_spider_input()`](https://doc.scrapy.org/en/latest/topics/spider-middleware.html#scrapy.spidermiddlewares.SpiderMiddleware.process_spider_input) 每个中间件的方法将以增加中间件的顺序（100,200,300，...）[`process_spider_output()`](https://doc.scrapy.org/en/latest/topics/spider-middleware.html#scrapy.spidermiddlewares.SpiderMiddleware.process_spider_output)被调用，并且每个中间件的 方法将按降序调用。

要决定分配给中间件的顺序，请参阅 [`SPIDER_MIDDLEWARES_BASE`](https://doc.scrapy.org/en/latest/topics/settings.html#std:setting-SPIDER_MIDDLEWARES_BASE)设置并根据要插入中间件的位置选择一个值。顺序很重要，因为每个中间件都执行不同的操作，而您的中间件可能依赖于某些以前（或后续）正在应用的中间件。

如果要禁用内置中间件（[`SPIDER_MIDDLEWARES_BASE`](https://doc.scrapy.org/en/latest/topics/settings.html#std:setting-SPIDER_MIDDLEWARES_BASE)默认情况下定义的内置中间件 ），则必须在项目[`SPIDER_MIDDLEWARES`](https://doc.scrapy.org/en/latest/topics/settings.html#std:setting-SPIDER_MIDDLEWARES)设置中定义它，并将None指定为其值。例如，如果您想禁用场外中间件：

```python
SPIDER_MIDDLEWARES  =  { 
    'myproject.middlewares.CustomSpiderMiddleware' ： 543 ，
    'scrapy.spidermiddlewares.offsite.OffsiteMiddleware' ： None ，
}
```

最后，请记住，某些中间件可能需要通过特定设置启用。有关更多信息，请参阅每个中间件文档。

### 编写你自己的蜘蛛中间件

每个中间件组件都是一个Python类，它定义了以下一种或多种方法：

>*class*`scrapy.spidermiddlewares.``SpiderMiddleware`
>
>> `process_spider_input`（*反应*，*蜘蛛*）
>>
>> 这种方法被称为每一个通过蜘蛛中间件和蜘蛛进行处理的响应。
>>
>> [`process_spider_input()`](https://doc.scrapy.org/en/latest/topics/spider-middleware.html#scrapy.spidermiddlewares.SpiderMiddleware.process_spider_input)应该返回`None`或引发异常。
>>
>> 如果它返回`None`，Scrapy将继续处理这个响应，执行所有其他中间件，直到最终将响应交给蜘蛛进行处理。
>>
>> 如果它引发异常，Scrapy将不会打扰任何其他蜘蛛中间件，[`process_spider_input()`](https://doc.scrapy.org/en/latest/topics/spider-middleware.html#scrapy.spidermiddlewares.SpiderMiddleware.process_spider_input)并会调用请求errback。errback的输出在另一个方向上被链接[`process_spider_output()`](https://doc.scrapy.org/en/latest/topics/spider-middleware.html#scrapy.spidermiddlewares.SpiderMiddleware.process_spider_output)以处理它，或者 [`process_spider_exception()`](https://doc.scrapy.org/en/latest/topics/spider-middleware.html#scrapy.spidermiddlewares.SpiderMiddleware.process_spider_exception)如果它引发异常。
>>
>> 参数：
>>
>> - **响应**（[`Response`](https://doc.scrapy.org/en/latest/topics/request-response.html#scrapy.http.Response)对象） - 正在处理的响应
>> - **蜘蛛**（[`Spider`](https://doc.scrapy.org/en/latest/topics/spiders.html#scrapy.spiders.Spider)物体） - 这个反应所针对的蜘蛛
>>
>> `process_spider_output`（*反应*，*结果*，*蜘蛛*）
>>
>> 该方法在Spider处理完响应后调用返回的结果。
>>
>> [`process_spider_output()`](https://doc.scrapy.org/en/latest/topics/spider-middleware.html#scrapy.spidermiddlewares.SpiderMiddleware.process_spider_output)必须返回可迭代的 [`Request`](https://doc.scrapy.org/en/latest/topics/request-response.html#scrapy.http.Request)，字典或[`Item`](https://doc.scrapy.org/en/latest/topics/items.html#scrapy.item.Item) 对象。
>>
>> 参数：
>>
>> - **响应**（[`Response`](https://doc.scrapy.org/en/latest/topics/request-response.html#scrapy.http.Response)对象） - 从蜘蛛生成此输出的响应
>> - **结果**（可迭代的[`Request`](https://doc.scrapy.org/en/latest/topics/request-response.html#scrapy.http.Request)，字典或[`Item`](https://doc.scrapy.org/en/latest/topics/items.html#scrapy.item.Item)对象） - 蜘蛛返回的结果
>> - **蜘蛛**（[`Spider`](https://doc.scrapy.org/en/latest/topics/spiders.html#scrapy.spiders.Spider)物体） - 结果正在处理的蜘蛛
>>
>> `process_spider_exception`（*反应*，*例外*，*蜘蛛*）
>>
>> 当蜘蛛或[`process_spider_input()`](https://doc.scrapy.org/en/latest/topics/spider-middleware.html#scrapy.spidermiddlewares.SpiderMiddleware.process_spider_input) 方法（来自其他蜘蛛中间件）引发异常时调用此方法。
>>
>> [`process_spider_exception()`](https://doc.scrapy.org/en/latest/topics/spider-middleware.html#scrapy.spidermiddlewares.SpiderMiddleware.process_spider_exception)应该返回一个`None`或一个迭代[`Response`](https://doc.scrapy.org/en/latest/topics/request-response.html#scrapy.http.Response)，字典或 [`Item`](https://doc.scrapy.org/en/latest/topics/items.html#scrapy.item.Item)对象。
>>
>> 如果它返回`None`，Scrapy将继续处理这个异常，[`process_spider_exception()`](https://doc.scrapy.org/en/latest/topics/spider-middleware.html#scrapy.spidermiddlewares.SpiderMiddleware.process_spider_exception)在下面的中间件组件中执行任何其他的事件，直到没有中间件组件被遗留并且异常到达引擎（它被记录和丢弃的地方）。
>>
>> 如果它返回一个可迭代的[`process_spider_output()`](https://doc.scrapy.org/en/latest/topics/spider-middleware.html#scrapy.spidermiddlewares.SpiderMiddleware.process_spider_output)管道，[`process_spider_exception()`](https://doc.scrapy.org/en/latest/topics/spider-middleware.html#scrapy.spidermiddlewares.SpiderMiddleware.process_spider_exception)则不会调用其他管道。
>>
>> 参数：
>>
>> - **响应**（[`Response`](https://doc.scrapy.org/en/latest/topics/request-response.html#scrapy.http.Response)对象） - 引发异常时处理的响应
>> - **异常**（[异常](https://docs.python.org/2/library/exceptions.html#exceptions.Exception)对象） - 引发的异常
>> - **蜘蛛**（[`Spider`](https://doc.scrapy.org/en/latest/topics/spiders.html#scrapy.spiders.Spider)物体） - 引发异常的蜘蛛
>>
>> `process_start_requests`（*start_requests*，*蜘蛛*）
>>
>> 新版本0.15。
>>
>> 该方法使用spider的启动请求调用，并且与该[`process_spider_output()`](https://doc.scrapy.org/en/latest/topics/spider-middleware.html#scrapy.spidermiddlewares.SpiderMiddleware.process_spider_output)方法的工作方式类似，只是它没有关联的响应，并且只能返回请求（不是项目）。
>>
>> 它接收一个iterable（在`start_requests`参数中）并且必须返回另一个可迭代的[`Request`](https://doc.scrapy.org/en/latest/topics/request-response.html#scrapy.http.Request)对象。
>>
>> > **！注意**
>> >
>> > 在蜘蛛中间件中实现此方法时，应始终返回一个可迭代的（跟随输入的）并且不会消耗所有的`start_requests`迭代器，因为它可能非常大（甚至无界）并导致内存溢出。Scrapy引擎设计用于在启动请求有能力处理启动请求时启动，因此启动请求迭代器可以在停止蜘蛛的某些其他条件（如时间限制或项目/页数）的情况下实现无限循环。
>>
>> 参数：
>>
>> - **start_requests**（可迭代的[`Request`](https://doc.scrapy.org/en/latest/topics/request-response.html#scrapy.http.Request)） - 启动请求
>> - **蜘蛛**（[`Spider`](https://doc.scrapy.org/en/latest/topics/spiders.html#scrapy.spiders.Spider)对象） - 启动请求所属的蜘蛛
>>
>> `from_crawler`（*CLS*，*履带*）
>>
>> 如果存在，这个classmethod被称为从a创建一个中间件实例[`Crawler`](https://doc.scrapy.org/en/latest/topics/api.html#scrapy.crawler.Crawler)。它必须返回一个新的中间件实例。抓取工具对象提供对所有Scrapy核心组件的访问，如设置和信号; 这是中间件访问它们并将其功能挂接到Scrapy的一种方式。
>>
>> 参数：
>>
>> - **爬虫**（[`Crawler`](https://doc.scrapy.org/en/latest/topics/api.html#scrapy.crawler.Crawler)对象） - 使用此中间件的爬虫

### 内置蜘蛛中间件参考

本页面描述了Scrapy附带的所有蜘蛛中间件组件。有关如何使用它们以及如何编写自己的蜘蛛中间件的信息，请参阅[蜘蛛中间件使用指南](https://doc.scrapy.org/en/latest/topics/spider-middleware.html#topics-spider-middleware)。

有关默认启用的组件列表（及其订单），请参阅该 [`SPIDER_MIDDLEWARES_BASE`](https://doc.scrapy.org/en/latest/topics/settings.html#std:setting-SPIDER_MIDDLEWARES_BASE)设置。

#### DepthMiddleware

>*class*`scrapy.spidermiddlewares.depth.``DepthMiddleware`

DepthMiddleware用于跟踪被抓取站点内每个请求的深度。它通过设置request.meta ['depth'] = 0来工作，只要没有先前设置的值（通常只是第一个请求）并以1递增即可。

它可以用来限制最大深度来刮取，根据它们的深度控制请求优先级，以及类似的事情。

该[`DepthMiddleware`](https://doc.scrapy.org/en/latest/topics/spider-middleware.html#scrapy.spidermiddlewares.depth.DepthMiddleware)可通过以下设置进行配置（详情参见设置文档）：

> - [`DEPTH_LIMIT`](https://doc.scrapy.org/en/latest/topics/settings.html#std:setting-DEPTH_LIMIT) - 允许任何网站抓取的最大深度。如果为零，则不会施加限制。
> - [`DEPTH_STATS`](https://doc.scrapy.org/en/latest/topics/settings.html#std:setting-DEPTH_STATS) - 是否收集深度统计。
> - [`DEPTH_PRIORITY`](https://doc.scrapy.org/en/latest/topics/settings.html#std:setting-DEPTH_PRIORITY) - 是否根据深度优先处理请求。

#### HttpErrorMiddleware

>*class*`scrapy.spidermiddlewares.httperror.``HttpErrorMiddleware`


根据[HTTP标准](https://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html)，成功的响应是那些状态代码在200-300范围内的响应。

如果您仍想处理该范围之外的响应代码，则可以使用`handle_httpstatus_list`spider属性或[`HTTPERROR_ALLOWED_CODES`](https://doc.scrapy.org/en/latest/topics/spider-middleware.html#std:setting-HTTPERROR_ALLOWED_CODES)设置来指定蜘蛛能够处理哪些响应代码 。

例如，如果你想让蜘蛛来处理404响应，你可以这样做：

```python
class  MySpider （CrawlSpider ）：
    handle_httpstatus_list  =  [ 404 ]
```

所述`handle_httpstatus_list`的键[`Request.meta`](https://doc.scrapy.org/en/latest/topics/request-response.html#scrapy.http.Request.meta)也可以被用于指定的响应代码，以允许在每个请求基础。您还可以设置meta键`handle_httpstatus_all` 来`True`，如果你想以允许请求的任何响应代码。

但请记住，除非你真的知道你在做什么，否则处理非200响应通常是一个坏主意。

有关更多信息，请参阅：[HTTP状态代码定义](https://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html)。

#### HttpErrorMiddleware设置

##### HTTPERROR_ALLOWED_CODES

默认： `[]`

通过此列表中包含的非200状态代码的所有响应。

##### HTTPERROR_ALLOW_ALL

默认： `False`

无论其状态码如何，都可以通过所有回复。

#### OffsiteMiddleware

>*class*`scrapy.spidermiddlewares.offsite.``OffsiteMiddleware`

过滤蜘蛛所涉及域之外的URL请求。

该中间件过滤掉每个请求的主机名称不在蜘蛛的[`allowed_domains`](https://doc.scrapy.org/en/latest/topics/spiders.html#scrapy.spiders.Spider.allowed_domains)属性中。列表中任何域的所有子域也都是允许的。例如，规则`www.example.org`也允许`bob.www.example.org` ，但不能`www2.example.com`也不`example.com`。

当你的蜘蛛返回一个不属于蜘蛛覆盖的域的请求时，这个中间件会记录一条类似于这个的调试信息：

```
DEBUG: Filtered offsite request to 'www.othersite.com': <GET http://www.othersite.com/some/page.html>
```

为了避免过多的噪音填充日志，它只会为每个过滤的新域打印这些消息之一。因此，例如，如果`www.othersite.com`过滤另一个请求，则不会打印日志消息。但是，如果`someothersite.com`过滤请求，则会打印一条消息（但仅限于过滤的第一个请求）。

如果蜘蛛没有定义 [`allowed_domains`](https://doc.scrapy.org/en/latest/topics/spiders.html#scrapy.spiders.Spider.allowed_domains)属性，或者该属性为空，则非现场中间件将允许所有请求。

如果请求具有`dont_filter`属性集，则即使其域未在允许的域中列出，异地中间件也将允许该请求。

#### RefererMiddleware

>*class*`scrapy.spidermiddlewares.referer.``RefererMiddleware`

`Referer`根据生成它的Response的URL 填充Request 头。

#### RefererMiddleware设置

##### REFERER_ENABLED

新版本0.15。

默认： `True`

是否启用引用中间件。

##### REFERRER_POLICY

1.4版本中的新功能。

默认： `'scrapy.spidermiddlewares.referer.DefaultReferrerPolicy'`

[引荐者策略](https://www.w3.org/TR/referrer-policy)在填充请求“引用者”标题时应用。

> **！注意**
>
> 您还可以使用特殊的`"referrer_policy"` [Request.meta](https://doc.scrapy.org/en/latest/topics/request-response.html#topics-request-meta)键为每个请求设置Referrer Policy ，其`REFERRER_POLICY`设置的值与可接受的值相同。

##### REFERRER_POLICY的可接受值

- 要么是一个`scrapy.spidermiddlewares.referer.ReferrerPolicy` 子类的路径- 一个自定义策略或一个内置的策略（见下面的类），
- 或者标准的W3C定义的字符串值之一，
- 或特殊的`"scrapy-default"`。

| 字符串值                                                     | 类名（作为字符串）                                           |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| `"scrapy-default"` （默认）                                  | [`scrapy.spidermiddlewares.referer.DefaultReferrerPolicy`](https://doc.scrapy.org/en/latest/topics/spider-middleware.html#scrapy.spidermiddlewares.referer.DefaultReferrerPolicy) |
| [“无引荐”](https://www.w3.org/TR/referrer-policy/#referrer-policy-no-referrer) | [`scrapy.spidermiddlewares.referer.NoReferrerPolicy`](https://doc.scrapy.org/en/latest/topics/spider-middleware.html#scrapy.spidermiddlewares.referer.NoReferrerPolicy) |
| [“无引荐，当降级”](https://www.w3.org/TR/referrer-policy/#referrer-policy-no-referrer-when-downgrade) | [`scrapy.spidermiddlewares.referer.NoReferrerWhenDowngradePolicy`](https://doc.scrapy.org/en/latest/topics/spider-middleware.html#scrapy.spidermiddlewares.referer.NoReferrerWhenDowngradePolicy) |
| [“同源”](https://www.w3.org/TR/referrer-policy/#referrer-policy-same-origin) | [`scrapy.spidermiddlewares.referer.SameOriginPolicy`](https://doc.scrapy.org/en/latest/topics/spider-middleware.html#scrapy.spidermiddlewares.referer.SameOriginPolicy) |
| [“起源”](https://www.w3.org/TR/referrer-policy/#referrer-policy-origin) | [`scrapy.spidermiddlewares.referer.OriginPolicy`](https://doc.scrapy.org/en/latest/topics/spider-middleware.html#scrapy.spidermiddlewares.referer.OriginPolicy) |
| [“严格的原产地”](https://www.w3.org/TR/referrer-policy/#referrer-policy-strict-origin) | [`scrapy.spidermiddlewares.referer.StrictOriginPolicy`](https://doc.scrapy.org/en/latest/topics/spider-middleware.html#scrapy.spidermiddlewares.referer.StrictOriginPolicy) |
| [“源 - 当交起源”](https://www.w3.org/TR/referrer-policy/#referrer-policy-origin-when-cross-origin) | [`scrapy.spidermiddlewares.referer.OriginWhenCrossOriginPolicy`](https://doc.scrapy.org/en/latest/topics/spider-middleware.html#scrapy.spidermiddlewares.referer.OriginWhenCrossOriginPolicy) |
| [“严格的原产地，当交起源”](https://www.w3.org/TR/referrer-policy/#referrer-policy-strict-origin-when-cross-origin) | [`scrapy.spidermiddlewares.referer.StrictOriginWhenCrossOriginPolicy`](https://doc.scrapy.org/en/latest/topics/spider-middleware.html#scrapy.spidermiddlewares.referer.StrictOriginWhenCrossOriginPolicy) |
| [“不安全的URL”](https://www.w3.org/TR/referrer-policy/#referrer-policy-unsafe-url) | [`scrapy.spidermiddlewares.referer.UnsafeUrlPolicy`](https://doc.scrapy.org/en/latest/topics/spider-middleware.html#scrapy.spidermiddlewares.referer.UnsafeUrlPolicy) |

>*class*`scrapy.spidermiddlewares.referer.``DefaultReferrerPolicy`

“降级时不使用引用者”的变体，并且在父请求正在使用`file://`或`s3://`计划时不会发送“引用者” 。

> **！警告**
>
> Scrapy的默认引用策略 - 就像W3C推荐的浏览器值- [“no-referrer-when-downgrade”一样](https://www.w3.org/TR/referrer-policy/#referrer-policy-no-referrer-when-downgrade)，即使域不同，它也会从`http(s)://`任意`https://`URL 向任何URL 发送非空的“Referer”头。
>
> 如果您想删除跨域请求的引荐来源信息，[“同源”](https://www.w3.org/TR/referrer-policy/#referrer-policy-same-origin)可能是更好的选择。

> *class*`scrapy.spidermiddlewares.referer.``NoReferrerPolicy`

<https://www.w3.org/TR/referrer-policy/#referrer-policy-no-referrer>

最简单的策略是“禁止引用者”，它指定不引用任何引用信息以及从特定请求客户端向任何源发出的请求。标题将被完全省略。

>*class*`scrapy.spidermiddlewares.referer.``NoReferrerWhenDowngradePolicy`

<https://www.w3.org/TR/referrer-policy/#referrer-policy-no-referrer-when-downgrade>

“no-referrer-when-downgrade”策略将完整的URL以及来自受TLS保护的环境设置对象的请求发送到可能可信的URL以及来自客户端的未受TLS保护的来自任何来源的请求。

另一方面，从受TLS保护的客户端请求非潜在可信的URL将不包含引荐信息。Referer HTTP头将不会被发送。

如果没有另外指定策略，这是用户代理的默认行为。

> **！注意**
>
> “no-referrer-when-downgrade”策略是W3C推荐的默认策略，并被主要的Web浏览器使用。
>
> 但是，这不是Scrapy的默认引用者策略（请参阅参考资料[`DefaultReferrerPolicy`](https://doc.scrapy.org/en/latest/topics/spider-middleware.html#scrapy.spidermiddlewares.referer.DefaultReferrerPolicy)）。

>*class*`scrapy.spidermiddlewares.referer.``SameOriginPolicy`

<https://www.w3.org/TR/referrer-policy/#referrer-policy-same-origin>

“相同来源”政策规定，在从特定请求客户端发出同源请求时，会将作为引用者使用的完整URL作为引用者信息发送。

另一方面，跨境请求将不包含引荐来源信息。Referer HTTP头将不会被发送。

>*class*`scrapy.spidermiddlewares.referer.``OriginPolicy`

<https://www.w3.org/TR/referrer-policy/#referrer-policy-origin>

“起源”策略规定，当从特定请求客户端发出同源请求和跨源请求时，只有请求客户端源的ASCII序列号作为引用者信息发送。

>*class*`scrapy.spidermiddlewares.referer.``StrictOriginPolicy`

<https://www.w3.org/TR/referrer-policy/#referrer-policy-strict-origin>

“严格来源”策略在发出请求时发送请求客户端的源代码的ASCII序列化： - 从受TLS保护的环境设置对象到可能可信的URL，以及 - 从非TLS保护的环境设置对象到任何起源。

另一方面，从TLS保护的请求客户端请求非潜在可信的URL将不包含引荐者信息。Referer HTTP头将不会被发送。

>*class*`scrapy.spidermiddlewares.referer.``OriginWhenCrossOriginPolicy`

<https://www.w3.org/TR/referrer-policy/#referrer-policy-strict-origin-when-cross-origin>

“严格来源时，交叉来源”政策规定，剥离用作引荐来源的完整URL作为引荐来源信息发送，当从特定请求客户端发出同源请求时，只有ASCII的序列化请求客户端进行跨域请求时的起源：

- 从受TLS保护的环境设置对象到可能可信的URL，以及
- 从非TLS保护的环境设置对象到任何原点。

另一方面，来自受TLS保护的客户端请求的不可信的URL将不包含引荐者信息。Referer HTTP头将不会被发送。

>*class*`scrapy.spidermiddlewares.referer.``StrictOriginWhenCrossOriginPolicy`

<https://www.w3.org/TR/referrer-policy/#referrer-policy-unsafe-url>

“unsafe-url”策略指定一个完整的URL被剥离用作引用链接，并且同时发送来自特定请求客户端的跨源请求和同源请求。

注意：保单的名称不是谎言; 这是不安全的。此政策会将来自TLS保护资源的来源和路径泄漏到不安全的来源。仔细考虑为可能敏感的文档设置此策略的影响。

*class*`scrapy.spidermiddlewares.referer.``UnsafeUrlPolicy`

> **！警告**
>
> 不建议使用“不安全URL”策略。

#### UrlLengthMiddleware

>*class*`scrapy.spidermiddlewares.urllength.``UrlLengthMiddleware`

筛选出URL长度超过URLLENGTH_LIMIT的请求

该[`UrlLengthMiddleware`](https://doc.scrapy.org/en/latest/topics/spider-middleware.html#scrapy.spidermiddlewares.urllength.UrlLengthMiddleware)可通过以下设置进行配置（详情参见设置文档）：

> - [`URLLENGTH_LIMIT`](https://doc.scrapy.org/en/latest/topics/settings.html#std:setting-URLLENGTH_LIMIT) - 允许抓取的网址的最大网址长度。













