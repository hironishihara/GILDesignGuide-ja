<!--
1. Overview
Images are essential in any image processing, vision and video project,
and yet the variability in image representations makes it difficult to write imaging algorithms that are both generic and efficient.
In this section we will describe some of the challenges that we would like to address.

In the following discussion an image is a 2D array of pixels.
A pixel is a set of color channels that represents the color at a given point in an image.
Each channel represents the value of a color component.

There are two common memory structures for an image.
Interleaved images are represented by grouping the pixels together in memory and interleaving all channels together, whereas planar images keep the channels in separate color planes.
Here is a 4x3 RGB image in which the second pixel of the first row is marked in red, in interleaved form:
and in planar form:
Note also that rows may optionally be aligned resulting in a potential padding at the end of rows.
-->

## 1. 概要

画像処理、ヴィジョン、動画のいずれの研究課題においても画像は最も重要な要素ですが、画像の形式の多様さは、汎用的かつ効率的な画像処理アルゴリズムを記述する際の障害となっています。
この章では、私たちがこれから取り組もうとしている課題について記述したいと思います。

以降の議論では、画像は画素の2次元配列であるものとします。
また、画素は画像中のある点における色を表現する色チャネルのセットとします。
各々の色チャネルはひとつの色成分の値を表現するものとします。

よく用いられる画像のメモリ構造は、2種類あります。
各画素の色チャネルが順番に連続で配置され、なおかつ、全画素がメモリ上でひとつにまとめられているインタリーブ画像と、
各色チャネルの値がそれぞれ個別の色平面として配置されているプラナー画像です。
第1行目の左から2番目の画素を赤で印をつけた4x3のRGB画像は、インタリーブ画像では次のように配置されます。

![インタリーブ画像](http://hironishihara.github.com/GILDesignGuide-ja/src/img/interleaved.jpg "インタリーブ画像")

プラナー画像では次のように配置されます。

![プラナー画像](http://hironishihara.github.com/GILDesignGuide-ja/src/img/planar.jpg "プラナー画像")

各行の末尾には、アラインメントを施した結果として、パディングが配置されているかもしれないことに注意してください。

<!--
Note also that rows may optionally be aligned resulting in a potential padding at the end of rows.
The Generic Image Library (GIL) provides models for images that vary in:
* Structure (planar vs. interleaved)
* Color space and presence of alpha (RGB, RGBA, CMYK, etc.)
* Channel depth (8-bit, 16-bit, etc.)
* Order of channels (RGB vs. BGR, etc.)
* Row alignment policy (no alignment, word-alignment, etc.)
It also supports user-defined models of images, and images whose parameters are specified at run-time.
GIL abstracts image representation from algorithms applied on images and allows us to write the algorithm once and
have it work on any of the above image variations while generating code that is comparable in speed to that
of hand-writing the algorithm for a specific image type.

This document follows bottom-up design. Each section defines concepts that build on top of concepts defined in previous sections.
It is recommended to read the sections in order.
-->

Generic Image Library (GIL)は、次に挙げるような特徴をもつ画像のためのモデルを提供します。

* メモリ構造 (プラナー vs. インタリーブ)
* 色空間とアルファチャネルの有無 (RGB, RGBA, CMYKなど)
* チャネル深度 (8-bit, 16-bitなど)
* チャネル順序 (RGB vs. BGRなど)
* アライメント (アラインメント無し, ワードアラインメントなど)
* ユーザ定義の画像、実行時にパラメータが指定される画像

GILは、画像に適用されるアルゴリズムから画像の形式を抽象化することで、書き上げたアルゴリズムが上に挙げたいずれの形式の画像でも動作することを可能にします。
また同時に、特定の形式の画像に特化したアルゴリズムに匹敵する速度で動作するコードの生成も実現します。

この文章はボトムアップ設計に従っています。
各章では、それ以前の章で定義したコンセプトに基づいて、新たなコンセプトを定義します。
先頭から順に読み進めることを推奨します。
