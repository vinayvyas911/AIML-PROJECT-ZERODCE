# Noise-Aware Low-Light Image Enhancement

A Colab-based study of what happens when a low-light enhancement network
(Zero-DCE / Zero-DCE++) is fed images that are corrupted by noise, and whether
detecting the noise first and cleaning it improves the result.

The pipeline corrupts images with three kinds of noise, predicts which kind
was applied, removes it with a trained denoiser, brightens the cleaned image,
and measures quality and classification accuracy at every stage.

## Why this project

Low-light enhancers amplify everything in a dark image, including noise. A
curve-based enhancer such as Zero-DCE brightens pixels but has no mechanism
for suppressing grain, so noisy inputs come out visibly worse. Different noise
sources also behave differently (additive, signal-dependent, impulsive), so a
single generic cleaner is a compromise. Here the cleaner is told what it is
dealing with.

## Pipeline

```
clean low-light image
        |
   add noise  (Gaussian | Poisson | Salt & Pepper, 3 strengths each)
        |
   noise identifier  (small CNN -> noise type + strength)
        |
   noise-conditioned denoiser  (uses the predicted type and strength)
        |
   Zero-DCE++ enhancement
        |
   evaluation  (identification accuracy, PSNR, SSIM)
```

### Noise models

| Noise | Nature | Strengths used |
|---|---|---|
| Gaussian | additive, signal independent | sigma = 5/255, 10/255, 20/255 |
| Poisson | signal dependent (shot noise) | peak = 120, 60, 30 |
| Salt & Pepper | impulsive, random black/white pixels | density = 1%, 3%, 5% |

### Components

1. **Noise identifier.** A compact CNN. Its input is the noisy image plus a
   high-pass residual (image minus a 3x3 local average), which makes the noise
   pattern easier to see. It has two heads: a 3-way classifier for the noise
   type and a regressor for the strength within that type's range.
2. **Noise-conditioned denoiser.** A five-layer convolutional network that
   receives the noisy image together with the noise type (one-hot) and strength
   as extra input channels. It predicts the noise residual and subtracts it
   from the input. It is trained with a Charbonnier-style loss.
3. **Enhancer.** The pretrained Zero-DCE++ curve estimator, used as published
   and not retrained.

### Compared pipelines

- **Baseline:** noisy image -> Zero-DCE++
- **Proposed:** noisy image -> identifier -> denoiser -> Zero-DCE++

## Metrics

- **Identification accuracy:** overall, per noise type, and as a confusion matrix.
- **PSNR and SSIM** between the clean low-light image and the denoiser output.
- **PSNR and SSIM** of the enhanced result, computed against the enhancement of
  the noise-free image. The dataset has no normal-light ground truth, so this
  measures how close each pipeline gets to what Zero-DCE++ would produce without
  any noise. It is not a comparison with a true well-lit photograph.

## Results

Fill this table from `results/metrics.csv` after running the notebook.

| Noise | ID accuracy | Noisy PSNR | Denoised PSNR | Baseline PSNR | Proposed PSNR |
|---|---|---|---|---|---|
| Gaussian | | | | | |
| Poisson | | | | | |
| Salt & Pepper | | | | | |
| **Overall** | | | | | |

Sample outputs are stored in `results/figures/`.

## Repository layout

```
.
├── notebook/
│   └── ZeroDce.ipynb      # full experiment
├── results/
│   ├── metrics.csv        # per-image, per-noise, per-level scores
│   └── figures/           # confusion matrix and visual comparisons
├── requirements.txt
├── .gitignore
└── README.md
```

## How to run

1. Open `notebook/ZeroDce.ipynb` in Google Colab.
2. Run the cells in order. The notebook clones the official Zero-DCE and
   Zero-DCE++ repositories at runtime, so their code and weights are not
   stored here.
3. A GPU runtime is optional; training on CPU works but is slower.

To run locally instead:

```bash
pip install -r requirements.txt
jupyter notebook notebook/ZeroDce.ipynb
```

The notebook's upload cells (`files.upload()`) are Colab-specific and need to be
replaced with local file paths if you run it outside Colab.

## Data

The experiments use the low-light sample images that ship with the official
Zero-DCE repository (DICM and related folders), split 80/20 into training and
evaluation sets with a fixed seed. These are few images, so the numbers are
indicative rather than definitive.

## Limitations

- Small training and evaluation sets.
- Noise is synthetic; real sensor noise mixes several sources.
- The denoiser was trained for a short schedule, and its loss decreases slowly.
- Poisson noise on very dark pixels is weak, so it can be confused with
  low-level Gaussian noise.
- Enhancement quality is measured against a pseudo-reference, as described above.

## Credits

This work builds on the following, which should be cited if you reuse it:

- C. Guo, C. Li, J. Guo, C. C. Loy, J. Hou, S. Kwong, R. Cong.
  *Zero-Reference Deep Curve Estimation for Low-Light Image Enhancement.* CVPR 2020.
- C. Li, C. Guo, C. C. Loy.
  *Learning to Enhance Low-Light Image via Zero-Reference Deep Curve Estimation.*
  IEEE TPAMI, 2021 (Zero-DCE++).
- Official code: https://github.com/Li-Chongyi/Zero-DCE and
  https://github.com/Li-Chongyi/Zero-DCE_extension

The noise models, noise identifier, noise-conditioned denoiser and evaluation
code are original to this project.

## Author

Your Name, Your Institution, Your Course
