---
sticker: emoji//1f343
---
# MongoDB

## 安装

参考 [install-mongodb-enterprise-on-ubuntu](https://www.mongodb.com/docs/manual/tutorial/install-mongodb-enterprise-on-ubuntu/)

从终端安装 `gnupg` 和 `curl`（如果尚未安装）：

```shell
sudo apt-get install gnupg curl
```

要导入 MongoDB 公共 GPG 密钥，请运行以下命令：

```shell
curl -fsSL https://www.mongodb.org/static/pgp/server-7.0.asc | \
   sudo gpg -o /usr/share/keyrings/mongodb-server-7.0.gpg \
   --dearmor
```

为 Ubuntu 22.04 (Jammy) 创建 `/etc/apt/sources.list.d/mongodb-org-7.0.list` 文件

```shell
echo "deb [ arch=amd64,arm64 signed-by=/usr/share/keyrings/mongodb-server-7.0.gpg ] https://repo.mongodb.org/apt/ubuntu jammy/mongodb-org/7.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-7.0.list
```

```shell
sudo apt-get update
```

### 安装 MongoDB 软件包

#### 安装最新

```shell
sudo apt-get install -y mongodb-org
```

#### 安装特定版本

要安装特定版本，必须单独指定每个组件包以及版本号，如以下示例所示

```shell
sudo apt-get install -y mongodb-org=7.0.7 mongodb-org-database=7.0.7 mongodb-org-server=7.0.7 mongodb-mongosh=7.0.7 mongodb-org-mongos=7.0.7 mongodb-org-tools=7.0.7
```

如果您仅安装`mongodb-org=7.0.7`并且不包含组件包，则无论您指定哪个版本，都将安装每个 MongoDB 包的最新版本

#### 固定当前安装的版本（可选）

可选。虽然您可以指定任意可用版本的 MongoDB，但当有新版本可用时，`apt-get` 仍会升级这些包。要防止意外升级，可将此包固定到当前安装的版本

```shell
echo "mongodb-org hold" | sudo dpkg --set-selections
echo "mongodb-org-database hold" | sudo dpkg --set-selections
echo "mongodb-org-server hold" | sudo dpkg --set-selections
echo "mongodb-mongosh hold" | sudo dpkg --set-selections
echo "mongodb-org-mongos hold" | sudo dpkg --set-selections
echo "mongodb-org-tools hold" | sudo dpkg --set-selections
```

## 运行
### 目录

如果通过软件包管理器安装，则在安装过程中会创建数据目录 `/var/lib/mongodb` 和日志目录 `/var/log/mongodb`。

默认情况下，MongoDB 使用 `mongodb` 用户账户运行。如果更改运行 MongoDB 进程的用户，您还必须修改数据和日志目录以赋予该用户访问这些目录的权限。

### 配置文件

MongoDB 官方软件包有一份配置文件 (`/etc/mongod.conf`)。这些设置（如数据目录和日志目录规范）将在启动时生效。换言之，如果在 MongoDB 实例运行时更改该配置文件，则必须重新启动实例才能使更改生效

更多配置参考 [MongoDB-configuration-options](https://www.mongodb.com/docs/manual/reference/configuration-options/)

### 初始化系统

```shell
sudo systemctl daemon-reload
sudo systemctl enable mongod
sudo systemctl start mongod
```

### 解决无法启动问题

```shell
dyp@ubuntu:~$ sudo systemctl status mongod.service
× mongod.service - MongoDB Database Server
     Loaded: loaded (/lib/systemd/system/mongod.service; enabled; vendor preset: enabled)
     Active: failed (Result: exit-code) since Wed 2024-06-05 15:32:44 CST; 3h 40min ago
       Docs: https://docs.mongodb.org/manual
   Main PID: 1786605 (code=exited, status=1/FAILURE)
        CPU: 31ms

6月 05 15:32:44 ubuntu systemd[1]: Started MongoDB Database Server.
6月 05 15:32:44 ubuntu mongod[1786605]: {"t":{"$date":"2024-06-05T07:32:44.620Z"},"s":"I",  "c":"CONTROL",  "id":7484500, ">
6月 05 15:32:44 ubuntu mongod[1786605]: {"t":{"$date":"2024-06-05T07:32:44.621Z"},"s":"F",  "c":"CONTROL",  "id":20574,   ">
6月 05 15:32:44 ubuntu systemd[1]: mongod.service: Main process exited, code=exited, status=1/FAILURE
6月 05 15:32:44 ubuntu systemd[1]: mongod.service: Failed with result 'exit-code'.
```

这是由于 `/var/lib/mongodb` 和  `/var/log/mongodb` 目录 `mongodb` 用户没有权限写入。只需要给 `mongodb` 用户赋予写入嵌入即可

```shell
sudo chown -R mongodb:mongodb /var/lib/mongodb
sudo chown -R mongodb:mongodb /var/log/mongodb
```

再次执行

```shell
sudo systemctl start mongod
```

查看 `mongod` 的状态

```shell
dyp@ubuntu:~$ sudo systemctl status mongod.service
● mongod.service - MongoDB Database Server
     Loaded: loaded (/lib/systemd/system/mongod.service; enabled; vendor preset: enabled)
     Active: active (running) since Wed 2024-06-05 19:18:28 CST; 13s ago
       Docs: https://docs.mongodb.org/manual
   Main PID: 1791546 (mongod)
     Memory: 170.1M
        CPU: 356ms
     CGroup: /system.slice/mongod.service
             └─1791546 /usr/bin/mongod --config /etc/mongod.conf

6月 05 19:18:28 ubuntu systemd[1]: Started MongoDB Database Server.
6月 05 19:18:28 ubuntu mongod[1791546]: {"t":{"$date":"2024-06-05T11:18:28.645Z"},"s":"I",  "c":"CONTROL",  "id":7484500, ">
lines 1-12/12 (END)
```

发现，`mongod` 以及完启动

## 基本概念

### NoSQL

NoSQL，指的是非关系型的数据库。**`NoSQL` 有时也称作 `Not Only SQL` 的缩写**，是对不同于传统的关系型数据库的数据库管理系统的统称。

**_NoSQL 用于超大规模数据的存储_**。（例如谷歌或 Facebook 每天为他们的用户收集万亿比特的数据）。这些类型的 **数据存储 _不需要固定的模式_**，**无需多余操作就可以横向扩展**。

今天我们可以通过第三方平台（如：Google,Facebook等）可以很容易的访问和抓取数据。用户的个人信息，社交网络，地理位置，用户生成的数据和用户操作日志已经成倍的增加。我们如果要对这些用户数据进行挖掘，那 SQL 数据库已经不适合这些应用了, NoSQL 数据库的发展却能很好的处理这些大的数据。

> [!tip] NoSQL 出现的原因
> 
> 用户的个人信息，社交网络，地理位置，用户数据和用户操作日志已经成倍的增加，那 SQL 数据库已经不适合这些应用了, NoSQL 数据库的发展却能很好的处理这些大的数据

### CAP 定理

在计算机科学中, **CAP定理**（CAP theorem）, 又被称作 **布鲁尔定理**（Brewer's theorem）, 它指出 _对于一个分布式计算系统来说，**不可能同时满足以下三点**_:
+ **一致性(Consistency)** (所有节点在同一时间具有相同的数据)
+ **可用性(Availability)** (保证每个请求不管成功或者失败都有响应)
+ **分区容错性(Partition tolerance)** (系统中任意信息的丢失或失败不会影响系统的继续运作)

CAP理论的核心是：一个分布式系统不可能同时很好的满足一致性，可用性和分区容错性这三个需求，最多只能同时较好的满足两个。

因此，根据 CAP 原理将 NoSQL 数据库分成了满足 CA 原则、满足 CP 原则和满足 AP 原则三 大类：

- **CA** - 单点集群，满足一致性，可用性的系统，通常在可扩展性上不太强大
- **CP** - 满足一致性，分区容忍性的系统，通常性能不是特别高
- **AP** - 满足可用性，分区容忍性的系统，通常可能对一致性要求低一些

![[Pasted image 20240605193350.png|900]]

### BASE

BASE：Basically Available, Soft-state, Eventually Consistent。 由 Eric Brewer 定义。

CAP 理论的核心是：一个分布式系统不可能同时很好的满足一致性，可用性和分区容错性这三个需求，最多只能同时较好的满足两个。

BASE 是 NoSQL 数据库通常对可用性及一致性的弱要求原则:
- **B**asically **A**vailable --**_基本可用_**
- **S**oft-state --**_软状态/柔性事务_**。 "Soft state" 可以理解为 "无连接" 的, 而 "Hard state" 是"面向连接"的
- **E**ventually Consistency -- **最终一致性**， 也是 ACID 的最终目的

### MongoDB

MongoDB 是一个 **文档型数据库**，数据以 **类似 JSON 的文档形式存储**。

MongoDB 的设计理念是为了 **应对大数据量、高性能和灵活性需求**。

MongoDB使用集合（Collections）来组织文档（Documents），每个文档都是由键值对组成的。
- **数据库（Database）**：存储数据的容器，类似于关系型数据库中的数据库。
- **集合（Collection）**：数据库中的一个集合，类似于关系型数据库中的表。
- **文档（Document）**：集合中的一个数据记录，类似于关系型数据库中的行（row），以 BSON 格式存储。

MongoDB 将数据存储为一个文档，数据结构由键值`(key=>value)`对组成，**文档类似于 JSON 对象**，字段值可以包含其他文档，数组及文档数组：

![[Pasted image 20240605193741.png|900]]

### MongoDB 概念解析

不管我们学习什么数据库都应该学习其中的基础概念，在 MongoDB 中基本的概念是 **文档**、**集合**、**数据库**，下面我们挨个介绍。

下表将帮助您更容易理解 MongoDB 中的一些概念

| SQL 术语/概念     | MongoDB 术语/概念 | 解释/说明                   |
| ------------- | ------------- | ----------------------- |
| `database`    | `database`    | 数据库                     |
| `table`       | `collection`  | 数据库表/集合                 |
| `row`         | `document`    | 数据记录行/文档                |
| `column`      | `field`       | 数据字段/域                  |
| `index`       | `index`       | 索引                      |
| `table joins` |               | 表连接,MongoDB 不支持         |
| `primary key` | `primary key` | 主键,MongoDB自动将_id字段设置为主键 |

通过下图实例，我们也可以更直观的了解 Mongo 中的一些概念

![[Pasted image 20240605193912.png|900]]
MongoDB 中可以创建使用多个库，但有一些数据库名是保留的，可以直接访问这些有特殊作用的数据库。
- `admin`: 从权限的角度来看，**这是 `"root"` 数据库**。要是将一个用户添加到这个数据库，这个用户自动继承所有数据库的权限。一些特定的服务器端命令也只能从这个数据库运行，比如列出所有的数据库或者关闭服务器
- `local`: 这个数据永远不会被复制，可以用来 **存储限于本地单台服务器的任意集合**
- `config`: 当 Mongo 用于分片设置时，`config` 数据库在内部使用，用于保存分片的相关信息

## 连接

可以使用 [MongoDB shell](https://www.mongodb.com/try/download/shell) 来连接 MongoDB 服务器。也可以使用 Go 来连接 MongoDB 服务器。现在使用 MongoDB shell 来连接 Mongodb 服务，之后的我们将会介绍如何通过 Go 来连接 MongoDB 服务。

### 连接字符串

标准 `URI` 连接方案的格式为

```shell
mongosh mongodb://[username:password@]host1[:port1][,...hostN[:portN]][/[defaultauthdb][?options]]
```

> [!tip] `mongosh` 是 MongoDB Server 的命令行客户端

连接字符串包括以下组件


|               组件                | 描述                                                                                                                                        |
| :-----------------------------: | :---------------------------------------------------------------------------------------------------------------------------------------- |
| `mongodb:// `或 `mongodb+srv://` | 准连接字符串格式一个必需的前缀                                                                                                                           |
|      `username:password@`       | 可选的。身份验证凭据。                                                                                                                               |
|          `host[:port]`          | `mongod` 实例(或者分片集群中的 `mongos` 实例)运行的主机(以及可选的端口号)。默认端口 `27017` 被使用                                                                         |
|        `/defaultauthdb`         | 可选的。如果连接字符串包含 `username:password@` 身份验证凭据，但未指定 `authSource` 选项，则使用身份验证数据库。如果未指定 `authSource` 和 `defaultauthdb`，客户端将尝试向 `admin` 数据库验证指定的用户 |
|          `?<options>`           | 可选的。一个查询字符串，指定连接特定的选项为 `<name>=<value>` 对。有关这些选项的完整描述                                                                                     |

下表列出来一些选项

| 选项                      | 描述                                                                                                                                                                                                                         |
| ----------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `replicaSet=name`       | 验证 `replica set `的名称。 `Impliesconnect=replicaSet`.                                                                                                                                                                         |
| `slaveOk=true`\|`false` | - `true`:在 `connect=direct` 模式下，驱动会连接第一台机器，即使这台服务器不是主。在 `connect=replicaSet` 模式下，驱动会发送所有的写请求到主并且把读取操作分布在其他从服务器。<br>- `false`: 在` connect=direct` 模式下，驱动会自动找寻主服务器. 在 `connect=replicaSet` 模式下，驱动仅仅连接主服务器，并且所有的读写命令都连接到主服务器。 |
| `safe=true`\|`false`    | - `true`: 在执行更新操作之后，驱动都会发送 `getLastError` 命令来确保更新成功。(还要参考 `wtimeoutMS`).<br>- `false`: 在每次更新之后，驱动不会发送 `getLastError` 来确保更新成功。                                                                                              |
| `w=n`                   | 驱动添加 `{ w : n } `到 `getLastError` 命令. 应用于 `safe=true`。                                                                                                                                                                     |
| `wtimeoutMS=ms`         | 驱动添加 `{ wtimeout : ms } `到 `getlasterror` 命令. 应用于 `safe=true`.                                                                                                                                                             |
| `fsync=true`\|`false`   | - `true`: 驱动添加 `{ fsync : true }` 到 `getlasterror` 命令.应用于` safe=true`.<br>- `false`: 驱动不会添加到 `getLastError` 命令中。                                                                                                           |
| `journal=true`\|`false` | 如果设置为 `true`, 同步到 `journal` (在提交到数据库前写入到实体中). 应用于 `safe=true`                                                                                                                                                              |
| `connectTimeoutMS=ms`   | 可以打开连接的时间。                                                                                                                                                                                                                 |
| `socketTimeoutMS=ms`    | 发送和接受 `sockets` 的时间。                                                                                                                                                                                                       |

### 示例

使用默认端口来连接 MongoDB 的服务

```shell
mongodb://localhost
```

使用用户 `admin` 使用密码 `123456` 连接到本地的 `MongoDB` 服务上

```shll
mongodb://admin:123456@localhost/
```

使用用户名和密码连接登录到指定数据库，格式如下：

```shell
mongodb://admin:123456@localhost/test
```

连接 `replica pair`, 服务器1为 `example1.com` 服务器2为 `example2`。

```shell
mongodb://example1.com:27017,example2.com:27017
```

连接 `replica set` 三台服务器 (端口 27017, 27018, 和27019):

```shell
mongodb://localhost,localhost:27018,localhost:27019
```

连接 `replica set` 三台服务器, 写入操作应用在主服务器 并且分布查询到从服务器

```shell
mongodb://host1,host2,host3/?slaveOk=true
```

直接连接第一个服务器，无论是 `replica set` 一部分或者主服务器或者从服务器。
```shell
mongodb://host1,host2,host3/?connect=direct;slaveOk=true
```

安全模式连接到 `localhost`:

```shell
mongodb://localhost/?safe=true
```

以安全模式连接到 `replica set`，并且等待至少两个复制服务器成功写入，超时时间设置为2秒

```shell
mongodb://host1,host2,host3/?safe=true;w=2;wtimeoutMS=2000
```

## 库操作

在MongoDB中，数据库的创建是一个简单的过程，当你 **首次向 MongoDB 中插入数据时，如果数据库不存在，MongoDB会自动创建它**。

我们只需选择一个数据库名称，并开始向其中插入文档即可

### 查看已存在的数据库

```js
test> show dbs  // 查看已存在的数据库
admin   40.00 KiB
config  60.00 KiB
local   72.00 KiB
```

### 创建/切换数据库

当你使用 `use` 命令来指定一个数据库时，如果该数据库不存在，MongoDB 将自动创建它。

`MongoDB` 创建数据库的语法格式如下：

```js
use DATABASE_NAME
```

如果数据库不存在，则创建数据库，否则切换到指定数据库

```js
test> use school  // 创建 school 数据库，并切换到 school 数据库
switched to db school
school>
```

执行 `use school` 命令后，MongoDB 将创建名为 `school` 的新数据库。此时，你可以开始在这个数据库中创建集合和插入文档。

如果你想查看所有数据库，可以使用 **show dbs** 命令：

```js
school> show dbs
admin    40.00 KiB
config  108.00 KiB
local    72.00 KiB
```

可以看到，我们刚创建的数据库 `school` 并不在数据库的列表中， 要显示它，我们需要向 `school` 数据库插入一些数据

```js
school> db.school.insertOne({"name":"上海应用技术大学"})  // 插入数据
{
  acknowledged: true,
  insertedId: ObjectId('66605b7a9aaae5e47ba26a13')
}
school> show dbs  // 查看数据库
admin    40.00 KiB
config  108.00 KiB
local    72.00 KiB
school    8.00 KiB  // 我们创建的数据库出现在这里了
```

### 删除数据库

如果需要删除数据库，可以使用 `db.dropDatabase()` 方法：

```js
use myDatabase
db.dropDatabase()
```

上述命令将删除当前正在使用的 `myDatabase` 数据库及其所有集合

### 默认数据库

MongoDB 中 **默认的数据库为 `test`**， 如果你没有创建新的数据库，集合将存放在 `test` 数据库中。

当您通过 shell 连接到 MongoDB 实例时，如果 [[#连接字符串]] 中未指定数据库 并且未使用 `use` 命令切换到其他数据库，则会默认使用 `test` 数据库。

例如，在启动 MongoDB 实例并连接到 MongoDB shell 后，如果您开始插入文档而未显式指定数据库，MongoDB 将默认使用 test 数据库

```js
use test
db.myCollection.insertOne({ name: "Alice", age: 30 })
```

> [!attention] 注意
> 
> 数据库名不能包含空格 点(`.`) 或美元符号(`$`)
> 
> **数据库的创建是自动的，不需要显式创建**，除非你需要在创建时指定特定的配置选项。
> 
> 在 MongoDB 中，只有在数据库中至少有一个集合时，数据库才会在 `show dbs` 命令的输出中显示

## 集合操作

### 创建集合

MongoDB 中使用 **createCollection()** 方法来创建集合

```js
db.createCollection(name, options)
```

参数说明：
- `name`: 要创建的集合名称
- `options`: 可选参数, 指定有关内存大小及索引的选项

`options` 可以是如下参数

| 参数名                | 类型  | 描述                                                                                                           | 示例值                             |
| ------------------ | --- | ------------------------------------------------------------------------------------------------------------ | ------------------------------- |
| `capped`           | 布尔值 | 是否创建一个固定大小的集合                                                                                                | `true`                          |
| `size`             | 数值  | 集合的最大大小（以字节为单位）。仅在 `capped` 为 `true` 时有效。                                                                    | `10485760` (`10MB`)             |
| `max`              | 数值  | 集合中允许的最大文档数。仅在 `capped` 为 `true` 时有效                                                                         | `5000`                          |
| `validator`        | 对象  | 用于文档验证的表达式                                                                                                   | `{ $jsonSchema: { ... }}`       |
| `validationLevel`  | 字符串 | 指定文档验证的严格程度<br>- `"off"`：不进行验证<br>- `"strict"`：插入和更新操作都必须通过验证（默认）<br>- `"moderate"`：仅现有文档更新时必须通过验证，插入新文档时不需要 | `"strict"`                      |
| `validationAction` | 字符串 | 指定文档验证失败时的操作<br>- `"error"`：阻止插入或更新（默认）<br>- `"warn"`：允许插入或更新，但会发出警告                                         | `"error"`                       |
| `storageEngine`    | 对象  | 为集合指定存储引擎配置                                                                                                  | `{ wiredTiger: { ... }}`        |
| `collation`        | 对象  | 指定集合的默认排序规则                                                                                                  | `{ locale: "en", strength: 2 }` |

在插入文档时，`MongoDB` 首先检查固定集合的 `size` 字段，然后检查 `max` 字段。

使用这些选项创建一个集合的实例：

```js
db.createCollection("myComplexCollection", {
  capped: true,
  size: 10485760,
  max: 5000,
  validator: { $jsonSchema: {
    bsonType: "object",
    required: ["name", "email"],
    properties: {
      name: {
        bsonType: "string",
        description: "必须为字符串且为必填项"
      },
      email: {
        bsonType: "string",
        pattern: "^.+@.+$",
        description: "必须为有效的电子邮件地址"
      }
    }
  }},
  validationLevel: "strict",
  validationAction: "error",
  storageEngine: {
    wiredTiger: { configString: "block_compressor=zstd" }
  },
  collation: { locale: "en", strength: 2 }
});
```

这个例子创建了一个集合，具有以下特性：
- 固定大小，最大 10MB，最多存储 5000 个文档
- 文档必须包含 `name` 和 `email` 字段，其中 `name` 必须是字符串，`email` 必须是有效的电子邮件格式
- 验证级别为严格，验证失败将阻止插入或更新
- 使用 `WiredTiger` 存储引擎，指定块压缩器为 `zstd`
- 默认使用英语排序规则

### 更新集合名字

在 MongoDB 中，不能直接通过命令来重命名集合。

MongoDB 可以使用 **renameCollection** 方法来来重命名集合。

**renameCollection 方法在 MongoDB 的 admin 数据库中运行**，可以将一个集合重命名为另一个名称

renameCollection 命令的语法：

```js
db.adminCommand({
  renameCollection: "sourceDb.sourceCollection",
  to: "targetDb.targetCollection",
  dropTarget: <boolean>
})
```

**参数说明：**
- **renameCollection**：要重命名的集合的完全限定名称（包括数据库名）
- **to**：目标集合的完全限定名称（包括数据库名）
- **dropTarget**（可选）：布尔值。如果目标集合已经存在，是否删除目标集合。默认值为 `false`

> [!attention] 
> **权限要求**：执行 `renameCollection` 命令需要具有对源数据库和目标数据库的适当权限。通常需要 `dbAdmin` 或 `dbOwner` 角色
> 
> **目标集合不存在**：目标集合不能已经存在。如果目标集合存在，则会返回错误
> 
>**索引和数据**：重命名集合会保留所有文档和索引。

### 删除集合

MongoDB 中使用 **drop()** 方法来删除集合。

**drop()** 方法可以 **永久地从数据库中删除指定的集合及其所有文档**，这是一个 **不可逆** 的操作，因此需要谨慎使用。

```js
db.collectionName.drop()
```

## 文档操作

### 插入

#### 插入单个文档 `db.collectionName.insertOne() `

以下示例将一个新文档插入到库存集合中。如果文档没有指定 `_id` 字段，MongoDB 会将 `_id` 字段添加到新文档，并赋予其一个 `ObjectId` 值

```js
school> db.school.insertOne({ item: "canvas", qty: 100, tags: ["cotton"], size: { h: 28, w: 35.5, uom: "cm" } })
// 返回结果
{
  acknowledged: true,
  insertedId: ObjectId('666060c29aaae5e47ba26a14')
}
```

`insertOne()` 返回一个文档，其中包含新插入文档的 `_id` 字段值有关

#### 插入多个文档 `db.collectionName.insertMany()`

下面的示例将三个新文档插入到 `school` 集合中。如果文档没有指定 `_id` 字段，MongoDB会将带有 `ObjectId` 值的 `_id` 字段添加到每个文档中

```js
school> db.school.insertMany([
...    { item: "journal", qty: 25, tags: ["blank", "red"], size: { h: 14, w: 21, uom: "cm" } },
...    { item: "mat", qty: 85, tags: ["gray"], size: { h: 27.9, w: 35.5, uom: "cm" } },
...    { item: "mousepad", qty: 25, tags: ["gel", "blue"], size: { h: 19, w: 22.85, uom: "cm" } }
... ])
// 返回结果
{
  acknowledged: true,
  insertedIds: {
    '0': ObjectId('6660619a9aaae5e47ba26a15'),
    '1': ObjectId('6660619a9aaae5e47ba26a16'),
    '2': ObjectId('6660619a9aaae5e47ba26a17')
  }
}
```

`insertMany()` 返回一个文档，其中包含新插入的文档的 `_id` 字段的值

> [!tip] 插入行为
> 
> 如果 **集合当前不存在**，则插入操作将 **创建集合**
> 
> 在 MongoDB 中，存储在集合中的每个文档都需要一个 **唯一的 `_id` 字段作为主键**。如果插入的文档省略了 `_id` 字段，MongoDB 驱动会 **自动** 为 `_id`字段生成一个 `ObjectId`
> 
> MongoDB 中所有的 **写操作在单个文档层面上都是原子的**
> 
> 对于写操作，可以指定 MongoDB 对写操作请求的确认级别

### 查询

#### 无条件查询

```js
db.connectionName.find()   // 无条件查询, 查询出所有数据
db.connectionName.find().pretty()  //  查询出所有数据并格式化显示
```

**无条件查询示例**

```shell
school> db.school.find( {} )
[
  { _id: ObjectId('66605b7a9aaae5e47ba26a13'), name: '上海应用技术大学' },
  {
    _id: ObjectId('666060c29aaae5e47ba26a14'),
    item: 'canvas',
    qty: 100,
    tags: [ 'cotton' ],
    size: { h: 28, w: 35.5, uom: 'cm' }
  },
  {
    _id: ObjectId('6660619a9aaae5e47ba26a15'),
    item: 'journal',
    qty: 25,
    tags: [ 'blank', 'red' ],
    size: { h: 14, w: 21, uom: 'cm' }
  },
  {
    _id: ObjectId('6660619a9aaae5e47ba26a16'),
    item: 'mat',
    qty: 85,
    tags: [ 'gray' ],
    size: { h: 27.9, w: 35.5, uom: 'cm' }
  },
  {
    _id: ObjectId('6660619a9aaae5e47ba26a17'),
    item: 'mousepad',
    qty: 25,
    tags: [ 'gel', 'blue' ],
    size: { h: 19, w: 22.85, uom: 'cm' }
  }
]
```

#### 条件查询
查询过滤器文档可以使用 **查询操作符** 来指定下列形式的条件

```js
{ <field1>: { <operator1>: <value1> }, ... }
```

`operatior` 参考 [query-selectors](https://www.mongodb.com/docs/manual/reference/operator/query/#query-selectors)


| 比较运算符                                                                                             | 描述       | 示例                            |                       |
| ------------------------------------------------------------------------------------------------- | -------- | ----------------------------- | --------------------- |
| [`$eq`](https://www.mongodb.com/docs/manual/reference/operator/query/eq/#mongodb-query-op.-eq)    | `=`      | `{"age": {"$eq": 10}}`        | `age = 10`            |
| [`$gt`](https://www.mongodb.com/docs/manual/reference/operator/query/gt/#mongodb-query-op.-gt)    | `>`      | `{"age": {"$gt": 17}}`        | `age > 17`            |
| [`$gte`](https://www.mongodb.com/docs/manual/reference/operator/query/gte/#mongodb-query-op.-gte) | `>=`     | `{"age": {"$gte": 18}}`       | `age >= 18`           |
| [`$lt`](https://www.mongodb.com/docs/manual/reference/operator/query/lt/#mongodb-query-op.-lt)    | `<`      | `{"age": {"$lt": 18}}`        | `age < 18`            |
| [`$lte`](https://www.mongodb.com/docs/manual/reference/operator/query/lte/#mongodb-query-op.-lte) | `<=`     | `{"age": {"$lte": 18}}`       | `age <= 18`           |
| [`$ne`](https://www.mongodb.com/docs/manual/reference/operator/query/ne/#mongodb-query-op.-ne)    | `!=`     | `{"age": {"$ne": 18}}`        | `age != 18`           |
| [`$in`](https://www.mongodb.com/docs/manual/reference/operator/query/in/#mongodb-query-op.-in)    | `in`     | `{"age": {"$nin": [15, 16]}}` | `age in [15, 16]`     |
| [`$nin`](https://www.mongodb.com/docs/manual/reference/operator/query/nin/#mongodb-query-op.-nin) | `not in` | `{"age": {"$nin": [15, 16]}}` | `age not in [15, 16]` |

|                                               逻辑运算符                                               | 描述  |                          示例                          |                         |
| :-----------------------------------------------------------------------------------------------: | :-: | :--------------------------------------------------: | ----------------------- |
| [`$and`](https://www.mongodb.com/docs/manual/reference/operator/query/and/#mongodb-query-op.-and) |  与  | `{"and": [{"age": {"$gt": 15}},{"age":{"$lt":25}}]}` | `age > 15 and age < 25` |
|  [`$or`](https://www.mongodb.com/docs/manual/reference/operator/query/or/#mongodb-query-op.-or)   |  或  | `{"or": [{"age": {"$gt": 15}},{"age":{"$lt":25}}]}`  | `age > 15 or age < 25`  |
| [`$nor`](https://www.mongodb.com/docs/manual/reference/operator/query/nor/#mongodb-query-op.-nor) |     | `{"nor": [{"age": {"$gt": 15}},{"age":{"$lt":25}}]}` |                         |
| [`$not`](https://www.mongodb.com/docs/manual/reference/operator/query/not/#mongodb-query-op.-not) |  非  |            `{"age": {"$not":{"$lt":25}}}`            |                         |

| 运算符                                                                                                        | 描述                                                                                                          | 示例                              |
| ---------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------- | ------------------------------- |
| [`$exists`](https://www.mongodb.com/docs/manual/reference/operator/query/exists/#mongodb-query-op.-exists) | 文档中是否有这个字段                                                                                                  | `{ "name": { $exists: true } }` |
| [`$type`](https://www.mongodb.com/docs/manual/reference/operator/query/type/#mongodb-query-op.-type)       | 字段是否是指定的类型，[可用类型](https://www.mongodb.com/zh-cn/docs/manual/reference/operator/query/type/#available-types) | `{"age": {"$type": 16}}`        |


**查询示例**

```js
db.student.find( { age: {$lt: 20}) } )  // 查询出 age<20 的学生

db.student.find( {$and:[{age:{$lt: 20}}, {age: {$gt:18}}]} ) // 查询出18 < age < 20的学生

db.student.find({$or:[{age:{$lt: 20}}, {age: {$gt:18}}]})  // 查询出 age < 20 或 age > 18 的学生
```

> [!tip] 详细查询操作符，参考 [查询选择器](https://www.mongodb.com/zh-cn/docs/manual/reference/operator/query/#query-selectors)

#### 投影

选择时可以进行投影：指定那些字段选择，那些字段不选择。参见 [投影操作符](https://www.mongodb.com/zh-cn/docs/manual/reference/operator/query/#projection-operators)

| 投影运算符                                                                                                                                 | 描述                                                                                                                                                            |
| :------------------------------------------------------------------------------------------------------------------------------------ | :------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| [`$`](https://www.mongodb.com/zh-cn/docs/manual/reference/operator/projection/positional/#mongodb-projection-proj.-)                  | 对数组中与查询条件匹配的第一个元素进行投影                                                                                                                                         |
| [`$elemMatch`](https://www.mongodb.com/zh-cn/docs/manual/reference/operator/projection/elemMatch/#mongodb-projection-proj.-elemMatch) | 对数组中与指定 [`$elemMatch`](https://www.mongodb.com/zh-cn/docs/manual/reference/operator/projection/elemMatch/#mongodb-projection-proj.-elemMatch) 条件匹配的第一个元素进行投影。 |
| [`$meta`](https://www.mongodb.com/zh-cn/docs/manual/reference/operator/aggregation/meta/#mongodb-expression-exp.-meta)                | 对`$text`操作期间分配的文档分数进行投影                                                                                                                                       |
| [`$slice`](https://www.mongodb.com/zh-cn/docs/manual/reference/operator/projection/slice/#mongodb-projection-proj.-slice)             | 限制从数组中投影的元素数量。支持跳过切片和对切片进行数量限制。                                                                                                                               |

### 更新

更新通用涉及各种运算符，参见 [更新操作符](https://www.mongodb.com/zh-cn/docs/manual/reference/operator/update/)

#### 更新单个文档 `db.collectionName.updateOne()`

```js
db.collection.updateOne(
   <filter>,
   <update>,
   {
     upsert: <boolean>,
     writeConcern: <document>,
     collation: <document>,
     arrayFilters: [ <filterdocument1>, ... ],
     hint:  <document|string>,
     let: <document>
   }
)
```


**参数说明**。详细参见 [updateOne-parameters](https://www.mongodb.com/docs/manual/reference/method/db.collection.updateOne/#parameters)
 -   **filter** : `update` 的查询条件，类似`sql update`查询内 `where` 后面的
 -   **update** : `update` 的对象和一些更新的操作符（如`$,$`inc...）等，也可以理解为 `sql update` 查询内 `set` 后面的
 -   **upsert** : 可选，这个参数的意思是，如果不存在 `update` 的记录，是否插入 `objNew` ,
	 - `true` 为插入，默认是 `false`，不插入
 -   **multi** : 可选，mongodb 默认是 `false`,**只更新找到的第一条记录**
	- 如果这个参数为 `true`, 就把按条件查出来多条记录全部更新
 -   **writeConcern** :可选，抛出异常的级别

**示例**

```js
db.inventory.updateOne(
   { item: "paper" },
   {
     $set: { "size.uom": "cm", status: "P" },
     $currentDate: { lastModified: true }
   }
)
```

- 使用 `$set` 操作符将 `size.uom` 字段的值更新为 `"cm"`，将 `status` 字段的值更新为 `"P"`
- 使用 `$currentDate` 操作符将 `lastModified` 字段的值更新为当前日期。如果 `lastModified` 字段不存在，`$currentDate` 将创建该字段

#### 更新多个文档 `db.collectionName.updateMany()`

```js
db.collection.updateMany(
   <filter>,
   <update>,
   {
     upsert: <boolean>,
     writeConcern: <document>,
     collation: <document>,
     arrayFilters: [ <filterdocument1>, ... ],
     hint:  <document|string>,
     let: <document>
   }
)
```

参数说明参考 [updateMany-parameters](https://www.mongodb.com/docs/manual/reference/method/db.collection.updateMany/#parameters)

**示例**

```js
db.inventory.updateMany(
   { "qty": { $lt: 50 } },
   {
     $set: { "size.uom": "in", status: "P" },
     $currentDate: { lastModified: true }
   }
)
```

 使用 `$set` 操作符将 `size.uom` 字段的值更新为 `"in"`，将 `status` 字段的值更新为 `"P"`

 使用 `$currentDate` 操作符将 `lastModified` 字段的值更新为当前日期。如果 `lastModified` 字段不存在，`$currentDate` 将创建该字段

#### 替换文档 `db.collectionName.replaceOne()`

```js
db.collection.replaceOne(
   <filter>,
   <replacement>,
   {
      upsert: <boolean>,
      writeConcern: <document>,
      collation: <document>,
      hint: <document|string>
   }
)
```

详细参考 [replaceOne-syntax](https://www.mongodb.com/docs/manual/reference/method/db.collection.replaceOne/#syntax)


```js
db.inventory.replaceOne(
   { item: "paper" },
   { item: "paper", instock: [ { warehouse: "A", qty: 60 }, { warehouse: "B", qty: 40 } ] }
)
```

### 删除

#### 删除多条 `db.collectionName.deleteMany()`

```js
db.collection.deleteMany(
   <filter>,
   {
      writeConcern: <document>,
      collation: <document>
   }
)
```

详细参考 [deleteMany](https://www.mongodb.com/docs/manual/reference/method/db.collection.deleteMany/#syntax)

#### 删除一条 `db.collection.deleteOne()`

```js
db.collection.deleteOne(
    <filter>,
    {
      writeConcern: <document>,
      collation: <document>,
      hint: <document|string>
    }
)
```

详细参考 [deleteOne](https://www.mongodb.com/docs/manual/reference/method/db.collection.deleteOne/#syntax)

