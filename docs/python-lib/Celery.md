# Celery + Redis/RabbitMQ + Flower

## 目录

* [Redis](#Redis)
* [Celery](#Celery)
  * [简介](#简介)
  * [安装Celery](#安装Celery)
  * [示例](#简单示例)
  * [进阶用法](#进阶用法)
* [Flower](#Flower)

## Redis

请参考[Redis](../databases/redis.md)部分

## Celery

Celery是分布式任务队列，能实时处理任务， 同时支持task scheduling。
Celery工作原理如下：

1. celery client发送message给`broker`

2. `worker`从`broker`中消费消息，并将结果存储在result_end中

### 简介

* Brokers

`brokers` 中文意思为中间人，在这里就是指任务队列本身，Celery 扮演生产者和消费者的角色，`brokers` 就是生产者和消费者存放/拿取产品的地方(队列)

常见的 brokers 有 `rabbitmq`、`redis`、`Zookeeper` 等

* Result Stores / Backend

顾名思义就是结果储存的地方，队列中的任务运行完后的结果或者状态需要被任务发送者知道，那么就需要一个地方储存这些结果，就是 Result Stores 了

常见的 backend 有 `redis`、`Memcached` 甚至常用的数据库(`mysql`, `postgresql`等)都可以。

* Workers

就是 Celery 中的工作者，类似与生产/消费模型中的消费者，其从队列中取出任务并执行

* Tasks

就是我们想在队列中进行的任务，一般由用户、触发器或其他操作将任务入队，然后交由 workers 进行处理。

下面我们用`redis`作为`celery`的`broke`r和`backend`作为使用示例

### 安装Celery

```python pip安装celery以及Python对redis的支持
pip install celery
pip install redis
```

### 简单示例

我们用`redis`作为`broker`和`backend`，下面我们来编写`tasks`和`workers`

* tasks.py

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

* trigger.py

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

* 首先，我们可以让被修饰的函数成为 `task` 对象的绑定方法，这样就相当于被修饰的函数 `add` 成了 `task` 的实例方法，可以调用 `self` 获取当前 `task` 实例的很多状态及属性。

* 其次，我们也可以自己复写 `task` 类然后让这个自定义 `task` 修饰函数 `add` ，来做一些自定义操作。

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

#### 绑定任务为实例方法

绑定任务为实例方法，执行中的任务可以获取到自己执行任务的各种信息，可以根据这些信息做很多其他操作。
关于 `celery.task.request` 对象的详细信息可以查看[task-request-info](http://docs.celeryproject.org/en/latest/userguide/tasks.html#task-request-info)

```python tasks.py
# tasks.py
from celery.utils.log import get_task_logger

logger = get_task_logger(__name__)

@app.task(bind=True):
def minus(self, x, y):
    logger.info(self.request.__dict__)
    return x - y
```

重新运行结果如下, 可以看到request的dict信息：

```log
[2018-02-08 10:13:51,396: INFO/MainProcess] Received task: tasks.minus[75354357-7604-4ce2-8f7a-6020009fd924]  
[2018-02-08 10:13:51,397: INFO/ForkPoolWorker-1] tasks.minus[75354357-7604-4ce2-8f7a-6020009fd924]: {'lang': 'py', 'task': 'tasks.minus', 'id': '75354357-7604-4ce2-8f7a-6020009fd924', 'eta': None, 'expires': None, 'group': None, 'retries': 0, 'timelimit': [None, None], 'root_id': '75354357-7604-4ce2-8f7a-6020009fd924', 'parent_id': None, 'argsrepr': '(1, 2)', 'kwargsrepr': '{}', 'origin': 'gen11241@master', 'reply_to': '85c9beec-d626-385d-99f3-7dee1b7fc0f2', 'correlation_id': '75354357-7604-4ce2-8f7a-6020009fd924', 'delivery_info': {'exchange': '', 'routing_key': 'celery', 'priority': 0, 'redelivered': None}, 'args': [1, 2], 'kwargs': {}, 'hostname': 'celery@master', 'is_eager': False, 'callbacks': None, 'errbacks': None, 'chain': None, 'chord': None, 'called_directly': False, '_protected': 1}
[2018-02-08 10:13:51,399: INFO/ForkPoolWorker-1] Task tasks.minus[75354357-7604-4ce2-8f7a-6020009fd924] succeeded in 0.0018258970230817795s: -1

```

#### 任务状态回调

当我们有个耗时时间较长的任务进行时一般我们想得知它的实时进度，这里就需要我们自定义一个任务状态用来说明进度并手动更新状态，从而告诉回调当前任务的进度。

对于 Celery 其内建任务状态有如下几种：
参数 | 说明
---|---
PENDING | 任务等待中
STARTED | 任务已开始
SUCCESS | 任务执行成功
FAILURE | 任务执行失败
RETRY | 任务将被重试
REVOKED | 任务取消

用一个简单的例子说明,`tasks.py`中增加：

```python tasks.py
# tasks.py
import time

@app.task(bind=True)
def test_msg(self):
    for i in range(1, 11):
        time.sleep(0.1)
        self.update_state(state="PROGRESS", meta={'percent': i*10})
    return 'finished'
```

`trigger.py` 中增加:

```python trigger.py
# trigger.py
from task import add, test_msg
import sys

def process_msg(body):
    res = body.get('result')
    if body.get('status') == 'PROGRESS':
        sys.stdout.write('\r任务进度: {0}%'.format(res.get('percent')))
        sys.stdout.flush()
    else:
        print('\n', res)

result = test_msg.delay()
print result.get(on_message=process_msg, propagate=False)
```

#### 定时/周期任务

`Celery` 进行周期任务也很简单，只需要在配置中配置好周期任务，然后在运行一个周期任务触发器(`beat`)即可：

新建`Celery` 配置文件`celery_config.py`

```python celery_config.py
# celery_config.py
from datetime import timedelta
from celery.schedules import crontab

CELERYBEAT_SCHEDULE = {
    'ptask': {
        'task': 'tasks.period_task',  # period_task为自己定义的任务名
        'schedule': timedelta(seconds=5),
    },
}

CELERY_TIMEZONE = 'Asia/Shanghai' # 时区为中国
```

在`tasks.py`中增加周期执行的任务

```python tasks.py
# tasks.py
app.config_from_object('celery_config') # 导入配置

@app.task(bind=True)
def period_task(self):
    print('period task done: {}'.format(self.request.id))
```

然后重新运行`worker`,接着运行`beat`：`celery -A tasks beat`

```log
[2018-02-08 11:03:41,368: WARNING/ForkPoolWorker-1] period task done: e64252ea-66fa-4c5c-ba71-74eada56cf17
[2018-02-08 11:03:41,369: INFO/ForkPoolWorker-1] Task tasks.period_task[e64252ea-66fa-4c5c-ba71-74eada56cf17] succeeded in 0.00070146843791008s: None
[2018-02-08 11:03:46,367: INFO/MainProcess] Received task: tasks.period_task[88fa4886-d6d4-4aea-92d5-b7e876f75737]  
[2018-02-08 11:03:46,368: WARNING/ForkPoolWorker-1] period task done: 88fa4886-d6d4-4aea-92d5-b7e876f75737
[2018-02-08 11:03:46,369: INFO/ForkPoolWorker-1] Task tasks.period_task[88fa4886-d6d4-4aea-92d5-b7e876f75737] succeeded in 0.0007591797038912773s: None

```

#### 链式任务

有些任务可能需由几个子任务组成，此时调用各个子任务的方式就变的很重要，尽量不要以同步阻塞的方式调用子任务，而是用异步回调的方式进行链式任务的调用：

错误示范:

```python chain_wrong
@app.task
def update_page_info(url):
    page = fetch_page.delay(url).get()
    info = parse_page.delay(url, page).get()
    store_page_info.delay(url, info)

@app.task
def fetch_page(url):
    return myhttplib.get(url)

@app.task
def parse_page(url, page):
    return myparser.parse_document(page)

@app.task
def store_page_info(url, info):
    return PageInfo.objects.create(url, info)
```

正确示范1：

```python chain1
def update_page_info(url):
    # fetch_page -> parse_page -> store_page
    chain = fetch_page.s(url) | parse_page.s() | store_page_info.s(url)
    chain()

@app.task()
def fetch_page(url):
    return myhttplib.get(url)

@app.task()
def parse_page(page):
    return myparser.parse_document(page)

@app.task(ignore_result=True)
def store_page_info(info, url):
    PageInfo.objects.create(info=info, url=url)
```

正确示范2：

```python apply_async
fetch_page.apply_async((url), link=[parse_page.s(), store_page_info.s(url)])
```

链式任务中前一个任务的返回值默认是下一个任务的输入值之一 ( 不想让返回值做默认参数可以用 si() 或者 s(`immutable=True`) 的方式调用 )。

这里的 s() 是方法 `celery.signature()` 的快捷调用方式，`signature` 具体作用就是生成一个包含调用任务及其调用参数与其他信息的对象，个人感觉有点类似偏函数的概念：先不执行任务，而是把任务与任务参数存起来以供其他地方调用。

#### 调用任务

前面讲了调用任务不能直接使用普通的调用方式，而是要用类似 `add.delay(2, 2)` 的方式调用，而链式任务中又用到了 `apply_async` 方法进行调用，实际上 `delay` 只是 `apply_async` 的快捷方式，二者作用相同，只是 `apply_async` 可以进行更多的任务属性设置，比如 `callbacks/errbacks` 正常回调与错误回调、执行超时、重试、重试时间等等，具体参数可以参考[这里](http://docs.celeryproject.org/en/latest/reference/celery.app.task.html#celery.app.task.Task.apply_async)

#### AsycResult

`AsyncResult` 主要用来储存任务执行信息与执行结果，有点类似 `tornado` 中的 `Future` 对象，都有储存异步结果与任务执行状态的功能，对于写 js 的朋友，它有点类似 `Promise` 对象，当然在 `Celery 4.0` 中已经支持了 promise 协议，只需要配合 `gevent` 一起使用就可以像写 js promise 一样写回调：

```python AsyncResult
import time
import gevent.monkey
from celery import Celery

monkey.patch_all()
app = Celery(broker='amqp://', backend='rpc')

@app.task
def add(x, y):
    return x + y

def on_result_ready(result):
    print('Received result for id %r: %r' % (result.id, result.result,))

add.delay(2, 2).then(on_result_ready)
```

要注意的是这种 `promise` 写法现在只能用在` backend` 是 `RPC (amqp)` 或 `Redis` 时。 并且独立使用时需要引入`gevent` 的猴子补丁，可能会影响其他代码。 官方文档给的建议是这个特性结合异步框架使用更合适，例如 `tornado`、 `twisted` 等。

`delay` 与 `apply_async` 生成的都是 `AsyncResult` 对象，此外我们还可以根据 `task id` 直接获取相关 task 的 `AsyncResult`: `AsyncResult(task_id=xxx)`

关于 AsyncResult 更详细的内容，可以参考[这里](http://docs.celeryproject.org/en/latest/reference/celery.result.html?highlight=result#celery.result.AsyncResult)

## Flower

### 安装

```pip install flower```

### 使用

启动上述celery进程后，运行flower服务并打开浏览器http://localhost:5555 ,默认端口为5555，你可以使用`--port`参数指定自定义的端口,远程访问可以将localhost换成你服务器的ip

```$ flower -- port=5555```

或者从Celery运行

```$ celery flower --address=127.0.0.1 --port=5555```

Broker URL和其他配置选项能够通过一个标准的Celery选项来指定:

```$ celery flower --broker=amqp://guest:guest@localhost:5672//```

例如上面的例子在启动`celery -A tasks worker -l info`进程后，可以再开一个进程`celery -A tasks flower --port=5555`，然后就可以在http://server_ip:5555/看到任务概况啦

- tasks

![flower-task](../../pics/flower-task.png)

- monitor

![flower-monitor](../../pics/flower-monitor.png)

