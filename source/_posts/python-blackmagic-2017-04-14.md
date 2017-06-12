---
title: Python中的黑魔法
date: 2017-04-14 14:07:22
tags: [Python]
categories: 编程
---
8. 多线程

9. 多进程

11. 设计模式

12. 算法

14. 对于numpy和scipy是作为科学计算用的，提供各种向量矩阵计算、优化、随机数生成等等

15. matplotlib使用 python的数据可视化最常用的matplotlib，用于科学制图——基础的绘图，已经集成在pandas里此外，ggplot2在R语言下的绘图神器，也同时支持python的哟，非常推荐。

17. scrapy框架使用

19. statsmodels：统计包，设计各种统计模型，包括回归、广义回归、假设检验等，结果类似于R语言，会给出各种检验结果。

20. nlp

python数据处理的包：
1. 自带正则包， 文本处理足够了
2. cElementTree, lxml  默认的xml速度在数据量过大的情况下不足
3. beautifulsoup  处理html
4. hadoop(可以用python) 并行处理，支持python写的map reduce，足够了， 顺便说一下阿里巴巴的odps，和hadoop一样的东西，支持python写的udf，嵌入到sql语句中
5. numpy, scipy, scikit-learn 数值计算，数据挖掘
6. dpark类似hadoop一样的东西

1，2，3，5是处理文本数据的利器（python不就处理文本数据方便嘛），4，6是并行计算的框架（大数据处理的效率在于良好的分布计算逻辑，而不是什么语言）暂时就这些，最好说一个方向，否则不知道处理什么样的数据也不好推荐包，所以没有头绪从哪里开始介绍这些包

