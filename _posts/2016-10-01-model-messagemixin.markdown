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

# 実装すべきメソッド・フィールド
MessageMixinは、大抵の必要なメソッドを実装してくれますが、一部のメソッドについては、モジュールを利用するクラスが提供しなければなりません。具体的には以下のメソッドです。

## description

### 説明

本文です。ツイートでいうところの、ツイートの本文にあたります。

### 戻り値

- **<a href="https://docs.ruby-lang.org/ja/latest/class/String.html">String</a>** 本文

## user

### 説明

ユーザです。ユーザ名、ユーザアイコンなどを表示するために使われます。

### 戻り値

- [UserMixin]({{ site.baseurl }}{% post_url 2016-10-01-model-usermixin %}) にかかれているメソッドを実装した **[Model]({{ site.baseurl }}{% post_url 2016-09-05-model %})**

## created

### 説明

投稿が行われた時刻です。Twitterでは、ツイートの投稿日時です。

### 戻り値

- **<a href="https://docs.ruby-lang.org/ja/latest/class/Time.html">Time</a>** 投稿日時

# タイムラインに表示したいなら、registerも忘れずに呼ぶ

ツイートのように、直接タイムラインに表示されるModelは、定義の中で`register`メソッドを呼んでおきましょう。

```ruby
class Item < Retriever::Model
  include Retriever::Model::MessageMixin

  register :rss, name: "RSS Topic"
end
```

registerメソッドの詳しい情報は[メタ情報]({{ site.baseurl }}{% post_url 2016-12-19-register-model %})の記事を参照してください。

# 参考
- [Retriever::Model::MessageMixin](http://mikutter.hachune.net/rdoc/Model/MessageMixin.html)
- [UserMixinについて]({{ site.baseurl }}{% post_url 2016-10-01-model-usermixin %})
- [メタ情報]({{ site.baseurl }}{% post_url 2016-12-19-register-model %})
