# Python入門ハンズオン

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
fruits = ["apple", "banana", "orange"]

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
orange
0 apple
1 banana
2 orange
```

> enumerate関数を使えば、インデックスとリストの要素を取り出すことができます。


### ディクショナリ

Pythonは {} キーワードでディクショナリを生成できます。ディクショナリとはキーと値をマッピングしたものです。

```python
fruits = {"apple":100, "banana":200, "orange":300}

print(fruits["apple"])

for key in fruits:
    print(key, fruits[key])
```

実行結果は次のように表示されるでしょう。

```
100
apple 100
banana 200
orange 300
```

<div style="page-break-before:always"></div>


### 関数

Pythonではdefキーワードを使って関数を定義します。

```python
def myfunc(obj):
    print("Object => {}".format(obj))
    return len(obj)

print(myfunc("cat"))
print(myfunc("tiger"))
```

実行結果は次のように表示されるでしょう。

```
Object => cat
3
Object => tiger
5
```

<div style="page-break-before:always"></div>

### クラス

Pythonはオブジェクト指向言語です。クラスを定義するには次のように実装します。

```python
class Dnp:
    def __init__(self, name="Tom", age=30):
        self.name = name
        self.age = age
    def salary(self):
        print("{}: {}".format(self.name, self.age * 10000))
        
d = Dnp("Tiger", 35)
d.salary()

Dnp().salary() # デフォルト値で呼び出し
```

実行結果は次のように表示されるでしょう。

```
Tiger: 350000
Tom: 300000
```

クラスの中でインスタンス自身を参照するにはselfキーワードを使います。またクラスに定義したメソッドの第1引数にはselfを指定する必要があります。また、クラスにコンストラクタを定義する場合は \_\_init\_\_メソッドを実装します。

> クラスのインスタンスを生成するときにnewキーワードは不要です。
