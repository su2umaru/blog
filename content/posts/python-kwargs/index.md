---
title: "Python で可変長キーワード引数のキーが位置引数やデフォルト引数と一致すると位置引数やデフォルト引数として扱われる"
date: 2021-08-21T19:00:00+09:00
lastmod: 2021-08-21T19:00:00+09:00
draft: false
description: "su2umaru (すずまる) です。Python で可変長キーワード引数のキーが位置引数やデフォルト引数と一致すると位置引数やデフォルト引数として扱われることに気づきました。これを機に引数について書きます。"

resources:
- name: "featured-image"
  src: "featured-image.png"

tags: ["Python", "可変長キーワード引数"]
categories: ["プログラミング"]

toc:
  enable: true
lightgallery: true
---

[@su2umaru](https://twitter.com/su2umaru) です。Python で可変長キーワード引数のキーが位置引数やデフォルト引数と一致すると位置引数やデフォルト引数として扱われることに気づきました。これを機に引数について書きます。

<!--more-->

## Python の引数

まず用語の整理をします。Python の引数は次の4種類に分かれます。

1. 位置引数

```python
def fit(arg):
    print(arg)

fit(1)  # 出力: 1
fit(arg=2)  # 出力: 2
```

2. デフォルト引数

```python
def fit(arg=1):
    print(arg)

fit()  # 出力: 1
fit(2)  # 出力: 2
fit(arg=3)  # 出力: 3
```

3. 可変長位置引数

```python
def fit(*args):
    print(args)

fit(1)  # 出力: (1,)
fit(1, 2)  # 出力: (1, 2)
fit(1, 2, 3)  # 出力: (1, 2, 3)
```

4. 可変長キーワード引数

```python
def fit(**kwargs):
    print(kwargs)

fit(key1=1)  # 出力: {'key1': 1}
fit(key1=1, key2=2)  # 出力: {'key1': 1, 'key2': 2}
fit(key1=1, key2=2, key3=3)  # 出力: {'key1': 1, 'key2': 2, 'key3': 3}
```

4つ目の可変長キーワード引数についてさらに掘り下げます。

## Python の可変長キーワード引数の使われ方

可変長キーワード引数を使うとき、先に引数に与える辞書型の変数を定義しておく方法がよくとられます。

```python
def fit(**fit_kwargs):
    print(f"fit_kwargs: {fit_kwargs}")
    print(f"fit_kwargs['time_budget']: {fit_kwargs['time_budget']}")

automl_settings = {
    "time_budget": 2,
    "log_file_name": "iris.log",
}

fit(**automl_settings)
```

上のコードを実行すると次のように出力されます。

```
fit_kwargs: {'time_budget': 2, 'log_file_name': 'iris.log'}
fit_kwargs['time_budget']: 2
```

先に定義した辞書型の変数 `automl_settings` を関数 `fit` の引数として与えると、関数 `fit` 内で辞書型の変数 `fit_kwargs` として処理されます。そのため `fit_kwargs['time_budget']` のようにキーを指定するとキーに対する値を取得できます。

## 可変長キーワード引数と位置引数やデフォルト引数を一緒に使う

関数の引数のうち一部を位置引数、一部をデフォルト引数、一部を可変長キーワード引数として使うこともできます。

```python
def fit(task, metric="auto", **fit_kwargs):
    print(f"task: {task}")
    print(f"metric: {metric}")
    print(f"fit_kwargs: {fit_kwargs}")
    print(f"fit_kwargs['time_budget']: {fit_kwargs['time_budget']}")

automl_settings = {
    "time_budget": 2,
    "log_file_name": "iris.log",
}

fit("classification", metric="accuracy", **automl_settings)
```

上のコードを実行すると次のように出力されます。

```
task: classification
metric: accuracy
fit_kwargs: {'time_budget': 2, 'log_file_name': 'iris.log'}
fit_kwargs['time_budget']: 2
```

## 可変長キーワード引数のキーが位置引数やデフォルト引数と一致すると位置引数やデフォルト引数として扱われる

では次のコードの場合はどのように処理されるでしょうか。

```python
def fit(task, metric="auto", **fit_kwargs):
    print(f"task: {task}")
    print(f"metric: {metric}")
    print(f"fit_kwargs: {fit_kwargs}")
    print(f"fit_kwargs['time_budget']: {fit_kwargs['time_budget']}")

automl_settings = {
    "task": "classification",
    "metric": "accuracy",
    "time_budget": 2,
    "log_file_name": "iris.log",
}

fit(**automl_settings)
```

上のコードを実行すると次のように出力されます。

```
task: classification
metric: accuracy
fit_kwargs: {'time_budget': 2, 'log_file_name': 'iris.log'}
fit_kwargs['time_budget']: 2
```

エラーとはならずに実行できました。そして位置引数やデフォルト引数を明示的に与えていませんが、関数 `fit` 内で `task` と `metric` には値が与えられているようです。よく見ると `automl_settings` として定義した値が与えられていますね。この挙動を残しておきたく、この記事を書きました。

## 可変長キーワード引数と位置引数やデフォルト引数の競合

上の挙動を踏まえ、可変長キーワード引数と位置引数やデフォルト引数に同じ値を与えたときの挙動が気になったので実験を行いました。

```python
def fit(task, metric="auto", **fit_kwargs):
    print(f"task: {task}")
    print(f"metric: {metric}")
    print(f"fit_kwargs: {fit_kwargs}")
    print(f"fit_kwargs['time_budget']: {fit_kwargs['time_budget']}")


automl_settings = {
    "task": "classification",
    "metric": "accuracy",
    "time_budget": 2,
    "log_file_name": "iris.log",
}

fit("classification", **automl_settings)
```

上のコードを実行すると次のように出力されます。

```
Traceback (most recent call last):
  File "/Users/su2umaru/dev/kwargs/main.py", line 15, in <module>
    fit("classification", **automl_settings)
TypeError: fit() got multiple values for argument 'task'
```

エラーとなってしまいました。同じ値を与えても、2つ以上の値を与えるとエラーとなるようです。

## 引数について整理できたいい機会でした

普段何気なく使っている引数について、一歩立ち止まって挙動を確認できたいい機会でした。言語仕様を理解した上で、今後より早くより正しく開発を進めていきます。