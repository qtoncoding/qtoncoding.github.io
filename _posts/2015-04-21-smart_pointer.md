---
layout: post
title: "Reference Counting Smart Pointer in C++"
description: "A Simple Implementation of Reference Counting Smart Pointer in C++"
category: cplusplus
tags: [C++, modern C++, pointer, smart]
---

<!-- Overview -->
<h3>Overview</h3>

Pointers are hard to use and think about. There are many hazards that come with them such as dangling pointers and memory leaking. A solution to these issues is to use a smart pointer. A smart pointer usually is a wrapper around a raw pointer, with a reference count to keep track of the object, automatically cleaning up when the object is no longer referred to and avoid memory leaks.

<!-- Example -->
<h3>Example</h3>
We start out with a template declaration
<!-- Code -->
{% highlight cplusplus %}
template <typename T>
class SmartPointer
{
};
{% endhighlight %}
<!-- END Code -->
 A template will make this pointer usable with any type of object we want.
 
 A smart pointer contains a raw pointer to the object it points to, as well as a reference counter.
 
 {% highlight cplusplus %}
 private:
    T* rawPointer;
    int* refCount;
 {% endhighlight %}


Next we provide the default constructor, copy constructor, move constructor, assignment operator for object pointer and smart pointer and finally a destructor.
{% highlight cplusplus linenos %}
 
{% endhighlight %}
