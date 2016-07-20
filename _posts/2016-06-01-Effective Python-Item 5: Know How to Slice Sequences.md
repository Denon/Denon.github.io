---
layout: post
title: Effective Python-Item 5: Know How to Slice Sequences
tags:
    - python
excerpt: 《Effective Python》的读书笔记, Python 切片的一些特性
---

## 切片的语法
Python 提供了切片的语法, 最常见的使用场景是对 `list`, `str`, `bytes` 对象进行切片操作. 实现了 `__getitem__` 和 `__setitem__` 的类也可以使用切片方法(Item 28: Inherit from collections.abc for Custom Container Types).
切片最简单的语法是 `somelist[start:end]`
{% highlight python linenos %}
a = ['a', 'b', 'c', 'd', 'e', 'f', 'g', 'h']
print(a[:4])
print(a[-4:])
print(a[3:-3])
{% endhighlight %}

## 切片的特性
1. 即使 start 或者 end 的值超过了最大最大长度, 切片语法也不会报错
{% highlight python linenos %}
first_twenty_items = a[:20]
last_twenty_items = a[-20:]
{% endhighlight %}
>NOTE: 使用负数时, 如果 n < 0, 得到的结果是符合预期的; 如果 n = 0, 将会得到一个原数组的 copy

2.通过切片语法得到的数组是一个全新的数组, 因此对这个新数组进行修改将不会影响旧的数组
3.使用切片进行赋值时, 只会替换那些在范围类的元素, 在范围之外的元素都不会受到影响, `list` 会自动的缩小(扩大)来适应新的值
{% highlight python linenos %}
print('Before ', a)
a[2:7] = [99, 22, 14]
print('After ', a)
{% endhighlight %}
{% highlight bash linenos %}
>>>
Before ['a', 'b', 'c', 'd', 'e', 'f', 'g', 'h']
After ['a', 'b', 99, 22, 14, 'h']
{% endhighlight %}
