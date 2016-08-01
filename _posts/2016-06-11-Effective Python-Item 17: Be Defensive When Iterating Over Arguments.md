---
layout: post
title: "Effective Python-Item 17: Be Defensive When Iterating Over Arguments"
tags:
    - python
excerpt: 《Effective Python》的读书笔记
---

当一个 function 接受一个 list 作为参数的时候, 会经常需要遍历这个 list 很多遍.
比如说: 你想统计一下美国德州的游客人数, 你有每个城市的游客人数. 通过这些, 你可以算出每个城市接待游客的百分比. 为了完成这个功能, 你需要一个初始化函数. 它会计算每年游客的总和, 然后再计算每个城市的百分比.
{% highlight python linenos %}
def normalize(numbers):
    total = sum(numbers)
    result = []
    for value in numbers:
        percent = 100 * value / total
        result.append(percent)
    return result

visits = [15, 35, 80]
percentages = normalize(visits)
print percentages
{% endhighlight %}
{% highlight bash linenos %}
>>>
[11, 26, 61]
{% endhighlight %}

为了读取更多的数据, 现在需要从文件里面来读取德州每个城市的数据了. 因此, 需要定义一个 `generator` 来做这件事.
{% highlight python linenos %}
def read_visits(data_path):
    with open(data_path) as f:
        for line in f:
            yield int(line)
{% endhighlight %}
但是, 使用这个函数并不能生成正确的结果
{% highlight python linenos %}
it = read_visits('/tmp/city.data')
percentages = normalize(it)
print(percentages)
{% endhighlight %}
{% highlight bash linenos %}
>>>
[]
{% endhighlight %}
这是因为一个 iterator 只是会迭代一遍数据, 因此第二次遍历这个列表的时候, 只会得到一个空数组, 而你并没有得到任何异常的信息, 因为很多 python 内置的函数里面已经捕获到了这个异常了. 为了解决这个问题, 最简单是在 normalize 内部对这个 iterator 做一个 copy
{% highlight python linenos %}
def normalize(numbers):
    numbers = list(numbers)
    total = sum(numbers)
    result = []
    for value in numbers:
        percent = 100 * value / total
        result.append(percent)
    return result
{% endhighlight %}
这样当然可以正常工作, 但是这有可能会生成一个巨大的数据, 那么用 iterator 就没有意义了. 因此需要改进一下, 可以再写一个函数来专门返回一个新的 iterator
{% highlight python linenos %}
def normalize(get_iter):
    total = sum(get_iter())
    result = []
    for value in get_iter():
        percent = 100 * value / total
        result.append(percent)
    return result
{% endhighlight %}
我们可以用一个 lambda 函数来做
{% highlight python linenos %}
percentages = normalize(lambda:read_visits(path))
{% endhighlight %}
但是这样写代码实在太丑了, 更好的办法是写一个类, 并且实现类的 iterator protocol.
当 python 执行 `for x in foo` 的时候, 实际上他会先调用 `iter(foo)`, 而 `iter` 函数则实际调用 `foo.__iter__` 方法. `__iter__` 方法必须返回一个 iterator 对象(也就是实现了 `__next__` 方法的对象). 这样 for 循环会不断的调用 `next` 方法, 直到 iterator 抛出 `StopIteration` 异常.

那么我们可以这样来写这个类
{% highlight python linenos %}
class ReadVisits(object):
    def __init__(self, data_path):
        self.data_path = data_path

    def __iter__(self):
        with open(data_path) as f:
        for line in f:
            yield int(line)
{% endhighlight %}
使用如下
{% highlight python linenos %}
visits = ReadVisits(path)
percentages = normalize(visits)
{% endhighlight %}

同时为了保证 normalize 函数的运行, 我们需要对输入值做一下判断
{% highlight python linenos %}
def normalize(numbers):
    if iter(numbers) is iter(numbers):
        raise TypeError("Must supply a container")
    total = sum(numbers)
    result = []
    for value in numbers:
        percent = 100 * value / total
        result.append(percent)
    return result
{% endhighlight %}
这是因为对 iterator 本身做 iter 操作, 会返回 iterator 本身
