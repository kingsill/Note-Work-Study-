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



## 定时器

[JavaScript 计时事件 | 菜鸟教程](https://www.runoob.com/js/js-timing.html)

### 防抖、节流

#### 防抖

闭包相关，变量的作用区域

[JavaScript 闭包 | 菜鸟教程](https://www.runoob.com/js/js-function-closures.html)

用户点击按钮的时候，点太快了点了两下，那么这个函数就会被执行两次

```javascript
    function so() {
        console.log("hello");
    }

    function debounce(func, delay) {
        let timer; // 这里的 timer 要保存到 debounce 作用域]
        let count = 0//只在debounce初次运行时初始化一次
        return function (...args) { // 返回一个闭包
            clearTimeout(timer); // 清除之前的定时器
            count = count + 1
            console.log(`${count}`)          
            timer = setTimeout(() => {
                func.apply(this, args); // 延迟执行传入的函数
            }, delay)
        }
    }

    // 重新包装 so 函数
    const debouncedSo = debounce(so, 1000);
    const debouncedSo2 = debounce(so2, 1000);
```

#### 节流

在规定的单位时间内，只能有一次触发事件的回调函数执行，如果在同一时间内被触发多次，只会生效一次

```javascript

    function throttle(fn, delay) {
        let t1 = new Date()
        return function (...arg) {
            let t2 = Date.now()
            let diff = t2 - t1
            t1 = t2
            console.log(diff)
            if (diff > delay) {
                fn.apply(this, arg)
            }
        }
    }

    const throttleHandler = throttle(so, 1000)
```



## window界面属性

### 屏幕，窗口，视口

```JavaScript
console.log("视口", window.innerWidth, window.innerHeight)
console.log("窗口", window.outerWidth, window.outerHeight)
console.log("屏幕", window.screen.width, window.screen.height)
```

navigator

[枫枫知道个人博客](https://www.fengfengzhidao.com/special/2/54)