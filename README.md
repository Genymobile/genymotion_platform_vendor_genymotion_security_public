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
