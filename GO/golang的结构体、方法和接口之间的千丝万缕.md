### 1. 结构体和方法之间

###### 方法接收器是结构体的值与指针中的区别：

```go
package main

import "fmt"

type Person struct {
	Name string
	Age  int
}

func (this Person) Call() {
	fmt.Println("啊啊啊")
}

func (this *Person) Touch() {
	fmt.Println("痒痒痒")
}

func main() {
	person := Person{
		Name: "董小贱",
		Age:  18,
	}
	person.Call()  
	person.Touch()
}
```

以上的代码可以正常执行，也就是说不论方法接收器是值还是指针类型，都可以通过结构体的值调用。在方法体中，`this`(搞PY交易的我还是习惯叫这玩意为`self`，我还是入乡随俗,用`this`)的值时结构体的值还是指针类型，这跟方法的接收器有关，接收器接收的参数是值，则`this`为结构体的值(结构体是值传递)；接收的参数是指针，则`this`的值为指针类型。这里的调用看不出来问题点，我们修改一下结构体中的字段试一下：

```go
package main

import "fmt"

type Person struct {
	Name string
	Age  int
}

func (this Person) Call() {
	this.Name = "dong"
	fmt.Println("啊啊啊")
}

func (this *Person) Touch() {
	this.Name = "xiaojian"
	fmt.Println("痒痒痒")
}

func main() {
	person := Person{
		Name: "董小贱",
		Age:  18,
	}
	fmt.Println(person) //{董小贱 18}
	person.Call() // 啊啊啊
	fmt.Println(person) // {董小贱 18}
	person.Touch() // 痒痒痒
	fmt.Println(person) //{xiaojian 18}
}
```

从执行结果中可以看出，接收指针类型的方法`Touch`对全局变量结构体中的`Name`字段的值进行了修改；而接收为值的方法`Call`对全局变量的结构体的`Name`没有修改成功,`Call`中对`Name`的修改，只是在其方法体中起作用。

这里调用`Touch`方法，其实应该是`(&person).Touch()`， golang在这里做了简化，可以直接使用`person.Touch() `来进行调用。

从以上可以看出，结构体传参是值传参。可以试下，分别打印下`person`、`Call`方法中的`this`、`Touch`方法中的`this`的地址，会发现，全局变量中的`person`和`Touch`方法中的`this`是统一的！

以上不是本笔记的重点，以下才是...

### 2 . 结构体、方法和接口之间的千丝万缕

先看一段代码：

```go
package main

import "fmt"

type sportsMan interface {
	Run()
	Jump()
}

type runner interface {
	Run()
}

type highJumper interface {
	Jump()
}

type Person struct {
	Name string
	Age  int
}

func (this Person) Run() {
	fmt.Println("跑跑跑")
}
func (this *Person) Jump() {
	fmt.Println("跳跳跳")
}

func main(){
	person := Person{
		Name:"董小贱",
		Age:18,
	}

	person.Run()  
	person.Jump()

	runner := runner(person)
	runner.Run()

	highjumper := highJumper(person)
  highjumper.Jump()

}
```

以上代码会报错：`cannot convert person (type Person) to type highJumper:Person does not implement highJumper (Jump method has pointer receiver)`。 把传入的值改为`&person`就可以正常运行了。那么问题来了，为啥子`Person`的实例不能直接赋值给接口`highJumper`？ 我看了golang的官方文档以及查了好多资料，现在整理如下：

先说一个定义：`方法集`： 定义了接口的接收规则。方法集定义了一组关联到给定类型的值或者指针的方法。 定义方法是使用的接收者的类型决定了这个方法是关联到值还是关联到指针，还是两个都关联。

| 变量类型 | 方法接收器类型  |
| :------: | :-------------: |
|    T     |      (t T)      |
|    *T    | (t T) && (t *T) |

简单的讲:  T类型的值得方法集只包含值接收声明的方法。指向T类型的指针方法集即包含值接受者声明的方法，也包含指针接收者声明的方法。 从方法接收者的角度来看看之间的关系：

| 方法接收器类型 | 变量类型 |
| :------------: | :------: |
|    （t T）     | T && *T  |
|     (t *T)     |    *T    |

这两张表描述的是一个规则：如果使用指针接收器来实现一个接口，那么只有指向那个类型的指针才能够实现对应的接口。 如果使用值接收器来实现一个接口，那么这个类型的值和指针都能实现对应的接口。

说了这么多，还是拿代码说事：

```go
package main

import "fmt"

type sportsMan interface {
	Run()
	Jump()
}

type runner interface {
	Run()
}

type highJumper interface {
	Jump()
}

type Person struct {
	Name string
	Age  int
}

func (this Person) Run() {
	fmt.Println("跑跑跑")
}
func (this *Person) Jump() {
	fmt.Println("跳跳跳")
}

func main(){
	person := Person{
		Name:"董小贱",
		Age:18,
	}

	person.Run()
	person.Jump()

	runner := runner(person)
	runner.Run()
  
	highjumper := highJumper(&person)
	highjumper.Jump()
  
	sportsman := sportsMan(&person)
	sportsman.Jump()
	sportsman.Run()
}
```

`highJumper`接口中的方法接收器的是指针类型，所以将`&person`类型传入不会报错。（如上边表中的描述, `runner`方法中也能传入指针类型，这就能呼应上了）

`sportsMan`接口中的方法包含指针类型接收器和值类型接收器，如果传入值类型的值，则不满足指针类型的接收器，只有传入指针类型的值，才能满足值类型接收器和指针类型接收器。参考上边方法集的那两个表，这里也就能呼应上了。

总结一句话：接口中方法是值类型接收器，随便传; 接口中方法是指针类型接收器，只能传指针类型；接口中方法中的接收器类型都有，为了都满足也只能传指针类型。

其实还有一个点需要注意： 接口中方法的值类型接收器，传入的是指针类型，在方法中操作结构体的字段的指也不会改变。



