# WP Proxy Companion

## 參考來源
https://github.com/JrCs/docker-letsencrypt-nginx-proxy-companion/blob/master/docs/Advanced-usage.md

## 說明
本 Repo 為建立一個自動申請及更新 Let's Encrypt 憑證的 Container (將使用 80 及 443 Port)，並且可以讓多個網站容器共用。

## 運作原理說明
以 Nginx 作為 Reverse Proxy 的 Container，並以 Docker-gen 監看 Docker Network 裡各網站的 VIRTUAL_HOST 及 LETSENCRYPT_HOST 設定，生成 Nginx Conf 檔，依據設定自動在幾秒內生成憑證並且自動更新。

## 開始使用
* 請先安裝 Docker 及 Docker Compose。
* 請確認 80 及 443 Port 有開啟並且沒有被其他服務佔用。
* 請先建立 Docker Network，取名 wp-proxy：
```
  docker network create wp-proxy
```
* 下載本 Repo，建議 git clone 即可。
* 建立 wp proxy companion 容器：
```
  docker-compose up -d --build
```
* 現在 wp proxy companion 會在背景運行，之後只要有新的網站容器加入，並且設定了 VIRTUAL_HOST 環境變數， wp proxy companion 就會幫忙完成反向代理，讓該網址可以指向正確的網站容器；若設定了 LETSENCRYPT_HOST 變數 (需先完成網址 DNS 指向)，wp proxy companion 就會在 5~30 秒內自動安裝 HTTPS (Let's Encrypt) 憑證。

* 開始建立 WordPress 網站，請另外使用這個 Repo：WP Proxy Sites (https://github.com/mrmu/wp-proxy-sites)

## 其他
* 查詢 let's encrypt 憑證的狀況
```
docker exec wp-proxy-letsencrypt /app/cert_status
```

* 強制 renew
```
docker exec wp-proxy-letsencrypt /app/force_renew
```
