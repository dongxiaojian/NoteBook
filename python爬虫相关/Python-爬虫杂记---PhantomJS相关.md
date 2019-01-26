## selenium  phantomJS相关
### 1. 设置phantomJS请求头：
    from selenium import webdriver
    from selenium.webdriver.common.desired_capabilities import DesiredCapabilities
 
    dcap = dict(DesiredCapabilities.PHANTOMJS)  
    dcap["phantomjs.page.settings.userAgent"] = "Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/52.0.2743.116 Safari/537.36"
    driver = webdriver.PhantomJS(executable_path='./phantomjs.exe', desired_capabilities=dcap) 
### 2.设置PhantomJS禁用图片 
*注：以下设置禁用图片的方法， 并不适用于Chrome*

     dcap["phantomjs.page.settings.loadImages"] = False  # 这步的配置在1的基础上实现。

### 3. PhantomJS请求https没内容解决：
![](https://upload-images.jianshu.io/upload_images/11227136-2172e68fc02ff83a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

    driver = webdriver.PhantomJS(service_args=['--ignore-ssl-errors=true', '--ssl-protocol=any']) # 这个设置应该是忽略ssl协议的设置

### 新增 3.1 PhantomJs 添加代理

    service_args = [
    '--proxy=%s' % ip:prot,     # 代理 ip：prot    （比如：192.168.22.22:8080）
    '--proxy-type=http’,        # 代理类型：https/http
    ‘--load-images=no’,         # 关闭图片加载
    '--disk-cache=yes’,         # 开启缓存
    '--ignore-ssl-errors=true’,  # 忽略https错误
    '--ssl-protocol=any'，       # ssl验证任何， 配合https错误验证
    ]
    driver = webdriver.PhantomJS(service_args=service_args)  
    注： 禁止加载图片得方法这两种都可以。


### 新增 3.2 PhantomJs 设置日志文件路径
    service_log_path='/dong/logs/phantomjs.log'   # 日志文件路径以及文件名
    driver = webdriver.PhantomJS(service_log_path=service_log_path)


### 4. 判断元素存不存在：
    is_disappeared = WebDriverWait(driver, 8, 0.5, ignored_exceptions=TimeoutException).until(lambda x: x.find_element_by_id("id").is_displayed())

### 5. selenium一些不常用的方法：
    get_attribute() :可以获取文本框中的值

    title_is: 判断当前页面的title是否精确等于预期

     title_contains: 判断当前页面的title是否包含预期字符串

    presence_of_element_located: 判断某个元素是否被加到了dom树里，并不代表该元素一定可见

    visibility_of_element_located: 判断某个元素是否可见.可见代表元素非隐藏，并且元素的宽和高都不等于0

    visibility_of: 跟上面的方法做一样的事情，只是上面的方法要传入locator，这个方法直接传定位到的element就好了

    presence_of_all_elements_located: 判断是否至少有1个元素存在于dom树中。举个例子，如果页面上有n个元素的    class都是'column-md-3'，那么只要有1个元素存在，这个方法就返回True

    text_to_be_present_in_element: 判断某个元素中的text是否包含了预期的字符串

    text_to_be_present_in_element_value: 判断某个元素中的value属性是否包含了预期的字符串

    frame_to_be_available_and_switch_to_it: 判断该frame是否可以switch进去，如果可以的话，返回True并且switch进去，否则返回False

    invisibility_of_element_located: 判断某个元素中是否不存在于dom树或不可见

    element_to_be_clickable: 判断某个元素中是否可见并且是    enable的，这样的话才叫clickable

    staleness_of: 等某个元素从dom树中移除，注意，这个方法也是返回True或False

    element_to_be_selected: 判断某个元素是否被选中了,一般用在下拉列表

    element_selection_state_to_be: 判断某个元素的选中状态是否符合预期

    element_located_selection_state_to_be: 跟上面的方法作用一样，只是上面的方法传入定位到的element，而这个方法传入locator

    alert_is_present: 判断页面上是否存在alert，这是个老问题，很多同学会问到

####  *本人水平有限， 如有错误欢迎提出指正！如有参考， 请注明出处！！禁止抄袭，遇抄必肛！！！*




  


