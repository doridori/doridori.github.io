---
layout: post
title: "Android Security: The Forgetful Keystore"
---

You've just moved in to a new house and have been given the master key for the front door. You only have one of these so you know you need to keep it safe. Your really paranoid so you hire an armed guard, whose sole job is to protect this key, in fact, this is all he has been trained to do and has a catchy slogan of "need to protect a key, its what I was born to do!". You install an extra lock on your front door as you feel the bodyguard isn't enough, this is a rough area anyway and who's going to make sure no-ones about to break in and steal all your crap. You return to your key guard only to be informed he has thrown the key away. You shout and scream at him but he just blankly says "I don't have it anymore, I didn't think it was important". You can't contain your anger "What the hell, your a jerk! You had one thing to do and you failed, this causes me a lot of problems, why didn't you tell me you might do this?! What do I do now?!"

Ok, stories arnt my strong point. The point is, this is how I feel when using the Android `Keystore` to protect my private encryption key, it drives me a bit nuts. I really appreciate the effort thats gone into it but its still frustrating when your on a deadline, documentation is lacking, and something isn't working as it should

I have been working on some apps with a high security requirement, and one of the requirements is to take advantage of the best level of security offered by the platform. For our case this includes utilizing the system `Keystore` to store an asymmetric key pair. One can then use this for encryption/decryption directly or use it to encrypt an symmetric key (like AES) which can then be used directly to encrypt your plaintext (or fed into something like [java-aes-crypto](https://github.com/tozny/java-aes-crypto)), which is much faster than using the asymmetric directly.

Lots of people seem to use encryption in the wild but they hard code the key or store is as plaintext or obfuscated. None of which are recommended.

# Why is the `Keystore` good for storing keys?

Good question! The Keystore can hold its keys is two ways.

## Software

Key access is stored on the file system with access given via the OS, only allowing an app with the creating-app's UID to access. This is basically how Android file-permissions work in general. Stuff like this is the first to go on rooted devices as afaik `su` can adopt any UID.

## Hardware

I think the Nexus 4 was the first device to offer this, with APIs available from 4.3+.

> "Android also now supports hardware-backed storage for your KeyChain credentials, providing more security by making the keys unavailable for extraction. That is, once keys are in a hardware-backed key store (Secure Element, TPM, or TrustZone), they can be used for cryptographic operations but the private key material cannot be exported. Even the OS kernel cannot access this key material. While not all Android-powered devices support storage on hardware, you can check at runtime if hardware-backed storage is available by calling KeyChain.IsBoundKeyAlgorithm()" from [Release notes for 4.3](https://developer.android.com/about/versions/android-4.3.html#Security)

## API

_The first draft of this post (early 2015) used the (deprecated in `L-6-23`) `KeyPairGeneratorSpec` class for key generation, and the original behaviour matrices were generated using this API. I have updated some areas of the post to include information about the post-M `KeyGenParameterSpec` API._

The api method I used to generate the key-pair was `KeyPairGeneratorSpec` which was added in 4.3 (18). There were ways to use this stuff before then (see Nikolays blog links below) but I'm going to ignore that for now.

_Some additional notes of mine on the `KeyStore` APIs and OS changes can be found [here](https://github.com/doridori/Android-Security-Reference/blob/master/framework/keystore.md)_

# The Problem

"This sounds great" I hear you cry. Well yes it is, until the Keystore-stored-key becomes inaccessible, leaving you with a useless steaming pile-o-bits™.

Previously this could happen when the device security settings (device-lock) are changed by the user, namely switching between `None`, `Swipe`, `Pattern`, `Pin` and `Password` but also OS versions will behave differently.

Reading the [related android bug-tracker list](https://code.google.com/p/android/issues/list?can=1&q=KeyPairGeneratorSpec&colspec=ID+Type+Status+Owner+Summary+Stars&cells=tiles) some cry 'its a bug' and others 'its a feature'. If its a feature documentation is much needed, if a bug then fixes are welcome. From this point the most I can do it outline what I have seen so as to enable others to work around it / be aware.

If you do get into the position of having a wiped `KeyStore` I have seen this manifest in the below ways (in my initial testing):

## `KeyPairGeneratorSpec`

_Deprecated in M_

- Most of the time ~98% in my original testing the key data _AND_ alias is wiped - giving a `InvalidKeyException` at point of reading the `KeyPair` from the `KeyStore`.
- A small amount of the time ~2% the key data is lost but the alias is _not_ giving `IllegalArgException`

See below for the `InvalidKeyException` source.

<div data-gist-id="cfe0fca74a9c05fc6a57" data-gist-file="InvalidKeyException">InvalidKeyException Example</div>

From the above you can see it's not just a silent fail yielding an empty keystore but an undocumented-exception-throwing fail which requires you to reset the alias ([Keystore.deleteEntry()](https://developer.android.com/reference/java/security/KeyStore.html#deleteEntry(java.lang.String))).

## `KeyGenParameterSpec`

_Added in M_

- Using the newer `KeyGenParameterSpec` api will now throw a dedicated exception for this, `KeyPermanentlyInvalidatedException` which is a subclass of `InvalidKeyException`. This seems to be thrown at point of use i.e. `Cipher.encrypt`, `Cipher.decrypt` or `Cipher.unwrap` (not with `Cipher.wrap` - assume this is as the public key is not encrypted in the first place) rather than when reading the Key handle.

## Device locks & `.setEncryptionRequired()`

Its worth noting that when creating a key with [`KeyPairGeneratorSpec.Builder.setEncryptionRequired()`](http://developer.android.com/reference/android/security/KeyPairGeneratorSpec.Builder.html#setEncryptionRequired()) its required that the device have a device-lock set (as this is used as part of the keying material _when the KeyStore is software backed_). If you do `setEncryptionRequired()` and one isn't, you get a friendly

- Pre M-6-23 `java.lang.IllegalStateException: Android keystore must be in initialized and unlocked state if encryption is required`
- Post M-6-23 `java.lang.IllegalStateException: Encryption at rest using secure lock screen credential requested for key pair, but the user has not yet entered the credential`

so you need to manually ensure that something is set. Check out [my SO answer](http://stackoverflow.com/a/27801128/236743) for an approach to use here :). The great [Android Security Cookbook](https://www.packtpub.com/application-development/android-security-cookbook) has a recipe for using the Device Admin Policy.

## `allowBackups=true`

I observed an interesting issue, which I need to revisit and verify.

I found that having `allowBackups=true` (which it is by default) was not deleting `KeyStore` backed keys on app uninstall (or was restoring the aliases on re-install). This led to me re-installing an application with had fingerprint auth backed keys, which I could no longer access, i.e. the aliases were present but keys unusable. Need to research this one a bit more. I recommend setting  `allowBackups=false` anyhow if your working with this stuff.

# Behaviour Matrix

_These were created using the now deprecated `KeyPairGeneratorSpec` API_

As mentioned above different OSs and different transitions between device-lock states can wipe the keystore.

I have written a small test app and run this on a few devices and emulator to see what results I get. This test app will be using the [`SecretKeyWrapper`](https://android.googlesource.com/platform/development/+/master/samples/Vault/src/com/example/android/vault/SecretKeyWrapper.java) from the Android `Vault` package examples. I have added two methods to this to enable testing. See the full (dirty) source on [github](https://github.com/doridori/AndroidKeystoreWipeTest/tree/master).

See the tests below. T = Pass, F = Fail. Some cells are blank as I didn't test that combo - but there is enough there too see the pattern in each case :)

# O preview 1 - Nexus 5X

| to ↓        from > | NONE | PIN | PASS | PATTERN |
|--------------------|------|-----|------|---------|
| NONE               |  T   |  T  |  T   |  T      |
| PIN                |  T   |  T  |  T   |  T      |
| PASS               |  T   |  T  |  T   |  T      |
| PATTERN            |  T   |  T  |  T   |  T      |

**`.setEncryptionRequired()`**

| to ↓        from > | NONE | PIN | PASS | PATTERN |
|--------------------|------|-----|------|---------|
| NONE               | N/A  |**F**|**F** | **F**   |
| PIN                | N/A  |  T  |  T   |   T     |
| PASS               | N/A  |  T  |  T   |   T     |
| PATTERN            | N/A  |  T  |  T   |   T     |

If device lock is NONE you cannot create a keystore entry with `setEncryptionRequired()`. If you try you'll see an  IllegalStateException.


## Android 7.1 - API 25 - Nexus 5X

_//todo no encryption required_

**`setEncryptionRequired()`**

| to ↓        from > | NONE | PIN | PASS | PATTERN |
|--------------------|------|-----|------|---------|
| NONE               | N/A  |**F**|**F** | **F**   |
| PIN                | N/A  |  T  |  T   |   T     |
| PASS               | N/A  |  T  |  T   |   T     |
| PATTERN            | N/A  |  T  |  T   |   T     |


## Android 7.0 - API 25 - Nexus 5X

| to ↓        from > | NONE | PIN | PASS | PATTERN |
|--------------------|------|-----|------|---------|
| NONE               |  T   |  T  |  T   |   T     |
| PIN                |  T   |  T  |  T   |   T     |
| PASS               |  T   |  T  |  T   |   T     |
| PATTERN            |  T   |  T  |  T   |   T     |

**`.setEncryptionRequired()`**

| to ↓        from > | NONE | PIN | PASS | PATTERN |
|--------------------|------|-----|------|---------|
| NONE               | N/A  |**F**|**F** | **F**   |
| PIN                | N/A  |  T  |  T   |   T     |
| PASS               | N/A  |  T  |  T   |   T     |
| PATTERN            | N/A  |  T  |  T   |   T     |



## N Preview 3 | qemu-system-x86_64 (Emulator 2.0)

| to ↓        from > | NONE | PIN | PASS | PATTERN |
|--------------------|------|-----|------|---------|
| NONE               |  N/A |  T  |  T   |         |
| PIN                |  T   |     |  T   |   T     |
| PASS               |  T   |  T  |      |         |
| PATTERN            |  T   |     |      |         |

**`.setEncryptionRequired()`**

| to ↓        from > | NONE | PIN | PASS | PATTERN |
|--------------------|------|-----|------|---------|
| NONE               |  N/A |**F**|      |  **F**  |
| PIN                |  N/A |  T  |      |         |
| PASS               |  N/A |  T  |      |         |
| PATTERN            |  N/A |     |  T   |         |

More N/As on this one as `.setEncryptionRequired()` will throw if you try to create a keypair with a NONE state.

## M-6.0-23 | Nexus 5

M introduced some changes here explicitly:

> "Keys which do not require encryption at rest will no longer be deleted when secure lock screen is disabled or reset (for example, by the user or a Device Administrator). Keys which require encryption at rest will be deleted during these events.

From [http://developer.android.com/about/versions/marshmallow/android-6.0-changes.html#behavior-keystore](http://developer.android.com/about/versions/marshmallow/android-6.0-changes.html#behavior-keystore)

| to ↓        from > | NONE | PIN | PASS | PATTERN |
|--------------------|------|-----|------|---------|
| NONE               |  N/A |     |  T   |   T     |
| PIN                |  T   |  T  |  T   |         |
| PASS               |      |  T  |      |   T     |
| PATTERN            |  T   |  T  |      |         |

**`.setEncryptionRequired()`**

<b>It seems there has actually been a regression from the improvements made in L regarding not allowing the user to corrupt the `KeyStore` contents by not allowing lock screen removal when using `.setEncryptionRequired()`.</b> ε(´סּ︵סּ`)з

| to ↓        from > | NONE | PIN | PASS | PATTERN |
|--------------------|------|-----|------|---------|
| NONE               |  N/A |**F**|**F** |         |
| PIN                |  N/A | T   |  T   |         |
| PASS               |  N/A | T   |      |   T     |
| PATTERN            |  N/A | T   |  T   |         |

More N/As on this one as `.setEncryptionRequired()` will throw if you try to create a keypair with a NONE state.

## L-5.0.1-21 | Nexus 4

| to ↓        from > | NONE | PIN | PASS | PATTERN |
|--------------------|------|-----|------|---------|
| NONE               | N/A  | N/A | N/A  | N/A     |
| PIN                | **F**| T   |      |         |
| PASS               | **F**|     | T    | T       |
| PATTERN            | T    | T   |      |         |

Once a keypair has been generated the system will not let you revert to NONE (regardless of if the `KeyStore` is encrypted or not) unless the app has been deleted / the pair removed hence the N/A. This looks like

[![Disabled NONE options](https://raw.githubusercontent.com/doridori/doridori.github.io/master/images/blog/keystore_lock_disabled.jpg)]

**`.setEncryptionRequired()`**

| to ↓        from > | NONE | PIN | PASS | PATTERN |
|--------------------|------|-----|------|---------|
| NONE               | N/A  | N/A | N/A  | N/A     |
| PIN                | N/A  | T   | T    | T       |
| PASS               | N/A  | T   | T    | T       |
| PATTERN            | N/A  | T   | T    | T       |

More N/As on this one as `.setEncryptionRequired()` will throw if you try to create a keypair with a NONE state.

The user will not be able to revert to NONE until selecting 'Clear Credentials' from the security menu. On my N5 uninstalling the test app did not allow setting to NONE until a manual clear.

**EDIT** It seems that there are still issues on 5 when using non-primary user accounts, which seems to behave like older OS versions and will still wipe the keystore :( See [this bug](https://code.google.com/p/android/issues/detail?id=61989#c21)

## K-4.4.4-19 | Nexus 7

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

## JB-4.3-18 | Genymotion Emulator

| to ↓        from > | SLIDE | PIN | PASS | PATTERN |
|--------------------|-------|-----|------|---------|
| SLIDE              |       | F   |      |         |
| PIN                | F     | F   | F    | F       |
| PASS               | F     | F   | F    |         |
| PATTERN            |       |     | T    | T       |

Sad face.

# Conclusion

Knowing the above I see a few potential solutions

1. <del>**Target 5+** If your requirement is to have strong encryption using the keystore and losing the encrypted data is a no-no - targeting L+ (5+) seems like the only choice, using `.setEncryptionRequired()`. However be warned that this is no guarantee that it will always work as this seems to still be broken for non-primary user accounts.</del> This is no longer true due to the regression mentioned above in M-6-23.

2. **`.setEncryptionRequired()` `KeyStore` and OK to lose Keys**. If your requirement is software keys encrypted by the keyguard and losing the encrypted data is ok - then you can detect the above exceptions mentioned and have some form of re-initialization for those cases. Target 4.3+. Remember to check the device has the required lock-screen security settings (as mentioned above) before generating the keyPair. Checking for the exceptions at point of key unwrap and notifying the user whilst clearing any key aliases is a simple enough approach for most. Keep in mind if supporting the fingerprint sensor to auth access to your keys you will lose them when a new finger is enrolled anyway (_EDIT: This is now optional with the M+ key generation API via an N+ method  [setInvalidatedByBiometricEnrollment](https://developer.android.com/reference/android/security/keystore/KeyGenParameterSpec.Builder.html#setInvalidatedByBiometricEnrollment(boolean))_). Put some thought into the UX for key loss.

3. **Use some PBKDF**. Look at another way of storing key data, for example generating one from a users input. This will not suit all applications of course. Target any platform version. Check out [java-aes-crypto](https://github.com/tozny/java-aes-crypto) for a really sweet and easy to use key generator which can be used with arbitrary password input. Its worth nothing that this may be really easy to brute-force depending on your required password length.

4. **NON `.setEncryptionRequired()` `KeyStore`**. Don't require the software `KeyStore` to be encrypted under the keyguard and target 6.0+ and don't support user authentication for `KeyStore` access (via respective API level apis). Cross your fingers and hope the behaviour does not change again. Hope (or check for) hardware storage for some extra confidence. Understand the user may have a software store only and may have a slightly higher chance of being exploited with keys being obtained by some nefarious character.

I hope that this sheds some light on this slightly confusing behaviour! As always, any thoughts / comments welcome.

# Further reading on the `Keystore` (& `KeyChain`)

- Check out [my overview](https://github.com/doridori/Android-Security-Reference/blob/master/api/keystore.md) to the `KeyStore`
- Submitted [bug](https://code.google.com/p/android/issues/detail?id=210402&) in tracker
- Check out the [Android docs for Keystore](http://developer.android.com/reference/java/security/KeyStore.html) and the [training article](http://developer.android.com/training/articles/keystore.html)
- Android developer blog post 2012 "[Unifying Key Store Access in ICS](http://android-developers.blogspot.co.uk/2012/03/unifying-key-store-access-in-ics.html)"
- Nikolay Elenkov blog 2011 [ICS Credential Storage Implementation](http://nelenkov.blogspot.jp/2011/11/ics-credential-storage-implementation.html)
- Nikolay Elenkov blog 2012 [Jelly Bean hardware-backed credential storage](http://nelenkov.blogspot.jp/2012/07/jelly-bean-hardware-backed-credential.html)
- Nikolay Elenkov's great book [Android Security Internals](http://www.nostarch.com/androidsecurity)
- [Android keystore key leakage between security domains](http://jbp.io/2014/04/07/android-keystore-leak/)
- [Android Keystore System](https://developer.android.com/training/articles/keystore.html) _EDIT: (29/05/15)_
- [Android `Vault` Example](https://github.com/android/platform_development/tree/master/samples/Vault)

# Appendix: Fingerprint api

Since Android M-6-23 keys can be stored which require authentication via fingerprint before they can be used. Note the below in regards to lifetime of such keys.

> The key will become irreversibly invalidated once the secure lock screen is disabled (reconfigured to None, Swipe or other mode which does not authenticate the user) or when the secure lock screen is forcibly reset (e.g., by a Device Administrator). Additionally, if the key requires that user authentication takes place for every use of the key, it is also irreversibly invalidated once a new fingerprint is enrolled or once\ no more fingerprints are enrolled. Attempts to initialize cryptographic operations using such keys will throw KeyPermanentlyInvalidatedException.
This authorization applies only to secret key and private key operations. Public key operations are not restricted.

Taken from [KeyGenParameterSpec.Builder.setUserAuthenticationRequired(...)](http://developer.android.com/reference/android/security/keystore/KeyGenParameterSpec.Builder.html#setUserAuthenticationRequired(boolean)).

This means that losing keys will always be an inevitability if using Fingerprint on 6.
