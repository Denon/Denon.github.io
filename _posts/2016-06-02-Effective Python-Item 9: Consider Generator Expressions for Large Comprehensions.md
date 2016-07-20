---
layout: post
title: "Effective Python-Item 9: Consider Generator Expressions for Large Comprehensions"
tags:
    - python
excerpt: 《Effective Python》的读书笔记, 主要关于生成器表达式
---

使用列表生成式(list comprehensions)的时候, 我们会创建一个包含所有输入值的完整列表. 当输入值不多的时候, 使用列表生成式没什么问题, 但是对于大量的输入值, 使用列表生成式辉消耗大量的内存从而导致你的程序崩溃.
比如说: 你想通缉一个文件每一行有多少个字符, 使用列表生成式就需要把每一行的长度都保存来列表(内存)里面, 如果这个文件非常大, 或者是一直运行的 network socket, 这就会很有问题了
{% highlight python linenos %}
value = [len(x) for x in open('/tmp/my_file.txt')]
print value
{% endhighlight %}

python 提供了生成器表达式(generator expressions)来解决这个问题. 生成器表达式一次只会生成一个值.
{% highlight python linenos %}
it = (len(x) for x in open('/tmp/my_file.txt'))
print(it)
{% endhighlight %}
```bash
>>>
<generator object <genexpr> at <xxxxxxxxx>
```
你可以像使用普通的 generator 一样去操作这个返回的对象
{% highlight python linenos %}
print(next(it))
{% endhighlight %}
生成器表达式另外一个很强大的功能就是它们可以组合在一起, 既一个生成器表达式可以当作另外一个生成器表达式的输入
{% highlight python linenos %}
roots = ((x, x**0.5) for x in it)
print(next(roots))
{% endhighlight %}
>NOTE: 需要注意的是, 生成器表达式的返回值只能使用一次(和普通iterator一样)
