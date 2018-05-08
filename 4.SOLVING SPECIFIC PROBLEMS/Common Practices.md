### 实践经验  
本章节记录了使用Scrapy的一些实践经验(common practices)。 这包含了很多使用不会包含在其他特定章节的的内容。  
#### 在脚本中运行Scrapy  
除了常用的 scrapy crawl 来启动Scrapy，您也可以使用 API 在脚本中启动Scrapy。

需要注意的是，Scrapy是在Twisted异步网络库上构建的， 因此其必须在Twisted reactor里运行。  
您可以用来运行您的爬行器的第一个实用程序 scrapy.crawler.CrawlerProcess。该类将为您启动一个Twisted反应器，配置日志记录并设置关闭处理程序。这个类是所有Scrapy命令使用的。
这里有一个例子，说明如何用它来运行一只爬虫： 

```
import scrapy
from scrapy.crawler import CrawlerProcess

class MySpider(scrapy.Spider):
    # Your spider definition
    ...

process = CrawlerProcess({
    'USER_AGENT': 'Mozilla/4.0 (compatible; MSIE 7.0; Windows NT 5.1)'
})

process.crawl(MySpider)
process.start() # the script will block here until the crawling is finished
```
确保检查CrawlerProcess文档了解其使用细节。  
如果您在一个剪贴项目中，您可以使用一些额外的助手来导入项目中的组件。您可以自动导入您的爬行器，将它们的名称传递给CrawlerProcess，并使get_project_settings来获得您的项目设置的Settings实例。  
下面是一个工作示例，说明如何使用testspiders项目作为示例。  

```
from scrapy.crawler import CrawlerProcess
from scrapy.utils.project import get_project_settings

process = CrawlerProcess(get_project_settings())

# 'followall' is the name of one of the spiders of the project.
process.crawl('followall', domain='scrapinghub.com')
process.start() # the script will block here until the crawling is finished
```
还有另一种可以为爬行过程提供更多控制的工具:scratch .crawler. crawlerrunner。这个类是一个封装了一些简单的助手来运行多个爬虫的包装器，但是它不会以任何方式启动或干扰现有的运行。  
使用这个类，应该在调度您的爬虫后显式地运行反应器。如果您的应用程序已经使用Twisted，并且您想在同一个反应器中运行Scrapy，建议您使用CrawlerRunner而不是CrawlerProcess。  
注意，在爬行完成后，你还必须自己关闭Twisted的反应器。这可以通过CrawlerRunner.crawl方法将回调函数添加到延迟返回来实现。
这里是它的用法示例，以及在MySpider完成运行后手动停止反应器的回调。  
```
from twisted.internet import reactor
import scrapy
from scrapy.crawler import CrawlerRunner
from scrapy.utils.log import configure_logging

class MySpider(scrapy.Spider):
    # Your spider definition
    ...

configure_logging({'LOG_FORMAT': '%(levelname)s: %(message)s'})
runner = CrawlerRunner()

d = runner.crawl(MySpider)
d.addBoth(lambda _: reactor.stop())
reactor.run() # the script will block here until the crawling is finished
```
#### 同一进程运行多个spider  
默认情况下，当您执行 scrapy crawl 时，Scrapy每个进程运行一个spider。 当然，Scrapy通过 内部(internal)API 也支持单进程多个spider。  
这里有一个同时运行多个爬行器的例子。  

```
import scrapy
from scrapy.crawler import CrawlerProcess

class MySpider1(scrapy.Spider):
    # Your first spider definition
    ...

class MySpider2(scrapy.Spider):
    # Your second spider definition
    ...

process = CrawlerProcess()
process.crawl(MySpider1)
process.crawl(MySpider2)
process.start() # the script will block here until all crawling jobs are finished
```
相同的例子,使用CrawlerRunner:

```
import scrapy
from twisted.internet import reactor
from scrapy.crawler import CrawlerRunner
from scrapy.utils.log import configure_logging

class MySpider1(scrapy.Spider):
    # Your first spider definition
    ...

class MySpider2(scrapy.Spider):
    # Your second spider definition
    ...

configure_logging()
runner = CrawlerRunner()
runner.crawl(MySpider1)
runner.crawl(MySpider2)
d = runner.join()
d.addBoth(lambda _: reactor.stop())

reactor.run() # the script will block here until all crawling jobs are finished
```
同样的例子，但是通过链接延迟来运行。

```
from twisted.internet import reactor, defer
from scrapy.crawler import CrawlerRunner
from scrapy.utils.log import configure_logging

class MySpider1(scrapy.Spider):
    # Your first spider definition
    ...

class MySpider2(scrapy.Spider):
    # Your second spider definition
    ...

configure_logging()
runner = CrawlerRunner()

@defer.inlineCallbacks
def crawl():
    yield runner.crawl(MySpider1)
    yield runner.crawl(MySpider2)
    reactor.stop()

crawl()
reactor.run() # the script will block here until the last crawl call is finished
```