Scrapy广泛使用信号来通知特定事件发生的时间。您可以捕获Scrapy项目中的一些信号（例如使用[扩展](https://doc.scrapy.org/en/latest/topics/extensions.html#topics-extensions)）来执行其他任务或扩展Scrapy以添加不提供的功能。

即使信号提供了多个参数，捕获它们的处理程序也不需要接受所有这些参数 - 信号分派机制只会传递处理程序接收到的参数。

您可以通过[Signals API](https://doc.scrapy.org/en/latest/topics/api.html#topics-api-signals)连接到信号（或发送自己的 [信号）](https://doc.scrapy.org/en/latest/topics/api.html#topics-api-signals)。

下面是一个简单的例子，展示如何捕捉信号并执行一些操作：

```python
from scrapy import signals
from scrapy import Spider


class DmozSpider(Spider):
    name = "dmoz"
    allowed_domains = ["dmoz.org"]
    start_urls = [
        "http://www.dmoz.org/Computers/Programming/Languages/Python/Books/",
        "http://www.dmoz.org/Computers/Programming/Languages/Python/Resources/",
    ]


    @classmethod
    def from_crawler(cls, crawler, *args, **kwargs):
        spider = super(DmozSpider, cls).from_crawler(crawler, *args, **kwargs)
        crawler.signals.connect(spider.spider_closed, signal=signals.spider_closed)
        return spider


    def spider_closed(self, spider):
        spider.logger.info('Spider closed: %s', spider.name)


    def parse(self, response):
        pass
```

## 延迟信号处理程序

某些信号支持从处理程序返回[Twisted延迟](https://twistedmatrix.com/documents/current/core/howto/defer.html)，请参阅下面的[内置信号参考](https://doc.scrapy.org/en/latest/topics/signals.html#topics-signals-ref)以了解哪些[信号](https://doc.scrapy.org/en/latest/topics/signals.html#topics-signals-ref)。

## 内置信号参考

以下是Scrapy内置信号列表及其含义。

### engine_started

>`scrapy.signals.``engine_started`（）

当Scrapy引擎开始爬行时发送。

该信号支持从处理程序返回延迟。

> **！注意**
>
> 这个信号可能被解雇*后*的[`spider_opened`](https://doc.scrapy.org/en/latest/topics/signals.html#std:signal-spider_opened)信号，这取决于蜘蛛是如何开始。所以**不要**依赖这个信号之前被解雇[`spider_opened`](https://doc.scrapy.org/en/latest/topics/signals.html#std:signal-spider_opened)。

### engine_stopped

>`scrapy.signals.``engine_stopped`（）

Scrapy引擎停止时发送（例如，爬网过程完成时）。

该信号支持从处理程序返回延迟。

### item_scraped

>`scrapy.signals.``item_scraped`（*物品*，*反应*，*蜘蛛*）

当一件物品已经通过所有 [物品管线](https://doc.scrapy.org/en/latest/topics/item-pipeline.html#topics-item-pipeline)阶段（没有被丢弃）后被发送。

该信号支持从处理程序返回延迟。

参数：

- **项目**（字典或[`Item`](https://doc.scrapy.org/en/latest/topics/items.html#scrapy.item.Item)对象） - 项目被刮掉
- **蜘蛛**（[`Spider`](https://doc.scrapy.org/en/latest/topics/spiders.html#scrapy.spiders.Spider)物体） - 刮掉物品的蜘蛛
- **响应**（[`Response`](https://doc.scrapy.org/en/latest/topics/request-response.html#scrapy.http.Response)对象） - 项目被刮取的响应

### item_dropped

>`crapy.signals.``item_dropped`（*物品*，*反应*，*例外*，*蜘蛛*）

当某个阶段发生异常时，从[项目管道中](https://doc.scrapy.org/en/latest/topics/item-pipeline.html#topics-item-pipeline)删除某个项目后发送[`DropItem`](https://doc.scrapy.org/en/latest/topics/exceptions.html#scrapy.exceptions.DropItem)。

该信号支持从处理程序返回延迟。

参数：

- **项目**（字典或[`Item`](https://doc.scrapy.org/en/latest/topics/items.html#scrapy.item.Item)对象） - 从[项目管道中](https://doc.scrapy.org/en/latest/topics/item-pipeline.html#topics-item-pipeline)删除的[项目](https://doc.scrapy.org/en/latest/topics/item-pipeline.html#topics-item-pipeline)
- **蜘蛛**（[`Spider`](https://doc.scrapy.org/en/latest/topics/spiders.html#scrapy.spiders.Spider)物体） - 刮掉物品的蜘蛛
- **响应**（[`Response`](https://doc.scrapy.org/en/latest/topics/request-response.html#scrapy.http.Response)对象） - 项目被放置的响应
- **异常**（[`DropItem`](https://doc.scrapy.org/en/latest/topics/exceptions.html#scrapy.exceptions.DropItem)异常） - [`DropItem`](https://doc.scrapy.org/en/latest/topics/exceptions.html#scrapy.exceptions.DropItem)导致项目被删除的异常（必须是 子类）

### spider_closed

>`scrapy.signals.``spider_closed`（*蜘蛛*，*原因*）

蜘蛛关闭后发送。这可以用来释放保留的每个蜘蛛资源[`spider_opened`](https://doc.scrapy.org/en/latest/topics/signals.html#std:signal-spider_opened)。

该信号支持从处理程序返回延迟。

### spider_opened

>`scrapy.signals.``spider_opened`（*蜘蛛*）

在蜘蛛被打开爬行后发送。这通常用于保留每个蜘蛛资源，但可以用于打开蜘蛛时需要执行的任何任务。

该信号支持从处理程序返回延迟。

参数：

- **蜘蛛**（[`Spider`](https://doc.scrapy.org/en/latest/topics/spiders.html#scrapy.spiders.Spider)物体） - 已被打开的蜘蛛

### spider_idle

>`scrapy.signals.``spider_idle`（*蜘蛛*）

当蜘蛛闲置时发出，这意味着蜘蛛没有进一步的发现：

- 等待下载的请求
- 请求安排
- 项目流水线中正在处理的项目

如果在此信号的所有处理程序完成后，空闲状态仍然存在，则引擎开始关闭蜘蛛。蜘蛛完成关闭后，[`spider_closed`](https://doc.scrapy.org/en/latest/topics/signals.html#std:signal-spider_closed)发送信号。

你可以举一个[`DontCloseSpider`](https://doc.scrapy.org/en/latest/topics/exceptions.html#scrapy.exceptions.DontCloseSpider)例外来防止蜘蛛被关闭。

该信号不支持从处理程序返回延迟。

参数：

- **蜘蛛**（[`Spider`](https://doc.scrapy.org/en/latest/topics/spiders.html#scrapy.spiders.Spider)物体） - 空闲的蜘蛛

> **！注意**
>
> 在您的[`spider_idle`](https://doc.scrapy.org/en/latest/topics/signals.html#std:signal-spider_idle)处理程序中调度某些请求并 **不能**保证它可以防止蜘蛛被关闭，尽管有时可以。这是因为如果调度程序拒绝所有计划的请求（例如，由于重复进行过滤），则蜘蛛可能仍保持空闲状态。

### spider_error

>`scrapy.signals.``spider_error`（*失败*，*回应*，*蜘蛛*）

当一个蜘蛛回调产生一个错误（即引发一个异常）时发送。

该信号不支持从处理程序返回延迟。

参数：

- **失败**（[Failure](https://twistedmatrix.com/documents/current/api/twisted.python.failure.Failure.html)对象） - 作为Twisted [Failure](https://twistedmatrix.com/documents/current/api/twisted.python.failure.Failure.html)对象引发的异常
- **响应**（[`Response`](https://doc.scrapy.org/en/latest/topics/request-response.html#scrapy.http.Response)对象） - 引发异常时处理的响应
- **蜘蛛**（[`Spider`](https://doc.scrapy.org/en/latest/topics/spiders.html#scrapy.spiders.Spider)物体） - 引发异常的蜘蛛

### request_scheduled

>`scrapy.signals.``request_scheduled`（*请求*，*蜘蛛*）

发动机安排a时发送[`Request`](https://doc.scrapy.org/en/latest/topics/request-response.html#scrapy.http.Request)，稍后再下载。

该信号不支持从处理程序返回延迟。

参数：

- **请求**（[`Request`](https://doc.scrapy.org/en/latest/topics/request-response.html#scrapy.http.Request)对象） - 到达调度程序的请求
- **蜘蛛**（[`Spider`](https://doc.scrapy.org/en/latest/topics/spiders.html#scrapy.spiders.Spider)物体） - 产生请求的蜘蛛

### request_dropped

>`scrapy.signals.``request_dropped`（*请求*，*蜘蛛*）

当[`Request`](https://doc.scrapy.org/en/latest/topics/request-response.html#scrapy.http.Request)稍后下载的引擎安排的a 被调度程序拒绝时发送。

该信号不支持从处理程序返回延迟。

参数：

- **请求**（[`Request`](https://doc.scrapy.org/en/latest/topics/request-response.html#scrapy.http.Request)对象） - 到达调度程序的请求
- **蜘蛛**（[`Spider`](https://doc.scrapy.org/en/latest/topics/spiders.html#scrapy.spiders.Spider)物体） - 产生请求的蜘蛛

### response_received

`scrapy.signals.``response_received`（*回应*，*请求*，*蜘蛛*）

当引擎收到[`Response`](https://doc.scrapy.org/en/latest/topics/request-response.html#scrapy.http.Response)来自下载器的新消息时发送。

该信号不支持从处理程序返回延迟。

参数：

- **响应**（[`Response`](https://doc.scrapy.org/en/latest/topics/request-response.html#scrapy.http.Response)对象） - 收到的响应
- **请求**（[`Request`](https://doc.scrapy.org/en/latest/topics/request-response.html#scrapy.http.Request)对象） - 生成响应的请求
- **蜘蛛**（[`Spider`](https://doc.scrapy.org/en/latest/topics/spiders.html#scrapy.spiders.Spider)物体） - 反应意图的蜘蛛

### response_downloaded

> `scrapy.signals.``response_downloaded`（回应，请求，蜘蛛）

当引擎收到[`Response`](https://doc.scrapy.org/en/latest/topics/request-response.html#scrapy.http.Response)来自下载器的新消息时发送。

该信号不支持从处理程序返回延迟。

参数：

- **响应**（[`Response`](https://doc.scrapy.org/en/latest/topics/request-response.html#scrapy.http.Response)对象） - 下载的响应
- **请求**（[`Request`](https://doc.scrapy.org/en/latest/topics/request-response.html#scrapy.http.Request)对象） - 生成响应的请求
- **蜘蛛**（[`Spider`](https://doc.scrapy.org/en/latest/topics/spiders.html#scrapy.spiders.Spider)物体） - 反应意图的蜘蛛

