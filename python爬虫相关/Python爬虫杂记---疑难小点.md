 # 疑难杂项
 ###### 这篇的文章没有归类，也不太好归类。 在爬虫中经常用到的问题点。 比如说编码问题， 编码问题在Python中简直就是一门玄学， 在爬虫中简直是玄而又玄。再比如打印请求头， 使用selenium时的定位截图等等，都有自己固定的使用场景。
    # coding:utf-8
    import requests
    headers = {
    'User-Agent':'Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:59.0) Gecko'
    }
    #基本代码
    response = requests.get('https://www.jianshu.com/u/10b8bef7b005', headers=headers)
#### 1. 编码问题
###### 1.1通用方法`response.content.decode(response.encoding)`
        
    response.encoding  # 返回网页的编码格式
    response.content.decode(response.encoding)  # 这种方法就是将返回的内容通过网页本身的编码解码， 可以在通过encode编码成指定格式

###### 2.2 特殊方法`处理GBK编码的格式`
     #存在这样一种情况， 当 response.encoding 的值为 GBK 时， 即网页编码为GBK时， 有时解码会报错， 原因可能是GBK的编码不兼容造成的
     response.content.decode('GB18030')  #当将GBK换为GB18030这种编码格式即可解决，GB18030比GBK包含的字符更多


#### 2. 不标准的json数据转换
    # 返回json数据的Key不带引号的， 无法通过json.loads直接转换， 用以下方法可以转换成dict
    #json_similar 表示不标准的json数据
    dic = eval(json_similar, type('Dummy', (dict,), dict(__getitem__=lambda s, n: n))())
    return dic   # 返回标准的字典

#### 3. 打印除请求头的信息
      # 打印请求头
    import httplib
    def rewrite_send():
       old_send = httplib.HTTPConnection.send
       def new_send(self, data):
         print data
         return old_send(self, data)
       httplib.HTTPConnection.send = new_send、
    # 执行函数
    rewrite_send()  # 需要注意的时这个函数应在请求发出之前执行
    # 以下是执行文章开头基本代码后的结果
    GET /u/10b8bef7b005 HTTP/1.1
    Host: www.jianshu.com
    Connection: keep-alive
    Accept-Encoding: gzip, deflate
    Accept: */*
    User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:59.0) Gecko

#### 4. 定位截图`selenium截取验证码常用`
###### 4.1 使用xpath对元素定位的方法
      # 截取带有验证码的网页
     imgelement = driver.find_element_by_xpath('//a[@id="yzm_top"]/img')
      #  定位验证码
       location = imgelement.location
      #  获取验证码x,y轴坐标
      size = imgelement.size
      #  获取验证码的长宽
      rangle = (int(location['x']), int(location['y']), int(location['x'] + size['width']), int(location['y'] + size['height']))
      #  写成我们需要截取的位置坐标
      i = Image.open(file_path+'temp.png')
      #  打开截图
      frame = i.crop(rangle)
      #  使用Image的crop函数，从截图中再次截取我们需要的区域
      frame.save('temp2.png')
   ###### 4.2 手动找坐标的方法
      #bbox是一个元组，是相对于大图片左上角为原点的小图的左上角的像素点的坐标和右下角的像素点的坐标， windows可以用画图工具找到对象点的坐标
      bbox = (831, 249, 891, 280)
      img = Image.open('a.png')  # a.png 表示含有验证码的图片
      cropimg = img.crop(bbox)
      cropimg.save('yzm5.png')



#### 5.生成随机字符串
    # 这个是小贱在给一些不太重要的文件命名的时候经常用的，根据业务要求，需要将爬下来的网页内容保存， 所以这个是小贱经常用的
    # 62 表示字符串位数， 也是最大的生成字符串的位数， 可以改为自己需要的位数
  
    ran_str = ''.join(random.sample(string.ascii_letters + string.digits, 62))
    print ran  # 生成的字符串位数是数字字母组合
    # 打印结果
    F5hXLmiPl1v94QOtapM3qS0zBoYVZTfAynwjk6HJs27xCcIubd8GNRrUEWDeKg
  
#### 6. 月份处理
    # 爬虫在写按月份提取内容的时候， 手写提取规则，月份加减，这个方法挺好用， 当时初见， 相见恨晚。
    import datetime
    from dateutil.relativedelta import relativedelta
    print datetime.date.today()+ relativedelta(months=-1)  # 参数months为负表示前一个月， months为正表示后一个月 

####  *本人水平有限， 如有错误欢迎提出指正！如有参考， 请注明出处！！禁止抄袭，遇抄必肛！！！*




    


















