好久没有用过scrapy框架，甚至有一些手生了。学习东西最痛苦的事情是，学了的东西不用。过一段时间我都怀疑自己是否学过了。 还是古话说的好，温故而知新。在这里记录一下感觉重要的几个点。先从简单的来。

## 0.设置User-Agent
设置User-Agent的方法有很多种，其他的都是设置配置之类的。这里只说一个通过改写middlewares的。
###### 0.0.1 首先来说一个第三方库[fake_useragent](https://github.com/hellysmile/fake-useragent), 这个库是可以提供随机的User-Agent，免去了自己维护User-Agent的麻烦。
 
    # 安装fake_useragent
    pip install fake-useragent
    # 使用示例
    from fake_useragent import UserAgent
    ua = UserAgent()
    ua.ie
    # Mozilla/5.0 (Windows; U; MSIE 9.0; Windows NT 9.0; en-US);
    ua.msie
    # Mozilla/5.0 (compatible; MSIE 10.0; Macintosh; Intel Mac OS X 10_7_3; Trident/6.0)'
    ua['Internet Explorer']
    # Mozilla/5.0 (compatible; MSIE 8.0; Windows NT 6.1; Trident/4.0; GTB7.4; InfoPath.2; SV1; .NET CLR       3.3.69573; WOW64; en-US)
    ua.opera
    # Opera/9.80 (X11; Linux i686; U; ru) Presto/2.8.131 Version/11.11
    ua.chrome
    # Mozilla/5.0 (Windows NT 6.1) AppleWebKit/537.2 (KHTML, like Gecko) Chrome/22.0.1216.0     Safari/537.2'
    ua.google
    # Mozilla/5.0 (Macintosh; Intel Mac OS X 10_7_4) AppleWebKit/537.13 (KHTML, like Gecko) Chrome/24.0.1290.1 Safari/537.13
    ua['google chrome']
    # Mozilla/5.0 (X11; CrOS i686 2268.111.0) AppleWebKit/536.11 (KHTML, like Gecko) Chrome/20.0.1132.57 Safari/536.11
    ua.firefox
    # Mozilla/5.0 (Windows NT 6.2; Win64; x64; rv:16.0.1) Gecko/20121011 Firefox/16.0.1
    ua.ff
    # Mozilla/5.0 (X11; Ubuntu; Linux i686; rv:15.0) Gecko/20100101 Firefox/15.0.1
    ua.safari
    # Mozilla/5.0 (iPad; CPU OS 6_0 like Mac OS X) AppleWebKit/536.26 (KHTML, like Gecko) Version/6.0 Mobile/10A5355d Safari/8536.25

    # 随机生成一个UA
    ua.random
###### 0.0.2 编写middleware.py 文件中的类
    from fake_useragent import UserAgent
    class RandomUserAgentMiddleware(object):

        def __init__(self, crawler):
            super(RandomUserAgentMiddleware, self).__init__()
            self.ua = UserAgent()

        @classmethod
        def from_crawler(cls, crawler):
            return cls(crawler)

        def process_requests(self, request, spider):
            request.headers.setdefault("User-Agent", self.ua.random)

###### 0.0.3 修改setting文件
     # 值得注意的是原生的UserAgentMiddleware 必须设置为None。
    DOWNLOADER_MIDDLEWARES = {
    'spiderProject.middlewares.RandomUserAgentMiddleware': 543,
    "scrapy.downloadermiddlewares.useragent.UserAgentMiddleware": None
    }

## 1.设置全局代理
同样是编写middleware。这个是非常的简单。然后配置到setting文件中就ok。

    class ProxyMiddleware(object):
        def process_requests(self, request, spider):
            request.meta["proxy"] = "http://121.61.0.5:9999"


## 3.scrapy与selenium结合
###### 3.1 首先编写中间件
    class SeleniumRequestsMiddleware(object):
        def process_request(self, request, spider):
            if spider.name == "baidu":
                driver = webdriver.Chrome()
                driver.get(request.url)
                import time
                time.sleep(3)
                from scrapy.http import HtmlResponse  # scapy会将HtmlResponse 直接返回，所以这里要使用这个
               return HtmlResponse(
                    url=driver.current_url,
                    body=driver.page_source,
                    encoding="utf-8",
                    request=request
                )

这段代码其实是有问题的：
1. 只要满足spider.name == "baidu"的请求都会创建一个driver，会极大的影响请求速度。
2. 当有信号spider close的时候，不一定会执行driver.quit()，内存不一定释放，所以要对关闭进行处理。
3. 当一个项目同时启动多个spider，会共用到Middleware中的selenium，不利于并发。

###### 3.2 针对以上问题的解决方案

     # middleware.py 文件
    class SeleniumRequestsMiddleware(object):
        def process_request(self, request, spider):
           if spider.name == "baidu":
                spider.driver.get(request.url)
                import time
                time.sleep(3)
                from scrapy.http import HtmlResponse
                return HtmlResponse(
                    url=spider.driver.current_url,
                    body=spider.driver.page_source,
                    encoding="utf-8",
                    request=request
                )

    # spider文件  baidu.py
    class BaiduSpider(scrapy.Spider):
        name = 'baidu'
        allowed_domains = ['baidu.com']
        start_urls = ['http://www.baidu.com/']

        def __init__(self): # 在这里可以定制一些webdriver.Chrome()的相关配置
            self.driver = webdriver.Chrome()
            super(BaiduSpider, self).__init__()
            dispatcher.connect(receiver=self.driver_close,  
                           signal=signals.spider_closed)  # 通过信号控制，当是spider_closed信号时，调用driver_close.

        def driver_close(self, spider):
            self.driver.quit()


问题： 
 这里其实并没有实现并发操作，这个我从github上边，有一个实例[scrapy-phantomjs-downloader](https://github.com/flisky/scrapy-phantomjs-downloader/blob/master/scrapy_phantomjs/downloader/handler.py)，重写了downloader,如果需要可以进行参考。

## 4. 关于setting
setting.py文件中已经有了很多全局化的配置，当然，还有很多配置并没有提示出来,参考[setting设置](https://doc.scrapy.org/en/latest/topics/settings.html#how-to-access-settings), 这里需要注意的是其配置的优先级，否则很容易出问题。这里重点说两个需要注意的地方。

###### 4.1 为每个spider配置私有配置

    class MySpider(scrapy.Spider):
        name = 'myspider'
        custom_settings = {
            'SOME_SETTING': 'some value',
        }
    # 这个优先级要比settings.py中的要高，通过custom_settings中的配置会覆盖settings.py中的配置。

###### 4.2 在spider文件中过去设置，可以用self.settings获取配置
    class MySpider(scrapy.Spider):
        name = 'myspider'
        start_urls = ['http://example.com']

        def parse(self, response):
            print("Existing settings: %s" % self.settings.attributes.keys())

###### 4.3可以通过`scrapy.crawler.Crawler.settings`获取配置文件中的配置

    #Crawler可以通过settings属性传递给from_crawler扩展，中间件和项目管道中，固定的写法如下：
    @classmethod
    def from_crawler(cls, crawler):
        settings = crawler.settings
        return cls(crawler)
    

######注：`settings`在初始化蜘蛛之后，在基础Spider类中设置该属性。如果要在初始化之前使用设置（例如，在spider的`__init__()`方法中），则需要覆盖该 `from_crawler()`方法。

##5. 关于信号Signals
signals感觉挺重要的，了解信号的机制，对于拓展scrapy来讲相当重要，这里不做多解释，下边是[官网](https://doc.scrapy.org/en/latest/topics/signals.html#deferred-signal-handlers) 上的一个例子：

    from scrapy import signals
    from scrapy import Spider


    class DmozSpider(Spider):
        name = "dmoz"
        allowed_domains = ["dmoz.org"]
        start_urls = [
            "http://www.dmoz.org/Computers/Programming/Languages/Python/Books/",
            "http://www.dmoz.org/Computers/Programming/Languages/Python/Resources/",
        ]

        @classmethod
        def from_crawler(cls, crawler, *args, **kwargs):
            spider = super(DmozSpider, cls).from_crawler(crawler, *args, **kwargs)
            crawler.signals.connect(spider.spider_closed, signal=signals.spider_closed)
            return spider

        def spider_closed(self, spider):
            spider.logger.info('Spider closed: %s', spider.name)

        def parse(self, response):
            pass



# *`注： 关于scrapy的框架好久没用了，暂时想到这么多。深入理解爬虫中间件、下载中间件以及信号处理的方式，那么拓展scrapy是相当方便的，关于信号的处理我这里也是比较薄弱，还更需努力，欢迎读者老爷不吝指教，谢谢 `*












