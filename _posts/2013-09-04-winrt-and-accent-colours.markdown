---
layout: post
title: WinRT and accent colours
---

So the blank WinRT XAML project supplies you with a nice dark scheme, with purple as an accent colour.

Of course, you want to differentiate, and give your app a different accent colour, right?

Well, here's how to do it.

Take a control you're using. Say, `ComboBox`. Then, simply search the internet for "ComboBox styles and templates".
The first MSDN link should have numerous brushes listed under "Theme resources".

In the case of `ComboBox`, we're provided with these for the dark theme:

{% highlight xml %}
<SolidColorBrush x:Key="ComboBoxArrowDisabledForegroundThemeBrush" Color="#66FFFFFF" />
<SolidColorBrush x:Key="ComboBoxArrowForegroundThemeBrush" Color="#FF000000" />
<SolidColorBrush x:Key="ComboBoxArrowPressedForegroundThemeBrush" Color="#FF000000" />   
<SolidColorBrush x:Key="ComboBoxBackgroundThemeBrush" Color="#CCFFFFFF" />
<SolidColorBrush x:Key="ComboBoxBorderThemeBrush" Color="#CCFFFFFF" />
<SolidColorBrush x:Key="ComboBoxDisabledBackgroundThemeBrush" Color="Transparent" />
<SolidColorBrush x:Key="ComboBoxDisabledBorderThemeBrush" Color="#66FFFFFF" />
<SolidColorBrush x:Key="ComboBoxDisabledForegroundThemeBrush" Color="#66FFFFFF" />
<SolidColorBrush x:Key="ComboBoxFocusedBackgroundThemeBrush" Color="White" />
<SolidColorBrush x:Key="ComboBoxFocusedBorderThemeBrush" Color="White" />
<SolidColorBrush x:Key="ComboBoxFocusedForegroundThemeBrush" Color="White" />
<SolidColorBrush x:Key="ComboBoxForegroundThemeBrush" Color="#FF000000" />
<SolidColorBrush x:Key="ComboBoxPointerOverBackgroundThemeBrush" Color="#DEFFFFFF" />
<SolidColorBrush x:Key="ComboBoxPointerOverBorderThemeBrush" Color="#DEFFFFFF" />
<SolidColorBrush x:Key="ComboBoxPopupBackgroundThemeBrush" Color="#FFFFFFFF" />
<SolidColorBrush x:Key="ComboBoxPopupBorderThemeBrush" Color="#FF212121" />
<SolidColorBrush x:Key="ComboBoxPopupForegroundThemeBrush" Color="#FF000000" />
<SolidColorBrush x:Key="ComboBoxPressedBackgroundThemeBrush" Color="#FFFFFFFF" />
<SolidColorBrush x:Key="ComboBoxPressedBorderThemeBrush" Color="#FFFFFFFF" />
<SolidColorBrush x:Key="ComboBoxPressedHighlightThemeBrush" Color="#FFD3D3D3" />
<SolidColorBrush x:Key="ComboBoxPressedForegroundThemeBrush" Color="#FF000000" />
<SolidColorBrush x:Key="ComboBoxSelectedBackgroundThemeBrush" Color="#FF4617B4" />
<SolidColorBrush x:Key="ComboBoxSelectedPointerOverBackgroundThemeBrush" Color="#FF5F37BE" />
{% endhighlight %}

What you want is to provide your own custom brushes that override the purple coloured brushes.
Here, it's these two:

{% highlight xml %}
<SolidColorBrush x:Key="ComboBoxSelectedBackgroundThemeBrush" Color="#FF4617B4" />
<SolidColorBrush x:Key="ComboBoxSelectedPointerOverBackgroundThemeBrush" Color="#FF5F37BE" />
{% endhighlight %}

But this isn't enough. As `ComboBox` also has `ComboBoxItem`s, you'll need to find and override the brushes for those as well.
As a lucky break, MSDN provides a link to "ComboBoxItem styles and templates" just below "ComboBox styles and templates" in the sidebar.
So in total:

{% highlight xml %}
<SolidColorBrush x:Key="ComboBoxSelectedBackgroundThemeBrush" Color="#FF4617B4" />
<SolidColorBrush x:Key="ComboBoxSelectedPointerOverBackgroundThemeBrush" Color="#FF5F37BE" />
<SolidColorBrush x:Key="ComboBoxItemSelectedBackgroundThemeBrush" Color="#FF4617B4" />
<SolidColorBrush x:Key="ComboBoxItemSelectedPointerOverBackgroundThemeBrush" Color="#FF5F37BE"/>
{% endhighlight %}

As you can tell by the naming of the brushes, you'll need to repeat this process for each and every control you use.
This can get quickly unwieldy, so you'll most likely want to assign your accent colours to a `<Color>` element, like so:

{% highlight xml %}
<Color x:Key="AppAccentColorPrimary">#FF4617B4</Color>
<Color x:Key="AppAccentColorLight">#FF5F37BE</Color>
<SolidColorBrush x:Key="ComboBoxItemSelectedBackgroundThemeBrush" Color="{StaticResource AppAccentColorPrimary}"/>
<SolidColorBrush x:Key="ComboBoxItemSelectedPointerOverBackgroundThemeBrush" Color="{StaticResource AppAccentColorLight}"/>
<SolidColorBrush x:Key="ComboBoxSelectedBackgroundThemeBrush" Color="{StaticResource AppAccentColorPrimary}"/>
<SolidColorBrush x:Key="ComboBoxSelectedPointerOverBackgroundThemeBrush" Color="{StaticResource AppAccentColorLight}"/>
{% endhighlight %}

Also of note, is that these instructions do *not* apply to a certain case -- text selection.
It will always remain purple, no matter what.

Do these requirements change in Windows 8.1?

**No**, save for one exception -- [text selection](http://msdn.microsoft.com/en-us/library/windows/apps/windows.ui.xaml.controls.textbox.selectionhighlightcolor.aspx).
