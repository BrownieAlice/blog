---
title: "Arch LinuxでROS Lunarを使う"
date: 2018-04-30T21:48:13+09:00
categories:
 - "TechNote"
tags:
 - "ROS"
toc: true
thumbnail: "images/ros-lunar_on_arch/rqt_graph.png"
---

## 経緯
ロボット界隈では相当有名で, 最近では[aibo](https://aibo.sony.jp/)でも使われていることで話題になっている[ROS](http://wiki.ros.org/).
現在の最新版であるROS Lunarが公式に対応[^1]しているのはUbuntu/Debian/Gentooあたりです.[^2]  
最近私は[Arch Linux](https://www.archlinux.org/)に乗り換えたのでArchでもROSを使おうと思い始めました.
実はAURにROS Lunarは存在し, [日本語wikiページ](https://wiki.archlinux.jp/index.php/Ros)も存在します.  
ただ

> ROS Lunar は AUR パッケージでインストールすることができ、問題なく動作します。

と書かれているものの, そのままではインストールに失敗するところがいくつかありました.
正直Archを使ってる人には不要だと思いますが, どうすればうまくインストールすることが出来るのか書いておきます.

## インストール
今回は[ros-lunar-desktop-full](https://aur.archlinux.org/packages/ros-lunar-desktop-full/)をインストールします.  
そのままインストールしていると, おそらく[ros-lunar-image-view](https://aur.archlinux.org/packages/ros-lunar-image-view/)のビルドで失敗します.
これはコメントにも書かれていますが, 本来は必要な[gtkglext](https://www.archlinux.org/packages/extra/x86_64/gtkglext/)が依存関係に含まれていないためです.
前もってインストールしておきましょう.  
おそらくインストールはこれで無事終わるはずです.
なのでインストール自体はこんな感じで終わるはずです.

```bash
sudo pacman -S gtkglext
yaourt -S ros-lunar-desktop-full
```

ちなみに, ros-lunar-desktop-fullはひたすらyを押す必要があるので `--noconfirm` オプションをつけておくと良いでしょう.

## 初期設定
次にROSの `catkin_ws` をビルドします.
ですが, このビルドも失敗します.  
理由としては `python-empy` と `catkin-pkg` の2つが認識できていないためです.
前者はAURにある[python-empy](https://aur.archlinux.org/packages/python-empy/)をインストールすることで解決します.  
後者は, 依存関係により[python2-catkin_pkg](https://aur.archlinux.org/packages/python2-catkin_pkg/)がインストールされているはずなのですが何故か認識されません.
なので, pipの `catkin-pkg` を無理やり入れて解決しました.  
結果としては以下のコマンドを売ってから `catkin_make` しました.

```bash
yaourt -S python-empy
sudo pacman -S python-pip
pip install --user catkin-pkg
```

## 起動確認
とりあえずturtlesimでも起動してみます.

{{% img src="images/ros-lunar_on_arch/turtle_sim.png" caption="turtlesim" %}}

無事動きました.
他にもrqt_bagやrvizや自作プログラムを走らせましたが今のところ問題なさそうでした.  
この辺の話は本来ならissueを投げるみたいなことをすべきなのかもしれませんが, Archは最近使い始めたばかりでコミュニティについて全然理解してないので自分のブログに書く程度にとどめておきます.[^3]

[^1]:実験的にサポートされているものを含む.
[^2]:http://wiki.ros.org/lunar/Installation
[^3]:gtkglextに関してはすでにコメントされてますしね.
