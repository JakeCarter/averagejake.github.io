---
layout: post
title: Attach to Process by PID or Name...
tags: [xcode, lldb]
---

Some how Xcode's "Attach to Process by PID or Name.." came up recently in conversation and several of my developer friends had never heard about it. I'd like to quickly walk through my favorite use case, auto attaching to a process when it launches.

So, here's the problem... Let's say you want to debug some code that only executes from a cold launch of your application and only when some arguments are being passed in. One concrete example would be an iOS app's `application(_, didFinishLaunchingWithOptions:) -> Bool` where another application is passing you a file to open. Perhaps a user tapped on one of your files in Mail and wants to open the file in your application.

One way to make sure the debugger is attached by the time that method is called is by using Xcode's "Attach to Process by PID or Name..." menu option found under the "Debug" menu. If you read the text in that dialog, you'll see that it states "If you enter a process name that isn't currently running, the debugger will wait for that process to start..." This is the feature we care about. Enter your process name into the field and hit attach, then perform whatever action you need to cause your application to launch.[^1]

I've had much better success with this while debugging on actual hardware instead of the simulator, but that's pretty much it. Hopefully this helps someone else with this sort of problem.


[^1]: Keep in mind, that the debugger will be watching for the process on whatever device/simulator you have chosen as the destination in Xcode's [Scheme menu](http://help.apple.com/xcode/mac/8.3/#/devdc0193470).
