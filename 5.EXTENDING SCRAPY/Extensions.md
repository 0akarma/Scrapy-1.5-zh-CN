扩展框架提供了一种将您自己的自定义功能插入到Scrapy中的机制。

扩展只是在扩展初始化时在Scrapy启动时实例化的常规类。

### 扩展设置

与其他Scrapy代码一样，扩展使用[Scrapy设置](https://doc.scrapy.org/en/latest/topics/settings.html#topics-settings)来管理其设置。

扩展名通常以其自己的名称作为其设置的前缀，以避免与现有（和未来）扩展名冲突。例如，处理[Google Sitemaps](https://en.wikipedia.org/wiki/Sitemaps)的假设性扩展会使用像GOOGLESITEMAP_ENABLED，GOOGLESITEMAP_DEPTH等设置 。

### 加载和激活扩展

扩展在启动时通过实例化扩展类的单个实例来加载和激活。因此，所有的扩展初始化代码必须在类构造函数（`__init__`方法）中执行。

要使扩展程序可用，请将其添加到[`EXTENSIONS`](https://doc.scrapy.org/en/latest/topics/settings.html#std:setting-EXTENSIONS)Scrapy设置中的设置。在中[`EXTENSIONS`](https://doc.scrapy.org/en/latest/topics/settings.html#std:setting-EXTENSIONS)，每个扩展都用一个字符串表示：扩展类名的完整Python路径。例如：

```
EXTENSIONS  =  { 
    'scrapy.extensions.corestats.CoreStats' ： 500 ，
    'scrapy.extensions.telnet.TelnetConsole' ： 500 ，
}
```

如您所见，该[`EXTENSIONS`](https://doc.scrapy.org/en/latest/topics/settings.html#std:setting-EXTENSIONS)设置是一个字典，其中的键是扩展路径，它们的值是定义扩展*加载*顺序的顺序。该[`EXTENSIONS`](https://doc.scrapy.org/en/latest/topics/settings.html#std:setting-EXTENSIONS)设置与[`EXTENSIONS_BASE`](https://doc.scrapy.org/en/latest/topics/settings.html#std:setting-EXTENSIONS_BASE)Scrapy中定义的设置合并 （并不意味着被覆盖），然后按顺序排序以获得最终的已启用扩展的排序列表。

由于扩展通常不相互依赖，因此在大多数情况下，加载顺序无关紧要。这就是为什么该[`EXTENSIONS_BASE`](https://doc.scrapy.org/en/latest/topics/settings.html#std:setting-EXTENSIONS_BASE)设置使用相同的顺序（`0`）定义所有扩展。但是，如果您需要添加取决于已加载的其他扩展的扩展，则可以利用此功能。

### 可用，启用和禁用的扩展

并非所有可用的分机都将启用。其中一些通常取决于特定的设置。例如，默认情况下，HTTP Cache扩展可用，但除非[`HTTPCACHE_ENABLED`](https://doc.scrapy.org/en/latest/topics/downloader-middleware.html#std:setting-HTTPCACHE_ENABLED)设置已设置，否则禁用。

### 禁用扩展

为了禁用默认启用的扩展（即[`EXTENSIONS_BASE`](https://doc.scrapy.org/en/latest/topics/settings.html#std:setting-EXTENSIONS_BASE)设置中包含的扩展），您必须将其顺序设置为`None`。例如：

```
EXTENSIONS  =  { 
    'scrapy.extensions.corestats.CoreStats' ： 无，
}
```

### 编写自己的扩展

每个扩展都是一个Python类。Scrapy扩展的主要入口点（这也包括中间件和管道）是`from_crawler`接收`Crawler`实例的类方法。通过Crawler对象，您可以访问设置，信号，统计信息并控制爬行行为。

通常，分机连接到[信号](https://doc.scrapy.org/en/latest/topics/signals.html#topics-signals)并执行由它们触发的任务。

最后，如果该`from_crawler`方法引发 [`NotConfigured`](https://doc.scrapy.org/en/latest/topics/exceptions.html#scrapy.exceptions.NotConfigured)异常，则扩展将被禁用。否则，该扩展将被启用。

### 示例扩展

这里我们将实现一个简单的扩展来说明上一节中描述的概念。每次该扩展都会记录一条消息：

- 一只蜘蛛被打开
- 一只蜘蛛关闭
- 特定数量的项目被刮掉

该分机将通过`MYEXT_ENABLED`设置启用，并且项目数量将通过`MYEXT_ITEMCOUNT`设置指定。

这是这种扩展的代码：

```python
import logging
from scrapy import signals
from scrapy.exceptions import NotConfigured

logger = logging.getLogger(__name__)

class SpiderOpenCloseLogging(object):

    def __init__(self, item_count):
        self.item_count = item_count
        self.items_scraped = 0

    @classmethod
    def from_crawler(cls, crawler):
        # first check if the extension should be enabled and raise
        # NotConfigured otherwise
        if not crawler.settings.getbool('MYEXT_ENABLED'):
            raise NotConfigured

        # get the number of items from settings
        item_count = crawler.settings.getint('MYEXT_ITEMCOUNT', 1000)

        # instantiate the extension object
        ext = cls(item_count)

        # connect the extension object to signals
        crawler.signals.connect(ext.spider_opened, signal=signals.spider_opened)
        crawler.signals.connect(ext.spider_closed, signal=signals.spider_closed)
        crawler.signals.connect(ext.item_scraped, signal=signals.item_scraped)

        # return the extension object
        return ext

    def spider_opened(self, spider):
        logger.info("opened spider %s", spider.name)

    def spider_closed(self, spider):
        logger.info("closed spider %s", spider.name)

    def item_scraped(self, item, spider):
        self.items_scraped += 1
        if self.items_scraped % self.item_count == 0:
            logger.info("scraped %d items", self.items_scraped)
```

## 内置扩展参考

### 通用扩展

#### 日志统计扩展

>*class*`scrapy.extensions.logstats.``LogStats`

记录抓取的页面和抓取的项目等基本统计信息。

#### 核心统计扩展

>*class*`scrapy.extensions.corestats.``CoreStats`

如果启用了统计信息收集，请启用核心统计信息收集（请参阅[统计信息收集](https://doc.scrapy.org/en/latest/topics/stats.html#topics-stats)）。

#### Telnet控制台扩展

>*class*`scrapy.extensions.telnet.``TelnetConsole`

提供一个Telnet控制台，用于在当前运行的Scrapy过程中进入Python解释器，这对调试非常有用。

远程登录控制台必须通过该[`TELNETCONSOLE_ENABLED`](https://doc.scrapy.org/en/latest/topics/settings.html#std:setting-TELNETCONSOLE_ENABLED) 设置启用，服务器将监听端口中指定的端口[`TELNETCONSOLE_PORT`](https://doc.scrapy.org/en/latest/topics/telnetconsole.html#std:setting-TELNETCONSOLE_PORT)。

#### 内存使用扩展

>*class*`scrapy.extensions.memusage.``MemoryUsage`

>**！注意**
>
>此扩展在Windows中不起作用。

监控运行蜘蛛的Scrapy进程使用的内存，并：

1. 超过某个值时发送通知电子邮件
2. 超过一定值时关闭蜘蛛

当达到某个警告值（[`MEMUSAGE_WARNING_MB`](https://doc.scrapy.org/en/latest/topics/settings.html#std:setting-MEMUSAGE_WARNING_MB)）并且达到最大值（）时，通知电子邮件可以被触发，[`MEMUSAGE_LIMIT_MB`](https://doc.scrapy.org/en/latest/topics/settings.html#std:setting-MEMUSAGE_LIMIT_MB)这也会导致蜘蛛被关闭并且Scrapy进程被终止。

该扩展程序由该[`MEMUSAGE_ENABLED`](https://doc.scrapy.org/en/latest/topics/settings.html#std:setting-MEMUSAGE_ENABLED)设置启用，并且可以使用以下设置进行配置：

- [`MEMUSAGE_LIMIT_MB`](https://doc.scrapy.org/en/latest/topics/settings.html#std:setting-MEMUSAGE_LIMIT_MB)
- [`MEMUSAGE_WARNING_MB`](https://doc.scrapy.org/en/latest/topics/settings.html#std:setting-MEMUSAGE_WARNING_MB)
- [`MEMUSAGE_NOTIFY_MAIL`](https://doc.scrapy.org/en/latest/topics/settings.html#std:setting-MEMUSAGE_NOTIFY_MAIL)
- [`MEMUSAGE_CHECK_INTERVAL_SECONDS`](https://doc.scrapy.org/en/latest/topics/settings.html#std:setting-MEMUSAGE_CHECK_INTERVAL_SECONDS)

#### 内存调试器扩展

>*class*`scrapy.extensions.memdebug.``MemoryDebugger`

调试内存使用的扩展。它收集有关以下信息：

- Python垃圾收集器未收集的对象
- 不应该留下活着的物体。有关更多信息，请参阅[使用trackref调试内存泄漏](https://doc.scrapy.org/en/latest/topics/leaks.html#topics-leaks-trackrefs)

要启用此扩展程序，请打开[`MEMDEBUG_ENABLED`](https://doc.scrapy.org/en/latest/topics/settings.html#std:setting-MEMDEBUG_ENABLED)设置。信息将存储在统计信息中。

#### 关闭蜘蛛扩展

>*class*`scrapy.extensions.closespider.``CloseSpide`

在满足某些条件时自动关闭蜘蛛，并针对每种情况使用特定的关闭原因。

关闭蜘蛛的条件可以通过以下设置进行配置：

- [`CLOSESPIDER_TIMEOUT`](https://doc.scrapy.org/en/latest/topics/extensions.html#std:setting-CLOSESPIDER_TIMEOUT)
- [`CLOSESPIDER_ITEMCOUNT`](https://doc.scrapy.org/en/latest/topics/extensions.html#std:setting-CLOSESPIDER_ITEMCOUNT)
- [`CLOSESPIDER_PAGECOUNT`](https://doc.scrapy.org/en/latest/topics/extensions.html#std:setting-CLOSESPIDER_PAGECOUNT)
- [`CLOSESPIDER_ERRORCOUNT`](https://doc.scrapy.org/en/latest/topics/extensions.html#std:setting-CLOSESPIDER_ERRORCOUNT)

##### CLOSESPIDER_TIMEOUT

默认： `0`

一个指定秒数的整数。如果蜘蛛保持打开的时间超过这个秒数，它将会自动关闭`closespider_timeout`。如果为零（或未设置），则蜘蛛将不会因超时而关闭。

##### CLOSESPIDER_ITEMCOUNT

默认： `0`

一个指定项目数量的整数。如果蜘蛛的刮伤量超过了这个数量，并且这些物品通过了物品管道，蜘蛛将会被关闭`closespider_itemcount`。目前处于下载队列中的[`CONCURRENT_REQUESTS`](https://doc.scrapy.org/en/latest/topics/settings.html#std:setting-CONCURRENT_REQUESTS)请求（直至 请求）仍然处理。如果为零（或未设置），蜘蛛将不会按传递项目的数量关闭。

##### CLOSESPIDER_PAGECOUNT

0.11版本的新功能。

默认： `0`

一个整数，指定要抓取的最大响应数。如果蜘蛛爬得比这更多，蜘蛛就会被原因关闭`closespider_pagecount`。如果为零（或未设置），则爬网的响应数量将不会关闭蜘蛛。

##### CLOSESPIDER_ERRORCOUNT

0.11版本的新功能。

默认： `0`

一个整数，指定在关闭蜘蛛之前接收的最大错误数。如果蜘蛛生成的错误数量超过了这个数量，它将会因为原因而被关闭`closespider_errorcount`。如果为零（或未设置），则蜘蛛将不会按错误数量关闭。

#### StatsMailer扩展

>*class*`scrapy.extensions.statsmailer.``StatsMailer`

这个简单的扩展可用于每次域名完成抓取时发送通知电子邮件，包括收集的Scrapy统计信息。电子邮件将发送给[`STATSMAILER_RCPTS`](https://doc.scrapy.org/en/latest/topics/settings.html#std:setting-STATSMAILER_RCPTS) 设置中指定的所有收件人。

### 调试扩展

#### 堆栈跟踪转储扩展

> *class*scrapy.extensions.debug.``StackTraceDump

在 收到[SIGQUIT](https://en.wikipedia.org/wiki/SIGQUIT)或[SIGUSR2](https://en.wikipedia.org/wiki/SIGUSR1_and_SIGUSR2)信号时转储有关正在运行的进程的信息。所甩的信息如下：

1. 引擎状态（使用`scrapy.utils.engine.get_engine_status()`）
2. 活动引用（请参阅[使用trackref调试内存泄漏](https://doc.scrapy.org/en/latest/topics/leaks.html#topics-leaks-trackrefs)）
3. 所有线程的堆栈跟踪

堆栈跟踪和引擎状态转储后，Scrapy进程继续正常运行。

此扩展只适用于符合POSIX的平台（即非Windows），因为[SIGQUIT](https://en.wikipedia.org/wiki/SIGQUIT)和[SIGUSR2](https://en.wikipedia.org/wiki/SIGUSR1_and_SIGUSR2)信号在Windows上不可用。

至少有两种方式向Scrapy发送[SIGQUIT](https://en.wikipedia.org/wiki/SIGQUIT)信号：

1. 在Scrapy进程运行时按下Ctrl键（仅限Linux？）
2. 通过运行这个命令（假设`<pid>`是Scrapy进程的进程ID）：

```
kill -QUIT <pid>
```

#### 调试器扩展

*class*`scrapy.extensions.debug.``Debugger`

在 收到[SIGUSR2](https://en.wikipedia.org/wiki/SIGUSR1_and_SIGUSR2)信号时，在正在运行的Scrapy过程中调用[Python调试器](https://docs.python.org/2/library/pdb.html)。调试器退出后，Scrapy进程继续正常运行。

有关更多信息，请参阅Python中的调试。

此扩展只适用于符合POSIX的平台（即非Windows）。

- 