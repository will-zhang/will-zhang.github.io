---
layout: default
title: 理解Python中的元类(metaclass)
---


## {{ page.title }}


### 类也是对象

```
class Foo(object):
    pass
```
运行这段代码实际上会在内存中创建一个名为Foo的对象，和普通的对象并没有本质的区别。


### 动态创建类
type除了可以返回对象的类型外还可以用于动态创建类。
type(类名,父类的元组,包含的属性字典)

```
class Foo(object):
    pass
# 等价于
Foo = type('Foo', (), {})
# 创建一个名为Foo的类对象，并将它赋值给变量Foo


class FooChild(Foo):
    pass
# 等价于
FooChild = type('FooChild', (Foo,), {})
```

### 元类
元类就是用来创建类的东西, type实际上就是一个元类。


#### __class__属性
```
class Foo(object): pass
f = Foo()
print f.__class__
# <class '__main__.Foo'>
print f.__class__.__class__
# <type 'type'>
```


#### __metaclass__属性
```
class Foo(object):
    __metaclass__ = something...
```
Python在创建Foo类对象的时候会在类的定义中寻找__metaclass__属性，如果找到了，Python会用它来创建Foo，如果没有找到，就会用type来创建这个Foo。

```
class Foo(Bar):
    pass
```
先查找Foo的定义中是否有__metaclass__，如果没有会去父类中寻找，如果都没有会尝试在模块层次中寻找，如果还是找不到，就用type来创建。


#### 自定义元类
元类的主要目的就是为了在创建类时可以改变类。通常会在设计API时做这样的事情。
一个典型的例子是Django ORM。它允许像下面这样定义：
```
class Person(models.Model):
    name = models.CharField(max_length=30)
    age = models.IntegerField()

guy = Person(name='bob', age='35')
print guy.age
```
实际上guy.age并不是返回一个IntegerField，而是返回一个int。


### 结语
还有其他两种技术来修改类
1. Monkey patching
2. class decorators
当你需要动态修改类时，99%的时间最好使用上面两种技术。但是，在99%的时间根部就不需要动态修改类!

