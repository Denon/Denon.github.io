---
layout: post
title: "Effective Python-Item 10: Prefer enumerate Over range"
tags:
    - python
excerpt: 《Effective Python》的读书笔记, 介绍一下enumerate这个函数
---

当你想循环一个数据结构, 比如一个数组, 你可以直接对这个对象进行for循环
{% highlight python linenos %}
flavor_list = ['vanilla', 'chocolate', 'pecan', 'strawberry']
for flavor in flavor_list:
    print('%s is delicious' % flavor)
{% endhighlight %}

不过很多时候你可能还需要每个元素的index, 所以你会改成这个样子
{% highlight python linenos %}
for i in range(len(flavor_list)):
    flavor = flavor_list[i]
    print('%d: %s' % (i + 1, flavor))
{% endhighlight %}

不过这看起来太丑了, python 提供了一种比较优雅的方法来完成这件事
{% highlight python linenos %}
for i, flavor in enumerate(flavor_list, 1):
    print('%d: %s' % (i, flavor))
{% endhighlight %}
