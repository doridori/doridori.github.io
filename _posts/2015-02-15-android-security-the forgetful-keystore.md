---
layout: post
title: "Android Security: The Forgetful Keystore"
---

_You've just moved in to a new house and have been given the master key for the front door. You only have one of these so you know you need to keep it safe. Your really paranoid so you hire an armed guard, whose sole job is to protect this key, in fact, this is all he has been trained to do and has a catchy slogan of "need to protect a key, its what I was born to do!". You install an extra lock on your front door as you feel the bodyguard isn't enough, this is a rough area anyway and who's going to make sure no-ones about to break in and steal all your crap. You return to your key guard only to be informed he has thrown the key away. You shout and scream at him but he just blankly says "I don't have it anymore, I didn't think it was important". You can't contain your anger "What the hell, your a jerk! You had one thing to do and you failed, this causes me a lot of problems, why didn't you tell me you might do this?! What do I do now?!"_

Ok, stories aren't my strong point. The point is, this is how I feel when using the Android `Keystore` to protect my private encryption key, it drives me a bit nuts. I really appreciate the effort thats gone into it but its still frustrating when your on a deadline, documentation is lacking, and something isn't working as it should.

I have been working on some apps with a high security requirement, and one of the requirements is to take advantage of the best level of security offered by the platform for key management. For us this includes utilizing the system `Keystore` to store an asymmetric key pair. One can then use this for encryption/decryption directly or use it to encrypt an symmetric key (like AES) _(EDIT: AES keys can be stored in the Keystore directly since [M-6-23](https://github.com/doridori/Android-Security-Reference/blob/master/framework/keystore.md#version-changes))_ which can then be used directly to encrypt your plaintext (or fed into something like [java-aes-crypto](https://github.com/tozny/java-aes-crypto)), which is much faster than using the asymmetric directly.

Lots of people seem to use encryption in the wild but they often hard-code the key, storing it as plaintext or obfuscated, in Java or the NDK. None of which are recommended as key-lifting becomes a purely static analysis based exercise.

# Why is the `Keystore` good for storing keys?

Good question! The Keystore can hold its keys is two ways.

## Software

Key access is stored on the file system with access given via the OS, only allowing an app with the creating-app's UID to access. This is basically how Android file-permissions work in general. Stuff like this is the first to go on rooted devices as afaik `su` can adopt any UID.

## Hardware

I think the Nexus 4 was the first device to offer this, with APIs available from 4.3+.

> "Android also now supports hardware-backed storage for your KeyChain credentials, providing more security by making the keys unavailable for extraction. That is, once keys are in a hardware-backed key store (Secure Element, TPM, or TrustZone), they can be used for cryptographic operations but the private key material cannot be exported. Even the OS kernel cannot access this key material. While not all Android-powered devices support storage on hardware, you can check at runtime if hardware-backed storage is available by calling KeyChain.IsBoundKeyAlgorithm()" from [Release notes for 4.3](https://developer.android.com/about/versions/android-4.3.html#Security)

## API

_The first draft of this post (early 2015) used the (deprecated in `L-6-23`) `KeyPairGeneratorSpec` class for key generation, and the original behaviour matrices were generated using this API. I have updated some areas of the post to include information about the post-M `KeyGenParameterSpec` API._

The api method I used to generate the key-pair was `KeyPairGeneratorSpec` which was added in `J-4.3-18`. There were ways to use this stuff before then (see Nikolays blog links at the end of this post) but I'm going to ignore that for now.

_Some additional notes of mine on the `KeyStore` APIs and OS changes can be found [here](https://github.com/doridori/Android-Security-Reference/blob/master/framework/keystore.md)_

# The Problem

"This sounds great" I hear you cry. Well yes it is, until the Keystore-stored-key becomes inaccessible, leaving you with a useless steaming pile-o-bits™.

When the KeyStore api was introduced, this could happen when the device security settings (device-lock) are changed by the user, namely switching between `None`, `Swipe`, `Pattern`, `Pin` and `Password`. Subsequent OS versions will behave differently however. Outlining these differences is the purpose of this post.

Reading the [related android bug-tracker list](https://code.google.com/p/android/issues/list?can=1&q=KeyPairGeneratorSpec&colspec=ID+Type+Status+Owner+Summary+Stars&cells=tiles) some cry 'its a bug' and others 'its a feature'. If its a feature documentation is much needed, if a bug then fixes are welcome. From this point the most I can do it outline what I have seen so as to enable others to work around it / be aware.

If you do get into the position of having a wiped `KeyStore` I have seen this manifest in the below ways (in my initial testing):

## `KeyPairGeneratorSpec`

_Deprecated in M, this method was the focus of the original blog post_

- Most of the time ~98% in my original testing the key data _AND_ alias is wiped - giving a `InvalidKeyException` at point of reading the `KeyPair` from the `KeyStore`.
- A small amount of the time ~2% the key data is lost but the alias is _not_ giving `IllegalArgException`

See below for the `InvalidKeyException` source.

<div data-gist-id="cfe0fca74a9c05fc6a57" data-gist-file="InvalidKeyException">InvalidKeyException Example</div>

From the above you can see it's not just a silent fail yielding an empty KeyStore but an undocumented-exception-throwing fail which requires you to reset the alias ([Keystore.deleteEntry()](https://developer.android.com/reference/java/security/KeyStore.html#deleteEntry(java.lang.String))).

## Device locks & `KeyPairGeneratorSpec.Builder.setEncryptionRequired()`

Its worth noting that when creating a key with [`KeyPairGeneratorSpec.Builder.setEncryptionRequired()`](http://developer.android.com/reference/android/security/KeyPairGeneratorSpec.Builder.html#setEncryptionRequired()) its required that the device have a device-lock set (as this is used as part of the keying material _when the KeyStore is software backed_). If you do `setEncryptionRequired()` and one isn't, you get a friendly

- Pre M-6-23 `java.lang.IllegalStateException: Android keystore must be in initialized and unlocked state if encryption is required`
- Post M-6-23 `java.lang.IllegalStateException: Encryption at rest using secure lock screen credential requested for key pair, but the user has not yet entered the credential`

so you need to manually ensure that something is set. Check out [my SO answer](http://stackoverflow.com/a/27801128/236743) for an approach to use here :). The great [Android Security Cookbook](https://www.packtpub.com/application-development/android-security-cookbook) has a recipe for using the Device Admin Policy.

## `KeyGenParameterSpec`

_Added in M_

- Using the newer `KeyGenParameterSpec` api will now throw a dedicated exception for this, `KeyPermanentlyInvalidatedException` which is a subclass of `InvalidKeyException`. This seems to be thrown at point of use i.e. `Cipher.encrypt`, `Cipher.decrypt` or `Cipher.unwrap` (not with `Cipher.wrap` - assume this is as the public key is not encrypted in the first place) rather than when reading the Key handle.

# `KeyPairGeneratorSpec` Behaviour Matrix

_These were created using the now deprecated `KeyPairGeneratorSpec` API_

As mentioned above different OSs and different transitions between device-lock states can wipe the keystore.

I have written a small test app and run this on a few devices and emulator to see what results I get. This test app will be using the [`SecretKeyWrapper`](https://android.googlesource.com/platform/development/+/master/samples/Vault/src/com/example/android/vault/SecretKeyWrapper.java) from the Android `Vault` package examples. I have added two methods to this to enable testing. See the full (dirty) source on [github](https://github.com/doridori/AndroidKeystoreWipeTest/tree/master).

See the tests below. T = Pass, F = Fail. Some cells are blank as I didn't test that combo - but there is enough there too see the pattern in each case :)

## O preview 1 - Nexus 5X

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

If device lock is **NONE** you cannot create a keystore entry with `setEncryptionRequired()`. If you try you'll see an  `IllegalStateException`.


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

More N/As on this one as `.setEncryptionRequired()` will throw if you try to create a keypair with a **NONE** state.

## M-6.0-23 | Nexus 5

### Changes

M introduced some changes here explicitly:

> "Keys which do not require encryption at rest will no longer be deleted when secure lock screen is disabled or reset (for example, by the user or a Device Administrator). Keys which require encryption at rest will be deleted during these events.

From [http://developer.android.com/about/versions/marshmallow/android-6.0-changes.html#behavior-keystore](http://developer.android.com/about/versions/marshmallow/android-6.0-changes.html#behavior-keystore)

Also, the screen-lock settings screen will now warn the user if they try to revert to **NONE**. This looks like:

![Keyguard removed](https://raw.githubusercontent.com/doridori/doridori.github.io/master/images/blog/O_keyguard_removed.png)

### Tests

| to ↓        from > | NONE | PIN | PASS | PATTERN |
|--------------------|------|-----|------|---------|
| NONE               |  N/A |     |  T   |   T     |
| PIN                |  T   |  T  |  T   |         |
| PASS               |      |  T  |      |   T     |
| PATTERN            |  T   |  T  |      |         |

**`.setEncryptionRequired()`**

It seems there has actually been a **regression** from the improvements made in L regarding not allowing the user to corrupt the `KeyStore` contents by not allowing lock screen removal when using `.setEncryptionRequired()`. ε(´סּ︵סּ\`)

| to ↓        from > | NONE | PIN | PASS | PATTERN |
|--------------------|------|-----|------|---------|
| NONE               |  N/A |**F**|**F** |         |
| PIN                |  N/A | T   |  T   |         |
| PASS               |  N/A | T   |      |   T     |
| PATTERN            |  N/A | T   |  T   |         |

More N/As on this one as `.setEncryptionRequired()` will throw if you try to create a keypair with a NONE state.

## L-5.0.1-21 | Nexus 4

### Changes

Once a keypair has been generated the system will not let you revert to NONE (regardless of if the `KeyStore` is encrypted or not) unless the app has been deleted / the pair removed hence the N/A. This looks like:

[![Disabled NONE options](https://raw.githubusercontent.com/doridori/doridori.github.io/master/images/blog/keystore_lock_disabled.jpg)

### Tests

| to ↓        from > | NONE | PIN | PASS | PATTERN |
|--------------------|------|-----|------|---------|
| NONE               | N/A  | N/A | N/A  | N/A     |
| PIN                | **F**| T   |      |         |
| PASS               | **F**|     | T    | T       |
| PATTERN            | T    | T   |      |         |





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

# Post `M-6-23` `KeyGenParameterSpec` Fingerprint APIs

Since Android M-6-23 keys can be stored which require authentication via fingerprint before they can be used. Note the below in regards to lifetime of such keys.

> The key will become irreversibly invalidated once the secure lock screen is disabled (reconfigured to None, Swipe or other mode which does not authenticate the user) or when the secure lock screen is forcibly reset (e.g., by a Device Administrator). Additionally, if the key requires that user authentication takes place for every use of the key, it is also irreversibly invalidated once a new fingerprint is enrolled or once\ no more fingerprints are enrolled. Attempts to initialize cryptographic operations using such keys will throw KeyPermanentlyInvalidatedException.
This authorization applies only to secret key and private key operations. Public key operations are not restricted.

Taken from [KeyGenParameterSpec.Builder.setUserAuthenticationRequired(...)](http://developer.android.com/reference/android/security/keystore/KeyGenParameterSpec.Builder.html#setUserAuthenticationRequired(boolean)).

This means that losing keys will always be an inevitability if using Fingerprint on 6.

On N-7-24 however, there is now a `setInvalidatedByBiometricEnrollment(false)` method, which will persist the keys when a new finger is enrolled.

Its worth noting that the new KeyGenParameterSpec.Builder.setUserAuthenticationRequired(true) method does not use encryption with a password-derived key, performed in keystore. It encrypts blobs with a TEE-based key. Password checking is done by a TEE-based component called "Gatekeeper", which implements brute force mitigation by forcing increasing delays after repeated failures. After a success, Gatekeeper generates an "authentication token", which is a data structure MACed with a key shared between Keymaster (the TEE-based key management component) and Gatekeeper. That token provides information about which user authenticated, how they authenticated (e.g. password or fingerprint; there's a TEE-based fingerprint component that can also produce these tokens) and when they authenticated. Keymaster uses that information to determine whether access to a particular key should be granted (see setUserAuthenticationValidityDurationSeconds, setInvalidatedByBiometricEnrollment and setUserAuthenticationVAlidWhileOnBody). Keymaster will only decrypt and use a key if it has received an appropriate auth token. All of this happens in the TEE (well, not on-body checking), so even an attacker that has completely compromised the Android kernel is unable to manipulate it.

See [source.android.com/security/authentication/](https://source.android.com/security/authentication/) for more info on the above.

The new API is similar in that removing your password will invalidate all authentication-bound keys. This is because if the password is removed, anyone can pick up the device and use it, and the security guarantee provided to app developers is that they can have confidence that if they create an auth-bound key, it can only be used if the user has authenticated in the way specified at key creation time.

# Conclusion

With the above we can see that:

- Using the old pre-M `KeyPairGeneratorSpec` api and **not** setting `.setEncryptionRequired()` from `L-6-23`+ seems to be safe from key-loss for the above scenarios.
  - Quoting Shawn from the comments "Without `setEncryptionRequired()` your keys should never disappear (except on app uninstall or factory reset)"
  - If you **do** use `setEncryptionRequired(true)`, since M it will no longer be the case that changing the screen lock type will cause keys to be deleted. _Removing_ the screen lock will, but that's by design; if there's no password to use to encrypt they keys, the keys cannot be encrypted.
  - However, `setEncryptionRequired()` is now deprecated, and developers shouldn't use it.


- Using the new post-M `KeyGenParameterSpec` api and not setting `setUserAuthenticationRequired` should be safe from key loss in all scenarios
  - Setting `setUserAuthenticationRequired(true)` will wipe the keys if the user removes the screen-lock entirely, but they will be warned about this, and a specific exception will be thrown when key usage is attempted.
  - Setting `setUserAuthenticationRequired(true)` will cause the keys to be wiped when a new finger is enrolled, _unless_ `setInvalidatedByBiometricEnrollment(false)` is also set.

I hope that this sheds some light on this slightly confusing behaviour! As always, any thoughts / comments welcome.

# What key management choices you have

1. Use the KeyStore in `L-6-23`+ in one the above configurations which seem to be "safe" from key loss and have a relative degree of confidence your keys are not going anywhere. I personally would still handle the key-loss scenario by triggering some re-onboarding UX, but maybe thats me being paranoid.

2. Use the KeyStore on lower than `L-6-23`, or in a "lossy" configuration, and handle the re-onbaording UX, and expect this will not be some rare edge case path.

3. Use some [**PBKDF**](https://en.wikipedia.org/wiki/PBKDF2). Look at another way of storing key data, for example generating one from a users input. This will not suit all applications of course. Target any platform version. Check out [java-aes-crypto](https://github.com/tozny/java-aes-crypto) for a really sweet and easy to use key generator which can be used with arbitrary password input. Its worth nothing that this may be really easy to brute-force depending on your required password length.

4. Re-designing your solution so client side keys are not required (easier said than done in many cases!)

# Further reading on the `Keystore` (& `KeyChain`)

- Check out [my overview](https://github.com/doridori/Android-Security-Reference/blob/master/framework/keystore.md) to the `KeyStore`
- Submitted [bug](https://code.google.com/p/android/issues/detail?id=210402&) in tracker
- Check out the [Android docs for Keystore](http://developer.android.com/reference/java/security/KeyStore.html) and the [training article](http://developer.android.com/training/articles/keystore.html)
- Android developer blog post 2012 "[Unifying Key Store Access in ICS](http://android-developers.blogspot.co.uk/2012/03/unifying-key-store-access-in-ics.html)"
- Nikolay Elenkov blog 2011 [ICS Credential Storage Implementation](http://nelenkov.blogspot.jp/2011/11/ics-credential-storage-implementation.html)
- Nikolay Elenkov blog 2012 [Jelly Bean hardware-backed credential storage](http://nelenkov.blogspot.jp/2012/07/jelly-bean-hardware-backed-credential.html)
- Nikolay Elenkov's great book [Android Security Internals](http://www.nostarch.com/androidsecurity)
- [Android keystore key leakage between security domains](http://jbp.io/2014/04/07/android-keystore-leak/)
- [Android Keystore System](https://developer.android.com/training/articles/keystore.html) _EDIT: (29/05/15)_
- [Android `Vault` Example](https://github.com/android/platform_development/tree/master/samples/Vault)

