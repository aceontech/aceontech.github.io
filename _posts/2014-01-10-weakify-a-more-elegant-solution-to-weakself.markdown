---
layout: post
title:  "@weakify(self): A more elegant solution to weakSelf in Objective-C"
date:   2014-01-10 13:30:00
categories: objc ios
image: ios7-logo.png
words: 540
---

The introduction of Blocks and Automatic Reference Counting in Objective-C opened the way to drastically more concise code in the iOS developer community. Apple, too, is transitioning its APIs to this new model. And while ARC solves the majority of memory issues by obsoleting manual reference counting, the use of blocks often introduces a new gotcha: issues with capturing variables. Blocks clearly need to increase variables' retain counts during execution. Most of the time, this is exactly what you want; it keeps variables around long enough. However convenient, this also comes with a caveat specifically  pertaining to capturing *self*.

It may be unclear to new Objective-C developers that referencing *self* in a block actually creates a 'strong reference cycle to self' ([Apple](https://developer.apple.com/library/ios/documentation/cocoa/conceptual/ProgrammingWithObjectiveC/WorkingwithBlocks/WorkingwithBlocks.html#//apple_ref/doc/uid/TP40011210-CH8-SW16)). Because of the mechanics of ARC, Objective-C will prevent *self* from properly being deallocated, thus keeping it around indefinitely. Entire object graphs (UIViewControllers etc.) linger in memory this way and create memory leaks which accumulate over time. Needless to say, you will want to avoid this scenario at all cost.

## Enter weakSelf

The obvious solution to this problem is to define a weak reference to *self* before the block (let's call it *weakSelf*), and use that instead while calling into *self* from a block. For example:

{% highlight objc %}
// Create a weak reference to self
__weak typeof(self)weakSelf = self;
[self.context performBlock:^{
    // Create a strong reference to self, based on the previous weak reference.
    // This prevents a direct strong reference so we don't get into a retain cycle to self.
    // Also, it prevents self from becoming nil half-way, but still properly decrements the retain count
    // at the end of the block.
    __strong typeof(weakSelf)strongSelf = weakSelf;

    // Do something else

    NSError *error;
    [strongSelf.context save:&error];

    // Do something else
}];
{% endhighlight %}

Simple enough, but tedious and ugly. It's also error-prone,  because human-beings have the tendency of being forgetful. I know I am, all the time.

Adding insult to injury, it's also a regular [PITA](http://www.urbandictionary.com/define.php?term=PITA) to type over and over again. I recently came across one of [@mattt](https://twitter.com/mattt)'s GitHub repositories and was delighted to find the Xcode snippet for weakSelf, which I mapped to *defwself*. Quite the time-saver:

<blockquote class="twitter-tweet" lang="en"><p>&quot;Weak Self&quot; <a href="https://twitter.com/search?q=%23Xcode&amp;src=hash">#Xcode</a> snippet by <a href="https://twitter.com/mattt">@mattt</a> <a href="http://t.co/IU3XIOSYhZ">http://t.co/IU3XIOSYhZ</a> (installation instructions: <a href="http://t.co/0PeSjXd92b">http://t.co/0PeSjXd92b</a>)</p>&mdash; Alex Manarpies (@aceontech) <a href="https://twitter.com/aceontech/statuses/411149400053125121">December 12, 2013</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

## Hello @weakify(self)

It's awfully verbose, but a necessity. So just do it and thank me later. There is, however, one more optimization we can adopt: the **@weakify()** and **@strongify()** macros provided by the  [libextobjc](https://github.com/jspahrsummers/libextobjc) library. Check it out:

{% highlight objc %}
// Do stuff
@weakify(self)
[self.context performBlock:^{
    // Analog to strongSelf in previous code snippet.
    @strongify(self)

    // You can just reference self as you normally would. Hurray.
    NSError *error;
    [self.context save:&error];

    // Do something
}];
{% endhighlight %}

I love this solution because:

- It's easy to type
- It's easy to read and pick out with syntax highlighting
- It obviates the use of variables like *weakSelf* and *strongSelf*, because it effectively shadows *self*

## Adding it to your project (CocoaPods)

You can add the entire **libextobjc** via *CocoaPods* to your project, or choose to add just the **libextobjc/EXTScope** module.

If you're already using [ReactiveCocoa](https://github.com/ReactiveCocoa/ReactiveCocoa), the macros are already available to you, since ReactiveCocoa depends on *libextobjc*.

## A note on the use of macros

Sometimes it's okay to use macros.