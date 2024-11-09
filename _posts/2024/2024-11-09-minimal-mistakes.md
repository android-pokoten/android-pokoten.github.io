---
toc: true
title:  "Jekyll のテーマを Minimal Mistakes に変更する"
date:   2024-11-09 09:00:00 +0900
categories: ブログ
tags:
- jekyll
- github
- Minimal Mistakes
---
### 背景
初期状態の Minima を使用していましたが、テーマを変更してみました。使用するテーマは [Minimal Mistakes][mistakes]。

ただ、テーマによって各記事の書き方が変わってしまう場合もあるようで、今回も各記事の Front Matter を変更する必要があったりと、ある程度使ってからテーマを変更しようとすると手間が増えます。可能であれば、最初にある程度テーマは決めておいたほうが良かったと感じました。

デフォルトの Minima テーマから移行して、機能の不足などは見当たらないので、このまま Minimal Mistakes を使ってみたいと考えています。

### 変更手順
基本的には Minimal Mistakes の GitHub にある、Installation 項の [Remote theme method][remote-theme-method] で実施します。具体的には、GitHub Pages のリポジトリのルートにある `Gemfile` に以下の行を追加します。

``` patch
+gem "jekyll-include-cache", group: :jekyll_plugins
```

同様に `_config.yml` を修正します。必須のプラグインである `jekyll-include-cache` を `plugins` に追加します。

``` patch
 plugins:
   - webrick
   - jekyll-feed
   - jekyll-remote-theme
   - jekyll-sitemap
   - jekyll-paginate
+  - jekyll-include-cache
```
`remote_theme` は `minima` から `minimal-mistakes` へ変更します。

``` patch
-remote_theme: jekyll/minima
+remote_theme: "mmistakes/minimal-mistakes@4.26.2"
```

また、`_config.yml` のファイル末尾に以下の行を追加します。これにより、各投稿の Front Matter で指定しなかった場合のデフォルト値を定義することができるので、一つ一つ記載する必要がなくなり便利です。

``` patch
+# Defaults
+defaults:
+  # _posts
+  - scope:
+      path: ""
+      type: posts
+    values:
+      layout: single
+      author_profile: true
+      read_time: true
+      comments: # true
+      share: true
+      related: true
```

そのうえで、各ポストの Front Matter から
`layout: post`
の行をを削除します。正確には、Minimal Mistakes では `layout: single` を指定しますが、_config.yml でデフォルトを指定するので、削除で OK です。

修正箇所はこれだけですが、ポストをすべて修正する必要があるので手間ですね。

_config.yml を修正しているので、一度 jekyll を再起動する必要があります。docker compose で管理していれば、`docker compose stop` で止めて `docker compose start` すれば以下の様にテーマが変更されるはずです。

![minimal mistakes][img01]

なお、プラグインでキャッシュを有効にしたからだと思いますが、今までは修正したらページを更新すれば修正後の表示に切り替わりましたが、うまく切り替わらない場合があります。その場合は、一度別のページへ移動してから再度リンクをクリックするか、Ctrl+F5 で更新することで最新の状態に切り替わると思います。

### 追加設定
Minimal Mistakes に変更して、`_config.yml` で設定する・できる項目があるため、現時点でわかる範囲で設定しています。

``` patch
+locale                   : "ja-JP"
+repository               : "android-pokoten/android-pokoten.github.io"
+enable_copy_code_button  : true
```
`locale` を `ja-JP` に設定し、`_data` ディレクトリに `ui-text.yml` ファイルを配置すると、UI のテキストが日本語表示になります。ただし、ui-text.yml に訳語の定義がないものは表示されないですし、ui-text.yml を修正すれば独自の表記にすることも可能だと思います。

`repository` はこれを追加しないとページの更新でエラーが発生したため、追加するようにしました。

`enable_copy_code_button` はコードハイライトのブロックにコードをコピーするボタンを表示するフラグです。ただ、このボタンは Markdown 記述のコードブロックでは有効ですが、Jekyll 記述のコードブロック ({% raw %} {% highlight %}〜{% endhighlight %} {% endraw %}) では表示されませんでした。

``` patch
+# Analytics
+analytics:
+  provider         : "google-gtag"
+  google:
+    tracking_id    : 
```

サイトのトラッキング情報です。`provider` は `google` だとうまくデータが取れなかったので、`google-gtag` に変更したところ、正常にデータが取れるようになりました。

``` patch
+# Site Author
+author:
+  name             : "Pokoten"
+  avatar           : "/assets/images/bio-photo.png"
+  bio              : "購入したデバイスのレビューや、PC、スマートフォンの不具合対処方法、Python や機械学習での試行錯誤をメモしていきます。ただし、解決方法は金銭的な負担が生じないモノを優先で。"
+  links:
+    - label: "GitHub"
```

Minima テーマではフッターなどに表示されていた内容が、Minimal Mistakes だと表示されないようだったので、一応見える場所に表示されるようにしてみました。

なお、記事ごとに作成者が異なる場合には、[ドキュメントの Authors ページ][authors] が参考になりそうですが、このブログは一人で作成しますので、`_config.yml` への記載にしました。


[mistakes]:https://mmistakes.github.io/minimal-mistakes/
[remote-theme-method]:https://github.com/mmistakes/minimal-mistakes/tree/master?tab=readme-ov-file#remote-theme-method
[authors]:https://mmistakes.github.io/minimal-mistakes/docs/authors/

[img01]:/assets/images/2024/11/ss-20241109-01.png
