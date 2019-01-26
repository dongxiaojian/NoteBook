#   ttf 文件反爬
 ` 想写这篇文章的起源是在一个技术群里，有人讨论去哪网(手机端)的反爬：请求下来的数字跟浏览器上的数字有规律的不同，查看字体文件之后， 发现字体文件中的数字位置颠倒了...， 后有朋友老冀爬取汽车之家精品贴也出现了类似的情况，不太清楚这种反爬的成本， 但凭直觉将来这种反爬措施可能越来越普遍， 拿汽车之家为例， 遂记录之！源码在最后！！`


#### 1. 开发者模式查看网页内容
![未显示正确字体的方框就是改变了编码格式的字体](https://upload-images.jianshu.io/upload_images/11227136-2d777bb1d6d983a8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
#### 2. 下载网页源码保存至本地查看

![网页源码保存至本地， 显示的乱码](https://upload-images.jianshu.io/upload_images/11227136-a7e0b0cbe223deeb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 3. 通过fontTools进行解析字库文件
    # 解析字体库
    font = TTFont('fonts.ttf')
    # 读取字体的映射关系
    uni_list = font['cmap'].tables[0].ttFont.getGlyphOrder() # 参数'cmap' 表示汉字对应的映射 为unicode编码
    print(uni_list)
    打印的结果为：['.notdef', 'uniECD5', 'uniEC83', 'uniED37', 'uniECE5', 'uniED98', 'uniEC58', 'uniEDFA', 'uniECB9', 'uniED6D', 'uniED1B', 'uniEDCE', 'uniED7D', 'uniEC3C', 'uniECEF', 'uniEC9E', 'uniED51', 'uniEE04', 'uniEDB3', 'uniEC72', 'uniEC20', 'uniECD4', 'uniED87', 'uniED35', 'uniEDE9', 'uniECA8', 'uniEC56', 'uniED0A', 'uniECB8', 'uniED6B', 'uniEC2B', 'uniEDCD', 'uniEC8C', 'uniED40', 'uniECEE', 'uniEDA1', 'uniED4F', 'uniEE03', 'uniECC2']
    需要注意的是：.notdef 并不是汉字的映射， 而是表示字体家族名称。真是数据是从下标 1 开始。
fontTools库详解： https://darknode.in/font/font-tools-guide/
##### 4. 将映射列表转换成utf-8的类型
    utf_list = [eval(r"u'\u" + x[3:] + "'") for x in uni_list[1:]]

#### 5. 通过软件查看字库的对应的映射关系  `font creator`
![查看字库中的映射关系](https://upload-images.jianshu.io/upload_images/11227136-5f460ce167cc7b71.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

    # 得到字符列表
    word_list = [u'一', u'七', u'三', u'上', u'下', u'不', u'九', u'了', u'二', u'五', u'低', u'八', u'六',
                 u'十', u'的', u'着', u'近', u'远', u'长', u'右', u'呢', u'和', u'四', u'地', u'坏', u'多',
                 u'大', u'好', u'小', u'少', u'短', u'矮', u'高', u'左', u'很', u'得', u'是', u'更']

###### 参考资料：

[知乎作者谢俊杰的文章](https://zhuanlan.zhihu.com/p/32087297)
[fontTools库详解](https://darknode.in/font/font-tools-guide/)
[github  fonttools库详解]( https://github.com/fonttools/fonttools)

# `总注：汽车之家的字库对应的字形顺序是不变的，只有映射的unicode的编码在改变， 用这种方法是可以处理的。 但应对unicode编码在变， 字形的顺序也在改变， 这种方法是无法解决的。具体的处理办法， 将在下一篇文中介绍。`

####  *本人水平有限， 如有错误欢迎提出指正！如有参考， 请注明出处！！禁止抄袭，遇抄必肛！！！*
源码:

    # coding:utf-8
    import re
    import requests
    from lxml import etree
    from fontTools.ttLib import TTFont

    headers = {
            "User-Agent": "Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/59.0.3071.86 "
                  "Safari/537.36 "
    }

    url = 'https://club.autohome.com.cn/bbs/thread/1d0784305887ec3f/72381110-1.html#pvareaid=102410'

    # 请求内容
    response = requests.get(url, headers=headers)
    response_html = response.content.decode('gbk')

    # xpath 获取帖子内容
    response_xml = etree.HTML(response_html)
    content_list = response_xml.xpath('//div[@xname="content"]//div[@class="tz-paragraph"]//text()')
    content_str = ''.join(content_list)
    print(content_str)

    # 获取字体的连接文件
    fonts_ = re.search(r",url\('(//.*\.ttf)?'\) format",response_html).group(1)

    # 请求字体文件， 字体文件是动态更新的
    fonts_url = 'https:'+fonts_
    response = requests.get(fonts_url, headers=headers).content
    # 讲字体文件保存到本地
    with open('fonts.ttf', 'wb') as f:
        f.write(response)

    # 解析字体库
    font = TTFont('fonts.ttf')

    # 读取字体的映射关系
    uni_list = font['cmap'].tables[0].ttFont.getGlyphOrder()

    # 转换格式
    utf_list = [eval(r"u'\u" + x[3:] + "'") for x in uni_list[1:]]

    # 被替换的字体的列表
    word_list = [u'一', u'七', u'三', u'上', u'下', u'不', u'九', u'了', u'二', u'五', u'低', u'八', u'六',
             u'十', u'的', u'着', u'近', u'远', u'长', u'右', u'呢', u'和', u'四', u'地', u'坏', u'多',
             u'大', u'好', u'小', u'少', u'短', u'矮', u'高', u'左', u'很', u'得', u'是', u'更']
    #遍历需要被替换的字符
    for i in range(len(utf_list)):
        content_str = content_str.replace(utf_list[i], word_list[i])

    print (content_str)








