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

## <a name="section_13"> 13. I/O Extension

GILのI/O Extensionは、ローレベルの画像I/Oユーティリティを提供します。
このI/O Extensionは、対応するライブラリとのリンクが必要な各種画像フォーマットについての読み込みや保存をサポートしています。

- JPEG: JPEGファイルを利用するには、`gil/extension/io/jpeg_io.hpp`をインクルードする必要があります。
もし、実行時に型が決まるImageを利用するのであれば、かわりに`gil/extension/io/jpeg_dynamic_io.hpp`をインクルードする必要があります。
また、`libjpeg.lib` (<http://www.ijp.org> から利用可能です)のコンパイルとリンクを行い、`jpeglib.h`をInclude Pathに追加する必要があります。

- TIFF: TIFFファイルを利用するには、`gil/extension/io/tiff_io.hpp`をインクルードする必要があります。
もし、実行時に型が決まるImageを利用するのであれば、かわりに`gil/extension/io/dynamic_dynamic_io.hpp`をインクルードする必要があります。
また、`libtiff.lib` (<http://www.libtiff.org/> から利用可能です)のコンパイルとリンクを行い、`tiffio.h`をInclude Pathに追加する必要があります。

- PNG: PNGファイルを利用するには、`gil/extension/io/png_io.hpp`をインクルードする必要があります。
もし、実行時に型が決まるImageを利用するのであれば、かわりに`gil/extension/io/png_dynamic_io.hpp`をインクルードする必要があります。
また、`libpng.lib` (<http://www.libpng.org/> から利用可能です)のコンパイルとリンクを行い、`png.h`をInclude Pathに追加する必要があります。

これらのライブラリを全てインストールする必要はありません。
使うものだけで十分です。
JPEGファイル用のI/O APIを示します(他のライブラリを使う場合には、"jpeg"を"tiff"または"png"に置き換えてください)。

{% highlight C++ %}

// Returns the width and height of the JPEG file at the specified location.
// Throws std::ios_base::failure if the location does not correspond to a valid JPEG file
point2<std::ptrdiff_t> jpeg_read_dimensions(const char*);

// Allocates a new image whose dimensions are determined by the given jpeg image file, and loads the pixels into it.
// Triggers a compile assert if the image color space or channel depth are not supported by the JPEG library or by the I/O extension.
// Throws std::ios_base::failure if the file is not a valid JPEG file, or if its color space or channel depth are not
// compatible with the ones specified by Image
template <typename Img> void jpeg_read_image(const char*, Img&);

// Allocates a new image whose dimensions are determined by the given jpeg image file, and loads the pixels into it,
// color-converting and channel-converting if necessary.
// Triggers a compile assert if the image color space or channel depth are not supported by the JPEG library or by the I/O extension.
// Throws std::ios_base::failure if the file is not a valid JPEG file or if it fails to read it.
template <typename Img>               void jpeg_read_and_convert_image(const char*, Img&);
template <typename Img, typename CCV> void jpeg_read_and_convert_image(const char*, Img&, CCV color_converter);

// Loads the image specified by the given jpeg image file name into the given view.
// Triggers a compile assert if the view color space and channel depth are not supported by the JPEG library or by the I/O extension.
// Throws std::ios_base::failure if the file is not a valid JPEG file, or if its color space or channel depth are not
// compatible with the ones specified by View, or if its dimensions don't match the ones of the view.
template <typename View> void jpeg_read_view(const char*, const View&);

// Loads the image specified by the given jpeg image file name into the given view and color-converts (and channel-converts) it if necessary.
// Triggers a compile assert if the view color space and channel depth are not supported by the JPEG library or by the I/O extension.
// Throws std::ios_base::failure if the file is not a valid JPEG file, or if its dimensions don't match the ones of the view.
template <typename View>               void jpeg_read_and_convert_view(const char*, const View&);
template <typename View, typename CCV> void jpeg_read_and_convert_view(const char*, const View&, CCV color_converter);

// Saves the view to a jpeg file specified by the given jpeg image file name.
// Triggers a compile assert if the view color space and channel depth are not supported by the JPEG library or by the I/O extension.
// Throws std::ios_base::failure if it fails to create the file.
template <typename View> void jpeg_write_view(const char*, const View&);

// Determines whether the given view type is supported for reading
template <typename View> struct jpeg_read_support {
    static const bool value = ...;
};

// Determines whether the given view type is supported for writing
template <typename View> struct jpeg_write_support {
    static const bool value = ...;
};

{% endhighlight %}

Dynamic Image Extensionを使う場合には、"`jpeg_io.hpp`"に代えて"`jpeg_dynamic_io.hpp`"をインクルードするようにしてください。
Dynamic Imageを扱う場合には、上記のメソッドに加えて、次に示すオーバーロードをもちます。

{% highlight C++ %}

// Opens the given JPEG file name, selects the first type in Images whose color space and channel are compatible to those of the image file
// and creates a new image of that type with the dimensions specified by the image file.
// Throws std::ios_base::failure if none of the types in Images are compatible with the type on disk.
template <typename Images> void jpeg_read_image(const char*, any_image<Images>&);

// Saves the currently instantiated view to a jpeg file specified by the given jpeg image file name.
// Throws std::ios_base::failure if the currently instantiated view type is not supported for writing by the I/O extension
// or if it fails to create the file.
template <typename Views>  void jpeg_write_view(const char*, any_image_view<Views>&);

{% endhighlight %}

上記の全てのメソッドは、`const char*`の代わりに`std::string`を取るオーバーロードをもちます。
