# 文件读写
这一章主要讲了两个问题：0. 怎么读取写入  1.从哪里读取

### 0.读取，读取多用的[read_csv](http://pandas.pydata.org/pandas-docs/stable/generated/pandas.read_csv.html)
######  读取前言
解析函数：
`pd.read_csv` 从文件、url、文件型对象中加载带分隔符得数据。默认分隔符为逗号。（看到read_csv的参数,简直吓尿了）
`pd.read_table` 从文件、url、文件型对象中加载分隔符的数据， 默认分隔符为制表符。
`pd.ExcelFile` 从excel中读取数据。
`pd.read_fwf` 读取定宽列格式数据。
`pd.read_clipbpoard` 读取剪切板中的数据，将网页数据转换成表格时很有用。

###### 0.1 类型推断
类型推断： 即不需要指定列的类型到底时数值、整数、布尔值或者字符串。但是日期和其他自定义类型的处理需要多花点功夫。 
      
###### 0.2 索引 
默认索引: 当读取的文件没有列名时， 可以使用pandas分配默认的列名。
    
    原始数据为:
    with open('123.csv','r')as f:
        m  = f.read()
    print(m)
    # 0,0,1,2,3
    # 1,4,5,6,7
    # 2,8,9,10,11
    # 3,12,13,14,15
    
    # 默认索引
    pd.read_csv('123.csv', header=None)
    # 返回值：
    	0	1	2	3	4
    0	0	0	1	2	3
    1	1	4	5	6	7
    2	2	8	9	10	11
    3	3	12	13	14	15
    
自定义索引:手动指定索引， 并可以设置指定的行索引。

    # 自定义索引 header默认就是0
    pd.read_csv('123.csv', names=['a','b','c','d'], header=0)
    # 返回值
    	a	b	c	d
    1	4	5	6	7
    2	8	9	10	11
    3	12	13	14	15
    
    # 设定指定的行索引,index_col 书中的用法已经不适用， 需要传入int。
    c = pd.read_csv('123.csv', names=['a','b','c','d'], index_col=3)
    	a	b	c	d
    2	0	0	1	3
    6	1	4	5	7
    10	2	8	9	11
    14	3	12	13	15


层次化索引：指定多个索引，形成层次化索引。
    
    # index_col传入列表， 按顺序形成层次化索引。 
    pd.read_csv('234.csv', index_col=['key1','key2'])
    
                    key3	key4
    key1	key2		
    one	    a   	1   	2
            b   	2   	1
            c   	5   	6
    two	    a   	7   	8
            b   	9   	10
          

###### 0.3 数据转换
将指定的表示缺失数据的元素转换成NAN。

    # 原始数据
    A	B	C	D
    a	1.0	2.0	3.0	NaN
    b	1.0	NaN	2.0	3.0
    c	NaN	1.0	3.0	2.0
    d	2.0	3.0	NaN	1.0
    
    # 除了源数据中的Na值， 也会将na-values中的值置为NA
    pd.read_csv('134.csv',index_col=0, na_values=['1'])
    
    	A	B	C	D
    a	NaN	2.0	3.0	NaN
    b	NaN	NaN	2.0	3.0
    c	NaN	NaN	3.0	2.0
    d	2.0	3.0	NaN	NaN
    
    # 传入字典，可以指定某列的元素置为Na
    pd.read_csv('134.csv',na_values={'A':['1','3']})
    
            	Unnamed: 0	A	B	C	D
        0          	a   	NaN	2.0	3.0	NaN
        1         	b   	NaN	NaN	2.0	3.0
        2        	c   	NaN	1.0	3.0	2.0
        3        	d   	2.0	3.0	NaN	1.0
        
        
    # 注 如果index_col有值的情况下，传入字典会报错。

###### 0.4 日期解析
日期处理以后的文章统一处理，

###### 0.5 块读取(迭代)


    # chunksize参数将文件都读成迭代器，每次读取相应的值    
    chun = pd.read_csv('134.csv',chunksize=2)
    for info in chun:
        print (info)
        print ("*"*20)
        
    # 返回值
          Unnamed: 0    A    B    C    D
    0          a  1.0  2.0  3.0  NaN
    1          b  1.0  NaN  2.0  3.0
    ********************
      Unnamed: 0    A    B    C    D
    2          c  NaN  1.0  3.0  2.0
    3          d  2.0  3.0  NaN  1.0
    ********************




###### 0.6 不规整数据问题
手动处理分隔符格式、Json数据的处理以及HTML和XML处理数据的方式，作为爬虫来讲，用的比较多， 这里不过多展开， 简单说一下啊excel的文件的处理以及numbers文件的的处理。
处理Excel多用的是pd.read_excel， 当然也可以将excel保存为csv， 然后以read_csv的方式进行读取。 如果是mac系统的话， 可以将numbers文件保存为csv， 再用read_csv的方式进行读取。


### 1.写入
将文件写成csv，多用的是[to_csv](http://pandas.pydata.org/pandas-docs/stable/generated/pandas.DataFrame.to_csv.html).

    # 参数为默认
    dat = DataFrame(np.arange(16).reshape(4,4),index=list('abcd'), columns=list('ABCD'))
    dat.to_csv('test.csv')
    # 文件中的值为
    # ,A,B,C,D
    # a,0,1,2,3
    # b,4,5,6,7
    # c,8,9,10,11
    # d,12,13,14,15
    
    # 指定的分隔符
    dat.to_csv('test.csv', sep="|")
    # 文件中的值为
    # |A|B|C|D
    # a|0|1|2|3
    # b|4|5|6|7
    # c|8|9|10|11
    # d|12|13|14|15
    
    #禁用行、列标签
    dat.to_csv('test.csv', sep="|",index=False, header=False)
    # 文件中的值为
    # 0|1|2|3
    # 4|5|6|7
    # 8|9|10|11
    # 12|13|14|15
        
    # 指定要写的列
    
    dat.to_csv('test.csv',columns=['A','C'])
    # 文件中的值
    # ,A,C
    # a,0,2
    # b,4,6
    # c,8,10
    # d,12,14
    

#`总注：本章内容耽搁的时间比较久了。就其内容本身，本节内容主要是read_csv和to_csv的使用，这两个方法的参数很多，都弄明白是比较琐碎的事情，只能在以后的工作中，按需查找。其他不常用的理解一下就搞，等到用的时候，知道取哪里找才是关键。`
####  *本人水平有限， 如有错误欢迎提出指正！如有参考， 请注明出处！！禁止抄袭！*
