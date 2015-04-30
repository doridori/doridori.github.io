---
layout: post
title: "Android App Architecture: Lifecycle Events and Asynchronicity"
---

> Edit: This blog post is now deprecated. It was a bit of a brain dump which prompted me to write up my approach starting [here](http://doridori.github.io/Android-Architecture-MV%3F/)

> Edit: The day after this post Square published a post called [Advocating Against Android Fragments](http://corner.squareup.com/2014/10/advocating-against-android-fragments.html) which covers a similar approach also using `Presenters`. Defo worth a read!

Right, I have been developing Android apps for a long time now, 5 years ago this month in fact, and one thing has always bugged me; how to make the Android lifecycle play nice with the inherent asyncrhonicity that comes with the average mobile app. 

Asynchronous tasks usually relates to io (network or disk), sensor access and long-running computation. I always end up not being 100% happy with the approaches I end up using and have seen bucketloads of code where its obvious that others have not even spared a thought for this common issue.

This post is not just about async calls but also the architecture that surrounds an app that shows data being from some async data src.

Its also me just thinking aloud and hoping someone will read and say "hey, have you thought of doing it this way?".

##The problem with async

Poorly managed aysnc-ops can lead to device crashes, restarted operations and memory leaks (let alone hard to maintain & read code-bases). This is usually related to how an app handles lifecycle events and component restarts.

This most commonly manifests its-self when the app encounters a device config-change, which can be triggered by common actions like orientation changes or the hardware keyboards pulled-out (see a full list [here](http://developer.android.com/guide/topics/manifest/activity-element.html#config)) or when the acitivty has been killed in the background and then recreated when returning to foreground.

The default result of an `Activity` / `Fragment` restart is that the current activity will be destroyed and recreated, losing any async callbacks in the process. If not handled correctly this can easily result in a memory-leak of the view-hierarchy, the app crashing when trying to manipulate a detached view-hierarchy, or the long running operation being restarted. These are all common issues that any Android dev will have encountered after an app or two, and there are many articles / blog posts on the web that talk about these issues.

##A poor fix #1: Disable config-changes

A often-used quick fix is to try and remove the source of Activity restarts. This can be partially achieved by locking the app to a single orientation and disabling all config-changes via the AndroidManifest. While this does reduce the amount of activity restarts encountered its generally very bad as

1. You are disabling core functionality of the OS like orientation, locale changes etc
2. It doesnt fix the problem as the Activities can still be restarted in the background

##A poor fix #2 : Retained Fragments

I have seen retained fragments being used to try and fix this issue, as it can maintain callbacks. This is not a good solution 

1. Retained fragments are buggy
2. It doesnt fix the problem as the Activities can still be restarted in the background

##A poor fix #3 : Loaders

Loaders are good for local DB work but not great for networking due to restarting unfinished requests (As shown in [this Robospice motivations](https://camo.githubusercontent.com/db2f449c1862af4b74400546d12f956328cac131/68747470733a2f2f7261772e6769746875622e636f6d2f6f63746f2d6f6e6c696e652f726f626f73706963652f6d61737465722f6766782f526f626f53706963652d496e666f47726170686963732e706e67) diagram - more on this later). 

I also dont like how:

1. They used to be buggy and some edge case bahaviour would result in no callbacks on config-change
2. They create a relationship between loading of data and the Activities lifecycle
3. You cant just lift the loader out into a contextless class

##An OK fix #4 : Eventbus

This is a halfway house - its an ok solution but does not tick all my boxes. The idea is that an async request is made and the result is communicated via an event bus. This does work well for config-change scenarios but generally not-so-much when the activity was killed when the result was returned and then recreated. 

A potential solution for this is to use 'Sticky' events so the last result is saved to can be picked up on recreation.

Each async event could have an ID and a single event type to show the below;

- call started
- call successful
- call failed

The id could be saved in the saved state and checked in onCreate(). If no sticky event found then treat like the call was never started (as the App has been recreated), if `call started` found with matching id then show a loading spinner, if `successful` then show the data, if `failed` show the error view. This is similar to the approached used with `RoboSpice` (see below).

The problem with the sticky approach is that the event object itself can be messy or it can result in boilerplate code writing.

##What does async-heaven look like?

Really we want a solution that can cope with the following use-cases

1. Async-op returns before any further lifecycle events have occured
2. Async-op returns after `onDestroy()` and before activity recreated with `onCreate()`
3. Async-op returns after `Activity` has been recreated (i.e. `onDestroy()` & `onCreate()` has been called)

Coping by

1. Reconnecting to a long-running task if its still in progress
2. Grabbing the last long-running task result if one exists

As a side note we need the app UI to be able to show the current status i.e.

1. Ready to load
2. Loading
3. Loaded
4. Error

Its also worth noting I seem to come across two types of IO call / op which generally are handled a bit differntly.

1. Ones which require no-input i.e. `show the latest data from src x`. Generally a data refresh and the latest data is shown if available on view recreation, plus the current / last task state is shown.
2. Ones which require input i.e. `log me in with these details` or `get me data filtered by y`. Generally on view-recreation one will want to reconnect or grab the last async-task that was started, plus the current / last task state is shown.

##My General Approach

My current general approach is to use `Robospice`. In fact, I just wrote a blog post about using this which some code snippets to get you up and running quickly. See it [here](http://systemdotrun.blogspot.co.uk/2014/10/using-robospice-and-okhttp-on-android.html).

I do think `Robospice` is fantastic and major-kudos to its creators. I has just started writing something similar when it came out and they did a much better job that I would've by far. 

`Volley` is pretty similar to `RoboSpice`, apart from IMHO it never really caught on and the docs are lacking - so I have pretty much ignored it, maybe at my peril. 

A lot of people mention `Retrofit` and other similar great Open Source projects. While these are fantastically useful and I do use them, on their own they do not satisfy the above criteria for my desired 'async-heaven'. 

##Whats wrong with my current approach

It bugs me. 

Robospice is great for what it does, but I feel like it can be a little verbose for my liking (not that I have identified a better solution) and is geared towards network IO. For example, trying to implement a long-running task which is not linked to the devices network connectivity status was more hassle than it shouldve been. Also I have always wanted a way to compose network requests, parsers and post-parsing operations together simply, and I feel like its not as clean as it should be when using RS. I will say again its a great lib and Im sure I will continue to use, I just have a niggling that its not as flexible as I would like. I find that tying to contexts is both a blessing and a curse, sometimes it makes life easier (most of the time) and sometimes not. It gets around the issue of contexts being short-lived and the resulting call-back issues this can create, but by including context-tracking in the implementation it can lead me to some edge-cases where I have to think-harder than I want to. Maybe handling context lifecycles without tying to contexts is a bit of a holy-grail, but I feel it may be possible. Sometimes my intuition tells there is a better way without any good reason that I can explain. this is one of those times.

The other thing thats bugs me about approaches I have seen (& used), is that data is often tied to the Activity. In my experience people generally use one of these approaches

1. Async-op loads data tied to Activity. May or may not be saved and restored in saved-state bundles on lifecycle changes.
	- PLUS data not in view does not take up memory
	- MINUS data will be lost if savedInstanceState not implemented
	- MINUS only accessible in one part of app
	- MINUS saving and restoring may be slow
	- MINUS background updating of that data can be difficult
2. Async-op loads data to in-memory singleton. 
	- PLUS loaded data persists easily over context restarts
	- MINUS without data mgmt app mem footprint can bloat
	- MINUS No unified approach for handling loading issues *
3. Async-op loads data to DB.
    - PLUS data persists FOREVER!
    - MINUS can be slow
    - MINUS can result in lots of boilerplate DB code
    - MINUS data may not be suitable for storing in a relational DB

> \* If using the singleton approach above (no 2.) I will have observers on the data. Any errors in loading notifies the UI via an eventBus (I use [greenrobots](https://github.com/greenrobot/EventBus) but [Otto](http://square.github.io/otto/) is an alternative). 

Any of the above can of course implement caching at the http layer.

##What I want

What I want to explore is if a good base-approach for Android-apps-of-today exists. Everytime I start a project I think about this stuff and use a slightly different approach and learn a little, but I want to have a defined off-the-shelf approach I can use and recommend to people that results in clean code and solves the above identified issues. I feel like there is loads of posts about the above tech and loads of questions about how to handle this stuff but Im never satisfied by the answers or solutions. 

##Acceptance criteria

Not having to think about the `Activity` or `Context` lifecycle __AT ALL__ when working with async!

##Promising Directions

###RxJava

Unless you have been living under a rock (or at least your not a full-time Android dev) you will have at least heard of [RxJava](https://github.com/ReactiveX/RxJava). Its a port of a popular .net library with origins that go back decades. I wont go into detail here but in a sentence...

>RxJava is a Java VM implementation of Reactive Extensions: a library for composing asynchronous and event-based programs by using observable sequences.

I have been researching this and will have a play in an app soon, but for me at least this seems like a big deal. Moving away from imperative to a functional approach for the above definitely seems like it will clean up a lot of code and allow me the composition of data requesting & processing code that I have been searching for. Plus moving away from callback chaining and so on is a no-brainer. Im 95% sure this is going to be a part of my new approach, but not the whole enchilada, as on its own it does not solve all the issues.

###MVP & MVC

Now every single Android app I have seen the code base of treats the `Activity` or `Fragment` class as the view controller. Any data loading and user actions are processed here, as well as setting up the view hierarchy and so on. 

This does lead to the problem that all of this 'controller' code is subject to the same lifecycle events as the view, which is often what causes the issues with anything relating to async. 

A promising direction to look in could be one which separates the view (i.e. `Activity`) from the data loading. `MVC` and `MVP` are two loose design patterns that spring to mind. This way we can perform all the async code inside a Controller or pPresenter that is not strongly coupled to the Activity. MVP & MVC are not as intuitive on Android as other frameworks, but is possible. See the _Related Links_ below for more on this, especially Squares Mortar & Flow, which achieves similar to my example below but with a bit more style and support!

The idea is that the Presenter class will hold all the logic for moving between states and network IO etc.

##A Simple Example Using Presenters

Lets take a log in screen. We have a username and password field and a submit button. With the naive approaches outlined above what would happen when the submit button is pressed and the screen is rotated? We lose the callback of course.

If we separate the Presenter logic, so the responsibilities are

- `Activity` only hooks up to the views and calls `Presenter` methods with data from the view
- `Presenter` performs the async calls and callbacks

For this example lets say the Presenter is injected using [Dagger](https://github.com/square/dagger) as a `@Singleton` so it will persist across `Activity` instantiations. 

The Presenter holds some state in some way that is either 

- IDLE
- CONTACTING_SERVER
- AUTH_OK
- AUTH_FAILED

The `Activity` / `View` needs to;

1. Be able to query this state in `onCreate` to show the appropriate view(s)
2. Be updated in any changes to this state

The above states should also be able to contain some meta-data about that state i.e. `AUTH_FAILED` has some info about why it failed, `AUTH_OK` may have some complex object structure show to the user etc. This means that we want to be able to have custom interfaces for each state. Some solutions I have seen talk about just passing the state as an enum and maybe some optional primitive meta-data, but this is a bit messy IMHO and not very flexible. 

###Exposing `State` interfaces

Lets say we have a `State` interface, and our Presenter has a `State` field, and each concrete state is a subclass of `State` i.e. `IdleState implements State`.

Really we want each concrete `State` to be able to expose its own interface. This could be done using runtime type checking (e.g. using `instanceOf`) but we really want to avoid this, as its a code-smell and we lose compile time type checking.

We could implement the callbacks in the following ways. 

####The `Presenter` exposes an interface 

All the state classes extend from `LoginState`.  The Login Presenter can just maintain a state-machine with single variable of type `LoginState` and call the callback methods when the state changes or when requested. 

- Advantage - The implementor has to handle all the states as enforced by compiler
- Disadvantage - this will probably internally end up using some run-time type checking and casting of the State subclasses (as mentioned above) so as able to call the specific state callbacks. We could just expose a single interface which passes a State object but this will not allow each concrete State subclass to expose its own interface, again, unless casting or runt-time checking is used. This leads to messy code-design.

####The `Presenter` posts events

- Advantage - Events _usually_ easier to test with in comparison to interface callbacks
- Disadvatage - in practise sometimes find it hard to debug if an expected event is not sent or received
- Disadvantage - Either need to create an event receiving method for each concrete state type (if your event bus supports subscriber and event inheritence) or one supertype event. If one per concrete type can be If using a supertype event then you end up runtime type checking.

####Introducing the `Visitor` design pattern

This is a situation that seems perfect for the `Visitor` design pattern. Quoting the famous 'Gang of Four' [Design Patterns](http://www.amazon.co.uk/gp/product/0201633612/ref=as_li_tl?ie=UTF8&camp=1634&creative=6738&creativeASIN=0201633612&linkCode=as2&tag=dori-21&linkId=QMYKC32VM7UALJ47) book, the Visitor pattern can be used when 

> An object structure contains many classes of objects with differing interfaces, and you want to perform operations on these objects that depend on their concrete classes.

Well, that sounds pretty much what we want to do here! 

We can implement this simply with classes like the below (only including one actual concrete state class for example).

<div data-gist-id="fe860bdd2a88c40275e5"></div>

Your View / Activity / Fragment can just implement the `LoginVisitor` interface and then grab States from the `LoginPresenter` and render the UI accordingly.

We would also need to add some form of __observer__ so that the LoginVisitor implementor, which would probably be your View / Activity / Fragment would be able to revisit the current state and make the appropriate UI changes. This could be done by incorperating an Observer, Callback or Event into the `LoginPresenter`, the choice is yours!

#Conclusion

So maybe the simple example above looks a little more complicated than was expected. However, the power of it is it can work in many situations with large state-machines and custom interfaces per state. This makes for clean, compile time checked code. Due to the nature of the Presenter class being decoupled from the Activity class (and therefore the lifecycle) no more async issues, we can make our login call to the server while the Android lifecycle goes crazy, knowing that when we return to the Activity the state will be picked up and the correct UI operations will take place.

##Related Links

- [Google I/O 2010 - Android REST client applications](https://www.youtube.com/watch?v=xHXn3Kg2IQE)
- [Android Architecture blog series](http://www.therealjoshua.com/2011/12/android-architecture-part-9-conclusion/) talks about MVC on Android, State pattern and DAOs
- Squares [Mortar](https://github.com/square/mortar/) Controller (Presenter) lib for Android views
- [Mortar & Flow](http://corner.squareup.com/2014/01/mortar-and-flow.html) 
- [Android Architecture: Message-based MVC](http://mindtherobot.com/blog/675/android-architecture-message-based-mvc/)
- [Visitor design pattern](http://en.wikipedia.org/wiki/Visitor_pattern) 
- [Advocating Against Android Fragments](http://corner.squareup.com/2014/10/advocating-against-android-fragments.html)


