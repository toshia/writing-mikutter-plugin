---
layout: post
title:  "UserMixinについて"
date:   2016-10-01 00:00:00 +0900
categories:
- model
---

[カスタムModelをタイムラインに表示する]({{ site.baseurl }}{% post_url 2016-10-01-model-messagemixin %})の`user`メソッドが返すModelは、ユーザの名前やアイコンを表示するために、幾つかのインターフェイスを要求します。その基準を満たしていない場合、タイムラインに表示することはできません。

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

やはりいくつか要求されるメソッドがありますが、[カスタムModelをタイムラインに表示する]({{ site.baseurl }}{% post_url 2016-10-01-model-messagemixin %})と同じように、モジュールをincludeするだけで大抵のインターフェイスが備わります。

こちらにincludeするのは、`Retriever::Model::UserMixin`です。

# 実装すべきメソッド

以下のメソッドは、必ず実装してください。

## icon

### 説明

アイコン画像です。Twitterでいうところの、ユーザのアイコンです。

この画像が、タイムライン上にアイコンとして表示されます。

### 戻り値

- **[Photo Model]({{ site.baseurl }}{% post_url 2016-11-30-photo-model %})**

## name

### 説明

名前です。Twitterでいうところの、ユーザの名前です。

Twitterのスクリーンネームと名前はよく混同されますが、スクリーンネームは「@」から始まるもので、一般的には「ID」と呼ばれているものです。一方、名前は日本語を使えます。

このフィールドは「名前」の部分に表示されるテキストです。

### 戻り値

- **<a href="https://docs.ruby-lang.org/ja/latest/class/String.html">String</a>** 名前

# 参考
- [Retriever::Model::UserMixin](http://mikutter.hachune.net/rdoc/Model/UserMixin.html)
- [カスタムModelをタイムラインに表示する]({{ site.baseurl }}{% post_url 2016-10-01-model-messagemixin %})
