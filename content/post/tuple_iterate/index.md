---
title: "std::tupleの全要素の走査"
date: 2018-12-29T04:24:53+09:00
categories:
 - "TechNote"
tags:
 - "C++"
thumbnail: "thumnail.png"
---
C++には `std::tuple` というタプル型が存在します.  
タプル型の全要素についてある処理を行いたい場合に, これといったやり方がないように思えます.  
そこで `std::tuple` の全要素の走査のやり方についての個人的にメモしておきます.

## 我流
私がよくやるやり方です.  
constexpr if文を使っているのでC++17以上出ないとコンパイルできません.

```c++
#include <iostream>
#include <tuple>

template <size_t N = 0, typename T>
void iterate_tuple(const T &t) {
  if constexpr(N < std::tuple_size<T>::value) {
      const auto &x = std::get<N>(t);
      std::cout << x << std::endl;
      // ここでやりたい処理を行う(今回は標準出力へ).

      iterate_tuple<N + 1>(t);
  }
}

int main() {
  const auto t1 = std::make_tuple(42, "browniealice", "po", 3.14159);
  iterate_tuple(t1);

  const auto t2 = std::make_tuple("maquinista", 18, 2.718);
  iterate_tuple(t2);
}
```

出力
```
42
browniealice
po
3.14159
maquinista
18
2.718
```

N番目の要素にアクセスするというのを再帰的に行っています.  
constexpr if文を使うことでスッキリした感じになってます.  
インターネットで調べてみると, 結構複雑なやり方で行ってる例が多いですね[^1].  
このやり方ならコードの量も少なく, 直感的でわかりやすいのでいいかなと.

## Boost.Hanaを利用
[Stack Overflow](https://stackoverflow.com/)を眺めていたら, [c++ - iterate over tuple - Stack Overflow](https://stackoverflow.com/questions/1198260/iterate-over-tuple)の解答の中に[Boost.Hana](http://boostorg.github.io/hana/)を使う方法が書いてありました.  
Boost.Hanaって初めて聞きましたが, 我流の方法と同じことは以下のコードで出来るみたいです.

```c++
#include <iostream>
#include <tuple>
#include <boost/hana.hpp>
#include <boost/hana/ext/std/tuple.hpp>

int main() {
  const auto t1 = std::make_tuple(42, "browniealice", "po", 3.14159);
  boost::hana::for_each(t1, [](const auto &x) {std::cout << x << std::endl;});

  const auto t2 = std::make_tuple("maquinista", 18, 2.718);
  boost::hana::for_each(t2, [](const auto &x) {std::cout << x << std::endl;});
}
```

ラムダ式指定するだけで良いんですね…. すごい.  
似たようなライブラリとして[Boost.Fusion](http://boost.org/libs/fusion)の `for_each` 関数があるみたいですが, こっちはラムダ式を引数に取れないのでちょっと使い勝手悪そうですね.  
Boostあんま使いたくない/使えないときは我流のやり方で, Boostガンガン使えるときはBoost.Hanaを利用するやり方でやっていこうかなと思います.  
しかし, Boostってホント色々あるんですね.

[^1]: 2017年より前の情報が大半でC++17機能が使えないので仕方ないと思いますが.
