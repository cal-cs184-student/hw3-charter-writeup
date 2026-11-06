# Pikachu dash and motion blur

We built a simplified Pikachu from closed ellipsoid and prism meshes: a yellow
body and head, black-tipped ears, red cheeks, eyes, a small smile, and a lightning
tail. A blue stage and lane markings stay fixed while the character moves.

We use one horizontal back-and-forth cycle:
`offset(t) = (1.8 * sin(2*pi*t), 0, 0)`.
Each moving triangle stores its vertices at both extremes. We evaluate the
same rigid translation for all character parts. This supports translation,
not a deforming skeleton or rotating geometry.

We sample one time uniformly between the shutter endpoints for each camera
ray. Every shadow and reflected ray retains that time. Intersections translate
the ray into the original geometry's frame, so rendering threads never mutate
the shared meshes. BVH boxes cover the complete shutter interval, including
any turning point inside it.

## Comparisons

The four report images use the same camera, 512 camera samples per pixel, four
samples per light, maximum depth three, and 640 by 360 output:

| View | Shutter interval, in cycles | What it shows |
| --- | --- | --- |
| Sharp center | 0 to 0 | Frozen character at the center. |
| Short exposure | -0.02 to 0.02 | Blur while crossing the center quickly. |
| Long exposure | -0.10 to 0.10 | A longer yellow trail and softened moving shadow. |
| Turning point | 0.23 to 0.27 | The same duration as the short exposure, near the reversal. |

The [animation](images/pikachu_dash.gif) contains 16 rendered frames covering
one full cycle. Each frame uses 128 samples per pixel, two light samples,
maximum depth two, and a shutter width of 0.06 cycles. Playback uses 80 ms per
frame. The original PNG frames remain available; the GIF only encodes them for
playback. We apply no image-space blur to create the motion effect.

We use the manifest's monotonic process duration (`seconds`) for these runs.
Some internal renderer timers exceed that duration, so their raw values are
retained for traceability rather than used as performance measurements.

## Reproduction

```sh
python3 tools/create_motion_scene.py
python3 tools/render_report.py --match p6_motion
python3 tools/render_report.py --animation --match motion_frame_
```

Use `-c dae/custom/pikachu_dash.cam` when rendering the scene directly. The
environment controls are `HW3_SHUTTER_OPEN`, `HW3_SHUTTER_CLOSE`, and
`HW3_MOTION_AMPLITUDE`. Equal shutter endpoints give a sharp view at that time.
Defaults keep existing scenes static and consume no extra random samples.
