---
layout: post
title:  "カスタムModelをタイムラインに表示する"
date:   2016-10-01 00:00:00 +0900
categories:
- model
---

独自に定義したModelは、一定の基準を満たせばTimelineに表示することができます。その基準を満たしていない場合、タイムラインに表示することはできません。

このセクションでは、実際にカスタムModelをタイムラインに表示している[RSSプラグイン](https://github.com/toshia/rss)を参考にして、その方法を解説します。

# 例

以下のコードは、RSSプラグインの[Itemクラス](https://github.com/toshia/rss/blob/master/model/item.rb)です。

```ruby
class Item < Retriever::Model
  include Retriever::Model::MessageMixin

  register :rss, name: "RSS Topic"

  field.string :guid
  field.string :link
  field.string :title, required: true
  field.string :description
  field.time   :created
  field.has    :site, Plugin::RSS::Site, required: true

  def to_show
    @to_show ||= self[:title].gsub(/&(gt|lt|quot|amp);/){|m| {'gt' => '>', 'lt' => '<', 'quot' => '"', 'amp' => '&'}[$1] }.freeze
  end

  def user
    site
  end
end
```

# MessageMixin
タイムライン、すなわちGUIにModelを表示するには、多くのメソッドを提供しなければいけません。

それらのメソッドはたいてい固定値を返していれば良いため、そういったメソッドをほとんどすべて実装した<a href="http://mikutter.hachune.net/rdoc/Model/MessageMixin.html">Retriever::Model::MessageMixin</a>をincludeし、そこから必要なメソッドをオーバライドしていきます。

# 実装すべきメソッド
MessageMixinは、必要なメソッドをすべて実装してはくれません。デフォルト値を返すようにしてくれるだけなので、デフォルト値が存在しないようなメソッドは定義する必要があります。具体的には、次のメソッドです。

## to_show

表示するテキストを`String`で返すメソッドです。
最初の例では、RSSのタイトルから、エスケープされた記号をもとに戻したものを返しています。

ここで返した文字列は、mikutterのタイムライン上には、ツイートの本文のように表示されます。

## user

ユーザ名、ユーザアイコンなどを表示するために使われます。

## created

このModelが示すリソースが作成された時刻を`Time`のインスタンスで返します。
例ではcreatedメソッドを直接定義していませんが、このコードで`created`メソッドが定義されています。

```ruby
field.time   :created
```

詳しくは [Modelのfield]({% post_url 2016-10-01-model-field %}) を参照してください。

## 定義すべきメソッドの代わりにフィールドを使う

`created`のように、実装すべきメソッドを定義する代わりに、その名前のフィールドを作成するというテクニックもあります。これは`user`にも同じテクニックを利用できます。単純にコードが短くなるため、憶えておくべきテクニックですが、以下の様な場合には使うべきではありません。

### to_showフィールドは定義せず、必ずメソッドを使う

`to_show`は本文を表すフィールドから取ってきた値を加工するなどして、タイムラインに表示する文字列を返すメソッドにするべきです。RSSの例でも表示するテキストのために加工をしています。

加工されることを期待しているので、`to_show`という名前のフィールドを作って手を抜かないようにしましょう。次に上げるような理由からも、`to_show`フィールドは定義してはいけません。

### タイムラインに表示するために、明らかに格納しているものとは違う名前をつける

例にしたRSSプラグインでは、`user`メソッドの戻り値は、`site`フィールドの内容そのままです。
`site`フィールドをuserという名前に変更すればこのようなことをしなくて良くなりますが、これは悪いアイデアです。

なぜなら`site`フィールドには、`Plugin::RSS::Site`のインスタンスが格納されるようになっており、ユーザではありません。RSSフィードのアイコンをタイムラインに表示するために`user`メソッドを定義しているだけです。このような都合のためにフィールド名を変えるべきではなく、何を格納するかわかりやすい名前をつけるべきです。

もちろん、SNSのプラグインのためにModelを定義しているなら、投稿者として`user`フィールドを定義するのは妥当です。Modelの性質によって適切な方法は変わってきます。

# register

ツイートのように、直接タイムラインに表示されるModelは、定義の中で`register`メソッドを呼んでおきましょう。

```ruby
register :rss, name: "RSS Topic"
```

registerメソッドの詳しい情報は[メタ情報]({{ site.baseurl }}{% post_url 2016-12-19-register-model %})の記事を参照してください。

# 参考
- [Retriever::Model::MessageMixin](http://mikutter.hachune.net/rdoc/Model/MessageMixin.html)
- [UserMixinについて]({{ site.baseurl }}{% post_url 2016-10-01-model-usermixin %})
- [メタ情報]({{ site.baseurl }}{% post_url 2016-12-19-register-model %})
