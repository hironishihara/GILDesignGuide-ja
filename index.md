---
layout: default
---

<!--
          Copyright Hiroaki Nishihara 2014.
 Distributed under the Boost Software License, Version 1.0.
    (See accompanying file LICENSE_1_0.txt or copy at
          http://www.boost.org/LICENSE_1_0.txt)
-->

# Generic Image Library Design Guide 日本語訳

この文章は[Generic Image Library Design Guide](http://stlab.adobe.com/gil/html/gildesignguide.html)の日本語訳です。
この日本語訳は原文の著者や権利者とまったく関係がありません。
また、原文と訳文の正確さに関して、訳者は一切の責任を負いません。

#### 原文:
Generic Image Library Design Guide  
<http://stlab.adobe.com/gil/html/gildesignguide.html>

#### 原文のVersion:
2.1

#### 原文のライセンス:
Boost Software License, Version 1.0 (2008)  
MIT License (2005-2007)

#### 訳者:
Hiroaki Nishihara (<hiro.nishihara@gmail.com>)

#### 訳文のLicense:
Boost Software License, Version 1.0

#### 翻訳日時:
2014年6月6日〜

### 目次

* [序文](src/section_00.html)
* [概要](src/section_01.html)
* [Conceptについて](src/section_02.html)
* [Point](src/section_03.html)
* [Channel](src/section_04.html)
* [Color SpaceとLayout](src/section_05.html)
* [Color Base](src/section_06.html)
* [Pixel](src/section_07.html)
* [Pixel Iterator](src/section_08.html)
    * 基本となるIterator
    * Iteratorアダプタ
    * Pixel間接参照アダプタ
    * Step Iterator
    * Pixelロケータ
    * 2次元画像上のIterator  
* [Image View](src/section_09.html)
    * メモリ上の画素群からImage Viewを作成する
    * 他のImage ViewからImage Viewを作成する
* [Image](src/section_10.html)
* [実行時に型を指定するImageとImage View](src/section_11.html)
* [メタ関数とTypedef](src/section_12.html)
* [I/O拡張](src/section_13.html)
* [サンプルコード](src/section_14.html)
    * Pixelレベルの処理
    * 安全のためのバッファを備えた、Imageのコピー
    * ヒストグラム
    * Image Viewの使用  
* [Generic Image Libraryの拡張](src/section_15.html)
    * 独自のColor Space定義  
    * Color Conversionの多重定義  
    * 独自のChannel Type定義  
    * 独自のImage View定義  
* [より専門的な事項](src/section_16.html)
* [まとめ](src/section_17.html)
