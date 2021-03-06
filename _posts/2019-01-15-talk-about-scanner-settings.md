---
layout: post
title:  "分布式扫描填坑杂谈"
date:   2019-01-15 15:49:21 +0800
categories: scanner
category: scanner
---

<p>
	<span style="color:#00B050;"><strong>前段时间团队新加了几台服务器，因为现行的扫描器是支持分布式的，所以及时进行了分布式部署。结果在调试分布式配置时，还是遇到许多坑，忙完一阵子闲下来了，正好整理下记录。</strong></span>
</p>

在本文中，将简单谈谈我这边采用的技术栈，有关对分布式填坑的经历，大家注意这里依旧是采用的python code。

- celery库版本

以root权限启动root的问题，低版本不识别，会报错，这里使用的是celery==3.1.2x。
```
from celery import Celery, platforms
platforms.C_FORCE_ROOT = True

```

- redis库版本

版本不合适，或者worker和server版本不对等的话，可能出现celery worker 开启不久就offline的问题，或者redis会不时的掉线重连。

这里把redis库的版本，从原来的redis==3.0.x降级到redis==2.10.x，基本解决了上述bug。

需要注意的是，在celery高版本的时候（比如4.x），可能会需要匹配3.x的redis库，大家看情况而定。


- 重连机制

我们这里在后端存储结果，考虑到复杂sql交叉调用，还是采用的关系数据库mysql。

局域网中数据库写入，需要考虑重连机制，常用的如MySQLdb库，原生无重连机制，需要自己重写。

这里推荐两个模块，都支持重连：
```
torndb
facebook开源的一个基于MySQLdb二次封装的一个mysql模块。
pymysql
比较常用，需要python3。
```

- 加强存取稳定性

如果想要加强局域网数据存取的稳定性，除了把上述的超时和重连配置好以外，暂时有两个办法：

> 第一：可以将存入局域网数据库服务器失败的数据，暂存本地关系数据库。采用定时调度器，或者监控网络空闲时，再进行同步
。

> 第二：可以将命令或者数据直接写入本地中间件，当监控到网络空闲时间，自动同步。

另外，笔者这方面经验比较浅，欢迎运维大佬和coding大佬多多指教下这一块儿有没有其他可优化的点。


- 局域网调试

检测网络状态，可以看出ping其他worker是通的：
```
>>> app = Celery('test', backend='redis://random_host:6379/0', broker='redis://random_host:6379/0')
>>> app.control.ping(timeout=0.5)

[{'worker1.example.com': 'pong'},
```

查看时区差异:
```
>>> from celery.utils.timeutils import utcoffset
>>> utcoffset()

0
```
库版本不同步等原因，可能造成worker和server的时间不同步。而且时间不同步，似乎会导致celery的超时机制不可用。

我们可以安装ntp服务，自动更新时间，同步server和worker的指定python库版本，示例命令如下【每个系统情况不一定适用】：

```
apt-get install ntp
service ntp start
#手动更新时间
date -R

```

监控woker状态的一些点，可以参考这篇文章：
```
https://fangchichen.github.io/fangchichen.github.io/2018/08/08/celery%E5%87%BA%E7%8E%B0worker%E5%BC%82%E5%B8%B8offline%E6%83%85%E5%86%B5/
```

- celery关键配置

还是那句话，各系统情况不同，
这里仅仅贴出几个关键点,大家觉得可以优化的地方可以讨论下：

```
#任务预取功能，就是每个工作的进程／线程在获取任务的时候，会尽量多拿 n 个，以保证获取的通讯成本可以压缩。
CELERYD_PREFETCH_MULTIPLIER = 1

#这个表示每个工作的进程／线程 在执行 n 次任务后，主动销毁，之后会起一个新的，主要解决一些资源释放的问题。
CELERYD_MAX_TASKS_PER_CHILD = 1

#不存取返回结果，加快响应速度。
CELERY_IGNORE_RESULT=False

#该配置可以保证task不丢失，中断的task在下次启动时将会重新执行。
TASK_REJECT_ON_WORKER_LOST = True
#不会多拿任务，只有当worker完成了这个task时，任务才被标记为ack状态。
#只有当worker完成了这个task时，任务才被标记为ack状态
CELERY_ACKS_LATE = True

#解决时区同步问题
CELERY_TIMEZONE = 'Asia/Shanghai'
CELERY_ENABLE_UTC = True
USE_TZ = True

#broker的连接超时时间。
BROKER_CONNECTION_TIMEOUT = 20

#如果确认是因为当前worker的并发是prefork（多进程）,并且可能是由于死锁原因造成，4.0之后的版本不支持。
CELERYD_FORCE = True

#任务超时会分配给其他worker
BROKER_TRANSPORT_OPTIONS = {'visibility_timeout': 3600}

#禁用所有速度限制，如果网络资源有限，不建议开足马力。
#CELERY_DISABLE_RATE_LIMITS = True
#CELERY_ACKS_LATE = True
#CELERY_IGNORE_RESULT = True

#这个表示保存任务结果的时长，这个时间会被设置到 backend 里面
#CELERY_TASK_RESULT_EXPIRES = 3600

```

- celery命令行参数

Queue还是建议启用的，方便清空，也方便指定queuename运行特定任务。
比如配置文件里这么写:
```
CELERY_QUEUES = (
            Queue('default', Exchange('default'), routing_key='default'),
            Queue('wakaka', Exchange('wakaka'), routing_key='wakaka'),
            )
```

那么，命令行可以这么输入：

```
celery -A test worker -E -l INFO -n workername -Q wakaka --concurrency=4
```

注意，--concurrency的值是并发进程数，这是由你的CPU个数决定性能的，不要设太高。

win下的话，在高版本celery 4.x，默认的是prefork，报错解决方法如下：
```
pip install eventlet
celery -A proj worker -l info -P eventlet

```

最后，建议大家不要在win下运行celery，似乎4.x以后某个版本已经放弃支持，而且win下有很多坑没法填。

- 任务调度框架

关于任务调度这一块儿，除了celery，感觉dramatiq和rq的坑会少些，以后会抽空来谈谈。

### 结语

以上配置和分析内容，是摸索和查资料得来的，感谢前辈们的开源共享。