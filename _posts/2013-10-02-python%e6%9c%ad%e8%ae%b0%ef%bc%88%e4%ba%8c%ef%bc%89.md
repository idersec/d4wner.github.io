---
layout: post
title: PYTHON学习札记（二）
date: 2013-10-02 15:25
author: DEMON
comments: true
categories: coding
---
<span style="color: #ff0000;">1.stat模块:</span>
对于文件的状态能直接获取，通过集合分别赋值给几个变量（注意，不超过10项），说到底就是获取的文件描述符的状态。
<pre lang="python">import os
import time 
file = "../src/xx.txt"
def dump( st ):
    mode, ino, dev, nlink, uid, gid, size, atime, mtime, ctime = st
    print "- size:", size, "bytes"
    print "- owner:", uid, gid
    print "- created:", time.ctime( ctime )
    print "- last accessed:", time.ctime( atime )
    print "- last modified:", time.ctime( mtime )
    print "- mode:", oct( mode )
    print "- inode/dev:", ino, dev 
    st = os.stat( file )
    print "stat", file
    dump( st )</pre>
还有种类似的法子:
<pre lang="python">fp = open( file )
st = os.fstat( fp.fileno() )</pre>
相当于先获取I/O操作句柄，再获取状态，比上面一个法子稍复杂。

<span style="color: #ff0000;">2.大杂烩</span>
（1）os.path的全自动化文件名分割。
(2)集合操作
<pre lang="python">import operator
sequence = 1, 2, 4
print "add", "=&gt;;", reduce(operator.add, sequence)
print "sub", "=&gt;;", reduce(operator.sub, sequence)</pre>
其实吧，上面的亮点不是operater，前面的py内建函数reduce才是[（1,2）+3] =&gt;无限循环处理数组集合的主,编程时将给我们带来
一定的便捷性。至于operater本身，是个集合操作的好手，定位，分割，组合，确定有无，等等，非常方便。
事见：http://blog.csdn.net/lindaydk/article/details/6314444
（3）循环读取文本
<pre lang="python">import fileinput
import sys
for line in fileinput.input( "xx.txt" ):
    sys.stdout.write( "-&gt; " )
    sys.stdout.write( line )</pre>
若采用glob.glob模块可以直接匹配多个文件和路径，批量循环读取！
(4)shutil模块
可以使用shutil复制整个目录,然后删除目录。
shutil.copytree[,rmtree,copy]
(5)捕获输入输出
cStringIO和StringIO。
(6)字典和列表的添加貌似特殊构造挺麻烦，可以采用其他法子。
事见：http://www.cnblogs.com/rollenholt/archive/2011/08/08/2131053.html

<span style="color: #ff0000;">3.pickle模块的基本使用</span>
pickle模块主要特点为通过句柄就直接能完成文件操作，不知道是否类似于管道？其两个主要函数是dump()和load()。dump()函数接受一个文件句柄和一个数据对象作为参数，把数据对象以特定的格式保存到给定的文件中。当我们使用load()函数从文件中取出已保存的对象时，pickle知道如何恢复这些对象到它们本来的格式-----可以用于反弹交互，后门，socket？
<strong>待续。。。。。。。。。</strong>

后记:查阅了网上一部分资料，感谢Rollen Holt。
