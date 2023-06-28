# multi_streamlit_apps_reverse_proxy


- 一つのインスタンスに
   - nginxコンテナ
   - streamlit_app1コンテナ
   - streamlit_app2コンテナ
     
    を docker comppose で起動

 
- reverse-proxyにて、サブドメイン別アクセスを同インスタンス内のアプリたちへ転送

  - サブドメイン 1.mydomain.com:80 へのリクエスト	->	streamlit_app1（:8501）へ
  - サブドメイン 2.mydomain.com:80 へのリクエスト	->	streamlit_app2（:8502）へ

* サブドメイン含めDNS設定は終えておく
* certbotにてSSL証明書発行しhttps対応

---


## w/o ssl 
```shell
cd composedir
```

ディレクトリ配置
```shell
$ tree ../composedir

../composedir
.
├── docker-compose.yaml
├── nginx-reverse-proxy
│   └── apps_nginx.conf
│
├── streamlit_1
│   ├── Dockerfile
│   ├── app.py
│   └── requirements.txt
│
└── streamlit_2
    ├── Dockerfile
    ├── app.py
    └── requirements.txt
```


docker-compose.yaml 
```
version: '3'

services:
  nginx:
    image: nginx
    container_name: nginx-reverse-proxy
    volumes:
      - ./nginx-reverse-proxy/apps_nginx.conf:/etc/nginx/conf.d/apps_nginx.conf
    ports:
      - 80:80
    networks:
      - mynetwork

  streamlit_1:
    build:
      context: ./streamlit_1
      dockerfile: Dockerfile
    container_name: streamlit_1
    working_dir: /app
    volumes:
      - ./streamlit_1:/app
    ports:
      - 8501:8501
    networks:
      - mynetwork

  streamlit_2:
    build:
      context: ./streamlit_2
      dockerfile: Dockerfile
    container_name: streamlit_2
    working_dir: /app
    volumes:
      - ./streamlit_2:/app
    ports:
      - 8502:8502
    networks:
      - mynetwork

networks:
  mynetwork:

```
./composedir/nginx-reverse-proxy/apps_nginx.conf 
```
server {
    listen 80;
    server_name 1.mydomain.com;

    location / {
        proxy_pass http://streamlit_1:8501;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}

server {
    listen 80;
    server_name 2.mydomain.com;

    location / {
        proxy_pass http://streamlit_2:8502;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

### ビルド、起動
```shell
docker compose build
docker compose up
```
```
docker compose ps
NAME                  IMAGE                    COMMAND                  SERVICE             CREATED             STATUS              PORTS
nginx-reverse-proxy   nginx                    "/docker-entrypoint.…"   nginx               23 hours ago        Up 11 seconds       0.0.0.0:80->80/tcp, :::80->80/tcp
streamlit_1           composedir-streamlit_1   "streamlit run app.p…"   streamlit_1         23 hours ago        Up 11 seconds       0.0.0.0:8501->8501/tcp, :::8501->8501/tcp
streamlit_2           composedir-streamlit_2   "streamlit run app.p…"   streamlit_2         22 hours ago        Up 11 seconds       0.0.0.0:8502->8502/tcp, :::8502->8502/tcp
```

 `http://1.mydomain.com`
 `http://2.mydomain.com`
へアクセス出来ればOK


---
## w/ ssl 
```shell
cd composedir_ssl
```

### `certbot`のインストール、証明書発行

```shell
#インストール
sudo apt-get install certbot

#発行
sudo certbot certonly --standalone -d 1.mydomain.com
sudo certbot certonly --standalone -d 2.mydomain.com

#確認
sudo tree /etc/letsencrypt/live/

```



### 各アプリドメイン毎に証明書をコピー配置

```shell

mkdir -p certs/1
sudo cp /etc/letsencrypt/live/1.devcfspi.com/fullchain.pem  ./certs/1/
sudo cp /etc/letsencrypt/live/1.devcfspi.com/privkey.pem    ./certs/1/

mkdir -p certs/2
sudo cp /etc/letsencrypt/live/2.devcfspi.com/fullchain.pem  ./certs/2/
sudo cp /etc/letsencrypt/live/2.devcfspi.com/privkey.pem    ./certs/2/

```

ディレクトリ配置
```shell
$ tree ../composedir_ssl

../composedir_ssl
.
├── certs
│   ├── 1
│   │   ├── fullchain.pem
│   │   └── privkey.pem
│   └── 2
│       ├── fullchain.pem
│       └── privkey.pem
│
├── docker-compose.yaml
├── nginx-reverse-proxy
│   └── apps_nginx.conf
│
├── streamlit_1
│   ├── Dockerfile
│   ├── app.py
│   └── requirements.txt
│
└── streamlit_2
    ├── Dockerfile
    ├── app.py
    └── requirements.txt
```

docker-compose.yaml 
```
version: '3'

services:
  nginx:
    image: nginx
    container_name: nginx-reverse-proxy_ssl
    volumes:
      - ./nginx-reverse-proxy/apps_nginx.conf:/etc/nginx/conf.d/apps_nginx.conf
      - ./certs:/etc/nginx/certs #証明書ディレクトリ
    ports:
      - 80:80
      - 443:443 #https
    networks:
      - mynetwork

  streamlit_1:
    build:
      context: ./streamlit_1
      dockerfile: Dockerfile
    container_name: streamlit_1_ssl
    working_dir: /app
    volumes:
      - ./streamlit_1:/app
    ports:
      - 8501:8501
    networks:
      - mynetwork

  streamlit_2:
    build:
      context: ./streamlit_2
      dockerfile: Dockerfile
    container_name: streamlit_2_ssl
    working_dir: /app
    volumes:
      - ./streamlit_2:/app
    ports:
      - 8502:8502
    networks:
      - mynetwork

networks:
  mynetwork:
```

./composedir_ssl/nginx-reverse-proxy/apps_nginx.conf 
```
server {
    listen 80;
    server_name 1.mydomain.com;
    return 301 https://$host$request_uri;  # httpからhttpsへのリダイレクト
}

server {
    listen 443 ssl;
    server_name 1.mydomain.com;

    ssl_certificate /etc/nginx/certs/1/fullchain.pem;  # サブドメイン1の証明書のパス
    ssl_certificate_key /etc/nginx/certs/1/privkey.pem;  # サブドメイン1の秘密鍵のパス

    location / {
        proxy_pass http://streamlit_1:8501;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}

server {
    listen 80;
    server_name 2.mydomain.com;
    return 301 https://$host$request_uri;  # httpからhttpsへのリダイレクト
}

server {
    listen 443 ssl;
    server_name 2.mydomain.com;

    ssl_certificate /etc/nginx/certs/2/fullchain.pem;  # サブドメイン2の証明書のパス
    ssl_certificate_key /etc/nginx/certs/2/privkey.pem;  # サブドメイン2の秘密鍵のパス

    location / {
        proxy_pass http://streamlit_2:8502;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

### ビルド、起動
```shell
docker compose build
docker compose up
```
```
docker compose ps
NAME                      IMAGE                        COMMAND                  SERVICE             CREATED             STATUS              PORTS
nginx-reverse-proxy_ssl   nginx                        "/docker-entrypoint.…"   nginx               7 seconds ago       Up 5 seconds        0.0.0.0:80->80/tcp, :::80->80/tcp, 0.0.0.0:443->443/tcp, :::443->443/tcp
streamlit_1_ssl           composedir_ssl-streamlit_1   "streamlit run app.p…"   streamlit_1         7 seconds ago       Up 5 seconds        0.0.0.0:8501->8501/tcp, :::8501->8501/tcp
streamlit_2_ssl           composedir_ssl-streamlit_2   "streamlit run app.p…"   streamlit_2         7 seconds ago       Up 5 seconds        0.0.0.0:8502->8502/tcp, :::8502->8502/tcp
```

`https://1.mydomain.com`
`https://2.mydomain.com`
へアクセス出来ればOK
