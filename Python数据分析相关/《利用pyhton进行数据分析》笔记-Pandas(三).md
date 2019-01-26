# Pandas笔记第三章
这一章`排序`以及`缺失值处理`比较重要。后边会加上 python中各种排序的对比。


### 1.排序和排名
其中的排序可对比于python中的sort。

排序： 1： 对索引进行排序 2： 对某个维度或者某几个维度的值进行排序。

排名：跟排序密切相关，且它会增设一个排名值（从1开始，一直到数组中有效数据的数量）。 它可以根据某种规则破坏平级关系。

###### 1.1 对轴进行排序

    ser = Series(range(5), index=['a','b','c','d','e'])
    ser.sort_index() 
    # 返回值：
    # a    0
    # b    1
    # c    2
    # d    3
    # e    4
    # dtype: int64
    
    
    daf = DataFrame(np.arange(16).reshape(4,4), index=['a','c','b','d'],
                   columns=list("ABDC"))
    
    daf.sort_index() # 默认对0轴进行排序
    # 返回值
    # 	A	B	D	C
    # a	0	1	2	3
    # b	8	9	10	11
    # c	4	5	6	7
    # d	12	13	14	15
    
    
    
    daf.sort_index(axis=1) # 对1轴进行排序
    
    # 返回值
    #  A	B	C	D
    # a	0	1	3	2
    # c	4	5	7	6
    # b	8	9	11	10
    # d	12	13	15	14
    
    
    daf.sort_index(axis=1).sort_index() # 同时对两个轴索引进行排序
    # 返回值
    # 	A	B	C	D
    # a	0	1	3	2
    # b	8	9	11	10
    # c	4	5	7	6
    # d	12	13	15	14  
    
以上的方法中， 都有ascending这个参数，Fales为逆序，默认为True，正序。
    
###### 1.2 对值进行排序 
注意： 书中的order方法已经弃用， 改用sort_values。

    ser2 = Series([1,2,3,4, np.nan], index=list('abcef'))
    ser2.sort_values()
    # 返回值
    # a    1.0
    # b    2.0
    # c    3.0
    # e    4.0
    # f    NaN
    # dtype: float64
    
    ser2.sort_values(ascending=False)
    # 返回值：
    # e    4.0
    # c    3.0
    # b    2.0
    # a    1.0
    # f    NaN
    # dtype: float64


    # daf.sort_index(by='B',ascending=False) # 这个方法还可以用， 但是会出现警告了， 不建议使用
    
    
    daf.sort_values(by='B',ascending=False) # 根据B列的值， 对行进行逆序
    # 返回值
    #   A	B	D	C
    # d	12	13	14	15
    # b	8	9	10	11
    # c	4	5	6	7
    # a	0	1	2	3
    
    daf.sort_values(axis=1,by='b',ascending=False) # 根据b行的值，对列进行逆序
    # 返回值
    #   C	D	B	A
    # a	3	2	1	0
    # c	7	6	5	4
    # b	11	10	9	8
    # d	15	14	13	12
    

在对值进行排序的时，

1.值为NaN时，不管正序逆序，其排序永远在最后。
2. Dafaframe的sort_values方法的by参数可以接受列表，对多个索引的值进行排序。
    
###### 1.3 排名
 排名： 跟排序关系密切， 且它会增设一个排名值(从1开始，一直到数组中有效数据的数据量)。它可以通过某种规则破坏平级(值相同即为平级)关系。

    ser = Series([7,2,1,3,3,9,8], index=['a','b','c','d','e','f','l'])
    
    ser.rank()
    # 返回值
    # a    5.0
    # b    2.0
    # c    1.0
    # d    3.5
    # e    3.5
    # f    7.0
    # l    6.0
    # dtype: float64
    
    ser.rank(method='max')
    # 返回值
    # a    5.0
    # b    2.0
    # c    1.0
    # d    4.0
    # e    4.0
    # f    7.0
    # l    6.0
    # dtype: float64
    
    ser.rank(method='min')
    # 返回值
    # a    5.0
    # b    2.0
    # c    1.0
    # d    3.0
    # e    3.0
    # f    7.0
    # l    6.0
    # dtype: float64
    
    ser.rank(method='first')
    # 返回值
    # a    5.0
    # b    2.0
    # c    1.0
    # d    3.0
    # e    4.0
    # f    7.0
    # l    6.0
    # dtype: float64
    
    daf = DataFrame(np.arange(12).reshape(3,4), index=['a', 'b','c'],columns=['A','B','C','D']
        )
    
    daf.rank(method="max")

    # 返回值
    # 	A	B	C	D
    # a	1.0	1.0	1.0	1.0
    # b	2.0	2.0	2.0	2.0
    # c	3.0	3.0	3.0	3.0

    
    
排名时用于破会平级关系的method选项， 有四个方式：
1. average  默认方式， 在值相等的情况下，为各个值分配平均排名。
2. min      使用整个分组中的最小排名。
3. max      使用整个分组中的最大排名。
4. first    按值在原始数据中的出现顺序分配排名。


### 2.带有重复值的轴索引
Panda的很多函数都要求标签唯一， 但这并不是强制性的。索引值可以重复。

    ser = Series(range(4),index=['a','a','b', 'c'])
    ser['a']
    # 返回值
    # a    0
    # a    1
    # dtype: int64
    ser.index.is_unique # 判断索引值是否唯一
    # 返回值
    # False
    
    daf = DataFrame(np.arange(12).reshape(3,4), index=['a', 'a','c'],columns=['A','A','C','D']
            )
    
    daf['A']
    # 返回值
    # 	A	A
    # a	0	1
    # a	4	5
    # c	8	9
    
    daf.loc['a']
    # 返回值
    # 	A	A	C	D
    # a	0	1	2	3
    # a	4	5	6	7
    
    
### 3.处理缺失数据
缺失数据， 在大部分的数据来源中都很常见。pandas使用浮点值NaN(Not a Number)来表示浮点和非浮点数中的缺失值。处理缺失值的方式一般有两种： 滤除和填充。

###### 3.1 滤除缺失值

    from numpy import nan
    ser = Series([7,2,nan,3,3,9,8], index=['a','b','c','d','e','f','l'])
    # 通过方法直接删除
    ser.dropna()
    # 返回值
    # a    7.0
    # b    2.0
    # d    3.0
    # e    3.0
    # f    9.0
    # l    8.0
    # dtype: float64
    
    # 通过布尔型索引截取
    ser[ser.notnull()]
    # 返回值
    # a    7.0
    # b    2.0
    # d    3.0
    # e    3.0
    # f    9.0
    # l    8.0
    # dtype: float64
    
    
    daf = DataFrame([[1,2,2,3],[2,nan,nan,1],[nan, nan,nan,nan]], index=['a', 'a','c'],columns=['A','A','C','D']     )
    
    daf.dropna()
    # 返回值
    # 	A	A	C	D
    # a	1.0	2.0	2.0	3.0
    daf.dropna(how='all')
    # 返回值
    # 	A	A	C	D
    # a	1.0	2.0	2.0	3.0
    # a	2.0	NaN	NaN	1.0
    
    daf.dropna(thresh=1)
    # 返回值
    # 	A	A	C	D
    # a	1.0	2.0	2.0	3.0
    # a	2.0	NaN	NaN	1.0
    
滤除NaN用到的主要的方法是`dropna(axis=0, how='any', thresh=None)` ,`axis` 选择轴； `how`默认是any， 即只要含有NaN的行都被删除， all表示值都为NaN的时候才被删除；`thresh `参数接收整数， 表示一行中至少有 thresh 个非 NA 值时将其保留。


###### 3.2 填充缺失值

在实际应用中，由于获取的数据很重要，在少量的数据不全时， 多采用填充缺失值。

    daf = DataFrame([[1,2,2,3],[2,nan,nan,1],[nan, nan,nan,nan]], index=['a',     'b','c'],columns=['A','B','C','D'] )
    
    daf.fillna(9)
    # 返回值；
    # A	A	C	D
    # a	1.0	2.0	2.0	3.0
    # a	2.0	9.0	9.0	1.0
    # c	9.0	9.0	9.0	9.0
    
    
    daf.fillna({'A': 4.0, 'C': -4.0})
    # 返回值
    #  	A	B	C	D
    # a	1.0	2.0	2.0	3.0
    # b	2.0	NaN	-4.0	1.0
    # c	4.0	NaN	-4.0	NaN
    
    daf.fillna(daf.mean()) # 填充平均值
    
fillna 中的参数：
`value`： 用于填充缺失值的标量值，字典， Series或DataFrame
`method`： {'backfill'，'bfill'，'pad'，'ffill'，None}，默认None
用于填充重新索引的填充孔的方法系列填充/填充：将最后有效观察向前传播到下一个有效回填/填充：使用NEXT有效观察填补空白
`axis `： 轴 默认 0
`inplace `bool值， 默认为Flase。返回新的数据。如果为True，直接在源数据上修改。
`limit`：int，默认None,如果指定了method，则这是向前/向后填充的连续NaN值的最大数量。换句话说，如果存在超过此数量的连续NaN的间隙，则仅部分填充。如果未指定method，则这是沿整个轴填充NaN的最大条目数。如果不是None，则必须大于0。




# *本人水平有限， 如有错误欢迎提出指正！如有参考， 请注明出处！！禁止抄袭*  


## 附 排序对比

    import numpy as np
    from pandas import DataFrame
    
    temp_list = [1,3,2,4,6]
    arr = np.array(temp_list)
    dat = DataFrame(temp_list)
    
    sorted(temp_list) # 返回值  [1, 2, 3, 4, 6]
    sorted(arr) # 返回的是list  [1, 2, 3, 4, 6]
    sorted(dat) # 返回值 [0]
    
    temp_list.sort()# 无返回值， 直接在原始数据上修改
    arr.argsort() # 返回排序的索引
    arr.sort() # 无返回值，数据直接修改
    dat.sort_values(by=0)
    # 返回值
    # 0
    # 0	1
    # 2	2
    # 1	3
    # 3	4
    # 4	6
    
    temp_list2 = [1,3,2,4,6,'7', '9','0']
    arr2 = np.array(temp_list2) # 返回值 array(['1', '3', '2', '4', '6', '7', '9', '0'],     dtype='<U11')
    dat2 = DataFrame(temp_list2) 
    
    # sorted(temp_list2) # 报错
    # temp_list2.sort()# 报错 
    
    sorted(temp_list2, key=str) # 返回值 ['0', 1, 2, 3, 4, 6, '7', '9']
    sorted(arr2) # 返回值  ['0', '1', '2', '3', '4', '6', '7', '9']
    sorted(dat2) # 返回值[0]
    
    temp_list2.sort(key=str) # temp_list2 返回值 ['0', 1, 2, 3, 4, 6, '7', '9']
    arr2.argsort()  # 返回值： array([7, 0, 2, 1, 3, 4, 5, 6], dtype=int64)
    arr2.sort() # 无返回值，数据直接修改
    dat2.sort_values(by=0) # 报错


值得注意的是：
1.sorted 返回值都是list，且不能用于DataFrame。
2.sort  无返回值，都是就地修改数据。
3.当list中的数据类型不一样时，sorted和sort需要传入key参数。(python2中不存在这个问题。)
4.array的argsort方法，有返回值， 是返回排好序的索引。
5.DataFrame的sort_values无法排序不同的数据类型。



