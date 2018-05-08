#项目
抓取的主要目标是从非结构化来源（通常是网页）中提取结构化数据。Scrapy蜘蛛可以将提取的数据作为Python字典返回。虽然方便且熟悉，但Python字典缺乏结构：很容易在字段名称中输入错误的拼写或返回不一致的数据，尤其是在包含许多蜘蛛的大型项目中。

为了定义通用输出数据格式Scrapy提供了`Item`类。` Item`对象是用于收集刮取数据的简单容器。他们提供了一个类似字典的便捷语法`API`，以用于声明其可用字段。

各种Scrapy组件使用Items提供的额外信息：输出端查看已声明的字段以确定要导出的列，可以使用Item字段元数据定制序列化，`trackref `跟踪Item实例以帮助查找内存泄漏（请参阅`使用trackref调试内存泄漏`）等。
##声明项目
项目使用简单的类定义语法和`Field` 对象来声明。这里是一个例子：

	import scrapy
	class Product(scrapy.Item):
	    name = scrapy.Field()
	    price = scrapy.Field()
	    stock = scrapy.Field()
	    last_updated = scrapy.Field(serializer=str)
>注意
>
熟悉`Django`的人会注意到Scrapy项目被声明为类似于`Django Models`，除了Scrapy项目更简单以外没有不同概念的字段类型。

##项目字段
`Field`对象用于为每个字段指定元数据。例如，以上示例中所示对`last_updated`字段进行序列化的函数。

您可以为每个字段指定任何类型的元数据。`Field`对象接受的值没有限制。出于同样的原因，没有所有可用元数据密钥的参考列表。在`Field`对象中定义的每个键都可以被不同的组件使用，只有那些组件才知道它。您也可以根据自己的需要定义和使用项目中的任何其他的`Field`键。`Field`对象的主要目标 是提供一种在一个地方定义所有字段元数据的方法。通常，那些行为依赖于每个字段的组件使用特定的字段键来配置该行为。您必须参考其文档以查看每个组件使用哪些元数据密钥。

请注意，`Field`用于声明项目的对象不会保留为类属性。相反，他们可以通过`Item.fields`属性访问。
##使用项目
以下是使用上面声明的`Product`项目对项目执行的常见任务的一些示例 。你会注意到`API`与`dict API`非常相似。
###创建项目

	>>> product = Product(name='Desktop PC', price=1000)
	>>> print product
	Product(name='Desktop PC', price=1000)
###获取字段值

	>>> product['name']
	Desktop PC
	>>> product.get('name')
	Desktop PC
	>>> product['price']
	1000
	>>> product['last_updated']
	Traceback (most recent call last):
	...
	KeyError: 'last_updated'
	>>> product.get('last_updated', 'not set')
	not set
	>>> product['lala'] # getting unknown field
	Traceback (most recent call last):
	...
	KeyError: 'lala'
	>>> product.get('lala', 'unknown field')
	'unknown field'
	>>> 'name' in product  # is name field populated?
	True
	>>> 'last_updated' in product  # is last_updated populated?
	False
	>>> 'last_updated' in product.fields  # is last_updated a declared field?
	True
	>>> 'lala' in product.fields  # is lala a declared field?
	False
###设置字段值

	>>> product['last_updated'] = 'today'
	>>> product['last_updated']
	today
	>>> product['lala'] = 'test' # setting unknown field
	Traceback (most recent call last):
	...
	KeyError: 'Product does not support field: lala'
###访问所有填充值
要访问所有填充值，只需使用典型的`dict API`：
	
	>>> product.keys()
	['price', 'name']
	>>> product.items()
	[('price', 1000), ('name', 'Desktop PC')]
###其他常见任务
####复制项目：
	
	>>> product2 = Product(product)
	>>> print product2
	Product(name='Desktop PC', price=1000)
	>>> product3 = product2.copy()
	>>> print product3
	Product(name='Desktop PC', price=1000)
####从项目创建字典：

	>>> dict(product) # create a dict from all populated values
	{'price': 1000, 'name': 'Desktop PC'}
####从字典创建项目：
	
	>>> Product({'name': 'Laptop PC', 'price': 1500})
	Product(price=1500, name='Laptop PC')
	>>> Product({'name': 'Laptop PC', 'lala': 1500}) # warning: unknown field in dict
	Traceback (most recent call last):
	...
	KeyError: 'Product does not support field: lala'
##扩展项目
您可以通过声明原始项目的子类来扩展项目（以添加更多字段或更改某些字段的某些元数据）。

例如：

	class DiscountedProduct(Product):
	    discount_percent = scrapy.Field(serializer=str)
	    discount_expiration_date = scrapy.Field()
您还可以使用以前的字段元数据并附加更多值或更改现有值来扩展字段元数据，如下所示：
	
	class SpecificProduct(Product):
	    name = scrapy.Field(Product.fields['name'], serializer=my_serializer)
这会添加（或替换）`serializer`字段的元数据关键字`name`，并保留所有先前存在的元数据值。
##项目对象
>classscrapy.item.Item（[arg ]）

返回一个新的Item，可以从给定的参数中初始化。

项目复制标准的字典API，包括其构造函数。Item提供的唯一附加属性是：
>fields

一个包含这个Item的所有声明字段的字典，不仅包含那些填充的字段。键是字段名称，值是`Field`在Item声明中使用的对象。
##字段对象
>classscrapy.item.Field（[arg ]）

该`Field`类只是一个别名内置的字典类，并没有提供任何额外功能或属性。换句话说， `Field`对象是普通的Python字典。一个单独的类用于支持 基于类属性的项目声明语法。
