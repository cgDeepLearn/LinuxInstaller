# Celery + Redis/RabbitMQ + Flower

## Redis

请参考[Redis](../databases/redis.md)部分

## Celery

Celery是分布式任务队列，能实时处理任务， 同时支持task scheduling。
Celery工作原理如下：

1. celery client发送message给`broker`

2. `worker`从`broker`中消费消息，并将结果存储在result_end中

### 简介

- Brokers

`brokers` 中文意思为中间人，在这里就是指任务队列本身，Celery 扮演生产者和消费者的角色，`brokers` 就是生产者和消费者存放/拿取产品的地方(队列)

常见的 brokers 有 `rabbitmq`、`redis`、`Zookeeper` 等

- Result Stores / Backend

顾名思义就是结果储存的地方，队列中的任务运行完后的结果或者状态需要被任务发送者知道，那么就需要一个地方储存这些结果，就是 Result Stores 了

常见的 backend 有 `redis`、`Memcached` 甚至常用的数据库(`mysql`, `postgresql`等)都可以。

- Workers

就是 Celery 中的工作者，类似与生产/消费模型中的消费者，其从队列中取出任务并执行

- Tasks

就是我们想在队列中进行的任务，一般由用户、触发器或其他操作将任务入队，然后交由 workers 进行处理。

下面我们用`redis`作为`celery`的`broke`r和`backend`作为使用示例

### 安装celery 

```python pip安装celery以及Python对redis的支持
pip install celery
pip install redis
```

### 简单示例

我们用`redis`作为`broker`和`backend`，下面我们来编写`tasks`和`workers`

- tasks.py

```python tasks.py
from celery import Celery
# 配置broker和backend
app = Celery('tasks', backend='redis://localhost:6379/0', broker='redis://localhost:6379/1')

@app.task  #普通函数装饰为celery task
def add(x, y):
    return x + y
```

启动`celery`: `celery -A tasks worker -l info`

我们可以看到类似以下的输出：

```
[2017-02-07 16:13:55,227: INFO/MainProcess] Connected to redis://:**@localhost:6379/1
[2017-02-07 16:13:55,237: INFO/MainProcess] mingle: searching for neighbors
[2017-02-07 16:13:56,259: INFO/MainProcess] mingle: all alone
[2017-02-07 16:13:56,267: INFO/MainProcess] celery@master ready.
```

- trigger.py

触发任务:我们每次运行trigger就把随机生成的两个数加起来的(当然你也可以通过命令行传参)

```python trigger.py
import random
import time
from tasks import add
pairs = [(1, 2), (3, 1), (2, 0)]
pair = random.choice(pairs)
result = add.delay(*pair)
while not result.ready():
    time.sleep(1)
print('task done: {}'.format(result.get()))
```

项目主目录运行trigger.py,看一下返回结果: `python trigger.py`

```
[2018-02-07 16:14:39,880: INFO/ForkPoolWorker-1] Task tasks.add[2a5385eb-4385-430d-85e9-a22477f97c4a] succeeded in 0.0051999082788825035s: 3
[2018-02-07 16:15:51,881: INFO/MainProcess] Received task: tasks.add[99aa884b-a242-4008-867b-b70058bc5ef3]
[2018-02-07 16:15:51,883: INFO/ForkPoolWorker-1] Task tasks.add[99aa884b-a242-4008-867b-b70058bc5ef3] succeeded in 0.00046310946345329285s: 8
```

我们运行了两次，可以看到两个结果 `3` 和 `8`。

### 进阶用法

上面的装饰器`app.task`实际上是将一个正常的函数修饰成了一个 `celery task` 对象，所以这里我们可以给修饰器加上参数来决定修饰后的 `task` 对象的一些属性。

- 首先，我们可以让被修饰的函数成为 `task` 对象的绑定方法，这样就相当于被修饰的函数 `add` 成了 `task` 的实例方法，可以调用 `self` 获取当前 `task` 实例的很多状态及属性。

- 其次，我们也可以自己复写 `task` 类然后让这个自定义 `task` 修饰函数 `add` ，来做一些自定义操作。

下面用几个例子说明

#### 根据任务状态执行不同操作

任务执行后，根据任务状态执行不同操作需要我们复写 task 的 `on_failure`、`on_success` 等方法：

我们自定义一个继承自`celery.Task`的类，修改上面两个方法, 然后把他装饰在`divide`函数上，如果y是0则抛出错误

```python tasks.py
# tasks.py
class MyTask(celery.Task):
    def on_success(self, retval, task_id, args, kwargs):
        print 'task done: {0}'.format(retval)
        return super(MyTask, self).on_success(retval, task_id, args, kwargs)
    
    def on_failure(self, exc, task_id, args, kwargs, einfo):
        print 'task fail, reason: {0}'.format(exc)
        return super(MyTask, self).on_failure(exc, task_id, args, kwargs, einfo)
 
@app.task(base=MyTask)
def divide(x, y):
    if y == 0:
        raise ZeroDivisionError
    return x / y
```

同时修改`trigger.py`引入并调用`divide`,当传入2,0时就会有如下的输出:

```
23-8f9c-f1829311b5f7]
[2018-02-07 16:50:15,479: WARNING/ForkPoolWorker-1] task fail, reason:
[2018-02-07 16:50:15,479: ERROR/ForkPoolWorker-1] Task tasks.divide[903e16d1-846b-4123-8f9c-f1829311b5f7] raised unexpected: ZeroDivisionError()
Traceback (most recent call last):
  File "/home/cg/program/celery-tutorial/celeryenv/lib/python3.6/site-packages/celery/app/trace.py", line 374, in trace_task
    R = retval = fun(*args, **kwargs)
  File "/home/cg/program/celery-tutorial/celeryenv/lib/python3.6/site-packages/celery/app/trace.py", line 629, in __protected_call__
    return self.run(*args, **kwargs)
  File "/home/cg/program/celery-tutorial/tasks.py", line 27, in divide
    raise ZeroDivisionError
ZeroDivisionError
[2018-02-07 16:50:59,722: INFO/MainProcess] Received task: tasks.divide[66738a51-bc0f-439e-b7e0-5aadd21ef13e]
[2018-02-07 16:50:59,723: WARNING/ForkPoolWorker-1] task done: 0.5
[2018-02-07 16:50:59,724: INFO/ForkPoolWorker-1] Task tasks.divide[66738a51-bc0f-439e-b7e0-5aadd21ef13e] succeeded in 0.0006447387859225273s: 0.5
```

## Flower

### 安装

```pip install flower```

### 使用

