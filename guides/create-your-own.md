# Create your own boot logo and animation

## Boot animation

A `bootanimation.zip` is a plain archive with a `desc.txt` file and one or more folders of numbered PNG frames:

```
bootanimation.zip
├── desc.txt
└── part0/
    ├── 000000001.png
    ├── 000000002.png
    └── …
```

`desc.txt` describes the size, speed and playback. The Fioriture Sintetiche animation uses:

```
870 433 20
p 0 0 part0
```

| Line | Meaning |
| ---- | ------- |
| `870 433 20` | Width, height and frames per second. Android centres the animation on screen. |
| `p 0 0 part0` | Play `part0`, looping forever (`0`) with no pause (`0`) until boot finishes. Use `p 1 0 part0` to play it once. |

Several parts play in order, one line each. Frameo's original animation, for example, uses five parts, with `p 1 8 part1` pausing 8 frames at the end of `part1`.

### From a video

Extract the frames with [FFmpeg](https://ffmpeg.org/download.html), scaled to your chosen size:

```sh
mkdir part0
ffmpeg -i animation.mp4 -vf "fps=20,scale=870:433" part0/%09d.png
```

### Build the zip

The archive **must be stored without compression**, otherwise Android shows a black screen. With 7-Zip:

```sh
7z a -tzip -mx0 bootanimation.zip desc.txt part0
```

Or with `zip` on Linux/macOS: `zip -0 -r bootanimation.zip desc.txt part0`. You can check the result with `7z l -slt bootanimation.zip`: every entry must say `Method = Store`.

## Boot logo

The logo must have exactly the same format as the original: check it first with `magick identify`. For the 800 × 1280 panels used here that is a 24-bit BMP3:

```sh
magick my-logo.png -resize 800x1280! -background white -alpha remove -type TrueColor BMP3:logo.bmp
magick identify logo.bmp
```

The second command should print `BMP3 800x1280 … 8-bit sRGB`, the same as the original. Adjust `800x1280` and the background colour to your frame and artwork.
