# 手順書
## 使い方
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
設定ファイルの中身
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
        MYSQL_DATABASE: techc
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
### 起動

```sh
docker compose up
```
