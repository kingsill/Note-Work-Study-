# 函数

```python
def function_name(input):
    function body
    reuturn somevalue
```



## 函数的参数

定义函数部分为形参

传入的为实参

## 函数的返回值

return返回值

return会直接结束函数。后续代码不再执行

### none

不写return其实返回none字面量 // 主动返回none

1. 用于if判断 ，none等同于False
2. 暂时无需具体值的变量



# 函数的说明文档

## 通过注释解释函数

格式：

1. 多行注释

   > ```python
   > def func(x,y):
   >     """
   >     函数说明
   >     :param x: （空格）函数x的说明
   >     :param : 函数x的说明
   >     :return : 返回值的说明
   >     """
   >     函数体
   >     return
   > ```



# 局部变量、全局变量

写在函数外部就是全局变量

卸载函数内部就是局部变量

## global关键字

想在函数内部修改全局变量的值，需要声明自己所使用的变量为全局变量

```python
num=100
def try_global():
    # try to change num
    global num
    num=200
    
try_global()
pirnt(num)
```



