链接提取器是唯一目的是从网页（[`scrapy.http.Response`](https://doc.scrapy.org/en/latest/topics/request-response.html#scrapy.http.Response)对象）中提取链接的对象，这些链接最终将被跟踪。

`scrapy.linkextractors.LinkExtractor`在Scrapy 中可用，但您可以通过实现简单的界面来创建自己的自定义链接提取器以满足您的需求。

每个链接提取器所具有的唯一公共方法是`extract_links`接收一个[`Response`](https://doc.scrapy.org/en/latest/topics/request-response.html#scrapy.http.Response)对象并返回一个`scrapy.link.Link`对象列表。链接提取器意味着被实例化一次，并且他们的`extract_links`方法被多次调用以提取不同的响应来提取要遵循的链接。

链接提取器[`CrawlSpider`](https://doc.scrapy.org/en/latest/topics/spiders.html#scrapy.spiders.CrawlSpider) 通过一系列规则在类中使用（在Scrapy中可用），但即使不从子类继承，也可以在您的蜘蛛 中使用链接提取器[`CrawlSpider`](https://doc.scrapy.org/en/latest/topics/spiders.html#scrapy.spiders.CrawlSpider)，因为它的目的非常简单：提取链接。

## 内置链接提取器参考

[`scrapy.linkextractors`](https://doc.scrapy.org/en/latest/topics/link-extractors.html#module-scrapy.linkextractors)模块中提供了与Scrapy捆绑在一起的链接提取器类 。

默认链接提取器是`LinkExtractor`，它与以下内容相同 [`LxmlLinkExtractor`](https://doc.scrapy.org/en/latest/topics/link-extractors.html#scrapy.linkextractors.lxmlhtml.LxmlLinkExtractor)：

```python
from scrapy.linkextractors import LinkExtractor
```

>*class*`scrapy.linkextractors.lxmlhtml.``LxmlLinkExtractor`(*allow=()*, *deny=()*, *allow_domains=()*, *deny_domains=()*, *deny_extensions=None*, *restrict_xpaths=()*, *restrict_css=()*, *tags=('a'*, *'area')*, *attrs=('href'*, *)*, *canonicalize=False*, *unique=True*, *process_value=None*, *strip=True*)

### LxmlLinkExtractor

参数：

- **允许**（*正则表达式**（或**列表**）*） - 单个正则表达式（或正则表达式列表），以便（（绝对））网址必须匹配才能被提取。如果没有给出（或空），它将匹配所有链接。

- **拒绝**（*一个正则表达式**（或**列表**）*） - 一个正则表达式（或正则表达式列表），为了被排除（即不被提取），（绝对）URL必须匹配。它优先于`allow`参数。如果没有给出（或空），它不会排除任何链接。

- **allow_domains**（*str* *或*[*list*](https://doc.scrapy.org/en/latest/topics/api.html#scrapy.loader.SpiderLoader.list)） - 包含将被考虑用于提取链接的域的单个值或字符串列表

- **deny_domains**（*str* *或*[*list*](https://doc.scrapy.org/en/latest/topics/api.html#scrapy.loader.SpiderLoader.list)） - 单个值或包含域的字符串列表，这些字段不会被视为提取链接

- **deny_extensions**（[*list*](https://doc.scrapy.org/en/latest/topics/api.html#scrapy.loader.SpiderLoader.list)） - 包含扩展的单个值或字符串列表，在提取链接时应被忽略。如果没有给出，它将默认为[scrapy.linkextractors](https://github.com/scrapy/scrapy/blob/master/scrapy/linkextractors/__init__.py)包`IGNORED_EXTENSIONS`中定义的 列表 。

- **restrict_xpaths**（*str* *或*[*list*](https://doc.scrapy.org/en/latest/topics/api.html#scrapy.loader.SpiderLoader.list)） - 是一个XPath（或XPath的列表），它定义了应该从中提取链接的响应内的区域。如果给定，只有那些XPath选择的文本才会被扫描以查找链接。看下面的例子。

- **restrict_css**（*str* *或*[*list*](https://doc.scrapy.org/en/latest/topics/api.html#scrapy.loader.SpiderLoader.list)） - 一个CSS选择器（或选择器列表），它定义响应中应从中提取链接的区域。具有与......相同的行为`restrict_xpaths`。

- **标记**（*str* *或*[*list*](https://doc.scrapy.org/en/latest/topics/api.html#scrapy.loader.SpiderLoader.list)） - 提取链接时要考虑的标记或标记列表。默认为。`('a', 'area')`

- **attrs**（[*列表*](https://doc.scrapy.org/en/latest/topics/api.html#scrapy.loader.SpiderLoader.list)） - 查找提取链接时应考虑的属性或属性列表（仅适用于`tags` 参数中指定的那些标签）。默认为`('href',)`

- **canonicalize**（*布尔*） - 规范每个提取的URL（使用w3lib.url.canonicalize_url）。默认为`False`。请注意，canonicalize_url用于重复检查; 它可以更改服务器端可见的URL，因此对于使用规范化和原始URL的请求，响应可能会有所不同。如果您使用LinkExtractor来跟踪链接，那么保留默认值会更加健壮`canonicalize=False`。

- **唯一**（*布尔*） - 是否应对抽取的链接应用重复筛选。

- process_value（可调用） -

  接收从标签提取的每个值和被扫描属性的函数，并且可以修改该值并返回一个新值，或者返回`None`忽略该链接。如果没有给出，则`process_value`默认为。`lambda x: x`

  例如，要从此代码中提取链接：

```
<a href="javascript:goToPage('../other/page.html'); return false">Link text</a>
```

您可以在以下功能中使用以下功能`process_value`：

```python
def process_value(value):
    m = re.search("javascript:goToPage\('(.*?)'", value)
    if m:
        return m.group(1)
```

- **strip**（*布尔*） - 是否从提取的属性中去除空白。根据HTML5标准，前导和尾部空格必须从被剥离`href`的属性`<a>`，`<area>` 以及许多其他的元素，`src`属性`<img>`，`<iframe>` 元件等，所以LinkExtractor默认条空间字符。设置`strip=False`关闭它（例如，如果你从元素或属性，允许前导/尾随空格提取网址）。