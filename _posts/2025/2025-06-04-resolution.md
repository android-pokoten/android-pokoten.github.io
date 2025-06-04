---
toc: true
title:  "FramePack の生成速度向上"
date:   2025-06-04 09:00:00 +0900
categories: プログラム
tags:
- python
- 機械学習
- ComfyUI
- FramePack
---
[TeaCache を使って FramePack の生成速度向上][teacache] で書きましたが、TeaCache を使用しても 2 秒生成するのに 50 分かかる状況でした。パラメータを調整することで、画質は落ちてもいいので速度を向上させることができないか試してみます。もちろん前提は RT2070 を使用した場合なので、他の GPU を使用している場合は結果が異なる場合もあると思います。

# ステップ数を減らす
FramePackSampler の「steps」を、初期値の 30 から半分の 15 にしてみます。

![steps15][img01]

これで出力してみると 1928 秒。30 分近くまで短縮しました。ステップ数が半分で生成時間が 60% 程度に短縮したので、おおむね短縮の効果はでているとおもいますが、もう少し短縮したいところです。出力される動画には、それほど違いを感じませんでした。(絵柄は違いますが、生成時間はほとんど変化はないので今回はこちらの画像を使用しています)

![framepack-steps15][img06]

<video controls playsinline width="640">
  <source src="/assets/images/2025/06/FramePack_640.mp4" type="video/mp4">
  お使いのブラウザは動画タグをサポートしていません。
</video>



# 出力解像度を減らす
![resolution][img02]

Find Nearest Bucket ノードの「base_resolution」を変更すると、出力する動画の解像度を変えることができます。初期値は 640 でした。「base_resolution」に近い解像度に合わせて画像をリサイズするような動作になるので、元画像のアスペクト比は変わらず、解像度が指定した値に近いところになります。

解像度が減ればそれだけ処理量が減るので、生成時間の短縮が期待できると思います。なお、元画像は 1024x1024 の解像度。また、TeaCache オン、ステップ数は 15 のままで生成しています。

## base_resolution = 480
「base_resolution」を `480` にしてみると、生成時間は 881 秒。15 分程度に短縮されました。

![480][img03]

生成された動画はこんな感じ。画質は落ちている感じを受けますが、それほど気になりません。ただ、意図した動作にはならず、手が片方しか動いていないです。

<video controls playsinline width="640">
  <source src="/assets/images/2025/06/FramePack_480.mp4" type="video/mp4">
  お使いのブラウザは動画タグをサポートしていません。
</video>

## base_resolution = 320
さらに減らして「base_resolution」を `320` にしてみると、生成時間は 389 秒。11 分程度です。

![320][img04]

生成された動画はこんな感じ。明らかに画質は落ちています。こちらも、手が片方しか動かないですし、かなり描写も怪しい感じになっています。

<video controls playsinline width="640">
  <source src="/assets/images/2025/06/FramePack_320.mp4" type="video/mp4">
  お使いのブラウザは動画タグをサポートしていません。
</video>

## base_resolution = 240
さらに減らして「base_resolution」を `240` にしてみると 377 秒まで減りました。

![240][img05]

画質も動きの質もかなり低下しますね。生成時間もあまり減らないですし、ここまで減らすとデメリットのほうが大きくなる気がします。

<video controls playsinline width="640">
  <source src="/assets/images/2025/06/FramePack_240.mp4" type="video/mp4">
  お使いのブラウザは動画タグをサポートしていません。
</video>

## Image Resize の method
base_resolution を減らすと生成時間の短縮には効果がありますが、画像の一部しか動かせないような挙動になります。そこで、「Image Resize」ノードの「method」を初期値の `strech` から `fill / crop` に変更したところ、解像度を下げる前と同じような動画が生成されました。(左手の残像がありますが・・・)

![fill/crop][img07]

<video controls playsinline width="640">
  <source src="/assets/images/2025/06/FramePack_FillCrop.mp4" type="video/mp4">
  お使いのブラウザは動画タグをサポートしていません。
</video>

# まとめ
steps を 15、base_resolution を 480 や 320 くらいにして、Image Resize の method を fill / crop に変更した状態でプロンプトなどを試行錯誤して、ある程度見えてきたら steps や base_resolution を大きくして出力をよくする、ということができそうな処理時間も見えてきました。(それでも 2 秒生成するのに 10 分以上かかりますけども・・・)



[framepack]:{% post_url 2025/2025-05-13-framepack %}
[teacache]:[% post_url 2025/2025-05-19-teachace %]

[img01]:/assets/images/2025/06/ss-20250604-01.png
[img02]:/assets/images/2025/06/ss-20250604-02.png
[img03]:/assets/images/2025/06/ss-20250604-03.png
[img04]:/assets/images/2025/06/ss-20250604-04.png
[img05]:/assets/images/2025/06/ss-20250604-05.png
[img06]:/assets/images/2025/06/ss-20250604-06.png
[img07]:/assets/images/2025/06/ss-20250604-07.png

[img01m]:/assets/images/2025/06/FramePack_640.mp4
[img03m]:/assets/images/2025/06/FramePack_480.mp4
[img04m]:/assets/images/2025/06/FramePack_320.mp4
[img05m]:/assets/images/2025/06/FramePack_240.mp4
[img07m]:/assets/images/2025/06/FramePack_FillCrop.mp4

