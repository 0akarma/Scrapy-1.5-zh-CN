# 部署Spider

本节介绍您为部署Scrapy蜘蛛而定期运行它们的不同选项。在本地机器上运行Scrapy蜘蛛程序对于（早期）开发阶段来说非常方便，但是当您需要执行长时间运行的蜘蛛或移动蜘蛛来持续运行时，并不是那么重要。这是部署Scrapy蜘蛛解决方案的地方。

部署Scrapy蜘蛛的普遍选择是：

- [Scrapyd](https://doc.scrapy.org/en/latest/topics/deploy.html#deploy-scrapyd)（开源）
- [Scrapy云](https://doc.scrapy.org/en/latest/topics/deploy.html#deploy-scrapy-cloud)（基于云）

## 部署到Scrapyd服务器

[Scrapyd](https://github.com/scrapy/scrapyd)是运行Scrapy蜘蛛的开源应用程序。它提供了一个HTTP API的服务器，能够运行和监控Scrapy蜘蛛。

要将Spider部署到Scrapyd，您可以使用由[scrapyd-client](https://github.com/scrapy/scrapyd-client)包提供的scrapyd-deploy工具。请参阅[scrapyd-deploy文档](https://scrapyd.readthedocs.io/en/latest/deploy.html)以获取更多信息。

Scrapyd由一些Scrapy开发人员维护。

## 部署到Scrapy Cloud

[Scrapy Cloud](https://scrapinghub.com/scrapy-cloud)是[Scrapy](https://scrapinghub.com/scrapy-cloud)背后的[Scrapinghub](https://scrapinghub.com/)托管的基于云的服务。

Scrapy Cloud不需要安装和监控服务器，并提供了一个很好的用户界面来管理蜘蛛并查看抓取的项目，日志和统计信息。

要将Scider部署到Scrapy Cloud，您可以使用[shub](https://doc.scrapinghub.com/shub.html)命令行工具。请参阅[Scrapy Cloud文档](https://doc.scrapinghub.com/scrapy-cloud.html)以获取更多信息。

Scrapy Cloud与Scrapyd兼容，并且可以根据需要在它们之间切换 - 从`scrapy.cfg`文件中读取配置`scrapyd-deploy`。