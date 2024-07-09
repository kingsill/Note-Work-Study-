`go run main.go api`
接口--控制层

OP OrderPact 订购合同

# cmd
## api.go
### 插件部分
#### cobra
用来创建CLI（命令行界面）应用程序的库
[学习视频](https://www.bilibili.com/video/BV1mR4y1y7fC/?spm_id_from=333.1007.top_right_bar_window_history.content.click)
[快捷初始化cobra-cli工具](https://github.com/spf13/cobra-cli)
[官方cobra user guide](https://github.com/spf13/cobra/blob/main/site/content/user_guide.md)

- var xxxCmd = &cobra.Command{}中
    - `Use`即为命令行中需要输入的
    - `Short\Long` 为介绍
    - 之后接具体执行` PersistentPreRun-- PreRun -- Run --PostRun -- PersistentPostRun`
- func Execute(){}
    -  内部定义的方法，执行cmd
- main.go中
    调用execute，进行cobra的初始化
- 需要添加命令，在cmd下新建 xx.go，在init中 通过rootcmd.addCommand 来添加
#### gin
##### bind绑定
在使用参数绑定时需要对我们定义的struct结构体 **注释** tag参数，例如json、xml等，具体取决于前端传来的是什么格式

`shouldbind`根据content-type自动选择引擎

##### 中间件和路由
`Gin`中的中间件必须是一个`gin.HandlerFunc`类型,
> `HandleFunc`实际上是`*context`的别名
[更多相关知识](https://docs.fengfengzhidao.com/#/docs/Gin%E6%A1%86%E6%9E%B6%E6%96%87%E6%A1%A3/6.%E4%B8%AD%E9%97%B4%E4%BB%B6%E5%92%8C%E8%B7%AF%E7%94%B1)
- 中间件放行  `.next`的执行顺序
- abort 后续都不再执行
- 全局路由在子路由之前运行

##### 全局上下文获取数据
controller--middleware.go
```go
// 将当前请求的username信息保存到请求的上下文c上
CSetUser(c, *user)
```
controller--helpsession.go
```go
if tmp, ok := c.Get("user"); ok {
    return tmp.(model.User).RoleId
}
```
##### 参数查询
- 动态参数（Dynamic Route Parameters）：
    格式：动态参数被包裹在路由路径中，以冒号（:）开头，例如 **/users/:id**。
    作用：动态参数用于捕获 URL 中的可变部分，并将其作为参数传递给处理该路由的函数。它们通常用于标识资源的唯一标识符uri，例如用户 ID、文章 ID 等。
    示例：在 /users/123 这个URL中，动态参数的名称是 "id"，对应的值是 "123"。
- 查询参数（Query Parameters）：**查询参数写路由时只写前缀即可**
    格式：查询参数是以 ? 开始，后面跟着键值对的形式，例如 **/search?q=apple&category=fruits**。
    作用：查询参数用于向服务器传递额外的信息，可以有多个键值对。它们通常用于过滤、排序、分页等操作。
    示例：在 /search?q=apple&category=fruits 这个 URL 中，查询参数有两个键值对：q 的值是 "apple"，category 的值是 "fruits"。


##### 全局上下文获取


# common
通常用于放项目中重复使用的代码
## conn
## Error
## global
## request
### user.go
在gin过程中所需的部分用户user相关的结构体定义，用于参数绑定

## response
### base64.go
### common.go
### encrypt.go
### validate.go
自定义验证器
> v.RegisterValidation(tag, validateFunc)
> `tag` 即为 `binding` 的对象，`validateFunc` 为定义的验证功能


# config viper.go
## viper 插件

# controller 控制层相关
## c_login.go 登录控制
## c_order.go 
### 插件 
- [xlsx](https://github.com/ivahaev/go-xlsx-templater)
    用于根据map[string]interface{}来生成excel表,需要读取模板选择保存等
    ```go
    doc := xlst.New()
	doc.ReadTemplate("./template.xlsx")
	doc.Render(ctx)
	doc.Save("./report.xlsx")
    ```
- [excelize](https://github.com/qax-os/excelize)
    [官方文档](https://xuri.me/excelize/zh-hans/)
    用于读取excel文件
## helper.go
- shouldBind函数
    自定义封装gin中shouldBind，传入上下文context、要绑定的结构体、以及对应的错误消息集，返回是否转换成功bool
    
## middleware.go 中间件业务
- NOTE：跨域请求相关
    在处理跨域请求时，浏览器会在某些条件下自动发起 **OPTIONS** 请求，这也被称为“预检”请求。以下是触发 OPTIONS 请求的一些常见条件：

    1. **跨域请求**：当一个请求尝试访问与其不同源（协议、域名或端口不同）的资源时。
    2. **自定义请求头**：如果请求中包含非标准的自定义头部字段。
    3. **特定的 HTTP 方法**：如 PUT、DELETE 等，这些方法可能对服务器数据产生副作用。
    4. **特定的 Content-Type**：当 POST 请求的 `Content-Type` 是 `application/json`, `application/x-www-form-urlencoded`, `multipart/form-data`, `text/plain` 之外的其他类型时。

    当这些条件中的任何一个满足时，浏览器会先发送一个 OPTIONS 请求到服务器，询问服务器是否允许这样的跨域请求。如果服务器响应允许，则浏览器会继续发起实际的请求¹²。

    此外，服务器可以通过设置 `Access-Control-Max-Age` 响应头来指定预检请求的结果可以缓存多长时间，这样在缓存有效期内，相同的跨域请求就不会再次触发 OPTIONS 请求¹。

    如果您在编程或者配置服务器时遇到任何问题，或者需要更多帮助，请随时告诉我！

    源: 与 Copilot 的对话， 2024/6/7
    (1) 什么时候会发送options请求（预检请求）_什么样的请求会 .... https://blog.csdn.net/yexudengzhidao/article/details/124342081.
    (2) 什么是options请求？为什么会有options请求？ - CSDN博客. https://blog.csdn.net/gwdgwd123/article/details/100554117.
    (3) 什么时候会发送options请求 - 掘金. https://juejin.cn/post/6844903821634699277.
    (4) undefined. https://xxxx.com/micro/share/getShareRecord.

- 代码注释部分
    您提供的代码是设置 CORS（跨源资源共享）策略的 HTTP 响应头部。这些头部用于定义哪些跨域请求是被允许的。下面是每个头部的分析：

    1. `Access-Control-Allow-Origin`: 指定哪些域可以访问资源。在这里，`*` 表示接受任何域的请求¹。
    2. `Access-Control-Allow-Headers`: 列出了浏览器在实际请求中允许携带的头部字段。这里包括了 `Content-Type`, `AccessToken`, `X-CSRF-Token`, `Authorization`, 和 `Token`¹。
    3. `Access-Control-Allow-Methods`: 指定允许的 HTTP 方法，这里允许 `POST`, `GET`, 和 `OPTIONS` 请求¹。
    4. `Access-Control-Expose-Headers`: 指定哪些响应头部可以暴露给前端的 JavaScript 处理。这里包括了 `Content-Length`, `Access-Control-Allow-Origin`, `Access-Control-Allow-Headers`, `Content-Type`, 和 `Content-Disposition`⁵。
    5. `Access-Control-Allow-Credentials`: 表示是否允许发送 Cookie。`true` 表示允许携带认证信息（如 Cookie 或 HTTP 认证）¹。
    6. `Access-Control-Max-Age`: 指定预检请求的结果可以缓存多长时间，单位是秒。这里设置为 `86400`，意味着 24 小时内不需要再发送预检请求¹。

    这些设置通常在服务器配置中使用，以允许来自不同源的前端应用程序安全地请求资源。如果您有任何其他问题或需要帮助，请告诉我！

    源: 与 Copilot 的对话， 2024/6/7
    (1) Cross-Origin Resource Sharing (CORS) - HTTP | MDN - MDN Web Docs. https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS.
    (2) Access-Control-Expose-Headers - HTTP | MDN - MDN Web Docs. https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Access-Control-Expose-Headers.
    (3) What is CORS? Complete Tutorial on Cross-Origin Resource Sharing - Auth0. https://auth0.com/blog/cors-tutorial-a-guide-to-cross-origin-resource-sharing/.
    (4) CORS - MDN Web Docs Glossary: Definitions of Web-related terms | MDN. https://developer.mozilla.org/en-US/docs/Glossary/CORS.
    (5) The Access-Control-Allow-Origin Header Explained – With a CORS Example. https://www.freecodecamp.org/news/access-control-allow-origin-header-explained/.
    (6) undefined. https://domain-a.com.
    (7) undefined. https://domain-b.com/data.json.
    (8) undefined. https://example.com.

# model 
`model` 中通常放置业务相关逻辑 以及 数据结构 数据库 相关内容 
## 插件
### gorm
便捷使用`go`语言进行数据库管理
gorm的update接口如果字段为零，默认不更新，是由于struct与map转换的问题
>解决：select(*)

gorm的关联查询 hasOne 模式 
>数据表不建立实际外键，通过model里设置外键即可，不需要migration也可以
>例子：m_order.go ----  OneOrder方法

gorm的软删除
>如果模型中包含有g`orm.DeletedAt`字段，调用delete删除时会更新该字段，但不会直接删除该记录

### bcrypt
对密码进行加密
重要函数：
- `bcrypt.CompareHashAndPassword()` 将加密后与待验证密码进行对比
- `bcrypt.bcrypt.GenerateFromPassword(password,cost)` 根据cost生成加密密码。一般cost设置为12，是性能和安全的平衡点 
## db.go
用于Mysql数据库的连接，创建并返回DB实例
## m_user.go
user表对应的结构体数据
以及 相关的自定义的快速的查表的方法



# util
## bucket
## file_util
## limiter
## log logger.go
自定义logger
`callDepth=3`,确保得到调用logger的信息
其他可见注释

# service 
## s_order.go
![image](note.assets/image.png)
各个type有重复等