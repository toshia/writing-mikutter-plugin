---
layout: post
title:  "プラグインの作成支援機能"
date:   2016-05-06 02:01:10 +0900
categories:
- common
---
mikutterにはプラグインの作成を支援する機能があります。mikutter.rbにはいくつかのコマンドラインオプションがあります。

```shell
$ mikutter.rb --help
command are:
generate [plugin_slug]       generate plugin template at ~/.mikutter/plugin/
spec [directory]             generate plugin spec. ex) mikutter spec ~/.mikutter/plugin/test
```

### 雛形の作成(generate)

プラグインを新しく作成するときに、以下の様なコマンドを実行すれば必要なディレクトリを作成し、ひな形ファイルを作成します。

```shell
$ mikutter.rb generate test_plugin
```

これで、 **~/.mikutter/plugin/test\_plugin/** ディレクトリが作成され、その中に **test\_plugin.rb** というファイルが作成されます。また、このファイルには最低限のプラグインのテンプレートが書かれています。

```ruby
# -*- coding: utf-8 -*-

Plugin.create(:test_plugin) do

end
```

### 定義ファイルの作成(spec)

mikutter 0.2からはプラグインに定義ファイルを持たせることが推奨されています。定義ファイルは、プラグインの説明、バージョン、作者、依存関係などの情報を含むファイルで、なければ不適切な環境でプラグインがロードされ、クラッシュするおそれがあります。書式を長々と説明するよりも、これも自動的に生成させることができます。

```shell
$ mikutter.rb spec ~/.mikutter/plugin/test_plugin/
```

引数には、プラグインのスラッグではなく、ディレクトリパスを与えることに注意してください。これを実行すると、対話的に２，３質問されるので、適当に答えましょう。すると、`~/.mikutter/plugin/test_plugin/.mikutter.yml` というspecファイルが生成されるはずです。
**test_plugin**は何も内容がないので、specファイルも見所がありません。みっくストア(<https://github.com/toshia/mikustore>)のspecファイルがこの方法で生成されているので、ちょっと見てみましょう。

```yaml
---
slug: :mikustore
depends:
  mikutter: 0.2.0.1051
  plugin:
    - settings
version: '0.1'
author: toshi_a
name: みっくストア
description: >-
  mikutterにプラグインのパッケージ管理機能を追加します。
  みっくストアに登録されているプラグインはクリックだけでダウンロードできます！
```

適切な情報が自動的にspecファイルに書き込まれました。もう一度、specファイルを作成するコマンドを実行してみましょう。今度は何も聞かれずにすぐにコマンドが終了してしまったはずです。というのも、対話プロンプトで入力されるような内容は、既に存在するspecファイルに書かれているからです。二度目以降は、依存関係などを更新するだけとなります。
