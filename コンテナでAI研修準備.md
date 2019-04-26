# インフラ関連

## 仮想マシンの準備

CentOS7系を準備する。<br>
受講者に合わせてCPUとメモリを調整する。ディスクは20MB程度が望ましい。<br>
また必要に応じて静的IPを割り当てる。

## 初期セットアップスクリプト

以下の初期セットアップスクリプトをrootで実行する。（ユーザデータとして実行でもよい）<br>
このスクリプトはGCPでの実行例

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

# yum install -y yum-utils device-mapper-persistent-data lvm2
yum install -y yum-utils
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
yum install -y docker-ce-18.09.4-3.el7 docker-ce-cli-18.09.4-3.el7 containerd.io
systemctl start docker
systemctl enable docker
usermod -aG docker cloudedu_teacher
usermod -aG docker admin

docker run -d -e TZ=Asia/Tokyo --name aiedu -p 443:443 metroedu/tensor-edu
docker run -d -e TZ=Asia/Tokyo --name aidoc -p 8080:8080 metroedu/ai-doc

mv /etc/selinux/config /etc/selinux/config.org
sed 's/SELINUX=enforcing/SELINUX=disabled/' /etc/selinux/config.org > /etc/selinux/config

echo "0 18 * * * /sbin/shutdown -h now" >> /var/spool/cron/root
chmod 600 /var/spool/cron/root
```

※GCPの設定では、一度起動した後はGCEのメタデータからstartup-scriptの項目を削除する。（再起動のたびに実行されないように）

## 受講者ディレクトリの作成

jupyter notebookのホームに、受講者毎のディレクトリを作成する。

```
docker exec -it aiedu bash
cd
vi make_study.sh
　：
./make_study.sh
```

## SE-Linuxの設定反映のため、マシン再起動

SE-Linuxの設定反映（Disabled）のため、仮想マシンを再起動する。
コンテナは手動で停止する。

```
docker stop aiedu aidoc
shutdown -h now
```

このときCNNモデル作成のためCPU数を増やすスケールアップをしてもよい。

## CNNモデルの作成

画像認識の演習用に、CNNの学習モデルを作成する。

