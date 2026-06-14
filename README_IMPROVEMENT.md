# Improvement: Progressive 3D Smoothing Filter

## Overview

This branch implements a progressive 3D smoothing filter schedule for Mip-Splatting (CVPR 2024).
See the project report (Chapter 6: Proposed Improvements, Section 6.2) for detailed motivation,
methodology, and analysis.

## Identified Limitation

The original Mip-Splatting applies a fixed-strength 3D smoothing filter from the first iteration.
This overly restricts high-frequency Gaussian exploration during early training, potentially
hindering optimal spatial distribution establishment. See report Chapter 4 (Identified Limitations)
for detailed discussion.

## Proposed Improvement

A progressive schedule that weakens the filter in early iterations and gradually restores full strength:

- **Iteration 0-15,000**: `filter_scale = 0.3` (30% strength, allowing free exploration)
- **Iteration 15,001-30,000**: Linear increase from 0.3 to 1.0 (full strength)

## Files Modified

| File | Changes |
|------|---------|
| `scene/gaussian_model.py` | Added `filter_3D_scale` attribute; modified `get_scaling_with_3D_filter` and `get_opacity_with_3D_filter`; added `set_filter_3D_scale()` method |
| `train.py` | Added progressive schedule logic before `render()` call |

## Reproduction Commands

### Environment

- Python 3.10
- PyTorch 2.1.0 + CUDA 12.1
- RTX 4090 24GB
- See original repository for full dependency installation

### Baseline (Original)

```bash
python train.py -s /path/to/bicycle -m /path/to/output/bicycle --eval
python train.py -s /path/to/room -m /path/to/output/room --eval --port 34567
```

### Improved (Progressive Filter)

```bash
python train.py -s /path/to/bicycle -m /path/to/output/bicycle_improved --eval
python train.py -s /path/to/room -m /path/to/output/room_improved --eval --port 34567
```

Note: The progressive schedule is automatically applied in `train.py` when using this branch.

## Experimental Results

| Scene | Metric | Baseline | Improved | Delta |
|-------|--------|----------|----------|-------|
| bicycle | PSNR | 25.52 | 25.495 | -0.025 |
| bicycle | SSIM | 0.780 | 0.781 | +0.001 |
| bicycle | LPIPS | 0.188 | 0.187 | -0.001 |
| room | PSNR | 31.96 | 31.760 | -0.20 |
| room | SSIM | 0.932 | 0.932 | 0.000 |
| room | LPIPS | 0.180 | 0.180 | 0.000 |

Full quantitative results, rendering outputs, and ground-truth comparisons are provided in the
submitted experiment data package. See report Chapter 6 (Experiments) for detailed analysis.

## Experiment Logs

Training logs and evaluation outputs are organized as follows:

- `logs/baseline_metrics.json` - Baseline reproduction metrics summary
- `logs/bicycle_metrics_log.txt` - bicycle baseline training/evaluation log
- `logs/room_metrics_log.txt` - room baseline training/evaluation log
- `experiment_data/metrics/metrics.json` - Complete comparison metrics (baseline vs. improved)
- `experiment_data/metrics/metrics.csv` - Comparison metrics in CSV format

Rendered images for qualitative comparison:
- `experiment_data/bicycle/baseline/` - Original Mip-Splatting renderings
- `experiment_data/bicycle/improved/` - Progressive filter renderings
- `experiment_data/bicycle/gt/` - Ground truth images
- `experiment_data/room/baseline/` - Original Mip-Splatting renderings
- `experiment_data/room/improved/` - Progressive filter renderings
- `experiment_data/room/gt/` - Ground truth images

## Conclusion

The progressive schedule did not significantly improve PSNR. Analysis suggests that 3DGS
convergence is primarily determined by the latter half of training, making early-phase
filter reduction effects negligible in the final result. See report Chapter 8 (Discussion and
Conclusion) for detailed failure analysis and future directions.

## Citation

Original paper:
```bibtex
@inproceedings{yu2024mip,
  title={Mip-Splatting: Alias-free 3D Gaussian Splatting},
  author={Yu, Zehao and Chen, Anpei and Huang, Binbin and Sattler, Torsten and Geiger, Andreas},
  booktitle={CVPR},
  year={2024}
}
```
