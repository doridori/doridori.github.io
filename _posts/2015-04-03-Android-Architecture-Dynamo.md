---
layout: post
title: "Android Architecture: Introducing Dynamo"
---

Welcome, welcome. Come in, it looks mighty cold out there. The code is laying heavy across the hills, disorder rules the day and the nights are long. We yearn for a day where the suns shines forth once more.

This is post #2 of my very short Android Architecture series (can two be a series?). Post #1 was a primer for this post. It contained

 1. A brief outline of the problems when defaulting to stock Android architecture.
 2. An overview of common general architectural concepts. 

This post is really a short intro to a small library + Wiki I have created. This library fills an architectural hole which I see in the Android dev world. There is no unified approach to generic Android app architecture and this creates a vacuum which sucks in well meaning devs and results is messy code bases strewn across the land. 

Plus, I feel sorry for new Android devs who turn are turning up later to the party and are overwhealmed by the Android ecosystem and need to get up to speed quickly on what these issues are before even getting to the point of thinking about potential solutions. Some of the issues address by this approach are solved by fantastic existing librarys but again, I think for a large chunk of devs these can be difficult concepts to grasp and get running with. These are mentioned in the projects wiki for further reading.

In my experience at least there are two core ideas around the center of most apps codebases. From the code-bases I have seen & inherited both of which have little or no thought applyed to them.

#1. State

Most apps and views are *state-based* however the speggetti-level that captures the state logic is generally very high.

#2. Asynchronous code

Pretty much every app involves some asyncronous code. Pretty much every app allows this to touch the Android lifecycle. This _always_ leads to sadness.

_I spoke about this is the first part of [this brain dump of a blog post](http://doridori.github.io/Android%20App%20Architecture-%20Lifecycle%20Events%20and%20Asynchronicity/)_

#Introducing Dynamo

<img src="https://github.com/doridori/Dynamo/blob/master/gfx/DynamoDroid.png" alt="dynamo droid"/>

[`Dynamo`](https://github.com/doridori/Dynamo) is a small library and wiki I have created to show my approach to an Android architecure that works with these two bug-spawning-hair-loss-inducing areas. 

It will reduce your `Activity`, `Fragment` and `View` classes to dealing with View behaviour and States only via a simple interface, for example

```java 
public ComputationActivity
{
	...


	@Override
	public void onState(ComputationDynamo.UninitializedState state)
	{
	    mResultTxt.setText("Ready & waiting");
	}

	@Override
	public void onState(ComputationDynamo.PerformingComputationState state)
	{
	    mResultTxt.setText("Thinking...");
	    mGoButton.setEnabled(false);
	}

	@Override
	public void onState(ComputationDynamo.ComputationFinishedState state)
	{
	    mResultTxt.setText("I think its "+state.getResult()+"!");
	    mGoButton.setText("Compute another!");
	    mGoButton.setEnabled(true);
	}
}
```

Enjoy!



