---
layout: post
title: Trying out ReactiveUI
---

I'm new to this whole Reactive Extensions thing, but I've recently decided to take a look into ReactiveUI.

Something nice about it is that you can use just parts of it with your existing MVVM app, so if you
don't get anything about this "Reactive" stuff you can just branch and mess around with it.

(And probably let it slowly assimilate your app into the ReactiveBorg, whereupon your previous MVVM framework gets eaten for lunch.)

Even though it's hit version 5 right now, there isn't really much in terms of docs.
If you're unfamiliar with Reactive Extensions, the best you can do really is just
*a lot* of searching everything everywhere (especially samples), flooding your browser with tabs.

Anyway, while there are samples around, the coolness of ReactiveUI is still only very slowly starting to click in my head.
The best example is probably [this one](https://gist.github.com/paulcbetts/810428) that's on its homepage.

From what I've gathered so far, ReactiveUI encourages you to keep viewmodel logic relating to how things
are glued together (e.g. Is this command enabled? What criteria makes it enabled? What does the command trigger?)
in the constructor, and *you should REALLY move logic out of property setters if you have any*.

I think this makes it so that this kind of logic is closer together in the constructor, instead
of being spread out (often unevenly) in your viewmodel.

Also, moving to reactiveness means a move away from events. You instead work with these kinds of LINQ/lambda style
things, which is probably a lot cooler, though I haven't yet grokked the benefits.

---

Here's an event-to-observable rewrite of some undo/redo code I have:

{% highlight c# %}
var canUndo = this.WhenAnyObservable(x => x._undoRedoManager.StateChanged)
    .Select(undoRedoManager => undoRedoManager.UndoStack.Count > 0);
Undo = new ReactiveCommand(canUndo);
Undo.Subscribe(_ => { _undoRedoManager.Undo(); HasUnsavedChanges = true; });
{% endhighlight %}

This required me to change `StateChanged` in my undo/redo manager class from an event to an `IObservable`.
I achieved this by changing the `EventHandler` to a `Subject<T>` (this class is from Rx), and

{% highlight c# %}
if (StateChanged != null)
    StateChanged(this, args);
{% endhighlight %}

to

{% highlight c# %}
_stateChanged.OnNext(arg);
{% endhighlight %}

Then, the `+=` hook changed to a `StateChanged.Subscribe(_ => ...)`.

What I was doing in the hook though, was changing `CanUndo`/`CanRedo` properties, which I had a button binding onto:

{% highlight c# %}
_undoRedoManager.StateChanged += (s, e) => {
    RaisePropertyChanged(() => CanUndo);
    RaisePropertyChanged(() => CanRedo);
};
{% endhighlight %}

And so that can be removed, along with those properties, leaving us with just needing an `ICommand` property:

{% highlight c# %}
var canUndo = this.WhenAnyObservable(x => x._undoRedoManager.StateChanged)
    .Select(undoRedoManager => undoRedoManager.UndoStack.Count > 0);
Undo = new ReactiveCommand(canUndo);
...
public ReactiveCommand Undo { get; private set; }
{% endhighlight %}

{% highlight xml %}
<Button Command="{Binding Undo}" AutomationProperties.Name="Undo" Style="{StaticResource UndoAppBarButtonStyle}"></Button>
{% endhighlight %}

---

Funnily enough, a thing that's made me unexpectedly happy is something that is so absolutely unimportant,
but it's `RaiseAndSetIfChanged`.

Basically, it lets you change this:

{% highlight c# %}
bool _hasUnsavedChanges;
public bool HasUnsavedChanges
{
    get { return _hasUnsavedChanges; }
    private set { _hasUnsavedChanges = value; RaisePropertyChanged(() => HasUnsavedChanges); }
}
{% endhighlight %}

Into this:

{% highlight c# %}
bool _hasUnsavedChanges;
public bool HasUnsavedChanges
{
    get { return _hasUnsavedChanges; }
    private set { this.RaiseAndSetIfChanged(ref _hasUnsavedChanges, value); }
}
{% endhighlight %}

Which is hardly any change. But it makes me happy. (Besides, anything that reduces boilerplate is nice, right?)

To use it with another MVVM framework you'll need to copy it [like so](https://gist.github.com/rzhw/6956761),
since its only been coded with ReactiveObject (the ReactiveUI base class for viewmodels) in mind.

---

As I mentioned earlier, samples are one of the more useful things you can look at to try and get an idea of what you can do with ReactiveUI.

Two good repos to look at are [Play for Windows](https://github.com/play/play-windows), and, well, [ReactiveUI.Samples](https://github.com/reactiveui/ReactiveUI.Samples).

Try out the search box up top on GitHub if you're trying to look at examples of how something is used;
for example, [I was looking for an example of `ObservableForProperty`](https://github.com/play/play-windows/search?q=observableforproperty&ref=cmdform).

It's fun stuff to poke at and figure out.