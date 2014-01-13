---
layout: post
title:  "ReactiveCocoa By Example: UIBarButtonItem & UIButton"
date:   2014-01-13 17:01:00
categories: objc ios reactivecocoa
image: ios7-logo.png
words: 211
---

Listening for tap events on UIBarButtonItem and UIButton generally requires you to follow the [Target-Action pattern as described by Apple](https://developer.apple.com/library/ios/documentation/general/conceptual/Devpedia-CocoaApp/TargetAction.html). As a result, you're declaring the button in one place, and the event handler in another. The [ReactiveCocoa framework](https://github.com/ReactiveCocoa/ReactiveCocoa) includes the RACCommand class, which can be attached to any UIButton or UIBarButtonItem. Like [BlocksKit](https://github.com/pandamonia/BlocksKit), this allows the event handler to be declared as an ObjC block, thus localizing it to the button declaration.

Consider the following example:

{% highlight objc %}
- (UIBarButtonItem *)downloadsButton
{
    if (_downloadsButton) {
        _downloadsButton = [[UIBarButtonItem alloc] initWithTitle:NSLocalizedString(@"Downloads", nil) style:UIBarButtonItemStylePlain target:self action:@selector(downloadsButtonTapped:)];
    }
    return _downloadsButton;
}

// .. more code here ..

- (void)downloadsButtonTapped:(UIBarButtonItem *)sender
{
    // Do something
}
{% endhighlight %}

By using ReactiveCocoa, this can be rewritten to:

{% highlight objc %}
- (UIBarButtonItem *)downloadsButton
{
    if (!_downloadsButton)
    {
        _downloadsButton = [[UIBarButtonItem alloc] init];
        _downloadsButton.title = NSLocalizedString(@"Downloads", nil);

        @weakify(self)
        _downloadsButton.rac_command = [[RACCommand alloc] initWithSignalBlock:^RACSignal *(id input) {
            @strongify(self)

            // EVENT HANDLER code here

            // An empty signal is a signal that completes immediately.
            return [RACSignal empty];
        }];
    }
    return _downloadsButton;
}
{% endhighlight %}

Dandy!

This clearly shouldn't be the sole reason for you to consider using ReactiveCocoa - it offers so much more. Check out ReactiveCocoa's [GitHub README](https://github.com/ReactiveCocoa/ReactiveCocoa) for a brief introduction.