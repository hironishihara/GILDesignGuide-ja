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

## 11. 実行時に型を指定するImageとImage View
Color Space、Channel深度、Channel順、インタリーブ形式/プラナー形式といったImageの構造は、コンパイル時に結びつけられる、テンプレートパラメータの型によって定義されます。
それらのパラメータのいくつかが実行時になって初めて決まるといった場合もしばしば存在します。
例として、与えられたパスにある画像を開き、くるりと回転させ、画像オリジナルのColor SpaceとChannel深度で保存する、というモジュールを書く場合を考えましょう。
我々のgeneric imageを使って、これをどのように書くことができるでしょうか？
読み込むコードから返されるImageはどのような型でしょうか？

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

`any_image`と`any_image_view`は、インスタンス化される型について非テンプレートの基本的なBase型と一意のインスタンス型識別子に分解した、GILの`variant`クラスのサブクラスです。
この基本的なBase型インスタンスは、バイトのブロックとして表わされます。
そのブロックは、指定された型の中で最大の型を保持するために十分な規模となります。
GILの`variant`は`boost::variant`と考え方は似ています(だからこそ、そこから名前を拝借したわけです)が、現在のboostの実装とは幾つかの点で異なっています。
おそらく、最大の違いは、許可する型を列挙するMPLランダムアクセスシークエンスをGILの`variant`が引数として常に1個とることです。
インタフェイスをひとつにしたことで、GILの`variant`はジェネリックコードの中で使いやすくなっています。

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

処理を実行するための関数オブジェクトを`apply_operation`に渡すことによって、`variant`上での処理が実行されます。
その`variant`で許可されている全ての型のためのコードがインスタンス化され、`switch`文を経由して適切なインスタンスが選択されます。
Image Viewアルゴリズムは、一般的に、少なくともPixel数に対して線形な時間計算量をもつことから、Image View `variant`による1個の`switch`文がテンプレートによるImage Viewと比較して実際に測定可能なほどのパフォーマンスのオーバーヘッドを加えることはありません。

`variant`は基本的な型と同じように振舞います。
`variant`のコピーコンストラクタは、基本的な型のインスタンスのコピーコンストラクタを呼び出します。
比較演算子は、ふたつのインスタンスの型が同じか否かを確認し、それから`operator==`などを呼び出します。
`variant`のデフォルトコンストラクタは、最初に指定されている型のデフォルトコンストラクタを呼びます。
これは、`any_image_view`が浅いデフォルトコンストラクタ、コピーコンストラクタ、代入、比較をもつ一方で、`any_image`が深いデフォルトコンストラクタ、コピーコンストラクタ、代入、比較をもつことを意味します。

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

`variant`は、それが取りうる全てのModel毎にアルゴリズムをインスタンス化するので、(特に、ふたつ以上の`variant`を引数に取るアルゴリズムでは)注意して用いるべきです。
これは、コンパイル時間と実行ファイルのサイズに甚大な影響を与える可能性があります。
これらのような欠点もありますが、`variant`はコンパイル時の型解決によるスピードと実行時の型解決による柔軟性を組み合わせることを可能にする強力な手法です。
`variant`は、パラメータが異なる複数の画像をひとつのコレクションとして扱い、それらを同じコンテナに収めることを可能にします。
