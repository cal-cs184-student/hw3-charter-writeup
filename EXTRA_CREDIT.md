# Extra-credit status

We implemented four of the listed extensions:

| Extension | Evidence |
| --- | --- |
| Jittered pixel sampler | Fixed-sample shaded and normal-shaded comparisons against the random sampler, with a high-sample reference and image error measurements. |
| Surface area heuristic | Optional 12-bin BVH split rule, with construction and rendering times on three scenes. |
| Iterative BVH construction and traversal | Optional `std::stack` implementations, compared with recursion on the same three scenes. |
| Bilateral denoising | Optional 5 by 5 filter in linear RGB, paired noisy and filtered renders, and comparison with a high-sample reference. |

All explanations, images, and measurements belong in Part 6 of the report.
The render manifest records the exact commands. The required random sampler,
midpoint BVH, and unfiltered output remain the defaults.

## Awaiting artistic direction

We deferred **motion blur**. To proceed, we need the scene or subject, the
desired movement, and how much blur to show. The listed extension requires
storing start and end vertex positions and sampling time along each ray.

## Other extensions not implemented

These are technical extensions. They do not need artistic direction, but they
are not part of the completed implementation:

- More compact BVH storage with measured memory savings. The starter already
  stores primitive pointers in one vector; we do not claim that as our work.
- New live-rendering interface features, incremental saves, or multipass buffers.
- A separate profiled parallelization or arithmetic speedup.
- A more advanced adaptive sampling rule.
- An alternative acceleration structure such as a k-d tree or octree.
- Non-axis-aligned bounding boxes with measured traversal benefits.
- Bidirectional path tracing.
- Photon mapping.
- Metropolis Light Transport.
- Vertex Connection and Merging.
- GPU execution.

We kept the main code within the course's path-tracing design. The advanced
transport methods and GPU execution would require much larger changes. The
[official extra-credit page](https://cs184.eecs.berkeley.edu/fa26/hw/hw3/ec/)
describes the full set of options and asks students to contact course staff
before attempting the highest challenge level.
