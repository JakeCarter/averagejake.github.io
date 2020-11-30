---
layout: post
tags: [public-speaking, swift, objective-c]
date: 2016-10-27T14:29:06-07:00
title: Starbucks Tech Talk
---

I was recently asked to speak at Starbucks for an internal thing they call Tech Talks. It was a great crowd and lots of fun. I gave a talk about adding Swift into an existing Objective-C application and framework. This post is mostly to share my slides and code.

## The Slides

The slides probably won't mean much without me yapping about them, but maybe you can gleam something from them.

[Starbucks Tech Talk Slides](starbucks-tech-talk.pdf)

## The Code

The code is broken up into two repos. The first is the [App Code](https://bitbucket.org/JakeCarter/destiny-nightfall-modifiers) and the second is the [Framework Code](https://bitbucket.org/JakeCarter/destiny-nightfall-info).

I've created tags — _demo0 - demo4_ — in the App repo that should step you through the demo. These tags should also include the updates to the submodule that pulls in the framework, so theoretically you should just be able to step through those tags, while updating the submodule, and you'll get everything you need. (I haven't tested that yet though.)

## The Script

This has to be one of the coolest parts. I used an open source app called [KeyGrip](https://github.com/rubbercitywizards/KeyGrip) to have a script on my iPhone. The app also made code blocks in script tappable and, via the KeyGrip Server, put the code blocks into the pasteboard on my Mac. This made 'live coding' super easy. I highly recommend checking out KeyGrip.

[Demo Script](starbucks-tech-talk.md)
