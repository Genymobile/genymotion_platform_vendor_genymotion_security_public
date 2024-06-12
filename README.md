# Genymotion release keys

Copyright (C) Genymobile

## Description

This repository stores Genymotion Android release keys, and corresponding `.mk` that we include internally in our `device.mk`. Third party developpers can use the release keys to sign their own applications in order to grant them system privileges on Genymotion.

`release-keys` are used by signature script to re-sign the build as a release one. Properties are flagged as `release-keys` and apk files are re-signed.

**Notes:** Please refer to the documentation of the Genymotion product you are using for more details about versions which support this feature:
* **Genymotion Device Image**: [Root Access documentation](https://docs.genymotion.com/paas/10_Using_root_access/)
* **Genymotion Desktop**: [Root Access documentation](https://docs.genymotion.com/desktop/Using_root_access/)
* **Genymotion SaaS**: [Root Access documentation](https://docs.genymotion.com/saas/Using_root_access/)

Other versions will still rely on the default `test-keys` from AOSP, see [the AOSP documentation](https://source.android.com/docs/core/ota/sign_builds) for more information.

## Key generation

This part is for internal use. It describes our procedure to generate the release keys.

In the Genymotion Android root source: 

```
$ subject='/C=FR/ST=Seine/L=Paris/O=Genymobile/OU=Genymotion/CN=Genymobile/emailAddress=contact@genymotion.com'
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
