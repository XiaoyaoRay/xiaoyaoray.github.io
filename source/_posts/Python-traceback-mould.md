---
title: Python中traceback模块
date: 2018-10-14 10:33:03
categories:
- Python
tags:
- Python 
---

## 转载：[python异常处理之 traceback 解析示例](https://blog.csdn.net/helloxiaozhe/article/details/79230374)

**python异常处理之 traceback 解析示例**

**其中，cgitb适合在开发的过程中进行调试，而logging适合在线上环境使用，二者都非常方便**

### Python中的异常栈跟踪

```python
def func(a, b):
    return a / b
if __name__ == '__main__':
    import sys
    import traceback
    try:
        func(1, 0)
    except Exception as e:
        print "print exc"
        traceback.print_exc(file=sys.stdout)

```

<!--more-->

输出：

```sql
print exc
Traceback (most recent call last):
  File "/Users/a6/test_exception_process.py", line 34, in <module>
    func(1, 0)
  File "/Users/a6/test_exception_process.py", line 29, in func
    return a / b
ZeroDivisionError: integer division or modulo by zero

```

其实traceback.print_exc()函数只是traceback.print_exception()函数的一个简写形式，而它们获取异常相关的数据都是通过sys.exc_info()函数得到的。

```python
def func(a, b):
    return a / b
if __name__ == '__main__':
    import sys
    import traceback
    try:
        func(1, 0)
    except Exception as e:
        print "print_exception()"
        exc_type, exc_value, exc_tb = sys.exc_info()
        print 'the exc type is:', exc_type
        print 'the exc value is:', exc_value
        print 'the exc tb is:', exc_tb
        traceback.print_exception(exc_type, exc_value, exc_tb)

```

输出：

```vbnet
print_exception()
the exc type is: <type 'exceptions.ZeroDivisionError'>
the exc value is: integer division or modulo by zero
the exc tb is: <traceback object at 0x10acc9dd0>
Traceback (most recent call last):
  File "/Users/a6/Downloads/PycharmProjects/speiyou_di_my/hive/shuangshi_haibian/test_exception_process.py", line 49, in <module>
    func(1, 0)
  File "/Users/a6/Downloads/PycharmProjects/speiyou_di_my/hive/shuangshi_haibian/test_exception_process.py", line 44, in func
    return a / b
ZeroDivisionError: integer division or modulo by zero

```

**sys.exc_info()返回的值是一个元组，其中第一个元素，exc_type是异常的对象类型，exc_value是异常的值，exc_tb是一个traceback对象，对象中包含出错的行数、位置等数据。然后通过print_exception函数对这些异常数据进行整理输出。** 

### traceback模块提供了extract_tb函数来更加详细的解释traceback对象所包含的数据：

```python
def func(a, b):
    return a / b
if __name__ == '__main__':
    import sys
    import traceback
    try:
        func(1, 0)
    except:
        _, _, exc_tb = sys.exc_info()
        for filename, linenum, funcname, source in traceback.extract_tb(exc_tb):
            print "%-23s:%s '%s' in %s()" % (filename, linenum, source, funcname)

```

输出：

```python
/Users/a6/test_exception_process.py:67 'func(1, 0)' in <module>()
/Users/a6/test_exception_process.py:62 'return a / b' in func()

```

### 使用cgitb来简化异常调试

**如果平时开发喜欢基于log的方式来调试，那么可能经常去做这样的事情，在log里面发现异常之后，因为信息不足，那么会再去额外加一些debug log来把相关变量的值输出。调试完毕之后再把这些debug log去掉。其实没必要这么麻烦，Python库中提供了cgitb模块来帮助做这些事情，它能够输出异常上下文所有相关变量的信息，不必每次自己再去手动加debug log。 cgitb的使用简单的不能想象:**

```python
def func(a, b):
    return a / b
if __name__ == '__main__':
    import cgitb
 
    cgitb.enable(format='text')
    import sys
    func(1, 0)

```

输出：

```kotlin
/System/Library/Frameworks/Python.framework/Versions/2.7/bin/python2.7 /Users/a6/test_exception_process.py
<type 'exceptions.ZeroDivisionError'>
Python 2.7.10: /System/Library/Frameworks/Python.framework/Versions/2.7/Resources/Python.app/Contents/MacOS/Python
Thu Feb  1 16:58:32 2018
 
A problem occurred in a Python script.  Here is the sequence of
function calls leading up to the error, in the order they occurred.
 
 /Users/a6/test_exception_process.py in <module>()
   79     cgitb.enable(format='text')
   80     import sys
   81     func(1, 0)
   82 
   83 
func = <function func>
 
 /Users/a6/test_exception_process.py in func(a=1, b=0)
   73 
   74 def func(a, b):
   75     return a / b
   76 if __name__ == '__main__':
   77     import cgitb
a = 1
b = 0
<type 'exceptions.ZeroDivisionError'>: integer division or modulo by zero
    __class__ = <type 'exceptions.ZeroDivisionError'>
    __delattr__ = <method-wrapper '__delattr__' of exceptions.ZeroDivisionError object>
    __dict__ = {}
    __doc__ = 'Second argument to a division or modulo operation was zero.'
    __format__ = <built-in method __format__ of exceptions.ZeroDivisionError object>
    __getattribute__ = <method-wrapper '__getattribute__' of exceptions.ZeroDivisionError object>
    __getitem__ = <method-wrapper '__getitem__' of exceptions.ZeroDivisionError object>
    __getslice__ = <method-wrapper '__getslice__' of exceptions.ZeroDivisionError object>
    __hash__ = <method-wrapper '__hash__' of exceptions.ZeroDivisionError object>
    __init__ = <method-wrapper '__init__' of exceptions.ZeroDivisionError object>
    __new__ = <built-in method __new__ of type object>
    __reduce__ = <built-in method __reduce__ of exceptions.ZeroDivisionError object>
    __reduce_ex__ = <built-in method __reduce_ex__ of exceptions.ZeroDivisionError object>
    __repr__ = <method-wrapper '__repr__' of exceptions.ZeroDivisionError object>
    __setattr__ = <method-wrapper '__setattr__' of exceptions.ZeroDivisionError object>
    __setstate__ = <built-in method __setstate__ of exceptions.ZeroDivisionError object>
    __sizeof__ = <built-in method __sizeof__ of exceptions.ZeroDivisionError object>
    __str__ = <method-wrapper '__str__' of exceptions.ZeroDivisionError object>
    __subclasshook__ = <built-in method __subclasshook__ of type object>
    __unicode__ = <built-in method __unicode__ of exceptions.ZeroDivisionError object>
    args = ('integer division or modulo by zero',)
    message = 'integer division or modulo by zero'
 
The above is a description of an error in a Python program.  Here is
the original traceback:
 
Traceback (most recent call last):
  File "/Users/a6/test_exception_process.py", line 81, in <module>
    func(1, 0)
  File "/Users/a6/test_exception_process.py", line 75, in func
    return a / b
ZeroDivisionError: integer division or modulo by zero
 
Process finished with exit code 1

```

完全不必再去log.debug(“a=%d” % a)了，个人感觉cgitb在线上环境不适合使用，适合在开发的过程中进行调试，非常的方便。 

也许你会问，cgitb为什么会这么屌？能获取这么详细的出错信息？其实它的工作原理同它的使用方式一样的简单，它只是覆盖了默认的sys.excepthook函数，sys.excepthook是一个默认的全局异常拦截器，可以尝试去自行对它修改：

```python
def func(a, b):
    return a / b
 
def my_exception_handler(exc_type, exc_value, exc_tb):
    print "i caught the exception:", exc_type
    while exc_tb:
        print "the line no:", exc_tb.tb_lineno
        print "the frame locals:", exc_tb.tb_frame.f_locals
        exc_tb = exc_tb.tb_next
 
if __name__ == '__main__':
    import sys
    sys.excepthook = my_exception_handler
    func(1, 0)

```

输出：

```python
i caught the exception: <type 'exceptions.ZeroDivisionError'>
the line no: 99
the frame locals: {'my_exception_handler': <function my_exception_handler at 0x1069800c8>, '__builtins__': <module '__builtin__' (built-in)>, '__file__': '/Users/a6/Downloads/PycharmProjects/speiyou_di_my/hive/shuangshi_haibian/test_exception_process.py', '__package__': None, 'sys': <module 'sys' (built-in)>, 'func': <function func at 0x10697ac08>, '__name__': '__main__', '__doc__': None}
the line no: 87
the frame locals: {'a': 1, 'b': 0}

```

### 使用logging模块来记录异常

在使用Java的时候，用log4j记录异常很简单，只要把Exception对象传递给log.error方法就可以了，但是在Python中就不行了，如果直接传递异常对象给log.error，那么只会在log里面出现一行异常对象的值。 

在Python中正确的记录Log方式应该是这样的：

```python
logging.exception(ex)
logging.error(ex, exc_info=1) # 指名输出栈踪迹, logging.exception的内部也是包了一层此做法
logging.critical(ex, exc_info=1) # 更加严重的错误级别

```

具体示例如下:



```python
import logging
import os
from datetime import datetime
class LoggingInstance():
    def __init__(self):
        self.log_dir="logging.log"
    def make_log(self):
        # 初始化日志文件
        if not os.path.exists(self.log_dir):
            os.mkdir(self.log_dir)
        logging.basicConfig(level=logging.INFO,
                            filename="%s/log.%s" % (self.log_dir, datetime.now().strftime("%Y%m%d")), filemode='a',
                            format='%(asctime)s [%(levelname)s] [%(filename)s] [%(funcName)s] [%(lineno)d] %(message)s')
        logger = logging.getLogger(__name__)
        self.target_date = datetime.strptime(datetime.now().strftime("%Y%m%d"), "%Y%m%d")
        logger.info("init success, target_date=%s", self.target_date)
    def func(self, a, b):
        return a / b
 
if __name__ == '__main__':
    obj_logging_instance=LoggingInstance()
    obj_logging_instance.make_log()
    import sys
    import traceback
    try:
        obj_logging_instance.func(1, 0)
    except Exception as ex:
        print "print using exc logging"
        logging.info("分割线 @@@@@@@ target_date=%s ",datetime.now())
        logging.exception(ex)  # 抛出异常方法一
        logging.info("分割线 @@@@@@@ target_date=%s ",datetime.now())
        logging.error(ex,exc_info=1)  # 抛出异常方法二
        logging.info("分割线 @@@@@@@ target_date=%s ",datetime.now())
        logging.warning(ex,exc_info=1)  # 抛出异常方法三
        logging.info("分割线 @@@@@@@ target_date=%s ",datetime.now())
        logging.fatal(ex,exc_info=1)    # 抛出异常方法四
        logging.info("分割线 @@@@@@@ target_date=%s ",datetime.now())
        #traceback.print_exc()  # 可以在输出窗口输出错误信息

```

输出日志文件logging.log文件内容如下：

```vbnet
2018-02-01 17:25:03,999 [INFO] [test_exception_process.py] [make_log] [156] init success, target_date=2018-02-01 00:00:00
2018-02-01 17:25:03,999 [INFO] [test_exception_process.py] [<module>] [169] 分割线 @@@@@@@ target_date=2018-02-01 17:25:03.999508 
2018-02-01 17:25:03,999 [ERROR] [test_exception_process.py] [<module>] [170] integer division or modulo by zero
Traceback (most recent call last):
  File "/Users/a6/test_exception_process.py", line 166, in <module>
    obj_logging_instance.func(1, 0)
  File "/Users/a6/test_exception_process.py", line 158, in func
    return a / b
ZeroDivisionError: integer division or modulo by zero
2018-02-01 17:25:03,999 [INFO] [test_exception_process.py] [<module>] [171] 分割线 @@@@@@@ target_date=2018-02-01 17:25:03.999858 
2018-02-01 17:25:03,999 [ERROR] [test_exception_process.py] [<module>] [172] integer division or modulo by zero
Traceback (most recent call last):
  File "/Users/a6/test_exception_process.py", line 166, in <module>
    obj_logging_instance.func(1, 0)
  File "/Users/a6/test_exception_process.py", line 158, in func
    return a / b
ZeroDivisionError: integer division or modulo by zero
2018-02-01 17:25:04,000 [INFO] [test_exception_process.py] [<module>] [173] 分割线 @@@@@@@ target_date=2018-02-01 17:25:04.000040 
2018-02-01 17:25:04,000 [WARNING] [test_exception_process.py] [<module>] [174] integer division or modulo by zero
Traceback (most recent call last):
  File "/Users/a6/test_exception_process.py", line 166, in <module>
    obj_logging_instance.func(1, 0)
  File "/Users/a6test_exception_process.py", line 158, in func
    return a / b
ZeroDivisionError: integer division or modulo by zero
2018-02-01 17:25:04,000 [INFO] [test_exception_process.py] [<module>] [175] 分割线 @@@@@@@ target_date=2018-02-01 17:25:04.000206 
2018-02-01 17:25:04,000 [CRITICAL] [test_exception_process.py] [<module>] [176] integer division or modulo by zero
Traceback (most recent call last):
  File "/Users/a6/test_exception_process.py", line 166, in <module>
    obj_logging_instance.func(1, 0)
  File "/Users/a6/test_exception_process.py", line 158, in func
    return a / b
ZeroDivisionError: integer division or modulo by zero
2018-02-01 17:25:04,000 [INFO] [test_exception_process.py] [<module>] [177] 分割线 @@@@@@@ target_date=2018-02-01 17:25:04.000367 

```

参考网址：http://blog.csdn.net/wbiblem/article/details/73432506