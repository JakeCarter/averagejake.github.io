---
layout: post
aliases: [/post/114113936507/logical-ownership-of-uiviewcontroller]
date: 2015-03-20T02:38:42-00:00
title: Logical Ownership of UIViewController Presentations || Button, button, who’s got the button?
tags: [ios-dev, uiviewcontroller, presentation]
---

Prior to iOS 8, view controller presentation worked like so:

```swift
class ViewControllerA: UIViewController { 
	func presentTheThing() {
		let theThing = ViewControllerB()
		theThing.modalPresentationStyle = .SomeStyle
		self.presentViewController(theThing, animated: true, completion: nil)
	}
}
```

Here's my problem with this. Who's job is it to make sure that `theThing` is dismissible? Traditionally, if the `modalPresentationStyle` required a dismiss button (basically any presentation style other than popover requires a way to dismiss the `presentedViewController`), it was up to the presenting view controller to shove the view controller to be presented into a UINavigationController so that a dismiss button could be added to the top bar. This works fine when your presenting content view controllers, but quickly falls down in other situations. What if the view controller to be presented was already a UINavigationController? Or what if the view controller to be presented was a custom container. Who is responsible then for making sure the dismiss button is actually there? Before today, I would have argued that to comply with __Encapsulation__ (one of the major pillars of OO design) the view controller being presented should know how to style itself for any presentation. The alternative would be that the view controller doing the presenting would have to know _way too much_ about the view controller it's about present. With that mindset, the custom container (which has it's own navigation bar, by the way) should know when it should add a dismiss button to its navigation bar and when it shouldn't depending on how it's currently being presented. Unfortunately, there really isn't any API to figure that out.

Let's move on to iOS 8 for now...

With iOS 8 Apple added UIPresentationController into the mix. No longer are the `presentingViewController` and `presentedViewController` the only ones involved in presentation. Now, the view controller to be presented will vend a `presentationController` (or `popoverPresentationController` as appropriate) so that the presentation can participate in adaptability. The `presentationController` also allows a delegate to tweak how adaptability is handled. The two delegate callbacks that we care about for this discussion are:

```swift
// Allows the delegate to change the presentation style from its default adaptive alternitive
adaptivePresentationStyleForPresentationController(controller: UIPresentationController) -> UIModalPresentationStyle

// Allows the delegate to change the view controller hierarchy for a given adaptive presentation style, usually this means shoving the presentedViewController into a UINavigationController
presentationController(controller: UIPresentationController, viewControllerForAdaptivePresentationStyle style: UIModalPresentationStyle) -> UIViewController?
```

So, back to my custom container, still in the __Encapsulation__ mindset...

My initial thought was that my custom container could become the delegate of its `presentationController` and when it got the `presentationController(_,viewControllerForAdaptivePresentationStyle:)` call, instead of wrapping it up, just add or remove its own dismiss button from its own navigation bar. This fits in with encapsulation, but feels _very_ weird from a Cocoa design stand point. For one, when should `ViewControllerB` assign itself as its `presentationController.delegate`? 

Slight tangent...
The default `modalPresentationStyle` is `.FullScreen`. If you ask a view controller for its `presentationController` before you set its `modalPresentationStyle` you will get a full screen presentation controller. If you then set the `modalPresentationStyle` to something other than `.FullScreen` and ask for the `presentationController` again, you'll get back the same full screen presentation controller. Still need to file this one, but I've seen other presentation controller weirdness so I bet this gets closed "Works as intended."

So, back to `ViewControllerB` grabbing its `presentationController.delegate`... It needs to wait until its `modalPresentationStyle` has been set. So it could override the setter (or add a property observer in Swift) and do that then, but what if someone else grabs it later. It is a common design pattern, during the whole presentation dance, that the view controller doing the presenting will set itself as the delegate of the view controller to be presented if it wants to be. (Think about presenting a UIImagePickerController.) So it might also make sense that it should do the same with the `presentationController.delegate` of the view controller to be presented. Now I would need some way to guarantee that anytime `ViewControllerB` is presented, that it is its own `presentationController.delegate`.

One of the most important things I've learned over the years about Cocoa/CocoaTouch development... If it feels like you're fighting the frameworks, you're probably doing it wrong.

`ViewControllerA` is setting `ViewControllerB` up for presentation. There is currently no API for a `presentedViewController` to know how it is currently being presented. Now that adaptability has been introduced, you could set `.Popover` but actually be in a `.FullScreen` presentation, and the `presentedViewController` has no clear way to find that out. There is, however, API for a `presentationController` to notify its delegate about adaptability. And we already have a design pattern that sets president for `ViewControllerA` to become the delegate of `ViewControllerB.presentationController`.

So... "Button, button, who's got the button?"

My answer, `ViewControllerA` logically owns the presentation and it should subscribe as the `presentationController.delegate`. If it's presenting a content view controller, it should use `presentationController(_,viewControllerForAdaptivePresentationStyle:)` to wrap that content view controller in a UINavigationController when appropriate and it should push some logic down to any custom containers letting them know they are now being adapted. Yes, that still means that `ViewControllerA` needs to know _way too much_ about `ViewControllerB` but really, it's in the perfect position to do so.
