---
layout: post
title:  "域名采集爬虫"
date:   2017-10-19 20:59:18 +0800
categories: spider
category: spider
---
<p>
    <span style="color:#00B050;"><strong>
    前段时间在研究扫描器的问题，在涉及域名采集这块时， 突然有了一些特别的想法，所以单独将这个模块提取出来，做成了一套独立系统。
    </strong></span>
</p>

对于域名采集技术，市面上大概有这样几种：

* DNS域传送泄露
* 接口查询（包括aizhan，shodan等）
* 字典枚举
* Github泄露
* 搜索引擎泄露
* SSL/TLS证书泄露
* DNS历史记录泄露
* 置换扫描枚举法
* 互联网自治系统号码(ASN)枚举
* NSEC记录枚举
* 互联网项目数据集枚举


不过这里要谈的显然不是这种，尽管自研的扫描器里已经集成了前面说的这些，今天的主角是域名采集爬虫。

那么何谓域名采集爬虫呢，其实也是老概念。我们要先从一个URL或者多个URL入口进行爬取。在开始，需要设定好爬行深度和允许爬行的根域名，通过不断地迭代存取和过滤，直到爬取完所有可触及的域名为止。

这样的域名采集方法，较其他技术优势在于：

* 能采到不被搜索引擎收录的可用域名。
* 囊括范围广，除去CDN影响，能更多地去收集目标资产。
* 细致入微，在域名采集普及的时代，厂商一般子域名都会多加注意起名，这样做能多收获一些隐藏域名。
* 可分布式，相对于普通的单机大字典爆破，可以选择性节省更多的时间。

当然讲了这么多，其劣势也很明显：

* 将耗费更多的时间和各类资源，即使采用了分布式，由于需要考虑部分目标主机的脆弱性，建立的连接数还是有限的。
* 容易被封，WAF和各种IDS的规则在检测到爬虫后，可能会直接封禁IP，不过这点可以通过技术手段缓解。
* 不易部署，相对于常用的域名接口查询和爆破，这点的劣势还是相当明显的。
* 比较鸡肋，没有太多Team会专门为了爬取域名单独建立一个可用的爬虫体系，一般是扫描器只兼容了传统接口。

优劣对比还是比较明显的，这里不多做探讨，下面纯技术介绍。

这里的系统采用的是scrapy+django+mongodb，每采集好一批数据后，将存入mongodb，其中的数据包括：

```
页面源URL
子域名
根域名

```

后续其他项目将会解析生成的所有子域名，最后再进行资产汇总计算。这些本不属于此项目，不再细说。

另外，在采集过程中，爬虫会在中间件（合规的做法如此，笔者则是直接在爬虫文件里进行的过滤），根据条件过滤重复和不合规的URL。

代码其实很简单，现在已有的是前台查询+后台爬虫（命令行）的形式。后期如果项目可用，会加上前台scrapyd发布任务+MMQ分布式队列缓存。

另外，以后也考虑集成传统接口、泄露查询和枚举模块。

至于为啥现在没有做，因为懒。

代码由于还需要优化，暂时没有放出，以后会补上。

待续。