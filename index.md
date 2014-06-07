---
layout: default
---

<!--
          Copyright Hiroaki Nishihara 2014.
 Distributed under the Boost Software License, Version 1.0.
    (See accompanying file LICENSE_1_0.txt or copy at
          http://www.boost.org/LICENSE_1_0.txt)
-->

# Generic Image Library設計ガイド 日本語訳

この文章は[Generic Image Library Design Guide](http://stlab.adobe.com/gil/html/gildesignguide.html)の日本語訳です。
この日本語訳は原文の著者や権利者とまったく関係がありません。
また、原文と訳文の正確さに関して、訳者は一切の責任を負いません。

#### 原文:
Generic Image Library Design Guide  
<http://stlab.adobe.com/gil/html/gildesignguide.html>

#### 原文のVersion:
2.1

#### 訳者:
Hiroaki Nishihara (<hiro.nishihara@gmail.com>)

#### 訳文のLicense:
Boost Software License, Version 1.0

#### 翻訳日時:
2014年6月6日〜

### 目次

1. [概要](src/section_00.html)
2. [Conceptについて](src/section_01.html)
3. Point
4. Channel
5. Color SpaceとLayout
6. Color Base
7. Pixel
8. Pixel Iterator  
    * 基本となるIterator
    * Iteratorアダプタ
    * Pixel間接参照アダプタ
    * Step Iterator
    * Pixelロケータ
    * 2次元画像上のIterator  
9. Image View  
    * メモリ上の画素群からImage Viewを作成する
    * 他のImage ViewからImage Viewを作成する
10. Image
11. 実行時に型を指定するImageとImage View
12. メタ関数とTypedef
13. I/O拡張
14. サンプルコード
    * Pixelレベルの処理
    * 安全のためのバッファを備えた、Imageのコピー
    * ヒストグラム
    * Image Viewの使用  
15. Generic Image Libraryの拡張  
    * 独自のColor Space定義  
    * Color Conversionの多重定義  
    * 独自のChannel Type定義  
    * 独自のImage View定義  
16. より専門的な事項
17. まとめ
