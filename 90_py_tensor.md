# AIプログラミング環境構築メモ

次を前提として構築  
* OS: ubuntu16系
* Cloud: GCP f1.micro または AWS t2.micro
* linuxユーザ: gcpadmin

## 開発者用サーバの場合の権限

cronで自動的にスナップショットバックアップを取得することを想定し、以下の設定を行なう。<br>
Compute Engine のストレージ権限に、「読み取り / 書き込み」を追加（後からの権限追加はできないので注意）

### スナップショットの取得コマンド

$ gcloud compute disks snapshot ai-edu2 --zone=us-east1-b --snapshot-names=ai-edu-base

* ディスク名：ai-edu2<br>
* ゾーン：us-east1-b<br>
* スナップショット名：ai-edu-base<br>

次のスクリプトをcronから起動する。

base_create.sh

```
#!/bin/bash
#
# gcloud config set compute/zone
#
gcloud compute snapshots delete ai-edu-snap2 -q
gcloud compute snapshots delete ai-edu-base -q
gcloud compute disks snapshot ai-edu2 --zone=us-east1-b --snapshot-names=ai-edu-base
```

snap_create.sh

```
#!/bin/bash
#
gcloud compute snapshots delete ai-edu-snap2 -q
gcloud compute disks snapshot ai-edu2 --zone=us-east1-b --snapshot-names=ai-edu-snap2
```

## その他メモ

### 他のユーザにプロジェクトのイメージを見せるための設定

GCPコンソールから対象のプロジェクトを選んで、「IAM」-「メンバー追加」<br>
権限として以下を選択<br>
プロジェクト - 参照者<br>
Computeエンジン - Computeイメージユーザー

## Python インストール

GCP,AWSともにmicroインスタンスの場合、SWAPを有効にしないと厳しい

```
sudo su -
dd if=/dev/zero of=/swapfile bs=1M count=2048
chmod 600 /swapfile
mkswap /swapfile
swapon /swapfile
free -m
cp -P /etc/fstab /etc/fstab.org
echo "/swapfile swap swap defaults 0 0" >> /etc/fstab
```

システムのタイムゾーンを東京に設定

```
timedatectl set-timezone Asia/Tokyo
```

SE-Linuxを無効化

```
vi /etc/selinux/config
SELINUX=disabled

reboot
```

### Anaconda3 (5.x.x) の入手とインストール

https://www.anaconda.com/download/ から対応プラットフォーム版を入手しインストールする。パスも通す。

```
bash Anaconda3-2019.07-Linux-x86_64.sh -b

vi .profile
PATH=$HOME/anaconda3/bin:$PATH
```

### TensorFlow と Keras のインストール

TensorFlowのモジュール名は、その時のバージョンにより適宜変更する。<br>

```
pip install tensorflow==2.0.0
pip install keras
```

### jupyter notebook の設定

> Diins環境ではデフォルトの8888ポートでは動作しなかった。これはWebSocketがプロキシを
通過できないからと思われる。そこでhttps（443）を使って通信するように設定した。
443ポートを使うためjupyter notebookはrootで起動する。

jupyter notebookの起動設定と起動はrootでの作業。<br>
あらかじめjupyter notebookにはパスを通す。<br>
作業用フォルダは ~/aihome とした。<br>
httpsで起動するため、opensslで鍵の生成と登録を行う。<br>

```
mkdir aihome
jupyter notebook --generate-config
cd .jupyter
openssl req -x509 -nodes -days 365 -newkey rsa:1024 -keyout mykey.key -out mycert.pem
vi jupyter_notebook_config.py
c.NotebookApp.ip = '*'
c.NotebookApp.notebook_dir = '/root/aihome'
c.NotebookApp.open_browser = False
c.NotebookApp.token = 'StudyAI123'
c.NotebookApp.certfile = '/root/.jupyter/mycert.pem'
c.NotebookApp.keyfile = '/root/.jupyter/mykey.key'
c.NotebookApp.port = 443
c.NotebookApp.allow_remote_access = True
c.NotebookApp.allow_root = True
c.NotebookApp.terminals_enabled = False
c.NotebookApp.allow_password_change = False
```

### jupyter notebook の起動

SSLの標準ポート（443）を使う場合、rootで起動する必要がある。
起動ファイルの作成

```
vi start_jupyter
#!/bin/bash
/home/gcpadmin/anaconda3/bin/jupyter notebook > /root/jupyter.log 2>&1 &
```

### サービスへの登録

ファイル jupyter.service を作成し、サービスへ登録し自動起動の設定を行う。

```
[Unit]
Description=Jupyter notebook
After=network.target
[Service]
Type=forking
User=root
ExecStart=/root/start_jupyter.sh
[Install]
WantedBy=multi-user.target
```

```
cp jupyter.service /etc/systemd/system
systemctl enable jupyter
systemctl start jupyter
```


## GitBucket のインストールと起動

> ドキュメント管理にGitHubを使いたかったが、プライベートリポジトリの作成は有料であるためGitHubクローンを使用

GitBucket（gitbucket.war）を入手する。

```
wget https://github.com/gitbucket/gitbucket/releases/download/4.18.0/gitbucket.war
2018/09/30時点の最新は以下（一応動作確認済）
wget https://github.com/gitbucket/gitbucket/releases/download/4.29.0/gitbucket.war
今使っている最新はこちら
wget https://github.com/gitbucket/gitbucket/releases/download/4.31.1/gitbucket.war
```

### java8をインストールし、GitBucketを起動する

```
sudo apt-get install openjdk-8-jre

# start_gitbucket.sh へ以下を保存

#!/bin/bash
java -jar /home/gcpadmin/gitbucket/gitbucket.war > /home/gcpadmin/gitbucket/log 2>&1 &
```
ポート番号はデフォルトで8080。<br>
変更したい場合は`--port=9090`などとオプションで指定する。

### GitBucketのサービス登録

ファイル gitbucket.service を作成し、サービスへ登録し自動起動の設定を行う。

```
[Unit]
Description=Gitbucket
After=network.target

[Service]
Type=forking
User=gcpadmin
Group=gcpadmin
ExecStart=/home/gcpadmin/gitbucket/start_gitbucket.sh

[Install]
WantedBy=multi-user.target
```
