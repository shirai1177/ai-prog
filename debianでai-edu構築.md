# Ubuntu , Debianで環境構築のメモ

次を前提として構築  
* OS: ubuntu18.0
* Cloud: GCP n1-standard-1
* linuxユーザ: gcpadmin

## OSの基本設定とPythonインストール

### vimrcの編集

```
vi .vimrc

set tabstop=4
syntax off
set noautoindent
```

### 必要なパッケージのインストールとSWAPの有効化

```
sudo su -
apt-get -y update
apt-get -y install libpam-systemd dbus
apt-get -y install bzip2

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
rm /etc/localtime 
echo Asia/Tokyo > /etc/timezone 
dpkg-reconfigure -f noninteractive tzdata
```

SE-Linuxの無効化の確認

```
id -Z
"--context (-Z) は SELinux が有効なカーネルのみ動作します"と出れば、 SELINUX が無効の状態
```

ここで一旦、端末を抜けたあと再起動する。

### Anaconda3 (5.x.x) の入手とインストール

Anacondaの公式サイトから対応プラットフォーム版を入手しインストールする。

```
wget https://repo.anaconda.com/archive/Anaconda3-2019.07-Linux-x86_64.sh
bash Anaconda3-2019.07-Linux-x86_64.sh

gcpadminとrootで、パスを /home/admin/anaconda3/bin に通す
```

### TensorFlow と Keras のインストール

TensorFlowのモジュール名は、その時のバージョンにより適宜変更する。<br>

```
pip install tensorflow
pip install keras
```

> バージョンを指定してインストールする場合は<br>
> pip install tensorflow==1.3.0<br>

### jupyter notebook の設定

> Diins環境ではデフォルトの8888ポートでは動作しなかった。これはWebSocketがプロキシを
通過できないからと思われる。そこでhttps（443）を使って通信するように設定。
443ポートを使うためjupyter notebookはrootで起動する。

jupyter notebookの起動設定と起動はrootでの作業。<br>
あらかじめjupyter notebookにはパスを通す。<br>
作業用フォルダは ~/aihome <br>
httpsで起動するため、opensslで鍵の生成と登録を行う。<br>

```

mkdir aihome
jupyter notebook --generate-config
cd .jupyter
openssl req -x509 -nodes -days 3650 -newkey rsa:1024 -keyout mykey.key -out mycert.pem
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

### %matplotlib inline の指定を定義

```
ipython profile create
~/.ipython/profile_default/ipython_config.py を編集
c.InteractiveShellApp.exec_lines = ['%matplotlib inline']
```

### jupyter notebookのサービスへの登録

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

自動起動の設定をする前に、jupyter notebookの起動コマンドを、start_jupyter.shとして保存。実行権も与える

```
vi start_jupyter.sh

#!/bin/bash
/home/gcpadmin/anaconda3/bin/jupyter notebook > /root/jupyter.log 2>&1 &

chmod 755 start_jupyter.sh
```

サービスへの登録と起動・自動起動設定

```
cp jupyter.service /etc/systemd/system
chown root:root /etc/systemd/system/jupyter.service
systemctl daemon-reload
systemctl status jupyter.service
systemctl enable jupyter
systemctl start jupyter
```

## GitBucket のインストールと起動

GitBucket（gitbucket.war）を入手する。

```
mkdir gitbucket
cd gitbucket
wget https://github.com/gitbucket/gitbucket/releases/download/4.32.0/gitbucket.war
```

### java8をインストールし、GitBucketを起動する

```
sudo apt-get -y install default-jre

java -jar gitbucket.war
```

ポート番号はデフォルトで8080。<br>
変更したい場合は`--port=9090`などとオプションで指定する。

### 起動ファイルの作成

```
vi start_gitbucket.sh

#!/bin/bash
java -jar /home/gcpadmin/gitbucket/gitbucket.war > /home/gcpadmin/gitbucket/git.log 2>&1 &

chmod 755 start_gitbucket.sh
```

### GitBucketのサービス登録

ファイル gitbucket.service を作成し、サービスへ登録し自動起動の設定を行う。

```
[Unit]
Description=Gitbucket
After=network.target

[Service]
Type=forking
User=admin
Group=admin
ExecStart=/home/gcpadmin/gitbucket/start_gitbucket.sh

[Install]
WantedBy=multi-user.target
```

サービスへの登録と起動・自動起動設定

```
sudo cp gitbucket.service /etc/systemd/system
sudo chown root:root /etc/systemd/system/gitbucket.service
sudo systemctl daemon-reload
sudo systemctl status gitbucket.service
sudo systemctl enable gitbucket
sudo systemctl start gitbucket
```
