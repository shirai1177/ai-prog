# Python インストール

OSはCentOS7

## Anaconda3 (4.x.x) の入手とインストール

https://www.anaconda.com/download/ から対応プラットフォーム版を入手する。


## jupyter notebook の設定

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
```
export TF_BINARY_URL=https://storage.googleapis.com/tensorflow/linux/cpu/tensorflow-1.0.1-cp36-cp36m-linux_x86_64.whl
pip install --ignore-installed --upgrade $TF_BINARY_URL
pip install keras

conda install nomkl
```
