---
layout: post
title: "Effective Python-Item 25: Initialize Parent Classes with super"
tags:
    - python
excerpt: 《Effective Python》的读书笔记, super的用法
---

在以前， 如果想实例化父类, 很多人的做法是直接在子类里面调用父类的 `__init__ ` 方法.
{% highlight python linenos %}
class MyBaseClass(object):
    def __init__(self, value):
        self.value = value

class My ChildClass(MyBaseClass):
    def __init__(self):
        MyBaseClass.__init__(self, 5)
{% endhighlight %}
一般的情况下, 这样做是可以的. 但是如果涉及到多继承, 直接调用父类的 `__init__` 方法将会导致不可预料的后果.  

* 不能控制调用 `__init__` 方法的顺序.比如:

    {% highlight python linenos %}
    class TimesTwo(object):
        def __init__(self):
            self.value *= 2

    class PlusFive(object):
        def __init__(self):
            self.value += 5

    class OneWay(MyBaseClass, TimesTwo, PlusFive):
        def __init__(self, value):
            MyBaseClass.__init__(self, value)
            TimesTwo.__init__(self)
            PlusFive.__init__(self)
    foo = OneWay(5)
    print('First ordering is (5 * 2) + 5 =', foo.value)

    class AnotherWay(MyBaseClass, PlusFive, TimesTwo):
        def __init__(self, value):
            MyBaseClass.__init__(self, value)
            TimesTwo.__init__(self)
            PlusFive.__init__(self)

    bar = AnotherWay(5)
    print('Second ordering still is ', bar.value)
    {% endhighlight %}
    {% highlight bash linenos %}
    >>>
    First ordering is (5 * 2) + 5 = 15
    Second ordering still is 15
    {% endhighlight %}

* 钻石继承(Diamond inheritance)  

    可以参考这个[回答](http://stackoverflow.com/questions/3277367/how-does-pythons-super-work-with-multiple-inheritance)

---------------------------

为了解决这两个问题, python2.2 引入了 `super` 方法以及定义了 [MRO](https://www.python.org/download/releases/2.3/mro/).
{% highlight python linenos %}
class TimesFiveCorrect(MyBaseClass):
    def __init__(self, value):
        self.value *= 5

class PlusTwoCorrect(MyBaseClass):
    def __init__(self, value):
        self.value += 2
{% endhighlight %}
{% highlight python linenos %}
# python2
class GoodWay(TimesFiveCorrect, PlusTwoCorrect):
    def __init__(self, value):
        super(GoodWay, self).__init__(value)

foo = GoodWay(5)
print 'Should be 5 * (5 + 2) = 35 and is ', foo.value
{% endhighlight %}
{% highlight bash linenos %}
>>>
Should be 5 * (5 + 2) = 35 and is 35
{% endhighlight %}
我们可以用 `mro()` 来查看这个类的 mro 属性
{% highlight python linenos %}
from pprint import pprint
pprint(GoodWay.mro())
{% endhighlight %}
{% highlight bash linenos %}
[<class '__main__.GoodWay'>,
 <class '__main__.TimesFiveCorrect'>,
 <class '__main__.PlusTwoCorrect'>,
 <class '__main__.MyBaseClass'>
 <class 'object'>]
{% endhighlight %}
可以看到 `GoodWay` 的调用顺序.
不过在 python2 中, 这个 `super` 方法的调用实在很奇怪. 幸运的是, python3 修改了这个方法.
{% highlight python linenos %}
# python3
class Explicit(MyBaseClass):
    def __init__(self, value):
        super(__class__, self).__init__(value * 2)

class Implicit(MyBaseClass):
    def __init__(self, value):
        super().__init__(value * 2)
{% endhighlight %}
