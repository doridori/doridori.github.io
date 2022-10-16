---
layout: post
title: "Application-as-a-Function Thinking"
---

**TL;DR** Architecting an application with a pure function at it's core can be a first step towards the "functional-core imperative shell" ideal which can simplify testing and lower an applications complexity bar.

## Why?

Simplification.

As a novel alternative to [KISS](https://en.wikipedia.org/wiki/KISS_principle), we start with a quote from [The Grug Brained Developer](https://grugbrain.dev/):

> complexity *very*, *very* bad

And for completeness:

> Complexity: a measure of how difficult it is to understand how a system will behave or to predict the consequences of changing it

For many applications the challenge is how to simply & correctly represent a large number of domain requirements in code, and how to raise the complexity-bar sufficently to enable a high degree of engineer productivity & runtime stability .

A _low_ complexity-bar reduces the amount of time it takes for a developer to get to the brain-meltdown-event-horizon-like state after which completion of new work slows down expontentially due to sheer codebase and test suite complexity which results in low grokability (i.e. code which is hard to understand or follow), an increasing degree of emergent behaviour, a higher probability of introducing regressions and generally, just _slow_ development.

A _high_ complexity-bar conversely allows a desired order and serenity to take hold and more time spent on adding value as opposed to fire-fighting.

I have observed different reasons for a low complexity bar, including:

- Domain requirements being spread throughout all codebase components
- Non-existent / incomplete / disorganised domain requirements documentation
- A non-existent / disorganised / bloated / poorly performing test suite
- Poor naming
- Race condition rich OOP state manipulations
- Low architectural cohesion across a codebase / team

In addition if your application is highly event driven (user input / network / OS / peripherals / sensors) not having a simple approach to process incoming events can lead to chaotic code. Multiply this if you are working within a sensitive domain which has strict requirements around runtime data retention.

I find having a single function at the center of an application can help to address all the above pain points.

## How?

Fundamentally an _application_ can be written to have at it's core, a single, stateless _[pure function](https://en.wikipedia.org/wiki/Pure_function)_, behold!

<div data-gist-id="4acb19b6ac4e965552ba6961e1bc9054" data-gist-file="reduce.kt">reduce.kt</div>

An `Action` represents something which has _happened_ (past tense) in the system, e.g.

- `UserSelectedLogin`
- `ApplicationForegrounded`
- `UserInteractionTimeOutLimitReached`
- `BleDeviceConnectionLost`

`State` fundamentally represents the state of the application, which can be implemented as an immutable flattened data hierarchy, in a fashion similar to [Redux](https://redux.js.org/introduction/core-concepts).

`Effect` at its simplest could represent the new applications immutable `State` and optionally any `Commands` which need to be handled e.g. `Effect(State, Command)`.

`Command` represents something which _needs to happen_ (future tense), probably in an imperitive fashion and likely interfacing with the real world, like IO  e.g. `AttemptLoginToRemoteServer(userName: String) : Command`. Note that a `Command` could easily represent a _"side-effect"_ in functional parlance. This is similar to Elm's [Commands](https://elmprogramming.com/commands.html#summary), as opposed to something like Redux's [Middleware](https://redux.js.org/understanding/history-and-design/middleware) or MVI's [preprocessing](https://adambennett.dev/2019/07/mvi-the-good-the-bad-and-the-ugly/).

### A Simple Architectural View

<img src="/images/blog/application-as-a-function-v2.png" alt="Runtime Implementation" />

The above is probably the simplest representation of this style of architecture I can think of. **It's important to reiterate this represents the _entire application_**. 

The left side of the diagram realises the fundamental Unidirectional Data Flow (UDF) [¹](https://en.wikipedia.org/wiki/Unidirectional_Data_Flow_(computer_science)) [²](https://developer.android.com/topic/architecture/ui-layer#udf) principles.

### An Example Test

Regardless of the specifics of how you choose to define `State`, keeping the core `reduce()` function pure allows us to write lightning fast automated functional [unit tests](https://martinfowler.com/bliki/UnitTest.html) which run on the JVM. These tests could potentially cover a large chunk of your applications functional requirements. For example a requirement such as:

**GIVEN** the user is logged in  
**WHEN** the application moves to the background  
**THEN** the user should be logged out  

Can be expressed at test time like:

<div data-gist-id="4acb19b6ac4e965552ba6961e1bc9054" data-gist-file="test.kt">test.kt</div>

It's worth noting tests of this form are often very simple to read, and also serve as living documentation for a given application and therefore describing the current set of supporting features / functions / requirements. This is something I have often seen organisations and test suites struggle to achieve, much to the detriment of productivity.

An a professional Android developer for over 12 years, I have myself written and seen others write tests over more traditional Android architectures which try to express a flow such as the above which end up being:

- Slow: due to a combination of instrumentation and integration complexity
- Flakey: due to concurrency being involved in the code under test
- Complicated & brittle: due to excessive mocking or integration

For me a healthy test suite == a healthy application and the above test is at the other end of the simplicity spectrum as it's:

- Fast: No instrumentation needed, so in the Android world this means lightweight JVM testing
- Stable: No concurrency
- Simple: No mocking or integration testing possible as its a pure function

### Android specifics

On a system such as Android, when following this approach I have found I have little use for anything more than a single `Activity` and a bunch of thin `Views`. `ViewModels` become a little redundant and the UI layer of the application becomes pretty simple as really it just needs to render state and map user inputs to `Actions`. More details can be found in my previous post [Android Architecture: Runtime Centric Thinking](https://doridori.github.io/Android-Architecture-Runtime/).

## A Note On Functional Programming

### Is this functional programming?

Well, not really. However, this post does introduce the concepts of pure-functions and modelling side-effects as  `Effect` (or `Command`) value types both of which are [functional programming](https://en.wikipedia.org/wiki/Functional_programming) concepts. However, there is no talk here about the more esoteric functional concepts like referencial transparency, monoids, currying and monads. We are lucky that Kotlin has allowed us work with functions as first class citizens and we as developers _could_ get much deeper into the functional world than I am proposing in this post, but we can can see some easy but tangible benefits by pulling in some of the simpler & more approchable functional concepts as outlined in this post.

Also this approach is a first step towards the functional ideal of **seperating _decisions_ from _dependencies_** and harvesting the rewards of such an approach.

### Functional Core, Imperative Shell

The concept of a **_functional-core, imperative-shell_** is a powerful one and a pragmatic way to start gaining some of the benefits of functional approaches which includes seperation of concerns and high maintainability / testability.

> A core principle in purely functional programming is to separate effects and data as much as possible. This naturally leads to applications with a [functional core](https://www.destroyallsoftware.com/talks/boundaries) and [imperative shell](https://www.destroyallsoftware.com/screencasts/catalog/functional-core-imperative-shell). The vast majority of code is written as side-effect-free functions and data and only at the boundaries of the application do the effects show up. The boundaries of the application are where our core logic meets the outside world, whether via API requests, outside input, components rendering to the page, and so on. [¹](https://thomashoneyman.com/guides/real-world-halogen/push-effects-to-the-edges/)

The application-as-a-function approach is one of many possible interpretations of the functional-core, imperative-shell idea and there are many facinating resources to dig deeper into this subject (links at end of the post).

## Paradigm Shift

Architecture is so subjecitve, what fits one developers mindset and ethos may be totally alien or disagreeable to another. Additionally, an architecture that may be a perfect fit for one project may be terrible for another, plus if you have a large team with a high amount of churn you would need to carefully weight up the pros and cons of a more esoteric architecture which slows onboarding time vs more mainstream architecture which may be slower in terms of general development for onboarded team members.

For _me_, and the way my mind works, thinking of application design from an application-as-a-function / functional-core imperative-shell perspective simplifies development & testing **massively** for many kinds of Android application. Development can:

- Be fast, truely self-documenting and have a high complexity bar
- Be free from common pain points around the Android framework and tooling, including test time tooling
- Fit well into UDF thinking, which is useful for enabling a clean UI implementation (Compose or otherwise)
- Take a step away from a variety of race-condiftions commonly found in OOP style state-mutations when combined with concurrent code
- Include core application events logging using just a single line of code in front of the reduce function 

Its potential excites me and it so far feels like a intoxicating gateway into the functional world 

## Related Thinking

The above concepts are not new to software architecure. UDF thinking has been around a while, as has functional programming (~1950s) and [Flux / Elm / Redux](https://staltz.com/unidirectional-user-interface-architectures.html) inspired architectures. We can find similar concepts in [Clean Architecture](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html), Hexagonal Architecture / [Ports & Adapters](https://www.kennethlange.com/ports-and-adapters/) and [Onion Architecture](https://jeffreypalermo.com/2008/07/the-onion-architecture-part-1/).

On Android somewhat similar concepts may surface in Model-View-Intent (MVI) thinking for _feature_ level implementation, but with less of an emphasis on functional thinking, testability & simplicity.

Application-as-a-function is a simple realisation of similar ideas, but with an emphasis on **_application-wide functional-core imperitive-shell_** thinking.

## Conclusion

I hope this post has provided some food for thought and introduced to some readers the notion of application design from a functional-core imperative-shell mindset, and one idea of what a manifestation of this principle can look like on a mobile platform like Android.

## Links

### Functional Core Imperative Shell

- [The Functional Core, Imperative Shell Pattern]([https://www.kennethlange.com/functional-core-imperative-shell/])
- [Boundaries - Gary Bernhardt](https://www.destroyallsoftware.com/talks/boundaries)
- [Functional Core Imperative Shell - Gary Bernhardt](https://www.destroyallsoftware.com/screencasts/catalog/functional-core-imperative-shell) talk
  - https://news.ycombinator.com/item?id=18043058
- [Test Doubles Are A Scam – Matt Diephouse](https://www.youtube.com/watch?app=desktop&v=7AGQ9dhWCX0)

### Books

- [Domain Modelling Made Functional](https://pragprog.com/titles/swdddf/domain-modeling-made-functional/)
- [Functional Programming in Kotlin](https://www.manning.com/books/functional-programming-in-kotlin)

### My related posts

- [Android Architecture: Runtime Centric Thinking](https://doridori.github.io/Android-Architecture-Runtime/)

### Social

- [Reddit link](https://www.reddit.com/r/androiddev/comments/y1hpwa/applicationasafunction_thinking/)

## Thanks

Thanks to [@scottyab](https://twitter.com/scottyab) and [@andyb129](https://twitter.com/andyb129) for the proof read :)

![aaaf-aw-featured](https://user-images.githubusercontent.com/1244430/196021578-162e152f-b3b3-4cba-9c23-707aea3100c2.svg)

