---
layout: post
title:  "システムメッセージの利用"
date:   2016-09-05 00:00:00 +0900
categories:
- example
---
# システムメッセージの利用<a id="sec-6" name="sec-6"></a>

mikutterを使っていると、mikutter\_botというアカウントのツイートとしてシステムメッセージがホームTLに入ることがあります。
このセクションでは、このメッセージを使って、ユーザに情報を提示する方法について学習します。

## コード<a id="sec-6-1" name="sec-6-1"></a>

```ruby
# -*- coding: utf-8 -*-

Plugin.create(:time_signal) do

  defactivity "hour_signal", "時報"

  now = Time.new
  time = Time.mktime now.year, now.mon, now.day, now.hour

  def next_hour(time)
    time += 3600
    notice "next hour #{time}"
    Reserver.new(time) {
      activity :hour_signal, "#{time.hour} 時です"
      next_hour(time)
    }
  end
  next_hour(time)
end
```

## 解説<a id="sec-6-2" name="sec-6-2"></a>

### アクティビティとは<a id="sec-6-2-1" name="sec-6-2-1"></a>

アクティビティは、mikutter 0.2から追加された仕組みで、mikutter上やTwitter上で起こった通知情報を統合管理する仕組みです。

![img](activity.png)

まず、何らかのプラグインがactivityメソッドを使って通知を発生させます。その通知はactivityプラグインが受け取り、 **:modify\_activity** というイベントを発生させます。このイベントを受け取って、ホームTLやアクティビティタブに通知が表示されるのです。
アクティビティとアクティビティタブは異なります。アクティビティは通知の仕組みのことで、アクティビティタブは表示手段の一つです（上の図を参照してください）。

### 通知の種類<a id="sec-6-2-2" name="sec-6-2-2"></a>

通知には、お気に入り、DM、エラー通知といった種類があります。今回は時報のための **hour\_signal** という種類を新しく定義しています。

    defactivity "hour_signal", "時報"

とはいっても、普通は最初から用意されている system という種類の通知を使うので、その場合は **defactivity** を使う必要はないです。

### 実際に通知を発生させる<a id="sec-6-2-3" name="sec-6-2-3"></a>

通知を発生させるのはごく簡単です。例のコードでは、以下の部分です。

    activity :hour_signal, "#{time.hour} 時です"

これで、「x 時です」と、毎時0分に通知されるようになります。しかしこのプラグインを実際に使うと、タイムラインには表示されず、アクティビティタブにしか表示されません。理由は簡単で、「設定」の「アクティビティ」で、時報をTLに表示する設定がデフォルトで無効になっているからです。設定でこのチェックを入れるとTLに表示されるようになります。

どうしてもホームTLに表示したい場合があるでしょう。その場合、Plugin.callで無理やりシステムメッセージを挿入する古い方法の代わりに、 **system** 通知を使いましょう。これは、種類を分けるまでもない一般的なシステムメッセージのために予約されており、デフォルトでTLに表示されるようになっています。もちろんユーザがTLに表示しないように設定できるので、プラグインがアクティビティの仕組みを使えば、ユーザは通知をミュートすることができます。
