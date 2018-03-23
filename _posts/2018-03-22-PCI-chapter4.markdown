---
layout:     post
title:      "搜索与排名:一个简单的搜索引擎的实现"
subtitle:   "《集体智慧编程》 chapter 4"
header-img: "img/post-bg-js-version.jpg"
date:       2018-03-22
author:     "Linzb"
tags:
    - 集体智慧编程
    - machine learning
---
# 搜索引擎
网页抓取：从一个或一组特定的网页开始，根据网页内部链接逐步追踪到其他网页。这样递归进行爬取，直到到达一定深度或达到一定数量为止。
建立索引：建立数据表，包含文档中所有单词的位置信息，文档本身不一定要保存到数据库中，索引信息只需简单的保存到一个指向文档所在位置的引用即可。
查询和排名：根据合适的网页排序方法，返回一个经过排序的页面列表。

## 零、网页抓取：一个简单的爬虫
该爬虫程序首先输入一个初始的网页，获取该网页的内容并进行解析，建立“网页—词—位置”的关系，存入 wordlocation 数据表中。随后从网页中寻找链接，若有链接，则将链接的相对路径转换成绝对路径，并存储，作为下一层检索的URL。同时将“原网页URL—链接URL—链接文本”关系保存到link表中。至此，一层爬取完毕。根据新加入的URL，爬取下一层网页。


程序框架：
```
class crawler:
  # 初始化crawler类并传入数据库名称
  def __init__(self, dbname):
    pass

  def __del__(self):
    pass

  def dbcommit(self):
    pass

  # 辅助函数，用于从table表获取filed元组值为value的条目的id，并且如果条目不存在，就将其加入数据库中
  def getentryid(self, table, field, value, createnew=True):
    return None

  # 为url网页建立索引
  def addtoindex(self, url, soup):
    print('Indexing %s' %url)

  # 从HTML网页中提取文字（不带html标签）
  def gettextonly(self, soup):
    return None

  # 依据任何非字母字符进行分词处理
  def separatewords(self, text):
    return None

  # 如果url已经建过索引，则返回true
  def isindexed(self, url):
    return False

   # 添加一个关联两个网页的链接  #将“原URL—链接URL—链接文本”关系保存到link表
  def addlinkref(self, urlFrom, urlTo, linkText):
    pass

  # 从一小组网页开始进行广度优先搜索，直至某一给定深度，
  # 期间为网页建立索引
  def crawl(self, pages, depth=2):
    pass
    
  # 创建数据库表
  def createindextables(self):
    pass
```


在此之前，先简单介绍两个相关函数库。
### 1、urllib库
[urllib](https://docs.python.org/3.6/library/urllib.html)是Python自带的标准库，无需安装，直接可以用。包括以下模块：

    urllib.request 请求模块：     打开和读取URLs
    urllib.error 异常处理模块：    处理由urllib.request引发的异常
    urllib.parse url解析模块
    urllib.robotparser robots.txt解析模块
本章中，将利用 urllib.request 模块下载等待建立索引的网页。

### 2、Beautiful Soup
[Beautiful Soup](https://www.crummy.com/software/BeautifulSoup/bs4/doc.zh/)是一个可以从HTML或XML文件中提取数据的Python库。它能够通过你喜欢的转换器实现惯用的文档导航,查找,修改文档的方式。

将一段文档传入BeautifulSoup 的构造方法,就能得到一个文档的对象, 可以传入一段字符串或一个文件句柄.

    from bs4 import BeautifulSoup

    soup = BeautifulSoup(open("index.html"))
    soup = BeautifulSoup("<html>data</html>")

几个简单的浏览结构化数据的方法:

    soup.title
    # <title>The Dormouse's story</title>

    soup.title.name
    # u'title'

    soup.title.string
    # u'The Dormouse's story'

    soup.title.parent.name
    # u'head'

    soup.p
    # <p class="title"><b>The Dormouse's story</b></p>

    soup.p['class']
    # u'title'

    soup.a
    # <a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>

    soup.find_all('a')
    # [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
    #  <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>,
    #  <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]

    soup.find(id="link3")
    # <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>

### 3、爬虫

```
# 从一小组网页pages开始进行广度优先搜索，直至某一给定深度，
# 期间为网页建立索引
def crawl(self, pages, depth=2):
    for i in range(depth):
        newpages = set()
        for page in pages:
            try:
                c = urllib.request.urlopen(page)  #首先输入一个初始的网页，
            except:
                print("Could not open %s" % page)
                continue
            soup = BeautifulSoup(c.read()) #获取该网页的内容并进行解析
            self.addtoindex(page, soup)    #建立“网页—词—位置”的关系，存入 wordlocation 数据表中。

            #
            links = soup('a')   
            for link in links:          #从网页中寻找链接
                if ('herf' in dict(link.attrs)):
                    url = urljoin(page, link['herf']) #若有链接，则将链接的相对路径转换成绝对路径
                    if url.find("'") != -1: continue

                    url = url.split('#')[0]
                    if url[0:4] == 'http' and not self.isindexed(url):
                        newpages.add(url)     #存储，作为下一层检索的URL。
                    linkText = sefl.gettextonly(link)
                    self.addlinkref(page,url,linkText) #将“原URL—链接URL—链接文本”关系保存到link表

            self.dbcommit()
        pages = newpages
```





















##  一、建立索引



### 可视化分级聚类-树形图


## 二、查询和排名






## 三、参考
