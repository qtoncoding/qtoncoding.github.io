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
    template <typename T>
    class SmartPointer
    {
    };

<!-- END Code -->
 A template will make this pointer usable with any type of object we want.
 
 A smart pointer contains a raw pointer to the object it points to, as well as a reference counter.
    private:
       T* rawPointer;
       int* refCount;

Next we provide the default constructor, copy constructor, move constructor, assignment operator for object pointer and smart pointer and finally a destructor.
    // Default Constructor
    SmartPointer<T>() : rawPointer(nullptr), refCount(nullptr) {}
    
    // Copy Constructor
    SmartPointer<T>(SmartPointer<T> const & other) : rawPointer(other.rawPointer), refCount(other.refCount)
    {
        ++(*refCount);
    }

    // Move Constructor
    SmartPointer<T>(SmartPointer<T> && other)
    {
        rawPointer = nullptr;
        refCount = nullptr;
        std::swap(rawPointer, other.rawPointer);
        std::swap(refCount, other.refCount);
    }

    // Constructor from object pointer (for calling new)
    SmartPointer<T>(T* object) : rawPointer(object), refCount(new int(1)) {}

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
