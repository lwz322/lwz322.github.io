---
layout: article
title: 爬取Tesla充电桩数据
mathjax: true
mermaid: true
chart: true
toc: true
mode: immersive
tags : Python 数学建模
header:
  theme: dark
article_header:
  type: overlay
  theme: dark
  background_color: '#ffffff'
  background_image:
    src: https://img.vim-cn.com/b5/8ffebe4387c5e66c15e62360a81733307a4d15.png
    gradient: 'linear-gradient(0deg, rgba(0, 0, 0 , .7), rgba(0, 0, 0, .7))'
---
源起于2018年[MCM\ICM](https://www.comap.com/undergraduate/contests/mcm/contests/2018/problems/)的D题，没有数据是很麻烦的，所以数据就只能自己爬取了，首先就是Tesla的充电桩的地理信息，最近尝试了下，发现没有之前想的那么难

<!--more-->

## 环境

python3环境即可，不过这里可以提的一点是网络环境，大概是我所在的地方访问Tesla的速度比较慢，爬取数据需要几个小时，后面我就把代码放到了Google云上面运行，几分钟就可以跑出结果，而且也不需要特别高的配置，因为这里用到基本的requests和正则匹配
**声明**：因为个人学识有限，对多线程编程以及减轻对服务器的压力没有做过多的处理，可能会带来一些问题，所以只提供以一种解决问题的思路，顺带分享下作为初学者的代码

## 分析

这里需要的主要是

- 充电桩的位置——经纬度信息
- 充电桩的数目——充电站内的充电桩的数目

要得到这些，联系到之前接触过的东西就是

- 地图的API，通过详细的地名给出经纬度信息

  比如说[Google MAP API](https://developers.google.com/maps/documentation/)中的[Geocoding](https://developers.google.com/maps/documentation/geocoding/)可以由具体的位置返回一个含有经纬度的Json

- Tesla官网上的信息

  [Supercharger](https://www.tesla.com/supercharger)中地图右下有[View list of location](https://www.tesla.com/findus/list/superchargers/United%20States)

  ![1544246432708](https://img.vim-cn.com/96/ba7c025b8e1e5c2b112e7dd3d9bea677d380d0.png)

  跳转后的网站就给出了美国的充电站的详细位置，那么又如何去获取经纬度呢？

  进入某个充电站的详细信息之后就有

  ![1544246603839](https://img.vim-cn.com/29/3c1355a2f0d6e10c2315c2fa63fc11c20acbb0.png)

  通过浏览器的F12的检查功能就可以看到地图给出了经纬度的信息，另外左侧的文字也给出了充电站内的充电桩的数目，经过这样的分析之后发现，其实只需要做下正则匹配就行了。

## 代码

  这里因为程序是单线程的，索性就把两种充电桩的代码分开写和分开运行了（假装是多线程....），不同的部分主要是网页url和匹配的规则

### Supercharger

  ```python
  #!/usr/bin/env python
  # coding=utf-8
  import xlwt
  import requests
  import re
  #从Tesla的美国官网获得美国境内的的充电桩位置信息，以及个数，再由地图导出经纬度信息
  ##获取网页的源代码
  def get_one_page(url):
   try:
      headers = {'User-Agent':'Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) ''Chrome/51.0.2704.63 Safari/537.36'}
      response = requests.get(url,headers = headers, timeout = 30)
      if response.status_code == 200:
          return response.text
      else:
          print(response.status_code)
          return None
   except
      print('ConnectionError')
      return None
  #创建表格,添加工作表
  book = xlwt.Workbook(encoding='utf-8',style_compression=0)
  sheet = book.add_sheet('sheet1',cell_overwrite_ok=True)
  #对网页源代码进行匹配
  url = 'https://www.tesla.com/findus/list/superchargers/United+States'
  all_html = get_one_page(url)
  ##编译正则匹配对象(就是括号内的部分)
  ##re.S正则表达式修饰符:使 . 匹配包括换行在内的所有字符
  pattern = re.compile('<address.*?<a.*?href="(.*?)".*?>(.*?)</a>.*?</address>',re.S)
  ##匹配所有的位置条目
  items = re.findall(pattern,all_html)
  #输出形如(/findus/location/charger/dc2789，Benson&#039;s Appliance Center)的tuple组成的list
  #对匹配的位置条目查询器经纬度信息
  i = 0
  k = 0
  for item in items:
      sheet.write(i,0,item[1])
      t_url = "https://www.tesla.com"+item[0]
      t_html = get_one_page(t_url)
      t_pattern = re.compile('&center=(.*?)&zoom',re.S)
      n_pattern = re.compile('<p><strong>[Cc]harging</strong>.*?>(.*?) [Ss]uperchargers.*?</p>',re.S)
      try:
          t_items = re.findall(t_pattern,t_html) ##输出经纬度的list
          sheet.write(i,1,t_items[0])
      except:
          print('Error',k,':',t_url)
          k+=1
      try:
          n_items = re.findall(n_pattern,t_html) ##输出充电桩的个数
          sheet.write(i,2,n_items[0])
      except:
          sheet.write(i,2,'0')
      print(i,':',item[1],t_items[0])
      i+=1
  #直接把结果保存在当前目录下的xls文件里面
  book.save('tesla_super_chargers.xls')
  print('Finished,totally got %d Chargers,and %d Error'% (i,k))
  ```
### Destination Charger

  ```python
  #!/usr/bin/env python
  # coding=utf-8
  import xlwt
  import requests
  import re
  #从Tesla的美国官网获得美国境内的的充电桩位置信息，再由地图导出经纬度信息
  ##获取网页的源代码
  def get_one_page(url):
   try:
      headers = {'User-Agent':'Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) ''Chrome/51.0.2704.63 Safari/537.36'}
      response = requests.get(url,headers = headers, timeout = 30)
      if response.status_code == 200:
          return response.text
      else:
          print(response.status_code)
          return None
   except
      print('Error on HTTP')
      return None
  #创建表格,添加工作表
  book = xlwt.Workbook(encoding='utf-8',style_compression=0)
  sheet = book.add_sheet('sheet1',cell_overwrite_ok=True)
  #对网页源代码进行匹配
  url = 'https://www.tesla.com/findus/list/chargers/United+States'
  all_html = get_one_page(url)
  ##编译正则匹配对象(就是括号内的部分)
  ##re.S正则表达式修饰符:使 . 匹配包括换行在内的所有字符
  pattern = re.compile('<address.*?<a.*?href="(.*?)".*?>(.*?)</a>.*?</address>',re.S)
  ##匹配所有的位置条目
  items = re.findall(pattern,all_html)
  #输出形如(/findus/location/charger/dc2789，Benson&#039;s Appliance Center)的tuple组成的list
  #对匹配的位置条目查询器经纬度信息
  i = 0
  k = 0
  for item in items:
      sheet.write(i,0,item[1])
      t_url = "https://www.tesla.com"+item[0]
      t_html = get_one_page(t_url)
      t_pattern = re.compile('&center=(.*?)&zoom',re.S)
      n_pattern = re.compile('<p><strong>[Cc]harging</strong>.*?>(.*?)Tesla.*?</p>',re.S)
      try:
          t_items = re.findall(t_pattern,t_html) ##输出经纬度的list
          sheet.write(i,1,t_items[0])
      except:
          print('Error',k,':',t_url)
          k+=1
      try:
          n_items = re.findall(n_pattern,t_html) ##输出充电桩的个数
          sheet.write(i,2,n_items[0])
      except:
          sheet.write(i,2,'0')
      print(i,':',item[1],t_items[0])
      i+=1
  #直接把结果保存在当前目录下的xls文件里面
  book.save('tesla_des_chargers.xls')
  print('Finished,totally got %d Chargers,and %d Error'% (i,k))
  ```

## 结果

程序大概要跑几分钟，爬下来的数据大概是4000+条，最后在[Google MAP](https://www.google.com/maps/d/edit?hl=en&hl=en&mid=1Txaoldp6_ZeG7_rwjwO1IW8zB66107oW&ll=36.200869445648266%2C-98.56588939157513&z=5)画出来就是下面这样， 这里也把即将建成的充电站画了进来，不过充电桩数目为0![1544247339926](https://img.vim-cn.com/b5/8ffebe4387c5e66c15e62360a81733307a4d15.png)

### ECharts.js可视化

这里用了下[ScatterGL](http://echarts.baidu.com/examples/editor.html?c=scatterGL-gps&amp;gl=1，点的数量不多，但是可以看出密度越高的部分也就越亮

![US](https://img.vim-cn.com/05/b5ce6dd81606c034ffb88d60bee2bce0cf6a70.jpg)

### 推一下舍友的工作

[Equations.online](http://equations.online/2018/12/09/chargebar/)上面是从北汽爬取的数据，数据量要大很多