---
layout: post
title:  "参考文献格式"
date:   2022-1-18 20:35:35 +0800
categories: command
---

* 引用的文献要求是近3-5年内的，过早发表的论文不能引用。如不能引用1997年、1964年发表的论文。

* 引用的文献能是期刊论文就不要引用会议论文，除非是所讨论的内容只有这个会议提出，别的期刊论文没有涉及。

* 使用Latex引用论文的格式混乱，不统一。标准格式如下：

期刊格式
```
\bibitem{Salinas_Flunkert_2003}%格式：第一作者姓_第二作者姓_发表年份

D. Salinas, V. Flunkert and J. Gasthaus, ``Financial time series forecasting using support vector machines,” \emph{Neurocomputing}, vol. 55, no. 1, pp. 307–319, Sep. 2003.
```

注意点
1. 列出所有作者，不要省略，如不能用`et al.`
2. 最后一个作者与倒数第二个作者之间用and，不要用“, and”。如应为V. Flunkert and J. Gasthaus，而不是V. Flunkert and J. Gasthaus
3. 每个作者列出名字首字母 姓全称，如J. Wang
4. 论文题目左侧的引号比较特殊，英文模式下，使用键盘上的数字1左侧的按键输入，如``
5. 论文实词首字母大写，如`Multiqueue Scheduling`
6. 论文题目右侧的引号，英文模式下，使用回车键左侧的引号输入，如”
7. 期刊名称给出全称，如不能写`IEEE Trans. on Ind. Inf.`
8. 期刊名称应为斜体，使用\emph{}实现
9. 给出volume和issue号，如vol. 15, no. 10
10. 起始页码和终止页码之间是2个英文的横线，不是1个。如，pp. 5404—5412
11. 给出月份和发表年份，其中月份用缩写。如Oct. 2019


会议格式
```
%格式：第一作者姓_第二作者姓_发表年份
\bibitem{Bi_Yuan_2017}


M.-T. Luong, H. Pham and C. D. Manning. ``Effective Approaches to Attention-based Neural Machine Translation", in \emph{Proc. of the 2015 Conference on Empirical Methods in Natural Language Processing}, Lisbon, Portugal, 2015, pp. 1412--1421
```

注意点
1. 列出所有作者，不要省略，如不能用`et al.`
2. 最后一个作者与倒数第二个作者之间用and，不要用“, and”。如应为V. Flunkert and J. Gasthaus，而不是V. Flunkert and J. Gasthaus
3. 论文题目左侧的引号比较特殊，英文模式下，使用键盘上的数字1左侧的按键输入，如``
4. 论文实词首字母大写，如`Multiqueue Scheduling`
5. 论文题目右侧的引号，英文模式下，使用回车键左侧的引号输入，如”
6. 会议名称格式为，in `\emph{Proc. of XXXXXX}`
7. XXXXXX 部分应给出全称，如`2015 IEEE 8th International Conference on Cloud Computing`，不能为`2015 IEEE 8th Inter. Conf. on Cloud Comp.`
8. 会议名称应为斜体，使用`\emph{}`实现
9. 会议地列出城市、州缩写、国家，如`New York, NY, USA`
10. 列出会议发表的年份，如`2015`
11. 起始页码和终止页码之间是2个英文的横线，不是1个。如`pp. 5404—5412`
