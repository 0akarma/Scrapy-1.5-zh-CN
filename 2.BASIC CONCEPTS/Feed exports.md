# Feed exports
0.10 新版功能.

实现爬虫时最经常提到的需求就是能合适的保存爬取到的数据，或者说，生成一个带有爬取数据的”输出文件”(通常叫做”输出feed”)，来供其他系统使用。

Scrapy自带了Feed输出，并且支持多种序列化格式(serialization format)及存储方式(storage backends)。

## 序列化方式(Serialization formats)
序列化方式(Serialization formats)
feed输出使用到了 [Item exporters](https://doc.scrapy.org/en/latest/topics/exporters.html#topics-exporters) 。其自带支持的类型有:

* [JSON](https://doc.scrapy.org/en/latest/topics/feed-exports.html#topics-feed-format-json)
* [JSON lines](https://doc.scrapy.org/en/latest/topics/feed-exports.html#topics-feed-format-jsonlines)
* [CSV](https://doc.scrapy.org/en/latest/topics/feed-exports.html#topics-feed-format-csv)
* [XML](https://doc.scrapy.org/en/latest/topics/feed-exports.html#topics-feed-format-xml)
您也可以通过 FEED_EXPORTERS 设置扩展支持的属性。

### JSON
* FEED_FORMAT: json
* 使用的exporter: JsonItemExporter
* 大数据量情况下使用JSON请参见 [这个警告](https://doc.scrapy.org/en/latest/topics/exporters.html#json-with-large-data)

### JSON lines
* FEED_FORMAT: jsonlines
* 使用的 Exporter: JsonLinesItemExporter

### CSV
* FEED_FORMAT: csv
* 使用的exporter: CsvItemExporter

### XML
* FEED_FORMAT: xml
* 使用的exporter: XmlItemExporter

### Pickle
* FEED_FORMAT: pickle
* 使用的exporter: PickleItemExporter

### Marshal
* FEED_FORMAT: marshal
* 使用的exporter: MarshalItemExporter

## 存储(Storages)
使用feed输出时您可以通过使用 [URI](https://en.wikipedia.org/wiki/Uniform_Resource_Identifier) (通过 FEED_URI 设置) 来定义存储端。 feed输出支持URI方式支持的多种存储后端类型。

自带支持的存储后端有:
* [本地文件系统](https://doc.scrapy.org/en/latest/topics/feed-exports.html#topics-feed-storage-fs)
* [FTP](https://doc.scrapy.org/en/latest/topics/feed-exports.html#topics-feed-storage-ftp)
* [S3](https://doc.scrapy.org/en/latest/topics/feed-exports.html#topics-feed-storage-s3) (需要 [boto](https://github.com/boto/botocore) 或者 [botocore](https://github.com/boto/boto))
* [标准输出](https://doc.scrapy.org/en/latest/topics/feed-exports.html#topics-feed-storage-stdout)
有些存储后端会因所需的外部库未安装而不可用。例如，S3只有在 boto 库安装的情况下才可使用。

## 存储URI参数
存储URI也包含参数。当feed被创建时这些参数可以被覆盖:
* %(time)s - 当feed被创建时被timestamp覆盖
* %(name)s - 被spider的名字覆盖

其他命名的参数会被spider同名的属性所覆盖。例如， 当feed被创建时， %(site_id)s 将会被 spider.site_id 属性所覆盖。

下面用一些例子来说明:

* 存储在FTP，每个spider一个目录:
ftp://user:password@ftp.example.com/scraping/feeds/%(name)s/%(time)s.json
* 存储在S3，每一个spider一个目录:
s3://mybucket/scraping/feeds/%(name)s/%(time)s.json

## 存储端(Storage backends)
### 本地文件系统
将feed存储在本地系统。
* URI scheme: file
* URI样例: file:///tmp/export.csv
* 需要的外部依赖库: none
注意: (只有)存储在本地文件系统时，您可以指定一个绝对路径 /tmp/export.csv 并忽略协议(scheme)。不过这仅仅只能在Unix系统中工作。

### FTP
将feed存储在FTP服务器。

* URI scheme: ftp
* URI样例: ftp://user:pass@ftp.example.com/path/to/export.csv
* 需要的外部依赖库: none

### S3
将feed存储在 [Amazon S3](http://aws.amazon.com/s3/) 。

* URI scheme: s3
* URI样例:
     s3://mybucket/path/to/export.csv
     s3://aws_key:aws_secret@mybucket/path/to/export.csv
* 需要的外部依赖库: [boto](https://github.com/boto/boto) 或者 [botocore](https://github.com/boto/botocore)
您可以通过在URI中传递user/pass来完成AWS认证，或者也可以通过下列的设置来完成:

* AWS_ACCESS_KEY_ID
* AWS_SECRET_ACCESS_KEY

## 标准输出
feed输出到Scrapy进程的标准输出。

* URI scheme: stdout
* URI样例: stdout:
* 需要的外部依赖库: none

## 设定(Settings)
这些是配置feed输出的设定:
* FEED_URI (必须)
* FEED_FORMAT
* FEED_STORAGES
* FEED_EXPORTERS
* FEED_STORE_EMPTY
* FEED_EXPORT_ENCODING
* FEED_EXPORT_FIELDS
* FEED_EXPORT_INDENT

## FEED_URI
Default: None

输出feed的URI。支持的URI协议请参见 [存储端(Storage backends)](https://doc.scrapy.org/en/latest/topics/feed-exports.html#topics-feed-storage-backends) 。

为了启用feed输出，该设定是必须的。

## FEED_FORMAT
输出feed的序列化格式。可用的值请参见 [序列化方式(Serialization formats)](https://doc.scrapy.org/en/latest/topics/feed-exports.html#topics-feed-format) 。

## FEED_EXPORT_ENCODING
Default: None

被用于Feed的编码

如果未设置或设置为无（默认），则对于除JSON输出之外的所有内容都使用UTF-8，由于历史原因，JSON输出使用安全数字编码（\ uXXXX序列）。

如果您也想为JSON使用UTF-8，请使用utf-8。

## FEED_EXPORT_FIELDS
Default: None
要导出的字段列表，可选。例：
FEED_EXPORT_FIELDS = ["foo", "bar", "baz"].

使用FEED_EXPORT_FIELDS选项来定义要导出的字段及其顺序。当FEED_EXPORT_FIELDS为空或无（默认）时，Scrapy使用字段中定义的字段或蜘蛛正在产生的Item子类。

如果导出程序需要一组固定的字段（这是CSV导出格式的情况），并且FEED_EXPORT_FIELDS为空或无，则Scrapy会尝试从导出的数据中推断字段名称 - 目前它使用第一项中的字段名称。

## FEED_EXPORT_INDENT
Default: 0
用于缩进每个级别的输出的空间量。 如果FEED_EXPORT_INDENT是一个非负整数，则数组元素和对象成员将与该缩进级别相匹配。 缩进级别0（默认值）或负值会将每个项目放在一个新行中。 没有选择最紧凑的表示。

目前仅通过JsonItemExporter和XmlItemExporter实现，即当您导出为.json或.xml时。

## FEED_STORE_EMPTY
Default: False
是否导出空的Feed（即没有项目的Feed）。

## FEED_STORAGES
Default: {}

一个包含您的项目支持的额外的后端存储后端的字典。 密钥是URI方案，值是存储类的路径。

## FEED_STORAGES_BASE
Default:
```
{
    '': 'scrapy.extensions.feedexport.FileFeedStorage',
    'file': 'scrapy.extensions.feedexport.FileFeedStorage',
    'stdout': 'scrapy.extensions.feedexport.StdoutFeedStorage',
    's3': 'scrapy.extensions.feedexport.S3FeedStorage',
    'ftp': 'scrapy.extensions.feedexport.FTPFeedStorage',
}
```
包含Scrapy支持的内置Feed存储后端的字典。 您可以通过将FEED_STORAGES中的URI方案分配为无效来禁用这些后端中的任何后端。 例如，要禁用内置FTP存储后端（无需替换），请将其放置在settings.py中：
```
FEED_STORAGES = {
    'ftp': None,
}
```

## FEED_EXPORTERS
Default: {}
包含您的项目支持的其他出口商的字典。 这些键是序列化格式，值是Item导出器类的路径。

## FEED_EXPORTERS_BASE
Default:
```
{
    'json': 'scrapy.exporters.JsonItemExporter',
    'jsonlines': 'scrapy.exporters.JsonLinesItemExporter',
    'jl': 'scrapy.exporters.JsonLinesItemExporter',
    'csv': 'scrapy.exporters.CsvItemExporter',
    'xml': 'scrapy.exporters.XmlItemExporter',
    'marshal': 'scrapy.exporters.MarshalItemExporter',
    'pickle': 'scrapy.exporters.PickleItemExporter',
}
```
包含Scrapy支持的内置Feed出口商的字典。 您可以通过在FEED_EXPORTERS中将它们的序列化格式指定为无效来禁用这些导出器中的任何一个。 例如，要禁用内置的CSV导出器（无需替换），请将其置于settings.py中：
```
FEED_EXPORTERS = {
    'csv': None,
}
```