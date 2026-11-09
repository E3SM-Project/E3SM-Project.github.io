# ACE2-EAMv3

!!! warning
    This page was written in November 2026 and may be out of date. When in doubt, check the paper and release page.

An atmosphere-only emulator with prescribed SST, released by [Ai2](https://allenai.org/). It was trained on a 1970–2020 EAMv3 AMIP-style simulation with constant 2010 CO₂, and takes 6-hour steps with 8 vertical layers.

- **Paper:** [Wu et al. 2025](https://agupubs.onlinelibrary.wiley.com/doi/10.1029/2025JH000774)
- **Architecture:** Spherical Fourier neural operator (SFNO)
- **Grid:** 1° Gaussian grid (180×360)
- **Release:** [Hugging Face](https://huggingface.co/allenai/ACE2-EAMv3)
- **Code:** [ACE repository](https://github.com/ai2cm/ace)
- **Training data:** [E3SM Data Docs](https://docs.e3sm.org/e3sm_data_docs/_build/html/AITraining/simulation_data/simulation_table.html)

To get started, visit the [ACE quickstart](https://ai2-climate-emulator.readthedocs.io/en/latest/quickstart.html).
