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
2014年6月6日〜2014年6月17日

#### Repository:
<https://github.com/hironishihara/GILDesignGuide-ja>

***


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


<!--
1. Overview
-->

## <a name="section_01"> 1. 概要

<!--
Images are essential in any image processing, vision and video project,
and yet the variability in image representations makes it difficult to write imaging algorithms that are both generic and efficient.
In this section we will describe some of the challenges that we would like to address.
-->

画像処理、ヴィジョン、動画のいずれの研究課題においても画像は最も重要な要素ですが、画像の形式の多様さは、汎用的かつ効率的な画像処理アルゴリズムを記述する際の障害となっています。
この章では、私たちがこれから取り組む課題について記述します。

<!--
In the following discussion an image is a 2D array of pixels.
A pixel is a set of color channels that represents the color at a given point in an image.
Each channel represents the value of a color component.
-->

以降の議論では、画像はPixelの2次元配列であるものとします。
また、Pixelは画像中のある点における色を表現する色Channelのセットとします。
各々の色Channelはひとつの色成分の値を表現するものとします。

<!--
There are two common memory structures for an image.
Interleaved images are represented by grouping the pixels together in memory and interleaving all channels together, whereas planar images keep the channels in separate color planes.
Here is a 4x3 RGB image in which the second pixel of the first row is marked in red, in interleaved form:
-->

画像のメモリ構造は、よく用いられる2種類があります。
各Pixelの色Channelが順番に連続で配置され、なおかつ、全Pixelがメモリ上でひとつにまとめられているインタリーブ画像と、
各色Channelの値がそれぞれ個別の色平面として配置されているプラナー画像です。
第1行目の左から2番目のPixelを赤で印をつけた4x3のRGB画像は、インタリーブ画像では次のように配置されます。

![インタリーブ画像](http://hironishihara.github.com/GILDesignGuide-ja/src/img/interleaved.jpg "インタリーブ画像")

<!--
and in planar form:
-->

プラナー画像では次のように配置されます。

![プラナー画像](http://hironishihara.github.com/GILDesignGuide-ja/src/img/planar.jpg "プラナー画像")

<!--
Note also that rows may optionally be aligned resulting in a potential padding at the end of rows.
-->

各行の末尾には、アラインメントを施した結果として、パディングが配置されているかもしれないことに注意してください。

<!--
The Generic Image Library (GIL) provides models for images that vary in:

Structure (planar vs. interleaved)
Color space and presence of alpha (RGB, RGBA, CMYK, etc.)
Channel depth (8-bit, 16-bit, etc.)
Order of channels (RGB vs. BGR, etc.)
Row alignment policy (no alignment, word-alignment, etc.)
-->

Generic Image Library (GIL)は、次に挙げるような特徴をもつ画像のためのModelを提供します。

* メモリ構造 (プラナー vs. インタリーブ)
* 色空間とアルファChannelの有無 (RGB, RGBA, CMYKなど)
* Channel深度 (8-bit, 16-bitなど)
* Channel順序 (RGB vs. BGRなど)
* アライメント (アラインメント無し, ワードアラインメントなど)
* ユーザ定義の画像、実行時にパラメータが指定される画像

<!--
It also supports user-defined models of images, and images whose parameters are specified at run-time.
GIL abstracts image representation from algorithms applied on images and allows us to write the algorithm once and
have it work on any of the above image variations while generating code that is comparable in speed to that
of hand-writing the algorithm for a specific image type.
-->

また、ユーザ定義の画像Modelや実行時にパラメータが決まる画像もサポートしています。
GILは、画像に適用されるアルゴリズムから画像の形式を抽象化することで、書き上げたアルゴリズムが上に挙げたいずれの形式の画像でも動作することを可能にします。
また同時に、特定の形式の画像に特化したアルゴリズムに匹敵する速度で動作するコードの生成も実現します。

<!--
This document follows bottom-up design. Each section defines concepts that build on top of concepts defined in previous sections.
It is recommended to read the sections in order.
-->

この文章はボトムアップ設計に従っています。
各章では、それより前の章で定義したConceptに基づいて、新たなConceptを定義していきます。
先頭の章から順に読み進めることを推奨します。


<!--
2. About Concepts
-->

## <a name="section_02"> 2. Conceptについて

<!--
All constructs in GIL are models of GIL concepts.
A concept is a set of requirements that a type (or a set of related types) must fulfill to be used correctly in generic algorithms.
The requirements include syntactic and algorithming guarantees.
For example, GIL's class pixel is a model of GIL's PixelConcept.
The user may substitute the pixel class with one of their own, and, as long as it satisfies the requirements of PixelConcept, all other GIL classes and algorithms can be used with it.
See more about concepts here: http://www.generic-programming.org/languages/conceptcpp/
-->

GILで用いられる全ての構成概念(コンストラクト)は、GILが定めるConceptに基づいたModelです。
Conceptとは、型(もしくは、関連する型のセット)がジェネリックアルゴリズム内で正しく利用されるために満たさなければならない要件のセットです。
これらの要件には、構文的な保証とアルゴリズム的な保証が含まれます。
例えば、GILコンストラクトのひとつであるPixelクラスは、GILの`PixelConcept`に基づいたModelです。
`PixelConcept`に示された要件を満たす限りにおいて、ユーザはPixelクラスを独自のPixelクラスに置き換えることができ、その独自のPixelクラスを他のGILクラスやアルゴリズムと共に使用することができます。
Conceptに関する詳細は、次のURLを参照ください。  
<http://www.generic-programming.org/languages/conceptcpp/>  

<!--
In this document we will use a syntax for defining concepts that is described in a proposal
for a Concepts extension to C++0x specified here: http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2006/n2081.pdf
-->

この文章では、次のURLにあるC++0xのConcept拡張の提案書に記述されている、Concept定義のための構文を使用します。  
<http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2006/n2081.pdf>

<!--
Here are some common concepts that will be used in GIL.
Most of them are defined here: http://www.generic-programming.org/languages/conceptcpp/concept_web.php
-->

ここで、GILでよく用いられるいくつかのConceptを紹介します。
そのほとんどは、次のサイトで定義されています。  
<http://www.generic-programming.org/languages/conceptcpp/concept_web.php>  

{% highlight C++ %}

auto concept DefaultConstructible<typename T> {
    T::T();
};

auto concept CopyConstructible<typename T> {
    T::T(T);
    T::~T();
};

auto concept Assignable<typename T, typename U = T> {
    typename result_type;
    result_type operator=(T&, U);
};

auto concept EqualityComparable<typename T, typename U = T> {
    bool operator==(T x, T y);
    bool operator!=(T x, T y) { return !(x==y); }
};

concept SameType<typename T, typename U> {  unspecified  };
template<typename T> concept_map SameType<T, T> {  unspecified  };

auto concept Swappable<typename T> {
    void swap(T& t, T& u);
};

{% endhighlight %}

<!--
Here are some additional basic concepts that GIL needs:
-->

また、GILが必要とする基本的なConceptを追加でいくつか挙げておきます。

{% highlight C++ %}

auto concept Regular<typename T> : DefaultConstructible<T>, CopyConstructible<T>, EqualityComparable<T>, Assignable<T>, Swappable<T> {};

auto concept Metafunction<typename T> {
    typename type;
};

{% endhighlight %}


<!--
3. Point
-->

## <a name="section_03"> 3. Point

<!--
A point defines the location of a pixel inside an image.
It can also be used to describe the dimensions of an image.
In most general terms, points are N-dimensional and model the following concept:
-->

Pointは、Pixelの画像上における位置を定義します。
また、画像の次元数を表現するためにも用いられます。
一般に、PointはN次元であり、次に示すConceptに基づいたModelです。

{% highlight C++ %}

concept PointNDConcept<typename T> : Regular<T> {
    // the type of a coordinate along each axis
    template <size_t K> struct axis; where Metafunction<axis>;

    const size_t num_dimensions;

    // accessor/modifier of the value of each axis.
    template <size_t K> const typename axis<K>::type& T::axis_value() const;
    template <size_t K>       typename axis<K>::type& T::axis_value();
};

{% endhighlight %}

<!--
GIL uses a two-dimensional point, which is a refinement of PointNDConcept in which both dimensions are of the same type:
-->

GILは、2つの次元の座標の型が同じになるように改良した`PointNDConcept`である、2次元のPointを用います。

{% highlight C++ %}

concept Point2DConcept<typename T> : PointNDConcept<T> {
    where num_dimensions == 2;
    where SameType<axis<0>::type, axis<1>::type>;

    typename value_type = axis<0>::type;

    const value_type& operator[](const T&, size_t i);
          value_type& operator[](      T&, size_t i);

    value_type x,y;
};

{% endhighlight %}

<!--
Related Concepts:

PointNDConcept<T>
Point2DConcept<T>
-->

#### 関連するConcept:

- `PointNDConcept<T>`
- `Point2DConcept<T>`

<!--
Models:

GIL provides a model of Point2DConcept, point2<T> where T is the coordinate type.
-->

#### Model:

GILは、`Point2DConcept`に基づいたModelである`point2<T>`を提供します。この`T`は座標の型を表しています。


<!--
4. Channel
-->

## <a name="section_04"> 4. Channel

<!--
A channel indicates the intensity of a color component (for example, the red channel in an RGB pixel).
Typical channel operations are getting, comparing and setting the channel values.
Channels have associated minimum and maximum value.
GIL channels model the following concept:
-->

Channelは、色成分の強度を示します (例: RGB Pixelの赤Channel)。
基本的なChannel操作として、値の取得(get)や比較(compare)や代入(set)があります。
また、Channelには最小値と最大値が設定されています。
GILのChannelは、次に示すConceptに基づいたModelです。

{% highlight C++ %}

concept ChannelConcept<typename T> : EqualityComparable<T> {
    typename value_type      = T;        // use channel_traits<T>::value_type to access it
       where ChannelValueConcept<value_type>;
    typename reference       = T&;       // use channel_traits<T>::reference to access it
    typename pointer         = T*;       // use channel_traits<T>::pointer to access it
    typename const_reference = const T&; // use channel_traits<T>::const_reference to access it
    typename const_pointer   = const T*; // use channel_traits<T>::const_pointer to access it
    static const bool is_mutable;        // use channel_traits<T>::is_mutable to access it

    static T min_value();                // use channel_traits<T>::min_value to access it
    static T max_value();                // use channel_traits<T>::min_value to access it
};

concept MutableChannelConcept<ChannelConcept T> : Swappable<T>, Assignable<T> {};

concept ChannelValueConcept<ChannelConcept T> : Regular<T> {};

{% endhighlight %}

<!--
GIL allows built-in integral and floating point types to be channels.
Therefore the associated types and range information are defined in channel_traits with the following default implementation:
-->

GILは、組み込みの整数型と浮動小数点型をChannelとして認めています。
そのため、Channelに関連づけられた型とレンジ情報に関する情報は`channel_traits`で定義されています。
`channel_traits`のデフォルトの実装を次に示します。

{% highlight C++ %}

template <typename T>
struct channel_traits {
    typedef T         value_type;
    typedef T&        reference;
    typedef T*        pointer;
    typedef T& const  const_reference;
    typedef T* const  const_pointer;

    static value_type min_value() { return std::numeric_limits<T>::min(); }
    static value_type max_value() { return std::numeric_limits<T>::max(); }
};

{% endhighlight %}

<!--
Two channel types are compatible if they have the same value type:
-->

ふたつのChannelが同じ`value_type`をもつ場合、そのふたつのChannelには互換性があります。

{% highlight C++ %}

concept ChannelsCompatibleConcept<ChannelConcept T1, ChannelConcept T2> {
    where SameType<T1::value_type, T2::value_type>;
};

{% endhighlight %}

<!--
A channel may be convertible to another channel:
-->

また、あるChannelが他のChannelに変換可能な場合もあります。

{% highlight C++ %}

template <ChannelConcept Src, ChannelValueConcept Dst>
concept ChannelConvertibleConcept {
    Dst channel_convert(Src);
};

{% endhighlight %}

<!--
Note that ChannelConcept and MutableChannelConcept do not require a default constructor.
Channels that also support default construction (and thus are regular types) model ChannelValueConcept.
To understand the motivation for this distinction, consider a 16-bit RGB pixel in a "565" bit pattern.
Its channels correspond to bit ranges.
To support such channels, we need to create a custom proxy class corresponding to a reference to a subbyte channel.
Such a proxy reference class models only ChannelConcept, because, similar to native C++ references, it may not have a default constructor.
-->

`ChannelConcept`と`MutableChannelConcept`が、デフォルトコンストラクタを要求していないことに注意してください。
デフォルトコンストラクタをサポートする(その結果、正則型である)Channelは、`ChannelValueConcept`に基づいたModelです。
このような区別を設けた動機を理解するために、"565"のビットパターンをもつ16bit RGB Pixelを考えます。
各Channelは、それぞれのビットレンジに対応しています。
このようなChannelをサポートするためには、バイト境界をまたがるChannel参照にも対応する特別なProxyクラスをつくる必要があります。
このときProxy参照クラスは`ChannelConcept`だけに従って実装されます。
なぜなら、このようなChannelは、C++における参照のように、デフォルトコンストラクタをもたない可能性があるからです。

<!--
Note also that algorithms may impose additional requirements on channels, such as support for arithmentic operations.
-->

また、アルゴリズムが、算術演算子のサポートなど、追加の要件を課すかもしれないことにも注意が必要です。

<!--
Related Concepts:

ChannelConcept<T>
ChannelValueConcept<T>
MutableChannelConcept<T>
ChannelsCompatibleConcept<T1,T2>
ChannelConvertibleConcept<SrcChannel,DstChannel>
-->

#### 関連するConcept:

- `ChannelConcept<T>`
- `ChannelValueConcept<T>`
- `MutableChannelConcept<T>`
- `ChannelsCompatibleConcept<T1,T2>`
- `ChannelConvertibleConcept<SrcChannel,DstChannel>`

<!--
Models:

All built-in integral and floating point types are valid channels.
GIL provides standard typedefs for some integral channels:
-->

#### Model:

組み込みの整数型と浮動小数点型は、全て、有効なChannelです。
GILは、いくつかの整数型について、標準の`typedef`を提供しています。

{% highlight C++ %}

typedef boost::uint8_t  bits8;
typedef boost::uint16_t bits16;
typedef boost::uint32_t bits32;
typedef boost::int8_t   bits8s;
typedef boost::int16_t  bits16s;
typedef boost::int32_t  bits32s;

{% endhighlight %}

<!--
The minimum and maximum values of a channel modeled by a built-in type correspond to the minimum and maximum physical range of the built-in type, as specified by its std::numeric_limits.
Sometimes the physical range is not appropriate. GIL provides scoped_channel_value, a model for a channel adapter that allows for specifying a custom range.
We use it to define a [0..1] floating point channel type as follows:
-->

組み込み型を用いたChannelの最小値と最大値は、その型の`std::numeric_limits`で定められている、組み込み型のフィジカルレンジに由来する最小値と最大値に対応しています。
しかし、状況によってはフィジカルレンジが適切でない場合もあります。
GILは、特別なレンジを定めるためのChannelアダプタのModelである、`scoped_channel_value`を提供します。
私たちは、[0..1]の浮動小数点を定義するために、`scoped_channel_value`を次のように用います。

{% highlight C++ %}

struct float_zero { static float apply() { return 0.0f; } };
struct float_one  { static float apply() { return 1.0f; } };
typedef scoped_channel_value<float,float_zero,float_one> bits32f;

{% endhighlight %}

<!--
GIL also provides models for channels corresponding to ranges of bits:
-->

GILは、ビット単位のレンジをもつChannelのためのModelも提供しています。

{% highlight C++ %}

// Value of a channel defined over NumBits bits. Models ChannelValueConcept
template <int NumBits> class packed_channel_value;

// Reference to a channel defined over NumBits bits. Models ChannelConcept
template <int FirstBit,
          int NumBits,       // Defines the sequence of bits in the data value that contain the channel
          bool Mutable>      // true if the reference is mutable
class packed_channel_reference;

// Reference to a channel defined over NumBits bits.
Its FirstBit is a run-time parameter.
Models ChannelConcept
template <int NumBits,       // Defines the sequence of bits in the data value that contain the channel
          bool Mutable>      // true if the reference is mutable
class packed_dynamic_channel_reference;

{% endhighlight %}

<!--
Note that there are two models of a reference proxy which differ based on whether the offset of the channel range is specified as a template or a run-time parameter.
The first model is faster and more compact while the second model is more flexible.
For example, the second model allows us to construct an iterator over bitrange channels.
-->

各Channelレンジまでのオフセットについて、テンプレートで指定する参照Proxyと実行時のパラメータで指定する参照Proxyの異なる2種類の参照Proxy Modelがあることに注意してください。
前者は軽快かつコンパクトなModelであり、後者はより適応性のあるModelです。
例を挙げると、後者のModelはビット単位のレンジをもつChannel上で動作するIteratorを構築することができます。

<!--
Algorithms:

Here is how to construct the three channels of a 16-bit "565" pixel and set them to their maximum value:
-->

#### Algorithms:

"565"のビットパターンをもつ16bit Pixelを構築し、各Channelに最大値を代入する方法を示します。

{% highlight C++ %}

typedef packed_channel_reference<0,5,true> channel16_0_5_reference_t;
typedef packed_channel_reference<5,6,true> channel16_5_6_reference_t;
typedef packed_channel_reference<11,5,true> channel16_11_5_reference_t;

boost::uint16_t data=0;
channel16_0_5_reference_t   channel1(&data);
channel16_5_6_reference_t   channel2(&data);
channel16_11_5_reference_t  channel3(&data);

channel1=channel_traits<channel16_0_5_reference_t>::max_value();
channel2=channel_traits<channel16_5_6_reference_t>::max_value();
channel3=channel_traits<channel16_11_5_reference_t>::max_value();
assert(data==65535);

{% endhighlight %}

<!--
Assignment, equality comparison and copy construction are defined only between compatible channels:
-->

代入、比較、コピーコンストラクタは、互換性をもつChannel間にだけ定義されます。

{% highlight C++ %}

packed_channel_value<5> channel_6bit = channel1;
channel_6bit = channel3;

//channel_6bit = channel2; // compile error: Assignment between incompatible channels.

{% endhighlight %}

<!--
All channel models provided by GIL are pairwise convertible:
-->

GILによって提供される全てのChannel Modelが互いに変換可能です。

{% highlight C++ %}

channel1 = channel_traits<channel16_0_5_reference_t>::max_value();
assert(channel1 == 31);

bits16 chan16 = channel_convert<bits16>(channel1);
assert(chan16 == 65535);

{% endhighlight %}

<!--
Channel conversion is a lossy operation.
GIL's channel conversion is a linear transformation between the ranges of the source and destination channel.
It maps precisely the minimum to the minimum and the maximum to the maximum.
(For example, to convert from uint8_t to uint16_t GIL does not do a bit shift because it will not properly match the maximum values.
  Instead GIL multiplies the source by 257).
-->

Channel変換は、不可逆な操作です。
GILのChannel変換は、変換元Channelのレンジと変換先Channelのレンジとの線形変換です。
最小値と最小値、最大値と最大値がぴったりと対応します。
(ひとつ例を挙げます。GILは`uint8_t`から`uint16_t`への変換に際してビットシフトは行いません。というのも、ビットシフトでは最大値がぴったり一致しない可能性があるからです。
そのかわりに、GILは変換元のChannel値と257の積を求めます。)

<!--
All channel models that GIL provides are convertible from/to an integral or floating point type.
Thus they support arithmetic operations.
Here are the channel-level algorithms that GIL provides:
-->

GLが提供する全てのChannel Modelは、整数型と実数型の間で相互に変換可能です。
そして、これらのChannel Modelは算術演算子をサポートしています。
ここで、GILが提供するChannelレベルのアルゴリズムを示します。

{% highlight C++ %}

// Converts a source channel value into a destrination channel. Linearly maps the value of the source
// into the range of the destination
template <typename DstChannel, typename SrcChannel>
typename channel_traits<DstChannel>::value_type channel_convert(SrcChannel src);

// returns max_value - x + min_value
template <typename Channel>
typename channel_traits<Channel>::value_type channel_invert(Channel x);

// returns a * b / max_value
template <typename Channel>
typename channel_traits<Channel>::value_type channel_multiply(Channel a, Channel b);

{% endhighlight %}


<!--
5. Color Space and Layout
-->

## <a name="section_05"> 5. Color SpaceとLayout

<!--
A color space captures the set and interpretation of channels comprising a pixel.
It is an MPL random access sequence containing the types of all elements in the color space.
Two color spaces are considered compatible if they are equal (i.e. have the same set of colors in the same order).
-->

Color Spaceは、Pixelを構成するChannelに関して、それらの組み合わせと解釈を保持します。
Color Spaceは、そのColor Spaceがもつ全ての要素の型を包含したMPLランダムアクセスシークエンスです。
ふたつのColor Spaceが等しい(同じ色のセットを同じ順序でもつ)場合に限って、それらのColor Space間には互換性があると見なされます。

<!--
Related Concepts:

ColorSpaceConcept<ColorSpace>
ColorSpacesCompatibleConcept<ColorSpace1,ColorSpace2>
ChannelMappingConcept<Mapping>
-->

#### 関連するConcept:

- `ColorSpaceConcept<ColorSpace>`
- `ColorSpacesCompatibleConcept<ColorSpace1,ColorSpace2>`
- `ChannelMappingConcept<Mapping>`

<!--
Models:

GIL currently provides the following color spaces: gray_t, rgb_t, rgba_t, and cmyk_t.
It also provides unnamed N-channel color spaces of two to five channels, devicen_t<2>, devicen_t<3>, devicen_t<4>, devicen_t<5>.
Besides the standard layouts, it provides bgr_layout_t, bgra_layout_t, abgr_layout_t and argb_layout_t.
-->

#### Model:

現在のところ、GILは`gray_t`、`rgb_t`、`rgba_t`、`cmyk_t`といったColor Spaceを提供しています。
また、2〜5個までのChannelをもった無名のN-Channel Color Spaceである、`devicen_t<2>`、`devicen_t<3>`、`devicen_t<4>`、`devicen_t<5>`も提供しています。
Layoutについて言えば、GILはスタンダードなLayout以外に`bgr_layout_t`、`bgra_layout_t`、`abgr_layout_t`、`argb_layout_t`などのLayoutを提供しています。

<!--
As an example, here is how GIL defines the RGBA color space:
-->

ひとつの例として、GILがどのようにしてRGBA Color Spaceを定義しているか示します。

{% highlight C++ %}

struct red_t{};
struct green_t{};
struct blue_t{};
struct alpha_t{};
typedef mpl::vector4<red_t,green_t,blue_t,alpha_t> rgba_t;

{% endhighlight %}

<!--
The ordering of the channels in the color space definition specifies their semantic order.
For example, red_t is the first semantic channel of rgba_t.
While there is a unique semantic ordering of the channels in a color space, channels may vary in their physical ordering in memory.
The mapping of channels is specified by ChannelMappingConcept, which is an MPL random access sequence of integral types.
A color space and its associated mapping are often used together. Thus they are grouped in GIL's layout:
-->

Color Spaceの定義におけるChannelの順序は、Channelのセマンテックな順序を規定します。
例を挙げると、`red_t`は`rgba_t`のセマンティックな順序における最初のChannelです。
あるColor Spaceにおいて、セマンティックなChannel順序は一意に決まる一方、メモリ上でのフィジカルなChannel順序は異なっている可能性があります。
Channelのマッピングは、整数型のMPLランダムアクセスシークエンスである、`ChannelMappingConcept`によって規定されています。
Color Spaceとその中のChannelのマッピングはしばしば一緒に使用されます。
そのため、このふたつはGILのLayoutとしてまとめられています。

{% highlight C++ %}

template <typename ColorSpace,
          typename ChannelMapping = mpl::range_c<int,0,mpl::size<ColorSpace>::value> >
struct layout {
    typedef ColorSpace      color_space_t;
    typedef ChannelMapping  channel_mapping_t;
};

{% endhighlight %}

<!--
Here is how to create layouts for the RGBA color space:
-->

RGBA Color SpaceのLayoutの作り方は、次の通りです。

{% highlight C++ %}

typedef layout<rgba_t> rgba_layout_t; // default ordering is 0,1,2,3...
typedef layout<rgba_t, mpl::vector4_c<int,2,1,0,3> > bgra_layout_t;
typedef layout<rgba_t, mpl::vector4_c<int,1,2,3,0> > argb_layout_t;
typedef layout<rgba_t, mpl::vector4_c<int,3,2,1,0> > abgr_layout_t;

{% endhighlight %}


<!--
6. Color Base
-->

## <a name="section_06"> 6. Color Base

<!--
A color base is a container of color elements.
The most common use of color base is in the implementation of a pixel, in which case the color elements are channel values.
The color base concept, however, can be used in other scenarios. For example, a planar pixel has channels that are not contiguous in memory.
Its reference is a proxy class that uses a color base whose elements are channel references.
Its iterator uses a color base whose elements are channel iterators.
-->

Color Baseは色要素のコンテナです。
Color BaseはPixelの実装のなかで、すなわち色要素がChannelの値になっている場合に、よく用いられます。
しかし、Color BaseのConceptは他の用途に用いられる場合もあります。
例えば、プラナー画像のPixelはメモリ上で不連続なChannelをもっています。
その参照は、各Channelの参照を要素とする、Color Baseを用いたProxyクラスです。
そのIteratorは、各ChannelのIteratorを要素とするColor Baseを使用します。

<!--
Color base models must satisfy the following concepts:
-->

Color BaseのModelは、次に示すConceptを満たさなければなりません。

{% highlight C++ %}

concept ColorBaseConcept<typename T> : CopyConstructible<T>, EqualityComparable<T> {
    // a GIL layout (the color space and element permutation)
    typename layout_t;

    // The type of K-th element
    template <int K> struct kth_element_type;
        where Metafunction<kth_element_type>;

    // The result of at_c
    template <int K> struct kth_element_const_reference_type;
        where Metafunction<kth_element_const_reference_type>;

    template <int K> kth_element_const_reference_type<T,K>::type at_c(T);

    template <ColorBaseConcept T2> where { ColorBasesCompatibleConcept<T,T2> }
        T::T(T2);
    template <ColorBaseConcept T2> where { ColorBasesCompatibleConcept<T,T2> }
        bool operator==(const T&, const T2&);
    template <ColorBaseConcept T2> where { ColorBasesCompatibleConcept<T,T2> }
        bool operator!=(const T&, const T2&);

};

concept MutableColorBaseConcept<ColorBaseConcept T> : Assignable<T>, Swappable<T> {
    template <int K> struct kth_element_reference_type;
        where Metafunction<kth_element_reference_type>;

    template <int K> kth_element_reference_type<T,K>::type at_c(T);

    template <ColorBaseConcept T2> where { ColorBasesCompatibleConcept<T,T2> }
        T& operator=(T&, const T2&);
};

concept ColorBaseValueConcept<typename T> : MutableColorBaseConcept<T>, Regular<T> {
};

concept HomogeneousColorBaseConcept<ColorBaseConcept CB> {
    // For all K in [0 ... size<C1>::value-1):
    //     where SameType<kth_element_type<K>::type, kth_element_type<K+1>::type>;
    kth_element_const_reference_type<0>::type dynamic_at_c(const CB&, std::size_t n) const;
};

concept MutableHomogeneousColorBaseConcept<MutableColorBaseConcept CB> : HomogeneousColorBaseConcept<CB> {
    kth_element_reference_type<0>::type dynamic_at_c(const CB&, std::size_t n);
};

concept HomogeneousColorBaseValueConcept<typename T> : MutableHomogeneousColorBaseConcept<T>, Regular<T> {
};

concept ColorBasesCompatibleConcept<ColorBaseConcept C1, ColorBaseConcept C2> {
    where SameType<C1::layout_t::color_space_t, C2::layout_t::color_space_t>;
    // also, for all K in [0 ... size<C1>::value):
    //     where Convertible<kth_semantic_element_type<C1,K>::type, kth_semantic_element_type<C2,K>::type>;
    //     where Convertible<kth_semantic_element_type<C2,K>::type, kth_semantic_element_type<C1,K>::type>;
};

{% endhighlight %}

<!--
A color base must have an associated layout (which consists of a color space, as well as an ordering of the channels).
There are two ways to index the elements of a color base: A physical index corresponds to the way they are ordered in memory, and a semantic index corresponds to the way the elements are ordered in their color space.
For example, in the RGB color space the elements are ordered as {red_t, green_t, blue_t}.
For a color base with a BGR layout, the first element in physical ordering is the blue element, whereas the first semantic element is the red one.
Models of ColorBaseConcept are required to provide the at_c<K>(ColorBase) function, which allows for accessing the elements based on their physical order.
GIL provides a semantic_at_c<K>(ColorBase) function (described later) which can operate on any model of ColorBaseConcept and returns the corresponding semantic element.
-->

Color Baseは、Layoutを必ず1個もっていなければなりません (そのLayoutはColor SpaceとChannelの順序から構成されています)。
Color Baseの各要素へのインデクシングには2種類の方法があります。
メモリ上での各要素の配置に対応したフィジカルインデクスと、Color Spaceが示す順序に対応したセマンティックインデクスです。
例えば、RGB Color Spaceでは、各要素が{`red_t`, `green_t`, `blue_t`}の順に並んでいます。
Color BaseがBGR Layoutをもつなら、フィジカルな順序でみると最初の要素は青(blue)ですが、一方、セマンティックな順序でみると最初の要素は赤(red)となります。
`ColorBaseConcept`のModelは、フィジカルな順序に基づいて各要素にアクセスする関数`at_c<K>(ColorBase)`を提供することが求められます。
GILは、あらゆる`ColorBaseConcept`のModel上で動作し、セマンティックに要素を返す関数`semantic_at_c<K>(ColorBase)` (あとで述べます)を提供します。

<!--
Two color bases are compatible if they have the same color space and their elements (paired semantically) are convertible to each other.
-->

ふたつのColor Baseは、同じColor Spaceをもち、セマンティックに対をなす各要素が互いに変換可能であるとき、互換性をもちます。


<!--
Models:

GIL provides a model for a homogeneous color base (a color base whose elements all have the same type).
-->

#### Model:

GILは、ホモジーニアスなColor Base(各要素が全て同じ型のColor Base)のためのModelを提供します。

{% highlight C++ %}

namespace detail {
    template <typename Element, typename Layout, int K> struct homogeneous_color_base;
}

{% endhighlight %}

<!--
It is used in the implementation of GIL's pixel, planar pixel reference and planar pixel iterator.
Another model of ColorBaseConcept is packed_pixel - it is a pixel whose channels are bit ranges.
See the 7. Pixel section for more.
-->

このModelは、GILのPixel、Planar Pixelの参照、Planar PixelのIteratorの実装に使われています。
もうひとつの`ColorBaseConcept`のModelは`packed_pixel`であり、ビット単位のレンジをもつChannelに基づいたPixelです。
詳しくは、第7章を参照ください。

<!--
Algorithms:

GIL provides the following functions and metafunctions operating on color bases:
-->

#### Algorithm:

GILは、次に示す、Color Base上で動作する関数とメタ関数を提供します。

{% highlight C++ %}

// Metafunction returning an mpl::int_ equal to the number of elements in the color base
template <class ColorBase> struct size;

// Returns the type of the return value of semantic_at_c<K>(color_base)
template <class ColorBase, int K> struct kth_semantic_element_reference_type;
template <class ColorBase, int K> struct kth_semantic_element_const_reference_type;

// Returns a reference to the element with K-th semantic index.
template <class ColorBase, int K>
typename kth_semantic_element_reference_type<ColorBase,K>::type       semantic_at_c(ColorBase& p)
template <class ColorBase, int K>
typename kth_semantic_element_const_reference_type<ColorBase,K>::type semantic_at_c(const ColorBase& p)

// Returns the type of the return value of get_color<Color>(color_base)
template <typename Color, typename ColorBase> struct color_reference_t;
template <typename Color, typename ColorBase> struct color_const_reference_t;

// Returns a reference to the element corresponding to the given color
template <typename ColorBase, typename Color>
typename color_reference_t<Color,ColorBase>::type get_color(ColorBase& cb, Color=Color());
template <typename ColorBase, typename Color>
typename color_const_reference_t<Color,ColorBase>::type get_color(const ColorBase& cb, Color=Color());

// Returns the element type of the color base. Defined for homogeneous color bases only
template <typename ColorBase> struct element_type;
template <typename ColorBase> struct element_reference_type;
template <typename ColorBase> struct element_const_reference_type;

{% endhighlight %}

<!--
GIL also provides the following algorithms which operate on color bases.
Note that they all pair the elements semantically:
-->

GILは、Color Baseで動作する、次のようなアルゴリズムも提供しています。
これらのアルゴリズムが各要素をセマンティックなペアで扱うことに注意してください。

{% highlight C++ %}

// Equivalents to std::equal, std::copy, std::fill, std::generate
template <typename CB1,typename CB2>   bool static_equal(const CB1& p1, const CB2& p2);
template <typename Src,typename Dst>   void static_copy(const Src& src, Dst& dst);
template <typename CB, typename Op>    void static_generate(CB& dst,Op op);

// Equivalents to std::transform
template <typename CB ,             typename Dst,typename Op> Op static_transform(      CB&,Dst&,Op);
template <typename CB ,             typename Dst,typename Op> Op static_transform(const CB&,Dst&,Op);
template <typename CB1,typename CB2,typename Dst,typename Op> Op static_transform(      CB1&,      CB2&,Dst&,Op);
template <typename CB1,typename CB2,typename Dst,typename Op> Op static_transform(const CB1&,      CB2&,Dst&,Op);
template <typename CB1,typename CB2,typename Dst,typename Op> Op static_transform(      CB1&,const CB2&,Dst&,Op);
template <typename CB1,typename CB2,typename Dst,typename Op> Op static_transform(const CB1&,const CB2&,Dst&,Op);

// Equivalents to std::for_each
template <typename CB1,                          typename Op> Op static_for_each(      CB1&,Op);
template <typename CB1,                          typename Op> Op static_for_each(const CB1&,Op);
template <typename CB1,typename CB2,             typename Op> Op static_for_each(      CB1&,      CB2&,Op);
template <typename CB1,typename CB2,             typename Op> Op static_for_each(      CB1&,const CB2&,Op);
template <typename CB1,typename CB2,             typename Op> Op static_for_each(const CB1&,      CB2&,Op);
template <typename CB1,typename CB2,             typename Op> Op static_for_each(const CB1&,const CB2&,Op);
template <typename CB1,typename CB2,typename CB3,typename Op> Op static_for_each(      CB1&,      CB2&,      CB3&,Op);
template <typename CB1,typename CB2,typename CB3,typename Op> Op static_for_each(      CB1&,      CB2&,const CB3&,Op);
template <typename CB1,typename CB2,typename CB3,typename Op> Op static_for_each(      CB1&,const CB2&,      CB3&,Op);
template <typename CB1,typename CB2,typename CB3,typename Op> Op static_for_each(      CB1&,const CB2&,const CB3&,Op);
template <typename CB1,typename CB2,typename CB3,typename Op> Op static_for_each(const CB1&,      CB2&,      CB3&,Op);
template <typename CB1,typename CB2,typename CB3,typename Op> Op static_for_each(const CB1&,      CB2&,const CB3&,Op);
template <typename CB1,typename CB2,typename CB3,typename Op> Op static_for_each(const CB1&,const CB2&,      CB3&,Op);
template <typename CB1,typename CB2,typename CB3,typename Op> Op static_for_each(const CB1&,const CB2&,const CB3&,Op);

// The following algorithms are only defined for homogeneous color bases:
// Equivalent to std::fill
template <typename HCB, typename Element> void static_fill(HCB& p, const Element& v);

// Equivalents to std::min_element and std::max_element
template <typename HCB> typename element_const_reference_type<HCB>::type static_min(const HCB&);
template <typename HCB> typename element_reference_type<HCB>::type       static_min(      HCB&);
template <typename HCB> typename element_const_reference_type<HCB>::type static_max(const HCB&);
template <typename HCB> typename element_reference_type<HCB>::type       static_max(      HCB&);

{% endhighlight %}

<!--
These algorithms are designed after the corresponding STL algorithms, except that instead of ranges they take color bases and operate on their elements.
In addition, they are implemented with a compile-time recursion (thus the prefix "static_").
Finally, they pair the elements semantically instead of based on their physical order in memory.
For example, here is the implementation of static_equal:
-->

これらのアルゴリズムは、レンジのかわりにColor Baseを使って各要素のオペレーションを行うという点を除いて、STLアルゴリズムに対応するようにデザインされています。
さらに、コンパイル時の再帰を用いる実装になっています (そのため、prefixに`static_`がついてます)。
そして、これらのアルゴリズムは、メモリ上のフィジカルな順序ではなく、セマンテックな順序に基づいて要素のペアを作ります。
例えば、`static_equal`の実装を挙げると次のようになります。

{% highlight C++ %}

namespace detail {
template <int K> struct element_recursion {
    template <typename P1,typename P2>
    static bool static_equal(const P1& p1, const P2& p2) {
        return element_recursion<K-1>::static_equal(p1,p2) &&
               semantic_at_c<K-1>(p1)==semantic_at_c<N-1>(p2);
    }
};
template <> struct element_recursion<0> {
    template <typename P1,typename P2>
    static bool static_equal(const P1&, const P2&) { return true; }
};
}

template <typename P1,typename P2>
bool static_equal(const P1& p1, const P2& p2) {
    gil_function_requires<ColorSpacesCompatibleConcept<P1::layout_t::color_space_t,P2::layout_t::color_space_t> >();
    return detail::element_recursion<size<P1>::value>::static_equal(p1,p2);
}

{% endhighlight %}

<!--
This algorithm is used when invoking operator== on two pixels, for example.
By using semantic accessors we are properly comparing an RGB pixel to a BGR pixel.
Notice also that all of the above algorithms taking more than one color base require that they all have the same color space.
-->

このアルゴリズムは、例えば、ふたつのPixel間の`operator==`を実行するときに用います。
セマンティックなアクセサを使うことで、RGB PixelとBGR Pixelを適切に比較できます。
また、上記の2個以上のColor Baseを引数にとる全てのアルゴリズムは、全てのColorBaseが同じColor Spaceをもつよう要求することに注意しましょう。


<!--
7. Pixel
-->

## <a name="section_07"> 7. Pixel

<!--
A pixel is a set of channels defining the color at a given point in an image.
Conceptually, a pixel is little more than a color base whose elements model ChannelConcept.
All properties of pixels inherit from color bases: pixels may be homogeneous if all of their channels have the same type; otherwise they are called heterogeneous.
The channels of a pixel may be addressed using semantic or physical indexing, or by color; all color-base algorithms work on pixels as well.
Two pixels are compatible if their color spaces are the same and their channels, paired semantically, are compatible.
Note that constness, memory organization and reference/value are ignored.
For example, an 8-bit RGB planar reference is compatible to a constant 8-bit BGR interleaved pixel value.
Most pairwise pixel operations (copy construction, assignment, equality, etc.) are only defined for compatible pixels.
-->

Pixelは、ある画像中のある1点に対応する、色が定義されたChannelのセットです。
概念的に言えば、Pixelと`ChannelConcept`に基づいた要素をもったColor Baseはほとんど同じようなものです。
Pixelの全てのプロパティはColor Baseから受け継いだものです。
すなわち、Color Baseの全てのChannelが同じ型であったなら、それを受け継いだPixelはホモジーニアスです。
そうでないのなら、それを受け継いだPixelはヘテロジーニアスと呼ばれます。
PixelのChannelは、セマンティックなインデクス、フィジカルなインデクス、もしくは色によって呼び出されます。
全てのColor BaseアルゴリズムがPixel上でも同じように動作します。
同じColor Spaceをもち、その各Channelがセマンティックなペアに対して互換性をもつとき、そのふたつのPixel間には互換性があります。
Const性、メモリ上での配置、参照であるか値であるかは無視されることに注意してください。
例を挙げると、8bitのRGBプラナー形式Pixelの参照は、8bitのBGRインタリーブ形式Pixelと互換性があります。
ほとんどのPixelの2項演算(コピーコンストラクション, 代入、等価など)は互換性をもつPixel間でだけ定義されています。

<!--
Pixels (as well as other GIL constructs built on pixels, such as iterators, locators, views and images) must provide metafunctions to access their color space, channel mapping, number of channels, and (for homogeneous pixels) the channel type:
-->

Pixelは、(Pixelに基づいて構築されるIterator、Locator、View、Imageなどといった他のGILコンストラクトと同様、)そのColor Space、Channelマッピング、Channel数、(そのPixelがホモジニアスであるなら)Channel型にアクセスするためのメタ関数を提供しなければなりません。

{% highlight C++ %}

concept PixelBasedConcept<typename T> {
    typename color_space_type<T>;
        where Metafunction<color_space_type<T> >;
        where ColorSpaceConcept<color_space_type<T>::type>;
    typename channel_mapping_type<T>;
        where Metafunction<channel_mapping_type<T> >;  
        where ChannelMappingConcept<channel_mapping_type<T>::type>;
    typename is_planar<T>;
        where Metafunction<is_planar<T> >;
        where SameType<is_planar<T>::type, bool>;
};

concept HomogeneousPixelBasedConcept<PixelBasedConcept T> {
    typename channel_type<T>;
        where Metafunction<channel_type<T> >;
        where ChannelConcept<channel_type<T>::type>;
};

{% endhighlight %}

<!--
Pixels model the following concepts:
-->

Pixelは次のConceptに基づいたModelです。

{% highlight C++ %}

concept PixelConcept<typename P> : ColorBaseConcept<P>, PixelBasedConcept<P> {
    where is_pixel<P>::type::value==true;
    // where for each K [0..size<P>::value-1]:
    //      ChannelConcept<kth_element_type<K> >;

    typename value_type;       where PixelValueConcept<value_type>;
    typename reference;        where PixelConcept<reference>;
    typename const_reference;  where PixelConcept<const_reference>;
    static const bool P::is_mutable;

    template <PixelConcept P2> where { PixelConcept<P,P2> }
        P::P(P2);
    template <PixelConcept P2> where { PixelConcept<P,P2> }
        bool operator==(const P&, const P2&);
    template <PixelConcept P2> where { PixelConcept<P,P2> }
        bool operator!=(const P&, const P2&);
};

concept MutablePixelConcept<typename P> : PixelConcept<P>, MutableColorBaseConcept<P> {
    where is_mutable==true;
};

concept HomogeneousPixelConcept<PixelConcept P> : HomogeneousColorBaseConcept<P>, HomogeneousPixelBasedConcept<P> {
    P::template element_const_reference_type<P>::type operator[](P p, std::size_t i) const { return dynamic_at_c(P,i); }
};

concept MutableHomogeneousPixelConcept<MutablePixelConcept P> : MutableHomogeneousColorBaseConcept<P> {
    P::template element_reference_type<P>::type operator[](P p, std::size_t i) { return dynamic_at_c(p,i); }
};

concept PixelValueConcept<typename P> : PixelConcept<P>, Regular<P> {
    where SameType<value_type,P>;
};

concept PixelsCompatibleConcept<PixelConcept P1, PixelConcept P2> : ColorBasesCompatibleConcept<P1,P2> {
    // where for each K [0..size<P1>::value):
    //    ChannelsCompatibleConcept<kth_semantic_element_type<P1,K>::type, kth_semantic_element_type<P2,K>::type>;
};

{% endhighlight %}

<!--
A pixel is convertible to a second pixel if it is possible to approximate its color in the form of the second pixel.
Conversion is an explicit, non-symmetric and often lossy operation (due to both channel and color space approximation).
Convertability requires modeling the following concept:
-->

あるPixelが自身の色をもう一方のPixelの形式に近似できるとき、もう一方のPixelと変換可能です。
変換は、数学的に陽であり、非対称であり、ほとんどの場合は(ChannelとColor Space両方の近似が原因で)不可逆変換です。
交換可能性は次のConceptに基づいた実装を要求します。

{% highlight C++ %}

template <PixelConcept SrcPixel, MutablePixelConcept DstPixel>
concept PixelConvertibleConcept {
    void color_convert(const SrcPixel&, DstPixel&);
};

{% endhighlight %}

<!--
The distinction between PixelConcept and PixelValueConcept is analogous to that for channels and color bases - pixel reference proxies model both, but only pixel values model the latter.
-->

`PixelConcept`と`PixelValueConcept`の違いは、ChannelとColor Baseの違いと似ています。
Pixel参照Proxyは両方のConceptに基づいたModelですが、Pixelは後者のConceptだけに基づいたModelです。

<!--
Related Concepts:

PixelBasedConcept<P>
PixelConcept<Pixel>
MutablePixelConcept<Pixel>
PixelValueConcept<Pixel>
HomogeneousPixelConcept<Pixel>
MutableHomogeneousPixelConcept<Pixel>
HomogeneousPixelValueConcept<Pixel>
PixelsCompatibleConcept<Pixel1,Pixel2>
PixelConvertibleConcept<SrcPixel,DstPixel>
-->

#### 関連するConcept:

- `PixelBasedConcept<P>`
- `PixelConcept<Pixel>`
- `MutablePixelConcept<Pixel>`
- `PixelValueConcept<Pixel>`
- `HomogeneousPixelConcept<Pixel>`
- `MutableHomogeneousPixelConcept<Pixel>`
- `HomogeneousPixelValueConcept<Pixel>`
- `PixelsCompatibleConcept<Pixel1,Pixel2>`
- `PixelConvertibleConcept<SrcPixel,DstPixel>`

<!--
Models:

The most commonly used pixel is a homogeneous pixel whose values are together in memory.
For this purpose GIL provides the struct pixel, templated over the channel value and layout:
-->

#### Model:

最もよく用いられるPixelは、メモリ上でひとまとまりになったホモジーニアスPixelです。
このために、GILはChannelに関連づけられた型とLayoutでテンプレート化した`struct pixel`を提供しています。

{% highlight C++ %}

// models HomogeneousPixelValueConcept
template <typename ChannelValue, typename Layout> struct pixel;

// Those typedefs are already provided by GIL
typedef pixel<bits8, rgb_layout_t> rgb8_pixel_t;
typedef pixel<bits8, bgr_layout_t> bgr8_pixel_t;

bgr8_pixel_t bgr8(255,0,0);     // pixels can be initialized with the channels directly
rgb8_pixel_t rgb8(bgr8);        // compatible pixels can also be copy-constructed

rgb8 = bgr8;            // assignment and equality is defined between compatible pixels
assert(rgb8 == bgr8);   // assignment and equality operate on the semantic channels

// The first physical channels of the two pixels are different
assert(at_c<0>(rgb8) != at_c<0>(bgr8));
assert(dynamic_at_c(bgr8,0) != dynamic_at_c(rgb8,0));
assert(rgb8[0] != bgr8[0]); // same as above (but operator[] is defined for pixels only)

{% endhighlight %}

<!--
Planar pixels have their channels distributed in memory.
While they share the same value type (pixel) with interleaved pixels, their reference type is a proxy class containing references to each of the channels.
This is implemented with the struct planar_pixel_reference:
-->

プラナーPixelは、メモリ上の離れた地点に配置されたChannelをもちます。
Channelに関連づけられた型についてインタリーブPixelと同じ型を共有している場合、その参照型は各Channelの参照をもつProxyクラスになっています。
これは`struct planar_pixel_reference`で実装されています。

{% highlight C++ %}

// models HomogeneousPixel
template <typename ChannelReference, typename ColorSpace> struct planar_pixel_reference;

// Define the type of a mutable and read-only reference. (These typedefs are already provided by GIL)
typedef planar_pixel_reference<      bits8&,rgb_t> rgb8_planar_ref_t;
typedef planar_pixel_reference<const bits8&,rgb_t> rgb8c_planar_ref_t;

{% endhighlight %}

<!--
Note that, unlike the pixel struct, planar pixel references are templated over the color space, not over the pixel layout.
They always use a cannonical channel ordering.
Ordering of their elements is unnecessary because their elements are references to the channels.
-->

`struct planar_pixel_reference`は、Layoutでテンプレート化されている`struct pixel`とは異なり、Color Spaceでテンプレート化されていることに注意してください。
これらは常に標準化されたChannel順を用います。
各要素は各Channelから参照されるため、要素の順序に関する情報は不要なのです。

<!--
Sometimes the channels of a pixel may not be byte-aligned.
For example an RGB pixel in '5-5-6' format is a 16-bit pixel whose red, green and blue channels occupy bits [0..4],[5..9] and [10..15] respectively.
GIL provides a model for such packed pixel formats:
-->

Pixelの各Channelの境界がバイト境界と一致していない可能性もあります。
例えば、'556' RGB Pixelは赤(Red)、緑(Green)、青(Blue)の各Channelが[0..4]、[5..9]、[10..15]bitを占める16bitのPixelです。
GILは上記のようなPaced Pixel形式のためのModelを提供しています。

{% highlight C++ %}

// define an rgb565 pixel
typedef packed_pixel_type<uint16_t, mpl::vector3_c<unsigned,5,6,5>, rgb_layout_t>::type rgb565_pixel_t;

function_requires<PixelValueConcept<rgb565_pixel_t> >();
BOOST_STATIC_ASSERT((sizeof(rgb565_pixel_t)==2));

// define a bgr556 pixel
typedef packed_pixel_type<uint16_t, mpl::vector3_c<unsigned,5,6,5>, bgr_layout_t>::type bgr556_pixel_t;

function_requires<PixelValueConcept<bgr556_pixel_t> >();

// rgb565 is compatible with bgr556.
function_requires<PixelsCompatibleConcept<rgb565_pixel_t,bgr556_pixel_t> >();

{% endhighlight %}

<!--
In some cases, the pixel itself may not be byte aligned.
For example, consider an RGB pixel in '2-3-2' format.
Its size is 7 bits.
GIL refers to such pixels, pixel iterators and images as "bit-aligned".
Bit-aligned pixels (and images) are more complex than packed ones.
Since packed pixels are byte-aligned, we can use a C++ reference as the reference type to a packed pixel, and a C pointer as an x_iterator over a row of packed pixels.
For bit-aligned constructs we need a special reference proxy class (bit_aligned_pixel_reference) and iterator class (bit_aligned_pixel_iterator).
The value type of bit-aligned pixels is a packed_pixel.
Here is how to use bit_aligned pixels and pixel iterators:
-->

ある場合には、Pixel全体の長さがバイト単位にならないかもしれません。
例として、'232' RGB Pixelを考えてみましょう。
このサイズは7bitです。
GILはこのようなPixel、Pixel Iterator、Imageを"Bit-aligned"と呼びます。
Bit-aligned Pixel (とImage)は、Channel長の合計がバイト単位になるものより複雑です。
Packed Pixelはバイト単位なので、そのPixel参照にはC++の参照を使用できますし、Packed Pixelの行に対する`x_iterator`にはCのポインタを使用できます。
Bit-alignedの場合には、特別な参照Proxyクラス(`bit_aligned_pixel_reference`)とIteratorクラス(`bit_aligned_pixel_iterator`)を必要とします。
Bit-aligned Pixelの値の型は`packed_pixel`です。
ここで、Bit-aligned PixelとIteratorの使い方を示します。

{% highlight C++ %}

// Mutable reference to a BGR232 pixel
typedef const bit_aligned_pixel_reference<mpl::vector3_c<unsigned,2,3,2>, bgr_layout_t, true>  bgr232_ref_t;

// A mutable iterator over BGR232 pixels
typedef bit_aligned_pixel_iterator<bgr232_ref_t> bgr232_ptr_t;

// BGR232 pixel value. It is a packed_pixel of size 1 byte. (The last bit is unused)
typedef std::iterator_traits<bgr232_ptr_t>::value_type bgr232_pixel_t;
BOOST_STATIC_ASSERT((sizeof(bgr232_pixel_t)==1));

bgr232_pixel_t red(0,0,3); // = 0RRGGGBB, = 01100000 = 0x60

// a buffer of 7 bytes fits exactly 8 BGR232 pixels.
unsigned char pix_buffer[7];
std::fill(pix_buffer,pix_buffer+7,0);

// Fill the 8 pixels with red
bgr232_ptr_t pix_it(&pix_buffer[0],0);  // start at bit 0 of the first pixel
for (int i=0; i<8; ++i) {
    *pix_it++ = red;
}
// Result: 0x60 0x30 0x11 0x0C 0x06 0x83 0xC1

{% endhighlight %}

<!--
Algorithms:

Since pixels model ColorBaseConcept and PixelBasedConcept all algorithms and metafunctions of color bases can work with them as well:
-->

#### Algorithm:

Pixelは`ColorBaseConcept`と`PixelBaseConcept`に基づいたModelなので、全てのColor Baseアルゴリズムとメタ関数はPixel上でも問題なく動作します。

{% highlight C++ %}

// This is how to access the first semantic channel (red)
assert(semantic_at_c<0>(rgb8) == semantic_at_c<0>(bgr8));

// This is how to access the red channel by name
assert(get_color<red_t>(rgb8) == get_color<red_t>(bgr8));

// This is another way of doing it (some compilers don't like the first one)
assert(get_color(rgb8,red_t()) == get_color(bgr8,red_t()));

// This is how to use the PixelBasedConcept metafunctions
BOOST_MPL_ASSERT(num_channels<rgb8_pixel_t>::value == 3);
BOOST_MPL_ASSERT((is_same<channel_type<rgb8_pixel_t>::type, bits8>));
BOOST_MPL_ASSERT((is_same<color_space_type<bgr8_pixel_t>::type, rgb_t> ));
BOOST_MPL_ASSERT((is_same<channel_mapping_type<bgr8_pixel_t>::type, mpl::vector3_c<int,2,1,0> > ));

// Pixels contain just the three channels and nothing extra
BOOST_MPL_ASSERT(sizeof(rgb8_pixel_t)==3);

rgb8_planar_ref_t ref(bgr8);    // copy construction is allowed from a compatible mutable pixel type

get_color<red_t>(ref) = 10;     // assignment is ok because the reference is mutable
assert(get_color<red_t>(bgr8)==10);  // references modify the value they are bound to

// Create a zero packed pixel and a full regular unpacked pixel.
rgb565_pixel_t r565;
rgb8_pixel_t rgb_full(255,255,255);

// Convert all channels of the unpacked pixel to the packed one & assert the packed one is full
get_color(r565,red_t())   = channel_convert<rgb565_channel0_t>(get_color(rgb_full,red_t()));
get_color(r565,green_t()) = channel_convert<rgb565_channel1_t>(get_color(rgb_full,green_t()));
get_color(r565,blue_t())  = channel_convert<rgb565_channel2_t>(get_color(rgb_full,blue_t()));
assert(r565 == rgb565_pixel_t((uint16_t)65535));

{% endhighlight %}

<!--
GIL also provides the color_convert algorithm to convert between pixels of different color spaces and channel types:
-->

また、GILはColor SpaceとChannel型が異なるPixel間の変換を行う`color_convert`アルゴリズムを提供します。

{% highlight C++ %}

rgb8_pixel_t red_in_rgb8(255,0,0);
cmyk16_pixel_t red_in_cmyk16;
color_convert(red_in_rgb8,red_in_cmyk16);

{% endhighlight %}


<!--
8. Pixel Iterator
-->

## <a name="section_08"> 8. Pixel Iterator

<!--
Fundamental Iterator

Pixel iterators are random traversal iterators whose value_type models PixelValueConcept.
Pixel iterators provide metafunctions to determine whether they are mutable (i.e. whether they allow for modifying the pixel they refer to), to get the immutable (read-only) type of the iterator, and to determine whether they are plain iterators or adaptors over another pixel iterator:
-->

### <a name="section_08_1"> 基本となるIterator

Pixel Iteratorは、`PixelValueConcept`に基づいたModelである`value_type`のランダム走査Iteratorです。
Pixel Iteratorは、mutableであるか否か(すなわち、指し示すPixelが変更可能か否か)を判定するメタ関数、immutable (read-only)なIteratorを取得するメタ関数、素のIteratorかアダプタをまとった他のIteratorなのかを判定するメタ関数を提供します。

{% highlight C++ %}

concept PixelIteratorConcept<RandomAccessTraversalIteratorConcept Iterator> : PixelBasedConcept<Iterator> {
    where PixelValueConcept<value_type>;
    typename const_iterator_type<It>::type;
        where PixelIteratorConcept<const_iterator_type<It>::type>;
    static const bool  iterator_is_mutable<It>::type::value;
    static const bool  is_iterator_adaptor<It>::type::value;   // is it an iterator adaptor
};

template <typename Iterator>
concept MutablePixelIteratorConcept : PixelIteratorConcept<Iterator>, MutableRandomAccessIteratorConcept<Iterator> {};

{% endhighlight %}

<!--
Related Concepts:

PixelIteratorConcept<Iterator>
MutablePixelIteratorConcept<Iterator>
-->

#### 関連するConcept:

- `PixelIteratorConcept<Iterator>`
- `MutablePixelIteratorConcept<Iterator>`

<!--
Models:

A built-in pointer to pixel, pixel<ChannelValue,Layout>*, is GIL's model for pixel iterator over interleaved homogeneous pixels.
Similarly, packed_pixel<PixelData,ChannelRefVec,Layout>* is GIL's model for an iterator over interleaved packed pixels.
-->

#### Model:

Pixelのビルトインポインタ`pixel<ChannelValue,Layout>*`は、インタリーブ形式ホモジーニアスPixelを対象とするPixel IteratorのためのGILのModelです。
同様に、`packed_pixel<PixelData,ChannelRefVec,Layout>*`は、インタリーブ形式バイト単位Pixelを対象とするIteratorのためのGILのModelです。

<!--
For planar homogeneous pixels, GIL provides the class planar_pixel_iterator, templated over a channel iterator and color space.
Here is how the standard mutable and read-only planar RGB iterators over unsigned char are defined:
-->

プラナー形式ホモジーニアスPixelのために、GILはChannel IteratorとColor Spaceをパラメータにとるテンプレートである`planar_pixel_iterator`クラスを提供します。
ここで、unsigned char型プラナー形式RGB Pixelについて、mutableなIteratorとread-onlyのIteratorがどのように定義されているのかを示します。

{% highlight C++ %}

template <typename ChannelPtr, typename ColorSpace> struct planar_pixel_iterator;

// GIL provided typedefs
typedef planar_pixel_iterator<const bits8*, rgb_t> rgb8c_planar_ptr_t;
typedef planar_pixel_iterator<      bits8*, rgb_t> rgb8_planar_ptr_t;

{% endhighlight %}

<!--
planar_pixel_iterator also models HomogeneousColorBaseConcept (it subclasses from homogeneous_color_base) and, as a result, all color base algorithms apply to it.
The element type of its color base is a channel iterator.
For example, GIL implements operator++ of planar iterators approximately like this:
-->

`planar_pixel_iterator`は`HomogeneousColorBaseConcept` (`homogeneous_color_base`のサブクラス)に基づいたModelであり、つまり、全てのColor Baseアルゴリズムを適用できます。
そのColor Baseの要素の型はChannel Iteratorです。
例を挙げると、GILではプラナー形式Iteratorの`operator++`をおおよそ次のように実装しています。

{% highlight C++ %}

template <typename T>
struct inc : public std::unary_function<T,T> {
    T operator()(T x) const { return ++x; }
};

template <typename ChannelPtr, typename ColorSpace>
planar_pixel_iterator<ChannelPtr,ColorSpace>&
planar_pixel_iterator<ChannelPtr,ColorSpace>::operator++() {
    static_transform(*this,*this,inc<ChannelPtr>());
    return *this;
}

{% endhighlight %}

<!--
Since static_transform uses compile-time recursion, incrementing an instance of rgb8_planar_ptr_t amounts to three pointer increments.
GIL also uses the class bit_aligned_pixel_iterator as a model for a pixel iterator over bit-aligned pixels.
Internally it keeps track of the current byte and the bit offset.
-->

`static_transform`はコンパイル時の再帰を用いるので、`rgb8_planar_ptr_t`インスタンスのインクリメントは3個のpointerのインクリメントに変換されます。
また、GILは、ビット単位Pixelを走査するPixel IteratorのModelとして、`bit_aligned_pixel_iterator`クラスを用います。
内部的には、各時点でのバイト位置とビットオフセットを記録しています。

<!--
Iterator Adaptor

Iterator adaptor is an iterator that wraps around another iterator.
Its is_iterator_adaptor metafunction must evaluate to true, and it needs to provide a member method to return the base iterator, a metafunction to get its type, and a metafunction to rebind to another base iterator:
-->

### <a name="section_08_2"> Iteratorアダプタ

Iteratorアダプタは他のIteratorをラップしたIteratorです。
その`is_iterator_adaptor`というメタ関数は`true`でなければなりません。
また、Base Iteratorを返すメンバ関数、型を取得するメタ関数、他のBase Iteratorに再結合するメタ関数を提供する必要があります。

{% highlight C++ %}

concept IteratorAdaptorConcept<RandomAccessTraversalIteratorConcept Iterator> {
    where SameType<is_iterator_adaptor<Iterator>::type, mpl::true_>;

    typename iterator_adaptor_get_base<Iterator>;
        where Metafunction<iterator_adaptor_get_base<Iterator> >;
        where boost_concepts::ForwardTraversalConcept<iterator_adaptor_get_base<Iterator>::type>;

    typename another_iterator;
    typename iterator_adaptor_rebind<Iterator,another_iterator>::type;
        where boost_concepts::ForwardTraversalConcept<another_iterator>;
        where IteratorAdaptorConcept<iterator_adaptor_rebind<Iterator,another_iterator>::type>;

    const iterator_adaptor_get_base<Iterator>::type& Iterator::base() const;
};

template <boost_concepts::Mutable_ForwardIteratorConcept Iterator>
concept MutableIteratorAdaptorConcept : IteratorAdaptorConcept<Iterator> {};

{% endhighlight %}

<!--
Related Concepts:

IteratorAdaptorConcept<Iterator>
MutableIteratorAdaptorConcept<Iterator>
-->

#### 関連するConcept:

- `IteratorAdaptorConcept<Iterator>`
- `MutableIteratorAdaptorConcept<Iterator>`

<!--
Models:

GIL provides several models of IteratorAdaptorConcept:
-->

#### Model:

GILは`IteratorAdaptorConcept`のModelをいくつか提供しています。

<!--
memory_based_step_iterator<Iterator>: An iterator adaptor that changes the fundamental step of the base iterator (see Step Iterator)
dereference_iterator_adaptor<Iterator,Fn>: An iterator that applies a unary function Fn upon dereferencing.
It is used, for example, for on-the-fly color conversion.
It can be used to construct a shallow image "view" that pretends to have a different color space or channel depth.
See Creating Image Views from Other Image Views for more.
The unary function Fn must model PixelDereferenceAdaptorConcept (see below).
-->

- `memory_based_step_iterator<Iterator>`: Base Iteratorの基本的なステップを変更するIteratorアダプタ。(ステップIteratorを参照)
- `dereference_iterator_adaptor<Iterator,Fn>`: ひとつの引数をとる関数`Fn`に間接参照した値を適用するIteratorアダプタ。
例を挙げると、on-the-flyな色変換に用いられます。
これは、実際とは異なるColor SpaceやChannel深度をもっているかのように見せかける浅いImage "View"を構築するときに用いられます。
詳細は、"ほかのImage ViewからImage Viewを作成する"をみてください。
ひとつの引数をとる関数`Fn`は`PixelDereferenceAdaptorConcept`(次を見てください)に基づいたModelでなければなりません。

<!--
Pixel Dereference Adaptor

Pixel dereference adaptor is a unary function that can be applied upon dereferencing a pixel iterator.
Its argument type could be anything (usually a PixelConcept) and the result type must be convertible to PixelConcept
-->

### <a name="section_08_3"> Pixel間接参照アダプタ

Pixel間接参照アダプタは、Pixel Iteratorから間接参照した値を受け取る、ひとつの引数をもつ関数です。
この引数の型はどんなものでも構いません(よくあるのは`PixelConcept`です)。また、戻り値の型は`PixelConcept`に変換可能でなければなりません。

{% highlight C++ %}

template <boost::UnaryFunctionConcept D>
concept PixelDereferenceAdaptorConcept : DefaultConstructibleConcept<D>, CopyConstructibleConcept<D>, AssignableConcept<D>  {
    typename const_t;         where PixelDereferenceAdaptorConcept<const_t>;
    typename value_type;      where PixelValueConcept<value_type>;
    typename reference;       where PixelConcept<remove_reference<reference>::type>;  // may be mutable
    typename const_reference;   // must not be mutable
    static const bool D::is_mutable;

    where Convertible<value_type, result_type>;
};

{% endhighlight %}

<!--
Models:

GIL provides several models of PixelDereferenceAdaptorConcept
-->

#### Model:

GILは`PixelDereferenceAdaptorConcept`のModelをいくつか提供します。

<!--
color_convert_deref_fn: a function object that performs color conversion
detail::nth_channel_deref_fn: a function object that returns a grayscale pixel corresponding to the n-th channel of a given pixel
deref_compose: a function object that composes two models of PixelDereferenceAdaptorConcept. Similar to std::unary_compose, except it needs to pull the additional typedefs required by PixelDereferenceAdaptorConcept
-->

- `color_convert_deref_fn`: 色変換を行う関数オブジェクト。
- `detail::nth_channel_deref_fn`: 与えられたPixelのN番目Channelに対応するグレイスケールPixelを返す関数オブジェクト。
- `deref_compose`: 2つの`PixelDereferenceAdaptorConcept`のModelを合成する関数オブジェクト。`PixelDereferenceAdaptorConcept`で要求される追加の`typedef`を取る必要がある点を除いて、`std::unary_compose`と似ています。

<!--
GIL uses pixel dereference adaptors to implement image views that perform color conversion upon dereferencing, or that return the N-th channel of the underlying pixel.
They can be used to model virtual image views that perform an arbitrary function upon dereferencing, for example a view of the Mandelbrot set.
dereference_iterator_adaptor<Iterator,Fn> is an iterator wrapper over a pixel iterator Iterator that invokes the given dereference iterator adaptor Fn upon dereferencing.
-->

GILは、間接参照した値に色変換を実行するImage ViewやPixelのN番目のChannelを返すImage Viewを実装するために、Pixel間接参照アダプタを使用します。
これらのPixel間接参照アダプタは、間接参照した値に任意関数を実行するVirtual Image View (例えば、マンデルブロ集合を表すViewなど)を実装する際に使用されます。
`dereference_iterator_adaptor<Iterator,Fn>`は、`Iterator`で間接参照した値を引数にして与えられた間接参照Iteratorアダプタ`Fn`を実行する、Pixel Iteratorである`Iterator`を包むIteratorラッパです。

<!--
Step Iterator

Sometimes we want to traverse pixels with a unit step other than the one provided by the fundamental pixel iterators.
Examples where this would be useful:
-->

### <a name="section_08_4"> ステップIterator

基本的なPixel Iteratorによって提供される1ステップ以上のまとまったステップ数でPixelの走査を行いたい場合があります。
例えば、次のような場合です。

<!--
a single-channel view of the red channel of an RGB interleaved image
left-to-right flipped image (step = -fundamental_step)
subsampled view, taking every N-th pixel (step = N*fundamental_step)
traversal in vertical direction (step = number of bytes per row)
any combination of the above (steps are multiplied)
-->

- RGBインタリーブ画像の赤Channelだけをみる単独Channel View
- 左右反転画像 (step = -fundamental_step)
- N個間隔でPixelを取ってきたView (step = N*fundamental_step)
- 垂直方向への移動 (step = number bytes per row)
- 上記の組み合わせ (stepは各stepの積)

<!--
Step iterators are forward traversal iterators that allow changing the step between adjacent values:
-->

ステップIteratorは、隣り合う要素への移動についてのステップ数の変更を許可する、前方移動Iteratorです。

{% highlight C++ %}

concept StepIteratorConcept<boost_concepts::ForwardTraversalConcept Iterator> {
    template <Integral D> void Iterator::set_step(D step);
};

concept MutableStepIteratorConcept<boost_concepts::Mutable_ForwardIteratorConcept Iterator> : StepIteratorConcept<Iterator> {};

{% endhighlight %}

<!--
GIL currently provides a step iterator whose value_type models PixelValueConcept.
In addition, the step is specified in memory units (which are bytes or bits).
This is necessary, for example, when implementing an iterator navigating along a column of pixels - the size of a row of pixels may sometimes not be divisible by the size of a pixel; for example rows may be word-aligned.
-->

いまのところ、GILは`PixelValueConcept`に基づいて実装された`value_type`をもつステップIteratorを提供します。
そのとき、ステップにはメモリ上での単位(バイト単位かビット単位か)が指定されています。
例えば、Pixelの列に沿って走査するIteratorを実装する場合などに必要だからです。
各行のサイズは、Pixelのサイズで割り切れるとは限りません。
例えば、各行にワード単位アラインメントが施されているかもしれません。

<!--
To advance in bytes/bits, the base iterator must model MemoryBasedIteratorConcept.
A memory-based iterator has an inherent memory unit, which is either a bit or a byte.
It must supply functions returning the number of bits per memory unit (1 or 8), the current step in memory units, the memory-unit distance between two iterators, and a reference a given distance in memunits away.
It must also supply a function that advances an iterator a given distance in memory units.
memunit_advanced and memunit_advanced_ref have a default implementation but some iterators may supply a more efficient version:
-->

数バイト毎または数ビット毎に進む場合、そのBase Iteratorは`MemoryBasedIteratorConcept`に基づいて実装されていなければなりません。
メモリベースIteratorは、1ビットまたは1バイトの固有のメモリ単位をもっています。
そして、メモリ単位をビット数(1または8)で返す関数、メモリ単位で数えた現在のステップ数を返す関数、メモリ単位で数えた2つのIterator間の距離を返す関数、メモリ単位に基づいて指定された距離分だけ先にある参照を返す関数を提供しなければなりません。
また、メモリ単位に基づいて指定された距離分だけIteratorを進める関数も提供しなければなりません。
`memunit_advanced`と`memunit_advanced_ref`はデフォルトの実装をもっていますが、いくつかのIteratorではより効率的なバージョンを提供しているかもしれません。

{% highlight C++ %}

concept MemoryBasedIteratorConcept<boost_concepts::RandomAccessTraversalConcept Iterator> {
    typename byte_to_memunit<Iterator>; where metafunction<byte_to_memunit<Iterator> >;
    std::ptrdiff_t      memunit_step(const Iterator&);
    std::ptrdiff_t      memunit_distance(const Iterator& , const Iterator&);
    void                memunit_advance(Iterator&, std::ptrdiff_t diff);
    Iterator            memunit_advanced(const Iterator& p, std::ptrdiff_t diff) { Iterator tmp; memunit_advance(tmp,diff); return tmp; }
    Iterator::reference memunit_advanced_ref(const Iterator& p, std::ptrdiff_t diff) { return *memunit_advanced(p,diff); }
};

{% endhighlight %}

<!--
It is useful to be able to construct a step iterator over another iterator.
More generally, given a type, we want to be able to construct an equivalent type that allows for dynamically specified horizontal step:
-->

他のIteratorからステップIteratorを構築できれば便利です。
より一般的に言えば、ある型を与えたとき、それと等価で水平方向のステップ数を動的に指定可能な型を構築したいのです。

{% highlight C++ %}

concept HasDynamicXStepTypeConcept<typename T> {
    typename dynamic_x_step_type<T>;
        where Metafunction<dynamic_x_step_type<T> >;
};

{% endhighlight %}

<!--
All models of pixel iterators, locators and image views that GIL provides support HasDynamicXStepTypeConcept.
-->

GILが提供する全てのPixel Iterator、Locator、Image ViewのModelは、`HasDynamicXStepConcept`をサポートしています。

<!--
Related Concepts:

StepIteratorConcept<Iterator>
MutableStepIteratorConcept<Iterator>
MemoryBasedIteratorConcept<Iterator>
HasDynamicXStepTypeConcept<T>
-->

#### 関連するConcept:

- `StepIteratorConcept<Iterator>`
- `MutableStepIteratorConcept<Iterator>`
- `MemoryBasedIteratorConcept<Iterator>`
- `HasDynamicXStepTypeConcept<T>`

<!--
Models:

All standard memory-based iterators GIL currently provides model MemoryBasedIteratorConcept.
GIL provides the class memory_based_step_iterator which models PixelIteratorConcept, StepIteratorConcept, and MemoryBasedIteratorConcept.
It takes the base iterator as a template parameter (which must model PixelIteratorConcept and MemoryBasedIteratorConcept) and allows changing the step dynamically.
GIL's implementation contains the base iterator and a ptrdiff_t denoting the number of memory units (bytes or bits) to skip for a unit step.
It may also be used with a negative number.
GIL provides a function to create a step iterator from a base iterator and a step:
-->

#### Model:

現在GILが提供している基本的なメモリベースIteratorは、全て`MemoryBasedIteratorConcept`に基づいたModelです。
GILは`PixelIteratorConcept`、`StepIteratorConcept`、`MemoryBasedIteratorConcept`に基づいたModelである`memory_based_step_iterator`クラスを提供しています。
これは、テンプレートのパラメータとしてBase Iterator(`PixelIteratorConcept`と`MemoryBasedIteratorConcept`に基づいたModelでなければなりません)をとり、ステップを動的に変更することを許可します。
GILの実装では、Base Iteratorと、1ステップで進む数をメモリ単位に基づいて示す`ptrdiff_t`を含みます。
`ptrdiff_t`には負の数を使うこともできます。
GILは、Base Iteratorとステップを指定することでステップIteratorを作成する関数を提供しています。

{% highlight C++ %}

template <typename I>  // Models MemoryBasedIteratorConcept, HasDynamicXStepTypeConcept
typename dynamic_x_step_type<I>::type make_step_iterator(const I& it, std::ptrdiff_t step);

{% endhighlight %}

<!--
GIL also provides a model of an iterator over a virtual array of pixels, position_iterator.
It is a step iterator that keeps track of the pixel position and invokes a function object to get the value of the pixel upon dereferencing.
It models PixelIteratorConcept and StepIteratorConcept but not MemoryBasedIteratorConcept.
-->

GILは、`position_iterator`という、仮想的なPixel配列に対するIteratorのModelも提供しています。
これは、Pixelの位置情報を保持し、その位置にあるPixelの値を間接参照で取得する関数オブジェクトを実行するステップIteratorです。
これは、`PixelIteratorConcept`と`StepIteratorConcept`に基づいたModelですが、`MemoryBasedIteratorConcept`に基づいたModelではありません。

<!--
Pixel Locator

A Locator allows for navigation in two or more dimensions.
Locators are N-dimensional iterators in spirit, but we use a different name because they don't satisfy all the requirements of iterators.
For example, they don't supply increment and decrement operators because it is unclear which dimension the operators should advance along.
N-dimensional locators model the following concept:
-->

### <a name="section_08_5"> Pixel Locator

Locatorは2次元もしくはそれ以上の次元でのナビゲーションを可能にします。
Locatorは、本来であればN次元Iteratorと呼ぶべきですが、Iteratorが満たすべき要件を完全には満たしていないため、このように違う名前を使っています。
例を挙げると、Locatorは、どの軸にそって移動するべきか明確でないために、インクリメントやデクリメントを行う演算子を提供しません。
N次元Locatorは次のConceptに基づいたModelです。

{% highlight C++ %}

concept RandomAccessNDLocatorConcept<Regular Loc> {
    typename value_type;        // value over which the locator navigates
    typename reference;         // result of dereferencing
    typename difference_type; where PointNDConcept<difference_type>; // return value of operator-.
    typename const_t;           // same as Loc, but operating over immutable values
    typename cached_location_t; // type to store relative location (for efficient repeated access)
    typename point_t  = difference_type;

    static const size_t num_dimensions; // dimensionality of the locator
    where num_dimensions = point_t::num_dimensions;

    // The difference_type and iterator type along each dimension. The iterators may only differ in
    // difference_type. Their value_type must be the same as Loc::value_type
    template <size_t D> struct axis {
        typename coord_t = point_t::axis<D>::coord_t;
        typename iterator; where RandomAccessTraversalConcept<iterator>; // iterator along D-th axis.
        where iterator::value_type == value_type;
    };

    // Defines the type of a locator similar to this type, except it invokes Deref upon dereferencing
    template <PixelDereferenceAdaptorConcept Deref> struct add_deref {
        typename type;        where RandomAccessNDLocatorConcept<type>;
        static type make(const Loc& loc, const Deref& deref);
    };

    Loc& operator+=(Loc&, const difference_type&);
    Loc& operator-=(Loc&, const difference_type&);
    Loc operator+(const Loc&, const difference_type&);
    Loc operator-(const Loc&, const difference_type&);

    reference operator*(const Loc&);
    reference operator[](const Loc&, const difference_type&);

    // Storing relative location for faster repeated access and accessing it
    cached_location_t Loc::cache_location(const difference_type&) const;
    reference operator[](const Loc&,const cached_location_t&);

    // Accessing iterators along a given dimension at the current location or at a given offset
    template <size_t D> axis<D>::iterator&       Loc::axis_iterator();
    template <size_t D> axis<D>::iterator const& Loc::axis_iterator() const;
    template <size_t D> axis<D>::iterator        Loc::axis_iterator(const difference_type&) const;
};

template <typename Loc>
concept MutableRandomAccessNDLocatorConcept : RandomAccessNDLocatorConcept<Loc> {
    where Mutable<reference>;
};

{% endhighlight %}

<!--
Two-dimensional locators have additional requirements:
-->

2次元Locatorには追加の要件があります。

{% highlight C++ %}

concept RandomAccess2DLocatorConcept<RandomAccessNDLocatorConcept Loc> {
    where num_dimensions==2;
    where Point2DConcept<point_t>;

    typename x_iterator = axis<0>::iterator;
    typename y_iterator = axis<1>::iterator;
    typename x_coord_t  = axis<0>::coord_t;
    typename y_coord_t  = axis<1>::coord_t;

    // Only available to locators that have dynamic step in Y
    //Loc::Loc(const Loc& loc, y_coord_t);

    // Only available to locators that have dynamic step in X and Y
    //Loc::Loc(const Loc& loc, x_coord_t, y_coord_t, bool transposed=false);

    x_iterator&       Loc::x();
    x_iterator const& Loc::x() const;
    y_iterator&       Loc::y();
    y_iterator const& Loc::y() const;

    x_iterator Loc::x_at(const difference_type&) const;
    y_iterator Loc::y_at(const difference_type&) const;
    Loc Loc::xy_at(const difference_type&) const;

    // x/y versions of all methods that can take difference type
    x_iterator        Loc::x_at(x_coord_t, y_coord_t) const;
    y_iterator        Loc::y_at(x_coord_t, y_coord_t) const;
    Loc               Loc::xy_at(x_coord_t, y_coord_t) const;
    reference         operator()(const Loc&, x_coord_t, y_coord_t);
    cached_location_t Loc::cache_location(x_coord_t, y_coord_t) const;

    bool      Loc::is_1d_traversable(x_coord_t width) const;
    y_coord_t Loc::y_distance_to(const Loc& loc2, x_coord_t x_diff) const;
};

concept MutableRandomAccess2DLocatorConcept<RandomAccess2DLocatorConcept Loc> : MutableRandomAccessNDLocatorConcept<Loc> {};

{% endhighlight %}

<!--
2D locators can have a dynamic step not just horizontally, but also vertically.
This gives rise to the Y equivalent of HasDynamicXStepTypeConcept:
-->

2次元Locatorは、水平方向だけではなく垂直方向にも、動的なステップをもつことができます。
これはつまり、Y軸における`HasDynamicXStepTypeConcept`です。

{% highlight C++ %}

concept HasDynamicYStepTypeConcept<typename T> {
    typename dynamic_y_step_type<T>;
        where Metafunction<dynamic_y_step_type<T> >;
};

{% endhighlight %}

<!--
All locators and image views that GIL provides model HasDynamicYStepTypeConcept.
-->

GILが提供する全てのLocatorとImage Viewは`HasDynamicYStepTypeConcept`に基づいたModelです。

<!--
Sometimes it is necessary to swap the meaning of X and Y for a given locator or image view type (for example, GIL provides a function to transpose an image view).
Such locators and views must be transposable:
-->

与えられたLocatorやImage Viewについて、X軸とY軸の入れ替えが必要になることがあります(例を挙げると、GILはImage Viewの転置変換を行う関数を提供しています)。
上記のようなLocatorやViewは転置変換可能でなければなりません。

{% highlight C++ %}

concept HasTransposedTypeConcept<typename T> {
    typename transposed_type<T>;
        where Metafunction<transposed_type<T> >;
};

{% endhighlight %}

<!--
All GIL provided locators and views model HasTransposedTypeConcept.
-->

GILが提供する全てのLocatorとViewは、`HasTransposedTypeConcept`に基づいたModelです。


<!--
The locators GIL uses operate over models of PixelConcept and their x and y dimension types are the same.
They model the following concept:
-->

GILが用いるLocatorは、`PixelConcept`のModel上で動作し、X軸とY軸の次元の型が同じです。
これらのLocatorは次に示すConceptに基づいたModelです。

{% highlight C++ %}

concept PixelLocatorConcept<RandomAccess2DLocatorConcept Loc> {
    where PixelValueConcept<value_type>;
    where PixelIteratorConcept<x_iterator>;
    where PixelIteratorConcept<y_iterator>;
    where x_coord_t == y_coord_t;

    typename coord_t = x_coord_t;
};

concept MutablePixelLocatorConcept<PixelLocatorConcept Loc> : MutableRandomAccess2DLocatorConcept<Loc> {};

{% endhighlight %}

<!--
Related Concepts:

HasDynamicYStepTypeConcept<T>
HasTransposedTypeConcept<T>
RandomAccessNDLocatorConcept<Locator>
MutableRandomAccessNDLocatorConcept<Locator>
RandomAccess2DLocatorConcept<Locator>
MutableRandomAccess2DLocatorConcept<Locator>
PixelLocatorConcept<Locator>
MutablePixelLocatorConcept<Locator>
-->

#### 関連するConcept:

- `HasDynamicYStepTypeConcept<T>`
- `HasTransposedTypeConcept<T>`
- `RandomAccessNDLocatorConcept<Locator>`
- `MutableRandomAccessNDLocatorConcept<Locator>`
- `RandomAccess2DLocatorConcept<Locator>`
- `MutableRandomAccess2DLocatorConcept<Locator>`
- `PixelLocatorConcept<Locator>`
- `MutablePixelLocatorConcept<Locator>`

<!--
Models:

GIL provides two models of PixelLocatorConcept - a memory-based locator, memory_based_2d_locator and a virtual locator virtual_2d_locator.
-->

#### Model:

GILは2種類の`PixelLocatorConcept`のModelを提供します。
メモリベースLocatorである`memory_based_2d_locator`と、Virtual Locatorである`virtual_2d_locator`です。

<!--
memory_based_2d_locator is a locator over planar or interleaved images that have their pixels in memory.
It takes a model of StepIteratorConcept over pixels as a template parameter.
(When instantiated with a model of MutableStepIteratorConcept, it models MutablePixelLocatorConcept).
-->

`memory_based_2d_locator`は、メモリ上にPixelがあるプラナー画像もしくはインタリーブ画像におけるLocatorです。
このLocatorは、テンプレートのパラメータとして`StepIteratorConcept`のModelを取ります。
(`MutableStepIteratorConcept` Modelの場合を例にすると、これは`MutablePixelLocatorConcept`に基づいたModelです。)

{% highlight C++ %}

template <typename StepIterator>  // Models StepIteratorConcept, MemoryBasedIteratorConcept
class memory_based_2d_locator;

{% endhighlight %}

<!--
The step of StepIterator must be the number of memory units (bytes or bits) per row (thus it must be memunit advanceable).
The class memory_based_2d_locator is a wrapper around StepIterator and uses it to navigate vertically, while its base iterator is used to navigate horizontally.
-->

ステップIteratorのステップは、各行において、メモリ単位(バイト数かビット数で示されます)の倍数でなければなりません(すなわち、ステップIteratorはメモリ単位で移動しなければなりません)。
`memory_based_2d_locator`クラスはステップIteratorのラッパであり、垂直方向のナビゲートにステップIteratorが使われる一方で、水平方向のナビゲートにはそのステップIteratorのBase Iteratorが使われます。

<!--
Combining fundamental and step iterators allows us to create locators that describe complex pixel memory organizations.
First, we have a choice of iterator to use for horizontal direction, i.e. for iterating over the pixels on the same row.
Using the fundamental and step iterators gives us four choices:

pixel<T,C>* (for interleaved images)
planar_pixel_iterator<T*,C> (for planar images)
memory_based_step_iterator<pixel<T,C>*> (for interleaved images with non-standard step)
memory_based_step_iterator<planar_pixel_iterator<T*,C> > (for planar images with non-standard step)
-->

Base IteratorとステップIteratorの合成によって、Pixelについての複雑なメモリ配置を記述したLocatorの作成が可能になります。
始めに、水平方向すなわちに同じ行にあるPixelに対する走査に用いるIteratorを選択します。
Base IteratorやステップIteratorには、4つの選択肢が与えられています。

- `pixel<T,C>*` (インタリーブ画像用)
- `planar_pixel_iterator<T*,C>` (プラナー画像用)
- `memory_based_step_iterator<pixel<T,C>*>` (標準以外のステップをもつインタリーブ画像用)
- `memory_based_step_iterator<planar_pixel_iterator<T*,C> >` (標準以外のステップをもつプラナー画像用)

<!--
Of course, one could provide their own custom x-iterator.
One such example described later is an iterator adaptor that performs color conversion when dereferenced.
-->

もちろん、独自の水平方向Iteratorを提供することもできます。
この先に記述されている一例として、間接参照された際に色変換を実行するIteratorアダプタがあります。

<!--
Given a horizontal iterator XIterator, we could choose the y-iterator, the iterator that moves along a column, as memory_based_step_iterator<XIterator> with a step equal to the number of memory units (bytes or bits) per row.
Again, one is free to provide their own y-iterator.
-->

水平方向Iteratorである`XIterator`が与えられるとき、メモリ単位(バイト数かビット数で示されます)の倍数と等しいステップをもつ`memory_based_step_iterator<XIterator>`として、ある列に沿って移動するIteratorである垂直方向Iteratorを選ぶことができます。
ここでも、独自の垂直方向Iteratorを提供することは自由です。

<!--
Then we can instantiate memory_based_2d_locator<memory_based_step_iterator<XIterator> > to obtain a 2D pixel locator, as the diagram indicates:
-->

ここでは、ダイアグラムが示すように、2次元Pixel Locatorを得るために`memory_based_2d_locator<memory_based_step_iterator<XIterator> >`を作成します。

![2次元Pixel Locator](http://hironishihara.github.com/GILDesignGuide-ja/src/img/step_iterator.gif "2次元Pixel Locator")

<!--
virtual_2d_locator is a locator that is instantiated with a function object invoked upon dereferencing a pixel.
It returns the value of a pixel given its X,Y coordiantes.
Virtual locators can be used to implement virtual image views that can model any user-defined function.
See the GIL tutorial for an example of using virtual locators to create a view of the Mandelbrot set.
-->

`virtual_2d_locator`は、間接参照したPixelに対して実行される関数オブジェクトと共に作成されるLocatorです。
これは、与えられたXY座標にあるPixelの値を返します。
Virtual Locatorは、任意のユーザ定義関数に基づいたVirtual Image Viewを実装するときに使うことができます。
Virtual Locatorを用いてマンデルブロ集合のViewを作成する例については、GILチュートリアルを参照ください。

<!--
Both the virtual and the memory-based locators subclass from pixel_2d_locator_base, a base class that provides most of the interface required by PixelLocatorConcept.
Users may find this base class useful if they need to provide other models of PixelLocatorConcept.
-->

Virtual LocatorとメモリベースLocatorは、`PixelLocatorConcept`から要求されるほとんどのインタフェースを提供する基本クラスである`pixel_2d_locator_base`のサブクラスです。
この基本クラスは、他の`PixelLocatorConcept`のModelを提供する必要が生じた際に利用できるかもしれません。

<!--
Here is some sample code using locators:
-->

ここで、Locatorを用いたサンプルコードをいくつか示します。

{% highlight C++ %}

loc=img.xy_at(10,10);            // start at pixel (x=10,y=10)
above=loc.cache_location(0,-1);  // remember relative locations of neighbors above and below
below=loc.cache_location(0, 1);
++loc.x();                       // move to (11,10)
loc.y()+=15;                     // move to (11,25)
loc-=point2<std::ptrdiff_t>(1,1);// move to (10,24)
*loc=(loc(0,-1)+loc(0,1))/2;     // set pixel (10,24) to the average of (10,23) and (10,25) (grayscale pixels only)
*loc=(loc[above]+loc[below])/2;  // the same, but faster using cached relative neighbor locations

{% endhighlight %}

<!--
The standard GIL locators are fast and lightweight objects.
For example, the locator for a simple interleaved image consists of one raw pointer to the pixel location plus one integer for the row size in bytes, for a total of 8 bytes.
++loc.x() amounts to incrementing a raw pointer (or N pointers for planar images).
Computing 2D offsets is slower as it requires multiplication and addition. Filters, for example, need to access the same neighbors for every pixel in the image, in which case the relative positions can be cached into a raw byte difference using cache_location.
In the above example loc[above] for simple interleaved images amounts to a raw array index operator.
-->

標準的なGIL Locatorは、高速で軽量なオブジェクトです。
例を挙げると、シンプルなインタリーブ画像のためのLocatorは、Pixelの位置を示す生ポインタとバイト単位での行サイズを値にもつ整数型との合計8バイトで構成されます。
`++loc.x()`は、生ポインタのインクリメントと等価(プラナー画像の場合、N個のポインタのインクリメントと等価)です。
2次元オフセットの計算は、積と和を必要とするために比較的低速です。
例えば、フィルタ処理では画像中の各Pixelにおいて同じ位置関係にある隣接Pixelへのアクセスが必要ですが、そのような場合に、相対的な位置は`cache_location`を利用して差分を表す整数のなかにキャッシュできます。
上記の例で言うと、インタリーブ画像の`loc[above]`は生配列のインデクス演算子と等価です。

<!--
Iterator over 2D image

Sometimes we want to perform the same, location-independent operation over all pixels of an image.
In such a case it is useful to represent the pixels as a one-dimensional array.
GIL's iterator_from_2d is a random access traversal iterator that visits all pixels in an image in the natural memory-friendly order left-to-right inside top-to-bottom.
It takes a locator, the width of the image and the current X position.
This is sufficient information for it to determine when to do a "carriage return". Synopsis:
-->

### <a name="section_08_6"> 2次元画像上でのIterator

ときには、画像中の全Pixelに対して位置に依存しない一律の処理を実行したいといった場合も考えられます。
このようなとき、一律に扱うPixel全てを一次元配列のように扱うことができれば便利です。
GILの`itarator_from_2d`は、画像中の全Pixelを左から右、上から下というメモリフレンドリな順序で走査するランダムアクセスIteratorです。
これは、ひとつのLocatorと画像の幅と現在のX座標をもっています。
これは"キャリッジリターン"のタイミングを決定するために十分な情報です。

<!--
Synopsis:
-->

#### Synopsis:

{% highlight C++ %}

template <typename Locator>  // Models PixelLocatorConcept
class iterator_from_2d {
public:
    iterator_from_2d(const Locator& loc, int x, int width);

    iterator_from_2d& operator++(); // if (++_x<_width) ++_p.x(); else _p+=point_t(-_width,1);

    ...
private:
    int _x, _width;
    Locator _p;
};

{% endhighlight %}

<!--
Iterating through the pixels in an image using iterator_from_2d is slower than going through all rows and using the x-iterator at each row.
This is because two comparisons are done per iteration step - one for the end condition of the loop using the iterators, and one inside iterator_from_2d::operator++ to determine whether we are at the end of a row.
For fast operations, such as pixel copy, this second check adds about 15% performance delay (measured for interleaved images on Intel platform).
GIL overrides some STL algorithms, such as std::copy and std::fill, when invoked with iterator_from_2d-s, to go through each row using their base x-iterators, and, if the image has no padding (i.e. iterator_from_2d::is_1d_traversable() returns true) to simply iterate using the x-iterators directly.
-->

`iterator_from_2d`を用いた画像中の全Pixelへの走査は、水平方向Iteratorを用いた各行での走査の全行分の合算よりも低速です。
これは、1ステップ毎にIteratorによるループの終了判定と`iterator_from_2d::operator++`による行の終端判定との2個の比較が行われることが原因です。
Pixelのコピーのような高速な処理では、この2個目の判定は約15%の遅延の原因となります(Intelプラットホーム上にて、インタリーブ画像で計測)。
GILは`std::copy`や`std::fill`といったいくつかのSTLアルゴリズムをオーバーライドしており、実行時に`iterator_from_2d`が渡された場合には、各行に対してBase Iteratorである水平方向Iteratorを用います。
また、画像にパディングがない(例：`iterator_from_2d::is_1d_traversable()`が`true`を返す)場合には、シンプルな水平方向Iteratorを直接使用します。


<!--
9. Image View
-->

## <a name="section_09"> 9. Image View

<!--
An image view is a generalization of STL's range concept to multiple dimensions.
Similar to ranges (and iterators), image views are shallow, don't own the underlying data and don't propagate their constness over the data.
For example, a constant image view cannot be resized, but may allow modifying the pixels.
For pixel-immutable operations, use constant-value image view (also called non-mutable image view).
Most general N-dimensional views satisfy the following concept:
-->

Image Viewは、STLのRangeというConceptを複数次元に一般化したものです。
Renge (と、そのIterator)と同様、Image Viewは浅く、自身でデータをもたず、自身のconst性をデータにまで伝えません。
例を挙げると、constantなImage Viewはサイズを変更できませんが、Pixelの値を変更することは可能かもしれません。
Pixelの値を変更しない処理には、値がconstantなImage View (non-mutableなImage Viewとも呼ばれます)を使用します。
最も一般的なN次元Viewは、次のConceptを満たします。

{% highlight C++ %}

concept RandomAccessNDImageViewConcept<Regular View> {
    typename value_type;      // for pixel-based views, the pixel type
    typename reference;       // result of dereferencing
    typename difference_type; // result of operator-(iterator,iterator) (1-dimensional!)
    typename const_t;  where RandomAccessNDImageViewConcept<View>; // same as View, but over immutable values
    typename point_t;  where PointNDConcept<point_t>; // N-dimensional point
    typename locator;  where RandomAccessNDLocatorConcept<locator>; // N-dimensional locator.
    typename iterator; where RandomAccessTraversalConcept<iterator>; // 1-dimensional iterator over all values
    typename reverse_iterator; where RandomAccessTraversalConcept<reverse_iterator>;
    typename size_type;       // the return value of size()

    // Equivalent to RandomAccessNDLocatorConcept::axis
    template <size_t D> struct axis {
        typename coord_t = point_t::axis<D>::coord_t;
        typename iterator; where RandomAccessTraversalConcept<iterator>;   // iterator along D-th axis.
        where SameType<coord_t, iterator::difference_type>;
        where SameType<iterator::value_type,value_type>;
    };

    // Defines the type of a view similar to this type, except it invokes Deref upon dereferencing
    template <PixelDereferenceAdaptorConcept Deref> struct add_deref {
        typename type;        where RandomAccessNDImageViewConcept<type>;
        static type make(const View& v, const Deref& deref);
    };

    static const size_t num_dimensions = point_t::num_dimensions;

    // Create from a locator at the top-left corner and dimensions
    View::View(const locator&, const point_type&);

    size_type        View::size()       const; // total number of elements
    reference        operator[](View, const difference_type&) const; // 1-dimensional reference
    iterator         View::begin()      const;
    iterator         View::end()        const;
    reverse_iterator View::rbegin()     const;
    reverse_iterator View::rend()       const;
    iterator         View::at(const point_t&);
    point_t          View::dimensions() const; // number of elements along each dimension
    bool             View::is_1d_traversable() const;   // Does an iterator over the first dimension visit each value?

    // iterator along a given dimension starting at a given point
    template <size_t D> View::axis<D>::iterator View::axis_iterator(const point_t&) const;

    reference operator()(View,const point_t&) const;
};

concept MutableRandomAccessNDImageViewConcept<RandomAccessNDImageViewConcept View> {
    where Mutable<reference>;
};

{% endhighlight %}

<!--
Two-dimensional image views have the following extra requirements:
-->

2次元のImage Viewは、次に示す追加の要件をもっています。

{% highlight C++ %}

concept RandomAccess2DImageViewConcept<RandomAccessNDImageViewConcept View> {
    where num_dimensions==2;

    typename x_iterator = axis<0>::iterator;
    typename y_iterator = axis<1>::iterator;
    typename x_coord_t  = axis<0>::coord_t;
    typename y_coord_t  = axis<1>::coord_t;
    typename xy_locator = locator;

    x_coord_t View::width()  const;
    y_coord_t View::height() const;

    // X-navigation
    x_iterator View::x_at(const point_t&) const;
    x_iterator View::row_begin(y_coord_t) const;
    x_iterator View::row_end  (y_coord_t) const;

    // Y-navigation
    y_iterator View::y_at(const point_t&) const;
    y_iterator View::col_begin(x_coord_t) const;
    y_iterator View::col_end  (x_coord_t) const;

    // navigating in 2D
    xy_locator View::xy_at(const point_t&) const;

    // (x,y) versions of all methods taking point_t
    View::View(x_coord_t,y_coord_t,const locator&);
    iterator View::at(x_coord_t,y_coord_t) const;
    reference operator()(View,x_coord_t,y_coord_t) const;
    xy_locator View::xy_at(x_coord_t,y_coord_t) const;
    x_iterator View::x_at(x_coord_t,y_coord_t) const;
    y_iterator View::y_at(x_coord_t,y_coord_t) const;
};

concept MutableRandomAccess2DImageViewConcept<RandomAccess2DImageViewConcept View>
  : MutableRandomAccessNDImageViewConcept<View> {};

{% endhighlight %}

<!--
Image views that GIL typically uses operate on value types that model PixelValueConcept and have some additional requirements:
-->

GILが通常用いるImage Viewは、`PixelValueConcept`に基づいたModelであるPixel型で動作し、いくつかの追加の要件をもっています。

{% highlight C++ %}

concept ImageViewConcept<RandomAccess2DImageViewConcept View> {
    where PixelValueConcept<value_type>;
    where PixelIteratorConcept<x_iterator>;
    where PixelIteratorConcept<y_iterator>;
    where x_coord_t == y_coord_t;

    typename coord_t = x_coord_t;

    std::size_t View::num_channels() const;
};

concept MutableImageViewConcept<ImageViewConcept View> : MutableRandomAccess2DImageViewConcept<View> {};

{% endhighlight %}

<!--
Two image views are compatible if they have compatible pixels and the same number of dimensions:
-->

ふたつのImage Viewが、互換性のあるPixelをもち、同じ次元数であるとき、それらのImage Viewの間には互換性があります。

{% highlight C++ %}

concept ViewsCompatibleConcept<ImageViewConcept V1, ImageViewConcept V2> {
    where PixelsCompatibleConcept<V1::value_type, V2::value_type>;
    where V1::num_dimensions == V2::num_dimensions;
};

{% endhighlight %}

<!--
Compatible views must also have the same dimensions (i.e. the same width and height).
Many algorithms taking multiple views require that they be pairwise compatible.
-->

互換性のあるViewは、同じサイズ(すなわち、同じWidthとHeight)でなければなりません。
複数のViewを用いるアルゴリズムの多くが、それぞれのViewの間での互換性を要求します。

<!--
Related Concepts:

RandomAccessNDImageViewConcept<View>
MutableRandomAccessNDImageViewConcept<View>
RandomAccess2DImageViewConcept<View>
MutableRandomAccess2DImageViewConcept<View>
ImageViewConcept<View>
MutableImageViewConcept<View>
ViewsCompatibleConcept<View1,View2>
-->

#### 関連するConcept:

- `RandomAccessNDImageViewConcept<View>`
- `MutableRandomAccessNDImageViewConcept<View>`
- `RandomAccess2DImageViewConcept<View>`
- `MutableRandomAccess2DImageViewConcept<View>`
- `ImageViewConcept<View>`
- `MutableImageViewConcept<View>`
- `ViewsCompatibleConcept<View1,View2>`

<!--
Models:

GIL provides a model for ImageViewConcept called image_view.
It is templated over a model of PixelLocatorConcept.
(If instantiated with a model of MutablePixelLocatorConcept, it models MutableImageViewConcept).
-->

#### Model:

GILは`image_view`と呼ばれる`ImageViewConcept`のためのModelを提供します。
それは`PixelLocatorConcept`のModelをパラメータにしたテンプレートになっています。
(`MutablePixelLocatorConcept`のModelを用いてインスタンス化された場合には、それは`MutableImageViewConcept`に基づいたModelになります。)

<!--
Synopsis:
-->

#### Synopsis:

{% highlight C++ %}

template <typename Locator>  // Models PixelLocatorConcept (could be MutablePixelLocatorConcept)
class image_view {
public:
    typedef Locator xy_locator;
    typedef iterator_from_2d<Locator> iterator;
    ...
private:
    xy_locator _pixels;     // 2D pixel locator at the top left corner of the image view range
    point_t    _dimensions; // width and height
};

{% endhighlight %}

<!--
Image views are lightweight objects.
A regular interleaved view is typically 16 bytes long - two integers for the width and height (inside dimensions) one for the number of bytes between adjacent rows (inside the locator) and one pointer to the beginning of the pixel block.

Algorithms:
-->

Image Viewは軽量なオブジェクトです。
正規のインタリーブ形式Viewであれば、基本的に16バイトです。
その内訳は、Dimensionsに含まれるWidthとHeightを表す2つの整数と、Locatorに含まれる1行のバイト数を示す整数と、Pixel配列の先頭を指すポインタです。

#### Algorithm:

<!--
Creating Views from Raw Pixels

Standard image views can be constructed from raw data of any supported color space, bit depth, channel ordering or planar vs. interleaved structure.
Interleaved views are constructed using interleaved_view, supplying the image dimensions, number of bytes per row, and a pointer to the first pixel:
-->

### <a name="section_09_1"> メモリ上のPixel生データからのImage View作成

一般的なImage Viewは、サポートされているどのようなColor Space、Channel深度、Channel順、プラナー形式またはインタリーブ形式の生データからでも構成することができます。
インタリーブ形式Viewは、画像のDimensionsと1行あたりのバイト数と最初のPixelを指すポインタを指定した`interleaved_view`を使って構成されます。

{% highlight C++ %}

template <typename Iterator> // Models pixel iterator (like rgb8_ptr_t or rgb8c_ptr_t)
image_view<...> interleaved_view(ptrdiff_t width, ptrdiff_t height, Iterator pixels, ptrdiff_t rowsize)

{% endhighlight %}

<!--
Planar views are defined for every color space and take each plane separately.
Here is the RGB one:
-->

プラナー形式Viewは、あらゆるColor Spaceのために定義されており、各Planeを個別に用意します。
ここに、RGB形式のプラナー形式Viewを示します。

{% highlight C++ %}

template <typename IC>  // Models channel iterator (like bits8* or const bits8*)
image_view<...> planar_rgb_view(ptrdiff_t width, ptrdiff_t height,
                                 IC r, IC g, IC b, ptrdiff_t rowsize);

{% endhighlight %}

<!--
Note that the supplied pixel/channel iterators could be constant (read-only), in which case the returned view is a constant-value (immutable) view.
-->

戻り値のViewが値がconstantな(immutableな)Viewである場合、提供されるPixel/Channel Iteratorがconstant (read-only)になることに注意してください。

<!--
Creating Image Views from Other Image Views

It is possible to construct one image view from another by changing some policy of how image data is interpreted.
The result could be a view whose type is derived from the type of the source.
GIL uses the following metafunctions to get the derived types:
-->

### <a name="section_09_2"> 他のImage ViewからのImage View作成

画像データがどのように解釈されるのかを示したいくつかのポリシーを変更することで、あるImage Viewから他のImage Viewを構築することが可能です。
その結果は、元となる型から派生した型のViewになっている可能性もあります。
GILは、生成された型を取得するために、次に示すメタ関数を用います。

{% highlight C++ %}

// Some result view types
template <typename View>
struct dynamic_xy_step_type : public dynamic_y_step_type<typename dynamic_x_step_type<View>::type> {};

template <typename View>
struct dynamic_xy_step_transposed_type : public dynamic_xy_step_type<typename transposed_type<View>::type> {};

// color and bit depth converted view to match pixel type P
template <typename SrcView, // Models ImageViewConcept
          typename DstP,    // Models PixelConcept
          typename ColorConverter=gil::default_color_converter>
struct color_converted_view_type {
    typedef ... type;     // image view adaptor with value type DstP, over SrcView
};

// single-channel view of the N-th channel of a given view
template <typename SrcView>
struct nth_channel_view_type {
    typedef ... type;
};

{% endhighlight %}

<!--
GIL Provides the following view transformations:
-->

GILは、次に示すView変換を提供します。

{% highlight C++ %}

// flipped upside-down, left-to-right, transposed view
template <typename View> typename dynamic_y_step_type<View>::type             flipped_up_down_view(const View& src);
template <typename View> typename dynamic_x_step_type<View>::type             flipped_left_right_view(const View& src);
template <typename View> typename dynamic_xy_step_transposed_type<View>::type transposed_view(const View& src);

// rotations
template <typename View> typename dynamic_xy_step_type<View>::type            rotated180_view(const View& src);
template <typename View> typename dynamic_xy_step_transposed_type<View>::type rotated90cw_view(const View& src);
template <typename View> typename dynamic_xy_step_transposed_type<View>::type rotated90ccw_view(const View& src);

// view of an axis-aligned rectangular area within an image
template <typename View> View                                                 subimage_view(const View& src,
             const View::point_t& top_left, const View::point_t& dimensions);

// subsampled view (skipping pixels in X and Y)
template <typename View> typename dynamic_xy_step_type<View>::type            subsampled_view(const View& src,
             const View::point_t& step);

template <typename View, typename P>
color_converted_view_type<View,P>::type                                       color_converted_view(const View& src);
template <typename View, typename P, typename CCV> // with a custom color converter
color_converted_view_type<View,P,CCV>::type                                   color_converted_view(const View& src);

template <typename View>
nth_channel_view_type<View>::view_t                                           nth_channel_view(const View& view, int n);

{% endhighlight %}

<!--
The implementations of most of these view factory methods are straightforward.
Here is, for example, how the flip views are implemented.
The flip upside-down view creates a view whose first pixel is the bottom left pixel of the original view and whose y-step is the negated step of the source.
-->

これらView Factoryメソッドの実装のほとんどは、単純です。
例として、反転Viewがどのように実装されているのかを示します。
上下反転Viewは、元のViewの最左下Pixelが先頭Pixelの、垂直方向ステップが元のステップと逆向きになったViewをつくります。

{% highlight C++ %}

template <typename View>
typename dynamic_y_step_type<View>::type flipped_up_down_view(const View& src) {
    gil_function_requires<ImageViewConcept<View> >();
    typedef typename dynamic_y_step_type<View>::type RView;
    return RView(src.dimensions(),typename RView::xy_locator(src.xy_at(0,src.height()-1),-1));
}

{% endhighlight %}

<!--
The call to gil_function_requires ensures (at compile time) that the template parameter is a valid model of ImageViewConcept.
Using it generates easier to track compile errors, creates no extra code and has no run-time performance impact.
We are using the boost::concept_check library, but wrapping it in gil_function_requires, which performs the check if the BOOST_GIL_USE_CONCEPT_CHECK is set.
It is unset by default, because there is a significant increase in compile time when using concept checks.
We will skip gil_function_requires in the code examples in this guide for the sake of succinctness.
-->

`gil_function_requires`関数の呼び出しは、テンプレートのパラメータが`ImageViewConcept`の有効なModelであることを(コンパイル時に)保証します。
これを使うことで、コンパイルエラーの追跡は容易になり、余計なコードの生成されず、実行時のパフォーマンスへの影響もありません。
私たちは`boost::concept_check`ライブラリを使用していますが、`BOOST_GIL_USE_CONCEPT_CHECK`がセットされているときにだけチェックを実行するように、`gil_function_requires`の中に`boost::concept_check`ライブラリをラップしています。
そして、デフォルトでは`BOOST_GIL_USE_CONCEPT_CHECK`はセットされていません。なぜなら、コンセプトチェックを使用するとコンパイルタイムが大幅に長くなるからです。
このガイド内のサンプルコードでは、簡潔さのために`gil_function_requires`はスキップすることにします。

<!--
Image views can be freely composed (see section 12. Useful Metafunctions and Typedefs for the typedefs rgb16_image_t and gray16_step_view_t):
-->

Image Viewは自由に構成することが出来ます("第12章 メタ関数とTypedef"を参照ください)。

{% highlight C++ %}

rgb16_image_t img(100,100);    // an RGB interleaved image

// grayscale view over the green (index 1) channel of img
gray16_step_view_t green=nth_channel_view(view(img),1);

// 50x50 view of the green channel of img, upside down and taking every other pixel in X and in Y
gray16_step_view_t ud_fud=flipped_up_down_view(subsampled_view(green,2,2));

{% endhighlight %}

<!--
As previously stated, image views are fast, constant-time, shallow views over the pixel data.
The above code does not copy any pixels; it operates on the pixel data allocated when img was created.
-->

前述のとおり、Image Viewは高速で、定量時間で動作する、浅いViewです。
上記のコードではひとつのPixelもコピーされていません。
すなわち、ViewはImageが作られたときに確保されたPixelデータを操作するのです。

<!--
STL-Style Algorithms on Image Views

Image views provide 1D iteration of their pixels via begin() and end() methods, which makes it possible to use STL algorithms with them.
However, using nested loops over X and Y is in many cases more efficient.
The algorithms in this section resemble STL algorithms, but they abstract away the nested loops and take views (as opposed to ranges) as input.
-->

### <a name="section_09_3"> Image View上で動作するSTL-Styleアルゴリズム

Image Viewは`begin()`メソッドと`end()`メソッドを通じて1次元走査を提供しており、Image Viewを対象にSTLアルゴリズムを使うことを可能にしています。
しかしながら、多くの場合、XとYでネストされたループを使う方がより効果的です。
ここで紹介するアルゴリズムはSTLアルゴリズムと共通点がありますが、ネストされたループを抽象化し、入力として(Rangeではなく)Viewを受け取ります。

{% highlight C++ %}

// Equivalents of std::copy and std::uninitialized_copy
// where ImageViewConcept<V1>, MutableImageViewConcept<V2>, ViewsCompatibleConcept<V1,V2>
template <typename V1, typename V2>
void copy_pixels(const V1& src, const V2& dst);
template <typename V1, typename V2>
void uninitialized_copy_pixels(const V1& src, const V2& dst);

// Equivalents of std::fill and std::uninitialized_fill
// where MutableImageViewConcept<V>, PixelConcept<Value>, PixelsCompatibleConcept<Value,V::value_type>
template <typename V, typename Value>
void fill_pixels(const V& dst, const Value& val);
template <typename V, typename Value>
void uninitialized_fill_pixels(const V& dst, const Value& val);

// Equivalent of std::for_each
// where ImageViewConcept<V>, boost::UnaryFunctionConcept<F>
// where PixelsCompatibleConcept<V::reference, F::argument_type>
template <typename V, typename F>
F for_each_pixel(const V& view, F fun);
template <typename V, typename F>
F for_each_pixel_position(const V& view, F fun);

// Equivalent of std::generate
// where MutableImageViewConcept<V>, boost::UnaryFunctionConcept<F>
// where PixelsCompatibleConcept<V::reference, F::argument_type>
template <typename V, typename F>
void generate_pixels(const V& dst, F fun);

// Equivalent of std::transform with one source
// where ImageViewConcept<V1>, MutableImageViewConcept<V2>
// where boost::UnaryFunctionConcept<F>
// where PixelsCompatibleConcept<V1::const_reference, F::argument_type>
// where PixelsCompatibleConcept<F::result_type, V2::reference>
template <typename V1, typename V2, typename F>
F transform_pixels(const V1& src, const V2& dst, F fun);
template <typename V1, typename V2, typename F>
F transform_pixel_positions(const V1& src, const V2& dst, F fun);

// Equivalent of std::transform with two sources
// where ImageViewConcept<V1>, ImageViewConcept<V2>, MutableImageViewConcept<V3>
// where boost::BinaryFunctionConcept<F>
// where PixelsCompatibleConcept<V1::const_reference, F::first_argument_type>
// where PixelsCompatibleConcept<V2::const_reference, F::second_argument_type>
// where PixelsCompatibleConcept<F::result_type, V3::reference>
template <typename V1, typename V2, typename V3, typename F>
F transform_pixels(const V1& src1, const V2& src2, const V3& dst, F fun);
template <typename V1, typename V2, typename V3, typename F>
F transform_pixel_positions(const V1& src1, const V2& src2, const V3& dst, F fun);

// Copies a view into another, color converting the pixels if needed, with the default or user-defined color converter
// where ImageViewConcept<V1>, MutableImageViewConcept<V2>
// V1::value_type must be convertible to V2::value_type.
template <typename V1, typename V2>
void copy_and_convert_pixels(const V1& src, const V2& dst);
template <typename V1, typename V2, typename ColorConverter>
void copy_and_convert_pixels(const V1& src, const V2& dst, ColorConverter ccv);

// Equivalent of std::equal
// where ImageViewConcept<V1>, ImageViewConcept<V2>, ViewsCompatibleConcept<V1,V2>
template <typename V1, typename V2>
bool equal_pixels(const V1& view1, const V2& view2);

{% endhighlight %}

<!--
Algorithms that take multiple views require that they have the same dimensions.
for_each_pixel_position and transform_pixel_positions pass pixel locators, as opposed to pixel references, to their function objects.
This allows for writing algorithms that use pixel neighbors, as the tutorial demonstrates.
-->

複数のViewを取るアルゴリズムは、それらが同じDimensionsをもつことを要求します。
`for_each_pixel_position`と`transform_pixel_positions`は、Pixelの参照ではなく、Pixel Locatorを関数オブジェクトに渡します。
このことは、チュートリアルで示したように、隣接するPixelを使用するアルゴリズムの記述を可能にします。

<!--
Most of these algorithms check whether the image views are 1D-traversable.
A 1D-traversable image view has no gaps at the end of the rows.
In other words, if an x_iterator of that view is advanced past the last pixel in a row it will move to the first pixel of the next row.
When image views are 1D-traversable, the algorithms use a single loop and run more efficiently.
If one or more of the input views are not 1D-traversable, the algorithms fall-back to an X-loop nested inside a Y-loop.
-->

これらのほとんどのアルゴリズムで、Image Viewが1次元走査可能であるか否かを調べています。
1次元走査可能なImage Viewは、各行の終端にギャップをもちません。
言い換えれば、1次元走査可能なImage Viewの`x_iterator`がある行の最後のPixelを通過したとき、その`x_iterator`はその次の行の最初のPixelへ移動しています。
Image Viewが1次元走査可能である場合、これらのアルゴリズムは一重のループを使用し、より効率的に動作します。
入力されたViewの中に1次元走査のできないViewが含まれていた場合、これらのアルゴリズムはYループのなかにネストされたXループへと後退します。

<!--
The algorithms typically delegate the work to their corresponding STL algorithms.
For example, copy_pixels calls std::copy either for each row, or, when the images are 1D-traversable, once for all pixels.
-->

基本的に、これらのアルゴリズムはその処理を対応するSTLアルゴリズムに委託します。
例を挙げると、`copy_pixels`は、1次元走査可能なViewの場合は全Pixelを対象に、それ以外のViewの場合は各行を対象に、`std::copy`を呼びます。

<!--
In addition, overloads are sometimes provided for the STL algorithms.
For example, std::copy for planar iterators is overloaded to perform std::copy for each of the planes.
std::copy over bitwise-copiable pixels results in std::copy over unsigned char, which STL typically implements via memmove.
-->

さらに、STLアルゴリズムのオーバーロードが提供される場合もあります。
例えば、プラナー形式Iteratorに対する`std::copy`は、各Planeに対して`std::copy`を実行するようにオーバーライドされます。
ビット単位でコピー可能なPixel上で動作する`std::copy`は、結果的に、STLでは一般的に`memmove`を介して実装される、unsigned charで動作する`std::copy`となります。

<!--
As a result copy_pixels may result in a single call to memmove for interleaved 1D-traversable views, or one per each plane of planar 1D-traversable views, or one per each row of interleaved non-1D-traversable images, etc.
-->

まとめると、`copy_pixels`は、インタリーブ形式の1次元走査可能なViewに対しては1回の`memmove`呼び出しになり、プラナー形式の1次元走査可能なViewに対しては各Plane毎の`memmove`呼び出しになり、インタリーブ形式の1次元走査ができないViewに対しては各行毎の`memmove`呼び出しになる、といった感じです。

<!--
GIL also provides some beta-versions of image processing algorithms, such as resampling and convolution in a numerics extension available on http://opensource.adobe.com/gil/download.html.
This code is in early stage of development and is not optimized for speed
-->

GILは、リサンプリングや畳み込みなど、いくつかの画像処理アルゴリズムのβバージョンの提供も行っています。
それらは、<http://opensource.adobe.com/gil/download.html> のnumerics extensionからダウンロードできます。
これらのコードは開発の初期段階にあり、速度の最適化はなされていません。


<!--
10. Image
-->

## <a name="section_10"> 10. Image

<!--
An image is a container that owns the pixels of a given image view.
It allocates them in its constructor and deletes them in the destructor.
It has a deep assignment operator and copy constructor. Images are used rarely, just when data ownership is important.
Most STL algorithms operate on ranges, not containers.
Similarly most GIL algorithms operate on image views (which images provide).
-->

Imageは、あるImage Viewが扱うPixel配列を所有するコンテナです。
Imageは、コンストラクタでPixel配列を確保し、デストラクタでそのPixel配列を解放します。
Imageは、深い代入演算子とコピーコンストラクタをもちます。
Imageは、データの所有が重要な意味をもつ場合にだけ使用されます。
ほとんどのSTLアルゴリズムは、コンテナ上ではなく、Range上で動作します。
ほどんどのGILアルゴリズムも同じように、(Imageが提供する)Image Viewの上で動作します。

<!--
In the most general form images are N-dimensional and satisfy the following concept:
-->

一般的に、ImageはN次元であり、次のConceptを満たします。

{% highlight C++ %}

concept RandomAccessNDImageConcept<typename Img> : Regular<Img> {
    typename view_t; where MutableRandomAccessNDImageViewConcept<view_t>;
    typename const_view_t = view_t::const_t;
    typename point_t      = view_t::point_t;
    typename value_type   = view_t::value_type;
    typename allocator_type;

    Img::Img(point_t dims, std::size_t alignment=1);
    Img::Img(point_t dims, value_type fill_value, std::size_t alignment);

    void Img::recreate(point_t new_dims, std::size_t alignment=1);
    void Img::recreate(point_t new_dims, value_type fill_value, std::size_t alignment);

    const point_t&        Img::dimensions() const;
    const const_view_t&   const_view(const Img&);
    const view_t&         view(Img&);
};

{% endhighlight %}

<!--
Two-dimensional images have additional requirements:
-->

2次元のImageは、追加の要件をもっています。

{% highlight C++ %}

concept RandomAccess2DImageConcept<RandomAccessNDImageConcept Img> {
    typename x_coord_t = const_view_t::x_coord_t;
    typename y_coord_t = const_view_t::y_coord_t;

    Img::Img(x_coord_t width, y_coord_t height, std::size_t alignment=1);
    Img::Img(x_coord_t width, y_coord_t height, value_type fill_value, std::size_t alignment);

    x_coord_t Img::width() const;
    y_coord_t Img::height() const;

    void Img::recreate(x_coord_t width, y_coord_t height, std::size_t alignment=1);
    void Img::recreate(x_coord_t width, y_coord_t height, value_type fill_value, std::size_t alignment);
};

{% endhighlight %}

<!--
GIL's images have views that model ImageViewConcept and operate on pixels.
-->

GILのImageは、`ImageViewConcept`に基づいたModelでありPixel上で動作するViewをもちます。

{% highlight C++ %}

concept ImageConcept<RandomAccess2DImageConcept Img> {
    where MutableImageViewConcept<view_t>;
    typename coord_t  = view_t::coord_t;
};

{% endhighlight %}

<!--
Images, unlike locators and image views, don't have 'mutable' set of concepts because immutable images are not very useful.
-->

LocatorやImage Viewと異なり、immutableなImageはそもそも不便であるため、わざわざ'mutable'を指定したConceptのセットはもちません。

<!--
Related Concepts:

RandomAccessNDImageConcept<Image>
RandomAccess2DImageConcept<Image>
ImageConcept<Image>
-->

#### 関連するConcept:

- `RandomAccessNDImageConcept<Image>`
- `RandomAccess2DImageConcept<Image>`
- `ImageConcept<Image>`

<!--
Models:

GIL provides a class, image, which is templated over the value type (the pixel) and models ImageConcept.
-->

#### Model:

GILは、Value型(すなわち、Pixel)をパラメータにもつテンプレートであり、ImageConceptに基づいたModelである、Imageクラスを提供します。

{% highlight C++ %}

template <typename Pixel, // Models PixelValueConcept
          bool IsPlanar,  // planar or interleaved image
          typename A=std::allocator<unsigned char> >
class image;

{% endhighlight %}

<!--
By default images have 1 memory unit (no) alignment - i.e. there is no padding at the end of rows.
Many operations are faster using such 1D-traversable images image_view::x_iterator can be used to traverse the pixels, instead of the more complicated image_view::iterator.
The image constructor takes an alignment parameter which allows for constructing images that are word-aligned or 8-byte aligned.
Beware that the alignment parameter is in memory units, which is usually but not always bytes.
Specifically, for bit-aligned images the memory unit is a bit.
-->

デフォルトのImageは、メモリ単位0個分でアラインメントされます。すなわち、各行の末尾にパディングはありません。
比較的複雑な`image_view::iterator`の代わりに、上記のような1次元走査可能なImageでは`image_view::x_iterator`をPixelの走査に使うことができるため、多くの処理が高速です。
Imageのコンストラクタは、ワードアラインメントや8バイトアラインメントといったImageの生成を可能にする、`alignment`パラメータを取ります。
この`alignment`パラメータが、必ずしもバイトであると限らない、メモリ単位によって指定されることに注意してください。
特に、ビットアラインメントImageのメモリ単位は、1ビットです。


<!--
11. Run-time specified images and image views
-->

## <a name="section_11"> 11. 実行時に型を指定するImageとImage View

<!--
The color space, channel depth, channel ordering, and interleaved/planar structure of an image are defined by the type of its template argument, which makes them compile-time bound.
Often some of these parameters are available only at run time.
Consider, for example, writing a module that opens the image at a given file path, rotates it and saves it back in its original color space and channel depth.
How can we possibly write this using our generic image?
What type is the image loading code supposed to return?
-->

Color Space、Channel深度、Channel順、インタリーブ形式/プラナー形式といったImageの構造は、コンパイル時に結びつけられる、テンプレートパラメータの型によって定義されます。
それらのパラメータのいくつかが実行時になって初めて決まるといった場合もしばしば存在します。
例として、与えられたパスにある画像を開き、くるりと回転させ、画像オリジナルのColor SpaceとChannel深度で保存する、というモジュールを書く場合を考えましょう。
我々のgeneric imageを使って、これをどのように書くことができるでしょうか？
読み込むコードから返されるImageはどのような型でしょうか？

<!--
GIL's dynamic_image extension allows for images, image views or any GIL constructs to have their parameters defined at run time.
Here is an example:
-->

GILの`dynamic_image` extensionは、実行時にパラメータを決めるImageやImage Viewやその他のGILクラスを実現します。
ここで、例を示します。

{% highlight C++ %}

#include <boost/gil/extension/dynamic_image/dynamic_image_all.hpp>
using namespace boost;

#define ASSERT_SAME(A,B) BOOST_STATIC_ASSERT((is_same< A,B >::value))

// Define the set of allowed images
typedef mpl::vector<rgb8_image_t, cmyk16_planar_image_t> my_images_t;

// Create any_image class (or any_image_view) class
typedef any_image<my_images_t> my_any_image_t;

// Associated view types are available (equivalent to the ones in image_t)
typedef any_image_view<mpl::vector2<rgb8_view_t,  cmyk16_planar_view_t > > AV;
ASSERT_SAME(my_any_image_t::view_t, AV);

typedef any_image_view<mpl::vector2<rgb8c_view_t, cmyk16c_planar_view_t> > CAV;
ASSERT_SAME(my_any_image_t::const_view_t, CAV);
ASSERT_SAME(my_any_image_t::const_view_t, my_any_image_t::view_t::const_t);

typedef any_image_view<mpl::vector2<rgb8_step_view_t, cmyk16_planar_step_view_t> > SAV;
ASSERT_SAME(typename dynamic_x_step_type<my_any_image_t::view_t>::type, SAV);

// Assign it a concrete image at run time:
my_any_image_t myImg = my_any_image_t(rgb8_image_t(100,100));

// Change it to another at run time. The previous image gets destroyed
myImg = cmyk16_planar_image_t(200,100);

// Assigning to an image not in the allowed set throws an exception
myImg = gray8_image_t();        // will throw std::bad_cast

{% endhighlight %}

<!--
any_image and any_image_view subclass from GIL's variant class, which breaks down the instantiated type into a non-templated underlying base type and a unique instantiation type identifier.
The underlying base instance is represented as a block of bytes.
The block is large enough to hold the largest of the specified types.
-->

`any_image`と`any_image_view`は、インスタンス化される型について非テンプレートの基本的なBase型と一意のインスタンス型識別子に分解した、GILの`variant`クラスのサブクラスです。
この基本的なBase型インスタンスは、バイトのブロックとして表わされます。
そのブロックは、指定された型の中で最大の型を保持するために十分な規模となります。

<!--
GIL's variant is similar to boost::variant in spirit (hence we borrow the name from there) but it differs in several ways from the current boost implementation.
Perhaps the biggest difference is that GIL's variant always takes a single argument, which is a model of MPL Random Access Sequence enumerating the allowed types.
Having a single interface allows GIL's variant to be used easier in generic code.
-->

GILの`variant`は`boost::variant`と考え方は似ています(だからこそ、そこから名前を拝借したわけです)が、現在のboostの実装とは幾つかの点で異なっています。
おそらく、最大の違いは、許可する型を列挙するMPLランダムアクセスシークエンスをGILの`variant`が引数として常に1個とることです。
インタフェイスをひとつにしたことで、GILの`variant`はジェネリックコードの中で使いやすくなっています。

<!--
Synopsis:
-->

#### Synopsis:

{% highlight C++ %}

template <typename Types>    // models MPL Random Access Container
class variant {
    ...           _bits;
    std::size_t   _index;
public:
    typedef Types types_t;

    variant();
    variant(const variant& v);
    virtual ‾variant();

    variant& operator=(const variant& v);
    template <typename TS> friend bool operator==(const variant<TS>& x, const variant<TS>& y);
    template <typename TS> friend bool operator!=(const variant<TS>& x, const variant<TS>& y);

    // Construct/assign to type T. Throws std::bad_cast if T is not in Types
    template <typename T> explicit variant(const T& obj);
    template <typename T> variant& operator=(const T& obj);

    // Construct/assign by swapping T with its current instance. Only possible if they are swappable
    template <typename T> explicit variant(T& obj, bool do_swap);
    template <typename T> void move_in(T& obj);

    template <typename T> static bool has_type();

    template <typename T> const T& _dynamic_cast() const;
    template <typename T>       T& _dynamic_cast();

    template <typename T> bool current_type_is() const;
};

template <typename UOP, typename Types>
   UOP::result_type apply_operation(variant<Types>& v, UOP op);
template <typename UOP, typename Types>
   UOP::result_type apply_operation(const variant<Types>& v, UOP op);

template <typename BOP, typename Types1, typename Types2>
   BOP::result_type apply_operation(      variant<Types1>& v1,       variant<Types2>& v2, UOP op);

template <typename BOP, typename Types1, typename Types2>
   BOP::result_type apply_operation(const variant<Types1>& v1,       variant<Types2>& v2, UOP op);

template <typename BOP, typename Types1, typename Types2>
   BOP::result_type apply_operation(const variant<Types1>& v1, const variant<Types2>& v2, UOP op);

{% endhighlight %}

<!--
GIL's any_image_view and any_image are subclasses of variant:
-->

GILの`any_image_view`と`any_image`は`variant`のサブクラスです。

{% highlight C++ %}

template <typename ImageViewTypes>
class any_image_view : public variant<ImageViewTypes> {
public:
    typedef ... const_t; // immutable equivalent of this
    typedef std::ptrdiff_t x_coord_t;
    typedef std::ptrdiff_t y_coord_t;
    typedef point2<std::ptrdiff_t> point_t;

    any_image_view();
    template <typename T> explicit any_image_view(const T& obj);
    any_image_view(const any_image_view& v);

    template <typename T> any_image_view& operator=(const T& obj);
    any_image_view&                       operator=(const any_image_view& v);

    // parameters of the currently instantiated view
    std::size_t num_channels()  const;
    point_t     dimensions()    const;
    x_coord_t   width()         const;
    y_coord_t   height()        const;
};

template <typename ImageTypes>
class any_image : public variant<ImageTypes> {
    typedef variant<ImageTypes> parent_t;
public:
    typedef ... const_view_t;
    typedef ... view_t;
    typedef std::ptrdiff_t x_coord_t;
    typedef std::ptrdiff_t y_coord_t;
    typedef point2<std::ptrdiff_t> point_t;

    any_image();
    template <typename T> explicit any_image(const T& obj);
    template <typename T> explicit any_image(T& obj, bool do_swap);
    any_image(const any_image& v);

    template <typename T> any_image& operator=(const T& obj);
    any_image&                       operator=(const any_image& v);

    void recreate(const point_t& dims, unsigned alignment=1);
    void recreate(x_coord_t width, y_coord_t height, unsigned alignment=1);

    std::size_t num_channels()  const;
    point_t     dimensions()    const;
    x_coord_t   width()         const;
    y_coord_t   height()        const;
};

{% endhighlight %}

<!--
Operations are invoked on variants via apply_operation passing a function object to perform the operation.
The code for every allowed type in the variant is instantiated and the appropriate instantiation is selected via a switch statement.
Since image view algorithms typically have time complexity at least linear on the number of pixels, the single switch statement of image view variant adds practically no measurable performance overhead compared to templated image views.
-->

処理を実行するための関数オブジェクトを`apply_operation`に渡すことによって、`variant`上での処理が実行されます。
その`variant`で許可されている全ての型のためのコードがインスタンス化され、`switch`文を経由して適切なインスタンスが選択されます。
Image Viewアルゴリズムは、一般的に、少なくともPixel数に対して線形な時間計算量をもつことから、Image View `variant`による1個の`switch`文がテンプレートによるImage Viewと比較して実際に測定可能なほどのパフォーマンスのオーバーヘッドを加えることはありません。

<!--
Variants behave like the underlying type.
Their copy constructor will invoke the copy constructor of the underlying instance.
Equality operator will check if the two instances are of the same type and then invoke their operator==, etc.
The default constructor of a variant will default-construct the first type.
That means that any_image_view has shallow default-constructor, copy-constructor, assigment and equaty comparison, whereas any_image has deep ones.
-->

`variant`は基本的な型と同じように振舞います。
`variant`のコピーコンストラクタは、基本的な型のインスタンスのコピーコンストラクタを呼び出します。
比較演算子は、ふたつのインスタンスの型が同じか否かを確認し、それから`operator==`などを呼び出します。
`variant`のデフォルトコンストラクタは、最初に指定されている型のデフォルトコンストラクタを呼びます。
これは、`any_image_view`が浅いデフォルトコンストラクタ、コピーコンストラクタ、代入、比較をもつ一方で、`any_image`が深いデフォルトコンストラクタ、コピーコンストラクタ、代入、比較をもつことを意味します。

<!--
It is important to note that even though any_image_view and any_image resemble the static image_view and image, they do not model the full requirements of ImageViewConcept and ImageConcept.
In particular they don't provide access to the pixels.
There is no "any_pixel" or "any_pixel_iterator" in GIL.
Such constructs could be provided via the variant mechanism, but doing so would result in inefficient algorithms, since the type resolution would have to be performed per pixel.
Image-level algorithms should be implemented via apply_operation.
That said, many common operations are shared between the static and dynamic types.
In addition, all of the image view transformations and many STL-like image view algorithms have overloads operating on any_image_view, as illustrated with copy_pixels:
-->

`any_image_view`や`any_image`はstaticな`image_view`や`image`と似ていますが、`ImageViewConcept`や`ImageConcept`の全ての要件に基づいたModelではない点に注意しなければなりません。
特に`variant`はPixelへのアクセスを提供しません。
GILに`any_pixel`や`any_pixel_iterator`といったものは存在しません。
`variant`メカニズムを用いてこのようなものを提供することは可能ですが、各Pixelに対して型解決を実行しなければならないことから、結局のところ非効率なアルゴリズムとなってしまうでしょう。
Imageレベルのアルゴリズムは`apply_operation`経由で実装されるべきです。
つまり、多くの基本的な処理については、staticな型とdynamicな型の間で共有されているのです。
加えて、全てのImage View変換とSTL-likeなImage Viewアルゴリズムの多くは、`copy_pixels`の例で示したように、`any_image`で動作するオーバーロードをもっています。

{% highlight C++ %}

rgb8_view_t v1(...);  // concrete image view
bgr8_view_t v2(...);  // concrete image view compatible with v1 and of the same size
any_image_view<Types>  av(...);  // run-time specified image view

// Copies the pixels from v1 into v2.
// If the pixels are incompatible triggers compile error
copy_pixels(v1,v2);

// The source or destination (or both) may be run-time instantiated.
// If they happen to be incompatible, throws std::bad_cast
copy_pixels(v1, av);
copy_pixels(av, v2);
copy_pixels(av, av);

{% endhighlight %}

<!--
By having algorithm overloads supporting dynamic constructs, we create a base upon which it is possible to write algorithms that can work with either compile-time or runtime images or views.
The following code, for example, uses the GIL I/O extension to turn an image on disk upside down:
-->

dynamicな型をサポートするアルゴリズムのオーバーロードをもつことによって、コンパイル時に決定するImageやViewと実行時に決定するImageやViewのどちらかで動作するアルゴリズムの記述を可能にする基盤を作ります。
例を挙げると、次に示すコードはディスク上の画像を上下反転して返すGIL I/O extensionを使用します。

{% highlight C++ %}

#include <boost¥gil¥extension¥io¥jpeg_dynamic_io.hpp>

template <typename Image>    // Could be rgb8_image_t or any_image<...>
void save_180rot(const std::string& file_name) {
    Image img;
    jpeg_read_image(file_name, img);
    jpeg_write_view(file_name, rotated180_view(view(img)));
}

{% endhighlight %}

<!--
It can be instantiated with either a compile-time or a runtime image because all functions it uses have overloads taking runtime constructs.
For example, here is how rotated180_view is implemented:
-->

使用する全ての関数が実行時に決定する型のインスタンスを引数に取るオーバーロードをもっていることから、コンパイル時に決定するImageと実行時に決定するImageのどちらであってもインスタンス化が可能です。
ここで、`rotated180_view`がどのように実装されているかを示します。

{% highlight C++ %}

// implementation using templated view
template <typename View>
typename dynamic_xy_step_type<View>::type rotated180_view(const View& src) { ... }

namespace detail {
    // the function, wrapped inside a function object
    template <typename Result> struct rotated180_view_fn {
        typedef Result result_type;
        template <typename View> result_type operator()(const View& src) const {
            return result_type(rotated180_view(src));
        }
    };
}

// overloading of the function using variant. Takes and returns run-time bound view.
// The returned view has a dynamic step
template <typename ViewTypes> inline // Models MPL Random Access Container of models of ImageViewConcept
typename dynamic_xy_step_type<any_image_view<ViewTypes> >::type rotated180_view(const any_image_view<ViewTypes>& src) {
    return apply_operation(src,detail::rotated180_view_fn<typename dynamic_xy_step_type<any_image_view<ViewTypes> >::type>());
}

{% endhighlight %}

<!--
Variants should be used with caution (especially algorithms that take more than one variant) because they instantiate the algorithm for every possible model that the variant can take.
This can take a toll on compile time and executable size.
Despite these limitations, variant is a powerful technique that allows us to combine the speed of compile-time resolution with the flexibility of run-time resolution.
It allows us to treat images of different parameters uniformly as a collection and store them in the same container.
-->

`variant`は、それが取りうる全てのModel毎にアルゴリズムをインスタンス化するので、(特に、ふたつ以上の`variant`を引数に取るアルゴリズムでは)注意して用いるべきです。
これは、コンパイル時間と実行ファイルのサイズに甚大な影響を与える可能性があります。
これらのような欠点もありますが、`variant`はコンパイル時の型解決によるスピードと実行時の型解決による柔軟性を組み合わせることを可能にする強力な手法です。
`variant`は、パラメータが異なる複数の画像をひとつのコレクションとして扱い、それらを同じコンテナに収めることを可能にします。


<!--
12. Useful Metafunctions and Typedefs
-->

## <a name="section_12"> 12. メタ関数とtypedef

<!--
Flexibility comes at a price. GIL types can be very long and hard to read.
To address this problem, GIL provides typedefs to refer to any standard image, pixel iterator, pixel locator, pixel reference or pixel value.
They follow this pattern:
-->

柔軟性は高くつきます。
GILの型はとても長くて読みづらいものになりがちです。
この問題に取り組むために、GILは基本的なImage、Pixel Iterator、Pixel Locator、Pixel参照、Pixel値を参照する`typedef`を提供しています。
それらの`typedef`は、次のようなパターンに従います。

<!--
ColorSpace + BitDepth + ["s|f"] + ["c"] + ["_planar"] + ["_step"] + ClassType + "_t"
-->

ColorSpace + BitDepth + [`s` \| `f`] + [`c`] + [`_planar`] + [`_step`] + ClassType + `_t`

<!--
Where ColorSpace also indicates the ordering of components.
Examples are rgb, bgr, cmyk, rgba.
BitDepth can be, for example, 8,16,32. By default the bits are unsigned integral type.
Append s to the bit depth to indicate signed integral, or f to indicate floating point.
c indicates object whose associated pixel reference is immutable.
_planar indicates planar organization (as opposed to interleaved).
_step indicates the type has a dynamic step and ClassType is _image (image, using a standard allocator), _view (image view), _loc (pixel locator), _ptr (pixel iterator), _ref (pixel reference), _pixel (pixel value).
Here are examples:
-->

ここでのColorSpaceは、色要素とその順序を示します。
例えば、`rgb`、`bgr`、`cmyk`、`rgba`です。
BitDepthは、例えば、`8`、`16`、`32`をとることができます。
デフォルトでは、これらのビット数は符号なし整数型です。
BitDepthに続く`s`は符号つき整数型であることを示し、また、`f`は浮動小数点型であることを示します。
`c`は、オブジェクトに結びつけられたPixel参照がimmutableであることを示します。
`_planar`は、(インタリーブ形式ではなく)プラナー形式であることを示しています。
`_step`は、その型がdynamicなステップをもつことを示します。
ClassTypeは、(一般的なアロケータを用いるImageであることを示す)`_image`、(Image Viewであることを示す)`_view`、(Pixel Locatorであることを示す)`_loc`、(Pixel Iteratorであることを示す)`_ptr`、(Pixel参照であることを示す)`_ref`、(Pixel値であることを示す)`_pixel`をとります。
いくつか例を挙げます。

{% highlight C++ %}

bgr8_image_t               i;     // 8-bit unsigned (unsigned char) interleaved BGR image
cmyk16_pixel_t;            x;     // 16-bit unsigned (unsigned short) CMYK pixel value;
cmyk16sc_planar_ref_t      p(x);  // const reference to a 16-bit signed integral (signed short) planar CMYK pixel x.
rgb32f_planar_step_ptr_t   ii;    // step iterator to a floating point 32-bit (float) planar RGB pixel.

{% endhighlight %}

<!--
GIL provides the metafunctions that return the types of standard homogeneous memory-based GIL constructs given a channel type, a layout, and whether the construct is planar, has a step along the X direction, and is mutable:
-->

GILは、与えられたChannel型、Layout、プラナー形式であるか否か、X方向のステップをもっているか否か、mutableであるか否かの情報に基づいて、基本的なホモジーニアスメモリベースGILクラスの型を返すメタ関数を提供します。

{% highlight C++ %}

template <typename ChannelValue, typename Layout, bool IsPlanar=false,                     bool IsMutable=true>
struct pixel_reference_type { typedef ... type; };

template <typename Channel, typename Layout>
struct pixel_value_type { typedef ... type; };

template <typename ChannelValue, typename Layout, bool IsPlanar=false, bool IsStep=false,  bool IsMutable=true>
struct iterator_type { typedef ... type; };

template <typename ChannelValue, typename Layout, bool IsPlanar=false, bool IsXStep=false, bool IsMutable=true>
struct locator_type { typedef ... type; };

template <typename ChannelValue, typename Layout, bool IsPlanar=false, bool IsXStep=false, bool IsMutable=true>
struct view_type { typedef ... type; };

template <typename ChannelValue, typename Layout, bool IsPlanar=false, typename Alloc=std::allocator<unsigned char> >
struct image_type { typedef ... type; };

template <typename BitField, typename ChannelBitSizeVector, typename Layout, typename Alloc=std::allocator<unsigned char> >
struct packed_image_type { typedef ... type; };

template <typename ChannelBitSizeVector, typename Layout, typename Alloc=std::allocator<unsigned char> >
struct bit_aligned_image_type { typedef ... type; };

{% endhighlight %}

<!--
There are also helper metafunctions to construct packed and bit-aligned images with up to five channels:
-->

5個までのChannelをもつバイト単位Imageとビット単位Imageを構築する、ヘルパメタ関数もあります。

{% highlight C++ %}

template <typename BitField, unsigned Size1,
          typename Layout, typename Alloc=std::allocator<unsigned char> >
struct packed_image1_type { typedef ... type; };

template <typename BitField, unsigned Size1, unsigned Size2,
          typename Layout, typename Alloc=std::allocator<unsigned char> >
struct packed_image2_type { typedef ... type; };

template <typename BitField, unsigned Size1, unsigned Size2, unsigned Size3,
          typename Layout, typename Alloc=std::allocator<unsigned char> >
struct packed_image3_type { typedef ... type; };

template <typename BitField, unsigned Size1, unsigned Size2, unsigned Size3, unsigned Size4,
          typename Layout, typename Alloc=std::allocator<unsigned char> >
struct packed_image4_type { typedef ... type; };

template <typename BitField, unsigned Size1, unsigned Size2, unsigned Size3, unsigned Size4, unsigned Size5,
          typename Layout, typename Alloc=std::allocator<unsigned char> >
struct packed_image5_type { typedef ... type; };

template <unsigned Size1,
          typename Layout, typename Alloc=std::allocator<unsigned char> >
struct bit_aligned_image1_type { typedef ... type; };

template <unsigned Size1, unsigned Size2,
          typename Layout, typename Alloc=std::allocator<unsigned char> >
struct bit_aligned_image2_type { typedef ... type; };

template <unsigned Size1, unsigned Size2, unsigned Size3,
          typename Layout, typename Alloc=std::allocator<unsigned char> >
struct bit_aligned_image3_type { typedef ... type; };

template <unsigned Size1, unsigned Size2, unsigned Size3, unsigned Size4,
          typename Layout, typename Alloc=std::allocator<unsigned char> >
struct bit_aligned_image4_type { typedef ... type; };

template <unsigned Size1, unsigned Size2, unsigned Size3, unsigned Size4, unsigned Size5,
          typename Layout, typename Alloc=std::allocator<unsigned char> >
struct bit_aligned_image5_type { typedef ... type; };

{% endhighlight %}

<!--
Here ChannelValue models ChannelValueConcept.
We don't need IsYStep because GIL's memory-based locator and view already allow the vertical step to be specified dynamically.
Iterators and views can be constructed from a pixel type:
-->

この`ChannelValue`は`ChannelValueConcept`に基づいたModelです。
GILのメモリベースLocatorとViewでは、垂直方向ステップを動的に指定することが可能なので`IsYStep`は必要ありません。
IteratorとViewについては、Pixel型から構築することができます。

{% highlight C++ %}

template <typename Pixel, bool IsPlanar=false, bool IsStep=false, bool IsMutable=true>
struct iterator_type_from_pixel { typedef ... type; };

template <typename Pixel, bool IsPlanar=false, bool IsStepX=false, bool IsMutable=true>
struct view_type_from_pixel { typedef ... type; };

{% endhighlight %}

<!--
Using a heterogeneous pixel type will result in heterogeneous iterators and views.
Types can also be constructed from horizontal iterator:
-->

ヘテロジーニアスPixel型からは、ヘテロジーニアスIteratorとヘテロジーニアスViewが得られます。
また、次に示す各種の型については、水平方向Iteratorから構築することができます。

{% highlight C++ %}

template <typename XIterator>
struct type_from_x_iterator {
    typedef ... step_iterator_t;
    typedef ... xy_locator_t;
    typedef ... view_t;
};

{% endhighlight %}

<!--
There are metafunctions to construct the type of a construct from an existing type by changing one or more of its properties:
-->

既存の型の幾つかのプロパティを変更してクラス型を構築するメタ関数があります。

{% highlight C++ %}

template <typename PixelReference,
          typename ChannelValue, typename Layout, typename IsPlanar, typename IsMutable>
struct derived_pixel_reference_type {
    typedef ... type;  // Models PixelConcept
};

template <typename Iterator,
          typename ChannelValue, typename Layout, typename IsPlanar, typename IsStep, typename IsMutable>
struct derived_iterator_type {
    typedef ... type;  // Models PixelIteratorConcept
};

template <typename View,
          typename ChannelValue, typename Layout, typename IsPlanar, typename IsXStep, typename IsMutable>
struct derived_view_type {
    typedef ... type;  // Models ImageViewConcept
};

template <typename Image,
          typename ChannelValue, typename Layout, typename IsPlanar>
struct derived_image_type {
    typedef ... type;  // Models ImageConcept
};

{% endhighlight %}

<!--
You can replace one or more of its properties and use boost::use_default for the rest.
In this case IsPlanar, IsStep and IsMutable are MPL boolean constants.
For example, here is how to create the type of a view just like View, but being grayscale and planar:
-->

いくつかのプロパティを置き換えて、それ以外のプロパティには`boost::use_default`を使うことができます。
このケースにおいて、`IsPlanar`、`IsStep`、`IsMutable`はMPLブール型定数です。
例として、Viewとよく似た、ただしグレイスケール化とプラナー形式化を行った、新たなViewの型をどのように作成するのかを示します。

{% highlight C++ %}

typedef typename derived_view_type<View, boost::use_default, gray_t, mpl::true_>::type VT;

{% endhighlight %}

<!--
You can get pixel-related types of any pixel-based GIL constructs (pixels, iterators, locators and views) using the following metafunctions provided by PixelBasedConcept, HomogeneousPixelBasedConcept and metafunctions built on top of them:
-->

`PixelBasedConcept`と`HomogeneousPixelBasedConcept`とそれらの上に構築されたメタ関数から提供されるメタ関数を用いることで、PixelベースのGILクラス(Pixel、Iterator、Locator、View)からPixelに関する型を取得することができます。

{% highlight C++ %}

template <typename T> struct color_space_type { typedef ... type; };
template <typename T> struct channel_mapping_type { typedef ... type; };
template <typename T> struct is_planar { typedef ... type; };

// Defined by homogeneous constructs
template <typename T> struct channel_type { typedef ... type; };
template <typename T> struct num_channels { typedef ... type; };

{% endhighlight %}

<!--
These are metafunctions, some of which return integral types which can be evaluated like this:
-->

これらのメタ関数のいくつかは、次のように評価することが可能な整数型を返します。

{% highlight C++ %}

BOOST_STATIC_ASSERT(is_planar<rgb8_planar_view_t>::value == true);

{% endhighlight %}

<!--
GIL also supports type analysis metafunctions of the form:
[pixel_reference/iterator/locator/view/image] + "_is_" + [basic/mutable/step].
For example:
-->

GILは、次に示す命名規則に従う、型解析を行うメタ関数についてもサポートしています。

[`pixel_reference` \| `iterator` \| `locator` \| `view` \| `image`] + `_is_` + [`basic` \| `mutable` \| `step`]

例を挙げます。

{% highlight C++ %}

if (view_is_mutable<View>::value) {
   ...
}

{% endhighlight %}

<!--
A basic GIL construct is a memory-based construct that uses the built-in GIL classes and does not have any function object to invoke upon dereferencing.
For example, a simple planar or interleaved, step or non-step RGB image view is basic, but a color converted view or a virtual view is not.
-->

基本的なGILコンストラクトは、ビルトインGILクラスを用いたメモリベースコンストラクトであり、間接参照に対して実行されるいかなる関数オブジェクトももちません。
例を挙げると、単純なプラナーまたはインタリーブ形式でstepまたはnon-stepなRGB Image Viewは基本的なGILコンストラクトですが、Color Converted ViewやVirtual Viewはそうではありません。


<!--
13. I/O Extension
-->

## <a name="section_13"> 13. I/O Extension

<!--
GIL's I/O extension provides low level image i/o utilities.
It supports loading and saving several image formats, each of which requires linking against the corresponding library:
-->

GILのI/O Extensionは、ローレベルの画像I/Oユーティリティを提供します。
このI/O Extensionは、対応するライブラリとのリンクが必要な各種画像フォーマットについての読み込みや保存をサポートしています。

<!--
JPEG: To use JPEG files, include the file gil/extension/io/jpeg_io.hpp.
If you are using run-time images, you need to include gil/extension/io/jpeg_dynamic_io.hpp instead.
You need to compile and link against libjpeg.lib (available at http://www.ijg.org).
You need to have jpeglib.h in your include path.

TIFF: To use TIFF files, include the file gil/extension/io/tiff_io.hpp.
If you are using run-time images, you need to include gil/extension/io/tiff_dynamic_io.hpp instead.
You need to compile and link against libtiff.lib (available at http://www.libtiff.org).
You need to have tiffio.h in your include path.

PNG: To use PNG files, include the file gil/extension/io/png_io.hpp.
If you are using run-time images, you need to include gil/extension/io/png_dynamic_io.hpp instead.
You need to compile and link against libpng.lib (available at http://wwwlibpng.org).
You need to have png.h in your include path.
-->

- JPEG: JPEGファイルを利用するには、`gil/extension/io/jpeg_io.hpp`をインクルードする必要があります。
もし、実行時に型が決まるImageを利用するのであれば、かわりに`gil/extension/io/jpeg_dynamic_io.hpp`をインクルードする必要があります。
また、`libjpeg.lib` (<http://www.ijp.org> から利用可能です)のコンパイルとリンクを行い、`jpeglib.h`をInclude Pathに追加する必要があります。

- TIFF: TIFFファイルを利用するには、`gil/extension/io/tiff_io.hpp`をインクルードする必要があります。
もし、実行時に型が決まるImageを利用するのであれば、かわりに`gil/extension/io/dynamic_dynamic_io.hpp`をインクルードする必要があります。
また、`libtiff.lib` (<http://www.libtiff.org/> から利用可能です)のコンパイルとリンクを行い、`tiffio.h`をInclude Pathに追加する必要があります。

- PNG: PNGファイルを利用するには、`gil/extension/io/png_io.hpp`をインクルードする必要があります。
もし、実行時に型が決まるImageを利用するのであれば、かわりに`gil/extension/io/png_dynamic_io.hpp`をインクルードする必要があります。
また、`libpng.lib` (<http://www.libpng.org/> から利用可能です)のコンパイルとリンクを行い、`png.h`をInclude Pathに追加する必要があります。

<!--
You don't need to install all these libraries; just the ones you will use.
Here are the I/O APIs for JPEG files (replace "jpeg" with "tiff" or "png" for the APIs of the other libraries):
-->

これらのライブラリを全てインストールする必要はありません。
使うものだけで十分です。
JPEGファイル用のI/O APIを示します(他のライブラリを使う場合には、"jpeg"を"tiff"または"png"に置き換えてください)。

{% highlight C++ %}

// Returns the width and height of the JPEG file at the specified location.
// Throws std::ios_base::failure if the location does not correspond to a valid JPEG file
point2<std::ptrdiff_t> jpeg_read_dimensions(const char*);

// Allocates a new image whose dimensions are determined by the given jpeg image file, and loads the pixels into it.
// Triggers a compile assert if the image color space or channel depth are not supported by the JPEG library or by the I/O extension.
// Throws std::ios_base::failure if the file is not a valid JPEG file, or if its color space or channel depth are not
// compatible with the ones specified by Image
template <typename Img> void jpeg_read_image(const char*, Img&);

// Allocates a new image whose dimensions are determined by the given jpeg image file, and loads the pixels into it,
// color-converting and channel-converting if necessary.
// Triggers a compile assert if the image color space or channel depth are not supported by the JPEG library or by the I/O extension.
// Throws std::ios_base::failure if the file is not a valid JPEG file or if it fails to read it.
template <typename Img>               void jpeg_read_and_convert_image(const char*, Img&);
template <typename Img, typename CCV> void jpeg_read_and_convert_image(const char*, Img&, CCV color_converter);

// Loads the image specified by the given jpeg image file name into the given view.
// Triggers a compile assert if the view color space and channel depth are not supported by the JPEG library or by the I/O extension.
// Throws std::ios_base::failure if the file is not a valid JPEG file, or if its color space or channel depth are not
// compatible with the ones specified by View, or if its dimensions don't match the ones of the view.
template <typename View> void jpeg_read_view(const char*, const View&);

// Loads the image specified by the given jpeg image file name into the given view and color-converts (and channel-converts) it if necessary.
// Triggers a compile assert if the view color space and channel depth are not supported by the JPEG library or by the I/O extension.
// Throws std::ios_base::failure if the file is not a valid JPEG file, or if its dimensions don't match the ones of the view.
template <typename View>               void jpeg_read_and_convert_view(const char*, const View&);
template <typename View, typename CCV> void jpeg_read_and_convert_view(const char*, const View&, CCV color_converter);

// Saves the view to a jpeg file specified by the given jpeg image file name.
// Triggers a compile assert if the view color space and channel depth are not supported by the JPEG library or by the I/O extension.
// Throws std::ios_base::failure if it fails to create the file.
template <typename View> void jpeg_write_view(const char*, const View&);

// Determines whether the given view type is supported for reading
template <typename View> struct jpeg_read_support {
    static const bool value = ...;
};

// Determines whether the given view type is supported for writing
template <typename View> struct jpeg_write_support {
    static const bool value = ...;
};

{% endhighlight %}

<!--
If you use the dynamic image extension, make sure to include "jpeg_dynamic_io.hpp" instead of "jpeg_io.hpp".
In addition to the above methods, you have the following overloads dealing with dynamic images:
-->

Dynamic Image Extensionを使う場合には、"`jpeg_io.hpp`"に代えて"`jpeg_dynamic_io.hpp`"をインクルードするようにしてください。
Dynamic Imageを扱う場合には、上記のメソッドに加えて、次に示すオーバーロードをもちます。

{% highlight C++ %}

// Opens the given JPEG file name, selects the first type in Images whose color space and channel are compatible to those of the image file
// and creates a new image of that type with the dimensions specified by the image file.
// Throws std::ios_base::failure if none of the types in Images are compatible with the type on disk.
template <typename Images> void jpeg_read_image(const char*, any_image<Images>&);

// Saves the currently instantiated view to a jpeg file specified by the given jpeg image file name.
// Throws std::ios_base::failure if the currently instantiated view type is not supported for writing by the I/O extension
// or if it fails to create the file.
template <typename Views>  void jpeg_write_view(const char*, any_image_view<Views>&);

{% endhighlight %}

<!--
All of the above methods have overloads taking std::string instead of const char*
-->

上記の全てのメソッドは、`const char*`の代わりに`std::string`を取るオーバーロードをもちます。


<!--
14. Sample Code
-->

## <a name="section_14"> 14. サンプルコード

<!--
Pixel-level Sample Code

Here are some operations you can do with pixel values, pointers and references:
-->

### <a name="section_14_1"> Pixelレベルの処理

Pixelの値、ポインタ、参照を用いる処理を、ここにいくつか示します。

{% highlight C++ %}

rgb8_pixel_t p1(255,0,0);     // make a red RGB pixel
bgr8_pixel_t p2 = p1;         // RGB and BGR are compatible and the channels will be properly mapped.
assert(p1==p2);               // p2 will also be red.
assert(p2[0]!=p1[0]);         // operator[] gives physical channel order (as laid down in memory)
assert(semantic_at_c<0>(p1)==semantic_at_c<0>(p2)); // this is how to compare the two red channels
get_color(p1,green_t()) = get_color(p2,blue_t());  // channels can also be accessed by name

const unsigned char* r;
const unsigned char* g;
const unsigned char* b;
rgb8c_planar_ptr_t ptr(r,g,b); // constructing const planar pointer from const pointers to each plane

rgb8c_planar_ref_t ref=*ptr;   // just like built-in reference, dereferencing a planar pointer returns a planar reference

p2=ref; p2=p1; p2=ptr[7]; p2=rgb8_pixel_t(1,2,3);    // planar/interleaved references and values to RGB/BGR can be freely mixed

//rgb8_planar_ref_t ref2;      // compile error: References have no default constructors
//ref2=*ptr;                   // compile error: Cannot construct non-const reference by dereferencing const pointer
//ptr[3]=p1;                   // compile error: Cannot set the fourth pixel through a const pointer
//p1 = pixel<float, rgb_layout_t>();// compile error: Incompatible channel depth
//p1 = pixel<bits8, rgb_layout_t>();// compile error: Incompatible color space (even though it has the same number of channels)
//p1 = pixel<bits8,rgba_layout_t>();// compile error: Incompatible color space (even though it contains red, green and blue channels)

{% endhighlight %}

<!--
Here is how to use pixels in generic code:
-->

続いては、Pixelをジェネリックコードの中でどのように使うのかを示します。

{% highlight C++ %}

template <typename GrayPixel, typename RGBPixel>
void gray_to_rgb(const GrayPixel& src, RGBPixel& dst) {
    gil_function_requires<PixelConcept<GrayPixel> >();
    gil_function_requires<MutableHomogeneousPixelConcept<RGBPixel> >();

    typedef typename color_space_type<GrayPixel>::type gray_cs_t;
    BOOST_STATIC_ASSERT((boost::is_same<gray_cs_t,gray_t>::value));

    typedef typename color_space_type<RGBPixel>::type  rgb_cs_t;
    BOOST_STATIC_ASSERT((boost::is_same<rgb_cs_t,rgb_t>::value));

    typedef typename channel_type<GrayPixel>::type gray_channel_t;
    typedef typename channel_type<RGBPixel>::type  rgb_channel_t;

    gray_channel_t gray = get_color(src,gray_color_t());
    static_fill(dst, channel_convert<rgb_channel_t>(gray));
}

// example use patterns:

// converting gray l-value to RGB and storing at (5,5) in a 16-bit BGR interleaved image:
bgr16_view_t b16(...);
gray_to_rgb(gray8_pixel_t(33), b16(5,5));

// storing the first pixel of an 8-bit grayscale image as the 5-th pixel of 32-bit planar RGB image:
rgb32f_planar_view_t rpv32;
gray8_view_t gv8(...);
gray_to_rgb(*gv8.begin(), rpv32[5]);

{% endhighlight %}

<!--
As the example shows, both the source and the destination can be references or values, planar or interleaved, as long as they model PixelConcept and MutablePixelConcept respectively.
-->

この例が示すように、入力と出力が共に`PixelConcept`と`MutablePixelConcept`に各々基づいたModelである限りにおいて、それらは、参照であっても値であっても構いませんし、プラナー形式であってもインタリーブ形式であっても構いません。

<!--
Creating a Copy of an Image with a Safe Buffer

Suppose we want to convolve an image with multiple kernels, the largest of which is 2K+1 x 2K+1 pixels.
It may be worth creating a margin of K pixels around the image borders.
Here is how to do it:
-->

### <a name="section_14_2"> 安全のためのバッファを備えた、Imageのコピー

最大で2K+1×2K+1の2次元カーネルを用いて画像のコンボリュージョンを行いたい場合を想定します。
そのような場合には、画像の境界線の外側にK個のPixel分のマージンを作ってみるのはどうでしょうか。
次のように作成します。

{% highlight C++ %}

template <typename SrcView,   // Models ImageViewConcept (the source view)
          typename DstImage>  // Models ImageConcept     (the returned image)
void create_with_margin(const SrcView& src, int k, DstImage& result) {
    gil_function_requires<ImageViewConcept<SrcView> >();
    gil_function_requires<ImageConcept<DstImage> >();
    gil_function_requires<ViewsCompatibleConcept<SrcView, typename DstImage::view_t> >();

    result=DstImage(src.width()+2*k, src.height()+2*k);
    typename DstImage::view_t centerImg=subimage_view(view(result), k,k,src.width(),src.height());
    std::copy(src.begin(), src.end(), centerImg.begin());
}

{% endhighlight %}

<!--
We allocated a larger image, then we used subimage_view to create a shallow image of its center area of top left corner at (k,k) and of identical size as src, and finally we copied src into that center image.
If the margin needs initialization, we could have done it with fill_pixels.
Here is how to simplify this code using the copy_pixels algorithm:
-->

十分な大きさのImageを確保し、`subimage_view`を使って(k,k)を始点とする`src`と同じサイズの中心領域を指定する浅いImage(すなわち、View)を作成し、`src`をその中心領域へコピーします。
もしマージンに初期化が必要であれば、`fill_pixels`を実行しておくこともできるでしょう。
`copy_pixels`アルゴリズムを使って、このコードをいかにシンプルにするかを示します。

{% highlight C++ %}

template <typename SrcView, typename DstImage>
void create_with_margin(const SrcView& src, int k, DstImage& result) {
    result.recreate(src.width()+2*k, src.height()+2*k);
    copy_pixels(src, subimage_view(view(result), k,k,src.width(),src.height()));
}

{% endhighlight %}

<!--
(Note also that image::recreate is more efficient than operator=, as the latter will do an unnecessary copy construction).
Not only does the above example work for planar and interleaved images of any color space and pixel depth; it is also optimized.
GIL overrides std::copy - when called on two identical interleaved images with no padding at the end of rows, it simply does a memmove.
For planar images it does memmove for each channel. If one of the images has padding, (as in our case) it will try to do memmove for each row.
When an image has no padding, it will use its lightweight horizontal iterator (as opposed to the more complex 1D image iterator that has to check for the end of rows).
It choses the fastest method, taking into account both static and run-time parameters.
-->

(`image::recreate`は、`operator=`が不必要なコピーコンストラクションを行う分、効率的であることにも注意してください。)
上記のサンプルはColor Spece、Pixel深度、プラナー形式であるかインタリーブ形式であるかを問わずに動作するだけではありません。
最適化されているのです。
GILは`std::copy`をオーバーライドします。
すなわち、行の末尾にパディングのない同じサイズの2個のインタリーブImageの上で呼ばれた場合には、単に`memmove`を実行します。
プラナーImageのときには、各Channelに対して`memmove`を実行します。
一方のImageがパディングをもっていた場合、(ちょうど今回の場合と同じように、)各行に対して`memmove`を実行するでしょう。
Imageがパディングをもっていない場合、(行の末尾をチェックが必要な少々複雑な1次元Iteratorではなく、)軽量な水平方向Iteratorを使うでしょう。
staticなパラメータと実行時に型が決まるパラメータの両方を考慮して、最速の方法を選択します。

<!--
Histogram

The histogram can be computed by counting the number of pixel values that fall in each bin.
The following method takes a grayscale (one-dimensional) image view, since only grayscale pixels are convertible to integers:
-->

### <a name="section_14_3"> ヒストグラム

ヒストグラムは、各瓶に振り分けられたPixel値の数をカウントすることで得られます。
グレイスケールPixelは整数型に変換可能なので、次に示すメソッドはグレイスケール(単要素の)Image Viewを取ります。

{% highlight C++ %}

template <typename GrayView, typename R>
void grayimage_histogram(const GrayView& img, R& hist) {
    for (typename GrayView::iterator it=img.begin(); it!=img.end(); ++it)
        ++hist[*it];
}

{% endhighlight %}

<!--
Using boost::lambda and GIL's for_each_pixel algorithm, we can write this more compactly:
-->

`boost::lambda`とGILの`for_each_pixel`アルゴリズムを用いると、もっとコンパクトに書くことができます。

{% highlight C++ %}

template <typename GrayView, typename R>
void grayimage_histogram(const GrayView& v, R& hist) {
    for_each_pixel(v, ++var(hist)[_1]);
}

{% endhighlight %}

<!--
Where for_each_pixel invokes std::for_each and var and _1 are boost::lambda constructs.
To compute the luminosity histogram, we call the above method using the grayscale view of an image:
-->

ここの`for_each_pixel`は`std::for_each`を呼び出しており、`var`と`_1`は`boost::lambda`コンストラクトです。
明度のヒストグラムを算出するには、ImageのグレイスケールViewを使って上記のメソッドを呼び出します。

{% highlight C++ %}

template <typename View, typename R>
void luminosity_histogram(const View& v, R& hist) {
    grayimage_histogram(color_converted_view<gray8_pixel_t>(v),hist);
}

{% endhighlight %}

<!--
This is how to invoke it:
-->

これは、次のように呼び出します。

{% highlight C++ %}

unsigned char hist[256];
std::fill(hist,hist+256,0);
luminosity_histogram(my_view,hist);

{% endhighlight %}

<!--
If we want to view the histogram of the second channel of the image in the top left 100x100 area, we call:
-->

また、画像の2番目のChannelの左上100x100についてのヒストグラムが見たい場合には、次のように呼びます：

{% highlight C++ %}

grayimage_histogram(nth_channel_view(subimage_view(img,0,0,100,100),1),hist);

{% endhighlight %}

<!--
No pixels are copied and no extra memory is allocated - the code operates directly on the source pixels, which could be in any supported color space and channel depth.
They could be either planar or interleaved.
-->

Pixelがコピーされることもなければ、余計なメモリが確保されることもありません。
すなわち、サポートされたColor SpaceとChannel深度であれば、このコードは入力Pixelの上で直接実行されます。
プラナー形式でもインタリーブ形式でも問題ありません。

<!--
Using Image Views

The following code illustrates the power of using image views:
-->

### <a name="section_14_4"> Image Viewの使用

次のコードで、Image Viewの威力を説明したいと思います。

{% highlight C++ %}

jpeg_read_image("monkey.jpg", img);
step1=view(img);
step2=subimage_view(step1, 200,300, 150,150);
step3=color_converted_view<rgb8_view_t,gray8_pixel_t>(step2);
step4=rotated180_view(step3);
step5=subsampled_view(step4, 2,1);
jpeg_write_view("monkey_transform.jpg", step5);

{% endhighlight %}

<!--
The intermediate images are shown here:
-->

途中経過の画像をここに示します。

![途中経過](http://hironishihara.github.com/GILDesignGuide-ja/src/img/monkey_steps.jpg "途中経過")

<!--
Notice that no pixels are ever copied.
All the work is done inside jpeg_write_view.
If we call our luminosity_histogram with step5 it will do the right thing.
-->

Pixelは全くコピーされていません。
全ての作業は`jpeg_write_view`のなかで行われます。
さきほどの`luminosity_histgram`をstep5で呼べば、うまく動作するはずです。


<!--
15. Extending the Generic Image Library
-->

## <a name="section_15"> 15. Generic Image Libraryの拡張

<!--
You can define your own pixel iterators, locators, image views, images, channel types, color spaces and algorithms.
You can make virtual images that live on the disk, inside a jpeg file, somewhere on the internet, or even fully-synthetic images such as the Mandelbrot set.
As long as they properly model the corresponding concepts, they will work with any existing GIL code.
Most such extensions require no changes to the library and can thus be supplied in another module.
-->

GILでは、独自のPixel Iterator、Locator、Image View、Image、Channel型、Color Space、アルゴリズムを定義することができます。
ディスク上の画像やjpegファイルに保存された画像やインターネット上にある画像のVirtual Imageをつくることもできますし、マンデルブロ集合といった完全な合成画像までもつくることができます。
関連するコンセプトに基づいた適切なModelである限りにおいて、それらは既存のあらゆるGILコードと共に動作します。
このような拡張のほとんどはライブラリに変更を求めませんし、だからこそ、これらの拡張を他のモジュールへ提供することが可能なのです。

<!--
Defining New Color Spaces

Each color space is in a separate file.
To add a new color space, just copy one of the existing ones (like rgb.hpp) and change it accordingly.
If you want color conversion support, you will have to provide methods to convert between it and the existing color spaces (see color_convert.h).
For convenience you may want to provide useful typedefs for pixels, pointers, references and images with the new color space (see typedefs.h).
-->

### <a name="section_15_1"> 独自のColor Space定義

それぞれのColor Spaceは個別のファイルに記述されています。
新しいColor Spaceを追加するには、あるColor Space(たとえば、`rgb.hpp`)をコピーして、適当に編集するだけです。
もし色変換をサポートしたいと思ったら、新しいColor Spaceと既存のColor Spaceの変換を行うメソッドを提供しなければならないでしょう(`color_convert.hpp`を参照してください)。
利便性を考えて、Pixel、ポインタ、参照、Imageについての`typedef`を提供したくなるかもしれません(`typedefs.h`を参照してください)。

<!--
Defining New Channel Types

Most of the time you don't need to do anything special to use a new channel type.
You can just use it:
-->

### <a name="section_15_2"> 独自のChannel型定義

ほとんどの場合、新しいChannel型を使用する際に特別なことをする必要はありません。
それをただ使うだけです。

{% highlight C++ %}

typedef pixel<double,rgb_layout_t>   rgb64_pixel_t;    // 64 bit RGB pixel
typedef rgb64_pixel*                 rgb64_pixel_ptr_t;// pointer to 64-bit interleaved data
typedef image_type<double,rgb_layout_t>::type rgb64_image_t;    // 64-bit interleaved image

{% endhighlight %}

<!--
If you want to use your own channel class, you will need to provide a specialization of channel_traits for it (see channel.hpp).
If you want to do conversion between your and existing channel types, you will need to provide an overload of channel_convert.
-->

もし独自のChannel型を使いたいと考えたときには、そのChannel型のための`channel_traits`の実装を提供する必要があるでしょう(`channel.hpp`を参照ください)。
もし、独自のChannel型と既存のChannel型の変換を行いたいと考えたときには、`channel_convert`のオーバーロードを提供する必要があるでしょう。

<!--
Overloading Color Conversion

Suppose you want to provide your own color conversion.
For example, you may want to implement higher quality color conversion using color profiles.
Typically you may want to redefine color conversion only in some instances and default to GIL's color conversion in all other cases.
Here is, for example, how to overload color conversion so that color conversion to gray inverts the result but everything else remains the same:
-->

### <a name="section_15_3"> 色変換のオーバーロード

独自の色変換を提供したい場合について考えてみましょう。
例えば、カラープロファイルを使った高品質な色変換を実装したいと考えている場合などが考えられます。
いつものように、特定の組み合わせにおける色変換だけを再定義して、その他の組み合わせについては既存のGILが用意する色変換を使いたいと考えているかもしれません。
例として、反転したグレイスケール画像を結果として受け取るという色変換でオーバーロードする方法について示します。

{% highlight C++ %}

// make the default use GIL's default
template <typename SrcColorSpace, typename DstColorSpace>
struct my_color_converter_impl
  : public default_color_converter_impl<SrcColorSpace,DstColorSpace> {};

// provide specializations only for cases you care about
// (in this case, if the destination is grayscale, invert it)
template <typename SrcColorSpace>
struct my_color_converter_impl<SrcColorSpace,gray_t> {
    template <typename SrcP, typename DstP>  // Model PixelConcept
    void operator()(const SrcP& src, DstP& dst) const {
        default_color_converter_impl<SrcColorSpace,gray_t>()(src,dst);
        get_color(dst,gray_color_t())=channel_invert(get_color(dst,gray_color_t()));
    }
};

// create a color converter object that dispatches to your own implementation
struct my_color_converter {
    template <typename SrcP, typename DstP>  // Model PixelConcept
    void operator()(const SrcP& src,DstP& dst) const {
        typedef typename color_space_type<SrcP>::type SrcColorSpace;
        typedef typename color_space_type<DstP>::type DstColorSpace;
        my_color_converter_impl<SrcColorSpace,DstColorSpace>()(src,dst);
    }
};

{% endhighlight %}

<!--
GIL's color conversion functions take the color converter as an optional parameter.
You can pass your own color converter:
-->

GILの色変換関数はオプションのパラメータとしてカラーコンバータをとります。
独自のカラーコンバータを渡すこともできます。

{% highlight C++ %}

color_converted_view<gray8_pixel_t>(img_view,my_color_converter());

{% endhighlight %}

<!--
Defining New Image Views

You can provide your own pixel iterators, locators and views, overriding either the mechanism for getting from one pixel to the next or doing an arbitrary pixel transformation on dereference.
For example, let's look at the implementation of color_converted_view (an image factory method that, given any image view, returns a new, otherwise identical view, except that color conversion is performed on pixel access).
First we need to define a model of PixelDereferenceAdaptorConcept; a function object that will be called when we dereference a pixel iterator.
It will call color_convert to convert to the destination pixel type:
-->

### <a name="section_15_4"> 独自のImage View定義

あるPixelから他のPixelを得るメカニズムか間接参照に対して任意のPixel変換を行うメカニズムかをオーバーロードすることで、独自のPixel IteratorやLocatorやViewを提供することが出来ます。
例として、`color_converted_view` (あるImage Viewが与えられたとき、Pixelの色変換が実行される以外は全くそっくりの新しいViewを作成するfactoryメソッド)の実装をみてみましょう。
まずは、Pixel Iteratorから間接参照するときに呼ばれる関数オブジェクトである、`PixelDereferenceAdaptorConcept`のModelを定義する必要があります。
これは、目的のPixel型へ変換するために`color_convert`を呼びます。

{% highlight C++ %}

template <typename SrcConstRefP,  // const reference to the source pixel
          typename DstP>          // Destination pixel value (models PixelValueConcept)
class color_convert_deref_fn {
public:
    typedef color_convert_deref_fn const_t;
    typedef DstP                value_type;
    typedef value_type          reference;      // read-only dereferencing
    typedef const value_type&   const_reference;
    typedef SrcConstRefP        argument_type;
    typedef reference           result_type;
    BOOST_STATIC_CONSTANT(bool, is_mutable=false);

    result_type operator()(argument_type srcP) const {
        result_type dstP;
        color_convert(srcP,dstP);
        return dstP;
    }
};

{% endhighlight %}

<!--
We then use the add_deref member struct of image views to construct the type of a view that invokes a given function object (deref_t) upon dereferencing.
In our case, it performs color conversion:
-->

そのとき、間接参照の上で与えられた関数オブジェクト(`deref_t`)を実行するView型を構築する、Image Viewのメンバstructである`add_deref`を使用します。
このケースでは、色変換を実行します。

{% highlight C++ %}

template <typename SrcView, typename DstP>
struct color_converted_view_type {
private:
    typedef typename SrcView::const_t::reference src_pix_ref;  // const reference to pixel in SrcView
    typedef color_convert_deref_fn<src_pix_ref, DstP> deref_t; // the dereference adaptor that performs color conversion
    typedef typename SrcView::template add_deref<deref_t> add_ref_t;
public:
    typedef typename add_ref_t::type type; // the color converted view type
    static type make(const SrcView& sv) { return add_ref_t::make(sv, deref_t()); }
};

{% endhighlight %}

<!--
Finally our color_converted_view code simply creates color-converted view from the source view:
-->

最終的に、`color_converted_view`のコードは、元となるViewから簡単に`color-converted view`を作成します。

{% highlight C++ %}

template <typename DstP, typename View> inline
typename color_converted_view_type<View,DstP>::type color_convert_view(const View& src) {
    return color_converted_view_type<View,DstP>::make(src);
}

{% endhighlight %}

<!--
(The actual color convert view transformation is slightly more complicated, as it takes an optional color conversion object, which allows users to specify their own color conversion methods).
See the GIL tutorial for an example of creating a virtual image view that defines the Mandelbrot set.
-->

(実際の色変換Viewによる変換は、ユーザ独自の色変換メソッドの記述を許可する追加の色変換オブジェクトを取るなど、少し複雑です。)
マンデルブロ集合を定義するVirtual Image Viewを作成する例については、GILチュートリアルを見てください。


<!--
16. Technicalities
-->

## <a name="section_16"> 16. より専門的な事項

<!--
Creating a reference proxy

Sometimes it is necessary to create a proxy class that represents a reference to a given object.
Examples of these are GIL's reference to a planar pixel (planar_pixel_reference) and GIL's subbyte channel references.
Writing a reference proxy class can be tricky.
One problem is that the proxy reference is constructed as a temporary object and returned by value upon dereferencing the iterator:
-->

### <a name="section_16_1"> 参照Proxyの作成

与えられたオブジェクトへの参照となるProxy型の作成が必要となる場合があります。
これらの例として、GILのプラナー形式Pixelへの参照やGILのサブバイトChannelへの参照が挙げられます。
参照Proxy型の記述に際しては、注意が必要です。
問題の1番目は、Proxy参照が一時的なオブジェクトとして構築され、Iteratorの間接参照から取得した値を返すことです。

{% highlight C++ %}

struct rgb_planar_pixel_iterator {
   typedef my_reference_proxy<T> reference;
   reference operator*() const { return reference(red,green,blue); }
};

{% endhighlight %}

<!--
The problem arises when an iterator is dereferenced directly into a function that takes a mutable pixel:
-->

この問題は、mutableなPixel型を引数に取る関数がIteratorの間接参照から取得した値を引数に取るときに起こります。

{% highlight C++ %}

template <typename Pixel>    // Models MutablePixelConcept
void invert_pixel(Pixel& p);

rgb_planar_pixel_iterator myIt;
invert_pixel(*myIt);        // compile error!

{% endhighlight %}

<!--
C++ does not allow for matching a temporary object against a non-constant reference.
The solution is to:

Use const qualifier on all members of the reference proxy object:
-->

C++では一時的なオブジェクトを非constantな参照と組み合わせることを許可していません。
この問題は次のように解決します。

- 参照Proxyオブジェクトの全メンバに対して、const修飾子を使います

{% highlight C++ %}

template <typename T>
struct my_reference_proxy {
    const my_reference_proxy& operator=(const my_reference_proxy& p) const;
    const my_reference_proxy* operator->() const { return this; }
    ...
};

{% endhighlight %}

<!--
Use different classes to denote mutable and constant reference (maybe based on the constness of the template parameter)
Define the reference type of your iterator with const qualifier:
-->

- (おそらくはテンプレートパラメタのconst性に基づく)mutableでconstantな参照であることを示すためには、異なるクラスを使用します
- const修飾子を伴った、独自Iteratorの参照型を定義します

{% highlight C++ %}

struct iterator_traits<rgb_planar_pixel_iterator> {
   typedef const my_reference_proxy<T> reference;
};

{% endhighlight %}

<!--
A second important issue is providing an overload for swap for your reference class.
The default std::swap will not work correctly.
You must use a real value type as the temporary.
A further complication is that in some implementations of the STL the swap function is incorreclty called qualified, as std::swap.
The only way for these STL algorithms to use your overload is if you define it in the std namespace:
-->

2番目の問題は、独自の参照クラスをスワップするためのオーバーロードの提供についてです。
デフォルトの`std::swap`は正確に動作しません。
一時的な値として、実際の値をもつ型を使用しなければなりません。
さらに複雑なことには、STLのいくつかの実装では`swap`関数を呼んだ際に誤って正規の`std::swap`が呼ばれます。
これらのSTLアルゴリズムで独自のオーバーロードが用いられるようにするための唯一の方法は、それを`std::namespace`に定義することです。

{% highlight C++ %}

namespace std {
   template <typename T>
   void swap(my_reference_proxy<T>& x, my_reference_proxy<T>& y) {
      my_value<T> tmp=x;
      x=y;
      y=tmp;
   }
}

{% endhighlight %}

<!--
Lastly, remember that constructors and copy-constructors of proxy references are always shallow and assignment operators are deep.
-->

最後に、Proxy参照のコンストラクタやコピーコンストラクタは常に浅く、代入演算子は常に深いことを覚えておいてください。

<!--
We are grateful to Dave Abrahams, Sean Parent and Alex Stepanov for suggesting the above solution.
-->

上記の解決策を提案してくれたDave Abrahams、Sean Parent、Alex Stepanovに感謝します。


<!--
17. Conclusion
-->

## <a name="section_17"> 17. 結び

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
