# PostgreSQL

PostgreSQL 是一个先进的、企业级的、开源的关系数据库系统。 PostgreSQL 支持 **SQL**（关系型）和 **JSON**（非关系型）查询。

> [!tip] 应用场景
> 
> LAPP 代表**L** inux、 **A** pache、 **PostgreSQL**和**P** HP（或 Python 和 Perl）。 PostgreSQL 主要用作强大的后端数据库，为许多 **动态网站** 和 **Web 应用程序** 提供支持。
> 
> 大型公司和初创公司都使用 PostgreSQL 作为主要数据库来支持其应用程序和产品
> 
> 具有 **PostGIS**  扩展的 PostgreSQL 支持地理信息系统 (GIS) 的地理空间数据库
> 
> 

PostgreSQL 支持流行的编程语言，包括但不限于 Python、Java、Go、C/C++ 等

## 示例数据库

PostgresSQL 官方提供了一个学习使用的示例数据库：[DVD 租赁数据库](https://www.postgresqltutorial.com/wp-content/uploads/2019/05/dvdrental.zip)

DVD 租赁数据库代表 DVD 租赁商店的业务流程。 DVD 租赁数据库有许多对象，包括
- $15$ 张 **表**
- $1$ 个 **触发器**
- $7$ 个 **视图**
- $8$ 个 **函数**
- $1$ 个 **域**
- $13$ 个 **序列**


![[Pasted image 20240809221215.png|900]]


DVD 租赁数据库中的 $15$ 张表总的内容如下

|       表明        | 存储内容                    |
| :-------------: | :---------------------- |
|     `actor`     | 存储演员数据，包括名字和姓氏。         |
|     `film`      | 存储电影数据，例如标题、发行年份、长度、评级等 |
|   `category`    | 存储电影的类别数据               |
| `film_category` | 存储电影和类别之间的关系            |
|     `store`     | 包含商店数据，包括经理人员和地址        |
|   `inventory`   | 存储库存数据                  |
|    `rental`     | 存储租赁数据                  |
|    `payment`    |  存储客户的付款                |
|     `staff`     | 存储员工数据                  |
|   `customer`    | 存储客户数据                  |
|    `address`    | 存储员工和客户的地址数据            |
|     `city`      | 存储城市名称                  |
|    `country`    | 存储国家/地区名称               |

## 安装 PostgreSQL

要在 Windows 上安装 PostgreSQL，请按照下列步骤操作
+ 首先，下载适用于 Windows 的 [PostgreSQL 安装程序](https://www.enterprisedb.com/downloads/postgres-postgresql-downloads)
+ 其次，使用安装程序安装 PostgreSQL。一路默认安装即可
+ 最后，将 PostgreSQL 的 bin 目录的路径添加到 PATH 环境变量中

也可以使用 scoop 安装工具安装

## 连接到 PostgreSQL 数据库服务器

连接数据库需要使用 `psql`，这是一个基于终端的应用程序。`psql` 命令的连接选项有下面几个

```
  -h, --host=主机名        数据库服务器主机或socket目录(默认："localhost")
  -p, --port=端口          数据库服务器的端口(默认："5432")
  -U, --username=用户名    指定数据库用户名
  -w, --no-password        永远不提示输入口令
  -W, --password           强制口令提示 (自动)
```

连接本地的 PostgreSQL 服务器最简单的命令为

```shell
psql -U postgres
```

然后，会提示输入密码。正确输入密码后，您将连接到 PostgreSQL 服务器。命令提示符将更改为如下所示

```
postgres=#
```

接下来就可以输入 SQL 语句并执行了。例如，**查看 PostgreSQL 的版本**

```sql
postgres=# SELECT version();
                          version
------------------------------------------------------------
 PostgreSQL 16.4, compiled by Visual C++ build 1940, 64-bit
(1 行记录)
```

**要显示当前数据库**，可以使用以下命令

```sql
postgres=# SELECT current_database();
 current_database
------------------
 postgres
(1 行记录)
```

**要显示当前连接的IP地址和端口**，可以执行以下命令

```sql
postgres=# SELECT
postgres-#      inet_server_addr(),
postgres-#      inet_server_port();
 inet_server_addr | inet_server_port
------------------+------------------
 ::1              |             5432
(1 行记录)
```

## 导入示例数据库

`psql` 是 PostgreSQL 的基于终端的客户端工具。它允许您输入查询，将它们发送到 PostgreSQL 执行，并显示结果

`pg_restore`是一个用于从存档恢复数据库的实用程序

要创建数据库并从存档文件加载数据，请执行以下步骤
+ 首先，使用 `psql` 连接到PostgreSQL 数据库服务器
+ 其次，创建一个名为`dvdrental`的空白数据库
+ 最后，用`pg_restore`将数据从示例数据库文件加载到`dvdrental`数据库中

来看如何创建数据库：使用 `CREATE DATABASE` 语句创建一个名为 `dvdrental` 的新数据库：

```sql
postgres=# CREATE DATABASE dvdrental;
CREATE DATABASE
```

命令 `\l` 将显示 PostgreSQL 服务器中的所有数据库

```sql
postgres=# \l
                                                                              数据库列表
   名称    |  拥有者  | 字元编码 | Locale Provider |            校对规则            |             Ctype              | ICU Locale | ICU Rules |       存取权限
-----------+----------+----------+-----------------+--------------------------------+--------------------------------+------------+-----------+-----------------------
 dvdrental | postgres | UTF8     | libc            | Chinese (Simplified)_China.936 | Chinese (Simplified)_China.936 |            |           |
 postgres  | postgres | UTF8     | libc            | Chinese (Simplified)_China.936 | Chinese (Simplified)_China.936 |            |           |
 template0 | postgres | UTF8     | libc            | Chinese (Simplified)_China.936 | Chinese (Simplified)_China.936 |            |           | =c/postgres          +
           |          |          |                 |                                |                                |            |           | postgres=CTc/postgres
 template1 | postgres | UTF8     | libc            | Chinese (Simplified)_China.936 | Chinese (Simplified)_China.936 |            |           | =c/postgres          +
           |          |          |                 |                                |                                |            |           | postgres=CTc/postgres
(4 行记录)

```

命令 `eixt` 可以退出 `psql` 。然后，使用 `pg_restore` 加载 `dvdrental` 数据库

```shell
pg_restore -U postgres -d dvdrental D:\sampledb\postgres\dvdrental.tar
```

+ `-U postgres` 表示 `pg_restore` 使用 `postgres` 用户连接 PostgreSQL 服务器

+ `-d dvdrental` 指定要加载的目标数据库

输入 `postgres` 的密码即可完成加载。为了验证加载是否成功，再次连接数据库。使用 `\c` 命令切换数据

```sql
postgres=# \c dvdrental
您现在已经连接到数据库 "dvdrental",用户 "postgres".
dvdrental=#
```

命令 `\dt` 显示所有的表

```sql
dvdrental=# \dt
                   关联列表
 架构模式 |     名称      |  类型  |  拥有者
----------+---------------+--------+----------
 public   | actor         | 数据表 | postgres
 public   | address       | 数据表 | postgres
 public   | category      | 数据表 | postgres
 public   | city          | 数据表 | postgres
 public   | country       | 数据表 | postgres
 public   | customer      | 数据表 | postgres
 public   | film          | 数据表 | postgres
 public   | film_actor    | 数据表 | postgres
 public   | film_category | 数据表 | postgres
 public   | inventory     | 数据表 | postgres
 public   | language      | 数据表 | postgres
 public   | payment       | 数据表 | postgres
 public   | rental        | 数据表 | postgres
 public   | staff         | 数据表 | postgres
 public   | store         | 数据表 | postgres
(15 行记录)
```

## 关系模型与SQL

1970 年 6 月，IBM 的研究员 Edgar Frank Codd 在 San Jose 发表了《大型共享数据库中的数据的关系模型》（A Relational Model of Data for Large Shared Data Banks）的论文，提出了 **关系模型** 的概念，奠定了关系模型的理论基础。使用 **行**、**列** 组成的 **二维表** 来组织数据和关系，表中 **行（记录）** 即可以描述数据 **实体**，也可以描述 **实体间关系**

![[Pasted image 20240810093353.png|900]]

**数据是一行行存的，列必须固定多少列**。**行(Row)**，也称为记录(Record)，元组，**代表一个实体**，即一个实体的完整属性信息。如上，一行代表了一个雇员的基本属性信息。**列(Column)**，也称为字段(Field)、属性。 字段的取值范围叫做 **域**(Domain)。例如 gender 字段的取值就是 M 或者 F 两个值

SQL（Structured Query Language）：1976年，IBM 实验室 System R 项目，通过实现数据结构和操作来证明关系模型实用性，并直接产生了结构化查询语言 SQL。1987年，SQL 被 ISO 组织标准化。它是一种对关系型数据库进行查询、更新、管理的可编程语言。


**SQL（结构化查询语言）** 使用关系模型的数据库查询语言。由 IBM 上世纪70年代开发出来。后由美国国家标准局（ANSI）开始着手制定 SQL 标准，先后有 `SQL-86` ， `SQL-89` ， `SQL-92` ， `SQL-99` 等标准。

SQL 有两个重要的标准，分别是 SQL92 和 SQL99，它们分别代表了 92 年和 99 年颁布的 SQL 标准，我们今天使用的 SQL 语言依然遵循这些标准


> [!important] 
> 
> SQL 中，字符串字面值使用单引号标注 `'....'`
> 

