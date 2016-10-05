---
layout: post
title: "Move Semantics"
description: "Explanation and demonstration of the most important C++11 feature, Move Semantic."
category: cplusplus
tags: [C++, modern C++, move, semantic]
---

<!-- Overview -->
<h3>Why?</h3>
A couple of weeks ago, I had the opportunity to talk with potential Microsoft interns and employee candidates at SFU's career fair. One particular student mentioned he knows C++11, which makes me curious since most SFU students take a course that uses "C with class"
 and say they know C++. To say (and to be able to prove) that you know C++11 is a big plus, since it shows that you invested time outside of school in a (relatively) new version of the language that changes the way C++ is written a lot.
 
Unfortunately, the student couldn't show much aside from some syntactic sugar that comes with C++11. And he was oblivious to "Move Semantic", the most important part of C++11.

<h3>So, what is move semantic?</h3>
To understand move semantic, there are a few concepts that need to be understood. Those are lvalue, rvalue and rvalue reference.
Let's start with ```lvalue```.
Let's say we have this code:

{% highlight c++ %}
int x = 10;
{% endhighlight %}

Here, ```x``` is a name we call an ```int``` with value ```10```. With this name, we can later read from and write to this instance of ```int```.
In other words, if we were to do something terrible to x (like killing it, or stealing it's value), someone (down the line) will be upset.
This is the idea of ```lvalue```, 'l' stands for "left", as in how ```x``` is on the left hand side of the statement ```int x = 10;```.
In short, an ```lvalue``` is a value that has a name and will be missed if something were to happen to it.

Next, onto ```rvalue```.
With the same code:

{% highlight c++ %}
int x = 10;
{% endhighlight %}

Now what do we call ```10```? That's right, nothing. It doesn't have a name, and there is no way to refer to it outside of this statement. And since there is no way to refer to it, it definitely won't be missed if something were to happen to it.
```10``` is an ```rvalue```, since it's on the right hand side of the statement ```int x = 10;```.

The example above might not shows what ```rvalue``` is in its entirety, so we have another example:

{% highlight c++ %}
int x = 10;
int y = x + 10;
{% endhighlight %}

In this example, a copy of ```x``` is made, then 10 was added to it, then give the resulting value to y. ```x + 10``` is an ```rvalue``` because there is no way to refer of that temporary instance of ```int```.

At this point, you might say "Who cares, it's int, it doesn't matter.", and that's true. So let's look at a more involved example.
Let's say we have an implementation of string that we handle the data ourselves.

{% highlight c++ %}
class myString
{
private:
  char* data;
}
{% endhighlight %}

This is a class that contains an array of character with a '\0' at the end, or a C-String.
To initialize an instance of myString, we use a constructor:

{% highlight c++ %}
myString(const char* inputString)
{
  size_t size = strlen(inputString) + 1;
  data = new char[size];
  memcpy(data, inputString, size);
}
{% endhighlight %}

By creating this constructor, we suppressed the compiler from generating the Destructor and Copy Constructor, so we have to write those as well (this is called the Rule of Three).

{% highlight c++ %}
// Destructor
~myString()
{
  if (data != nullptr)
    delete[] data;
}

// Copy constructor
myString(const myString& other)
{
  size_t size = strlen(other.data) + 1;
  data = new char[size];
  memcpy(data, other.data, size);
}
{% endhighlight %}

Now let's use this class:

{% highlight c++ %}
int main()
{
  myString first("first string ");
  myString second("second string ");
  return 0;
}
{% endhighlight %}

If we run this program, we'll see the constructor gets called twice, and destructor gets called twice. Nothing strange here.
Now what if we have an operator that adds 2 of these strings? Let's add this code to our class.

{% highlight c++ %}
  friend myString operator+(const myString& first, const myString& second);
{% endhighlight %}

This declares an operator ```+``` that takes 2 myString's by reference and return a new myString that contains the concatenated string. The code for this will be an exercise for the readers. I'll also include a quick implementation of this as a reference.
Let's use this operator:

{% highlight c++ %}
int main()
{
  myString first("first string ");
  myString second("second string ");
  
  myString combined(first + second);
  return 0;
}
{% endhighlight %}

Pop quiz: How many times the copy constructors were called for the line ```myString combined(first + second);```? And in turn, how many times do we allocate the string "first string second string "?

Answer: The copy constructor is called once, and we allocate the string twice, once in the constructor for the temporary object resulted from operator+, and once in the copy constructor that copies the temporary object to ```combined```. Right after the copy, the temporary object is destroyed, and it's copy of the string is deallocated.

We can see that there is an allocation of the string just for the purpose of copying it to another allocation and get destroyed right after. If this isn't just a string, but an expensive resource, it is really inefficient.
What if we can just steal the allocated string inside the temporary object? After all, it's an ```rvalue```, no one will have a problem with it's content being modified or stolen.

That's where move semantic comes in. To ```Move``` an object means to steal its guts instead of making a copy. So let's write a move constructor our little myString class:

{% highlight c++ %}
myString(myString&& other)
{
  data = other.data;
  other.data = nullptr;
}
{% endhighlight %}

There are a few things to say about the code above. First is ```myClass&&```. This is an "```rvalue reference``` to an object of type myString". What this means is telling the compiler to pick this contructor when we construct our object from a temporary instance, like in the case of ```combined``` above.
The body of this constructor is simple, we take the pointer to the character array from the temporary object and assign null to the temporary object's data pointer.

What does this mean, though? When we call ```myString combined(first + second);```, a temporary instance of myString is created, with the concatenated string. Then the move constructor is called, we steal that pointer to the concatenated string and give it to ```combined``` and give the temporary instance a nullptr. Right after, the temporary object is destroyed, it's destructor is called. 
And since the destructor doesn't do anything if the data pointer is null, we are left with a ```combined``` combined containing the string allocated for the temporary isntance.

Now how many allocations did we make? 1. That's right, we only allocate the string once, and the allocated string is ```moved``` from the temporary instance to ```combined```.

This behavior makes codes that pass big objects containing heap allocated data around much more efficient. There are no longer redundant allocations. 

Move semantic can be used in many other different ways though, for example ```unique_ptr```.

And that is the gist of Move Semantic.

Cheers,
Thai
