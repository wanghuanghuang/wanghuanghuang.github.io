---
title: java 静态页面爬虫
date: 2019-03-11 16:14:23
tags:
---

找一个页面 <a href="https://m.yuyouge.com/wapbook/66769_160173736.html">静态页面url</a>
如图：
![万道龙皇小说](source/images/静态页面爬虫/静态页面1.png)

我们的任务就是把这页的小说爬取下来。
F12打开 
看到静态页面代码：
![静态页面代码](source/images/静态页面爬虫/静态页面2.png)
我们可以看到 内容均在div id=cContent 中
代码如下：
```$xslt
import org.jsoup.Jsoup;
import org.jsoup.nodes.Document;
import org.jsoup.nodes.Element;

 @Test
   public void testPQ() {
        Document document = null;
        try {
            document = Jsoup.connect("https://m.yuyouge.com/wapbook/66769_160173736.html").get();
        } catch (IOException e) {
            e.printStackTrace();
        }
        List<Element> list = document.select("#cContent");
        for (Element element : list) {
            System.out.println(element);
        }
    }

```

运行单元测试 效果如下：
![静态代码爬取结果](source/images/静态页面爬虫/静态页面3.png)