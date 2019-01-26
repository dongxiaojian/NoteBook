# xpath 高级用法

##### 1. 匹配当前节点下的所有： `.//`

	.   表示当前
	//  表示当前标签下的所有标签
	注： 要配合使用

##### 2. 匹配某标签的属性值： `/@属性名称`
 	
		这里以input里的value值为例：
		例：xpath(//input/@value)

##### 3. 匹配多个路径：`|`
	
		在一个xpath中写的多个表达式用 | 分开， 每个表达式互不干扰。
		例：xpath("//tr[6]/td[2]/text() | //tr[7]/td[2]/text()")

##### 4.按属性匹配：`@`
	获取所有id="test"的所有文本内容
	xpath('//*[@id="test"]//text()')


##### 5. 匹配不包含某个属性的标签 `not`
		
		多用于表格中匹配中不包含表头信息的数据
		例：xpath('//table/tr[not(@class="tbhead")]')

##### 6. 匹配包含多个属性的标签: `and`
		匹配所有的tr中不包含 tbhead 属性 和包含 head 的tr标签
		xpath('//table/tr[not(@class="tbhead") and @class="head"]')


##### 7. 匹配包含不同属性的名称相同的标签: `or`
	匹配包含class="speedbar" 或者 class="content-wrap" 的标签
	例：xpath（'//div[@class="speedbar" or @class="content-wrap"]'）


##### 8. 将对象还原为字符串：`etree.tostring（）`

	将匹配到的对象，作为etree.tostring（）的参数即可，  注： 返回字符串
	 
	sObj = xml.xpath('//*[@id="test"]')[0] #使用xpath定位一个节点
	 sStr = etree.tostring(sObj)




#####9.按轴(Axes)匹配
###### 9.1 选取当前节点的所有子元素: `child`
	获取div下的tr的标签
	例：xpath('//div[@id="testid"]/child::tr/td/text()') # 感觉这种方法鸡肋， //div[@id="testid"]//tr/td 也可以实现

###### 9.2  选取当前节点的所有属性：`attribute`
	获取div标签所有的属性值
	例： xpath('//div/attribute::*') # 感觉这种方法鸡肋，//div/@* 同样能实现
 
###### 9.3  ancestor：父辈元素 / ancestor-or-self：父辈元素及当前元素
	获取父辈元素的div的所有属性值， 在不好定位的情况下，通过孩子标签定位，这种方法可以用
	xpath('//div[@id="test"]/ancestor::div/@*')
	xpath('//div[@id="test"]/ancestor-or-self::div/@*')

###### 9.4  descendant：后代 / descendant-or-self：后代及当前节点本身
	获取孩子元素的div的所有属性值，感觉鸡肋
	xpath('//div[@id="test"]/descendant::div/@*')
	xpath('//div[@id="test"]/descendant-or-self::div/@*')
###### 9.5 选取当前节点的所有命名空间节点:`namespace`

	xpath('//div[@id="test"]/namespace::*')

###### 9.6 定位:`position`
	和通过下标定位一样， 方法鸡肋
	xpath('//*[@id="test"]/ol/li[position()=2]/text()')




##### 10.Xpath 函数：
 ###### 10.1统计数量：`count` 
    统计符合要求节点的数量，  注： 返回字符串
    xpath('count(//tr[@info])')

###### 10.2字符串拼接 ：`concat`
    统计出来的两个内容的字符串进行“ + ”处理， 注： 返回字符串
    xpath('concat(//li[@id="one"]/text(),//li[@id="three"]/text())')
###### 10.3 解析当前节点下的字符：`string` 
    string()直解析匹配的第一个标签的值，  注： 返回字符串
    xpath('string(//tr)') 

###### 10.4 获取当前节点的节点名称: `local-name`
	返回当前属性的节点名称，  注： 返回字符串
	xpath('local-name(//*[@id="test"])')

###### 10.5 以指定的字符开头：`starts-with`
	starts-with定位属性值以8开头的li元素
	xpath('//tr[starts-with(@code,"one")]/text()')

###### 10.6 小于：`<`
	匹配所有tr标签属性info小于200的内容
	xpath('//tr[@info<200]/text()')

#####11. 根据指定的文本内容选择
    # 指定的文本内容可以是文本内容的部分， 也可以是全部
    //div[2]/ul/li[contains(text(), "指定的文本内容")]/span/text()




*注： 以上内容， 除标注外， 均返回列表！！*


**`总注： 在平常的xpath的使用中， 前8项的内容已基本够用， 使用时也应该使用简洁、易读、高效的匹配式。其余两项定位在较难的地方备选使用。切不可以为了追求高逼格， 刻意使用难懂难写低效的匹配式。人生苦短...``**





####  *本人水平有限， 如有错误欢迎提出指正！如有参考， 请注明出处！！禁止抄袭，遇抄必肛！！！*
