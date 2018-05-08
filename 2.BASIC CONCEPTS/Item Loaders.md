#项目加载器
项目加载器为填充爬虫项目提供了一种便利的机制。尽管可以使用自己的字典式API填充项目，但项目加载器通过自动执行一些常见任务（例如在分配原始提取数据之前解析原始提取的数据），为从填充过程中填充它们提供了更方便的API。

换句话说，`Items`提供了刮取数据的容器，而`Item Loaders`提供了填充该容器的机制。

项目加载器旨在提供一种灵活，高效且简单的机制来扩展和覆盖不同的字段解析规则，无论是通过蜘蛛还是通过源格式（HTML，XML等），而不会成为维护的噩梦。
##使用项目加载器来填充项目
要使用Item Loader，你必须首先实例化它。您可以使用类似字典的对象（例如Item或dict）或没有它的实例化，在这种情况下，使用`ItemLoader.default_item_class` 属性中指定的Item类在Item Loader构造函数中自动实例化Item 。

然后，你开始收集值到Item Loader中，通常使用 选择器。您可以将多个值添加到相同的项目字段; Item Loader将知道如何使用适当的处理函数“加入”这些值。

这是一个典型的项目填充器使用的蜘蛛，在`项目章节`中由`产品项目`声明:
	
	from scrapy.loader import ItemLoaderfrom myproject.items import Product
	def parse(self, response):
	    l = ItemLoader(item=Product(), response=response)
	    l.add_xpath('name', '//div[@class="product_name"]')
	    l.add_xpath('name', '//div[@class="product_title"]')
	    l.add_xpath('price', '//p[@id="price"]')
	    l.add_css('stock', 'p#stock]')
	    l.add_value('last_updated', 'today') # you can also use literal values
	    return l.load_item()
通过快速查看代码，我们可以看到`name`字段是从页面中的两个不同XPath位置提取的：

1.`//div[@class="product_name"]`

2.`//div[@class="product_title"]`

换句话说，通过使用`add_xpath()`方法从两个XPath位置提取数据来收集数据。这是稍后将分配给该`name`字段的数据。

之后，使用类似`price`和`stock`字段的调用（后者在`add_css()`方法中使用CSS选择器），最后，使用不同的`add_value()`方法直接填充`last_update`字段。
最后，收集到所有数据时，`ItemLoader.load_item()`方法被调用，返回先前调用`add_xpath()`， `add_css()`和`add_value()`提取并收集到的数据并填充项目。
##输入和输出处理器
项目加载器为每个（项目）字段包含一个输入处理器和一个输出处理器。输入处理器只要它接收到所提取的数据便通过`add_xpath()`，`add_css()`或 `add_value()`方法进行处理，输入处理器的结果被收集和保存在`ItemLoader`。

收集所有数据后，`ItemLoader.load_item()`调用该方法来填充数据，以及获取被填充的`Item`对象。在输出处理器被调用处理之前收集的数据（使用输入处理器处理）的情况下。输出处理器的结果是分配给项目的最终值。

我们来看一个例子来说明如何为特定字段调用输入和输出处理器（这同样适用于其他字段）：
	
	l = ItemLoader(Product(), some_selector)
	l.add_xpath('name', xpath1) # (1)
	l.add_xpath('name', xpath2) # (2)
	l.add_css('name', css) # (3)
	l.add_value('name', 'test') # (4)
	return l.load_item() # (5)
所以发生了什么：

1.来自`xpath1`的数据被提取，并通过`name`字段的输入处理器。输入处理器的结果被收集并保存在Item Loader中（但尚未分配给该项目）。

2.来自`xpath2`的数据被提取，并通过（1）中使用的相同的输入处理器。输入处理器的结果附加到（1）中收集的数据（如果有的话）。

3.除了数据是从`css` CSS选择器中提取并通过（1）和（2）中使用的相同输入处理器之外，这种情况与之前的类似。输入处理器的结果附加到（1）和（2）中收集的数据（如果有的话）。

4.这种情况也类似于以前的情况，不同之处在于要收集的值是直接分配的，而不是从XPath表达式或CSS选择器中提取。但是，该值仍然通过输入处理器。在这种情况下，由于该值不可迭代，因此在将其传递给输入处理器之前将其转换为单个元素的迭代，因为输入处理器总是接收迭代。

5.步骤（1），（2），（3）和（4）中收集的数据通过该字段的输出处理器`name`。输出处理器的结果是分配给name 项目中字段的值。

值得注意的是，处理器仅仅是可调用的对象，它们被调用以解析数据，并返回一个解析的值。所以你可以使用任何功能作为输入或输出处理器。唯一的要求是它们必须接受一个（且只有一个）位置参数，它将是一个迭代器。

>注意
>
输入和输出处理器都必须接收迭代器作为其第一个参数。这些函数的输出可以是任何东西。输入处理器的结果将被附加到包含收集值（对于该字段）的内部列表（在加载程序中）。输出处理器的结果是最终将分配给该项目的值。

另外需要注意的是输入处理器返回的值在内部收集（以列表形式），然后传递给输出处理器以填充字段。

最后，Scrapy附带了一些内置的常用处理器以方便使用。
##声明项目加载器
通过使用类定义语法，Item Loaders被声明为Items。这里是一个例子：
	
	from scrapy.loader import ItemLoaderfrom scrapy.loader.processors import TakeFirst, MapCompose, Join
	class ProductLoader(ItemLoader):
	
	    default_output_processor = TakeFirst()
	
	    name_in = MapCompose(unicode.title)
	    name_out = Join()
	
	    price_in = MapCompose(unicode.strip)
	
	    # ...
如您所见，输入处理器使用`_in`后缀声明，而输出处理器使用`_out`后缀声明。你也可以使用`ItemLoader.default_input_processor`和 `ItemLoader.default_output_processor`属性声明一个默认的输入/输出处理器 。
##声明输入和输出处理器
如前一节所述，可以在Item Loader定义中声明输入和输出处理器，并且以这种方式声明输入处理器是很常见的。但是，还有一个地方可以指定要使用的输入和输出处理器：在“ 项目字段” 元数据中。这里是一个例子：
	
	import scrapyfrom scrapy.loader.processors import Join, MapCompose, TakeFirstfrom w3lib.html import remove_tags
	def filter_price(value):
	    if value.isdigit():
	        return value
	class Product(scrapy.Item):
	    name = scrapy.Field(
	        input_processor=MapCompose(remove_tags),
	        output_processor=Join(),
	    )
	    price = scrapy.Field(
	        input_processor=MapCompose(remove_tags, filter_price),
	        output_processor=TakeFirst(),
	    )
	>>> from scrapy.loader import ItemLoader
	>>> il = ItemLoader(item=Product())
	>>> il.add_value('name', [u'Welcome to my', u'<strong>website</strong>'])
	>>> il.add_value('price', [u'&euro;', u'<span>1000</span>'])>>> il.load_item()
	{'name': u'Welcome to my website', 'price': u'1000'}
输入和输出处理器的优先顺序如下：

1.项目加载程序字段特定的属性：`field_in`和`field_out`（最优先）

2.字段元数据（`input_processor`和`output_processor`密钥）

3.项目加载程序默认值：`ItemLoader.default_input_processor()`和`ItemLoader.default_output_processor()`（最低优先级）

另请参阅：[重用和扩展项目装入程序]()。

##项目加载器上下文
Item Loader Context是Item Loader中所有输入和输出处理器共享的key/values的字典。它可以在声明，实例化或使用Item Loader时传递。它们用于修改输入/输出处理器的行为。

例如，假设您有一个接收文本值并从中提取长度的`parse_length`函数：
	
	def parse_length(text, loader_context):
	    unit = loader_context.get('unit', 'm')
	    # ... length parsing code goes here ...
	    return parsed_length
通过接受`loader_contex`t参数，函数明确地告诉Item Loader它能够接收Item Loader上下文，因此Item Loader在调用它时传递当前活动的上下文，并且处理函数（`parse_length`在这种情况下）可以使用它们。

有几种方法可以修改Item Loader Context值：

1.通过修改当前活动的Item Loader Context（`context` 属性）：
	
	loader = ItemLoader(product)
	loader.context['unit'] = 'cm'

2.在Item Loader实例（Item Loader构造函数的关键字参数存储在Item Loader Context中）中：

	loader = ItemLoader(product, unit='cm')

3.在Item Loader声明中，对于那些支持使用Item Loader Context实例化它们的输入/输出处理器。`MapCompose`是其中之一：
	
	class ProductLoader(ItemLoader):
	    length_out = MapCompose(parse_length, unit='cm')
##ItemLoader对象
>class scrapy.loader.ItemLoader([item, selector, response, ]**kwargs)

返回一个新的Item Loader来填充给定的Item。如果没有给出项目，则使用`default_item_class`类中的一个自动实例化。

当使用选择器或响应参数实例化时，`ItemLoader`类提供了使用选择器从网页提取数据的便捷机制。

**参数**：

- item（`Item`对象） -利用后续调用`add_xpath()`，`add_css()`或`add_value()`来填充项目实例.
- selector（`Selector`对象） - 使用`add_xpath()`（resp.`add_css()`）或`replace_xpath()`（resp.`replace_css()`）方法时从中提取数据的选择器 。
- response（`Respons`对象） - 用于构造选择器的响应 `default_selector_class`，除非给出选择器参数，否则此参数将被忽略。

项目，选择器，响应和其余关键字参数分配给Loader Context（可通过`context`属性访问）。

`ItemLoader` 实例具有以下方法：

>get_value(value, *processors, **kwargs)

通过给定`processors`参数和关键字参数处理给定的`value`参数。

**可用关键字参数：**

- re（str 或编译正则表达式） - 一种正则表达式`extract_regex()`，用于在处理器之前应用使用方法从给定值中提取数据

例子：
	
	>>> from scrapy.loader.processors import TakeFirst
	>>> loader.get_value(u'name: foo', TakeFirst(), unicode.upper, re='name: (.+)')
	'FOO`
>add\_value（field\_name，value，* processors，** kwargs ）

处理`value`，然后添加给给定的字段。

该值首先通过`get_value()`给出 `processorsand`来`kwargs`传递，然后通过`字段输入处理器`及其结果附加到为该字段收集的数据。如果该字段已包含收集的数据，则添加新数据。

给定`field\_name`可以是`None`，在这种情况下，可以添加多个字段的值。处理后的值应该是一个字段，并将`field\_name`映射到值。

例子：

	loader.add_value('name', u'Color TV')
	loader.add_value('colours', [u'white', u'blue'])
	loader.add_value('length', u'100')
	loader.add_value('name', u'name: foo', TakeFirst(), re='name: (.+)')
	loader.add_value(None, {'name': u'foo', 'sex': u'male'})
	
>replace\_value（field\_name，value，* processors，** kwargs ）

类似于`add_value()`使用新值替换收集的数据，而不是添加它。
>get\_xpath(xpath, *processors, **kwargs)

与`ItemLoader.get_value()`类似，但接收XPath而不是值，该值用于从与此关联的`ItemLoader`选择器中提取unicode字符串列表。

**参数：	**

- xpath（str） - 给定提取数据的Xpath
- re（str 或编译正则表达式） - 用于从所选XPath区域提取数据的正则表达式

例子：
	
	# HTML snippet: <p class="product-name">Color TV</p>
	loader.get_xpath('//p[@class="product-name"]')
	# HTML snippet: <p id="price">the price is $1200</p>
	loader.get_xpath('//p[@id="price"]', TakeFirst(), re='the price is (.*)')
>add\_xpath（field_name，xpath，* processors，** kwargs ）

与`ItemLoader.add_value()`类似，但接收XPath而不是值，该值用于从与ItemLoader关联的选择器中提取unicode字符串列表。

见`get_xpath()`的`kwargs`。

**参数：**	

- xpath（str） - 用于提取数据的XPath

例子：

	# HTML snippet: <p class="product-name">Color TV</p>
	loader.add_xpath('name', '//p[@class="product-name"]')
	# HTML snippet: <p id="price">the price is $1200</p>
	loader.add_xpath('price', '//p[@id="price"]', re='the price is (.*)')

>replace_xpath（field_name，xpath，* processors，** kwargs ）

与`add_xpath()`收集的数据类似，但替代收集的数据。
>get_css（css，*processors，** kwargs ）

与`ItemLoader.get_value()`类似，但接收CSS选择器而不是值，该值用于从与ItemLoader关联的选择器中提取unicode字符串列表。

**参数：**	

- css（str） - 用于提取数据的CSS选择器
- re（str 或编译正则表达式） - 用于从所选CSS区域提取数据的正则表达式

例子：
	
	# HTML snippet: <p class="product-name">Color TV</p>
	loader.get_css('p.product-name')
	# HTML snippet: <p id="price">the price is $1200</p>
	loader.get_css('p#price', TakeFirst(), re='the price is (.*)')
>add_css（field_name，css，* processors，** kwargs ）

与`ItemLoader.add_value()`类似，但接收CSS选择器而不是值，该值用于从与ItemLoader关联的选择器中提取unicode字符串列表。

见`get_css()`的`kwargs`。

**参数：**	

- css（str） - 从中​​提取数据的CSS选择器

例子：
	
	# HTML snippet: <p class="product-name">Color TV</p>
	loader.add_css('name', 'p.product-name')
	# HTML snippet: <p id="price">the price is $1200</p>
	loader.add_css('price', 'p#price', re='the price is (.*)')
>replace_css（field_name，css，* processors，** kwargs ）

与`add_css()`收集的数据类似，但替代收集的数据。
>load\_item（）

用目前收集的数据填充项目，然后返回。收集到的数据首先通过输出处理器，以获得分配给每个项目字段的最终值。
>nested\_xpath（xpath ）

用xpath选择器创建一个嵌套的加载器。提供的选择器是相对于与此相关的选择器应用的`ItemLoader`。嵌套装载机股份Item 与母公司ItemLoader如此呼吁add_xpath()，add_value()，replace\_value()等会像预期的那样。
>nested\_css（css）

用css选择器创建一个嵌套的加载器。提供的选择器相对于与`ItemLoader`相关的选择器应用。嵌套装载机股份`Item`与母公司`ItemLoader`如此呼吁`add_xpath()，add_value()，replace_value()`等会像预期的那样。
>get\_collected\_values（field_name）

返回给定字段的收集值。
>get\_output\_value（field_name ）

对于给定的字段，返回使用输出处理器分析的收集值。此方法根本不填充或修改项目。
>get\_input\_processor（field_name ）

返回给定字段的输入处理器。
>get\_output\_processor（field_name ）

返回给定字段的输出处理器。

`ItemLoader` 实例具有以下属性：

>item

该Item对象通过这个项目装载机正在解析。
>context

此项目加载器的当前活动上下文。
>default_item_class

Item类（或工厂），用于在构造函数中未给出时实例化项目。
>default\_input\_processor

默认输入处理器用于那些没有指定一个的字段。
>default\_output\_processor

默认输出处理器用于那些没有指定的字段。
>default\_selector\_class

使用的类构造selector的此 ItemLoader，如果只响应在构造函数中给出。如果在构造函数中给出选择器，则忽略该属性。该属性有时在子类中被覆盖。
>selector

Selector从中提取数据的对象。它可以是构造函数中给出的选择符，也可以是使用构造函数中给出的响应创建的选择符 default\_selector\_class。该属性意味着是只读的。
##嵌套装载机
从文档的子部分解析相关值时，创建嵌套的加载程序可能很有用。想象一下，您正在从页面的页脚中提取详细信息，如下所示：

例：

	<footer>
	    <a class="social" href="https://facebook.com/whatever">Like Us</a>
	    <a class="social" href="https://twitter.com/whatever">Follow Us</a>
	<a class="email" href="mailto:whatever@example.com">Email Us</a>
	</footer>
没有嵌套的加载器，你需要为你想要提取的每个值指定完整的xpath（或css）。

例：

	loader = ItemLoader(item=Item())
	# load stuff not in the footer
	loader.add_xpath('social', '//footer/a[@class = "social"]/@href')
	loader.add_xpath('email', '//footer/a[@class = "email"]/@href')
	loader.load_item()
相反，您可以使用页脚选择器创建一个嵌套加载器，并添加相对于页脚的值。功能相同，但您可以避免重复页脚选择器。

例：
	
	loader = ItemLoader(item=Item())
	# load stuff not in the footer
	footer_loader = loader.nested_xpath('//footer')
	footer_loader.add_xpath('social', 'a[@class = "social"]/@href')
	footer_loader.add_xpath('email', 'a[@class = "email"]/@href')
	# no need to call footer_loader.load_item()
	loader.load_item()
您可以任意嵌套装载器，并且可以使用xpath或css选择器。作为一般指导原则，当您的代码变得更简单时，请使用嵌套加载器，但不要过度嵌套，否则解析器会变得难以阅读。
##重用和扩展项目加载器
随着项目越来越大并且获得越来越多的蜘蛛，维护成为一个基本问题，尤其是当你必须处理每个蜘蛛的许多不同的分析规则时，有很多例外情况，而且还想重用公共处理器。

项目加载器旨在减轻解析规则的维护负担，同时又不失灵活性，同时还提供了扩展和覆盖它们的便利机制。出于这个原因，项目加载器支持传统的Python类继承来处理特定的蜘蛛（或蜘蛛组）的差异。

例如，假设某个特定网站用三条短横线（例如---Plasma TV---）包围它们的产品名称，并且您不希望最终产品名称中出现这些短划线。
您可以通过重新使用和扩展默认Product Item 
Loader（`ProductLoader`）来删除这些破折号：
	
	from scrapy.loader.processors import MapComposefrom myproject.ItemLoaders import ProductLoader
	def strip_dashes(x):
	    return x.strip('-')
	class SiteSpecificLoader(ProductLoader):
	    name_in = MapCompose(strip_dashes, ProductLoader.name_in)
另一种扩展项目加载器的情况会非常有用，那就是当你有多种源格式时，例如XML和HTML。在XML版本中，您可能希望删除CDATA事件。以下是如何执行此操作的示例：

	from scrapy.loader.processors import MapCompose
	from myproject.ItemLoaders import ProductLoader
	from myproject.utils.xml import remove_cdata
	class XmlProductLoader(ProductLoader):
	    name_in = MapCompose(remove_cdata, ProductLoader.name_in)
这就是通常扩展输入处理器的方法。

至于输出处理器，在字段元数据中声明它们更为常见，因为它们通常仅取决于字段，而不取决于每个特定的站点解析规则（如输入处理器所做的那样）。另请参阅： [声明输入和输出处理器]()。

还有很多其他可能的方式来扩展，继承和覆盖您的项目加载器，而不同的项目加载器层次结构可能更适合不同的项目。Scrapy只提供机制; 它不会对您的Loaders集合实施任何特定的组织 - 这取决于您和您的项目需求。
##可用的内置处理器
尽管您可以使用任何可调用的函数作为输入和输出处理器，但Scrapy提供了一些常用的处理器，下面将对其进行介绍。其中的一些像`MapCompose`（通常用作输入处理器）组成按顺序执行的几个函数的输出，以产生最终解析值。

以下是所有内置处理器的列表：

>class scrapy.loader.processors.Identity

最简单的处理器，它什么都不做。它返回原始值不变。它不接收任何构造函数参数，也不接受Loader上下文。

例：

	>>> from scrapy.loader.processors import Identity
	>>> proc = Identity()
	>>> proc(['one', 'two', 'three'])['one', 'two', 'three']
	
>class scrapy.loader.processors.Identity

从接收到的值中返回第一个非空/非空值，因此它通常用作单值字段的输出处理器。它不接收任何构造函数参数，也不接受Loader上下文。

例：

	>>> from scrapy.loader.processors import Identity
	>>> proc = Identity()
	>>> proc(['one', 'two', 'three'])
	['one', 'two', 'three']
	
>class scrapy.loader.processors.Join（separator = u'' ）

返回与构造函数中给定的分隔符连接的值，默认值为。它不接受Loader Context。`u' '`

使用默认分隔符时，此处理器等同于以下功能： `u' '.join`

例子：
	
	>>> from scrapy.loader.processors import TakeFirst
	>>> proc = TakeFirst()
	>>> proc(['', 'one', 'two', 'three'])
	'one'
	
>class scrapy.loader.processors.Compose(*functions, **default_loader_context)

由给定函数组构成的处理器。这意味着该处理器的每个输入值都被传递给第一个函数，并且该函数的结果被传递给第二个函数，依此类推，直到最后一个函数返回该处理器的输出值。
默认情况下，停止处理`None`值。这个行为可以通过传递关键字参数来改变`stop_on_none=False`。

例：
	
	>>> from scrapy.loader.processors import Compose
	>>> proc = Compose(lambda v: v[0], str.upper)
	>>> proc(['hello', 'world'])
	'HELLO'
每个函数都可以选择性的接收`loader_context`参数。对于那些接受了的，这个处理器将通过该参数传递当前活动的`Loader context` 。

在构造函数中传递的keyword参数用作传递给每个函数调用的默认Loader上下文值。但是，传递给函数的最终Loader上下文值将被当前活动的Loader上下文覆盖，并通过`ItemLoader.context()`属性进行访问。
>class scrapy.loader.processors.MapCompose(*functions, **default_loader_context)

与处理器类似，由给定功能的组成构成的`Compose`处理器。这个处理器的不同之处在于内部结果在各个函数之间传递的方式，如下所示：

这个处理器的输入值被迭代，并且第一个函数被应用于每个元素。这些函数调用的结果（每个元素一个）被连接起来构造一个新的迭代器，然后用于应用​​第二个函数，等等，直到最后一个函数被应用到所收集的值列表的每个值为止远。最后一个函数的输出值被连接在一起以产生该处理器的输出。

每个特定的函数都可以返回一个值或一个值列表，这些值是与应用于其他输入值的同一个函数返回的值列表一致的。函数也可以返回，`None`在这种情况下，该函数的输出将被忽略，以便通过链进一步处理。
该处理器提供了一种便捷的方式来组合仅使用单个值（而不是迭代）的函数。由于这个原因，`MapCompose`处理器通常用作输入处理器，因为通常使用选择器的`extract()`方法提取数据 ，选择器返回一个unicode字符串列表。

下面的例子应该说明它的工作原理：
	
	>>> def filter_world(x):
	...     return None if x == 'world' else x
	...
	>>> from scrapy.loader.processors import MapCompose
	>>> proc = MapCompose(filter_world, unicode.upper)
	>>> proc([u'hello', u'world', u'this', u'is', u'scrapy'])
	[u'HELLO, u'THIS', u'IS', u'SCRAPY']
与Compose处理器一样，函数可以接收Loader上下文，并将构造函数关键字参数用作默认上下文值。查看 `Compose`处理器了解更多信息。
>class scrapy.loader.processors.SelectJmes（json_path ）

使用提供给构造函数的json路径查询该值并返回输出。需要运行jmespath（[https://github.com/jmespath/jmespath.py]()）。该处理器一次只有一个输入。

例：
	
	>>> from scrapy.loader.processors import SelectJmes, Compose, MapCompose
	>>> proc = SelectJmes("foo") #for direct use on lists and dictionaries
	>>> proc({'foo': 'bar'})
	'bar'
	>>> proc({'foo': {'bar': 'baz'}})
	{'bar': 'baz'}
与Json合作：
	
	>>> import json
	>>> proc_single_json_str = Compose(json.loads, SelectJmes("foo"))
	>>> proc_single_json_str('{"foo": "bar"}')
	u'bar'
	>>> proc_json_list = Compose(json.loads, MapCompose(SelectJmes('foo')))
	>>> proc_json_list('[{"foo":"bar"}, {"baz":"tar"}]')
	[u'bar']