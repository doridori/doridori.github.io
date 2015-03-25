---
layout: post
title: Using Robospice with OkHttp on Android
---

What is Robospice?

Taken from [the projects Github](https://github.com/stephanenicolas/robospice) 

> RoboSpice is a modular android library that makes writing asynchronous long running tasks easy. It is specialized in network requests, supports caching and offers REST requests out-of-the box using extension modules.

Essentially is makes it easier to persist ongoing network connection overs Activity and fragment lifecycle events i.e. config changes like rotation. For apps that dont want to restart the request each time this can be a really nice architecture approach and is putting into practise the concepts outlined in the getting-older but great talk [Google I/O 2010 - Android REST client applications](https://www.youtube.com/watch?v=xHXn3Kg2IQE)

_Note: Robospice is great and I have used in a lot of apps I have built. There seems to be quite a trend of moving towards RxJava approaches to network comms which seems to make it easier to work with async opps and relativly simple to allow re-connection to existing calls etc. This is something I am playing with at the moment and will probably write a post about in time._

##What is OkHttp

`OkHttp` is another lib from the fantastic dev team at Square. It offers a stable replacement for the `UrlConnection` and now pretty much deprecated `HttpClient`. It allows you to stop worrying about platform version specific issues with both the the Android supplied http classes and have the same code running on all targeted platform versions. Equally cool is that it conforms to the `HttpClient` and `UrlConnection` interfaces so its pretty easy to drop in if you are putting into an existing app.

#How to use

I would advise reading the Robospice docs / wiki but below are a few notes to get you up and running quickly

##Add to gradle build file

<div data-gist-id="997acd09d24c65faf046" data-gist-file="build.gradle">build.gradle</div>

##Create your own SpiceService subclass

###In the Manifest

<!-- SERVICES -->
```
<service android:name=".remote.spice.MySpiceService" android:exported="false" />
```

###Class

<div data-gist-id="997acd09d24c65faf046" data-gist-file="BasicSpiceService.java">BasicSpiceService.java</div>

##Create a base Activity / Fragment

<div data-gist-id="997acd09d24c65faf046" data-gist-file="BaseFragment.java">BaseFragment.java</div>

##Making a non-cached call 

You can just leave the empty `CacheManager` and as long as you dont pass in a call id when calling `execute` (i.e. use the two arg constructor instead) the cache will be bypassed. 

Its important to note that your app will not receive callbacks after `SpiceManager.shouldStop()` is called so if your locking the `SpiceManager` lifecycle calls to `onStart` & `onStop` your code should be able to handle a missed response (due to activity / fragment being in the background OR recreated)

In most cases you will want to be using cached calls / pick up the response to the last call made regardless of lifecycle events. 

You may want to use caching in your http layer based upon http headers as opposed to in RoboSpice - which may be another reason to use non-caching execute() methods.

##Making a Cached call 

Good for reconnecting to pending requests / operations after lifecycle events i.e. activity restarted) and / or not hitting the network for saved data.

>####*A note on call types and ids
>In my mind there are two general types of calls in regards to caching. __"One-time"__ calls (like login / submitting something etc) where the response is specific to what args / data was sent and __"updating"__ calls where you may be grabbing the newest set of data for something (say tweets).

>__One-time__ For the call ids in regard to caching I generally use generating `UUID`s for one-time calls (see code below) which would need to be in the savedStateBundle. Combining with an in-mem LRU cache of size 1 (see below) would be a sensible strategy here.

>__Updating__ For updating calls a static final cache id may make more sense (or the call URL). You may want to combine this with a persistent file cache.

###Updating caching example

This can be pretty simple, basically by

- checking if the call is pending already (using a static final cache key)
- if not pending start request
- if is pending reconnect to request

The important code for the above would be something like

<div data-gist-id="997acd09d24c65faf046" data-gist-file="Reconnect.java">Reconnect.java</div>

###One-time caching example

For "one-time" calls this is made up of the below steps (explained below)

1. Generate a unique `id` for the api call *
2. persist and load that `id` in your saves state bundle
3. pass the call id and cache duration check into the `SpiceManager.execute(...)` method
4. Check if need to reconnect with existing/previous call
5. Make sure the `CacheManager` is setup correctly

<div data-gist-id="997acd09d24c65faf046" data-gist-file="OneTimeCache.java">OneTimeCache.java</div>

### Creating Your CacheManager

For each object / result that you want to cache you will need to set this up via the [`CacheManager`](http://stephanenicolas.github.io/robospice/site/latest/apidocs/com/octo/android/robospice/persistence/CacheManager.html) instance you create inside your SpiceService subclass. 

See [Design of RoboSpice ](https://github.com/stephanenicolas/robospice/wiki/Design-of-RoboSpice) and [this Robospice wiki link](https://github.com/stephanenicolas/robospice/wiki/A-User's-Perspective-on-RoboSpice) for a good overview to this.

You create/use an [`ObjectPersister`](http://stephanenicolas.github.io/robospice/site/latest/apidocs/com/octo/android/robospice/persistence/ObjectPersister.html) subclass for each. The main subclasses of this are

1. `InDatabaseObjectPersister`
2. [`InFileObjectPersister`](http://stephanenicolas.github.io/robospice/site/latest/apidocs/com/octo/android/robospice/persistence/file/InFileObjectPersister.html)   
3. [`LruCacheObjectPersister`](http://stephanenicolas.github.io/robospice/site/latest/apidocs/com/octo/android/robospice/persistence/memory/LruCacheObjectPersister.html) (memory)

which in turn have many useful subclasses (like `JsonObjectPersister`)

For the below example I am using a [GsonObjectPersister](https://gist.github.com/doridori/68a13a2dc9648b4d6fd0). The linked code is just pulled from one of the [Spring extensions](https://github.com/stephanenicolas/robospice/tree/release/extensions/robospice-spring-android-parent/robospice-spring-android/src/main/java/com/octo/android/robospice/persistence/springandroid/json/gson) but can be used standalone with the gist (tested with RS 1.4.12).

<div data-gist-id="997acd09d24c65faf046" data-gist-file="CacheManager.java">CacheManager.java</div>

###Common Gotyas

- __`RequestListener.OnRequestSuccess(Object)` passing `null`__. This can happen when checking the cache using **`SpiceManager.getFromCache()`** and the cache is empty - make sure to account for this. Solution - check or wrap the callback with your own to add a more descriptive callback.
- Default **auto-retries** will be 3. Disable per request with `setRetryPolicy(null);`
- Be careful if placing the `SpiceManager` `start` and `stop` calls in the `Activities` / `Fragments` `onStart` & `onStop` methods thats you **dont instigate the request before this point**, as this can result in edge cases where the response is lost and the UI is not updated. You can mitigate this with a combo of checking for cached / executing calls before starting your api call in onStart.
- If dont specify a class to your cache manager the cache will not be checked and the normal callbacks wont be called - this can be with a silent log if you have disabled log so watch out.
- __Offline__ operations - you will still place the code in the network method override so this wont by default work when the device has no network connection. See [here](https://groups.google.com/forum/#!topic/robospice/TPf_-Id3l88) for more. 

##Great Links

[A User's Perspective on RoboSpice](https://github.com/stephanenicolas/robospice/wiki/A-User's-Perspective-on-RoboSpice)
