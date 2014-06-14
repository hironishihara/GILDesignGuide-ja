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


## 8. Pixel Iterator

### 基本となるIterator
Pixel Iteratorは、`PixelValueConcept`に基づいたModelである`value_type`のランダム走査Iteratorです。
Pixel Iteratorは、mutableであるか否か(すなわち、指し示すPixelが変更可能か否か)を判定するメタ関数、immutable (read-only)なIteratorを取得するメタ関数、素のIteratorかAdaptorをまとった他種のIteratorなのかを判定するメタ関数を提供します。

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

##### 関連するConcept:


- `PixelIteratorConcept<Iterator>`
- `MutablePixelIteratorConcept<Iterator>`

##### Model:

Pixelのビルトインポインタ`pixel<ChannelValue,Layout>*`は、インタリーブ形式のホモジーニアスPixelを対象とするPixel IteratorのためのGILのModelです。
同様に、`packed_pixel<PixelData,ChannelRefVec,Layout>*`は、インタリーブ形式バイト単位Pixelを対象とするIteratorのためのGILのModelです。

For planar homogeneous pixels, GIL provides the class planar_pixel_iterator, templated over a channel iterator and color space.
プラナー型ホモジーニアスPixelのために、GILはChannel IteratorとColor Spaceについてテンプレート化した`planar_pixel_iterator`クラスを提供します。

ここで、unsigned char型プラナー形式RGB PixelについてmutableなIteratorとread-onlyのIteratorがどのように定義されているのかを示します。

```cpp
template <typename ChannelPtr, typename ColorSpace> struct planar_pixel_iterator;

// GIL provided typedefs
typedef planar_pixel_iterator<const bits8*, rgb_t> rgb8c_planar_ptr_t;
typedef planar_pixel_iterator<      bits8*, rgb_t> rgb8_planar_ptr_t;
```

`planar_pixel_iterator`は`HomogeneousColorBaseConcept` (`homogeneous_color_base`のサブクラス)に基づいたModelであり、つまり、全てのcolor baseアルゴリズムを適用できます。
そのColor Baseの要素の型はChannel Iteratorです。
For example, GIL implements operator++ of planar iterators approximately like this:
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

`static_transform`はコンパイル時の再帰を用いるので、`rgb8_planar_ptr_t`インスタンスのインクリメントは3個のpointerのインクリメントに変換されます。
また、GILは、ビット単位Pixelを走査するPixel IteratorのModelとして、`bit_aligned_pixel_iterator`クラスを用います。
内部的には、各時点でのバイト位置とビットオフセットを記録しています。

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

##### 関連するConcept:

- `IteratorAdaptorConcept<Iterator>`
- `MutableIteratorAdaptorConcept<Iterator>`

##### Model:

GILは`IteratorAdaptorConcept`のModelをいくつか提供しています。

- `memory_based_step_iterator<Iterator>`: Base Iteratorの基本的なステップを変更するIteratorアダプタ。(ステップIteratorを参照)
- `dereference_iterator_adaptor<Iterator,Fn>`: ひとつの引数をとる関数`Fn`に間接参照した値を適用するIteratorアダプタ。
例を挙げると、on-the-flyな色変換に用いられます。
これは、実際とは異なるColor SpaceやChannel深度をもっているかのように見せかける浅いImage "View"を構築するときに用いられます。
詳細は、"ほかのImage ViewからImage Viewを作成する"をみてください。
ひとつの引数をとる関数`Fn`は`PixelDereferenceAdaptorConcept`(次を見てください)に基づいたModelでなければなりません。

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

##### Model:

GILは`PixelDereferenceAdaptorConcept`のModelをいくつか提供します。

- `color_convert_deref_fn`: 色変換を行う関数オブジェクト。
- `detail::nth_channel_deref_fn`: 与えられたPixelのN番目Channelに対応するグレイスケールPixelを返す関数オブジェクト。
- `deref_compose`: 2つの`PixelDereferenceAdaptorConcept`のModelを合成する関数オブジェクト。`PixelDereferenceAdaptorConcept`で要求される追加の`typedef`を取る必要がある点を除いて、`std::unary_compose`と似ています。

GILは、間接参照した値に色変換を実行するImage ViewやPixelのN番目のChannelを返すImage Viewを実装するために、Pixel間接参照アダプタを使用します。
これらのPixel間接参照アダプタは、間接参照した値に任意関数を実行するVirtual Image View (例えば、マンデルブロ集合を表すViewなど)を実装する際に使用されます。
`dereference_iterator_adaptor<Iterator,Fn>`は、`Iterator`で間接参照した値を引数にして与えられた間接参照Iteratorアダプタ`Fn`を実行する、Pixel Iteratorである`Iterator`を包むIteratorラッパーです。

## ステップIterator
基本的なPixel Iteratorによって提供される1ステップ以上のまとまったステップ数でPixelの走査を行いたい場合があります。
例えば、次のような場合です。
- RGBインタリーブ画像の赤Channelだけをみる単独Channel View
- 左右反転画像 (step = -fundamental_step)
- N個間隔でPixelを取ってきたView (step = N*fundamental_step)
- 垂直方向への移動 (step = number bytes per row)。
- 上記の組み合わせ (stepは各stepの積)

Step iterators are forward traversal iterators that allow changing the step between adjacent values:
ステップIteratorは、隣り合う値に対してのステップ数の変更を許可する前方移動Iteratorです。

```cpp
concept StepIteratorConcept<boost_concepts::ForwardTraversalConcept Iterator> {
    template <Integral D> void Iterator::set_step(D step);
};

concept MutableStepIteratorConcept<boost_concepts::Mutable_ForwardIteratorConcept Iterator> : StepIteratorConcept<Iterator> {};
```

いまのところ、GILは`PixelValueConcept`に基づいて実装された`value_type`をもつステップIteratorを提供します。
そのとき、ステップにはメモリ上での単位(バイト単位かビット単位か)が指定されています。
例えば、Pixelの列に沿って走査するIteratorを実装する場合などに必要だからです。
各行のサイズは、Pixelのサイズで割り切れるとは限りません。
例えば、各行にワード単位アラインメントが施されているかもしれません。

数バイト毎または数ビット毎に進む場合、その基本となるIteratorは`MemoryBasedIteratorConcept`に基づいて実装されていなければなりません。
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

他のIteratorからステップIteratorを構築できれば便利です。
より一般的に言えば、ある型を与えたとき、それと等価で水平方向のステップ数を動的に指定可能な型を構築したいのです。

```cpp
concept HasDynamicXStepTypeConcept<typename T> {
    typename dynamic_x_step_type<T>;
        where Metafunction<dynamic_x_step_type<T> >;
};
```

GILが提供する全てのPixel Iterator、Locator、Image ViewのModelは、`HasDynamicXStepConcept`をサポートしています。

##### 関連するConcept:

- `StepIteratorConcept<Iterator>`
- `MutableStepIteratorConcept<Iterator>`
- `MemoryBasedIteratorConcept<Iterator>`
- `HasDynamicXStepTypeConcept<T>`

##### Model:

現在GILが提供している全ての基本的なメモリベースIteratorは`MemoryBasedIteratorConcept`に基づいたModelです。
GILは`PixelIteratorConcept`、`StepIteratorConcept`、`MemoryBasedIteratorConcept`に基づいたModelである`memory_based_step_iterator`クラスを提供しています。
これは、テンプレートのパラメータとして基本となるIterator(`PixelIteratorConcept`と`MemoryBasedIteratorConcept`に基づいたModelでなければなりません)をとり、ステップを動的に変更することを許可します。
GIL's implementation contains the base iterator and a ptrdiff_t denoting the number of memory units (bytes or bits) to skip for a unit step.
GILの実装では、基本となるIteratorと、1ステップで進む数をメモリ単位に基づいて示す`ptrdiff_t`を含みます。
`ptrdiff_t`には負の数を使うこともできます。
GILは、基本となるIteratorとステップを指定してステップIteratorを作成する関数を提供しています。

```cpp
template <typename I>  // Models MemoryBasedIteratorConcept, HasDynamicXStepTypeConcept
typename dynamic_x_step_type<I>::type make_step_iterator(const I& it, std::ptrdiff_t step);
```

GILは、`position_iterator`という、仮想的なPixel配列に対するIteratorのModelも提供しています。
これは、Pixelの位置情報を保持し、その位置にあるPixelの値を間接参照で取得する関数オブジェクトを実行するステップIteratorです。
これは、`PixelIteratorConcept`と`StepIteratorConcept`に基づいたModelですが、`MemoryBasedIteratorConcept`に基づいたModelではありません。

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

2次元Locatorは、水平方向だけではなく垂直方向にも、動的なステップをもつことができます。
これはつまり、Y軸における`HasDynamicXStepTypeConcept`です。

```cpp
concept HasDynamicYStepTypeConcept<typename T> {
    typename dynamic_y_step_type<T>;
        where Metafunction<dynamic_y_step_type<T> >;
};
```

GILが提供する全てのLocatorとImage Viewは`HasDynamicYStepTypeConcept`に基づいたModelです。

Sometimes it is necessary to swap the meaning of X and Y for a given locator or image view type (for example, GIL provides a function to transpose an image view).
与えられたLocatorやImage Viewについて、X軸とY軸の入れ替えが必要になることがあります(例を挙げると、GILはImage Viewの転置変換を行う関数を提供しています)。
上記のようなLocatorやViewは転置変換可能でなければなりません。

```cpp
concept HasTransposedTypeConcept<typename T> {
    typename transposed_type<T>;
        where Metafunction<transposed_type<T> >;
};
```

GILが提供する全てのLocatorとViewは、`HasTransposedTypeConcept`に基づいたModelです。

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

##### 関連するConcept:

- `HasDynamicYStepTypeConcept<T>`
- `HasTransposedTypeConcept<T>`
- `RandomAccessNDLocatorConcept<Locator>`
- `MutableRandomAccessNDLocatorConcept<Locator>`
- `RandomAccess2DLocatorConcept<Locator>`
- `MutableRandomAccess2DLocatorConcept<Locator>`
- `PixelLocatorConcept<Locator>`
- `MutablePixelLocatorConcept<Locator>`


##### Model:

GILは2種類の`PixelLocatorConcept`のModelを提供します。
メモリベースLocatorである`memory_based_2d_locator`と、Virtual Locatorである`virtual_2d_locator`です。

`memory_based_2d_locator`は、メモリ上にPixelがあるプラナー画像もしくはインタリーブ画像におけるLocatorです。
このLocatorは、テンプレートのパラメータとして`StepIteratorConcept`のModelを取ります。
(`MutableStepIteratorConcept` Modelの場合を例にすると、これは`MutablePixelLocatorConcept`に基づいたModelです。)

```cpp
template <typename StepIterator>  // Models StepIteratorConcept, MemoryBasedIteratorConcept
class memory_based_2d_locator;
```

ステップIteratorのステップは、各行において、メモリ単位(数バイトまたは数ビット)の倍数でなければなりません(すなわち、ステップIteratorはメモリ単位で移動しなければなりません)。
`memory_based_2d_locator`クラスはステップIteratorのラッパであり、ステップIteratorが垂直方向のナビゲートに使われる一方で、そのステップIteratorの基本となるIteratorが水平方向のナビゲートに使われます。

基本となるIteratorとステップIteratorの合成によって、Pixelについての複雑なメモリ配置を記述したLocatorの作成が可能になります。
始めに、水平方向すなわちに同じ行にあるPixelに対する走査に用いるIteratorを選択します。

基本となるIteratorやステップIteratorには、4つの選択肢が与えられています。

- `pixel<T,C>*` (インタリーブ画像用)
- `planar_pixel_iterator<T*,C>` (プラナー画像用)
- `memory_based_step_iterator<pixel<T,C>*>` (標準以外のステップをもつインタリーブ画像用)
- `memory_based_step_iterator<planar_pixel_iterator<T*,C> >` (標準以外のステップをもつプラナー画像用)

もちろん、独自の水平方向Iteratorを提供することもできます。
この先に記述されている一例として、間接参照された際に色変換を実行するIteratorアダプタがあります。

水平方向Iteratorである`XIterator`が与えられるとき、メモリ単位(数バイトもしくは数ビット)の倍数と等しいステップをもつ`memory_based_step_iterator<XIterator>`として、ある列に沿って移動するIteratorである垂直方向Iteratorを選ぶことができます。
ここでも、独自の垂直方向Iteratorを提供することは自由です。

ここでは、ダイアグラムが示すように、2次元Pixel Locatorを得るために`memory_based_2d_locator<memory_based_step_iterator<XIterator> >`を作成します。

![2次元Pixel Locator](http://hironishihara.github.com/GILDesignGuide-ja/src/img/step_iterator.gif "2次元Pixel Locator")

`virtual_2d_locator`は、間接参照したPixelに対して実行される関数オブジェクトと共に作成されるLocatorです。
これは、与えられたXY座標にあるPixelの値を返します。
Virtual Locatorは、任意のユーザ定義関数に基づいたVirtual Image Viewを実装するときに使うことができます。
Virtual Locatorを用いてマンデルブロ集合のViewを作成する例については、GILチュートリアルを参照ください。

Virtual LocatorとメモリベースLocatorは、`PixelLocatorConcept`から要求されるほとんどのインタフェースを提供する基本クラスである`pixel_2d_locator_base`のサブクラスです。
この基本クラスは、他の`PixelLocatorConcept`のModelを提供する必要が生じた際に利用できるかもしれません。

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

標準的なGIL Locatorは、高速で軽量なオブジェクトです。
例を挙げると、シンプルなインタリーブ画像のためのLocatorは、Pixelの位置を示す生ポインタとバイト単位での行サイズを値にもつ整数型との合計8バイトで構成されます。
`++loc.x()`は、生ポインタのインクリメントと等価(プラナー画像の場合、N個のポインタのインクリメントと等価)です。
2次元オフセットの計算は、積と和を必要とするために比較的低速です。
例えば、フィルタ処理では画像中の各Pixelにおいて同じ位置関係にある隣接Pixelへのアクセスが必要ですが、そのような場合に、相対的な位置は`cache_location`を利用して差分を表す整数のなかにキャッシュできます。
上記の例で言うと、インタリーブ画像の`loc[above]`は生配列のインデクス演算子と等価です。

### 2次元画像上でのIterator
ときには、画像中の全Pixelに対して位置に依存しない一律の処理を実行したいといった場合も考えられます。
このようなとき、一律に扱うPixel全てを一次元配列のように扱うことができれば便利です。
GILの`itarator_from_2d`は、画像中の全Pixelを左から右、上から下というメモリフレンドリな順序で走査するランダムアクセスIteratorです。
これは、ひとつのLocatorと画像の幅と現在のX座標をもっています。
これは"キャリッジリターン"のタイミングを決定するために十分な情報です。

##### Synopsis:

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

`iterator_from_2d`を用いた画像中の全Pixelへの走査は、水平方向Iteratorを用いた各行での走査の全行分の合算よりも低速です。
これは、1ステップ毎にIteratorによるループの終了判定と`iterator_from_2d::operator++`による行の終端判定との2個の比較が行われることが原因です。
Pixelのコピーのような高速な処理では、この2個目の判定は約15%の遅延の原因となります(Intelプラットホーム上にて、インタリーブ画像で計測)。
GILは`std::copy`や`std::fill`といったいくつかのSTLアルゴリズムをオーバーライドしており、実行時`iterator_from_2d`が渡された場合には各行に対して基本となる水平Iteratorを用います。
また、画像にパディングがない(例：`iterator_from_2d::is_1d_traversable()`が`true`を返す)ときには、シンプルに水平Iteratorを直接用いる走査を行います。
