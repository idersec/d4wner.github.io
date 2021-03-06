---
layout: post
title: PHP学习札记（二）
date: 2013-09-24 12:07
author: DEMON
comments: true
categories: coding
---
<span style="color: #00B050;">1.对于assert的分析：</span>
估计不少人都知道，assert这个函数，此为php官方声明文档中，建议用于debug的。在执行时，效果类似于eval，返回true or false;只不过可容错，可执行不是特别符合规范的语句。说白点就是php中的@容错。当配合用上callback的方法，可以知道具体的出错信息。
个人觉得需要注意的几个小点：
（1）当被程序员用于调试时，最好调试完毕用其他替换，毕竟assert虽好，使用时需要开启开关ASSERT_ACTIVE以此来控制是否开启debug，实际运营环节是很容易被disable的。
（2）assert使用时很灵活，其本质是判断一个表达式是否成立，如用于奇技淫巧，可用于加密后对各种WAF的绕过。
（3）另外，注意下assert_option()的配置，几个细节很值得研究：
EX:assert_option()
默认值ASSERT_ACTIVE=1 //Assert函数的开关
ASSERT_WARNING =1 //当表达式为false时，是否要输出警告性的错误提示,issue a PHP warning for each failed assertion
ASSERT_BAIL= 0 //是否要中止运行；terminate execution on failed assertions
ASSERT_QUIET_EVAL= 0 //是否关闭错误提示，在执行表达式时；disable error_reporting during assertion expression evaluation
ASSERT_CALLBACK= (NULL) // 是否启动回调函数 user function to call on failed assertions

<span style="color: #00B050;">2.劫持引出的来源分析：</span>
前段时间劫持快照蛮火，不知道现在如何，博主挺笨又胆儿小，没有作深入研究。至于劫持脚本如何判断爬虫可以借鉴一下。
（1）.htacess，一般放在网站的根目录，抑或htdocs下也是较常见的。主要用于对目录配置的二次覆盖，以及封禁特定IP地址的用户、只允许特定IP地址的用户。即使服务器是比较安全的那种，可以考虑不要使用它。打个比方，一个站群服务器，如果AllowOverride启用了.htaccess文件，则Apache需要在每个目录中查找.htaccess文件，因此，无论是否真正用到，启用.htaccess都会导致性能的下降。另外，对每一个请求，都需要读取一次.htaccess文件，并且如果启用的话，Apache必须在所有上级的目录中查找.htaccess文件。
（2）$_SERVER['HTTP_REFERER']，其实重点是HTTP_REFFRER,在接收post包时，可能会出现，一般对应的是http的消息报头referer项，可以判断的是网页的定向来源，这在抓取post包时可以抓到。
（3）HTTP_USER_AGENT，也是$_SERVER的参数，可判断主机来源，这里需要注意的是，与上不同，上面的reffer判断的是主机地址，这个则需要匹配主机关键词如google、yahoo神马的。
（4）X-Forwarded-For，介个是博主自己添加的，一般在代表客户端，在服务器使用了负载均衡和代理时会使用，有时也会拿来判断真实IP或者来源（比如某些略坑的hack游戏）.

<span style="color: #00B050;">3.不安全因素过滤引发的思考</span>
（1）htmlspecialchars()，许多php程序猿会用到的一个过滤器，此函数把一些预定义的字符转换为 HTML实体。如：" （双引号） 成为 "' （单引号） 成为 &amp;#039。输出的转义的字符无法对浏览器造成跨站攻击(记得编码时双单引号的可编码性是可选的)。
（2）addslashes()，对字符进行转义。当magic_quotes_gpc=off时，此函数会对输入的字符进行加斜杠处理；当为on时，使用了此函数，输出应该用stripslashes()去掉多余的反斜杠。当然在这里我们需要注意宽字符‘縗’造成截断的影响（这里不再赘言）。值得一提的是，magic_quotes_gpc特性在PHP5.3.0中已经废弃并且在5.4.0中已经移除了，这对于转义显得更为灵活。
（3）参数化查询防止SQL注入（这里插一句，是补充的是mysql相关，与php并无太大关系。）,或许部分朋友会觉得这个在mssql开发里听到过，其实mysql里也可以实现的。那么，参数化查询是如何防止注入的呢。参数化查询中，可以重用执行计划，并且如果重用执行计划的话，SQL所要表达的语义就不会变化，所以就可以防止SQL注入。
MySqlConnection   myconn   =   new   MySqlConnection(constr);
 MySqlCommand   mycomm   =new   MySqlCommand(constr);
 strsql= "select   cust_id,cust_name   from   vt_frmcust   where   punid=？str1 ";
 mycomm.Parameters.Add( "？str1 ",MySqlDbType.VarChar,32);
 mycomm.Parameters[ "？str1 "].Value=Request[ "unid "]; 
在此之前，SQL语句如果中如果预计掺入了注入，可以先考虑sql语句拼接+过滤，不过动态执行SQL同样有风险。
mysql中的动态执行：
mysql&gt; SET @a=1;
mysql&gt; PREPARE STMT FROM "SELECT * FROM tbl LIMIT ?";
mysql&gt; EXECUTE STMT USING @a;
预定义一个语句，并将它赋给 STMT,"？"参数带入，动态执行。PS: MySQL VER&gt;5.0.13,可用于存储过程，但仍不支持用于自定义函数。参考资料中有几个讨论有点意思，机油们可以看看：
<span style="color: #00ffff;">http://bbs.csdn.net/topics/380206077</span>
<span style="color: #00ffff;"> http://cs.now.cn/html/FAQ/service/201302/27-7014.html</span>
<span style="color: #00ffff;"> http://database.51cto.com/art/201301/377069.htm</span>
<span style="color: #00ffff;"> http://blog.csdn.net/moshuchao/article/details/2153342#</span>


