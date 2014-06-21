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

## <a name="section_14"> 14. サンプルコード

### <a name="section_14_1"> Pixelレベルの処理

Pixelの値、ポインタ、参照を用いる処理を、ここにいくつか示します。

{% highlight C++ %}

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

{% endhighlight %}

続いては、Pixelをジェネリックコードの中でどのように使うのかを示します。

{% highlight C++ %}

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

{% endhighlight %}

この例が示すように、入力と出力が共に`PixelConcept`と`MutablePixelConcept`に各々基づいたModelである限りにおいて、それらは、参照であっても値であっても構いませんし、プラナー形式であってもインタリーブ形式であっても構いません。

### <a name="section_14_2"> 安全のためのバッファを備えた、Imageのコピー

最大で2K+1×2K+1の2次元カーネルを用いて画像のコンボリュージョンを行いたい場合を想定します。
そのような場合には、画像の境界線の外側にK個のPixel分のマージンを作ってみるのはどうでしょうか。
次のように作成します。

{% highlight C++ %}

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

{% endhighlight %}

十分な大きさのImageを確保し、`subimage_view`を使って(k,k)を始点とする`src`と同じサイズの中心領域を指定する浅いImage(すなわち、View)を作成し、`src`をその中心領域へコピーします。
もしマージンに初期化が必要であれば、`fill_pixels`を実行しておくこともできるでしょう。
`copy_pixels`アルゴリズムを使って、このコードをいかにシンプルにするかを示します。

{% highlight C++ %}

template <typename SrcView, typename DstImage>
void create_with_margin(const SrcView& src, int k, DstImage& result) {
    result.recreate(src.width()+2*k, src.height()+2*k);
    copy_pixels(src, subimage_view(view(result), k,k,src.width(),src.height()));
}

{% endhighlight %}

(`image::recreate`は、`operator=`が不必要なコピーコンストラクションを行う分、効率的であることにも注意してください。)
上記のサンプルはColor Spece、Pixel深度、プラナー形式であるかインタリーブ形式であるかを問わずに動作するだけではありません。
最適化されているのです。
GILは`std::copy`をオーバーライドします。
すなわち、行の末尾にパディングのない同じサイズの2個のインタリーブImageの上で呼ばれた場合には、単に`memmove`を実行します。
プラナーImageのときには、各Channelに対して`memmove`を実行します。
一方のImageがパディングをもっていた場合、(ちょうど今回の場合と同じように、)各行に対して`memmove`を実行するでしょう。
Imageがパディングをもっていない場合、(行の末尾をチェックが必要な少々複雑な1次元Iteratorではなく、)軽量な水平方向Iteratorを使うでしょう。
staticなパラメータと実行時に型が決まるパラメータの両方を考慮して、最速の方法を選択します。

### <a name="section_14_3"> ヒストグラム
ヒストグラムは、各瓶に振り分けられたPixel値の数をカウントすることで得られます。
グレイスケールPixelは整数型に変換可能なので、次に示すメソッドはグレイスケール(単要素の)Image Viewを取ります。

{% highlight C++ %}

template <typename GrayView, typename R>
void grayimage_histogram(const GrayView& img, R& hist) {
    for (typename GrayView::iterator it=img.begin(); it!=img.end(); ++it)
        ++hist[*it];
}

{% endhighlight %}

`boost::lambda`とGILの`for_each_pixel`アルゴリズムを用いると、もっとコンパクトに書くことができます。

{% highlight C++ %}

template <typename GrayView, typename R>
void grayimage_histogram(const GrayView& v, R& hist) {
    for_each_pixel(v, ++var(hist)[_1]);
}

{% endhighlight %}

ここの`for_each_pixel`は`std::for_each`を呼び出しており、`var`と`_1`は`boost::lambda`コンストラクトです。
明度のヒストグラムを算出するには、ImageのグレイスケールViewを使って上記のメソッドを呼び出します。

{% highlight C++ %}

template <typename View, typename R>
void luminosity_histogram(const View& v, R& hist) {
    grayimage_histogram(color_converted_view<gray8_pixel_t>(v),hist);
}

{% endhighlight %}

これは、次のように呼び出します。

{% highlight C++ %}

unsigned char hist[256];
std::fill(hist,hist+256,0);
luminosity_histogram(my_view,hist);

{% endhighlight %}

また、画像の2番目のChannelの左上100x100についてのヒストグラムが見たい場合には、次のように呼びます：

{% highlight C++ %}

grayimage_histogram(nth_channel_view(subimage_view(img,0,0,100,100),1),hist);

{% endhighlight %}

Pixelがコピーされることもなければ、余計なメモリが確保されることもありません。
すなわち、サポートされたColor SpaceとChannel深度であれば、このコードは入力Pixelの上で直接実行されます。
プラナー形式でもインタリーブ形式でも問題ありません。

### <a name="section_14_4"> Image Viewの使用

次のコードで、Image Viewの威力を説明したいと思います。

{% highlight C++ %}

jpeg_read_image("monkey.jpg", img);
step1=view(img);
step2=subimage_view(step1, 200,300, 150,150);
step3=color_converted_view<rgb8_view_t,gray8_pixel_t>(step2);
step4=rotated180_view(step3);
step5=subsampled_view(step4, 2,1);
jpeg_write_view("monkey_transform.jpg", step5);

{% endhighlight %}

途中経過の画像をここに示します。

![途中経過](http://hironishihara.github.com/GILDesignGuide-ja/src/img/monkey_steps.jpg "途中経過")

Pixelは全くコピーされていません。
全ての作業は`jpeg_write_view`のなかで行われます。
さきほどの`luminosity_histgram`をstep5で呼べば、うまく動作するはずです。
