---
title: "LFTPでハマった話"
date: 2020-01-07T00:29:58+09:00
categories:
 - "TechNote"
tags:
 - "server"
thumbnail: "CICD.png"
---

技術的な話というよりただの愚痴に近いです.  

[LFTP](http://lftp.yar.ru/)はFTPやSFTPなどのいくつかのプロトコルに対応したファイル転送ソフトウェアです.  
私の所属する研究室では, 研究室のHPを大学のサーバ代行サービスを用いて大学のサーバ上で動かしています.
そのサーバはファイル転送プロトコルとしてSFTPのみしか受け付けていないため, HPを更新する際はSFTPでファイルを更新しています.  
また, 研究室のHP自体は[Middleman](https://middlemanapp.com/)という静的サイトジェネレーターを利用しているため,

1. ローカル環境でファイルを変更
2. 研究室のGitLabにpush
3. GitLab Runnerによってビルド
4. GitLab Runnerによって成果物がアップロード

という形式でHPを更新しています.  
自動化する都合上, SFTPを直接実行するのは面倒なので前述したLFTPを利用していました.  

GitLab Runnerが新しく立ち上げたDockerコンテナ上でLFTPを動かしてSFTPでサイトを更新するため, SFTPで大学のサーバに入る際は毎回初回アクセスになります.  
すなわち, 毎回新しいホスト鍵を受け入れるかどうか聞かれます.
LFTPはこのような事態も考慮しており `sftp:auto-confirm` を `yes` に設定しておくと, 自動で鍵を受け入れてくれます.  
GitLab Runnerに以下のスクリプトを実行させることで自動でサイトを更新するCD環境を構築できました.

```bash
#!/bin/sh

lftp <<EOF
set cmd:fail-exit true
set ssl:verify-certificate no
set sftp:auto-confirm yes
open -u $DEPLOY_USERNAME,$DEPLOY_PASSWORD sftp://$DEPLOY_URL
lcd build
cd www
mirror -R
exit
EOF
```

というわけで, 快適な環境を構築できたのですが突然動かなくなりました.
なぜ?

前述した鍵を受け入れるオプションは割と力技的な実装で, OpenSSHが *(yes/no)?* と聞いてきたときに *yes* と返すだけのようです.
それがOpenSSH 8.0以降では *(yes/no/[fingerprint])?* と聞いてくるようになり, 従来の文字列マッチにヒットしないような質問文になったようです.
もちろんすでに対応されており, LFTPのバージョン4.9.0からは適切に処理できるようです. [^1][^2]

サイト更新を行うGitLab RunnerのコンテナのイメージにはAlpine Linuxの最新版[^3]を利用していたのですが, 不幸なことにそのソフトウェアリポジトリにはOpenSSH 8.1とLFTP 4.8.4が登録されていました.  
Runnerは起動する度にOpenSSHとLFTPをインストールような設定で動かしていたため, 運悪くOpenSSHの更新だけが反映されたタイミングに巡り合ってしまい突然動かなくなったようです.  
最終的にはAlpine Linuxの開発版(edge)を利用することで解決しましが, どこが問題なのかの突き止めるのに時間がかかって非常に面倒でした...  
必要なソフト全部入れたイメージを作ってそれで動かせって話だとは思うんですけど, これくらいいやろって思ってたんでうーんという感じでした.  
Dockerとかって嫌なところを消し去ってくれて便利ですけど, 変な問題に遭遇すると消し去ってた嫌なところを確認しに行かないといけないんだなというお気持ちになりました.

[^1]: https://github.com/lavv17/lftp/issues/525
[^2]: http://lftp.yar.ru/news.html
[^3]: 当時はv3.11
