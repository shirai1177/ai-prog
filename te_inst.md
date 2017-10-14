# Python インストール

OSはCentOS7。GCPのmicroインスタンスの場合、SWAPを有効にする

```
sudo su -
dd if=/dev/zero of=/swapfile bs=1M count=1024
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

## Anaconda3 (5.x.x) の入手とインストール

https://www.anaconda.com/download/ から対応プラットフォーム版を入手する。

```
sudo yum -y install bzip2
bash Anaconda3-5.0.0.1-Linux-x86_64.sh
```

## jupyter notebook の設定

作業用フォルダを aihome とする場合。<br>
httpsで起動するため、opensslで鍵の生成と登録を行う。<br>

```
mkdir aihome
jupyter notebook --generate-config
cd .jupyter
openssl req -x509 -nodes -days 365 -newkey rsa:1024 -keyout mykey.key -out mycert.pem
vi jupyter_notebook_config.py
c.NotebookApp.ip = '*'
c.NotebookApp.notebook_dir = '/home/admin/aihome'
c.NotebookApp.open_browser = False
c.NotebookApp.token = 'Study123'
c.NotebookApp.certfile = '/home/admin/.jupyter/mycert.pem'
c.NotebookApp.keyfile = '/home/admin/.jupyter/mykey.key'
c.NotebookApp.port = 443
```

## jupyter notebook の起動
```
cd ~/.jupyter
sudo /home/admin/anaconda3/bin/jupyter notebook --allow-root
```

## TensorFlow と Keras のインストール

TensorFlowのモジュール名は、その時のバージョンにより適宜変更する。
```
export TF_BINARY_URL=https://storage.googleapis.com/tensorflow/linux/cpu/tensorflow-1.0.1-cp36-cp36m-linux_x86_64.whl
pip install --ignore-installed --upgrade $TF_BINARY_URL
pip install keras
```

# GitBucket のインストールと起動

GitBucketをあらかじめ入手する。  
gitbucket.war


## java8をインストールし、GitBucketを起動する

```
yum search openjdk
sudo yum install java-1.8.0-openjdk.x86_64

java -jar gitbucket.war
```
