---
layout: post
title:  "安全运维优化思考"
date:   2019-07-13 23:40 +0800
categories: operation
category: operation
---

<p>
	<span style="color:#DAA520;"><strong>近期由乙方安服实验室，转入了甲方的业务安全部门。在接触了一部分业务安全的运维工作后，也做了些对于自身工作的优化方向的思考。</strong></span>
</p>

#### 资产统计与变更
这段时间正好碰上了FastJson漏洞爆发。由于笔者所在的甲方，属于有一定规模的互联网公司，所以近期也在连夜配合各业务部门进行漏洞修补。

虽然公司本身具有强大的统计平台，也具有较为成熟的资产规范，但还是花了大量时间去统计、升级那些受影响的资产。

举个例子，自家公司的It资产已经接入了自动化运维，采用了基于的ES集群的平台。资产的变更能够及时以*key为基准，计入全局搜索，方便以后搜索和资产统计，以及后续的自动化升级和部署。

但这种统一部署变更的法子，也存在部分痛点。其中的一个就是，一旦基于ES部署平台和Agent自身存在问题，如果没有得到及时修复的话，其自身不小的体量，可能会影响全局资产的修复进度。

因此，拥有多份资产升级/变更/检测方案，能做到轻量/重量级方案互相制约，窃以为还是有必要的。

另外，如果做好了内网Git资产梳理，通过代码扫描定位可能存在的漏洞和服务，也是能辅助统计受影响的资产的。

#### 自动化检测认证
对于大型甲方，如果安全团队在研究好新出的漏洞Poc，对IT资产安全进行自检的时候，如果按正常流程去做，可能首先应该是先进行任务申请上报，然后向全集团发邮件，最后再进行扫描。

不过，在碰上比较紧急的漏洞应急时，在跨部门协作的情况下，经常会来不及走完所有流程。

笔者也曾见过研发、运维等部门，因为突然查到攻击Log，半夜一惊一乍的，去找安全部门验证攻击来源是否属于内部自检。

那么针对这一点，如何做去做优化呢？

不少大型企业，有时会采用统一的签名和加密机制，或者直接构建单独的平台，用于保证传输加密的可信认证。

笔者窃以为，这点是可以借鉴的。如果能做好一定范围内的成本控制，在每次做自动化安全检测的时候，将加密认证信息加入检测数据包头部，以用作内部安全检测的授信，各个BU会更加轻松的识别出真实的攻击事件。

#### 敏感数据泄漏控制
防止敏感数据的泄漏，以及进行事后的责任追溯，一直是甲方比较重视的点，据悉大致有这样几种方案：

1.DLP数据防泄漏 

DLP软件一般是为了定位公司敏感数据外发行为，对于数据流量内容进行监控审计，现在市面上也有了不少成熟的合规产品。

2.堡垒机 

堡垒机上具有监控，限制数据传输和全程录屏等功能，配合查询系统的水印功能，也能在一定程度上防止数据泄漏，以及对泄漏源进行追溯。

3.数据脱敏 

在存储和展示敏感数据的时候，本身应该做好脱敏操作，对于数据进行加密存储和非完全展示，防止内鬼和意外泄漏事件发生。

4.数据监控 

虽然如同Github监控和舆情监控一般的产品，并不能有效抑制数据泄漏。但在防止数据扩散，以及追溯数据泄漏来源的层面来看，还是比较有用的。

如果能综合利用多类产品，再加上企业本身的安全管理规范，应该是能够在一定程度上保证数据安全的。

#### 产品检测流程
在原来的乙方安全测试岗位，如果需要对产品做安检的时候，随便去咨询个资深的相关产品、售前或者研发，基本上都能问出个所以然来。

然而到了现在的甲方安全运维岗，可就厉害了。在工作流程细化和文档化以后，需要做安全检测时，得挨个询问多个QA/RD/PM，一点一点把他们的需求和设计方案抠出来，最后还得去找API文档自己做补充和完善，才能进行下一步的操作。

笔者这两天还去拜访了一家非互联网甲方，跟那边负责安全的Leader朋友聊了下，产品上线合规的紧要性，确实是远优先于安全合规的，当然这个也是不得已而为之。

总的来说，合规化有益于流程梳理，简化有益于加速产品上线，也算各有各的好处吧。

当然笔者见识有限，窥一斑而不得见全豹。但总的来说，确实可以根据不同产品的实际情况，去对流程进行一些灵活变通。

#### 后记
以上只是简单谈了一些感受，这方面的工作资历尚浅，期待各路读者斧正和指教。

