`头一段时间做了某网站的滑动验证码， 用的是阿里的滑动验证码。用自动化模拟滑块的拖动， 然而尝试了多种方法， 仍没能成功。最终得出结论，阿里的反爬做的太好了。虽然没能成功， 但是自动化的模拟点击拖动的经验算是积累了， 遂记录之....`

    初始代码：
    # coding: utf-8
    from selenium import webdriver
    drivers =webdriver.Chrome()
    browser.get('http://www.runoob.com/try/try.php?filename=jqueryui-api-droppable')
####1. 通过ActionChains进行模拟点击拖动
######1.1 通过元素进行模拟点击拖动
    from selenium.webdriver import ActionChains
    drivers.switch_to.frame('iframeResult')   # 切换到指定的frame
    source = drivers.find_element_by_id('draggable')      # 被拖拽的对象
    target = drivers.find_element_by_id('droppable')      # 目标对象
    action = ActionChains(drivers)  # 实例action
    action.drag_and_drop(source, target)  # 将需要执行的步骤加入到对列
    action.perform() # 执行队列中的操作
######1.2 通过相对坐标模拟点击拖动
    from selenium.webdriver import ActionChains
    drivers.switch_to.frame('iframeResult')   # 切换到指定的frame
    source = drivers.find_element_by_id('draggable')      # 被拖拽的对象
    action = ActionChains(drivers) # 实例action
    action.click_and_hold(source).perform() #单击并保持
    action.move_by_offset(xoffset=250, yoffset=0) # 移动的距离， 相对坐标
    action.release(source) # 释放单击
    action.perform() # 执行队列中的操作

注：这类方法更容易被反爬识别，这个操作更像是js执行的点击、拖动事件。

####2.通过pyautogui进行鼠标拖动(查看资料pyautogui是跨平台的)
    import pyautogui
    pyautogui.mouseDown(700, 400)  # 鼠标按下的坐标，相对于屏幕的左上角
    pyautogui.mouseUp(950, 388)  # 鼠标抬起的坐标，相对于屏幕的左上角


`总注：这两类不止能调用鼠标，也可以控制键盘。python中调用键鼠的方法很多，像在win中还有pywin32等等。 以上两种只是简单介绍了鼠标拖动的方式，算是处理滑动验证码的主要用的几种形式， 算是抛砖引玉。`

参考文章：
[pyautogui参考文章：csdn作者:蜗v牛](https://blog.csdn.net/ibiao/article/details/77859997)
[ActionChains参考文章：csdn作者:huilan_same](https://blog.csdn.net/huilan_same/article/details/52305176)


####  *本人水平有限， 如有错误欢迎提出指正！如有参考， 请注明出处！！禁止抄袭，遇抄必肛！！！*



