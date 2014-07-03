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
2. About Concepts
-->

## <a name="section_02"> 2. Conceptについて

<!--
All constructs in GIL are models of GIL concepts.
A concept is a set of requirements that a type (or a set of related types) must fulfill to be used correctly in generic algorithms.
The requirements include syntactic and algorithming guarantees.
For example, GIL's class pixel is a model of GIL's PixelConcept.
The user may substitute the pixel class with one of their own, and, as long as it satisfies the requirements of PixelConcept, all other GIL classes and algorithms can be used with it.
See more about concepts here: http://www.generic-programming.org/languages/conceptcpp/
-->

GILで用いられる全ての構成概念(コンストラクト)は、GILが定めるConceptに基づいたModelです。
Conceptとは、型(もしくは、関連する型のセット)がジェネリックアルゴリズム内で正しく利用されるために満たさなければならない要件のセットです。
これらの要件には、構文的な保証とアルゴリズム的な保証が含まれます。
例えば、GILコンストラクトのひとつであるPixelクラスは、GILの`PixelConcept`に基づいたModelです。
`PixelConcept`に示された要件を満たす限りにおいて、ユーザはPixelクラスを独自のPixelクラスに置き換えることができ、その独自のPixelクラスを他のGILクラスやアルゴリズムと共に使用することができます。
Conceptに関する詳細は、次のURLを参照ください。  
<http://www.generic-programming.org/languages/conceptcpp/>  

<!--
In this document we will use a syntax for defining concepts that is described in a proposal
for a Concepts extension to C++0x specified here: http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2006/n2081.pdf
-->

この文章では、次のURLにあるC++0xのConcept拡張の提案書に記述されている、Concept定義のための構文を使用します。  
<http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2006/n2081.pdf>

<!--
Here are some common concepts that will be used in GIL.
Most of them are defined here: http://www.generic-programming.org/languages/conceptcpp/concept_web.php
-->

ここで、GILでよく用いられるいくつかのConceptを紹介します。
そのほとんどは、次のサイトで定義されています。  
<http://www.generic-programming.org/languages/conceptcpp/concept_web.php>  

{% highlight C++ %}

auto concept DefaultConstructible<typename T> {
    T::T();
};

auto concept CopyConstructible<typename T> {
    T::T(T);
    T::~T();
};

auto concept Assignable<typename T, typename U = T> {
    typename result_type;
    result_type operator=(T&, U);
};

auto concept EqualityComparable<typename T, typename U = T> {
    bool operator==(T x, T y);
    bool operator!=(T x, T y) { return !(x==y); }
};

concept SameType<typename T, typename U> {  unspecified  };
template<typename T> concept_map SameType<T, T> {  unspecified  };

auto concept Swappable<typename T> {
    void swap(T& t, T& u);
};

{% endhighlight %}

<!--
Here are some additional basic concepts that GIL needs:
-->

また、GILが必要とする基本的なConceptを追加でいくつか挙げておきます。

{% highlight C++ %}

auto concept Regular<typename T> : DefaultConstructible<T>, CopyConstructible<T>, EqualityComparable<T>, Assignable<T>, Swappable<T> {};

auto concept Metafunction<typename T> {
    typename type;
};

{% endhighlight %}
