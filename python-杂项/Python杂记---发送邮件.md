`最近碰见一个大佬问我python发送邮件的问题， 之前用过， 但也忘了七七八八了，现在趁此机会， 重新整理之， 以作后用。`　

######  Python发送邮件的两个包： `smtplib 用来发送邮件。`  `email 用来构建邮件。`
 #####  `Python 的 email 模块里包含了许多实用的邮件格式设置函数，用来创建邮件。使用的 MIMEText 对象，为底层的MIME协议传输创建了一封空邮件，最后通过SMTP 协议发送出去。 MIMEText 对象 msg 包括收发邮箱地址、邮件正文和主题，Python 通过MIMEText 就可以创建一封格式正确文本邮件。用MIMEMultipart构建附件。smtplib 模块用来设置服务器连接的相关信息。`

###### 这里以网易邮箱为为例：需要打开网易是指开启SMTP服务， 接下来获取授权码，授权码在代码中是用来代替密码来操作的。(QQ邮箱的SMTP服务开启后，代码中需要用SMTP_SSL构建会话实例， 不然会有报错:Connection unexpectedly closed)

![网易邮箱设置](https://upload-images.jianshu.io/upload_images/11227136-b86174158716229c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 1.构建简单的文本文件`MIMEText`

    # coding:utf-8
    import smtplib
    from email.mime.text import MIMEText
    from email.header import Header
    from email.utils import formataddr

    sender = '********'@163.com' # 邮件发送者的邮箱地址
    receivers = '********'@qq.com'  # 邮件接收者的邮箱地址

    # 三个参数：第一个为邮件正文文本内容，第二个 plain 设置文本格式，第三个 utf-8 设置编码
    message = MIMEText('这是一封正常的测试邮件， 并不是异常的！', 'plain', 'utf-8')
    message['From'] = formataddr(["来自简书作者的问候", sender])  # 发送者
    message['To'] = formataddr(["来自简书作者董小贱的问候", receivers])  # 接收者

    subject = '这是一封正常的测试邮件， 并不是异常的！'
    message['Subject'] = Header(subject, 'utf-8') # 邮件的主题

    smtpObj = smtplib.SMTP('smtp.163.com', port=25)
    smtpObj.login(user=sender, password='********')  # password并不是邮箱的密码，而是开启邮箱的授权码
    smtpObj.sendmail(sender, receivers, message.as_string()) # 发送邮件


![与代码中所对应的关系](https://upload-images.jianshu.io/upload_images/11227136-fb0bbefe490b5060.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

2. 构建带附件的邮件`MIMEMultipart` 

       # coding:utf-8
       import smtplib
       from email.mime.multipart import MIMEMultipart
       from email.mime.text import MIMEText
       from email.header import Header
       from email.utils import formataddr

       sender = '********'@163.com'
       receivers = '********'@qq.com'  # 接收邮件，可设置为你的QQ邮箱或者其他邮箱

       message = MIMEMultipart()
       message['From'] = formataddr(["来自简书作者的问候", sender])  # 发送者
       message['To'] = formataddr(["来自简书作者董小贱的问候", receivers])  # 接收者

       subject = '这是一封正常正常邮件， 并不是异常的！'

       message['Subject']= Header(subject, 'utf-8')

       message.attach(MIMEText('这是邮件的正文内容', 'plain', 'utf-8 '))  # 邮件的正文内容
       att1 = MIMEText(open('dongxiaojian.txt').read(), 'base64', 'utf-8')  # 构建邮件附件

       att1["Content-Disposition"] = 'attachment;filename="dongxiaojian.txt"' # 这里的filename就是收到附件的名字
       message.attach(att1)


       smtpObj = smtplib.SMTP('smtp.163.com', port=25)
       smtpObj.login(user=sender, password='********'')
       smtpObj.sendmail(sender, receivers, message.as_string())

  ![附件示意图.png](https://upload-images.jianshu.io/upload_images/11227136-51aff298a34d883e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

3.构建带图片的邮件`MIMEImage`  (在第二步的基础上)

    # coding:utf-8

    import smtplib
    from email.mime.multipart import MIMEMultipart
    from email.mime.text import MIMEText
    from email.mime.image import MIMEImage
    from email.header import Header
    from email.utils import formataddr

    sender = '********'@163.com'
    receivers = '********'@qq.com'  # 接收邮件，可设置为你的QQ邮箱或者其他邮箱

    message = MIMEMultipart()
    message['From'] = formataddr(["来自简书作者的问候", sender])  # 发送者
    message['To'] = formataddr(["来自简书作者董小贱的问候", receivers])  # 接收者

    subject = '这是一封正常正常邮件， 并不是异常的！'
    message['Subject'] = Header(subject, 'utf-8')

    message.attach(MIMEText('这是邮件的正文内容', 'plain', 'utf-8 '))  # 邮件的正文内容
    att1 = MIMEText(open('dongxiaojian.txt').read(), 'base64', 'utf-8')  # 构建邮件附件
    att1["Content-Disposition"] = 'attachment;filename="dongxiaojian.txt"'  # 这里的filename就是收到附件的名字
    message.attach(att1)

    # 以附件的形式添加图片
    f = open('timg.jpeg', 'rb')
    att2 = MIMEImage(f.read()) 
    f.close()
    att2["Content-Disposition"] = 'attachment;filename="danding.jpeg"'  # 这里的filename就是收到附件图片的的名字
    message.attach(att2)

    smtpObj = smtplib.SMTP('smtp.163.com', port=25)
    smtpObj.login(user=sender, password='********'')
    smtpObj.sendmail(sender, receivers, message.as_string())



# `注： 这里主要示例了发送邮件的几种简单的格式，这应付常见的各式已基本够用， 具体的理论知识、以及各个库的详解不再罗列， 需要的话请移步相关的文档。`

####  *本人水平有限， 如有错误欢迎提出指正！如有参考， 请注明出处！！禁止抄袭，遇抄必肛！！！*










