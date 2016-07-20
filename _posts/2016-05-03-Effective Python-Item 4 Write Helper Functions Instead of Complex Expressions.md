---
layout: post
title: "Effective Python-Item 4: Write Helper Functions Instead of Complex Expressions"
tags:
    - python
excerpt: 《Effective Python》的读书笔记, 有些时候, 与其写一个很复杂的表达式, 还不如写一个 helper 函数来得清晰明了.
---

python 精炼的语法可以让你很容易地在一行写出很多逻辑. 比如说: 你想从 URL 中获取查询参数
{% highlight python linenos %}
from urllib.parse import parse_qs
my_values = parse_qs('red=5&blue=0&green=', keep_blank_value=True)
print(repr(my_values))
{% endhighlight %}

{% highlight bash linenos %}
>>>
{'red': ['5'], 'green':[''], 'blue': ['0']}
{% endhighlight %}
有些参数可能会有多个值， 而有些参数有可能只有一个值, 有一些参数则可能完全没有. 则 `get` 这个会在不同的情况返回不同的值
{% highlight python linenos %}
print('Red :', my_values.get('red'))
print('Green :', my_values.get('green'))
print('Opacity: ', my_values.get('opacity'))
{% endhighlight %}
{% highlight bash linenos %}
>>>
Red: ['5']
Green: ['']
Opacity: None
{% endhighlight %}
如果需要把默认值设为0, 那么我们会这么写
{% highlight python linenos %}
red = my_values.get('red', [''])[0] or 0
green = my_values.get('green', [''])[0] or 0
opacity = my_values.get('opacity', [''])[0] or 0
print('Red :', red)
print('Green :', green)
print('Opacity: ', opacity)
{% endhighlight %}
{% highlight bash linenos %}
>>>
Red: '5'
Green: 0
Opacity: 0
{% endhighlight %}
这样虽然可以成功, 但是你需要必须确保从 `my_values` 中取得的是 `int` 类型, 所以你又必须使用 `int()` 来保证参数的类型
{% highlight python linenos %}
red = int(my_values.get('red', [''])[0] or 0)
{% endhighlight %}

这样的代码就很难读了, 对于这样复杂的逻辑, 最好的做法是写一个 helper function

{% highlight python linenos %}
def get_first_int(values, key, default=0):
    found = values.get(key, [''])
    if found[0]:
        found = int(found[0])
    else:
        found = default
    return found

green = get_frist_int(my_values, 'green')
{% endhighlight %}
这样可读性就好多了
