---
layout: post
title:  "UserMixinについて"
date:   2016-10-01 00:00:00 +0900
categories:
- model
---

[カスタムModelをタイムラインに表示する]({% post_url 2016-10-01-model-messagemixin %})の`user`メソッドが返すModelは、ユーザの名前やアイコンを表示するために、幾つかのインターフェイスを要求します。その基準を満たしていない場合、タイムラインに表示することはできません。

このセクションでは、実際にカスタムModelをタイムラインに表示している[RSSプラグイン](https://github.com/toshia/rss)を参考にして、その方法を解説します。

# 例

以下のコードは、RSSプラグインの[Siteクラス](https://github.com/toshia/rss/blob/master/model/site.rb)です。

```ruby
module Plugin::RSS
  class Site < Retriever::Model
    include Retriever::Model::UserMixin

    field.string :name, required: true
    field.string :description
    field.string :link
    field.time   :created
    field.string :profile_image_url
    field.string :feed_url

    def idname
      link
    end

    def perma_link
      link
    end

    def modified
      created
    end
  end
end
```

# Retriever::Model::UserMixin

やはりいくつか要求されるメソッドがありますが、[カスタムModelをタイムラインに表示する]({% post_url 2016-10-01-model-messagemixin %})と同じように、モジュールをincludeするだけで大抵のインターフェイスが備わります。

こちらにincludeするのは、`Retriever::Model::UserMixin`です。

# 実装すべきメソッド

## profile\_image\_url

アイコン画像のURLを文字列で返します。いまのところ、アイコンは省略することができません。

## name

ユーザの名前です。

## idname

ユーザのスクリーンネームです。ただし、スクリーンネームはTwitter独自の機能なので、`nil`を返すことができます。その場合、タイムライン上には名前のみが表示されるようになります。

# 参考
- [Retriever::Model::UserMixin](http://mikutter.hachune.net/rdoc/Model/UserMixin.html)
- [カスタムModelをタイムラインに表示する]({{ site.baseurl }}{% post_url 2016-10-01-model-messagemixin %})
