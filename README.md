# Genymotion development and release keys

Copyright (C) Genymobile

## Description

This repository stores Genymotion Android development and release keys, and corresponding .mk that we include internally in our `device.mk`. Third party developpers can use the release keys to sign their own applications in order to grant them system privileges on Genymotion. 

**Note:** only the Android 10 version of Genymotion Device Image (PaaS) starting from v13.1 rely on these keys. Our other Android versions rely on the `test-keys` from AOSP. This note will be updated when we migrate other versions to the keys stored in this repository.

`dev-keys` are used in `device.mk` to sign the system at build (properties are flagged with dev-keys).
We use `dev-keys/testkey`, but the build system flags them as `dev-keys`, as they are not the default AOSP keys.

See: https://android.googlesource.com/platform/build/+/refs/tags/android-10.0.0_r25/core/Makefile#306

`release-keys` are used by signature script to re-sign the build as a release one. Properties are flagged as `release-keys`, apk files are re-signed.

## Key generation

This part is for internal use. It describes our procedure to generate the development and release keys.

In the Genymotion Android root source: 

```
$ subject='/C=FR/ST=Seine/L=Paris/O=Genymobile/OU=Genymotion/CN=Genymobile/emailAddress=contact@genymotion.com'
```

Generate dev keys: 

```
$ for x in testkey platform shared media networkstack sdk_sandbox bluetooth; do \
   ./development/tools/make_key ./vendor/genymotion/security/dev-keys/$x "$subject"; \
done
```

Generate release keys: 

```
$ for x in releasekey platform shared media networkstack sdk_sandbox, bluetooth; do \
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
