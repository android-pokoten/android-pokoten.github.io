---
toc: true
title:  "AtomS3 Snake Game"
date:   2025-02-19 00:00:00 +0900
categories: デバイス
tags:
- M5Stack
- IoT
- platformio
- wsl
- atomjoystick
---
# 背景
![snake game][img01]
昔 Nokia の携帯電話で遊んだ記憶があるスネークゲームが、M5Burner の AtomS3 にありました。懐かしくて思わず遊んでみたのですが、姿勢センサーで操作するので思ったように操作できません。

そんなとき、ふと [Stamp Fly と一緒に購入した AtomJoystick][joystick] が目に入りました。これで操作できるのでは？

# 参考元
## Snake Game のソースコード
M5Burner の画面に、GitHub のリンクがあります。どうやら [https://github.com/Forairaaaaa/Stupinake][stupinake] がそのようです。リポジトリのトップにあるのは CMake でビルドするコードですが、`Arduino/AtomS3/PIO` の中に `main.cpp` がありました。これが PlatformIO IDE で使えそうです。

## Atom JoyStick のソースコード
[https://github.com/m5stack/Atom-JoyStick][atomjoystick] にあります。`src` 内の `AtomJoyStick.cpp` と `AtomJoyStick.h` をプロジェクトの `src` ディレクトリに配置することで、使用できそうです。

どのようにジョイスティックの値を取得するかは、上記リポジトリの `examples/GetValue` ディレクトリ内のコードが参考になりそうです。具体的には

``` c++
#include "AtomJoyStick.h"
```

とヘッダーをインクルードして、

``` c++
AtomJoyStick joystick;
```

と `AtomJoyStick` 型の変数を宣言します。さらに `setup()` 内で

``` c++
while (!joystick.begin(&Wire, ATOM_JOYSTICK_ADDR, 38, 39, 400000U)) {
    USBSerial.println("Couldn't find Atom JoyStick");
    delay(2000);
}
```

と変数 `joystick` の `begin` で初期化します。あとは `loop()` 内などで、getXX のメソッドを使用することでスティックの操作量やボタンのオン・オフ、取り付けたバッテリーの残量も取得可能です。

| メソッド | 取得する値 |
| --- | --- |
| joystick.getJoy1ADCValueX(_12bit) | 左スティックの左右 |
| joystick.getJoy1ADCValueY(_12bit) | 左スティックの上下 |
| joystick.getJoy2ADCValueX(_12bit) | 右スティックの左右 |
| joystick.getJoy2ADCValueY(_12bit) | 右スティックの上下 |
| joystick.getBattery1ADCValue(_12bit) | バッテリー1のセンサー値 |
| joystick.getBattery1Voltage(_12bit) | バッテリー1の電圧 |
| joystick.getBattery2ADCValue(_12bit) | バッテリー2のセンサー値 |
| joystick.getBattery2Voltage(_12bit) | バッテリー2の電圧 |

バッテリーのセンサー値は、それを元に電圧を算出している・・・と思っています。なので、バッテリーの状態を確認するには、電圧を見れば問題ないと思います。今回のスネークゲームでは、特にバッテリーの監視はしないつもりなので、スティックの値を見ればよいと思います。

# コーディング
## platformio.ini ファイル
`platformio.ini` ファイルには、ライブラリ関連を追記します。

``` ini
lib_deps =
  m5stack/M5AtomS3 @ 0.0.3
  lovyan03/LovyanGFX @ ^1.2.0
  jasonlzt/FastLED@^3.5.0
```

`M5unified` ではなく、元のソースコードが `M5AtomS3` を前提の記述だったので、それにあわせます。また、新しいバージョンだとビルドでエラーになるので、少し古めのバージョンにしました。

## AtomJoystick のライブラリ
上記の AtomJoystick の GitHub より、`AtomJoyStick.cpp` と `AtomJoyStick.h` をダウンロードして、プロジェクトの `src` ディレクトリに配置します。

## main.cpp ファイル
AtomJoystick のソースコードのところで記載したとおり、ヘッダーファイルをインクルードしたり、setup() で初期化したりしたうえで、もともとは姿勢センサーの値を取得していた `Game_Input_Update_Callback` を修正します。

``` c++
 void Game_Input_Update_Callback(MoveDirection_t &MoveDirection)
 {
     /* Change moving direction, e.g. MoveDirection = MOVE_RIGHT */
     /*
     float ax, ay, az = 0;
     M5.IMU.getAccel(&ax, &ay, &az);
     if (ay > 0.3)
         MoveDirection = MOVE_UP;
     else if (ay < -0.3)
         MoveDirection = MOVE_DOWN;
     else if (ax < -0.3)
         MoveDirection = MOVE_LEFT;
     else if (ax > 0.3)
         MoveDirection = MOVE_RIGHT;
    */
    uint16_t joy1_x, joy1_y;

    joy1_x = joystick.getJoy1ADCValueX(_12bit);
    joy1_y = joystick.getJoy1ADCValueY(_12bit);
    //USBSerial.printf("x: %d\n", joy1_x);
    if (joy1_y > 2500)
        MoveDirection = MOVE_DOWN;
    else if (joy1_y < 1500)
        MoveDirection = MOVE_UP;
    else if (joy1_x < 1500)
        MoveDirection = MOVE_LEFT;
    else if (joy1_x > 2500)
        MoveDirection = MOVE_RIGHT;

    //USBSerial.print(MoveDirection);
 }
```

コード全体は [https://github.com/android-pokoten/Stupinake][mysource] にあります。

![playing][img02]

これを書き込んで、Atom JoyStick の電源を入れると左スティックで操作して遊ぶことができます。


[joystick]:{% post_url 2025/2025-01-23-m5stamp-fly %}
[stupinake]:https://github.com/Forairaaaaa/Stupinake
[atomjoystick]:https://github.com/m5stack/Atom-JoyStick
[mysource]:https://github.com/android-pokoten/Stupinake

[img01]:/assets/images/2025/02/ss-20250219-01.png
[img02]:/assets/images/2025/02/ss-20250219-02.jpg
