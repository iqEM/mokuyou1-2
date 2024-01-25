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
設定ファイルの中身(compose.yml)

https://github.com/iqEM/mokuyou1-2/blob/c1176de834ddcd9263d1c89c4d13fdf9a790ade6/compose.yml
```
### nginx

設定ファイルのディレクトリ作成とファイル作成
```
mkdir nginx
mkdir nginx/conf.d
vim nginx/conf.d/default.conf
```
https://github.com/iqEM/mokuyou1-2/blob/c1176de834ddcd9263d1c89c4d13fdf9a790ade6/nginx/conf.d/default.conf
配信するフォルダをつくる。
```
mkdir public
```
phpの設定ファイル
```
vim php.ini
```
https://github.com/iqEM/mokuyou1-2/blob/476f5522038218f73a08fff1ce4f738b82b4428a/php.ini

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
CREATE TABLE `users` (
    `id` INT UNSIGNED NOT NULL AUTO_INCREMENT PRIMARY KEY,
    `name` TEXT NOT NULL,
    `email` TEXT NOT NULL,
    `password` TEXT NOT NULL,
    `created_at` DATETIME DEFAULT CURRENT_TIMESTAMP
);
ALTER TABLE `users` ADD COLUMN icon_filename TEXT DEFAULT NULL;
CREATE TABLE `bbs_entries` (
    `id` INT UNSIGNED NOT NULL AUTO_INCREMENT PRIMARY KEY,
    `user_id` INT UNSIGNED NOT NULL,
    `body` TEXT NOT NULL,
    `image_filename` TEXT DEFAULT NULL,
    `created_at` DATETIME DEFAULT CURRENT_TIMESTAMP
);
CREATE TABLE `user_relationships` (
    `id` INT UNSIGNED NOT NULL AUTO_INCREMENT PRIMARY KEY,
    `followee_user_id` INT UNSIGNED NOT NULL,
    `follower_user_id` INT UNSIGNED NOT NULL,
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

会員登録機能と登録完了画面
```
vim public/signup.php
vim public/signup_finish.php
```
https://github.com/iqEM/mokuyou1-2/blob/476f5522038218f73a08fff1ce4f738b82b4428a/public/signup.php
https://github.com/iqEM/mokuyou1-2/blob/476f5522038218f73a08fff1ce4f738b82b4428a/public/signup_finish.php

ログインとログイン完了ページ
```
vim public/login.php
vim public/login_finish.php
```
https://github.com/iqEM/mokuyou1-2/blob/ef66f4b24a7384cae7b11de3f23207a0a3750221/public/login.php
https://github.com/iqEM/mokuyou1-2/blob/ef66f4b24a7384cae7b11de3f23207a0a3750221/public/login_finish.php

プロフィールページ
```
vim public/profile.php
```
https://github.com/iqEM/mokuyou1-2/blob/37597372258f457dbd559fef0e3c20d660c99337/public/profile.php

ユーザー設定画面
https://github.com/iqEM/mokuyou1-2/tree/cdd6a116446f043cbae32c9bd4426271ef9b8d2c/public/setting



フォロー機能
```
vim public/follow.php
```
https://github.com/iqEM/mokuyou1-2/blob/476f5522038218f73a08fff1ce4f738b82b4428a/public/follow.php

フォローしている会員一覧画面
```
vim public/follow_list.php
```
https://github.com/iqEM/mokuyou1-2/blob/ef66f4b24a7384cae7b11de3f23207a0a3750221/public/follow_list.php

タイムライン
```
vim public/timeline.php
vim public/timeline_json.php
```
https://github.com/iqEM/mokuyou1-2/blob/476f5522038218f73a08fff1ce4f738b82b4428a/public/timeline.php
https://github.com/iqEM/mokuyou1-2/blob/305d80a96f105d196e3de27092991ebcb6fa0a24/public/timeline_json.php

会員一覧ページ
```
vim public/users.php
```
https://github.com/iqEM/mokuyou1-2/blob/2f4ebca24abdef46c491d69d2a9fdd30c6f457a7/public/users.php


### 動作確認
EC2インスタンスを起動する。
Dockerの起動
```
docker compose up
```
#### ブラウザで確認
EC2インスタンスのアドレス/signup.php
