#数据管道
数据被蜘蛛抓取后，它将被发送到数据管线，该数据管线通过顺序执行的多个组件处理它。

每个项目管道组件（有时简称为“项目管道”）是一个实现简单方法的Python类。他们收到数据并对其执行操作，并决定该数据是否应该通过管道继续运行，或者被丢弃并不再处理。

项目管道的典型用途是：

- 清理HTML数据
- 验证刮取的数据（检查项目是否包含某些字段）
- 检查重复项（并放下它们）
- 将刮取的项目存储在数据库中
##编写自己的项目管道
每个项目管道组件都是一个Python类，它必须实现以下方法：
>process_item(self, item, spider)

每个项目管道组件都会调用`process_item()`方法。必须：用数据返回一个字典，返回一个`Item `（或任何后代类）对象，返回[Twisted Deferred]()或者引发`DropItem`异常。丢弃的物品不再被进一步的管道组件处理。

**参数：**	

- 项目（`Item`对象或字典） - 项目被刮掉
- 蜘蛛（`Spider`物体） - 刮掉物品的蜘蛛

另外，他们还可以实施以下方法：
>open_spider(self, spider)

这个方法在蜘蛛打开时被调用。

**参数：**	**spider**（Spider对象） - 被打开的蜘蛛
>close_spider(self, spider)

这个方法在蜘蛛关闭时被调用。

**参数：**	**spider**（Spider对象） - 被关闭的蜘蛛
>from_crawler（CLS，crawler）

如果存在，就调用这个`classmethod`来创建一个来自`Crawler`的管道实例。它必须返回一个新的管道实例。抓取工具对象提供对所有Scrapy核心组件的访问，如设置和信号; 它是管道访问它们并将其功能挂接到Scrapy的一种方式。

**参数：	Crawler**（`Crawler`对象） - 使用此管道的搜寻器
##项目管道示例
###值验证和丢弃没有值的项
让我们来看看下面的假设管道，它调整`price`那些不包含增值税（`price_excludes_vat`属性）的项目的属性，并删除那些不包含值的项目：
	
	from scrapy.exceptions import DropItem
	class PricePipeline(object):
	
	    vat_factor = 1.15
	
	    def process_item(self, item, spider):
	        if item['price']:
	            if item['price_excludes_vat']:
	                item['price'] = item['price'] * self.vat_factor
	            return item
	        else:
	            raise DropItem("Missing price in %s" % item)
###将项目写入JSON文件
以下管道将所有抓取的项目（来自所有蜘蛛）存储到一个items.jl文件中，每行包含一个以JSON格式序列化的项目：
	
	import json
	class JsonWriterPipeline(object):
	
	    def open_spider(self, spider):
	        self.file = open('items.jl', 'w')
	
	    def close_spider(self, spider):
	        self.file.close()
	
	    def process_item(self, item, spider):
	        line = json.dumps(dict(item)) + "\n"
	        self.file.write(line)
	        return item
	        返回 项目
>注意
>
JsonWriterPipeline的目的只是介绍如何编写项目管道。如果你真的想把所有被抓取的项目存储到一个JSON文件中，你应该使用Feed输出。

###将项目写入MongoDB
在这个例子中，我们将使用`pymongo`将项目写入`MongoDB`。MongoDB地址和数据库名称在Scrapy设置中指定; MongoDB集合以item类命名。

这个例子的要点是展示如何使用from_crawler() 方法以及如何正确清理资源。
	
	import pymongo
	class MongoPipeline(object):
	
	    collection_name = 'scrapy_items'
	
	    def __init__(self, mongo_uri, mongo_db):
	        self.mongo_uri = mongo_uri
	        self.mongo_db = mongo_db
	
	    @classmethod
	    def from_crawler(cls, crawler):
	        return cls(
	            mongo_uri=crawler.settings.get('MONGO_URI'),
	            mongo_db=crawler.settings.get('MONGO_DATABASE', 'items')
	        )
	
	    def open_spider(self, spider):
	        self.client = pymongo.MongoClient(self.mongo_uri)
	        self.db = self.client[self.mongo_db]
	
	    def close_spider(self, spider):
	        self.client.close()
	
	    def process_item(self, item, spider):
	        self.db[self.collection_name].insert_one(dict(item))
	        return item
###以项目的截图
这个例子演示了如何从process_item()方法返回Deferred。它使用Splash渲染项目url的屏幕截图。管道请求本地运行的Splash实例。下载请求并延迟回调后，它将项目保存到一个文件并将文件名添加到项目。
	
	import scrapyimport hashlibfrom urllib.parse import quote
	
	class ScreenshotPipeline(object):
	    """Pipeline that uses Splash to render screenshot of    every Scrapy item."""
	
	    SPLASH_URL = "http://localhost:8050/render.png?url={}"
	
	    def process_item(self, item, spider):
	        encoded_item_url = quote(item["url"])
	        screenshot_url = self.SPLASH_URL.format(encoded_item_url)
	        request = scrapy.Request(screenshot_url)
	        dfd = spider.crawler.engine.download(request, spider)
	        dfd.addBoth(self.return_item, item)
	        return dfd
	
	    def return_item(self, response, item):
	        if response.status != 200:
	            # Error happened, return item.
	            return item
	
	        # Save screenshot to file, filename will be hash of url.
	        url = item["url"]
	        url_hash = hashlib.md5(url.encode("utf8")).hexdigest()
	        filename = "{}.png".format(url_hash)
	        with open(filename, "wb") as f:
	            f.write(response.body)
	
	        # Store filename in item.
	        item["screenshot_filename"] = filename
	        return item
###重复过滤器
过滤器查找重复的项目，并删除已处理的项目。假设我们的物品具有唯一的ID，但我们的蜘蛛会使用相同的ID返回多个物品：
	
	from scrapy.exceptions import DropItem
	class DuplicatesPipeline(object):
	
	    def __init__(self):
	        self.ids_seen = set()
	
	    def process_item(self, item, spider):
	        if item['id'] in self.ids_seen:
	            raise DropItem("Duplicate item found: %s" % item)
	        else:
	            self.ids_seen.add(item['id'])
	            return item
##激活项目管道组件
要激活Item Pipeline组件，您必须将其类添加到 ITEM_PIPELINES设置中，如下例所示：
	
	ITEM_PIPELINES = {
	    'myproject.pipelines.PricePipeline': 300,
	    'myproject.pipelines.JsonWriterPipeline': 800,}
您在此设置中分配给类的整数值决定了它们的运行顺序：项目从较低值到较高值的类别。通常在0-1000范围内定义这些数字。
