# Allwinner: change the boot logo and animation

On the Allwinner frame no flashing was needed. The new files are copied to the frame over ADB and moved into place with a root-capable file manager running on the frame itself.

**You need:** ADB, a root-capable file manager APK (we used ES File Explorer), your `bootlogo.bmp` and your `bootanimation.zip` ([how to make them](create-your-own.md)).

## 1. Install and open the file manager

The frame only shows its photo app, so start the file manager from the PC with `monkey`. For ES File Explorer:

```sh
adb install es-file-explorer.apk
adb shell monkey --pct-syskeys 0 -p com.estrongs.android.pop -c android.intent.category.LAUNCHER 1
```

For another file manager, replace `com.estrongs.android.pop` with its package name (`adb shell pm list packages` lists them).

## 2. Copy the new files to the frame

```sh
adb push bootlogo.bmp /sdcard/bootlogo.bmp
adb push bootanimation.zip /sdcard/bootanimation.zip
```

## 3. Replace the originals

In the file manager on the frame, enable root access and find the original files. Their names are self-explanatory and the file manager's search finds them straight away:

| File | Where it is |
| ---- | ----------- |
| `bootanimation.zip` | `/system/media/`, as on any Android device |
| `bootlogo.bmp` | The bootloader partition: search for `bootlogo.bmp` to find its folder |

**Rename each original** to keep a backup (for example `bootanimationFRAMEO.zip`), then move the new file from `/sdcard` into the same folder. Reboot the frame.

## The logo is upside down

The panel on this frame is mounted rotated, so the boot logo must be stored **rotated 180°**. Compare [`examples/allwinner/bootlogo.bmp`](../examples/allwinner/bootlogo.bmp) with the Rockchip one: same artwork, flipped. Rotate yours with ImageMagick:

```sh
magick logo.bmp -rotate 180 BMP3:bootlogo.bmp
```

Check the orientation of your frame's original logo first: not every panel is mounted the same way.
