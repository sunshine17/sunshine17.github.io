---
layout: post
category: "pi"
title:  "解決pi的unmet dependencies報錯"
tags: [linux,pi]
summary: "當apt的依賴被某個新裝的軟件包破壞時，無論你運行什麼簡單的系統工具，都會出現類似找不到lib或存在unmet dependencies之類的報錯"
---
## **起因**

想給pi裝個zabbix來監控系統狀態，在zabbix官網下載了一個xz的壓縮包。解壓後才知道xz是一種解壓後會自動運行腳本的壓縮包，它自動寫入了一些文件到系統路徑，同時自動啟動了zabbix-agent後臺進程。
這種壓縮包其實是非常危險的，就像病毒一樣，你一觸發它，它有權限在你毫不知情的情況下做任何事。
果然，隨後我想重啟php-fpm時就報錯找不到相關庫文件了：

>
error while loading shared libraries: libcurl.so.4: cannot open shared object file: No such file or directory

## **解決過程**

### 常規做法：google出錯信息

當php啟動提示加載不到庫時，直覺告訴我這只是其中一個異象，實質的問題仍未暴露。
然後我嘗試用apt-get安裝另一個軟件包時，真正的報錯出現了，提示如下：

>Reading package lists... Done
>Building dependency tree       
>Reading state information... Done
>Some packages could not be installed. This may mean that you have
>requested an impossible situation or if you are using the unstable
>distribution that some required packages have not yet been created
>or been moved out of Incoming.
>The following information may help to resolve the situation:
>
>The following packages have unmet dependencies:
> curl : Depends: libcurl3-gnutls (= 7.47.0-1) but it is not going to be installed
> E: Unable to correct problems, you have held broken packages.

關鍵在於**unmet dependencies**這個提示，明顯是apt的依賴關係被破壞了。於是我google了一下這個關鍵詞。很快看到了好幾個stackoverflow上的類似問題，我選擇了搜索結果的第一個來嘗試。這裡給出**[鏈接](https://askubuntu.com/questions/140246/how-do-i-resolve-unmet-dependencies-after-adding-a-ppa)**。

### **以下是我在pi3上的修復過程**

- 清除一些本地倉庫裏的軟件包及數據庫: `sudo apt-get clean`
- fix被破壞的依賴關係 `sudo apt-get -f install`
若提示有些包不再需要用到:

>Reading package lists... Done
>Building dependency tree       
>Reading state information... Done
>Calculating upgrade... The following package was automatically installed and is no longer required:
>  pypy-upstream-doc
>Use 'apt-get autoremove' to remove it.

則用命令刪除之：`sudo apt-get autoremove`

- 再fix一次: ` sudo apt-get -f install`
同樣，若提示需要autoremove，則執行一次`sudo apt-get autoremove`

- `sudo apt-get -u dist-upgrade`

若提示有"held packages"，則用以下命令清除之，再執行上面的命令，直接沒有"held packages"提示為止。

```
sudo apt-get -o Debug::pkgProblemResolver=yes dist-upgrade
```

最終會提示如下：

```
0 upgraded, 0 newly installed, 0 to remove and 0 not upgraded.
```

- ` sudo apt-get autoclean `

- 最後，缺什麼軟件就用`apt-get install`安裝即可，依賴已修復

## **結論**

其實很簡單，把命令連起來就是以下：

```
sudo apt-get autoremove && sudo apt-get autoclean && sudo apt-get -f install && sudo dpkg-reconfigure -a && && sudo apt-get autoclean
```
