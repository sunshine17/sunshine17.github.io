---
layout: post
category: "web"
title:  "本博客的架构"
tags: [blog,jekyll,github pages]
summary: "这个博客是纯静态网站，原本想使用github pages的服务，后来发现git的所有服务都撞墙了，只好将其放在VPS上。本文记录一下我在vps上使用nginx + jekyll + github搭建个人静态博客的方法。"
---
这个博客是纯静态网站，原本想使用github pages的服务，后来发现git的所有服务都撞墙了，只好将其放在VPS上。本文记录一下我在vps上使用nginx + jekyll + github搭建个人静态博客的方法。

##最終效果
- 訪問[http://sunshine17.github.io/](http://sunshine17.github.io/)時，提供服務的是github pages
- 訪問[http://blog.4377.me/](http://blog.4377.me/)時，提供服務的是VPS;

##Github Pages
使用[Jekyll](https://jekyllrb.com/)來生成靜態網頁內容。Github提供兩種靜態網站服務：

1.Project Pages Site
項目官網。使用gh-pages分支，當你push/merge內容到這個分支時，github會自動build出對應的靜態網頁內容。域名：username.github.io/projectname

2.User Pages site
個人網站。使用master分支，同樣的，當你push/merge內容到這個分支時，github會自動build出對應的靜態網頁內容。域名：username.github.io。本博客就是一個User Pages site，所以這裡重點針對此類型說明。

##靜態網站(配合Github)的優點
非常多，Google一下就知道。我只簡單列幾點：

- 只需要web服務器，低功耗
- vim的高效文字編輯 + markdown語法描述排版，可謂兼顧效率與美觀 
- git做version controll，打tag、出release，簡直就是強大的出版器
- 大道至簡：everything is done on my keyboard, no more mouse

##架構圖
一圖勝千言，先上圖。<br/>
<img src="/images/blog_architecture.png" />


### 角色說明
- **VPS**: Virtual Private Server，不解釋了，幾乎每個愛玩的服務端開發人員都會有一個
- **Nginx**: web服務器，提供靜態文件訪問服務，只需把Jekyll build的結果目錄_site配置為nginx的document root即可
- **Jekyll**: static site generator，開啟--watch選項，設為後臺進程來監聽文件變化，當有文章更新時會即時build一次
- **SiteFolder**: Jekyll build的輸出目錄，默認設置為你repository的_site文件夾，例如sunshine17.github.io/_site/, 需把此文件夾加入git的ignore列表
- **Git Repository**: 在VPS上的博客源代碼倉庫，**下一篇我會討論自動同步Github更新的策略**
- **github.io**: github上的git服務器，充當版本控制的中心服務器，VPS上的repository會pull這個git server，你在任何電腦上寫完文章也是push到這個git server







