---
title: hexo系列-03 讓google可以搜尋到你的網站
date: 2019-12-14 01:55:26
tags: [hexo, Google Search Console, Google Analytics, sitemap]
categories: hexo
thumbnail: /uploads/hexo-cover.png
toc: true
---

## 前言

原本以為要讓自己的網站在網路上可以被搜尋到，只要能用網址打開網站，之後Google搜尋引擎就可以搜尋到相對應的內容，沒有想到事情不是那麼簡單的~~我還是太年輕了~~。

<!--more-->

要讓搜尋引擎能搜到自己的網站，首先要去[Google網站管理員](ttps://www.google.com/webmasters/)的[Google Search Console](https://search.google.com/search-console/)提交網站的一些設定，詳細的會在下面一一列出。

## 安裝sitemap套件

```bash
npm install hexo-generator-sitemap --save
```

安裝這個套件會直接幫你生成需要的檔案，接著在`theme/_config.yml`加上下面這段：

```yaml
#Sitemap
sitemap:
    path: sitemap.xml
```

接下來用：

```bash
hexo g
```

生成`sitemap.xml`的檔案，位於`XXX.github.io/public/`中。

## 創建`robots.txt`

在`XXX.github.io/source`中創建`robots.txt`文件：

```yaml
# hexo robots.txt
User-agent: *
Allow: /
Allow: /archives/
Allow: /categories/
Allow: /tags/
Allow: /about/ 

# Disallow: /js/
# Disallow: /css/
# Disallow: /fancybox/

Sitemap: https://augustushsu.github.io/sitemap.xml
```

`robots.txt`是用來告訴網路搜尋引擎的漫遊器(又稱網路蜘蛛)，此網站中的哪些內容是不應被搜尋引擎的漫遊器取得的，哪些是可以被漫遊器取得的。

其中`Allow`後面加的就是你的menu，也就是允許漫遊器搜尋的到網頁，而`Disallow`則相反。

你可以將在測試的網頁資料夾寫在`Disallow`上，這樣一些漫遊器就不會去搜到你不想公開的網頁囉，詳細的說明可以參考[Googel文件](https://support.google.com/webmasters/answer/6062596?hl=zh-Hant)，還有[這篇文章](https://www.awoo.com.tw/blog/robotstxt-crawl/)。

## Google Search Console

前面有說要讓Google搜尋的到你需要在[Google Search Console](https://search.google.com/search-console/)中填寫關於你訊息，登入Google帳戶後，你應該會看到這樣的畫面：

![Google Search Console](GSC-01.jpg)

輸入你`Github`上的網址，之後會要你去驗證：

![Google Search Console Verification](GSC-02.jpg)

這邊選擇的是用`HTML標記`，將google提供的html程式碼複製到`XXX.github.io/themes/icarus/layout/common/head.ejs`，直接加在最上面就可以：

![Google Search Console Finish](GSC-03.jpg)

接著上傳到GitHub：

```bash
hexo -g d
```

按下驗證，如果設置的正確，就會跳出以下畫面：

![Google Search Console HTML](GSC-finish.png)

接下來要將剛剛建的`sitemap.xml`提交到`Google Search Console`：

![Google Search Console Sitemap](GSC-sitemap.jpg)

驗證之後，Google會花一些時間將你的網站建檔，等過了一天你就可以用`Google Search Console`來分析自己的網頁囉～

![Google Search Console Overview](GSC-overview.png)

## robots.txt

點選`網址審查`把你的網頁貼上，會顯示`正在從 Google 索引擷取資料`完成後，點選`查看以檢索的網頁`再點`更多資訊`就可以看到前面`Disallow`不想讓其他人看到的部分了：

![Google Search Console Robots](GSC-robots.jpg)

## Google Analytics

打開[Google Analytics](https://analytics.google.com)網站，註冊一個帳號，然後點選追蹤程式碼，複製你的`追蹤ID`，形式大概是`UA-XXXXXXXX-X`。

將上面的ID輸入到`Icarus`下`_config.yml`檔案中的：

```yaml
google-analytics:
  # Google Analytics tracking id
  tracking_id: UA-XXXXXXXX-X
```

最後再將你的成品上傳到Github就完成囉～

```bash
hexo -g d
```

