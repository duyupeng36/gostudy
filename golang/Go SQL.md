# GO SQL

Go 标准包 `database/sql` 提供了 SQL 数据库的通用接口。`database/sql` 必须与 **数据库驱动** 结合使用

Go 支持的 SQL 数据库驱动在 [SQLDrivers](https://go.dev/wiki/SQLDrivers) 中完整列出。有些数据库可以存在多个驱动，我们使用列表中的第一个即可

PostgreSQL 的驱动使用 [pgx](https://github.com/jackc/pgx)

```shell
go get github.com/jackc/pgx/v5
go get github.com/jackc/pgx/v5/pgxpool@v5.6.0
```

`pgx` 是 PostgreSQL 的纯 Go 驱动程序和工具包。`pgx` 驱动程序是一个低级、高性能接口，它公开了 PostgreSQL 特定的功能，例如 `LISTEN`  `NOTIFY`和`COPY` 。它还包括标准 `database/sql` 接口的适配器

工具包组件是一组相关的包，用于实现 PostgreSQL 功能，例如解析 PostgreSQL 和 Go 之间的有线协议和类型映射。这些底层包可用于实现替代驱动程序、代理、负载平衡器、逻辑复制客户端等。

## 连接

客户端连接 PostgreSQL 服务器的连接字符串有两种形式：`key-value` 形式和 URL 形式

### Key-Value 形式的连接字符串

连接 PosgreSQL 服务端至少需要指定下面几个关键

> [!tip]
> **host**：服务所在的主机名
> 
> **port**：PostgreSQL 服务进程的端口
> 
> **user**：用户名
> 
> **password**：密码
> 
> **dbname**：连接数据库的名称
> 
> **TimeZone**：连接所在的时区
> 
> **sslmode**：是否启用 SSL

```go
// 连接 PostgreSQL 服务端的连接字符串
dsn := "host=localhost user=gorm password=gorm dbname=gorm port=9920 sslmode=disable TimeZone=Asia/Shanghai"
```

> [!seealso] 
> 更多关键字参考 [PostgreSQL-Connection](https://www.postgresql.org/docs/current/libpq-connect.html#LIBPQ-PARAMKEYWORDS)

### URL 形式

URL 形式的连接字符串的基本结果如下

```
postgresql://[ user:password@ ][ host:port[, host:port]...][ /dbname ][ ?paramspec ]
```

下面是一些 URL 连接字符串的形式

```
postgresql://
postgresql://localhost
postgresql://localhost:5433
postgresql://localhost/mydb
postgresql://user@localhost
postgresql://user:secret@localhost
postgresql://other@localhost/otherdb?connect_timeout=10&application_name=myapp
postgresql://host1:123,host2:456/somedb?target_session_attrs=any&application_name=myapp
```

后续我们需要使用 GORM，因此我们这里介绍如何使用 `database/sql` 接口建立连接

```go
package main

import (
	"database/sql"
	"fmt"
	_ "github.com/jackc/pgx/v5/stdlib" // 标准接口
	"os"
)

func main() {
	dsn := "host = localhost user=postgres password=dyp1996 dbname=atguigudb port=5432 sslmode=disable TimeZone=Asia/Shanghai"
	db, err := sql.Open("pgx", dsn)
	if err != nil {
		fmt.Fprintf(os.Stderr, "Unable to connect to database: %v\n", err)
		os.Exit(1)
	}
	defer db.Close()

	if err := db.Ping(); err != nil {
		fmt.Fprintf(os.Stderr, "Unable to connect to database: %v\n", err)
		os.Exit(1)
	}

	var employee_id int
	var name string

	err = db.QueryRow("SELECT employee_id, last_name FROM employees WHERE employee_id = $1", 200).Scan(&employee_id, &name)
	if err != nil {
		fmt.Fprintf(os.Stderr, "Unable to retrieve employee: %v\n", err)
		os.Exit(1)
	}
	fmt.Printf("Employee ID: %d\n", employee_id)
	fmt.Printf("Name: %s\n", name)

}
```

通常我们会将建立连接的代码在 `init` 函数中

```go
type DataSourceName map[string]interface{}

func (dsn DataSourceName) String() string {

	var buf bytes.Buffer

	for k, v := range dsn {
		buf.WriteString(k)
		buf.WriteString("=")
		buf.WriteString(fmt.Sprintf("%v ", v))
	}
	return strings.TrimSpace(buf.String())
}

var db *sql.DB

var dsn DataSourceName = DataSourceName{
	"host":     "localhost",
	"port":     5432,
	"user":     "postgres",
	"password": "dyp1996",
	"database": "atguigudb",
	"sslmode":  "disable",
	"TimeZone": "Asia/Shanghai",
}

func init() {
	var err error

	// 只是产生了一个 DB 对象，并没有真正的连接
	db, err = sql.Open("pgx", dsn.String())

	if err != nil {
		log.Fatalf("Unable to connect to database: %v\n", err)
	}

	// 尝试与 PostgreSQL 建立连接
	if err := db.Ping(); err != nil {
		log.Fatalf("Unable to ping database: %v\n", err)
	}
}
```


当然，可以使用 `database/sql` 标准接口，而是使用 `pgx` 提供的接口

```go
package main

import (
	"context"
	"fmt"
	"os"

	"github.com/jackc/pgx/v5"
)

func main() {
	
	dsn := "host = localhost user=postgres password=dyp1996 dbname=atguigudb port=5432 sslmode=disable TimeZone=Asia/Shanghai"
	conn, err := pgx.Connect(context.Background(), dsn)
	if err != nil {
		fmt.Fprintf(os.Stderr, "Unable to connect to database: %v\n", err)
		os.Exit(1)
	}
	defer conn.Close(context.Background())

	var employee_id int
	var name string
	err = conn.QueryRow(context.Background(), "SELECT employee_id, last_name FROM employees WHERE employee_id = $1", 200).Scan(&employee_id, &name)
	if err != nil {
		fmt.Fprintf(os.Stderr, "QueryRow failed: %v\n", err)
		os.Exit(1)
	}

	fmt.Printf("Employee ID: %d\n", employee_id)
	fmt.Printf("Name: %s\n", name)
}
```

> [!tip]
> 对于不需要 `database/sql` 的应用场景，建议使用 `pgx` 提供的接口，因为它的性能更高，并且可以提供 PostgreSQL 高级功能


## CURD

如下信息是我们测试的表结构

```sql
atguigudb=# \d employees;

-- Output
                      数据表 "public.employees"
      栏位      |         类型          | 校对规则 |  可空的  | 预设
----------------+-----------------------+----------+----------+------
 employee_id    | integer               |          | not null |
 first_name     | character varying(20) |          |          |
 last_name      | character varying(25) |          | not null |
 email          | character varying(25) |          | not null |
 phone_number   | character varying(20) |          |          |
 hire_date      | date                  |          | not null |
 job_id         | character varying(10) |          | not null |
 salary         | double precision      |          |          |
 commission_pct | double precision      |          |          |
 manager_id     | integer               |          |          |
 department_id  | integer               |          |          |
索引：
    "employees_pkey" PRIMARY KEY, btree (employee_id)
    "emp_dept_fk" btree (department_id)
    "emp_email_uk" UNIQUE, btree (email)
    "emp_emp_id_pk" UNIQUE, btree (employee_id)
    "emp_job_fk" btree (job_id)
    "emp_manager_fk" btree (manager_id)
外部键(FK)限制：
    "emp_dept_fk" FOREIGN KEY (department_id) REFERENCES departments(department_id)
    "emp_job_fk" FOREIGN KEY (job_id) REFERENCES jobs(job_id)
    "emp_manager_fk" FOREIGN KEY (manager_id) REFERENCES employees(employee_id)
由引用：
    TABLE "departments" CONSTRAINT "dept_mgr_fk" FOREIGN KEY (manager_id) REFERENCES employees(employee_id)
    TABLE "employees" CONSTRAINT "emp_manager_fk" FOREIGN KEY (manager_id) REFERENCES employees(employee_id)
    TABLE "job_history" CONSTRAINT "jhist_emp_fk" FOREIGN KEY (employee_id) REFERENCES employees(employee_id)
```
### 查询

假设，我们只查询下面结构体中的字段

```go
type employee struct {
	employee_id  int
	last_name    string
	phone_number string
}
```

#### 单行查询

**`db.QueryRow()` 执行一次查询**，并期望返回 **最多一行结果（即 `Row` 类型的对象）**。`QueryRow` 总是返回非 `nil` 的值，**直到返回值的 `Scan` 方法被调用时，才会返回被延迟的错误**。（如：未找到结果）

```go
func (db *DB) QueryRow(query string, args ...interface{}) *Row
```

**`row.Scan`** 方法读取查询的结构的值

```go
func (r *Row) Scan(dest ...any) error
```

> [!NOTE] 注意，在 `Scan` 时，**`dest` 实参按照 `query` SQL 语句查询的结果顺序放置**，否则可能会出现 `Scan` 失败
> 

##### 示例代码
```go
func queryOne() {
	var emp employee

	query := "select employee_id, last_name, phone_number from employees where employee_id=$1"
	err := db.QueryRow(query, 100).Scan(&emp.employee_id, &emp.last_name, &emp.phone_number)
	if err != nil {
		log.Println(err)
		return
	}
	fmt.Printf("employee_id: %d last_name: %s phone_number: %s\n", emp.employee_id, emp.last_name, emp.phone_number)
}
```

#### 多行查询

**`db.Query()` 执行一次查询，返回多行结果（即 `Rows` 类型对象）**，一般用于执行 `select` 命令。参数 `args` 表示 `query` 中的占位参数

```go
func (db *DB) Query(query string, args ...interface{}) (*Rows, error)
```

`Rows`对象是查询的结果集。其 **游标** 从结果集的第一行开始。使用 `Rows.Next `可以从一行前进到另一行

```go
func (rs *Rows) Next() bool
```

使用 `rows.Scan` 将 **当前行** 中的列复制到 `dest` 指向的值中。 `dest` 中的值数必须与 `Rows` 中的列数相同

```go
func (rs *Rows) Scan(dest ...any) error
```

##### 示例代码

```go
// queryRows 查询 雇员 100 <= id <= 110 的 last_name phone_number
func queryRows() {
	var emp employee
	query := "select employee_id, last_name, phone_number from employees where employee_id>=$1 and employee_id <= $2"
	rows, err := db.Query(query, 100, 110)
	if err != nil {
		log.Println(err)
		return
	}
	defer rows.Close() // 延迟关闭游标
	// 循环输出
	for rows.Next() {
		rows.Scan(&emp.employee_id, &emp.last_name, &emp.phone_number)
		fmt.Printf("employee_id: %d last_name: %s phone_number: %s\n", emp.employee_id, emp.last_name, emp.phone_number)
	}
}
```

### 插入

插入、更新和删除操作都使用`db.Exec`方法

```go
func (db *DB) Exec(query string, args ...interface{}) (Result, error)
```

`db.Exec` 执行一次命令（包括查询、删除、更新、插入等），返回的 `Result` 是对已执行的 `SQL` 命令的总结。参数 `args` 表示 `query` 中的占位参数

#### `Result` 接口

```go
type Result interface {
    // LastInsertId 返回一个数据库生成的回应命令的整数。
    // 当插入新行时，一般来自一个"自增"列。
    // 不是所有的数据库都支持该功能，该状态的语法也各有不同。
    LastInsertId() (int64, error)

    // RowsAffected 返回被 update、insert 或 delete 命令影响的行数。
    // 不是所有的数据库都支持该功能。
    RowsAffected() (int64, error)
}
```


#### 示例代码

```go
// 插入数据
func insertRowDemo() {
	query := "insert into user(name, age) values ($1,$2)"
	ret, err := db.Exec(query, "王五", 38)
	if err != nil {
		fmt.Printf("insert failed, err:%v\n", err)
		return
	}
	theID, err := ret.LastInsertId() // 新插入数据的 id
	if err != nil {
		fmt.Printf("get lastinsert ID failed, err:%v\n", err)
		return
	}
	fmt.Printf("insert success, the id is %d.\n", theID)
}
```

### 更新

具体更新数据示例代码如下

```go
// 更新数据
func updateRowDemo() {
	sqlStr := "update user set age=? where id = $1"
	ret, err := db.Exec(sqlStr, 39, 3)
	if err != nil {
		fmt.Printf("update failed, err:%v\n", err)
		return
	}
	n, err := ret.RowsAffected() // 操作影响的行数
	if err != nil {
		fmt.Printf("get RowsAffected failed, err:%v\n", err)
		return
	}
	fmt.Printf("update success, affected rows:%d\n", n)
}
```

### 删除

具体删除数据的示例代码如下：

```go
// 删除数据
func deleteRowDemo() {
	sqlStr := "delete from user where id =$1"
	ret, err := db.Exec(sqlStr, 3)
	if err != nil {
		fmt.Printf("delete failed, err:%v\n", err)
		return
	}
	n, err := ret.RowsAffected() // 操作影响的行数
	if err != nil {
		fmt.Printf("get RowsAffected failed, err:%v\n", err)
		return
	}
	fmt.Printf("delete success, affected rows:%d\n", n)
}
```

## 预处理

什么是预处理？

普通 `SQL` 语句执行过程：
1. 客户端对 `SQL` 语句进行占位符替换得到完整的 `SQL` 语句
2. 客户端发送完整 `SQL` 语句到 `MySQL` 服务端
3. `MySQL` 服务端执行完整的 `SQL` 语句并将结果返回给客户端


预处理执行过程：
1. 把 `SQL` 语句分成两部分，命令部分与数据部分
2. 先把命令部分发送给 `MySQL` 服务端，`MySQL` 服务端进行 `SQL` 预处理
3. 然后把数据部分发送给 `MySQL` 服务端，`MySQL` 服务端对 `SQL` 语句进行占位符替换
4. `MySQL` 服务端执行完整的 `SQL` 语句并将结果返回给客户端


为什么要预处理？
1. 优化 `MySQL` 服务器重复执行 `SQL` 的方法，可以提升服务器性能，提前让服务器编译，一次编译多次执行，节省后续编译的成本
2. 避免 `SQL` 注入问题

#### Go 实现预处理

`db.Prepare`方法来实现预处理操作

```go
func (db *DB) Prepare(query string) (*Stmt, error)
```

`Prepare` 方法会先将sql语句发送给 `MySQL` 服务端，返回一个 **准备好的状态对象** 用于之后的查询和命令。返回值可以同时执行多个查询和命令

#### `Stmt` 类型

`Stmt` 是一个 **准备好的声明**。 `Stmt` 对于多个 `goroutine` 并发使用是安全的

```go
type Stmt struct {
	// contains filtered or unexported fields
}

func (s *Stmt) Close() error
func (s *Stmt) Exec(args ...any) (Result, error)
func (s *Stmt) ExecContext(ctx context.Context, args ...any) (Result, error)
func (s *Stmt) Query(args ...any) (*Rows, error)
func (s *Stmt) QueryContext(ctx context.Context, args ...any) (*Rows, error)
func (s *Stmt) QueryRow(args ...any) *Row
func (s *Stmt) QueryRowContext(ctx context.Context, args ...any) *Row
```


查询操作的预处理示例代码如下

```go
// 预处理查询示例
func prepareQueryDemo() {
	query := "SELECT employees.employee_id, employees.last_name FROM employees WHERE employee_id > $1 AND employee_id < $2"
	stmt, err := db.Prepare(query)
	if err != nil {
		fmt.Printf("prepare failed, err:%v\n", err)
		return
	}
	defer func(stmt *sql.Stmt) {
		err := stmt.Close()
		if err != nil {
			log.Fatalf("stmt.Close failed, err:%v\n", err)
		}
	}(stmt)

	rows, err := stmt.Query(100, 105)

	if err != nil {
		fmt.Printf("query failed, err:%v\n", err)
		return
	}

	defer func(rows *sql.Rows) {
		err := rows.Close()
		if err != nil {
			log.Fatalf("rows.Close failed, err:%v\n", err)
		}
	}(rows)

	// 循环读取结果集中的数据
	for rows.Next() {
		var u struct {
			employeeId int
			lastName   string
		}

		err := rows.Scan(&u.employeeId, &u.lastName)
		if err != nil {
			log.Fatalf("rows.Scan failed, err:%v\n", err)
		}

		fmt.Printf("id:%d name:%s\n", u.employeeId, u.lastName)
	}
}
```

## SQL注入问题

> [!WARNING] **我们任何时候都不应该自己拼接 SQL 语句！**
> - 自行拼接 SQL 语句可能会被客户端构造一个注释语句创建出始终为真值的 `where` 条件

这里我们演示一个自行拼接 `SQL` 语句的示例，编写一个根据 `name` 字段查询 `user` 表的函数如下：
```go
// sql注入示例
func sqlInjectDemo(name string) {
	sqlStr := fmt.Sprintf("select id, name, age from user where name='%s'", name)
	fmt.Printf("SQL:%s\n", sqlStr)
	var u user
	err := db.QueryRow(sqlStr).Scan(&u.id, &u.name, &u.age)
	if err != nil {
		fmt.Printf("exec failed, err:%v\n", err)
		return
	}
	fmt.Printf("user:%#v\n", u)
}

```
此时以下输入字符串都可以引发SQL注入问题
```go
sqlInjectDemo("xxx' or 1=1#")  // select id, name, age from user where name='xxx' or 1=1#  永真的 where 条件
sqlInjectDemo("xxx' union select * from user #") // select id, name, age from user where name='xxx' union select * from user #
sqlInjectDemo("xxx' and (select count(*) from user) <10 #") // select id, name, age from user where name='xxx' and (select count(*) from user) <10 #
```

**补充：** 不同的数据库中，SQL语句使用的占位符语法不尽相同。

| 数据库        | 占位符语法       |
| ---------- | ----------- |
| MySQL      | `?`         |
| PostgreSQL | `$1`, `$2`等 |
| SQLite     | `?` 和`$1`   |
| Oracle     | `:name`     |

## 事务

**事务**：一个 **最小的不可再分的工作单元**；通常 **一个事务对应一个完整的业务**(例如银行账户转账业务，该业务就是一个最小的工作单元)，同时这个完整的业务需要执行多次的 **DML**(`insert`、`update`、`delete`)语句共同联合完成。`A` 转账给 `B`，这里面就需要执行两次 `update` 操作

### ACID

通常事务必须满足 $4$ 个条件（ACID）：**原子性**（`Atomicity`，或称不可分割性）、**一致性**（`Consistency`）、**隔离性**（`Isolation`，又称独立性）、**持久性**（`Durability`）。

| 条件      | 解释                                                                                                                                                                                 |
| ------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **原子性** | **一个事务（`transaction`）中的所有操作，_要么全部完成，要么全部不完成_，不会结束在中间某个环节**。事务在执行过程中发生错误，会被 **回滚**（`Rollback`）到事务开始前的状态，就像这个事务从来没有执行过一样                                                             |
| **一致性** | **在事务开始之前和事务结束以后，数据库的 _完整性没有被破坏_**。这表示写入的资料必须完全符合所有的预设规则，这包含资料的精确度、串联性以及后续数据库可以自发性地完成预定的工作                                                                                         |
| **隔离性** | **数据库允许多个并发事务同时对其数据进行读写和修改的能力**，隔离性可以防止多个事务并发执行时由于交叉执行而导致数据的不一致。事务隔离分为不同级别，包括 **读未提交**（Read uncommitted）、**读提交**（read committed）、**可重复读**（repeatable read）和 **串行化**（Serializable）。 |
| **持久性** | **事务处理结束后，对数据的修改就是永久的**，即便系统故障也不会丢失。                                                                                                                                               |

### Go实现事务

Go 语言中使用以下三个方法实现 `MySQL` 中的事务操作

**开始事务** `db.Begin()` 方法返回一个 `Tx` 事务对象

```go
func (db *DB) Begin() (*Tx, error)
```

可以在 `Tx` 对象上执行 `DML` 操作，然后 **提交事务**，使用事务对象 `tx.Commit()` 方法，返回 `err` 表示事务错误

```go
func (tx *Tx) Commit() error
```

如果事务出错，则需要 **回滚**，使用事务对象的 `tx.Rollback()` 方法进行回滚

```go
func (tx *Tx) Rollback() error
```

### 示例代码

下面的代码演示了一个简单的事务操作，该**事物操作能够确保两次更新操作要么同时成功要么同时失败，不会存在中间状态**

```go
// 事务操作示例
func transactionDemo() {
	tx, err := db.Begin() // 开启事务
	if err != nil {
		// 开启事务失败，然而返回了事务对象，就需要对事务进行回滚
		if tx != nil {
			tx.Rollback() // 回滚
		}
		fmt.Printf("begin trans failed, err:%v\n", err)
		return
	}
	sqlStr1 := "Update user set age=30 where id=$1"
	ret1, err := tx.Exec(sqlStr1, 2)
	if err != nil {
		tx.Rollback() // 回滚
		fmt.Printf("exec sql1 failed, err:%v\n", err)
		return
	}
	affRow1, err := ret1.RowsAffected()
	if err != nil {
		tx.Rollback() // 回滚
		fmt.Printf("exec ret1.RowsAffected() failed, err:%v\n", err)
		return
	}

	sqlStr2 := "Update user set age=40 where id=$1"
	ret2, err := tx.Exec(sqlStr2, 3)
	if err != nil {
		tx.Rollback() // 回滚
		fmt.Printf("exec sql2 failed, err:%v\n", err)
		return
	}
	affRow2, err := ret2.RowsAffected()
	if err != nil {
		tx.Rollback() // 回滚
		fmt.Printf("exec ret1.RowsAffected() failed, err:%v\n", err)
		return
	}

	fmt.Println(affRow1, affRow2)
	if affRow1 == 1 && affRow2 == 1 {
		fmt.Println("事务提交啦...")
		tx.Commit() // 提交事务
	} else {
		tx.Rollback()
		fmt.Println("事务回滚啦...")
	}
	fmt.Println("exec trans success!")
}
```
