---
title: "LaTeXでの転置行列の表記"
date: 2018-01-14T00:50:09+09:00
categories:
 - "TechNote"
tags:
  - "latex"
toc: true
thumbnail: "transpose.svg"
---
## 転置記号の候補
$\rm \LaTeX$で転置行列を書こうとするときに, 右上に書く転置記号をどう記載するかで悩んだのでメモしておく.  
$A$行列の転置行列を表記する場合, だいたい次のどれかで書くことが多いと思う.
$$
\begin{align}
A^T \tag{1} \newline
A^\mathrm{T} \tag{2} \newline
A^\mathsf{T} \tag{3} \newline
A^\intercal \tag{4} \newline
A^\top \tag{5}
\end{align}
$$
ソースコードは以下.
```tex
\begin{align}
A^T \\
A^\mathrm{T} \\
A^\mathsf{T} \\
A^\intercal \\
A^\top
\end{align}
```

## 表記法解説
それぞれの表記について簡単に解説する.

1. $A^T$  
もっとも簡単な表記法. イタリック体で記述される.  
この書き方だと転置記号なのか変数$T$なのか区別がつかないという問題がある.

2. $A^\mathrm{T}$  
ローマン体で記述する方法. $\sin$などの演算子や$\mathrm{kg}$などの単位もローマン体で記述する事になっている.  
概ね良さそうだが, セリフ体なのが気になる所か.

3. $A^\mathsf{T}$  
サンセリフ体で記述する方法. よさそう.

4. $A^\intercal$  
`\intercal`コマンドで記述する方法. 位置がやや下気味なのが気になる.  $[a\ b]^\intercal$と記述すると違和感を感じる.  
このコマンドは `amssymb.sty` で定義されている. そのユーザガイドである[User’s Guide to AMSFonts](http://ftp.yz.yamagata-u.ac.jp/pub/CTAN/fonts/amsfonts/doc/amsfndoc.pdf)をみると Binary operators として紹介されているので転置記号として用いるのは違う気がする.

5. $A^\top$  
`\top`コマンドで記述する方法. 文字がかなり細い気がするがおかしくはない.  
ただ[List of mathematical symbols - Wikipedia](https://en.wikipedia.org/wiki/List_of_mathematical_symbols)では束論や型理論で使う記号として説明されている.

## 実際の使われ方
### 本
数学書ではどう使われているのかを見ようと思い[線型代数入門](http://www.utp.or.jp/book/b302039.html)や[解析入門](http://www.utp.or.jp/book/b302042.html)を見たが, ${}^t\\!A$と記述されていたので参考にならなかった.  
ただ工学系では今まで紹介してきた記法がよく用いられると思う. [パターン認識の機械学習 上](https://pub.maruzen.co.jp/book_magazine/book_data/search/9784621061220.html)では$A^\mathrm{T}$(ローマン体)と, [モデル予測制御 制約のもとでの最適制御](https://www.tdupress.jp/bd/isbn/9784501324605/)では$A^T$(イタリック体)と記載されていた. 工学系では雑に$A^T$(イタリック体)とする本も多いように思える.  

### Webサイト
[転置行列 - Wikipedia](https://ja.wikipedia.org/wiki/%E8%BB%A2%E7%BD%AE%E8%A1%8C%E5%88%97)では$A^\top$(`\top`コマンド)が, [Transpose - Wikipedia](https://en.wikipedia.org/wiki/Transpose)では$A^\mathsf{T}$(サンセリフ体)が使われていた.  
また[転置行列の基本的な４つの性質と証明 | 高校数学の美しい物語](https://mathtrain.jp/transpose)では$A^\top$(`\top`コマンド)が, [LaTeXコマンド集 - 行列 (転置行列,対角和,行列式)](http://www.latex-cmd.com/equation/matrix.html)では$A^\mathrm{T}$(ローマン体)が使われていた.

## 規格
### JIS
数学記号はJISZ8201で規格化されているが, 転置行列に関する記述はない.  
一応

> 演算の記号は, 原則として立体とする。

となっているので原則に基づくと$A^\mathrm{T}$(ローマン体)となる気もするが, 転置記号って演算記号にあたるのか微妙な気もする.

### ISO規格
数学記号はISO 80000-2:2009で規格化されている. こちらは転置行列に関する記述もあり, よく見ると$A^\mathsf{T}$(サンセリフ体)が使われているように見える.

## 結論
サンセリフ体である$A^\mathsf{T}$(`A\mathsf{T}`)を使うか, ${}^t\\!A$としてしまうのが良さそう.  
ただ界隈によって無関心だったり, こだわりがあったりすると思うのでそれに従うのがいいのかな. 工学系はいい加減なところも多い気もするのでそこまで考えなくてもいいのかも.

## 参考文献
* [math mode - What is the best symbol for vector/matrix transpose? - TeX - LaTeX Stack Exchange](https://tex.stackexchange.com/questions/30619/what-is-the-best-symbol-for-vector-matrix-transpose/30636)
* [TeXで最も美しい転置記号 - wosugi blog](http://wosugi.hatenablog.com/entry/20110210/1297330034)
