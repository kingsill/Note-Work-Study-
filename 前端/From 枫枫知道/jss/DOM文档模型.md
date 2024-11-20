## HTML DOM (文档对象模型)

当网页被加载时，浏览器会创建页面的文档对象模型（Document Object Model）。

![DOM HTML tree](https://zilong-blog-butterfly.oss-cn-shanghai.aliyuncs.com/article/pic_htmltree.gif)

通过可编程的对象模型，JavaScript 获得了足够的能力来创建动态的 HTML。

- JavaScript 能够改变页面中的所有 HTML 元素
- JavaScript 能够改变页面中的所有 HTML 属性
- JavaScript 能够改变页面中的所有 CSS 样式
- JavaScript 能够对页面中的所有事件做出反应

## 查找 HTML 元素

通常，通过 JavaScript，您需要操作 HTML 元素。

为了做到这件事情，您必须首先找到该元素。有三种方法来做这件事：

- 通过 id 找到 HTML 元素
- 通过标签名找到 HTML 元素
- 通过类名找到 HTML 元素

[JavaScript HTML DOM 改变 HTML 内容 | 菜鸟教程](https://www.runoob.com/js/js-htmldom-html.html)

### 

| 特性         | `getElementById`       | `querySelector`                |
| ------------ | ---------------------- | ------------------------------ |
| **选择范围** | 仅通过 `id` 查找       | 支持任意 CSS 选择器            |
| **返回值**   | 单个元素               | 第一个匹配的元素               |
| **性能**     | 更快                   | 稍慢（解析选择器需要更多时间） |
| **兼容性**   | 适用于所有浏览器       | 现代浏览器（IE8+）             |
| **用法场景** | 简单、快速按 `id` 查找 | 复杂选择器、灵活查找           |

**建议：**

- 如果只需要按 `id` 查找，推荐使用 `getElementById`。
- 如果需要更复杂的选择器或灵活性，可以使用 `querySelector`。