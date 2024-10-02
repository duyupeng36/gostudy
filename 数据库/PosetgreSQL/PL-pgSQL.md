# PL-pgSQL

PL/pgSQL 是 PostgreSQL 数据库系统的过程编程语言。

PL/pgSQL 过程语言添加了许多过程元素，例如 **控制结构** **循环** 和 **复杂计算**，以扩展标准 SQL。它允许在 PostgreSQL 中开发使用普通 SQL 可能无法实现的 **复杂函数** 和 **存储过程**

PL/pgSQL 允许通过创建具有复杂逻辑的 **服务器对象** 来扩展 PostgreSQL 数据库服务器的功能。

>[!tip] L/pgSQL 的设计目的在于：
>创建用户定义的函数、存储过程和触发器
>
>通过添加控制结构（例如 `if-else` `case` 和循环语句）来扩展标准 SQL。
>
>继承所有用户定义的函数、运算符和类型。

从 PostgreSQL 9.0 开始，默认安装 PL/pgSQL。


SQL 是一种查询语言，可让您有效地管理数据库中的数据。但是，**PostgreSQL 只能单独执行 SQL 语句**。这意味着有多个语句，就需要 **逐个执行它们**。这个过程会产生进程间通信和网络开销。

PL/pgSQL 将多个语句包装在一个对象中并将其存储在 PostgreSQL 数据库服务器上。无需逐个向服务器发送多个语句，只需发送一个语句即可执行存储在服务器中的对象

> [!tip] pgSQL 的优点
> 减少应用程序和 PostgreSQL 数据库服务器之间的往返次数
> 
> 避免在应用程序和服务器之间传输即时结果

> [!tip] psSQL 的缺点
> 
> 可能无法移植到其他数据库管理系统
> 
> 难以管理版本并且难以调试
> 
> 软件开发速度较慢，因为 PL/pgSQL 需要许多开发人员不具备的专业技能
> 

PL/pgSQL 中，字符串字面值可以使用 `$<tag>$ ....$<tag>$` 方式包围(`<tag>` 是可选的)。例如

```sql
select $$I'm a string constant$$;
```

> [!tip]
>  
> 使代码更具可读性
> 

## 块结构

PL/pgSQL 是一种块 **结构语言**。以下是 PL/pgSQL 中块的语法

```sql
[ <<label>> ]  -- 标签：可选的，注意 lable 两边的 <<>> 
[ declare
    declarations ] -- 声明部分：可选的
begin
    statements;
	...
end [ label ];  -- end [label]; 标识块结束，label可选
```

> [!tip] 每个块都有两部分
> **declare**：声明部分，这是 _可选的_
> 
> **body**：主体部分，主体部分是不能省略的

**声明部分是声明主体部分中使用的所有变量的地方**。声明部分中的每个语句都以分号( `;` ) 结束。声明变量的语法如下：

```sql
variable_name type = initial_value;
```

以下声明了一个名为 `counter` 变量，其类型为 `int` 且初始值为 $0$

```sql
counter int = 0;
```

还可以使用 `:=` 声明变量

```sql
counter int := 0;
```

**初始值是可选的**。例如，可以声明一个名为`max`且类型为`int`变量，如下所示

```sql
max int;
```

下面的示例说明了一个简单的块。由于该块没有名称，因此称为 **匿名块**

```sql
do $$
<<first_block>>
declare
	film_count integer := 0;
begin
	-- get the number of films
	select count(*)
	into film_count 
	from file;
	
	-- display a message
	raise notice 'The number of films is %', film_count;
end first_block $$;
```

> [!tip]
> 
> **`DO`语句** 不属于该块，**用于执行匿名块**
> 

> [!tip]
> 
> 匿名块必须用单引号引起来。这里我们使用 `$$` 使其可读性更好

PL/pgSQL 允许将一个块放置在另一个块的主体内。嵌套在另一个块内的块称为**子块**。包含子块的封闭块通常称为 **外部块**

通常，将一个大块划分为更小、更具逻辑性的子块。以下示例说明了如何在块内使用子块：

```sql
do
$$
<<outer>>
declare
   x int = 0;
begin
   x = x + 1;
   <<inner>>
   declare
       y int = 2;
   begin
   	   y = y + x;
	   raise notice 'x=% y=%', x, y;
   end inner;
end outer;
$$
```

## 变量




