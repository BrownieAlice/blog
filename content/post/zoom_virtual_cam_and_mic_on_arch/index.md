---
title: "Arch Linux上でZoom用の仮想カメラ/マイクを作る"
date: 2020-08-26T15:39:22+09:00
categories:
  - "TechNote"
thumbnail: "zoom.png"
---

新型コロナウイルスの影響で卒論発表が Zoom で開催されることとなり, 従来研究室の学生が行っていたタイムキーパーも Zoom 上で行うことになりました.  
ブラウザ上に[Time Keeper](https://maruta.github.io/timekeeper/)を表示させ, その画面と音を仮想カメラ/マイクとして Zoom 上に流し込む形式にしました.
行ったことは[A virtual webcam - Getkey](https://getkey.eu/blog/55106b9f/a-virtual-webcam)の内容とほぼ同じですが, 備忘録として書き留めておきます.

## 必要ソフトのインストールと起動

まず以下のソフトをインストールします.

```sh
sudo pacman -S ffmpeg pavucontrol linux-headers v4l2loopback-dkms
```

次に仮想カメラと仮想マイクを起動します.[^launch]

```
sudo modprobe v4l2loopback
sudo modprobe snd-aloop
```

## 仮想カメラの起動

まず `v4l2-ctl --list-devices` で起動した仮想カメラを確認します.
私の環境では以下のようになりました.

```txt
Dummy video device (0x0000) (platform:v4l2loopback-000):
	/dev/video2

HD Pro Webcam C920 (usb-0000:0b:00.1-5):
	/dev/video0
	/dev/video1
```

`/dev/video2` が仮想カメラのデバイスとわかったので, ffmpeg でスクリーンをキャプチャして流し込みます.

```sh
ffmpeg -f x11grab -s 1920x1080 -i $DISPLAY+0,0 -vf format=pix_fmts=yuv420p -f v4l2 /dev/video2
```

## 仮想マイクの起動

最初にブラウザの音声を「内部オーディオ アナログステレオ」に出力させます.  
`pavucontrol` を起動し再生タブを開きます.
一時的にブラウザから音を鳴らし, 出力先を「内部オーディオ アナログステレオ」に変更します.

{{% img src="pavucontrol.png" caption="オーディオ設定" %}}

次に仮想的なスピーカへの出力を仮想的なマイクへの入力にループさせます.[^sound]
`cat /proc/asound/cards` でサウンドカードを確認します.
私の環境では以下のようになりました.

```txt
 0 [NVidia         ]: HDA-Intel - HDA NVidia
                      HDA NVidia at 0xf7080000 irq 109
 1 [Generic        ]: HDA-Intel - HD-Audio Generic
                      HD-Audio Generic at 0xf7b00000 irq 111
 2 [C920           ]: USB-Audio - HD Pro Webcam C920
                      HD Pro Webcam C920 at usb-0000:0b:00.1-5, high speed
 3 [S5             ]: USB-Audio - SteelSeries Arctis 5
                      SteelSeries SteelSeries Arctis 5 at usb-0000:0b:00.3-4, full speed
 4 [Loopback       ]: Loopback - Loopback
                      Loopback 1
```

4 番が仮想マイクになっているので, ffmpeg でループさせます.

```sh
ffmpeg -f alsa -i hw:4,1 -f wav -f alsa hw:4,1
```

## Zoom 設定

あとは Zoom 側でカメラとして「Dummy video device」を選択し, マイクとして「内部オーディオ アナログステレオ」を選択すれば完了です.

{{% img src="zoom.png" caption="タイムキーパー" %}}

## その他のビデオ会議ソフト

Zoom 以外のソフトの場合, ffmpeg を用いずに pulse audio の機能で仮想マイクを作ることができます.  
`pavucontrol` の録音タブから入力デバイスを「Monitor of 内部オーディオ アナログステレオ」にすれば ffmpeg でループさせる必要はなくなります.  
Zoom の場合は上述の操作は反映されませんが, Chromium 上で動く Google Meet の場合はこの操作で問題ありません.  
一方 Chromium は v4l2loopback に追加のパラメータを渡す必要があるようなので注意が必要です. [^chromium]

## 参考文献

- [A virtual webcam - Getkey](https://getkey.eu/blog/55106b9f/a-virtual-webcam)
- [#linux v4l2loopback で画面キャプチャをカメラ入力にする - Kotet's Personal Blog](https://blog.kotet.jp/2020/04/v4l2loopback/)

[^launch]: このやり方では再起動するたびに `modprobe` で起動する必要が有ります.
[^sound]: 適切に設定すれば ffmpeg を使わずにループできそうな気がしますが, 私にはわかりませんでした.
[^chromium]: [Cannot see the v4l2loopback using google-chrome or chromium · Issue #78 · umlaeute/v4l2loopback](https://github.com/umlaeute/v4l2loopback/issues/78)
