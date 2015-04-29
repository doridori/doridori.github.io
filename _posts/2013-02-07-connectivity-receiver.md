---
layout: post
title: "Android: Connectivity Receiver"
---

Here is a small snippit of how I handle Connectivity within some the applications I work on. A lot of apps dont seem to handle loss of connection correctly IMHO by either not showing a relevant error message or informing the user that there is an issue. Depending on the type of application, especially one that is meant to have a persistant network connection it can be useful to show some form of visual feedback when this is not the case. 

The below is based upon [Determining and Monitoring the Connectivity Status](http://developer.android.com/training/monitoring-device-state/connectivity-monitoring.html)

The `ConnectivityReceiver` helper class I use can be found [on github](https://github.com/doridori/AndroidUtilDump/blob/master/android/AndroidUtilDump/src/couk/doridori/android/lib/io/ConnectivityReceiver.java).

The javadoc pretty much says it all but points to note are

- Remember to hook up to your `Activity`s lifecycle methods
- Note that this is and always has been based upon a **Sticky** broadcast but this is not noted in the [Android documentation](http://developer.android.com/reference/android/net/ConnectivityManager.html#CONNECTIVITY_ACTION). Something to be aware of that is mentioned in the `ConnectivityReceiver` class doc.

I have used this in the past by incorperating into a base class and using to trigger a [`Crouton`](https://github.com/keyboardsurfer/Crouton)

This is the way I default to handling connectivity issues but I would be interested to hear of other approaches - let me know if you do it another way!! 

**EDIT**: (10/06/13) I have just seen that the guys at [Novoda](http://novoda.com/) have created a small library to do this too. See [Merlin on Github](https://github.com/novoda/merlin?utm_source=Android+Weekly&utm_campaign=2d2c5bbf26-Android_Weekly_63&utm_medium=email&utm_term=0_4eb677ad19-2d2c5bbf26-310455189).
