---
layout: post
title: "Effective Python-Item24:Use @classmethod Polymorphism to Construct Objects Generically"
tags:
    - python
excerpt: 《Effective Python》的读书笔记, @classmethod的用法
---

在 Python , 可以很轻松的实现多态. 多态的定义好处就不多说了, google 一下就有
比如说, 你正在写一个 MapReduce 的实现.
首先你想用一个公共的类来代表数据输入
{% highlight python linenos %}
class InputData(object):
    def read(self):
        raise NotImplementedError

class PathInputData(InputData):
    def __init__(self, path):
        super().__init__()
        self.path = path

    def read(self):
        return open(self.path).read()
{% endhighlight %}
可能会有很多个像 `PathInputData` 一样的子类, 并且它们每一个都会实现 `read` 方法.
然后, 你会使用差不多的方法来实现 `Worker`
{% highlight python linenos %}
class Worker(object):
    def __init__(self, input_data):
        self.input_data = input_data
        self.result = None

    def map(self):
        raise NotImplementedError

    def reduce(self, other):
        raise NotImplementedError

class LineCountWorker(Worker):
    def map(self):
        data = self.input_data.read()
        self.result = data.count('\n')

    def reduce(self, other):
        self.result += other.result
{% endhighlight %}
然后, 需要写一些方法把这两个类串联起来, 实现最终的 MapReduce
{% highlight python linenos %}
def generate_inputs(data_dir):
    for name in os.listdir(data_dir):
        yield PathInputData(os.path.join(data_dir, name))

def create_workers(input_list):
    workers = []
    for input_data in input_list:
        workers.append(LineCountWorker(input_data))
    return workers

def execute(workers):
    threads = [Thread(target=w.map) for w in workers]
    for thread in threads:
        thread.start()
    for thread in threads:
        thread.join()

    first, rest = workers[0], workers[1:]
    for worker in rest:
        first.reduce(worker)
    return first.result

def mapreduce(data_dir):
    inputs = generate_inputs(data_dir)
    workers = create_workers(inputs)
    return execute(workers)
{% endhighlight %}
这样看起来好像可以, 但实际上有一个巨大的问题, 那就是这个 `mapreduce` 太不通用了, 如果你用另外的子类, 那么 `generate_inputs` 和 `create_workers` 这两个方法都要重写. 这个时候, `classmethod` 装饰器就可以出场了. 把方法改写如下
{% highlight python linenos %}
class InputData(object):
    def read(self):
        raise NotImplementedError

    @classmethod
    def generate_inputs(cls, config):
        raise NotImplementedError

class PathInputData(InputData):
    def __init__(self, path):
        super().__init__()
        self.path = path

    def read(self):
        return open(self.path).read()

    @classmethod
    def generate_inputs(cls, config):
        data_dir = config['data_dir']
        for name in os.listdir(data_dir):
            yield cls(os.path.join(data_dir, name))
{% endhighlight %}
同样, 对 `Worker` 也做同样的修改
{% highlight python linenos %}
class Worker(object):
    def __init__(self, input_data):
        self.input_data = input_data
        self.result = None

    def map(self):
        raise NotImplementedError

    def reduce(self, other):
        raise NotImplementedError

    @classmethod
    def create_workers(cls, input_class, config):
        workers = []
        for input_data in input_class.generate_inputs(config):
            workers.append(cls(input_data))
        return workers

class LineCountWorker(Worker):
    # ...
{% endhighlight %}
最后, 再重写一下 `mapreduce`
{% highlight python linenos %}
def mapreduce(worker_class, input_class, config):
    workers = worker_class.create_workers(input_class, config)
    return execute(workers)
{% endhighlight %}
