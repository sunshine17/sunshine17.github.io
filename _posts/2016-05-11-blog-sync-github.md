---
layout: post
category: "web"
title:  "Github鐿象(VPS)自動更新方案"
tags: [blog,github pages,git webhook]
summary: "這個博客是部署在VPS上的，而寫文章時，我們通常只會把本地的commit push到github上，本文討論VPS上的repository與github的即時同步策略，以及分享下我在解決此問題時的思路"
---
這個博客是部署在VPS上的，但我并没有在VPS上部署git server，VPS上只是一个git repository(即Github的鐿象)。Github在本博的架构里是作为中心仓库的角色存在的，在寫文章時，只需把本地的commit push到Github上。 

但这样数据同步的问题就来了：
當Github收到新push的時候，Github Pages會自動build出靜態HTML，但VPS上的repository是不知道有更新的，這需要有個機制讓VPS上的repository感知Github的更新信息，然後自動pull Github倉庫的最新改動。至於jekyll的自動build，可以通過jekyll的`--watch`選項監控源代碼變量自動build。

## **博客自動更新:**
以daemon模式啟動jekyll的build進程，開啟watch選項，這樣就可以在更新代碼時自動重新build出整站的HTML代碼到_site目錄中：

```shell
nohup jekyll build --watch > /dev/null 2>&1 & 
```

## **方案1：[Github Webhooks](https://help.github.com/articles/about-webhooks/)**
Webhooks是Github提供的一個事件觸發服務，當指定的事件發生時，Webhooks會把該事件通知給事先在Github.io上配置的外部web服務器。**Webhooks事件举例**：

- 新的push
- 开启了一个pull请求
- 一个GitHub Pages站点的建立
- Git Team新增了一个新团队成员

通过GitHub API，你可以配合webhooks搭建一系列项目构建服务，例如持续集成([Continuous Integration](https://en.wikipedia.org/wiki/Continuous_integration))、同步備份鏡象(我的VPS其實就是GitHub的一個鏡象)、甚至是部署項目到你的production server。

前提, 要使用Webhooks，**你必須**：

1. 要有一個能被GitHub訪問的web server;
2. 按GitHub API規範實現對應的接口;
3. 到GitHub上對應項目的settion頁面配置Webhooks, [完整的Webhooks開發指南在這裡](http://developer.github.com/webhooks)。

Webhooks的鐿象同步方案架構：
![用Webhook實現的VPS鐿象同步方案](/images/blog_sync_webhook.png "用Webhook實現的VPS鐿象同步方案")

從架構圖可以看出此方案的重點：

1. Github提供的**webhook服務**（圖中作為一個組件的形式）: 監控Git倉庫的變動，有變動時就把對應的事件post到VPS上的**HTTP API**服務(event notification)，從而可以在HTTP API中調用相應的shell命令實現VPS倉庫的GIT PULL動作來同步Github上的最新改動到VPS
2. HTTP API: 需要自己實現的web service, 用來接收Github推送過來的事件，需要參考[Webhooks開發指南](http://developer.github.com/webhooks)來實現
3. 在Github上配置Webhook

Webhooks作為Github提供的項目構建接口方案，定義了事件、通用API協議、一組對應的安全配置等規範，對於大型項目來說用起來是很有價值的。但對於我的需求來說，只需要當Github有更新時，VPS自動pull一下代碼而已。對接一個定義得完整的API是有一定工作量的，我一看webhook的文檔就沒有對接的興趣了(沒有快速接入說明)，於是我開始思考有沒有更簡單的辦法。

## **方案2：[Github Webhooks](https://help.github.com/articles/about-webhooks/)**
其實只是一個事件通知，不使用Webhooks也能自己實現。這樣就能避免接入webhooks的開發與調試工作量了。自己實現通知的架構圖如下：
![簡單VPS鐿象同步方案](/images/blog_sync.png "簡單VPS鐿象同步方案")

**與【方案1】的區別：**

- 在寫作環境push代碼後直接調用VPS的HTTP API同步，因為寫作環境是可變的，為避免手工寫調用API的命令，我用一個shell腳本封裝了push與調用API兩個動作(FileName: push.sh)：

```shell
git push
curl http://4377.me/sync_blog.php
```
- HTTP API無需參考Github API規範，邏輯也很簡單，就是在VPS的repository上`git pull`一下最新改動到VPS。直接用PHP即可簡單實現，代碼如下(FileName: sync_blog.php)：

```php
<?php
$out = array();
exec('/var/wsp/blog/sunshine17.github.io/sync.sh', $out);   // 邏輯部分由shell腳本實現
echo json_encode($out);
```
php調用的shell腳本(FileName: sync.sh)：

```shell
#!/bin/zsh
cd "$(dirname "$0")"
git pull 
```

這樣，在寫作環境寫完文章後，只要執行push.sh就可以自動同步代碼到Github倉庫，且調用VPS上的HTTP API執行一次同步Github操作，從而自動更新了博客。

## **方案3**
使用**[IFTTT](https://ifttt.com/)**。

這是國外做得很不錯的一個條件觸發服務，整合了很多知名的互聯網服務，包括天氣、股票、facebook、twitter、google等。甚至還能發中國運營商的短信（幾年前我試過發當天氣預報有雨時發信息到我的中國移動手機卡，但用了一段時間就用不了了）。

可惜的是，查了一下IFTTT上的Github Channel，貌似沒有git push的觸發條件，只好作罷。
其實IFTTT的方案本質也是一樣，只是IFTTT充當了觸發器的角色（Webhooks）。


## **總結**
一開始，我想到最簡單的辦法是【方案1】，可以順便試用一下Git提供的Webhooks。但後來我選擇了【方案2】, 原因如下：

1. 目前的需求只是一個博客鐿象同步，鐿象也只有一個VPS，方案2在能實現的基礎上會更簡單
2. 暫時未想到要如何運用Webhooks在實際項目中，根據我的按需而學原則，未用得著又沒興趣深入的，不學
3. 博客的重點不是技術，而是內容，只要能實現功能就足夠了，更多的時間應該花在其他更有價值的技術研究上


