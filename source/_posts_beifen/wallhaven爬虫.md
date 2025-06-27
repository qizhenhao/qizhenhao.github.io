---
title: wallhaven爬虫
date: 2022-12-05 00:19:14
tags: 爬虫

---

# py爬取壁纸（setu）

###### wallhaven网站爬取

<!-- more -->

<iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=330 height=86 src="//music.163.com/outchain/player?type=2&id=2012263046&auto=1&height=66"></iframe>

刚开始爬的时候一切顺利，但最后遇到了反爬虫机制，就是必须要登录，否则无法显示一些色图。

首先想到的就是py模拟登录，这里说两种解法：

​	1.在post请求中把账号，密码输入，一般都不会成功，应为会有js加密，这里可以解密，具体我也不了解就不往下说了

​	2.可行且简单的方法，用cookie绕过去，先说一下cookie的概念，在客户端对服务器发送请求，服务器会产生客户端的记录，来得知客户端之前做过什么，就比如你登录了bilbil，关了之后，在进入就免登录了，这之间就是cookie的功劳，让服务器记得你，回归正题，模拟登录就是首先你要登录这个网站，再记录cookie，爬取的headers填上cookie，让后就进入登录后的界面了，就可以下载图片了

```python
import requests
import parsel
import os

for shuzi in range(1,75):
    print(f"正在下载第{shuzi}页")
    url = 'https://wallhaven.cc/hot?page='
    #头请求就不展示了
    headers = {
        'user - agent':
#！！！这个网站的反爬虫机制就是登录，用cookie绕过登录---->    ,'cookie' : 

    }

    req = requests.get(url+str(shuzi),headers=headers,cookies=cook)

    print(url+str(shuzi))
    sele = parsel.Selector(req.text)
    lis = sele.css('.preview')

    for li in lis:

        pic_url = li.css('.preview ::attr(href)').get()

        print(pic_url)
        pos = requests.get(pic_url,headers=headers,verify=False)
        pic_sele = parsel.Selector(pos.text)
        pic_lis = pic_sele.xpath('//div[@class="scrollbox"]//img/@src').getall()
        print(pic_lis)
        for pic_urll in pic_lis:
            pic = requests.get(pic_urll,headers=headers,verify=False).content
            pic_name = pic_urll.split('/')[-1]
            if not os.path.exists(f'..//{pic_name}'):
                with open(f'..//{pic_name}','wb') as f:
                    f.write(pic)
                print('  完成图片'+pic_name)
            else:
                print(' 已存在'+pic_name)
```

# 图片壁纸需要自取

下面是wallhaven网站热门壁纸，果然色图是人的第一生产力，因为不能上传太多，就随便传了几张，可以尝试自己爬取哦

![wallhaven-1p3kjg](wallhaven%E7%88%AC%E8%99%AB/wallhaven-1p3kjg.jpg)

![wallhaven-2yemrm](wallhaven%E7%88%AC%E8%99%AB/wallhaven-2yemrm.jpg)

![wallhaven-6dk7q7](wallhaven%E7%88%AC%E8%99%AB/wallhaven-6dk7q7.jpg)

![wallhaven-6dk8dw](wallhaven%E7%88%AC%E8%99%AB/wallhaven-6dk8dw.jpg)

![wallhaven-6dk8jl](wallhaven%E7%88%AC%E8%99%AB/wallhaven-6dk8jl.jpg)

![wallhaven-6dk8m7](wallhaven%E7%88%AC%E8%99%AB/wallhaven-6dk8m7.jpg)

![wallhaven-6dk8y6](wallhaven%E7%88%AC%E8%99%AB/wallhaven-6dk8y6.png)

![wallhaven-6dk38l](wallhaven%E7%88%AC%E8%99%AB/wallhaven-6dk38l.png)

![wallhaven-6dk39l](wallhaven%E7%88%AC%E8%99%AB/wallhaven-6dk39l.jpg)

![wallhaven-6dk89l](wallhaven%E7%88%AC%E8%99%AB/wallhaven-6dk89l.jpg)

![wallhaven-6dk326](wallhaven%E7%88%AC%E8%99%AB/wallhaven-6dk326.jpg)

![wallhaven-6dk356](wallhaven%E7%88%AC%E8%99%AB/wallhaven-6dk356.jpg)

![wallhaven-6dkd5l](wallhaven%E7%88%AC%E8%99%AB/wallhaven-6dkd5l.jpg)

![wallhaven-6dke3w](wallhaven%E7%88%AC%E8%99%AB/wallhaven-6dke3w.png)

![wallhaven-6dkxlw](wallhaven%E7%88%AC%E8%99%AB/wallhaven-6dkxlw.jpg)

![wallhaven-6dkxp6](wallhaven%E7%88%AC%E8%99%AB/wallhaven-6dkxp6.jpg)

![wallhaven-7p6zqy](wallhaven%E7%88%AC%E8%99%AB/wallhaven-7p6zqy.jpg)

![wallhaven-9d611k](wallhaven%E7%88%AC%E8%99%AB/wallhaven-9d611k.jpg)

![wallhaven-9d612x](wallhaven%E7%88%AC%E8%99%AB/wallhaven-9d612x.png)

![wallhaven-gp8yz7](wallhaven%E7%88%AC%E8%99%AB/wallhaven-gp8yz7.jpg)

![wallhaven-jx5d9y](wallhaven%E7%88%AC%E8%99%AB/wallhaven-jx5d9y.jpg)

![wallhaven-jx5z3w](wallhaven%E7%88%AC%E8%99%AB/wallhaven-jx5z3w.jpg)

![wallhaven-kxwzm7](wallhaven%E7%88%AC%E8%99%AB/wallhaven-kxwzm7.jpg)

![wallhaven-l8ml8y](wallhaven%E7%88%AC%E8%99%AB/wallhaven-l8ml8y.jpg)

![wallhaven-m3dq31](wallhaven%E7%88%AC%E8%99%AB/wallhaven-m3dq31.jpg)

![wallhaven-o5x8z5](wallhaven%E7%88%AC%E8%99%AB/wallhaven-o5x8z5.png)

![wallhaven-o5xpg7](wallhaven%E7%88%AC%E8%99%AB/wallhaven-o5xpg7.jpg)

![wallhaven-p98e3j](wallhaven%E7%88%AC%E8%99%AB/wallhaven-p98e3j.jpg)

![wallhaven-p9816j](wallhaven%E7%88%AC%E8%99%AB/wallhaven-p9816j.jpg)
