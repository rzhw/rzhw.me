---
layout: post
title: Learning programming languages by porting
---

Last weekend, I came across a tiny C++ library/function that decodes PNG files into a pixel array,
called [picoPNG](http://lodev.org/lodepng/picopng.cpp).

Recently, I've also been thinking about experimenting with Lua a little bit, which I haven't used before.
If you've seen my recent posts, you also could guess that I've been picking up C++ over the past month or so.

So it goes that today I've tried to port that function over to Lua.

Unluckily, porting ~500 lines of code between two not-very-similar languages isn't exactly a breeze.

Luckily, even with only about 1/3rd of it completed,
it's already been a good opportunity for me to learn some more about STL
(since picoPNG makes heavy use of vectors),
and some of the (fun) idiosyncrasies of Lua.

For example, did you know in Lua:

- Instead of arrays, lists, key/value types, or anything of the sort,
  there's an oddly interesting mish-mash of them all, called a table?
    * (A better analogy would be combining JavaScript arrays and objects)
- Numerical indexing of a table starts at 1, not 0?
    * But you can still fake it, e.g. `arr = {[0]=1,2,3}` (albeit `#arr` for getting the table size won't work)
- Classes aren't a thing, but you can [fake them](http://lua-users.org/wiki/SimpleLuaClasses),
  with a `:` symbol in your functions, and operator overloading?
- From just a syntactic standpoint, it reminds me of Ruby! (Yay!)
- No simple bitwise operators (aw!)... instead, you need the [bit32 library](http://www.lua.org/manual/5.2/manual.html#6.7)

I have no list for C++ as well sadly, since it's just some more bits and pieces of STL (learned a new constructor and a few methods),
and solidification of core language things (e.g. references, pass by value vs reference etc.),
but the extra knowledge doesn't hurt, right?

You don't have to choose two languages that you both don't know very well; one works just fine too.
I've attempted some simple porting before where I was already familiar with one language, and it still helped a lot.

So if you're interested in figuring out a language, and can find something simple to port, give it a shot.
It's definitely one of the more fun ways to learn. Beats an hour of reading and hoping you've absorbed something.