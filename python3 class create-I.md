[TOC]



## python类构造时的几个函数

### \__init__(self, ...)  

构造函数 

### \__call__(self, ...)  

实例化后可调用

```python
class A(object):
    def __init__(self,distords):
        self._distords = distords
        print("create A,and set distords to %s"%distords)
    def __call__(self,distords=None):
        print("here is the call from {},id is {}" .format(self._distords,distords))
        return "return from call"
def fun(input_class):
    obj=input_class
    print(obj("222"))

objb = A(True)
fun(A("111"))


```

outputs:

```bash
create A,and set distords to True #objb=A(True)
create A,and set distords to 111 #A("111")
here is the call from 111,id is 222
return from call

Process finished with exit code 0
```



### \__new__(self)



### \__del__ (self)

析构函数

### \__str__(self) 

将值转化为适合人阅读的形式

## 下划线 单双

- **__foo__**: 定义的是特殊方法，一般是系统定义名字 ，类似 __init__() 之类的。
- **_foo**: 以单下划线开头的表示的是 protected 类型的变量，即保护类型只能允许其本身与子类进行访问，不能用于 from module import *
- **__foo**: 双下划线的表示的是私有类型(private)的变量, 只能是允许这个类本身进行访问了。



## super()

是调用父类（超类）的一个方法。如果是单个继承，就是调用父类，如果是多重继承，就按照MRO(查找顺序)向上进行调用。顶层只会调用一次。

语法：（一般直接super.fun()）就行

```python
super(type[, object-or-type])

```

**使用super可以避免构造函数被调用两次**

print （class_name.mro）可以看到继承顺序

```python
     ---> B ---
A --|          |--> D
     ---> C ---
     

class A():
    def __init__(self):
        print('enter A')
        print('leave A')


class B(A):
    def __init__(self):
        print('enter B')
        super().__init__()
        print('leave B')


class C(A):
    def __init__(self):
        print('enter C')
        super().__init__()
        print('leave C')


class D(B, C):
    def __init__(self):
        print('enter D')
        super().__init__()
        print('leave D')

print(D.mro())
d = D()



```

```
[<class '__main__.D'>, <class '__main__.B'>, <class '__main__.C'>, <class '__main__.A'>, <class 'object'>]
enter D
enter B #这里的super.__init__()进入的是c，因为mro的顺序B上层是c
enter C
enter A
leave A
leave C
leave B
leave D


如果把B中的super().__init__去掉
因为B中没有继续向上调用__init__
enter D
enter B
leave B
leave D
```