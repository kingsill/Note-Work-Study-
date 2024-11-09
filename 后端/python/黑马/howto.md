# 注释 comments

单行 
# 

多行 
’‘’ 
‘’‘

> python中'和“没有区别



# 数据类型

变量无类型，数据有类型

## 通过type（）查看类型



## 数据类型转换

1. int()
2. float()
3. str()



## 标识符

### 什么是标识符

变量的名字

方法的名字

类的名字

> 限制 
>
> 1. 内容限定
>    不能符号
>    不能数字开头
> 2. 大小写敏感
> 3. 不可使用关键词



### 变量命名规范

多个单词组合，下划线分割

英文字母全小写



# 字符串

## 字符串格式化

相关问题：1. 变量过多，拼接麻烦

			2. 无法和其他类型拼接

```python
name = ("黑马程序员")
message = "学IT，就来%s" % name
print(message)
```

% 占位符

s 将变量 转换为 字符串 放入占位的 地方

多个变量占位，变量用括号括起来



###  类型占位

%s

%d

%f



数字精度控制

m.n

m代表宽度

n代表小数点精度



%5.2f

%5d



## 字符串快速格式化

```python
age = 11.2
print(f'我的年龄为{age}')
```



f"'语句{变量}'

f 为format



## 对表达式进行格式化

表达式：一条具有明确执行结果的语句

```python
name = "dragon fly"
stock_price = 10
stock_code = 111
stock_price_daily_growth_factor = 1.2
growth_days = 10
print(f"公司：{name},stock number:{stock_code},now price:{stock_price}")
print("daily growth:%.2f ,with %d grow,price now = %.2f" % (
    stock_price_daily_growth_factor, growth_days, stock_price_daily_growth_factor ** growth_days * stock_price))
```



# 数据输入

## input

input接收的都看作字符串

