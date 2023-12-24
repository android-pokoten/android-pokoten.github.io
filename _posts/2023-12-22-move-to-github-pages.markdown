---
layout: post
title:  "Blogger から GitHub Pages へ移行します"
date:   2023-12-22 07:00:08 +0900
categories: jekyll github ブログ docker
---
### 背景
[Blogger][pokotenote] に不満があるわけではないのですが、コードを整形して表示するのが面倒でした。そして、ここにきて突然その表示がされなくなり、どうやっても元通りにならないというのもきっかけです。
Qiita や Zenn も考えたのですが、技術寄りではない投稿も結構あるので、シンプルに静的なページを公開できる (かつ無料で) GitHub Pages を使ってみることにしました。

### 実行
GitHub Pages は、HTML を配置すればそのまま公開されますが、静的サイトジェネレーターの jekyll が組み込まれている (こういう表現が適切かどうかはわかりませんが) ので、jekyll に対応した記述をすれば自動的にページが生成されます。ブログのように、順次投稿が増えていくサイトを作るにはちょうどいいですね。

なお、jekyll は ruby によって記述されています。ruby は使ったことがありませんが、あくまでツールを使うだけで ruby のコードを書いたりするわけではないだろう、と考えています。といっても、ruby の実行環境を整えるのが大変そうだったので、こちらも docker を使うことにします。

まずは作業用のフォルダ構成です。今回は work/jekyll というフォルダを作ります。そこへ移動し、
{% highlight bash %}
>$ git clone git@github.com:android-pokoten/android-pokoten.github.io.git
{% endhighlight %}
でリポジトリをクローンします。

この状態で、`work/jekyll`直下に`docker-compose.yml`ファイルとして以下の内容を記述します。

{% highlight yaml %}
version: "3"
services:
  service_jekyll:
    image: jekyll/jekyll:pages
    container_name: local_jekyll
    volumes:
      - ./android-pokoten.github.io:/srv/jekyll
      - ./scripts:/tmp/scripts
      - /usr/share/zoneinfo/Asia/Tokyo:/etc/localtime
    command: sh -c "/tmp/scripts/jekyll-init.sh && jekyll serve --watch --verbose --trace"
    environment:
      TZ: "Asia/Tokyo"
    ports:
      - "4000:4000"
{% endhighlight %}

さらに、`scripts`ディレクトリを作り、コンテナの初回起動のときだけ実行されるようにスクリプトを`scripts/jekyll-init.sh`として用意します。
この処理は 2 回目以降のコンテナ実行時には実行されてほしくないので、check ファイルを作っておいてファイルが存在するときはスキップするようにしています。

{% highlight bash %}
if [ ! -e '/check' ]; then
    echo "initializing..."
    jekyll new --skip-bundle .
    apk update
    apk add alpine-sdk ruby-dev linux-headers
    patch -p1 < /tmp/scripts/jekyll.patch 
    touch /check
fi
{% endhighlight %}

jekyll の設定を一部変更する必要があるので、パッチをあてます。「scripts/jekyll.patch」として以下内容を用意します。

{% highlight patch %}
diff -up old/Gemfile new/Gemfile
--- old/Gemfile	2023-12-24 10:28:12.820095018 +0900
+++ new/Gemfile	2023-12-24 10:25:15.250092428 +0900
@@ -8,14 +8,14 @@ source "https://rubygems.org"
 #
 # This will help ensure the proper Jekyll version is running.
 # Happy Jekylling!
-gem "jekyll", "~> 3.9.2"
+#gem "jekyll", "~> 3.9.2"
 
 # This is the default theme for new Jekyll sites. You may change this to anything you like.
 gem "minima", "~> 2.0"
 
 # If you want to use GitHub Pages, remove the "gem "jekyll"" above and
 # uncomment the line below. To upgrade, run `bundle update github-pages`.
-# gem "github-pages", group: :jekyll_plugins
+gem "github-pages", group: :jekyll_plugins
 
 # If you have any plugins, put them here!
 group :jekyll_plugins do
@@ -39,3 +39,5 @@ gem "kramdown-parser-gfm"
 # Lock `http_parser.rb` gem to `v0.6.x` on JRuby builds since newer versions of the gem
 # do not have a Java counterpart.
 gem "http_parser.rb", "~> 0.6.0", :platforms => [:jruby]
+
+gem "webrick"
diff -up old/_config.yml new/_config.yml
--- old/_config.yml	2023-12-24 10:28:15.980093908 +0900
+++ new/_config.yml	2023-12-24 10:25:15.250092428 +0900
@@ -29,6 +29,7 @@ markdown: kramdown
 theme: minima
 plugins:
   - jekyll-feed
+  - webrick
 
 # Exclude from processing.
 # The following items will not be processed, by default. Create a custom list
{% endhighlight %}

これで、以下のようなファイル、ディレクトリ構成になっていると思います。

{% highlight bash %}
work/
  jekyll/
    android-pokoten.github.io/
      .git/
    scripts/
      jekyll-init.sh
      jekyll.patch
    docker-compose.yml
{% endhighlight %}

この状態で`work/jekyll`直下に移動し`docker-compose up -d` を実行します。しばらく待って`http://localhost:4000`へアクセスすると、
![](/assets/images/ss_20231224.png)

(少し編集後の画面しか残してませんでした・・・)

うまくいかない場合は、`docker-compose logs -f`でログを見てつぶしていく感じですね。ruby のバージョン違いだからか、webrick を追加しないとエラーになったり、サイトの初期化が必要だったりと少し手間がかかりましたが、とりあえず docker-compose でプレビュー環境を立ち上げられるようになったのは一安心です。

初期設定用のスクリプトで、`work/jekyll/android-pokoten.github.io`直下へ jekyll の初期ファイルが配置されます。そのファイルを順次編集して動作確認し、問題なければ github へファイルをアップロードします。

jekyll はファイルの変更を自動的に反映してくれます。ただ、`_config`を編集した場合の反映は、jekyll の再起動が必要かもしれません。これは、
{% highlight bash %}
>$ docker-compose restart
{% endhighlight %}
と実行します。ブログの更新をしないときは、`docker-compose stop`で止めておくことも可能です。

github へアップロードする際に、`_site`配下のファイルはアップロードする必要はありません。jekyll が自動的に実行されるようです。(なので、jekyll が生成する .gitignore ファイルで`_site`を指定しています)
{% highlight bash %}
>$ git puth -u origin HEAD
{% endhighlight %}

1分くらい待ってから [github.io][github.io] を開くと、jekyll で生成されたページが表示されると思います。

[pokotenote]: https://pokotenote.blogspot.com/
[github.io]: https://android-pokoten.github.io/

