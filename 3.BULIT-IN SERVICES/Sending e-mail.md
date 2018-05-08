### 发送邮件  
尽管Python通过smtplib库使发送电子邮件变得相对容易，但是scrapy提供了它自己的发送电子邮件的工具，使得电子邮件很容易，而且它是使用 Twisted库的 non-blocking IO实现的。为了避免干扰爬虫的non-blocking IO，它还提供了一个简单的API来发送附件，并且一些配置很容易配置。
#### 简单的例子  
有两种方法来实例化邮件发送者。您可以使用标准构造器实例化它：

```
from scrapy.mail import MailSender
mailer = MailSender()
```
或者你可以通过scrapy设置一个对象来实例化它，它会遵守settings：

```
mailer = MailSender.from_settings(settings)
```
下面是如何使用它发送电子邮件（没有附件）：

```
mailer.send(to=["someone@example.com"], subject="Some subject", body="Some body", cc=["another@example.com"])
```
#### Mailspider类引用：
MailSender是用于从剪贴簿发送电子邮件的首选类，因为它像框架的其余部分一样使用了non-blocking IO。


>   class scrapy.mail.MailSender(smtphost=None, mailfrom=None, smtpuser=None, smtppass=None, smtpport=None)


参数：
- smtphost（str or bytes）：用来发送邮件的SMTP（简单邮件传送协议服务器），如果省略，将使用MAIL_HOST设置。
- mailfrom（str）：他曾经发送电子邮件的地址（在From:header中）。如果省略，将使用MAIL_FROM设置。
- smtpuser：SMTP的用户，如果省略，将使用 MAIL_USER设置。如果没有给出，将不会执行SMTP认证。
- smtppass (str or bytes)：用于身份验证的SMTP通行证。
 - smtpport (int)：连接到的SMTP端口。
- smtptls (boolean)：强制使用SMTP STARTTLS。
- smtpssl (boolean)：强制使用安全SSL连接。


>  classmethod from_settings(settings)

使用scrapy设置实例化对象，它将尊重这些scrapy设置  
参数：settings (scrapy.settings.Settings 对象) -电子邮件收件人
>  send(to, subject, body, cc=None, attachs=(), mimetype='text/plain', charset=None)     

发送电子邮件给指定的收件人。  
参数：  
- to (str or list of str) – 电子邮件收件人  
- subject (str) - 电子邮件的主题  
- cc (str or list of str) -  抄送的邮件
- body (str) – 电子邮件的内容
- attachs (iterable) - 一个元祖(attach\_name, mimetype, file\_object) ，其中attach\_name是一个字符串，它的名字将出现在电子邮件附件中，mimetype是附件的MIME类型，file\_object是一个可读的文件对象，带有附件的内容
- mimetype (str) – 电子邮件的MIME类型
- charset (str) – 用于电子邮件内容的字符编码  
#### 邮件设置   
这些设置定义了MailSender类的默认构造器值，并且可以用来在您的项目中配置电子邮件通知，而不需要编写任何代码（对于那些使用MailSender的扩展和代码）。
##### MAIL_FROM  
默认：'scrapy@localhost'  
邮件的发送者（从header）  
##### MAIL_HOST  
默认：'localhost'  
用于发送电子邮件的SMTP主机  
##### MAIL_PORT 
默认：25 
用于发送电子邮件的SMTP端口 
##### MAIL_USER 
默认：None 
用户可用于SMTP身份验证。如果禁用了SMTP身份验证，则将执行  
##### MAIL_TLS 
默认：False
强制使用STARTTLS. STARTTLS 是一种利用SSL/TLS将其升级到安全连接的方法  
##### MAIL_SSL 
默认：False 
使用SSL加密连接强制连接  