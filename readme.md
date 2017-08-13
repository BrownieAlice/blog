# readme
[Hugo](https://gohugo.io/)を用いてGitHub Pages上でBlogを公開しています.
Blogは[Alice in the Machine - Blog](https://browniealice.github.io/blog/)に公開してあります.

## Hugoインストール
[Install Hugo](https://gohugo.io/getting-started/installing)を参考にすれば良い.
Ubuntuのバイナリインストールでいいなら

```bash
snap install hugo
```

で完了.

## Blog環境構築
すでに中身はgit上で公開しているので

```bash
cd ~/Document
git clone https://github.com/BrownieAlice/blog.git
```

でよい.
あとは
`~/Document/blog`
上で作業する.

## 記事の制作
以下が一連の動作.

```bash
hugo new post/hoge.md
hugo server -D -w
hugo undraft content/post/hoge.md
hugo
git add *
git commit -m "new post hoge"
git push origin master
```

`hugo server`
でローカルにサーバーを立ち上げてブラウザ上で確認できるようになる.
[http://localhost:1313/](http://localhost:1313/)がそのリンク先.

そして下書き記事を正式な記事にしてコンパイルし変更をコミットする.

### カテゴリー
カテゴリとして`po`と`popo`を追加する場合, 記事の最初のフロントマターを以下のように設定します.

```yaml
---
title: "hoge"
date: piyo
categories :
 - "po"
 - "popo"
---
```

## 参考サイト
- [GitHub Pagesでブログ立ち上げ - Hugoを使う](https://www.kaitoy.xyz/2015/08/28/using-hugo/)
- [GitHub Pagesの新機能、ソース設定が地味にいい](https://www.kaitoy.xyz/2016/08/18/simpler-github-pages-publishing/)
- [静的サイトジェネレータHugoを使ったサイト構築（コンテンツ編１）](http://staff.feedtailor.jp/2016/05/18/hugo_06/)
