# 函数的多返回值

`return 1,2`	使用','进行变量的分割



# 传参方式

**位置参数**，根据位置传入

```python
def user_info(name,id):
    ......

user_info("wang",1)
```



**关键字传参**	关键字传参 没有顺序

```python
def user_info(name,id):
    .......
    
user_info(name="wang",id=1)
```

关键字和位置同时传参时，位置参数必须在关键字参数



**缺省参数** 	可以设置默认值,默认参数需要写在最后面

```python
def user_info(name,id,gender='male'):
    .......
    
```



**不定长参数**

1. 位置传递
   ```python
   def user_info(*args):
       type(args)
       ....
      
   ```

   args为元组

2.  关键字传递
   ```python
   def user_info(**kwargs):
       ...
       
   user_info(name="wang",age=11)
   ```

   kwargs为字典



# 匿名函数

## 将函数作为参数传递

```python
def test_func(compute):
    result=compute(1,2)
    print(result)
    
def compute(x,y)
	return x+y

num=test_func(compute)
```

计算逻辑的传递，而非数据的传递

数据是确定的，但是逻辑没确定

## 匿名函数 lambda

- def关键字       有名称	可重 复使用
- lambda关键字        无名称       只能临时使用一次

```python
lambda 传入参数：函数体			函数体只能有一行代码，默认return
```

