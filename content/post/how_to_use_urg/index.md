---
title: "URG使い方"
date: 2018-01-01T21:25:44+09:00
categories:
 - "TechNote"
tags:
 - "ROS"
toc: true
thumbnail: "rosgraph.svg"
---
[北陽電機](https://www.hokuyo-aut.co.jp/)が発売している測域センサ(URG)を[ROS](http://wiki.ros.org/ja)の[urg_node](http://wiki.ros.org/urg_node)を利用して使う方法について記述する. 動作環境はUbuntu 16.04 LTS.

## 各種解説
### URG
北陽電機が発売している測域センサで無人搬送車(AGV)や安全柵無しの産業用ロボットなどによく使われている. NHK学生ロボコンやつくばチャレンジでもよく見かけるセンサ.  
同一平面上の物体の距離を測定できる. Class1レーザを周囲に照射し, 自身に戻ってくるまでの時間(位相差)を測ることによって距離を計測する(TOFセンサ). 基本的には270度の範囲を1081点の点列で取得できる(機種による). 範囲は主に数10mで操作時間は数10ms. 最近平面だけでなく3Dで取得可能なタイプ[YVT-35LX](https://www.hokuyo-aut.co.jp/search/single.php?serial=165)も発売された. この辺りの話は北陽電機のサイトを見たほうが早いだろう.  
似たようなセンサである[Velodyne](http://www.velodynelidar.com/index.html)のLiDARと比べると検出距離が短い(Velodyneは数100mまで検知可能[^1])一方で価格は数10万円と安い(Velodyneは数100万円)という違いがある.  
LRFとも呼称される事もあるが, URGが商品ブランド名で, LRFは学術系で用いられる測域センサの名称, LiDARは産業界で用いられる測域センサの名称として使われているように感じる.

### ROS
ROSはRobot Operating Systemの略で, オープンソース(BSDライセンス)で開発されているロボット用のプラットフォーム. マイコンではなくシングルボードコンピュータなどでの利用がメイン.  
Operating Systemとは言っているものの, GNU/Linuxの代替になるものではなく既存のOS上に構築する. センサなどのハードウェアの抽象化が行え, 各種ビジュアライザやライブラリなどが豊富なため開発の効率化が図れる.  
[ROS日本語ページ](http://wiki.ros.org/ja)では以下のように説明されている.

> ROS (Robot Operating System)はソフトウェア開発者のロボット・アプリケーション作成を支援するライブラリとツールを提供しています. 具体的には, ハードウェア抽象化, デバイスドライバ，ライブラリ，視覚化ツール, メッセージ通信，パッケージ管理などが提供されています. ROSはオープンソースの一つ, BSDライセンスにより, ライセンス化されています.

### URG Library
[URG Library](http://urgnetwork.sourceforge.net/html_ja/)は北陽電機が公式で配布しているライブラリ. URGからの情報を取得しすることができる. 言語はC/C++でオープンソース(修正BSD/LGPL).  
このライブラリはROSとは関係がない. ROSを利用せずに用いる場合はこのライブラリを使うことになるだろう.

### urg_node
urg_nodeはURG Library用いた, ROS用のURGからデータを取得するパッケージ. 北陽電機もROSでURGを利用する際にはこのパッケージを推奨している.[^2]  
ROSの場合は基本的にurg_nodeで事足りる. 適切にパラメータを設定したり, roslaunchファイルを書いたりすれば良いだろう.

## URGセットアップ
### 通信方式
URGの通信方式は2種類ある. Ethernetを利用するタイプとUSBを利用するタイプだ. [TOP-URG](https://www.hokuyo-aut.co.jp/search/single.php?serial=21)や最も安い[URG-04LX-UG-01](https://www.hokuyo-aut.co.jp/search/single.php?serial=17)はUSBタイプ. ただしUSBよりEthernetの方がノイズに強いため最近販売している機種は基本的にEthernetらしい.  
USBタイプは簡単に利用できるが, Ethernetタイプは一回いじってから利用したほうが便利になる.

### Ethernetタイプ
Ethernetタイプは初期状態では `192.168.0.10:100940` でデータを送信している. しかし, `192.168.x.x` はプライベートIPアドレスの中でもwifi接続時によく割り当てられるIPでもある. wifi接続している状態で, URGに `192.168.0.10` が割り当てられると, URGの接続先がルータでもない限りはIPが干渉して上手くネットに接続できなくなることがある. そのため, 他のプライベートIPである `172.16.x.x` をURGに割り当てることでネット接続を両立することができる.  
URGのIP変更するためのソフトが北陽電機より配布されている. [UTM-30LX-EW](https://www.hokuyo-aut.co.jp/search/single.php?serial=146)の各種ダウンロードからアクセスできる "UTM-30LX-EW IPチェンジャー" がそのソフト. UTM-30LX-EWでなくとも, [UST-20LX](https://www.hokuyo-aut.co.jp/search/single.php?serial=16)でもキチンと動作した.  
このソフトはWindowsで動かすものなので, 残念ながらWindowsを立ち上げないといけない. 立ち上げた後はEthernetケーブルを指し, ローカルエリア・ネットワークでEthernetポートに適切にIPを割り当ててやる. [^3]  
あとはそのソフトを立ち上げれば設定できる. 今回はこのように設定した.

| 対象 | 変更後アドレス |
| --- | --- |
| IP address | 172.16.0.10 |
| Subnet mask | 255.255.255.0 |
| Gateway address | 172.16.0.1 |

ちなみに, EthernetタイプのURGを複数台同時運用する際にもこのように異なるIPを割り当てたほうが良い. 同じIPだとURG同士を簡単には区別することができない. そこで異なるIPを割り当て, スイッチングハブなどを介してコンピュータにつなげることで区別したまま同時運用ができるはずである.

## 動作確認
ひとまずきちんと動作しデータを取得できるか確認する.  
ROSは導入してある前提として

```bash
sudo apt-get install ros-${ROS_DISTRO}-urg-node
```

を実行すれば必要なパッケージは揃う.

### USBタイプ
まずURGのUSBケーブルをコンピュータにつなげる. すると `/dev/ttyACM*` が現れる. `*` は実際には数字なので, どの数字になっているかを調べる必要がある.

```bash
ls /dev | grep ttyACM*
```
を実行して表示されるものを順番に試していけば良い. 何も表示されなければ何かがおかしい. 挿しなおすといいだろう. ここでは `/dev/ttyACM0` だとする.  
次のコマンドを打てばデータを取得できるはずである.

```bash
sudo chmod a+rw /dev/ttyACM0
roscore
```

新しい端末で以下を実行.

```bash
rosrun urg_node urg_node _serial_port:="/dev/ttyACM0"
```

赤色の文字が出ていたら失敗, そうでなければ上手く行っている.

### Ethernetタイプ
まずコンピュータのEthernetポートにIPを割り当てる. `ifconfig` コマンドを実行すればEthernetポートに割り当てられている名前がわかる. 昔なら `eth0` が, 最近なら `enp0s25` が用いられる. ここでは `enp0s25` という名前が割り当てられていたとする.
次のコマンドを打てばデータを取得できるはずである.

```bash
sudo ifconfig enp0s25 172.168.0.20
roscore
```

新しい端末で以下を実行.

```bash
rosrun urg_node urg_node _ip_address:="172.16.0.10"
```

赤色の文字が出ていたら失敗, そうでなければ上手く行っている.

### データの表示
上手くurg_nodeが立ち上がったなら, [rviz](http://wiki.ros.org/ja/rviz)を用いてデータを可視化して表示することができる. 次のコマンドを打てばrvizが立ち上がる.

```bash
rviz
```

まずはFixed Frameを変更する. デフォルトのmapからlaserに変える.

{{% img src="rviz_fixed_frame.png" caption="Fixed Frame 変更" %}}

そして左下のAddを押す.

{{% img src="rviz_add.png" caption="rviz 追加" %}}

By topicのタブを選択してLaserScanをダブルクリックする.

{{% img src="rviz_urg.png" caption="urg　選択" %}}

するとこんな感じでURGの距離情報が取得できる. 色の付いている各点が取得した位置情報で, 色が強度情報(紫が強く赤が弱い)を表している.

{{% img src="rviz_laser.png" caption="rviz URG" %}}

## 設定の永続化
今まで説明した方法は一時的なやり方. 毎回こんなコマンドを打つのはうざったいので自動で済ませるようにする.

### USBタイプ
USBのデバイスファイル名は接続した順番で変わってしまう. それではあまりにも使いにくいのでデバイス名を固定する.  
まずUSBタイプのURGを接続した状態で `lsusb` コマンドを打つ. すると例えば次のような出力を得られる.

```txt
Bus 002 Device 001: ID 1d6b:0003 Linux Foundation 3.0 root hub
Bus 001 Device 003: ID 056e:00e6 Elecom Co., Ltd
Bus 001 Device 002: ID 1038:1607 SteelSeries ApS
Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
```

SteelSeries ApSが目的のデバイスだとすると, そのデバイスのidVendorが1038でidProductが1607だとわかる. 実際にはなんの説明も書かれていないものがURGだったはずである.  
あとは `/etc/udev/rules.d/99-local.rules` を開いて

```bash
sudo emacs /etc/udev/rules.d/99-local.rules
```

以下を追記すれば良い.

```txt
# URG
KERNEL=="ttyACM*",  ATTRS{idVendor}=="1038", ATTRS{idProduct}=="1607", SYMLINK+="ttyACM_URG"
```

これで `ttyACM_URG` を接続先とすればいける.  
ただし, このままだと権限がないため `sudo` でプログラミングを起動しない限りPermission deniedになる. 毎回 `chmod` で適切な権限を設定するのが面倒くさい場合は, ユーザーのアカウント情報を変更して素のままでもアクセスできるようにする.  
以下のコマンドを打てば良い. ただし `$USER` を各自のユーザーネームに書き換えること(カレントユーザーで設定するならそのままでもよいはず).

```bash
sudo usermod -a -G dialout $USER
```

複数のUSBタイプのURGがある場合の対象は面倒くさそう. たしかidProductは各々のURGに固有の値が割り当てられているのではなく0になっている気がしたので(要検証)[^4], urg_nodeのgetIDなどのライブラリ関数を用いて固有番号を取得するしかないと思われる.

### Ethernetタイプ
Ethernetポートをインターネット接続にはほとんど使わず, インターネットはもっぱら無線lanを介して利用している場合の設定方法. EthernetポートをURGを利用するためだけに設定する.  
まず自分のEthernetポートのインターフェイス名を確認する. `ifconfig` コマンドを打ってeから始まるもの(昔なら `eth*` 今なら `enp*s*` がほとんど)を見つける. ここでは `enp0s25` だとする.  
`/etc/network/interface` を開き

```bash
sudo emacs /etc/network/interface
```

以下を追記する. ただし, `enp0s25` 関連の設定がある場合はそれを削除する.

```bash
auto enp0s25
iface enp0s25 inet static
      address 172.16.0.20
      netmask 255.255.255.0

```

このときgatewayは設定しない. そうすることでインターネットはNIC側で通信するので利用できる. 割り当てるアドレスは `172.16.0.20` でなくてもよい.  
ネットワーク周りの知識は疎いので, これより望ましい設定方法があるかもしれない.

## 実際の利用にあたって
### roslaunchを用いたurg_nodeの起動
urg_nodeにはいくつかのパラメータがあるが, これはコマンドライン引数を使わなくても[rosparam](http://wiki.ros.org/rosparam)を用いて変更することができる. どんなパラメータが存在するのかは, [公式リポジトリのlaunchファイル](https://github.com/ros-drivers/urg_node/blob/indigo-devel/launch/urg_lidar.launch)を見るとわかる. 基本的には `ip_address` か `serial_port` を設定するだけで上手く行くだろう.

### sensor_msgs/LaserScan
URGのデータはsensor_msgs/LaserScanというメッセージ型で送信される. C++では距離情報は `std::vector<float>` 型のrangesというメンバ変数に, 強度情報は同じく `std::vector<float>` 型のintensitiesというメンバ変数に格納されている.  
vector型で格納されているので `<algorithm>` や `<numeric>` の関数を利用するとより効率的にコードをかけるだろう. [algorithm - cpprefjp C++日本語リファレンス](https://cpprefjp.github.io/reference/algorithm.html)や[numeric - cpprefjp C++日本語リファレンス](https://cpprefjp.github.io/reference/numeric.html)を参考にすると良い.

### サンプルコード
需要があるかはわからないがサンプルコードを一応GitHubに公開した. roslaunchで立ち上げれば所定のURGからデータを取得して, その中央の点の距離情報と強度情報を出力する.  
[サンプルコード](https://github.com/BrownieAlice/urg_test/)

## Tips
### 三角関数テーブル
同じURGを使う限り各ステップの角度は固定である. 三角関数は比較的計算量がかかる処理なので, 毎回毎回計算せずにテーブル[^5]として保有し適宜参照するほうが計算時間の短縮に繋がる.

### 論文
URGに関する日本語の論文も結構ある. LRFや測域センサで調べるとそれなりにヒットする. ロボット以外の分野でもそれなりに使われるんだなってことがわかる.

### Raspberry Piで学ぶ ROSロボット入門
読んでないけど[Raspberry Piで学ぶ ROSロボット入門](https://www.amazon.co.jp/Raspberry-Pi%E3%81%A7%E5%AD%A6%E3%81%B6-ROS%E3%83%AD%E3%83%9C%E3%83%83%E3%83%88%E5%85%A5%E9%96%80-%E4%B8%8A%E7%94%B0-%E9%9A%86%E4%B8%80/dp/4822239292?SubscriptionId=AKIAJG2LX7IUECW3UXMQ&tag=ryuichiueda-22&linkCode=xm2&camp=2025&creative=165953&creativeASIN=4822239292)という本に測域センサで地図を作成という章がある. [作者のブログ](https://blog.ueda.tech/?p=9376)を見るとURGでSLAMもしているようなので参考になるのかもしれない.

## 参考サイト

* [北陽電機株式会社](https://www.hokuyo-aut.co.jp/)
* [Documentation - ROS Wiki](http://wiki.ros.org/)
* [URG Network / Wiki / top_jp](https://sourceforge.net/p/urgnetwork/wiki/top_jp/)
* [udevでデバイス名を固定する - Qiita](http://qiita.com/caad1229/items/309be550441515e185c0)
* [デバイス名を固定する - Qiita](http://qiita.com/osada9000/items/3e6ff429ba782624def1)
* [Read/Write to a Serial Port Without Root?](http://unix.stackexchange.com/questions/14354/read-write-to-a-serial-port-without-root)
* [cpprefjp - C++日本語リファレンス](https://cpprefjp.github.io/)

[^1]:長距離検知可能な特性から自動運転車に搭載されていることが多い.
[^2]:[URG Network / Wiki / ROS_jp](https://sourceforge.net/p/urgnetwork/wiki/ROS_jp/)
[^3]:[URG Network / Wiki / ip_address_jp](https://sourceforge.net/p/urgnetwork/wiki/ip_address_jp/)
[^4]:残念ながら現在USB接続タイプのURGを持っていないので試すことができない. 以前使った時の曖昧な記憶によればこうだったはず.
[^5]:配列やstd::arrayなどで予めsinやcosの値を保有しておくこと.
