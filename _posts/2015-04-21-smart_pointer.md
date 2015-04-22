---
layout: post
title: "Reference Counting Smart Pointer in C++"
description: "A Simple Implementation of Reference Counting Smart Pointer in C++"
category: cplusplus
tags: [C++, modern C++, pointer, smart]
---

<!-- Overview -->
<h3>Overview</h3>

Pointers are hard to use and think about. There are many hazards that come with them such as dangling pointers and memory leaking. A solution to these issues is to use a smart pointer. A smart pointer usually is a wrapper around a raw pointer, with a reference count to keep track of the object, automatically cleaning up when the object is no longer referred to.

In the Standard Templated Library of C++ there are 2 kinds of smart pointers: ```unique_ptr``` and ```shared_ptr```. They should be used instead of raw pointer in most cases, unless your target device cannot afford the neligible overhead of these pointers.

If you are interested in how a smart pointer is generally implemented, this post will show you how it's done.

<h3>Implementation</h3>
We start out with a template declaration

{% highlight c++ %}
template <typename T>
class SmartPointer
{
};
{% endhighlight %}

A template will make this pointer usable with any type of object we want.
 
A smart pointer contains a raw pointer to the object it points to, as well as a reference counter.

{% highlight c++ %}
private:
   T* rawPointer;
   int* refCount;
{% endhighlight %}

Next we provide the default constructor, copy constructor, move constructor, assignment operator for object pointer and smart pointer and finally a destructor.

For default constructor, we use initializer list to initialize the raw pointer and reference counter to  ```nullptr```.

{% highlight c++ %}
public:
    // Default Constructor
    SmartPointer<T>() : rawPointer(nullptr), refCount(nullptr) {}
{% endhighlight %}

Copy constructor uses initializer list to initilize ```rawPointer``` to ```other.rawPointer``` and ```refCount``` to ```other.refCount```. However, we also need to increment the reference counter.

{% highlight c++ %}
        // Copy Constructor
    SmartPointer<T>(SmartPointer<T> const & other) : rawPointer(other.rawPointer), refCount(other.refCount)
    {
        ++(*refCount);
    }
{% endhighlight %}

The move constructor is something new to people who are not familier to C++11. What it does is it steals the content of the other pointer without copying data over, leaving the other pointer with garbage or null. We accomplish this by initializing our fields to ```nullptr``` and use ```std::swap``` to swap the data around.

{% highlight c++ %}
        // Move Constructor
    SmartPointer<T>(SmartPointer<T> && other)
    {
        rawPointer = nullptr;
        refCount = nullptr;
        std::swap(rawPointer, other.rawPointer);
        std::swap(refCount, other.refCount);
    }
{% endhighlight %}

The next constructor is to create our pointer from a raw pointer, to use with syntax such as 

{% highlight c++ %}
SmartPointer<Cat> catPtr = new Cat();
{% endhighlight %}

We do this by initializing the ```rawPointer``` with the ```object``` pointer, and initialize ```refCount``` with a dynamically allocated ```int```.

{% highlight c++ %}
        // Constructor from object pointer (for calling new)
    SmartPointer<T>(T* object) : rawPointer(object), refCount(new int(1)) {}
{% endhighlight %}

Next up, we define the assignment operators for another SmartPointer or a raw object pointer. To do this, we first decrement the reference counter, clean up the object if reference hits 0. After that assign ```rawPointer``` the object pointer, ```refCount``` to ```other.refCount``` or 1 if assigning to an object pointer and increment ```refCount```.

{% highlight c++ %}
        // Assignment operator for SmartPointer
    SmartPointer<T> & operator= (SmartPointer<T> const & other) 
    {
        // Decrement refCount on old value, clean up if needed
        if (refCount != nullptr)
        {
            --(*refCount);
            if (*refCount == 0)
            {
                delete rawPointer;
                rawPointer = nullptr;
                delete refCount;
                refCount = nullptr;
            }
        }

        // Assign values
        rawPointer = other.rawPointer;
        refCount = other.refCount;
        ++(*refCount);
        return *this;
    }

    SmartPointer<T> & operator= (T* const object)
    {
        if (refCount != nullptr)
        {
            --(*refCount);
            if (*refCount == 0)
            {
                delete rawPointer;
                rawPointer = nullptr;
                delete refCount;
                refCount = nullptr;
            }
        }
        rawPointer = object;
        refCount = new int(1);
        return *this;
    }
{% endhighlight %}

Finally, we write the destructor. In the destructor we decrement reference counter, clean up the object and the reference counter if reference count is 0.

{% highlight c++ %}
        // Destructor
    ~SmartPointer<T>() 
    {
        if (refCount != nullptr)
        {
            --(*refCount);
            if (*refCount == 0)
            {
                delete rawPointer;
                rawPointer = nullptr;
                delete refCount;
                refCount = nullptr;
            }
        }
    } 
{% endhighlight %}

So far so good. But a pointer that cannot be dereferenced isn't that useful. So let's write the dereference operators. These operators return a reference to the object we point to.
{% highlight c++ %}
        // Dereference operators
    T & operator->() 
    {
        return *rawPointer;
    }

    T & operator*()
    {
        return *rawPointer;
    }
{% endhighlight %}

<h3>Example</h3>

Here is a simple program to test out the smart pointer

{% highlight c++ linenos %}
int main()
{
    SmartPointer<int> p1;                      // Default constructor, rawPointer and refCount are nullptr
    SmartPointer<int> p2(new int(10));         // Copy constructor from object, *rawPointer = 10, refCount = 1
    {
        SmartPointer<int> p3 = p2;             // Assignment operator, *rawPointer = 10, refCount = 2
    }                                          // p3 goes out of scope, refCount = 1
    
    {
        SmartPointer<int> p4(p2);              // Assignment operator, *rawPointer = 10, refCount = 2
    }                                          // p4 goes out of scope, refCount = 1
    
    SmartPointer<int> p5 = std::move(p2);      // Move constructor, *rawPointer = 10, refCount = 1
                                               // p2->rawPointer = nullptr, p2->refCount = nullptr
    std::cout << "Value: " << *p5 << std:endl; // Dereference
}                                              // p5 goes out of scope, refCount goes to 0, object gets deleted 
{% endhighlight %}

There you go! Reference counting Smart Pointer isn't so hard now huh?


Cheers,
Thai
