# WP Proxy Companion

## 參考來源
https://github.com/JrCs/docker-letsencrypt-nginx-proxy-companion/blob/master/docs/Advanced-usage.md

## 說明
本 Repo 為建立一個自動申請及更新 Let's Encrypt 憑證的 Container (將使用 80 及 443 Port)，並且可以讓多個網站容器共用。本專案是一個組合技概念，需要兩個 Repo 合作才能運行，所以設定完本 Repo "Wp Proxy Companion"後，還需要另一個 Repo：WP Proxy Sites ( https://github.com/mrmu/wp-proxy-sites ) 來負責網站各容器的實際設定，請搭配使用。另外，以下說明均以 Ubuntu 指令介紹。

## 運作原理說明
以 Nginx 作為 Reverse Proxy 的 Container，並以 Docker-gen 監看 Docker Network 裡各網站的 VIRTUAL_HOST 及 LETSENCRYPT_HOST 設定，生成 Nginx Conf 檔，依據設定自動在幾秒內生成憑證並且自動更新。

## 開始使用
1. 先安裝好該安裝的東西，若已安裝可跳過：
    * 安裝 Docker 和 Docker Compose
    ```
    sudo apt-get update
    sudo apt install docker.io
    ```
    * 啟用 docker
    ```
    sudo systemctl start docker
    sudo systemctl enable docker
    ```
    * 安裝 curl 
    ```
    sudo apt install curl
    ```
    * 安裝 docker-compose
    ```
    sudo curl -L "https://github.com/docker/compose/releases/download/1.24.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
    sudo chmod +x /usr/local/bin/docker-compose
    ```
    * 將自己的帳號加入 docker 群組，從此下指令就不加 sudo (下完指令下次登入後生效)：
    ```
    sudo usermod -a -G docker 你的帳號
    ```
2. 請確認 80/443/3306 Port 有開啟並且沒有被其他服務佔用 (因為接下來 wp-proxy-companion 會佔用它們，如果已經有在跑 apache/nginx、mysql/mariaDB 等服務，請先停止或另尋其他空間安裝，要停止常見的服務比如 apache 就用 sudo service apache2 stop 就好)。
    ```
    sudo lsof -i -P -n | grep LISTEN
    ```
3. 請先建立 Docker Network 以便之後串連 Wp Proxy Companion 和 Wp Proxy Sites，預設取名為 wp-proxy，指令如下：
    ```
    docker network create wp-proxy
    ```
4. 如果還沒安裝 git，也要安裝：
    ```
    sudo apt-get install git-all
    ```
5. 另建目錄或找個目錄存放本設定和網站相關檔案，比如正式環境可以放 /var/docker-www/ (名稱隨你取) 或本機放 /Users/xxxx/，進入該目錄後 git clone 此 repo。(看你有沒有其他慣放 Docker 設定的目錄也行)：
    ```
    cd /var
    sudo mkdir docker-www
    sudo chown 可寫入權限:可寫作權限 docker-www
    cd /var/docker-www/
    git clone https://github.com/mrmu/wp-proxy-companion.git
    ```
6. 進入 wp-proxy-companion 目錄，你可以瞄一下 docker-compose.yml 看看待會要建立的容器內容，後面會有說明，現在我們先開始建立容器，下指令 (如果下完指令發生錯誤訊息，很可能是 Port 有被佔用，請見上方第2步的說明來排除問題，排除後再執行一次以下指令應該就會成功了)：
    ```
    docker-compose up -d --build
    ```
7. 現在 wp proxy companion 會在背景運行監看，之後只要有新的網站容器加入，companion 就會自動做完指向的工作。以下是實際工作的說明，如果沒興趣可往下一步XD… 

    總之，companion 只要確認新的網站容器設定了 VIRTUAL_HOST 環境變數， wp proxy companion 就會幫忙完成反向代理，讓該網址可以指向正確的網站容器；若之後再設定 LETSENCRYPT_HOST 變數 (需先完成網址 DNS 指向)，wp proxy companion 就會在 5~30 秒內通知 Let's Encrypt 發出 challenge 請求，通過後就會自動安裝 HTTPS (Let's Encrypt) 憑證。在全站網址改為 https 前，請確認 wp-proxy-companion/nginx/certs 裡有該網域的憑證檔 (*.chain.pem, *.key, *.crt, *.dhparam.pem)

8. Wp Proxy Companion 正常運作後，request 會是由 nginx 先處理，再進行反向代理指向正確的網站容器(apache)，所以預設找不到的情況下就會出現 503 Service Temporarily Unavailable，現在還沒有加入任何網站容器，你開瀏覽器輸入 該主機 ip 或指向該主機的網址就會顯示 503。

8. 現在開始建立 WordPress 網站，請安裝運行：WP Proxy Sites ( https://github.com/mrmu/wp-proxy-sites )

9. 基本上 Wp-Proxy-Companion 就是執行一次放在背景跑，之後開發網站時除非發生嚴重錯誤，否則幾乎不太會再動到它了。

## 工作流程
<img width="1434" alt="wp-proxy-scope" src="https://user-images.githubusercontent.com/271049/61531755-5ddc2d80-aa5a-11e9-9383-ac099f4eb3a6.png">

本 docker-compose.yml 共設定三個容器：
1. wp-proxy：它的 image 來源是 nginx，主要是拿來作反向代理，看要將請求轉向哪個網站容器 (WP + apache)
2. wp-proxy-gen：它的 image 來源是 jwilder/docker-gen，主要是自動產生對應網站容器的 nginx 設定檔，如果 docker 跑起來了，但瀏覽器卻連不到網站容器，很大的可能是 docker-gen 沒有正確完成它的工作，此時要去 nginx/ 下查看 default.conf 有沒有生成，並且看 default.conf 的內容有沒有該網站容器的設定。
3. wp-proxy-letsencrypt：它的 image 來源是 jrcs/letsencrypt-nginx-proxy-companion，主要是負責自動產生及renew 各網站容器的 Let's Encrypt 憑證。

## 其他
* 查詢 let's encrypt 憑證的狀況
```
docker exec wp-proxy-letsencrypt /app/cert_status
```

* 強制 renew
```
docker exec wp-proxy-letsencrypt /app/force_renew
```
