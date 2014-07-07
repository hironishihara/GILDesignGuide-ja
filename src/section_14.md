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
14. Sample Code
-->

## 14. サンプルコード

<!--
Pixel-level Sample Code

Here are some operations you can do with pixel values, pointers and references:
-->

### Pixelレベルの処理

Pixelの値、ポインタ、参照を用いる処理を、ここにいくつか示します。

```cpp
rgb8_pixel_t p1(255,0,0);     // make a red RGB pixel
bgr8_pixel_t p2 = p1;         // RGB and BGR are compatible and the channels will be properly mapped.
assert(p1==p2);               // p2 will also be red.
assert(p2[0]!=p1[0]);         // operator[] gives physical channel order (as laid down in memory)
assert(semantic_at_c<0>(p1)==semantic_at_c<0>(p2)); // this is how to compare the two red channels
get_color(p1,green_t()) = get_color(p2,blue_t());  // channels can also be accessed by name

const unsigned char* r;
const unsigned char* g;
const unsigned char* b;
rgb8c_planar_ptr_t ptr(r,g,b); // constructing const planar pointer from const pointers to each plane

rgb8c_planar_ref_t ref=*ptr;   // just like built-in reference, dereferencing a planar pointer returns a planar reference

p2=ref; p2=p1; p2=ptr[7]; p2=rgb8_pixel_t(1,2,3);    // planar/interleaved references and values to RGB/BGR can be freely mixed

//rgb8_planar_ref_t ref2;      // compile error: References have no default constructors
//ref2=*ptr;                   // compile error: Cannot construct non-const reference by dereferencing const pointer
//ptr[3]=p1;                   // compile error: Cannot set the fourth pixel through a const pointer
//p1 = pixel<float, rgb_layout_t>();// compile error: Incompatible channel depth
//p1 = pixel<bits8, rgb_layout_t>();// compile error: Incompatible color space (even though it has the same number of channels)
//p1 = pixel<bits8,rgba_layout_t>();// compile error: Incompatible color space (even though it contains red, green and blue channels)
```

<!--
Here is how to use pixels in generic code:
-->

続いては、Pixelをジェネリックコードの中でどのように使うのかを示します。

```cpp
template <typename GrayPixel, typename RGBPixel>
void gray_to_rgb(const GrayPixel& src, RGBPixel& dst) {
    gil_function_requires<PixelConcept<GrayPixel> >();
    gil_function_requires<MutableHomogeneousPixelConcept<RGBPixel> >();

    typedef typename color_space_type<GrayPixel>::type gray_cs_t;
    BOOST_STATIC_ASSERT((boost::is_same<gray_cs_t,gray_t>::value));

    typedef typename color_space_type<RGBPixel>::type  rgb_cs_t;
    BOOST_STATIC_ASSERT((boost::is_same<rgb_cs_t,rgb_t>::value));

    typedef typename channel_type<GrayPixel>::type gray_channel_t;
    typedef typename channel_type<RGBPixel>::type  rgb_channel_t;

    gray_channel_t gray = get_color(src,gray_color_t());
    static_fill(dst, channel_convert<rgb_channel_t>(gray));
}

// example use patterns:

// converting gray l-value to RGB and storing at (5,5) in a 16-bit BGR interleaved image:
bgr16_view_t b16(...);
gray_to_rgb(gray8_pixel_t(33), b16(5,5));

// storing the first pixel of an 8-bit grayscale image as the 5-th pixel of 32-bit planar RGB image:
rgb32f_planar_view_t rpv32;
gray8_view_t gv8(...);
gray_to_rgb(*gv8.begin(), rpv32[5]);
```

<!--
As the example shows, both the source and the destination can be references or values, planar or interleaved, as long as they model PixelConcept and MutablePixelConcept respectively.
-->

この例が示すように、入力と出力が共に`PixelConcept`と`MutablePixelConcept`に各々基づいたModelである限りにおいて、それらは、参照であっても値であっても構いませんし、プラナー形式であってもインタリーブ形式であっても構いません。

<!--
Creating a Copy of an Image with a Safe Buffer

Suppose we want to convolve an image with multiple kernels, the largest of which is 2K+1 x 2K+1 pixels.
It may be worth creating a margin of K pixels around the image borders.
Here is how to do it:
-->

### 安全のためのバッファを備えた、Imageのコピー

最大で2K+1×2K+1の2次元カーネルを用いて画像のコンボリュージョンを行いたい場合を想定します。
そのような場合には、画像の境界線の外側にK個のPixel分のマージンを作ってみるのはどうでしょうか。
次のように作成します。

```cpp
template <typename SrcView,   // Models ImageViewConcept (the source view)
          typename DstImage>  // Models ImageConcept     (the returned image)
void create_with_margin(const SrcView& src, int k, DstImage& result) {
    gil_function_requires<ImageViewConcept<SrcView> >();
    gil_function_requires<ImageConcept<DstImage> >();
    gil_function_requires<ViewsCompatibleConcept<SrcView, typename DstImage::view_t> >();

    result=DstImage(src.width()+2*k, src.height()+2*k);
    typename DstImage::view_t centerImg=subimage_view(view(result), k,k,src.width(),src.height());
    std::copy(src.begin(), src.end(), centerImg.begin());
}
```

<!--
We allocated a larger image, then we used subimage_view to create a shallow image of its center area of top left corner at (k,k) and of identical size as src, and finally we copied src into that center image.
If the margin needs initialization, we could have done it with fill_pixels.
Here is how to simplify this code using the copy_pixels algorithm:
-->

十分な大きさのImageを確保し、`subimage_view`を使って(k,k)を始点とする`src`と同じサイズの中心領域を指定する浅いImage(すなわち、View)を作成し、`src`をその中心領域へコピーします。
もしマージンに初期化が必要であれば、`fill_pixels`を実行しておくこともできるでしょう。
`copy_pixels`アルゴリズムを使って、このコードをいかにシンプルにするかを示します。

```cpp
template <typename SrcView, typename DstImage>
void create_with_margin(const SrcView& src, int k, DstImage& result) {
    result.recreate(src.width()+2*k, src.height()+2*k);
    copy_pixels(src, subimage_view(view(result), k,k,src.width(),src.height()));
}
```

<!--
(Note also that image::recreate is more efficient than operator=, as the latter will do an unnecessary copy construction).
Not only does the above example work for planar and interleaved images of any color space and pixel depth; it is also optimized.
GIL overrides std::copy - when called on two identical interleaved images with no padding at the end of rows, it simply does a memmove.
For planar images it does memmove for each channel. If one of the images has padding, (as in our case) it will try to do memmove for each row.
When an image has no padding, it will use its lightweight horizontal iterator (as opposed to the more complex 1D image iterator that has to check for the end of rows).
It choses the fastest method, taking into account both static and run-time parameters.
-->

(`image::recreate`は、`operator=`が不必要なコピーコンストラクションを行う分、効率的であることにも注意してください。)
上記のサンプルはColor Spece、Pixel深度、プラナー形式であるかインタリーブ形式であるかを問わずに動作するだけではありません。
最適化されているのです。
GILは`std::copy`をオーバーライドします。
すなわち、行の末尾にパディングのない同じサイズの2個のインタリーブImageの上で呼ばれた場合には、単に`memmove`を実行します。
プラナーImageのときには、各Channelに対して`memmove`を実行します。
一方のImageがパディングをもっていた場合、(ちょうど今回の場合と同じように、)各行に対して`memmove`を実行するでしょう。
Imageがパディングをもっていない場合、(行の末尾をチェックが必要な少々複雑な1次元Iteratorではなく、)軽量な水平方向Iteratorを使うでしょう。
staticなパラメータと実行時に型が決まるパラメータの両方を考慮して、最速の方法を選択します。

<!--
Histogram

The histogram can be computed by counting the number of pixel values that fall in each bin.
The following method takes a grayscale (one-dimensional) image view, since only grayscale pixels are convertible to integers:
-->

### ヒストグラム

ヒストグラムは、各瓶に振り分けられたPixel値の数をカウントすることで得られます。
グレイスケールPixelは整数型に変換可能なので、次に示すメソッドはグレイスケール(単要素の)Image Viewを取ります。

```cpp
template <typename GrayView, typename R>
void grayimage_histogram(const GrayView& img, R& hist) {
    for (typename GrayView::iterator it=img.begin(); it!=img.end(); ++it)
        ++hist[*it];
}
```

<!--
Using boost::lambda and GIL's for_each_pixel algorithm, we can write this more compactly:
-->

`boost::lambda`とGILの`for_each_pixel`アルゴリズムを用いると、もっとコンパクトに書くことができます。

```cpp
template <typename GrayView, typename R>
void grayimage_histogram(const GrayView& v, R& hist) {
    for_each_pixel(v, ++var(hist)[_1]);
}
```

<!--
Where for_each_pixel invokes std::for_each and var and _1 are boost::lambda constructs.
To compute the luminosity histogram, we call the above method using the grayscale view of an image:
-->

ここの`for_each_pixel`は`std::for_each`を呼び出しており、`var`と`_1`は`boost::lambda`コンストラクトです。
明度のヒストグラムを算出するには、ImageのグレイスケールViewを使って上記のメソッドを呼び出します。

```cpp
template <typename View, typename R>
void luminosity_histogram(const View& v, R& hist) {
    grayimage_histogram(color_converted_view<gray8_pixel_t>(v),hist);
}
```

<!--
This is how to invoke it:
-->

これは、次のように呼び出します。

```cpp
unsigned char hist[256];
std::fill(hist,hist+256,0);
luminosity_histogram(my_view,hist);
```

<!--
If we want to view the histogram of the second channel of the image in the top left 100x100 area, we call:
-->

また、画像の2番目のChannelの左上100x100についてのヒストグラムが見たい場合には、次のように呼びます：

```cpp
grayimage_histogram(nth_channel_view(subimage_view(img,0,0,100,100),1),hist);
```

<!--
No pixels are copied and no extra memory is allocated - the code operates directly on the source pixels, which could be in any supported color space and channel depth.
They could be either planar or interleaved.
-->

Pixelがコピーされることもなければ、余計なメモリが確保されることもありません。
すなわち、サポートされたColor SpaceとChannel深度であれば、このコードは入力Pixelの上で直接実行されます。
プラナー形式でもインタリーブ形式でも問題ありません。

<!--
Using Image Views

The following code illustrates the power of using image views:
-->

### Image Viewの使用

次のコードで、Image Viewの威力を説明したいと思います。

```cpp
jpeg_read_image("monkey.jpg", img);
step1=view(img);
step2=subimage_view(step1, 200,300, 150,150);
step3=color_converted_view<rgb8_view_t,gray8_pixel_t>(step2);
step4=rotated180_view(step3);
step5=subsampled_view(step4, 2,1);
jpeg_write_view("monkey_transform.jpg", step5);
```

<!--
The intermediate images are shown here:
-->

途中経過の画像をここに示します。

![途中経過](http://hironishihara.github.com/GILDesignGuide-ja/src/img/monkey_steps.jpg "途中経過")

<!--
Notice that no pixels are ever copied.
All the work is done inside jpeg_write_view. 
If we call our luminosity_histogram with step5 it will do the right thing.
-->

Pixelは全くコピーされていません。
全ての作業は`jpeg_write_view`のなかで行われます。
さきほどの`luminosity_histgram`をstep5で呼べば、うまく動作するはずです。
