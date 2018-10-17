---
layout: post
title:  "Building QEMU Instances for Scaled Dynamic Android App Analysis"
date:   2017-11-03 10:58:00 +0800
categories: Android
tags: Android QEMU
---
Dynamic analysis on Android apps needs emulator instances or real devices on which it runs real world apps. There are existing solutions including:

1. **[Official Android Emulator from Google][avd]**

    Using Google-API images from Android SDK, one gets most complete features among emulator methods. However, official images don't provide x86-ARM translation, that is, one can only run apps with ARM native code in ARM images based on QEMU ARM emulator, which is quite slow.
2. **[Android x86 Project][andx86]**

    This is a project porting AOSP to x86. It provides solutions that equip x86 emulators with x86-ARM translation, the `houdini` technology from Intel. However it lacks Google service and many device features like GPS and camera by default.

3. **[Genymotion Emulator with ARM translator][geny]**

    Genymotion provides highly customized emulator with third-party x86-ARM translation support from [23pin (Chinese page)][23pin]. Unfortunately Genymotion depends on VirtualBox which is not quite light-weighted and flexible, and it's a commercial software, with limited features in free version.
4. Real Devices

    It's quite straight forward to plug piles of Android devices onto a host machine and run analysis on them, gaining full device features. Disadvantages include cost, scalability and limited ROM code customization.

In this article, I'll introduce a method to create QEMU-based Android emulator instances for scalable dynamic analysis of real-world apps. The emulator has the following advantages:

* Light-weighted, easy to start and stop
* Supports x86-ARM translation
* Fully customizable code
* Supports Google API support
* Supports limited location service

Intuitively these features guarantee that most commercial apps can run nicely in the emulator.

### **Install Android on QEMU**

First, we build a basic Android x86 installation as a QEMU image. We create a disk image file as the Android's storage by:

    $ qemu-img create -f qcow2 android.img 8G

This creates a `qcow2` format image naming "android.img" with 8GB capacity in current directory. Then we start the QEMU instance for installation by:

    $ qemu-system-i386 -cdrom android_x86.iso -boot order=d -hda android.img -enable-kvm -machine q35

Modify the value of `-cdrom` parameter to the path to your own Android x86 ISO image. Note that machine type is specified as q35, for default machine type (i440fx) doesn't support some advanced features, leading to error during disk partition step.

**Choose to 1) install GRUB 2) install /system folder with r/w permission during installation.** After installation, close current QEMU, and start a new instance by

    $ qemu-system-i386 -hda android.img -enable-kvm -machine q35 -net nic -net user,hostfwd=tcp::4444-:5555

This command forwards ADB port to local 4444. One can connect to ADB by

    $ adb connect localhost:4444

Finally, enable x86-ARM translation by
1. Enable `App Compatibility` in `Settings`
2. Enter ADB shell, and

        $ su
        $ enable_nativebridge

Now we get a QEMU instance capable of running ARM apps, with debugging support.

### **Adjust Screen Orientation**

By default, Android x86 runs in tablet mode, with horizontal orientation. To make orientation vertical, one has to enter the debug mode (by choosing in GRUB menu), and use

    $ mount -o rw,remount /mnt

to make `/mnt/grub/menu.lst` writable. Then edit `menu.lst`, add `video=768x1280 DPI=320` in kernel parameters. Start the QEMU instance again, we get a 768x1280 screen like what's in a normal phone.

### **Install Google Service**

One can install OpenGApps in order to provide Google Play Services for apps depending on it. To do so,

1. Download OpenGApps Pico version from [opengapps.org](http://opengapps.org/)
2. Extract packages:

        $ unzip open_gapps-x86_64-6.0-pico-20170304.zip 'Core/*'
        $ rm Core/setup*
        $ lzip -d Core/*.lz
        $ for f in $(ls Core/*.tar); do
        $ tar -x --strip-components 2 -f $f
3. Install packages:

        $ adb remount
        $ adb push etc /system/etc
        $ adb push framework /system/framework
        $ adb push app /system/app
        $ adb push priv-app /system/priv-app
4. Restart the emulator.

After reboot, one might encounter "Unfortunately, Google Play Services has stopped" message. From Logcat, one can find the cause is that `ACCESS_FINE_LOCATION` permission is not granted to the `com.google.android.gms` package. To solve the problem, use

    $ adb shell pm grant com.google.android.gms android.permission.ACCESS_FINE_LOCATION

and restart the emulator.

### **Add GPS Support**

There is no native GPS emulator support in Android x86. One can use `Fake GPS Location` app to mock GPS signal, like [this one](https://play.google.com/store/apps/details?id=com.lexa.fakegps).
Without the loss of generality, one can enable other sensor data by similar methods.

### **Conclusion**

Now we get a QEMU image ready for scaled deployment. As a QEMU image, many QEMU techniques would apply, like machine state snapshot, computing resource allocation, etc.

### **References**

1. [Installing Google Play Services on an Android Studio Emulator](https://medium.com/@dai_shi/installing-google-play-services-on-an-android-studio-emulator-fffceb2c28a1)

[andx86]: http://www.android-x86.org/
[avd]: https://developer.android.com/studio/run/emulator.html
[geny]: https://www.genymotion.com/
[23pin]: http://23pin.logdown.com/posts/697026
