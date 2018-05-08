# �����ռ�(Stats Collection)
Scrapy�ṩ�˷�����ռ����ݵĻ��ơ�������key/value��ʽ�洢��ֵ����Ǽ���ֵ�� �û��ƽ��������ռ���(Stats Collector)������ͨ�� Crawler API ������ stats ��ʹ�á���������½� ���������ռ���ʹ�÷��� ������������˵����

���������ռ�(stats collection)�������߹رգ������ռ�����Զ���ǿ��õġ� ���������import���Լ���ģ�鲢ʹ����API(����ֵ���������µ�״̬��(stat keys))�� ��������Ϊ�˼������ռ��ķ���: ����Ӧ��ʹ�ó���һ�д������ռ�����spider��Scrpay��չ���κ���ʹ�������ռ���������ͷ��״̬��

�����ռ�������һ��������(������״̬��)�ܸ�Ч��(�ڹر������)�ǳ���Ч(�����������)��

�����ռ�����ÿ��spider����һ��״̬����spider����ʱ���ñ��Զ��򿪣���spider�ر�ʱ���Զ��رա�

## ���������ռ���ʹ�÷���
ͨ�� stats ������ʹ�������ռ����� ����������չ��ʹ��״̬������:

class ExtensionThatAccessStats(object):

    def __init__(self, stats):
        self.stats = stats

    @classmethod
    def from_crawler(cls, crawler):
        return cls(crawler.stats)
��������:

>stats.set_value('hostname', socket.gethostname())

��������ֵ:

>stats.inc_value('pages_crawled')

���µ�ֵ��ԭ����ֵ��ʱ��������:

>stats.max_value('max_items_scraped', value)

���µ�ֵ��ԭ����ֵСʱ��������:

>stats.min_value('min_free_memory_percent', value)

��ȡ����:
```
>>>stats.get_value('pages_crawled')
8
```
��ȡ��������:

>>> stats.get_stats()
{'pages_crawled': 1238, 'start_time': datetime.datetime(2009, 7, 14, 21, 47, 28, 977139)}
���õ������ռ���
���˻����� StatsCollector ��ScrapyҲ�ṩ�˻��� StatsCollector �������ռ����� ������ͨ�� STATS_CLASS ������ѡ��Ĭ��ʹ�õ��� MemoryStatsCollector ��

### MemoryStatsCollector
class scrapy.statscol.MemoryStatsCollector
һ���򵥵������ռ���������spider������Ϻ������ݱ������ڴ��С����ݿ���ͨ�� spider_stats ���Է��ʡ���������һ����spider����Ϊ��(key)���ֵ䡣

����Scrapy��Ĭ��ѡ��

spider_stats
������ÿ��spider���һ����ȡ��״̬���ֵ�(dict)�����ֵ���spider����Ϊ����ֵҲ���ֵ䡣

### DummyStatsCollector
class scrapy.statscol.DummyStatsCollector
�������ռ����������κ����鵫�ǳ���Ч(��Ϊʲô������(д�ĵ��������Ƥo(�s���t)o))�� ������ͨ������ STATS_CLASS ��������ռ��������ر������ռ������Ч�ʡ� �����������ռ������ܸ��������Scrapy�����Ĵ���(�������ҳ��)��˵�Ƿǳ�С�ġ�