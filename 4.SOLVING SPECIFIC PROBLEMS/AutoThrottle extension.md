# 自动限速(AutoThrottle)扩展

该扩展能根据Scrapy服务器及您爬取的网站的负载自动限制爬取速度。

## 设计目标

1. 更友好的对待网站，而不使用默认的下载延迟0。
2. 自动调整scrapy来优化下载速度，使得用户不用调节下载延迟及并发请求数来找到优化的值。 用户只需指定允许的最大并发请求数，剩下的都交给扩展来完成。

## 怎么运行的

AutoThrottle扩展可动态调整下载延迟，以使蜘蛛向[`AUTOTHROTTLE_TARGET_CONCURRENCY`](https://doc.scrapy.org/en/latest/topics/autothrottle.html#std:setting-AUTOTHROTTLE_TARGET_CONCURRENCY)每个远程网站平均发送 并发请求。

它使用下载延迟来计算延迟。其主要思想是：如果一台服务器需要`latency`秒钟响应，客户端应该发送一个请求的每个`latency/N`秒，具有`N`并行处理的请求。

而不是调整延迟，可以设置一个小的固定下载延迟并对并发使用[`CONCURRENT_REQUESTS_PER_DOMAIN`](https://doc.scrapy.org/en/latest/topics/settings.html#std:setting-CONCURRENT_REQUESTS_PER_DOMAIN)或 [`CONCURRENT_REQUESTS_PER_IP`](https://doc.scrapy.org/en/latest/topics/settings.html#std:setting-CONCURRENT_REQUESTS_PER_IP)选项施加硬性限制 。它会提供类似的效果，但有一些重要的区别：

- 由于下载延迟很小，偶尔会出现一些请求;
- 通常非200（错误）响应可以比常规响应更快地返回，因此，在服务器开始返回错误时，如果下载延迟较小，并发限制爬网程序将更快地向服务器发送请求。但这与爬虫应该做的事情相反 - 如果发生错误，减慢速度更有意义：这些错误可能是由高请求率造成的。

AutoThrottle没有这些问题。

## 节流算法

AutoThrottle算法根据以下规则调整下载延迟：

1. 蜘蛛总是以下载延迟开始 [`AUTOTHROTTLE_START_DELAY`](https://doc.scrapy.org/en/latest/topics/autothrottle.html#std:setting-AUTOTHROTTLE_START_DELAY);
2. 当接收到响应时，目标下载延迟被计算为 其中响应的等待时间，并且是。`latency / N``latency``N`[`AUTOTHROTTLE_TARGET_CONCURRENCY`](https://doc.scrapy.org/en/latest/topics/autothrottle.html#std:setting-AUTOTHROTTLE_TARGET_CONCURRENCY)
3. 下次请求的下载延迟设置为上次下载延迟的平均值和目标下载延迟;
4. 不允许200个响应的延迟减少延迟;
5. 下载延迟不能小于[`DOWNLOAD_DELAY`](https://doc.scrapy.org/en/latest/topics/settings.html#std:setting-DOWNLOAD_DELAY)或大于[`AUTOTHROTTLE_MAX_DELAY`](https://doc.scrapy.org/en/latest/topics/autothrottle.html#std:setting-AUTOTHROTTLE_MAX_DELAY)

注意

AutoThrottle扩展支持标准Scrapy设置的并发性和延迟。这意味着它会尊重 [`CONCURRENT_REQUESTS_PER_DOMAIN`](https://doc.scrapy.org/en/latest/topics/settings.html#std:setting-CONCURRENT_REQUESTS_PER_DOMAIN)和 [`CONCURRENT_REQUESTS_PER_IP`](https://doc.scrapy.org/en/latest/topics/settings.html#std:setting-CONCURRENT_REQUESTS_PER_IP)选择，永远不会将下载延迟设置为低于[`DOWNLOAD_DELAY`](https://doc.scrapy.org/en/latest/topics/settings.html#std:setting-DOWNLOAD_DELAY)。

在Scrapy中，下载延迟时间的测量是建立TCP连接和接收HTTP头之间的时间。

请注意，这些延迟在协作式多任务环境中很难准确测量，因为Scrapy可能正忙于处理Spider回调，并且无法参加下载。但是，这些延迟仍应该对Scrapy（最终是服务器）的繁忙程度进行合理估计，并且此扩展建立在此前提之上。

## 设置

用于控制AutoThrottle扩展的设置是：

- [`AUTOTHROTTLE_ENABLED`](https://doc.scrapy.org/en/latest/topics/autothrottle.html#std:setting-AUTOTHROTTLE_ENABLED)
- [`AUTOTHROTTLE_START_DELAY`](https://doc.scrapy.org/en/latest/topics/autothrottle.html#std:setting-AUTOTHROTTLE_START_DELAY)
- [`AUTOTHROTTLE_MAX_DELAY`](https://doc.scrapy.org/en/latest/topics/autothrottle.html#std:setting-AUTOTHROTTLE_MAX_DELAY)
- [`AUTOTHROTTLE_TARGET_CONCURRENCY`](https://doc.scrapy.org/en/latest/topics/autothrottle.html#std:setting-AUTOTHROTTLE_TARGET_CONCURRENCY)
- [`AUTOTHROTTLE_DEBUG`](https://doc.scrapy.org/en/latest/topics/autothrottle.html#std:setting-AUTOTHROTTLE_DEBUG)
- [`CONCURRENT_REQUESTS_PER_DOMAIN`](https://doc.scrapy.org/en/latest/topics/settings.html#std:setting-CONCURRENT_REQUESTS_PER_DOMAIN)
- [`CONCURRENT_REQUESTS_PER_IP`](https://doc.scrapy.org/en/latest/topics/settings.html#std:setting-CONCURRENT_REQUESTS_PER_IP)
- [`DOWNLOAD_DELAY`](https://doc.scrapy.org/en/latest/topics/settings.html#std:setting-DOWNLOAD_DELAY)

有关更多信息，请参阅[它如何工作](https://doc.scrapy.org/en/latest/topics/autothrottle.html#autothrottle-algorithm)。

### AUTOTHROTTLE_ENABLED

默认： `False`

启用AutoThrottle扩展。

### AUTOTHROTTLE_START_DELAY

默认： `5.0`

最初的下载延迟（以秒为单位）。

### AUTOTHROTTLE_MAX_DELAY

默认： `60.0`

在高延迟情况下设置的最大下载延迟（以秒为单位）。

### AUTOTHROTTLE_TARGET_CONCURRENCY

版本1.1中的新功能

默认： `1.0`

Scrapy应平行发送到远程网站的平均请求数量。

默认情况下，AutoThrottle调整延迟以向每个远程网站发送单个并发请求。将此选项设置为更高的值（例如`2.0`）以增加吞吐量和远程服务器的负载。较低的`AUTOTHROTTLE_TARGET_CONCURRENCY`值（例如`0.5`）会使抓取工具更加保守和礼貌。

请注意，[`CONCURRENT_REQUESTS_PER_DOMAIN`](https://doc.scrapy.org/en/latest/topics/settings.html#std:setting-CONCURRENT_REQUESTS_PER_DOMAIN) 和[`CONCURRENT_REQUESTS_PER_IP`](https://doc.scrapy.org/en/latest/topics/settings.html#std:setting-CONCURRENT_REQUESTS_PER_IP)在启用AutoThrottle扩展选项仍然遵守。这意味着如果 `AUTOTHROTTLE_TARGET_CONCURRENCY`设置的值高于 [`CONCURRENT_REQUESTS_PER_DOMAIN`](https://doc.scrapy.org/en/latest/topics/settings.html#std:setting-CONCURRENT_REQUESTS_PER_DOMAIN)或者[`CONCURRENT_REQUESTS_PER_IP`](https://doc.scrapy.org/en/latest/topics/settings.html#std:setting-CONCURRENT_REQUESTS_PER_IP)，搜寻器将不会达到这个并发请求数。

在每个给定的时间点，Scrapy可以发送比或多或少的并发请求`AUTOTHROTTLE_TARGET_CONCURRENCY`; 它是爬虫尝试接近的建议值，而不是硬限制。

### AUTOTHROTTLE_DEBUG

默认： `False`

启用AutoThrottle调试模式，该模式将显示收到的每个响应的统计数据，以便您可以看到如何实时调整调节参数。