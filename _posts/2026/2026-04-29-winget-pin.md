---
toc: true
title:  "winget でバージョン固定"
date:   2026-04-29 00:00:00 +0900
categories: トラブル
tags:
- Windows
- winget
---
### 事象
Clip Studio をインストールしている PC で `winget upgrade` をすると、Clip Studio のバージョンアップが表示されるようになっていました。

{% include thumb.html path="2026/04/ss-20260429-01.png" alt="Clip Studio 5のバージョンアップ" %}

しかし、所有しているライセンスが永続版なので、メジャーバージョンアップは別途費用が必要となります。案の定、winget コマンドでバージョンアップしてしまったら、ライセンスがないため起動できなくなりました。(Clip Studio を起動しようとするとバージョンダウンを案内されるので、その指示通りに操作すればライセンスが有効なバージョンに置き換えることはできました)

ダウングレードはできますが毎回それを行うのは面倒ですし、とはいえこのままだと `winget upgrade` するたびに Clip Studio のバージョンアップが表示されてしまうので、何とかできないかと考えました。

### 対策
winget にはバージョンのピン留めという機能があるようです。これを使うことで、現在のバージョンで固定することができるようです。

### 詳細
Clip Studio をピン留めするアプリとして登録するには、

``` terminal
winget pin add --id Celsys.ClipStudioPaint
```

と `pin add` を使います。その状態で `winget upgrade` すると Clip Studio のバージョンアップは表示されなくなり、「アップグレードを止めるピンがある」メッセージが表示されるようになります。

{% include thumb.html path="2026/04/ss-20260429-02.png" alt="アップグレードのピン留め" %}

ピン留めしたアプリは 

``` terminal
winget pin list
```

で一覧表示できます。

{% include thumb.html path="2026/04/ss-20260429-03.png" alt="ピン留めリスト" %}

バージョンを固定したいだけでなく、インストールすると自動的に winget の管理下になってしまうソフトを winget で管理させないようにするためにも、ピン留めが使えそうな気がします。
