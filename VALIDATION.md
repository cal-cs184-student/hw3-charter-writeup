# Validation

We completed the required work on November 5, 2026, and added the motion study
on November 6, on an Apple M2 with 8 GB of memory.

- The Release build succeeds with Apple Clang and the starter C++11 settings.
- All four CTest targets pass: geometry, lighting, denoising, and motion.
- Geometry checks include 18,000 BVH comparisons against brute-force tests,
  including midpoint, median, and surface area splits with recursive and
  iterative traversal.
- Lighting checks cover diffuse sampling, analytic direct and indirect light,
  area-light probabilities, shadows, exact and accumulated bounce counts,
  roulette compensation, adaptive stopping, and jittered sample cells.
- Filter checks cover constant colors, noise smoothing, boundaries, strong
  edges, symmetry, finite results, and invalid parameter handling.
- Motion checks include 3,500 timed BVH comparisons against brute-force
  intersections, swept bounds across shutter intervals and turns, unchanged
  source geometry, and consistent primary, reflected, and shadow-ray times.
- The default static cow render remains byte-identical to its original
  one-worker normal-shaded reference after adding ray times.
- The original 63 jobs produced 126 PNGs at 480 by 360 pixels. The motion study
  adds 20 jobs and 40 PNGs at 640 by 360 pixels: four still comparisons and 16
  animation frames. Both adaptive rate maps vary across pixels.
- `figure_checksums.sha256` records the original image bytes.
- The strict report builder succeeds without missing evidence. The 22-page PDF's
  62 image placements match the webpage in order and content. The webpage
  also includes the 16-frame motion loop, which the PDF links to directly.
- We rendered every PDF page with Poppler and inspected the layouts. The final
  margin and heading placement corrections preserve readable figures and
  captions. No preview labels or unfinished placeholders remain.

The measurements are in `render_manifest.csv`. The original renderer logs
are in `docs/render_logs/` in the private code repository. Timing comparisons
use one worker; the original main renders use six. Fixed seeds do not guarantee
pixel-identical multiworker runs because
workers may claim tiles in different orders.
The motion stills and animation use four workers. Their render processes can
overlap, so their timings are reproduction records rather than speed comparisons.
For motion runs, `seconds` is the surrounding process duration measured with
a monotonic clock. Some renderer-reported `render_seconds` values exceed that
duration; we retain the raw log values but do not use them for timing claims.
