---
layout:     post
title:      "基于用户的协同过滤推荐系统"
subtitle:   "《集体智慧编程》 chapter 2"
header-img: "img/post-bg-js-version.jpg"
date:       2018-03-16
author:     "Linzb"
tags:
    - 集体智慧编程
    - machine learning
---
# 推荐系统

基于用户的协同过滤推荐(User-based Collaborative Filtering Recommendation)先使用统计技术寻找与目标用户有相同喜好的邻居，然后根据目标用户的邻居的喜好产生向目标用户的推荐。基本原理就是利用用户访问行为的相似性来互相推荐用户可能感兴趣的资源。


## 零、准备数据
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

## 一、相似性度量
 在搜集完数据之后，得有一种方法来判断用户之间的相似度，找到与指定的用户相似的用户是推荐系统的基础。有许多方法可以衡量两组数据间的相似程度，使用哪一种方法最优完全取决于具体应用。 本章用到的相似度计算法方法主要有欧几里得距离和皮尔逊相关系数。
###  欧几里得距离
欧几里得度量（euclidean metric）（也称欧氏距离）是一个通常采用的距离定义，指在m维空间中两个点之间的真实距离，或者向量的自然长度（即该点到原点的距离）。在二维和三维空间中的欧氏距离就是两点之间的实际距离。

x = (x1,...,xn) 和 y = (y1,...,yn) 之间的欧几里得距离为：
![ ](/img/in-post/2018-03-16-PCI-chapter2.png)
```
    from math import sqrt  
    #相似性度量方法：欧几里德距离
    def sim_distance(prefs,person1,person2):
        # prefs:用户评价数据(上文中的critics)
        # 计算用户person1与person2的相似度
        #返回值介于0到1之间，1表示偏好相同

        #统计双方都曾评价过的物品列表
        si={}
        for item in prefs[person1]:
            if item in prefs[person2]: si[item]=1

        #如果两者没有共同之处，则返回0
        if len(si) == 0: return 0

        #计算所有差值的平方和
        sum_of_squares = sum([pow(prefs[person1][item]-prefs[person2][item],2)
                            for item in prefs[person1] if item in prefs[person2]])
        ##返回（距离+1）的倒数；将距离加1，避免被零整除的错误
        return 1/(1+sqrt(sum_of_squares))
```



### 皮尔逊相关系数
在统计学中，皮尔逊相关系数( Pearson correlation coefficient），又称皮尔逊积矩相关系数（Pearson product-moment correlation coefficient，简称 PPMCC或PCCs），是用于度量两个变量X和Y之间的相关（线性相关），其值介于-1与1之间。

  ![ ](/img/in-post/2018-03-16-PCI-chapter2-peason1.png)

  ![ ](/img/in-post/2018-03-16-PCI-chapter2-peason2.png)

  ![ ](/img/in-post/2018-03-16-PCI-chapter2-peason3.png)

  ![ ](/img/in-post/2018-03-16-PCI-chapter2-peason4.png)

以上列出的四个公式等价，其中E是数学期望，cov表示协方差，N表示变量取值的个数。
本章使用第四个公式进行相似性度量。
```
    #相似性度量方法：皮尔逊相关系数
    def sim_pearson(prefs,p1,p2):
        #返回值介于-1到1之间，1表示两人对每件物品评价一致

        #统计双方都曾评价过的物品列表
        si={}
        for item in prefs[p1]:
            if item in prefs[p2]: si[item] = 1
        n=len(si)

        #如果两者没有共同的物品，则返回0
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
```
## 二、推荐

以上相似性度量可以计算每一对用户的相似度，那么可以针对某一目标用户，找出与其兴趣最为相似的用户。

我们可以得到与某一目标用户兴趣相似的用户，然而我们的目的的是给目标用户推荐他可能喜欢的物品。如果从其相似用户评价高的电影中找随便挑其一个没看过的电影做一个推荐的话，这太随意了。如何预测用户对一个未评价过的物品的评价？为了杜绝这样的随意，采用一种加权平均的计算方式来预测目标用户对未观看过的电影的评价。以相似度作为权值，与用户越相似的其它用户对物品的评价，占权重越大。预测用户对物品评价，根据评价分数结果排序，给用户推荐评价分数高的物品。所以用户对物品的评价分数就可以通过如下公式预测：
  ![ ](/img/in-post/2018-03-16-PCI-chapter2-critics.png)

```
  def topMatches(prefs, person, n=5, similarity=sim_pearson):
      '''返回prefs字典中与person最相似的n个 '''
      scores=[(similarity(prefs, person, other), other)
                  for other in prefs if other!=person]
      scores.sort()
      scores.reverse()
      return scores[0:n]

  def getRecommendations(prefs, person, similarity=sim_pearson):
      '''利用prefs中其它人的评价与相关系数进行加权平均，为person提供建议'''
      totals={}
      simSums={}

      for other in prefs:
          if other==person: continue  #不与自己做比较
          sim=similarity(prefs,person,other)

          if sim<=0: continue     #忽略相似度小于等于零的情况
          for item in prefs[other]:
              if item not in prefs[person] or prefs[person][item]==0:
                  totals.setdefault(item,0)
                  totals[item]+=prefs[other][item]*sim
                  simSums.setdefault(item,0)
                  simSums[item]+=sim

      rankings=[(total/simSums[item],item) for item,total in totals.items()]

      rankings.sort()
      rankings.reverse()
      return rankings
```



## 三、参考
[各种距离算法汇总](http://blog.csdn.net/mousever/article/details/45967643)
