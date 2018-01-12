# AIプログラミング環境構築メモ

次を前提として構築  
* OS: CentOS7
* Cloud: GCP f1.micro または AWS t2.micro
* linuxユーザ: admin

## Python インストール

GCP,AWSともにmicroインスタンスの場合、SWAPを有効にしないと厳しい

```
sudo su -
dd if=/dev/zero of=/swapfile bs=1M count=2048
chmod 600 /swapfile
mkswap /swapfile
swapon /swapfile
free -m
vi /etc/fstab
/swapfile                                 swap                    swap    defaults        0 0
```

SE-Linuxを無効化

```
vi /etc/selinux/config
SELINUX=disabled

reboot
```

### Anaconda3 (5.x.x) の入手とインストール

https://www.anaconda.com/download/ から対応プラットフォーム版を入手しインストールする。

```
sudo yum -y install bzip2
bash Anaconda3-5.0.0.1-Linux-x86_64.sh
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
```

### jupyter notebook の起動

SSLの標準ポート（443）を使う場合、rootで起動する必要がある。

```
/home/admin/anaconda3/bin/jupyter notebook --allow-root > /root/jupyter.log 2>&1 &
```

> rootで起動しているため、pythonのシェル実行コマンドでシステムファイルを
消されてしまう危険がある。だから普通はやらない。

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

> 自動起動の設定をする前に、jupyter notebookの起動コマンドを、start_jupyter.shとして保存


## TensorFlow と Keras のインストール

TensorFlowのモジュール名は、その時のバージョンにより適宜変更する。
```
export TF_BINARY_URL=https://storage.googleapis.com/tensorflow/linux/cpu/tensorflow-1.0.1-cp36-cp36m-linux_x86_64.whl
pip install --ignore-installed --upgrade $TF_BINARY_URL
pip install keras
```

## GitBucket のインストールと起動

> ドキュメント管理にGitHubを使いたかったが、プライベートリポジトリの作成は有料であるためGitHubクローンを使用

GitBucket（gitbucket.war）を入手する。

```
wget https://github.com/gitbucket/gitbucket/releases/download/4.18.0/gitbucket.war
```

### java8をインストールし、GitBucketを起動する

```
yum search openjdk
sudo yum install java-1.8.0-openjdk.x86_64

java -jar gitbucket.war
```
ポート番号はデフォルトで8080。<br>
変更したい場合は`--port=9090`などとオプションで指定する。
