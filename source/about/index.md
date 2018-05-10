---
title: about
date: 2016-07-09 17:23:11

---

# 个人信息

 - 赵怀鹏
 - 技术博客：[hpzhao.com](http://hpzhao.com)
 - Github：http://github.com/hpzhao
 - Email : <a href="mailto:huaipengzhao@gmail.com">huaipengzhao@gmail.com</a>
 
---

# 教育经历

+ **哈尔滨工业大学** 计算机科学与技术专业 *2012~2016 *
+ **哈尔滨工业大学** 社会计算与信息检索中心 *2016~今*

---


# 主要项目

## NLP&CC-2017 情感回复评测

+ 在经典的Attention-Based Seq2Seq做了一些尝试，加了3-hop attention用来学习更好的表示，加入了情感embedding用来控制生成回复的情感，另外用了我们实验室CR组朱庆福博士的工作Learning To Start来生成首字

## CoNLL 2017 Shared Task - Universal Dependencies

1. 负责词性标注模块，最终64种语言词性标注单项评价[第五名](http://universaldependencies.org/conll17/results-upos.html)
2. 用到的框架是rnn-rnn，第一层*char_level_rnn*用字符来表示词，第二层*word_level_rnn*用词表示句子。字符级别能够为OOV提供信息，并且还包含了一定的前后缀信息，后者在POS Tagger中是比较重要的信息。另外还加入了布朗聚类，在大多数语言的开发集上都有显著的提升

## 中文顺滑

1. 主要研究语音识别之后文本的规范化，去掉一些不合理的词，例如一些插入语。主要尝试了下面的几种框架:
2. 序列标注模型，这里尝试了CRF，BI-LSTM，BI-LSTM-CRF三种模型。实验结果来看后两种模型效果相似，但明显好于CRF，达到了83%左右的F值。
3. 级联模型，将LSTM的结果作为CRF的特征，做了Stacking操作，比单纯的CRF要好一些。
4. 基于生成的Pointer Network模型，这是基于seq2seq的改版，在生成词的时候用一个指针去和原文做local attention，选出窗口中分数最高的词。效果要好于BI-LSTM-CRF。
5. Transition-based模型，效果基本和Pointer Network持平。

## 863高考语文题干分析

+ 理解题目是解题的第一步。题干分析的工作就是从题干中抽取一些有用的信息，例如：答案数目，题目触发词等等。