## 学習フェーズ

```
from keras.datasets import mnist
from keras.models import Sequential
from keras.layers import Dense, Activation, Dropout, Flatten
from keras.layers import Conv2D, MaxPooling2D
from keras.utils import to_categorical

(x_train, y_train), (x_test, y_test) = mnist.load_data()

x_train = x_train.reshape(60000, 28, 28, 1)
x_test = x_test.reshape(10000, 28, 28, 1)
x_train = x_train.astype('float32') / 255
x_test = x_test.astype('float32') / 255

y_train = to_categorical(y_train, 10)
y_test = to_categorical(y_test, 10)

model = Sequential()
model.add(Conv2D(32, kernel_size=(3, 3), input_shape=(28, 28, 1)))
model.add(Activation("relu"))
model.add(MaxPooling2D(pool_size=(2, 2)))
model.add(Conv2D(64, kernel_size=(3, 3)))
model.add(Activation("relu"))
model.add(MaxPooling2D(pool_size=(2, 2)))
model.add(Dropout(0.25))
model.add(Flatten())
model.add(Dense(128))
model.add(Activation("relu"))
model.add(Dropout(0.5))
model.add(Dense(10))
model.add(Activation("softmax"))
model.compile(optimizer="Adam", loss="categorical_crossentropy", metrics=["accuracy"])
model.summary()
model.fit(x_train, y_train, batch_size=32, epochs=1)

score = model.evaluate(x_test, y_test, verbose=0)
print("テストデータによる汎化性能の評価 : ", score[1])

# モデルの保存
model.save("cnn-mnist.model")

```

## 予測フェーズ

```
import os,sys
import numpy as np
from PIL import Image
import matplotlib.pyplot as plt
from keras.models import load_model

model = load_model("../teacher/cnn-mnist.model")  # 学習済みモデルのロード
        
filename = [ "num5.png", "num8.png" ]

for name in filename:
    # グレースケールとして画像の読み込み
    img = Image.open(name).convert("L")

    # 画像を28x28に変換
    img = img.resize((28, 28), Image.ANTIALIAS)
    plt.imshow(np.array(img))
    plt.show()

    # フロート型の行列に変換
    img = np.array(img, dtype=np.float32)
    # 黒0~255白の画像データをMNISTのデータと同じ白0~1黒に変える
    img = 1 - np.array(img / 255)

    handwrite = img.reshape(1, 28, 28, 1)
    p = model.predict(handwrite)
    np.set_printoptions(suppress=True)
    print(p)
    print("予測: %d, 確率: %2.1f%%" % (np.argmax(p), p[0][np.argmax(p)]*100)) # 予測値の表示
```
