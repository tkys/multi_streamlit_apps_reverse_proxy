# multi_streamlit_apps_reverse_proxy


- 一つのインスタンスに　nginxコンテナ, streamlit_app1コンテナ, streamlit_app2コンテナ を docker comppose で起動

 
- reverse-proxyにて、サブドメイン別アクセスを同インスタンス内のアプリたちへ転送

  - サブドメイン 1.mydomain.com:80 へのリクエスト	->	streamlit_app1（:8501）へ
  - サブドメイン 2.mydomain.com:80 へのリクエスト	->	streamlit_app2（:8502）へ

* サブドメイン含めDNS設定は終えておく

---


## w/o ssl 
```shell
cd composedir
```

ディレクトリ配置はこんな感じ
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

### 起動
```shell
docker compose build
docker compose up
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
sudo ls /etc/letsencrypt/live/*

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

ディレクトリ配置はこんな感じ
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


### 起動
```shell
docker compose build
docker compose up
```

`https://1.mydomain.com`
`https://2.mydomain.com`
へアクセス出来ればOK
