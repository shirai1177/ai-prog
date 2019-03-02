# Tensorflow + jupyter + keras + scikit-learn コンテナ構築

Dockerのインストールは終わっている前提。 

## tensorflow+jupyterの公式コンテナの利用

Tensorflow公式コンテナに必要なソフトウエアをインストールする。

```
[host]$ docker run -it -p 443:443 tensorflow/tensorflow:nightly-py3-jupyter
# pip install keras
# pip install scikit-learn
# apt-get -y install vim
# cd
# mkdir aihome
# jupyter notebook --generate-config
# cd .jupyter
# openssl req -x509 -nodes -days 365 -newkey rsa:1024 -keyout mykey.key -out mycert.pem
# vi jupyter_notebook_config.py
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

コンテナから抜けた後、 docker container commitでイメージを作る。

