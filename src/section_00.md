---
layout: default
---

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


# Generic Image Library デザインガイド

#### 著者:
Lubomir Bourdev (<bourdev@adobe.com>)  
Hailin Jin (<hljin@adobe.com>)  
Adobe Systems Incorporated

#### Version:
2.1

#### 作成日時:
2007年9月15日  

<!--
This document describes the design of the Generic Image Library,
a C++ image-processing library that abstracts image representation from algorithms on images.
It covers more than you need to know for a causal use of GIL.
You can find a quick, jump-start GIL tutorial on the main GIL page at http://opensource.adobe.com/gil
-->

本文書は、画像に適用するアルゴリズムから画像形式を抽象化するC++画像処理ライブラリのひとつである、Generic Image Libraryの設計について記述しています。
この文章には、GILをカジュアルに使用する場合には必要ない知識まで網羅されています。
使用方法を手早く知りたい場合には、GILのチュートリアル <http://opensource.adobe.com/gil> を参照ください。

<!--
1. Overview
2. About Concepts
3. Point
4. Channel
5. Color Space and Layout
6. Color Base
7. Pixel
8. Pixel Iterator
    * Fundamental Iterator
    * Iterator Adaptor
    * Pixel Dereference Adaptor
    * Step Iterator
    * Pixel Locator
    * Iterator over 2D Image
9. Image View
    * Creating Views from Raw Pixels
    * Creating Image Views from Other Image Views
10. Image
11. Run-time specified Images and Image Views
12. Useful Metafunctions and Typedefs
13. I/O Extension
14. Sample Code
    * Pixel-level Sample Code
    * Creating a Copy of an Image with a Safe Buffer  
    * Histogram  
    * Using Image Views  
15. Extending the Generic Image Library
    * Defining New Color Spaces
    * Overloading Color Conversion
    * Defining New Channel Types
    * Defining New Image Views
16. Technicalities
17. Conclusion
-->

1. [概要](#section_01)
2. [Conceptについて](#section_02)
3. [Point](#section_03)
4. [Channel](#section_04)
5. [Color SpaceとLayout](#section_05)
6. [Color Base](#section_06)
7. [Pixel](#section_07)
8. [Pixel Iterator](#section_08)
    * [基本となるIterator](#section_08_1)
    * [Iteratorアダプタ](#section_08_2)
    * [Pixel間接参照アダプタ](#section_08_3)
    * [ステップIterator](#section_08_4)
    * [Pixel Locator](#section_08_5)
    * [2次元画像上のIterator](#section_08_6)
9. [Image View](#section_09)
    * [メモリ上のPixel生データからのImage View作成](#section_09_1)
    * [他のImage ViewからのImage View作成](#section_09_2)
    * [Image View上で動作するSTL-Styleアルゴリズム](#section_09_3)
10. [Image](#section_10)
11. [実行時に型を指定するImageとImage View](#section_11)
12. [メタ関数とTypedef](#section_12)
13. [I/O Extension](#section_13)
14. [サンプルコード](#section_14)
    * [Pixelレベルの処理](#section_14_1)
    * [安全のためのバッファを備えた、Imageのコピー](#section_14_2)
    * [ヒストグラム](#section_14_3)
    * [Image Viewの使用](#section_14_4)
15. [Generic Image Libraryの拡張](#section_15)
    * [独自のColor Space定義](#section_15_1)
    * [独自のChannel Type定義](#section_15_2)
    * [色変換のオーバーロード](#section_15_3)
    * [独自のImage View定義](#section_15_4)
16. [より専門的な事項](#section_16)
    * [参照Proxyの作成](#section_16_1)
17. [結び](#section_17)

[section_01]: index.md#section_01 "section_01"
[section_02]: index.md#section_02 "section_02"
[section_03]: index.md#section_03 "section_03"
[section_04]: index.md#section_04 "section_04"
[section_05]: index.md#section_05 "section_05"
[section_06]: index.md#section_06 "section_06"
[section_07]: index.md#section_07 "section_07"
[section_08]: index.md#section_08 "section_08"
[section_08_1]: index.md#section_08_1 "section_08_1"
[section_08_2]: index.md#section_08_2 "section_08_2"
[section_08_3]: index.md#section_08_3 "section_08_3"
[section_08_4]: index.md#section_08_4 "section_08_4"
[section_08_5]: index.md#section_08_5 "section_08_5"
[section_09]: index.md#section_09 "section_09"
[section_09_1]: index.md#section_09_1 "section_09_1"
[section_09_2]: index.md#section_09_2 "section_09_2"
[section_09_3]: index.md#section_09_3 "section_09_3"
[section_10]: index.md#section_10 "section_10"
[section_11]: index.md#section_11 "section_11"
[section_12]: index.md#section_12 "section_12"
[section_13]: index.md#section_13 "section_13"
[section_14]: index.md#section_14 "section_14"
[section_14_1]: index.md#section_14_1 "section_14_1"
[section_14_2]: index.md#section_14_2 "section_14_2"
[section_14_3]: index.md#section_14_3 "section_14_3"
[section_14_4]: index.md#section_14_4 "section_14_4"
[section_15]: index.md#section_15 "section_15"
[section_15_1]: index.md#section_15_1 "section_15_1"
[section_15_2]: index.md#section_15_2 "section_15_2"
[section_15_3]: index.md#section_15_3 "section_15_3"
[section_15_4]: index.md#section_15_4 "section_15_4"
[section_16]: index.md#section_16 "section_16"
[section_16_1]: index.md#section_16_1 "section_16_1"
[section_17]: index.md#section_17 "section_17"
