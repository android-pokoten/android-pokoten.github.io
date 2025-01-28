---
toc: true
title:  "Stamp Fly に ArduPilot のファームウェアを書き込んでみる"
date:   2025-01-27 09:00:00 +0900
categories: デバイス
tags:
- Stamp-fly
- IoT
- M5Stack
- ardupilot
- mission planner
---
# 前提
[ArduPilot (アルジュパイロット)][ardupilot] という、ドローンのオープンソースソフトウェアがあるそうです。Stamp Fly で動作する ArduPilot もあるようなので、それをビルドして書き込み、さらに ArduPilot からリリースされている[地上制御ソフトウェア Mission Planner][missionplanner] と連携させることも可能なようです。

# まとめ
* Stamp Fly の ArduPilot ファームウェアをビルドして書き込むことができました
* Mission Planner から各種センサーの値を取得することができました
* 同じく Mission Planner を使用して手動操作で Stamp Fly を飛ばすことができました
* 自動飛行はできません。GPS が必須のようです。また、高度の自動調整もできていないので、標準のファームウェアに比べて安定して飛ばすのがかなり難しいです
* 元のファームウェアに戻すには、[M5Burner][m5burner] で再度書き込みを行えば OK です

# ファームウェアのビルド
## ビルド環境
Windows11 の WSL で Ubuntu を動作させ、その中で Podman を使ってコンテナ環境を作り、その中でビルドを行います。また、ビルド環境からファームウェアを Stamp Fly へ書き込むことになります。そのため、コンテナから Stamp Fly を認識できるようにする必要があります。

(手元にあった環境の都合上、Docker ではなく Podman を使用しました)

### Git リポジトリのクローン
ArduPilot のリポジトリではなく、Stamp Fly 向けにフォークしたリポジトリを使います。[https://github.com/tpwrules/ardupilot.git][stampfly-ardupilot] がそうです。なので、このリポジトリをクローンします。

``` bash
git clone https://github.com/tpwrules/ardupilot.git
```

でローカルにコピーできます。

### コンテナイメージのビルド
まずビルド用のコンテナイメージをビルドします。やらなくてもよい (コンテナを起動した後に環境を作る、でも可能) ですが、コンテナを閉じるとまた一から環境作り直しだと時間がかかるので、ビルドしたほうがスムーズです。

ビルドするには、クローンしたディレクトリへ移動し、以下のように `podman build` コマンドを実行します。

``` bash
cd ardupilot
podman build . -t ardupilot --build-arg USER_UID=$(id -u) --build-arg USER_GID=$(id -g)
```

クローンしたディレクトリには `Dockerfile` という名前のファイルがあるので、その記述に沿ってコンテナイメージがビルドされます。開発ツールなどを導入するので、それなりに時間がかかると思います。

これでファームウェアビルド用のコンテナイメージが作成されますが、このイメージは ArduPilot 標準のビルド環境です。今回は Stamp Fly、つまり ESP32 向けにビルドをしたいので、もう少し追加の手順が必要となります。

先ほどのコマンドで作成したイメージを使い、以下コマンドでコンテナを起動します。

``` bash
podman run --rm -it -v "$(pwd):/ardupilot" -u "$(id -u):$(id -g)" --userns=keep-id --group-add=keep-groups ardupilot:latest bash
```

コンテナが起動したら、以下のコマンドを実行して ESP32 用のビルド環境を追加します。

``` bash
sudo apt update
sudo apt install -y libusb-1.0-0 python3.10-venv cmake
python3 -m pip install pexpect empy future
pushd modules/esp_idf/
./install.sh
```

こちらも各種ツールのダウンロードとインストールを行うため、それなりに時間がかかります。インストールが完了したら、コンテナを終了させないように注意してください (終了すると ESP32 用のビルド環境を追加するところを再実行する必要があります)。その状態で、Windows Terminal の新しいタブを開き、 コンテナの外の Ubuntu のターミナルから以下コマンドを実行します。

``` bash
podman commit [コンテナ ID] ardupilot:m5stamp
```

`コンテナ ID` は、`podman ps` コマンドなどで調べます。これで、すぐにファームウェアのビルドが開始できるコンテナが用意できました。先ほどのコンテナのターミナルに戻り、`exit` でコンテナを閉じます (閉じずにファームウェアのビルドに進むことも可能です)。

### ArduPilot のビルド
コンテナを閉じた場合は、再度コンテナを起動します。この場合、イメージは ESP32 用のビルド環境を追加したイメージを指定します。

``` bash
podman run --rm -it -v "$(pwd):/ardupilot" -u "$(id -u):$(id -g)" --userns=keep-id --group-add=keep-groups ardupilot:m5stamp bash
```

コンテナが起動したら、以下コマンドを実行してファームウェアのビルドを実行します。

``` bash
. modules/esp_idf/export.sh
./waf configure
./waf configure --board=esp32s3m5stampfly
python -m pip install empy pexpect
./waf copter
```

ビルドもそれなりに時間がかかりますが、問題なくビルドが完了しました。

### Stamp Fly へ書き込み
書き込む Stamp Fly を USB ケーブルで PC に接続します。

ビルドしたファームウェアを Stamp Fly へ書き込むには、ビルド時に使用した waf コマンドを使用します。このコマンドで、直接シリアルポート経由でファームウェアを書き込むので、コンテナ内へ Stamp Fly を接続する必要があります。Podman を rootless で動作させていることもあってこれが少し苦労しました。結果的には以下コマンドで、コンテナ内から /dev/ttyACM0 にアクセスが可能となり、Permision error も出ずに書き込みが可能になりました。

``` bash
podman run --rm -it --device=/dev/ttyACM0 -v "$(pwd):/ardupilot" -u "$(id -u):$(id -g)" --userns=keep-id --group-add=keep-groups ardupilot:m5stamp bash
```

なお、前提としてコンテナ外の Ubuntu にて、Podman を実行するユーザーを `dialout` グループに追加している必要があります。これは、/dev/ttyACM0 を所有するグループになります。もし追加していない場合は、

``` bash
sudo usermod -a -G dialout [ユーザー名]
```

を実行することで追加できます。

/dev/ttyACM0 を認識できる状態でコンテナを起動したら、以下のコマンドでファームウェアを Stamp Fly へアップロードします。

``` bash
. modules/esp_idf/export.sh
./waf configure
./waf configure --board=esp32s3m5stampfly
python -m pip install empy pexpect
ESPBAUD=1500000 ESPPORT=/dev/ttyACM0 ./waf copter --upload
```

![upload][img01]

このように、Writing が正常に完了していれば OK です。Stamp Fly の USB ケーブルは外して、コンテナも閉じてしまって問題ありません。

# Mission Planner との接続
## 操作環境
Mission Planner は Linux でも動作するようですが、Mono 上で動作させる形となり、実行ファイル自体は Windows と共通のもののようです。今回は Windows11 の PC を使用しました。Stamp Fly とは Wi-Fi で接続する形になります。

### Wi-Fi AP 接続
Stamp Fly にバッテリーを接続し、水平な場所へ置きます。PC から Wi-Fi の一覧を確認すると、`ardupilot123` という AP が見えるはずです。これに接続します (パスワードは SSID と同じです)。

![ssid][img02]

`ardupilot123` に接続した状態で Mission Planner を起動すると (先に Mission Planner を起動してから AP に接続しても問題ないと思います)、ポップアップが表示され、Stamp Fly とステータスなどの情報をやり取りしているようです。

![status][img03]

この状態で、Mission Planner にデータが表示されます。試しに、Stamp Fly を手で持って動かすと、それに合わせて高度や機体の傾きが表示されます。

![sensors][img04]

### 初期設定
ArduPilot の [Mandatory Hardware Configuration][first-time-setup] の手順を実施して、Stamp Fly のパラメータを調整します。

![initial configuration][img05]

こんな感じで、機体の種類を選択したり、指定の向きに機体を傾けてセンサー値を調整したりします。

[Altitude Hold Mode][althold] の記述もあるので、高度維持の設定もありそうなのですが、今のところ手元の Stamp Fly では実現できていません・・・

# Mission Planner から手動操作
## ジョイスティックの設定
Mission Planner の初期設定＞オプションハードウェア＞Joystick と選択します。

![joystick][img06]

PC にゲームコントローラーを接続すると、「Joystick」欄にそれが表示されると思います。まだ `Enable` はクリックしません。ここで、ゲームコントローラーのボタン類と、Stamp Fly の操作を紐づけます。`RC1`、`RC2` と表示されているのが Stamp Fly へのスティック操作、`But1`、`But2` はボタン操作になります。`Controller Axis` はゲームコントローラーのスティックの指定です。`Y` や `X`(ここでは設定していませんが) は左スティック、`Rx` や `Ry` が右スティックです。では `Rc1` や `RC2` は何を示しているのかというと、必須ハードウェア＞ラジオ キャリブレーション を選択すると確認できます。

![radio][img07]

これを見ると、`RC1` がロール、`RC2` がピッチ、`RC3` がスロットル、`RC4` がヨーとなっています。それをもとにジョイスティックの設定を行い、`Save` をクリックして保存しておきます。

## 飛行
Mission Planner の「アクション」タブを開き、「ジョイスティック」をクリックすると先ほどのジョイスティックの設定画面がポップアップで開きます。そこで「Enable」をクリックしてジョイスティックの入力を有効にします。「Arm/Disarm」をクリックすると、Stamp Fly のモーターが回転を始めると思います。

![action][img08]

この状態でスロットルに割り当てたスティックを操作すると、モーターの回転をコントロールすることができると思います。ヨーやピッチ、ロールに割り当てたスティック操作で、Stamp Fly がコントロールできると思いますが、高度維持にならないのと、直接操作ではないからか、多少の遅延が発生するようで、安定した操作はとても難しいと思います。チューニングが必要そうですね。

# 補足 (WSL の Ubuntu に USB デバイスを接続する)
WSL の Ubuntu に USB デバイスを接続するには、`usbipd` を使用します。最初に、管理者権限のターミナルを開いて、デバイスを共有可能にする必要があります。これは、最初に一回行えば、USB ケーブルを抜き差ししても再実行する必要はありませんでした。

USB ケーブルを Stamp Fly と PC に接続した状態で、管理者権限のターミナルから `usbipd list` を実行し、目的のデバイスを確認します。以下の例では `BUSID` が「2-1」のデバイスが Stamp Fly です。そのため、`usbipd bind -b 2-1` というコマンドを実行して、共有可能にします。再度 `usbipd list` コマンドを実行すると、BUSID 2-1 の行の `STATE` が `Shared` になったことがわかります。

![admin terminal][img09]

その後、元のユーザーでターミナルを開きます。この時点で、WSL の Ubuntu も起動状態にしておきます。`usbipd ` はコマンドプロンプトでは実行できないので、PowerShell で開きます。その状態で、`usbipd attach -w ubuntu -b 2-1` と実行します。これでデバイスを認識する音が鳴り、Windows からは Stamp Fly が見えなくなり、Ubuntu へ接続された形になります。

※メッセージを見ると、ディストリビューションを指定する必要はなくなったみたいですね。以前は `attach` のサブコマンドに `wsl` があって、WSL にはそのサブコマンドを使ってアタッチしていた記憶もあるのですが、usbipd コマンドが更新されてオプションが変更になるかもしれない点は注意です。

![attach][img10]

そのうえで Ubuntu から確認すると、以下のように `/dev/ttyACM0` が存在します。

![ttyacm0][img11]

[ardupilot]:https://ardupilot.org/
[missionplanner]:https://ardupilot.org/planner/
[stampfly-ardupilot]:https://github.com/tpwrules/ardupilot
[m5burner]:https://docs.m5stack.com/en/download
[first-time-setup]:https://ardupilot.org/copter/docs/configuring-hardware.html
[althold]:https://ardupilot.org/copter/docs/altholdmode.html#altholdmode

[img01]:/assets/images/2025/01/ss-20250126-01.png
[img02]:/assets/images/2025/01/ss-20250126-02.png
[img03]:/assets/images/2025/01/ss-20250126-03.png
[img04]:/assets/images/2025/01/ss-20250126-04.png
[img05]:/assets/images/2025/01/ss-20250126-05.png
[img06]:/assets/images/2025/01/ss-20250126-06.png
[img07]:/assets/images/2025/01/ss-20250126-07.png
[img08]:/assets/images/2025/01/ss-20250126-08.png
[img09]:/assets/images/2025/01/ss-20250126-09.png
[img10]:/assets/images/2025/01/ss-20250126-10.png
[img11]:/assets/images/2025/01/ss-20250126-11.png
