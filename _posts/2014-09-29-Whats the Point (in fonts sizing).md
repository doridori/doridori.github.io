---
layout: post
title: Whats the Point in font sizing?
---

### TL;DR

_The 'Point' unit of measurement has more than one value. Apple use their own point unit terminology. Android has a more complicated and powerful concept of density-independence. If using a design doc with pts you need to be aware of the docs dpi as on Android you need to map between this and the android baseline dpi. Quick conversion for Android __1pt = 1px = 1dp IF docDensity = 72dpi AND docResolution = 320x480px___

# QUICK CONVERSION TABLE

The approach I am using is

	(docDpi / 72) = px per point size = ppps
	ppps / androidDensity = PtMultiplier
	PtMultiplier * docPtSize = AndroidPx
	AndroidPx = AndroidDp

<br/>

| PS doc dpi* | PS doc res | designing FOR Android density | PS Pt size | Formula            | Android Dp/Sp  |
|-------------|------------|-------------------------------|------------|--------------------|----------------|
| 72dpi       | 320x480    | MD (160 dpi) 1x               | 10         | (72/72) * 1 * 10   | 10             |
| 72dpi       | 480x720    | HD (240 dpi) 1.5x             | 15         | (72/72) / 1.5 * 15 | 10             |
| 72dpi       | 480x960    | XHD (320 dpi) 2x              | 20         | (72/72) / 2 * 20   | 10             |
| 160dpi      | 320x480    | MD (160dpi) 1x                | 10         | (160/72) / 1 * 10  | 22.2           |
| 320dpi      | 640x960    | XHD (320dpi) 2x               | 10         | (320/72) / 2 * 10  | 22.2           |

\* could also be named 'Designing AT density'

# EXPLANATION

This is intended as a small note regarding how to harmoniously discuss Point sizing between UI designers, iOS devs and Android devs. I found out the below after a confusing few hours trying to decipher a design document supplied by a client that specified font sizing in `Points`, which worked fine for the sister iOS app but was crazy-large on the Android app I was working on. If I had read this post before hand (which would have involved a time machine) it would have made things much clearer. I wrote this as I was looking into the subject so Im sorry if I have made a seemingly-simple subject more confusing!

Worth mentioning I am an Android dev so please point out any iOS related errors in the post. For that matter, please point out _any_ errors about anything!

Forgive the list approach but here is a short primer.

## Prerequisites

#### 1) A Digital Pt (for type)

The word __Point__ (__`pt`__ from now on) in relation to font sizing in the Digital word is generally referring to [1/72 of an inch](http://en.wikipedia.org/wiki/Point_(typography)). _This has the property that it should not matter what the actual device display density is_, as this is irrelevant.

#### 2) Apple's Point

Apple, use the word __`Point`__ to refer to another more general unit of measurement on iOS. See ['Points Don’t Correspond to Pixels'](https://developer.apple.com/library/mac/documentation/GraphicsAnimation/Conceptual/HighResolutionOSX/Explained/Explained.html). This is a step towards density independence on the system, but is a sort of halfway house. It seems that developers can work in points rather than pixels where one point is equal one pixel on normal resolution devices, and 4 on high res (i.e. retina). But this is not true density independence as there is no garentee of physical screen size of the final displayed object, it is basically abstracting over the device display without much control of final physcial display size. Setting a point size to a view or font on iOS always means apples `point` size and __not__ the traditional `pt` being discussed in this post.

#### 3) Photoshop default

Most UI designers I know of at present use Photoshop and stick to the default document setting of __72dpi__. [This is probably due to legacy monitor hardware specs](http://www.photoshopessentials.com/essentials/the-72-ppi-web-resolution-myth/).

#### 4) Mobile design defaults

Mobile UI designers generally target xHD (Android) / Retina (iOS) mobile devices.

#### 5) Android DPs

Android has a nice way of abstracting over density by using __DPs__ / __DiPs__ (Density Independent Pixels). This allows a dev to specify view dimensions as if they were normal pixels on a 160dpi MDPI (medium density) device. These roughly correspond to an actual physical size on device output (due to the way android buckets screen sizes and densitys there is a discrete scale of densitys used for this computation so there are small differences between actual devices - so really it gives a small range of outputs)

#### 6) Mobile Hardware DPI baselines

iOS and Android baseline original devices have ~150-160 dpi known as Medium-Density (Android).

## The Problem

Points 3 & 4 combined means its common in the mobile dev world to come across ui design for mobile screens with something like a PS doc with 72dpi and doc size of 640x960 (or something similar). While this may look like a mobile size screen when zoomed out inside PS and pixel dimensions of views etc can be used as-is this is unfortunatly not the case for fonts defined in __Pts__. 

A doc of the size mentioned above is actually in real world sizes `640 (doc px) / 72 (doc dpi)` inches wide, which is __~8.9 inches__. This is obvioulsy larger than the mobile screen its being designed for, which is probably in the 3-4" range as a minimum. If a `pt` size is taken from this doc it will probably be ~3 times bigger than desired (based on average current smartphone screen size).

## So whats the solution? 

### Solution 1: Work out what the `pt` conversion rate should be to match the DPI bucket of the target device. Result __`Pt`__

If you get supplied a design doc with iOS `points` for a multi-platform app you will need to know what the PS docs settings are - but a sensible guess IMHO at time of writing would be 72dpi with 640px width (for phones) for reasons described above. To convert the point size from this doc to something usuble on you device you would do something like

_(the below will assume a doc with 72dpi, 640px width with text at 60pt)_

#### Convert from a XHD (x2) design doc to MD (x1)

60dp / 2 = 30dp

#### Get density differnce factor 

160dpi / 72dpi = ~2.22

#### Device given pt size by this

i.e. 30pt / 2.22 = 13.51pt

You could then go on to work out what SP to use on Android pretty easily from the corrent pt size (as DPs work based on 160dpi)

### Solution 2: Use a design doc with the "correct" dpi. Result __`Pt`__

The PS doc should reflect a real devices settings

What happens if we use a doc with a sensible DPI that you would find on a current day mobile device. If we change the doc to 160dpi and keep the size at 640x960 we still have a device width of 4', which is too large for your general medium sized smartphone. This is becuase we are using a MD (medium density (160dpi) setting with a typical XHD / Retina (640x960) resolution. 

Mixing the correct DPI with typical device resolution for that DPI would be a good start. So for example

- 160dpi 320x480px
- 320dpi 640x960px

Would then allow you to use the `pt` values directly from PS in your android app.

### Solution 3: Convert to px / dp / sp for android. __Simplest.__ Result __`px / dp / sp`__

The easiest solution is to use a non-pt dimension when creating your android layouts. If the PS doc has the default 72 dpi then you can just

1) Look at the doc size
2) Translate the PS pt into an android DP
3) Use an SP in place of the DP if required

__Example A__ - doc size 320x480 for a medium phone size

PS text at 24pt would be equivilent to 24px in the PS doc. The resolution would be medium-density (MD) @ x1 which means that 1 px = 1 dp. Therefore 24pt would equal 24dp. 1 sp = 1dp at user default text scaling settings so using 24sp would adhere to the supplied design and also accessability guidlines.

__Example B__ - doc size 640x960 for a medium phone size

PS text at 48pt would be equivilent to 48px in the PS doc, however as the target resolution would be extra-high-density (XHD) @ x2 this would require 24px (48px/2) as the android text size.

As this seems to be the most-common situation dividing by 2 is a simple general rule to follow.

## Afterword

#### 1pt != px

Due to PS defaults being 72dpi and the fact that a __pt__ = [~1/72"](http://www.oberonplace.com/dtp/fonts/point.htm) this means that you will hear a lot that __1pt = 1px__ when looking this stuff up online (due to the fact that the design doc has matched the two). 

This is a dangerous assumption to have and it muddies the water in some ways as its ignoring the fact that really `pt` is a __density-INdependent__ measurement while `px` is obviously __density-dependent__. 

A more accurate rule would be __1pt = 1px IF docDensity = 72dpi__ and in addition __1pt = 1dp IF docDensity = 72dpi AND docResolution = 320x480px__

Example: A 320x480px 72dpi doc has some text at size 72pts. This would be an inch wide when viewed at 100%, but when the doc is scaled so it has a virtual density of ~160dp, 72pts would be 72/160th (~1/2.22) of an inch. This text would then equate to 72dps. 

For practical purposes 1pt = 1px when the photoshop doc is using 72dpi and the dimensions of the doc are equal to the dimensions of the screen you are coding for, but this does NOT take into account any density-independence, hence the above solutions will need to also be taken into account if any density-indendent dimension is to be used. You _could_ use the density-dependent `px` size directly but this will only display at the right size when your output device exactly matches the dimensions of the PS doc.

#### Emulated Dpi

Even though the DPIs are differnt the actual target devices DPI is accounted for visually by the user zooming out of the PS doc until the _emulated DPI_ equals that of the target device. Its worth noting that this is a sort of fragile relationship to rely on without thinking about it too much as as soon as an PS doc with a differnt DPI is encountered this can break down.

#### Android Density Buckets 

Android has discrete density buckets shown below

- ldpi (low) ~120dpi
- mdpi (medium) ~160dpi
- hdpi (high) ~240dpi
- xhdpi (extra-high) ~320dpi
- xxhdpi (extra-extra-high) ~480dpi
- xxxhdpi (extra-extra-extra-high) ~640dpi

_Taken from [here](http://developer.android.com/guide/practices/screens_support.html)_

Devices are placed into one of these buckets and all UI computation and scaling is based off the bucket. The calculation of what bucket the device should be placed is it not a purely deterministic one based upon the devices physical properties, as one would hope. AFAIK it is based upon device metrics set by the OEM. These are usually correct but sometimes you get devices with a `Medium` screen size reporting as `Large` or one which is closer to one density bucket reporting as another. Thankfully these seem to be in the minority but worth taking into account that there will be small variations in sizing on Android devices due to this with the occasional outlier.

See [this Stack answer](http://stackoverflow.com/a/3414215/236743) for an example of some devices allocated density bucket being differnt to what it should be / what would make more sense. 

## Finally

If you managed to stay with this exiciting subject this far and you have any thoughts / have noticed something I have got the wrong way round or does not make sense, please let me know! :)


