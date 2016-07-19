---
layout: post
title: Effective Python-Item 3 Know the Differences Between bytes, str, and unicode
tags:
    - python
excerpt: 《Effective Python》的读书笔记, 比较在 python2 和 python3 中各种字符类型的区别
---

## 1. python2 与 python3 中表示字符的差异

在 python3 中, 用 `bytes` 来表示8位的字符， 用 `str` 来表示 unicode 字符;
而在 python2 中, 则是用 `str` 表示8位的字符, 用 `unicode` 表示 unicode 字符.

## 2. python2 与 python3 的编码转换方法
无论在 python2 还是在 python3 中, 要把 `unicode` 字符转换为二进制的数据, 都必须使用 `encode`方法. 反过来则需要使用 `decode` 方法
我们大致会写下这样的 helper function:
{% highlight python linenos %}
    # python3
    def to_str(bytes_or_str):
        if isinstance(bytes_or_str, bytes):
            value = bytes_or_str.decode('utf-8')
        else:
            value = bytes_or_str
        return value

    def to_bytes(bytes_or_str):
        if isinstance(bytes_or_str, str):
            value = bytes_or_str.encode('utf-8')
        else:
            value = bytes_or_str
        return value

    # python2
    def to_str(unicode_or_str):
        if isinstance(unicode_or_str, unicode):
            value = unicode_or_str.encdoe('utf-8')
        else:
            value = unicode_or_str
        return value

    def to_unicode(unicode_or_str):
        if isinstance(unicode_or_str, str):
            value = unicode_or_str.decode('utf-8')
        else:
            value = unicode_or_str
        return value
{% endhighlight %}

## 3. 会遇到的一些问题
但是在处理字符串过程中, 会遇到一些坑

1. 在 python2 中, 如果 `str` 字符串只包含 7位的 ASCII字符时, `unicode` 和 `str` 似乎是同一个类型.
- 可以用 + 操作符来连接
- 可以判断两者是否相等
- 在 format 字符串(如`%s`)可以使用 unicode 字符串

2. 在 python3 中, 文件句柄(`open()`的返回值) 是默认使用 UTF-8 编码的. 而在 python2 中, 则是默认使用二进制编码的. 因此, 在 python3 中如果要在文件中写入二进制数据, 则必须指明使用二进制写数据
{% highlight python linenos %}
with open('/tmp/random.bin', 'wb') as f:
    f.write(os.urandom(10))
{% endhighlight %}
