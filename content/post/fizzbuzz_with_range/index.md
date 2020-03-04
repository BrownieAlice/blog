---
title: "C++のRangeでFizzBuzzしてみた"
date: 2020-03-05T01:10:01+09:00
categories:
 - "TechNote"
tags:
 - "C++"
thumbnail: "thumnail.png"
---

C++初心者の記事なのでいろいろアレだとは思いますがお許しください.

## Rangeライブラリ
C++20ではRangeが新しい標準ライブラリに追加されます.[^0]
残念ながらほとんどのコンパイラは2020年3月現在は対応していないようですが,[^1] 提案者のEric Niebler氏が提案の基礎としてRangeライブラリを作り公開されています.  
今回は, その[range-v3](https://github.com/ericniebler/range-v3)ライブラリを用いてFizzBuzzしてみました.
インストールは簡単で, Arch Linuxであれば

```bash
sudo pacman -S range-v3
```

だけで終わりです.  
ヘッダオンリーのライブラリのようなので, コンパイル時にリンクさせる必要もありません.

## シンプルバージョン
`views::iota`関数で指定の長さの連続整数列を生成し, `views::transform`関数で数値か *Fizz* か *Buzz* か *FizzBuzz* に変換して終わりです.  
ただ, そのままだと標準出力できないので`views::all`関数を利用しています.[^2]
コードは以下の通り.

```C++
#include <iostream>
#include <range/v3/view/iota.hpp>
#include <range/v3/view/transform.hpp>
#include <string>

int main() {
  using namespace ranges;
  std::cout << views::all(views::iota(1, 50) |
                          views::transform([](const auto i) -> std::string {
                            if (i % 15 == 0) {
                              return "Fizz Buzz";
                            } else if (i % 5 == 0) {
                              return "Buzz";
                            } else if (i % 3 == 0) {
                              return "Fizz";
                            } else {
                              return std::to_string(i);
                            }
                          }))
            << std::endl;
}
```

案外普通な感じでつまんないですね.

## おふざけバージョン
シンプルバージョンは面白くなかったので少し凝ってみました.
`views::transform`関数を使うだけだと面白くないので, `views::replace_if`関数[^3]を用いて順番に *FizzBuzz* ・ *Buzz* ・ *Fizz* を生成する感じにしました.  
ただ, そのままでは文字列と数値を混在させることはできないので, variantを使って無理やり両方扱わせています.  
コードは以下の通り.

```C++
#include <iostream>
#include <range/v3/view/iota.hpp>
#include <range/v3/view/replace_if.hpp>
#include <range/v3/view/transform.hpp>
#include <string>
#include <variant>

int main() {
  using namespace ranges;
  std::cout << views::all(views::iota(1, 50) |
                          views::transform([](const auto i) {
                            return std::variant<int, std::string>(i);
                          }) |
                          views::replace_if(
                              [](const auto &i) {
                                return std::holds_alternative<int>(i) &&
                                       std::get<int>(i) % 15 == 0;
                              },
                              "Fizz Buzz") |
                          views::replace_if(
                              [](const auto &i) {
                                return std::holds_alternative<int>(i) &&
                                       std::get<int>(i) % 5 == 0;
                              },
                              "Buzz") |
                          views::replace_if(
                              [](const auto &i) {
                                return std::holds_alternative<int>(i) &&
                                       std::get<int>(i) % 3 == 0;
                              },
                              "Fizz") |
                          views::transform([](const auto &i) {
                            if (std::holds_alternative<int>(i)) {
                              return std::to_string(std::get<int>(i));
                            } else {
                              return std::get<std::string>(i);
                            }
                          }))
            << std::endl;
}
```

なんかカオスなコードになってきて面白いですね.
シンプルバージョンに比べてコンパイル時間もだいぶ増えて, 3秒くらいかかるようになっちゃいました.

分厚い提案書以外の情報があまりないRangeライブラリですが, コンパイラがサポートし始めたらどんどん使っていきたいですね.

## 参考文献
* [ericniebler/range-v3: Range library for C++14/17/20, basis for C++20's std::ranges](https://github.com/ericniebler/range-v3)
* [Standard Ranges – Eric Niebler](http://ericniebler.com/2018/12/05/standard-ra.nges/)
* [The overview of C++20 Range view](https://ezoeryou.github.io/blog/article/2019-01-10-range-view.html)
* [Ranges library (C++20) - cppreference.com](https://en.cppreference.com/w/cpp/ranges)
* [範囲ライブラリ (C++20) - cppreference.com](https://ja.cppreference.com/w/cpp/ranges)

[^0]: [P0896R4 "The One Ranges Proposal"](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2018/p0896r4.pdf)
[^1]: https://en.cppreference.com/w/cpp/compiler_support
[^2]: ここあんまりよくわかってないのでおまじない感覚で入れてます...
[^3]: この関数はC++20のRangeライブラリに導入されるのかよくわかりません
