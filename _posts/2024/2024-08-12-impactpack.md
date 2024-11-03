---
toc: true
title:  "ComfyUI で顔など画像の一部を再生成する [ltdrdata/ComfyUI-Impact-Pack]"
date:   2024-08-12 09:00:00 +0900
categories: プログラム
tags:
- python
- 機械学習
- StableAudioOpen
- ComfyUI
- CustomNodes
---
ComfyUI の便利なカスタムノード、Impact Pack を紹介します。いろいろな機能を持っているカスタムノードですが、その中でも顔などの画像の特定のエリアの検出や、検出した箇所を新たに生成して置き換えることが可能になります。

同じような機能として「インペインティング」がありますが、インペインティングは画像全体を再生成します。一方で Impack Pack の場合は再生成する箇所のみを生成するので、生成にかかる時間が短くて済むという利点があります。

### ComfyUI 環境準備
[Stable Diffusion 3 Medium を実行する環境を ComfyUI で用意した手順][comfyui] を実行し、コンテナで ComfyUI を実行できるようにします。

### 起動 & Custom Node 導入
ComfyUI のディレクトリで以下コマンドを実行し、起動します。

{% highlight bash %}
python3 main.py
{% endhighlight %}

ブラウザで UI を開いたら、右下の `Manager` をクリックします。

![ui][img-1]

ComfyUI Manager Menu が開くので、`Custom Node Manager` をクリックします。

![customnode][img-2]

上部の「Search」欄に `impact` と入力します。すると `Comfyui Impact Pack` があるので、そこの `Install` をクリックします。同時に `Comfyui Inspire Pack` があるので、こちらも `Install` しておきます。

※Inspire Pack は今回は使用しませんが、今後使用する際は改めて紹介する予定です

![impacktpack][img01]

しばらく待つと「Restart Required」と表示されるので、`Restart` ボタンをクリックします。あわせて、ブラウザの再読み込みを行い、画面を更新します。

![restart][img02]

### 生成
ComfyUI を起動したら、メニューの「Clear」などでワークフローを初期化します。空白部分でダブルクリックし、検索画面を出したら `detector` と検索します。すると `UltralycsDetectorProvider` があるので、それをクリックします。

![UltralyticsDetectorProvider][img03]

さらに検索画面を出して `simple detector` と検索し、`Simple Detector (SEGS)` をクリックします。

![simple detector][img04]

続けて検索画面を出して `saml` と検索し、`SAMLoader (Impact)` をクリックします。

![samloder][img05]

さらに検索画面を出して `segspre` と検索し、`SEGSPreview` をクリックします。

![preview][img06]

ここまで来たら、以下の画面のように各ノードを接続します。UltralyticsDetectorProvider と SAMLoader にモデルが必要ですが、Impact Pack をインストールしたときに合わせて導入されたと思います。もし不足している場合は別途ダウンロードして所定のパスに配置してください。

(ComfyUI Manager からダウンロードできると思います)

「Queue Prompt」をすると、「SEGSPreview」に検出されたエリアのみ明るく残った画像が表示されます。(今回は顔を検出している)

ここで意図しない検出になっている場合は、Simple Detector のパラメータを調整する必要があると思います。今回は、多少髪の毛も検出されていますが、おおむね OK としてこのまま進めます。

![test run][img08]


検出がうまくいったら、次は検出した箇所を再生成します。再度検索画面を出して、`detailer` と検索し、`Detailer (SEGS)` をクリックします。

![detailer][img09]

Detailer (SEGS) にはいろいろな入力が必要ですが、基本的には KSampler とほぼ同じです。そのため、`Load Checkpoint` でモデルを、`CLIP Text Encoder (Prompt)` でポジティブ・ネガティブプロンプトを入力します。`image` は元画像を、`segs` は `Simpler Detector (SEGS)` を接続します。

![detailer][img10]

最終的なワークフロー全体です。これで画像を生成したあとからでも、「表情だけ変えたい」や「顔だけ指定のキャラクターにする」といったことが可能になります。[ComfyUI で LoRA を適用する][lora] のように、Load LoRA ノードを使えば LoRA を適用して再生成することも可能です。

![workflow][img11]

また、顔ではなく手を検出させることも可能です。以下のように、UltralyticsDetectorProvider のモデルで「bbox/hand_yolov8s.pt」を選択すると、赤丸で示した通り手の部分を検出しています。指はどうしても破綻しやすいので、そこだけ修正したい、という要望にも対応しやすそうですね。

![hand][img12]


[comfyui]:{% post_url 2024/2024-06-20-comfyui %}
[impactpack]:https://github.com/ltdrdata/ComfyUI-Impact-Pack
[lora]:{% post_url 2024/2024-08-11-lora %}

[img-1]:/assets/images/2024/06/ss-20240620-02.png
[img-2]:/assets/images/2024/07/ss-20240714-01.png
[img01]:/assets/images/2024/08/ss-20240812-01.png
[img02]:/assets/images/2024/08/ss-20240812-02.png
[img03]:/assets/images/2024/08/ss-20240812-03.png
[img04]:/assets/images/2024/08/ss-20240812-04.png
[img05]:/assets/images/2024/08/ss-20240812-05.png
[img06]:/assets/images/2024/08/ss-20240812-06.png
[img08]:/assets/images/2024/08/ss-20240812-08.png
[img09]:/assets/images/2024/08/ss-20240812-09.png
[img10]:/assets/images/2024/08/ss-20240812-10.png
[img11]:/assets/images/2024/08/ss-20240812-11.png
[img12]:/assets/images/2024/08/ss-20240812-12.png

