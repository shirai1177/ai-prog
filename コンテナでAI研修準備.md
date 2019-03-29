# インフラ関連

## 仮想マシンの準備

CentOS7系を準備する。<br>
受講者に合わせてCPUとメモリを調整する。また必要に応じて静的IPを割り当てる。

## 初期セットアップスクリプト

以下の初期セットアップスクリプトをrootで実行する。（ユーザデータとして実行でもよい）

```
#! /bin/bash
dd if=/dev/zero of=/swapfile bs=1M count=2048
chmod 600 /swapfile
mkswap /swapfile
swapon /swapfile
free -m
cp -P /etc/fstab /etc/fstab.org
echo "/swapfile swap swap defaults 0 0" >> /etc/fstab
timedatectl set-timezone Asia/Tokyo

yum install -y yum-utils device-mapper-persistent-data lvm2
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
yum install -y docker-ce-18.09.0-3.el7
systemctl start docker
systemctl enable docker
usermod -aG docker cloudedu_teacher
usermod -aG docker admin

docker run -d -e TZ=Asia/Tokyo -name aiedu -p 443:443 metroedu/tensor-edu
docker run -d -e TZ=Asia/Tokyo -p 8080:8080 metroedu/ai-doc
```

## 受講者ディレクトリの作成

jupyter notebookのホームに、受講者毎のディレクトリを作成する。

```
docker exec -it -name aiedu bash
cd
vi make_study.sh
./make_study.sh
```


