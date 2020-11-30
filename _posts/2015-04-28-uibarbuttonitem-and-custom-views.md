---
layout: post
aliases: [/post/117614151727/uibarbuttonitem-and-custom-views]
date: 2015-04-28T13:06:32-00:00
title: UIBarButtonItem and Custom Views
tags: [uibarbuttonitem, ios-dev]
---

First things first. UIBarButtonItems are __not__ views. They're more akin to model level objects. You create them and set up styling on them, but the framework converts their properties into actual UIView subclasses. And this is were the majority of problems that I have with UIBarButtonItem come from.

## My Most Recent Problem
Most recently I've had an issue with resizing a UIBarButtonItem's custom view when the UIToolbar it's in resizes into its compact size. (This happens in compact-compact environments.) Standard UIBarButtonItems — either system ones or those initialized with a title or image — will resize automatically to handle the smaller height of the UIToolbar. UIBarButtonItems initialized with a custom view do not seem to get any public/supported way to update they're drawing to take the smaller height into account.

## Workarounds I've Tried
My first thought was to use a custom subclass of UIView as the custom view and in that subclass, override `-layoutIfNeeded`. My thought was that if the UIToolbar that my custom view is being added to resizes, surely it would need to tell its subviews that they too need to re-layout. Unfortunately `-layoutIfNeeded` is not called when the UIToolbar resizes. 

My next guess was to see if my custom view were asked for `-sizeThatFits:` or even `-sizeToFit`. Again, nothing. Neither one were being called during rotation/UIToolbar resize.

Next I overrode all layout methods I could think of, including `-systemLayoutSizeFittingSize:`, `-setNeedsLayout` and `-layoutSubviews`. Out of all of the layout methods, the only one getting called on rotation/UIToolbar resize was `-layoutSubviews`. This isn't the worst thing ever, but it's not great either.

With `-layoutSubviews` I'm only able to do just that, layout my subviews, __not__ layout/resize myself[^1]. So, if I'm to make this work, I could create a container view to set as the custom view of the UIBarButtonItem. This container view could listen for the `-layoutSubviews` call, verify that its superview is a UIToolbar and check the toolbar's height to see how to layout its subviews. This seems odd to me, but I think I might be able to make something like this work.

## Spelunking with Hopper & LLDB
When I got this far I decided to crack open UIKit with [Hopper](http://www.hopperapp.com) to see if I could glean anything from the way that UIBarButtonItems handle this. Looking at the header for UIBarButtonItem you can see that there's a private flag for something called `isMinibarView`. After poking around for awhile I found that UIToolbar has a notion of entering 'mini-bar' mode when it's in a compact-compact environment. The way toolbars handles this is to ask standard UIBarButtonItems for a new view for the current toolbar. Standard UIBarButtonItems then return a new view at the appropriate size.[^2] Unfortunately the toolbar skips over UIBarButtonItems with custom views when switching between 'mini-bar' mode.

## Conclusion
I'm hoping I'm wrong about all of this and just missed something. If not, I'd really love some sort of public API so that custom views in UIBarButtonItems could play nicely in 'mini-bar' mode.[^3]

**Update:** [@zachwaugh](https://twitter.com/zachwaugh/status/593151837882195968) pointed out on twitter that my custom view could just override `-traitCollectionDidChange:` and check the vertical size class to handle the 'mini-bar' mode. I misunderstood traitCollections to return the traitCollection of the messaged view and assumed that, because my custom view's size does not change, that its traitCollection wouldn't either. I was wrong about that. (I've verified in a sample app.) I'll have to look into this as a means to fix the issue I was working on. Thank @zachwaugh!


[^1]: Please tell me if I'm wrong about this assumption, but as far as I know, a view should not resize itself during `-layoutSubviews`. It should do that during `-layoutIfNeeded`.
[^2]: You can verify this in the debugger. `po` a UIToolbars subviews in regular-compact, rotate then `po` again. You'll see that the subviews are actually new views, not just resized.
[^3]: I've filed rdar:///20624041 (UIBarButtonItems with custom views need way to handle compact toolbars). Please dup if you would find this useful.
