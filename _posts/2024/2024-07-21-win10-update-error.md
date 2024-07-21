---
layout: post
title:  "Windows10 の Windows Update でダウンロードエラー(0x80070643)"
date:   2024-07-21 09:00:08 +0900
categories: トラブル
tags:
- windows10
- Windows Update
- 0x80070643
---
### 事象
Windows10 にて、Windows Update がエラーになることに気が付きました。それも、2024-01 の更新で KB5034441。状態はダウンロードエラー - 0x80070643 です。「再試行」をしても、PC を再起動しても症状は変わりません。

![Windows Update エラー][img01]

ただ、より新しい 2024-07 の更新はインストールに成功しています。最近の更新は累積更新なので、新しいものがインストールされれば古い更新も含まれると思っていましたが、KB5034441 についてはそうはいかないようです。

![新しい更新は OK][img02]

### 対策
以下 Microsoft のサイトの手順にそって回復パーティションのサイズを変更し、再度 Windows Update を実行したところ、改善しました。

[KB5028997: WinRE 更新プログラムをインストールするためにパーティションのサイズを手動で変更する手順][kb5028997]

### 詳細
どうやら回復パーティションの容量不足が原因のようです。そのため、OS がインストールされているパーティション (いわゆる C ドライブ) の容量を減らして、回復パーティションのサイズを確保する必要がありそうです。しかし、勝手にパーティションを縮小することもできないので、手動での対処が必要になるのでしょう。

上記 URL に記載のとおりの手順ですが、順番に実行してみました。

1. 管理者権限でコマンドプロンプトを開き、`reagentc /info` で回復環境の状態が有効 (Enabled) であることを確認する
1. `reagentc /disable` で回復環境を無効にする

   ![reagentc][img03]

1. `diskpart` コマンドを実行する
1. diskpart プロンプトで `list disk` を実行する。ディスクの一覧が表示されるので、複数存在する場合は回復パーティションが存在するディスクの番号を確認する
1. `sel disk 0` (ディスク番号が `0` の場合) を実行する

   ![diskpart][img04a]

1. `list part` を実行する。パーティションの一覧が表示されるので、縮小するパーティションを番号を確認する
1. `sel part 3` (パーティション番号が `3` の場合) を実行する
1. `shrink desired=250 minimum=250` を実行する (250MB 縮小するので、該当のパーティションに 250MB 以上の空き容量があることを確認したうえで実行します)

   ![diskpart][img04b]

1. `sel part 4` (`list part` の結果で回復パーティションの番号が `4` の場合) を実行する
1. `delete partition override` を実行し、回復パーティションを削除する (回復パーティションは保管が必要なデータもないのと、パーティションの先頭位置をずらす必要があるため、サイズの拡張ではなく、削除して作成する手順になっていると思われます)

   ![diskpart][img04c]

1. `create partition primary id=de94bba4-06d1-4d40-a16a-bfd50179d6ac` を実行し、パーティションを作成します
1. `gpt attributes =0x8000000000000001` を実行し、パーティションの属性を割り当てます (MBR の場合はコマンドが異なるため、Microsoft のサイトを参照してください)

   ![diskpart][img04d]

1. `format quick fs=ntfs label=”Windows RE tools”` を実行してフォーマットします
1. `list vol` で回復パーティションが作成されていること、またサイズが増加していることを確認します
1. `exit` で diskpart を終了します

   ![diskpart][img04e]
   ![diskpart][img04f]

1. `reagentc /enable` コマンドを実行し、回復環境を有効にします
1. `reagentc /info` コマンドを実行し、正常に回復環境が有効になっていることを確認します

![reagentc][img05]

この状態で再度 Windows Update を実行したところ、正常に更新が完了しました。

![正常終了][img06]



[kb5028997]:https://support.microsoft.com/ja-jp/topic/kb5028997-winre-%E6%9B%B4%E6%96%B0%E3%83%97%E3%83%AD%E3%82%B0%E3%83%A9%E3%83%A0%E3%82%92%E3%82%A4%E3%83%B3%E3%82%B9%E3%83%88%E3%83%BC%E3%83%AB%E3%81%99%E3%82%8B%E3%81%9F%E3%82%81%E3%81%AB%E3%83%91%E3%83%BC%E3%83%86%E3%82%A3%E3%82%B7%E3%83%A7%E3%83%B3%E3%81%AE%E3%82%B5%E3%82%A4%E3%82%BA%E3%82%92%E6%89%8B%E5%8B%95%E3%81%A7%E5%A4%89%E6%9B%B4%E3%81%99%E3%82%8B%E6%89%8B%E9%A0%86-400faa27-9343-461c-ada9-24c8229763bf

[img01]:/assets/images/2024/07/ss-20240721-01.png
[img02]:/assets/images/2024/07/ss-20240721-02.png
[img03]:/assets/images/2024/07/ss-20240721-03.png

[img04a]:/assets/images/2024/07/ss-20240721-04-1.png
[img04b]:/assets/images/2024/07/ss-20240721-04-2.png
[img04c]:/assets/images/2024/07/ss-20240721-04-3.png
[img04d]:/assets/images/2024/07/ss-20240721-04-4.png
[img04e]:/assets/images/2024/07/ss-20240721-04-5.png
[img04f]:/assets/images/2024/07/ss-20240721-04-6.png

[img05]:/assets/images/2024/07/ss-20240721-05.png
[img06]:/assets/images/2024/07/ss-20240721-06.png
