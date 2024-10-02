
# SOCKET

`SOCKET` 是一种 `IPC` 方法，它 **允许位于同一主机**（计算机）或 **使用网络连接起来的不同主机** 上的 **应用程序之间交换数据**

在一个典型的客户端/服务器场景中，应用程序使用 `socket` 进行通信的方式如下
- 各个应用程序创建一个 `socket`。`socket` 是一个允许通信的“设备”，两个应用程序都需要用到它
- 服务器将自己的 `socket` 绑定到一个众所周知的地址（名称）上使得客户端能够定位到它的位置

使用 `socket()` 系统调用能够创建一个 `socket`，它返回一个用来在后续系统调用中引用该 `socket` 的文件描述符

```c
fd = socket(domain, type, protocol);
```

`socket` 存在于一个 **通信 domain** 中，它确定：
- 识别出一个 `socket` 的方法（即 `socket` “地址”的格式）
- **通信范围**
	- 是在位于 **同一主机上的应用程序之间**
	- 还是在位于 **使用一个网络连接起来的不同主机上的应用程序之间**）

> [!tip] 现代操作系统支持的 DOMAIN 
> 
> 现代操作系统至少支持下列 **domain**
> + **UNIX  domain** (`AF_UNIX`) 允许在同一主机上的应用程序之间进行通信
> 	+ `POSIX.1g` 使用名称 `AF_LOCAL` 作为 `AF_UNIX` 的同义词
> + **IPv4  domain**(`AF_INET`) 允许在使用因特网协议第 4 版（IPv4）网络连接起来的主机上的应用程序之间进行通信
> + **IPv6  domain**(`AF_INET6`) 允许在使用因特网协议第 6 版（IPv6）网络连接起来的主机上的应用程序之间进行通信。尽管 IPv6 被设计成了 IPv4 接任者，但目前后一种协议仍然是使用最广的协议

 `socket` 实现都至少提供了两种 `socket`：**流** 和 **数据报**。这两种 `socket` 类型在 `UNIX` 和`Internet domain` 中都得到了支持

> [!tip] **流 socket (SOCK_STREAM)** 提供了一个 _可靠_ 的 _双向_ 的 _字节流_ 通信信道
> 
> **可靠的**：表示可以 **保证发送者传输的数据会完整无缺地到达接收应用程序**
> 
> **双向的**：表示数据可以在两个 `socket` 之间的任意方向上传输
> 
> **字节流**：表示**不存在消息边界的概念**

> [!tip] **数据报  socket（SOCK_DGRAM）** 允许数据以被称为数据报的消息的形式进行交换
> 在数据报 socket 中，**消息边界得到了保留**，但 **数据传输是不可靠的**。消息的到达可能是无序的、重复的或者根本就无法到达
> 
> 数据报 `socket` 是更一般的无连接 `socket` 概念的一个示例。与流 `socket` 不同，**一个数据报 `socket` 在使用时无需与另一个 `socket` 连接**。

## SOCKET 相关的系统调用

关键的 `socket` 系统调用包括以下几种
- `socket()` 系统调用创建一个新 `socket`
- `bind()` 系统调用将一个 `socket` 绑定到一个地址上。通常，服务器需要使用这个调用来将其 `socket` 绑定到一个众所周知的地址上使得客户端能够定位到该 `socket` 上
- `listen()` 系统调用允许一个流 `socket` 接受来自其他 `socket` 的接入连接
- `accept()` 系统调用在一个监听流 `socket` 上接受来自一个对等应用程序的连接，并可选地返回对等 `socket` 的地址
- `connect()` 系统调用建立与另一个 `socket` 之间的连接

`socket I/O` 可以使用传统的 `read()` 和 `write()` 系统调用或使用一组 `socket` 特有的系统调用（如 `send()`、`recv()`、`sendto()` 以及 `recvfrom()`）来完成。在默认情况下，这些系统调用在 **I/O 操作无法被立即完成时会阻塞**。通过使用 `fcntl() F_SETFL` 操作来启用 `O_NONBLOCK` 打开文件状态标记可以执行 **非阻塞 I/O**

## 客户端/服务器示例(TCP 回显服务器)

服务端的代码

```c
#include <stdio.h>
#include <string.h>
#include <ctype.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <unistd.h>
#include <arpa/inet.h>


int main(int argc, char *argv[]) {
    
    if (argc < 2) {
        fprintf(stderr, "Usage: %s host port\n", argv[0]);
        return -1;
    }

    setvbuf(stdout, NULL, _IONBF, 0);

    printf("服务端: 创还能套接字\n");
    int socket_fd;
    if ((socket_fd = socket(AF_INET, SOCK_STREAM, 0)) == -1) {
        perror("SOCKET");
        return -1;
    }

    printf("服务端：准备地址结构\n");
    struct sockaddr_in address; 
    memset(&address, 0, sizeof(address));
    address.sin_family = AF_INET;  // IPv4 
    address.sin_port = htons(argv[2]);  // 端口，需要将host字节序转为网络字节序
    // address.sin_addr.s_addr = INADDR_ANY;  // 0.0.0.0 

    // 绑定地址
    if (inet_pton(AF_INET, argv[1], &address.sin_addr.s_addr) == -1) {
        perror("INET_PTON");
        return -1;
    }

    printf("服务端: 绑定地址结构\n");
    if (bind(socket_fd, (struct sockaddr *)&address, sizeof(address)) == -1) {
        perror("BIND");
        return -1;
    }

    printf("服务端: 监听套接字\n");
    if(listen(socket_fd, SOMAXCONN) == -1) {
        perror("LISTEN");
        return -1;
    }

    printf("服务端: 等待接收客户端的链接\n");
    struct sockaddr_in cli; // 保存客户端地址

    int cli_fd = accept(socket_fd, (struct sockaddr *)&cli, sizeof(cli));
    if (cli_fd == -1) {
        perror("ACCEPT");
        return -1;
    }

    
    printf("服务器: 接收了 %s:%hu 的连接\n", inet_ntoa(cli.sin_addr), ntohs(cli.sin_port));

    for (;;) {
        char buf[1024];
        ssize_t size;

        if ((size = read(cli_fd, buf, 1024)) == -1) {
            perror("READ");
            return -1;
        }

        if (size == 0) {
            fprintf(stderr, "服务端: 客户端关闭连接\n");
            close(cli_fd);  // 关闭套接字
            return -1;
        }

        printf("服务端接收到: ");
        for(int i =0; i < size; i++) {
            putc(buf[i], stdout);
            buf[i] = toupper(buf[i]);
        }
        putc('\n', stdout);

        write(cli_fd, buf, size);

    }

    close(socket_fd);
}
```

客户端

```c
#include <stdio.h>
#include <string.h>
#include <stdlib.h>
#include <ctype.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <unistd.h>
#include <arpa/inet.h>


int main(int argc, char *argv[]) {
    
    if (argc < 2) {
        fprintf(stderr, "Usage: %s host port\n", argv[0]);
        return -1;
    }

    setvbuf(stdout, NULL, _IONBF, 0);

    printf("客户端: 创还能套接字\n");
    int socket_fd;
    if ((socket_fd = socket(AF_INET, SOCK_STREAM, 0)) == -1) {
        perror("SOCKET");
        return -1;
    }

    printf("客户端：准备地址结构\n");
    struct sockaddr_in address; 
    memset(&address, 0, sizeof(address));
    address.sin_family = AF_INET;  // IPv4 
    address.sin_port = htons(atoi(argv[2]));  // 端口，需要将host字节序转为网络字节序
    // address.sin_addr.s_addr = INADDR_ANY;  // 0.0.0.0 

    // 绑定地址
    if (inet_pton(AF_INET, argv[1], &address.sin_addr.s_addr) == -1) {
        perror("INET_PTON");
        return -1;
    }

    printf("客户端: 连接服务端\n");
    if (connect(socket_fd, (struct sockaddr *)&address, (socklen_t)sizeof(address)) == -1) {
        perror("CONNECT");
        return -1;
    }

    // 向服务端发送消息
    for (;;) {
        char buf[1024];
        ssize_t size;
        printf(">>> ");
        fgets(buf, 1024, stdin);

        if (strcmp(buf, "q\n") == -1) {
            printf("结束\n");
            break;
        }

        // 向服务器发送消息
        if ((size = write(socket_fd, buf, strlen(buf))) == -1) {
            perror("WRITE");
            return -1;
        }

        // 接受服务端返回的消息
        read(socket_fd, buf, size);
        printf("服务端返回的消息: ");
        puts(buf);
    }

    close(socket_fd);
}
```

基于进程的并发服务器：每接受一个客户端的连接就创建一个进程处理与客户端的通信

```c

#include <stdio.h>
#include <string.h>
#include <stdlib.h>
#include <stdbool.h>
#include <errno.h>
#include <ctype.h>

#include <sys/types.h>
#include <sys/socket.h>
#include <unistd.h>
#include <sys/wait.h>
#include <arpa/inet.h>

#include <signal.h>
// 信号处理函数，回收子进程
static void sigchld_handler(int sig) {
    for (;;) {
        pid_t pid = waitpid(-1, NULL, WNOHANG);
        if (pid == -1) {
            if(errno == ECHILD) {
                printf("服务器：子进程完全结束\n");
                break;
            } else {
                perror("WAITPID");
                return;
            }
        } else if (pid == 0) {
            printf("服务器：子进程正在运行\n");
            break;
        } else {
            printf("服务器: 回收子进程 %d\n", pid);
        }
    }
}

int main(int argc, char *argv[])
{

    if (argc < 2)
    {
        fprintf(stderr, "Usage: %s host port\n", argv[0]);
        return -1;
    }
    setvbuf(stdout, NULL, _IONBF, 0);
    
    printf("服务端: 注册信号处理函数\n");

    struct sigaction sa;
    sa.sa_handler = sigchld_handler;
    sa.sa_flags = SA_RESTART;
    sigemptyset(&sa.sa_mask);
    if(sigaction(SIGCHLD, &sa, NULL) == -1)
    {
        perror("sigaction");
        return -1;
    }

    printf("服务端: 创还能套接字\n");
    int socket_fd;
    if ((socket_fd = socket(AF_INET, SOCK_STREAM, 0)) == -1)
    {
        perror("SOCKET");
        return -1;
    }

    printf("服务端：准备地址结构\n");
    struct sockaddr_in address;
    memset(&address, 0, sizeof(address));
    address.sin_family = AF_INET;            // IPv4
    address.sin_port = htons(atoi(argv[2])); // 端口，需要将host字节序转为网络字节序
    // address.sin_addr.s_addr = INADDR_ANY;  // 0.0.0.0

    // 绑定地址
    if (inet_pton(AF_INET, argv[1], &address.sin_addr.s_addr) == -1)
    {
        perror("INET_PTON");
        return -1;
    }

    printf("服务端: 绑定地址结构\n");
    if (bind(socket_fd, (struct sockaddr *)&address, sizeof(address)) == -1)
    {
        perror("BIND");
        return -1;
    }

    printf("服务端: 监听套接字\n");
    if (listen(socket_fd, SOMAXCONN) == -1)
    {
        perror("LISTEN");
        return -1;
    }

    while (true)
    {
        printf("服务端: 等待接收客户端的链接\n");
        struct sockaddr_in cli; // 保存客户端地址
        socklen_t len = sizeof(cli);
        int cli_fd = accept(socket_fd, (struct sockaddr *)&cli, &len);
        if (cli_fd == -1)
        {
            perror("ACCEPT");
            return -1;
        }

        printf("服务器: 接收了 %s:%hu 的连接\n", inet_ntoa(cli.sin_addr), ntohs(cli.sin_port));
        printf("服务端：创建子进程用处理该通信");

        switch (fork())
        {
        case -1:
            perror("FORK");
            break;
        case 0:
            // 子进程不等待连接，所以关闭监听套接字
            close(socket_fd);
            for (;;)
            {
                char buf[1024];
                ssize_t size;

                if ((size = read(cli_fd, buf, 1024)) == -1)
                {
                    perror("READ");
                    return -1;
                }

                if (size == 0)
                {
                    fprintf(stderr, "服务端: 客户端关闭连接\n");
                    close(cli_fd); // 关闭套接字
                    return -1;
                }

                printf("服务端接收到: ");
                for (int i = 0; i < size; i++)
                {
                    putc(buf[i], stdout);
                    buf[i] = toupper(buf[i]);
                }
                putc('\n', stdout);

                write(cli_fd, buf, size);
            }

        default:
            // 父进程不在需要管理通信，直接关闭
            close(cli_fd);
        }
    }

    close(socket_fd);
}
```

## Go TCP 编程

Go 将 Socket 隐藏在了底层，通过 `net.Listen*` 系列的函数开启不同的 Socket。下面我们来看使用 Go 编写 TCP 服务端

```go
package main

import (
	"fmt"
	"log"
	"net"
)

func main() {

	fmt.Println("服务端: 解析TCP地址")
	laddr, err := net.ResolveTCPAddr("tcp", "127.0.0.1:8080")
	if err != nil {
		log.Panicf("ResolveTCPAddr: %v\n", err)
	}
	fmt.Printf("服务端: 创建socket, 绑定地址并开启监听")
	// 创建 socket bind listen
	server, err := net.ListenTCP("tcp", laddr)

	defer server.Close()

	if err != nil {
		log.Panicf("ListenTCP: %v\n", err)
	}

	for {
		fmt.Println("服务端: 等待客户端连接")
		// 接受客户发起的连接，返回用于通信的对等套接字
		conn, err := server.Accept()
		if err != nil {
			log.Panicf("Accept: %v\n", err)
		}
		fmt.Printf("服务端: 与客户端 %v 建立连接\n", conn.RemoteAddr())

		// 用户空间中的缓冲区
		var buf = make([]byte, 1024)

		n, err := conn.Read(buf)

		if err != nil {
			log.Panicf("Accept: %v\n", err)
		}
		fmt.Printf("客户端发来信息: %s\n", buf[:n])
		conn.Write(buf[:n]) // 返回
		conn.Close()        // 关闭连接
	}
}
```

这个例程在接受客户端一次消息后就断开了与客户端的连接，我们想让服务端可以持续给这个连接服务，并在客户端关闭连接后才关闭连接套接字

```go
package main

import (
	"fmt"
	"log"
	"net"
)

func handleConnection(conn net.Conn) {
	defer func() {
		err := conn.Close()
		if err != nil {
			log.Fatalf("关闭连接失败: %v\n", err)
		}
	}()

	// 用户空间中的缓冲区
	var buf = make([]byte, 1024)

	for {
		n, err := conn.Read(buf)

		if n == 0 {
			fmt.Printf("服务端：客户端主动断开\n")
			break
		}

		if err != nil {
			fmt.Printf("服务端：读取客户端消息失败：%v\n", err)
			break
		}

		fmt.Printf("客户端发来信息: %s\n", buf[:n])
		conn.Write(buf[:n]) // 返回
	}
}

func main() {

	fmt.Println("服务端: 解析TCP地址")
	laddr, err := net.ResolveTCPAddr("tcp", "127.0.0.1:8080")
	if err != nil {
		log.Panicf("ResolveTCPAddr: %v\n", err)
	}

	fmt.Printf("服务端: 创建socket, 绑定地址并开启监听")
	// 创建 socket bind listen
	server, err := net.ListenTCP("tcp", laddr)

	if err != nil {
		log.Panicf("服务单监听 TCP 连接失败: %v\n", err)
	}

	defer func(server net.Listener) {
		err := server.Close()
		if err != nil {
			log.Fatalf("服务端: 关闭监听套接字失败: %v\n", err)
		}
	}(server)

	for {
		fmt.Println("服务端: 等待客户端连接")
		// 接受客户发起的连接，返回用于通信的对等套接字
		conn, err := server.Accept()
		if err != nil {
			log.Printf("服务端接受客户端的连接失败: %v\n", err)
		}

		fmt.Printf("服务端: 与客户端 %v 建立连接\n", conn.RemoteAddr())

		handleConnection(conn)
	}
}
```

现在，这个服务器依旧只能服务一个连接，只有当前连接结束之后，才能服务下一个连接
