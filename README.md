# Genymotion dev and release keys.

Copyright (C) Genymobile

## Description

Repository to store dev and release keys, and corresmonding .mk that are included from device.mk.

dev-keys/ is used in device/genymotion/vbox86/device.mk to sign the system at build (properties are flagged with dev-keys).
We use dev-keys/testkey, but the build system flag for dev-keys, as they are not the AOSP default keys.
See : https://android.googlesource.com/platform/build/+/refs/tags/android-10.0.0_r25/core/Makefile#306

release-keys/ is used with signature script to resign as release (properties are flagged with release-keys, apk are resigned).

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

## Genymotion Keystore :

### Generate the keystore

To sign an application APK with the release keys, gradle/Android Studio need a keystore.
For debug builds, a default keystore $HOME/.android/debug.keystore is useD.
For release builds, we can specify another keystore generated from our release keys.

How to generate the release keystore in the directory release-keys/keystore : 

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

The generated keystore (platform_release.keystore) can now be used to build and sign Android application.

Example for genymotion-testing/GenymotionTestReleaseKey : 
* Copy platform_release.keystore in genymotion-testing/GenymotionTestReleaseKey/app/platform_release.keystore
genymotion-testing/GenymotionTestReleaseKey
* Update build.gradle file to use this keystore : 

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
