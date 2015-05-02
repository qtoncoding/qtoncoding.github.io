---
layout: post
title: "Basic Win32 Application"
description: "Basic structures of a Win32 application"
category: cplusplus
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

* The first argument is a ```HANDLE``` to the instance of our app (hence ```HINSTANCE```, ```HANDLE-INSTANCE```).

* The second argument is the ```HANDLE``` to the previous instance of our app. 
If an app is already running and it gets launched for the second time, the first instance will be passed into this argument.

* The third argument is a long pointer to a string, which is the command lines used when the app is launched.

* The last argument is a number, an enum if you will, for which state the application will start in, normal window, minimized or maximized.

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

* ```WNDCLASS WindowClass = {}``` will create a new struct of type WNDCLASS and initialize all fields to NULL. 
This is needed because we are not manually filling in all fields of the struct and they are better off being NULL.

* ```WindowClass.style``` sets the style of the Window Class. Here we use ```CS_HREDRAW``` and ```CS_VREDRAW``` by OR-ing them together.
These constances are bit flags, so to use them together, bitwise OR is the operation we want. ```CS_HREDRAW``` will make sure the drawing area of our app
is redrawn when we resize it horizontally. ```CS_VREDRAW``` does the same thing, vertically.

* ```WindowClass.hInstance``` is where we attach the application instance to the Window Class. So we just assign ```Instance``` to it.

* ```WindowClass.hIcon``` is the Icon for our app. Setting it to ```nullptr``` will force Windows to use a default icon.

* ```WindowClass.hCursor``` is the mouse cursor for the app. Here we use ```LoadCursor``` to load the standard arrow cursor of Windows by passing it ```nullptr``` for the instance to get the cursor resource from, and ```IDC_ARROW``` for the default arrow.

* ```WindowClass.lpszClassName``` is the name for the class. Simply assign a string to it.

* This left us with ```WindowClass.lpfnWndProc```. This is a long pointer to a function called Window Procedure. 
This function is a callback what will handle the Window Messages passed to our app from Windows.

Now that we have a Window Class, we need to Register it with Windows by using ```RegisterClass```.

{% highlight c++ %}

if (RegisterClass(&WindowClass))
{

}
else
{
    OutputDebugStringA("Cannot register Window Class\n");
}

{% endhighlight %}

Once our Window Class is successfully registered, we create a window on the screen. This is done by calling [CreateWindowEx](https://msdn.microsoft.com/en-us/library/windows/desktop/ms632680(v=vs.85).aspx).

{% highlight c++ %}

HWND WindowHandle = CreateWindowEx(0,
                                   WindowClass.lpszClassName,
                                   "AwesomeWindowOnTheScreen",
                                   WS_OVERLAPPEDWINDOW | WS_VISIBLE,
                                   CW_USEDEFAULT,
                                   CW_USEDEFAULT,
                                   CW_USEDEFAULT,
                                   CW_USEDEFAULT,
                                   nullptr,
                                   nullptr,
                                   Instance,
                                   nullptr);
                                   
{% endhighlight %}

This function is rather lengthy, so I'll explain each argument in simple terms.

* The first argument is a bit field for [Extended Window Style](https://msdn.microsoft.com/en-us/library/windows/desktop/ff700543(v=vs.85).aspx). For now, we don't use any of them, so we pass ```0``` in.

* The second argument is the Window Class name, so pass the WindowClass.lpszClassName right in there.

* The third argument is the [Window Style](https://msdn.microsoft.com/en-us/library/windows/desktop/ms632600(v=vs.85).aspx). It's also a bit field, and we use ```WS_OVERLAPPEDWINDOW```, which is a preset that gives us a window with a title bar, resizable borders, a system menu when you right click on the title bar, minimize and maximize buttons. We also use ```WS_VISIBLE``` to make the window visible right away.

* The next four arguments are X and Y coordinate of the window on the screen as well as width and height, in that order. Here we use the constance ```CW_USEDEFAULT``` to make Windows give use default value;

* The eigth argument is the ```HANDLE``` to the parent window. Since this is the first window in our app, we pass ```nullptr``` to it.

* The next argument is the ```HANDLE``` to the menus on our windows (such as Files, Edit, View, Help etc...). Since we don't want any menus, we pass ```nullptr``` to it.

* The second to last argument is the instance of our app. Pass ```Instance``` to it.

* The last argument is a bit tricky. It's a pointer to a custom chunk of data to be passed to the Window, it will be in the ```LPARAM``` of the ```WM_CREATE``` message passed to the window. This can be any data, casted to ```void*``` by a ```reinterpret_cast``` and casted back to the original data type in the message handler.

After this call to ```CreateWindowEx``` we would have a window on the screen.

A window constantly receives messages from Windows to notify it of things happening. We need to fetch these message, translate them and dispatch them.

{% highlight c++ %}

if (WindowHandle)
{
    MSG message;
    while (GetMessage(&message, nullptr, 0, 0)
    {
        TranslateMessage(&message);
        DispatchMessage(&message);
    }
}
else 
{
    OutputDebugStringA("Cannot create Window\n");
}

{% endhighlight %}

* [GetMessage](https://msdn.microsoft.com/en-us/library/windows/desktop/ms644936(v=vs.85).aspx) takes 4 arguments, the first is a reference to the message struct, the second one is the window to listen to, if we pass ```nullptr``` to it, this will get the messages from the queue of our app in general. The last two arguments are low and high filters for messages. Passing ```0``` to both of them will not filter any message.

* [TranslateMessage](https://msdn.microsoft.com/en-us/library/windows/desktop/ms644955(v=vs.85).aspx) translate virtual-key messages to character messages. This is used to handle keyboard inputs, mapping keys to characters.

* [DispatchMessage](https://msdn.microsoft.com/en-us/library/windows/desktop/ms644934(v=vs.85).aspx) sends the message to the Windows Procedure for processing. We will discuss the Window Procedure right now.

<h3>Window Procedure</h3>

Window Procedure is a ```CALLBACK``` that gets called after ```DispatchMessage```. It processes the message dispatched. Let's add that above ```WinMain```.

{% highlight c++ %}

LRESULT CALLBACK
MainWindowCallback (HWND WindowHandle,
                    UINT message,
                    WPARAM wParam,
                    LPARAM lParam)
{

}

{% endhighlight %}

The Window Procedure returns an LRESULT, which signals the app how each message was handled.

* The first argument is the ```HANDLE``` to the window that we are processing messages for.

* The second argument is the message itself.

* The third argument is a WPARAM (unsigned int pointer) that contains data values, usually for sizes and positions (They can be extracted using ```HIWORD``` and ```LOWORD``` macros). This varies for each message.

* The last argument is a LPARAM (long void pointer) that contains custom values to be passed along with the message. The data can be read by ```reinterpret_cast``` this parameter to the type intended. This varies for each message.

Inside this function, we perform a ```switch``` on the ```message``` argument to determine which message it is.

{% highlight c++ %}

LRESULT Result = 0;
switch (message)
{
    default:
    {
        Result = DefWindowProc(WindowHandle, message, wParam, lParam);
    }
    break;
}

return Result;

{% endhighlight %}

A list of Window Messages can be found [here](https://msdn.microsoft.com/en-us/library/windows/desktop/ms644927(v=vs.85).aspx). A message can be system-defined or application-defined. For most basic applications, we only process messages with ```WM``` prefix.

Here in the ```default``` case, we just call ```DefWindowProc```, which is a default Window Procedure created by Windows to to default behaviors on messages.

Finally, we need to assign this function to ```WindowClass.lpfnWndProc```.

{% highlight c++ %}

WindowClass.lpfnWndProc = MainWindowCallback;

{% endhighlight %}

That's all you need to know about the essential of a Win32 app. I know there is a lot to take in here and I'm glad there are alternative frameworks to create applications for Windows now. But they all boil down to these concept, and sometimes it's nice to know what's going on under the hood. If you have any questions, don't hesitate to ask me on Facebook.
