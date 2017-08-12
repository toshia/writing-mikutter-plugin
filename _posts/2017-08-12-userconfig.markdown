---
layout: post
title:  "UserConfig"
date:   2017-08-12 00:00:00 +0900
categories:
- basis
---

mikutterが再起動されると、変数にセットした値はすべて初期化されます。
大きなデータを恒久的に保持するには、ファイルなどに書いておく必要がありますが、小さなデータの場合は、標準のKey-Value Storeを利用できます。

mikutterには標準でConfigLoaderとUserConfigという、データを恒久的に保持する二つの仕組みがあります。

このうち、UserConfigはConfigLoaderを使って実装されているので、ConfigLoaderの使い方を学ぶと、UserConfigの理解もスムーズです。

# ConfigLoader

## 基本的な使い方

プラグインのコンテキストで、 `at` と `store` というメソッドが利用できます。 `store` で保存したデータはファイルに保存されるので、mikutterを再起動してもそのデータを読み出すことができます。

```ruby
Plugin.create :miku_otaku do
  store(:wife, '初音ミク') # 再起動後も嫁を覚えている
end
```

第一引数にkey（後述）、第二引数に値を渡します。

読み出す時は、 `at` を使います。

```ruby
Plugin.create :miku_otaku do
  puts at(:wife) # 「初音ミク」と出力される
end
```

この時、 `at` の第一引数には `store` に渡したのと同じキーを渡す必要があります。

```ruby
Plugin.create :miku_otaku do
  puts at(:friend) # key「friend」に対応する値はstoreされてないので、nilが返ってくる(何も表示されない)
end
```

もちろん、 `:friend` に対して値が事前にstoreされていれば、その値を受け取れます。

## 保存できるデータの種類

Rubyのクラスによって制限されています。許可されていない

ConfigLoaderの実装である `core/configloader.rb` には、次のように書かれており、このリストにあるクラスのインスタンスであれば、正しく保存できます。

```ruby
AVAILABLE_TYPES = [Hash, Array, Set, Numeric, String, Symbol, Time, NilClass, TrueClass, FalseClass]
```

このリストにないクラスのインスタンスを `store` に渡すと、例外[ArgumentError](https://docs.ruby-lang.org/ja/latest/class/ArgumentError.html) が発生します。

## プラグイン毎に異なる領域に保存される

基本的な使い方の例で扱った miku_otaku プラグインのデータが残った状態で、以下のようなプラグインをロードしても、 `at` は `nil` を返すため、何も表示されません。

```ruby
Plugin.create :atelier_otaku do
  puts at(:wife)
end
```

ConfigLoaderは、プラグイン毎に異なる仮想領域に保存しているため、異なるプラグインが同じキーに対して別々の値を保持できます。

```ruby
Plugin.create :atelier_otaku do
  store(:wife, 'logy')
end
```

他のプラグインが保持している値を読み書きする方法はありません。

# UserConfig

## 基本的な使い方

UserConfigは、ConfigLoaderと違ってプラグインコンテキストに限らず、どこでも利用できます。

```ruby
UserConfig[:foobar] = 1
puts UserConfig[:foobar] # 「1」 と表示される
```

## 保存できるデータの種類

内部的には、ConfigLoaderを使って保存しているだけなので、全く同じ制約を受けます。

## 単一の領域に保存される

ConfigLoaderからは、UserConfigは一つのプラグインのように見えていて、UserConfigという仮想領域を与えています。
したがって、UserConfigに書き込んだのと読み込んだのが異なるプラグインだったとしても、キーが同じなら同じ値を受け取れます。

```ruby
puts UserConfig[:foobar] # 「1」と表示される

Plugin.create :miku_otaku do
  puts UserConfig[:foobar] # 「1」と表示される
end
```

これは言い換えれば、単純な名前では意図せずキーが衝突してしまう危険性があるということです。なので、UserConfigを使う場合は、キーの前にプラグインスラッグをつける習慣があります。

```ruby
puts UserConfig[:foobar] # 「1」と表示される

Plugin.create :miku_otaku do
  puts UserConfig[:miku_otaku_foobar] # nilが返り、何も表示されない
end
```

## 値の変化をイベントで受け取る

UserConfigは、値が変更されると `:userconfig_modify` イベントが発生します。値が変化したら即座に何かしたい場合は、このイベントを活用しましょう。

```ruby
Plugin.create :miku_otaku do
  on_userconfig_modify do |key, newval|
    if key == :favorited_anime && newval == "ゆゆ式"
      Plugin.call(:send_bluray, rand(6))
    end
  end
end
```

このコードは、 `UserConfig[:favorited_anime] = 'ゆゆ式'` のようなコードが実行されると、 `:send_bluray`イベントを発生させます。例えばmikutterではタブの位置を設定画面で「上下左右」いずれかから選択出来るようになっていて、変更した瞬間にタブの位置が変わります。これは、 `userconfig_modify` イベントでタブの位置の設定を監視しているからです。

引数の `newval` は、このイベントが発生した理由となった更新後の値です。普通は `UserConfig[key]` の値と同じですが、イベントリスナが実行される前に複数回値を書き換えられた場合は、一致しない場合があります。その場合、変更された回数だけイベントが発生します。

# 使い分け

ConfigLoaderで事足りるなら、ConfigLoaderを使いましょう。しかし、多くの場合はUserConfigを使うことになると思います。

## キーの衝突について

プラグイン間のキーの衝突を避けてくれるという点でConfigLoaderは優れていますが、他のプラグインのキーを読み出すことはできません。

したがって、設定画面で値を設定させるような場面では、UserConfigを使わざるを得ません。殆どの場合、設定画面で設定させるためにUserConfigを利用することになります。

## 値の変化を監視する

UserConfigは、値が変化する度にイベントを発生させますが、ConfigLoaderにはその機能がありません。何故ならConfigLoaderはプラグインに対して仮想的なKey-Value型の記憶領域を与えるためのものなので、他のプラグインに通知することに意味がないからです。

# ConfigLoaderのつかいどころ

ConfigLoader（とUserConfig）は、多くのプラグインによって長い間使われてきたため、パフォーマンスとデータの安全性において一定の実績がありますが、万能ではありません。実際にどのようにデータが保存されるかを知ることで、ConfigLoaderを利用すべきときと避けるべきときを判断する助けになるでしょう。

## データフォーマット

`~/.mikutter/settings/setting.yml` に、YAML形式で値を保存します。異なるPluginのConfigLoaderも、UserConfigも、すべてこの1ファイルに収まっています。過去にはpstoreを用いていて、その時はMarshalを使っていました。フォーマットやパスは将来変わるかもしれませんが、ConfigLoaderやUserConfigのインターフェイスが変わりません。

YAMLだと、設定値が人間でも読めてしまいますし、変更もできます。YAMLにしたのは、設定値をmikutterを介さずに読み書きしたいという要望に応えたためで、意図的なものです。外部サービスの認証情報など、読まれたくないデータを保存するのには向きません。

## ロード

mikutterが起動する時に、一度だけファイルが読み込まれ、YAMLをパースした結果をメモリ上に記憶します。

実際に値が参照される時には、ファイルは参照せず、常にメモリ上の内容を返します。そのため参照は高速で、ストレージに負荷を与えません。

このことから、ConfigLoaderに画像などの巨大なデータを設置するのは向いていないということがわかります。一方、小さなデータを何度も読み出すのはConfigLoaderの得意とするところです。

また、ごく稀にしか読まれないようなデータをstoreするのは、避けるべきではないですが、向いてもいません。事前にロードされてしまうので、必ず1度は使うようなデータに向いています。

## セーブするタイミング

ConfigLoaderのデータがファイルに書き出されるのは次の2つの時です。

1. mikutterが終了する時
2. いずれかのキーの更新があってから5秒後

mikutterが終了する時には必ず値を保存します。クラッシュしたときでも最後の情報をセーブするようになっているため、値のロールバックが発生する可能性は低いです。

それ以外の場合は、キーの更新があってから5秒後に、データをファイルに書き出します。これは、頻繁に更新があっても、ストレージにできるだけ負荷を掛けないことを目的とした実装です。したがって、1秒に1度のペースで書き換えが発生したとしても、全体で見ればキッチリ5秒毎にファイルへの保存が行われます。
ConfigLoaderはあらゆる要因で値が変化しますが、1ファイルで管理しているおかげで、書き込み回数を最小限に減らしているのです。

メモリへの更新は即時行われるため、ファイルに書き出される前に `at` などを使ったとしても、常に最新の値が得られます。

UserConfigが書き換えられた時に発生する `userconfig_modify` イベントも、ファイルに書き出されるタイミングに関わらず、 `UserConfig#[]=` で値が書き換えられる度に発生します。

## セーブの手順

データをファイルに書き出す時は、一度同じディレクトリに違う名前の一時ファイルを作って、そちらにデータを保存します。正常に保存し終わったら、一時ファイルの名前を `setting.yml` に変更して上書きし、セーブを完了します。この仕組みによって、書き込み中にクラッシュしてしまってYAMLファイルが破損することを防いでいます。

