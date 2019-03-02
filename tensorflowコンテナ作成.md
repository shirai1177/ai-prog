# Tensorflow + jupyter + keras + scikit-learn コンテナ構築

Dockerのインストールは終わっている前提。 

## tensorflow+jupyterコンテナ

```
host$ docker run -it -p 443:443 tensorflow/tensorflow:nightly-py3-jupyter
# cd
# pip install keras
# pip install scikit-learn
```

