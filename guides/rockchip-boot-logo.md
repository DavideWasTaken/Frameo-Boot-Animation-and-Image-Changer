# Rockchip: change the boot logo

The boot logo is the first still image after power-on, before Android starts. On Rockchip frames it lives in the **`resource` partition** as two bitmaps, `logo.bmp` (bootloader) and `logo_kernel.bmp` (kernel). To change it you dump the firmware, replace the two images inside `resource.img` and flash only that partition back.

**You need:** ADB, [Rockchip DriverAssistant](https://dl.radxa.com/tools/windows/DriverAssitant_v5.0.zip), [rkDumper](https://xdaforums.com/t/tool-rkdumper-utility-for-backup-firmware-of-rockchips-devices.2915363/), [imgRePackerRK](https://forum.xda-developers.com/showthread.php?t=2257331), [RKDevTool](https://github.com/vicharak-in/rockchip-linux-tools/tree/master/windows) and [ImageMagick](https://imagemagick.org/script/download.php). Run the Windows tools as administrator.

## 1. Install the Rockchip drivers

Extract DriverAssistant, run `DriverInstall.exe` and click **Install Driver**. Reconnect the frame afterwards.

## 2. Dump the current firmware

Put the frame in loader mode:

```sh
adb reboot bootloader
```

Then run rkDumper from its folder:

```sh
rkDumper.exe dump
```

It writes every partition to `Output\Image` (`boot.img`, `resource.img`, `system.img`…) plus the partition maps `Output\config_8.cfg` and `Output\config_16.cfg`. **Keep this folder**: it is your full backup.

## 3. Unpack the resource image

Copy `Output\Image\resource.img` next to imgRePackerRK and unpack it:

```sh
imgRePackerRK.exe resource.img
```

This creates `resource.img.dump\` with `logo.bmp`, `logo_kernel.bmp` and `rk-kernel.dtb`, and the rebuild recipe `resource.img.cfg`.

> On some firmware the logo is packed inside `boot.img` instead of a separate `resource` partition. The steps are the same with `boot.img`.

## 4. Replace the logo

The new image must match the original **exactly**: same size, same bit depth, same BMP version. Check the original:

```sh
magick identify resource.img.dump\logo.bmp
```

On our frame this prints `BMP3 800x1280 … 8-bit sRGB`, a 24-bit Windows 3.x bitmap. Convert your image to the same format ([details](create-your-own.md#boot-logo)) and overwrite **both** `logo.bmp` and `logo_kernel.bmp` in `resource.img.dump\`.

## 5. Rebuild the resource image

```sh
imgRePackerRK.exe resource.img.cfg
```

This writes the new `resource.img` in the imgRePackerRK folder. The original stays safe in the rkDumper `Output\Image` folder.

## 6. Flash the resource partition

1. Open **RKDevTool** with the frame connected and booted. Wait for **Found One ADB Device** at the bottom.
2. Run `adb reboot bootloader` right away. When RKDevTool shows **Found One LOADER Device**, it is ready.
3. In the **Download Image** tab, right-click the partition list, choose **Load Config** and select `config_16.cfg` from the rkDumper output. The list now matches your frame's partitions and addresses.
4. Tick **only** the `resource` row and set its path to your new `resource.img`.
5. Tick **Write by Address** and click **Run**.

When the log reports success, the frame reboots with the new logo.

## Restore the original

Repeat step 6 with the original `resource.img` from the rkDumper `Output\Image` folder.

## Troubleshooting

| Symptom | Fix |
| ------- | --- |
| RKDevTool never shows a LOADER device | Reinstall DriverAssistant, try another USB cable or port, and run `adb reboot bootloader` again while RKDevTool is open. |
| rkDumper does not find the device | Check the frame is in loader mode and the tool runs as administrator. The rkDumper thread reports problems with some newer Rockchip driver versions. |
| Flash fails right after `Download resource…` | The partition list does not match the frame: load `config_16.cfg` from *your* dump, not someone else's. |
| Logo is garbled or the old one still shows | The BMP format differs from the original, or only one of `logo.bmp` / `logo_kernel.bmp` was replaced. |
