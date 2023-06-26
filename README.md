# multi_streamlit_apps_reverse_proxy

```shell
git clone ~~
```


## w/o ssl 
```shell
cd composedir
```

### 起動
```shell
docker compose build
docker compose up
```


---
## w/ ssl 
```shell
cd composedir_ssl
```

### 証明書のディレクトリ配置
```shell
mkdir certs
mkdir -p cert2/1 certs/2
```

### 証明書発行

```shell
sudo apt-get install certbot

sudo certbot certonly --standalone -d 1.devcfspi.com
sudo certbot certonly --standalone -d 2.devcfspi.com
```


### 証明書ごとにコピー配置
```shell
sudo cp /etc/letsencrypt/live/1.devcfspi.com/fullchain.pem  ./certs/1/
sudo cp /etc/letsencrypt/live/1.devcfspi.com/privkey.pem    ./certs/1/
```

```shell
sudo cp /etc/letsencrypt/live/2.devcfspi.com/fullchain.pem  ./certs/2/
sudo cp /etc/letsencrypt/live/2.devcfspi.com/privkey.pem    ./certs/2/
```

ディレクトリ配置はこんな感じ
```shell
$ tree
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


### 起動
```shell
docker compose build
docker compose up
```
