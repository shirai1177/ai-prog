## 構築手順

```
apt-get update
apt-get upgrade -y
apt-get install -y python3-opengl xvfb cmake zlib1g-dev swig imagemagick mpich

pip install mpi4py opencv-python

git clone https://github.com/openai/gym
cd gym
pip install -e .

cd ~
nohup xvfb-run -s "-screen 0 1400x900x24" jupyter notebook &
```
