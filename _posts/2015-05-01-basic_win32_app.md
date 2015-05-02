---
layout: post
title: "Basic Win32 Application"
description: "Basic structures of a Win32 application"
category: Win32
tags: [win32, c++, windows]
---

<h3>Overview</h3>

Have you ever wondered how Windows programs with UI are made? 
If you tried to create a Win32 App in Visual Studio, the generated code is rather confusing and overwhelming
if you don't know what's going on. In this blog post I'll show you the basic and essential of setting up a Win32 app.

<h3>Main function</h3>

Let's start by creating a new .cpp file. On the top of the file we will add ```#include <windows.h>```. 
This header file has most things you will need to create a Win32 app. 

Same as any C++ application, we need a ```main``` function. For a Win32 App, it's a ```CALLBACK``` function named ```WinMain```.

{% highlight c++ %}

int CALLBACK
WinMain(HINSTANCE Instance,
        HINSTANCE PrevInstance,
        LPSTR CmdLine,
        int ShowOption)
{

}

{% endhighlight %}

Unlike normal C++ programs, WinMain will not be called immediately when we start our app. 
It's supposed to be called by the Windows Runtime. This is why this function is a ```CALLBACK```, 
that means the system will call back into our code from external code. 

The first argument is a ```HANDLE``` to the instance of our app (hence ```HINSTANCE```, ```HANDLE-INSTANCE```).

The second argument is the ```HANDLE``` to the previous instance of our app. 
If an app is already running and it gets launched for the second time, the first instance will be passed into this argument.

The third argument is a long pointer to a string, which is the command lines used when the app is launched.

The last argument is a number, an enum if you will, for which state the application will start in, normal window, minimized or maximized.

Now that we have the main function, what to put in there? 

Applications are identified by their "Class", or "Window Class".
This can be regarded as a kind of namespace to keeps the windows of apps fromb being confused with each other. 
For example, "Explorer" and "Chrome" can both have a window called "Root Frame", but because they belong to different "Window Classes", 
or different application, they are not confused by Windows.

We will start by declaring a [WNDCLASS](https://msdn.microsoft.com/en-us/library/windows/desktop/ms633576(v=vs.85).aspx). 
This struct has a lot of fields, but with most apps, we don't need all of them. Let's add ```WindowClass``` to ```WinMain```.

{% highlight c++ %}

WNDCLASS WindowClass = {};
WindowClass.style = CS_HREDRAW | CS_VREDRAW;
WindowClass.lpfnWndProc = /* To fill in later */;
WindowClass.hInstance = Instance;
WindowClass.hIcon = nullptr;
WindowClass.hCursor = LoadCursor(nullptr, IDC_ARROW);
WindowClass.lpszClassName = "CoolAwesomeWin32App";

{% endhighlight %}

```WNDCLASS WindowClass = {}``` will create a new struct of type WNDCLASS and initialize all fields to NULL. 
This is needed because we are not manually filling in all fields of the struct and they are better off being NULL.

```WindowClass.style``` sets the style of the Window Class. Here we use ```CS_HREDRAW``` and ```CS_VREDRAW``` by OR-ing them together.
These constances are bit flags, so to use them together, bitwise OR is the operation we want. ```CS_HREDRAW``` will make sure the drawing area of our app
is redrawn when we resize it horizontally. ```CS_VREDRAW``` does the same thing, vertically.

```WindowClass.hInstance``` is where we attach the application instance to the Window Class. So we just assign ```Instance``` to it.

```WindowClass.hIcon``` is the Icon for our app. Setting it to ```nullptr``` will force Windows to use a default icon.

```WindowClass.hCursor``` is the mouse cursor for the app. Here we use ```LoadCursor``` to load the standard arrow cursor of Windows by passing it ```nullptr``` for the instance to get the cursor resource from, and ```IDC_ARROW``` for the default arrow.

```WindowClass.lpszClassName``` is the name for the class. Simply assign a string to it.

This left us with ```WindowClass.lpfnWndProc```. This is a long pointer to a function called Window Procedure. 
This function is a callback what will handle the Window Messages passed to our app from Windows.

Now that we have a Window Class, we need to Register it with Windows by using ```RegisterClass```.

{% highlight c++ %}

if (RegisterClass(&WindowClass))
{

}

{% endhighlight %}



