# 概要

Hinemos による 「Docker の監視・ジョブ運用検証報告書」に記載の検証環境を構築する

https://www.hinemos.info/technology/nttdata/2015091501

* VirtualBox + Vagrant を用いて仮想サーバを構築する
* Docker-Compose を用いて複数コンテナを起動する

# 環境
## Linux サーバ一覧
<img src="./images/nw.png" width=70%>

| ホスト名 | OS | IPアドレス | 用途 |
| :- | :- | :- | :- |
| manager_server | CentOS7 | 192.168.50.101 | Hinemosマネージャ・エージェント，サンプルスクリプトをインストール |
| docker_host1 | CentOS7 | 192.168.50.102 | Docker が動作するサーバ、EtherCalc（表計算アプリ）を起動する |
| docker_host2 | CentOS7 | 192.168.50.103 | Docker が動作するサーバ、Libreboard（ToDoアプリ）を起動する |
| client_server | CentOS7 | 192.168.50.104 | Hinemos webクライアントが動作するサーバ |

# インストール方法

## 事前準備

### VirtualBox のインストール
* VirtualBox をインストール

	https://www.virtualbox.org/wiki/Downloads

* Host-Only ネットワークのアダプタで ``192.168.50.1`` を設定する

	https://qiita.com/koki276/items/28a87eb06abb4d8b5e13#%EF%BC%91virtuakbox%E3%81%AE%E3%83%9B%E3%82%B9%E3%83%88%E3%83%8D%E3%83%83%E3%83%88%E3%83%AF%E3%83%BC%E3%82%AF%E3%82%92%E8%A8%AD%E5%AE%9A

### Vagrant のインストール
* Vagrant をインストール

	https://www.vagrantup.com/downloads.html

	```
	$ vagrant -v
	```
* Vagrant プラグインをインストール

	```
	$ vagrant plugin install vagrant-hosts
	$ vagrant plugin install vagrant-reload
	$ vagrant plugin install vagrant-vbguest
	```

### リポジトリをクローン
* 任意のディレクトリ（例：ホームディレクトリ）に移動し、git リポジトリをクローン

	```
	$ git clone https://github.com/muroya2355/hinemos_docker.git
	```

* リポジトリに移動

	```
	$ cd hinemos_docker
	```

### サンプルスクリプトを配置
* Hinemos 公式サイトから、サンプルスクリプトの圧縮ファイルをダウンロード

	https://www.hinemos.info/themes/custom/hinemos/download/hinemos_docker_sample_v1.0.0.tar.gz


* ファイルを hinemos_docker/vagrant/shared/manager_server に配置

	```
	$ ls ./vagrant/shared/manager_server
	hinemos_docker_sample_v1.0.0.tar.gz
	```


## 環境構築

### 仮想マシン作成
* Vagrant の実行
	
	```
	$ cd ./vagrant
	$ vagrant up
	```
	⇒ エラーが出なければOK

### サーバを Hinemos に登録
* ブラウザから Hinemos Web クライアントにアクセス
	
	http://192.168.50.104/

* manager_server, docker_host1, docker_host2 をノードとして登録

	[参考] https://qiita.com/0ta2/items/8f9fb87938cc3083db89#%E7%9B%A3%E8%A6%96%E5%AF%BE%E8%B1%A1%E3%81%AE%E8%BF%BD%E5%8A%A0

### Docker コンテナの作成
* docker_host1 に SSH 接続
	
	```
	$ vagrant ssh docker_host1
	```

* コンテナの作成、起動	
	```
	[vagrant@docker_host1 ~]$ cd shared
	[vagrant@docker_host1 shared]$ ls
	docker-compose.yml
	[vagrant@docker_host1 shared]$ docker-compose up -d
	Creating network "shared_default" with the default driver
	Creating shared_data00redis_1 ... done
	Creating shared_redis_1       ... done
	Creating shared_ethercalc_1   ... done
	```

* ``docker ps -a`` と打ってコンテナが作成されていることを確認

* ``http://192.168.50.102:8000`` でアプリにアクセスできる

* ``exit`` で仮想マシンからログアウトできる

* 同様に docker_host2 にもログインしてコンテナ起動する．docker_host2 では ``http://192.168.50.103:5555`` でアクセスできる


### Docker コンテナを Hinemos に登録
* manager_server に SSH 接続
	```
	$ vagrant ssh manager_server
	```

* [docker_host1] コンテナ登録のスクリプトを実行
	```
	[vagrant@manager_server ~]$ sudo /opt/hinemos_agent/docker/bin/RepositoryDockerNodeAdd.py -i 192.168.50.102 -f docker_host1
	```
	⇒ Hinemos Web クライアントにアクセスして、コンテナが登録されていることを確認

* [docker_host2] コンテナ登録のスクリプトを実行
	```
	[vagrant@manager_server ~]$ sudo /opt/hinemos_agent/docker/bin/RepositoryDockerNodeAdd.py -i 192.168.50.103 -f docker_host2
	```
	⇒ ``WebFault: Server raised fault: 'FacilityInfo, primaryKey = docker_container'``というエラーを吐いて停止する。
	
	スコープが二重登録されることが問題なので、/opt/hinemos_agent/docker/bin/RepositoryDockerNodeAdd.py の１３６行目をコメントアウト
	```
	# register_docker_scopes(hclient, args.facility_id)
	```

	再度スクリプトを実行すると、別のエラーが吐かれて停止するが、コンテナは Hinemos に登録される

	⇒ Hinemos Web クライアントにアクセスして、コンテナが登録されていることを確認

## 報告書に従って検証を進める