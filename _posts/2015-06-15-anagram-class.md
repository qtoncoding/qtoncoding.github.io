---
layout: post
title: "Find Anagram Classes In A Dictionary"
description: "My take on the problem of finding all anagrams in a dictionary"
category: cplusplus
tags: [C++, modern C++, anagram, dictionary]
---

<!-- Overview -->
<h3>Story Time!</h3>

When I was doing my Foundry Internship at Microsoft, we interns used to ask each other coding questions, just for kicks. I was asked a question along this line:

"Find all anagrams in a dictionary"

Then I was really focused on reducing problem size and trying to find ways to reduce works that has to be done, like find all words with the same length, check if they are anagrams...
In general, that approach will never work, because the time complexity is proportional to the factorial of the length of each word, and that has to apply to all words in the dictionary. A solution with this mindset would take hours to run on a large input.

But it was all fun and games, I didn't give it much thought until recently. When I was working with hash tables, I got the idea that maybe all words with the same characters (anagrams) can have the same hash, that way if I look up a hash I can find all words that belong to the same anagram class!
It took me 2 hours to develop the idea into a working solution, and 15 minutes to code it out. Today's post will walk you through my solution for this problem.

<h3>Implementation</h3>
For testing and profiling purpose, I'm using 2 input files, 1 is one that I typed in manually, containing mostly anagrams, and the other is the Mieliestronk English word list (which can be found [here](http://www.mieliestronk.com/corncob_lowercase.txt)).

And since I love C++, let's use a lot of cool modern C++ features in this implementation!

{% highlight c++ %}
#include <string>
#include <vector>
#include <fstream>
#include <algorithm>
using namespace std;
{% endhighlight %}

These are the headers I'll include in this project. ```<string>``` for standard library string, ```<vector>``` for standard array, ```<fstream>``` to handle file I/O and ```<algorithm>``` to use built-in algorithms of C++.
And for convenience, I use the namespace ```std```.


First, we read the dictionary into a vector of strings.

{% highlight c++ %}
// Read file into vector
ifstream inputFile("input.txt");
vector<string> dictionary;
string line;
while (getline(inputFile, line))
{
    dictionary.push_back(line);
}
inputFile.close();
{% endhighlight %}

First, I open the file in a file input stream with ```ifstream```. Just provide the file path and we are good to go!
Then I create a vector of strings, read every line of the file until EOF (end of file) is reached, add the line to the dictionary.
Finally, close the input stream.

Next up, we do a bit of preprocessing on the dictionary. For each word, we find a signature for it by sorting all characters in the word, using this sorted string as the "hash" for words with the same characters.

{% highlight c++ %}
// process input
vector<pair<string, string>> signedDictionary;
for (auto & word : dictionary)
{
    string sortedWord = word;
    sort(sortedWord.begin(), sortedWord.end());
    auto entry = make_pair(sortedWord, word);
    signedDictionary.push_back(entry);
}
{% endhighlight %}

Here I use a standard templated library ```pair``` to store the signature and the word itself. The pairs are then put into a dictionary called ```signedDictionary```. 
The ```for``` loop is a ranged for-loop, using ```auto``` type deduction, referencing each word in the dictionary (so we don't have to copy it everytime).
A duplicate is made for each word, then I apply the standard sorting algorithm on the word, passing the ```begin``` and ```end``` iterators in. The standard sort is an in place sort, that's why we have to make a duplicate of our word.
Finally an entry for the signed dictionary is made by ```make_pair```.

Next step is to sort this signed dictionary, so all words that belong to the same anagram class (in other words, are anagrams of each other) are in one place.

{% highlight c++ %}
// Sort signed dictionary
sort(signedDictionary.begin(), 
     signedDictionary.end(), 
     [](pair<string, string> left, pair<string, string> right) 
        {
            if (left.first == right.first)
                return left.second < right.second;
            return left.first < right.first;
        });
{% endhighlight %}

Here you can see the standard sort algorithm comes into play again, but this time it has a different form. Previously, since I was just sorting characters in a string, I let C++ use its default comparator. This time, however, since we are sorting pairs of strings, I have to provide my own comparator.
One cool feature of modern C++ is lambda, and here you can see I pass a lambda as the 3rd argument to ```sort```. This lambda takes 2 arguments, each is a pair of string string. It will then compare each pair by comparing the signature first, then if the signature are the same, it compares the actual word. In this case, standard string has comparison operator overloaded, so I don't have to roll my own string comparison.

Finally, we output our results. I want all words that are anagrams to be on the same line, each line will be an anagram class.

{% highlight c++ %}
// Output result
ofstream outputFile("output.txt");
string prevSign;
int lineNumber = 0;
for (auto & entry : signedDictionary)
{
    // If new group, go to next line
    if (entry.first != prevSign && lineNumber > 0)
    {
        outputFile << '\n';
    }
    // print out current word
    prevSign = entry.first;
    ++lineNumber;
    outputFile << entry.second << " ";
}
outputFile.close();
{% endhighlight %}

First I open a file for writing using ```ofstream```. Then I create a string called ```prevSign``` to keep track of which anagram class we are currently in, and an int called ```lineNumber```, which isn't at all that useful, except for letting me bypass the first line gracefully, and possibly keep track of number of anagram class, in case I want to do so.

For every entry in the signed dictionary, if we encounter a new anagram class, print out a new line to the output file. Then assign the new anagram signature to ```prevSign```, increase lineNumber and print the actual word out to output file, followed by a space.

<h3>Performance</h3>
This program was compiled with Visual Studio 2015 RC on my computer, when tested with the smaller input file of 40 words/lines, it took 14ms to finish. 
When tested with a larger input file, the word list from the Internet, of 58110 words/lines, it took 433ms to finish. Personally, I think that's pretty fast.

There we have it, a nice simple solution to a rather daunting problem at first. Hope you find this useful!

Cheers,
Thai

