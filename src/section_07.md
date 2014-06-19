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

## 7. Pixel
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
ほとんどのPixelの2項演算(コピーコンストラクタ, 代入、等号など)は互換性をもつPixel間でだけ定義されています。

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

あるPixelが自身の色をもう一方のPixelの形式に近似できるとき、もう一方のPixelと変換可能です。
変換は、数学的に陽であり、非対称であり、ほとんどの場合は(ChannelとColor Space両方の近似が原因で)不可逆変換です。

交換可能性は次のConceptに基づいた実装を要求します。

{% highlight C++ %}

template <PixelConcept SrcPixel, MutablePixelConcept DstPixel>
concept PixelConvertibleConcept {
    void color_convert(const SrcPixel&, DstPixel&);
};

{% endhighlight %}

`PixelConcept`と`PixelValueConcept`の違いは、ChannelとColor Baseの違いと似ています。
Pixel参照Proxyは両方のConceptに基づいたModelですが、Pixelは後者のConceptだけに基づいたModelです。

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

`struct planar_pixel_reference`は、Layoutでテンプレート化されている`struct pixel`とは異なり、Color Spaceでテンプレート化されていることに注意してください。
これらは常に標準化されたChannel順を用います。
各要素は各Channelから参照されるため、要素の順序に関する情報は不要なのです。

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

また、GILはColor SpaceとChannel型が異なるPixel間の変換を行う`color_convert`アルゴリズムを提供します。

{% highlight C++ %}

rgb8_pixel_t red_in_rgb8(255,0,0);
cmyk16_pixel_t red_in_cmyk16;
color_convert(red_in_rgb8,red_in_cmyk16);

{% endhighlight %}
