orm原理

## 0. 从@property装饰器说起

##### **0.1** 在绑定属性时, 如果我们直接把属性暴露出去,会导致意想不到的错误.比如:把体重设置为负值. 

	先来看一段代码, 输出BMI指数的:

```python
class BMI:
    """计算BMI指数"""
    
    def __init__(self, name, height, weight):
        self.name = name
        self.height = height / 100
        self.weight = weight
    
    def bmi(self):
        """体重指数BMI=体重/身高的平方（国际单位kg/㎡)"""
        bmi_value = self.weight / self.height ** 2
        print(f"{self.name}的BMI指数是{bmi_value:.2f}")


bmi = BMI("董小贱", 174, 87)
bmi.bmi()
>> 董小贱的BMI指数是28.74 
```

很显然, 在输入的height和weight的值是不能小于等于0的数.但是现在的情况下,并没有做这类的限制,并不能满足现实中的要求,那么现在就可以用@property来实现需求.

```python
class BMI:
    """计算BMI指数"""
    
    def __init__(self, name, height, weight):
        self.name = name
        self.height = height / 100
        self.weight = weight
    
    def bmi(self):
        """体重指数BMI=体重/身高的平方（国际单位kg/㎡)"""
        bmi_value = self.weight / self.height ** 2
        print(f"{self.name}的BMI指数是{bmi_value:.2f}")
    
    @property
    def height(self):
        return self.__height
    
    @height.setter
    def height(self, value):
        if value <= 0:
            raise ValueError('Height must not be less than or equal to 0')
        else:
            self.__height = value
            
    @property
    def weight(self):
        return self.__weight
    
    @weight.setter
    def weight(self, value):
        if value <= 0:
            raise ValueError('weight must not be less than or equal to 0')
        else:
            self.__weight = value


bmi = BMI("董小贱", 174, 87)
bmi.bmi()

```

*注:为什么把值赋到`self.__height`和`self.__weight`中, 是因为如果还是把值给到self.height和self.weight中的话, 会造成死循环而导致异常的抛出*

另外需要注意的是: **特性都是类属性, 但是特性管理的其实都是实例属性的存取.**

##### **0.2** 关于特性的一些问题点

###### 0.2.1: 实例属性会覆盖类属性

```python
In [2]: class Test():
   ...:     name = "dongxiaojian"
   ...:     

In [3]: test = Test() 

In [4]: test.name # 获取实例对象中不存在的属性name, 此时获取到的是类属性name
Out[4]: 'dongxiaojian'

In [5]: test.name = "董小贱" # 为实例对象的name属性赋值

In [6]: test.name #获取实对象的name属性, 现在获取到的不再是类属性
Out[6]: '董小贱'

In [7]: Test.name # 现在类属性通过类来访问
Out[7]: 'dongxiaojian'
```

###### 0.2.2 实例属性不会覆盖类特性

```python
In [8]: class Test():
   ...:     @property
   ...:     def name(self):
   ...:         return "dongxiaojian"
   ...:     

In [9]: Test.name # 通过类访问name特性, 获取的是特性对象本身，不会运行特性的读值方法。
Out[9]: <property at 0x1082d5f48>

In [10]: test = Test()

In [11]: test.name # 通过实例访问返回的是值
Out[11]: 'dongxiaojian'

In [12]: test.name = "董小贱" # 通过实例赋值报错,导致赋值失败
---------------------------------------------------------------------------
AttributeError                            Traceback (most recent call last)
<ipython-input-12-babb940ddd04> in <module>()
----> 1 test.name = "董小贱"

AttributeError: can't set attribute

In [13]: vars(test) # 实例属性还是为空
Out[13]: {}

In [14]: test.__dict__['name'] = "董小贱" # 可以把值存入到__dict__中,不会报错

In [15]: test.name # 存入__dict__中的值不会被实例访问到, 特性没有被实例属性所遮盖.
Out[15]: 'dongxiaojian'

In [16]: Test.name = "dong" # 通过类将特性覆盖掉

In [17]: test.name # 现在可以通过实例访问到保存在__dict__中的属性值
Out[17]: '董小贱'

In [18]: Test.name # 现在的类属性的值
Out[18]: 'dong'
```

###### 0.2.3 新的类特性会覆盖实例属性(以下代码在0.2.2的基础上)

```python
In [19]: Test.name # 接着0.2.2的, 类属性
Out[19]: 'dong'

In [20]: test.name  # 接着0.2.2的, 实例属性
Out[20]: '董小贱'

In [21]: Test.name = property(lambda self: "这是创建的新特性") # 使用特性覆盖类属性

In [22]: test.name # 实例属性已经被覆盖
Out[22]: '这是创建的新特性'

In [23]: Test.name # 类属性返回的对象
Out[23]: <property at 0x1082f56d8>

In [24]: del Test.name # 删掉类属性

In [25]: test.name # 实例属性恢复
Out[25]: '董小贱'

In [26]: Test.name # 无法访问到类属性
---------------------------------------------------------------------------
AttributeError                            Traceback (most recent call last)
<ipython-input-26-ee2a064643f5> in <module>()
----> 1 Test.name

AttributeError: type object 'Test' has no attribute 'name'
```



##### **0.3** 通过property函数来实现一个特性工厂函数

###### 0.3.1 可以发现, 0.1中的代码实现了具体的需求, 但是可以发现代码是很冗余, 那怎么实现呢? 请看以下:

```python
class BMI:
	"""计算BMI指数"""

	def __init__(self, name, height, weight):
		self.name = name
		self.height = height / 100
		self.weight = weight
	
	def bmi(self):
		"""体重指数BMI=体重/身高的平方（国际单位kg/㎡)"""
		bmi_value = self.weight / self.height ** 2
		print(f"{self.name}的BMI指数是{bmi_value:.2f}")
	
	def get_height(self):
		return self.__height
	
	def set_hight(self, value):
		if value <= 0:
			raise ValueError('value must not be less than or equal to 0')
		else:
			self.__height = value
	
	def get_weight(self):
		return self.__weight
	
	def set_weight(self, value):
		if value <= 0:
			raise ValueError('value must not be less than or equal to 0')
		else:
			self.__weight = value
	
	weight = property(get_weight,set_weight )
	height = property(get_height, set_hight)
	
bmi = BMI("董小贱", 174, 87)
bmi.bmi()
```

###### 0.3.2 可以看得出, 开始很冗余, 那么根据上边的代码来实现工厂函数:

```python
def func(storage_name):
	"""为属性提供限制"""
	def get_value(instance):
		return instance.__dict__[storage_name]
  
	def set_value(instance, value):
		if isinstance(value, int): value = float(value)
		if not isinstance(value, float): raise ValueError("Value must be float")
		if value < 0:
			raise ValueError("value must not be less than or equal to 0")
		else:
			instance.__dict__[storage_name] = value
	
	return property(get_value, set_value)

class BMI:
	"""计算BMI指数"""
	height = func('height')
	weight = func('weight')
	
	def __init__(self, name, height, weight):
		self.name = name
		self.height = height / 100
		self.weight = weight
	
	def bmi(self):
		"""体重指数BMI=体重/身高的平方（国际单位kg/㎡)"""
		bmi_value = self.weight / self.height ** 2
		print(f"{self.name}的BMI指数是{bmi_value:.2f}")


bmi = BMI("董小贱", 174, 87)
bmi.bmi()

```

 其中的func函数,就是一种装饰器的写法, *instance*其对应的应该是BMI对象的实例, 这里如果写成*self*会很怪,所以用的*instance*替代. 类属性*height*和*weight*其实是property对象, 所以这里会覆盖实例中的同名属性.这里的代码看起来很奇怪, 可以理解为把0.3.1中类中的代码提取出来,然后封装成函数了.

##### 0.4 通过特性删除属性

上边讲的大都是设置值以及获取值, 通过特性还能删除属性.下边一个简单的例子, 通过装饰器实现:

```python
In [19]: class Test():
    ...:     
    ...:     def __init__(self,value):
    ...:         self.height = value
    ...:     
    ...:     @property
    ...:     def height(self):
    ...:         return self._height
    
    ...:     @height.setter
    ...:     def height(self, value):
    ...:         self._height = value
    ...:     
    ...:     @height.deleter
    ...:     def height(self):
    ...:         self._height = 0
    ...:         print (self._height)
    ...:   
In [20]: test = Test(150)

In [21]: test.height
Out[21]: 150

In [22]: del test.height
0

In [23]: test.height
Out[23]: 0

In [24]: test.height = 160

In [25]: test.height
Out[25]: 160

In [26]: del test.height
0

```

property()函数也有对应的参数, 下边是property的帮助文档, 其中fdel对应的接受的参数是删除属性值得函数:

```python
In [27]: help(property)

Help on class property in module builtins:

class property(object)
 |  property(fget=None, fset=None, fdel=None, doc=None) -> property attribute
 |  
 |  fget is a function to be used for getting an attribute value, and likewise
 |  fset is a function for setting, and fdel a function for del'ing, an
 |  attribute. 
 |  
```

特性的内容差不多就这么多的东西, 下边看下个内容, 描述符

## 1. 描述符相关

###### 1.1 描述符是对多个属性运用相同存取逻辑的一种方式. 描述符是实现了特定协议的类, 这个协议包括`__get__`、`__set__`和`__delete__`方法. property类实现了完整的描述符协议.

现在将0.3.2的函数改写成描述符类:

```python
class Quantity:
	"""为属性提供限制"""
	
	def __init__(self, storage_name):
		self.storage_name = storage_name
	
	def __set__(self, instance, value):  # 这里改为__set__()方法.
		if isinstance(value, int): value = float(value)
		if not isinstance(value, float): raise ValueError("Value must be float")
		if value < 0:
			raise ValueError("value must not be less than or equal to 0")
		else:
			instance.__dict__[self.storage_name] = value

class BMI:
	"""计算BMI指数"""
	height = Quantity('height')
	weight = Quantity('weight')
	
	def __init__(self, name, height, weight):
		self.name = name
		self.height = height / 100
		self.weight = weight
	
	def bmi(self):
		"""体重指数BMI=体重/身高的平方（国际单位kg/㎡)"""
		bmi_value = self.weight / self.height ** 2
		print(f"{self.name}的BMI指数是{bmi_value:.2f}")


bmi = BMI("董小贱", 174, 87)
bmi.bmi()
```

通过描述符类写跟0.3.2中的函数实现的一样的效果, 这其中的几个定义:

- 描述符类: 实现描述符协议的类, 即:Quantity类.
- 托管类: 把描述符实例声明为类属性的类, 即BMI类.
- 描述符实例: 描述符类的各个实例, 声明为托管类的类属性.即:height和weight.
- 托管实例: 托管类的实例. 即bmi.
- 储存属性: 托管实例中存储自身托管属性的属性.实例中的height和weight属性就是存储属性.
- 托管属性: 托管类中有描述符实例处理的公开属性.值存储在储存属性中. 也就是说, 描述符实例和存储属性为托管属性建立了基础.

值得注意的是: 编写`__set__`方法时, 要记住self和instance参数的意思: self是描述符实例, instance是托管实例.管理实例属性的描述符应该把值存储在托管实例中. 因此, Python 才为描述符中的那个方法提供了 instance 参数。如果将各个托管属性的值直接存在描述符实例中,就是讲上边的例子中的`instance.__dict__[self.storage_name] = value`写成`self.__dict__[self.storage_name] = value`, 这种写法是有问题的, 这其中的self是描述符实例, 即使托管类中的类属性, 实际运行中, 可能有多个托管实例,而托管类的类属性即描述符实例只有两个: BMI.height和BMI.weight, 多个托管实例共享两个描述符实例所对应的值显然是有问题的.

以上的代码看起起来还是不够简洁, 我们并不想在Quantity()实例中写成固定参数, 现在修改如下:

```python
import uuid


class Quantity:
	"""为属性提供限制"""
	
	def __init__(self):
		self.storage_name = str(uuid.uuid4())
	
	def __set__(self, instance, value):
		if isinstance(value, int): value = float(value)
		if not isinstance(value, float): raise ValueError("Value must be float")
		if value < 0:
			raise ValueError("value must not be less than or equal to 0")
		else:
			instance.__dict__[self.storage_name] = value
			
	def __get__(self, instance, owner): # 这里必须指定__get__, 因为storage_name和托管属性的名称不相同.
		return getattr(instance, self.storage_name)

class BMI:
	"""计算BMI指数"""
	height = Quantity()
	weight = Quantity()
	
	def __init__(self, name, height, weight):
		self.name = name
		self.height = height / 100
		self.weight = weight
	
	def bmi(self):
		"""体重指数BMI=体重/身高的平方（国际单位kg/㎡)"""
		bmi_value = self.weight / self.height ** 2
		print(f"{self.name}的BMI指数是{bmi_value:.2f}")


bmi = BMI("董小贱", 174, 87)
bmi.bmi()

```

值得注意的是: `__get__`方法有三个参数: self, instance和owner.  其中instance指的是托管实例即bmi, 通过描述符获取实例属性时用的到, owner指的是托管类即BMI的引用, 通过描述符获取类属性时用的到.以上的例子中, 如果通过BMI.height时会报错:`AttributeError: 'NoneType' object has no attribute 'eceb1f2d-5e4a-462f-8177-ce9b5ec836a7'`

那么怎么想property那样返回描述符的对象呢? 只需要在`__get__`加个判断就好, 改动如下:

```python
def __get__(self, instance, owner): 
  if instance:
		return getattr(instance, self.storage_name)
  else:
    return self
```

其实以上还有个问题, 就是报错的信息都是uuid的信息, 并不是对应的托管实例属性的属性信息, 调试起来的话相当不方便, 解决这个问题的方法先按下不表, 咱们接着看关于描述符的一些信息.

##### 2.1 描述符类型(覆盖性描述符和非覆盖型描述符)

python中存取属性的方式是不对等的: *通过实例读取属性时, 通常返回的是实例中定义的属性, 如果实例中没有指定的属性, name会获取类属性;  但是为实例属性赋值时,通常会在实例中创建属性, 不会影响到类.* 这种不对等的方式也影响到了描述符的行为. 根据描述符是否实现了`__set__`方法(是否会覆盖实例属性的值), 分为*覆盖型描述符*和*非覆盖型描述符*

###### 2.1.1 覆盖型描述符

实现 `__set__`方法的描述符属于*覆盖型描述符*,描述符是类属性, 实现了`__set__`方法的话, 会覆盖对实例属性的赋值操作.

1. 如果同时实现了`__set__`和`__get__`方法, 也称强制描述符(影响了实例属性的读写, 实例属性的读写都要通过描述符处理). 例子如上边的1.1
2. 如果只实现了`__set__`,没有实现`__get__`的覆盖型描述符,通过实例读取描述符会返回描述符对象本身, 因为没有处理读操作的 `__get__` 方法。如果直接通过实例的` __dict__` 属性创建同名实例属性, 以后再设置那个属性时, 仍会由 `__set__` 方法插手接管， 但是读取那个属性的话，就会直接从实例中返回新赋予的值, 而不会返回描述符对象。也 就是说, 实例属性会遮盖描述符, 不过只有读操作是如此.(这里有点绕, 可以简单的理解为通过` __dict__`修改的实例属性, 会覆盖通过`__set__`修改的值, 正常修改的话, 还是会通过`__set__`方法设置.)(影响实例属性的写操作, 不影响其读操作.)

```python
import uuid

class Quantity:
	"""为属性提供限制"""
	
	def __init__(self):
		self.storage_name = str(uuid.uuid4())
	
	def __set__(self, instance, value):
		if isinstance(value, int): value = float(value)
		if not isinstance(value, float): raise ValueError("Value must be float")
		if value < 0:
			raise ValueError("value must not be less than or equal to 0")
		else:
			instance.__dict__[self.storage_name] = value
	
class BMI:
	"""计算BMI指数"""
	weight = Quantity()
	
	def __init__(self):
	
		self.weight = 3
 ######################以下是执行结果########################
In [31]: bmi = BMI()

In [32]: BMI.weight
Out[32]: <__main__.Quantity at 0x10f41f320>

In [33]: bmi.weight
Out[33]: <__main__.Quantity at 0x10f41f320>

In [34]: bmi.weight = 6

In [35]: BMI.weight
Out[35]: <__main__.Quantity at 0x10f41f320>

In [36]: bmi.weight
Out[36]: <__main__.Quantity at 0x10f41f320>

In [37]: bmi.__dict__['weight'] = 9

In [38]: BMI.weight
Out[38]: <__main__.Quantity at 0x10f41f320>

In [39]: bmi.weight
Out[39]: 9
	
```

 从此可以看出:特性也是强制描述符, 如果没有提供设置值函数, 获取特性的值时会抛出AttributeError异常.

###### 2.1.2 非覆盖性描述符

只实现了`__get__`方法的描述符属于*非覆盖性描述符*, 如果设置了同名的实例属性, 实例属性会覆盖描述符(影响描述符的读写操作), 只是描述符无法处理那个实例属性.

```python
In [45]: import uuid
    ...: 
    ...: 
    ...: class Quantity:
    ...:     """为属性提供限制"""
    ...:     
    ...:     def __init__(self):
    ...:         self.storage_name = str(uuid.uuid4())
    ...:     
    ...:             
    ...:     def __get__(self, instance, owner):
    ...:         return self
    ...: 
    ...: 
    ...: class BMI:
    ...:     """计算BMI指数"""
    ...:     weight = Quantity()
    ...:     
    ...:     def __init__(self,):
    ...:         self.weight = 3
    ...:     
    ...: 

In [46]:  ## 以下是执行结果

In [46]: bmi = BMI()

In [47]: bmi.weight
Out[47]: 3

In [48]: BMI.weight
Out[48]: <__main__.Quantity at 0x10f4086d8>

In [49]: bmi.weight = 9

In [50]: bmi.weight
Out[50]: 9

In [51]: BMI.weight
Out[51]: <__main__.Quantity at 0x10f4086d8>
```

##### 2.2 在类中覆盖描述符

读类属性的操作可以由依附在托管类上定义 有 `__get__` 方法的描述符处理，但是写类属性的操作不会由依附在托管类上定义有 `__get__` 方法的描述符处理。所以, 这就造成了不管描述符是不是覆盖型, 为类属性赋值都能覆盖描述符.

好, 现在的描述符已经差不多了, 现在解决之前遗留的那个问题: 使用描述符, 报错的信息不是实例属性的名称,如何使其成为实例属性的名称.

#### 3. 上述问题的解决
##### 3.1 类装饰器
看下边的代码

```
import uuid

def class_decorator(cls):
    for key, attr in cls.__dict__.items():
        if isinstance(attr, Quantity):
            type_name = type(attr).__name__
            attr.storage_name = f"{type_name}_{key}" # 注意，这里不能直接用key的值，否则，获取对应值的时候会到导致死循环
          
    return cls


class Quantity:
    """为属性提供限制"""
    
    def __init__(self):
        self.storage_name = str(uuid.uuid4())
    
    def __set__(self, instance, value):
        if isinstance(value, int): value = float(value)
        if not isinstance(value, float): raise ValueError("Value must be float")
        if value < 0:
            raise ValueError("value must not be less than or equal to 0")
        else:
            instance.__dict__[self.storage_name] = value
    
    def __get__(self, instance, owner):
        return getattr(instance, self.storage_name) # 如果上边直接用key的值，会导致这里造成死循环， 导致错误抛出


@class_decorator
class BMI:
    """计算BMI指数"""
    height = Quantity()
    weight = Quantity()
    
    def __init__(self, name, height, weight):
        self.name = name
        self.height = height / 100
        self.weight = weight
    
    def bmi(self):
        """体重指数BMI=体重/身高的平方（国际单位kg/㎡)"""
        bmi_value = self.weight / self.height ** 2
        print(f"{self.name}的BMI指数是{bmi_value:.2f}")


bmi = BMI("董小贱", 174, 87)
bmi.bmi()
```
通过类装饰器， 将本来产生的uuid替换掉， 可以实现报错的情况下显示可追溯的报错信息。 但是新的问题也随之出来了： 装饰器不能继承，只队直接依附的类有效。被装饰的类的子类可能继承也可能不继承装饰器所作的改动。
那么就需要用到元编程了.

##### 3.2 元编程基础

元类是制造类的工厂, 是用于构建类的类. 

python 中一切皆对象, 那么, 类也是对象. 一般的类都是都是继承自`object`, 默认的情况下, python中的类是`type`类的实例. 那么他们之间的关系是:`object` 是 `type` 的实例，而 `type` 是 `object` 的子类.(先有鸡还是先有蛋??). 所有的类都是`type`的实例,`元类`就是`type`的子类.因此可以作为`类工厂`(其实例就是类). 普通的类是通过`__init__`方法来初始化实例.同样的, 元类可以通过实现 `__init__` 方法定制实例(即类)。元类的 `__init__` 方法可以做到类装饰器能做的任何事情.

##### 3.3 用元编程实现

```python
import uuid

class Quantity:
	"""为属性提供限制"""

	def __init__(self):
		self.storage_name = str(uuid.uuid4())
	
	def __set__(self, instance, value):
		if isinstance(value, int): value = float(value)
		if not isinstance(value, float): raise ValueError("Value must be float")
		if value < 0:
			raise ValueError("value must not be less than or equal to 0")
		else:
			instance.__dict__[self.storage_name] = value
	
	def __get__(self, instance, owner):
		return getattr(instance, self.storage_name)  

class Meta(type): # 继承自type制作作元类
  
	def __init__(cls, name, bases, attr_dict): # 一般情况下, self写作cls, 因为元类产生的实例是类.
		super().__init__(name, bases, attr_dict)
		for key, attr in attr_dict.items():
			if isinstance(attr, Quantity):
				type_name = type(attr).__name__
				attr.storage_name = f"{type_name}_{key}"

class BMI(metaclass=Meta): # 指定元类是Meta
	"""计算BMI指数"""
	height = Quantity()
	weight = Quantity()
	
	def __init__(self, name, height, weight):
		self.name = name
		self.height = height / 100
		self.weight = weight
	
	def bmi(self):
		"""体重指数BMI=体重/身高的平方（国际单位kg/㎡)"""
		bmi_value = self.weight / self.height ** 2
		print(f"{self.name}的BMI指数是{bmi_value:.2f}")

bmi = BMI("董小贱", 174, 87)
bmi.bmi()

```