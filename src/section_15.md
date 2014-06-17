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

## 15. Generic Image Libraryの拡張

GILでは、独自のPixel Iterator、Locator、Image View、Image、Channel型、Color Space、アルゴリズムを定義することができます。
ディスク上の画像やjpegファイルに保存された画像やインターネット上にある画像のVirtual Imageをつくることもできますし、マンデルブロ集合といった完全な合成画像までもつくることができます。
関連するコンセプトに基づいた適切なModelである限りにおいて、それらは既存のあらゆるGILコードと共に動作します。

このような拡張のほとんどはライブラリに変更を求めませんし、だからこそ、これらの拡張を他のモジュールへ提供することが可能なのです。

### 独自のColor Space定義
それぞれのColor Spaceは個別のファイルに記述されています。
新しいColor Spaceを追加するには、あるColor Space(たとえば、`rgb.hpp`)をコピーして、適当に編集するだけです。
もし色変換をサポートしたいと思ったら、新しいColor Spaceと既存のColor Spaceの変換を行うメソッドを提供しなければならないでしょう(`color_convert.hpp`を参照してください)。
利便性を考えて、Pixel、ポインタ、参照、Imageについての`typedef`を提供したくなるかもしれません(`typedefs.h`を参照してください)。

### 独自のChannel型定義
ほとんどの場合、新しいChannel型を使用する際に特別なことをする必要はありません。
それをただ使うだけです。

```cpp
typedef pixel<double,rgb_layout_t>   rgb64_pixel_t;    // 64 bit RGB pixel
typedef rgb64_pixel*                 rgb64_pixel_ptr_t;// pointer to 64-bit interleaved data
typedef image_type<double,rgb_layout_t>::type rgb64_image_t;    // 64-bit interleaved image
```

もし独自のChannel型を使いたいと考えたときには、そのChannel型のための`channel_traits`の実装を提供する必要があるでしょう(`channel.hpp`を参照ください)。
もし、独自のChannel型と既存のChannel型の変換を行いたいと考えたときには、`channel_convert`のオーバーロードを提供する必要があるでしょう。

### 色変換のオーバーロード

独自の色変換を提供したい場合について考えてみましょう。
例えば、カラープロファイルを使った高品質な色変換を実装したいと考えている場合などが考えられます。
いつものように、特定の組み合わせにおける色変換だけを再定義して、その他の組み合わせについては既存のGILが用意する色変換を使いたいと考えているかもしれません。
例として、反転したグレイスケール画像を結果として受け取るという色変換でオーバーロードする方法について示します。

```cpp
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
```

GILの色変換関数はオプションのパラメータとしてカラーコンバータをとります。
独自のカラーコンバータを渡すこともできます。

```cpp
color_converted_view<gray8_pixel_t>(img_view,my_color_converter());
```

### 独自のImage View定義
あるPixelから他のPixelを得るメカニズムか間接参照に対して任意のPixel変換を行うメカニズムかをオーバーロードすることで、独自のPixel IteratorやLocatorやViewを提供することが出来ます。
例として、`color_converted_view` (あるImage Viewが与えられたとき、Pixelの色変換が実行される以外は全くそっくりの新しいViewを作成するfactoryメソッド)の実装をみてみましょう。
まずは、Pixel Iteratorから間接参照するときに呼ばれる関数オブジェクトである、`PixelDereferenceAdaptorConcept`のModelを定義する必要があります。
これは、目的のPixel型へ変換するために`color_convert`を呼びます。

```cpp
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
```

そのとき、間接参照の上で与えられた関数オブジェクト(`deref_t`)を実行するView型を構築する、Image Viewのメンバstructである`add_deref`を使用します。
このケースでは、色変換を実行します。

```cpp
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
```

最終的に、`color_converted_view`のコードは、元となるViewから簡単に`color-converted view`を作成します。

```cpp
template <typename DstP, typename View> inline
typename color_converted_view_type<View,DstP>::type color_convert_view(const View& src) {
    return color_converted_view_type<View,DstP>::make(src);
}
```

(実際の色変換Viewによる変換は、ユーザ独自の色変換メソッドの記述を許可する追加の色変換オブジェクトを取るなど、少し複雑です。)
マンデルブロ集合を定義するVirtual Image Viewを作成する例については、GILチュートリアルを見てください。
