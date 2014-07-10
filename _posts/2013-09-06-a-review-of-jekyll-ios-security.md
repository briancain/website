---
layout: post
title: "A Review of Jekyll iOS Security"
modified: 2013-09-06
tags: [security,ios,app]
image:
  feature: 
  credit: 
  creditlink: 
comments: true
share: 
---

Earlier this week I did a review of the paper "Jekyll on iOS: When Benign Apps Become Evil"; A paper from USENIX 22 presented a couple weeks back. The main focus of the paper is to present a model that allows malicious attackers to design a seemingly innocent iOS app that can accomplish several powerful attacks including taking photos, sending SMS/Email/Tweets, exploiting the OS kernel, and a _Trampoline Attack_ that takes advantage of Safaris extra privileges. They call this model or app Jekyll because of how the app behaves. If you're familiar with the old story [_Strange Case of Dr Jekyll and Mr Hyde_](https://en.wikipedia.org/wiki/Strange_Case_of_Dr_Jekyll_and_Mr_Hyde), you should have a good understanding of the basic idea.

I found this paper particularly interesting because it seems like iOS exploits are rarely talked about. This at least seems to be true in our weekly Kansas State security reading group. Often our mobile security discussions are about Android. I don't believe it's because iOS is that much more secure over Android, but rather a lot of process behind iOS isn't as open as it should be. This paper brings several potential issues with the App Signing process, the way apps are sandboxed on iOS, and other issues related to a few security models iOS implements such as Data Execution Prevention (DEP) and Address Space Layout Randomization. Both of these two techniques aim to prevent third-party apps from becoming malicious. What's interesting about this paper is the researchers present a way to get around these security measures and implement a pretty powerful app. They also got it published on the App store with the Apple seal of approval.

The full paper can be found [here](https://www.usenix.org/system/files/conference/usenixsecurity13/sec13-paper_wang-updated-8-23-13.pdf). And the official USENIX page can be found [here](https://www.usenix.org/conference/usenixsecurity13/jekyll-ios-when-benign-apps-become-evil) (which includes a video of the full talk from the conference).

Below are the slides I used for my talk at K-State about the paper.

<iframe src="https://docs.google.com/presentation/d/1EFOpoGHbaJnwnt-1A4qbCI7-81H47AYXXKqw-s7Us9g/embed?start=false&loop=false&delayms=3000" frameborder="0" width="480" height="389" allowfullscreen="true" mozallowfullscreen="true" webkitallowfullscreen="true"></iframe>
