---
layout: post
aliases: [/post/126378888342/followup-custom-presentation-controllers-and]
date: 2015-08-10T19:55:39-00:00
title: Followup – Custom Presentation Controllers and Adaptability
tags: [adaptability, ios-dev, presentation]
---

As the title suggests, this is a followup to my previous post about Custom Presentation Controllers and Adaptability. In that post I argued that the current API for custom presentation controllers is a little lacking when it comes to switching between custom and system provided presentations during the life of a single presentation. A friend of mine proposed a workaround that I’d like to explore. Feel free to grab the code and follow along: CustomAdaptablePresentation

## The Workaround

In this workaround, we’re always using a standard .Popover presentation and not actually ever assigning a custom presentation. Instead, we have the popover presentation controller’s delegate listen for adaptivePresentationStyleForPresentationController(_, traitCollection) and presentationController(_, willPresentWithAdaptiveStyle, transitionCoordinator:). All adaptivePresentationStyleForPresentationController... is tasked with is returning .Popover when in regular width and .OverFullScreen otherwise. The ‘brains’ of the workaround live in adaptivePresentationStyleForPresentationController... and the presented view controller.

With the workaround we’re never really switching to a custom presentation. We rely on the fact that a popover will adapt into a full screen presentation in a compact width environment. So ultimately we’re going to fake the half height presentation. Basically, we teach the presented view controller how to layout for a .Popover or for .FullScreen — or .OverFullScreen in our case because we want to ensure the presenting view stays in the hierarchy. To do this, we nest another view inside our presented view controller’s root view. This embedded view is where all of the normal UI goes. The outer, container view, is only used to position the embedded view.

Popover presentation is nothing special, as far as our nested views go. All we want is for the embedded view to be the same size as it’s container. For the adaptive state — the .OverFullScreen presentation — we allow the container view to be full screen, but we augment the frame of the embedded view to fake our half-height layout. For me, this was just adjusting the top constraint.

![](view-debugger.png)

Back to the 'brains’ of this workaround, what we do is define methods on the presented view controller that have it configure itself for popover or half-height. During adaptivePresentationStyleForPresentationController... we check the style being passed in and then call the appropriate configuration method.

## Downsides

Although this is a decent workaround, there are some pretty obvious downsides to this approach. The most egregious to me is the fact that this logic should clearly live at the presentation level and that the presented view controller should not have to worry about configuring itself. As it stands now, this code is not very reusable. Sure, I might be able to refactor this out to some container view controller that knew how to switch between popover and half-height, but if this lived in a presentation controller to begin with, we would get reuse for free. If you play around with this sample app, you’ll also notice other little things, like during the half-height presentation, buttons in the presenting view don’t get dimmed or desaturated. There might be something I can do to fix this, but I haven’t really looked into it.

## Conclusion

It’s a fun workaround but ultimately I don’t think we’re going to use it. I still feel like my radar is valid and I hope Apple can give is a way to switch between system provided presentations and custom ones.
