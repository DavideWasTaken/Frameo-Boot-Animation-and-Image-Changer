# Rockchip: change the boot animation

The boot animation is the clip Android plays while it starts, after the static logo. On these frames it is a normal file, `/system/media/bootanimation.zip`, so no flashing is needed: you only need root access over ADB to replace it.

**You need:** ADB, [adbd Insecure v2.00](https://forum.xda-developers.com/showthread.php?t=1687590) and your new `bootanimation.zip` ([how to make one](create-your-own.md)).

## 1. Get root access over ADB

Install adbd Insecure and open it from the PC. The frame only shows its photo app, so `monkey` starts adbd Insecure for you:

```sh
adb install adbd-Insecure-v2.00.apk
adb shell pm list packages | findstr chainfire
adb shell monkey --pct-syskeys 0 -p eu.chainfire.adbd -c android.intent.category.LAUNCHER 1
```

On the frame's screen, enable **Enable insecure adbd**. ADB now runs as root.

## 2. Back up the original animation

Save a copy on the PC before changing anything:

```sh
adb pull /system/media/bootanimation.zip bootanimation-original.zip
```

## 3. Make `/system` writable and keep the original on the frame

```sh
adb shell
su
mount -o rw,remount /dev /system
mv /system/media/bootanimation.zip /system/media/bootanimationFRAMEO.zip
exit
exit
```

## 4. Copy the new animation

```sh
adb push bootanimation.zip /sdcard/
adb shell
su
mv /sdcard/bootanimation.zip /system/media/
chmod 644 /system/media/bootanimation.zip
exit
exit
adb reboot
```

The new animation plays on the next boot.

## Restore the original

```sh
adb shell
su
mount -o rw,remount /dev /system
mv /system/media/bootanimationFRAMEO.zip /system/media/bootanimation.zip
chmod 644 /system/media/bootanimation.zip
```

## Troubleshooting

| Symptom | Fix |
| ------- | --- |
| `su: not found` or `Permission denied` | adbd Insecure is not active: reopen it and enable insecure adbd again. |
| `Read-only file system` | Run the `mount -o rw,remount` line again in the same `su` shell. |
| Black screen instead of the animation | The zip is compressed or `desc.txt` is wrong. Rebuild it as described in [Create your own files](create-your-own.md). |
