# WP Proxy Companion

## 參考來源
https://github.com/JrCs/docker-letsencrypt-nginx-proxy-companion/blob/master/docs/Advanced-usage.md

## 說明
本 Repo 為建立一個自動申請及更新 Let's Encrypt 憑證的 Container (將使用 80 及 443 Port)，並且可以讓多個網站容器共用。本專案是一個組合技概念，需要兩個 Repo 合作才能運行，所以設定完本 Repo "Wp Proxy Companion"後，還需要另一個 Repo：WP Proxy Sites ( https://github.com/mrmu/wp-proxy-sites ) 來負責網站各容器的實際設定，請搭配使用。

## 運作原理說明
以 Nginx 作為 Reverse Proxy 的 Container，並以 Docker-gen 監看 Docker Network 裡各網站的 VIRTUAL_HOST 及 LETSENCRYPT_HOST 設定，生成 Nginx Conf 檔，依據設定自動在幾秒內生成憑證並且自動更新。

## 開始使用
1. 請先安裝 Docker 及 Docker Compose。
2. 請確認 80 及 443 Port 有開啟並且沒有被其他服務佔用。
3. 請先建立 Docker Network 以便之後串連 Wp Proxy Companion 和 Wp Proxy Sites，要取名為 wp-proxy，指令如下：
```
  docker network create wp-proxy
```
4. 找個目錄存放本設定和網站相關檔案，假設為 /Users/xxxx/，進入該目錄後 git clone 此 repo。(看你有沒有其他慣放 Docker 設定的目錄也行)
5. 到這步 /Users/xxxx/ 會有 wp-proxy-companion 這個目錄。
6. 進入 wp-proxy-companion 目錄，開始建立 wp proxy companion 容器，下指令：
```
  docker-compose up -d --build
```
7. 現在 wp proxy companion 會在背景運行監看，之後只要有新的網站容器加入，並且設定了 VIRTUAL_HOST 環境變數， wp proxy companion 就會幫忙完成反向代理，讓該網址可以指向正確的網站容器；若設定了 LETSENCRYPT_HOST 變數 (需先完成網址 DNS 指向)，wp proxy companion 就會在 5~30 秒內自動安裝 HTTPS (Let's Encrypt) 憑證。

8. 開始建立 WordPress 網站，請安裝運行：WP Proxy Sites ( https://github.com/mrmu/wp-proxy-sites )

9. 基本上 Wp-Proxy-Companion 就是執行一次放在背景跑，之後開發網站時除非發生嚴重錯誤，否則幾乎不太會再動到它了。

## 其他
* 查詢 let's encrypt 憑證的狀況
```
docker exec wp-proxy-letsencrypt /app/cert_status
```

* 強制 renew
```
docker exec wp-proxy-letsencrypt /app/force_renew
```
