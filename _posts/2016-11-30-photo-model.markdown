---
layout: post
title:  "Photo Model"
date:   2016-11-30 00:00:00 +0900
categories:
- model
---

Photo Modelは、画像を扱うModelです。

mikutter 3.5からは、テクスチャやアイコン等、ありとあらゆる画像にPhoto Modelを利用します。

# Photo Modelとは

Photo Modelとは、 **特定の一つのModelを指す言葉ではなく、画像を表わすModelの総称** です。

具体的には、次のメソッドを実装したModelのことです。

- download
- completed?
- downloading?
- ready?

これらは、独自に定義したModelに <a href="http://mikutter.hachune.net/rdoc/Model/PhotoMixin.html">Retriever::Model::PhotoMixin</a> をincludeすることで実装されます。mikutter標準プラグインが提供するPhoto Modelは、必ずこの <a href="http://mikutter.hachune.net/rdoc/Model/PhotoMixin.html">Retriever::Model::PhotoMixin</a> をincludeしています。

また、gtkプラグインが入っている環境では、 /core/mui/gtk\_photo\_pixbuf.rb によって、 <a href="http://mikutter.hachune.net/rdoc/Model/PhotoMixin.html">Retriever::Model::PhotoMixin</a> に次のようなメソッドが追加されます。

- download\_pixbuf
- load\_pixbuf
- pixbuf

# ねらい

mikutterでは、タイムライン上のユーザのアイコンや、タブ、mikutterコマンド等、いろんなものに画像を使われていますが、あるときはPixbufを渡したり、ある場所では画像のパスやURLを渡したりと、同じ画像を扱うのにインターフェイスが統一されておらず、プラグイン作成の学習コストが上がる一因となっていました。mikutter 3.5では、互換性を保つために従来の方法は残しながらも、全ての画像を受け取る場所でPhoto Modelを受け付けるようにし、Photo Modelを渡すのを全ての場所で推奨しています。

キャッシュの効率も向上しました。例えばいくつかのサードパーティプラグインを入れて「タイムラインに表示された画像のサムネイルをクリックしてイメージビューアで画像を見て、それをローカルに保存」する時には、今までだと

- タイムラインにサムネイルが表示された時
- イメージビューアで画像を開く時
- それをローカルに保存する時

の3回ダウンロードされていました。Photo Modelの登場のおかげで、全ての画像は短時間であればメモリキャッシュされ、ダウンロードの回数を減らすことを期待できるようになりました。他にも、画像をメモリ上で可能な限り共有し、同じ画像が重複してメモリ上に存在しにくい作りになっているなど、サードパーティプラグインでPhoto Modelを採用すればユーザにも大きなメリットがあります。

# Photo Modelを得る

Photo Modelは、標準プラグインやサードパーティプラグインが独自に実装しています。mikutterでは、他のプラグインが定義したクラスやモジュールに依存するのは推奨されないため、これらのクラスを直接読みに行ってはいけません。

## 画像を直接表わすURL又はローカルのパス

**photo_filter** フィルタを利用しましょう。

```ruby
Plugin.filtering(:photo_filter, 'http://mikutter.hachune.net/img/mikutter.png', [])[1]
```

これで、全てのPhoto Modelの中から、第一引数のURLを表わすものを全て取得します。
また、ローカルにある画像に対しても使えます。

```ruby
Plugin.filtering(:photo_filter, '/path/to/image.png', [])[1]
```

## スキンに対応するPhoto Modelを得る

スキンの画像を示すPhoto Modelも取得することができます。以下の例は、 **icon.png** を取得するコードです。

```ruby
Skin['icon.png']
# または、以下のように書く
Skin.photo('icon.png')
```

このPhoto Modelは少し特殊で、当然 **download** のようなメソッドは使えますが、 **pixbuf** メソッドが若干異なる挙動をします。

**pixbuf** メソッドは、メモリ上にその画像の GdkPixbuf::Pixbuf があればそれを返し、無ければnilを返すメソッドですが、Skinに限っては、メモリ上に無ければロードしてPixbufを作成します。決してnilを返すことはありません。

# Photo Modelを使う

## 画像ビューアなどで開く

Photo Modelに対してIntentを発行すれば、対応しているプラグインで開くことができます。通常のmikutterであれば、イメージビューアプラグインが開いてくれるでしょう。

```ruby
photo = Plugin.filtering(:photo_filter, 'http://mikutter.hachune.net/img/mikutter.png', [])[1].first
Plugin.call(:open, photo)
```

## ユーザのアイコンとして

UserMixinをincludeしたModelのiconメソッドでPhotoを返せば、これがアイコンとして扱われます。

# 参考

- <a href="http://mikutter.hachune.net/rdoc/Model/PhotoMixin.html">Retriever::Model::PhotoMixin</a>
