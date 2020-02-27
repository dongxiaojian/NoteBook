1. make和new的区别:

   `make`和`new`都在堆上分配内存. 但是他们的行为不同, 适用于不同的类型.

   

   new(T) 返回的是T的指针, 其中T为一个类型, 不是一个值, 为T类型新值分配内存空间并将此空间初始化为T类型的的零值, 返回的是新值得地址,即T的指针*T的值, 该指针指向T的新分配的零值.

   make(T, args) 用来为slice,map货channel类型分配内存和初始化一个对象, 返回类型的引用we不是指针, 返回值根据T的不同而不同.

   简单来讲: new 的作用是初始化一个指向类型的指针(*T)，make 的作用是为 slice，map 或 chan 初始化并返回引用(T)

   ```go
   testNew := new([2]int)
   	fmt.Println(testNew) //返回 &[0 0]
   	(*testNew)[0] = 123
   	fmt.Println(testNew) // 返回: &[123 0]
   
   
   	testMake := make([]int, 3, 5)
   	fmt.Println(testMake) // 返回 [0 0 0]
   	testMake[0] = 1
   	fmt.Println(testMake) // [2 0 0]
   	
   
   ```

   

2. golang中的单引号 双引号和反引号:

   *双引号* 用来创建可即系的字符串面量(支持转义, 但不能用来引用多行);

   *反引号* 用来创建原生的字符串面量, 这些字符串可能有多行组成(不支持任何转义序列,比如\t, \n,这些都是字符串), 原生的字符串面量多用于书写多行消息, html以及正则表达式;

   *单引号* 是Golang中一种特殊的类型:rune, 指的是: 码点字面量, 不做任何转义的字面内容.Golang中单引号的类型是int32. 所以,单引号中只能有一个字符.且多用`[]rune`来处理中文字符.

   ```go
   	
   	a := 's'
   	fmt.Printf("%v, %T\n", a, a)  // 115, int32
   	b := "s"
   	fmt.Printf("%v, %T\n", b, b) // s, string
   	c := `s`
   	fmt.Printf("%v, %T\n", c, c) // s, string
   ```

   字符串的基本处理:

   ```go
   func backQuote(){
   	testStr := `董小贱的简书`
   	fmt.Println(len(testStr))
   	fmt.Println(utf8.RuneCountInString(testStr))
   	fmt.Println(testStr[:10])
   	testRune := []rune(testStr)
   	fmt.Println(testRune)
   	fmt.Println(string(testRune[:2]))
   	for i:=0; i<len(testRune); i++{
   		fmt.Println(string(testRune[i]))
   	}
   
   }
   
   func doubleQuote(){
   	testStr := "董小贱的简书"
   	fmt.Println(len(testStr))
   	fmt.Println(utf8.RuneCountInString(testStr))
   	fmt.Println(testStr[:10])
   	testRune := []rune(testStr)
   	fmt.Println(testRune)
   	fmt.Println(string(testRune[:2]))
   	for i:=0; i<len(testRune); i++{
   		fmt.Println(string(testRune[i]))
   	}
   }
   ```

   以上的数据大致相同, 唯一的区别是, 遍历字符串的时候,需要转换成`[]rune`类型, 遍历之后再准换成`string`类型才可以.需要注意的是:对`\t`等转义字符的处理, 在单引号中`\t`等转义字符被当做字符串对待, 不进行转义处理, 而在双引号中, `\t`会被转义处理. 

   

`len(str)`的数据跟`[]rune(str)`转换后的数据差三倍(都是中文的情况下)原因: Go中string底层是通过byte数组实现的. 中文字符在unicode下占两个字节, 在utf-8编码下占三个字节. golang默认编码是utf-8. `byte`是指的`uint8`, `rune`是指的`int32`

3. Go语言切片的三种状态: 零切片、空切片和nil切片

   `零切片`： 表示底层数组的二进制内容都是0. 即对应的类型的零值。(即：make之后，长度和容量不为0的切片)

   ```go
testSlice := make([]int, 7, 9)
   fmt.Println(testSlice)
   
   // 结果：[0 0 0 0 0 0 0] 
   
   ```
   
   大概的状态就是：
   
   ![零切片](/Users/dong/Desktop/零切片.png)
   

`nil切片`： 没有使用make进行激活的切片. 与nil比较为true, 可以通过append进行扩容。

```go
	var testSlice []int
	var testSlice2 = *(new([]int))

	fmt.Println(len(testSlice), cap(testSlice))
	fmt.Println(len(testSlice2), cap(testSlice2))
	fmt.Println(testSlice == nil) // true
	fmt.Println(testSlice2 == nil) // true

```



`空切片`：已经激活，但是其长度和容量都为0， 与nil比较为false。可以通过append进行扩容。

```GO

	var testSlice = make([]int, 0)
	var testSlice2 = []int{}

	fmt.Println(len(testSlice), cap(testSlice)) // 0 0
	fmt.Println(len(testSlice2), cap(testSlice2)) // 0  0

	fmt.Println(testSlice == nil) // false
	fmt.Println(testSlice2 == nil) // false


```

One more thing....

切片的类型是一个结构体，将结构体的数据转换成数组类型，如下：

`nil切片`：

```go
var testSlice []int
testSlice2 := *(new([]int))
fmt.Println(*(*[3]int)(unsafe.Pointer(&testSlice))) // [0 0 0]
fmt.Println(*(*[3]int)(unsafe.Pointer(&testSlice2))) // [0 0 0]
```

可以看出nil切片的示意图如下：

![001](/Users/dong/Desktop/001.png)

`零切片`

```GO

	testSlice := make([]int,0)
	testSlice2 := []int{}
	fmt.Println(*(*[3]int)(unsafe.Pointer(&testSlice)))  // [824634167032 0 0]
	fmt.Println(*(*[3]int)(unsafe.Pointer(&testSlice2))) // [824634167032 0 0]
```

name零切片的示意图为：其地址都指向一个全局变量 `zerobase` 这里不展开

![002](/Users/dong/Desktop/002.png)





参考资料：http://www.meirixz.com/archives/80658.html

