---
layout: post
tags: [ios-dev, presentation]
date: 2016-11-03T15:54:36-07:00
title: Timing Matters
---

Just a quick heads up, since this just bit me again.

Let's try a little experiment. Feel free to follow along in a Swift Playground.

Given the following code, solve for `vc.presentationController`.

```Swift
let vc = UIViewController()
vc.modalPresentationStyle = .popover
vc.presentationController
```

If you guessed `UIPopoverPresentationController` you are correct. Let's try another one.

```Swift
let vc = UIViewController()
vc.presentationController
```

This one is a tiny trick question. The real answer is `_UIFullscreenPresentationController` which is a private class, but I would have accepted "some type of full screen presentation controller" given that `.fullScreen` is the default `modalPresentationStyle`.

Ok, one last one — and the point of this blog post.

```Swift
let vc = UIViewController()
vc.presentationController // Obviously _UIFullscreenPresentationController
vc.modalPresentationStyle = .popover
vc.presentationController // ???
```

If you guessed `UIPopoverPresentationController` you would be dead wrong. The correct answer is `_UIFullscreenPresentationController`. `presentationController` is cached when created and the only way I know of to invalidate that is to present the view controller and then dismiss it. This is just a warning that timing matters when it comes to these things.

My workaround is to first make sure the view controller in question is currently presented — by verifying its presentingViewController is non-nil — before asking for its `presentationController`.

By the way, I have filed this as a radar. Works as intended, which is fine but I just thought more people might want to know how it works.
