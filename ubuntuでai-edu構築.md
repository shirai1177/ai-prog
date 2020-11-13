# AIプログラミング環境構築メモ

次を前提として構築  
* OS: ubuntu16系
* linuxユーザ: root
* ストレージは10GBほどあったほうが良い


## OS関連の設定

GCP,AWSともSWAPを有効にする

```
sudo su -
apt-get update
apt-get upgrade -y
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

ここらでシステム反映を確認するためリブート

```
reboot
```

### crontab設定

インスタンスの落とし忘れ防止<br>
rootで設定

```
crontab -e
15 18 * * * /sbin/shutdown -h now
```


### Anaconda3 (5.x.x) の入手とインストール

https://www.anaconda.com/download/ から対応プラットフォーム版を入手しインストールする。パスも通す。

```
wget https://repo.anaconda.com/archive/Anaconda3-2019.07-Linux-x86_64.sh
bash Anaconda3-2019.07-Linux-x86_64.sh -b

# インストールが成功したら元ファイルは消しておくのが無難。
rm Anaconda3-2019.07-Linux-x86_64.sh

vi .profile
PATH=$HOME/anaconda3/bin:$PATH
```

### TensorFlow と Keras のインストール

TensorFlowのモジュール名は、その時のバージョンにより適宜変更する。<br>

```
pip install tensorflow==2.0.0
pip install keras==2.4.1

# tensorflowバージョンの確認
python -c "import tensorflow as tf; print(tf.__version__)"
```

> tensorflow2系のpredict()がメモリーリークしているため、1系をインストールする方法<br>
> 特に強化学習では現時点(2020/8/18)では2系は使えない<br>
> pip install tensorflow==1.15.3<br>
> pip install keras==2.2.4

### jupyter notebook の設定

> Diins環境ではデフォルトの8888ポートでは動作しなかった。これはWebSocketがプロキシを
通過できないからと思われる。そこでhttps（443）を使って通信するように設定した。
443ポートを使うためjupyter notebookはrootで起動する。

jupyter notebookの起動設定と起動はrootでの作業。<br>
あらかじめjupyter notebookにはパスを通す。<br>
作業用フォルダは ~/aihome とする。<br>
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
vi ~/.ipython/profile_default/ipython_config.py
c.InteractiveShellApp.exec_lines = ['%matplotlib inline']
```

### jupyter notebook の起動

SSLの標準ポート（443）を使う場合、rootで起動する必要がある。
起動ファイルの作成(Jupyterはフルパスを指定)

```
vi start_jupyter.sh
#!/bin/bash
/root/anaconda3/bin/jupyter notebook > /root/jupyter.log 2>&1 &

chmod 755 start_jupyter.sh
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


# （おまけ） 

## GitBucket のインストールと起動

作業は一般ユーザ

GitBucket（gitbucket.war）を入手する。

```
mkdir gitbucket
cd !$

今使っている最新はこちら
wget https://github.com/gitbucket/gitbucket/releases/download/4.31.1/gitbucket.war
```

### java8をインストールし、GitBucketを起動する

gitbucketの実行ファイルはフルパスを指定する

```
sudo apt-get install openjdk-8-jre

vi start_gitbucket.sh
#!/bin/bash
java -jar /root/gitbucket/gitbucket.war --port=80 > /root/gitbucket/log 2>&1 &

chmod 755 start_gitbucket.sh
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
User=root
#Group=yyyyyy
ExecStart=/root/gitbucket/start_gitbucket.sh

[Install]
WantedBy=multi-user.target
```

```
sudo cp gitbucket.service /etc/systemd/system
sudo systemctl enable gitbucket
sudo systemctl start gitbucket
```

