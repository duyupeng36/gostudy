# HTML


HTML(**H**yper**T**ext **M**arkup **L**anguage) 代表 **超文本标记语言**。它用于 Web 前端，并为网页提供结构

## HTML 骨架

```html
<!DOCTYPE html>  <!--定义文档类型为 HTML5-->

<html lang="zh_CN"> <!--起始标签：html-->
    <head> <!--头部：文档中，不需要展示数据-->
        <title>Title</title> <!-- 文档的标题：显示在浏览器的标签页-->
    </head>
    
    <body> <!--可视化部分：html 渲染在浏览器上的部分-->
        Hello World
    </body>
</html> <!--结束标签： /html-->
```

+ `<！DOCTYPE html>` 声明定义此文档是 HTML5 文档
+ `<html>` 元素是 HTML 页面的根元素
+ `<head>` 元素包含有关 HTML 页面的元信息
+ `<title>` 元素指定 HTML 页面的标题
+ `<body>` 元素定义文档的正文，是所有可见内容的容器，例如标题、段落、图像、超链接、表格、列表等

> [!tip] HTML 元素
> 
> HTML 元素由 `start` 标签、一些 `content` 和 `end` 标签定义
> 
>```html
><tagname> Content goes here... </tagname>
>```
>
>HTML **元素**是从 `start` 标签到 `end` 标签的所有内容
>

## 基本标签

### meta 标签

`meta` 标签定义文档的元素数据

```html
<!DOCTYPE html>  <!--定义文档类型为 HTML-->>

<html lang="zh_cn"> <!--起始标签：html-->
    <head> <!--头部：文档中，不需要展示数据-->
        <meta charset="UTF-8">  <!--文档元数据：指定字符集为 utf-8-->
        <meta name="description" content="网页描述的内容"> <!--文档元数据：定义文档的描述文本，方便搜索引擎搜索-->
        <title>Title</title> <!-- 文档的标题：显示在浏览器的标签页-->
    </head>
    
    <body> <!--可视化部分：html 渲染在浏览器上的部分-->
        Hello World
    </body>
</html> <!--结束标签： /html-->
```

`meta` 标签的属性

| 属性                                                 | 描述        |
| :------------------------------------------------- | :-------- |
| `charset`                                          | 定义文档字符集   |
| `name="description"` 和 `content="..."`             | 定义文档的描述信息 |
| `name="author"` 和 `content="username"`             | 定义文档的作者   |
| `name="keywords"` 和 `content="..."`                | 定义文档的关键字  |
| `name="viewport"` 和 `content="width=device-width"` | 定义视口宽度等   |

### 标题标签

HTML 提供了 $6$ 中标题(`head`)标签，分别是 `h1` 标签 \~ `h6` 标签

```html
<!DOCTYPE html>

<html lang="zh_cn">
    <head> 
        <meta charset="UTF-8">  
        <meta name="description" content="网页描述的内容"> 
        <title>Title</title>
    </head>
    
    <body> <!--可视化部分：html 渲染在浏览器上的部分-->
        <h1>h1: Hello World</h1>  <!--标签标题：h1 - h6 -->
        <h2>h2: Hello World</h2>
        <h3>h3: Hello World</h3>
        <h4>h4: Hello World</h4>
        <h5>h5: Hello World</h5>
        <h6>h6: Hello World</h6>
    </body>
</html>
```

### 段落标签

当我们需要组织一段文本时，就需要使用 `p` 标签(`paragraph`)

```html
<!DOCTYPE html>  <!--定义文档类型为 HTML-->

<html lang="zh_cn"> <!--起始标签：html-->
    <head> <!--头部：文档中，不需要展示数据-->
        <meta charset="UTF-8">  <!--文档元数据：指定字符集为 utf-8-->
        <meta name="description" content="网页描述的内容"> <!--文档元数据：定义文档的描述文本，方便搜索引擎搜索-->
        <title>Title</title> <!-- 文档的标题：显示在浏览器的标签页-->
    </head>
    
    <body> <!--可视化部分：html 渲染在浏览器上的部分-->
        <h1>Hello World</h1>  <!--标签标题：h1 - h6 -->
        <h2>Test Website</h2>
        <br/>  <!--换行-->
        <p>This <small>is</small> a <b>paragraph</b></p>
        <hr/> <!--水平线-->
        <p>This is <i>another</i> <b><i>paragraph</i></b></p>

        <p>
            10<sup>2</sup><!--上标：sup-->
            <br/>
            w<sub>2</sub> <!--下表：sub-->
        </p>  
        
    </body>
</html> <!--结束标签： /html-->
```

>[!tip] 文本样式标签
>+ `<b> text </b>` 加粗文本
>+ `<i> text </i>` 倾斜文本
>+ `<small> text </small>` 文本变小
>+ `<sup> </sup>` 上标
>+ `<sub> </sub>` 下标

### 换行与水平线

浏览器在解析 HTML 时，空格只会保留一个，换行被忽略。**浏览器只关注 HTML 元素**。因此，需要在 HTML 中换行，就需要使用 `br` 标签(`break`)

```html
<!DOCTYPE html>

<html lang="zh_cn"> 
    <head>
        <meta charset="UTF-8"> 
        <meta name="description" content="网页描述的内容">
        <title>Title</title>
    </head>
    
    <body> 
        <h1>Hello World</h1>  
        <h2>Test Website</h2>
        <br>  <!--换行-->
        <p>This is a <b>paragraph</b></p>
        <p>This is <i>another</i> <b><i>paragraph</i></b></p>
        
    </body>
</html> 
```

`hr` 标签(`horizontal`) 会绘制一条 *贯穿浏览器的水平线*

```html
<!DOCTYPE html>

<html lang="zh_cn">
    <head> 
        <meta charset="UTF-8">  
        <meta name="description" content="网页描述的内容"> 
        <title>Title</title>
    </head>
    
    <body> 
        <h1>Hello World</h1>  
        <h2>Test Website</h2>
        <br/>  <!--换行-->
        <p>This is a <b>paragraph</b></p>
        <hr/> <!--水平线-->
        <p>This is <i>another</i> <b><i>paragraph</i></b></p>
        
    </body>
</html>
```

> [!tip] `br` 和 `hr` 标签属于单标签

![[Pasted image 20240910221004.png|900]]

## 链接标签

**链接** 允许用户从一个页面单击到另一个页面。链接不必是文本。链接可以是图像或任何其他 HTML 元素！

HTML `<a>` 标签定义超链接。其属性 `href` 指定连接的地址

```html
<!DOCTYPE html>

<html lang="zh_cn"> 
    <head> 
        <meta charset="UTF-8"> 
        <meta name="description" content="网页描述的内容"> 
        <title>Title</title> 
    </head>
    
    <body> 
        <h2>链接</h2>
		<a href="https://www.baidu.com" target="_blank">百度</a> <!--超链接由 a 标签提供：属性 href 指定连接位置 -->
    </body>
</html>
```

> [!tip] 默认情况下，链接在所有浏览器中的显示方式如下
> + **未访问** 的链接带有 _下划线_ 和 _蓝色_
> + **已访问** 的链接带有 _下划线_ 和 _紫色_
> + **活动** 的链接带有 _下划线_ 和 _红色_

默认情况下，链接的页面将显示在当前浏览器窗口中。要更改此设置，必须为链接指定另一个目标。属性 `target` 指定打开链接的位置

| 属性值       | 描述                |
| :-------- | :---------------- |
| `_self`   | 默认，在当前窗口打开链接      |
| `_black`  | 在新窗口打开链接          |
| `_parent` | 在父 `iframe` 中打开链接 |
| `_top`    | 在窗口的整个正文中打开链接     |

## 图片标签

图片标签 `img` 用于在浏览器中展示一张图标，其属性 `src` 指定需要展示的图片资源地址

```html
<!DOCTYPE html>

<html lang="zh_cn"> 
    <head> 
        <meta charset="UTF-8"> 
        <meta name="description" content="网页描述的内容"> 
        <title>Title</title>
    </head>
    
    <body>
        <h2>Image</h2>
        <img src="https://k.sinaimg.cn/n/sinacn20116/614/w850h564/20190705/781f-hzmafvm4729243.jpg/w700d1q75cms.jpg" alt="小熊猫">
    </body>
</html> 

```

`img` 标签的 `alt` 属性用于指定图片加载失败显示的内容，对于使用屏幕阅读器的用户来说，指定 `alt` 属性可以帮助他们了解图片的内容

> [!tip] `img` 标签是一个单标签

## 列表标签

HTML 提供三种形式的列表 `<ul>` `<ol>` 和 `<dl>`

> [!tip] `<ul>` 和 `<ol>` 必须和 `li` 标签一起使用

```html
<!DOCTYPE html>

<html lang="zh_cn"> 
    <head> 
        <meta charset="UTF-8">  
        <meta name="description" content="网页描述的内容"> 
        <title>Title</title> 
    </head>
    
    <body> 
        <h2>列表</h2>
        <h3>无序列表</h3>
        <ul>
            <li>Apples</li>
            <li>Oranges</li>
            <li>Banans</li>
        </ul>

        <h3>有序列表</h3>
        <ol>
            <li>Apples</li>
            <li>Oranges</li>
            <li>Banans</li>
        </ol>

        <h3>描述列表</h3>
        <dl>
            <dt>Apples</dt> <!--列表值-->
            <dd>- They ary red</dd> <!--描述信息-->
            <dt>Oranges</dt>
            <dd>- They ary orange</dd>
            <dt>Banans</dt>
            <dd>- They ary yellow</dd>
        </dl>
    </body>
</html>
```

## 表格

在 HTML 中如果需要创建表格，需要使用 `table` 标签。表格是由 **行**(`row`) 和 **列**(`column`) 组成。HTML 中，行由标签 `tr`(table row) 定义，列则由标签 `td`(table data) 定义

```html
<!DOCTYPE html>  <!--定义文档类型为 HTML-->

<html lang="zh_cn"> 
    <head> 
        <meta charset="UTF-8"> 
        <meta name="description" content="网页描述的内容"> 
        <title>Title</title>
    </head>
    
    <body> 
        <h2>表格</h2>

        <table>
            <caption>表格标题: 数字列表</caption>
            <thead> <!--thead 定义表格表头-->
                <tr>
                    <th>num1</th>  <!--表头-->
                    <th>num2</th>
                    <th>num2</th>
                </tr> <!-- tr 定义表格的一行-->
            </thead>
            <tbody> <!--tbody 定义表格主体部分-->
                <tr>
                    <td>1</td> <!--表格数据：占据一列-->
                    <td>2</td>
                    <td>3</td>
                </tr>
                <tr>
                    <td>4</td>
                    <td>5</td>
                    <td>6</td>
                </tr>
                <tr>
                    <td>7</td>
                    <td>8</td>
                    <td>9</td>
                </tr>
            </tbody>
            <tfoot> <!--tfoot 定义表格每列的汇总-->
                <td>12</td>
                <td>15</td>
                <td>18</td>
            </tfoot>
        </table>


    </body>
</html>
```

## 容器标签

HTML 中最常用的两个容器标签为 `div` 和 `span` 标签，它们是常用于布局的 **无语标签**，区别在于 `div` 标签是 **块元素** 独占一行；而 `span` 标签是 **行内元素**，宽度和高度由内容内容撑开

> [!tip] 块元素和行内元素
> + 块元素：块元素独占一行，可以设置宽度和高度
> + 行内元素：行内元素的宽度和高度由其内容撑开，不能设置宽度和高度。

## 全局属性

HTML 由两个广泛使用的全局属性 `class` 和 `id`

| 属性      | 描述                      |
| :------ | :---------------------- |
| `class` | 定义一个标签类别。通常用于指定 CSS     |
| `id`    | 标签的全局唯一标识符。通常用于 JS 寻找元素 |

## 表单标签

HTML `<form>` 标签用于创建用于用户输入的 **表单**

```html
<form action="" method="post" target="_black">
.
form elements
.
</form>
```

下表列出了各种属性的含义

| 属性        | 描述                                                                              |
| :-------- | :------------------------------------------------------------------------------ |
| `action`  | 提交数据的目标地址，默认为当前页面地址                                                             |
| `method`  | 提交数据的方法                                                                         |
| `target`  | 指定响应的位置                                                                         |
| `enctype` | 指定提交数据的类型: `multipart/form-data` 提交文件；`application/x-www-form-urlencode` 提交普通表单 |

`<form>` 元素是不同类型输入元素的容器，例如：文本字段、复选框、单选按钮、提交按钮等

### input 标签

HTML `<input>` 标签是最常用的表单标签。`<input>` 元素可以通过多种方式显示，具体取决于属性 **type**

| Type                      | Description | Default Style                              |
| ------------------------- | ----------- | ------------------------------------------ |
| `<input type="text">`     | 单行文本输入字段    | <input type="text" placeholder="输入用户民">    |
| `<input type="password">` | 密码框         | <input type="password" placeholder="输入密码"> |
| `<input type="radio">`    | 单选按钮        | <input type="radio">                       |
| `<input type="checkbox">` | 复选框         | <input type="checkbox">                    |
| `<input type="submit">`   | 提交按钮        | <input type="submit" value="提交">           |
| `<input type="button">`   | 普通按钮        | <input type="button" value="按钮">           |
| `<input type="color">`    | 颜色输入        |                                            |
| `<input type="file">`     | 文件输入        |                                            |
| `<input type="hidden">`   | 隐藏域         |                                            |

```html
<!DOCTYPE html>

<html lang="zh_cn">
    <head>
        <meta charset="UTF-8"> 
        <meta name="description" content="网页描述的内容">
        <title>Title</title> 
    </head>
    
    <body> 
        <h2>form表单</h2>
        <form action="/" method="post">
            <label for="first">First Name</label> : <input type="text" id="first" name="firstName" placeholder="名字">
            <label for="last">Last Name</label> : <input type="text" id="last" name="lastName" placeholder="姓氏">
        </form>
    </body>
</html>
```

`input` 标签的重要属性

| 属性            | 描述                                   |
| :------------ | :----------------------------------- |
| `type`        | 定义 `input` 标签的样式：`text` `password` 等 |
| `name`        | 指定提交数据的名字。如果未指定，提交的数据就没有名字           |
| `placeholder` | 输入框的提示信息                             |
| `value`       | 指定 `input` 标签携带的值，这个值就是提交到后端的值       |
| `readonly`    | 只读：`value` 属性值无法更改，提交表单时会发送只读输入字段的值  |
| `disabled`    | 禁用：提交表单时不发送                          |
| `min`和`max`   | 指定输入字段的最小值和最大值，只有在特定 `type` 下有用      |
| `pattern`     | 校验的正则表达式                             |
| `required`    | 必须提供                                 |

> [!tip] 单选框和复选框通过 `name` 属性进行分组

### textarea 标签

`textarea` 标签是一个 **多行纯文本编辑控件**，适用于 **允许用户输入大量自由格式文本** 的场景

```html
<textarea name="content" id="content" rows="3" cols="30"></textarea>
```

属性 `rows` 和 `cols` 用于指定 `<textarea>` 的精确尺寸


### select 标签

`select` 标签用于定义下拉列表

```html
<label for="cars">Choose a car:</label>  
<select id="cars" name="cars">  
  <option value="volvo">Volvo</option>  
  <option value="saab">Saab</option>  
  <option value="fiat">Fiat</option>  
  <option value="audi">Audi</option>  
</select>
```

默认情况下，`select` 标签只显示一个选项。`size` 属性可以指定可见的选项数列

```html
<label for="cars">Choose a car:</label>
<select id="cars" name="cars" size="3">  <!-- size 属性指定可见选项的数量 -->
  <option value="volvo">Volvo</option>
  <option value="saab">Saab</option>
  <option value="fiat">Fiat</option>
  <option value="audi">Audi</option>
</select>
```

`select` 标签默认只允许选择一个选项，如果需要选择多个值，需要指定 `multiple` 属性

```html
<label for="cars">Choose a car:</label>
<select id="cars" name="cars" size="4" multiple> <!-- multiple 属性允许选中多个值 -->
  <option value="volvo">Volvo</option>
  <option value="saab">Saab</option>
  <option value="fiat">Fiat</option>
  <option value="audi">Audi</option>
</select>
```

`<option>`标签定义了一个 **选项**。默认情况下，选择下拉列表中的第一项。`selected`属性指定默认选择的选项

```html
<option value="fiat" selected>Fiat</option> <!-- selected 默认选中 -->
```

`optgrop` 标签用于给 `option` 标签分组

```html
<label for="telephone">Choose a telephone:</label>  
<select id="telephone" name="telephone">  
	<optgroup label="xiaomi">
		<option value="14">小米14</option>
		<option value="14">小米14</option>
		<option value="12">小米12</option>
	</optgroup>
	<optgroup label="huawei">
		<option value="60">mate60</option>
		<option value="pro">mate60pro</option>
	</optgroup>
</select>
```

### button 标签

`<button>` 元素定义了一个可点击的按钮

```html
<button type="button" onclick="alert('Hello World!')">Click Me!</button>
```

### fieldset 标签

**`<fieldset>`** 元素用于将 **相关数据分组** 到表单中。`<legend>`元素定义该 `<fieldset>` 元素的标题 

```html
<!DOCTYPE html>
<html>
<head>
    <!--给浏览器的标签上显示标题，也可以作为网页收藏的文字-->
    <title>form表单-fieldset</title>
</head>

<body>

    <div class="container">
        <h3>注册</h3>
        <form action="xxx" method="post" enctype="multipart/form-data">
            <fieldset>
                <legend>基本信息</legend>
                <!--简单的文本框-->
                <p><label>姓名<input/></label></p>
                <!--有类型和名字和id-->
                <p><label>密码<input type="text"  value="123456" maxlength="20"  placeholder="请输入账号"  name="username" id="username"/></label></p>
                <p>
                    <label>性别：</label>
                    <label><input type="radio" name="male" checked value="1">男</label>
                    <label><input type="radio" name="male" value="0">女</label>
                    <label><input type="radio" name="male" value="2">保密</label>
                </p>

            </fieldset>
            <!--第一段：基本信息-->
            
            <fieldset>
                <legend>个人兴趣和介绍</legend>
                <p>
                    <label>兴趣爱好：</label>
                    <label><input type="checkbox" name="hobbys" checked value="1">旅游</label>
                    <label><input type="checkbox" name="hobbys" value="2">电影</label>
                    <label><input type="checkbox" name="hobbys" checked value="3">游泳</label>
                    <label><input type="checkbox" name="hobbys" value="4">看书</label>
                </p>
                <p>
                    <label>
                    自我介绍：<textarea  style="resize:none;" name="intro"></textarea>
                </label>
                </p>
            </fieldset>
			<!--第二段：兴趣和爱好-->

            <fieldset>
                <legend>城市信息</legend>
                <p>
                    <label>
                    上传用户头像 <input type="file" name="file"/>
	                </label>
                </p>
                <p>
                    <label>来自城市：
	                    <select name="city" multiple>
	                        <option value="">--请选择城市--</option>
	                        <option value="1">广州</option>
	                        <option value="2">深圳</option>
	                        <option selected value="3">北京</option>
	                        <option value="4">湖南</option>
	                    </select>
	
	                    <select name="city">
	                        <option value="">--请选择城市--</option>
	                        <optgroup  label="华南地区">
	                            <option value="1">广州</option>
	                            <option value="2">深圳</option>
	                            <option value="4">湖南</option>
	                        </optgroup>
	                        
	                        <optgroup  label="华北地区">
	                            <option selected value="3">北京</option>
	                        </optgroup>
	                    </select>
		            </label>
                </p>
            </fieldset>
            <!--第三段：城市信息-->
            
            <fieldset>
                <legend>提交按钮</legend>
                <p>
                    <input type="submit" value="注册">
                    <input type="reset" value="重置">
                </p>
                <p>
                    <button type="submit">注册</button>
                    <button type="reset">重置</button>
                </p>
            </fieldset>
            <!--最后一段：提交-->

        </form>
        
    </div>
</body>

</html>
```

### datalist 标签

`<datalist>`标签指定 `<input>` 标签的 **预定义选项列表**。用户输入数据时将看到预定义选项的下拉列表。

`<input>` 元素的 `list` 属性，必须引用 `<datalist>` 元素`id`的属性

```html
<form action="/action_page.php">
  <input list="browsers"> <!--list 属性引用 datalist 的id 属性-->
  <datalist id="browsers">
    <option value="Edge">
    <option value="Firefox">
    <option value="Chrome">
    <option value="Opera">
    <option value="Safari">
  </datalist>
</form>
```
