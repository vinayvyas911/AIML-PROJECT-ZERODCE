# AIML Project: Noise Detection, Removal and Low-Light Enhancement

This is our AIML project. The idea is simple: dark images from a phone or camera often come with noise, and when we brighten them the noise also becomes brighter and the image looks worse. So in this project we first add noise to images, then find out which type of noise it is, remove it, and then brighten the image using Zero-DCE++.

## What the project does

1. Takes low-light images.
2. Adds 3 types of noise to them.
3. Uses a small CNN to guess which noise was added.
4. Removes the noise using a denoising network that we trained.
5. Enhances the cleaned image using the pretrained Zero-DCE++ model.
6. Checks how good the results are using accuracy, PSNR and SSIM.

## Types of noise used

| Noise | How it looks | Values we used |
|---|---|---|
| Gaussian | fine grain over the whole image | sigma = 5/255, 10/255, 20/255 |
| Poisson | grain that depends on pixel brightness | peak = 120, 60, 30 |
| Salt and Pepper | random black and white dots | 1%, 3%, 5% of pixels |

## How it works

**Noise identifier:** A small CNN that looks at the noisy image and predicts which of the 3 noises it has and roughly how strong it is. We also give it a "high-pass" version of the image (image minus its blurred version) because that shows the noise pattern more clearly.

**Denoiser:** Another CNN that takes the noisy image along with the noise type and strength, and learns to predict the noise so it can be subtracted from the image. Since it knows the noise type, it can handle each noise differently.

**Enhancer:** We did not train this part. We used the pretrained Zero-DCE++ model from the original authors (see credits).

We compared two pipelines:
- Baseline: noisy image -> Zero-DCE++
- Ours: noisy image -> identify noise -> denoise -> Zero-DCE++

## How we measured results

- **Accuracy** of the noise identifier (overall, for each noise, and a confusion matrix).
- **PSNR and SSIM** between the clean image and the denoised image.
- **PSNR and SSIM** of the final enhanced image. Our dataset does not have normal-light photos for the same scenes, so we compared against what Zero-DCE++ gives on the noise-free image. This is not the same as comparing with a real bright photo, so the numbers should be read with that in mind.

## Results

| Noise | Identification accuracy | Noisy PSNR | Denoised PSNR | Baseline PSNR | Our PSNR |
|---|---|---|---|---|---|
| Gaussian | | | | | |
| Poisson | | | | | |
| Salt and Pepper | | | | | |
| Overall | | | | | |

The full scores are in `results/metrics.csv` and the images are in `results/figures/`.

## Folder structure

```
notebook/ZeroDce.ipynb    main notebook with all the code
results/metrics.csv       scores for every image and noise level
results/figures/          confusion matrix and before/after images
requirements.txt          libraries needed
```

## How to run

1. Open `notebook/ZeroDce.ipynb` in Google Colab.
2. Run the cells one by one from the top. The notebook downloads the original Zero-DCE code and weights by itself, so they are not kept in this repo.
3. Some cells use `files.upload()`, which only works in Colab.

To run on your own computer: `pip install -r requirements.txt` and open the notebook in Jupyter. You will need to change the upload cells to normal file paths.

## Dataset

We used the sample low-light images that come with the Zero-DCE repository (the DICM folder). We split them 80% for training and 20% for testing.

## Limitations

- The dataset is small, so the results are only a rough idea.
- The noise is added by code, real camera noise is more complicated.
- We trained the denoiser for only 15 epochs and the loss went down slowly, so it can be improved with more training.
- Poisson noise on very dark pixels is very weak, so the identifier sometimes mixes it up with Gaussian noise.

## Credits

- Zero-DCE: Guo et al., "Zero-Reference Deep Curve Estimation for Low-Light Image Enhancement", CVPR 2020. Code: https://github.com/Li-Chongyi/Zero-DCE
- Zero-DCE++: Li, Guo and Loy, "Learning to Enhance Low-Light Image via Zero-Reference Deep Curve Estimation", TPAMI 2021. Code: https://github.com/Li-Chongyi/Zero-DCE_extension

The enhancement model is from the above authors. The noise code, noise identifier, denoiser and evaluation are written by us for this project.

## Submitted by

Vinay Vyas and Anklesh Pathak

## Submitted to

Ankush Jain
