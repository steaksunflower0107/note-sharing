# HTML标签

## 标签汇总

| 标签          | 功能说明                                               |
| :------------ | :----------------------------------------------------- |
| `<html>`      | 定义 HTML 文档的根元素                                 |
| `<head>`      | 定义文档的头部，包含了文档的元信息和引用的外部资源     |
| `<title>`     | 定义文档的标题，显示在浏览器的标题栏或页签上           |
| `<body>`      | 定义文档的主体部分，包含要显示在浏览器窗口中的内容     |
| `<h1>`-`<h6>` | 定义标题，`<h1>` 为最高级别标题，`<h6>` 为最低级别标题 |
| `<p>`         | 定义段落                                               |
| `<br>`        | 在文本中插入换行符                                     |
| `<a>`         | 定义超链接，用于创建指向其他网页、文件、位置的链接     |
| `<img>`       | 插入图像                                               |
| `<ul>`        | 定义无序列表                                           |
| `<ol>`        | 定义有序列表                                           |
| `<li>`        | 定义列表项                                             |
| `<table>`     | 定义表格                                               |
| `<tr>`        | 定义表格的行                                           |
| `<td>`        | 定义表格的单元格                                       |
| `<div>`       | 定义文档中的一个块级区域，用于组合其他 HTML 元素       |
| `<span>`      | 定义文档中的一个行内区域，用于组合其他 HTML 元素       |
| `<form>`      | 定义表单，用于用户输入和提交数据                       |
| `<input>`     | 定义输入字段，如文本输入框、复选框、单选按钮等         |
| `<button>`    | 定义按钮                                               |
| `<select>`    | 定义下拉列表框                                         |
| `<option>`    | 定义下拉列表框中的选项                                 |
| `<textarea>`  | 定义多行文本输入框                                     |
| `<label>`     | 定义表单元素的标签                                     |
| `<header>`    | 定义文档或区块的页眉部分                               |
| `<footer>`    | 定义文档或区块的页脚部分                               |
| `<nav>`       | 定义导航链接的部分                                     |
| `<aside>`     | 定义内容之外的内容，如侧边栏                           |
| `<section>`   | 定义文档中的一个区块，通常具有相关的主题               |
| `<article>`   | 定义独立的文章内容                                     |
| `<iframe>`    | 定义内联框架，用于在页面中嵌入其他页面或文档           |
| `<audio>`     | 定义音频播放器                                         |
| `<video>`     | 定义视频播放器                                         |
| `<canvas>`    | 定义绘图容器，用于绘制图形、动画等                     |
| `<script>`    | 定义客户端脚本，通常用于添加交互行为和动态内容         |
| `<style>`     | 定义样式信息，通常用于设置元素的外观和布局             |
| `<link>`      | 定义外部资源的链接，如 CSS 样式表、图标等              |
| `<meta>`      | 定义文档的元信息，如字符编码、关键词等                 |

## 图片标签`<img>`

img标签的常见属性有:

- src: 图片的源路径(使用相对路径时，./表示当前目录，../表示当前目录的父目录，以此类推)
- width: 图片宽度
- height: 图片高度

对于长宽，可以传入的参数有:

1. px: 如`width = 300px`，表示图片的宽占用300个像素的位置
2. %: 如`width = 40%`，表示占用父元素的空间比

设置长宽时，推荐的做法是**只设置其中一个参数**，由此另一参数会根据比例缩放，避免图片变形

## 超链接标签`<a>`

```html
<a href="" target=""></a>
```

其主要属性是:

- href: 指定资源的URL
- target: 指定用户点击超链接后如何打开链接网页，其有两个常见值:
  - _self: 默认值，在当前打开
  - _blank: 在新的空白页打开

## 视频标签`<video>`

```html
<video src=""></video>
```

其主要属性是:

- src: 视频的源URL
- controls: 显示播放控件
- width: 宽度
- height: 长度

## 表格标签`<table>`

```html
<table> </table>
```

页面表格可以通过`<table>`标签展示，其主要属性为:

- border: 指定表格边框的宽度
- width: 指定表格的宽度
- cellspacing: 单元格之间分割线的宽度

表格行可通过`<tr>`标签展示，可以包裹多个单元格`<td>`

如果是**表头**单元格，可以替换为`<th>`，会有**加粗居中的效果**

```html
<table border="2px" width="600px">
  <tr>
    <th>id</th>
    <th>name</th>
    <th>gender</th>
    <th>age</th>
  </tr>
  <tr>
    <td>1</td>
    <td>ryan</td>
    <td>male</td>
    <td>20</td>
  </tr>
  <tr>
    <td>2</td>
    <td>rose</td>
    <td>female</td>
    <td>18</td>
  </tr>
  <tr>
    <td>3</td>
    <td>curry</td>
    <td>male</td>
    <td>35</td>
  </tr>
</table>
```

## 表单标签`<form>`

```html
<form action="" method=""></form>
```

相对于表格可以实现**用户输入数据的采集**

其有两个重要属性:

- action: 表单数据的提交处URL，若不指定则提交到当前页面
- method: 表单提交时所发起的请求方式，如GET/POST，主要的区别是提交数据的长度限制，默认是GET

### 表单项标签

常见的表单项标签有`<input>`、`<select>`、`<textarea>`等

input即是用户输入的载体，其属性**type**可指定其具体的功能:

| `type` 值        | 功能说明                         |
| :--------------- | :------------------------------- |
| `text`           | 创建单行文本输入框               |
| `password`       | 创建密码输入框，输入内容会被隐藏 |
| `checkbox`       | 创建复选框                       |
| `radio`          | 创建单选按钮                     |
| `submit`         | 创建提交按钮                     |
| `reset`          | 创建重置按钮                     |
| `button`         | 创建普通按钮                     |
| `file`           | 创建文件上传字段                 |
| `hidden`         | 创建隐藏字段                     |
| `image`          | 创建图像提交按钮                 |
| `date`           | 创建日期选择框                   |
| `time`           | 创建时间选择框                   |
| `datetime`       | 创建日期时间选择框               |
| `datetime-local` | 创建本地日期时间选择框           |
| `month`          | 创建月份选择框                   |
| `week`           | 创建周选择框                     |
| `number`         | 创建数值输入框                   |
| `range`          | 创建滑动条                       |
| `email`          | 创建邮箱输入框                   |
| `url`            | 创建 URL 输入框                  |
| `tel`            | 创建电话号码输入框               |
| `search`         | 创建搜索框                       |
| `color`          | 创建颜色选择框                   |

select提供包含若干选项的下拉框供用户选择:

```html
<select name="degree">
  <option value="">---choose your degree---</option>
  <option value="1">本科</option>
  <option value="2">硕士</option>
  <option value="3">博士</option>
</select>
```

textarea可收集大容量的文本:

```html
<textarea name="description" cols="30" rows="10"></textarea>
```

其中**cols设置每行的字符数上限，rows为行数上限**

## JS引入`<script>`标签

`<script>`可以被写在HTML文档中的任何地方，也可以被写任意次，每一次都是独立的运行事件

**一般会将该标签置于`<body>`元素的底部**，该方式可以改善显示速度

#  CSS样式表

## CSS引入

HTML中的元素被CSS修饰的引入方式有三种

- 行内样式: 将css样式写在标签的**style**属性中，只会对当前标签生效，故复用性不佳

  ```html
  <h1 style="color: red">
    Breaking news!
  </h1>
  ```

- 内嵌样式(标签选择器): 写在style标签中，style标签可以写在HTML文本中的任何位置，但规范的写法是写在head标签中

  ```html
  <head>
    <meta charset="UTF-8">
    <title>Sina News</title>
    <style>
        h1 {
            color: red;
        }
    </style>
  </head>
  ```

- 外联样式: 将css样式写在.css文件中，并在被修饰的标签中通过`<link>`标签引入

  ```html
  <head>
    <meta charset="UTF-8">
    <title>Sina News</title>
    <link rel="stylesheet" href=".css文件路径">
  </head>
  ```

## 选择器

选择器的设计目标是在使用内联样式引入css时，可以灵活的对同一类标签或其中的不同元素进行修饰，写在`<style>`标签中

- **元素选择器**

  对某一种标签的所有元素进行修饰，如:

  ```css
  h1 {
      color: darkorchid;
  }
  ```

  则该HTML文本中的所有h1标签都会被修饰

- **id选择器**

  通过指定标签的id以精准的对某个**唯一的**标签进行修饰

  如有一标签的id为`new`:

  ```html
  <h1 id="new"> Breaking news! </h1>
  ```

  通过id选择器选择并指定颜色:

  ```html
  <style>
      #new {
        color: red;
      }
  </style>
  ```

- **类选择器**

  对某个自定义类标记过的标签进行修饰

  如有两个标签均属于`info`类:

  ```html
  <span class="info">2023.12.17 15:04:05</span> <span class="info">ESPN News</span>
  ```

  通过类选择器选择并指定颜色:

  ```html
  <style>
      .info {
        color: red;
      }
  </style>
  ```

同一个标签在同时被三种选择器修饰时，优先级是:

**id选择器 > 类选择器 > 元素选择器**

## 基础属性

### 颜色

颜色使用关键字`color  `指定

颜色表示方式有:

- 预定义的颜色名: 

  ``` css
  color: red"
  ```

- rgb表示: 即red、green、blue三原色，每项取值范围为0~255

  ```css
  color: rgb(255,0,0)
  ```

- 十六进制表示，以`#`开头，从左往右每两位分别设置的是红、绿、蓝，0最小f最大，当控制同一颜色的位为00或ff时可以简写为0或f:

  ```css
  color: #ffff00
  ```

### 字体

#### 字体大小

使用关键字`font-size`指定

```css
span {
  color: #968D92;
  font-size: 13px;
}
```

可以使用像素单位，也可以使用一些预制的大小如`large`、`medium`等

#### 字体修饰

使用关键字`text-decoration`指定

可用于指定颜色、是否有下划线等，常用于去除`<a>`标签中自带的下划线:

```css
#sina {
  color: red;
  text-decoration: none;
}
```

### 首行缩进&对齐

关键字`text-indent`设置首行缩进，设置值可以为像素值

关键字`text-align`设置对齐方式，`left`、`right`、`mid`分别为居左、居右、居中对齐

## 盒子模型

所谓盒子模型（Box Model）就是把HTML页面中的元素看作是一个矩形的盒子，也就是一个盛装内容的容器。每个矩形都是由元素的**内容（content）、内边距（padding）、边框（border）和外边距（margin）**组成，其中**外边距是不在盒子之内的**

这也是CSS的原生属性也是根据盒子模型的设计理念设定的

![image-20231218151309870](/Users/steaksunflower/Library/Application Support/typora-user-images/image-20231218151309870.png)

在设置一个盒子模型中的原生部分比如padding时，设置的先后次序是**从top开始，逆时针依次设定**:

```css
div {
  width: 200px;
  height: 200px;
  box-sizing: border-box;
  background-color: #999999;

  padding: 20px 40px 20px 40px; /* 内边距 上 右 下左 */
  border: 10px solid; /* 边框，宽度 线条类型 颜色 */
  margin: 30px; /* 外边框 */
}
```

盒子模型的实践会使用到大量的无语义的`<div>`和`span`标签，即**布局标签**

### 块级标签`<div>`

`<div>`具有以下特性:

- 一个div标签即会占据页面中的一行
- 标签所占的宽度默认是父元素的宽度，高度由实际承载的内容决定
- 可设置width与height

### 行级标签`<span>`

`<span>`具有以下特性:

- 一行中可有多个span标签
- 宽度与高度均是由承载的内容决定的
- 不可设置width与height

