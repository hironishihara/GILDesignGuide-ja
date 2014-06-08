# Generic Image Library設計ガイド

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
この文章では、GILを使う際に必要となる範囲外の知識まで網羅されています。
手早く使用方法を知りたい場合に役立つGILのチュートリアルについては <http://opensource.adobe.com/gil> を参照ください。

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

1. 概要
2. Conceptについて
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
