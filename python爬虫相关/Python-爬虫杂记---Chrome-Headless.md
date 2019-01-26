# Chrome Headless使用

    测试 Chrome 版本： 62.0.3202.89（正式版本）（64 位)
    Python环境：python2.7

    注： Headless模式需要59版本及以上！

    

Chrome的安装与配置不在此赘述， 不过需要注意的是：
######版本号与驱动的映射关系！
#####版本号与驱动的映射关系！！
###版本号与驱动的映射关系！！！

Chrome与Chromedriver的映射关系表:
![](https://upload-images.jianshu.io/upload_images/11227136-629d0be42502cd24.PNG?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
Chromedriver下载链接：http://chromedriver.storage.googleapis.com/index.html



##### 1. 使用Headless模式

	from selenium import webdriver
    from selenium.webdriver.chrome.options import Options

	chrome_options = Options()
	# 无头模式启动
	chrome_options.add_argument('--headless')
	# 谷歌文档提到需要加上这个属性来规避bug
	chrome_options.add_argument('--disable-gpu')
	# 初始化实例
	driver= webdriver.Chrome(chrome_options=chrome_options）
	# 请求百度
	driver.get("http://www.baidu.com")

##### 2. 禁用图片
###### 2.1  网上大多数的资料给的是以下这种方式， 但是在Headless的模式下并没有生效， 在非Headless的模式下是生效的。
    prefs = {"profile.managed_default_content_settings.images": 2}
    chrome_options.add_experimental_option("prefs", prefs)

######  2.2 以下的这种方式在Headless的模式下是生效的， 非Headless模式下也是生效的。
		
	chrome_options.add_argument('blink-settings=imagesEnabled=false')
		
##### 3. 添加代理

	chrome_options.add_argument("--proxy-server=http://" + ip：port)


##### 4. 修改User-Agent
	
	Chrome Headless模式为什么要修改User-Agent, 来看一下同一个浏览器不同模式下的User-Agent：
	"User-Agent":"Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) HeadlessChrome/62.0.3202.89 Safari/537.36"
	"User-Agent":"Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/62.0.3202.89 Safari/537.36"
	# 修改User-Agent
	chrome_options.add_argument('user-agent= '你想修改成的User-Agent')

##### 5. 打开新的标签页
	#打开空白标签页的方式有很多， 在此只演示一种
	js='window.open("https://www.jianshu.com/p/4fef4142b33f");'
	driver.execute_script(js) # 通过打开新的标签页， 可以节省浏览器打开的时间，减少资源的浪费。

##### 6. 关闭新打开的标签页
    # 分两步走
    # 1.获取标签页的句柄
    handlesList = driver.window_handles  # 返回一个浏览器中所有标签的句柄列表， 顺序为打开窗口的顺序
    # 2. 切换窗口， 关闭标签
    driver.switch_to.window(handlesList[0]) # 切换到百度标签
    driver.close()    # 关闭标签，这里必须用 driver.close() ，用driver.quit()会导致浏览器关闭

##### 7. driver使用完之后切记要`driver.quit()`， 不然或导致内存爆满

#####8. 其他：
 切换标签的时候，我之前用的方法会出现这种状况， 原因是该方法不支持了：![](https://upload-images.jianshu.io/upload_images/11227136-f93e2f6050448a4f.PNG?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
不支持的不只是这一种， 还有如下：
   

    driver.switch_to_active_element()       ==>    driver.switch_to.active_element()            定位到当前聚焦的元素上

    driver.switch_to_alert()                ==>    driver.switch_to.alert()                     切换到alert弹窗
    
    driver.switch_to_default_content()      ==>    driver.switch_to.default_content()           切换到最上层页面
                                                   
    driver.switch_to_frame(frame_reference)  ==>   driver.switch_to.frame(frame_reference)       通过id、name、element(定位的某个元素)、索引来切换到某个frame

    driver.switch_to_window()                ==>   driver.switch_to.window()                     切换到指定的标签页

    driver.switch_to.parent_frame()  switch_to中独有，可以切换到上一层的frame，对于层层嵌套的frame很有用

    注： 用pycharm编程的时候还是提示switch_to_window， 但此方法已经不能用了！！



####  *本人水平有限， 如有错误欢迎提出指正！如有参考， 请注明出处！！禁止抄袭，遇抄必肛！！！*


