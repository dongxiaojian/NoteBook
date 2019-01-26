###### 用python进行数据分析， 一般都离不开两个库：`Numpy` 和 `pandas`。这本书之前刷过一半， 太久没看， 细节忘却，这次重拾， 顺便记录之。
######  前言
1. `NumPy`本身并没有提供那么多高级的数据分析的功能，但理解Numpy数数组结构，能更好的理解以及以及运用Pandas，所以， 还是从Numpy开始搞起，等学起pandas还是能省不少力的。
2. `ndarray`:Numpy中的多维数组队对象(可类比与矩阵)，用numpy操作数据的时候， 基本上也是操作ndarry。其中的运算也跟高数中的矩阵非常像，ndarry是一个通用的同构数据多维容器， 其中的`所有的元素必须是相同的类型，所有的元素必须是相同的类型，所有的元素必须是相同的类型`(`重说三`)。
3. 惯例中引入numpy的方法： `import numpy as np`  ，建议大家养成这个习惯， 因为基本都是这么做的。
4. 先来明确一下`ndarray` 、`array`和`矩阵`：`首先ndarray即数组。np.array只是numpy一个便捷的函数，用来创建一个ndarray。矩阵是线性代数中的概念， 其中的元素必须是数字，而ndarray中的的元素可以为字符串。`
5. 以下的操作均在 `jupyter`中进行，以后的文章也是。

#### 1.创建一个ndarry`np.aarray`

    import numpy as np 
    # 创建ndarray
    # 1.list转换
    temp_list = [1,2,3]
    np.array(temp_list)  # 返回一维  array([1, 2, 3])

    temp_list2 = [[1,2,3],[4,5,6]]
    np.array(temp_list2) # 返回ndarray  array([[1, 2, 3], [4, 5, 6]])


    np.array2string(np.array(temp_list)) # 转换成字符串  返回值  '[1 2 3]'

    #2.直接生成
    # 创建全0的ndarray
    np.zeros(10) # 返回值   array([0., 0., 0., 0., 0., 0., 0., 0., 0., 0.])

    np.zeros((3,6)) # 接收的(3,6)为array的形状   返回值  array([[0., 0., 0., 0., 0., 0.],
    #                                                       [0., 0., 0., 0., 0., 0.],
    #                                                       [0., 0., 0., 0., 0., 0.]])


    np.zeros_like(np.array([1,2,3]))  # 返回形状相同的全0数组，array([0, 0, 0])


    # 创建随机的ndarray， array数据是一些未初始化的垃圾值。
    np.empty((2,1)) # 返回值  array([[ 1.49166815e-154], [-2.00390439e+000]])
      
    # 类似range， 返回的是
    np.arange(5) # 返回值 array([0, 1, 2, 3, 4])


    # 创建全1的ndarray
    np.ones(3) #返回值  array([1., 1., 1.])
    np.ones((2,3))  # 接收的(2，3)为array的形状，返回值  array([[1., 1., 1.], [1., 1., 1.]])

    np.ones_like(np.array([1,2,3]))  # 返回形状相同的全1数组，array([1, 1, 1])


    # 创建N*N的ndarray
    np.eye(3)  # 创建3*3正方形矩阵，对角线为1，其余为0.的array。   array([[1., 0., 0.],
    #                                                             [0., 1., 0.],
    #                                                                [0., 0., 1.]])
 
    np.identity(3)  # 同 eye.  array([[1., 0., 0.],
    #                                 [0., 1., 0.],
    #                                 [0., 0., 1.]])

     # 从字符串中获取
    temp_str = '1,2,3,4'
    np.fromstring(temp_str,sep=',')  # 返回值：array([1., 2., 3., 4.])


#### 2.ndarray的结构`轴(axes)`和`秩(rank)`

`轴`：每一个线性的数组称为是一个轴，也就是维度（dimensions)。
`秩`：维数，一维数组的秩为1，二维数组的秩为2，以此类推。即轴的个数。

    temp_array = np.eye(5) # 创建一个5*5的数组
    temp_array.shape  # 显示数组在每个维度上的大小，返回值一个元组(5,5)  即5行5列的2维数组。
    temp_array.ndim #显示数组的秩， 返回值为2， 即  len(temp_array.shape)

    # 三维数组
    temp_list3 = [[[1,2,3],[3,4,5]],
                   [[5,6,7], [7,8,9]]]
                    
    temp_array3 = np.array(temp_list3)

    temp_array3.shape  # 显示数组在每个维度上的大小，返回值是 (2, 2, 3)，即结构为2*2*3的3维数组
    temp_array3.ndim # 返回3， 即秩为3
    temp_array3[1]    # 返回下标为1的元素， 即[[5 6 7]， [7 8 9]]


 #### 3.ndarry的数据类型`dtype`
 `dtype(数据类型)`是一种特殊的对象， 它含有ndarry将一块内存解释为特定的数据类型所需的信息。dtype是Numpy如此强大和灵活的原因之一。
`来自图书作者的提示:记不住这些NumPy的dtype也没关系。 通常只需知道你所处理的数据的大致类型是：浮点数、复数、整数、布尔值、字符串，还是普通的Python对象即可。当你需要控制数据在内存和磁盘中的存储方式时(尤其是对大数据集)，那就得了解如何控制存储的类型。`
    
    #np.array默认的类型
    temp_list = ['1','2','3']
    temp_array = np.array(temp_list)
    temp_array.dtype #dtype('<U1')

    temp_list2 = [1,2,3,4]
    temp_array2 = np.array(temp_list2)
    temp_array2.dtype #dtype('int64')

    temp_list3 = [1.,2.,3.]
    temp_array3 = np.array(temp_list3)
    temp_array3.dtype #dtype('float64')

    temp_list4 = [True, False]
    temp_array4 = np.array(temp_list4)
    temp_array4.dtype # dtype('bool')

    temp_list5= ['董小贱','小菜鸡']
    temp_array5 = np.array(temp_list5)
    temp_array5.dtype # dtype('<U3')

    temp_list6 = ['a','b','c']
    temp_list6 = np.array(temp_list6)
    temp_list6.dtype # dtype('<U1')

    temp_list7 = [-1,0,-3]
    temp_list7 = np.array(temp_list7)
    temp_list7.dtype # dtype('int64')
    
    #创建指定类型的ndarray
    temp_list8 = [1,2,3,4]
    temp8 = np.array(temp_list8) # 默认类型
    temp8.dtype #dtype('int64')  temp8的值为 array([1, 2, 3, 4])


    temp_array8 = np.array(temp_list8, dtype=np.int32)
    temp_array8.dtype #dtype('int32') temp_array8的值为 array([1, 2, 3, 4], dtype=int32)

    temp_array9 = np.array(temp_list8, dtype=np.float32)
    temp_array9.dtype #dtype('int32')   temp_array9的值为：array([1., 2., 3., 4.], dtype=float32)


    temp_array10 = np.array(temp_list8, dtype=np.uint8)
    temp_array10.dtype #dtype('uint8')  temp_array10 的值为    array([1, 2, 3, 4], dtype=uint8)



###### numpy中的数据类型(也可参考《利用python进行数据分析》)
![数据类型.png](https://upload-images.jianshu.io/upload_images/11227136-5e017c372f7a947a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


###### numpy中的类型转换`astype`

    # 数据类型转换
    temp_list11 = [1,2,3,4]
    temp_array11 = np.array(temp_list11)
    temp_array11.dtype #dtype('int64')

    temp_array12 = temp_array11.astype(np.float64) #这里是返回一个新的数组， 而不是改变旧的数组 array([1., 2., 3., 4.])
    temp_array11.dtype #dtype('int64')
    temp_array12.dtype #dtype('float64')

    temp_list13 = [1.1,2.2,3.1]
    temp_list13 = np.array(temp_list13)
    temp_list13.astype(int) #  array([1, 2, 3])   dtype类型为 dtype('int64')
     注：如果astype的参数写int，float， 将会使用默认的int64， 以及float64类型， 建议使用np.int64这种方式， 一目了然。

    # 另类操作，直接给数组的dtype赋值，原始数据直接改变， 一般不会使用这种方式，改变dtype类型，还是使用astype
    temp_list13 = [1.1,2.2,3.1]
    temp_list13 = np.array(temp_list13)
    temp_list13 #array([1.1, 2.2, 3.1])
    temp_list13.dtype='int64'
    temp_list13 # array([4607632778762754458, 4612136378390124954, 4614162998222441677])

    temp_list13.dtype = 'int32'
    temp_list13 # array([-1717986918,  1072798105, -1717986918,  1073846681,  -858993459,1074318540], dtype=int32)
        



 #4.数组标量之间的运算(可类比于矩阵的运算法则，请注意区别)

 `标量`：只有大小，没有方向，即一个数字。
`矢量`：有大小，有方向，比如数组。
数组可以不用编写循环即可对数据进行批量运算，这通常叫作`矢量化`。大小相等的数组之间的任何算数运算都将会应用的元素级。


    temp_list = [1,2,3,4,5,6]
    temp_array = np.array(temp_list)

    temp_list2 = [6,5,4,3,2,1]
    temp_array2 = np.array(temp_list2)

    temp_array * temp_array2 #返回值： array([ 6, 10, 12, 12, 10,  6])

    temp_list2 + temp_array #返回值 array([7, 7, 7, 7, 7, 7])

    temp_list2 - temp_array #array([ 5,  3,  1, -1, -3, -5])

    temp_list2 / temp_array #array([6.        , 2.5       , 1.33333333, 0.75      , 0.4       ,
    #                               0.16666667])



    #数组*标量  
    temp_array2 * 3 # 返回值 array([18, 15, 12,  9,  6,  3])



    # 多维数组
    temp_array3 = np.eye(6)


    temp_array3*3

    # array([[3., 0., 0., 0., 0., 0.],
    #        [0., 3., 0., 0., 0., 0.],
    #        [0., 0., 3., 0., 0., 0.],
    #        [0., 0., 0., 3., 0., 0.],
    #        [0., 0., 0., 0., 3., 0.],
    #        [0., 0., 0., 0., 0., 3.]])

    temp_array * temp_array3
    # array([[1., 0., 0., 0., 0., 0.],
    #        [0., 2., 0., 0., 0., 0.],
    #        [0., 0., 3., 0., 0., 0.],
    #        [0., 0., 0., 4., 0., 0.],
    #        [0., 0., 0., 0., 5., 0.],
    #        [0., 0., 0., 0., 0., 6.]])

    temp_array3 * temp_array  
    # array([[1., 0., 0., 0., 0., 0.],
    #        [0., 2., 0., 0., 0., 0.],
    #        [0., 0., 3., 0., 0., 0.],
    #        [0., 0., 0., 4., 0., 0.],
    #        [0., 0., 0., 0., 5., 0.],
    #        [0., 0., 0., 0., 0., 6.]])


    temp_array3 / temp_array
    # array([[1., 0., 0. , 0., 0. ,0.],
    #         
    #        [0.        , 0.5       , 0.        , 0.        , 0.        , 0.        ],      
    #        [0.        , 0.        , 0.33333333, 0.        , 0.        , 0.        ],
    #        [0.        , 0.        , 0.        , 0.25      , 0.        ,  0.       ],
    #        [0.        , 0.        , 0.        , 0.        , 0.2       , 0.        ],
    #        [0.         , 0.        , 0.        , 0.        , 0.        , 0.16666667]])
 

    temp_array3 - temp_array
    # array([[ 0., -2., -3., -4., -5., -6.],
    #        [-1., -1., -3., -4., -5., -6.],
    #        [-1., -2., -2., -4., -5., -6.],
    #        [-1., -2., -3., -3., -5., -6.],
    #        [-1., -2., -3., -4., -4., -6.],
    #        [-1., -2., -3., -4., -5., -5.]])


    temp_array3 + temp_array
    # array([[2., 2., 3., 4., 5., 6.],
    #        [1., 3., 3., 4., 5., 6.],
    #        [1., 2., 4., 4., 5., 6.],
    #        [1., 2., 3., 5., 5., 6.],
    #        [1., 2., 3., 4., 6., 6.],
    #        [1., 2., 3., 4., 5., 7.]])

    temp_array + temp_array3
    # array([[2., 2., 3., 4., 5., 6.],
    #        [1., 3., 3., 4., 5., 6.],
    #        [1., 2., 4., 4., 5., 6.],
    #        [1., 2., 3., 5., 5., 6.],
    #        [1., 2., 3., 4., 6., 6.],
    #        [1., 2., 3., 4., 5., 7.]])



    # (2,3) * (3,2) 的类型  numpy中无法进行操作，矩阵中可以进行运算
    temp_list5 = [[1,2,3],[3,4,5]]
    temp_array5 = np.array(temp_list5) # temp_array5 的形状 (2, 3)

    temp_list6 = [[1,2],[3,4],[5,6]]
    temp_array6 = np.array(temp_list6) # temp_array6 的形状(3, 2)

    temp_array5 * temp_array6 # 报错 ValueError: operands could not be     broadcast together with shapes (2,3) (3,2) 

    # 注： 不同大小的数组之间的运算叫作广播，稍后介绍。
     
  
# `总注：本次笔记记录到此，以上内容大多参考于《利用python进行数据处理》,内容上也多有引用。因小贱在实际工作中实际应用中用到的并不多，还请读者老爷发现问题多多指教，在此不胜感激！`


####  *本人水平有限， 如有错误欢迎提出指正！如有参考， 请注明出处！！禁止抄袭，遇抄必肛！！！*











