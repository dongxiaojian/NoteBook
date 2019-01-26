# 字体文件反爬
` 在搞定静态字库反爬之后， 可以解决部分字体文件的反爬， 但动态字文件反爬是解决不掉的。此文章就是为解决动态字体文件的反反爬而写。本想以去哪儿网（手机端）的为例， 奈何手机端的字库反爬可能需要账号密码才会出现， 遂改用猫眼电影网的字体文件反爬为例。源码在最后！`

#### 1. 开发者模式查看网页内容
![开发者模式中的字体无法显示](https://upload-images.jianshu.io/upload_images/11227136-30e66a1545a41509.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



#### 2. 下载网页源码保存至本地查看
![网页源码下载到本地后查看](https://upload-images.jianshu.io/upload_images/11227136-303e85b8c040f179.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 3. 字体文件保存在本地通过 `font creator` 查看字体文件信息
![字体文件1](https://upload-images.jianshu.io/upload_images/11227136-d46b7330547eb2df.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![字体文件2](https://upload-images.jianshu.io/upload_images/11227136-c982b54ab7803839.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

`可以查看到字体文件的映射关系是动态的， 需要解决的就是通过某种方法来找到映射与数字之间的关系。`

#### 4.通过fontTools库， 将字库文件转换成xml格式


![映射关系与字体信息](https://upload-images.jianshu.io/upload_images/11227136-eb0de776f7f605b9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

    # 将字体文件转换成xml格式可以发现字体保存的图行信息是不变的， 变化的是unicode映射
    #以此为突破点， 找到图形对应的编码， 然后再匹配到对应的数字，以不变应万变。
    onlineFonts = TTFont('fonts.woff')
    onlineFonts.saveXML('test.xml')
    注： 此步骤在实际生产过程中并不需要， 为了便于理解， 我加上了 
#### 5.通过fontTools库， 获取字体信息中不变的部分
    baseFonts = TTFont('basefonts.woff') # basefonts.woff 这个文件是保存在本地的， 需要手动解析一个字体库， 作为不变的部分
    base_nums = ['7', '9', '0', '3', '6', '5', '2', '1', '4', '8'] # 基本的数字表
    base_fonts = ['uniF04C', 'uniE374', 'uniF426', 'uniEAAA', 'uniF519', 'uniEEC4', 'uniF543', 'uniF7C7', 'uniF046',
              'uniF08E'] # 基本的映射表 
    onlineFonts = TTFont('fonts.woff') # 网络上下载的动态的字体文件
    uni_list = onlineFonts.getGlyphNames()[1:-1] # 只有中间的部分是数字
    temp = {}
    # 解析字体库
    for i in range(10):
        onlineGlyph = onlineFonts['glyf'][uni_list[i]]  # 返回的是unicode对应信息的对象
        for j in range(10):
            baseGlyph = baseFonts['glyf'][base_fonts[j]]
            if onlineGlyph == baseGlyph:
                temp["&#x" + uni_list[i][3:].lower() + ';'] = base_nums[j]


#### 6.最终结果展示
![最终结果](https://upload-images.jianshu.io/upload_images/11227136-a0490f9c15253440.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

参考资料：
[知乎作者谢俊杰的文章](https://zhuanlan.zhihu.com/p/33112359)
[fontTools库详解](https://darknode.in/font/font-tools-guide/)
[github  fonttools库详解]( https://github.com/fonttools/fonttools)

####  *本人水平有限， 如有错误欢迎提出指正！如有参考， 请注明出处！！禁止抄袭，遇抄必肛！！！*
源码:

    # coding:utf-8

    import re
    import requests
    from fontTools.ttLib import TTFont
    from lxml import etree

    headers = {
                "User-Agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_13_4) AppleWebKit/537.36 (KHTML, like Gecko) "
                  "Chrome/66.0.3359.139 Safari/537.36 "
        }

    index_url = 'http://maoyan.com/'
    # 获取首页内容
    response_index = requests.get(index_url, headers=headers).text
    index_xml = etree.HTML(response_index)
    info_list = index_xml.xpath('//*[@id="app"]/div/div[1]/div[1]/div/div[2]/ul/li[1]/a/div[2]/div//text()')
    a = u'电影名称：%s, 票房总数：%s' % (info_list[1], info_list[4])
    print (a)

    # 获取字体文件的url
    woff_ = re.search(r"url\('(.*\.woff)'\)", response_index).group(1)
    woff_url = 'http:' + woff_
    response_woff = requests.get(woff_url, headers=headers).content

    with open('fonts.woff', 'wb') as f:
        f.write(response_woff)

    #base_nums， base_fonts 需要自己手动解析映射关系， 要和basefonts.woff一致
    baseFonts = TTFont('basefonts.woff')
    base_nums = ['7', '9', '0', '3', '6', '5', '2', '1', '4', '8']
    base_fonts = ['uniF04C', 'uniE374', 'uniF426', 'uniEAAA', 'uniF519', 'uniEEC4', 'uniF543', 'uniF7C7', 'uniF046',
                  'uniF08E']
    onlineFonts = TTFont('fonts.woff')

    # onlineFonts.saveXML('test.xml')

    uni_list = onlineFonts.getGlyphNames()[1:-1]
    temp = {}
    # 解析字体库
    for i in range(10):
        onlineGlyph = onlineFonts['glyf'][uni_list[i]]
        for j in range(10):
            baseGlyph = baseFonts['glyf'][base_fonts[j]]
            if onlineGlyph == baseGlyph:
                temp["&#x" + uni_list[i][3:].lower() + ';'] = base_nums[j]

    # 字符替换
    pat = '(' + '|'.join(temp.keys()) + ')'
    response_index = re.sub(pat, lambda x: temp[x.group()],     response_index)

    # 内容提取
    index_xml = etree.HTML(response_index)
    info_list = index_xml.xpath('//*[@id="app"]/div/div[1]/div[1]        /div/div[2]/ul/li[1]/a/div[2]/div//text()')
    a = u'电影名称：%s, 票房总数：%s' % (info_list[1], info_list[4])
    print(a)







    


