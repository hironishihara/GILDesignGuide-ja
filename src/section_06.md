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


## 6. Color Base
Color Baseは色要素のコンテナです。
Color BaseはPixelの実装のなかで、すなわち色要素がChannelの値になっている場合に、よく用いられます。
しかし、Color BaseのConceptは他の用途に用いられる場合もあります。
例えば、プラナー画像のPixelはメモリ上で不連続なChannelをもっています。
その参照は、各Channelの参照を要素とする、Color Baseを用いたProxyクラスです。
そのIteratorは、各ChannelのIteratorを要素とするColor Baseを使用します。
Color BaseのModelは、次に示すConceptを満たさなければなりません。

```cpp
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
```

Color Baseは、Layoutを必ず1個もっていなければなりません (そのLayoutはColor SpaceとChannelの順序から構成されています)。
Color Baseの各要素へのインデクシングには2種類の方法があります。
メモリ上での各要素の配置に対応したフィジカルインデクスと、Color Spaceが示す順序に対応したセマンティックインデクスです。
例えば、RGB Color Spaceでは、各要素が{`red_t`, `green_t`, `blue_t`}の順に並んでいます。
Color BaseがBGR Layoutをもつなら、フィジカルな順序でみると最初の要素は青(blue)ですが、一方、セマンティックな順序でみると最初の要素は赤(red)となります。
`ColorBaseConcept`のModelは、フィジカルな順序に基づいて各要素にアクセスする関数`at_c<K>(ColorBase)`を提供することが求められます。
GILは、あらゆる`ColorBaseConcept`のModel上で動作し、セマンティックに要素を返す関数`semantic_at_c<K>(ColorBase)` (あとで述べます)を提供します。
ふたつのColor Baseは、同じColor Spaceをもち、セマンティックに対をなす各要素が互いに変換可能であるとき、互換性をもちます。

##### Model:

GILは、ホモジーニアスなColor Base(各要素が全て同じ型のColor Base)のためのModelを提供します。

```cpp
namespace detail {
    template <typename Element, typename Layout, int K> struct homogeneous_color_base;
}
```

このModelは、GILのPixel、Planar Pixelの参照、Planar PixelのIteratorの実装に使われています。
もうひとつの`ColorBaseConcept`のModelは`packed_pixel`であり、ビット単位のレンジをもつChannelに基づいたPixelです。
詳しくは、第7章を参照ください。

##### Algorithm:

GILは、次に示す、Color Base上で動作する関数とメタ関数を提供します。

```cpp
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
```

GILは、Color Baseで動作する、次のようなアルゴリズムも提供しています。
これらのアルゴリズムが各要素をセマンティックなペアで扱うことに注意してください。

```cpp
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
```

これらのアルゴリズムは、レンジのかわりにColor Baseを使って各要素のオペレーションを行うという点を除いて、STLアルゴリズムに対応するようにデザインされています。
さらに、コンパイル時の再帰を用いる実装になっています (そのため、prefixに`static_`がついてます)。
そして、これらのアルゴリズムは、メモリ上のフィジカルな順序ではなく、セマンテックな順序に基づいて要素のペアを作ります。
例えば、`static_equal`の実装を挙げると次のようになります。

```cpp
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
```

このアルゴリズムは、例えば、ふたつのPixel間の`operator==`を実行するときに用います。
セマンティックなアクセサを使うことで、RGB PixelとBGR Pixelを適切に比較できます。
また、上記の2個以上のColor Baseを引数にとる全てのアルゴリズムは、全てのColorBaseが同じColor Spaceをもつよう要求することに注意しましょう。
