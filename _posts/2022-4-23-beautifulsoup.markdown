---
layout: post
title:  "beautifulsoup"
date:   2022-4-23 20:35:35 +0800
categories: command
---


安装
```
pip install beautifulsoup4
from bs4 import BeautifulSoup
soup = BeautifulSoup(html_doc, 'html.parser')

print(soup.prettify())
# <html>
#  <head>
#   <title>
#    The Dormouse's story
#   </title>
#  </head>
#  <body>
#   <p class="title">
#    <b>
#     The Dormouse's story
#    </b>
#   </p>
#   <p class="story">
#    Once upon a time there were three little sisters; and their names were
#    <a class="sister" href="http://example.com/elsie" id="link1">
#     Elsie
#    </a>
#    ,
#    <a class="sister" href="http://example.com/lacie" id="link2">
#     Lacie
#    </a>
#    and
#    <a class="sister" href="http://example.com/tillie" id="link3">
#     Tillie
#    </a>
#    ; and they lived at the bottom of a well.
#   </p>
#   <p class="story">
#    ...
#   </p>
#  </body>
# </html>
```

数据结构查询
```
soup.find_all('a')
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
#  <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>,
#  <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]

soup.find(id="link3")
# <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>
```

keyword参数
```
# [<a class="sister" href="http://example.com/elsie" id="link1">three</a>]
soup.find_all(href=re.compile("elsie"), id='link1')

# 按属性查找
# [<div data-foo="value">foo!</div>]
data_soup.find_all(attrs={"data-foo": "value"})
```

按class查找
```
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
#  <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>,
#  <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]
soup.find_all("a", class_="sister")

# [<p class="title"><b>The Dormouse's story</b></p>]
soup.find_all(class_=re.compile("itl"))
```

内容获取
```
# 完整数据获取
print(soup.prettify())

# 获取特定的URL地址
link_node = soup.find('a',href="http://example.com/elsie")
print(link_node.name,link_node['href'],link_node['class'],link_node.get_text())
```