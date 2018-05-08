有时，对于大型网站，最好暂停爬网并稍后恢复。

Scrapy通过提供以下功能支持此功能：

- 一个在磁盘上保存预定请求的调度程序
- 一个重复过滤器，它保留磁盘上的访问请求
- 批次间保持一些蜘蛛状态（键/值对）的扩展

### 工作目录

要启用持久性支持，只需通过设置定义一个*作业目录即可*`JOBDIR`。该目录将用于存储所有必需的数据以保持单个作业的状态（即蜘蛛程序运行）。需要注意的是，该目录不能由不同的蜘蛛共享，甚至不能由同一个蜘蛛的不同作业/运行共享，因为它意在用于存储*单个*作业的状态。

### 如何使用它

要启用支持启用持久性的spider，请像这样运行它：

```python
scrapy crawl somespider -s JOBDIR=crawls/somespider-1
```

然后，您可以随时安全地停止蜘蛛（通过按Ctrl-C或发送信号），稍后通过发出相同的命令来恢复它：

```Python
scrapy crawl somespider -s JOBDIR=crawls/somespider-1
```

### 在批次之间保持持久状态

有时你会想要在暂停/恢复批次之间保持一些持久的蜘蛛状态。你可以使用该`spider.state`属性，这应该是一个字典。当蜘蛛启动和停止时，有一个内置的扩展，负责序列化，存储和加载作业目录中的属性。

下面是一个使用蜘蛛状态的回调示例（为简洁起见，其他蜘蛛代码被省略）：

```Python
def parse_item(self, response):
    # parse item here
    self.state['items_count'] = self.state.get('items_count', 0) + 1
```

### 持久性陷阱

如果您希望能够使用Scrapy持久性支持，请注意以下几点：

#### Cookies过期

Cookie可能会过期。所以，如果你不能迅速恢复你的蜘蛛，预定的请求可能不再有效。如果你的蜘蛛不依赖cookies，这不会成为问题。

#### 请求序列化

请求必须由pickle模块序列化，以便持久化工作，所以你应该确保你的请求是可序列化的。

这里最常见的问题是`lambda`在请求回调中使用不能被持久化的函数。

所以，例如，这不起作用：

```python
def some_callback(self, response):
    somearg = 'test'
    return scrapy.Request('http://www.example.com', callback=lambda r: self.other_callback(r, somearg))

def other_callback(self, response, somearg):
    print "the argument passed is:", somearg
```

但是这将会：

```python
def some_callback(self, response):
    somearg = 'test'
    return scrapy.Request('http://www.example.com', callback=self.other_callback, meta={'somearg': somearg})

def other_callback(self, response):
    somearg = response.meta['somearg']
    print "the argument passed is:", somearg
```

如果您希望记录无法序列化的请求，可以将该 [`SCHEDULER_DEBUG`](https://doc.scrapy.org/en/latest/topics/settings.html#std:setting-SCHEDULER_DEBUG)设置设置为`True`在项目的设置页面中。这是`False`默认的。

