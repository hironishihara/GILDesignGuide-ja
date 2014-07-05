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
8. Pixel Iterator
-->

## 8. Pixel Iterator

<!--
Fundamental Iterator

Pixel iterators are random traversal iterators whose value_type models PixelValueConcept.
Pixel iterators provide metafunctions to determine whether they are mutable (i.e. whether they allow for modifying the pixel they refer to), to get the immutable (read-only) type of the iterator, and to determine whether they are plain iterators or adaptors over another pixel iterator:
-->

### 基本となるIterator

Pixel Iteratorは、`PixelValueConcept`に基づいたModelである`value_type`のランダム走査Iteratorです。
Pixel Iteratorは、mutableであるか否か(すなわち、指し示すPixelが変更可能か否か)を判定するメタ関数、immutable (read-only)なIteratorを取得するメタ関数、素のIteratorかアダプタをまとった他のIteratorなのかを判定するメタ関数を提供します。

```cpp
concept PixelIteratorConcept<RandomAccessTraversalIteratorConcept Iterator> : PixelBasedConcept<Iterator> {
    where PixelValueConcept<value_type>;
    typename const_iterator_type<It>::type;
        where PixelIteratorConcept<const_iterator_type<It>::type>;
    static const bool  iterator_is_mutable<It>::type::value;
    static const bool  is_iterator_adaptor<It>::type::value;   // is it an iterator adaptor
};

template <typename Iterator>
concept MutablePixelIteratorConcept : PixelIteratorConcept<Iterator>, MutableRandomAccessIteratorConcept<Iterator> {};
```

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

```cpp
template <typename ChannelPtr, typename ColorSpace> struct planar_pixel_iterator;

// GIL provided typedefs
typedef planar_pixel_iterator<const bits8*, rgb_t> rgb8c_planar_ptr_t;
typedef planar_pixel_iterator<      bits8*, rgb_t> rgb8_planar_ptr_t;
```

<!--
planar_pixel_iterator also models HomogeneousColorBaseConcept (it subclasses from homogeneous_color_base) and, as a result, all color base algorithms apply to it.
The element type of its color base is a channel iterator.
For example, GIL implements operator++ of planar iterators approximately like this:
-->

`planar_pixel_iterator`は`HomogeneousColorBaseConcept` (`homogeneous_color_base`のサブクラス)に基づいたModelであり、つまり、全てのColor Baseアルゴリズムを適用できます。
そのColor Baseの要素の型はChannel Iteratorです。
例を挙げると、GILではプラナー形式Iteratorの`operator++`をおおよそ次のように実装しています。

```cpp
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
```

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

### Iteratorアダプタ
Iteratorアダプタは他のIteratorをラップしたIteratorです。
その`is_iterator_adaptor`というメタ関数は`true`でなければなりません。
また、Base Iteratorを返すメンバ関数、型を取得するメタ関数、他のBase Iteratorに再結合するメタ関数を提供する必要があります。

```cpp
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
```

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

### Pixel間接参照アダプタ

Pixel間接参照アダプタは、Pixel Iteratorから間接参照した値を受け取る、ひとつの引数をもつ関数です。
この引数の型はどんなものでも構いません(よくあるのは`PixelConcept`です)。また、戻り値の型は`PixelConcept`に変換可能でなければなりません。

```cpp
template <boost::UnaryFunctionConcept D>
concept PixelDereferenceAdaptorConcept : DefaultConstructibleConcept<D>, CopyConstructibleConcept<D>, AssignableConcept<D>  {
    typename const_t;         where PixelDereferenceAdaptorConcept<const_t>;
    typename value_type;      where PixelValueConcept<value_type>;
    typename reference;       where PixelConcept<remove_reference<reference>::type>;  // may be mutable
    typename const_reference;   // must not be mutable
    static const bool D::is_mutable;

    where Convertible<value_type, result_type>;
};
```

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

### ステップIterator
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

```cpp
concept StepIteratorConcept<boost_concepts::ForwardTraversalConcept Iterator> {
    template <Integral D> void Iterator::set_step(D step);
};

concept MutableStepIteratorConcept<boost_concepts::Mutable_ForwardIteratorConcept Iterator> : StepIteratorConcept<Iterator> {};
```

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

```cpp
concept MemoryBasedIteratorConcept<boost_concepts::RandomAccessTraversalConcept Iterator> {
    typename byte_to_memunit<Iterator>; where metafunction<byte_to_memunit<Iterator> >;
    std::ptrdiff_t      memunit_step(const Iterator&);
    std::ptrdiff_t      memunit_distance(const Iterator& , const Iterator&);
    void                memunit_advance(Iterator&, std::ptrdiff_t diff);
    Iterator            memunit_advanced(const Iterator& p, std::ptrdiff_t diff) { Iterator tmp; memunit_advance(tmp,diff); return tmp; }
    Iterator::reference memunit_advanced_ref(const Iterator& p, std::ptrdiff_t diff) { return *memunit_advanced(p,diff); }
};
```

<!--
It is useful to be able to construct a step iterator over another iterator.
More generally, given a type, we want to be able to construct an equivalent type that allows for dynamically specified horizontal step:
-->

他のIteratorからステップIteratorを構築できれば便利です。
より一般的に言えば、ある型を与えたとき、それと等価で水平方向のステップ数を動的に指定可能な型を構築したいのです。

```cpp
concept HasDynamicXStepTypeConcept<typename T> {
    typename dynamic_x_step_type<T>;
        where Metafunction<dynamic_x_step_type<T> >;
};
```

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

```cpp
template <typename I>  // Models MemoryBasedIteratorConcept, HasDynamicXStepTypeConcept
typename dynamic_x_step_type<I>::type make_step_iterator(const I& it, std::ptrdiff_t step);
```

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

### Pixel Locator

Locatorは2次元もしくはそれ以上の次元でのナビゲーションを可能にします。
Locatorは、本来であればN次元Iteratorと呼ぶべきですが、Iteratorが満たすべき要件を完全には満たしていないため、このように違う名前を使っています。
例を挙げると、Locatorは、どの軸にそって移動するべきか明確でないために、インクリメントやデクリメントを行う演算子を提供しません。
N次元Locatorは次のConceptに基づいたModelです。

```cpp
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
```

<!--
Two-dimensional locators have additional requirements:
-->

2次元Locatorには追加の要件があります。

```cpp
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
```

<!--
2D locators can have a dynamic step not just horizontally, but also vertically.
This gives rise to the Y equivalent of HasDynamicXStepTypeConcept:
-->

2次元Locatorは、水平方向だけではなく垂直方向にも、動的なステップをもつことができます。
これはつまり、Y軸における`HasDynamicXStepTypeConcept`です。

```cpp
concept HasDynamicYStepTypeConcept<typename T> {
    typename dynamic_y_step_type<T>;
        where Metafunction<dynamic_y_step_type<T> >;
};
```

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

```cpp
concept HasTransposedTypeConcept<typename T> {
    typename transposed_type<T>;
        where Metafunction<transposed_type<T> >;
};
```

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

```cpp
concept PixelLocatorConcept<RandomAccess2DLocatorConcept Loc> {
    where PixelValueConcept<value_type>;
    where PixelIteratorConcept<x_iterator>;
    where PixelIteratorConcept<y_iterator>;
    where x_coord_t == y_coord_t;

    typename coord_t = x_coord_t;
};

concept MutablePixelLocatorConcept<PixelLocatorConcept Loc> : MutableRandomAccess2DLocatorConcept<Loc> {};
```

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

```cpp
template <typename StepIterator>  // Models StepIteratorConcept, MemoryBasedIteratorConcept
class memory_based_2d_locator;
```

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

```cpp
loc=img.xy_at(10,10);            // start at pixel (x=10,y=10)
above=loc.cache_location(0,-1);  // remember relative locations of neighbors above and below
below=loc.cache_location(0, 1);
++loc.x();                       // move to (11,10)
loc.y()+=15;                     // move to (11,25)
loc-=point2<std::ptrdiff_t>(1,1);// move to (10,24)
*loc=(loc(0,-1)+loc(0,1))/2;     // set pixel (10,24) to the average of (10,23) and (10,25) (grayscale pixels only)
*loc=(loc[above]+loc[below])/2;  // the same, but faster using cached relative neighbor locations
```

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

### 2次元画像上でのIterator

ときには、画像中の全Pixelに対して位置に依存しない一律の処理を実行したいといった場合も考えられます。
このようなとき、一律に扱うPixel全てを一次元配列のように扱うことができれば便利です。
GILの`itarator_from_2d`は、画像中の全Pixelを左から右、上から下というメモリフレンドリな順序で走査するランダムアクセスIteratorです。
これは、ひとつのLocatorと画像の幅と現在のX座標をもっています。
これは"キャリッジリターン"のタイミングを決定するために十分な情報です。

<!--
Synopsis:
-->

#### Synopsis:

```cpp
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
```

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
