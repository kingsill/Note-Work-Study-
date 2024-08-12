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

