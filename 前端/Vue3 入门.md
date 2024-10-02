# Vue3

Vue 是一款用于构建用户界面的 JavaScript 框架。基于标准 HTML CSS 和 JavaScript 构建，并提供了一套声明式的、组件化的编程模型

## 创建 Vue 工程

创建 Vue 工程可以使用 `npm` 

```shell
➜  Documents npm create vue@latest

> npx
> create-vue


Vue.js - The Progressive JavaScript Framework

√ 请输入项目名称： ... vue-learn
√ 是否使用 TypeScript 语法？ ... 否 / 是  # 是
√ 是否启用 JSX 支持？ ... 否 / 是  # 否
√ 是否引入 Vue Router 进行单页面应用开发？ ... 否 / 是  # 否，学习路由时手动配置
√ 是否引入 Pinia 用于状态管理？ ... 否 / 是  # 否，学习状态管理时手动配置
√ 是否引入 Vitest 用于单元测试？ ... 否 / 是  # 否
√ 是否要引入一款端到端（End to End）测试工具？ » 不需要
√ 是否引入 ESLint 用于代码质量检测？ ... 否 / 是  # 否
√ 是否引入 Vue DevTools 7 扩展用于调试? (试验阶段) ... 否 / 是  # 否

正在初始化项目 C:\Users\duyup\Documents\vue-learn...

项目初始化完成，可执行以下命令：

  cd vue-learn
  npm install
  npm run dev
```

工程的目录结构

```shell
.
├── README.md          # 项目介绍文件
├── env.d.ts           # ts 环境设置：.js .css 这些文件 ts 无法识别
├── index.html         # html 入口
├── node_modules       # 安装依赖后的目录
├── package-lock.json  # 执行完npm install后, 记录的当前想起使用的依赖的具体版本
├── package.json       # 项目文件，记载着一些命令和依赖还有简要的项目描述信息 
├── public             # 纯静态资源, 入口文件也在里面
├── src                # 源码目录
│   ├── App.vue          # root组件
│   ├── assets           # 附件目录
│   │   ├── base.css
│   │   ├── logo.svg
│   │   └── main.css
│   ├── components       # 存放 子组件目录
│   └── main.ts          # 
├── tsconfig.app.json  # ts 配置
├── tsconfig.json
├── tsconfig.node.json
└── vite.config.ts  # vite 配置文件
```

首先，我们来看 HTML 入口文件 `index.html`

```html title:index.html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8">
    <link rel="icon" href="/favicon.ico">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Vite App</title>
  </head>
  <body>
    <div id="app"></div>  <!-- 容器：渲染组件的容器 -->
    <script type="module" src="/src/main.ts"></script> <!-- 引入源码中的 main.ts 文件-->
  </body>
</html>
```

```ts title:src/main.ts
import './assets/main.css'

import { createApp } from 'vue'  /* 从 vue 对象中导入 createApp 函数 */
import App from './App.vue'     /* 导入 App 组件：root 组件 */

createApp(App).mount('#app')  /* 创建一个 APP 设置 root 组件，并挂在到 id 为 #app 的容器上 */
```

```html title:src/App.vue
<script setup lang="ts">
import HelloWorld from './components/HelloWorld.vue'  /* 导入子组件 */
import TheWelcome from './components/TheWelcome.vue' 
</script>

<template>
  <header>
    <img alt="Vue logo" class="logo" src="./assets/logo.svg" width="125" height="125" />

    <div class="wrapper">
      <HelloWorld msg="You did it!" />  <!-- 应用子组件 -->
    </div>
  </header>

  <main>
    <TheWelcome /> <!-- 应用子组件 -->
  </main>
</template>

<style scoped>
header {
  line-height: 1.5;
}

.logo {
  display: block;
  margin: 0 auto 2rem;
}

@media (min-width: 1024px) {
  header {
    display: flex;
    place-items: center;
    padding-right: calc(var(--section-gap) / 2);
  }

  .logo {
    margin: 0 2rem 0 0;
  }

  header .wrapper {
    display: flex;
    place-items: flex-start;
    flex-wrap: wrap;
  }
}
</style>
```

> [!tip] 组件，就类似于自定义标签

## 自定义组件

Vue组件中可以定义三个顶级标签 `<template></template>` `<script> </script>` 和 `<style> </style>`

```html
<template>
<!--组件的视图模板-->
</template>


<script lang="ts">
/* 组件的 ts 或 js 代码 */
</script>

<style lang="css">

/* 组件视图的 css 代码 */ 
</style>

```

下面定义了一个 `App.vue` 和 `Person.vue`

```html
<template>
  <div class="container">
    <h2> 你好啊！ </h2>
    <Person/>
  </div>
</template>


<script lang="ts">
import Person  from "./components/Person.vue";
export default {
  name: "App", // 定义组件的名字
  components:{Person}, // 注册组件
}
</script>

<style lang="css">
.container {
  background-color: gray;
  height: 300px;
  border: 2px solid #333;
  box-shadow:10px 5px 5px #444;
}
</style>
```

```html
<template>
    <div class="person-box">
        <h3>name: {{ name }}</h3>
        <h3>age: {{ age }}</h3>
        <button v-on:click="showTel">点击显示联系方式</button>
    </div>
</template>


<script lang="ts">
    export default {
        name: "Person",
        data() {
            return {
                name:"张三",
                age:27,
                tel:"13788889999"
            }
        },
        methods:{
            showTel() {
                alert(this.tel)
            }
        }
    }
</script>

<style lang="css">
.person-box {
    background-color: skyblue;
    width: 600px;
    height: 200px;
    border: 2px solid #333;
    box-shadow:10px 5px 5px #444;
    margin: auto;
}
</style>
```

> [!tip] 这里我们使用的是 Vue2 的语法
> 
> 在 Vue3 中兼容 Vue2 的语法
> 

## 选项式 API 和 组合式 API

我们在 `Person.vue` 中的 `script` 标签中写的就是 **选项式 API**。数据，方法，计算属性等分散在各自的 **选项**(`data` `method` `computed`) 中，这非常不便于维护

> [!question] 选项式 API 将功能的数据和方法分散了。如果需要修改，就需要修改所有选项中有关的内容

Vue3 推荐采用 **组合式 API**，采用 **函数** 的方式组织代码，**让与功能相关数据 方法 计算属性组合在一起**

## setup

Vue3 中，`setup` 是一个全新的配置项，它是一个函数

```html
<template>
    <div class="person-box">
        <h3>name: {{ name }}</h3>
        <h3>age: {{ age }}</h3>
        <button v-on:click="showTel">点击显示联系方式</button>
    </div>
</template>


<script lang="ts">
    export default {
        name: "Person",
        setup() {
	        // 数据定义：这样定义的数据是非响应式的
            let name = "张三"
            let age = 19
            let tel = "13888888888"

			// 方法定义
            function showTel() {
                alert(tel)
            }
            // return 将函数和方法转交给模板使用
            return {name, age, tel, showTel}
        },
    }
</script>

<style lang="css">
.person-box {
    background-color: skyblue;
    width: 600px;
    height: 200px;
    border: 2px solid #333;
    box-shadow:10px 5px 5px #444;
    margin: auto;
}
</style>
```

### setup 与 选项式 API

> [!tip] `data` 选项和 `methods` 选项可以和 `setup` 函数同时存在

> [!tip] `data` 选项可以读取 `setup` 返回的数据，通过 `this` 读取
> 
> + `setup` 是 Vue3 中最早的声明周期函数。因此，`data` 和 `methods` 选项中可以读取 `setup` 返回的数据
> + 明显，`setup` 中不能使用 `data` 和 `methods` 选择中的数据和函数

> [!attention] 
> 不建议在 Vue3 的项目中在写 选项式 API
>

### setup 语法糖

在编写 setup 函数式，我们只希望写 **数据** 和 **方法**。然而，在上面的示例中，我们不得不写一个 `setup` 函数，和其返回值

> [!question] 
> 
> 如果忘记 `return` 定义的数据和方法，那么就无法在模板中使用了
> 

Vue3 提供了一个语法糖，只需要在组件中新增一个 `script` 标签，并指定 `setup` 属性，在这个标签中定义的数据和方法将会自动 `return`

```html
<template>
    <div class="person-box">
        <h3>name: {{ name }}</h3>
        <h3>age: {{ age }}</h3>
        <button v-on:click="showTel">点击显示联系方式</button>
    </div>
</template>

<!--定义组件名字-->
<script lang="ts">
export default {
    name: "Person",
}
</script>

<!-- setup 语法糖 -->
<script setup lang="ts">
let name = "张三"
let age = 19
let tel = "13888888888"

function showTel() {
    alert(tel)
}
</script>

<style lang="css">
.person-box {
    background-color: skyblue;
    width: 600px;
    height: 200px;
    border: 2px solid #333;
    box-shadow: 10px 5px 5px #444;
    margin: auto;
}
</style>
```

> [!tip] 所有的 `script` 标签必须使用相同的 `lang` 属性值

配置组件的名字需要一个 `script` 标签，配置组合式 API 有需要一个 `script` 标签。导致代码看起来重复，通过 **开发插件** `vite-plugin-vue-setup-extend` 提供支持，让 `script` 标签减少一个

```shell
npm i vite-plugin-vue-setup-extend -D
```

并在 `vite` 的配置文件中导入带插件

```ts
import { fileURLToPath, URL } from 'node:url'

import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'
import vueSetupExtend from "vite-plugin-vue-setup-extend" /* 导入插件 */

// https://vitejs.dev/config/
export default defineConfig({
  plugins: [
    vue(),
    vueSetupExtend() /* 加载插件 */
  ],
  resolve: {
    alias: {
      '@': fileURLToPath(new URL('./src', import.meta.url))
    }
  }
})
```

这样，就可以在 `setup` 的 `script` 标签中定义 `name` 属性，用于指定组件名字

```html
<template>
    <div class="person-box">
        <h3>name: {{ name }}</h3>
        <h3>age: {{ age }}</h3>
        <button v-on:click="showTel">点击显示联系方式</button>
    </div>
</template>


<!-- <script lang="ts">
export default {
    name: "Person",
}
</script> -->

<script setup lang="ts" name="Person123">
let name = "张三"
let age = 19
let tel = "13888888888"

function showTel() {
    alert(tel)
}
</script>

<style lang="css">
.person-box {
    background-color: skyblue;
    width: 600px;
    height: 200px;
    border: 2px solid #333;
    box-shadow: 10px 5px 5px #444;
    margin: auto;
}
</style>
```

> [!tip] 组件名通常和文件名相同，一般不写 `name`

## 响应式数据：ref, reactive

在 Vue2 中，只要在 `data` 选项中定义的数据，它自动变为响应式数据(Vue2 劫持数据，并代理它)。

然而，在 Vue3 中，使用组合式 API，响应式数据的定义发生了变化。通过 `ref` 和 `reactive` 定义

### ref：基本类型

现在，我们新增两个按钮，分别用于修改名字和修改年龄

```html
<template>
  <div class="person-box">
    <h3>姓名: {{ name }}</h3>
    <h3>年龄: {{ age }}</h3>
    <button v-on:click="changeName">修改名字</button>
    <button v-on:click="changeAge">修改年龄</button>
    <button v-on:click="showTel">点击显示联系方式</button>
  </div>
</template>

<script setup lang="ts" name="Person123">
let name = "张三"
let age = 19
let tel = "13888888888"

function  changeName() {
  name = "zhang san"
}

function changeAge() {
  age += 1
}

function showTel() {
  alert(tel)
}

</script>

<style lang="css">
.person-box {
  background-color: skyblue;
  width: 600px;
  height: 200px;
  border: 2px solid #333;
  box-shadow: 10px 5px 5px #444;
  margin: auto;
}
</style>
```

然而，这样做时，点击按钮后，页面并没有任何变化。因为，`name` 和 `age` 定义的数据时普通数据，不是响应式数据

通过 `ref()` 函数，才会定义一个响应式数据

```html
<template>
  <div class="person-box">
    <h3>姓名: {{ name }}</h3>
    <h3>年龄: {{ age }}</h3>
    <button v-on:click="changeName">修改名字</button>
    <button v-on:click="changeAge">修改年龄</button>
    <button v-on:click="showTel">点击显示联系方式</button>
  </div>
</template>

<script setup lang="ts" name="Person123">
import {ref} from "vue";

let name = ref("张三") /* 返回一个 RefImpl 对象 */
let age = ref(19)
let tel = "13888888888"

function  changeName() {
  name.value = "zhang san" /* RefImpl 对象的 vaule 属性存储响应数据*/
}

function changeAge() {
  age.value += 1
}

function showTel() {
  alert(tel)
}

</script>

<style lang="css">
.person-box {
  background-color: skyblue;
  width: 600px;
  height: 200px;
  border: 2px solid #333;
  box-shadow: 10px 5px 5px #444;
  margin: auto;
}
</style>
```

> [!tip] `ref` 用于创建 **基本类型** 的响应式数据
> 
> + 也可以用于定义对象类型的响应式数据
> 

### reactive：对象类型

`reactive` 函数将对象类型通过 `Proxy` 进行代理，也就是会返回一个 `Proxy` 对象
 
```html
<template>
  <div class="person-box">
    <h3>姓名: {{ person.name }}</h3>
    <h3>年龄: {{ person.age }}</h3>
    <button v-on:click="changeName">修改名字</button>
    <button v-on:click="changeAge">修改年龄</button>
    <button v-on:click="showTel">点击显示联系方式</button>
  </div>
</template>

<script setup lang="ts" name="Person123">
import {reactive} from "vue";

// 使用 reatcive 创建对象类型的响应式数据
let person = reactive({name:"张三", age:19, tel:"13788899999"}) // 返回的是 Proxy 对象

function  changeName() {
  person.name = "zhang san"
}

function changeAge() {
  person.age += 1
}

function showTel() {
  alert(person.tel)
}

</script>

<style lang="css">
.person-box {
  background-color: skyblue;
  width: 600px;
  height: 200px;
  border: 2px solid #333;
  box-shadow: 10px 5px 5px #444;
  margin: auto;
}
</style>
```

> [!tip] 
> 
> 通过 `reactive` 定义的响应式对象被替换之后，页面不会自动刷新。应该使用 `Object.assign()` 的方式进行复制
> + `Object.assign(obj1, obj2,...)` 将 `obj2` 中的所有属性复制到 `obj1` 中
> 
> 通过 `ref` 定义的响应式对象，也只能替换 `.value` 的值
> 

```html
<template>
  <div class="person-box">
    <h3>姓名: {{ person.name }}</h3>
    <h3>年龄: {{ person.age }}</h3>
    <button v-on:click="changeName">修改名字</button>
    <button v-on:click="changeAge">修改年龄</button>
    <button v-on:click="showTel">显示联系方式</button>
    <button v-on:click="changePerson">修改对象</button>
  </div>
</template>

<script setup lang="ts" name="Person123">
import {reactive, ref} from "vue";
import Person from "@/components/Person.vue";

let person = reactive({name:"张三", age:19, tel:"13788899999"})

function  changeName() {
  person.name = "zhang san"
}

function changeAge() {
  person.age += 1
}

function showTel() {
  alert(person.tel)
}

function changePerson() {
  // person = reactive({name: "李四", age: 20, tel:"13699787789"})
  Object.assign(person,{name: "李四", age: 20, tel:"13699787789"})  // 会使用第二个参数的属性覆盖 第一个参数的属性
}
</script>

<style lang="css">
.person-box {
  background-color: skyblue;
  width: 600px;
  height: 200px;
  border: 2px solid #333;
  box-shadow: 10px 5px 5px #444;
  margin: auto;
}
</style>
```

### toRefs

**从响应式对象中解构出的数据不是响应式数据**。为了让它变为响应式数据，使用 `toRefs` 函数，就可以解构出响应式数据

```html
<template>
  <div class="person-box">
    <h3>姓名: {{ person.name }}</h3>
    <h3>年龄: {{ person.age }}</h3>
    <button v-on:click="changeName">修改名字</button>
    <button v-on:click="changeAge">修改年龄</button>
    <button v-on:click="showTel">显示联系方式</button>
  </div>
</template>

<script setup lang="ts" name="Person123">
import {reactive, ref, toRefs} from "vue";

let person = reactive({name:"张三", age:19, tel:"13788899999"})

let {name, age, tel} = toRefs(person) // 将person 中的属性转变为 RefImpl 对象。这些属性和 person 中的属性时关联的

function  changeName() {
  name.value += "~"
}

function changeAge() {
  age.value += 1
}

function showTel() {
  alert(tel.value)
}
</script>

<style lang="css">
.person-box {
  background-color: skyblue;
  width: 600px;
  height: 200px;
  border: 2px solid #333;
  box-shadow: 10px 5px 5px #444;
  margin: auto;
}
</style>
```

> [!tip] `toRefs` 将 `reactive` 定义的响应式对象的所有属性包装为 `RefImpl` 对象
> 

## MVVM 如何诞生的

现在主流的前端框架都是 `MVVM` 模型, `MVVM` 分为三个部分：

- M（Model，**模型层** ）: 模型层，主要负责业务数据相关, 对应 `vue` 中的 `data` 部分
- V（View，**视图层**）: 视图层，顾名思义，负责视图相关，细分下来就是 `html+css` 层, 对应于 `vue` 中的模版部分
- VM（ViewModel, **控制器**）: `V` 与 `M` 沟通的桥梁，负责监听 `M` 或者 `V` 的修改，是实现 `MVVM` 双向绑定的要点, 对应 `vue` 中双向绑定

Vue 就是这种思想下的产物, 但是要讲清楚这个东西，我们不妨来看看 `web` 技术的进化史

### CGI 

最早的 HTML 页面是完全静态的网页，它们是预先编写好的存放在 Web 服务器上的 html 文件, 浏览器请求某个 URL 时，Web 服务器把对应的 html 文件扔给浏览器，就可以显示 html 文件的内容了

如果要针对不同的用户显示不同的页面，显然不可能给成千上万的用户准备好成千上万的不同的 html 文件，所以，服务器就需要针对不同的用户，动态生成不同的 html 文件。一个最直接的想法就是利用C、C++这些编程语言，**直接向浏览器输出拼接后的字符串**。这种技术被称为 **CGI**：Common Gateway Interface

### 后端模版

很显然，像新浪首页这样的复杂的 HTML 是不可能通过拼字符串得到的, 于是，人们又发现，**其实拼字符串的时候，大多数字符串都是 HTML 片段，是不变的**，变化的只有少数和用户相关的数据, 所以我们做一个 **模版** 出来，把不变的部分写死, **变化的部分动态生成**, 其实就是一套模版渲染系统,

但是，**一旦浏览器显示了一个 HTML 页面，要更新页面内容，唯一的方法就是重新向服务器获取一份新的 HTML 内容**。如果浏览器想要自己修改 HTML 页面的内容，怎么办？那就需要等到1995年年底，JavaScript 被引入到浏览器

### JavaScript 原生

有了 JavaScript 后，浏览器就可以运行 JavaScript，然后，对页面进行一些修改。JavaScript 还可以通过修改 HTML 的 DOM 结构和 CSS 来实现一些动画效果，而这些功能没法通过服务器完成，必须在浏览器实现

```html
<p id="userInfo">
姓名:<span id="name">Gloria</span>
性别:<span id="sex">男</span>
职业:<span id="job">前端工程师</span>
</p>
```

有以上 html 片段，想将其中个人信息替换为 alice 的，我们的做法

```js
// 通过ajax向后端请求, 然后利用js动态修改展示页面
document.getElementById('name').innerHTML = alice.name;
document.getElementById('sex').innerHTML = alice.sex;
document.getElementById('job').innerHTML = alice.job;
```

jQuery 脱颖而出

```html
<div id="name" style="color:#fff">前端你别闹</div> <div id="age">3</div>
<script>
$('#name').text('好帅').css('color', '#000000'); $('#age').text('666').css('color', '#fff');
/* 最终页面被修改为 <div id="name" style="color:#fff">好帅</div> <div id="age">666</div> */
</script>
```

在此情况下可以下 **前后端算是分开** 了, 后端提供数据, **前端负责展示**, 只是现在 **前端里面的数据和展示并有分开**，不易于维护

### 前端模版

在架构上前端终于走上后端的老路: **模版系统**, 有引擎就是 `ViewModel` 动态完成渲染

```html
<div id="userInfoTemplate">
姓名:<span>{name}</span>
性别:<span>{sex}</span>
职业:<span>{job}</span>
</div>
```

```js
var userInfo = document.getElementById('userInfo');
var userInfoTemplate = document.getElementById('userInfoTemplate').innerHTML;
userInfo.innerHTML = templateEngine.render(userInfoTemplate, users.alice);
```

### 虚拟DOM技术

用我们传统的开发模式，原生 JS 或 JQuery 操作 DOM 时，浏览器会从构建 DOM 树开始从头到尾执行一遍流程, 操作 DOM 的代价仍旧是昂贵的，频繁操作还是会出现页面卡顿，影响用户体验

**如何才能对 Dom 树做局部更新，而不是全局更新喃?** 答案就是由 js 动态来生产这个树, 修改时动态更新, 这就是 **虚拟 Dom**

### 组件化

`MVVM` 最早由微软提出来，它借鉴了桌面应用程序的 MVC 思想，在前端页面中，把 `Model` 用纯 JavaScript 对象表示，View 负责显示，两者做到了最大限度的分离。

![[Pasted image 20240915171312.png]]

结合虚拟 DOM 技术, 我们就可以动态生成 view, 在集合 mvvm 的思想, 前端终于迎来了组件化时代

> [!tip] 
> 
> + 页面由多个组建构成
>+ 每个组件都有自己的 MVVM

![[Pasted image 20240915171407.png]]

## 响应式原理

Vue 实现了 `view` 和 `model` 的 **双向绑定,** 如下

![[Pasted image 20240915174204.png]]

而 **响应式** 指的就是 **`model`(数据)有变化时，能反馈到 `view` 上**, 当然这个反馈是由 `view` 实例来帮我们完成的， 那 `view` 怎么知道 `model`(数据)有变化喃?

### Proxy

实现响应式的手段之一就是 [Proxy](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy)， 其行为表现与一般对象相似。不同之处在于 Vue 能够跟踪对响应式对象 **属性** 的 _访问_ 与 _更改_ 操作

```js
const monster1 = { eyeCount: 4 };

const handler1 = {
  set(obj, prop, value) {
    if ((prop === 'eyeCount') && ((value % 2) !== 0)) {
      console.log('Monsters must have an even number of eyes');
    } else {
      return Reflect.set(...arguments);
    }
  }
};

const proxy1 = new Proxy(monster1, handler1);

proxy1.eyeCount = 1;
// expected output: "Monsters must have an even number of eyes"

console.log(proxy1.eyeCount);
// expected output: 4

proxy1.eyeCount = 2;
console.log(proxy1.eyeCount);
// expected output: 2
```

当我修改数据时, Vue 实例就会感知到, 感知到后 使用 js 操作 `dom(vdom)`, 完成视图的更新。那 Vue 必须提供一个构造函数用于初始化 **原生对象**，这样 Vue 才能跟踪数据变化,  Vue 提供一个 `reactive` 函数 就是用来干这个的

```js
function reactive(obj) {
  return new Proxy(obj, {
    get(target, key) {
      track(target, key)
      return target[key]
    },
    set(target, key, value) {
      trigger(target, key)
      target[key] = value
    }
  })
}
```

### getter/setters

**`Proxy` 仅对对象类型有效**（对象、数组和 Map、Set 这样的集合类型），而对 `string`、`number` 和 `boolean` 这样的 **原始类型 无效**

为了解决 `reactive()` 带来的限制，Vue 也提供了一个 `ref()` 方法来允许我们创建可以使用任何值类型的响应式对象

`ref()` 利用的是 JavaScript 的 `getter/setters` 的方式 **劫持属性访问**

```js
function ref(value) {
  const refObject = {
    get: value() {
      track(refObject, 'value')
      return value
    },
    set: value(newValue) {
      trigger(refObject, 'value')
      value = newValue
    }
  }
  return refObject
}
```

### DOM异步更新

注意这是一个很重要的概念, **当你响应式数据发送变化时, DOM 也会自动更新**, 但是这个更新并不是同步的，而是**异步**的: **vue 会将你的变更放到一个缓冲队列, 等待更新周期到达时, 一次性完成DOM(视图)的更新**。

比如下面这个例子:

```vue
<script setup>
import { onMounted } from "vue";

// 使用ref来为基础类型 构造响应式变量
let name = $ref("老喻");

// 只有等模版挂载好后，我门才能获取到对应的HTML元素
onMounted(() => {
  // 通过value来设置 基础类型的值(Setter方式)
  name = "张三";
  console.log(document.getElementById("name").innerText);
});
</script>
```

由于修改 `name` 后, 并没有立即更新DOM, 所以获取到的 `name` 依然时初始值, 那如果想要获取到当前值怎么办?

vue 在 dom 更新时 为我们提供了一个钩子: `nextTick() `

该钩子就是当 vue 实例 到达更新周期后, 更新完Dom后，留给我们操作的口子, 因此我们改造下:

```vue
<script setup>
import { nextTick, onMounted } from "vue";

// 使用ref来为基础类型 构造响应式变量
let name = $ref("老喻");

// 只有等模版挂载好后，我门才能获取到对应的HTML元素
onMounted(() => {
  // 通过value来设置 基础类型的值(Setter方式)
  name = "张三";

  // 等待vue下次更新到来后, 执行下面的操作
  nextTick(() => {
    console.log(document.getElementById("name").innerText);
  });
});
</script>
```

一定要理解 vue 的异步更新机制, 因为你写的代码 并不是按照你的预期同步执行的, 这是引起很多魔幻 bug 的根源

## 计算属性：computed

Vue2 中计算属性通过 `computed` 选项定义。在 Vue3 中使用 `computed()`  函数定义，该函数接收一个 **对象** 或者一个 **函数**

> [!tip] 对象参数
> 
> 这对象必须提供 `get()` 和 `set()` 方法，`get` 用于获取计算属性，`set` 用于修改计算属性
> 

> [!tip] 函数参数
> 
> 如果值传递一个函数，则该函数等价于 `get`
> 

下面的 Vue 组件颜色一个计算全名的例子

```html

<template>
  <div class="container">
    <h2>计算全名</h2>
<!--    <label for="#last-name">姓:</label> <input id="last-name" type="text" v-model:value="lastName">-->
    <!--对于input标签来说，其实是 v-model 默认绑定的就是 value，所以不需要再写一个value-->
    <label for="#last-name">姓:</label> <input id="last-name" type="text" v-model="lastName">
    <label for="#first-name">名:</label> <input id="first-name" type="text" v-model="firstName">
    全名: <span> {{fullName}} </span>
    <hr>
    <button v-on:click="changeFullName">修改名字为:li-si</button>

  </div>

</template>

<script setup lang="ts" name="FullName">

import {computed, ref} from "vue";

let lastName = ref("张")
let firstName = ref("三")

let fullName = computed({
  get: function() {
    return lastName.value.slice(0,1).toUpperCase().concat("",lastName.value.slice(1)).concat("-", firstName.value)
  },
  set: function(val) {
    [lastName.value, firstName.value] = val.split("-");
  }
})

function changeFullName() {
  fullName.value = "li-si"
}

</script>


<style scoped lang="css">

</style>
```

## 监视：watch

`watch` 用于 **监视数据的变化**，当数据满足一定条件时，触发一些操。Vue3 中 `watch` 只能监视下面 $4$ 种数据

> [!tip] watch 能够监视的数据
> 
> + `ref` 定义的数据
> + `reactive` 定义的数据
> + 函数的返回值
> + 上述内容的一个数组

`watch` 函数接收两个参数，第一个参数是监视的对象，第二个参数是数据变化时触发的回调函数

> [!tip] 该回调函数需要接收两个参数：第一参数时新的值，第二个参数时旧的值

> [!tip] `watch` 函数会返回一个用于取消监视的函数

### 情形一

监视 `ref` 定义的 **基本类型**。`watch` 对象的第一个参数直接写 `ref` 对象的名字即可，监视的是其 `value` 值的变化

```html
<template>
  <div class="person-box">

    <h3> sum= {{sum}}</h3>

    <button v-on:click="addSum">点击sum+1</button>
  </div>
</template>

<script setup lang="ts" name="Person123">
import {reactive, ref, watch} from "vue";

let sum = ref(0)

function addSum() {
  sum.value += 1;
}

let cancelWatch = watch(sum, (newVal, oldVal) => {
  console.log(newVal, oldVal);
  if (newVal >= 10) {
    cancelWatch()
  }
})

</script>

<style lang="css">
.person-box {
  background-color: skyblue;
  width: 600px;
  height: 200px;
  border: 2px solid #333;
  box-shadow: 10px 5px 5px #444;
  margin: auto;
}
</style>
```

### 情形二

监视 `ref` 定义的 **对象类型**。`watch` 对象的第一个参数直接写 `ref` 对象的名字即可，监视的是其 `value` 值(**对象的引用地址**)的变化。如果需要 **监视对象的属性**，需要 **手动开启深度监视**

> [!attention] 
> 
> 若修改的是 `ref` 定义的对象中的属性，`setter` 函数的 `newVal` 和 `oldVal` 都是新值，因为它们是同一个对象
> 
> 若修改的是整个 `ref` 定义的对象，`setter` 函数的 `newVal` 和 `oldVal` 是不同的，因为不是同一个对象了
> 

```html
<template>
  <div class="person-box">

    <h2>watch ref(object)</h2>

    <h3> 姓名: {{person.name}}</h3>
    <h3>年龄: {{person.age}}</h3>

    <button @click="changeName" >修改名字</button>
    <button @click="changeAge" >修改年龄</button>
    <button @click="changePerson" >修改person</button>

  </div>
</template>

<script setup lang="ts" name="Person123">
import {reactive, ref, watch} from "vue";

// 数据
let person = ref({
  name:"张三",
  age:19,
})

// 方法
function changeName() {
  person.value.name += "~"
}

function changeAge() {
  person.value.age += 1
}

function changePerson() {
  person.value = {
    name:"李四",
    age: 20
  }
}

// 监视：只关注 person.value 的值(对象的引用)是否发生改变
watch(person,(newVal, oldVal) => {
  console.log("new:", newVal)
  console.log("old:", oldVal)
})

// 监视：关注对象内部属性的变化，第三参数是一个配置对象，deep 开启深度监视
watch(person,(newVal, oldVal) => {
  console.log("new:", newVal)
  console.log("old:", oldVal)
}, {
  deep: true, // 开深度监视
  immediate: true, // 立即执行异常监视的回调函数
})

</script>

<style lang="css">
.person-box {
  background-color: skyblue;
  width: 600px;
  height: 200px;
  border: 2px solid #333;
  box-shadow: 10px 5px 5px #444;
  margin: auto;
}
</style>
```

### 情形三

监视 `reactive` 定义的对象，这样会 **默认开启深度监视**，并且 **深度监视无法手动关闭**

```html
<template>
  <div class="person-box">

    <h2>watch reactive(object)</h2>

    <h3> 姓名: {{person.name}}</h3>
    <h3>年龄: {{person.age}}</h3>

    <button @click="changeName" >修改名字</button>
    <button @click="changeAge" >修改年龄</button>
    <button @click="changePerson" >修改person</button>

  </div>
</template>

<script setup lang="ts" name="Person123">
import {reactive, ref, watch} from "vue";

// 数据
let person = reactive({
  name:"张三",
  age:19,
})

// 方法
function changeName() {
  person.name += "~"
}

function changeAge() {
  person.age += 1
}

function changePerson() {
  // person = {
  //   name:"李四",
  //   age: 20
  // }

  Object.assign(person,{
      name:"李四",
      age: 20
  })
  
}

// 监视：关注对象内部属性的变化，隐式开启深度监视，这个深度监视无法关闭
watch(person,(newVal, oldVal) => {
  console.log("new:", newVal)
  console.log("old:", oldVal)
})

</script>

<style lang="css">
.person-box {
  background-color: skyblue;
  width: 600px;
  height: 200px;
  border: 2px solid #333;
  box-shadow: 10px 5px 5px #444;
  margin: auto;
}
</style>
```

### 情形四

如果只想 **监视对象的一个属性**，
+ 如果，这个属性不是对象类型，需要**使用函数(`getter`)包装一下**
+ 如果，这个属性是一个对象类型，可以直接监视。但是，建议也使用 `getter` 函数包装一下，如果需要关注监视对象中的属性，需要开启深度监视

```html
<template>
  <div class="person-box">

    <h2>watch reactive(object)</h2>

    <h3> 姓名: {{ person.name }}</h3>
    <h3>年龄: {{ person.age }}</h3>
    <h3>成绩:</h3>
    <ul>
      <li v-for="s in person.score">{{ s }}</li>
    </ul>
    <hr>
    <button @click="changeName" >修改名字</button>
    <button @click="changeAge" >修改年龄</button>
    <button @click="changePerson" >修改person</button>
    <button @click="addEnglish">修改 english 成绩</button>
    <button @click="addMath">修改 math 成绩</button>

  </div>
</template>

<script setup lang="ts" name="Person123">
import {reactive, ref, watch} from "vue";

// 数据
let person = reactive({
  name:"张三",
  age:19,
  score: {
    math:90,
    english:89,
  }
})

// 方法
function changeName() {
  person.name += "~"
}

function addEnglish() {
  person.score.english += 1
}
function addMath() {
  person.score.math += 1
}

function changeAge() {
  person.age += 1
}

function changePerson() {
  // person = {
  //   name:"李四",
  //   age: 20
  // }

  Object.assign(person,{
      name:"李四",
      age: 20
  })
}

// 监视对象中的一个属性(基本类型)
watch(()=>person.name, (newVal, oldVal) => {
  console.log(newVal, oldVal);
})

// 监视对象中的一个属性(对象类型)，通过函数包装后的属性，如果需要监视对象的属性，需要开启 deep 监视
watch(()=>person.score, (newVal, oldVal) => {
  console.log(newVal, oldVal);
}, {deep: true})

</script>

<style lang="css">
.person-box {
  background-color: skyblue;
  width: 600px;
  height: auto;
  border: 2px solid #333;
  box-shadow: 10px 5px 5px #444;
  margin: auto;
}
</style>
```

> [!tip]
> 
> 如果监视一个对象中的属性，只使用 `getter` 函数。在必要的时候(属性为对象，并且需要监视该对象中属性)，就开启深度监视
> 

###  情形五

监视一个数组：数组中的内容，必须是 `ref` `reactive` 或者是一个 `getter` 函数

```html
<template>
  <div class="person-box">

    <h2>watch reactive(object)</h2>

    <h3> 姓名: {{ person.name }}</h3>
    <h3>年龄: {{ person.age }}</h3>
    <h3>成绩:</h3>
    <ul>
      <li v-for="s in person.score">{{ s }}</li>
    </ul>
    <hr>
    <button @click="changeName" >修改名字</button>
    <button @click="changeAge" >修改年龄</button>
    <button @click="changePerson" >修改person</button>
    <button @click="addEnglish">修改 english 成绩</button>
    <button @click="addMath">修改 math 成绩</button>

  </div>
</template>

<script setup lang="ts" name="Person123">
import {reactive, ref, watch} from "vue";

// 数据
let person = reactive({
  name:"张三",
  age:19,
  score: {
    math:90,
    english:89,
  }
})

// 方法
function changeName() {
  person.name += "~"
}

function addEnglish() {
  person.score.english += 1
}
function addMath() {
  person.score.math += 1
}

function changeAge() {
  person.age += 1
}

function changePerson() {
  // person = {
  //   name:"李四",
  //   age: 20
  // }

  Object.assign(person,{
      name:"李四",
      age: 20
  })
}

watch([()=>person.name, ()=>person.score], (value, oldValue) => {
  console.log(value, oldValue)
}, {deep: true})


</script>

<style lang="css" scoped> /*scoped 代表局部样式*/
.person-box {
  background-color: skyblue;
  width: 600px;
  height: auto;
  border: 2px solid #333;
  box-shadow: 10px 5px 5px #444;
  margin: auto;
}
</style>
```

### watchEffect

`watchEffect` 会 *立即执行一个回调函数*，同时 *响应式地追踪其依赖*，并在依赖更改时执行该函数

> [!tip] `watchEffect` 不用明确指出监视的数据，函数中用到那些属性，就监视那些属性

## 标签的ref属性

`ref` 属性用于注册模板引用

> [!tip] 
> 
> `ref` 属性用在普通的 **HTML 标签** 上，获取的就是 **DOM 对象**
> 
> `ref` 属性用在 **组件标签** 上，获取的就是 **组件对象**
> 

> [!attention] 使用 `ref()` 函数获取 `ref` 属性对应的标签。注意，定义的变量名要与 `ref` 属性值相同


### 组件

```html
<template>

  <div class="container">
    <Person ref="hello"/>
    <button @click="showObject">显示 ref 引用的对象</button>
  </div>

</template>


<script lang="ts" setup name="App">
import Person  from "./components/Person.vue";
import {ref} from "vue";

let hello = ref()  // 获取的 Person 组件对象

function showObject(){
  console.log(hello.value)
}
</script>

<style lang="css" scoped>
.container {
  background-color: cadetblue;
  width: 600px;
  height: 600px;
  border: 2px solid #333;
  box-shadow: 10px 5px 5px #444;
  margin: auto;
}
</style>

```

### 普通 HTML 标签

```html
<template>
  <div class="person-box">
    <button @click="showObject">显示 ref 引用的对象</button>

    <h2 ref="hello">你好</h2>
  </div>
</template>

<script setup lang="ts" name="Person123">
import {ref} from "vue";

let hello = ref()  // 获取 ref="hello" 标签对象

function showObject(){
  console.log(hello.value)
}

</script>

<style lang="css" scoped>
.person-box {
  background-color: skyblue;
  width: 600px;
  height: auto;
  border: 2px solid #333;
  box-shadow: 10px 5px 5px #444;
  margin: 100px auto;
}
</style>
```

## props

**父组件向子组件传递数据** 时，需要通过 `props` 传递。在父组件中，通过在 **组件标签** 上的 **自定义属性** 传递数据；在子组件中，通过 `defineProps([])` 接收父组件传递的数据

在父组件(`App.vue`)中

```html title:App.vue
<template>

  <div class="container">
    <Person v-bind:persons="persons"/>
  </div>

</template>


<script lang="ts" setup name="App">
import Person  from "./components/Person.vue";
import type {Persons} from "@/types";
import {reactive} from "vue";


let persons = reactive<Persons>([
  {id: "1000", name:"张三", age: 20},
  {id: "1001", name:"李四", age: 21},
  {id: "1002", name:"王五", age: 19}
])

</script>

<style lang="css" scoped>
.container {
  background-color: cadetblue;
  width: 600px;
  height: 600px;
  border: 2px solid #333;
  box-shadow: 10px 5px 5px #444;
  margin: auto;
}
</style>

```

在子组件(`Person.vue`)中

```html title:componets/Person.vue
<template>
  <div class="person-box">
    <ul>
      <li v-for="p in persons" :key="p.id">{{p.name}} - {{p.age}}</li>
    </ul>
  </div>
</template>

<script setup lang="ts" name="Person">


// 如果需要，限制接收对象的类型
import  {type Persons} from "@/types";
// defineProps(["persons"]) // 只接收，不限父组件传递数据的类型

withDefaults(defineProps<{persons?:Persons}>(), {persons:()=>[{id:"1111", name: "王宝器", age:20}]}) // 限制父组件传递的数据类型, ? 表示非必要的


</script>

<style lang="css" scoped>
.person-box {
  background-color: skyblue;
  width: 600px;
  height: auto;
  border: 2px solid #333;
  box-shadow: 10px 5px 5px #444;
  margin: 100px auto;
}
</style>
```

## 生命周期

每个 `Vue` 组件实例在创建时都需要经历一系列的初始化步骤，比如设置好数据侦听，编译模板，挂载实例到 DOM，以及在数据改变时更新 DOM。在此过程中，它也会运行被称为**生命周期钩子的函数**，让开发者有机会在特定阶段运行自己的代码。

组件的生命周期的 $4$ 个阶段：**创建** **挂载** **更新** **销毁**。每个节点都有两个特定的钩子函数(`hooks`)。当组件到达特定的时刻时，就会调用这些 `hook` 函数

### 选项式

下图展示了各种生命周期钩子函数的执行时机(**选项式API**)

![[Pasted image 20240914214707.png]]

### 组合式 API

> [!tip] 组合式 API 没有创建前后的钩子，它们被 `setup` 替代了

```html
<template>
  <div class="person-box">

  </div>
</template>

<script setup lang="ts" name="Person">

import {onBeforeMount, onBeforeUnmount, onBeforeUpdate, onMounted, onUnmounted, onUpdated} from "vue";

// 注册 挂载前的 hook 函数
onBeforeMount(() => {

})

// 注册 挂载后的 hook 函数
onMounted(()=> {

})

// 注册 更新前的 hook 函数
onBeforeUpdate(() => {

})

// 注册 更新后的 hook 函数
onUpdated(() => {

})


// 注册 卸载前的 hook 函数
onBeforeUnmount(() => {

})

// 注册 卸载后的 hook 函数
onUnmounted(() => {

})

</script>

<style lang="css" scoped>
.person-box {
  background-color: skyblue;
  width: 600px;
  height: auto;
  border: 2px solid #333;
  box-shadow: 10px 5px 5px #444;
  margin: 100px auto;
}
</style>
```

> [!tip] root 组件最先开始解析，最后被挂载

### 自定义 hooks

在前面的示例中，我们的数据和方法全都写在了 `setup` 中，这导致了不同功能的数据和方法杂糅在一起。这就类似于 Vue2 那样

hooks 都是一个一个可以调用的函数，因此可以通过自定义 hooks 函数来封装与功能相关数据和方法

```ts title:hooks/useDog.ts
import {reactive} from "vue";
import axios from "axios";

export default function () {
    let apiUrl = "https://dog.ceo/api/breed/pembroke/images/random"

    let dogs = reactive([
        "https://images.dog.ceo/breeds/groenendael/n02105056_961.jpg"
    ])

    async function getDog() {
        try{
            let result = await axios.get(apiUrl)
            dogs.push(result.data.message)
        }catch(error){
            alert(error)
        }
    }
    return {dogs, getDog}
}
```

然后，在组件中使用自定义 hooks 即可

```html title:components/Dog.vue
<template>
  <div class="person-box">

    <ul>
      <li v-for="uri in dogs"><img :src="uri" alt="狗狗"></li>
    </ul>

    <button @click="getDog">获取狗狗</button>

  </div>
</template>

<script setup lang="ts" name="Dog">

import useDog from "@/hooks/useDog"; // 导入自定义的 hooks

const {dogs, getDog} = useDog(); // 调用函数hook函数

</script>

<style lang="css" scoped>
.person-box {
  background-color: lightslategray;
  width: 600px;
  height: auto;
  border: 2px solid #333;
  box-shadow: 10px 5px 5px #444;
  margin: 100px auto;
}

img {
  width: 200px;
  margin: auto;
}

button {
  width: 200px;
  margin-top: 20px;
  margin-left: 200px;
  color: lightcyan;
  background-color: lightslategray;
  border-radius: 2em;
}
</style>
```

> [!tip]
> 
> 通过自定义 hooks 可以将功能点相关的 **数据**，**方法**，**计算属性**，**hooks** 等全都集中在一起
> 
> 

## 模板语法

Vue 使用一种基于 HTML 的模板语法，使我们能够声明式地将其组件实例的数据绑定到呈现的 DOM 上。所有的 Vue 模板都是语法层面合法的 HTML，可以被符合规范的浏览器和 HTML 解析器解析。

在底层机制中，Vue 会将模板编译成高度优化的 JavaScript 代码。结合响应式系统，当应用状态变更时，Vue 能够智能地推导出需要重新渲染的组件的最少数量，并应用最少的 DOM 操作

### 文本插值

最基本的数据绑定形式是文本插值，它使用的是“Mustache”语法 (即 **双大括号**)：

```html
<span>Message: {{ msg }}</span>
```

双大括号标签会被替换为 **相应组件实例中 msg 属性的值**。同时每次 `msg` 属性更改时它也会同步更新。

双大括号中支持 JavaScript 表达式

```js
{{ number + 1 }}

{{ ok ? 'YES' : 'NO' }}

{{ message.split('').reverse().join('') }}
```

### v-html

双大括号会将数据解释为纯文本，而不是 HTML。若想 **插入 HTML**，你需要使用 **v-html 指令**：

```html
<p>Using text interpolation: {{ rawHtml }}</p>
<p>Using v-html directive: <span v-html="rawHtml"></span></p>
```

> [!tip] 指令
> 
> 这里我们遇到了一个新的概念。这里看到的 **`v-html` 属性** 被称为一个**指令**。**指令由 `v-*` 作为前缀，表明它们是一些 _由 Vue 提供的特殊属性_**，你可能已经猜到了，它们将为渲染的 DOM 应用特殊的响应式行为。这里我们做的事情简单来说就是：在当前组件实例上，将此元素的 `innerHTML` 与 `rawHtml` 属性保持同步。
> 

> [!attention] 
> 
> 在网站上动态渲染任意 HTML 是非常危险的，因为这非常容易造成 [XSS 漏洞](https://zh.wikipedia.org/wiki/%E8%B7%A8%E7%B6%B2%E7%AB%99%E6%8C%87%E4%BB%A4%E7%A2%BC)。请仅在内容安全可信时再使用 `v-html`，并且**永远不要**使用用户提供的 HTML 内容
> 

### 属性绑定

双大括号不能在 HTML 属性中使用。想要 **响应式地绑定一个 _属性值_**，应该使用 **v-bind 指令**：

```html
<div v-bind:id="dynamicId"></div>
```

`v-bind` 指令指示 Vue 将元素的 `id` 属性与组件的 `dynamicId` 属性保持一致。如果绑定的值是 `null` 或者 `undefined`，那么该 属性将会从渲染的元素上移除。

因为 `v-bind` 非常常用，Vue 提供了特定的 **简写** 语法：

```html
<div :id="dynamicId"></div>
```

#### 绑定多个值

如果你有像这样的一个包含多个属性的 JavaScript 对象

```js
const objectOfAttrs = {
  id: 'container',
  class: 'wrapper',
  style: 'background-color:green'
}
```

通过不带参数的 `v-bind`，你可以将它们绑定到单个元素上：

```html
<div v-bind="objectOfAttrs"></div>
```

#### 支持 JavaScript 表达式

 Vue 在所有的数据绑定中都支持完整的 JavaScript 表达式：
 
```html
<div :id="`list-${id}`"></div>
```


> [!tip]
> 
> 每个绑定仅支持 **单一表达式**，也就是 **一段能够被求值的 JavaScript 代码**。一个简单的判断方法是是否可以合法地写在 `return` 后面。
> 

### 事件绑定

JavaScript原生事件都是以 `on*` 开头的。而在 Vue 中，这些都被识别为文本。Vue 提供了一个 `v-on` 指令，用于绑定事件

```html
<a v-on:click="doSomething"> ... </a>

<!-- 简写 -->
<a @click="doSomething"> ... </a>
```

这里的参数是要监听的事件名称：`click`。`v-on` 有一个相应的缩写，即 `@` 字符

![[Pasted image 20240916174020.png]]

### Class 与 Style 绑定

**数据绑定的一个常见需求场景是操纵元素的 CSS class 列表和内联样式**。因为 `class` 和 `style` 都是 attribute，我们可以和其他 attribute 一样使用 `v-bind` 将它们和动态的字符串绑定。但是，在处理比较复杂的绑定时，通过拼接生成字符串是麻烦且易出错的。因此，Vue 专门为 `class` 和 `style` 的 `v-bind` 用法提供了特殊的功能增强。除了字符串外，表达式的值也可以是对象或数组。

参考 [class-and-style](https://cn.vuejs.org/guide/essentials/class-and-style.html)

### 条件渲染

| 指令          | 描述                                      |
| :---------- | :-------------------------------------- |
| `v-if`      | 内容只会在指令的表达式返回真值时才被渲染                    |
| `v-else`    | 为 `v-if` 添加一个 else 区域                   |
| `v-else-if` | 为 `v-if` 添加一个 else-if 区域                |
| `v-show`    | `v-show` 仅切换了该元素上名为 `display` 的 CSS 属性。 |

### 列表渲染


我们可以使用 `v-for` 指令基于一个数组来渲染一个列表。`v-for` 指令的值需要使用 `item in items` 形式的特殊语法，其中 `items` 是源数据的数组，而 `item` 是迭代项的**别名**：

```js
const items = ref([{ message: 'Foo' }, { message: 'Bar' }])
```

```html
<li v-for="item in items">
  {{ item.message }}
</li>
```

Vue 默认按照 **“就地更新”** 的策略来更新通过 `v-for` 渲染的元素列表。当数据项的顺序改变时，Vue 不会随之移动 DOM 元素的顺序，而是就地更新每个元素，确保它们在原本指定的索引位置上渲染

默认模式是高效的，但 **只适用于列表渲染输出的结果不依赖子组件状态或者临时 DOM 状态 (例如表单输入值) 的情况**

为了给 Vue 一个提示，以便它可以跟踪每个节点的标识，从而重用和重新排序现有的元素，你需要为每个元素对应的块提供一个唯一的 `key` 属性

```html
<div v-for="item in items" :key="item.id">
  <!-- 内容 -->
</div>
```
