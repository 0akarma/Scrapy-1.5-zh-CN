### 远程登录控制  
scrapy附带了一个内置的telnet控制台，用于检查和控制scrapy运行过程。telnet控制台只是一个常规的python shell，它运行在scrapy过程中，可以在上面做任何事情。
telnet控制台是一个内置的scrapy扩展，它在默认情况下是启用的，但是如果您愿意，也可以禁用它。有关扩展本身的更多信息，请参阅Telnet控制台扩展。 
#### 如何访问telnet控制台
telnet控制台侦听在TELNETCONSOLE_PORT设置中定义的TCP端口，默认为6023。要访问控制台，您需要输入:
```
telnet localhost 6023
>>>
```
您需要在Windows或大多数Linux发行版中默认安装的telnet程序。
#### telnet控制台中的可用变量 
telnet控制台就像一个常规的Python shell，它运行在scrapy过程中，因此您可以从它中做任何事情，包括导入新的模块等等。

然而，telnet控制台附带了一些更为方便的默认变量 

快捷方式| 描述
---|---
crawler | Scrapy Crawler (scrapy.crawler.Crawler 对象)
engine | Crawler.engine属性
spider | 当前激活的爬虫
slot | the engine slot
extensions | 扩展管理器(manager) (Crawler.extensions属性)
stats |状态收集器 (Crawler.stats属性)
settings |Scrapy设置(setting)对象 (Crawler.settings属性)
est |打印引擎状态的报告
prefs 	|针对内存调试 (参考调试内存溢出)
p 	|pprint.pprint 函数的简写
hpy |针对内存调试 (参考 调试内存溢出)     
#### 远程登录控制台用法示例
下面是您可以使用telnet控制台完成的一些示例任务：
##### 查看引擎状态  
在终端中你可以使用 Scrapy 引擎的 est() 方法来快速查看状态：

```
telnet localhost 6023
>>> est()
Execution engine status

time()-engine.start_time                        : 8.62972998619
engine.has_capacity()                           : False
len(engine.downloader.active)                   : 16
engine.scraper.is_idle()                        : False
engine.spider.name                              : followall
engine.spider_is_idle(engine.spider)            : False
engine.slot.closing                             : False
len(engine.slot.inprogress)                     : 16
len(engine.slot.scheduler.dqs or [])            : 0
len(engine.slot.scheduler.mqs)                  : 92
len(engine.scraper.slot.queue)                  : 0
len(engine.scraper.slot.active)                 : 0
engine.scraper.slot.active_size                 : 0
engine.scraper.slot.itemproc_size               : 0
engine.scraper.slot.needs_backout()             : False
```
#### 暂停，恢复和停止 Scrapy 引擎 

```
#暂停：

telnet localhost 6023
>>> engine.pause()
>>>

#恢复：

telnet localhost 6023
>>> engine.unpause()
>>>

#停止：

telnet localhost 6023
>>> engine.stop()
Connection closed by foreign host.
```
#### Telnet 终端信号 
> scrapy.telnet.update_telnet_vars(telnet_vars)     

在 telnet 终端开启前发送该信号。您可以挂载(hook up)该信号来添加，移除或更新 telnet 本地命名空间可用的变量。您可以通过在您的处理函数(handler)中更新 telnet_vars 字典来实现该修改。  
参数: telnet_vars (dict) – telnet 变量的字典
#### Telnet 设定 
以下是终端的一些设定：
##### TELNETCONSOLE_PORT
默认：[6023, 6073]  
telnet 终端使用的端口范围。如果设为 None 或 0， 则动态分配端口。 
##### TELNETCONSOLE_HOST
默认：“127.0.0.1”
telnet 终端监听的接口(interface)。