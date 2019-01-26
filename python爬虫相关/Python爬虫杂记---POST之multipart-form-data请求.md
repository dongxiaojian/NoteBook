# 模拟multipart/form-data请求
`原以为requests请求十分强大， 但遇到了模拟multipart/form-data类型的post请求， 才发现requests库还是有一丢丢的不足。 不过也可能是我理解的不足， 还希望读者老爷不吝指教！ 在此感谢！`

#### 1. 什么是multipart/form-data请求

   enctype属性: 
    enctype：规定了form表单在发送到服务器时候编码方式，它有如下的三个值。 
    ①application/x-www-form-urlencoded：默认的编码方式。但是在用文本的传输和MP3等大型文件的时候，使用这种编码就显得 效率低下。 
    ②multipart/form-data：指定传输数据为二进制类型，比如图片、mp3、文件。
    ③text/plain：纯文体的传输。空格转换为 “+” 加号，但不对特殊字符编码。

![enctype在html中的样子](https://upload-images.jianshu.io/upload_images/11227136-6586368a6a185c33.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


#### 2. multipart/form-data请求请求体的格式(以某网站模拟登录为例)

![multipart请求体的格式](https://upload-images.jianshu.io/upload_images/11227136-81b32e28c183b97f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

值得注意的是：请求头的Content-Type属性与其他post请求的不同

#### 3. 实现请求体的拼接
###### 3.1 第一种：使用 requests库
    # coding: utf-8
    from collections import OrderedDict
    import requests

    # 构建有序字典
    params = OrderedDict([("username", (None, '130533193203240022')),
                      ("password", (None, 'qwerqwer')),
                      ('captchaId', (None, 'img_captcha_7d96b3cd-f873-4c36-8986-584952e38f20')),
                      ('captchaWord', (None, 'rdh5')),
                      ('_csrf', (None, '200ea95d-90e9-4789-9e0b-435a6dd8b57b'))])

    res = requests.get('http://www.baidu.com', files=params)
    print res.request.body

     # 打印的结果：
    --6c7a1966e0294e1cb89b06b95cf3da84
    Content-Disposition: form-data; name="username"

     130533193203240022
     --6c7a1966e0294e1cb89b06b95cf3da84
    Content-Disposition: form-data; name="password"

    qwerqwer
    --6c7a1966e0294e1cb89b06b95cf3da84
    Content-Disposition: form-data; name="captchaId"

    img_captcha_7d96b3cd-f873-4c36-8986-584952e38f20
    --6c7a1966e0294e1cb89b06b95cf3da84
    Content-Disposition: form-data; name="captchaWord"

    rdh5
    --6c7a1966e0294e1cb89b06b95cf3da84
    Content-Disposition: form-data; name="_csrf"

    200ea95d-90e9-4789-9e0b-435a6dd8b57b
    --6c7a1966e0294e1cb89b06b95cf3da84--

    需要注意的是， 可以发现分隔符是随机生成的， 跟制定的不太一样， 这需要我们自己手动替换  
    # 替换使用的re
     temp = re.search(r'--(.*)--', res.request.body).group(1)                          
     data = re.sub(temp, '----WebKitFormBoundaryKPjN0GYtWEjAni5F', res.request.body)   

     注：这种方法可以构建想要的请求体， 麻烦的是分隔符并不是制定的那样，而是默认的 uuid4().hex   需要手动替换。 files可以接收的参数， 
         源码中解释截图在文末。

###### 3.2 第二种：使用 encode_multipart_formdata函数
    # coding: utf-8
    from collections import OrderedDict
    from urllib3 import encode_multipart_formdata

    params = OrderedDict([("username", (None, '130533193203240022', 'multipart/form-data')),
                      ("password", (None, 'qwerqwer', 'multipart/form-data')),
                      ('captchaId', (None, 'img_captcha_7d96b3cd-f873-4c36-8986-584952e38f20', 'multipart/form-data')),
                      ('captchaWord', (None, 'rdh5', 'multipart/form-data')),
                      ('_csrf', (None, '200ea95d-90e9-4789-9e0b-435a6dd8b57b','multipart/form-data'))])
    m = encode_multipart_formdata(params, boundary='----WebKitFormBoundaryKPjN0GYtWEjAni5F')
    print m[0]

    # 打印结果
    ------WebKitFormBoundaryKPjN0GYtWEjAni5F
    Content-Disposition: form-data; name="username"
    Content-Type: multipart/form-data

    130533193203240022
    ------WebKitFormBoundaryKPjN0GYtWEjAni5F
    Content-Disposition: form-data; name="password"
    Content-Type: multipart/form-data

    qwerqwer
    ------WebKitFormBoundaryKPjN0GYtWEjAni5F
    Content-Disposition: form-data; name="captchaId"
    Content-Type: multipart/form-data

    img_captcha_7d96b3cd-f873-4c36-8986-584952e38f20
    ------WebKitFormBoundaryKPjN0GYtWEjAni5F
    Content-Disposition: form-data; name="captchaWord"
    Content-Type: multipart/form-data

    rdh5
    ------WebKitFormBoundaryKPjN0GYtWEjAni5F
    Content-Disposition: form-data; name="_csrf"
    Content-Type: multipart/form-data

    200ea95d-90e9-4789-9e0b-435a6dd8b57b
    ------WebKitFormBoundaryKPjN0GYtWEjAni5F--

       可以看得到， 这种方法多出来一个 Content-Type（我传递的参数中指定了这个值，
    如果没有指定，这个Content-Type依然存在，值为：application/octet-stream），
    我现在也没有太确定多的这个值对最后的结果有没有影响。还没试...[手动捂脸]

#### 4. 最终的请求
###### 参数拼接完， 最终的请求要用post， 参数是data, 不要再用files。记得Headers的Content-Type
##### 参数拼接完， 最终的请求要用post， 参数是data， 不要再用files。记得Headers的Content-Type
`总注：上边这两种构建参数的方式各有不同， 用起来感觉并不是那么的灵活，所以感叹requests有那么一丢丢丢的不足。值的注意的是，requests.post中files参数接收字典的形式和encode_multipart_formdata中params参数接收字典形式的区别！人生苦短......`

####  *本人水平有限， 如有错误欢迎提出指正！如有参考， 请注明出处！！禁止抄袭，遇抄必肛！！！*
![files接收参数的格式](https://upload-images.jianshu.io/upload_images/11227136-67e4dc7fa798d04e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![boundary的取值](https://upload-images.jianshu.io/upload_images/11227136-9c470e8789aef23c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![默认的boundary](https://upload-images.jianshu.io/upload_images/11227136-ed1ed12207bdc9ef.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![源码中这个函数boundary默认为None](https://upload-images.jianshu.io/upload_images/11227136-46b93e9b2556fb16.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
























