# 一些陷阱

## 工作目录(Job directory)

这里要注意，这个目录只适用于一个spider

## 序列化(Request serialization)

pickle模块实现了基本的数据序列化和反序列化。通过pickle模块的序列化操作我们能够将程序中运行的对象信息保存到文件中去，永久存储；通过pickle模块的反序列化操作，我们能够从文件中创建上一次程序保存的对象。

**序列化操作包括：**

- `pickle.dump()`
- `Pickler(file, protocol).dump(obj)`

**反序列化操作包括：**

- `pickle.load()`
- `Unpickler(file).load()`

# 下载中间件应用

scrapy每一个请求与回答都要通过下载中间件，所以，我们可以通过写属于自己的下载中间件，达到一些目的

### 轻度使用

如果我们要改UA或者使用代理IP的话，都需要重写自己的下载中间件。

比如，如果要使用本地的代理ip：

1. 在项目文件夹的settings.py中，把`DOWNLOADER_MIDDLEWARES`的注释取消掉。
2. `middleares.py`中写入一个中间件类

### 重度使用

重度使用就可能会需要用到`process_request(request, spider)`见[文档](5.EXTENDING SCRAPY/Downloader Middleware.md)

通过process_request，我们可以令scrapy在我们需要的时候增加代理，也可以取消重试发送请求，其他用法还要自己阅读文档进行实验哦！

# 蜘蛛中间件应用

spider中间件，给予了我们指挥spider去做什么的权力，当我们爬取小姐姐、小哥哥图片时，我们就可以给spider加一些特定功能，让她更加方便更加高效地去完成我们给她的任务。



# API应用

