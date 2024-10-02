# GO Mongo

程序需要和 MongoDB Server 通信，需要使用 **驱动**

> [!tip] 驱动
> 
> 就是封装了连接，数据解析等操作应用程序接口(API)

MongoDB 官方提供了 [MongoDB Go 驱动](https://www.mongodb.com/zh-cn/docs/drivers/go/current/)

需要再下载 Go 驱动，并引入到当前项目中

```shell
go get -u go.mongodb.org/mongo-driver/mongo
```

此外，使用 `godotenv` 包从环境变量中读取 MongoDB 连接字符串，避免在源代码中嵌入凭据

```shell
go get -u github.com/joho/godotenv
```

Go 程序连接 MongoDB 也要设置 [[MongoDB#连接字符串]]。我们可以通过环境变量进行设置，避免在代码中硬编码用户凭据。下图展示一个 MongoDB 的连接字符串的各个部分

![[Pasted image 20240816191134.png|900]]

## 连接

要连接到 MongoDB，必须创建一个客户端。客户端可管理您的连接并运行数据库命令。

> [!tip] 客户端重用
> 
> **在会话和操作中重用客户端**。可以 **使用同一 Client 实例来执行多项任务**，而不是每次都创建一个新实例。**_Client 类型可以安全地被多个 goroutine 并发使用_**

通过向 `Connect()` 方法传递 `ClientOptions` 对象，可以创建使用连接字符串和其他客户端选项的客户端

```go
mongo.Connect(context.TODO(), connectOptions)
```

要指定连接 URI，可将其传递给 `ClientOptions` 对象的 `ApplyURI()` 方法，该方法会返回一个新的 `ClientOptions` 实例

```go
// 准备连接选项
connectOptions := options.Client()
connectOptions.ApplyURI(uri)
```

下表列出来几种常见的 MongoDB 连接和身份验证选项。可以将连接选项作为连接 `URI` 的参数传递，以指定客户端的行为。 更多信息参考 [连接字符串选项](https://www.mongodb.com/zh-cn/docs/manual/reference/connection-string/#connection-string-options)

| 选项名                          | 类型                  | 默认值     | 描述                                                                                                 |
| ---------------------------- | ------------------- | ------- | -------------------------------------------------------------------------------------------------- |
| **timeoutMS**                | `integer`           | `null`  | 指定 `Client`上运行的单个操作在返回超时错误前所需的 **毫秒数**。只有在操作 "下文"上没有截止时间时，操作才遵从会此设置                                |
| **connectTimeoutMS**         | `integer`           | `30000` | 指定 TCP 连接超时前的等待毫秒数                                                                                 |
| **maxPoolSize**              | `integer`           | `100`   | 指定连接池在给定时间内的最大连接数                                                                                  |
| **maxPoolSize**              | `integer`           | `100`   | 指定连接池在给定时间内的最大连接数                                                                                  |
| **replicaSet**               | `string`            | `null`  | 指定群集的副本集名称。副本集中的所有节点必须具有相同的副本集名称，否则客户端不会将它们视为副本集的一部分。                                              |
| **maxIdleTimeMS**            | `integer`           | `0`     | 指定连接被移除和关闭前在连接池中 **闲置的最长时间**。默认值为 **"0"**，这意味着连接可以 **无限期地保持闲置**。                                   |
| **minPoolSize**              | `integer`           | `0`     | 指定驱动程序在单个连接池中维护的最小连接数                                                                              |
| **socketTimeoutMS**          | `integer`           | `0`     | 指定在返回网络错误前等待套接字读取或写入的毫秒数。默认值 **"0"** 表示没有超时                                                        |
| **serverSelectionTimeoutMS** | `integer`           | `30000` | 指定等待找到合适服务器执行操作的毫秒数                                                                                |
| **heartbeatFrequencyMS**     | `integer`           | `10000` | 指定后台服务器定期检查之间的等待毫秒数                                                                                |
| **tls**                      | `boolean`           | `false` | 指定是否与实例建立传输层安全 (TLS) 连接。**在连接字符串中使用 DNS 种子列表 (SRV) 时，该值会自动设置为 "true"**。您可以通过将该值设置为 "false "来覆盖此行为。 |
| **w**                        | `string or integer` | `null`  | 指定写入关注点。要了解有关值的更多信息，请参阅服务器文档[写关注选项.](https://www.mongodb.com/docs/manual/reference/write-concern/) |
| **directConnection**         | `boolean`           | `false` | 指定是否向连接 URI 中指定的主机强制分派 **所有** 操作                                                                   |

以下代码展示了如何使用 `SetTimeout` 选项 **在客户端上设置超时选项**：

```go
opts := options.Client().SetTimeout(5 * time.Second)
```

下面的示例展示了如何 **使用 URI 选项设置单次超时**，并执行继承此设置的操作：

```go
uri := "mongodb://user:pass@sample.host:27017/?timeoutMS=5000"
```

下面的代码展示了如何创建一个使用 连接字符串和稳定 API 版本的客户端，连接到 MongoDB 并验证连接是否成功：

```go
package main

import (
	"context"
	"fmt"
	"go.mongodb.org/mongo-driver/mongo"
	"go.mongodb.org/mongo-driver/mongo/options"
	"log"
)

var client *mongo.Client

func init() {

	// mongodb://[username:password@]host1[:port1][,...hostN[:portN]][/[defaultauthdb][?options]]

	// 准备连接选项：连接选项是包含了 URI 的
	uri := "mongodb://localhost:27017/school"
	connectOptions := options.Client()
	connectOptions.ApplyURI(uri)
	connectOptions.SetMaxPoolSize(20)

	var err error

	// 创建客户端
	client, err = mongo.Connect(context.TODO(), connectOptions)
	if err != nil {
		log.Fatal(err)
	}
	// 注意，这是一个懒连接。只是创建好连接对象，并没有真的建立 TCP 连接

	// 通过 Ping 方法发起真正的连接
	if err := client.Ping(context.TODO(), nil); err != nil {
		log.Fatal(err)
	}

	fmt.Println("Connected to MongoDB!")
}
```

### 启用压缩

Go 驱动程序启用网络压缩。可以指定一个客户端选项来压缩消息，从而减少 MongoDB 和应用程序之间通过网络传递的数据量。

Go 驱动程序支持以下压缩算法：
1. [Snappy](https://google.github.io/snappy/): 在 MongoDB 3.4 及更高版本中可用。
2. [Zlib](https://zlib.net/): 在 MongoDB 3.6 及更高版本中可用。
3. [Zstandard](https://github.com/facebook/zstd/): 在 MongoDB 4.2 及更高版本中可用。

连接选项可以使用连接字符串指定，也可通过客户端选项对象的方法指定

```go
// 使用连接字符串
opts := options.Client().ApplyURI("mongodb://localhost:27017/?compressors=snappy,zlib,zstd")
client, _ := mongo.Connect(context.TODO(), opts)
```

```go
// 通过客户端选项
opts := options.Client().SetCompressors([]string{"snappy", "zlib", "zstd"})
client, _ := mongo.Connect(context.TODO(), opts)
```

> [!tip] 启用压缩需要安装依赖
> ```shell
> go get github.com/golang/snappy  # Snappy 算法
> 
> go get -u github.com/klauspost/compress # Zstd 
> 
>```
> 

要在应用程序中添加 `Zlib` 压缩算法，请 **导入内置的 `zlib` 包**。必须在使用 `Zlib` 压缩实例化客户端的应用程序文件中添加以下导入语句：

```go
import "compress/zlib"
```

### 启用 TLS

您可以通过以下方式之一在连接到 MongoDB 实例时启用 TLS

使用连接字符串启动 `TLS`

```go
uri := "mongodb://<hostname>:<port>?tls=true"
opts := options.Client().ApplyURI(uri)
client, _ := mongo.Connect(context.TODO(), opts)
```

使用 `ClientOptions` 启动 `TLS`

```go
uri := "<connection string>"
opts := options.Client().ApplyURI(uri).SetTLSConfig(&tls.Config{})
client, _ := mongo.Connect(context.TODO(), opts)
```

在执行 CURD 之前，还必须选中要操作的 **数据库** 和 **集合**

```go
db := client.Database("school") // 相当于 use school
collection := db.Collection("python") // 选中 db 中的 python 集合
```

## BSON

Go 驱动程序使用 BSON 与 MongoDB Server 之间进行数据交互

> [!tip]
> 
> 将 GO 类型的数据转换为 BSON 的过程称为**序列化(编组)**，反之称为**反序列化(解组)**


> [!tip] **MongoDB 以称为 BSON 的二进制表示形式存储文档**，可实现轻松灵活的数据处理

Go 驱动程序提供了四类主要驱动程序来处理 BSON 数据

+ `D` ：BSON 文档的有序表示（切片）
+ `M` ：BSON 文档的无序表示（映射）
+ `A` ：BSON 数组的有序表示形式
+ `E` ：D 类型中的单个元 `（key, value）`

### 结构体标记

在 Go 中，**结构体** 是具有已声明数据类型的数据字段的集合。可以使用 `struct Tag`（附加到 `struct` 字段的可选元数据）来修改 `struct` 字段的默认 `Marshalling` 和 `Unmarshalling` 行为。`struct Tag` 最常用的用途是在 BSON 文档中指定与 `struct` 字段相对应的字段名

下表介绍了可以在 Go 驱动程序中使用的其他 `struct` 标记：

| 结构标记        | 说明                                                                                                                                                |
| ----------- | ------------------------------------------------------------------------------------------------------------------------------------------------- |
| `omitempty` | 如果字段设置为与字段类型对应的零值，则不会对字段编组。                                                                                                                       |
| `minsize`   | 如果该字段的类型为 `int64`、`uint`、`uint32` 或 `uint64`，并且该字段的值适合采用有符号 `int32` 来表示，则该字段将被序列化为 BSON `int32` 而不是BSON `int64`。如果字段值不适合采用有符号 `int32` 来表示，则忽略此标签。 |
| `truncate`  | 如果字段类型是非浮点数字类型， 则解组到该字段中的 BSON 双精度数将在小数点处截断。                                                                                                      |
| `inline`    | 如果字段类型是结构或映射字段，则字段在编组时将扁平化， 而在解组时将取消扁平化。                                                                                                          |

> [!tip] 如果 **不指定结构体标记**，则 Go 驱动程序将 **使用以下规则编组结构体**
> **仅编组和解组已导出的字段**
> 
> 驱动程序使用相应结构体字段的小写字母生成 BSON 密钥
> 
> 此驱动程序会将嵌入的结构字段封送为子文档。每个键 均为该字段的小写类型表示法。
> 
> 如果指针不为 `nil`，则驱动程序会将指针字段作为底层类型。 如果指针为 `nil`，则驱动程序将其编组为 BSON `null` 值
> 
> 解组时，Go 驱动程序会针对 这些 `D/M` 类型映射 类型的字段遵循 `interface{}`。驱动程序将 BSON 文档解组为 `D` 类型，解组到 `interface{}` 字段中。
> 

下面的示例演示了 Go 驱动程序如何使用各种 `struct` 标记对结构体进行序列化

```go
type Address struct {
       Street string
       City   string
       State  string
}

type Student struct {
       FirstName string  `bson:"first_name,omitempty"`
       LastName  string  `bson:"last_name,omitempty"`
       Address   Address `bson:"inline"`
       Age       int
}

coll := client.Database("db").Collection("students")
address1 := Address{ "1 Lakewood Way", "Elwood City", "PA" }
student1 := Student{ FirstName : "Arthur", Address : address1, Age : 8}
_, err = coll.InsertOne(context.TODO(), student1)
```

相应的 BSON 表示法如下：

```json
{
  "_id" : ObjectId("..."),
  "first_name" : "Arthur",
  "street" : "1 Lakewood Way",
  "city" : "Elwood City",
  "state" : "PA",
  "age" : 8
}
```

下面的示例演示了 Go 驱动程序如何在不使用任何 `struct` 标记的情况下对 `struct` 进行序列化：

```go
type Address struct {
       Street string
       City   string
       State  string
}

type Student struct {
       FirstName string
       LastName  string
       Address   Address
       Age       int
}

coll := client.Database("db").Collection("students")
address1 := Address{ "1 Lakewood Way", "Elwood City", "PA" }
student1 := Student{ FirstName : "Arthur", Address : address1, Age : 8}
_, err = coll.InsertOne(context.TODO(), student1)
```

相应的 BSON 表示法如下：

```json
{
  "_id" : ObjectId("..."),
  "firstname" : "Arthur",
  "lastname" : "",
  "address": {
               "street" : "1 Lakewood Way",
               "city" : "Elwood City",
               "state" : "PA"
             },
  "age" : 8
}
```

### BSON 选项

可以指定 BSON 选项，以调整客户端实例的 `Marshalling` 和 `Unmarshalling` 行为。要在 `Client` 上设置 `BSON` 选项，请创建并配置 `BSONOptions` 实例。


通过配置以下设置创建 `BSONOptions` 实例：
+ 将 `UseJSONStructTags` 字段设置为 `true`，指示驱动程序在未指定 `"bson"` 结构标记的情况下使用` "json"` 结构标记
+ 将 `NilSliceAsEmpty` 字段设置为 `true`，指示驱动程序将 `nil` Go 切片转换为空 BSON 数组。

将 `BSONOptions` 实例传递给 `SetBSONOptions()` 辅助方法，以指定 `ClientOptions` 实例

创建客户端，以应用指定的 `BSON` 压缩和解压缩行为

```go
bsonOpts := &options.BSONOptions {
    UseJSONStructTags: true,  // true 代表在未使用 bson 标记时，使用 json 标记
    NilSliceAsEmpty: true, // nil 切片转换为 BSON 数组
}

clientOpts := options.Client().
    ApplyURI("<connection string>").
    SetBSONOptions(bsonOpts)

client, err := mongo.Connect(context.TODO(), clientOpts)
```

### 数据封装

可以将 MongoDB 的文档封装为一个 `struct` 或者一个 `map[string]interface{}`

> [!tip] 选择使用 `struct` 来组织文档，因为这会保留类型信息


```go
type User struct {
	ID   primitive.ObjectID `bson:"_id,omitempty"`
	Name string             `bson:"name"`
	Age  int                `bson:"age,minsize"`
}
```

> [!tip] ID 字段一定要设置为 `omitempty`
> 
> User 结构体中 ID 一定要使用 `omitempty`，**新增时结构体 ID 不设置则为零值**，提交时不会提交 ID，数据库自动生成 `_id`
> 
> 如果 User 结构体 ID 没有设置为 `omitemty`，就会将零值插入到文档中
> 

`ObjectId` 有 $12$ 字节组成，参考 `bson/primitive/objectid.go/NewObjectID()` 函数

> [!tip] ObjectID 的组成
> $4$ 字节时间戳
> 
> $5$ 字节进程唯一值
> 
> $3$ 字节随机数，每次加1

## CURD

### Create：插入文档

[插入文档](https://www.mongodb.com/zh-cn/docs/drivers/go/current/fundamentals/crud/write-operations/insert/) 的详细描述请参考官方文档

插入文档使用 `Collection` 的 `InsertOne()`（插入一条文档） 和 `InsertMany()`（插入多条文档） 两个方法完成

```go
func (coll *Collection) InsertOne(ctx context.Context, document interface{},
	opts ...*options.InsertOneOptions) (*InsertOneResult, error)

func (coll *Collection) InsertMany(ctx context.Context, documents []interface{},
	opts ...*options.InsertManyOptions) (*InsertManyResult, error)
```

#### 插入一条文档

`InsertOne()` 方法成功插入后，将返回一个包含新文档 `_id` 的 `InsertOneResult` 实例

```go
func InsertOneStudent(collection *mongo.Collection, student Student) {
	insertOneResult, err := collection.InsertOne(context.TODO(), student)
	if err != nil {
		log.Printf("Error inserting student: %v", err)
	}

	fmt.Printf("Inserted student ID: %v\n", insertOneResult.InsertedID)
}

InsertOneStudent(studentCollection, Student{
	FirstName: "Steven",
	LastName:  "King",
	Address: Address{
		Street: "2004 Charade Rd",
		City:   "Seattle",
		State:  "Washington",
	},
	Age: 27,
})
```

`InserOneOptions` 有一个选项 `BypassDocumentValidation`，这个选项用于开启 [文档校验](https://www.mongodb.com/zh-cn/docs/manual/core/schema-validation/)。默认为 `false`。构造一个`InsertOneOptions`，如下所示：

```go
opts := options.InsertOne().SetBypassDocumentValidation(true)
```

#### 插入多条文档

`InsertMany()` 方法成功插入后，会返回个 `InsertManyResult` 实例，其中包含了所有成功插入的文档 `_id`

```go
func InsertManyStudent(collection *mongo.Collection, students []*Student) {

	var documents []interface{}
	for _, student := range students {
		documents = append(documents, student)
	}

	insertManyResult, err := collection.InsertMany(context.TODO(), documents)
	if err != nil {
		log.Printf("Error inserting students: %v", err)
	}

	fmt.Println("Inserted students")

	for _, insertID := range insertManyResult.InsertedIDs {
		fmt.Printf("ID: %v\n", insertID)
	}
}

InsertManyStudent(studentCollection, []*Student{
	{
		FirstName: "Tom",
		LastName:  "Deved",
		Address: Address{
			Street: "宛平路100号",
			City:   "上海",
			State:  "中国",
		},
		Age: 18,
	},
	{
		FirstName: "Jerry",
		LastName:  "Deved",
		Address: Address{
			Street: "宛平路101号",
			City:   "上海",
			State:  "中国",
		},
		Age: 20,
	},
	{
		FirstName: "Tom",
		LastName:  "Jon",
		Address: Address{
			Street: "宛平路100号",
			City:   "上海",
			State:  "中国",
		},
		Age: 22,
	},
})
```

同 `InserOne()` 一样，也可以使用 `InserManyOptions` 选项修改插入的行为。用于 `InsertMany()` 的选项除了 `BypassDocumentValidation` 外，还有一个 `Ordered` 选项。`Ordered` 选项可以让 Go 驱动 **按文档提供的顺序发送文档**，默认为 `false`

构建 `InsertManyOptions` 的方法如下

```go
opts := options.InsertMany().SetBypassDocumentValidation(true).SetOrdered(false)
```

### Read：查找文档

查找一般需要使用 BSON 构建一些条件。MongoDB 提供的条件运算符比较复杂

下的表格中列出了用于构建条件的各种运算符

| 比较运算符                                                                                             | 描述       | BSON示例                                         |                       |
| ------------------------------------------------------------------------------------------------- | -------- | ---------------------------------------------- | --------------------- |
| [`$eq`](https://www.mongodb.com/docs/manual/reference/operator/query/eq/#mongodb-query-op.-eq)    | `=`      | `bson.M{"age": bson.M{"$eq": 10}}`             | `age = 10`            |
| [`$gt`](https://www.mongodb.com/docs/manual/reference/operator/query/gt/#mongodb-query-op.-gt)    | `>`      | `bson.M{"age": bson.M{"$gt": 17}}`             | `age > 17`            |
| [`$gte`](https://www.mongodb.com/docs/manual/reference/operator/query/gte/#mongodb-query-op.-gte) | `>=`     | `bson.M{"age": bson.M{"$gte": 18}}`            | `age >= 18`           |
| [`$lt`](https://www.mongodb.com/docs/manual/reference/operator/query/lt/#mongodb-query-op.-lt)    | `<`      | `bson.M{"age": bson.M{"$lt": 18}}`             | `age < 18`            |
| [`$lte`](https://www.mongodb.com/docs/manual/reference/operator/query/lte/#mongodb-query-op.-lte) | `<=`     | `bson.M{"age": bson.M{"$lte": 18}}`            | `age <= 18`           |
| [`$ne`](https://www.mongodb.com/docs/manual/reference/operator/query/ne/#mongodb-query-op.-ne)    | `!=`     | `bson.M{"age": bson.M{"$ne": 18}}`             | `age != 18`           |
| [`$in`](https://www.mongodb.com/docs/manual/reference/operator/query/in/#mongodb-query-op.-in)    | `in`     | `bson.M{"age": bson.M{"$nin": []int{15, 16}}}` | `age in [15, 16]`     |
| [`$nin`](https://www.mongodb.com/docs/manual/reference/operator/query/nin/#mongodb-query-op.-nin) | `not in` | `bson.M{"age": bson.M{"$nin": []int{15, 16}}}` | `age not in [15, 16]` |

|                                               逻辑运算符                                               | 描述  |                                   示例                                   |
| :-----------------------------------------------------------------------------------------------: | :-: | :--------------------------------------------------------------------: |
| [`$and`](https://www.mongodb.com/docs/manual/reference/operator/query/and/#mongodb-query-op.-and) |  与  | `bson.M{"$and": []bson.M{{"name": "tom"}, {"age": bson.M{"$gt":40}}}}` |
|  [`$or`](https://www.mongodb.com/docs/manual/reference/operator/query/or/#mongodb-query-op.-or)   |  或  | `bson.M{"$or": []bson.M{{"name": "tom"}, {"age": bson.M{"$lt":20}}}}`  |
| [`$not`](https://www.mongodb.com/docs/manual/reference/operator/query/not/#mongodb-query-op.-not) |  非  |          `bson.M{"age": bson.M{"$not": bson.M{"$gte": 20}}}`           |

| 运算符                                                                                                        | 描述                                                                                                          | 示例                                        |
| ---------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------- | ----------------------------------------- |
| [`$exists`](https://www.mongodb.com/docs/manual/reference/operator/query/exists/#mongodb-query-op.-exists) | 文档中是否有这个字段                                                                                                  | `bson.M{"Name": bson.M{"$exists": true}}` |
| [`$type`](https://www.mongodb.com/docs/manual/reference/operator/query/type/#mongodb-query-op.-type)       | 字段是否是指定的类型，[可用类型](https://www.mongodb.com/zh-cn/docs/manual/reference/operator/query/type/#available-types) | `bson.M{"age": bson.M{"$type": 16}}`      |

下面的示例展示了如何查询

```go
func Find(collection *mongo.Collection) []Student {
	filter := bson.M{"age": bson.M{"$gte": 20}} // age >= 20
	cursor, err := collection.Find(context.TODO(), filter)
	if err != nil {
		log.Printf("Error finding student: %v", err)
	}

	var results []Student

	if err := cursor.All(context.TODO(), &results); err != nil {
		log.Printf("Error finding student: %v", err)
	}
	return results
}

result := Find(studentCollection)
fmt.Printf("Student found: %v\n", result)
```

常用的几个查询的选项

#### 投影

```go
opt := options.Find()
opt.SetProjection(bson.M{"name": false, "age": false}) // name、age字段不投影，都显示为零值
opt.SetProjection(bson.M{"name": true}) // name投影，age字段零值显示
```

#### 排序

```go
opt.SetSort(bson.M{"age": 1}) // 升序
opt.SetSort(bson.M{"age": -1}) // 降序
```

#### 分页

```go
opt.SetSkip(1) // offset
opt.SetLimit(1) // limit
```

### Update：更新

下表列出来常用的更新操作符，详细的查看 [更新操作符](https://www.mongodb.com/zh-cn/docs/manual/reference/operator/update/) 

| 更新操作符   | 描述           | 示例                                     |
| :------ | ------------ | -------------------------------------- |
| `$inc`  | 对给定字段数字值增减   | `bson.M{"$inc": bson.M{"age": -5}}`    |
| `$set`  | 设置字段值，不存在则创建 | `bson.M{"$set": bson.M{"gender":"M"}}` |
| `$uset` | 移除字段         | `{'$unset':{'Name':""}}`               |

#### 更新一条文档

```go
// 更新一个
func updateOne() {
	filter := bson.M{"age": bson.M{"$exists": true}} // 所有有age字段的文档
	update := bson.M{"$inc": bson.M{"age": -5}} // age 字段减5
	ur, err := users.UpdateOne(context.TODO(), filter, update)
	if err != nil {
		log.Fatal(err)
	}
	fmt.Println(ur.MatchedCount, ur.ModifiedCount)
}
```

#### 更新多条文档

```go
// 更新多个
func updateMany() {
	filter := bson.M{"age": bson.M{"$exists": true}} // 所有有age字段的文档
	update := bson.M{"$set": bson.M{"gender": "M"}} // 为符合条件的文档设置 gender 字段
	users.UpdateMany(context.TODO(), filter, update)
}
```

#### 替换文档

替换 `_id` 字段以外的所有字段

```go
// 找到一批更新第一个，ReplaceOne 更新除 ID 以外所有字段
filter := bson.M{"age": bson.M{"$exists": true}} // 所有有age字段的文档
replacement := User{Name: "Sam", Age: 48}
ur, err := users.ReplaceOne(context.TODO(), filter, replacement)
if err != nil {
	log.Fatal(err)
}
fmt.Println(ur.MatchedCount, ur.ModifiedCount)
```

### Delete：删除

#### 删除一篇文档

```go
// 删除一个
func deleteOne() {
	filter := bson.M{} // 没有条件，匹配所有文档
	dr, err := users.DeleteOne(context.TODO(), filter)
	if err != nil {
		log.Fatal(err)
	}
	fmt.Println(dr.DeletedCount)
}
```


#### 删除多篇文档


```go
// 删除多个
func deleteMany() {
	filter := bson.M{} // 没有条件，匹配所有文档
	dr, err := users.DeleteMany(context.TODO(), filter)
	if err != nil {
		log.Fatal(err)
	}
	fmt.Println(dr.DeletedCount)
}
```
