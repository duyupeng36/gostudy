# 网络 IO 模型

目前为止，我们的程序中使用的 IO 模型都是单个进程 **每次 _只在一个文件描述符_ 上执行 IO 操作**，**每次 IO 系统调用都会 _阻塞_ 直到完成数据传输**

比如，在进程间通信中使用 **管道**，当从一个管道中读取数据时，如果管道中恰好没有数据，那么通常 `read()` 会阻塞。而如果管道中没有足够的空间保存待写入的数据时，`write()` 也会阻塞。当在其他类型的文件如 **FIFO** 和 **套接字** 上执行 I/O 操作时，也会出现相似的行为

> [!tip] 
> 当在某个文件描述符上读取数据时，如果无法该文件描述符上读取数据，那么 `read()` 系统调用就会阻塞
> 
> 当在某个文件描述符上写入数据时，如果文件描述关联文件没有空间写入时，`write()` 系统调用也会阻塞

> [!attention] 磁盘文件是个特例
> 内核采用缓冲区 `cache` 来加速磁盘 I/O 请求。因而一旦请求的数据传输到内核的缓冲区 `cache`，对磁盘的 `write()` 操作将立刻返回，而不用等到将数据实际写入磁盘后才返回（除非在打开文件时指定了 `O_SYNC` 标志）
> 
> 与之对应的是，`read()` 调用将数据从内核缓冲区 `cache` 移动到用户的缓冲区中，如果请求的数据不在内核缓冲区 `cache`，那么内核就会让进程休眠，同时执行对磁盘的读操作
> 

对于许多应用来说，传统的 **阻塞式 I/O 模型** 已经足够了，但这不代表所有的应用都能得到满足。特别的，有些应用需要处理以下某项任务，或者两者都需要兼顾
+ 如果可能的话，**_以非阻塞的方式检查文件描述符上是否可进行 I/O 操作_**
+ **_同时检查多个文件描述符_**，看它们中的任何一个是否可以执行 I/O 操作

目前为止，我们使用 **多进程** 或 **多线程** 技术来部分满足这两个需求

## IO 模型

IO 过程分两阶段
1. **数据准备阶段**。从设备读取数据到内核空间的缓冲区（淘米，把米放饭锅里煮饭） 
2. **从内核空间复制** 回用户空间进程缓冲区阶段（盛饭，从内核这个饭锅里面把饭装到碗里来）

> [!tip] IO 两个阶段
> 
> **数据准备阶段**：将数据从设备搬运到内核空间中的缓冲区
> 
> **进程获取数据阶段**：从内核缓冲区复制到用户空间的缓冲区

### 同步与异步

**同步**：不达目的誓不罢休，如果是函数调用，函数必须返回我要的最终结果。例如 `t=add(4，5)`，在 `add` 计算的结果 `9` 返回之前，进程不在向下执行，即 **阻塞**

```python
def add(x，y):
	time.sleep(3600)
	return x+y
```

**异步**：我们不关心最终结果，但是我不能表现出来，可以不取走最终结果。例如，`f= add(4，5)`，`add` 假设可能需要 `3600s`，调用时 **立即返回** 一个值赋值给 `f`，但 `f` 不是最终结果 `9` ，而是 **_获取最终结果的凭据_**

> [!tip] 异步任务的结果获取
>  当异步任务执行完成后，内核通知应用进程，通过凭证获取最终结果

### 阻塞 IO

**阻塞 IO** 是指在执行 IO 操作时，**调用线程会被阻塞**，等待 **IO 操作完成后才能继续执行** 后续代码。此时，**线程处于等待状态**，无法执行其他任务

> [!tip]
> 当一个阻塞 IO 操作（如读取文件或从网络接收数据）被发起时，调用线程会暂停，直到数据读取或写入操作完成。操作完成后，线程才会继续执行后续代码

![[Drawing 2024-08-03 18.16.56.excalidraw|900]]

> [!tip] 阻塞 IO 特点
> 
> 进程等待（阻塞），直到读写完成。（全程等待

### 非阻塞 IO 

**非阻塞 I/O** 可以让我们 **周期性** 地检查（“轮询”）某个文件描述符上是否可执行 I/O 操作

> [!tip]
> 比如，我们可以让一个输入文件描述符成为非阻塞式的，然后周期性地执行非阻塞式的读操作

如果我们需要 **同时检查多个文件描述符**，那么就需要将它们 **都设为非阻塞**，然后依次对它们 **轮询**

> [!tip]
> 在一个紧凑的循环中 **做轮询就是在浪费 CPU**

下图描述了 **非阻塞 IO** 的轮询过程

![[Drawing 2024-08-03 18.35.05.excalidraw|900]]

下面示例是套接字的非阻塞IO

```python
import socket
import datetime

# 设置tcp服务端套接字
server = socket.socket()

# 将套接字设置为非阻塞套接字
server.setblocking(False)

# 绑定ip和端口
server.bind(('127.0.0.1', 8080))

# 监听客户端连接
server.listen()

# 保存连接生成的套接字
connection_list = []

while True:
    try:
        # 接收连接请求。
        connection, address = server.accept()  # 此处不会等待连接，如果连接未就绪，就会除法 BlockingIOError
        print(f'客户端{address}建立了连接')
        # 将生成的对等套接字也设置为非阻塞套接字
        connection.setblocking(False)
        # 将用于和客户端通信的对等套接字保存到列表
        connection_list.append(connection)
    except BlockingIOError as e:
        print(f"{datetime.datetime.now()}没有连接")

    # 复制一份列表
    to_handle = [connection for connection in connection_list]
    # 遍历列表为每一个客户端服务
    for connection in to_handle:
        try:
            # 接收客户端发来的消息
            recv_data = connection.recv(1024)  # 非阻塞，没有数据则会触发 BlockingIOError
            # 判断消息是否为空
            if recv_data:
                # 打印消息
                print(f'来自{connection.getpeername()}的消息：{recv_data.decode()}')
                # 将消息返回给客户端
                connection.send(recv_data)
            else:
                print(f'客户端{connection.getpeername()}关闭了连接')
                # 关闭连接
                connection.close()
                # 将已处理的客户端从列表中移除
                connection_list.remove(connection)
        except BlockingIOError as e:
            print(f"{connection.getpeername()} 没有发送消息")
```

### IO 多路复用（事件驱动 IO）

**IO 多路复用** 允许我们同时检查多个文件描述符，看其中任意一个是否可执行 I/O 操作。有两个功能几乎相同的 **系统调用** 来执行 I/O 多路复用操作：`select()` 和 `poll()`

![[Pasted image 20240803191749.png|900]]

可以在 **普通文件**、**管道**、**FIFO**、**套接字** 以及一些其他类型的 **字符型设备** 上使用  `select()` 和 `poll()` 来检查文件描述符。这两个系统调用都 _允许进程要么 **一直等待文件描述符成为就绪态**，要么在调用中 **指定一个超时时间**_

> [!tip]
> 同时监控多个IO，称为多路IO，哪怕只有一路准备好了，就不需要等了就可以开始处理这一路的数据。这种方式提高了同时处理 IO 的能力

> [!tip] 
> `select` 几乎所有操作系统平台都支持，`poll` 是对的 `select` 的升级。
> 
> `epoll`，Linux 系统内核 2.5+开始支持，对 `select` 和 `poll` 的增强，在监视的基础上，增加回调机制
> 
> BSD、Mac平台有 `kqueue`，Windows有 `iocp`

以 `select` 为例，将关注的 IO 操作告诉 `select` 函数并调用，进程阻塞，内核“监视” `select` 关注的文件描述符 `fd`，被关注的任何一个 `fd` 对应的 IO 准备好了数据，`select` 返回。再使用 `read` 将数据制到用户进程

`Epoll` 与 `select` 相比，解决了 `select` 监听 `fd` 的限制和 `O(n)` 遍历效率问题，提供回调机制等，效率更高

>[!attention]  IO 多路复用中，请使用非阻塞IO


```python

import socket
import selectors

def recv(connection: socket.socket):
    try:
        data = connection.recv(1024)
        if not data:
            print(connection.getpeername(), " 断开连接")
            selector.unregister(connection)
            return None
        connection.send(data)
    except Exception as e:
        print(e)
        selector.unregister(connection)

def accept(server: socket.socket):
    connection, address = server.accept()
    print(address, " 建立连接")
    connection.setblocking(False)
    selector.register(connection, selectors.EVENT_READ, recv)



if __name__ == "__main__":
    server = socket.socket(socket.AF_INET, socket.SOCK_STREAM, 0)
    server.setblocking(False)
    server.bind(("172.17.171.86", 8080))
    server.listen(1024)  # 等待连接队列的大小

    selector = selectors.DefaultSelector()
    print(selector)

    selector.register(server, selectors.EVENT_READ, accept)

    while True:
         for key, event in selector.select():
             callback = key.data   # 注册的函数
             fd = key.fileobj   # 注册的文件描述符
             # key.events  # 事件
             callback(fd)
```

