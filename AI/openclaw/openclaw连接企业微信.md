==原链接==: https://open.work.weixin.qq.com/help2/pc/cat?doc_id=21657

![img](https://wwcdn.weixin.qq.com/node/wework/images/helpcenter_wecomlogo_zh_2x.0d97f2a95e.png)

帮助中心



- [注册与登录](https://open.work.weixin.qq.com/help2/pc/14506)
- [企业设置与管理](https://open.work.weixin.qq.com/help2/pc/14508)
- [智能办公](https://open.work.weixin.qq.com/help2/pc/21617)
  - 
  - [数据与智能专区](https://open.work.weixin.qq.com/help2/pc/21631)
  - [智能机器人](https://open.work.weixin.qq.com/help2/pc/21632)
  - [智能总结](https://open.work.weixin.qq.com/help2/pc/21629)
  - [智能搜索](https://open.work.weixin.qq.com/help2/pc/21633)
- [连接上下游](https://open.work.weixin.qq.com/help2/pc/17218)
- [企业微信认证](https://open.work.weixin.qq.com/help2/pc/14500)
- [通讯录](https://open.work.weixin.qq.com/help2/pc/14505)
- [聊天](https://open.work.weixin.qq.com/help2/pc/14504)
- [连接微信](https://open.work.weixin.qq.com/help2/pc/14501)
- [效率工具](https://open.work.weixin.qq.com/help2/pc/17163)
- [安全管理](https://open.work.weixin.qq.com/help2/pc/20857)
- [打卡](https://open.work.weixin.qq.com/help2/pc/14502)
- [审批](https://open.work.weixin.qq.com/help2/pc/14503)
- [工作台应用](https://open.work.weixin.qq.com/help2/pc/14507)
- [家校沟通](https://open.work.weixin.qq.com/help2/pc/14510)
- [行业方案](https://open.work.weixin.qq.com/help2/pc/18275)
- [高级功能](https://open.work.weixin.qq.com/help2/pc/19812)
- [智慧硬件](https://open.work.weixin.qq.com/help2/pc/14509)
- [其他](https://open.work.weixin.qq.com/help2/pc/14511)

智能办公 智能机器人 OpenClaw如何接入企业微信智能机器人

OpenClaw如何接入企业微信智能机器人

- [一、接入前期准备](https://open.work.weixin.qq.com/help2/pc/cat?doc_id=21657#一、接入前期准备)
- [二、标准接入流程](https://open.work.weixin.qq.com/help2/pc/cat?doc_id=21657#二、标准接入流程)
  - [2.1 以长连接方式创建智能机器人，获取Bot ID和Secret](https://open.work.weixin.qq.com/help2/pc/cat?doc_id=21657#2.1 以长连接方式创建智能机器人，获取Bot ID和Secret)
  - [2.2 关联企业微信机器人与OpenClaw](https://open.work.weixin.qq.com/help2/pc/cat?doc_id=21657#2.2 关联企业微信机器人与OpenClaw)
- [三、附1：机器人通过MCP使用文档使用指引](https://open.work.weixin.qq.com/help2/pc/cat?doc_id=21657#三、附1：机器人通过MCP使用文档使用指引)
  - [3.1 功能介绍](https://open.work.weixin.qq.com/help2/pc/cat?doc_id=21657#3.1 功能介绍)
  - [3.2 配置方式](https://open.work.weixin.qq.com/help2/pc/cat?doc_id=21657#3.2 配置方式)
- [四、附2：智能表格webhook使用指引](https://open.work.weixin.qq.com/help2/pc/cat?doc_id=21657#四、附2：智能表格webhook使用指引)
  - [4.1 核心配置方式](https://open.work.weixin.qq.com/help2/pc/cat?doc_id=21657#4.1 核心配置方式)
  - [4.2 典型适用场景](https://open.work.weixin.qq.com/help2/pc/cat?doc_id=21657#4.2 典型适用场景)
- [五、附3：企业微信API调用指引](https://open.work.weixin.qq.com/help2/pc/cat?doc_id=21657#五、附3：企业微信API调用指引)
  - [5.1 获取企业ID与应用Secret](https://open.work.weixin.qq.com/help2/pc/cat?doc_id=21657#5.1 获取企业ID与应用Secret)
  - [5.2 API调用流程](https://open.work.weixin.qq.com/help2/pc/cat?doc_id=21657#5.2 API调用流程)
- [六、常见问题](https://open.work.weixin.qq.com/help2/pc/cat?doc_id=21657#六、常见问题)
  - [1、OpenClaw如何在本地终端部署安装？](https://open.work.weixin.qq.com/help2/pc/cat?doc_id=21657#1、OpenClaw如何在本地终端部署安装？)
  - [2、长连接接入和URL回调接入的区别是什么？](https://open.work.weixin.qq.com/help2/pc/cat?doc_id=21657#2、长连接接入和URL回调接入的区别是什么？)
  - [3、OpenClaw接入企微智能机器人是只能企业超管才可以操作么？](https://open.work.weixin.qq.com/help2/pc/cat?doc_id=21657#3、OpenClaw接入企微智能机器人是只能企业超管才可以操作么？)
  - [4、可以对企业成员限制OpenClaw接入机器人么？](https://open.work.weixin.qq.com/help2/pc/cat?doc_id=21657#4、可以对企业成员限制OpenClaw接入机器人么？)
  - [5、本地终端部署OpenClaw，需要先申请大模型Token吗？](https://open.work.weixin.qq.com/help2/pc/cat?doc_id=21657#5、本地终端部署OpenClaw，需要先申请大模型Token吗？)
  - [6、通过腾讯云lighthouse中部署OpenClaw并关联机器人，但运行后智能机器人没有回复，是什么问题？](https://open.work.weixin.qq.com/help2/pc/cat?doc_id=21657#6、通过腾讯云lighthouse中部署OpenClaw并关联机器人，但运行后智能机器人没有回复，是什么问题？)
  - [7、OpenClaw机器人能否在外部群/客户群中使用？](https://open.work.weixin.qq.com/help2/pc/cat?doc_id=21657#7、OpenClaw机器人能否在外部群/客户群中使用？)
  - [8、接入OpenClaw后，企业微信给的接入权限都有那些？可以实现那些功能？](https://open.work.weixin.qq.com/help2/pc/cat?doc_id=21657#8、接入OpenClaw后，企业微信给的接入权限都有那些？可以实现那些功能？)
  - [9、OpenClaw接入智能机器人后，可以识别附件吗？](https://open.work.weixin.qq.com/help2/pc/cat?doc_id=21657#9、OpenClaw接入智能机器人后，可以识别附件吗？)

### 一、接入前期准备

在将OpenClaw接入企业微信智能机器人前，需完成设备端基础环境配置，确保软硬件及应用版本符合要求：
1、企业微信客户端已安装**最新版本**；
2、本地设备已安装OpenClaw，或在**腾讯云轻量应用服务器Lighthouse（及其他云厂商服务）**中完成OpenClaw部署；
3、确认操作终端可正常执行命令行指令（本地部署用本地终端，云部署用对应服务器终端）。

### 二、标准接入流程

#### 2.1 以长连接方式创建智能机器人，获取Bot ID和Secret

通过**API模式（长连接）**创建的智能机器人，支持被动回复多条消息、主动向用户发消息，为对接OpenClaw的专属创建方式，操作步骤如下：
1、打开企业微信客户端，进入**工作台->智能机器人**，点击**创建机器人->手动创建**；
2、进入创建页面后，选择**API模式创建**（页面提示「如需使用自有系统获取成员与机器人的聊天并输出回复，可切换至API模式创建」）；
![img](https://wework.qpic.cn/wwpic3az/721668_lgQ2gLHZSoCTG29_1773394878/0)
![img](https://wework.qpic.cn/wwpic3az/909078_yNZxt-2fTSyhDxR_1773395074/0)
3、在API配置页面，选择**连接方式为「使用长连接」**（无需域名/IP即可接收消息并返回结果，区别于URL回调方式）；
4、配置完成后，页面将自动生成并展示**Bot ID**和**Secret**，妥善保存该信息（后续关联OpenClaw需使用）；
![img](https://wework.qpic.cn/wwpic3az/436435_9BtGGSbSR3KQTaE_1773395125/0)
5、补充配置机器人**可见范围**，其余项保持默认即可，API模式暂不支持预览与调试，直接保存机器人配置。
说明：长连接方式为对接OpenClaw的推荐方式，云服务商部署的OpenClaw也可选择URL回调方式快速创建机器人对接。

#### 2.2 关联企业微信机器人与OpenClaw

本地/云部署的OpenClaw均通过**终端命令**行完成插件安装、渠道添加、信息配置。

**2.2.1在腾讯云lighthouse中部署OpenClaw并关联机器人**
1、**使用腾讯云轻量应用服务器Lighthouse。**
2、**进入[轻量云](https://console.cloud.tencent.com/lighthouse)**选中已部署OpenClaw的轻量应用服务器，点击服务器实例卡片即可进入“管理实例”页面，进入“应用管理”页。
![img](https://wework.qpic.cn/wwpic3az/787280_PQ97rkI1RWiE8Gx_1773395329/0)
3、**在“模型（Models）”下需要添加模型**
![img](https://wework.qpic.cn/wwpic3az/286406_yIyXqTvLQeS2DGL_1773395408/0)
未添加模型对话中会有如下报错
![img](https://wework.qpic.cn/wwpic3az/921856_E-PRhsR4RmOBtyQ_1773395422/0)
4、在通道中，选择**企微机器人（长连接）**
依次在输入框内输入前文创建企业微信机器人时随机生成的 Bot ID 和 Secret 。填写完成后，点击界面中的添加并应用按钮，并在弹出的弹框中点击确定，稍等片刻，即可在界面中看到已经完成的企业微信机器人配置。
![img](https://wework.qpic.cn/wwpic3az/661325_OWMCcUE4Q2G_BfK_1773395557/0)
5、输入后，点击添加并应用，并完成重启。
![img](https://wework.qpic.cn/wwpic3az/720249_UVHJXSWfQheCNhx_1773375015/0)
6、回到企业微信机器人创建页面，保存并创建。即可在企业微信中与智能机器人正常对话。
![img](https://wework.qpic.cn/wwpic3az/600045_fREPFsrNSyKyKTp_1773395632/0)
7、如配置完成后未能找到机器人，可在以下路径中找到：工作台->智能机器人->详情->去使用->发消息
![img](https://wework.qpic.cn/wwpic3az/576923_AQEVY0JKRHqYYzf_1773395656/0)
![img](https://wework.qpic.cn/wwpic3az/119448_mhg4PvJURjmF5tI_1773395677/0)

**2.2.2在本地终端部署OpenClaw并关联机器人**
1、**安装企微插件**：打开本地终端，打开终端，依次执行如下命令

```
npx -y @wecom/wecom-openclaw-cli install
```

2、**企微扫码接入**：选择扫码接入，并使用企业微信扫描二维码
![img](https://wework.qpic.cn/wwpic3az/467099_14j8CUPPSzuA9GF_1773464998/0)

3、**一键创建机器人**：无需更多配置，只需点击一键创建机器人
![img](https://wework.qpic.cn/wwpic3az/414704_0nL36rgOQoiWZff_1773465043/0)

4、**自动接入OpenClaw**：成功创建机器人后，无需手动配对，即可自动接入本地openclaw，你可以点击「前往使用」开始和小龙虾对话了
![img](https://wework.qpic.cn/wwpic3az/713573_FdPc31eGRvWlrJA_1773465089/0)
![img](https://wework.qpic.cn/wwpic3az/248748_YLOE3KtlT6Kp7cS_1773465098/0)

### 三、附1：机器人通过MCP使用文档使用指引

**前提：请升级为最新的企业微信OpenClaw插件。如果还未升级，可在本地终端中输入以下代码：**

```
openclaw plugins update wecom-openclaw-plugin
```

#### 3.1 功能介绍

3月13日更新的5.0.7版本已支持机器人通过MCP创建文档、智能表格

#### 3.2 配置方式

**3.2.1 配置方式1：通过对话授权并使用文档**
1、可直接要求机器人操作文档，例如：“帮我搜集本周零售行业重要新闻，汇总成一篇企业微信文档”。机器人会自动发送授权连接，请求授权。
![img](https://wework.qpic.cn/wwpic3az/875476_HFfbqyI3TKGOYf2_1773398127/0)

2、点击进入授权页面，点击“确认授权”，授权机器人文档使用权限。
![img](https://wework.qpic.cn/wwpic3az/160938_nFSa8I1ETjCuX70_1773398163/0)

3、授权成功，只需要告诉机器人“授权了”，就会自动对接MCP服务并生成文档。
![img](https://wework.qpic.cn/wwpic3az/872163_pAPwxzOPQUqTnIV_1773398185/0)

**3.2.2 配置方式2：手动授权**
**前提：已经创建过API模式机器人，并完成OpenClaw接入。**

1、入口：工作台->智能机器人->找到对应接入了OpenClaw的机器人，进入编辑页，点击可使用权限。
![img](https://wework.qpic.cn/wwpic3az/633243_qnZItz9bRPyEtjx_1773398232/0)

2、点击授权，并复制streamableHTTP URL或者JSON Config，两者任一即可（实测各模型对JSON Config的推理安装更稳定）。
![img](https://wework.qpic.cn/wwpic3az/690080_8G-ua-RLRYqsvlJ_1773398297/0)
![img](https://wework.qpic.cn/wwpic3az/587586_Ws6rmnFwTfmVm0c_1773398316/0)

3、将复制的配置，发送给机器人，即可完成安装。
![img](https://wework.qpic.cn/wwpic3az/666624_71oGYp_LSg-PgE8_1773398334/0)

**3.2.3 典型配置案例**

**案例1**：搜集本周零售行业重要新闻，汇总成一篇企业微信文档。
![img](https://wework.qpic.cn/wwpic3az/168500_I77cwzP0TJakw13_1773398409/0)
![img](https://wework.qpic.cn/wwpic3az/225317_EsY4j5zLSESdzv2_1773398423/0)

**案例2**：整理餐饮行业top10企业的基础信息和本月动向，整理成一份企业微信智能表格，包括企业名称、企业规模、主营餐饮类目、近期动向。
![img](https://wework.qpic.cn/wwpic3az/963613_8HE5n4zrSCuxigo_1773398455/0)
![img](https://wework.qpic.cn/wwpic3az/90965_9kAu6idcQuGMQvx_1773398470/0)

### 四、附2：智能表格webhook使用指引

OpenClaw可通过webhook快速接入企业微信智能表格，实现数据的自动写入与更新，智能表格作为数据载体，支持多场景数据同步与可视化展示。详细配置及接入方法可参考：[接收外部数据到智能表格](https://developer.work.weixin.qq.com/document/path/101239)

#### 4.1 核心配置方式

1、在企业微信智能表格中开启**「接收外部数据」** 功能，系统将生成该表格的**唯一Webhook地址**；
2、通过标准**HTTP POST请求**，将OpenClaw产生的数据同步至该地址，实现智能表格记录的**新增/更新**。

#### 4.2 典型适用场景

![img](https://wework.qpic.cn/wwpic3az/96944_JOMkEW47R6qPoIk_1772971542/0)

### 五、附3：企业微信API调用指引

若需通过OpenClaw进一步调用企业微信各类应用API（如文档、审批、打卡等），需先获取企业微信的**企业ID**和**应用Secret**，再完成API调用授权，步骤如下：

#### 5.1 获取企业ID与应用Secret

1、**获取企业ID**：登录企业微信管理后台，进入**我的企业**，页面下方将展示企业唯一的**企业ID（corpid）**；
![img](https://wework.qpic.cn/wwpic3az/11694_DwYW4FBsTcq_Ds4_1773398888/0)

2、**获取应用Secret：**

- 在管理后台进入**应用管理->自建应用**，新建或选择已有的自建应用（建议新建企业完整体验）；
- 进入应用详情页，点击「查看Secret」，**按提示发送后，可在企业微信消息列表中获取该应用的Secret。**

![img](https://wework.qpic.cn/wwpic3az/665138_vuRMVoxXTw2HomK_1773398918/0)

#### 5.2 API调用流程

1、将获取的**企业ID**和**应用Secret**发送给已关联OpenClaw的企业微信机器人；
2、向机器人发起[「获取access token」](https://developer.work.weixin.qq.com/document/path/91039)指令，获取API调用的核心凭证；
3、使用获取的**access token**，调用企业微信各类服务端API（如新建文档、发起审批、查询打卡数据等）；
4、API调用示例（以新建智能表格为例）：
说明：企业微信各功能模块API的具体调用规则，可参考官方[「调用API文档」](https://developer.work.weixin.qq.com/document/path/97392)。
![img](https://wework.qpic.cn/wwpic3az/626329_lTUGXd1iQGmiNfC_1773399057/0)

### 六、常见问题

#### 1、OpenClaw如何在本地终端部署安装？

目前企业微信暂无 OpenClaw 下载链接，由您自行在本地设备中安装 OpenClaw，或在腾讯云轻量应用服务器 Lighthouse（及其他云厂商服务）实例完成 OpenClaw 的部署。

#### 2、长连接接入和URL回调接入的区别是什么？

长连接方式为对接OpenClaw的推荐方式，云服务商部署的OpenClaw也可选择URL回调方式快速创建机器人对接
注意：长连接接入方式无需企业域名和IP；URL回调的方式需要企业可信域名和IP，或备案主体与企业主体相同的域名企业，[可信域名说明](https://open.work.weixin.qq.com/help2/pc/21316)

#### 3、OpenClaw接入企微智能机器人是只能企业超管才可以操作么？

目前管理员及具备创建智能机器人权限的成员，可在智能机器人中接入 OpenClaw。

#### 4、可以对企业成员限制OpenClaw接入机器人么？

管理员可配置机器人长连接方式的可用成员。操作路径：
管理后台->安全与管理->管理工具->智能机器人->管理->API模式管理->可配置「长连接」方式接入的成员
![img](https://wework.qpic.cn/wwpic3az/405058_LD43uGM-Sr6nPSz_1773399391/0)

（1）默认设置：

- 对于通讯录人数>1000人的大型企业，此设置项默认为空，即“企业内成员均不可配置长连接方式的智能机器人”，管理员可自行管理权限。
- 对于通讯录人数≤1000人的中小企业，此设置项范围与“机器人创建权限”的成员一致，即“企业内可创建智能机器人的成员，均可配置长连接方式机器人”，后续管理员可自行管理。

（2）无权限的成员在尝试使用长连接方式接入时，Bot ID 和 Secret将不展示，并且页面会提示“获取 Bot ID 和 Secret，需联系管理员开通「长连接」权限”。

#### 5、本地终端部署OpenClaw，需要先申请大模型Token吗？

需要。

#### 6、通过腾讯云lighthouse中部署OpenClaw并关联机器人，但运行后智能机器人没有回复，是什么问题？

通过腾讯云lighthouse部署的OpenClaw，使用的插件非企业微信提供的插件，建议可以寻求lighthouse团队的帮助。

#### 7、OpenClaw机器人能否在外部群/客户群中使用？

目前配置 OpenClaw 接入企业微信智能机器人，暂不支持在外部群 / 客户群中使用。

#### 8、接入OpenClaw后，企业微信给的接入权限都有那些？可以实现那些功能？

公开skills均可安装，未来将支持更多官方插件，敬请期待。

#### 9、OpenClaw接入智能机器人后，可以识别附件吗？

OpenClaw接入智能机器人，需为使用多模态模型的，才支持附件识别。



 有帮助



 无帮助

企业微信服务

API

SDK

私有化

第三方服务商

应用和小程序

官网

开发服务协议

智慧硬件

防疫测温

智慧考勤

智慧会议

智慧打印

学习与帮助

学习园地

使用手册

帮助中心

更多

助力防疫抗疫

企业案例

安全中心

联系我们







中文  |  English

关于腾讯 | 用户协议 | 使用规范 | 隐私政策 © 1998 - 2026 Tencent Inc. All Rights Reserved