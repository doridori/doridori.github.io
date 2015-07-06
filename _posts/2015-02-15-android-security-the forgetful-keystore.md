---
layout: post
title: "Android Security: The Forgetful Keystore"
---

You've just moved in to a new house and have been given the master key for the front door. You only have one of these so you know you need to keep it safe. Your really paranoid so you hire an armed guard, whose sole job is to protect this key, in fact, this is all he has been trained to do and has a catchy slogan of "need to protect a key, its what I was born to do!". You install an extra lock on your front door as you feel the bodyguard isnt enough, this is a rough area anyway and who's going to make sure no-ones about to break in and steal all your crap. You return to your key guard only to be informed he has thrown the key away. You shout and scream at him but he just blankly says "I dont have it anymore, I didnt think it was important". You can't contain your anger "What the hell, your a jerk! You had one thing to do and you failed, this causes me a lot of problems, why didnt you tell me you might do this?! What do I do now?!"

Ok, stories arnt my strong point. The point is, this is how I feel when using the Android `Keystore` to protect my private encryption key, it drives me a bit nuts. I really appreciate the effort thats gone into it but its still frustrating when your on a deadline, documentation is lacking, and something isn't working as it should 

I have been working on some apps with a high security requirement, and one of the requirements is to take advantage of the best level of security offered by the platform. For our case this includes utilising the system `Keystore` to store an asymmetric key pair. One can then use this for encryption/decryption directly or use it to encrypt an symettric key (like AES) which can then be used to encrypt your plaintext, which is much faster than using the asymmetric directly. 

Lots of people seem to use encryption in the wild but they hard code the key or store is as plaintext or obfuscated. None of which are recommended.

#Why is the `Keystore` good for storing keys?

Good question! The Keystore can hold its keys is two ways. 

##Software

Key access is stored on the file system with access given via the OS, only allowing an app with the creating apps UID to access. This is bascially how Android file-permissions work in general. Stuff like this is the first to go on rooted devices as afaik `su` can adopt any UID.

##Hardware

I think the Nexus 4 was the first device to offer this, with APIs available from 4.3+.

> "Android also now supports hardware-backed storage for your KeyChain credentials, providing more security by making the keys unavailable for extraction. That is, once keys are in a hardware-backed key store (Secure Element, TPM, or TrustZone), they can be used for cryptographic operations but the private key material cannot be exported. Even the OS kernel cannot access this key material. While not all Android-powered devices support storage on hardware, you can check at runtime if hardware-backed storage is available by calling KeyChain.IsBoundKeyAlgorithm()" from [Release notes for 4.3](https://developer.android.com/about/versions/android-4.3.html#Security)

##API

The api method im using to generate the key pair uses `KeyPairGeneratorSpec` which was added in 4.3 (18). There were ways to use this stuff before then (see Nikolays blog links below) but im going to ignore that for now.

#The Problem

"This sounds great" I hear you cry. Well yes it is, until the Keystore-stored-key becomes inaccessible, leaving you with a useless steaming pile-o-bits™.

Currently this can happen when the device security settings (device-lock) are changed by the user, namely switching between `None`, `Swipe`, `Pattern`, `Pin` and `Password` but also OS versions will behave differntly. 

Reading the [related android bug-tracker list](https://code.google.com/p/android/issues/list?can=1&q=KeyPairGeneratorSpec&colspec=ID+Type+Status+Owner+Summary+Stars&cells=tiles) some cry 'its a bug' and others 'its a feature'. If its a feature documentation is much needed, if a bug then fixes are welcome. From this point the most I can do it outline what I have seen so as to enable others to work around it / be aware.

If you do get into the position of having a wiped keystore I have seen this manifest in two ways (in my initial testing):

1. Most of the time ~98% in my original testing the key data _AND_ alias is wiped - giveng a `InvalidKeyException` at point of use. 
2. A small amount of the time ~2% the key data is lost but the alias is _not_ giving `IllegalArgException`

See below for the `InvalidKeyException` source.

<div data-gist-id="cfe0fca74a9c05fc6a57" data-gist-file="InvalidKeyException">InvalidKeyException Example</div>

From the above you can see it's not just a silent fail yielding an empty keystore but an undocumented-exception-throwing fail which requires you to reset the alias ([Keystore.deleteEntry()](https://developer.android.com/reference/java/security/KeyStore.html#deleteEntry(java.lang.String))).

#Device locks & `.setEncryptionRequired()`

Its worth noting that when creating a key with [`KeyPairGeneratorSpec.Builder.setEncryptionRequired()`](http://developer.android.com/reference/android/security/KeyPairGeneratorSpec.Builder.html#setEncryptionRequired()) its required that the device have a device-lock set (as this is used as part of the keying material). If you do `setEncryptionRequired()` and one isnt, you get a friendly `java.lang.IllegalStateException: Android keystore must be in initialized and unlocked state if encryption is required` so you need to manually ensure that something is set. Check out [my SO answer](http://stackoverflow.com/a/27801128/236743) for an approach to use here :). The great [Android Security Cookbook](https://www.packtpub.com/application-development/android-security-cookbook) has a recipie for using the Device Admin Policy.

#Behaviour Matrix

As mentioned above different OSs and differnt transitions between device-lock states can wipe the keystore. 

I have written a small test app and run this on a few devices and emulator to see what results I get. This test app will be using the [`SecretKeyWrapper`](https://android.googlesource.com/platform/development/+/master/samples/Vault/src/com/example/android/vault/SecretKeyWrapper.java) from the Android `Vault` package examples. I have added two methods to this to enable testing. See the full (dirty) source on [github](https://github.com/doridori/AndroidKeystoreWipeTest/tree/master).

See the tests below. T = Pass, F = Fail. Some cells are blank as I didnt test that combo - but there is enough there too see the pattern in each case :)

**EDIT 29/05/15:** _This post needs to be updated with M preview data. They have mentioned that_ 

> Keys which do not require encryption at rest will no longer be deleted when secure lock screen is disabled or reset (for example, by the user or a Device Administrator). Keys which require encryption at rest will be deleted during these events. [link](https://developer.android.com/preview/behavior-changes.html#behavior-keystore)

_It will be interesting to run the below tests on the M preview._

##Nexus 4 | 5.0.1

| to ↓        from > | NONE | PIN | PASS | PATTERN |
|--------------------|------|-----|------|---------|
| NONE               | N/A  | N/A | N/A  | N/A     |
| PIN                | **F**| T   |      |         |
| PASS               | **F**|     | T    | T       |
| PATTERN            | T    | T   |      |         |

Once a keypair has been generated the system will not let you revert to NONE unless the app has been deleted / the pair removed hence the N/A

**`.setEncryptionRequired()`**

| to ↓        from > | NONE | PIN | PASS | PATTERN |
|--------------------|------|-----|------|---------|
| NONE               | N/A  | N/A | N/A  | N/A     |
| PIN                | N/A  | T   | T    | T       |
| PASS               | N/A  | T   | T    | T       |
| PATTERN            | N/A  | T   | T    | T       |
 
More N/As on this one as `.setEncryptionRequired()` will throw if you try to create a keypair with a NONE state.

**EDIT** It seems that there are still issues on 5 when using non-primary user accounts, which seems to behave like older OS versions and will still wipe the keystore :( See [this bug](https://code.google.com/p/android/issues/detail?id=61989#c21)

##Nexus 7 | 4.4.4

| to ↓        from > | NONE | PIN | PASS | PATTERN |
|--------------------|------|-----|------|---------|
| NONE               | T    | F   | F    | F       |
| PIN                | F    | F   | F    | F       |
| PASS               | F    | F   | F    | F       |
| PATTERN            | T    | T   | T    | T       |

4.4 Does seem to allow you to switch back to NONE unlike 5 above
 
**`.setEncryptionRequired()`**

| to ↓        from > | NONE | PIN | PASS | PATTERN |
|--------------------|------|-----|------|---------|
| NONE               | N/A  | F   | F    | F       |
| PIN                | N/A  |     |      |         |
| PASS               | N/A  |     | F    | F       |
| PATTERN            | N/A  | T   |      |         |

Like 5.* generating the key pair with NONE set will just throw
 
##Genymotion Emulator | 4.3

| to ↓        from > | SLIDE | PIN | PASS | PATTERN |
|--------------------|-------|-----|------|---------|
| SLIDE              |       | F   |      |         |
| PIN                | F     | F   | F    | F       |
| PASS               | F     | F   | F    |         |
| PATTERN            |       |     | T    | T       |

Sad face.
 
#Conclusion

Knowing the above I see a few potential solutions

1. If your requirement is to have strong encryption using the keystore and losing the encrypted data is a no-no - targeting L+ (5+) seems like the only choice, using `.setEncryptionRequired()`. However be warned that this is no guarentee that it will always work as this seems to still be broken for non-primary user accounts.

2. If your requirement is to have strong encryption using the keystore and losing the encrypted data is ok - then you can detect the above exceptions mentioned and have some form of re-inititialisation for those cases. Target 4.3+. Remember to check the device has the required security settings before generating the keyPair.

3. Look at another way of storing key data, for example generating one from a users input. This will not suit all applications of course. Target any platform version.

If you are planning on using the system keystore for Lollipop (5) or below then you need to have a way to recover from the keystore being lost / corrupt. This will be easier for some that others but being aware of the issue is a first step!

I hope that this sheds some light on this slightly confusing behaviour! As always, any thoughts / comments welcome.

#Further reading on the `Keystore` (& `KeyChain`)

- Check out the [Android docs for Keystore](http://developer.android.com/reference/java/security/KeyStore.html)
- Android developer blog post 2012 "[Unifying Key Store Access in ICS](http://android-developers.blogspot.co.uk/2012/03/unifying-key-store-access-in-ics.html)"
- Nikolay Elenkov blog 2011 [ICS Credential Storage Implementation](http://nelenkov.blogspot.jp/2011/11/ics-credential-storage-implementation.html)
- Nikolay Elenkov blog 2012 [Jelly Bean hardware-backed credential storage](http://nelenkov.blogspot.jp/2012/07/jelly-bean-hardware-backed-credential.html)
- Nikolay Elenkov's great book [Android Security Internals](http://www.nostarch.com/androidsecurity)
- [Android keystore key leakage between security domains](http://jbp.io/2014/04/07/android-keystore-leak/)
- [Android Keystore System](https://developer.android.com/training/articles/keystore.html) _EDIT: (29/05/15)_

