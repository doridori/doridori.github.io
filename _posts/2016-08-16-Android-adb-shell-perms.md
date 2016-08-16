---
layout: post
title: "Android: A Journey Into ADB shell permissions"
---

I was chatting online about how adb permissions may differ from a shell open on an OEM device by an installed 3rd party application. I realised I did not really know how to quickly verify this so I went on a little journey.

First of all I had a flick through the excellent [Android Security Internals: An In-Depth Guide to Android's Security Architecture](https://www.amazon.co.uk/Android-Security-Internals-Depth-Architecture/dp/1593275811) to get an overview about how process permissions worked on android. Most of the overview is just a condensed version of a few pages of this great book.

This post will mostly talk about low how this stuff works with low-level permissions work in the OS, as opposed to high-level operations that involve the package manager (`pm`). 

## A brief summary of the low-level stuff is:

- Privileges are based upon the processess UID, GID and supplimentry GIDs
- Like all POSIX systems (clarification needed) access to system resources regualted by the kernel (files, sockets etc) is based on the owner and access mode of the resourse and the UID & GID of accessing process
- Some permissions on Android are mapped to GIDs [data/etc/platform.xml](https://android.googlesource.com/platform/frameworks/base/+/master/data/etc/platform.xml)
  - Other permissions are checked via `pm` and I'm guessing are not checkable by the process outlined in this post.
- These GIDs are mapped to AIDs [android_filesystem_config.h](https://android.googlesource.com/platform/system/core/+/master/include/private/android_filesystem_config.h)
- For applications (quick diversion) the package manager will add the GIDs for the application at install time, for permissions that appear in the `platform.xml` file, to `data/system/packages.list` for the applications entry.
- When a process is forked from the _zygote_ process, as it does for new application processes, the UID and GIDs are set. Kernel and system daemons use these to grant access to resources and functions.

## Relative shell permissions

What prompted me to look into this was the claim that a shell started from an application had the same permissions as a shell started via `adb`. This didnt seem right to me, so I had a peak using the above knowledge.

Starting with an `adb shell` we can see the parent process tree via a cut down `ps` output:

<div data-gist-id="21ce5f812bd0d9d43d5ff2a3fd28c4b5" data-gist-file="1">1</div>

And using this to look into the GIDs that the shell user has:

<div data-gist-id="21ce5f812bd0d9d43d5ff2a3fd28c4b5" data-gist-file="2">2</div>

We can compare this to a shell command from an installed application. Due to quirks with `Runtime.exec()` I found this simplest to do by issuing a slow shell command (`sleep 100`) from an installed app and inspecting from my already open `adb shell`. Below as expected, we see a totally differnt PPID tree

<div data-gist-id="21ce5f812bd0d9d43d5ff2a3fd28c4b5" data-gist-file="3">3</div>

And querying for GIDs we have:

<div data-gist-id="21ce5f812bd0d9d43d5ff2a3fd28c4b5" data-gist-file="4">4</div>

Where `10089` is the UID of the application that issues the shell command. 

From the above we can see that the GID permissions are not infact identicle, and that the users are also differnt!

One obvious example of this is that the `adb shell` has GID `3003` which is 

`#define AID_INET          3003  /* can create AF_INET and AF_INET6 sockets */`

so by default it will effectivly have the `INTERNET` permission. 

It would be interesting to know what the os resources the `shell` user has access to. I imagine this can be found by a simple command on a rooted / emulated device.

## Last Word

I found this an interesting poke around kernal permissions (a sentence I never thought I would say!) so thought I would share. If you see anything incorrect here please let me know. 

## Appendix: ro.*

A side note here. According to [ADB
(Android Debug Bridge):
How it works?](https://events.linuxfoundation.org/images/stories/pdf/lf_abs12_kobayashi.pdf) if the sys prop `ro.secure` = 1 (as it should be with all OEM devices) than `adbd` will run as `shell`, otherwise it will run as `root` i.e. like the emulator, and probably the now defunct dev devices.
