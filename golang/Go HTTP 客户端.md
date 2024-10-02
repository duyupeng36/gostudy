# HTTP 客户端

这里使用 [`httpbin`](https://httpbin.org/get) 作为测试。使用 Go 帮助我们发起请求主要涉及 `http` 包中的 `Client` 和 `Request` 类型

## URL

我们在 [[网络#HTTP 协议]] 中介绍了 URL 的组成成分。`URL` 的一般形式为

```
[scheme:][//[username:password@]host:port][/]path[?query][#fragment]
```

>[!tip] URL 每个部分
>`scheme` 协议：代表 **访问资源所使用的协议类型**，例如 `http`、`https`、`ftp` 等。
> 
> `username:password`： 用户信息。**用于访问需要认证的资源**。通常用于 FTP 协议中，例如：`ftp://username:password@ftp.example.com`
>  
>  `host:port` 主机和端口：用于 **标识资源所在的服务器地址和端口号**。例如：`http://www.example.com:8080`
>  
>  `path` 路径：**标识服务器上资源的路径**，例如：`/images/logo.png`
>   
>  `query` 查询参数：用于向服务器传递参数，以便获取特定的资源。例如：`http://www.example.com/search?q=golang&sort=recent`
>   
>   `fragment` 片段标识符：指定 **文档中的某个锚点**，例如：`http://www.example.com/index.html#top`

Go 在 `net/url` 包中提供了表示解析 URL 字符串的类型 `url.URL`

```go
type URL struct {
    Scheme   string    // 协议  
    Opaque   string    // 加密的不透明数据
    User     *Userinfo // 用户名和密码信息
    Host     string    // host 或 host:port
    Path     string    // 路径（相对路径可以省略前导斜杠)
    RawQuery string    // 编码后的查询字符串，没有'?'
    Fragment string    // 引用的片段（文档位置），没有'#'
}
```

为了解析 URL 字符串，需要使用 `url.Parse()` 函数

```go
package main

import (
	"fmt"
	"log"
	url2 "net/url"
)

func main() {

	url, err := url2.Parse("https://www.example.com/search?q=golang#top")

	if err != nil {
		log.Fatalf("解析URL失败: %v\n", err)
	}

	fmt.Printf("Scheme: %v\n", url.Scheme)
	fmt.Printf("Host: %v\n", url.Host)
	fmt.Printf("Path: %v\n", url.Path)
	fmt.Printf("Query: %v\n", url.Query().Encode())
	fmt.Printf("Fragment: %v\n", url.Fragment)
}
```

### query 参数

对于 URL 的 query 参数，在 `net/url` 包中的 `Values` 类型就代表了 `query` 参数。`Values` 本质上一个 `map` 类型

```go
type Values map[string][]string
```

`url.ParseQuery()` 将 URL 中的 query 参数解析为 `Values` 类型

```go
package main

import (
	"fmt"
	"net/url"
)

func main() {
	// 解析查询参数
	values, err := url.ParseQuery("q=golang&sort=recent&limit=10")
	if err != nil {
		fmt.Println(err)
		return
	}

	// 获取特定参数的值
	q := values.Get("q")
	sort := values.Get("sort")
	limit := values.Get("limit")

	fmt.Println("q:", q)
	fmt.Println("sort:", sort)
	fmt.Println("limit:", limit)
}
```

同时提供了用于修改、添加、删除这些 query 参数以及重新编码为字符串的方法

```go
// Get 会获取 key 对应的值集的第一个值。如果没有对应 key 的值集会返回空字符串。获取值集请直接用 map
func (v Values) Get(key string) string

// Set 方法将 key 对应的值集设为只有 value，它会替换掉已有的值集
func (v Values) Set(key, value string)

// Add 将 value 添加到 key 关联的值集里原有的值的后面。
func (v Values) Add(key, value string)

// Del 删除 key 关联的值集
func (v Values) Del(key string)

// Encode 方法将 v 编码为 url 编码格式("bar=baz&foo=quux")，编码时会以键进行排序
func (v Values) Encode() string
```

### UserInfo 

`Userinfo` 类型代表了 `URL` 中的 `username:password` 段

```go
type Userinfo struct {
	username    string 
	password    string
	passwordSet bool
}

// User返回一个[Userinfo]对象，其中包含提供的用户名，没有设置密码。
func User(username string) *Userinfo {
	return &Userinfo{username, "", false}
}

// UserPassword 返回一个包含用户名和密码的 Userinfo 对象。
func UserPassword(username, password string) *Userinfo {
	return &Userinfo{username, password, true}
}


// Username 返回用户名
func (u *Userinfo) Username() string

// Password返回密码以及是否设置
func (u *Userinfo) Password() (string, bool)

// String 返回编码后的标准形式 "username[:password]" 的用户信息。
func (u *Userinfo) String() string
```

## 发起 HTTP 请求

`http.Client` 类型代表了一个 HTTP 客户端，其定义如下

```go
type Client struct {
    // Transport 指定了发起单个HTTP请求的机制
    // 如果为nil，则使用 DefaultTransport。
    Transport RoundTripper
    // CheckRedirect指定重定向处理策略
    
    // 如果 CheckRedirect 不为nil，客户端会在HTTP重定向之后调用它。参数 req 和 via 是即将到来的请求和已经发出的请求，最老的先发出
    // 如果CheckRedirect返回错误，客户端的 Get 方法会返回前一个响应(响应体已关闭)和CheckRedirect的错误(包装在url.Error中)，而不是直接发起请求
    // 有一种特殊情况，如果 CheckRedirect 返回ErrUseLastResponse，那么返回的是最近的响应，且响应体未关闭，返回 nil 错误
    // 如果CheckRedirect为nil，客户端使用默认策略，即在连续10次请求后停止。
    CheckRedirect func(req *Request, via []*Request) error

    // Jar指定cookie的容器。
    // Jar用于将相关 cookie 插入到每个出站请求中，并使用每个入站响应的 cookie 值进行更新。客户端每次重定向时都会咨询这个 Jar
    // 如果Jar为nil, cookie只有在请求中明确设置时才会发送。
    Jar CookieJar

    // Timeout指定此客户端发出请求的时间限制。超时包括连接时间、任何重定向和读取响应体。在Gect、Head、Post或Do return之后，定时器仍然在运行，并且会中断对Rcesponse.Body的读取。
    
    // 超时为0意味着没有超时。

    // 客户端取消对底层传输的请求，就像请求的上下文已经结束一样。

    // 为了兼容性，客户端还会在 Transport 端使用已弃用的 CancelRequest 方法。新的 RoundTripper 实现应该使用请求的上下文来实现取消，而不是实现 CancelRequest。
    Timeout time.Duration
}
```

请求的执行都是由 `http.Client` 发起。 它提供了一些便利的请求方法，比如我们要发起一个 `Get` 请求，可通过 `client.Get(url)` 实现。更通用的方式是通过  `client.Do(req)` 实现，`req` 属于 `Request` 类型，即我们构造的 **_请求报文_**

```go
type Request struct {
    Method string  // 请求方法
    URL *url.URL  // 请求 URL
    Proto      string // 协议版本字符串("HTTP/1.0")
    ProtoMajor int    // 主要版本 1
    ProtoMinor int    // 次要版本 0
    Header Header     // 请求头
    Body io.ReadCloser // 请求体
    ContentLength int64 // 关联内容的长度
    TransferEncoding []string // 从最外层到最内层的传输编码
    Close bool  // Close对于服务器来说，是在响应此请求后关闭连接，还是在客户端发送此请求并读取其响应后关闭连接
    Host string // 对于服务器请求，Host 指定查找 URL 的主机
    Form url.Values // 解析后的表单数据，包括 URL 字段的查询参数和 PATCH、POST 或 PUT 表单数据
    PostForm url.Values // 从 PATCH、POST 或 PUT 主体参数解析的表单数据
    MultipartForm *multipart.Form // 解析后的多部分表单，包括文件上传。
    Trailer Header // 在请求主体之后发送的其他请求头
    RemoteAddr string  // 允许 HTTP 服务器和其他软件记录发送请求的网络地址，通常用于日志记录
    RequestURI string
    TLS *tls.ConnectionState  // 允许 HTTP 服务器和其他软件记录接收到请求的TLS连接信息
}
```

`http` 包中提供了一个 **默认客户端**，可以更便捷的发起 HTTP 请求

```go
var DefaultClient = &Client{}
```

### 便捷的请求发送

`http.Get, http.Head, http.Post` 和 `http.PostForm` 就是使用 `DefauoltClient` 发起 `HTTP`（或 `HTTPS`）请求，从而获得服务端的 **响应对象**，即 `http.Response` 结构体，它是 **_响应报文_**，通过该对象就可以 HTTP 服务端响应的数据

下面的示例展示了如何便捷的发送 HTTP GET 请求

```go
package main

import (
	"bufio"
	"fmt"
	"io"
	"log"
	"net/http"
)

func main() {
	// 使用 DefaultClient 发起 Get 请求
	response, err := http.Get("https://httpbin.org/get")
	if err != nil {
		log.Fatalf("Get 请求失败: %v\n", err)
	}

	// 响应报文中的响应体是一个满足 ReadCloser 接口的对象
	// 读取数据完毕之后需要关闭它
	defer func(body io.Closer) {
		if err := body.Close(); err != nil {
			log.Panicf("关闭响应体失败: %v\n", err)
		}
	}(response.Body)

	// 读取数据
	reader := bufio.NewReader(response.Body)
	var buf = make([]byte, 4096)
	for {
		n, err := reader.Read(buf)
		if err != nil {
			if err == io.EOF {
				log.Println("数据读取完毕")
				break
			} else {
				log.Fatalf("读取数据失败: %v\n", err)
			}
		}

		fmt.Printf("%s", buf[:n])
	}

	fmt.Println()
}
```

上述代码的运行结果为

```json
{
  "args": {}, 
  "headers": {
    "Accept-Encoding": "gzip", 
    "Host": "httpbin.org", 
    "User-Agent": "Go-http-client/2.0", 
    "X-Amzn-Trace-Id": "Root=1-6628bd31-299b7e7f207dcf576c5f1730"
  }, 
  "origin": "103.182.96.112", 
  "url": "https://httpbin.org/get"
}
```

### 通用请求发送

更为通用的请求发送是使用 `Client` 的 `Do()` 方法。下面的示例是使用 `client.Do()` 发起的 HTTP GET 请求

```go
package main

import (
	"bufio"
	"fmt"
	"io"
	"log"
	"net/http"
)

func main() {
	// 构造 GET 请求的请求报文，body 为 nil
	request, err := http.NewRequest(http.MethodGet, "https://httpbin.org/get", nil)
	if err != nil {
		log.Fatalf("请求报文构造失败: %v\n", err)
	}

	response, err := http.DefaultClient.Do(request)
	if err != nil {
		log.Fatalf("Get 请求失败: %v\n", err)
	}

	// 响应报文中的响应体是一个满足 ReadCloser 接口的对象
	// 读取数据完毕之后需要关闭它
	defer func(body io.Closer) {
		if err := body.Close(); err != nil {
			log.Panicf("关闭响应体失败: %v\n", err)
		}
	}(response.Body)

	// 读取数据
	reader := bufio.NewReader(response.Body)
	var buf = make([]byte, 4096)
	for {
		n, err := reader.Read(buf)
		if err != nil {
			if err == io.EOF {
				log.Println("数据读取完毕")
				break
			} else {
				log.Fatalf("读取数据失败: %v\n", err)
			}
		}

		fmt.Printf("%s", buf[:n])
	}

	fmt.Println()
}
```


#### Get 请求

`Get` 请求用于获取一个对象

```go
// 构造 Get 请求报文
req, err := http.NewRequest(http.MethodGet, "https://httpbin.org/get", nil)

resp, err := http.DefaultClient.Do(req)
```

#### Post 请求

`Post` 请求用于向服务端发送数据

```go
// 构造 Post 请求报文
req, err := http.NewRequest(http.MethodPost, "http://httpbin.org/post", body)

resp, err := http.DefaultClient.Do(req)
```

#### Put 请求

`Put` 请求替换 URL 指定的数据

```go
// 构造 Put 请求报文
req, err := http.NewRequest(http.MethodPut, "http://httpbin.org/put", body)

resp, err := http.DefaultClient.Do()
```

#### Head 请求

`Head` 请求，类似于 `Get` 但是只返回响应头。通常用于获取文件的属性信息

```go
// 构造 Head 请求报文
req, err := http.NewRequest(http.MethodHead, "http://httpbin.org/get", nil)

resp, err := http.DefaultClient.Do(req)
```

#### Options 请求

`Options` 请求，用于通知或查询通信选项

```go
// 构造 Options 请求
req, err := http.NewRequest(http.MethodOptions, "http://httpbin.org/get", nil)

resp, err := http.DefaultClient.Do(req)
```

#### Delete 请求

`Delete` 请求，删除 URL 指定的文件

```go
req, err := http.NewRequest(http.MethodDelete, "https://httpbin.org/delete", nil)

resp, err := http.DefaultClient.Do(req)
```

## 响应

执行请求成功，如何查看响应信息。要查看响应信息，可以大概了解下，响应通常哪些内容？ 常见的有 **主体内容**（`Body`）、**状态信息**（`Status`）、**响应头部**（`Header`）、 **内容编码**（`Encoding`）等

### Body 响应体

主体中保存的是 `url` 对应的网页内容。是一个 `io.ReadCloser` 接口对象。其操作类似与文件， 支持读取内容和关闭。

响应内容多样，如果是 `json`，可以直接使用 `json.Unmarshal` 进行解码。读取数据完毕后 要记得关闭。

### Status 和 StatusCode

响应信息中，除了 Body 主体内容，还有其他信息，比如 `status code` 和 `charset` 等

其中，`r.StatusCode` 是 `HTTP` 返回状态码码， `Status` 是返回状态描述信息

###  Header 响应头

请求的响应头信息保存在该字段中，可以通过如下方式获取

```go
r.Header.Get("content-type")
r.Header.Get("Content-Type")
```

### Encoding 内容编码

对于响应内容，编码方式可能与程序解析不同，导致解析出的数据出现乱码情况。采用`http://golang.org/x/net/html/charset` 包来完成内容编码识别。使用 `transform` 包完成编码转换

```go
func determineEncoding(r *bufio.Reader) encoding.Encoding {
	// 读取 1024 字节，但是不移动指针
	bytes, err := r.Peek(1024)
	if err != nil {
		fmt.Printf("err %v", err)
		return unicode.UTF8
	}
	 // 识别编码
	e, _, _ := charset.DetermineEncoding(bytes, "")
 
	return e
}


bodyReader := bufio.NewReader(r.Body)

e := determineEncoding(bodyReader)

fmt.Printf("Encoding %v\n", e)
 
 // 使用 decodeReader 读取解码完成的数据
 decodeReader := transform.NewReader(bodyReader, e.NewDecoder())
```

### 文件下载

如果访问内容是一张图片，我们如何把它下载下来呢? 如下程序所示

```go
package main

import (
	"io"
	"log"
	"net/http"
	"os"
)

func main() {
	// 构造 GET 请求的请求报文，body 为 nil
	request, err := http.NewRequest(http.MethodGet, "https://pic.netbian.com//uploads/allimg/240714/235440-17209724800aad.jpg", nil)
	if err != nil {
		log.Fatalf("请求报文构造失败: %v\n", err)
	}

	response, err := http.DefaultClient.Do(request)
	if err != nil {
		log.Fatalf("Get 请求失败: %v\n", err)
	}

	// 响应报文中的响应体是一个满足 ReadCloser 接口的对象
	// 读取数据完毕之后需要关闭它
	defer func(body io.Closer) {
		if err := body.Close(); err != nil {
			log.Panicf("关闭响应体失败: %v\n", err)
		}
	}(response.Body)

	// 读取数据
	file, err := os.Create("八重神子.jpg")
	if err != nil {
		log.Fatalf("创建文件失败: %v\n", err)
	}

	defer func(f io.Closer) {
		if err := f.Close(); err != nil {
			log.Fatalf("关闭文件失败: %v\n", err)
		}

	}(file)

	_, err = file.ReadFrom(response.Body)
	if err != nil {
		log.Fatalf("保存文件失败: %v\n", err)
	}
}
```

`response` 即 `Response`，利用 `os.Create()` 创建了新的文件，然后再通过 `file.ReadFrom()` 将响应的内容保存进文件中

## 请求定制

### Request Header 设置

Go `http` 包将请求头封装为了 `Header` 对象。并提供了一些列的方法添加和删除请求头

```go
type Header map[string][]string

// Get返回键对应的第一个值，如果键不存在会返回 ""
// 如要获取该键对应的值切片，请直接用规范格式的键访问 map
func (h Header) Get(key string) string

// Set 添加键值对到 h，如键已存在则会用只有新值一个元素的切片取代旧值切片
func (h Header) Set(key, value string)

// Add 添加键值对到 h，如键已存在则会将新的值附加到旧值切片后面
func (h Header) Add(key, value string)

// Del 删除键值对
func (h Header) Del(key string)

// Write 以有线格式将头域写入 w
func (h Header) Write(w io.Writer) error

// WriteSubset以有线格式将头域写入 w。当 exclude 不为 nil  时，如果 h 的键值对的键在 exclude 中存在且其对应值为真，该键值对就不会被写入 w。
func (h Header) WriteSubset(w io.Writer, exclude map[string]bool) error
```

在请求报文(`Request`)中携带了 `Header` 对象，通过 `Header` 对象的方式定制请求头

```go
package main

import (
	"bufio"
	"fmt"
	"io"
	"log"
	"net/http"
)

func main() {

	// 构造请求
	req, err := http.NewRequest(http.MethodGet, "https://httpbin.org/get", nil)
	if err != nil {
		log.Panicf("请求构造失败: %s\n", err)
	}

	// 定制请求头
	req.Header.Set("User-Agent", "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_0)")

	res, err := http.DefaultClient.Do(req)
	if err != nil {
		log.Panicf("Get 请求失败: error = %s\n", err)
	}

	defer func(Body io.ReadCloser) {
		err := Body.Close()
		if err != nil {
			log.Printf("关闭响应体失败: error = %s\n", err)
		}
	}(res.Body)

	// 读取响应体数据
	reader := bufio.NewReader(res.Body)

	var data string
	var buf = make([]byte, 1024)

	for {
		n, err := reader.Read(buf)
		if err != nil && err == io.EOF {
			fmt.Println("数据读取完毕")
			break
		}
		data += string(buf[:n])
	}
	fmt.Println(data)
}
```

### url query 参数

通过将`键-值对`置于`URL`中，我们可以实现向特定地址传递数据。 该`键-值`将跟在一个问号的后面，例如`http://httpbin.org/get?key=val`

使用 `net/url` 包中 `url.Values` 类型组织 query 参数。在 `Request` 中的 `Request.RawQuery` 保存的就是原始的 query 参数

```go
// 构造请求
req, err := http.NewRequest(http.MethodGet, "https://httpbin.org/get", nil)
if err != nil {
	log.Panicf("请求构造失败: %s\n", err)
}


// 定制 URL 参数
params := make(url.Values)
params.Add("key1", "value1")
params.Add("key2", "value2")

req.URL.RawQuery = params.Encode()
```

### `Body` 数据定制

`Post` 提交的数据就是携带在请求报文的 `Body` 中。提交 `Post` 数据需要同步告诉服务端提交数据的类型，即 **`Content-Type`**。常用了请求头的 `Content-Type` 有
- `application/json`：提交的数据为 JSON 格式
- `application/x-www-form-urlencoded`：表单数据
- `multipart/form-data`：上传文件

> [!NOTE] 常见的 Content-Type
> 
> **`text` 开头**
> - `text/html`： HTML格式
> - `text/plain`：纯文本格式
> - `text/xml`： XML格式
> 
> **图片格式**
> - `image/gif` ：gif 图片格式
> - `image/jpeg` ：jpg 图片格式
> - `image/png`：png 图片格式
> 
> **`application` 开头**
> - `application/xhtml+xml`：XHTML 格式
> - `application/xml`：XML 数据格式
> - `application/atom+xml`：Atom XML 聚合格式
> - `application/json`：JSON 数据格式
> - `application/pdf`：pdf 格式
> - `application/msword`：Word 文档格式
> - `application/octet-stream`：二进制流数据（如常见的文件下载）
> - `application/x-www-form-urlencoded`：表单发送默认格式
> 
> **媒体文件**
> - `audio/x-wav`：wav 文件
> - `audio/x-ms-wma`：w 文件
> - `audio/mp3`：mp3文件
> - `video/x-ms-wmv`：wmv 文件
> - `video/mpeg4`：mp4 文件
> - `video/avi`：avi文件

> [!WARNING] 注意
> **`Body` 是一个 `io.Reader` 对象**。无论如何，都需要构建一个 `io.Reader` 对象

#### 提交 JSON 数据

```go
package main

import (
	"bufio"
	"encoding/json"
	"fmt"
	"io"
	"log"
	"net/http"
	"strings"
)

func main() {

	var person = struct {
		Username string `json:"username"`
		Age      int    `json:"age"`
	}{Username: "dyp", Age: 27}

	jsonByte, err := json.Marshal(&person)
	if err != nil {
		log.Panicf("生成 json 字符串失败：error = %s\n", err)
	}

	// 构建一个 body 它是一个 io.Reader 对象
	body := bytes.NewReader(jsonByte)

	// 构建 Post 请求
	req, err := http.NewRequest(http.MethodPost, "https://httpbin.org/post", body)
	if err != nil {
		log.Panicf("构建请求报文失败: error = %s\n", err)
	}
	
	// 设置 Content-Type 为 application/json
	req.Header.Add("Content-Type", "application/json")

	// 发送请求
	resp, err := http.DefaultClient.Do(req)
	if err != nil {
		log.Panicf("请求失败: error=%s\n", err)
	}

	defer func(Body io.ReadCloser) {
		err := Body.Close()
		if err != nil {
			log.Printf("响应体关闭失败: error = %s\n", err)
		}
	}(resp.Body)

	// 读取响应体数据
	reader := bufio.NewReader(resp.Body)
	var data string
	var buf = make([]byte, 1024)
	for {
		n, err := reader.Read(buf)
		if err != nil && err == io.EOF {
			fmt.Println("数据读取完毕")
			break
		}
		data += string(buf[:n])
	}
	fmt.Println(data)
}
```
运行结果为
```json
{
  "args": {}, 
  // data 就是 body 中的数据
  "data": "{\"username\":\"dyp\",\"age\":27}", 
  "files": {}, 
  "form": {}, 
  "headers": {
    "Accept-Encoding": "gzip", 
    "Content-Length": "27", 
    "Content-Type": "application/json",  // Content-Type
    "Host": "httpbin.org", 
    "User-Agent": "Go-http-client/2.0", 
    "X-Amzn-Trace-Id": "Root=1-6628d8b7-3418f3976d7a58ca68acfa55"
  }, 
  "json": {
    "age": 27, 
    "username": "dyp"
  }, 
  "origin": "103.182.96.112", 
  "url": "https://httpbin.org/post"
}
```
#### 提交表单数据

表单提交是一个很常用的功能，故而在 `net/http` 中，除了提供标准的用法外， 还给我们提供了简化的方法

`Post` 提交表单的 `payload` 是形如 `name=poloxue&password=123456` 的字符串， 故而我们可以通过 `url.Values` 进行组织

构造请求报文时，内容必须是实现 `io.Reader` 接口的类型， 所以需要 `strings.NewReader`转化下。`Form` 表单提交的 `Content-Type` 是 `application/x-www-form-urlencoded`

```go
package main

import (
	"bufio"
	"fmt"
	"io"
	"log"
	"net/http"
	"net/url"
	"strings"
)

func main() {
	// 构建 payload
	payload := make(url.Values)
	payload.Add("name", "dyp")
	payload.Add("password", "123456")
	
	// 生成 payload 的 Body
	payloadBody := strings.NewReader(payload.Encode())
	
	// 构建 Post 请求
	req, err := http.NewRequest(http.MethodPost, "https://httpbin.org/post", payloadBody)
	if err != nil {
		log.Panicf("构建请求报文失败: error = %s\n", err)
	}
	// 设置 Content-Type 为 application/x-www-form-urlencoded
	req.Header.Add("Content-Type", "application/x-www-form-urlencoded")
	// 发送请求
	resp, err := http.DefaultClient.Do(req)
	if err != nil {
		log.Panicf("请求失败: error=%s\n", err)
	}

	defer func(Body io.ReadCloser) {
		err := Body.Close()
		if err != nil {
			log.Printf("响应体关闭失败: error = %s\n", err)
		}
	}(resp.Body)

	// 读取响应体数据
	reader := bufio.NewReader(resp.Body)
	var data string
	var buf = make([]byte, 1024)
	for {
		n, err := reader.Read(buf)
		if err != nil && err == io.EOF {
			fmt.Println("数据读取完毕")
			break
		}
		data += string(buf[:n])
	}
	fmt.Println(data)
}
```
这段代码的运行结果为
```json
{
  "args": {}, 
  "data": "", 
  "files": {}, 
  // 这就是我们构建的表单对象
  "form": {
    "name": "dyp", 
    "password": "123456"
  }, 
  "headers": {
    "Accept-Encoding": "gzip", 
    "Content-Length": "24", 
    "Content-Type": "application/x-www-form-urlencoded",  // 表单的 Content-Type
    "Host": "httpbin.org", 
    "User-Agent": "Go-http-client/2.0", 
    "X-Amzn-Trace-Id": "Root=1-6628db90-51d2868275996aab34fd0a29"
  }, 
  "json": null, 
  "origin": "103.182.96.112", 
  "url": "https://httpbin.org/post"
}
```

接着再介绍简化的方式，其实表 **单提交只需调用 `http.PostForm` 即可完成**。示例代码如下
```go
package main

import (
	"bufio"
	"fmt"
	"io"
	"log"
	"net/http"
	"net/url"
)

func main() {

	// 构建 payload
	payload := make(url.Values)
	payload.Add("name", "dyp")
	payload.Add("password", "123456")

	// 发送请求，让 Client 帮我们自行打包为表单数据
	resp, err := http.DefaultClient.PostForm("https://httpbin.org/post", payload)
	if err != nil {
		log.Panicf("请求失败: error=%s\n", err)
	}

	defer func(Body io.ReadCloser) {
		err := Body.Close()
		if err != nil {
			log.Printf("响应体关闭失败: error = %s\n", err)
		}
	}(resp.Body)

	// 读取响应体数据
	reader := bufio.NewReader(resp.Body)
	var data string
	var buf = make([]byte, 1024)
	for {
		n, err := reader.Read(buf)
		if err != nil && err == io.EOF {
			fmt.Println("数据读取完毕")
			break
		}
		data += string(buf[:n])
	}
	fmt.Println(data)
}
```
运行结果为
```json
{
  "args": {}, 
  "data": "", 
  "files": {}, 
  "form": {
    "name": "dyp", 
    "password": "123456"
  }, 
  "headers": {
    "Accept-Encoding": "gzip", 
    "Content-Length": "24", 
    "Content-Type": "application/x-www-form-urlencoded", 
    "Host": "httpbin.org", 
    "User-Agent": "Go-http-client/2.0", 
    "X-Amzn-Trace-Id": "Root=1-6628dc7c-2196a5803cf304f41e7e9ce0"
  }, 
  "json": null, 
  "origin": "103.182.96.112", 
  "url": "https://httpbin.org/post"
}
```
#### 上传文件

HTTP 客户端在上传文件时，会 **将文件拆分多个部分**，即 **`multipart/form-data`** 的类型。Go 的 `mime/multipart` 包实现了 `MIME` 的 `multipart` 解析，该实现适用于 `HTTP` 和常见浏览器生成的 `multipart` 主体

要将文件组织为 `mutipart/form-data` 类型的数据，需要经过下面几步
- 首先，需要打开将要上传的文件，使用 `defer f.Close()` 做好资源释放的准备
- 然后创建一个 `bytes.Buffer` 对象，用于存储将要上传的内容
- 通过 `multipart.NewWriter` 创建一个 `mutipart.Writer` 对象用于生成 `multipart` 信息
- 通过 `writer.CreateFormFile` 创建上传文件并通过 `io.Copy` 向其中写入内容
- 通过 `writer.WriteField` 添加其他的附加信息，注意最后要把 `writer` 关闭

至此，文件上传的数据就组织完成了。接下来，只需调用 `Post` 方法即可完成文件上传

```go
package main

import (
	"bufio"
	"bytes"
	"fmt"
	"io"
	"log"
	"mime/multipart"
	"net/http"
	"os"
)

func main() {

	// 打开文件 猫羽雫.jpg
	file, err := os.Open("猫羽雫.jpg")
	if err != nil {
		log.Panicf("打卡文件失败: error=%s\n", err)
	}
	defer func(file *os.File) {
		err := file.Close()
		if err != nil {
			log.Printf("关闭文件失败: %s\n", err)
		}
	}(file)

	// 创建一个 buffer 对象
	var uploadBody = &bytes.Buffer{}
	// 创建 writer 用于向 buffer 中写入数据
	writer := multipart.NewWriter(uploadBody)

	// 创建 multipart/form-data
	formFileWriter, err := writer.CreateFormFile("uploadFile", "猫羽雫.jpg")

	if err != nil {
		log.Panicf("创建 multipart/form-data 对象失败: error = %s\n", err)
	}

	// 将 文件中的数据拷贝到 formFileWriter 中，formFileWriter 就会将数据按照 multipart/form-data 格式写入到 uploadBody 中
	_, err = io.Copy(formFileWriter, file)
	if err != nil {
		log.Panicf("拷贝数据出错: error=%s\n", err)
	}
	// 添加附加信息
	fileField := map[string]string{
		"filename": "猫羽雫.jpg",
	}
	for k, v := range fileField {
		err := writer.WriteField(k, v)
		if err != nil {
			log.Printf("添加文件附加信息出错: %s\n", err)
			break
		}
	}
	// 数据构建完成，关闭 writer
	err = writer.Close()
	if err != nil {
		log.Printf("关闭 multipart.Writer 失败: error=%s\n", err)
	}

	// 构建 Post 请求
	req, err := http.NewRequest(http.MethodPost, "https://httpbin.org/post", uploadBody)
	if err != nil {
		log.Panicf("构建请求报文失败: error = %s\n", err)
	}
	// 设置 Content-Type。通过 writer.FormDataContentType 获得。 multipart/form-data; boundary=
	fmt.Println("Content-Type: ", writer.FormDataContentType())
	req.Header.Add("Content-Type", writer.FormDataContentType())

	// 发送请求
	resp, err := http.DefaultClient.Do(req)
	if err != nil {
		log.Panicf("请求失败: error=%s\n", err)
	}

	defer func(Body io.ReadCloser) {
		err := Body.Close()
		if err != nil {
			log.Printf("响应体关闭失败: error = %s\n", err)
		}
	}(resp.Body)

	// 读取响应体数据
	reader := bufio.NewReader(resp.Body)
	var data string
	var buf = make([]byte, 1024)
	for {
		n, err := reader.Read(buf)
		if err != nil && err == io.EOF {
			fmt.Println("数据读取完毕")
			break
		}
		data += string(buf[:n])
	}
	fmt.Println(data)
}
```

运行结果为

```json
{
  "args": {}, 
  "data": "", 
  "files": {
	// uploadFile 就是在 CreateFormFile 中指定的名称
    "uploadFile": "data:application/octet-stream;base64,......."
  }, 
  "form": {
	// 上传文件的附加信息
    "filename": "\u732b\u7fbd\u96eb.jpg"
  }, 
  "headers": {
    "Accept-Encoding": "gzip", 
    "Content-Length": "1446237", 
    "Content-Type": "multipart/form-data; boundary=a599c1f9a16c966eebd0de375f8245e7e78ec34fc3d264620a0e62a17c28", 
    "Host": "httpbin.org", 
    "User-Agent": "Go-http-client/2.0", 
    "X-Amzn-Trace-Id": "Root=1-6628e826-541c78e07ef9e4de6db27d82"
  }, 
  "json": null, 
  "origin": "103.182.96.112", 
  "url": "https://httpbin.org/post"
}
```

### 设置 Cookie

**HTTP 协议是无状态的协议**。这简化了服务器的设计，并且允许工程师们去开发可以同时处理数以千计的 TCP 连接的高性能 Web 服务器

然而一个 **Web 站点通常希望能够识别用户**，可能是因为 **服务器希望限制用户的访问**，或者因为它希望把内容与用户身份联系起来。为此，**HTTP 使用了 `cookie`**，它允许站点对用户进行跟踪

当客户端第一次请求服务器程序时，服务器程序会在其 **响应头** 中的 `Set-cookie` 字段添加一个 **唯一标识**。客户端收到该响应之后，会解析出 `Set-cokie` 并存储在客户管理 `cookie` 的文件中。当客户端再次向该服务器程序发起请求时，就会携带该 `cookie`，服务器程序就能标识特定的用户了。

**`cookie` 技术有 4 个组件**：
- 在 **HTTP 响应报文** 中的一个 **cookie 首部行**
- 在 **HTTP 请求报文** 中的一个 **cookie 首部行**
- 在 **用户端系统中保留有一个 cookie 文件**，并由用户的 **客户端进行管理**
- 服务器应用程序对 cookie 的校验

#### `Cookie` 类型  

`net/http` 包中提供了对 `Cookie` 的封装

```go
type Cookie struct {
    Name       string
    Value      string
    Path       string
    Domain     string
    Expires    time.Time
    RawExpires string
    // MaxAge=0表示未设置Max-Age属性
    // MaxAge<0表示立刻删除该cookie，等价于"Max-Age: 0"
    // MaxAge>0表示存在Max-Age属性，单位是秒
    MaxAge   int
    Secure   bool
    HttpOnly bool
    Raw      string
    Unparsed []string // 未解析的“属性-值”对的原始文本
}

// String 方法返回该 cookie 的序列化结果
func (c *Cookie) String() string
```

`Cookie` 代表一个出现在 `HTTP` 响应的头域中 `Set-Cookie` 头的值里或者 `HTTP` 请求的头域中 `Cookie` 头的值里的HTTP `cookie`


> [!tip] 
> 
> 如果 **只设置了 `Name` 和 `Value` 字段**，序列化结果可用于 `HTTP` 请求的 `Cookie` 头或者 `HTTP` 回复的 `Set-Cookie` 头（即：既可以用于 HTTP 请求头，也可以用于 HTTP 响应头）
> 
> 如果 **设置了其他字段**，序列化结果 **只能用于 `HTTP` 回复的 `Set-Cookie` 头**

#### CookieJar 接口

`CookieJar` 管理 `cookie` 的存储和在 `HTTP` 请求中的使用。`CookieJar` 的实现必须能安全的被多个 `go` 程同时使用

```go
type CookieJar interface {

    // SetCookies 管理从 u 的回复中收到的 cookie
    // 根据其策略和实现，它可以选择是否存储 cookie
    SetCookies(u *url.URL, cookies []*Cookie)
    
    // Cookies 返回发送请求到 u 时应使用的 cookie
    // 本方法有责任遵守 RFC 6265 规定的标准 cookie限制
    Cookies(u *url.URL) []*Cookie
}
```

`net/http/cookiejar` 包提供了一个 `CookieJar` 的实现。其中，`Jar` 类型实现了 `net/http` 包的 `http.CookieJar` 接口

```go
type Jar struct {
    // 内含隐藏或非导出字段
}

// New 返回一个新的 Jar，nil 指针等价于 Options 零值的指针
func New(o *Options) (*Jar, error)

// 实现 CookieJar 接口的 Cookies 方法，如果 URL 协议不是 HTTP/HTTPS 会返回空切片
func (j *Jar) Cookies(u *url.URL) (cookies []*http.Cookie)

// 实现 CookieJar 接口的 SetCookies 方法，如果 URL 协议不是 HTTP/HTTPS 则不会有实际操作
func (j *Jar) SetCookies(u *url.URL, cookies []*http.Cookie)
```

通过上面的描述，为了在请求中带上 `Cookie`，可以在两个位置添加 `Cookie`
+ 设置在 `Client` 上
+ 设置在 `Request` 上

#### 在 `Client` 上设置 `Cookie`

在 `Client` 上设置 `Cookie` 就是 **模拟保存服务端响应 `Cookie`**

> [!tip] 在 Client 上设置 Cookie 的步骤
> 首先，我们需要创建一个 `http.Cookie` 切片，然后向其中添加了 `Cookie` 数据
> 
> 然后通过 `cookiejar` 保存所有的 `Cookie` 数据，这一步需要将 `Cookie` 与 URL 绑定
> 
> 最后，新建一个 `Client` 对象，并将 `cookiejar` 与 `Client` 绑定
> 
> 注意，不能在 `DefaultClient` 上设置 `Cookie`

```go
package main

import (
	"bufio"
	"fmt"
	"io"
	"log"
	"net/http"
	"net/http/cookiejar"
	url2 "net/url"
)

func main() {

	// 模拟构建服务端响应的 Cookie
	cookies := make([]*http.Cookie, 0, 5)
	cookies = append(cookies, &http.Cookie{
		Name:   "name",
		Value:  "poloxue",
		Domain: "httpbin.org",
		Path:   "/cookies",
	})
	cookies = append(cookies, &http.Cookie{
		Name:   "id",
		Value:  "10000",
		Domain: "httpbin.org",
		Path:   "/elsewhere",
	})

	// 创建 url 对象
	url, err := url2.Parse("https://httpbin.org/cookies")
	if err != nil {
		log.Panicf("解析 URL 失败: error = %s\n", err)
	}

	// 创建 Cookiejar 对象
	jar, err := cookiejar.New(nil)
	if err != nil {
		log.Panicf("创建 cookiejar 对象失败: error = %s\n", err)
	}
	// 将 URL 与 cookies 绑定
	jar.SetCookies(url, cookies)

	// 给 Client 设置 cookies
	var client = http.Client{
		Jar: jar,
	}

	// 构建 Get 请求
	req, err := http.NewRequest(http.MethodGet, "https://httpbin.org/cookies", nil)
	if err != nil {
		log.Panicf("构建请求报文失败: error = %s\n", err)
	}

	// 发送请求
	resp, err := client.Do(req)
	if err != nil {
		log.Panicf("请求失败: error=%s\n", err)
	}

	defer func(Body io.ReadCloser) {
		err := Body.Close()
		if err != nil {
			log.Printf("响应体关闭失败: error = %s\n", err)
		}
	}(resp.Body)

	// 读取响应体数据
	reader := bufio.NewReader(resp.Body)
	var data string
	var buf = make([]byte, 1024)
	for {
		n, err := reader.Read(buf)
		if err != nil && err == io.EOF {
			fmt.Println("数据读取完毕")
			break
		}
		data += string(buf[:n])
	}
	fmt.Println(data)
}
```

#### 在 `Request` 上设置 `Cookie`

在 `Request` 上设置 `Cookie` 模拟的是 **请求携带的 `Cookie`**。`Request` 对象的 `AddCookie` 方法即可非常方便的添加 `Cookie`

```go
package main

import (
	"bufio"
	"fmt"
	"io"
	"log"
	"net/http"
)

func main() {

	cookie := &http.Cookie{
		Name:   "name",
		Value:  "poloxue",
		Domain: "httpbin.org",
		Path:   "/cookies",
	}
	// 构建 Get 请求
	req, err := http.NewRequest(http.MethodGet, "https://httpbin.org/cookies", nil)
	if err != nil {
		log.Panicf("构建请求报文失败: error = %s\n", err)
	}
	// 在请求报文上添加 Cookie
	req.AddCookie(cookie)
	// 发送请求
	resp, err := http.DefaultClient.Do(req)
	if err != nil {
		log.Panicf("请求失败: error=%s\n", err)
	}

	defer func(Body io.ReadCloser) {
		err := Body.Close()
		if err != nil {
			log.Printf("响应体关闭失败: error = %s\n", err)
		}
	}(resp.Body)

	// 读取响应体数据
	reader := bufio.NewReader(resp.Body)
	var data string
	var buf = make([]byte, 1024)
	for {
		n, err := reader.Read(buf)
		if err != nil && err == io.EOF {
			fmt.Println("数据读取完毕")
			break
		}
		data += string(buf[:n])
	}
	fmt.Println(data)
}
```

> [!tip] 在 `Client` 上添加 `Cookie` 和在 `Request` 上添加 `Cookie` 的区别
>  
> 在 **`Request`的`cookie`只是当次请求有效**
> 
> 在 **`Client` 上的 `cookie` 是随时有效的**， 只要你用的是这个新创建的 `Client`

### 重定向

默认情况下，**所有类型请求都会自动处理重定向**。`net/http` 中的重定向控制可以通过 `Client` 中的一个名为 `CheckRedirect` 的成员控制，它是函数类型。定义如下

```go
type Client struct {
	...
	CheckRedirect func(req *Request, via []*Request) error
	...
}
```

在遇到 **循环重定向问题** 时，可以设置 `CheckRedirect` 解决

```go
package main

import (
	"bufio"
	"errors"
	"fmt"
	"io"
	"log"
	"net/http"
	"net/url"
)

func main() {

	// 构造请求
	req, err := http.NewRequest(http.MethodGet, "https://httpbin.org/get", nil)
	if err != nil {
		log.Panicf("请求构造失败: %s\n", err)
	}

	// 设置 CheckRedirect 已解决循环重定向问题
	// req 下次将要请求的 Request, via 已经请求过的 Request
	http.DefaultClient.CheckRedirect = func(req *http.Request, via []*http.Request) error {
		// 当 已经请求大于或等于 10 次时，就不请求
		if len(via) >= 10 {
			return errors.New("redirect to many times")
		}
		return nil
	}

	res, err := http.DefaultClient.Do(req)
	if err != nil {
		log.Panicf("Get 请求失败: error = %s\n", err)
	}
	defer func(Body io.ReadCloser) {
		err := Body.Close()
		if err != nil {
			log.Printf("关闭响应体失败: error = %s\n", err)
		}
	}(res.Body)

	// 读取响应体数据
	reader := bufio.NewReader(res.Body)
	var data string
	var buf = make([]byte, 1024)
	for {
		n, err := reader.Read(buf)
		if err != nil && err == io.EOF {
			fmt.Println("数据读取完毕")
			break
		}
		data += string(buf[:n])
	}
	fmt.Println(data)
}
```

### 超时设置

`Request` 发出后，如果服务端迟迟没有响应，客户端也不能一致等待。通常，客户端会等待一段时间，如果迟迟未得到响应就会放弃该次请求。

超时可以分为 **连接超时** 和 **响应读取超时**，这些都可以设置。 但正常情况下，并不想有那么明确的区别，那么也可以设置个 **总超时**

#### 总超时

总的超时时间的设置是绑定在 `Client` 对象的一个名为 `Timeout` 的成员之上， `Timeout` 是 `time.Duration`类型。假设这是超时时间为 `10` 秒，示例代码：

```go
client := http.Client{
	Timeout:   time.Duration(10 * time.Second),
}
```

#### 连接超时

连接超时可通过 `Client` 中的 `Transport` 实现。 `Transport` 中有个名为 `Dial` 的成员函数，可用设置连接超时。 **`Transport` 是 `HTTP` 底层的数据运输者**，`RoundTripper` 接口

```go
type RoundTripper interface {
    // RoundTrip 执行单次 HTTP 事务，接收并发挥请求 req 的回复
    // RoundTrip 不应试图解析/修改得到的回复。
    // 尤其要注意，只要 RoundTrip 获得了一个回复，不管该回复的 HTTP 状态码如何，
    // 它必须将返回值 err 设置为 nil。
    // 非 nil 的返回值 err 应该留给获取回复失败的情况。
    // 类似的，RoundTrip 不能试图管理高层次的细节，如重定向、认证、cookie。
    //
    // 除了从请求的主体读取并关闭主体之外，RoundTrip不应修改请求，包括（请求的）错误。
    // RoundTrip函数接收的请求的URL和Header字段可以保证是（被）初始化了的。
    RoundTrip(*Request) (*Response, error)
}
```

`Transport` 类型实现了 `RoundTripper` 接口，支持 `http`、`https` 和 `http/https`代理。`Transport` 类型可以缓存连接以在未来重用


```go
type Transport struct {
	....
	// DialContext 指定了用于创建未加密 TCP 连接的拨号函数
	// 如果 DialContext 为 nil（下面废弃的 Dial 也为 nil），则传输使用包 net 拨号。
	// DialContext 与 RoundTrip 调用同时运行
	// 当先前的连接在后面的 DialContext 完成之前处于空闲状态时，发起拨号的 RoundTrip 调用可能会最终使用先前拨号的连接。
	DialContext func(ctx context.Context, network, addr string) (net.Conn, error)

	// DialTLSContext 指定了一个可选的拨号函数，用于为非代理 HTTPS 请求创建 TLS 连接
	// 如果 DialTLSContext 为 nil（下面废弃的 DialTLS 也为 nil），则使用 DialContext 和 TLSClientConfig。
	// 如果设置了 DialTLSContext，HTTPS 请求将不使用 Dial 和 DialContext 钩子，TLSClientConfig 和 TLSHandshakeTimeout 将被忽略。返回的 net.Conn 假设已经 已通过 TLS 握手。
	DialTLSContext func(ctx context.Context, network, addr string) (net.Conn, error)

	// LSHandshakeTimeout 指定等待 TLS 握手完成的最长时间。零值表示不设置超时
	TLSHandshakeTimeout time.Duration

	// 如果 DisableKeepAlives 为真，会禁止不同HTTP请求之间TCP连接的重用
	DisableKeepAlives bool

	// 如果DisableCompression为真，会禁止Transport在请求中没有Accept-Encoding头时，
    // 主动添加"Accept-Encoding: gzip"头，以获取压缩数据。
    // 如果Transport自己请求gzip并得到了压缩后的回复，它会主动解压缩回复的主体。
    // 但如果用户显式的请求gzip压缩数据，Transport是不会主动解压缩的。
	DisableCompression bool

	// 如果MaxIdleConnsPerHost!=0，会控制每个主机下的最大闲置连接。
    // 如果MaxIdleConnsPerHost==0，会使用DefaultMaxIdleConnsPerHost。
	MaxIdleConns int
	....

	//  IdleConnTimeout 是空闲（保持连接）连接在关闭前保持空闲的最长时间。 零表示没有限制
	IdleConnTimeout time.Duration
	
	// ResponseHeaderTimeout 指定在发送完请求（包括其可能的主体）之后，
    // 等待接收服务端的回复的头域的最大时间。零值表示不设置超时。
    // 该时间不包括获取回复主体的时间。
	ResponseHeaderTimeout time.Duration
	
	// ExpectContinueTimeout（如果非零）用于指定在完全写入请求头（如果请求有 "Expect: 100-continue "头）后等待服务器第一个响应头的时间。零表示不超时，并导致立即发送正文，而不等待服务器批准。这段时间不包括发送请求头的时间。
	ExpectContinueTimeout time.Duration
	....
}
```

假设设置连接超时时间为 $2$ 秒

```go
// 设置超时
transport := &http.Transport{
	// 设置连接超时
	DialContext: func(ctx context.Context, network, addr string) (net.Conn, error) {
		timeout := time.Duration(2 * time.Second)
		return net.DialTimeout(network, addr, timeout)
	},
}

client := http.Client{
	Transport: transport
	Timeout:   time.Duration(10 * time.Second),
}
```

#### 读取超时

读取超时也要通过 `Client` 的 `Transport` 设置，比如设置响应的读取为 `8` 秒


```go
t := &http.Transport{
	ResponseHeaderTimeout: time.Second * 8,
}
```

#### 示例：设置所有类型的超时

```go
package main

import (
	"bufio"
	"fmt"
	"golang.org/x/net/context"
	"io"
	"log"
	"net"
	"net/http"
	"time"
)

func main() {

	// 构造请求
	req, err := http.NewRequest(http.MethodGet, "https://httpbin.org/get", nil)
	if err != nil {
		log.Panicf("请求构造失败: %s\n", err)
	}
	// 定制请求头
	req.Header.Set("User-Agent", "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_0)")

	// 配置 传输器
	transport := &http.Transport{
		// 设置连接超时
		DialContext: func(ctx context.Context, network, addr string) (net.Conn, error) {
			timeout := time.Duration(2 * time.Second)
			return net.DialTimeout(network, addr, timeout)
		},
		//读取超时
		ResponseHeaderTimeout: time.Second * 8,
	}
	// 设置为客户端设置超时处理
	http.DefaultClient.Transport = transport
	http.DefaultClient.Timeout = time.Duration(10 * time.Second)

	res, err := http.DefaultClient.Do(req)
	if err != nil {
		log.Panicf("Get 请求失败: error = %s\n", err)
	}
	defer func(Body io.ReadCloser) {
		err := Body.Close()
		if err != nil {
			log.Printf("关闭响应体失败: error = %s\n", err)
		}
	}(res.Body)

	// 读取响应体数据
	reader := bufio.NewReader(res.Body)
	var data string
	var buf = make([]byte, 1024)
	for {
		n, err := reader.Read(buf)
		if err != nil && err == io.EOF {
			fmt.Println("数据读取完毕")
			break
		}
		data += string(buf[:n])
	}
	fmt.Println(data)
}
```

### 请求代理

当我们对一个网站短时间内发起大量的请求时，网站为了自身安全就会禁止我们 IP 的访问请求。代理还是挺重要的，特别对于开发爬虫。那 `net/http` 怎么设置代理？ 这个工作还是要依赖 `Client` 的成员 `Transport` 实现， 这个 `Transport` 还是挺重要的。

`Transport` 有个名为 `Proxy` 的成员，它是 `func(*Request) (*url.URL, error)` 类型的函数。在 `net/http` 包的一个包级函数 `ProxyURL` 就是返回一个使用在 `fixedURL` 的代理函数

```go
func ProxyURL(fixedURL *url.URL) func(*Request) (*url.URL, error)
```

```go
package main

import (
	"bufio"
	"crypto/tls"
	"fmt"
	"io"
	"log"
	"net/http"
	url2 "net/url"
)

func main() {

	// 构造请求
	req, err := http.NewRequest(http.MethodGet, "https://httpbin.org/get", nil)
	if err != nil {
		log.Panicf("请求构造失败: %s\n", err)
	}
	// 定制请求头
	req.Header.Set("User-Agent", "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_0)")

	// 解析代理服务器的 url
	proxyUrl, err := url2.Parse("http://127.0.0.1:7890")
	if err != nil {
		log.Panicf("解析代理服务器URL失败: %s\n", err)
	}
	// 设置代理
	transport := &http.Transport{
		Proxy:           http.ProxyURL(proxyUrl),
		TLSClientConfig: &tls.Config{InsecureSkipVerify: true}, // 设置跳过TLS认证
	}
	http.DefaultClient.Transport = transport
	// 发起请求
	res, err := http.DefaultClient.Do(req)
	if err != nil {
		log.Panicf("Get 请求失败: error = %s\n", err)
	}
	defer func(Body io.ReadCloser) {
		err := Body.Close()
		if err != nil {
			log.Printf("关闭响应体失败: error = %s\n", err)
		}
	}(res.Body)

	// 读取响应体数据
	reader := bufio.NewReader(res.Body)
	var data string
	var buf = make([]byte, 1024)
	for {
		n, err := reader.Read(buf)
		if err != nil && err == io.EOF {
			fmt.Println("数据读取完毕")
			break
		}
		data += string(buf[:n])
	}
	fmt.Println(data)
}
```

## 彼岸图网 4K动漫

```go
package main

import (
	"bytes"
	"fmt"
	"golang.org/x/text/encoding/simplifiedchinese"
	"golang.org/x/text/transform"
	"io"
	"log"
	"net/http"
	"net/http/cookiejar"
	url2 "net/url"
	"os"
	"path/filepath"
	"regexp"
	"sync"
	"time"
)

func generateUrl() (urls []string) {
	urlTemplate := "https://pic.netbian.com/4kdongman/index_%v.html"
	urls = append(urls, "https://pic.netbian.com/4kdongman/index.html")
	for i := 2; i < 128; i++ {
		urls = append(urls, fmt.Sprintf(urlTemplate, i))
	}
	return
}

var client *http.Client

func init() {
	// 组织 cookie
	cookies := []*http.Cookie{
		{
			Name:   "PHPSESSID",
			Value:  "njvvmrqpoitbaaefgoujjref83",
			Domain: "pic.netbian.com",
			Path:   "/",
		},
		{
			Name:   "zkhanecookieclassrecord",
			Value:  "%2C54%2C66%2C",
			Domain: "pic.netbian.com",
			Path:   "/",
		},
		{
			Name:   "zkhanmlauth",
			Value:  "db633717a4fc01427e8ea8121cc3bc66",
			Domain: "pic.netbian.com",
			Path:   "/",
		},
		{
			Name:   "zkhanmlgroupid",
			Value:  "1",
			Domain: "pic.netbian.com",
			Path:   "/",
		},
		{
			Name:   "zkhanmlrnd",
			Value:  "K6mAJzOeGTVYvEz65Vzo",
			Domain: "pic.netbian.com",
			Path:   "/",
		},
		{
			Name:   "zkhanmluserid",
			Value:  "4529286",
			Domain: "pic.netbian.com",
			Path:   "/",
		},
		{
			Name:   "zkhanmlusername",
			Value:  "%BE%B2%DA%D7%D6%AE%C9%D1",
			Domain: "pic.netbian.com",
			Path:   "/",
		},
	}

	// 创建 url 对象
	url, err := url2.Parse("https://pic.netbian.com/")
	if err != nil {
		log.Panicf("解析 URL 失败: error = %s\n", err)
	}

	// 创建 cookiejar
	jar, err := cookiejar.New(nil)
	if err != nil {
		log.Panicf("创建 cookiejar 对象失败: error = %s\n", err)
	}
	// 将 url 与 cookies 绑定
	jar.SetCookies(url, cookies)

	// 将 cookiejar 绑定给 client

	client = &http.Client{
		Jar: jar,
	}
}

func CreateRequest(url string) *http.Request {
	req, err := http.NewRequest(http.MethodGet, url, nil)
	if err != nil {
		log.Printf("构建请求报文失败: error = %s\n", err)
	}

	// 添加 header
	req.Header.Add("user-agent", "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/127.0.0.0 Safari/537.36")
	req.Header.Add("referer", "https://pic.netbian.com/4kdongman/")

	req.Header.Add("sec-ch-ua", "Not)A;Brand")
	req.Header.Add("sec-ch-ua", `v="99", "Google Chrome"`)
	req.Header.Add("sec-ch-ua", `v="127", "Chromium"`)
	req.Header.Add("sec-ch-ua", `v="127"`)

	req.Header.Add("sec-ch-ua-mobil", "?0")

	req.Header.Add("sec-ch-ua-platform", "Windows")
	return req
}

var wg sync.WaitGroup

func main() {

	for _, url := range generateUrl() {
		// 构建 Get 请求
		req := CreateRequest(url)
		//发起请求
		resp, err := client.Do(req)
		if err != nil {
			log.Printf("请求失败: error=%s\n", err)
		}

		// 读取响应结果
		var buffer bytes.Buffer

		WithClose(resp.Body, func() {
			reader := transform.NewReader(resp.Body, simplifiedchinese.GBK.NewDecoder())
			if _, err := buffer.ReadFrom(reader); err != nil && err == io.EOF {
				log.Println("数据读取完毕")
			}
		})

		html := buffer.String()

		wg.Add(1)
		go func() {
			defer wg.Done()

			urlTitle := GetImageUrl(parsURL(html, `<li><a href="(.*?)" target="_blank"><img src=".*?" alt=".*?" /><b>(.*?)</b></a></li>`))

			failedIndices := make([]int, 0, len(urlTitle))
			for i, v := range urlTitle {
				time.Sleep(1 * time.Second)
				req := CreateRequest(v[0])

				resp, err := client.Do(req)
				if err != nil {
					log.Println(err)
					failedIndices = append(failedIndices, i)
					continue
				}
				file, err := os.Create(v[1])

				if err != nil {
					log.Println(err)
					failedIndices = append(failedIndices, i)
					continue
				}

				WithClose(resp.Body, func() {
					WithClose(file, func() {
						_, err := file.ReadFrom(resp.Body)
						if err != nil {
							return
						}
					})
					fmt.Println("图片", v[1], "下载完成")
				})
			}

			fmt.Println(url, " 失败索引：", failedIndices)
		}()

		time.Sleep(1 * time.Second)
	}

	wg.Wait()
}

func WithClose(c io.Closer, fn func()) {
	defer func() {
		if err := c.Close(); err != nil {
			log.Printf("响应体关闭失败: error = %s\n", err)
		}
	}()

	fn()
}

func parsURL(html string, nick string) []string {
	reg := regexp.MustCompile(nick)

	result := make([]string, 0, 20)

	for _, v := range reg.FindAllStringSubmatch(html, -1) {
		result = append(result, "https://pic.netbian.com/"+v[1])
	}
	return result
}

func GetImageUrl(urls []string) [][]string {

	result := make([][]string, 0)
	reg := regexp.MustCompile(`<div class="photo-pic"><a href="" id="img"><img src="(.*?)" data-pic=".*?" alt=".*?" title="(.*?)"></a></div>`)

	for _, url := range urls {
		req := CreateRequest(url)
		resp, err := client.Do(req)
		if err != nil {
			log.Printf("请求失败: error=%s\n", err)
		}

		// 读取响应结果
		var buffer bytes.Buffer

		WithClose(resp.Body, func() {
			reader := transform.NewReader(resp.Body, simplifiedchinese.GBK.NewDecoder())
			if _, err := buffer.ReadFrom(reader); err != nil && err == io.EOF {
				log.Println("数据读取完毕")
			}
		})

		for _, v := range reg.FindAllStringSubmatch(buffer.String(), -1) {
			result = append(result, []string{"https://pic.netbian.com" + v[1], filepath.Base(v[1])})
		}

		time.Sleep(1 * time.Second)
	}
	return result
}
```

