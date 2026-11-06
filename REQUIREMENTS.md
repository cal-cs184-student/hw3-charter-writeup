# HW3 requirements and evidence checklist

We audited the official Fall 2026 instructions on November 5, 2026. This file records the requirements before implementation and preserves the full evidence checklist. An unchecked box means that we have not verified the item here; it is not a claim that no implementation exists. The report and render manifest must give the final completion status.

Sources: [Getting Started](https://cs184.eecs.berkeley.edu/fa26/hw/hw3/getting-started/), [Part 1](https://cs184.eecs.berkeley.edu/fa26/hw/hw3/part1/), [Part 2](https://cs184.eecs.berkeley.edu/fa26/hw/hw3/part2/), [Part 3](https://cs184.eecs.berkeley.edu/fa26/hw/hw3/part3/), [Part 4](https://cs184.eecs.berkeley.edu/fa26/hw/hw3/part4/), [Part 5](https://cs184.eecs.berkeley.edu/fa26/hw/hw3/part5/), [Extra Credit](https://cs184.eecs.berkeley.edu/fa26/hw/hw3/ec/), and [Deliverables](https://cs184.eecs.berkeley.edu/fa26/hw/hw3/deliverables/).

## Required implementation

### Part 1: Camera rays and intersections

- [ ] Implement `Camera::generate_ray` in `src/pathtracer/camera.cpp`. Map normalized image coordinates to the sensor at camera-space z = -1. Use half-field-of-view tangents, converting degrees to radians. Image origin is bottom left. Transform the direction with `c2w`, normalize it, use `pos` for the origin, and set the valid ray range to `nclip` through `fclip`.
- [ ] Implement `PathTracer::raytrace_pixel` in `src/pathtracer/pathtracer.cpp`. Draw `ns_aa` uniform samples within each pixel, normalize by image dimensions, generate camera rays, estimate their radiance, and store their mean in `sampleBuffer`.
- [ ] Implement `Triangle::has_intersection` and `Triangle::intersect` in `src/scene/triangle.cpp`. Accept hits only within the ray's valid interval and shorten `max_t` on a hit. The detailed form must record t, the interpolated vertex normal using triangle weights, the primitive pointer, and its material.
- [ ] Implement `Sphere::has_intersection` and `Sphere::intersect` in `src/scene/sphere.cpp`. Select the nearest valid root, including the farther root when the nearer root is invalid. Shorten `max_t` and fill the same intersection fields. Compute the normalized outward sphere normal.

### Part 2: Bounding volume hierarchy

- [ ] Record a timing baseline without acceleration before comparing the completed implementation. The cow example uses eight threads and 800 × 600 pixels.
- [ ] Implement `BVHAccel::construct_bvh` in `src/scene/bvh.cpp`: enclose all primitives in a node box; make leaves at `max_leaf_size`; otherwise split using primitive-box centers and recurse. Explain the axis and split rule. Handle an empty partition so construction always terminates.
- [ ] Implement `BBox::intersect` in `src/scene/bbox.cpp`. Compute the ray's overlapping interval through the three box slabs and update `t0`/`t1`. Rays starting or ending within a box count as hits when their valid intervals overlap.
- [ ] Implement BVH any-hit and nearest-hit traversal. Reject boxes outside the ray interval. Any-hit can stop on the first primitive hit; nearest-hit must preserve the closest result across all relevant children and primitives.
- [ ] Confirm large meshes render correctly and compare timings with the same scenes, settings, and machine.

### Part 3: Direct light

- [ ] Implement `DiffuseBSDF::f` in `src/pathtracer/bsdf.cpp` using diffuse reflectance divided by pi.
- [ ] Implement `zero_bounce_radiance`: return the hit material's emission.
- [ ] Implement `estimate_direct_lighting_hemisphere`: sample uniform local hemisphere directions, transform rays into world space, intersect the scene, and use the hit surface's emission as incoming light. Include material response, cosine, sampling density, and sample-count normalization.
- [ ] Implement `estimate_direct_lighting_importance`: sample every light directly; take one sample for a point light and `ns_area_light` samples for an area light. Reject directions below the surface. Test visibility up to the sampled light distance, with epsilon bounds. Include the material, cosine, and light probability, and average per light.
- [ ] Implement the selector in `one_bounce_radiance`, controlled by `direct_hemisphere_sample`/`-H`.
- [ ] Connect emission and direct light to the top-level radiance estimate. Keep local directions for material calculations and world directions for intersections. Offset secondary-ray valid intervals with `EPS_F`.

### Part 4: Indirect light

- [ ] Implement `DiffuseBSDF::sample_f` using the provided cosine-weighted hemisphere sampler. Write both the sampled incoming direction and its probability; return the diffuse material value.
- [ ] Complete `at_least_one_bounce_radiance` and `est_radiance_global_illumination`. At each hit estimate direct light, sample one reflected direction, trace the next ray, and recursively weight its contribution by material response × cosine / sampling probability.
- [ ] Initialize each camera ray's depth to `max_ray_depth`, decrement on recursion, and stop at the cutoff or a miss.
- [ ] Support accumulated bounces (`isAccumBounces=true`, `-o 1`) and only the selected bounce (`false`, `-o 0`). Depth zero shows emission. An isolated depth n image contains the nth reflected-light contribution.
- [ ] Add Russian Roulette termination and divide surviving paths by their continuation probability. The spec suggests a termination probability of 0.3–0.4. If maximum depth exceeds one, always trace the first indirect bounce regardless of Roulette.
- [ ] Keep a way to reproduce the fixed-depth comparison separately from Roulette results. Preserve direct-only and indirect-only report modes or document how to reproduce them.

Only diffuse materials are required. The official instructions say unsupported materials may appear black. Mirror, glass, refraction, and other advanced materials are not required HW3 features.

### Part 5: Adaptive sampling

- [ ] Extend `raytrace_pixel` to track each radiance sample's brightness via `illum()`. Keep the count n, brightness sum s1, and squared-brightness sum s2.
- [ ] At batch boundaries compute mean = s1/n and variance = (s2 - s1²/n)/(n - 1). Test convergence with `1.96 * sqrt(variance/n) <= maxTolerance * mean`.
- [ ] Stop when converged or when the `ns_aa` maximum is reached, and average using the number of samples actually taken. Guard the one-sample case and roundoff in the variance.
- [ ] Fill `sampleCountBuffer` with actual sample counts so the sample-rate image is correct. Set `PART` to 5 when the starter uses that switch to write rate images.
- [ ] Support `-a <samplesPerBatch> <maxTolerance>`. The stated defaults are batch 32 and tolerance 0.05; with adaptive sampling, `-s` is the maximum sample count. The example uses `-a 64 0.05`.

## Required written explanations and render evidence

We must describe our actual implementation and observed results plainly, using “We.” We must explain problems we encountered and their fixes, identify any missing or incorrect results honestly, and end with the extent of AI assistance. We must not invent timings or claim completion from images that have not been rendered and inspected.

### Render matrix

Save original PNG output in `docs/images/` using `-f` or the program's S key. Do not use operating-system screenshots, convert PNGs, or resize image files. The recommended resolution is 480 × 360. CSS display sizing can preserve original assets. Keep the camera, exposure, resolution, and other settings matched within each comparison.

| Evidence | Scenes | Required settings | Outputs and discussion |
|---|---|---|---|
| Part 1 normal shading | A few small `.dae` scenes | Normal diagnostic shading; sample count not specified | Explain ray generation, the intersection pipeline, and the triangle method in our own words. |
| Part 2 large normal renders | A few large meshes | Normal shading with BVH | Explain construction and split choice. |
| Part 2 timing comparison | A few moderately complex scenes | Identical settings with and without BVH; official cow example: `-t 8 -r 800 600` | Record measured timings and give one paragraph of analysis. |
| Part 3 method comparison | Some scenes, matched between methods | Uniform hemisphere (`-H`) and light sampling (without `-H`) | Explain both methods; show each and compare in one paragraph. |
| Part 3 soft-shadow sweep | One scene with an area light | Light sampling; `-s 1`; `-l 1, 4, 16, 64` | Four images and a comparison of soft-shadow noise. |
| Part 4 global illumination | Some scenes | `-s 1024`, both direct and indirect light | Explain the indirect-light implementation. |
| Part 4 light separation | One scene, same view | `-s 1024`; direct-only and indirect-only | Two images; clearly distinguish the two contributions. |
| Part 4 isolated bounces | **`CBbunny.dae`** | `-s 1024`; `-m 0, 1, 2, 3, 4, 5`; `-o 0`; fixed-depth mode | Six images. Explain the second and third bounces and their contribution compared with rasterization. |
| Part 4 accumulated bounces | **`CBbunny.dae`** | `-s 1024`; `-m 0, 1, 2, 3, 4, 5`; `-o 1`; fixed-depth mode | Six images; compare against the matching isolated-bounce images. |
| Part 4 Roulette | **`CBbunny.dae`** | `-s 1024`; `-m 0, 1, 2, 3, 4, 100`; Roulette enabled | Six images; identify the termination rule used. |
| Part 4 sample sweep | One scene, same view | `-s 1, 2, 4, 8, 16, 64, 1024`; `-l 4` | Seven images showing the sample-count effect. |
| Part 5 adaptive sampling | **Two scenes** | Maximum `-s` at least 2048; `-l 1`; `-m` at least 5; adaptive sampling enabled | For each scene show a clean final render and a rate image with clearly different sampling levels across regions. Explain the algorithm and implementation. |

The 1024-sample fixed-rate comparisons should use their stated full sample count rather than stopping early with adaptive sampling. Part 5 specifically uses adaptive sampling, so its 2048 requirement is a maximum budget per pixel, not a requirement that every pixel use all 2048 samples. Keep per-render settings and measured timings in a reproducible manifest. A render can serve two requirements only when its scene and all settings match both.

### Writing and packaging checklist

- [ ] Overview: describe the renderer as a whole, our approach, and what we learned.
- [ ] Parts 1–5: include the explanations, original images, comparison paragraphs, and measured data above.
- [ ] Part 6: put **all** completed extra credit here; extra credit described elsewhere is not graded. Explain the method and relevant implementation details and provide evidence and comparison results.
- [ ] If we worked with a human partner, include a joint paragraph about collaboration, how it went, and what we learned. Do not invent a partner.
- [ ] At the end, acknowledge the actual scope of LLM help, including code generation, debugging, experiments, and report drafting where applicable.
- [ ] Publish an `index.html` report in the separate public write-up repository provisioned by Repo184, with relative asset paths and readable permissions. Keep the private starter code in the private code repository.
- [ ] Put the public webpage link at the top of the write-up.
- [ ] Export a PDF containing every required image, and visually verify its pages. The PDF is the principal graded deliverable; the public site is also required.
- [ ] Submit code to Gradescope “[Homework 3] Code: PathTracer” and the PDF to “[Homework 3] Write-Up: PathTracer,” selecting the PDF pages.

Code and report are due **Friday, November 6, 2026**. The checkpoint was due **Sunday, October 18, 2026**, with no checkpoint slip days. Homework slip days are counted from the later of the code and report submissions. These pages give dates without a deadline time; do not assume a time from this document.

The website has inconsistent point labels: Getting Started says equal weights, while its table and the detailed rubric differ. The detailed Deliverables rubric lists overview 3, Part 1 20, Part 2 17, Part 3 20, Part 4 25, and Part 5 15. We follow the actual task requirements rather than omitting anything based on those inconsistencies.

## Every official optional extra-credit opportunity

All items below are optional. Their unchecked state is the audit baseline, not an assertion about later code changes. User direction is required before an artistic extension is executed. Most listed extensions are technical and do not inherently require an artistic decision.

### Challenge level 1

- [ ] Better pixel sampling: jittered or low-discrepancy sampling, usually by extending `Sampler2D`. Compare noise and aliasing with random sampling.
- [ ] Surface-area BVH split heuristic. Measure several scenes against simpler splits.
- [ ] Iterative BVH construction and traversal using `std::stack` and loops. Compare build and render time.
- [ ] Smaller BVH storage: shared primitive-pointer storage with compact leaf ranges, or another documented compression. Compare memory usage. The starter already shares primitive storage, so do not claim that existing design as a new extension.
- [ ] Better live rendering controls, such as click-to-focus, a region that receives extra samples, incremental image saves, or a buffer that preserves individual samples for repeated passes. Describe the actual addition and show screenshots where relevant.
- [ ] Faster parallel work or inner-loop arithmetic. Profile the baseline, identify the bottleneck, and report a significant measured increase in rays per second.
- [ ] Improved adaptive-sampling rules. Compare with the assignment's confidence-interval method.

### Challenge level 2

- [ ] Motion blur: provide start/end vertex positions, sample ray times, and update scene intersections. **Deferred for artistic direction** before choosing the moving subject, motion, and scene composition.
- [ ] Another acceleration structure, such as a k-d tree or octree. Measure against the BVH.
- [ ] Bilateral image denoising. Show the effect of the filter and describe the method.

### Challenge level 3

The course asks students to contact their GSI before pursuing these advanced extensions.

- [ ] Non-axis-aligned bounding boxes, with measured traversal benefit.
- [ ] Bidirectional path tracing.
- [ ] Photon mapping.
- [ ] Metropolis Light Transport.
- [ ] Vertex Connection and Merging.
- [ ] GPU workload execution.

For any completed extension, record the code changes, explain the method, show relevant images, and include timing/speedup or memory comparisons when appropriate. For items left unimplemented, identify them explicitly in Part 6. Optional art or custom scene choices remain deferred to the user's direction.

## Reproduction controls from the official interface

| Flag | Meaning |
|---|---|
| `-s N` | Camera samples per pixel; default 1, normally a power of two. |
| `-l N` | Samples per area light; default 1. |
| `-t N` | Worker threads; default 1, official recommendation 4–8. |
| `-m N` | Maximum ray depth; default 1. |
| `-o N` | Accumulate bounces when nonzero; default 1. |
| `-f image.png` | Save a windowless render. |
| `-r W H` | Output resolution. |
| `-c camera-file` | Reuse a saved camera. |
| `-a B T` | Adaptive batch size and tolerance. |
| `-H` | Use hemisphere direct-light sampling. |
| `-p x y dx dy` | Render a selected cell in windowless mode. |

Use the S key for application screenshots and D to save camera settings. BVH visualization is available with V and arrow-key traversal. Temporary diagnostic modes added for the report must be documented so the submitted renderer can reproduce its evidence.
