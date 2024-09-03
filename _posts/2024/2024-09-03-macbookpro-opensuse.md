---
layout: post
title:  "MacBook Pro (Retina, 13-inch, Late 2012) に OpenSUSE Tumbleweed をインストール"
date:   2024-09-03 09:00:08 +0900
categories: デバイス
tags:
- MacBook
- OpenSUSE
---
### 購入
ジャンク扱いで表題の MacBook を安く手に入れることができました。A1425、EMC2632 という型番です。動作未確認品での購入でしたが、分解して確認したところ、問題になりそうなのはバッテリーが膨張しているのと SSD が無い点ぐらい。ディスプレイは問題なさそうだったので、バッテリーを交換して使用してみることにします。

### 修理
バッテリー交換の様子を YouTube にアップしました。

<iframe width="560" height="315" src="https://www.youtube.com/embed/jbYJDPJ1m8s?si=XLBGEa5cB4ue-hpY" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>


### ハードウェア
[MacBook Pro (Retina, 13-inch, Late 2012) - 技術仕様][a1425]

13 インチで 2,560x1,600 ピクセルはすごいですね。ブラウザを開いているだけでも情報量の違いに驚きます。

メモリはオンボードで増設・交換不可。チップを張り替えるようなことをすれば何とかならないこともなさそうですが、一般的には 8GB で固定と思っておいてよさそうです。

SSD。2.5 インチのディスクを収めるスペースはありそうな気もしますけどね。しかも、ただの M.2 ではない独自コネクタ。さらに、MacBook の中でも年代によってコネクタの形状が異なるという厄介者。機能的には NVMe と同じなので、コネクタ形状さえ変換すれば一般的な SSD が使えるものの、SSD によっては相性だったりドライバが対応しない等あって注意が必要です。

![ssd][img01]

Thunderbolt ポート。Thunderbolt 3 になると USB Type-C と互換性があるので便利ですが、この場合は mini DisplayPort として外部ディスプレイに出せるくらい？ 純正の Thunderbolt - FireWire アダプタと Thunderbolt - Ethernet アダプタはあるようですが、Thunderbolt - USB アダプタが個人的には欲しいです。

![thunderbolt][img02]

キーボードは、文字キーはかわらないものの、周辺のキーが全然違います。「Ctrl」キー (表記は「control」) が左横にあり (青丸)、「Alt」は「option」、「Windows キー」は「command」と表記されています (緑丸)。「delete」と表記されているキーは「BackSpace」キーの動作で、「Delete」キーに相当するキーはなく (黄丸)、「PageUp/Down」「Home/End」は Fn キーとカーソルキーの同時押しで実現します (赤丸)。これは Windows PC でもよくある組み合わせですが、印字がないので最初はわかりませんでした。

![keyboard][img04]

### OS
macOS は、当該機種はすでに最新バージョンの対象から外れているようなので却下。

Windows も Windows11 はハードウェアがサポート外、Windows10 はもうサポート終了が見えてきているので避けたいところ。

となると Linux ですが、今だと ChromeOS Flex という選択肢もあります。ChromeOS は Linux 環境も使えますが、せっかくの高解像度ディスプレイを生かしたいと思うと、ChromeOS だけでは満足しない気がするので、そうなると直接 Linux を入れた方が都合がいいだろうと考えました。

Linux だと定番は Ubuntu ですが、Ubuntu のインストールメディアが手元になく、たまたま OpenSUSE のインストールメディアがあったので、それをインストールすることにしました。

なので、特段 OpenSUSE にしたい理由もないので、何かあれば他に乗り換えることも考えつつ、OpenSUSE で環境を整えていきます。

#### OpenSUSE 
#### 無線 LAN のドライバ
最初に大きな壁です。MacBook の Wi-Fi アダプタは Broadcom 製です。基本的にディストリビューションに同梱はされません。(正確にはドライバは同梱されているものの、ファームウェア等を追加しないと動作しない状態)

カーネルアップデートのたびに更新が必要なので、リポジトリを登録して自動更新できるようにしておきます。

まず、[OpenSUSE の broadcom-wl のサイト][broadcom-wl] を開きます。

![broadcom-wl][img05]

少し下へスクロールすると、ディストリビューションごとのリンクがあるので、今回は Tumbleweed の 「home:Sauerland」を使います。「1クリックインストール」を右クリックしてリンク先を保存します。

![download][img06]

Dolphin (ファイルマネージャ) でダウンロードしたファイル「broadcom-wl.ymp」を探し、右クリックから「YaST 1-click Install で開く」をクリックします。

![install][img07]

YaST2 の画面が開くので、そのまま「次へ」をクリックします。

![yast2][img08]

念のため、追加するリポジトリに間違いがないか確認して「次へ」をクリックします。

![confirm][img09]

処理が行われて問題がなければインストールに成功した旨、表示されるので、「完了」をクリックします。

![complete][img10]

続けて、ドライバパッケージをインストールします。メニューから YaST を開き、「ソフトウェア管理」をクリックします。

![control center][img11]

検索欄に `broadcom` と入力し検索すると「broadcom-wl」パッケージが表示されるので、チェックをつけてインストールします。

![broadcom][img12]


再起動しなくても認識したと記憶していますが、念のため再起動したほうがいいと思います。

#### 日本語入力
特段の設定をすることなく、「かな」で日本語入力、「英数」で直接入力を切り替えらるようになっていました。「半角/全角キーはどこだ？」と半日悩みました・・・(上のキーボードの写真参照)。結果的に、MacBook Pro ではキーボード横のキーで切り替えるようにしています。

#### F1〜F12 キー
最近のノート PC は、基本的に F1〜F12 はマルチメディアキーが優先になっていて、ファンクションキーで使いたい場合は Fn キーと同時押しですね。しかし、日本語入力していると、どうしても F7 や F10 は使いますし、ブラウザのリロードで F5 もとっさに手が動きます。

ThinkPad や HP は、Fn キーと何かを同時に押すことで Fn キーをロックすることができるのですが、MacBook の場合は OS から設定する方法しか見つかりません。検索した結果、

{% highlight bash %}
echo 0x02 > /sys/module/hid_apple/parameters/fnmode
{% endhighlight %}

と実行することで Fn ロック状態にすることが可能でした。ただ、これは再起動すると設定が戻ってしまいます。/etc/modprobe.d/ 以下に hid_apple.conf のようなファイル名 (ファイル名は任意ですが、拡張子として .conf をつけないと有効になりません) でテキストファイルを作り、

{% highlight bash %}
options hid_apple fnmode=2
{% endhighlight %}

と記載すればシステム起動時に読み込まれるようです。再起動しても反映されなかったような気がするのですが、2，3回再起動していたら問題なく起動時から Fn ロックされた状態になっています。

#### キーボードバックライト
特に設定等不要で動作していました。また、バックライトの明るさ調整も可能です。ディスプレイのバックライトと同じインターフェースで確認可能です。F5 と F6 のキーが、キーボードバックライトの調整ボタンです。

![backlight][img03]

#### サウンド
サウンドカードは「CS4206」です。しかし、音は出るのですがマイクが音を拾えません。なぜか、ヘッドホンジャックに接続したマイクの音も拾ってくれません。しかし、USB 接続のヘッドセットを接続すると、そっちのマイク入力は可能でしたので、おそらくオンボードのサウンドデバイスがうまく動作していない様子です。

こちらも検索した結果、snd_hda_intel のモデルを指定することで解決するようです。/etc/modprobe.d/ 以下に 50-sound.conf のようなファイル名 (ファイル名は任意ですが、拡張子として .conf をつけないと有効になりません) でテキストファイルを作り、

{% highlight bash %}
options snd_hda_intel model=mbp101
{% endhighlight %}

と記載すると、問題なくマイクも使用できるようになりました。

調べると「model=intel-mac-auto」と記載すると自動認識してくれるという記述もありましたが、それだと症状は改善しませんでした。

#### スクリーンショット
Mac のキーボードには、プリントスクリーンキーはないのですね・・・


### 感想
ディスプレイやキーボードには、それほど使い込んだ感じはなく、普段使いにも問題なさそうな感じです。

Ctrl キーの位置が慣れないのと、プリントスクリーンのキーがないのが不便ですが、ThnikPad と交互に使ってもまあ何とかなる程度。(キー入力している感覚は ThinkPad のキーボードの方が好みですが)

パフォーマンス的にも問題ないのですが、USB があと 1 つ多い方が便利だったかな、と思っています。


[a1425]:https://support.apple.com/ja-jp/118463
[broadcom-wl]:https://software.opensuse.org/package/broadcom-wl

[img01]:/assets/images/2024/09/ss-20240902-01.jpg
[img02]:/assets/images/2024/09/ss-20240902-02.jpg
[img03]:/assets/images/2024/09/ss-20240902-03.jpg
[img04]:/assets/images/2024/09/ss-20240902-04.jpg
[img05]:/assets/images/2024/09/ss-20240902-05.png
[img06]:/assets/images/2024/09/ss-20240902-06.png
[img07]:/assets/images/2024/09/ss-20240902-07.png
[img08]:/assets/images/2024/09/ss-20240902-08.png
[img09]:/assets/images/2024/09/ss-20240902-09.png
[img10]:/assets/images/2024/09/ss-20240902-10.png
[img11]:/assets/images/2024/09/ss-20240902-11.png
[img12]:/assets/images/2024/09/ss-20240902-12.png
