---
layout: post
title: "Effective Python-Item 15: Know How Closures Interact with Variable Scope"
tags:
    - python
excerpt: 《Effective Python》的读书笔记, python里面的闭包
---

当你向以某种规则来对一个数组进行排序的时候, 最普遍的做法是写一个 helper function 来作为 `sort` 的参数.
比如:
{% highlight python linenos %}
def sort_priority(values, group):
    def helper(x):
        if x in group:
            return (0, x)
        return (1, x)
    values.sort(key=helper)
{% endhighlight %}
这段代码能生效的原因是
1. python 支持闭包(closures), 函数可以访问外层的变量
2. Functions 在 python 里是 `first-class` 对象, 这意味着你可以直接引用它, 或者把某个 function 赋予某个变量, 或者把 function 当作参数传给另外的 function.
3. python 在比较 `turple` 的时候, 会依次比较 `turple` 里面的元素

既然闭包可以决定排序的优先级, 那么我们干脆在闭包里面来判断是否有高优先级的元素在 `values` 里.
{% highlight python linenos %}
def sort_priority2(values, group):
    found = False
    def helper(x):
        if x in group:
            found = True
            return (0, x)
        return (1, x)
    values.sort(key=helper)
    return found
{% endhighlight %}

{% highlight python linenos %}
numbers = [8, 3, 1, 2, 5, 4, 7, 6]
group = {2, 3, 5, 7}
found = sort_priority2(numbers, group)
print numbers
{% endhighlight %}
{% highlight bash linenos %}
>>>
Found: False
[2, 3, 5, 7, 1, 4, 5, 8]
{% endhighlight %}
排序的结果是正确的, 但是 `found` 结果是错误的.

当你在表达式中引用一个变量的时候, python 会按照以下顺序在各个作用域中来找这个引用
1. 当前 function 的作用域
2. 其他任何闭包作用域
3. global scope
4. built-in 作用域
如果以上这些地方都没有找到, 那么 python 就会抛出一个 `NameError` 的异常

变量赋值则完全不是这样, 如果一个变量在**当前作用域**已经定义了, 那么只是赋予新的值. 反之, python 则会把赋值语句当成是定义一个新变量
这就是为什么 `found` 的返回值会是错的了. 因为在 `helper` 内部的 `found` 是一个新的值.

如果想要得到想要的结果, 在 python3 中, 可以这样做
{% highlight python linenos %}
# python3
def sort_priority2(values, group):
    found = False
    def helper(x):
        nonlocal found
        if x in group:
            found = True
            return (0, x)
        return (1, x)
    values.sort(key=helper)
    return found
{% endhighlight %}
当然一般不建议这么做, 最好的办法是使用类

如果是在 python2 中, 可以这样做
{% highlight python linenos %}
def sort_priority2(values, group):
    found = [False]
    def helper(x):
        if x in group:
            found[0] = True
            return (0, x)
        return (1, x)
    values.sort(key=helper)
    return found[0]
{% endhighlight %}
这是因为 `list` 是一个 mutable 对象, 闭包可以修改这个对象的值.
