---
title: NLP Lecture
date: 2023-03-06 17:03:10
katex: true
tags:
---
NLP学习
<!--more-->
# Text Preprocessing
### Defination
* word: sequence of characters
* sentence: sequence of words
* Corpus(语料): collection of documents
* word token: each word
* word type: distinct word

> 为什么做preprocessing？
> 语言是**compositional**,人类理解语言是将文本分解成句子和词，电脑也应该这样理解。

# Prepocessing steps
- Remove unwanted information
- Sentence segmentation: 将document分解为sentence
- Word tokenisation: 将句子分解为word token
- Word normalisation: 将word进行标准化
- Stopword removal: 移除不想要的word

## Sentence segmentation
- Naive method: 利用标点符号(punctuation)分割; 但是像句号(periods)在英文中常被用来缩写(abbreviations)
- Second way: 利用正则表达式(regex)结合大写字符(capital);但很多缩写会跟随名字，无法查分开
- Better way: 构建词汇表(lexicon);但很难含括全部的名字和缩写
- State-of-the-art:利用ML来进行预测
> 二分类做句子分割
> 查看每一个period，来判断是否为一句话的结尾；可以利用决策树或逻辑回归。
> 特征可以包括: 句号前后的word; word的shape;词性tag

## Word Tokenisation
> Word Tokenisation是nlp任务中重要的环节，因为我们可以通过分析文本中的单词，来
解释文本的含义。
Naive method就是按照字符串进行分割，但是存在很多的问题：
- 在英文中，存在缩写，连字符，数字，日期，多词组合等情况。
- 在中文中，没有空格可以将词分割开；同时，英文单词可能对应多个中文词；
- 在德文中，很长的词，需要复合分离器(compound splitter)
更进阶的方法可以先构建词库，然后利用**MaxMatch**算法；该算法的思想就是贪心地匹配最长的词在词表中。一直增加字符与词表中的词进行匹配，直到找不到对应的词。
![example](maxmatch.png)

### Byte-pair encoding(BPE)
BPE算法属于**subword toeknisation**。该类方法可以解决unknown word的情况。一般NLP算法会学习一些关于语料库的facts通过训练集，然后利用这些facts来在test集合做相关决策。但如果test集合中出现了train中未曾有过的词，那系统可能就不知道如何处理了。
而这样的分词方法可以自动的归纳包含小于单词的token集合。这些supword也会包含一些类似-est/-er的后缀词，这样一些从未见过的词，就可以通过不同的subword的组合来构成，从而可以解决上述的问题。该类方法主要有：BPE,[Unigram language modeling](https://doi.org/10.18653/v1/P18-1007), [wordpiece](https://doi.org/10.18653/v1/D18-2012)本节重点介绍BPE算法.

BPE算法的核心思想是：**迭代的合并高频率字符对**。有以下优点：
- Data-informed tokenisation
- Works for different language
- Deal better with unknown words
该算法通常是在单词内部进行，所以输入的语料会先被空格分割开以此来给出一组字符串，每一个对应一个单词的字符，再加上特殊的结尾符合“—”,以及计数。
![example](BytePair-init.png)
1. 首先会构建一个初始的词汇表(vocabulary)，该词汇表只有单独的字符来组成。
2. 然后检查词汇表，选择最频繁相邻的两个字符(例如A,B)，将新合并的字符AB添加到词汇表中，并且在语料中替换每个相邻的AB，得到带有新的AB的语料
3. 继续计数和融合，不断构造新的更长的字符串，直到**k**次融合后，构建k个新的词汇。一般k是算法的参数。
![example](Final_vocab.png)
在实际操作中，BPE算法可能会运行上千次的融合操作来构建出一个巨大的语料；大多高频率出现的词，会在语料中被表示为完整的词；一些比较罕见的词会被分解成subword。最坏的情况下，在test集中的unknown word会被分解成单独的字母

## Word Normalisation
为了更好的进行处理，需要将word进行标准化的操作，将word的表示统一。以此来达到**减少词数量**和**将word匹配到相同类型**的目的。当然，这样的操作可能会丢失拼写信息，在机器翻译，情感分析，信息提取中不太适用。而对于信息检索等任务比较适用。例如**Inflectional Morphology**(不同词形)会产生很多语法变种，包括名词，动词，形容词。其他语言例如法语的现象更多。
除此之外，还存在**Derivational Morphology**(衍生词汇)的情况，它会产生很多不同的词；英文中包含两类：
- suffixes(后缀): 往往会改变词汇的类别(lexical category)，例如write->writer
- prefixes(前缀): 往往会改变含义而不改变类别，例如write->rewrite

## Lemmatisation——词形还原
Lemmatisation可以用来判断两个词是否有相同的词根，通过移除任意的变形来达到还原。这就需要**lexicon of lemmas**(词典)来辅助。
## Stemming——提取主干
移除所有的后缀，只留下主干，这种方法会比lemmatisation更低的词汇稀疏性，在信息检索中非常流行。但并不总是可解释的。
### 两种方法的讨论
1. 不同点
Stemming方法是将后缀移除，这样的方法在某些场景很合适，但并不总是这样。
Lemmatisation方法是需要考虑词的形态分析，所以需要构建相关的词典。lemma是所有变形体的根本，而stem不一定是，所以构建的词典是关于lemma的。
Stemming会导致**更低的词汇稀疏性**：因为stemming的方法会导致一些单词被错误地截断为相同的基本形式，增加歧义，减少准确，降低词汇的稀疏性(因为有些词出现的频率增加了，词汇稀疏性便降低)。而lemmatisation不仅考虑形态，还会考虑上下文和词性等信息，减少歧义，更高稀疏性，更高准确，但是计算成本较高，也需要更多的时间和成本。
2. 如何选择
一般会根据任务情况选择其中的一种方法。Stemming常常用于信息检索任务，大多其他的任务都是使用lemmatisation。因为Stemming之后，会丢失大量的和token相关的信息，虽然lemmatisation也会丢失一些，但总体较少，对于大多数NLP任务，以丢失更多单词信息为代价来换取更低的词汇稀疏性是不值得的。


### The Porter Stemmer
在英文中流行的算法，应用重写的规则，首先去除词形变化的后缀，再去除衍生词的后缀。规则包含：
- c(小写c): 代表辅音(consonant) 例如: b, c, d
- v(小写v): 代表元音(vowel)例如: a, e, i, o, u
- C(大写C): 代表sequence of consonant 例如: s, ss
- V(大写V): 代表sequence of vowels 例如: o, oo

一个单词基本是以下四种形式中的一种：
- CVCV...C
- CVCV...V
- VCVC...C
- VCVC...V
可以进一步缩写表示为:\[C\](VC)^m\[V\]; m为**measure**;例子如下图：
![example](measure.png)
该算法的规则形式为：(condition)S1 -> S2；condition中会给出关于m的限制，计算给出词的m时，注意：要**取掉S1来计算**，同时要利用**最长匹配**的原则来匹配S1；示例如下图
![example](porter_stemmer.png)

词形还原的顺序：
- 首先处理plurals(复数)和inflectional(时态变形)
- 然后处理derivational(衍生词)
- 最后再做一些清洗操作

# Stopword Removal
Stop word是一系列需要从文档中移除的词；特别是bag-of-words表示时候，但不太适合sequence比较重要的情况。一般stop word有：
- all closed-class or function，例如the, a, of, for...
- 高频出现的词语
- 或通过工具来筛选，比如NLTK等
Final word是最终保留下来的词，对下游的应用有重要的影响。

[1] https://web.stanford.edu/~jurafsky/slp3