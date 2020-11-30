---
layout: post
aliases: [/post/101913953207/ios-simulator-arbitrary-scale]
date: 2014-11-06T03:13:28-00:00
title: iOS Simulator Arbitrary Scale
tags: [xcode, ios-simulator]
---

Mostly putting this here so I can find it in the future.

At home I tend to code on my MacBook Air. I love this machine, but sometime my 13" screen just isn't big enough to fit some of the iOS Simulator devices. Most notably the iPad or iPhone 6 Plus. There is a setting for scaling the window down, but the only options are 100%, 75% and 50%. Unfortunately those bigger devices are still too big even at 50%. I've been wishing there were a way to scale the window to an arbitrary value. There's nothing in the UI but I did [stumble](http://apple.stackexchange.com/questions/62757/how-do-i-resize-the-ios-simulator#comment170674_93313) upon a defaults write hack that seems to work for now.[^1]

    defaults write ~/Library/Preferences/com.apple.iphonesimulator SimulatorWindowLastScale "0.25"

Of course you can change the 0.25 to whatever you want. I'm sure you can get the simulator into some weird scales and visually break it, but something reasonable like 0.25 works great for me on my MacBook Air.

Hope this helps someone else too.

[^1]: This is currently working with the simulator that ships with Xcode 6.1 (6A1052d).