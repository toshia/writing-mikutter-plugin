---
layout: post
title:  "プロフィール・投稿詳細にタブを追加する"
date:   2016-05-22 00:00:00 +0900
categories:
  - twitter-client
---

プロフィールやツイートの詳細タブの中には、いくつかタブがあります。

このタブをプラグインから追加できると、プロフィールに追加情報を表示できますね。

## fragment
プロフィール、ツイート詳細のタブはそれぞれ、次のプラグインによって追加されるものです。

| function   | Plugin name           |
|------------|-----------------------|
| プロフィール | user\_detail\_view    |
| ツイート詳細 | message\_detail\_view |

user\_detail\_view および message\_detail\_view が作ったタブの中のタブのことを **fragment** と呼びます。そして、fragmentは別のプラグインから追加できるので、fragmentの作り方を学べば、今回の目的は達成できそうです。

## fragmentを追加するDSL
### user\_detail\_viewの場合

### message\_detail\_viewの場合
