# GO

### tour

1. 结构体指针 语法糖：隐式解引用

    ```go
    type Person struct{
        Age int
        Gender bool
    }
    
    Wang:=&Person{18,true}//Wang是指向这个结构体的指针
    fmt.Println(Wang.Age)//但是可以通过指针直接得到内部
    
    ```

2. string.Fields 自动分割字符 随时查找方便使用的函数，**不要夯，非要自己写**

   ```go
   ss:=string.Fields("Wang Zi Long")
   /*
   ss []string{
       "wang",
       "Zi",
       "Long"
   }
   */
   ```

3. 闭包
   Go可以返回函数，如果在函数内部有自由变量，则为闭包。
   一般函数内为局部变量，使用完重置，但是这里闭包扩大了其作用域。

4. 注意Go的值传递、引用传递  // 值类型、引用类型

5. 方法可以自动解指针引用

## threads

goroutine 用户态线程

I/O concurrency
Parallelism 使用多个cpu核心

**多线程** // 异步（时间驱动）编程

### Race

- mu.lock
- mu.unlock

### coordination

- channle
- sync.Cond
- waitGroup

