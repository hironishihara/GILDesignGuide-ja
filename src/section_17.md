<!-- Copyright 2014 Hiroaki Nishihara

     Distributed under the Boost Software License, Version 1.0.
     (See accompanying file LICENSE_1_0.txt or copy at
     http://www.boost.org/LICENSE_1_0.txt)
-->

<!-- Copyright 2008 Lubomir Bourdev and Hailin Jin

     Distributed under the Boost Software License, Version 1.0.
     (See accompanying file LICENSE_1_0.txt or copy at
     http://www.boost.org/LICENSE_1_0.txt)
-->

<!--
    Copyright 2005-2007 Adobe Systems Incorporated
    Distributed under the MIT License (see accompanying file LICENSE_1_0_0.txt
    or a copy at http://stlab.adobe.com/licenses.html)

    Some files are held under additional license.
    Please see "http://stlab.adobe.com/licenses.html" for more information.
-->


<!--
17. Conclusion
-->

## 17. 結び

<!--
The Generic Image Library is designed with the following five goals in mind:
-->

Generic Image Libraryは次に示す5個の目標を念頭にデザインされています。

<!--
Generality.
Abstracts image representations from algorithms on images.
It allows for writing code once and have it work for any image type.

Performance.
Speed has been instrumental to the design of the library.
The generic algorithms provided in the library are in many cases comparable in speed to hand-coding the algorithm for a specific image type.

Flexibility.
Compile-type parameter resolution results in faster code, but severely limits code flexibility.
The library allows for any image parameter to be specified at run time, at a minor performance cost.

Extensibility.
Virtually every construct in GIL can be extended - new channel types, color spaces, layouts, iterators, locators, image views and images can be provided by modeling the corresponding GIL concepts.

Compatibility.
The library is designed as an STL complement.
Generic STL algorithms can be used for pixel manipulation, and they are specifically targeted for optimization.
The library works with existing raw pixel data from another image library.
-->

- 一般性: 画像処理アルゴリズムから画像の形式を抽象化し、記述したコードがどのような形式の画像でも動作できるにします。
- パフォーマンス: このライブラリの設計において、速度は大きな指針です。このライブラリによるジェネリックアルゴリズムは、多くの場合、特定の画像形式に合わせて作られたアルゴリズムと速度で匹敵します。
- 柔軟性: コンパイル時の型解決は高速なコードをもたらしますが、コードの柔軟性に重大な制限が課せられます。このライブラリは、わずかなパフォーマンスコストと引き換えに、あらゆる画像パラメータについて実行時の型指定を可能にしています。
- 拡張性: 関連するGILコンセプトに基づいたModelにすることで、Channel型、Color Space、Layout、Iterator、Locator、Image View、Imageといった、実質的にあらゆるGILコンストラクトが拡張可能です。
- 互換性: このライブラリはSTLを補完するものとしてデザインされています。Pixel操作にはSTLアルゴリズムが用いられており、そのSTLアルゴリズムは特に最適化の対象となっています。このライブラリは、他のライブラリからの生Pixelデータでも動作します。
