# Validation

We completed this work on November 5, 2026, on an Apple M2 with 8 GB of memory.

- The Release build succeeds with Apple Clang and the starter C++11 settings.
- All three CTest targets pass: geometry, lighting, and denoising.
- Geometry checks include 18,000 BVH comparisons against brute-force tests,
  including midpoint, median, and surface area splits with recursive and
  iterative traversal.
- Lighting checks cover diffuse sampling, analytic direct and indirect light,
  area-light probabilities, shadows, exact and accumulated bounce counts,
  roulette compensation, adaptive stopping, and jittered sample cells.
- Filter checks cover constant colors, noise smoothing, boundaries, strong
  edges, symmetry, finite results, and invalid parameter handling.
- All 63 planned render jobs finished. We verified 126 renderer PNGs at their
  original 480 by 360 dimensions. Both adaptive rate maps vary across pixels.
- `figure_checksums.sha256` records the original image bytes.
- The strict report builder succeeds without missing evidence. The PDF has
  21 pages and 58 image placements matching the webpage in order and content.
- We rendered every PDF page with Poppler and inspected the layouts. The final
  margin and heading placement corrections preserve readable figures and
  captions. No preview labels or unfinished placeholders remain.

The measurements are in `render_manifest.csv` and the original renderer logs
are in `render_logs/`. Timing comparisons use one worker; the main renders use
six. Fixed seeds do not guarantee pixel-identical multiworker runs because
workers may claim tiles in different orders.
