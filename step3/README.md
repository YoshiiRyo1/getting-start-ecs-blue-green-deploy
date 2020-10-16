# Step 3
## ECR
リポジトリを作成します。  
<a href="https://ap-northeast-1.console.aws.amazon.com/ecr/repositories?region=ap-northeast-1" target="_blank">ECR 画面</a> を開きます。  

**リポジトリを作成** をクリックしてリポジトリを作成します。  
今回は2つ作成してください。  

* tomcat10
* nginx1192

Cloud9 のコンソールから以下を実行します。(aws_account_id はご自身のものへ変更)  
**Login Succeeded** と表示されれば成功です。  

```
aws ecr get-login-password | docker login --username AWS --password-stdin https://<aws_account_id>.dkr.ecr.ap-northeast-1.amazonaws.com

Login Succeeded
```

## Dockerfile
作業用の任意のディレクトリを作成します。  
そのディレクトリ配下を以下のような構成にします。  
ファイルはこれから作成します。ファイル作成時に Cloud9 の素晴らしさが実感できるはずです。ssh で作業することとの違いを感じてもらえれば嬉しいです。  

```
.
├── docker-compose.yml
├── nginx
│   ├── Dockerfile
│   ├── conf
│   │   └── default.conf
│   └── html
│       └── index.html
└── tomcat
    ├── Dockerfile
```

### docker-compose.yml
以下の内容でファイルを新規作成します。(aws_account_id と region はご自身のものへ変更)  

<details>
<summary>docker-compose.yml</summary><div>

```
version: '3'
services:
  tomcat:
    build: ./tomcat
    image: <aws_account_id>.dkr.ecr.<region>.amazonaws.com/tomcat10
    environment:
      TZ: "Asia/Tokyo"
    ports:
      - "8080:8080"
  web:
    build: ./nginx
    image: <aws_account_id>.dkr.ecr.<region>.amazonaws.com/nginx1192
    environment:
      TZ: "Asia/Tokyo"
    ports:
      - "80:80"
```
</div></details>

### nginx/Dockerfile
Docker Hub の公式イメージを使用します。  

<details>
<summary>nginx/Dockerfile</summary><div>

```
FROM nginx:1.19.2

COPY ./conf /etc/nginx/conf.d/
COPY ./html /usr/share/nginx/html/
```
</div></details>

### nginx/conf/default.conf
フロント用 nginx です。  
80番ポートで受付、裏側の tomcat に流しています。  

<details>
<summary>nginx/conf/default.conf</summary><div>

```
server {
    listen 80;
    
    access_log /dev/stdout;
    error_log /dev/stdout;

    location / {
        root /usr/share/nginx/html;
        index index.html index.htm;
    }
    
    location = /health {
        empty_gif;
        break;
    }
    
    location /sample {
        proxy_set_header X-Forwarded-Host $host:$server_port;
        proxy_set_header X-Forwarded-Server $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_pass http://localhost:8080/sample/;
    }
}
```
</div></details>

### nginx/html/index.html
動作確認用の index.html です。  

<details>
<summary>nginx/html/index.html</summary><div>

```
<html lang="ja">
<head>
  <meta charset="UTF-8">
  <title>HTML Sample</title>
</head>
<body>
    <h1>HTML Sample</h1>
</body>
</html>
```
</div></details>

### tomcat/Dockerfile
Docker Hub の公式イメージを使用します。  
サンプルの war をデプロイします。  

<details>
<summary>tomcat/Dockerfile</summary><div>

```
FROM tomcat:10.0.0-jdk11-corretto

WORKDIR /usr/local/tomcat/webapps/
RUN curl -O hhttps://tomcat.apache.org/tomcat-10.0-doc/appdev/sample/sample.war

CMD ["catalina.sh", "run"]
```
</div></details>

### ローカルで動作確認
ファイル一式の作成が完了したらローカルで動作確認をします。  
docker-compose.yml が存在するディレクトリで以下のコマンドを実行します。  

```
docker-compose up -d --build

docker-compose ps
  Name                 Command               State           Ports         
---------------------------------------------------------------------------
nginx1192   /docker-entrypoint.sh ngin ...   Up      0.0.0.0:80->80/tcp    
tomcat10    catalina.sh run                  Up      0.0.0.0:8080->8080/tcp
```

上記のように2つのコンテナが State = Up と表示されれば OK です。  

アクセスしてみましょう。  

```
curl http://localhost/index.html
～～省略～～
<body>
    <h1>HTML Sample</h1>
</body>
～～省略～～


curl http://localhost:8080/sample
～～省略～～
<h1>Sample "Hello, World" Application</h1>
<p>This is the home page for a sample application used to illustrate the
source directory organization of a web application utilizing the principles
outlined in the Application Developer's Guide.
～～省略～～
```

うまく動かない、コンテナ内部を調べたい、そんな場合は以下のコマンドです。  

```
docker exec -it nginx1192 bash
```

特に問題がなければ停止しておきます。  
```
docker-compose down
```

### ECR へプッシュ
正常に動作するイメージは ECR へプッシュします。  
後ほど ECS 上で起動してもらいます。  

```
eval $(aws ecr get-login --no-include-email --region ap-northeast-1)
docker-compose build
docker-compose push
```

----

[Step4](../step4/README.md)
