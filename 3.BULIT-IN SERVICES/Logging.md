# Logging
>Note
scrapy.log已被弃用，并且支持显式调用Python标准日志记录。 继续阅读以了解更多关于新日志记录系统的信息。

Scrapy使用Python的内置日志记录系统进行事件日志记录。 我们将提供一些简单的示例来帮助您开始，但对于更高级的用例，强烈建议您仔细阅读其文档。

日志功能可以直接使用，并且可以通过记录设置中列出的Scrapy设置进行一定程度的配置。

Scrapy调用scrapy.utils.log.configure_logging（）来设置合理的默认值，并在运行命令时处理Logging设置中的这些设置，所以如果您从脚本运行Scrapy（如运行Scrapy中描述的脚本），建议手动调用它。

## Log levels

Python的内建日志记录定义了5个不同的级别来指示给定日志消息的严重性。 这里是标准的，按递减顺序列出：

1.logging.CRITICAL - for critical errors (highest severity)
2.logging.ERROR - for regular errors
3.logging.WARNING - for warning messages
4.logging.INFO - for informational messages
5.logging.DEBUG - for debugging messages (lowest severity)

## How to log messages

以下是如何使用logging.WARNING级别记录消息的简单示例：
```
import logging
logging.warning("This is a warning")
```

有任何标准5级发布日志消息的快捷方式，还有一个通用的logging.log方法，它将给定级别作为参数。 如果需要，最后一个例子可以改写为：
```
import logging
logging.log(logging.WARNING, "This is a warning")
```

最重要的是，您可以创建不同的“记录器”来封装消息。 （例如，通常的做法是为每个模块创建不同的记录器）。 这些记录器可以独立配置，并且允许分层结构。

前面的示例在后台使用根记录器，这是所有消息传播到的顶级记录器（除非另有说明）。 使用日志助手仅仅是明确获取根日志记录器的捷径，所以这也是最后一个片段的等价物：
```
import logging
logger = logging.getLogger()
logger.warning("This is a warning")
```

您可以通过使用logging.getLogger函数获取其名称来使用其他记录器：

```
import logging
logger = logging.getLogger('mycustomlogger')
logger.warning("This is a warning")
```

最后，您可以确保使用__name__变量为您正在处理的任何模块创建自定义记录器，该变量填充当前模块的路径：

```
import logging
logger = logging.getLogger(__name__)
logger.warning("This is a warning")
```

>see also
模块日志，HowTo
基本记录教程
模块日志记录器
有关记录仪的进一步文件


## Logging from Spiders
Scrapy在每个Spider实例中提供一个记录器，可以像这样访问和使用它：

```
import scrapy

class MySpider(scrapy.Spider):

    name = 'myspider'
    start_urls = ['https://scrapinghub.com']

    def parse(self, response):
        self.logger.info('Parse function called on %s', response.url)
```

该记录器是使用Spider的名称创建的，但您可以使用任何您想要的自定义Python记录器。 例如：

```
import logging
import scrapy

logger = logging.getLogger('mycustomlogger')

class MySpider(scrapy.Spider):

    name = 'myspider'
    start_urls = ['https://scrapinghub.com']

    def parse(self, response):
        logger.info('Parse function called on %s', response.url)
```

## Logging configuration
记录器自己不管理如何显示通过它们发送的消息。 对于这项任务，可以将不同的“处理程序”附加到任何记录程序实例，并将这些消息重定向到适当的目标，例如标准输出，文件，电子邮件等。

默认情况下，Scrapy根据下面的设置设置和配置根记录器的处理程序。

### Logging settings
这些设置可用于配置日志记录：
* LOG_FILE
* LOG_ENABLED
* LOG_ENCODING
* LOG_LEVEL
* LOG_FORMAT
* LOG_DATEFORMAT
* LOG_STDOUT
* LOG_SHORT_NAMES

第一对设置定义了日志消息的目的地。 如果设置了LOG_FILE，则通过根记录器发送的消息将被重定向到LOG_ENCODING编码的名为LOG_FILE的文件。 如果未设置且LOG_ENABLED为True，则日志消息将显示在标准错误上。 最后，如果LOG_ENABLED为False，则不会有任何可见的日志输出。

LOG_LEVEL确定要显示的最低严重级别，这些严重级别较低的消息将被过滤掉。 它的范围通过日志级别中列出的可能级别。

LOG_FORMAT和LOG_DATEFORMAT指定用作所有消息布局的格式化字符串。 这些字符串可以包含日志的logrecord属性文档和日期时间的strftime和strptime指令中分别列出的任何占位符。

如果设置了LOG_SHORT_NAMES，那么日志将不会显示打印日志的scrapy组件。 它在默认情况下是未设置的，因此日志包含负责该日志输出的scrapy组件。

### Command-line options

有一些命令行参数可用于所有命令，您可以使用它们来覆盖有关日志记录的一些Scrapy设置。
--logfile FILE
		Overrides LOG_FILE
--loglevel/-L LEVEL
		Overrides LOG_LEVEL
--nolog
		Sets LOG_ENABLED to False
>See also
Module logging.handlers
Further documentation on available handlers

### Advanced customization
由于Scrapy使用stdlib日志记录模块，因此可以使用stdlib日志记录的所有功能自定义日志记录。

例如，假设您正在抓取一个返回许多HTTP 

>2016-12-16 22:00:06 [scrapy.spidermiddlewares.httperror] INFO: Ignoring
response <500 http://quotes.toscrape.com/page/1-34/>: HTTP status code
is not handled or not allowed

首先要注意的是记录器名称 - 它位于括号内：[scrapy.spidermiddlewares.httperror]。 如果你只是[scrapy]，那么LOG_SHORT_NAMES可能设置为True; 将其设置为False并重新运行爬网。

接下来，我们可以看到该消息具有INFO级别。 为了隐藏它，我们应该为高于INFO的scrapy.spidermiddlewares.httperror设置日志记录级别; INFO之后的下一个级别是WARNING。 它可以完成例如 在蜘蛛的__init__方法中：

```
import logging
import scrapy


class MySpider(scrapy.Spider):
    # ...
    def __init__(self, *args, **kwargs):
        logger = logging.getLogger('scrapy.spidermiddlewares.httperror')
        logger.setLevel(logging.WARNING)
        super().__init__(*args, **kwargs)
```
如果您再次运行此蜘蛛，则来自scrapy.spidermiddlewares.httperror记录器的INFO消息将消失。

## scrapy.utils.log module
>scrapy.utils.log.configure_logging(settings=None, install_root_handler=True)

初始化Scrapy的日志记录默认值。

参数：
设置（字典，设置对象或无） - 用于为根记录器创建和配置处理程序的设置（默认值：无）。
install_root_handler（bool） - 是否安装根日志处理程序（默认值：True）
这个功能确实：

通过Python标准日志记录路由警告和扭曲日志记录
分别为Scrapy和Twisted记录器分配DEBUG和ERROR级别
如果LOG_STDOUT设置为True，则路由stdout以记录日志
当install_root_handler为True（默认）时，此函数还会根据给定设置为根记录器创建一个处理程序（请参阅记录设置）。您可以使用设置参数覆盖默认选项。当设置为空或无时，使用默认值。

在使用Scrapy命令时会自动调用configure_logging，但在运行自定义脚本时需要显式调用configure_logging。在这种情况下，它的使用不是必需的，但建议。

如果您打算自己配置处理程序仍然建议您调用此函数，传递install_root_handler = False。请记住，在这种情况下，默认情况下不会设置任何日志输出。

为了让您开始手动配置日志记录的输出，您可以使用logging.basicConfig（）来设置基本的根处理程序。这是如何将INFO或更高版本的消息重定向到文件的示例：	
```
import logging
from scrapy.utils.log import configure_logging

configure_logging(install_root_handler=False)
logging.basicConfig(
    filename='log.txt',
    format='%(levelname)s: %(message)s',
    level=logging.INFO
)
```

请参阅脚本中的运行Scrapy，以获取更多关于如何使用Scrapy的详细信息。