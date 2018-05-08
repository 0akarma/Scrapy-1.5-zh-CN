一旦你刮掉了你的物品，你通常会想要坚持或导出这些物品，以便在其他应用程序中使用这些数据。毕竟，这是刮削过程的全部目的。

为此，Scrapy为不同的输出格式提供了一组项目导出器，如XML，CSV或JSON。

## 使用项目出口商

如果您很匆忙，只想使用项目导出器来输出抓取的数据，请参阅[Feed输出](https://doc.scrapy.org/en/latest/topics/feed-exports.html#topics-feed-exports)。否则，如果您想知道Item Exporters如何工作或需要更多自定义功能（不包括默认导出），请继续阅读以下内容。

为了使用Item Exporter，你必须用它需要的参数来实例化它。每个项目导出器需要不同的参数，因此请检查每个导出器文档以确保在“ [内置项目导出器”参考中](https://doc.scrapy.org/en/latest/topics/exporters.html#topics-exporters-reference)。在您实例化出口商后，您必须：

1.调用该方法[`start_exporting()`](https://doc.scrapy.org/en/latest/topics/exporters.html#scrapy.exporters.BaseItemExporter.start_exporting)以指示输出过程的开始

2.调用[`export_item()`](https://doc.scrapy.org/en/latest/topics/exporters.html#scrapy.exporters.BaseItemExporter.export_item)您想要导出的每个项目的方法

3.最后调用该[`finish_exporting()`](https://doc.scrapy.org/en/latest/topics/exporters.html#scrapy.exporters.BaseItemExporter.finish_exporting)信号表示导出过程结束

在这里您可以看到一个[项目管道](https://doc.scrapy.org/en/latest/topics/item-pipeline.html)，它使用多个项目导出器根据其中一个字段的值将分组项目分组到不同的文件中：

```python
from scrapy.exporters import XmlItemExporter

class PerYearXmlExportPipeline(object):
    """Distribute items across multiple XML files according to their 'year' field"""

    def open_spider(self, spider):
        self.year_to_exporter = {}

    def close_spider(self, spider):
        for exporter in self.year_to_exporter.values():
            exporter.finish_exporting()
            exporter.file.close()

    def _exporter_for_item(self, item):
        year = item['year']
        if year not in self.year_to_exporter:
            f = open('{}.xml'.format(year), 'wb')
            exporter = XmlItemExporter(f)
            exporter.start_exporting()
            self.year_to_exporter[year] = exporter
        return self.year_to_exporter[year]

    def process_item(self, item, spider):
        exporter = self._exporter_for_item(item)
        exporter.export_item(item)
        return item
```

## 项目字段的序列化

默认情况下，字段值不加修改地传递给底层的序列化库，并且如何序列化它们的决定被委托给每个特定的序列化库。

但是，您可以*在将*每个字段值序列化为序列化*之前*自定义*其序列化库*。

有两种方法可以自定义字段序列化的方式，这将在下面介绍。

### 1.在现场声明一个序列化器

如果你使用[`Item`](https://doc.scrapy.org/en/latest/topics/items.html#scrapy.item.Item)你可以在[字段元数据中](https://doc.scrapy.org/en/latest/topics/items.html#topics-items-fields)声明一个序列化器 。序列化程序必须是可调用的，它接收一个值并返回其序列化形式。

例：

```python
import scrapy

def serialize_price(value):
    return '$ %s' % str(value)

class Product(scrapy.Item):
    name = scrapy.Field()
    price = scrapy.Field(serializer=serialize_price)
```

### 2.重写serialize_field（）方法

您也可以重写该[`serialize_field()`](https://doc.scrapy.org/en/latest/topics/exporters.html#scrapy.exporters.BaseItemExporter.serialize_field)方法来自定义字段值的导出方式。

确保[`serialize_field()`](https://doc.scrapy.org/en/latest/topics/exporters.html#scrapy.exporters.BaseItemExporter.serialize_field)在自定义代码之后调用基类方法。

例：

```python
from scrapy.exporter import XmlItemExporter

class ProductXmlExporter(XmlItemExporter):

    def serialize_field(self, field, name, value):
        if field == 'price':
            return '$ %s' % str(value)
        return super(Product, self).serialize_field(field, name, value)
```

## 内置项目导出器参考

以下是与Scrapy捆绑在一起的Item Exporters列表。其中一些包含输出示例，它们假设您正在导出这两个项目：

```
Item(name='Color TV', price='1200')
Item(name='DVD player', price='200')
```

### BaseItemExporter

>*class*`scrapy.exporters.``BaseItemExporter`(*fields_to_export=None*, *export_empty_fields=False*, *encoding='utf-8'*, *indent=0*)

这是所有项目导出器的（抽象）基类。它提供对所有（具体）项目导出器使用的常用功能的支持，例如定义要导出的字段，是否导出空字段或使用哪种编码。

这些功能可以通过该填充它们各自的实例属性的构造器参数进行配置：[`fields_to_export`](https://doc.scrapy.org/en/latest/topics/exporters.html#scrapy.exporters.BaseItemExporter.fields_to_export)，[`export_empty_fields`](https://doc.scrapy.org/en/latest/topics/exporters.html#scrapy.exporters.BaseItemExporter.export_empty_fields)，[`encoding`](https://doc.scrapy.org/en/latest/topics/exporters.html#scrapy.exporters.BaseItemExporter.encoding)，[`indent`](https://doc.scrapy.org/en/latest/topics/exporters.html#scrapy.exporters.BaseItemExporter.indent)。

- `export_item`（*项目*）

  导出给定的项目。这个方法必须在子类中实现。


- `serialize_field`（*字段*，*名称*，*值*）

  返回给定字段的序列化值。如果要控制如何序列化/导出特定字段或值，则可以覆盖此方法（在自定义项目导出器中）。默认情况下，此方法查找[在项目字段中声明](https://doc.scrapy.org/en/latest/topics/exporters.html#topics-exporters-serializers)的序列化程序[，](https://doc.scrapy.org/en/latest/topics/exporters.html#topics-exporters-serializers)并返回将该序列化程序应用于该值的结果。如果没有找到序列化程序，则返回值不变，除了使用在属性中声明的编码`unicode`进行编码的值 。`str`[`encoding`](https://doc.scrapy.org/en/latest/topics/exporters.html#scrapy.exporters.BaseItemExporter.encoding)

参数：

- **字段**（[`Field`](https://doc.scrapy.org/en/latest/topics/items.html#scrapy.item.Field)对象或空字典） - 字段被序列化。如果原始字典正在导出（不[`Item`](https://doc.scrapy.org/en/latest/topics/items.html#scrapy.item.Item)）*字段*值是一个空的字典。
- **name**（*str*） - 被序列化的字段的名称
- **值** - 正在序列化的值


- `start_exporting`（）

  指示出口过程的开始。一些出口商可能会使用它来生成一些必需的标题（例如，[`XmlItemExporter`](https://doc.scrapy.org/en/latest/topics/exporters.html#scrapy.exporters.XmlItemExporter)）。您必须在导出任何项目之前调用此方法。


- `finish_exporting`（）

  指示出口过程的结束。一些出口商可能会使用它来生成一些必需的页脚（例如，[`XmlItemExporter`](https://doc.scrapy.org/en/latest/topics/exporters.html#scrapy.exporters.XmlItemExporter)）。在没有更多项目要导出后，您必须始终调用此方法。


- `fields_to_export`

  包含要导出的字段名称的列表，如果您要导出所有字段，则为None。默认为None。一些出口商（如[`CsvItemExporter`](https://doc.scrapy.org/en/latest/topics/exporters.html#scrapy.exporters.CsvItemExporter)）尊重该属性中定义的字段的顺序。一些出口商可能需要fields_to_export列表，以便在蜘蛛返回字典（不是`Item`实例）时正确导出数据。


- `export_empty_fields`

  是否在导出的数据中包含空白/未填充的项目字段。默认为`False`。一些出口商（如[`CsvItemExporter`](https://doc.scrapy.org/en/latest/topics/exporters.html#scrapy.exporters.CsvItemExporter)）忽略这个属性，并始终导出所有空字段。字典项目将忽略此选项。


- `encoding`

  将用于编码unicode值的编码。这只会影响unicode值（它总是使用此编码序列化为str）。其他值类型不变地传递给特定的序列化库。


- `indent`

  用于缩进每个级别的输出的空间量。默认为`0`。`indent=None` 选择最紧凑的表示形式，同一行中的所有项目都没有缩进`indent<=0` 每个项目在其自己的行，没有缩进`indent>0` 每个项目在它自己的行上，用提供的数字值缩进

### XmlItemExporter

>*class*`scrapy.exporters.``XmlItemExporter`（*file*，*item_element ='item'*，*root_element ='items'*，**\* kwargs* ）

将XML格式的项目导出到指定的文件对象。

参数：

- **文件** - 用于导出数据的文件类对象。它的`write`方法应该接受`bytes`（以二进制模式打开的磁盘文件，`io.BytesIO`对象等）
- **root_element**（*str*） - 导出的XML中的根元素的名称。
- **item_element**（*str*） - 导出的XML中每个项目元素的名称。

此构造函数的其他关键字参数传递给 [`BaseItemExporter`](https://doc.scrapy.org/en/latest/topics/exporters.html#scrapy.exporters.BaseItemExporter)构造函数。

这个出口商的典型产出是：

```python
<?xml version="1.0" encoding="utf-8"?>
<items>
  <item>
    <name>Color TV</name>
    <price>1200</price>
 </item>
  <item>
    <name>DVD player</name>
    <price>200</price>
 </item>
</items>
```

除非在`serialize_field()`方法中重写，否则通过序列化`<value>`元素内的每个值来导出多值字段。这是为了方便，因为多值字段非常常见。

例如，该项目：

```
Item(name=['John', 'Doe'], age='23')
```

将被序列化为：

```
<?xml version="1.0" encoding="utf-8"?>
<items>
  <item>
    <name>
      <value>John</value>
      <value>Doe</value>
    </name>
    <age>23</age>
  </item>
</items>
```

### CsvItemExporter

>*class*`scrapy.exporters.``CsvItemExporter`（*file*，*include_headers_line = True*，*join_multivalued ='*，*'*，**\* kwargs* ）

将CSV格式的项目导出到给定的类文件对象。如果该 `fields_to_export`属性已设置，则将用于定义CSV列及其顺序。该`export_empty_fields`属性对此导出器没有影响。

参数：

- **文件** - 用于导出数据的文件类对象。它的`write`方法应该接受`bytes`（以二进制模式打开的磁盘文件，`io.BytesIO`对象等）
- **include_headers_line**（*str*） - 如果启用，使导出器输出包含从[`BaseItemExporter.fields_to_export`](https://doc.scrapy.org/en/latest/topics/exporters.html#scrapy.exporters.BaseItemExporter.fields_to_export)第一个导出项目字段获取的字段名称的标题行 。
- **join_multivalued** - 将用于加入多值字段的char（或字符），如果找到。

此构造函数的其他关键字参数传递给 [`BaseItemExporter`](https://doc.scrapy.org/en/latest/topics/exporters.html#scrapy.exporters.BaseItemExporter)构造函数，以及[csv.writer](https://docs.python.org/2/library/csv.html#csv.writer)构造函数的剩余参数 ，因此您可以使用任何csv.writer构造函数参数来定制此导出器。

这个出口商的典型产出是：

```
product,price
Color TV,1200
DVD player,200
```

### PickleItemExporter

>*class*`scrapy.exporters.``PickleItemExporter`(*file*, *protocol=0*, **\*kwargs*)

将项目以pickle格式导出到给定的类文件对象。

参数：

- **文件** - 用于导出数据的文件类对象。它的`write`方法应该接受`bytes`（以二进制模式打开的磁盘文件，`io.BytesIO`对象等）
- **protocol**（*int*） - 使用的pickle协议。

有关更多信息，请参阅pickle模块文档。此构造函数的其他关键字参数传递给 BaseItemExporter构造函数。Pickle不是一种人们可读的格式，所以没有提供输出示例。

### PprintItemExporter

>*class*`scrapy.exporters.``PprintItemExporter`(*file*, **\*kwargs*)

以漂亮打印格式将项目导出到指定的文件对象。

参数：

- **文件** - 用于导出数据的文件类对象。它的`write`方法应该接受`bytes`（以二进制模式打开的磁盘文件，`io.BytesIO`对象等）

此构造函数的其他关键字参数传递给 [`BaseItemExporter`](https://doc.scrapy.org/en/latest/topics/exporters.html#scrapy.exporters.BaseItemExporter)构造函数。

这个出口商的典型产出是：

```
{'name': 'Color TV', 'price': '1200'}
{'name': 'DVD player', 'price': '200'}
```

较长的行（当存在时）格式很好。

### JsonItemExporter

>*class*`scrapy.exporters.``JsonItemExporter`(*file*, **\*kwargs*)

将JSON格式的项目导出到指定的类文件对象，将所有对象写为对象列表。额外的构造函数参数传递给[`BaseItemExporter`](https://doc.scrapy.org/en/latest/topics/exporters.html#scrapy.exporters.BaseItemExporter)构造函数，以及[JSONEncoder](https://docs.python.org/2/library/json.html#json.JSONEncoder)构造函数的剩余参数，因此您可以使用任何 [JSONEncoder](https://docs.python.org/2/library/json.html#json.JSONEncoder)构造函数参数来自定义此导出器。

参数：

- **文件** - 用于导出数据的文件类对象。它的`write`方法应该接受`bytes`（以二进制模式打开的磁盘文件，`io.BytesIO`对象等）

这个出口商的典型产出是：

```
[{"name": "Color TV", "price": "1200"},
{"name": "DVD player", "price": "200"}]
```

>**！警告**
>
>JSON是非常简单和灵活的序列化格式，但是对于大量数据来说它不能很好地扩展，因为在JSON解析器（任何语言）之间，增量（即流模式）解析得不到很好的支持（如果有的话），并且他们中的大多数只是解析整个对象在内存中。如果您希望JSON的强大功能和简单性以更流的格式，请考虑使用[`JsonLinesItemExporter`](https://doc.scrapy.org/en/latest/topics/exporters.html#scrapy.exporters.JsonLinesItemExporter) ，或者将输出分成多个块。

### JsonLinesItemExporter

>*class*`scrapy.exporters.``JsonLinesItemExporter`(*file*, **\*kwargs*)

将JSON格式的项目导出到指定的类文件对象，每行写入一个JSON编码的项目。额外的构造函数参数传递给[`BaseItemExporter`](https://doc.scrapy.org/en/latest/topics/exporters.html#scrapy.exporters.BaseItemExporter)构造函数，以及[JSONEncoder](https://docs.python.org/2/library/json.html#json.JSONEncoder)构造函数的剩余参数，因此您可以使用任何[JSONEncoder](https://docs.python.org/2/library/json.html#json.JSONEncoder) 构造函数参数来自定义此导出器。

参数：

- **文件** - 用于导出数据的文件类对象。它的`write`方法应该接受`bytes`（以二进制模式打开的磁盘文件，`io.BytesIO`对象等）

这个出口商的典型产出是：

```
{"name": "Color TV", "price": "1200"}
{"name": "DVD player", "price": "200"}
```

与生成的不同[`JsonItemExporter`](https://doc.scrapy.org/en/latest/topics/exporters.html#scrapy.exporters.JsonItemExporter)，此导出器生成的格式非常适合序列化大量数据。