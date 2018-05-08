### 网络服务
web service已经被转移到一个单独的项目中。
Github地址：  
[scrapy-jsonrpc](https://github.com/scrapy-plugins/scrapy-jsonrpc)  
#### scrapy-jsonrpc  
scrapy-jsonrpc是通过JSON-RPC控制运行的scrapy爬虫的扩展。该服务通过JSON-RPC 2.0协议提供对主爬虫对象的访问。  
##### 安装  
使用pip安装scrapy-jsonrp：  
> $ pip install scrapy-jsonrpc     

##### 配置  
首先，您需要在settings.py中包含扩展命令EXTENSIONS,例如：
> EXTENSIONS = {
    'scrapy_jsonrpc.webservice.WebService': 500,   
}  

接着，你需要将 JSONRPC\_ENABLED设置为True来启动这个拓展。web服务器将在JSONRPC\_PORT侦听指定的端口(默认情况下，它将尝试监听端口6080)，并在JSONRPC_LOGFILE将日志记录到指定的文件。  
访问爬虫对象的端点是：  
> http://localhost:6080/crawler  

##### 客户端例子
这里有一个命令行工具，用于说明如何构建客户端。你可以在example-client.py中找到它。它支持一些基本的命令，如列出运行的爬虫等。   
-  设置
    -  JSONRPC_ENABLED   默认：False  
    一个布尔值，它指定是否启用web服务(如果它的扩展也被启用)。
    -  JSONRPC_LOGFILE   默认：None
    用于记录对web服务的HTTP请求的文件。如果未设置的web日志被发送到标准的剪贴日志。
    -  JSONRPC_PORT      默认：[6080,7030]
    用于web服务的端口范围。如果设置为None或0，则使用动态指定的端口。
    -  JSONRPC_HOST      默认：'127.0.0.1'
    web服务应该侦听的接口。