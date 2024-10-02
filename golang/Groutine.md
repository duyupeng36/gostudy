# Goroutines

## 创建 Groutine

**在 Go 语言中，每一个并发的执行单元叫作一个 `goroutine`**。设想这里的一个程序有两个函数，一个函数做计算，另一个输出结果，假设两个函数没有相互之间的调用关系
- 一个线性的程序会先调用其中的一个函数，然后再调用另一个
- 如果程序中包含多个 `goroutine`，对两个函数的调用则可能发生在同一时刻。马上就会看到这样的一个程序

如果你使用过操作系统或者其它语言提供的线程，那么你可以简单地把 `goroutine` 类比作一个线程，这样你就可以写出一些正确的程序了

当一个程序启动时，其主函数即在一个单独的 `goroutine` 中运行，我们叫它 `main goroutine`。**新的 `goroutine` 会用 `go` 语句来创建**。在语法上，`go` 语句是一个普通的函数或方法调用前加上关键字 `go`。`go` 语句会使其语句中的函数在一个新创建的 `goroutine` 中运行。而 `go` 语句本身会迅速地完成

```go
f()    // 调用 f(); 等待它返回
go f() // 创建一个新的 goroutine 然后调用 f(); 不等待它的返回
```

下面的例子，`main goroutine` 将计算菲波那契数列的第 $45$ 个元素值。由于计算函数使用低效的递归，所以会运行相当长时间，在此期间我们想让用户看到一个可见的标识来表明程序依然在正常运行，所以来做一个动画的小图标

```go
package main

import (
	"fmt"
	"time"
)

func main() {
	go spinner(100 * time.Millisecond)
	const n = 45
	fibN := fib(n)
	fmt.Printf("\rFibonacci(%d) = %d\n", n, fibN)
}

func spinner(delay time.Duration) {
	for {
		for _, r := range `-\|/` {
			fmt.Printf("\r%c", r)
			time.Sleep(delay)
		}
	}
}

func fib(x int) int {
	if x < 2 {
		return x
	}
	return fib(x-1) + fib(x-2)
}
```

动画显示了几秒之后，`fib(45)` 的调用成功地返回，并且打印结果：

![[spinner.gif|900]]

然后主函数返回。**主函数返回时，所有的 `goroutine` 都会被直接打断，程序退出**。除了从主函数退出或者直接终止程序之外，没有其它的编程方法能够让一个 `goroutine` 来打断另一个的执行，但是之后可以看到一种方式来实现这个目的，**通过 `goroutine` 之间的通信来让一个 `goroutine` 请求其它的 `goroutine`，并让被请求的 `goroutine` 自行结束执行**。

留意一下这里的两个独立的单元是如何进行组合的，**`spinning` 和菲波那契的计算。分别在独立的函数中，但两个函数会同时执行**

## 示例：并发的 `Clock` 服务

**网络编程是并发大显身手的一个领域**，由于服务器是最典型的需要同时处理很多连接的程序，这些连接一般来自于彼此独立的客户端。下面将基于 Go 语言的 `net` 包，编写一个始终服务器。这个包提供编写一个网络客户端或者服务器程序的基本组件，无论两者间通信是使用 `TCP`，`UDP` 或者 `Unix domain sockets`

首先，我们先完成顺序执行的时钟服务器
```go
package main

import (
	"fmt"
	"io"
	"log"
	"net"
	"time"
)

// 处理连接对象的函数
func handleConn(c net.Conn) {
	// 函数退出时关闭 连接
	defer c.Close()
	for {
		// err 标识不能写入了，函数返回
		_, err := io.WriteString(c, time.Now().Format("2006-01-02 15:04:05\n"))
		if err != nil {
			fmt.Printf("客户端 %v 断开连接\n", c.RemoteAddr().String())
			return
		}
		time.Sleep(1 * time.Second)
	}
}

func main() {
	// 创建一个 Listener 对象
	listener, err := net.Listen("tcp", "0.0.0.0:8000")
	if err != nil {
		log.Fatal(err)
	}
	for {
		// 接受客户端的连接
		conn, err := listener.Accept()
		fmt.Printf("接受到 %v 的连接\n", conn.RemoteAddr().String())
		if err != nil {
			log.Print(err)
		}
		handleConn(conn)
	}
}
```

`Listen` 函数创建了一个 `net.Listener` 的对象，这个对象会监听一个网络端口上到来的连接，在这个例子里我们用的是 TCP 的 `localhost:8000` 端口。`listener` 对象的 `Accept` 方法会直接阻塞，直到一个新的连接被创建，然后会返回一个 `net.Conn` 对象来表示这个连接

`handleConn` 函数会处理一个完整的客户端连接。在一个 `for` 死循环中，用 `time.Now()` 获取当前时刻，然后写到客户端。由于 `net.Conn` 实现了 `io.Writer` 接口，我们可以直接向其写入内容。这个死循环会一直执行，直到写入失败。最可能的原因是客户端主动断开连接。这种情况下 `handleConn` 函数会用 `defer` 调用关闭服务器侧的连接，然后返回到主函数，继续等待下一个连接请求

**`time.Time.Format` 方法提供了一种格式化日期和时间信息的方式**。它的参数是一个格式化模板标识如何来格式化时间，而这个格式化模板限定为 **`Mon Jan 2 03:04:05PM 2006 UTC-0700`**。有 $8$ 个部分(周几，月份，一个月的第几天，等等)。**可以以任意的形式来组合前面这个模板**；出现在模板中的部分会作为参考来对时间格式进行输出。在上面的例子中我们用到了年、月、日、小时、分钟和秒

`time` 包里定义了很多标准时间格式，比如 `time.RFC1123`。在进行格式化的逆向操作`time.Parse` 时，也会用到同样的策略

> [!NOTE] Go 的时间格式化
> 这是 Go 语言和其它语言相比比较奇葩的一个地方。。你需要记住格式化字符串是 **1月2日下午3点4分5秒二零零六年UTC-0700**，而不像其它语言那样 `Y-m-d H:i:s` 一样，当然了这里可以用 `1234567` 的方式来记忆，倒是也不麻烦

为了连接例子里的服务器，我们需要一个客户端程序，比如 `netcat` 这个工具(`nc`命令)，这个工具可以用来执行网络连接操作

![[clock.gif|900]]

客户端将服务器发来的时间显示了出来，我们用 `Control+C` 来中断客户端的执行，在 Unix 系统上，你会看到 `^C` 这样的响应

如果你的系统没有装 `nc` 这个工具，你可以用 `telnet` 来实现同样的效果，或者也可以用我们下面的这个用 Go 写的简单的 `telnet` 程序，用 `net.Dial` 就可以简单地创建一个TCP连接

```go
package main

import (
	"io"
	"log"
	"net"
	"os"
)

func main() {
	conn, err := net.Dial("tcp", "localhost:8000")
	if err != nil {
		log.Fatal(err)
	}
	defer conn.Close()
	mustCopy(os.Stdout, conn)
}
func mustCopy(dst io.Writer, src io.Reader) {
	if _, err := io.Copy(dst, src); err != nil {
		log.Fatal(err)
	}
}
```

我们发现，**只有一个客户端可以接受到服务端返回的时间**。为了让服务端可以处理多个客户端的连接，只需要在 `handleConn` 前添加关键字 `go` 即可启动一个 `goroutine` 用于处理和客户端的通信，让 `main goroutine` 继续等待客户端的连接

```go
package main

import (
	"fmt"
	"io"
	"log"
	"net"
	"time"
)

// 处理连接对象的函数
func handleConn(c net.Conn) {
	// 函数退出时关闭 连接
	defer c.Close()
	for {
		// err 标识不能写入了，函数返回
		_, err := io.WriteString(c, time.Now().Format("2006-01-02 15:04:05\n"))
		if err != nil {
			fmt.Printf("客户端 %v 断开连接\n", c.RemoteAddr().String())
			return
		}
		time.Sleep(1 * time.Second)
	}
}

func main() {
	// 创建一个 Listener 对象
	listener, err := net.Listen("tcp", "0.0.0.0:8000")
	if err != nil {
		log.Fatal(err)
	}
	for {
		// 接受客户端的连接
		conn, err := listener.Accept()
		fmt.Printf("接受到 %v 的连接\n", conn.RemoteAddr().String())
		if err != nil {
			log.Print(err)
		}
		go handleConn(conn)
	}
}
```

让我们再次运行该服务端程序。并使用 `nc` 命令进行连接

![[clock2.gif|900]]

这样我们的服务端可以处理多个客户端的连接了

## 示例：并发的 `Echo` 服务

`clock` 服务器每一个连接都会起一个 `goroutine`。在本节中我们会创建一个 `echo` 服务器，这个服务在每个连接中会有多个 `goroutine`。大多数 `echo` 服务仅仅会返回他们读取到的内容，就像下面这个简单的 `handleConn` 函数所做的一样

```go
func handleConn(c net.Conn) {
    io.Copy(c, c) // NOTE: ignoring errors
    c.Close()
}
```

一个更有意思的 `echo` 服务应该模拟一个实际的 `echo` 的“回响”，并且一开始要用大写 `HELLO` 来表示“声音很大”，之后经过一小段延迟返回一个有所缓和的 `Hello`，然后一个全小写字母的 `hello` 表示声音渐渐变小直至消失，像下面这个版本的 `handleConn` (译注：笑看作者脑洞大开)：

```go
func echo(c net.Conn, shout string, delay time.Duration) {
    fmt.Fprintln(c, "\t", strings.ToUpper(shout))
    time.Sleep(delay)
    fmt.Fprintln(c, "\t", shout)
    time.Sleep(delay)
    fmt.Fprintln(c, "\t", strings.ToLower(shout))
}

func handleConn(c net.Conn) {
    input := bufio.NewScanner(c)
    for input.Scan() {
        echo(c, input.Text(), 1*time.Second)
    }
    // NOTE: ignoring potential errors from input.Err()
    c.Close()
}
```

我们需要升级我们的客户端程序，这样它就可以发送终端的输入到服务器，并把服务端的返回输出到终端上，这使我们有了使用并发的另一个好机会

当 `main goroutine` 从标准输入流中读取内容并将其发送给服务器时，另一个 `goroutine` 会读取并打印服务端的响应。当 `main goroutine` 碰到输入终止时，例如，用户在终端中按了`Control-D(^D)`，在 Windows 上是`Control-Z`，这时程序就会被终止，尽管其它 `goroutine` 中还有进行中的任务

```go
➜  echo ./netcat
Hello?
         HELLO?
         Hello?
         hello?
Is there anybody there?
         IS THERE ANYBODY THERE?
         Is there anybody there?
         is there anybody there?
^C      
➜  echo ./netcat
Hello?
         HELLO?
         Hello?
         hello?
Is there anybody there?
         IS THERE ANYBODY THERE?
Yooo-hooo!
         Is there anybody there?
         is there anybody there?
         YOOO-HOOO!
         Yooo-hooo!
         yooo-hooo!
2024/04/14 18:31:37 writeto tcp [::1]:41974->[::1]:8001: write /dev/stdout: use of closed network connection
➜  echo killall echo  
[1]  - 499260 terminated  ./echo
```

注意客户端的第三次 `shout` 在前一个 `shout` 处理完成之前一直没有被处理，这貌似看起来不是特别“现实”。真实世界里的回响应该是会由三次 `shout` 的回声组合而成的。为了模拟真实世界的回响，我们需要更多的 `goroutine` 来做这件事情。这样我们就再一次地需要 `go` 这个关键词了，这次我们用它来调用 `echo`

```go
func handleConn(c net.Conn) {
	input := bufio.NewScanner(c)
	for input.Scan() {
		go echo(c, input.Text(), 1*time.Second)
	}
}
```

`go` 后跟的函数的参数会在 `go` 语句自身执行时被求值；因此 `input.Text()` 会在 `main goroutine` 中被求值。 现在回响是并发并且会按时间来覆盖掉其它响应了

```
➜  echo ./netcat 
Is there anybody there?
         IS THERE ANYBODY THERE?
Yooo-hooo!
         YOOO-HOOO!
         Is there anybody there?
         Yooo-hooo!
         is there anybody there?
         yooo-hooo!
```

**让服务使用并发不只是处理多个客户端的请求，甚至在处理单个连接时也可能会用到**，就像我们上面的两个 `go` 关键词的用法。然而在我们使用 `go` 关键词的同时，**需要慎重地考虑  `net.Conn` 中的方法在并发地调用时是否安全**，事实上对于大多数类型来说也确实不安全

## runtime 包

包 `runtime` 包含与 Go 运行时系统交互的操作，例如 **控制 `goroutines` 的函数**。它还包含 `reflect` 包使用的底层类型信息。这里，我们先看几个关于控制 `goroutines` 的函数

### `runtime.Gosched()`

`runtime.Gosched()` 该函数用于 **让出当前 `goroutine` 占用的 CPU 时间片**， 调度器安排其他等待的任务运行，并在下次再获得 CPU 时间轮片的时候， 从该出让CPU的位置恢复执行

```go
package main

import (
	"fmt"
	"runtime"
	"sync"
)

var wg sync.WaitGroup

func sing() {
	defer wg.Done()
	for i := 0; ; i++ {
		runtime.Gosched() // Gosched放弃了处理器，允许其他 goroutines 运行。它不会挂起当前的 goroutine，因此执行会自动恢复。
		fmt.Printf("---唱歌---%d\n", i+1)
	}
}

func dance() {
	defer wg.Done()
	for i := 0; ; i++ {
		fmt.Printf("---跳舞---%d\n", i+1)
	}
}

func main() {
	wg.Add(1)
	go sing()
	wg.Add(1)
	go dance()
	wg.Wait()
}
```

运行结果为

```
....
---跳舞---40188
---跳舞---40189
---跳舞---40190
---跳舞---40191
---跳舞---40192
---跳舞---40193
---唱歌---807
---跳舞---40194
---跳舞---40195
---跳舞---40196
....
---跳舞---41112
---跳舞---41113
---唱歌---833
---跳舞---41114
---跳舞---41115
```

### `runtime.Goexit()`

**`Goexit` 会终止调用它的 `goroutine`**。其他 `goroutine` 不会受到影响。`Goexit` 会在终止 `goroutine` 之前运行所有延迟调用。由于 `Goexit` 并非`panic`，因此这些延迟函数中的任何 `recover` 调用都将返回 `nil`

```go
package main

import (
	"fmt"
	"runtime"
)

func main() {
	go func() {
		defer fmt.Println("A.defer")

		func() {
			defer fmt.Println("B.defer")
			runtime.Goexit() // 终止当前 goroutine, import "runtime"
			fmt.Println("B") // 不会执行
		}()

		fmt.Println("A") // 不会执行
	}() //不要忘记()

	//死循环，目的不让主goroutine结束
	for {
		;
	}
}
```

### `runtime.GOMAXPROCS()`

`GOMAXPROCS` 设置可以同时执行的 CPU 的最大数目，并返回前一个设置。默认值为`runtime.NumCPU`。如果 `n < 1`，则不改变当前设置。当调度器的值发生改变时，返回之前的值。

```go
package main

import (
	"fmt"
	"runtime"
	"time"
)

func main() {
	fmt.Println(runtime.GOMAXPROCS(2)) // 16 返回的是前一次的最大cpu数

	for {
		go fmt.Print(1)
		fmt.Print(0)
		time.Sleep(1 * time.Second)
	}
}
```

