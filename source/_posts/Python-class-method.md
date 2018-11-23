---
title: Python中类的继承及类的属性和方法总结
date: 2018-10-22 11:15:12
categories:
- Python
tags:
- Python
---

## 转载：[Python中类的继承及类的属性和方法总结](http://blog.51cto.com/fengyunshan911/2060037)

### 类的继承

#### 类的继承

-  继承是面向对象的重要特性之一
-  继承关系继承是相对两个类而言的父子关系
-  子类继承了父类的所有公有属性和方法
-  继承，实现了代码重用

<!--more-->

#### 使用继承

- 继承可以重用已经存在的数据和行为，减少代码的重复编写,

- Python在类名后使用一对括号来表示继承关系，括号中的即类为父类class Myclass(ParentClass)

- 如果父类定义了__init__方法，子类必须显式调用父类的__init__方法，ParentClass.__init__(self,[args...])

- 如果子类需要扩展父类的行为，可以添加__init__方法的参数.

```
#!/usr/bin/env python
#-*- coding:utf-8  -*-
class People(object):
    color = 'yellow'
    
    def think(self):
    self.color = "black"
    print "I am a %s "  % self.color
    print ("I am a thinker")
    
class Chinese(People):
    pass
    
cn = Chinese()
print cn.color
cn.think()
```

- 父类中有构造函数：

```
#!/usr/bin/env python
#-*- coding:utf-8  -*-
class People(object):
    color = 'yellow'
     def __init__(self):
        print "Init..."
        self.dwell = 'Earth'
    def think(self):
        print "I am a %s "  % self.color
        print ("I am a thinker")
class Chinese(People):
    pass
cn = Chinese()
print cn.dwell
cn.think()
```

- 参数大于两个：

```
#!/usr/bin/env python
#-*- coding:utf-8  -*-
class People(object):
    color = 'yellow'
    def __init__(self,c):
        print "Init..."
        self.dwell = 'Earth'
     def think(self):
        print "I am a %s "  % self.color
        print ("I am a thinker")
class Chinese(People):
     def __init__(self):
        People.__init__(self,'red')
        pass
cn = Chinese()
```



#### Super 函数

```
class A(object):
        def __init__(self):
            print "enter A"
            print "leave A"
class B(object):
        def __init__(self):
            print "enter B"
            super(B,self),__init__()
            print "leave B"
b = B()
```

```
#!/usr/bin/env python
#-*- coding:utf-8  -*-
class People(object):
    color = 'yellow'
    def __init__(self,c):
        print "Init..."
        self.dwell = 'Earth'
    def think(self):
        print "I am a %s "  % self.color
        print ("I am a thinker")
class Chinese(People):
    def __init__(self):
       super(Chinese,self).__init__('red')
       pass
cn = Chinese()
cn.think()
```

```
#!/usr/bin/env python
#-*- coding:utf-8  -*-
class People(object):
    color = 'yellow'
    def __init__(self,c):
        print "Init..."
        self.dwell = 'Earth'
    def think(self):
        print "I am a %s "  % self.color
        print ("I am a thinker")
class Chinese(People):
    def __init__(self):
        super(Chinese,self).__init__('red')
     def talk(self):
        print "I like taking."
cn = Chinese()
cn.think()
cn.talk()
```



### 多重继承

​    Python支持多重继承，第一个类可以继承多个父类

##### 语法:

    class class_name(Parent_c1,Parent_c2,...)
```
#!/usr/bin/env python
#-*- coding:utf-8  -*-
class People(object):
    color = 'yellow'
    def __init__(self):
        print "Init..."
        self.dwell = 'Earth'
    def think(self):
        print "I am a %s "  % self.color
        print ("My home is %s ") % self.dwell
class Martian(object):
    color = 'red'
    def __init__(self):
        self.dwell = 'Martian'
class Chinese(People,Martian):
    def __init__(self):
        People.__init__(self)
cn = Chinese()
cn.think()
```
##### 注意:

​    当父类中出现多个自定义的__init__的方法时，多重继承，`只执行第一个累的__init_方法，其他不执行`。

```
#!/usr/bin/env python
#-*- coding:utf-8  -*-
class People(object):
    def __init__(self):
        self.dwell = 'Earth'
         self.color = 'yellow'
    def think(self):
        print "I am a %s "  % self.color
        print ("My home is %s ") % self.dwell
class Martian(object):
    color = 'red'
    def __init__(self):
        self.dwell = 'Martian'
    def talk(self):
        print "I like talking"
class Chinese(Martian,People):
    def __init__(self):
        People.__init__(self)
cn = Chinese()
cn.think()
cn.talk()
```





### 类的属性总结

- 类属性，也是公有属性，

-  类的私有属性，

-  对象的共有属性，

- 对象的私有属性，

- 内置属性，

-  函数的局部变量，

- 全局变量，

```python
#/usr/bin/env python
# -*- coding:utf-8 -*-
class MyClass(object):
        var1 = '类属性，类的公有属性 var1'
        __var2 = '类的私有属性 __var2'
        def func1(self):
            self.var3 = '对象的公有属性 var3'
            self.__var4 = '对象的私有属性 __var4'
            var5 = '函数的局部变量'
mc = MyClass()
mc.func1()     #调用后才测打印出var3
print mc.var1
print mc._MyClass__var2
print mc.var3
mc1 = MyClass()
# mc1.func1()    #mc1没有调用方法
print mc1.var3    
```

#### 通过类访问：

```python
    #/usr/bin/env python
    # -*- coding:utf-8 -*-
    # @time   :2018/1/2 21:06
    # @Author :FengXiaoqing
    # @file   :__init__.py.py
    #
    var6 = '全局变量 '
    class MyClass(object):
        var1 = '类属性，类的公有属性 var1'    ##定义在方法外
        __var2 = '类的私有属性 __var2'
        def func1(self):
            self.var3 = '对象的公有属性 var3'      ##定义在方法内
            self.__var4 = '对象的私有属性 __var4'
            var5 = '函数的局部变量'
        def func2(self):
            print self.var1
            print self.__var2
            print self.var3
            print self.__var4
            print self.var6
    mc = MyClass()
    mc.func1()
    mc.func2()
    print '*'*50
    print mc.__dict__
    print MyClass.var1
    #print MyClass.__var2    #不测通过类访问
    print mc.var3       #对象的属性只能通过对象来访问
    #print MyClass.__var4
    print MyClass.__dict__
```



### 类的方法总结

- 公有方法

- 私有方法

- 类方法

- 静态方法

- 内置方法

```python
class MyClass(object):
    name = 'Test'
    def func1(self):
        print self.name,
        print "我是公有方法."
        self.__func2() #func1间接调用了func2的私有方法
    def __func2(self):
        print self.name,
        print "我是私有方法."
    def classFun(self):
        print self.name,
        print "我是类方法."
    def staticFun(self):
        print s.name,
        print "我是静态方法."
mc = MyClass()
mc.func1()
```

#### 调用类方法:用装饰器

```
    @classmethod
    def classFun(self):
        print self.name,
        print "我是类方法."
    def staticFun(self):
        print s.name,
        print "我是静态方法."
mc = MyClass()
mc.func1()
MyClass.classFun()
```

#### 调用静态方法：

```
    @staticmethod
    def staticFun():
        print MyClass.name,    
        print "我是静态方法."
mc = MyClass()
mc.func1()
MyClass.classFun()
MyClass.staticFun()
```

#### 调用内置方法:

```
class MyClass(object):
    name = 'Test'
    def __init__(self):
        self.func1()
        self.__func2()
        self.classFun()
        self.staticFun()
    def func1(self):
        print self.name,
        print "我是公有方法."
    def __func2(self):
        print self.name,
        print "我是私有方法."
    @classmethod
    def classFun(self):
        print self.name,
        print "我是类方法."
    @staticmethod
    def staticFun():
        print MyClass.name,
        print "我是静态方法."
mc = MyClass()
```