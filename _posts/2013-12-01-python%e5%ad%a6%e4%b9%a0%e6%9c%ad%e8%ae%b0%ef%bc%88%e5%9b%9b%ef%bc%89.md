---
layout: post
title: PYTHON学习札记（四）
date: 2013-12-01 03:19
author: DEMON
comments: true
categories: coding
---
1.Python抓取页面中超链接(URL)的三中方法比较(HTMLParser、pyquery、正则表达式) 
2.Python提供了原始字符串，顾名思义，就是保留原始字符的意思，不对反斜杠及反斜杠后面的字符进行转义，声明原始字符串的方法是在字符串前面加上'r'或者'R'。
3.findall里面可以直接使用正则，不考虑转义？
4.re.X re.I 
5.?i ?:--->匹配大小写  
6.Python中最常用的从键盘获取输入的函数是raw_input()和input()。最好使用前者，统一以字符串形式返回。
7.print打印输出可以'前面加编码
8.urlopen后只能进行一次read，第二次为str类型，open选项加上timeout
9.except错误类型最好统一Exception，以免意外错误。
10.Python 报错'ascii' codec can't decode byte 0xe5 in position 0: ordinal not in range(128)，尝试decode，如不能写入尝试encode成byte流。
11.Python抓取页面中超链接(URL)的3中方法比较(HTMLParser、pyquery、正则表达式)==>http://www.myexception.cn/HTML-CSS/639814.html
12.python判断是否为空可用if xx is None或者if not xx,后者应用相对更广且效果更佳。
13.按行读取url读取去掉\n
<pre lang="python">
for line in file.readlines():
    line=line.strip('\n')
</pre>
14.对于urlsplit、urlparse、urlunparse的详细介绍：
http://www.cnblogs.com/huangcong/archive/2011/08/31/2160633.html
http://hi.baidu.com/springemp/item/64613c7457731517d0dcb3a7
15.获取网页状态码，需要requests模块http://www.oschina.net/code/snippet_862981_23032
16.local variable 'xx' referenced before assignment 需要全局
17.对于url不变，内容跳转的，也就是那种防扫描的，可以用urllib直接open,catch报错即可。
ex：http://segmentfault.com/q/1010000000095769 Nginx配置
18.urllib2.geturl() 可以拿到跳转后的最终页面，302？
19.如何获取网页状态码：
<pre lang="python">
f=urllib.urlopen("xxxxxx") 
print f.getcode() 
==========================
import requests
def getStatusCode(url):
r = requests.get(url, allow_redirects = False)
return r.status_code #使用的requests库在2.7或者2.6好像是没有的
===========================
conn = httplib.HTTPConnection("192.168.1.212");      
#开始进行数据提交   同时也可以使用get进行      
conn.request(method="POST",url="/newsadd.asp?action=newnew",body=params,headers=headers);      
#返回处理后的数据      
response = conn.getresponse();      
#判断是否提交成功      
if response.status == 302:  
</pre>
20.httplib request的用法, getresponse() 用以进行返回数据
21.get_header探测远程文件是否存在可能需要细看是否取空




