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


### CartPoleを動かすコード

```
import gym
import numpy as np
from matplotlib import animation
from matplotlib import pyplot as plt
%matplotlib nbagg
from keras.models import load_model
from keras import backend as K
import tensorflow as tf

def huberloss(y_true, y_pred):
    err = y_true - y_pred
    cond = K.abs(err) < 1.0
    L2 = 0.5 * K.square(err)
    L1 = (K.abs(err) - 0.5)
    loss = tf.where(cond, L2, L1)
    return K.mean(loss)

# model = load_model('./dqn180.model', custom_objects={"huberloss": huberloss})
model = load_model('./dqn.model')

env = gym.make('CartPole-v0')

observation = env.reset()
cum_reward = 0
frames = []
for t in range(200):
    # Render into buffer. 
    frames.append(env.render(mode = 'rgb_array'))
    # action = env.action_space.sample()
    action = np.argmax(model.predict(np.reshape(observation, [1, 4]))[0])
    observation, reward, done, info = env.step(action)
    if done:
        if t < 199:
            for i in range(40):
                frames.append(env.render(mode = 'rgb_array'))
                action = 1 - action
                env.step(action)
            break

fig = plt.gcf()
patch = plt.imshow(frames[0])
plt.axis('off')

def animate(i):
    patch.set_data(frames[i])

anim = animation.FuncAnimation(fig, animate, frames = len(frames), interval=5)
# anim.save("CartPole-v0.gif", writer = 'imagemagick')

```

DDQNで学習するコード

```
import gym
import numpy as np
import time
from keras.models import Sequential
from keras.layers import Dense
from keras.optimizers import Adam
from keras.utils import plot_model
from collections import deque
from gym import wrappers  # gymの画像保存
from keras import backend as K
import tensorflow as tf

def huberloss(y_true, y_pred):
    err = y_true - y_pred
    cond = K.abs(err) < 1.0
    L2 = 0.5 * K.square(err)
    L1 = (K.abs(err) - 0.5)
    loss = tf.where(cond, L2, L1)  # Keras does not cover where function in tensorflow :-(
    return K.mean(loss)


class QNetwork:
    def __init__(self, learning_rate=0.01, state_size=4, action_size=2, hidden_size=10):
        self.model = Sequential()
        self.model.add(Dense(hidden_size, activation='relu', input_dim=state_size))
        self.model.add(Dense(hidden_size, activation='relu'))
        self.model.add(Dense(action_size, activation='linear'))
        self.optimizer = Adam(lr=learning_rate)
        self.model.compile(loss='mean_squared_error', optimizer='adam')
        # self.model.compile(loss=huberloss, optimizer=self.optimizer)

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

            if not (next_state_b == np.zeros(state_b.shape)).all(axis=1):
                retmainQs = self.model.predict(next_state_b)[0]
                next_action = np.argmax(retmainQs)  # 最大の報酬を返す行動を選択する
                target = reward_b + gamma * targetQN.model.predict(next_state_b)[0][next_action]

            targets[i] = self.model.predict(state_b)    # Qネットワークの出力
            targets[i][action_b] = target               # 教師信号

        self.model.fit(inputs, targets, epochs=5, verbose=0)  # epochsは訓練データの反復回数、verbose=0は表示なしの設定

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
    def get_action(self, state, episode, mainQN):   # [C]ｔ＋１での行動を返す
        # 徐々に最適行動のみをとる、ε-greedy法
        epsilon = 0.001 + 0.9 / (1.0+episode)

        if epsilon <= np.random.uniform(0, 1):
            retTargetQs = mainQN.model.predict(state)[0]
            action = np.argmax(retTargetQs)  # 最大の報酬を返す行動を選択する
        else:
            action = np.random.choice([0, 1])  # ランダムに行動する

        return action

DQN_MODE = 0    # 1がDQN、0がDDQNです
LENDER_MODE = 1 # 0は学習後も描画なし、1は学習終了後に描画する

env = gym.make('CartPole-v0')
num_episodes = 500  # 総試行回数
max_number_of_steps = 200  # 1試行のstep数
goal_average_reward = 195  # この報酬を超えると学習終了
num_consecutive_iterations = 10  # 学習完了評価の平均計算を行う試行回数
total_reward_vec = np.zeros(num_consecutive_iterations)  # 各試行の報酬を格納
gamma = 0.99    # 割引係数
islearned = 0  # 学習が終わったフラグ
isrender = 0  # 描画フラグ

hidden_size = 16               # 隠れ層のニューロンの数
# learning_rate = 0.00001         # Q-networkの学習係数
learning_rate = 0.001         # Q-networkの学習係数
memory_size = 10000            # バッファーメモリの大きさ
batch_size = 32                # Q-networkを更新するバッチの大記載

mainQN = QNetwork(hidden_size=hidden_size, learning_rate=learning_rate)     # メインのQネットワーク
targetQN = QNetwork(hidden_size=hidden_size, learning_rate=learning_rate)   # 価値を計算するQネットワーク
# plot_model(mainQN.model, to_file='Qnetwork.png', show_shapes=True)        # Qネットワークの可視化
memory = Memory(max_size=memory_size)
actor = Actor()

for episode in range(num_episodes):
    env.reset()
    state, reward, done, _ = env.step(env.action_space.sample())
    state = np.reshape(state, [1, 4])   # 1行4列の行列に変換
    episode_reward = 0

    targetQN.model.set_weights(mainQN.model.get_weights())

    for t in range(max_number_of_steps + 1):
        if (islearned == 1) and LENDER_MODE:  # 学習終了したらcartPoleを描画する
            env.render()
            time.sleep(0.1)

        action = actor.get_action(state, episode, mainQN)   # 時刻tでの行動を決定する
        next_state, reward, done, info = env.step(action)   # 行動a_tの実行による、s_{t+1}, _R{t}を計算する
        next_state = np.reshape(next_state, [1, 4])     # list型のstateを、1行4列の行列に変換

        # 報酬を設定し、与える
        if done:
            next_state = np.zeros(state.shape)  # 次の状態s_{t+1}はない
            if t < 195:
                reward = -1  # 報酬クリッピング、報酬は1, 0, -1に固定
            else:
                reward = 1  # 立ったまま195step超えて終了時は報酬
        else:
            reward = 0  # 各ステップで立ってたら報酬追加（はじめからrewardに1が入っているが、明示的に表す）

        episode_reward += 1 # reward  # 合計報酬を更新

        memory.add((state, action, reward, next_state))     # メモリの更新する
        state = next_state  # 状態更新


        # Qネットワークの重みを学習・更新する replay
        if (memory.len() > batch_size) and not islearned:
            mainQN.replay(memory, batch_size, gamma, targetQN)

        if DQN_MODE:
            targetQN.model.set_weights(mainQN.model.get_weights())

        # 1施行終了時の処理
        if done:
            total_reward_vec = np.hstack((total_reward_vec[1:], episode_reward))  # 報酬を記録
            print(total_reward_vec)
            print('%d Episode finished after %d time steps / mean %f' % (episode, t + 1, total_reward_vec.mean()))
            break

    # 複数施行の平均報酬で終了を判断
    if total_reward_vec.mean() >= goal_average_reward:
        print('Episode %d train agent successfuly!' % episode)
        islearned = 1
        mainQN.save()
        if isrender == 0:   # 学習済みフラグを更新
            isrender = 1
            # env = wrappers.Monitor(env, './movie/cartpoleDDQN')  # 動画保存する場合
```

## 研修で使う

CartPole-v0を動かすコード

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

for t in range(200):
    frames.append(env.render(mode = 'rgb_array'))
    #action = env.action_space.sample()
    action = np.argmax(model.predict(np.reshape(state, [1, 4]))[0])
    state, reward, done, info = env.step(action)
    if done:
        if t < 199:
            for i in range(40):
                frames.append(env.render(mode = 'rgb_array'))
                action = 1 - action
                env.step(action)
            break

fig = plt.gcf()
patch = plt.imshow(frames[0])
plt.axis('off')

def animate(i):
    patch.set_data(frames[i])

anim = animation.FuncAnimation(fig, animate, frames = len(frames), interval=5)
# anim.save("CartPole-v0.gif", writer = 'imagemagick')
```

### DQNによる学習プログラム

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

#            if not (next_state_b == np.zeros(state_b.shape)).all(axis=1):
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
    def get_action(self, state, episode, mainQN):   # [C]ｔ＋１での行動を返す
        # 徐々に最適行動のみをとる、ε-greedy法
        epsilon = 0.001 + 0.9 / (1.0 + episode)

        if epsilon <= np.random.uniform(0, 1):
            retTargetQs = mainQN.model.predict(state)[0]
            action = np.argmax(retTargetQs)  # 最大の報酬を返す行動を選択する
        else:
            action = np.random.choice([0, 1])  # ランダムに行動する

        return action
#
# main処理
#
DQN_MODE = 0    # 1がDQN、0がDDQN

env = gym.make('CartPole-v0')
num_episodes = 20  # 総試行回数
max_number_of_steps = 200  # 1試行のstep数
goal_average_reward = 195  # この報酬を超えると学習終了
num_consecutive_iterations = 5  # 学習完了評価の平均計算を行う試行回数
total_reward_vec = np.zeros(num_consecutive_iterations)  # 各試行の報酬を格納
gamma = 0.99    # 割引係数


hidden_size = 16               # 隠れ層のニューロンの数
learning_rate = 0.001          # Adamの学習係数 def.0.001
memory_size = 10000            # エクスペリエンスキューの大きさ
batch_size = 32                # DNNのバッチサイズ

mainQN = QNetwork(hidden_size=hidden_size, learning_rate=learning_rate)     # メインのQネットワーク
targetQN = QNetwork(hidden_size=hidden_size, learning_rate=learning_rate)   # 価値を計算するQネットワーク
memory = Memory(max_size=memory_size)
actor = Actor()

for episode in range(num_episodes):
    env.reset()
    state, reward, done, _ = env.step(env.action_space.sample())
    state = np.reshape(state, [1, 4])   # 1行4列の行列に変換
    episode_reward = 0

    targetQN.model.set_weights(mainQN.model.get_weights())

    for t in range(max_number_of_steps + 1):
        action = actor.get_action(state, episode, mainQN)   # 時刻tでの行動を決定する
        next_state, reward, done, info = env.step(action)   # 行動a_tの実行による、s_{t+1}, _R{t}を計算する
        next_state = np.reshape(next_state, [1, 4])     # list型のstateを、1行4列の行列に変換

        # 報酬の設定
        # reward -= abs(next_state[0][2]) * 2  # 棒の傾きが大きいと報酬を減らす
        reward = 0

        if done:
            next_state = np.zeros(state.shape)  # 次の状態s_{t+1}はない
            reward = (t - 99) * 0.01            # 1エピソード終了時に進んだステップにより -1 ～ 1 の報酬

        episode_reward += 1 # reward  # 合計報酬を更新

        memory.add((state, action, reward, next_state))     # エクスペリエンスを登録する
        state = next_state  # 状態更新

        # Qネットワークの重みを学習・更新する
        if (memory.len() > batch_size):
            mainQN.replay(memory, batch_size, gamma, targetQN)

        if DQN_MODE:
            targetQN.model.set_weights(mainQN.model.get_weights())

        # 1エピソード終了時の処理
        if done:
            total_reward_vec = np.hstack((total_reward_vec[1:], episode_reward))  # 報酬を記録
            print(total_reward_vec)
            print('%d Episode finished after %d time steps / mean %f' % (episode, t + 1, total_reward_vec.mean()))
            break

    # 複数施行の平均報酬で終了を判断
    if total_reward_vec.mean() >= goal_average_reward:
        print('Episode %d train agent successfuly!' % episode)
        break

# 学習結果をモデルとして保存
print('Saved model.')
mainQN.save()
```
