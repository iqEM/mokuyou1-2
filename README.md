# 手順書

### AWSの設定
EC2から「インスタンスの起動」を選択
### EC2インスタンスの起動
1. 名前を入力（適当）
2. キーペアの作成をする(タイプはrsa,ファイル形式は.pemファイル)
3. ネットワーク設定の「インターネットからの HTTPS、HTTP トラフィックを許可」にチェックボックスをいれる
4. ストレージ設定を無料分最大に設定する

### コマンドプロンプトでSSH接続をする
SSHでの接続は ssh ec2-user@{EC2インスタンスのアドレス} -i {.pemファイルのパス}
### Amazon Linux2 Vimのインストール

```
sudo yum install vim -y
```
## Vimの使い方
###vimの起動とファイルの開き方
```
vim ファイル名
```
存在しないファイルを指定すると新しく作成される。
### ファイルの編集
編集するには「i」キーを押すと挿入モードに入る。
挿入モードを終了するには、エスケープキーを押す。

### ファイルの保存とvimの終了
挿入モードを終了させてから「:w」で保存,「:q」で終了する。

### vimの設定
↓vimの設定ファイル
```
vim ~/.vimrc
```
設定内容
```
set number
set expandtab
set tabstop=2
set shiftwidth=2
```
他にも色々設定できるので自分の好みで設定しましょう。
### Docker インストール

```
sudo yum install -y docker
sudo systemctl start docker
sudo systemctl enable docker
sudo usermod -a -G docker ec2-user
```
### Docker Compose インストール
```
sudo mkdir -p /usr/local/lib/docker/cli-plugins/
sudo curl -SL https://github.com/docker/compose/releases/download/v2.2.2/docker-compose-linux-x86_64 -o /usr/local/lib/docker/cli-plugins/docker-compose
sudo chmod +x /usr/local/lib/docker/cli-plugins/docker-compose
```
インストールできたか確認
```
docker compose version
```

作業用のディレクトリを作ってその中に移動
```
mkdir dockertest
cd dockertest
```
設定ファイルをいじっていく。
```
vim compose.yml
```
設定ファイルの中身(compose.yml)
```
services:
   web:
     image: nginx:latest
     ports:
       - 80:80
     volumes:
       - ./nginx/conf.d/:/etc/nginx/conf.d/
       - ./public/:/var/www/public/
       - image:/var/www/upload/image/
      depends_on:
        - php
    php:
      container_name: php
      build:
        context: .
        target: php
      volumes:
        - ./public/:/var/www/public/
        - image:/var/www/upload/image/
    mysql:
      container_name: mysql
      image: mysql:8.0
      environment:
        MYSQL_DATABASE: hoge
        MYSQL_ALLOW_EMPTY_PASSWORD: 1
        TZ: Asia/Tokyo
      volumes:
        - mysql:/var/lib/mysql
      command: >
        mysqld
        --character-set-server=utf8mb4
        --collation-server=utf8mb4_unicode_ci
        --max_allowed_packet=4MB
  volumes:
    image:
      driver: local
    mysql:
```
### nginx

設定ファイルのディレクトリ作成とファイル作成
```
mkdir nginx
mkdir nginx/conf.d
vim nginx/conf.d/default.conf
```
```
server {
   listen  0.0.0.0:80;
   server_name _;
   charset utf-8;
   root /var/www/public;

   client_max_body_size 5M;

   location ~ \.php$ {
     fastcgi_pass php:9000;
     fastcgi_index index.php;
     fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
     include fastcgi_params;
   }

     location /image/ {
         root /var/www/upload;
     }
}
```
配信するフォルダをつくる。
```
mkdir public
```


### Dockerfileの設定
Dockerfileをカレントディレクトリに作成する。
```
vim Dockerfile
```
Dockerfileの設定内容
```
FROM php:8.1-fpm-alpine AS php
RUN docker-php-ext-install pdo_mysql
RUN install -o www-data -g www-data -d /var/www/upload/image/
RUN echo -e "post_max_size = 5M\nupload_max_filesize = 5M" >> ${PHP_INI_DIR}/php.ini
```
imageの構築
```
docker compose build
```
### Dockerコンテナ「mysql」内のMySQLサーバーに接続
```
docker compose exec mysql mysql hoge
```
docker compose exec 対象コンテナ名 mysql 接続するデータベース名
compose.ymlで設定した内容を参照してください。

### テーブルの作成
MySQLクライアントで下記のクエリを実行
```
CREATE TABLE `bbs_entries` (
    `id` INT UNSIGNED NOT NULL AUTO_INCREMENT PRIMARY KEY,
    `body` TEXT NOT NULL,
    `created_at` DATETIME DEFAULT CURRENT_TIMESTAMP
);
```
画像投稿機能の追加
```
ALTER TABLE `bbs_entries` ADD COLUMN image_filename TEXT DEFAULT NULL;
```
テーブルの中身の確認方法
```
SELECT * FROM テーブル名;
```

### アプリケーションの実装

ファイルの作成
```

```
中身は下記の通り
```

```

### 動作確認
EC2インスタンスを起動する。
Dockerの起動
```
docker compose up
```
#### ブラウザで確認
EC2インスタンスのアドレス/signup.php
