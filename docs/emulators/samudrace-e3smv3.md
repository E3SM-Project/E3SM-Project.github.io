# SamudrACE-E3SMv3

!!! warning
    This page was written in November 2026 and may be out of date. When in doubt, check the paper and release page.

A stochastic coupled emulator of the atmosphere, ocean, and sea ice, released by [Ai2](https://allenai.org/). It was trained on the E3SMv3 preindustrial control simulation. The atmosphere takes 6-hour steps and the ocean takes 5-day steps.

- **Paper:** [Wu et al. 2026](https://arxiv.org/abs/2608.10277)
- **Architecture:** A stochastic spherical Fourier neural operator (SFNO) for the atmosphere (ACE2S), coupled to a ConvNeXt U-Net for the ocean ([Samudra](https://arxiv.org/abs/2412.03795))
- **Grid:** 1° Gaussian grid (180×360) for both the atmosphere and the ocean
- **Release:** [Hugging Face](https://huggingface.co/allenai/SamudrACE-E3SMv3)
- **Code:** [ACE repository](https://github.com/ai2cm/ace)
- **Training data:** [E3SM Data Docs](https://docs.e3sm.org/e3sm_data_docs/_build/html/AITraining/simulation_data/simulation_table.html)

To get started, visit the [ACE coupled model documentation](https://ai2-climate-emulator.readthedocs.io/en/latest/coupled.html).
