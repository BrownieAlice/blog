---
title: "構造化束縛分かってなかった"
date: 2019-01-04T09:46:46+09:00
categories:
 - "TechNote"
tags:
 - "C++"
toc: true
thumbnail: "images/i_do_not_know_structured_bindings/thumnail.png"
---

C++初心者なので, 誤りなどがあるかもしれません><;

## 構造化束縛
C++17には構造化束縛(Structured bindings)という機能が存在します[^0].  
これは, 配列[^1]や`std::tuple`[^2]の各要素, クラス[^union]のpublicな[^3]各メンバ変数を分解して受け取る機能です.
下にサンプルコードを示します.
```C++
#include <tuple>

struct Hoge {
  int i;
  double d;
  unsigned int u;
};

int main() {
  int a[] = {1, 3, 5};
  auto [a1, a2, a3] = a;
  // 配列の各要素をを構造化束縛宣言で取り出す.
  // a1 == 1, a2 == 3, a3 == 5.

  auto t = std::make_tuple(1, "po", 5.0);
  auto [t1, t2, t3] = t;
  // std::tupleの各要素を構造化束縛宣言で取り出す.
  // t1 == 1, t2 == "po", t3 == 5.0.

  Hoge h = {1, 5.0, 100};
  auto [h1, h2, h3] = h;
  // 構造体の各publicメンバ変数を構造化束縛宣言で取り出す.
  // h1 == 1, h2 == 5.0, h3 == 100.
}
```
最初は`int`型の配列の各要素を, 2番目は`std::tuple`の各要素を, 3番目は`Hoge`構造体のpublicな各メンバ変数を分解して受け取っています.  
構造化束縛では`auto`キーワードを使い, 分解されたそれぞれの変数の型は明示的には宣言しません.
そして, 分解して定義されたそれぞれの変数の型が私の直感とは少し異なっていたというお話です.

## 型推論
C++11からは`auto`キーワードを使い, 型推論を用いた変数宣言が出来るようになりました.  
ここでは本筋に関係あるところだけ, 厳密には間違っているような雑な説明だけします.
詳細な話は[Effective Modern C++](https://www.oreilly.co.jp/books/9784873117362/)などを参考にすると良いかと思います.

### `auto`による型推論
ざっくりというと`auto 変数名 = 式`[^4]という変数宣言の場合, この変数の型は式の型から参照と`const`と`volatile`が取り払われたものになります.  
実際に型推論させた変数の型を[Boost.Typeindex](http://boostorg.github.io/type_index/)を用いて型情報を調べてみましょう.
```C++
#include <iostream>
#include <boost/type_index.hpp>

template<typename T1, typename T2>
void output_types() {
  //  T1の型とT2の型を出力する.
  //      T1の型, T2の型
  //  のように出力される.
  std::cout << boost::typeindex::type_id_with_cvr<T1>().pretty_name()
	    << ", "
	    << boost::typeindex::type_id_with_cvr<T2>().pretty_name()
	    << std::endl;
}

int main() {
  int a1 = 0;
  auto b1 = a1;
  output_types<decltype(a1), decltype(b1)>();
  // a1はint型, b1もint型.

  const double a2 = 0;
  auto b2 = a2;
  output_types<decltype(a2), decltype(b2)>();
  // a2はconst double型, b2はdouble型.

  int &a3 = a1;
  auto b3 = a3;
  output_types<decltype(a3), decltype(b3)>();
  // a3はint&型, b3はint型.

  const double &a4 = a2;
  auto b4 = a4;
  output_types<decltype(a4), decltype(b4)>();
  // a4はconst double&型, b4はdouble型.
}
```

出力
```
int, int
double const, double
int&, int
double const&, double
```

出力を見ると, 型推論を用いて宣言した`b1`~`b4`の型はたしかに参照と`const`が取れているのがわかるかと思います.  
きちんとした話をすると, 推論規則は3種類に場合分けできてユニバーサル参照のときは云々……となると思うのですが, 厳密な話は先に上げた本なり規格書なりを参考にしてください.

### `decltype(auto)`による型推論
`decltype(auto) 変数名 = 式`という形で型推論を行うこともできます.  
この場合, `auto`キーワードによる推論ではあった参照や`const`が取り払われる振る舞いがなくなります.  
```C++
#include <iostream>
#include <boost/type_index.hpp>

template<typename T1, typename T2>
void output_types() {
  //  T1の型とT2の型を出力する.
  //      T1の型, T2の型
  //  のように出力される.
  std::cout << boost::typeindex::type_id_with_cvr<T1>().pretty_name()
	    << ", "
	    << boost::typeindex::type_id_with_cvr<T2>().pretty_name()
	    << std::endl;
}

int main() {
  double a5 = 0.0;
  decltype(auto) b5 = a5;
  output_types<decltype(a5), decltype(b5)>();
  // a5はdouble型, b5もdouble型.

  const int a6 = 0;
  decltype(auto) b6 = a6;
  output_types<decltype(a6), decltype(b6)>();
  // a6はconst int型, b6もconst int型.

  const double &a7 = a5;
  decltype(auto) b7 = a7;
  output_types<decltype(a7), decltype(b7)>();
  // a7はconst double&型, b7もconst double&型.
}
```

出力
```
double, double
int const, int const
double const&, double const&
```
`decltype(auto)`で型推論させた変数の型は, 初期化に用いた変数の型と同じ型になっているのがわかります.  
この形式の型推論も丸括弧をつけると振る舞いが変わったりするそうなのですが, その辺の話はよくわかりません><;

## 構造化束縛によって決まる型
クラスのpublicな各メンバ変数を構造化束縛で分解して受け取った時, どのような型になるのでしょうか.  
構造化束縛は`auto`キーワードを使って宣言するので直感的には, 各メンバ変数を`auto`で型推論した型になりそうです.  
しかし, C++17の規格書相当の[N4659](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2017/n4659.pdf)の11.5.4項を見てみると, 分解されたそれぞれの変数の型は元のクラスで定義された型になる[^type]と読み取れます.  
つまり, `auto`を用いた型推論の型になるわけではないということです.
具体的にどのような差があるか見ていきましょう.

```C++
#include <iostream>
#include <boost/type_index.hpp>

template<typename T1, typename T2>
void output_types() {
  //  T1の型とT2の型を出力する.
  //      T1の型, T2の型
  //  のように出力される.
  std::cout << boost::typeindex::type_id_with_cvr<T1>().pretty_name()
	    << ", "
	    << boost::typeindex::type_id_with_cvr<T2>().pretty_name()
	    << std::endl;
}

struct Hoge {
  const int a;
  double &b;
  // const int型, double&型のメンバ変数.
  Hoge(const int a, double &b): a(a), b(b){
 }
};

int main() {
  double d = 3.14;
  Hoge h(1, d);

  auto a1 = h.a;
  auto a2 = h.b;
  // 各要素を直接型推論して受け取る場合.
  auto [b1, b2] = h;
  // 構造化束縛で受け取る場合.

  output_types<decltype(a1), decltype(b1)>();
  // a1はint型, b1はconst int型.
  output_types<decltype(a2), decltype(b2)>();
  // a2はdouble型, b2はdouble&型.

  decltype(auto) c1 = h.a;
  decltype(auto) c2 = h.b;
  // 各要素をdecltype(auto)で直接型推論して受け取る場合.

  output_types<decltype(c1), decltype(b1)>();
  // c1はconst int型, b1もconst int型.
  output_types<decltype(c2), decltype(b2)>();
  // c2はdouble&型, b2もdouble&型.
}
```

出力
```
int, int const
double, double&
int const, int const
double&, double&
```

`Hoge`構造体は, `const int`型のメンバ変数`a`と`double&`型のメンバ変数`b`を持っています.  
直接メンバ変数にアクセスして型推論を用いて変数の初期化を行うと, `const`や参照が取り外され, それぞれの変数の型は`int`と`double`になります.  
一方で構造化束縛で変数を宣言した場合, それぞれの変数の型はメンバ変数の型と同じ`const int`型と`double&`型になります.
この性質は`decltype(auto)`を用いて型推論を行い変数宣言した場合と似ています[^decl].

## 構造化束縛のその他の性質
### `std::tuple`
`std::tuple`を構造化束縛により分解して受け取った場合の型の決定規則は上述したものとは違います.  
分解しようとする型`E`に対し, i番目に受け取った変数の型は`std::tuple_element<i, E>::type`で決まるようです.
この場合も, `auto`の型推論ではあった参照や`const`外しの操作が行われないことがわかります.

### 構造化束縛の構文
構造化束縛の構文は以下のようになります(optはオプション).  
*attribute-specifier-seq*(opt)  *decl-specifier-seq*  *ref-qualifier*(opt)  [*identifier-list*]  *initializer*;  
正直これだとなにがなんだかわからないと思いますが, この構文の中の *decl-specifier-seq* が今まで`auto`キーワードを用いてた部分になります.[^spec]  
*decl-specifier-seq* は型指定子(*type-specifier*)を含むのですが, 構造化束縛の際には型指定子として`auto`しか用いてはいけないようです.
なので

```C++
decltype(auto) [a1, a2, a3] = std::make_tuple(1, 1.3, "po");
```

は構文エラーになります.
`decltype(auto)`に似た規則で型が定まるのに, `decltype(auto)`と書いてはいけないんですね….

## まとめ
構造化束縛はC++17の便利な機能.  
しかし, `auto`キーワードを使って宣言するのに, それぞれの型は`auto`キーワードを使った型推論とは異なった規則で定まる.
とくに参照や`const`, `volatile`がある場合は注意しないといけない.

## 参考文献
* [Effective Modern C++ --C++11/14プログラムを進化させる42項目](https://www.oreilly.co.jp/books/9784873117362/)
* [Working Draft, Standard for Programming Language C++ N4659](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2017/n4659.pdf)
* [cppreference.com](https://ja.cppreference.com/w/)
* [江添亮の詳説C++17](https://ezoeryou.github.io/cpp17book/)

[^0]: 構造化束縛宣言(Structured bindings declarations)のほうが正しいのかもしれませんが, 日本語だとほとんど構造化束縛と呼ばれているので本記事でもこのように記載します.
[^1]: `std::array`ではなくC言語スタイルの配列の方です.
[^2]: `std::tuple`に似た機能を持つクラスも対象です. 雑に言うと`std::tuple_size<T>`が使えたり, `get<i>`メンバ関数か`std::get<i>`で各要素が取り出せるものです. `std::pair`や`std::shared_ptr`(C++20以降)などが当てはまります.
[^union]: 無名共用体(anonymous union)メンバがなく, 全ての非staticなメンバ変数がpublicである(C++17の場合)もしくは…みたいな条件があります.
[^3]: C++20からは, publicなメンバ変数である必要はなくなるようです([N4761](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2018/n4791.pdf) 9.5.5項).
[^4]: ここで考えているのはちゃんと書くと, auto *identifer* = *assignment-expression* という形式ものです(多分).
[^type]: 正確には, 構造化束縛の際に`const`や`volatile`や参照修飾子を用いて宣言しているならこれらも反映した型になります.
[^decl]: どんな状況でも各要素を`decltype(auto)`で型推論させて受け取った場合と同じになる, と言い切っていいのかどうかは(知識不足なので)よくわかりません.
[^spec]: `const auto`や`auto &`なども *decl-specifier-seq* に相当します.
