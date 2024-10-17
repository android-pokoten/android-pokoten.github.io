---
layout: post
title:  "Jekyll にカテゴリ一覧とタグ一覧のページを追加する"
date:   2024-10-17 07:00:08 +0900
categories: ブログ
tags:
- jekyll
- github
---
### 背景
ブログにカテゴリやタグを設定していましたが、それを表示していなかったので、まるで役に立っていませんでした。とりあえず、それぞれ一覧表示することができるページを作ってみました。

どのように表示するか迷いましたが、すでにいくつか項目を配置しているヘッダー直下に並べることにしました。

### カテゴリ一覧、タグ一覧
リポジトリのルート直下に `category.html` という名前でファイルを作成します。

{% highlight jekyll %}
{% raw %}
---
layout: page
title: カテゴリ一覧
permalink: /categories/
---
{% for category in site.categories %}
<article>
    {% capture category_name %}{{ category | first}}{% endcapture %}
    <h1 id="tag.{{ category_name }}">{{ category_name }}</h1>
    <ul>
        {% for post in site.categories[category_name] %}
        <li><a href="{{ post.url }}">{{ post.title }}</a></li>
        {% endfor %}
    </ul>
</article>
{% endfor %}
{% endraw %}
{% endhighlight %}

同様に `tag.html` という名前でファイルを作成します。

{% highlight jekyll %}
{% raw %}
---
layout: page
title: タグ一覧
permalink: /tags/
---
{% for tag in site.tags %}
<article>
    <h1 id="tag.{{ tag[0] }}">{{ tag[0] }}</h1>
    <ul>
        {% for post in tag[1] %}
        <li><a href="{{ post.url }}">{{ post.title }}</a></li>
        {% endfor %}
    </ul>
</article>
{% endfor %}
{% endraw %}
{% endhighlight %}


これで Jekyll を再起動すると、以下のようにヘッダーにメニューが追加されます。

![メニュー項目][img01]

「カテゴリ一覧」をクリックすると、以下のように一覧が表示されます。

![カテゴリ一覧][img02]

とはいえ、シンプルに一覧表示されるだけで、どんなカテゴリがあるのか、どんなタグがあるのかを探すのが難しいように感じます。特にタグは 1 つの記事に複数つけているので、数が多く、せめてソートぐらいはしてほしいように感じます。今後の改善ポイントですね。


### minima 設定更新
[minima の GitHub][minima] を見て、`_config.yml` で設定できる項目があることに気が付きました。

* 配色を変更
* 日付の表示形式を変更
* ソーシャルアカウントの記述方法を変更

日付形式は、違和感はあったものの、わからないわけでもないのでそのままにしてましたが、見慣れた形式のほうが便利だと感じますね。

記事名にカテゴリを表示したり、サイドバーにタグ一覧を表示したりできると、便利かもしれない等、今後の改善ポイントも出てきそうですね。


[minima]: https://github.com/jekyll/minima/blob/master/_config.yml

[img01]:/assets/images/2024/10/ss-20241017-01.png
[img02]:/assets/images/2024/10/ss-20241017-02.png
