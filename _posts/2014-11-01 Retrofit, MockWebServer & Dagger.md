---
layout: post
title: Android Testing with Dagger, Retrofit & MockWebServer
---

This is a quick little note as I forget how to easily setup an enviroment which allows me to use `Retrofit` to interface with a remote API and easily swap out the API implementation when testing to swap in canned responses from `MockWebServer` or `Mockito`

I imagine there are many approaches to achieve the same goal so I would be interested to hear of anyone elses.

#Step 1 - Gradle setup

<div data-gist-id="602f00e37fdeeac7756d" data-gist-file="build.gradle">build.gradle</div>

#Step 2 - Create Retrofit API interface

See [the Retrofit site](http://square.github.io/retrofit/)

#Step 3 - Create ApiModule `Provider` class for DI

This Module class is whats used to satisfy the runtime production api dependency via `Dagger`

<div data-gist-id="602f00e37fdeeac7756d" data-gist-file="ServerApiModule.java">ServerApiModule.java</div>

#Step 4 - Init Dagger in `Application` class

I am using a custom `DaggerHelper` class than makes it easy to swap the production and test modules. This needs to be init with `DaggerHelper.initProductionModules();` in the `Application.onCreate()`

<div data-gist-id="602f00e37fdeeac7756d" data-gist-file="DaggerHelper.java">DaggerHelper.java</div>

#Step 5 - Inject any classes

Add `@Inject` to any class fields that are provided by a module and call `DaggerHelper.inject(this)` from the class constructor. See Dagger docs for more.

#Step 6 - Create a test

 <div data-gist-id="602f00e37fdeeac7756d" data-gist-file="ExampleTest.java">ExampleTest.java</div>

#Step 7 - Party!

![Party!](http://i.imgur.com/cu4L5Am.jpg)