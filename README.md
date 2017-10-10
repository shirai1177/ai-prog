## Python入門ハンズオン

それでは実際にPythonプログラムの概要を見てみましょう。

### Hello World

画面に"Hello World"を表示してみましょう。

```python
print("Hello World")
```

実行結果は次のように表示されるでしょう。

```
Hello World
```


### 変数とデータ型

Pythonの文字列、数値、実数、真偽値型、None型を確認してみましょう。Pythonは「動的型付き言語」です。変数の型は状況に応じて動的に決定されます。

```python
name = "John"
age = 20
height = 170.1
license = True # False
money = None

print(type(name))
print(type(age))
print(type(height))
print(type(license))
print(type(money))
```

実行結果は次のように表示されるでしょう。

```
<class 'str'>
<class 'int'>
<class 'float'>
<class 'bool'>
<class 'NoneType'>
```

type関数を使えばデータ型を確認できます。

> Pythonプログラムの中でコメントは # で表現します。

<div style="page-break-before:always"></div>


### if文

if文は次のようになります。論理積を表現するときは and キーワードでつなぎます。また他言語のように {} は使わずにインデントでスコープを表現します。

```python
name = "John"
age = 20
license = True

if age >= 20 and license:
    print("OK " + name)
    print("OK {}".format(name))
else:
    print("NG")
```

実行結果は次のように表示されるでしょう。真の場合の２つのprintは同じ意味です。

```
OK John
OK John
```

> 論理和は or キーワードを使います。


### for文

for文を使って繰り返し構造を定義できます。指定回数繰り返すにはrange関数を使います。

```python
for i in range(3):
    print(i)

for i in range(11, 13):
    print(i)

for i in range(103, 101, -1):
    print(i)
```

実行結果は次のように表示されるでしょう。

```
0
1
2
11
12
103
102
```

<div style="page-break-before:always"></div>


### リスト

Pythonは [] キーワードでリスト（配列）を生成できます。

```python
fruits = ["apple", "banana", "cherry"]

print("fruit = " + fruits[1])

for f in fruits:
    print(f)

for i, f in enumerate(fruits):
    print(i, f)
```

実行結果は次のように表示されるでしょう。

```
fruit = banana
apple
banana
cherry
0 apple
1 banana
2 cherry
```

> enumerate関数を使えば、インデックスとリストの要素を取り出すことができます。


### ディクショナリ

Pythonは {} キーワードでディクショナリを生成できます。ディクショナリとはキーと値をマッピングしたものです。

```python
fruits = {"apple":100, "banana":200, "cherry":300}

print(fruits["apple"])

for key in fruits:
    print(key, fruits[key])
```

実行結果は次のように表示されるでしょう。

```
100
apple 100
banana 200
cherry 300
```

<div style="page-break-before:always"></div>


### 関数

Pythonではdefキーワードを使って関数を定義します。

```python
def sum(start, end):
    result = 0
    for i in range(start, end + 1):
        result = result + i
    return result

print(sum(1, 1))
print(sum(1, 5))
print(sum(1, 10))
```

実行結果は次のように表示されるでしょう。

```
1
15
55
```

<div style="page-break-before:always"></div>

### クラス

Pythonはオブジェクト指向言語です。クラスを定義するには次のように実装します。

```python
class Car:
    def __init__(self, name, gas):
        self.name = name
        self.gas = gas

    def move(self):
        if self.gas > 0:
            self.gas = self.gas - 1
            print("{}: move".format(self.name))
        else:
            print("{}: stop".format(self.name))


car1 = Car('kbox', 3)
car2 = Car('Kwagon', 5)

for i in range(5):
    car1.move()
    car2.move()
```

実行結果は次のように表示されるでしょう。

```
kbox: move
Kwagon: move
kbox: move
Kwagon: move
kbox: move
Kwagon: move
kbox: stop
Kwagon: move
kbox: stop
Kwagon: move
```

クラスの中でインスタンス自身を参照するにはselfキーワードを使います。またクラスに定義したメソッドの第1引数にはselfを指定する必要があります。また、クラスにコンストラクタを定義する場合は \_\_init\_\_メソッドを実装します。

> クラスからインスタンスを生成するときにnewキーワードは不要です。
