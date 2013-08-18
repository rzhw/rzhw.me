---
layout: post
title: How const works in C++, from a beginner's perspective
---

Coming from languages like C# and Java where `const` is strictly something that comes before a type when defining a variable,
it's been a bit odd to try to wrap my head around it. Here I'll try and jot down a few things to note.
I'll assume you definitely know the basics of variables, and know at least a bit about C pointers.

## The `const` keyword

Adding `const` before a primitive or user-defined type, like many other languages, creates a constant.

{% highlight c++ %}
int a = 1;
const int b = 2;
a = 3; // works
b = 4; // compiler error
{% endhighlight %}

So is a `const int *` a constant `int *`?

{% highlight c++ %}
int c = 1;
const int *d = &c;
d = NULL; // works
{% endhighlight %}

I could assign to it, so it's clearly not! How about `int const *`?

{% highlight c++ %}
int e = 1;
int const *f = &e;
f = NULL; // works
{% endhighlight %}

Same thing!

If you didn't know, like in C, the `const` can come on the *right* side of the type as well.

{% highlight c++ %}
int const g = 1;
g = 2; // compiler error
{% endhighlight %}

It helps to think of the above examples as something like `(const int) *` and `(int const) *`, then.

And that means...

{% highlight c++ %}
int h = 1;
const int *i = &h;
*i = 2; // compiler error
{% endhighlight %}

... the pointee is being treated as a `const int`.

What if you wanted a constant pointer? The `const` comes *after* the `*`.

{% highlight c++ %}
int j = 1;
int * const k = &j;
*k = 2; // j is now 2
k = NULL; // compiler error
{% endhighlight %}

What would a constant pointer to a constant int be, then? You should be able to come up with it.

{% highlight c++ %}
int l = 1;
const int * const m = &l;
*m = 2; // compiler error
m = NULL; // compiler error
{% endhighlight %}

**TL;DR**:

- `const int` == `int const`
- `const int *` == `int const *` == pointer to a const int
- `int * const` == const pointer to an int
- `const int * const` == `int const * const` == const pointer to a const int
- If a `const` is on the right of a `*`, it's describing the pointer
- If a `const` is on the left of a `*`, it's describing the pointee

## Casting your `const` worries away with `const_cast`

Apart from the usual implicit and explicit casting from the likes of C,
C++ also comes with some casting operators `static_cast`, `dynamic_cast`, `reinterpret_cast`, `const_cast`.

`const_cast` basically lets you manipulate the `const`ness of a variable.

Say we know a `const int *` is definitely pointing to a non-const `int`.
That's fine.

{% highlight c++ %}
int n = 1;
const int *o = &n;

int *p = const_cast<int *>(o);
*p = 2; // works, n == 2
{% endhighlight %}

But if it's not, your code *will* compile, but C++ doesn't define what will happen.
So your fate is left up to the compiler.

{% highlight c++ %}
const int q = 1;
const int *r = &q;

int *s = const_cast<int *>(r);
*s = 2; // undefined behaviour
{% endhighlight %}

It's useful in situations where you have a function which takes in e.g. `char *`,
but doesn't actually modify anything.

{% highlight c++ %}
void print(char *str) {
    cout << str << endl;
}
int main() {
    const char *str = "non stl string";
    print(const_cast<char *>(str));
}
{% endhighlight %}

To finish on a related note, you can see that it's useful to mark your functions as taking in `const` pointers where possible.