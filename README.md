# Genymotion development and release keys

Copyright (C) Genymobile

## Description

This repository stores Genymotion Android development and release keys, and corresponding .mk that are included from device.mk.

`dev-keys` are used in `device.mk` to sign the system at build (properties are flagged with dev-keys).
We use `dev-keys/testkey`, but the build system flags them as `dev-keys`, as they are not the default AOSP keys.

See: https://android.googlesource.com/platform/build/+/refs/tags/android-10.0.0_r25/core/Makefile#306

`release-keys` are used by signature script to re-sign the build as a release one. Properties are flagged as `release-keys`, apk files are re-signed.

## Key generation

In the Genymotion Android root source : 

```
$ subject='/C=FR/ST=Seine/L=Paris/O=Genymobile/OU=Genymotion/CN=Genymobile/emailAddress=contact@genymotion.com'
```

Generate dev keys : 

```
$ for x in testkey platform shared media networkstack; do \
   ./development/tools/make_key ./vendor/genymotion/security/dev-keys/$x "$subject"; \
done
```

Generate release keys : 

```
$ for x in releasekey platform shared media networkstack; do \
   ./development/tools/make_key ./vendor/genymotion/security/release-keys/$x "$subject"; \
done
```

## Genymotion Keystore

### Generate the keystore

To sign an application APK with the release keys, gradle/Android Studio needs a keystore.
For debug builds, a default `$HOME/.android/debug.keystore` keystore is used.
For release builds, we can specify another keystore generated from our release keys.

How to generate the release keystore in the directory `release-keys/keystore`: 

```
$ openssl pkcs8 -inform DER -nocrypt -in ../platform.pk8 | \
    openssl pkcs12 -export -in ../platform.x509.pem -inkey /dev/stdin \
    -name platform -password pass:genymotion \
    -out platform_release.pem
$ keytool -importkeystore \
    -destkeystore platform_release.keystore -deststorepass genymotion \
    -srckeystore platform_release.pem -srcstoretype PKCS12 -srcstorepass 'genymotion'
$ rm platform_release.pem
```

### Use the keystore

The generated keystore (platform_release.keystore) can now be used to sign Android applications.

* Copy `platform_release.keystore` into your application sources: `<project>/app/platform_release.keystore`.
* Update your `build.gradle` file to use this keystore: 

```
android {
    ...
    signingConfigs {
        release {
            storeFile file("platform_release.keystore")
            storePassword 'genymotion'
            keyAlias 'platform'
            keyPassword 'genymotion'
        }
        debug {
            storeFile file("platform_test.keystore")
            storePassword 'genymotion'
            keyAlias 'platform'
            keyPassword 'genymotion'
        }
    }
    
    buildTypes {
        release {
            ...
            signingConfig signingConfigs.release
        }
        debug {
            ...
            signingConfig signingConfigs.debug
        }
    }
}
```
