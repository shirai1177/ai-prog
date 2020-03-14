## 構築手順

```
apt-get update
apt-get upgrade -y
apt-get install -y python3-opengl xvfb cmake zlib1g-dev swig imagemagick mpich
# apt-get install -y ffmpeg

pip install mpi4py opencv-python
pip install gym[atari]

nohup xvfb-run -s "-screen 0 1400x900x24" jupyter notebook &
```

### 参考にしたサイト

https://qiita.com/bpzAkiyama/items/94ebd897c37ba4176492

https://qiita.com/sugulu/items/bc7c70e6658f204f85f9

***

### CartPoleの問題

CartPoleの実行コード

```
import gym
import numpy as np
from matplotlib import animation
from matplotlib import pyplot as plt
%matplotlib nbagg
from keras.models import load_model

model = load_model('./dqn.model')

env = gym.make('CartPole-v0')
state = env.reset()
frames = []

for t in range(300):
    frames.append(env.render(mode = 'rgb_array'))
    # action = env.action_space.sample()       # ランダムに選択
    # action = 0 if state[2] < 0 else 1        # 棒の傾きに合わせて移動
    action = np.argmax(model.predict(np.reshape(state, [1, 4]))[0])    # AIが選択
    state, reward, done, info = env.step(action)
    if done:
        if t < 199:
            for i in range(40):
                frames.append(env.render(mode = 'rgb_array'))
                action = 1 - action
                state, reward, done, info = env.step(action)
        break

fig = plt.gcf()
patch = plt.imshow(frames[0])
plt.axis('off')

def animate(i):
    patch.set_data(frames[i])

anim = animation.FuncAnimation(fig, animate, frames = len(frames), interval=10)
# anim.save("CartPole-v0.gif", writer = 'imagemagick')
```

CartPoleをDDQNで学習するコード

```
import gym
import numpy as np
import time
from keras.models import Sequential
from keras.layers import Dense
from keras.optimizers import Adam
from collections import deque


class QNetwork:
    def __init__(self, learning_rate=0.01, state_size=4, action_size=2, hidden_size=10):
        self.model = Sequential()
        self.model.add(Dense(hidden_size, activation='relu', input_dim=state_size))
        self.model.add(Dense(hidden_size, activation='relu'))
        self.model.add(Dense(action_size, activation='linear'))
        self.optimizer = Adam(lr=learning_rate)
        self.model.compile(loss='mse', optimizer=self.optimizer)

    def save(self, path='./dqn.model'):
        self.model.save(path)
        
    # 重みの学習
    def replay(self, memory, batch_size, gamma, targetQN):
        inputs = np.zeros((batch_size, 4))
        targets = np.zeros((batch_size, 2))
        mini_batch = memory.sample(batch_size)

        for i, (state_b, action_b, reward_b, next_state_b) in enumerate(mini_batch):
            inputs[i:i + 1] = state_b
            target = reward_b

            if next_state_b[0][0]:
                retmainQs = self.model.predict(next_state_b)[0]
                next_action = np.argmax(retmainQs)
                # ベルマン方程式で未来の報酬値を推定し教師データとする
                target = reward_b + gamma * targetQN.model.predict(next_state_b)[0][next_action]

            targets[i] = self.model.predict(state_b)
            targets[i][action_b] = target

        self.model.fit(inputs, targets, epochs=3, verbose=0)

class Memory:
    def __init__(self, max_size=1000):
        self.buffer = deque(maxlen=max_size)

    def add(self, experience):
        self.buffer.append(experience)

    def sample(self, batch_size):
        idx = np.random.choice(np.arange(len(self.buffer)), size=batch_size, replace=False)
        return [self.buffer[ii] for ii in idx]

    def len(self):
        return len(self.buffer)

class Actor:
    def get_action(self, state, episode, mainQN):
        # 徐々に最適行動のみをとる、ε-greedy法
        epsilon = 0.9 / (1.0 + episode)

        if epsilon <= np.random.uniform(0, 1):
            retTargetQs = mainQN.model.predict(state)[0]
            action = np.argmax(retTargetQs)  # 最大の報酬を返す行動を選択する
        else:
            action = np.random.choice([0, 1])

        return action
#
# main処理
#
env = gym.make('CartPole-v0')
num_episodes = 100  # 総試行回数
max_number_of_steps = 200  # 1試行のstep数
goal_average_reward = 195  # この報酬を超えると学習終了
num_consecutive_iterations = 5  # 学習完了評価の平均計算を行う試行回数
total_reward_vec = np.zeros(num_consecutive_iterations)  # 各試行の報酬を格納
gamma = 0.99    # 割引係数

hidden_size = 16               # 隠れ層のニューロンの数
learning_rate = 0.001          # Adamの学習係数 def.0.001
memory_size = 10000            # エクスペリエンスメモリーの大きさ
batch_size = 32                # 学習に使うデータ数

mainQN = QNetwork(hidden_size=hidden_size, learning_rate=learning_rate)     # メインのネットワーク
targetQN = QNetwork(hidden_size=hidden_size, learning_rate=learning_rate)   # 価値を計算するネットワーク
memory = Memory(max_size=memory_size)
actor = Actor()

for episode in range(num_episodes):
    env.reset()
    state, reward, done, _ = env.step(env.action_space.sample())
    state = np.reshape(state, [1, 4])
    episode_reward = 0

    targetQN.model.set_weights(mainQN.model.get_weights())

    for t in range(max_number_of_steps + 1):
        action = actor.get_action(state, episode, mainQN)
        next_state, reward, done, info = env.step(action)
        next_state = np.reshape(next_state, [1, 4])

        # 報酬の設定
        reward -= abs(next_state[0][2]) * 3  # 棒の傾きが大きいと報酬を減らす
        # reward = 0                             # 通常の行動は評価しない

        if done:
            next_state = np.zeros(state.shape)  # 次の状態をクリア
            reward = (t - 99) * 0.01            # 1エピソード終了時に進んだステップにより -1 ～ 1 の報酬

        episode_reward += 1

        memory.add((state, action, reward, next_state))     # エクスペリエンスを登録する
        state = next_state  # 状態更新

        # 学習
        if (memory.len() > batch_size):
            mainQN.replay(memory, batch_size, gamma, targetQN)

        # 1エピソード終了時の処理
        if done:
            total_reward_vec = np.hstack((total_reward_vec[1:], episode_reward))
            print(total_reward_vec)
            print('%d Episode finished after %d time steps / mean %f' % (episode, t + 1, total_reward_vec.mean()))
            if (episode + 1) % 10 == 0:
                print('Saved.')
                mainQN.save()
            break

    # 平均報酬で終了を判断
    if total_reward_vec.mean() >= goal_average_reward:
        print('Episode %d train agent successfuly!' % episode)
        break

# 学習結果をモデルとして保存
print('Saved model.')
mainQN.save()
```

***

### MountainCar の問題

MountainCarの実行コード

```
import gym
import numpy as np
from matplotlib import animation
from matplotlib import pyplot as plt
%matplotlib nbagg
from keras.models import load_model

model = load_model('./mountain.model')

env = gym.make('MountainCar-v0')
state = env.reset()
frames = []

for t in range(200):
    frames.append(env.render(mode = 'rgb_array'))
    # action = env.action_space.sample()       # ランダムに選択
    action = np.argmax(model.predict(np.reshape(state, [1, 2]))[0])    # AIが選択
    state, reward, done, info = env.step(action)
    if done:
        break

fig = plt.gcf()
patch = plt.imshow(frames[0])
plt.axis('off')

def animate(i):
    patch.set_data(frames[i])

anim = animation.FuncAnimation(fig, animate, frames = len(frames), interval=50)
```

MountainCarの学習

```
import gym
import numpy as np
import time
from keras.models import Sequential
from keras.layers import Dense
from keras.optimizers import Adam
from collections import deque


class QNetwork:
    def __init__(self, learning_rate=0.01, state_size=2, action_size=3, hidden_size=10):
        self.model = Sequential()
        self.model.add(Dense(hidden_size, activation='relu', input_dim=state_size))
        self.model.add(Dense(hidden_size, activation='relu'))
        self.model.add(Dense(action_size, activation='linear'))
        self.optimizer = Adam(lr=learning_rate)
        self.model.compile(loss='mse', optimizer=self.optimizer)

    def save(self, path='./mountain.model'):
        self.model.save(path)
        
    # 重みの学習
    def replay(self, memory, batch_size, gamma, targetQN):
        inputs = np.zeros((batch_size, 2))
        targets = np.zeros((batch_size, 3))
        mini_batch = memory.sample(batch_size)

        for i, (state_b, action_b, reward_b, next_state_b) in enumerate(mini_batch):
            inputs[i:i + 1] = state_b
            target = reward_b

            if next_state_b[0][0]:
                retmainQs = self.model.predict(next_state_b)[0]
                next_action = np.argmax(retmainQs)
                # ベルマン方程式で未来の報酬値を推定し教師データとする
                target = reward_b + gamma * targetQN.model.predict(next_state_b)[0][next_action]

            targets[i] = self.model.predict(state_b)
            targets[i][action_b] = target

        self.model.fit(inputs, targets, epochs=3, verbose=0)

class Memory:
    def __init__(self, max_size=1000):
        self.buffer = deque(maxlen=max_size)

    def add(self, experience):
        self.buffer.append(experience)

    def sample(self, batch_size):
        idx = np.random.choice(np.arange(len(self.buffer)), size=batch_size, replace=False)
        return [self.buffer[ii] for ii in idx]

    def len(self):
        return len(self.buffer)

class Actor:
    def get_action(self, state, episode, mainQN):
        # 徐々に最適行動のみをとる、ε-greedy法
        epsilon = 0.9 * (1 / (1 + episode))

        if epsilon <= np.random.uniform(0, 1):
            retTargetQs = mainQN.model.predict(state)[0]
            action = np.argmax(retTargetQs)  # 最大の報酬を返す行動を選択する
        else:
            action = np.random.choice([0, 1, 2])  # ランダムに行動する

        return action
#
# main処理
#

env = gym.make('MountainCar-v0')
num_episodes = 100  # 総試行回数
max_number_of_steps = 200  # 1試行のstep数
goal_reward = 160  # 直近の報酬の平均がこれを下回ると学習終了
num_consecutive_iterations = 5  # 学習完了評価の平均計算を行う試行回数
total_reward_vec = np.zeros(num_consecutive_iterations)  # 各試行の報酬を格納
gamma = 0.99    # 割引係数


hidden_size = 16               # 隠れ層のニューロンの数
learning_rate = 0.001          # Adamの学習係数 def.0.001
memory_size = 10000            # エクスペリエンスメモリーの大きさ
batch_size = 32

mainQN = QNetwork(hidden_size=hidden_size, learning_rate=learning_rate)
targetQN = QNetwork(hidden_size=hidden_size, learning_rate=learning_rate)
memory = Memory(max_size=memory_size)
actor = Actor()

for episode in range(num_episodes):
    env.reset()
    state, reward, done, _ = env.step(env.action_space.sample())
    state = np.reshape(state, [1, 2])   # 1行N列の行列に変換
    episode_reward = 0

    targetQN.model.set_weights(mainQN.model.get_weights())

    for t in range(max_number_of_steps + 1):
        action = actor.get_action(state, episode, mainQN)
        next_state, reward, done, info = env.step(action)
        next_state = np.reshape(next_state, [1, 2])

        # 報酬の設定
        reward = abs(next_state[0][1]) * 2   # 速度を報酬にする

        if done:
            next_state = np.zeros(state.shape)
            # reward = 2 - (t * 0.01)
            if t+1 < 199:
                reward = 10  # ゴールに到達したら大きな報酬

        episode_reward += 1

        memory.add((state, action, reward, next_state))     # エクスペリエンスを登録する
        state = next_state  # 状態更新

        # Qネットワークの重みを学習・更新する
        if (memory.len() > batch_size):
            mainQN.replay(memory, batch_size, gamma, targetQN)

        # 1エピソード終了時の処理
        if done:
            total_reward_vec = np.hstack((total_reward_vec[1:], episode_reward))
            print(total_reward_vec)
            print('%d Episode finished after %d time steps / mean %f' % (episode, t + 1, total_reward_vec.mean()))
            mainQN.save() # エピソード毎に保存
            break

    # 複数施行の平均報酬で終了を判断
    if total_reward_vec[0] and total_reward_vec.mean() < goal_reward:
        print('Episode %d train agent successfuly!' % episode)
        break

# 学習結果をモデルとして保存
print('Saved model.')
mainQN.save()
```
