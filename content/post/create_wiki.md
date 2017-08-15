---
title: "wikiを作成した"
date: 2017-08-16T00:18:07+09:00
categories:
 - "diary"
tags:
 - "Hugo"
toc: true
---

## wikiについて
Blogを作ったはいいものの, 技術的なことをまとめるのはBlogではなくwikiのほうがいいと思ったのでwikiを作りました.  
wikiは [Alice in the Machine - wiki](https://browniealice.github.io/wiki/) に公開してあります.

## wikiシステムについて
ゆくゆくは自前の鯖借りて運用したいと思っていますが, とりあえずGitHub Pagesにて公開できるHPがいいので静的サイトに限定して考えました. また編集は自分1人ができればいいシステムにしようと思いました.

### MDwiki
最初はMDwikiを用いようと思いました. wikiで調べていると, 1人だけで運用しかつ静的サイトという条件だとそれがピッタリのようでした.  
ただ, syntax highlight周りにバグがありリリース版ではたまに無限ループするという自体がありました. それを避けようとして自分でmakeしてみるも生成されたmdwiki.htmlがうまく動作しないので断念しました.

### Hugo
というわけで, MDwikiを断念し静的サイトジェネレータでwiki風なサイトを作ることにしました. まぁBlogと同じシステムがいいだろうというのと, ページが増えて重くなるのは困るという理由です.  
テーマは [bootie-docs](https://progrhyme.github.io/bootie-docs-demo/) を選んだのですが, 上のタブが改行されたときの動作が微妙なのが難点. [LEARN](https://matcornic.github.io/hugo-learn-doc/basics/what-is-this-hugo-theme/) にしようか悩み中.