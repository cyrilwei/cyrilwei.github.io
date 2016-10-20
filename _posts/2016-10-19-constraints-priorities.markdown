---
layout: post
title:  "iPad Multitasking, Split View, Auto Layout, and Constraint Priorities"
date:   2016/10/18 20:41:00 +0800
categories: iOS
---

Yesterday, a colleague asked me: In the iPad landscape mode, how to tell whether the app is in full screen mode or 2/3 split view mode by `size class`.

That is to say, how to distinguish the two colorized situations in the picture below.

<img src="/assets/images/2016/10/ipadsplitview/multitasking_size_classes.png" style="width: 720px">

(The picture is from Apple's [Adopting Multitasking Enhancements on iPad][1])

Unfortunately, according to Apple's documents, the `size class` in those situations are both `Regular`. So the answer is: No, we can't tell the difference simply based on `size class`.

So I asked some further questions to find out their real requirements. And I was told that there was this iPad app that displayed two views side by side. The right view was designed as a support view with fixed `374` points width. However, when in the 2/3 mode, the main view on the left seemed too small. So they wanted those views to halve the screen. (As for when in the 50/50 mode, the app is already able to hide the right view, because the size class would be changed to `Compact`.)

#### **Constraint Priorities**

After understanding the need, the first thing immediately came to my mind was 'Constraint Priorities'.

Then, with some calculations, my guess was confirmed. That is, that requirement can be implemented by Auto Layout with Constraint Priorities. Now, I'll show you how.

Firstly, the width of the app in 2/3 mode is `694` points (I got this from Xcode). So `50%` of that is `694 * 0.5 = 347`. And `347 < 374` --- this is the point.

The key is using 'Inequality Constraints'

We add two constraints with different priorities to the width property of the right view.
constraint1: `width <= 374 @1000`  
constraint2: `width = 0.5 * containerView.width @999`

<img src="/assets/images/2016/10/ipadsplitview/constraints.png" style="width: 320px">

This `@1000` here is the priority of constraint. About constraint priorities, here are some descriptions from Apple's document:

>All constraints have a priority between 1 and 1000. Constraints with a priority of 1000 are required. All other constraints are optional.
>
>When calculating solutions, Auto Layout attempts to satisfy all the constraints in priority order from highest to lowest. If it cannot satisfy an optional constraint, that constraint is skipped and it continues on to the next constraint.
>
>Even if an optional constraint cannot be satisfied, it can still influence the layout. If there is any ambiguity in the layout after skipping the constraint, the system selects the solution that comes closest to the constraint. In this way, unsatisfied optional constraints act as a force pulling views towards them.
>
> Excerpt From: developer.apple.com, [Auto Layout Guide][2]

With these two constraints, we actually set two different width value to the right view.  
So in full screen mode, constraint2 set `width = 0.5 * 1024 = 512`.  
in 2/3 mode, constraint2 set `width = 0.5 * 694 = 347`.

Since the constraint2 is optional, the Auto Layout will try to satisfy constraint1 first. But, as the above quote said, the constraint1 is ambiguity, for it's an 'Inequality Constraint', a range. So the Auto Layout will pick the constraint with the highest priority from all the other width-related ones, then calculate the width. At the same time, the Auto Layout will try to satisfy the optional constraint as much as it possibly can.

And that means in the full screen mode, the constraint2 will try to pull the view's width to `512` points, but because it has a slightly lower priority, it can't break the `'<= 374'` rule from the constraint1. In the end, Auto Layout solves the width to equal to the nearest value to `512` that satisfies `'<= 374'` rule: `374` points. (Optional constraints show as dash lines in Xcode)

<img src="/assets/images/2016/10/ipadsplitview/full_screen.png" style="width: 720px">

While in 2/3 mode, the value from constraint2 is `347`, and this value satisfies `'<= 374'` rule. Then the Auto Layout will be more than happy to satisfy one more constraint. So we got `347` points for the width of the view.

<img src="/assets/images/2016/10/ipadsplitview/two_thirds_screen.png" style="width: 720px">

As a result, here we've got what we need, based on the Auto Layout's solving logic.

#### **Always exceptions to every rule**

If you look back the size classes picture at the beginning of this article, you will find that the iPad Pro 12.9" is an exception. In the 50/50 mode, the size class is still `Regular` for both apps. Then the width of the right view will be set to `0.5 * 678 = 339` points, instead of be hidden. Meanwhile the left view's width will also be set to `339` points.

🤔hmm... I'm wondering if our client would accepts this.

#### **One more thing**

When the iPad is in multitasking split view mode, the black splitter in between those two apps is `10` points width. Therefore, if two apps halve the screen, there will be `10` points missing.

[1]: https://developer.apple.com/library/content/documentation/WindowsViews/Conceptual/AdoptingMultitaskingOniPad/QuickStartForSlideOverAndSplitView.html "Adopting Multitasking Enhancements on iPad"
[2]: https://developer.apple.com/library/content/documentation/UserExperience/Conceptual/AutolayoutPG/AnatomyofaConstraint.html "Auto Layout Guide"
