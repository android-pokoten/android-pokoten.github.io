---
toc: true
title:  "TransBook Mini T103HAF のタッチパネルは MPP"
date:   2025-02-10 00:00:00 +0900
categories: デバイス
tags:
- T103HAF
- MPP
- ubuntu
---
# 背景
[TransBook Mini T103HAF][t103haf] を使用しています。モデルによってはペンが付属しているようですが、自分が購入したモデルには付属していませんでした。とはいえ、指でタッチするものの代替くらいのイメージだったので、あまり必要もないと思っていたのですが、どうやら T103HAF のタッチパネルは MPP (Microsoft Pen Protocol) 対応で 1024 段階の筆圧感知に対応しているようです。そのため、ASUS 製のペンでなくても、MPP 対応のペンが使用できます。ただ、MPP は 2.0 も登場していて、そちらは 4096段階の筆圧検知とパームリジェクションにも対応するようですが、T103HAF は MPP1.0 の対応となります。

MPP 対応のペンを購入し、Windows10 では問題なく使用できていました。お絵かきソフトを使って、筆圧検知させることも可能です。(もちろん性能が低いので、快適ではないですが・・・)

しかし、ハードウェア要件が Windows11 に対応していないですし、Windows10 でも動作が遅い気がするので、Linux に入れ替えてみて、その状態でも MPP のペンが正常に使えるか試してみました。

# Ubuntu 24.04
使用するディストリビューションは Ubuntu 24.04 にしました。いったんデフォルトのフレーバーを使いますが、これはこれで負荷が高めなので、問題なくタッチパネルが使えることを確認したら、別のフレーバーも試してみようと思っています。

インストール自体は問題なく完了しました。追加のハードウェアドライバも特段不要です。肝心のタッチパネルですが、マウスカーソルや指での操作は問題ないものの、MPP ペンで操作すると、カーソルの位置があいません。90° ずれた動きをするので、画面の回転に追従できていないような状態です。

## Wayland
結論から言うと、Ubuntu 24.04 標準の Wayland だと改善は (少なくとも 2025年2月時点では) 難しい様子です。タッチパネルを画面の回転に追従させるには、xinput コマンドを使うようですが、これでデバイス一覧を表示する `xinput list` コマンドを実行した結果が以下の通り。

![xinput][img02]

この状態で設定をしてみようとすると、

![warning][img03]

`WARNING: running xinput against an Xwayland server.` と表示されます。そして、この状態だと設定を変更しても効果がありません。

ではどうするか？ というと、Xorg に変更することで設定を反映させることが可能です。Xorg に変更するには、いったんログアウトしてログイン画面に戻り、ログインするユーザーを選択した状態で画面右下の歯車アイコンをクリックすると、セッションを選択するメニューが表示されます。

![session selection][img04]

「Ubuntu」は標準の Wayland、「Ubuntu on Xorg」にすると Xorg になります。なので、Ubuntu on Xorg を選択してログインすると、Xorg を使用することになります。

## Xorg
Xorg だと、`xinput list` コマンドの結果はこのようになります。

![xinput on xorg][img05]

この状態であれば、

``` bash
xinput set-prop "ELAN2562:00 04F3:2562 Stylus pen (0)" "Coordinate Transformation Matrix" 0 1 0 -1 0 1 0 0 1
```

とコマンドを実行すれば、MPP ペンの入力が画面 (横画面にした場合) に一致します。筆圧感知も正常に動作していることを確認しました。

一応、手元の端末では以下のコマンドで各回転方向と MPP の入力を一致させることができました。

* 横方向 (キーボードと接続して表示させる状態)
``` bash
xinput set-prop "ELAN2562:00 04F3:2562 Stylus pen (0)" "Coordinate Transformation Matrix" 0 1 0 -1 0 1 0 0 1
```

* 逆方向 (横方向の上下反転)
``` bash
xinput set-prop "ELAN2562:00 04F3:2562 Stylus pen (0)" "Coordinate Transformation Matrix" 0 -1 1 1 0 0 0 0 1
```

* 左方向 (USB ポートが上になる状態)
``` bash
xinput set-prop "ELAN2562:00 04F3:2562 Stylus pen (0)" "Coordinate Transformation Matrix" 1 0 0 0 1 0 0 0 1
```

* 右方向 (HDMI ポートが上になる状態)
``` bash
xinput set-prop "ELAN2562:00 04F3:2562 Stylus pen (0)" "Coordinate Transformation Matrix" -1 0 1 0 -1 1 0 0 1
```

コマンドで入力すると長いので、スクリプトで書いておけば画面を回転してもタッチパネルは使用できそうです。


[t103haf]:https://www.asus.com/jp/laptops/for-home/everyday-use/asus-transformer-mini-t103/

[img01]:/assets/images/2025/02/ss-20250208-01.png
[img02]:/assets/images/2025/02/ss-20250208-02.png
[img03]:/assets/images/2025/02/ss-20250208-03.png
[img04]:/assets/images/2025/02/ss-20250208-04.jpg
[img05]:/assets/images/2025/02/ss-20250208-05.png
