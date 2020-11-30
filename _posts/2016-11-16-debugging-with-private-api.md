---
layout: post
tags: [ios-dev, objective-c, swift, lldb, uikit-dynamics]
date: 2016-11-16T08:47:33-08:00
title: Debugging with Private API
---

UIKit has a built in physics engine called UIKit Dynamics. It's how Apple has implemented quite a few of the animations in iOS. It's pretty powerful but it can also be a pain in the ass to debug. According to [Session 229](https://developer.apple.com/videos/play/wwdc2015/229/) from WWDC 2015, the supported way for debugging is to use private API on UIDynamicAnimator. There are some hacks for gaining access to private API inside of Objective-C — things like using NSInvocation — but they're not supported in Swift. That's ok though, because Apple actually wants us to use lldb to access this private API.

## Objective-C

Accessing the private API of an Objective-C stack frame via lldb is quite easy. All we need is the `expression` lldb command, `expr` for short.

```
(lldb) expr -- [animator setDebugEnabled:YES]
```

Assuming you have a variable named `animator` that points to a UIDynamicAnimator you're done. That's all there is to it. 🎉

## Swift

There is __no__ way[^1] to access private API via Swift. That's one of the points of Swift. _Safety!_ So, to access this private API via lldb, we're going to need to do a few things.

We're going to be in a Swift stack frame, so we need to let the `expr` command know to interpret the expression we give it as Objective-C. Luckily `expr` has a option to set the language.

```
(lldb) expr -l objc++ -- ...
```

_That is not a type-o. Yes, it's `objc++`. I don't know why, ask your friendly neckbeard._

Next we need to fill in the expression. If you were to try what we used in our Obj-C stack frame, you'd get the following response:

```
(lldb) expr -l objc++ -- [animator setDebugEnabled:YES]
error: use of undeclared identifier 'animator'
```

The symbol `animator` does not come across the the Obj-C/Swift bridge. 😭 To make this work, we first need a pointer to the animator.

```
(lldb) p animator
(UIDynamicAnimator?) $R0 = 0x00007fb8b4e0c8e0 {
	ObjectiveC.NSObject = {}
}
```

To use the pointer, we have to give lldb a bunch of type hints, but this incantation should work:

```
(lldb) expr -l objc++ -- (void)[(UIDynamicAnimator *)0x00007fb8b4e0c8e0 setDebugEnabled:YES]
```

And that's it! 🎉🙄

## My Solution

The above solutions are fine if you just want to trigger them once or twice. But I wanted an easier way. Luckily lldb has built in support for Python scripting. With it, we can create our own lldb commands. If you've never used lldb Python scripting, feel free to go [watch](https://vimeo.com/162786845) my Seattle Xcoders talk from last year.

Now that we're all caught up, here's my script in its entirety:

```python
#!/usr/bin/python

import lldb

def debugDynamicAnimator(debugger, command, exe_ctx, result, internal_dict):
	"""Uses the private setter `setDebugEnabled:` on a UIDynamicAnimator. Need to have a local variable named `a` that contains a pointer to a UIDynamicAnimator. Useful in breakpoint commands with 'Auto continue' turned on."""
	
	ci = debugger.GetCommandInterpreter()
	ro = lldb.SBCommandReturnObject()

	frame = exe_ctx.GetFrame()
	print >>result, "Got frame: {}".format(frame)
	
	animatorValue = frame.FindVariable("a")
	print >>result, "animator: {}".format(animatorValue)
	
	value = animatorValue.value
	print >>result, "value: {}".format(value)
	
	cmdString = "expr -l objc++ -- (void)[(UIDynamicAnimator *){} setDebugEnabled:1]".format(value)
	print >>result, "cmdString: {}".format(cmdString)
	
	ci.HandleCommand(cmdString, ro)	
	output = ro.GetOutput().strip()
	print >>result, "output: {}".format(output)

def __lldb_init_module(debugger, internal_dict):
	debugger.HandleCommand('command script add -f debugDynamicAnimator.debugDynamicAnimator dda')
	print 'The "dda" python command has been installed and is ready for use.'
```

Things to note:

- This requires that you have a local variable named `a`.
- I suggest adding a breakpoint directly after you've set `a`.
	- Add a _Debugger Command_ action and enter `dda` into the command field.
	- Turn on _Automatically continue after evaluating actions_
- Now you have a breakpoint you can toggle to enable/disable debugging of that UIDynamicAnimator.

Obviously this script could be better, but I have work to do. This works for me, feel free to use it and make it work better for you. Hopefully this can help someone not waste a half day fighting with UIDynamicAnimator debugging, like it cost me.

[^1]: No way that I know of. If you know, feel free to let me know on [Twitter](http://twiter.com/jakecarter/).