# インストール方法
## VirtualBox のインストール
* VirtualBox をインストール
	https://www.virtualbox.org/wiki/Downloads

## Vagrant のインストール
* Vagrant をインストール
	https://www.vagrantup.com/downloads.html
	```
	$ vagrant -v
	```
* Vagrant プラグインをインストール
	```
	$ vagrant plugin install vagrant-hosts
	$ vagrant plugin install vagrant-reload
	$ vagrant plugin install vagrant-reload
	```

## リポジトリをクローン
* 任意のディレクトリ（例：ホームディレクトリ）に移動し、git リポジトリをクローン

* リポジトリに移動

## サンプルスクリプトを配置
* サンプルスクリプトの圧縮ファイルをダウンロード

* ファイルを指定のディレクトリに配置

## 

# 環境
## Linux サーバ一覧
<img src="./images/nw.png" width=70%>

| ホスト名 | OS | IPアドレス | 用途 |
| :- | :- | :- | :- |
| manager_server | CentOS7 | 192.168.50.101 | Hinemosマネージャ・エージェント，サンプルスクリプトをインストール |
| docker_host1 | CentOS7 | 192.168.50.102 | Docker が動作するサーバ |
| docker_host2 | CentOS7 | 192.168.50.103 | Docker が動作するサーバ |
| client_server | CentOS7 | 192.168.50.104 | Hinemos webクライアントが動作するサーバ |