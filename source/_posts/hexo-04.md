---
title: hexo系列-04 目前為止遇到的問題
date: 2020-06-03 12:00:00
tags: [hexo, bug, categories, highlight]
categories: hexo
thumbnail: /uploads/hexo-cover.png
toc: true
---

這邊羅列一些在使用Hexo建構自己的Blogger時遇到的一些問題。

<!--more-->

## Categories跟Sub-Categories

第一個先記錄一個比較智障的問題：一直以來在其他hexo使用者的網頁中，他們的分類中下方有一個較小的分類，就是在網站中的categories下能有像是樹狀圖的層層結構，我一直以為那是需要額外安裝的，加上我文章量本來就比較少，所以我一直不以為意XD，就像下面那樣：

![Origin Categories Page](Origin_Categories.png)

經過一番搜尋發現只需要在.md上面的categories改成：

```markdown
categories: [母類別, 子類別]
```

就可以產生如下的成果：

![After Changing Categories Page](After_Categories.png)

## 調整圖片的長寬

這個問題跟上面的問題差不多，算是一個使用hexo的小技巧，基本上就是直接html的語法來調整長跟寬：

```markdown
<img src="/image/test.jpg" width="50%" height="50%">
```

<img src="Sample.jpg" width="50%" height="50%">



## Highlight中的Clipboard和複製到行號

如題所述，這也是困擾了我一陣子的問題，每次在寫Blogger，複製程式碼的時候都會一起複製到行號，另外點選右上角clipboard只會複製到第一行的內容：

![Highlight_Problem](Highlight_Problem.png)

經過[搜尋](https://github.com/ppoffice/hexo-theme-icarus/issues/563)發現是hexo-util這個套件的問題，在某一次更新後highlight的渲染出了問題，我透過更新npm還有hexo來解決：

```bash
npm update
npm install hexo
cd /your/hexo/folder
# 更新完後需要在渲染一次你的頁面
hexo g
```

理論上這樣操作就可以解決這個問題了，不過我經過重開機後才解決這個問題，猜測也許是npm的一些設定在安裝好新版本的hexo後沒有更新，如果更新完後發現沒有效果，可以先試試看重新開機。

## Google Drive圖片

圖片在Github上使用，個人覺得當圖片檔案太大的時候，loading的時間會認人感覺有點久，有時候還會卡卡的，所以我找到將圖片放在Google Drive可以直接使用圖片的方法。

原本將圖片放在Google Drive上面，分享出來後會看到下面這個網址：

```markdown
https://drive.google.com/file/d/****************************/view?usp=sharing
```

https://drive.google.com/file/d//view?usp=sharing

將星號的部分套用在下面的地方就可以直接使用了：

```markdown
https://drive.google.com/uc?export=view&id=****************************
```



![Example](https://drive.google.com/uc?export=view&id=1yz5TPFKnkic5rLKfLVlPFG_0n0HvPgnj)