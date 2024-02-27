# JS引入

- 内部引入: 在`<body>`标签后加入`<script>`标签，并在其中编写JS代码
- 外部引入: 类似.css文件的外部引入，注意`<script>`标签不能自闭和

# 语法规则

- 大小写严格
- 语句结尾不要求一定有分号
- 注释方式和java相同
- JS是**弱类型**的语言，同一变量可以承载不同类型的数据类型

# 变量

使用关键字`var `声明的变量的作用域是**全局的**，而在ES6中新增的`let`变量是局部的，只在当前代码块中有效，**且变量名不允许被重复声明**

JS中声明常量的关键字和go相同都是`const`

# 数据类型

基本类型:

- number : 数值类型
- string : 单双引号均可
- boolean
- null
- Undefined: 声明的变量未进行初始化时，默认值为undefined

## 类型转换

JS中有全局方法的概念，比如`parseInt`

**字符串转为数字类型:**

`parseInt()`: 其会将字符串中第一个非数字字符前的字符均转换为数值类型，对于无法转换的字符串则会返回`NaN`(not a number)

**其他类型转boolean:**

Number: 只有0和NaN会转为false，其余均为true

String: 空字符为false其余为true

Null和Undefined: 均为false

# 运算符

JS中有特殊的运算符:`===`

这是解释性语言所特有的，JS中使用`==`比较时，会首先判断比较的双方的数据类型是否一致，若不一致则会**自动转换成一致的类型后再进行比较**，而`===`**不会进行自动转换**，若类型不一致则直接返回`false`

# 函数

JS为弱类型语言，**在函数的形参列表中不需要指定参数的数据类型，返回值也不需要**

JS中的函数的声明也可以像go语言中的匿名函数一样:

```javascript
let add = function (a, b) {
    return a + b
}
console.log(add(2,3))
```

# 引用类型(对象)

## 数组

有两种声明方式:

1. 通过new关键字:

   ```javascript
   var array = new Array(1,2,3,4)
   ```

2. 简化声明:

   ```
   var array = [1,2,3,4]
   ```

访问直接使用索引即可，如array[2]

JS中的数组类似于Java中的ArrayList，**可以存储任意类型的数据，且可以自动扩缩容**

#### 数组遍历

for循环遍历:

```javascript
let array = [1,2,3,4]
for (let i = 0; i < array.length; i++) {
  console.log(array[i])
}
```

foreach遍历:

```javascript
let array = [1,2,3,4]
array.forEach(function (ele) {
  console.log(ele)
})
```

两者在遍历上是有区别的，**前者会遍历所有的元素，后者只遍历有值的元素**，比如若在索引10处增加一个元素`array[10] = 10`，那么前者会遍历到索引6~9处的元素即`undefined`，后者不会遍历到索引6~9

#### 追加元素

数组对象的有一原生方法`push(...items: T[])`，相当于go中的append，在数组尾部追加元素

#### 删除元素

原生方法`splice(start: number, deleteCount?: number)`

- start: 开始进行删除操作的索引处
- deleteCount: 删除元素数

## String

JS原生的SDK中的很多String中的方法和java中是很类似的，比如:

- `charAt()`: 获取指定索引处的字符
- `indexOf()`: 获取给定子串在字符串中第一次出现的位置
- `trim()`: 去除字符串两边的字符串
- `subString(start, end)`: 截取子串

## 自定义对象

JS中可以在不创建类的情况下**创建对象**，使用`var`或`let`关键字声明，在`{}`中声明对象的结构与信息，比如:

```javascript
let user = {
  name: "ryan",
  age: 20,
  gender: "male",
  run: function () {
    console.log(user.name + " is running")
  }
}

user.run()
```

## JSON对象

JSON(JavaScript Object Notation)，JSON是语法规则很大程度上参照了JavaScript，**主要的区别是JSON中的key都需要加上`""`**，如:

```javascript
let user = `{"name":"ryan", "age":18, "skills":["java", "golang", "python"]}`
```

- 数值类型: 数字即可
- 字符串: 用`""`包裹
- 布尔值: true、false
- 数组: 用`[]`包裹
- 对象: 用`{}`包裹
- null

### JSON转JS对象

通过`JSON.parse()`方法

```javascript
let user = `{"name":"ryan", "age":18, "skills":["java", "golang", "python"]}`
JSON.parse(user)
```

### JS对象转JSON

```javascript
let user = `{"name":"ryan", "age":18, "skills":["java", "golang", "python"]}`
JSON.parse(user)
let json = JSON.stringify(user)
```



# ES6新特性: 箭头函数

ES6在语法上提供了一个新特性用于简化匿名函数的声明:

```javascript
old: function()  new: () =>
```

比如:

```javascript
// old
array.forEach(function (ele) {
  console.log(ele)
})

// new
array.forEach((ele) => {
  console.log(ele)
}) 
```

# BOM

BOM(Brower Object Model)，JavaScript原生的将游览器中的各个组件以面向对象的理念封装好，可供用户对其进行操作

## Window

游览器窗口对象

主要的属性有(获取其他BOM对象):

- history: 对History对象的只读引用
- location: 对窗口或框架的location对象
- navigator: 对Navigator对象的只读引用

主要方法有:

- alert()

- confirm()

  ```javascript
  confirm(message?: string): boolean;
  ```

   提示框，常用于再一次确认用户的操作，防误触，其返回值是一个布尔值，用于接收用户的选择

  ``` javascript
  let decision = window.confirm("are you sure?")
  ```

- setInterval():

  ```javascript
  setInterval(handler: TimerHandler, timeout?: number, ...arguments: any[]): number;
  ```

  周期性的执行给定的函数

  ```javascript
  // 每2s给num+1
  let num = 0
  window.setInterval(function() {
    num++
  }, 2000)
  ```

- setTimeout():

  ```javascript
  setTimeout(handler: TimerHandler, timeout?: number, ...arguments: any[]): number;
  ```

  延迟给定时间后，执行一次给定的函数

  ```javascript
  window.setTimeout(function(){
    alert("time is up")
  }, 3000)
  ```

## Screen

屏幕对象

## Location

地址栏对象

主要的属性有:

- href: 设置或返回完整的URL

```javascript
// 自动跳转到给定域名
location.href = "www.baidu.com"
```

## History

历史记录对象

## Navigator

游览器对象，包括内核信息、版本信息等

# DOM

DOM(Document Object Model)，JavaScript将标记语言(如HTML)的各个组成部分进行封装，可供用户操作

通过DOM操作， 可以实现:

- 改变HTML中的元素(标签)的内容
- 改变HTML中的元素(标签)的样式(属性)
- 对HTML DOM事件做出反应
- 添加和删除HTML元素

## Document

文档对象，即整个HTML文档，通过**Window**对象获取

### 选取标签对象

Document提供了四个方法，供我们选择到指定的标签对象:

- 根据**id属性值**，返回标签对象:

  ```javascript
  let idObject = document.getElementById("1")
  ```

- 根据**标签名称**，返回标签**数组**:

  ```javascript
  let divs = document.getElementsByTagName("div")
  ```

- 根据**name属性值**，返回标签对象:

  ```javascript
  let nameObject = document.getElementsByName("username")
  ```

- 根据**class属性值**，返回标签**数组**:

  ```javascript
  let classObject = document.getElementsByClassName("cls")
  ```

## Element

元素对象，即各个标签，比如`<head>`、`<body>`、`<a>`等，`<html>`是根对象

## Attribute

属性对象，元素的属性，比如`<a>`标签的`<href>`属性

## Text

文本对象，展示的内容，比如`<p>hi, this is me</p>`中的`hi,this is me`即为文本对象

## Comment

注释对象



# 事件监听

事件是指**用户对HTML元素的操作**，比如**按钮被点击、鼠标悬浮在了某个元素上、键盘输入到某个元素中**等等

## 事件绑定

即将JS的交互逻辑和用户对某一标签的操作时间绑定在一起

1. 通过HTML中的**事件属性**进行绑定:

   ```html
   <input type="button" onclick="on()" value="button1">
   <script>
     function on() {
       console.log("I have been clicked")
     }
   </script>
   ```

2. 通过DOM元素属性绑定，类似匿名函数:

   ```html
   <input type="button" id="btn" value="button1">
   <script>
     document.getElementById("btn").onclick=function () {
       console.log("I have been clicked")
     }
   </script>
   ```

## 常见事件

鼠标事件：

- `click`：鼠标单击事件
- `dblclick`：鼠标双击事件
- `mousemove`：鼠标移动事件
- `mouseover`：鼠标移入元素事件
- `mouseout`：鼠标移出元素事件
- `mousedown`：鼠标按下事件
- `mouseup`：鼠标松开事件

键盘事件：

- `keydown`：按下键盘按键事件
- `keyup`：松开键盘按键事件
- `keypress`：按下并松开键盘按键事件

表单事件：

- `submit`：提交表单事件
- `input`：输入内容改变事件
- `change`：表单元素值改变事件
- `focus`：元素获得焦点事件
- `blur`：元素失去焦点事件

文档加载事件：

- `DOMContentLoaded`：HTML 文档解析完成事件
- `load`：页面和所有资源加载完成事件
- `unload`：页面关闭或离开事件

触摸事件：

- `touchstart`：触摸开始事件
- `touchmove`：触摸移动事件
- `touchend`：触摸结束事件

滚动事件：

- `scroll`：滚动事件

窗口事件：

- `resize`：窗口大小改变事件
- `scroll`：窗口滚动事件

# Vue

基于MVVM(Model-View-ViewModel)的理念，实现数据的**双向绑定**

![image-20231221155106840](/Users/steaksunflower/Library/Application Support/typora-user-images/image-20231221155106840.png)

## 插值表达式

vue中，可以使用`{{}}`双花括号，在html标签的“**内容区域**”中表现数据

```
{{表达式}}
```

表达式可以是:

- 变量
- 常量
- 算数运算符
- 比较运算符、逻辑运算符、三元运算符

```html
<div id="app">
    <p>{{ msg }}</p>						  <!--变量-->
    <p>{{ 500 }}</p>  					  <!--常量-->
    <p>{{ score + 10 }}</p> 				   <!--算术运算-->
    <p>{{ score > 10 }}</p> 				   <!--比较运算-->
    <p>{{ score > 80 && age > 18 }}</p>  	    <!--逻辑运算-->
    <p>{{ age > 18 ? '成年' : '少年' }}</p>    <!--三元运算-->
</div>

<script src="./vue.js"></script>

<script>
    var vm = new Vue({
        // delimiters:['$','#'],
        el:'#app',
        data:{
            msg:'数据',
            score: 90,
            age:20
        }
    })
</script>
```

## vue指令

引入vue.js后标签中会以`v`开头的属性，即为vue的指令

| 指令      | 描述                                                         |
| :-------- | :----------------------------------------------------------- |
| `v-if`    | 根据条件判断是否渲染元素，如果条件为真，则渲染元素；否则，从DOM中移除元素。 |
| `v-show`  | 根据条件判断是否显示元素，如果条件为真，则显示元素；否则，通过CSS设置元素的`display`属性为`none`。 |
| `v-for`   | 遍历数组或对象，渲染列表或生成重复的元素。                   |
| `v-bind`  | 动态绑定属性或响应式地更新元素的属性。                       |
| `v-on`    | 绑定事件监听器，用于触发指定的事件处理函数。                 |
| `v-model` | 在表单元素上双向绑定数据，实现数据的输入和响应。             |
| `v-text`  | 更新元素的文本内容，类似于使用`textContent`属性。            |
| `v-html`  | 更新元素的HTML内容，类似于使用`innerHTML`属性。              |
| `v-pre`   | 跳过元素和其子元素的编译过程，直接输出原始内容。             |
| `v-cloak` | 在Vue实例编译完成前，隐藏带有该指令的元素。                  |
| `v-once`  | 只渲染元素和组件一次，不再随数据的变化重新渲染。             |

### v-bind

绑定属性，动态响应式的更新属性

比如动态的绑定一个a标签的url:

```html
<a v-bind:href="url"></a>
```

动态绑定:

```javascript
new Vue({
  el: "#app", // 选择控制区域
  data: {
    url: ""
  }
})
```

v-bind可以简写为:

```html
<a :href="url"></a>
```

### v-model

动态的控制表单中的属性，是**双向的**(JS改变变量也会改变标签属性，用户改变标签属性也会改变JS中的变量)比如表单内容

```html
<input type="text" v-model="modelContent">
```

动态绑定:

```javascript
new Vue({
  el: "#app", // 选择控制区域
  data: {
    modelContent: "RYAN"
  }
})
```

### v-on

为标签绑定事件

```html
<input type="button" value="btn" v-on:click="hasClicked">
```

简化写法:

```html
<input type="button" value="btn" @click="hasClicked">
```

### v-if & v-show

v-if: 对于控制的元素，在表达式条件成立时渲染，否则不渲染

```html
<body>
  <div id="app">
    年龄 <input type="text" v-model="age">
    <span v-if="age <= 35">年轻人</span>
    <span v-else-if="age > 35 && age < 60">中年人</span>
    <span v-else>老年人</span>
  </div>
</body>
<script>
  new Vue({
    el: "#app", // 选择控制区域
    data: {
      age:20
    },
  })
</script>
```

v-show: 和v-if的区别是切换的是display属性的值

```html
<body>
  <div id="app">
    年龄 <input type="text" v-model="age">
    <span v-show="age <= 35">年轻人</span>
    <span v-show="age > 35 && age < 60">中年人</span>
    <span v-show="age >= 60">老年人</span>
  </div>
</body>
<script>
  new Vue({
    el: "#app", // 选择控制区域
    data: {
      age:20
    },
  })
</script>
```

v-show会在无论条件是否成立的情况下都将元素渲染出来，条件不成立的话其`display`属性的值为`none`

![image-20231224150420762](/Users/steaksunflower/Library/Application Support/typora-user-images/image-20231224150420762.png)

### v-for

遍历容器如数组的元素或对象的属性，类似python的for循环，有关键字`in`

```html
<body>
  <div id="app">
    <div v-for="addr in addrs">{{addr}}</div>
    <div v-for="(addr, index) in addrs">{{index+1}} : {{addr}}</div>
  </div>
</body>
<script>
  new Vue({
    el: "#app", // 选择控制区域
    data: {
      addrs:['BeiJing', 'ShangHai', 'GuangZhou', 'ShenZhen']
    },
  })
</script>
```

渲染效果:

```html
BeiJing
ShangHai
GuangZhou
ShenZhen
1 : BeiJing
2 : ShangHai
3 : GuangZhou
4 : ShenZhen
```

### 动态绑定展示实例

```html
<body>
  <div id="app">
    <table border="1" cellspacing="0" width="60%">
      <tr>
        <th>id</th>
        <th>name</th>
        <th>age</th>
        <th>gender</th>
        <th>score</th>
        <th>degree</th>
      </tr>
      <tr align="center" v-for="(user,index) in users">
        <td>{{index+1}}</td>
        <td>{{user.name}}</td>
        <td>{{user.age}}</td>
        <td>
          <span v-if="user.gender == 1">male</span>
          <span v-if="user.gender == 2">female</span>
        </td>
        <td>{{user.score}}</td>
        <td>
          <span v-if="user.score >= 90">A</span>
          <span v-if="user.score < 90 && user.score >= 80">B</span>
          <span v-if="user.score < 80 && user.score >= 60">C</span>
          <span v-if="user.score < 60" style="color: red">F</span>
        </td>
      </tr>
    </table>
  </div>
</body>
<script>
  new Vue({
    el: "#app", // 选择控制区域
    data: {
      users: [{
        name:"tom",
        age:21,
        gender:1,
        score:80,
      },{
        name:"rose",
        age:18,
        gender:2,
        score:85,
      },{
        name:"tony",
        age:30,
        gender:1,
        score:70,
      },{
        name:"jack",
        age:35,
        gender:1,
        score:55,
      }]
    },
  })
</script>
```

## vue生命周期

每触发一个生命周期事件，会自动执行一个生命周期方法(钩子)

| 生命周期钩子    | 含义                                                         |
| :-------------- | :----------------------------------------------------------- |
| `beforeCreate`  | 在实例被创建之初，数据观测 (data observer) 和事件/生命周期事件回调 (event/callbakc) 之前调用。在这个阶段，实例的属性和方法还不可用。 |
| `created`       | 实例已经创建完成，属性和方法已经设置好，但是DOM还没有生成，无法访问DOM。可以进行异步操作，如发送请求获取数据等。 |
| `beforeMount`   | 在模板编译/挂载之前调用。在这个阶段，Vue将编译模板生成了可渲染的函数，并准备把实例挂载到DOM上。 |
| `mounted`       | 实例被挂载到DOM后调用，此时可以访问DOM元素。通常用于初始化操作，如获取DOM元素、绑定事件等。 |
| `beforeUpdate`  | 数据更新时，在重新渲染之前调用。在这个阶段，可以在更新之前访问现有的DOM。 |
| `updated`       | 数据更新完成，DOM已经重新渲染。在这个阶段可以操作更新后的DOM，但要避免无限循环的更新。 |
| `beforeDestroy` | 实例销毁之前调用。在这个阶段，实例仍然完全可用，可以执行一些清理操作，如清除定时器、解绑事件等。 |
| `destroyed`     | 实例销毁后调用。在这个阶段，实例所有的指令已经解绑，事件监听已经移除，无法再访问实例的任何属性和方法。 |

通过重载vue生命周期中的钩子方法，可以在对象历经某个阶段时**自动触发**

其中常会被用户重载的方法是**mounted**，即对象挂载完成阶段，vue初始化成功，HTML页面渲染成功，此时可以通过重载的mounted方法自动发送请求到服务端加载数据

# Ajax

Ajax(Asynchronous JavaScript And XML)

通过Ajax可以完成:

- 和服务器交互: 向服务器发生请求并接收服务器相应的数据
- **异步**交互: 可以在**不重新加载整个页面的情况下**，与服务器交换数据并更新部分网页中的元素，如搜索框中内容的更新也会引起搜索联想的更新

## 原生Ajax请求

原生的Ajax请求是通过XMLHttpRequest对象完成的

```javascript
function loadDoc() {
  var xhttp = new XMLHttpRequest();
  xhttp.onreadystatechange = function() {
    if (this.readyState == 4 && this.status == 200) {
     document.getElementById("demo").innerHTML = this.responseText;
    }
  };
  xhttp.open("GET", "ajax_info.txt", true);
  xhttp.send();
} 
```

## Axios

Axios封装了原生Ajax，可以提升编码效率

https://www.axios-http.cn/docs/intro

引入:

```html
  <script src="https://cdn.jsdelivr.net/npm/axios/dist/axios.min.js"></script>
```

### 发送请求

```html
<body>
  <input type="button" value="get" onclick="get()">
</body>
<script>
  function get() {
    axios({
      method: "get",
      url: "",
    }).then(result => {
      console.log(result.data)
    })
  }
</script>
```

可以简写为:

```javascript
axios.get("").then((result) => {
  console.log(result.data)
})
```

# Vue脚手架vue-cli

vue-cli是vue提供的脚手架可以帮助快速生成开发模板

## 启动命令

```shell
npm run serve
```

## 配置端口

在vue.config.js中，增加属性`devServer`，并设置`port`

```javascript
const { defineConfig } = require('@vue/cli-service')
module.exports = defineConfig({
  transpileDependencies: true,
  devServer: {
    port: 7000
  }
})
```

## vue组件

vue组件以.vue结尾，通过`export`关键字导出，每个组件由三个部分组成: `<template>`、`<script>`、`<style>`

- `<template>`: 模板部分，即HTML标签
- `<script>`: 控制模板的数据来源及行为
- `<style>`: css样式

# element组件库

在main.js中引入element:

```javascript
import Vue from 'vue'
import ElementUI from 'element-ui'
import 'element-ui/lib/theme-chalk/index.css'
import App from './App.vue'
import router from "@/router";

Vue.config.productionTip = false
Vue.use(ElementUI)

new Vue({
  router,
  render: h => h(App)
}).$mount('#app')
```

## 创建element组件

在`./src/views`下创建目录`element`，用于存放element组件

在新创建目录下创建`ElementVue.vue`，注意新组件名要使用**大驼峰**，否则会报错

后再`ElementVue.vue`中粘贴组件代码即可:

```html
<template>
  <div>
      <el-row>
          <el-button>默认按钮</el-button>
          <el-button type="primary">主要按钮</el-button>
          <el-button type="success">成功按钮</el-button>
          <el-button type="info">信息按钮</el-button>
          <el-button type="warning">警告按钮</el-button>
          <el-button type="danger">危险按钮</el-button>
      </el-row>
  </div>
</template>

<script>
export default {
    name: "elementVue"
}

</script>

<style>

</style>
```

## 展示element组件

在`App.vue`中的js部分引入ElementVue.vue的路径，并在componts属性中引入，即可在模板中使用`element-view`标签展示element组件

```vue
<template>
  <div>
<!--    <h1>{{message}}</h1>-->
      <element-view>

      </element-view>
  </div>
</template>

<script>
  import ElementView from './views/element/ElementVue.vue'
  export default {
      components : {ElementView},
      data() {
          return {
              message: "hello vue~"
          }
      },
      methods: {

      }
  }
</script>

<style>

</style>
```

## Table组件

在粘贴模版的代码时，需要注意模板中的标签中的属性有哪些，并把相应的属性的js代码也粘贴上

模板层:

```html
<!--            可见有属性data-->
						<el-table
                :data="tableData"
                border
                style="width: 100%">
            <el-table-column
                    prop="date"
                    label="日期"
                    width="180">
            </el-table-column>
            <el-table-column
                    prop="name"
                    label="姓名"
                    width="180">
            </el-table-column>
            <el-table-column
                    prop="address"
                    label="地址">
            </el-table-column>
        </el-table>
```

控制层:

```javascript
    data() {
        return{
            tableData: [{
                date: '2016-05-02',
                name: '王小虎',
                address: '上海市普陀区金沙江路 1518 弄'
            }, {
                date: '2016-05-04',
                name: '王小虎',
                address: '上海市普陀区金沙江路 1517 弄'
            }, {
                date: '2016-05-01',
                name: '王小虎',
                address: '上海市普陀区金沙江路 1519 弄'
            }, {
                date: '2016-05-03',
                name: '王小虎',
                address: '上海市普陀区金沙江路 1516 弄'
            }]
        }
    }
```

## 分页组件

```html
        <el-pagination
            background
            layout="prev, pager, next"
            :total="1000">
        </el-pagination>
```

其中layout是控制是否显示上一页、下一页按钮、页数跳转等

增加事件:

```html
        <el-pagination
            background
            layout="prev, pager, next"
            @size-change="handleSizeChange"
            @current-change="handleCurrentChange"
            :total="1000">
        </el-pagination>
```

两个事件分别是在页数变化和当前页数变化时被触发，需要在控制层中编写相应的事件函数:

```javascript
    methods: {
        handleSizeChange: function (val) {
            alert("每页记录数变化" + val)
        },

        handleCurrentChange: function (val) {
            alert("页码发送变化" + val)
        }
    }
```

## 对话框组件

```html
        <el-button type="text" @click="dialogTableVisible = true">打开嵌套表格的 Dialog</el-button>

        <el-dialog title="收货地址" :visible.sync="dialogTableVisible">
            <el-table :data="gridData">
                <el-table-column property="date" label="日期" width="150"></el-table-column>
                <el-table-column property="name" label="姓名" width="200"></el-table-column>
                <el-table-column property="address" label="地址"></el-table-column>
            </el-table>
        </el-dialog>
```

其中有数据模型`gridData`和事件`dialogTableVisible`:

```javascript
            gridData: [{
                date: '2016-05-02',
                name: '王小虎',
                address: '上海市普陀区金沙江路 1518 弄'
            }, {
                date: '2016-05-04',
                name: '王小虎',
                address: '上海市普陀区金沙江路 1518 弄'
            }, {
                date: '2016-05-01',
                name: '王小虎',
                address: '上海市普陀区金沙江路 1518 弄'
            }, {
                date: '2016-05-03',
                name: '王小虎',
                address: '上海市普陀区金沙江路 1518 弄'
            }],
            dialogTableVisible: false,
```

## form表单组件

下述示例是一个在Dialog中的form表单:

```html
        <el-button type="text" @click="dialogTableVisible = true">打开Form的Dialog</el-button>

        <el-dialog title="Form表单" :visible.sync="dialogTableVisible">
            <el-form ref="form" :model="form" label-width="80px">
                <el-form-item label="活动名称">
                    <el-input v-model="form.name"></el-input>
                </el-form-item>

                <el-form-item label="活动区域">
                    <el-select v-model="form.region" placeholder="请选择活动区域">
                        <el-option label="区域一" value="shanghai"></el-option>
                        <el-option label="区域二" value="beijing"></el-option>
                    </el-select>
                </el-form-item>

                <el-form-item label="活动时间">
                    <el-col :span="11">
                        <el-date-picker type="date" placeholder="选择日期" v-model="form.date1" style="width: 100%;"></el-date-picker>
                    </el-col>
                    <el-col class="line" :span="2">-</el-col>
                    <el-col :span="11">
                        <el-time-picker placeholder="选择时间" v-model="form.date2" style="width: 100%;"></el-time-picker>
                    </el-col>
                </el-form-item>

                <el-form-item>
                    <el-button type="primary" @click="onSubmit">提交</el-button>
                    <el-button>取消</el-button>
                </el-form-item>

            </el-form>
        </el-dialog>
```

其通过v-model绑定了`form`这一数据模型:

```javascript
            form: {
                name: '',
                region: '',
                date1: '',
                date2: '',
            },
```

还有一个提交按钮，触发`onSubmit()`事件:

```javascript
        onSubmit() {
            alert(JSON.stringify(this.form));
        }
```

## 组件示例

```html
<template>
    <el-container style="height: 700px; border: 1px solid #eee">
        <el-header style="font-size: 40px; background-color: rgb(238, 241, 246)">智能学习辅助系统</el-header>
        <el-container>
            <el-aside width="200px" style="border: 1px solid #eee">
                <el-menu :default-openeds="['1', '3']">
                    <el-submenu index="1">
                        <template slot="title"><i class="el-icon-message"></i>系统信息管理</template>
                        <el-menu-item-group>
                            <el-menu-item index="1-1">部门管理</el-menu-item>
                            <el-menu-item index="1-2">员工管理</el-menu-item>
                        </el-menu-item-group>
                    </el-submenu>
                </el-menu>
            </el-aside>

            <el-main>

                <el-form :inline="true" :model="formInLine" class="demo-form-inline">
                    <el-form-item label="姓名">
                        <el-input v-model="formInLine.name" placeholder="姓名"></el-input>
                    </el-form-item>
                    <el-form-item label="性别">
                        <el-select v-model="formInLine.gender" placeholder="性别">
                            <el-option label="男" value="1"></el-option>
                            <el-option label="女" value="2"></el-option>
                        </el-select>
                    </el-form-item>

                    <el-form-item label="入职日期">
                        <span class="demonstration"></span>
                        <el-date-picker v-model="formInLine.entrydate" placeholder="入职日期"
                            type="daterange"
                            range-separator="至"
                            start-placeholder="开始日期"
                            end-placeholder="结束日期">
                        </el-date-picker>
                    </el-form-item>

                    <el-form-item>
                        <el-button type="primary" @click="onSubmit">查询</el-button>
                    </el-form-item>
                </el-form>

                <el-table :data="tableData" border>
                    <el-table-column prop="name" label="姓名" width="120">
                    </el-table-column>
                    <el-table-column prop="image" label="图片" width="120">
                        <template slot-scope="scope">
                            <img :src="scope.row.image" width="100px" height="70px" alt="">
                        </template>
                    </el-table-column>
                    <el-table-column prop="gender" label="性别" width="120">
                        <template slot-scope="scope">
                            {{scope.row.gender == 1 ? "男": "女"}}
                        </template>
                    </el-table-column>
                    <el-table-column prop="position" label="职位" width="120">
                    </el-table-column>
                    <el-table-column prop="entrydate" label="入职日期" width="120">
                    </el-table-column>
                    <el-table-column prop="updatetime" label="最后更新时间" width="120">
                    </el-table-column>
                    <el-table-column label="操作">
                        <el-button type="primary" size="mini">编辑</el-button>
                        <el-button type="danger" size="mini">删除</el-button>
                    </el-table-column>
                </el-table>

                <br>

                <el-pagination
                    background
                    layout="prev, pager, next"
                    @size-change="handleSizeChange"
                    @current-change="handleCurrentChange"
                    :total="1000">
                </el-pagination>

            </el-main>

        </el-container>
    </el-container>
</template>

<script>
import axios from "axios"

export default {
    data() {
        return {
            tableData: [],
            formInLine: {
                name: "",
                gender: "",
                entrydate: [],
            }
        }
    },

    methods: {
        onSubmit: function () {
            alert("queryData")
        },

        handleSizeChange: function (val) {
            alert("每页记录数变化" + val)
        },

        handleCurrentChange: function (val) {
            alert("页码发送变化" + val)
        },
    },

    mounted() {
        // 发送异步请求
        // https://yapi.smart-xwork.cn/mock/169327/emp/list
        axios.get("").then((result) => {
            this.tableData = result.data.data
        })
    }
}
</script>

<style scoped>

</style>
```

![image-20231227195642212](/Users/steaksunflower/Library/Application Support/typora-user-images/image-20231227195642212.png)

# Vue路由

前端中的路由是指**URL中的hash(#号)与组件之间的对应关系**

vue官方提供了Vue Router来实现前端路由，主要有三部分:

- VueRouter: 路由器类，根据路由请求在路由视图中动态渲染选中的组件
- `<router-link>`: 请求链接组件，游览器会解析成`<a>`
- `<router-view>`: 动态视图组件，用来渲染与路由路径对应的组件

譬如，当URL的以`/user`结尾，游览器请求和渲染的就是`UserView.vue`中的内容

当在使用vue脚手架创建项目时，如果勾选了router功能，则会在`src/router`目录下生成`index.js`，其中包含了路由表，如在访问`/emp`时，就请求和渲染`EmpView.vue`组件:

```js
const routes = [
  {
    path: '/emp',
    name: 'emp',
    component: () => import(/* webpackChunkName: "about" */ '../views/tlias/EmpView.vue')
  },
  {
    path: '/',
    redirect: '/emp'
  }
]
```

在侧边栏中配置路由信息:

```html
<el-menu-item index="1-1">
    <router-link to="/dept">部门管理</router-link>
</el-menu-item>
<el-menu-item index="1-1">
    <router-link to="/emp">员工管理</router-link>
</el-menu-item>        
```

此时，在`App.vue`中，我们就不需使用组件的便签，直接在希望组件展示的区域使用`<router-view>`即可:

```html
<template>
  <div>
<!--    <emp-view>-->

<!--    </emp-view>-->
      <router-view></router-view>
  </div>
</template>

<script>
  export default {
      
  }
</script>

<style>

</style>
```

# Vue打包

实际项目中，前后端的工程文件会部署在不同的服务器上，前端常见的部署服务器是Nginx

vue中的打包命令:

```shell
vue-cli-service build
```

打包完成后会在当前项目主目录生成新目录`dist`，其中有子目录`css`、`fonts`、`js`

## nginx部署vue项目

1. 官网安装nginx压缩包并解压

2. 下载[PRCE库](https://sourceforge.net/projects/pcre/files/)用于后续编译时所用的工具，并放在和nginx同级的目录中

3. 把dist目录中的文件移动到nginx目录下的html目录中

4. 在nginx解压后的目录下编译:

   ```shell
   sudo ./configure --with-pcre=../pcre-8.45/ 
   
   sudo make
   
   sudo make install
   ```

5. 加入环境变量:

   ```shell
   vim ~/.zshrc
   
   export PATH="/usr/local/nginx/sbin:$PATH"
   
   source ~/.zshrc
   ```

5. 启动服务

   ```shell
   sudo ./nginx
   ```

之后访问nginx所提供服务的端口即可访问vue项目



上面的不一定对，2.2更新

1. 寻找nginx目录

   ```shell
   where nginx
   ```

2. 把dist目录中的文件移动到nginx目录下的html目录中

3. 复制一份nginx.conf文件

4. 进入sbin目录启动服务



nginx可以

1. nginx可以缓存，如果用户请求的是在缓存未过期的时间内的同一数据，则无需访问后端服务器，可以直接把缓存返回给用户
2. 进行负载均衡
3. 提高安全性，使得请求不直接进入到服务器中，而是先到nginx，nginx再在内网中进行转发

开启反向代理:

```.conf
# 反向代理,处理管理端发送的请求
location /api/ {
proxy_pass   http://localhost:8080/admin/;
    #proxy_pass   http://webservers/admin/;
}

# 反向代理,处理用户端发送的请求
location /user/ {
    proxy_pass   http://webservers/user/;
}
```



| 负载均衡策略         | 含义                                                         |
| :------------------- | :----------------------------------------------------------- |
| Round Robin          | 轮询策略，按顺序将请求分配给后端服务器，循环往复。每个后端服务器依次接收到请求，实现请求的均衡分发。 |
| Least Connections    | 最小连接数策略，将请求分配给当前连接数最少的后端服务器。这样可以确保负载相对均衡，将请求发送到连接数较少的服务器，避免过载。 |
| IP Hash              | IP 哈希策略，通过对客户端 IP 地址进行哈希运算，将同一 IP 的请求发送到同一个后端服务器。这样可以实现会话保持，确保来自同一客户端的请求都由同一台服务器处理。 |
| Generic Hash         | 通用哈希策略，通过对请求的某个标识符进行哈希运算，将具有相同标识符的请求发送到同一个后端服务器。这样可以实现自定义级别的请求分发，并确保相同标识符的请求都由同一台服务器处理。 |
| Random               | 随机策略，随机选择一个后端服务器来处理请求。随机性可以带来一定程度的负载均衡，但无法保证每个服务器的负载均衡性。 |
| Weighted Round Robin | 加权轮询策略，根据后端服务器的权重进行轮询分发。权重越高的服务器，每轮接收到的请求数量越多。这样可以根据服务器的性能和能力进行请求分发。 |
