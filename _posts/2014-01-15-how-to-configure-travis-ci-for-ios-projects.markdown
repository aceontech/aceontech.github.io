---
layout: post
title:  "How to configure Travis-CI for iOS projects"
date:   2014-01-15 22:45:00
categories: objc ios
image: ios7-logo.png
words: 304
---

Travis-CI.org is hosted continuous integration for open-source projects. The service is tightly integrated with GitHub; it validates builds and runs tests as commits are pushed. Best of all, it's free (with a fair-use policy) for public repositories. Initially a Ruby-oriented service, support for more languages have been added steadily. This conveniently includes Objective-C and iOS.

While self-hosted solutions like Jenkins are kludgy at best - requiring worker machines, VMs and a bunch of tricky configuration - Travis-CI is totally hands-off. Configuration is done through a single file, which resides at the root of your GitHub repository, called **.travis.yml**.

After [hooking up your account](http://about.travis-ci.org/docs/user/getting-started/), it's surprisingly easy to configure Travis-CI to build your iOS projects & run any unit tests you may have. If you use [CocoaPods](http://cocoapods.org/) to manage dependencies, fear not, Travis knows how to handle this too.

The following **.travis.yml** builds all dependencies, the iOS project itself and runs all tests in the **iOS Simulator** (from [aceontech/ReactiveNSXMLParser](https://github.com/aceontech/ReactiveNSXMLParser/blob/master/.travis.yml)):

{% highlight yaml %}
language: objective-c

before_install:
    - gem install cocoapods --no-rdoc --no-ri --no-document --quiet
    - cd ReactiveNSXMLParserLib

# Path to the Xcode workspace.
#If you're using a project, use xcode_project: ...
xcode_workspace: ReactiveNSXMLParserLib.xcworkspace

# Define the Xcode scheme to use. Make sure the scheme is
 # "Shared" (in Xcode, menu Product > Scheme > Manage Schemes).
xcode_scheme: ReactiveNSXMLParserLib

# Instruct Travis to run tests in the simulator instead of a real device
xcode_sdk: iphonesimulator
{% endhighlight %}

Shell commands you want to run the before install process (phase before the actual build, used for setting up to the build config) go in the **before_install** block. Said commands need to be indented as per the YAML spec and prefixed by a dash. If your Xcode project doesn't reside in the root of the repository (like ReactiveNSXMLParserLib/), this is also where you would *cd* into the right directory.