---
File: python/未命名.md
---
# vim 编辑器

`vim` 是一个命令行的文本编辑器。`vim` 有三种模式：**命令模式**  **编辑模式** 和 **末行模式**

当使用 `vim` 打开文件时，`vim` 默认处于命令模式

> [!tip] 从命令模式进入编辑模式
> + `i` 在光标处插入
> + `I` 在光标行首插入
> + `a` 在光标后插入
> + `A` 在光标行尾插入
> + `o` 在光标下一行插入
> + `O` 在光标上一行插入

> [!tip] 从编辑模式进入命令模式
> + `esc` 从编辑模式进入命令模式

## 命令模式

在命令模式下，支持操作有很多

### 删除与复制

| 命令      | 描述                  |
| :------ | :------------------ |
| `x`     | 删除光标处的字符            |
| `dd`    | 删除光标所在的整行           |
| `[n]dd` | 从光标所在位置开始删除连续 $n$ 行 |
| `D`     | 删除光标至行尾             |
| `yy`    | 复制光标所在行             |
| `[n]yy` | 从光标所在行开始复制连续的 $n$ 行 |
| `p`     | 在光标处粘贴              |
| `u`     | 撤销上次操作              |

### 移动光标


| 命令       | 描述           |
| :------- | :----------- |
| `^`      | 光标移动到行首      |
| `$`      | 光标移动到行尾      |
| `ctrl+d` | 向下翻半页        |
| `ctrl+f` | 向下翻一页        |
| `ctrl+u` | 向上翻半页        |
| `ctrl+b` | 向上翻一页        |
| `gg`     | 光标定位到文档开头    |
| `G`      | 光标定位到文档末尾    |
| `H`      | 光标定位到当前页首    |
| `L`      | 光标定位到当前页尾    |
| `w`      | 光标向后移动一个单词   |
| `b`      | 光标向前移动一个单词   |
| `[n]+`   | 光标向后移动 $n$ 行 |
| `[n]`-   | 光标向前移动 $n$ 行 |

### 查找

命令模式输入 `/search-string` 可以全文查找指定的字符串。按回车后，光标定位到第一个 `search-string` 的开头位置
+ `n` 光标移动到下一个匹配位置
+ `N` 光标移动到上一个匹配位置

### 块操作

命令模式中的 `v` 命令，可以启用块选择。

命令模式 `ctrl+v` 命令，可以按列选择

## 末行模式

想要进入末行模式，首先 `vim` 必须先处于命令模式，然后按 `:` 进入末行模式

### 保存和退出

编辑完成之后，如果需要退出 `vim` 可以使用下面几个末行命令

> [!tip] 末行命令
> + `q` 退出，未修改
> + `q!` 强制退出，放弃修改内容
> + `w` 写入修改
> + `wq` 保存并退出
> + `wq!` 保持并强制退出
>  
>  这些命令必须在末行模式下
>  

### 替换

在末行模式中，可以执行替换操作。替换的末行命令为： `:start,end/[src]/[dst]/ig`

> [!tip]
> 
> `start` 起始行号
> 
> `end` 结束行号
> 
> `/[src]` 原始文本
> 
> `/[dst]` 目标文本
> 
> `/i` 忽略大小写，`/g` 全部替换

如果需要执行全文替换，末行命令为：`:1,$/[src]/[dst]/ig` 或者 `:%s/[src]/[dst]/ig`

### 以十六进制显示文件内容

末行命令 `:%!xxd` 以十六进制显示文件内容。`:%!xdd -r` 返回文本模式

