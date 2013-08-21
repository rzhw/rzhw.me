---
layout: post
title: "A quick look at C++ STL for C# users, part 1: where's my List<T>?"
---

The moment you realise you've moved on from mere arrays to powerful generics to hold your data,
with the dizzying array of methods at your disposal, is definitely quite a special one.

And if you happen to have ventured into a low-level procedural language like C for a bit,
and tried your hand at creating abstract data types, like linked lists, from the ground up there,
you *really* get to appreciate all that hard work that goes behind the scenes
so that you've got all these fantastic abstractions for you.

Luckily, when it comes to collections, it's almost like dealing with them when you're going from C# to Java
(or the other way around). If you haven't done anything of the sort, basically it'll take you
a little time to get used to a different set of methods. But a huge chunk of the way you think and reason
about collections can thankfully be reused.

## List basics

Let's look at a basic `List<T>`.

{% highlight c# %}
using System.Collections;
using System.Linq;
var list = new List<int>();
list.Add(1);
list.Add(2);
list.Add(4);
{% endhighlight %}

Instead with C++, you have a `vector<T>`. Yep, looks like a generic, but they're called containers.
(Note that unlike C#, user-defined types, including `vector`, will have their
default constructor called if not assigned to a value.)

{% highlight c++ %}
using namespace std;
vector<int> vec;
vec.push_back(1);
vec.push_back(2);
vec.push_back(4);
{% endhighlight %}

Similarly to `List`s, `vector`s are backed by an array that gets reallocated when more space is needed.
So if you need to reason more about the implementation, they're quite similar.
For example, both expand at most `log n` times for inserting `n` elements.

And as you can see, the basics are covered.

{% highlight c# %}
Console.WriteLine(list.Count); // 3
Console.WriteLine(list.First()); // 1
Console.WriteLine(list.Last()); // 4
Console.WriteLine(list[0] + list[2]); // 5
{% endhighlight %}

{% highlight c++ %}
cout << vec.size() << endl; // 3
cout << vec.front() << endl; // 1
cout << vec.back() << endl; // 4
cout << vec[0] + vec[2] << endl; // 5
{% endhighlight %}

There's not much fun in name dropping methods, so I'll leave it there.

Other list implementations are also provided by STL,
including a linked list (aptly named `list<t>`), though not as many as C#.
C++11 also introduced some more containers and methods, though that info probably isn't
of much use yet until you familiarise yourself a little bit further with the language.

If you'd like to dig around more, cplusplus.com has a [good table][containers]
showing you what there is, what's usable for each container, and what C++11 adds.

## End of part 1

More to come soon. I'm intending to write about iterators and maps.

## Useful links

- [Containers - C++ Reference][containers]
- [vector - C++ Reference][vectors]

  [containers]: http://www.cplusplus.com/reference/stl/
  [vectors]: http://www.cplusplus.com/reference/vector/vector/