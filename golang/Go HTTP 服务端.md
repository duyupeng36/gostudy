# HTTP 服务端


在 `net/http` 包中关于 HTTP 服务器程序 涉及两个主要类型  `Server` 和 `ServerMux`

> [!tip] `Server` 类型：定义了运行 HTTP 服务的参数。 零值是有效的配置

> [!tip]  `ServeMux` 是一个 HTTP **请求多路复用器**，它会 将每个传入 _请求的 URL_ 与已注册 _模式_ 的列表进行匹配，并 _调用_ 与 URL 最匹配的模式的 _处理程序_

## Server 类型

`Server` 定义了 **运行 HTTP 服务器的参数**。`Server` 的零值是有效的配置

```go
type Server struct {
	....
}
```

下表列出来 Server 结构体中的成员

|         成员          | 描述                                                     |
| :-----------------: | :----------------------------------------------------- |
|       `Addr`        | 服务器监听的TCP地址，形式为 `"host:port"`                          |
|      `Handler`      | 处理器，实现了 `net.Handler` 结构的类型实例                          |
|     `TLSConfig`     | TLS 配置                                                 |
|    `ReadTimeout`    | 读取请求的最长时间                                              |
| `ReadHeaderTimeout` | 读取请求头的最长时间，如果为 $0$ 使用 `ReadTimeout`                    |
|   `WriteTimeout`    | 写入响应的最长时间                                              |
|    `IdleTimeout`    | `Keep-Alive` 模式下等待下一个请求的最大时间，如果为 $0$ 使用 `WriteTimeout` |
|  `MaxHeaderBytes`   | 请求头的最长字节数                                              |
|     `ConnState`     | 回调函数，在一个与客户端的连接改变状态时被调用。                               |
|     `ErrorLog`      | 错误处理器                                                  |
|    `BaseContext`    | 服务器上传入请求的基本上下文。默认为 `context.Background()`              |
|    `ConnContext`    | 连接的上下文                                                 |

该类型具有如下几个方法

```go
// Close 立即关闭所有活动的 net.Listeners 和处于 StateNew、StateActive 或 StateIdle 状态的连接
func (srv *Server) Close() error

// ListenAndServe 侦听 TCP 网络地址 srv.Addr，然后调用 Serve 来处理传入连接的请求。接受的连接配置为启用TCP保持活动。
func (srv *Server) ListenAndServe() error

// ListenAndServeTLS侦听TCP网络地址srv.Addr，然后调用ServeTLS来处理传入的TLS连接上的请求。接受的连接被配置为启用TCP保持活动状态。
func (srv *Server) ListenAndServeTLS(certFile, keyFile string) error

// RegisterOnShutdown 函数注册在 Server.Shutdown 上调用的函数。这可以用于优雅地关闭已经经历了ALPN协议升级或被劫持的连接。这个函数应该启动特定协议的优雅关闭，但不应该等待关闭完成。
func (srv *Server) RegisterOnShutdown(f func())

// Serve 在监听器 l 上接受传入连接，为每个连接创建一个新的服务 goroutine。服务 goroutines 读取请求，然后调用 srv.Handler 来回复
func (srv *Server) Serve(l net.Listener) error

// ServeTLS 在监听器l上接受传入连接，为每个连接创建一个新的服务goroutine。服务goroutines执行TLS设置，然后读取请求，并调用srv.Handler来回复请求。
func (srv *Server) ServeTLS(l net.Listener, certFile, keyFile string) error

// SetKeepAlivesEnabled 控制是否启用 HTTP keep-alive。默认情况下，keep-alive 是一直启用的。只有资源非常受限的环境或正在关闭的服务器才应该禁用它们。
func (srv *Server) SetKeepAlivesEnabled(v bool)

// Shutdown 是一个优雅地关闭服务器的操作，它不会中断任何活动连接。具体而言，Shutdown 首先关闭所有打开的监听器，然后关闭所有空闲连接，最后无限期地等待连接返回空闲状态，然后关闭
func (srv *Server) Shutdown(ctx context.Context) error
```

### Handler 接口

```go
type Handler interface {
	ServeHTTP(ResponseWriter, *Request)
}
```

实现了 `Handler` 接口的对象可以注册到 HTTP 服务端，为特定的路径及其子路径提供服务。

`ServeHTTP` 应该将回复的头域和数据写入 `ResponseWriter` 接口然后返回。返回标志着该请求已经结束，HTTP 服务端可以转移向该连接上的下一个请求

> [!tip] ServerHTTP 用于分派请求的处理函数

## ServerMux 类型

`ServeMux` 是一个 HTTP 请求多路复用器，它会 将每个请求传入的 _URL_ 与 _已注册模式_ 的列表进行匹配，并 _调用_ 与URL最匹配的模式的 _处理程序_

> [!tip] ServerMux 就是实现了 Handler 接口的处理器

```go
type ServeMux struct {
	mu       sync.RWMutex
	tree     routingNode
	index    routingIndex
	patterns []*pattern  // TODO(jba): 尽可能移除
	mux121   serveMux121 // 仅当GODEBUG=httpmuxgo121=1时使用
}

// NewServeMux 是一个函数，用于分配并返回一个新的 ServeMux
func NewServeMux() *ServeMux

// Handle 函数用于为给定的 URL 模式注册处理程序。如果给定的模式与已注册的某个模式冲突，Handle 将引发 panic
// handler 是一个接口，需要实现 ServerHTTP 方法
func (mux *ServeMux) Handle(pattern string, handler Handler)

// HandleFunc 为给定的模式注册处理程序函数。如果给定的模式与已经注册的模式冲突，HandleFunc会出错。
func (mux *ServeMux) HandleFunc(pattern string, handler func(ResponseWriter, *Request))

// Handler 返回用于给定请求的处理程序
func (mux *ServeMux) Handler(r *Request) (h Handler, pattern string)

// ServeHTTP 将请求分派给模式与请求 URL 最匹配的处理程序
func (mux *ServeMux) ServeHTTP(w ResponseWriter, r *Request)
```

### 模式

`net/http` 中 **模式** 的样式为：**请求方法** **主机名** **请求资源路径** 三部分组成

```
[METHOD ][HOST]/[PATH]
```

> [!tip] 模式的三个部分都是可选的

> [!tip] METHOD：指定该模式只能处理 METHOD 指定的请求。如果省略，则处理所有请求
> 
> 例如 `GET [HOST]/[PATH]` 就只能处理 GET 请求
> 
> **METHOD 之后一定要有空格**
> 

> [!tip] HOST：指定该模式只处理 HOST 主机的请求。如果省略，则处理所有主机的请求

> [!tip] PATH：指定请求资源的路径
> 
> PATH 中可以包含 `{NAME}` 或 `{NAME...}` 形式的通配段
> 
> 特殊通配符 `{$}` 仅匹配 URL 的末尾
> 
> 模式 `"/{$}"` 仅与路径 `"/"` 匹配， 而模式 `"/"` 与每个路径匹配
> 


模式的一些例子:

- `"/index.html"` 匹配任何主机和方法的路径 `"/index.html"`
- `"GET /static/"` 匹配路径以 `"/static/"` 开头的 GET 请求
- `"example.com/"` 匹配任何到主机 `"example.com"` 的请求
- `"example.com/{$}"` 匹配主机 `"example.com"` 和路径 `"/"` 的请求
- `"b/{bucket}/o/{objectname...}"` 匹配首段为`"b"` 第三段为 `"o"` 的路径。名称 `"bucket"` 表示第二个段，`"objectname` 表示路径的剩余部分


## 启动一个 HTTP 服务

```go
package main

import (
	"net/http"
	"strings"
)

type ApiHandler struct {
}

// ServerHTTP 实现 http.Handler 接口
func (h *ApiHandler) ServeHTTP(w http.ResponseWriter, req *http.Request) {
	// 匹配 /api/ 开头的请求

	if !strings.HasPrefix(req.URL.Path, "/api") {
		http.NotFound(w, req)
		return
	}

	if req.Method == http.MethodGet {
		// 处理 GET 请求
		h.Get(w, req)
	}
}

func (h *ApiHandler) Get(w http.ResponseWriter, req *http.Request) {
	// 响应 Get 请求
	w.Write([]byte("hello, API"))
}

func main() {

	// 创建服务端结构体
	server := http.Server{
		Addr:    ":8080",
		Handler: &ApiHandler{}, // 匹配 模式 并调用该模式的 处理函数
	}

	// 开启请求
	server.ListenAndServe()
}
```

`net/http` 提供了基础的 Web 功能，即 **监听端口**，**映射静态路由**，**解析HTTP报文**

> [!tip] Web 开发中场景简单需求，标准包并没有实现
> 
> **_动态路由_**：例如 `hello/:name`，`hello/*` 这类的规则
> 
> **_路由分组_** 
> 
> **_鉴权_**：没有分组/统一鉴权的能力，需要在每个路由映射的 `handler` 中实现
> 
> **_模板_**：没有统一简化的 `HTML` 机制

这时候 HTTP 服务框架，例如，Gin, Beego 等就给我提供了这些能力。后续我们将学习 Gin 框架，然后在动手实现一个简单的 Web 框架
