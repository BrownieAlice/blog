---
title: "Disqusを導入した"
date: 2017-08-16T08:35:22+09:00
categories:
 - "TechNote"
tags:
 - "Hugo"
toc: true
thumbnail: "images/disqus_introduce/icon.svg"
---
ヘッダ画像は [Disqus](https://disqus.com/) の [Logo](https://disqus.com/brand/) より引用.
## Disqusとは
[Disqus](https://disqus.com/) はブログのコメントシステムです. このブログはWebサービス型のブログは用いていないので, 当たり前ですがコメントシステムはありません.  
ただブログにはコメントが必要だろうということで, コメントシステムを設けたいと思いました. 調べてみると色々とシステムはあるようですがDisqusが一番メジャーみたいです.

## 導入
導入は非常に簡単でした. というのもDisqusが相当流行っているので, Hugoも使用したテーマもDisqusに対応していたのでちょちょっとやるだけでした.  
まず [Disqus](https://disqus.com/) の `GET STARTED` より登録します. いろいろと設定していくと `Shortname` というのが渡されます. それを用いて `config.toml` を以下のように設定してやれば導入終了です.

```toml
disqusShortname = "hoge"
```

簡単.

## 所感
ただDisqus自体があまり匿名性を重視したサービスじゃないのが微妙かもしれないです. やっぱインターネットといったら匿名なのが利点なので, そうじゃないシステムはあまり好きじゃないかな….  
それとコメントのガイドラインを求められるのでいつか設定しておかないとなという感じです. 現状何も指定していないので….

