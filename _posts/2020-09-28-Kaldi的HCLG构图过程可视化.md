---
title: Kaldi的HCLG构图过程可视化
date: 2020-09-28 17:34:49
description: 本文希望通过简单的中文三句话，将整个HCLG的过程：构建语言模型（G.fst），发音词典模型（L.fst），以及合并的模型（LG.fst和CLG.FST），生成HMM模型（H.fst），最终合并、确定、最小化得到HCLG.fst，展现并将其中的WFST模型进行可视化。
categories:
 - 语音识别
photos:
 - /gallery/hclg/HCLG.png
tags:
 - 语音识别
 - kaldi
 - WFST
 - HCLG
---


基于DNN-HMM的语音识别系统，需要在深度网络的推理之后外接解码来实现最后的语句识别，相关的语音识别理论基础不在此赘述，需要简单描述的是，解码实际上是得到音素状态转移的声学判决后，在一个HMM模型（H），Context Dependent Phone（C），Lexicon（L）和语言模型（G）合并的WFST（Weighted Finite-State Transducers，加权有限状态机）上进行路径寻优。WFST的基本概念和操作请查看：从WFST到语音识别。


