---
layout: post
title:  "メタ情報"
date:   2016-12-19 00:00:00 +0900
categories:
- model
---

# メタ情報とは

Modelの`register`クラスメソッドで、Modelにいくつかのメタ情報を付与することができます。

```ruby
class Topic < Retriever::Model
  register :rss,
           name: "RSS Topic",
           timeline: true
end
```

`register`の第一引数はslugで、mikutter内部でのModel固有の名前を設定します。同時にインストールされる他のModelと被らないようにします。

## メタ情報を定義すべきModelとそうでないModel

基本的には、以下のようなことを想定しているModelでは、メタ情報が必要です。これらの操作をする時、メタ情報が無ければmikutterはクラッシュします。

- `handle`メソッドを利用して、特定のURIに対応させた時
- [Retriever::Model::MessageMixin]({{ site.baseurl }}{% post_url 2016-10-01-model-messagemixin %}) をincludeするなどして、タイムラインに表示されることを想定したModel

# 定義できるメタ情報の一覧

## slug

これだけは名前付き引数ではありません。第一引数に単にSymbolを与えます。
これは、他のModelとは重複しない名前をつけてください。また、`register`を呼ぶ時には、この名前は省略することができません。

slugは内部の設定値に使うので、後から変更することは推奨されません。変更したリリースをすると、このModelに関する設定をユーザがしていた場合、リセットされる可能性があります。

被らない名前をつけるために、`<プラグインslug>_<Modelのslug>`のような名前をつけることをお勧めします。

### 例
```ruby
class Location < Retriever::Model
  register :location

  field.float :lat
  field.float :lng
end
```

## name

設定画面やIntentプラグインによって表示される名前です。

slugがmikutter内部で利用する名前であるのに対して、ユーザに表示することが目的となるので、Stringで渡します。多言語対応プラグインであればこの値は多言語化します。

### 例

```ruby
class Topic < Retriever::Model
  register :rss,
           name: "RSS Topic"
end
```

## timeline

タイムラインに表示することを想定したModelなら、`true`を設定します。

今のところ、`timeline`が`false`でもタイムラインに表示することができますが、表示設定にこのModelのフォントや背景色を設定することができなくなります。

タイムラインに表示されることを想定しているModelには、この値をtrueに設定しておきましょう。

デフォルト値は`false`です。

## reply

このModelがリプライに対応しているならば`true`を渡します。

`true`の場合、表示設定に、リプライだった場合の背景色を設定する項目が追加されます。このModelの`mentioned\_by\_me?`が`true`を返した場合、リプライの設定が反映されるようになります。

基本的に、`timeline`が`true`でない場合は無視される値です。

デフォルト値は`true`です。

## myself

このModelがユーザ自身によって生成される可能性がある類のものであれば`true`を渡します。
例えば、Twitterのツイートのような、ユーザが投稿するようなオブジェクトであれば`true`です。一方でシステムメッセージやRSS等、ユーザが一方的に受け取るものの場合は`false`です。

`true`の場合、表示設定に、自分が生成したオブジェクトの背景色を設定する項目が追加されます。このModelの`myself?`が`true`を返した場合、その設定が反映されるようになります。

基本的に、`timeline`が`true`でない場合は無視される値です。

デフォルト値は`true`です。

# 参考
- [カスタムModel]({{ site.baseurl }}{% post_url 2016-10-01-custom-model %})
