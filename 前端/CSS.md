# CSS

CSS(**C**ascading **S**tyle **S**heets) 是一种 **样式表语言**，用来描述 [[HTML]] 或 XML（包括如 `SVG`、`MathML` 或 `XHTML` 之类的 `XML` 分支语言）文档的呈现方式。CSS 描述了在屏幕、纸质、音频等其他媒体上的元素应该如何被渲染的问题。

## 引入 CSS

我们最想做的就是让 HTML 文档能够遵守我们给它的 CSS 规则。其实有三种方式可以实现，而目前我们更倾向于利用最普遍且有用的方式——**在文档的开头链接 CSS**

> [!tip] 在 `head` 标签中使用 `link` 标签引入 CSS 文件

```html title:index.html
<!DOCTYPE html>
<html lang="zh">
  <head>
    <meta charset="utf-8" />
    <link rel="stylesheet" href="css/index.css">  <!--引入 CSS 文件-->
    <title>开始学习 CSS</title>
  </head>

  <body>
    <h1>我是一级标题</h1>

    <p>
      这是一个段落文本。在文本中有一个 <span>span element</span> 并且还有一个
      <a href="http://example.com">链接</a>.
    </p>

    <p>这是第二段。包含了一个 <em>强调</em> 元素。</p>

    <ul>
      <li>项目 1</li>
      <li>项目 2</li>
      <li>项目 <em>三</em></li>
    </ul>
  </body>
</html>
```

>[!tip] `link` 标签
>
>+ `rel` 属性指定引入文档类型
>+ `href` 属性指定 `.css` 文件的链接或，路径
>

```css title:css/index.css
h1 { /* h1 是选择器，通过标签名选择所有的 h1 标签*/
    color: red;  /* 设置：文本颜色为红色*/
}
```

将 `h1` 标签中文本颜色设置为红色后的页面的样式如下图

![[Pasted image 20240911130223.png]]

标签 `style` 可以嵌入在页面的 `head` 标签内部，并在 `style` 标签中书写 CSS 

```html
<!DOCTYPE html>
<html lang="zh">
  <head>
    <meta charset="utf-8" />
    <title>开始学习 CSS</title>
    <link rel="stylesheet" href="css/index.css">  <!--引入 CSS 文件-->
    <style> 
        h1 {
            color: green;
        }
    </style ><!--内部 CSS 样式-->
    
  </head>

  <body>
    <h1>我是一级标题</h1>
    <p>
      这是一个段落文本。在文本中有一个 <span>span element</span> 并且还有一个
      <a href="http://example.com">链接</a>.
    </p>

    <p>这是第二段。包含了一个 <em>强调</em> 元素。</p>

    <ul>
      <li>项目 1</li>
      <li>项目 2</li>
      <li>项目 <em>三</em></li>
    </ul>
  </body>
</html>
```

> [!attention] 注意：`style` 标签和 `link` 标签的顺序，如果 `link` 标签在 `style` 标签后面，则 `link` 引入的 CSS 样式生效
> + 浏览器从上到下解析，最后解析的 CSS 样式才会生效
> + 同一个元素的 CSS 样式，先加载的 CSS 会被后加载的样式覆盖

标签的 `style` 属性也可以设置 CSS 样式，这一定是标签最后加载的样式

```html
<!DOCTYPE html>
<html lang="zh">
  <head>
    <meta charset="utf-8" />
    <title>开始学习 CSS</title>
    <link rel="stylesheet" href="css/index.css">  <!--引入 CSS 文件-->
    <style> 
        h1 {
            color: green;
        }
    </style> <!--内部 CSS 样式-->
    
  </head>

  <body>
    <h1 style="color:pink;"> 我是一级标题 </h1> <!--内敛样式-->

    <p>
      这是一个段落文本。在文本中有一个 <span>span element</span> 并且还有一个
      <a href="http://example.com">链接</a>.
    </p>

    <p>这是第二段。包含了一个 <em>强调</em> 元素。</p>

    <ul>
      <li>项目 1</li>
      <li>项目 2</li>
      <li>项目 <em>三</em></li>
    </ul>
  </body>
</html>

```

## 选择器

### 标签 类和 ID 选择器

标签名字作为选择器时，该选择器选中页面上同名的所有标签。例如，

```css
h1 {
}
```

标签可以设置 `class` 属性，该属性的值作为选择器时，需要在前面加 `.`

```css
.box {
}
```

标签还可以设置 `id` 属性，该属性的值作为选择器时，需要在前面加 `#`

```css
#unique {
}
```

> [!attention]  注意：标签的 ID 一个页面中是唯一的

如果某些选择器的样式相同，可以使用 **选择器列表**

```css
h1, .box, #unique {
}
```


### 标签属性选择器


`[attr]` 匹配带有一个属性名为 `attr` 的元素，例如

```css
a[title] {
}
```

`[attr = value]` 匹配带有一个属性名为 `attr` 且值为 `value` 的元素

```css
a[href="https://example.com"]{
}
```

`[attr ~= value]` 匹配带有一个属性名为 `attr` 且值含有 `value` 的元素

```css
li[class~="a"] {
}
```

`[attr |= value]` 匹配带有一个属性名为 `attr` 且值为 `value` 或者开始为 `value` 的元素

```css
div[lang|="zh"] {
}
```

`[attr^=value]` 匹配带有一个属性名为 `attr` 且值以 `value` 开头的元素

```css
li[class^="box-"] {
}
```

`[attr$=value]`  匹配带有一个属性名为 `attr` 且值以 `value` 结尾的元素

```css
li[class$="-box"] {
}
```

`[attr*=value]` 匹配带有一个属性名为 `attr` 且值中出现了 `value` 的元素

```css
li[class*="box"] {
}
```

### 伪类和伪元素选择器

#### 伪类

**伪类** 是选择器的一种，它用于 **选择处于特定状态的元素**

> [!tip] 
> 当它们是这一类型的第一个元素时，或者是当鼠标指针悬浮在元素上面的时候

伪类就是开头为冒号的关键字

```css
:pseudo-class-name
```

几个简单的伪类

| 伪类             | 描述          |
| :------------- | :---------- |
| `:first-child` | 第一个子元素      |
| `:last-child`  | 最后一个子元素     |
| `:only-child`  | 没有任何兄弟元素的元素 |
| `:invalid`     | 未通过验证的表单元素  |

用户行为伪类

| 伪类         | 描述                                             |
| :--------- | :--------------------------------------------- |
| `:hover`   | 鼠标悬停：指针挪到元素上的时候才会激活                            |
| `:visited` | 访问过：仅适用于带有 `href` 属性的 `<a>` 和 `<area>` 元素      |
| `:active`  | 活动的：                                           |
| `:link`    | 未被访问：匹配每个具有 `href` 属性的未访问的 `<a>` 或 `<area>` 元素 |

完整的伪类列表

|选择器|描述|
|---|---|
|[`:active`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/:active)|在用户激活（例如点击）元素的时候匹配。|
|[`:any-link`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/:any-link)|匹配一个链接的`:link`和`:visited`状态。|
|[`:blank`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/:blank)|匹配空输入值的[`<input>`元素](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/input)。|
|[`:checked`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/:checked)|匹配处于选中状态的单选或者复选框。|
|[`:current`](https://developer.mozilla.org/en-US/docs/Web/CSS/:current "此页面目前仅提供英文版本")|匹配正在展示的元素，或者其上级元素。|
|[`:default`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/:default)|匹配一组相似的元素中默认的一个或者更多的 UI 元素。|
|[`:dir`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/:dir)|基于其方向性（HTML[`dir`](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Global_attributes/dir)属性或者 CSS[`direction`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/direction)属性的值）匹配一个元素。|
|[`:disabled`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/:disabled)|匹配处于关闭状态的用户界面元素|
|[`:empty`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/:empty)|匹配除了可能存在的空格外，没有子元素的元素。|
|[`:enabled`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/:enabled)|匹配处于开启状态的用户界面元素。|
|[`:first`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/:first)|匹配[分页媒体](https://developer.mozilla.org/zh-CN/docs/Web/CSS/CSS_paged_media)的第一页。|
|[`:first-child`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/:first-child)|匹配兄弟元素中的第一个元素。|
|[`:first-of-type`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/:first-of-type)|匹配兄弟元素中第一个某种类型的元素。|
|[`:focus`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/:focus)|当一个元素有焦点的时候匹配。|
|[`:focus-visible`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/:focus-visible)|当元素有焦点，且焦点对用户可见的时候匹配。|
|[`:focus-within`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/:focus-within)|匹配有焦点的元素，以及子代元素有焦点的元素。|
|[`:future`](https://developer.mozilla.org/en-US/docs/Web/CSS/:future "此页面目前仅提供英文版本")|匹配当前元素之后的元素。|
|[`:hover`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/:hover)|当用户悬浮到一个元素之上的时候匹配。|
|[`:indeterminate`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/:indeterminate)|匹配未定态值的 UI 元素，通常为[复选框](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/input/checkbox)。|
|[`:in-range`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/:in-range)|用一个区间匹配元素，当值处于区间之内时匹配。|
|[`:invalid`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/:invalid)|匹配诸如`<input>`的位于不可用状态的元素。|
|[`:lang`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/:lang)|基于语言（HTML[lang](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Global_attributes/lang)属性的值）匹配元素。|
|[`:last-child`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/:last-child)|匹配兄弟元素中最末的那个元素。|
|[`:last-of-type`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/:last-of-type)|匹配兄弟元素中最后一个某种类型的元素。|
|[`:left`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/:left)|在[分页媒体](https://developer.mozilla.org/zh-CN/docs/Web/CSS/CSS_paged_media)中，匹配左手边的页。|
|[`:link`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/:link)|匹配未曾访问的链接。|
|[`:local-link`](https://developer.mozilla.org/en-US/docs/Web/CSS/:local-link "此页面目前仅提供英文版本")|匹配指向和当前文档同一网站页面的链接。|
|[`:is()`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/:is)|匹配传入的选择器列表中的任何选择器。|
|[`:not`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/:not)|匹配作为值传入自身的选择器未匹配的物件。|
|[`:nth-child`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/:nth-child)|匹配一列兄弟元素中的元素——兄弟元素按照_an+b_形式的式子进行匹配（比如 2n+1 匹配元素 1、3、5、7 等。即所有的奇数个）。|
|[`:nth-of-type`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/:nth-of-type)|匹配某种类型的一列兄弟元素（比如，`<p>`元素）——兄弟元素按照_an+b_形式的式子进行匹配（比如 2n+1 匹配元素 1、3、5、7 等。即所有的奇数个）。|
|[`:nth-last-child`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/:nth-last-child)|匹配一列兄弟元素，从后往前倒数。兄弟元素按照_an+b_形式的式子进行匹配（比如 2n+1 匹配按照顺序来的最后一个元素，然后往前两个，再往前两个，诸如此类。从后往前数的所有奇数个）。|
|[`:nth-last-of-type`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/:nth-last-of-type)|匹配某种类型的一列兄弟元素（比如，`<p>`元素），从后往前倒数。兄弟元素按照_an+b_形式的式子进行匹配（比如 2n+1 匹配按照顺序来的最后一个元素，然后往前两个，再往前两个，诸如此类。从后往前数的所有奇数个）。|
|[`:only-child`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/:only-child)|匹配没有兄弟元素的元素。|
|[`:only-of-type`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/:only-of-type)|匹配兄弟元素中某类型仅有的元素。|
|[`:optional`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/:optional)|匹配不是必填的 form 元素。|
|[`:out-of-range`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/:out-of-range)|按区间匹配元素，当值不在区间内的时候匹配。|
|[`:past`](https://developer.mozilla.org/en-US/docs/Web/CSS/:past "此页面目前仅提供英文版本")|匹配当前元素之前的元素。|
|[`:placeholder-shown`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/:placeholder-shown)|匹配显示占位文字的 input 元素。|
|[`:playing`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/:playing)|匹配代表音频、视频或者相似的能“播放”或者“暂停”的资源的，且正在“播放”的元素。|
|[`:paused`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/:paused)|匹配代表音频、视频或者相似的能“播放”或者“暂停”的资源的，且正在“暂停”的元素。|
|[`:read-only`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/:read-only)|匹配用户不可更改的元素。|
|[`:read-write`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/:read-write)|匹配用户可更改的元素。|
|[`:required`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/:required)|匹配必填的 form 元素。|
|[`:right`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/:right)|在[分页媒体](https://developer.mozilla.org/zh-CN/docs/Web/CSS/CSS_paged_media)中，匹配右手边的页。|
|[`:root`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/:root)|匹配文档的根元素。|
|[`:scope`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/:scope)|匹配任何为参考点元素的元素。|
|[`:valid`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/:valid)|匹配诸如`<input>`元素的处于可用状态的元素。|
|[`:target`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/:target)|匹配当前 URL 目标的元素（例如如果它有一个匹配当前[URL 分段](https://en.wikipedia.org/wiki/Fragment_identifier)的元素）。|
|[`:visited`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/:visited)|匹配已访问链接。|

#### 伪元素

**伪元素** 以类似方式表现，不过表现得是 **像往标记文本中加入全新的 HTML 元素一样**，而不是向现有的元素上应用类。伪元素开头为双冒号 `::`。

```css
::pseudo-element-name
```

伪元素列表

| 选择器                                                                                     | 描述                         |
| --------------------------------------------------------------------------------------- | -------------------------- |
| [`::after`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/::after)                   | 匹配出现在原有元素的实际内容之后的一个可样式化元素。 |
| [`::before`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/::before)                 | 匹配出现在原有元素的实际内容之前的一个可样式化元素。 |
| [`::first-letter`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/::first-letter)     | 匹配元素的第一个字母。                |
| [`::first-line`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/::first-line)         | 匹配包含此伪元素的元素的第一行。           |
| [`::grammar-error`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/::grammar-error)   | 匹配文档中包含了浏览器标记的语法错误的那部分。    |
| [`::selection`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/::selection)           | 匹配文档中被选择的那部分。              |
| [`::spelling-error`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/::spelling-error) | 匹配文档中包含了浏览器标记的拼写错误的那部分。    |

> [!tip] 更多内容请参考 [伪类和伪元素](https://developer.mozilla.org/zh-CN/docs/Learn/CSS/Building_blocks/Selectors/Pseudo-classes_and_pseudo-elements)

### 关系选择器

后代选择器：`selector1 selector2{}`，选择 `selector1` 选中元素的内部元素 `selector2`

```css
body article p {
}
```


子代关系选择器：`selector1 > selector2 {}`，只选择 `selector1` 内部第一层的元素 `selector2`，更远的后代不会被选中

```css
article > p {
}
```

相邻兄弟选择器：`selector1 + selector2 {}`，选择与 `selector1` 同级且相邻的 `selector2` 元素

```css
p + img {
}
```

兄弟选择器：`selector1 ~ selector2{}`，选择与 `selector1` 同级的 `selector2`元素

```css
p ~ img {
}
```

示例

```css
ul > li[class="a"] {
}
```

## 层叠 优先级和继承

样式表 **层叠**——简单的说，就是 **CSS 规则的顺序很重要**；当应用两条同级别的规则到一个元素的时候，写在后面的就是实际使用的规则。

![[Pasted image 20240911165310.png|900]]

浏览器是根据 **优先级** 来决定当 **多个规则有不同选择器对应相同的元素** 的时候需要 **使用哪个规则**。它基本上是一个 **衡量选择器具体选择哪些区域的尺度**：

> [!tip] 优先级
> 
> + **标签选择器不是很具体**，则会选择页面上该类型的所有元素，所以它的 **优先级就会低一些**
> 
> + **类选择器稍微具体点**，则会选择该页面中有特定 `class` 属性值的元素，所以它的 **优先级就要高一点**

![[Pasted image 20240911165621.png|900]]

**继承** 需要在上下文中去理解——一些 **设置在父元素上的 CSS 属性是可以被子元素继承** 的，有些则不能

## 颜色

CSS 中提供了一些属性（例如 `color`、`background`）来设置 HTML 元素的颜色（例如元素的背景颜色或字体颜色），我们可以通过不同形式的值来指定颜色，如下表所示：

| 值      | 描述                                                                                         | 实例                               |
| ------ | ------------------------------------------------------------------------------------------ | -------------------------------- |
| 颜色名称   | 使用颜色名称来设置具体的颜色，比如 red、blue、brown、lightseagreen 等，颜色名称不区分大小写                                | `color: red;`                    |
| 十六进制码  | 使用十六进制码以 `#RRGGBB` 或 `#RGB`（比如 `#ff0000`）的形式设置具体颜色，`"#"` 后跟 `6` 位或 `3` 位十六进制字符(`0-9, A-F`) | `color: #f03;`                   |
| `RGB`  | 通过 `rgb()` 函数对 `red`、`green`、`blue` 三原色的强度进行控制，从而实现不同的颜色                                   | `color: rgb(155,0,51);`          |
| `RGBA` | `RGBA` 扩展了 `RGB`，在 `RGB` 的基础上增加了 `alpha` 通道来设置颜色的透明度，需要使用 `rgba()` 函数实现                    | `color: rgba(155,0,0,0.1);`      |
| `HSL`  | 通过 `hsl()` 函数对颜色的 **色调**、**饱和度**、**亮度** 进行调节，从而实现不同的颜色                                     | `color: hsl(120,100%,25%);`      |
| `HSLA` | `HSLA` 扩展了 `HSL`，在 `HSL` 的基础上增加了 `alpha` 通道来设置颜色的透明度，需要使用 `hsla() `函数实现                    | `color: hsla(240,100%,50%,0.5);` |
### 十六进制

十六进制码是指通过一个以 `#` 开头的 $6$ 位十六进制数(`0 ~ 9，a ~ f`) 表示颜色的方式，这个六位数可以分为三组，每组两位，依次表示 `red`、`green`、`blue` 三种颜色的强度，例如

```css
h1 {  color: #ffa500;}
p {   color: #00ff00;}
```

> [!NOTE] 提示
> 在使用十六进制码表示颜色时，如果每组的两个十六进制数是相同的，例如 `#00ff00`、`#ffffff`、`#aabbcc`，则可以将它们简写为 `#0f0`、`#fff`、`#abc`。

### RGB 和 RGBA

`RGB` 是 `red`、`green`、`blue` 的缩写，它是一种色彩模式，可以通过对 `red`、`green`、`blue` 三种颜色的控制来实现各式各样的颜色

![[Pasted image 20240426161912.png|900]]


`CSS` 中要使用 `RGB` 模式来设置颜色需要借助 **`rgb()` 函数**，函数的语法格式如下：

```css
rgb(red, green, blue);
/* red, green, blue 取值范围为 0 ~ 255 */
/* red, green, blue 取值还可以是百分比：0% ~ 100% */
```

```css
h1 { color: rgb(255, 165, 0);}
p { color: rgb(0%, 100%, 0%);}
```

`RGBA` 是 `RGB` 的扩展，在 `RGB` 的基础上又增加了对 `Alpha` 通道的控制，`Alpha` 通道可以设置颜色的透明度

### HSL 和 HSLA

`HSL` 代表 **色调**、**饱和度** 和 **亮度**。`HSLA` 颜色值是带有 `Alpha` 通道（不透明度）的 `HSL` 的扩展

在 HTML 中，可以使用色调、饱和度和亮度 (`HSL`) 来指定颜色，格式如下：

```css
hsl(hue, saturation, lightness);
/* hue: 色调，取值范围为 0~360；0 代表 red，120 代表 green，240 代表 blue*/
/* saturation：饱和度,是一个百分比值。0% 表示灰度，100% 表示全色*/
/* lightness：亮度，是一个百分比值。 0% 为黑色，100% 为白色*/
```

`HSLA` 颜色值是 `HSL` 颜色值的扩展，带有 `Alpha` 通道，用于指定颜色的不透明度。`HSLA` 颜色值通过以下方式指定

```css
hsla(hue, saturation, lightness, alpha)
```

> [!tip] 参考网站
> + [coolors](https://coolors.co/)

## 单位和尺寸

### 绝对单位

| 单位   | 名称     | 等价换算                       |
| ---- | ------ | -------------------------- |
| `cm` | 厘米     | `1cm = 37.8px = 25.2/64in` |
| `mm` | 毫米     | `1mm = 1/10th of 1cm`      |
| `Q`  | 四分之一毫米 | `1Q = 1/40th of 1cm`       |
| `in` | 英寸     | `1in = 2.54cm = 96px`      |
| `pc` | 派卡     | `1pc = 1/6th of 1in`       |
| `pt` | 点      | `1pt = 1/72th of 1in`      |
| `px` | 像素     | `1px = 1/96th of 1in`      |

### 相对单位

相对长度单位是相对于其他某些东西的

> [!tip] 
> + `em` 和 `rem` 分别相对于父元素和根元素的字体大小
> + `vh` 和 `vw` 分别相对于视口的高度和宽度
> + 百分比：相对于父元素

## 盒子模型

### 块盒子和行盒子

在 CSS 中，有 **块盒子** 和 **行盒子**。类型指的是 **盒子在页面流中的行为方式** 以及 **与页面上其他盒子的关系**。盒子有 **内部显示** 和 **外部显示** 两种类型

> [!tip] 外部显示
> 
> 一个拥有 `block` 外部显示类型的盒子会表现出以下行为
> + 盒子会产生换行
> + `width` 和 `height` 属性可以发挥作用
> + **内边距**、**外边距** 和 **边框** 会 **“推开”** 当前盒子周围其他元素
> + 如果未指定 `width`，方框将沿行向扩展，以填充其容器中的可用空间。在大多数情况下，盒子会变得与其容器一样宽，占据可用空间的 `100%`
> 
> 一个拥有 `inline` 外部显示类型的盒子会表现出以下行为
> + 盒子不会产生换行
> + `width` 和 `height` 属性将不起作用
> + **垂直方向** 的 _内边距_、_外边距_ 以及 _边框_ 会被应用但是 **不会** 把其他处于 `inline` 状态的盒子推开
> + **水平方向** 的 _内边距_、_外边距_ 以及 _边框_ 会被应用且 **会** 把其他处于 `inline` 状态的盒子推开
> 

> [!tip] 内部显示
> 
> 盒子还有 _内部_ 显示类型，它 **决定了盒子内元素的布局方式**
> 
> **块** 和 **行内** 布局是网络上的默认行为方式。默认情况下，在没有任何其他指令的情况下，方框内的元素也会以标准流的方式布局，并表现为区块或行内盒子
> 

**CSS 盒模型** 整体上适用于 **块盒子**，它定义了盒子的不同部分（**外边距(margin)**、**边框(border)**、**内边距(padding)** 和 **内容(content)** ）如何协同工作，以创建一个在页面上可以看到的盒子

> [!tip] **行内盒子** 使用的只是盒模型中定义的 _部分_ 行为。

下图显示了 CSS 盒模型的不同部分

![[Pasted image 20240911182644.png|900]]

+ `Margin`：元素之间的距离
+ `Border`：元素的边框
+ `Padding`：内容与边框的距离
+ `Content`：内容

### 标准盒模型

在标准盒模型中，如果在盒子上设置了 `inline-size` 和 `block-size`（或 `width` 和 `height`）属性值，这些值就定义了 _内容盒子_ 的 `inline-size` 和 `block-size`（水平语言中为 `width` 和 `height`）。然后将任何内边距和边框添加到这些尺寸中，以获得盒子所占的总大小

假设一个盒子的 CSS 如下

```css
.box {
  width: 350px;
  height: 150px;
  margin: 10px;
  padding: 25px;
  border: 5px solid black;
}
```

方框 _实际_ 占用的空间宽为 `410px`(`=width + 2 * padding + 2 * border-size =350 + 25 + 25 + 5 + 5`)，高为 `210px`(`=height + 2 * padding + 2 * boder-size = 150 + 25 + 25 + 5 + 5`)

![[Pasted image 20240911183032.png|900]]

> [!tip] 标准盒模型：`width` 和 `height` 代表的是 `content` 部分的宽和高 

### 替代盒模型

在替代盒模型中，**任何宽度都是页面上可见方框的宽度**。**内容区域的宽度是该宽度减去填充和边框的宽度**。无需将边框和内边距相加，即可获得盒子的实际大小

要为某个元素使用替代模型，可对其设置 `box-sizing: border-box`

```css
.box {
  box-sizing: border-box;
}
```

假设一个盒子的 CSS 为

```css
.box {
  width: 350px;
  inline-size: 350px;
  height: 150px;
  block-size: 150px;
  margin: 10px;
  padding: 25px;
  border: 5px solid black;
}
```

现在，盒子 _实际_ 占用的空间在行向为 `350px`，在纵向为 `150px`

![[Pasted image 20240911183534.png|900]]

> [!tip] 详细样式参考 [盒子模型-margin-padding-border](https://developer.mozilla.org/zh-CN/docs/Learn/CSS/Building_blocks/The_box_model#%E5%A4%96%E8%BE%B9%E8%B7%9D%E3%80%81%E5%86%85%E8%BE%B9%E8%B7%9D%E5%92%8C%E8%BE%B9%E6%A1%86l)

## 样式

### 文本样式

| 属性                | 示例                            | 描述      |
| :---------------- | ----------------------------- | :------ |
| `font-size`       | `font-size:20px;`             | 字体大小    |
| `font-family`     | `font-family: sans-serif;`    | 字体      |
| `font-weight`     | `font-weight:100,200,...,900` | 加粗      |
| `font-style`      | `font-style:italic;`          | 倾斜      |
|                   |                               |         |
| `color`           | `color:green;`                | 文本颜色    |
| `text-decoration` | `text-decoration: underline;` | 装饰线     |
| `text-transform`  | `text-transform:upper;`       | 文本大小写转换 |
| `text-align`      | `text-align:justify;`         | 文本对齐    |
| `text-indent`     | `text-indent:2em;`            | 首行缩进    |
| `line-height`     | `line-height:1.5;`            | 行高      |
| `letter-spacing`  | `letter-spacing:0.25em;`      | 字母之间的间距 |
| `word-spacing`    | `word-spacing:0.1em;`         | 单词之间的间距 |

> [!tip] 更多的文本样式参考 [text-\*](https://developer.mozilla.org/en-US/docs/Web/CSS/text-align) [font-\*](https://developer.mozilla.org/en-US/docs/Web/CSS/font)

### 链接样式

`a` 标签有一些默认的样式

+ 下划线
+ 未访问的链接显示为蓝色
+ 访问过的链接显示为紫色
+ 活动的链接显示为红色
+ 鼠标悬停时，显示小手指

如果需要去掉下划线

```css
a {
	text-decoration:none; /* 去除下划线 */
	cursor: pointer; /* 鼠标样式 */  
	color: blue;
}
```

设置链接样式需要使用四个 **伪类**

```css
/* 未访问链接的样式 */ 
a:link {
	color: blue;
}

/*访问过的链接*/
a:visited {
	color:purple;
}

/* 鼠标悬停 */
a:hover {
	color:dodgerblue;
}

/* 鼠标点击尚未松开时：活动链接*/ 
a:active {
	color: red;
}
```

### 列表样式

| 属性                    | 示例                                    | 描述          |
| :-------------------- | ------------------------------------- | ----------- |
| `list-style-type`     | `list-style-type:circle,upper-roman;` | `marker` 类型 |
| `list-style-position` | `list-style-position:outside;`        | `marker` 位置 |
| `list-style-image`    | `list-style-image:url(path);`         | 图片 `marker` |
| `list-style`          | `list-style:type image position;`     |             |

伪元素 `::marker` 可以单独设置列表 `marker` 的样式

```css
::marker {
	color: black;
}
```


## 练习

完成下面的样式

![[Pasted image 20240911205817.png]]

```html title:index.html
<!DOCTYPE html>
<html lang="zh">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>CSS Menu</title>
    <link rel="stylesheet" href="css/index.css">
</head>
<body>
    
    <nav>
        <h2>CSS Menu</h2>
        <ul>
            <li>
                <a href="one.html">Appetizers</a>
            </li>
            <li>
                <a href="one.html">Entrees</a>
            </li>
            <li>
                <a href="two.html">Desserts</a>
            </li>
            <li>
                <a href="two.html">Beverages</a>
            </li>
            <li>
                <a href="two.html">About</a>
            </li>
        </ul>  
    </nav>

    <p>
        Lear more about <a href="">Amazing Foods</a>!
    </p>

</body>
</html>
```

```css title:css/index.css

* {
    /* CSS 重置 */
    margin: 0;
    padding: 0;
    box-sizing: border-box;
}

body {
    margin: 0.5rem;
    font-family: 'Times New Roman', Times, serif;
    text-align: center;
}

nav {
    border: 2px solid #333;
    border-radius: 2rem;
    margin: 0 auto 1rem;
    max-width: 600px;
    font-size: 3rem;
    line-height: 7rem;
}

h2 {
    padding: 1rem;
    background-color: gold;
    border-radius: 2rem 2rem 0 0;  /* 左上 右上 左下 右下 */
}

ul {
    list-style-type: none;
}

li {
    border-top: 1px solid #333;
}

li a {
    display: block;
}


li a, li a:visited {
    text-decoration: none;
    color: #333;
}

li a:hover, li a:focus {
    background-color: #333;
    color: whitesmoke;
    cursor: pointer;
}

li:last-child a {
    border-radius: 0 0 2rem 2rem;
}
```

## 布局

### desplay 属性


| 值              | 描述       |
| :------------- | :------- |
| `inline`       | 设置为行盒子   |
| `block`        | 设置为块盒子   |
| `inline-block` | 设置为行内块盒子 |
| `none`         | 隐藏元素     |

### 浮动

设置了 `float` 属性的元素会脱离 **文档流**，从而漂浮起来。`float`属性可以具有以下值之一：

+ `left`：元素浮动到其容器的左侧
+ `right`：元素浮动到其容器的右侧
+ `none`：元素不浮动

`float`属性最简单的用途就是用于将 **文本环绕图像**

![[Pasted image 20240911220624.png|900]]

```html 
<!DOCTYPE html>
<html lang="zh">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>CSS Menu</title>
    <link rel="stylesheet" href="css/index.css">
</head>

<body>

    <section class="clearfix">
        <div class="left">Float Left</div>
        <p>This CSS Full Course for Beginners is an all-in-one beginner tutorial and
            complete course full of over 11 hours of CSS code and instruction to level
            up your web development skills. This course teaches CSS3. Think of this CSS full course tutorial as a CSS3
            video
            textbook with 24 clearly defined chapters.
        </p>
    </section>

    <p>
        This CSS Full Course for Beginners is an all-in-one beginner tutorial and
        complete course full of over 11 hours of CSS code and instruction to level
        up your web development skills. This course teaches CSS3. Think of this CSS full course tutorial as a CSS3 video
        textbook with 24 clearly defined chapters.
    </p>
    <p>This CSS Full Course for Beginners is an all-in-one beginner tutorial and
        complete course full of over 11 hours of CSS code and instruction to level
        up your web development skills. This course teaches CSS3. Think of this CSS full course tutorial as a CSS3 video
        textbook with 24 clearly defined chapters.
    </p>

    <div class="clearfix">
        <div class="right">Float Right</div>

        <p>This CSS Full Course for Beginners is an all-in-one beginner tutorial and
            complete course full of over 11 hours of CSS code and instruction to level
            up your web development skills. This course teaches CSS3. Think of this CSS full course tutorial as a CSS3
            video
            textbook with 24 clearly defined chapters.
        </p>
    </div>
    
    <p>This CSS Full Course for Beginners is an all-in-one beginner tutorial and
        complete course full of over 11 hours of CSS code and instruction to level
        up your web development skills. This course teaches CSS3. Think of this CSS full course tutorial as a CSS3
        video
        textbook with 24 clearly defined chapters.
    </p>
    <p>This CSS Full Course for Beginners is an all-in-one beginner tutorial and
        complete course full of over 11 hours of CSS code and instruction to level
        up your web development skills. This course teaches CSS3. Think of this CSS full course tutorial as a CSS3
        video
        textbook with 24 clearly defined chapters.
    </p>

</body>

</html>
```


```css
body {
    width: 900px;
    margin: 0 auto;
}

.left, .right {
    width: 200px;
    height: 200px;

    text-align: center;
    line-height: 200px;

    background-color: #333;
    color: whitesmoke;

    margin-right: 10px;
}

.left {
    float: left;
 
}

.right {
    float: right;
}


section {
    border: 1px solid rebeccapurple;
    /* 解决子元素浮动导致父元素塌陷的问题: display: flow-root;  */
}

/* 解决子元素浮动导致父元素塌陷的问题：在父元素上添加类属性 clearfix  */
.clearfix::after{
    display: table;
    content: "";
    clear: both;
}
```

### 列

`column-*` 可以将父元素分为多列

| 属性             | 示例                           | 描述       |
| :------------- | :--------------------------- | :------- |
| `column-count` | `column-count: 4;`           | 定义最大的列数  |
| `column-width` | `column-width:250px;`        | 定义每列的宽度  |
| `column-rule`  | `column-rule:1px solid #333` | 定义每列的分割线 |
| `column-gap`   | `column-gap:3rem;`           | 定义每列的间距  |
| `column`       | `column:count width;`        | 定义列布局样式  |

### 定位

定位涉及下面几个属性

| 属性         | 描述           |
| :--------- | :----------- |
| `position` | 指定定位方式       |
| `top`      | 元素距离参考元素的上边界 |
| `bottom`   | 元素距离参考元素的下边界 |
| `right`    | 元素距离参考元素的右边界 |
| `left`     | 元素距离参考元素的左边界 |

`position` 属性有下面 $5$ 种取值

| `position` | 描述                                                                               |
| :--------- | :------------------------------------------------------------------------------- |
| `static`   | 静态定位，默认值                                                                         |
| `relative` | 相对于自身原始位置                                                                        |
| `absloute` | 相对于父元素进行定位，起始顶点为父元素的左顶点。父元素必须是非 `static` 的，否则向上寻找。如果没找到，则相对于浏览器的左顶点              |
| `fixed`    | 相对浏览器的左顶点的定位属性                                                                   |
| `sticky`   | 它是一种 `relative` 和 `fixed` 的结合体，默认是 `relative`,当`top/left`达到一个临界值的时候就会转换成 `fixed` |
#### 相对定位

开始时的网页结构

![[Pasted image 20240911235418.png]]

该结构的 HTML 和 CSS 如下
```html title:index.html
<!DOCTYPE html>
<html lang="zh">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>CSS Menu</title>
    <link rel="stylesheet" href="css/index.css">
</head>

<body>

    <div class="first">first</div>
    <div class="second">second</div>
    <div class="third">third</div>

</body>

</html>
```

```css title:css/index.css
.first, .second, .third {
    width: 200px;
    height: 200px;
    font-size: 32px;
    text-align: center;
    line-height: 200px;
}

.first {
    background-color: hsl(65, 75%, 64%);
    color: rgb(10, 9, 9);
}

.second {
    background-color: hsl(193, 70%, 60%);
    color: rgb(10, 9, 9);
}

.third {
    background-color: hsl(281, 73%, 59%);
    color: rgb(10, 9, 9);
}
```

我们对 `.second` 元素进行相对自身的定位

```css
.second {
    background-color: hsl(193, 70%, 60%);
    color: rgb(10, 9, 9);

    /*相对定位：相对自身*/
    position: relative;
    left: 200px;
}
```

![[Pasted image 20240912095129.png]]

#### 绝对定位

绝对定位 _相对于指定非 `static` 的祖先元素进行偏移定位_，如果没有找到就相对于浏览器的可视区进行偏移定位

> [!tip] 绝对定位会让元素脱离文档流

```html title:index.html
<!DOCTYPE html>
<html lang="zh">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>CSS Menu</title>
    <link rel="stylesheet" href="css/index.css">
</head>

<body>

    <div class="outer">
        <div class="inner-first">first</div>
        <div class="inner-second">second</div>
    </div>

</body>

</html>
```

```css title:css/index.css
.outer {
    width: 400px;
    height: 400px;
    margin: 0 auto;
    background-color: burlywood;
}

.inner-first, .inner-second {
    width: 150px;
    height: 150px;
    
}

.inner-first {
    background-color: antiquewhite;
}

.inner-second {
    background-color:azure;
}
```

上面代码的样式为：

![[Pasted image 20240912095911.png]]


现在，使用绝对定位将 `.inner-first` 元素向右移动 `150px`

```css
.outer {
    width: 400px;
    height: 400px;
    margin: 0 auto;
    background-color: burlywood;
    position: relative; /* 父元素的定位方式为非 static */
}

.inner-first {
    background-color: antiquewhite;

    /* 子元素绝对定位：左边距离父元素左顶点 150px*/
    position: absolute;
    left: 150px;
}
```

修改后，`.inner-first` 元素脱离文档流，`.inner-second` 元素顶替了 `.inner-first` 原来的位置

![[Pasted image 20240912100103.png]]

#### 固定定位

固定定位 **相对于浏览器可视区域的左顶点** 进行定位，它会把元素固定在浏览器的可视区域，并且使得元素脱离文档流

```html title:index.html
<!DOCTYPE html>
<html lang="zh">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>CSS Menu</title>
    <link rel="stylesheet" href="css/index.css">
</head>

<body>

    <div class="outer">
        <div class="inner-first">first</div>
        <div class="inner-second">second</div>
    </div>

</body>

</html>
```

```css title:css/index.css
.outer {
    width: 400px;
    height: 2000px;
    margin: 0 auto;
    background-color: burlywood;
}

.inner-first, .inner-second {
    width: 150px;
    height: 150px;
    
}

.inner-first {
    background-color: antiquewhite;

    /* 子元素固定定位：固定在左边距离浏览器左顶点10px 位置*/
    position: fixed;
    left: 10px;
}

.inner-second {
    background-color:azure;
}

```

![[fixed-position.gif]]

#### 粘性定位

粘性定位可以被认为是 **相对定位和固定定位的混合**。元素在跨越特定的阈值前为相对定位 `relative`，之后为 `fixed` 固定定位

```html title:index.html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>相对定位relative</title>
    <style>

        body{
            height: 2000px;
        }

        .box1 {
            height: 30px;
            border:1px solid #000;
            position: sticky;
            top:10px
        }

    </style>
</head>

<body>
    <p>aaaaaaaaaaaaaaaaaaaaaa</p>
    <p>aaaaaaaaaaaaaaaaaaaaaa</p>
    <p>aaaaaaaaaaaaaaaaaaaaaa</p>
    <p>aaaaaaaaaaaaaaaaaaaaaa</p>
    <p>aaaaaaaaaaaaaaaaaaaaaa</p>
    <div class="box1">我是一个盒子</div>
    <p>aaaaaaaaaaaaaaaaaaaaaa</p>
    <p>aaaaaaaaaaaaaaaaaaaaaa</p>
    <p>aaaaaaaaaaaaaaaaaaaaaa</p>
    <p>aaaaaaaaaaaaaaaaaaaaaa</p>
    <p>aaaaaaaaaaaaaaaaaaaaaa</p>
    <p>aaaaaaaaaaaaaaaaaaaaaa</p>
    <p>aaaaaaaaaaaaaaaaaaaaaa</p>
    <p>aaaaaaaaaaaaaaaaaaaaaa</p>
</body>

</html>
```

![[sticky-position.gif]]

### 弹性

`Flexbox` 弹性盒子是一种用于 **按照行** 和 **按照列** 布局元素的 **一维布局方式**。元素可以膨胀以填充额外的空间，收缩以使用更小的空间

>[!tip]
>
> 如果 **父容器的宽度固定或者高度固定**，用弹性布局就是非常绝佳的选择
> 
>
>
>

如果需要使用弹性盒子，需要指定  `desplay:flex;`

#### main axis

![[Pasted image 20240510154320.png]]

`flex` 是去控制父元素的样式，子元素会根据父元素的样式自动排列

> [!tip] 
> 
> `main axis` 指定子元素的排序方向，默认的 `main axis` 是水平方向
> 

> [!attention] `flex` 样式必须应用在父元素上

属性 `flex-direction` 用于设置主轴

| `flex-direction` | 描述          |
| :--------------- | :---------- |
| `row`            | 默认，按行从左到右排列 |
| `row-reverse`    | 按行从右到左排列    |
| `column`         | 按列从上到下排列    |
| `column-reverse` | 按列从下到上排列    |

```html title:index.html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>CSS Learn</title>
    <link rel="stylesheet" href="css/index.css">
</head>

<body>
    <div class="container">
        <div>1</div>
        <div>2</div>
        <div>3</div>
    </div>
</body>
</html>
```


```css title:css/index.css
.container {
    width: 700px;
    height: 200px;
    background-color: black;
}

.container div {
    width: 200px;
    height: 200px;
    background-color: lightgreen;
}
```

![[Pasted image 20240912111216.png]]

现在，我们期望内部元素水平方向排列。使用弹性盒子非常容易可以实现

```css title:css/index.css
.container {
    width: 700px;
    height: 200px;
    background-color: black;
    /*只需要给父元素设置 display:flex; 属性即可完成*/
    display: flex;
}
```

![[Pasted image 20240912111350.png]]

如果想要反向，则可以使用 `flex-direction` 设置即可

```css title:css/index.css
.container {
    width: 700px;
    height: 200px;
    background-color: black;
    /*只需要给父元素设置 display:flex; 属性即可完成*/
    display: flex;
    flex-direction: row-reverse;
}
```

![[Pasted image 20240912111656.png]]

#### 主轴对齐方式

属性 `justify-content` 可以设置 **主轴的对齐方式**

| `justify-content` | 描述           |
| :---------------- | :----------- |
| `flex-start`      | 主轴开始位置对齐，默认的 |
| `flex-end`        | 主轴结束位置对齐     |
| `center`          | 居中对齐         |
| `space-around`    | 环绕对齐         |
| `space-between`   | 两端对齐         |
| `space-evenly`    | 间隙平均         |

##### flex-end

`flex-end` 按照主轴结束位置对齐

![[Pasted image 20240912112516.png]]
```css title:css/index.css
.container {
    width: 700px;
    height: 200px;
    background-color: black;

    display: flex;
    flex-direction: row;
    justify-content: flex-end; /* 主轴结束位置对齐 */
}
```

##### center

`center` 所有子元素居中排列

![[Pasted image 20240912112700.png]]

```css title:css/index.css
.container {
    width: 700px;
    height: 200px;
    background-color: black;

    display: flex;
    flex-direction: row;
    justify-content: center;
}
```

##### space-around

`space-around` 环绕：两端间隙相等，中间间隙均分

![[Pasted image 20240912112852.png]]

```css  title:css/index.css
.container {
    width: 700px;
    height: 200px;
    margin: auto; 
    background-color: black;

    display: flex;
    flex-direction: row;
    justify-content: space-around;
}
```

##### space-between

`space-between` 双端对齐：两端无间隙，中间间隙均分

![[Pasted image 20240912113100.png]]

```css title:css/index.css
.container {
    width: 700px;
    height: 200px;
    margin: auto;
    background-color: black;
    
    display: flex;
    flex-direction: row;
    justify-content: space-between;
}
```

##### space-evenly

`space-evenly` 所有间隙均分

![[Pasted image 20240912113206.png]]

```css title:css/index.css
.container {
    width: 700px;
    height: 200px;
    margin: auto;
    background-color: black;
    
    display: flex;
    flex-direction: row;
    justify-content: space-evenly;
}
```

#### 包围

默认情形下，弹性盒子的子元素值按照主轴排列。如果子元素的宽度之和超过主轴，则会对子元素进行压缩

`flex-wrap` 属性可以可以设置子元素的包围方式

| `flex-wrap`    | 描述     |
| :------------- | :----- |
| `nowrap`       | 不换行，默认 |
| `wrap`         | 换行     |
| `wrap-reverse` | 反向     |
##### nowrap

```html title:index.html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>CSS Learn</title>
    <link rel="stylesheet" href="css/index.css">
</head>

<body>
    <div class="container">

        <div>1</div>
        <div>2</div>
        <div>3</div>
        <div>4</div>
        <div>5</div>
        <div>6</div>
    </div>
</body>

</html>
```

```css title:css/index.cee
.container {
    width: 700px;
    height: 500px;

    margin: auto;

    background-color: black;
    /*只需要给父元素设置 display:flex; 属性即可完成*/
    display: flex;
    flex-direction: row;
    justify-content: space-between;
    flex-wrap: nowrap;
}
```

![[Pasted image 20240912114553.png]]

##### wrap

如果子元素的宽度之后大于弹性盒子的宽度，可以让其自动换行

![[Pasted image 20240912114824.png]]
```css title:css/index.css
.container {
    width: 700px;
    height: 500px;

    margin: auto;

    background-color: black;
    /*只需要给父元素设置 display:flex; 属性即可完成*/
    display: flex;
    flex-direction: row;
    justify-content: space-between;
    flex-wrap: wrap;
}
```

#### 交叉轴的对齐方式

`align-content` 和 `align-items` 设置交叉轴的上元素的对齐方式

> [!attention] 
> 
> 当不折行的情况下：`align-content` 是不生效的
> 
>`align-items` 不要求折行
> 

| `align-content/align-items` | 描述                         |
| :-------------------------- | :------------------------- |
| `stretch`                   | 默认，子元素未指定 `height`，就自动全部填充 |
| `flex-start`                | 开始位置对齐                     |
| `flex-end`                  | 结束位置对齐                     |
| `center`                    | 居中对齐                       |
| `space-around`              | 环绕对齐                       |
| `space-between`             | 两端对齐                       |
| `space-evenly`              | 间隙平均                       |

##### flex-start/flex-end

```css
.container {
    width: 700px;
    height: 500px;

    margin: auto;

    background-color: black;
    /*只需要给父元素设置 display:flex; 属性即可完成*/
    display: flex;
    flex-direction: row;
    flex-wrap: wrap;
    justify-content: space-between;
    align-content: flex-start; /* 交叉轴开始处对齐 */   
}
```

![[Pasted image 20240912115353.png]]

```css
.container {
    width: 700px;
    height: 500px;

    margin: auto;

    background-color: black;
    /*只需要给父元素设置 display:flex; 属性即可完成*/
    display: flex;
    flex-direction: row;
    flex-wrap: wrap;
    justify-content: space-between;
    align-content: flex-end; /* 交叉轴结束处对齐 */
}
```

![[Pasted image 20240912115502.png]]

##### space-around

```css
.container {
    width: 700px;
    height: 500px;

    margin: auto;

    background-color: black;
    /*只需要给父元素设置 display:flex; 属性即可完成*/
    display: flex;
    flex-direction: row;
    flex-wrap: wrap;
    justify-content: space-between;
    align-content: space-around; /* space-around：上下相等，中间均分 */
}
```

![[Pasted image 20240912115636.png]]

##### space-between

```css
.container {
    width: 700px;
    height: 500px;

    margin: auto;

    background-color: black;
    /*只需要给父元素设置 display:flex; 属性即可完成*/
    display: flex;
    flex-direction: row;
    flex-wrap: wrap;
    justify-content: space-between;
    align-content: space-between; /*  space-between：两端对齐*/
}
```

![[Pasted image 20240912115749.png]]

##### space-evenly

```css
.container {
    width: 700px;
    height: 500px;

    margin: auto;

    background-color: black;
    /*只需要给父元素设置 display:flex; 属性即可完成*/
    display: flex;
    flex-direction: row;
    flex-wrap: wrap;
    justify-content: space-between;
    align-content: space-evenly; /* space-evenly: 上下间隙均分 */
}
```

![[Pasted image 20240912115932.png]]

#### 控制子元素

##### order

控制子元素的位置

```css
.box div:nth-child(4) {
	order:-1; 
}
```

##### align-self

控制子元素自身的交叉轴的对齐方式

| `align-self`    | 描述                         |
| :-------------- | :------------------------- |
| `flex-start`    | 开始位置对齐                     |
| `flex-end`      | 结束位置对齐                     |
| `center`        | 居中对齐                       |
| `space-around`  | 环绕对齐                       |
| `space-between` | 两端对齐                       |
| `space-evenly`  | 间隙平均                       |

### 网格

**网格** 是一个用于 Web 的二维布局系统，利用网格，我们可以把内容按照 **行** 和 **列** 的格式进行排版。另外，网格还能非常轻松地实现一些复杂的布局。

> [!tip] 和弹性盒子一样，网格属性也是设置在父元素上的
> 

#### 定义网格

要定义一个网格，首先需要设置 `display:grid;`

属性 `grid-template-rows` 和 `grid-template-columns` 用于定义网格的行和列。属性值的个数表示行数和列数

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>CSS Learn</title>
    <link rel="stylesheet" href="css/index.css">
    <link rel="stylesheet" href="css/normalize.css">
</head>

<body>
    <div class="container">

        <div>1</div>
        <div>2</div>
        <div>3</div>
        <div>4</div>
        <div>5</div>
        <div>6</div>
        <div>7</div>
        <div>8</div>
        <div>9</div>
    </div>
</body>

</html>
```

```css
.container {
    width: 800px;
    height: 800px;
    margin: auto;

    background-color: lightgray;

    display: grid;

    grid-template-rows: 1fr 1fr 1fr;
    grid-template-columns: 1fr 1fr 1fr;
}

.container div {
    width: 200px;
    height: 200px;
    background-color: lightgreen;
}
```

![[Pasted image 20240912141409.png]]



##### 网格命名和网格合并

属性 `grid-template-areas` 用于定义单元格的合并规则。相同的名字表示合并

属性 `grid-area` 用于子元素，给子元素命名

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>CSS Learn</title>
    <link rel="stylesheet" href="css/index.css">
    <link rel="stylesheet" href="css/normalize.css">
</head>

<body>
    <div class="container">

        <div>1</div>
        <div>2</div>
        <div>3</div>
        <div>4</div>
    </div>
</body>

</html>
```

```css
.container {
    width: 600px;
    height: 600px;
    margin: auto;

    background-color: lightgray;

    display: grid;
    grid-area: 2px;
    grid-template-rows: 1fr 1fr 1fr;
    grid-template-columns: 1fr 1fr 1fr;

    grid-template-areas: 
        "t1 t1 t2" 
        "t1 t1 t2"
        "t3 t3 t4";

}

.container div {
    width: auto;
    height: auto;
}

.container div:nth-child(1) {
    grid-area: t1;
    background-color: lightgreen;
}

.container div:nth-child(2) {
    grid-area: t2;
    background-color: lightcoral;
}

.container div:nth-child(3) {
    grid-area: t3;
    background-color: lightyellow;
}

.container div:nth-child(4) {
    grid-area: t4;
    background-color: lightblue;
}

```

![[Pasted image 20240912142538.png]]

##### 间隙

属性 `grap`  用于定义单元格之间的间隙

```css
grap: row column;
```

如果需要单独设置行和列的间隙，使用 `row-grap` 和 `column-grap` 属性

#### 网格元素的对齐方式

控制网格中的元素的对齐方式的属性有

| 属性              | 描述                                   |
| :-------------- | :----------------------------------- |
| `justify-items` | 主轴对齐方式                               |
| `align-items`   | 交叉轴对齐方式                              |
| `place-items`   | `justify-items` 和 `align-items` 复合写法 |

>[!ttip] 
>这些属性是 **对每个网格的元素** 进行对齐，粒度更细

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>CSS Learn</title>
    <link rel="stylesheet" href="css/index.css">
    <link rel="stylesheet" href="css/normalize.css">
</head>

<body>
    <div class="container">

        <div>1</div>
        <div>2</div>
        <div>3</div>
        <div>4</div>
        <div>5</div>
        <div>6</div>
        <div>7</div>
        <div>8</div>
        <div>9</div>
    </div>
</body>

</html>
```

```css
.container {
    width: 600px;
    height: 600px;
    margin: auto;

    background-color: lightgray;

    display: grid;
    grid-area: 2px;
    grid-template-rows: 1fr 1fr 1fr;
    grid-template-columns: 1fr 1fr 1fr;


}

.container div {
    width: 150px;
    height: 150px;
    background-color: violet;
    border: 1px solid #000;
}
```


##### justify-items

`justify-items` 属性的取值如下表

| `justify-items` | 描述                |
| :-------------- | :---------------- |
| `start`         | 对齐到每个网格主轴的开始位置，默认 |
| `end`           | 对齐到每个网格主轴的结束为     |
| `center`        | 对齐到每个网格主轴的的居中位置   |

##### align-items

`align-items` 属性取值如下表

| `align-items` | 描述                 |
| :------------ | :----------------- |
| `start`       | 对齐到每个网格交叉轴的开始位置，默认 |
| `end`         | 对齐到每个网格交叉轴的结束为     |
| `center`      | 对齐到每个网格交叉轴的的居中位置   |

#### 网格整体的对齐方式

所有网格元素整体在容器中进行对齐。使用到的属性如下

| 属性                | 描述                                        |
| :---------------- | :---------------------------------------- |
| `justify-content` | 主轴的对齐方式                                   |
| `align-content`   | 交叉轴的对齐方式                                  |
| `place-content`   | `justify-content` 和 `align-content` 的复合写法 |

这些属性的取值和 `justify-items` 与 `align-items` 相同

#### 网格自适应

在定义网格时，可以使用 `repeat()` 函数，定义每个网格站的宽度

> [!tip]  `repeat(auto-fill, 固定值)` 自动分割固定值
> 

> [!tip] `minmax()` 设置变化的最小值

使用 `grid-auto-flow` 设置网格溢出的操作

| `grid-auto-flow` | 描述        |
| :--------------- | --------- |
| `row`            | 网格溢出时，添加行 |
| `column`         | 网格溢出时，添加列 |

`grid-auto-rows` 设置下一行的位置；`grid-auto-columns` 下一列出现的位置

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>CSS Learn</title>
    <link rel="stylesheet" href="css/index.css">
    <link rel="stylesheet" href="css/normalize.css">
</head>

<body>
    <div class="container">
        <div>1</div>
        <div>2</div>
        <div>3</div>
        <div>4</div>
        <div>5</div>
        <div>6</div>
        <div>7</div>
        <div>8</div>
        <div>9</div>
    </div>
</body>

</html>
```

```css
.container {
    width: 600px;
    height: 600px;
    margin: auto;

    background-color: lightgray;

    display: grid;
    gap: 5px;
    grid-template-columns: repeat(3, 1fr);
    grid-auto-flow: row;
}

.container div {   
    background-color: violet;
}
```

## CSS 变量

定义 CSS 变量时，可以使用伪类 `:root`，在 `:root` 样式中定义 CSS 变量

```css
:root {
	--BGCOLOR: #333;
}
```

在其他地方引用该变量时，使用 `var(--BGCOLOR)` 即可引用

## CSS 函数

> [!seealso] CSS 参考 [菜鸟教程-CSS 函数](https://www.runoob.com/cssref/css-functions.html)

## CSS 动画

> [!seealso] CSS 动画参考 [菜鸟教程-CSS 动画](https://www.runoob.com/cssref/css-animatable.html)
