<h1 align="center">CSAC 225 · Digital Image Processing</h1>

<p align="center">
  Coursework in Python and OpenCV: resizing, colour channels, intensity transformations and spatial filtering.
</p>

<p align="center">
  <img alt="Python 3.11+" src="https://img.shields.io/badge/python-3.11%2B-3776AB?logo=python&logoColor=white">
  <img alt="OpenCV" src="https://img.shields.io/badge/OpenCV-4.x%20%7C%205.x-5C3EE8?logo=opencv&logoColor=white">
  <img alt="Programs: 4" src="https://img.shields.io/badge/programs-4-blue">
  <img alt="Period: 2025-02 to 2025-03" src="https://img.shields.io/badge/period-2025--02%20to%202025--03-informational">
  <img alt="Status: coursework archive" src="https://img.shields.io/badge/status-coursework%20archive-lightgrey">
</p>

## Overview

This repository collects the programs written for the **CSAC 225 Digital Image Processing** course: three problem sets, made up of four Jupyter notebooks. Everything operates on images stored as NumPy arrays, using OpenCV for I/O and filtering and Matplotlib for plotting.

**Skills demonstrated**

- Resizing images with bilinear interpolation (`cv2.resize`)
- Splitting and merging colour channels (`cv2.split`, `cv2.merge`)
- Point operations on grayscale images: negative, global and adaptive thresholding, log, power-law (gamma), gray-level slicing and bit-plane slicing
- Spatial filtering: mean and median filters written from scratch with zero-padded borders, plus Gaussian blur, Laplacian and Sobel filters from OpenCV, and a comparison of the two edge detectors

## Table of contents

- [Getting started](#getting-started)
- [Repository structure](#repository-structure)
- [Programs](#programs)
- [Methodology](#methodology)
- [Sample results](#sample-results)

## Getting started

**Requirements:** Python 3.11 or newer (the notebooks were authored on 3.11 and 3.13; they were re-run for this README on 3.12 with OpenCV 4.11 and on 3.14 with OpenCV 5.0).

```bash
git clone <this-repo-url>
cd csac225-projects
python -m venv .venv
source .venv/bin/activate          # Windows: .venv\Scripts\activate
pip install opencv-python numpy matplotlib jupyter
```

Use `opencv-python`, not `opencv-python-headless`: the PS1 and PS2 notebooks open `cv2.imshow` windows (press a key in a window to continue).

**Run a notebook** from its own folder, because images are loaded by relative filename:

```bash
cd problem-sets/ps2-intensity-transformations
jupyter notebook intensity-transformations.ipynb
```

The same applies to `ps1-image-basics` and `ps3-spatial-filtering`.

**Missing input for PS1:** both PS1 notebooks read `VAR1.3.png`, which is not included in this repository. Put an image of that name in `problem-sets/ps1-image-basics/`, or change the filename in the notebook.

## Repository structure

```text
.
├── README.md
├── .gitignore
└── problem-sets/
    ├── ps1-image-basics/
    │   ├── problem-1-image-resizing.ipynb
    │   └── problem-2-color-channels.ipynb
    ├── ps2-intensity-transformations/
    │   ├── intensity-transformations.ipynb
    │   ├── animal-img.jpg
    │   ├── book-img.jpg
    │   └── cat-img.jpg
    └── ps3-spatial-filtering/
        ├── spatial-filtering.ipynb
        └── ningning-1080.jpg
```

## Programs

Dates are each file's **last-modified date** (YYYY-MM-DD) as recorded before this cleanup, not necessarily the date the work was first written. "—" means the date is unknown.

| Program | Description | Concept practiced | Last modified |
|---|---|---|---|
| [problem-1-image-resizing.ipynb](problem-sets/ps1-image-basics/problem-1-image-resizing.ipynb) | `resize_image_multi(image_path)` resizes an image to 1024², 512², 256², 128², 64² and 32² pixels, shows each in a window and returns them in a dictionary | Image resolution, bilinear interpolation | 2025-02-05 |
| [problem-2-color-channels.ipynb](problem-sets/ps1-image-basics/problem-2-color-channels.ipynb) | Splits an image into its blue, green and red channels, displays each, then merges them back | Colour channels, BGR ordering | 2025-03-19 |
| [intensity-transformations.ipynb](problem-sets/ps2-intensity-transformations/intensity-transformations.ipynb) | Six problems on grayscale images: negative, thresholding (global T = 120 and adaptive mean/Gaussian), log transform, gamma (0.4, 1.0, 2.5), gray-level slicing (50–100) and bit-plane slicing | Point (intensity) transformations | 2025-03-19 |
| [spatial-filtering.ipynb](problem-sets/ps3-spatial-filtering/spatial-filtering.ipynb) | Adds Gaussian noise to a photo, denoises it with mean and median filters written from scratch, then applies OpenCV's Gaussian blur, Laplacian and Sobel filters and compares the two edge detectors | Spatial filtering, noise reduction, edge detection | —¹ |

¹ The notebook's file date is 2026-10-19 because it was re-saved with its outputs cleared; that is not its authoring date, so it is shown as unknown. Its input image, [ningning-1080.jpg](problem-sets/ps3-spatial-filtering/ningning-1080.jpg), has a last-modified date of 2025-03-08.

## Methodology

Every notebook follows the same pattern: load an image with OpenCV, transform the pixel array with NumPy or OpenCV, and display the input next to the result. Images are loaded as 8-bit arrays (`uint8`, values 0–255). OpenCV uses BGR channel order, so Matplotlib figures convert to RGB first where colour is shown. The sections below state what each notebook does and which parameters it uses.

### PS1: image basics

- **Resizing.** `resize_image_multi(image_path)` reads the image, then calls `cv2.resize` with bilinear interpolation (`cv2.INTER_LINEAR`) for six fixed targets (1024², 512², 256², 128², 64², 32²). Each target is square, so the aspect ratio is not preserved. The function shows every result in its own window and returns a dictionary keyed by size (`"1024x1024"`, …).
- **Colour channels.** `cv2.split` separates the BGR image into three single-channel arrays, each displayed as grayscale. `cv2.merge` then recombines them and the merged image is displayed.

### PS2: intensity transformations

All six problems load the image as grayscale (`cv2.imread(file, 0)`) and apply a point operation, where each output pixel depends only on the same input pixel `r`.

| Problem | Operation | Parameters in the notebook |
|---|---|---|
| 1. Negative | A per-pixel loop that writes the inverted value back; the output is saved, re-read and shown beside the original | The code computes `1 - r` on `uint8` values, which wraps around; a true negative is `255 - r` |
| 2. Thresholding | `cv2.threshold` with binary and inverse-binary modes, and `cv2.adaptiveThreshold` with a mean and a Gaussian neighbourhood | Global `T = 120`; adaptive block size 21, constant `C = 80` |
| 3. Log transform | `s = c · ln(1 + r)` with `c = 255 / ln(256)`, cast to `uint8`, then min-max normalised to 0–255 | `c = 255 / ln(256)` |
| 4. Gamma (power-law) | `s = 255 · (r / 255)^γ`, shown side by side with the original for each γ | γ = 0.4, 1.0, 2.5 |
| 5. Gray-level slicing | A boolean mask selects pixels with `Imin ≤ r ≤ Imax`. Two outputs: selected pixels white and the rest black, or selected pixels white and the rest unchanged | `Imin = 50`, `Imax = 100` |
| 6. Bit-plane slicing | Plane `k` is `(r >> k) & 1`, scaled by 255 for display; planes are shown in a 2 × 4 grid with the original in the first slot. The image is then rebuilt from the four most significant planes (bits 4–7) | Planes 0–7 |

### PS3: spatial filtering

1. **Noise.** The colour image is previewed, then processed as grayscale. Noise drawn from a normal distribution (mean 0, standard deviation 1) is cast to `uint8` and added with `cv2.add`, which saturates at 255. All filters below run on this noisy image.
2. **Mean and median filters, written from scratch.** The image is zero-padded with `cv2.copyMakeBorder` by `kernel_size // 2`. A window is then slid over every pixel, and the output pixel is `np.mean` (mean filter, kernel 4) or `np.median` (median filter, kernel 5) of the window. The two results are compared side by side.
3. **Gaussian blur.** `cv2.GaussianBlur` with a 5 × 5 kernel and σ = 4.
4. **Laplacian.** `cv2.Laplacian` with a 5 × 5 aperture into a signed 16-bit result (`CV_16S`) to avoid overflow, plus an absolute-value version from `cv2.convertScaleAbs`. It is also applied to the Gaussian-blurred image to show how smoothing first reduces noise sensitivity.
5. **Sobel.** `cv2.Sobel` with a 5 × 5 aperture in x and y (`CV_16S`). The "gradient magnitude" is computed as `0.5·Gx + 0.5·Gy` with `cv2.addWeighted`, and absolute-value versions are shown too.
6. **Edge-detector comparison.** The Laplacian and Sobel results are shown together, with the notebook's conclusion that the Laplacian picks up fine detail but is more sensitive to noise, while Sobel gives smoother edges.

### How the results in this README were produced

Each notebook's code cells were run top to bottom in a scratch directory outside the repository, with `cv2.imshow` and `plt.show` replaced by log lines so no window is needed. The runs used Python 3.14 with OpenCV 5.0.0 and Python 3.12 with OpenCV 4.11.0. This confirms that every cell executes and that the outputs have the expected shapes; it does not check how the images look.

## Sample results

These are real results from running the programs headlessly (no GUI) on Python 3.14 with OpenCV 5.0.0, with each `cv2.imshow` / `plt.show` call logged instead of displayed. All four notebooks also completed without errors on Python 3.12 with OpenCV 4.11.0. Sizes depend on the input images: PS1 used a 2048 × 2048 image standing in for the missing `VAR1.3.png`.

<details>
<summary><b>problem-1-image-resizing.ipynb</b></summary>

```text
cv2.imshow  '1024x1024'  shape=(1024, 1024, 3)  dtype=uint8
cv2.imshow  '512x512'  shape=(512, 512, 3)  dtype=uint8
cv2.imshow  '256x256'  shape=(256, 256, 3)  dtype=uint8
cv2.imshow  '128x128'  shape=(128, 128, 3)  dtype=uint8
cv2.imshow  '64x64'  shape=(64, 64, 3)  dtype=uint8
cv2.imshow  '32x32'  shape=(32, 32, 3)  dtype=uint8
```

The notebook also prints the dictionary of pixel arrays to standard output; that dump is omitted here.

</details>

<details>
<summary><b>problem-2-color-channels.ipynb</b></summary>

```text
cv2.imshow  'blue channel'  shape=(2048, 2048)  dtype=uint8
cv2.imshow  'green channel'  shape=(2048, 2048)  dtype=uint8
cv2.imshow  'red channel'  shape=(2048, 2048)  dtype=uint8
cv2.imshow  'merged image'  shape=(2048, 2048, 3)  dtype=uint8
```

</details>

<details>
<summary><b>intensity-transformations.ipynb</b></summary>

```text
RuntimeWarning: overflow encountered in scalar subtract        (Problem 1)
RuntimeWarning: divide by zero encountered in log              (Problem 3)
RuntimeWarning: invalid value encountered in cast              (Problem 3)
cv2.imshow  'Original vs Processed'  shape=(427, 1280)  dtype=uint8
cv2.imshow  'Adaptive Gaussian Threshold'  shape=(1080, 1440)  dtype=uint8
cv2.imshow  'Adaptive Mean Threshold'  shape=(1080, 1440)  dtype=uint8
cv2.imshow  'Inverse Binary Threshold'  shape=(1080, 1440)  dtype=uint8
cv2.imshow  'Binary Threshold'  shape=(1080, 1440)  dtype=uint8
cv2.imshow  'Original'  shape=(1080, 1440)  dtype=uint8
cv2.imshow  'Original Image'  shape=(5464, 8192)  dtype=uint8
cv2.imshow  'Logarithmic Transformation'  shape=(5464, 8192)  dtype=uint8
cv2.imshow  'Original Image vs Gamma Correction (gamma = 0.4)'  shape=(5464, 16384)  dtype=uint8
cv2.imshow  'Original Image vs Gamma Correction (gamma = 1.0)'  shape=(5464, 16384)  dtype=uint8
cv2.imshow  'Original Image vs Gamma Correction (gamma = 2.5)'  shape=(5464, 16384)  dtype=uint8
cv2.imshow  'Original : Without Background : With Background'  shape=(427, 1920)  dtype=uint8
cv2.imshow  'Original Image and Bit Planes'  shape=(854, 2560)  dtype=uint8
cv2.imshow  'Reconstructed Image'  shape=(427, 640)  dtype=uint8
```

The Problem 1 warning comes from `1 - pixel` overflowing `uint8`. The Problem 3 warnings come from `img + 1` wrapping 255 to 0 before the logarithm is taken.

</details>

<details>
<summary><b>spatial-filtering.ipynb</b></summary>

```text
plt.show    1 panel(s): ['Original Image']
plt.show    1 panel(s): ['Noisy Image']
plt.show    2 panel(s): ['Noisy Image', 'Mean Filtered Image']
plt.show    2 panel(s): ['Noisy Image', 'Median Filtered Image']
plt.show    2 panel(s): ['Mean Filtered Image', 'Median Filtered Image']
plt.show    2 panel(s): ['Noisy Image', 'Gaussian Filtered Image']
plt.show    3 panel(s): ['Noisy Image', 'Laplacian Filtered Image', '|Laplacian Filtered Image|']
plt.show    2 panel(s): ['Laplacian', 'Gaussian + Laplacian']
plt.show    6 panel(s): ['Sobel X (Horizontal)', 'Sobel Y (Vertical)', 'Gradient Magnitude', '|Sobel X (Horizontal)|', '|Sobel Y (Vertical)|', '|Gradient Magnitude|']
plt.show    4 panel(s): ['|Laplacian Filtered Image|', 'Laplacian Filtered Image', '|Sobel Filtered Image Gradient Magnitude|', 'Sobel Filtered Image Gradient Magnitude']
```

On a 1080 × 1080 image the from-scratch mean filter took about 11 s and the median filter about 30 s in this run.

</details>
