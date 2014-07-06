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
12. Useful Metafunctions and Typedefs
-->

## 12. メタ関数とtypedef

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

```cpp
bgr8_image_t               i;     // 8-bit unsigned (unsigned char) interleaved BGR image
cmyk16_pixel_t;            x;     // 16-bit unsigned (unsigned short) CMYK pixel value;
cmyk16sc_planar_ref_t      p(x);  // const reference to a 16-bit signed integral (signed short) planar CMYK pixel x.
rgb32f_planar_step_ptr_t   ii;    // step iterator to a floating point 32-bit (float) planar RGB pixel.
```

<!--
GIL provides the metafunctions that return the types of standard homogeneous memory-based GIL constructs given a channel type, a layout, and whether the construct is planar, has a step along the X direction, and is mutable:
-->

GILは、与えられたChannel型、Layout、プラナー形式であるか否か、X方向のステップをもっているか否か、mutableであるか否かの情報に基づいて、基本的なホモジーニアスメモリベースGILクラスの型を返すメタ関数を提供します。

```cpp
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
```

<!--
There are also helper metafunctions to construct packed and bit-aligned images with up to five channels:
-->

5個までのChannelをもつバイト単位Imageとビット単位Imageを構築する、ヘルパメタ関数もあります。

```cpp
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
```

<!--
Here ChannelValue models ChannelValueConcept.
We don't need IsYStep because GIL's memory-based locator and view already allow the vertical step to be specified dynamically.
Iterators and views can be constructed from a pixel type:
-->

この`ChannelValue`は`ChannelValueConcept`に基づいたModelです。
GILのメモリベースLocatorとViewでは、垂直方向ステップを動的に指定することが可能なので`IsYStep`は必要ありません。
IteratorとViewについては、Pixel型から構築することができます。

```cpp
template <typename Pixel, bool IsPlanar=false, bool IsStep=false, bool IsMutable=true>
struct iterator_type_from_pixel { typedef ... type; };

template <typename Pixel, bool IsPlanar=false, bool IsStepX=false, bool IsMutable=true>
struct view_type_from_pixel { typedef ... type; };
```

<!--
Using a heterogeneous pixel type will result in heterogeneous iterators and views.
Types can also be constructed from horizontal iterator:
-->

ヘテロジーニアスPixel型からは、ヘテロジーニアスIteratorとヘテロジーニアスViewが得られます。
また、次に示す各種の型については、水平方向Iteratorから構築することができます。

```cpp
template <typename XIterator>
struct type_from_x_iterator {
    typedef ... step_iterator_t;
    typedef ... xy_locator_t;
    typedef ... view_t;
};
```

<!--
There are metafunctions to construct the type of a construct from an existing type by changing one or more of its properties:
-->

既存の型の幾つかのプロパティを変更してクラス型を構築するメタ関数があります。

```cpp
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
```

<!--
You can replace one or more of its properties and use boost::use_default for the rest.
In this case IsPlanar, IsStep and IsMutable are MPL boolean constants.
For example, here is how to create the type of a view just like View, but being grayscale and planar:
-->

いくつかのプロパティを置き換えて、それ以外のプロパティには`boost::use_default`を使うことができます。
このケースにおいて、`IsPlanar`、`IsStep`、`IsMutable`はMPLブール型定数です。
例として、Viewとよく似た、ただしグレイスケール化とプラナー形式化を行った、新たなViewの型をどのように作成するのかを示します。

```cpp
typedef typename derived_view_type<View, boost::use_default, gray_t, mpl::true_>::type VT;
```

<!--
You can get pixel-related types of any pixel-based GIL constructs (pixels, iterators, locators and views) using the following metafunctions provided by PixelBasedConcept, HomogeneousPixelBasedConcept and metafunctions built on top of them:
-->

`PixelBasedConcept`と`HomogeneousPixelBasedConcept`とそれらの上に構築されたメタ関数から提供されるメタ関数を用いることで、PixelベースのGILクラス(Pixel、Iterator、Locator、View)からPixelに関する型を取得することができます。

```cpp
template <typename T> struct color_space_type { typedef ... type; };
template <typename T> struct channel_mapping_type { typedef ... type; };
template <typename T> struct is_planar { typedef ... type; };

// Defined by homogeneous constructs
template <typename T> struct channel_type { typedef ... type; };
template <typename T> struct num_channels { typedef ... type; };
```

<!--
These are metafunctions, some of which return integral types which can be evaluated like this:
-->

これらのメタ関数のいくつかは、次のように評価することが可能な整数型を返します。

```cpp
BOOST_STATIC_ASSERT(is_planar<rgb8_planar_view_t>::value == true);
```

<!--
GIL also supports type analysis metafunctions of the form:
[pixel_reference/iterator/locator/view/image] + "_is_" + [basic/mutable/step].
For example:
-->

GILは、次に示す命名規則に従う、型解析を行うメタ関数についてもサポートしています。

[`pixel_reference` \| `iterator` \| `locator` \| `view` \| `image`] + `_is_` + [`basic` \| `mutable` \| `step`]

例を挙げます。

```cpp
if (view_is_mutable<View>::value) {
   ...
}
```

<!--
A basic GIL construct is a memory-based construct that uses the built-in GIL classes and does not have any function object to invoke upon dereferencing.
For example, a simple planar or interleaved, step or non-step RGB image view is basic, but a color converted view or a virtual view is not.
-->

基本的なGILコンストラクトは、ビルトインGILクラスを用いたメモリベースコンストラクトであり、間接参照に対して実行されるいかなる関数オブジェクトももちません。
例を挙げると、単純なプラナーまたはインタリーブ形式でstepまたはnon-stepなRGB Image Viewは基本的なGILコンストラクトですが、Color Converted ViewやVirtual Viewはそうではありません。
