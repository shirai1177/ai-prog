## 構築手順

```
apt-get update
apt-get upgrade -y
apt-get install -y python3-opengl xvfb cmake zlib1g-dev swig imagemagick mpich

apt-get install -y ffmpeg


pip install mpi4py opencv-python

git clone https://github.com/openai/gym
cd gym
pip install -e .

cd ~
nohup xvfb-run -s "-screen 0 1400x900x24" jupyter notebook &
```

CartPoleを動かすコード

```
import gym
from matplotlib import animation
from matplotlib import pyplot as plt
%matplotlib nbagg

env = gym.make('CartPole-v0')

# Run a demo of the environment
observation = env.reset()
cum_reward = 0
frames = []
for t in range(5000):
    # Render into buffer. 
    frames.append(env.render(mode = 'rgb_array'))
    action = env.action_space.sample()
    observation, reward, done, info = env.step(action)
    if done:
        break
#env.render(close=True)

fig = plt.gcf()
patch = plt.imshow(frames[0])
plt.axis('off')

def animate(i):
    patch.set_data(frames[i])

anim = animation.FuncAnimation(fig, animate, frames = len(frames), interval=50)
#anim

#anim.save("CartPole-v0.gif", writer = 'imagemagick')
```
