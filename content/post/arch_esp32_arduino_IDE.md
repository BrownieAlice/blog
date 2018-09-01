---
title: "Arch LinuxでESP32の環境を構築(Arduino IDE)"
date: 2018-09-02T03:51:18+09:00
categories:
 - "TechNote"
tags:
 - "ESP32"
toc: true
thumbnail: "images/arch_esp32_arduino_IDE/esp32.JPG"
---
## ESP32とは
ESP32はWifiモジュールとBluetoothモジュールがついてデュアルコアなIoTマイコンです.  
秋月電子でもESP32のモジュールと開発ボード(ESP32-DevKitC)が販売されています.[^1] [^2]  
消費電力は大きいものの, 高性能で値段が手頃なこともあり, 一部の界隈で流行っている気がします.

## 開発環境について
ESP32の開発環境は2つあります.  
両方とも公式ですが, 1つは[ESP-IDF](https://docs.espressif.com/projects/esp-idf/en/stable/index.html)と呼ばれる独自の開発環境.  
もう1つはArduino IDEを利用した開発環境である[Arduino core for ESP32 WiFi chip](https://github.com/espressif/arduino-esp32).  
Arduino IDEのほうが親しみがあるのか, 独自環境ではなくArduino IDEを使ってる人を多く見かけます.  
私もArduino IDEを利用したほうの環境を構築しました.  
しかし, 公式に書かれているインストール方法はWindows/Mac/Debian/Ubuntu/Fedora/OpenSUSEのみです.  
なので, Arch Linux向けのインストール方法を一応書いておきます.

## 環境構築(Arch Linux)
Arch Linux向けのインストール方法がないとは言え, 基本的にはUbuntu向けのやつと同じです.  
以下のようにすればインストールできます.

```bash
sudo pacman -S arduino python-pip
sudo gpasswd -a $USER uucp
sudo gpasswd -a $USER lock
sudo pip install pyserial
sudo mkdir -p /usr/share/arduino/hardware/espressif
cd /usr/share/arduino/hardware/espressif/
sudo git clone https://github.com/espressif/arduino-esp32.git esp32
cd esp32
sudo git submodule update --init --recursive
cd tools
sudo python2 get.py
```

Ubuntu向けとの差としては

* pacmanでArduinoをインストールしているため, インストール場所が変わっていること
* USBと通信するためのアカウントが `dialout` から `uucp` と `lock` になっていること

くらいです.  
Arch Linuxを使っている人には自明なことだらけだと思いますが, 特に問題なく導入できたということが伝えられれば幸いです.

## 参考文献
* [Debianでesp32の環境構築をしてLチカするまで](https://qiita.com/TYuto/items/7065fc2f30e50dcc95c6)
* [Installation instructions for Debian / Ubuntu OS](https://github.com/espressif/arduino-esp32/blob/master/docs/arduino-ide/debian_ubuntu.md)
* [Arduino - ArchWiki](https://wiki.archlinux.jp/index.php/Arduino)

[^1]:[ＥＳＰ－ＷＲＯＯＭ－３２: 無線、高周波関連商品 秋月電子通商-電子部品・ネット通販](http://akizukidenshi.com/catalog/g/gM-11647/)
[^2]:[ＥＳＰ３２－ＤｅｖＫｉｔＣ　ＥＳＰ－ＷＲＯＯＭ－３２開発ボード: 無線、高周波関連商品 秋月電子通商-電子部品・ネット通販](http://akizukidenshi.com/catalog/g/gM-11819/)
