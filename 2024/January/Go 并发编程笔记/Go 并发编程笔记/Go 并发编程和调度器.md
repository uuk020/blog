# Go 并发编程和调度器

并发编程, 目的就是提高程序的吞吐能力和性能, 能够同时处理很多业务, 而不管这些业务是相同的还是不同的. 

为什么并发编程能够提升程序的吞吐能力和性能呢?

- **合理地拆分程序的逻辑, 使不同的业务模块可以并发执行, 同时可以处理多个业务模块, 从而在相同时间内有更多业务单位可以被处理.** 在串行编程的模式下, 必须一个模块执行完, 才能执行下一个模块, 其中一个模块在执行的时候, 其他模块就一直等待, 同一个时间点只能有一个模块执行. 需注意的是程序拆解并不是一件容易的事, 因为业务逻辑中有些部分必须等待其他并发部分全部或者部分完成后才能执行.如统计访问几大搜索完整的平均延迟,统计模块需要等待访问百度, bing, 谷歌等并发模块都返回结果后才能统计. **需要多个并发模块协调处理叫作编排.** 对于任务的编排也不是一件容易的事情. 有的时候, 并发使用某个资源还会涉及竞争的问题或者相关共享数据可能出现意想不到的结果, 这时候就需要**同步原语**的支持
    
    > 同步原语指帮助你开发并发编程的一些库和数据结构, 专门用来做并发同步的数据结构
    > 
- 对于 I/O 敏感型的程序, 并发编程对其性能提升巨大. I/O 敏感型的程序会将大量的时间耗费在等待 I/O 完成上, 如从网络中读取数据, 往磁盘写入批量的持久化数据,或者访问一个数据库等. 对于串行编程模型, 程序在等待的过程不能执行其他业务逻辑(即使 CPU 空闲); 但是对于并行编程模型, 程序在等待 I/O 完成的过程中完全可以处理其他业务逻辑, 程序不会被阻塞, 所以并行对 I/O 敏感程序的性能提升有时候是巨大的
- 并发编程可以充分利用现代 CPU 的多核能力.

### Go 特别适合并发编程

Go 语言从语言设计上为并发编程提供了最简单的方式, 并且设计了比线程跟更轻量级的 goroutine, 还为 CSP 模型提供了容易使用的 channel 类型, 并将其作为内建类型直接提供

在 Go 语言中, 实现并发编程非常简单, 在函数的执行语句的前面加上 go 关键字, 该函数就会自动生成一个 goroutine 来执行

```go
package main

import (
	"fmt"
	"net/http"
	"time"
)

func getFromBaidu() {
	resp, err := http.Get("http://www.baidu.com")
	if err != nil {
		fmt.Println(err)
		return
	}
	fmt.Println(resp.Status)
}

func main() {
	go func() {
		fmt.Println("Hello from a goroutine")
	}() // 并发输出

	go getFromBaidu() // 并发访问

	time.Sleep(time.Second)
}
```

在函数和方法的调用前面加上 **go,** 就会创建一个 goroutine, 这个 goroutine 会加入 Go 调度器的队列进行调度或者执行, 调用者不会被阻塞, 而是会继续执行, 所以避免了程序直接退出. go 语句并不理会函数的返回值的数量和类型.

```go
func add(x, y int) (int error) {
	if y == 0 {
		return 0, fmt.Errorf("y can not be zero")
	}
  return x + y, nil
}

func main() {
	go add(1, 2)
	time.Sleep(time.Second)
}
```

这样语句会被 go linter 工具检查出不符合要求, 因为 error 并没有被处理. 可以在不想做 golangcli-lint 检查的那一行添加了注释 nolint:

```go
go add(1, 2) // nolint
```

如果需要处理 error 信息, 则可以使用匿名函数将其封装起来

```go
go func() {
	if err := http.ListenAndServe(":8080", nil); err != nil {
		fmt.Println(err)
	}
}()
```

go 语句的参数求值和正常的函数/方法调用都是一样的

```go
	var list []int
	go func(l int) {
		time.Sleep(time.Second)
		fmt.Printf("passed len: %d, current len: %d\n", l, len(list))
	}(len(list))

	list = append(list, 1)
	time.Sleep(2 * time.Second)
// passed len: 0, current len: 1
```

go 语句的这一点和 defer 语句是类似的.

```go
typ  list := []int{1}

	foo := func(l int) {
		time.Sleep(time.Second)
		fmt.Printf("passed len: %d, current len: %d\n", l, len(list))
	}

	fmt.Println("before go")
	go foo(len(list))

	foo = func(l int) {
		fmt.Printf("passed len: %d, current len: %d\n", l*100, len(list)*100)
	}

	time.Sleep(2 * time.Second)
	fmt.Println("after go")
	foo(len(list))
// passed len: 1, current len: 1
// passed len: 100, current len: 100
```

在上面的代码, go 语句执行的还是被修改前的 foo 的值

```go
 	type Student struct {
		Name string
	}
	s := &Student{
		Name: "博文",
	}

	go func() {
		time.Sleep(time.Second)
		fmt.Printf("student name: %s\n", s.Name) // 博文博文
	}()

	s.Name = "博文博文"
	time.Sleep(2 * time.Second)
```

Go 总是传值的, 即使传递一个指针, 也是把指针的值传递进去.

### 并发(concurrency) vs 并行(parallelism)

Go 三巨头之一的 Rob Rike 在 2012 年分享了一个著名的演讲 - “并发不是并行” , 详细对比了这两个概念, 并演示了将一个串行程序改成并发/ 并行执行的几种方式.

- 并发是同时处理很多事情, 并行是同时做很多事情
    
    **并发是同时处理很多事情, 有时间段的概念,** 这些事情可能会有一个先后顺序,  但是会在这个时间段内去做. 从这个时间段来看, 这些事情都被处理了. **并行是在同一个时间点有多件事情都在做**
    
    并发例子 - 调度在同一个 CPU 上编写代码同时还在听着音乐
    
    并行例子 - 两台电脑, 一台机器写代码, 一台机器播音乐
    
- 并发和并行并不相同, 但是相关
    
    **并发和并行区别的第一条, 是同时”处理” 和 同时”做”的区别**. 如去银行办理业务, 进门后, 大堂经理检查相关证件, 然后在取号机取个号给我, 我的业务正式被”处理”了.  环顾四周, 发现大堂里全是已被”处理”但是还在等待窗口办理的顾客. 有限的几个窗口正在”做”业务, 每个”做”的过程都非常漫长, 有可能需要等半天才能轮到我”做”业务. 银行办理业务是并发执行的 ,但是并行执行的就寥寥几个业务
    
- 并发的焦点是设计结构, 并行的焦点是程序的执行
    
    并发的本质是我们要设计/实现一个结构, 这个结构可以使程序的不同计算模块并发地执行. 这些模块的执行可能真的是并行, 比如在多核的 CPU 上, 不同模块不会相互阻塞, 它们被分配到不同的核上, 所以可以并行地执行.而在一个核的 CPU 上, 它们不能并行地执行, 但是可以并发地执行, 在一段时间内每个模块都可以占用 CPU 的时间片, 每个模块都可以被执行. **并发编程的本质就是要设计/实现这样的程序结构, 以便不同的模块可以并发地执行. 并发的目标之一就是能利用并行(多核)的能力, 但是并行的目标不是并发.**
    

并发例子 - 《 深入理解 Go 并发编程》- “鸟窝客栈”

### Go 并发并不一定最快

[阿姆达尔定律](https://www.wikiwand.com/zh-hans/%E9%98%BF%E5%A7%86%E8%BE%BE%E5%B0%94%E5%AE%9A%E5%BE%8B): 提升系统的一部分性能对整个系统的性能有多大影响的经验法则. 在设计并发程序的时候, 应尽量让可并发的部分在整个系统中占比比较大, 这样才可能得到更大的加速比. 尽量减少串行的部分, 同时尽量提升并发部分的性能.

一般来说, 将串行程序改为并行程序之后, 其性能会得到提升. 但也不是绝对的, 在某些情况下, 串行编程性能反而更好.

```go
func parttion(a []int, lo, hi int) int {
	pivot := a[hi]
	i := lo - 1
	for j := lo; j < hi; j++ {
		if a[j] < pivot {
			i++
			a[j], a[i] = a[i], a[j]
		}
	}
	a[i+1], a[hi] = a[hi], a[i+1]
	return i + 1
}
// 串行执行
func quickSort(a []int, lo, hi int) {
	if lo >= hi {
		return
	}
	p := parttion(a, lo, hi)
	quickSort(a, lo, p-1)
	quickSort(a, p+1, hi)
}
// 并发执行
func quickSort_go(a []int, lo, hi int, done chan struct{}) {
	if lo >= hi {
		done <- struct{}{}
		return
	}
	p := parttion(a, lo, hi)
	childDone := make(chan struct{}, 2)
	go quickSort_go(a, lo, p-1, childDone)
	go quickSort_go(a, p+1, hi, childDone)
	<-childDone
	<-childDone
	done <- struct{}{}
}
```

随机生成测试, 测试它们运行所花费的时间:

```go
// 不同机器耗时可能不同, 这个例子并发执行是会慢于串行执行
// 串行执行, 耗时: 939.441433ms
// 并发执行, 耗时: 4.359995399s
func Bench_Quicksort() {
	rand.New(rand.NewSource(time.Now().UnixNano()))
	n := 10000000
	testData1, testData2, testData3 := make([]int, 0, n), make([]int, 0, n), make([]int, 0, n)
	for i := 0; i < n; i++ {
		val := rand.Intn(n * 100)
		testData1 = append(testData1, val)
		testData2 = append(testData2, val)
		testData3 = append(testData3, val)
	}
	start := time.Now()
	quickSort(testData1, 0, len(testData1)-1)
	fmt.Println("串行执行, 耗时:", time.Since(start))

	done := make(chan struct{})
	start = time.Now()
	go quickSort_go(testData2, 0, len(testData2)-1, done)
	<-done
	fmt.Println("并发执行, 耗时:", time.Since(start))
}

```

使用 1000 万个数据进行排序, 并发程序会将数据分成两个部分, 这两个部分使用两个子 goroutine 来运行, 每个子 goroutine 负责其中一部分数据. 同样地, 每个子 goroutine 也会将自己要排序的部分分成两个孙 goroutine, 这样整体上并发程序会创建许许多多的 goroutine. 虽然 goroutine 相对于线程是轻量级的, 但这不意味着它没有资源的占用和性能的损耗. 与操作系统的线程相比, goroutine 的内存占用更小: Go 1. 4 以后的 goroutine  只占用 2KB, 在运行中 goroutine 的内存占用还会按需进行调整; 更进一步, Go 1.19 会根据历史 goroutine 栈的使用率来初始化新的 goroutine 栈的大小, goroutine 栈的大小不再是固定的 2KB. 因为创建新的 goroutine 会伴随栈的分配, 这也会损耗性能. 同时大量的 goroutine 在调度和垃圾回收检查时也会占用一定的时间, 所以整体上来说, 这些额外时间反而让程序的性能下降了.

那么因为生成太多的 goroutine 影响并发程序的性能, 那么可以减少 goroutine 的生成, 并且递归深度在 3 以内, 采用并发的方式快速排序; 如果递归深度超过 3, 则改为串行的方式, 这样就既利用了并发执行的优势, 又减少了过多 goroutine 带来的管理损耗

```go
func quickSort_go2(a []int, lo, hi int, done chan struct{}, depth int) {
	if lo >= hi {
		done <- struct{}{}
		return
	}
	depth--
	p := parttion(a, lo, hi)
	if depth > 0 {
		childDone := make(chan struct{}, 2)
		go quickSort_go2(a, lo, p-1, childDone, depth)
		go quickSort_go2(a, p+1, hi, childDone, depth)
		<-childDone
		<-childDone
	} else {
		quickSort(a, lo, p-1)
		quickSort(a, p+1, hi)
	}
	done <- struct{}{}
}
```

运行并测试

```go
// 串行执行, 耗时: 941.411448ms
// 并发执行, 耗时: 4.569095304s
// 优化并发执行, 耗时: 512.385379ms
func Bench_Quicksort() {
	rand.New(rand.NewSource(time.Now().UnixNano()))
	n := 10000000
	testData1, testData2, testData3 := make([]int, 0, n), make([]int, 0, n), make([]int, 0, n)
	for i := 0; i < n; i++ {
		val := rand.Intn(n * 100)
		testData1 = append(testData1, val)
		testData2 = append(testData2, val)
		testData3 = append(testData3, val)
	}
	start := time.Now()
	quickSort(testData1, 0, len(testData1)-1)
	fmt.Println("串行执行, 耗时:", time.Since(start))

	done := make(chan struct{})
	start = time.Now()
	go quickSort_go(testData2, 0, len(testData2)-1, done)
	<-done
	fmt.Println("并发执行, 耗时:", time.Since(start))

	done1 := make(chan struct{})
	start = time.Now()
	go quickSort_go2(testData3, 0, len(testData3)-1, done1, 2)
	<-done1
	fmt.Println("优化并发执行, 耗时:", time.Since(start))
}
```

为什么是递归深度为3, 通过基准测试的方式来确定这个值的.

### Go 运行时调度器

当操作系统的线程切换到另一个线程时, CPU 会执行一个操作, 叫做**上下文切换(context  switch)** , 操作系统会在中断, 系统调用时执行线程上下文切换. 线程上下文切换是一种昂贵的操作, 因为操作系统需要将用户态转换到内核态, 保存要切换线程的执行状态, 也就是将一些重要的寄存器的值和进程状态保存在线程控制块数据结构中. 当恢复线程的运行时, 需要将这些状态加载到寄存器中, 从内核态转移到用户态. 这个过程相当复杂, 耗时,如果又涉及进程上下文切换, 就更加耗时了.  

goroutine 的调度是由 Go 运行时控制的, 每个编译的 Go 程序都会附加一个很小的 Go 运行时, 负责内存分配,  goroutine  调度和垃圾回收. goroutine 会和某个线程绑定, 它是用户态, 并且初始的栈也比较小, 所以它的上下文切换开销比较小, 大致上,  可以认为 goroutine 的上下文切换开销是线程上下文切换开销的十分之一(大概).

最早的 Go 运行时(1.0 版以下) 采用 GM 模型, 随着 Go 运行时的优化, 改成了 GPM 模型

![Untitled](Go%20并发编程和调度器/Untitled.png)

- G: 表示 goroutine, 存储了与 goroutine 相关的信息, 比如栈, 状态, 要执行的函数等.
- P: 表示逻辑 processor , 它跟 CPU 的处理器没有半点关系. P 负责把 M 和 G 捏合起来, 让一系列的 goroutine 在 某个 M 上顺序执行. 在默认情况下, P 的数量等于 CPU 逻辑核的数量.也可以使用 runtine.GOMAXPROCS 改变它. 每个 P 都有一个自己本地 goroutine 队列
- M: 表示执行计算资源单元, Go 会把它和操作系统的线程一一对应. 只有在 P 和 M 绑定之后, 才能让 P 的本地队列中的 goroutine 运行起来.

运行 Go 程序时, 程序的入口并不是 main 函数,  而是会调用 Go 运行时的一个函数, 完成运行时的初始化工作, 包括调度器初始化和垃圾回收等工作. 最开始会创建 m0 和 g0 , 完成调度器初始化的工作, 为 main.main 生成一个 goroutine, 并且被 m0 执行

Go 运行时会启动 N 个 P 和 M, 并把 P 和 M 捏合在一起, 除非有阻塞的 I/O 导致 P 和 M 解绑,  P 再找到”新欢”(新的 M). 每个 P 都有自己的本地队列, 它会顺序执行其本地的 goroutine 队列, 但是每61次或者本地队列没有 P 可执行的 goroutine 时, 它会从全局队列找到一个 goroutine 来执行,  避免全局队列的 goroutine 没有机会执行.  还有工作窃取算法, 如果一个 P 太多清闲, 没有什么 goroutine 可执行, 它也会尝试从其他的 goroutine 队列窃取一半的 goroutine 过来, 让工作比较均衡. 

但是这里还有两个大家可能不熟悉的场景, 也就是 timer 和 netpoll .  timer 经过几次演化, 它的四叉堆依附在 P 上, P 在调度的时候也会检查这个四叉堆上是否有 timer 需要触发, 同时也会窃取 timer . 调度器还会检查 netpoll, 对于那些网络I/O 已经就绪的 goroutine,  也是有机会执行读/写操作的. sysmon 是一个独立的 goroutine, 不依附在某个 P 上, 而是运行在一个独立的 M中, 定时运行一次, 检查与网络 I/O 相关的 goroutine 和那些长时间运行的 goroutine(超过 10 ms),  避免某个 goroutine 长时间占用计算资源.

 Go1.14 实现了基于信号的抢占式调度, 向正在运行的 goroutine 所绑定的那个 M 发出 [SIGURG 信号](https://golang.design/under-the-hood/zh-cn/part2runtime/ch06sched/preemption/), 信号处理函数会进行调度, 让其他 goroutine 有机会执行

m0 和 g0 是两个特殊的对象. m0是 Go 程序启动时的第一个线程, 也就是主线程. 这个 m0是放在全局变量中的, 不像其他的 m 都是运行时的局部变量. 程序运行的时候只有一个 m0

g0 是调度的 goroutine, 每个 m 都有一个 g0. g0 和其他的 goroutine 是有区别的, g0的栈是系统分配的, 在 Linux 上栈大小默认固定为 8MB, 不能扩展和缩小. 而普通的 goroutine  栈大小默认是 2KB(Go1.10 改成根据历史使用自动调整的方式), 它们的栈是可扩展的, 一个 goroutine 运行完成后, 或者新建一个 goroutine 时, 调度器会先被切换到 g0 上, 让 g0 负责调度.

每个 m 都有一个 g0 用来调度它的 goroutine, 而且都叫 g0. 其中与 m0 绑定的那个 g0 也是放在全局变量中的.

> 1.  Rob Rike 演讲 - [https://go.dev/blog/waza-talk](https://go.dev/blog/waza-talk)
2.  Rob Rike 的 PPT 文件 - [https://go.dev/talks/2012/waza.slide#1](https://go.dev/talks/2012/waza.slide#1)
3. 幼麟实验室 - GPM [https://www.bilibili.com/video/BV1hv411x7we/?p=16](https://www.bilibili.com/video/BV1hv411x7we/?p=16)
>