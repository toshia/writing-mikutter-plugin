---
layout: post
title:  "プログラムから特定の時間にツイートする"
date:   2016-09-05 00:00:00 +0900
categories:
- example
---
# 特定の時間にツイートする<a id="sec-2" name="sec-2"></a>

このセクションでは、特定の時間に定型文をツイートする、botのようなプラグインを作成します。

## コード<a id="sec-2-1" name="sec-2-1"></a>

```ruby
# -*- coding: utf-8 -*-
# say "よるほー"
# よるほーの例です。あくまでReserverのデモンストレーション用に作りました。
# なので実戦投入はしないこと。よるほーは自分でタイミングを合わせることにこそ意義があるのです。

Plugin.create :yoruho do
  def main
    Reserver.new(nextyrhtime){
      say_yoruho(Service.primary)
      sleep 1
      main } end

  # 次回のよるほー時間を取得
  def nextyrhtime
    now = Time.new
    result = Time.local(now.year, now.month, now.day, 0, 0)
    while result < now
      result += 86400 end
    result end

  # よるほーとつぶやく
  def say_yoruho(service)
    service.update(message: 'ておほー') end

  main
end
```

## 解説<a id="sec-2-2" name="sec-2-2"></a>

ほぼすべてのプラグインは、以下のテンプレートから作り始めることになります。

    Plugin.create :プラグイン名 do
      (プラグインの定義)
    end

Plugin.create は、プラグインをコアに登録します。また、ブロック内でプラグインを実装するための様々なメソッドを提供しています。
この中で変数や関数を定義すれば外のスコープも汚さないので、基本的にはこのブロックの中だけでプラグインは完結させるべきです。

### メソッドの定義<a id="sec-2-2-1" name="sec-2-2-1"></a>

ブロックの中は、すぐにPluginのインスタンスの中で評価されます。defで
メソッドを定義して、メソッドの内外で呼ぶことができます。例では main
と next\_yrh\_time と say\_yoruho を定義していますね。

### 文字列を投稿する<a id="sec-2-2-2" name="sec-2-2-2"></a>

1.  Serviceオブジェクト

    自動投稿を実現するためには、Serviceクラスのインスタンスを得ます。
    Serviceクラスは、Twitter APIのラッパで、ログインしているTwitterアカウ
    ントの数だけインスタンスがあります。なので、プラグインがこのクラス
    をnewすることはありません。
    
    これを書いている現在では、mikutterは１つのアカウントでしかサインアッ
    プできないということになっているので、以下のように書けば、Serviceクラ
    スを得ることができると覚えておいてください。
    
        Service.primary
    
    このメソッドは、現在アクティブな(選択されている)アカウントのServiceを返します。マルチアカウント下ではその時々によって返す値が変わる可能性があるので、注意してください。

2.  update

    Service#update で、ツイートが投稿されます。
    
        Service.primary.update(:message => "ツイートの本文")
    
    ツイートの投稿は非同期で行われるので、処理がここで止まるというこ
    とはありません。

### Reserver 予約実行<a id="sec-2-2-3" name="sec-2-2-3"></a>

特定の時間に処理を実行したいことがあります。今回の場合だと、毎日
0:00:00に、よるほーとつぶやくコードを実行したいわけです。そういう
時は、mikutterが提供しているReserverを使います。

String, Time, Integerのいずれかの値を渡します。

Stringの場合はTime.parse()された値が使われ、Timeを渡すと、その時刻
になったら実行します。過去の時刻を渡すとすぐに実行されます。

    Reserver.new("10:00"){ 朝10時に実行 }

Integerの場合は、その秒数待ってから実行します。

    Reserver.new(30){ 30秒後に実行 }
    Reserver.new(HYDE){ 156秒後に実行 }

今のところ、Reserverのブロックは、メインスレッド **以外** のスレッドで実
行されます。非同期処理によるバグには気をつけてください。

## まとめ<a id="sec-2-3" name="sec-2-3"></a>

ReserverとService#updateについて学びました。
Service.primary.update() でツイートの投稿ができます。
Reserverは特定の時間に処理を実行するためのクラスです。
