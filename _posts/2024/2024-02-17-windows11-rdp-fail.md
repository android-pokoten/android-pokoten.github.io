---
layout: post
title:  "Windows11 の RDP クライアントで資格情報が使えない"
date:   2024-02-17 09:00:08 +0900
categories: トラブル
tags:
- windows11
- RDP
- 資格情報
---
Windows 11 からリモートデスクトップしようとすると、資格情報の入力を求められてしまいます。保存することもできるはずですが、なぜか保存してもまた入力を求められてしまいます。
![資格情報使用不可][img01]

パスワードを入力すれば接続できるのであまり気にしていなかったのですが、別の Windows 11 の端末では保存するとパスワード入力することなく接続できたので、改めてダイアログボックスをよく見てみました。

`お使いの資格情報は機能しませんでした`  
`Windows Defender Credential Guard では、保存された資格情報を使用できません。`

というメッセージです。とりあえず「Windows Defender Credential Guard」をキーワードに調べてみると、以下のサイトにたどり着きました。

[Windows 11 22H2 - Can't use saved credential][mslearn]

`cmdkey`コマンドを使って資格情報を保存することで解消するようです。これを見て、改めて資格情報マネージャーを見てみると、リモートデスクトップの資格情報 (「TERMSRV/」で始まる資格情報) は、「Windows 資格情報」欄に登録しています。以前は、確かこれでリモートデスクトップ接続時の資格情報として機能していたと記憶しています。
![Windows資格情報][img02]

しかし、上記サイトの情報によると、Generic、つまり「汎用資格情報」に登録する必要があるようです。  
cmdkey を使えばいいのですが、資格情報マネージャー画面からでも設定可能なようです。「汎用資格情報の追加」をクリックします。
![汎用資格情報][img03]

ここはいつも通り。アドレスは「TERMSRV/[リモートデスクトップの接続先]」で指定します。
![資格情報の追加][img04]

このように、「汎用資格情報」に資格情報が追加されます。
![資格情報][img05]

この状態でリモートデスクトップ接続を行うと、パスワード入力不要で接続することができました。


[mslearn]:https://learn.microsoft.com/en-us/answers/questions/1021785/windows-11-22h2-cant-use-saved-credential

[img01]:/assets/images/2024/ss-20240217-01.png
[img02]:/assets/images/2024/ss-20240217-02.png
[img03]:/assets/images/2024/ss-20240217-03.png
[img04]:/assets/images/2024/ss-20240217-04.png
[img05]:/assets/images/2024/ss-20240217-05.png
