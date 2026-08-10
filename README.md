<img width="900" alt="AUTO Arm assembling an optical bench" src="https://github.com/user-attachments/assets/158b2e49-fe04-482e-b284-c9e55afadde4">

# AUTO Arm — Autonomous Optical Assembly

**A robot that assembles complex optical configurations to 0.1 mm and 0.1°, roughly 90% faster than doing it by hand.**

Built for the **MIT theoretical physics lab**. Setting up an optical bench — placing mirrors, beam splitters and mounts to interferometric tolerance — is slow, repetitive, and bounded by human patience rather than human skill. AUTO Arm does it autonomously: it finds components in a parts bank, picks them up, and places them accurately enough to assemble a working interferometer.

The longer-term aim is to let ML search over bench layouts directly, which is what makes automated second-harmonic generation experiments tractable.

**Resulted in a publication and a patent.** [Paper](https://anonymous.4open.science/r/AutomateOptics-7C7C/README.md)

---

## How it reaches 0.1 mm

Precision at this scale does not come from one good sensor. It comes from handing off between a wide, coarse sensor and a narrow, accurate one:

1. **ArUco fiducials** mark every component in the parts bank, so the system knows what it is looking at as well as where.
2. **A 4K stereo pair** localizes the tag coarsely across the whole bench — wide field of view, enough accuracy to get the arm into the neighbourhood.
3. **The arm repositions**, placing the wrist-mounted RealSense directly above the tag.
4. **The RealSense refines** position, depth and orientation up close, where its accuracy is highest. Orientation is recovered from the tag corner geometry.
5. **The pick executes** at 0.1 mm translational and 0.1° rotational precision.

The coarse-to-fine handoff is the core idea: the stereo rig alone cannot resolve 0.1 mm across a bench, and the depth camera alone cannot find a component it is not already pointed at.

## Hardware

| | |
|---|---|
| Arm | UFACTORY xArm 7 |
| Stereo | 2 × [ELP 4K USB cameras](https://www.amazon.com/dp/B0BVFKTM6Z) (5–50 mm varifocal) |
| Depth | Intel RealSense D435, wrist-mounted |
| Host | Windows 10, Python 3.10 |

## Code

| file | role |
|---|---|
| `fulladjustment.py` | **main entry point** — the full autonomous pick-and-place routine |
| `alginment.py` | interferometer assembly, used to test end-to-end placement precision |
| `aligment_with_drop.py` | interferometer assembly including mirror release |
| `beam_aligment.py` | beam alignment routines |
| `test_center_beam_code.py` | beam-centering tests |
| `fine_adjustment.py` | close-range refinement using the RealSense alone |
| `mainv2.py` | earlier version: pure stereo, no depth sensing |
| `finding_camera.py` | enumerate camera ports (indices vary per machine — run this first) |
| `internaltest.py` | rotation-math validation |
| `motor_control_test.py` | low-level arm motion checks |
| `thorcam.py` | ThorLabs camera interface |
| `stereo_calibration.npz` | saved stereo calibration intrinsics/extrinsics |

## Getting started

```bash
pip install -r requirements.txt
python finding_camera.py     # find your camera port indices, then set them in the scripts
python fulladjustment.py     # full autonomous pick-and-place
```

Camera port indices are machine-specific and are the most common first-run failure. Calibration lives in `stereo_calibration.npz`; regenerate it if you change lenses or baseline.

For choosing lenses and working distances, [this note on focal length and field of view](https://www.edmundoptics.com/knowledge-center/application-notes/imaging/understanding-focal-length-and-field-of-view/) is the reference used when configuring the stereo rig.

## Related

The fiducial detection, stereo calibration and 6D pose estimation this system is built on were developed in **[AprilTag_Detection](https://github.com/ShrishChou/AprilTag_Detection)**.

## Repository note

This repository carries its committed `.venv/`, build output, and captured frame archives alongside the source, so a clone is large. The code you want is the top-level Python files listed above.
