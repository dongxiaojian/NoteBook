  `Pandas是这本书的重点，以后的操作基本上都是在Pandas的基础上完成的。就用法上，Pandas要比Numpy更加的强大、灵活和简单。Pandas一些东西我也是初学，还没有形成一个系统，这次记录可能会有知识上的漏洞，待有了一个整体的系统的认识，时间充足的话可能会对这部分进行重构......`
######按照惯例，导入pandas的方式如下，在py文件中看到pd一般就是pandas：

    import pandas as pd
    from pandas import DataFrame, Series

### 1. pandas的数据结构：`Series`和`DataFrame`
 `Series：`一种类似于一维数组的对象。只有行索引， 没有列索引(相比于DataFrame)。
 `DataFrame：` 是一个表格型数据结构，既有行索引也有列索引(可以想象一下execl)，其面向行和面向列的操作基本上是平衡的。
#### 1.1 Series

    # Series
    temp_obj = Series([1,2,3,4])
    temp_obj
    # 返回值：
    # 0    1
    # 1    2
    # 2    3
    # 3    4
    # dtype: int64

    temp_obj1 = Series([[1,2,3,4],[3,4,5,6]])
    temp_obj1 # 传入的参数是二维的，但Series将对应的数组当成一个值来处理了， 所以说Series是类似于一维数组的对象
    # 返回值：
    # 0    [1, 2, 3, 4]
    # 1    [3, 4, 5, 6]
    # dtype: object

    temp_obj2 = Series([[[1,2],[3,4],[4,5],[6,7]],[[7,8],[8,9],[10,11],[11,12]]])
    temp_obj2 # 三维也是如此，Series是类似于一维数组的对象
    # 返回值：
    # 0        [[1, 2], [3, 4], [4, 5], [6, 7]]
    # 1       [[7, 8], [8, 9], [10, 11], [11, 12]]
    # dtype: object

    # 也可以通过字典来创建Series
    temp_dict = {
        'a':100,
        'b':200,
        'c':300,
    }
    temp_obj4 = Series(temp_dict)
    temp_obj4
    # 返回值：
    # a    100
    # b    200
    # c    300
    # dtype: int64

    temp_obj1 = Series([[1,2,3,4],[3,4,5,6]],index=['哈','哼'])
    temp_obj1
    # 返回值：
    # 哈    [1, 2, 3, 4]
    # 哼    [3, 4, 5, 6]
    # dtype: object

    # 可以通过索引来选取对应的行
    temp_obj1["哈"] # 返回值：[1, 2, 3, 4]
    temp_obj1[["哼",'哈']]
    # 返回值：
    # 哼    [3, 4, 5, 6]
    # 哈    [1, 2, 3, 4]
    # dtype: object
    # temp_obj1[temp_obj1>3] # 会报错，原因是这样比较是每行，而不是里边的元素， TypeError: '>' not supported between instances of 'list' and 'int'

    temp_obj1.index # 返回值是index对象， Index(['哈', '哼'], dtype='object')
    temp_obj1.values # 返回值：array([list([1, 2, 3, 4]), list([3, 4, 5, 6])], dtype=object)

    temp_obj1*2 # 直接乘以2，只是把里边的数数据复制1份,相当于每行的list*2，而不是广播到每个元素


    '哈' in temp_obj1 # 判断索引是否存在于temp_obj1，返回值为bool类型

    # 索引可以通过直接赋值的方式直接修改 
    temp_obj1.index = ['a','b'] 
    temp_obj1
    #返回值：
    # a    [1, 2, 3, 4]
    # b    [3, 4, 5, 6]
    # dtype: object

    # Series对象本身的name属性， 以及索引的name的属性。
    temp_obj1.name = '董小贱'
    temp_obj1.index.name = 'dongxiaojian'
    temp_obj1
    # 返回值：
    # dongxiaojian
    # a    [1, 2, 3, 4]
    # b    [3, 4, 5, 6]
    # Name: 董小贱, dtype: object

#### 1.2 DataFrame   
`可以对比于Series，有很多方法都是相似的。不同的地方是: DataFrame即有行索引，也有列索引(可对比于execl表格)。也就是说DataFrame中的数据是以一个或者多个二维块存放的。`

    import pandas as pd
    from pandas import DataFrame
    data = {'科目':['数学','语文','英语'],'分数':[100,90,80]}
    temp_frame = DataFrame(data)
    temp_frame #可以发现返回值中自动添加了行索引
    # 返回值
    #       分数	科目
    #0	100	数学
    #1	90	语文
    #2	80	英语


    data = {'科目':['数学','语文','英语'],'分数':[100,90,80]}
    temp_frame = DataFrame(data,index=['a','b','c',], columns=['科目','分数','名次'])# index 指定行索引，columns指定列索引，可以添加列， 如果没有对应的值则直接用NaN补全
    temp_frame

    # 返回值
    #    	    科目	分数	名次
    #   a	    数学	100	NaN
    #   b	    语文	90	NaN
    #   c	    英语	80	NaN

   
 
######  1.2.1   查
    #通过索引获取对应的行或者列
    #通过索引获取列
    temp_frame.科目 # 返回值的类型：Series
    # 返回值：
    # a    数学
    # b    语文
    # c    英语
    # Name: 科目, dtype: object

    # 根据列索引获取列详细信息，与以上等价
    temp_frame['分数']
    temp_frame.get('名次')


    # 通过索引获取行
    temp_frame.ix['a']
    # 返回值：

    # 科目     数学
    # 分数    100
    # 名次    NaN
    # Name: a, dtype: object

    # 对列进行赋值：直接对源数据进行修改


######  1.2.2   改

    # 传单独的一个值， 会对整列的每个元素进行修改
    temp_frame.名次 = 30

    temp_frame
    # 返回值：
    # 	科目	分数	名次
    # a	数学	100	30
    # b	语文	90	30
    # c	英语	80	30

    # 传入的可迭代对象元素个数要和数整列数据的元素个数一一对应
    temp_frame.名次 = [1,2,3]
    temp_frame

    #返回值
    # 科目	分数	名次
    # a	数学	100	1
    # b	语文	90	2
    # c	英语	80	3



    temp_frame.名次['a'] = 10
    temp_frame


######  1.2.3   增


    # 添加新的行
    insert_row = pd.DataFrame([['理综','100','15']],index=['d'], columns=['科目','    分数','名次'])
    temp_frame.append(insert_row,)
    temp_frame
    temp_frame2 = pd.concat([temp_frame,insert_row])
    temp_frame

    # 添加新的列
    temp_frame.insert(0,'班级',10)
    temp_frame


######  1.2.4   删

    # 删除行
    temp_frame2.drop(['d'])  # 会产生新的DataFrame

    # 删除列
    temp_frame2.pop('科目') # 返回值为 pop出的数据， 原数据也会修改
    temp_frame2

    temp_frame2.drop(['班级'],axis=1) # 会产生新的DataFrame

### 2.索引对象
  `索引对象负责管理series或者DataFrame数据的轴标签和其他原数据。构建数据是， 所用到的任何数据或者其他序列的标签都会被转换成一个Index。Index是不可修改的，这样才能保证index对象在多个数据结构之间安全共享。虽不可修改，但可以重置！`

    import pandas as pd
    from pandas import Series, DataFrame

    obj = Series(range(3), index = ['a','b','c'])
    index = obj.index
    index
    # 返回值：Index(['a', 'b', 'c'], dtype='object')

    # 切片
    index[:2] #返回值：Index(['a', 'b'], dtype='object')

    # index[0] = 'd'  会报错， 不可修改

    frame = DataFrame(np.arange(9).reshape((3,3)),index=['a','b','c'],columns = ['one','two','three'])

    frame.index # 返回值 Index(['a', 'b', 'c'], dtype='object')    返回的是行索引



###### 2.1 重新索引(对索引重新排序)
    # Series
    obj 
    # 返回值：
    # a    0
    # b    1
    # c    2
    # dtype: int64

    obj.reindex(['d','c','f'])
    # 返回值
    # d    1
    # c    0
    # f    2
    # dtype: int64


    obj.reindex(['d','c','f','m'],fill_value=10) # 添加新的索引，默认值维NAN， 可以通过fill_value进行赋值
    # 返回值：
    # d     1
    # c     0
    # f     2
    # m    10
    # dtype: int64

    # 插值处理

    obj.reindex(['d','c','f','m','n'],method='ffill') # 前向填充， 根据前一个值填充后一个值
    # 返回值：
    # d    1
    # c    0
    # f    2
    # m    2
    # n    2
    # dtype: int64

    obj.reindex(['d','a','n','c','f',],method='bfill')  # 后向填充， 根据后一个值填前一个值
    # 返回值
    # d    1.0
    # a    0.0
    # n    NaN
    # c    0.0
    # f    2.0
    # dtype: float64

    #  DataFrame
    frame.reindex(columns=['two','one','three'])
    # 返回值：
    #    two one three
    # 0  1    0    2
    # 1  4    3    5
    # 2  7    6    8

    # 同时对行 列同时进行重新索引
 
    frame.reindex(index=[1,2,0],columns=['two','one','three']) # 
    # 返回值
    #   two	one	three
    # 1	4	3	5
    # 2	7	6	8
    # 0	1	0	2

    frame.ix[[1,2,0], ['two','one','three']] #与以上等价

###### 2.2 问题点
`前面已经说了，Index是不可修改的，但是看以下的例子你会发现，可以重新赋值，不太理解这种操作，欢迎读者老爷留言交流。`

    # Series
    obj 
    # 返回值：
    # a    0
    # b    1
    # c    2
    # dtype: int64

    obj.index = ['c','d','f']
    obj

    # 返回值
    # c    0
    # d    1
    # f    2
    # dtype: int64

    # Dataframe
    frame.index # 返回值 Index(['a', 'b', 'c'], dtype='object')     返回的是行索引
    frame.index = [0,1,2]
    frame
    # 返回值
    #       one	two	three
    # 0	0	1	2
    # 1	3	4	5
    # 2	6	7	8

#  `注：中间有点琐碎的事，导致这节的内容耽搁的比较久了。本节内容比较琐碎，平时用的过程中也都容易忽略。其中关于查的部分，下节单独拿出来做一下笔记，应该是用的比较多。欢迎读者老爷们指正。`
####  *本人水平有限， 如有错误欢迎提出指正！如有参考， 请注明出处！！禁止原文抄袭，遇抄必肛！！！*



