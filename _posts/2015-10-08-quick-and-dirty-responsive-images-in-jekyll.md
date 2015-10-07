---
layout: post
title: Quick and dirty responsive images in Jekyll
---

One of the images I included in my previous blog post, [the one about rice roll shakers]({% post_url 2015-10-07-what-on-earth-is-a-rice-roll-shaker %}), was a screenshot taken on a Retina MacBook Pro.

I've previously used [retina.js](https://imulus.github.io/retinajs/) for responsive images. It automatically checks for and grabs `@2x` labelled versions of your images, without needing any extra markup. Back when I still followed webdev closely, there wasn't really any standardised way of doing things, and everyone was getting super giddy over the rMBPs. Gotta jump on the hype train!

So that made it a nice (but lazy) solution. Thing is: 1. It's JavaScript. 2. You were still sucking down both the @1x *and* @2x versions of your images. That's not as bad for icons and so on, but that's pretty bad for big images.

With that screenshot at 2782x1770, and not being entirely composed of simple shapes, it weighs in at about 626 KB, and that's *after* being optimised[^imageoptim]. Making a @1x version is 307 KB after optimisation, so if you're going to download the @2x version on top of that, that's an extra fun hit of **933 KB** total for a single image.

Luckily, looks like good progress has been happening with responsive image support. A quick search shows [srcset has good support](http://www.webdesignerdepot.com/2015/08/the-state-of-responsive-images/) these days, so off to implement it!

Except, you know, saving time is good, and having more markup and less content in your Markdown posts doesn't look nice. What to do? A quick search for Jekyll srcset brings me to the aptly named `jekyll-srcset`, which gets used like this:

{% raw %}

    {% image_tag src="/image.png" width="100" %}

And generates something like this:

    <img src="/image-100x50.png" srcset="/image-100x50.png 1x, /image-200x100.png 2x, /image-300x150.png 3x" width="100">

Here's the problem: Using Jekyll with GitHub pages, you can't use any plugins [except for a few](https://help.github.com/articles/using-jekyll-plugins-with-github-pages/)[^security]. So what to do?

Thankfully, you can get something sort of like custom Liquid tags with [includes](http://jekyllrb.com/docs/templates/#includes). With that, I whipped up a quick solution, with a mindset inspired by retina.js (that is, having your high-res images have @2x appended to their filename).

Here's my include, which I've dropped in as `_includes/retinaimg.html`:

    <img srcset="{{ include.src | replace: '.jpg', '@2x.jpg' | replace: '.png', '@2x.png' }} 2x" src="{{ include.src }}" alt="{{ include.alt }}" />

And used like so:

    {% include retinaimg.html src="/blog/images/daisomangoes.png" alt="Picture of Daiso dried mangoes" %}

{% endraw %}

Not bad for a quick hack, nicer to type out than the full thing when I use high-res images more, and while not as nice to look at as the `![]()` syntax, it's certainly a good thing to keep data usage lower these days when we've all got these phones with data caps[^mobilefirst].

If I wanted to get more serious about saving data, I could definitely do more work with how I attach images. The screenshot in question @1x is 1391x885, and my blog at time of writing has a default post display of 800px wide. Resizing to 800x509 and optimising, I get down to just 138 KB.

It's a tradeoff of convenience versus bandwidth concerns.

  [^imageoptim]: Seriously, optimise your images. I shaved down to 626 KB from the 899 KB png that OS X gave me. I've been giving [ImageOptim](https://imageoptim.com/) a try and it's a joy to use, though be careful about how it overwrites the images you feed it.
  [^security]: Naturally, GitHub Pages isn't for arbitrary code execution.
  [^mobilefirst]: I can't bring myself to say "mobile-first" right now.
