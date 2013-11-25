---
layout: post
title:  "How to find and delete duplicate files in Ubuntu/Linux"
date:   2012-04-27 15:18:32
categories: howto linux
image: ubuntu-logo.png
---

Massively duplicated files are oftentimes a problem with music and movie collections. Because hunting for dupes by hand is definitely not the way to go, you may want to look to command-line tools like **fdupes** for help.

**fdupes** is available via apt-get in Ubuntu, so install it first:

{% highlight bash %}
sudo apt-get install fdupes
{% endhighlight %}

This is the basic syntax for looking up duplicated files:

{% highlight bash %}
fdupes -r [target-directory]
{% endhighlight %}

How to delete all duplicates and generate a report at the same time:

{% highlight bash %}
fdupes -rdN [target-directory] > textfile.txt
{% endhighlight %}

A quick overview of what the options mean:

* -r recursive, traverse subdirectories
* -d delete, delete duplicates
* -N keep the first file, remove other (duplicate) files

Needless to say: use this with caution! Files will be deleted forever.

Find more information on **fdupes** [here](http://www.ubuntugeek.com/how-to-find-duplicate-copies-of-files-using-fdupes-in-ubuntu.html) (UbuntuGeek).