本文档描述了Scrapy的架构以及它的组件如何交互。

### 概观

下图显示了Scrapy体系结构及其组件的概述以及在系统内部发生的数据流概述（用红色箭头表示）。下面将对这些组件进行简要说明，并提供有关这些组件的更多详细信息的链接。数据流也在下面描述。

### 数据流

<img src='https://doc.scrapy.org/en/latest/_images/scrapy_architecture_02.png'>

Scrapy中的数据流由执行引擎控制，如下所示：

1. 该[引擎](https://doc.scrapy.org/en/latest/topics/architecture.html#component-engine)获得初始请求从抓取 [蜘蛛](https://doc.scrapy.org/en/latest/topics/architecture.html#component-spiders)。
2. 该[引擎](https://doc.scrapy.org/en/latest/topics/architecture.html#component-engine)安排在请求 [调度程序](https://doc.scrapy.org/en/latest/topics/architecture.html#component-scheduler)和要求下一个请求抓取。
3. 该[计划](https://doc.scrapy.org/en/latest/topics/architecture.html#component-scheduler)返回下一请求的[引擎](https://doc.scrapy.org/en/latest/topics/architecture.html#component-engine)。
4. 该[引擎](https://doc.scrapy.org/en/latest/topics/architecture.html#component-engine)发送请求到 [下载器](https://doc.scrapy.org/en/latest/topics/architecture.html#component-downloader)，通过 [下载器中间件](https://doc.scrapy.org/en/latest/topics/architecture.html#component-downloader-middleware)（见 [`process_request()`](https://doc.scrapy.org/en/latest/topics/downloader-middleware.html#scrapy.downloadermiddlewares.DownloaderMiddleware.process_request)）。
5. 一旦页面完成下载， [Downloader会](https://doc.scrapy.org/en/latest/topics/architecture.html#component-downloader)生成一个响应（包含该页面）并将其发送到引擎，并通过[Downloader Middlewares](https://doc.scrapy.org/en/latest/topics/architecture.html#component-downloader-middleware)（请参阅 [参考资料](https://doc.scrapy.org/en/latest/topics/architecture.html#component-downloader-middleware)[`process_response()`](https://doc.scrapy.org/en/latest/topics/downloader-middleware.html#scrapy.downloadermiddlewares.DownloaderMiddleware.process_response)）。
6. 该[引擎](https://doc.scrapy.org/en/latest/topics/architecture.html#component-engine)接收来自响应 [下载器](https://doc.scrapy.org/en/latest/topics/architecture.html#component-downloader)并将其发送到所述 [蜘蛛](https://doc.scrapy.org/en/latest/topics/architecture.html#component-spiders)进行处理，通过[蜘蛛中间件](https://doc.scrapy.org/en/latest/topics/architecture.html#component-spider-middleware)（见[`process_spider_input()`](https://doc.scrapy.org/en/latest/topics/spider-middleware.html#scrapy.spidermiddlewares.SpiderMiddleware.process_spider_input)）。
7. [Spider](https://doc.scrapy.org/en/latest/topics/architecture.html#component-spiders)处理响应，并通过 [蜘蛛中间件](https://doc.scrapy.org/en/latest/topics/architecture.html#component-spider-middleware)（请参阅 [参考资料](https://doc.scrapy.org/en/latest/topics/architecture.html#component-spider-middleware)）返回抓取的项目和新的请求（接下来）到 [引擎](https://doc.scrapy.org/en/latest/topics/architecture.html#component-engine)。[`process_spider_output()`](https://doc.scrapy.org/en/latest/topics/spider-middleware.html#scrapy.spidermiddlewares.SpiderMiddleware.process_spider_output)
8. 该[引擎](https://doc.scrapy.org/en/latest/topics/architecture.html#component-engine)发送处理的项目，以 [项目管道](https://doc.scrapy.org/en/latest/topics/architecture.html#component-pipelines)，然后把处理的请求的[调度](https://doc.scrapy.org/en/latest/topics/architecture.html#component-scheduler)，并要求今后可能要求抓取。
9. 该过程重复（从第1步开始），直到[调度程序](https://doc.scrapy.org/en/latest/topics/architecture.html#component-scheduler)没有更多请求 。

### 组件

#### Scrapy引擎

引擎负责控制系统所有组件之间的数据流，并在发生某些操作时触发事件。有关更多详细信息，请参阅上面的 [数据流](https://doc.scrapy.org/en/latest/topics/architecture.html#data-flow)部分

#### 调度

调度程序接收来自引擎的请求，并将它们排入队列，以便在引擎请求它们时将它们提供给它们（也引擎）。

#### 下载

下载器负责获取网页并将它们馈送到引擎，然后引擎将它们馈送给蜘蛛。

#### 蜘蛛

蜘蛛程序是由Scrapy用户编写的自定义类，用于解析响应并从中提取项目（也称为抓取的项目）或追加的其他请求。欲了解更多信息，请参阅[蜘蛛](https://doc.scrapy.org/en/latest/topics/spiders.html#topics-spiders)。

#### 物品管道

物品管道负责处理物品，一旦它们被蜘蛛提取（或刮掉）。典型的任务包括清理，验证和持久性（如将项目存储在数据库中）。欲了解更多信息，请参阅[项目管道](https://doc.scrapy.org/en/latest/topics/item-pipeline.html#topics-item-pipeline)。

#### 下载中间件

下载器中间件是位于引擎和下载器之间的特定钩子，当它们从引擎传递到下载器时处理请求，以及从下载器传递到引擎的响应。

如果您需要执行以下操作之一，请使用Downloader中间件：

- 在将请求发送到Downloader之前处理请求（即，在Scrapy将请求发送到网站之前）;
- 在将其传递给蜘蛛之前改变接收到的响应;
- 发送新的请求，而不是将接收到的响应传递给蜘蛛;
- 向蜘蛛传递响应而不需要获取网页;
- 默默地放下一些请求。

欲了解更多信息，请参阅[下载中间件](https://doc.scrapy.org/en/latest/topics/downloader-middleware.html#topics-downloader-middleware)。

#### 蜘蛛中间件

蜘蛛中间件是引擎和蜘蛛之间的特定钩子，能够处理蜘蛛输入（响应）和输出（项目和请求）。

如果需要，请使用Spider中间件

- spider回调的后处理输出 - 更改/添加/删除请求或项目;
- 后处理start_requests;
- 处理蜘蛛异常;
- 根据响应内容为一些请求调用errback而不是回叫。

欲了解更多信息，请参阅[蜘蛛中间件](https://doc.scrapy.org/en/latest/topics/spider-middleware.html#topics-spider-middleware)。

### 事件驱动的网络

Scrapy是用[Twisted](https://twistedmatrix.com/trac/)编写的，这是一个流行的事件驱动的Python网络框架。因此，它使用非阻塞（又称异步）代码来实现并发。

有关异步编程和Twisted的更多信息，请参阅以下链接：

- [扭曲的延期介绍](https://twistedmatrix.com/documents/current/core/howto/defer-intro.html)
- [扭曲 - 你好，异步编程](http://jessenoller.com/2009/02/11/twisted-hello-asynchronous-programming/)
- [扭曲的介绍 - Krondo](http://krondo.com/an-introduction-to-asynchronous-programming-and-twisted/)

