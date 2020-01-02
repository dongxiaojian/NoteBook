orm原理

## 0. 从@property装饰器说起

​	在绑定属性时, 如果我们直接把属性暴露出去,会导致意想不到的错误.比如:把体重设置为负值. 

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

*注:为什么把值赋到self.__height和self.__weight中, 是因为如果还是把值给到self.height和self.weight中的话, 会造成死循环而导致异常的抛出 *

这一版本的代码看起来很繁琐,但也是实现了对应的需求.



