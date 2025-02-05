---
toc: true
title:  "AtomS3 に PlatformIO IDE を使ってプログラムを実行させる"
date:   2025-02-05 00:00:00 +0900
categories: デバイス
tags:
- M5Stack
- IoT
- platformio
- wsl
- usbipd
---
M5Stack は提供されるファームウェアだけでなく、自分で作ったコードを動かすことも比較的簡単です。今回は AtomS3 を使って、コードをビルドして実際にデバイス上で実行するまでをやってみます。

# 使用する環境
Windows11 の WSL に環境を用意します。コンテナを使うとリセットが容易なのですが、作成したコードを転送する際にシリアルで通信する必要があります。特にデバッグなどでは機器の再起動も発生しますが、コンテナ内だと再起動時にシリアルポートを見失ってしまい、コンテナを立ち上げなおす必要があるなど不便です。

とはいえ、Windows11 に直接環境を作ってしまうと、何かあったときに面倒なことになるので、PlatformIO 実行用に WSL のディストリビューションを使う形にします。

使用するディストリビューションはなんでもいいのですが、ひとつ前の Ubuntu あたりが無難かと考えて Ubuntu 22.04 を選びました。

# 準備
## Ubuntu
まずは普通に WSL で Ubuntu 22.04 をインストールします。Windows11 で PowerShell を開き、以下のコマンドを実行します。

``` bash
wsl --install Ubuntu-22.04
```

![ubuntu-22.04][img01]

ユーザー名とパスワードを設定すれば、いつも通りのプロンプトが起動します。次に、必要なパッケージをインストールします。

``` bash
sudo apt update
sudo apt upgrade -y
sudo apt install -y python3-venv git
```

## VScode
Windows11 で VScode を起動します。リモート エクスプローラー ＞ WSL ターゲット ＞ Ubuntu-22.04 と選択し、接続します。

![remote explorer][img02]

左下に「WSL:Ubuntu-22.04」と表示されていることを確認し、拡張機能を開き検索欄から「platform」を検索します。以下の画面のように「PlatformIO IDE」が見つかると思うので、これをインストールします。

![install PlatformIO][img03]

ライブラリのダウンロードもあるので、少し待ちます。拡張機能のインストールが終わると、確か VScode の再起動を求められるので、再起動すると以下のように PlatformIO IDE のアイコンが追加されます。

![PlatformIO][img04]

# プロジェクト作成
本来は PlatformIO IDE の画面からプロジェクトを作成したりできるのですが、今回は Windows11 で起動した VScode で Ubuntu 側の PlatformIO IDE を操作するため、パス区切りに不具合があるなどすんなりとはいきません。そのため、CLI を使ってプロジェクトの作成を行います。

まず、PlatformIO IDE 拡張の「PlatformIO Core CLI」を開きます。

![ide][img05]

開いたコンソールで適当なディレクトリを作成し、その中で `pio init` コマンドを実行してプロジェクトを初期化します。

![pio init][img06]

※`--borad` に渡す引数の確認方法は後ほど記載します

このように `Project has been successfully initialized!` と表示されれば OK です。

![initialized][img07]

VScode のファイルエクスプローラーより、フォルダーを開くで先ほど初期化したフォルダーを開けば、自動的に PlatformIO IDE が認識してプロジェクトを開いてくれます。ウインドウ下部に PlatformIO IDE のコントロールアイコンが並んでいるのがわかります。

![open project][img08]

## board の引数
`pio init` コマンドのオプションで `--board` がありましたが、ここで対象のデバイスを指定する必要があります。`pio init` を実行する CLI で、

``` bash
pio boards
```

と実行すると、`--board` で利用可能な引数の一覧がでますが、数が多いので探すのが大変です。その場合は、

``` bash
pio boards atom
```

と検索することができます。この場合だと「M5Stack AtomS3」と「M5Stack-ATOM」が出てきたので、今回は AtomS3 を使います。

![pio boards][img09]

# Hello World
今回使用する AtomS3 にはディスプレイがあるので、そこに「Hello World」と表示してみましょう。まず、ディスプレイに出力するために必要なライブラリを定義するため、以下の行をプロジェクトのルートディレクトリに作成された「platformio.ini」ファイルに追記します。

``` ini
lib_deps = m5stack/M5Unified@^0.1.16
```

![platformio.ini][img10]

platformio.ini ファイルを保存すると、自動的にライブラリのロードが行われます。そのうえで、プロジェクトのルートディレクトリにある `src` の中に `main.cpp` というテキストファイルを作成し、以下の内容を記述します。

``` cpp
#include <M5Unified.h> 

void setup() {
    auto cfg = M5.config();

    M5.begin(cfg);
    M5.Display.setTextSize(3);
    M5.Display.print("Hello World!!");
}

void loop() {
    
}
```

ファイルを保存したら、VScode のウインドウ下部にある「PlatformIO: Build」のボタンをクリックしてビルドをします。

![build][img11]

ビルドが正常に終わると「SUCCESS」と表示されます。

![success][img12]

あとはデバイスにアップロードします。AtomS3 を USB で PC に接続し、usbipd コマンドを使って WSL にデバイスを接続します ([前回の記事の「補足 (WSL の Ubuntu に USB デバイスを接続する)」][usbipd] が参考になります)。

そのうえで、VScode のウインドウ下部にある「PlatformIO: Upload」のボタンをクリックします。

![upload][img13]

アップロードが行われ、こちらも「SUCCESS」と表示されれば OK です。この状態で AtomS3 の画面には、「Hello World」と表示されているはずです。

![hello World][img14]




[usbipd]:{% post_url 2025/2025-01-27-stampfly-ardupilot %}

[img01]:/assets/images/2025/01/ss-20250204-01.png
[img02]:/assets/images/2025/01/ss-20250204-02.png
[img03]:/assets/images/2025/01/ss-20250204-03.png
[img04]:/assets/images/2025/01/ss-20250204-04.png
[img05]:/assets/images/2025/01/ss-20250204-05.png
[img06]:/assets/images/2025/01/ss-20250204-06.png
[img07]:/assets/images/2025/01/ss-20250204-07.png
[img08]:/assets/images/2025/01/ss-20250204-08.png
[img09]:/assets/images/2025/01/ss-20250204-09.png
[img10]:/assets/images/2025/01/ss-20250204-10.png
[img11]:/assets/images/2025/01/ss-20250204-11.png
[img12]:/assets/images/2025/01/ss-20250204-12.png
[img13]:/assets/images/2025/01/ss-20250204-13.png
[img14]:/assets/images/2025/01/ss-20250204-14.png
[img15]:/assets/images/2025/01/ss-20250204-15.png
