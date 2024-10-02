# GORM

**对象关系映射**（Object Relational Mapping，**ORM**）是一种为了解决 **面向对象** 与 **关系数据库**（如 PostgreSQL 数据库）存在的互不匹配的现象的技术。简单的说，ORM 是通过使用描述 **对象和数据库之间映射** 的元数据，将程序中的对象自动持久化到关系数据库中

面向对象是从 **软件工程基本原则**（如耦合、聚合、封装）的基础上发展起来的，而关系数据库则是从 **数学理论** 发展而来的，两套理论存在显著的区别。为了解决这个不匹配的现象，**对象关系映射** 技术应运而生

在 Go 中，关系模型和 Go 对象之间的映射如下表

|   关系模型   |              Go 对象               | 说明                          |
| :------: | :------------------------------: | --------------------------- |
| `table`  |         `struct`, `map`          | 表映射为结构体或者 `map`             |
|  `row`   | `struct` 类型的一个变量<br>`map` 类型的变量  | 表中的一行映射为一个变量                |
| `column` | `struct` 类型的一个成员<br>`map` 中一个关键字 | 每一列映射结构体的成员或者 `map` 的 `key` |

可以认为 ORM 是一种高级抽象，对象的操作最终还是会转换成对应关系数据库操作的 SQL 语句，数据库操作的结构会被封装成对象

GORM 是一个非常受欢迎的 ORM 框架，它提供了高效且简单的接口。首先我们需要安装 GORM 和 数据库驱动

GORM 官方支持的数据库类型有：MySQL, **PostgreSQL**, SQLite, SQL Server 和 TiDB

```shell
go get -u gorm.io/gorm
go get -u gorm.io/driver/postgres
```

GORM 的 `postgres` 驱动使用 [pgx](https://github.com/jackc/pgx) 作为 PostgresSQL 的 `database/sql` 驱动。如果没有正确安装，请使用下面的命令进行安装

```shell
go get -u github.com/jackc/pgx/v5
```

## 连接数据库

### 简单连接

```go
package main

import (
	"bytes"
	"fmt"
	"gorm.io/driver/postgres"
	"gorm.io/gorm"
	"log"
	"strings"
)

var db *gorm.DB

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

	db, err = gorm.Open(postgres.Open(dsn.String()), &gorm.Config{})

	if err != nil {
		log.Fatalf("Failed to connect to database: %v", err)
	}
}
```

`pgx` 默认启用 prepared statement 缓存，如果需要关闭，请如下配置

```go
db, err = gorm.Open(postgres.New(postgres.Config{
	DSN:                  dsn.String(), // 数据源名
	PreferSimpleProtocol: true,         // 禁用 prepared statement 缓存
}), &gorm.Config{})
```

### 使用现有连接

GORM 允许通过一个现有的数据库连接来初始化 `*gorm.DB`

```go
import (
  "database/sql"
  "gorm.io/driver/mysql"
  "gorm.io/gorm"
)

sqlDB, err := sql.Open("mysql", "mydb_dsn")  // database/sql 打开的连接

gormDB, err := gorm.Open(mysql.New(mysql.Config{
	Conn: sqlDB,
}), &gorm.Config{})
```

### 日志

GORM 默认只显示错误和慢SQL。在连接时可以配置显示日志的等级

```go
db, err = gorm.Open(postgres.Open(dsn.String()), &gorm.Config{
		Logger: logger.Default.LogMode(logger.Info), // 指定显示 >= INFO 等级的日子
	})
```

如果只需要某个会话显示日志，可以使用 `Session()` 方法进行配置

```go
db.Session(&gorm.Session{  
    Logger: logger.Default.LogMode(logger.Info),  
})
```

## 模型定义

GORM 通过将 **Go 结构体**（Go structs） 映射到 **数据库表** 来简化数据库交互

**模型是使用 _普通结构体定义_ 的**。 这些结构体可以包含具有 **基本Go类型**、**指针** 或这些类型的 **别名**，甚至是 **自定义类型**（只需要实现 `database/sql` 包中的 [Scanner](https://pkg.go.dev/database/sql/?tab=doc#Scanner) 和 [Valuer](https://pkg.go.dev/database/sql/driver#Valuer) 接口）

```go
type User struct {
  ID           uint           // 约定：ID 字段作为主键
  Name         string         // 常规字符串字段
  Email        *string        // 具体类型的指针, 允许 NULL 
  Age          uint8          // 常规整数字段
  Birthday     *time.Time     // 指向 time.Time 的指针，可以为 NULL
  MemberNumber sql.NullString // 使用 sql.NullString 处理可空字符串
  ActivatedAt  sql.NullTime   // 使用 sql.NullTime 来处理可归零的时间字段
  CreatedAt    time.Time      // 创建时间（由 GORM 自动管理）
  UpdatedAt    time.Time      // 最后一次更新时间（由 GORM 自动管理）
}
```

> [!tip]
> 
> 小写成员无法被在其他包中访问。也就是说，**结构体中小写的字段无法生成列的**
> 

GORM 崇尚 **约定大于配置**，遵循这些约定可以大大减少需要编写的配置或代码量。 但是，GORM 也具有灵活性，**允许根据自己的需求自定义这些约定**

> [!tip] GORM 约定
> 
> **主键** ：结构体中 **名为 ID 的字段作为默认主键字段** 
> 
> **表名**：**结构体名** 称转换为 **snake_case** 并为表名加上 **_复数_** 形式
> 
> **列名**：**结构体字段名(大驼峰规范)** 转换为 **snake_case**
> 
> **时间**：名为 **CreatedAt** 和 **UpdatedAt** 的字段，**由 GORM 自动跟踪**

GORM 提供了一个预定义的结构体，名为 `gorm.Model`，其中包含常用字段

```go
// gorm.Model 的定义
type Model struct {
  ID        uint           `gorm:"primaryKey"`// 主键 
  CreatedAt time.Time                         // 记录创建时间
  UpdatedAt time.Time                         // 记录更新时间
  DeletedAt gorm.DeletedAt `gorm:"index"`     // 用于软删除
}
```

**直接将这结构 _嵌入_ 到我们定义的结构体中**，以便自动包含这些字段

```go
type User struct {
	gorm.Model    // 嵌入到该模型中
	Name string
}
// 等效于
type User struct {
	ID        uint           `gorm:"primaryKey"`
	CreatedAt time.Time
	UpdatedAt time.Time
	DeletedAt gorm.DeletedAt `gorm:"index"`
	Name string
}
```

还可以使用 `embedded` 标签

```go
type Author struct {
	Name  string
	Email string
}

type Blog struct {
	ID      int
	Author  Author `gorm:"embedded"`
	Upvotes int32
}
// 等效于
type Blog struct {
	ID    int64
	Name  string
	Email string
	Upvotes  int32
}
```

可以使用 **标签 `embeddedPrefix` 来为 数据库中的字段名添加前缀**，

```go
type Blog struct {
	ID      int
	Author  Author `gorm:"embedded;embeddedPrefix:author_"`
	Upvotes int32
}
// 等效于
type Blog struct {
	ID          int64
	AuthorName string
	AuthorEmail string
	Upvotes     int32
}
```

### 字段标签

声明 `model` 时，**`tag` 是可选的**，GORM 支持以下 `tag`

| 标签名          | 说明                                                                                           |
| :----------- | :------------------------------------------------------------------------------------------- |
| `column`     | 指定列名                                                                                         |
| `type`       | 列数据类型，通常指定 **数据库的数据类型**                                                                      |
| `size`       | 定义列数据类型的大小或长度，例如：`size:256`。`string` 类型的 `size` 代表字符数量。`int` 类型的 `size` 代表占用的位数              |
| `precision`  | 指定列的精度                                                                                       |
| `scale`      | 指定列大小                                                                                        |
| `<-`         | 设置字段 **写入的权限**<br>"<-:create" ：只创建<br>"<-:update"： 只更新<br>"<-:false"：无写入权限<br>"<-"：具有创建和更新权限 |
| `->`         | 设置字段 **读的权限**<br>"->:false" 无读权限                                                             |
| `-`          | 忽略该字段，**-  无读写权限**                                                                           |
| `comment`    | 迁移时 **为字段添加注释**                                                                              |
| `serializer` | 指定将数据序列化或反序列化到数据库中的序列化器, 例如: `serializer:json/gob/unixtime`                                  |

> [!tip]
>  **tag 名大小写不敏感**，但 **建议使用 camelCase 风格**

此外还可以指定列约束条件

| 标签名                      | 说明                                    |
| :----------------------- | :------------------------------------ |
| `primaryKey`             | 指定列为主键，**主键约束**                       |
| `unique`                 | 指定列为唯一，**唯一约束**                       |
| `default`                | 指定列的默认值，**默认值约束**                     |
| `not null`               | 指定列为` NOT NULL`，**非空约束**              |
| `autoIncrement`          | 指定列为自动增长，**自增约束**                     |
| `autoIncrementIncrement` | 自动步长，控制连续记录之间的间隔                      |
| `index`                  | 根据参数创建索引，多个字段使用相同的名称则创建复合索引           |
| `uniqueIndex`            | 与 `index` 相同，但创建的是唯一索引，**唯一约束**       |
| `check`                  | 创建检查约束，例如 `check:age > 13`，查看 约束 获取详情 |

### 查询时使用的模型

```go
package models

import "time"

// Employee 雇员表
type Employee struct {
	EmployeeID    int       `gorm:"primaryKey;type:BIGSERIAL;not null"`
	FirstName     string    `gorm:"size:20"`
	LastName      string    `gorm:"size:25;not null"`
	Email         string    `gorm:"size:25;not null"`
	PhoneNumber   string    `gorm:"size:20"`
	HireDate      time.Time `gorm:"type:date;not null"`
	JobID         string    `gorm:"size:10;not null"`
	Salary        float64   `gorm:"type:float8"`
	CommissionPct *float64  `gorm:"type:float8"`
	ManagerID     *int      `gorm:"size:32;not null"`
	DepartmentID  *int      `gorm:"size:32;not null"`
}

// Department 部门表
type Department struct {
	DepartmentID   int    `gorm:"primaryKey;size:32"`
	DepartmentName string `gorm:"size:30;not null"`
	ManagerID      int    `gorm:"size:32"`
	LocationID     int    `gorm:"size:32"`
}

// Job 工作表
type Job struct {
	JobID     string `gorm:"size:10;primaryKey"`
	JobTitle  string `gorm:"size:35;not null"`
	MinSalary int    `gorm:"size:32"`
	MaxSalary int    `gorm:"size:32"`
}

// JobHistory 历史工作表
type JobHistory struct {
	EmployeeID   int       `gorm:"primaryKey;type:int"`
	StartDate    time.Time `gorm:"primaryKey;type:date"`
	EndDate      time.Time `gorm:"type:date;not null"`
	JobID        string    `gorm:"size:10;not null"`
	DepartmentID int       `gorm:"size:32"`
}

func (jh *JobHistory) TableName() string {
	return "job_history"
}

// Location 位置表
type Location struct {
	LocationID    int    `gorm:"primaryKey;size:32"`
	StreetAddress string `gorm:"size:40"`
	PostalCode    string `gorm:"size:12"`
	City          string `gorm:"size:30;not null"`
	StateProvince string `gorm:"size:25"`
	CountryID     string `gorm:"type:char(2)"`
}

// Country 国家表
type Country struct {
	CountryID   string `gorm:"type:char(2);primaryKey"`
	CountryName string `gorm:"size:40"`
	RegionID    int    `gorm:"size:32"`
}

// Region 区域表
type Region struct {
	RegionID   int    `gorm:"primaryKey;size:32"`
	RegionName string `gorm:"size:25"`
}
```

## 数据库迁移

`AutoMigrate` 用于自动迁移您的 `schema`，保持您的 `schema` 是最新的

> [!WARNING] 注意
> `AutoMigrate` 会 **创建表、缺失的外键、约束、列和索引**
> 
> 如果 大小、精度、是否为空可以更改，则 `AutoMigrate` 会改变列的类型
> 
> 出于保护您数据的目的，**它 不会 删除未使用的列**

```go
db.AutoMigrate(&User{}) 

db.AutoMigrate(&User{}, &Product{}, &Order{})

// 创建表时添加后缀
db.Set("gorm:table_options", "ENGINE=InnoDB").AutoMigrate(&User{})

```

> [!WARNING] 注意
> `AutoMigrate` 会 **自动创建数据库外键约束**，您可以在初始化时禁用此功能，例如

```go
db, err := gorm.Open(sqlite.Open("gorm.db"), &gorm.Config{
	DisableForeignKeyConstraintWhenMigrating: true,  // gorm 配置：禁用外键约束
})
```

> [!tip]
> 
> `AutoMigrate` 迁移逻辑： **只新增**，不删除也不修改
> 
> 

GORM 提供了 `Migrator` 接口，该 **接口为每个数据库提供了 _统一的 API 接口_，可用来 _为数据库构建独立迁移_**，例如：

```go
type Migrator interface {
  // AutoMigrate 自动迁移
  AutoMigrate(dst ...interface{}) error

  // Database 数据库相关
  CurrentDatabase() string  // 获取当前数据库名
  FullDataTypeOf(*schema.Field) clause.Expr

  // Tables 表相关方法
  CreateTable(dst ...interface{}) error
  DropTable(dst ...interface{}) error
  HasTable(dst interface{}) bool  //  检查 dst 对应的表是否存在
  RenameTable(oldName, newName interface{}) error
  GetTables() (tableList []string, err error)

  // Columns 列相关方法
  AddColumn(dst interface{}, field string) error
  DropColumn(dst interface{}, field string) error
  AlterColumn(dst interface{}, field string) error
  MigrateColumn(dst interface{}, field *schema.Field, columnType ColumnType) error
  HasColumn(dst interface{}, field string) bool
  RenameColumn(dst interface{}, oldName, field string) error
  ColumnTypes(dst interface{}) ([]ColumnType, error)

  // Views 视图相关方法
  CreateView(name string, option ViewOption) error
  DropView(name string) error

  // Constraints 约束相关方法
  CreateConstraint(dst interface{}, name string) error
  DropConstraint(dst interface{}, name string) error
  HasConstraint(dst interface{}, name string) bool

  // Indexes 索引相关方法
  CreateIndex(dst interface{}, name string) error
  DropIndex(dst interface{}, name string) error
  HasIndex(dst interface{}, name string) bool
  RenameIndex(dst interface{}, oldName, newName string) error
}
```

`gorm.DB` 的 `Migrator()` 方法获得一个符合 `Migrator` 接口的对象，通过这个对象就能完成数据库迁移

## 单表插入

GORM 插入数据使用 `db.Create()` 方法。例如，下面的模型

```go
type User struct {
	gorm.Model
	Name     string         `gorm:"size:255;not null;unique" json:"name"`
	Email    sql.NullString `gorm:"size:255" json:"email"`
	Age      int            `gorm:"type:smallint" json:"age"`
	Birthday sql.NullTime   `gorm:"type:TIMESTAMPTZ" json:"birthday"`
}
```

如果需要插入数据，只需要调用 `Create()` 方法，并传入指向数据的指针即可

```go
// Create 会返回一个上下文的 DB 对象
tx := db.Create(&User{
	Name: "杜宇鹏",
	Email: sql.NullString{
		String: "duyupeng36@foxmail.com",
		Valid:  true,
	},
	Age: 28,
})
```

生成的 SQL 语句为

```sql
INSERT INTO "users" ("created_at","updated_at","deleted_at","name","email","age","birthday") VALUES ('2024-08-14 15:00:02.498','2024-08-14 15:00:02.498',NULL,'杜宇鹏','duyupeng36@foxmail.com',28,NULL) RETURNING "id"
```

> [!tip]
> 
>GORM 会将插入数据的主键字段的值返回给实例的主键字段
> 

### 插入 Hooks

如果想要 `"birthday"` 字段根据 `"Age"` 字段自动生成，可以给模型定义钩子函数。GROM 允许用户通过实现这些接口 `BeforeSave`, `BeforeCreate`, `AfterSave`, `AfterCreate` 来自定义 **钩子**。 这些钩子方法会在 **创建一条记录时被调用**

钩子方法的函数签名应该是 `func(*gorm.DB) error`

> [!tip] 创建时可用的钩子
> 
> **BeforeSave** ：在保存数据之前调用
> 
> **BeforeCreate** ：创建数据之前调用
> 
> 
> **AfterCreate**：数据创建完成后调用
> 
> **AfterSave**：数据保存之后调用
> 

> [!attention] **注意** 
> 
> 在 GORM 中 **保存** **删除** 操作会 **默认运行在事务** 上
> 
> 因此在事务完成之前该事务中所作的更改是不可见的，**如果钩子返回了任何错误，则修改将被回滚**

我们可以为 `User` 结构体定义 `BeforeCreate` 方法，在其中根据 `Age` 生成 `Birthday` 字段

```go
// BeforeCreate 在 Create 调用之前
func (u *User) BeforeCreate(tx *gorm.DB) error {
	// u.Birthday 是 sql.NullTime 类型，其中 Valid 字段指定了当前值是否可用
	// 如果当前值不可用，则需要最佳当前值
	if !u.Birthday.Valid {
		u.Birthday = sql.NullTime{
			Time:  time.Date(time.Now().Year()-u.Age, time.Now().Month(), time.Now().Day(), 0, 0, 0, 0, time.UTC),
			Valid: true,
		}
		fmt.Printf("BeforeCreate: %v\n", u.Birthday.Time)
	}
	return nil
}
```

### 批量插入

要高效地插入大量记录，请将切片传递给 `Create` 方法。 GORM 将生成一条 SQL 来插入所有数据，以 **返回所有主键值**，并触发  `Hook`  方法。 当这些记录可以被分割成多个批次时，GORM会开启一个事务来处理它们

```go
// Create 批量插入
users := []*User{
	{
		Name: "aaa",
		Email: sql.NullString{
			String: "aaa@gmail.com",
			Valid:  true,
		},
		Age: 11,
	},
	{
		Name: "bbb",
		Email: sql.NullString{
			String: "bbb@gmail.com",
			Valid:  true,
		},
		Age: 13,
	},
	{
		Name: "ccc",
		Email: sql.NullString{
			String: "ccc@gmail.com",
			Valid:  true,
		},
		Age: 17,
	},
}
tx := db.Create(users)
```

生成的 SQL 语句为

```sql
INSERT INTO "users" ("created_at","updated_at","deleted_at","name","email","age","birthday") VALUES ('2024-08-14 15:23:44.362','2024-08-14 15:23:44.362',NULL,'aaa','aaa@gmail.com',11,'2013-08-14 00:00:00'),('2024-08-14 15:23:44.362','2024-08-14 15:23:44.362',NULL,'bbb','bbb@gmail.com',13,'2011-08-14 00:00:00'),('2024-08-14 15:23:44.362','2024-08-14 15:23:44.362',NULL,'ccc','ccc@gmail.com',17,'2007-08-14 00:00:00') RETURNING "id"
```

可以通过 `db.CreateInBatches` 方法来指定批量插入的 **批次大小**

```go
var users = []User{{Name: "jinzhu_1"}, ...., {Name: "jinzhu_10000"}}

// batch size 100
db.CreateInBatches(users, 100)
```

还可以通过 `gorm.Config{}` 指定 ` CreateBatchSize` 成员配置批次大小

```go
db, err := gorm.Open(sqlite.Open("gorm.db"), &gorm.Config{
  CreateBatchSize: 1000,
})

db := db.Session(&gorm.Session{CreateBatchSize: 1000})
```

### 根据 Map 创建

GORM 支持通过 `map[string]interface{}` 与 `[]map[string]interface{}{}`来创建记录。

```go
db.Model(&User{}).Create(map[string]interface{}{
  "Name": "jinzhu", "Age": 18,
})

// 从 []map[string]interface{}{} 中批量添加
db.Model(&User{}).Create([]map[string]interface{}{
  {"Name": "jinzhu_1", "Age": 18},
  {"Name": "jinzhu_2", "Age": 20},
})
```

> [!tip] 不建议使用 Map 
> 
> 屏蔽 GORM 自动管理的字段
> 
> 不会触发钩子
> 

## 单表查询

### 查询单个对象

GORM 提供了 `db.First()` `db.Take()` `db.Last()` 方法，以便 **从数据库中 _检索单个对象_**。当查询数据库时它添加了 `LIMIT 1` 条件，且没有找到记录时，它会返回 `ErrRecordNotFound` 错误

```go
func First(db *gorm.DB) {
	var employee models.Employee

	tx := db.First(&employee) 
	// SELECT * FROM "employees" ORDER BY "employees"."employee_id" LIMIT 1

	if tx.Error != nil {
		if errors.Is(tx.Error, gorm.ErrRecordNotFound) {
			log.Println("Employee not exist")
			return
		} 
		log.Fatalf("Error while finding employee: %v", tx.Error)
	}

	fmt.Printf("Query employee: %+v", employee)
}

func Last(db *gorm.DB) {
	var employee models.Employee

	tx := db.Last(&employee)
	// SELECT * FROM "employees" ORDER BY "employee"."employee_id" DESC LIMIT 1

	if tx.Error != nil {
		if errors.Is(tx.Error, gorm.ErrRecordNotFound) {
			log.Printf("Employee not found")
			return
		} else {
			log.Fatalf("Error while fetching employee: %v", tx.Error)
		}
	}
	fmt.Printf("Employee found: %#v", employee)
}

func Take(db *gorm.DB) {

	var employee models.Employee

	tx := db.Take(&employee)
	// SELECT * FROM "employee" LIMIT 1
	
	if tx.Error != nil {
		if errors.Is(tx.Error, gorm.ErrRecordNotFound) {
			log.Printf("Employee not found")
			return
		} else {
			log.Fatalf("Error while fetching employee: %v", tx.Error)
		}
	}
	fmt.Printf("Employee found: %#v", employee)
}
```

> [!tip]
> 
> **First** 和 **Last** 默认会 **按照 _主键_ 进行排序**。如果模型没有主键，则按照第一个成员进行排序
> 
> First 和 Last 只有在 **目标 struct 是指针** 或者 通过 `db.Model()` 指定 model 时，该方法才有效。
> 

### 查询全部对象

GORM 提供的 `db.Find()` 方法可以获取表中的全部记录

```go
func Find(db *gorm.DB) {
	var employees []*models.Employee

	tx := db.Find(&employees)
	// SELECT * FROM "employees"

	if tx.Error != nil {
		log.Fatalf("find employees failed: %v", tx.Error)
	}

	fmt.Printf("find %v records\n", len(employees))
}
```

### 指定表查询

如果需要选择特定的表，可以通过 `db.Model()` 或者 `db.Table(tb_name)` 进行查询

```go
func Model(db *gorm.DB) {
	var employee models.Employee

	tx := db.Model(&models.Employee{}).First(&employee)
	// SELECT * FROM "employees" ORDER BY "employees"."employee_id" LIMIT 1

	if tx.Error != nil {
		if errors.Is(tx.Error, gorm.ErrRecordNotFound) {
			log.Println("Employee not exist")
			return
		} else {
			log.Fatalf("Error while finding employee: %v", tx.Error)
		}
	}

	fmt.Printf("Query employee: %+v", employee)
}

func Table(db *gorm.DB) {
	var employee models.Employee

	tx := db.Table("employees").First(&employee)
	// SELECT * FROM "employees" ORDER BY "employees"."employee_id" LIMIT 1

	if tx.Error != nil {
		if errors.Is(tx.Error, gorm.ErrRecordNotFound) {
			log.Println("Employee not exist")
			return
		} else {
			log.Fatalf("Error while finding employee: %v", tx.Error)
		}
	}
	fmt.Printf("Query employee: %+v", employee)
}
```

### 条件

GORM 使用  `db.Where()` 指定条件。提供给 `db.Where()` 的条件有两种：**字符串形式** 和 **Struct/Map形式**

#### 字符串条件

字符串形式的条件就是 SQL 语句中 `where` 子句的条件部分直接以字符串的方式提供即可

```go
func String(db *gorm.DB) {
	// 查找 employee_id = 107 的 employee
	var employee models.Employee

	tx := db.Where("employee_id = ?", 107).Take(&employee)
	// ELECT * FROM "employees" WHERE employee_id = 107 LIMIT 1

	if tx.Error != nil {
		if errors.Is(tx.Error, gorm.ErrRecordNotFound) {
			log.Printf("employee_id = %v not exists\n", 107)
			return
		} else {
			log.Fatalf("query apprea error: %v\n", tx.Error)
		}
	}

	fmt.Printf("employee is %#v", employee)
}
```

诸如 `AND` `OR` `NOT`  `IN` `LIKE` `BETWEEN ... AND ...` 等都可以直接使用字符串形式条件提供

```go
func String(db *gorm.DB) {
	// 查找 employee_id 在 107 ~ 110 之间的 employees
	var employees []*models.Employee

	tx := db.Where("employee_id BETWEEN $1 AND $2", 107, 110).Find(&employees)
	// SELECT * FROM "employees" WHERE employee_id BETWEEN 107 AND 110
	
	if tx.Error != nil {
		log.Fatalf("Query employee error: %v", tx.Error)
	}

	fmt.Printf("query %d employees\n", len(employees))
}
```

#### Struct & Map 条件

向 `Where` 传递一个结构体，会将结构体中的字段当做 `AND` 条件

```go
func Struct(db *gorm.DB) {
	// 查找 employee_id = 107 AND first_name = 'Diana' 的 employee
	var employee = models.Employee{}

	tx := db.Where(&models.Employee{
		EmployeeID: 107,
		FirstName:  "Diana",
	}).Find(&employee)
	//  SELECT * FROM "employees" WHERE "employees"."employee_id" = 107 AND "employees"."first_name" = 'Dianna'
	if tx.Error != nil {
		log.Fatalf("Query employee error: %v", tx.Error)
	}

	fmt.Printf("employee is %#v\n", employee)
}
```

同理，向 `Where` 传递一个 Map，会将所有的 `key-value` 对当做 `AND` 条件

```go
func Map(db *gorm.DB) {
	// 查找 employee_id = 107 AND first_name = 'Diana' 的 employee
	var employee = models.Employee{}

	tx := db.Where(map[string]interface{}{
		"first_name":  "Diana",
		"employee_id": 107,
	}).Find(&employee)
	// SELECT * FROM "employees" WHERE "employee_id" = 107 AND "first_name" = 'Diana'

	if tx.Error != nil {
		log.Fatalf("Query employee error: %v", tx.Error)
	}
	fmt.Printf("employee is %#v\n", employee)
}
```

对于主键，可以传递一个切片，当做 `IN` 条件

```go
func Slice(db *gorm.DB) {
	// 查找 employee_id IN (107, 109, 110) 的 employee
	var employees = []*models.Employee{}

	tx := db.Where([]int{107, 108, 110}).Find(&employees)
	// SELECT * FROM "employees" WHERE "employees"."employee_id" IN (107,108,110)
	
	if tx.Error != nil {
		log.Fatalf("Query employee error: %v", tx.Error)
	}
	fmt.Printf("employees is %#v\n", employees)
}
```

#### 内联条件

`db.Find()` `db.First()` `db.Last()` `db.Take()` 都支持 **内联条件**

如果根据 **主键** 查找，那么可以直接 **传递主键的值** 即可

```go
var employees []*models.Employee

// 查找 employee_id in (101, 102, 103)
tx := db.Find(&employees, []int{101, 102, 103})
// SELECT * FROM "employees" WHERE "employees"."employee_id" IN (101,102,103)

var employee models.Employee

// 查找 employee_id = 109
tx := db.First(&employee, 109)
// SELECT * FROM "employees" WHERE "employees"."employee_id" = 109 ORDER BY "employees"."employee_id" LIMIT 1
```

 也可以像 `db.Where()` 一样传递条件

```go
var employee models.Employee
tx := db.Last(&employee, "last_name ILIKE $1", "D%")
// SELECT * FROM "employees" WHERE last_name ILIKE 'D%' ORDER BY "employees"."employee_id" DESC LIMIT 1
```

```go
var employee models.Employee
tx := db.Take(&employee, &models.Employee{
	JobID: "SH_CLERK",
})
// SELECT * FROM "employee" LIMIT 1 WHERE "employee"."job_id" = 'SH_CLERK' LIMIT 1
```

#### Not 条件

构建 `NOT` 条件，工作方式类似于 `db.Where`

```go
var employees []models.Employee  
// 字符串条件  
tx := db.Not("last_name LIKE ?", "W%").Find(&employees)  
// SELECT * FROM "employees" WHERE NOT "employees"."last_name" LIKE 'W%'

employees = []models.Employee{}
// Map 条件
tx = db.Not(map[string]interface{}{"last_name": []string{"Baer", "Higgins", "Gietz"}}).Find(&employees)  
// SELECT * FROM "employees" WHERE "employees"."last_name" NOT IN ('Baer', 'Higgins', 'Gietz')

employees = []models.Employee{}
// Stuct 条件
tx = db.Not(&models.Employee{LastName: "Higgins", ManagerID: 101}).Find(&employees)
// SELECT * FROM "employees" WHERE ("employees"."last_name" <> 'Higgins' AND "employees"."manager_id" <> 101)


employees = []models.Employee{}
// 主键的切片
tx = db.Not([]int{101, 102, 103}).Find(&employees)
// SELECT * FROM "employees" WHERE "employees"."employee_id" NOT IN (101,102,103)
```

#### Or 条件

```go
// 字符串条件
tx := db.Where([]int{107, 108, 110}).Or("first_name IN (?, ?)", "Steven", "Bruce").Find(&employees)
// SELECT * FROM "employees" WHERE "employees"."employee_id" IN (107,108,110) OR first_name IN ('Steven', 'Bruce')
	
// Struct 条件
tx := db.Where([]int{107, 108, 110}).Or(&models.Employee{
	Salary: 6000,
}).Find(&employees)
// SELECT * FROM "employees" WHERE "employees"."employee_id" IN (107,108,110) OR "employees"."salary" = 6000

// Map 条件
tx := db.Where([]int{107, 108, 110}).Or(map[string]interface{}{"first_name": []string{"Steven", "Bruce"}}).Find(&employees)
// SELECT * FROM "employees" WHERE "employees"."employee_id" IN (107,108,110) OR first_name IN ('Steven', 'Bruce')

```

### 选择特定字段

`db.Select()` 可以指定从数据库中检索的字段。如果不指定，GORM 默认全选字段

```go
tx := db.Model(&models.Employee{}).Select("employee_id, first_name, last_name, salary").Where("employee_id = ?", 107).Find(&result)
// SELECT employee_id, first_name, last_name, salary FROM "employees" WHERE employee_id = 107
```

GORM 可以进行 **智能字段选择**

```go
var results []struct {
	EmployeeID uint
	FirstName  string
	LastName   string
	Salary     float64
}
tx := db.Model(&models.Employee{}).Limit(10).Find(&results)
// SELECT "employees"."employee_id","employees"."first_name","employees"."last_name","employees"."salary" FROM "employees" LIMIT 1
```

> [!tip] 
> 选择特定字段，通常的做法就是使用 `db.Model()` 或 `db.Table()` 首先指定表

### 排序

`db.Order()` 可以指定排序字段

```go
db.Order("salary DESC").Find(&employees)
// SELECT * FROM "employees" ORDER BY salary DESC

db.Order("salary DESC, employee_id").Find(&employees)
// SELECT * FROM "employees" ORDER BY salary DESC, employee_id
```

### Limit & Offset 分页

```go
db.Model(&models.Employee{}).Order("employee_id DESC").Limit(10).Offset(10).Find(&employees)
// SELECT * FROM "employees" ORDER BY employee_id DESC LIMIT 10 OFFSET 10
```

### Group & Having 分组与聚合

```go
db.Model(&models.Employee{}).Select("department_id, COUNT(*) AS total").Group("department_id").Find(&results)
// SELECT department_id, COUNT(*) AS total FROM "employees" GROUP BY "department_id"

db.Model(&models.Employee{}).Select("department_id, COUNT(*) AS total").Where("department_id IS NOT NULL").Group("department_id").Having("COUNT(*) > 5").Find(&results)
// SELECT department_id, COUNT(*) AS total FROM "employees" WHERE department_id IS NOT NULL GROUP BY "department_id" HAVING COUNT(*) > 5
```

### Distinct 去重

```go
db.Distinct("name", "age").Order("name, age desc").Find(&results)
```

### Rows & Scan

```go
// 返回所有行
rows, err := db.Model(&models.Employee{}).Select("employee_id, first_name, last_name").Where("employee_id > ? AND employee_id < ?", 107, 117).Rows()

if err != nil {
	panic(err)
}

defer rows.Close()

var result struct {
	EmployeeId int
	FirstName  string
	LastName   string
}

// Scan
for rows.Next() {
	rows.Scan(&result.EmployeeId, &result.FirstName, &result.LastName)
	fmt.Printf("%v\n", result)
}

```

## 单表更新

> [!attention] GORM 会阻止不带任何条件的 `Update/Updates`，并且返回 `ErrMissingWhereClause` 错误

### 更新所有字段

`Save` 会保存所有的字段，即使字段是零值。下面是 GORM 官方给的示例

```go
db.First(&user)

user.Name = "jinzhu 2"
user.Age = 100
db.Save(&user)
// UPDATE users SET name='jinzhu 2', age=100, birthday='2016-01-01', updated_at = '2013-11-17 21:34:10' WHERE id=111;
```

`Save` 是一个组合函数。如果 `Save` 的值不包含主键，它将执行 `Create`，否则它将执行 `Update` (包含所有字段)。

```go
db.Save(&User{Name: "jinzhu", Age: 100})
// INSERT INTO `users` (`name`,`age`,`birthday`,`update_at`) VALUES ("jinzhu",100,"0000-00-00 00:00:00","0000-00-00 00:00:00")

db.Save(&User{ID: 1, Name: "jinzhu", Age: 100})
// UPDATE `users` SET `name`="jinzhu",`age`=100,`birthday`="0000-00-00 00:00:00",`update_at`="0000-00-00 00:00:00" WHERE `id` = 1
```

> [!tip] Save 方法会更新所有的字段，即使字段是零值

### 更新单列

如果只需要更新一个字段，可以使用 `Update()` 方法

> [!tip]
> 
> **Update** 方法的使用必须要有条件限制，否则 **GORM 会阻止进行全表更新**，并返回 `ErrMissingWhereClause` 错误

当使用 `Model` 方法，并且它有主键值时，主键将会被用于构建条件

```go
// 根据条件更新
db.Model(&User{}).Where("active = ?", true).Update("name", "hello")
// UPDATE users SET name='hello', updated_at='2013-11-17 21:34:10' WHERE active=true;

// User 的 ID 是 `111`
db.Model(&user).Update("name", "hello")
// UPDATE users SET name='hello', updated_at='2013-11-17 21:34:10' WHERE id=111;

// 根据条件和 model 的值进行更新
db.Model(&user).Where("active = ?", true).Update("name", "hello")
// UPDATE users SET name='hello', updated_at='2013-11-17 21:34:10' WHERE id=111 AND active=true;
```

### 更新多列

如果需要更新多列，使用 `Updates()` 方法。该方法支持 `Struct & Map` 参数

> [!tip]
> 
> 如果传递个 **Updates** 的是 `Struct`，则更新 **只会更新非零字段** 
> 

```go
// 根据 struct 更新属性，只会更新非零值的字段
db.Model(&user).Updates(User{Name: "hello", Age: 18, Active: false})
// UPDATE users SET name='hello', age=18, updated_at = '2013-11-17 21:34:10' WHERE id = 111;

// 根据 map 更新属性
db.Model(&user).Updates(map[string]interface{}{"name": "hello", "age": 18, "active": false})
// UPDATE users SET name='hello', age=18, active=false, updated_at='2013-11-17 21:34:10' WHERE id=111;
```

### 更新选定字段

在更新时可以使用 `Select` 选定字段，或者使用 `Omit` 忽略某些字段

```go
// 选择 Map 的字段
// User 的 ID 是 `111`:
db.Model(&user).Select("name").Updates(map[string]interface{}{"name": "hello", "age": 18, "active": false})
// UPDATE users SET name='hello' WHERE id=111;

// 忽略 Omit 指定字段
db.Model(&user).Omit("name").Updates(map[string]interface{}{"name": "hello", "age": 18, "active": false})
// UPDATE users SET age=18, active=false, updated_at='2013-11-17 21:34:10' WHERE id=111;

// 选择 Struct 的字段（会选中零值的字段）
db.Model(&user).Select("Name", "Age").Updates(User{Name: "new_name", Age: 0})
// UPDATE users SET name='new_name', age=0 WHERE id=111;

// 选择所有字段（选择包括零值字段的所有字段）
db.Model(&user).Select("*").Updates(User{Name: "jinzhu", Role: "admin", Age: 0})

// 选择除 Role 外的所有字段（包括零值字段的所有字段）
db.Model(&user).Select("*").Omit("Role").Updates(User{Name: "jinzhu", Role: "admin", Age: 0})
```

> [!tip]
> 使用 Select 选定字段时，如果选定字段指定的值是零值，也会更新

### 批量更新

如果我们没有在 `Model` 中指定有主键值的记录，GORM 会执行批量更新

> [!tip]
> 如果使用 `Model` 选中表，但是 **没有在其中指定主键**，则 GORM 执行批量更新

```go
// Update with struct
db.Model(User{}).Where("role = ?", "admin").Updates(User{Name: "hello", Age: 18})
// UPDATE users SET name='hello', age=18 WHERE role = 'admin';

// Update with map
db.Table("users").Where("id IN ?", []int{10, 11}).Updates(map[string]interface{}{"name": "hello", "age": 18})
// UPDATE users SET name='hello', age=18 WHERE id IN (10, 11);
```

### 更新 Hook

更新时可用的 hook

```go
// 开始事务
BeforeSave
BeforeUpdate
// 关联前的 save
// 更新 db
// 关联后的 save
AfterUpdate
AfterSave
// 提交或回滚事务
```

```go
func (u *User) BeforeUpdate(tx *gorm.DB) (err error) {
  if u.readonly() {
    err = errors.New("read only user")
  }
  return
}

// 在同一个事务中更新数据
func (u *User) AfterUpdate(tx *gorm.DB) (err error) {
  if u.Confirmed {
    tx.Model(&Address{}).Where("user_id = ?", u.ID).Update("verfied", true)
  }
  return
}
```

## 单表删除

> [!attention] GORM 阻止不带任何条件的 `Delete`，并且返回 `ErrMissingWhereClause` 错误

### 删除一条记录

调用 `Delete` 方法 **删除一条记录时，删除对象需要 _指定主键_**，否则会触发 **批量删除**

> [!tip]
> 如果删除对象的主键未指定，则会触发批量删除


```go
// Email 的 ID 是 `10`
db.Delete(&email)
// DELETE from emails where id = 10;

// 带额外条件的删除
db.Where("name = ?", "jinzhu").Delete(&email)
// DELETE from emails where id = 10 AND name = "jinzhu";
```

### 根据主键删除

GORM 允许通过 **主键**(可以是复合主键)和 **内联条件** 来删除对象

```go
db.Delete(&User{}, 10)
// DELETE FROM users WHERE id = 10;

db.Delete(&User{}, "10")
// DELETE FROM users WHERE id = 10;

db.Delete(&users, []int{1,2,3})
// DELETE FROM users WHERE id IN (1,2,3);
```

### 批量删除

如果指定的值不包括主属性，那么 GORM 会执行批量删除，它将删除所有匹配的记录

```go
db.Where("email LIKE ?", "%jinzhu%").Delete(&Email{})
// DELETE from emails where email LIKE "%jinzhu%";

db.Delete(&Email{}, "email LIKE ?", "%jinzhu%")
// DELETE from emails where email LIKE "%jinzhu%";
```

可以将一个主键切片传递给 `Delete` 方法，以便更高效的删除数据量大的记录

```go
var users = []User{{ID: 1}, {ID: 2}, {ID: 3}}
db.Delete(&users)
// DELETE FROM users WHERE id IN (1,2,3);

db.Delete(&users, "name LIKE ?", "%jinzhu%")
// DELETE FROM users WHERE name LIKE "%jinzhu%" AND id IN (1,2,3); 
```

### 返回删除行的数据

返回被删除的数据，仅当数据库支持回写功能时才能正常运行，如下例：

```go
// 回写所有的列
var users []User
db.Clauses(clause.Returning{}).Where("role = ?", "admin").Delete(&users)
// DELETE FROM `users` WHERE role = "admin" RETURNING *
// users => []User{{ID: 1, Name: "jinzhu", Role: "admin", Salary: 100}, {ID: 2, Name: "jinzhu.2", Role: "admin", Salary: 1000}}

// 回写指定的列
db.Clauses(clause.Returning{Columns: []clause.Column{{Name: "name"}, {Name: "salary"}}}).Where("role = ?", "admin").Delete(&users)
// DELETE FROM `users` WHERE role = "admin" RETURNING `name`, `salary`
// users => []User{{ID: 0, Name: "jinzhu", Role: "", Salary: 100}, {ID: 0, Name: "jinzhu.2", Role: "", Salary: 1000}}
```

### 软删除

> [!tip] 如果模型包含了 `gorm.DeletedAt` 字段，那么该模型自动获得 **软删除** 的能力
> 当调用 `Delete` 时，GORM 并 **不会从数据库中删除该记录**，而是将该记录的 `DeleteAt` 设置为当前时间，而后的一般查询方法将无法查找到此条记录

```go
// user's ID is `111`
db.Delete(&user)
// UPDATE users SET deleted_at="2013-10-29 10:23" WHERE id = 111;

// Batch Delete
db.Where("age = ?", 20).Delete(&User{})
// UPDATE users SET deleted_at="2013-10-29 10:23" WHERE age = 20;

// Soft deleted records will be ignored when querying
db.Where("age = 20").Find(&user)
// SELECT * FROM users WHERE age = 20 AND deleted_at IS NULL;
```

如果你并不想嵌套`gorm.Model`，你也可以像下方例子那样开启软删除特性

```go
type User struct {
  ID      int
  Deleted gorm.DeletedAt
  Name    string
}
```

### 删除 Hook

删除时可用的 hook

```go
// 开始事务
BeforeDelete
// 删除 db 中的数据
AfterDelete
// 提交或回滚事务

```

```go
func (u *User) BeforeDelete(tx *gorm.DB) (err error) {
    if u.Role == "admin" {
        return errors.New("admin user not allowed to delete")
    }
    return
}

// 在同一个事务中更新数据
func (u *User) AfterDelete(tx *gorm.DB) (err error) {
	if u.Confirmed {
		tx.Model(&Address{}).Where("user_id = ?", u.ID).Update("invalid", false)
	}
	return
}
```

## 关联关系

这里我们说的关系是实体与实体之间的关系

### 一对多

GORM 将一对多关系分为了 `Belongs To`（属于谁）和 `Has Many`（我拥有的）两类。但是归根结底它们是同一种关系。

例如，一个博客网站的 **用户** 和用户发布的 **文章** 就是这种一对多的关系：一个用户可以拥有多篇文章，一篇文章只属于一个用户

我们站在文章角度来看，文章就是 `Belongs To` 用户的。我需要在文章模型中建立外键关系

```go
// Belongs To
type User struct {
	ID   int
	Name string
}

type Article struct {
	ID      int
	Name    string
	Content string
	UserID  int  // 约定 {User}ID 字段默认用来与 User 建立外键关系
	User    User // 约定 使用 User.ID 作为参考字段
}
```

> [!tip]
> **UserID 是外键**，用于维护关联关系
> 
> **User 是关联字段**，通过关联字段可以获取关联的实例

站在用户角度来看，就是一个用户 `Has Many`文章。同样需要 `Article` 中建立外键，但是关系写在模型 `User` 中

```go
// Has Many
type User struct {
	ID      int
	Name    string `gorm:"unique"`
	Article []Article
}

type Article struct {
	ID      int
	Name    string
	Content string
	UserID  int // 用于建立外键关系的字段
}
```

如果需要同时支持 `Belongs To` 和 `Has Many`

```go
type User struct {
	ID       int
	Name     string    `gorm:"unique"`
	Articles []Article // Has Many Article
}

type Article struct {
	ID      int
	Name    string
	Content string
	UserID  int  // 用于建立外键关系的字段
	User    User // Belongs To User
}
```

> [!attention] 
> 
> **外键字段的类型应该和父表主键类型的保持相同**
> 

#### 重写外键

> [!tip] 约定：**外键名** 使用 `父模型名` + `父模型的主键字段名`
> 

GORM 同时提供自定义外键名字的方式：使用 `gorm:"foreignKey:外键字段名"`

```go
type User struct {
	ID       int
	Name     string    `gorm:"unique"`
	Articles []Article `gorm:"foreignKey:UID"` // 重写外键
}

type Article struct {
	ID      int
	Name    string
	Content string
	UID     int
	User    User `gorm:"foreignKey:UID"` // 重写外键
}
```

> [!tip] 外键在关联字段上进行重写

#### 重写引用

> [!tip] 约定：外键默认 **引用** 的父模型的主键字段

也可以使用标签 `references` 来更改它：`gorm:"references:引用父模型的字段名"`

```go
type User struct {
	ID       int
	Name     string    `gorm:"unique;size:25"`
	Articles []Article `gorm:"foreignKey:UserName;references:Name"` // 重写外键, 重写引用
}

type Article struct {
	ID       int
	Name     string
	Content  string
	UserName string `gorm:"size:25"`
	User     User   `gorm:"foreignKey:UserName;references:Name"` // 重写外键， 重写引用
}
```

> [!tip] 引用在关联字段上重写


> [!attention]  外键字段最好不要和父模型的主键字段同名，否则无法同时建立 `Has Many` 和 `Belongs To`  关系

#### 外键约束

还可以通过  `constraint` 标签配置 `OnUpdate`、`OnDelete` 实现外键约束，在使用 GORM 进行迁移时它会被创建

```go
type User struct {
	ID       int
	Name     string    `gorm:"unique;size:25"`
	Articles []Article `gorm:"foreignKey:UserName;references:Name;constraint:OnUpdate:CASCADE,OnDelete:SET NULL"` // 重写外键, 重写引用
}

type Article struct {
	ID       int
	Name     string
	Content  string
	UserName string `gorm:"size:25"` // 设置了外键约束，主表的被参照字段被更新时级联更新该字段；被删除时 ，该字段设置为 NULL
	User     User   `gorm:"foreignKey:UserName;references:Name;constraint:OnUpdate:CASCADE,OnDelete:SET NULL"` // 重写外键， 重写引用
}
```

> [!tip] 外键约束也是写在关联字段上的
> 

### 一对一

GORM 中一对一关系为 `Has One`（我有一个）

例如，**用户** 与 **用户信息**，一个用户只有一个用户信息，而一个用户信息也只属于一个用户

```go
type User struct {
	ID     int
	Name   string `gorm:"unique;size:25"`
	Gender string `gorm:"type:ENUM('男', '女', '保密')"`

	UserInfo UserInfo // 关联字段
}

type UserInfo struct {
	ID          int
	UserID      int    // 外键字段
	PhoneNumber string `gorm:"unique;size:25"`
	Address     string `gorm:"size:255"`
}
```

> [!tip] 同一对多关系一样，也可以重写外键，重写引用，以及设置外键约束

### 多对多

GORM 中多对多关系为 `Many To Many`

例如，博客网站中 **文章** 和 **标签**：一篇文章可以属于多个标签，一个标签可以拥有多篇文章


使用 `gorm:"many2many:连接表名"` 创建多对多关系

```go
type Article struct {
	ID      int
	Name    string
	Content string
	UserID  int
	Tags    []*Tag `gorm:"many2many:article_tags;"`
}

type Tag struct {
	ID       int
	Name     string    `gorm:"unique;size:25"`
	Articles []*Article `gorm:"many2many:article_tags;"`
}
```

> [!tip] 当使用 GORM 的 `AutoMigrate` 为 `User` 创建表时，**GORM 会自动创建连接表**

#### 自定义连接表

让 GORM 自动创建连接表，我们对连接表的控制就全部交给了 GORM。GORM 运行自行创建连接表模型

```go
type Article struct {
	ID      int
	Name    string
	Content string
	UserID  int
	Tags    []*Tag `gorm:"many2many:article_tags;"`
}

type Tag struct {
	ID       int
	Name     string     `gorm:"unique;size:25"`
	Articles []*Article `gorm:"many2many:article_tags;"`
}

type ArticleTag struct {
	ArticleID int `gorm:"primary_key"` // 外键 参考 Article.ID
	TagID     int `gorm:"primary_key"` // 外键 参考 Tag.ID
	// 添加其他字段
}

// 修改 Article 的 Tags 的连接表为 ArticleTag
db.SetupJoinTable(&Article{}, "Tags", &ArticleTag{})
// 或者 修改 Tag 的 Articles 的连接表为 ArticleTag
db.SetupJoinTable(&Tag{}, "Articles", &ArticleTag{})
```

#### 重写外键

对于 `many2many` 关系，**连接表会同时拥有两个模型的外键**。

```go
type Article struct {
	ID      int
	Name    string
	Content string
	Tags    []*Tag `gorm:"many2many:article_tags"`
}

type Tag struct {
	ID       int
	Name     string     `gorm:"unique;size:25"`
	Articles []*Article `gorm:"many2many:article_tags"`
}

// 连接表 article_tags
// 外键: article_id, 参考: users.id
// 外键: tag_id, 参考: tags.id
```

若要重写它们，可以使用标签 `foreignKey` `references` `joinforeignKey` `joinReferences`

> [!tip]
> 当然，**不需要使用全部的标签**，可以仅使用其中的一个重写部分的外键、引用
> 
> `foreignKey` 和 `references` 是重写自己的外键和引用
> 
> `joinForeignKey` 和 `joinReferences` 是重写关联表的外键和引用
> 

```go
type Article struct {
	ID      int
	Name    string
	Content string
	UserID  int    // 设置了外键约束，主表的被参照字段被更新时级联更新该字段；被删除时 ，该字段设置为 NULL                                                                 //
	User    User   `gorm:"constraint:OnUpdate:CASCADE,OnDelete:SET NULL"` // 重写外键， 重写引用
	Tags    []*Tag `gorm:"many2many:article_tags;joinForeignKey:AID;joinReferences:TID"`
}

type Tag struct {
	ID       int
	Name     string     `gorm:"unique;size:25"`
	Articles []*Article `gorm:"many2many:article_tags;joinForeignKey:TID;joinReferences:AID"`
}

type ArticleTag struct {
	AID     int     `gorm:"primaryKey"`
	TID     int     `gorm:"primaryKey"`
	Article Article `gorm:"foreignKey:AID;references:ID"`
	Tag     Tag     `gorm:"foreignKey:TID;references:ID"`
}
```

### 关联关系演示

```go
package models

import "time"

// Employee 雇员表
type Employee struct {
	EmployeeID  int       `gorm:"primaryKey;size:32"` // 数字主键，自动创建序列
	FirstName   string    `gorm:"size:20"`
	LastName    string    `gorm:"size:25;not null"`
	Email       string    `gorm:"size:25;not null"`
	PhoneNumber string    `gorm:"size:20"`
	HireDate    time.Time `gorm:"type:date;not null"`

	// Employee 和 Job：一个 Employee 可以完成一份 Job，一份 Job 需要多个 Employee 完成
	JobReferID string `gorm:"size:10;not null;column:job_id"` // 外键
	Job        *Job   `gorm:"foreignKey:JobReferID;references:JobID"`

	Salary        float64 `gorm:"type:float8"`
	CommissionPct float64 `gorm:"type:float8"`

	// 自引用： Employee 和 Manager(本身也是 Employee) 一个员工由一个管理员，一个管理员可以管理多个员工
	ManagerID int         `gorm:"size:32"`
	Manager   *Employee   `gorm:"foreignKey:ManagerID;references:EmployeeID"`
	Employees []*Employee `gorm:"foreignKey:ManagerID;references:EmployeeID"`

	// Employee 和 Department：一个 Employee 属于一个 Department，一个 Department 拥有多个员工
	DepartmentReferID int         `gorm:"size:32;column:department_id"`
	Department        *Department `gorm:"foreignKey:DepartmentReferID;references:DepartmentID"`
}

// Department 部门表  
type Department struct {  
    DepartmentID   int    `gorm:"primaryKey;size:32"`  
    DepartmentName string `gorm:"size:30;not null"`  
  
    // Department 与 Employee(管理员)：一个 Department 只有一个 Manager，一个 Manager 只管理一个 Department  
    ManagerID int  
    Manager   *Employee `gorm:"foreignKey:ManagerID;references:EmployeeID"`  
  
    // Department 和 Location：：一个 Department 只在一个 Location，一个 Location 可以拥有多个 Department  
    LocationReferID int       `gorm:"size:32;column:location_id"`  
    Location        *Location `gorm:"foreignKey:LocationReferID;references:LocationID"`  
  
    // Employees 可查当前部门的所有员工  
    Employees []*Employee `gorm:"foreignKey:DepartmentReferID;references:DepartmentID"`  
}


// Job 工作表
type Job struct {
	JobID     string `gorm:"size:10;primaryKey"`
	JobTitle  string `gorm:"size:35;not null"`
	MinSalary int    `gorm:"size:32"`
	MaxSalary int    `gorm:"size:32"`

	// 可以查看到完成 Job 的所有 Employees
	Employees []*Employee `gorm:"foreignKey:JobReferID;references:JobID"`
}

// JobHistory 历史工作表
type JobHistory struct {
	EmployeeReferID int       `gorm:"primaryKey;type:int;column:employee_id"`
	Employee        *Employee `gorm:"foreignKey:EmployeeReferID;references:EmployeeID"`

	StartDate time.Time `gorm:"primaryKey;type:date"`
	EndDate   time.Time `gorm:"type:date;not null"`

	JobReferID string `gorm:"size:10;not null;column:job_id"`
	Job        *Job   `gorm:"foreignKey:JobReferID;references:JobID"`

	DepartmentReferID int         `gorm:"size:32;column:department_id"`
	Department        *Department `gorm:"foreignKey:DepartmentReferID;references:DepartmentID"`
}

func (jh *JobHistory) TableName() string {
	return "job_history"
}


// Location 位置表
type Location struct {
	LocationID    int    `gorm:"primaryKey;size:32"`
	StreetAddress string `gorm:"size:40"`
	PostalCode    string `gorm:"size:12"`
	City          string `gorm:"size:30;not null"`
	StateProvince string `gorm:"size:25"`

	// Location 和 Country：一个 Location 只属于一个 Country，一个 Country 有多个 Location
	CountryReferID string   `gorm:"type:char(2)"`
	Country        *Country `gorm:"foreignKey:CountryReferID;references:CountryID"`

	// 通过 Departments 可以获取该 Location 中的所有部门
	Departments []*Department `gorm:"foreignKey:LocationReferID;references:LocationID"`
}

// Country 国家表
type Country struct {
	CountryID   string `gorm:"type:char(2);primaryKey"`
	CountryName string `gorm:"size:40"`

	// Country 和 Region：一个 Country 只属于一个 Region，而一个 Region 有多个 Country
	RegionReferID int     `gorm:"size:32"`
	Region        *Region `gorm:"foreignKey:RegionReferID;references:RegionID"`

	Locations []*Location `gorm:"foreignKey:CountryReferID;references:CountryID"`
}

// Region 地区表
type Region struct {
	RegionID   int    `gorm:"primaryKey;size:32"`
	RegionName string `gorm:"size:25"`

	Countries []*Country `gorm:"foreignKey:RegionReferID;references:RegionID"`
}
```

## 实体关联

### 自动创建与更新

 > [!tip] 创建或更新记录时，GORM 会自动保存其关联和引用
 > 
 > GORM 在创建或更新记录时会自动地保存其关联和引用，主要使用 `upsert` 技术来更新现有关联的外键引用
 > 

当你创建一条新的记录时，GORM 会自动保存它的关联数据。 这个过程包括 **向关联表插入数据** 以及 **维护外键引用**

```go
var user = User{
	Name:   "杜宇鹏",
	Gender: "男",
	UserInfo: UserInfo{
		PhoneNumber: "19374291900",
		Address:     "sc-zy",
	},
	Articles: []*Article{
		{
			Name:    "Go",
			Content: "Golang",
			Tags: []*Tag{
				{
					Name: "编程语言",
				},
			},
		},
	},
}

db.Create(&user)

```

对于需要全面更新关联数据（不止外键）的情况，就应该使用 `FullSaveAssociations` 方法

```go
// 更新用户并完全更新其所有关联
db.Session(&gorm.Session{FullSaveAssociations: true}).Create(&user)

```

使用`FullSaveAssociations` 方法来确保模型的整体状态，包括其所有关联都反映在了数据库中，从在应用中保持数据的完整性和一致性

### 跳过自动创建与更新

若要在创建、更新时跳过自动保存，可以使用 `Select` 或 `Omit`，例如：

```go
db.Select("Name").Create(&user)
// INSERT INTO "users" ("name") VALUES ('杜玉洋') RETURNING "id"

db.Omit("UserInfo").Save(&user)

// 创建时忽略关联
db.Omit(clause.Associations).Create(&user)
```

在 GORM 中，创建或更新记录时，您可以使用`Select`和`Omit`方法专门 **包含或排除 _关联模型的某些字段_**。

```go
user := User{
  Name:            "jinzhu",
  BillingAddress:  Address{Address1: "Billing Address - Address 1", Address2: "addr2"},
  ShippingAddress: Address{Address1: "Shipping Address - Address 1", Address2: "addr2"},
}

// 创建一个用户，并且同时创建与该用户相关联的 BillingAddress 和 ShippingAddress，但只包括 BillingAddress 的指定字段
db.Select("BillingAddress.Address1", "BillingAddress.Address2").Create(&user)
// SQL: Creates user and BillingAddress with only 'Address1' and 'Address2' fields

// 要创建一个用户以及与其关联的 BillingAddress 和 ShippingAddress，同时排除 BillingAddress 的某些字段
db.Omit("BillingAddress.Address2", "BillingAddress.CreatedAt").Create(&user)
// SQL: Creates user and BillingAddress, omitting 'Address2' and 'CreatedAt' fields
```

### 删除关联

GORM 允许在删除主记录时使用 `Select` 方法删除特定的关联关系（有一个、有很多、多对多）。此功能对于维护数据库完整性和确保相关数据在删除时得到适当管理特别有用。

可以使用 `Select` 指定应将哪些关联与主记录一起删除。

```go
// Delete a user's account when deleting the user
db.Select("Account").Delete(&user)

// Delete a user's Orders and CreditCards associations when deleting the user
db.Select("Orders", "CreditCards").Delete(&user)

// Delete all of a user's has one, has many, and many2many associations
db.Select(clause.Associations).Delete(&user)

// Delete each user's account when deleting multiple users
db.Select("Account").Delete(&users)
```

> [!attention] 注意
> 
> 需要注意的是，**只有当删除记录的主键不为零时，关联才会被删除**。 GORM 使用这些主键作为删除所选关联的条件。
> 

```go
// This will not work as intended
db.Select("Account").Where("name = ?", "jinzhu").Delete(&User{})
// SQL: Deletes all users with name 'jinzhu', but their accounts won't be deleted
```

> [!tip] 这里删除的是从表中的一条记录

### 关联模式

GORM 中的关联模式提供了各种辅助方法来处理模型之间的关系，提供了管理关联数据的有效方法。

> [!tip] 启用关联模式的条件
> 
> 要启动关联模式，请指定 **源模型** 和 **关系的字段名称**。**源模型必须包含主键**，并且 **关系的字段名称应与现有关联匹配**

#### 查询关联

> [!tip] 查询外键字段对饮的记录

例如，一个公司的 **员工** 和 **管理员**，用于管理员本身也是员工，可以通过自引用完成关联

> [!tip] 员工和管理员：典型的一对多关系
> 
>  一名员工只属于一个管理员，一个管理员可以有多名员工
> 

```go
// Employee 雇员表
type Employee struct {
	EmployeeID  int       `gorm:"primaryKey;size:32"` // 数字主键，自动创建序列
	FirstName   string    `gorm:"size:20"`
	LastName    string    `gorm:"size:25;not null"`
	Email       string    `gorm:"size:25;not null"`
	PhoneNumber string    `gorm:"size:20"`
	HireDate    time.Time `gorm:"type:date;not null"`

	JobReferID string `gorm:"size:10;not null;column:job_id"` // 外键

	Salary        float64 `gorm:"type:float8"`
	CommissionPct float64 `gorm:"type:float8"`

	// 自引用： Employee 和 Manager(本身也是 Employee) 一个员工由一个管理员，一个管理员可以管理多个员工
	ManagerID int         `gorm:"size:32"`                                    // 外键 引用 Employee.EmployeeID
	Manager   *Employee   `gorm:"foreignKey:ManagerID;references:EmployeeID"`
	Employees []*Employee `gorm:"foreignKey:ManagerID;references:EmployeeID"`

	DepartmentReferID int `gorm:"size:32;column:department_id"` // 外键 引用 Department.DepartmentID
}
```

> [!tip]
> 
> `Manager` 关联员工的管理员
> 
> `Employees` 关联管理员的员工
> 

既可以通过员工查询管理员

```go
var employee models.Employee
var manager models.Employee
db.Take(&employee, 103)
db.Model(&employee).Association("Manager").Find(&manager)
```

也可以通过管理员查询其手下的员工

```go
// 通过管理员查询员工列表
var employees []models.Employee
db.Model(&manager).Where("salary >= ?", 4800).Association("Employees").Find(&employees)
```

#### 追加关联

为 `many to many`  `has many` 添加新关联，或替换为 `has one`   `belongs to`当前关联

> [!tip] `Append` 主要影响外键引用
> 
> 通常是针对已有数据，给已有数据添加关联
> 

```go
// 给员工添加管理员
var employee models.Employee
var manager models.Employee

db.Take(&employee, 207)
db.Take(&manager, 205)
db.Model(&employee).Association("Manager").Append(&manager)
```

```go
// 给管理员添加员工
var employees []models.Employee
var manager models.Employee

// 查找出需要添加管理员的员工
db.Find(&employees, "employee_id IN ?", []int{208, 209})

// 找出管理员
db.Take(&manager, 205)

db.Model(&manager).Association("Employees").Append(&employees)
```

#### 替换关联

用新的关联替换当前的关联

> [!tip] `Replace` 主要修改外键值

```go
var employees []models.Employee
var manager models.Employee

db.Find(&employees, "employee_id IN ?", []int{208, 209})
db.Take(&manager, 108)

// 替换关联
db.Model(&employees[0]).Association("Manager").Replace(&manager)
db.Model(&employees[1]).Association("Manager").Replace(&models.Employee{EmployeeID: 108})
```

#### 删除关联

删除源和参数之间的关系，仅删除引用

```go
var employees []models.Employee
var manager models.Employee

db.Find(&employees, "employee_id IN ?", []int{208, 209})
db.Take(&manager, 108)

// 删除关联
db.Model(&employees[0]).Association("Manager").Delete(&manager)
db.Model(&employees[1]).Association("Manager").Delete(&models.Employee{EmployeeID: 108})
```

#### 清空关联

删除源和关联之间的所有引用

```go
db.Model(&employees[0]).Association("Manager").Clear()
```

#### 关联计数

获取当前关联的计数，无论是否有条件

```go
// 统计管理员关联员工的数目
db.Model(&manager).Association("Employees").Count()
```

## 预加载

GORM允许使用 `Preload` 通过多个 SQL 中来 **直接加载关系**。比如，普通的查找是无法通过模型的实例带出其关联字段的数据的

```go
var employee models.Employee

db.Find(&employee, 107)
fmt.Printf("%+v\n", employee)
// {EmployeeID:107 FirstName:Diana LastName:Lorentz Email:DLORENTZ PhoneNumber:590.423.5567
// HireDate:1999-02-07 00:00:00 +0000 UTC JobReferID:IT_PROG Salary:4200 CommissionPct:0
// ManagerID:103 Manager:<nil> Employees:[] DepartmentReferID:60}
// Manager 为nil，并没有带出 107 号员工的管理员信息
```

通过预加载，可以带出关联字段的值

```go
db.Preload("Manager").Find(&employee, 107)
fmt.Printf("%+v\n", employee)
// {EmployeeID:107 FirstName:Diana LastName:Lorentz Email:DLORENTZ PhoneNumber:590.423.5567
// HireDate:1999-02-07 00:00:00 +0000 UTC JobReferID:IT_PROG Salary:4200 CommissionPct:0
// ManagerID:103 Manager:0xc0002b1080 Employees:[] DepartmentReferID:60}
// Manager 已经有数据了
```

> [!tip] `Preload()`  需要提供查询模型中的 **关联字段名称**


### Joins 预加载

可以使用单个 SQL `Joins`加载关联，例如

```go
// 通过员工查询管理员
var employee models.Employee

db.Joins("Manager").Find(&employee, 107)
fmt.Printf("%+v\n", employee)
// {EmployeeID:107 FirstName:Diana LastName:Lorentz Email:DLORENTZ PhoneNumber:590.423.5567 
// HireDate:1999-02-07 00:00:00 +0000 UTC JobReferID:IT_PROG Salary:4200 CommissionPct:0 
//ManagerID:103 Manager:0xc00029cdc0 Employees:[] DepartmentReferID:60}
// 会将关联字段 Manager 数据携带出来
```

> [!tip]  向 `Joins()` 提供查询模型中的关联字段名称

