# psql命令

## 连接 PostgreSQL 数据库

```shell
# 以下命令连接到特定用户下的数据库。按Enter后 PostgreSQL 将询问用户的密码。
psql -d database -U  user -W

# 如果要连接到驻留在另一台主机上的数据库，请添加 -h 选项
psql -h host -d database -U user -W

# 如果想使用 SSL 模式进行连接，只需指定它
psql -U user -h host "dbname=db sslmode=require"
```


下面是建立连接之后可以执行的命令

## 切换连接到新数据库

```
\c dbname username
```

## 列出可用的数据库

```
\l
```

## 列出可用表

```
\dt
```

## 查看表结构

```
\d table_name
```

## 列出可用模式

```
\dn
```

## 列出可用函数

```
\df
```

## 列出可用视图

```
\dv
```

## 列出用户及其角色

```
\du
```

## 执行上一条命令

```
\g
```

## 命令历史

要显示命令历史记录

```
\s
```

将命令历史记录保存到文件中

```
\s filename
```

## 从文件执行psql命令

```
\i filename
```

## 获取有关 psql 命令的帮助

```
\?
```

要获取有关特定 PostgreSQL 语句的帮助，请使用`\h`命令

```
\h ALTER TABLE
```

## 打开查询执行时间

要打开查询执行时间，可以使用`\timing`命令

```
dvdrental=# \timing
Timing is on.
dvdrental=# select count(*) from film;
 count
-------
  1000
(1 row)

Time: 1.495 ms
dvdrental=#
```

您可以使用相同的命令`\timing`将其关闭

```
dvdrental=# \timing
Timing is off.
dvdrental=#
```

## 退出psql

```
\q
```
