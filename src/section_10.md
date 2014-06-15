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

## 10. Image
Imageは、あるImage Viewが扱うPixel配列を所有するコンテナです。
Imageは、コンストラクタでPixel配列を確保し、デストラクタでそのPixel配列を解放します。
Imageは、深い代入演算子とコピーコンストラクタをもちます。
Imageは、データの所有が重要な意味をもつ場合にだけ使用されます。
ほとんどのSTLアルゴリズムは、コンテナ上ではなく、Range上で動作します。
ほどんどのGILアルゴリズムも同じように、(Imageが提供する)Image Viewの上で動作します。

一般的に、ImageはN次元であり、次のConceptを満たします。

```cpp
concept RandomAccessNDImageConcept<typename Img> : Regular<Img> {
    typename view_t; where MutableRandomAccessNDImageViewConcept<view_t>;
    typename const_view_t = view_t::const_t;
    typename point_t      = view_t::point_t;
    typename value_type   = view_t::value_type;
    typename allocator_type;

    Img::Img(point_t dims, std::size_t alignment=1);
    Img::Img(point_t dims, value_type fill_value, std::size_t alignment);

    void Img::recreate(point_t new_dims, std::size_t alignment=1);
    void Img::recreate(point_t new_dims, value_type fill_value, std::size_t alignment);

    const point_t&        Img::dimensions() const;
    const const_view_t&   const_view(const Img&);
    const view_t&         view(Img&);
};
```

2次元のImageは、追加の要件をもっています。

```cpp
concept RandomAccess2DImageConcept<RandomAccessNDImageConcept Img> {
    typename x_coord_t = const_view_t::x_coord_t;
    typename y_coord_t = const_view_t::y_coord_t;

    Img::Img(x_coord_t width, y_coord_t height, std::size_t alignment=1);
    Img::Img(x_coord_t width, y_coord_t height, value_type fill_value, std::size_t alignment);

    x_coord_t Img::width() const;
    y_coord_t Img::height() const;

    void Img::recreate(x_coord_t width, y_coord_t height, std::size_t alignment=1);
    void Img::recreate(x_coord_t width, y_coord_t height, value_type fill_value, std::size_t alignment);
};
```

GILのImageは、`ImageViewConcept`に基づいたModelでありPixel上で動作するViewをもちます。

```cpp
concept ImageConcept<RandomAccess2DImageConcept Img> {
    where MutableImageViewConcept<view_t>;
    typename coord_t  = view_t::coord_t;
};
```

LocatorやImage Viewと異なり、immutableなImageはそもそも不便であるため、わざわざ'mutable'を指定したConceptのセットはもちません。

##### 関連するConcept:

- `RandomAccessNDImageConcept<Image>`
- `RandomAccess2DImageConcept<Image>`
- `ImageConcept<Image>`

##### Model:

GILは、Value型(すなわち、Pixel)をパラメータにもつテンプレートであり、ImageConceptに基づいたModelである、Imageクラスを提供します。

```cpp
template <typename Pixel, \\ Models PixelValueConcept
          bool IsPlanar,  \\ planar or interleaved image
          typename A=std::allocator<unsigned char> >
class image;
```

デフォルトのImageは、メモリ単位0個分でアラインメントされます。すなわち、各行の末尾にパディングはありません。
比較的複雑な`image_view::iterator`の代わりに、上記のような1次元走査可能なImageでは`image_view::x_iterator`をPixelの走査に使うことができるため、多くの処理が高速です。
Imageのコンストラクタは、ワードアラインメントや8バイトアラインメントといったImageの生成を可能にする、`alignment`パラメータを取ります。
この`alignment`パラメータが、必ずしもバイトであると限らない、メモリ単位によって指定されることに注意してください。
特に、ビットアラインメントImageのメモリ単位は、1ビットです。
