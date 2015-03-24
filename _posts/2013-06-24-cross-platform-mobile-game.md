---
layout: post
title: Cross-platform mobile game framework comparison
---

I am writing this post as I am looking into learning a cross-platform mobile game framework. I went to a talk by a Corona guy recently which made me interested in this again so here goes. The more I look into this the more options there seem to be!

![](http://doridori.co.uk/blog_images/game_frameworks_little_time.png)

I have written a few simple 2d games before, some more successful than others (one sat at no. 1 on Amazon UK for a couple of months) and its also how I kick started my Android career - but they have all been in Java and mostly Android only (one was an Applet - yikes!). It would also be great to be able to target more than one platform with the same codebase.

![](http://doridori.co.uk/blog_images/game_frameworks_platforms.png)

These are just my first impressions after a quick look at the below so will not be a detailed comparison. I know I have left many out but these looked like the most promising from my research.

#My Requirements

- Support as many platforms as possible (Android and iOS a must)
- Not be really expensive - ideally for me thats not really more than a few hundred dollars a year
- Have good support network (forums, dev support, wikis, large community)
- Have some form of RAD development - can be code hot-swapping or tiny compile / load time. Asset hot swapping would be nice.
- Not have some weird language. I would _like_ to use Lua if possible as I have heard some good things about it and its quite widely used in game dev and RAD tools. My background is mostly Java but I want to learn something new!
- Good for 2D games
- Stable
- I would like to learn some OpenGL so am not looking for pure abstraction of the graphics layer either

#Potential candidates

- Corona
- Marmalade (plus Marmalade Quick)
- LidGDX
- Loom
- Unity
- PlayN
- Gideros
- MOAI
- Cocos2D-x (framework)

A quick rundown of things I have heard about the above.



* * *

##Corona
<http://www.coronalabs.com/products/corona-sdk/>

###Good

- Very quick to get up and running
- FREE for Android / iOS / Kindle / Nook. See <http://www.coronalabs.com/store/>
- Lua
- I won a free pro license to this!
- There is a free version which could be used for RAD dev.
- Physics: Box2D (people say is bloated)

###Bad

- If something is not supported then your a bit stuck (unless you have the top tier account type I think)
- Anecdotally people on line are saying bugs take a long time to fix
- 9mb min app size
- [Some](http://pushpoke.com/2013/02/why-did-we-choose-gideros/) people say lacks in performance

###Overall

Seems good for getting up and running quickly - I cant see any great games made with it but good for first timers I reckon.



* * *

## Marmalade (plus Quick)
<http://www.madewithmarmalade.com/>

###Good

- Quick uses Lua, Core uses C++
- Quick is a good code swapping RAD environment
- iOS and Android from $15 a month so a good price - can add more platforms for more. See <http://www.madewithmarmalade.com/buy>
- Supports WinMo - not many (if any) of the other platforms mentioned here do.
- Can use Cocos2d-x or IwGame on top of this 
- Some good games made with this (Draw Something, Cut The Rope, others..)

###Bad

- Some people complain docs are not great and support also lacks.
- I remember when this was Airplay and none of the apps made with it would work on my phone they would just crash out iOS style and disappear! I also had this experience with Draw Something a lot. However its hard to tell if this is the framework or the developers.

###Overall

I may try this one out - powerful and can be extended. I would prefer it was open source (Quick is but the Core isn't) so the community could bug fix. My reservations may be based on out of date experiences so I should give it a shot really.

###Further Research

**Others Experiences:** Looking into this some more I found [these slides](http://www.slideshare.net/Nekuromento/marmalade-bittersweet-experience-17211067) from Mad Hat Games which seems to reinforce my worries. This is from March 2013 and says the same stuff others have said - the stand-outs for me are the bugs going unfixed for months and sdk regressions - eep! This seems to be related to working in C++ but considering Quick sits on top of this it does not fill me with confidence.

Regarding **Bug Tracking**, I have also been checking out the site as I saw a post from one of the marmalade team saying they knew they were not perfect but were trying to implement bug trackers etc - I clicked through to their bug tracking page and was quite surprised to reach a [google doc](https://docs.google.com/spreadsheet/ccc?key=0AvXsJhJ4hVdodFhCSXF4LVJ4NElpWE4td3d2aW15dGc#gid=0) that seems to be populated from JIRA. I find this quite weird as most code services I use seem to use a public bug tracker and enable users to vote for and discuss issues etc - a strange one this. 

Regarding **Forums**, looking at their '[Marmalade Answers](https://devnet.madewithmarmalade.com/questions/1089/welcome-to-marmalade-answers-please-read.html)' site it says go to the forums for more general discussion - but when you go to the forum its closed, looking back on the previous page there is an edit below that said the forums closed in April and they are setting up a new one - considering its now the end of June thats quite a while ago.

Looking at **Support**, oh my. I went on the Answers site referenced above and clicked on 'most voted', guess what the top topic is titled! ["Why is marmalade support so poor?"](https://devnet.madewithmarmalade.com/questions/7813/why-is-marmalade-support-so-poor.html). The answers from the Marmalade team are quite shocking / rude and people are complaining again about regressions and so on - but the scariest thing is they say that even splash screens and screen rotation are broken - that is a really bad sign. A quote from the *Marmalade team*...

>didn't get You question. this will never change - products will always have bugs, their fixes will never be instant, and there will be never someone to cry on his shoulder... is this what You wanted to hear? could You focus us on something more specific? could You at least share Your experience? who provides better support? how quickly bugfixes are produced? does it cost less? etc.

Ok fair enough I dont know the nature of the bugs and how reasonable the complaints are but considering this is the highest voted issue on the site - this is a bad way to respond to it! It also seems that you need the top tier $1k a year account to get any support outside of the public forums. Ouch.

I imagine that someday that the management will realise what they need to do and start offering support and proper regression testing - but until that day this just feels too risky - especially as its not open source so you cant just fix yourself - a real shame.

###Hello World

Well I installed it and the RAD component Quick, and I cant even create my HelloWorld project, I just get a blank error dialog (well thats useful!). I reinstalled everything making sure I hadent missed any steps and stil the same issue. I try and run the samples and I just get a 'failed' message with nothing on the console/log or trace window. This sucks a little. I also just received 2 emails from them as there newsletter has messed up. From the above research and my own attempts to run it I really get the feeling their QA process in minimal. I [posted a question](https://devnet.madewithmarmalade.com/questions/13581/error-when-creating-new-project-using-quick.html) on the 'Answers' site to find out whats going on and 3 days later still no answer to what should be a pretty simple issue. I guess I shouldnt be suprised from what I have read. Pretty poor dev practise that led to a blank dialog box IMO! As means I cant trial/use it at all it its pretty crappy.



* * *

##LibGdx
<http://libgdx.badlogicgames.com/>

_EDIT: I didnt really look into this much as its Java based and I mentioned above that I wanted to learn a new language as well so its sort of tainted my view here maybe! I had another look into this and Im really impressed and it has moved up in my estimations by a long way. Also they have redone their website since I started writing this post and it looks much nicer / more accessible_

###Good

- Windows, Linux, Mac OS X, Android, iOS and HTML5 (WebGL)
- OPEN SOURCE
- A [LOT of games](http://libgdx.badlogicgames.com/gallery.html) made with it. I dont see many big ones - from a quick scan I recognise Apparatus which is cool. 
- Since I started writing this post it seems they have a new web site up - looks a lot better than the old one
- Graphics: OpenGL 1.1 & 2.0 - can make openGl calls (i.e. not heavily abstracted)
- Audio: comprehensive and functional audio lib (+FFT)
- Physics:Box2d and others
- [Whole load of other stuff](http://libgdx.badlogicgames.com/features.html)!
- Pretty busy [forum](http://www.badlogicgames.com/forum/)
- 2D & 3D (high level 2D api too!)
- High and low level API's - most frameworks seem to only have one level - this is great.
- Big [Wiki](https://code.google.com/p/libgdx/wiki/)
- Proper [bug tracker](https://code.google.com/p/libgdx/issues/list)
- Android 1.6+

###Bad

- __Personal one__ but I want to something other than Java for a while
- No RAD component as far as I can see (as Java based). A compile required each run? As its not mobile-only this can be on the desktop which is going to be faster than mobile only at least - no packaging and installing each cycle
- Running on a few layers on iOS (see below for more)
- Slightly restricted Java API usage for iOS
- iOS support is paid ($299 a year). Again see below.

###Overall

Im sure this is good for lots of people but an alarm bell rings in my mind when I come across a cross platform Java framework. Maybe this stems from the issues I had when writing a game Applet and getting it to work everywhere and the issues I had with JVMs accross platforms, this was partly to do with running inside a browser however. This could be rubbish however as obviously Java was built for cross platform. Once bitten I guess. I realise this could be irrational so may give it a go at some point.

It seems to use a ported JVM ([IKVM](http://www.ikvm.net/)) via <del>MonoTouch</del> [Xamarin.iOS](http://xamarin.com/monotouch) (rebrand) for iOS. This is a JVM written in .NET and then the Xamarin lib is used to run/port/recompile the C# code for iOS (presuably there is a .NET layer written some C variant) - this scares me as every layer can create more hassle when debugging and more complexity, but maybe my fear is unfounded and the layers are stable (would be good to hear from anyone regarding this). 

The framework is free (thanks guys!) but costs $299 a year to support iOS via <del>MonoTouch</del> [Xamarin.iOS](http://xamarin.com/monotouch), which is a little expensive but not a deal-breaker. There are [moves to go over to the FREE RoboVM](http://www.badlogicgames.com/wordpress/?p=3063) for iOS support which is very cool.

I want to do something other than Java for a while! Defo something I will have a go with in the future.

###Related Links
- [Is libgdx usable for 2d games?](http://stackoverflow.com/questions/10335423/is-libgdx-usable-for-2d-games)
- [LibGDX, Legends of Yore and getting to iOS (again)](http://www.cokeandcode.com/main/libgdx-legends-of-yore-and-getting-to-ios-again/)


* * *

##Loom
<http://theengine.co/loom>

###Good

- Code *and* asset hotswapping
- $40 a month
- You only have to subscribe for one month and you get all the code which they say you can use to build with forever (you just dont get the bugfixes and new features) which gives you options and is great.
- They seem pretty flexible on how you use it / modular and can add in your own stuff
- Integrated unit test framework
- Empty loom app only 2mb

###Bad

- I personally dont like the fact they have their own scripting lang (mix of Lua / C# / AS3 / JS) - i dont want to have to learn one lang for one framework. I imagine I could just use Lua however so maybe this is a silly comment.
- The [language docs](http://theengine.co/wiki/loomscript_language_reference) are incomplete and as it's their own lang it will quite hard to work it out without docs! 
- Android 3.2 and up - cuts out 40% of market (2013/07)
- API docs are sparse

###Other

- Game Framework: Smash/PushButton Engine
- UI lib using LML/CSS
- Physics: Chipmunk
- Audio: Simple implementation [so far](http://theengine.co/forums/loom-with-loomscript/topics/the-api-discussion-thread-sound-audio-cocosdenshion-simpleaudioengine)
- OpenGL:

###Overall

They are a relatively new company it seems and I worry about building on top of their platform and then they disappear. I guess this is an issue with anyone who is not open sourcing the engine / environment. 

[A Closer look](http://www.gamefromscratch.com/post/2013/03/22/A-closer-look-at-the-Loom-Game-Engine-The-Conclusion.aspx) on gamesfromscratch.com



* * *

##Unity
<http://unity3d.com/>

###Good

- Very established and stable
- [Lots (and lots) of impressive games](http://unity3d.com/gallery/made-with-unity/game-list). Some mobile and 2D, for example Bad Piggies.
- I hear its free for iOS and Android at present
- big community (forum areas have activity within mins as opposed to days). Even plugins like [2DToolKit](http://www.unikronsoftware.com/2dtoolkit/) have [more active forums](http://unikronsoftware.com/2dtoolkit/forum/index.php/board,2.0.html?PHPSESSID=1c7764152c32c82db8461357b79f6d1a) than most or all of the other engines in this post!
- loads of plugins
- Large company (200+) so does not look like its going anywhere soon
- Chipmunk physics coming soon. See this [Kickstarter](http://www.kickstarter.com/projects/howlingmoonsoftware/chipmunk2d-for-unity?ref=thanks) 
- Looks like good docs (and hear that the api docs are all commented with lots of examples)
- Can implement OpenGl (using [GL](http://docs.unity3d.com/Documentation/ScriptReference/GL.html) class) and write custom Shaders
- Has an Asset store for tools / textures etc


###Bad

- min 10mb app size for non-pro accounts
- Some features are pro account only (which is _really_ expensive for my needs at $4.5k)
- (a personal one) I dont like using / learning UIs! I would much rather work in my normal IDE than learn a busy interface. Hopefully Unity's will be well thought out. I think I will not be using a lot of features as going for 2D and low performance mobile devices so I wonder how much clutter there will be.
- (another personal one) I am an Android dev really and I use IntelliJ - Unity uses C# and comes with the MonoDevelop IDE. Annoyingly I cant use IntelliJ for C# and I dont really want to switch IDE!

###Languages

- C# (most used), JS, Boo Script (least used)

###Further Research

Im turned off by this as it seems to be for 3D primarily. Some people say they have success building 2d games but a lot more say its overkill using unity for 2d. I also feel that learning to use Unity will not give me as much transferable knowledge / skills as some other framework / engines (this maybe unfounded). If I was going 3D this would be my first port of call for sure.

_EDIT:_ After talking to a few devs who use Unity for 2D full time I may have another look at it. They were previously working with Cocos2d-x and say **unity is much quicker for dev time**. They have a [2D Toolkit](http://u3d.as/content/unikron-software-ltd/2d-toolkit/1Wi) to make things easier, this is a $65 extension. 

One of the newer 2D games to come out built on top of Unity is [Gun Monkeys](http://store.steampowered.com/app/239450), which looks great. To see more about 2D dev read [The Swindle by Size Five Games](http://unity3d.com/gallery/made-with-unity/profiles/sizefivegames-theswindle) and [Using Unity3D for The Swindle’s 2D](http://www.sizefivegames.com/2012/11/07/using-unity3d-for-the-swindles-2d/). 

Note that some of the effects and post-processing effects ([see this forum post](http://forum.unity3d.com/threads/188953-Post-processing-effects-using-shaders) they used are Unity Pro extensions - Unity Pro costs $1500 and then its $3k for Android and iOS support so this does **not come cheap**!! These are perpetual licenses however - but does not entitle you to the next major upgrade. I have Unity 3.x with Android and iOS extensions and that only gives me $500 off an upgrade! As Im looking for Android and iOS this does not matter _that_ much as post-processing effects are discouraged on mobile due to hardware speed (see [here](http://docs.unity3d.com/Documentation/Manual/iphone-OptimizedGraphicsMethods.html) for more).

Some **great 2D plugins** exist for Unity like the aforementioned 2DToolkit. There is also [Smooth Moves](http://forum.unity3d.com/threads/124211-Smooth-Moves-2D-Skeletal-Animation) for charecter animation. These are paid extras but under $100. 

I also just went to a free Unity workshop yesterday (which was good timing as I am writing this!) and was talking to some of the Unity team and they said there are some **new 2D tools coming soon** but they were not allowed to say much more than that. 



* * *

##PlayN
<https://code.google.com/p/playn/>

PlayN is cool - it was used for the Angry Birds in Chrome stuff a few years ago at Google IO and is also being used by King.com in Europe.

###Good

- Open Source
- A few big names use this

###Bad

- Like LibGdx this is is Java and supports iOS via MonoTouch (personal unfounded irrational distrust here!)

###Overall

This looks pretty cool - I guess it does not excite me as much due to it being in Java which I have been using for years do I wont be learning as much new stuff. Will keep this is mind though and defo keep an eye on it.



* * *

##Gideros
<http://www.giderosmobile.com/>

###Good

- Cheap (Free even!)
- I see a lot of people talking about this one with positive things to say
- Lua
- RAD
- Audio: OpenAL access from Lua?
- <del>Similar in ease to Corona but [better performance](http://qr.ae/plnA3). Then again [in contrast](http://mastosch.wordpress.com/2013/03/19/corona-sdk-vs-gideros-in-performance-test/)</del>
- Physics: Box2D (people say is bloated)
- I like how the devs make a point of saying they will fix bugs before adding new features. A contrast to the other frameworks.
- Blog is busy - 5 posts in the last month
- Docs look pretty comprehensive
- Android: 2.2+ ([~98.5%](http://developer.android.com/about/dashboards/index.html) at time of writing)

###Bad

- Android & iOS only (Windows 8 & phone & OUYA on roadmap)
- Closed source
- Limited functionality similar to Corona? (hinted at [here](https://news.ycombinator.com/item?id=3986124))
- Still on OpenGL 1.1 (25/06/13)

###Overall

This looks interesting. I cant see any really impressive stuff made with it but maybe worth a deeper look. 

###Further Research

Run by a small Turkish company. General vibe on the web is good. They seem to have a lot on their [roadmap](http://giderosmobile.com/roadmap) which is great and you have access to their [labs](http://giderosmobile.com/labs) also.

Like Marmalade I thought I would check out the __forums__ to see what the general vibe is and so on. [The first thread I came across](http://www.giderosmobile.com/forum/discussion/3153/status-update-what-to-expect-when-you-are-expecting-opengl-es-2-) is very __positive__, the devs are announcing whats next and saying that openGL 2.0 support is delayed due to fixing bugs in the previous version - the exact opposite to Marmalade (which is great IMHO). 

UPDATE: __Good forum__. On first use I had an issue running the example code on my Android device - I posted a question on the forum and had a detailed troubleshooting list within hours - much better than Marmalade!

Reading further down it seems that users are getting a little upset due to lack of communication from the dev team and they mention they do not have lots of money behind them and therefore not communicate effectively enough to keep users informed. My worry with closed source is always that a project will cease to run due to financial reasons and current users will be stuck - so if they are saying this a _small_ warning sign for me, but on the otherhand it definatly seems like there is constant work on it and does not seem to be losing momementum like some other engines.

Push Poke seem to have had a [good experience](http://pushpoke.com/2013/02/why-did-we-choose-gideros/) compared to Corona. Some like its pretty each to customize for each platform also.

Also the examples and tools are **really easy to get started and up and running quickly** - of which for various reasons I can not say of Marmalade(broken SDK tools), MOAI (bit more complicated), UNITY (very powerful but also means bit more work to get up and running and to work out what I need to discount for 2D only stuff). 

I have played for a day so far with this one and am **really impressed**. I have been an Android dev for years now and the speed of getting something up and running using this is _much_ quicker. I look forward to playing with more!

* * *

##MOAI
<http://getmoai.com/>

###Good

- Lua & C++
- Open source
- used in some cool games (Strikefleet, DoubleFine are using for the kickstarted backed DFA which looks great)
- FREE
- Has a Lua backed cloud service for networked / cloud enabled apps (this is where they make their money i think)
- Supports all* apart from WinMo. Supports Web via Chrome native. 
- OpenGL ES 1.1 & 2.0. All openGL calls are in retained mode.
- Physics: Chipmunk (good!) & Box2D
- Audio: [Untz](http://getmoai.com/news/untz-audio.html) - you also have the option to use the commercial [FMOD](http://www.fmod.org/) (see tut [here](http://www.gamefromscratch.com/post/2012/09/17/Moai-Tutorial-Part-6-Who-put-the-bomp-in-the-Moai-Moai-Moai-Playing-audio-in%E2%80%A6-you-guessed-it%E2%80%A6-Moai.aspx)). Looks like they are already thinking about a more fuly featured audio engine [here](http://franciscotufro.com/tag/untz/)
- Simple 3D support which is great for the occasional 3d effect. See the 3D section on [this page](http://getmoai.com/blog/moai-2012-roadmap.html)

\*Note: The docs do however say the bundled desktop support is using GLUT and may not be the best choice for commercial desktop games and instead...

>In the Moai source tree, sample implementations are provided for iPhone, Android and GLUT. GLUT is a minimalist, open source, cross platform UI framework for writing OpenGL sample programs. As useful as it is for writing samples, GLUT is probably not the correct choice for writing a commercial desktop game. You should instead write a native host using a cross platform library such as Qt.

Taken from <http://getmoai.com/wiki/index.php?title=Moai_Project_Setup>

###BAD

- Some people say a little more work to get up and running than something like Corona.
- The forum has activity but seems a little quiet - I can see the last activity was 4 days ago on the whole forum! They do however offer [paid support](http://getmoai.com/pricing.html)

###Overall

Seems like there may be no RAD component but this may bot be as issue as maybe be very quick to get new Lua code up and running as no compile needed. DoubleFine are working on some RAD tools for this (called 2HB) which they said they will open source over the new year or so.

This one excites me, even with their not very good looking website. The fact its been used for some good games and has the interest of doublefine says a lot. Plus its open source so potentially any bugs can be fixed yourself.

I like the idea of using Lua as its used in so many places and Moai offers a thin wrapper over OpenGl (ES) so would go good to gain some exposure to those API's also. 

While I really want to use Moai I do worry as a one-man-band I may end up spending lots of time getting specific things to work (I have heard the Untz audio lib did not work very well on Android for example) and it worries me that the forums are quite quiet - not good when your in a fix! Also I have heard that only the iOS host is stable at present (Im sure this will change soon). I wonder if the OS project is losing momentum?

Even if I dont use this at the moment I will keep an eye on it to see what DoubleFine add to the mix. Its definitly a great starting point just have to see if it catches on a bit more and develops more of a community.

###More

- [Moai: First Impressions](http://dev.ionous.net/2012/07/moai-first-impressions.html)
- [Hanappe](https://github.com/makotok/Hanappe) game framework (community [likes](http://getmoai.com/forums/hanappe-framework-t707/))
- [Rapnui](https://github.com/ymobe/rapanui) another game framework
- [Some notes on Lua Architecture](http://getmoai.com/forums/architecture-question-t1920/)
- [Moai or Corona – Which SDK for You?](http://gamedevnation.com/game-development/moai-or-corona-which-sdk-for-you/)

* * *

##Coscos2D-x
<http://www.cocos2d-x.org/>

A cross platform port of the Cocos2d framework on iOS.

###Good

- [Supports C++ / Lua / JS](http://www.cocos2d-x.org/projects/cocos2d-x/wiki/Supported_Platforms_and_Programming_Languages)
- Supports Lua using [OpenQuick](http://www.cocos2d-x.org/news/88) (same code that Marmalade Quick is build upon)
- FREE
- Physics: Box2D and Chipmunk
- Busy forums
- Busy [open source project](https://github.com/cocos2d/cocos2d-x) 
- Looks flexible. Have access to source and can code at C++ level so should be able to do most things
- [Supports most platforms](http://www.cocos2d-x.org/projects/cocos2d-x/wiki/Supported_Platforms_and_Programming_Languages)

###Bad

- I have also read this is a bit messy to work work and not as nice as Cocos2D.
- Most people seem to use with Marmalade - so inherits all the good / bad points from there
- OpenGL ES 1.1 (Some may see this as a good thing)
- Not sure if can be used for RAD development - even though supports Lua bindings dont know if is as simple as things like Gideros / Corona / Quick. 

###Notes

As far as I understand it this is mainly implemented in C++ and can be used natively on most platforms. I think for an individual like me I would use in on top of Marmalade - but it seems I would be doing this mostly in C++. I should look into why some choose to use this as a layer on top of marmalade and some dont, I think its so you dont need to mess around with native bindings and so on. Someone told me they used this and then switched to Unity and found that much easier to work with. 

I need to look into this one a bit more...It looks good but I want something I can get up and running quickly and from a quick look (I havent gone through any examples) this seems like it would take longer to get into than something like Gideros - but maybe someone will tell me why I am wrong here (please do!).

- [Interesting article about using Cocos2d](http://www.cocos2d-iphone.org/badland-a-cocos2d-iphone-game/) (not x) with links for lots of 3rd party pool for game dev. 
- [Cocos2d-X by Example Beginner's Guide](http://www.amazon.co.uk/gp/product/178216734X) _I just bought this so should be able to update this with more info soon!_

#Summary

From the above (and my requirements and existing language skills) I think the following look most viable for me at present.

1. Gideros <del>Corona</del> _(Gideros seems to me a better alternative to this)_ 
2. LibGDX _(if devving in Lua is too slow for me as Im more used to Java I may fall back to this)_
3. Unity
4. Cocos2D-x 
5. Loom
6. <del>Marmalade Quick</del> _im skipping this for now due to the unhappy community surrounding it & poor support, maybe will look again in a year or so._

_UPDATE:_ I was hoping to to finish this comparison and research with a game engine in mind I can commit my time too but I think thats was maybe wishful thinking. Like programming languages there is not one tool that is best for every job so I guess it really depends on the fundamental requirement for using it. I think my two main requirements are Rapid Application Development (RAD) and a stable, reliable and flexible framework. At the moment Im now leaning towards using <del>Corona</del> Gideros for my rapid prototyping and then _maybe_ Unity for any actual releases if I need more platforms or features, but this may not be needed. Bit of a pain as its two frameworks but I cant see a better solution at the moment (unless I find the RAD speed in Unity compares to that of Corona)

##Related Reading

- [Battle of the Lua Game Engines: Corona vs. Gideros vs. Love vs. Moai](http://www.gamefromscratch.com/post/2012/09/21/Battle-of-the-Lua-Game-Engines-Corona-vs-Gideros-vs-Love-vs-Moai.aspx)
- [SDL Layer](http://wiki.libsdl.org/moin.fcg/Introduction) _Not really related but an interesting x-platform lib ([more...](http://gamedevgeek.com/tutorials/getting-started-with-sdl/))_
- [Moai or Corona – Which SDK for You?](http://gamedevnation.com/game-development/moai-or-corona-which-sdk-for-you/)
- [2D cross-platform game engine for Android and iOS?](http://stackoverflow.com/questions/12234457/2d-cross-platform-game-engine-for-android-and-ios)
- [Some reddit comments on this post](http://www.reddit.com/r/androiddev/comments/1ijzhg/a_blog_post_i_wrote_on_cross_platform_mobile_game/)






