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

<!--

A color space captures the set and interpretation of channels comprising a pixel.
It is an MPL random access sequence containing the types of all elements in the color space.
Two color spaces are considered compatible if they are equal (i.e. have the same set of colors in the same order).

Related Concepts:

ColorSpaceConcept<ColorSpace>
ColorSpacesCompatibleConcept<ColorSpace1,ColorSpace2>
ChannelMappingConcept<Mapping>
Models:

GIL currently provides the following color spaces: gray_t, rgb_t, rgba_t, and cmyk_t.
It also provides unnamed N-channel color spaces of two to five channels, devicen_t<2>, devicen_t<3>, devicen_t<4>, devicen_t<5>.
Besides the standard layouts, it provides bgr_layout_t, bgra_layout_t, abgr_layout_t and argb_layout_t.

As an example, here is how GIL defines the RGBA color space:

struct red_t{};
struct green_t{};
struct blue_t{};
struct alpha_t{};
typedef mpl::vector4<red_t,green_t,blue_t,alpha_t> rgba_t;
The ordering of the channels in the color space definition specifies their semantic order.
For example, red_t is the first semantic channel of rgba_t.
While there is a unique semantic ordering of the channels in a color space, channels may vary in their physical ordering in memory.
The mapping of channels is specified by ChannelMappingConcept, which is an MPL random access sequence of integral types.
A color space and its associated mapping are often used together. Thus they are grouped in GIL's layout:

template <typename ColorSpace,
          typename ChannelMapping = mpl::range_c<int,0,mpl::size<ColorSpace>::value> >
struct layout {
    typedef ColorSpace      color_space_t;
    typedef ChannelMapping  channel_mapping_t;
};
Here is how to create layouts for the RGBA color space:

typedef layout<rgba_t> rgba_layout_t; // default ordering is 0,1,2,3...
typedef layout<rgba_t, mpl::vector4_c<int,2,1,0,3> > bgra_layout_t;
typedef layout<rgba_t, mpl::vector4_c<int,1,2,3,0> > argb_layout_t;
typedef layout<rgba_t, mpl::vector4_c<int,3,2,1,0> > abgr_layout_t;

-->

## <a name="section_05"> 5. Color SpaceとLayout
Color Spaceは、Pixelを構成するChannelに関して、それらの組み合わせと解釈を保持します。
Color Spaceは、そのColor Spaceがもつ全ての要素の型を包含したMPLランダムアクセスシークエンスです。
ふたつのColor Spaceが等しい(同じ色のセットを同じ順序でもつ)場合に限って、それらのColor Space間には互換性があると見なされます。

#### 関連するConcept:

- `ColorSpaceConcept<ColorSpace>`
- `ColorSpacesCompatibleConcept<ColorSpace1,ColorSpace2>`
- `ChannelMappingConcept<Mapping>`

#### Model:

現在のところ、GILは`gray_t`, `rgb_t`, `rgba_t`, `cmyk_t`といったColor Spaceを提供しています。
また、2〜5個までのChannelをもった無名のN-Channel Color Spaceである、`devicen_t<2>`, `devicen_t<3>`, `devicen_t<4>`, `devicen_t<5>`も提供しています。
Layoutについて言えば、GILはスタンダードなLayout以外に`bgr_layout_t`, `bgra_layout_t`, `abgr_layout_t`, `argb_layout_t`などのLayoutを提供しています。

ひとつの例として、GILがどのようにしてRGBA Color Spaceを定義しているか示します。

{% highlight C++ %}

struct red_t{};
struct green_t{};
struct blue_t{};
struct alpha_t{};
typedef mpl::vector4<red_t,green_t,blue_t,alpha_t> rgba_t;

{% endhighlight %}

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

RGBA Color SpaceのLayoutの作り方は、次の通りです。

{% highlight C++ %}

typedef layout<rgba_t> rgba_layout_t; // default ordering is 0,1,2,3...
typedef layout<rgba_t, mpl::vector4_c<int,2,1,0,3> > bgra_layout_t;
typedef layout<rgba_t, mpl::vector4_c<int,1,2,3,0> > argb_layout_t;
typedef layout<rgba_t, mpl::vector4_c<int,3,2,1,0> > abgr_layout_t;

{% endhighlight %}
