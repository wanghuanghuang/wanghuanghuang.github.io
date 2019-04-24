---
title: java 静态页面爬虫
date: 2019-03-11 16:14:23
tags:
---

##一.小说爬取
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


##二.途虎爬取摩托车品牌

返回实体类：
```
public class CarInfo {
    /**
     * 轮胎品牌
     */
    private String brand;

    /**
     * 产品规格
     */
    private String specification;

    /**
     * 产品类别
     */
    private String category;

    /**
     * 产品价格
     */
    private String price;

    /**
     * 适配车型
     */
    private String adaptableVehicle;
```


爬取代码：
```
import java.io.File;
import java.io.IOException;
import java.text.ParseException;
import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.CompletableFuture;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.regex.Matcher;
import java.util.regex.Pattern;
import java.util.stream.Collectors;
import java.util.stream.Stream;

import org.jsoup.Jsoup;
import org.jsoup.nodes.Document;
import org.jsoup.nodes.Element;
import org.jsoup.select.Elements;

import com.alibaba.fastjson.JSON;
import com.kindle.utils.file.IOUtils;

public class Test3 {

    private static final ExecutorService crawlerService = Executors.newFixedThreadPool(30);

    private static final Pattern pattern = Pattern.compile ("(?<=\\[)(\\S+)(?=\\])");

    public static String getAdaptableVehicle(String text){
        text = text.replaceAll(" ","");
        Matcher matcher = pattern.matcher(text);
        while (matcher.find ()) {
            return matcher.group();
        }
        return "";
    }

    public static void main(String[] args) throws IOException, ParseException {
        int page = getTotalPage();
        List<CompletableFuture<List<CarInfo>>> tasks = Stream.iterate(1, p -> p + 1)
                .limit(page)
                .map(p -> CompletableFuture.supplyAsync(() -> getCarInfo(p), crawlerService))
                .collect(Collectors.toList());
        List<List<CarInfo>> result = tasks.stream().map(CompletableFuture::join)
                .collect(Collectors.toList());
        //System.out.println(JSON.toJSONString(result));
        File file = new File("./json.txt");
        if(file.exists()){
            file.delete();
        }
        file.createNewFile();
        IOUtils.write(JSON.toJSONString(result), file);
    }

    public static int getTotalPage() {
        return 163;
    }

    public static List<CarInfo> getCarInfo(int page){
        String pageUrl = "https://item.tuhu.cn/Tires/" + page + "/au1-f0.html";
        List<CarInfo> list = new ArrayList<>();
        try {
            Document document = Jsoup.connect(pageUrl).get();
            Elements listTable = document.select("#Products > .List");
            Elements td2Elements = listTable.select("td.td2 > a.DisplayName");
            Elements td3Elements = listTable.select("td.td3 > div.price > strong");
            Element td2Element;
            Element td3Element;
            for(int i=0;i < td2Elements.size();i++){
                td2Element = td2Elements.get(i);
                td3Element = td3Elements.get(i);
                Document tmpDoc = Jsoup.connect(td2Element.attr("href")).get();
                Elements tmpElements = tmpDoc.select(".properties > ul >li");
                String h1Text = tmpDoc.select("h1.DisplayName").text();
                String adaptableVehicle = getAdaptableVehicle(h1Text);
                CarInfo carInfo = new CarInfo();
                carInfo.setBrand(getText("轮胎品牌：",tmpElements));
                carInfo.setSpecification(getText("产品规格：",tmpElements));
                carInfo.setCategory(getText("轮胎类别：",tmpElements));
                carInfo.setPrice(td3Element.text());
                carInfo.setAdaptableVehicle(adaptableVehicle);
                list.add(carInfo);
            }
            System.out.println("page:" + page + " finish!");
            return list;
        } catch (IOException e) {

        }
        return list;
    }

    public static String getText(String properties, Elements tmpElements){
        String text;
        for (Element tmpElement : tmpElements) {
            text = tmpElement.text();
            if(text.contains(properties)){
                return text.replaceAll(properties, "");
            }
        }
        return "";
    }

}
```

可以转出csv格式URL地址：
[json转csv](https://www.jianshu.com/p/191d1e21f7ed)