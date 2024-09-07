---
title: NLP lecture n-gram
date: 2023-03-20 17:21:21
katex: true
tags:
---
N-gram Language Models
<!--more-->
# N-gram Probabilities
我们的目标是求m个单词的序列的概率，首先第一步是利用**Chain Rule**将联合概率转化为条件概率。
![prob_format](format1.png)
但是这样的条件概率还是比较棘手，所以我们需要采取**Markov Assumption**马尔可夫假设。对于不同阶的设定，可以分为
以下几种情况。
![n-gram](n-gram.png)
> The Markov Assumption: 为状态空间中经过从一个状态到另一个状态的转换的随机过程。该过程要求具备“无记忆”的性质：
> 下一状态的概率分布只能由当前状态决定，在时间序列中它前面的事件均与之无关。

那么如何在计算这些概率呢？在这里使用**MLE**的方法。基本就是计数的方法。对于不同的gram模型，有以下的计算方法
![MLE_counts](MLE.png)
Btw,我们需要一些特殊的tag来标记序列的开始和结尾"\<s\>"和"<\/s>".注意我们需要在公式中加入对"<\/s>"的预测，因为这是句子的结尾。以下是trigram的例子。
![Trigram](trigram_exp.png)
## Problems
- 语言模型存在着长距离影响，可能会需要非常大的**n**.
- 概率有时候会很小，这时候可以采用**log**的方法来避免数值下溢。
- 如果在公式中存在corpus从来没有出现过的grams，那整个公式的结果就会变为0，这样就需要**smoothing**的方法。

# Smoothing
基本的想法是，给从来没有见过的预设概率，也要保证所有的概率的和为1
## Laplacian(Add-one)
基本思想：假装我们比实际看到的每个 n-gram 都多一次。具体例子如下：
![add_one](addone.png)
其中，在unigram中**M**表示在corpus中全部token的数量；而**V**代表的词汇表是有多少个不同的词(理解为set)。
在**V**中，我们需要加入"<\/s>"，因为需要对末尾进行预测，而"\<s\>"则不需要，因为我们不需要去推断它的条件概率。下图是一个bigram的例子
![add_one_exp](addone_exp.png)
## Lidstone(Add-k)
基本思想：有时候直接add one的方法会过于多，所以取而代之的是add 一个小数**k**。公式如下图。关于k的选择不同的应用需求不同，有时候可能需要finetune来进行调整
![add-k](add-k.png)
## Absolute Discounting
基本思想：从观察到的 n-gram 计数中借用固定概率。然后重新分配给没有看见过的n-gram。
我们从能观察(observed)到gram中拿出来一些counts分配给从未出现过(unobserved)的gram。如下图所示，其中**0.1**就表示从observed中提取多少，0.1*5因为一共有五个observed的，除以2是因为有两个unobserved，这样就得到了新的有效计数(effective counts),然后概率的计算就根据bigram的基础公式，对应组合的有效计数除以全部的counts。
![absolute](absolute_exp.png)
## Backoff(Katz)
基本思想：重新分配的时候，不是像absolute那样平均的分配，而是通过**lower order**的方式，例如原来是bigram，此时就依据unigram。
![backoff](backoff.png)
- 其中第一行就是计算observed的情况，考虑减去discount
> - 其中第二行就是计算unobserved的情况，
> - 其中第一项表示，从observed中获取到的count的概率；
> - 第二项就是利用lower order来控制分配的权重。第二项中,分子计算的是unigram的概率，而分母计算的是所有与context没有共同出现过的word的unigram概率的和。
但是该方法存在问题，如果在计算unigram的概率时，某些错误词汇的出现次数反而更多，这样计算的结果会更大，而这却是错误的。
## Kneser-Ney Smooth
基本思想：为了解决上述问题，该方法会基于低阶 n-gram 的通用性。
> 什么是通用性(versatility AKA continuation probability)?
> 高: 对于一个word，与很多不同的词co-occurs
> 低: 对于一个word，与很少的词co-occurs
具体公式如下，不同的点在于第二行的第二项，第二项也叫做continuation probability
![Kneser-Ney](Kneser-Ney.png)
- 分子是counts(注意不是概率了)，含义是词汇表中出现在单词w之前的单词类型的数量，也叫做continuation count；
计算continuation count的时候，考虑整个corpus包括"\<s\>"和"<\/s>"。
- 分母是和wi-1从未共同出现过的wj的continuation count的总和
这样一来，就可以解决虽然错误的词出现很多，但是它有更低的versatility。

具体例子如下，给出chunk a，求a的continuation probability
![continuation_count](continuation_count.png)
![continuation_prob](continuation_prob.png)
## Interpolation(插补)
基本思想：一种更好的方式来结合不同阶的n-gram
![interpolation](interpolation.png)

