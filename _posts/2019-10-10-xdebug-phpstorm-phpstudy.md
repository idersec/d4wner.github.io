---
layout: post
title:  "xdebug+phpstorm+phpstudy本地调试踩坑"
date:   2019-10-10 21:13 +0800
categories: vuln
category: vuln
---

<p>
	<span style="color:#00B050;"><strong>这两天用idea审java比较顺手，顺便也迁移了下php调试环境，从vscode迁移到jetbrains家的phpstorm。以前习惯了纯搜索字符串，通过打印调试，没有动态hook变量看起来比较费劲。迁移过程中遇到一些坑，故此记录一下。
</strong></span>
</p>

### phpstorm配置

先贴下我自己的php.ini配置【XDebug部分】：


```
[XDebug]
xdebug.profiler_output_dir="D:\phpstudy_pro\Extensions\tmp\xdebug"
xdebug.trace_output_dir="D:\phpstudy_pro\Extensions\tmp\xdebug"
zend_extension="D:\phpstudy_pro\Extensions\php\php_xdebug-2.2.5-5.6-vc11-nts-x86_64.dll"
xdebug.remote_port = 9000
xdebug.remote_autostart= On
xdebug.remote_enable = On
xdebug.idekey = phpstorm-xdebug
xdebug.remote_handler = "dbgp"
xdebug.remote_host = "127.0.0.1"
```
### 前人踩坑总结


中间遇到一个坑，网上大部分配置操作都照做了。

这里着重讲几个重要的配置，跟下面网上扒拉的图不大一致【由于没找到合适的图床，暂时只能引用网上的】：

顶栏点击File->Settings，搜索框输入debug，ideakey是我自己设置的**phpstorm-xdebug**：

![image](http://www.pianshen.com/images/875/e473bbd1c2ba3874beb5b2c1be586e3b.png)

下图是网图，我设置的是**9000**：

![image](http://www.pianshen.com/images/213/403b3aca2b50f4b3529c098af21b19f5.png)

因为是本地映射调试，下面网图的Use path mappings不要勾选，但填不填域名关系不大，我自己填的**127.0.0.1**：

![image](http://www.pianshen.com/images/497/bcad8b94b1ec8f582f0d14a176a0f439.png)

注意，最新版的phpstorm下面不是web application，而是**web page**：

![image](http://static.oschina.net/uploads/space/2014/1128/150910_uD44_174025.png)


### 设置CLI Interpreter

设置链路为：

```
Settings->Languages&Frameworks->PHP
```

设置为你引用的php.exe绝对路径即可，另外该页面的Include Path按默认的就好。


### 文件映射map报错，可以看变量，无法跟踪文件

记得在调试variables的时候，映射本地路径的时候，会要求**做文件夹路径映射**，否则会报错，提示map错误，无法定位跟踪需要调试的文件。

### PHPStorm Interpreter无法拦截

另外，还有个大坑就是，在配置时遇到interpreter无法拦截：

```
PHPStorm报错：Cannot accept external Xdebug connection: Cannot evaluate expression'isset($_SERVER['PHP_IDE_CONFIG'])';
```


我看了下网上的说法，PHPStorm这个报错，是因为对于xdebug，zend_extension和extension不能同时启用，否则拦截启用不了（vscode似乎不受影响，我测的时候是可以拦截的）。

导致此问题的一个可能原因是：服务器端的php.ini中配置了：

```
extension=/path/xdebug.dll
```

应该只保留下面一个，而且zend_extension需要绝对路径，extension不需要（至少win下是如此）：

```
zend_extension=/path/xdebug.dll
```

原文是xdebug.so，按理说win下应该改成xdebug.dll，网上答案全tm爬虫抄的。

这里我受了误导，耽误了很长时间。因为我发现，鄙人在win下的php.ini针对xdebug，只配置了zend_extension，并没有如网上所述去设置extension。

那么为什么还会出现这种情况呢？

后来终于想明白了，最开始，我在phpstudy界面自己加载了扩展，当时就直接：

```
网站->管理->PHP扩展->勾选php_xdebug
```

但是这里的配置，我所使用的php版本的php.ini里是看不到的，也许是引用的全局配置，具体在哪儿我没搞明白。

反正因为这个重复了，但是vscode没有影响，在PHPStorm就直接无法拦截了。

当我**取消勾选**这里的**xdebug扩展（php_xdebug）**时，问题就解决了。


#### xdebug下断点超时

在调试的时候，看着看着variables突然就没了，回网页一看，结果发现页面超时500。
尝试修改php.ini的超时配置，并没有卵用：

```
max_execution_time=600
```
然后尝试直接在Apache（或者其他webserver）里做配置，成功解决超时问题：

```
FcgidIOTimeout 3600
```

QA:
```
明天试试在引用处下断点，因为多空间引用可能走不到那一点，超时问题咋解决?
```


### 参考资料

[《waiting for incoming connetcion with ide key 17142 问题解决
》](http://www.pianshen.com/article/7930277434/)

[《MAMP 与 PhpStorm 远程调试
》](http://ju.outofmemory.cn/entry/331038)

[《使用PHPStorm+XDebug搭建单步调试环境》](https://www.jb51.net/article/128545.htm)

[《PHP Warning: Xdebug MUST be loaded as a Zend extension in Unknown on line 0 解决办法》](https://blog.csdn.net/yizhou35/article/details/17043925)


[《PhpStorm和WAMP配置调试参数，问题描述Error. Interpreter is not specified or invalid. Press “Fix” to edit your pro》](https://blog.csdn.net/universee/article/details/74516250)

[《phpstorm中配置真正的远程调试(xdebug)》](https://www.cnblogs.com/yjken/p/6555438.html)

