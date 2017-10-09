# 1 AI

<img src="img/01_04.png" width="500px">

これは「AI」というトピックでのGoogle Trendsの結果です。2014年から徐々に増加しはじめて、2016年から2017年に掛けて増加の勢いが増しているのが見て取れます。

現在のAIブームは第3次AIブームと言われています。過去2度のブームは大きな社会の流れにはならずに終焉しました。しかし、今回ブームはこの傾向はまだ暫く続くだけでなく、社会を変革するトレンドになると多くのシンクタンクやメディア、政府、アナリストが予想しています。

それでは、これまでのブームと現在のAIは一体何が違うのでしょうか？現在のAIでは一体何が出来るようになったのでしょうか？

この講座ではハンズオンを交えながら技術的側面から、AIの現状を理解し、実践できるようになることを目標としています。AIは今後も変化・進歩の激しい分野ではありますが、本講座の学習内容が、AIの理解の第一歩となればと思っています。

|時代|AIの特徴|
|:--|:--|
|第1次AIブーム<br>1950 〜1960年代|探索・推論アルゴリズムの時代<br>迷路やチェスのようなゲーム攻略（トイ・プロブレム）が中心|
|第2次AIブーム<br>1980年代|エキスパートシステムの時代<br>専門家の知識をAIに登録する|
|第3次AIブーム<br>2010年頃〜|ディープラーニングによるブレイクスルー（子どものAI）<br>画像認識や音声認識技術の大幅な精度の向上|

<div style="page-break-before:always"></div>

## 1.1 AIの定義

AI（=Artificial Intelligence=人工知能）は非常に広い概念で、様々な分類・区分けがあります。その一つが「汎用AI」と「特化型AI」です。

### 汎用AI

人間のように、あらゆる状況を判断することの出来るAIを言います。人間と同等の学習、推論、反応を示すことを目標としています。汎用AIの研究は進んでいますが、実用化にはまだまだ時間のかかる研究分野です。

> 例）ドラえもん（ドラえもん）／J.A.R.V.I.S.（アイアンマン）

### 特化型AI

特定の課題を解決する目的で、学習、推論が出来るAIを言いいます。対象とする課題においては人間以上の成果を示すこともあります。現在ビジネスの場で活用されているAIはこの特化型AIを指します。

> 例）AlphaGo（DeepMind）／Watson（IBM）


本講座で学習するAIは、この特化型AIに分類されます。


## 1.2 大人のAI、子どものAI

「コンピュータ（AI）はチェスに強くても、花を見て花と認識したり、ものをつかんだり、といった、子どもにできることができない。」

これはモラベックのパラドックスと呼ばれるもので、従来のAIの得意なこと、苦手なことをうまく言い表しています。実は、現在のAIは花を見て花と認識することや、ものをつかむ、という動作を実現できる段階にあります。それでは従来のAIと現在のAIの違いとは一体何なのでしょうか。

AIを分類するときに大人のAI、子どものAIといった分け方があります。大人のAIとは人間が作り込んだAIを指します。これは人が設計したルールをコンピュータにプログラミングしたAIのことです。

一方の子どものAIは、コンピュータが子どものように自分でルールを学習していくプログラムです。子どものAIでは、AIの開発者が複雑なルールを考え、設計する必要はありません。膨大なデータからコンピュータに学習させて、コンピュータ自身に法則性を見つけさせるのです。

子どものAIを実現する手法として注目を集めているのがディープラーニングです。AIの専門家でなくても、誰もがAIを開発できる時代が来ているのです。

<div style="page-break-before:always"></div>


## 1.3 AIとプログラム

AIも結局はプログラムであることに変わりはありませんが、AIと呼ばれるプログラムと一般的なプログラムには違いがあります。

たとえば次の画像ファイルについて考えてみましょう。

<img src="img/01_03.png">

上記のような画像ファイルを処理するプログラムは次のようなものがあるでしょう。

+ 画像サイズ（400, 400）を（200, 200）に変更する
+ 画像をグレースケールに変更する
+ 200,200の位置のピクセルのカラーを取得する

これらの処理は従来のプログラムで実装可能です。画像処理の経験のあるプログラマーであれば実装方法が想像できるでしょう。

それでは以下の処理はどうでしょうか。

+ この花はアサガオかどうか判定する
+ この花は美しいかどうか判定する
+ この花が枯れるのはいつか予測する

プログラマーであっても「アサガオかどうか判定する方法」が具体的ではないため、実装方法に戸惑うでしょう。同様に「美しい花」の定義が曖昧ですし、「いつ枯れるのか」のいう予測も、花の開き具合から判断するのか、そもそも花の開き具合を判定する方法はどのように測るのか、と考えていくと簡単ではないと気づくでしょう。

現在のAIが得意な分野はここにあります。AIの中にはルールがあります。たとえば囲碁を打つAIであれば、次の一手を決定するのがルールです。このルールを人が設計するのか、コンピュータ自身が設計するのか、ここに大きな違いがあります。現在話題になる多くのAIは機械学習と呼ばれる手法を用いて、コンピュータがデータからルールを設計します。このルールのことをモデルと呼ぶこともあります。

<div style="page-break-before:always"></div>


## 1.4 AIの活用方法

少し視点を変えて、現在のビジネスシーンにおいてAIを活用する方法を整理してみましょう。大きく2つにわけることができるでしょう。

+ AIサービスの活用
+ 機械学習ライブラリによる開発


### AIサービスの活用

Googleのクラウドプラットフォーム（GCP）やIBMのコグニティブサービスWatsonでは、画像認識や音声認識、自然言語処理といった機能がサービスとして公開されています。これらのサービスは専用のアカウントを作成すれば、すぐに活用することができます。

<img src="img/01_02.png" width="500px">


### 機械学習ライブラリによる開発

Python言語を中心にTensorFlowなどの機械学習ライブラリが充実しています。これらのライブラリを活用することで自社のビジネスに特化したAIを開発することも可能です。

本講座では機械学習ライブラリの活用方法を紹介します。

<img src="img/01_01.png" width="500px">


<div style="page-break-before:always"></div>

## 1.5 AIの活用事例

最後にAIの活用事例をいくつか見てみましょう。

### チャットボットまなみさん LOHACO By ASKUL

インターネット通販サービスLOHACOではお客様サポートページにチャットボットまなみさんを導入しています。

https://lohaco.jp/support/index.html

IBMのWatsonをベースに開発されたチャットボットは、利用者からの問い合わせに対して適切なサポートメッセージを表示します。

<img src="img/01_05.png" width="500px">

> チャットボットのような言語処理は、AIの中でも自然言語処理と呼ばれる分野です。


<div style="page-break-before:always"></div>

### Microsoft Pix カメラ

Microsoft Pix カメラは 連写して撮影した写真の中から一番良い写真を判別してくれます。「一番良い写真」という曖昧な判定条件は従来のプログラムでは設計の難しい分野です。人間のような感覚を搭載したAIをうまく活用しているケースと言えるでしょう。

<img src="img/01_06.png" width="500px">


<div style="page-break-before:always"></div>

### トンネル切羽AI自動評価システム 安藤ハザマ

画像認識技術はビジネスシーンでも活用されています。安藤ハザマ社の開発したトンネル切羽AI自動評価システムはコーポレートページ上で以下のように解説されています。

> ｢トンネル切羽AI自動評価システム｣は、人工知能の画像認識技術を活用し、切羽写真から岩盤の工学的特性を自動評価するものです。

採掘現場の切羽写真とその地質の特性をディープラーニング（畳み込みニューラルネットワーク）によって学習し独自のAIシステムを開発しています。

<img src="img/01_07.png" width="500px">

## 1.6 Python入門

本講座ではPython言語を活用して機械学習の理解を深めます。Pythonは多目的に活用できるシンプルで扱いやすい言語です。インデントによってif文やfor文などのスコープを表現するという特徴があります。また機械学習ライブラリが充実しているため近年注目を集めています。

### Anaconda

Pythonで機械学習を始めるにはCONTINUUM社の配布しているはAnacondaディストリビューションを活用すると良いでしょう。以下のURLからダウンロートすることができます。

https://www.continuum.io/downloads


AnacondaにはPython本体だけでなく、機械学習に必要なライブラリも梱包されているため、すぐに機械学習プログラミングを始めることができます。


<img src="img/2_8.png" width="600px">

> 本講座のプログラムは Python 3.5.2 :: Anaconda 4.2.0 (x86_64) で動作確認しています。

<div style="page-break-before:always"></div>


### ipython

Anacondaにはipythonというインタラクティブなpythonツールが含まれています。ターミナル（コマンドプロンプト）上で ipython コマンドを実行すると対話形式でpythonプログラムを実行できます。標準の python コマンドも対話形式でプログラムを実行できますが、ipython はコードハイライトやコード補完機能が強化されています。

<img src="img/2_6.png" width="600px">


<div style="page-break-before:always"></div>

### Jupyter Notebook

AnacondaにはJupyter Notebookというブラウザ上で動作するPython開発環境も付属しています。ターミナル（コマンドプロンプト）上で jupyter-notebook コマンドを実行するとローカルでサーバプログラムが起動し、ブラウザが起動します。

<img src="img/2_7.png" width="600px">

> デフォルトでは8888番ポートで起動します。

Jupyter Notebookでは、ノートブックという形式でファイルを作成できます。ノートブックにはPythonコードや実行結果だけでなく、Markdown形式で文章を挿入することもできるので、データ分析の評価レポートを作成しやすいようになっています。

> ノートブックファイルは拡張子.ipynbという形式保存されます。

Jupyter Notebookの実体はブラウザで利用するWebアプリケーションです。そのためサーバ上でJupyter Notebookを起動すれば、手元のパソコンのブラウザからサーバ上でPythonプログラムを実行することも可能です。複数の利用者がデータ解析を共有する場合にも便利でしょう。

<div style="page-break-before:always"></div>


## 1.7 速習 Python

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

Pythonの文字列、数値、実数、真偽値型、None型を確認してみましょう。

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
    print("OK {}".format(name))
else:
    print("NG")
```

実行結果は次のように表示されるでしょう。

```
OK John
```

> 論理和は or キーワードを使います。


### for文

for文を使って繰り返し構造を定義できます。指定回数繰り返すにはrange関数を使います。

```python
for i in range(3):
    print(i)

for i in range(1, 3):
    print(i)

for i in range(3, 1, -1):
    print(i)
```

実行結果は次のように表示されるでしょう。

```
0
1
2
1
2
3
2
```

<div style="page-break-before:always"></div>


### リスト

Pythonは [] キーワードでリスト（配列）を生成できます。

```python
fruits = ["apple", "banana", "cherry"]

print(fruits[1])

for fruit in fruits:
    print(fruit)

for i, fruit in enumerate(fruits):
    print(i, fruit)
```

実行結果は次のように表示されるでしょう。

```
banana
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
