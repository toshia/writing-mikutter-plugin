---
layout: post
title:  "カスタムModel"
date:   2016-10-01 00:00:00 +0900
categories:
- model
---

# Modelとは

mikutterでTwitter上のオブジェクト以外のものを扱うには、Modelを新しく定義し、それを利用します。Modelはmikutter上で値を扱うことに特化しており、Modelを使ってデータを表現することで、例えばそれをタイムライン上でツイートのように表示することができるようになります。

MessageやUserといった、mikutterでよく扱うオブジェクトも、全てModelの一部です。mikutter 3.5では、プラグインがこのModelを独自に定義することで、Twitter以外のオブジェクトを扱うことができるようになります。

# 最小のModel

```ruby
class BlankModel < Retriever::Model
end
```

Modelは、`Retriever::Model`を継承したクラスのことで、新たなModelを追加するには、これを継承したクラスを新たに作れば良いのです。

## 値の読み書き

この状態では、`[]`や`[]=`を使って、`Hash` のように値を読み書きすることができます。

```ruby
bm = BlankModel.new
bm[:foo] = 1
bm[:foo] # => 1
```

ただし、キーは`Symbol`にしましょう。

## URI

すべてのModelのインスタンスには、URIが割り振られています。違うデータを示すなら、必ずこれは違う値になります。

```ruby
bm = BlankModel.new
bm.uri # => blankmodel://blankmodel/550e8400-e29b-41d4-a716-446655440000
```

デフォルトでは、上のように、インスタンスごとにランダムな値が生成されます。

インスタンスが異なっても、本質的に同じデータを指すものであれば、同じURIを返すようにしましょう。例えばツイートのURIは、以下の様に、ツイートのパーマリンクになっています。

```
https://twitter.com/toshi_a/status/781963811016155136
```

mikutterは、これによってModelの同一性を判定します。あなたが定義するModelも、一意なURIを返すように`uri`をオーバライドしましょう。

なお、`Retriever::Model#uri`の戻り値は、<a href="https://docs.ruby-lang.org/ja/latest/class/URI=3a=3aGeneric.html">URI::Generic</a>か、そのサブクラスです。

## perma_linkとURI

Modelが`perma_link`メソッド(またはフィールド)を持っている場合、その戻り値がURIとして使われます。

httpまたはhttpsスキームを持っていて、Web上のリソースを指すのが`perma_link`です。一方`uri`は、それ以外のURI一般を含みます。`perma_link`は`uri`として使うことができるので、`perma_link`を定義して、それが一意なURLを返すなら、`uri`メソッドを定義する必要がありません。
