![transformer整体结构](./transformer_image/transformer整体结构.png)

# Transformer模型详解

> 建议大家看一下李宏毅老师讲解的Transformer，非常简单易懂（个人觉得史上最强transformer讲解）：[https://www.youtube.com/watch?v=ugWDIIOHtPA&list=PLJV_el3uVTsOK_ZK5L0Iv_EQoL1JefRL4&index=60](https://link.zhihu.com/?target=https%3A//www.youtube.com/watch%3Fv%3DugWDIIOHtPA%26list%3DPLJV_el3uVTsOK_ZK5L0Iv_EQoL1JefRL4%26index%3D60)

## 前言

Transformer由论文[《Attention is All You Need》](https://zhida.zhihu.com/search?content_id=163422979&content_type=Article&match_order=1&q=《Attention+is+All+You+Need》&zd_token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJ6aGlkYV9zZXJ2ZXIiLCJleHAiOjE3NzA1MTM3NDQsInEiOiLjgIpBdHRlbnRpb24gaXMgQWxsIFlvdSBOZWVk44CLIiwiemhpZGFfc291cmNlIjoiZW50aXR5IiwiY29udGVudF9pZCI6MTYzNDIyOTc5LCJjb250ZW50X3R5cGUiOiJBcnRpY2xlIiwibWF0Y2hfb3JkZXIiOjEsInpkX3Rva2VuIjpudWxsfQ.y4H3uzQr3nJiIzNoMEKBcoLXyc5cnvaW78SGMIuDD5I&zhida_source=entity)提出，现在是谷歌云TPU推荐的参考模型。论文相关的[Tensorflow](https://zhida.zhihu.com/search?content_id=163422979&content_type=Article&match_order=1&q=Tensorflow&zd_token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJ6aGlkYV9zZXJ2ZXIiLCJleHAiOjE3NzA1MTM3NDQsInEiOiJUZW5zb3JmbG93IiwiemhpZGFfc291cmNlIjoiZW50aXR5IiwiY29udGVudF9pZCI6MTYzNDIyOTc5LCJjb250ZW50X3R5cGUiOiJBcnRpY2xlIiwibWF0Y2hfb3JkZXIiOjEsInpkX3Rva2VuIjpudWxsfQ.VPJS_iRpGwAX6k-IZ_weBm7JKC5MTJeNXmlzBAj_Nj4&zhida_source=entity)的代码可以从GitHub获取，其作为Tensor2Tensor包的一部分。哈佛的NLP团队也实现了一个基于[PyTorch](https://zhida.zhihu.com/search?content_id=163422979&content_type=Article&match_order=1&q=PyTorch&zd_token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJ6aGlkYV9zZXJ2ZXIiLCJleHAiOjE3NzA1MTM3NDQsInEiOiJQeVRvcmNoIiwiemhpZGFfc291cmNlIjoiZW50aXR5IiwiY29udGVudF9pZCI6MTYzNDIyOTc5LCJjb250ZW50X3R5cGUiOiJBcnRpY2xlIiwibWF0Y2hfb3JkZXIiOjEsInpkX3Rva2VuIjpudWxsfQ.2EEKznFT_4lz8sDa7rtlONggWhP1tjnOsK54HdxBaeA&zhida_source=entity)的版本，并注释该论文。

在本文中，我们将试图把模型简化一点，并逐一介绍里面的核心概念，希望让普通读者也能轻易理解。

Attention is All You Need：[Attention Is All You Need](https://link.zhihu.com/?target=https%3A//arxiv.org/abs/1706.03762)

## 1.Transformer 整体结构

首先介绍 Transformer 的整体结构，下图是 Transformer 用于中英文翻译的整体结构：![transformer整体结构2](./transformer/transformer整体结构2.png)

​				Transformer 的整体结构，左图Encoder和右图Decoder

可以看到 **Transformer 由 Encoder 和 Decoder 两个部分组成**，Encoder 和 Decoder 都包含 6 个 block。Transformer 的工作流程大体如下：

**第一步：**获取输入句子的每一个单词的表示向量 **X**，**X**由单词的 Embedding（Embedding就是从原始数据提取出来的Feature） 和单词位置的 Embedding 相加得到。

