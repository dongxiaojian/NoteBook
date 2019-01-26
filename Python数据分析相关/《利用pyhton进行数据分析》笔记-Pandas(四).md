# Pandas(四)

`本次笔记主要时层次化索引以及整数索引的一些内容。内容比较琐碎。`



### 0. 层次化索引
层次化索引:在一个轴上拥有多个(两个以上)索引级别，能以低维度的形式处理高维度的数据。
###### 0.1 层次化索引初识

    ser = Series(np.random.randn(10), index=[list('aaabbbccdd'), [1,2,3,1,2,3,1,2,2,3]])
    ser.index
    # 返回值：
    MultiIndex(levels=[['a', 'b', 'c', 'd'], [1, 2, 3]],
           labels=[[0, 0, 0, 1, 1, 1, 2, 2, 3, 3], [0, 1, 2, 0, 1, 2, 0, 1, 1, 2]])
           
           
    dat = DataFrame(np.arange(12).reshape((4,3)), index=[['a','a','b','b'],[1,2,1,2]], columns=[['Ohio','Ohio','Colorado' ],['Green','Red','Green']])
    dat
    # 返回值
    # 		Ohio	Colorado
    #      Green	Red 	Green
    # a	1	0   	1   	2
    #   2	3   	4   	5
    # b	1	6   	7   	8
    #   2	9   	10  	11
    dat.index
    # 返回值
    # MultiIndex(levels=[['a', 'b'], [1, 2]],
    #            labels=[[0, 0, 1, 1], [0, 1, 0, 1]])
    dat.columns
    # 返回值
    # MultiIndex(levels=[['Colorado', 'Ohio'], ['Green', 'Red']],
    #            labels=[[1, 1, 0], [0, 1, 0]])
    
    
`MultiIndex`参数:	
`levels` : 每个级别的唯一标签。
`labels` : 每个级别的整数指定每个位置的标签。
`sortorder` :排序级别（必须按字典顺序按该级别排序）。
`names` :每个索引级别的名称。
`copy` : 复制元数据 默认为Flase。
`verify_integrity` : 检查级别/标签是否一致且有

###### 0.2 选取值：

    ser.iloc[5] # 返回值 0.6029141024292601
    
    ser[:,1] 
    # 返回值
    # a    0.11169
    # b    1.21725
    # c   -1.71559
    # dtype: float64
    
    ser.loc[('a'),(1)]# 返回值  0.11168972243989066
    
    ser.loc[('a','b'),(1)]
    # 返回值
    # a  1    0.11169
    # b  1    1.21725
    # dtype: float64
    
    dat.index.names = ['key1','key2']  # 为不同层的索引命名
    dat
    # 返回值
    		       Ohio	          Colorado
                   Green	Red 	Green
    key1	key2			
    a   	1   	0   	1   	2
            2   	3   	4   	5
    b   	1   	6   	7   	8
            2   	9   	10   	11
            
    dat.loc[('a','Colorado')]
    # 返回值
    #   	Green
    # key2	
    # 1	    2
    # 2     5
    
    
    dat['Colorado']
    # 返回值
    # 	                 Green
    # key1	key2	
    # a	      1	          2
    #         2	          5
    # b	      1	          8
    #         2	         11
    
    dat['Ohio']['Red']
    # 返回值
    # key1  key2
    # a     1        1
    #       2        4
    # b     1        7
    #       2       10
    # Name: Red, dtype: int32
    
    dat.loc['a'].loc[1]
    # 返回值
    # Ohio      Green    0
    #           Red      1
    # Colorado  Green    2
    # Name: 1, dtype: int32
    
    dat.loc[('a',1)]
    # 返回值
    # Ohio      Green    0
    #           Red      1
    # Colorado  Green    2
    # Name: (a, 1), dtype: int32
    
    
###### 0.3 重新分级排序
重新调整某条轴上的各级别的顺序，或者根据指定级别上的值对数据进行排序。

    dat.swaplevel('key1','key2')
    # 返回值
    # 		       Ohio     	Colorado
    #              Green	Red	 Green
    # key2	key1			
    #  1	a   	0   	1   	2
    #  2	a   	3   	4   	5
    #  1	b   	6   	7   	8
    #  2	b   	9   	10   	11
    
    dat.sort_index(1)
    # 返回值
    #    	        Colorad Ohio
    #               Green  	Green	Red
    # key1	key2			
    #   a	1   	2   	0   	1
    #       2    	5   	3   	4
    #   b	1   	8   	6   	7
    #       2   	11   	9   	10
    
    dat.swaplevel(0,1).sort_index(0)
    # 返回值
    # 		        Ohio	        Colorado
    #              Green	Red	    Green
    # key2	key1			
    #   1	a   	0   	1   	2
    #       b   	6   	7   	8
    #   2	a   	3   	4   	5
    #       b   	9   	10   	11
    


`swaplevel(i = -2，j = -1，axis= 0)`：在特定轴上的MultiIndex中交轴上的级别顺序。参数：	

  `i，j`：int，string（可以混合使用) 要交换的索引级别。可以将级别名称作为字符串传递。
`sort_index`: 按标签排序对象（沿轴)。参数：	
`axis` ：排序的轴。
`level`：int或级别名称或int列表或级别名称列表，如果不是None，则对指定索引级别的值进行排序。
`ascending` ：bool，默认为True 为升序，
`inplace`：bool，默认为False。 如果为True，则就地执行操作
`kind`：{'quicksort'，'mergesort'，'heapsort'}，默认'quicksort'，选择排序算法。mergesort是唯一稳定的算法。对于DataFrames，此选项仅在对单个列或标签进行排序时应用。
`na_position`：{'first'，'last'}，默认'last'首先将NaN 置于开头，最后将NaN放在最后。没有为MultiIndex实现。
`sort_remaining`：bool，默认为True。如果为true且按级别和索引排序是多级的，则在按指定级别排序后按其他级别（按顺序）排序,

### 0.4 根据级别汇总统计
许多对DataFarame和Series的描述和汇总统计都有一个level选项，它用于指定在某条轴上求和的级别。

    dat.sum(level='key2')
    # 返回值
    #   	Ohio	       Colorado
    #       Green	Red	   Green
    # key2			
    # 1     	6	8   	10
    # 2     	12	14  	16
    
    dat.columns.names=['state','color']
    dat.sum(level='state', axis=1)
    # 返回值
    # 	   state	Ohio	Colorado
    # key1	key2		
    # a  	1   	1   	2
    #       2   	7   	5
    # b  	1   	13  	8
    #       2   	19  	11
    
书中有说，这种方法其使用了pandas的groupby的功能，之后对其详解。

###### 0.5 使用DataFrame的列
这个简单的说， 就是让其中的列设置为行索引。

    dat2 = DataFrame([['一' , 'one' ,'z',  1],['二' , 'two' ,'x',  2],['一' , 'one' ,'z',      1],['二' , 'two' ,'x',  2]], index=['a', 'b', 'c', 'd'],
                    columns=[1,2,3,4])
    dat2
    
    dat2.set_index(2)
    # 返回值
    # 	1	3	4
    # 2			
    # one	一	z	1
    # two	二	x	2
    # one	一	z	1
    # two	二	x	2
    
    dat2.set_index(2).reset_index()
    # 返回值
    # 	2	1	3	4
    # 0	one	一	z	1
    # 1	two	二	x	2
    # 2	one	一	z	1
    # 3	two	二	x	2
    
    dat2.set_index([1,2])
    # 返回值
    # 		3	4
    # 1	2		
    # 一	one	z	1
    # 二	two	x	2
    # 一	one	z	1
    # 二	two	x	2


`set_index(keys, drop=True, append=False, inplace=False, verify_integrity=False)`
参数:
`keys`：列标签或列标签组成的数组列表
`drop`：布尔值，默认为True，删除要用作新索引的列
`append`：bool，默认为False，是否将列附加到现有索引
`inplace`：bool，默认为False，修改DataFrame（不要创建新对象），为True是
`verify_integrity`：bool，默认为False，检查新索引是否有重复项。否则，请在必要时推迟检查。设置为False将改善此方法的性能。


### 1. 整数索引
 先来看一下例子：
 
    # 当索引为整数时
    ser = Series(np.arange(3))
    ser
    # 返回值：
    # 0    0
    # 1    1
    # 2    2
    # dtype: int32
    ser[2] # 返回值 2
    ser[-1] # 直接抛出错误
    
    # 当所因为非整数时
    
    ser = Series(np.arange(3),index=['a','b','c'])
    ser[-1]
    # 返回值
    # 2
由此可见， 对于非整数索引时没有问题的。而对于第一种，使用的默认的整数索引。这样，通过整数索引取值的时候会产生歧义，无法分辨时整数索引还是位置下标。为了保持良好的一致性，如果时整数索引，根据整数取值的是时候总是面向标签而非下标的。

解决办法：（书中的iget_value已经不存在）

    ser = Series([1,3,2,4])
    ser
    ser.ix[:1]
    # 返回值
    # 0    1
    # 1    3
    # dtype: int64
    ser.ix[2:3]
    # 返回值
    # 2    2
    # 3    4
    # dtype: int64
    
    dat = DataFrame(np.arange(16).reshape(4,4))
    dat.iloc[-1]
    # 返回值
    # 0    12
    # 1    13
    # 2    14
    # 3    15
    # Name: 3, dtype: int32
    
    
    # dat.irow(0) # 这个方法也已经不存在了
    
注：如果没什么特殊情况， 还是不要用整数索引了， 看着脑壳疼。

#`总注： 本节的笔记比较琐碎，在实际中我遇到的也并不多，尤其时整数索引，如果不注意实际中还真会摸不着头脑了。希望多注意吧。`

####  *本人水平有限， 如有错误欢迎提出指正！如有参考， 请注明出处！！禁止抄袭！*
