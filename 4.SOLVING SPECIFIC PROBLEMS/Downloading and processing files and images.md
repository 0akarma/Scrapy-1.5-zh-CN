# 下载和处理文件和图像

Scrapy提供可重复使用的[项目管道，](https://doc.scrapy.org/en/latest/topics/item-pipeline.html)用于下载附加到特定项目的文件（例如，当您刮擦产品并且还想在本地下载其图像时）。这些管道共享一些功能和结构（我们将它们称为介质管道），但通常您可以使用“文件管道”或“图像管道”。

两个管道都实现这些功能：

- 避免重新下载最近下载的媒体
- 指定存储介质的位置（文件系统目录，Amazon S3存储桶，Google云存储存储桶）

图像管道有几个额外的功能来处理图像：

- 将所有下载的图像转换为通用格式（JPG）和模式（RGB）
- 生成缩略图
- 检查图像宽度/高度以确保它们符合最小限制

管道还保留当前正在计划下载的那些媒体URL的内部队列，并将包含相同媒体到达的那些响应连接到该队列。这避免了多个项目共享多次下载相同的媒体。

## 使用文件管道

典型的工作流程`FilesPipeline`如下所示：

1. 在一个爬虫里，您抓取一个项目并将所需的URL放入一个 `file_urls`字段中。
2. 该项目从爬虫内返回并转到物品管道。
3. 当该项目进入`FilesPipeline`，该`file_urls`字段中的URL 将使用标准的Scrapy调度器和下载器（这意味着调度器和下载器中间件可以重复使用）计划下载，但具有更高的优先级，会在其他页面被抓取前处理。该项目在该特定管道阶段保持“锁定(locker)”状态，直到文件完成下载（或由于某种原因未完成下载）。
4. 下载文件时，另一个字段（`files`）将被更新到结构中。该字段将包含一个含有下载文件信息的字典列表，例如下载的路径，原始抓取的URL（从`file_urls`字段获取）和文件校验码。该`files`字段列表中的文件将保留原始`file_urls`字段的相同顺序。如果某个文件下载失败，则会记录下错误信息并且该文件不会出现在该`files`字段中。

## 使用图像管道

使用[`ImagesPipeline`](https://doc.scrapy.org/en/latest/topics/media-pipeline.html#scrapy.pipelines.images.ImagesPipeline)很像使用`FilesPipeline`，除了使用的默认字段名称不同：您的`image_urls`用于项目的图像URL，它将填充`images`字段以获取有关下载图像的信息。

对图像文件使用[`ImagesPipeline`](https://doc.scrapy.org/en/latest/topics/media-pipeline.html#scrapy.pipelines.images.ImagesPipeline)的优点是，您可以配置一些额外的功能，例如生成缩略图和根据图像大小过滤图像。

Images Pipeline使用[Pillow](https://github.com/python-pillow/Pillow)将图像缩略图和规格化为JPEG / RGB格式，因此您需要安装此库才能使用它。 在大多数情况下，[Python成像库](http://www.pythonware.com/products/pil/)（PIL）也应该可以工作，但是在某些设置中会导致麻烦，所以我们建议使用[Pillow](https://github.com/python-pillow/Pillow)而不是PIL。

## 启用媒体管道

要启用媒体管道，您必须先将其添加到您的项目 [`ITEM_PIPELINES`](https://doc.scrapy.org/en/latest/topics/settings.html#std:setting-ITEM_PIPELINES)设置中。

对于图像管道，请使用：

```
ITEM_PIPELINES = {'scrapy.pipelines.images.ImagesPipeline': 1}
```

对于文件管道，请使用：

```
ITEM_PIPELINES = {'scrapy.pipelines.files.FilesPipeline': 1}
```

注意

您也可以同时使用文件和图像管道。

然后，将目标存储设置配置为用于存储下载图像的有效值。否则，管道将保持禁用状态，即使您将其包含在[`ITEM_PIPELINES`](https://doc.scrapy.org/en/latest/topics/settings.html#std:setting-ITEM_PIPELINES)设置中。

对于文件管道，请设置以下[`FILES_STORE`](https://doc.scrapy.org/en/latest/topics/media-pipeline.html#std:setting-FILES_STORE)设置：

```
FILES_STORE = '/path/to/valid/dir'
```

对于图像管线，设置[`IMAGES_STORE`](https://doc.scrapy.org/en/latest/topics/media-pipeline.html#std:setting-IMAGES_STORE)设置：

```
IMAGES_STORE = '/path/to/valid/dir'
```

## 支持的存储

文件系统目前是唯一官方支持的存储，但也支持在[Amazon S3](https://aws.amazon.com/s3/)和[Google云存储中](https://cloud.google.com/storage/)存储文件。

### 文件系统存储

这些文件使用它们的URL 的[SHA1散列](https://en.wikipedia.org/wiki/SHA_hash_functions)来存储文件名。

例如，以下图片网址：

```
http://www.example.com/image.jpg
```

谁的SHA1hash是：

```
3afec3b4765f8f0a07b78f98c07b83f013567a0a
```

将被下载并存储在以下文件中：

```
<IMAGES_STORE>/full/3afec3b4765f8f0a07b78f98c07b83f013567a0a.jpg
```

哪里：

- `<IMAGES_STORE>`是[`IMAGES_STORE`](https://doc.scrapy.org/en/latest/topics/media-pipeline.html#std:setting-IMAGES_STORE)图像管道设置中定义的目录。
- `full`是将完整图像与缩略图分开的子目录（如果使用的话）。有关更多信息，请参阅[图像的缩略图生成](https://doc.scrapy.org/en/latest/topics/media-pipeline.html#topics-images-thumbnails)。

### Amazon S3存储

[`FILES_STORE`](https://doc.scrapy.org/en/latest/topics/media-pipeline.html#std:setting-FILES_STORE)和[`IMAGES_STORE`](https://doc.scrapy.org/en/latest/topics/media-pipeline.html#std:setting-IMAGES_STORE)可以代表Amazon S3存储桶。Scrapy会自动将文件上传到存储桶。

例如，这是一个有效的[`IMAGES_STORE`](https://doc.scrapy.org/en/latest/topics/media-pipeline.html#std:setting-IMAGES_STORE)值：

```
IMAGES_STORE = 's3://bucket/images'
```

您可以修改用于存储文件的访问控制列表（ACL）策略，这是由[`FILES_STORE_S3_ACL`](https://doc.scrapy.org/en/latest/topics/media-pipeline.html#std:setting-FILES_STORE_S3_ACL)和 [`IMAGES_STORE_S3_ACL`](https://doc.scrapy.org/en/latest/topics/media-pipeline.html#std:setting-IMAGES_STORE_S3_ACL)设置定义的。默认情况下，ACL被设置为 `private`。要使文件公开可用，请使用以下`public-read` 策略：

```
IMAGES_STORE_S3_ACL = 'public-read'
```

有关更多信息，请参阅Amazon S3开发人员指南中的[预装ACL](https://docs.aws.amazon.com/AmazonS3/latest/dev/acl-overview.html#canned-acl)。

### Google云端存储

[`FILES_STORE`](https://doc.scrapy.org/en/latest/topics/media-pipeline.html#std:setting-FILES_STORE)和[`IMAGES_STORE`](https://doc.scrapy.org/en/latest/topics/media-pipeline.html#std:setting-IMAGES_STORE)可以代表Google云存储存储分区。Scrapy会自动将文件上传到存储桶。（需要[谷歌云存储](https://cloud.google.com/storage/docs/reference/libraries#client-libraries-install-python)）

例如，这些是有效的[`IMAGES_STORE`](https://doc.scrapy.org/en/latest/topics/media-pipeline.html#std:setting-IMAGES_STORE)和[`GCS_PROJECT_ID`](https://doc.scrapy.org/en/latest/topics/media-pipeline.html#std:setting-GCS_PROJECT_ID)设置：

```
IMAGES_STORE = 'gs://bucket/images/'
GCS_PROJECT_ID = 'project_id'
```

有关身份验证的信息，请参阅此[文档](https://cloud.google.com/docs/authentication/production)。

为了首先使用媒体管道，[启用它](https://doc.scrapy.org/en/latest/topics/media-pipeline.html#topics-media-pipeline-enabling)。

然后，如果spider用URLs键（`file_urls`或者 `image_urls`分别为文件或图像管线）返回字典，管道将把结果放在相应的键（`files`或`images`）下。

如果您更喜欢使用[`Item`](https://doc.scrapy.org/en/latest/topics/items.html#scrapy.item.Item)，那么使用必要的字段定义一个自定义项目，例如图像管道的示例：

```
import scrapy

class MyItem(scrapy.Item):

    # ... other item fields ...
    image_urls = scrapy.Field()
    images = scrapy.Field()
```

如果要为URL键或结果键使用另一个字段名称，也可以覆盖它。

对于文件管道，设置[`FILES_URLS_FIELD`](https://doc.scrapy.org/en/latest/topics/media-pipeline.html#std:setting-FILES_URLS_FIELD)和/或 [`FILES_RESULT_FIELD`](https://doc.scrapy.org/en/latest/topics/media-pipeline.html#std:setting-FILES_RESULT_FIELD)设置：

```
FILES_URLS_FIELD = 'field_name_for_your_files_urls'
FILES_RESULT_FIELD = 'field_name_for_your_processed_files'
```

对于图像管线，设置[`IMAGES_URLS_FIELD`](https://doc.scrapy.org/en/latest/topics/media-pipeline.html#std:setting-IMAGES_URLS_FIELD)和/或 [`IMAGES_RESULT_FIELD`](https://doc.scrapy.org/en/latest/topics/media-pipeline.html#std:setting-IMAGES_RESULT_FIELD)设置：

```
IMAGES_URLS_FIELD = 'field_name_for_your_images_urls'
IMAGES_RESULT_FIELD = 'field_name_for_your_processed_images'
```

如果您需要更复杂的内容并希望覆盖自定义管道行为，请参阅[扩展介质管道](https://doc.scrapy.org/en/latest/topics/media-pipeline.html#topics-media-pipeline-override)。

如果您有多个图像管道从ImagePipeline继承，并且您希望在不同管道中具有不同的设置，则可以设置以管道类的大写名称开头的设置键。例如，如果您的管道被称为MyPipeline，并且您想定制IMAGES_URLS_FIELD，则可以定义设置MYPIPELINE_IMAGES_URLS_FIELD并使用您的自定义设置。

## 附加功能

### 文件到期

图像管道避免了下载最近下载的文件。要调整此保留延迟，请使用[`FILES_EXPIRES`](https://doc.scrapy.org/en/latest/topics/media-pipeline.html#std:setting-FILES_EXPIRES)设置（或者 [`IMAGES_EXPIRES`](https://doc.scrapy.org/en/latest/topics/media-pipeline.html#std:setting-IMAGES_EXPIRES)在图像管道的情况下），该设置指定延迟天数：

```
# 120 days of delay for files expiration
FILES_EXPIRES = 120

# 30 days of delay for images expiration
IMAGES_EXPIRES = 30
```

这两个设置的默认值是90天。

如果你有管道的子类FilesPipeline，你想有不同的设置，你可以设置以大写的类名开头的设置键。例如，给定管道类名为MyPipeline，您可以设置设置键：

MYPIPELINE_FILES_EXPIRES = 180

并且管道类MyPipeline将到期时间设置为180。

### 为图像生成缩略图

图像管道可以自动创建下载图像的缩略图。

为了使用此功能，您必须设置[`IMAGES_THUMBS`](https://doc.scrapy.org/en/latest/topics/media-pipeline.html#std:setting-IMAGES_THUMBS)一个字典，其中的键是缩略图名称，值是它们的尺寸。

例如：

```
IMAGES_THUMBS = {
    'small': (50, 50),
    'big': (270, 270),
}
```

当您使用此功能时，图像管道将使用以下格式创建每个指定尺寸的缩略图：

```
<IMAGES_STORE>/thumbs/<size_name>/<image_id>.jpg
```

哪里：

- `<size_name>`是在指定的[`IMAGES_THUMBS`](https://doc.scrapy.org/en/latest/topics/media-pipeline.html#std:setting-IMAGES_THUMBS) 字典键（`small`，`big`等）
- `<image_id>`是图片网址的[SHA1哈希值](https://en.wikipedia.org/wiki/SHA_hash_functions)

使用`small`和`big`缩略图名称存储的图像文件示例：

```
<IMAGES_STORE>/full/63bbfea82b8880ed33cdb762aa11fab722a90a24.jpg
<IMAGES_STORE>/thumbs/small/63bbfea82b8880ed33cdb762aa11fab722a90a24.jpg
<IMAGES_STORE>/thumbs/big/63bbfea82b8880ed33cdb762aa11fab722a90a24.jpg
```

第一个是从网站下载的完整图像。

### 过滤掉小图片

使用图像管线时，可以通过指定[`IMAGES_MIN_HEIGHT`](https://doc.scrapy.org/en/latest/topics/media-pipeline.html#std:setting-IMAGES_MIN_HEIGHT)和 [`IMAGES_MIN_WIDTH`](https://doc.scrapy.org/en/latest/topics/media-pipeline.html#std:setting-IMAGES_MIN_WIDTH)设置中允许的最小尺寸来放置太小的图像。

例如：

```
IMAGES_MIN_HEIGHT = 110 
IMAGES_MIN_WIDTH = 110
```

注意

大小限制完全不影响缩略图生成。

可以只设置一个尺寸约束或者两者兼有。设置它们时，只会保存同时满足最小尺寸的图像。对于上述示例，大小（105 x 105）或（105 x 200）或（200 x 105）的图像将全部被删除，因为至少一个维度比约束更短。

默认情况下，没有大小限制，因此所有图像都被处理。

### 允许重定向

默认情况下，媒体管道忽略重定向，即对媒体文件URL请求的HTTP重定向意味着媒体下载被认为失败。

要处理媒体重定向，请将此设置设置为`True`：

```
MEDIA_ALLOW_REDIRECTS = True
```

## 扩展媒体管道

在这里看到你可以在自定义文件管道中覆盖的方法：

*class*`scrapy.pipelines.files.``FilesPipeline`

- `get_media_requests`(*item*, *info*)

  如工作流程所示，管道将获取要从项目下载的图像的URL。为了做到这一点，您可以覆盖该 [`get_media_requests()`](https://doc.scrapy.org/en/latest/topics/media-pipeline.html#scrapy.pipelines.files.FilesPipeline.get_media_requests)方法并为每个文件URL返回一个请求：

  ```
  def get_media_requests(self, item, info):
      for file_url in item['file_urls']:
          yield scrapy.Request(file_url)
  ```

这些请求将由管道处理，并且当它们完成下载时，结果将会作为2-element元祖的列表发送到[`item_completed()`](https://doc.scrapy.org/en/1.5/topics/media-pipeline.html#scrapy.pipelines.files.FilesPipeline.item_completed)。每个元组将包含以下内容：`(success, file_info_or_error)`

- `success`是一个布尔值，如果图像下载成功则返回`True` 由于某种原因失败了返回`False`
- `file_info_or_error`是一个包含以下键（如果成功返回`Ture` ）的字典，或者如果出现了问题，则是 [Twisted Failure](https://twistedmatrix.com/documents/current/api/twisted.python.failure.Failure.html)
  - `url` - 文件的下载地址。这是从[`get_media_requests()`](https://doc.scrapy.org/en/latest/topics/media-pipeline.html#scrapy.pipelines.files.FilesPipeline.get_media_requests) 方法返回的请求的url 。
  - `path`- [`FILES_STORE`](https://doc.scrapy.org/en/latest/topics/media-pipeline.html#std:setting-FILES_STORE)文件存储位置的路径（相对于）
  - `checksum`- 图像内容的[MD5哈希](https://en.wikipedia.org/wiki/MD5)

接收到的元组列表[`item_completed()`](https://doc.scrapy.org/en/latest/topics/media-pipeline.html#scrapy.pipelines.files.FilesPipeline.item_completed)保证保持从该[`get_media_requests()`](https://doc.scrapy.org/en/latest/topics/media-pipeline.html#scrapy.pipelines.files.FilesPipeline.get_media_requests)方法返回的请求的相同顺序 。

这是一个典型的`results`参数值：

```
[(True,
  {'checksum': '2b00042f7481c7b056c4b410d28f33cf',
   'path': 'full/7d97e98f8af710c7e7fe703abc8f639e0ee507c4.jpg',
   'url': 'http://www.example.com/images/product1.jpg'}),
 (True,
  {'checksum': 'b9628c4ab9b595f72f280b90c4fd093d',
   'path': 'full/1ca5879492b8fd606df1964ea3c1e2f4520f076f.jpg',
   'url': 'http://www.example.com/images/product2.jpg'}),
 (False,
  Failure(...))]
```

默认情况下，该[`get_media_requests()`](https://doc.scrapy.org/en/latest/topics/media-pipeline.html#scrapy.pipelines.files.FilesPipeline.get_media_requests)方法返回`None`，这意味着没有文件要下载的项目。

`item_completed`(*results*, *items*, *info*)

[`FilesPipeline.item_completed()`](https://doc.scrapy.org/en/latest/topics/media-pipeline.html#scrapy.pipelines.files.FilesPipeline.item_completed)当单个项目的所有文件请求都已完成（完成下载或由于某种原因失败）时调用此方法。

该[`item_completed()`](https://doc.scrapy.org/en/latest/topics/media-pipeline.html#scrapy.pipelines.files.FilesPipeline.item_completed)方法必须返回将发送到后续项目管道阶段的输出，因此您必须返回（或丢弃）该项目，就像在任何管道中一样。

以下是[`item_completed()`](https://doc.scrapy.org/en/latest/topics/media-pipeline.html#scrapy.pipelines.files.FilesPipeline.item_completed)我们将下载的文件路径（传递到结果中）存储在`file_paths` 项目字段中的方法示例，如果项目不包含任何文件，我们将删除该项目：

```
from scrapy.exceptions import DropItem

def item_completed(self, results, item, info):
    image_paths = [x['path'] for ok, x in results if ok]
    if not image_paths:
        raise DropItem("Item contains no images")
    item['image_paths'] = image_paths
    return item
```

  默认情况下，该[`item_completed()`](https://doc.scrapy.org/en/latest/topics/media-pipeline.html#scrapy.pipelines.files.FilesPipeline.item_completed)方法返回该项目。

在这里看到您可以在自定义图像管道中覆盖的方法：

*class*`scrapy.contrib.pipeline.images.ImagesPipeline`

​	这[`ImagesPipeline`](https://doc.scrapy.org/en/latest/topics/media-pipeline.html#scrapy.pipelines.images.ImagesPipeline)是`FilesPipeline`对字段名称进行自定义并为图像添加自定义行为的扩展。

​	`get_media_requests`（*item*，*info* ）

​		以与方法相同的方式工作`FilesPipeline.get_media_requests()`，但对图像网址使用不同的字段名称。

​		必须为每个图片网址返回一个请求。

​	`item_completed`(*results*, *item*, *info*)

​		[`ImagesPipeline.item_completed()`](https://doc.scrapy.org/en/latest/topics/media-pipeline.html#scrapy.pipelines.images.ImagesPipeline.item_completed)当单个项目的所有图像请求都已完成（完成下载或由于某种原因失败）时调用此方法。

​		以与方法相同的方式工作`FilesPipeline.item_completed()`，但使用不同的字段名称来存储图像下载结果。

​		默认情况下，该[`item_completed()`](https://doc.scrapy.org/en/latest/topics/media-pipeline.html#scrapy.pipelines.images.ImagesPipeline.item_completed)方法返回该项目。

## 自定义图像管道示例

下面是图像管道的完整示例，其示例方法如上所示：

```
import scrapy
from scrapy.pipelines.images import ImagesPipeline
from scrapy.exceptions import DropItem

class MyImagesPipeline(ImagesPipeline):

    def get_media_requests(self, item, info):
        for image_url in item['image_urls']:
            yield scrapy.Request(image_url)

    def item_completed(self, results, item, info):
        image_paths = [x['path'] for ok, x in results if ok]
        if not image_paths:
            raise DropItem("Item contains no images")
        item['image_paths'] = image_paths
        return item
```