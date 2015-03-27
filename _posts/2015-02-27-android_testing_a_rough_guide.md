---
layout: post
title: "Android Testing: A Rough Guide"
---

Do you have a go-to approach when writing automated tests on Android?

Every project I work on has a different testing strategy, and each time I learn something new along the way. This post is a rough outline of my approaches to automated testing on Android (including past, present & future). The post serves 3 purposes:

1. Writing stuff down always helps me think about what Im actually doing and if it makes sense.
2. For others who have less experience writing tests; this will hopefully offer some ideas of paths to explore.
3. For others with more experience; you can give me some hints of what I'm missing!

Im _not_ going to talk much (or at all) about _how_ you use the tools, otherwise this post would be huge. This is more on a jumping off point for further exploration :) It there is something you want more info on, investigate it, write a post about it, and I can link to it here!

#Why test?

Automated testing and TDD is one of those things in software development that some love and some hate (actually, what _isn't_ like that in dev?!). Regardless of what side of the fence your on there is one thing thats hard to argue against, writing tests forces you to think about your software architecture. I think actually this is main thing I like about it, but probably not its biggest advantage. 

#Testing on Android

In the past I have felt like we were short-changed in the Android world when it comes to testing, compared to other languages or frameworks that have been designed from the start to ease the path of testing (i.e. RoR). But the tools are improving all the time which is fantastic. 

_Edit: After starting this I listened to [Episode 1](http://fragmentedpodcast.com/episodes/1/) of the new [Fragmented podcast](http://fragmentedpodcast.com/), which is interesting as they echo my thoughts here exactly!_

#Tooling
      
##Test Frameworks

There are a number of ways to run your tests, the simplest first choice to me seems to run them on a device or emulator via a test apk (see below for emulator choices) rather than using something like [Roboelectic](http://robolectric.org/) (which lots of people do like and use) as I have heard too many people fall out of favor with it. 

The new [Android Testing Support Library](https://code.google.com/p/android-test-kit/) now supports JUnit 4 (yay) via the [`AndroidJUnitRunner`](https://code.google.com/p/android-test-kit/wiki/AndroidJUnitRunnerUserGuide) which allows for more expressive test declaration which is great. Using this you can write some of your tests against the JVM (if your not using Android classes in those tests). 

- See [this post on the Android Tools](http://tools.android.com/tech-docs/unit-testing-support) site for how to set up for Android Studio 1.1+.

##UI Automation

Lots of solutions out there but I think the best bet currently is [Espresso](https://code.google.com/p/android-test-kit/wiki/Espresso). This is direct from Google and has just hit 2.0 so a great time to start using if your not already. The only bugbear with it is the [`IdlingResource`](https://code.google.com/p/android-test-kit/wiki/EspressoSamples#Using_registerIdlingResource_to_synchronize_with_custom_resourc-c)s imho but with the right architectural choice you shouldn't have to actually mess with these that often. For me this replaces (and improves upon) Robotium.

##CI servers

I have not used CI servers much but there are a good amount of options here. I wont compare them as thats a whole post in itself. Some are free, some are hosted, everyone will have their own requirements here.

- [Jenkins](http://jenkins-ci.org/) (Good article [here](http://www.bignerdranch.com/blog/continuous-delivery-for-android/))
- [Travis](travis-ci.org)
- [Ship.io](https://ship.io/)
- [Greenhouse](http://greenhouseci.com/)
- [TeamCity](https://www.jetbrains.com/teamcity/)

From talking to people Travis would be my first choice (if I didn't have to pay personally for closed-source) otherwise one of the free ones (ship or greenhouse).

##GIT

Version control is a given, and Git is the go-to for most. Combined with a CI server you can run your test suite on commit / pull requests / pushes and make sure no-one is breaking code!

##Mocking

These tools can be used to write your actual tests and are much recommended

- Mockito (see the [javadoc](https://mockito.googlecode.com/hg-history/1.5/javadoc/org/mockito/Mockito.html) for more info on how to use)
- MockWebServer 
    - See the [github project](https://github.com/square/okhttp/tree/master/mockwebserver) for more
    - See [a post I wrote](http://systemdotrun.blogspot.co.uk/2014/11/android-testing-with-dagger-retrofit.html) about using MockWebServer with Dagger and Retrofit

##Emulators

[Genymotion](https://www.genymotion.com/#!/) - Android emulator on steroids!

#Supporting Architectural Decisions

So we have the tools, but what concepts will help you write clean code which is each to test? Again here is a ultra-high-level intro!

##Inversion Of Control (IoC) and Dependency Injection

A common approach for writing decoupled code - which is what you want pretty much all the time but especially when your writing isolated tests. Helps to break down your dependencies. I pretty much use [Dagger](http://square.github.io/dagger/) to achieve this aim.

##4 Simple Rules

Ok, so more of a general programming one this one, but I would recommend [this book](https://leanpub.com/4rulesofsimpledesign) to make you think about simplifying your code, which as you guessed it, will help you test your code!

##Coping With Asynchronous Code

One of the things that is not-so-simple when writing Android tests is how to test your asynchronous code. This causes problems in a normal test-case as by default a test will not wait for the async callback so the test will exit (and may pass or fail depending on your test code) and then your async callback may happen and actually case another test to fail. 

I shouldn't mention this but years ago I [hacked a solution](https://github.com/doridori/AndroidUtils/blob/master/App/src/main/java/com/doridori/lib/testing/FutureAsyncTester.java) to allow me to wait for async callbacks which I thought was really clever, but I now look back on it in horror!!! This is just way too complicated and the wrong approach to take completely.

Its a good idea to test the components behind your asynchronous interfaces separately - but for integration tests and UI automation its good to have a few of the below techniques in your toolbox.

Thanks to [@chrisjenx](https://twitter.com/chrisjenx) as the below has been discussed with him on a few occasions and he has helped shed light on the following.

The better approach seems to be to either;

- Make asynchronous code run **synchronously** when testing 
	- If using **`Retrofit`** for your api methods you can [swap out the Executors](http://stackoverflow.com/questions/23142437/) so it runs synchronously. Combine this with loading data from the `MockWebServer` and its can be a nice approach.
	- If using **`Dagger`** you can easily swap out your async components at test time with synchronous mocks. The code inside the async ops can be tested separately.
- Use a framework that **auto-waits** for any running worker `Threads` or `Executors` to finish.
	- If using **`Espresso`** to execute your UI-tests, this will utilize the `IdlingResource` interface which you can plug into your async code, which causes Espresso to wait for your async operations to finish before continuing with the test. This works out of the box for stock `AsyncTasks` but for more interesting threaded behavior you may need to implement this interface directly.
- Use a **timeout** to wait for callbacks. 
    - Lots of people do this with `Thread.sleep` but this is a **bad idea**! Your tests will be slow and you will probably stop running them!
    - An alternative is to use the [`Mockito.timeout()`](http://docs.mockito.googlecode.com/hg/org/mockito/Mockito.html#timeout(int)) method which will wait up to the declared time for the expected behavior to happen. This is not as bad as `Thread.sleep` as it will only take a long time when it fails - which should not be the norm. Bear in mind, the documentation states _"Allows verifying with timeout. May be useful for testing in concurrent conditions. It feels this feature should be used rarely - figure out a better way of testing your multi-threaded system"_.
    - Use [`PollingWait`](https://junit-toolbox.googlecode.com/git/javadoc/com/googlecode/junittoolbox/PollingWait.html) from the [JUnit Toolbox](https://code.google.com/p/junit-toolbox/).

**RxJava** is a great one to check out as the base idea is to get away from the pains of asynchronous programming (think callback hell). One approach to look at here is [toBlocking()](https://github.com/ReactiveX/RxJava/wiki/Blocking-Observable-Operators). Below are some starting links for an intro to Rx

- [RxJava and Android: Just What the Doctor Ordered](http://markhudnall.com/2013/10/15/rxjava-and-android/)
- [Learning RxJava With Android by Example](http://nerds.weddingpartyapp.com/tech/2014/09/15/learning-rxjava-with-android-by-example/)

##MVP

I will cover this more in another post but adopting a good separation between the views and app life-cycle (I would include `Activitiess` and `Fragments` in the _View_ camp) and your Controllers / Presenters / do-stuff-thingys makes things a heck of a lot easier to test. Check out [An Investigation into Flow and Mortar](http://www.bignerdranch.com/blog/an-investigation-into-flow-and-mortar/) for an overview of Squares approach to this separation. 

#Test Devices

So, as you all know, in the Android world we have the joy of [well over 10k device models](http://opensignal.com/reports/2014/android-fragmentation/) and debugging those device specific issues aint easy. Generally I have been able to get my hands on the device thats causing an issue, or its repeatable on the emulator or another similar device. I have not used any test clouds up to this point so would be interesting to hear any recommendations.

Some companies build their own test-suite for the most popular devices, I am yet to come across a good blog post outlining peoples solutions here. If you were to go this route I would check out [Spoon](https://github.com/square/spoon). A google talk I saw a while back seemed to recommend against this approach as the speaker claimed ~98% of all bugs are due to programmer error or the framework itself so focusing time on spinning up emulator configs is more worth while.

#Conclusion

Well thats it! When I go to approach testing on my next project this is pretty much the approach I will use. Would be great as always to hear any others experiences or responses :)

#Interesting Links

- Great talk from Droidcon London with some interesting ideas for maintaining app quality [Testing Applications At Facebook](https://skillsmatter.com/skillscasts/5630-testing-applications-at-facebook)
- [How to write good tests](https://github.com/mockito/mockito/wiki/How-to-write-good-tests)
