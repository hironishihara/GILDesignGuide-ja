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


<!--
16. Technicalities
-->

## <a name="section_16"> 16. より専門的な事項

<!--
Creating a reference proxy

Sometimes it is necessary to create a proxy class that represents a reference to a given object.
Examples of these are GIL's reference to a planar pixel (planar_pixel_reference) and GIL's subbyte channel references.
Writing a reference proxy class can be tricky.
One problem is that the proxy reference is constructed as a temporary object and returned by value upon dereferencing the iterator:
-->

### <a name="section_16_1"> 参照Proxyの作成

与えられたオブジェクトへの参照となるProxy型の作成が必要となる場合があります。
これらの例として、GILのプラナー形式Pixelへの参照やGILのサブバイトChannelへの参照が挙げられます。
参照Proxy型の記述に際しては、注意が必要です。
問題の1番目は、Proxy参照が一時的なオブジェクトとして構築され、Iteratorの間接参照から取得した値を返すことです。

{% highlight C++ %}

struct rgb_planar_pixel_iterator {
   typedef my_reference_proxy<T> reference;
   reference operator*() const { return reference(red,green,blue); }
};

{% endhighlight %}

<!--
The problem arises when an iterator is dereferenced directly into a function that takes a mutable pixel:
-->

この問題は、mutableなPixel型を引数に取る関数がIteratorの間接参照から取得した値を引数に取るときに起こります。

{% highlight C++ %}

template <typename Pixel>    // Models MutablePixelConcept
void invert_pixel(Pixel& p);

rgb_planar_pixel_iterator myIt;
invert_pixel(*myIt);        // compile error!

{% endhighlight %}

<!--
C++ does not allow for matching a temporary object against a non-constant reference.
The solution is to:

Use const qualifier on all members of the reference proxy object:
-->

C++では一時的なオブジェクトを非constantな参照と組み合わせることを許可していません。
この問題は次のように解決します。

- 参照Proxyオブジェクトの全メンバに対して、const修飾子を使います

{% highlight C++ %}

template <typename T>
struct my_reference_proxy {
    const my_reference_proxy& operator=(const my_reference_proxy& p) const;
    const my_reference_proxy* operator->() const { return this; }
    ...
};

{% endhighlight %}

<!--
Use different classes to denote mutable and constant reference (maybe based on the constness of the template parameter)
Define the reference type of your iterator with const qualifier:
-->

- (おそらくはテンプレートパラメタのconst性に基づく)mutableでconstantな参照であることを示すためには、異なるクラスを使用します
- const修飾子を伴った、独自Iteratorの参照型を定義します

{% highlight C++ %}

struct iterator_traits<rgb_planar_pixel_iterator> {
   typedef const my_reference_proxy<T> reference;
};

{% endhighlight %}

<!--
A second important issue is providing an overload for swap for your reference class.
The default std::swap will not work correctly.
You must use a real value type as the temporary.
A further complication is that in some implementations of the STL the swap function is incorreclty called qualified, as std::swap.
The only way for these STL algorithms to use your overload is if you define it in the std namespace:
-->

2番目の問題は、独自の参照クラスをスワップするためのオーバーロードの提供についてです。
デフォルトの`std::swap`は正確に動作しません。
一時的な値として、実際の値をもつ型を使用しなければなりません。
さらに複雑なことには、STLのいくつかの実装では`swap`関数を呼んだ際に誤って正規の`std::swap`が呼ばれます。
これらのSTLアルゴリズムで独自のオーバーロードが用いられるようにするための唯一の方法は、それを`std::namespace`に定義することです。

{% highlight C++ %}

namespace std {
   template <typename T>
   void swap(my_reference_proxy<T>& x, my_reference_proxy<T>& y) {
      my_value<T> tmp=x;
      x=y;
      y=tmp;
   }
}

{% endhighlight %}

<!--
Lastly, remember that constructors and copy-constructors of proxy references are always shallow and assignment operators are deep.
-->

最後に、Proxy参照のコンストラクタやコピーコンストラクタは常に浅く、代入演算子は常に深いことを覚えておいてください。

<!--
We are grateful to Dave Abrahams, Sean Parent and Alex Stepanov for suggesting the above solution.
-->

上記の解決策を提案してくれたDave Abrahams、Sean Parent、Alex Stepanovに感謝します。
