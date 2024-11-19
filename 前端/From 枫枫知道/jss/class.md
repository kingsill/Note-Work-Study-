es6之前靠prototype面向对象，es6新增class和模块化的概念

## class定义

```JavaScript
class Animal{
    name
    age
    constructor(name) {
        this.name = name
        this.age = 2
    }
    eat(){
        console.log(`${this.name}在吃饭`)
    }
}

const cat = new Animal("加菲猫")

console.log(cat)
cat.eat()
```

其中的constructor属于构造方法，name、age属于对象属性，eat属于对象方法

其中的eat方法是在原型上的

![img](https://zilong-blog-butterfly.oss-cn-shanghai.aliyuncs.com/article/20240627142319.png)

## 继承

使用class之后就能很方便的使用继承功能

使用extends 去继承父类

这样子类就有父类的属性和方法了

```JavaScript
class Animal{
    name
    constructor(name) {
        this.name = name
    }
    eat(){
        console.log(`${this.name}在吃饭`)
    }
}
// 猫继承动物的属性或方法
class Cat extends Animal{
    color
    constructor(name, color) {
        super(name);
        this.color = color
    }
    running(){
        console.log("猫在行走")
    }
}
const cat = new Cat("加菲猫", "白色")
console.log(cat)
```

其中的super()方法是用于实例化继承的类

```javascript
super(arguments);  // 调用父构造函数
  super.parentMethod(arguments);  // 调用父方法
```

## 静态属性和方法

只能由类去调用

```JavaScript
class Animal{
    static name = "动物"
    static eat(){
        console.log(`在吃饭`)
    }
}

Animal.eat()
console.log(Animal.name)
```