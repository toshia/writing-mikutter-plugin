---
layout: post
title:  "定数、変数、メソッド"
date:   2016-05-11 00:00:00 +0900
categories:
- basis
---

プラグイン内で変数やメソッドを利用する方法です。

## メソッド

```ruby
Plugin.create :method_sample do
  on_update do |s, ms|
    uwa(ms) # リスナ内部なら定義より前で呼べる
  end

  # 不思議な力で存在を消す
  def uwa(obj)
    puts "(((#{obj}))) うわああああああ"
  end

  uwa(true) # プラグインDSL上から呼ぶには定義の後に書く
end

# uwa(true) # ここでは呼ぶことができない。メソッドはプラグインの内部でのみ呼べる
```

上記例では`uwa()`というメソッドを定義しています。メソッドの定義には、通常のRubyのそれと同じで`def`を使います。プラグインDSL内部で定義すれば、プラグイン固有のメソッドになるので、他のプラグインと名前が衝突してしまう心配はありません。

メソッドを定義できる範囲と呼べる範囲はどちらも `Plugin.create do` から `end` までの間です。

## 変数

### ローカル変数

```ruby
Plugin.create :local_variable_sample do
  lex = nil # 変数の定義

  on_update do |s, ms|
    lex = ms   # 上で定義した変数に代入される
    local = ms # これは、onupdate do ... end の間でしか使えない
  end

  # 不思議な力で存在を消す
  def uwa(obj)
    puts "#{local}☛ (((#{obj}))) うわああああああ" # on_updateの中のlocalはアクセスできない
    puts "#{lex}☛ (((#{obj}))) うわああああああ"   # メソッド内ではプラグインの中で定義したローカル変数もアクセスできない
  end
end

# lex # プラグインの外なのでアクセスできない
```

ローカル変数には大きく分けて二種類あります。

- **プラグインDSL上で定義した変数**
  例では`lex`です。プラグイン内のブロックからアクセスできます。メソッドからはアクセスできません。
- **イベントリスナ等のブロック上で定義した変数**
  例では`local`です。そのブロックの外からアクセスすることができません。



## 実用例

次のコードは、毎日0時になったら自動的に「よるほー」とツイートする例です。24行目で`Service.primary.post`を使用しています。

<script src="https://gist.github.com/toshia/1306093.js"></script>

## 参考
- <a href="http://mikutter.hachune.net/rdoc/Service.html">Service</a>
- <a href="http://mikutter.hachune.net/rdoc/Service.html#method-i-primary">Service.primary</a>
- <a href="http://mikutter.hachune.net/rdoc/MikuTwitter/APIShortcuts.html#method-i-post">MikuTwitter::APIShortcuts#update</a>
