---
layout: post
title: lvalues and rvalues, postfix and prefix
---

The concept of lvalues and rvalues is described in quite complex ways, but it essentially boils down to something simple:

{% highlight c %}
int two = 1 + 1;
{% endhighlight %}

On the left we have the variable `two`, and on the right we have the value `1 + 1`.

You can define lvalue as an object's identity or address,
while rvalue as an object's value.

It can help to think of lvalues as a subset of rvalues.

Everything has to be a value (rvalue), but some have identities (lvalue), whether semantic or locational.

Here's an example; if you've ever tried something like modifying the return value of a function,
you would have realised it didn't work.

{% highlight c %}
int one() {
    return 1;
}
one() += 1;
{% endhighlight %}

That's because `one()` returns the *value* of 1, and not a *reference* to a variable that contains 1.

So now we have a way of describing that; `one()` is an rvalue (and not an lvalue), and can't be modified.

## Postfix and prefix operators

If you're like me, you might have been using the beloved `i++` for quite a while
without caring about its fine details, and perhaps even completely ignored `++i`.

Personally, I think it's best left as something used on its own,
since it's a bit more mentally taxing to see it being used in assignment and other
contexes, but as we're talking lvalues and rvalues, let's look a bit closer.

This'll be in a C++ context. That is, function arguments are normally pass-by-value,
unless we use a reference type e.g. `int &`, to indicate a *reference*, not a value, is being passed.

### Postfix

{% highlight c %}
int a = 0;
int b = 0;
a = b++;
{% endhighlight %}

This is roughly equivalent to:

{% highlight c++ %}
int postfix(int &value) {
    int tempcopy = value;
    value = value + 1;
    return tempcopy;
}
int a = 0;
int b = 0;
a = postfix(b); // a == 0 && b == 1
{% endhighlight %}

That is, the postfix operator assigns the *original* value of `b` as well as incrementing `b`.
Note also that the *value* of `b` is getting assigned. That makes the postfix operator an rvalue.

So this is invalid:

{% highlight c++ %}
postfix(b) = 5;
{% endhighlight %}

### Prefix

{% highlight c %}
int a = 0;
int b = 0;
a = ++b;
{% endhighlight %}

This is roughly equivalent in C++ to:

{% highlight c++ %}
int& prefix(int &value) {
    value = value + 1;
    return value;
}
int a = 0;
int b = 0;
a = prefix(b); // a == 1 && b == 1
{% endhighlight %}

That is, the prefix operator modifies `b` and returns `b`.
Note also that it returns a *reference* to `b`.

So this *is* valid:

{% highlight c++ %}
prefix(b) = 5;
{% endhighlight %}

Although I'm not sure why you'd want to do that.

*However*, this is only in a C++ context.
This is *invalid* in C# and Java, as the prefix operator for them both return the incremented *value*.

Doing so in a C++ way would look mostly the same, except we return an `int` instead of an `int &`.

{% highlight c++ %}
int prefix2(int &value) {
    value = value + 1;
    return value;
}
int a = 0;
int b = 0;
a = prefix2(b); // a == 1 && b == 1
{% endhighlight %}

So this would be invalid:

{% highlight c++ %}
prefix2(b) = 5;
{% endhighlight %}

## TL;DR

- All lvalues are rvalues
- A literal value (i.e. not a variable) is not an lvalue
- Postfix (`i++`) assigns the *original* value and increments
- Prefix (`++i`) increments and assigns the *new* value
- Postfix is not an lvalue
- Prefix is an lvalue in C++, and **is not an lvalue** in C# and Java

## Further reading

- [Value (computer science) - Wikipedia](https://en.wikipedia.org/wiki/Value_%28computer_science%29)
- [What is the difference between prefix and postfix operators? - Stack Overflow](http://stackoverflow.com/questions/7031326/what-is-the-difference-between-prefix-and-postfix-operators)
- [++i operator difference in C# and C++ - Stack Overflow](http://stackoverflow.com/questions/9516926/i-operator-difference-in-c-sharp-and-c)