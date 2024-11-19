## 浏览器对象模型 (BOM)

浏览器对象模型（**B**rowser **O**bject **M**odel (BOM)）尚无正式标准。

由于现代浏览器已经（几乎）实现了 JavaScript 交互性方面的相同方法和属性，因此常被认为是 BOM 的方法和属性。

## Location 浏览器地址

以 http://do.com:3000/xx/yy?name=ff 为例

| 属性                            | 值                           | 说明                                                         |
| ------------------------------- | ---------------------------- | ------------------------------------------------------------ |
| location.portocol 位置.portocol | http: http：                 | 页面使用的协议，通常是`http:`或`https:`                      |
| location.hostname 位置.主机名   | do.com                       | 服务器域名                                                   |
| location.port 位置.端口         | 3000                         | 请求的端口号                                                 |
| location.host 位置.主机         | do.com:3000                  | 服务器名及端口号                                             |
| location.origin 位置.原点       | http://do.com:3000           | url的源地址，只读                                            |
| location.href 位置.href         | 完整的url地址                | 等价于`window.location`                                      |
| location.pathname 位置.路径名   | `/`(这里指的是3000后面的`/`) | url中的路径和文件名，不会返回hash和search后面的内容，只有当打开的页面是一个文件时才会生效 |
| location.search 位置.搜索       | ?name=ff ?名称=ff            | 查询参数                                                     |

可以重新赋值