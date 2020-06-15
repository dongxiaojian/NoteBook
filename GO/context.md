

Go的标准库 context

###### *在 Go http包的Server中，每一个请求在都有一个对应的 goroutine 去处理。请求处理函数通常会启动额外的 goroutine 用来访问后端服务，比如数据库和RPC服务。用来处理一个请求的 goroutine 通常需要访问一些与请求特定的数据，比如终端用户的身份认证信息、验证相关的token、请求的截止时间。 当一个请求被取消或超时时，所有用来处理该请求的 goroutine 都应该迅速退出，然后系统才能释放这些 goroutine 占用的资源。*

那么如何控制goroutine优雅的退出呢？

## 1. 实例

### 1.0. 基础代码

```go
package main

import (
	"fmt"
	"sync"
	"time"
)

var wg sync.WaitGroup

func DoSomething() {
	for {
		fmt.Println("董小贱")
		time.Sleep(2 * time.Second)
	}
	wg.Done()
}

func main() {
	wg.Add(1)
  go DoSomething()  //   这个DoSomething() 没有停止条件，不会停止
	wg.Wait()
	fmt.Println("DoSomething over")
}
```

### 1. 1全局变量方式

```go
package main

import (
	"fmt"
	"sync"
	"time"
)

var wg sync.WaitGroup
var status bool = true

func DoSomething() {
	for {
		fmt.Println("董小贱")
		time.Sleep(2 * time.Second)
		if !status {
			break
		} else {
			continue
		}
	}
	wg.Done()
}

func main() {
	wg.Add(1)
	go DoSomething()
	time.Sleep(10 * time.Second)
	status = false 	//通过修改全局变量的状态，给gorotine发送终止信号
	wg.Wait()
	fmt.Println("DoSomething over")
}
```



### 1.2. select + channel 版本

```go
package main

import (
	"fmt"
	"sync"
	"time"
)

var wg sync.WaitGroup
var chann = make(chan bool)

func DoSomething() {
LoopTag:
	for {
		fmt.Println("董小贱")
		time.Sleep(2 * time.Second)
		select {
		case <-chann: // 信号接收
			break LoopTag // 退出整个lable
		default:
			continue
		}
	}
	wg.Done()
}

func main() {
	wg.Add(1)
	go DoSomething()
	time.Sleep(10 * time.Second)
	chann <- false // 通过无缓存的channel来给goroutine发送终止信号
	wg.Wait()
	fmt.Println("DoSomething over")
}

```

### 1.3.context版本

```go
package main

import (
	"context"
	"fmt"
	"sync"
	"time"
)

var wg sync.WaitGroup

func DoSomething(ctx context.Context) {
LoopTag:
	for {
		fmt.Println("董小贱")
		time.Sleep(2 * time.Second)
		select {
		case <-ctx.Done(): // 等待父级通知
			break LoopTag
		default:
			continue
		}
	}
	wg.Done()

}

func main() {
	ctx, cancel := context.WithCancel(context.Background())
	wg.Add(1)
	go DoSomething(ctx)
	time.Sleep(10 * time.Second)
	cancel() //cancel()执行，通知子goroutine结束
	wg.Wait()
	fmt.Println("DoSomething over")
}

```

当在`DoSomething`中再开启子goroutine时，将ctx传入到函数当中即可：

```go
package main

import (
	"context"
	"fmt"
	"sync"
	"time"
)

var wg sync.WaitGroup

func DoSomething2(ctx context.Context) {
LoopTag:
	for {
		fmt.Println("董小小贱")
		time.Sleep(2 * time.Second)
		select {
		case <-ctx.Done():
			break LoopTag
		default:
			continue
		}
	}
	wg.Done()
}

func DoSomething(ctx context.Context) {
	go DoSomething2(ctx)
LoopTag:
	for {
		fmt.Println("董小贱")
		time.Sleep(2 * time.Second)
		select {
		case <-ctx.Done():
			break LoopTag
		default:
			continue
		}
	}
	wg.Done()

}

func main() {
	ctx, cancel := context.WithCancel(context.Background())
	wg.Add(1)
	go DoSomething(ctx)
	time.Sleep(10 * time.Second)
	cancel()
	wg.Wait()
	fmt.Println("DoSomething over")
}
```

那么context的作用就呼之欲出了：简单讲就是用来处理goroutine之间传递截止日期、取消信号和其他请求范围内的值

## 2. context相关

***

Go1.7 加入的新的标准库`context` 它定义了`Context`类型， 它跨API边界和进程之间传递截止日期、取消信号和其他请求范围内的值。

对服务器的传入请求应该创建上下文，而对服务器的传出调用应该接受上下文。它们之间的函数调用链必须传播上下文，可以选择将其替换为使用`WithCancel`、`WithDeadline`、`WithTimeout`或`WithValue`创建的派生上下文。当一个上下文被取消时，从它派生的所有上下文也被取消。

`WithCancel`、`WithDeadline`和`WithTimeout`函数接收一个`Context`并返回一个派生子`Context`和一个`CancelFunc`。调用`CancelFunc`取消子进程及其子进程，删除父进程对子进程的引用，并停止所有相关的计时器。如果没有调用`CancelFunc`，则会泄漏子节点及其子节点，直到父节点被取消或计时器触发。

不要将`Context`存储在结构类型中，应该显示地将上下文传递给需要它的每个函数，`Context`应该是第一个参数，通常命名为ctx。

 																																			--- 以上来自context包文档  

***



#### 2.1 context包相关

##### 2.1.1 `context.Context`是一个接口，该接口定义了四个需要实现的方法。具体签名如下：

```go
type Context interface {
    Deadline() (deadline time.Time, ok bool)
    Done() <-chan struct{}
    Err() error
    Value(key interface{}) interface{}
}


// Deadline方法需要返回当前Context被取消的时间，也就是完成工作的截止时间（deadline）；
//Done方法需要返回一个Channel，这个Channel会在当前工作完成或者上下文被取消之后关闭，多次调用Done方法会返回同一个Channel；
//Err方法会返回当前Context结束的原因，它只会在Done返回的Channel被关闭时才会返回非空的值；
		//如果当前Context被取消就会返回Canceled错误；
		//如果当前Context超时就会返回DeadlineExceeded错误；
//Value方法会从Context中返回键对应的值，对于同一个上下文来说，多次调用Value 并传入相同的Key会返回相同的结果，该方法仅用于传递跨API和进程间跟请求域的数据；
```

###### 2.1.2  CancelFunc

```go
type CancelFunc func()

// CancelFunc告诉一个goroutine放弃它的工作。CancelFunc不会等待工作停止。在第一次调用之后，对CancelFunc的后续调用什么也不做。
```

##### 2.1.3 Background()

```go
func Background() Context

// Background返回一个非nil的空上下文。它从未被取消，没有值，也没有期限。它通常由主函数、初始化和测试使用，并作为传入请求的根context（最顶层的context，其他的都是它的派生子context）
```

###### 2.1.4 TODO()

```go
func TODO() Context

//TODO返回一个非nil的空context。代码应该使用context。TODO当不清楚要使用哪个context或者它还不可用时(因为周围的函数还没有被扩展到接受context参数)。TODO由静态分析工具识别，该工具确定context是否在程序中正确传播

//这个就是知道要用context，但不清楚用哪个，埋点用。
```

###### `background`和`todo`本质上都是`emptyCtx`结构体类型，是一个不可取消，没有设置截止时间，没有携带任何值的Context。

#### 2.2 With系列函数

###### 2.2.1 WithCancel

```go
func WithDeadline(parent Context, deadline time.Time) (Context, CancelFunc)

//WithCancel返回带有新Done通道的父进程的一个副本。当返回的cancel函数被调用时，或者当父context的Done通道被关闭时，返回context的Done通道将被关闭，以最先发生的情况为准。

// 取消此context释放与之关联的资源，因此代码应该在此context中运行的操作完成后立即调用cancel。
```

具体用法可以看上边的例子。

###### 2.2.2 WithDeadline

```go
func WithDeadline(parent Context, deadline time.Time) (Context, CancelFunc)

// deadline 是绝对时间，比如：d := time.Now().Add(50 * time.Millisecond)

//WithDeadline返回父context的一个副本，并将deadline调整为不迟于d。如果父上下文的deadline已经早于d，则WithDeadline(parent, d)在语义上等同于父上下文。返回的上下文的Done通道在deadline过期时关闭，在调用返回的cancel函数时关闭，或者在关闭父上下文的Done通道时关闭，以最先发生的情况为准。

// 取消此上下文释放与之关联的资源，因此代码应该在此上下文中运行的操作完成后立即调用cancel。
```

一个例子：

```go
func main() {
	d := time.Now().Add(50 * time.Millisecond)
	ctx, cancel := context.WithDeadline(context.Background(), d)
	defer cancel()
	select {
	case <-time.After(1 * time.Second):
		fmt.Println("overslept")
	case <-ctx.Done():
		fmt.Println(ctx.Err())
	}
}


// 这里会先执行ctx.Done() 然后打印异常
```

###### 2.2.1 WithTimeout

```go
func WithTimeout(parent Context, timeout time.Duration) (Context, CancelFunc)

// 跟WithDeadline的区别在于，WithTimeout是用的相对时间。
```

###### 2.2.1 WithValue

```go
func WithValue(parent Context, key, val interface{}) Context

//WithValue返回父节点的一个副本，其中与键关联的值为val。

//只将上下文值用于传输流程和api的请求范围的数据，而不用于将可选参数传递给函数。

// 提供的键必须是可比较的，并且不应该是string类型或任何其他内置类型，以避免使用context的包之间的冲突。WithValue的用户应该定义自己的键类型。为了避免在分配给interface{}时进行分配，上下文键通常具有具体类型struct{}。或者，导出的上下文关键变量的静态类型应该是指针或接口。

```

一个例子：

```go
package main

import (
	"context"
	"fmt"
)

type favContextKey string

func main() {

	f := func(ctx context.Context, k favContextKey) {
		if v := ctx.Value(k); v != nil {
			fmt.Println("found value:", v)
			return
		}
		fmt.Println("key not found:", k)
	}

	k := favContextKey("language")
	ctx := context.WithValue(context.Background(), k, "Go")

	f(ctx, k)
	f(ctx, favContextKey("color"))
}
```



#### 2.3 context的注意事项：

```
	推荐以参数的方式显示传递Context
​ 以Context作为参数的函数方法，应该把Context作为第一个参数。
​ 给一个函数方法传递Context的时候，不要传递nil，如果不知道传递什么，就使用context.TODO()
​ Context的Value相关方法应该传递请求域的必要数据，不应该用于传递可选参数
​ Context是线程安全的，可以放心的在多个goroutine中传递
```



