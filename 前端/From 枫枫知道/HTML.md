# HTML

HyperText  Markup Language 超文本编辑语言

> 非编程语言 --- 标记语言



大多有模板，没有的使用 ” ！ “ 可以自动创建

其中head标签被称为网页的头部，title标签是网页的标题，body是网页可见的内容部分



### 标签

双标签

```HTML
<body></body>
```

单标签

```HTML
<br/>
```

### 注释

```html
<!--注释-->
```





## 常用标签

### 块级标签

- 独占一行
- 可以设置宽度，高度，margin,padding
- 宽度默认所在容器的宽度

| 标签    | 作用                 |
| ------- | -------------------- |
| table   | 定义表格             |
| h1 ~ h6 | 定义标题             |
| hr      | 定义一条水平线       |
| p       | 定义段落             |
| li      | 标签定义列表项目     |
| ul      | 定义无序列表         |
| ol      | 定义有序列表         |
| div     | 定义文档中的分区或节 |
| form    | 创建 HTML 表单       |

#### table

thead

tr

th

tbody

td

属性放在table中

```html
<!--align属性名，属性值为left center right，规定表格相对周围元素的对齐方式-->
<!--border属性名，属性值1或"",规定表格是否有边框，默认为""，表示没有边框-->
<!--cellpadding属性名，属性值像素值,单元格内容和边框之间的空白，默认为1-->
<!--cellspacing属性名，属性值像素值,单元格和单元格之间的距离，默认为2-->
<!--width属性名，属性值像素值或百分比,规定表格的宽度-->
<!--height属性名，属性值像素值或百分比,规定表格的高度-->
```

### 列表标签

**ol** order lists

**ul** unordered lists

**li** list item

```html
<ul>
    <!--无序标签-->
    <li>一条</li>
    <li>一条</li>
</ul>
```

### 

```html
<ol>
    <!--有序标签-->
    <li>第一条</li>
    <li>第二条</li>
</ol>
```

### form标签

<form action="/xxx" method="get" name="form1">    <input value="提交" type="submit"/>    <input value="重置" type="reset"/> </form>

## 行内标签

```html
<a href="CSS.md">块级标签</a>
```

<a href="CSS.md">块级标签</a>

- 与其他行内元素并排
- 设置宽高无效
- 默认的宽度就是文本内容的宽度
- 水平方向的 padding 和 margin 属性可以使用
- 只能容纳文本级元素和内联元素

| 标签     | 作用                  |
| -------- | --------------------- |
| a        | 标签定义超链接        |
| span     | 组合文档中的行内元素  |
| br       | 定义换行              |
| b        | 定义字体缩写          |
| label    | 标签                  |
| 表单标签 | input textarea select |
| img      | 图片                  |

### a标签

所谓的超链接是指从一个网页指向一个目标的连接关系，这个目标可以是另一个网页，也可以是相同网页上的不同位置，还可以是一个图片，一个电子邮件地址，一个文件，甚至是一个应用程序。

属性：

target：

- _blank表示在新标签页中打开目标网页
- _self表示在当前标签页中打开目标网页

download：用于下载

### input标签

| type属性值 | 表现形式     | 对应代码           |
| ---------- | ------------ | ------------------ |
| text       | 单行输入文本 | <input type=text"> |
| password   | 密码输入框   |                    |
| date       | 日期输入框   |                    |
| checkbox   | 复选框       |                    |
| radio      | 单选框       |                    |
| submit     | 提交按钮     |                    |
| reset      | 重置按钮     |                    |
| button     | 普通按钮     |                    |
| hidden     | 隐藏输入框   |                    |
| file       | 文本选择框   |                    |

属性说明：

- name：表单提交时的“键”，注意和id的区别
- value：表单提交时对应项的值
  - type="button", "reset", "submit"时，为按钮上显示的文本内容
  - type="text","password","hidden"时，为输入框的初始值
  - type="checkbox", "radio", "file"，为输入相关联的值
- checked：radio和checkbox默认被选中的项
- readonly：text和password设置只读
- disabled：禁用，所有input均适用

label标签中的for属性会和input中的id关联

```HTML
<label for="user">
    用户名
</label>
<input placeholder="请输入用户名" id="user">
```

## span和div
https://blog.csdn.net/Obito_TXP/article/details/120106931

# 绝对路径 相对路径

"/"开头的是绝对路径

**文件的相对路径和绝对路径**
相对路径

```
<img src="./avatar.png" alt="">
<img src="avatar.png" alt="">
```
绝对路径
```
<img src="G:\IT\前端项目\qianduan_study\html\avatar.png" alt="">
```
**web中相对路径和绝对路径**

```
相对路径
<form action="xxx">
    <input type="submit" value="提交">
</form>

绝对路径
<form action="/xxx">
    <input type="submit" value="提交">
</form>
```
相对路径
```
http://localhost:63342/qianduan_study/html/4.%E7%9B%B8%E5%AF%B9%E8%B7%AF%E5%BE%84%E5%92%8C%E7%BB%9D%E5%AF%B9%E8%B7%AF%E5%BE%84.html
提交 地址 变成了
http://localhost:63342/qianduan_study/html/xxx?
```
绝对路径
```
http://localhost:63342/qianduan_study/html/4.%E7%9B%B8%E5%AF%B9%E8%B7%AF%E5%BE%84%E5%92%8C%E7%BB%9D%E5%AF%B9%E8%B7%AF%E5%BE%84.html
提交 地址 变成了
http://localhost:63342/xxx?
```