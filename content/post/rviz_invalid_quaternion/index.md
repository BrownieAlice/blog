---
title: "rvizとquaternion"
date: 2018-01-02T00:09:13+09:00
categories:
 - "TechNote"
tags:
 - "ROS"
thumbnail: "thumnail.png"
---

前までやってたとおりにrviz上でvisualization_msgsのMarkerを表示させようとしたら表示されない. よくよく見てみると

> Contains invalid quaternions (length not equal to 1)!

ってエラーメッセージが書いてある…なにこれ.

{{% img src="error.png" caption="エラー" %}}

Quaternionのエラーらしいく, 大きさが0じゃないといけないらしい. [visualization_msgs/Marker](http://docs.ros.org/api/visualization_msgs/html/msg/Marker.html)のメンバ変数のposeは[geometry_msgs/Pose](http://docs.ros.org/api/geometry_msgs/html/msg/Pose.html)型で, Quaternion型のorientationというメンバ変数を持っている. これが0じゃないといけないらしい.  
こうじゃなくて

```C++
visualization_msgs::Marker marker;
marker.pose.orientation.x = 0.0;
marker.pose.orientation.y = 0.0;
marker.pose.orientation.z = 0.0;
marker.pose.orientation.w = 0.0;
```

こういう風にするべきとのこと.

```C++
visualization_msgs::Marker marker;
marker.pose.orientation.x = 0.0;
marker.pose.orientation.y = 0.0;
marker.pose.orientation.z = 0.0;
marker.pose.orientation.w = 1.0;
```

前まで問題なく出来てたのにおかしいなーと思ったら, [quaternionsをチェックするプルリク](https://github.com/ros-visualization/rviz/pull/1167/files/3905e151679b9bc21bac54af7f01ad165b755e62)が2017年の12月頃にマージされたみたい. 通りで前と勝手が変わったわけだ.  
Quaternionがよくわからないので調べてみる. [Quaternionによる3次元の回転変換 - Qiita](https://qiita.com/kenjihiranabe/items/945232fbde58fab45681)によれば, 無回転の場合は `w = 1.0` にすればいいみたい. 素人には全然感覚がわからないので難しい….
