# STRATA

!!! warning
    This page was written in November 2026 and may be out of date. When in doubt, check the paper and release page.

A kilometer-scale global atmosphere emulator of SCREAM, released by [NVIDIA](https://www.nvidia.com/). It was trained on 17 days of SCREAMv1 output and takes 10-minute steps with 24 vertical levels.

- **Paper:** [Hu et al. 2026](https://arxiv.org/abs/2606.31248)
- **Architecture:** A vision-transformer-style (ViT) model that works on image patches, with 3D neighborhood attention ([NATTEN](https://github.com/SHI-Labs/NATTEN))
- **Grid:** SCREAM's native ne1024pg2 cubed-sphere grid (about 4.9 km), processed as overlapping tiles
- **Release:** [Hugging Face](https://huggingface.co/nvidia/strata)
- **Code:** [NVlabs/STRATA](https://github.com/NVlabs/STRATA)
- **Training data:** Linked from the release page

To get started, visit the [NVlabs/STRATA repository](https://github.com/NVlabs/STRATA).
