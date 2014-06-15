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

## 9. Image View
Image Viewは、STLのRangeというConceptを複数次元に一般化したものです。
Renge (と、そのIterator)と同様、Image Viewは浅く、自身でデータをもたず、自身のconst性をデータにまで伝えません。
例を挙げると、constantなImage Viewはサイズを変更できませんが、Pixelの値を変更することは可能かもしれません。
Pixelの値を変更しない処理には、値がconstantなImage View (non-mutableなImage Viewとも呼ばれます)を使用します。
最も一般的なN次元Viewは、次のConceptを満たします。

```cpp
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
```

2次元のImage Viewは、次に示す追加の要件をもっています。

```cpp
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
```

GILが通常用いるImage Viewは、`PixelValueConcept`に基づいたModelであるValue型で動作し、いくつかの追加の要件をもっています。

```cpp
concept ImageViewConcept<RandomAccess2DImageViewConcept View> {
    where PixelValueConcept<value_type>;
    where PixelIteratorConcept<x_iterator>;
    where PixelIteratorConcept<y_iterator>;
    where x_coord_t == y_coord_t;

    typename coord_t = x_coord_t;

    std::size_t View::num_channels() const;
};

concept MutableImageViewConcept<ImageViewConcept View> : MutableRandomAccess2DImageViewConcept<View> {};
```

ふたつのImage Viewが、互換性のあるPixelをもち、同じ次元数であるとき、それらのImage Viewの間には互換性があります。

```cpp
concept ViewsCompatibleConcept<ImageViewConcept V1, ImageViewConcept V2> {
    where PixelsCompatibleConcept<V1::value_type, V2::value_type>;
    where V1::num_dimensions == V2::num_dimensions;
};
```

互換性のあるViewは、同じサイズ(すなわち、同じWidthとHeight)でなければなりません。
複数のViewを用いるアルゴリズムの多くが、それぞれのViewの間での互換性を要求します。

#### 関連するConcept:

- `RandomAccessNDImageViewConcept<View>`
- `MutableRandomAccessNDImageViewConcept<View>`
- `RandomAccess2DImageViewConcept<View>`
- `MutableRandomAccess2DImageViewConcept<View>`
- `ImageViewConcept<View>`
- `MutableImageViewConcept<View>`
- `ViewsCompatibleConcept<View1,View2>`

#### Model:

GILは`image_view`と呼ばれる`ImageViewConcept`のためのModelを提供します。
それは`PixelLocatorConcept`のModelをパラメータにしたテンプレートになっています。
(`MutablePixelLocatorConcept`のModelを用いてインスタンス化された場合には、それは`MutableImageViewConcept`に基づいたModelになります。)

#### Synopsis:

```cpp
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
```

Image Viewは軽量なオブジェクトです。
正規のインタリーブ形式Viewであれば、基本的に16バイトです。
その内訳は、Dimensionsに含まれるWidthとHeightを表す2つの整数と、Locatorに含まれる1行のバイト数を示す整数と、Pixel配列の先頭を指すポインタです。

#### Algorithm:

### メモリ上のPixel生データからのImage View作成

一般的なImage Viewは、サポートされているどのようなColor Space、Channel深度、Channel順、プラナー形式またはインタリーブ形式の生データからでも構成することができます。
インタリーブ形式Viewは、画像のDimensionsと1行あたりのバイト数と最初のPixelを指すポインタを指定した`interleaved_view`を使って構成されます。

```cpp
template <typename Iterator> // Models pixel iterator (like rgb8_ptr_t or rgb8c_ptr_t)
image_view<...> interleaved_view(ptrdiff_t width, ptrdiff_t height, Iterator pixels, ptrdiff_t rowsize)
```

プラナー形式Viewは、あらゆるColor Spaceのために定義されており、各Planeを個別に用意します。
ここに、RGB形式のプラナー形式Viewを示します。

```cpp
template <typename IC>  // Models channel iterator (like bits8* or const bits8*)
image_view<...> planar_rgb_view(ptrdiff_t width, ptrdiff_t height,
                                 IC r, IC g, IC b, ptrdiff_t rowsize);
```

戻り値のViewが値がconstantな(immutableな)Viewである場合、提供されるPixel/Channel Iteratorがconstant (read-only)になることに注意してください。

### 他のImage ViewからのImage View作成

画像データがどのように解釈されるのかを示したいくつかのポリシーを変更することで、あるImage Viewから他のImage Viewを構築することが可能です。
その結果は、元となる型から派生した型のViewになっている可能性もあります。
GILは、生成された型を取得するために、次に示すメタ関数を用います。

```cpp
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
```

GILは、次に示すView変換を提供します。

```cpp
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
```

これらView Factoryメソッドのほとんどの実装は、単純です。
例として、反転Viewがどのように実装されているのかを示します。
The flip upside-down view creates a view whose first pixel is the bottom left pixel of the original view and whose y-step is the negated step of the source.
上下反転Viewは、元のViewの最左下Pixelが先頭Pixelの、垂直方向ステップが元のステップと逆向きになったViewをつくります。

```cpp
template <typename View>
typename dynamic_y_step_type<View>::type flipped_up_down_view(const View& src) {
    gil_function_requires<ImageViewConcept<View> >();
    typedef typename dynamic_y_step_type<View>::type RView;
    return RView(src.dimensions(),typename RView::xy_locator(src.xy_at(0,src.height()-1),-1));
}
```

`gil_function_requires`関数の呼び出しは、テンプレートのパラメータが`ImageViewConcept`の有効なModelであることを(コンパイル時に)保証します。
これを使うことで、コンパイルエラーの追跡は容易になり、余計なコードの生成されず、実行時のパフォーマンスへの影響もありません。
私たちは`boost::concept_check`ライブラリを使用していますが、`BOOST_GIL_USE_CONCEPT_CHECK`がセットされているときにだけチェックを実行するように、`gil_function_requires`の中に`boost::concept_check`ライブラリをラップしています。
デフォルトでは`BOOST_GIL_USE_CONCEPT_CHECK`はセットされていません。なぜなら、コンセプトチェックを使用するとコンパイルタイムが大幅に長くなるのです。
このガイド内のサンプルコードでは、簡潔さのために`gil_function_requires`をスキップすることにします。
Image Viewは自由に構成することが出来ます("第12章 メタ関数とTypedef"を参照ください)。

```cpp
rgb16_image_t img(100,100);    // an RGB interleaved image

// grayscale view over the green (index 1) channel of img
gray16_step_view_t green=nth_channel_view(view(img),1);

// 50x50 view of the green channel of img, upside down and taking every other pixel in X and in Y
gray16_step_view_t ud_fud=flipped_up_down_view(subsampled_view(green,2,2));
```

前述のとおり、Image Viewは高速で、定量時間で動作する、浅いViewです。

上記のコードではひとつのPixelもコピーされていません。
すなわち、ViewはImageが作られたときに確保されたPixelデータを操作するのです。

### Image View上で動作するSTL-Styleアルゴリズム

Image Viewは`begin()`メソッドと`end()`メソッドを通じて1次元走査を提供しており、Image Viewを対象にSTLアルゴリズムを使うことを可能にしています。
しかしながら、多くの場合、XとYでネストされたループを使う方がより効果的です。
ここで紹介するアルゴリズムはSTLアルゴリズムと共通点がありますが、ネストされたループを抽象化し、入力として(Rangeではなく)Viewを受け取ります。

```cpp
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
```

複数のViewを取るアルゴリズムは、それらが同じDimensionsをもつことを要求します。
`for_each_pixel_position`と`transform_pixel_positions`は、Pixelの参照ではなく、Pixel Locatorを関数オブジェクトに渡します。
このことは、チュートリアルで示したように、隣接するPixelを使用するアルゴリズムの記述を可能にします。

これらのほとんどのアルゴリズムで、Image Viewが1次元走査可能であるか否かを調べています。
1次元走査可能なImage Viewは、各行の終端にギャップをもちません。
言い換えれば、1次元走査可能なImage Viewの`x_iterator`がある行の最後のPixelを通過したとき、その`x_iterator`はその次の行の最初のPixelへ移動しています。
Image Viewが1次元走査可能である場合、これらのアルゴリズムは一重のループを使用し、より効率的に動作します。
入力されたViewの中に1次元走査のできないViewが含まれていた場合、これらのアルゴリズムはYループのなかにネストされたXループへと後退します。

基本的に、これらのアルゴリズムはその処理を対応するSTLアルゴリズムに委託します。
例を挙げると、`copy_pixels`は、1次元走査可能なViewの場合は全Pixelを対象に、それ以外のViewの場合は各行を対象に、`std::copy`を呼びます。

さらに、STLアルゴリズムのオーバーロードが提供される場合もあります。
例えば、プラナー形式Iteratorに対する`std::copy`は、各Planeに対して`std::copy`を実行するようにオーバーライドされます。
ビット単位でコピー可能なPixel上で動作する`std::copy`は、結果的に、STLでは一般的に`memmove`を介して実装される、unsigned charで動作する`std::copy`となります。

まとめると、`copy_pixels`は、インタリーブ形式の1次元走査可能なViewに対しては1回の`memmove`呼び出しになり、プラナー形式の1次元走査可能なViewに対しては各Plane毎の`memmove`呼び出しになり、インタリーブ形式の1次元走査ができないViewに対しては各行毎の`memmove`呼び出しになる、といった感じです。

GILは、リサンプリングや畳み込みなど、いくつかの画像処理アルゴリズムのβバージョンの提供も行っています。
それらは、<http://opensource.adobe.com/gil/download.html> のnumerics extensionからダウンロードできます。
これらのコードは開発の初期段階にあり、速度の最適化はなされていません。
