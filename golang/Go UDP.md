# Go UDP

**UDP协议**（User Datagram Protocol）中文名称是 **用户数据报协议**，是 OSI（Open System Interconnection，开放式系统互联）参考模型中一种 **无连接** 的传输层协议，不需要建立连接就能直接进行数据发送和接收，属于不可靠的、没有时序的通信，但是 UDP 协议的实时性比较好，通常用于视频直播相关领域

##  服务端

![[Pasted image 20240808193127.png|900]]

由于 **`UDP` 是“无连接”的**，所以，服务器端 **不需要额外创建监听套接字**， 只需要指定好 `IP` 和`port`，然后 **监听该地址**，等待客户端与之建立连接，即可通信

- 创建`UDPAddr`: `func ResolveUDPAddr(network, address string) (*UDPAddr, error)`
- 创建用于通信的`socket`: `func ListencUDP(network string, laddr *UDPAddr) (*UDPConn, error)`
- 接收 udp 数据: `func (c *UDPConn) ReadFromUDP(b []byte) (int, *UDPAddr, error)`
- 写出数据到 udp: `func (c *UDPConn) WriteToUDP(b []byte, addr *UDPAddr) (int, error)`

```go
// UDP/server/main.go

// UDP server端
func main() {
	listen, err := net.ListenUDP("udp", &net.UDPAddr{
		IP:   net.IPv4(0, 0, 0, 0),
		Port: 30000,
	})
	if err != nil {
		fmt.Println("listen failed, err:", err)
		return
	}
	defer listen.Close()
	for {
		var data [1024]byte
		n, addr, err := listen.ReadFromUDP(data[:]) // 接收数据
		if err != nil {
			fmt.Println("read udp failed, err:", err)
			continue
		}
		fmt.Printf("data:%v addr:%v count:%v\n", string(data[:n]), addr, n)
		_, err = listen.WriteToUDP(data[:n], addr) // 发送数据
		if err != nil {
			fmt.Println("write to udp failed, err:", err)
			continue
		}
	}
}
```

## UDP客户端

使用Go语言的`net`包实现的 UDP 客户端代码与 TCP 客户端代码相似

```go
// UDP 客户端
func main() {
	socket, err := net.DialUDP("udp", nil, &net.UDPAddr{
		IP:   net.IPv4(0, 0, 0, 0),
		Port: 30000,
	})
	if err != nil {
		fmt.Println("连接服务端失败，err:", err)
		return
	}
	defer socket.Close()
	sendData := []byte("Hello server")
	_, err = socket.Write(sendData) // 发送数据
	if err != nil {
		fmt.Println("发送数据失败，err:", err)
		return
	}
	data := make([]byte, 4096)
	n, remoteAddr, err := socket.ReadFromUDP(data) // 接收数据
	if err != nil {
		fmt.Println("接收数据失败，err:", err)
		return
	}
	fmt.Printf("recv:%v addr:%v count:%v\n", string(data[:n]), remoteAddr, n)
}
```


