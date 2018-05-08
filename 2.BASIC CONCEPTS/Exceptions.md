# 异常(Exceptions)
## 内置异常参考手册(Built-in Exceptions reference)
下面是Scrapy提供的异常及其用法。

### DropItem
>exception scrapy.exceptions.DropItem

该异常由item pipeline抛出，用于停止处理item。详细内容请参考 Item Pipeline 。

### CloseSpider
>exception scrapy.exceptions.CloseSpider(reason='cancelled')

该异常由spider的回调函数(callback)抛出，来暂停/停止spider。支持的参数:

参数:	reason (str) – 关闭的原因
样例:

```def parse_page(self, response):
    if 'Bandwidth exceeded' in response.body:
        raise CloseSpider('bandwidth_exceeded')

```

### IgnoreRequest
>exception scrapy.exceptions.IgnoreRequest

该异常由调度器(Scheduler)或其他下载中间件抛出，声明忽略该request。

### NotConfigured
>exception scrapy.exceptions.NotConfigured

该异常由某些组件抛出，声明其仍然保持关闭。这些组件包括:

### NotConfigured
>exception scrapy.exceptions.NotConfigured

某些组件可能会引发此异常，以表明它们将保持禁用状态。 这些组件包括：
* Extensions
* Item pipelines
* Downloader middlewares
* Spider middlewares
必须在组件的__init__方法中引发异常。

### NotSupported
>exception scrapy.exceptions.NotSupported

该异常声明一个不支持的特性。