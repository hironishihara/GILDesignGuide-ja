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
3. Point
-->

## <a name="section_03"> 3. Point

<!--
A point defines the location of a pixel inside an image.
It can also be used to describe the dimensions of an image.
In most general terms, points are N-dimensional and model the following concept:
-->

Pointは、Pixelの画像上における位置を定義します。
また、画像の次元数を表現するためにも用いられます。
一般に、PointはN次元であり、次に示すConceptに基づいたModelです。

{% highlight C++ %}

concept PointNDConcept<typename T> : Regular<T> {
    // the type of a coordinate along each axis
    template <size_t K> struct axis; where Metafunction<axis>;

    const size_t num_dimensions;

    // accessor/modifier of the value of each axis.
    template <size_t K> const typename axis<K>::type& T::axis_value() const;
    template <size_t K>       typename axis<K>::type& T::axis_value();
};

{% endhighlight %}

<!--
GIL uses a two-dimensional point, which is a refinement of PointNDConcept in which both dimensions are of the same type:
-->

GILは、2つの次元の座標の型が同じになるように改良した`PointNDConcept`である、2次元のPointを用います。

{% highlight C++ %}

concept Point2DConcept<typename T> : PointNDConcept<T> {
    where num_dimensions == 2;
    where SameType<axis<0>::type, axis<1>::type>;

    typename value_type = axis<0>::type;

    const value_type& operator[](const T&, size_t i);
          value_type& operator[](      T&, size_t i);

    value_type x,y;
};

{% endhighlight %}

<!--
Related Concepts:

PointNDConcept<T>
Point2DConcept<T>
-->

#### 関連するConcept:

- `PointNDConcept<T>`
- `Point2DConcept<T>`

<!--
Models:

GIL provides a model of Point2DConcept, point2<T> where T is the coordinate type.
-->

#### Model:

GILは、`Point2DConcept`に基づいたModelである`point2<T>`を提供します。この`T`は座標の型を表しています。
