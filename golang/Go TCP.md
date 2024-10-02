# TCP

TCP/IP（Transmission Control Protocol/Internet Protocol ），即 **传输控制协议/因特网互联协议**

它是一个包含很多工作在 **不同层** 的协议的 **协议族**，其中最著名的 $2$ 个协议分别是 TCP 和 IP 协议

它最早起源于美国国防部（缩写为DoD）的 ARPA 网项目，1982 年应用于美国所有军事网络。IBM、AT&T、DEC从1984年起就开始使用TCP/IP协议。TCP/IP更加广泛的传播是在1989年，加州大学伯克利分校在BSD中加入了该协议。微软是在Win95中增加

TCP/IP 协议，共定义了四层：**数据链路**、**网络层**、**传输层**、**应用层**。

![[Pasted image 20240808181547.png|900]]


TCP/IP 协议是事实标准。目前局域网和广域网基本上也都用该协议

关于 TCP/IP 协议栈，我们在 [[网络#TCP/IP 分层协议]] 中简单介绍了

## 传输层协议

传输层有两个协议 TCP 和 UDP


|          |   TCP    |     UDP     |
| :------: | :------: | :---------: |
|  是否有连接   |    是     |      否      |
|   是否可靠   |    是     |      否      |
| 数据到达是否有序 |    有     |      无      |
|   使用场景   | 大多数场景都适合 | 主要在音频和视频的传输 |

由于 TCP 需要保证传输的可靠性和数据包的有序性，需要通信双方先建立连接。参考 [[网络#TCP 连接管理]]，其中介绍了 **三次握手建立连接** 和 **四次挥手断开连接** 的完整过程

**TCP协议是流协议**，也就是一 **大段数据看做字节流**，一段段持续发送这些字节

## TCP 服务端编程

TCP 服务端编程模型

![[Pasted image 20240808182744.png|900]]

> [!tip]
> 创建 Socket 对象
> 
> 绑定 IP 地址 (Address) 和端口(Port)
> 
> 将套接字变为被动套接字(开启监听, listen)
> 
> 客户端向服务端请求连接时，会得到用于传输的新的 Socket 对象
> 
> 之后通过这个新的 Socket 对象与客户端进行数据收发

```
socket对象 --> bind((IP, PORT)) --> listen --> accept --> close
											 |--> recv or send --> close
```

TCP服务端程序的处理流程：
- 监听端口: `server, err := net.ListenTCP("tcp", "ip:port")`
- 接收客户端请求建立连接: `connection, err := server.Accept()`
- 创建 Goroutine 处理连接: `go func(conn net.Conn){}()`

Go 将 Socket 进行了隐藏，提供了更便捷的接口

```go
package main

import (
	"bytes"
	"fmt"
	"log"
	"net"
	"runtime"
)

func main() {
	localTCPAddr, err := net.ResolveTCPAddr("tcp", "localhost:8080") // 解析字符串形式的地址，返回 TCPAddr 结构体的指针
	if err != nil {
		log.Fatal(err)
	}

	// 创建 socket 绑定地址 开启监听这三步被封装成了一个函数
	server, err := net.ListenTCP("tcp", localTCPAddr)
	if err != nil {
		log.Fatal(err)
	}

	// 不断的接受客户端的连接
	for {

		conn, err := server.Accept()
		if err != nil {
			log.Printf("接受客户端连接失败，进行下一次接受")
		}

		// 重启一个 Goroutine 处理与客户端的通信，避免阻塞后续的客户端连接
		go func(conn net.Conn) {
			defer func() {
				if err := conn.Close(); err != nil {
					log.Printf("关闭与客户端 %v 的连接失败: %v", conn.RemoteAddr(), err)
					runtime.Goexit()
				}
			}()

			var buf [4096]byte
			for {
				n, err := conn.Read(buf[:])

				if n == 0 {
					log.Printf("客户端 %v 主动关闭连接\n", conn.RemoteAddr().String())
					break
				}

				if err != nil {
					log.Printf("读取出现错误")
					break
				}

				fmt.Printf("客户端发来消息: %s\n", buf[:n])
				conn.Write(bytes.ToUpper(buf[:n]))
			}

		}(conn)
	}
}
```

## TCP 客户端编程


一个TCP客户端进行TCP通信的流程如下：
1. 建立与服务端的链接 `conn, err:= net.DialTCP("tcp",localTCPAddr remoteTCPAddr)`
	1. `localTCPAddr` 通常指定为 `nil`，让程序随机占领
2. 进行数据收发 
	1. 向服务器发送数据`conn.Write()`，让 main Goroutine 做这件事
	2. 读取服务器回传的数据 `conn.Read()` ，单独另起一个 Goroutine 做这件事情
3. 关闭链接: `conn.Close()`

```go
package main

import (
	"io"
	"log"
	"net"
	"os"
)

func catchErr(err error) {
	if err != nil {
		log.Fatal(err)
	}
}

func main() {
	remoteTCPAddr, err := net.ResolveTCPAddr("tcp", "localhost:8080")
	catchErr(err)

	conn, err := net.DialTCP("tcp", nil, remoteTCPAddr)
	catchErr(err)

	done := make(chan struct{})
	// 启动一个 Goroutine 用于处理服务端的返回
	go func(conn net.Conn) {
		_, err := io.Copy(os.Stdout, conn)
		if err != nil {
			log.Println("done")
		}
		done <- struct{}{}
	}(conn)

	// main Goroutine 处理向客户端发送数据
	_, err = io.Copy(conn, os.Stdin)
	if err != nil {
		log.Println(err)
	}
	conn.Close()
	<-done
}
```

## 粘包

服务端代码如下

```go
package main

import (
	"bufio"
	"fmt"
	"io"
	"net"
)

// socket_stick/server/main.go

func process(conn net.Conn) {
	defer conn.Close()
	reader := bufio.NewReader(conn)
	var buf [1024]byte
	for {
		n, err := reader.Read(buf[:])
		if err == io.EOF {
			break
		}
		if err != nil {
			fmt.Println("read from client failed, err:", err)
			break
		}
		recvStr := string(buf[:n])
		fmt.Println("收到client发来的数据：", recvStr)
	}
}

func main() {

	listen, err := net.Listen("tcp", "127.0.0.1:30000")
	if err != nil {
		fmt.Println("listen failed, err:", err)
		return
	}
	defer listen.Close()
	for {
		conn, err := listen.Accept()
		if err != nil {
			fmt.Println("accept failed, err:", err)
			continue
		}
		go process(conn)
	}
}
```

客户端代码如下

```go
package main

import (
	"fmt"
	"net"
)

// socket_stick/client/main.go

func main() {
	conn, err := net.Dial("tcp", "127.0.0.1:30000")
	if err != nil {
		fmt.Println("dial failed, err", err)
		return
	}
	defer conn.Close()
	for i := 0; i < 20; i++ {
		msg := `Hello, Hello. How are you?`
		conn.Write([]byte(msg))
	}
}
```

将上面的代码保存后，分别编译。先启动服务端再启动客户端，可以看到服务端输出结果如下：

```
收到client发来的数据： Hello, Hello. How are you?
收到client发来的数据： Hello, Hello. How are you?Hello, Hello. How are you?Hello, Hello. How are you?Hello, Hello. How are you?
收到client发来的数据： Hello, Hello. How are you?
收到client发来的数据： Hello, Hello. How are you?
收到client发来的数据： Hello, Hello. How are you?
```

客户端分 20 次发送的数据，在服务端并没有成功的输出 20 次，而是 **多条数据“粘”到了一起**。

### 原因

> [!question] 为什么会出现粘包?
> 
> 主要原因就是 **TCP 数据传递模式是流模式**，在保持长连接的时候可以进行多次的收和发。

**“粘包"** 可可以发生在发送端，也可以发生在接收端

> [!tip] 发送端粘包：由拥塞避免算法（例如，Nagle 算法）
>`Nagle` 算法是一种改善网络传输效率的算法。简单来说就是当我们提交一段数据给 TCP 发送时，**TCP 并不立刻发送此段数据**，而是等待一小段时间看看在等待期间是否还有要发送的数据，若有则 **会一次把这多段数据发送出去**


> [!tip] 接收端粘包：接收数据不及时
> 
> TCP 会把接收到的数据存在自己的缓冲区中，然后通知应用层取数据。当应用层由于某些原因不能及时的把 TCP 的数据取出来，就会造成 TCP 缓冲区中存放了几段数据

### 解决方案

出现"粘包"的关键在于 **_接收方不确定将要传输的数据包的大小_**，因此我们可以对数据包进行封包和拆包的操作

**封包**：封包就是 **_给一段数据加上包头_**，这样一来数据包就分为包头和包体两部分内容了(过滤非法包时封包会加入"包尾"内容)。包头部分的长度是固定的，并且它存储了包体的长度，根据包头长度固定以及包头中含有包体长度的变量就能正确的拆分出一个完整的数据包

我们可以自己定义一个协议，比如数据包的前 $4$ 个字节为包头，里面存储的是发送的数据的长度

```go
// socket_stick/proto/proto.go
package proto

import (
	"bufio"
	"bytes"
	"encoding/binary"
)

// Encode 将消息编码
func Encode(message string) ([]byte, error) {
	// 读取消息的长度，转换成int32类型（占4个字节）
	var length = int32(len(message))
	var pkg = new(bytes.Buffer)
	// 写入消息头
	err := binary.Write(pkg, binary.LittleEndian, length)
	if err != nil {
		return nil, err
	}
	// 写入消息实体
	err = binary.Write(pkg, binary.LittleEndian, []byte(message))
	if err != nil {
		return nil, err
	}
	return pkg.Bytes(), nil
}

// Decode 解码消息
func Decode(reader *bufio.Reader) (string, error) {
	// 读取消息的长度
	lengthByte, _ := reader.Peek(4) // 读取前4个字节的数据
	lengthBuff := bytes.NewBuffer(lengthByte)
	var length int32
	err := binary.Read(lengthBuff, binary.LittleEndian, &length)
	if err != nil {
		return "", err
	}
	// Buffered返回缓冲中现有的可读取的字节数。
	if int32(reader.Buffered()) < length+4 {
		return "", err
	}

	// 读取真正的消息数据
	pack := make([]byte, int(4+length))
	_, err = reader.Read(pack)
	if err != nil {
		return "", err
	}
	return string(pack[4:]), nil
}
```

接下来在服务端和客户端分别使用上面定义的`proto`包的`Decode`和`Encode`函数处理数据

服务端

```go
// socket_stick/server2/main.go

func process(conn net.Conn) {
	defer conn.Close()
	reader := bufio.NewReader(conn)
	for {
		msg, err := proto.Decode(reader)
		if err == io.EOF {
			return
		}
		if err != nil {
			fmt.Println("decode msg failed, err:", err)
			return
		}
		fmt.Println("收到client发来的数据：", msg)
	}
}

func main() {

	listen, err := net.Listen("tcp", "127.0.0.1:30000")
	if err != nil {
		fmt.Println("listen failed, err:", err)
		return
	}
	defer listen.Close()
	for {
		conn, err := listen.Accept()
		if err != nil {
			fmt.Println("accept failed, err:", err)
			continue
		}
		go process(conn)
	}
}
```

客户端

```go
// socket_stick/client2/main.go

func main() {
	conn, err := net.Dial("tcp", "127.0.0.1:30000")
	if err != nil {
		fmt.Println("dial failed, err", err)
		return
	}
	defer conn.Close()
	for i := 0; i < 20; i++ {
		msg := `Hello, Hello. How are you?`
		data, err := proto.Encode(msg)
		if err != nil {
			fmt.Println("encode msg failed, err:", err)
			return
		}
		conn.Write(data)
	}
}
```

## 非阻塞 IO

Go 的网络 IO 的底层使用了 **非阻塞 IO** 和 **IO 多路复用**。但是在编程上，Go 又将其做成了阻塞 IO 的模样

为了让网络 IO 变回非阻塞的模式，可以设置超时时间

```go
// SetReadDeadline()
conn.SetReadDeadline(time.Now().Add(time.Second)) // 每次 Read 最长等待 1s

// SetWriteDeadline()
conn.SetWriteDeadline(time.Now().Add(time.Second)) // 每次 Write 最长等待 1s

// SetDeadline
conn.SetDeadline(time.Now().Add(time.Second)) // 每次 Read 或 Write 最长等待时间
```

非阻塞 IO 的客户端

```go
package main

import (
	"fmt"
	"io"
	"log"
	"net"
	"os"
	"time"
)

func catchErr(err error) {
	if err != nil {
		log.Fatal(err)
	}
}

func main() {
	remoteTCPAddr, err := net.ResolveTCPAddr("tcp", "localhost:8080")
	catchErr(err)

	conn, err := net.DialTCP("tcp", nil, remoteTCPAddr)
	catchErr(err)

	done := make(chan struct{})
	// 启动一个 Goroutine 用于处理服务端的返回
	go func(conn net.Conn) {
		buffer := make([]byte, 4096)

		for {
			conn.SetReadDeadline(time.Now().Add(time.Second)) // 设置 Read 超时底线。如果 1s 后还未读取成功，则返回错误

			n, err := conn.Read(buffer)

			if err != nil {
				// 检查是否是超时错误
				if _, ok := err.(*net.OpError); !ok {
					// 不是超时错误
					done <- struct{}{} // 退出了
					return
				}
				log.Println("超时错误")
				continue // 是超时错误开启下一次读取
			}

			// 读取成功
			fmt.Printf("服务器 %v 返回的消息为: %s\n", conn.RemoteAddr(), buffer[:n])

		}

	}(conn)

	// main Goroutine 处理向客户端发送数据
	_, err = io.Copy(conn, os.Stdin)
	if err != nil {
		log.Println(err)
	}
	conn.Close()
	<-done
}
```
