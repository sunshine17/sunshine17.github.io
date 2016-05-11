---
layout: post
category: "web"
title:  "本博客的架构--VPS自動同步github更新"
tags: [blog,github pages,git webhook]
summary: "這個博客是部署在VPS上的，而寫文章時，我們通常只會把本地的commit push到github上，本文討論VPS上的repository與github的即時同步策略，以及分享下我在解決此問題時的思路"
---
這個博客是部署在VPS上的，但我并没有在VPS上部署git server，VPS上只是一个git repository。Github在本博的架构里是作为中心仓库的角色存在的，这样在寫文章時，只需把本地的commit push到Github上。但这样数据同步的问题就来了：當Github收到新push的時候，Github Pages會自動build出靜態HTML，但VPS上的repository是不知道有更新的，這需要有個機制讓VPS上的repository感知Github的更新信息，然後自動pull Github倉庫的最新改動。至於jekyll的自動build，可以通過jekyll的--watch選項監控源代碼變量自動build。

**先放出最終方案:**
![VPS自動同步GITHUB方案](/images/blog_sync.png "VPS自動同步GITHUB方案")


## 方案一：[Github Webhooks](https://help.github.com/articles/about-webhooks/)
Webhooks是Github提供的一個事件觸發服務，當指定的事件發生時，Webhooks會把該事件通知給事先在Github.io上配置的外部web服務器。










