---
layout: post
title: "Effective Python-Item 23: Accept Functions for Simple Interfaces Instead of Class"
tags:
    - python
excerpt: 《Effective Python》的读书笔记
---

很多 python 的 API 的允许你传入一个函数作为参数来定制自己的需求。(也就是我们常说的 callback). 比如说 `sort` 这个函数
{% highlight python linenos %}
names = ['Socrates', 'Archimedes', 'Plato', 'Aristotle']
names.sort(key=lambda x: len(x))
print(names)
{% endhighlight %}
{% highlight bash linenos %}
>>>
['Plato', 'Socrates', 'Aristotle', 'Archimedes']
{% endhighlight %}

有时候, 你想定制一下 `defaultdict` 这个类的行为的时候, 就可以通过传入自己的方法来达到自己的目的. 比如说我想记录当 key 不存在, 然后返回0作为默认值.可以这样写
{% highlight python linenos %}
def log_missing():
    print('Key added')
    return 0

current = {'green': 12, 'blue': 3}
increments = [
    ('red', 5)
    ('blue', 17)
    ('orange', 9)
]
result = defaultdict(log_missing, current)
print('Before: ', dict(result))
for key, amount in increments:
    result[key] += amount
print('After: ', dict(result))
{% endhighlight %}
{% highlight bash linenos %}
>>>
Before: {'green': 12, 'blue': 3}
Key added
Key added
After: {'orange': 9, 'green': 12, 'blue': 20, 'red': 5}
{% endhighlight %}
实际上， 除了 function , 任何实现了 `__call__` 的类也都可以作为 callback 当作参数传进去
{% highlight python linenos %}
class BetterCountMissing(object):
    def __init__(self):
        self.added = 0
    def __call__(self):
        self.added += 1
        return 0

counter = BetterCountMissing()
result = defaultdict(counter, current)
{% endhighlight %}
