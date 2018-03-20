---
layout:     post
title:      "对博客进行聚类：分层聚类、K-Mesns 聚类"
subtitle:   "《集体智慧编程》 chapter 3"
header-img: "img/post-bg-js-version.jpg"
date:       2018-03-19
author:     "Linzb"
tags:
    - 集体智慧编程
    - machine learning
---
# 聚类
聚类分析（Cluster analysis，亦称为群集分析）是对于统计数据分析的一门技术，在许多领域受到广泛应用。聚类就是按照某个特定标准(如距离)把一个数据集分割成不同的类或簇，使得同一个簇内的数据对象的相似性尽可能大，同时不在同一个簇中的数据对象的差异性也尽可能地大。也即聚类后同一类的数据尽可能聚集到一起，不同类数据尽量分离。

主要分为层次化聚类算法，划分式聚类算法，基于密度的聚类算法，基于网格的聚类算法，基于模型的聚类算法等。本章主要介绍层次化聚类中的分层聚类算法和划分聚类中的 K-Means 聚类算法，并使用该算法对一组博客订阅源进行聚类。

## 零、准备数据
为了对这些博客进行聚类，需要的数据是一组指定的词汇在每个博客出现的次数。根据单词出现的频度对博客进行聚类，或许可以帮助我们分析出是否存在一些博客经常撰写相似的主题或写作风格类似。

### 1、解析订阅源
RSS订阅源是一个包含博客及其所有文章条目信息的简单的XML文档。为了给每个博客中的单词计数，首先第一步就是要解析这些订阅源。有一个非常不错的程序能够完成这项工作，它就是[Universal Feed Parser](https://pythonhosted.org/feedparser/index.html)。使用Universal Feed Parser，可以很轻松地从任何RSS或Atom订阅源中得到标题、链接和文章的条目。
```
import feedparser
import re

def getwordcount(url):
    d = feedparser.parse(url)
    wc = {}

    for e in d.entries: #每个entries是一篇文章（html格式）
        if 'summary' in e:  summary = e.summary
        else:   summary = e.description

        words = getwords(e.title + ' ' + summary)   #调用getword函数
        for word in words:
            wc.setdefault(word, 0)    
            wc[word] += 1
    return d.feed.title, wc            #feed.title是博客的名称， wc是{单词：单词数}的dict

def getwords(html):
    #解析html文本，返回html的单词list
    txt = re.compile(r'<[^>]+>').sub(' ', html)   #去除字符串中html标签

    words = re.compile(r'[^A-Z^a-z]+').split(txt)  # 利用所有非字母字符拆分出单词  

    return [word.lower() for word in words if word != ' ' ]    # 转化成小写形式  

```

### 2、过滤过于常见及过于鲜见的单词
像“the”这样的单词几乎到处都是，而像“film-flam”这样的单词则有可能只出现在个别博客中，所以通过只选择介于某个百分比范围内的单词，我们可以减少需要考查的单词总量。在本例中，我们可以将10%定为下届，将50%定为上界，不过假如你发现有过多常见或鲜见的单词出现，不妨尝试一下不同的边界值。

        单词百分比 = 出现该单词的博客数量 / 博客总数

最后，我们利用上述单词列表和博客列表来建立一个文本文件，其中包含一个大的矩阵，记录着针对每个博客的所有单词的统计情况。
```
if __name__ == '__main__':

    apcount ={} #记录出现某一单词的博客数
    wordcounts = {}
    feedlist = [line for line in open('F:\\CI\\3\\feedlist.txt')] #feedlist.txt包含一组博客订阅源
    for feedurl in feedlist:
        try:
            title, wc = getwordcount(feedurl)
            wordcounts[title] = wc
            for word, count in wc.items():
                apcount.setdefault(word, 0)
                if count > 1:
                    apcount[word] += 1
        except:
            print('Failed to parse feed %s' %feedurl)
    wordlist = []
    for w, bc in apcount.items():
        frac = float(bc) / len(feedlist)
        if frac > 0.1 and frac < 0.5: wordlist.append(w)      #过滤过于常见及过于鲜见的单词

    #建立一个文本文件包含一个大的矩阵，记录着针对每个博客的所有单词的统计情况。
    out = open('blogdata.data','w')  
    out.write('Blog')
    for word in wordlist:
            out.write('\t%s' %word)
    out.write('\n')

    for blog, wc in wordcounts.items():
        out.write(blog)
        for word in wordlist:
            if word in wc:
                out.write('\t%d' % wc[word])
            else:
                out.write('\t0')
        out.write('\n')

```


## 一、聚类


###  分级聚类



### K-Means 聚类


## 二、可视化聚类-树形图




## 三、参考
[知乎：用于数据挖掘的聚类算法有哪些，各有何优势？](https://www.zhihu.com/question/34554321)
