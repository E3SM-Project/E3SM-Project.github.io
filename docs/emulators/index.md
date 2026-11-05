# Emulators

!!! warning
    These pages were written in November 2026. Emulators and their software change quickly, so details here may be out of date. When in doubt, check each emulator's paper and release page.

Emulators are machine learning models trained on E3SM output. The emulators below were released by our partners at [Ai2](https://allenai.org/) and [NVIDIA](https://www.nvidia.com/), so their papers and release pages are the authoritative source. Each page here gives the essentials and points you to them. They are listed from oldest to newest.

| Emulator | Released | Partner | Emulates |
| --- | --- | --- | --- |
| [ACE-climSST-EAMv2](ace-climsst-eamv2.md) | 2024 | Ai2 | Atmosphere, with prescribed climatological SST |
| [ACE2-EAMv3](ace2-eamv3.md) | 2025 | Ai2 | Atmosphere, with prescribed SST |
| [SamudrACE-E3SMv3](samudrace-e3smv3.md) | 2026 | Ai2 | Coupled atmosphere, ocean, and sea ice |
| [STRATA](strata.md) | 2026 | NVIDIA | Storm-resolving global atmosphere (SCREAM) |

## How they differ from running E3SM

- **Setup:** You install a Python package or use a container and download a checkpoint, rather than building the model and creating a case.
- **Cost:** Most runs need far fewer resources than the model they emulate, often just one GPU, which makes large ensembles and long runs practical.
- **Fixed design:** The physics, grid, time step, and output variables are set by training. You can generally only change the initial conditions, forcing, run length, and which outputs to save.
- **Limited validity:** An emulator is most trustworthy in the climate it was trained on. Anything else is extrapolation.
- **Reproducibility:** Stochastic emulators differ from run to run by design, so compare ensemble statistics rather than single trajectories.

## What you usually need

- **A GPU:** Run on a compute node with a GPU, not on a login node.
- **A Python environment:** Any tool will work, and each emulator's documentation lists what to install. We like [uv](https://docs.astral.sh/uv/) because it is fast. On shared systems, keep its cache somewhere with enough space, because PyTorch is large.
- **The release files:** These include a checkpoint and usually initial conditions and forcing data. Many releases use [`git-lfs`](https://git-lfs.com/), so download them to a scratch filesystem.
- **Disk space for output:** Saving every time step adds up quickly, so save only what you need.
