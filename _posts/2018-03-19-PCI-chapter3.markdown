---
layout:     post
title:      "对博客进行聚类：分层聚类、K-Means 聚类"
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
RSS订阅源是一个包含博客及其所有文章条目信息的简单的XML文档。为了给每个博客中的单词计数，首先第一步就是要解析这些订阅源。[Universal Feed Parser](https://pythonhosted.org/feedparser/index.html)是一个非常不错的Python库能够完成这项工作。使用Universal Feed Parser，可以很轻松地从任何RSS或Atom订阅源中得到标题、链接和文章的条目。
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


##  一、分级聚类
分级聚类算法（Hierarchical Cluster Analysis，HCA）通过连续不断的将最为相似的群组两两合并，来构造出一个群组的层级结构。其中的每个群组都是从一个简单元素开始的。在每次迭代的过程中，分级聚类算法会计算每两个群组间的距离，并将最近的两个群组合并成一个新的群组。这个过程一直重复下去，知道只剩下一个群组为止。

我们可以发现，这个算法有点类似于哈夫曼树的构造：

    1，查找节点集合中，距离最近的两个节点。

    2，将找到的两个节点合并为一个节点，添加到节点集合中，并将先前的两个节点从节点集合中移除。

    3，重复第一步和第二步，直到节点集合中只剩下一个节点。

根据上述算法，我们可以得到一颗聚类二叉树。
当使用上述算法的时候，距离是作为分类的基础。本章使用[皮尔逊相似性度量方法](http://linzb.xyz/2018/03/16/PCI-chapter2/)
```
class bicluster(object):    #定义二叉树节点
    def __init__(self, vec, left=None, right=None, distance=0, id=None):
                  #vec为单词数向量
        self.left = left
        self.right = right
        self.vec = vec
        self.id = id
        self.distance = distance

def hcluster(rows, distance=peason):    # rows为数据矩阵  peason 参见上一章
    distances = {}
    currentclustid = -1

    # 初始化聚类为 数据集中的所有行
    clust = [bicluster(rows[i], id=i) for i in range(len(rows))]

    while len(clust) > 1:
        lowestpair = (0, 1)
        closest = distance(clust[0].vec, clust[1].vec)

        # 遍历每一个配对，寻找最小距离  
        for i in range(len(clust)):
            for j in range(i+1, len(clust)):

                # 用distances来缓存距离的计算值  
                if (clust[i].id, clust[j].id) not in distances:
                    distances[(clust[i].id, clust[j].id)] = distance(clust[i].vec, clust[j].vec)

                d = distances[(clust[i].id, clust[j].id)]
                if d < closest:
                    closest = d
                    lowestpair = (i, j)

        # 计算两个聚类的平均值  
        mergevec = [(clust[lowestpair[0]].vec[i] + clust[lowestpair[1]].vec[i]) / 2.0
                            for i in range(len(clust[0].vec))]

        # 建立新的聚类
        newcluster = bicluster(mergevec, left=clust[lowestpair[0]],
                                right=clust[lowestpair[1]],
                                distance=closest, id=currentclustid)

        # 不在原始集合中的聚类，其id为负数  
        currentclustid -= 1
        del clust[lowestpair[1]]
        del clust[lowestpair[0]]
        clust.append(newcluster)

    return clust[0]
```


### 可视化分级聚类-树形图


## 二、K-Means 聚类
k-平均算法（k-means clustering）作为一种聚类分析方法流行于数据挖掘领域。K-Means聚类的目的是：把n个点划分到k个聚类中，使得每个点都属于离他最近的均值（此即聚类中心）对应的聚类，以之作为聚类的标准。

经典K-means算法流程：

    1、随机地初始化选择k个簇的中心（通常使用的初始化方法有Forgy和随机划分(Random Partition)方法）；
    2、对剩余的每个对象，根据其与各簇中心的距离，将它赋给最近的簇；
    3、重新计算每个簇的平均值，更新为新的簇中心；
    4、不断重复2、3，直到准则函数收敛。
```
# K-均值聚类算法
def kcluster(rows, distance=pearson, k=4):
    '''
    :param rows: 表示每一个博客词语的计数（即readfile函数中所返回的每一行的数据）
    :param distance: 表示元素之间距离计算公式（这里使用皮尔逊相关度计算方法）
    :param k: K-均值聚类的聚类个数K
    :return: 最终聚类结果（k个聚类）
    '''

    # 确定每个点的最小值和最大值
    # ranges = [(min([row[i] for row in rows]), max([row[i] for row in rows])) for i in range(len(rows[0]))]

    # 随机创建k个中心点
    # clusters = [[random.random() * (ranges[i][1] - ranges[i][0]) + ranges[i][0] for i in range(len(rows[0]))] for j in range(k)]


    # 聚类迭代过程
    lastmatches = None
    for t in range(100): # 迭代次数为100次
        print('Iteration %d' % t)
        bestmatches = [[] for i in range(k)]

        # 在每一行（每一个数据项）中寻找距离最近的中心点
        for j in range(len(rows)):
            row = rows[j]
            # 对于每一行row（每一个数据项），求出与该row距离最近的中心点bestmatch
            bestmatch = 0
            for i in range(k):
                d = distance(clusters[i], row)
                if d < distance(clusters[bestmatch], row):
                    bestmatch = i
            # bestmatches[bestmatch].append(row)
            bestmatches[bestmatch].append(j)


        # 如果迭代结果与上一次相同，则整个过程结束
        if bestmatches == lastmatches:
            break
        lastmatches = bestmatches

        # 把中心点移到其所有成员的平均位置处
        for i in range(k):
            avgs = [0.0] * len(rows[0])
            if len(bestmatches[i]) > 0:
                for rowid in bestmatches[i]:
                    for m in range(len(rows[rowid])):
                        avgs[m] += rows[rowid][m]
                for j in range(len(avgs)):
                    avgs[j] = avgs[j] / len(bestmatches[i])
                clusters[i] = avgs

    return bestmatches
```






## 三、参考
[知乎：用于数据挖掘的聚类算法有哪些，各有何优势？](https://www.zhihu.com/question/34554321)
