---
layout: post
title: "Android Architecture: MV?"
---

The architecture of most Android-apps is a _mess_. 

#The Problem

I'm a contractor and I'm often brought on to projects have long since passed their first commit. I feel like I have had a good insight how to the average dev approaches Android app architecture from this, and also from open-source projects, android blog posts, the Android developer site and more. One thing that strikes me about ~99% of the code I see is that there is not much of a notion of a clean, overarching application architecture. By this I mean that people seem genuinely happy to shoehorn their code into `Activitys` & `Fragments` with the occasional class named something like `MyCrazyController`. Why is this an issue? The problem with this is that it results in

- hard to test code
- hard to read code
- hard to refactor code
- hard to reuse code
- hard to handle-config changes code
- **hard to write asynchronous code**
- hard to not create loads of edge-case code (i.e. its buggy)
- hard to hand-over code

Its an easy thing to fall into doing because

- ~99% of all Android examples are written in this way
- It sort of works most of the time
- there are framework classes (i.e. `Loaders`) and librarys (i.e. RoboSpice) around now to work around _some_ of the issues that manifest from using this stock approach. While I have used these approaches in the past I find they can make a code base more complicated than it needs to be.
- People have got used to hearing that "Android is hard to test" or "just lock it to portrait" and "restart / reconnect to the web service call on rotation" and just think "well thats how it is"

I admit, I would also include my past-self into this group, which is why I am here, attempting to atone for my coding sins and mine and others unknowingly devout following of the book of anti-patterns.

How has this come to be?

<img src="/images/blog/droid_confused_big.jpg" alt="Confused Droid" />

#MV?, who's in control?

I believe a big part of this is that not much thought has gone into the actual generic app architecture approach that devs could use. In my opinion there is a vast lack of some sort of (and I use this term _extremely_ loosely) `Controller` for application components.

__SNAP QUIZ: What does the term `Controller` mean to you?__

There seem to be a few general camps of thought relating to `MVC` architecture and Android

1. "What's this MVC?"
2. XML is the `View`, `Activity` / `Fragment` is the `Controller`,  SQLite / in memory data is the `Model` 
3. `Activity` / `Fragment` is the `View`, some crazy class is the `Controller` with many mixed responsibilities, SQLite / in memory data is the `Model` 
4. I use some open source library and guidelines to structure my app
5. I have my own solution

~98% of the apps I have seen fall into 1,2 & 3. Maybe my sample size is too small, or maybe people just don't have time to think about this stuff, either way, there is a lot of room for improvement here.

#A brief overview of MV* Architectures

I will get on to an approach that works well for me, but as a Primer a quick introduction to the MVC family of terms.

##MVC (Model-View-Controller)

[Wiki link](http://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93controller)

First things first. The term MVC is in my opinion _highly subjective_ and _highly abstract_. The pain point seems to be around the definition of `Controller`. Really its more of a _concept_ that a _design pattern_. Put simply and by someone (Martin Fowler) who knows a bucketload more than myself

> Take Model-View-Controller as an example. It's often referred to as a pattern, but I don't find it terribly useful to think of it as a pattern because it contains quite a few different ideas. Different people reading about MVC in different places take different ideas from it and describe these as 'MVC'. If this doesn't cause enough confusion you then get the effect of misunderstandings of MVC that develop through a system of Chinese whispers. (From [GUI Architectures](http://martinfowler.com/eaaDev/uiArchs.html))

Reading up on the history of MVC, by the fact it was invented in the 70's at the time when UI stuff was in its infancy its not surprising it has taken on many different faces over the years when encountering new tech stacks and frameworks. 

`MVC` can be interpreted in two ways it seems

1. (less-often) An over-arching term, which is a superset of all MV* architectures that separate some View from some Model and other middle-ish layer than may or may not communicated with the View & Model components directly. 
2. (more-often) A slightly more concrete notion of a pattern that relates the M V and C components in a **triangular relationship** which does not allow the user to interact with the View directly but only via a View and / or Controller. View displays the Model and Model is manipulated via the Controller. 

Unfortunately in practice I feel (from code I have audited) is that some devs feel like naming a class `Controller` is enough to ensure 'good' architecture, even if that controller class breaks every rule in the book of OOP (ok, I'm thinking of one project only here, but this is also the only project I saw where the dev even attempted to pull the 'controller code' outside of the `Activity`) including leaking views via static references in the Controller code.

Anyhow, my advise would be not to actually think about MVC very much. IMHO its too abstract and therefore loses one of the major powers of design-patterns in the first place, which is to quickly convey a structural concept between developers. It is likely to cause more confusion than a more explicit term.

##MVP (Model-View-Presenter)

[Wiki link](http://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93presenter) 

MVP is again quite loose. Even from its inception people had different ideas of what responsibilities the `Presenter` component had.

> One of the variations in thinking about MVP is the degree to which the presenter controls the widgets in the view. On one hand there is the case where all view logic is left in the view and the presenter doesn't get involved in deciding how to render the model. This style is the one implied by Potel. The direction behind Bower and McGlashan was what I'm calling Supervising Controller, where the view handles a good deal of the view logic that can be described declaratively and the presenter then comes in to handle more complex cases. 

> You can also move all the way to having the presenter do all the manipulation of the widgets. This style, which I call Passive View isn't part of the original descriptions of MVP but got developed as people explored testability issues. I'm going to talk about that style later, but that style is one of the flavors of MVP. (Also from [GUI Architectures](http://martinfowler.com/eaaDev/uiArchs.html))

The thing that all MVP approaches seem to have in common is that the `Presentor` has some notion of what the the UI can do and the state of the UI elements. Depending on where you look this can be anything from the Presenter object having the same interface in code as the UI does to the user to a slight notion of what colors things should display etc.

Unlike the triangular relationship of components in the general MVC way of thinking MVP is linear, meaning the `Presenter` sits _between_ the `Model` and the `View`.

For more info I would recommend you check out another Martin Fowler article [Retirement note for Model View Presenter Pattern](http://martinfowler.com/eaaDev/ModelViewPresenter.html).



##MVVM (Model-View-ViewModel)

[Wiki link](http://en.wikipedia.org/wiki/Model_View_ViewModel)

This is a popular one in the Microsoft (& Xamarin) world. The thing that all MVVM architectures have in common is...data-binding! In a nutshell the ViewModel can define the binding between Views and data properties and if bound two ways then changes to the view update the model and changes to the model update the view, pretty much for free. There are some MVVM librarys for Android, but I have never played with them, or spoken to anyone who has. If you interested your can check out [RoboBinding](http://robobinding.github.io/RoboBinding/).

##MVA (Model-View-Adapter)

[Wiki link](http://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93adapter)

This one is interesting as its pretty simple. This is like a linear MVC (the same as MVP above), meaning that there is no communication between the `View` and the `Model`, the `Adapter` sits in-between them like a burger in a bun.

From my own reading this seems like it can easily be the same as an MVP pattern if the `Presenter`/`Adapater` isn't storing much `View` state. As you can see, its all a bit of a gray area!

See [MODEL-VIEW-ADAPTER](https://www.palantir.com/2009/04/model-view-adapter/) for more.

##MV*

The above are the main ones you see around at present. I have come to the conclusion that whenever you hear about MV* to actually look at how the existing code or framework is actually doing the things above, to get a handle on whats really going on.

#What Now?

I have outlined a few common architectural terms and is laying the way for me to talk about a pattern than I have found works for me that fixes the issues outlined at the beginning of this post. My next post will present this pattern.

#Down the rabbit hole (links)

While MV* approaches are often talked about, the following are also used on Android and the subject of the occasional discussion.

- [Hexagonal Architecture (Ports & Adapters)](http://alistair.cockburn.us/Hexagonal+architecture)
- [Clean Architecture](http://blog.8thlight.com/uncle-bob/2012/08/13/the-clean-architecture.html) (_contains some good links itself_)
- [Presentation Model](http://martinfowler.com/eaaDev/PresentationModel.html) (_agree with opening but do not recreate view interface in code as it will need V->PM Observer sync whereas flow makes more sense most of the time in Android land_) 
- [GUI Architectures](http://martinfowler.com/eaaDev/uiArchs.html)

And some related links

- [Model View Whatever](http://khanlou.com/2014/03/model-view-whatever/)

#Appendix 1: iOS ViewControllers

I have extremely limited iOS exposure but from poking around it seems that the iOS version of MVC is closer to MVP as the Controller directly manipulates the Views and also contains 'traditional' controller logic (even though posts and docs say its MVA). Would be interested to hear an iOS devs take on this. Quite a lot of responsibilities which it seems earnt the title of [Massive View Controller](https://twitter.com/Colin_Campbell/status/293167951132098560).

- [Model View Whatever](http://khanlou.com/2014/03/model-view-whatever/)
- [Apples MVC guidelines](https://developer.apple.com/library/ios/documentation/General/Conceptual/CocoaEncyclopedia/Model-View-Controller/Model-View-Controller.html)
- [View Controller Programming Guide for iOS](https://developer.apple.com/library/ios/featuredarticles/ViewControllerPGforiPhoneOS/Introduction/Introduction.html)






