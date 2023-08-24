# 手順書
## 使い方
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
挿入モードを終了させてから「:w」で保存,「:q」で終了

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
### 起動

```sh
docker compose up
```
