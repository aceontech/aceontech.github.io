---
layout: post
title:  "How to parse RSS/Podcast feeds with NodeJS"
date:   2013-11-27 17:00:00
categories: howto nodejs
image: nodejs-logo.png
words: 819
---

RSS is still very much alive and kicking, irrespective of Google severing the umbilical cord from [Google Reader](http://news.cnet.com/8301-1023_3-57591616-93/rip-google-reader/) in July. It still powers much of the web's news sites and delivers thousands of podcasts every day. But who wants to parse XML anymore these days? Here's how to get to parsing those feeds in minutes with [NodeJS](http://nodejs.org/) and the [feedparser](https://npmjs.org/package/feedparser) module.

**Impatient**? [Skip](#parsing) to the code.

## Prerequisites
I'm assuming you've already set up the following:

- NodeJS 0.10.x
- NPM
- A NodeJS app with a **package.json**

## Add dependency to feedparser
Because **feedparser** is available from the [NPM Registry](https://npmjs.org/package/feedparser), we can add it to our app by simply adding a line to our **package.json** (shortened for brevity):

{% highlight javascript %}
{
  "name": "YourApp",
  "version": "0.1.0",
  "dependencies": {
    "express": "3.4.4",

    // Include feedparser 0.16.3
    "feedparser": "0.16.3"
  }
}
{% endhighlight %}

Now run **npm install**:

{% highlight bash %}
$ cd YourApp/
$ npm install
{% endhighlight %}

**feedparser** is now includable via **require()**.

## A quick note on NodeJS Streams
With NodeJS Streams ([API documentation](http://nodejs.org/api/stream.html#stream_class_stream_readable)), inherently slow I/O channels (like HTTP or file-system access) are exposed as unified event-emitting APIs in the form of stream.Readable and stream.Writeable. The only thing you really need to know about this is that the stream APIs return data in **chunks**. You - as the developer - are responsible for handling these chunks properly, i.e. don't expect the 'chunk' or 'readable' events to mean 'done'.

You can find more information on NodeJS Streams on [Max Ogden](http://maxogden.com/)'s [blog](http://maxogden.com/node-streams.html).

## <a name="parsing"></a>The meat of the matter: getting & parsing
RSS/Podcast feeds usually live at a remote location and need to be fetched before they can be parsed into a more usable form. So, technically, we'll need one more tool to achieve this: **http.get()**. In summary, this will be the code flow:

1. HTTP **GET** the resource containing the RSS feed
2. Store feed **meta** data for later use
3. Add an episode object to the array every time a **readable** chunk comes in
4. Create a JSON object to return to the user

First off, **include** feedparser and http using **require()**, somewhere at the top of your file:

{% highlight javascript %}
var FeedParser = require('feedparser');
var http = require('http');
{% endhighlight %}

Next, call into **http** to **get()** the feed. In this example, let's use [Security Now's feed](http://twit.tv/sn) (one of my favorite podcasts):

{% highlight javascript %}
http.get('http://leoville.tv/podcasts/sn.xml', function(res) {
    // TODO: parse response
});
{% endhighlight %}

Feedparser takes in a readable stream. The **res** variable happens to be such a thing. How convenient! Let's **pipe** feedparser onto the stream and subscribe to the **error**, **meta**, **readable** and **end** events (in one fell swoop):

{% highlight javascript %}
var feedMeta;
var episodes = [];

http.get('http://leoville.tv/podcasts/sn.xml', function(res) {
    res.pipe(new FeedParser({}))
            .on('error', function(error){
                // TODO: Tell the user we just had a melt-down
            })
            .on('meta', function(meta){
                // Store the metadata for later use
                feedMeta = meta;
            })
            .on('readable', function(){
                var stream = this, item;
                while (item = stream.read()){
                    // Each 'readable' event will contain 1 article
                    // Add the article to the list of episodes
                    var ep = {
                        'title': item.title,
                        'mediaUrl': item.link,
                        'publicationDate': item.pubDate,
                    };
                    episodes.push(ep);
                }
            })
            .on('end', function(){
                var result = {
                    'feedName': feedMeta.title,
                    'feedArtist': feedMeta['itunes:author']['#'],
                    'website': feedMeta.link,
                    'albumArt': {
                        'url': feedMeta.image.url,
                        'width': parseInt(feedMeta['rss:image']['width']['#']),
                        'height': parseInt(feedMeta['rss:image']['height']['#'])
                    },
                    'episodes': episodes
                });

                // TODO: return 'result' to the user in some fashion
            });
});
{% endhighlight %}

And.. That's all there is to it, really. In essence, just a couple of lines of JavaScript were required to convert the RSS feed into a plain JSON object. This object can now be stored away into a database or the like. Not too shabby.

## GitHub
You can find [my working example](http://github.com/) on GitHub. It uses Express to route HTTP call to the correct controller. Try it out.

## Errata
As a front-end app developer I don't often get to dabble with server-side code. Why? Perhaps because it's a little too abstract, or just a wee bit daunting to approach without sufficient background. The JavaScript-based  [NodeJS](http://nodejs.org/), however, turns out to be a different beast altogether. It has been on my radar since its inception in 2009, but until recently, I never dug into it for real (I have merely used it once or twice to hack something together quickly). The ecosystem surrounding Node has become vibrant and mature, with the [NPM Registry](https://npmjs.org/) as the main vehicle for acquiring third-party/open source libraries. It currently serves almost 50,000 modules, and solicits over 7 million downloads per day. It's functionally comparable to [CocoaPods](http://cocoadocs.org/) for iOS, or [Bower](http://bower.io/) for front-end JavaScript (honorable mentions go to [NuGet](http://www.nuget.org/) and [Gradle](http://www.gradle.org/)).

In addition, the emergence of affordable cloud hosting for NodeJS apps makes it an attractive alternative to more established solutions based on Java, .NET and Ruby. I'm certainly keeping my eye on [Modulus](https://modulus.io/) and [Azure NodeJS](http://www.windowsazure.com/en-us/develop/nodejs/) for future Node hosting.