##Pandas笔记第二章： 

## `接上一章， 本章主是Pandas数据的增删改查，以及算术运算和函数的应用以及映射。`

    
  ### 1.索引、选取和过滤
    # Series
    obj = Series(np.arange(4), index=['a', 'b', 'c', 'd'])

    obj['b']  # 选取 索引b 值， 返回值为 1
    obj[1] # 根据下标获取值

    obj[:2] # 下标切片索引
    # 返回值： 
    # a    0
    # b    1
    # dtype: int64

    obj['a':'c'] # 标签切片
    # 返回值
    # a    0
    # b    1
    # c    2
    # dtype: int64

    obj[obj>1] # 跟据元素的值过滤
    # 返回值
    # c    2
    # d    3
    # dtype: int64

    obj[['a','b','c']] # 根据标签索引多个
    # 返回值
    # a    0
    # b    1
    # c    2
    # dtype: int64

    obj<3 # 每个元素的比较
    # 返回bool值：
    # a     True
    # b     True
    # c     True
    # d    False
    # dtype: bool

    # DataFrame

    data = DataFrame(np.arange(16).reshape((4,4)), index=['a', 'b', 'c', 'd'], columns=  ['one', 'two', 'three', 'four'])

    data['one']
    # 返回值
    # a     0
    # b     4
    # c     8
    # d    12
    # Name: one, dtype: int64


    data[['one','two']]
    # 返回值
    #   	one	two
    #   a	0	1
    #   b	4	5
    #   c	8	9
    #   d	12	13


    data[1:2] # 开区间，右边的数不包含
    # 返回值
    #    one	two	three	four
    #  b	4	5	6	7

    data['one':'two']   # 注意， 这种方法根本不可用
    # 返回值
    # 	one	two	three	four


    data[data['one']>4]
    # 返回值
    #   one	two	three	four
    #   c	8	9	10	11
    #   d	12	13	14	15


    data<8
    # 返回值：
    #   one	two	three	four
    # a	True	True	True	True
    # b	True	True	True	True
    # c	False	False	False	False
    # d	False	False	False	False


    data.iloc[1:4]
    # 返回值

    #   one	two	three	four
    # b	4	5	6	7
    # c	8	9	10	11
    # d	12	13	14	15

    data.loc['a':'c']
    # 返回值
    # 	    one	two	three	four
    # a	0	1	2	3
    # b	4	5	6	7
    # c	8	9	10	1

    data.loc ['a',['one', 'three' ]] #
    # 返回值：
    # one      0
    # three    2
    # Name: a, dtype: int32
       
    data.loc['a'].iloc[[2,3]]
    # 返回值
    # three    2
    # four     3
    # Name: a, dtype: int32

    data.ix[data.one > 3,:3]  
    # 返回值
    #   one	two	three
    # b	4	5	6
    # c	8	9	10
    # d	12	13	14

    data.loc[data.one > 3].iloc[:,:3] # 与上边等价
    # 返回值
    #   one	two	three
    # b	4	5	6
    # c	8	9	10
    # d	12	13	14

需要注意的有两点：
1. 利用下标索引与利用标签索引的区别：下标索引为开区间， 标签索引为闭区间。
2. ix方法已经不推荐始用，改用loc以及iloc。
  loc是根据定义的index标签索引。 iloc根据行号(下标)索引。需要注意的是， 这两种方法都是用来对行进行操作。

 ### 2. 算术运算和数据对齐
 pandas可以对不同索引的对象进行算术运算，如果存在不同的索引对，结果的索引就是该索引对的并集。
###### 2.1 Series

    s1 = Series([1,2,3,4,5], index=['a','b','c','d','e'])
    s2 = Series([6,5,4,3,2,1,],index=['a','d', 'e','g', 'n','m'])
    
    s1+s2
    # 返回值：
    a    7.0
    b    NaN
    c    NaN
    d    9.0
    e    9.0
    g    NaN
    m    NaN
    n    NaN
    dtype: float64
    

自动的数据对齐操作在不重叠的索引处引入NaN。缺失值会在算术运算过程中传播。简单的讲就是：两个对象相同的索引进行运算， 不同的索引置NAN，并取并集。

###### 2.2 DataFrame
DataFrame 数据对齐的操作， 会同时发生在行和列上， 即:行索引和列索引都行同时才进行运算，不同的置NAN,然后取并集。

    df1 = DataFrame(np.arange(9).reshape((3,3)),index=['a','b','c'], columns=['A','B','C'])
    df2 = DataFrame(np.arange(16).reshape((4,4)),index=['a','e','c','m'], columns=['A','E','C','M'])
    df1+df2  # 两值想乘同样如此
    # 返回值：
    	A	B	C	E	M
    a	0.0	NaN	4.0	NaN	NaN
    b	NaN	NaN	NaN	NaN	NaN
    c	14.0NaN	18.	NaN	NaN
    e	NaN	NaN	NaN	NaN	NaN
    m	NaN	NaN	NaN	NaN	NaN
    
    
###### 2.3 在算术方法中进行填充

    df1.add(df2,fill_value=0)  # 其他的运算方法： sub（-）  div（/）  mul（*）
    # 返回值：
    # 	A   	 B	     C  	E   	M
    # a	0.0	    1.0 	4.0	    1.0	   3.0
    # b	3.0	    4.0	    5.0 	NaN 	NaN
    # c	14.0	7.0 	18.0	9.0	   11.0
    # e	4.0 	NaN	    6.0	    5.0	    7.0
    # m	12.0	NaN 	14.0	13.0	15.0
    
    
通过这个例子可以看出：
1.将两个对象取并集， 行、列索引相同时，进行相应的运算;
2.其中的一个对象里边没有值时，以fill_value的值进行填充， 然后进行运算;
3.当两个对象中都没有值时，则置NAN;


###### 2.4 DataFrame 和 Series中之间的运算(要区别于矩阵运算)
    df3 =DataFrame(np.arange(16).reshape((4,4)),index=['a','e','c','m'], columns=['A','E','C','M'])
    # 值：
    #     A   E   C   M
    # a   0   1   2   3
    # e   4   5   6   7
    # c   8   9  10  11
    # m  12  13  14  15
    
    ser = df3.iloc[0]
    # 值
    # A    0
    # E    1
    # C    2
    # M    3
    
    df3 - ser
    #返回值
    # 	A	E	C	M
    # a	0	0	0	0
    # e	4	4	4	4
    # c	8	8	8	8
    # m	12	12	12	12
    
    
    ser2 = Series(range(3),index=['E','M','F'])
    print (ser2)
    # 返回值
    # E    0
    # M    1
    # F    2
    
    df3-ser2
    # 返回值
    #     A   C     E   F     M
    # a NaN NaN   1.0 NaN   2.0
    # e NaN NaN   5.0 NaN   6.0
    # c NaN NaN   9.0 NaN  10.0
    # m NaN NaN  13.0 NaN  14.0
    
    df3.sub(ser2, axis=0)
    # 返回值
    # A	E	C	M
    # E	NaN	NaN	NaN	NaN
    # F	NaN	NaN	NaN	NaN
    # M	NaN	NaN	NaN	NaN
    # a	NaN	NaN	NaN	NaN
    # c	NaN	NaN	NaN	NaN
    # e	NaN	NaN	NaN	NaN
    # m	NaN	NaN	NaN	NaN


通过以上的例子可以看出：
1.DataFrame和Series之间的算术运算，会将Series的索引匹配到DataFrame的列（Series本身就是一维数据）， 然后对DataFrame的每行数据进行运算。
2. 当其中一个对象的索引不存在时时，则置NAN。
3. 可以通过算术方法改变广播的轴。

值得注意的是：
1. 书中的例子 ix方法已经不推荐使用，需要用iloc 或者loc进行替代。
2. 算术方法中fill_value会报错。

### 3.函数的应用和映射
    df3
    # 返回值
    # 	A	E	C	M
    # a	0	1	2	3
    # e	4	5	6	7
    # c	8	9	10	11
    # m	12	13	14	15
    
    
    df3.apply(lambda x:x.max()-x.min()) # 对于轴级别， 默认为0轴
    # 返回值
    # A    12
    # E    12
    # C    12
    # M    12
    # dtype: int64
    
    df3.apply(lambda x:x.max()-x.min(), axis=1)  # 对于轴级别，指定为1轴
    # 返回值
    # a    3
    # e    3
    # c    3
    # m    3
    # dtype: int64
    
    df3.applymap(lambda x:x+1) # 元素级
    # 返回值
    # 	A	E	C	M
    # a	1	2	3	4
    # e	5	6	7	8
    # c	9	10	11	12
    # m	13	14	15	16
        
        
注意区别python中的map， Series中的map。

`总注：本章的笔记比较零碎。要对比于pyhton中的list，numpy中array进行学习效果更佳。再强调一点:pandas中， 已经不建议始用ix， 建议改成iloc以及loc的始用。`

####  *本人水平有限， 如有错误欢迎提出指正！如有参考， 请注明出处！！禁止抄袭！*






