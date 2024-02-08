---
layout: post
title:  "GitHub Pages のカスタムドメイン (続き)"
date:   2024-02-08 09:00:08 +0900
categories: ブログ
tags:
- jekyll
- github
- AWS
- Route53
---
以下投稿の続きです。  
[GitHub Pages のカスタムドメインを AWS の Route53 で用意する][prev]


![](/assets/images/2024/ss_20240208_01.png)
DNS Check in Progress は時間がかかるかも、と書きましたが、successful になっていました。


### ドメイン検証
結果的に関係ないのかもしれないのですが、GitHub のドキュメントにドメイン検証の記載があったので、こちらも実施していました。  
これをやったことで上記の Check が successful になったかもしれないですが、これが必須かどうかは不明です。


GitHub のサイトで、右上のアカウントアイコンをクリックし、開いたメニューの「Settings」をクリックします。
![](/assets/images/2024/ss_20240208_02.png)


左ペインの「Pages」をクリックし、右ペインの「Add a domain」をクリックします。
![](/assets/images/2024/ss_20240208_03.png)


カスタムドメインを入力します。www などを付けないドメイン名を指定する形です。
![](/assets/images/2024/ss_20240208_04.png)


「Add a DNS TXT record」と、ランダムな文字列が表示されます。平行して AWS の Route53 を開き、レコード追加画面を出します。  
レコードタイプは「TXT」にして、以下画面のとおり GitHub の設定画面に表示される文字列を、レコード追加画面に転記して「レコードを追加」をクリックします。  
レコードが追加されたら GitHub の設定画面の「Verify」をクリックするのですが、DNS の更新が反映されるまでは検証に失敗するので、その場合はしばらく待ってから再度 Verify してみます。
![](/assets/images/2024/ss_20240208_05.png)


うまくいくとこのように「Successfully verified」と表示されます。
![](/assets/images/2024/ss_20240208_06.png)


[prev]:{% post_url 2024/2024-02-07-pages-custom-domain-route53 %}
