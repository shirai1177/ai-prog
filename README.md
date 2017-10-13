## NumPyライブラリ

機械学習・ディープラーニングの実装では、配列や行列の計算が多く登場します。
NumPy（ナムパイ）ライブラリの配列クラスには便利なメソッドが多く用意されており、このハンズオンにおいてもそれらのメソッドを利用します。

NumPyを使うには次のようにimportします。

```python
import numpy as np
```

<div style="page-break-before:always"></div>


### NumPy配列の作成

NumPy配列を生成するにはいくつかの方法があります。

```python
import numpy as np

a = np.array([1, 2, 3])
print(a)

b = np.arange(5)
print(b)
```

実行結果は次のように表示されます。

```
[1 2 3]
[0 1 2 3 4]
```

配列の要素の範囲を指定して生成する場合は np.arange メソッドを使います。

```python
a = np.arange(5)
```

上記の場合 [0 1 2 3 4] というNumPy配列が生成されます。

またnp.arrayメソッドを使えば通常の配列をNumPy配列に変換することができます。

```python
b = np.array([1, 2, 3])
```

上記の場合 [1 2 3] というNumPy配列が生成されます。

<div style="page-break-before:always"></div>


### 配列の操作

配列の操作について見てみましょう。

```python
import numpy as np

a = np.array([1, 2, 3, 4, 5, 6, 7, 8, 9])
print(a)
print(a.shape)
print(a[1])
print(a[1:5])
print(a[5:])
```

実行結果は次のように表示されます。

```
[1 2 3 4 5 6 7 8 9]
(9,)
2
[2 3 4 5]
[6 7 8 9]
```

NumPy配列は shape プロパティにアクセスすることで、配列の次元ごとの要素数をタプルで取得できます。上記の場合は (9,) と表示されます。

> NumPy配列でなく通常のリストでも上記のようにアクセスできます。

<div style="page-break-before:always"></div>


NumPy配列は次元数を変更することもできます。

```python
import numpy as np

a = np.array([1, 2, 3, 4])
b = a.reshape(2, 2)
print(b)
print(b.shape)
print(b[1])
print(b[1,1])
print(b[0:1])
print(b[0:1,1])
print(b[:,1])

c = b.flatten()
print(c)
```

実行結果は次のように表示されます。

```
[[1 2]
 [3 4]]
(2, 2)
[3 4]
4
[[1 2]]
[2]
[2 4]
[1 2 3 4]
```

NumPy配列は reshape メソッドによって配列の次元数を変更できます。

```python
b = a.reshape(2, 2)
```

また多次元配列は次のようにカンマ区切りで要素にアクセスできます。

```python
print(b[1,1])
```

2次元目の要素番号だけを指定することもできます。

```python
print(b[:,2])
```

N次元配列を1次元に戻すには、flatten メソッドを使います。

```python
c = b.flatten()
```

<div style="page-break-before:always"></div>


### NumPy配列の演算

NumPy配列の演算を見てみましょう。

```python
import numpy as np

a = np.array([1, 2])
b = np.array([3, 4])

print(a + b)
print(a * b)

c = a.dot(b)
print(c)
```

実行結果は次のように表示されるでしょう。

```
[4 6]
[3 8]
11
```

NumPy配列の各要素ごとに加算や、積算することができます。

```python
print(a + b)  #=> [5 7 9]
print(a * b)  #=> [ 4 10 18]
```

また2つの配列（ベクトル）の内積を求めるにはdotメソッドを使います。

```python
c = a.dot(b)
print(c) #=> 11
```

> dot積は (1 \* 3) + (2 \* 4) = 11 となります。
> 数学では1次元配列をベクトル、2次元配列を行列、3次元以上をテンソルまたは多次元配列とよびます。

<div style="page-break-before:always"></div>


### ブロードキャスト

NumPy配列は次元数の異なるオペランドを演算することができます。このような仕組みをブロードキャストと呼びます。

```python
import numpy as np

a = np.array([1, 2])
b = a + 1
print(b)

c = np.array([[1, 2], [3, 4]])
d = c + a
print(d)
```

実行結果は次のように表示されるでしょう。

```
[2 3]
[[2 4]
 [4 6]]
```

<img src="img/01_08.png" width="400px">

<img src="img/01_09.png" width="400px">
