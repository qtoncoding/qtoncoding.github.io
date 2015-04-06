---
layout: post
title: "Recursion with Variadic Template"
description: "Recursive technique with Variadic Template in C++11"
category: cplusplus
tags: [C++, modern C++, Variadic, Template]
---
{% include JB/setup %}

<!-- Overview -->
<h3>Overview</h3>

Variadic Template is a feature in C++11 that allows programmer to make template classes or functions with multiple template arguments. This post will demonstrate a technique to iterate through each of the arguments recursively.

<!-- Example -->
<h3>Example</h3>


A variadic template class looks like this

<!-- Code -->
{% highlight cplusplus linenos %}
template <typename T ... >
class MultiTemplate 
{
	// ...
}
{% endhighlight %}
<!-- END Code -->
