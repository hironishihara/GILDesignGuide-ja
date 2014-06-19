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
A channel indicates the intensity of a color component (for example, the red channel in an RGB pixel).
Typical channel operations are getting, comparing and setting the channel values.
Channels have associated minimum and maximum value.
GIL channels model the following concept:

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

-->

## 4. Channel
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
Two channel types are compatible if they have the same value type:

concept ChannelsCompatibleConcept<ChannelConcept T1, ChannelConcept T2> {
    where SameType<T1::value_type, T2::value_type>;
};
A channel may be convertible to another channel:

template <ChannelConcept Src, ChannelValueConcept Dst>
concept ChannelConvertibleConcept {
    Dst channel_convert(Src);
};
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

ふたつのChannelが同じ`value_type`をもつ場合、そのふたつのChannelには互換性があります。

{% highlight C++ %}

concept ChannelsCompatibleConcept<ChannelConcept T1, ChannelConcept T2> {
    where SameType<T1::value_type, T2::value_type>;
};

{% endhighlight %}

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
Its channels correspond to bit ranges. To support such channels, we need to create a custom proxy class corresponding to a reference to a subbyte channel.
Such a proxy reference class models only ChannelConcept, because, similar to native C++ references, it may not have a default constructor.

Note also that algorithms may impose additional requirements on channels, such as support for arithmentic operations.

Related Concepts:

ChannelConcept<T>
ChannelValueConcept<T>
MutableChannelConcept<T>
ChannelsCompatibleConcept<T1,T2>
ChannelConvertibleConcept<SrcChannel,DstChannel>
Models:

All built-in integral and floating point types are valid channels. GIL provides standard typedefs for some integral channels:

typedef boost::uint8_t  bits8;
typedef boost::uint16_t bits16;
typedef boost::uint32_t bits32;
typedef boost::int8_t   bits8s;
typedef boost::int16_t  bits16s;
typedef boost::int32_t  bits32s;
The minimum and maximum values of a channel modeled by a built-in type correspond to the minimum and maximum physical range of the built-in type, as specified by its std::numeric_limits.
Sometimes the physical range is not appropriate. GIL provides scoped_channel_value, a model for a channel adapter that allows for specifying a custom range.
We use it to define a [0..1] floating point channel type as follows:

struct float_zero { static float apply() { return 0.0f; } };
struct float_one  { static float apply() { return 1.0f; } };
typedef scoped_channel_value<float,float_zero,float_one> bits32f;
GIL also provides models for channels corresponding to ranges of bits:

// Value of a channel defined over NumBits bits. Models ChannelValueConcept
template <int NumBits> class packed_channel_value;

// Reference to a channel defined over NumBits bits. Models ChannelConcept
template <int FirstBit,
          int NumBits,       // Defines the sequence of bits in the data value that contain the channel
          bool Mutable>      // true if the reference is mutable
class packed_channel_reference;

// Reference to a channel defined over NumBits bits. Its FirstBit is a run-time parameter. Models ChannelConcept
template <int NumBits,       // Defines the sequence of bits in the data value that contain the channel
          bool Mutable>      // true if the reference is mutable
class packed_dynamic_channel_reference;
Note that there are two models of a reference proxy which differ based on whether the offset of the channel range is specified as a template or a run-time parameter.
The first model is faster and more compact while the second model is more flexible. For example, the second model allows us to construct an iterator over bitrange channels.

Algorithms:
-->

`ChannelConcept`と`MutableChannelConcept`が、デフォルトコンストラクタを要求していないことに注意してください。
デフォルトコンストラクタをサポートする(その結果、正則型である)Channelは、`ChannelValueConcept`に基づいたModelです。
このような区別を設けた動機を理解するために、"565"のビットパターンをもつ16bit RGB Pixelを考えます。
各Channelは、それぞれのビットレンジに対応しています。
このようなChannelをサポートするためには、バイト境界をまたがるChannel参照にも対応する特別なProxyクラスをつくる必要があります。
このときProxy参照クラスは`ChannelConcept`だけに従って実装されます。
なぜなら、このようなChannelは、C++における参照のように、デフォルトコンストラクタをもたない可能性があるからです。
また、アルゴリズムが、算術演算子のサポートなど、追加の要件を課すかもしれないことにも注意が必要です。

#### 関連するConcept:

- `ChannelConcept<T>`
- `ChannelValueConcept<T>`
- `MutableChannelConcept<T>`
- `ChannelsCompatibleConcept<T1,T2>`
- `ChannelConvertibleConcept<SrcChannel,DstChannel>`

#### Model:

組み込みの整数型と浮動小数点型は、全て、有効なChannelです。
GILは、いくつかの整数型について、標準のtypedefを提供しています。

{% highlight C++ %}

typedef boost::uint8_t  bits8;
typedef boost::uint16_t bits16;
typedef boost::uint32_t bits32;
typedef boost::int8_t   bits8s;
typedef boost::int16_t  bits16s;
typedef boost::int32_t  bits32s;

{% endhighlight %}

組み込み型を用いたChannelの最小値と最大値は、その型の`std::numeric_limits`で定められている、組み込み型のフィジカルレンジに由来する最小値と最大値に対応しています。
しかし、状況によってはフィジカルレンジが適切でない場合もあります。
GILは、特別なレンジを定めるためのChannelアダプタのModelである、`scoped_channel_value`を提供します。
私たちは、[0..1]の浮動小数点を定義するために、`scoped_channel_value`を次のように用います。

{% highlight C++ %}

struct float_zero { static float apply() { return 0.0f; } };
struct float_one  { static float apply() { return 1.0f; } };
typedef scoped_channel_value<float,float_zero,float_one> bits32f;

{% endhighlight %}

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

各Channelレンジまでのオフセットについて、テンプレートで指定する参照Proxyと実行時のパラメータで指定する参照Proxyの異なる2種類の参照Proxy Modelがあることに注意してください。
前者は軽快かつコンパクトなModelであり、後者はより適応性のあるModelです。
例を挙げると、後者のModelはビット単位のレンジをもつChannel上で動作するIteratorを構築することができます。

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

代入、比較、コピーコンストラクタは、互換性をもつChannel間にだけ定義されます。

{% highlight C++ %}

packed_channel_value<5> channel_6bit = channel1;
channel_6bit = channel3;

//channel_6bit = channel2; // compile error: Assignment between incompatible channels.

{% endhighlight %}

GILによって提供される全てのChannel Modelが互いに変換可能です。

{% highlight C++ %}

channel1 = channel_traits<channel16_0_5_reference_t>::max_value();
assert(channel1 == 31);

bits16 chan16 = channel_convert<bits16>(channel1);
assert(chan16 == 65535);

{% endhighlight %}

Channel変換は、不可逆な操作です。
GILのChannel変換は、変換元Channelのレンジと変換先Channelのレンジとの線形変換です。
最小値と最小値、最大値と最大値がぴったりと対応します。
(ひとつ例を挙げます。GILは`uint8_t`から`uint16_t`への変換に際してビットシフトは行いません。というのも、ビットシフトでは最大値がぴったり一致しない可能性があるからです。
そのかわりに、GILは変換元Channelの値と257の積を求めます。)
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
