# execjs 使用

    有了selenium+Chrome Headless 加载页面为什么还要用execjs来运行js？
       selenium+Chrome Headless  必然是爬虫的一大利器，可是缺点依然存在， 性能问题不可忽视。
    但这构不成舍弃它而不用的理由。我认为舍弃包括Chrome Headless、PhantomJS在内的无头浏览器
    的原因主要有以下几点：
             1. 页面结构改变、弹窗(一些网站的页面结构经常无规则改变)， 影响代码的健壮性。
             2. 无头浏览器的应用场景主要是一些模拟登陆账号密码加密的场景， 爬虫全程使用无头浏览器， 影响性能和效率， 浪费资源。
             3. 通过js加密的网站， 可以看得到加密过程，可以拿得到加密源码。

#### 1. 安装 
    pip install PyExecJS  # 需要注意， 包的名称：PyExecJS  

#### 2. 简单使用
    import execjs
    execjs.eval("new Date")
    返回值为： 2018-04-04T12:53:17.759Z
    execjs.eval("Date.now()")
    返回值为：1522847001080  # 需要注意的是返回值是13位， 区别于python的time.time()
`需要注意的是： 个别的JS语句， 用execjs返回的结果跟浏览器环境返回的结果是有区别的， 以下是浏览器环境返回的结果`
![浏览器环境运行的结果](https://upload-images.jianshu.io/upload_images/11227136-f5d1af18660b275b.PNG?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


#### 3. 调用函数
      # 实际生产中处理的js有几百几千行， 不方便贴上来。来看一下源码中给的例子：
      ctx = execjs.compile("""
            function add(x, y) {
                    return x + y;
               }
    """)
      ctx.call("add", 1, 2)  # 第一个参数 “add” 为JS函数名的字符串， 后边依次为实参
      返回值：3
      
execjs的用法非常简单， 下边来看一下执行JS的环境， 以及性能：
#### 4. 执行JS的环境
    # 1. 在windows上不需要其他的依赖便可运行execjs， 也可以调用其他的JS环境
       # windows 默认的执行JS的环境
           execjs.get().name
           返回值： JScript
       # 作者本人的windows上装有Node.js ， 所以返回值不同
          execjs.get().name
          返回值： Node.js (V8)

    #2. 在ubuntu下需要安装执行JS环境依赖, 作者的环境为PhantomJS
           execjs.get().name
           返回值： PhantomJS

    #3. 源码中给出， 可执行execjs的环境：
      PyV8           = "PyV8"
      Node           = "Node"
      JavaScriptCore = "JavaScriptCore"
      SpiderMonkey   = "SpiderMonkey"
      JScript        = "JScript"
      PhantomJS      = "PhantomJS"
      SlimerJS       = "SlimerJS"
      Nashorn        = "Nashorn"

###### `注1：作者之前在ubuntu环境下执行execjs碰见过因为没有环境而报错，因时间久远，无法肯定。 现在环境齐全， 报错无法复原，如有读者出现错误， 请留言， 多谢！`
###### `更新注1：经过朋友老冀的指正(在此感谢)，在ubuntu环境下， 没有JS环境会报错:Could not find an available JavaScript runtime. 由此可见， execjs在ubuntu需要安装JS环境 。具体的JS环境需根据具体的需求安装， 切不可超过以上8种。`
  
#### 5.环境切换
        # 1. 通过os.environ
        os.environ["EXECJS_RUNTIME"] = "Node"
        execjs.get().name
        execjs.eval("1 + 2")
        # 2. 通过execjs.get 切换
         jscript = execjs.get(execjs.runtime_names.JScript)  # runtime_names 便是execjs源码中给出的执行环境的。 execjs.runtime_names.xxx  xxx必须在上一节 #3中取
         jscript.eval("1 + 2")

  `注: 在切换环境时， 当环境不存在不会报错， 会使用默认的环境。 另外需要注意的是， 两种方式的区别`



#### 6. 简易性能分析
     # 作者只简单试了三种， 在windows下
    import  execjs
    import os
    import time

    # 先用JScript
    os.environ["EXECJS_RUNTIME"] = "JScript"
    print execjs.get().name

    time1 = time.time()
    for i in range(100):
        execjs.eval("new Date")
    print time.time() - time1

    # 切换环境 使用Nodejs
    os.environ["EXECJS_RUNTIME"] = "Node"
    print execjs.get().name

    time2 = time.time()
    for l in range(100):
        execjs.eval("new Date"）
    print time.time() - time2

    # 打印的结果为：
    JScript
    4.70900011063
    Node.js (V8)
    27.501999855

    # 在ubuntu下试的是PhantoJS ， 结果竟然高达 30+ S


`此注释来自execjs作者：PyExecJS的缺点之一就是性能。PyExecJS通过文本传递JavaScript运行时，并且速度很慢。另一个缺点是它不完全支持运行时特定的功能。对于某些用例，PyV8可能是更好的选择。`


# `总注：使用execjs的难点并不是在execjs这个库， 而是解析JS的过程， 因为没有浏览器的环境， 没有加密源码的依赖。从成千上万行的JS中择出想要的内容，可能是一段孤零零的JS函数，也可能是从几个JS文件去找出各自找出一段JS代码， 并可以通过execjs顺利执行， 这并非易事。 需要慢慢积累经验。 一旦掌握， 便可以提高爬虫的效率， 以及代码的健壮性， 节省资源！`

####  *本人水平有限， 如有错误欢迎提出指正！如有参考， 请注明出处！！禁止抄袭，遇抄必肛！！！*
















    
    

