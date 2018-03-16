---
layout:     post
title:      "推荐系统"
subtitle:   "《集体智慧编程》 chapter 2"
header-img: "img/post-bg-js-version.jpg"
date:       2018-03-16
author:     "Linzb"
tags:
    - 集体智慧编程
    - machine learning
---
# 推荐系统

>  本章介绍了利用协作型算法对物品进行推荐。一个协作型算法的关键是对一大群人进行搜索，从中找出与我们品味最相似的一群人，算法会对这些人的所偏爱的其它内容进行考查，并将它们组合起来构造出一个经过排名的推荐列表。

## 准备数据
本章构建一个电影推荐系统，数据为用户对电影的评分。评分数据采用嵌套字典的形式给出，如下所示：
```
critics={
'Lisa Rose': {'Lady in the Water': 2.5, 'Snakes on a Plane': 3.5,
'Just My Luck': 3.0, 'Superman Returns': 3.5, 'You, Me and Dupree': 2.5,
'The Night Listener': 3.0},

'Gene Seymour': {'Lady in the Water': 3.0, 'Snakes on a Plane': 3.5,
'Just My Luck': 1.5, 'Superman Returns': 5.0, 'The Night Listener': 3.0,
'You, Me and Dupree': 3.5},

'Michael Phillips': {'Lady in the Water': 2.5, 'Snakes on a Plane': 3.0,
'Superman Returns': 3.5, 'The Night Listener': 4.0},

'Claudia Puig': {'Snakes on a Plane': 3.5, 'Just My Luck': 3.0,
'The Night Listener': 4.5, 'Superman Returns': 4.0,
'You, Me and Dupree': 2.5},

'Mick LaSalle': {'Lady in the Water': 3.0, 'Snakes on a Plane': 4.0,
'Just My Luck': 2.0, 'Superman Returns': 3.0, 'The Night Listener': 3.0,
'You, Me and Dupree': 2.0},

'Jack Matthews': {'Lady in the Water': 3.0, 'Snakes on a Plane': 4.0,
'The Night Listener': 3.0, 'Superman Returns': 5.0, 'You, Me and Dupree': 3.5},

'Toby': {'Snakes on a Plane':4.5,'You, Me and Dupree':1.0,'Superman Returns':4.0}
}
```

## 相似性度量
 在搜集完数据之后，必须有一种方法来判断用户之间的相似度的算法，找到与指定的用户相似的用户是推荐系统的基础。实际中用到的相似度计算法方法主要有欧几里得距离和皮尔逊相关系数。
###  欧几里得距离
欧几里得度量（euclidean metric）（也称欧氏距离）是一个通常采用的距离定义，指在m维空间中两个点之间的真实距离，或者向量的自然长度（即该点到原点的距离）。在二维和三维空间中的欧氏距离就是两点之间的实际距离。

x = (x1,...,xn) 和 y = (y1,...,yn) 之间的欧几里得距离为：
![ ](/img/in-post/2018-03-16-PCI-chapter2.png)

...

def sim_distance(prefs,person1,person2):
    #相似性度量方法：欧几里德距离
    si={}
    for item in prefs[person1]:
        if item in prefs[person2]: si[item]=1

    if len(si) == 0: return 0

    sum_of_squares = sum([pow(prefs[person1][item]-prefs[person2][item],2)
                        for item in prefs[person1] if item in prefs[person2]])

    return 1/(1+sqrt(sum_of_squares))
    
...


### 皮尔逊相关系数
在统计学中，皮尔逊相关系数( Pearson correlation coefficient），又称皮尔逊积矩相关系数（Pearson product-moment correlation coefficient，简称 PPMCC或PCCs），是用于度量两个变量X和Y之间的相关（线性相关），其值介于-1与1之间。

  ![ ](/img/in-post/2018-03-16-PCI-chapter2-peason1.png)

  ![ ](/img/in-post/2018-03-16-PCI-chapter2-peason2.png)

  ![ ](/img/in-post/2018-03-16-PCI-chapter2-peason3.png)

  ![ ](/img/in-post/2018-03-16-PCI-chapter2-peason4.png)

以上列出的四个公式等价，其中E是数学期望，cov表示协方差，N表示变量取值的个数。
本章使用第四个公式进行相似性度量。

'''

def sim_pearson(prefs,p1,p2):
    #相似性度量方法：皮尔逊相关系数
    si={}
    for item in prefs[p1]:
        if item in prefs[p2]: si[item] = 1
    n=len(si)

    if n==0:
        return 0

    sum1 = sum([prefs[p1][it] for it in si])
    sum2 = sum([prefs[p2][it] for it in si])

    sum1Sq = sum([pow(prefs[p1][it],2) for it in si])
    sum2Sq = sum([pow(prefs[p2][it],2) for it in si])

    pSum = sum([prefs[p1][it]*prefs[p2][it] for it in si])

    num = pSum-(sum1*sum2/n)
    den = sqrt((sum1Sq-pow(sum1,2)/n)*(sum2Sq-pow(sum2,2)/n))
    if den==0: return 0
    r = num/den
    return r
'''

## 参考
[各种距离算法汇总](http://blog.csdn.net/mousever/article/details/45967643)
